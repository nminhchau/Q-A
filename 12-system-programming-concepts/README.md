# System Programming Concepts Interview Questions

System programming questions test whether a C/C++ candidate understands how programs interact with the operating system, memory, files, processes, and hardware-level representation. These topics are especially important for embedded, backend infrastructure, networking, database, and operating-system roles.

## 1. What is a process?

**Short answer:** A process is an instance of a running program with its own virtual address space and operating-system resources.

**Detailed answer:**
When an executable runs, the operating system creates a process. The process has memory regions such as code, data, heap, and stack. It also owns resources such as file descriptors, environment variables, signal handlers, and threads.

**Example concepts:**
- The same executable can have multiple running processes.
- Each process usually has isolated virtual memory.
- Processes communicate through IPC mechanisms such as pipes, sockets, shared memory, or files.

**Interview point:** A process is heavier than a thread because processes have separate address spaces, while threads in the same process share memory.

---

## 2. What is the difference between a process and a thread?

**Short answer:** A process has its own address space. Threads are execution units inside a process and share the process memory.

**Detailed answer:**
Threads share global variables, heap memory, file descriptors, and other process resources. Each thread has its own stack and execution state. Because threads share memory, communication is cheaper, but synchronization is required to avoid data races.

**Comparison:**

| Concept | Process | Thread |
|---|---|---|
| Address space | Separate | Shared within process |
| Communication | IPC required | Shared memory possible |
| Creation cost | Usually higher | Usually lower |
| Failure isolation | Better | Worse |
| Synchronization | Less shared state | More shared-state risk |

**Common mistake:** Saying threads are always better because they are lighter. Processes can provide stronger isolation and fault containment.

---

## 3. What is virtual memory?

**Short answer:** Virtual memory gives each process the illusion of a large, private address space.

**Detailed answer:**
The operating system and hardware memory management unit map virtual addresses to physical memory. This provides isolation between processes, enables paging, supports memory-mapped files, and allows programs to use addresses independently of physical RAM layout.

**Benefits:**
- Process isolation
- Simplified memory model for programs
- Demand paging
- Memory protection
- Shared libraries and memory mapping

**Interview point:** A pointer value in one process is meaningful only in that process's virtual address space. The same numeric address in another process can refer to different memory or be invalid.

---

## 4. What are the typical memory regions of a process?

**Short answer:** Common regions include text/code, read-only data, initialized data, BSS, heap, stack, and memory-mapped regions.

**Detailed answer:**
A process memory layout is platform-dependent, but many systems organize memory into common regions.

**Common regions:**
- **Text/code:** executable instructions.
- **Read-only data:** constants and string literals.
- **Data:** initialized global and static variables.
- **BSS:** zero-initialized global and static variables.
- **Heap:** dynamic allocations.
- **Stack:** function call frames and local variables.
- **Memory-mapped region:** shared libraries, mapped files, anonymous mappings.

**Example:**

```cpp
int globalInitialized = 10; // data
int globalZero;             // BSS

int main() {
    int local = 1;           // stack
    int* heap = new int(2);  // heap
    delete heap;
}
```

**Common mistake:** Assuming the exact layout is fixed by the C++ standard. It is mostly an implementation and OS detail.

---

## 5. What is a system call?

**Short answer:** A system call is a controlled entry from user mode into the operating-system kernel.

**Detailed answer:**
Programs cannot directly perform privileged operations such as reading files, creating processes, allocating certain OS resources, or accessing hardware. They request these services through system calls.

**Examples:**
- `read`
- `write`
- `open`
- `close`
- `fork`
- `exec`
- `mmap`
- `socket`

**Example in C:**

```c
#include <unistd.h>

const char message[] = "hello\n";
write(1, message, sizeof(message) - 1); // fd 1 is stdout
```

**Interview point:** System calls are usually more expensive than normal function calls because they cross the user-kernel boundary.

---

## 6. What is a file descriptor?

**Short answer:** A file descriptor is a small integer handle used by the operating system to refer to an open file-like resource.

**Detailed answer:**
On Unix-like systems, file descriptors can represent regular files, sockets, pipes, terminals, devices, and more. Standard descriptors are usually:

- `0`: standard input
- `1`: standard output
- `2`: standard error

**Example:**

```c
#include <fcntl.h>
#include <unistd.h>

int fd = open("data.txt", O_RDONLY);
if (fd >= 0) {
    char buffer[128];
    ssize_t n = read(fd, buffer, sizeof(buffer));
    close(fd);
}
```

**Interview point:** In C++, wrap file descriptors in RAII classes to avoid leaks when exceptions or early returns occur.

**Common mistake:** Thinking file descriptors only refer to disk files. They can represent many kernel-managed resources.

---

## 7. What is endianness?

**Short answer:** Endianness describes the byte order used to store multi-byte values in memory.

**Detailed answer:**
In little-endian systems, the least significant byte is stored at the lowest address. In big-endian systems, the most significant byte is stored at the lowest address.

**Example:**
The 32-bit value `0x12345678` is stored as:

