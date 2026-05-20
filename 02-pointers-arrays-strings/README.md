# Pointers, Arrays, and Strings Interview Questions

This topic focuses on one of the most important areas in C and C++ interviews. Strong candidates should understand not only syntax, but also memory layout, decay rules, pointer arithmetic, ownership risks, and the difference between C-style strings and C++ string abstractions.

## 1. What is pointer arithmetic?

**Short answer:** Pointer arithmetic moves by elements, not by raw bytes.

**Detailed answer:**
When you add `1` to a pointer, the address increases by `sizeof(*pointer)` bytes. This allows pointers to move across arrays naturally. For example, if `p` is an `int*`, then `p + 1` points to the next `int`, not the next byte.

**Example:**

```cpp
#include <iostream>

int main() {
    int values[] = {10, 20, 30};
    int* p = values;

    std::cout << *p << '\n';       // 10
    std::cout << *(p + 1) << '\n'; // 20
    std::cout << *(p + 2) << '\n'; // 30
}
```

**Important rule:** Pointer arithmetic is only valid within the same array object, including one position past the end. Dereferencing one-past-the-end is undefined behavior.

```cpp
int values[] = {1, 2, 3};
int* end = values + 3; // allowed: one past the last element
// *end;              // undefined behavior
```

**Common mistake:** Thinking `p + 1` always means "address plus one byte." That is only true when `p` is a `char*`, because `sizeof(char)` is 1.

---

## 2. What does it mean when an array decays to a pointer?

**Short answer:** In many expressions, an array expression is automatically converted to a pointer to its first element.

**Detailed answer:**
An array has its own type and size, but when passed to a function or used in many expressions, it decays into a pointer. This is why a function parameter written as `int arr[]` is actually treated as `int* arr`.

**Example:**

```cpp
#include <iostream>

void printSize(int arr[]) {
    std::cout << sizeof(arr) << '\n'; // size of pointer, not array
}

int main() {
    int values[5] = {1, 2, 3, 4, 5};

    std::cout << sizeof(values) << '\n'; // 5 * sizeof(int)
    printSize(values);
}
```

**How to preserve size in C++:**

```cpp
#include <array>
#include <iostream>

void printSize(const std::array<int, 5>& values) {
    std::cout << values.size() << '\n';
}
```

**Follow-up questions:**
- Why do C functions often take both a pointer and a length?
- How is `std::array` different from a raw array?
- How is `std::vector` different from both?

**Common mistake:** Assuming `sizeof(arr)` inside a function gives the number of elements. It does not when the parameter has decayed to a pointer.

---

## 3. What is the difference between `int* p`, `int p[]`, and `int (*p)[N]`?

**Short answer:** `int* p` points to an `int`; `int p[]` in a function parameter is adjusted to `int*`; `int (*p)[N]` points to an entire array of `N` integers.

**Detailed answer:**
These declarations look similar but have different meanings. Understanding them is important when working with multidimensional arrays or APIs that require exact array shapes.

**Example:**

```cpp
int values[3] = {1, 2, 3};

int* p1 = values;      // points to values[0]
int (*p2)[3] = &values; // points to the whole array
```

`p1 + 1` moves by one `int`. `p2 + 1` moves by one entire array of 3 integers.

```cpp
std::cout << *(p1 + 1) << '\n';    // values[1]
std::cout << (*p2)[1] << '\n';     // values[1]
```

**Why interviewers ask this:** It reveals whether the candidate understands C declarations, array decay, and memory layout.

**Common mistake:** Treating a pointer to the first element and a pointer to the entire array as the same type. Their address value may be numerically equal, but their types and arithmetic behavior differ.

---

## 4. What is a dangling pointer?

**Short answer:** A dangling pointer points to memory that is no longer valid.

**Detailed answer:**
Dangling pointers commonly happen after freeing heap memory, returning the address of a local variable, or keeping a pointer/reference to an object that has gone out of scope. Dereferencing a dangling pointer is undefined behavior.

**Bad example:**

```cpp
int* createBadPointer() {
    int value = 42;
    return &value; // bad: value is destroyed when the function returns
}
```

**Another bad example:**

```cpp
int* p = new int(10);
delete p;

// *p = 20; // undefined behavior: p is dangling
```

**Better C++ approach:**

```cpp
#include <memory>

std::unique_ptr<int> createValue() {
    return std::make_unique<int>(42);
}
```

**Common mistake:** Setting one pointer to `nullptr` after `delete` does not fix other copies of the same pointer. Those copies still dangle.

---

## 5. What is the difference between a null pointer and an uninitialized pointer?

**Short answer:** A null pointer intentionally points to nothing. An uninitialized pointer has an indeterminate value.

**Detailed answer:**
A null pointer is a known invalid pointer value that can be checked before use. An uninitialized pointer contains garbage. Reading or dereferencing it can lead to undefined behavior.

**Example:**

```cpp
int* p1 = nullptr; // safe known state
int* p2;           // uninitialized, dangerous

if (p1 == nullptr) {
    // handle no object
}
```

**C vs C++ note:**
In modern C++, prefer `nullptr` over `NULL` or `0` because `nullptr` has type `std::nullptr_t` and avoids overload ambiguity.

```cpp
void f(int);
void f(int*);

f(nullptr); // calls f(int*)
// f(NULL); // may be ambiguous or call the wrong overload depending on definition
```

**Common mistake:** Believing a pointer is automatically initialized to null. Local raw pointers are not automatically initialized.

---

## 6. What is a C-style string?

**Short answer:** A C-style string is a sequence of characters terminated by a null character `\0`.

