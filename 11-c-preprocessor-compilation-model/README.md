# C Preprocessor and Compilation Model Interview Questions

This topic covers what happens before and during compilation. Interviewers use these questions to test whether candidates understand macros, headers, translation units, linkage, symbol resolution, and why some errors appear at compile time while others appear at link time.

## 1. What does the C/C++ preprocessor do?

**Short answer:** The preprocessor handles directives such as `#include`, `#define`, conditional compilation, and macro expansion before compilation.

**Detailed answer:**
The preprocessor transforms source code into a translation unit that the compiler then compiles. It performs textual operations, not type-checked C++ operations.

**Common directives:**

```cpp
#include <iostream>
#define MAX_SIZE 100
#ifdef DEBUG
#define LOG(x) std::cerr << x << '\n'
#endif
```

**Interview point:** Preprocessor macros are powerful but dangerous because they are not scoped, not type-safe, and can produce surprising code after expansion.

**Common mistake:** Thinking macros behave like normal functions or variables. They are textual substitutions before compilation.

---

## 2. What is the difference between `#include <file>` and `#include "file"`?

**Short answer:** Angle brackets are typically used for system or library headers. Quotes are typically used for project headers.

**Detailed answer:**
The exact search order is implementation-defined, but compilers generally search quoted includes in the current file's directory first, then configured include paths. Angle-bracket includes usually search system include paths directly.

**Example:**

```cpp
#include <vector>      // standard library header
#include "user.hpp"   // project header
```

**Interview point:** Include path configuration matters in real builds. Accidentally shadowing a system header with a local file can cause confusing behavior.

**Common mistake:** Assuming the two forms are always identical across all compilers and build systems.

---

## 3. What are include guards?

**Short answer:** Include guards prevent a header from being included multiple times in the same translation unit.

**Detailed answer:**
Headers often include other headers. Without include guards, repeated inclusion can cause redefinition errors for types, inline functions, and templates.

**Example:**

```cpp
#ifndef USER_HPP
#define USER_HPP

class User {
public:
    void login();
};

#endif
```

**Alternative:**

```cpp
#pragma once
```

`#pragma once` is widely supported and simpler, but include guards are standard and fully portable.

**Common mistake:** Using non-unique guard names, which can accidentally suppress the wrong header.

---

## 4. What is a macro, and why can macros be dangerous?

**Short answer:** A macro is a preprocessor replacement rule. It can be dangerous because it is not type-safe and may evaluate arguments multiple times.

**Detailed answer:**
Macros operate before the compiler understands types and scopes. Function-like macros can lead to precedence bugs, side effects, and poor debugging.

**Bad example:**

```cpp
#define SQUARE(x) x * x

int result = SQUARE(1 + 2); // expands to 1 + 2 * 1 + 2, result is 5
```

**Better macro:**

```cpp
#define SQUARE(x) ((x) * (x))
```

But even this can be dangerous:

```cpp
int i = 3;
int result = SQUARE(++i); // increments i twice
```

**Better C++ approach:**

```cpp
template <typename T>
T square(T x) {
    return x * x;
}
```

**Interview point:** Prefer `constexpr`, `inline` functions, templates, and scoped constants over macros when possible.

---

## 5. What is conditional compilation?

**Short answer:** Conditional compilation includes or excludes code based on preprocessor conditions.

**Detailed answer:**
Conditional compilation is often used for platform-specific code, debug builds, feature flags, and compiler-specific behavior.

**Example:**

```cpp
#ifdef _WIN32
#include <windows.h>
#else
#include <unistd.h>
#endif
```

**Debug example:**

```cpp
#ifdef DEBUG
std::cerr << "debug info\n";
#endif
```

**Interview point:** Conditional compilation can make code harder to test because different builds may compile different code paths.

**Common mistake:** Using preprocessor flags for ordinary runtime decisions that could be handled with normal code.

---

## 6. What is a translation unit?

**Short answer:** A translation unit is a source file after all included headers and preprocessing have been applied.

**Detailed answer:**
Each `.c` or `.cpp` file is usually compiled independently into an object file. The compiler sees one translation unit at a time. Later, the linker combines object files into the final executable or library.

**Example:**

```text
main.cpp + included headers -> main.o
user.cpp + included headers -> user.o
link main.o + user.o -> app
```

**Interview point:** Because translation units are compiled separately, a function can compile successfully in one file but fail at link time if its definition is missing.

---

## 7. What is the difference between declaration, definition, and linkage?

**Short answer:** A declaration introduces a name, a definition provides the entity, and linkage determines whether the same name refers to the same entity across translation units.

**Detailed answer:**
Headers commonly contain declarations. Source files commonly contain definitions. Linkage controls symbol visibility across translation units.

**Example:**

