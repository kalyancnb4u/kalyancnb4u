---
title: "Complete C++ Mastery Part 5: Standard Template Library (STL)"
date: 2024-02-20 18:00:00 +0000
categories: [C++, Programming, STL]
tags: [cpp, stl, containers, algorithms, iterators, ranges, functional]
---

# Complete C++ Mastery Part 5: Standard Template Library (STL)

## Introduction

Welcome to Part 5 of the Complete C++ Mastery series. The Standard Template Library (STL) is C++'s crown jewel—a collection of powerful, generic, and efficient data structures and algorithms. Mastering the STL is essential for productive C++ development, as it provides battle-tested implementations of common patterns that would take hundreds of hours to write from scratch.

This part provides comprehensive coverage of the STL: containers (sequential, associative, unordered), algorithms, iterators, function objects, and modern features like ranges (C++20). We'll explore the design philosophy, performance characteristics, and best practices for each component.

**Prerequisites:** This part assumes knowledge from Parts 1-4, particularly templates and generic programming.

---

## 5.1 STL Architecture and Design

### STL Philosophy

The STL is built on three pillars:

1. **Containers**: Data structures (vector, list, map, etc.)
2. **Algorithms**: Operations on data (sort, find, transform, etc.)
3. **Iterators**: Glue between containers and algorithms

**Key Design Principles:**
- **Generic programming**: Templates enable type independence
- **Separation of concerns**: Algorithms don't depend on container internals
- **Performance**: Zero-overhead abstractions
- **Composability**: Components work together seamlessly

### Iterator Categories

Iterators abstract the concept of "position in a sequence":

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <iterator>

int main() {
    // Iterator categories (from weakest to strongest):
    
    // 1. Input Iterator: Read once, forward only
    //    Example: istream_iterator
    
    // 2. Output Iterator: Write once, forward only
    //    Example: ostream_iterator
    
    // 3. Forward Iterator: Read/write, forward only, multi-pass
    //    Example: forward_list::iterator
    
    // 4. Bidirectional Iterator: Forward + backward
    //    Example: list::iterator
    std::list<int> lst = {1, 2, 3, 4, 5};
    auto it = lst.begin();
    ++it;  // Forward
    --it;  // Backward
    
    // 5. Random Access Iterator: Bidirectional + jump to arbitrary position
    //    Example: vector::iterator, array::iterator
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto vit = vec.begin();
    vit += 3;  // Jump forward by 3
    vit -= 1;  // Jump backward by 1
    
    // 6. Contiguous Iterator (C++20): Random access + contiguous memory
    //    Example: vector::iterator, array::iterator, string::iterator
    
    return 0;
}
```

**Iterator Requirements:**

| Category | Operations | Multi-pass | Example |
|----------|-----------|------------|---------|
| Input | `++`, `*`, `==`, `!=` | No | `istream_iterator` |
| Output | `++`, `*` (write) | No | `ostream_iterator` |
| Forward | Input + multi-pass | Yes | `forward_list::iterator` |
| Bidirectional | Forward + `--` | Yes | `list::iterator` |
| Random Access | Bidirectional + `[]`, `+`, `-`, `<` | Yes | `vector::iterator` |
| Contiguous | Random + contiguous memory | Yes | `vector::iterator` |

### Generic Programming Principles

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// Generic function working with any container
template<typename Container>
void print_container(const Container& c) {
    for (const auto& elem : c) {
        std::cout << elem << " ";
    }
    std::cout << "\n";
}

// Generic function using iterators (more flexible)
template<typename Iterator>
void print_range(Iterator first, Iterator last) {
    while (first != last) {
        std::cout << *first << " ";
        ++first;
    }
    std::cout << "\n";
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    print_container(vec);
    print_range(vec.begin(), vec.end());
    
    // Print only part of container
    print_range(vec.begin() + 1, vec.end() - 1);  // 2 3 4
    
    return 0;
}
```

### Exception Safety Guarantees

STL provides three levels of exception safety:

1. **Basic guarantee**: No resource leaks, objects in valid state
2. **Strong guarantee**: Operation succeeds completely or has no effect
3. **No-throw guarantee**: Operation never throws

```cpp
#include <vector>
#include <iostream>

void exception_safety_demo() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    try {
        // push_back provides strong guarantee
        // If allocation fails, vector unchanged
        vec.push_back(6);
        
        // swap provides no-throw guarantee
        std::vector<int> other;
        vec.swap(other);  // Never throws
        
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << "\n";
    }
}
```

---

## 5.2 Sequential Containers

### std::vector - Dynamic Array

The most commonly used container. Provides random access with cache-friendly contiguous storage.

```cpp
#include <iostream>
#include <vector>

int main() {
    // Construction
    std::vector<int> v1;                    // Empty
    std::vector<int> v2(5);                 // 5 elements, value-initialized (0)
    std::vector<int> v3(5, 42);            // 5 elements, all 42
    std::vector<int> v4 = {1, 2, 3, 4, 5}; // Initializer list
    std::vector<int> v5(v4);               // Copy constructor
    std::vector<int> v6(v4.begin(), v4.end()); // Iterator range
    
    // Capacity operations
    std::cout << "Size: " << v4.size() << "\n";         // 5
    std::cout << "Capacity: " << v4.capacity() << "\n"; // >= 5
    std::cout << "Empty: " << v4.empty() << "\n";       // false
    
    v4.reserve(100);  // Pre-allocate capacity (no reallocation until 100)
    std::cout << "After reserve - Capacity: " << v4.capacity() << "\n";
    
    v4.shrink_to_fit();  // Reduce capacity to size (optional)
    
    // Element access
    std::cout << "v4[0] = " << v4[0] << "\n";       // No bounds checking
    std::cout << "v4.at(1) = " << v4.at(1) << "\n"; // Bounds checking (throws)
    std::cout << "v4.front() = " << v4.front() << "\n"; // First element
    std::cout << "v4.back() = " << v4.back() << "\n";   // Last element
    
    // Modifiers
    v4.push_back(6);           // Add to end
    v4.pop_back();             // Remove from end
    v4.insert(v4.begin() + 2, 99);  // Insert at position
    v4.erase(v4.begin() + 3);  // Erase at position
    v4.clear();                // Remove all elements
    
    // Emplace (construct in-place)
    v4.emplace_back(10);  // More efficient than push_back for complex types
    
    return 0;
}
```

