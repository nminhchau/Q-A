# Concurrency and Multithreading Interview Questions

Concurrency is a high-value C++ interview topic because it combines language knowledge, operating-system concepts, memory models, and debugging skill. Strong answers should focus on correctness first, then performance.

## 1. What is the difference between concurrency and parallelism?

**Short answer:** Concurrency is about dealing with multiple tasks at once. Parallelism is about executing multiple tasks at the same time.

**Detailed answer:**
A concurrent program may interleave tasks on a single CPU core. A parallel program actually runs tasks simultaneously on multiple cores. Concurrency is a design concept; parallelism is an execution property.

**Example:**
A web server handling many client connections is concurrent. If it processes several requests at the exact same time on different CPU cores, it is also parallel.

**Interview point:** Multithreading can enable parallelism, but it can also be used for responsiveness, I/O overlap, and task organization.

**Common mistake:** Using "concurrent" and "parallel" as if they always mean the same thing.

---

## 2. How do you create and join a thread in C++?

**Short answer:** Use `std::thread` to start a thread and `join()` to wait for it to finish.

**Detailed answer:**
A `std::thread` begins execution immediately after construction. If a joinable `std::thread` is destroyed without being joined or detached, the program calls `std::terminate`.

**Example:**

```cpp
#include <iostream>
#include <thread>

void work() {
    std::cout << "working\n";
}

int main() {
    std::thread t(work);
    t.join();
}
```

**Modern C++ note:** C++20 provides `std::jthread`, which joins automatically when destroyed and supports cooperative cancellation.

```cpp
#include <thread>

std::jthread worker([] {
    // work here
});
```

**Common mistake:** Forgetting to join or detach a `std::thread` before destruction.

---

## 3. What is a race condition?

**Short answer:** A race condition happens when program behavior depends on unpredictable timing between threads.

**Detailed answer:**
Race conditions occur when multiple threads access shared state and the final result depends on the order of execution. Not all race conditions are data races, but data races are one of the most dangerous forms.

**Bad example:**

```cpp
#include <thread>

int counter = 0;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        ++counter;
    }
}
```

If two threads run `increment`, updates may be lost because `++counter` is not atomic.

**Interview point:** Race conditions are often intermittent and hard to reproduce, which makes them dangerous in production.

---

## 4. What is a data race?

**Short answer:** A data race occurs when two or more threads access the same memory location concurrently, at least one access is a write, and there is no proper synchronization.

**Detailed answer:**
In C++, a data race causes undefined behavior. This is stronger than simply "wrong result". The compiler and CPU may make optimizations that assume data races do not exist.

**Bad example:**

```cpp
int value = 0;

void writer() {
    value = 42;
}

void reader() {
    if (value == 42) {
        // unsynchronized read
    }
}
```

If `writer` and `reader` run concurrently without synchronization, this is a data race.

**Fix options:**
- Protect access with `std::mutex`.
- Use `std::atomic` for simple shared values.
- Avoid shared mutable state.

**Common mistake:** Thinking `volatile` fixes data races. It does not provide thread synchronization in C++.

---

## 5. How does `std::mutex` help with thread safety?

**Short answer:** A mutex protects shared data by allowing only one thread at a time to enter a critical section.

**Detailed answer:**
A mutex must be locked before accessing shared state and unlocked afterward. In C++, prefer RAII wrappers such as `std::lock_guard` or `std::unique_lock` so the mutex is released automatically.

**Example:**

```cpp
#include <mutex>

std::mutex m;
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(m);
    ++counter;
}
```

`lock_guard` locks the mutex in its constructor and unlocks it in its destructor.

**Interview point:** Keep critical sections small, but not so small that the protected invariant becomes unclear.

**Common mistake:** Manually calling `lock()` and `unlock()` without RAII, which can leave the mutex locked if an exception occurs.

---

## 6. What is a deadlock?

**Short answer:** A deadlock occurs when threads wait forever for resources held by each other.

**Detailed answer:**
A common deadlock happens when two threads acquire the same two mutexes in different orders.

**Bad example:**

```cpp
std::mutex a;
std::mutex b;

void thread1() {
    std::lock_guard<std::mutex> lockA(a);
    std::lock_guard<std::mutex> lockB(b);
}

void thread2() {
    std::lock_guard<std::mutex> lockB(b);
    std::lock_guard<std::mutex> lockA(a);
}
```

If `thread1` holds `a` and `thread2` holds `b`, both may wait forever.

**Better:**

```cpp
std::scoped_lock lock(a, b);
```

`std::scoped_lock` can lock multiple mutexes while avoiding deadlock.

**Common prevention strategies:**
- Always lock mutexes in a consistent order.
- Use `std::scoped_lock` for multiple mutexes.
- Avoid holding locks while calling unknown external code.

---

## 7. What is `std::atomic`?

**Short answer:** `std::atomic` provides operations on shared variables that are indivisible and safe from data races.

**Detailed answer:**
Atomic types are useful for simple shared state such as counters, flags, and reference counts. They avoid data races without using a mutex, but they do not automatically make complex invariants safe.

**Example:**

```cpp
#include <atomic>

std::atomic<int> counter = 0;

void increment() {
    counter.fetch_add(1, std::memory_order_relaxed);
}
```

**Interview point:** Atomics are not always faster or simpler than mutexes. For multi-variable invariants, a mutex is often clearer and safer.

