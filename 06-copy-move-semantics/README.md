# Copy Semantics and Move Semantics Interview Questions

This topic covers how C++ objects are copied, moved, assigned, and returned. Interviewers use these questions to evaluate whether a candidate understands ownership, performance, object lifetime, and modern C++ design.

## 1. What is copy semantics?

**Short answer:** Copy semantics define what happens when one object is copied from another object.

**Detailed answer:**
Copying should usually create a new object with the same logical value as the original. For simple classes, the compiler-generated copy constructor and copy assignment operator may be enough. For classes that own resources directly, custom copy behavior may be required.

**Example:**

```cpp
#include <string>

class User {
public:
    User(std::string name) : name_(std::move(name)) {}

private:
    std::string name_;
};

User a("Ada");
User b = a; // copy construction
```

`std::string` handles its own memory, so the compiler-generated copy behavior for `User` is usually correct.

**Interview point:** Copy semantics should preserve value meaning, not necessarily copy every internal implementation detail exactly.

---

## 2. What is the difference between a copy constructor and copy assignment operator?

**Short answer:** The copy constructor creates a new object from an existing object. Copy assignment replaces the value of an already existing object.

**Detailed answer:**
The copy constructor is called during initialization. The copy assignment operator is called when assigning to an object that already exists.

**Example:**

```cpp
class Widget {
public:
    Widget() = default;
    Widget(const Widget& other) = default;            // copy constructor
    Widget& operator=(const Widget& other) = default; // copy assignment
};

Widget a;
Widget b = a; // copy constructor
Widget c;
c = a;        // copy assignment
```

**Important difference:** Copy assignment must handle the old resources already owned by the target object. It also should be safe for self-assignment.

**Common mistake:** Treating initialization and assignment as the same operation.

---

## 3. What is shallow copy vs deep copy?

**Short answer:** A shallow copy copies pointer values. A deep copy duplicates the pointed-to resource.

**Detailed answer:**
A shallow copy may be fine for non-owning pointers, but it is dangerous for owning raw pointers. If two objects believe they own the same raw pointer, both may try to delete it, causing double free.

**Bad shallow-copy example:**

```cpp
class Buffer {
public:
    explicit Buffer(std::size_t size)
        : size_(size), data_(new int[size]) {}

    ~Buffer() {
        delete[] data_;
    }

private:
    std::size_t size_;
    int* data_;
};

Buffer a(10);
Buffer b = a; // compiler-generated shallow copy: dangerous
```

Both objects now contain the same `data_` pointer.

**Deep copy idea:**

```cpp
Buffer(const Buffer& other)
    : size_(other.size_), data_(new int[other.size_]) {
    std::copy(other.data_, other.data_ + size_, data_);
}
```

**Modern C++ recommendation:** Use `std::vector<int>` instead of manually owning an array.

---

## 4. What is move semantics?

**Short answer:** Move semantics allow resources to be transferred from one object to another instead of copied.

**Detailed answer:**
Moving is useful when an object owns expensive resources such as heap memory, file handles, or large buffers. Instead of allocating and copying, the destination can take ownership of the source's resource. The source remains valid but its value is unspecified unless documented otherwise.

**Example:**

```cpp
#include <string>
#include <utility>

std::string a = "large text";
std::string b = std::move(a);
```

After the move, `b` owns the content. `a` is still valid and can be assigned a new value, but code should not rely on its previous content.

**Interview point:** Move semantics are about ownership transfer and performance, not about simply copying faster.

---

## 5. What is an rvalue reference?

**Short answer:** An rvalue reference is a reference type written with `&&` that can bind to temporary objects and move candidates.

**Detailed answer:**
Rvalue references enable move constructors and move assignment operators. They allow a function to distinguish objects that may be safely moved from objects that should be copied.

**Example:**

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

private:
    int* data_ = nullptr;
    std::size_t size_ = 0;
};
```

The move constructor steals the pointer and leaves the source in a destructible state.

**Common mistake:** Thinking every `&&` always means an rvalue. In templates, `T&&` can be a forwarding reference depending on context.

---

## 6. What does `std::move` actually do?

**Short answer:** `std::move` casts an expression to an rvalue reference. It does not move anything by itself.

**Detailed answer:**
`std::move` simply says, "this object may be treated as movable." The actual move happens if a move constructor or move assignment operator is selected.

**Example:**

```cpp
#include <string>
#include <utility>