**Performance Characteristics:**
```cpp
// vector<T> complexity:
// - Random access: O(1)
// - Insert/erase at end: O(1) amortized
// - Insert/erase at arbitrary position: O(n)
// - Search: O(n) unsorted, O(log n) sorted
```

**Vector Growth Strategy:**
```cpp
#include <iostream>
#include <vector>

void show_growth() {
    std::vector<int> vec;
    size_t prev_capacity = 0;
    
    for (int i = 0; i < 100; ++i) {
        vec.push_back(i);
        if (vec.capacity() != prev_capacity) {
            std::cout << "Size: " << vec.size() 
                      << ", Capacity: " << vec.capacity() << "\n";
            prev_capacity = vec.capacity();
        }
    }
}
```

**Common Pitfalls:**
```cpp
#include <vector>

void vector_pitfalls() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // ❌ Iterator invalidation after modification
    auto it = vec.begin();
    vec.push_back(6);  // May reallocate, invalidating 'it'
    // *it;  // ⚠️ Undefined behavior!
    
    // ✅ Correct: Refresh iterator
    it = vec.begin();
    vec.push_back(7);
    it = vec.begin();  // Safe
    
    // ❌ Reserve doesn't change size
    vec.reserve(100);
    // vec[50] = 10;  // ⚠️ Undefined behavior! Size is still small
    
    // ✅ Correct: Resize changes size
    vec.resize(100);
    vec[50] = 10;  // OK
}
```

### std::deque - Double-Ended Queue

Supports efficient insertion/deletion at both ends.

```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> dq = {3, 4, 5};
    
    // Insert at both ends (efficient)
    dq.push_front(2);  // O(1)
    dq.push_back(6);   // O(1)
    
    dq.pop_front();    // O(1)
    dq.pop_back();     // O(1)
    
    // Random access (like vector)
    std::cout << "dq[0] = " << dq[0] << "\n";
    
    // Print all elements
    for (int val : dq) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

**Vector vs Deque:**
```cpp
// Use vector when:
// - Need contiguous memory (cache-friendly)
// - Only insert/delete at end
// - Need best cache performance

// Use deque when:
// - Need efficient push_front()
// - Don't need contiguous memory
// - Larger capacity changes acceptable
```

### std::list - Doubly-Linked List

Efficient insertion/deletion anywhere, but no random access.

```cpp
#include <iostream>
#include <list>
#include <algorithm>

