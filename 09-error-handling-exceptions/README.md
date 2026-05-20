# Error Handling and Exceptions Interview Questions

Error handling in C and C++ is a practical interview topic because it reveals how candidates think about failure, API design, resource cleanup, exception safety, and performance tradeoffs. Good answers should distinguish recoverable errors, programming bugs, and system-level failures.

## 1. How does C usually handle errors?

**Short answer:** C usually handles errors through return codes, sentinel values, `errno`, and output parameters.

**Detailed answer:**
C does not have exceptions. Functions commonly return a value that indicates success or failure. For example, many POSIX APIs return `-1` on failure and set `errno` to describe the error. Some functions return `NULL` for failure.

**Example:**

```c
#include <errno.h>
#include <stdio.h>

FILE* file = fopen("data.txt", "r");
if (file == NULL) {
    perror("fopen failed");
}
```

**Common C patterns:**
- Return `0` for success and nonzero for failure.
- Return `NULL` for pointer-producing functions.
- Use `errno` for additional failure information.
- Use output parameters for results.

**Interview point:** C error handling is explicit but easy to ignore if callers forget to check return values.

---

## 2. What is an exception in C++?

**Short answer:** An exception is a mechanism for reporting errors by transferring control to a matching handler.

**Detailed answer:**
A function can `throw` an exception when it cannot complete normally. The runtime searches for a matching `catch` block while unwinding the stack. During stack unwinding, destructors for local objects are called.

**Example:**

```cpp
#include <stdexcept>

int divide(int a, int b) {
    if (b == 0) {
        throw std::invalid_argument("division by zero");
    }
    return a / b;
}
```

**Handling:**

```cpp
try {
    int result = divide(10, 0);
} catch (const std::invalid_argument& ex) {
    std::cerr << ex.what() << '\n';
}
```

**Interview point:** Exceptions are useful for errors that prevent a function from fulfilling its contract.

---

## 3. When should you use exceptions instead of error codes?

**Short answer:** Use exceptions for exceptional failure paths where normal control flow cannot continue cleanly. Use error codes when failure is expected, frequent, or part of normal logic.

**Detailed answer:**
Exceptions can simplify APIs by separating normal logic from error handling. They also work well with RAII because resources are cleaned up during stack unwinding. Error codes may be better for low-level systems, C APIs, performance-critical paths, embedded systems, or codebases where exceptions are disabled.

**Exception-friendly case:**

```cpp
Config loadConfig(const std::string& path); // throws if invalid or missing
```

**Error-code-friendly case:**

```cpp
bool tryParseInt(std::string_view text, int& output); // failure is expected
```

**Interview point:** The best choice depends on API expectations, project style, performance constraints, and whether callers can recover.

**Common mistake:** Saying exceptions are always better or always bad. Real projects use tradeoffs.

---

## 4. What is stack unwinding?

**Short answer:** Stack unwinding is the process of exiting scopes and destroying local objects while an exception propagates.

**Detailed answer:**
When an exception is thrown, C++ looks for a matching `catch` block. As it exits each scope, destructors of fully constructed local objects are called in reverse order of construction.

**Example:**

```cpp
#include <iostream>
#include <stdexcept>

class Guard {
public:
    explicit Guard(const char* name) : name_(name) {}
    ~Guard() { std::cout << "cleanup " << name_ << '\n'; }

private:
    const char* name_;
};

void work() {
    Guard g("resource");
    throw std::runtime_error("failed");
}
```

`Guard` is destroyed even though `work` exits through an exception.

**Interview point:** Stack unwinding is why RAII is so important in exception-safe C++.

---

## 5. What are the basic exception safety guarantees?

**Short answer:** The common guarantees are no guarantee, basic guarantee, strong guarantee, and no-throw guarantee.

**Detailed answer:**
Exception safety describes what remains true if an operation throws.

**Levels:**
- **No guarantee:** The object or program may be left in an invalid state.
- **Basic guarantee:** No resources leak, and objects remain valid, but values may change.
- **Strong guarantee:** The operation either succeeds completely or has no observable effect.
- **No-throw guarantee:** The operation will not throw.

**Example idea:**
A copy-and-swap assignment operator can provide the strong guarantee: first make a copy, then swap. If copying fails, the original object remains unchanged.

```cpp
class Buffer {
public:
    Buffer& operator=(Buffer other) {
        swap(other);
        return *this;
    }

    void swap(Buffer& other) noexcept;
};
```

**Interview point:** Exception safety is about object invariants and resource cleanup, not just catching exceptions.

---

## 6. Why should destructors generally not throw?

**Short answer:** Throwing from a destructor can terminate the program if another exception is already being handled.

**Detailed answer:**
During stack unwinding, destructors are called automatically. If a destructor throws while another exception is active, C++ calls `std::terminate` because it cannot safely handle two simultaneous exception paths.

**Bad example:**

```cpp
class BadDestructor {
public:
    ~BadDestructor() {
        throw std::runtime_error("bad cleanup"); // dangerous
    }
};
```

**Better approach:**
Destructors should catch and handle cleanup errors internally, log them if appropriate, or provide an explicit `close()`/`commit()` function that can report errors before destruction.

```cpp
class FileWriter {
public:
    void close(); // can report failure
    ~FileWriter() noexcept {
        // best-effort cleanup only
    }
};
```

**Common mistake:** Treating destructors like ordinary functions that can freely report failure through exceptions.

---

## 7. What does `noexcept` mean?

**Short answer:** `noexcept` declares that a function is not expected to throw exceptions.

