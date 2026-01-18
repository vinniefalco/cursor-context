# Beast2 HTTP Server Design Principles

---

## Possible APIs

Sketches of API designs to evaluate against principles.

### Idea 1: Multi-port with shared router and worker counts

```cpp
// Stream factories carry their configuration
auto plain_tcp = http::plain();
auto tls = http::wolfssl(cert, key);           // or http::openssl(cert, key)
auto auto_tls = http::detect(cert, key);       // auto-detect TLS vs plain

// Two ports, router shared ownership, flatten on run()
app.bind(plain_tcp, 80, 40, my_www);    // 40 workers
app.bind(tls, 443, 10000, my_www);      // 10000 workers
app.run();
```

- Stream factory as first parameter — carries SSL context, certs, options
- Port number
- Worker count per listener
- Router (shared ownership allows same router on multiple ports)
- `run()` flattens all routers and starts listeners

**Stream factories:**

```cpp
http::plain()                         // no config needed
http::wolfssl(cert, key)              // WolfSSL with certs
http::wolfssl(ssl_ctx)                // WolfSSL with pre-built context
http::openssl(cert, key)              // OpenSSL with certs
http::openssl(ssl_ctx)                // OpenSSL with pre-built context
http::detect(ssl_ctx)                 // auto-detect, TLS or plain
```

### Idea 2: App-is-Router (one router per app)

```cpp
app.use(ws_upgrade(my_ws_coro()));
app.bind(80, 443);
app.run();
```

- App derives from or contains a single router
- Convenient for simple servers
- Less flexible for multi-port with different routes

### Idea 3: Router-per-Port (can be shared)

```cpp
http::router<http::route_params> my_router;
my_router.use(ws_upgrade(my_ws_coro()));
app.bind(80, my_router);
app.run();
```

- More explicit separation of concerns
- Same router can be shared across ports
- Different ports can have different routers (admin port, public port)

---

## API Design Considerations

### Multi-Port Binding

- Per-port worker counts: `app.bind(wolfssl, 443, 10000, my_www)`
- Shared router ownership across ports
- Use cases: HTTP->HTTPS redirect, admin port on private network

### WebSocket Integration

- `ws_upgrade()` middleware converts route to WebSocket
- Handler signature: `my_ws_session(http::route_params& rp, ws::stream& st) -> capy::task<route_result>`
- Pure WebSocket server: `app.use(ws_upgrade(my_ws_coro()))`

```cpp
capy::task<route_result>
my_ws_session(
    http::route_params& rp,
    ws::stream& st);

app.use("/ws", ws_upgrade(my_ws_coro()));
```

### Reactor Selection

```cpp
app.reactor(beast2::single_threaded);
app.reactor(beast2::single_threaded_no_mutex);
app.reactor(beast2::multi_threaded);
app.reactor(beast2::multi_context);
```

### Run Modes

- `run()` - blocking (scoped app)
- `run_async()` - non-blocking

### Builder Pattern

- Methods valid before `run()`: `use()`, `bind()`, `reactor()`
- Methods valid after `run()`: `stop()` (maybe just signal handling)
- Router flattened to immutable `flat_router` at launch
- Route state kept on connection, not router

```cpp
http::app_router r;
// ... configure ...
http::flat_router f(std::move(r)); // r is empty after move
```

---

## Technical Requirements

The HTTP server framework must support the following capabilities. Each requirement has an easy-path expectation (minimal user effort) and a control-path option (maximum flexibility).

### T1. Listening Port

Open and bind to a TCP port for accepting connections.

- **Easy path**: Single port number, framework handles socket options
- **Control path**: User can specify address, reuse options, backlog

### T2. Router Setup

Define URL patterns and associate them with handlers.

- **Easy path**: `app.get("/path", handler)` style registration
- **Control path**: Build `http::router`, transfer ownership to server
- Router compiles to `flat_router` for O(1) dispatch

### T3. Route Handlers

User-provided functions that process requests and produce responses.

- **Easy path**: Lambda with `http::context&` parameter
- **Control path**: Coroutine handlers, access to raw request/response
- Handlers must not know about execution model

### T4. TLS Configuration

Enable HTTPS with choice of SSL backend.