int main() {
    std::list<int> lst = {1, 2, 3, 4, 5};
    
    // Insert anywhere (O(1) if have iterator)
    auto it = lst.begin();
    ++it;  // Point to second element
    lst.insert(it, 99);  // Insert before second element
    
    // Erase anywhere (O(1))
    lst.erase(it);
    
    // List-specific operations
    lst.push_front(0);
    lst.push_back(6);
    
    // Splice: Move elements from another list (O(1)!)
    std::list<int> lst2 = {100, 200};
    lst.splice(lst.end(), lst2);  // Move all from lst2 to lst
    
    // Sort (stable sort for lists)
    lst.sort();
    
    // Remove duplicates (must be sorted first)
    lst.unique();
    
    // Remove specific values
    lst.remove(99);
    
    // Remove if predicate
    lst.remove_if([](int x) { return x > 50; });
    
    for (int val : lst) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### std::forward_list - Singly-Linked List

More memory-efficient than list, but only forward iteration.

```cpp
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> flst = {1, 2, 3, 4, 5};
    
    // Only forward iteration
    for (auto it = flst.begin(); it != flst.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";
    
    // Insert after position (not before!)
    auto it = flst.before_begin();
    flst.insert_after(it, 0);  // Insert at beginning
    
    // Erase after position
    flst.erase_after(flst.before_begin());
    
    // No size() method (by design, for efficiency)
    size_t count = std::distance(flst.begin(), flst.end());
    std::cout << "Count: " << count << "\n";
    
    return 0;
}
```

### std::array - Fixed-Size Array

Thin wrapper around C array with STL interface.

```cpp
#include <iostream>
#include <array>
#include <algorithm>

int main() {
    // Size must be compile-time constant
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    
    // STL interface
    std::cout << "Size: " << arr.size() << "\n";
    std::cout << "Front: " << arr.front() << "\n";
    std::cout << "Back: " << arr.back() << "\n";
    
    // Random access
    arr[0] = 10;
    
    // Works with STL algorithms
    std::sort(arr.begin(), arr.end());
    
    // Aggregate initialization
    std::array<int, 3> arr2 = {10, 20, 30};
    
    // No dynamic allocation (stack-based)
    constexpr size_t N = 100;
    std::array<double, N> large_arr;  // On stack
    
    return 0;
}
```

**Container Comparison:**

| Container | Random Access | Insert at End | Insert at Begin | Insert in Middle | Memory |
|-----------|--------------|---------------|-----------------|------------------|--------|
| `vector` | O(1) | O(1) amort | O(n) | O(n) | Contiguous |
| `deque` | O(1) | O(1) | O(1) | O(n) | Chunked |
| `list` | O(n) | O(1) | O(1) | O(1) | Scattered |
| `forward_list` | O(n) | N/A | O(1) | O(1) | Scattered |
| `array` | O(1) | N/A | N/A | N/A | Contiguous |

---

## 5.3 Associative Containers

### std::set and std::multiset

Ordered containers based on balanced binary tree (typically Red-Black tree).

```cpp
#include <iostream>
#include <set>

int main() {
    // set: unique elements, ordered
    std::set<int> s = {5, 2, 8, 1, 9, 2};  // 2 appears once
    
    // Automatically sorted
    for (int val : s) {
        std::cout << val << " ";  // 1 2 5 8 9
    }
    std::cout << "\n";
    
    // Insert
    s.insert(3);
    s.insert(3);  // Duplicate ignored
    
    // Find
    auto it = s.find(5);
    if (it != s.end()) {
        std::cout << "Found: " << *it << "\n";
    }
    
    // Erase
    s.erase(2);
    
    // Count (0 or 1 for set)
    std::cout << "Count of 8: " << s.count(8) << "\n";
    
    // Lower/upper bound
    auto lb = s.lower_bound(5);  // First element >= 5
    auto ub = s.upper_bound(5);  // First element > 5
    
    // multiset: allows duplicates
    std::multiset<int> ms = {1, 2, 2, 3, 3, 3};
    std::cout << "Count of 3 in multiset: " << ms.count(3) << "\n";  // 3
    
    return 0;
}
```

**Custom Comparator:**
```cpp
#include <iostream>
#include <set>
#include <string>

struct Person {
    std::string name;
    int age;
};

// Custom comparator
struct CompareByAge {
    bool operator()(const Person& a, const Person& b) const {
        return a.age < b.age;
    }
};

int main() {
    std::set<Person, CompareByAge> people;
    
    people.insert({"Alice", 30});
    people.insert({"Bob", 25});
    people.insert({"Charlie", 35});
    
    for (const auto& p : people) {
        std::cout << p.name << " (" << p.age << ")\n";
    }
    // Sorted by age: Bob (25), Alice (30), Charlie (35)
    
    return 0;
}
```

### std::map and std::multimap

Associative arrays with ordered keys.

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    // map: unique keys
    std::map<std::string, int> ages;
    
    // Insert
    ages["Alice"] = 30;
    ages["Bob"] = 25;
    ages.insert({"Charlie", 35});
    ages.insert(std::make_pair("David", 28));
    
    // Access (creates if doesn't exist!)
    std::cout << "Alice's age: " << ages["Alice"] << "\n";
    
    // Safe access (doesn't create)
    auto it = ages.find("Eve");
    if (it != ages.end()) {
        std::cout << "Eve's age: " << it->second << "\n";
    } else {
        std::cout << "Eve not found\n";
    }
    
    // Iterate (sorted by key)
    for (const auto& [name, age] : ages) {  // C++17 structured binding
        std::cout << name << ": " << age << "\n";
    }
    
    // Erase
    ages.erase("Bob");
    
    // Check existence
    if (ages.count("Alice")) {
        std::cout << "Alice exists\n";
    }
    
    // multimap: allows duplicate keys
    std::multimap<std::string, std::string> phone_book;
    phone_book.insert({"Alice", "123-4567"});
    phone_book.insert({"Alice", "987-6543"});  // Same key!
    
    auto range = phone_book.equal_range("Alice");
    std::cout << "Alice's numbers:\n";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << "  " << it->second << "\n";
    }
    
    return 0;
}
```

**Map Performance:**
```cpp
// map<K, V> complexity (balanced tree):
// - Insert: O(log n)
// - Find: O(log n)
// - Erase: O(log n)
// - Iteration: O(n) in sorted order
```

---

## 5.4 Unordered Containers (Hash Tables)

### std::unordered_set and std::unordered_multiset

Hash-based containers with O(1) average-case operations.

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> us = {5, 2, 8, 1, 9};
    
    // Insert: O(1) average
    us.insert(3);
    
    // Find: O(1) average
    if (us.find(5) != us.end()) {
        std::cout << "Found 5\n";
    }
    
    // Iteration (unordered!)
    for (int val : us) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    // Hash table information
    std::cout << "Bucket count: " << us.bucket_count() << "\n";
    std::cout << "Load factor: " << us.load_factor() << "\n";
    std::cout << "Max load factor: " << us.max_load_factor() << "\n";
    
    return 0;
}
```

### std::unordered_map and std::unordered_multimap

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    std::unordered_map<std::string, int> word_count;
    
    // Count words
    std::string text[] = {"hello", "world", "hello", "cpp"};
    for (const auto& word : text) {
        word_count[word]++;  // Creates if doesn't exist
    }
    
    // Print counts
    for (const auto& [word, count] : word_count) {
        std::cout << word << ": " << count << "\n";
    }
    
    return 0;
}
```

**Custom Hash Function:**
```cpp
#include <iostream>
#include <unordered_set>
#include <string>

struct Person {
    std::string name;
    int age;
    
    bool operator==(const Person& other) const {
        return name == other.name && age == other.age;
    }
};

// Custom hash
struct PersonHash {
    size_t operator()(const Person& p) const {
        return std::hash<std::string>{}(p.name) ^ 
               std::hash<int>{}(p.age);
    }
};

int main() {
    std::unordered_set<Person, PersonHash> people;
    
    people.insert({"Alice", 30});
    people.insert({"Bob", 25});
    
    std::cout << "People count: " << people.size() << "\n";
    
    return 0;
}
```

**Ordered vs Unordered:**

| Feature | `set`/`map` | `unordered_set`/`unordered_map` |
|---------|-------------|----------------------------------|
| Ordering | Sorted | None |
| Implementation | Balanced tree | Hash table |
| Insert | O(log n) | O(1) average |
| Find | O(log n) | O(1) average |
| Iterate | Sorted order | Arbitrary order |
| Memory | Less | More (hash table) |
| Use when | Need order | Need speed |

---

## 5.5 Container Adaptors

### std::stack

LIFO (Last In, First Out) adapter.

```cpp
#include <iostream>
#include <stack>
#include <vector>
#include <list>