**Detailed answer:**
If a `noexcept` function throws, `std::terminate` is called. `noexcept` is useful for optimization, documenting intent, and enabling standard containers to move objects safely.

**Example:**

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept;
    Buffer& operator=(Buffer&& other) noexcept;
};
```

**Why it matters for containers:**
`std::vector` may prefer moving elements during reallocation only if the move constructor is `noexcept`; otherwise it may copy to preserve exception safety.

**Conditional noexcept:**

```cpp
template <typename T>
void swapValues(T& a, T& b) noexcept(noexcept(a.swap(b))) {
    a.swap(b);
}
```

**Common mistake:** Adding `noexcept` without being sure. If the function throws, the program terminates.

---

## 8. Should exceptions be caught by value or by reference?

**Short answer:** Catch exceptions by `const` reference.

**Detailed answer:**
Catching by value copies the exception object and can cause slicing if the actual exception is derived from the caught type. Catching by reference preserves the dynamic type and avoids unnecessary copying.

**Good:**

```cpp
try {
    doWork();
} catch (const std::exception& ex) {
    std::cerr << ex.what() << '\n';
}
```

**Bad:**

```cpp
catch (std::exception ex) { // copies and may slice
    std::cerr << ex.what() << '\n';
}
```

**Interview point:** Catch more specific exceptions before more general exceptions.

```cpp
catch (const std::invalid_argument& ex) {
    // specific
} catch (const std::exception& ex) {
    // general
}
```

---

## 9. What is the difference between `throw;` and `throw ex;` inside a catch block?

**Short answer:** `throw;` rethrows the current exception preserving its original type. `throw ex;` throws a new copy and may slice it.

**Detailed answer:**
Use `throw;` when you want to add local handling, logging, or cleanup and then let the original exception continue upward.

**Good:**

```cpp
try {
    doWork();
} catch (const std::exception& ex) {
    log(ex.what());
    throw; // preserves original exception
}
```

**Bad:**

```cpp
catch (const std::exception& ex) {
    throw ex; // creates a new exception object, may slice
}
```

**Interview point:** `throw;` can only be used inside a catch block handling an active exception.

---

## 10. What is `std::expected`, and when is it useful?

**Short answer:** `std::expected` represents either a successful value or an error value without using exceptions.

**Detailed answer:**
`std::expected<T, E>` is useful when failure is expected and should be handled explicitly by the caller. It is especially useful for parsing, validation, APIs that avoid exceptions, and code where error information should be part of the return type.

**Example concept:**

```cpp
#include <expected>
#include <string>

std::expected<int, std::string> parsePositiveInt(std::string_view text) {
    if (text.empty()) {
        return std::unexpected("empty input");
    }

    // parse value here
    return 42;
}
```

**Why it matters:**
Unlike exceptions, `std::expected` makes failure visible in the function signature.

**Interview point:** `std::expected` is a modern alternative to exceptions for APIs where failure is common, local, and recoverable.

**Common mistake:** Using exceptions for ordinary control flow or using error codes so inconsistently that callers forget to check them.

---

## 11. When should you define a custom exception type?

**Short answer:** Define a custom exception type when callers need to distinguish a specific failure category and recover differently.

**Detailed answer:**
Many programs can use standard exceptions such as `std::invalid_argument`, `std::runtime_error`, or `std::system_error`. A custom exception is useful when the error has domain-specific meaning or needs structured information that callers should inspect.

**Example:**

```cpp
#include <stdexcept>
#include <string>

class ConfigError : public std::runtime_error {
public:
    explicit ConfigError(const std::string& message)
        : std::runtime_error(message) {}
};

Config loadConfig(const std::string& path) {
    throw ConfigError("missing required field");
}
```

**Interview point:** Custom exception hierarchies should be small and meaningful. Too many exception types can make APIs harder to use.

**Common mistake:** Throwing raw strings, integers, or unrelated types instead of exception classes derived from `std::exception`.

---

## 12. What does exception neutrality mean?

**Short answer:** Exception-neutral code lets exceptions from lower-level operations propagate without corrupting state or leaking resources.

**Detailed answer:**
A generic function or library often should not catch exceptions unless it can add useful context or recover. Instead, it should maintain invariants, clean up through RAII, and allow the caller to decide how to handle the error.

**Example:**

```cpp
#include <vector>

template <typename T>
void appendAll(std::vector<T>& destination, const std::vector<T>& source) {
    for (const auto& value : source) {
        destination.push_back(value); // may throw
    }
}
```

If `push_back` throws, `std::vector` still maintains its own invariants. The function does not need to catch the exception unless it can provide a stronger guarantee or useful context.

**Interview point:** Catching exceptions too early can hide errors, lose context, or force an API into a recovery decision it cannot make correctly.

**Common mistake:** Writing broad `catch (...)` blocks that swallow failures and let the program continue with invalid assumptions.

---

## 13. Why should errors often be translated at API boundaries?

**Short answer:** API boundaries should expose error handling in the form expected by their callers.

**Detailed answer:**
Different layers may use different error styles. A C++ implementation might use exceptions internally, while a C API boundary must return error codes. A service boundary might translate internal exceptions into response status codes. Translation keeps implementation details from leaking across boundaries.

**Example C boundary:**

```cpp
extern "C" int plugin_init() {
    try {
        initializePlugin();
        return 0;
    } catch (const std::exception&) {
        return -1;
    }
}
```

**Interview point:** Do not let C++ exceptions escape across C ABI boundaries, plugin boundaries, or threads that have no matching handler.

**Common mistake:** Mixing error styles randomly inside the same API layer, making it unclear whether callers should check return codes or catch exceptions.

---

## 14. What is the difference between `std::optional` and `std::expected` for error handling?

**Short answer:** `std::optional<T>` represents a value that may be absent. `std::expected<T, E>` represents either a value or a specific error.

**Detailed answer:**
Use `std::optional` when absence is normal and no detailed error explanation is needed. Use `std::expected` when failure details matter and the caller should inspect the reason.

**Example:**

```cpp
#include <expected>
#include <optional>
#include <string>

