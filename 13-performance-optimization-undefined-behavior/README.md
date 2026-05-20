# Performance, Optimization, and Undefined Behavior Interview Questions

Performance questions in C and C++ interviews are rarely just about writing faster code. Strong candidates should understand measurement, cache behavior, compiler optimization, algorithmic complexity, undefined behavior, and the risk of optimizing before proving where the bottleneck is.

## 1. What should you do before optimizing code?

**Short answer:** Measure first. Use profiling to identify the real bottleneck before changing code.

**Detailed answer:**
Performance intuition is often wrong. A slow program may be limited by algorithmic complexity, memory allocation, cache misses, I/O, synchronization, branch misprediction, or external systems. Profiling shows where time is actually spent.

**Good process:**
1. Define a performance goal.
2. Create a representative benchmark or workload.
3. Profile the program.
4. Optimize the bottleneck.
5. Measure again to confirm improvement.

**Tools:**
- `perf`
- Valgrind Callgrind
- compiler sanitizers
- benchmark frameworks
- tracing and logging

**Interview point:** Optimization without measurement can make code more complex without improving real performance.

---

## 2. Why does algorithmic complexity matter?

**Short answer:** Algorithmic complexity describes how runtime or memory grows as input size grows.

**Detailed answer:**
For large inputs, choosing a better algorithm often matters more than micro-optimizations. An `O(n log n)` algorithm can outperform an `O(n^2)` algorithm dramatically as data grows.

**Example:**

```cpp
// O(n^2): nested search
for (const auto& a : values) {
    for (const auto& b : values) {
        // compare
    }
}
```

Using a hash set may reduce a lookup-heavy problem from quadratic to near linear.

```cpp
std::unordered_set<int> seen;
```

**Interview point:** Big-O ignores constant factors, but it is still essential for understanding scalability.

**Common mistake:** Optimizing tiny operations while keeping a poor algorithmic design.

---

## 3. Why does cache locality matter in C++ performance?

**Short answer:** CPUs access nearby memory much faster than random memory due to caching.

**Detailed answer:**
Modern CPUs load memory in cache lines. Contiguous data structures such as `std::vector` often perform better than pointer-heavy structures because they improve spatial locality and reduce cache misses.

**Example:**

```cpp
std::vector<int> values;
for (int value : values) {
    sum += value;
}
```

This is cache-friendly because elements are contiguous.

A linked list may require following pointers to unrelated memory locations, causing many cache misses.

```cpp
std::list<int> values;
```

**Interview point:** `std::vector` is often faster than `std::list` even for workloads where linked-list insertion looks theoretically attractive.

---

## 4. What is branch prediction?

**Short answer:** Branch prediction is a CPU optimization that guesses which path a conditional branch will take.

**Detailed answer:**
Modern CPUs execute instructions speculatively. If a branch is predictable, execution is fast. If it is unpredictable, the CPU may discard speculative work and pay a penalty.

**Example:**

```cpp
for (int value : values) {
    if (value > threshold) {
        ++count;
    }
}
```

If the condition is random, branch prediction may perform poorly. If data is sorted or highly patterned, prediction may improve.

**Interview point:** Branch prediction matters in tight loops, but it should be optimized only after profiling.

**Common mistake:** Adding branchless tricks everywhere. They can hurt readability and may not improve performance.

---

## 5. What is inlining?

**Short answer:** Inlining replaces a function call with the function body at the call site.

**Detailed answer:**
Inlining can remove function call overhead and enable further optimization such as constant propagation and dead-code elimination. However, excessive inlining can increase binary size and hurt instruction cache performance.

**Example:**

```cpp
inline int square(int x) {
    return x * x;
}
```

The `inline` keyword does not force the compiler to inline the function. It also has linkage meaning in C++.

**Interview point:** Compilers are usually better than humans at deciding what to inline. Use profiling before forcing inline behavior with compiler-specific attributes.

**Common mistake:** Believing `inline` guarantees inlining.

---

## 6. What is undefined behavior, and why do compilers optimize based on it?

**Short answer:** Undefined behavior means the C/C++ standard imposes no requirements on what happens. Compilers assume it does not occur and optimize accordingly.

**Detailed answer:**
If code contains undefined behavior, the compiler may generate surprising results because it is allowed to assume impossible states never happen. This can make bugs appear only in optimized builds.

**Examples:**

```cpp
int* p = nullptr;
*p = 1; // undefined behavior
```

```cpp
int x = 2147483647;
++x; // signed integer overflow is undefined behavior
```

```cpp
int arr[3];
int value = arr[10]; // undefined behavior
```

**Interview point:** Undefined behavior is not simply "runtime error." It invalidates assumptions about the whole program.

---

## 7. What is implementation-defined behavior?

**Short answer:** Implementation-defined behavior is behavior where the standard allows multiple choices, but the compiler or platform must document which choice it uses.

**Detailed answer:**
Unlike undefined behavior, implementation-defined behavior is not arbitrary. It is controlled by the implementation and should be documented.