int main() {
    // Default: uses deque
    std::stack<int> s1;
    
    // Use vector as underlying container
    std::stack<int, std::vector<int>> s2;
    
    // Use list as underlying container
    std::stack<int, std::list<int>> s3;
    
    // Operations
    s1.push(10);
    s1.push(20);
    s1.push(30);
    
    std::cout << "Top: " << s1.top() << "\n";  // 30
    std::cout << "Size: " << s1.size() << "\n";
    
    while (!s1.empty()) {
        std::cout << s1.top() << " ";
        s1.pop();
    }
    std::cout << "\n";  // 30 20 10
    
    return 0;
}
```

### std::queue

FIFO (First In, First Out) adapter.

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;
    
    // Enqueue
    q.push(10);
    q.push(20);
    q.push(30);
    
    // Access front and back
    std::cout << "Front: " << q.front() << "\n";  // 10
    std::cout << "Back: " << q.back() << "\n";    // 30
    
    // Dequeue
    while (!q.empty()) {
        std::cout << q.front() << " ";
        q.pop();
    }
    std::cout << "\n";  // 10 20 30
    
    return 0;
}
```

### std::priority_queue

Heap-based priority queue (max-heap by default).

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <functional>

int main() {
    // Max-heap (default)
    std::priority_queue<int> max_heap;
    max_heap.push(30);
    max_heap.push(10);
    max_heap.push(20);
    
    std::cout << "Max-heap: ";
    while (!max_heap.empty()) {
        std::cout << max_heap.top() << " ";
        max_heap.pop();
    }
    std::cout << "\n";  // 30 20 10
    
    // Min-heap (using greater comparator)
    std::priority_queue<int, std::vector<int>, std::greater<int>> min_heap;
    min_heap.push(30);
    min_heap.push(10);
    min_heap.push(20);
    
    std::cout << "Min-heap: ";
    while (!min_heap.empty()) {
        std::cout << min_heap.top() << " ";
        min_heap.pop();
    }
    std::cout << "\n";  // 10 20 30
    
    return 0;
}
```

---

## 5.6 String Handling

### std::string

```cpp
#include <iostream>
#include <string>

int main() {
    // Construction
    std::string s1 = "Hello";
    std::string s2("World");
    std::string s3(5, 'A');  // "AAAAA"
    
    // Concatenation
    std::string s4 = s1 + " " + s2;
    s4 += "!";
    
    // Access
    std::cout << "s4[0] = " << s4[0] << "\n";
    std::cout << "s4.at(1) = " << s4.at(1) << "\n";
    
    // Substring
    std::string sub = s4.substr(0, 5);  // "Hello"
    std::cout << "Substring: " << sub << "\n";
    
    // Find
    size_t pos = s4.find("World");
    if (pos != std::string::npos) {
        std::cout << "Found 'World' at position " << pos << "\n";
    }
    
    // Replace
    s4.replace(0, 5, "Hi");  // Replace "Hello" with "Hi"
    std::cout << "After replace: " << s4 << "\n";
    
    // Comparison
    if (s1 == "Hello") {
        std::cout << "s1 is Hello\n";
    }
    
    // Conversion
    int num = std::stoi("42");
    double dbl = std::stod("3.14");
    std::string num_str = std::to_string(123);
    
    return 0;
}
```

### std::string_view (C++17)

Non-owning view of string data (avoids copies).

```cpp
#include <iostream>
#include <string>
#include <string_view>

void process(std::string_view sv) {  // No copy!
    std::cout << "Processing: " << sv << "\n";
}

int main() {
    std::string s = "Hello World";
    
    // Create view (no copy)
    std::string_view sv = s;
    
    // Substring view (no allocation)
    std::string_view sub = sv.substr(0, 5);
    
    process(s);           // Implicit conversion
    process("literal");   // Works with string literals
    
    // ⚠️ Danger: Don't let view outlive string
    std::string_view dangling;
    {
        std::string temp = "temporary";
        dangling = temp;  // ⚠️ Dangerous!
    }  // temp destroyed
    // std::cout << dangling;  // ⚠️ Undefined behavior!
    
    return 0;
}
```

[Due to length constraints, this is approximately 50% of Part 5. Would you like me to continue with sections 5.7-5.10 covering Iterators, Algorithms, Function Objects/Lambdas, and Utilities?]

---

## 5.7 Iterators

### Iterator Basics

```cpp
#include <iostream>
#include <vector>
#include <list>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Iterator traversal
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";
    
    // Const iterator
    for (auto it = vec.cbegin(); it != vec.cend(); ++it) {
        // *it = 10;  // ❌ Error: const iterator
        std::cout << *it << " ";
    }
    std::cout << "\n";
    
    // Reverse iterator
    for (auto it = vec.rbegin(); it != vec.rend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";  // 5 4 3 2 1
    
    return 0;
}
```

### Iterator Adapters

**Back Insert Iterator:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> source = {1, 2, 3, 4, 5};
    std::vector<int> dest;
    
    // Copy using back_inserter (automatically calls push_back)
    std::copy(source.begin(), source.end(), 
              std::back_inserter(dest));
    
    for (int val : dest) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

**Front Insert Iterator:**
```cpp
#include <list>
#include <algorithm>
#include <iterator>
#include <iostream>

int main() {
    std::list<int> source = {1, 2, 3};
    std::list<int> dest;
    
    // Insert at front (reverses order)
    std::copy(source.begin(), source.end(),
              std::front_inserter(dest));
    
    for (int val : dest) {
        std::cout << val << " ";
    }
    std::cout << "\n";  // 3 2 1
    
    return 0;
}
```

**Stream Iterators:**
```cpp
#include <iostream>
#include <iterator>
#include <vector>
#include <algorithm>
#include <sstream>

int main() {
    // istream_iterator: read from stream
    std::istringstream iss("1 2 3 4 5");
    std::vector<int> vec(
        std::istream_iterator<int>(iss),
        std::istream_iterator<int>()  // End-of-stream iterator
    );
    
    std::cout << "Read from stream: ";
    for (int val : vec) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    // ostream_iterator: write to stream
    std::cout << "Using ostream_iterator: ";
    std::copy(vec.begin(), vec.end(),
              std::ostream_iterator<int>(std::cout, " "));
    std::cout << "\n";
    
    return 0;
}
```

### Iterator Traits

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <iterator>

template<typename Iterator>
void print_iterator_category() {
    using category = typename std::iterator_traits<Iterator>::iterator_category;
    
    if (std::is_same_v<category, std::random_access_iterator_tag>) {
        std::cout << "Random access iterator\n";
    } else if (std::is_same_v<category, std::bidirectional_iterator_tag>) {
        std::cout << "Bidirectional iterator\n";
    } else if (std::is_same_v<category, std::forward_iterator_tag>) {
        std::cout << "Forward iterator\n";
    }
}

int main() {
    std::vector<int> vec;
    std::list<int> lst;
    
    print_iterator_category<decltype(vec.begin())>();  // Random access
    print_iterator_category<decltype(lst.begin())>();  // Bidirectional
    
    return 0;
}
```