```cpp
// header.hpp
extern int globalCount; // declaration
void logMessage();      // declaration

// source.cpp
int globalCount = 0;    // definition
void logMessage() {}    // definition
```

**Types of linkage:**
- External linkage: visible across translation units.
- Internal linkage: visible only within one translation unit.
- No linkage: local names such as function-local variables.

**Common mistake:** Defining non-inline global variables in headers, causing multiple-definition errors.

---

## 8. What does `static` mean at file scope in C/C++?

**Short answer:** At file scope, `static` gives a variable or function internal linkage.

**Detailed answer:**
Internal linkage means the name is only visible within the current translation unit. This helps avoid symbol collisions across files.

**Example:**

```cpp
static int helperCount = 0;

static void helper() {
    ++helperCount;
}
```

Only code in the same `.cpp` file can refer to these names.

**Modern C++ alternative:** Use an unnamed namespace for internal linkage in C++ source files.

```cpp
namespace {
void helper() {}
}
```

**Common mistake:** Confusing file-scope `static` with class static members or function-local static variables. They are different uses of the same keyword.

---

## 9. What is a linker error?

**Short answer:** A linker error happens when object files cannot be combined because symbols are missing, duplicated, or incompatible.

**Detailed answer:**
Compilation checks syntax and type correctness within a translation unit. Linking resolves references between translation units and libraries.

**Common linker errors:**
- Undefined reference: a declaration exists but no definition is found.
- Multiple definition: the same non-inline entity is defined in more than one object file.
- Missing library: the required object code was not linked.

**Example:**

```cpp
// declared but never defined
void process();

int main() {
    process(); // compiles, but linker fails
}
```

**Interview point:** Understanding link errors requires knowing the difference between declarations and definitions.

---

## 10. What is the One Definition Rule (ODR)?

**Short answer:** The One Definition Rule says that certain entities must have exactly one definition in a program, while some entities may have identical definitions in multiple translation units under specific rules.

**Detailed answer:**
The ODR prevents conflicting definitions of the same entity. Non-inline functions and global variables with external linkage generally need one definition in the whole program. Class definitions, templates, and inline functions can appear in multiple translation units if their definitions are identical.

**Bad example:**

```cpp
// config.hpp
int globalValue = 42; // bad if included in multiple .cpp files
```

This can cause multiple-definition linker errors.

**Better:**

```cpp
// config.hpp
extern int globalValue;

// config.cpp
int globalValue = 42;
```

**Modern C++17 option:**

```cpp
inline int globalValue = 42;
```

**Common mistake:** Putting ordinary global variable definitions in headers without `inline` or `extern`.

---

## 11. What is a forward declaration, and when is it useful?

**Short answer:** A forward declaration tells the compiler that a name exists without providing the full definition.

**Detailed answer:**
Forward declarations reduce header dependencies and compile times when code only needs pointers, references, or function declarations involving a type. The full definition is required when creating objects by value, accessing members, inheriting from the type, or using `sizeof`.

**Example:**

```cpp
// user_fwd.hpp
class User;

// session.hpp
class Session {
public:
    explicit Session(User& user);

private:
    User& user_;
};
```

`Session` can store a reference to `User` without including the full `User` definition in the header.

**Interview point:** Forward declarations help manage build scalability, but overusing them can make interfaces harder to understand.

**Common mistake:** Forward-declaring a type and then trying to store it by value without including its full definition.

---

## 12. What does `inline` mean for functions and variables?

**Short answer:** `inline` allows certain definitions to appear in multiple translation units as long as the definitions are identical.

**Detailed answer:**
Despite the name, `inline` is not a command that forces the compiler to inline machine code. Its important language meaning is related to the One Definition Rule. Inline functions and C++17 inline variables can be defined in headers and included by multiple translation units.

**Example:**

```cpp
// math.hpp
inline int add(int a, int b) {
    return a + b;
}
```

C++17 inline variable example:

```cpp
// config.hpp
inline constexpr int maxConnections = 100;
```

**Interview point:** The optimizer decides whether to actually inline a call. The `inline` keyword mainly affects linkage and multiple definitions.

**Common mistake:** Thinking `inline` guarantees better performance or always removes function-call overhead.

---

## 13. Why do unnecessary header includes hurt build times?

**Short answer:** Every included header becomes part of the translation unit, so unnecessary includes increase the amount of code the compiler must parse.

**Detailed answer:**
C and C++ compile each translation unit separately. If a widely included header pulls in many other headers, small changes can force large parts of the project to rebuild. Reducing includes can improve incremental build times and reduce coupling.

**Example:**

```cpp
// Better when only a declaration is needed
class Database;

class Service {
public:
    explicit Service(Database& database);

private:
    Database& database_;
};
```

The `.cpp` file can include the full `database.hpp` when it needs to call methods.

**Interview point:** Include what you use, but avoid exposing heavy dependencies in public headers unless the interface truly requires them.

