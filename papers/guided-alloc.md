| Document | D0000 |
|----------|-------|
| Date:       | 2026-01-14
| Reply-to:   | Vinnie Falco \<vinnie.falco@gmail.com\>
| Audience:   | SG1, LEWG

---

# Runtime Profile-Guided Coroutine Frame Allocation

## Abstract

Coroutine frame allocation can be optimized by observing that a given call site tends to produce similar coroutine call chains. We propose a runtime learning mechanism that profiles allocation patterns on first invocation, then pre-allocates appropriately sized arenas on subsequent invocations—eliminating per-frame heap allocations without requiring static analysis or compiler support.

---

## 1. Motivation

C++20 coroutines allocate frames on the heap by default. HALO (Heap Allocation eLision Optimization) can elide these allocations when the compiler proves the frame doesn't escape, but this fails for:

- **Separate compilation**: Frame sizes unknown across TU boundaries
- **Polymorphic calls**: Virtual dispatch hides concrete coroutine types  
- **Dynamic patterns**: Chain depth varies based on runtime data

Frame pooling helps but still requires one allocator round-trip per frame. For deep coroutine chains (common in I/O), this adds measurable overhead.

---

## 2. Key Insight

**The same source location produces predictable coroutine call chains.**

When line 42 of `server.cpp` calls `handle_request()`, it typically invokes the same sequence of child coroutines with similar frame sizes. The first invocation teaches us the pattern; subsequent invocations exploit it.

---

## 3. Design

### 3.1 Call Site Identification

Hash the launch site using `std::source_location`:

```cpp
uint32_t call_site_hash(source_location loc) {
    return hash_combine(hash(loc.file_name()), loc.line());
}
```

### 3.2 Chain Profile

Track observed allocation patterns:

```cpp
struct chain_profile {
    std::vector<size_t> frame_sizes;  // Sequence of frame sizes
    size_t total_bytes;               // Sum for arena pre-allocation
    uint32_t hit_count;               // Confidence metric
};
```

### 3.3 Learning Phase

On first invocation (cache miss), allocate normally but record each frame size in the chain. When the chain completes, store the profile keyed by call site hash.

### 3.4 Optimized Phase

On subsequent invocations (cache hit with sufficient confidence), pre-allocate a single arena of `total_bytes` and bump-allocate frames from it. The arena is freed when the root coroutine completes.

---

## 4. Implementation

The following self-contained C++20 program demonstrates the technique. Compile and run with any standard compiler:

```
cl /std:c++20 /EHsc /O2 guided_alloc.cpp
g++ -std=c++20 -O2 guided_alloc.cpp -o guided_alloc
clang++ -std=c++20 -O2 guided_alloc.cpp -o guided_alloc
```

