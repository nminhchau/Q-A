# Object-Oriented Programming in C++ Interview Questions

This topic covers core object-oriented programming concepts in C++. Interviewers usually expect more than definitions: they want to know whether you understand object lifetime, virtual dispatch, inheritance tradeoffs, slicing, destructors, and interface design.

## 1. What is a class in C++?

**Short answer:** A class is a user-defined type that groups data and functions together.

**Detailed answer:**
A class can contain data members, member functions, constructors, destructors, access control, overloaded operators, static members, and nested types. It is the main mechanism for encapsulation in C++.

**Example:**

```cpp
#include <string>

class User {
public:
    User(std::string name, int age)
        : name_(std::move(name)), age_(age) {}

    const std::string& name() const {
        return name_;
    }

    int age() const {
        return age_;
    }

private:
    std::string name_;
    int age_;
};
```

**Interview point:** A class should protect invariants. Private data is not just about hiding fields; it helps ensure objects remain valid.

**Common mistake:** Treating classes only as containers for data. Good classes model behavior and ownership, not just fields.

---

## 2. What is the difference between `class` and `struct` in C++?

**Short answer:** The only language-level difference is default access: `class` defaults to private, while `struct` defaults to public.

**Detailed answer:**
Both `class` and `struct` can have constructors, destructors, member functions, inheritance, templates, and access specifiers. In C++, `struct` is not limited to plain data like in C.

**Example:**

```cpp
class A {
    int x; // private by default
};

struct B {
    int x; // public by default
};
```

**Common convention:** Use `struct` for simple aggregate-like data types and `class` for types with stronger invariants or behavior.

**Common mistake:** Saying structs cannot have methods in C++. They can.

---

## 3. What is encapsulation?

**Short answer:** Encapsulation means keeping data and the operations on that data together while controlling access to internal state.

**Detailed answer:**
Encapsulation allows a class to expose a stable public interface while hiding implementation details. This makes code easier to maintain because internal representation can change without affecting callers.

**Example:**

```cpp
class BankAccount {
public:
    explicit BankAccount(int initialBalance)
        : balance_(initialBalance) {}

    bool withdraw(int amount) {
        if (amount < 0 || amount > balance_) {
            return false;
        }
        balance_ -= amount;
        return true;
    }

    int balance() const {
        return balance_;
    }

private:
    int balance_;
};
```

The class prevents invalid withdrawals and protects the account invariant.

**Interview point:** Getters and setters for every field can weaken encapsulation if they expose the object as just a data bag.

---

## 4. What is inheritance?

**Short answer:** Inheritance lets one class derive from another class, reusing or extending its interface and behavior.

**Detailed answer:**
Inheritance models an "is-a" relationship when used correctly. A derived class can use base class members, override virtual functions, and be used polymorphically through a base pointer or reference.

**Example:**

```cpp
#include <iostream>

class Animal {
public:
    virtual void speak() const {
        std::cout << "some sound\n";
    }
};

class Dog : public Animal {
public:
    void speak() const override {
        std::cout << "bark\n";
    }
};
```

**Interview point:** Prefer composition over inheritance when the relationship is not truly "is-a". Inheritance creates tighter coupling.

**Common mistake:** Using inheritance only to reuse code. Composition is often a better design for code reuse.

---

## 5. What is polymorphism?

**Short answer:** Polymorphism allows code to work with different types through a common interface.

**Detailed answer:**
C++ supports compile-time polymorphism through function overloading and templates, and runtime polymorphism through virtual functions.

**Runtime polymorphism example:**

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
};

class Circle : public Shape {
public:
    explicit Circle(double radius) : radius_(radius) {}

    double area() const override {
        return 3.14159 * radius_ * radius_;
    }

private:
    double radius_;
};

void printArea(const Shape& shape) {
    std::cout << shape.area() << '\n';
}
```

`printArea` can work with any type derived from `Shape`.

**Common mistake:** Thinking polymorphism only means inheritance. Templates also provide polymorphism, but at compile time.

---

## 6. What is a virtual function?

**Short answer:** A virtual function is a member function whose call can be resolved at runtime based on the dynamic type of the object.

**Detailed answer:**
When a function is declared `virtual` in a base class, derived classes can override it. If the function is called through a base pointer or reference, the derived implementation is selected at runtime.

**Example:**

```cpp
#include <iostream>

class Base {
public:
    virtual void print() const {
        std::cout << "Base\n";
    }
};

class Derived : public Base {
public:
    void print() const override {
        std::cout << "Derived\n";
    }
};

