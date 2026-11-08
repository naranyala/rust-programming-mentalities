# Rust Programming Mentalities

> 72 compact habits for writing Rust that is correct, idiomatic, fast, and maintainable.

Rust is not just “C with a borrow checker.” It is a language with its own design pressure: ownership replaces manual memory management, `Result` replaces exceptions, traits replace inheritance, and `unsafe` is a proof obligation rather than a permission slip. These mentalities are written **for Rust specifically** — each one names Rust types, crates, tools, and failure modes.

The broader, language-agnostic catalog lives in [GENERAL.md](GENERAL.md). This file is the Rust edition: shorter entries, sharper practices, and no advice that ignores the borrow checker.

---

## Contents

**Start:** [How to use](#how-to-use) · [The Rust layer map](#the-rust-layer-map) · [Start here](#start-here)

**Catalog:** [Complete index](#complete-index) — all 72 entries

**Reference:** [Self-audit](#self-audit) · [Review lens](#review-lens) · [Anti-patterns](#anti-patterns) · [How to share](#how-to-share) · [Four-week plan](#four-week-plan) · [Further reading](#further-reading) · [Contributing](#contributing)

---

## How to use

| Mode | What it looks like | Cadence |
| --- | --- | --- |
| **Adopt** | Pick one mentality. Apply it to a real crate or pull request. | One per week |
| **Share** | Cite the number in review, or run a 15-minute brown bag. | One per sprint |
| **Relearn** | Answer the relearn question from memory, then teach it. | One per month |

Three rules:

1. **One mentality at a time.** Depth changes code; breadth changes nothing.
2. **A mentality without a diff is a slogan.** Every entry must alter a real decision.
3. **If you cannot name the Rust cost, you have not adopted it.** “It compiles” is not an argument.

Examples assume stable Rust. Nightly-only features are called out explicitly.

---

## The Rust layer map

Read **bottom-up** to go from ownership to ecosystem. Read **top-down** to trace a compile error or production bug to its design cause.

| Layer | Domain | Core question | Entries |
| --- | --- | --- | --- |
| 11 | Tooling, build, and ecosystem | Can I reproduce and trust the build? | 67–72 |
| 10 | Testing, debugging, and verification | How do I know it works? | 61–66 |
| 9 | API and trait design | What have I promised to callers? | 55–60 |
| 8 | Performance and iterators | What does this abstraction cost? | 49–54 |
| 7 | Async and runtimes | What happens while this task waits? | 43–48 |
| 6 | Concurrency | What can run at the same time, and what is shared? | 37–42 |
| 5 | Unsafe and FFI | What invariant makes this sound? | 31–36 |
| 4 | Memory and allocation | Who allocates, and when is it freed? | 25–30 |
| 3 | Errors and `Option` | Is this absent, failed, or a bug? | 19–24 |
| 2 | Types and invariants | Can the wrong thing even compile? | 13–18 |
| 1 | Lifetimes and references | What keeps this data alive? | 7–12 |
| 0 | Ownership and borrowing | Who owns this, and who may borrow it? | 1–6 |

---

## Start here

If 72 is too many, begin with these ten. They prevent the most common Rust design mistakes.

**1** borrow, don’t clone · **3** one owner · **13** illegal states · **19** `Result` is the default · **31** unsafe is a proof · **37** `Send`/`Sync` are contracts · **44** don’t block the executor · **49** iterators are zero-cost · **55** accept borrowed, return owned · **64** Miri, loom, sanitizers

Then add the layer you touch most: ownership, async, unsafe, or API design.

---

## 0. Ownership and Borrowing

*The borrow checker is not an obstacle; it is a design reviewer that never gets tired.*

### 1. Borrow, don’t clone

> *A clone is a design decision, not a compiler appeasement.*

- **Adopt:** Pass `&T` or `&mut T`; clone only for isolation, ownership transfer, or measured simplicity. Name the reason in review.
- **Smell:** `.clone()` added to silence a borrow error; clones inside hot loops; cloning before a function that could take ownership.
- **Relearn:** Is this clone buying independence, or hiding an ownership design problem?

### 2. Move is a feature

> *Transfer ownership; let the old binding die.*

- **Adopt:** Move values into functions and structs. Use `std::mem::take` or `replace` when you need to move out of a borrowed place.
- **Smell:** Cloning before passing to a function that could accept ownership; fighting move errors with `.clone()`.
- **Relearn:** Who should own this value after the call?

### 3. One owner at a time

> *Shared ownership is a tool, not a default.*

- **Adopt:** Model ownership in structs. Use `Rc` or `Arc` only when shared ownership is genuinely required; prefer borrowing otherwise.
- **Smell:** `Rc<RefCell<T>>` everywhere; `Arc<Mutex<T>>` for data only one task touches.
- **Relearn:** Does this value need shared ownership, or just a borrow?

### 4. Split borrows instead of fighting the checker

> *If two fields are independent, borrow them independently.*

- **Adopt:** Destructure structs into disjoint fields; write methods that borrow only the fields they need; use free functions when `&mut self` is too broad.
- **Smell:** A `&mut self` method that needs two fields and fails to compile; reaching for `RefCell` to work around field-level borrowing.
- **Relearn:** Which fields are actually borrowed at the same time?

### 5. Interior mutability is a deliberate escape hatch

> *`Cell`, `RefCell`, `Mutex`, and atomics change the rules — say why.*

- **Adopt:** Use `Cell`/`RefCell` for single-threaded interior mutation, `Mutex`/`RwLock`/atomics for shared state, and keep the escape hatch encapsulated.
- **Smell:** `RefCell` runtime panics in production; `Mutex` around single-threaded data; interior mutability leaking through public APIs.
- **Relearn:** Why can’t this be `&mut`, and what runtime cost am I accepting?

### 6. `Drop` is part of the contract

> *Cleanup happens on every path, including the ones you did not write.*

- **Adopt:** Use RAII guards for files, locks, and transactions. Implement `Drop` for resources, but do not rely on it for async cleanup or fallible work.
- **Smell:** Manual `close()` on the happy path; leaked file descriptors; `Drop` that panics; `mem::forget` used casually.
- **Relearn:** What happens when this value is dropped, and on which paths?

---

## 1. Lifetimes and References

*Lifetimes describe relationships. They do not create lifetime.*

### 7. Lifetimes describe, not create

> *`&'a T` means “borrow valid for `'a`,” not “make it live longer.”*

- **Adopt:** Fix the owner, not the annotation. Read lifetime errors as “this reference outlives its data,” then change the ownership or scope.
- **Smell:** Adding `'a` everywhere to silence the compiler; lifetime parameters that do not express a real relationship.
- **Relearn:** Which value actually owns the data, and how long does it live?

### 8. Prefer owned data at boundaries

> *Borrow internally; own at the edge.*

- **Adopt:** Return `String` or `Vec<T>` from public APIs when the caller needs ownership. Accept `&str`, `&[T]`, and `impl AsRef<T>` on the way in.
- **Smell:** Returning `&str` tied to a temporary; lifetime gymnastics across module boundaries; `&String` and `&Vec<T>` parameters.
- **Relearn:** Does the caller need a borrow or ownership?

### 9. Do not outlive your borrow

> *A reference is only as good as its owner.*

- **Adopt:** Keep the owner alive for the whole borrow. Use `Arc` or `Cow` when data must be shared or conditionally owned. Never return a reference to a local.
- **Smell:** `return &local`; dangling references hidden behind `unsafe`; storing references in long-lived structs without a plan.
- **Relearn:** What keeps this data alive while the reference exists?

### 10. `'static` is not “forever”

> *`&'static T` and `T: 'static` are different promises.*

- **Adopt:** Distinguish a static borrow from a type that owns all its data. Use `Box::leak` only for genuinely process-lifetime data.
- **Smell:** `Box::leak` to satisfy `'static`; `'static` bounds added everywhere; leaking memory to spawn a task.
- **Relearn:** Does this need to live forever, or just own its data?

### 11. Variance matters

> *Lifetimes and generics have subtyping rules; wrappers can change them.*

- **Adopt:** Understand covariance, contravariance, and invariance. Use `PhantomData` to control variance and auto traits deliberately.
- **Smell:** Unexpected lifetime errors in generic wrappers; `PhantomData` copied without understanding; `unsafe` variance assumptions.
- **Relearn:** What variance does this type have, and why?

### 12. Self-referential types need a plan

> *A value that borrows itself cannot move casually.*

- **Adopt:** Prefer a redesign that avoids self-reference. If you need it, use `Pin`, `self_cell`, or `ouroboros` and understand the safety conditions.
- **Smell:** Hand-rolled self-referential structs; `Pin` used as a buzzword; unsafe code that assumes a stable address without enforcing it.
- **Relearn:** Can I avoid the self-reference, or do I need a pinned owner?

---

## 2. Types and Invariants

*The cheapest bug is the one the compiler refuses to accept.*

### 13. Make illegal states unrepresentable

> *Use the type system as the first test suite.*

- **Adopt:** Use enums for alternatives and structs for “all fields together.” Make invalid combinations impossible to construct; validate at the boundary.
- **Smell:** `Option` fields that must be `Some` together; `is_running` and `is_failed` booleans that can both be true; validation repeated in every caller.
- **Relearn:** Which combination of fields is nonsense, and why does the type allow it?

### 14. Newtype everything meaningful

> *A raw `u64` does not say what it means.*

- **Adopt:** Use `struct UserId(u64)`, `struct Meters(f64)`, and `struct Email(String)`. Implement only the traits that make sense.
- **Smell:** Raw integers for every ID; unit mixups; `String` for every semantic string.
- **Relearn:** What does this primitive actually mean, and can the type say it?

### 15. Enums over boolean flags

> *A state is not a bag of booleans.*

- **Adopt:** Model states as an enum and match exhaustively. Let the compiler tell you when a new variant needs handling.
- **Smell:** `is_running`, `is_paused`, `is_failed`; `match` with a catch-all that hides new variants.
- **Relearn:** What are the legal states, and what happens in each?

### 16. Parse, don’t validate

> *Turn untrusted input into a type that cannot be wrong.*

- **Adopt:** Implement `TryFrom`, `FromStr`, or a constructor that returns `Result<Validated, Error>`. Pass the validated type inward.
- **Smell:** `validate()` returning `bool`; raw `String` passed through ten layers; checks repeated at every boundary.
- **Relearn:** What type proves this input is valid on the other side?

### 17. Typestate for protocols

> *If a call is only valid in one state, encode the state in the type.*

- **Adopt:** Use marker types such as `Connecting`, `Connected`, and `Closed`. Consume `self` on transitions so invalid sequences cannot compile.
- **Smell:** Runtime checks for states the compiler could enforce; “you must call `start()` first” comments.
- **Relearn:** Can the compiler prevent this call in the wrong state?

### 18. Sealed traits for controlled extension

> *A public trait is a promise about downstream implementations.*

- **Adopt:** Use a private supertrait to seal traits that are not meant to be implemented externally. Document the extension policy.
- **Smell:** Public traits that are de facto sealed; downstream impls blocking a breaking change; adding methods to widely implemented traits.
- **Relearn:** Should downstream crates be able to implement this trait?

---

## 3. Errors and `Option`

*Rust makes failure visible. Keep it that way.*

### 19. `Result` is the default

> *Failure is part of the signature, not an interruption.*

- **Adopt:** Return `Result<T, E>`. Use `thiserror` for libraries and `anyhow` at application boundaries. Keep error types meaningful.
- **Smell:** `Box<dyn Error>` in public APIs; stringly typed errors; `Result` ignored with `let _ =`.
- **Relearn:** What can fail here, and who decides the policy?

### 20. `?` is for propagation, not policy

> *Propagate with context; decide where you have enough information.*

- **Adopt:** Use `?` to pass errors upward, add context at boundaries, and handle errors where recovery is possible.
- **Smell:** `?` everywhere with no context; errors swallowed by a catch-all; handling policy buried in a low-level helper.
- **Relearn:** Who is responsible for handling this error?

### 21. Error types are API

> *A caller should be able to match on what went wrong.*

- **Adopt:** Design error enums with meaningful variants. Implement `Display`, `std::error::Error`, and `source`. Treat variants as semver-relevant.
- **Smell:** `Other(String)` as the only variant; breaking error variants in a patch release; errors with no source chain.
- **Relearn:** Can a caller match on this error and act on it?

### 22. Do not `unwrap` in libraries

> *`unwrap` is a claim that failure is impossible.*

- **Adopt:** Return an error or use `expect` with a reason only for a proven invariant. Test the failure path.
- **Smell:** `unwrap()` in library code; `expect("should never happen")` on user input; panics crossing an FFI boundary.
- **Relearn:** Is this failure impossible, or just unlikely?

### 23. `Option` is not an error

> *Absence and failure are different outcomes.*

- **Adopt:** Use `Option<T>` for “may be absent” and `Result<T, E>` for “may fail.” Do not collapse them into one type.
- **Smell:** `Option<Result<T, E>>`; `None` meaning “failed”; error information lost in a `map`/`and_then` chain.
- **Relearn:** Is this absent, or did it fail?

### 24. Panics are for bugs

> *A panic is a broken invariant, not a control-flow tool.*

- **Adopt:** Use `assert!` for preconditions, `debug_assert!` for internal invariants, and `panic!` only for programmer errors.
- **Smell:** Panicking on user input; `catch_unwind` as error handling; `unwrap` in a request handler.
- **Relearn:** Is this a bug or an expected failure?

---

## 4. Memory and Allocation

*Rust gives you control; it does not remove the cost.*

### 25. Stack by default, heap by necessity

> *Allocation is a cost, not a reflex.*

- **Adopt:** Keep small, fixed-size values on the stack. Box large or recursive values deliberately. Measure before assuming the heap is the problem.
- **Smell:** `Box<T>` around everything; huge arrays on the stack; allocation in a hot loop because it was convenient.
- **Relearn:** Does this value need heap allocation, and why?

### 26. `Box` is for indirection, not everything

> *Use `Box` to solve a shape problem, not to avoid thinking about ownership.*

- **Adopt:** Use `Box` for recursive types, large values, and trait objects. Use generics when static dispatch is enough; use `Rc`/`Arc` for sharing.
- **Smell:** `Box<dyn Trait>` where a generic would do; double indirection; boxing to satisfy a lifetime you could restructure.
- **Relearn:** What problem is this indirection solving?

### 27. `Vec` capacity is a performance decision

> *Every reallocation copies; reserve when you know the size.*

- **Adopt:** Use `Vec::with_capacity`, reuse buffers with `clear()`, and avoid collecting into temporaries in hot paths.
- **Smell:** Repeated `push` without capacity; `collect()` just to iterate; allocating a new `Vec` per request.
- **Relearn:** How many allocations does this loop make?

### 28. `Cow` is for conditional ownership

> *Borrow when you can; own when you must.*

- **Adopt:** Return `Cow<'a, str>` or `Cow<'a, [T]>` when a value is sometimes borrowed and sometimes owned. Keep the type simple.
- **Smell:** `Cow` everywhere as a complexity tax; `String` returned for static data; cloning just to satisfy a return type.
- **Relearn:** Does this value need ownership, or only sometimes?

### 29. Arenas for phase-scoped data

> *Match the allocator to the lifetime.*

- **Adopt:** Use a bump allocator such as `bumpalo` for request-scoped or compiler-style data that is freed all at once.
- **Smell:** An arena for long-lived data; lifetime complexity that outweighs the allocation savings; arenas crossing async boundaries without a plan.
- **Relearn:** What is the lifetime of this allocation, and can it be bulk-freed?

### 30. Leaks are safe, rarely acceptable

> *`mem::forget` is safe; that does not make it free.*

- **Adopt:** Use `Box::leak`, `mem::forget`, and `ManuallyDrop` only with a documented reason. Prefer owning values or `Arc` over leaks.
- **Smell:** Leaking to satisfy `'static`; forgetting destructors; memory growth that “does not matter” until it does.
- **Relearn:** What is leaked, how much, and for how long?

---

## 5. Unsafe and FFI

*`unsafe` does not mean “dangerous”; it means “the compiler cannot prove this.” You must.*

### 31. `unsafe` is a proof obligation

> *Every `unsafe` block needs a reason the compiler cannot check.*

- **Adopt:** Write a `// SAFETY:` comment for every block: which invariant holds, why it holds, and what would break it.
- **Smell:** `unsafe` with no comment; “trust me”; copying unsafe code without understanding its preconditions.
- **Relearn:** What invariant makes this sound, and what breaks it?

### 32. Minimize the unsafe surface

> *One small safe wrapper beats unsafe scattered everywhere.*

- **Adopt:** Put `unsafe` behind a safe abstraction, keep blocks small, and test the abstraction’s contract.
- **Smell:** `unsafe` in business logic; unsafe blocks copied across modules; safe APIs that can still cause UB through misuse.
- **Relearn:** Can this be safe outside one module?

### 33. Document safety invariants

> *The caller needs to know what you need them to guarantee.*

- **Adopt:** Document aliasing, alignment, initialization, lifetime, and thread-safety requirements. Use `# Safety` sections in rustdoc.
- **Smell:** Safety docs that restate the code; missing preconditions; `unsafe fn` with no `# Safety`.
- **Relearn:** What must the caller guarantee for this to be sound?

### 34. FFI boundaries are contracts

> *Across the boundary, Rust’s guarantees stop.*

- **Adopt:** Define ownership, allocation, error, and panic behavior across FFI. Use `extern "C"`, `#[no_mangle]`, and explicit conversion types.
- **Smell:** Passing Rust types with no layout guarantee; unwinding across FFI; freeing memory in the wrong allocator.
- **Relearn:** Who allocates, who frees, and what happens on panic?

### 35. `repr(C)` and layout

> *If another language reads it, the layout must be guaranteed.*

- **Adopt:** Use `#[repr(C)]` or `#[repr(transparent)]` at FFI boundaries. Check `size_of`, `align_of`, and field order. Do not assume `repr(Rust)` layout.
- **Smell:** Transmuting structs; assuming field order; sending `bool` or enums across FFI without a defined representation.
- **Relearn:** Is this layout guaranteed by the language, or only by today’s compiler?

### 36. Miri and sanitizers

> *Unsafe code needs a tool that checks the rules you cannot see.*

- **Adopt:** Run `cargo miri test` for UB, ASan/TSan for FFI and concurrency, and fuzz unsafe parsers.
- **Smell:** Unsafe code tested only on the happy path; no Miri in CI; UB hidden behind passing tests.
- **Relearn:** Which tool would catch the class of UB I fear?

---

## 6. Concurrency

*Rust prevents data races, not deadlocks, starvation, or bad designs.*

### 37. `Send` and `Sync` are contracts

> *Auto traits encode what may cross a thread boundary.*

- **Adopt:** Understand why a type is `Send` or `Sync`. Use `PhantomData` to opt out. Never `unsafe impl Send` without a written proof.
- **Smell:** `unsafe impl Send` to silence an error; `Rc` across threads; `Sync` assumed because a type is immutable.
- **Relearn:** What makes this type safe to send or share?

### 38. Prefer channels to shared state

> *Send ownership; do not share mutation.*

- **Adopt:** Use `std::sync::mpsc`, `crossbeam`, or `flume` to move messages. Keep mutable shared state small and explicit.
- **Smell:** A global `Mutex<HashMap<...>>`; lock contention; threads mutating the same structure.
- **Relearn:** Can I send the data instead of sharing it?

### 39. `Arc<Mutex<T>>` is not a default

> *Shared mutation is a design choice with a cost.*

- **Adopt:** Use `Arc<Mutex<T>>` when shared mutation is required, `RwLock` for read-heavy access, and message passing when it is not.
- **Smell:** `Arc<Mutex<Vec<T>>>` everywhere; long critical sections; locking in async code without care.
- **Relearn:** Who needs to mutate this, and how often?

### 40. Atomics need ordering

> *Atomicity is not the same as visibility.*

- **Adopt:** Choose `Relaxed`, `Acquire`, `Release`, or `SeqCst` deliberately. Document the ordering each access requires.
- **Smell:** `SeqCst` everywhere “to be safe”; `Relaxed` used for publication; hand-rolled fences without a model.
- **Relearn:** What ordering does this access require, and what breaks if I weaken it?

### 41. Scoped threads beat `'static` clones

> *If the thread ends before the scope, it can borrow.*

- **Adopt:** Use `std::thread::scope` to borrow stack data safely. Avoid `Arc` clones and `'static` bounds when the scope already proves the lifetime.
- **Smell:** Cloning `Arc` just to spawn a short-lived thread; `'static` everywhere; detached threads that outlive their data.
- **Relearn:** Does this thread need to outlive the current scope?

### 42. Rayon for data parallelism

> *Use the right parallelism for the workload.*

- **Adopt:** Use `rayon` for CPU-bound, divisible work. Keep I/O on async or dedicated threads. Measure before adding parallelism.
- **Smell:** Rayon for I/O; nested parallelism causing oversubscription; parallelizing a workload that is already memory-bound.
- **Relearn:** Is this workload CPU-bound and divisible?

---

## 7. Async and Runtimes

*Async is a scheduling model, not a magic performance switch.*

### 43. Async is for I/O concurrency

> *Use async to wait on many things, not to compute faster.*

- **Adopt:** Use async for network and disk waits. Use threads or `rayon` for CPU-bound work. Keep the distinction visible in the API.
- **Smell:** Async for CPU-heavy loops; one task per core; async used because it is fashionable.
- **Relearn:** Is this workload waiting or computing?

### 44. Do not block the executor

> *A blocked task blocks every task on that worker.*

- **Adopt:** Use `spawn_blocking` or a dedicated thread for blocking I/O and CPU work. Keep tasks short between `.await` points.
- **Smell:** `std::fs` or blocking HTTP in async code; long CPU loops; `Mutex` held across `.await`.
- **Relearn:** What is this task preventing the executor from doing?

### 45. Cancellation is a drop

> *Dropping a future cancels it; cleanup must survive that.*

- **Adopt:** Make futures cancel-safe. Use guards for cleanup and avoid partial writes that cannot be resumed.
- **Smell:** Cleanup only on success; partial state after cancellation; `select!` dropping a future mid-write.
- **Relearn:** What happens if this future is dropped mid-await?

### 46. `select!` and timeouts

> *Every await needs a bound, and every race needs a policy.*

- **Adopt:** Wrap operations in `tokio::time::timeout`. Use `select!` for racing, and decide what happens to the losing branch.
- **Smell:** No timeouts; `select!` without an `else` branch; cancelled work that leaves state behind.
- **Relearn:** What is the deadline, and what happens when it fires?

### 47. Structured concurrency with `JoinSet`

> *Every task has an owner and a shutdown path.*

- **Adopt:** Use `JoinSet` or `TaskTracker` to own tasks. Await them on shutdown. Avoid detached `tokio::spawn` unless the task is truly independent.
- **Smell:** Fire-and-forget `tokio::spawn`; tasks outliving the request; shutdown that waits forever.
- **Relearn:** Who owns this task, and when is it joined?

### 48. Runtime choice is an API decision

> *A library should not force a runtime on its users.*

- **Adopt:** Keep libraries runtime-agnostic where possible. Choose Tokio or async-std at the application edge, and document the choice.
- **Smell:** Tokio types in a public API; multiple runtimes in one process; runtime-specific behavior leaking into a generic crate.
- **Relearn:** Is this crate tied to a runtime, and should it be?

---

## 8. Performance and Iterators

*The optimizer is not a promise. The cost model is your responsibility.*

### 49. Iterators are zero-cost until they aren’t

> *An iterator chain is an abstraction over a loop, not a guarantee about the loop.*

- **Adopt:** Prefer iterator chains for clarity, then check the generated code or benchmark when the path is hot. Watch for `Box<dyn Iterator>`, closures that capture heavily, and chains that hide allocation.
- **Smell:** “Iterators are always zero-cost” used to skip measurement; `dyn Iterator` in a tight loop; a chain that is rewritten by hand for speed without a profile.
- **Relearn:** What machine code did this chain become, and did I look?

### 50. Avoid needless `collect`

> *Collecting is a materialization decision, not punctuation.*

- **Adopt:** Chain lazily and consume once. Collect only when you need random access, reuse, or a boundary. Use `collect::<Result<Vec<_>, _>>()` deliberately.
- **Smell:** `collect()` followed immediately by `iter()`; collecting into a `Vec` just to call `.len()`; multiple passes over materialized data that could be one pass.
- **Relearn:** Does this data need to exist all at once, or can it flow?

### 51. Bounds checks are usually cheap; measure

> *Safety has a cost, but often less than the folklore claims.*

- **Adopt:** Write indexed code clearly first. If a bounds check shows up in a profile, restructure with iterators, slices, or `get_unchecked` behind a proven invariant.
- **Smell:** `unsafe { get_unchecked }` before profiling; assuming every check is a bottleneck; removing checks without a `// SAFETY:` proof.
- **Relearn:** Did the profiler blame the check, or did I?

### 52. `&[u8]` and `Bytes` for I/O

> *Move bytes without copying them.*

- **Adopt:** Accept `&[u8]` and `impl AsRef<[u8]>` for parsing. Use `bytes::Bytes` for cheaply cloned, shared buffers in async and network code. Avoid `Vec<u8>` copies at every layer.
- **Smell:** `Vec<u8>` cloned per handler; `String` for binary data; converting between `&str` and `&[u8]` repeatedly.
- **Relearn:** How many times are these bytes copied, and who owns them?

### 53. SIMD and target features

> *Vectorization is a portability decision as much as a performance one.*

- **Adopt:** Let the compiler auto-vectorize clean loops first. Use `std::simd` (nightly) or `wide`/`std::arch` when measured. Gate target features with `#[target_feature]` and runtime detection.
- **Smell:** Hand-written SIMD before a scalar baseline; `target_feature` without a runtime check; code that only builds on one CPU.
- **Relearn:** Which CPUs must run this, and how is the feature selected?

### 54. Profile before optimizing

> *Intuition about performance is a hypothesis, not a measurement.*

- **Adopt:** Use `perf`, `cargo flamegraph`, `criterion`, and `iai-callgrind`. Establish a baseline, change one thing, and compare distributions, not single runs.
- **Smell:** Optimizing the code that “looks slow”; micro-benchmarks with no realistic workload; claiming a win without a before-and-after.
- **Relearn:** What did the measurement say, and what did I change?

---

## 9. API and Trait Design

*A public API is a contract the compiler cannot fully enforce.*

### 55. Accept borrowed, return owned

> *Be liberal in what you accept and explicit in what you return.*

- **Adopt:** Take `&str`, `&[T]`, and `impl AsRef<T>` on the way in. Return `String`, `Vec<T>`, or an owned type when the caller must keep it. Use `Cow` only when the condition is real.
- **Smell:** `&String` and `&Vec<T>` parameters; returning references tied to internal state; forcing callers to clone before every call.
- **Relearn:** Does the caller need to keep this value, or only look at it?

### 56. Generics for zero-cost, `dyn` for flexibility

> *Static dispatch is fast and viral; dynamic dispatch is flexible and indirect.*

- **Adopt:** Use generics for hot paths and small, known type sets. Use `dyn Trait` for heterogeneous collections, plugin boundaries, and reducing monomorphization. Measure compile time and binary size too.
- **Smell:** Generic explosion across a large API; `dyn` in a tight numeric loop; choosing one without naming the tradeoff.
- **Relearn:** Am I optimizing for runtime, compile time, or extensibility?

### 57. Traits should be small and composable

> *A trait is a role, not a bag of unrelated abilities.*

- **Adopt:** Prefer several focused traits with blanket impls over one large trait. Use associated types for one-per-impl relationships and generic parameters for many-per-impl.
- **Smell:** God traits with twenty methods; default methods that assume implementation details; traits that exist only to group functions.
- **Relearn:** What single role does this trait describe?

### 58. Semver is a promise

> *Every public item is part of the contract, including error variants and trait impls.*

- **Adopt:** Treat public types, trait methods, enum variants, and feature names as semver-relevant. Use `cargo-semver-checks` in CI. Add variants only with care.
- **Smell:** Breaking changes in a patch release; `#[non_exhaustive]` added too late; “it’s just an internal type” that is actually public.
- **Relearn:** What did I promise to downstream crates, and did this change break it?

### 59. Feature flags are a compatibility surface

> *Every feature combination is a crate you have not tested.*

- **Adopt:** Keep features additive, avoid mutually exclusive flags, and test the combinations you claim to support. Document what each feature enables and its default.
- **Smell:** Features that change public types; default features pulling heavy dependencies; combinations that fail to compile.
- **Relearn:** Which feature combinations do I actually support, and where is that tested?

### 60. Documentation is part of the API

> *If the docs do not state the contract, the contract is whatever the code happens to do.*

- **Adopt:** Write crate-level docs with examples, `# Errors`, `# Panics`, and `# Safety` sections. Use doctests as executable examples. Document invariants and complexity.
- **Smell:** Missing `# Safety` on `unsafe fn`; examples that no longer compile; docs that describe the implementation instead of the contract.
- **Relearn:** Could a new user use this correctly without reading the source?

---

## 10. Testing, Debugging, and Verification

*Tests are evidence, not decoration.*

### 61. Unit tests next to code

> *Test the unit where it is defined, including private behavior.*

- **Adopt:** Use `#[cfg(test)] mod tests` beside the code. Test invariants, edge cases, and error paths, not just the happy path.
- **Smell:** Tests only in a distant integration suite; no test for the error branch; tests that assert implementation details.
- **Relearn:** Which invariant does this test prove, and where does it live?

### 62. Integration tests at crate boundary

> *Test the API the user actually sees.*

- **Adopt:** Put tests in `tests/` and exercise the public API. Use `trybuild` for compile-fail tests and doc tests for examples.
- **Smell:** Only unit tests, never a public-API test; integration tests reaching into private modules; examples that are not tested.
- **Relearn:** Does this test use the crate the way a caller would?

### 63. Property tests and fuzzing

> *Example tests check the cases you thought of; properties check the ones you did not.*

- **Adopt:** Use `proptest` or `quickcheck` for invariants, and `cargo-fuzz`/`libFuzzer` for parsers and unsafe boundaries. Keep failing cases as regression tests.
- **Smell:** Only hand-picked inputs; fuzzing without a corpus; property tests that restate the implementation.
- **Relearn:** What property must hold for all inputs, and what generated the counterexample?

### 64. Miri, loom, and sanitizers

> *Some bugs are invisible to ordinary tests.*

- **Adopt:** Run `cargo miri test` for undefined behavior, `loom` for concurrency models, and ASan/TSan for FFI and threaded code. Put them in CI where they matter.
- **Smell:** Unsafe code tested only on one platform; concurrency “verified” by a passing test run; sanitizers run once and never again.
- **Relearn:** Which tool would catch the bug class I cannot see?

### 65. `dbg!`, `tracing`, and backtraces

> *Debugging is a skill of asking the system what it did.*

- **Adopt:** Use `dbg!` locally, `tracing` for structured events, and `RUST_BACKTRACE=1` with `panic = "abort"` considered. Keep spans across async boundaries.
- **Smell:** `println!` debugging left in production; logs without request IDs; backtraces disabled in release with no alternative.
- **Relearn:** What evidence would tell me what actually happened?

### 66. Reproduce, reduce, bisect

> *A bug you cannot reproduce is a bug you do not yet understand.*

- **Adopt:** Capture the exact input, environment, and version. Reduce to the smallest failing case. Use `git bisect` and `cargo bisect-rustc` when the compiler or dependency changed.
- **Smell:** Fixing by guesswork; “cannot reproduce” closing the issue; changing several things at once.
- **Relearn:** What is the smallest input that still fails?

---

## 11. Tooling, Build, and Ecosystem

*The build is part of the program.*

### 67. Cargo is the build contract

> *If it is not in `Cargo.toml`, it is not reproducible.*

- **Adopt:** Keep dependencies, features, profiles, and metadata explicit. Commit `Cargo.lock` for binaries, use workspaces for multi-crate repos, and document build requirements.
- **Smell:** Hand-edited build scripts that bypass Cargo; floating dependency versions; a lockfile that is never updated or always churning.
- **Relearn:** Can a new machine build this from the repository alone?

### 68. Clippy and rustfmt are defaults

> *Style is not a debate; it is a baseline.*

- **Adopt:** Run `cargo fmt --check` and `cargo clippy -- -D warnings` in CI. Fix lints or allow them with a reason. Keep the toolchain pinned.
- **Smell:** Formatting-only diffs; dozens of ignored lints; `#[allow(clippy::all)]` at the crate root.
- **Relearn:** Which lint is telling me something about the design?

### 69. Pin and audit dependencies

> *Every dependency is code you ship and trust.*

- **Adopt:** Pin versions in `Cargo.lock`, run `cargo audit` and `cargo deny`, and review new dependencies for license, maintenance, and unsafe use.
- **Smell:** Dependencies added for one function; no audit in CI; transitive crates nobody has reviewed.
- **Relearn:** What does this dependency bring, and who maintains it?

### 70. MSRV is a promise

> *A minimum supported Rust version is a compatibility contract.*

- **Adopt:** Declare `rust-version` in `Cargo.toml`, test it in CI, and treat raising it as a semver-relevant change. Avoid newer syntax until the MSRV allows it.
- **Smell:** MSRV discovered by users; raising it in a patch release; CI testing only stable.
- **Relearn:** What is the oldest compiler this crate promises to support?

### 71. Reproducible builds and supply chain

> *The artifact should be the same tomorrow as it was today.*

- **Adopt:** Pin toolchains with `rust-toolchain.toml`, vendor or lock dependencies, and verify checksums. Consider `cargo-vet`, signed releases, and provenance.
- **Smell:** Builds that differ between machines; unpinned nightly; no record of what produced a release binary.
- **Relearn:** Could I rebuild this exact artifact and prove it matches?

### 72. Keep the dependency graph small

> *The best dependency is the one you did not add.*

- **Adopt:** Prefer the standard library and small, focused crates. Check `cargo tree` and binary size. Replace a heavy dependency with a few lines when the maintenance cost is lower.
- **Smell:** A framework for one feature; duplicate crates doing the same job; compile times dominated by transitive dependencies.
- **Relearn:** What is the smallest dependency set that solves this problem?

---

## Complete index

All 72 entries, grouped by domain.

- **0 Ownership and Borrowing:** 1 Borrow, don’t clone · 2 Move is a feature · 3 One owner at a time · 4 Split borrows instead of fighting the checker · 5 Interior mutability is a deliberate escape hatch · 6 `Drop` is part of the contract
- **1 Lifetimes and References:** 7 Lifetimes describe, not create · 8 Prefer owned data at boundaries · 9 Do not outlive your borrow · 10 `'static` is not “forever” · 11 Variance matters · 12 Self-referential types need a plan
- **2 Types and Invariants:** 13 Make illegal states unrepresentable · 14 Newtype everything meaningful · 15 Enums over boolean flags · 16 Parse, don’t validate · 17 Typestate for protocols · 18 Sealed traits for controlled extension
- **3 Errors and `Option`:** 19 `Result` is the default · 20 `?` is for propagation, not policy · 21 Error types are API · 22 Do not `unwrap` in libraries · 23 `Option` is not an error · 24 Panics are for bugs
- **4 Memory and Allocation:** 25 Stack by default, heap by necessity · 26 `Box` is for indirection, not everything · 27 `Vec` capacity is a performance decision · 28 `Cow` is for conditional ownership · 29 Arenas for phase-scoped data · 30 Leaks are safe, rarely acceptable
- **5 Unsafe and FFI:** 31 `unsafe` is a proof obligation · 32 Minimize the unsafe surface · 33 Document safety invariants · 34 FFI boundaries are contracts · 35 `repr(C)` and layout · 36 Miri and sanitizers
- **6 Concurrency:** 37 `Send` and `Sync` are contracts · 38 Prefer channels to shared state · 39 `Arc<Mutex<T>>` is not a default · 40 Atomics need ordering · 41 Scoped threads beat `'static` clones · 42 Rayon for data parallelism
- **7 Async and Runtimes:** 43 Async is for I/O concurrency · 44 Do not block the executor · 45 Cancellation is a drop · 46 `select!` and timeouts · 47 Structured concurrency with `JoinSet` · 48 Runtime choice is an API decision
- **8 Performance and Iterators:** 49 Iterators are zero-cost until they aren’t · 50 Avoid needless `collect` · 51 Bounds checks are usually cheap; measure · 52 `&[u8]` and `Bytes` for I/O · 53 SIMD and target features · 54 Profile before optimizing
- **9 API and Trait Design:** 55 Accept borrowed, return owned · 56 Generics for zero-cost, `dyn` for flexibility · 57 Traits should be small and composable · 58 Semver is a promise · 59 Feature flags are a compatibility surface · 60 Documentation is part of the API
- **10 Testing, Debugging, and Verification:** 61 Unit tests next to code · 62 Integration tests at crate boundary · 63 Property tests and fuzzing · 64 Miri, loom, and sanitizers · 65 `dbg!`, `tracing`, and backtraces · 66 Reproduce, reduce, bisect
- **11 Tooling, Build, and Ecosystem:** 67 Cargo is the build contract · 68 Clippy and rustfmt are defaults · 69 Pin and audit dependencies · 70 MSRV is a promise · 71 Reproducible builds and supply chain · 72 Keep the dependency graph small

---

## Self-audit

Score each line honestly. “Not sure” counts as no.

- [ ] I can explain who owns the data at every public API boundary.
- [ ] I have removed a `.clone()` in the last month for a design reason.
- [ ] My public types make invalid states hard to construct.
- [ ] Every fallible function returns `Result`, and callers can match on errors.
- [ ] I do not `unwrap()` in library or request-handling code.
- [ ] Every `unsafe` block has a `// SAFETY:` comment that names the invariant.
- [ ] I know which `Send`/`Sync` impls are unsafe, and why they are sound.
- [ ] I can name the ordering on every atomic access I wrote.
- [ ] No async task blocks the executor on file, network, or CPU work.
- [ ] I know what happens to each future when it is dropped.
- [ ] I have profiled the hot path instead of guessing.
- [ ] My crate’s public API has documented errors, panics, and safety contracts.
- [ ] My CI runs formatting, clippy, tests, and at least one of Miri, loom, or a sanitizer.
- [ ] I know my MSRV and test it.
- [ ] I can rebuild the last release artifact from the repository.

**Reading the score:** 13–15 yes: pick one “not sure” and close it. 8–12: choose three entries from different domains and make them habits. 0–7: start with the ten in [Start here](#start-here) and add one domain per week.

---

## Review lens

Use this when reviewing a Rust pull request. It is a checklist of questions, not a style guide.

| Layer | Ask | Entries |
| --- | --- | --- |
| **Ownership** | Who owns this value, and is the clone justified? | 1–6 |
| **Lifetimes** | What keeps this reference alive, and does the API force a lifetime on callers? | 7–12 |
| **Types** | Can an invalid state be constructed? Is the meaning in the type? | 13–18 |
| **Errors** | Can the caller distinguish absence from failure and act on it? | 19–24 |
| **Memory** | How many allocations per call, and what is their lifetime? | 25–30 |
| **Unsafe** | What invariant makes this sound, and what tests it? | 31–36 |
| **Concurrency** | What is shared, what is sent, and what ordering is required? | 37–42 |
| **Async** | What blocks, what is cancelled, and who owns the task? | 43–48 |
| **Performance** | What did the profiler say, and what does the abstraction cost? | 49–54 |
| **API** | What is promised to downstream crates, and is it semver-safe? | 55–60 |
| **Verification** | Which failure path is tested, and would a mutant survive? | 61–66 |
| **Build** | Is the build pinned, linted, audited, and reproducible? | 67–72 |

A useful review comment names the mentality and the concrete risk:

> “This `Arc<Mutex<Vec<_>>>` is shared mutation by default (#39). Only the metrics task writes it; can we send updates over a channel instead (#38)?”

---

## Anti-patterns

The fastest way to lose a good Rust habit is to keep a comforting one nearby.

| Anti-pattern | What it sounds like | Replacement |
| --- | --- | --- |
| **Clone to compile** | “Just add `.clone()`.” | “Who should own this, and why is borrowing hard?” (#1, #2) |
| **`Rc<RefCell<T>>` by default** | “It works.” | “Is shared mutation actually required?” (#3, #5) |
| **Boolean state soup** | “We check the flags.” | “Model the states as an enum.” (#15) |
| **Stringly typed errors** | “It returned an error.” | “Can the caller match on the failure?” (#21) |
| **`unwrap()` in production** | “It can’t fail.” | “Prove it, or return an error.” (#22) |
| **Unsafe without a proof** | “Trust me.” | “Write the `// SAFETY:` invariant.” (#31) |
| **`Arc<Mutex<T>>` everywhere** | “Shared state is simple.” | “Send ownership instead of sharing mutation.” (#38, #39) |
| **Blocking the executor** | “It’s just one file read.” | “Use `spawn_blocking` or a dedicated thread.” (#44) |
| **Fire-and-forget tasks** | “It runs in the background.” | “Who joins it on shutdown?” (#47) |
| **Collect-then-iterate** | “It’s clearer.” | “Does this data need to exist all at once?” (#50) |
| **Optimize by intuition** | “This looks slow.” | “What did the profiler measure?” (#54) |
| **Semver optimism** | “It’s only a small change.” | “What did I promise downstream?” (#58) |
| **Dependency reflex** | “There’s a crate for that.” | “What does it bring, and who maintains it?” (#72) |

---

## How to share

Pick the format that fits your team, not the one that feels most impressive.

- **15-minute brown bag:** one mentality, one Rust example from your own crate, one question at the end.
- **Review comments:** cite the number. “This is #44” is faster than a paragraph and easier to remember.
- **Onboarding:** choose the ten mentalities that matter most for your codebase and make them required reading.
- **Postmortems:** map contributing factors to missing mentalities, then add the missing test or lint.
- **Lunch-and-learn series:** one domain per week, cycling through the stack, then repeat with new examples.

---

## Four-week plan

One domain cluster per week. Each week ends with an artifact, not a feeling.

### Week 1 — Ownership, lifetimes, types, errors

- Read entries 1–24. Pick one public API and one error type.
- Remove one unjustified clone; make one invalid state unrepresentable; add one error test.
- **Artifact:** a diff that makes ownership or failure clearer.

### Week 2 — Memory, unsafe, concurrency

- Read entries 25–42. Pick one allocation hotspot and one `unsafe` block.
- Measure allocations; add a `// SAFETY:` proof; run Miri or loom on the risky path.
- **Artifact:** a measured allocation change or a documented safety proof.

### Week 3 — Async, performance, API design

- Read entries 43–60. Pick one async task, one hot path, and one public trait.
- Add a timeout or cancellation test; profile the hot path; check the API against semver.
- **Artifact:** a profile result plus one API decision recorded.

### Week 4 — Testing, debugging, tooling

- Read entries 61–72. Pick one untested failure path and one build promise.
- Add a property or fuzz test; run clippy and audit; verify the MSRV and lockfile.
- **Artifact:** a CI change that would have caught a real bug.

---

## Further reading

**Language and ownership**

- *The Rust Programming Language*, Klabnik and Nichols — the canonical starting point.
- *Programming Rust*, Blandy, Orendorff, and Tindall — systems-oriented coverage of ownership and performance.
- *Rust for Rustaceans*, Jon Gjengset — idiomatic design, traits, async, and unsafe.
- *Effective Rust*, David Drysdale — focused advice with concrete examples.

**Unsafe, concurrency, and async**

- *The Rustonomicon* — the unsafe code guidelines every `unsafe` author should read.
- *Rust Atomics and Locks*, Mara Bos — memory ordering and concurrent data structures.
- *Async Rust*, Maxwell Flitton and Caroline Morton — futures, runtimes, and practical async design.
- The Miri and loom documentation — executable checks for UB and concurrency.

**API design and ecosystem**

- *Rust API Guidelines* — naming, traits, documentation, and semver.
- *Zero to Production in Rust*, Luca Palmieri — building and operating a real service.
- The Cargo Book, the Rust Reference, and `cargo-semver-checks` documentation.

---

## Contributing

- Keep entries **Rust-specific**: name the type, crate, lint, or tool.
- Keep the four-part shape: mantra, **Adopt**, **Smell**, **Relearn**.
- Prefer one concrete practice over three vague ones.
- If you add an entry, update the domain range, the complete index, and the self-audit.
- If you add a domain, update the layer map, the start-here set, and the four-week plan.

The general, language-agnostic catalog lives in [GENERAL.md](GENERAL.md).