---

## 5.8 Algorithms

### Non-Modifying Algorithms

**for_each:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Apply function to each element
    std::for_each(vec.begin(), vec.end(), [](int x) {
        std::cout << x * 2 << " ";
    });
    std::cout << "\n";
    
    return 0;
}
```

**find, find_if:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Find specific value
    auto it1 = std::find(vec.begin(), vec.end(), 3);
    if (it1 != vec.end()) {
        std::cout << "Found: " << *it1 << "\n";
    }
    
    // Find with predicate
    auto it2 = std::find_if(vec.begin(), vec.end(), [](int x) {
        return x > 3;
    });
    if (it2 != vec.end()) {
        std::cout << "First element > 3: " << *it2 << "\n";
    }
    
    return 0;
}
```

**count, count_if:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 2, 4, 2, 5};
    
    int count_2 = std::count(vec.begin(), vec.end(), 2);
    std::cout << "Count of 2: " << count_2 << "\n";
    
    int count_even = std::count_if(vec.begin(), vec.end(), [](int x) {
        return x % 2 == 0;
    });
    std::cout << "Count of even: " << count_even << "\n";
    
    return 0;
}
```

**all_of, any_of, none_of:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {2, 4, 6, 8};
    
    bool all_even = std::all_of(vec.begin(), vec.end(), [](int x) {
        return x % 2 == 0;
    });
    std::cout << "All even: " << all_even << "\n";
    
    bool any_odd = std::any_of(vec.begin(), vec.end(), [](int x) {
        return x % 2 == 1;
    });
    std::cout << "Any odd: " << any_odd << "\n";
    
    bool none_negative = std::none_of(vec.begin(), vec.end(), [](int x) {
        return x < 0;
    });
    std::cout << "None negative: " << none_negative << "\n";
    
    return 0;
}
```

### Modifying Algorithms

**copy, copy_if:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> source = {1, 2, 3, 4, 5, 6};
    std::vector<int> dest1, dest2;
    
    // Copy all
    std::copy(source.begin(), source.end(), 
              std::back_inserter(dest1));
    
    // Copy with condition
    std::copy_if(source.begin(), source.end(),
                 std::back_inserter(dest2),
                 [](int x) { return x % 2 == 0; });
    
    std::cout << "dest1: ";
    for (int x : dest1) std::cout << x << " ";
    std::cout << "\n";
    
    std::cout << "dest2 (even): ";
    for (int x : dest2) std::cout << x << " ";
    std::cout << "\n";
    
    return 0;
}
```

**transform:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::vector<int> result;
    
    // Transform each element
    std::transform(vec.begin(), vec.end(),
                   std::back_inserter(result),
                   [](int x) { return x * x; });
    
    std::cout << "Squares: ";
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";
    
    // Transform two ranges
    std::vector<int> vec2 = {10, 20, 30, 40, 50};
    std::vector<int> sum;
    
    std::transform(vec.begin(), vec.end(),
                   vec2.begin(),
                   std::back_inserter(sum),
                   [](int a, int b) { return a + b; });
    
    std::cout << "Sums: ";
    for (int x : sum) std::cout << x << " ";
    std::cout << "\n";
    
    return 0;
}
```

**fill, generate:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec1(5);
    std::vector<int> vec2(5);
    
    // Fill with value
    std::fill(vec1.begin(), vec1.end(), 42);
    
    // Generate with function
    int counter = 0;
    std::generate(vec2.begin(), vec2.end(), [&counter]() {
        return counter++;
    });
    
    std::cout << "vec1: ";
    for (int x : vec1) std::cout << x << " ";
    std::cout << "\n";
    
    std::cout << "vec2: ";
    for (int x : vec2) std::cout << x << " ";
    std::cout << "\n";
    
    return 0;
}
```

**remove, remove_if:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 2, 4, 2, 5};
    
    // remove moves elements to end, returns new logical end
    auto new_end = std::remove(vec.begin(), vec.end(), 2);
    
    std::cout << "After remove (before erase): ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    // Actually erase the removed elements
    vec.erase(new_end, vec.end());
    
    std::cout << "After erase: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    // remove_if with predicate
    vec = {1, 2, 3, 4, 5, 6, 7, 8};
    vec.erase(
        std::remove_if(vec.begin(), vec.end(), [](int x) {
            return x % 2 == 0;  // Remove even numbers
        }),
        vec.end()
    );
    
    std::cout << "Odd only: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    return 0;
}
```

**unique:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 1, 2, 2, 2, 3, 4, 4, 5};
    
    // unique removes consecutive duplicates
    auto new_end = std::unique(vec.begin(), vec.end());
    vec.erase(new_end, vec.end());
    
    std::cout << "After unique: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";  // 1 2 3 4 5
    
    return 0;
}
```

### Sorting and Related

**sort:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9, 3};
    
    // Sort ascending
    std::sort(vec.begin(), vec.end());
    
    std::cout << "Ascending: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    // Sort descending
    std::sort(vec.begin(), vec.end(), std::greater<int>());
    
    std::cout << "Descending: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    // Custom comparator
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return std::abs(a - 5) < std::abs(b - 5);  // Closest to 5 first
    });
    
    std::cout << "By distance from 5: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    return 0;
}
```

**partial_sort:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    
    // Sort first 3 elements
    std::partial_sort(vec.begin(), vec.begin() + 3, vec.end());
    
    std::cout << "First 3 sorted: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";  // 1 2 3 (rest unspecified order)
    
    return 0;
}
```

**nth_element:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    
    // Partition around 5th element (median)
    std::nth_element(vec.begin(), vec.begin() + 4, vec.end());
    
    std::cout << "5th element (median): " << vec[4] << "\n";
    std::cout << "After nth_element: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    // Elements before vec[4] are smaller, after are larger
    
    return 0;
}
```

