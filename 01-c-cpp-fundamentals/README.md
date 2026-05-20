# C/C++ Fundamentals Interview Questions

This topic covers the language basics that interviewers often use to check whether a candidate understands how C and C++ actually work, not just how to write syntax.

## 1. What is the difference between C and C++?

**Short answer:** C is a procedural language, while C++ supports procedural, object-oriented, generic, and modern resource-management styles.

**Detailed answer:**
C focuses on functions, structs, pointers, and manual memory management. It gives the programmer direct control over memory and hardware-level details, which is why it is common in embedded systems, operating systems, and performance-critical code.

C++ was designed to be mostly compatible with C, but it adds features such as classes, constructors/destructors, function overloading, templates, exceptions, references, RAII, smart pointers, and the STL. Modern C++ encourages safer abstractions while still allowing low-level control when needed.

**Example:**

```cpp
#include <iostream>

class Counter {
public:
    Counter() : value(0) {}
    void increment() { ++value; }
    int get() const { return value; }

private:
    int value;
};

int main() {
    Counter c;
    c.increment();
    std::cout << c.get() << '\n';
}
```

**Follow-up questions:**
- Can C code usually be compiled as C++ code without changes?
- Why might a project choose C instead of C++?

**Common mistake:** Saying "C++ is just C with classes." That ignores templates, RAII, the STL, move semantics, and modern C++ design.

---

## 2. What is the difference between declaration and definition?

**Short answer:** A declaration introduces a name and its type. A definition actually creates the object or provides the function body.

**Detailed answer:**
A declaration tells the compiler that something exists. A definition provides the storage or implementation. A program can have multiple declarations for the same entity, but usually only one definition.

**Example:**

```cpp
extern int count;        // declaration
int add(int a, int b);   // declaration

int count = 0;           // definition

int add(int a, int b) {  // definition
    return a + b;
}
```

**Interview point:** This matters when working with header files and source files. Headers usually contain declarations; source files contain definitions.

**Common mistake:** Defining global variables in header files without `extern`, which can cause multiple-definition linker errors.

---

## 3. What is the difference between compile time and runtime?

**Short answer:** Compile time is when source code is translated into machine code. Runtime is when the program is actually executing.

**Detailed answer:**
Compile-time errors include syntax errors, missing declarations, type errors, and template instantiation errors. Runtime errors happen while the program runs, such as segmentation faults, division by zero, invalid memory access, or logic errors.

**Example:**

```cpp
int x = "hello"; // compile-time error: invalid conversion

int* p = nullptr;
*p = 10;         // runtime error: likely crash
```

**Follow-up questions:**
- What can be checked at compile time in C++ templates?
- What is `constexpr` used for?

---

## 4. What is the difference between stack and heap memory?

**Short answer:** Stack memory is automatically managed for local variables and function calls. Heap memory is manually or dynamically managed and can live beyond a single function call.

**Detailed answer:**
The stack is usually fast and automatically cleaned up when a function returns. It stores local variables, return addresses, and function call frames. The heap is used for dynamic allocation with `malloc/free` in C or `new/delete` and smart pointers in C++.

**Example:**

```cpp
void example() {
    int local = 10;              // stack
    int* dynamicValue = new int(20); // heap

    delete dynamicValue;
}
```

**Modern C++ note:** Prefer RAII containers or smart pointers instead of raw `new` and `delete`.

```cpp
#include <memory>

void example() {
    auto value = std::make_unique<int>(20);
}
```

**Common mistake:** Thinking heap memory is always better because it is larger. Heap allocation is usually slower and must be managed carefully.

---

## 5. What is a pointer?

**Short answer:** A pointer is a variable that stores the memory address of another object.

**Detailed answer:**
Pointers are central to C and still important in C++. They allow direct memory access, dynamic allocation, arrays, linked data structures, and interaction with low-level APIs. A pointer has its own address and value; its value is usually the address of another object.

**Example:**

```cpp
int x = 42;
int* p = &x;

std::cout << *p << '\n'; // prints 42
```

`&x` means "address of x". `*p` means "the object pointed to by p".

**Follow-up questions:**
- What is a null pointer?
- What is a dangling pointer?
- What is pointer arithmetic?

**Common mistake:** Confusing the pointer variable with the value it points to.

---

## 6. What is a reference in C++?

**Short answer:** A reference is an alias for an existing object.

**Detailed answer:**
Unlike pointers, references must usually be initialized when created and cannot be reseated to refer to a different object. They are commonly used for function parameters to avoid copying and to allow modification of the caller's object.

**Example:**

```cpp
void increment(int& value) {
    ++value;
}

int main() {
    int x = 10;
    increment(x);
    std::cout << x << '\n'; // 11
}
```

**Pointer vs reference:**

```cpp
int a = 1;
int b = 2;

int* p = &a;
p = &b;       // pointer can point somewhere else

int& r = a;
r = b;        // assigns b's value to a, does not rebind r
```

**Common mistake:** Saying references are always implemented differently from pointers. Compilers may implement references using addresses internally, but the language rules are different.

---

## 7. What is the difference between pass by value, pointer, and reference?

**Short answer:** Pass by value copies the argument. Pass by pointer passes an address. Pass by reference passes an alias.