std::optional<int> findUserId(std::string_view name);

std::expected<int, std::string> parseUserId(std::string_view text);
```

`findUserId` may simply find nothing. `parseUserId` may need to report whether the input was empty, not numeric, or out of range.

**Interview point:** The return type should communicate whether failure is expected and whether error details are important.

**Common mistake:** Returning `optional` and then logging or storing hidden global error state to explain why the value is missing.

---

## 15. What happens when an exception is not caught?

**Short answer:** If an exception escapes without a matching handler, the program calls `std::terminate`.

**Detailed answer:**
An uncaught exception cannot continue normal execution because no code has agreed to handle the failure. `std::terminate` is also called if a `noexcept` function throws or if a destructor throws during stack unwinding.

**Example:**

```cpp
#include <stdexcept>

void run() {
    throw std::runtime_error("fatal error");
}

int main() {
    run(); // no catch block: program terminates
}
```

**Practical approach:**
Applications often place a top-level catch block around major execution boundaries to log the error and return a controlled failure status.

```cpp
int main() {
    try {
        run();
    } catch (const std::exception& ex) {
        log(ex.what());
        return 1;
    }
}
```

**Interview point:** Top-level handlers are for reporting and controlled shutdown, not for pretending the program can always recover.

**Common mistake:** Catching all exceptions at a low level and continuing as if the failed operation succeeded.

---

## 16. What is `std::error_code`, and when is it useful?

**Short answer:** `std::error_code` is a lightweight object that represents platform or library error values without throwing exceptions.

**Detailed answer:**
`std::error_code` stores an integer error value and an error category that explains how to interpret it. It is useful for APIs where failure is common, exceptions are disabled, or errors come from operating system calls.

**Example:**

```cpp
#include <filesystem>
#include <system_error>

std::error_code ec;
std::filesystem::remove("missing.txt", ec);

if (ec) {
    // inspect ec.message(), ec.value(), or ec.category()
}
```

Many standard library APIs offer both throwing and non-throwing overloads.

**Interview point:** `std::error_code` is useful at system boundaries because it can represent OS-specific failures while still using a common C++ type.

**Common mistake:** Returning only `bool` for failures where callers need to know the reason.

---

## 17. What is `std::system_error`?

**Short answer:** `std::system_error` is an exception type that carries a `std::error_code`.

**Detailed answer:**
`std::system_error` bridges exception-based error handling with error-code-based system failures. It is commonly used for filesystem, threading, networking-like libraries, and operating-system-related failures.

**Example:**

```cpp
#include <stdexcept>
#include <system_error>

void openRequiredFile() {
    std::error_code ec = std::make_error_code(std::errc::no_such_file_or_directory);
    throw std::system_error(ec, "failed to open config");
}
```

The caller can inspect both the message and the underlying code.

```cpp
try {
    openRequiredFile();
} catch (const std::system_error& ex) {
    auto code = ex.code();
}
```

**Interview point:** Use `std::system_error` when an exception should preserve structured low-level error information.

**Common mistake:** Throwing a plain `runtime_error` and losing the original `errno` or platform error code.

---

## 18. What is the difference between recoverable errors and programming bugs?

**Short answer:** Recoverable errors are expected failure conditions a program can handle. Programming bugs are violations of assumptions or contracts that should be fixed in code.

**Detailed answer:**
A missing user-provided file, invalid input, temporary network failure, or permission denial may be recoverable. Out-of-bounds access, violated invariants, null dereference, and data races are programming bugs. They should usually be caught by tests, assertions, sanitizers, or code review rather than treated as ordinary runtime errors.

**Example distinction:**

```cpp
if (!isValidUserInput(text)) {
    return std::unexpected("invalid input"); // recoverable
}

assert(index < values.size()); // programming invariant
```

**Interview point:** Good error handling does not turn bugs into normal control flow. It makes expected failures explicit and catches incorrect assumptions early.

**Common mistake:** Catching every exception or returning error codes for impossible states instead of fixing broken invariants.

---

## 19. How should errors be handled across threads?

**Short answer:** Exceptions do not automatically propagate from one thread to another; they must be caught and communicated explicitly.

**Detailed answer:**
If an exception escapes a `std::thread` function, the program calls `std::terminate`. Thread functions should catch exceptions at the thread boundary and communicate failure through `std::promise`, `std::future`, shared state, a queue, or another agreed mechanism.

**Example with `std::async`:**

```cpp
#include <future>
#include <stdexcept>

std::future<int> result = std::async(std::launch::async, [] {
    throw std::runtime_error("worker failed");
    return 42;
});