**Common mistake:** Replacing every mutex with atomics without understanding memory ordering and invariants.

---

## 8. What is a condition variable?

**Short answer:** A condition variable lets one thread wait until another thread signals that a condition may be true.

**Detailed answer:**
Condition variables are commonly used for producer-consumer queues. A waiting thread releases the mutex while waiting and reacquires it when awakened.

**Example:**

```cpp
#include <condition_variable>
#include <mutex>
#include <queue>

std::mutex m;
std::condition_variable cv;
std::queue<int> q;

void producer() {
    {
        std::lock_guard<std::mutex> lock(m);
        q.push(42);
    }
    cv.notify_one();
}

void consumer() {
    std::unique_lock<std::mutex> lock(m);
    cv.wait(lock, [] { return !q.empty(); });
    int value = q.front();
    q.pop();
}
```

**Important rule:** Always wait with a predicate because wakeups can be spurious.

**Common mistake:** Using `if` instead of a predicate or loop around condition-variable waits.

---

## 9. What is the C++ memory model?

**Short answer:** The C++ memory model defines how operations in different threads interact and what synchronization is required for well-defined behavior.

**Detailed answer:**
The memory model specifies rules for data races, atomic operations, ordering, visibility, and the "happens-before" relationship. It lets compilers and CPUs optimize aggressively while still giving programmers tools to write correct concurrent code.

**Key idea:**
If one operation happens-before another, the effects of the first operation are visible to the second.

**Example:**
A mutex unlock in one thread synchronizes with a later lock of the same mutex in another thread, creating a happens-before relationship.

```cpp
std::mutex m;
int value = 0;

void writer() {
    std::lock_guard<std::mutex> lock(m);
    value = 42;
}

void reader() {
    std::lock_guard<std::mutex> lock(m);
    // safe to read value
}
```

**Interview point:** You do not need to memorize every memory-order rule for most interviews, but you should understand that unsynchronized shared mutable data is undefined behavior.

---

## 10. What is the difference between `memory_order_relaxed`, acquire, and release?

**Short answer:** `relaxed` guarantees atomicity only. `release` publishes prior writes. `acquire` observes writes published by a release operation.

**Detailed answer:**
`memory_order_relaxed` ensures the atomic operation itself is indivisible but does not create ordering guarantees for other memory operations. It is useful for counters where ordering does not matter.

Release/acquire ordering is used when one thread publishes data and another thread consumes it.

**Example concept:**

```cpp
std::atomic<bool> ready = false;
int data = 0;

void producer() {
    data = 42;
    ready.store(true, std::memory_order_release);
}

void consumer() {
    if (ready.load(std::memory_order_acquire)) {
        // safe to see data == 42
    }
}
```

The release store ensures prior writes become visible to a thread that performs a matching acquire load.

**Interview point:** Prefer mutexes unless there is a clear reason to use low-level atomics. Memory ordering bugs are subtle.

**Common mistake:** Using `relaxed` because it seems faster without proving that ordering is unnecessary.

---

## 11. What problem does `std::jthread` solve?

**Short answer:** `std::jthread` automatically joins on destruction and supports cooperative cancellation.

**Detailed answer:**
`std::thread` requires manual `join()` or `detach()`. If a joinable `std::thread` is destroyed, the program terminates. `std::jthread`, introduced in C++20, joins automatically in its destructor, making thread ownership safer.

**Example:**

```cpp
#include <chrono>
#include <thread>

void worker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}

int main() {
    std::jthread t(worker);
    t.request_stop();
}
```

When `t` goes out of scope, it requests cleanup through normal object lifetime and joins automatically.

**Interview point:** `std::jthread` does not forcibly kill a thread. Cancellation is cooperative, so the worker must check the stop token.

**Common mistake:** Assuming `request_stop()` immediately terminates the running function.

---

## 12. What are `std::future`, `std::promise`, and `std::async` used for?

**Short answer:** They are tools for communicating a result or exception from asynchronous work back to another thread.

**Detailed answer:**
A `std::future<T>` represents a value that will become available later. A `std::promise<T>` lets one thread provide that value. `std::async` can launch a function asynchronously and return a future for its result.

**Example with `std::async`:**

```cpp
#include <future>

int compute() {
    return 42;
}

std::future<int> result = std::async(std::launch::async, compute);
int value = result.get();
```

If `compute` throws, the exception is stored in the future and rethrown by `get()`.

**Interview point:** `future::get()` can be called only once because it consumes the stored result.

**Common mistake:** Assuming `std::async` always creates a new thread without specifying `std::launch::async`. The default policy may defer execution.

---

## 13. What is a thread pool, and why is it useful?

**Short answer:** A thread pool reuses a fixed set of worker threads to execute many tasks.

**Detailed answer:**
Creating and destroying threads for every small task can be expensive. A thread pool keeps worker threads alive and feeds them tasks through a queue. This improves throughput and helps limit the number of active threads.

**Conceptual structure:**

```cpp
class ThreadPool {
public:
    void submit(Task task);

private:
    std::vector<std::jthread> workers_;
    std::queue<Task> tasks_;
    std::mutex mutex_;
    std::condition_variable cv_;
};
```

**Interview point:** A good thread pool must handle synchronization, shutdown, backpressure, exception handling, and task lifetime carefully.

**Common mistake:** Creating one thread per request or per item without considering scheduling overhead and resource limits.

---

## 14. What is `std::shared_mutex`?