**Detailed answer:**
Use pass by value when the function does not need to modify the caller's object and copying is cheap. Use pass by reference when modification is needed or when copying would be expensive. Use `const` reference for large read-only objects. Use pointers when null is meaningful or reseating is needed.

**Example:**

```cpp
void byValue(int x) {
    x = 100;
}

void byPointer(int* x) {
    if (x) {
        *x = 100;
    }
}

void byReference(int& x) {
    x = 100;
}
```

**Best practice:**

```cpp
void printName(const std::string& name) {
    std::cout << name << '\n';
}
```

This avoids copying while preventing modification.

---

## 8. What is `const` used for?

**Short answer:** `const` marks something as read-only through that name.

**Detailed answer:**
`const` improves safety and communicates intent. It can apply to variables, pointers, function parameters, return values, and member functions.

**Examples:**

```cpp
const int x = 10; // x cannot be modified

const int* p1 = &x; // pointer to const int; *p1 cannot be modified
int* const p2 = nullptr; // const pointer; p2 cannot point elsewhere

class User {
public:
    std::string name() const {
        return name_;
    }

private:
    std::string name_;
};
```

**Interview tip:** Read pointer declarations from right to left:
- `const int* p`: pointer to const int
- `int* const p`: const pointer to int
- `const int* const p`: const pointer to const int

---

## 9. What is undefined behavior?

**Short answer:** Undefined behavior means the C/C++ standard gives no rules for what happens.

**Detailed answer:**
When a program has undefined behavior, anything can happen: it may appear to work, crash, produce wrong output, or behave differently after optimization. Compilers assume undefined behavior does not happen and may optimize based on that assumption.

**Examples:**

```cpp
int* p = nullptr;
*p = 5; // undefined behavior
```

```cpp
int arr[3] = {1, 2, 3};
int x = arr[10]; // undefined behavior
```

```cpp
int x = 2147483647;
++x; // signed integer overflow is undefined behavior
```

**Common mistake:** Saying undefined behavior means "the program will crash." A crash is only one possible result.

---

## 10. What happens during the C/C++ build process?

**Short answer:** The main stages are preprocessing, compilation, assembly, and linking.

**Detailed answer:**
1. **Preprocessing:** Handles `#include`, `#define`, and conditional compilation.
2. **Compilation:** Converts preprocessed source code into assembly or intermediate representation.
3. **Assembly:** Converts assembly into object files.
4. **Linking:** Combines object files and libraries into an executable.

**Example:**

```bash
g++ -E main.cpp -o main.i   # preprocessing
g++ -S main.cpp -o main.s   # compilation to assembly
g++ -c main.cpp -o main.o   # object file
g++ main.o -o app           # linking
```

**Follow-up questions:**
- What is a linker error?
- What is the difference between static and dynamic linking?
- Why should function definitions usually not be placed in headers unless they are `inline` or templates?

---

## 11. What is the difference between initialization and assignment?

**Short answer:** Initialization creates an object with an initial value. Assignment changes the value of an object that already exists.

**Detailed answer:**
Initialization happens when an object is created. Assignment happens after the object already exists. This distinction matters for `const` objects, references, class members, and performance.

**Example:**

```cpp
std::string a = "hello"; // initialization
std::string b;           // default initialization
b = "world";             // assignment
```

For class members, prefer member initializer lists when constructing objects.

```cpp
class User {
public:
    User(std::string name) : name_(std::move(name)) {}

private:
    std::string name_;
};
```

**Interview point:** References and `const` members must be initialized; they cannot be assigned later in the constructor body.

**Common mistake:** Treating `T x = value;` as assignment. It is initialization.

---

## 12. What is the difference between `auto`, `decltype`, and explicit types?

**Short answer:** `auto` deduces a type from an initializer. `decltype` produces the declared or expression type. Explicit types state the type directly.

**Detailed answer:**
`auto` is useful when the type is obvious from the initializer or too verbose to write. `decltype` is useful in templates and generic code when preserving exact types matters.

**Example:**

```cpp
auto x = 42;        // int
const int value = 7;
auto y = value;     // int, top-level const is dropped

decltype(value) z = 7; // const int
```

`decltype((value))` is different from `decltype(value)` because the extra parentheses make it an expression.

```cpp
decltype(value) a = 1;   // const int
decltype((value)) b = a; // const int&
```

**Interview point:** `auto` can improve readability, but explicit types are clearer when deduction would surprise the reader.

**Common mistake:** Assuming `auto` always preserves references and top-level `const`. It usually does not unless written as `auto&`, `const auto&`, or `decltype(auto)`.

---

## 13. What is the difference between `constexpr`, `const`, and `consteval`?

**Short answer:** `const` means read-only through a name. `constexpr` means usable in constant expressions when inputs allow it. `consteval` requires compile-time evaluation.

**Detailed answer:**
`const` is about mutability. It does not always mean compile-time constant. `constexpr` can define variables or functions that may be evaluated at compile time. `consteval` functions must produce a compile-time result.

**Example:**

```cpp
const int runtimeConst = std::rand(); // read-only, not compile-time constant
constexpr int size = 16;              // compile-time constant

constexpr int square(int x) {
    return x * x;
}

consteval int buildId() {
    return 2026;
}
```