void call(const Base& object) {
    object.print();
}
```

If `call` receives a `Derived`, it prints `Derived`.

**Interview point:** Use `override` in derived classes so the compiler catches signature mistakes.

**Common mistake:** Expecting runtime polymorphism when passing objects by value. Passing by value can slice the derived part.

---

## 7. What is a pure virtual function?

**Short answer:** A pure virtual function is declared with `= 0` and usually makes the class abstract.

**Detailed answer:**
A class with at least one pure virtual function cannot be instantiated directly. It defines an interface or partial interface that derived classes must implement.

**Example:**

```cpp
class Logger {
public:
    virtual ~Logger() = default;
    virtual void log(const std::string& message) = 0;
};
```

A derived class must provide `log` before it can be instantiated.

```cpp
class ConsoleLogger : public Logger {
public:
    void log(const std::string& message) override {
        std::cout << message << '\n';
    }
};
```

**Important note:** A pure virtual function can still have a definition, but the class remains abstract.

**Common mistake:** Forgetting to make the base destructor virtual when the class is intended for polymorphic deletion.

---

## 8. Why should a polymorphic base class have a virtual destructor?

**Short answer:** So deleting a derived object through a base pointer calls the derived destructor correctly.

**Detailed answer:**
If a base class has virtual functions and objects may be deleted through base pointers, the destructor should be virtual. Without a virtual destructor, deleting through a base pointer causes undefined behavior.

**Bad example:**

```cpp
class Base {
public:
    virtual void work() {}
    ~Base() = default; // not virtual
};

class Derived : public Base {
public:
    ~Derived() {
        // cleanup
    }
};

Base* p = new Derived();
delete p; // undefined behavior
```

**Correct example:**

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual void work() = 0;
};
```

**Interview tip:** If a class has any virtual function, consider whether its destructor should be virtual. For interface-like bases, it almost always should be.

---

## 9. What is object slicing?

**Short answer:** Object slicing happens when a derived object is copied into a base object by value, losing the derived part.

**Detailed answer:**
When passing or assigning a derived object to a base object by value, only the base subobject is copied. Virtual dispatch no longer sees the original derived object because the result is a real base object.

**Example:**

```cpp
#include <iostream>

class Base {
public:
    virtual ~Base() = default;
    virtual void print() const {
        std::cout << "Base\n";
    }
};

class Derived : public Base {
public:
    void print() const override {
        std::cout << "Derived\n";
    }
};

void bad(Base object) {
    object.print(); // Base, because slicing occurred
}

void good(const Base& object) {
    object.print(); // Derived if actual object is Derived
}
```

**Common fix:** Use references, pointers, or smart pointers for polymorphic objects.

**Common mistake:** Thinking `virtual` prevents slicing. It does not.

---

## 10. What is the difference between public, protected, and private inheritance?

**Short answer:** They control how accessible base class members become in the derived class interface.

**Detailed answer:**
With public inheritance, public base members stay public and the relationship usually means "is-a." With protected inheritance, public and protected base members become protected in the derived class. With private inheritance, they become private.

**Example:**

```cpp
class Base {
public:
    void publicMethod() {}
protected:
    void protectedMethod() {}
};

class PublicDerived : public Base {};
class PrivateDerived : private Base {};
```

A `PublicDerived` object can be used where a `Base` is expected. A `PrivateDerived` object does not expose that public base interface to callers.

**Interview point:** Public inheritance models substitutability. Private inheritance is more like implementation reuse, but composition is often clearer.

**Common mistake:** Using private inheritance when a member object would be simpler and less coupled.

---

## 11. What is the difference between inheritance and composition?

**Short answer:** Inheritance models an "is-a" relationship. Composition models a "has-a" or "uses-a" relationship.

**Detailed answer:**
Inheritance lets a derived class reuse and override base class behavior, but it tightly couples the derived class to the base class interface and implementation assumptions. Composition builds a type by storing other objects as members, which is often more flexible and easier to change.

**Inheritance example:**

```cpp
class Dog : public Animal {
public:
    void speak() const override;
};
```

**Composition example:**

```cpp
class Car {
public:
    void start() {
        engine_.start();
    }

private:
    Engine engine_;
};
```

**Interview point:** Prefer composition when the relationship is not truly substitutable. Inheritance should mean a derived object can safely be used wherever the base is expected.

**Common mistake:** Using inheritance only to reuse code.

---

## 12. What is the Liskov Substitution Principle?

**Short answer:** Derived classes should be usable anywhere the base class is expected without breaking correctness.

**Detailed answer:**
The Liskov Substitution Principle is about behavioral compatibility, not just matching function signatures. A derived class should not violate the promises made by the base class.

**Example problem:**
If a base class `Bird` has a `fly()` method, then deriving `Penguin` from `Bird` creates a design issue because penguins cannot fly. The type hierarchy does not match real behavior.

**Better design:**

```cpp
class Bird {};

class FlyingBird : public Bird {
public:
    virtual void fly() = 0;
};

class Penguin : public Bird {};
```

**Interview point:** Good inheritance design depends on substitutability, not taxonomy alone.

**Common mistake:** Assuming real-world category relationships always make good inheritance relationships.

---

## 13. What is the difference between an abstract class and an interface-like class in C++?

**Short answer:** An abstract class cannot be instantiated because it has at least one pure virtual function. An interface-like class usually contains only pure virtual functions and a virtual destructor.

**Detailed answer:**
C++ does not have a separate `interface` keyword. Interface-like classes are built using abstract base classes with pure virtual functions. Abstract classes may also contain data members, helper functions, and partial implementations.

**Example interface-like class:**

```cpp
class Reader {
public:
    virtual ~Reader() = default;
    virtual std::string read() = 0;
};
```