- **Easy path**: `app.tls("cert.pem", "key.pem")` — auto-detect backend
- **Control path**: Explicit `wolfssl_context` or `openssl_context`
- Backend choice is a link-time decision, not a code change (A7)

### T5. Certificate Management

Load certificates, private keys, and optionally CA chains.

- **Easy path**: File paths to PEM files
- **Control path**: In-memory buffers, custom certificate stores
- Support both file-based and programmatic configuration

### T6. Certificate Verification

Verify client certificates (mutual TLS) when required.

- **Easy path**: Off by default, simple enable flag
- **Control path**: Custom verification callbacks, certificate pinning
- Optional—most servers don't need this

### T7. Concurrency Model

Choose how connections are distributed across threads/cores.

- **Easy path**: Framework picks optimal strategy (thread pool)
- **Control path**: User selects model explicitly:
  - Single-threaded `io_context`
  - Multi-threaded `io_context` with strands per connection
  - N single-threaded `io_context` instances (io-per-core)
- Same handler code works with any model (B7)

### T8. Signal Handling

Graceful shutdown on SIGINT/SIGTERM.

- **Easy path**: Automatic—`listen()` handles signals internally
- **Control path**: User registers signal handlers, calls `server.stop()`
- Clean connection draining before exit

### T9. Logging

Configurable logging for requests, errors, and diagnostics.

- **Easy path**: Sensible defaults, logs to stderr
- **Control path**: Custom log sink, log levels, structured logging
- **Zero-cost disable**: When logging is off, no runtime overhead (no format strings evaluated, no function calls)

---

## Part A: Founding Principles (Non-Negotiable)

These are the core Beast2 principles. All designs must support them. Achieving some may require more elaborate user APIs—that's acceptable.

### A1. Clean Call Sites

No platform or implementation headers leak through to user code.

- User code never includes `<windows.h>`, `<sys/socket.h>`, etc.
- Implementation details stay in translation units
- Public headers are minimal and stable

### A2. Extensible Concepts

User-defined buffer sequences and executors work seamlessly. Stream wrappers require no templates.

- Concept-driven where it enables user extension
- `io_stream&` not `template<class Stream>`
- Users can plug in custom buffer types, executors

### A3. Zero-Alloc Steady State

No coroutine frame allocations in hot paths during normal operation.

- "Delete before dispatch" frame reuse
- Pre-allocated connection pools where appropriate
- Allocation happens at setup, not per-request

### A4. Minimal Template Exposure

Implementation details stay in the translation unit. Templates don't proliferate at API boundaries.

- Public API uses concrete or type-erased types
- Templates acceptable internally
- Fast compilation, stable ABI

### A5. Allocation-Free Type Erasure

User types preserved through abstraction boundaries without small-buffer optimization or malloc.

- Coroutine frame pays for type erasure
- No `std::function`-style heap allocation for handlers
- User types flow through without copies

### A6. Fine-Grained Execution Control

Strands, processor affinity, priority scheduling available when needed.

- Easy path hides this complexity
- Control path exposes full execution machinery
- User can pin connections to cores, prioritize traffic, etc.

### A7. ABI Stability by Design

SSL implementation, stream types can change without recompilation.

- `io_stream&` polymorphism, not templates
- Link against different SSL backends without rebuilding
- Runtime configuration, not compile-time macros

### A8. Implementation Hiding

No platform headers at call sites. No implementation structures in user code.

- Translation-unit isolation by default
- Pimpl where appropriate
- User code compiles fast and stays clean

---

## Part B: API Design Principles

These guide the user experience and API shape.

### B1. Layered Abstraction

High-level convenience APIs are implemented in terms of lower-level building blocks. Users can operate at any level of the stack.

- **Easy path**: Framework owns and manages everything
- **Control path**: User owns execution, framework provides components
- Each layer is a complete, usable API—not just internal implementation detail

### B2. Bulletproof Defaults

The easy path requires zero configuration and performs optimally out of the box.

- Sensible thread pool sizing
- Automatic signal handling and graceful shutdown
- Production-ready without tuning
- "Pit of success"—doing the simple thing is also the correct thing

### B3. Opt-in Complexity

