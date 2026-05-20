# Memory Management Interview Questions

Memory management is one of the most important interview topics for C and C++. Interviewers use it to evaluate whether a candidate understands ownership, lifetime, allocation strategies, resource cleanup, and common sources of undefined behavior.

## 1. What is the difference between `malloc/free` and `new/delete`?

**Short answer:** `malloc/free` allocate and release raw memory. `new/delete` allocate memory and also construct or destroy C++ objects.

**Detailed answer:**
`malloc` is a C library function. It returns a `void*` pointing to uninitialized raw memory and does not call constructors. `free` releases memory but does not call destructors.

`new` is a C++ operator. It allocates memory and calls the object's constructor. `delete` calls the destructor and then releases the memory.

**Example:**

```cpp
#include <cstdlib>
#include <string>

int* a = static_cast<int*>(std::malloc(sizeof(int)));
*a = 10;
std::free(a);

std::string* name = new std::string("Ada");
delete name;
```

**Important rule:** Do not mix allocation and deallocation families.

```cpp
int* p = new int(5);
// std::free(p); // wrong: undefined behavior
delete p;
```

**Common mistake:** Saying `malloc` and `new` are equivalent. They are not equivalent for objects with constructors, destructors, or invariants.

---

## 2. What is a memory leak?

**Short answer:** A memory leak happens when dynamically allocated memory is no longer reachable and therefore cannot be released.

**Detailed answer:**
A leak usually occurs when a program allocates memory but loses the pointer to it or forgets to release it. In long-running programs, leaks can gradually increase memory usage until performance degrades or the process is killed.

**Bad example:**

```cpp
void leak() {
    int* values = new int[100];
    // no delete[]
}
```

**Better C++ approach:**

```cpp
#include <vector>

void noLeak() {
    std::vector<int> values(100);
}
```

**Interview point:** In modern C++, raw owning pointers should be rare. Prefer standard containers, `std::unique_ptr`, and `std::shared_ptr` when ownership is dynamic.

**Common mistake:** Thinking leaks only matter at program exit. They are especially serious in servers, embedded systems, games, and services that run continuously.

---

## 3. What is double free?

**Short answer:** Double free means releasing the same dynamically allocated memory more than once.

**Detailed answer:**
After memory is released, the allocator may reuse it for something else. Freeing it again corrupts allocator state or causes undefined behavior. It may crash immediately, or it may cause a hard-to-debug failure later.

**Bad example:**

```cpp
int* p = new int(10);
delete p;
delete p; // undefined behavior
```

**Better pattern:**

```cpp
#include <memory>

auto p = std::make_unique<int>(10);
```

`std::unique_ptr` automatically deletes the object exactly once when it goes out of scope.

**Common mistake:** Setting `p = nullptr` after `delete` only protects that one pointer variable. Other aliases may still point to the freed memory.

---

## 4. What is the difference between `delete` and `delete[]`?

**Short answer:** Use `delete` for a single object allocated with `new`; use `delete[]` for arrays allocated with `new[]`.

**Detailed answer:**
`delete` calls the destructor for one object. `delete[]` calls destructors for each element in the array and releases the array allocation. Using the wrong form is undefined behavior.

**Example:**

```cpp
int* one = new int(5);
delete one;

int* many = new int[5];
delete[] many;
```

**For class types:**

```cpp
struct Item {
    ~Item() {
        // cleanup
    }
};

Item* items = new Item[3];
delete[] items; // calls destructor for all 3 elements
```

**Modern C++ recommendation:** Prefer `std::vector<T>` for dynamic arrays.

```cpp
std::vector<int> values(5);
```

**Common mistake:** Believing `delete` and `delete[]` differ only stylistically. They have different semantics.

---

## 5. What is ownership in C++?

**Short answer:** Ownership means responsibility for releasing a resource.

**Detailed answer:**
A resource can be memory, a file handle, socket, mutex, database connection, or any object that must be cleaned up. Good C++ code makes ownership clear so that resources are released exactly once.

**Examples of ownership:**

```cpp
#include <memory>

std::unique_ptr<int> createValue() {
    return std::make_unique<int>(42); // caller receives ownership
}
```

Here, the returned `std::unique_ptr` owns the integer. When the pointer is destroyed, the integer is automatically deleted.

**Non-owning pointer example:**

```cpp
void printValue(const int* value) {
    if (value) {
        std::cout << *value << '\n';
    }
}
```

This function observes the value but does not own it.

**Interview tip:** Be explicit when a pointer is owning or non-owning. Raw pointers are usually best used as non-owning observers in modern C++.

---

## 6. What is RAII?

**Short answer:** RAII means Resource Acquisition Is Initialization. A resource is acquired in an object's constructor and released in its destructor.

**Detailed answer:**
RAII is a core C++ technique for safe resource management. It ties resource lifetime to object lifetime, which makes cleanup automatic even when exceptions or early returns occur.

**Example:**

```cpp
#include <fstream>
#include <string>

void writeLog(const std::string& message) {
    std::ofstream file("log.txt");
    file << message << '\n';
} // file is closed automatically here
```

`std::ofstream` owns the file handle. Its destructor closes the file.

**RAII with memory:**

```cpp
#include <memory>

auto value = std::make_unique<int>(10);
```

The `std::unique_ptr` destructor releases the memory.

**Common mistake:** Thinking RAII is only about memory. It applies to all resources that need cleanup.

---

## 7. What is the difference between `std::unique_ptr` and `std::shared_ptr`?

**Short answer:** `std::unique_ptr` represents exclusive ownership. `std::shared_ptr` represents shared ownership through reference counting.