std::string source = "hello";
std::string target = std::move(source); // move constructor may run
```

For trivial types, moving and copying may be identical.

```cpp
int a = 10;
int b = std::move(a); // just copies the int value
```

**Common mistake:** Using an object normally after `std::move` and assuming it still has its old value. It is valid but may no longer contain the same value.

---

## 7. What is the Rule of Three?

**Short answer:** If a class needs a custom destructor, copy constructor, or copy assignment operator, it probably needs all three.

**Detailed answer:**
The Rule of Three applies to classes that manually manage resources. If a destructor releases a resource, copying must usually be customized so two objects do not incorrectly share ownership of the same resource.

**Example:**

```cpp
class Buffer {
public:
    explicit Buffer(std::size_t size);
    ~Buffer();
    Buffer(const Buffer& other);
    Buffer& operator=(const Buffer& other);
};
```

**Interview point:** The Rule of Three is a warning sign that the class is doing manual resource management.

**Modern C++ note:** Prefer standard library members so you can avoid writing these functions yourself.

---

## 8. What is the Rule of Five?

**Short answer:** If a class manually manages resources and defines copy/destruction behavior, it may also need move constructor and move assignment operator.

**Detailed answer:**
C++11 introduced move semantics. A resource-owning type can often be moved efficiently by transferring ownership instead of performing a deep copy.

**Rule of Five members:**

```cpp
class Buffer {
public:
    ~Buffer();
    Buffer(const Buffer& other);
    Buffer& operator=(const Buffer& other);
    Buffer(Buffer&& other) noexcept;
    Buffer& operator=(Buffer&& other) noexcept;
};
```

**Why `noexcept` matters:** Standard containers such as `std::vector` may prefer copying instead of moving during reallocation if the move constructor might throw.

**Common mistake:** Defining a destructor and copy operations but forgetting move operations, causing unnecessary copies or disabled moves.

---

## 9. What is the Rule of Zero?

**Short answer:** Design classes so they do not need custom destructors, copy constructors, copy assignment operators, move constructors, or move assignment operators.

**Detailed answer:**
The Rule of Zero says resource management should be delegated to members that already manage resources correctly, such as `std::string`, `std::vector`, `std::unique_ptr`, `std::shared_ptr`, file wrapper classes, and lock guards.

**Example:**

```cpp
#include <string>
#include <vector>

class Document {
public:
    void addLine(std::string line) {
        lines_.push_back(std::move(line));
    }

private:
    std::string title_;
    std::vector<std::string> lines_;
};
```

No custom destructor or copy/move operations are needed.

**Interview point:** The Rule of Zero is usually the best modern C++ design goal.

---

## 10. What is copy elision?

**Short answer:** Copy elision is a compiler optimization that removes unnecessary copy or move operations.

**Detailed answer:**
When returning objects by value, the compiler can construct the object directly in the caller's storage. Since C++17, some cases of copy elision are guaranteed.

**Example:**

```cpp
#include <string>

std::string makeName() {
    return std::string("Ada");
}

std::string name = makeName();
```

The returned string may be constructed directly as `name` without an extra copy or move.

**Named return value optimization:**

```cpp
std::string makeName() {
    std::string result = "Ada";
    return result;
}
```

Compilers commonly apply NRVO here, though it is not guaranteed in every case.

**Common mistake:** Avoiding return-by-value because of outdated performance concerns. Modern C++ often makes return-by-value efficient and clean.

---

## 11. What state is an object in after it has been moved from?

**Short answer:** A moved-from object must remain valid and destructible, but its value is usually unspecified.

**Detailed answer:**
After moving from an object, you can destroy it, assign a new value to it, or call functions that explicitly allow its current state. You should not assume it still contains its old value unless the type documents that guarantee.

**Example:**

```cpp
#include <string>
#include <utility>

std::string source = "hello";
std::string target = std::move(source);

source = "new value"; // safe: assigning a new value
```

For standard library types, moved-from objects are valid but their content is generally unspecified.

**Interview point:** Move constructors should leave the source object in a state where its destructor and assignment operator can run safely.

**Common mistake:** Checking a moved-from object as if it must be empty. Some implementations may leave it empty, but that is not the general rule.

---

## 12. What is perfect forwarding?

**Short answer:** Perfect forwarding preserves whether an argument was an lvalue or rvalue when passing it to another function.

**Detailed answer:**
Perfect forwarding is usually implemented with a forwarding reference and `std::forward`. It is common in generic factories, wrapper functions, and container emplacement functions.

**Example:**

```cpp
#include <memory>
#include <utility>

template <typename T, typename... Args>
std::unique_ptr<T> makeObject(Args&&... args) {
    return std::make_unique<T>(std::forward<Args>(args)...);
}
```

`std::forward<Args>(args)` preserves the value category of each original argument.

**Interview point:** Use `std::move` when you unconditionally want to treat something as movable. Use `std::forward` when preserving the caller's value category in a template.

**Common mistake:** Replacing every `std::forward` with `std::move`, which can incorrectly move from lvalue arguments.

---

## 13. What is the copy-and-swap idiom?

**Short answer:** Copy-and-swap implements assignment by making a copy first, then swapping it with the current object.

**Detailed answer:**
The copy-and-swap idiom can provide strong exception safety for assignment. If copying fails, the original object is unchanged. If copying succeeds, swapping commits the new value.

**Example:**

```cpp
#include <algorithm>
#include <cstddef>

class Buffer {
public:
    Buffer& operator=(Buffer other) {
        swap(other);
        return *this;
    }