**Detailed answer:**
C-style strings are represented as `char*` or `const char*`. Functions such as `strlen`, `strcpy`, and `strcmp` rely on the null terminator to know where the string ends. If the null terminator is missing, these functions may read past the valid memory range.

**Example:**

```cpp
#include <cstring>
#include <iostream>

int main() {
    char text[] = "hello";

    std::cout << sizeof(text) << '\n'; // 6: h e l l o \0
    std::cout << strlen(text) << '\n'; // 5: characters before \0
}
```

**Important distinction:**
- `sizeof(text)` gives the array size in bytes when `text` is an array in scope.
- `strlen(text)` counts characters until the first `\0`.

**Common mistake:** Forgetting space for the null terminator when manually allocating a string buffer.

```cpp
char buffer[5] = "hello"; // error: needs 6 chars including \0
```

---

## 7. What is the difference between `char*`, `const char*`, and `char[]`?

**Short answer:** `char*` is a mutable pointer to characters, `const char*` points to read-only characters, and `char[]` is an array of characters.

**Detailed answer:**
String literals in C++ have type `const char[N]`, so they should be stored in `const char*` or an array if a mutable copy is needed.

**Example:**

```cpp
const char* text = "hello"; // points to a string literal; do not modify

char copy[] = "hello";      // mutable array copy
copy[0] = 'H';
```

**Bad example:**

```cpp
char* p = "hello"; // invalid in modern C++; dangerous in old C-style code
// p[0] = 'H';     // undefined behavior if it points to a string literal
```

**Interview tip:** If the data should not be modified, use `const char*` or `std::string_view` in modern C++.

**Common mistake:** Thinking `char* p = "hello"` creates a writable character array. It does not.

---

## 8. When should you use `std::string` instead of C-style strings?

**Short answer:** Use `std::string` for owning and manipulating text in C++ unless you specifically need a C-compatible API.

**Detailed answer:**
`std::string` manages memory automatically, knows its size, supports safe copying and assignment, and integrates with the C++ standard library. C-style strings are useful for low-level code, embedded constraints, or C API boundaries, but they are easier to misuse.

**Example:**

```cpp
#include <iostream>
#include <string>

int main() {
    std::string name = "Ada";
    name += " Lovelace";

    std::cout << name << '\n';
    std::cout << name.size() << '\n';
}
```

**C API boundary:**

```cpp
std::string fileName = "data.txt";
FILE* file = fopen(fileName.c_str(), "r");
```

**Common mistake:** Returning `c_str()` from a local `std::string`.

```cpp
const char* bad() {
    std::string value = "temporary";
    return value.c_str(); // dangling pointer after value is destroyed
}
```

---

## 9. What is `std::string_view`, and what risk does it introduce?

**Short answer:** `std::string_view` is a non-owning view of character data. Its main risk is dangling if the referenced data no longer exists.

**Detailed answer:**
`std::string_view` is useful for read-only function parameters because it can refer to a `std::string`, string literal, or character buffer without copying. However, it does not own the data, so the caller must ensure the data outlives the view.

**Example:**

```cpp
#include <iostream>
#include <string_view>

void print(std::string_view text) {
    std::cout << text << '\n';
}

int main() {
    std::string name = "Grace";
    print(name);
    print("Hopper");
}
```

**Bad example:**

```cpp
#include <string>
#include <string_view>

std::string_view badView() {
    std::string local = "temporary";
    return local; // dangling view
}
```

**Common mistake:** Treating `std::string_view` like a safer `std::string`. It is efficient, but it has lifetime risks because it does not own memory.

---

## 10. How do multidimensional arrays work in C/C++?

**Short answer:** A multidimensional array is stored contiguously in row-major order.

**Detailed answer:**
For `int matrix[2][3]`, memory contains 6 integers in order: `matrix[0][0]`, `matrix[0][1]`, `matrix[0][2]`, `matrix[1][0]`, `matrix[1][1]`, `matrix[1][2]`.

**Example:**

```cpp
#include <iostream>

int main() {
    int matrix[2][3] = {
        {1, 2, 3},
        {4, 5, 6}
    };

    std::cout << matrix[1][2] << '\n'; // 6
}
```

When passing a multidimensional array to a function, all dimensions except the first must be known so the compiler can compute offsets.

```cpp
void printMatrix(const int matrix[][3], int rows) {
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < 3; ++j) {
            std::cout << matrix[i][j] << ' ';
        }
        std::cout << '\n';
    }
}
```

**Common mistake:** Assuming `int**` is the same as `int matrix[rows][cols]`. A true 2D array is contiguous; an `int**` usually points to separate row allocations or pointers.

---

## 11. What is the difference between `const int*`, `int* const`, and `const int* const`?

**Short answer:** `const int*` points to a read-only int, `int* const` is a read-only pointer to a mutable int, and `const int* const` is a read-only pointer to a read-only int.

**Detailed answer:**
Const placement changes whether the pointer itself is const or whether the pointed-to object is const through that pointer.

**Example:**

```cpp
int a = 1;
int b = 2;

const int* p1 = &a;
// *p1 = 3; // not allowed
p1 = &b;    // allowed

int* const p2 = &a;
*p2 = 3;    // allowed
// p2 = &b; // not allowed

const int* const p3 = &a;
// *p3 = 4; // not allowed
// p3 = &b; // not allowed
```

**Interview tip:** Read declarations from right to left: `p2` is a const pointer to int.

**Common mistake:** Saying `const int*` makes the original integer immutable. It only prevents modification through that pointer.

---

## 12. What is a pointer to a function?

**Short answer:** A function pointer stores the address of a function with a specific signature.