**Detailed answer:**
Use `std::unique_ptr` when one owner is responsible for the object. It is lightweight and clearly communicates ownership. Use `std::shared_ptr` when multiple independent owners must keep an object alive.

**Example:**

```cpp
#include <memory>

std::unique_ptr<int> uniqueValue = std::make_unique<int>(10);

std::shared_ptr<int> sharedA = std::make_shared<int>(20);
std::shared_ptr<int> sharedB = sharedA;
```

When the last `shared_ptr` owning the object is destroyed, the object is deleted.

**Interview tip:** Default to `unique_ptr` unless shared ownership is truly required.

**Common mistake:** Using `shared_ptr` everywhere because it seems safer. It can hide ownership design problems and has overhead.

---

## 8. What problem does `std::weak_ptr` solve?

**Short answer:** `std::weak_ptr` observes an object managed by `std::shared_ptr` without increasing the reference count.

**Detailed answer:**
`std::weak_ptr` is mainly used to break ownership cycles. If two objects hold `shared_ptr`s to each other, their reference counts may never reach zero, causing a memory leak.

**Example cycle problem:**

```cpp
#include <memory>

struct B;

struct A {
    std::shared_ptr<B> b;
};

struct B {
    std::shared_ptr<A> a; // cycle if A and B point to each other
};
```

**Better design:**

```cpp
struct B {
    std::weak_ptr<A> a; // observes A without owning it
};
```

To use a `weak_ptr`, call `lock()` to get a temporary `shared_ptr` if the object is still alive.

```cpp
if (auto owner = weak.lock()) {
    // safe to use owner
}
```

**Common mistake:** Thinking `weak_ptr` is just a raw pointer replacement. It is specifically tied to `shared_ptr` control blocks.

---

## 9. What is placement new?

**Short answer:** Placement new constructs an object in memory that has already been allocated.

**Detailed answer:**
Normal `new` allocates memory and constructs an object. Placement new only constructs the object at a specified memory address. It is used in allocators, memory pools, embedded systems, and performance-sensitive code.

**Example:**

```cpp
#include <new>
#include <string>

alignas(std::string) unsigned char storage[sizeof(std::string)];

std::string* text = new (storage) std::string("hello");

text->~basic_string(); // destructor must be called manually
```

**Important rule:** Since placement new does not allocate memory, you must not call `delete` on the object. You must manually call the destructor if needed.

**Common mistake:** Forgetting manual destruction after placement new. That can leak resources held inside the object.

---

## 10. What tools can help detect memory errors?

**Short answer:** Common tools include AddressSanitizer, LeakSanitizer, Valgrind, compiler warnings, and static analyzers.

**Detailed answer:**
Memory bugs can be hard to reproduce because undefined behavior may appear only under certain builds or workloads. Tools help detect invalid reads/writes, use-after-free, leaks, double free, and stack buffer overflows.

**AddressSanitizer example:**

```bash
g++ -fsanitize=address -g main.cpp -o app
./app
```

**LeakSanitizer example:**

```bash
g++ -fsanitize=address,leak -g main.cpp -o app
./app
```

**Valgrind example:**

```bash
valgrind --leak-check=full ./app
```

**Interview point:** Tools are not a replacement for understanding ownership and lifetime, but they are essential in real-world debugging.

**Common mistake:** Only testing with optimized release builds. Debug symbols and sanitizers are usually better for diagnosing memory errors.

---

## 11. What is memory fragmentation?

**Short answer:** Memory fragmentation happens when free memory exists but is split into pieces that are not useful for future allocation requests.

**Detailed answer:**
Fragmentation can be external or internal. External fragmentation means free memory is spread across many small blocks, making it hard to satisfy a large allocation. Internal fragmentation means allocated blocks are larger than requested due to allocator size classes, alignment, or padding.

**Example scenario:**
A long-running service repeatedly allocates and frees objects of different sizes. Over time, the heap may contain many small gaps. Total free memory may look sufficient, but large contiguous allocations may fail or become slower.

**Mitigation strategies:**
- Use object pools for many same-sized objects.
- Prefer contiguous containers such as `std::vector` when appropriate.
- Reuse buffers in hot paths.
- Avoid unnecessary allocation churn.
- Use specialized allocators when profiling proves allocator overhead.

**Interview point:** Fragmentation is especially important in embedded systems, games, real-time systems, and long-running services.

---

## 12. What is an allocator in C++?

**Short answer:** An allocator controls how a container obtains and releases memory.

**Detailed answer:**
Standard containers such as `std::vector`, `std::map`, and `std::string` use allocators to manage memory. The default allocator uses ordinary dynamic allocation, but custom allocators can use memory pools, arenas, shared memory, or tracking mechanisms.

**Example concept:**

```cpp
#include <memory>
#include <vector>

std::vector<int, std::allocator<int>> values;
```

Modern C++ also provides polymorphic memory resources through `<memory_resource>`.

```cpp
#include <memory_resource>
#include <vector>

std::pmr::monotonic_buffer_resource resource;
std::pmr::vector<int> values{&resource};
```

**Interview point:** Custom allocators are powerful but add complexity. Use them when measurement shows allocation behavior is a real bottleneck or when memory must come from a specific region.

**Common mistake:** Reaching for custom allocators before fixing ownership, data structure choice, or allocation frequency.

---

## 13. What is an arena allocator?

**Short answer:** An arena allocator allocates many objects from a memory region and frees them all together.

**Detailed answer:**
Arena allocation is useful when many objects share the same lifetime. Instead of freeing each object individually, the whole arena is reset or destroyed at once. This can be very fast and reduce fragmentation.

**Example use cases:**
- Parsing a request or file
- Compiler AST construction
- Game frame temporary allocations
- Batch processing with clear phase lifetimes

