# Coding Interview Problems in C/C++

This topic contains practical coding interview problems that are commonly asked for C and C++ roles. Each answer emphasizes the approach, complexity, edge cases, and a clean C++ implementation.

## 1. Reverse a string in place

**Problem:** Given a mutable string, reverse it in place.

**Approach:** Use two pointers: one at the beginning and one at the end. Swap characters while moving toward the center.

**C++ solution:**

```cpp
#include <string>
#include <utility>

void reverseString(std::string& text) {
    std::size_t left = 0;
    std::size_t right = text.size();

    while (left < right) {
        --right;
        std::swap(text[left], text[right]);
        ++left;
    }
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Empty string
- Single-character string
- Even and odd lengths

**Interview follow-up:** How would this change for UTF-8 text? Reversing bytes or `char`s may break multi-byte characters.

---

## 2. Check whether a string is a palindrome

**Problem:** Return whether a string reads the same forward and backward.

**Approach:** Compare characters from both ends moving inward.

**C++ solution:**

```cpp
#include <string_view>

bool isPalindrome(std::string_view text) {
    std::size_t left = 0;
    std::size_t right = text.size();

    while (left < right) {
        --right;
        if (text[left] != text[right]) {
            return false;
        }
        ++left;
    }

    return true;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Empty string is usually considered a palindrome.
- Case sensitivity and punctuation rules should be clarified with the interviewer.

**Common mistake:** Creating a reversed copy when an in-place two-pointer check is enough.

---

## 3. Find the first non-repeating character

**Problem:** Given a string, return the first character that appears exactly once.

**Approach:** Count character frequencies, then scan the string again to find the first character with frequency 1.

**C++ solution:**

```cpp
#include <array>
#include <optional>
#include <string_view>

std::optional<char> firstNonRepeatingChar(std::string_view text) {
    std::array<int, 256> counts{};

    for (unsigned char ch : text) {
        ++counts[ch];
    }

    for (char ch : text) {
        if (counts[static_cast<unsigned char>(ch)] == 1) {
            return ch;
        }
    }

    return std::nullopt;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)` for fixed byte alphabet

**Edge cases:**
- Empty string
- All repeated characters
- Character encoding assumptions

**Interview follow-up:** How would you handle Unicode or a very large alphabet?

---

## 4. Detect a cycle in a linked list

**Problem:** Given the head of a singly linked list, determine whether the list contains a cycle.

**Approach:** Use Floyd's tortoise and hare algorithm. Move one pointer one step at a time and another pointer two steps at a time. If they meet, there is a cycle.

**C++ solution:**

```cpp
struct Node {
    int value;
    Node* next;
};

bool hasCycle(const Node* head) {
    const Node* slow = head;
    const Node* fast = head;

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;

        if (slow == fast) {
            return true;
        }
    }

    return false;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Common mistake:** Using a hash set when the interviewer expects constant extra space.

---

## 5. Reverse a singly linked list

**Problem:** Reverse a singly linked list and return the new head.

**Approach:** Iterate through the list while reversing each `next` pointer.

**C++ solution:**

```cpp
struct Node {
    int value;
    Node* next;
};

Node* reverseList(Node* head) {
    Node* previous = nullptr;
    Node* current = head;

    while (current != nullptr) {
        Node* next = current->next;
        current->next = previous;
        previous = current;
        current = next;
    }

    return previous;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Empty list
- Single-node list
- Two-node list

**Interview follow-up:** Can you solve it recursively? What are the stack-space tradeoffs?

---

## 6. Find two numbers that sum to a target

**Problem:** Given an array of integers and a target, return indices of two numbers that sum to the target.

**Approach:** Use a hash map from value to index. For each number, check whether its complement was seen earlier.

**C++ solution:**

```cpp
#include <optional>
#include <unordered_map>
#include <utility>
#include <vector>

std::optional<std::pair<int, int>> twoSum(const std::vector<int>& values, int target) {
    std::unordered_map<int, int> indexByValue;

    for (int i = 0; i < static_cast<int>(values.size()); ++i) {
        int complement = target - values[i];

        auto it = indexByValue.find(complement);
        if (it != indexByValue.end()) {
            return std::pair{it->second, i};
        }

        indexByValue[values[i]] = i;
    }

    return std::nullopt;
}
```

**Complexity:**
- Time: average `O(n)`
- Space: `O(n)`

**Edge cases:**
- Duplicate values
- Negative values
- No valid pair

**Common mistake:** Inserting the current value before checking the complement, which can accidentally reuse the same element.

---

## 7. Find the maximum subarray sum

**Problem:** Given an array of integers, find the maximum sum of a contiguous subarray.

**Approach:** Use Kadane's algorithm. At each position, decide whether to extend the previous subarray or start a new one.

**C++ solution:**

```cpp
#include <algorithm>
#include <stdexcept>
#include <vector>

int maxSubarraySum(const std::vector<int>& values) {
    if (values.empty()) {
        throw std::invalid_argument("values must not be empty");
    }

    int bestEndingHere = values[0];
    int bestOverall = values[0];

    for (std::size_t i = 1; i < values.size(); ++i) {
        bestEndingHere = std::max(values[i], bestEndingHere + values[i]);
        bestOverall = std::max(bestOverall, bestEndingHere);
    }

    return bestOverall;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- All negative numbers
- Single element
- Empty input policy

---

## 8. Validate balanced parentheses

**Problem:** Given a string containing brackets, determine whether every opening bracket is closed in the correct order.

**Approach:** Use a stack. Push opening brackets. For closing brackets, check whether the stack top is the matching opener.

**C++ solution:**

```cpp
#include <stack>
#include <string_view>

bool isBalanced(std::string_view text) {
    std::stack<char> stack;

    for (char ch : text) {
        if (ch == '(' || ch == '[' || ch == '{') {
            stack.push(ch);
        } else if (ch == ')' || ch == ']' || ch == '}') {
            if (stack.empty()) {
                return false;
            }

            char open = stack.top();
            stack.pop();

            if ((ch == ')' && open != '(') ||
                (ch == ']' && open != '[') ||
                (ch == '}' && open != '{')) {
                return false;
            }
        }
    }

    return stack.empty();
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(n)`

**Common mistake:** Only counting brackets instead of checking correct nesting order.

---

## 9. Binary search in a sorted array

**Problem:** Given a sorted array and a target, return the index of the target or `-1` if not found.

**Approach:** Repeatedly compare the target with the middle element and discard half of the search space.

**C++ solution:**

```cpp
#include <vector>

int binarySearch(const std::vector<int>& values, int target) {
    int left = 0;
    int right = static_cast<int>(values.size()) - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (values[mid] == target) {
            return mid;
        }

        if (values[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}
```

**Complexity:**
- Time: `O(log n)`
- Space: `O(1)`

**Common mistake:** Computing `mid` as `(left + right) / 2`, which can overflow for very large indices.

---

## 10. Find the lowest common ancestor in a binary search tree

**Problem:** Given a binary search tree and two values, find their lowest common ancestor.

**Approach:** Use the BST property. If both values are smaller than the current node, go left. If both are larger, go right. Otherwise, the current node is the split point and therefore the LCA.

**C++ solution:**

```cpp
struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

const TreeNode* lowestCommonAncestor(const TreeNode* root, int a, int b) {
    const TreeNode* current = root;

    while (current != nullptr) {
        if (a < current->value && b < current->value) {
            current = current->left;
        } else if (a > current->value && b > current->value) {
            current = current->right;
        } else {
            return current;
        }
    }

    return nullptr;
}
```

**Complexity:**
- Time: `O(h)`, where `h` is tree height
- Space: `O(1)`

**Edge cases:**
- Empty tree
- One value is ancestor of the other
- Values may not exist in the tree, depending on problem definition

---

## 11. Merge two sorted arrays

**Problem:** Given two sorted arrays, return a new sorted array containing all elements from both inputs.

**Approach:** Use two indices. Repeatedly take the smaller current element, then append any remaining elements.

**C++ solution:**

```cpp
#include <vector>

std::vector<int> mergeSortedArrays(const std::vector<int>& a, const std::vector<int>& b) {
    std::vector<int> result;
    result.reserve(a.size() + b.size());

    std::size_t i = 0;
    std::size_t j = 0;

    while (i < a.size() && j < b.size()) {
        if (a[i] <= b[j]) {
            result.push_back(a[i++]);
        } else {
            result.push_back(b[j++]);
        }
    }

    while (i < a.size()) {
        result.push_back(a[i++]);
    }

    while (j < b.size()) {
        result.push_back(b[j++]);
    }

    return result;
}
```

**Complexity:**
- Time: `O(n + m)`
- Space: `O(n + m)` for the output

**Edge cases:**
- One input is empty
- Duplicate values
- Negative values

**Interview follow-up:** How would you merge in place if the first array has enough unused capacity at the end?

---

## 12. Check whether two strings are anagrams

**Problem:** Given two strings, determine whether they contain the same characters with the same frequencies.

**Approach:** Count characters in one string and subtract counts using the other string.

**C++ solution:**

```cpp
#include <array>
#include <string_view>

bool areAnagrams(std::string_view a, std::string_view b) {
    if (a.size() != b.size()) {
        return false;
    }

    std::array<int, 256> counts{};

    for (unsigned char ch : a) {
        ++counts[ch];
    }

    for (unsigned char ch : b) {
        --counts[ch];
        if (counts[ch] < 0) {
            return false;
        }
    }

    return true;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)` for fixed byte alphabet

**Edge cases:**
- Empty strings
- Different lengths
- Character encoding and case sensitivity

**Common mistake:** Sorting both strings without discussing the `O(n log n)` cost or character-set assumptions.

---

## 13. Find the middle node of a linked list

**Problem:** Given the head of a singly linked list, return the middle node.

**Approach:** Use a slow pointer and a fast pointer. Move slow by one step and fast by two steps. When fast reaches the end, slow is at the middle.

**C++ solution:**

```cpp
struct Node {
    int value;
    Node* next;
};

const Node* middleNode(const Node* head) {
    const Node* slow = head;
    const Node* fast = head;

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Empty list
- Single-node list
- Even-length list, where this returns the second middle node

**Interview follow-up:** How would you return the first middle node for even-length lists?

---

## 14. Inorder traversal of a binary tree

**Problem:** Given a binary tree, return the inorder traversal of its values.

**Approach:** Recursively visit left subtree, current node, then right subtree.

**C++ solution:**

```cpp
#include <vector>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

void inorder(const TreeNode* node, std::vector<int>& result) {
    if (node == nullptr) {
        return;
    }

    inorder(node->left, result);
    result.push_back(node->value);
    inorder(node->right, result);
}

std::vector<int> inorderTraversal(const TreeNode* root) {
    std::vector<int> result;
    inorder(root, result);
    return result;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack, where `h` is tree height

**Edge cases:**
- Empty tree
- Skewed tree
- Single-node tree

**Interview follow-up:** How would you implement this iteratively with an explicit stack?

---

## 15. Find the top K frequent elements

**Problem:** Given an array of integers and an integer `k`, return the `k` most frequent elements.

**Approach:** Count frequencies with a hash map, then keep the best `k` entries using a min-heap ordered by frequency.

**C++ solution:**

```cpp
#include <queue>
#include <unordered_map>
#include <utility>
#include <vector>

std::vector<int> topKFrequent(const std::vector<int>& values, int k) {
    std::unordered_map<int, int> frequency;
    for (int value : values) {
        ++frequency[value];
    }

    using Entry = std::pair<int, int>; // frequency, value
    std::priority_queue<Entry, std::vector<Entry>, std::greater<Entry>> heap;

    for (const auto& [value, count] : frequency) {
        heap.push({count, value});
        if (static_cast<int>(heap.size()) > k) {
            heap.pop();
        }
    }

    std::vector<int> result;
    while (!heap.empty()) {
        result.push_back(heap.top().second);
        heap.pop();
    }

    return result;
}
```

**Complexity:**
- Time: `O(n log k)`
- Space: `O(n)` for the frequency map and heap storage

**Edge cases:**
- `k` is zero
- `k` is larger than the number of unique elements
- Ties in frequency, where output order may need clarification

**Common mistake:** Sorting all unique values by frequency without mentioning the `O(u log u)` cost, where `u` is the number of unique values.

---

## 16. Find the first missing positive integer

**Problem:** Given an unsorted array of integers, find the smallest missing positive integer.

**Approach:** Place each value `x` in index `x - 1` when `x` is in the range `[1, n]`. After rearrangement, the first index where `values[i] != i + 1` gives the answer.

**C++ solution:**

```cpp
#include <vector>
#include <utility>

int firstMissingPositive(std::vector<int>& values) {
    const int n = static_cast<int>(values.size());

    for (int i = 0; i < n; ++i) {
        while (values[i] >= 1 && values[i] <= n && values[values[i] - 1] != values[i]) {
            std::swap(values[i], values[values[i] - 1]);
        }
    }

    for (int i = 0; i < n; ++i) {
        if (values[i] != i + 1) {
            return i + 1;
        }
    }

    return n + 1;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)` extra space

**Edge cases:**
- Empty array
- Negative numbers and zero
- Duplicate values
- Array already contains `1..n`

**Common mistake:** Using sorting and getting `O(n log n)` when the interviewer asks for linear time and constant extra space.

---

## 17. Detect the start of a cycle in a linked list

**Problem:** Given the head of a singly linked list, return the node where the cycle begins, or `nullptr` if there is no cycle.

**Approach:** Use Floyd's algorithm. First detect whether slow and fast pointers meet. Then move one pointer back to head and advance both one step at a time; their meeting point is the cycle start.

**C++ solution:**

```cpp
struct Node {
    int value;
    Node* next;
};

Node* cycleStart(Node* head) {
    Node* slow = head;
    Node* fast = head;

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;

        if (slow == fast) {
            Node* current = head;
            while (current != slow) {
                current = current->next;
                slow = slow->next;
            }
            return current;
        }
    }

    return nullptr;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Empty list
- No cycle
- Cycle starts at head
- Single node pointing to itself

**Interview follow-up:** Why does moving one pointer to the head after the meeting point find the cycle entry?

---

## 18. Merge intervals

**Problem:** Given a list of intervals, merge all overlapping intervals.

**Approach:** Sort intervals by start value, then scan from left to right. If the next interval overlaps the current merged interval, extend the end; otherwise, start a new interval.

**C++ solution:**

```cpp
#include <algorithm>
#include <vector>

struct Interval {
    int start;
    int end;
};

std::vector<Interval> mergeIntervals(std::vector<Interval> intervals) {
    if (intervals.empty()) {
        return {};
    }

    std::sort(intervals.begin(), intervals.end(), [](const Interval& a, const Interval& b) {
        return a.start < b.start;
    });

    std::vector<Interval> merged;
    merged.push_back(intervals[0]);

    for (std::size_t i = 1; i < intervals.size(); ++i) {
        Interval& last = merged.back();
        if (intervals[i].start <= last.end) {
            last.end = std::max(last.end, intervals[i].end);
        } else {
            merged.push_back(intervals[i]);
        }
    }

    return merged;
}
```

**Complexity:**
- Time: `O(n log n)` due to sorting
- Space: `O(n)` for output

**Edge cases:**
- Empty input
- Already merged intervals
- Touching intervals, depending on whether `[1, 2]` and `[2, 3]` should merge
- Nested intervals

**Common mistake:** Trying to merge without sorting first.

---

## 19. Find the kth largest element

**Problem:** Given an array of integers and an integer `k`, return the kth largest element.

**Approach:** Use a min-heap of size `k`. Keep the largest `k` values seen so far. The heap top is the kth largest.

**C++ solution:**

```cpp
#include <queue>
#include <stdexcept>
#include <vector>

int kthLargest(const std::vector<int>& values, int k) {
    if (k <= 0 || k > static_cast<int>(values.size())) {
        throw std::invalid_argument("invalid k");
    }

    std::priority_queue<int, std::vector<int>, std::greater<int>> heap;

    for (int value : values) {
        heap.push(value);
        if (static_cast<int>(heap.size()) > k) {
            heap.pop();
        }
    }

    return heap.top();
}
```

**Complexity:**
- Time: `O(n log k)`
- Space: `O(k)`

**Edge cases:**
- `k == 1`
- `k == values.size()`
- Duplicate values
- Invalid `k`

**Interview follow-up:** How would you solve this with quickselect, and what is the average-case complexity?

---

## 20. Serialize and deserialize a binary tree

**Problem:** Convert a binary tree to a string representation and reconstruct the same tree from that representation.

**Approach:** Use preorder traversal with a marker for null children. During deserialization, read tokens in the same order and recursively rebuild the tree.

**C++ solution:**

```cpp
#include <sstream>
#include <string>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

void serialize(const TreeNode* node, std::ostream& out) {
    if (node == nullptr) {
        out << "# ";
        return;
    }

    out << node->value << ' ';
    serialize(node->left, out);
    serialize(node->right, out);
}

std::string serializeTree(const TreeNode* root) {
    std::ostringstream out;
    serialize(root, out);
    return out.str();
}

TreeNode* deserialize(std::istream& in) {
    std::string token;
    if (!(in >> token) || token == "#") {
        return nullptr;
    }

    TreeNode* node = new TreeNode{std::stoi(token), nullptr, nullptr};
    node->left = deserialize(in);
    node->right = deserialize(in);
    return node;
}

TreeNode* deserializeTree(const std::string& text) {
    std::istringstream in(text);
    return deserialize(in);
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack, plus output storage

**Edge cases:**
- Empty tree
- Negative values
- Skewed tree
- Duplicate values

**Common mistake:** Serializing only inorder traversal. Inorder alone is not enough to reconstruct an arbitrary binary tree.

---

## 21. Validate a binary search tree

**Problem:** Given the root of a binary tree, determine whether it is a valid binary search tree.

**Approach:** Use recursive lower and upper bounds. Every node must be greater than all allowed lower values and less than all allowed upper values. Passing bounds avoids the common mistake of checking only parent-child relationships.

**C++ solution:**

```cpp
#include <limits>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

bool isValidBst(const TreeNode* node, long long low, long long high) {
    if (node == nullptr) {
        return true;
    }

    if (node->value <= low || node->value >= high) {
        return false;
    }

    return isValidBst(node->left, low, node->value) &&
           isValidBst(node->right, node->value, high);
}

bool isValidBst(const TreeNode* root) {
    return isValidBst(
        root,
        std::numeric_limits<long long>::lowest(),
        std::numeric_limits<long long>::max()
    );
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack

**Edge cases:**
- Empty tree
- Duplicate values, depending on the BST definition
- Values equal to `INT_MIN` or `INT_MAX`
- A violation deep in a subtree

**Common mistake:** Checking only `node->left->value < node->value < node->right->value`, which misses deeper violations.

---

## 22. Find the longest substring without repeating characters

**Problem:** Given a string, return the length of the longest substring that contains no repeated characters.

**Approach:** Use a sliding window with the last seen index of each character. Move the left side of the window forward when a repeated character appears inside the current window.

**C++ solution:**

```cpp
#include <algorithm>
#include <array>
#include <string>

int longestUniqueSubstring(const std::string& text) {
    std::array<int, 256> lastSeen;
    lastSeen.fill(-1);

    int best = 0;
    int left = 0;

    for (int right = 0; right < static_cast<int>(text.size()); ++right) {
        unsigned char ch = static_cast<unsigned char>(text[right]);
        if (lastSeen[ch] >= left) {
            left = lastSeen[ch] + 1;
        }

        lastSeen[ch] = right;
        best = std::max(best, right - left + 1);
    }

    return best;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)` for fixed-size byte alphabet

**Edge cases:**
- Empty string
- All characters same
- All characters unique
- Non-ASCII text, where byte-based logic may not match character-based expectations

**Common mistake:** Moving the left pointer backward when seeing a repeated character that is already outside the current window.

---

## 23. Search in a rotated sorted array

**Problem:** Given a sorted array rotated at an unknown pivot, return the index of a target value or `-1` if it is absent.

**Approach:** Use modified binary search. At each step, one half of the array is sorted. Decide whether the target lies in that sorted half, then discard the other half.

**C++ solution:**

```cpp
#include <vector>

int searchRotated(const std::vector<int>& values, int target) {
    int left = 0;
    int right = static_cast<int>(values.size()) - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (values[mid] == target) {
            return mid;
        }

        if (values[left] <= values[mid]) {
            if (values[left] <= target && target < values[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } else {
            if (values[mid] < target && target <= values[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }

    return -1;
}
```

**Complexity:**
- Time: `O(log n)` when values are distinct
- Space: `O(1)`

**Edge cases:**
- Empty array
- Not rotated
- Single element
- Target at pivot or boundaries

**Interview follow-up:** How does the solution change if duplicate values are allowed?

---

## 24. Find the lowest common ancestor in a binary tree

**Problem:** Given a binary tree and two target nodes, return their lowest common ancestor.

**Approach:** Recursively search left and right subtrees. If both sides find a target, the current node is the lowest common ancestor. If only one side finds a target, return that result upward.

**C++ solution:**

```cpp
struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* a, TreeNode* b) {
    if (root == nullptr || root == a || root == b) {
        return root;
    }

    TreeNode* left = lowestCommonAncestor(root->left, a, b);
    TreeNode* right = lowestCommonAncestor(root->right, a, b);

    if (left != nullptr && right != nullptr) {
        return root;
    }

    return left != nullptr ? left : right;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack

**Edge cases:**
- One target is the ancestor of the other
- One or both targets are missing, depending on problem definition
- Empty tree
- Highly skewed tree

**Common mistake:** Using the BST-specific LCA approach on an ordinary binary tree.

---

## 25. Count islands in a grid

**Problem:** Given a grid of `0`s and `1`s, count connected groups of `1`s. Connections are horizontal and vertical.

**Approach:** Scan every cell. When an unvisited land cell is found, increment the count and run DFS or BFS to mark the whole island as visited.

**C++ solution:**

```cpp
#include <vector>

void dfs(std::vector<std::vector<char>>& grid, int row, int col) {
    const int rows = static_cast<int>(grid.size());
    const int cols = static_cast<int>(grid[0].size());

    if (row < 0 || row >= rows || col < 0 || col >= cols || grid[row][col] != '1') {
        return;
    }

    grid[row][col] = '0';

    dfs(grid, row + 1, col);
    dfs(grid, row - 1, col);
    dfs(grid, row, col + 1);
    dfs(grid, row, col - 1);
}

int countIslands(std::vector<std::vector<char>>& grid) {
    if (grid.empty() || grid[0].empty()) {
        return 0;
    }

    int count = 0;
    for (int row = 0; row < static_cast<int>(grid.size()); ++row) {
        for (int col = 0; col < static_cast<int>(grid[row].size()); ++col) {
            if (grid[row][col] == '1') {
                ++count;
                dfs(grid, row, col);
            }
        }
    }

    return count;
}
```

**Complexity:**
- Time: `O(rows * cols)`
- Space: `O(rows * cols)` worst-case recursion depth

**Edge cases:**
- Empty grid
- All water
- All land
- Single row or single column

**Interview follow-up:** How would you avoid recursion depth problems for a very large grid?

---

## 26. Remove duplicates from a sorted array

**Problem:** Given a sorted array, remove duplicates in place and return the number of unique elements.

**Approach:** Use two pointers. One pointer scans the array, and the other tracks the next position for a unique value.

**C++ solution:**

```cpp
#include <vector>

int removeDuplicates(std::vector<int>& values) {
    if (values.empty()) {
        return 0;
    }

    int write = 1;
    for (int read = 1; read < static_cast<int>(values.size()); ++read) {
        if (values[read] != values[write - 1]) {
            values[write] = values[read];
            ++write;
        }
    }

    return write;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Empty array
- All elements unique
- All elements identical
- Single element

**Common mistake:** Allocating a new array when the problem requires in-place modification.

---

## 27. Move zeroes to the end

**Problem:** Given an array, move all zeroes to the end while preserving the relative order of nonzero elements.

**Approach:** Compact nonzero values to the front, then fill the remaining positions with zeroes.

**C++ solution:**

```cpp
#include <vector>

void moveZeroes(std::vector<int>& values) {
    int write = 0;

    for (int value : values) {
        if (value != 0) {
            values[write] = value;
            ++write;
        }
    }

    while (write < static_cast<int>(values.size())) {
        values[write] = 0;
        ++write;
    }
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- No zeroes
- All zeroes
- Empty array
- Zeroes already at the end

**Interview follow-up:** How would the solution change if preserving order was not required?

---

## 28. Product of array except self

**Problem:** Given an array of integers, return an array where each position contains the product of all other elements, without using division.

**Approach:** Compute prefix products from the left and suffix products from the right. The result at each index is the product of values before and after it.

**C++ solution:**

```cpp
#include <vector>

std::vector<int> productExceptSelf(const std::vector<int>& values) {
    const int n = static_cast<int>(values.size());
    std::vector<int> result(n, 1);

    int prefix = 1;
    for (int i = 0; i < n; ++i) {
        result[i] = prefix;
        prefix *= values[i];
    }

    int suffix = 1;
    for (int i = n - 1; i >= 0; --i) {
        result[i] *= suffix;
        suffix *= values[i];
    }

    return result;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)` extra space if output does not count

**Edge cases:**
- One zero
- Multiple zeroes
- Negative numbers
- Single element, depending on problem definition

**Common mistake:** Using division and failing when the input contains zero.

---

## 29. Three sum

**Problem:** Given an array of integers, return all unique triplets whose sum is zero.

**Approach:** Sort the array. For each fixed first value, use two pointers to find pairs that complete the triplet. Skip duplicate values to avoid duplicate triplets.

**C++ solution:**

```cpp
#include <algorithm>
#include <vector>

std::vector<std::vector<int>> threeSum(std::vector<int> values) {
    std::sort(values.begin(), values.end());
    std::vector<std::vector<int>> result;

    for (int i = 0; i < static_cast<int>(values.size()); ++i) {
        if (i > 0 && values[i] == values[i - 1]) {
            continue;
        }

        int left = i + 1;
        int right = static_cast<int>(values.size()) - 1;

        while (left < right) {
            int sum = values[i] + values[left] + values[right];
            if (sum == 0) {
                result.push_back({values[i], values[left], values[right]});
                ++left;
                --right;
                while (left < right && values[left] == values[left - 1]) {
                    ++left;
                }
                while (left < right && values[right] == values[right + 1]) {
                    --right;
                }
            } else if (sum < 0) {
                ++left;
            } else {
                --right;
            }
        }
    }

    return result;
}
```

**Complexity:**
- Time: `O(n^2)`
- Space: `O(1)` extra space excluding output

**Edge cases:**
- Fewer than three values
- Duplicate values
- All zeroes
- No valid triplet

**Common mistake:** Forgetting to skip duplicates after finding a valid triplet.

---

## 30. Container with most water

**Problem:** Given heights of vertical lines, find the maximum area of water that can be contained by two lines.

**Approach:** Use two pointers at both ends. Compute area, then move the pointer at the shorter line because the limiting height must improve to find a larger area.

**C++ solution:**

```cpp
#include <algorithm>
#include <vector>

int maxArea(const std::vector<int>& height) {
    int left = 0;
    int right = static_cast<int>(height.size()) - 1;
    int best = 0;

    while (left < right) {
        int width = right - left;
        int current = width * std::min(height[left], height[right]);
        best = std::max(best, current);

        if (height[left] < height[right]) {
            ++left;
        } else {
            --right;
        }
    }

    return best;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Fewer than two lines
- Equal heights
- Strictly increasing or decreasing heights

**Interview follow-up:** Why is it safe to move the pointer at the shorter line?

---

## 31. Minimum window substring

**Problem:** Given strings `s` and `t`, return the shortest substring of `s` containing all characters of `t` including multiplicities.

**Approach:** Use a sliding window with frequency counts. Expand the right side until all required characters are covered, then shrink from the left while the window remains valid.

**C++ solution:**

```cpp
#include <array>
#include <climits>
#include <string>

std::string minWindow(const std::string& s, const std::string& t) {
    if (t.empty()) {
        return "";
    }

    std::array<int, 256> need{};
    for (unsigned char ch : t) {
        ++need[ch];
    }

    int missing = static_cast<int>(t.size());
    int bestStart = 0;
    int bestLength = INT_MAX;
    int left = 0;

    for (int right = 0; right < static_cast<int>(s.size()); ++right) {
        unsigned char r = static_cast<unsigned char>(s[right]);
        if (need[r] > 0) {
            --missing;
        }
        --need[r];

        while (missing == 0) {
            int length = right - left + 1;
            if (length < bestLength) {
                bestStart = left;
                bestLength = length;
            }

            unsigned char l = static_cast<unsigned char>(s[left]);
            ++need[l];
            if (need[l] > 0) {
                ++missing;
            }
            ++left;
        }
    }

    return bestLength == INT_MAX ? "" : s.substr(bestStart, bestLength);
}
```

**Complexity:**
- Time: `O(n + m)`
- Space: `O(1)` for byte alphabet

**Edge cases:**
- `t` is empty
- `s` is shorter than `t`
- Repeated characters in `t`
- No valid window

**Common mistake:** Tracking only distinct characters and ignoring multiplicity.

---

## 32. Add two numbers represented by linked lists

**Problem:** Given two non-empty linked lists representing non-negative integers in reverse digit order, return their sum as a linked list in the same format.

**Approach:** Traverse both lists, add digits with carry, and create result nodes as needed.

**C++ solution:**

```cpp
struct ListNode {
    int value;
    ListNode* next;
};

ListNode* addTwoNumbers(ListNode* a, ListNode* b) {
    ListNode dummy{0, nullptr};
    ListNode* tail = &dummy;
    int carry = 0;

    while (a != nullptr || b != nullptr || carry != 0) {
        int sum = carry;
        if (a != nullptr) {
            sum += a->value;
            a = a->next;
        }
        if (b != nullptr) {
            sum += b->value;
            b = b->next;
        }

        carry = sum / 10;
        tail->next = new ListNode{sum % 10, nullptr};
        tail = tail->next;
    }

    return dummy.next;
}
```

**Complexity:**
- Time: `O(max(m, n))`
- Space: `O(max(m, n))` for the result list

**Edge cases:**
- Different list lengths
- Final carry
- One list is null, depending on problem definition
- Sum is zero

**Common mistake:** Forgetting to append a final carry node.

---

## 33. Remove nth node from end of list

**Problem:** Remove the nth node from the end of a singly linked list and return the new head.

**Approach:** Use two pointers separated by `n` nodes. When the fast pointer reaches the end, the slow pointer is just before the node to remove.

**C++ solution:**

```cpp
struct ListNode {
    int value;
    ListNode* next;
};

ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy{0, head};
    ListNode* fast = &dummy;
    ListNode* slow = &dummy;

    for (int i = 0; i < n; ++i) {
        fast = fast->next;
    }

    while (fast->next != nullptr) {
        fast = fast->next;
        slow = slow->next;
    }

    ListNode* removed = slow->next;
    slow->next = removed->next;
    delete removed;

    return dummy.next;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(1)`

**Edge cases:**
- Removing the head
- Single-node list
- Removing the tail
- Invalid `n`, if inputs are not guaranteed valid

**Interview follow-up:** Why does a dummy node simplify head removal?

---

## 34. Check whether a binary tree is balanced

**Problem:** Determine whether a binary tree is height-balanced, meaning every node's left and right subtree heights differ by at most one.

**Approach:** Compute height bottom-up. Return a sentinel value when an unbalanced subtree is found so work can stop early.

**C++ solution:**

```cpp
#include <algorithm>
#include <cstdlib>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

int heightOrUnbalanced(const TreeNode* node) {
    if (node == nullptr) {
        return 0;
    }

    int left = heightOrUnbalanced(node->left);
    if (left == -1) {
        return -1;
    }

    int right = heightOrUnbalanced(node->right);
    if (right == -1) {
        return -1;
    }

    if (std::abs(left - right) > 1) {
        return -1;
    }

    return std::max(left, right) + 1;
}

bool isBalanced(const TreeNode* root) {
    return heightOrUnbalanced(root) != -1;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack

**Edge cases:**
- Empty tree
- Single node
- Skewed tree
- Unbalance deep in a subtree

**Common mistake:** Recomputing subtree heights for every node and getting `O(n^2)` on skewed trees.

---

## 35. Level order traversal of a binary tree

**Problem:** Return the values of a binary tree level by level from top to bottom.

**Approach:** Use BFS with a queue. For each level, process the current queue size before moving to the next level.

**C++ solution:**

```cpp
#include <queue>
#include <vector>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

std::vector<std::vector<int>> levelOrder(TreeNode* root) {
    std::vector<std::vector<int>> result;
    if (root == nullptr) {
        return result;
    }

    std::queue<TreeNode*> queue;
    queue.push(root);

    while (!queue.empty()) {
        int levelSize = static_cast<int>(queue.size());
        std::vector<int> level;

        for (int i = 0; i < levelSize; ++i) {
            TreeNode* node = queue.front();
            queue.pop();
            level.push_back(node->value);

            if (node->left != nullptr) {
                queue.push(node->left);
            }
            if (node->right != nullptr) {
                queue.push(node->right);
            }
        }

        result.push_back(std::move(level));
    }

    return result;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(w)`, where `w` is maximum tree width

**Edge cases:**
- Empty tree
- Single node
- Complete tree
- Highly skewed tree

**Common mistake:** Not separating levels and returning one flat BFS order when the problem asks for grouped levels.

---

## 36. Word break

**Problem:** Given a string and a dictionary, determine whether the string can be segmented into dictionary words.

**Approach:** Use dynamic programming. `dp[i]` means the prefix ending before index `i` can be segmented.

**C++ solution:**

```cpp
#include <string>
#include <unordered_set>
#include <vector>

bool wordBreak(const std::string& text, const std::unordered_set<std::string>& dictionary) {
    std::vector<bool> dp(text.size() + 1, false);
    dp[0] = true;

    for (std::size_t i = 1; i <= text.size(); ++i) {
        for (std::size_t j = 0; j < i; ++j) {
            if (dp[j] && dictionary.contains(text.substr(j, i - j))) {
                dp[i] = true;
                break;
            }
        }
    }

    return dp[text.size()];
}
```

**Complexity:**
- Time: `O(n^3)` in many implementations because substring creation costs `O(n)`
- Space: `O(n)`

**Edge cases:**
- Empty string
- Empty dictionary
- Overlapping word choices
- Repeated prefixes

**Interview follow-up:** How could you reduce substring overhead with string views, trie lookup, or maximum word length?

---

## 37. Coin change minimum coins

**Problem:** Given coin denominations and an amount, return the minimum number of coins needed to make that amount, or `-1` if impossible.

**Approach:** Use bottom-up dynamic programming where `dp[x]` is the minimum coins needed for amount `x`.

**C++ solution:**

```cpp
#include <algorithm>
#include <vector>

int coinChange(const std::vector<int>& coins, int amount) {
    const int impossible = amount + 1;
    std::vector<int> dp(amount + 1, impossible);
    dp[0] = 0;

    for (int value = 1; value <= amount; ++value) {
        for (int coin : coins) {
            if (coin <= value) {
                dp[value] = std::min(dp[value], dp[value - coin] + 1);
            }
        }
    }

    return dp[amount] == impossible ? -1 : dp[amount];
}
```

**Complexity:**
- Time: `O(amount * number_of_coins)`
- Space: `O(amount)`

**Edge cases:**
- Amount is zero
- No possible combination
- Coin denomination larger than amount
- Duplicate coin denominations

**Common mistake:** Using a greedy algorithm for arbitrary coin systems; greedy is not always optimal.

---

## 38. Longest increasing subsequence

**Problem:** Given an array of integers, return the length of the longest strictly increasing subsequence.

**Approach:** Maintain a `tails` array where `tails[i]` is the smallest possible tail value of an increasing subsequence of length `i + 1`.

**C++ solution:**

```cpp
#include <algorithm>
#include <vector>

int lengthOfLis(const std::vector<int>& values) {
    std::vector<int> tails;

    for (int value : values) {
        auto it = std::lower_bound(tails.begin(), tails.end(), value);
        if (it == tails.end()) {
            tails.push_back(value);
        } else {
            *it = value;
        }
    }

    return static_cast<int>(tails.size());
}
```

**Complexity:**
- Time: `O(n log n)`
- Space: `O(n)`

**Edge cases:**
- Empty input
- All decreasing
- All increasing
- Duplicate values

**Interview follow-up:** How would you reconstruct the actual subsequence, not just its length?

---

## 39. Course schedule cycle detection

**Problem:** Given courses and prerequisite pairs, determine whether all courses can be completed.

**Approach:** Model courses as a directed graph. Use Kahn's topological sort: repeatedly take courses with indegree zero. If all courses are processed, there is no cycle.

**C++ solution:**

```cpp
#include <queue>
#include <utility>
#include <vector>

bool canFinish(int courseCount, const std::vector<std::pair<int, int>>& prerequisites) {
    std::vector<std::vector<int>> graph(courseCount);
    std::vector<int> indegree(courseCount, 0);

    for (auto [course, prerequisite] : prerequisites) {
        graph[prerequisite].push_back(course);
        ++indegree[course];
    }

    std::queue<int> ready;
    for (int course = 0; course < courseCount; ++course) {
        if (indegree[course] == 0) {
            ready.push(course);
        }
    }

    int processed = 0;
    while (!ready.empty()) {
        int course = ready.front();
        ready.pop();
        ++processed;

        for (int next : graph[course]) {
            --indegree[next];
            if (indegree[next] == 0) {
                ready.push(next);
            }
        }
    }

    return processed == courseCount;
}
```

**Complexity:**
- Time: `O(V + E)`
- Space: `O(V + E)`

**Edge cases:**
- No prerequisites
- Self-dependency
- Disconnected graph
- Multiple independent valid orders

**Common mistake:** Treating prerequisites as an undirected graph.

---

## 40. Clone an undirected graph

**Problem:** Given a node in a connected undirected graph, create a deep copy of the graph.

**Approach:** Use DFS or BFS with a map from original nodes to cloned nodes. Create each clone once, then connect cloned neighbors.

**C++ solution:**

```cpp
#include <unordered_map>
#include <vector>

struct GraphNode {
    int value;
    std::vector<GraphNode*> neighbors;
};

GraphNode* cloneGraph(GraphNode* node, std::unordered_map<GraphNode*, GraphNode*>& clones) {
    if (node == nullptr) {
        return nullptr;
    }

    if (clones.contains(node)) {
        return clones[node];
    }

    GraphNode* copy = new GraphNode{node->value, {}};
    clones[node] = copy;

    for (GraphNode* neighbor : node->neighbors) {
        copy->neighbors.push_back(cloneGraph(neighbor, clones));
    }

    return copy;
}

GraphNode* cloneGraph(GraphNode* node) {
    std::unordered_map<GraphNode*, GraphNode*> clones;
    return cloneGraph(node, clones);
}
```

**Complexity:**
- Time: `O(V + E)`
- Space: `O(V)` for the clone map and recursion stack in the worst case

**Edge cases:**
- Null input
- Single node with no neighbors
- Self-loop
- Cycles in the graph

**Common mistake:** Recursively cloning neighbors without a visited map, causing infinite recursion on cycles.

---

## 41. Subarray sum equals K

**Problem:** Given an integer array and a target `k`, count the number of continuous subarrays whose sum equals `k`.

**Approach:** Use prefix sums. If the current prefix sum is `sum`, then a previous prefix sum of `sum - k` forms a subarray ending at the current index.

**C++ solution:**

```cpp
#include <unordered_map>
#include <vector>

int subarraySum(const std::vector<int>& values, int k) {
    std::unordered_map<int, int> seen;
    seen[0] = 1;

    int sum = 0;
    int count = 0;

    for (int value : values) {
        sum += value;
        if (seen.contains(sum - k)) {
            count += seen[sum - k];
        }
        ++seen[sum];
    }

    return count;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(n)`

**Edge cases:**
- Negative numbers
- Zero target
- Empty array
- Multiple overlapping subarrays

**Common mistake:** Using a sliding window even though negative numbers break the monotonic window property.

---

## 42. Set matrix zeroes

**Problem:** Given a matrix, if an element is zero, set its entire row and column to zero in place.

**Approach:** Use the first row and first column as marker storage, while separately remembering whether they originally contained zero.

**C++ solution:**

```cpp
#include <vector>

void setZeroes(std::vector<std::vector<int>>& matrix) {
    if (matrix.empty() || matrix[0].empty()) {
        return;
    }

    const int rows = static_cast<int>(matrix.size());
    const int cols = static_cast<int>(matrix[0].size());
    bool firstRowZero = false;
    bool firstColZero = false;

    for (int col = 0; col < cols; ++col) {
        firstRowZero = firstRowZero || matrix[0][col] == 0;
    }
    for (int row = 0; row < rows; ++row) {
        firstColZero = firstColZero || matrix[row][0] == 0;
    }

    for (int row = 1; row < rows; ++row) {
        for (int col = 1; col < cols; ++col) {
            if (matrix[row][col] == 0) {
                matrix[row][0] = 0;
                matrix[0][col] = 0;
            }
        }
    }

    for (int row = 1; row < rows; ++row) {
        for (int col = 1; col < cols; ++col) {
            if (matrix[row][0] == 0 || matrix[0][col] == 0) {
                matrix[row][col] = 0;
            }
        }
    }

    if (firstRowZero) {
        for (int col = 0; col < cols; ++col) {
            matrix[0][col] = 0;
        }
    }
    if (firstColZero) {
        for (int row = 0; row < rows; ++row) {
            matrix[row][0] = 0;
        }
    }
}
```

**Complexity:**
- Time: `O(rows * cols)`
- Space: `O(1)` extra space

**Edge cases:**
- Empty matrix
- Zero in first row
- Zero in first column
- All zeroes

**Common mistake:** Zeroing rows immediately during the first scan and accidentally creating new zero markers.

---

## 43. Spiral matrix traversal

**Problem:** Given a matrix, return its elements in spiral order.

**Approach:** Maintain four boundaries: top, bottom, left, and right. Traverse one edge at a time and shrink the boundary after each pass.

**C++ solution:**

```cpp
#include <vector>

std::vector<int> spiralOrder(const std::vector<std::vector<int>>& matrix) {
    std::vector<int> result;
    if (matrix.empty() || matrix[0].empty()) {
        return result;
    }

    int top = 0;
    int bottom = static_cast<int>(matrix.size()) - 1;
    int left = 0;
    int right = static_cast<int>(matrix[0].size()) - 1;

    while (top <= bottom && left <= right) {
        for (int col = left; col <= right; ++col) {
            result.push_back(matrix[top][col]);
        }
        ++top;

        for (int row = top; row <= bottom; ++row) {
            result.push_back(matrix[row][right]);
        }
        --right;

        if (top <= bottom) {
            for (int col = right; col >= left; --col) {
                result.push_back(matrix[bottom][col]);
            }
            --bottom;
        }

        if (left <= right) {
            for (int row = bottom; row >= top; --row) {
                result.push_back(matrix[row][left]);
            }
            ++left;
        }
    }

    return result;
}
```

**Complexity:**
- Time: `O(rows * cols)`
- Space: `O(1)` extra space excluding output

**Edge cases:**
- Single row
- Single column
- Non-square matrix
- Empty matrix

**Common mistake:** Forgetting the boundary checks before traversing the bottom row or left column.

---

## 44. Merge K sorted lists

**Problem:** Given `k` sorted linked lists, merge them into one sorted linked list.

**Approach:** Use a min-heap containing the current head of each non-empty list. Repeatedly extract the smallest node and push its next node.

**C++ solution:**

```cpp
#include <queue>
#include <vector>

struct ListNode {
    int value;
    ListNode* next;
};

struct CompareNode {
    bool operator()(const ListNode* a, const ListNode* b) const {
        return a->value > b->value;
    }
};

ListNode* mergeKLists(const std::vector<ListNode*>& lists) {
    std::priority_queue<ListNode*, std::vector<ListNode*>, CompareNode> heap;

    for (ListNode* node : lists) {
        if (node != nullptr) {
            heap.push(node);
        }
    }

    ListNode dummy{0, nullptr};
    ListNode* tail = &dummy;

    while (!heap.empty()) {
        ListNode* node = heap.top();
        heap.pop();

        tail->next = node;
        tail = tail->next;

        if (node->next != nullptr) {
            heap.push(node->next);
        }
    }

    return dummy.next;
}
```

**Complexity:**
- Time: `O(n log k)`
- Space: `O(k)`

**Edge cases:**
- No lists
- Empty lists
- One list
- Duplicate values

**Interview follow-up:** How would a divide-and-conquer merge compare with the heap approach?

---

## 45. Binary tree maximum path sum

**Problem:** Given a binary tree, find the maximum path sum. A path may start and end at any nodes but must follow parent-child links.

**Approach:** Use DFS. For each node, compute the best downward path that can be extended by its parent, while updating a global best path that may pass through both children.

**C++ solution:**

```cpp
#include <algorithm>
#include <limits>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

int bestDownward(TreeNode* node, int& best) {
    if (node == nullptr) {
        return 0;
    }

    int left = std::max(0, bestDownward(node->left, best));
    int right = std::max(0, bestDownward(node->right, best));

    best = std::max(best, node->value + left + right);
    return node->value + std::max(left, right);
}

int maxPathSum(TreeNode* root) {
    int best = std::numeric_limits<int>::min();
    bestDownward(root, best);
    return best;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack

**Edge cases:**
- All negative values
- Single node
- Skewed tree
- Best path does not pass through root

**Common mistake:** Returning a path that uses both left and right children to the parent; only one side can be extended upward.

---

## 46. Diameter of a binary tree

**Problem:** Given a binary tree, return the length of the longest path between any two nodes, measured in edges.

**Approach:** Use DFS to compute subtree heights. At each node, the path through that node has length `leftHeight + rightHeight`.

**C++ solution:**

```cpp
#include <algorithm>

struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;
};

int height(TreeNode* node, int& diameter) {
    if (node == nullptr) {
        return 0;
    }

    int left = height(node->left, diameter);
    int right = height(node->right, diameter);
    diameter = std::max(diameter, left + right);

    return 1 + std::max(left, right);
}

int diameterOfBinaryTree(TreeNode* root) {
    int diameter = 0;
    height(root, diameter);
    return diameter;
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(h)` recursion stack

**Edge cases:**
- Empty tree
- Single node
- Skewed tree
- Longest path entirely inside one subtree

**Common mistake:** Recomputing height for every node and turning the solution into `O(n^2)`.

---

## 47. Pacific Atlantic water flow

**Problem:** Given a grid of heights, find cells from which water can flow to both the Pacific and Atlantic oceans.

**Approach:** Reverse the flow. Start DFS/BFS from each ocean border and move to neighboring cells with height greater than or equal to the current cell. Cells reached from both oceans are answers.

**C++ solution:**

```cpp
#include <vector>

void dfs(const std::vector<std::vector<int>>& heights, std::vector<std::vector<bool>>& seen, int row, int col) {
    seen[row][col] = true;
    const int rows = static_cast<int>(heights.size());
    const int cols = static_cast<int>(heights[0].size());
    const int directions[4][2] = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

    for (const auto& direction : directions) {
        int nextRow = row + direction[0];
        int nextCol = col + direction[1];
        if (nextRow < 0 || nextRow >= rows || nextCol < 0 || nextCol >= cols) {
            continue;
        }
        if (!seen[nextRow][nextCol] && heights[nextRow][nextCol] >= heights[row][col]) {
            dfs(heights, seen, nextRow, nextCol);
        }
    }
}

std::vector<std::vector<int>> pacificAtlantic(const std::vector<std::vector<int>>& heights) {
    std::vector<std::vector<int>> result;
    if (heights.empty() || heights[0].empty()) {
        return result;
    }

    const int rows = static_cast<int>(heights.size());
    const int cols = static_cast<int>(heights[0].size());
    std::vector<std::vector<bool>> pacific(rows, std::vector<bool>(cols, false));
    std::vector<std::vector<bool>> atlantic(rows, std::vector<bool>(cols, false));

    for (int row = 0; row < rows; ++row) {
        dfs(heights, pacific, row, 0);
        dfs(heights, atlantic, row, cols - 1);
    }
    for (int col = 0; col < cols; ++col) {
        dfs(heights, pacific, 0, col);
        dfs(heights, atlantic, rows - 1, col);
    }

    for (int row = 0; row < rows; ++row) {
        for (int col = 0; col < cols; ++col) {
            if (pacific[row][col] && atlantic[row][col]) {
                result.push_back({row, col});
            }
        }
    }

    return result;
}
```

**Complexity:**
- Time: `O(rows * cols)`
- Space: `O(rows * cols)`

**Edge cases:**
- Empty grid
- Single cell
- Flat grid
- Strictly increasing or decreasing grid

**Common mistake:** Starting a search from every cell instead of reversing the problem from the ocean borders.

---

## 48. Find median from data stream

**Problem:** Design a data structure that supports adding numbers and finding the median at any time.

**Approach:** Use two heaps. A max-heap stores the smaller half, and a min-heap stores the larger half. Keep their sizes balanced.

**C++ solution:**

```cpp
#include <queue>
#include <vector>

class MedianFinder {
public:
    void addNum(int value) {
        if (lower_.empty() || value <= lower_.top()) {
            lower_.push(value);
        } else {
            upper_.push(value);
        }

        if (lower_.size() > upper_.size() + 1) {
            upper_.push(lower_.top());
            lower_.pop();
        } else if (upper_.size() > lower_.size()) {
            lower_.push(upper_.top());
            upper_.pop();
        }
    }

    double findMedian() const {
        if (lower_.size() == upper_.size()) {
            return (lower_.top() + upper_.top()) / 2.0;
        }
        return lower_.top();
    }

private:
    std::priority_queue<int> lower_;
    std::priority_queue<int, std::vector<int>, std::greater<int>> upper_;
};
```

**Complexity:**
- Add: `O(log n)`
- Find median: `O(1)`
- Space: `O(n)`

**Edge cases:**
- No values, depending on API contract
- Even count
- Odd count
- Negative values

**Common mistake:** Sorting the full stream after every insertion.

---

## 49. Decode ways

**Problem:** Given a string of digits, count how many ways it can be decoded where `1 -> A`, `2 -> B`, ..., `26 -> Z`.

**Approach:** Use dynamic programming. At each position, add ways from a valid one-digit decode and a valid two-digit decode.

**C++ solution:**

```cpp
#include <string>
#include <vector>

int numDecodings(const std::string& s) {
    if (s.empty() || s[0] == '0') {
        return 0;
    }

    std::vector<int> dp(s.size() + 1, 0);
    dp[0] = 1;
    dp[1] = 1;

    for (std::size_t i = 2; i <= s.size(); ++i) {
        if (s[i - 1] != '0') {
            dp[i] += dp[i - 1];
        }

        int twoDigit = (s[i - 2] - '0') * 10 + (s[i - 1] - '0');
        if (twoDigit >= 10 && twoDigit <= 26) {
            dp[i] += dp[i - 2];
        }
    }

    return dp[s.size()];
}
```

**Complexity:**
- Time: `O(n)`
- Space: `O(n)`, reducible to `O(1)`

**Edge cases:**
- Leading zero
- Isolated zero
- `10` and `20`
- Long strings with many valid splits

**Common mistake:** Treating `0` as independently decodable.

---

## 50. Edit distance

**Problem:** Given two strings, compute the minimum number of insertions, deletions, and replacements needed to convert one string into the other.

**Approach:** Use dynamic programming. `dp[i][j]` is the edit distance between the first `i` characters of `a` and the first `j` characters of `b`.

**C++ solution:**

```cpp
#include <algorithm>
#include <string>
#include <vector>

int minDistance(const std::string& a, const std::string& b) {
    std::vector<std::vector<int>> dp(a.size() + 1, std::vector<int>(b.size() + 1, 0));

    for (std::size_t i = 0; i <= a.size(); ++i) {
        dp[i][0] = static_cast<int>(i);
    }
    for (std::size_t j = 0; j <= b.size(); ++j) {
        dp[0][j] = static_cast<int>(j);
    }

    for (std::size_t i = 1; i <= a.size(); ++i) {
        for (std::size_t j = 1; j <= b.size(); ++j) {
            if (a[i - 1] == b[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + std::min({
                    dp[i - 1][j],
                    dp[i][j - 1],
                    dp[i - 1][j - 1]
                });
            }
        }
    }

    return dp[a.size()][b.size()];
}
```

**Complexity:**
- Time: `O(m * n)`
- Space: `O(m * n)`, reducible to `O(min(m, n))`

**Edge cases:**
- One string empty
- Both strings equal
- Completely different strings
- Repeated characters

**Common mistake:** Confusing edit distance with longest common subsequence and using the wrong recurrence.