**Interview point:** A `constexpr` function can still run at runtime if called with runtime values. A `consteval` function cannot.

**Common mistake:** Saying every `const` variable is known at compile time.

---

## 14. What is name mangling, and why does it matter?

**Short answer:** Name mangling encodes extra type information into symbol names so C++ can support features such as function overloading.

**Detailed answer:**
C++ functions can be overloaded by parameter types. Linkers usually work with symbol names, so compilers encode function signatures into linker symbols. C does not use C++-style name mangling.

**Example:**

```cpp
void log(int value);
void log(const char* value);
```

These two functions need different linker symbols even though both are named `log` in source code.

When exposing C++ functions to C code or dynamic loaders, use `extern "C"` to request C linkage.

```cpp
extern "C" void plugin_init();
```

**Interview point:** `extern "C"` affects linkage naming, not whether the implementation can be written in C++.

**Common mistake:** Forgetting `extern "C"` for plugin entry points or C interop, causing symbol lookup failures.

---

## 15. What is the difference between static storage duration, automatic storage duration, and dynamic storage duration?

**Short answer:** Static storage lasts for the program lifetime, automatic storage usually lasts for a block scope, and dynamic storage lasts until explicitly released or managed by an owning object.

**Detailed answer:**
Storage duration is about how long the memory for an object exists.

**Examples:**

```cpp
int globalValue = 1; // static storage duration

void function() {
    int local = 2;              // automatic storage duration
    static int cached = 3;      // static storage duration
    int* dynamic = new int(4);  // dynamic storage duration
    delete dynamic;
}
```

**Modern C++ note:** Prefer containers and smart pointers for dynamic storage so lifetime is tied to RAII objects.

**Interview point:** Storage duration is different from scope. A function-local `static` has local scope but static storage duration.

**Common mistake:** Saying every local variable is stored on the stack. Function-local `static` variables are not automatic objects.

---

## 16. What are lvalues and rvalues?

**Short answer:** An lvalue generally refers to an object with identity. An rvalue is usually a temporary value or a value that can be moved from.

**Detailed answer:**
Value categories affect overload resolution, reference binding, move semantics, and template deduction. A named variable is an lvalue even if its type is an rvalue reference. Temporaries and many expression results are rvalues.

**Example:**

```cpp
int x = 10;
int& ref = x;        // x is an lvalue
int&& temp = 20;     // 20 is an rvalue

std::string a = "hello";
std::string b = std::move(a); // casts a to an rvalue expression
```

**Interview point:** `std::move` does not move by itself. It changes the value category so move construction or move assignment can be selected.

**Common mistake:** Saying an rvalue reference variable is always an rvalue. A named rvalue reference is an lvalue expression.

---

## 17. What is the difference between `sizeof` and `strlen`?

**Short answer:** `sizeof` gives the size of a type or object in bytes at compile time in many cases. `strlen` counts characters in a null-terminated C string at runtime.

**Detailed answer:**
`sizeof` includes all bytes in an object, including padding and null terminators in arrays. `strlen` walks through memory until it finds `\0`, so it requires a valid null-terminated string.

**Example:**

```cpp
#include <cstring>

char text[] = "hello";

std::size_t a = sizeof(text); // 6, includes '\0'
std::size_t b = std::strlen(text); // 5
```

For a pointer, `sizeof` gives the pointer size, not the size of the pointed-to array.

```cpp
const char* p = text;
std::size_t c = sizeof(p); // size of pointer
```

**Common mistake:** Using `sizeof(pointer)` to determine the length of a dynamically allocated array or C string.

---

## 18. What are C++ casts, and why are they preferred over C-style casts?

**Short answer:** C++ casts make the programmer's intent more explicit than C-style casts.

**Detailed answer:**
C++ provides several named casts for different purposes. `static_cast` is for well-defined conversions such as numeric conversions or upcasts. `const_cast` adds or removes constness. `reinterpret_cast` performs low-level bit reinterpretation and should be used carefully. `dynamic_cast` checks polymorphic casts at runtime.

**Example:**

```cpp
double value = 3.14;
int truncated = static_cast<int>(value);

const int x = 10;
const int* p = &x;
int* q = const_cast<int*>(p); // modifying x through q would be undefined behavior
```

**Interview point:** C-style casts can hide whether the code is doing a static conversion, removing const, or reinterpreting bits. Named casts are easier to search for and review.

**Common mistake:** Treating `reinterpret_cast` as a safe way to convert any object representation into any type.

---

## 19. What are integer promotions and usual arithmetic conversions?

**Short answer:** C and C++ often convert smaller integer types and mixed arithmetic operands to common types before evaluating expressions.

**Detailed answer:**
Types such as `char`, `short`, and `bool` are often promoted to `int` before arithmetic. When signed and unsigned types are mixed, conversions can produce surprising results because the signed value may be converted to an unsigned type.

**Example:**

```cpp
unsigned int u = 1;
int i = -2;

if (i < u) {
    // may be false because i is converted to unsigned
}
```

**Interview point:** Signed/unsigned comparisons are a common source of bugs, especially with container sizes because `size()` returns an unsigned type.