    void swap(Buffer& other) noexcept {
        std::swap(data_, other.data_);
        std::swap(size_, other.size_);
    }

private:
    int* data_ = nullptr;
    std::size_t size_ = 0;
};
```

The parameter is taken by value, so it is copy-constructed for lvalues and move-constructed for rvalues.

**Tradeoff:** Copy-and-swap is simple and safe, but it may allocate more than a carefully optimized assignment operator.

**Common mistake:** Forgetting that assignment must handle self-assignment and existing resources owned by the target.

---

## 14. Why should move constructors often be marked `noexcept`?

**Short answer:** `noexcept` tells generic code that moving cannot throw, which allows containers to move elements safely during reallocation.

**Detailed answer:**
Standard containers such as `std::vector` need to preserve exception safety when growing. If moving an element might throw, a container may choose to copy elements instead, because copying can leave the original elements intact if something fails.

**Example:**

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

private:
    int* data_ = nullptr;
    std::size_t size_ = 0;
};
```

**Interview point:** A move operation that only swaps pointers, sizes, or handles should usually be `noexcept`.

**Common mistake:** Omitting `noexcept` and then being surprised that a standard container copies instead of moves.

---

## 15. What is a move-only type?

**Short answer:** A move-only type can be moved but not copied.

**Detailed answer:**
Move-only types represent exclusive ownership or resources that cannot be duplicated safely. Examples include `std::unique_ptr`, file handles, sockets, threads, and lock guards. Copy operations are deleted, while move operations transfer ownership.

**Example:**

```cpp
#include <memory>

class Owner {
public:
    explicit Owner(std::unique_ptr<int> value)
        : value_(std::move(value)) {}

    Owner(const Owner&) = delete;
    Owner& operator=(const Owner&) = delete;

    Owner(Owner&&) noexcept = default;
    Owner& operator=(Owner&&) noexcept = default;

private:
    std::unique_ptr<int> value_;
};
```

**Interview point:** Move-only design communicates ownership clearly and prevents accidental sharing or double cleanup.

**Common mistake:** Trying to store move-only objects in APIs that require copying, or passing them by value without using `std::move` when transferring ownership.

---

## 16. What is self-assignment, and why does it matter?

**Short answer:** Self-assignment happens when an object is assigned to itself, such as `x = x`.

**Detailed answer:**
Compiler-generated assignment operators usually handle self-assignment correctly. Custom assignment operators for resource-owning types must be careful not to destroy the current resource before copying from it.

**Bad example:**

```cpp
Buffer& operator=(const Buffer& other) {
    delete[] data_;
    size_ = other.size_;
    data_ = new int[size_];
    std::copy(other.data_, other.data_ + size_, data_);
    return *this;
}
```

If `other` is the same object as `*this`, this deletes the source before copying from it.

**Safer approaches:**
- Check `if (this == &other)` when needed.
- Allocate the new resource before releasing the old one.
- Use copy-and-swap for strong exception safety.

**Interview point:** Self-assignment is less about writing `x = x` directly and more about aliases causing the same object to appear on both sides.

**Common mistake:** Handling normal assignment but forgetting aliasing cases.

---

## 17. What is move assignment, and how is it different from move construction?

**Short answer:** Move construction creates a new object by taking resources from another object. Move assignment replaces an existing object's resources with resources from another object.

**Detailed answer:**
Move assignment must release or replace resources already owned by the target object. It also should be safe for self-move, even though self-move is less common than self-copy.

**Example:**

```cpp
class Buffer {
public:
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }

private:
    int* data_ = nullptr;
    std::size_t size_ = 0;
};
```

**Interview point:** Move construction starts with no existing resource in the destination. Move assignment must account for the destination's current state.

**Common mistake:** Implementing move assignment by stealing the source pointer without first releasing the target's old resource.

---

## 18. What is a forwarding reference?

**Short answer:** A forwarding reference is a `T&&` function parameter where `T` is a deduced template parameter.

**Detailed answer:**
Forwarding references can bind to both lvalues and rvalues. They are used with `std::forward` to preserve the caller's value category.

**Example:**

```cpp
#include <utility>

void consume(const std::string& value);
void consume(std::string&& value);

template <typename T>
void wrapper(T&& value) {
    consume(std::forward<T>(value));
}
```

If the caller passes an lvalue, `T` deduces as an lvalue reference type. If the caller passes an rvalue, `T` deduces as a non-reference type.

**Interview point:** Not every `&&` is a forwarding reference. `std::string&&` is an rvalue reference, not a forwarding reference.

**Common mistake:** Calling forwarding references "universal references" without understanding the deduction rule that makes them special.

---

## 19. What is the difference between `std::move` and `std::forward`?

**Short answer:** `std::move` always casts to an rvalue. `std::forward` conditionally casts based on the deduced template type.

**Detailed answer:**
Use `std::move` when you know the current function is done with an object and wants to allow moving from it. Use `std::forward` in forwarding-reference templates when passing an argument onward while preserving whether the original argument was an lvalue or rvalue.

**Example:**

```cpp
#include <utility>

void sink(std::string&& text);

void passLocal(std::string text) {
    sink(std::move(text));
}

template <typename T>
void forwardArgument(T&& value) {
    sink(std::forward<T>(value));
}
```

