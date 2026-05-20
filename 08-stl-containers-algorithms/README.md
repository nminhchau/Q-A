# STL Containers and Algorithms Interview Questions

The C++ Standard Template Library is a major interview topic because it tests practical knowledge of containers, complexity, iterator invalidation, algorithms, and modern C++ coding style. Strong answers should explain not only which container to use, but why.

## 1. What is the STL?

**Short answer:** The STL is a set of generic containers, iterators, algorithms, and function objects in the C++ standard library.

**Detailed answer:**
The STL lets programmers write generic, reusable, type-safe code. Containers store data, iterators provide a common way to traverse data, algorithms operate on iterator ranges, and function objects or lambdas customize behavior.

**Example:**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> values = {4, 1, 3, 2};
    std::sort(values.begin(), values.end());

    for (int value : values) {
        std::cout << value << ' ';
    }
}
```

**Interview point:** STL algorithms are separated from containers through iterators. This is why the same algorithm can work with many container types.

---

## 2. What is the difference between `std::vector` and `std::list`?

**Short answer:** `std::vector` stores elements contiguously. `std::list` stores elements as a doubly linked list.

**Detailed answer:**
`std::vector` has fast random access, good cache locality, and efficient insertion/removal at the end. Inserting or removing in the middle can be expensive because elements may need to shift.

`std::list` supports constant-time insertion/removal when you already have an iterator to the position, but it has poor cache locality and no random access.

**Example:**

```cpp
std::vector<int> v = {1, 2, 3};
std::list<int> l = {1, 2, 3};

int x = v[1];      // fast random access
// int y = l[1];   // invalid: list has no operator[]
```

**Interview tip:** Prefer `std::vector` by default unless you have a specific reason not to. Its cache locality often beats linked-list theoretical advantages.

**Common mistake:** Choosing `std::list` for frequent insertions without considering traversal cost and cache misses.

---

## 3. What is the difference between `std::map` and `std::unordered_map`?

**Short answer:** `std::map` is ordered and usually tree-based. `std::unordered_map` is hash-table-based and does not maintain sorted order.

**Detailed answer:**
`std::map` typically provides `O(log n)` lookup, insertion, and deletion. It keeps keys sorted and supports ordered traversal, lower_bound, and range queries.

`std::unordered_map` provides average `O(1)` lookup, insertion, and deletion, but worst-case `O(n)` if many keys collide. It does not keep keys sorted.

**Example:**

```cpp
#include <map>
#include <unordered_map>

std::map<std::string, int> ordered;
std::unordered_map<std::string, int> hashed;
```

**Choose `std::map` when:**
- You need sorted iteration.
- You need range queries.
- You need stable ordering.

**Choose `std::unordered_map` when:**
- You mainly need fast key lookup.
- Ordering does not matter.
- A good hash function exists.

**Common mistake:** Assuming `unordered_map` is always faster. Hashing cost, collisions, memory overhead, and iteration patterns matter.

---

## 4. What is iterator invalidation?

**Short answer:** Iterator invalidation happens when a container operation makes existing iterators, pointers, or references no longer safe to use.

**Detailed answer:**
Different containers have different invalidation rules. For example, adding to a `std::vector` may reallocate its storage, invalidating all iterators, pointers, and references to its elements.

**Example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3};
int* p = &values[0];

values.push_back(4); // may reallocate
// *p may now be invalid
```

**Common cases:**
- `std::vector::push_back` may invalidate everything if capacity grows.
- `std::vector::erase` invalidates iterators at and after the erased position.
- `std::list` insertion generally does not invalidate existing iterators.
- `std::unordered_map` rehashing invalidates iterators.

**Interview point:** Iterator invalidation is a common source of subtle bugs in C++.

---

## 5. What is the erase-remove idiom?

**Short answer:** The erase-remove idiom removes elements from sequence containers such as `std::vector` using `std::remove` followed by `erase`.

**Detailed answer:**
`std::remove` does not actually erase elements from the container. It rearranges elements so the kept values are moved to the front and returns an iterator to the new logical end. The container size remains unchanged until `erase` is called.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {1, 2, 3, 2, 4};

values.erase(
    std::remove(values.begin(), values.end(), 2),
    values.end()
);
```

After this, `values` contains `{1, 3, 4}`.

**Modern C++ note:** C++20 provides `std::erase` and `std::erase_if` for many containers.

```cpp
std::erase(values, 2);
```

**Common mistake:** Calling only `std::remove` and expecting the container size to shrink.

---

## 6. What is the difference between `push_back` and `emplace_back`?

**Short answer:** `push_back` inserts an existing object or moved value. `emplace_back` constructs the element directly in the container.

**Detailed answer:**
`emplace_back` forwards constructor arguments to create the element in place. It can avoid a temporary object, although modern compilers often optimize simple cases well.

**Example:**

```cpp
#include <string>
#include <vector>

std::vector<std::string> names;