**Common mistake:** Assuming arithmetic always happens in the original operand types.

---

## 20. What is the difference between `nullptr`, `NULL`, and `0`?

**Short answer:** `nullptr` is the modern C++ null pointer literal with its own type. `NULL` is a macro, often defined as `0` or `0L`. `0` is an integer literal that can also convert to a null pointer.

**Detailed answer:**
`nullptr` was introduced to avoid overload ambiguity and make null pointer intent explicit. It has type `std::nullptr_t` and converts to pointer types, but not to ordinary integer types.

**Example:**

```cpp
void f(int);
void f(int*);

f(nullptr); // calls f(int*)
f(0);       // calls f(int)
```

Depending on how `NULL` is defined, `f(NULL)` may call the wrong overload or be ambiguous.

**Interview point:** Prefer `nullptr` in modern C++ code. It communicates pointer intent clearly and avoids overload surprises.

**Common mistake:** Using `NULL` in C++ out of habit from C code.

---

## 21. What is the difference between scope, lifetime, and storage duration?

**Short answer:** Scope controls where a name can be used, lifetime controls when an object exists, and storage duration controls how long the object's storage lasts.

**Detailed answer:**
These concepts are related but not the same. Scope is a compile-time name visibility rule. Lifetime is the runtime period during which an object is valid to use. Storage duration describes how long memory for the object is reserved.

**Example:**

```cpp
int* leakedPointer;

void example() {
    int local = 42;      // block scope, automatic storage, lifetime until scope exit
    leakedPointer = &local;
}
```

After `example` returns, the name `local` is out of scope and the object lifetime has ended. `leakedPointer` still contains an address, but using it would be undefined behavior.

**Interview point:** A pointer can outlive the object it points to. Valid pointer syntax does not imply valid object lifetime.

**Common mistake:** Saying "out of scope" and "destroyed" as if they are always identical. They often happen together for automatic objects, but they are different concepts.

---

## 22. What is linkage, and how is it different from scope?

**Short answer:** Scope is where a name is visible in source code. Linkage determines whether declarations in different scopes or translation units refer to the same entity.

**Detailed answer:**
A name can have external linkage, internal linkage, or no linkage. External linkage allows the same entity to be referred to from different translation units. Internal linkage restricts the entity to one translation unit. Scope does not by itself determine whether another translation unit can refer to the same object or function.

**Example:**

```cpp
// file1.cpp
int globalValue = 1;        // external linkage
static int fileLocal = 2;   // internal linkage

// file2.cpp
extern int globalValue;     // refers to file1.cpp's globalValue
```

**Interview point:** Linkage is central to understanding headers, `extern`, `static`, anonymous namespaces, and linker errors.

**Common mistake:** Thinking a global variable is always visible everywhere automatically. Other files need a declaration, and internal linkage can intentionally prevent cross-file access.

---

## 23. What are cv-qualifiers?

**Short answer:** cv-qualifiers are `const` and `volatile`; they qualify a type and affect how an object can be accessed through that type.

**Detailed answer:**
`const` prevents modification through a particular name or access path. `volatile` tells the compiler that accesses have observable behavior and should not be optimized away. The combined term "cv-qualified" appears often in type deduction, overload resolution, and template code.

**Example:**

```cpp
const int value = 10;
const int* pointerToConst = &value;

volatile int* hardwareRegister = reinterpret_cast<volatile int*>(0x40000000);
```

Top-level cv-qualifiers may be dropped in some type deduction contexts.

```cpp
const int x = 1;
auto y = x; // y is int, not const int
```

**Interview point:** `const` is about access through a type, not necessarily about physical immutability of memory.

**Common mistake:** Thinking `volatile` is a thread-safety feature. It is not a replacement for atomics or mutexes.

---

## 24. What is the difference between trivial, standard-layout, and POD types?

**Short answer:** These are type property categories used to describe low-level behavior, layout guarantees, and compatibility with C-style operations.

**Detailed answer:**
A trivial type has simple compiler-generated construction, copying, and destruction behavior. A standard-layout type has layout rules intended to support interoperability with C-like layouts. Older C++ used the term POD, meaning Plain Old Data, for types that were both trivial and standard-layout; modern C++ has moved toward more precise traits.

**Example:**

```cpp
#include <type_traits>

struct Point {
    int x;
    int y;
};

static_assert(std::is_trivially_copyable_v<Point>);
static_assert(std::is_standard_layout_v<Point>);
```

**Interview point:** These properties matter for serialization, binary protocols, shared memory, C interop, and low-level optimization, but they do not make arbitrary byte copying safe for every type.

**Common mistake:** Treating any simple-looking class as safe to `memcpy`, especially if it owns resources or has non-trivial invariants.

---

## 25. What is the as-if rule?

**Short answer:** The as-if rule allows the compiler to transform code in any way that preserves observable behavior.

**Detailed answer:**
Compilers can reorder, inline, remove, or combine operations as long as the program's observable behavior is unchanged. Observable behavior includes things such as volatile accesses, I/O, and program termination behavior. Undefined behavior gives the compiler even more freedom because there is no required behavior to preserve.

**Example:**