**Short answer:** `std::shared_mutex` allows multiple readers or one writer to hold a lock.

**Detailed answer:**
A normal `std::mutex` is exclusive: only one thread can hold it. A `std::shared_mutex` supports shared locking for read-only access and exclusive locking for writes. It is useful when reads are frequent and writes are rare.

**Example:**

```cpp
#include <shared_mutex>
#include <string>
#include <unordered_map>

std::unordered_map<std::string, int> cache;
std::shared_mutex mutex;

int readValue(const std::string& key) {
    std::shared_lock lock(mutex);
    return cache.at(key);
}

void writeValue(const std::string& key, int value) {
    std::unique_lock lock(mutex);
    cache[key] = value;
}
```

**Interview point:** Reader-writer locks help only when read contention dominates and the protected work is large enough to justify the extra complexity.

**Common mistake:** Using `shared_mutex` everywhere even when a normal mutex would be simpler and faster.

---

## 15. What is false sharing?

**Short answer:** False sharing happens when independent variables used by different threads occupy the same cache line and cause unnecessary cache coherence traffic.

**Detailed answer:**
CPUs move memory through caches in cache-line-sized chunks. If two threads update different variables on the same cache line, the cache line may bounce between cores even though the threads are not logically sharing the same variable.

**Example scenario:**
Two worker threads update adjacent counters in an array:

```cpp
struct Counter {
    std::atomic<int> value = 0;
};

Counter counters[2]; // counters may share one cache line
```

Padding or alignment can separate hot per-thread counters.

```cpp
struct alignas(64) PaddedCounter {
    std::atomic<int> value = 0;
};
```

**Interview point:** False sharing is a performance problem, not a data race. The code can be correct but unexpectedly slow.

**Common mistake:** Looking only for algorithmic complexity while ignoring cache coherence costs in highly concurrent code.

---

## 16. What is the difference between blocking, non-blocking, lock-free, and wait-free algorithms?

**Short answer:** Blocking algorithms may wait for locks. Non-blocking algorithms do not block on locks. Lock-free guarantees system-wide progress. Wait-free guarantees per-thread progress.

**Detailed answer:**
A blocking algorithm can stop if a thread holding a lock is delayed. A non-blocking algorithm avoids mutex blocking, often using atomic operations. Lock-free means at least one thread makes progress in a finite number of steps. Wait-free means every thread completes its operation in a bounded number of steps.

**Example idea:**

```cpp
std::atomic<int> counter = 0;
counter.fetch_add(1, std::memory_order_relaxed);
```

This increment is atomic and does not use a mutex, but real lock-free data structures are much harder than simple counters.

**Interview point:** Lock-free does not automatically mean faster or simpler. Correct memory reclamation and ordering are difficult.

**Common mistake:** Calling any code that uses atomics "lock-free" without checking progress guarantees and `is_lock_free()` where relevant.

---

## 17. What is the ABA problem in lock-free programming?

**Short answer:** The ABA problem happens when a value changes from A to B and back to A, making a thread think nothing changed.

**Detailed answer:**
Lock-free algorithms often use compare-and-swap. If a pointer has the same value when checked again, a thread might assume the object is unchanged. But another thread may have removed, freed, reused, and restored the same address in between.

**Conceptual example:**

```cpp
// Thread 1 reads head == A
// Thread 2 pops A, pops B, then pushes A again
// Thread 1 sees head == A and incorrectly assumes the stack is unchanged
```

**Common mitigations:**
- Tagged pointers or version counters
- Hazard pointers
- Epoch-based reclamation
- Avoiding custom lock-free structures unless necessary

**Interview point:** The ABA problem shows that atomic pointer updates are not enough; object lifetime and memory reclamation are part of correctness.

**Common mistake:** Designing a lock-free stack and ignoring when removed nodes can safely be deleted.

---

## 18. What are spurious wakeups and lost wakeups?

**Short answer:** A spurious wakeup occurs when a waiting thread wakes without the condition being true. A lost wakeup occurs when notification happens but the waiter misses the condition due to incorrect synchronization.

**Detailed answer:**
Condition variables must always be used with a mutex-protected predicate. The predicate represents the real condition; the notification only tells waiters to check again.

**Correct pattern:**

```cpp
std::unique_lock<std::mutex> lock(mutex);
cv.wait(lock, [] {
    return ready;
});
```

The predicate form handles spurious wakeups by checking the condition again.

**Lost wakeups are avoided by:**
- Protecting the condition with the same mutex used for waiting.
- Updating the condition before notifying.
- Checking the predicate before sleeping.

**Interview point:** The condition variable is not the condition. The shared state protected by the mutex is the condition.

**Common mistake:** Calling `wait()` without a predicate and assuming every wakeup means work is available.

---

## 19. What is thread-safe initialization of local static variables?

**Short answer:** Since C++11, function-local static variables are initialized exactly once in a thread-safe way.

**Detailed answer:**
If multiple threads call a function containing a local static variable at the same time, C++ ensures the variable is initialized once before any thread uses it. This is useful for lazy initialization.

**Example:**

```cpp
Logger& globalLogger() {
    static Logger logger;
    return logger;
}
```

The first call constructs `logger`; later calls reuse it.

**Interview point:** Thread-safe initialization solves the initialization race, but it does not make all operations on the object thread-safe.

**Common mistake:** Assuming a safely initialized singleton means the singleton's methods are automatically safe for concurrent use.

---

## 20. How do you shut down worker threads safely?