```text
Little-endian: 78 56 34 12
Big-endian:    12 34 56 78
```

**Why it matters:**
Endianness matters in binary file formats, network protocols, serialization, embedded systems, and cross-platform data exchange.

**Interview point:** Network byte order is traditionally big-endian. Use conversion functions or serialization libraries instead of assuming host byte order.

---

## 8. What is memory alignment?

**Short answer:** Memory alignment means storing objects at addresses that satisfy the hardware or ABI alignment requirements for their type.

**Detailed answer:**
Many CPU architectures access aligned data more efficiently, and some require alignment for correctness. The compiler may insert padding into structs so each member is properly aligned.

**Example:**

```cpp
struct Example {
    char c;
    int i;
};
```

This struct is often larger than 5 bytes because padding is inserted between `c` and `i`, and possibly at the end.

**Check alignment:**

```cpp
#include <iostream>

std::cout << alignof(int) << '\n';
std::cout << sizeof(Example) << '\n';
```

**Interview point:** Reordering struct members can reduce padding, but ABI compatibility and readability also matter.

**Common mistake:** Assuming `sizeof(struct)` is always the sum of member sizes.

---

## 9. What is `mmap`, and when is it useful?

**Short answer:** `mmap` maps files or anonymous memory into a process's virtual address space.

**Detailed answer:**
Memory mapping allows a program to access file contents as if they were memory. It can be useful for large files, shared memory, lazy loading, and zero-copy-style designs.

**Conceptual example:**

```c
#include <sys/mman.h>

void* memory = mmap(NULL, length, PROT_READ, MAP_PRIVATE, fd, 0);
if (memory != MAP_FAILED) {
    // read bytes from memory
    munmap(memory, length);
}
```

**Benefits:**
- Can avoid explicit read loops.
- Can share memory between processes.
- Can let the OS page data on demand.

**Tradeoffs:**
Error handling can be subtle, page faults may occur during access, and mapped files require careful lifetime management.

---

## 10. What does `volatile` mean in C/C++?

**Short answer:** `volatile` tells the compiler that a value may change in ways it cannot see, so accesses should not be optimized away.

**Detailed answer:**
`volatile` is mainly useful for memory-mapped hardware registers, signal handlers in limited cases, or special low-level situations. It does not make operations atomic and does not provide thread synchronization.

**Example use case:**

```cpp
volatile unsigned int* hardwareRegister = reinterpret_cast<volatile unsigned int*>(0x40000000);
unsigned int value = *hardwareRegister;
```

The compiler should actually perform the load instead of assuming the value is unchanged.

**Important warning:**
For multithreaded C++ code, use `std::atomic`, mutexes, condition variables, or other synchronization primitives. Do not use `volatile` as a threading tool.

**Common mistake:** Thinking `volatile int counter` makes increments thread-safe. It does not.

---

## 11. What is inter-process communication (IPC)?

**Short answer:** IPC is a set of mechanisms that allow separate processes to exchange data or coordinate work.

**Detailed answer:**
Processes usually have separate virtual address spaces, so they cannot directly access each other's ordinary memory. IPC mechanisms provide controlled ways to communicate across process boundaries.

**Common IPC mechanisms:**
- Pipes
- Unix domain sockets
- TCP sockets
- Shared memory
- Message queues
- Signals
- Files

**Example concept:**
A parent process can create a pipe, fork a child process, and use the pipe file descriptors to send bytes between them.

**Interview point:** IPC choices involve tradeoffs between performance, complexity, portability, reliability, and whether communication is local or across a network.

**Common mistake:** Treating IPC like ordinary function calls. IPC involves buffering, failure handling, serialization, blocking behavior, and process lifetime issues.

---

## 12. What is the difference between `fork` and `exec`?

**Short answer:** `fork` creates a new process by duplicating the current one. `exec` replaces the current process image with a new program.

**Detailed answer:**
On Unix-like systems, `fork()` creates a child process. The child initially continues from the same point as the parent, but with a different process ID. `exec()` loads a new executable into the current process, replacing its code, data, heap, and stack.

**Conceptual example:**

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

pid_t pid = fork();
if (pid == 0) {
    execlp("ls", "ls", "-l", (char*)NULL);
    _exit(1);
} else if (pid > 0) {
    waitpid(pid, NULL, 0);
}
```

The child process runs `ls -l`; the parent waits for it.

**Interview point:** After `fork`, both parent and child continue executing. Code must check the return value to know which process it is in.

**Common mistake:** Assuming `exec` creates a new process. It does not; it replaces the current process image.

---

## 13. What are signals?

**Short answer:** Signals are asynchronous notifications sent to a process or thread to report events such as termination requests, timers, or child process changes.

**Detailed answer:**
Signals are used by Unix-like systems for events such as `SIGINT`, `SIGTERM`, `SIGSEGV`, and `SIGCHLD`. A process can use default handling, ignore some signals, or install a signal handler.

**Example concept:**

```c
#include <signal.h>
#include <stdio.h>