**Example abstract class with shared behavior:**

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;

    bool isLarge() const {
        return area() > 1000.0;
    }
};
```

**Interview point:** Keep interface-like classes small and focused. Large interfaces force implementations to depend on methods they may not need.

---

## 14. What is the cost of virtual dispatch?

**Short answer:** Virtual dispatch usually costs an indirect function call and can prevent some compiler optimizations.

**Detailed answer:**
Virtual calls are typically implemented through a vtable. Calling a virtual function through a base pointer or reference requires loading the function address indirectly. This can inhibit inlining and may affect branch prediction or instruction-cache behavior.

**Example:**

```cpp
void drawAll(const std::vector<std::unique_ptr<Shape>>& shapes) {
    for (const auto& shape : shapes) {
        shape->draw(); // virtual dispatch
    }
}
```

**Important perspective:**
The cost is often small compared with I/O, allocation, cache misses, or real work inside the function. Avoid virtual dispatch only when profiling shows it matters or when compile-time polymorphism is a better design.

**Alternatives:**
- Templates for compile-time polymorphism
- `std::variant` for closed sets of types
- Function objects or callbacks

**Common mistake:** Avoiding all virtual functions for performance without measurement.

---

## 15. What is multiple inheritance, and what problems can it cause?

**Short answer:** Multiple inheritance lets a class inherit from more than one base class, but it can introduce ambiguity and diamond-shaped inheritance problems.

**Detailed answer:**
Multiple inheritance is useful for combining interface-like base classes, but inheriting implementation from multiple bases can become complex.

**Diamond problem:**

```cpp
class Animal {};
class Mammal : public Animal {};
class WingedAnimal : public Animal {};
class Bat : public Mammal, public WingedAnimal {};
```

`Bat` contains two separate `Animal` subobjects unless virtual inheritance is used.

**Virtual inheritance:**

```cpp
class Mammal : virtual public Animal {};
class WingedAnimal : virtual public Animal {};
```

**Interview point:** Multiple inheritance is generally safer for pure interfaces than for shared implementation.

**Common mistake:** Using multiple inheritance where composition or smaller interfaces would be clearer.

---

## 16. What is the difference between overriding and overloading?

**Short answer:** Overriding replaces a virtual base-class function in a derived class. Overloading defines multiple functions with the same name but different parameter lists.

**Detailed answer:**
Overriding is a runtime polymorphism concept. The derived function must match the base virtual function's signature closely enough to override it. Overloading is a compile-time name resolution concept and can happen within the same scope or across class scopes.

**Example:**

```cpp
class Base {
public:
    virtual void print(int value) const;
};

class Derived : public Base {
public:
    void print(int value) const override; // overrides Base::print
    void print(double value) const;       // overloads print
};
```

**Interview point:** Always use `override` for derived virtual functions. It catches mistakes such as missing `const`, wrong parameter types, or accidental overloads.

**Common mistake:** Thinking a same-named function automatically overrides a base function. If the signature differs, it may hide the base overload instead.

---

## 17. What is name hiding in class inheritance?

**Short answer:** A derived class declaration with the same name as a base member can hide all base overloads with that name.

**Detailed answer:**
In C++, name lookup happens before overload resolution. If a derived class declares a function named `print`, base-class functions named `print` may be hidden, even if their parameter lists are different.

**Example:**

```cpp
class Base {
public:
    void print(int value);
    void print(const char* text);
};

class Derived : public Base {
public:
    void print(double value);
};
```

Here, `Derived::print(double)` hides both `Base::print` overloads for ordinary lookup through `Derived`.

Use a `using` declaration to bring base overloads into the derived class scope.

```cpp
class BetterDerived : public Base {
public:
    using Base::print;
    void print(double value);
};
```

**Interview point:** Name hiding is a common source of surprising overload behavior in inheritance hierarchies.

**Common mistake:** Assuming overload resolution considers base and derived overloads equally without checking name lookup rules.

---

## 18. What is the Non-Virtual Interface pattern?

**Short answer:** The Non-Virtual Interface pattern exposes a public non-virtual function that calls private or protected virtual customization points.

**Detailed answer:**
NVI lets the base class control invariants, validation, locking, logging, or error handling around behavior customized by derived classes. Callers use the stable public function, while derived classes override a smaller internal function.

**Example:**

```cpp
class Processor {
public:
    void process() {
        beforeProcess();
        doProcess();
        afterProcess();
    }

private:
    virtual void doProcess() = 0;

    void beforeProcess() {}
    void afterProcess() {}
};
```

**Interview point:** NVI is useful when the base class must preserve an algorithm's structure or enforce preconditions and postconditions.

**Common mistake:** Making every customization point public and virtual, which gives derived classes and callers too many ways to bypass invariants.

---

## 19. What is object layout, and why does it matter for OOP?

**Short answer:** Object layout is how a C++ object and its subobjects are arranged in memory.

**Detailed answer:**
C++ object layout is affected by data members, base classes, padding, alignment, virtual functions, and multiple inheritance. Polymorphic classes usually contain implementation metadata such as a vptr, although the exact layout is implementation-defined.

**Example:**

```cpp
class Base {
public:
    virtual ~Base() = default;

private:
    int id_ = 0;
};

