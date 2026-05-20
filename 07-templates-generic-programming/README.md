# Templates and Generic Programming Interview Questions

Templates are one of the most powerful and frequently misunderstood parts of C++. Interviewers use this topic to test whether candidates understand compile-time polymorphism, type deduction, specialization, constraints, and the tradeoffs of generic code.

## 1. What is a template in C++?

**Short answer:** A template is a blueprint for generating functions or classes based on types or compile-time values.

**Detailed answer:**
Templates allow code to work with many types without rewriting the same logic for each type. The compiler generates concrete versions when the template is instantiated with specific arguments.

**Function template example:**

```cpp
template <typename T>
T maxValue(T a, T b) {
    return a > b ? a : b;
}

int x = maxValue(3, 7);
double y = maxValue(2.5, 1.5);
```

**Class template example:**

```cpp
template <typename T>
class Box {
public:
    explicit Box(T value) : value_(std::move(value)) {}
    const T& value() const { return value_; }

private:
    T value_;
};
```

**Interview point:** Templates provide compile-time polymorphism, unlike virtual functions which provide runtime polymorphism.

---

## 2. What is the difference between `typename` and `class` in template parameters?

**Short answer:** For type template parameters, `typename` and `class` usually mean the same thing.

**Detailed answer:**
Both forms declare a type parameter.

```cpp
template <typename T>
void f(T value) {}

template <class T>
void g(T value) {}
```

They are equivalent in this context. However, `typename` is also used to tell the compiler that a dependent name is a type.

```cpp
template <typename Container>
void printFirstType() {
    typename Container::value_type value{};
}
```

Without `typename`, the compiler may not know whether `Container::value_type` is a type or a static member.

**Common mistake:** Thinking `template <class T>` only accepts class types. It accepts any type, including `int`, pointers, and user-defined types.

---

## 3. What is template instantiation?

**Short answer:** Template instantiation is when the compiler generates concrete code from a template using specific template arguments.

**Detailed answer:**
A template itself is not a normal function or class. It becomes concrete only when used with actual types or values. This is why many template definitions must be visible in header files.

**Example:**

```cpp
template <typename T>
T square(T value) {
    return value * value;
}

int a = square(5);        // instantiates square<int>
double b = square(2.5);   // instantiates square<double>
```

The compiler may generate separate versions for `int` and `double`.

**Interview point:** Template code is checked in two phases: some errors are detected when the template is parsed, while type-dependent errors appear only when instantiated.

---

## 4. Why are template definitions usually placed in header files?

**Short answer:** The compiler usually needs the full template definition at the point where the template is instantiated.

**Detailed answer:**
Unlike normal functions, templates are compiled when instantiated with specific arguments. If only the declaration is visible, the compiler may not be able to generate the required code.

**Header example:**

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}
```

This is commonly placed in a `.hpp` or `.h` file.

**Alternative:** Explicit instantiation can be used when you know the exact types needed.

```cpp
template int add<int>(int, int);
```

**Common mistake:** Putting a template definition only in a `.cpp` file and then getting unresolved external linker errors from other translation units.

---

## 5. What is template specialization?

**Short answer:** Template specialization provides a custom implementation for a specific template argument or set of arguments.

**Detailed answer:**
Specialization is useful when the general template does not work well or needs optimized behavior for a particular type.

**Example:**

```cpp
template <typename T>
struct TypeName {
    static const char* value() { return "unknown"; }
};

template <>
struct TypeName<int> {
    static const char* value() { return "int"; }
};
```

`TypeName<double>::value()` returns `"unknown"`, while `TypeName<int>::value()` returns `"int"`.

**Interview point:** Function templates cannot be partially specialized, but class templates can be partially specialized.

**Common mistake:** Overusing specialization when overloads or `if constexpr` would be simpler.

---

## 6. What is partial specialization?

**Short answer:** Partial specialization customizes a class template for a pattern of template arguments rather than one exact type.

**Detailed answer:**
Partial specialization applies to class templates and variable templates, not function templates. It is often used in type traits and generic library implementation.

**Example:**

```cpp
template <typename T>
struct IsPointer {
    static constexpr bool value = false;
};

template <typename T>
struct IsPointer<T*> {
    static constexpr bool value = true;
};
```

`IsPointer<int>::value` is `false`, while `IsPointer<int*>::value` is `true`.

**Modern C++ note:** The standard library provides traits such as `std::is_pointer`, `std::is_same`, and `std::remove_reference`.

**Common mistake:** Trying to partially specialize a function template directly.

---

## 7. What is SFINAE?

**Short answer:** SFINAE means Substitution Failure Is Not An Error. If template substitution fails in certain contexts, that overload is removed instead of causing a hard error.

**Detailed answer:**
SFINAE allows templates to participate in overload resolution only when certain expressions or types are valid. Before C++20 concepts, it was a common way to constrain templates.

**Example:**

```cpp
#include <type_traits>