**Short answer:** Define a shutdown protocol: stop accepting work, notify workers, let them exit, and join the threads.

**Detailed answer:**
Thread shutdown is part of concurrency design. A safe design avoids detached threads accessing destroyed objects, workers blocking forever, or tasks being abandoned without a clear policy.

**Typical steps:**
- Set a stop flag or request stop.
- Wake blocked workers with `notify_all`.
- Let workers finish or cancel tasks according to policy.
- Join all worker threads before destroying shared state.

**Example idea:**

```cpp
void close() {
    {
        std::lock_guard<std::mutex> lock(mutex_);
        closed_ = true;
    }
    cv_.notify_all();
}
```

Workers should wait on a predicate such as `closed_ || !tasks_.empty()` and exit when the queue is closed and empty.

**Interview point:** Many concurrency bugs happen during shutdown, not during steady-state processing.

**Common mistake:** Detaching worker threads to avoid joining them, then letting them outlive the objects they use.

---

## 21. What is `std::call_once`, and when should you use it?

**Short answer:** `std::call_once` runs a function exactly once across multiple threads using a shared `std::once_flag`.

**Detailed answer:**
`std::call_once` is useful for one-time initialization that may be triggered by many threads. It avoids manual double-checked locking and handles synchronization correctly. If the called function throws, the initialization is not considered complete, and a later call may try again.

**Example:**

```cpp
#include <mutex>
#include <memory>

std::once_flag initFlag;
std::unique_ptr<Config> config;

void initializeConfig() {
    config = std::make_unique<Config>();
}

Config& getConfig() {
    std::call_once(initFlag, initializeConfig);
    return *config;
}
```

**Interview point:** `std::call_once` is a clear primitive for one-time initialization when function-local static initialization is not a good fit.

**Common mistake:** Implementing one-time initialization with an unsynchronized boolean flag.

---

## 22. What is double-checked locking, and why is it difficult in C++?

**Short answer:** Double-checked locking tries to avoid locking after initialization, but it is easy to implement incorrectly because of memory ordering and publication rules.

**Detailed answer:**
The pattern checks whether an object is initialized before taking a lock, then checks again inside the lock. Without correct atomic operations and memory ordering, another thread may observe a pointer to an object whose construction is not safely published.

Modern C++ often avoids this complexity with function-local statics, `std::call_once`, or simpler locking.

**Risky idea:**

```cpp
if (!instance) {
    std::lock_guard<std::mutex> lock(mutex);
    if (!instance) {
        instance = new Service();
    }
}
```

This sketch is not enough to prove safe publication across threads.

**Better options:**

```cpp
Service& service() {
    static Service instance;
    return instance;
}
```

**Interview point:** Correctness is more important than avoiding a cheap uncontended lock or relying on clever synchronization patterns.

**Common mistake:** Assuming a mutex around construction automatically makes the earlier unsynchronized read safe.

---

## 23. What is a latch or barrier in concurrent programming?

**Short answer:** A latch or barrier lets threads wait until a group reaches a specific synchronization point.

**Detailed answer:**
A `std::latch` is a one-time countdown synchronization primitive. Threads can count down work and wait until the counter reaches zero. A `std::barrier` is reusable across phases: a group of threads repeatedly arrive and wait until all participants reach the barrier.

**Example with `std::latch`:**

```cpp
#include <latch>
#include <thread>
#include <vector>

std::latch ready(3);

std::vector<std::thread> workers;
for (int i = 0; i < 3; ++i) {
    workers.emplace_back([&] {
        prepareWork();
        ready.count_down();
        ready.wait();
        runWork();
    });
}

for (auto& worker : workers) {
    worker.join();
}
```

**Interview point:** Latches and barriers coordinate phases of work; they do not protect shared data from races.

**Common mistake:** Using a barrier and assuming ordinary shared variables no longer need synchronization.

---

## 24. What is thread-local storage?

**Short answer:** Thread-local storage gives each thread its own instance of a variable.

**Detailed answer:**
A variable declared with `thread_local` has separate storage for each thread. It is useful for per-thread caches, random-number generators, scratch buffers, or context that should not be shared.

Thread-local variables avoid data races on that variable because each thread has a separate instance, but they can complicate lifetime, testing, and cleanup.

**Example:**

```cpp
#include <random>

int nextRandom() {
    thread_local std::mt19937 generator(std::random_device{}());
    std::uniform_int_distribution<int> distribution(1, 100);
    return distribution(generator);
}
```

Each thread gets its own generator.

**Interview point:** `thread_local` is useful for per-thread state, but it is not a general replacement for careful ownership and dependency design.

**Common mistake:** Hiding important program state in thread-local variables and making behavior hard to reason about.

---

## 25. What is backpressure in concurrent systems?

**Short answer:** Backpressure is a mechanism that prevents producers from overwhelming consumers when work is generated faster than it can be processed.

**Detailed answer:**
In producer-consumer systems, an unbounded queue can grow until memory is exhausted or latency becomes unacceptable. Backpressure forces producers to slow down, drop work, reject requests, or apply another policy when consumers fall behind.

**Example policies:**
- Use a bounded queue and block producers when full.
- Return an error or retry signal when the system is overloaded.
- Drop low-priority work.
- Scale consumers if the system design allows it.

**Conceptual example:**