```cpp
int square(int x) {
    return x * x;
}

int value = square(4); // compiler may replace this with 16
```

The generated machine code does not need to look like the source code if the observable result is the same.

**Interview point:** The as-if rule explains why compilers can optimize aggressively while still conforming to the language standard.

**Common mistake:** Expecting source-code statement order to always match machine-code execution order when there are no observable differences.

---

## 26. What is the difference between an expression and a statement?

**Short answer:** An expression produces a value or has a type and value category; a statement performs an action and controls execution flow.

**Detailed answer:**
Expressions include literals, variable names, function calls, arithmetic operations, assignments, and casts. Many expressions produce values, although some produce `void`. Statements include expression statements, declarations, `if`, `for`, `while`, `return`, and compound blocks.

Understanding the distinction helps with topics such as sequencing, side effects, `decltype`, value categories, and control flow.

**Example:**

```cpp
int x = 1 + 2;      // declaration statement containing an expression
x = x + 1;          // expression statement
if (x > 0) {        // if statement using a condition expression
    ++x;            // expression statement
}
```

`x + 1` is an expression. `if (...) { ... }` is a statement.

**Interview point:** Expressions are evaluated; statements are executed. Many statements contain expressions, but they are not the same concept.

**Common mistake:** Saying every line of C++ code is an expression. Declarations and control-flow constructs are statements.

---

## 27. What is operator precedence, and why should you not rely on it too heavily?

**Short answer:** Operator precedence defines how expressions are grouped when parentheses are absent, but clear code should use parentheses when grouping may surprise readers.

**Detailed answer:**
C and C++ have many precedence levels. For example, multiplication binds tighter than addition, and assignment binds looser than comparison. Precedence affects parsing, while evaluation order is a separate topic.

**Example:**

```cpp
int a = 2 + 3 * 4;      // 14, parsed as 2 + (3 * 4)
bool b = x & mask == 0; // surprising: parsed as x & (mask == 0)
```

The second example is a common bug because equality has higher precedence than bitwise AND.

**Clearer version:**

```cpp
bool b = (x & mask) == 0;
```

**Interview point:** Precedence controls grouping, not the runtime order in which operands are evaluated.

**Common mistake:** Confusing operator precedence with evaluation order or sequencing.

---

## 28. What is evaluation order, and how is it different from precedence?

**Short answer:** Precedence determines how expressions are grouped; evaluation order determines when subexpressions are evaluated at runtime.

**Detailed answer:**
C++ does not evaluate every expression strictly left to right. Some operators impose sequencing rules, but many function-call arguments and subexpressions have unspecified evaluation order. This matters when expressions have side effects.

**Example:**

```cpp
int i = 0;
foo(i++, i++); // argument evaluation order is not something to rely on
```

The two increments are both part of the full expression, but the language does not give portable left-to-right meaning for the argument order.

**Safer version:**

```cpp
int first = i++;
int second = i++;
foo(first, second);
```

**Interview point:** Parentheses can change grouping, but they usually do not force evaluation order between independent subexpressions.

**Common mistake:** Adding parentheses and assuming that makes side effects happen earlier.

---

## 29. What is sequence before in C++?

**Short answer:** `sequenced before` is the rule that says one evaluation must complete before another evaluation begins in the same thread.

**Detailed answer:**
Modern C++ describes evaluation ordering using sequencing. If evaluation A is sequenced before evaluation B, then A's value computation and side effects happen before B. If two side effects on the same scalar object are unsequenced, or a side effect is unsequenced relative to a value read of the same object, the program has undefined behavior.

**Example:**

```cpp
int i = 0;
int x = ++i + i; // well-defined since ++i is sequenced enough to produce its value before addition uses it
```

A more dangerous pattern is modifying the same value multiple times without sequencing.

```cpp
int i = 0;
int y = i++ + i++; // do not write code that relies on this kind of expression
```

Even when modern standards define more cases than older C++, such expressions are poor interview and production code because intent is unclear.

**Interview point:** Sequencing rules exist to reason about side effects, but clear code avoids multiple side effects on the same object in one expression.

**Common mistake:** Explaining these bugs only as "operator precedence problems" when the real issue is sequencing and side effects.

---

## 30. What is the difference between unspecified behavior and undefined behavior?

**Short answer:** Undefined behavior has no requirements at all; unspecified behavior means the implementation may choose among several valid possibilities without documenting which one.

**Detailed answer:**
Undefined behavior allows anything: the program may crash, appear to work, be optimized unexpectedly, or corrupt data. Unspecified behavior is less severe: the standard allows multiple outcomes, and each outcome is valid.

**Example of unspecified behavior:**

```cpp
int a = f() + g(); // the order of calling f and g may be unspecified
```

If both functions are independent, either order is valid.

**Example of undefined behavior:**

```cpp
int* p = nullptr;
*p = 42; // undefined behavior
```

**Interview point:** Unspecified behavior is not automatically a bug, but portable code should not depend on which valid choice the implementation makes.

**Common mistake:** Calling every portability issue undefined behavior. C++ distinguishes undefined, unspecified, and implementation-defined behavior.

---

## 31. What is implementation-defined behavior?

**Short answer:** Implementation-defined behavior is behavior where the C++ standard allows choices, but the compiler or platform must document which choice it makes.