**binary_search, lower_bound, upper_bound:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    // Binary search (requires sorted range)
    bool found = std::binary_search(vec.begin(), vec.end(), 5);
    std::cout << "5 found: " << found << "\n";
    
    // Lower bound: first element >= value
    auto lb = std::lower_bound(vec.begin(), vec.end(), 5);
    std::cout << "Lower bound of 5: " << *lb << "\n";
    
    // Upper bound: first element > value
    auto ub = std::upper_bound(vec.begin(), vec.end(), 5);
    std::cout << "Upper bound of 5: " << *ub << "\n";
    
    // Equal range: [lower_bound, upper_bound)
    auto er = std::equal_range(vec.begin(), vec.end(), 5);
    std::cout << "Equal range: [" << *er.first << ", " << *er.second << ")\n";
    
    return 0;
}
```

### Set Operations

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4, 5};
    std::vector<int> v2 = {3, 4, 5, 6, 7};
    std::vector<int> result;
    
    // Union
    std::set_union(v1.begin(), v1.end(),
                   v2.begin(), v2.end(),
                   std::back_inserter(result));
    std::cout << "Union: ";
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";  // 1 2 3 4 5 6 7
    
    // Intersection
    result.clear();
    std::set_intersection(v1.begin(), v1.end(),
                          v2.begin(), v2.end(),
                          std::back_inserter(result));
    std::cout << "Intersection: ";
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";  // 3 4 5
    
    // Difference
    result.clear();
    std::set_difference(v1.begin(), v1.end(),
                        v2.begin(), v2.end(),
                        std::back_inserter(result));
    std::cout << "Difference: ";
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";  // 1 2
    
    return 0;
}
```

### Heap Operations

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 9};
    
    // Make heap (max-heap by default)
    std::make_heap(vec.begin(), vec.end());
    
    std::cout << "Max element: " << vec.front() << "\n";
    
    // Push to heap
    vec.push_back(10);
    std::push_heap(vec.begin(), vec.end());
    
    std::cout << "After push, max: " << vec.front() << "\n";
    
    // Pop from heap
    std::pop_heap(vec.begin(), vec.end());  // Moves max to end
    vec.pop_back();  // Remove it
    
    std::cout << "After pop, max: " << vec.front() << "\n";
    
    // Sort heap (destroys heap property)
    std::sort_heap(vec.begin(), vec.end());
    std::cout << "Sorted: ";
    for (int x : vec) std::cout << x << " ";
    std::cout << "\n";
    
    return 0;
}
```

### Numeric Algorithms

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Accumulate (sum)
    int sum = std::accumulate(vec.begin(), vec.end(), 0);
    std::cout << "Sum: " << sum << "\n";
    
    // Product
    int product = std::accumulate(vec.begin(), vec.end(), 1,
                                   std::multiplies<int>());
    std::cout << "Product: " << product << "\n";
    
    // Inner product
    std::vector<int> vec2 = {10, 20, 30, 40, 50};
    int dot = std::inner_product(vec.begin(), vec.end(),
                                  vec2.begin(), 0);
    std::cout << "Dot product: " << dot << "\n";
    
    // Partial sum
    std::vector<int> partial_sums;
    std::partial_sum(vec.begin(), vec.end(),
                     std::back_inserter(partial_sums));
    std::cout << "Partial sums: ";
    for (int x : partial_sums) std::cout << x << " ";
    std::cout << "\n";  // 1 3 6 10 15
    
    // Adjacent difference
    std::vector<int> diffs;
    std::adjacent_difference(vec.begin(), vec.end(),
                            std::back_inserter(diffs));
    std::cout << "Adjacent differences: ";
    for (int x : diffs) std::cout << x << " ";
    std::cout << "\n";  // 1 1 1 1 1
    
    // Iota (fill with sequence)
    std::vector<int> sequence(10);
    std::iota(sequence.begin(), sequence.end(), 1);
    std::cout << "Iota: ";
    for (int x : sequence) std::cout << x << " ";
    std::cout << "\n";  // 1 2 3 4 5 6 7 8 9 10
    
    return 0;
}
```

---

## 5.9 Function Objects and Lambdas

### Function Objects (Functors)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// Function object with state
class Multiplier {
    int factor;
public:
    Multiplier(int f) : factor(f) {}
    
    int operator()(int x) const {
        return x * factor;
    }
};

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    Multiplier times_three(3);
    
    std::transform(vec.begin(), vec.end(), vec.begin(), times_three);
    
    for (int x : vec) {
        std::cout << x << " ";  // 3 6 9 12 15
    }
    std::cout << "\n";
    
    return 0;
}
```

### Standard Function Objects

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9};
    
    // Arithmetic function objects
    std::transform(vec.begin(), vec.end(), vec.begin(),
                   std::negate<int>());  // Negate all
    
    // Comparison function objects
    std::sort(vec.begin(), vec.end(), std::greater<int>());  // Descending
    
    // Logical function objects
    std::vector<bool> bools = {true, false, true};
    bool all = std::all_of(bools.begin(), bools.end(),
                           std::logical_not<bool>());
    
    return 0;
}
```

### Lambda Expressions