**Detailed answer:**
Function pointers are common in C callbacks, embedded code, plugin systems, and low-level APIs. In modern C++, lambdas, `std::function`, and templates often provide more flexible alternatives, but function pointers remain important for C interoperability.

**Example:**

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}

int apply(int a, int b, int (*operation)(int, int)) {
    return operation(a, b);
}

int main() {
    std::cout << apply(2, 3, add) << '\n';
    std::cout << apply(2, 3, multiply) << '\n';
}
```

**Modern C++ alternative:**

```cpp
template <typename Operation>
int apply(int a, int b, Operation operation) {
    return operation(a, b);
}
```

**Common mistake:** Confusing a function pointer with a pointer to member function. Member function pointers have different syntax and need an object instance.

---

## 13. What is a pointer to member?

**Short answer:** A pointer to member refers to a member variable or member function of a class, not to a standalone memory address by itself.

**Detailed answer:**
Pointers to members require an object to be used. They are different from ordinary pointers because class layout, inheritance, and virtual dispatch can affect how members are accessed.

**Example:**

```cpp
#include <iostream>

struct User {
    int age;
    void printAge() const {
        std::cout << age << '\n';
    }
};

int main() {
    int User::* ageMember = &User::age;
    void (User::* printMember)() const = &User::printAge;

    User user{30};
    std::cout << user.*ageMember << '\n';
    (user.*printMember)();
}
```

For pointers to objects, use `->*` instead of `.*`.

```cpp
User* userPtr = &user;
std::cout << userPtr->*ageMember << '\n';
```

**Interview point:** Pointers to members are niche, but they reveal deep understanding of C++ type syntax.

---

## 14. Why are functions like `strcpy` dangerous, and what should be used instead?

**Short answer:** `strcpy` does not know the destination buffer size, so it can overflow the buffer if the source is too large.

**Detailed answer:**
C string functions that rely only on null terminators can write or read past valid memory when inputs are malformed or buffers are too small. This can cause crashes, data corruption, or security vulnerabilities.

**Bad example:**

```cpp
char destination[8];
strcpy(destination, "this is too long"); // buffer overflow
```

**Safer C++ approach:**

```cpp
std::string destination = "this is too long";
```

**When using C APIs:**
Prefer APIs that accept explicit buffer sizes and check return values carefully.

```cpp
char buffer[8];
snprintf(buffer, sizeof(buffer), "%s", "hello");
```

**Interview point:** Safer code passes sizes with buffers or uses owning abstractions such as `std::string`, `std::vector`, and `std::array`.

**Common mistake:** Using `strncpy` blindly. It may not null-terminate the destination if the source is too long.

---

## 15. What is `std::span`, and how does it help with arrays?

**Short answer:** `std::span` is a non-owning view over a contiguous sequence of objects.

**Detailed answer:**
`std::span` carries both a pointer and a size, which makes it safer and clearer than passing a raw pointer and length separately. It can view arrays, `std::array`, `std::vector`, or other contiguous storage.

**Example:**

```cpp
#include <span>
#include <vector>

int sum(std::span<const int> values) {
    int total = 0;
    for (int value : values) {
        total += value;
    }
    return total;
}

int main() {
    int raw[] = {1, 2, 3};
    std::vector<int> vec = {4, 5, 6};

    sum(raw);
    sum(vec);
}
```

**Important lifetime rule:**
`std::span` does not own the data. The viewed storage must outlive the span.

**Common mistake:** Returning a span to a local array or temporary vector, which creates a dangling view.

---

## 16. What is pointer provenance and why does object lifetime matter?

**Short answer:** A pointer is only valid for objects whose lifetime has begun and whose storage it is allowed to access.

**Detailed answer:**
C and C++ do not treat pointers as just integer addresses. A pointer is tied to the object and storage it came from. Accessing storage through a pointer after the object lifetime has ended, before an object lifetime begins, or outside the allowed object can cause undefined behavior even if the numeric address looks reasonable.

**Example:**

```cpp
#include <new>
#include <string>

alignas(std::string) unsigned char storage[sizeof(std::string)];

std::string* text = new (storage) std::string("hello");
text->~basic_string();

// text->size(); // undefined behavior: object lifetime has ended
```

**Interview point:** Low-level code must reason about both storage and object lifetime. Allocation alone does not always mean a live object exists there.

**Common mistake:** Assuming that if an address still contains old bytes, using the old pointer is safe.

---

## 17. What is `restrict` in C, and does C++ have it?

**Short answer:** `restrict` is a C keyword that promises a pointer is the only way to access an object during its lifetime in that scope. Standard C++ does not have `restrict`.

**Detailed answer:**
`restrict` helps the compiler optimize because it can assume two restricted pointers do not alias the same object. This matters in numeric loops and low-level memory processing. Some C++ compilers provide extensions such as `__restrict`, but they are not portable standard C++.

**C example:**

```c
void add_arrays(int* restrict out, const int* restrict a, const int* restrict b, int n) {
    for (int i = 0; i < n; ++i) {
        out[i] = a[i] + b[i];
    }
}
```

The promise is only valid if `out`, `a`, and `b` do not overlap in a way that violates the contract.

**Interview point:** `restrict` is an optimization contract. Breaking that contract can produce incorrect optimized code.

**Common mistake:** Treating `restrict` as a runtime check. It is not checked at runtime.

---

## 18. What is the difference between `memcpy`, `memmove`, and `strcpy`?

**Short answer:** `memcpy` copies raw bytes without overlap support, `memmove` copies raw bytes safely with overlap, and `strcpy` copies a null-terminated C string.

**Detailed answer:**
`memcpy` and `memmove` copy a specified number of bytes. They do not care about null terminators. `strcpy` copies until it sees `\0`, so the destination must be large enough and the source must be a valid C string.

**Example:**

```cpp
#include <cstring>