**Tradeoffs:**
Arena allocation is efficient when lifetimes are grouped, but it is a poor fit when individual objects need independent lifetimes.

**Interview point:** Arena allocators are a lifetime design tool, not just a performance trick.

**Common mistake:** Using arenas when object lifetimes are actually independent, causing memory to stay alive longer than needed.

---

## 14. What is use-after-free?

**Short answer:** Use-after-free happens when code accesses memory after it has already been released.

**Detailed answer:**
After memory is freed, the allocator may reuse it for another object, return it to the operating system, or keep it in an internal cache. Accessing the old pointer is undefined behavior and can cause crashes, data corruption, or security vulnerabilities.

**Bad example:**

```cpp
int* p = new int(42);
delete p;

int value = *p; // undefined behavior: use-after-free
```

**Common causes:**
- Raw pointer aliases outliving the owner
- Incorrect callback lifetimes
- Containers invalidating references or iterators
- Async tasks capturing references to destroyed objects

**Prevention:**
- Make ownership explicit.
- Prefer RAII and smart pointers.
- Be careful with callbacks and asynchronous work.
- Use AddressSanitizer during testing.

**Interview point:** Use-after-free is both a correctness bug and a serious security issue.

---

## 15. Why does alignment matter for dynamic memory allocation?

**Short answer:** Objects must be placed at addresses that satisfy their alignment requirements.

**Detailed answer:**
Each type has an alignment requirement. If an object is placed at an incorrectly aligned address, the program may have undefined behavior or poor performance depending on the architecture.

Normal `new` returns memory correctly aligned for the allocated type. For raw storage, placement new, SIMD types, or custom allocators, alignment must be handled explicitly.

**Example:**

```cpp
#include <new>

struct alignas(64) CacheLineData {
    int value;
};

CacheLineData* data = new CacheLineData{};
delete data;
```

For manually managed raw storage, use aligned allocation tools or correctly aligned buffers.

```cpp
alignas(CacheLineData) unsigned char storage[sizeof(CacheLineData)];
auto* object = new (storage) CacheLineData{};
object->~CacheLineData();
```

**Common mistake:** Treating `char` buffers as safe storage for any object without considering alignment and object lifetime rules.

---

## 16. What happens when dynamic allocation fails?

**Short answer:** In C++, ordinary `new` throws `std::bad_alloc`; `malloc` returns `NULL`.

**Detailed answer:**
Allocation failure behavior depends on the API. C code using `malloc`, `calloc`, or `realloc` must check for a null pointer before using the memory. C++ code using ordinary `new` should expect an exception if allocation fails.

**Example:**

```cpp
#include <new>

try {
    int* values = new int[1000000000];
    delete[] values;
} catch (const std::bad_alloc&) {
    // allocation failed
}
```

C++ also supports nothrow `new`, which returns `nullptr` instead of throwing.

```cpp
int* values = new (std::nothrow) int[1000000000];
if (values == nullptr) {
    // allocation failed
}
delete[] values;
```

**Interview point:** Failure strategy depends on the application. A server may reject a request, an embedded system may use fixed pools, and a small utility may terminate.

**Common mistake:** Assuming allocation failure is impossible because the operating system has virtual memory or overcommit.

---

## 17. What is over-aligned allocation?

**Short answer:** Over-aligned allocation is allocation for types whose alignment requirement is stricter than the default allocation alignment.

**Detailed answer:**
Some types need stronger alignment for cache-line separation, SIMD instructions, hardware access, or ABI requirements. Since C++17, `new` handles over-aligned types by using aligned allocation forms when needed.

**Example:**

```cpp
#include <new>

struct alignas(64) CacheAlignedCounter {
    int value;
};

CacheAlignedCounter* counter = new CacheAlignedCounter{};
delete counter;
```

For manual allocation, the allocation and deallocation functions must match.

```cpp
void* memory = ::operator new(sizeof(CacheAlignedCounter), std::align_val_t{64});
::operator delete(memory, std::align_val_t{64});
```

**Interview point:** Alignment is part of an object's validity. Custom allocators and memory pools must return properly aligned storage for the objects they create.

**Common mistake:** Allocating raw bytes and assuming the returned address is suitable for every type.

---

## 18. What is the `shared_ptr` control block?

**Short answer:** The control block stores shared ownership metadata, such as reference counts and the deleter.

**Detailed answer:**
A `std::shared_ptr` usually points to an object and also refers to a control block. The control block tracks how many `shared_ptr`s own the object, how many `weak_ptr`s observe it, and how the object should be destroyed.

**Example:**

```cpp
#include <memory>

std::shared_ptr<int> a = std::make_shared<int>(42);
std::shared_ptr<int> b = a; // same object and same control block
```

`std::make_shared` usually allocates the object and control block together, improving locality and reducing allocation count.

```cpp
auto value = std::make_shared<int>(42);
```

**Interview point:** Creating two independent `shared_ptr`s from the same raw pointer creates two control blocks and can cause double deletion.

```cpp
int* raw = new int(42);
std::shared_ptr<int> first(raw);
// std::shared_ptr<int> second(raw); // wrong: separate control block
```

**Common mistake:** Thinking `shared_ptr` ownership is attached to the raw pointer itself. It is attached to the control block.

---

## 19. What are custom deleters in smart pointers?

**Short answer:** A custom deleter tells a smart pointer how to release a resource that needs cleanup other than plain `delete`.

**Detailed answer:**
Smart pointers can manage resources such as files, sockets, C library handles, and memory from special allocators. The deleter must match how the resource was acquired.

**Example with `unique_ptr` and `FILE*`:**

```cpp
#include <cstdio>
#include <memory>

using FilePtr = std::unique_ptr<std::FILE, int (*)(std::FILE*)>;

FilePtr file(std::fopen("data.txt", "r"), std::fclose);
```

