# Ivan Yanakiev: HFT Use Case & Corosio/Capy Requirements

**Date:** Feb 13, 2026
**Sources:** Slack conversation (6:37-8:03 AM Pacific) + Huddle (8:02-8:13 AM Pacific)
**Participants:** Vinnie, Steve Gerbino, Ivan Yanakiev, Michael Vandeberg

---

## Executive Summary

Ivan Yanakiev is a potential Corosio/Capy adopter running an HFT system built on 300KLOC of heavily customized ASIO. He's considering migration for three reasons: compile times, readability/maintainability (onboarding is hard), and performance. His architecture is a classic HFT pipeline: IO thread reads from sockets (multicast market data), immediately forks compute work (pricing graphs, XGBoost, neural nets) to a separate pinned CPU core, then writes signals back via IO. The IO thread must **never block**. Data moves between stages exclusively through **lock-free SPSC/MPMC shared-memory queues**. His P99 latency targets are 1.5us for market data and ~1us for order entry. Ivan referenced **TooManyCooks** as the gold standard for async runtime integration with ASIO, and noted that HRT and Citadel use stdexec for exactly this IO-to-compute dispatch pattern. The key message: Corosio must be either an extensible IO library that integrates cleanly with a broader async runtime, or it must provide its own cohesive IO + compute executor model. A pure IO library that can't dispatch to compute pools won't be adopted by this audience.

---

## Ivan's Background & Motivation

- **Current stack:** 300KLOC+ ASIO codebase with custom async compute pools, custom protocols
- **Pain points with ASIO:** compile times, hard to onboard new devs, ASIO backends not extensible without forking and depending on `detail/` headers
- **What would make him switch:** performance parity or better, extensibility to hook DPDK user-space network stacks without forking, clean IO/compute separation
- **Availability:** full-time job + near full-time own business; could try porting websocket or FIX code in ~1 month
- **Time zone:** works 7PM-4AM CET on personal projects; available weekends/evenings

---

## Ivan's Architecture (Reconstructed)

```
Multicast/Network (ASIO or DPDK)
        |
        v
  IO Executor (never blocks, busy poll on dedicated core)
        |
        |  fork / resume_on(compute_executor)
        v
  Compute Executor (pinned CPU core)
        |
        |  runs pricing graph (XGBoost, neural net, FPGA)
        v
  Signal produced
        |
        |  resume_on(io_executor)
        v
  IO Executor sends signal (TCP to OMS, risk manager, etc.)
```

### Key Details

1. **Component framework:** CRTP-based, zero-overhead. Each component has schema-defined inputs (e.g. protobuf). Components are small and composable.

2. **Graph construction:** Traders/quants define compute graphs via an API. A **graph optimizer** compiles these into an efficient callable. The graph executes when a market data message arrives.

3. **Executor model:** The async runtime must support:
   - `async_read` on the IO executor
   - `resume_on(compute_executor)` to dispatch the graph
   - `resume_on(io_executor)` to return and send signals
   - `fork` / `spawn` to avoid blocking the IO loop while the graph runs

4. **Data transport:** Only lock-free shared-memory queues (SPSC, SPMC, MPMC). No channels, no Go-style abstractions. Raw lock-free queues are the only mechanism.

5. **Compute targets:** CPU (XGBoost trees), GPU (neural networks), FPGA - the execution model must be flexible enough to dispatch to any of these.

6. **Migration path:** Companies are NOT rewriting their ASIO networking. They wrap ASIO behind sender/receiver abstractions and continue using it. The stdexec model layers on top of existing IO infrastructure.

7. **Market data path:** Read from socket, do compute, read from socket again. No multicast in the hot path - proxies read from shared memory and handle UDP multicast separately. Market data processors should be as fast as possible and only do IO for exchange data. Busy poll on a dedicated core, then offload processing to pinned CPU core.

8. **Latency targets:** Market data P99 = 1.5us (throughput + latency combo). Order entry = ~1us or below (pure latency, very hard to achieve).

---

## Ivan's Key Concerns About Corosio