try {
    int value = result.get(); // rethrows worker exception here
} catch (const std::exception& ex) {
    // handle worker failure
}
```

**Interview point:** Thread boundaries are error boundaries. Design how failures are reported, how cancellation works, and how shutdown proceeds.

**Common mistake:** Letting exceptions escape from raw `std::thread` entry functions.

---

## 20. How should you design error messages and error context?

**Short answer:** Error messages should include enough context for the caller or operator to understand what failed, without leaking sensitive data.

**Detailed answer:**
A low-level error often needs higher-level context. For example, "permission denied" is more useful when paired with what operation was attempted and which non-sensitive resource identifier was involved.

**Example:**

```cpp
Config loadConfig(std::string_view path) {
    try {
        return parseConfigFile(path);
    } catch (const std::exception& ex) {
        throw ConfigError(std::string("failed to load config: ") + ex.what());
    }
}
```

**Guidelines:**
- Include operation and safe identifiers.
- Preserve structured error codes when available.
- Avoid exposing secrets such as tokens, passwords, or full sensitive payloads.
- Avoid swallowing the original cause.

**Interview point:** Good error context shortens debugging time, but error messages are also part of the security and user-experience surface.

**Common mistake:** Returning vague errors such as "failed" or logging sensitive input in exception messages.

---

## 21. What is exception translation?

**Short answer:** Exception translation converts low-level exceptions into higher-level exceptions that match an API boundary or abstraction layer.

**Detailed answer:**
Low-level code often fails with errors that are too implementation-specific for callers. For example, a database library may throw a connection exception, but the service layer may want to report `UserRepositoryError`. Translation preserves useful cause information while exposing an error type that makes sense at the boundary.

**Example:**

```cpp
#include <stdexcept>
#include <string>

class StorageError : public std::runtime_error {
public:
    using std::runtime_error::runtime_error;
};

User loadUser(int id) {
    try {
        return databaseLoadUser(id);
    } catch (const DatabaseError& ex) {
        throw StorageError(std::string("failed to load user: ") + ex.what());
    }
}
```

**Interview point:** Translate errors at architectural boundaries, not at every small helper function.

**Common mistake:** Catching and wrapping exceptions so often that the original cause becomes noisy or hard to identify.

---

## 22. What is stack trace information, and why is it useful in error handling?

**Short answer:** Stack trace information shows the call path that led to an error, helping developers locate where a failure originated.

**Detailed answer:**
An error message explains what failed, while a stack trace helps explain where it failed. In C++, stack traces are not automatically attached to every exception in a portable way, though modern tooling, debuggers, crash reporters, logs, and C++23 `std::stacktrace` can help.

Stack traces are especially useful for unexpected failures, production crashes, and exceptions that cross several abstraction layers.

**Conceptual example:**

```cpp
#include <stacktrace>
#include <stdexcept>

void fail() {
    auto trace = std::stacktrace::current();
    throw std::runtime_error("operation failed");
}
```

A real system may log the trace at the catch boundary instead of storing it in every exception.

**Interview point:** Stack traces are diagnostic context. They should complement structured error handling, not replace clear error types and messages.

**Common mistake:** Logging only `ex.what()` for unexpected failures and losing the call path needed to debug the issue.

---

## 23. What is the difference between fail-fast and graceful recovery?

**Short answer:** Fail-fast stops quickly when an invariant is broken, while graceful recovery handles expected failures and keeps the program operating safely.

**Detailed answer:**
Fail-fast is appropriate for programming bugs, corrupted state, impossible conditions, or violated internal contracts. Continuing after such failures can make damage worse. Graceful recovery is appropriate for expected runtime conditions such as invalid user input, unavailable files, timeouts, or temporary network failures.

**Example:**

```cpp
#include <cassert>
#include <expected>

int getElement(const std::vector<int>& values, std::size_t index) {
    assert(index < values.size()); // internal contract
    return values[index];
}

std::expected<int, std::string> parsePort(std::string_view text) {
    // invalid user input is recoverable
}
```

**Interview point:** A strong error-handling design separates bugs from expected failures and chooses the right response for each.

**Common mistake:** Trying to recover from corrupted internal state as if it were ordinary user input.

---

## 24. How should C++ code handle errors from C APIs?

**Short answer:** Check the C API's failure convention immediately, preserve the error details, and translate them into a safer C++ abstraction when appropriate.

**Detailed answer:**
C APIs commonly report failure using null pointers, negative return values, sentinel values, `errno`, or output parameters. C++ wrappers should check these results close to the call site and convert them into RAII objects, exceptions, `std::expected`, or `std::error_code` depending on the API style.

**Example:**

```cpp
#include <cerrno>
#include <cstdio>
#include <cstring>
#include <stdexcept>

FILE* openFile(const char* path) {
    FILE* file = std::fopen(path, "r");
    if (!file) {
        throw std::runtime_error(std::string("fopen failed: ") + std::strerror(errno));
    }
    return file;
}
```

A production wrapper would usually return a RAII type so the file is closed automatically.

**Interview point:** C API errors must be checked before later calls overwrite error state such as `errno`.

**Common mistake:** Calling several C functions and only then checking `errno`, after the original failure information may have been lost.

---

## 25. What is the difference between error handling policy and error handling mechanism?

**Short answer:** A mechanism is how errors are represented; a policy is what the program decides to do when an error occurs.

**Detailed answer:**
Mechanisms include exceptions, error codes, `std::expected`, `std::optional`, assertions, logging, and process termination. Policy decisions include retrying, showing a message, falling back, rolling back, ignoring, escalating, or shutting down.

Separating mechanism from policy keeps low-level code reusable. A parser should usually report that parsing failed; the application layer should decide whether to ask the user again, reject a request, or use a default.

**Example:**

```cpp
std::expected<Config, ParseError> parseConfig(std::string_view text); // mechanism