names.push_back(std::string("Ada"));
names.emplace_back("Grace");
```

**Interview point:** `emplace_back` is not automatically better. If you already have an object, `push_back(std::move(obj))` is clear and appropriate.

**Common mistake:** Overusing `emplace_back` when `push_back` is simpler and equally efficient.

---

## 7. What is the difference between `std::set` and `std::unordered_set`?

**Short answer:** `std::set` stores unique elements in sorted order. `std::unordered_set` stores unique elements in hash-table buckets without sorted order.

**Detailed answer:**
`std::set` usually uses a balanced tree and provides `O(log n)` operations. It supports ordered traversal and range queries. `std::unordered_set` usually provides average `O(1)` operations but requires hashing and equality comparison.

**Example:**

```cpp
std::set<int> ordered = {3, 1, 2};
std::unordered_set<int> hashed = {3, 1, 2};
```

Iterating `ordered` produces sorted order. Iterating `hashed` does not guarantee order.

**Choose based on:**
- Need sorted order: use `set`.
- Need fast membership test and no ordering: use `unordered_set`.

**Common mistake:** Depending on the iteration order of `unordered_set`. It is not stable or sorted.

---

## 8. What are STL algorithm complexity guarantees?

**Short answer:** STL algorithms and containers document their expected time complexity, which helps choose the right tool.

**Detailed answer:**
Complexity matters in interviews because two correct-looking solutions may scale very differently. For example, `std::find` is linear, while `unordered_map::find` is average constant time.

**Examples:**

```cpp
std::find(values.begin(), values.end(), target); // O(n)

std::sort(values.begin(), values.end());         // O(n log n)

map.find(key);                                   // O(log n)

unorderedMap.find(key);                          // average O(1)
```

**Interview point:** Big-O is not the whole story. Cache locality, allocation cost, hashing cost, and constant factors can dominate in real programs.

---

## 9. What is a comparator in STL algorithms and containers?

**Short answer:** A comparator defines ordering between elements.

**Detailed answer:**
Comparators are used by algorithms like `std::sort` and containers like `std::set` or `std::map`. A comparator should usually implement strict weak ordering.

**Example:**

```cpp
#include <algorithm>
#include <vector>

struct Person {
    std::string name;
    int age;
};

std::vector<Person> people;

std::sort(people.begin(), people.end(), [](const Person& a, const Person& b) {
    return a.age < b.age;
});
```

**Strict weak ordering means:**
- An element is not less than itself.
- Ordering is transitive.
- Equivalent elements are handled consistently.

**Common mistake:** Writing a comparator using `<=` instead of `<`, which can violate strict weak ordering.

---

## 10. When should you use STL algorithms instead of manual loops?

**Short answer:** Use STL algorithms when they express intent clearly and correctly.

**Detailed answer:**
Algorithms such as `std::find`, `std::sort`, `std::count_if`, `std::transform`, and `std::accumulate` communicate what the code is doing at a higher level than manual loops. They also reduce off-by-one errors and make code easier to review.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {1, 2, 3, 4, 5};

bool hasEven = std::any_of(values.begin(), values.end(), [](int value) {
    return value % 2 == 0;
});
```

**Interview point:** Knowing algorithms shows practical C++ fluency. However, a manual loop is fine when it is clearer or when the logic does not map naturally to an algorithm.

**Common mistake:** Using complex chains of algorithms that are harder to understand than a simple loop.

---

## 11. What is the difference between `std::vector` and `std::deque`?

**Short answer:** `std::vector` stores elements contiguously, while `std::deque` stores elements in multiple blocks and supports efficient insertion at both ends.

**Detailed answer:**
`std::vector` provides the best cache locality and is usually the default sequence container. `std::deque` allows efficient `push_front` and `push_back` without moving all existing elements, but its storage is not one contiguous array.

**Example:**

```cpp
#include <deque>
#include <vector>

std::vector<int> values;
values.push_back(1);

std::deque<int> queue;
queue.push_back(1);
queue.push_front(0);
```

**Choose `std::deque` when:**
- You need frequent insertion/removal at both ends.
- You do not need contiguous storage.
- You want random access but not necessarily vector-like memory layout.

**Common mistake:** Assuming `deque` is just a slower `vector`. It solves a different problem, especially for double-ended growth.

---

## 12. What is `std::priority_queue`?

**Short answer:** `std::priority_queue` is a container adapter that keeps the highest-priority element accessible at the top.

**Detailed answer:**
`std::priority_queue` is usually implemented using a heap over an underlying container such as `std::vector`. It is useful when you repeatedly need the largest or smallest element, depending on the comparator.

**Example:**

```cpp
#include <queue>
#include <vector>

std::priority_queue<int> maxHeap;
maxHeap.push(3);
maxHeap.push(10);
maxHeap.push(5);

int largest = maxHeap.top(); // 10
```

For a min-heap:

```cpp
#include <functional>
#include <queue>
#include <vector>

std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
```

**Interview point:** `priority_queue` does not keep all elements sorted. It only guarantees efficient access to the top-priority element.

**Common mistake:** Iterating a priority queue expecting sorted order. It is an adapter, not a sorted container.

---

## 13. What is the difference between `size`, `capacity`, and `reserve` in `std::vector`?

**Short answer:** `size` is the number of elements stored, `capacity` is how many elements can fit before reallocation, and `reserve` requests capacity.

**Detailed answer:**
A vector may allocate more memory than it currently uses so future growth can be efficient. `reserve` can reduce reallocations when the approximate number of elements is known in advance. It does not change the vector's size.

**Example:**

```cpp
#include <vector>

std::vector<int> values;
values.reserve(100);

values.push_back(1);
values.push_back(2);

std::size_t count = values.size();     // 2
std::size_t storage = values.capacity(); // at least 100
```

**Interview point:** Reallocation can invalidate iterators, pointers, and references to vector elements.