**Examples:**
- Size of fundamental types such as `int`.
- Signedness of plain `char`.
- Right shift behavior for negative signed integers.

**Example:**

```cpp
std::cout << sizeof(int) << '\n';
```

This may vary between platforms.

**Interview point:** Portable code avoids relying on implementation-defined behavior unless the target platform is explicitly known.

---

## 8. What is unspecified behavior?

**Short answer:** Unspecified behavior means the standard allows multiple possible behaviors and the implementation does not need to document which one occurs.

**Detailed answer:**
Unspecified behavior is still valid program behavior, unlike undefined behavior. However, portable code should not depend on one particular result.

**Example:**

```cpp
int f();
int g();

int result = f() + g();
```

The order of evaluation of `f()` and `g()` may be unspecified in many contexts.

**Interview point:** Unspecified behavior can cause portability bugs even though it is not undefined behavior.

**Common mistake:** Treating undefined, unspecified, and implementation-defined behavior as the same thing.

---

## 9. How can sanitizers help detect performance and correctness issues?

**Short answer:** Sanitizers instrument code to detect classes of bugs such as memory errors, undefined behavior, data races, and leaks.

**Detailed answer:**
Sanitizers are extremely useful during development and testing. They add runtime checks that catch issues that may otherwise appear only under rare conditions.

**Examples:**

```bash
g++ -fsanitize=address -g main.cpp -o app
```

```bash
g++ -fsanitize=undefined -g main.cpp -o app
```

```bash
g++ -fsanitize=thread -g main.cpp -o app
```

**Common sanitizers:**
- AddressSanitizer: out-of-bounds, use-after-free.
- UndefinedBehaviorSanitizer: many UB cases.
- ThreadSanitizer: data races.
- LeakSanitizer: memory leaks.

**Interview point:** Sanitizers help find correctness bugs; profilers help find performance bottlenecks. They solve different problems.

---

## 10. What is false sharing?

**Short answer:** False sharing occurs when multiple threads modify different variables that happen to share the same CPU cache line.

**Detailed answer:**
Even if threads are not logically sharing the same variable, the CPU cache coherence protocol works at cache-line granularity. If two hot variables are on the same cache line and different cores keep writing to them, the cache line may bounce between cores, hurting performance.

**Example concept:**

```cpp
struct Counters {
    std::atomic<int> a;
    std::atomic<int> b;
};
```

If one thread updates `a` and another updates `b`, they may still interfere if both are on the same cache line.

**Possible mitigation:**
Use padding or alignment for highly contended counters.

```cpp
struct alignas(64) PaddedCounter {
    std::atomic<int> value;
};
```

**Interview point:** False sharing is a performance issue, not a correctness issue. It should be addressed only when profiling shows contention.

---

## 11. What are common benchmarking mistakes in C++?

**Short answer:** Common mistakes include measuring unrealistic workloads, allowing the compiler to optimize away the work, and ignoring noise from the system.

**Detailed answer:**
Microbenchmarks can be misleading if they do not represent real program behavior. The compiler may remove unused calculations, constant-fold results, or inline everything differently from production code. Hardware effects such as CPU frequency scaling, cache warmup, and background processes can also distort results.

**Example problem:**

```cpp
void benchmark() {
    int result = expensiveComputation();
    // result is never used, so optimizer may remove the work
}
```

**Better practice:**
Use a benchmark framework, consume the result in a way the optimizer cannot remove, run many iterations, and compare against realistic inputs.

**Interview point:** A benchmark should answer a specific performance question, not just produce a number.

**Common mistake:** Trusting a single timing result from one run without checking variance or whether the measured code was optimized away.

---

## 12. Why can dynamic allocation be expensive?

**Short answer:** Dynamic allocation can involve allocator bookkeeping, synchronization, fragmentation, cache misses, and sometimes operating-system interaction.

**Detailed answer:**
Heap allocation is flexible but not free. Frequent small allocations can dominate runtime in hot paths. Allocated objects may also be scattered in memory, reducing cache locality.

**Example:**

```cpp
std::vector<std::unique_ptr<Node>> nodes;
for (int i = 0; i < count; ++i) {
    nodes.push_back(std::make_unique<Node>());
}
```

This creates many separate heap allocations. A contiguous container may be faster if object identity and stable addresses are not required.

```cpp
std::vector<Node> nodes;
nodes.reserve(count);
```

**Interview point:** Reducing allocation frequency, grouping lifetimes, reserving capacity, and using contiguous storage can often improve performance more than low-level instruction tricks.

**Common mistake:** Focusing only on CPU instructions while ignoring allocation behavior and memory layout.

---

## 13. What is small string optimization?

**Short answer:** Small string optimization is an implementation technique where short strings are stored inside the `std::string` object itself instead of allocating heap memory.

**Detailed answer:**
Many standard library implementations store small strings inline in the string object. This avoids heap allocation for short text and improves locality. The exact size limit is implementation-specific.

**Example:**

```cpp
std::string shortName = "Ada";
std::string longText = "a very long string that may require heap allocation";
```