**Interview point:** `std::forward` without a deduced template type is usually a sign of confusion. `std::move` is for unconditional move intent.

**Common mistake:** Using `std::move` inside a generic wrapper and accidentally preventing lvalue overloads from being selected.

---

## 20. How do copy and move semantics affect standard containers?

**Short answer:** Containers copy or move elements during insertion, assignment, and reallocation depending on element type and operation.

**Detailed answer:**
When a `std::vector` grows, it must relocate existing elements into new storage. If moving is available and `noexcept`, the vector can usually move elements. If moving might throw and copying is available, it may copy to preserve exception safety.

**Example:**

```cpp
#include <vector>

std::vector<std::string> names;
names.push_back("Ada");

std::string text = "Grace";
names.push_back(text);            // copies text
names.push_back(std::move(text));  // may move from text
names.emplace_back("Linus");      // constructs in place
```

**Interview point:** Type design affects container performance. A resource-owning type with a `noexcept` move constructor works better with many standard containers.

**Common mistake:** Assuming `emplace_back` is always faster than `push_back`. It depends on the arguments and the type being constructed.

---

## 21. What is return value optimization, and how is it related to move semantics?

**Short answer:** Return value optimization lets the compiler construct a returned object directly in the caller's storage, avoiding a copy or move.

**Detailed answer:**
RVO and named return value optimization reduce unnecessary temporary objects. Since C++17, some forms of copy elision are mandatory, meaning no copy or move object is created at all.

This matters because `std::move` is not always needed when returning local objects. In fact, adding `std::move` can prevent NRVO in some cases.

**Example:**

```cpp
#include <string>

std::string makeName() {
    std::string name = "Ada";
    return name; // NRVO may construct directly in caller
}
```

**Mandatory copy elision example:**

```cpp
std::string makeTitle() {
    return std::string("Engineer"); // constructed directly in caller since C++17
}
```

**Interview point:** Copy elision is not just an optimization detail; modern C++ code often relies on returning objects by value as a clean and efficient design.

**Common mistake:** Writing `return std::move(local);` by habit, which can make code less optimizable.

---

## 22. What is the difference between moved-from state and invalid state?

**Short answer:** A moved-from object must remain valid and destructible, but its value is usually unspecified unless the type documents stronger guarantees.

**Detailed answer:**
After moving from an object, the object still exists. It can be destroyed, assigned a new value, or used in operations that the type explicitly supports. However, code should not assume it still contains its old value.

For standard library types, moved-from objects are valid but have unspecified values.

**Example:**

```cpp
#include <string>
#include <utility>

std::string source = "hello";
std::string target = std::move(source);

source = "new value"; // valid
```

It is valid to assign to `source`, but it is not portable to assume `source.empty()` after the move.

**Interview point:** Moving transfers resources or value representation, but it does not destroy the source object.

**Common mistake:** Treating a moved-from object as if it were destroyed, or assuming it always becomes empty.

---

## 23. How do deleted copy operations communicate ownership?

**Short answer:** Deleting copy operations tells users that the object cannot be duplicated, usually because it owns a unique resource.

**Detailed answer:**
Some resources cannot be safely copied: file descriptors, mutexes, sockets, threads, and unique heap ownership are common examples. Deleting copy construction and copy assignment prevents accidental double ownership.

Such types may still support moves, which transfer ownership instead of duplicating it.

**Example:**

```cpp
#include <memory>

class BufferOwner {
public:
    explicit BufferOwner(std::size_t size)
        : data_(std::make_unique<char[]>(size)) {}

    BufferOwner(const BufferOwner&) = delete;
    BufferOwner& operator=(const BufferOwner&) = delete;

    BufferOwner(BufferOwner&&) noexcept = default;
    BufferOwner& operator=(BufferOwner&&) noexcept = default;

private:
    std::unique_ptr<char[]> data_;
};
```

**Interview point:** Deleted functions are part of the API. They explain what operations are intentionally unsupported.

**Common mistake:** Leaving compiler-generated copy operations enabled for a class that owns a raw resource.

---

## 24. What are ref-qualified member functions?

**Short answer:** Ref-qualified member functions choose different overloads depending on whether `*this` is an lvalue or an rvalue.

**Detailed answer:**
A member function can be qualified with `&` or `&&`. This controls whether it can be called on lvalue objects, rvalue objects, or both. It is useful for APIs that want to prevent dangling references or enable efficient moves from temporaries.

**Example:**

```cpp
#include <string>
#include <utility>

class Message {
public:
    const std::string& text() const & {
        return text_;
    }

    std::string text() && {
        return std::move(text_);
    }

private:
    std::string text_ = "hello";
};
```

Calling `text()` on an lvalue returns a reference. Calling it on a temporary returns a string by value, avoiding a dangling reference.

**Interview point:** Ref-qualifiers let classes design APIs around the value category of the object itself, not just function parameters.