template <typename T>
std::enable_if_t<std::is_integral_v<T>, bool>
isEven(T value) {
    return value % 2 == 0;
}
```

This function only participates in overload resolution for integral types.

**Interview point:** SFINAE is powerful but can make error messages hard to read. C++20 concepts often express constraints more clearly.

**Common mistake:** Thinking SFINAE catches all template errors. It only applies to substitution failures in specific contexts.

---

## 8. What are C++20 concepts?

**Short answer:** Concepts are named compile-time constraints for template parameters.

**Detailed answer:**
Concepts make generic code easier to read and produce clearer compiler diagnostics. They let you say what operations or properties a type must support.

**Example:**

```cpp
#include <concepts>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

template <Number T>
T add(T a, T b) {
    return a + b;
}
```

`add` now only accepts integral or floating-point types.

**Why it matters:** Concepts document template requirements directly in code and reduce accidental misuse.

**Common mistake:** Treating concepts as runtime checks. Concepts are checked at compile time.

---

## 9. What is `if constexpr`?

**Short answer:** `if constexpr` is a compile-time conditional that discards inactive branches during compilation.

**Detailed answer:**
`if constexpr` allows one template to choose different code paths based on compile-time conditions. The discarded branch does not need to be valid for the instantiated type.

**Example:**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void printValue(const T& value) {
    if constexpr (std::is_pointer_v<T>) {
        if (value) {
            std::cout << *value << '\n';
        }
    } else {
        std::cout << value << '\n';
    }
}
```

For pointer types, it dereferences. For non-pointer types, it prints directly.

**Interview point:** `if constexpr` is often simpler than tag dispatch or complex SFINAE for local branching.

---

## 10. What are the tradeoffs of templates?

**Short answer:** Templates improve type safety and performance but can increase compile time, binary size, and error-message complexity.

**Detailed answer:**
Templates enable zero-overhead abstractions because decisions can be made at compile time and functions can be inlined for specific types. However, each instantiation may generate more code, which can increase binary size. Template-heavy code can also slow compilation and produce long diagnostics.

**Benefits:**
- Type-safe generic code
- Compile-time polymorphism
- Potentially better optimization
- No virtual dispatch overhead

**Costs:**
- Longer compile times
- Larger binaries from many instantiations
- More complex error messages
- Implementation often exposed in headers

**Interview point:** Templates are excellent when type-specific optimization matters, but virtual interfaces may be better when ABI stability, separate compilation, or runtime extensibility is more important.

---

## 11. What are non-type template parameters?

**Short answer:** Non-type template parameters are compile-time values passed to templates instead of types.

**Detailed answer:**
Templates can accept values such as integers, pointers, references, enum values, and since modern C++, more structural constant values. They are useful when a value must affect the type or enable compile-time optimization.

**Example:**

```cpp
#include <array>

template <typename T, std::size_t N>
class FixedBuffer {
public:
    std::size_t size() const {
        return N;
    }

private:
    std::array<T, N> data_{};
};

FixedBuffer<int, 16> buffer;
```

Here, `N` is part of the type. `FixedBuffer<int, 16>` and `FixedBuffer<int, 32>` are different types.

**Interview point:** Non-type template parameters are common in `std::array<T, N>`, fixed-size buffers, compile-time configuration, and embedded code.

**Common mistake:** Assuming template parameters must always be types.

---

## 12. What are variadic templates?

**Short answer:** Variadic templates accept a variable number of template arguments.

**Detailed answer:**
Variadic templates use parameter packs. They are useful for forwarding constructors, tuple-like types, formatting utilities, and generic wrappers that accept any number of arguments.

**Example:**

```cpp
#include <iostream>

template <typename... Args>
void printAll(const Args&... args) {
    ((std::cout << args << ' '), ...);
    std::cout << '\n';
}

printAll(1, "hello", 3.5);
```

The expression `((std::cout << args << ' '), ...)` is a fold expression that expands over all arguments.

**Interview point:** Variadic templates replaced many older techniques that required writing multiple overloads for different argument counts.

**Common mistake:** Forgetting that a parameter pack must be expanded in a valid expansion context.

---

## 13. What is CRTP?

**Short answer:** CRTP means Curiously Recurring Template Pattern, where a base class template takes the derived class as a template parameter.

**Detailed answer:**
CRTP provides compile-time polymorphism. The base class can call functions on the derived class without using virtual functions. It is common in mixins, static interfaces, and performance-sensitive libraries.

**Example:**

```cpp
template <typename Derived>
class Printable {
public:
    void print() const {
        static_cast<const Derived&>(*this).printImpl();
    }
};

class User : public Printable<User> {
public:
    void printImpl() const {
        // print user
    }
};
```

Calling `User{}.print()` resolves at compile time rather than through virtual dispatch.