**Common mistake:** Calling `reserve` and then writing with `values[i]` before elements exist. Use `resize` if you need to create elements.

---

## 14. What are C++20 ranges?

**Short answer:** Ranges are a modern way to work with sequences directly, often without manually passing iterator pairs.

**Detailed answer:**
The ranges library provides range-based algorithms and composable views. It can make code clearer by treating a sequence as one object rather than a `begin`/`end` pair.

**Example:**

```cpp
#include <algorithm>
#include <ranges>
#include <vector>

std::vector<int> values = {3, 1, 2};
std::ranges::sort(values);
```

Views can create lazy transformations or filters.

```cpp
auto evens = values | std::views::filter([](int value) {
    return value % 2 == 0;
});
```

**Interview point:** Ranges improve expressiveness, but candidates should still understand classic iterators because much C++ code and many APIs still use them.

**Common mistake:** Returning a view that references a local container, which creates dangling references.

---

## 15. What is the difference between `insert`, `emplace`, and `try_emplace` in associative containers?

**Short answer:** `insert` adds an existing value, `emplace` constructs a value in place, and `try_emplace` avoids constructing the mapped value if the key already exists.

**Detailed answer:**
For containers such as `std::map` and `std::unordered_map`, insertion APIs differ in how they construct objects and what happens when the key is already present.

**Example:**

```cpp
#include <map>
#include <string>

std::map<int, std::string> names;

names.insert({1, "Ada"});
names.emplace(2, "Grace");
names.try_emplace(3, "Linus");
```

`try_emplace` is especially useful when constructing the mapped value is expensive or move-only.

```cpp
std::map<int, std::unique_ptr<int>> values;
values.try_emplace(1, std::make_unique<int>(42));
```

**Interview point:** Prefer APIs that express intent. Use `try_emplace` when you only want to construct the value if insertion succeeds.

**Common mistake:** Using `operator[]` for lookup in a map when accidental insertion of a default value would be wrong.

---

## 16. What is the difference between `std::array` and `std::vector`?

**Short answer:** `std::array` has fixed size known at compile time. `std::vector` has dynamic size managed at runtime.

**Detailed answer:**
`std::array<T, N>` stores exactly `N` elements directly inside the object. It does not allocate dynamically and cannot grow or shrink. `std::vector<T>` owns a dynamically allocated buffer and can change size.

**Example:**

```cpp
#include <array>
#include <vector>

std::array<int, 3> fixed = {1, 2, 3};
std::vector<int> dynamic = {1, 2, 3};

dynamic.push_back(4);
```

**Choose `std::array` when:**
- The size is a compile-time constant.
- You want value semantics with no dynamic allocation.
- The object should be embedded directly in another object.

**Choose `std::vector` when:**
- The size is known only at runtime.
- The container must grow or shrink.
- You need dynamic ownership of a sequence.

**Common mistake:** Using `std::vector` for small fixed-size data where `std::array` would be simpler and allocation-free.

---

## 17. What are iterator categories, and why do they matter?

**Short answer:** Iterator categories describe what operations an iterator supports, such as single-pass traversal, bidirectional movement, or random access.

**Detailed answer:**
Algorithms have requirements on iterator capabilities. For example, `std::sort` requires random-access iterators, so it works with `std::vector` but not with `std::list`.

**Examples:**
- Input iterators: read values in one pass.
- Forward iterators: make multiple passes forward.
- Bidirectional iterators: move forward and backward.
- Random-access iterators: jump by offsets and compare positions efficiently.
- Contiguous iterators: point into contiguous memory.

**Example:**

```cpp
std::vector<int> values = {3, 1, 2};
std::sort(values.begin(), values.end());

std::list<int> items = {3, 1, 2};
// std::sort(items.begin(), items.end()); // error: not random access
items.sort();
```

**Interview point:** Iterator categories explain why some algorithms work only with certain containers.

**Common mistake:** Assuming all iterators behave like pointers.

---

## 18. What is the difference between `std::sort`, `std::stable_sort`, and `std::partial_sort`?

**Short answer:** `std::sort` sorts the whole range, `std::stable_sort` preserves equivalent-element order, and `std::partial_sort` sorts only the first part of a range.

**Detailed answer:**
Use `std::sort` for general full sorting. Use `std::stable_sort` when equivalent elements must keep their original relative order. Use `std::partial_sort` when only the smallest or largest `k` sorted elements are needed.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {5, 1, 4, 2, 3};

std::sort(values.begin(), values.end());
```

For top-k style problems:

```cpp
std::partial_sort(values.begin(), values.begin() + 3, values.end());
```

After this, the first three elements are the three smallest values in sorted order.

**Interview point:** Choosing the right sorting algorithm can improve performance and express intent.

**Common mistake:** Fully sorting a large range when only a small prefix is needed.

---

## 19. What is `std::span`, and how is it different from a container?

**Short answer:** `std::span` is a non-owning view over a contiguous sequence. It does not store or manage the elements.

**Detailed answer:**
`std::span<T>` can refer to an array, `std::vector`, or other contiguous storage. It is useful for function parameters when a function needs a sequence but should not care who owns it.

**Example:**

```cpp
#include <span>
#include <vector>

void fillZeros(std::span<int> values) {
    for (int& value : values) {
        value = 0;
    }
}

int raw[3] = {1, 2, 3};
std::vector<int> vec = {4, 5, 6};