**Common mistake:** Returning references from temporary objects without considering rvalue-qualified overloads.

---

## 25. What is perfect forwarding, and when can it be dangerous?

**Short answer:** Perfect forwarding passes arguments onward while preserving their value categories, but it can make overload resolution, lifetimes, and error messages harder to reason about.

**Detailed answer:**
Perfect forwarding combines forwarding references with `std::forward`. It is useful for wrapper functions, factories, and container insertion APIs. However, it can accidentally forward arguments to unintended overloads, keep references to short-lived objects, or make APIs too permissive.

**Example:**

```cpp
#include <memory>
#include <utility>

template <typename T, typename... Args>
std::unique_ptr<T> makeObject(Args&&... args) {
    return std::make_unique<T>(std::forward<Args>(args)...);
}
```

**Danger example:**

```cpp
template <typename T>
void store(T&& value) {
    savedReference = &value; // dangerous: may refer to a temporary
}
```

Forwarding preserves the caller's value category, but it does not automatically solve ownership or lifetime.

**Interview point:** Perfect forwarding is a powerful library technique, not something every ordinary function needs.

**Common mistake:** Using forwarding references everywhere instead of simpler overloads, `const&`, or pass-by-value designs.

---

## 26. What is the difference between an lvalue, xvalue, and prvalue?

**Short answer:** An lvalue has identity, an xvalue has identity and is eligible to be moved from, and a prvalue mainly computes or initializes a value.

**Detailed answer:**
Modern C++ value categories are more precise than just "lvalue vs rvalue." An lvalue names a persistent object, such as a variable. An xvalue is an expiring value, usually produced by `std::move`. A prvalue is a pure rvalue, such as a literal or temporary-producing expression.

**Example:**

```cpp
std::string s = "hello";

s;              // lvalue expression
std::move(s);   // xvalue expression
std::string{};  // prvalue expression
```

**Interview point:** Move constructors usually bind to xvalues and prvalues, but named variables are lvalues even if their type is an rvalue reference.

**Common mistake:** Thinking a variable declared as `T&&` is automatically an rvalue expression when used by name.

---

## 27. Why are named rvalue references lvalues?

**Short answer:** Because any named object expression has identity and can be referred to again, so it is an lvalue expression.

**Detailed answer:**
An rvalue reference type can bind to a temporary, but once inside a function, the parameter has a name. Using that name is an lvalue expression. You must use `std::move` or `std::forward` to pass it onward as an rvalue when appropriate.

**Example:**

```cpp
void consume(std::string&& text);

void wrapper(std::string&& text) {
    // consume(text);            // error or calls lvalue overload if available
    consume(std::move(text));    // OK: explicitly casts to rvalue
}
```

**Interview point:** Types and expression value categories are related but not identical.

**Common mistake:** Forgetting `std::move` inside move constructors or rvalue-reference overloads.

---

## 28. What is implicit move from local variables in return statements?

**Short answer:** When returning a local object by value, C++ may elide the copy or implicitly move when elision does not apply.

**Detailed answer:**
Returning local objects by value is idiomatic. The compiler may use NRVO, or it may treat the local as movable in the return statement. Adding `std::move` can sometimes prevent NRVO and should usually be avoided for ordinary local returns.

**Example:**

```cpp
std::vector<int> makeValues() {
    std::vector<int> values = {1, 2, 3};
    return values; // NRVO or implicit move
}
```

**Interview point:** Prefer clear return-by-value. Do not add `std::move` to local returns unless you have a specific reason.

**Common mistake:** Writing `return std::move(values);` by habit.

---

## 29. What is a defaulted move constructor?

**Short answer:** A defaulted move constructor asks the compiler to move each base and member according to its own move behavior.

**Detailed answer:**
Defaulted move operations are often correct when all members already manage their own resources. They preserve predictable member-wise behavior and reduce manual errors.

**Example:**

```cpp
class Buffer {
public:
    Buffer(Buffer&&) noexcept = default;
    Buffer& operator=(Buffer&&) noexcept = default;

private:
    std::vector<char> data_;
};
```

If a member is not movable, the defaulted move constructor may be deleted.

**Interview point:** Default move operations are strongest when the class follows the Rule of Zero or stores RAII members.

**Common mistake:** Writing a custom move constructor that does the same thing as the compiler but forgets one member.

---

## 30. When are implicit move operations suppressed?

**Short answer:** Declaring certain special member functions, especially copy operations or destructors in older rules, can prevent implicit move operations from being generated.

**Detailed answer:**
The compiler generates move operations only when language rules allow it. User-declared copy constructors, copy assignments, move operations, or destructors can affect generation. The exact rules vary somewhat by C++ standard, so explicit `= default` is often clearer when move behavior is intended.

**Example:**

```cpp
class FileHandle {
public:
    ~FileHandle();
    FileHandle(FileHandle&&) noexcept = default;
    FileHandle& operator=(FileHandle&&) noexcept = default;
};
```

**Interview point:** If a type has custom lifetime management, review all five special member functions deliberately.

**Common mistake:** Adding a destructor for logging and accidentally changing move/copy behavior expectations.