void handle(int signal) {
    (void)signal;
}

int main() {
    signal(SIGINT, handle);
}
```

**Important rule:** Signal handlers are restricted. Only async-signal-safe functions should be called from a signal handler.

**Interview point:** Signals are not a general-purpose callback system. They interrupt normal control flow and require careful design.

**Common mistake:** Doing complex work, allocating memory, or locking mutexes inside a signal handler.

---

## 14. What is the difference between blocking and non-blocking I/O?

**Short answer:** Blocking I/O waits until an operation can complete. Non-blocking I/O returns immediately if the operation would block.

**Detailed answer:**
With blocking I/O, a call such as `read()` may sleep until data is available. With non-blocking I/O, the call returns an error such as `EAGAIN` or `EWOULDBLOCK` when no data is currently available.

**Example use case:**
Servers often use non-blocking sockets with readiness APIs such as `select`, `poll`, `epoll`, or `kqueue` to manage many connections without one thread per connection.

**Conceptual example:**

```c
int flags = fcntl(fd, F_GETFL, 0);
fcntl(fd, F_SETFL, flags | O_NONBLOCK);
```

**Interview point:** Non-blocking I/O shifts complexity from the kernel blocking the thread to the application managing readiness, retries, and partial operations.

**Common mistake:** Assuming non-blocking I/O means operations always complete faster. It mainly changes waiting behavior and program structure.

---

## 15. What is shared memory?

**Short answer:** Shared memory allows multiple processes to map the same physical memory into their virtual address spaces.

**Detailed answer:**
Shared memory can be one of the fastest IPC mechanisms because processes can exchange data without copying through the kernel for every message. However, it requires explicit synchronization because multiple processes can access the same memory concurrently.

**Example mechanisms:**
- POSIX shared memory with `shm_open` and `mmap`
- Anonymous shared mappings with `mmap`
- System V shared memory
- Memory-mapped files

**Conceptual example:**

```c
void* memory = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
```

**Interview point:** Shared memory solves data movement costs but not coordination. You still need synchronization such as process-shared mutexes, semaphores, atomics, or careful lock-free protocols.

**Common mistake:** Forgetting that pointers stored inside shared memory may not be valid in another process because each process can map the region at a different virtual address.

---

## 16. What is copy-on-write after `fork`?

**Short answer:** Copy-on-write lets parent and child initially share physical memory pages after `fork` until one process writes to a page.

**Detailed answer:**
A naive `fork` would copy the entire address space immediately, which would be expensive. Modern Unix-like systems usually mark pages as copy-on-write. Parent and child share the same physical pages until either process modifies a page; then the kernel creates a private copy for the writer.

**Example consequence:**
After `fork`, changing a global variable in the child does not change the parent's variable because the page becomes private to the child.

```c
int value = 1;
pid_t pid = fork();
if (pid == 0) {
    value = 2; // child gets its own copy of the modified page
    _exit(0);
}
```

**Interview point:** Copy-on-write makes `fork` efficient, especially when followed quickly by `exec`.

**Common mistake:** Assuming parent and child continue sharing normal heap or global variables after one of them writes.

---

## 17. What is a zombie process?

**Short answer:** A zombie process is a terminated child process whose exit status has not yet been collected by its parent.

**Detailed answer:**
When a child exits, the kernel keeps a small process table entry so the parent can read its exit status using `wait` or `waitpid`. Until the parent reaps it, the child is a zombie.

**Example:**

```c
pid_t pid = fork();
if (pid == 0) {
    _exit(0);
}