char buffer[] = "abcdef";

std::memmove(buffer + 1, buffer, 3); // overlap-safe
```

Using `memcpy` for overlapping ranges is undefined behavior.

```cpp
// std::memcpy(buffer + 1, buffer, 3); // wrong if ranges overlap
```

**Interview point:** For C++ objects, raw byte copying is only safe for appropriate types and situations. Prefer constructors, assignment, and containers for ordinary objects.

**Common mistake:** Using `strcpy` for binary data or using `memcpy` on non-trivially copyable C++ objects.

---

## 19. What is the difference between `std::array`, raw arrays, and `std::vector`?

**Short answer:** A raw array is a built-in fixed-size array, `std::array` is a fixed-size standard-library wrapper, and `std::vector` is a dynamically sized owning container.

**Detailed answer:**
Raw arrays are lightweight but decay to pointers in many contexts and have limited interface support. `std::array<T, N>` keeps the size as part of the type and provides container-style functions such as `size()`, iterators, and assignment. `std::vector<T>` owns a resizable dynamic array.

**Example:**

```cpp
#include <array>
#include <vector>

int raw[3] = {1, 2, 3};
std::array<int, 3> fixed = {1, 2, 3};
std::vector<int> dynamic = {1, 2, 3};
```

**Interview point:** Prefer `std::array` for fixed-size arrays in modern C++ APIs and `std::vector` when size changes at runtime.

**Common mistake:** Passing raw arrays to functions and expecting size information to be preserved automatically.

---

## 20. How should you choose between `std::string`, `std::string_view`, and `const char*` parameters?

**Short answer:** Use `std::string` when ownership is needed, `std::string_view` for non-owning read-only text, and `const char*` mainly for C API boundaries or null-terminated requirements.

**Detailed answer:**
A function that stores text beyond the call should usually take or create an owning `std::string`. A function that only reads text during the call can often take `std::string_view`. A function that calls a C API or requires null termination may need `const char*`.

**Example:**

```cpp
#include <string>
#include <string_view>

void printName(std::string_view name);     // read-only, no ownership
void setName(std::string name);            // stores ownership
void openFile(const char* path);           // C API boundary or null-terminated requirement
```

**Important warning:**
`std::string_view` does not guarantee null termination. Passing `view.data()` to a C function expecting a C string can read past the view.

**Interview point:** Parameter type should communicate lifetime and ownership expectations.

**Common mistake:** Returning or storing a `std::string_view` that refers to a temporary string.

---

## 21. What is strict aliasing, and how does it affect pointers?

**Short answer:** Strict aliasing allows the compiler to assume that pointers to unrelated types do not refer to the same object.

**Detailed answer:**
C and C++ have rules about which pointer types may be used to access an object. Accessing an object through an incompatible pointer type can be undefined behavior. The optimizer uses these rules to reorder or eliminate memory operations.

**Bad example:**

```cpp
float value = 1.0f;
int* bits = reinterpret_cast<int*>(&value);
int raw = *bits; // undefined behavior
```

Use `std::bit_cast` in modern C++ when you need to copy object representation into another trivially copyable type.

```cpp
#include <bit>

int raw = std::bit_cast<int>(value);
```

**Interview point:** `reinterpret_cast` changes the pointer type, but it does not automatically make accessing through that pointer legal.

**Common mistake:** Treating pointer casts as a way to bypass the language's object model safely.

---

## 22. What is the difference between pointer equality and object equality?

**Short answer:** Pointer equality compares addresses. Object equality compares values or logical state.

**Detailed answer:**
Two pointers are equal when they point to the same object or both are null. Two different objects may have equal values but different addresses. Conversely, two pointers to the same object may compare equal even if the object's value changes.

**Example:**

```cpp
int a = 42;
int b = 42;

int* pa = &a;
int* pb = &b;

bool sameAddress = (pa == pb); // false
bool sameValue = (*pa == *pb); // true
```

**Interview point:** Comparing C strings with `==` compares pointer addresses, not text content.

```cpp
const char* x = "hello";
const char* y = "hello";
// x == y is not a portable string-content comparison
```

Use `std::strcmp`, `std::string`, or `std::string_view` for text comparison.

**Common mistake:** Using `==` on `char*` and expecting lexical string comparison.

---

## 23. What is pointer invalidation in containers?

**Short answer:** Pointer invalidation happens when a container operation makes existing pointers or references to elements no longer valid.

**Detailed answer:**
Containers can move or destroy elements during insertion, erasure, or reallocation. A pointer to a `std::vector` element may become dangling after `push_back` if the vector grows beyond its capacity.

**Example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3};
int* p = &values[0];

values.push_back(4); // may reallocate
// p may now be dangling
```

**Interview point:** Pointer stability depends on the container and operation. `std::vector` offers contiguous storage but weaker pointer stability under growth; node-based containers have different invalidation rules.

**Common mistake:** Storing raw pointers to vector elements across operations that may reallocate.

---

## 24. What is the difference between `char`, `signed char`, and `unsigned char`?

**Short answer:** `char` is a distinct character type whose signedness is implementation-defined; `signed char` and `unsigned char` are explicitly signed or unsigned integer character types.

**Detailed answer:**
All three are distinct types. `char` is used for character data. `signed char` and `unsigned char` are often used for small integer values or raw bytes. When indexing byte frequency tables, cast to `unsigned char` to avoid negative indices if `char` is signed.

