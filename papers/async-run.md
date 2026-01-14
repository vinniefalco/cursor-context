# Suspended Coroutine Launcher Pattern

A technique for wrapping user coroutines while controlling frame allocation order and minimizing the total number of coroutine frames.

---

## Problem Statement

When composing coroutines, we often want a "launcher" or "runner" that wraps a user's task to provide context, exception handling, or resource management. The naive approach creates an extra coroutine frame:

```cpp
// Naive: 3 frames (run + wrapper + user_task)
task<int> run() {
    co_return co_await wrapper(user_task());
}

task<int> wrapper(task<int> inner) {
    co_return co_await inner;  // wrapper's frame just forwards
}
```

We want:
1. The wrapper's frame to allocate **before** the user task's frame
2. Only **two** frames total (launcher + user task), not three

---

## C++17 Evaluation Order Guarantee

The solution exploits a C++17 sequencing rule. For a postfix call expression `E1(E2)`:

> The postfix expression `E1` is sequenced before the argument expression `E2`.

This means in `launcher()(user_task())`:
1. `launcher()` executes first, allocating its coroutine frame
2. `user_task()` executes second, allocating its frame
3. `operator()` is called with the user task

The frame allocation order is guaranteed by the language.

---

## Design Overview

The pattern has three components:

1. **`launcher()`** — A coroutine that allocates a frame and immediately suspends. Its body later transfers control to an inner task stored in its promise.

2. **`operator()`** — A regular function (not a coroutine) on the launcher's return type. It stores the user task's handle in the promise and returns an awaitable.

3. **`launch_awaitable`** — Returned by `operator()`. When awaited, it sets up the continuation chain and resumes the launcher, which transfers to the inner task.

The key insight: `operator()` does not create a new frame. It merely configures the existing launcher frame to work with the provided task.

---

## Complete Implementation

```cpp
#include <coroutine>
#include <exception>
#include <utility>

template<typename T> struct task;
struct launcher_wrapper;

//-----------------------------------------------------------------------------
// task<T> — A standard lazy coroutine task
//-----------------------------------------------------------------------------

template<typename T>
struct task {
    struct promise_type {
        T value;
        std::exception_ptr exception;
        std::coroutine_handle<> continuation;

        task get_return_object() {
            return task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept { return {}; }

        struct final_awaiter {
            bool await_ready() noexcept { return false; }
            std::coroutine_handle<> await_suspend(
                std::coroutine_handle<promise_type> h) noexcept {
                return h.promise().continuation;
            }
            void await_resume() noexcept {}
        };

        final_awaiter final_suspend() noexcept { return {}; }
        void return_value(T v) { value = std::move(v); }
        void unhandled_exception() { exception = std::current_exception(); }
    };

    std::coroutine_handle<promise_type> handle;

    T result() {
        if (handle.promise().exception)
            std::rethrow_exception(handle.promise().exception);
        return std::move(handle.promise().value);
    }

    ~task() { if (handle) handle.destroy(); }
    task(task&& o) noexcept : handle(std::exchange(o.handle, nullptr)) {}
    task& operator=(task&&) = delete;

private:
    friend struct launcher_wrapper;
    explicit task(std::coroutine_handle<promise_type> h) : handle(h) {}
};

//-----------------------------------------------------------------------------
// launcher_wrapper — The suspended coroutine returned by launcher()
//-----------------------------------------------------------------------------

struct launcher_wrapper {
    struct promise_type {
        std::coroutine_handle<> inner_handle;
        std::coroutine_handle<> continuation;

        std::suspend_always initial_suspend() noexcept { return {}; }

        struct final_awaiter {
            bool await_ready() noexcept { return false; }
            std::coroutine_handle<> await_suspend(
                std::coroutine_handle<promise_type> h) noexcept {
                auto cont = h.promise().continuation;
                return cont ? cont : std::noop_coroutine();
            }
            void await_resume() noexcept {}
        };

        final_awaiter final_suspend() noexcept { return {}; }

        launcher_wrapper get_return_object() {
            return launcher_wrapper{
                std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        void unhandled_exception() { std::terminate(); }
        void return_void() {}

        // Transfers control to the inner task
        struct transfer_to_inner {
            promise_type* p;

            bool await_ready() noexcept { return false; }
            std::coroutine_handle<> await_suspend(std::coroutine_handle<>) noexcept {
                return p->inner_handle;
            }
            void await_resume() noexcept {}
        };
    };

    std::coroutine_handle<promise_type> handle;

    // Awaitable returned by operator()
    template<typename T>
    struct launch_awaitable {
        std::coroutine_handle<promise_type> launcher;
        task<T>* inner;

        bool await_ready() noexcept { return false; }

        std::coroutine_handle<> await_suspend(std::coroutine_handle<> cont) {
            // Chain: caller <- launcher <- inner
            launcher.promise().continuation = cont;
            inner->handle.promise().continuation = launcher;
            return launcher;
        }

        T await_resume() {
            return inner->result();
        }
    };

    template<typename T>
    launch_awaitable<T> operator()(task<T>& inner) && {
        handle.promise().inner_handle = inner.handle;
        return launch_awaitable<T>{handle, &inner};
    }

    template<typename T>
    launch_awaitable<T> operator()(task<T>&& inner) && {
        return std::move(*this)(inner);
    }

    ~launcher_wrapper() { if (handle) handle.destroy(); }
    launcher_wrapper(launcher_wrapper&& o) noexcept
        : handle(std::exchange(o.handle, nullptr)) {}

private:
    explicit launcher_wrapper(std::coroutine_handle<promise_type> h)
        : handle(h) {}
};

//-----------------------------------------------------------------------------
// get_promise — Lets a coroutine access its own promise without suspending
//-----------------------------------------------------------------------------

struct get_promise {
    launcher_wrapper::promise_type* p;

    bool await_ready() noexcept { return false; }
    bool await_suspend(
        std::coroutine_handle<launcher_wrapper::promise_type> h) noexcept {
        p = &h.promise();
        return false;
    }
    launcher_wrapper::promise_type& await_resume() noexcept { return *p; }
};

//-----------------------------------------------------------------------------
// launcher() — The coroutine factory
//-----------------------------------------------------------------------------

inline launcher_wrapper launcher() {
    auto& promise = co_await get_promise{};
    co_await launcher_wrapper::promise_type::transfer_to_inner{&promise};
    co_return;
}
```