**Detailed answer:**
Implementation-defined behavior exists because hardware and operating systems differ. The behavior is not portable in the abstract language, but it is not undefined. Programmers can rely on it only when they intentionally target that implementation and understand the documentation.

**Examples:**
- The size of many fundamental types, such as `int`.
- Whether `char` is signed or unsigned.
- The result of right-shifting a negative signed integer.

**Example:**

```cpp
#include <climits>

static_assert(CHAR_BIT == 8); // common, but not guaranteed by pure C++ for every target
```

**Interview point:** Implementation-defined behavior is acceptable in low-level or platform-specific code when documented and isolated.

**Common mistake:** Treating implementation-defined behavior as undefined behavior or assuming it is portable across all compilers and targets.

---

## 32. What is a full expression?

**Short answer:** A full expression is an expression whose evaluation is complete before the next full expression begins.

**Detailed answer:**
Full expressions matter for temporary object lifetime and sequencing. In many cases, temporaries created during a full expression are destroyed at the end of that full expression.

**Example:**

```cpp
std::string makeName();

const char* p = makeName().c_str();
// p dangles after this full expression ends
```

The temporary `std::string` returned by `makeName()` is destroyed at the end of the initialization statement, leaving `p` dangling.

**Safer version:**

```cpp
std::string name = makeName();
const char* p = name.c_str();
```

**Interview point:** Temporary lifetime is usually tied to full expressions, with some important lifetime-extension exceptions.

**Common mistake:** Storing pointers or views into temporary objects that disappear at the end of the statement.

---

## 33. What is temporary lifetime extension?

**Short answer:** Temporary lifetime extension lets some temporaries bound to references live longer than the full expression, usually as long as the reference.

**Detailed answer:**
When a temporary is bound directly to a `const` lvalue reference or an rvalue reference in certain contexts, its lifetime can be extended. This rule is useful, but it has limits and does not apply through every chain of function calls or returned references.

**Example:**

```cpp
const std::string& name = std::string("Ada");
// temporary string lives as long as name
```

**Non-extension example:**

```cpp
std::string_view view = std::string("Ada");
// dangling view: string_view is not an owning reference that extends lifetime
```

**Interview point:** Lifetime extension applies to the temporary object bound to certain references, not to arbitrary non-owning views or pointers.

**Common mistake:** Assuming `std::string_view`, raw pointers, or references returned from helper functions extend temporary lifetime.

---

## 34. What is aggregate initialization?

**Short answer:** Aggregate initialization initializes aggregate types directly from a braced initializer list.

**Detailed answer:**
An aggregate is an array or a class type with rules that allow direct initialization of its elements or public members. Aggregates are common for simple data carriers. The exact aggregate rules have changed across C++ standards, but the idea is simple member-wise initialization without custom constructor logic.

**Example:**

```cpp
struct Point {
    int x;
    int y;
};

Point p{1, 2};
```

Designated initializers are available in C++20 for aggregates.

```cpp
Point q{.x = 1, .y = 2};
```

**Interview point:** Aggregates are useful for simple transparent data structures, but classes with invariants often need constructors to validate state.

**Common mistake:** Adding a constructor to a simple data type and accidentally changing how aggregate initialization works for users.

---

## 35. What is list initialization, and why does it matter?

**Short answer:** List initialization uses braces and helps prevent narrowing conversions in many contexts.

**Detailed answer:**
Brace initialization can initialize objects, aggregates, containers, and function arguments. One important benefit is that it rejects many narrowing conversions that would otherwise silently lose information.

**Example:**

```cpp
int a = 3.14;  // allowed, value becomes 3
int b{3.14};   // error: narrowing conversion
```

For classes, braces can also prefer `std::initializer_list` constructors when available, which can surprise programmers.

```cpp
std::vector<int> a(3, 1); // three elements: 1, 1, 1
std::vector<int> b{3, 1}; // two elements: 3, 1
```

**Interview point:** Braces improve safety against narrowing but interact with overload resolution in ways candidates should know.

**Common mistake:** Assuming parentheses and braces always choose equivalent constructors.

---

## 36. What is the difference between direct initialization and copy initialization?

**Short answer:** Direct initialization uses constructor-call syntax or braces directly; copy initialization uses `=` syntax and may consider conversions differently.

**Detailed answer:**
Despite the name, copy initialization does not necessarily copy an object. It describes a form of initialization. Direct initialization often allows explicit constructors, while copy initialization does not use explicit constructors for implicit conversion.

**Example:**

```cpp
class Port {
public:
    explicit Port(int value) : value_(value) {}
private:
    int value_;
};

Port a(8080);  // direct initialization: OK
Port b{8080};  // direct-list initialization: OK
// Port c = 8080; // error: explicit constructor not used for copy initialization
```

**Interview point:** Initialization syntax affects overload resolution and whether `explicit` constructors participate.

**Common mistake:** Believing `T x = value;` always means a copy constructor is called.

---

## 37. What is zero initialization?

**Short answer:** Zero initialization initializes an object to the zero value for its type before other initialization steps in certain contexts.