**Example:**

```cpp
#include <array>

std::array<int, 256> counts{};
char ch = '\xff';
++counts[static_cast<unsigned char>(ch)];
```

**Interview point:** Character signedness can affect portability, especially in parsing, hashing, and byte-processing code.

**Common mistake:** Using plain `char` directly as an array index for all possible byte values.

---

## 25. What is sentinel-terminated data?

**Short answer:** Sentinel-terminated data uses a special value to mark the end instead of storing a separate length.

**Detailed answer:**
C strings are the classic example: a sequence of `char` values ends at the first `\0`. This makes simple strings compact but requires scanning to find the length and is unsafe if the terminator is missing.

**Example:**

```cpp
const char* text = "hello"; // stored with a trailing '\0'
std::size_t length = std::strlen(text);
```

Length-based views such as `std::string_view` can represent substrings containing embedded null characters.

```cpp
std::string_view view("a\0b", 3); // length is 3
```

**Interview point:** Know whether an API expects sentinel termination or an explicit length. Confusing the two causes truncation bugs or out-of-bounds reads.

**Common mistake:** Passing `std::string_view::data()` to a C API that expects a null-terminated string.

---

## 26. What is the difference between a pointer to `const` and a `const` pointer?

**Short answer:** A pointer to `const` cannot modify the pointed-to object through that pointer; a `const` pointer cannot be reseated to point somewhere else.

**Detailed answer:**
The position of `const` matters. Read pointer declarations from right to left or inside out. `const int*` means the `int` is const through the pointer. `int* const` means the pointer itself is const. `const int* const` means both are const.

**Example:**

```cpp
int a = 1;
int b = 2;

const int* p1 = &a; // pointer to const int
p1 = &b;            // OK
// *p1 = 3;         // error

int* const p2 = &a; // const pointer to int
*p2 = 3;            // OK
// p2 = &b;         // error

const int* const p3 = &a; // const pointer to const int
```

**Interview point:** `const` on pointers controls either the pointer object, the pointee access, or both.

**Common mistake:** Saying `const int*` means the original object is physically immutable. It only prevents modification through that access path.

---

## 27. What is a pointer to `void`, and when is it useful?

**Short answer:** `void*` is a generic object pointer type that can hold the address of an object without knowing its concrete type.

**Detailed answer:**
C APIs often use `void*` for generic data, callbacks, allocators, and raw memory. A `void*` cannot be dereferenced directly because the compiler does not know the object type or size. It must be converted back to an appropriate pointer type first.

**Example:**

```cpp
#include <cstdlib>

void* raw = std::malloc(sizeof(int));
int* value = static_cast<int*>(raw);
*value = 42;
std::free(raw);
```

In modern C++, templates, type-erased wrappers, and safer abstractions are often better than passing `void*` through application code.

**Interview point:** `void*` removes static type information. Use it mainly at low-level or C interoperability boundaries.

**Common mistake:** Treating `void*` as if it preserves object type automatically. It only preserves an address.

---

## 28. What is the difference between `void*`, `char*`, and `std::byte*` for raw memory?

**Short answer:** `void*` is an untyped address, `char*` can inspect object representation as bytes, and `std::byte*` represents raw bytes without implying character or arithmetic meaning.

**Detailed answer:**
Raw memory work needs precise vocabulary. `void*` is useful for passing around untyped storage but cannot be dereferenced. Character types can access object representations byte by byte. `std::byte` is a C++17 type for raw memory that avoids treating bytes as text or small integers.

**Example:**

```cpp
#include <cstddef>
#include <cstring>

int value = 0x12345678;
unsigned char bytes[sizeof(value)];
std::memcpy(bytes, &value, sizeof(value));
```

`std::byte` is often clearer for buffers that are not text.

```cpp
#include <array>
#include <cstddef>

std::array<std::byte, 1024> buffer{};
```

**Interview point:** Use text types for text and byte types for raw storage. Mixing them carelessly causes aliasing, signedness, and readability problems.

**Common mistake:** Using `char*` for every raw buffer and later confusing binary data with null-terminated strings.

---

## 29. What is pointer alignment, and why does it matter?

**Short answer:** Pointer alignment means an object's address must satisfy the alignment requirement of its type.

**Detailed answer:**
Many types must be stored at addresses that are multiples of a specific alignment. Accessing an object through a misaligned pointer can be slower on some platforms and undefined behavior on others. Alignment matters for custom allocators, binary parsing, placement new, SIMD, and hardware interfaces.

**Example:**

```cpp
#include <cstdint>

alignas(16) int values[4];
static_assert(alignof(decltype(values)) >= alignof(int));
```

Unsafe binary parsing example:

```cpp
char buffer[sizeof(int)];
int* p = reinterpret_cast<int*>(buffer); // may be misaligned and object lifetime may be wrong
```

Use `std::memcpy` into a properly aligned object for portable parsing.

**Interview point:** Correct pointer type is not enough; the address must also be suitably aligned and refer to a live object.

**Common mistake:** Casting arbitrary network or file bytes directly to a struct pointer.

---

## 30. What is pointer provenance?

**Short answer:** Pointer provenance describes which object a pointer is associated with and whether it can legally be used to access that object.

**Detailed answer:**
Modern C and C++ object models do not treat pointers as just integer addresses. A pointer's validity depends on how it was obtained, the lifetime of the object, and whether it still points within the same object or one-past the end. Converting pointers to integers and back, or doing arithmetic outside an object, can break assumptions the optimizer relies on.

**Example:**

```cpp
int a[4] = {1, 2, 3, 4};
int* p = a + 2;     // points within the same array object
int* end = a + 4;   // one-past-the-end pointer, valid for comparison, not dereference
```