Users accept more elaborate APIs when they need more control. We don't pollute the easy path with control-path concerns.

- Easy path: `app.listen(3000)` — one line, blocks, done
- Control path: User manages io_context, threads, shutdown
- Complexity is additive, not mandatory

### B4. Composable Building Blocks

Components are independent, reusable pieces that can be assembled in different configurations.

- Router, server, acceptor, connection—each is a standalone unit
- No hidden coupling between components
- User can replace or extend individual pieces
- "Lego architecture"

### B5. Type Boundary Simplicity

Components interact through concrete types or type-erased interfaces—not templates at API boundaries.

- HTTP server sees one executor type (type-erased or fixed)
- Router handlers have one signature
- No template proliferation at composition points
- Templates acceptable internally, not at user-facing seams

### B6. Ownership Semantics

Clear ownership transfer model. User builds, then hands off.

- Router is mutable during construction, immutable after transfer
- `std::move` signals ownership transfer
- Framework compiles/optimizes after receiving ownership
- No shared mutable state between user code and framework

### B7. Execution Agnosticism

HTTP/application logic is decoupled from execution strategy.

- Same handler code works with any reactor model
- Single-threaded, thread pool, io-per-core—transparent to handlers
- Execution strategy is a deployment decision, not a code decision

### B8. Expressive Brevity

API reads like intent. Minimal ceremony for common operations.

- Express.js inspiration: `app.get("/path", handler)`
- Method chaining where natural
- Named parameters over positional where clarity helps
- Main function should be glanceable

---

## Evaluation Framework

When assessing a design approach, score against both principle sets.

### Part A: Founding Principles (Must Pass)

These are pass/fail. A design that violates these is rejected.

- **A1. Clean Call Sites** — Do user files include platform headers? If yes, fail.
- **A2. Extensible Concepts** — Can users provide custom buffer types, executors, streams without templates at boundaries?
- **A3. Zero-Alloc Steady State** — Does the hot path allocate? Profile it.
- **A4. Minimal Template Exposure** — Count templates in public headers. Fewer is better.
- **A5. Allocation-Free Type Erasure** — Does type erasure use SBO or malloc? If yes, fail.
- **A6. Fine-Grained Execution Control** — Can users access strands, affinity, priorities?
- **A7. ABI Stability** — Can SSL backend change without recompiling user code?
- **A8. Implementation Hiding** — Are implementation types visible in public headers?

### Part B: API Design Principles (Score 1-5)

These are quality measures. Higher is better.

- **B1. Layered Abstraction** — Can users operate at multiple levels without rewriting?
- **B2. Bulletproof Defaults** — Does the simplest usage produce optimal behavior?
- **B3. Opt-in Complexity** — Is the easy path free of control-path noise?
- **B4. Composable Blocks** — Can components be used independently or swapped?
- **B5. Type Boundary Simplicity** — Are user-facing APIs template-free at composition points?
- **B6. Ownership Semantics** — Is it clear who owns what and when?
- **B7. Execution Agnosticism** — Can the same app code run on different reactor strategies?
- **B8. Expressive Brevity** — Is the common case short and readable?

### Scoring Template

```
Design: [Name]

Part A (Pass/Fail):
  A1 Clean Call Sites:        [ ] Pass  [ ] Fail
  A2 Extensible Concepts:     [ ] Pass  [ ] Fail
  A3 Zero-Alloc Steady State: [ ] Pass  [ ] Fail
  A4 Minimal Template:        [ ] Pass  [ ] Fail
  A5 Allocation-Free Erasure: [ ] Pass  [ ] Fail
  A6 Execution Control:       [ ] Pass  [ ] Fail
  A7 ABI Stability:           [ ] Pass  [ ] Fail
  A8 Implementation Hiding:   [ ] Pass  [ ] Fail

Part B (1-5):
  B1 Layered Abstraction:     ___
  B2 Bulletproof Defaults:    ___
  B3 Opt-in Complexity:       ___
  B4 Composable Blocks:       ___
  B5 Type Boundary:           ___
  B6 Ownership Semantics:     ___
  B7 Execution Agnosticism:   ___
  B8 Expressive Brevity:      ___
  
  Total: ___/40

Notes:
```