fillZeros(raw);
fillZeros(vec);
```

**Interview point:** `std::span` is a view, so the caller must ensure the underlying data outlives the span.

**Common mistake:** Returning a `std::span` to a local vector or temporary array.

---

## 20. How do `std::lower_bound` and `std::upper_bound` work?

**Short answer:** They perform binary search on a sorted range to find insertion boundaries.

**Detailed answer:**
`std::lower_bound` returns the first position where the value could be inserted without going before any equivalent value. `std::upper_bound` returns the first position after all equivalent values.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {1, 2, 2, 2, 5};

auto lower = std::lower_bound(values.begin(), values.end(), 2);
auto upper = std::upper_bound(values.begin(), values.end(), 2);
```

`lower` points to the first `2`; `upper` points to `5`.

**Interview point:** These algorithms require the range to be sorted according to the same comparison used for searching.

**Common mistake:** Calling binary-search algorithms on an unsorted range and expecting meaningful results.

---

## 21. What is the difference between `std::find`, `std::binary_search`, and associative container lookup?

**Short answer:** `std::find` scans linearly, `std::binary_search` searches a sorted range logarithmically, and associative containers provide lookup based on their own data structures.

**Detailed answer:**
`std::find` works on any input range but takes linear time. `std::binary_search` requires a sorted range and only answers whether a value exists. Ordered associative containers such as `std::set` and `std::map` usually provide logarithmic lookup. Unordered containers such as `std::unordered_set` and `std::unordered_map` usually provide average constant-time lookup.

**Example:**

```cpp
#include <algorithm>
#include <set>
#include <unordered_set>
#include <vector>

std::vector<int> values = {1, 2, 3, 4, 5};
bool foundLinear = std::find(values.begin(), values.end(), 3) != values.end();
bool foundBinary = std::binary_search(values.begin(), values.end(), 3);

std::set<int> ordered = {1, 2, 3};
bool foundOrdered = ordered.find(3) != ordered.end();

std::unordered_set<int> hashed = {1, 2, 3};
bool foundHashed = hashed.find(3) != hashed.end();
```

**Interview point:** Lookup choice depends on whether data is sorted, whether insertion happens often, and whether ordering is needed.

**Common mistake:** Sorting a range just to do one binary search when a linear scan would be simpler and often faster.

---

## 22. What is `std::stable_sort`, and when is it useful?

**Short answer:** `std::stable_sort` sorts elements while preserving the relative order of elements that compare equivalent.

**Detailed answer:**
A stable sort matters when records have multiple keys or when earlier ordering carries meaning. For example, if a list is already sorted by first name, a stable sort by last name keeps first-name order among people with the same last name.

`std::sort` is not stable, but it may use less extra memory and is often faster. `std::stable_sort` usually requires additional memory.

**Example:**

```cpp
#include <algorithm>
#include <string>
#include <vector>

struct Person {
    std::string first;
    std::string last;
};

std::vector<Person> people = {
    {"Ada", "Lovelace"},
    {"Grace", "Hopper"},
    {"Katherine", "Johnson"},
    {"Alan", "Hopper"}
};

std::stable_sort(people.begin(), people.end(), [](const Person& a, const Person& b) {
    return a.last < b.last;
});
```

The two `Hopper` entries keep their previous relative order.

**Interview point:** Stability is a semantic property, not just a performance detail.

**Common mistake:** Using `std::sort` when equal-key records must keep their original order.

---

## 23. What are node handles in associative containers?

**Short answer:** Node handles let you extract, modify, and reinsert nodes from associative containers without reallocating the stored element.

**Detailed answer:**
Since C++17, associative containers support `extract`. It removes a node while preserving ownership of the element and internal allocation. This is useful for changing keys in maps or moving elements between compatible containers.

**Example:**

```cpp
#include <map>
#include <string>

std::map<int, std::string> users = {
    {1, "Ada"},
    {2, "Grace"}
};

auto node = users.extract(1);
node.key() = 42;
users.insert(std::move(node));
```

Changing a key directly inside a `std::map` is not allowed because it would break ordering. A node handle provides a safe remove-modify-reinsert workflow.

**Interview point:** Node handles are useful when key mutation or efficient transfer between containers is needed.

**Common mistake:** Trying to modify a `std::map` key through an iterator.

---

## 24. What is transparent lookup in associative containers?

**Short answer:** Transparent lookup lets associative containers search using a different but comparable key type without constructing the container's key type.

**Detailed answer:**
For ordered containers, transparent lookup is enabled by comparators such as `std::less<>`. This can avoid unnecessary temporary objects, especially with strings.

**Example:**

```cpp
#include <set>
#include <string>
#include <string_view>

std::set<std::string, std::less<>> names = {"Ada", "Grace"};

std::string_view query = "Ada";
auto it = names.find(query); // no std::string temporary needed
```

The comparator can compare `std::string` and `std::string_view`, so lookup can use the view directly.

**Interview point:** Transparent lookup improves performance and API ergonomics for heterogeneous key searches.

**Common mistake:** Using `std::less<std::string>` and expecting heterogeneous lookup to work automatically.

---

## 25. How do algorithms use iterator pairs and ranges differently?

**Short answer:** Traditional STL algorithms use iterator pairs, while C++20 ranges can operate directly on range objects and compose with views.

**Detailed answer:**
Classic algorithms usually take `[first, last)` iterator pairs. This is flexible but can be verbose and can accidentally mix iterators from different containers. Ranges algorithms accept a range directly and integrate with views for lazy transformations and filtering.