**Interview point:** Pointer arithmetic is defined within arrays and one-past the end. Arbitrary address arithmetic is not portable C++ object access.

**Common mistake:** Thinking that if a numeric address looks correct, dereferencing it must be legal.

---

## 31. What is a one-past-the-end pointer?

**Short answer:** A one-past-the-end pointer points just after the last element of an array and may be used for comparison, but not dereferenced.

**Detailed answer:**
C and C++ allow forming a pointer one element past an array. This enables half-open ranges such as `[begin, end)`. The pointer is useful as a sentinel but does not point to an object.

**Example:**

```cpp
int values[] = {1, 2, 3};
int* begin = values;
int* end = values + 3;

for (int* p = begin; p != end; ++p) {
    // *p is valid here
}

// *end is undefined behavior
```

**Interview point:** Many standard library iterator ranges use the same half-open idea as pointer ranges.

**Common mistake:** Dereferencing the end pointer after a loop or using `<= end` instead of `< end` style logic.

---

## 32. What is the difference between an array parameter and a pointer parameter?

**Short answer:** In function parameters, array syntax usually adjusts to pointer syntax, so the array size is not preserved.

**Detailed answer:**
A parameter written as `int values[]` is adjusted by the language to `int* values`. The function receives a pointer to the first element, not the whole array. This is why C APIs often pass a pointer and a length together.

**Example:**

```cpp
void f(int values[]) {
    // same as void f(int* values)
}

void g(int* values, std::size_t count);
```

To preserve the array size in C++, use a reference to array, `std::array`, or `std::span`.

```cpp
template <std::size_t N>
void h(int (&values)[N]);
```

**Interview point:** Array-to-pointer adjustment in parameters is a major reason raw array APIs lose size information.

**Common mistake:** Using `sizeof(values)` inside `void f(int values[])` and expecting the original array size.

---

## 33. How do you pass a multidimensional array to a function?

**Short answer:** All dimensions except the first must be known by the function type, or you should pass a view-like abstraction with explicit dimensions.

**Detailed answer:**
A multidimensional C-style array is contiguous storage with row-major layout. When passed to a function, the first dimension can decay, but the compiler must know the size of each row to compute indexing.

**Example:**

```cpp
void printMatrix(const int matrix[][3], std::size_t rows) {
    for (std::size_t r = 0; r < rows; ++r) {
        for (std::size_t c = 0; c < 3; ++c) {
            use(matrix[r][c]);
        }
    }
}
```

For dynamic dimensions, pass a flat buffer and explicit shape.

```cpp
void printFlat(const int* data, std::size_t rows, std::size_t cols) {
    use(data[1 * cols + 2]);
}
```

**Interview point:** `int**` is not the same as a pointer to a contiguous two-dimensional array.

**Common mistake:** Passing `int matrix[2][3]` to a function expecting `int**`.

---

## 34. What is the difference between `int*`, `int**`, and `int (*)[N]`?

**Short answer:** `int*` points to an `int`, `int**` points to a pointer to `int`, and `int (*)[N]` points to an array of `N` integers.

**Detailed answer:**
These types represent different memory layouts and indexing rules. `int**` is often used for arrays of pointers, where rows may be separately allocated. `int (*)[N]` points to a contiguous array row of fixed width `N`.

**Example:**

```cpp
int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};

int (*row)[3] = matrix; // pointer to array of 3 int
int value = row[1][2];  // 6
```

An `int**` cannot correctly represent this contiguous matrix without separate row pointers.

**Interview point:** Pointer syntax describes the type being pointed to, not just the number of `*` characters.

**Common mistake:** Assuming a two-dimensional array decays to `int**`.

---

## 35. What is a flexible array member in C?

**Short answer:** A flexible array member is a C struct's final array member with unspecified size, used to store variable-length data after the struct header.

**Detailed answer:**
C supports flexible array members as the last member of a struct. The program allocates extra storage and accesses the trailing array within that allocation. This is common in low-level C APIs and binary protocols.

**C example:**

```c
#include <stdlib.h>

struct Packet {
    size_t length;
    unsigned char data[];
};

struct Packet* packet = malloc(sizeof(struct Packet) + 128);
packet->length = 128;
free(packet);
```

C++ does not have standard flexible array members, though some compilers support them as extensions. In C++, prefer `std::vector`, `std::span`, or explicit allocation wrappers.

**Interview point:** Flexible array members require careful allocation-size calculations and lifetime management.

**Common mistake:** Placing fields after the flexible array member or allocating only `sizeof(struct Packet)`.

---

## 36. What is the difference between `strncpy` and a safe string copy?

**Short answer:** `strncpy` is often misunderstood; it may fail to null-terminate the destination and may pad with zeros.

**Detailed answer:**
Despite its name, `strncpy` is not a general safe string-copy function. It copies exactly up to `n` characters. If the source is at least `n` characters long, the destination is not null-terminated. If the source is shorter, the destination is padded with zeros.

**Problem example:**

```cpp
char dest[4];
std::strncpy(dest, "hello", sizeof(dest));
// dest is not null-terminated
```

A safer approach is to use `std::string`, `std::array<char, N>` with explicit termination, or APIs that take destination size and guarantee termination where available.

**Interview point:** Safe string handling requires knowing buffer size, termination rules, and truncation policy.

**Common mistake:** Replacing `strcpy` with `strncpy` and assuming the bug is fixed.

---

## 37. What is the difference between `strlen`, `strnlen`, and stored length?

**Short answer:** `strlen` scans until a null terminator, `strnlen` scans up to a maximum bound, and stored length avoids scanning.