**Tradeoff:** CRTP can be efficient, but it creates tighter compile-time coupling and can make error messages harder to understand.

**Common mistake:** Using CRTP when ordinary virtual functions or composition would be simpler and clearer.

---

## 14. What are type traits?

**Short answer:** Type traits are compile-time tools for querying or transforming types.

**Detailed answer:**
The standard library provides type traits in `<type_traits>`. They help generic code make decisions based on type properties, such as whether a type is integral, copy constructible, a pointer, or the same as another type.

**Example:**

```cpp
#include <type_traits>

template <typename T>
void process(T value) {
    if constexpr (std::is_integral_v<T>) {
        // integral-specific logic
    } else {
        // general logic
    }
}
```

Type transformation traits can produce related types.

```cpp
using Raw = std::remove_reference_t<int&>; // int
```

**Interview point:** Type traits are building blocks for concepts, SFINAE, `if constexpr`, and generic library implementation.

**Common mistake:** Reimplementing standard traits instead of using well-tested library traits.

---

## 15. How does template argument deduction work?

**Short answer:** Template argument deduction infers template parameters from function call arguments.

**Detailed answer:**
When calling a function template, the compiler compares parameter types with argument types to infer template arguments. Deduction follows specific rules for references, arrays, functions, cv-qualifiers, and forwarding references.

**Example:**

```cpp
template <typename T>
void byValue(T value) {}

template <typename T>
void byReference(const T& value) {}

int x = 10;
const int cx = 20;

byValue(cx);     // T is int; top-level const is dropped
byReference(cx); // T is int; parameter is const int&
```

Array arguments also behave differently depending on the parameter form.

```cpp
template <typename T>
void f(T value) {}

template <typename T, std::size_t N>
void g(T (&value)[N]) {}

int values[3]{};
f(values); // T is int*
g(values); // T is int, N is 3
```

**Interview point:** Understanding deduction prevents surprises with `auto`, forwarding references, overload resolution, and generic APIs.

**Common mistake:** Assuming template deduction always preserves `const`, references, and array sizes. It depends on the parameter type.

---

## 16. What is two-phase name lookup in templates?

**Short answer:** Two-phase name lookup means some names in templates are checked when the template is defined, while dependent names are checked when the template is instantiated.

**Detailed answer:**
Non-dependent names do not depend on template parameters, so they are resolved during template definition. Dependent names depend on template parameters, so their meaning may not be known until a specific instantiation.

**Example:**

```cpp
template <typename T>
void call(T value) {
    helper();      // non-dependent name: looked up at template definition
    value.process(); // dependent expression: checked during instantiation
}
```

This rule helps catch some errors early while still allowing templates to work with many possible types.

**Interview point:** Two-phase lookup explains why template errors sometimes appear only when a specific type is used.

**Common mistake:** Assuming all template code is fully checked only after instantiation.

---

## 17. What is a dependent name, and why might you need `typename` or `template`?

**Short answer:** A dependent name depends on a template parameter, so the compiler may need help knowing whether it names a type or a template.

**Detailed answer:**
When a nested name depends on a template parameter, the compiler cannot always know what kind of entity it names. Use `typename` when the dependent name is a type. Use the `template` keyword when calling a dependent member template.

**Example:**

```cpp
template <typename Container>
void useContainer(const Container& container) {
    typename Container::value_type value{};
}
```

For dependent member templates:

```cpp
template <typename T>
void callFactory(T& object) {
    object.template create<int>();
}
```

**Interview point:** These keywords are not decorative. They disambiguate parsing in generic code.

**Common mistake:** Removing `typename` from dependent types because the code "looks obvious" to a human reader.

---

## 18. What is tag dispatch?

**Short answer:** Tag dispatch selects an implementation by passing a small compile-time tag type to an overloaded helper function.

**Detailed answer:**
Before `if constexpr` and concepts, tag dispatch was a common way to choose different implementations based on type traits. It is still useful when separating overloads produces cleaner code.

**Example:**

```cpp
#include <iterator>

template <typename Iterator>
void advanceImpl(Iterator& it, int n, std::random_access_iterator_tag) {
    it += n;
}

template <typename Iterator>
void advanceImpl(Iterator& it, int n, std::input_iterator_tag) {
    while (n-- > 0) {
        ++it;
    }
}

template <typename Iterator>
void advance(Iterator& it, int n) {
    using Category = typename std::iterator_traits<Iterator>::iterator_category;
    advanceImpl(it, n, Category{});
}
```

**Interview point:** Tag dispatch moves compile-time decisions into overload resolution and keeps invalid code out of unsupported overloads.

**Common mistake:** Using tag dispatch for simple cases where `if constexpr` would be clearer.

---

## 19. What are variable templates?

**Short answer:** Variable templates define compile-time variable families parameterized by template arguments.

**Detailed answer:**
Variable templates are useful for constants and type-trait helpers. The standard library uses `_v` helpers such as `std::is_integral_v<T>` to expose trait values more conveniently.