// Parent should call waitpid(pid, &status, 0)
```

**Prevention:**
- Call `wait` or `waitpid` for child processes.
- Install appropriate `SIGCHLD` handling.
- Use process supervision patterns carefully.

**Interview point:** Zombies do not consume normal runtime memory, but they do consume process table entries and indicate missing child-process cleanup.

**Common mistake:** Confusing zombie processes with orphan processes. An orphan is still running but its parent has exited.

---

## 18. What is the difference between pipes and sockets?

**Short answer:** Pipes are commonly used for byte streams between related local processes. Sockets support more general communication, including network and bidirectional local communication.

**Detailed answer:**
A pipe provides a unidirectional byte stream. It is often used between a parent and child process. A socket can be local or networked, stream or datagram, and commonly supports bidirectional communication.

**Example pipe:**

```c
int fds[2];
pipe(fds);
// fds[0] is read end, fds[1] is write end
```

**Example socket concept:**

```c
int fd = socket(AF_INET, SOCK_STREAM, 0);
```

**Interview point:** Choose IPC based on communication pattern, locality, reliability, message boundaries, and portability.

**Common mistake:** Assuming all file descriptors behave like regular files. Pipes and sockets have different blocking, buffering, and shutdown behavior.

---

## 19. What is `select`, `poll`, and `epoll` used for?

**Short answer:** They are readiness notification APIs used to monitor multiple file descriptors without blocking on just one.

**Detailed answer:**
Servers often need to handle many sockets. Instead of dedicating one blocking thread to each connection, readiness APIs let a thread ask the kernel which descriptors are ready for reading or writing.

**Comparison:**
- `select`: portable but limited by fd-set size and requires rebuilding sets.
- `poll`: avoids fixed fd-set limits but still scans arrays.
- `epoll`: Linux-specific and scales better for large numbers of descriptors.

**Conceptual flow:**
```text
register fds -> wait for readiness -> read/write until EAGAIN -> wait again
```

**Interview point:** Readiness means an operation is likely possible now; it does not remove the need to handle short reads, short writes, disconnects, or errors.

**Common mistake:** Assuming readiness means a full application message is available.

---

## 20. What is memory-mapped I/O?

**Short answer:** Memory-mapped I/O exposes device registers or file/device memory through ordinary load and store instructions at specific addresses.

**Detailed answer:**
In embedded and systems programming, hardware registers may be mapped into the processor's address space. Software reads or writes those addresses to interact with devices. These accesses often require `volatile` and must follow hardware ordering rules.

**Example concept:**

```cpp
volatile std::uint32_t* control = reinterpret_cast<volatile std::uint32_t*>(0x40000000);
*control = 1;
```

**Important concerns:**
- Alignment and access width may matter.
- Reads and writes may have side effects.
- Compiler and CPU reordering rules may matter.
- Platform documentation controls correct usage.

**Interview point:** Memory-mapped I/O is different from ordinary RAM. Accesses can affect hardware state and must not be optimized away incorrectly.

**Common mistake:** Treating device registers like normal variables or using ordinary cached memory assumptions.

---

## 21. What is the difference between user mode and kernel mode?

**Short answer:** User mode runs application code with limited privileges, while kernel mode runs operating-system code with access to hardware and protected system resources.

**Detailed answer:**
Modern operating systems separate ordinary programs from privileged kernel code. Applications run in user mode and cannot directly access arbitrary physical memory, device registers, or privileged CPU instructions. When they need operating-system services, they make system calls that switch execution into kernel mode.

This boundary improves stability and security. A crash in a user process usually does not crash the whole operating system, while a kernel bug can affect the entire machine.

**Example flow:**

```text
application code -> system call -> kernel code -> return to application
```

**Interview point:** System calls are controlled transitions across the user/kernel boundary.

**Common mistake:** Thinking a library call such as `printf` is always itself a system call. It may buffer data and only later call `write` or another OS primitive.

---

## 22. What is a context switch?

**Short answer:** A context switch saves the execution state of one thread or process and restores another so the CPU can run different work.

**Detailed answer:**
During a context switch, the operating system may save registers, stack pointers, program counters, scheduling state, and memory-management information. Switching between threads in the same process is usually cheaper than switching between processes, but both have overhead.

Context switches happen because of scheduling, blocking I/O, time slices, synchronization, interrupts, or explicit yielding.

**Example scenario:**

```text
Thread A blocks waiting for disk I/O
OS saves Thread A state
OS restores Thread B state
CPU runs Thread B
```

**Interview point:** Excessive context switching can hurt performance due to scheduler overhead, cache disruption, and lost locality.

**Common mistake:** Creating many more runnable threads than useful CPU work and assuming more threads always improve throughput.

---

## 23. What is `errno`, and what are its limitations?

**Short answer:** `errno` is a thread-local error indicator used by many C and POSIX APIs, but it must be checked according to each function's documented failure rules.

**Detailed answer:**
Many system calls and C library functions report failure through a return value and store additional error information in `errno`. `errno` is not automatically meaningful after every call. A function may succeed without clearing it, so checking `errno` alone is wrong.

**Example:**

```cpp
#include <cerrno>
#include <cstdio>
#include <cstring>
#include <stdexcept>

FILE* file = std::fopen("config.txt", "r");
if (!file) {
    throw std::runtime_error(std::strerror(errno));
}
```

The null return from `fopen` indicates failure; `errno` gives more detail.

**Interview point:** Check the primary return value first, then read `errno` immediately before another library call can overwrite it.

**Common mistake:** Checking `errno` after a successful call or long after the failing call occurred.

---

## 24. What are file permissions and umask?

**Short answer:** File permissions control who can read, write, or execute a file, and `umask` removes permission bits from newly created files.

**Detailed answer:**
On POSIX-like systems, files have permission bits for owner, group, and others. Creation calls specify requested permissions, but the process `umask` clears some of those bits. This means the actual permissions may be more restrictive than the mode passed to `open` or `mkdir`.

**Example:**

```cpp
#include <fcntl.h>
#include <unistd.h>

int fd = open("data.txt", O_CREAT | O_WRONLY, 0666);
if (fd != -1) {
    close(fd);
}
```

With a typical `umask` of `0022`, the file becomes `0644`, not `0666`.

**Interview point:** Secure file creation requires thinking about requested mode, process umask, ownership, and whether existing files should be followed or overwritten.

**Common mistake:** Assuming the mode argument to `open` is always the final permission set on disk.

---

## 25. What is signal-safe code?

**Short answer:** Signal-safe code only calls operations that are safe to run from an asynchronous signal handler.

**Detailed answer:**
Signals can interrupt a program at almost any point. If a signal handler calls non-reentrant or locking functions, it may corrupt state or deadlock if the interrupted code was already using that function.

A common pattern is to do minimal work in the handler, set an atomic flag or write to a pipe, and let the main event loop handle the real work.

**Example:**

```cpp
#include <atomic>
#include <csignal>