```cpp
bool submit(Task task) {
    std::unique_lock<std::mutex> lock(mutex_);
    notFull_.wait(lock, [&] {
        return queue_.size() < maxSize_ || closed_;
    });
    if (closed_) {
        return false;
    }
    queue_.push(std::move(task));
    notEmpty_.notify_one();
    return true;
}
```

**Interview point:** A thread pool or queue design is incomplete unless it defines overload behavior.

**Common mistake:** Using an unbounded queue and calling it scalable because producers never block.

---

## 26. What is cooperative cancellation?

**Short answer:** Cooperative cancellation lets a task stop because it observes a cancellation request, rather than being forcibly killed.

**Detailed answer:**
C++ does not safely support killing arbitrary threads from the outside. A thread may hold locks, own resources, or be modifying shared state. Cooperative cancellation asks work to periodically check a cancellation signal and exit at safe points.

C++20 `std::jthread` integrates with `std::stop_token`, making cancellation requests easier to express.

**Example:**

```cpp
#include <chrono>
#include <thread>

std::jthread worker([](std::stop_token stop) {
    while (!stop.stop_requested()) {
        doOneUnitOfWork();
    }
});

worker.request_stop();
```

The worker decides where it is safe to stop.

**Interview point:** Cancellation is part of the task's protocol, not an external thread-kill operation.

**Common mistake:** Designing long-running loops with no cancellation points.

---

## 27. What is a cancellation-safe operation?

**Short answer:** A cancellation-safe operation leaves shared state consistent if cancellation is requested before, during, or after the operation.

**Detailed answer:**
Cancellation can happen at awkward times. Code should avoid stopping in the middle of an invariant update. A common approach is to check cancellation before starting a unit of work, perform the unit atomically with respect to invariants, then check again before starting the next unit.

**Example:**

```cpp
void process(std::stop_token stop) {
    while (!stop.stop_requested()) {
        Task task;
        if (!queue.pop(task, stop)) {
            break;
        }
        processOneTask(task); // should leave state consistent before returning
    }
}
```

Cancellation-safe code defines which operations are interruptible and which must complete once started.

**Interview point:** Cancellation and exception safety are similar: both require clear invariants and cleanup rules.

**Common mistake:** Checking a stop flag inside a critical update and returning with partially modified state.

---

## 28. What is lock granularity?

**Short answer:** Lock granularity describes how much data or work is protected by a single lock.

**Detailed answer:**
Coarse-grained locks protect large sections of state with fewer locks. They are simpler but can reduce parallelism. Fine-grained locks protect smaller pieces of state. They can improve concurrency but increase complexity, deadlock risk, and overhead.

**Example:**

```cpp
class Cache {
public:
    Value get(Key key) {
        std::lock_guard<std::mutex> lock(mutex_);
        return values_.at(key);
    }

private:
    std::mutex mutex_;
    std::unordered_map<Key, Value> values_;
};
```

This is coarse-grained: one mutex protects the whole map.

**Interview point:** Start with simple locking, then refine only when measurements show contention matters.

**Common mistake:** Adding many locks for theoretical parallelism before proving there is a bottleneck.

---

## 29. What is lock ordering?

**Short answer:** Lock ordering is a rule that all threads acquire multiple locks in the same order to avoid deadlock.

**Detailed answer:**
Deadlock can occur when two threads acquire the same locks in opposite orders. A global ordering rule prevents circular wait. C++ also provides `std::scoped_lock`, which can lock multiple mutexes using a deadlock-avoidance algorithm.

**Example:**

```cpp
void transfer(Account& from, Account& to, int amount) {
    std::scoped_lock lock(from.mutex, to.mutex);
    from.balance -= amount;
    to.balance += amount;
}
```

If manual locking is required, define and document a consistent lock order.

**Interview point:** Deadlock prevention is a design property, not something to patch after the fact.

**Common mistake:** Locking `this` object first in every method and accidentally creating cycles across objects.

---

## 30. What is a reader-writer lock?

**Short answer:** A reader-writer lock allows multiple readers at the same time but requires exclusive access for writers.

**Detailed answer:**
In C++, `std::shared_mutex` supports shared locks for read-only access and unique locks for writes. It can improve throughput when reads are frequent and writes are rare. It can also hurt performance if critical sections are tiny, writes are frequent, or reader/writer fairness becomes a problem.

**Example:**

```cpp
#include <shared_mutex>
#include <unordered_map>

class ConfigStore {
public:
    Value get(Key key) const {
        std::shared_lock lock(mutex_);
        return values_.at(key);
    }

    void set(Key key, Value value) {
        std::unique_lock lock(mutex_);
        values_[key] = std::move(value);
    }

private:
    mutable std::shared_mutex mutex_;
    std::unordered_map<Key, Value> values_;
};
```

**Interview point:** Reader-writer locks are useful only when read parallelism outweighs their extra overhead and complexity.

**Common mistake:** Using `shared_mutex` everywhere because it sounds more concurrent than `mutex`.

---

## 31. What is condition variable notification ordering?

**Short answer:** Notification ordering concerns when shared state is changed relative to `notify_one` or `notify_all`.

**Detailed answer:**
A waiting thread should check a predicate under the mutex. The notifying thread should modify the predicate-protected state while holding the same mutex, then notify. The notification itself may happen before or after unlocking, but the state change must be synchronized through the mutex.

**Example:**

```cpp
{
    std::lock_guard<std::mutex> lock(mutex_);
    ready_ = true;
}
condition_.notify_one();
```