**Common mistake:** Including large headers in other headers when a forward declaration would be enough.

---

## 14. What is symbol visibility?

**Short answer:** Symbol visibility controls whether functions or variables are exported from a binary or shared library.

**Detailed answer:**
When building shared libraries, not every internal function should be visible to users of the library. Controlling visibility improves encapsulation, reduces symbol collisions, and can improve load times and optimization opportunities.

**Example concept:**

```cpp
#if defined(_WIN32)
#define API_EXPORT __declspec(dllexport)
#else
#define API_EXPORT __attribute__((visibility("default")))
#endif

class API_EXPORT Plugin {
public:
    void run();
};
```

Internal helpers can remain hidden by default compiler flags or by not marking them for export.

**Interview point:** Visibility is especially important for shared libraries, plugin systems, and ABI-stable interfaces.

**Common mistake:** Exporting every symbol from a library, which leaks implementation details and can create ABI compatibility problems.

---

## 15. How do include paths affect builds?

**Short answer:** Include paths tell the compiler where to search for headers named in `#include` directives.

**Detailed answer:**
Build systems pass include directories to the compiler with options such as `-I` or target-specific configuration. The order of include paths matters because two directories might contain headers with the same name.

**Example:**

```bash
g++ -Iinclude -Ithird_party/lib/include -c main.cpp
```

If both directories contain `config.hpp`, the compiler's search order determines which one is included.

**Interview point:** Include path bugs can cause code to compile against the wrong header version, especially in large projects or when generated headers are involved.

**Common mistake:** Relying on accidental relative include paths instead of configuring the build system clearly.

---

## 16. What is the difference between static and dynamic linking?

**Short answer:** Static linking copies library code into the final binary, while dynamic linking loads shared library code at runtime.

**Detailed answer:**
With static linking, the linker includes required object code from static libraries into the executable. This can simplify deployment but increase binary size. With dynamic linking, the executable refers to shared libraries that must be available at runtime.

**Examples:**
- Static library: `libmath.a` on Unix-like systems
- Shared library: `libmath.so` on Linux, `.dylib` on macOS, `.dll` on Windows

**Interview point:** Dynamic linking affects deployment, ABI compatibility, symbol resolution, startup behavior, and versioning.

**Common mistake:** Thinking successful compilation means all runtime libraries will be found when the program starts.

---

## 17. What is ABI, and why does it matter in C++?

**Short answer:** ABI means Application Binary Interface. It defines binary-level details needed for compiled code to work together.

**Detailed answer:**
An ABI includes calling conventions, name mangling, object layout, exception handling, vtable layout, alignment, and standard-library binary compatibility. C++ ABI compatibility is difficult because many language features affect binary layout.

**Example concern:**
A shared library exposing `std::string` in its public interface may require callers to use a compatible compiler, standard library, build mode, and ABI version.

```cpp
class Api {
public:
    std::string name() const;
};
```

**Interview point:** Source compatibility and binary compatibility are different. Code can compile from source but still be incompatible with an existing binary interface.

**Common mistake:** Changing private-looking class layout in a shared library without considering whether users depend on the binary layout.

---

## 18. What is precompiled header, and what tradeoffs does it have?

**Short answer:** A precompiled header stores the compiled state of commonly included headers to reduce repeated parsing.

**Detailed answer:**
Large projects often include the same standard and framework headers in many translation units. A precompiled header can speed up builds by compiling those stable headers once and reusing the result.

**Good candidates:**
- Stable standard library headers
- Platform SDK headers
- Large third-party framework headers

**Tradeoffs:**
- Can hide unnecessary dependencies.
- Must be rebuilt when included headers or compiler options change.
- Can make builds more sensitive to configuration mismatches.
- Does not fix poor dependency design by itself.

**Interview point:** Precompiled headers are a build optimization, not a substitute for header hygiene.

**Common mistake:** Putting frequently changing project headers into a precompiled header, causing frequent expensive rebuilds.

---

## 19. What are modules in modern C++?

**Short answer:** C++20 modules are a language feature that can reduce dependence on textual inclusion and improve build isolation.

**Detailed answer:**
Traditional headers are textually included into every translation unit, which can slow builds and expose macros or implementation details. Modules let code export selected declarations and import them without repeatedly parsing the same text in the same way.

**Conceptual example:**

```cpp
export module math;

export int add(int a, int b) {
    return a + b;
}
```

A user imports the module:

```cpp
import math;

int value = add(1, 2);
```

**Interview point:** Modules can improve build times and encapsulation, but adoption depends on compiler support, build-system support, and migration cost.

**Common mistake:** Assuming modules automatically eliminate all headers. Projects often mix modules, legacy headers, and third-party headers for a long time.

---

## 20. What are common causes of unresolved external symbol errors?