**Example:**

```cpp
template <typename T>
constexpr bool IsPointerV = false;

template <typename T>
constexpr bool IsPointerV<T*> = true;

static_assert(IsPointerV<int*>);
```

This is similar in spirit to `std::is_pointer_v<T>`.

**Interview point:** Variable templates reduce noisy `::value` syntax and are common in modern generic code.

**Common mistake:** Confusing variable templates with ordinary global variables. They are instantiated at compile time for template arguments.

---

## 20. What is explicit template instantiation?

**Short answer:** Explicit template instantiation tells the compiler to generate a template specialization for specific arguments in a specific translation unit.

**Detailed answer:**
Explicit instantiation can reduce compile times and keep template implementation details out of widely included headers when the supported types are known. The template definition must be visible at the explicit instantiation point.

**Example:**

```cpp
// math.hpp
template <typename T>
T add(T a, T b);

// math.cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

template int add<int>(int, int);
template double add<double>(double, double);
```

Other translation units can call `add<int>` and `add<double>` through the declaration.

**Tradeoff:** This works best when the set of template arguments is small and controlled. Fully open generic libraries usually keep definitions in headers.

**Common mistake:** Moving a template definition to a `.cpp` file without explicit instantiations for every type used elsewhere.

---

## 21. What is template metaprogramming?

**Short answer:** Template metaprogramming uses templates to compute types, values, or decisions at compile time.

**Detailed answer:**
Before modern C++, template metaprogramming often used recursive templates and specializations. Modern C++ usually prefers clearer tools such as `constexpr`, `if constexpr`, concepts, and standard type traits, but templates are still central to compile-time programming.

Template metaprogramming can improve type safety and performance, but it can also increase compile times and produce difficult error messages.

**Example:**

```cpp
#include <type_traits>

template <typename T>
void printKind(const T& value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integer\n";
    } else {
        std::cout << "other\n";
    }
}
```

**Interview point:** Modern C++ metaprogramming should favor readable compile-time constructs over clever recursive template tricks when possible.

**Common mistake:** Using heavy template metaprogramming for problems that could be solved more simply with ordinary runtime code.

---

## 22. What is a fold expression?

**Short answer:** A fold expression reduces a parameter pack using an operator.

**Detailed answer:**
Fold expressions were introduced in C++17 to simplify variadic template code. They replace many recursive variadic patterns with a direct expression over all pack elements.

**Example:**

```cpp
#include <iostream>

template <typename... Args>
void printAll(const Args&... args) {
    ((std::cout << args << ' '), ...);
}
```

This prints each argument by folding over the comma operator.

**Sum example:**

```cpp
template <typename... Args>
auto sum(Args... args) {
    return (args + ...);
}
```

**Interview point:** Fold expressions are the modern default for many simple variadic-template operations.

**Common mistake:** Writing complex recursive variadic templates when a fold expression is clearer and shorter.

---

## 23. What is a template template parameter?

**Short answer:** A template template parameter is a template parameter that accepts another template as an argument.

**Detailed answer:**
Template template parameters are useful when generic code needs to abstract over a container or wrapper template, not just over a concrete type.

**Example:**

```cpp
#include <deque>
#include <vector>

template <template <typename, typename> class Container, typename T>
class StackLike {
public:
    void push(const T& value) {
        values_.push_back(value);
    }

private:
    Container<T, std::allocator<T>> values_;
};

StackLike<std::vector, int> vectorStack;
StackLike<std::deque, int> dequeStack;
```

The parameter `Container` is itself a class template.

**Interview point:** Template template parameters are powerful but often less flexible than accepting a fully formed container type.

**Common mistake:** Overusing template template parameters when `template <typename Container>` would be simpler and support more container variations.

---

## 24. What is a deduction guide?

**Short answer:** A deduction guide tells the compiler how to deduce class template arguments from constructor arguments.

**Detailed answer:**
Class template argument deduction lets code omit explicit template arguments when constructing class template objects. Deduction guides customize or clarify that deduction when constructors alone are not enough.

**Example:**

```cpp
#include <iterator>
#include <vector>

template <typename T>
class Box {
public:
    explicit Box(T value) : value_(value) {}

private:
    T value_;
};

Box(const char*) -> Box<std::string>;

Box a(42);        // Box<int>
Box b("hello");   // Box<std::string>
```

**Interview point:** Deduction guides affect how class template argument deduction behaves; they are part of a template's user-facing API.

**Common mistake:** Assuming class template argument deduction always chooses the semantic type you wanted instead of the exact constructor parameter type.

---

## 25. How do concepts improve template error messages and constraints?

**Short answer:** Concepts let templates state requirements directly, producing clearer constraints and often better diagnostics.