std::atomic<bool> stopRequested = false;

void handleSignal(int) {
    stopRequested.store(true, std::memory_order_relaxed);
}
```

Real POSIX async-signal-safety rules are stricter than ordinary C++ thread-safety rules, so production signal handlers should be very small.

**Interview point:** Signal handlers are not normal callbacks. They run under severe restrictions.

**Common mistake:** Logging, allocating memory, locking a mutex, or throwing exceptions directly from a signal handler.

---

## 26. What is a process exit status?

**Short answer:** A process exit status is the numeric result a process returns to its parent or shell when it terminates.

**Detailed answer:**
Programs conventionally return `0` for success and nonzero for failure. Shell scripts, service managers, and parent processes use exit status to decide whether a command succeeded, failed, or should be retried.

**Example:**

```cpp
int main() {
    if (!initialize()) {
        return 1;
    }
    run();
    return 0;
}
```

On POSIX systems, more detailed termination information can include whether the process exited normally or was killed by a signal.

**Interview point:** Exit status is part of a program's external API.

**Common mistake:** Printing an error message but still returning success.

---

## 27. What is environment variable inheritance?

**Short answer:** Child processes inherit a copy of the parent's environment variables unless the parent explicitly changes the environment for the child.

**Detailed answer:**
Environment variables are key-value strings available to a process at startup. They commonly configure paths, credentials, locale, feature flags, and runtime behavior. Because children inherit the environment, sensitive or accidental values can affect subprocesses.

**Example:**

```cpp
#include <cstdlib>

const char* home = std::getenv("HOME");
```

Changing an environment variable in one process does not change its already-running parent.

**Interview point:** Environment variables are process-local startup configuration, not a global system database.

**Common mistake:** Passing secrets through the environment without considering child-process inheritance and diagnostic exposure.

---

## 28. What is the current working directory of a process?

**Short answer:** The current working directory is the base directory used to resolve relative paths for a process.

**Detailed answer:**
Each process has its own current working directory. Relative file paths are interpreted relative to it. This can make behavior depend on where the program was launched from unless paths are normalized or configured explicitly.

**Example:**

```cpp
#include <filesystem>

std::filesystem::path path = "config/app.toml";
auto absolute = std::filesystem::absolute(path);
```

Daemons and services often avoid depending on the launch directory.

**Interview point:** Relative paths are convenient but can make programs fragile in production.

**Common mistake:** Assuming a program's working directory is the same as the executable's directory.

---

## 29. What is a file offset?

**Short answer:** A file offset is the current position used by a file descriptor or stream for the next read or write.

**Detailed answer:**
Sequential reads and writes advance the file offset. Duplicated file descriptors may share the same underlying open file description and therefore share an offset. Position-based APIs such as `pread` and `pwrite` avoid changing the shared offset.

**Example:**

```cpp
#include <unistd.h>

char buffer[128];
ssize_t n = pread(fd, buffer, sizeof(buffer), 1024); // read at offset 1024
```

This is useful for concurrent file access where shared offsets would cause races or confusing interleavings.

**Interview point:** The file descriptor number and the underlying open file description are different concepts.

**Common mistake:** Having multiple threads read from the same descriptor and assuming each has an independent offset.

---

## 30. What is file descriptor duplication?

**Short answer:** File descriptor duplication creates another descriptor that refers to the same open file description.

**Detailed answer:**
POSIX APIs such as `dup`, `dup2`, and `dup3` create a new descriptor referring to the same underlying open file description. The duplicated descriptors share file offset and file status flags, but each descriptor has its own descriptor flags such as close-on-exec.

**Example:**

```cpp
#include <unistd.h>

int savedStdout = dup(STDOUT_FILENO);
dup2(logFd, STDOUT_FILENO);
```

This pattern redirects standard output while optionally preserving the original descriptor.

**Interview point:** Descriptor duplication is fundamental to shell redirection and subprocess setup.

**Common mistake:** Forgetting that duplicated descriptors can share file offsets.

---

## 31. What is close-on-exec?

**Short answer:** Close-on-exec is a descriptor flag that automatically closes a file descriptor when a process successfully calls `exec`.

**Detailed answer:**
When a process executes a new program, open file descriptors normally remain open unless marked close-on-exec. Leaking descriptors into child processes can keep files, sockets, or pipes open unexpectedly and may expose sensitive resources.

**Example:**

```cpp
#include <fcntl.h>