The short string may not allocate, while the long string probably does.

**Interview point:** Small string optimization is useful to understand, but portable code must not depend on the exact inline capacity.

**Common mistake:** Assuming all `std::string` operations allocate memory. Many short-string operations may not allocate on common implementations.

---

## 14. What is SIMD and auto-vectorization?

**Short answer:** SIMD lets one CPU instruction operate on multiple data elements, and auto-vectorization is when the compiler transforms suitable loops to use SIMD instructions.

**Detailed answer:**
SIMD is useful for numeric loops, image processing, audio, compression, and other data-parallel workloads. Compilers can sometimes vectorize loops automatically when memory access is predictable and dependencies are clear.

**Example loop:**

```cpp
for (std::size_t i = 0; i < n; ++i) {
    output[i] = a[i] + b[i];
}
```

This loop is a good candidate for vectorization because each iteration is independent.

**Interview point:** Data layout, alignment, aliasing, and loop dependencies affect whether vectorization is possible.

**Common mistake:** Assuming the compiler can vectorize any loop. Pointer aliasing or hidden dependencies can prevent it.

---

## 15. What is strict aliasing?

**Short answer:** Strict aliasing is a rule that lets the compiler assume objects of unrelated types do not refer to the same memory.

**Detailed answer:**
The compiler uses aliasing rules for optimization. Reading or writing an object through an incompatible pointer type can cause undefined behavior, except for special cases such as character types inspecting object representation.

**Bad example:**

```cpp
float value = 1.0f;
int* bits = reinterpret_cast<int*>(&value);
int raw = *bits; // undefined behavior due to strict aliasing
```

**Better C++20 approach:**

```cpp
#include <bit>

float value = 1.0f;
int raw = std::bit_cast<int>(value);
```

**Interview point:** Strict aliasing bugs often appear only with optimization enabled because the optimizer relies on aliasing assumptions.

**Common mistake:** Using `reinterpret_cast` as if it makes any type-punning operation valid.

---

## 16. What is profile-guided optimization?

**Short answer:** Profile-guided optimization uses runtime profile data to help the compiler optimize for real execution paths.

**Detailed answer:**
With profile-guided optimization, the program is first built with instrumentation, then run with representative workloads. The compiler uses the collected profile to make better decisions about inlining, branch layout, code placement, and other optimizations.

**Typical flow:**
```text
build instrumented binary -> run representative workload -> rebuild using profile data
```

**Benefits:**
- Better branch prediction layout
- More informed inlining decisions
- Improved hot/cold code separation
- Potential speedups without source-level complexity

**Interview point:** PGO is only as good as the workload used to collect profile data. Misleading profiles can optimize the wrong paths.

**Common mistake:** Using synthetic or tiny workloads and assuming the resulting profile represents production behavior.

---

## 17. What is link-time optimization?

**Short answer:** Link-time optimization lets the compiler optimize across translation unit boundaries during linking.

**Detailed answer:**
Normally, each source file is optimized mostly independently. Link-time optimization keeps more intermediate representation until link time, allowing the optimizer to inline across files, remove unused code, and see whole-program information.

**Example command idea:**

```bash
g++ -O2 -flto main.cpp util.cpp -o app
```

**Benefits:**
- Cross-translation-unit inlining
- Better dead-code elimination
- More global optimization opportunities

**Tradeoffs:**
- Longer link times
- More memory usage during builds
- Possible toolchain compatibility issues

**Interview point:** LTO can improve performance without changing source code, but it affects build time and deployment toolchains.

**Common mistake:** Expecting LTO to fix poor algorithms or bad memory layout.

---

## 18. How does data-oriented design improve performance?

**Short answer:** Data-oriented design organizes data around access patterns so the CPU does less unnecessary work.

**Detailed answer:**
Object-oriented layouts sometimes scatter related data across many allocations. Data-oriented design asks which data is processed together and stores it contiguously when possible. This improves cache locality, vectorization, and memory bandwidth use.

**Example contrast:**

```cpp
struct Entity {
    float x;
    float y;
    float velocityX;
    float velocityY;
};

std::vector<Entity> entities;
```

If a loop only updates positions, a structure-of-arrays layout may be better.

```cpp
struct Positions {
    std::vector<float> x;
    std::vector<float> y;
};
```

**Interview point:** Data layout can matter more than individual instruction choices in hot loops.

**Common mistake:** Treating performance as only a CPU instruction problem instead of a data movement problem.

---

## 19. What is integer overflow, and why is signed overflow dangerous?

**Short answer:** Unsigned integer overflow wraps modulo the type range, but signed integer overflow is undefined behavior in C and C++.

**Detailed answer:**
Compilers assume signed overflow does not happen and may optimize based on that assumption. This can break checks written after an overflow has already occurred.

**Bad example:**

```cpp
int bytes = count * size;
if (bytes < count) {
    // too late: signed overflow may already be undefined behavior
}
```

**Safer approach:**
Check before multiplying or use a wider type when appropriate.

```cpp
if (count > max / size) {
    // would overflow
}
```