**Iterator-pair style:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {3, 1, 2};
std::sort(values.begin(), values.end());
```

**Ranges style:**

```cpp
#include <algorithm>
#include <ranges>
#include <vector>

std::vector<int> values = {3, 1, 2};
std::ranges::sort(values);
```

**View composition:**

```cpp
auto positives = values | std::views::filter([](int x) { return x > 0; });
```

**Interview point:** Ranges reduce boilerplate and enable composable views, but programmers still need to understand iterator categories, lifetimes, and algorithm complexity.

**Common mistake:** Returning or storing views that refer to short-lived ranges.

---

## 26. What is the difference between `std::vector::reserve` and `std::vector::resize`?

**Short answer:** `reserve` changes capacity without changing the number of elements; `resize` changes the number of elements.

**Detailed answer:**
`reserve(n)` asks a vector to allocate enough storage for at least `n` elements. It does not construct new elements and does not change `size()`. `resize(n)` changes `size()` and constructs or destroys elements as needed.

**Example:**

```cpp
#include <vector>

std::vector<int> values;

values.reserve(100); // size is 0, capacity is at least 100
values.resize(100);  // size is 100, elements are value-initialized to 0
```

Use `reserve` when you know how many elements you will append. Use `resize` when the vector should actually contain that many elements.

**Interview point:** Capacity is allocated storage; size is the number of live elements.

**Common mistake:** Calling `reserve` and then writing through `values[i]` before elements exist.

---

## 27. When should you use `std::list`?

**Short answer:** Use `std::list` when you need stable iterators and frequent splicing or insertion/removal in the middle after you already have the position.

**Detailed answer:**
`std::list` is a doubly linked list. It supports constant-time insertion and removal once an iterator is known, and iterators to other elements remain valid. However, it has poor cache locality, no random access, and extra memory overhead per node.

**Example:**

```cpp
#include <list>

std::list<int> a = {1, 2, 3};
std::list<int> b = {4, 5};

a.splice(a.end(), b); // moves nodes from b to a without copying elements
```

For most general sequences, `std::vector` is faster because contiguous storage works well with caches.

**Interview point:** `std::list` is not automatically faster for insertion-heavy workloads; finding the insertion position can dominate.

**Common mistake:** Choosing `std::list` because insertions are theoretically O(1) while ignoring allocation and cache costs.

---

## 28. What are iterator invalidation rules?

**Short answer:** Iterator invalidation rules define when iterators, pointers, and references to container elements become unusable after container operations.

**Detailed answer:**
Different containers invalidate iterators differently. `std::vector` reallocation invalidates all iterators, pointers, and references. Inserting into a `std::map` usually does not invalidate existing iterators. Erasing an element invalidates iterators to erased elements.

**Example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3};
auto it = values.begin();

values.push_back(4); // may reallocate
// *it is unsafe if reallocation happened
```

You must know the invalidation rules of the specific container and operation.

**Interview point:** Iterator invalidation is a common source of use-after-free-like bugs in C++.

**Common mistake:** Keeping iterators across `push_back`, `insert`, or `erase` without checking the container's rules.

---

## 29. How should you erase elements while iterating through a container?

**Short answer:** Use the iterator returned by `erase` for sequence containers and erase carefully for associative containers.

**Detailed answer:**
For containers such as `std::vector`, `std::deque`, `std::list`, `erase` returns the next valid iterator. Assigning that return value prevents using an invalidated iterator.

**Example:**

```cpp
#include <vector>

std::vector<int> values = {1, 2, 3, 4, 5};

for (auto it = values.begin(); it != values.end(); ) {
    if (*it % 2 == 0) {
        it = values.erase(it);
    } else {
        ++it;
    }
}
```

For bulk removal from vectors, `std::erase_if` in C++20 is often simpler.

**Interview point:** Erasing while iterating is safe only when the loop accounts for invalidation.

**Common mistake:** Incrementing an iterator after it has been erased.

---

## 30. What is `std::erase_if`?

**Short answer:** `std::erase_if` removes all elements matching a predicate from a standard container.

**Detailed answer:**
C++20 introduced `std::erase_if` for standard containers. It provides a clear way to remove elements by condition without manually writing erase-remove or iterator-erasure loops.

**Example:**

```cpp
#include <vector>
#include <algorithm>

std::vector<int> values = {1, 2, 3, 4, 5};

std::erase_if(values, [](int x) {
    return x % 2 == 0;
});
```

For `std::vector`, this removes matching elements and shifts the remaining elements. For associative containers, it erases matching nodes.

**Interview point:** `std::erase_if` improves readability and reduces invalidation mistakes.

**Common mistake:** Using `std::remove_if` alone and forgetting to call `erase` afterward.

---

## 31. What is the difference between ordered and unordered associative containers?

**Short answer:** Ordered containers keep keys sorted by comparison; unordered containers organize keys by hash.

**Detailed answer:**
`std::map` and `std::set` are ordered associative containers, usually implemented as balanced trees. They provide sorted iteration and logarithmic lookup. `std::unordered_map` and `std::unordered_set` use hashing, providing average constant-time lookup but no sorted order.

**Example:**

```cpp
#include <map>
#include <unordered_map>
#include <string>

std::map<std::string, int> sortedCounts;
std::unordered_map<std::string, int> hashedCounts;
```