void startApplication() {
    auto config = parseConfig(readConfigText());
    if (!config) {
        log(config.error());
        useSafeDefaults(); // policy
    }
}
```

**Interview point:** Libraries should usually report errors clearly and avoid deciding application-level policy unless that is their responsibility.

**Common mistake:** Logging, retrying, and terminating deep inside utility code, leaving callers no control over recovery.

---

## 26. What is the difference between an assertion and error handling?

**Short answer:** Assertions check programmer assumptions; error handling deals with expected runtime failures.

**Detailed answer:**
An assertion documents and checks an internal contract, such as "this pointer must not be null here" or "this index was validated earlier." If the assertion fails, the program has a bug. Error handling is for conditions that can occur in correct programs, such as invalid input, missing files, timeouts, or permission failures.

**Example:**

```cpp
#include <cassert>
#include <expected>
#include <string>
#include <string_view>

int elementAt(const std::vector<int>& values, std::size_t index) {
    assert(index < values.size());
    return values[index];
}

std::expected<int, std::string> parseAge(std::string_view text) {
    if (text.empty()) {
        return std::unexpected("age is empty");
    }
    // parse text
    return 42;
}
```

Do not use assertions to validate user input, because assertions may be disabled in release builds.

**Interview point:** Assertions are for bugs; error handling is for recoverable or reportable runtime conditions.

**Common mistake:** Replacing real validation with `assert` at a system boundary.

---

## 27. What is a precondition, and how does it relate to errors?

**Short answer:** A precondition is something a caller must satisfy before calling a function; violating it is usually a programming error, not a recoverable runtime error.

**Detailed answer:**
Preconditions define the contract of a function. For example, a function may require a non-null pointer, a sorted range, or a valid index. If the caller violates that contract, the callee may assert, document undefined behavior, or use defensive checks depending on the API boundary.

**Example:**

```cpp
#include <cassert>
#include <span>

int medianOfSorted(std::span<const int> values) {
    assert(!values.empty());
    assert(std::is_sorted(values.begin(), values.end()));
    return values[values.size() / 2];
}
```

Public APIs that receive untrusted input often validate and return errors. Internal helpers can rely more on documented preconditions.

**Interview point:** Treating every precondition violation as a recoverable error can make internal code noisy and hide bugs.

**Common mistake:** Failing silently when a precondition is violated, allowing corrupted assumptions to spread.

---

## 28. What is a postcondition?

**Short answer:** A postcondition is something a function promises will be true after it completes successfully.

**Detailed answer:**
Postconditions describe the result of a successful operation. They are useful for reasoning about correctness and exception safety. If a function reports success but leaves its promised state untrue, that is a bug.

**Example:**

```cpp
#include <cassert>
#include <vector>

void appendPositive(std::vector<int>& values, int value) {
    assert(value > 0);
    values.push_back(value);
    assert(values.back() == value);
}
```

In production code, postconditions are often documented, tested, or checked in debug builds.

**Interview point:** Clear postconditions make error handling easier because callers know what success means.

**Common mistake:** Returning success after only partially completing an operation without documenting the resulting state.

---

## 29. What is transactional error handling?

**Short answer:** Transactional error handling either completes an operation fully or rolls back so the system remains in a consistent state.

**Detailed answer:**
A transactional operation avoids leaving half-applied changes after failure. In C++, this often means preparing new state first, then committing with a small no-fail operation. This idea appears in databases, file updates, configuration reloads, and in-memory state changes.

**Example:**

```cpp
#include <vector>

void replaceValues(std::vector<int>& target, std::vector<int> replacement) {
    target.swap(replacement);
}
```

The replacement vector is built before the call. The swap commits the change. If building the replacement failed earlier, `target` would remain unchanged.

**Interview point:** Transactional design is a practical way to achieve the strong exception guarantee.

**Common mistake:** Mutating the original state step by step before all failure-prone work has succeeded.

---

## 30. How should retry logic be designed?

**Short answer:** Retry only transient failures, limit attempts, use backoff, and keep retry policy near the layer that understands the operation.

**Detailed answer:**
Retries are useful for temporary failures such as network timeouts, rate limits, or transient service unavailability. They are harmful for permanent failures such as invalid input, authentication errors, or corrupted data. Retry loops should have limits, delays, cancellation support, and observability.

**Example:**

```cpp
for (int attempt = 0; attempt != maxAttempts; ++attempt) {
    auto result = sendRequest();
    if (result) {
        return result;
    }
    if (!isTransient(result.error())) {
        return result;
    }
    waitBeforeRetry(attempt);
}
```

Retrying deep inside low-level utilities can surprise callers and make latency unpredictable.

**Interview point:** Retrying is an error-handling policy, not just a mechanism.

**Common mistake:** Retrying every failure and accidentally amplifying overload or hiding permanent errors.

---

## 31. What is error aggregation?

**Short answer:** Error aggregation collects multiple failures and reports them together instead of stopping at the first one.

**Detailed answer:**
Aggregation is useful when validating a configuration file, form input, batch job, or static analysis result. Reporting all problems helps the caller fix several issues at once. It is less suitable when later checks depend on earlier successful state.

**Example:**

```cpp
#include <string>
#include <vector>

struct ValidationResult {
    std::vector<std::string> errors;

    bool ok() const {
        return errors.empty();
    }
};
```

A validator can append errors for missing fields, invalid ranges, and inconsistent options before returning.

**Interview point:** Fail-fast and aggregate-errors are both valid, depending on whether continuing produces meaningful diagnostics.

**Common mistake:** Aggregating after the input is too invalid to inspect safely.

---

## 32. What is an error domain?

**Short answer:** An error domain groups related error values and defines how to interpret them.

**Detailed answer:**
In C++, `std::error_code` uses an error category to identify the domain of an error. The same integer value can mean different things in different domains, so the category is part of the meaning.

**Example:**

```cpp
#include <system_error>