class Derived : public Base {
private:
    double value_ = 0.0;
};
```

A `Derived` object contains a `Base` subobject plus its own members. Padding may be added to satisfy alignment requirements.

**Interview point:** Object layout matters for ABI compatibility, serialization, cache behavior, and unsafe casts. Do not assume a portable binary layout for ordinary C++ classes.

**Common mistake:** Using `memcpy` or network I/O directly on non-trivial C++ objects and assuming the bytes are portable.

---

## 20. What is the difference between dynamic polymorphism and static polymorphism?

**Short answer:** Dynamic polymorphism uses virtual dispatch at runtime. Static polymorphism uses templates, overloads, or concepts resolved at compile time.

**Detailed answer:**
Dynamic polymorphism is useful when the exact runtime type may vary behind a stable interface. Static polymorphism is useful when the set of types is known at compile time and performance, inlining, or type-specific optimization matters.

**Dynamic example:**

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
};

void printArea(const Shape& shape) {
    std::cout << shape.area() << '\n';
}
```

**Static example:**

```cpp
template <typename Shape>
void printArea(const Shape& shape) {
    std::cout << shape.area() << '\n';
}
```

**Tradeoff:**
Dynamic polymorphism gives runtime flexibility and stable interfaces. Static polymorphism can be faster and more type-safe, but may increase compile times and expose implementation details in headers.

**Common mistake:** Treating templates and virtual functions as interchangeable. They solve related but different design problems.

---

## 21. Why should polymorphic base classes usually have virtual destructors?

**Short answer:** A virtual destructor ensures that deleting a derived object through a base pointer destroys the whole object correctly.

**Detailed answer:**
If a class is meant to be used polymorphically, callers may own derived objects through base-class pointers. Without a virtual destructor, `delete basePointer;` where the pointer actually refers to a derived object has undefined behavior.

A virtual destructor lets destruction dispatch correctly: first the derived destructor runs, then base destructors run.

**Bad example:**

```cpp
class Base {
public:
    virtual void run() = 0;
};

class Derived : public Base {
public:
    ~Derived() {
        // release derived resources
    }

    void run() override {}
};

Base* p = new Derived();
delete p; // undefined behavior: Base destructor is not virtual
```

**Better example:**

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual void run() = 0;
};
```

**Interview point:** If a class has virtual functions and objects may be deleted through a base pointer, the base destructor should be virtual.

**Common mistake:** Adding virtual functions to a base class but leaving the destructor non-virtual.

---

## 22. What does `final` mean for classes and virtual functions?

**Short answer:** `final` prevents further overriding of a virtual function or further inheritance from a class.

**Detailed answer:**
`final` communicates that an inheritance point is intentionally closed. On a virtual function, it means derived classes cannot override that function again. On a class, it means no class can derive from it.

This can make designs clearer and may help optimization because the compiler has stronger information about possible overrides.

**Example:**

```cpp
class Base {
public:
    virtual ~Base() = default;
    virtual void process();
};

class Derived final : public Base {
public:
    void process() override final;
};
```

Here, `Derived` cannot be used as a base class, and `process` cannot be overridden further.

**Interview point:** `final` is useful when extension would break invariants or when a hierarchy has a deliberate endpoint.

**Common mistake:** Using inheritance by default without deciding which classes and functions are intended extension points.

---

## 23. What is downcasting, and why should it be used carefully?

**Short answer:** Downcasting converts a base-class pointer or reference to a derived-class pointer or reference. It is risky if the actual object is not of that derived type.

**Detailed answer:**
Downcasting often appears when code has a base interface but needs derived-specific behavior. In C++, `dynamic_cast` checks the runtime type for polymorphic classes and returns `nullptr` for failed pointer casts or throws `std::bad_cast` for failed reference casts.

`static_cast` can also downcast, but it does not check the runtime type. If the object is not actually the target derived type, using the result is undefined behavior.

**Example:**

```cpp
class Shape {
public:
    virtual ~Shape() = default;
};

class Circle : public Shape {
public:
    double radius() const { return 1.0; }
};

void inspect(Shape* shape) {
    if (auto* circle = dynamic_cast<Circle*>(shape)) {
        double r = circle->radius();
    }
}
```

**Design note:**
Frequent downcasting may indicate the base interface is missing needed behavior, or that the design should use visitors, variants, composition, or separate interfaces.

**Interview point:** `dynamic_cast` is safer than unchecked downcasts, but needing it often is a design smell.

**Common mistake:** Using `static_cast<Derived*>` on a base pointer just because the programmer expects the runtime type to match.

---

## 24. What are covariant return types in virtual functions?

**Short answer:** Covariant return types allow an overriding virtual function to return a more derived pointer or reference type than the base function returns.

**Detailed answer:**
C++ allows covariance for virtual function return types when the returns are pointers or references to classes in the same inheritance hierarchy. This is useful for clone-like APIs and fluent interfaces where a derived override can return a more specific type.

**Example:**

```cpp
class Animal {
public:
    virtual ~Animal() = default;
    virtual Animal* clone() const = 0;
};