**Detailed answer:**
Before concepts, templates commonly used SFINAE or type-trait tricks to reject unsupported types. Concepts provide a named, readable way to express what operations or properties a type must support.

**Example:**

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Printable = requires(std::ostream& os, const T& value) {
    os << value;
};

template <Printable T>
void print(const T& value) {
    std::cout << value << '\n';
}
```

If a type cannot be printed, the constraint failure points to the missing requirement more clearly than a deep template-instantiation error.

**Interview point:** Concepts are not just documentation; they participate in overload resolution and make generic APIs safer and clearer.

**Common mistake:** Writing overly broad unconstrained templates and letting errors appear far inside the implementation.

---

## 26. What is overload resolution with templates?

**Short answer:** Overload resolution chooses the best viable function from ordinary functions and function templates after template argument deduction and constraint checking.

**Detailed answer:**
When a call matches multiple functions, C++ checks which candidates are viable, how good their conversions are, whether templates can be deduced, and whether constraints are satisfied. A non-template function is not always chosen automatically; the best match still matters.

**Example:**

```cpp
void print(int);

template <typename T>
void print(T);

print(1);    // calls print(int)
print(1.5);  // calls template with T = double
```

With concepts, a constrained overload may be preferred when it is more specialized.

**Interview point:** Templates participate in overload resolution, so adding a generic overload can change which function existing calls select.

**Common mistake:** Adding an unconstrained `template <typename T>` overload and accidentally capturing calls meant for specific overloads.

---

## 27. What is partial ordering of function templates?

**Short answer:** Partial ordering decides which function template is more specialized when multiple templates match a call.

**Detailed answer:**
If two function templates are viable, C++ tries to determine whether one is more specialized. This lets a pointer-specific template beat a fully generic template, for example. Concepts and constraints can also affect which overload is considered more constrained.

**Example:**

```cpp
template <typename T>
void f(T);      // generic

template <typename T>
void f(T*);     // more specialized for pointers

int* p = nullptr;
f(p); // calls f(T*)
```

**Interview point:** Template overload sets should be designed so the most specific intended overload wins clearly.

**Common mistake:** Creating several templates that all match equally well and produce ambiguous calls.

---

## 28. What is a requires-expression?

**Short answer:** A requires-expression checks whether expressions, types, or nested requirements are valid for template parameters.

**Detailed answer:**
Requires-expressions are the building blocks of concepts. They can test whether a type supports an operation, has a nested type, or satisfies a semantic requirement expressible as a constraint.

**Example:**

```cpp
template <typename T>
concept HasSize = requires(const T& value) {
    { value.size() } -> std::convertible_to<std::size_t>;
};
```

This concept requires that `value.size()` is a valid expression and that its result can be converted to `std::size_t`.

**Interview point:** Requires-expressions test syntax and type relationships; they do not prove runtime semantics by themselves.

**Common mistake:** Assuming a concept named `SortedRange` can automatically prove a range is sorted at runtime.

---

## 29. What is a constrained auto parameter?

**Short answer:** A constrained `auto` parameter uses a concept directly in a function parameter list to create a concise constrained template.

**Detailed answer:**
C++20 allows abbreviated function templates. This is useful when a full template parameter list would add noise. The function is still a template.

**Example:**

```cpp
#include <concepts>
#include <iostream>

void printIntegral(std::integral auto value) {
    std::cout << value << '\n';
}
```

This accepts integral types and rejects non-integral types.

**Interview point:** Abbreviated templates improve readability for simple generic functions, but full template syntax is clearer when relationships between parameters matter.

**Common mistake:** Using separate `auto` parameters when both arguments must have the same type.

---

## 30. What is the difference between `auto` and template type deduction?

**Short answer:** `auto` deduction and template type deduction are closely related, but they differ in some contexts such as braced initializers and function return deduction.

**Detailed answer:**
Both usually drop top-level `const` and references unless the declared form preserves them. However, `auto` has special behavior with braced initializer lists in some declarations, and function template deduction does not deduce from every context.

**Example:**

```cpp
const int x = 1;
auto a = x;        // int
const auto& b = x; // const int&

template <typename T>
void f(T value);

f(x); // T = int
```

**Interview point:** Understand the declared pattern, not just the initializer. `auto&`, `const auto&`, and `auto&&` deduce differently.

**Common mistake:** Assuming `auto` always preserves references and `const` exactly.

---

## 31. What is a non-deduced context?

**Short answer:** A non-deduced context is a part of a template type where template arguments are not inferred from the function call.

**Detailed answer:**
C++ has rules for where template argument deduction can and cannot infer types. Non-deduced contexts are useful when you want callers to provide a template argument explicitly or when a type should depend on another deduced parameter.

**Example:**

```cpp
template <typename T>
void convert(typename std::type_identity<T>::type value);