**Interview point:** Integer overflow can be both a correctness issue and a security issue, especially in allocation size calculations.

**Common mistake:** Assuming signed integers wrap like unsigned integers on every platform and optimization level.

---

## 20. What is dead-code elimination?

**Short answer:** Dead-code elimination removes code whose results cannot affect observable program behavior.

**Detailed answer:**
Optimizing compilers remove computations, branches, and stores that have no observable effect. This is usually good, but it can make benchmarks invalid if the measured work is unused.

**Example:**

```cpp
int compute();

void test() {
    int value = compute();
}
```

If `value` is never used and `compute` has no observable side effects, the optimizer may remove the call entirely.

**Benchmark implication:**
Use a reliable benchmark framework or consume results in a way that prevents the optimizer from removing the work being measured.

**Interview point:** The as-if rule lets the compiler transform code as long as observable behavior is preserved.

**Common mistake:** Timing code that the optimizer has partially or completely eliminated.

---

## 21. What is prefetching, and when can it help performance?

**Short answer:** Prefetching brings data into cache before it is needed, reducing memory-latency stalls when access patterns are predictable.

**Detailed answer:**
Modern CPUs perform hardware prefetching automatically for common patterns such as sequential memory access. Manual prefetching asks the CPU to start loading a memory address before the program uses it.

Manual prefetching can help in low-level performance-critical code with predictable future accesses, but it can also hurt by wasting bandwidth, polluting caches, or making code harder to maintain.

**Conceptual example:**

```cpp
for (std::size_t i = 0; i < count; ++i) {
    __builtin_prefetch(&items[i + 16]);
    process(items[i]);
}
```

This is compiler-specific and must be benchmarked carefully.

**Interview point:** Prefer improving data layout and access patterns before adding manual prefetch instructions.

**Common mistake:** Adding prefetching without profiling and assuming it always improves memory-bound code.

---

## 22. What is memory bandwidth, and how is it different from latency?

**Short answer:** Latency is the delay to get one piece of data; bandwidth is the amount of data that can be transferred per unit time.

**Detailed answer:**
A program may be limited by latency when it follows unpredictable pointer chains where each load depends on the previous one. It may be limited by bandwidth when it streams large amounts of data and the CPU cannot receive bytes fast enough.

Optimization strategies differ. Latency problems often improve with better locality, fewer dependencies, or batching. Bandwidth problems often improve by reading less data, using compact layouts, avoiding extra passes, or improving cache reuse.

**Example:**

```cpp
struct Node {
    Node* next;
    int value;
};

int sumList(Node* node) {
    int sum = 0;
    while (node) {
        sum += node->value;
        node = node->next; // pointer chasing: latency-heavy
    }
    return sum;
}
```

A contiguous vector traversal is often easier for hardware to prefetch and can use memory bandwidth more efficiently.

**Interview point:** Performance tuning should identify whether the bottleneck is latency, bandwidth, computation, allocation, or synchronization.

**Common mistake:** Optimizing arithmetic instructions when the real cost is waiting for memory.

---

## 23. What is devirtualization?

**Short answer:** Devirtualization is an optimization where the compiler replaces a virtual call with a direct call when it can prove the dynamic target.

**Detailed answer:**
Virtual calls normally require runtime dispatch through a vtable-like mechanism. If the compiler knows the exact dynamic type, it may call the target directly and then inline it. This can remove dispatch overhead and enable more optimization.

**Example:**

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual int value() const = 0;
};

class Derived final : public Base {
public:
    int value() const override { return 42; }
};

int read(const Derived& d) {
    return d.value(); // compiler knows the exact type
}
```

Marking classes or functions `final`, using whole-program optimization, and keeping dynamic types visible can help devirtualization.

**Interview point:** Virtual dispatch cost is often less important than the optimization barriers it may create.

**Common mistake:** Removing virtual functions for performance without measuring or understanding whether dispatch is actually the bottleneck.

---

## 24. What is cache-friendly data layout?

**Short answer:** Cache-friendly data layout stores data so the program reads useful nearby bytes and avoids loading unnecessary data.

**Detailed answer:**
CPUs load memory in cache lines, not individual fields. If a loop only needs a few fields but objects are large and scattered, the CPU may waste cache capacity and bandwidth. Struct-of-arrays, compact representations, and separating hot from cold data can improve locality.

**Array-of-structs:**

```cpp
struct Particle {
    float x, y, z;
    float vx, vy, vz;
    std::string debugName;
};

std::vector<Particle> particles;
```

If a loop only updates positions and velocities, `debugName` is cold data mixed with hot data.

**Struct-of-arrays idea:**

```cpp
struct Particles {
    std::vector<float> x, y, z;
    std::vector<float> vx, vy, vz;
};
```

**Interview point:** Data-oriented optimization often starts by matching layout to access patterns.

**Common mistake:** Designing object layout around conceptual hierarchy while ignoring the loops that dominate runtime.

---

## 25. What is the performance impact of exceptions?

**Short answer:** In many C++ implementations, exceptions have near-zero cost when not thrown but can be expensive when thrown.

**Detailed answer:**
Many C++ ABIs use zero-cost exception handling, where normal execution does not check error codes after every call. Instead, metadata is used when an exception is thrown to unwind the stack and run destructors.

This means exceptions are usually unsuitable for hot-path expected control flow, but they can be appropriate for rare error paths where stack unwinding and context propagation are valuable.

**Example guidance:**

```cpp
// Good fit: rare failure
Config loadRequiredConfig(); // may throw if configuration cannot be loaded