class Dog : public Animal {
public:
    Dog* clone() const override {
        return new Dog(*this);
    }
};
```

`Dog::clone` overrides `Animal::clone` even though it returns `Dog*`, because `Dog*` is more specific than `Animal*`.

**Modern note:**
For ownership, prefer smart pointers, but smart pointer covariance does not work the same way because `std::unique_ptr<Dog>` is not an overriding return type for `std::unique_ptr<Animal>`.

```cpp
class Animal2 {
public:
    virtual ~Animal2() = default;
    virtual std::unique_ptr<Animal2> clone() const = 0;
};
```

**Interview point:** Covariance applies to raw pointers and references in virtual returns, not arbitrary wrapper types.

**Common mistake:** Expecting `std::unique_ptr<Derived>` to covariantly override `std::unique_ptr<Base>`.

---

## 25. How should class invariants influence OOP design?

**Short answer:** Class invariants are conditions that should remain true for every valid object, and good OOP design protects them through encapsulation and controlled mutation.

**Detailed answer:**
An invariant is a rule that defines a valid object state. For example, a `Date` object should not contain month 99, and a `BankAccount` balance may have rules about allowed overdraft. Constructors should establish invariants, and public member functions should preserve them.

Encapsulation is useful because it prevents outside code from directly putting the object into an invalid state.

**Example:**

```cpp
#include <stdexcept>

class Percentage {
public:
    explicit Percentage(int value) : value_(value) {
        if (value < 0 || value > 100) {
            throw std::out_of_range("percentage");
        }
    }

    int value() const {
        return value_;
    }

private:
    int value_;
};
```

Here, users cannot modify `value_` directly, so the class can keep its invariant after construction.

**Interview point:** Encapsulation is not just hiding data; it is protecting valid object state and simplifying reasoning about behavior.

**Common mistake:** Making all data members public and then relying on every caller to preserve the class rules manually.

---

## 26. What is the difference between an interface and an abstract base class in C++?

**Short answer:** C++ has no separate `interface` keyword; interface-like designs are usually abstract base classes with pure virtual functions and little or no state.

**Detailed answer:**
An abstract base class contains at least one pure virtual function. It may also contain shared implementation, state, protected helpers, or non-virtual public functions. An interface-like class usually contains only behavior contracts and a virtual destructor.

**Example:**

```cpp
class Drawable {
public:
    virtual ~Drawable() = default;
    virtual void draw() const = 0;
};
```

Adding data members or reusable behavior can be useful, but it also couples derived classes more tightly to the base.

**Interview point:** In C++, interface-like classes are a convention, not a separate language feature.

**Common mistake:** Putting shared mutable state into a base class just because several derived classes currently need it.

---

## 27. What is the fragile base class problem?

**Short answer:** The fragile base class problem happens when changes to a base class unintentionally break derived classes.

**Detailed answer:**
Inheritance creates tight coupling between base and derived classes. A base-class change may alter virtual call order, invariants, protected member expectations, or assumptions that derived classes relied on. This is especially risky when derived classes exist outside the base class author's control.

**Example scenario:**
A base class adds a new call to a virtual function inside a public method. A derived class override may now run at a time when its assumptions are not valid.

**Safer design techniques:**
- Prefer private data over protected data.
- Keep virtual customization points small and documented.
- Use NVI to control invariants.
- Prefer composition when reuse is the goal.

**Interview point:** Inheritance is not just code reuse; it is a long-term contract between base and derived classes.

**Common mistake:** Treating protected members as harmless implementation details. Derived classes can become dependent on them.

---

## 28. What is the difference between public, protected, and private inheritance?

**Short answer:** Public inheritance models an `is-a` relationship, while protected and private inheritance model implementation reuse with restricted base access.

**Detailed answer:**
With public inheritance, public base members remain public through the derived type, and clients can treat the derived object as a base object. With private inheritance, public and protected base members become private in the derived class. Protected inheritance makes them protected.

**Example:**

```cpp
class Engine {
public:
    void start();
};

class Car : private Engine {
public:
    void drive() {
        start();
    }
};
```

`Car` uses `Engine` as an implementation detail, but users of `Car` cannot call `start()` through the inheritance relationship.

**Interview point:** If the relationship is not substitutability, composition is usually clearer than private inheritance.

**Common mistake:** Using public inheritance for code reuse when the derived type should not be usable as the base type.

---

## 29. What is protected data, and why is it often discouraged?

**Short answer:** Protected data is directly accessible to derived classes, but it couples derived classes to base-class representation.

**Detailed answer:**
Protected member functions can be useful extension points. Protected data members are riskier because derived classes can depend on layout, meaning, and update rules of base internals. This makes base-class changes harder and can let derived classes violate invariants.

**Risky example:**

```cpp
class Account {
protected:
    int balance_ = 0;
};
```

Any derived class can modify `balance_` directly, bypassing validation.

**Better design:**
Expose protected or public operations that preserve invariants.

```cpp
class Account {
protected:
    void applyDelta(int amount);
private:
    int balance_ = 0;
};
```

**Interview point:** Encapsulation matters inside inheritance hierarchies too.

**Common mistake:** Making data protected because private feels inconvenient for derived classes.

---

## 30. What is substitutability in object-oriented design?

**Short answer:** Substitutability means code using a base type should work correctly with any derived type without knowing which derived type it received.

**Detailed answer:**
Substitutability is the practical heart of public inheritance. A derived class should preserve the expectations, invariants, and behavioral contract of the base class. If a derived class weakens guarantees or surprises callers, public inheritance may be the wrong design.

**Example problem:**

```cpp
class Bird {
public:
    virtual void fly() = 0;
};