**Detailed answer:**
Objects with static or thread storage duration are zero-initialized before other initialization. Value initialization of many scalar types also results in zero initialization. For class types, initialization rules may first zero-initialize storage and then run constructors depending on the form and type.

**Example:**

```cpp
int globalValue; // zero-initialized before program starts

void f() {
    int local;   // indeterminate value
    int value{}; // zero-initialized to 0
}
```

**Interview point:** Initialization depends on storage duration and syntax. Local automatic scalar variables are not automatically zero-initialized.

**Common mistake:** Assuming every uninitialized `int` starts as zero because globals do.

---

## 38. What is default initialization?

**Short answer:** Default initialization initializes an object when no initializer is provided, but for automatic scalar variables it leaves the value indeterminate.

**Detailed answer:**
Default initialization behaves differently for class types and fundamental types. For class types, a default constructor is called. For automatic scalar variables, no initialization happens, so reading the value before writing it is undefined behavior.

**Example:**

```cpp
struct Counter {
    Counter() : value(0) {}
    int value;
};

void f() {
    Counter c; // default constructor sets value to 0
    int x;     // indeterminate value
}
```

**Interview point:** Default initialization is not the same as zero initialization.

**Common mistake:** Reading an automatic local variable before assigning it a value.

---

## 39. What is value initialization?

**Short answer:** Value initialization uses forms such as `T{}` or `T()` and often produces a zero-like initialized value for fundamental types.

**Detailed answer:**
Value initialization is commonly used to request a clean default value. For scalar types, `T{}` gives zero. For class types, constructors are involved according to the type's rules.

**Example:**

```cpp
int x{};              // 0
std::string name{};   // empty string
std::vector<int> v{}; // empty vector
```

Value initialization is common in generic code because it can create a default value without spelling a specific literal.

```cpp
template <typename T>
T makeDefault() {
    return T{};
}
```

**Interview point:** `T{}` is often the safest way to request a default value, especially for fundamental types.

**Common mistake:** Thinking `T{}` and an uninitialized local `T variable;` always have the same behavior.

---

## 40. What is narrowing conversion?

**Short answer:** A narrowing conversion is a conversion that may lose information, such as converting a `double` to an `int` or a large integer to a smaller type.

**Detailed answer:**
C++ allows many implicit conversions for compatibility, but some can lose precision, range, or sign information. List initialization rejects many narrowing conversions, making accidental data loss easier to catch.

**Examples:**

```cpp
int a = 3.14;      // allowed, loses fractional part
// int b{3.14};    // error: narrowing

unsigned char c = 300; // value may wrap depending on conversion rules
```

Narrowing is especially important in APIs, binary protocols, arithmetic code, and security-sensitive length calculations.

**Interview point:** Avoid silent narrowing at API boundaries. Use explicit checks or casts when the conversion is intentional and safe.

**Common mistake:** Treating compiler warnings about narrowing as harmless noise instead of potential truncation or sign bugs.

---

## 41. What is an implicit conversion sequence?

**Short answer:** An implicit conversion sequence is the set of conversions the compiler may apply to make an expression match a required type.

**Detailed answer:**
C++ can convert values automatically in assignments, function calls, arithmetic expressions, and overload resolution. These conversions may include standard conversions, user-defined conversions, and qualification conversions. The ranking of conversion sequences affects which overloaded function is selected.

**Example:**

```cpp
void f(long);
void f(double);

int x = 10;
f(x); // prefers conversion from int to long over int to double on many implementations
```

Implicit conversions are convenient, but they can hide narrowing, signedness changes, or unexpected overload selection.

**Interview point:** Overload resolution is often about comparing conversion quality, not just matching function names.

**Common mistake:** Assuming the compiler always chooses the overload that humans consider most obvious.

---

## 42. What is an explicit conversion operator?

**Short answer:** An explicit conversion operator defines a user-defined conversion that is not used for most implicit conversions.

**Detailed answer:**
Classes can define conversion operators to convert objects to other types. Marking the operator `explicit` prevents accidental conversions in many contexts while still allowing direct casts or contextual conversions such as boolean checks.

**Example:**

```cpp
class FileHandle {
public:
    explicit operator bool() const {
        return fd_ >= 0;
    }

private:
    int fd_ = -1;
};

FileHandle file;
if (file) {
    // handle is valid
}
```

This pattern is common for resource handles and smart-pointer-like types.

**Interview point:** `explicit` helps prevent surprising conversions while keeping intentional checks readable.

**Common mistake:** Providing broad implicit conversions that make unrelated overloads viable.

---

## 43. What is type promotion in arithmetic expressions?

**Short answer:** Type promotion converts smaller arithmetic types to larger or more convenient types before many arithmetic operations.

**Detailed answer:**
Integer promotions convert types such as `char`, `short`, and many enum values to `int` or `unsigned int` before arithmetic. Then usual arithmetic conversions choose a common type for binary operators. This can affect overflow, signedness, and result type.

**Example:**

```cpp
unsigned char a = 250;
unsigned char b = 10;

auto sum = a + b; // usually int, not unsigned char
```

The result type is not necessarily the same as the operand type.

**Interview point:** Small integer types are often promoted before arithmetic, which can surprise candidates working with bytes or embedded code.