int flags = fcntl(fd, F_GETFD);
fcntl(fd, F_SETFD, flags | FD_CLOEXEC);
```

Many modern APIs provide creation flags such as `O_CLOEXEC` to set this atomically.

**Interview point:** Close-on-exec prevents accidental resource inheritance across `exec` boundaries.

**Common mistake:** Opening a sensitive file descriptor and forgetting that subprocesses may inherit it.

---

## 32. What is a pipe buffer?

**Short answer:** A pipe buffer is kernel-managed storage that temporarily holds bytes written to a pipe until another process reads them.

**Detailed answer:**
Pipes are byte streams with finite capacity. If a pipe is full, a blocking write may wait. If a pipe is empty, a blocking read may wait. Pipe capacity affects producer-consumer behavior and subprocess communication.

**Example:**

```text
producer process -> pipe buffer -> consumer process
```

If a parent waits for a child to exit while the child is blocked writing to a full pipe, the system can deadlock unless the parent reads concurrently.

**Interview point:** Pipes provide flow control through blocking behavior and finite buffers.

**Common mistake:** Capturing a child process's output but not reading it until after waiting for the child to exit.

---

## 33. What is socket binding?

**Short answer:** Socket binding associates a socket with a local address and port.

**Detailed answer:**
Servers bind sockets so clients know where to connect. Binding may fail if the address is unavailable, the port is already in use, permissions are insufficient, or the address family does not match.

**Conceptual flow:**

```text
socket() -> bind(local address) -> listen() -> accept()
```

Clients may also bind explicitly when they need a specific local interface or port, though most clients let the OS choose.

**Interview point:** Binding is about local address ownership; connecting is about establishing communication with a peer.

**Common mistake:** Confusing `bind` with `connect` or assuming only servers ever bind sockets.

---

## 34. What is `SO_REUSEADDR`?

**Short answer:** `SO_REUSEADDR` is a socket option that allows rebinding an address in certain cases, commonly after a server restart.

**Detailed answer:**
After a TCP connection closes, ports can remain associated with sockets in states such as `TIME_WAIT`. `SO_REUSEADDR` can allow a server to restart and bind again without waiting for all old connection state to disappear. Exact semantics vary across platforms.

**Conceptual example:**

```text
setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, ...)
bind(server_fd, address)
```

It should be used deliberately and with awareness of platform behavior.

**Interview point:** Socket options often have subtle OS-specific semantics.

**Common mistake:** Treating `SO_REUSEADDR` as a magic fix for all "address already in use" errors.

---

## 35. What is `epoll`, and how is it different from `select`?

**Short answer:** `epoll` is a Linux readiness-notification API designed to scale better than repeatedly scanning file descriptor sets with `select`.

**Detailed answer:**
`select` requires rebuilding and scanning descriptor sets and is limited by descriptor-set constraints. `epoll` lets a program register interest in descriptors and then wait for readiness events. This works well for servers managing many sockets.

**Conceptual flow:**

```text
epoll_create -> epoll_ctl(add fd) -> epoll_wait(events)
```

`epoll` is Linux-specific. Other platforms have alternatives such as `kqueue`, IOCP, or poll-based APIs.

**Interview point:** Readiness APIs tell you an operation may proceed without blocking; they do not perform the I/O for you.

**Common mistake:** Assuming readiness means a full application-level message is available.

---

## 36. What is edge-triggered vs level-triggered readiness?

**Short answer:** Level-triggered readiness repeats while a descriptor remains ready; edge-triggered readiness reports transitions to ready states.

**Detailed answer:**
In level-triggered mode, if data remains unread, the readiness API continues reporting the descriptor. In edge-triggered mode, the API reports when readiness changes, so the program must usually drain the descriptor until it would block.

**Example behavior:**

```text
Level-triggered: "socket is readable" repeated while bytes remain
Edge-triggered: "socket became readable" reported on transition
```

Edge-triggered APIs can reduce repeated notifications but require careful non-blocking I/O loops.

**Interview point:** Edge-triggered readiness is efficient but easier to get wrong.

**Common mistake:** Reading only once from an edge-triggered socket and leaving unread data without receiving another event.

---

## 37. What is memory overcommit?

**Short answer:** Memory overcommit lets an OS promise more virtual memory than it has physical memory available.

**Detailed answer:**
Some operating systems allow allocations or mappings to succeed before physical pages are actually committed. Memory may only be backed when pages are touched. This improves flexibility but means allocation success does not always guarantee future writes will succeed under memory pressure.

**Example:**

```text
allocate large virtual region -> pages committed when written
```

Behavior depends on OS policy and configuration.

**Interview point:** Virtual address reservation and physical memory commitment are different concepts.

**Common mistake:** Assuming a successful allocation always means all corresponding physical memory is already available.

---

## 38. What is page fault handling?

**Short answer:** Page fault handling is the OS process of responding when a program accesses a virtual page that is not currently mapped or present.

**Detailed answer:**
A page fault is not always a crash. It can be normal, such as loading a page from disk, allocating a zero-filled page on first write, or resolving a memory-mapped file page. It becomes a fatal fault when the access violates permissions or refers to an invalid mapping.

**Example cases:**

```text
first write to anonymous memory -> allocate physical page
read memory-mapped file page -> load page from disk
write to read-only page -> protection fault
```

**Interview point:** Page faults are part of normal virtual-memory operation as well as crash behavior.

**Common mistake:** Treating every page fault as a segmentation fault.

---

## 39. What is NUMA?

**Short answer:** NUMA means Non-Uniform Memory Access, where memory access cost depends on which CPU node owns the memory.

**Detailed answer:**
On NUMA systems, CPUs are grouped into nodes with local memory. Accessing local memory is faster than accessing memory attached to another node. High-performance systems may care about thread placement, memory allocation locality, and avoiding cross-node traffic.

**Conceptual model:**

```text
CPU node 0 -> local memory 0: faster
CPU node 0 -> memory on node 1: slower
```

Most ordinary applications do not manually manage NUMA, but it matters in databases, scientific computing, low-latency systems, and large servers.

**Interview point:** Hardware topology can affect system-level performance.

**Common mistake:** Assuming all memory has identical access cost on large multiprocessor machines.

---

## 40. How do you design safe system-level C/C++ code?

**Short answer:** Wrap OS resources with RAII, handle partial failures, define ownership clearly, and treat system calls as unreliable boundaries.

**Detailed answer:**
System-level code interacts with files, processes, memory mappings, sockets, signals, and kernel APIs. These operations can fail for environmental reasons even when the program is correct. Robust code checks return values, preserves error details, closes resources deterministically, and avoids unsafe work in signal handlers or after `fork` in multithreaded processes.

**Guidelines:**

- Use RAII wrappers for file descriptors, mappings, locks, and handles.
- Check system call results immediately.
- Preserve `errno` before calling other functions.
- Use close-on-exec for descriptors that children should not inherit.
- Prefer absolute or configured paths over fragile relative paths.
- Define blocking, timeout, and cancellation behavior.
- Test under resource limits and failure injection when possible.

**Interview point:** Good systems code assumes the environment is adversarial: resources run out, calls are interrupted, and partial progress happens.

**Common mistake:** Writing system-call code that only works on the happy path.

---

## 41. What is the difference between a process ID and a thread ID?

**Short answer:** A process ID identifies a process, while a thread ID identifies an execution thread within a process.

**Detailed answer:**
A process owns resources such as an address space, file descriptor table, environment, and permissions. Threads run inside a process and share most of those resources. Operating systems expose different identifiers for process-level operations and thread-level scheduling, signaling, or debugging.

**Conceptual model:**

```text
Process 1234
  Thread A
  Thread B
  Thread C