**Short answer:** Unresolved external errors happen when code references a symbol that the linker cannot find a definition for.

**Detailed answer:**
The compiler only needs declarations to compile calls. The linker must find exactly the needed definitions in object files or libraries.

**Common causes:**
- Function declared but never defined.
- Source file not added to the build.
- Library not linked or linked in the wrong order.
- Signature mismatch between declaration and definition.
- Missing template definition or explicit instantiation.
- C++ name mangling mismatch with C APIs.

**Example:**

```cpp
// header.hpp
void process(int value);

// source.cpp
void process(double value) {}
```

The declaration and definition are different functions, so a call to `process(int)` may link-fail.

**Interview point:** Linker errors are often build-graph or signature problems, not syntax problems.

**Common mistake:** Trying to fix unresolved externals by adding random includes. Headers provide declarations; linking requires definitions.

---

## 21. What is the difference between preprocessing, compiling, assembling, and linking?

**Short answer:** Preprocessing handles source transformations, compiling turns source into assembly or intermediate code, assembling creates object files, and linking combines object files and libraries into a program.

**Detailed answer:**
A C or C++ build is usually a pipeline. The preprocessor expands macros, handles `#include`, and applies conditional compilation. The compiler parses and type-checks the resulting translation unit and generates lower-level code. The assembler creates object files. The linker resolves symbols and produces an executable or library.

**Typical pipeline:**

```text
source.cpp -> preprocessed source -> assembly -> object file -> executable
```

**Example command:**

```bash
g++ -E main.cpp -o main.i   # preprocess only
g++ -S main.cpp -o main.s   # compile to assembly
g++ -c main.cpp -o main.o   # compile/assemble to object file
g++ main.o -o app           # link
```

**Interview point:** Knowing the build stages helps diagnose whether an error is from preprocessing, compilation, or linking.

**Common mistake:** Treating every build failure as a compiler error when unresolved symbols are linker errors.

---

## 22. What are macro hygiene problems?

**Short answer:** Macro hygiene problems happen when macros accidentally capture names, evaluate arguments multiple times, or change code outside their intended scope.

**Detailed answer:**
Macros are token substitution, not typed functions. They do not respect C++ scope, type checking, or evaluation rules the way functions and templates do. Poorly written macros can introduce subtle bugs.

**Bad example:**

```cpp
#define SQUARE(x) x * x

int value = SQUARE(1 + 2); // expands to 1 + 2 * 1 + 2
```

**Safer macro style when a macro is necessary:**

```cpp
#define SQUARE(x) ((x) * (x))
```

Even this still evaluates `x` twice.

```cpp
int i = 2;
int result = SQUARE(++i); // increments twice
```

**Interview point:** Prefer constants, inline functions, templates, and `constexpr` over macros when possible.

**Common mistake:** Writing function-like macros for ordinary computations that should be functions.

---

## 23. What is conditional compilation used for, and what are its risks?

**Short answer:** Conditional compilation includes or excludes code at preprocessing time, often for platforms, build modes, or feature selection.

**Detailed answer:**
Directives such as `#if`, `#ifdef`, and `#ifndef` let the build choose different code before compilation. This is useful for platform APIs, debug checks, optional features, and compatibility code.

However, excessive conditional compilation can create many code paths that are hard to test. Some branches may not compile on a developer's platform and may break silently until a specific configuration is built.

**Example:**

```cpp
#ifdef _WIN32
#include <windows.h>
#else
#include <unistd.h>
#endif
```

**Interview point:** Conditional compilation is powerful but should be isolated behind small platform abstraction layers when possible.

**Common mistake:** Spreading platform `#ifdef`s throughout business logic instead of containing them near system boundaries.

---

## 24. What are generated files in a C/C++ build, and how can they affect compilation?

**Short answer:** Generated files are source or header files produced by tools before compilation, and the build system must model their dependencies correctly.

**Detailed answer:**
Many C/C++ projects generate files from protocol definitions, parsers, configuration templates, UI definitions, or code generators. The compiler treats generated files like normal source files, but the build system must ensure they are created before anything includes or compiles them.

**Examples:**
- Protocol Buffers generating `.pb.h` and `.pb.cc` files.
- Parser generators producing source from grammar files.
- Build configuration generating a version header.

**Conceptual build dependency:**

```text
schema.proto -> generated.pb.h -> service.cpp
schema.proto -> generated.pb.cc -> generated.pb.o
```

**Interview point:** Missing generated-file dependencies often cause flaky clean builds or parallel-build failures.

**Common mistake:** Relying on an old generated header already present in the build directory instead of declaring the generator step properly.

---

## 25. What is reproducible build behavior, and why does it matter?

**Short answer:** Reproducible build behavior means the same source and build inputs produce equivalent outputs, making debugging, security review, and deployment more reliable.