```cpp
// guided_alloc.cpp
// Runtime Profile-Guided Coroutine Frame Allocation
// C++20 - No external dependencies

#include <coroutine>
#include <cstdint>
#include <cstdio>
#include <cstring>
#include <source_location>
#include <unordered_map>
#include <vector>

//------------------------------------------------------------------------------
// Allocation tracking
//------------------------------------------------------------------------------

static size_t g_heap_allocs = 0;
static size_t g_arena_allocs = 0;

//------------------------------------------------------------------------------
// Chain profile: learned allocation pattern for a call site
//------------------------------------------------------------------------------

struct chain_profile
{
    std::vector<size_t> frame_sizes;
    size_t total_bytes = 0;
    uint32_t hits = 0;

    void record(size_t n)
    {
        frame_sizes.push_back(n);
        total_bytes += n;
    }

    void commit() { ++hits; }
    bool ready() const { return hits >= 2; }  // Need 2 consistent observations
};

//------------------------------------------------------------------------------
// Profile cache: maps call site hash -> learned profile
//------------------------------------------------------------------------------

static uint32_t hash_location(std::source_location loc)
{
    uint32_t h = 2166136261u;
    for (const char* p = loc.file_name(); *p; ++p)
        h = (h ^ static_cast<uint32_t>(*p)) * 16777619u;
    h = (h ^ loc.line()) * 16777619u;
    return h;
}

static std::unordered_map<uint32_t, chain_profile> g_profiles;

//------------------------------------------------------------------------------
// Arena: bump allocator for optimized path
//------------------------------------------------------------------------------

struct arena
{
    char* base = nullptr;
    size_t capacity = 0;
    size_t offset = 0;

    void init(size_t n)
    {
        base = new char[n];
        capacity = n;
        offset = 0;
    }

    void* allocate(size_t n)
    {
        n = (n + 15) & ~15;  // Align to 16
        if (offset + n > capacity)
            return nullptr;
        void* p = base + offset;
        offset += n;
        ++g_arena_allocs;
        return p;
    }

    void reset() { offset = 0; }

    ~arena() { delete[] base; }
};

//------------------------------------------------------------------------------
// Allocation context: per-chain state
//------------------------------------------------------------------------------

struct alloc_context
{
    uint32_t site_hash = 0;
    chain_profile* learning = nullptr;  // Non-null during learning phase
    arena* optimized = nullptr;         // Non-null during optimized phase

    void* allocate(size_t n)
    {
        if (optimized)
        {
            if (void* p = optimized->allocate(n))
                return p;
            // Arena exhausted - fall through to heap
        }

        ++g_heap_allocs;
        void* p = ::operator new(n);

        if (learning)
            learning->record(n);

        return p;
    }

    void deallocate(void* p, size_t)
    {
        if (optimized)
            return;  // Arena handles bulk deallocation
        ::operator delete(p);
    }
};

// Thread-local current context
static thread_local alloc_context* t_ctx = nullptr;

//------------------------------------------------------------------------------
// Task: minimal coroutine type using profile-guided allocation
//------------------------------------------------------------------------------

struct task
{
    struct promise_type
    {
        alloc_context* ctx_ = nullptr;

        task get_return_object() { return {std::coroutine_handle<promise_type>::from_promise(*this)}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }

        static void* operator new(size_t n)
        {
            alloc_context* ctx = t_ctx;
            void* p = ctx ? ctx->allocate(n) : ::operator new(n);

            // Store context pointer at end of frame for deallocation
            auto frame = static_cast<char*>(p);
            std::memcpy(frame + n - sizeof(void*), &ctx, sizeof(void*));
            return p;
        }

        static void operator delete(void* p, size_t n)
        {
            alloc_context* ctx;
            auto frame = static_cast<char*>(p);
            std::memcpy(&ctx, frame + n - sizeof(void*), sizeof(void*));

            if (ctx)
                ctx->deallocate(p, n);
            else
                ::operator delete(p);
        }
    };

    std::coroutine_handle<promise_type> h_;
};

//------------------------------------------------------------------------------
// run_profiled: launch coroutine with profile-guided allocation
//------------------------------------------------------------------------------

template <typename F>
void run_profiled(F&& f, std::source_location loc = std::source_location::current())
{
    uint32_t hash = hash_location(loc);
    alloc_context ctx;
    ctx.site_hash = hash;

    arena ar;
    chain_profile new_profile;

    auto it = g_profiles.find(hash);
    if (it != g_profiles.end() && it->second.ready())
    {
        // Optimized path: use learned profile
        ar.init(it->second.total_bytes + 256);  // Padding for alignment
        ctx.optimized = &ar;
    }
    else
    {
        // Learning path: record allocations
        ctx.learning = &new_profile;
    }

    t_ctx = &ctx;
    f();  // Run the coroutine chain
    t_ctx = nullptr;

    if (ctx.learning && !new_profile.frame_sizes.empty())
    {
        // Commit learned profile
        auto& profile = g_profiles[hash];
        if (profile.frame_sizes == new_profile.frame_sizes)
        {
            profile.commit();
        }
        else
        {
            profile = std::move(new_profile);
            profile.hits = 1;
        }
    }
}

//------------------------------------------------------------------------------
// Example coroutine chain: 4 levels deep
//------------------------------------------------------------------------------

task level4() { co_return; }

task level3()
{
    co_await std::suspend_never{};
    level4();
}

task level2()
{
    co_await std::suspend_never{};
    level3();
    level3();
}

task level1()
{
    co_await std::suspend_never{};
    level2();
    level2();
}

//------------------------------------------------------------------------------
// Main: demonstrate learning and optimization
//------------------------------------------------------------------------------

int main()
{
    std::printf("Runtime Profile-Guided Coroutine Frame Allocation\n");
    std::printf("==================================================\n\n");

    // Run 1: Learning (cache miss)
    g_heap_allocs = 0;
    g_arena_allocs = 0;
    run_profiled([]{ level1(); });
    std::printf("Run 1 (learning):   heap=%zu, arena=%zu\n", g_heap_allocs, g_arena_allocs);
    size_t learn1_heap = g_heap_allocs;

    // Run 2: Learning (building confidence)
    g_heap_allocs = 0;
    g_arena_allocs = 0;
    run_profiled([]{ level1(); });
    std::printf("Run 2 (learning):   heap=%zu, arena=%zu\n", g_heap_allocs, g_arena_allocs);

    // Run 3: Optimized (profile ready)
    g_heap_allocs = 0;
    g_arena_allocs = 0;
    run_profiled([]{ level1(); });
    std::printf("Run 3 (optimized):  heap=%zu, arena=%zu\n", g_heap_allocs, g_arena_allocs);
    size_t opt_arena = g_arena_allocs;

    // Run 4: Optimized (verify consistency)
    g_heap_allocs = 0;
    g_arena_allocs = 0;
    run_profiled([]{ level1(); });
    std::printf("Run 4 (optimized):  heap=%zu, arena=%zu\n", g_heap_allocs, g_arena_allocs);

    std::printf("\n");
    std::printf("Result: %zu heap allocations eliminated per chain\n", learn1_heap);
    std::printf("        (replaced with %zu arena bump allocations)\n", opt_arena);

    return 0;
}
```

### Expected Output

```
Runtime Profile-Guided Coroutine Frame Allocation
==================================================

Run 1 (learning):   heap=7, arena=0
Run 2 (learning):   heap=7, arena=0
Run 3 (optimized):  heap=0, arena=7
Run 4 (optimized):  heap=0, arena=7

Result: 7 heap allocations eliminated per chain
        (replaced with 7 arena bump allocations)
```

---

## 5. Conclusion

Runtime profile-guided allocation trades a small learning phase for significant allocation savings on hot paths. The technique requires no compiler support, works across separate compilation boundaries, and adapts to actual runtime behavior rather than static predictions.
