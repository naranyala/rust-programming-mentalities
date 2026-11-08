# System Programming Mentalities

> 176 habits of mind for building systems that are correct, fast, secure, operable, and evolvable — from the memory hierarchy to the team that runs it.

Most “awesome” lists give you links. This one gives you **decision-making tools**. Each mentality has a mantra, a concrete practice, a smell that means it is missing, and a question that exposes whether you actually understand it.

The catalog is deliberately **full-stack**. Systems fail across layers, not inside them: a type error becomes a memory bug, a retry policy becomes a thundering herd, a naming choice becomes an outage. Coverage runs from silicon to schedules, because that is the distance a real incident travels.

---

## Contents

**Start:** [How to use](#how-to-use-this-catalog) · [Scope](#scope-what-comprehensive-means) · [Five themes](#five-cross-cutting-themes) · [Stack map](#the-stack-map) · [Start here](#start-here)

**Catalog:** [The complete index](#the-complete-index) — all 176 entries, grouped by domain

**Reference:** [Coverage matrix](#coverage-matrix) · [Self-audit](#the-self-audit) · [Code review lens](#the-code-review-lens) · [Anti-mentalities](#anti-mentalities) · [How to share](#how-to-share-this) · [Seven-week plan](#a-seven-week-relearning-plan) · [Further reading](#further-reading) · [Contributing](#contributing)

---

## How to use this catalog

| Mode | What it looks like | Cadence |
| --- | --- | --- |
| **Adopt** | Pick one mentality. Apply it to a real file or pull request. Write down what changed. | One per week |
| **Share** | Cite the number in review, tell the war story, run a 15-minute brown bag. | One per sprint |
| **Relearn** | Answer the relearn question from memory, then check the source and teach it. | One per month |

Three depth levels matter more than the number of entries:

- **Surface:** you recognize the name and can repeat the mantra.
- **Working:** you apply it in a real change and can show the before/after.
- **Expert:** you know when to break it, what it costs, and how to teach the tradeoff.

Three rules keep this from becoming a poster on the wall:

1. **One mentality at a time beats 176 at once.** Depth changes behavior; breadth changes nothing.
2. **A mentality without a practice is a slogan.** Every entry must alter a concrete decision.
3. **If you cannot explain it with an example, you have not adopted it.** You have only recognized it.

---

## Scope: what “comprehensive” means

This catalog is organized by the layers a real system crosses. The short version of the coverage:

- **Machine:** caches, pipelines, branch prediction, SIMD, DMA, accelerators, NUMA, coherence, representation, undefined behavior, arithmetic.
- **OS and runtime:** syscalls, processes, threads, fibers, scheduling, virtual memory, file descriptors, signals, startup and shutdown, dynamic linking, ABI, allocators.
- **Memory:** ownership, RAII, arenas, pools, copies, zero-copy, absence, fragmentation, leaks, OOM.
- **Correctness:** types, parsing, invariants, errors, boundaries, totality, determinism.
- **Concurrency:** message passing, single-writer, interleavings, lock ordering, atomics, memory ordering, lock-free, structured concurrency, cancellation.
- **Performance:** measurement, percentiles, scaling laws, queueing, benchmark realism, profiling, locality, batching.
- **Storage:** file abstraction, page cache, fsync, WAL, B-trees and LSM trees, checksums, backups, atomic rename.
- **Networking:** stream framing, flow and congestion control, head-of-line blocking, MTU, DNS, TLS, connection pools.
- **Distribution:** partial failure, fallacies, CAP and PACELC, consistency, replication, consensus, fencing, clocks, sharding, exactly-once, caching, sagas.
- **Reliability:** crash-only design, degradation, deadlines, retries, circuit breakers, load shedding, health checks, rollouts, SLOs, observability, postmortems, chaos, capacity, runbooks.
- **Security:** threat modeling, trust boundaries, least privilege, secure defaults, defense in depth, input and output encoding, memory safety, side channels, secrets, supply chain, sandboxing, DoS, crypto agility, audit logging.
- **Interfaces:** narrow waists, Hyrum’s law, Postel’s law, compatibility, semantic versioning, ABI stability, schema evolution, deprecation, mixed-version operation.
- **Build:** reproducibility, hermeticity, dependency pinning, SBOMs and signing, static and dynamic linking, cross-compilation, debug symbols, environment parity, caching.
- **Verification:** failure paths, boundaries, properties, differential and metamorphic tests, fuzzing, sanitizers, formal methods, deterministic simulation, load and soak tests, coverage, mutation testing.
- **Debugging:** mental models, evidence, reproduction, reduction, bisection, one-variable experiments, source reading, tool selection, whole-stack tracing, written records.
- **Design:** essential versus accidental complexity, simplicity, YAGNI, Gall’s law, deep modules, cohesion and coupling, stable dependencies, data-oriented design, Unix philosophy, reversibility.
- **Craft:** deletion, ADRs, naming, defaults, code review, refactoring, Chesterton’s fence, the boy scout rule, teaching, blameless learning, ownership and on-call.
- **Physical realities:** real-time deadlines, firmware and boot chains, power, thermal limits, and energy budgets.
- **Economic and human realities:** cost per request, data lifecycle and privacy, multi-region disaster recovery, incident command, and API fairness.

**What it is not:** a language tutorial, a replacement for specifications, or a claim that every possible system-programming topic is listed. It deliberately stops at adjacent disciplines such as chip design, legal compliance, organizational compensation, and domain-specific regulation. Use those where they apply; this catalog covers the engineering habits that connect them.

**Deliberate boundaries:** this is an engineering catalog, not a certification. It touches chip design through the performance model, compiler construction through ABI, undefined behavior, and representation, law through data lifecycle, and organizational design through ownership and on-call, but it does not replace specialists in those fields. For safety-critical domains such as medical, automotive, or aerospace, pair it with the relevant standard and a formal hazard analysis.

---

## Five cross-cutting themes

If you remember nothing else, remember that five forces recur in every layer:

1. **Time** — deadlines, clocks, latency, timeouts, leases, ordering, and the difference between wall time and logical time.
2. **Failure** — partial failure, crashes, corruption, overload, and the difference between “not working” and “working incorrectly.”
3. **Trust** — boundaries, authentication, least privilege, input validation, and the assumptions you make about other components.
4. **Cost** — CPU, memory, bandwidth, money, energy, complexity, and the attention of the people who operate the system.
5. **Change** — compatibility, evolution, deprecation, migration, and the fact that every interface is a promise about the future.

A mentality that ignores one of these is incomplete. A design that ignores all five is a demo.

---

## The stack map

Read **bottom-up** to go from the machine to the organization. Read **top-down** to trace a production failure back to the metal.

| Layer | Domain | Core question | Entries |
| --- | --- | --- | --- |
| 18 | Physical, economic, and human realities | What does it cost, and who operates it? | 169–176 |
| 17 | Craft, teams, and learning | Can the organization keep the system alive? | 158–168 |
| 16 | Design and complexity | What shape should the system have? | 148–157 |
| 15 | Debugging and diagnosis | How do we find the truth? | 140–147 |
| 14 | Testing and verification | How do we know it works? | 129–139 |
| 13 | Build, toolchain, and supply chain | Can we reproduce and trust the artifact? | 121–128 |
| 12 | Interfaces, ABI, and evolution | What have we promised, and for how long? | 113–120 |
| 11 | Security and safety | Who can do what to whom? | 99–112 |
| 10 | Reliability and operations | What happens when this fails? | 85–98 |
| 9 | Distributed systems | What happens when part of it is unreachable? | 73–84 |
| 8 | Networking and protocols | How do bytes become messages? | 65–72 |
| 7 | I/O, storage, and filesystems | What survives a crash? | 57–64 |
| 6 | Performance | What does the workload actually cost? | 49–56 |
| 5 | Concurrency and parallelism | What can interleave, and what breaks? | 41–48 |
| 4 | Correctness and types | Can the wrong thing even be expressed? | 33–40 |
| 3 | Memory and lifetime | Who owns this, and when does it end? | 25–32 |
| 2 | OS and runtime | What does the platform do on your behalf? | 17–24 |
| 1 | Machine and platform | What is the hardware actually doing? | 9–16 |
| 0 | Foundations | How do we think about any of this? | 1–8 |

---

## Start here

If 176 is too many, begin with these twelve. They cover the most common causes of real production pain.

**5** know the cost model · **25** ownership is the API · **27** bound everything · **33** illegal states · **35** invariants · **41** don't share memory · **43** any interleaving · **49** measure first · **59** fsync and durability · **73** partial failure · **87** deadlines · **99** threat modeling

Then add the layer you personally touch most: machine, storage, network, or team.

---

## The complete index

A compact map of all 176 entries, grouped by domain.

- **0 Foundations — How to Think:** 1 First principles over cargo cult · 2 The map is not the territory · 3 Follow the data · 4 Think in states · 5 Know the cost model · 6 Everything is a tradeoff · 7 Think in lifetimes · 8 Think in failure modes
- **1 Machine and Platform:** 9 The memory hierarchy is the algorithm · 10 The CPU is a pipeline · 11 Branch prediction is a design input · 12 SIMD, DMA, and accelerators · 13 Representation is a contract · 14 NUMA and cache coherence · 15 Undefined behavior is not theoretical · 16 Arithmetic has edges
- **2 OS and Runtime:** 17 The kernel is in your call path · 18 Processes, threads, and fibers · 19 The scheduler is not fair · 20 Virtual memory is a tool · 21 File descriptors and signals are resources · 22 Runtime lifecycle is part of the design · 23 Dynamic linking and ABI are contracts · 24 Allocators are policy
- **3 Memory and Lifetime:** 25 Ownership is the API · 26 Acquire and release in the same scope · 27 Bound everything · 28 Arenas and pools · 29 A copy is a design decision · 30 Zero-copy has a cost · 31 Absence needs a type · 32 Fragmentation, leaks, and OOM
- **4 Correctness and Types:** 33 Make illegal states unrepresentable · 34 Parse, don’t validate · 35 Invariants are the specification · 36 Errors are values · 37 Fail fast, fail closed · 38 Design for zero, one, and many · 39 Total functions and exhaustiveness · 40 Determinism and reproducibility
- **5 Concurrency and Parallelism:** 41 Don’t communicate by sharing memory · 42 One writer, many readers · 43 Assume any interleaving · 44 Lock ordering and granularity · 45 Never block the hot path · 46 Atomics and memory ordering · 47 Lock-free is not free · 48 Structured concurrency and cancellation
- **6 Performance:** 49 Measure before you believe · 50 Percentiles and tails · 51 Amdahl and Gustafson · 52 Little’s law and queueing · 53 Benchmark realism · 54 Profiling and flame graphs · 55 Locality beats cleverness · 56 Batch to amortize, stream to bound
- **7 I/O, Storage, and Filesystems:** 57 Everything is a file, until it isn’t · 58 Buffering and writeback · 59 fsync and durability · 60 Write-ahead logging · 61 B-trees and LSM trees · 62 Checksums and bit rot · 63 Backups are not replicas · 64 Atomic rename and file locking
- **8 Networking and Protocols:** 65 TCP is a stream, not messages · 66 Framing and message boundaries · 67 Flow control vs congestion control · 68 Head-of-line blocking · 69 MTU and fragmentation · 70 DNS is a dependency · 71 TLS is not optional · 72 Connection pools and keepalive
- **9 Distributed Systems:** 73 Partial failure · 74 Fallacies of distributed computing · 75 CAP and PACELC · 76 Consistency models · 77 Replication and quorums · 78 Consensus and leader election · 79 Split brain and fencing · 80 Clocks and logical time · 81 Sharding and hot keys · 82 Exactly-once is a lie; idempotency is not · 83 Caching and invalidation · 84 Outbox, sagas, and 2PC
- **10 Reliability and Operations:** 85 Crash-only software · 86 Design for graceful degradation · 87 Timeouts and deadlines · 88 Retries with jitter · 89 Circuit breakers and bulkheads · 90 Load shedding and admission control · 91 Health checks and readiness · 92 Rolling upgrades and rollback · 93 SLOs and error budgets · 94 Observability · 95 Blameless postmortems · 96 Chaos engineering and fault injection · 97 Capacity planning · 98 Runbooks and on-call
- **11 Security and Safety:** 99 Threat modeling · 100 Trust boundaries · 101 Least privilege · 102 Secure defaults · 103 Defense in depth · 104 Input validation and output encoding · 105 Memory safety · 106 Constant-time and side channels · 107 Secrets management · 108 Supply chain security · 109 Sandboxing and isolation · 110 Denial of service and resource exhaustion · 111 Crypto agility · 112 Audit logging
- **12 Interfaces, ABI, and Evolution:** 113 Narrow waist, thick ends · 114 Hyrum’s law · 115 Postel’s law, carefully · 116 Compatibility is a promise · 117 Semantic versioning and ABI stability · 118 Schema evolution · 119 Deprecation and migration · 120 Mixed-version operation
- **13 Build, Toolchain, and Supply Chain:** 121 Reproducible builds · 122 Hermetic builds · 123 Dependency pinning and lockfiles · 124 SBOM and signed artifacts · 125 Static vs dynamic linking · 126 Cross-compilation and toolchains · 127 Debug symbols and optimization levels · 128 Environment parity and build caching
- **14 Testing and Verification:** 129 Test the failure paths · 130 Boundary testing · 131 Property-based testing · 132 Differential and metamorphic testing · 133 Fuzzing · 134 Sanitizers and static analysis · 135 Formal methods and model checking · 136 Deterministic simulation · 137 Load, soak, and stress testing · 138 Coverage is not correctness · 139 Mutation testing
- **15 Debugging and Diagnosis:** 140 The bug is in your model · 141 Trust evidence, not memory · 142 Reproduce, reduce, bisect · 143 Change one thing at a time · 144 Read the source; the stack goes all the way down · 145 Use the right tool · 146 Understand the whole stack · 147 Write it down
- **16 Design and Complexity:** 148 Essential vs accidental complexity · 149 Simplicity is a destination · 150 YAGNI · 151 Gall’s law · 152 Deep modules and information hiding · 153 Cohesion and coupling · 154 Stable dependencies · 155 Data-oriented design · 156 Unix philosophy · 157 Reversibility and one-way doors
- **17 Craft, Teams, and Learning:** 158 Design for deletion · 159 ADRs and documentation · 160 Naming is design · 161 Defaults and configuration · 162 Code review as design review · 163 Refactoring and technical debt · 164 Chesterton’s fence · 165 The boy scout rule · 166 Teach it to learn it · 167 Blameless learning · 168 Ownership and on-call
- **18 Physical, Economic, and Human Realities:** 169 Real-time is a deadline contract · 170 Firmware, boot, and the trusted chain · 171 Power, thermal, and energy budgets · 172 Cost is a design constraint · 173 Data lifecycle and privacy · 174 Multi-region and disaster recovery · 175 Incident command and communication · 176 API limits, pagination, and fairness

---

## 0. Foundations — How to Think

*Before the machine, the model. These are the epistemic habits that keep everything else honest.*

### 1. First principles over cargo cult

> *If you cannot derive it, you do not understand it.*

- **Adopt:** Reconstruct the mechanism from constraints before copying a pattern. Ask what problem it solves, what it assumes, and what it costs.
- **Smell:** “Everyone uses it” as the whole argument; patterns imported without the forces that justify them.
- **Relearn:** Can I derive this choice from constraints, or only cite an authority?

### 2. The map is not the territory

> *Every model is a simplification; know what it hides.*

- **Adopt:** State the model’s assumptions and failure boundary. Use the simplest model that predicts the behavior you care about, and switch models when it stops predicting.
- **Smell:** Arguing about the diagram instead of the system; treating a benchmark, diagram, or type as reality.
- **Relearn:** What does my model deliberately ignore, and when will that omission matter?

### 3. Follow the data

> *Design around data flow, not nouns.*

- **Adopt:** Trace allocation, copies, transformations, ownership, and destruction. Let access patterns choose the structure.
- **Smell:** Layers of wrappers that each copy the payload; a clean object model with terrible locality.
- **Relearn:** Trace one byte end to end. How many copies and allocations does it survive?

### 4. Think in states

> *If it has states, make the state machine explicit.*

- **Adopt:** Enumerate states, events, transitions, and illegal transitions. Encode them in a type. Make startup, steady state, drain, and failure first-class.
- **Smell:** `is_connected && is_closing && is_ready` flags that can all be true; “impossible” transitions handled by hope.
- **Relearn:** What are the legal states, and what does each event do in each state?

### 5. Know the cost model

> *Every abstraction has a price; know the order of magnitude.*

- **Adopt:** Keep rough costs for register, cache, RAM, SSD, disk, and network. Annotate hot paths. Name the cost of every abstraction you add.
- **Smell:** Debating nanoseconds in a path that does a network round trip; ignoring a per-item syscall.
- **Relearn:** What are the three most expensive operations in my hot path, and roughly what do they cost?

### 6. Everything is a tradeoff

> *There is no free lunch; name what you are paying.*

- **Adopt:** For every design choice, write the benefit, the cost, and the condition under which you would reverse it. Optimize for the actual constraint, not the fashionable one.
- **Smell:** “Best practice” without context; silver bullets; decisions with no stated cost.
- **Relearn:** What did this choice make better, what did it make worse, and for whom?

### 7. Think in lifetimes

> *Every value is born, borrowed, moved, and destroyed.*

- **Adopt:** Track scope, ownership, borrowing, and destruction for every resource. Make lifetimes explicit in types, APIs, and documentation.
- **Smell:** “Who frees this?” comments; references that outlive their owner; cleanup on only the happy path.
- **Relearn:** When does this value begin, who may touch it, and when exactly does it end?

### 8. Think in failure modes

> *Design the failure path first; the happy path is the easy part.*

- **Adopt:** Enumerate ways each component can fail: crash, hang, corrupt, slow, partial, duplicate, reorder. Decide detection, containment, recovery, and user-visible behavior.
- **Smell:** Only the happy path is tested; failure is handled by “should never happen.”
- **Relearn:** What are the top three failure modes here, and what does the system do for each?

---

## 1. Machine and Platform

*The hardware is not a detail. It is the substrate that makes some designs cheap and others impossible.*

### 9. The memory hierarchy is the algorithm

> *Cache misses are the default cost.*

- **Adopt:** Treat a cache miss as the unit of pain. Keep hot fields together, cold fields apart; prefer contiguous arrays; avoid pointer chasing; measure with hardware counters.
- **Smell:** Linked-list or tree walks in the inner loop; false sharing; per-object allocation in a hot path.
- **Relearn:** If every access were a cache miss, how slow would this loop be? Which accesses actually are?

### 10. The CPU is a pipeline

> *The CPU executes instructions in parallel, not one at a time.*

- **Adopt:** Write code the pipeline can predict and overlap: simple loops, independent work, few branches, no aliasing surprises. Measure IPC and stalls.
- **Smell:** Assuming source order equals execution order; long dependency chains; branchy inner loops.
- **Relearn:** Where does this loop stall, and what would the pipeline like instead?

### 11. Branch prediction is a design input

> *Predictable branches are cheap; unpredictable ones are not.*

- **Adopt:** Keep hot branches biased and data sorted when it helps. Use branchless arithmetic for unpredictable conditions only after measuring. Prefer layouts that reduce branching.
- **Smell:** Random branchy code in a hot loop; assuming `if` is free; sorting data only for aesthetics.
- **Relearn:** Which branch in this hot path is unpredictable, and what does it cost?

### 12. SIMD, DMA, and accelerators

> *Move work to the unit that does it best.*

- **Adopt:** Vectorize regular, data-parallel work; use DMA or RDMA to avoid CPU copies; offload to GPU or specialized hardware when data movement pays for itself.
- **Smell:** Scalar loops over large arrays; CPU copying data a DMA engine could move; GPU offload with more transfer than compute.
- **Relearn:** Is this work regular enough to vectorize, and does the data movement cost less than the compute it saves?

### 13. Representation is a contract

> *Alignment, padding, endianness, and layout are part of the interface.*

- **Adopt:** Define wire and file formats byte by byte. Use fixed-width types and explicit endianness. Know struct padding and ABI layout. Serialize; do not dump memory.
- **Smell:** `memcpy` of a struct to disk or network; assuming `int` is 32 bits; host-endian protocols.
- **Relearn:** Can two different machines agree on the exact bytes this produces?

### 14. NUMA and cache coherence

> *Memory is not uniform, and cores talk behind your back.*

- **Adopt:** Keep data near the thread that uses it. Avoid false sharing; align hot per-thread data; measure remote accesses. Prefer partitioning over sharing.
- **Smell:** A global counter hammered by all cores; one cache line holding unrelated hot fields; ignoring NUMA on large machines.
- **Relearn:** Which core touches this line, and how far does the data travel?

### 15. Undefined behavior is not theoretical

> *The compiler is allowed to assume UB never happens.*

- **Adopt:** Know the UB rules of your language: aliasing, overflow, null dereference, out-of-bounds, uninitialized reads. Use sanitizers and careful types. Never “fix” UB by lowering optimization.
- **Smell:** Signed overflow used as a wrap-around trick; strict-aliasing violations; reading uninitialized memory; `-fwrapv` as a substitute for design.
- **Relearn:** Which line in this function is undefined if an assumption fails, and what can the optimizer do with that?

### 16. Arithmetic has edges

> *Integers wrap, floats round, and both have special values.*

- **Adopt:** Check overflow at boundaries; use wider or checked types. Respect floating-point precision, NaN, infinity, and signed zero. Never compare floats for exact equality without a reason.
- **Smell:** `a + b` on untrusted sizes; `x == 0.1`; assuming associativity in floating point; unsigned underflow.
- **Relearn:** What happens at zero, the maximum, overflow, NaN, and negative zero?

---

## 2. OS and Runtime

*Between your code and the hardware sits an operating system with its own costs, policies, and failure modes.*

### 17. The kernel is in your call path

> *Every syscall is a boundary crossing, not a function call.*

- **Adopt:** Count syscalls, page faults, context switches, and copies in hot paths. Batch, buffer, or map when fixed costs dominate. Know what the kernel does on your behalf.
- **Smell:** One `read` or `write` per byte; assuming a syscall is free; ignoring page faults in a latency budget.
- **Relearn:** What does the kernel do for this call, and how many times am I paying for it?

### 18. Processes, threads, and fibers

> *Choose the concurrency unit that matches the isolation and scheduling you need.*

- **Adopt:** Processes for isolation and fault containment; threads for shared-memory parallelism; fibers or async tasks for many lightweight waits. Know the cost of each switch.
- **Smell:** One process per request with shared state; threads for I/O-bound work that should be async; fibers doing CPU-bound work.
- **Relearn:** What does this unit isolate, what does it share, and what does it cost to switch?

### 19. The scheduler is not fair

> *The scheduler optimizes throughput, not your latency promise.*

- **Adopt:** Pin latency-critical work when justified; avoid oversubscription; measure run-queue delay; use priorities carefully and know about priority inversion.
- **Smell:** Assuming a thread runs when you wake it; CPU-bound work next to latency-critical work; priorities without analysis.
- **Relearn:** What is this thread waiting for, and who decides when it runs?

### 20. Virtual memory is a tool

> *Pages are a mapping, not a guarantee.*

- **Adopt:** Use `mmap` for large files, shared memory, and lazy allocation; understand page faults, TLB pressure, huge pages, and overcommit. Measure RSS, not just virtual size.
- **Smell:** Assuming `malloc` returns physical memory; ignoring page-fault cost; using `mmap` for tiny files.
- **Relearn:** What is actually resident, what is mapped, and when does touching it fault?

### 21. File descriptors and signals are resources

> *File descriptors leak, and signals are control flow.*

- **Adopt:** Treat every fd as an owned resource with a close on every path. Use `O_CLOEXEC`. Keep signal handlers tiny and async-signal-safe; communicate via self-pipe or `eventfd`.
- **Smell:** Fd leaks under load; `printf` in a signal handler; using signals for ordinary application logic.
- **Relearn:** What is the fd limit here, and what happens when a signal arrives mid-operation?

### 22. Runtime lifecycle is part of the design

> *Startup, warm-up, shutdown, and restart are features.*

- **Adopt:** Measure cold start and warm-up. Make initialization idempotent and observable. Design graceful drain, cancellation, and restart. Test the lifecycle, not just steady state.
- **Smell:** Only steady state is benchmarked; shutdown loses data; restart requires manual steps.
- **Relearn:** What happens from process start to first request, and from last request to exit?

### 23. Dynamic linking and ABI are contracts

> *The ABI is a promise made by the compiler, loader, and operating system.*

- **Adopt:** Understand calling conventions, symbol visibility, versioned symbols, and ABI compatibility. Prefer stable interfaces; test with old and new binaries.
- **Smell:** Changing a public struct layout; relying on undefined symbol resolution order; mixing incompatible runtimes.
- **Relearn:** Can a binary built last year run against this library today?

### 24. Allocators are policy

> *Every allocation is a policy decision about latency, fragmentation, and locality.*

- **Adopt:** Know your allocator’s behavior under contention. Use arenas or pools for uniform lifetimes; avoid allocation in hot paths; measure fragmentation and RSS.
- **Smell:** `malloc` in an inner loop; assuming all allocators behave the same; ignoring allocator-induced pauses.
- **Relearn:** Who allocates, how often, and what does the allocator do under contention?

---

## 3. Memory and Lifetime

*Memory is the resource that turns small mistakes into security incidents and long outages.*

### 25. Ownership is the API

> *Who owns this, and when does it end?*

- **Adopt:** Document allocation, mutation rights, borrowing, and destruction. Encode ownership in types when possible. Use arenas or pools for uniform lifetimes.
- **Smell:** “Does the caller own the returned pointer?” comments; double free; use-after-free; aliasing that mutates under a reader.
- **Relearn:** Who owns this memory, when does it end, and who may mutate it?

### 26. Acquire and release in the same scope

> *Every acquisition has exactly one release on every path.*

- **Adopt:** Wrap resources in guards or RAII. Clean up on success, error, panic, and cancellation. Test with leak detectors and failure injection.
- **Smell:** Manual frees on the happy path; `goto cleanup` spaghetti; a lock released only on success.
- **Relearn:** Does every acquisition have exactly one release on every path, including the ones you hope never happen?

### 27. Bound everything

> *Bound everything, including time.*

- **Adopt:** Put a maximum on queues, buffers, retries, recursion, connections, and duration. When the bound is hit, choose a policy: reject, shed, block, or spill.
- **Smell:** Unbounded channels; retry forever; recursion on untrusted input; “memory is cheap.”
- **Relearn:** What is the maximum of this resource, what happens at the maximum, and who decides?

### 28. Arenas and pools

> *Match allocation strategy to lifetime.*

- **Adopt:** Use arenas for phase-scoped work and pools for fixed-size objects. Bulk-free at once. Keep ownership simple; avoid long-lived references into short-lived arenas.
- **Smell:** Per-object allocation for a request; freeing one by one when a whole phase ends; pools that never return memory.
- **Relearn:** What is the lifetime of this object, and does a bulk strategy fit it?

### 29. A copy is a design decision

> *Every copy is a tradeoff, not a reflex.*

- **Adopt:** Count copies. Move instead of copy, borrow or slice instead of clone, and copy deliberately for isolation or simplicity. Measure before adding zero-copy complexity.
- **Smell:** Accidental deep copies in a loop; aliasing bugs caused by avoiding a cheap copy; copy-on-write without understanding the write rate.
- **Relearn:** Is this copy buying isolation, or just burning bandwidth? What breaks if I remove it?

### 30. Zero-copy has a cost

> *Zero-copy moves the cost; it does not delete it.*

- **Adopt:** Use zero-copy only when data movement dominates and lifetimes are safe. Account for pinning, page faults, cache pressure, and API complexity.
- **Smell:** Zero-copy at all costs; subtle lifetime bugs from borrowed buffers; more code than the copy saved.
- **Relearn:** What did zero-copy remove, what did it add, and which is larger here?

### 31. Absence needs a type

> *Zero, empty, missing, and unknown are four different things.*

- **Adopt:** Use `Option`, `Result`, or an explicit nullable type. Distinguish empty from absent. Avoid sentinel values unless an ABI forces you into them.
- **Smell:** `0` or `-1` meaning “unset”; `""` meaning “missing”; null checks scattered across every caller.
- **Relearn:** Does this value mean zero, empty, missing, or unknown? Can the type say so?

### 32. Fragmentation, leaks, and OOM

> *Memory failures are slow, then sudden.*

- **Adopt:** Measure RSS, fragmentation, and allocation rate. Decide OOM behavior explicitly: fail, shed, restart, or degrade. Test under memory pressure.
- **Smell:** Long-lived processes that grow forever; OOM handled by the kernel; no memory budget.
- **Relearn:** What is the memory budget, what grows, and what happens at the limit?

---

## 4. Correctness and Types

*The cheapest bug is the one the type system refuses to compile.*

### 33. Make illegal states unrepresentable

> *The best validation is a type that cannot be wrong.*

- **Adopt:** Parse, don’t validate. Use sum types, enums, and newtypes. Validate once at the boundary, then pass a value that carries its own proof. Keep constructors honest.
- **Smell:** A struct with `kind`, `payload`, and `is_error` that can disagree; validation repeated in every caller.
- **Relearn:** Which combination of fields is nonsense? Why does the type permit it?

### 34. Parse, don’t validate

> *Turn untrusted input into a trustworthy type at the boundary.*

- **Adopt:** Write one parser that returns a validated type or a precise error. Let the rest of the program accept only the parsed type. Never re-check what the type already guarantees.
- **Smell:** Validation functions returning booleans; raw strings passed deep into the system; checks repeated at every layer.
- **Relearn:** Where is the boundary, and what type proves the input is valid on the other side?

### 35. Invariants are the specification

> *An invariant without an owner is a rumor.*

- **Adopt:** Write the invariant down, give it an owner, assert it in debug builds and tests, and check it at trust boundaries. Property-based tests are invariant tests in disguise.
- **Smell:** “Must be sorted” in a comment; a data structure that is only sometimes consistent; silent repair that hides corruption.
- **Relearn:** What must always be true here, who guarantees it, and what detects when it stops being true?

### 36. Errors are values

> *Failure is part of the interface, not an interruption of it.*

- **Adopt:** Make failure part of the type or signature. Distinguish expected errors from bugs. Preserve context and cause. Decide policy at the layer that has enough information.
- **Smell:** Catch-all handlers; ignored return codes; panics for user input; errors logged and swallowed.
- **Relearn:** List every way this function can fail. What should the caller do about each one?

### 37. Fail fast, fail closed

> *Stop on corruption; deny by default.*

- **Adopt:** Validate at trust boundaries. Abort on impossible states rather than limping forward. Make security defaults deny, not allow. Corrupt state should stop, not spread.
- **Smell:** Continuing after a failed write; permissive defaults; “best effort” on integrity checks.
- **Relearn:** If this assumption is false, is it safer to stop or to continue? Which one does the code do?

### 38. Design for zero, one, and many

> *The boundary cases are the specification.*

- **Adopt:** Test empty, singleton, duplicate, maximum, overflow, and partial input. Use a `NonEmpty` type where the invariant matters. Check integer arithmetic at the edges.
- **Smell:** `items[0]` without a check; loops that assume at least one element; `count - 1` on an unsigned integer.
- **Relearn:** What happens with zero, one, a million, and duplicates?

### 39. Total functions and exhaustiveness

> *Every input has an answer; every case has a branch.*

- **Adopt:** Make functions total where possible. Use exhaustive pattern matching and let the compiler warn on missing cases. Avoid catch-all `else` that hides new variants.
- **Smell:** `default:` swallowing a new enum variant; partial functions that panic on valid input; silent fallthrough.
- **Relearn:** What happens for every possible input, and will the compiler tell me if I forget one?

### 40. Determinism and reproducibility

> *If you cannot reproduce it, you cannot fix it.*

- **Adopt:** Control time, randomness, scheduling, and environment in tests. Make builds reproducible. Log inputs and versions. Prefer deterministic algorithms where cost allows.
- **Smell:** Tests that pass sometimes; “works on my machine”; unseeded randomness in production logic.
- **Relearn:** Can I make this run produce the same result twice, and if not, what is uncontrolled?

---

## 5. Concurrency and Parallelism

*Concurrency is where correct code becomes incorrect without changing a line.*

### 41. Don’t communicate by sharing memory

> *Send data; don’t share it.*

- **Adopt:** Move ownership across channels or queues. Share immutable snapshots. Keep mutable shared state small and explicit. If you must share, define the protocol, not just the lock.
- **Smell:** Global mutable state; a lock around every access; several threads mutating the same map.
- **Relearn:** Can I send the data instead of sharing it? If not, what is the smallest thing that must be shared?

### 42. One writer, many readers

> *One writer beats many careful readers.*

- **Adopt:** Confine mutation to one thread, task, or actor. Shard state by key. Use immutable reads. Single-writer is easier to reason about than “mostly read.”
- **Smell:** Multiple writers to the same field or cache line; ad-hoc locks in different modules; “it’s only a counter.”
- **Relearn:** Who is allowed to write this? Can I make that exactly one place?

### 43. Assume any interleaving

> *Assume the worst schedule, not the likely one.*

- **Adopt:** Use lock ordering, atomics with documented memory order, and tools: thread sanitizers, model checkers, stress tests. Avoid check-then-act. Write down the invariant each lock protects.
- **Smell:** Sleeps to “fix” races; double-checked locking without the right barriers; TOCTOU on files or permissions.
- **Relearn:** What is the worst interleaving, which invariant breaks, and how would I detect it?

### 44. Lock ordering and granularity

> *Locks are a protocol; define the order and the scope.*

- **Adopt:** Establish a global lock order and document it. Keep critical sections small. Prefer fine-grained locks only when contention proves it; coarse locks are easier to reason about.
- **Smell:** Ad-hoc lock acquisition; nested locks without an order; holding a lock across I/O; lock convoy.
- **Relearn:** What is the lock order, what does each lock protect, and what is held across this call?

### 45. Never block the hot path

> *Never block the hot path.*

- **Adopt:** Keep critical sections tiny. No I/O, logging, allocation, or unbounded work under a lock or inside an event loop. Offload, queue, or preallocate.
- **Smell:** Logging while holding a lock; synchronous DNS in a request handler; garbage collection or allocation in a latency-critical loop.
- **Relearn:** What is this thread unable to do while it waits here, and who else is waiting on it?

### 46. Atomics and memory ordering

> *Atomics give you ordering, not just indivisibility.*

- **Adopt:** Choose the weakest correct memory order; document why. Use acquire/release for publication, relaxed for counters, and sequentially consistent only when you need a global order. Test with sanitizers.
- **Smell:** `relaxed` everywhere; assuming atomics make a whole struct consistent; hand-rolled fences without a model.
- **Relearn:** What ordering does this operation need, and what breaks if I weaken it?

### 47. Lock-free is not free

> *Lock-free trades blocking for complexity and memory reclamation.*

- **Adopt:** Use lock-free only when measured contention justifies it. Account for ABA, memory reclamation, progress guarantees, and verification cost. Prefer proven libraries.
- **Smell:** Hand-rolled lock-free queues; “faster than a mutex” without benchmarks; ignoring reclamation.
- **Relearn:** What progress guarantee do I actually need, and who reclaims the memory?

### 48. Structured concurrency and cancellation

> *Every task has an owner, a scope, and a cancellation path.*

- **Adopt:** Tie task lifetimes to a scope; propagate cancellation and deadlines; join before returning. Avoid detached fire-and-forget work unless it is truly independent.
- **Smell:** Orphaned tasks; cancellation ignored; shutdown that waits forever; async code that leaks.
- **Relearn:** Who owns this task, what cancels it, and what happens if it never finishes?

---

## 6. Performance

*Performance is a property of the whole system under a real workload, not a property of a line of code.*

### 49. Measure before you believe

> *Measure before you believe — including your own intuition.*

- **Adopt:** Benchmark realistic workloads. Report p50, p99, and p999 with variance. Use profilers and production counters. Watch for coordinated omission in latency tests.
- **Smell:** Optimizing by intuition; microbenchmarks that do not resemble production; a single average hiding a bad tail.
- **Relearn:** What exactly did I measure, on what workload, and how much does it vary?

### 50. Percentiles and tails

> *The average is a lie; the tail is the user experience.*

- **Adopt:** Track p50, p90, p99, p999, and max. Segment by operation and tenant. Watch for coordinated omission. Optimize the tail when it dominates user pain.
- **Smell:** Reporting only averages; hiding a 2-second p99 behind a 20 ms mean; ignoring slow requests.
- **Relearn:** What does the worst 1% experience, and is that acceptable?

### 51. Amdahl and Gustafson

> *Speedup is bounded by the serial part.*

- **Adopt:** Find the serial fraction and the critical path. Fix the constraint, then re-measure. Use scaling laws to predict, not to excuse.
- **Smell:** Parallelizing a serial path; optimizing code that is 1% of runtime; assuming linear scaling.
- **Relearn:** If I made this part infinitely fast, how much faster would the system be?

### 52. Little’s law and queueing

> *Latency, throughput, and concurrency are one equation.*

- **Adopt:** Use Little’s law (`L = λW`) to reason about queues. Watch utilization: latency explodes near saturation. Add capacity or shed load before the knee.
- **Smell:** Adding workers without fixing queueing; running at 95% utilization and expecting stable latency; ignoring queue depth.
- **Relearn:** At this arrival rate, how many requests are in the system, and how close am I to saturation?

### 53. Benchmark realism

> *A benchmark that does not resemble the workload is a random number generator.*

- **Adopt:** Use real data distributions, real concurrency, real I/O, and real failures. Warm up caches and JITs. Pin the environment. Report variance and methodology.
- **Smell:** Microbenchmarks with no I/O; synthetic uniform keys; benchmarking a warm cache only; no confidence intervals.
- **Relearn:** What does this benchmark omit from production, and could that omission reverse the conclusion?

### 54. Profiling and flame graphs

> *The profiler sees what the code does, not what you think it does.*

- **Adopt:** Use sampling profilers for CPU, off-CPU, memory, and locks. Read flame graphs by width, not height. Correlate with traces and production metrics.
- **Smell:** Guessing the hotspot; profiling only in a debug build; ignoring off-CPU time and lock waits.
- **Relearn:** Where does the time actually go, including waiting, and how do I know?

### 55. Locality beats cleverness

> *Locality beats cleverness.*

- **Adopt:** Design for cache lines and NUMA. Pack hot data, separate cold data, avoid pointer chasing, align shared structures, and measure misses instead of guessing.
- **Smell:** Clever algorithms with terrible constants; per-object allocations in a hot loop; false sharing between cores.
- **Relearn:** How many cache lines does this touch, and are any of them shared across cores?

### 56. Batch to amortize, stream to bound

> *Batch to amortize; stream to bound.*

- **Adopt:** Amortize fixed costs: batch syscalls, messages, and writes. Stream large data instead of materializing it. Choose buffer sizes deliberately.
- **Smell:** One syscall or message per item; loading a huge file into memory to process it; tiny writes in a loop.
- **Relearn:** Am I paying a fixed cost per item? Can I amortize it, or must I bound memory instead?

---

## 7. I/O, Storage, and Filesystems

*Storage is where durability, latency, and failure meet. The filesystem is a database with sharp edges.*

### 57. Everything is a file, until it isn’t

> *The file abstraction is powerful, but not universal.*

- **Adopt:** Use files and descriptors for uniform I/O, but respect the differences: sockets, pipes, devices, directories, and memory maps all have different semantics. Check capabilities, not assumptions.
- **Smell:** Treating a socket like a seekable file; assuming all descriptors support the same operations; ignoring `EINTR` and partial I/O.
- **Relearn:** What does this descriptor actually support, and what happens on partial or interrupted I/O?

### 58. Buffering and writeback

> *The page cache is a cache; writes are not durable until you say so.*

- **Adopt:** Understand user-space buffers, page cache, dirty writeback, and `fsync`. Batch small writes. Measure cache hit behavior. Do not assume `write` means disk.
- **Smell:** Assuming `write()` persisted data; unbounded dirty pages; mixing buffered and direct I/O without understanding.
- **Relearn:** Where is this byte now, and what makes it durable?

### 59. fsync and durability

> *Durability is a spectrum, and `fsync` is a promise with a cost.*

- **Adopt:** Know what `fsync`, `fdatasync`, and directory fsync guarantee on your platform. Order writes and barriers correctly. Measure the latency cost. Test power loss.
- **Smell:** Calling `fsync` everywhere “for safety” without ordering; never calling it for critical data; assuming a successful write survived a crash.
- **Relearn:** After a crash at this exact point, what data is guaranteed to be there?

### 60. Write-ahead logging

> *Log the intent before you change the state.*

- **Adopt:** Append the change to a durable log, then apply it. Use sequence numbers, checksums, and idempotent replay. Truncate only after the state is safely persisted.
- **Smell:** In-place updates without a log; replay that is not idempotent; logs without checksums.
- **Relearn:** If I crash mid-update, how does replay know what happened and what to redo?

### 61. B-trees and LSM trees

> *Storage structures encode a read/write/space tradeoff.*

- **Adopt:** B-trees for read-optimized, update-in-place workloads; LSM trees for write-heavy workloads with compaction. Understand amplification: read, write, and space.
- **Smell:** Choosing a storage engine by reputation; ignoring compaction stalls; assuming one structure fits all workloads.
- **Relearn:** What is the read, write, and space amplification of this design, and which one hurts?

### 62. Checksums and bit rot

> *Disks lie; verify the bytes.*

- **Adopt:** Checksum data at rest and in transit. Verify on read. Use end-to-end integrity, not just device ECC. Scrub periodically. Fail loudly on mismatch.
- **Smell:** Trusting a successful read; checksums only at the device layer; silent corruption repair.
- **Relearn:** How would I know if this byte changed, and what happens when it does?

### 63. Backups are not replicas

> *A replica copies mistakes; a backup lets you go back.*

- **Adopt:** Keep independent backups, test restores, define RPO and RTO, and protect against deletion and ransomware. A backup you have never restored is a hypothesis.
- **Smell:** Replication called a backup; untested restores; backups in the same failure domain.
- **Relearn:** When did I last restore from this backup, and how long did it take?

### 64. Atomic rename and file locking

> *Filesystems give you a few atomic primitives; build on them.*

- **Adopt:** Use write-temp-then-rename for atomic replacement. Use `O_EXCL` for creation. Understand advisory versus mandatory locking and its portability. Fsync the directory when needed.
- **Smell:** Writing a config file in place; assuming `rename` works across filesystems; relying on advisory locks for correctness.
- **Relearn:** What is atomic here, and what happens if the process dies halfway through?

---

## 8. Networking and Protocols

*The network is a shared, unreliable, adversarial medium. Most distributed bugs start here.*

### 65. TCP is a stream, not messages

> *TCP gives you bytes, not boundaries.*

- **Adopt:** Frame messages explicitly: length prefixes, delimiters, or fixed-size records. Handle partial reads and writes. Never assume one `read` equals one message.
- **Smell:** Parsing a message per `recv`; assuming writes are atomic; ignoring short writes.
- **Relearn:** Where is the message boundary, and what happens if it arrives in three pieces?

### 66. Framing and message boundaries

> *Every protocol needs a frame.*

- **Adopt:** Define length, type, version, and checksum. Set maximum frame sizes. Reject malformed frames early. Make framing independent of transport where possible.
- **Smell:** Newline-delimited JSON with no size limit; ambiguous delimiters; unbounded frames.
- **Relearn:** Can a receiver always tell where this message ends, even under fragmentation and corruption?

### 67. Flow control vs congestion control

> *Flow control protects the receiver; congestion control protects the network.*

- **Adopt:** Understand receive windows, send buffers, and congestion algorithms. Apply backpressure at the application layer. Do not confuse “the socket accepted it” with “the peer processed it.”
- **Smell:** Unbounded application queues; ignoring TCP backpressure; blaming the network for an application-level queue.
- **Relearn:** Who is being protected by this mechanism, and where does the pressure go?

### 68. Head-of-line blocking

> *One slow message can block everything behind it.*

- **Adopt:** Multiplex independent streams, shard connections, or use protocols without head-of-line blocking (HTTP/2, HTTP/3, QUIC). Isolate slow consumers.
- **Smell:** One giant connection for all requests; a slow response blocking a fast one; per-connection serial processing.
- **Relearn:** What is queued behind this slow item, and can I isolate it?

### 69. MTU and fragmentation

> *Packets have a maximum size; respect it.*

- **Adopt:** Know the path MTU. Avoid IP fragmentation. Size application frames to fit. Handle `EMSGSIZE` and path-MTU black holes.
- **Smell:** Sending huge UDP datagrams; assuming local MTU equals path MTU; silent drops behind a firewall.
- **Relearn:** What is the largest packet that survives this path, and what happens if it doesn’t?

### 70. DNS is a dependency

> *DNS is a distributed system with caching, failures, and latency.*

- **Adopt:** Cache with TTL awareness, use multiple resolvers, set timeouts, and avoid synchronous DNS in hot paths. Monitor resolution latency and failures. Plan for stale records.
- **Smell:** Assuming DNS is instant and reliable; no fallback; hard-coded IPs as the only mitigation.
- **Relearn:** What happens to this request if DNS is slow, wrong, or down?

### 71. TLS is not optional

> *Encrypt in transit, authenticate the peer, and rotate keys.*

- **Adopt:** Use modern TLS defaults, verify certificates and hostnames, set timeouts, and support rotation. Do not disable verification to “make it work.”
- **Smell:** `InsecureSkipVerify`; pinned certificates with no rotation plan; plaintext internal traffic assumed safe.
- **Relearn:** Who is the peer, how do I know, and what happens when the certificate expires?

### 72. Connection pools and keepalive

> *Connections are expensive; stale connections are worse.*

- **Adopt:** Pool with limits, health checks, idle timeouts, and jittered reconnect. Understand keepalive at TCP and application layers. Avoid one connection per request.
- **Smell:** Unbounded pools; no idle eviction; reconnect storms; assuming a pooled connection is alive.
- **Relearn:** How many connections, how long do they live, and what happens when one is half-dead?

---

## 9. Distributed Systems

*A distributed system is a system where a local failure becomes a global mystery. These are the laws of that world.*

### 73. Partial failure

> *Assume partial failure, because you will get it.*

- **Adopt:** Treat every remote call as fallible and slow. Add timeouts, fallbacks, bulkheads, and degradation. Test with faults, not just happy paths.
- **Smell:** Assuming RPC succeeds; all-or-nothing transactions across unreliable boundaries; one slow dependency taking down everything.
- **Relearn:** Which dependency can fail without taking me down? What degrades, and what stops?

### 74. Fallacies of distributed computing

> *The network is not reliable, latency is not zero, and topology changes.*

- **Adopt:** Revisit the fallacies explicitly: reliability, latency, bandwidth, security, topology, administration, transport cost, homogeneity. Design for each.
- **Smell:** Treating a remote call like a local call; assuming bandwidth is free; ignoring partial deployment.
- **Relearn:** Which fallacy is this design assuming away, and what happens when it returns?

### 75. CAP and PACELC

> *During a partition you choose; the rest of the time you also choose.*

- **Adopt:** Use CAP to frame partition behavior and PACELC to frame latency versus consistency tradeoffs in normal operation. Choose per operation, not per system.
- **Smell:** “We are CP” without defining the operation; ignoring latency during normal operation; consistency claims without a model.
- **Relearn:** What does this operation do during a partition, and what does it trade for latency when there is none?

### 76. Consistency models

> *Consistency is a contract about what a reader can observe.*

- **Adopt:** Name the model: linearizable, sequential, causal, read-your-writes, monotonic reads, eventual. Match it to user expectations and test it.
- **Smell:** “Eventually consistent” as a hand wave; a UI that assumes read-your-writes without providing it; mixing models in one flow.
- **Relearn:** What is the strongest anomaly a user can observe here, and is that acceptable?

### 77. Replication and quorums

> *Replication buys durability and availability, and costs coordination.*

- **Adopt:** Understand leader-follower, multi-leader, and quorum replication. Choose read and write quorums deliberately. Track replication lag and handle reads from lagging replicas.
- **Smell:** Assuming replicas are in sync; quorum math without failure models; ignoring lag in user-facing reads.
- **Relearn:** How many replicas must acknowledge, and what can a reader see before they do?

### 78. Consensus and leader election

> *Consensus is expensive; use it for the few things that need it.*

- **Adopt:** Use proven implementations (Raft, Paxos, Zab). Understand terms, quorums, and commit rules. Keep the consensus state small. Do not build consensus for every write.
- **Smell:** Hand-rolled leader election; consensus for high-throughput data; ignoring the cost of a round trip.
- **Relearn:** What exactly needs consensus here, and what can be decided locally instead?

### 79. Split brain and fencing

> *A leader that cannot be stopped must be fenced.*

- **Adopt:** Use leases, epochs, fencing tokens, and quorum checks. Make every write conditional on the current leader’s token. Assume the old leader may still be alive.
- **Smell:** Two leaders writing; leases without fencing; “the old leader will notice” as the safety mechanism.
- **Relearn:** What stops the old leader from writing after a new one is elected?

### 80. Clocks and logical time

> *Wall clocks jump; logical clocks order.*

- **Adopt:** Use monotonic clocks for durations, wall clocks for timestamps with skew awareness. Use logical clocks, vector clocks, or version vectors for causality. Never order events by wall time alone.
- **Smell:** Comparing timestamps across machines to decide order; assuming NTP is exact; using wall-clock timeouts.
- **Relearn:** What orders these events, and what happens if the clock jumps backward?

### 81. Sharding and hot keys

> *Distribution is only as good as its worst key.*

- **Adopt:** Choose a shard key with uniform access and stable growth. Use consistent hashing or range partitioning deliberately. Detect and split hot keys. Plan rebalancing.
- **Smell:** Hashing by tenant with one giant tenant; range shards with sequential writes; no hot-key monitoring.
- **Relearn:** What is the hottest key, and what happens when it grows 100x?

### 82. Exactly-once is a lie; idempotency is not

> *Exactly-once delivery is a fiction; exactly-once effect is a design.*

- **Adopt:** Assume at-least-once delivery. Make handlers idempotent with keys, deduplication, or conditional writes. Store request IDs with the effect.
- **Smell:** “Exactly once” claimed by a queue; duplicate charges on retry; dedup tables with no retention policy.
- **Relearn:** If this message is delivered twice, what prevents a second effect?

### 83. Caching and invalidation

> *A cache is a consistency decision with a latency benefit.*

- **Adopt:** Define staleness bounds, invalidation strategy, and fallback behavior. Use TTLs, versioned keys, and write-through or write-behind deliberately. Protect against stampedes.
- **Smell:** Caching without an invalidation story; unbounded TTLs; cache as the source of truth.
- **Relearn:** How stale can this be, and what happens when the cache is wrong or empty?

### 84. Outbox, sagas, and 2PC

> *Cross-service transactions are workflows, not locks.*

- **Adopt:** Use the outbox pattern for reliable messaging, sagas with compensations for multi-step workflows, and two-phase commit only when the cost is justified. Make steps idempotent.
- **Smell:** Distributed transactions across unreliable services; dual writes without an outbox; sagas with no compensation.
- **Relearn:** What happens if step three fails after step two committed, and who compensates?

---

## 10. Reliability and Operations

*Production is the only test that counts. Reliability is a set of designed behaviors, not an aspiration.*

### 85. Crash-only software

> *Design for `kill -9`.*

- **Adopt:** Assume the process can die at any instruction. Make startup recover state, order writes and fsyncs, and test crash points. Graceful shutdown is an optimization, not a correctness requirement.
- **Smell:** Complex shutdown code that is never tested; corruption after power loss; state that exists only in memory.
- **Relearn:** If I kill -9 now, what survives, and how does the system recover?

### 86. Design for graceful degradation

> *Decide what to turn off before you have to.*

- **Adopt:** Define degraded modes: stale reads, reduced features, lower fidelity, shed load. Make them observable and reversible. Test them.
- **Smell:** All-or-nothing availability; no feature flags for degradation; discovering the fallback during an incident.
- **Relearn:** What is the minimum useful service, and how do I enter and leave that mode?

### 87. Timeouts and deadlines

> *A request without a deadline is a request that can hang forever.*

- **Adopt:** Set deadlines end to end and propagate them. Distinguish connect, read, and total timeouts. Cancel work when the deadline expires.
- **Smell:** Infinite timeouts; timeouts longer than the caller’s deadline; retries that ignore the remaining budget.
- **Relearn:** What is the deadline, who enforces it, and what is cancelled when it expires?

### 88. Retries with jitter

> *Retry the transient; never retry the permanent.*

- **Adopt:** Retry only idempotent operations, with exponential backoff and jitter, bounded attempts, and a circuit breaker. Respect the caller’s deadline.
- **Smell:** Retry storms; synchronized clients; infinite retries; retrying validation errors.
- **Relearn:** Is this error transient, is the operation idempotent, and what is the retry budget?

### 89. Circuit breakers and bulkheads

> *Contain the failure before it spreads.*

- **Adopt:** Use circuit breakers to stop hammering a failing dependency. Use bulkheads to isolate pools, threads, and resources. Fail fast with a defined fallback.
- **Smell:** One slow dependency exhausting all workers; retries amplifying an outage; no isolation between tenants.
- **Relearn:** What is isolated here, and what happens when this dependency is slow but not down?

### 90. Load shedding and admission control

> *It is better to reject some work than to fail all of it.*

- **Adopt:** Admit based on capacity, not hope. Shed low-priority work first. Return explicit overload errors. Protect the critical path.
- **Smell:** Unbounded queues; accepting work you cannot finish; fairness collapse under overload.
- **Relearn:** When demand exceeds capacity, which work do I drop first, and how do I tell the caller?

### 91. Health checks and readiness

> *Liveness asks “is it alive?”; readiness asks “should it receive traffic?”*

- **Adopt:** Separate liveness, readiness, and startup checks. Make readiness reflect dependencies and load. Avoid restart loops from aggressive liveness probes.
- **Smell:** One endpoint for everything; readiness that checks an optional dependency; liveness that fails under load.
- **Relearn:** What should happen when this check fails: restart, remove from rotation, or nothing?

### 92. Rolling upgrades and rollback

> *Every deploy is a distributed-systems event.*

- **Adopt:** Deploy incrementally, drain connections, support mixed versions, and keep rollback fast. Use canaries and feature flags. Test the rollback path.
- **Smell:** Big-bang deploys; no mixed-version compatibility; rollback that requires a migration reversal.
- **Relearn:** Can old and new versions run together, and how fast can I go back?

### 93. SLOs and error budgets

> *Reliability is a target, not a slogan.*

- **Adopt:** Define SLIs from the user’s perspective, set SLOs, and track error budgets. Use burn rates for alerts. Let the budget drive release and reliability work.
- **Smell:** 100% availability as a target; alerts on every metric; no distinction between user pain and internal noise.
- **Relearn:** What is the user-visible target, and what will we stop doing when the budget is exhausted?

### 94. Observability

> *Observability is a feature, not a debugging afterthought.*

- **Adopt:** Instrument boundaries with structured logs, metrics, and traces. Log decisions and state transitions, not just errors. Use correlation IDs. Design dashboards for p99 and saturation.
- **Smell:** `printf` debugging in production; logs without request context; metrics with unbounded cardinality; no way to follow one request.
- **Relearn:** Can I answer “what happened to this request?” without attaching a debugger?

### 95. Blameless postmortems

> *Blame stops learning.*

- **Adopt:** Write timelines, contributing factors, detection gaps, and action items with owners. Ask how the system allowed the error, not who caused it. Share the result.
- **Smell:** “Human error” as the root cause; action items with no owner; the same incident twice.
- **Relearn:** What would have caught this earlier, and what changed in the system, not just the person?

### 96. Chaos engineering and fault injection

> *Test the failure you fear before production tests it for you.*

- **Adopt:** Inject latency, errors, partitions, and crashes in controlled experiments. Start small, have a hypothesis, and define stop criteria. Run in staging first.
- **Smell:** “We’ll handle failures when they happen”; chaos without observability; experiments with no abort plan.
- **Relearn:** What failure have I not tested, and what do I expect the system to do?

### 97. Capacity planning

> *Capacity is a forecast plus a safety margin.*

- **Adopt:** Model growth, peak-to-average ratio, and failure headroom. Load test at expected and surge levels. Know the bottleneck and the cost of adding capacity.
- **Smell:** Scaling only after saturation; no headroom for a zone failure; capacity measured in machines, not user load.
- **Relearn:** At 2x traffic with one zone down, what breaks first?

### 98. Runbooks and on-call

> *An incident is not the time to discover the procedure.*

- **Adopt:** Write runbooks for known failure modes, with symptoms, checks, mitigations, and escalation. Practice them. Make on-call humane and rotated.
- **Smell:** Tribal knowledge; runbooks that are stale; one hero who always fixes it.
- **Relearn:** Can someone who did not build this service mitigate the top failure mode at 3 a.m.?

---

## 11. Security and Safety

*Security is a property of the whole system, including the parts you did not write and the people who operate it.*

### 99. Threat modeling

> *Ask who can do what to whom, and what stops them.*

- **Adopt:** Model assets, actors, trust boundaries, entry points, and mitigations. Use STRIDE or attack trees. Revisit when the architecture changes.
- **Smell:** Security as a final checklist; no model of the attacker; assuming the internal network is safe.
- **Relearn:** Who is the attacker, what do they want, and which boundary stops them?

### 100. Trust boundaries

> *Everything crossing a boundary is untrusted until proven otherwise.*

- **Adopt:** Identify boundaries: network, process, user, tenant, privilege. Validate and authenticate at each. Do not let trust leak across.
- **Smell:** Trusting internal services; parsing before authenticating; shared secrets across tenants.
- **Relearn:** Where does trust change here, and what proves the other side is allowed?

### 101. Least privilege

> *Grant the minimum, for the shortest time, with the fewest paths.*

- **Adopt:** Scope tokens, roles, file permissions, and capabilities. Use short-lived credentials. Separate duties. Audit and remove unused access.
- **Smell:** Root by default; wildcard IAM; long-lived keys; one service account for everything.
- **Relearn:** What is the smallest permission this needs, and how quickly can it be revoked?

### 102. Secure defaults

> *The default must be safe, because most users never change it.*

- **Adopt:** Deny by default; encrypt by default; close ports by default; require explicit opt-in for risky features. Make the safe path the easy path.
- **Smell:** Open by default; debug endpoints enabled in production; “set this flag to secure it.”
- **Relearn:** If the user changes nothing, what is exposed?

### 103. Defense in depth

> *Assume one layer will fail.*

- **Adopt:** Combine validation, authentication, authorization, isolation, encryption, monitoring, and rate limits. Do not rely on a single control.
- **Smell:** One firewall as the whole security model; validation only in the UI; no detection after prevention fails.
- **Relearn:** If this control fails, what catches the attack next?

### 104. Input validation and output encoding

> *Validate on the way in; encode on the way out.*

- **Adopt:** Use allowlists, length limits, and canonicalization. Parameterize queries. Encode for the target context: HTML, SQL, shell, URL, JSON.
- **Smell:** String concatenation into SQL, shell, or HTML; blocklists; trusting client-side validation.
- **Relearn:** What language is this data entering, and what encoding makes it inert?

### 105. Memory safety

> *Memory bugs are security bugs.*

- **Adopt:** Prefer memory-safe languages where possible. In unsafe code, minimize the unsafe surface, use bounds checks, and run sanitizers. Treat every unsafe block as a proof obligation.
- **Smell:** Unchecked lengths; use-after-free; integer overflow leading to allocation; unsafe code without tests.
- **Relearn:** What invariant makes this unsafe block sound, and what proves it?

### 106. Constant-time and side channels

> *Secrets leak through time, cache, and power.*

- **Adopt:** Use constant-time comparison for secrets. Avoid secret-dependent branches and memory access. Understand cache and speculative side channels. Use vetted crypto libraries.
- **Smell:** `memcmp` for MACs; branching on secret bits; rolling your own crypto.
- **Relearn:** Does this code path or memory access depend on a secret?

### 107. Secrets management

> *A secret in code is a secret forever.*

- **Adopt:** Store secrets in a vault or secret manager, inject at runtime, rotate regularly, and scope access. Never log them. Scan repositories and artifacts.
- **Smell:** Secrets in git; secrets in environment dumps; shared static keys; no rotation.
- **Relearn:** Where does this secret live, who can read it, and how is it rotated?

### 108. Supply chain security

> *You are running code you did not write.*

- **Adopt:** Pin dependencies, verify signatures and hashes, generate an SBOM, and scan for known vulnerabilities. Know what is in the build and where it came from.
- **Smell:** Unpinned transitive dependencies; blind `latest` tags; no provenance for artifacts.
- **Relearn:** Can I name every component in this artifact and prove where it came from?

### 109. Sandboxing and isolation

> *Assume the code is hostile and give it only what it needs.*

- **Adopt:** Use namespaces, cgroups, seccomp, capabilities, VMs, or WASM as appropriate. Separate tenants and privilege levels. Test escape paths.
- **Smell:** Running untrusted code with full privileges; shared kernel surface with no hardening; isolation claimed but untested.
- **Relearn:** If this component is compromised, what can it reach?

### 110. Denial of service and resource exhaustion

> *An attacker can spend your resources faster than you can earn them.*

- **Adopt:** Bound every resource, rate-limit at the edge, authenticate expensive operations, and shed load. Protect against algorithmic complexity attacks.
- **Smell:** Unbounded parsing; expensive work before authentication; one tenant able to starve others.
- **Relearn:** What is the cheapest request that costs me the most, and what stops it?

### 111. Crypto agility

> *Algorithms age; your protocol should survive their retirement.*

- **Adopt:** Version algorithms and keys, negotiate parameters, and keep migration paths. Avoid hard-coding one cipher or key size into the data format.
- **Smell:** Algorithm identifiers baked into wire formats; no re-encryption plan; “we’ll never need to change this.”
- **Relearn:** How would I replace this algorithm without breaking every stored record?

### 112. Audit logging

> *If it matters, it leaves a trail.*

- **Adopt:** Log security-relevant events with actor, action, target, result, and time. Protect logs from tampering. Make them queryable and retained appropriately.
- **Smell:** No record of privileged actions; logs that can be edited by the attacker; secrets in logs.
- **Relearn:** After an incident, could I reconstruct who did what and when?

---

## 12. Interfaces, ABI, and Evolution

*An interface is a promise about the future. Most of the cost is paid after you ship it.*

### 113. Narrow waist, thick ends

> *Narrow waist, thick ends.*

- **Adopt:** Keep a small, stable core contract. Put policy and richness at the edges. Compose small tools. Every feature added to the waist taxes every user.
- **Smell:** God interfaces; every feature in the core protocol; clients forced to implement options they never use.
- **Relearn:** What is the smallest stable contract this system needs? What can live at the edge instead?

### 114. Hyrum’s law

> *Every observable behavior becomes a contract — Hyrum’s law is not optional.*

- **Adopt:** Treat errors, ordering, timing, logs, and defaults as API. Document them, test them, and deprecate them deliberately. Assume users will depend on whatever they can observe.
- **Smell:** “It’s just an implementation detail” followed by a broken user; silent behavior changes in a patch release.
- **Relearn:** What will someone depend on if I ship this? What have I already promised by accident?

### 115. Postel’s law, carefully

> *Be conservative in what you send, liberal in what you accept — but not infinitely.*

- **Adopt:** Accept reasonable variation at boundaries, but reject ambiguity and enforce limits. Log and measure what you tolerate. Tighten over time with data.
- **Smell:** Parsing anything and hoping; silent coercion that hides bugs; unbounded tolerance becoming a security hole.
- **Relearn:** What variation do I accept, why, and when will I stop accepting it?

### 116. Compatibility is a promise

> *Compatibility is a promise, not a version number.*

- **Adopt:** Prefer additive changes. Version protocols and schemas. Support mixed-version operation during rollout. Write the migration and the rollback plan before you need them.
- **Smell:** Breaking changes in minor versions; no migration path; old and new nodes unable to talk during a deploy.
- **Relearn:** Can old and new versions run together during a rollout? What happens on rollback?

### 117. Semantic versioning and ABI stability

> *The version number is a communication tool; the ABI is the real contract.*

- **Adopt:** Define what breaking means for your API and ABI. Use symbol versioning, feature detection, and compatibility tests. Automate the check.
- **Smell:** Major version numbers that mean nothing; ABI breaks in patch releases; no compatibility test suite.
- **Relearn:** What exactly breaks if a user upgrades without recompiling?

### 118. Schema evolution

> *Data outlives code; design for the readers you have not met.*

- **Adopt:** Add fields with defaults, never reuse tags, keep old readers working, and plan backfills. Version schemas and test old and new readers together.
- **Smell:** Renaming fields in place; reusing IDs; migrations with no rollback; assuming all readers upgrade at once.
- **Relearn:** Can a reader from last year still parse data written today?

### 119. Deprecation and migration

> *Removing something is a project, not an announcement.*

- **Adopt:** Announce early, provide a migration path, instrument usage, offer a window, and remove only when usage is zero or the risk is accepted. Make the new path easier than the old one.
- **Smell:** “Deprecated” for years with no removal; surprise removals; no usage data.
- **Relearn:** Who still uses this, what is their migration path, and when will I actually remove it?

### 120. Mixed-version operation

> *During every deploy, two versions of your system are running at once.*

- **Adopt:** Design for N-1 and N+1 compatibility. Test rolling upgrades and rollbacks. Avoid assumptions that all nodes share the same code or schema.
- **Smell:** Features that require a simultaneous upgrade; schema changes that break old nodes; no mixed-version tests.
- **Relearn:** What happens when half the fleet is on the old version?

---

## 13. Build, Toolchain, and Supply Chain

*The artifact is part of the system. If you cannot reproduce it, you cannot trust it.*

### 121. Reproducible builds

> *Same source, same toolchain, same bytes.*

- **Adopt:** Pin inputs, remove timestamps and paths, use deterministic archives, and verify hashes. Build twice and compare.
- **Smell:** “It builds on my machine”; different binaries from the same commit; unreproducible release artifacts.
- **Relearn:** Can I rebuild this exact artifact and get the same hash?

### 122. Hermetic builds

> *A build should not reach the network or depend on ambient state.*

- **Adopt:** Vendor or cache dependencies, isolate the build environment, and declare every input. Fail when an undeclared dependency appears.
- **Smell:** Builds that download from the internet mid-run; reliance on globally installed tools; hidden environment variables.
- **Relearn:** What can change outside the source tree and still affect this build?

### 123. Dependency pinning and lockfiles

> *“Latest” is a moving target, not a version.*

- **Adopt:** Pin direct and transitive dependencies, commit lockfiles, and update deliberately with tests. Know your dependency graph and its licenses.
- **Smell:** Unpinned ranges in production; lockfiles not committed; surprise transitive updates.
- **Relearn:** What exactly will be installed tomorrow if I rebuild today?

### 124. SBOM and signed artifacts

> *Know what is inside, and prove who built it.*

- **Adopt:** Generate a software bill of materials, sign artifacts, verify signatures at deploy, and keep provenance. Automate vulnerability scanning against the SBOM.
- **Smell:** No inventory of components; unsigned binaries; “we’ll check it during the audit.”
- **Relearn:** Can I list every component and prove this artifact came from our build?

### 125. Static vs dynamic linking

> *Linkage is a tradeoff between isolation, size, startup, and upgradeability.*

- **Adopt:** Understand symbol resolution, runtime dependencies, and ABI coupling. Prefer static linking for hermetic deployment and dynamic linking for shared security updates, as appropriate.
- **Smell:** “Static is always better”; dependency hell from unpinned shared libraries; accidental symbol interposition.
- **Relearn:** What does this binary need at runtime, and what happens when that changes?

### 126. Cross-compilation and toolchains

> *The build machine is not the target machine.*

- **Adopt:** Pin the target triple, libc, kernel headers, and toolchain. Test on the target or an emulator. Separate host and target dependencies.
- **Smell:** Assuming `sizeof(void*)` is the same; using host headers; testing only on the build machine.
- **Relearn:** What exactly differs between the build host and the deployment target?

### 127. Debug symbols and optimization levels

> *Optimized code still needs to be debuggable.*

- **Adopt:** Keep separate debug symbols, preserve frame pointers where useful, and test with the optimization level you ship. Understand how inlining and reordering affect debugging.
- **Smell:** Shipping stripped binaries with no symbol server; debugging only `-O0`; “it works in debug.”
- **Relearn:** Can I get a useful stack trace from the exact binary that failed in production?

### 128. Environment parity and build caching

> *The more the build environment differs, the less the test means.*

- **Adopt:** Keep dev, CI, and production close. Cache builds by content, not time. Make cache keys include toolchain and inputs. Invalidate safely.
- **Smell:** “Works in CI, fails in prod”; stale caches producing wrong artifacts; environment drift.
- **Relearn:** What is different between the environment that passed and the one that failed?

---

## 14. Testing and Verification

*Testing does not prove correctness; it buys evidence. Spend it where the risk is.*

### 129. Test the failure paths

> *The happy path is the demo; the failure path is the product.*

- **Adopt:** Test timeouts, partial writes, crashes, malformed input, dependency errors, cancellation, and retries. Assert on the error contract, not just the absence of a crash.
- **Smell:** Only success cases; error tests that assert “some error”; failure paths never exercised.
- **Relearn:** Which failure path has never run in a test, and what would it do?

### 130. Boundary testing

> *Bugs live at the edges.*

- **Adopt:** Test zero, one, many, maximum, minimum, empty, null, duplicate, overflow, and partial input. Use table-driven and property-based tests.
- **Smell:** Only the median case; no empty or maximum test; off-by-one assumptions.
- **Relearn:** What are the boundaries of this input, and what happens exactly at each?

### 131. Property-based testing

> *State the invariant, then let the machine find the counterexample.*

- **Adopt:** Write properties for round-trips, idempotence, ordering, and invariants. Use generators that include edge cases. Shrink failures to a minimal case.
- **Smell:** Only example-based tests; properties that restate the implementation; no shrinking.
- **Relearn:** What must be true for all inputs, and can I generate the input that breaks it?

### 132. Differential and metamorphic testing

> *When you cannot state the answer, compare implementations or transformations.*

- **Adopt:** Compare old and new versions, two implementations, or transformed inputs with known relationships. Use differential testing for parsers, compilers, and protocols.
- **Smell:** No oracle for complex output; rewriting without a compatibility test; assuming two implementations agree.
- **Relearn:** What can I compare this against, and what relationship must hold under transformation?

### 133. Fuzzing

> *Let the machine explore the input space you did not imagine.*

- **Adopt:** Fuzz parsers, protocols, and decoders with coverage guidance. Keep a corpus, minimize crashes, and run continuously. Combine with sanitizers.
- **Smell:** Hand-written malformed inputs only; fuzzing without sanitizers; crashes triaged and forgotten.
- **Relearn:** What is the most hostile input this code can receive, and has it been tried?

### 134. Sanitizers and static analysis

> *Find the bug before it becomes an incident.*

- **Adopt:** Run address, undefined-behavior, thread, and memory sanitizers in CI. Use static analysis and linters with a clean baseline. Treat new warnings as failures.
- **Smell:** Sanitizers only on developer machines; a warning backlog nobody reads; disabling checks to make CI green.
- **Relearn:** Which class of bug would this tool catch, and is it running on every change?

### 135. Formal methods and model checking

> *For the highest-risk protocols, prove the invariant or explore the state space.*

- **Adopt:** Use TLA+, Alloy, or model checkers for concurrency and consensus designs. Specify the invariant, model the failure cases, and check small instances exhaustively.
- **Smell:** “We tested it”; consensus or locking protocols with no model; formal methods reserved for academics.
- **Relearn:** What is the invariant, and can I show it holds in every reachable state?

### 136. Deterministic simulation

> *Control time, scheduling, and failure to make hard bugs reproducible.*

- **Adopt:** Build a simulation harness with a virtual clock, deterministic scheduler, and injectable faults. Replay seeds. Use it for distributed and concurrent systems.
- **Smell:** Flaky tests you rerun until green; nondeterministic schedules; “cannot reproduce.”
- **Relearn:** Can I replay this exact failure with the same seed and schedule?

### 137. Load, soak, and stress testing

> *Normal tests prove it works; load tests prove it survives.*

- **Adopt:** Test at expected, peak, and beyond-capacity load. Soak for leaks and degradation. Stress to find the breaking point. Measure latency, errors, and saturation.
- **Smell:** Only functional tests; load tests with unrealistic data; no soak test for long-lived processes.
- **Relearn:** What breaks first at 2x load, and what does the system do at 10x?

### 138. Coverage is not correctness

> *Coverage tells you what ran, not what was checked.*

- **Adopt:** Use coverage to find untested paths, not as a quality score. Pair it with mutation testing and assertions. Focus on risk, not a percentage.
- **Smell:** Chasing 100% line coverage; tests with no assertions; critical branches covered but not verified.
- **Relearn:** Which high-risk path is unexecuted, and which executed path is unasserted?

### 139. Mutation testing

> *If the tests cannot kill a mutant, they cannot catch a bug.*

- **Adopt:** Inject small code mutations and check whether tests fail. Use it on critical modules. Fix weak assertions, not the mutation score.
- **Smell:** High coverage with weak tests; no mutation testing on core logic; tests that only check “no exception.”
- **Relearn:** If I changed this condition, would any test notice?

---

## 15. Debugging and Diagnosis

*The system is always right; your model is what needs updating.*

### 140. The bug is in your model

> *“Impossible” means your model is incomplete.*

- **Adopt:** When reality disagrees, update the model, not the story. Read the code, check the version, reproduce, and write down the actual rules. Compare belief with evidence.
- **Smell:** Blaming the compiler, OS, or library; closing unreproducible bugs; fixing symptoms without a model.
- **Relearn:** What do I believe that the evidence contradicts? Where did that belief come from?

### 141. Trust evidence, not memory

> *The system logs what happened; your memory logs what you expected.*

- **Adopt:** Read the logs, traces, counters, and core dumps before theorizing. Reproduce with instrumentation. Write down the timeline from evidence, not recollection.
- **Smell:** “I’m sure it was deployed”; debugging from memory; ignoring timestamps and versions.
- **Relearn:** What is the evidence, and what am I merely assuming?

### 142. Reproduce, reduce, bisect

> *Reproduce, reduce, bisect.*

- **Adopt:** Capture the exact input and environment. Shrink the case. Bisect changes or time. A deterministic reproduction is half the fix.
- **Smell:** Guessing fixes; “cannot reproduce” as a resolution; debugging by changing many things at once.
- **Relearn:** What is the smallest input that still fails, and what changed just before it started?

### 143. Change one thing at a time

> *One variable, one conclusion.*

- **Adopt:** Form a hypothesis, change one thing, and observe. Keep a log. Revert failed experiments. Use bisection when the search space is large.
- **Smell:** Changing five flags and declaring success; no control experiment; fixing three bugs at once.
- **Relearn:** What is the single variable I am testing, and what result would falsify my hypothesis?

### 144. Read the source; the stack goes all the way down

> *Read the source; the stack goes all the way down.*

- **Adopt:** Read the library, runtime, and kernel code when behavior matters. Use debuggers, tracers, and disassembly when needed. Trust code over comments and docs for edge cases.
- **Smell:** Cargo-culting snippets; trusting docs for edge cases; not checking the version actually in use.
- **Relearn:** Where in the source does this behavior actually come from? Have I read it?

### 145. Use the right tool

> *A debugger, a tracer, and a core dump answer different questions.*

- **Adopt:** Use breakpoints for state, sampling profilers for CPU, off-CPU traces for waits, `strace`/`bpftrace` for syscalls, and core dumps for crashes. Learn the tool before the incident.
- **Smell:** `printf` for everything; guessing at a hang; no core dumps enabled in production.
- **Relearn:** What question am I asking, and which tool actually answers it?

### 146. Understand the whole stack

> *The bug is rarely where the stack trace ends.*

- **Adopt:** Trace from user input through the runtime, kernel, network, and dependency. Know the layers you depend on. Build a mental map of the request path.
- **Smell:** Fixing the symptom in the top layer; blaming “the network” without tracing; no end-to-end view.
- **Relearn:** Can I draw the full path of this request, including every boundary it crosses?

### 147. Write it down

> *A bug you cannot explain twice is a bug you will meet again.*

- **Adopt:** Record the symptom, timeline, hypothesis, evidence, fix, and prevention. Link the incident, the commit, and the test. Make the next debugger faster.
- **Smell:** Fixes with no explanation; incidents that vanish; the same investigation repeated.
- **Relearn:** If I handed this to someone else, could they understand what happened and why the fix works?

---

## 16. Design and Complexity

*Complexity is the only thing that compounds against you. Design is the art of keeping it payable.*

### 148. Essential vs accidental complexity

> *Some complexity is the problem; most is the solution.*

- **Adopt:** Separate the complexity inherent in the domain from the complexity introduced by tools, layers, and conventions. Remove the accidental kind first.
- **Smell:** Blaming the domain for self-inflicted layers; abstractions that add more concepts than they hide.
- **Relearn:** Which part of this complexity is the problem, and which part did we add?

### 149. Simplicity is a destination

> *Simplicity is a destination, not a starting point.*

- **Adopt:** Prefer fewer moving parts, fewer options, and fewer layers. Delete code. Make the simple path the default. Complexity must pay rent.
- **Smell:** Config for everything; layers that only forward; abstractions with one implementation and no plan for a second.
- **Relearn:** What can I delete, and what actually breaks? If nothing breaks, why is it here?

### 150. YAGNI

> *You aren’t going to need it — until the requirement exists.*

- **Adopt:** Build for the requirement you have, with a seam for the one you expect. Defer irreversible decisions. Keep the design easy to extend, not pre-extended.
- **Smell:** Speculative generality; plugin systems with no plugins; configuration for features nobody asked for.
- **Relearn:** What present requirement justifies this abstraction, and what would it cost to add later?

### 151. Gall’s law

> *Working complex systems evolve from working simple systems.*

- **Adopt:** Start with the smallest system that works end to end. Grow it in tested increments. Do not design the final architecture on a whiteboard and build it in one leap.
- **Smell:** Grand rewrites; architecture that has never run; “we’ll add the simple path later.”
- **Relearn:** What is the smallest end-to-end version that delivers real value now?

### 152. Deep modules and information hiding

> *A good interface hides more than it exposes.*

- **Adopt:** Design modules with simple interfaces and substantial implementations. Hide decisions likely to change. Expose invariants, not internals.
- **Smell:** Shallow modules that only forward calls; leaking implementation types; callers depending on internal ordering.
- **Relearn:** What does this module hide, and can I change its implementation without touching callers?

### 153. Cohesion and coupling

> *Things that change together belong together; things that change apart should not depend on each other.*

- **Adopt:** Group by reason to change, not by layer or convention. Minimize cross-module knowledge. Prefer explicit dependencies over ambient state.
- **Smell:** A “utils” module everything imports; cycles; changing one feature touches ten packages.
- **Relearn:** What changes together here, and what is unnecessarily coupled?

### 154. Stable dependencies

> *Depend in the direction of stability.*

- **Adopt:** Put volatile policy at the edges and stable abstractions at the center. Avoid depending on details that change often. Invert dependencies where the direction is wrong.
- **Smell:** Core domain importing UI or infrastructure; every module depending on a fast-moving framework.
- **Relearn:** Which dependency is most likely to change, and why does the stable core depend on it?

### 155. Data-oriented design

> *Design around the data and its access patterns, not the taxonomy of nouns.*

- **Adopt:** Group data by how it is used. Prefer contiguous, cache-friendly layouts for hot paths. Separate hot and cold fields. Let the workload choose the structure.
- **Smell:** Deep object graphs in hot loops; virtual dispatch per element; structures optimized for modeling, not access.
- **Relearn:** What is the access pattern, and does the layout match it?

### 156. Unix philosophy

> *Small tools, clear interfaces, composable behavior.*

- **Adopt:** Do one thing well. Prefer text or simple formats at boundaries. Compose via pipes and protocols. Make each piece independently testable and replaceable.
- **Smell:** Monolithic tools with every mode; proprietary formats for simple data; composition only through a framework.
- **Relearn:** Can this be split into smaller pieces with a clean boundary, and should it be?

### 157. Reversibility and one-way doors

> *Some decisions are cheap to undo; spend your deliberation accordingly.*

- **Adopt:** Classify decisions as one-way or two-way doors. Move fast on reversible ones; slow down, document, and seek dissent on irreversible ones.
- **Smell:** Treating every choice as permanent; agonizing over reversible details; rushing schema or public API changes.
- **Relearn:** If this is wrong, how expensive is it to reverse, and who pays?

---

## 17. Craft, Teams, and Learning

*The system includes the people who build, operate, and change it. A mentality that cannot survive the team is not a mentality.*

### 158. Design for deletion

> *Code you can remove is code you can afford to own.*

- **Adopt:** Keep features behind flags, isolate experiments, and delete dead paths. Prefer small, replaceable modules over permanent scaffolding. Measure what is actually used.
- **Smell:** Code nobody dares remove; feature flags that become permanent; “temporary” workarounds from years ago.
- **Relearn:** What can I delete this quarter, and what is the evidence it is safe?

### 159. ADRs and documentation

> *Record the why, not just the what.*

- **Adopt:** Write short architecture decision records: context, decision, alternatives, consequences. Keep them near the code. Update or supersede them when the decision changes.
- **Smell:** Decisions only in chat history; docs that describe the old architecture; no record of tradeoffs.
- **Relearn:** Why was this designed this way, and what did we reject?

### 160. Naming is design

> *A name is a compressed explanation.*

- **Adopt:** Name by purpose and invariant, not by type or implementation. Rename when understanding improves. Use the domain language consistently.
- **Smell:** `Manager`, `Helper`, `Util`, `data`, `handle`; names that lie about behavior; synonyms for the same concept.
- **Relearn:** Does this name tell the next reader what it guarantees and what it costs?

### 161. Defaults and configuration

> *Every default is a decision made for someone who will not read the docs.*

- **Adopt:** Make defaults safe, fast, and useful. Minimize configuration. Validate at startup. Document the cost of changing each option.
- **Smell:** Dangerous defaults; hundreds of knobs; configuration that can silently break invariants.
- **Relearn:** If nobody configures this, what behavior do they get, and is it safe?

### 162. Code review as design review

> *Review the decision, not just the diff.*

- **Adopt:** Ask about invariants, failure modes, ownership, and operability. Name the mentality and the concrete risk. Keep reviews small enough to think in.
- **Smell:** Style-only reviews; rubber-stamping; comments that say “nit” on a design flaw.
- **Relearn:** What could go wrong after this merges, and did the review ask about it?

### 163. Refactoring and technical debt

> *Debt is a tool; interest is the problem.*

- **Adopt:** Refactor in small, behavior-preserving steps with tests. Track debt explicitly. Pay it down where it slows delivery or increases risk. Distinguish deliberate debt from neglect.
- **Smell:** “We’ll clean it up later” with no owner; rewrites that mix behavior change and refactoring; debt nobody can name.
- **Relearn:** What interest is this debt charging, and what is the smallest payment that reduces it?

### 164. Chesterton’s fence

> *Do not remove a fence until you know why it was put there.*

- **Adopt:** Before changing or deleting something, learn why it exists. Check history, tests, incidents, and the people who wrote it. Replace it incrementally, with evidence.
- **Smell:** “This is dumb, delete it” followed by an incident; refactoring away a workaround without understanding the bug it worked around.
- **Relearn:** What problem was this solving? Is that problem gone, or did it just move?

### 165. The boy scout rule

> *Leave the code a little better than you found it.*

- **Adopt:** Make small, safe improvements in the direction of the design: a clearer name, a removed dead branch, a test for an uncovered invariant. Do not smuggle a rewrite into a bug fix.
- **Smell:** “Not my code”; drive-by refactors mixed with behavior changes; cleanliness deferred forever.
- **Relearn:** What is one small improvement I can leave behind without increasing risk?

### 166. Teach it to learn it

> *Teach it to learn it.*

- **Adopt:** Explain the mentality in five minutes with a concrete example. Write it down. Turn it into a review checklist item or a test. Teaching is the fastest way to find the holes in your understanding.
- **Smell:** Knowledge silos; “only Alice understands this”; docs that describe what, never why.
- **Relearn:** Can I explain this without notes, and can the listener apply it to a new case?

### 167. Blameless learning

> *Blame stops learning.*

- **Adopt:** Treat incidents as system behavior, not personal failure. Ask what information, guardrail, or design made the error easy. Share the learning widely.
- **Smell:** “Human error” as the root cause; punishment for reporting; the same incident twice.
- **Relearn:** What would have made the safe action the easy action?

### 168. Ownership and on-call

> *You build it, you run it, you learn from it.*

- **Adopt:** Give teams ownership of the full lifecycle: design, deploy, operate, and retire. Make on-call sustainable. Feed production pain back into design.
- **Smell:** “Throw it over the wall”; developers who never see pager alerts; on-call as punishment.
- **Relearn:** Who feels the consequences of this design at 3 a.m., and do they have a voice in it?

---

## 18. Physical, Economic, and Human Realities

*Software runs on hardware, costs money, and is operated by people. These are not externalities.*

### 169. Real-time is a deadline contract

> *A deadline missed is a failure, not a slow success.*

- **Adopt:** Classify hard, firm, and soft deadlines. Bound worst-case execution time, jitter, and blocking. Test under load and interference. Keep unbounded operations out of the critical path.
- **Smell:** Average latency used to claim real-time; GC or page faults in a control loop; priorities without worst-case analysis.
- **Relearn:** What is the worst-case time, not the average, and what happens when it is exceeded?

### 170. Firmware, boot, and the trusted chain

> *The system starts before your program does.*

- **Adopt:** Understand firmware, bootloaders, secure boot, measured boot, and update paths. Verify signatures and rollback protection. Treat the boot chain as part of the attack surface.
- **Smell:** Assuming the OS is the root of trust; unsigned firmware updates; no recovery path for a bad boot.
- **Relearn:** What runs before my code, and how do I know it was not modified?

### 171. Power, thermal, and energy budgets

> *Every operation has an energy cost and a heat budget.*

- **Adopt:** Measure power and thermal behavior, not just latency. Use frequency scaling, idle states, and batching. Avoid busy-waiting. Design for throttling and battery limits.
- **Smell:** Spin loops; ignoring thermal throttling in benchmarks; performance claims that assume unlimited power.
- **Relearn:** What happens to this workload when the CPU is throttled or the battery is low?

### 172. Cost is a design constraint

> *The bill is part of the architecture.*

- **Adopt:** Model compute, storage, egress, and operational cost per request. Set budgets and alerts. Prefer designs whose cost scales with value, not with accidental work.
- **Smell:** Cost discovered at the end of the month; unbounded cardinality; retaining data forever “because storage is cheap.”
- **Relearn:** What does one request cost, and which part scales worst?

### 173. Data lifecycle and privacy

> *Data is a liability with a lifetime.*

- **Adopt:** Classify data, minimize collection, define retention and deletion, encrypt at rest and in transit, and support subject access and deletion. Know where every copy lives.
- **Smell:** Collect everything; logs and backups outside the retention policy; deletion that misses replicas and caches.
- **Relearn:** Where does this data live, how long, who can see it, and how is it deleted?

### 174. Multi-region and disaster recovery

> *A region is a failure domain, not a backup.*

- **Adopt:** Define RPO and RTO, replication topology, failover and failback, and data conflict policy. Test regional loss. Account for quorum and latency across regions.
- **Smell:** Multi-region with no failover test; replicas in the same region; failback that loses writes.
- **Relearn:** If this region disappears now, what is lost, and how long until service returns?

### 175. Incident command and communication

> *During an incident, clarity is a control.*

- **Adopt:** Assign an incident commander, operations lead, and communications lead. Keep a shared timeline and status updates. Separate mitigation from root-cause analysis.
- **Smell:** Everyone debugging, nobody communicating; conflicting changes; status updates that say “still looking.”
- **Relearn:** Who is in charge, who is communicating, and what is the current hypothesis?

### 176. API limits, pagination, and fairness

> *An API without limits is a denial-of-service invitation.*

- **Adopt:** Define rate limits, quotas, pagination, idempotency, and fair queuing. Return actionable errors and retry hints. Protect shared resources from one tenant.
- **Smell:** Unbounded list endpoints; no rate limits; one tenant starving others; pagination that shifts under writes.
- **Relearn:** What stops one client from consuming the service, and how does a well-behaved client paginate safely?

---

## Coverage matrix

Use this to check that a review, a curriculum, or a team charter is not looking at only one dimension.

| Quality | Domains that carry it | Core question |
| --- | --- | --- |
| **Correctness** | 0, 3, 4, 5, 14 | Can the wrong thing happen, and would we know? |
| **Performance** | 1, 2, 6, 7, 8, 9 | What does the workload cost, and where is the bottleneck? |
| **Security** | 2, 11, 13, 14 | Who can reach it, and what stops them? |
| **Operability** | 7, 10, 12, 15 | Can we run it, observe it, and recover it? |
| **Evolvability** | 9, 12, 16, 17 | Can we change it without breaking the world? |
| **Sustainability** | 1, 6, 10, 18 | What does it cost in money, energy, and attention, and who pays? |

No domain is optional. A system with excellent correctness and no operability is a time bomb; a system with excellent performance and no security is a liability; a system with excellent code and no team ownership is a bus factor.

---

## The self-audit

Score each statement: **yes**, **no**, or **not sure**. “Not sure” is a “no” with better manners.

- [ ] I can name the most expensive operation in my hot path and its rough cost.
- [ ] Every important invariant has an owner, a test, and a failure detector.
- [ ] Invalid states are rejected by types or constructors, not by scattered checks.
- [ ] Every resource has exactly one release on every path.
- [ ] Every queue, buffer, retry, recursion, and connection is bounded.
- [ ] Every shared mutable value has exactly one writer.
- [ ] I can describe the worst interleaving of my concurrent code.
- [ ] I know my p50, p99, and the current bottleneck.
- [ ] I know what is durable after a crash at every critical write point.
- [ ] `kill -9` is a supported operation, and recovery is tested.
- [ ] Every request has a deadline, a trace, and an idempotency story.
- [ ] I know which dependency can fail without taking the system down.
- [ ] The system has a defined degraded mode, and it has been tested.
- [ ] Secrets are stored, scoped, rotated, and never logged.
- [ ] Untrusted input crosses a clear boundary and becomes a typed value.
- [ ] Old and new versions can run together during a rollout.
- [ ] I can rebuild the exact artifact and verify its provenance.
- [ ] The critical failure paths have tests, not just the happy path.
- [ ] I can reproduce the last production bug deterministically.
- [ ] I can explain why the weirdest piece of code in my area exists.
- [ ] I know the worst-case latency, not just the average, for real-time paths.
- [ ] I know what runs before my code in the boot chain and how it is verified.
- [ ] I know the cost per request and which component scales worst.
- [ ] I know where data lives, how long it is retained, and how it is deleted.
- [ ] I know what happens if a region disappears, and when I last tested it.
- [ ] I know who commands and communicates during an incident.
- [ ] I know what stops one client from consuming the whole service.

**Reading the score:** 23–27 yes: pick one “not sure” and close it. 15–22: choose three entries from different domains and make them habits. 0–14: start with the twelve in [Start here](#start-here) and add one layer per month.

---

## The code review lens

Paste the relevant rows into a pull request template, or use them as review prompts.

| Lens | Ask | Mentalities |
| --- | --- | --- |
| **Correctness** | Are illegal states possible? Is every failure handled? What happens at zero, one, and many? | 33–40 |
| **Resources** | Who owns this? Is every acquisition released? Is it bounded? Is that copy deliberate? | 25–32 |
| **Concurrency** | Who writes? What is the worst interleaving? Can this block? What applies backpressure? | 41–48 |
| **Performance** | What did we measure? Where is the bottleneck? How is locality? Are we paying per-item fixed costs? | 49–56 |
| **Storage** | What survives a crash? Is the write ordered and durable? Can we restore? | 57–64 |
| **Network** | Where is the message boundary? What if it is slow, partial, duplicated, or hostile? | 65–72 |
| **Distribution** | What if this dependency is unreachable? What is the consistency model? Is the effect idempotent? | 73–84 |
| **Reliability** | Deadline? Retry budget? Circuit breaker? What is the degraded mode? Can we observe it? | 85–98 |
| **Security** | Who can reach this? What is trusted? What is the least privilege? What leaks? | 99–112 |
| **Evolution** | Is this a promise? Can versions coexist? Is there a migration and rollback? | 113–120 |
| **Build** | Is the artifact reproducible, pinned, signed, and debuggable? | 121–128 |
| **Verification** | What failure path is tested? What is the oracle? Would a mutant survive? | 129–139 |
| **Debuggability** | Can we follow one request end to end? Do we understand the code we are changing? | 140–147 |
| **Design** | Is this simpler? What does it hide? What changes together? Is it reversible? | 148–157 |
| **Team** | Who owns this at 3 a.m.? Is the why recorded? Can someone else change it? | 158–168 |
| **Realities** | What is the worst-case deadline? What does it cost? Where does the data go? What if a region is lost? | 169–176 |

A good review comment names the mentality and the concrete risk:

> “This queue is unbounded (#27). Under a slow consumer it will grow until OOM. Can we cap it and reject with backpressure (#90)?”

---

## Anti-mentalities

The fastest way to lose a good mentality is to keep a comforting one nearby.

| Anti-mentality | What it sounds like | Replacement |
| --- | --- | --- |
| **Machine blindness** | “It’s fast enough.” | “Fast enough for which workload, at which percentile?” (#49) |
| **Happy-path engineering** | “That’ll never happen.” | “Design the failure path first.” (#8, #129) |
| **Lock optimism** | “Just add a lock.” | “Find the invariant; locks are a last resort.” (#42, #43) |
| **Unbounded optimism** | “Memory is cheap.” | “Bound it and choose a policy at the limit.” (#27) |
| **Copy reflex** | “Just clone it.” | “Is this copy buying isolation or wasting bandwidth?” (#29) |
| **Boundary denial** | “It works on my machine.” | “Then make my machine reproducible.” (#142) |
| **Component tribalism** | “Not my service.” | “Follow the request end to end.” (#3, #146) |
| **Abstraction hoarding** | “More layers means more general.” | “Abstraction is a cost paid on every read.” (#149) |
| **Rewrite romance** | “Let’s rewrite it.” | “Understand it, then replace it incrementally.” (#164) |
| **Documentation debt** | “The code is the documentation.” | “The code says what; the why is still missing.” (#159) |
| **Retry optimism** | “It’ll probably work the second time.” | “Retry only the transient, only if idempotent, with a budget.” (#88) |
| **Timeout denial** | “It usually responds quickly.” | “A request without a deadline can hang forever.” (#87) |
| **Security as a phase** | “We’ll harden it later.” | “Model the attacker before the architecture hardens.” (#99) |
| **Deploy courage** | “Big bang, then fix forward.” | “Roll out incrementally with a tested rollback.” (#92) |
| **Observability as logging** | “We have logs.” | “Can you follow one request through the whole system?” (#94) |
| **Hero culture** | “Alice will fix it.” | “Make the runbook work for someone who did not build it.” (#98) |
| **Coverage theater** | “We have 90% coverage.” | “Would a mutant survive this test?” (#139) |
| **Green CI, broken prod** | “All tests pass.” | “Which failure path, workload, and version did we not test?” (#129, #137) |
| **Average-time realism** | “The average is 5 ms.” | “What is the worst case, and what misses the deadline?” (#169) |
| **Boot blindness** | “The OS handles that.” | “What runs before the OS, and how is it verified?” (#170) |
| **Cost as an afterthought** | “Storage is cheap.” | “What does one request cost, and which part scales worst?” (#172) |
| **Data hoarding** | “We might need it later.” | “What is the retention, access, and deletion policy?” (#173) |
| **DR theater** | “We have multiple regions.” | “When did we last fail over and fail back?” (#174) |
| **Incident chaos** | “Everyone debug at once.” | “Who commands, who communicates, and what is the hypothesis?” (#175) |
| **Unlimited API** | “Clients can request what they need.” | “What stops one client from starving the rest?” (#176) |

---

## How to share this

Pick the format that fits your team, not the one that feels most impressive.

- **15-minute brown bag:** one mentality, one war story, one code example from your own repository. End with a question, not a summary.
- **Review comments:** cite the number. “This is #27” is faster than a paragraph and easier to remember.
- **Onboarding:** choose the ten mentalities that matter most for your codebase and make them required reading. Add a first-week exercise for each.
- **Postmortems:** map contributing factors to missing mentalities. Ask “which invariant did we not have?” rather than “who made the mistake?”
- **Team charter:** select five mentalities as explicit norms, and write down what violating them looks like in practice.
- **Architecture reviews:** walk the [stack map](#the-stack-map) from the layer under change to the layers above and below it.
- **Interviewing:** ask candidates to reason through one mentality on a system they know. The answer matters less than the model.
- **Lunch-and-learn series:** one domain per week, cycling through the stack, then repeat with new examples.
- **Personal practice:** keep a one-line journal: mentality, decision, result. Review it monthly.

---

## A seven-week relearning plan

Fifteen minutes a day. One artifact per week. The artifact is the point.

### Week 1 — Foundations, machine, and OS

- Read entries 1–24. Pick one hot path and annotate its costs: copies, syscalls, cache misses, allocations.
- Run a profiler or tracer. Find one number that surprises you.
- Apply one fix: batch, buffer, reorder, or remove a copy.
- **Artifact:** a one-page cost map of one real code path.

### Week 2 — Memory, correctness, and concurrency

- Read entries 25–48. Find one “impossible” state and one shared mutable value.
- Encode the state in a type or constructor; make the writer single or explicit.
- Add boundary tests and one invariant assertion.
- **Artifact:** a pull request that removes a class of bug, not one instance.

### Week 3 — Performance, storage, and networking

- Read entries 49–72. Pick one request path and one storage path.
- Measure p50/p99, find the bottleneck, and check what survives a crash.
- Add a deadline, a frame boundary, or a durability test.
- **Artifact:** a measured before/after with a reproducible benchmark.

### Week 4 — Distribution, reliability, and security

- Read entries 73–112. Pick one dependency and one trust boundary.
- Add a timeout, retry budget, idempotency key, or least-privilege scope.
- Inject one failure and observe the degraded mode.
- **Artifact:** a failure-mode diagram plus a tested mitigation.

### Week 5 — Interfaces, build, testing, and debugging

- Read entries 113–147. Pick one public contract and one critical failure path.
- Check mixed-version compatibility and artifact reproducibility.
- Add a failure-path test or a fuzz target; reproduce one bug deterministically.
- **Artifact:** a compatibility note and a regression test.

### Week 6 — Design, craft, and team

- Read entries 148–168. Pick one piece of accidental complexity and one undocumented decision.
- Delete or simplify something; write an ADR; review a design with the checklist.
- Teach one mentality to a colleague and ask them to apply it.
- **Artifact:** an ADR plus a colleague who can explain the mentality without you.

### Week 7 — Physical, economic, and human realities

- Read entries 169–176. Pick one latency-critical path, one cost line, and one data set.
- Measure worst-case latency, cost per request, and retention/deletion behavior.
- Test one regional failover or incident communication path.
- **Artifact:** a one-page reality check covering deadlines, cost, data lifecycle, and ownership.

---

## Further reading

Grouped by the layer they deepen.

**Foundations and craft**

- *The Practice of Programming*, Kernighan and Pike — debugging, testing, and small correct code.
- *A Philosophy of Software Design*, John Ousterhout — complexity, deep modules, and interface design.
- *The Art of Unix Programming*, Eric S. Raymond — narrow interfaces, composition, and the Unix philosophy.
- *Programming Pearls*, Jon Bentley — problem definition, back-of-the-envelope reasoning, and performance intuition.

**Machine, OS, and performance**

- *Computer Systems: A Programmer’s Perspective*, Bryant and O’Hallaron — the machine and the memory hierarchy from first principles.
- *Systems Performance* and *BPF Performance Tools*, Brendan Gregg — measurement, tracing, and the reality of production.
- *The Linux Programming Interface*, Michael Kerrisk — what the kernel actually does when you call it.
- *What Every Programmer Should Know About Memory*, Ulrich Drepper — the canonical deep dive on caches and NUMA.

**Storage and distributed systems**

- *Designing Data-Intensive Applications*, Martin Kleppmann — replication, partitioning, consistency, and evolution.
- *Database Internals*, Alex Petrov — storage engines, B-trees, LSM trees, and concurrency.
- *Site Reliability Engineering* and *The Site Reliability Workbook*, Google — SLOs, operations, and blameless learning.
- *Release It!*, Michael Nygard — timeouts, circuit breakers, bulkheads, and stability patterns.

**Security and verification**

- *Threat Modeling: Designing for Security*, Adam Shostack — a practical method for attacker thinking.
- *The Tangled Web*, Michal Zalewski — the security assumptions behind browsers and the web.
- *Security Engineering*, Ross Anderson — the broad systems view of security and incentives.
- *Software Foundations* or *Concrete Semantics* — formal reasoning for the highest-risk protocols.

**Concurrency and languages**

- *The Rust Programming Language* and *The Rustonomicon* — ownership and unsafe code as explicit design tools.
- *Java Concurrency in Practice*, Brian Goetz — the canonical treatment of shared-memory concurrency.
- *Communicating Sequential Processes*, Tony Hoare — the model behind channels and message passing.

**Teams and learning**

- *Accelerate*, Forsgren, Humble, and Kim — delivery performance as a system property.
- *The Phoenix Project* and *The DevOps Handbook* — flow, feedback, and learning across the lifecycle.
- *Thinking in Systems*, Donella Meadows — feedback loops, leverage points, and unintended consequences.

**Physical, economic, and human realities**

- *Real-Time Systems*, Jane W. S. Liu — deadlines, scheduling, and worst-case analysis.
- *Making Embedded Systems*, Elecia White — firmware, hardware constraints, and practical embedded design.
- *Cloud FinOps*, J. R. Storment and Mike Fuller — cost as an engineering constraint.
- *Incident Management for Operations*, Rob Schnepp et al. — command, communication, and coordination under pressure.
- *The Checklist Manifesto*, Atul Gawande — checklists and disciplined execution in high-stakes work.

---

## Contributing

This is a living document. Add a mentality if it changes a real decision, not if it sounds good on a slide.

Use this template:

```markdown
### N. Title

> *Mantra in one sentence.*

- **Adopt:** One concrete practice, with an example if possible.
- **Smell:** The observable anti-pattern that means the mentality is missing.
- **Relearn:** A question that exposes whether you actually understand it.
```

Guidelines:

- One idea per entry. If it needs two mantras, it is two entries.
- Keep each field under about 80 words.
- Prefer a concrete example over an adjective.
- Cite the source when an idea is borrowed (Hyrum’s law, Amdahl’s law, Gall’s law, Chesterton’s fence, Postel’s law).
- Avoid language wars. The catalog is about decisions, not tribes.
- If you add an entry, update the numbering, the stack map, the coverage matrix, and the code review lens.
- If you remove an entry, say why in the pull request; deletion is a design decision too.

---

> Pick one. Practice it on real code. Teach it to someone. That is the whole method.