**Detailed answer:**
C/C++ builds can accidentally depend on timestamps, absolute paths, environment variables, include order, compiler versions, random code generation, or undeclared generated files. These hidden inputs make builds harder to reproduce.

Reproducible builds improve confidence that the binary being tested is the same as the binary being released.

**Examples of hidden inputs:**
- `__DATE__` and `__TIME__` macros.
- Absolute source paths embedded in debug info.
- Undeclared environment variables affecting compiler flags.
- Generated files not tracked by the build graph.

**Interview point:** Build correctness is not only about successful compilation; it is also about deterministic, explainable inputs and outputs.

**Common mistake:** Debugging a production-only binary difference without first checking whether the build is reproducible.

---

## 26. What is an include dependency graph?

**Short answer:** An include dependency graph shows which files include which headers and how changes propagate through a build.

**Detailed answer:**
Every `#include` creates a dependency from one file to another. If a commonly included header changes, many translation units may need recompilation. Large include graphs slow builds, make dependencies harder to understand, and can accidentally expose implementation details across the project.

**Example:**

```text
main.cpp -> app.h -> database.h -> network.h
worker.cpp -> app.h -> database.h -> network.h
```

Changing `network.h` may force both `main.cpp` and `worker.cpp` to rebuild, even if they do not directly use networking.

**Interview point:** Reducing unnecessary header dependencies is one of the most effective ways to improve C++ build times.

**Common mistake:** Including heavy headers in public headers when a forward declaration or implementation-only include would be enough.

---

## 27. What is a transitive include, and why is relying on it risky?

**Short answer:** A transitive include is a header included indirectly through another header; relying on it is risky because that indirect relationship can change.

**Detailed answer:**
If `a.h` includes `b.h`, and `main.cpp` includes only `a.h`, then declarations from `b.h` may appear available in `main.cpp`. But `main.cpp` does not own that dependency. If `a.h` later stops including `b.h`, `main.cpp` breaks.

**Example:**

```cpp
// main.cpp
#include "service.h"

std::vector<int> values; // works only if service.h indirectly includes <vector>
```

The fix is to include what you use directly.

**Interview point:** Include hygiene makes files robust against unrelated header refactoring.

**Common mistake:** Removing an include because the file still compiles due to some unrelated transitive include.

---

## 28. What does "include what you use" mean?

**Short answer:** It means each source or header file should directly include the headers that provide the declarations it uses.

**Detailed answer:**
Include-what-you-use improves maintainability by making dependencies explicit. A `.cpp` file that uses `std::vector` should include `<vector>`. A header that exposes `std::string` in its public interface should include `<string>`.

**Example:**

```cpp
#include <string>
#include <vector>

class UserList {
public:
    void add(std::string name);

private:
    std::vector<std::string> names_;
};
```

This header is self-contained: it provides the dependencies needed to parse its declarations.

**Interview point:** A header should compile when included first in an otherwise empty translation unit.

**Common mistake:** Assuming another project header will always include the standard library header you need.

---

## 29. What is a self-contained header?

**Short answer:** A self-contained header can be included by itself and still compile correctly.

**Detailed answer:**
A self-contained header includes all declarations needed for its own contents and does not depend on include order. This property makes headers easier to test, reuse, and refactor.

**Example test:**

```cpp
// compile this file by itself
#include "widget.h"

int main() {}
```

If this fails because `widget.h` depends on a prior include, the header is not self-contained.

**Interview point:** Header self-containment prevents fragile build failures caused by include order.

**Common mistake:** Writing headers that compile only when included after a specific project-wide header.

---

## 30. What is a unity build?

**Short answer:** A unity build combines many source files into one larger translation unit to reduce build overhead.

**Detailed answer:**
In a unity build, the build system generates a file that includes multiple `.cpp` files. This can reduce repeated header parsing and speed full builds. It can also hide missing includes, expose name collisions, and change internal-linkage interactions.

**Conceptual example:**

```cpp
// unity.cpp
#include "a.cpp"
#include "b.cpp"
#include "c.cpp"
```

Unity builds are a build optimization, not a replacement for correct source hygiene.

**Interview point:** Code should compile both in normal builds and unity builds if the project supports both.

**Common mistake:** Letting unity builds mask missing includes that break non-unity or incremental builds.

---

## 31. What is a macro collision?

**Short answer:** A macro collision happens when a macro name unintentionally replaces tokens in unrelated code.

**Detailed answer:**
Macros are global within the preprocessed translation unit and do not obey C++ scopes or namespaces. A short macro name can interfere with variables, functions, enum values, or library headers included later.

**Example:**

```cpp
#define min(a, b) ((a) < (b) ? (a) : (b))

int min = 3; // preprocessor may rewrite this token unexpectedly
```

This is why macros should be rare, clearly named, and undefined when used only locally.