class Penguin : public Bird {
public:
    void fly() override; // awkward: penguins cannot fly
};
```

A better model might separate `Bird` from `FlyingAnimal` or use composition for capabilities.

**Interview point:** The question is not whether a derived class shares some properties with a base class, but whether it can safely replace it in client code.

**Common mistake:** Modeling taxonomies from the real world without considering behavior expected by the API.

---

## 31. What is dependency inversion in OOP?

**Short answer:** Dependency inversion means high-level code depends on abstractions rather than concrete low-level implementations.

**Detailed answer:**
Instead of constructing or hard-coding a concrete dependency, a class can accept an interface-like abstraction. This improves testability, replacement, and separation of concerns. However, unnecessary abstraction can make code harder to follow.

**Example:**

```cpp
class Logger {
public:
    virtual ~Logger() = default;
    virtual void write(std::string_view message) = 0;
};

class Service {
public:
    explicit Service(Logger& logger) : logger_(logger) {}

private:
    Logger& logger_;
};
```

**Interview point:** Dependency inversion is most useful at architectural boundaries, not necessarily inside every small helper.

**Common mistake:** Creating interfaces for every class before there is a real need for substitution.

---

## 32. What is type erasure, and how is it related to OOP?

**Short answer:** Type erasure hides concrete types behind a uniform runtime interface without requiring callers to know the exact type.

**Detailed answer:**
Type erasure can provide runtime polymorphism without exposing inheritance in the public API. Standard examples include `std::function`, `std::any`, and many custom wrapper types. Internally, type erasure often uses virtual functions, function pointers, or small-buffer optimization.

**Example:**

```cpp
#include <functional>

std::function<int(int)> operation;
operation = [](int x) { return x + 1; };
operation = [](int x) { return x * 2; };
```

The caller uses one type, while different callable implementations are stored inside.

**Interview point:** Type erasure is useful when templates would expose too much implementation or require all code to be compiled together.

**Common mistake:** Confusing type erasure with losing type safety entirely. A good erased interface still defines safe operations.

---

## 33. What is object lifetime in inheritance hierarchies?

**Short answer:** Base subobjects are constructed before derived parts and destroyed after derived parts, which affects what virtual behavior and members are safe to use.

**Detailed answer:**
During construction, the base class is built before the derived class. During destruction, the derived destructor body runs before base destructors. Virtual dispatch is limited during construction and destruction because the most-derived parts may not yet exist or may already be destroyed.

**Example:**

```cpp
class Base {
public:
    Base() { initialize(); }
    virtual void initialize();
};

class Derived : public Base {
public:
    void initialize() override;
};
```

Calling virtual functions from constructors is usually surprising because it will not dispatch as if the full derived object were ready.

**Interview point:** Construction and destruction are special phases where normal polymorphic assumptions do not fully apply.

**Common mistake:** Calling a virtual function in a base constructor expecting derived behavior.

---

## 34. What is object identity?

**Short answer:** Object identity is the idea that an object is a specific entity with a distinct lifetime and address, not just a value.

**Detailed answer:**
Two objects may have equal values but different identities. Identity matters for mutable objects, polymorphic objects, synchronization objects, handles, and objects stored in containers where references or pointers are used.

**Example:**

```cpp
Account a{100};
Account b{100};

bool sameValue = (a.balance() == b.balance());
bool sameObject = (&a == &b); // false
```

Value types usually emphasize equality by state. Entity types usually emphasize identity and lifetime.

**Interview point:** Good design distinguishes value objects from identity-bearing objects.

**Common mistake:** Giving identity-heavy objects copy semantics without defining what copying means.

---

## 35. What is a value object in C++ OOP design?

**Short answer:** A value object is primarily defined by its state and can usually be copied, compared, and passed around like a value.

**Detailed answer:**
Examples include points, dates, durations, IDs, and small configuration objects. Value objects should maintain invariants and often work well with simple constructors, equality operators, and no inheritance.

**Example:**

```cpp
class Point {
public:
    Point(int x, int y) : x_(x), y_(y) {}

    friend bool operator==(const Point&, const Point&) = default;

private:
    int x_;
    int y_;
};
```

**Interview point:** Not every class needs inheritance or virtual functions. Many good C++ classes are simple value types.

**Common mistake:** Turning simple values into polymorphic hierarchies and making them harder to copy, compare, and reason about.

---

## 36. What is the difference between an entity object and a value object?

**Short answer:** A value object is defined by its state; an entity object is defined by identity and continuity over time.

**Detailed answer:**
Two value objects with the same state are usually interchangeable. Two entity objects may have the same state but still represent different real objects or resources. Entity objects often should not be copied casually because copying identity is ambiguous.

**Example:**

```cpp
struct Money {
    int cents;
};