**Example with `free`:**

```cpp
#include <cstdlib>
#include <memory>

std::unique_ptr<int, void (*)(void*)> value(
    static_cast<int*>(std::malloc(sizeof(int))),
    std::free
);
```

**Interview point:** Custom deleters let RAII work with C APIs, but the deleter type affects the smart pointer's type and size.

**Common mistake:** Wrapping a `malloc` allocation in `std::unique_ptr<T>` without a deleter, causing `delete` to be used instead of `free`.

---

## 20. Why are callbacks and asynchronous work dangerous for object lifetime?

**Short answer:** A callback or async task may run after the object it references has already been destroyed.

**Detailed answer:**
Lifetime bugs often happen when asynchronous code captures raw pointers or references to objects that are owned elsewhere. If the owner destroys the object before the callback runs, the callback has a dangling reference.

**Bad example:**

```cpp
class Worker {
public:
    void start() {
        thread_ = std::thread([this] {
            doWork();
        });
    }

private:
    void doWork();
    std::thread thread_;
};
```

This is unsafe unless the class carefully joins the thread before destruction and prevents callbacks from outliving the object.

**Safer ownership patterns:**
- Join or stop worker threads in destructors.
- Use `std::jthread` for automatic joining and stop requests.
- Capture `std::weak_ptr` when a callback should not keep an object alive.
- Capture `std::shared_ptr` only when extending lifetime is intentional.

**Interview point:** Async lifetime design must define who owns the work, how cancellation happens, and what happens during shutdown.

**Common mistake:** Capturing `this` in a callback without proving the object outlives every possible callback execution.

---

## 21. What is the difference between `make_unique` / `make_shared` and direct `new`?

**Short answer:** `make_unique` and `make_shared` create objects and smart pointers in one expression, reducing leaks and ownership mistakes compared with manual `new`.

**Detailed answer:**
Using direct `new` separates allocation from ownership transfer. If an exception or early return happens between allocation and smart pointer construction, ownership can be lost. Factory functions avoid that by immediately returning a smart pointer that owns the object.

`std::make_unique` is usually preferred for exclusive ownership. `std::make_shared` is usually preferred for shared ownership because it can allocate the object and control block together, improving locality and reducing allocation overhead.

**Example:**

```cpp
#include <memory>

struct Session {
    explicit Session(int id) : id(id) {}
    int id;
};

auto uniqueSession = std::make_unique<Session>(1);
auto sharedSession = std::make_shared<Session>(2);
```

**Direct `new` example:**

```cpp
std::unique_ptr<Session> session(new Session(1)); // works, but less preferred
```

**When direct `new` may still appear:**
- When using a custom deleter with a resource that is not created by normal `new`.
- When adopting ownership from legacy APIs.
- When constructing objects with special allocation requirements.

**Interview point:** Prefer factory functions for normal object ownership because they express ownership immediately and avoid raw owning pointers.

**Common mistake:** Writing `std::shared_ptr<T>(new T)` everywhere instead of using `std::make_shared<T>()` for ordinary shared ownership.

---

## 22. What is `realloc`, and why is it dangerous with C++ objects?

**Short answer:** `realloc` resizes a C allocation, possibly moving it, but it does not run C++ constructors, destructors, or move operations.

**Detailed answer:**
`realloc` works on raw memory allocated by `malloc`, `calloc`, or `realloc`. It may expand the allocation in place or allocate a new block, copy bytes, and free the old block.

This byte-copy behavior is unsafe for most C++ objects because C++ object lifetime is not just bytes. Objects may own resources, maintain invariants, or require constructors and destructors. Moving them with `realloc` bypasses all of that.

**C-style use:**

```c
#include <stdlib.h>

int* values = malloc(4 * sizeof(int));
int* bigger = realloc(values, 8 * sizeof(int));
if (bigger != NULL) {
    values = bigger;
}
free(values);
```

**C++ alternative:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3, 4};
values.resize(8);
```

`std::vector` handles allocation, construction, destruction, and moving or copying elements correctly.

**Interview point:** `realloc` is for C-style raw storage, not for resizing arrays of non-trivial C++ objects.

**Common mistake:** Using `realloc` on memory allocated with `new[]`, or using it for objects such as `std::string`, `std::vector`, or classes with destructors.

---

## 23. What does ownership transfer mean in API design?

**Short answer:** Ownership transfer means responsibility for destroying a resource moves from one part of the program to another.

**Detailed answer:**
APIs should make ownership explicit. If a function only observes an object, it should usually take a reference, pointer, or view. If it takes ownership, it should use a type that communicates transfer, such as `std::unique_ptr<T>`.

Ownership clarity prevents leaks, double deletes, and ambiguous lifetime assumptions.

**Examples:**

```cpp
#include <memory>
#include <string_view>

struct Connection {};

void inspect(const Connection& connection);        // borrows, cannot be null
void maybeInspect(const Connection* connection);   // borrows, may be null
void store(std::unique_ptr<Connection> connection); // takes ownership
void logName(std::string_view name);               // borrows character data
```

**Calling ownership-transfer API:**

```cpp
auto connection = std::make_unique<Connection>();
store(std::move(connection)); // ownership transferred
```

After the move, `connection` no longer owns the object.

**Interview point:** Good C++ APIs encode ownership in parameter and return types instead of relying on comments or naming conventions.

**Common mistake:** Passing raw pointers without documenting or encoding whether the callee should delete the object.

---

## 24. What is allocator propagation in standard containers?

**Short answer:** Allocator propagation controls whether a container's allocator is copied, moved, or swapped when the container itself is copied, moved, or swapped.

**Detailed answer:**
Standard containers use allocators to obtain and release memory. For most everyday code, the default allocator is used and allocator propagation is invisible. In advanced code, containers may use custom allocators backed by memory pools, arenas, shared memory, or tracking systems.

When containers are assigned or swapped, the standard needs rules for what happens to their allocators. These rules are controlled by allocator traits such as:

- `propagate_on_container_copy_assignment`
- `propagate_on_container_move_assignment`
- `propagate_on_container_swap`

If allocators do not propagate and two containers use unequal allocators, operations may need element-wise moves or may have restrictions.

**Conceptual example:**

```cpp
#include <memory_resource>
#include <vector>

