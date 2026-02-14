# Huddle Summary: Corosio Performance & Benchmarks

**Date:** Feb 13, 2026, 7:40-8:02 AM Pacific
**Participants:** Vinnie, Steve Gerbino

---

## Executive Summary

Steve presented Corosio benchmark results showing competitive (and sometimes superior) throughput vs ASIO callbacks, achieved primarily through the symmetric transfer pump mechanism. The pump bypasses the queue entirely, reusing coroutine frames for fast execution. However, this trades throughput for P99 latency. Steve implemented a dynamic budget algorithm to balance these tradeoffs. Vinnie raised concerns about dynamic adaptation in foundational libraries (pigeonhole principle / unseen tradeoffs) but warmed to the idea when Steve confirmed the pump is configurable. Key technical discussion also covered frame recycler exhaustion during fan-out workloads, and potential optimizations using per-algorithm frame caches (like `right_now` and a proposed `when_all` improvement). The external user they discussed uses user-space sockets (DPDK) with their own reactor, meaning Corosio's networking layer is irrelevant to them - their P99 latency concern is about interior pipeline channels (Capy domain).

---

## Key Technical Insights

- **Compilation boundary cost is proportional to coroutine size.** For small, fast coroutines the overhead of crossing a compilation unit boundary (virtual dispatch, parameter marshalling) is a much larger percentage of total time. For large data transfers dominated by syscalls, it becomes negligible.

- **Symmetric transfer is the key to beating ASIO callbacks.** It reuses the same coroutine frame (no new allocation), executes quickly, and bypasses the global queue entirely. ASIO callbacks always go through the queue.

- **The pump increases throughput but hurts P99 latency.** Allowing multiple inline completions before returning to the queue means some queued work waits longer.

- **Fan-out workloads expose frame recycler limits.** The per-thread frame recycler sustains chains of ~8 allocations. Fan-out to 64 exhausts it, forcing trips to the global pool. Peter's optimization (grab the entire global pool at once under the lock) helps but doesn't fully solve high fan-out.

- **`right_now` pattern avoids frame allocator pressure.** Instead of calling a function, you declare a stack object with a 1-element frame cache. Repeated invocations reuse that cache without touching the recycled allocator.

- **`when_all` could have its own private frame cache** sized to the arity of its parameter list (e.g., vector of 20 elements = 20-frame cache). It would use the TLS hook to install its own custom allocator, giving each child runner a frame from its private stash.

- **Double-dispatch syntax is required** for `when_all` to set the TLS hook before spawning children (the two-call-operator pattern).

- **External user (DPDK/user-space sockets) doesn't use Corosio networking.** They have their own reactor and user-space socket implementation. Their P99 latency concern is about interior pipeline channels, which is Capy territory, not Corosio.

---

## The Pump Mechanism (Current State)

Steve's dynamic budget algorithm:

1. **No contention on the queue:** Budget ramps up: 2 -> 4 -> 8 -> 16 inline completions allowed.
2. **Contention detected** (queue still has work after pulling an item): A boolean flag is set, and the budget is capped at 4.
3. **Cost:** One boolean set + one boolean check per iteration.
4. **Intent:** Configurable mechanism with the dynamic algorithm as an intelligent default.

---

## Concerns Raised

- **Pigeonhole principle / Hazlitt's "unseen" tradeoffs:** Every use case the dynamic algorithm improves may disadvantage another unseen use case. Benchmarks only cover a subset of real-world workloads.
- **Dynamic adaptation in foundational libraries is risky.** Vinnie's experience suggests these tend to have problems. But configurability mitigates this - users can override the default.
- **Must be reviewed by Peter.** The dynamic pump mechanism and its tradeoffs need Peter's sign-off.

---

## Actionable Items

- [ ] **Steve:** Run the pump mechanism design by Peter for review
- [ ] **Steve:** Increase frame cache to 128 temporarily to measure fan-out impact and understand allocation pressure
- [ ] **Steve:** Look at the `right_now` pattern for reducing frame allocator pressure
- [ ] **Vinnie:** Refactor frame recycler into a reusable class (memory resource) that `when_all` and similar algorithms can use as a data member with automatic frame management
- [ ] **Steve:** Prototype `when_all` with private per-invocation frame cache using double-dispatch syntax and TLS hook
- [ ] **Vinnie:** Implement DPDK support (longer-term; needs more people)
- [ ] **Both:** Ensure pump mechanism is configurable (disable pump for P99-sensitive workloads, enable for throughput-sensitive ones like HTTP servers)
- [ ] **Steve:** Address the negative benchmark numbers (specific regressions to investigate)

---

## Key Quotes

> "The symmetric transfer alleviates a lot of the overhead of the coroutine machinery because it doesn't need to do anything with the frame. It reuses the same frame and executes rather quickly." -- Steve

> "Every use case that you make better, you're making another use case worse. You have to pay attention to that which is not seen." -- Vinnie (citing Henry Hazlitt)

> "If P99 latency is one of the most important things to you, you want to disable the pump and sacrifice throughput." -- Steve

> "Someone needs to implement DPDK." -- Vinnie