// convert(42);      // T cannot be deduced this way
convert<int>(42);    // OK
```

**Interview point:** Non-deduced contexts explain many surprising template deduction failures.

**Common mistake:** Expecting deduction to work through every type alias, nested type, or dependent expression.

---

## 32. What is template argument deduction failure?

**Short answer:** Template argument deduction failure means the compiler could not infer template arguments for a candidate function template.

**Detailed answer:**
Deduction can fail because types do not match, a parameter appears in a non-deduced context, constraints are not satisfied, or a braced initializer has no deducible type. Some failures remove a candidate by SFINAE; others become hard errors depending on where they occur.

**Example:**

```cpp
template <typename T>
void f(T value, T other);

f(1, 2);    // OK, T = int
// f(1, 2.0); // deduction conflict: int vs double
```

**Interview point:** Explicit template arguments or overloads can resolve some deduction failures, but constraints often make intent clearer.

**Common mistake:** Assuming the compiler will pick a common type for every template parameter conflict.

---

## 33. What is template instantiation depth?

**Short answer:** Template instantiation depth is how deeply templates recursively instantiate other templates during compilation.

**Detailed answer:**
Recursive metaprogramming can create long instantiation chains. Compilers limit template depth to prevent runaway compilation. Excessive depth also produces unreadable errors and slow builds.

**Example idea:**

```cpp
template <int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template <>
struct Factorial<0> {
    static constexpr int value = 1;
};
```

Modern C++ often replaces such patterns with `constexpr` functions.

**Interview point:** Compile-time computation has a cost. Use the simplest compile-time tool that solves the problem.

**Common mistake:** Building deeply recursive type machinery when `constexpr` or `if constexpr` would be clearer.

---

## 34. What are compile-time strings or fixed strings in templates?

**Short answer:** Fixed strings are compile-time string-like values used as non-type template parameters in modern C++ patterns.

**Detailed answer:**
C++20 expanded non-type template parameters, enabling more value-like template arguments. Libraries sometimes use fixed-string wrappers to parameterize types or functions by names known at compile time.

**Conceptual example:**

```cpp
template <std::size_t N>
struct FixedString {
    char data[N];
};

template <FixedString Name>
struct Field {};
```

Exact support and syntax depend on structural type rules.

**Interview point:** Value-based templates can express compile-time metadata, but they can also increase compile-time cost and API complexity.

**Common mistake:** Using type-level strings when ordinary runtime strings or enums would be simpler.

---

## 35. What is policy-based design?

**Short answer:** Policy-based design customizes behavior by passing policy types as template parameters.

**Detailed answer:**
A class template can delegate parts of behavior to policy classes. This enables compile-time customization without virtual dispatch. It is useful for allocators, locking strategies, error handling, comparison, and storage policies.

**Example:**

```cpp
struct NoLock {
    void lock() {}
    void unlock() {}
};

template <typename LockPolicy>
class Cache : private LockPolicy {
public:
    void get() {
        this->lock();
        this->unlock();
    }
};
```

**Interview point:** Policy-based design trades runtime flexibility for compile-time customization and potential code bloat.

**Common mistake:** Turning simple runtime choices into template policies without a performance or design reason.

---

## 36. What is expression SFINAE?

**Short answer:** Expression SFINAE enables or disables templates based on whether an expression is valid.

**Detailed answer:**
Before concepts, expression SFINAE was commonly used to detect operations such as `begin()`, `size()`, or stream insertion. If substituting template arguments into an expression fails in a SFINAE context, that overload is removed instead of causing a hard error.

**Example:**

```cpp
template <typename T>
auto hasSizeImpl(int) -> decltype(std::declval<T>().size(), std::true_type{});