// Poor fit: ordinary branch in a tight parser loop
// throw ParseMiss{} for every non-matching character;
```

**Interview point:** The cost model depends on implementation and workload. Measure if exception behavior is performance-critical.

**Common mistake:** Saying "exceptions are always slow" without distinguishing thrown and non-thrown paths.

---

## 26. What is Amdahl's Law?

**Short answer:** Amdahl's Law says the maximum speedup of a program is limited by the part that cannot be improved.

**Detailed answer:**
If only part of a program is optimized or parallelized, the unchanged part becomes the limiting factor. For example, if 80% of runtime can be made infinitely fast, the remaining 20% still limits total speedup to 5x.

**Conceptual formula:**

```text
speedup = 1 / ((1 - P) + P / S)
```

`P` is the fraction improved, and `S` is the speedup for that fraction.

**Interview point:** Optimize the dominant cost first, and remember that local improvements may not move total runtime much.

**Common mistake:** Spending time optimizing code that is not significant in the overall profile.

---

## 27. What is tail latency?

**Short answer:** Tail latency is the high-percentile response time, such as p95 or p99 latency, rather than the average latency.

**Detailed answer:**
Average latency can hide rare but important slow requests. In production systems, p95, p99, or p999 latency often matters because users and upstream systems experience slow outliers. Tail latency can come from cache misses, lock contention, scheduling delays, garbage collection in other languages, I/O stalls, page faults, or queueing.

**Example metrics:**

```text
average: 12 ms
p95:     40 ms
p99:    250 ms
```

The average looks healthy, but the slowest 1% may be unacceptable.

**Interview point:** Performance goals should specify the percentile and workload, not just "make it faster."

**Common mistake:** Optimizing average throughput while making p99 latency worse.

---

## 28. What is throughput vs latency?

**Short answer:** Throughput is work completed per unit time; latency is how long one operation takes.

**Detailed answer:**
A system can have high throughput but poor latency if work sits in queues. Batching often improves throughput by reducing overhead per item, but it can increase latency for individual requests. Low-latency systems may accept lower total throughput to reduce waiting time.

**Example:**

```text
Throughput: 100,000 messages/second
Latency: 5 milliseconds/message at p95
```

Both metrics are needed to understand performance.

**Interview point:** Optimization decisions depend on whether the goal is throughput, latency, or tail latency.

**Common mistake:** Reporting only operations per second when users care about response time.

---

## 29. What is warm-up in benchmarking?

**Short answer:** Warm-up is the initial execution period before measurements stabilize.

**Detailed answer:**
Even in C++, benchmark results can change after caches warm up, branch predictors learn patterns, pages are faulted in, dynamic libraries are loaded, CPU frequency changes, and memory allocators initialize. Measuring cold-start behavior can be valid, but it should be intentional.

**Example approach:**

```text
run setup
run warm-up iterations
measure steady-state iterations
report distribution
```

A good benchmark separates setup cost, cold-start cost, and steady-state cost.

**Interview point:** Benchmark methodology matters as much as the measured code.

**Common mistake:** Timing a single run and treating it as representative steady-state performance.

---

## 30. What is measurement noise in performance testing?

**Short answer:** Measurement noise is unrelated variation that affects benchmark results.

**Detailed answer:**
Noise can come from OS scheduling, background processes, CPU frequency scaling, thermal throttling, page faults, interrupts, allocator state, cache state, and input variation. Good benchmarking repeats measurements, reports distributions, pins or isolates resources when necessary, and compares against a baseline.

**Example report style:**

```text
baseline:  12.4 ms median, 14.1 ms p95
candidate: 10.8 ms median, 13.9 ms p95
```

This is more useful than one timing number.

**Interview point:** Small speedups are not meaningful unless they exceed measurement noise.

**Common mistake:** Claiming a 2% improvement from one noisy local run.

---

## 31. What is memory access stride?

**Short answer:** Memory access stride is the distance between consecutive memory locations accessed by a loop.

**Detailed answer:**
Sequential stride-1 access is cache-friendly and easy for hardware prefetchers. Large or irregular strides can waste cache lines and reduce memory bandwidth. Matrix traversal order is a common example.

**Example:**

```cpp
for (std::size_t row = 0; row < rows; ++row) {
    for (std::size_t col = 0; col < cols; ++col) {
        sum += matrix[row * cols + col]; // contiguous access
    }
}
```

Reversing the loop order may access memory with a large stride.

**Interview point:** Big-O can be identical while memory access patterns produce very different runtimes.

**Common mistake:** Ignoring data layout because the algorithmic complexity did not change.

---

## 32. What is TLB pressure?

**Short answer:** TLB pressure happens when a program touches many virtual pages and exceeds the CPU's translation cache capacity.

**Detailed answer:**
The Translation Lookaside Buffer caches virtual-to-physical address translations. Code that jumps across many pages can suffer TLB misses, adding latency even if data is otherwise in memory. Large datasets, pointer-heavy structures, and random access patterns can increase TLB pressure.

**Conceptual example:**

```text
linked list nodes spread across many pages -> many address translations
contiguous vector storage -> fewer pages touched for the same data count
```

Huge pages can reduce TLB pressure in some workloads, but they come with tradeoffs.

**Interview point:** Memory performance includes address translation, not only cache lines.

**Common mistake:** Treating all main-memory accesses as having the same cost.

---

## 33. What is branchless programming?

**Short answer:** Branchless programming avoids conditional branches, often to reduce branch misprediction in hot code.

**Detailed answer:**
Modern CPUs predict branches. When prediction fails, the pipeline loses work. In some tight loops, replacing unpredictable branches with arithmetic, masks, or conditional moves can help. But branchless code can be harder to read and may do extra work.

**Example:**

```cpp
int minValue(int a, int b) {
    return a < b ? a : b;
}
```

Compilers may already lower simple conditionals to branchless instructions when profitable.

**Interview point:** Branchless code is useful when branches are unpredictable and the code is hot.

**Common mistake:** Manually making code branchless without checking whether the compiler already did it or whether it improves performance.

---

## 34. What is loop unrolling?

**Short answer:** Loop unrolling duplicates loop body work to reduce loop overhead and expose more optimization opportunities.

**Detailed answer:**
Unrolling can reduce branch and index-update overhead and help instruction scheduling or vectorization. It can also increase code size, hurt instruction-cache locality, and make maintenance harder. Compilers often perform automatic unrolling when optimization is enabled.

**Example idea:**

```cpp
for (std::size_t i = 0; i + 3 < n; i += 4) {
    sum += values[i];
    sum += values[i + 1];
    sum += values[i + 2];
    sum += values[i + 3];
}
```

Manual unrolling should be justified by measurement.

**Interview point:** More instructions in source code do not always mean faster machine code.

**Common mistake:** Manually unrolling loops in non-hot code and increasing complexity for no measurable benefit.

---

## 35. What is escape analysis?

**Short answer:** Escape analysis determines whether an object's address or lifetime escapes a local scope.

**Detailed answer:**
If the compiler can prove an object does not escape, it may optimize allocation, eliminate copies, keep values in registers, or remove synchronization. C++ compilers use many related analyses even if they do not expose them under one user-facing feature name.

**Example:**

```cpp
int compute() {
    Point p{1, 2};
    return p.x + p.y;
}
```

The object can often be optimized away entirely.

**Interview point:** Clear ownership and limited aliasing make optimization easier.

**Common mistake:** Taking addresses unnecessarily and making it harder for the compiler to prove local behavior.

---

## 36. How can aliasing affect optimization?

**Short answer:** If two pointers or references might refer to the same object, the compiler must preserve behavior for that possibility.

**Detailed answer:**
Aliasing can prevent the compiler from reordering loads and stores or keeping values in registers. C has `restrict` for some no-alias promises. C++ relies on type-based aliasing rules, compiler analysis, and code structure.

**Example:**

```cpp
void add(int* a, int* b) {
    *a += 1;
    *b += 1;
}
```

If `a` and `b` might point to the same `int`, the result and ordering must respect that possibility.

**Interview point:** Alias information is a major input to optimizer decisions.

**Common mistake:** Assuming pointer-heavy code optimizes like code over local values.

---

## 37. What is signed integer overflow, and why does it matter for optimization?

**Short answer:** Signed integer overflow is undefined behavior in C++, so the compiler may assume it never happens.

**Detailed answer:**
Because signed overflow is undefined, optimizers can transform code under the assumption that arithmetic stays within range. This enables useful optimizations but can surprise programmers who expect wraparound behavior. Unsigned integer arithmetic wraps modulo 2^N, but that does not make every unsigned use correct.

**Example:**

```cpp
int f(int x) {
    return x + 1 > x; // compiler may assume true if signed overflow cannot occur
}
```

For intentional wrapping, use unsigned types or dedicated checked/wrapping arithmetic utilities.

**Interview point:** Undefined behavior is not just a runtime risk; it changes what the optimizer is allowed to assume.

**Common mistake:** Relying on two's-complement wraparound for signed integers in portable C++.

---

## 38. What is object lifetime optimization and why can it be dangerous with UB?

**Short answer:** The optimizer reasons about when objects begin and end lifetime; accessing objects outside their lifetime is undefined and can lead to surprising transformations.

**Detailed answer:**
C++ object lifetime rules affect aliasing, storage reuse, placement new, and destructor behavior. If code reads an object after its lifetime ended or before a new object's lifetime began, the optimizer can assume that invalid access does not occur.

**Example risk:**

```cpp
alignas(Widget) unsigned char storage[sizeof(Widget)];
auto* w = new (storage) Widget();
w->~Widget();
// w->use(); // undefined: lifetime ended
```

Low-level storage reuse should use correct lifetime management and facilities such as placement new and `std::launder` when required.

**Interview point:** Lifetime rules are part of the optimizer's model, not only a memory-management concern.

**Common mistake:** Treating raw storage bytes as if any object can be accessed there at any time.

---

## 39. What is performance portability?

**Short answer:** Performance portability means code performs reasonably well across different compilers, CPUs, operating systems, and workloads.

**Detailed answer:**
An optimization that helps one platform can hurt another. Cache sizes, SIMD width, branch predictors, memory bandwidth, OS scheduling, compiler heuristics, and ABI choices differ. Portable performance often comes from good algorithms, data layout, clear ownership, and measurement on target platforms.

**Example tradeoff:**

```text
manual SIMD path: fastest on one CPU family
compiler-vectorized generic path: good enough across several targets
```

**Interview point:** Production optimization should consider deployment diversity, not just one developer machine.

**Common mistake:** Hard-coding a micro-optimization based on a single benchmark environment.

---

## 40. How should you approach performance work in a mature C++ system?

**Short answer:** Define the goal, measure the current behavior, identify the bottleneck, change one thing, and verify correctness and performance.

**Detailed answer:**
Performance work should be systematic. Start with a specific goal such as throughput, latency, memory usage, binary size, startup time, or CPU cost. Use profiling and representative workloads to find the limiting factor. After changing code, compare against a baseline and check for regressions.

**Practical workflow:**

- Define the metric and target.
- Reproduce the workload.
- Profile before changing code.
- Prefer algorithmic and data-layout improvements first.
- Keep correctness tests running.
- Measure before and after with enough repetitions.
- Watch for tradeoffs in memory, readability, and tail latency.

**Interview point:** Senior performance work is evidence-driven and includes rollback criteria.

**Common mistake:** Starting with clever low-level changes before proving where the time or memory goes.

---

## 41. What is the difference between latency, throughput, and utilization?

**Short answer:** Latency measures how long one operation takes, throughput measures how many operations complete per unit time, and utilization measures how busy a resource is.

**Detailed answer:**
A system can have high throughput but poor latency if work queues up. It can also have high CPU utilization without doing useful work, such as when threads spin or contend on locks. Performance goals should name the metric being optimized because improving one metric may worsen another.

**Example tradeoff:**

```text
Batching requests:
  improves throughput
  may increase individual request latency