```

A process-level kill or wait operation is different from inspecting or signaling a particular thread, and exact APIs vary by OS.

**Interview point:** Processes are resource containers; threads are execution contexts inside those containers.

**Common mistake:** Assuming a thread is just a lightweight process with completely independent resources.

---

## 42. What are process groups and sessions?

**Short answer:** Process groups and sessions organize processes for job control, terminal management, and signal delivery.

**Detailed answer:**
Unix-like systems group related processes into process groups, often representing a shell pipeline or job. A session contains one or more process groups and may have a controlling terminal. Signals such as `SIGINT` from a terminal can be delivered to a foreground process group.

**Conceptual example:**

```text
Session: terminal login
  Foreground process group: shell pipeline
  Background process group: background job
```

Daemons often detach from a controlling terminal by creating a new session.

**Interview point:** Signal delivery is often controlled by process grouping, not only by individual process IDs.

**Common mistake:** Sending a signal to one process and expecting every related child process to receive it automatically.

---

## 43. What is descriptor passing between processes?

**Short answer:** Descriptor passing lets one process send an open file descriptor to another process, commonly over a Unix domain socket.

**Detailed answer:**
A file descriptor is meaningful only inside a process, but the underlying open file description can be shared. Unix-like systems can pass descriptors using ancillary data over Unix domain sockets. This is useful for privilege separation, service managers, socket activation, and sharing already-open resources.

**Conceptual flow:**

```text
Process A opens resource
Process A sends fd over Unix domain socket
Process B receives a new fd referring to the same open resource
```

The receiving process gets its own descriptor number.

**Interview point:** Descriptor passing separates permission to open a resource from permission to use an already-open resource.

**Common mistake:** Thinking descriptor integers can be copied through ordinary IPC and mean the same thing in another process.

---

## 44. What is a pseudo-terminal?

**Short answer:** A pseudo-terminal is a pair of virtual terminal devices that lets a program behave as if it is connected to a real terminal.

**Detailed answer:**
Pseudo-terminals are used by terminal emulators, SSH, shells, test harnesses, and tools that need interactive behavior. One side acts like the terminal device seen by the child process; the other side is controlled by the parent program.

**Conceptual model:**

```text
terminal emulator <-> pty master | pty slave <-> shell
```

Programs may change behavior when connected to a terminal, such as enabling line editing, colors, or interactive prompts.

**Interview point:** Terminal I/O is not the same as ordinary file or pipe I/O.

**Common mistake:** Testing an interactive program only through pipes and missing terminal-specific behavior.

---

## 45. What is zero-copy I/O?

**Short answer:** Zero-copy I/O reduces or avoids copying data between kernel buffers and user-space buffers.

**Detailed answer:**
Traditional I/O often copies data from kernel space to user space and then back to kernel space, such as when forwarding file data to a socket. Zero-copy techniques let the kernel move or map data more directly, reducing CPU usage and memory bandwidth. APIs and exact behavior are platform-specific.

**Conceptual example:**

```text
traditional: disk -> kernel buffer -> user buffer -> kernel socket buffer -> NIC
zero-copy:   disk -> kernel-managed path -> NIC
```

Zero-copy is most useful for high-throughput servers and large data transfers.

**Interview point:** Zero-copy optimizes data movement, but it often adds API constraints and platform-specific complexity.

**Common mistake:** Using zero-copy APIs before measuring whether memory copying is actually the bottleneck.

---

## 46. What is scatter-gather I/O?

**Short answer:** Scatter-gather I/O reads into or writes from multiple buffers in one system call.

**Detailed answer:**
Instead of combining several buffers into one contiguous block before writing, scatter-gather APIs can operate on an array of buffer descriptors. This reduces extra copies and can keep headers, payloads, and trailers separate in memory.

**Conceptual example:**

```text
writev([header buffer, body buffer, checksum buffer])
```

The kernel treats the buffers as one logical I/O operation.

**Interview point:** Scatter-gather I/O is a practical way to reduce copying without redesigning all data structures.

**Common mistake:** Concatenating buffers manually for every send path and adding unnecessary allocations.

---

## 47. What is a monotonic clock, and why is it important?

**Short answer:** A monotonic clock moves forward steadily and is not affected by wall-clock changes.

**Detailed answer:**
Timeouts, elapsed-time measurements, and retry deadlines should use a monotonic clock. Wall-clock time can jump forward or backward due to NTP adjustments, manual changes, daylight-saving transitions, or virtualization effects. A monotonic clock avoids many timeout bugs.

**Example:**

```cpp
auto start = std::chrono::steady_clock::now();
doWork();
auto elapsed = std::chrono::steady_clock::now() - start;
```

`std::chrono::steady_clock` is the usual C++ abstraction for monotonic elapsed time when it is steady on the platform.

**Interview point:** Wall-clock time answers "what time is it?"; monotonic time answers "how much time elapsed?"

**Common mistake:** Using calendar time to implement timeouts.

---

## 48. What are resource limits?

**Short answer:** Resource limits restrict how much of a system resource a process or user can consume.

**Detailed answer:**
Operating systems can limit open files, processes, address space, stack size, CPU time, locked memory, and other resources. Programs that work in development may fail in production if they assume unlimited descriptors, memory mappings, or child processes. Robust system code handles limit failures and exposes useful diagnostics.

**Example failures:**

```text
open() fails because the process reached its file descriptor limit
fork() fails because the user reached a process limit
mmap() fails because address-space or memory limits are exceeded
```

**Interview point:** Resource exhaustion is a normal system-level failure mode.

**Common mistake:** Testing only on machines with generous limits and missing production failure behavior.

---

## 49. What is privilege separation?

**Short answer:** Privilege separation divides a program so only a small component runs with elevated privileges.

**Detailed answer:**
A system program may need special permission to bind a low-numbered port, access a protected file, or perform an administrative operation. Instead of running all code with high privilege, it can isolate privileged operations in a small process or component and communicate with less-privileged workers.

**Conceptual model:**

```text
privileged supervisor: opens protected resource
unprivileged worker: handles ordinary requests
```

This reduces the damage if the larger, more exposed part of the program is compromised.

**Interview point:** Security-sensitive systems code should minimize the amount of code that runs with elevated privilege.

**Common mistake:** Running an entire network service as root because one startup operation needs privilege.

---

## 50. How do you review system-level C/C++ code?

**Short answer:** Review resource ownership, failure handling, blocking behavior, security boundaries, portability assumptions, and cleanup paths.

**Detailed answer:**
System-level code interacts with the operating system, so every call can fail or make partial progress. A strong review checks descriptor lifetimes, close-on-exec behavior, `errno` preservation, signal safety, timeout handling, path handling, process inheritance, and whether APIs are portable or intentionally platform-specific.

**Review checklist:**

- Are file descriptors, mappings, sockets, and handles owned by RAII wrappers?
- Are system call failures checked immediately?
- Is `errno` preserved before logging or cleanup can overwrite it?
- Can blocking calls hang shutdown or cancellation?
- Are descriptors unintentionally inherited across `exec`?
- Are signal handlers limited to async-signal-safe work?
- Are resource limits and partial reads/writes handled?
- Are privilege boundaries explicit and minimal?

**Interview point:** Senior systems review assumes failure is normal and verifies that the code remains safe under resource pressure and hostile inputs.

**Common mistake:** Reviewing only the success path and ignoring how the code behaves when the OS returns short reads, interruptions, or permission errors.