std::error_code ec = std::make_error_code(std::errc::permission_denied);

if (ec == std::errc::permission_denied) {
    // handle permission failure
}
```

Libraries can define their own categories for domain-specific errors.

**Interview point:** Error domains prevent unrelated error values from being confused just because they share a number.

**Common mistake:** Returning plain integers without documenting which domain gives them meaning.

---

## 33. What is the difference between local and boundary-level error handling?

**Short answer:** Local handling fixes or cleans up near the failure; boundary-level handling translates, logs, reports, or decides policy at an architectural edge.

**Detailed answer:**
Local code should handle errors only when it can add value, such as releasing resources, trying an alternate local path, or preserving invariants. Boundaries such as API endpoints, thread entry points, plugin interfaces, and command-line entry points are good places to translate errors and produce logs or user-facing messages.

**Example:**

```cpp
int main() {
    try {
        runApplication();
        return 0;
    } catch (const std::exception& ex) {
        logFatal(ex.what());
        return 1;
    }
}
```

Catching too low often produces noisy code and duplicated handling.

**Interview point:** Catch exceptions where you can recover, translate, or report with useful context.

**Common mistake:** Catching exceptions only to log and rethrow at many layers.

---

## 34. Why should exceptions not cross C ABI boundaries?

**Short answer:** C code does not understand C++ exception unwinding, so throwing through a C ABI boundary can cause undefined behavior or termination.

**Detailed answer:**
C callbacks, plugin APIs, operating-system entry points, and foreign-function interfaces usually cannot propagate C++ exceptions safely. C++ code should catch exceptions before crossing such boundaries and convert them to error codes or other agreed representations.

**Example:**

```cpp
extern "C" int plugin_entry() noexcept {
    try {
        runPlugin();
        return 0;
    } catch (...) {
        return -1;
    }
}
```

The `noexcept` makes the boundary expectation explicit.

**Interview point:** Exception safety includes respecting ABI and language boundaries.

**Common mistake:** Allowing exceptions to escape callbacks registered with C libraries.

---

## 35. What happens if an exception escapes a thread function?

**Short answer:** If an exception escapes the initial function of a `std::thread`, the program calls `std::terminate`.

**Detailed answer:**
Exceptions do not automatically propagate from one thread to another. A thread function should catch exceptions and communicate failure through `std::promise`, `std::future`, shared state, a queue, logging, or another explicit mechanism.

**Example:**

```cpp
#include <exception>
#include <future>
#include <thread>

std::promise<void> promise;

std::thread worker([p = std::move(promise)]() mutable {
    try {
        doWork();
        p.set_value();
    } catch (...) {
        p.set_exception(std::current_exception());
    }
});
```

The receiving thread can call `future.get()` to observe either success or the captured exception.

**Interview point:** Thread boundaries are error boundaries.

**Common mistake:** Assuming a `try`/`catch` around `join()` catches exceptions thrown inside the worker thread.

---

## 36. What is exception-safe logging?

**Short answer:** Exception-safe logging avoids throwing new exceptions while reporting an existing failure.

**Detailed answer:**
Logging often happens during error handling, destruction, or shutdown. If logging throws during these paths, it can mask the original error or trigger termination. Logging code used in catch blocks should be best-effort and should avoid throwing across the error-handling boundary.

**Example:**

```cpp
try {
    runTask();
} catch (const std::exception& ex) {
    try {
        logError(ex.what());
    } catch (...) {
        // suppress logging failure at this boundary
    }
    throw;
}
```

The better design is usually to make the logging sink itself non-throwing at call sites that handle failures.

**Interview point:** Error handling paths need their own reliability design.

**Common mistake:** Letting diagnostic code become a new source of failures that hides the original problem.

---

## 37. What is a fallback, and when is it dangerous?

**Short answer:** A fallback is an alternate behavior used after failure; it is dangerous when it hides correctness problems or produces unsafe results.

**Detailed answer:**
Fallbacks can improve availability, such as using cached data when a network call fails. But fallback behavior must be explicit, observable, and safe. Silent fallbacks can hide outages, security failures, stale data, or corrupted state.

**Example:**

```cpp
auto config = loadConfig();
if (!config) {
    logWarning(config.error());
    config = defaultConfig();
}
```

This is reasonable only if defaults are safe and the system clearly reports that fallback mode is active.

**Interview point:** Graceful degradation should preserve safety and observability.

**Common mistake:** Catching all exceptions and continuing with guessed values.

---

## 38. What is error normalization?

**Short answer:** Error normalization converts different low-level failure forms into a consistent representation.

**Detailed answer:**
A system may use exceptions, `errno`, HTTP status codes, database errors, and library-specific return codes. Normalization converts these into a common internal error model at boundaries so the rest of the system can handle failures consistently.

**Example:**

```cpp
struct AppError {
    ErrorKind kind;
    std::string message;
};