1. **Is the angle ergonomic IO with some perf sacrifice, or full async runtime + IO?** This is the central question. If it's just ergonomic IO, HFT won't adopt it.

2. **Extensible backends without forking.** ASIO requires forking and depending on `detail/` headers to provide a custom backend (e.g. DPDK). Corosio must not have this problem. People doing DPDK don't care about POSIX API compliance or full RFC compliance - they want to squeeze maximum hardware IO performance.

3. **IO + compute must be cohesive.** Two separate frameworks for IO and execution that don't integrate = nobody will use it. Either Corosio provides the full async runtime, or it must be extensible enough that an external compute runtime (like TooManyCooks) can merge with it into one cohesive unit.

4. **CPU affinity and executor control.** TooManyCooks provides "incredible support for cpu affinity and control in an intuitive and native to the executor manner api." Corosio needs something comparable.

5. **No synchronization between IO and compute executors.** This is how TooManyCooks handles it - zero sync overhead when switching between IO and compute.

---

## Industry Context (from Ivan)

- **HRT (Hudson River Trading):** Trades trillions using stdexec approach. CppCon engineer explained precisely how they do IO-to-compute dispatch.
- **Citadel:** Migrated an entire asset class to stdexec ~6 months ago.
- **TooManyCooks:** Ivan's preferred async runtime. Has ASIO support, integrates switching between CPU and IO work, has optimizations for Clang HALO that Corosio could potentially adopt.
- **stdexec / sender-receiver:** The HFT community is adopting this model. It works well for the IO -> compute -> IO pipeline pattern. Ivan doesn't personally use it (prefers TooManyCooks) but understands why the community likes it.
- **Golang comparison:** If performance isn't the differentiator, people will just write a Go backend for 50-70k req/s with better ergonomics than anything C++ offers.

---

## Requirements for Capy & Corosio

### What Ivan's use case demands from the async runtime:

| Requirement | Description | Library |
|---|---|---|
| Multi-executor dispatch | `resume_on(executor)` to move coroutine between IO and compute pools | Corosio |
| Fork / fire-and-forget spawn | Spawn graph computation without blocking the IO read loop | Corosio |
| Lock-free queue integration | SPSC/MPMC queues as the data transport between pipeline stages | Capy |
| Never block IO thread | IO loop must be free to read next multicast message immediately | Both |
| Zero-overhead components | CRTP-based component framework with schema inputs | User code (but Capy channels could serve as the glue) |
| Flexible compute dispatch | Route work to CPU thread pool, GPU, or FPGA | Corosio executor model |
| Extensible IO backend | Hook DPDK or custom user-space stack without forking or detail headers | Corosio |
| CPU affinity control | Pin executors to cores, NUMA-aware scheduling | Corosio |
| ASIO interop | Wrap existing ASIO code to participate in multi-executor pipelines | Corosio |

### Specific Technology Gaps to Address:

1. **`resume_on(executor)` primitive in Corosio** - Coroutine must be able to cheaply switch which executor it runs on mid-flight. This is the most critical building block.

2. **Fork/spawn that returns to caller immediately** - The IO loop spawns the graph onto the compute executor and immediately resumes reading. The spawned coroutine runs independently on the compute pool.

3. **Lock-free queue adapters in Capy** - The channel abstraction needs to support (or wrap) lock-free SPSC/SPMC/MPMC queues as the underlying transport. These are the ONLY data movement mechanism in HFT pipelines.

4. **Backpressure / buffering** - Ivan mentioned buffering mechanisms for when the compute pool falls behind. The channel/queue abstraction needs configurable backpressure policy.

5. **ASIO interop layer** - Companies won't rewrite networking. Corosio must interoperate cleanly with existing ASIO-based IO code, potentially wrapping it behind a sender/receiver interface.

6. **Pluggable IO backend** - Must support custom backends (DPDK, user-space stacks) through public API, not `detail/` headers.

7. **Clang HALO optimizations** - TooManyCooks has these; Corosio should research and potentially adopt similar techniques.

---