std::byte buffer[1024];
std::pmr::monotonic_buffer_resource arena(buffer, sizeof(buffer));

std::pmr::vector<int> values(&arena);
values.push_back(1);
values.push_back(2);
```

Here, `values` allocates from `arena`, so copying or moving it requires thinking about which memory resource the destination should use.

**Interview point:** Allocators are not just allocation functions; they are part of a container's type or runtime state and can affect assignment, move, swap, performance, and correctness.

**Common mistake:** Assuming moving a container is always cheap even when allocator rules force element-wise moves.

---

## 25. How can memory corruption be detected and isolated?

**Short answer:** Use sanitizers, debuggers, guard allocators, logging, code review, and small reproductions to find where memory is first corrupted, not just where the crash appears.

**Detailed answer:**
Memory corruption often crashes far away from the real bug. A buffer overflow, use-after-free, double delete, invalid cast, or data race may damage memory first, and the program may fail much later.

Good debugging focuses on finding the first invalid operation.

**Useful tools and techniques:**
- AddressSanitizer for buffer overflows, use-after-free, and double free.
- UndefinedBehaviorSanitizer for many forms of undefined behavior.
- ThreadSanitizer for data races.
- Valgrind or platform-specific heap checkers.
- Debug allocators with guard pages or poisoned memory.
- Watchpoints in a debugger to stop when a memory location changes.
- Reducing the failing case until the corruption becomes reproducible.

**Example sanitizer build:**

```bash
g++ -std=c++20 -fsanitize=address,undefined -g main.cpp -o app
```

**Bug example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3};
int* p = values.data();

values.push_back(4); // may reallocate
*p = 10;             // possible use-after-free
```

**Interview point:** In production debugging, the crash site is often only the symptom. The goal is to identify the earlier write, invalid free, or lifetime violation that corrupted memory.

**Common mistake:** Fixing the line where the crash happens without proving it is the source of corruption.

---

## 26. What is the difference between stack unwinding cleanup and manual cleanup?

**Short answer:** Stack unwinding automatically destroys already-constructed automatic objects, while manual cleanup requires explicit release on every path.

**Detailed answer:**
When an exception is thrown, C++ destroys automatic objects whose lifetimes have begun as the stack unwinds. RAII uses this rule to release resources reliably. Manual cleanup is fragile because every return path, error path, and exception path must remember to release resources.

**Manual cleanup risk:**

```cpp
File* file = openFile();
process(file); // if this throws, closeFile is skipped
closeFile(file);
```

**RAII cleanup:**

```cpp
auto file = makeFileHandle();
process(file); // file handle destructor still runs if this throws
```

**Interview point:** RAII turns cleanup into object lifetime management, which composes naturally with exceptions.

**Common mistake:** Adding `try/catch` blocks only to call cleanup manually instead of using an owning type.

---

## 27. What is a non-owning pointer, and how should it be documented?

**Short answer:** A non-owning pointer observes an object owned elsewhere and must not delete or extend the object's lifetime.

**Detailed answer:**
Raw pointers are often acceptable for non-owning optional access, especially at low-level boundaries. The API should make ownership clear by using references for required non-null borrowing, raw pointers for optional borrowing, and smart pointers only when ownership is involved.

**Example:**

```cpp
void draw(const Widget& widget);      // borrows, required
void drawIfPresent(const Widget* w);  // borrows, optional
void take(std::unique_ptr<Widget> w); // takes ownership
```

If a pointer is stored, the class must state or enforce that the pointee outlives the pointer.

**Interview point:** Raw pointer does not automatically mean bad code. Ambiguous ownership is the real problem.

**Common mistake:** Replacing every raw pointer with `std::shared_ptr` instead of distinguishing borrowing from ownership.

---

## 28. What is the difference between a memory leak and intentional retained memory?

**Short answer:** A leak is unreachable memory that should have been released; retained memory is still reachable and intentionally kept for reuse or lifetime policy.

**Detailed answer:**
Memory growth is not always a leak. Caches, arenas, object pools, and singletons may keep memory until shutdown. A leak occurs when the program loses the ability to release memory it no longer needs.

**Examples of retained memory:**
- A cache with a bounded size.
- An arena freed all at once after a request or phase.
- A process-lifetime lookup table.

**Leak example:**

```cpp
void f() {
    int* p = new int(42);
    // lost pointer, cannot delete
}
```

**Interview point:** Diagnose memory growth by asking whether memory is reachable, bounded, and intentionally retained.

**Common mistake:** Calling all memory that remains at process exit a leak without understanding process-lifetime ownership.

---

## 29. What is peak memory usage, and why can it matter more than leaks?

**Short answer:** Peak memory usage is the maximum memory used at one time, and it can cause failures even if all memory is eventually freed.

**Detailed answer:**
A program may have no leaks but still allocate too much temporary memory. Large copies, buffering whole files, unbounded queues, or inefficient algorithms can cause out-of-memory failures or severe paging.

**Example:**

```cpp
std::vector<std::string> lines = readAllLines(path);
std::vector<std::string> sorted = lines; // doubles memory temporarily
std::sort(sorted.begin(), sorted.end());
```