template <typename>
std::false_type hasSizeImpl(...);
```

Modern C++ often replaces this with requires-expressions.

**Interview point:** Concepts make many expression-SFINAE patterns easier to read and maintain.

**Common mistake:** Writing complex SFINAE when a simple `requires` clause expresses the same intent.

---

## 37. What is type erasure compared with templates?

**Short answer:** Templates provide compile-time polymorphism; type erasure provides runtime polymorphism behind one concrete wrapper type.

**Detailed answer:**
Templates generate code for each used type and can inline aggressively. Type erasure hides the concrete type, reducing template exposure and allowing heterogeneous runtime storage, often with some indirection cost.

**Example:**

```cpp
std::vector<std::function<void()>> tasks;
tasks.push_back([] { runA(); });
tasks.push_back([] { runB(); });
```

Each lambda has a different type, but `std::function<void()>` erases those types.

**Interview point:** Choose templates when compile-time type information matters; choose type erasure when runtime uniformity and ABI boundaries matter.

**Common mistake:** Exposing heavily templated APIs across boundaries where a stable erased interface would be simpler.

---

## 38. What is template code bloat?

**Short answer:** Template code bloat happens when many template instantiations generate large amounts of similar machine code.

**Detailed answer:**
Templates are instantiated for the types used. This can improve performance through specialization but may increase binary size, compile time, and instruction-cache pressure. Techniques such as type erasure, explicit instantiation, common helper functions, and reducing template parameters can help.

**Example scenario:**
A logging function templated on many string-like and numeric types may instantiate dozens of similar formatting paths.

**Interview point:** Templates are not free. Balance specialization benefits against build and binary costs.

**Common mistake:** Making every helper a template even when a non-template function would be sufficient.

---

## 39. What is an unevaluated operand in templates?

**Short answer:** An unevaluated operand is an expression context where the expression is checked for type or form but not executed.

**Detailed answer:**
Examples include `sizeof`, `decltype`, `noexcept`, and parts of requires-expressions. Unevaluated operands are widely used in templates to inspect types and expressions without running code.

**Example:**

```cpp
template <typename T>
auto sizeTypeOf(const T& value) -> decltype(value.size());
```

The `value.size()` expression in `decltype` is not executed; it is used to determine a type.

**Interview point:** Unevaluated operands are central to type traits, detection idioms, and constraints.

**Common mistake:** Expecting side effects inside `sizeof` or `decltype` to run.

---

## 40. How should you decide between templates, concepts, virtual functions, and type erasure?

**Short answer:** Use templates/concepts for compile-time generic code, virtual functions for explicit runtime hierarchies, and type erasure for runtime polymorphism behind value-like wrappers or stable APIs.

**Detailed answer:**
Each technique has tradeoffs. Templates give performance and type-specific optimization but expose implementation and increase compile-time coupling. Concepts make template requirements clearer. Virtual functions give stable runtime substitution but require inheritance. Type erasure hides concrete types behind a wrapper and is useful at boundaries.

**Decision guide:**
- Use templates when callers benefit from compile-time type information.
- Add concepts when requirements matter to readability and diagnostics.
- Use virtual functions for stable object-oriented interfaces.
- Use type erasure when you need runtime polymorphism without exposing inheritance or templates.

**Interview point:** Senior generic design is about choosing the right polymorphism mechanism, not using the most advanced one.

**Common mistake:** Replacing simple runtime interfaces with templates purely for style, or using virtual functions where a simple template algorithm is clearer.

---

## 41. What is a requires-clause compared with a requires-expression?

**Short answer:** A requires-clause constrains a template; a requires-expression checks whether expressions or type requirements are valid.

**Detailed answer:**
A requires-clause appears on a template or function declaration and controls whether it participates in overload resolution. A requires-expression appears inside a concept or constraint and evaluates compile-time requirements.

**Example:**

```cpp
template <typename T>
concept HasSize = requires(const T& value) {
    value.size();
};

template <typename T>
    requires HasSize<T>
auto sizeOf(const T& value) {
    return value.size();
}
```

The concept uses a requires-expression. The function uses a requires-clause.

**Interview point:** Both use the word `requires`, but they play different roles in template constraints.

**Common mistake:** Confusing the syntax that defines requirements with the syntax that applies requirements.

---

## 42. What is a compound requirement in a requires-expression?

**Short answer:** A compound requirement checks an expression and can also check `noexcept` and the expression's result type.

**Detailed answer:**
Compound requirements use braces inside a requires-expression. They can verify that an expression is valid, optionally non-throwing, and satisfies a type constraint.

**Example:**

```cpp
template <typename T>
concept Hashable = requires(const T& value) {
    { std::hash<T>{}(value) } -> std::convertible_to<std::size_t>;
};
```

This checks that hashing a `T` produces something convertible to `std::size_t`.

**Interview point:** Concepts can express semantic-looking interface requirements close to the code that needs them.

**Common mistake:** Checking only that a member exists while ignoring the required return type.

---

## 43. What is a nested requirement?

**Short answer:** A nested requirement is a `requires` statement inside a requires-expression that checks an additional compile-time boolean condition.

**Detailed answer:**
Nested requirements are useful when expression validity is not enough. They can check traits, relationships between types, sizes, or other constant expressions.

**Example:**

```cpp
template <typename T>
concept SmallType = requires {
    requires sizeof(T) <= 16;
};
```

This concept is satisfied only when the type size condition is true.

**Interview point:** Nested requirements connect concepts with arbitrary compile-time predicates.

**Common mistake:** Using nested requirements for checks that would be clearer as named concepts or traits.

---

## 44. What is a customization point object?

**Short answer:** A customization point object is a callable object that provides a controlled way for generic code to find customized behavior.

**Detailed answer:**
Modern generic libraries often use customization point objects to combine default behavior, argument-dependent lookup, constraints, and overload control. Ranges customization points such as `std::ranges::begin` are examples.

**Conceptual example:**

```cpp
inline constexpr struct size_fn {
    template <typename T>
    auto operator()(T&& value) const {
        return std::forward<T>(value).size();
    }
} size;
```

Real customization point objects are more careful about ADL, constraints, and fallback behavior.

**Interview point:** Customization points are a disciplined alternative to unconstrained overloads and ad hoc traits.

**Common mistake:** Exposing random overload hooks without defining lookup, priority, and constraints.

---

## 45. What is tag invocation?

**Short answer:** Tag invocation is a customization pattern where a tag object identifies an operation and user code customizes it by defining `tag_invoke` overloads.

**Detailed answer:**
The tag-invoke pattern gives libraries a uniform customization mechanism. The operation is represented by a tag type, and overload resolution finds a matching `tag_invoke` function for the argument types.

**Conceptual example:**

```cpp
struct serialize_t {};
inline constexpr serialize_t serialize{};