**Common mistake:** Assuming arithmetic on `char` or `unsigned char` stays in the same type.

---

## 44. What is the conditional operator's type selection?

**Short answer:** The conditional operator chooses a common result type from its second and third operands using language conversion rules.

**Detailed answer:**
The expression `condition ? a : b` is not just a runtime choice; it also has a compile-time type. If `a` and `b` have different types, C++ applies rules to find a common type or reject the expression.

**Example:**

```cpp
auto value = flag ? 1 : 2.5; // double
```

For class types, references, and value categories, the rules can be subtle. This matters in generic code where `auto` captures the conditional expression's type.

**Interview point:** The conditional operator participates in type deduction and conversions, so its result type should not be assumed casually.

**Common mistake:** Expecting `auto` to preserve both possible operand types instead of deducing one expression type.

---

## 45. What is an incomplete type?

**Short answer:** An incomplete type is a declared type whose full definition is not yet known.

**Detailed answer:**
A forward-declared class is incomplete until its definition is seen. You can declare pointers or references to incomplete types, but you cannot create objects by value, access members, or use `sizeof` because the compiler does not know the layout.

**Example:**

```cpp
class Widget; // incomplete type

void draw(const Widget& widget); // OK
Widget* makeWidget();            // OK
// Widget value;                 // error: incomplete type
```

Incomplete types are useful for reducing header dependencies and hiding implementation details.

**Interview point:** Incomplete types support decoupling, but value semantics require complete definitions.

**Common mistake:** Forward-declaring a type and then trying to store it by value in a class member.

---

## 46. What is object representation?

**Short answer:** Object representation is the sequence of bytes that stores an object's value in memory.

**Detailed answer:**
Every object has an object representation made of bytes. For trivially copyable types, the bytes can be copied with `std::memcpy` and later copied back to recreate the same value. But not every byte pattern is a valid value for every type, and padding bytes may exist.

**Example:**

```cpp
#include <array>
#include <bit>
#include <cstddef>

float value = 1.0f;
auto bytes = std::bit_cast<std::array<std::byte, sizeof(float)>>(value);
```

Object representation matters in serialization, hashing, binary protocols, and low-level debugging.

**Interview point:** Bytes are not the same thing as portable semantic values.

**Common mistake:** Serializing raw object bytes and expecting the format to be portable across compilers, platforms, and versions.

---

## 47. What is padding in structures?

**Short answer:** Padding is unused space inserted by the compiler to satisfy alignment requirements.

**Detailed answer:**
Compilers may place padding bytes between members or at the end of a struct so each member is properly aligned and arrays of the struct work correctly. Padding affects `sizeof`, binary layout, cache behavior, and serialization concerns.

**Example:**

```cpp
struct Example {
    char tag;
    int value;
};

// sizeof(Example) is often 8, not 5, because of padding before value.
```

Member order can change padding, but layout should be changed carefully when ABI or binary formats matter.

**Interview point:** Structure size is affected by alignment, not just the sum of member sizes.

**Common mistake:** Writing raw structs directly to disk or network and ignoring padding bytes and endianness.

---

## 48. What is a bit-field?

**Short answer:** A bit-field is a class or struct member that uses a specified number of bits.

**Detailed answer:**
Bit-fields can compactly store flags or small integer values. Their exact layout, ordering, alignment, and signedness details can be implementation-defined, so they are not ideal for portable wire formats.

**Example:**

```cpp
struct Flags {
    unsigned ready : 1;
    unsigned error : 1;
    unsigned mode  : 3;
};
```

Bit-fields can be useful for memory-constrained structures or hardware-adjacent code, but portable binary layout requires caution.

**Interview point:** Bit-fields describe storage width, but not a fully portable external representation.

**Common mistake:** Assuming bit-field layout is identical across compilers and CPU architectures.

---

## 49. What is the difference between declaration specifiers and declarators?

**Short answer:** Declaration specifiers describe the base type and properties; declarators describe names and how each name relates to that base type.

**Detailed answer:**
C and C++ declarations can be confusing because `*`, `&`, arrays, and function syntax bind to individual declarators, not always to the shared base type. This is why multiple declarations on one line can mislead readers.

**Example:**

```cpp
int* a, b; // a is int*, b is int
```

The base declaration specifier is `int`, while `*a` is part of the declarator for `a` only.

**Interview point:** Prefer one declaration per line for pointer or reference declarations to avoid ambiguity.

**Common mistake:** Believing `int* a, b;` declares two pointers.

---

## 50. What is the difference between a definition, declaration, and initialization in one statement?

**Short answer:** A single statement can declare a name, define an object, and initialize it at the same time.

**Detailed answer:**
These terms describe different roles. A declaration introduces a name. A definition creates the object or provides the function body. Initialization gives an object its initial value. Many ordinary variable statements do all three.

**Example:**

```cpp
extern int count;      // declaration only
int count = 0;         // definition and initialization
void f();              // function declaration
void f() {}            // function definition
```

Understanding the distinction helps with linkage, headers, multiple-definition errors, and object lifetime.

**Interview point:** The same line of code can serve multiple language roles, so candidates should explain which role is being discussed.

**Common mistake:** Using declaration, definition, and initialization as interchangeable words.