For large data, streaming or in-place processing may be better.

**Interview point:** Memory correctness includes lifetime and capacity planning, not just matching every allocation with a free.

**Common mistake:** Looking only for leaks when the real problem is a high temporary allocation spike.

---

## 30. What is a memory pool?

**Short answer:** A memory pool preallocates blocks and reuses them for many allocations of similar size or lifetime.

**Detailed answer:**
Memory pools reduce general allocator overhead, improve locality, and make allocation behavior more predictable. They are useful for high-frequency allocations, fixed-size objects, and real-time-ish systems. The tradeoff is more complex lifetime management and possible wasted memory.

**Conceptual example:**

```cpp
class NodePool {
public:
    Node* acquire();
    void release(Node* node);
};
```

A pool must define whether it owns constructed objects, raw storage, or both.

**Interview point:** Pools optimize allocation patterns only when object size, lifetime, and reuse behavior match the design.

**Common mistake:** Adding a custom pool before profiling allocator overhead or defining ownership clearly.

---

## 31. What is the difference between an arena allocator and an object pool?

**Short answer:** An arena usually frees many allocations at once; an object pool usually reuses individual objects or slots.

**Detailed answer:**
An arena is efficient when many objects share the same lifetime, such as all allocations during request parsing or compilation of one unit. Individual deallocation may be unsupported or a no-op. An object pool is better when individual objects are repeatedly acquired and released.

**Arena idea:**

```cpp
std::pmr::monotonic_buffer_resource arena;
std::pmr::vector<int> values(&arena);
```

All arena-backed allocations can be released when the arena is destroyed or reset.

**Interview point:** Choose the allocation strategy based on lifetime shape: bulk lifetime suggests arenas; repeated individual reuse suggests pools.

**Common mistake:** Using an arena for objects that need independent destruction at unpredictable times.

---

## 32. What is memory ownership in callback-based APIs?

**Short answer:** Callback-based APIs must define whether the callback borrows data only during the call or may store it for later use.

**Detailed answer:**
Many lifetime bugs happen when a callback receives a pointer or reference and stores it after the owner has destroyed the object. The API contract should say whether data is valid only for the callback duration, until cancellation, or until another explicit release event.

**Example:**

```cpp
void forEachLine(std::function<void(std::string_view)> callback);
```

If the callback receives a `std::string_view`, it should not store it unless the API explicitly guarantees the underlying storage lifetime.

**Interview point:** Async and callback APIs need stronger lifetime documentation than simple synchronous APIs.

**Common mistake:** Capturing references in callbacks that may run after the referenced object has gone out of scope.

---

## 33. What is the difference between shallow immutability and deep immutability?

**Short answer:** Shallow immutability prevents modifying an object's direct state; deep immutability also prevents modifying objects reachable through its pointers or handles.

**Detailed answer:**
A `const` object cannot modify its non-`mutable` data members through that object. But if a member is a pointer, `const` may only make the pointer itself immutable, not the pointee.

**Example:**

```cpp
struct View {
    int* data;
};

const View view{someIntPointer};
*view.data = 42; // allowed: pointee is not const
```

For deep immutability, use pointer-to-const or immutable ownership structures.

```cpp
struct ReadOnlyView {
    const int* data;
};
```

**Interview point:** `const` is not automatically transitive through pointers or handles.

**Common mistake:** Assuming a `const` object makes everything reachable from it immutable.

---

## 34. What is copy-on-write, and why is it tricky in modern C++?

**Short answer:** Copy-on-write shares data until a write occurs, but it is difficult to make safe and efficient with modern C++ iterator, threading, and reference rules.

**Detailed answer:**
Copy-on-write can reduce copying by sharing a buffer between objects. When a writer modifies the object, it first creates a private copy. This sounds attractive, but it complicates thread safety, iterator validity, reference stability, and performance predictability.

**Conceptual idea:**

```cpp
class CowString {
    std::shared_ptr<Buffer> buffer_;
};
```

Before mutation, the class checks whether the buffer is shared and detaches if necessary.

**Interview point:** Copy-on-write is a design tradeoff, not a free optimization. Modern standard strings do not rely on COW semantics because observable behavior and concurrency make it problematic.

**Common mistake:** Sharing mutable buffers without a correct detach policy and synchronization story.

---

## 35. What is memory poisoning?

**Short answer:** Memory poisoning fills memory with recognizable patterns after allocation or deallocation to expose invalid use.

**Detailed answer:**
Debug allocators and sanitizers may write special byte patterns into freed or uninitialized memory. If code reads poisoned memory, tools can report use-after-free, use-before-initialization, or buffer overrun.

**Example idea:**

```cpp
std::memset(buffer, 0xDD, size); // common debug pattern after free in some systems
```

Production code should not rely on these patterns; they are diagnostic aids.

**Interview point:** Memory poisoning helps fail closer to the real bug instead of letting corrupted state travel through the program.

**Common mistake:** Assuming a use-after-free is safe because the old value still appears to be present in a debug run.

---

## 36. What are guard pages?

**Short answer:** Guard pages are inaccessible memory pages placed around allocations or stacks to catch out-of-bounds access quickly.

**Detailed answer:**
Operating systems can mark pages as inaccessible. Debug allocators may place a guard page before or after an allocation so that buffer underflow or overflow triggers a fault immediately. Stacks also commonly use guard pages to detect overflow.

**Conceptual layout:**

```text
[guard page][allocation][guard page]
```

Guard pages are powerful but expensive because they work at page granularity.

**Interview point:** Guard pages are useful for isolating memory corruption, especially large overflows or stack growth errors.