---

## 31. What is the difference between copyable, movable, and semiregular types?

**Short answer:** A copyable type can be duplicated, a movable type can transfer state, and a semiregular type behaves much like an ordinary value with default construction, copy, move, and destruction.

**Detailed answer:**
Modern C++ generic code often talks about type requirements. A move-only type such as `std::unique_ptr` can be moved but not copied. A regular value type behaves like an `int`: it can be copied, moved, compared, and assigned without surprising identity effects.

**Example:**

```cpp
std::unique_ptr<int> p = std::make_unique<int>(1);
auto q = std::move(p); // movable, not copyable
```

**Interview point:** Generic algorithms and containers often require specific copy/move properties, so type design affects where a type can be used.

**Common mistake:** Making resource-owning types copyable without defining meaningful ownership semantics.

---

## 32. What is a trivially copyable type?

**Short answer:** A trivially copyable type can have its object representation copied with byte operations such as `std::memcpy` under specific rules.

**Detailed answer:**
Trivially copyable types have simple copy behavior without user-defined complex ownership semantics. They are important for binary I/O, shared memory, atomics in some cases, and low-level optimization. But byte copying still must respect object lifetime, alignment, and portability.

**Example:**

```cpp
#include <type_traits>

struct Point {
    int x;
    int y;
};

static_assert(std::is_trivially_copyable_v<Point>);
```

**Interview point:** Trivially copyable does not automatically mean portable serialization across machines or compiler versions.

**Common mistake:** Using `memcpy` on classes with pointers, ownership, virtual functions, or non-trivial destructors.

---

## 33. What is object slicing during copy?

**Short answer:** Object slicing occurs when copying a derived object into a base object by value, losing the derived part.

**Detailed answer:**
If a function takes a base class by value, passing a derived object copies only the base subobject. Virtual dispatch will then operate on the sliced base object, not the original derived object.

**Example:**

```cpp
class Base {
public:
    virtual void print() const;
};

class Derived : public Base {
public:
    void print() const override;
};

void log(Base value); // slices Derived objects
```

Use references, pointers, or clone functions for polymorphic objects.

**Interview point:** Polymorphic types should usually be passed by reference or pointer, not by value.

**Common mistake:** Storing derived objects in `std::vector<Base>` and expecting polymorphism.

---

## 34. What is a clone function in copyable polymorphic hierarchies?

**Short answer:** A clone function creates a dynamic copy of the actual derived object through a base interface.

**Detailed answer:**
Polymorphic objects cannot be copied correctly through a base value without slicing. A virtual clone function lets each derived class return a copy of itself while callers work through the base type.

**Example:**

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
};

class Circle : public Shape {
public:
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);
    }
};
```

**Interview point:** Clone expresses polymorphic copy explicitly and avoids accidental slicing.

**Common mistake:** Relying on a base-class copy constructor to copy derived state.

---

## 35. What is the copy-and-move unification pattern?

**Short answer:** It is an assignment style where a parameter is taken by value and then swapped into the object, handling both copy and move assignment.

**Detailed answer:**
Taking the right-hand side by value lets the caller decide whether construction is a copy or move. The assignment body then swaps with the temporary. This can be simple and exception-safe for value-like types.

**Example:**

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

**Tradeoff:**
This may do extra work compared with specialized copy and move assignment, especially for types where move assignment can cheaply reuse existing storage.

**Interview point:** Unified assignment is elegant for some value types but not automatically optimal for every resource-owning type.

**Common mistake:** Using copy-and-swap everywhere without considering storage reuse or performance.

---

## 36. What is self-move assignment?

**Short answer:** Self-move assignment happens when an object is move-assigned from itself, directly or indirectly.

**Detailed answer:**
Self-move is unusual but possible, especially in generic code. A robust move assignment operator should leave the object valid even if `this == &other`. It does not necessarily need to preserve the original value, but it must not double free or corrupt invariants.

**Example:**

```cpp
value = std::move(value); // self-move
```

**Interview point:** Standard library types remain valid after self-move; custom RAII types should avoid catastrophic behavior.

**Common mistake:** Implementing move assignment that deletes the resource and then reads the same resource from `other` when `other` is `*this`.

---

## 37. What is perfect forwarding overload hijacking?

**Short answer:** A forwarding-reference overload can match too broadly and steal calls from more appropriate overloads.

**Detailed answer:**
A template such as `template <typename T> f(T&&)` can bind to many argument types. It may become a better match than a copy constructor, overload, or intended specific function, causing surprising errors or behavior.

**Example:**

```cpp
class Person {
public:
    Person(const Person&);

    template <typename T>
    explicit Person(T&& name); // may accidentally match Person&
};
```

Constraints can prevent unintended matches.

```cpp
template <typename T>
    requires std::convertible_to<T, std::string>