---

## Execution Flow

Given `co_await launcher()(user_task())`:

| Step | Action | Frames |
|------|--------|--------|
| 1 | `launcher()` called | Launcher frame allocated, suspends at `initial_suspend` |
| 2 | `user_task()` called | User frame allocated, suspends at `initial_suspend` |
| 3 | `operator()` called | Stores user handle in launcher's promise, returns `launch_awaitable` |
| 4 | `co_await launch_awaitable` | Sets continuation chain, resumes launcher |
| 5 | Launcher runs `get_promise` | Gets promise reference without suspending |
| 6 | Launcher awaits `transfer_to_inner` | Symmetric transfer to user task |
| 7 | User task executes | Runs user code to completion |
| 8 | User task `final_suspend` | Returns launcher's handle, resuming launcher |
| 9 | Launcher `final_suspend` | Returns caller's handle, resuming caller |
| 10 | `await_resume` | Extracts result from user task |

---

## Key Components

| Component | Type | Purpose |
|-----------|------|---------|
| `launcher()` | Coroutine | Allocates wrapper frame, transfers to inner task |
| `operator()` | Regular function | Stores inner handle, returns awaitable |
| `launch_awaitable` | Awaitable | Sets up continuation chain on `await_suspend` |
| `transfer_to_inner` | Awaitable | Symmetric transfer from launcher to inner task |
| `get_promise` | Awaitable | Access promise from coroutine body without suspending |
| `final_awaiter` | Awaitable | Resumes continuation via symmetric transfer |

---

## Usage

```cpp
task<int> user_task() {
    co_return 42;
}

task<int> example() {
    // Exactly 2 frames: launcher + user_task
    // Launcher frame allocated before user_task frame
    int result = co_await launcher()(user_task());
    co_return result;
}
```

The launcher can be extended to perform setup before `transfer_to_inner` and cleanup after the inner task completes:

```cpp
inline launcher_wrapper launcher() {
    auto& promise = co_await get_promise{};

    // Setup: establish context, acquire resources, etc.

    co_await launcher_wrapper::promise_type::transfer_to_inner{&promise};

    // Cleanup: release resources, handle exceptions, etc.

    co_return;
}
```