**Detailed answer:**
`strlen` requires a valid null-terminated string. If the terminator is missing, it reads out of bounds. `strnlen` limits the scan length, which is safer for bounded buffers but still depends on sentinel termination within that bound. A stored length is best when data may contain embedded nulls or when repeated length queries must be efficient.

**Example:**

```cpp
const char text[] = {'a', '\0', 'b'};
std::size_t cLength = std::strlen(text); // 1
std::string_view view(text, 3);          // length 3
```

**Interview point:** Sentinel-terminated APIs and length-based APIs represent different contracts.

**Common mistake:** Using `strlen` on binary data or a buffer received from an untrusted source without ensuring termination.

---

## 38. What is `std::data`, and why is it useful?

**Short answer:** `std::data` returns a pointer to the underlying contiguous storage of arrays and standard containers that support it.

**Detailed answer:**
`std::data` provides a uniform way to get a raw pointer from C arrays, `std::array`, `std::vector`, and `std::string`. It is useful at C API boundaries where a pointer is required.

**Example:**

```cpp
#include <array>
#include <iterator>
#include <vector>

int raw[] = {1, 2, 3};
std::vector<int> vec = {4, 5, 6};
std::array<int, 3> arr = {7, 8, 9};

int* p1 = std::data(raw);
int* p2 = std::data(vec);
int* p3 = std::data(arr);
```

Use `std::size` with it when passing pointer-length pairs.

**Interview point:** `std::data` does not transfer ownership and does not make non-contiguous containers contiguous.

**Common mistake:** Expecting `std::data` to work meaningfully for containers such as `std::list`.

---

## 39. What are owning and non-owning string types?

**Short answer:** Owning string types manage character storage; non-owning string types only view storage owned elsewhere.

**Detailed answer:**
`std::string` owns its characters and controls their lifetime. `std::string_view` points to character data owned by another object. `const char*` may point to a string literal, an array, a string's internal storage, or invalid memory depending on context.

**Example:**

```cpp
std::string owner = "hello";
std::string_view view = owner;
const char* cstr = owner.c_str();
```

If `owner` is modified or destroyed, `view` and `cstr` may become invalid.

**Interview point:** Non-owning text types are efficient but require explicit lifetime reasoning.

**Common mistake:** Returning `std::string_view` to a local `std::string`.

---

## 40. What is the difference between contiguous and non-contiguous storage?

**Short answer:** Contiguous storage keeps elements adjacent in memory; non-contiguous storage may place elements in separate nodes or blocks.

**Detailed answer:**
Arrays, `std::array`, `std::vector`, and `std::string` store elements contiguously. This enables pointer arithmetic, cache-friendly traversal, and C API interoperability. Containers such as `std::list` store elements in separate nodes, so their elements are not adjacent.

**Example:**

```cpp
std::vector<int> values = {1, 2, 3};
int* p = values.data();
int second = *(p + 1); // OK for contiguous vector storage
```

For non-contiguous containers, use iterators rather than pointer arithmetic.

**Interview point:** Contiguity is an important part of API design, performance, and pointer validity.

**Common mistake:** Assuming every container's elements can be passed to a C API as one pointer-length block.

---

## 41. What is bounds checking, and which C++ facilities provide it?

**Short answer:** Bounds checking verifies that an index or pointer access stays within the valid range of an object or sequence.

**Detailed answer:**
Raw arrays and `operator[]` on standard containers usually do not perform runtime bounds checks. Accessing outside the valid range is undefined behavior. Some facilities, such as `std::vector::at`, `std::array::at`, and many debug iterator modes, check bounds and report errors.

**Example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3};

int a = values[1];      // no bounds check
int b = values.at(1);   // checks and throws on invalid index
```

Bounds checking is especially important at API boundaries where indexes, lengths, or offsets come from external input.

**Interview point:** Bounds safety is a contract issue: either prove the index is valid or use an API that checks it.

**Common mistake:** Believing `operator[]` on `std::vector` is safe because `std::vector` manages memory automatically.

---

## 42. What is the difference between `begin`/`end` iterators and pointer-length pairs?

**Short answer:** Both represent ranges, but iterators are generic over containers while pointer-length pairs specifically describe contiguous memory.

**Detailed answer:**
A pair of iterators `[begin, end)` can represent ranges from vectors, lists, maps, strings, and custom containers. A pointer-length pair such as `(data, size)` is appropriate for contiguous storage and C APIs.

**Example:**

```cpp
#include <span>
#include <vector>

void process(std::span<const int> values);

std::vector<int> data = {1, 2, 3};
process(data); // pointer and size are bundled safely
```

`std::span` is often a better C++ interface than separate pointer and length parameters because it keeps the two pieces together.

**Interview point:** Use iterator ranges for generic algorithms and spans for non-owning contiguous ranges.

**Common mistake:** Passing a pointer without a length and expecting the callee to know where the range ends.

---

## 43. What is a null-terminated string contract?

**Short answer:** A null-terminated string contract requires a valid character sequence ending with `\0`.

**Detailed answer:**
C string APIs such as `strlen`, `strcpy`, and many POSIX calls expect a pointer to a null-terminated character sequence. The pointer must be valid, and a terminator must appear before the accessible object ends. If the terminator is missing, the function may read beyond the buffer.

**Example:**

```cpp
char text[] = {'o', 'k'};
// std::strlen(text); // undefined behavior: no null terminator