**Interview point:** Macro names are part of the global preprocessing environment.

**Common mistake:** Defining common names such as `min`, `max`, `ERROR`, or `Status` as macros in public headers.

---

## 32. What are feature-test macros?

**Short answer:** Feature-test macros let code detect whether a language or library feature is available.

**Detailed answer:**
The C++ standard library exposes macros such as `__cpp_lib_expected` or `__cpp_concepts` to indicate support for features. They are useful when writing portable code across compiler and standard-library versions.

**Example:**

```cpp
#ifdef __cpp_lib_expected
#include <expected>
#endif
```

Feature-test macros are better than checking only compiler names because support depends on compiler version, standard mode, and library implementation.

**Interview point:** Feature detection is more robust than platform or compiler detection when selecting language/library capabilities.

**Common mistake:** Assuming `__cplusplus` alone proves every library feature from that standard is available.

---

## 33. What is conditional inclusion using `__has_include`?

**Short answer:** `__has_include` checks whether a header is available during preprocessing.

**Detailed answer:**
`__has_include` can be used to conditionally include optional headers. It is helpful for portability, but it should not replace proper dependency management. A header being present does not always mean the desired API is complete or compatible.

**Example:**

```cpp
#if __has_include(<version>)
#include <version>
#endif
```

It is often combined with feature-test macros for more reliable checks.

**Interview point:** Header availability and feature availability are related but not identical.

**Common mistake:** Using `__has_include` as the only test for a feature's semantics.

---

## 34. What is an archive or static library link order issue?

**Short answer:** Static library link order matters on many linkers because libraries are searched left to right.

**Detailed answer:**
When linking static archives, the linker usually extracts object files from a library only to satisfy unresolved symbols seen so far. If a library appears before the object or library that needs it, its symbols may not be pulled in.

**Example:**

```text
# often works
app.o -lservice -lcrypto

# may fail if service needs crypto but crypto appears too early
app.o -lcrypto -lservice
```

Build systems should model link dependencies so libraries appear in a correct order.

**Interview point:** Link errors can be caused by ordering, not only missing code.

**Common mistake:** Randomly rearranging libraries until the link succeeds without understanding dependency direction.

---

## 35. What is a duplicate symbol link error?

**Short answer:** A duplicate symbol error occurs when the linker finds more than one strong definition of the same symbol.

**Detailed answer:**
C++ allows many declarations but generally only one definition of an object or non-inline function with external linkage. Defining a function or variable in a header without `inline` can cause every including translation unit to provide a definition.

**Example:**

```cpp
// bad in a header
int globalCount = 0;

void helper() {}
```

Use `extern` declarations, `inline` variables/functions, or internal linkage as appropriate.

**Interview point:** Duplicate symbol errors are often ODR violations visible at link time.

**Common mistake:** Putting ordinary global variable definitions in headers.

---

## 36. What is an inline namespace used for?

**Short answer:** An inline namespace lets names appear as if they are in the enclosing namespace while still encoding a version or ABI namespace.

**Detailed answer:**
Libraries use inline namespaces to version APIs or ABIs without forcing users to spell the version namespace everywhere. Symbols can still carry version information, which helps manage compatibility.

**Example:**

```cpp
namespace lib {
inline namespace v2 {
class Widget {};
}
}

lib::Widget widget; // means lib::v2::Widget
```

Changing inline namespaces can affect ABI and linking behavior.

**Interview point:** Inline namespaces are commonly used by libraries for versioning and ABI control.

**Common mistake:** Treating inline namespaces as purely cosmetic and ignoring symbol compatibility.

---

## 37. What is symbol interposition?

**Short answer:** Symbol interposition allows one symbol definition to override or intercept another at link or load time.

**Detailed answer:**
On some platforms, especially ELF-based Unix systems, dynamic linking can resolve symbols in ways that allow one shared object or executable to interpose on another symbol. This can be used intentionally for wrappers, profiling, or compatibility, but it can also cause surprising behavior and optimization limits.

**Conceptual example:**

```text
executable defines malloc wrapper
shared library calls malloc
loader may resolve malloc to wrapper depending on visibility and link rules
```

Symbol visibility and linker options can reduce accidental interposition.

**Interview point:** Dynamic linking behavior can affect both correctness and optimization.

**Common mistake:** Assuming every function call inside a shared library must resolve to that library's own definition.

---

## 38. What is link-time dead code elimination?

**Short answer:** Link-time dead code elimination removes unused functions or data from the final binary.

**Detailed answer:**
Compilers and linkers can place functions or data in separate sections and then discard unused sections during linking. This reduces binary size, especially in template-heavy or embedded projects.

**Conceptual flags:**

```text
-ffunction-sections -fdata-sections -Wl,--gc-sections
```

Exact flags depend on compiler, linker, and platform.