The waiting side should use the predicate form of `wait`.

**Interview point:** The predicate is the memory of the event; the notification is only a wake-up signal.

**Common mistake:** Calling `notify_one` without changing shared state protected by the mutex.

---

## 32. What is a semaphore?

**Short answer:** A semaphore is a counter-based synchronization primitive that controls access to a limited number of permits.

**Detailed answer:**
C++20 provides `std::counting_semaphore` and `std::binary_semaphore`. A semaphore can limit concurrency, signal availability of resources, or coordinate producer-consumer handoff.

**Example:**

```cpp
#include <semaphore>
#include <thread>

std::counting_semaphore<4> slots(4);

void worker() {
    slots.acquire();
    doLimitedWork();
    slots.release();
}
```

At most four workers can pass the semaphore at a time.

**Interview point:** Semaphores are useful for permits, while mutexes are for protecting critical sections.

**Common mistake:** Using a semaphore as a substitute for protecting shared mutable data.

---

## 33. What is compare-and-exchange?

**Short answer:** Compare-and-exchange atomically updates a value only if it currently equals an expected value.

**Detailed answer:**
Atomic compare-and-exchange is a building block for lock-free algorithms. It reads the atomic value, compares it with an expected value, and if equal, replaces it with a desired value. If not equal, it reports failure and usually updates `expected` with the observed value.

**Example:**

```cpp
#include <atomic>

std::atomic<int> value{0};

int expected = 0;
bool changed = value.compare_exchange_strong(expected, 1);
```

If another thread changed `value` first, the operation fails and `expected` receives the current value.

**Interview point:** CAS loops must handle contention and retry carefully.

**Common mistake:** Assuming `compare_exchange_weak` fails only when the value differs; it may fail spuriously and is usually used in a loop.

---

## 34. What is an atomic read-modify-write operation?

**Short answer:** It is an atomic operation that reads a value, computes a new value, and writes it back as one indivisible operation.

**Detailed answer:**
Operations such as `fetch_add`, `fetch_sub`, `exchange`, and `compare_exchange` are read-modify-write operations. They prevent lost updates when multiple threads modify the same atomic variable.

**Example:**

```cpp
#include <atomic>

std::atomic<int> counter{0};

void increment() {
    counter.fetch_add(1, std::memory_order_relaxed);
}
```

Relaxed ordering is enough for a simple statistical counter when no other memory synchronization depends on it.

**Interview point:** Atomicity and memory ordering are related but separate concepts.

**Common mistake:** Using separate atomic load and store operations for an increment, which can still lose updates.

---

## 35. What is the difference between blocking and spinning?

**Short answer:** Blocking lets the OS suspend a thread; spinning repeatedly checks a condition while consuming CPU.

**Detailed answer:**
Blocking is appropriate when waits may be long. Spinning can be useful for very short waits on low-level synchronization paths where sleeping and waking would cost more than waiting briefly. Spinning wastes CPU and can reduce system throughput if used carelessly.

**Example:**

```cpp
while (flag.load(std::memory_order_acquire) == false) {
    // spin briefly
}
```

Production spin loops often include pause instructions, backoff, or a fallback to blocking.

**Interview point:** Spinning is a performance optimization with hardware and workload assumptions.

**Common mistake:** Replacing condition variables with spin loops in ordinary application code.

---

## 36. What is priority inversion?

**Short answer:** Priority inversion happens when a high-priority thread waits for a low-priority thread that holds a needed resource.

**Detailed answer:**
If a low-priority thread holds a mutex and a high-priority thread blocks on it, unrelated medium-priority work may prevent the low-priority thread from running and releasing the mutex. Real-time systems use techniques such as priority inheritance to reduce this risk.

**Example scenario:**

```text
Low-priority thread: holds lock
High-priority thread: waits for lock
Medium-priority thread: runs and delays low-priority thread
```

This can make a high-priority task miss deadlines.

**Interview point:** Scheduling policy and locking design interact; concurrency bugs are not only data races.

**Common mistake:** Ignoring lock hold times in latency-sensitive or real-time systems.

---

## 37. What is a thread-safe queue?

**Short answer:** A thread-safe queue allows producers and consumers to push and pop work without data races while defining blocking, shutdown, and capacity behavior.

**Detailed answer:**
A good thread-safe queue is more than a mutex around `std::queue`. It should specify whether `pop` blocks, how waiting threads wake during shutdown, whether capacity is bounded, and what happens when producers submit after close.

**Example interface:**

```cpp
class TaskQueue {
public:
    bool push(Task task);
    bool pop(Task& task);
    void close();
};
```

`false` might mean the queue is closed. A bounded version may block or reject when full.

**Interview point:** Queue semantics matter as much as the locking implementation.

**Common mistake:** Forgetting to wake waiting consumers when the queue is closed.

---

## 38. What is work stealing?

**Short answer:** Work stealing lets idle worker threads take tasks from other workers' queues to improve load balancing.

**Detailed answer:**
In a work-stealing scheduler, each worker often has a local deque. It pushes and pops its own work cheaply, while idle workers steal from others. This reduces contention compared with one global queue and helps balance uneven recursive or task-based workloads.

**Conceptual model:**

```text
Worker A: [task1, task2, task3]
Worker B: [] -> steals task from Worker A
```

Work stealing is common in task schedulers, parallel algorithms, and fork-join systems.

**Interview point:** Work stealing improves load balance but adds complexity around synchronization, locality, and shutdown.