class SocketConnection {
public:
    SocketConnection(const SocketConnection&) = delete;
};
```

`Money` is a value. `SocketConnection` represents a unique live connection.

**Interview point:** Copy/move behavior should follow whether the type is a value, an entity, or a unique resource owner.

**Common mistake:** Letting the compiler generate copy operations for entity objects that should have unique identity.

---

## 37. What is an anemic class, and when is it a problem?

**Short answer:** An anemic class mostly stores data with little behavior, which is a problem when important invariants and operations are scattered elsewhere.

**Detailed answer:**
Plain data structures are not inherently bad. They are useful for simple data transfer and aggregate values. The problem appears when a class has meaningful rules, but those rules are enforced inconsistently by external code instead of being owned by the class.

**Example smell:**

```cpp
struct Order {
    std::vector<Item> items;
    int total;
};
```

If many callers manually update `total`, bugs are likely. A better design may compute total or provide methods that preserve consistency.

**Interview point:** Put behavior near the data when it protects invariants or expresses domain rules.

**Common mistake:** Treating every data-only type as bad. Simple aggregates are fine when there are no hidden invariants.

---

## 38. What is the difference between static data members and instance data members?

**Short answer:** Instance data members belong to each object; static data members belong to the class as a whole.

**Detailed answer:**
Every object has its own instance members. A static data member is shared across all objects of the class. Static state can be useful for constants or shared registries, but mutable static state introduces global-state concerns and synchronization issues.

**Example:**

```cpp
class Counter {
public:
    Counter() { ++liveCount; }
    ~Counter() { --liveCount; }

    static int liveCount;

private:
    int value_ = 0;
};
```

`value_` exists per object. `liveCount` is shared.

**Interview point:** Static data members are not tied to one object's lifetime.

**Common mistake:** Using mutable static members as hidden global variables without considering thread safety or test isolation.

---

## 39. What are friend functions and friend classes?

**Short answer:** Friends are non-members or other classes granted access to private and protected members.

**Detailed answer:**
`friend` can be useful for symmetric operators, tightly coupled helper functions, or tests in limited cases. Friendship is not inherited or reciprocal. It should be used carefully because it expands the set of code that can depend on internals.

**Example:**

```cpp
class Point {
public:
    Point(int x, int y) : x_(x), y_(y) {}

    friend bool operator==(const Point& a, const Point& b) {
        return a.x_ == b.x_ && a.y_ == b.y_;
    }

private:
    int x_;
    int y_;
};
```

**Interview point:** `friend` does not break encapsulation automatically, but careless friendship can make internals part of a wider informal API.

**Common mistake:** Making large manager classes friends instead of designing a smaller public or internal interface.

---

## 40. How do you decide between OOP and procedural design in C++?

**Short answer:** Use OOP when identity, invariants, substitution, or encapsulated behavior matter; use procedural or value-based design when simple data transformations are clearer.

**Detailed answer:**
C++ supports multiple styles. OOP is useful for stable runtime interfaces, resource-owning entities, and behavior that must protect internal state. Procedural or generic designs can be better for algorithms over data, numeric code, parsing pipelines, or simple transformations.

**Guiding questions:**
- Does the type have invariants that need protection?
- Is runtime polymorphism actually needed?
- Is the data mostly transformed by free algorithms?
- Would value semantics make the code simpler?
- Will inheritance create a long-term contract worth maintaining?

**Interview point:** Senior C++ design is not about forcing everything into classes; it is about choosing the simplest model that preserves correctness and clarity.

**Common mistake:** Assuming "more object-oriented" automatically means better C++ design.

---

## 41. What is a class responsibility?

**Short answer:** A class responsibility is the specific behavior, invariant, or resource that a class is responsible for managing.

**Detailed answer:**
Good classes have clear responsibilities. A class may own a resource, enforce a domain invariant, provide a runtime interface, or represent a value. When a class mixes unrelated responsibilities, it becomes harder to test, reuse, and change.

**Example:**

```cpp
class FileReader {
public:
    explicit FileReader(std::filesystem::path path);
    std::string readAll() const;
};
```

This class should read files. If it also parses business rules, sends network requests, and updates UI state, its responsibility has become unclear.

**Interview point:** Class design starts by deciding what the class owns and what it promises.

**Common mistake:** Creating large “manager” classes that accumulate unrelated responsibilities.

---

## 42. What is representation exposure?

**Short answer:** Representation exposure happens when a class exposes internal data in a way that lets callers break its invariants.

**Detailed answer:**
Encapsulation is not only about making fields private. A class can still expose representation by returning mutable references, raw pointers, iterators, or views into internal storage without clear lifetime and mutation rules.

**Example:**

```cpp
class Team {
public:
    std::vector<Member>& members() { return members_; }

private:
    std::vector<Member> members_;
};
```

Callers can now modify the vector directly, possibly bypassing validation or consistency rules.

**Interview point:** Accessors should preserve invariants, not simply reveal private storage.

**Common mistake:** Making fields private but returning non-const references to everything.

---

## 43. What is behavioral subtyping?

**Short answer:** Behavioral subtyping means a derived type can replace a base type without violating the behavior callers expect.

**Detailed answer:**
This is the practical design meaning behind substitutability. A derived class should not strengthen preconditions, weaken postconditions, or surprise callers that use the base interface. The issue is not only syntax; it is whether behavior remains compatible.

**Example problem:**

```cpp
class Collection {
public:
    virtual void add(int value) = 0;
};