**Interview point:** Dead code elimination at link time depends on how code is emitted and how the linker sees references.

**Common mistake:** Expecting unused code to disappear from a binary without enabling the required compiler and linker support.

---

## 39. What are compiler diagnostics and warning levels?

**Short answer:** Compiler diagnostics report suspicious, non-portable, or invalid code; warning levels control how much the compiler reports.

**Detailed answer:**
Warnings catch many bugs before runtime, including uninitialized variables, narrowing conversions, missing returns, shadowing, and suspicious comparisons. Many teams treat warnings as errors in CI to prevent warning buildup.

**Example flags:**

```text
-Wall -Wextra -Wpedantic -Werror
```

Different compilers warn about different things, so cross-compiler builds can improve coverage.

**Interview point:** A clean warning policy is part of build quality, not just style preference.

**Common mistake:** Enabling strict warnings only after the project has accumulated thousands of warnings.

---

## 40. How do you debug complex C/C++ build failures?

**Short answer:** Identify the failing build stage, reduce the failing command, inspect dependencies and flags, then fix the underlying declaration, compilation, or linking issue.

**Detailed answer:**
C/C++ build failures can come from preprocessing, compilation, assembly, linking, code generation, or packaging. The first step is to determine which stage failed. Then inspect the exact command line, include paths, macro definitions, language standard, linked libraries, and generated-file dependencies.

**Practical checklist:**

- Is the error from the compiler or linker?
- Which translation unit or library failed?
- What exact command was run?
- Are include paths and macro definitions correct?
- Is a generated file missing or stale?
- Is there an ODR, visibility, ABI, or link-order issue?
- Does a clean build reproduce the failure?

**Example commands:**

```text
compiler -E file.cpp   # inspect preprocessed output
compiler -c file.cpp   # compile one translation unit
nm library.a           # inspect symbols on Unix-like systems
```

**Interview point:** Build debugging is systematic: classify the stage before changing code or flags.

**Common mistake:** Treating all build failures as compiler errors and ignoring preprocessing or linking evidence.

---

## 41. What is a compilation database?

**Short answer:** A compilation database records the exact compiler command used for each translation unit.

**Detailed answer:**
Tools such as language servers, static analyzers, indexers, and refactoring tools need to know the same include paths, macro definitions, language standard, and compiler options used by the real build. A compilation database, commonly `compile_commands.json`, provides that information in a machine-readable form.

**Example entry:**

```json
{
  "directory": "/project/build",
  "command": "c++ -std=c++20 -Iinclude -DDEBUG -c ../src/main.cpp",
  "file": "../src/main.cpp"
}
```

Without accurate flags, tools may report false errors or miss real problems.

**Interview point:** In C/C++, source files cannot be analyzed correctly without their build context.

**Common mistake:** Running static analysis with different macros or include paths than the actual build.

---

## 42. What is dependency scanning in C/C++ builds?

**Short answer:** Dependency scanning discovers which headers or generated files a translation unit depends on so the build system can rebuild when they change.

**Detailed answer:**
A `.cpp` file often depends on many headers through direct and transitive includes. Build systems need accurate dependency information to avoid stale builds and unnecessary rebuilds. Compilers can emit dependency files that list included headers for each object file.

**Example concept:**

```text
main.o: main.cpp app.hpp config.hpp platform.hpp
```

If `config.hpp` changes, `main.cpp` must be recompiled even if `main.cpp` itself did not change.

**Interview point:** Correct incremental builds depend on accurate dependency tracking.

**Common mistake:** Manually listing only direct headers and missing transitive or generated dependencies.

---

## 43. What is the difference between public and private compile definitions?

**Short answer:** Public compile definitions affect both a target and its consumers; private definitions affect only the target being built.

**Detailed answer:**
A macro that changes a library's public headers may need to be visible to every consumer. A macro used only inside implementation files should remain private. Leaking private macros makes builds fragile because consumers may accidentally depend on internal configuration.

**Example:**

```text
PUBLIC:  USE_LIBRARY_SHARED_ABI
PRIVATE: ENABLE_INTERNAL_LOGGING
```

If a public header changes declarations based on a macro, consumers must compile with the same macro setting.

**Interview point:** Build configuration is part of an API when it changes public declarations or ABI.

**Common mistake:** Marking all definitions as global and creating hidden coupling between unrelated targets.

---

## 44. What is an object file?

**Short answer:** An object file is the compiled output of one translation unit before final linking.

**Detailed answer:**
After preprocessing and compilation, the compiler produces machine code plus symbols, relocation information, debug information, and references to external definitions. The linker combines object files and libraries into an executable or shared library.

**Conceptual flow:**

```text
main.cpp -> main.o
util.cpp -> util.o
main.o + util.o + libraries -> executable
```

Object files explain why missing definitions, duplicate definitions, and symbol visibility problems are usually linker issues rather than parser issues.