**Common mistake:** Building a global-queue thread pool and expecting it to scale for all task shapes.

---

## 39. What is memory reclamation in lock-free data structures?

**Short answer:** Memory reclamation is the problem of safely freeing nodes that other threads might still be reading.

**Detailed answer:**
Lock-free structures often remove nodes while other threads may still hold raw pointers obtained earlier. Freeing memory immediately can cause use-after-free. Techniques such as hazard pointers, epoch-based reclamation, and reference counting delay reclamation until no thread can still access the node.

**Example risk:**

```text
Thread A reads pointer to node
Thread B removes and deletes node
Thread A dereferences stale pointer
```

This is separate from making the pointer update atomic.

**Interview point:** Lock-free algorithms are not complete without a safe memory reclamation strategy.

**Common mistake:** Implementing atomic pointer manipulation and forgetting object lifetime.

---

## 40. How should you design concurrent code for maintainability?

**Short answer:** Minimize shared mutable state, make ownership clear, use high-level primitives, and keep synchronization rules simple.

**Detailed answer:**
Maintainable concurrent code starts with design. Prefer message passing, immutable data, task ownership, RAII locks, and well-defined thread lifetimes. Keep critical sections small but understandable. Document lock ordering and shutdown protocols where they are part of the design.

**Guidelines:**

- Prefer `std::jthread` over detached threads.
- Prefer RAII lock wrappers over manual lock/unlock.
- Use mutexes before atomics unless measurements and design justify atomics.
- Avoid sharing data when ownership transfer or message passing works.
- Define cancellation, shutdown, and error propagation early.
- Test with sanitizers and stress tests.

**Interview point:** Senior concurrency design is about reducing possible interleavings, not showing off clever primitives.

**Common mistake:** Reaching for lock-free code before proving that simple synchronized code is too slow.

---

## 41. What is thread ownership?

**Short answer:** Thread ownership defines which object or scope is responsible for starting, stopping, joining, and cleaning up a thread.

**Detailed answer:**
A thread should have a clear owner. The owner decides when the thread starts, how cancellation is requested, how work is drained, how errors are reported, and when `join` happens. Unclear ownership leads to detached threads, shutdown races, use-after-free, and programs that hang during exit.

**Example:**

```cpp
#include <thread>

class Worker {
public:
    Worker() : thread_([this] { run(); }) {}

    ~Worker() {
        requestStop();
        if (thread_.joinable()) {
            thread_.join();
        }
    }

private:
    void run();
    void requestStop();
    std::thread thread_;
};
```

In modern C++, `std::jthread` often simplifies this pattern because it joins automatically and supports cooperative stop requests.

**Interview point:** Thread lifetime is resource ownership. Treat it with the same seriousness as file handles or heap allocations.

**Common mistake:** Detaching a thread to avoid deciding who owns its lifetime.

---

## 42. What is task-based concurrency?

**Short answer:** Task-based concurrency expresses work as tasks submitted to an executor or scheduler instead of manually managing one thread per operation.

**Detailed answer:**
Raw threads describe execution resources. Tasks describe units of work. A task-based design lets a thread pool, event loop, or scheduler decide where and when work runs. This can improve scalability, centralize shutdown, and reduce the overhead of creating many threads.

**Example:**

```cpp
ThreadPool pool;

pool.submit([] {
    processFile();
});

pool.submit([] {
    updateIndex();
});
```

The caller submits work without directly owning worker threads.

**Interview point:** Task-based designs often make concurrency easier to reason about than manually wiring many long-lived threads.

**Common mistake:** Creating a new `std::thread` for every small request or file operation.

---

## 43. What is contention?

**Short answer:** Contention occurs when multiple threads compete for the same resource, such as a mutex, atomic variable, cache line, queue, or allocator.

**Detailed answer:**
Contention reduces scalability because threads spend time waiting, retrying, or invalidating each other's cache lines. A design can be thread-safe but still slow if all work funnels through one shared lock or global counter. Reducing contention may involve sharding, batching, per-thread state, immutable snapshots, or message passing.

**Example:**

```cpp
std::mutex mutex;
std::vector<Event> events;

void record(Event event) {
    std::lock_guard lock(mutex);
    events.push_back(std::move(event));
}
```

This is correct, but it may become a bottleneck if many threads record events frequently.

**Interview point:** Correctness comes first, but scalable concurrency also requires reducing shared hot spots.

**Common mistake:** Measuring only single-thread performance and missing that all worker threads serialize on one lock.

---

## 44. What is lock convoying?

**Short answer:** Lock convoying happens when many threads repeatedly block and wake around the same lock, causing scheduling overhead and poor throughput.

**Detailed answer:**
A convoy can form when a heavily used lock is held long enough that many threads queue behind it. As each thread wakes, acquires the lock, and releases it, the system spends significant time on context switches and cache movement. Convoys are more likely with long critical sections, oversubscription, and blocking operations under locks.

**Example risk:**

```cpp
std::lock_guard lock(mutex);
writeToDisk(record); // slow operation while holding shared lock
```

The slow operation makes every other thread wait behind the lock.

**Interview point:** Lock performance depends on hold time, wait time, scheduler behavior, and workload shape.

**Common mistake:** Protecting too much work with one mutex because it is simpler than designing the shared state boundary.

---

## 45. What is oversubscription in multithreading?

**Short answer:** Oversubscription means running more active threads than the hardware and scheduler can efficiently execute.