AppError normalize(std::error_code ec) {
    if (ec == std::errc::permission_denied) {
        return {ErrorKind::PermissionDenied, ec.message()};
    }
    return {ErrorKind::Unknown, ec.message()};
}
```

Normalization should preserve enough original detail for debugging.

**Interview point:** Consistent error models reduce duplicated handling logic in large systems.

**Common mistake:** Normalizing by discarding the original error cause and diagnostic context.

---

## 39. How should error handling interact with resource cleanup?

**Short answer:** Resource cleanup should be automatic through RAII so every error path releases resources correctly.

**Detailed answer:**
Manual cleanup is fragile because every return path, exception path, and partial-construction path must remember to release resources. RAII ties cleanup to object lifetime, making cleanup deterministic during normal returns and stack unwinding.

**Example:**

```cpp
#include <fstream>
#include <mutex>

void writeData(const std::string& path) {
    std::ofstream file(path);
    std::lock_guard<std::mutex> lock(globalMutex);

    writeRecords(file); // if this throws, file and lock still clean up
}
```

The error handler can focus on reporting or recovery instead of remembering every cleanup action.

**Interview point:** RAII is the foundation of practical exception-safe C++.

**Common mistake:** Using `try`/`catch` mainly to manually clean up resources that should have had RAII wrappers.

---

## 40. How do you design a consistent error-handling strategy for a C++ codebase?

**Short answer:** Define which failures use exceptions, return values, assertions, or termination, and apply those choices consistently at boundaries.

**Detailed answer:**
A good strategy distinguishes bugs from expected failures, internal code from public boundaries, and libraries from applications. It should define where exceptions are allowed, where `std::expected` or error codes are preferred, how errors are logged, how context is attached, and which layers own recovery policy.

**Practical guidelines:**

- Use assertions or contracts for internal programming errors.
- Use exceptions when failures are exceptional and callers cannot reasonably check every operation inline.
- Use `std::expected` or error codes when failure is common and part of normal control flow.
- Convert errors at API, thread, process, plugin, and language boundaries.
- Preserve enough diagnostic context for debugging.
- Keep resource cleanup independent through RAII.

**Example boundary design:**

```cpp
std::expected<UserInput, ParseError> parseInput(std::string_view text);
User loadUser(UserId id); // may throw StorageError
int main() noexcept;      // catches, logs, converts to process exit code
```

**Interview point:** Consistency matters more than choosing one universal mechanism for every failure.

**Common mistake:** Mixing mechanisms randomly so callers cannot predict how a function reports failure.

---

## 41. What is the difference between a precondition violation and a recoverable input error?

**Short answer:** A precondition violation means the caller broke the function's contract. A recoverable input error is an expected invalid value from outside the trusted boundary.

**Detailed answer:**
Internal functions often assume their callers have already validated certain conditions. If those assumptions are violated, the program has a bug. Public APIs, command-line parsers, network handlers, and file readers receive untrusted input, so invalid data should usually be reported as a recoverable error.

**Example:**

```cpp
#include <cassert>
#include <expected>
#include <string>
#include <string_view>

int elementAt(const std::vector<int>& values, std::size_t index) {
    assert(index < values.size());
    return values[index];
}

std::expected<int, std::string> parsePositive(std::string_view text) {
    int value = parseInt(text);
    if (value <= 0) {
        return std::unexpected("value must be positive");
    }
    return value;
}
```

**Interview point:** The same condition can be a bug in one layer and a recoverable error at another layer. The boundary decides.

**Common mistake:** Treating all bad values as exceptions or all bad values as assertions without considering trust boundaries.

---

## 42. What is an error boundary?

**Short answer:** An error boundary is a layer where failures are caught, translated, logged, retried, or converted into another representation.

**Detailed answer:**
Error boundaries prevent low-level failure details from leaking everywhere. Typical boundaries include API handlers, thread entry functions, plugin interfaces, language ABI boundaries, process entry points, and asynchronous task schedulers. Inside a component, code can use a convenient mechanism; at the boundary, it converts failures into the contract expected by the caller.

**Example:**

```cpp
int main() noexcept {
    try {
        runApplication();
        return 0;
    } catch (const ConfigError& ex) {
        logError(ex.what());
        return 2;
    } catch (const std::exception& ex) {
        logError(ex.what());
        return 1;
    }
}
```

**Interview point:** A mature codebase has intentional error boundaries, not scattered catch blocks everywhere.

**Common mistake:** Catching errors too deep, where the code lacks enough context to recover or report usefully.

---

## 43. How should retry logic be designed in error handling?

**Short answer:** Retry only failures that are likely transient, limit attempts, use backoff, and make retries observable.

**Detailed answer:**
Retries are useful for temporary network failures, rate limits, timeouts, and unavailable services. They are dangerous for validation errors, permission failures, corrupted data, or non-idempotent operations. Good retry design includes maximum attempts, backoff, jitter, cancellation, logging, and clear idempotency rules.

**Example:**

```cpp
for (int attempt = 0; attempt != maxAttempts; ++attempt) {
    auto result = sendRequest();
    if (result) {
        return *result;
    }
    if (!isTransient(result.error())) {
        return std::unexpected(result.error());
    }
    waitBeforeRetry(attempt);
}
```

**Interview point:** Retry policy is part of system design, not just an error-handling convenience.

**Common mistake:** Retrying every failure and accidentally duplicating side effects or hiding permanent errors.

---

## 44. What is idempotency, and why does it matter for error handling?

**Short answer:** An idempotent operation can be repeated without changing the final result beyond the first successful execution.

**Detailed answer:**
When a call fails after partially completing, the caller may not know whether the operation took effect. Retrying a non-idempotent operation can create duplicates, double charges, repeated messages, or corrupted state. Error handling for distributed or external operations often needs idempotency keys, transaction IDs, or deduplication.

**Example:**

```cpp
struct PaymentRequest {
    std::string idempotencyKey;
    int cents;
};