char safe[] = {'o', 'k', '\0'};
```

Length-based APIs are safer for buffers that may contain embedded nulls or may not be terminated.

**Interview point:** `char*` alone does not prove a C string exists; termination is part of the contract.

**Common mistake:** Treating any character buffer as a valid C string.

---

## 44. What is the difference between `std::string::data()` and `c_str()`?

**Short answer:** Both return a pointer to string storage, but `c_str()` emphasizes null-terminated C-string access, while `data()` emphasizes raw character data access.

**Detailed answer:**
Since C++17, `std::string::data()` returns a null-terminated character array just like `c_str()` for const access, and non-const `data()` allows modifying characters within the string's size. The returned pointer is invalidated by operations that reallocate or modify the string in certain ways.

**Example:**

```cpp
#include <string>

std::string text = "hello";
const char* c = text.c_str();
char* p = text.data();
p[0] = 'H';
```

Do not write past `text.size()` or assume the pointer remains valid after resizing or appending.

**Interview point:** `data()` exposes storage but does not transfer ownership or capacity control.

**Common mistake:** Storing `c_str()` or `data()` pointers across string mutations.

---

## 45. What is pointer stability?

**Short answer:** Pointer stability means pointers or references to elements remain valid across operations on a container or object.

**Detailed answer:**
Some containers preserve element addresses across certain operations; others do not. `std::vector` may reallocate on growth, invalidating all element pointers and references. Node-based containers often preserve addresses of non-erased nodes, but they have different performance tradeoffs.

**Example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3};
int* p = &values[0];
values.push_back(4); // may reallocate
// p may now be dangling
```

Pointer stability matters when external code caches addresses into a container.

**Interview point:** Memory ownership and pointer validity must be considered together when choosing a container.

**Common mistake:** Keeping raw pointers into a vector while continuing to grow the vector.

---

## 46. What is the difference between byte length and character count?

**Short answer:** Byte length counts storage bytes; character count depends on text encoding and may require decoding.

**Detailed answer:**
In UTF-8, one user-visible character may use multiple bytes, and one user-perceived character may even involve multiple Unicode code points. `std::string::size()` returns bytes, not necessarily characters.

**Example:**

```cpp
#include <string>

std::string text = "é"; // UTF-8 may use two bytes
std::size_t bytes = text.size();
```

For ASCII-only data, byte count and character count match. For international text, they often do not.

**Interview point:** String correctness depends on the encoding contract, not just the C++ type.

**Common mistake:** Using byte indexes to split or truncate UTF-8 text without validating character boundaries.

---

## 47. What are embedded null characters in strings?

**Short answer:** Embedded null characters are `\0` bytes inside a string before its logical end.

**Detailed answer:**
`std::string` can store embedded nulls because it tracks length separately. C string APIs treat `\0` as the end of the string, so they see only the prefix before the first null byte.

**Example:**

```cpp
#include <cstring>
#include <string>

std::string text("a\0b", 3);
std::size_t cppSize = text.size();    // 3
std::size_t cSize = std::strlen(text.c_str()); // 1
```

This distinction matters in binary protocols, file formats, and security-sensitive parsing.

**Interview point:** Length-aware and sentinel-terminated APIs can interpret the same bytes differently.

**Common mistake:** Passing binary data through C string functions.

---

## 48. What is string interning, and what pointer risks can it introduce?

**Short answer:** String interning stores one shared copy of repeated strings, but pointers or views into interned storage must follow the interner's lifetime rules.

**Detailed answer:**
Interning can reduce memory usage and speed comparisons by allowing identity checks or shared storage. However, if interned strings can be removed, moved, or destroyed, cached `char*` pointers and `std::string_view`s may dangle.

**Conceptual example:**

```cpp
std::string_view name = interner.intern("content-type");
```

This is safe only if the interner guarantees the returned storage outlives the view.

**Interview point:** Interning changes ownership from individual strings to a shared table, so lifetime policy must be explicit.

**Common mistake:** Returning views from an interner whose storage can reallocate or erase entries unexpectedly.

---

## 49. What are sentinel values in arrays?

**Short answer:** A sentinel value marks the end or special meaning of data inside the data itself.

**Detailed answer:**
C strings use `\0` as a sentinel terminator. Other APIs may use `nullptr`, `-1`, or a special enum value. Sentinels avoid storing a separate length, but they require that the sentinel cannot be confused with valid data.

**Example:**

```cpp
const char* names[] = {"Ada", "Grace", nullptr};

for (const char** p = names; *p != nullptr; ++p) {
    use(*p);
}
```

Sentinel-based designs are fragile when data can contain the sentinel value or when termination is missing.

**Interview point:** Sentinel contracts are simple but less robust than explicit size when inputs are untrusted or binary.

**Common mistake:** Iterating until a sentinel that was never written.

---

## 50. How do you design a safe C API boundary for buffers?

**Short answer:** Pass explicit pointer, size, ownership, mutability, and lifetime information across the boundary.

**Detailed answer:**
C APIs often use raw pointers, so the contract must describe whether the pointer may be null, how many elements are accessible, whether the callee may write, who owns the memory, and how long the pointer remains valid. C++ wrappers can use `std::span`, `std::string_view`, `std::vector`, or RAII types internally, then convert at the boundary.

**Example:**

```cpp
extern "C" int process_bytes(const unsigned char* data, std::size_t size);

int process(std::span<const std::byte> bytes) {
    return process_bytes(
        reinterpret_cast<const unsigned char*>(bytes.data()),
        bytes.size()
    );
}
```

The C++ wrapper keeps pointer and length together and makes mutability explicit.

**Interview point:** A safe boundary is about a complete contract, not just replacing one pointer type with another.

**Common mistake:** Designing APIs that accept `void*` without size, ownership, or mutability rules.