Choose ordered containers when sorted traversal, range queries, or stable ordering matter. Choose unordered containers for fast lookup when a good hash is available.

**Interview point:** Big-O is not the only factor; ordering, hashing cost, memory layout, and worst-case behavior matter.

**Common mistake:** Assuming `unordered_map` is always faster than `map`.

---

## 32. What are custom hash functions used for?

**Short answer:** Custom hash functions let unordered containers hash user-defined key types or customize hashing behavior.

**Detailed answer:**
`std::unordered_map` needs a hash function and equality comparison for keys. Standard types already have hash support, but custom structs usually need a custom hash and `operator==`.

**Example:**

```cpp
#include <cstddef>
#include <functional>
#include <unordered_map>

struct Point {
    int x;
    int y;

    friend bool operator==(const Point&, const Point&) = default;
};

struct PointHash {
    std::size_t operator()(const Point& p) const noexcept {
        std::size_t h1 = std::hash<int>{}(p.x);
        std::size_t h2 = std::hash<int>{}(p.y);
        return h1 ^ (h2 << 1);
    }
};

std::unordered_map<Point, int, PointHash> counts;
```

The hash function should be consistent with equality: equal keys must produce equal hash values.

**Interview point:** A poor hash can destroy performance by creating many collisions.

**Common mistake:** Defining a hash that disagrees with equality.

---

## 33. What is load factor in `std::unordered_map`?

**Short answer:** Load factor is the average number of elements per bucket in an unordered container.

**Detailed answer:**
`load_factor()` is approximately `size() / bucket_count()`. A high load factor means more collisions and longer bucket chains or probes. `max_load_factor()` controls when the container rehashes.

**Example:**

```cpp
#include <unordered_map>
#include <string>

std::unordered_map<std::string, int> counts;
counts.max_load_factor(0.7f);
counts.reserve(1000);
```

Calling `reserve` can reduce repeated rehashing when the expected number of elements is known.

**Interview point:** Rehashing invalidates iterators and can be expensive, but it preserves references and pointers to elements in standard unordered associative containers.

**Common mistake:** Ignoring rehash costs in performance-sensitive insertion-heavy code.

---

## 34. What is the difference between `operator[]`, `at`, and `find` in maps?

**Short answer:** `operator[]` inserts a missing key, `at` requires the key to exist, and `find` checks without inserting.

**Detailed answer:**
For `std::map` and `std::unordered_map`, `operator[]` default-constructs a mapped value if the key is missing. `at` throws `std::out_of_range` when the key is absent. `find` returns an iterator and does not modify the container.

**Example:**

```cpp
#include <map>
#include <string>

std::map<std::string, int> counts;

counts["Ada"]++;           // inserts "Ada" with value 0, then increments
// counts.at("Grace");     // throws if missing

if (auto it = counts.find("Ada"); it != counts.end()) {
    int value = it->second;
}
```

Use `find` or `contains` for existence checks that should not mutate the map.

**Interview point:** Accidental insertion through `operator[]` can change program behavior and performance.

**Common mistake:** Using `operator[]` just to test whether a key exists.

---

## 35. What are heap algorithms in the STL?

**Short answer:** Heap algorithms maintain a heap structure over a range, usually to access the largest element efficiently.

**Detailed answer:**
The STL provides algorithms such as `std::make_heap`, `std::push_heap`, `std::pop_heap`, and `std::sort_heap`. They operate on random-access ranges, commonly vectors. By default, they create a max-heap.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {3, 1, 4, 2};

std::make_heap(values.begin(), values.end());
values.push_back(5);
std::push_heap(values.begin(), values.end());

std::pop_heap(values.begin(), values.end());
int largest = values.back();
values.pop_back();
```

`std::priority_queue` wraps this pattern in a container adaptor.

**Interview point:** After `pop_heap`, the selected element is moved to the end; you still need to remove it from the container if desired.

**Common mistake:** Calling `pop_heap` and expecting the container size to shrink automatically.

---

## 36. What are container adaptors?

**Short answer:** Container adaptors provide restricted interfaces over underlying containers.

**Detailed answer:**
`std::stack`, `std::queue`, and `std::priority_queue` are container adaptors. They are not standalone container implementations; they use another container internally and expose operations suited to a specific data structure.

**Example:**

```cpp
#include <queue>
#include <stack>
#include <vector>

std::stack<int> stack;
stack.push(1);
stack.pop();

std::priority_queue<int, std::vector<int>, std::greater<>> minHeap;
minHeap.push(3);
minHeap.push(1);
```

Adaptors intentionally hide iteration and arbitrary access.

**Interview point:** Adaptors express intent and prevent unsupported operations from leaking into code.

**Common mistake:** Expecting to iterate directly over a `std::stack` or `std::queue`.

---

## 37. What is the difference between `std::begin` and a container's `begin` member?

**Short answer:** `std::begin` is a generic function that works with containers, arrays, and some custom range-like types; member `begin` works only when the object provides that member.

**Detailed answer:**
Generic code often uses `std::begin` and `std::end` to support both standard containers and raw arrays. In modern C++, ranges utilities further generalize this idea.

**Example:**

```cpp
#include <iterator>
#include <vector>

template <typename Range>
auto firstIterator(Range& range) {
    using std::begin;
    return begin(range);
}

std::vector<int> values = {1, 2, 3};
int raw[] = {4, 5, 6};