```

For interactive systems, tail latency often matters more than average throughput.

**Interview point:** "Make it faster" is not precise enough; define the performance metric and workload first.

**Common mistake:** Optimizing CPU utilization as if 100% busy always means efficient work.

---

## 42. What is p95 or p99 latency?

**Short answer:** p95 or p99 latency is the latency below which 95% or 99% of requests complete.

**Detailed answer:**
Average latency can hide rare but painful slow requests. A service with a low average may still have unacceptable p99 latency if some requests wait on locks, page faults, garbage collection in another component, disk stalls, or network retries. Tail latency is especially important for user-facing and distributed systems.

**Example:**

```text
p50:  10 ms
p95:  80 ms
p99: 300 ms
```

Only 1% of requests exceed 300 ms, but that may still be too many at scale.

**Interview point:** Tail latency reveals outliers that averages hide.

**Common mistake:** Reporting only average latency after a performance change.

---

## 43. What is profile-guided optimization different from manual optimization?

**Short answer:** Profile-guided optimization uses runtime profile data to guide compiler decisions, while manual optimization changes source code directly.

**Detailed answer:**
Profile-guided optimization can improve inlining, branch layout, code placement, and indirect-call decisions using representative training runs. Manual optimization may improve algorithms or data layout but can also make code harder to maintain. Both require representative workloads and before/after measurement.

**Conceptual flow:**

```text
build instrumented binary -> run representative workload -> rebuild using profile data
```

If the profile workload is unrepresentative, PGO can optimize for the wrong behavior.

**Interview point:** PGO is powerful because it gives the compiler real execution-frequency data.

**Common mistake:** Using a tiny synthetic workload as the profile source for a complex production system.

---

## 44. What is the optimizer allowed to assume after undefined behavior?

**Short answer:** If a program has undefined behavior, the optimizer may assume that execution path never occurs.

**Detailed answer:**
Undefined behavior is not just an unpredictable runtime result. It changes the compiler's reasoning model. For example, if signed overflow is undefined, the compiler can optimize under the assumption that signed overflow does not happen. This can remove checks, reorder code, or transform loops in surprising ways.

**Example:**

```cpp
int addPositive(int a, int b) {
    int sum = a + b;
    if (sum < a) {
        return -1; // compiler may assume this cannot detect signed overflow
    }
    return sum;
}
```

Use checked arithmetic or unsigned arithmetic when wraparound is intended.

**Interview point:** UB gives the compiler freedom, so relying on observed behavior in one build is unsafe.

**Common mistake:** Testing a UB case once and treating the observed output as a language guarantee.

---

## 45. What is bounds-check elimination?

**Short answer:** Bounds-check elimination removes redundant range checks when the compiler or runtime can prove an access is safe.

**Detailed answer:**
In C++, unchecked operations such as `operator[]` avoid bounds checks, while checked functions such as `.at()` validate at runtime. More generally, compilers can remove checks or simplify branches when loop structure and invariants prove they are unnecessary. Clear loop bounds and simple invariants help optimization.

**Example:**

```cpp
for (std::size_t i = 0; i < values.size(); ++i) {
    use(values[i]);
}
```

The loop condition directly bounds the index.

**Interview point:** Code that makes invariants clear is easier for both humans and compilers to optimize safely.

**Common mistake:** Removing safety checks manually before measuring their cost or proving they are redundant.

---

## 46. What is allocation elision or allocation avoidance?

**Short answer:** It is reducing or eliminating dynamic memory allocations by changing ownership, storage, or data-flow design.

**Detailed answer:**
Allocations can cost CPU time, synchronization, cache misses, and fragmentation. Avoidance techniques include reserving capacity, reusing buffers, using stack storage for small fixed-size data, passing views instead of owning strings, and changing algorithms to stream results rather than build temporary containers.

**Example:**

```cpp
std::vector<int> values;
values.reserve(expectedCount);

