# Rust Programming Mentalities

> Habits of mind for writing Rust that is correct, idiomatic, fast, and maintainable.

Rust is not just “C with a borrow checker.” It is a language with its own design pressure: ownership replaces manual memory management, `Result` replaces exceptions, traits replace inheritance, and `unsafe` is a proof obligation rather than a permission slip. These mentalities are written **for Rust specifically** — each one names Rust types, crates, tools, and failure modes.

Entries are grouped by domain, not ranked. Read the domain you touch most, then follow the boundaries into the layers around it.

---

## Contents

**Foundations:** [Ownership and Borrowing](#ownership-and-borrowing) · [Lifetimes and References](#lifetimes-and-references) · [Types and Invariants](#types-and-invariants) · [Errors and `Option`](#errors-and-option) · [Memory and Allocation](#memory-and-allocation)

**Systems:** [Unsafe and FFI](#unsafe-and-ffi) · [Concurrency](#concurrency) · [Async and Runtimes](#async-and-runtimes) · [Performance and Iterators](#performance-and-iterators) · [Embedded and Systems Constraints](#embedded-and-systems-constraints)

**Design:** [API and Trait Design](#api-and-trait-design) · [Macros and Metaprogramming](#macros-and-metaprogramming) · [Serialization and Data Modeling](#serialization-and-data-modeling) · [Networking and I/O](#networking-and-io) · [Security and Cryptography](#security-and-cryptography)

**Practice:** [Testing, Debugging, and Verification](#testing-debugging-and-verification) · [Tooling, Build, and Ecosystem](#tooling-build-and-ecosystem) · [Observability and Operations](#observability-and-operations) · [Project Structure and Maintenance](#project-structure-and-maintenance)

**Reference:** [Index](#index) · [Self-audit](#self-audit) · [Review lens](#review-lens) · [Anti-patterns](#anti-patterns) · [How to share](#how-to-share) · [Four-week plan](#a-four-week-plan) · [Further reading](#further-reading) · [Contributing](#contributing)

---

## How to use

| Mode | What it looks like | Cadence |
| --- | --- | --- |
| **Adopt** | Pick one mentality. Apply it to a real crate or pull request. | One per week |
| **Share** | Cite the entry by name in review, or run a 15-minute brown bag. | One per sprint |
| **Relearn** | Answer the relearn question from memory, then teach it. | One per month |

Three rules:

1. **One mentality at a time.** Depth changes code; breadth changes nothing.
2. **A mentality without a diff is a slogan.** Every entry must alter a real decision.
3. **If you cannot name the Rust cost, you have not adopted it.** “It compiles” is not an argument.

Each entry has four parts: a mantra, a practice to adopt, a smell that means it is missing, and a question that exposes whether you understand it. Examples assume stable Rust; nightly-only features are called out explicitly.

---

## Domain map

Read **bottom-up** to go from ownership to ecosystem. Read **top-down** to trace a compile error or production bug to its design cause.

| Domain | Core question |
| --- | --- |
| [Ownership and Borrowing](#ownership-and-borrowing) | Who owns this, and who may borrow it? |
| [Lifetimes and References](#lifetimes-and-references) | What keeps this data alive? |
| [Types and Invariants](#types-and-invariants) | Can the wrong thing even compile? |
| [Errors and `Option`](#errors-and-option) | Is this absent, failed, or a bug? |
| [Memory and Allocation](#memory-and-allocation) | Who allocates, and when is it freed? |
| [Unsafe and FFI](#unsafe-and-ffi) | What invariant makes this sound? |
| [Concurrency](#concurrency) | What can run at the same time, and what is shared? |
| [Async and Runtimes](#async-and-runtimes) | What happens while this task waits? |
| [Performance and Iterators](#performance-and-iterators) | What does this abstraction cost? |
| [Embedded and Systems Constraints](#embedded-and-systems-constraints) | What does the platform not give me? |
| [API and Trait Design](#api-and-trait-design) | What have I promised to callers? |
| [Macros and Metaprogramming](#macros-and-metaprogramming) | Is this syntax or code generation? |
| [Serialization and Data Modeling](#serialization-and-data-modeling) | What have I promised to every reader? |
| [Networking and I/O](#networking-and-io) | What happens when the peer misbehaves? |
| [Security and Cryptography](#security-and-cryptography) | Who can reach it, and what stops them? |
| [Testing, Debugging, and Verification](#testing-debugging-and-verification) | How do I know it works? |
| [Tooling, Build, and Ecosystem](#tooling-build-and-ecosystem) | Can I reproduce and trust the build? |
| [Observability and Operations](#observability-and-operations) | Can I see what it is doing in production? |
| [Project Structure and Maintenance](#project-structure-and-maintenance) | Can someone else change it safely? |

---

## Start here

If the catalog feels large, begin with these. They prevent the most common Rust design mistakes.

**Borrow, don’t clone** · **Make illegal states unrepresentable** · **`Result` is the default** · **`unsafe` is a proof obligation** · **`Send` and `Sync` are contracts** · **Do not block the executor** · **Profile before optimizing** · **Accept borrowed, return owned** · **Miri, loom, and sanitizers** · **Treat the network as unreliable**

Then add the domain you touch most: ownership, async, unsafe, embedded, or API design.

---

## Ownership and Borrowing

*The borrow checker is not an obstacle; it is a design reviewer that never gets tired.*

### Borrow, don’t clone

> *A clone is a design decision, not a compiler appeasement.*

- **Adopt:** Pass `&T` or `&mut T`; clone only for isolation, ownership transfer, or measured simplicity. Name the reason in review.
- **Smell:** `.clone()` added to silence a borrow error; clones inside hot loops; cloning before a function that could take ownership.
- **Relearn:** Is this clone buying independence, or hiding an ownership design problem?

### Move is a feature

> *Transfer ownership; let the old binding die.*

- **Adopt:** Move values into functions and structs. Use `std::mem::take` or `replace` when you need to move out of a borrowed place.
- **Smell:** Cloning before passing to a function that could accept ownership; fighting move errors with `.clone()`.
- **Relearn:** Who should own this value after the call?

### One owner at a time

> *Shared ownership is a tool, not a default.*

- **Adopt:** Model ownership in structs. Use `Rc` or `Arc` only when shared ownership is genuinely required; prefer borrowing otherwise.
- **Smell:** `Rc<RefCell<T>>` everywhere; `Arc<Mutex<T>>` for data only one task touches.
- **Relearn:** Does this value need shared ownership, or just a borrow?

### Split borrows instead of fighting the checker

> *If two fields are independent, borrow them independently.*

- **Adopt:** Destructure structs into disjoint fields; write methods that borrow only the fields they need; use free functions when `&mut self` is too broad.
- **Smell:** A `&mut self` method that needs two fields and fails to compile; reaching for `RefCell` to work around field-level borrowing.
- **Relearn:** Which fields are actually borrowed at the same time?

### Interior mutability is a deliberate escape hatch

> *`Cell`, `RefCell`, `Mutex`, and atomics change the rules — say why.*

- **Adopt:** Use `Cell`/`RefCell` for single-threaded interior mutation, `Mutex`/`RwLock`/atomics for shared state, and keep the escape hatch encapsulated.
- **Smell:** `RefCell` runtime panics in production; `Mutex` around single-threaded data; interior mutability leaking through public APIs.
- **Relearn:** Why can’t this be `&mut`, and what runtime cost am I accepting?

### `Drop` is part of the contract

> *Cleanup happens on every path, including the ones you did not write.*

- **Adopt:** Use RAII guards for files, locks, and transactions. Implement `Drop` for resources, but do not rely on it for async cleanup or fallible work.
- **Smell:** Manual `close()` on the happy path; leaked file descriptors; `Drop` that panics; `mem::forget` used casually.
- **Relearn:** What happens when this value is dropped, and on which paths?

### Shared ownership is a last resort

> *An `Arc` is a lifetime decision disguised as a pointer.*

- **Adopt:** Prefer a single owner plus borrowing. Introduce `Arc` only at a true sharing boundary, and document why the owner cannot be singular.
- **Smell:** `Arc` in every struct; `Rc` cycles that never drop; strong counts that only grow.
- **Relearn:** Could a single owner and a reference replace this `Arc`?

---

## Lifetimes and References

*Lifetimes describe relationships. They do not create lifetime.*

### Lifetimes describe, not create

> *`&'a T` means “borrow valid for `'a`,” not “make it live longer.”*

- **Adopt:** Fix the owner, not the annotation. Read lifetime errors as “this reference outlives its data,” then change the ownership or scope.
- **Smell:** Adding `'a` everywhere to silence the compiler; lifetime parameters that do not express a real relationship.
- **Relearn:** Which value actually owns the data, and how long does it live?

### Prefer owned data at boundaries

> *Borrow internally; own at the edge.*

- **Adopt:** Return `String` or `Vec<T>` from public APIs when the caller needs ownership. Accept `&str`, `&[T]`, and `impl AsRef<T>` on the way in.
- **Smell:** Returning `&str` tied to a temporary; lifetime gymnastics across module boundaries; `&String` and `&Vec<T>` parameters.
- **Relearn:** Does the caller need a borrow or ownership?

### Do not outlive your borrow

> *A reference is only as good as its owner.*

- **Adopt:** Keep the owner alive for the whole borrow. Use `Arc` or `Cow` when data must be shared or conditionally owned. Never return a reference to a local.
- **Smell:** `return &local`; dangling references hidden behind `unsafe`; storing references in long-lived structs without a plan.
- **Relearn:** What keeps this data alive while the reference exists?

### `'static` is not “forever”

> *`&'static T` and `T: 'static` are different promises.*

- **Adopt:** Distinguish a static borrow from a type that owns all its data. Use `Box::leak` only for genuinely process-lifetime data.
- **Smell:** `Box::leak` to satisfy `'static`; `'static` bounds added everywhere; leaking memory to spawn a task.
- **Relearn:** Does this need to live forever, or just own its data?

### Variance matters

> *Lifetimes and generics have subtyping rules; wrappers can change them.*

- **Adopt:** Understand covariance, contravariance, and invariance. Use `PhantomData` to control variance and auto traits deliberately.
- **Smell:** Unexpected lifetime errors in generic wrappers; `PhantomData` copied without understanding; `unsafe` variance assumptions.
- **Relearn:** What variance does this type have, and why?

### Self-referential types need a plan

> *A value that borrows itself cannot move casually.*

- **Adopt:** Prefer a redesign that avoids self-reference. If you need it, use `Pin`, `self_cell`, or `ouroboros` and understand the safety conditions.
- **Smell:** Hand-rolled self-referential structs; `Pin` used as a buzzword; unsafe code that assumes a stable address without enforcing it.
- **Relearn:** Can I avoid the self-reference, or do I need a pinned owner?

### Borrow from the narrowest scope

> *The shortest borrow is the easiest borrow.*

- **Adopt:** Scope borrows tightly with blocks; end a borrow before mutating; let non-lexical lifetimes do their job instead of cloning the world.
- **Smell:** A long-lived borrow that forces a clone of the whole struct; “cannot borrow as mutable” worked around by cloning.
- **Relearn:** What is the shortest scope that satisfies this borrow?

### Reborrow instead of moving

> *Lend the reference again; do not surrender it.*

- **Adopt:** Pass `&*x` or `&mut *x` to reborrow explicitly. Let the compiler insert reborrows when the types allow it.
- **Smell:** A reference moved and then needed again; cloning to keep using a value after passing it.
- **Relearn:** Do I need to move this reference, or just lend it again?

---

## Types and Invariants

*The cheapest bug is the one the compiler refuses to accept.*

### Make illegal states unrepresentable

> *Use the type system as the first test suite.*

- **Adopt:** Use enums for alternatives and structs for “all fields together.” Make invalid combinations impossible to construct; validate at the boundary.
- **Smell:** `Option` fields that must be `Some` together; `is_running` and `is_failed` booleans that can both be true; validation repeated in every caller.
- **Relearn:** Which combination of fields is nonsense, and why does the type allow it?

### Newtype everything meaningful

> *A raw `u64` does not say what it means.*

- **Adopt:** Use `struct UserId(u64)`, `struct Meters(f64)`, and `struct Email(String)`. Implement only the traits that make sense.
- **Smell:** Raw integers for every ID; unit mixups; `String` for every semantic string.
- **Relearn:** What does this primitive actually mean, and can the type say it?

### Enums over boolean flags

> *A state is not a bag of booleans.*

- **Adopt:** Model states as an enum and match exhaustively. Let the compiler tell you when a new variant needs handling.
- **Smell:** `is_running`, `is_paused`, `is_failed`; `match` with a catch-all that hides new variants.
- **Relearn:** What are the legal states, and what happens in each?

### Parse, don’t validate

> *Turn untrusted input into a type that cannot be wrong.*

- **Adopt:** Implement `TryFrom`, `FromStr`, or a constructor that returns `Result<Validated, Error>`. Pass the validated type inward.
- **Smell:** `validate()` returning `bool`; raw `String` passed through ten layers; checks repeated at every boundary.
- **Relearn:** What type proves this input is valid on the other side?

### Typestate for protocols

> *If a call is only valid in one state, encode the state in the type.*

- **Adopt:** Use marker types such as `Connecting`, `Connected`, and `Closed`. Consume `self` on transitions so invalid sequences cannot compile.
- **Smell:** Runtime checks for states the compiler could enforce; “you must call `start()` first” comments.
- **Relearn:** Can the compiler prevent this call in the wrong state?

### Sealed traits for controlled extension

> *A public trait is a promise about downstream implementations.*

- **Adopt:** Use a private supertrait to seal traits that are not meant to be implemented externally. Document the extension policy.
- **Smell:** Public traits that are de facto sealed; downstream impls blocking a breaking change; adding methods to widely implemented traits.
- **Relearn:** Should downstream crates be able to implement this trait?

### Const generics for compile-time shape

> *If the length is known at compile time, say so in the type.*

- **Adopt:** Use const generics for array lengths, fixed dimensions, and buffer sizes. Let the compiler reject mismatched shapes.
- **Smell:** Runtime length checks that could be const; `Vec<T>` where `[T; N]` would encode the invariant; dimension mixups caught by tests.
- **Relearn:** What part of this shape is known at compile time?

### `PhantomData` for type-level meaning

> *A marker type is a claim about meaning, not decoration.*

- **Adopt:** Use `PhantomData` to encode units, ownership, variance, and auto-trait behavior. Document what the marker means.
- **Smell:** `PhantomData` copied without understanding; variance bugs in wrappers; accidental `Send`/`Sync` leakage.
- **Relearn:** What does this marker change about the type, and why is it sound?

### Zero-sized types carry proof

> *A value that occupies no space can still prove something.*

- **Adopt:** Use zero-sized tokens as capabilities. Make their constructors private so holding one means the invariant was checked.
- **Smell:** A ZST token anyone can construct; a type-level “proof” with no invariant behind it.
- **Relearn:** What does holding this value prove, and who can create it?

---

## Errors and `Option`

*Rust makes failure visible. Keep it that way.*

### `Result` is the default

> *Failure is part of the signature, not an interruption.*

- **Adopt:** Return `Result<T, E>`. Use `thiserror` for libraries and `anyhow` at application boundaries. Keep error types meaningful.
- **Smell:** `Box<dyn Error>` in public APIs; stringly typed errors; `Result` ignored with `let _ =`.
- **Relearn:** What can fail here, and who decides the policy?

### `?` is for propagation, not policy

> *Propagate with context; decide where you have enough information.*

- **Adopt:** Use `?` to pass errors upward, add context at boundaries, and handle errors where recovery is possible.
- **Smell:** `?` everywhere with no context; errors swallowed by a catch-all; handling policy buried in a low-level helper.
- **Relearn:** Who is responsible for handling this error?

### Error types are API

> *A caller should be able to match on what went wrong.*

- **Adopt:** Design error enums with meaningful variants. Implement `Display`, `std::error::Error`, and `source`. Treat variants as semver-relevant.
- **Smell:** `Other(String)` as the only variant; breaking error variants in a patch release; errors with no source chain.
- **Relearn:** Can a caller match on this error and act on it?

### Do not `unwrap` in libraries

> *`unwrap` is a claim that failure is impossible.*

- **Adopt:** Return an error or use `expect` with a reason only for a proven invariant. Test the failure path.
- **Smell:** `unwrap()` in library code; `expect("should never happen")` on user input; panics crossing an FFI boundary.
- **Relearn:** Is this failure impossible, or just unlikely?

### `Option` is not an error

> *Absence and failure are different outcomes.*

- **Adopt:** Use `Option<T>` for “may be absent” and `Result<T, E>` for “may fail.” Do not collapse them into one type.
- **Smell:** `Option<Result<T, E>>`; `None` meaning “failed”; error information lost in a `map`/`and_then` chain.
- **Relearn:** Is this absent, or did it fail?

### Panics are for bugs

> *A panic is a broken invariant, not a control-flow tool.*

- **Adopt:** Use `assert!` for preconditions, `debug_assert!` for internal invariants, and `panic!` only for programmer errors.
- **Smell:** Panicking on user input; `catch_unwind` as error handling; `unwrap` in a request handler.
- **Relearn:** Is this a bug or an expected failure?

### Errors should carry context

> *“Failed” is not a diagnosis.*

- **Adopt:** Add context at boundaries with `thiserror` sources or `anyhow::Context`. Preserve the source chain and include the identifiers needed to investigate.
- **Smell:** An error that says only “failed”; the source chain discarded; context that repeats `Display` without adding information.
- **Relearn:** What would I need to debug this at 3 a.m.?

### Design for the caller’s recovery

> *An error the caller cannot act on is a log line with extra steps.*

- **Adopt:** Distinguish retryable from permanent failures, expose typed reasons, and document the recovery path. Avoid collapsing everything into one opaque variant.
- **Smell:** Every error fatal; the caller cannot tell whether to retry; an error enum with a single `Other` variant.
- **Relearn:** What can the caller do differently because of this error?

---

## Memory and Allocation

*Rust gives you control; it does not remove the cost.*

### Stack by default, heap by necessity

> *Allocation is a cost, not a reflex.*

- **Adopt:** Keep small, fixed-size values on the stack. Box large or recursive values deliberately. Measure before assuming the heap is the problem.
- **Smell:** `Box<T>` around everything; huge arrays on the stack; allocation in a hot loop because it was convenient.
- **Relearn:** Does this value need heap allocation, and why?

### `Box` is for indirection, not everything

> *Use `Box` to solve a shape problem, not to avoid thinking about ownership.*

- **Adopt:** Use `Box` for recursive types, large values, and trait objects. Use generics when static dispatch is enough; use `Rc`/`Arc` for sharing.
- **Smell:** `Box<dyn Trait>` where a generic would do; double indirection; boxing to satisfy a lifetime you could restructure.
- **Relearn:** What problem is this indirection solving?

### `Vec` capacity is a performance decision

> *Every reallocation copies; reserve when you know the size.*

- **Adopt:** Use `Vec::with_capacity`, reuse buffers with `clear()`, and avoid collecting into temporaries in hot paths.
- **Smell:** Repeated `push` without capacity; `collect()` just to iterate; allocating a new `Vec` per request.
- **Relearn:** How many allocations does this loop make?

### `Cow` is for conditional ownership

> *Borrow when you can; own when you must.*

- **Adopt:** Return `Cow<'a, str>` or `Cow<'a, [T]>` when a value is sometimes borrowed and sometimes owned. Keep the type simple.
- **Smell:** `Cow` everywhere as a complexity tax; `String` returned for static data; cloning just to satisfy a return type.
- **Relearn:** Does this value need ownership, or only sometimes?

### Arenas for phase-scoped data

> *Match the allocator to the lifetime.*

- **Adopt:** Use a bump allocator such as `bumpalo` for request-scoped or compiler-style data that is freed all at once.
- **Smell:** An arena for long-lived data; lifetime complexity that outweighs the allocation savings; arenas crossing async boundaries without a plan.
- **Relearn:** What is the lifetime of this allocation, and can it be bulk-freed?

### Leaks are safe, rarely acceptable

> *`mem::forget` is safe; that does not make it free.*

- **Adopt:** Use `Box::leak`, `mem::forget`, and `ManuallyDrop` only with a documented reason. Prefer owning values or `Arc` over leaks.
- **Smell:** Leaking to satisfy `'static`; forgetting destructors; memory growth that “does not matter” until it does.
- **Relearn:** What is leaked, how much, and for how long?

### Avoid hidden allocations in APIs

> *A signature should not hide a heap trip.*

- **Adopt:** Document when a function allocates. Accept caller-provided buffers, return iterators, and avoid temporary `Vec` or `String` construction in hot paths.
- **Smell:** “No allocation” claims with a hidden `collect`; an API that allocates on every call; a clone inside a getter.
- **Relearn:** How many allocations does one call make?

### Reuse buffers across requests

> *Capacity is an asset; do not throw it away.*

- **Adopt:** Pool buffers, `clear()` and reuse them, accept `&mut Vec<u8>`, and use `BytesMut` where shared slices are needed.
- **Smell:** A new `Vec` per request; capacity churn; allocating inside a loop that already has a buffer.
- **Relearn:** What can be reused instead of reallocated?

---

## Unsafe and FFI

*`unsafe` does not mean “dangerous”; it means “the compiler cannot prove this.” You must.*

### `unsafe` is a proof obligation

> *Every `unsafe` block needs a reason the compiler cannot check.*

- **Adopt:** Write a `// SAFETY:` comment for every block: which invariant holds, why it holds, and what would break it.
- **Smell:** `unsafe` with no comment; “trust me”; copying unsafe code without understanding its preconditions.
- **Relearn:** What invariant makes this sound, and what breaks it?

### Minimize the unsafe surface

> *One small safe wrapper beats unsafe scattered everywhere.*

- **Adopt:** Put `unsafe` behind a safe abstraction, keep blocks small, and test the abstraction’s contract.
- **Smell:** `unsafe` in business logic; unsafe blocks copied across modules; safe APIs that can still cause UB through misuse.
- **Relearn:** Can this be safe outside one module?

### Document safety invariants

> *The caller needs to know what you need them to guarantee.*

- **Adopt:** Document aliasing, alignment, initialization, lifetime, and thread-safety requirements. Use `# Safety` sections in rustdoc.
- **Smell:** Safety docs that restate the code; missing preconditions; `unsafe fn` with no `# Safety`.
- **Relearn:** What must the caller guarantee for this to be sound?

### FFI boundaries are contracts

> *Across the boundary, Rust’s guarantees stop.*

- **Adopt:** Define ownership, allocation, error, and panic behavior across FFI. Use `extern "C"`, `#[no_mangle]`, and explicit conversion types.
- **Smell:** Passing Rust types with no layout guarantee; unwinding across FFI; freeing memory in the wrong allocator.
- **Relearn:** Who allocates, who frees, and what happens on panic?

### `repr(C)` and layout

> *If another language reads it, the layout must be guaranteed.*

- **Adopt:** Use `#[repr(C)]` or `#[repr(transparent)]` at FFI boundaries. Check `size_of`, `align_of`, and field order. Do not assume `repr(Rust)` layout.
- **Smell:** Transmuting structs; assuming field order; sending `bool` or enums across FFI without a defined representation.
- **Relearn:** Is this layout guaranteed by the language, or only by today’s compiler?

### Miri and sanitizers

> *Unsafe code needs a tool that checks the rules you cannot see.*

- **Adopt:** Run `cargo miri test` for UB, ASan/TSan for FFI and concurrency, and fuzz unsafe parsers.
- **Smell:** Unsafe code tested only on the happy path; no Miri in CI; UB hidden behind passing tests.
- **Relearn:** Which tool would catch the class of UB I fear?

### `Pin` is a safety contract

> *Pinning is a promise about address stability, not a smart pointer.*

- **Adopt:** Treat `Pin` as a guarantee that a value will not move. Use `pin!` or `Box::pin`, document structural pinning, and never move pinned data.
- **Smell:** `Pin` used as a buzzword; moving data out of a pinned value; `Unpin` assumptions that are not justified.
- **Relearn:** What must never move, and who guarantees it?

### Never unwind across FFI

> *A panic crossing the boundary is undefined behavior waiting to happen.*

- **Adopt:** Catch panics at the boundary with `catch_unwind`, or use `extern "C-unwind"` only when both sides support unwinding. Choose a panic strategy deliberately.
- **Smell:** A panic crossing into C or C++; no panic strategy at the boundary; `catch_unwind` without checking the result.
- **Relearn:** What happens if this panics at the boundary?

### Wrap C in a safe Rust API

> *Callers should not need to remember the C rules.*

- **Adopt:** Keep raw pointers and `unsafe` inside one module. Expose safe types that encode ownership and free resources in `Drop`.
- **Smell:** Raw pointers in a public API; callers must remember to call `free`; duplicate unsafe wrappers around the same library.
- **Relearn:** Can a safe caller misuse this API?

---

## Concurrency

*Rust prevents data races, not deadlocks, starvation, or bad designs.*

### `Send` and `Sync` are contracts

> *Auto traits encode what may cross a thread boundary.*

- **Adopt:** Understand why a type is `Send` or `Sync`. Use `PhantomData` to opt out. Never `unsafe impl Send` without a written proof.
- **Smell:** `unsafe impl Send` to silence an error; `Rc` across threads; `Sync` assumed because a type is immutable.
- **Relearn:** What makes this type safe to send or share?

### Prefer channels to shared state

> *Send ownership; do not share mutation.*

- **Adopt:** Use `std::sync::mpsc`, `crossbeam`, or `flume` to move messages. Keep mutable shared state small and explicit.
- **Smell:** A global `Mutex<HashMap<...>>`; lock contention; threads mutating the same structure.
- **Relearn:** Can I send the data instead of sharing it?

### `Arc<Mutex<T>>` is not a default

> *Shared mutation is a design choice with a cost.*

- **Adopt:** Use `Arc<Mutex<T>>` when shared mutation is required, `RwLock` for read-heavy access, and message passing when it is not.
- **Smell:** `Arc<Mutex<Vec<T>>>` everywhere; long critical sections; locking in async code without care.
- **Relearn:** Who needs to mutate this, and how often?

### Atomics need ordering

> *Atomicity is not the same as visibility.*

- **Adopt:** Choose `Relaxed`, `Acquire`, `Release`, or `SeqCst` deliberately. Document the ordering each access requires.
- **Smell:** `SeqCst` everywhere “to be safe”; `Relaxed` used for publication; hand-rolled fences without a model.
- **Relearn:** What ordering does this access require, and what breaks if I weaken it?

### Scoped threads beat `'static` clones

> *If the thread ends before the scope, it can borrow.*

- **Adopt:** Use `std::thread::scope` to borrow stack data safely. Avoid `Arc` clones and `'static` bounds when the scope already proves the lifetime.
- **Smell:** Cloning `Arc` just to spawn a short-lived thread; `'static` everywhere; detached threads that outlive their data.
- **Relearn:** Does this thread need to outlive the current scope?

### Rayon for data parallelism

> *Use the right parallelism for the workload.*

- **Adopt:** Use `rayon` for CPU-bound, divisible work. Keep I/O on async or dedicated threads. Measure before adding parallelism.
- **Smell:** Rayon for I/O; nested parallelism causing oversubscription; parallelizing a workload that is already memory-bound.
- **Relearn:** Is this workload CPU-bound and divisible?

### Avoid lock ordering inversions

> *Deadlock is a design bug the compiler will not catch.*

- **Adopt:** Establish a global lock order, document it, and keep critical sections small. Use `try_lock` to diagnose rather than to paper over the design.
- **Smell:** Two locks acquired in different orders; nested locks; a lock held across `.await`.
- **Relearn:** What is the lock order, and where is it written?

### Bound your queues

> *An unbounded queue is a memory leak with extra steps.*

- **Adopt:** Prefer bounded channels, apply backpressure, and choose a reject-or-block policy. Monitor queue depth and latency.
- **Smell:** Unbounded channels; memory grows with a slow consumer; no drop or rejection policy.
- **Relearn:** What happens when the consumer is slower than the producer?

### Poisoning is information

> *A poisoned lock means an invariant was already broken.*

- **Adopt:** Handle `PoisonError` deliberately: recover if the data is still valid, otherwise propagate. Avoid `unwrap()` on lock acquisition.
- **Smell:** `lock().unwrap()` everywhere; ignoring a poisoned state; panicking while holding a lock.
- **Relearn:** What invariant was broken when the lock was poisoned?

---

## Async and Runtimes

*Async is a scheduling model, not a magic performance switch.*

### Async is for I/O concurrency

> *Use async to wait on many things, not to compute faster.*

- **Adopt:** Use async for network and disk waits. Use threads or `rayon` for CPU-bound work. Keep the distinction visible in the API.
- **Smell:** Async for CPU-heavy loops; one task per core; async used because it is fashionable.
- **Relearn:** Is this workload waiting or computing?

### Do not block the executor

> *A blocked task blocks every task on that worker.*

- **Adopt:** Use `spawn_blocking` or a dedicated thread for blocking I/O and CPU work. Keep tasks short between `.await` points.
- **Smell:** `std::fs` or blocking HTTP in async code; long CPU loops; `Mutex` held across `.await`.
- **Relearn:** What is this task preventing the executor from doing?

### Cancellation is a drop

> *Dropping a future cancels it; cleanup must survive that.*

- **Adopt:** Make futures cancel-safe. Use guards for cleanup and avoid partial writes that cannot be resumed.
- **Smell:** Cleanup only on success; partial state after cancellation; `select!` dropping a future mid-write.
- **Relearn:** What happens if this future is dropped mid-await?

### `select!` and timeouts

> *Every await needs a bound, and every race needs a policy.*

- **Adopt:** Wrap operations in `tokio::time::timeout`. Use `select!` for racing, and decide what happens to the losing branch.
- **Smell:** No timeouts; `select!` without an `else` branch; cancelled work that leaves state behind.
- **Relearn:** What is the deadline, and what happens when it fires?

### Structured concurrency with `JoinSet`

> *Every task has an owner and a shutdown path.*

- **Adopt:** Use `JoinSet` or `TaskTracker` to own tasks. Await them on shutdown. Avoid detached `tokio::spawn` unless the task is truly independent.
- **Smell:** Fire-and-forget `tokio::spawn`; tasks outliving the request; shutdown that waits forever.
- **Relearn:** Who owns this task, and when is it joined?

### Runtime choice is an API decision

> *A library should not force a runtime on its users.*

- **Adopt:** Keep libraries runtime-agnostic where possible. Choose Tokio or async-std at the application edge, and document the choice.
- **Smell:** Tokio types in a public API; multiple runtimes in one process; runtime-specific behavior leaking into a generic crate.
- **Relearn:** Is this crate tied to a runtime, and should it be?

### Futures are lazy

> *A future does nothing until someone polls it.*

- **Adopt:** Await or spawn futures deliberately. Remember that constructing an `async fn` future does not run its body.
- **Smell:** A future created for its side effects and then dropped; an `async fn` called without `.await`; spawning only to start work that is never joined.
- **Relearn:** Who polls this future, and when?

### Backpressure belongs in the design

> *Accepting work is a promise to finish it.*

- **Adopt:** Bound channels, use semaphores, apply load shedding, and propagate pressure to the caller. Decide what to reject before the queue grows.
- **Smell:** Unbounded `spawn`; queues that grow under load; an accept loop with no limit.
- **Relearn:** What happens when work arrives faster than it completes?

### Spawn only when you can join

> *A detached task is ownership you gave away.*

- **Adopt:** Keep the `JoinHandle`, track tasks in a registry or `JoinSet`, and cancel or await them during shutdown.
- **Smell:** Detached `tokio::spawn`; no shutdown path; tasks that outlive the request that created them.
- **Relearn:** Who waits for this task, and how is it stopped?

---

## Performance and Iterators

*The optimizer is not a promise. The cost model is your responsibility.*

### Iterators are zero-cost until they aren’t

> *An iterator chain is an abstraction over a loop, not a guarantee about the loop.*

- **Adopt:** Prefer iterator chains for clarity, then check the generated code or benchmark when the path is hot. Watch for `Box<dyn Iterator>`, closures that capture heavily, and chains that hide allocation.
- **Smell:** “Iterators are always zero-cost” used to skip measurement; `dyn Iterator` in a tight loop; a chain that is rewritten by hand for speed without a profile.
- **Relearn:** What machine code did this chain become, and did I look?

### Avoid needless `collect`

> *Collecting is a materialization decision, not punctuation.*

- **Adopt:** Chain lazily and consume once. Collect only when you need random access, reuse, or a boundary. Use `collect::<Result<Vec<_>, _>>()` deliberately.
- **Smell:** `collect()` followed immediately by `iter()`; collecting into a `Vec` just to call `.len()`; multiple passes over materialized data that could be one pass.
- **Relearn:** Does this data need to exist all at once, or can it flow?

### Bounds checks are usually cheap; measure

> *Safety has a cost, but often less than the folklore claims.*

- **Adopt:** Write indexed code clearly first. If a bounds check shows up in a profile, restructure with iterators, slices, or `get_unchecked` behind a proven invariant.
- **Smell:** `unsafe { get_unchecked }` before profiling; assuming every check is a bottleneck; removing checks without a `// SAFETY:` proof.
- **Relearn:** Did the profiler blame the check, or did I?

### `&[u8]` and `Bytes` for I/O

> *Move bytes without copying them.*

- **Adopt:** Accept `&[u8]` and `impl AsRef<[u8]>` for parsing. Use `bytes::Bytes` for cheaply cloned, shared buffers in async and network code. Avoid `Vec<u8>` copies at every layer.
- **Smell:** `Vec<u8>` cloned per handler; `String` for binary data; converting between `&str` and `&[u8]` repeatedly.
- **Relearn:** How many times are these bytes copied, and who owns them?

### SIMD and target features

> *Vectorization is a portability decision as much as a performance one.*

- **Adopt:** Let the compiler auto-vectorize clean loops first. Use `std::simd` (nightly) or `wide`/`std::arch` when measured. Gate target features with `#[target_feature]` and runtime detection.
- **Smell:** Hand-written SIMD before a scalar baseline; `target_feature` without a runtime check; code that only builds on one CPU.
- **Relearn:** Which CPUs must run this, and how is the feature selected?

### Profile before optimizing

> *Intuition about performance is a hypothesis, not a measurement.*

- **Adopt:** Use `perf`, `cargo flamegraph`, `criterion`, and `iai-callgrind`. Establish a baseline, change one thing, and compare distributions, not single runs.
- **Smell:** Optimizing the code that “looks slow”; micro-benchmarks with no realistic workload; claiming a win without a before-and-after.
- **Relearn:** What did the measurement say, and what did I change?

### Measure allocations, not just time

> *A latency win that allocates more is often a throughput loss.*

- **Adopt:** Track allocations per operation with DHAT, heaptrack, or a counting allocator. Compare distributions and peak memory, not only wall time.
- **Smell:** A faster benchmark with an allocation regression; no allocation baseline; performance work that ignores memory.
- **Relearn:** What did this change cost in allocations and peak memory?

### Optimize the memory hierarchy

> *The fastest data is the data already in cache.*

- **Adopt:** Choose layouts for locality: struct-of-arrays when scanning, compact enums, contiguous buffers. Measure cache misses and branch misses.
- **Smell:** Linked lists in a hot path; `Vec<Box<T>>` for sequential data; random access over a large array.
- **Relearn:** Where does this data live, and how far is it from the CPU?

---

## Embedded and Systems Constraints

*On a small device, the platform is part of the design.*

### `no_std` is a discipline

> *Every dependency on the platform must be deliberate.*

- **Adopt:** Separate core logic from `std`. Use `core` and `alloc` explicitly, and test on the target, not only on the host.
- **Smell:** An accidental `std` dependency; heap allocation assumed; host-only tests for target behavior.
- **Relearn:** What does this code need from the platform?

### Panics are expensive

> *On a microcontroller, a panic is a reset with extra steps.*

- **Adopt:** Prefer `Result` in embedded paths, define a panic handler, and measure the code size and latency cost of failure.
- **Smell:** `unwrap()` in an interrupt handler; panics in `no_std`; unwinding on a target that cannot afford it.
- **Relearn:** What happens when this panics on the target?

### Interrupts are concurrency

> *An interrupt is another thread you did not spawn.*

- **Adopt:** Treat interrupt handlers as concurrent code. Use critical sections or atomics, keep handlers short, and defer work to the main loop.
- **Smell:** Shared state without synchronization; long interrupt handlers; blocking inside an interrupt.
- **Relearn:** What can preempt this code, and what does it touch?

### DMA and ownership

> *While the hardware reads a buffer, the CPU must not touch it.*

- **Adopt:** Model DMA buffers as owned or moved values. Enforce lifetime, alignment, and cache maintenance before and after transfer.
- **Smell:** CPU and DMA touching the same buffer; freeing a buffer during transfer; stale cache after DMA.
- **Relearn:** Who owns this buffer while the hardware reads it?

### Memory is fixed; plan it

> *On a constrained target, the worst case is the only case that matters.*

- **Adopt:** Prefer static allocation and pools. Budget stack usage, avoid unbounded recursion, and measure the worst-case footprint.
- **Smell:** Heap allocation in firmware without a need; unbounded recursion; stack overflow discovered in the field.
- **Relearn:** What is the worst-case memory use?

### Real-time is a deadline

> *A missed deadline is a failure, not a slow success.*

- **Adopt:** Bound worst-case execution time, jitter, and blocking. Test under load and interference. Keep unbounded operations out of the critical path.
- **Smell:** Average latency used to claim real-time; garbage collection or page faults in a control loop; priorities without worst-case analysis.
- **Relearn:** What is the worst case, and what happens when it is missed?

### Power is a design budget

> *Every cycle awake has an energy cost.*

- **Adopt:** Sleep when idle, batch work, avoid busy-waiting, and measure energy and thermal behavior. Design for throttling and battery limits.
- **Smell:** Spin loops; an always-on radio; performance claims that assume unlimited power.
- **Relearn:** What does this cost in energy, and when is it awake?

---

## API and Trait Design

*A public API is a contract the compiler cannot fully enforce.*

### Accept borrowed, return owned

> *Be liberal in what you accept and explicit in what you return.*

- **Adopt:** Take `&str`, `&[T]`, and `impl AsRef<T>` on the way in. Return `String`, `Vec<T>`, or an owned type when the caller must keep it. Use `Cow` only when the condition is real.
- **Smell:** `&String` and `&Vec<T>` parameters; returning references tied to internal state; forcing callers to clone before every call.
- **Relearn:** Does the caller need to keep this value, or only look at it?

### Generics for zero-cost, `dyn` for flexibility

> *Static dispatch is fast and viral; dynamic dispatch is flexible and indirect.*

- **Adopt:** Use generics for hot paths and small, known type sets. Use `dyn Trait` for heterogeneous collections, plugin boundaries, and reducing monomorphization. Measure compile time and binary size too.
- **Smell:** Generic explosion across a large API; `dyn` in a tight numeric loop; choosing one without naming the tradeoff.
- **Relearn:** Am I optimizing for runtime, compile time, or extensibility?

### Traits should be small and composable

> *A trait is a role, not a bag of unrelated abilities.*

- **Adopt:** Prefer several focused traits with blanket impls over one large trait. Use associated types for one-per-impl relationships and generic parameters for many-per-impl.
- **Smell:** God traits with twenty methods; default methods that assume implementation details; traits that exist only to group functions.
- **Relearn:** What single role does this trait describe?

### Semver is a promise

> *Every public item is part of the contract, including error variants and trait impls.*

- **Adopt:** Treat public types, trait methods, enum variants, and feature names as semver-relevant. Use `cargo-semver-checks` in CI. Add variants only with care.
- **Smell:** Breaking changes in a patch release; `#[non_exhaustive]` added too late; “it’s just an internal type” that is actually public.
- **Relearn:** What did I promise to downstream crates, and did this change break it?

### Feature flags are a compatibility surface

> *Every feature combination is a crate you have not tested.*

- **Adopt:** Keep features additive, avoid mutually exclusive flags, and test the combinations you claim to support. Document what each feature enables and its default.
- **Smell:** Features that change public types; default features pulling heavy dependencies; combinations that fail to compile.
- **Relearn:** Which feature combinations do I actually support, and where is that tested?

### Documentation is part of the API

> *If the docs do not state the contract, the contract is whatever the code happens to do.*

- **Adopt:** Write crate-level docs with examples, `# Errors`, `# Panics`, and `# Safety` sections. Use doctests as executable examples. Document invariants and complexity.
- **Smell:** Missing `# Safety` on `unsafe fn`; examples that no longer compile; docs that describe the implementation instead of the contract.
- **Relearn:** Could a new user use this correctly without reading the source?

### Builders for complex construction

> *Many optional parameters are a type asking for a builder.*

- **Adopt:** Use a builder for optional configuration, with a `build()` that validates. Use typestate builders when some fields are mandatory.
- **Smell:** An eight-argument constructor; boolean parameters; a public struct that can be constructed half-valid.
- **Relearn:** Can construction be guided step by step, with invalid combinations rejected?

### Defaults should be safe

> *The zero configuration is the configuration most users run.*

- **Adopt:** Choose secure, unsurprising defaults. Make risky behavior explicit opt-in. Implement `Default` only when the default is obvious and valid.
- **Smell:** An insecure default; `Default` with surprising behavior; a zero value that is invalid but constructible.
- **Relearn:** What happens if the user does nothing?

### Non-exhaustive for future growth

> *Leave room to add a variant without breaking the world.*

- **Adopt:** Put `#[non_exhaustive]` on public enums and structs likely to grow. Document that downstream matches need a wildcard.
- **Smell:** Adding a variant breaks downstream; exhaustive matches in user code; no migration path for new fields.
- **Relearn:** Can I add a variant or field without breaking callers?

### Prefer inherent methods until a trait is needed

> *A trait should enable genericity or extension, not decorate a type.*

- **Adopt:** Start with inherent methods. Introduce a trait when you need a generic abstraction, a shared contract, or external extension.
- **Smell:** A trait with one implementation; trait imports required for basic use; method-resolution confusion from overlapping traits.
- **Relearn:** Is this trait enabling genericity, or just grouping functions?

---

## Macros and Metaprogramming

*Macros are powerful because they see syntax. That is also why they are expensive.*

### Macros are a last resort

> *If a function, generic, or trait can express it, use that.*

- **Adopt:** Reach for a macro only for syntax that cannot be expressed otherwise, or for code generation that removes real repetition.
- **Smell:** A macro for a two-line function; a macro hiding control flow; an expansion nobody can read.
- **Relearn:** What can a function or generic do that this macro cannot?

### Declarative macros for syntax, procedural for structure

> *Match the macro kind to the job.*

- **Adopt:** Use `macro_rules!` for pattern-based repetition and simple syntax. Use procedural macros for derives, attributes, and generated items that need parsing.
- **Smell:** A procedural macro for simple substitution; `macro_rules!` parsing a complex grammar; a proc macro that reimplements a parser badly.
- **Relearn:** Which kind of macro fits this job?

### Hygiene is not optional

> *A macro must not accidentally capture the caller’s names.*

- **Adopt:** Use `$crate`, unique identifiers, and explicit paths. Avoid relying on names or imports the caller happens to have.
- **Smell:** A macro breaks when the caller defines a same-named item; unqualified `Vec`; reliance on caller imports.
- **Relearn:** What names can this macro accidentally capture?

### Keep macro expansion debuggable

> *Generated code is still code someone has to maintain.*

- **Adopt:** Inspect expansions with `cargo expand`, keep them small, and point spans at the user’s input. Test the expansion where it matters.
- **Smell:** Hundreds of lines from one macro; errors that point at macro internals; no one has ever run `cargo expand`.
- **Relearn:** Can a user understand the code this macro generates?

### Derive macros should be boring

> *A derive should add the obvious impl and nothing surprising.*

- **Adopt:** Keep derives predictable, document custom attributes, and avoid hidden behavior or side effects.
- **Smell:** A derive with side effects; surprising trait bounds; attributes that silently change semantics.
- **Relearn:** Would a user be surprised by what this derive does?

### Document macro syntax

> *A macro is a language; document its grammar.*

- **Adopt:** Document accepted forms, examples, and failure modes. Use doctests for macro examples and show the expansion when it helps.
- **Smell:** An undocumented DSL; examples only in tests; error messages nobody has explained.
- **Relearn:** Can someone use this macro from the docs alone?

---

## Serialization and Data Modeling

*A wire format is a promise to every future reader.*

### Serde is a schema, not a convenience

> *Deriving `Serialize` is a design decision about your public data.*

- **Adopt:** Define wire types separately from domain types when they diverge. Document field names, defaults, and compatibility rules. Test round trips against fixtures.
- **Smell:** Deriving `Serialize` on internal types; the wire format coupled to struct layout; no schema or compatibility test.
- **Relearn:** What have I promised to every reader of this format?

### Version your wire formats

> *Data outlives the code that wrote it.*

- **Adopt:** Include a version, prefer additive changes, define the unknown-field policy, and keep a migration path.
- **Smell:** A breaking field rename; no version marker; consumers break silently after a deploy.
- **Relearn:** How will next year’s reader handle today’s data?

### Validate at the boundary

> *Deserialization is where untrusted data becomes typed data.*

- **Adopt:** Deserialize into validated types, reject invalid input early, and do not trust field presence or ranges.
- **Smell:** `Value` passed deep into the system; validation repeated at every layer; defaults hiding missing data.
- **Relearn:** What type proves this message was valid?

### Avoid `serde_json::Value` in core logic

> *Dynamic JSON belongs at the edge, not in the domain.*

- **Adopt:** Deserialize into typed structs at the boundary. Use `Value` only for genuinely dynamic edges, and convert once.
- **Smell:** `Value` threaded through business logic; stringly field access; runtime type errors that the compiler could catch.
- **Relearn:** Where does dynamic data become typed?

### Own your serialization errors

> *A parser error is not a user-facing diagnosis.*

- **Adopt:** Map serde errors to domain errors with field paths and context. Do not leak parser internals or lose the location.
- **Smell:** Raw serde error returned to the user; no field path; a generic “invalid JSON” for every failure.
- **Relearn:** Can the caller tell which field failed?

### Zero-copy formats need alignment

> *Reading bytes in place is only sound if the layout allows it.*

- **Adopt:** Use formats such as `rkyv` or FlatBuffers with explicit alignment guarantees. Validate before access and document endianness.
- **Smell:** Transmuting bytes into structs; unaligned access; assuming host endianness on the wire.
- **Relearn:** What guarantees this buffer can be read in place?

### Unknown fields are a compatibility decision

> *Strictness is a policy, not an accident.*

- **Adopt:** Choose `deny_unknown_fields` for strict inputs and allow unknown fields for forward compatibility. Document the choice and test it.
- **Smell:** Accidental strictness; silent data loss; changing the policy in a patch release.
- **Relearn:** What should happen when a new field arrives?

---

## Networking and I/O

*The network is a hostile, asynchronous, partially ordered place.*

### Treat the network as unreliable

> *Assume loss, duplication, reordering, and partition.*

- **Adopt:** Design for idempotency, retries, and partial failure. Make message handling safe when the same message arrives twice.
- **Smell:** Assuming ordered delivery; no retry or deduplication; a protocol that breaks on reconnect.
- **Relearn:** What happens if this message arrives twice or not at all?

### Timeouts are mandatory

> *An operation without a deadline can wait forever.*

- **Adopt:** Set connect, read, write, idle, and total deadlines. Propagate them through the call stack and test that they fire.
- **Smell:** No timeout; infinite retry; a timeout only at the outermost edge.
- **Relearn:** What is the deadline, and who enforces it?

### Retries need jitter and budgets

> *A retry storm is an outage that feeds itself.*

- **Adopt:** Use exponential backoff with jitter, retry budgets, idempotency keys, and a capped attempt count.
- **Smell:** Immediate retry loops; synchronized retries across clients; retrying non-idempotent writes.
- **Relearn:** Could this retry make the outage worse?

### Framing is part of the protocol

> *A stream has no message boundaries until you define them.*

- **Adopt:** Use length-prefixed or delimiter framing with a maximum frame size. Handle partial reads and writes correctly.
- **Smell:** Assuming one read equals one message; no maximum size; delimiter injection.
- **Relearn:** Where does one message end and the next begin?

### Backpressure at every boundary

> *A fast writer and a slow reader are a memory leak in progress.*

- **Adopt:** Use bounded buffers, flow control, and load shedding. Propagate pressure instead of absorbing it silently.
- **Smell:** An unbounded read buffer; a slow consumer with no policy; an accept loop without a limit.
- **Relearn:** What happens when the peer reads slower than I write?

### TLS is not optional

> *Plaintext across a trust boundary is a data breach waiting for a network tap.*

- **Adopt:** Use `rustls` or platform TLS, verify certificates, set SNI correctly, and plan rotation. Test expiry and verification failures.
- **Smell:** Plaintext across a trust boundary; disabled certificate verification; a pinned certificate with no rotation plan.
- **Relearn:** Who can read or modify this traffic?

### Connection pools are state machines

> *A connection is a resource with a lifecycle, not a boolean.*

- **Adopt:** Bound the pool, health-check connections, apply acquisition and idle timeouts, and discard broken connections.
- **Smell:** An unbounded pool; stale connections reused after failure; pool exhaustion under load.
- **Relearn:** What happens when every connection is busy or dead?

---

## Security and Cryptography

*Security is a property of the whole system, not a crate you add.*

### Threat model before code

> *You cannot defend an asset you have not named.*

- **Adopt:** Identify assets, actors, trust boundaries, and attack surface. Write the model down and revisit it when the system changes.
- **Smell:** Security added at the end; no adversary model; “we’re internal, so it’s fine.”
- **Relearn:** Who would attack this, and what do they gain?

### Do not roll your own crypto

> *Cryptography is the one place where “I understand the math” is not enough.*

- **Adopt:** Use vetted crates such as `ring`, RustCrypto, or `sodiumoxide`. Prefer high-level APIs and never invent a protocol.
- **Smell:** A custom cipher; a hand-rolled nonce; “simple” crypto; a DIY key exchange.
- **Relearn:** Who has reviewed this construction, and what standard is it based on?

### Secrets are not strings

> *A `String` can be cloned, logged, and swapped.*

- **Adopt:** Use `secrecy` and `zeroize`, avoid `Debug`/`Display` on secrets, limit copies, and consider `mlock` for high-value keys.
- **Smell:** Secrets in logs; `String` secrets; cloned secrets; secrets in core dumps.
- **Relearn:** Where can this secret be copied, logged, or swapped?

### Validate untrusted input

> *Every boundary is an input validation problem.*

- **Adopt:** Parse into constrained types, enforce bounds and canonicalization, and reject by default. Treat deserialization as hostile input.
- **Smell:** Trusting lengths; path traversal; deserializing untrusted data into powerful types; shell interpolation.
- **Relearn:** What is the worst input an attacker can send?

### Constant-time where it matters

> *A timing difference can be a side channel.*

- **Adopt:** Use `subtle` for secret-dependent comparisons, avoid secret-dependent branches, and measure where the threat model requires it.
- **Smell:** `==` on MACs; an early return on a secret byte; timing leaks in token comparison.
- **Relearn:** Does the time this takes reveal the secret?

### Dependencies are attack surface

> *Every crate is code you ship and trust.*

- **Adopt:** Audit, minimize, and pin dependencies. Review build scripts, use `cargo-vet` or `cargo-deny`, and monitor advisories.
- **Smell:** Unmaintained crates; unknown build scripts; typosquatting; no audit in CI.
- **Relearn:** What code am I trusting that I did not write?

### Least privilege for processes

> *A process should have only the access it needs.*

- **Adopt:** Drop privileges, sandbox where possible, apply seccomp or capabilities, and run with a minimal filesystem view.
- **Smell:** Running as root; broad filesystem access; unnecessary capabilities; one process for everything.
- **Relearn:** What is the minimum this process needs?

### Log without leaking

> *Logs are a copy of your data with a different retention policy.*

- **Adopt:** Redact secrets and PII, use structured fields, set retention, and restrict access to logs.
- **Smell:** Tokens in logs; full request bodies; unbounded retention; logs shipped to a third party unchecked.
- **Relearn:** What would an attacker learn from my logs?

---

## Testing, Debugging, and Verification

*Tests are evidence, not decoration.*

### Unit tests next to code

> *Test the unit where it is defined, including private behavior.*

- **Adopt:** Use `#[cfg(test)] mod tests` beside the code. Test invariants, edge cases, and error paths, not just the happy path.
- **Smell:** Tests only in a distant integration suite; no test for the error branch; tests that assert implementation details.
- **Relearn:** Which invariant does this test prove, and where does it live?

### Integration tests at crate boundary

> *Test the API the user actually sees.*

- **Adopt:** Put tests in `tests/` and exercise the public API. Use `trybuild` for compile-fail tests and doc tests for examples.
- **Smell:** Only unit tests, never a public-API test; integration tests reaching into private modules; examples that are not tested.
- **Relearn:** Does this test use the crate the way a caller would?

### Property tests and fuzzing

> *Example tests check the cases you thought of; properties check the ones you did not.*

- **Adopt:** Use `proptest` or `quickcheck` for invariants, and `cargo-fuzz`/`libFuzzer` for parsers and unsafe boundaries. Keep failing cases as regression tests.
- **Smell:** Only hand-picked inputs; fuzzing without a corpus; property tests that restate the implementation.
- **Relearn:** What property must hold for all inputs, and what generated the counterexample?

### Miri, loom, and sanitizers

> *Some bugs are invisible to ordinary tests.*

- **Adopt:** Run `cargo miri test` for undefined behavior, `loom` for concurrency models, and ASan/TSan for FFI and threaded code. Put them in CI where they matter.
- **Smell:** Unsafe code tested only on one platform; concurrency “verified” by a passing test run; sanitizers run once and never again.
- **Relearn:** Which tool would catch the bug class I cannot see?

### `dbg!`, `tracing`, and backtraces

> *Debugging is a skill of asking the system what it did.*

- **Adopt:** Use `dbg!` locally, `tracing` for structured events, and `RUST_BACKTRACE=1` with `panic = "abort"` considered. Keep spans across async boundaries.
- **Smell:** `println!` debugging left in production; logs without request IDs; backtraces disabled in release with no alternative.
- **Relearn:** What evidence would tell me what actually happened?

### Reproduce, reduce, bisect

> *A bug you cannot reproduce is a bug you do not yet understand.*

- **Adopt:** Capture the exact input, environment, and version. Reduce to the smallest failing case. Use `git bisect` and `cargo bisect-rustc` when the compiler or dependency changed.
- **Smell:** Fixing by guesswork; “cannot reproduce” closing the issue; changing several things at once.
- **Relearn:** What is the smallest input that still fails?

### Test the failure path

> *The happy path is the part you already know works.*

- **Adopt:** Test errors, timeouts, cancellation, partial failure, and invalid input. Make mocks fail on purpose and assert the recovery behavior.
- **Smell:** Only success cases; an error branch with no test; a mock that never fails.
- **Relearn:** Which failure would hurt most in production, and where is its test?

### Golden tests for parsers

> *A parser is defined by its corpus.*

- **Adopt:** Keep snapshot inputs and outputs, review diffs deliberately, and include malformed and adversarial cases.
- **Smell:** Parser tests with no corpus; snapshot churn accepted blindly; no regression file for past bugs.
- **Relearn:** What input would break this parser, and is it in the corpus?

### Compile-fail tests for APIs

> *Some guarantees are enforced at compile time; test those too.*

- **Adopt:** Use `trybuild` to assert that invalid usage fails, auto traits behave, and error messages stay useful.
- **Smell:** Safety claims with no compile-fail test; an API that compiles when it should not; error messages nobody checks.
- **Relearn:** What should fail to compile, and is that tested?

---

## Tooling, Build, and Ecosystem

*The build is part of the program.*

### Cargo is the build contract

> *If it is not in `Cargo.toml`, it is not reproducible.*

- **Adopt:** Keep dependencies, features, profiles, and metadata explicit. Commit `Cargo.lock` for binaries, use workspaces for multi-crate repos, and document build requirements.
- **Smell:** Hand-edited build scripts that bypass Cargo; floating dependency versions; a lockfile that is never updated or always churning.
- **Relearn:** Can a new machine build this from the repository alone?

### Clippy and rustfmt are defaults

> *Style is not a debate; it is a baseline.*

- **Adopt:** Run `cargo fmt --check` and `cargo clippy -- -D warnings` in CI. Fix lints or allow them with a reason. Keep the toolchain pinned.
- **Smell:** Formatting-only diffs; dozens of ignored lints; `#[allow(clippy::all)]` at the crate root.
- **Relearn:** Which lint is telling me something about the design?

### Pin and audit dependencies

> *Every dependency is code you ship and trust.*

- **Adopt:** Pin versions in `Cargo.lock`, run `cargo audit` and `cargo deny`, and review new dependencies for license, maintenance, and unsafe use.
- **Smell:** Dependencies added for one function; no audit in CI; transitive crates nobody has reviewed.
- **Relearn:** What does this dependency bring, and who maintains it?

### MSRV is a promise

> *A minimum supported Rust version is a compatibility contract.*

- **Adopt:** Declare `rust-version` in `Cargo.toml`, test it in CI, and treat raising it as a semver-relevant change. Avoid newer syntax until the MSRV allows it.
- **Smell:** MSRV discovered by users; raising it in a patch release; CI testing only stable.
- **Relearn:** What is the oldest compiler this crate promises to support?

### Reproducible builds and supply chain

> *The artifact should be the same tomorrow as it was today.*

- **Adopt:** Pin toolchains with `rust-toolchain.toml`, vendor or lock dependencies, and verify checksums. Consider `cargo-vet`, signed releases, and provenance.
- **Smell:** Builds that differ between machines; unpinned nightly; no record of what produced a release binary.
- **Relearn:** Could I rebuild this exact artifact and prove it matches?

### Keep the dependency graph small

> *The best dependency is the one you did not add.*

- **Adopt:** Prefer the standard library and small, focused crates. Check `cargo tree` and binary size. Replace a heavy dependency with a few lines when the maintenance cost is lower.
- **Smell:** A framework for one feature; duplicate crates doing the same job; compile times dominated by transitive dependencies.
- **Relearn:** What is the smallest dependency set that solves this problem?

### Workspaces for boundaries

> *A workspace is a statement about which crates change together.*

- **Adopt:** Split crates when they have independent APIs, lifecycles, or compile costs. Share workspace dependencies and keep dependency edges acyclic.
- **Smell:** One giant crate; duplicated dependency versions across members; cyclic dev-dependencies.
- **Relearn:** What boundary does this workspace member represent?

### CI is the executable contract

> *If CI does not check it, it is a hope.*

- **Adopt:** Run formatting, clippy, tests, MSRV, audit, docs, and the feature matrix. Fail fast, cache deliberately, and keep CI reproducible.
- **Smell:** CI that only builds; warnings allowed to accumulate; feature combinations untested; a green build that proves little.
- **Relearn:** What does CI prove, and what does it not?

### Release profiles are product decisions

> *Optimization settings change behavior, size, and debuggability.*

- **Adopt:** Choose `opt-level`, LTO, `codegen-units`, and panic strategy deliberately. Measure speed, binary size, and build time.
- **Smell:** Default release settings for a latency-critical service; `panic = "abort"` without understanding the tradeoff; LTO as cargo cult.
- **Relearn:** What does this profile optimize for, and what does it cost?

---

## Observability and Operations

*If you cannot see it, you cannot operate it.*

### Structured logs, not printf

> *A log line is data, not a sentence.*

- **Adopt:** Use `tracing` with fields, spans, and stable event names. Include request IDs and outcome, and choose levels deliberately.
- **Smell:** `println!` in production; concatenated strings; inconsistent field names; logs that cannot be queried.
- **Relearn:** Can I query this log by request, user, and outcome?

### Metrics answer questions

> *A metric nobody queries is a cost with no benefit.*

- **Adopt:** Instrument RED and USE metrics, use histograms for latency, keep label cardinality bounded, and tie metrics to SLOs.
- **Smell:** High-cardinality labels; averages only; dashboards nobody uses; metrics with no owner.
- **Relearn:** Which question does this metric answer?

### Traces connect cause and effect

> *A request is a story that crosses process boundaries.*

- **Adopt:** Propagate trace context, create spans at meaningful boundaries, sample deliberately, and link logs and metrics to traces.
- **Smell:** Broken trace context; spans without attributes; 100% sampling in production; traces nobody can follow.
- **Relearn:** Can I follow one request across services?

### Health checks are contracts

> *Liveness and readiness answer different questions.*

- **Adopt:** Keep checks cheap and bounded. Readiness reflects the ability to serve; liveness reflects whether a restart would help. Test the failure behavior.
- **Smell:** Readiness that checks every dependency; liveness that restarts on a dependency failure; expensive health checks.
- **Relearn:** What should happen when this check fails?

### Graceful shutdown is a feature

> *Processes stop; in-flight work should not disappear with them.*

- **Adopt:** Handle signals, stop accepting, drain with a deadline, then force exit. Test shutdown under load.
- **Smell:** Only `SIGKILL`; dropped in-flight requests; shutdown that hangs forever; no drain period.
- **Relearn:** What happens to in-flight work when this process stops?

### Configuration is input

> *Configuration is untrusted data with a different delivery mechanism.*

- **Adopt:** Parse into typed configuration at startup, validate ranges, document defaults, and fail fast on malformed input.
- **Smell:** Unchecked environment variables; configuration read everywhere; silent fallback; a zero value that means “unset.”
- **Relearn:** What happens with a missing or malformed setting?

### Resource limits are part of design

> *Every resource is finite, including the ones you forgot to name.*

- **Adopt:** Bound memory, file descriptors, threads, and connections. Set limits, handle exhaustion, and alert before the cliff.
- **Smell:** Unbounded caches; file descriptor leaks; one thread per request; no limit on concurrent work.
- **Relearn:** What happens when this resource runs out?

### Runbooks are code for humans

> *An incident is the wrong time to discover the procedure.*

- **Adopt:** Document detection, mitigation, rollback, and escalation. Test the runbook in game days and keep it next to the service.
- **Smell:** Tribal knowledge; an outdated runbook; no rollback path; escalation that depends on one person.
- **Relearn:** Can someone on call act without me?

---

## Project Structure and Maintenance

*Code is read, changed, and deleted far more often than it is written.*

### Modules are boundaries

> *A module should hide something, not just group files.*

- **Adopt:** Organize by domain, keep the public surface small, and choose a consistent file layout. Let the module tree express ownership and visibility.
- **Smell:** A god module; circular dependencies; everything `pub`; modules named after file types instead of concepts.
- **Relearn:** What does this module hide from the rest of the crate?

### Visibility is a tool

> *Private by default; public on purpose.*

- **Adopt:** Start private, use `pub(crate)` for internal sharing, and expose only the API that needs a contract. Review every `pub` as a semver decision.
- **Smell:** `pub` everywhere; internal types leaking; accidental public API; visibility widened to fix a test.
- **Relearn:** Who actually needs to see this?

### Keep `lib.rs` small

> *The crate root is a facade, not a dumping ground.*

- **Adopt:** Use `lib.rs` to declare modules and re-export the intentional API. Keep implementation in focused modules.
- **Smell:** A thousand-line `lib.rs`; business logic in the facade; an export list nobody can explain.
- **Relearn:** Can I understand the crate’s surface from `lib.rs`?

### Crate boundaries follow ownership

> *Split when the contract is independent, not because the file is long.*

- **Adopt:** Create a crate when a component has its own lifecycle, API, or compile cost. Avoid premature splitting that adds versioning overhead.
- **Smell:** One crate per file; cyclic workspace dependencies; shared mutable internals across crates.
- **Relearn:** What independent contract does this crate own?

### Deprecate before removing

> *A removal is a breaking change; a deprecation is a migration path.*

- **Adopt:** Mark items `#[deprecated]` with a replacement and a migration window. Remove only in a major release, and document the path.
- **Smell:** Silent removal; deprecation without an alternative; items deprecated forever with no plan.
- **Relearn:** How does a user migrate off this?

### Refactor in small, compiling steps

> *A refactor that does not compile is a rewrite.*

- **Adopt:** Keep the tree green, separate mechanical changes from behavior changes, and rely on tests to preserve behavior.
- **Smell:** A giant refactor branch; renaming and behavior changes in one commit; no tests to catch regressions.
- **Relearn:** What is the smallest step that compiles and preserves behavior?

### Delete code

> *The safest code is the code that no longer exists.*

- **Adopt:** Remove dead code, unused features, and stale docs. Use the compiler, coverage, and `cargo udeps` to find candidates.
- **Smell:** Commented-out code; dead feature flags; “we might need it later”; code kept only because deleting feels risky.
- **Relearn:** What breaks if I delete this?

---

## Index

Grouped by domain. Use the domain links above to jump to the full entries.

- **Ownership and Borrowing:** Borrow, don’t clone · Move is a feature · One owner at a time · Split borrows instead of fighting the checker · Interior mutability is a deliberate escape hatch · `Drop` is part of the contract · Shared ownership is a last resort
- **Lifetimes and References:** Lifetimes describe, not create · Prefer owned data at boundaries · Do not outlive your borrow · `'static` is not “forever” · Variance matters · Self-referential types need a plan · Borrow from the narrowest scope · Reborrow instead of moving
- **Types and Invariants:** Make illegal states unrepresentable · Newtype everything meaningful · Enums over boolean flags · Parse, don’t validate · Typestate for protocols · Sealed traits for controlled extension · Const generics for compile-time shape · `PhantomData` for type-level meaning · Zero-sized types carry proof
- **Errors and `Option`:** `Result` is the default · `?` is for propagation, not policy · Error types are API · Do not `unwrap` in libraries · `Option` is not an error · Panics are for bugs · Errors should carry context · Design for the caller’s recovery
- **Memory and Allocation:** Stack by default, heap by necessity · `Box` is for indirection, not everything · `Vec` capacity is a performance decision · `Cow` is for conditional ownership · Arenas for phase-scoped data · Leaks are safe, rarely acceptable · Avoid hidden allocations in APIs · Reuse buffers across requests
- **Unsafe and FFI:** `unsafe` is a proof obligation · Minimize the unsafe surface · Document safety invariants · FFI boundaries are contracts · `repr(C)` and layout · Miri and sanitizers · `Pin` is a safety contract · Never unwind across FFI · Wrap C in a safe Rust API
- **Concurrency:** `Send` and `Sync` are contracts · Prefer channels to shared state · `Arc<Mutex<T>>` is not a default · Atomics need ordering · Scoped threads beat `'static` clones · Rayon for data parallelism · Avoid lock ordering inversions · Bound your queues · Poisoning is information
- **Async and Runtimes:** Async is for I/O concurrency · Do not block the executor · Cancellation is a drop · `select!` and timeouts · Structured concurrency with `JoinSet` · Runtime choice is an API decision · Futures are lazy · Backpressure belongs in the design · Spawn only when you can join
- **Performance and Iterators:** Iterators are zero-cost until they aren’t · Avoid needless `collect` · Bounds checks are usually cheap; measure · `&[u8]` and `Bytes` for I/O · SIMD and target features · Profile before optimizing · Measure allocations, not just time · Optimize the memory hierarchy
- **Embedded and Systems Constraints:** `no_std` is a discipline · Panics are expensive · Interrupts are concurrency · DMA and ownership · Memory is fixed; plan it · Real-time is a deadline · Power is a design budget
- **API and Trait Design:** Accept borrowed, return owned · Generics for zero-cost, `dyn` for flexibility · Traits should be small and composable · Semver is a promise · Feature flags are a compatibility surface · Documentation is part of the API · Builders for complex construction · Defaults should be safe · Non-exhaustive for future growth · Prefer inherent methods until a trait is needed
- **Macros and Metaprogramming:** Macros are a last resort · Declarative macros for syntax, procedural for structure · Hygiene is not optional · Keep macro expansion debuggable · Derive macros should be boring · Document macro syntax
- **Serialization and Data Modeling:** Serde is a schema, not a convenience · Version your wire formats · Validate at the boundary · Avoid `serde_json::Value` in core logic · Own your serialization errors · Zero-copy formats need alignment · Unknown fields are a compatibility decision
- **Networking and I/O:** Treat the network as unreliable · Timeouts are mandatory · Retries need jitter and budgets · Framing is part of the protocol · Backpressure at every boundary · TLS is not optional · Connection pools are state machines
- **Security and Cryptography:** Threat model before code · Do not roll your own crypto · Secrets are not strings · Validate untrusted input · Constant-time where it matters · Dependencies are attack surface · Least privilege for processes · Log without leaking
- **Testing, Debugging, and Verification:** Unit tests next to code · Integration tests at crate boundary · Property tests and fuzzing · Miri, loom, and sanitizers · `dbg!`, `tracing`, and backtraces · Reproduce, reduce, bisect · Test the failure path · Golden tests for parsers · Compile-fail tests for APIs
- **Tooling, Build, and Ecosystem:** Cargo is the build contract · Clippy and rustfmt are defaults · Pin and audit dependencies · MSRV is a promise · Reproducible builds and supply chain · Keep the dependency graph small · Workspaces for boundaries · CI is the executable contract · Release profiles are product decisions
- **Observability and Operations:** Structured logs, not printf · Metrics answer questions · Traces connect cause and effect · Health checks are contracts · Graceful shutdown is a feature · Configuration is input · Resource limits are part of design · Runbooks are code for humans
- **Project Structure and Maintenance:** Modules are boundaries · Visibility is a tool · Keep `lib.rs` small · Crate boundaries follow ownership · Deprecate before removing · Refactor in small, compiling steps · Delete code

---

## Self-audit

Answer each line honestly. “Not sure” counts as no. Close the gaps you marked.

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
- [ ] I know the allocation count per request on the hot path.
- [ ] My crate’s public API has documented errors, panics, and safety contracts.
- [ ] My wire formats have a version and a compatibility test.
- [ ] Every network call has a deadline, and retries have jitter and a budget.
- [ ] I know which inputs are untrusted, and they are parsed into constrained types.
- [ ] Secrets are redacted from logs and not stored as ordinary strings.
- [ ] My CI runs formatting, clippy, tests, MSRV, audit, and a feature matrix.
- [ ] I run Miri, loom, or a sanitizer on the code that needs it.
- [ ] I know my MSRV and test it.
- [ ] I can rebuild the last release artifact from the repository.
- [ ] I can follow one request across logs, metrics, and traces.
- [ ] I can explain what happens to in-flight work during shutdown.
- [ ] I can explain why the weirdest piece of code in my area exists.

---

## Review lens

Use this when reviewing a Rust pull request. It is a checklist of questions, not a style guide.

| Domain | Ask |
| --- | --- |
| **Ownership** | Who owns this value, and is the clone justified? |
| **Lifetimes** | What keeps this reference alive, and does the API force a lifetime on callers? |
| **Types** | Can an invalid state be constructed? Is the meaning in the type? |
| **Errors** | Can the caller distinguish absence from failure and act on it? |
| **Memory** | How many allocations per call, and what is their lifetime? |
| **Unsafe** | What invariant makes this sound, and what tests it? |
| **Concurrency** | What is shared, what is sent, and what ordering is required? |
| **Async** | What blocks, what is cancelled, and who owns the task? |
| **Performance** | What did the profiler say, and what does the abstraction cost? |
| **Embedded** | What does the platform not provide, and what is the worst case? |
| **API** | What is promised to downstream crates, and is it semver-safe? |
| **Macros** | Is this syntax or code generation, and can a user debug the expansion? |
| **Serialization** | What have I promised to every reader of this format? |
| **Networking** | What happens when the peer misbehaves or disappears? |
| **Security** | Who can reach it, what do they gain, and what stops them? |
| **Verification** | Which failure path is tested, and would a mutant survive? |
| **Build** | Is the build pinned, linted, audited, and reproducible? |
| **Operations** | Can I see what it is doing, and can someone else run it? |
| **Maintenance** | Can this be changed or deleted without a rewrite? |

A useful review comment names the mentality and the concrete risk:

> “This `Arc<Mutex<Vec<_>>>` is shared mutation by default. Only the metrics task writes it; can we send updates over a channel instead?”

---

## Anti-patterns

The fastest way to lose a good Rust habit is to keep a comforting one nearby.

| Anti-pattern | What it sounds like | Replacement |
| --- | --- | --- |
| **Clone to compile** | “Just add `.clone()`.” | “Who should own this, and why is borrowing hard?” |
| **`Rc<RefCell<T>>` by default** | “It works.” | “Is shared mutation actually required?” |
| **Boolean state soup** | “We check the flags.” | “Model the states as an enum.” |
| **Stringly typed errors** | “It returned an error.” | “Can the caller match on the failure?” |
| **`unwrap()` in production** | “It can’t fail.” | “Prove it, or return an error.” |
| **Unsafe without a proof** | “Trust me.” | “Write the `// SAFETY:` invariant.” |
| **`Arc<Mutex<T>>` everywhere** | “Shared state is simple.” | “Send ownership instead of sharing mutation.” |
| **Blocking the executor** | “It’s just one file read.” | “Use `spawn_blocking` or a dedicated thread.” |
| **Fire-and-forget tasks** | “It runs in the background.” | “Who joins it on shutdown?” |
| **Collect-then-iterate** | “It’s clearer.” | “Does this data need to exist all at once?” |
| **Optimize by intuition** | “This looks slow.” | “What did the profiler measure?” |
| **Semver optimism** | “It’s only a small change.” | “What did I promise downstream?” |
| **Dependency reflex** | “There’s a crate for that.” | “What does it bring, and who maintains it?” |
| **Macro-first design** | “A macro makes it concise.” | “Can a function or generic express this?” |
| **Schema by accident** | “Just derive `Serialize`.” | “What wire format did I just publish?” |
| **Timeout-free networking** | “It usually responds.” | “What is the deadline, and who enforces it?” |
| **Retry storm** | “Retry until it works.” | “Where is the jitter, budget, and idempotency?” |
| **Secrets as strings** | “It’s just a token.” | “Where can this be logged or cloned?” |
| **`println!` observability** | “The logs are in stdout.” | “Can I query this by request and outcome?” |
| **Permanent temporary code** | “We’ll clean it up later.” | “What breaks if I delete it now?” |

---

## How to share

Pick the format that fits your team, not the one that feels most impressive.

- **15-minute brown bag:** one mentality, one Rust example from your own crate, one question at the end.
- **Review comments:** cite the entry by name. “This is *Do not block the executor*” is faster than a paragraph and easier to remember.
- **Onboarding:** choose the mentalities that matter most for your codebase and make them required reading.
- **Postmortems:** map contributing factors to missing mentalities, then add the missing test or lint.
- **Lunch-and-learn series:** one domain per week, cycling through the stack, then repeat with new examples.

---

## A four-week plan

One cluster per week. Each week ends with an artifact, not a feeling.

### Week 1 — Foundations

- Read **Ownership and Borrowing**, **Lifetimes and References**, **Types and Invariants**, **Errors and `Option`**, and **Memory and Allocation**.
- Pick one public API and one error type. Remove an unjustified clone, make an invalid state unrepresentable, and add an error test.
- **Artifact:** a diff that makes ownership or failure clearer.

### Week 2 — Systems

- Read **Unsafe and FFI**, **Concurrency**, **Async and Runtimes**, **Performance and Iterators**, and **Embedded and Systems Constraints**.
- Pick one `unsafe` block, one shared-state path, and one hot path. Add a safety proof, bound a queue, and profile the hot path.
- **Artifact:** a documented safety proof or a measured performance change.

### Week 3 — Design and interfaces

- Read **API and Trait Design**, **Macros and Metaprogramming**, **Serialization and Data Modeling**, **Networking and I/O**, and **Security and Cryptography**.
- Pick one public trait, one wire format, and one network call. Check semver, add a compatibility test, and enforce a deadline.
- **Artifact:** an API decision record plus one boundary test.

### Week 4 — Practice and operations

- Read **Testing, Debugging, and Verification**, **Tooling, Build, and Ecosystem**, **Observability and Operations**, and **Project Structure and Maintenance**.
- Pick one untested failure path, one build promise, and one production signal. Add a property or fuzz test, verify the MSRV, and make one operation queryable.
- **Artifact:** a CI or observability change that would have caught a real problem.

---

## Further reading

**Language and ownership**

- *The Rust Programming Language*, Klabnik and Nichols — the canonical starting point.
- *Programming Rust*, Blandy, Orendorff, and Tindall — systems-oriented coverage of ownership and performance.
- *Rust for Rustaceans*, Jon Gjengset — idiomatic design, traits, async, and unsafe.
- *Effective Rust*, David Drysdale — focused advice with concrete examples.
- *Learn Rust With Entirely Too Many Linked Lists* — ownership and unsafe through deliberate exercises.

**Unsafe, concurrency, and async**

- *The Rustonomicon* — the unsafe code guidelines every `unsafe` author should read.
- *Rust Atomics and Locks*, Mara Bos — memory ordering and concurrent data structures.
- *Async Rust*, Maxwell Flitton and Caroline Morton — futures, runtimes, and practical async design.
- The Miri and loom documentation — executable checks for UB and concurrency.

**Performance and systems**

- *The Rust Performance Book*, Nicholas Nethercote — profiling, allocation, and optimization.
- *The Embedded Rust Book* — `no_std`, HALs, interrupts, and target workflows.
- *Making Embedded Systems*, Elecia White — firmware, hardware constraints, and practical embedded design.

**API design, networking, and security**

- *Rust API Guidelines* — naming, traits, documentation, and semver.
- *Zero to Production in Rust*, Luca Palmieri — building and operating a real service.
- The Cargo Book, the Rust Reference, and `cargo-semver-checks` documentation.
- The RustSec advisory database and `cargo-audit` documentation.
- *TCP/IP Illustrated*, W. Richard Stevens — the network behavior your abstractions assume.

**Operations and learning**

- *Site Reliability Engineering*, Beyer et al. — SLOs, error budgets, and operational design.
- *Incident Management for Operations*, Rob Schnepp et al. — command, communication, and coordination under pressure.
- *Accelerate*, Forsgren, Humble, and Kim — delivery performance as a system property.

---

## Contributing

- Keep entries **Rust-specific**: name the type, crate, lint, or tool.
- Keep the four-part shape: mantra, **Adopt**, **Smell**, **Relearn**.
- Prefer one concrete practice over three vague ones.
- Add entries to the domain that owns the decision, and update the index.
- If you add a domain, update the contents, the domain map, the review lens, and the four-week plan.
- Do not number entries. Names are the stable reference.