auto a = firstIterator(values);
auto b = firstIterator(raw);
```

The `using std::begin; begin(range);` pattern also allows argument-dependent lookup for custom ranges.

**Interview point:** Generic algorithms should avoid assuming every range is a standard container.

**Common mistake:** Writing templates that call `.begin()` and then fail for raw arrays.

---

## 38. What are projections in C++20 ranges algorithms?

**Short answer:** Projections transform each element before comparison or processing inside a ranges algorithm.

**Detailed answer:**
C++20 ranges algorithms often accept a projection parameter. This lets you sort or search by a member without writing a full comparator.

**Example:**

```cpp
#include <algorithm>
#include <ranges>
#include <string>
#include <vector>

struct Person {
    std::string name;
    int age;
};

std::vector<Person> people = {{"Ada", 36}, {"Grace", 85}};

std::ranges::sort(people, {}, &Person::age);
```

The empty comparator `{}` means use the default ordering after applying the projection.

**Interview point:** Projections improve readability when sorting or searching by a single field.

**Common mistake:** Writing overly complex lambdas when a simple projection would express the intent.

---

## 39. What is a dangling iterator or dangling view?

**Short answer:** A dangling iterator or view refers to a range or element that no longer exists.

**Detailed answer:**
Iterators and views are usually non-owning. If they refer to a temporary container or a container that has been destroyed or modified in an invalidating way, using them is undefined behavior.

**Example:**

```cpp
#include <ranges>
#include <vector>

std::vector<int> makeValues() {
    return {1, 2, 3, 4};
}

auto evens = makeValues() | std::views::filter([](int x) {
    return x % 2 == 0;
});
```

The view may refer to a temporary range whose lifetime does not support later use, depending on the range and view composition. Prefer storing the owning range when the view must live longer.

**Interview point:** Ranges improve composability but do not remove lifetime concerns.

**Common mistake:** Treating views as if they always own their data.

---

## 40. How do you choose the right STL container?

**Short answer:** Choose based on access pattern, ordering needs, iterator stability, memory layout, and operation complexity.

**Detailed answer:**
No single container is best for every situation. `std::vector` is usually the default sequence container because it is compact and cache-friendly. `std::deque` is useful for efficient growth at both ends. `std::list` is specialized for stable node-based operations. `std::map` and `std::set` provide sorted associative lookup. `std::unordered_map` and `std::unordered_set` provide hash-based lookup.

**Guiding questions:**

- Do you need random access?
- Do you need stable iterators or references?
- Do you need sorted iteration?
- Do you need fast lookup by key?
- Is memory locality important?
- Will insertions happen mostly at the end, the front, or the middle?

**Example choices:**

```cpp
std::vector<int> denseValues;              // compact sequential data
std::deque<int> workQueue;                 // push/pop at both ends
std::map<std::string, int> sortedCounts;   // sorted keys
std::unordered_map<std::string, int> cache; // hash lookup
```

**Interview point:** Good container choice is about workload, not memorizing one universal rule.

**Common mistake:** Using a theoretically attractive container while ignoring real access patterns and cache behavior.

---

## 41. What is the difference between `std::find` and container member `find`?

**Short answer:** `std::find` performs a linear scan; associative container member `find` uses the container's lookup structure.

**Detailed answer:**
`std::find` works on any iterator range, so it cannot assume hashing or tree ordering. For `std::map`, `std::set`, `std::unordered_map`, and related containers, member `find` uses the container's key lookup mechanism and is usually much faster.

**Example:**

```cpp
#include <algorithm>
#include <set>
#include <vector>

std::vector<int> values = {1, 2, 3};
auto a = std::find(values.begin(), values.end(), 2); // linear

std::set<int> ids = {1, 2, 3};
auto b = ids.find(2); // logarithmic tree lookup
```

**Interview point:** Prefer member lookup when the container has an indexed or associative search operation.

**Common mistake:** Using `std::find(map.begin(), map.end(), key)` and accidentally scanning key-value pairs linearly.

---

## 42. What is the difference between `lower_bound`, `upper_bound`, and `equal_range`?

**Short answer:** `lower_bound` finds the first position not less than a value, `upper_bound` finds the first position greater than it, and `equal_range` returns both bounds.

**Detailed answer:**
These algorithms operate on sorted ranges. They are useful for binary search, insertion positions, and finding all equivalent values. Associative containers also provide member versions that use the container's ordering directly.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {1, 2, 2, 2, 5};

auto first = std::lower_bound(values.begin(), values.end(), 2);
auto last = std::upper_bound(values.begin(), values.end(), 2);
```

The range `[first, last)` contains all `2` values.

**Interview point:** Binary-search algorithms require the range to be sorted according to the same ordering used by the search.

**Common mistake:** Calling `lower_bound` on an unsorted vector and expecting meaningful results.

---

## 43. What is the difference between `std::partition` and `std::stable_partition`?

**Short answer:** `std::partition` separates elements by predicate without preserving order; `std::stable_partition` preserves relative order within each group.

**Detailed answer:**
Partitioning rearranges a range so elements satisfying a predicate come before those that do not. Stable partitioning keeps the original relative order of both groups, usually with extra cost.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {1, 2, 3, 4, 5};