for (int x : input) {
    values.push_back(transform(x));
}
```

This avoids repeated reallocations while keeping ownership clear.

**Interview point:** Allocation optimization should preserve lifetime clarity; avoiding allocations by using dangling views is not a win.

**Common mistake:** Replacing safe owning objects with raw buffers and introducing lifetime bugs.

---

## 47. What is cache line alignment?

**Short answer:** Cache line alignment places data at boundaries that match the CPU cache line size or avoids unwanted sharing within a line.

**Detailed answer:**
A cache line is the unit transferred between memory and cache. Aligning frequently accessed data can help SIMD or reduce split-line accesses. Padding or aligning independent hot variables can reduce false sharing between threads. However, excessive padding increases memory footprint and can hurt cache capacity.

**Example:**

```cpp
struct alignas(64) Counter {
    std::atomic<int> value;
};
```

The exact useful alignment depends on hardware and access patterns.

**Interview point:** Alignment is a workload-specific optimization, not a universal rule.

**Common mistake:** Adding padding everywhere without measuring memory footprint and cache behavior.

---

## 48. What is instruction cache pressure?

**Short answer:** Instruction cache pressure occurs when the CPU spends more time fetching code because the hot code path is too large or poorly laid out.

**Detailed answer:**
Inlining, templates, unrolled loops, and large switch statements can improve performance in some cases but also increase code size. If hot code no longer fits well in instruction cache, performance can degrade. This is one reason smaller, simpler code can sometimes be faster.

**Example tradeoff:**

```text
More inlining:
  fewer function calls
  larger binary and possible instruction-cache misses