explicit Person(T&& name);
```

**Interview point:** Perfect forwarding should usually be constrained in public APIs.

**Common mistake:** Adding an unconstrained forwarding constructor and breaking copy construction.

---

## 38. What is moving from `const`?

**Short answer:** Moving from a `const` object usually cannot call a move constructor that modifies the source, so it often copies instead.

**Detailed answer:**
`std::move` does not move by itself; it casts. If the object is `const`, `std::move` produces `const T&&`. Most move constructors take `T&&`, not `const T&&`, because moving normally modifies the source.

**Example:**

```cpp
const std::string name = "Ada";
std::string other = std::move(name); // usually copies, because name is const
```

**Interview point:** Avoid marking local objects `const` if you intend to move from them later.

**Common mistake:** Believing `std::move` forces a move regardless of constness.

---

## 39. What is the difference between relocation and move construction?

**Short answer:** Move construction creates a new object from an existing one; relocation would move an object to new storage and end the old lifetime as one operation.

**Detailed answer:**
C++ move construction still runs constructors and destructors according to object lifetime rules. Containers often relocate elements during growth by move-constructing into new storage and destroying old elements. For trivially copyable types, implementations may optimize this with byte moves, but general C++ objects need proper move construction.

**Example:**

```cpp
std::vector<std::string> names;
names.push_back("Ada");
names.push_back("Grace"); // growth may move strings to new storage
```

**Interview point:** `memmove` is not a general replacement for moving C++ objects.

**Common mistake:** Reallocating raw storage and byte-copying objects with non-trivial move/destructor behavior.

---

## 40. How do move semantics affect API design?

**Short answer:** Move semantics let APIs express ownership transfer and efficient value passing, but they require clear post-move and lifetime expectations.

**Detailed answer:**
APIs can take move-only types to express ownership transfer, return objects by value efficiently, and accept values when the function will store a copy or move. Good APIs avoid requiring callers to reason about hidden moved-from states.

**Examples:**

```cpp
void setName(std::string name) {
    name_ = std::move(name);
}

void takeConnection(std::unique_ptr<Connection> connection);
```

The first pattern works well when the function stores the argument. The second clearly transfers ownership.

**Interview point:** Move semantics are not just an optimization; they are part of API ownership design.

**Common mistake:** Exposing rvalue-reference parameters everywhere instead of choosing simple pass-by-value, `const&`, or ownership-taking types.

---

## 41. What is a conditionally `noexcept` move constructor?

**Short answer:** A conditionally `noexcept` move constructor is `noexcept` only when the moves of its members are also `noexcept`.

**Detailed answer:**
Generic types often contain members whose move operations may or may not throw. A conditional `noexcept` specification lets the type accurately report its move safety. This matters for containers such as `std::vector`, which may prefer copying over moving during reallocation if moving can throw and copying is available.

**Example:**

```cpp
#include <type_traits>
#include <utility>

template <typename T>
class Box {
public:
    Box(Box&&) noexcept(std::is_nothrow_move_constructible_v<T>) = default;

private:
    T value_;
};
```

**Interview point:** `noexcept` is part of move-aware performance and exception-safety design.

**Common mistake:** Omitting `noexcept` from move operations of resource-owning types that cannot actually throw.

---

## 42. What is `std::move_if_noexcept`?

**Short answer:** `std::move_if_noexcept` moves an object only when moving is declared safe enough; otherwise it may return a const reference to encourage copying.

**Detailed answer:**
`std::move_if_noexcept` is useful in generic code that wants strong exception safety. If a move constructor might throw and a copy constructor is available, copying can preserve the original object in case construction of the new object fails.

**Example:**

```cpp
#include <utility>

T makeReplacement(T& value) {
    return T(std::move_if_noexcept(value));
}
```

Containers use this idea when relocating elements.

**Interview point:** Moving is not always the safest operation when strong exception guarantees matter.

**Common mistake:** Assuming generic code should always use `std::move` when it no longer needs a value.

---

## 43. What is copy elision in parameter passing?

**Short answer:** Copy elision can remove some temporary object copies, but function parameters are still real objects initialized from arguments.

**Detailed answer:**
Returning prvalues benefits from guaranteed copy elision in modern C++, but passing by value still constructs the parameter from the argument. If the argument is a temporary, the parameter can often be constructed directly. If the argument is an lvalue, it must be copied unless explicitly moved.

**Example:**

```cpp
void setName(std::string name);

setName("Ada");      // constructs parameter from temporary string
std::string s = "Grace";
setName(s);          // copies into parameter
setName(std::move(s)); // moves into parameter
```

**Interview point:** Pass-by-value is often good for sink parameters, but it still has different costs for lvalues and rvalues.

**Common mistake:** Saying pass-by-value is always free because of copy elision.

---

## 44. What is the difference between a sink parameter and a borrowing parameter?

**Short answer:** A sink parameter takes ownership or stores a value; a borrowing parameter only observes the argument during the call.

**Detailed answer:**
Move semantics make API intent clearer. A function that stores a string can take it by value and move from the parameter. A function that only reads a string should take `std::string_view` or `const std::string&` depending on needs.

**Example:**

```cpp
class User {
public:
    void setName(std::string name) {
        name_ = std::move(name);
    }