auto result = chargeCustomer(request);
if (!result && isTimeout(result.error())) {
    // safe only if the service deduplicates by idempotencyKey
    result = chargeCustomer(request);
}
```

**Interview point:** Retrying safely requires understanding the operation's side effects, not only the error code.

**Common mistake:** Assuming a timeout means nothing happened.

---

## 45. What is cancellation-aware error handling?

**Short answer:** Cancellation-aware code distinguishes requested cancellation from ordinary failure and cleans up safely.

**Detailed answer:**
In concurrent and asynchronous systems, a task may stop because the user canceled it, shutdown began, a deadline expired, or a parent task failed. Treating cancellation as an ordinary error can produce noisy logs or incorrect retries. Treating every error as cancellation can hide real failures.

**Example:**

```cpp
std::expected<Result, Error> runTask(std::stop_token token) {
    while (!token.stop_requested()) {
        auto step = doStep();
        if (!step) {
            return std::unexpected(step.error());
        }
    }
    return std::unexpected(Error::Cancelled);
}
```

**Interview point:** Cancellation is a control-flow outcome with cleanup requirements; it should be represented explicitly.

**Common mistake:** Logging user-requested cancellation as a production error.

---

## 46. What is the difference between local recovery and centralized recovery?

**Short answer:** Local recovery handles a failure near where it occurs; centralized recovery handles failures at a higher boundary with broader context.

**Detailed answer:**
Local recovery is appropriate when the code knows exactly how to continue safely, such as substituting a missing optional field. Centralized recovery is better when policy depends on user experience, service-level behavior, transactions, logging, or shutdown. Too much local recovery hides problems; too much centralized recovery loses precise handling opportunities.

**Example:**

```cpp
Config readConfig() {
    Config config = parseConfigFile();
    config.timeout = config.timeout.value_or(defaultTimeout());
    return config;
}
```

This local recovery is reasonable if a missing timeout has a safe default.

**Interview point:** Recovery should happen at the layer that has both enough information and enough authority to choose the response.

**Common mistake:** Catching exceptions locally just to return a generic failure that discards useful context.

---

## 47. How should destructors behave during error handling?

**Short answer:** Destructors should not let exceptions escape, especially during stack unwinding.

**Detailed answer:**
If a destructor throws while another exception is already being unwound, the program calls `std::terminate`. Destructors should perform best-effort cleanup, call non-throwing cleanup functions, or provide an explicit `close`/`commit` function for operations whose failure must be reported.

**Example:**

```cpp
class Writer {
public:
    void close();

    ~Writer() noexcept {
        try {
            closeNoThrow();
        } catch (...) {
        }
    }

private:
    void closeNoThrow() noexcept;
};
```

Important failures should be reported before destruction through an explicit operation.

**Interview point:** RAII cleanup must be reliable even when the program is already handling another error.

**Common mistake:** Reporting important flush or commit failures only from a destructor.

---

## 48. What is diagnostic context propagation?

**Short answer:** Diagnostic context propagation carries useful failure context across layers without losing the original cause.

**Detailed answer:**
As an error moves through the system, each layer may know something useful: operation name, resource ID, request ID, user-safe description, or retryability. Good propagation enriches the error while preserving structured cause information. Poor propagation either loses the original cause or wraps it in noisy text at every layer.

**Example:**

```cpp
struct ErrorContext {
    ErrorKind kind;
    std::string operation;
    std::string resource;
    std::error_code cause;
};
```

A service can log the structured fields and show a safer message to the user.

**Interview point:** Context should improve debugging and operations without leaking secrets or destroying machine-readable error details.

**Common mistake:** Building one long string and making later code parse text to understand the failure.

---

## 49. What is the difference between reporting an error and handling an error?

**Short answer:** Reporting makes a failure visible; handling chooses a safe response or recovery action.

**Detailed answer:**
Logging an exception is not the same as handling it. A program may report an error to logs, metrics, tracing, a user interface, or a caller, but it still must decide whether to retry, fail the request, roll back, continue with degraded behavior, or terminate. Separating reporting from handling avoids duplicated logs and unclear control flow.

**Example:**

```cpp
try {
    processRequest();
} catch (const std::exception& ex) {
    recordMetric("request.failed");
    logError(ex.what());
    return ErrorResponse{500};
}
```

The response is the handling decision; logging is only reporting.

**Interview point:** Good systems define both observability behavior and control-flow behavior for failures.

**Common mistake:** Catching an exception, logging it, and then continuing as if the operation succeeded.

---

## 50. How do you review error-handling code in a C++ code review?

**Short answer:** Check whether each failure path is explicit, safe, observable, and consistent with the API contract.

**Detailed answer:**
A good review asks what can fail, who owns recovery, whether resources are cleaned up, whether context is preserved, and whether the mechanism matches the frequency and severity of failure. It also checks boundary behavior: exceptions should not cross C ABI boundaries, thread failures should be captured, destructors should not throw, and expected input errors should not be confused with bugs.

**Review checklist:**

- Are expected failures represented in the function signature or documented contract?
- Are programming bugs handled with assertions, tests, or fail-fast behavior?
- Is cleanup automatic through RAII?
- Are exceptions caught only where recovery, translation, or reporting is meaningful?
- Is diagnostic context preserved without leaking secrets?
- Are retries limited, safe, and observable?

**Interview point:** Senior-level error handling review is about contracts and system behavior, not just syntax.

**Common mistake:** Reviewing only the happy path and assuming failures are rare enough to ignore.