**Detailed answer:**
A machine can handle more threads than CPU cores, especially when many block on I/O. But too many CPU-bound threads cause context switching, cache disruption, memory pressure, and unpredictable latency. Oversubscription often happens when multiple libraries each create their own thread pools without coordination.

**Example scenario:**

```text
8-core machine
Web server pool: 32 threads
Database client pool: 16 threads
Parallel algorithm: 8 more worker threads per request
```

The total runnable thread count can become much larger than the core count.

**Interview point:** Thread count is a capacity-planning decision, not just an implementation detail.

**Common mistake:** Assuming more threads always means more parallelism.

---

## 46. What is safe publication?

**Short answer:** Safe publication ensures that when one thread makes an object visible to another thread, the receiving thread also sees the object's fully initialized state.

**Detailed answer:**
Publishing a pointer or reference without synchronization can let another thread observe stale or partially initialized data. Safe publication can be achieved through mutexes, atomic release/acquire operations, thread creation rules, futures, condition variables, or other synchronization mechanisms that establish a happens-before relationship.

**Example:**

```cpp
std::shared_ptr<const Config> config;
std::mutex mutex;

void publish(std::shared_ptr<const Config> next) {
    std::lock_guard lock(mutex);
    config = std::move(next);
}

std::shared_ptr<const Config> getConfig() {
    std::lock_guard lock(mutex);
    return config;
}
```

The mutex protects both visibility and access to the shared pointer.

**Interview point:** Object lifetime and memory visibility must both be correct when sharing data between threads.

**Common mistake:** Assuming that because construction finished in one thread, another unsynchronized thread must see the constructed state.

---

## 47. What is asynchronous object lifetime management?

**Short answer:** It is the discipline of ensuring objects captured or referenced by asynchronous work remain alive until that work no longer needs them.

**Detailed answer:**
Async tasks, callbacks, timers, and continuations may run after the submitting function returns or after the owning object starts destruction. Capturing raw `this` or stack references is dangerous unless the lifetime is guaranteed. Common solutions include ownership transfer, `std::shared_ptr`, `std::weak_ptr`, cancellation, and joining or draining work before destruction.

**Example risk:**

```cpp
class Client {
public:
    void start() {
        queue_.submit([this] {
            sendRequest(); // unsafe if Client is destroyed first
        });
    }

private:
    TaskQueue& queue_;
};
```

A safer design makes ownership and cancellation explicit.

**Interview point:** Many concurrency bugs are lifetime bugs, not locking bugs.

**Common mistake:** Capturing `this` in callbacks without defining who keeps the object alive.

---

## 48. How do you test concurrent code?

**Short answer:** Combine deterministic unit tests, stress tests, sanitizers, fault injection, and design review; no single test style is enough.

**Detailed answer:**
Concurrency bugs often depend on rare interleavings. Unit tests should cover basic contracts such as shutdown, cancellation, and queue closure. Stress tests repeatedly exercise operations under load. ThreadSanitizer can detect many data races. Fault injection can force timing, errors, and cancellation paths that are hard to hit naturally.

**Example test targets:**

- Multiple producers and consumers.
- Shutdown while workers are waiting.
- Cancellation during long-running work.
- Exceptions thrown inside tasks.
- Repeated creation and destruction of concurrent objects.

**Interview point:** Testing concurrent code is partly about making rare interleavings less rare.

**Common mistake:** Running one happy-path test once and assuming the synchronization design is correct.

---

## 49. What is structured concurrency?

**Short answer:** Structured concurrency keeps child tasks tied to a lexical scope or parent operation so their lifetime, cancellation, and errors are controlled together.

**Detailed answer:**
Unstructured concurrency allows tasks to outlive the code that started them, making shutdown and error propagation difficult. Structured concurrency treats concurrent work like scoped resource management: when the scope exits, child work must complete, cancel, or report failure in a defined way.

**Conceptual example:**

```cpp
TaskGroup group;
group.spawn([] { loadUsers(); });
group.spawn([] { loadOrders(); });

group.wait(); // scope does not finish until child tasks are resolved
```

C++ does not yet have one universal standard structured-concurrency abstraction, but the design principle is widely useful.

**Interview point:** Structured concurrency reduces orphaned work and makes cancellation/error propagation easier to reason about.

**Common mistake:** Starting background tasks with no parent responsible for their completion.

---

## 50. How do you review concurrent C++ code?

**Short answer:** Review ownership, shared state, synchronization, lifetime, cancellation, shutdown, and testing strategy before focusing on low-level details.

**Detailed answer:**
A concurrency review should identify every shared mutable object, the synchronization protecting it, and the lifetime of every thread or task. It should check for data races, deadlocks, blocking under locks, missed notifications, unsafe captures, unclear shutdown, and excessive cleverness with atomics or lock-free code.

**Review checklist:**

- Who owns each thread, task, queue, and callback?
- What data is shared, and what protects it?
- Is there a documented lock order for multiple locks?
- Can shutdown wake every waiting thread?
- Can async work access destroyed objects?
- Are atomics using the weakest correct ordering, not a guessed ordering?
- Are tests and sanitizers covering failure and shutdown paths?

**Interview point:** Senior concurrency review focuses on reducing possible unsafe interleavings and making the remaining ones explicit.

**Common mistake:** Reviewing only whether each individual line is thread-safe instead of reviewing the whole concurrency protocol.