class ReadOnlyCollection : public Collection {
public:
    void add(int) override { throw std::logic_error("read-only"); }
};
```

If callers expect any `Collection` to accept `add`, the derived type violates the contract.

**Interview point:** Inheritance models an “is substitutable as” relationship, not just shared functions.

**Common mistake:** Using inheritance to reuse code when the derived behavior cannot satisfy the base contract.

---

## 44. What is the difference between interface inheritance and implementation inheritance?

**Short answer:** Interface inheritance exposes a substitutable contract; implementation inheritance reuses base code.

**Detailed answer:**
Interface inheritance is about runtime polymorphism and caller expectations. Implementation inheritance is about sharing code. Mixing the two can create fragile base classes, hidden coupling, and difficult override rules.

**Example:**

```cpp
class Drawable {
public:
    virtual ~Drawable() = default;
    virtual void draw() const = 0;
};
```

This is interface inheritance. A base class with protected helper functions and stored state may be implementation inheritance.

**Interview point:** Prefer composition for code reuse unless substitutability is truly needed.

**Common mistake:** Creating a base class only to avoid duplicating a few lines of code.

---

## 45. What is object ownership in OOP design?

**Short answer:** Object ownership defines which object or component is responsible for another object's lifetime.

**Detailed answer:**
OOP designs often contain graphs of objects. Some relationships are ownership relationships, while others are observation, borrowing, or association. The design should make ownership explicit through values, `std::unique_ptr`, references, `std::shared_ptr`, or documented non-owning pointers.

**Example:**

```cpp
class Window {
public:
    void addChild(std::unique_ptr<Widget> child);

private:
    std::vector<std::unique_ptr<Widget>> children_;
};
```

The window clearly owns its child widgets.

**Interview point:** Class relationships should distinguish “has a,” “uses a,” and “observes a.”

**Common mistake:** Storing raw pointers in object graphs without saying who deletes the objects.

---

## 46. What is a polymorphic value type?

**Short answer:** A polymorphic value type provides value-like copying while preserving the dynamic type internally.

**Detailed answer:**
Ordinary base-class copying can slice derived objects. A polymorphic value wrapper uses cloning or type erasure so callers can copy the wrapper as a value while the implementation preserves the concrete object.

**Conceptual example:**

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
};

class DrawingObject {
public:
    DrawingObject(const DrawingObject& other)
        : shape_(other.shape_->clone()) {}

private:
    std::unique_ptr<Shape> shape_;
};
```

**Interview point:** Value semantics and runtime polymorphism can coexist, but they require explicit design.

**Common mistake:** Copying polymorphic objects through base values and accidentally slicing them.

---

## 47. What is coupling between classes?

**Short answer:** Coupling is the degree to which one class depends on another class's details.

**Detailed answer:**
Some coupling is necessary, but excessive coupling makes changes ripple through a codebase. Classes can be coupled through concrete types, inheritance, friendship, shared mutable state, headers, callbacks, exceptions, and ownership assumptions.

**Example:**

```cpp
class ReportGenerator {
public:
    explicit ReportGenerator(DatabaseConnection& database);
};
```

This class is coupled to a concrete database connection. That may be fine, or an interface may be better if substitution is needed.

**Interview point:** Reduce coupling where change or testing pressure justifies it; do not abstract every dependency by default.

**Common mistake:** Measuring coupling only by the number of classes, not by how much one class knows about another's behavior and lifetime.

---

## 48. What is cohesion in class design?

**Short answer:** Cohesion is how closely a class's data and methods relate to one clear purpose.

**Detailed answer:**
A cohesive class has members that support the same responsibility. Low cohesion appears when methods operate on unrelated subsets of fields or when the class name is vague, such as `Manager`, `Helper`, or `Context`.

**Example smell:**

```cpp
class ApplicationManager {
    void parseConfig();
    void renderUi();
    void sendMetrics();
    void rotateLogs();
};
```

These responsibilities may deserve separate components.

**Interview point:** High cohesion makes invariants easier to understand and tests easier to write.

**Common mistake:** Splitting classes by technical layer only while leaving unrelated responsibilities mixed together.

---

## 49. What is a sealed hierarchy?

**Short answer:** A sealed hierarchy is an inheritance hierarchy where the set of derived types is intentionally limited.

**Detailed answer:**
C++ does not have a direct sealed-class feature like some languages, but `final`, private constructors, internal namespaces, and factory functions can limit extension. Sealed hierarchies are useful when the base class implementation assumes it knows all derived types.

**Example:**

```cpp
class Token {
public:
    virtual ~Token() = default;
};

class IdentifierToken final : public Token {};
class NumberToken final : public Token {};
```

If external users must add new token types, sealing would be the wrong choice.

**Interview point:** Open extension points require stronger contracts than closed internal hierarchies.

**Common mistake:** Exposing a base class for public inheritance without documenting what derived classes are allowed to override.

---

## 50. How do you design OOP code that is easy to test?

**Short answer:** Keep responsibilities small, make dependencies explicit, avoid hidden global state, and test behavior through stable interfaces.

**Detailed answer:**
Testable OOP code usually has clear constructors, explicit dependencies, deterministic behavior, and well-defined boundaries. Runtime polymorphism can help testing when real substitution is needed, but not every class requires an interface just for tests.

**Guidelines:**

- Prefer constructor injection for required dependencies.
- Keep side effects near boundaries.
- Avoid hidden singletons and mutable static state.
- Test invariants and observable behavior, not private implementation details.
- Use fakes or interfaces only where substitution is meaningful.

**Interview point:** Testability follows from good design; excessive mocking often signals unclear boundaries.

**Common mistake:** Adding virtual interfaces for every class instead of separating real external dependencies from ordinary implementation details.