```

**Interview point:** Optimization has a code-size dimension, not only a CPU-instruction-count dimension.

**Common mistake:** Forcing inline or unrolling everything without checking binary size and instruction-cache effects.

---

## 49. What is deoptimization of maintainability?

**Short answer:** It is when a performance change makes code significantly harder to understand, test, or modify without a proven performance benefit.

**Detailed answer:**
Some optimizations are justified because they solve measured bottlenecks. Others add cleverness, platform assumptions, unsafe lifetime tricks, or duplicated logic with little evidence. In mature systems, maintainability is part of performance engineering because future bugs and regressions are costly.

**Example warning signs:**

- Manual memory management replacing simple RAII without a measured need.
- Hand-written branchless code that hides correctness rules.
- Duplicated fast paths that are not tested like the normal path.
- Platform-specific intrinsics without a portable fallback or benchmark evidence.

**Interview point:** A senior engineer asks whether the speedup is real enough to justify the complexity.

**Common mistake:** Treating harder-to-read code as automatically more optimized.

---

## 50. How do you review performance-sensitive C++ code?

**Short answer:** Check the evidence, correctness, data layout, algorithmic complexity, resource usage, and long-term maintainability.

**Detailed answer:**
A performance review should start with the benchmark or profile that motivated the change. Then verify that the workload is representative, the measurements are stable, and correctness tests still cover the changed behavior. The review should also consider memory use, tail latency, portability, and whether the code relies on undefined behavior.

**Review checklist:**

- What metric improved: latency, throughput, memory, startup time, or binary size?
- Is the workload representative of real use?
- Are before/after results statistically meaningful?
- Does the change preserve defined behavior and object lifetimes?
- Does it improve the actual bottleneck shown by profiling?
- Are cache, allocation, and branch effects considered where relevant?
- Is the added complexity justified by the measured gain?

**Interview point:** Performance code review is evidence review plus correctness review.

**Common mistake:** Approving an optimization because it looks plausible without requiring a reproducible measurement.