auto middle = std::partition(values.begin(), values.end(), [](int x) {
    return x % 2 == 0;
});
```

After partitioning, even values appear before odd values, but their original order is not guaranteed.

**Interview point:** Stability is a semantic requirement; choose stable algorithms only when that requirement matters.

**Common mistake:** Assuming `std::partition` preserves original order.

---

## 44. What are `std::all_of`, `std::any_of`, and `std::none_of`?

**Short answer:** They test whether all, any, or none of the elements in a range satisfy a predicate.

**Detailed answer:**
These algorithms express intent more clearly than manual loops. They also short-circuit: `all_of` stops at the first false predicate, `any_of` stops at the first true predicate, and `none_of` stops at the first true predicate.

**Example:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> values = {2, 4, 6};

bool allEven = std::all_of(values.begin(), values.end(), [](int x) {
    return x % 2 == 0;
});
```

**Interview point:** Standard algorithms communicate both behavior and complexity expectations.

**Common mistake:** Writing verbose loops that obscure simple predicate checks.

---

## 45. What is `std::transform` used for?

**Short answer:** `std::transform` applies a function to elements and writes the results to an output range.

**Detailed answer:**
`std::transform` is useful for mapping one range into another or combining two ranges element-wise. The output range must have enough space or use an inserter such as `std::back_inserter`.

**Example:**

```cpp
#include <algorithm>
#include <iterator>
#include <string>
#include <vector>

std::vector<std::string> names = {"ada", "grace"};
std::vector<std::size_t> lengths;

std::transform(names.begin(), names.end(), std::back_inserter(lengths), [](const std::string& s) {
    return s.size();
});
```

**Interview point:** Algorithm output iterator requirements are part of the contract.

**Common mistake:** Passing `result.begin()` when `result` is empty instead of resizing or using `back_inserter`.

---

## 46. What is the difference between `std::back_inserter`, `std::inserter`, and `std::front_inserter`?

**Short answer:** They create output iterators that insert elements at the back, at a specified position, or at the front of a container.

**Detailed answer:**
Inserter adapters let algorithms grow containers instead of writing into existing elements. `std::back_inserter` calls `push_back`, `std::front_inserter` calls `push_front`, and `std::inserter` calls `insert` at a tracked position.

**Example:**

```cpp
#include <algorithm>
#include <iterator>
#include <vector>

std::vector<int> source = {1, 2, 3};
std::vector<int> destination;

std::copy(source.begin(), source.end(), std::back_inserter(destination));
```

**Interview point:** Inserters adapt container insertion APIs to algorithm output-iterator requirements.

**Common mistake:** Using `front_inserter` with containers that do not support `push_front`, such as `std::vector`.

---

## 47. What are polymorphic memory resource containers?

**Short answer:** PMR containers use `std::pmr::polymorphic_allocator` so allocation strategy can be selected at runtime.

**Detailed answer:**
The `std::pmr` namespace provides aliases such as `std::pmr::vector` and `std::pmr::string`. They store a pointer to a `memory_resource`, allowing containers to allocate from arenas, pools, or tracking resources without changing container logic.

**Example:**

```cpp
#include <memory_resource>
#include <vector>

std::byte buffer[4096];
std::pmr::monotonic_buffer_resource arena(buffer, sizeof(buffer));
std::pmr::vector<int> values{&arena};
```

The memory resource must outlive containers using it.

**Interview point:** PMR customizes allocation, not container semantics or ownership.

**Common mistake:** Returning a PMR container that uses a local memory resource.

---

## 48. What is iterator invalidation after `std::vector::erase`?

**Short answer:** Erasing from a vector invalidates iterators and references at or after the erased position.

**Detailed answer:**
`std::vector` stores elements contiguously. When an element is erased, later elements shift left to fill the gap. This changes their addresses and invalidates iterators and references from the erase point onward. Iterators before the erased position remain valid.

**Example:**

```cpp
std::vector<int> values = {1, 2, 3, 4};
auto it = values.begin() + 1;
it = values.erase(it); // returns iterator to element after erased one
```

Use the returned iterator to continue iteration safely.

**Interview point:** Erase invalidation differs across containers because their storage models differ.

**Common mistake:** Incrementing or dereferencing an iterator after erasing it from a vector.

---

## 49. What is the difference between `std::set` and a sorted `std::vector`?

**Short answer:** `std::set` supports logarithmic insertion and stable node addresses; a sorted vector provides compact storage and fast iteration but expensive middle insertions.

**Detailed answer:**
A sorted vector is often faster for small or mostly-read data because it is contiguous and cache-friendly. Lookup can use binary search. `std::set` is better when frequent insertions and erasures must preserve sorted order without shifting many elements.

**Example sorted-vector lookup:**

```cpp
std::vector<int> ids = {1, 3, 5, 7};
bool found = std::binary_search(ids.begin(), ids.end(), 5);
```

**Interview point:** Big-O does not capture cache locality and constant factors.

**Common mistake:** Choosing `std::set` automatically whenever sorted unique values are needed.

---

## 50. How do ranges views differ from owning containers?

**Short answer:** Views usually describe how to access or transform data without owning the underlying elements.

**Detailed answer:**
C++20 ranges views are often lazy and non-owning. They can filter, transform, take, drop, or otherwise adapt a range without allocating a new container. Because they often refer to existing storage, lifetime is critical.

**Example:**

```cpp
auto positives = values | std::views::filter([](int x) {
    return x > 0;
});
```

This view depends on `values` remaining alive and structurally valid while the view is used.

**Interview point:** Views improve composition but do not replace ownership design.

**Common mistake:** Returning a view that refers to a local container.