std::string tag_invoke(serialize_t, const User& user) {
    return user.name();
}
```

This pattern is advanced, but it helps avoid naming collisions and centralizes customization rules.

**Interview point:** Tag invocation is mainly relevant in library design, not everyday application code.

**Common mistake:** Introducing tag-invoke complexity when a simple overload or member function would be clearer.

---

## 46. What is a template recursion base case?

**Short answer:** A template recursion base case stops recursive template instantiation.

**Detailed answer:**
Before fold expressions and modern constexpr techniques, many metaprograms used recursive templates. Like runtime recursion, compile-time recursion needs a base case. Without one, compilation fails due to excessive instantiation depth.

**Example:**

```cpp
template <int N>
struct SumTo {
    static constexpr int value = N + SumTo<N - 1>::value;
};

template <>
struct SumTo<0> {
    static constexpr int value = 0;
};
```

**Interview point:** Template recursion errors often appear as long instantiation backtraces.

**Common mistake:** Forgetting the specialization or constraint that terminates recursive instantiation.

---

## 47. What is `std::integral_constant` used for?

**Short answer:** `std::integral_constant` wraps a compile-time value as a type.

**Detailed answer:**
Many type traits inherit from `std::integral_constant`, such as `std::true_type` and `std::false_type`. It lets compile-time values participate in type-based overloads, tag dispatch, and metaprogramming.

**Example:**

```cpp
#include <type_traits>

using yes = std::integral_constant<bool, true>;
static_assert(yes::value);
```

Modern code often uses `_v` helpers such as `std::is_integral_v<T>`, but understanding `integral_constant` helps explain how traits work.

**Interview point:** Type traits commonly represent answers as types carrying compile-time values.

**Common mistake:** Treating traits as runtime checks rather than compile-time information.

---

## 48. What is a dependent false helper?

**Short answer:** A dependent false helper is a template variable that stays dependent so `static_assert` fires only when a template branch is instantiated.

**Detailed answer:**
A plain `static_assert(false)` inside a template can fail immediately, even if that branch is never used. Making the false value depend on a template parameter delays the assertion until substitution or instantiation reaches that branch.

**Example:**

```cpp
template <typename>
inline constexpr bool dependent_false_v = false;

template <typename T>
void process(const T&) {
    if constexpr (std::is_integral_v<T>) {
        // handle integers
    } else {
        static_assert(dependent_false_v<T>, "unsupported type");
    }
}
```

**Interview point:** Dependency controls when template code is checked.

**Common mistake:** Using `static_assert(false)` in a discarded `if constexpr` branch and being surprised by an early compile error.

---

## 49. What is compile-time reflection, and does standard C++ have it?

**Short answer:** Compile-time reflection means inspecting program structure at compile time; current standard C++ has limited reflection-like tools but not full standardized reflection.

**Detailed answer:**
C++ templates, type traits, `decltype`, concepts, and compiler extensions provide partial introspection. Full reflection would allow code to inspect members, names, attributes, and structure in a standardized way. As of current mainstream C++ practice, libraries often use macros, code generation, traits, or external tools to fill the gap.

**Example workaround:**

```cpp
struct User {
    std::string name;
    int age;
};

// A library may require registering fields manually.
```

**Interview point:** Many serialization and binding systems solve the lack of standard reflection with explicit metadata.

**Common mistake:** Assuming templates can automatically enumerate arbitrary class data members.

---

## 50. How do you make template errors easier to understand?

**Short answer:** Use concepts, named constraints, small templates, clear static assertions, and simpler overload sets.

**Detailed answer:**
Template errors become difficult when requirements are implicit and failures occur deep inside implementation code. Concepts move requirements to the interface. Named helper concepts explain intent better than large inline constraints.

**Practical techniques:**

- Prefer named concepts for public APIs.
- Use `requires` clauses near declarations.
- Add targeted `static_assert` messages for unsupported cases.
- Keep template bodies small and delegate non-template work to ordinary functions.
- Avoid unconstrained forwarding overloads.
- Test templates with representative bad inputs as well as valid types.

**Interview point:** Good template design includes diagnostics as part of API usability.

**Common mistake:** Writing a powerful template that produces unreadable errors when used incorrectly.