**Interview point:** Understanding object files helps separate compile-time errors from link-time errors.

**Common mistake:** Expecting the compiler to know whether every externally declared function has a definition somewhere else.

---

## 45. What is relocation in linking?

**Short answer:** Relocation adjusts addresses and symbol references when object files are combined into a final binary.

**Detailed answer:**
Object files are compiled separately, so they cannot know final addresses of all functions, global variables, or sections. The linker resolves symbols and patches machine code or data references to point to the final locations. Some relocation is also involved in dynamic linking and position-independent code.

**Conceptual example:**

```text
call external_function
```

The compiler emits a placeholder reference, and the linker later resolves it to the actual address or dynamic symbol entry.

**Interview point:** Relocation is why linking is more than just concatenating object files.

**Common mistake:** Treating unresolved symbols as runtime errors when they are usually link-time resolution failures.

---

## 46. What is position-independent code?

**Short answer:** Position-independent code is machine code that can run correctly regardless of where it is loaded in memory.

**Detailed answer:**
Shared libraries are often loaded at different addresses in different processes. Position-independent code uses relative addressing and indirection so the loader can map the library without rewriting large amounts of code. On many Unix-like platforms, shared libraries are built with flags such as `-fPIC`.

**Example flag:**

```text
-fPIC
```

Exact requirements depend on architecture and platform.

**Interview point:** Shared-library build flags affect whether code can be linked and loaded efficiently.

**Common mistake:** Building a static library without PIC and later trying to link it into a shared library on a platform that requires PIC.

---

## 47. What is a generated header, and what build risks does it introduce?

**Short answer:** A generated header is a header produced by a build step rather than written directly by a developer.

**Detailed answer:**
Generated headers commonly contain configuration values, protocol definitions, version information, reflection metadata, or code produced by tools. They require correct build ordering: the generator must run before any translation unit includes the generated header. They also require dependency tracking so changes to generator inputs trigger regeneration and recompilation.

**Example:**

```cpp
#include "generated/config.hpp"
```

If the file is stale or missing, the build may fail or silently use outdated definitions.

**Interview point:** Generated code must be treated as part of the dependency graph, not as a side effect.

**Common mistake:** Relying on a developer's previous local build to have generated files that CI does not have.

---

## 48. What is a build configuration mismatch?

**Short answer:** It happens when different parts of a program are compiled with incompatible assumptions or flags.

**Detailed answer:**
C/C++ binaries can break when libraries and consumers disagree about language standard, debug vs release mode, runtime library, exception settings, RTTI, structure packing, macros that affect public headers, or ABI-related options. The code may compile but fail to link, crash, or corrupt data at runtime.

**Example risks:**

```text
Library built with exceptions disabled
Application expects exceptions across the boundary

Header compiled with different structure packing in two modules
```

**Interview point:** Build settings are part of binary compatibility.

**Common mistake:** Debugging an apparent memory bug without checking that all modules were built with compatible flags.

---

## 49. What is the difference between build-time configuration and run-time configuration?

**Short answer:** Build-time configuration changes what code is compiled; run-time configuration changes behavior of an already-built program.

**Detailed answer:**
Build-time configuration uses macros, generated headers, compiler flags, selected source files, or linked libraries. Run-time configuration uses command-line arguments, environment variables, config files, service discovery, or feature settings. Build-time choices can improve optimization or portability, but too many variants make testing and deployment harder.

**Example:**

```cpp
#ifdef ENABLE_TRACING
void trace(std::string_view message);
#endif
```

A run-time flag would keep the code present but enable or disable behavior while the program runs.

**Interview point:** Prefer run-time configuration when behavior must vary after deployment; use build-time configuration for platform, ABI, or code inclusion decisions.

**Common mistake:** Creating many compile-time variants for ordinary product settings and then testing only one of them.

---

## 50. How do you review C/C++ build-system changes?

**Short answer:** Check whether the change preserves correct dependencies, target boundaries, reproducibility, portability, and ABI assumptions.

**Detailed answer:**
Build changes can break code without touching source logic. A good review checks whether includes and link dependencies are target-specific, generated files are ordered correctly, public flags are intentionally public, warnings remain consistent, and platform-specific logic is isolated. It also checks whether the change affects binary compatibility or deployment artifacts.

**Review checklist:**

- Are include paths and compile definitions scoped to the right target?
- Are generated files declared as build outputs and dependencies?
- Are libraries linked in the right order and visibility?
- Does the change work for clean builds, not just incremental local builds?
- Are debug/release, static/shared, and platform variants considered?
- Does it change ABI-relevant flags or public macros?

**Interview point:** Senior build review focuses on reproducibility and dependency boundaries, not just making the local build pass.

**Common mistake:** Adding global flags or include paths because they fix one target while accidentally changing many others.