**Common mistake:** Expecting guard pages to catch every small overflow inside the same accessible page.

---

## 37. What is a dangling reference, and how is it different from a dangling pointer?

**Short answer:** Both refer to an object whose lifetime has ended, but a reference cannot be reseated or checked for null.

**Detailed answer:**
A dangling pointer may sometimes be set to `nullptr` after deletion, although that does not fix other copies. A dangling reference looks like an ordinary object access and has no built-in invalid state, making it especially dangerous.

**Example:**

```cpp
const std::string& badReference() {
    std::string local = "temporary";
    return local; // dangling reference
}
```

**Interview point:** References express non-null borrowing, but they still require the referenced object to outlive the reference.

**Common mistake:** Returning references to local variables or temporaries.

---

## 38. What is lifetime extension through smart pointers, and when is it harmful?

**Short answer:** Capturing or storing a `std::shared_ptr` extends an object's lifetime, which is useful when intentional but harmful when it creates cycles or hides shutdown problems.

**Detailed answer:**
`std::shared_ptr` keeps an object alive as long as at least one owner remains. This is useful for asynchronous work that must keep state alive. However, if callbacks, tasks, or objects keep shared pointers to each other, objects may never be destroyed.

**Example cycle:**

```cpp
struct Node {
    std::shared_ptr<Node> next;
    std::shared_ptr<Node> prev; // cycle risk
};
```

Use `std::weak_ptr` for non-owning back-references or callbacks that should not prolong lifetime.

**Interview point:** Shared ownership is a lifetime policy. Use it deliberately, not just to avoid thinking about ownership.

**Common mistake:** Capturing `shared_from_this()` in every callback and accidentally preventing shutdown.

---

## 39. What is memory pressure, and how should software respond to it?

**Short answer:** Memory pressure occurs when available memory becomes scarce, forcing the system or application to reclaim, reduce, or fail allocations.

**Detailed answer:**
Under memory pressure, systems may page heavily, kill processes, fail allocations, or slow down dramatically. Applications can respond by bounding caches, streaming data, applying backpressure, releasing optional memory, or failing requests gracefully.

**Practical responses:**
- Limit cache sizes.
- Avoid unbounded queues.
- Process large data in chunks.
- Prefer compact data structures when memory is the bottleneck.
- Monitor resident set size and allocation rate.

**Interview point:** Memory management includes operational behavior under pressure, not just correct ownership in normal cases.

**Common mistake:** Treating memory as unlimited until `new` throws or the process is killed.

---

## 40. What is the difference between logical ownership and physical allocation?

**Short answer:** Logical ownership is responsibility for lifetime; physical allocation is where and how memory is obtained.

**Detailed answer:**
An object may be logically owned by one component while physically allocated in an arena, pool, shared memory segment, stack frame, or heap block. Good design separates who is responsible for object lifetime from which allocator provides storage.

**Example:**

```cpp
std::pmr::monotonic_buffer_resource arena;
std::pmr::vector<std::pmr::string> names{&arena};
```

The vector owns its elements logically, while the arena provides physical storage.

**Interview point:** Allocation strategy and ownership model interact, but they are not the same question.

**Common mistake:** Saying "allocated from an arena" as if that alone explains who may use the object and when its lifetime ends.

---

## 41. What is allocator-aware programming in C++?

**Short answer:** Allocator-aware programming means designing containers and types so memory allocation strategy can be supplied from outside.

**Detailed answer:**
Standard containers are allocator-aware: they can use custom allocators to obtain storage. This is useful for arenas, shared memory, memory tracking, low-latency systems, and systems that need bounded allocation behavior. Modern C++ also provides polymorphic allocators through `std::pmr`.

**Example:**

```cpp
#include <memory_resource>
#include <vector>

std::byte buffer[4096];
std::pmr::monotonic_buffer_resource arena(buffer, sizeof(buffer));
std::pmr::vector<int> values{&arena};
```

The vector still owns its elements logically, but the arena supplies the bytes.

**Interview point:** Allocators customize storage acquisition, not object ownership semantics.

**Common mistake:** Assuming a custom allocator automatically makes all object lifetimes safe or deterministic.

---

## 42. What is allocator lifetime, and why does it matter?

**Short answer:** Allocator lifetime matters because containers using an allocator may depend on allocator state remaining alive.

**Detailed answer:**
Stateful allocators and memory resources can own buffers, pools, or tracking state. If a container outlives the allocator resource it uses, later destruction or reallocation can access invalid memory. This is especially important with `std::pmr` containers because they often store only a pointer to a memory resource.

**Example:**

```cpp
std::pmr::vector<int> makeValues() {
    std::byte buffer[1024];
    std::pmr::monotonic_buffer_resource arena(buffer, sizeof(buffer));
    std::pmr::vector<int> values{&arena};
    values.push_back(1);
    return values; // bad: resource and buffer are gone
}
```

**Interview point:** Allocator state is part of a container's lifetime dependencies.

**Common mistake:** Returning a PMR container that uses a stack-allocated memory resource.

---

## 43. What is cross-module memory ownership?

**Short answer:** Cross-module memory ownership is the problem of allocating memory in one module or runtime and freeing it in another.

**Detailed answer:**
Shared libraries, plugins, and applications may use different allocators, compiler runtimes, build modes, or ownership rules. If one side allocates and another side frees with a mismatched allocator, behavior can be undefined or platform-specific.

**Safer API pattern:**

```cpp
extern "C" char* library_create_message();
extern "C" void library_destroy_message(char* message);
```

The module that allocates also provides the corresponding destroy function.

**Interview point:** Ownership APIs must cross binary boundaries explicitly.

**Common mistake:** Returning heap memory from a library and telling callers to use ordinary `delete` or `free` without guaranteeing allocator compatibility.