## Connection to Earlier Huddle (7:40-8:02 AM)

This conversation confirms the hypothesis from the earlier session:

- The **external user with DPDK/user-space sockets** has the same architectural pattern as Ivan
- Their **P99 latency concern is about interior pipeline channels** (Capy's lock-free queues), not Corosio's networking layer
- The **pump mechanism** discussion is relevant to the IO executor side (throughput vs latency tradeoff for the read loop)
- The **frame recycler / `when_all` optimizations** matter for the compute dispatch side (fan-out of graph nodes)

---

## Development Plan Inputs

### Phase 1: Core Executor Primitives (Corosio)
- [ ] Implement `resume_on(executor)` for cross-executor coroutine migration
- [ ] Implement `fork` / fire-and-forget spawn onto a different executor
- [ ] Ensure the IO executor never starves when compute work is dispatched
- [ ] Validate with benchmark: multicast read loop + compute dispatch, measure whether IO thread stays saturated

### Phase 2: Lock-Free Queue Integration (Capy)
- [ ] Design channel adapter for lock-free SPSC queues
- [ ] Design channel adapter for lock-free MPMC queues
- [ ] Implement configurable backpressure policy (drop, block, buffer)
- [ ] Benchmark queue throughput under Capy's async model vs raw queue access

### Phase 3: Pipeline Composition
- [ ] Enable pipeline patterns: read -> dispatch -> compute -> return -> send
- [ ] Ensure the full pipeline can be expressed with Capy channels + Corosio executors
- [ ] Test with realistic HFT-like workload (high message rate, lightweight compute)

### Phase 4: Extensibility & Interop
- [ ] Pluggable IO backend API (support DPDK without forking or detail headers)
- [ ] ASIO interop: wrap ASIO operations behind Corosio's executor model
- [ ] Allow existing ASIO code to participate in multi-executor pipelines
- [ ] CPU affinity / NUMA-aware executor pinning
- [ ] Research Clang HALO optimizations from TooManyCooks

---

## Key Quotes

> "For me performance and low latency is the most important thing. If I see that corosio can bring that or ensures that it's extensible enough to hook a DPDK user space network stack without forking the library or depending on detail headers, I would consider it strongly." -- Ivan

> "If you have 2 separate frameworks for io and execution then it's not great and people will not use it unless the library is extensible enough so that the 2 can be merged into one cohesive unit." -- Ivan

> "People that will bother to do DPDK user space network stack are the same people that don't want LD_PRELOAD because you don't care that much about posix api or full rfc compliance you want to squeeze the maximum of the hardware in terms of IO." -- Ivan

> "If you bother to write network IO in c++ you have a very good reason for it and it's performance. Otherwise I'll just write a golang backend and will easily do 50-70k requests per second and it will be more ergonomic than anything c++ can offer." -- Ivan

> "You cannot afford on every message to pass it into the computer graph and block the IO thread." -- Ivan

> "Only shared memory queues. Lock-free SPSC or MPMC queues. That's the only way data is moved around." -- Ivan

> "They don't rewrite [ASIO networking]. They just wrap it and continue to use stdexec from now on." -- Ivan

> "We want to design Capy to address this use-case so there is an alternative to senders and receivers (which I hate)." -- Vinnie

> "High performance, HFT, compute - all of this is in-scope for Capy and Corosio. We aim to outperform everything." -- Vinnie

---

## Follow-Up

- [ ] Interview Ivan in depth (Steve to conduct, Vinnie to extract report from recording)
- [ ] Get link to CppCon talk Ivan referenced (HRT engineer, ASIO-to-stdexec migration)
- [ ] Review TooManyCooks library for design inspiration (ASIO integration, HALO optimizations, executor affinity API)
- [ ] Assess current Capy channel implementation against lock-free queue requirements
- [ ] Review Corosio executor model for `resume_on` feasibility
- [ ] Ivan to try porting websocket or FIX code to Corosio (~1 month timeframe, no hard ETA)
- [ ] Research Citadel's stdexec migration for patterns/lessons