**Basic Lambda:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Simple lambda
    std::for_each(vec.begin(), vec.end(), [](int x) {
        std::cout << x * 2 << " ";
    });
    std::cout << "\n";
    
    // Lambda with return type
    auto square = [](int x) -> int {
        return x * x;
    };
    
    std::cout << "square(5) = " << square(5) << "\n";
    
    return 0;
}
```

**Lambda Captures:**
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int factor = 3;
    int count = 0;
    
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // Capture by value [=]
    std::for_each(vec.begin(), vec.end(), [factor](int x) {
        std::cout << x * factor << " ";
    });
    std::cout << "\n";
    
    // Capture by reference [&]
    std::for_each(vec.begin(), vec.end(), [&count](int x) {
        count += x;
    });
    std::cout << "Sum: " << count << "\n";
    
    // Mixed capture
    int multiplier = 2;
    std::for_each(vec.begin(), vec.end(), [multiplier, &count](int x) {
        count += x * multiplier;
    });
    
    // Capture specific variables
    int a = 1, b = 2;
    auto lambda = [a, &b]() {
        // a captured by value, b by reference
    };
    
    // Capture all by value, specific by reference
    auto lambda2 = [=, &count]() {
        // All by value except count
    };
    
    // Capture all by reference, specific by value
    auto lambda3 = [&, factor]() {
        // All by reference except factor
    };
    
    return 0;
}
```

**Mutable Lambda:**
```cpp
#include <iostream>

int main() {
    int x = 0;
    
    // Non-mutable lambda (default)
    auto lambda1 = [x]() {
        // x = 10;  // ❌ Error: cannot modify captured value
        return x;
    };
    
    // Mutable lambda
    auto lambda2 = [x]() mutable {
        x = 10;  // ✅ OK: can modify copy
        return x;
    };
    
    std::cout << "Original x: " << x << "\n";  // 0
    std::cout << "lambda2(): " << lambda2() << "\n";  // 10
    std::cout << "x after lambda: " << x << "\n";  // 0 (original unchanged)
    
    return 0;
}
```

**Generic Lambda (C++14):**
```cpp
#include <iostream>
#include <vector>

int main() {
    // Generic lambda (auto parameters)
    auto print = [](const auto& x) {
        std::cout << x << "\n";
    };
    
    print(42);
    print(3.14);
    print("hello");
    
    // Generic lambda with multiple parameters
    auto add = [](auto a, auto b) {
        return a + b;
    };
    
    std::cout << add(1, 2) << "\n";      // 3
    std::cout << add(1.5, 2.5) << "\n";  // 4.0
    
    return 0;
}
```

### std::function

Type-erased wrapper for callable objects:

```cpp
#include <iostream>
#include <functional>
#include <vector>

int add(int a, int b) { return a + b; }

class Multiplier {
public:
    int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    // Store different callable types
    std::vector<std::function<int(int, int)>> operations;
    
    // Function pointer
    operations.push_back(add);
    
    // Functor
    operations.push_back(Multiplier());
    
    // Lambda
    operations.push_back([](int a, int b) { return a - b; });
    
    // Use them uniformly
    for (const auto& op : operations) {
        std::cout << op(10, 5) << "\n";
    }
    // Output: 15, 50, 5
    
    return 0;
}
```

**std::bind:**
```cpp
#include <iostream>
#include <functional>

int divide(int a, int b) {
    return a / b;
}

int main() {
    // Bind second argument
    auto divide_by_2 = std::bind(divide, std::placeholders::_1, 2);
    std::cout << divide_by_2(10) << "\n";  // 5
    
    // Bind first argument
    auto half_of = std::bind(divide, std::placeholders::_1, 2);
    std::cout << half_of(20) << "\n";  // 10
    
    // Reorder arguments
    auto reversed = std::bind(divide, std::placeholders::_2, std::placeholders::_1);
    std::cout << reversed(2, 10) << "\n";  // 5 (10/2)
    
    // Modern alternative: Use lambda (preferred)
    auto divide_by_2_lambda = [](int x) { return divide(x, 2); };
    
    return 0;
}
```

---

## 5.10 Utility Components

### std::pair

```cpp
#include <iostream>
#include <utility>
#include <string>

int main() {
    // Construction
    std::pair<int, std::string> p1(1, "one");
    std::pair<int, std::string> p2 = {2, "two"};
    auto p3 = std::make_pair(3, "three");
    
    // Access
    std::cout << "p1: " << p1.first << ", " << p1.second << "\n";
    
    // Comparison
    if (p1 < p2) {
        std::cout << "p1 < p2\n";
    }
    
    // Structured binding (C++17)
    auto [num, str] = p1;
    std::cout << "num: " << num << ", str: " << str << "\n";
    
    return 0;
}
```

### std::tuple (C++11)

```cpp
#include <iostream>
#include <tuple>
#include <string>

int main() {
    // Construction
    std::tuple<int, double, std::string> t1(1, 3.14, "hello");
    auto t2 = std::make_tuple(2, 2.71, "world");
    
    // Access by index
    std::cout << std::get<0>(t1) << "\n";  // 1
    std::cout << std::get<1>(t1) << "\n";  // 3.14
    std::cout << std::get<2>(t1) << "\n";  // hello
    
    // Access by type (C++14, if unique)
    std::cout << std::get<int>(t1) << "\n";
    std::cout << std::get<std::string>(t1) << "\n";
    
    // Structured binding (C++17)
    auto [i, d, s] = t1;
    std::cout << "i: " << i << ", d: " << d << ", s: " << s << "\n";
    
    // Tuple size
    constexpr size_t size = std::tuple_size<decltype(t1)>::value;
    std::cout << "Tuple size: " << size << "\n";  // 3
    
    return 0;
}
```

### std::optional (C++17)

Represents optional values (may or may not contain a value):

```cpp
#include <iostream>
#include <optional>
#include <string>

std::optional<int> parse_int(const std::string& s) {
    try {
        return std::stoi(s);
    } catch (...) {
        return std::nullopt;  // No value
    }
}

int main() {
    // Create optional
    std::optional<int> opt1 = 42;
    std::optional<int> opt2;  // Empty
    std::optional<int> opt3 = std::nullopt;  // Explicitly empty
    
    // Check if has value
    if (opt1.has_value()) {
        std::cout << "opt1: " << opt1.value() << "\n";
    }
    
    // Alternative: operator bool
    if (opt1) {
        std::cout << "opt1: " << *opt1 << "\n";
    }
    
    // Value or default
    std::cout << "opt2 or default: " << opt2.value_or(0) << "\n";
    
    // Using with function
    auto result1 = parse_int("123");
    auto result2 = parse_int("abc");
    
    if (result1) {
        std::cout << "Parsed: " << *result1 << "\n";
    }
    
    if (!result2) {
        std::cout << "Parse failed\n";
    }
    
    return 0;
}
```