---

## 44. What is ownership inversion in APIs?

**Short answer:** Ownership inversion happens when an API makes the wrong layer responsible for an object's lifetime.

**Detailed answer:**
An API may expose raw allocation details to callers that do not understand when resources should be released, or it may keep ownership internally while callers assume they can store references indefinitely. Good APIs align ownership with the component that has enough context to manage lifetime correctly.

**Example problem:**

```cpp
const Session* getCurrentSession(); // Who owns it? How long is it valid?
```

A clearer API might return a value, a scoped handle, a reference with documented lifetime, or a `shared_ptr` only if shared ownership is truly intended.

**Interview point:** Ownership should live where lifetime decisions can be made correctly.

**Common mistake:** Using raw pointers in public APIs and relying on naming conventions to communicate complex ownership rules.

---

## 45. What is memory lifetime in asynchronous work queues?

**Short answer:** Data submitted to asynchronous work must remain alive until the worker finishes using it.

**Detailed answer:**
When work runs later on another thread, references and pointers captured from the submitting scope may dangle. Safe designs copy needed data, move ownership into the task, or use shared ownership when the task must extend lifetime intentionally.

**Example bug:**

```cpp
void submitBad(Queue& queue) {
    std::string text = "hello";
    queue.push([&] {
        use(text); // dangling if task runs after submitBad returns
    });
}
```

A safer task captures by value or moves owned data into the closure.

**Interview point:** Async APIs turn local lifetime assumptions into cross-thread lifetime contracts.

**Common mistake:** Capturing references in callbacks without proving the referenced objects outlive the callback.

---

## 46. What is a memory ownership cycle?

**Short answer:** A memory ownership cycle occurs when objects keep each other alive through owning references.

**Detailed answer:**
Reference-counted ownership cannot reclaim cycles because each object keeps another object's count above zero. This commonly happens with `std::shared_ptr` graphs, callbacks, observers, and parent-child relationships with back-pointers.

**Example:**

```cpp
struct Parent;

struct Child {
    std::shared_ptr<Parent> parent;
};

struct Parent {
    std::shared_ptr<Child> child;
};
```

Use `std::weak_ptr` for back-references or observer relationships.

**Interview point:** Shared ownership should form an ownership graph without unintended strong cycles.

**Common mistake:** Using `shared_ptr` for every relationship because it avoids immediate dangling pointers.

---

## 47. What is a leak sanitizer?

**Short answer:** A leak sanitizer is a tool that reports allocated memory that remains unreachable at program exit or at leak-check points.

**Detailed answer:**
LeakSanitizer is commonly used with AddressSanitizer or as a standalone tool. It tracks allocations and determines whether they are still reachable. It helps catch missing frees, lost ownership, and cleanup bugs in tests.

**Typical use:**

```text
-fsanitize=address
ASAN_OPTIONS=detect_leaks=1
```

Leak reports should be interpreted carefully because intentional process-lifetime allocations and caches may be reachable or suppressed by policy.

**Interview point:** Leak tools find symptoms; engineers still need to decide whether retained memory is a bug, cache, or process-lifetime state.

**Common mistake:** Ignoring leak reports in tests because the operating system frees memory at process exit.

---

## 48. What is use-after-scope?

**Short answer:** Use-after-scope happens when code accesses an object after its local scope has ended.

**Detailed answer:**
Use-after-scope is a lifetime bug similar to use-after-free, but the storage is often stack memory. It can occur by returning references to locals, storing pointers to local buffers, or capturing stack variables by reference in callbacks that outlive the scope.

**Example:**

```cpp
std::string_view badView() {
    std::string text = "temporary";
    return text; // dangling view
}
```

Sanitizers can detect many use-after-scope cases when built with appropriate options.

**Interview point:** Stack allocation does not make lifetime bugs harmless; it can make them harder to reproduce.

**Common mistake:** Returning `std::string_view` or raw pointers to local storage.

---

## 49. What is a custom deleter, and when should it be part of the type?

**Short answer:** A custom deleter tells a smart pointer how to release a resource that does not use ordinary `delete`.

**Detailed answer:**
`std::unique_ptr` can store a custom deleter for resources such as `FILE*`, sockets, handles, or memory from special allocators. The deleter is part of the `unique_ptr` type, which can affect APIs and container storage.

**Example:**

```cpp
#include <cstdio>
#include <memory>

using FilePtr = std::unique_ptr<FILE, int (*)(FILE*)>;

FilePtr file(std::fopen("data.txt", "r"), std::fclose);
```

For common resources, a named RAII wrapper may be clearer than exposing a complex pointer type everywhere.

**Interview point:** The cleanup operation is part of ownership semantics.

**Common mistake:** Wrapping a C resource in `unique_ptr<T>` without specifying the correct release function.

---

## 50. How do you design memory-safe ownership in modern C++?

**Short answer:** Prefer clear single ownership, non-owning views for borrowing, RAII for resources, and shared ownership only when it models the real lifetime.

**Detailed answer:**
Modern C++ memory safety starts with ownership design. Use values where possible, `std::unique_ptr` for exclusive heap ownership, references or spans for non-owning access, and `std::shared_ptr` only when multiple owners genuinely share lifetime responsibility. Keep allocation strategies separate from ownership semantics.

**Checklist:**

- Who creates the object?
- Who destroys it?
- Can it be moved?
- Can observers outlive the owner?
- Can work run asynchronously after the caller returns?
- Is there a cycle risk?
- Does the memory cross module or language boundaries?

**Interview point:** Memory safety is a design property, not just a smart-pointer choice.

**Common mistake:** Treating `shared_ptr` as a universal fix instead of defining ownership and borrowing explicitly.