    bool hasName(std::string_view name) const;

private:
    std::string name_;
};
```

**Interview point:** Parameter passing should express lifetime and ownership, not only micro-optimization preferences.

**Common mistake:** Taking `T&&` for read-only parameters because it looks efficient.

---

## 45. What is a forwarding reference in a generic lambda?

**Short answer:** A generic lambda parameter declared as `auto&&` behaves like a forwarding reference.

**Detailed answer:**
In a generic lambda, `auto&&` can bind to lvalues and rvalues. Inside the lambda, the parameter has a name, so it is an lvalue expression. Use `std::forward<decltype(param)>(param)` to preserve the original value category.

**Example:**

```cpp
auto wrapper = [](auto&& value) {
    consume(std::forward<decltype(value)>(value));
};
```

This is the lambda equivalent of perfect forwarding in a function template.

**Interview point:** Forwarding rules apply to generic lambdas, not only named function templates.

**Common mistake:** Calling `std::move(value)` inside a generic lambda and moving from lvalues unexpectedly.

---

## 46. What is a move-only callback?

**Short answer:** A move-only callback is a callable object that cannot be copied, often because it captures move-only state.

**Detailed answer:**
A lambda that captures a `std::unique_ptr` by move becomes move-only. Older callback wrappers such as `std::function` require copyable callables, so move-only callbacks need alternatives such as C++23 `std::move_only_function`, custom wrappers, or direct template storage.

**Example:**

```cpp
auto task = [resource = std::make_unique<Resource>()]() mutable {
    resource->run();
};
```

The lambda owns `resource`, so copying it would imply duplicating unique ownership.

**Interview point:** Callback storage type must match callable ownership semantics.

**Common mistake:** Trying to put a move-only lambda into `std::function` and treating the compiler error as mysterious.

---

## 47. What is accidental copying in range-based loops?

**Short answer:** Accidental copying happens when a loop variable is declared by value instead of by reference.

**Detailed answer:**
Range-based loops copy each element when the loop variable is a value. This can be expensive or impossible for move-only types. Use `const auto&` for read-only access, `auto&` for mutation, and `auto&&` in generic code.

**Example:**

```cpp
std::vector<std::string> names = {"Ada", "Grace"};

for (auto name : names) {        // copies each string
    use(name);
}

for (const auto& name : names) { // borrows each string
    use(name);
}
```

**Interview point:** Copy semantics appear in everyday syntax, not only explicit constructors.

**Common mistake:** Using `auto` everywhere in loops without considering whether it copies.

---

## 48. What is the difference between copying a handle and duplicating a resource?

**Short answer:** Copying a handle value may only copy an identifier, while duplicating a resource creates an independent ownership relationship according to the operating system or API.

**Detailed answer:**
Some resources cannot be safely copied by copying their raw handle. For example, copying a file descriptor integer creates two wrapper objects that both think they own the same descriptor, causing double close. A true duplicate requires an API such as `dup` on POSIX or an equivalent handle-duplication function.

**Example risk:**

```cpp
class File {
public:
    File(const File&) = delete;
private:
    int fd_ = -1;
};
```

If copy semantics are needed, they must perform real resource duplication.

**Interview point:** Copying ownership wrappers requires semantic duplication, not just member-wise copying.

**Common mistake:** Letting the compiler-generated copy constructor copy raw resource handles.

---

## 49. What is a cheap-to-move type?

**Short answer:** A cheap-to-move type can transfer its state with little work, usually by swapping or copying a few handles or pointers.

**Detailed answer:**
Move semantics are most beneficial when moving avoids deep copying. Types such as `std::vector`, `std::string`, and `std::unique_ptr` are often cheap to move because they can transfer ownership of internal storage. Small trivially copyable types may be just as cheap to copy as to move.

**Example:**

```cpp
std::vector<int> a = makeLargeVector();
std::vector<int> b = std::move(a); // usually transfers buffer ownership
```

The moved-from vector remains valid but its contents are unspecified.

**Interview point:** Move is a semantic operation; its performance benefit depends on the type's representation.

**Common mistake:** Assuming moving every type is faster than copying every type.

---

## 50. How do you audit copy and move semantics for a custom type?

**Short answer:** Check ownership, invariants, exception guarantees, moved-from state, self-assignment, and whether generated operations match the type's meaning.

**Detailed answer:**
A custom type should define copy and move behavior according to what the type represents. Value types usually copy values. Unique resource owners usually delete copy and implement `noexcept` move. Polymorphic bases often disable value copying or provide `clone`.

**Audit checklist:**

- Does the type own a resource?
- Should copying be allowed, deep, shallow, or disabled?
- Does moving leave the source destructible and assignable?
- Are move operations `noexcept` when possible?
- Does assignment handle self-assignment and self-move safely?
- Do member types already implement correct behavior?
- Are tests covering copy, move, destruction, and containers?

**Interview point:** Correct copy/move semantics follow from the type's ownership model.

**Common mistake:** Writing special member functions manually before deciding what copying the type should mean.