### std::variant (C++17)

Type-safe union:

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    // Can hold int, double, or string
    std::variant<int, double, std::string> v;
    
    v = 42;
    std::cout << "int: " << std::get<int>(v) << "\n";
    
    v = 3.14;
    std::cout << "double: " << std::get<double>(v) << "\n";
    
    v = "hello";
    std::cout << "string: " << std::get<std::string>(v) << "\n";
    
    // Check which type is active
    std::cout << "Index: " << v.index() << "\n";  // 2 (string)
    
    // holds_alternative
    if (std::holds_alternative<std::string>(v)) {
        std::cout << "Currently holds string\n";
    }
    
    // Visit pattern
    std::visit([](const auto& val) {
        std::cout << "Value: " << val << "\n";
    }, v);
    
    return 0;
}
```

### std::any (C++17)

Can hold any copyable type:

```cpp
#include <iostream>
#include <any>
#include <string>

int main() {
    std::any a = 42;
    
    // Type checking
    if (a.has_value()) {
        std::cout << "Has value\n";
    }
    
    // Safe cast
    try {
        int i = std::any_cast<int>(a);
        std::cout << "int: " << i << "\n";
    } catch (const std::bad_any_cast& e) {
        std::cout << "Bad cast\n";
    }
    
    // Change type
    a = std::string("hello");
    std::cout << "string: " << std::any_cast<std::string>(a) << "\n";
    
    // Pointer cast (doesn't throw)
    if (auto* p = std::any_cast<std::string>(&a)) {
        std::cout << "String pointer: " << *p << "\n";
    }
    
    return 0;
}
```

---

## Comprehensive Example: Data Processing Pipeline

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <string>
#include <map>

struct Person {
    std::string name;
    int age;
    double salary;
};

int main() {
    std::vector<Person> people = {
        {"Alice", 30, 50000},
        {"Bob", 25, 45000},
        {"Charlie", 35, 60000},
        {"David", 28, 48000},
        {"Eve", 32, 55000}
    };
    
    std::cout << "=== Original Data ===\n";
    std::for_each(people.begin(), people.end(), [](const Person& p) {
        std::cout << p.name << " (age " << p.age << "): $" << p.salary << "\n";
    });
    
    // Filter: people over 28
    std::vector<Person> filtered;
    std::copy_if(people.begin(), people.end(),
                 std::back_inserter(filtered),
                 [](const Person& p) { return p.age > 28; });
    
    std::cout << "\n=== Over 28 ===\n";
    for (const auto& p : filtered) {
        std::cout << p.name << " (" << p.age << ")\n";
    }
    
    // Sort by salary
    std::sort(filtered.begin(), filtered.end(),
              [](const Person& a, const Person& b) {
                  return a.salary > b.salary;
              });
    
    std::cout << "\n=== Sorted by Salary (desc) ===\n";
    for (const auto& p : filtered) {
        std::cout << p.name << ": $" << p.salary << "\n";
    }
    
    // Calculate average salary
    double total_salary = std::accumulate(
        filtered.begin(), filtered.end(), 0.0,
        [](double sum, const Person& p) {
            return sum + p.salary;
        }
    );
    double avg_salary = total_salary / filtered.size();
    std::cout << "\nAverage salary: $" << avg_salary << "\n";
    
    // Group by age bracket
    std::map<int, std::vector<Person>> age_groups;
    for (const auto& p : people) {
        int bracket = (p.age / 10) * 10;  // 20s, 30s, etc.
        age_groups[bracket].push_back(p);
    }
    
    std::cout << "\n=== Age Groups ===\n";
    for (const auto& [bracket, group] : age_groups) {
        std::cout << bracket << "s: ";
        std::for_each(group.begin(), group.end(), [](const Person& p) {
            std::cout << p.name << " ";
        });
        std::cout << "\n";
    }
    
    // Transform: Give 10% raise
    std::vector<Person> raised = people;
    std::transform(raised.begin(), raised.end(), raised.begin(),
                   [](Person p) {
                       p.salary *= 1.1;
                       return p;
                   });
    
    std::cout << "\n=== After 10% Raise ===\n";
    for (const auto& p : raised) {
        std::cout << p.name << ": $" << p.salary << "\n";
    }
    
    return 0;
}
```

---

## Conclusion

Part 5 has provided comprehensive coverage of the Standard Template Library:

1. **STL Architecture** - Containers, algorithms, iterators design
2. **Sequential Containers** - vector, deque, list, forward_list, array
3. **Associative Containers** - set, multiset, map, multimap
4. **Unordered Containers** - Hash-based unordered_set, unordered_map
5. **Container Adaptors** - stack, queue, priority_queue
6. **String Handling** - string, string_view
7. **Iterators** - Categories, adapters, traits
8. **Algorithms** - Non-modifying, modifying, sorting, numeric
9. **Function Objects** - Functors, lambdas, std::function
10. **Utilities** - pair, tuple, optional, variant, any

**Key Principles:**
- **Generic Programming**: Works with any type
- **Composability**: Algorithms work with any container via iterators
- **Performance**: Zero-overhead abstractions
- **Type Safety**: Compile-time checking

**Best Practices:**
- Use appropriate container for your access pattern
- Prefer algorithms over hand-written loops
- Use lambdas for simple predicates
- Understand complexity guarantees
- Use structured bindings (C++17)
- Leverage ranges (C++20) when available

**Next in the series:** Part 6 will explore Concurrency & Multithreading, covering threads, synchronization, atomic operations, and parallel algorithms.

---

## References

- **cppreference.com**: Comprehensive STL reference
- **"Effective STL"** by Scott Meyers
- **"The C++ Standard Library"** by Nicolai M. Josuttis
- **C++ Core Guidelines**: STL usage guidelines

---

*This blog post is part of the Complete C++ Mastery series. Continue to Part 6 for Concurrency & Multithreading.*
