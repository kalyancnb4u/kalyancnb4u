---
title: "Complete C++ Mastery Part 16: Common Pitfalls & Best Practices"
date: 2024-02-21 16:00:00 +0000
categories: [C++, Programming, Best-Practices]
tags: [cpp, pitfalls, mistakes, debugging, best-practices, code-quality]
---

# Complete C++ Mastery Part 16: Common Pitfalls & Best Practices

## Introduction

Welcome to Part 16 of the Complete C++ Mastery series. Even experienced C++ developers can fall into common traps. This part catalogs frequent mistakes, their consequences, and how to avoid them. Learning from others' mistakes is faster than making them yourself.

We'll cover memory issues, undefined behavior, performance antipatterns, compilation errors, and best practices that prevent problems before they occur.

**Prerequisites:** This part synthesizes knowledge from all previous parts.

---

## 16.1 Memory Management Pitfalls

### Memory Leaks

```cpp
// ❌ PITFALL: Forgetting to delete
void memory_leak() {
    int* ptr = new int(42);
    // ... code that might throw
    // delete ptr;  // ⚠️ Never executed if exception thrown
}

// ❌ PITFALL: Losing pointer
void lost_pointer() {
    int* ptr = new int(42);
    ptr = new int(100);  // ⚠️ Lost original allocation!
}

// ❌ PITFALL: Leaking in constructor
class BadWidget {
    int* data1;
    int* data2;
public:
    BadWidget() {
        data1 = new int[100];
        data2 = new int[100];  // ⚠️ If this throws, data1 leaks!
    }
    ~BadWidget() {
        delete[] data1;
        delete[] data2;
    }
};

// ✅ SOLUTION: Use RAII/smart pointers
void no_leak() {
    auto ptr = std::make_unique<int>(42);
    // Automatic cleanup even with exceptions
}

class GoodWidget {
    std::unique_ptr<int[]> data1;
    std::unique_ptr<int[]> data2;
public:
    GoodWidget() 
        : data1(std::make_unique<int[]>(100)),
          data2(std::make_unique<int[]>(100)) {
        // If data2 throws, data1 automatically cleaned up
    }
};
```

### Double Delete

```cpp
// ❌ PITFALL: Deleting twice
void double_delete() {
    int* ptr = new int(42);
    delete ptr;
    delete ptr;  // ⚠️ Undefined behavior!
}

// ❌ PITFALL: Copying raw owning pointers
class BadClass {
    int* data;
public:
    BadClass() : data(new int(42)) {}
    ~BadClass() { delete data; }
    // ⚠️ Missing copy constructor/assignment
    // Default copies pointer, both objects try to delete!
};

void double_delete_via_copy() {
    BadClass obj1;
    BadClass obj2 = obj1;  // ⚠️ Shallow copy!
}  // ⚠️ Both destructors delete same pointer!

// ✅ SOLUTION: Smart pointers or rule of five
class GoodClass {
    std::unique_ptr<int> data;
public:
    GoodClass() : data(std::make_unique<int>(42)) {}
    // Compiler-generated destructor correct
    // Copy operations deleted (unique_ptr not copyable)
    // Move operations work correctly
};
```

### Dangling Pointers

```cpp
// ❌ PITFALL: Using pointer after delete
void dangling_pointer() {
    int* ptr = new int(42);
    delete ptr;
    std::cout << *ptr << "\n";  // ⚠️ Undefined behavior!
}

// ❌ PITFALL: Returning pointer to local
int* return_local() {
    int x = 42;
    return &x;  // ⚠️ x destroyed when function returns!
}

// ❌ PITFALL: Storing pointer to temporary
void pointer_to_temporary() {
    std::string* ptr;
    {
        std::string temp = "hello";
        ptr = &temp;
    }  // temp destroyed
    std::cout << *ptr << "\n";  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Don't store pointers to temporaries
std::string return_by_value() {
    std::string result = "hello";
    return result;  // ✅ Move or RVO
}
```

### Array vs Single Object Delete

```cpp
// ❌ PITFALL: Mismatched new/delete
void mismatch_delete() {
    int* single = new int(42);
    int* array = new int[10];
    
    delete[] single;  // ⚠️ Undefined behavior!
    delete array;     // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Match new/delete correctly
void correct_delete() {
    int* single = new int(42);
    int* array = new int[10];
    
    delete single;    // ✅ Correct
    delete[] array;   // ✅ Correct
}

// ✅ BETTER: Use smart pointers/containers
void modern_approach() {
    auto single = std::make_unique<int>(42);
    auto array = std::make_unique<int[]>(10);
    std::vector<int> vec(10);  // Even better!
}
```

---

## 16.2 Undefined Behavior

### Uninitialized Variables

```cpp
// ❌ PITFALL: Using uninitialized variables
void uninitialized() {
    int x;
    std::cout << x << "\n";  // ⚠️ Undefined behavior!
    
    int* ptr;
    *ptr = 42;  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Always initialize
void initialized() {
    int x = 0;  // ✅ Or use {}
    int y{};    // ✅ Zero-initialized
    
    int* ptr = nullptr;  // ✅ Safe default
    if (ptr) {
        *ptr = 42;
    }
}
```

### Signed Integer Overflow

```cpp
// ❌ PITFALL: Signed overflow is undefined
void signed_overflow() {
    int x = INT_MAX;
    ++x;  // ⚠️ Undefined behavior!
    
    int y = -1;
    y = y * INT_MIN;  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Check before operations
void safe_increment(int& x) {
    if (x < INT_MAX) {
        ++x;
    } else {
        // Handle overflow
    }
}

// ✅ OR: Use unsigned for wrapping behavior
void unsigned_wrapping() {
    unsigned int x = UINT_MAX;
    ++x;  // ✅ Defined: wraps to 0
}
```

### Out-of-Bounds Access

```cpp
// ❌ PITFALL: Array bounds violation
void out_of_bounds() {
    int arr[5] = {1, 2, 3, 4, 5};
    std::cout << arr[10] << "\n";  // ⚠️ Undefined behavior!
    
    std::vector<int> vec = {1, 2, 3};
    std::cout << vec[10] << "\n";  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Use at() for bounds checking
void bounds_checked() {
    std::vector<int> vec = {1, 2, 3};
    try {
        std::cout << vec.at(10) << "\n";  // ✅ Throws exception
    } catch (const std::out_of_range& e) {
        std::cerr << "Index out of range\n";
    }
}

// ✅ SOLUTION: Check size before access
void manual_check() {
    std::vector<int> vec = {1, 2, 3};
    size_t index = 10;
    if (index < vec.size()) {
        std::cout << vec[index] << "\n";
    }
}
```

### Null Pointer Dereference

```cpp
// ❌ PITFALL: Dereferencing null
void null_dereference() {
    int* ptr = nullptr;
    *ptr = 42;  // ⚠️ Undefined behavior!
    
    std::unique_ptr<int> smart = nullptr;
    *smart = 42;  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Always check before dereferencing
void safe_dereference(int* ptr) {
    if (ptr) {
        *ptr = 42;
    }
}

// ✅ SOLUTION: Use references when non-null guaranteed
void use_reference(int& ref) {
    ref = 42;  // ✅ References can't be null
}

// ✅ SOLUTION: Use optional for nullable values
std::optional<int> get_value(bool has_value) {
    if (has_value) {
        return 42;
    }
    return std::nullopt;
}
```

---

## 16.3 Object Lifetime Issues

### Use After Move

```cpp
// ❌ PITFALL: Using moved-from object
void use_after_move() {
    std::vector<int> vec1 = {1, 2, 3, 4, 5};
    std::vector<int> vec2 = std::move(vec1);
    
    std::cout << vec1.size() << "\n";  // ⚠️ Unspecified state!
    vec1.push_back(6);  // ⚠️ Dangerous!
}

// ✅ SOLUTION: Don't use after move, or reset
void correct_after_move() {
    std::vector<int> vec1 = {1, 2, 3, 4, 5};
    std::vector<int> vec2 = std::move(vec1);
    
    // Either don't use vec1, or:
    vec1 = std::vector<int>{};  // ✅ Reset to known state
    vec1.push_back(6);  // ✅ Now safe
}
```

### Iterator Invalidation

```cpp
// ❌ PITFALL: Using invalidated iterators
void invalidated_iterator() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        if (*it == 3) {
            vec.erase(it);  // ⚠️ Invalidates it!
            // ++it here would be undefined behavior
        }
    }
}

// ✅ SOLUTION: Use return value from erase
void correct_erase() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    for (auto it = vec.begin(); it != vec.end(); ) {
        if (*it == 3) {
            it = vec.erase(it);  // ✅ Get next valid iterator
        } else {
            ++it;
        }
    }
}

// ✅ BETTER: Use algorithms
void algorithm_approach() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    vec.erase(std::remove(vec.begin(), vec.end(), 3), vec.end());
}
```

### Returning Reference to Local

```cpp
// ❌ PITFALL: Returning reference to local variable
const std::string& bad_function() {
    std::string local = "hello";
    return local;  // ⚠️ Returns reference to destroyed object!
}

// ❌ PITFALL: Storing reference to temporary
void store_temp_reference() {
    const std::string& ref = std::string("hello");  // ⚠️ Temporary destroyed!
    std::cout << ref << "\n";  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Return by value (move/RVO)
std::string good_function() {
    std::string local = "hello";
    return local;  // ✅ Move or RVO
}

// ✅ SOLUTION: Store by value
void store_by_value() {
    std::string str = std::string("hello");  // ✅ Owns the data
    std::cout << str << "\n";
}
```

---

## 16.4 Const Correctness

### Missing Const

```cpp
// ❌ PITFALL: Non-const member functions
class BadContainer {
    std::vector<int> data;
public:
    int size() { return data.size(); }  // ⚠️ Should be const!
    int& at(size_t index) { return data.at(index); }
};

void use_const_object() {
    const BadContainer container;
    // container.size();  // ❌ Error: can't call non-const function
}

// ✅ SOLUTION: Mark non-modifying functions const
class GoodContainer {
    std::vector<int> data;
public:
    size_t size() const { return data.size(); }  // ✅ Const
    
    int& at(size_t index) { return data.at(index); }
    const int& at(size_t index) const { return data.at(index); }  // ✅ Const overload
};
```

### Const_cast Abuse

```cpp
// ❌ PITFALL: Casting away const to modify
void bad_const_cast() {
    const int x = 42;
    int& ref = const_cast<int&>(x);
    ref = 100;  // ⚠️ Undefined behavior!
}

// ✅ SOLUTION: Don't cast away const to modify
// If you need to modify, don't make it const in the first place
void correct_approach() {
    int x = 42;  // ✅ Not const if you need to modify
    x = 100;
}
```

---

## 16.5 Type Safety Issues

### Implicit Conversions

```cpp
// ❌ PITFALL: Unintended implicit conversions
class Widget {
    int value;
public:
    Widget(int v) : value(v) {}  // ⚠️ Allows implicit conversion
};

void take_widget(const Widget& w) {}

void implicit_conversion() {
    take_widget(42);  // ⚠️ Implicitly creates Widget(42)
    Widget w = 42;    // ⚠️ Implicit conversion
}

// ✅ SOLUTION: Use explicit constructors
class GoodWidget {
    int value;
public:
    explicit GoodWidget(int v) : value(v) {}  // ✅ No implicit conversion
};

void no_implicit() {
    // take_widget(42);  // ❌ Error
    take_widget(GoodWidget(42));  // ✅ Explicit
}
```

### Narrowing Conversions

```cpp
// ❌ PITFALL: Narrowing conversions lose data
void narrowing() {
    int large = 1000000;
    short small = large;  // ⚠️ Narrowing, may lose data
    
    double d = 3.14159;
    int i = d;  // ⚠️ Narrowing, loses precision
}

// ✅ SOLUTION: Use uniform initialization (prevents narrowing)
void no_narrowing() {
    int large = 1000000;
    // short small{large};  // ❌ Error: narrowing
    
    double d = 3.14159;
    // int i{d};  // ❌ Error: narrowing
    
    int i = static_cast<int>(d);  // ✅ Explicit intent
}
```

### C-style Casts

```cpp
// ❌ PITFALL: C-style casts are dangerous
void dangerous_casts() {
    int x = 42;
    double* ptr = (double*)&x;  // ⚠️ Reinterprets int as double!
    
    const int y = 100;
    int* mutable_ptr = (int*)&y;  // ⚠️ Casts away const!
}

// ✅ SOLUTION: Use C++ casts (explicit intent)
void safe_casts() {
    int x = 42;
    double d = static_cast<double>(x);  // ✅ Conversion
    
    // Reinterpret only when necessary
    double* ptr = reinterpret_cast<double*>(&x);  // ✅ Explicit intent
    
    const int y = 100;
    // int* p = const_cast<int*>(&y);  // ✅ Explicit, but still dangerous!
}
```

---

## 16.6 Performance Pitfalls

### Unnecessary Copies

```cpp
// ❌ PITFALL: Copying when not needed
void unnecessary_copies() {
    std::vector<std::string> vec = get_strings();
    
    // ❌ Copies each string
    for (std::string s : vec) {
        std::cout << s << "\n";
    }
    
    // ❌ Returns copy
    std::string get_name() {
        std::string name = "John";
        return name;  // Could be worse without RVO
    }
}

// ✅ SOLUTION: Use const references
void avoid_copies() {
    const std::vector<std::string>& vec = get_strings();
    
    // ✅ No copies
    for (const auto& s : vec) {
        std::cout << s << "\n";
    }
}

// ✅ SOLUTION: Use string_view for read-only
void use_string_view(std::string_view sv) {
    std::cout << sv << "\n";  // ✅ No copy
}
```

### Premature Pessimization

```cpp
// ❌ PITFALL: Preventing compiler optimizations
std::vector<int> bad_return() {
    std::vector<int> result;
    // ... fill result
    return std::move(result);  // ⚠️ Prevents RVO!
}

// ✅ SOLUTION: Let compiler optimize
std::vector<int> good_return() {
    std::vector<int> result;
    // ... fill result
    return result;  // ✅ RVO or move
}
```

### Inefficient Algorithms

```cpp
// ❌ PITFALL: O(n²) when O(n) possible
bool has_duplicate_slow(const std::vector<int>& vec) {
    for (size_t i = 0; i < vec.size(); ++i) {
        for (size_t j = i + 1; j < vec.size(); ++j) {
            if (vec[i] == vec[j]) return true;
        }
    }
    return false;
}

// ✅ SOLUTION: Use appropriate data structure
bool has_duplicate_fast(const std::vector<int>& vec) {
    std::unordered_set<int> seen;
    for (int val : vec) {
        if (!seen.insert(val).second) return true;
    }
    return false;
}
```

---

## 16.7 Compilation and Linking Issues

### Missing Virtual Destructor

```cpp
// ❌ PITFALL: Non-virtual destructor in base class
class BadBase {
public:
    ~BadBase() { /* cleanup */ }  // ⚠️ Not virtual!
};

class Derived : public BadBase {
    int* data;
public:
    Derived() : data(new int[100]) {}
    ~Derived() { delete[] data; }
};

void memory_leak() {
    BadBase* ptr = new Derived();
    delete ptr;  // ⚠️ Only ~BadBase called, ~Derived not called!
}

// ✅ SOLUTION: Virtual destructor in base
class GoodBase {
public:
    virtual ~GoodBase() = default;  // ✅ Virtual destructor
};
```

### One Definition Rule (ODR) Violations

```cpp
// header.h
// ❌ PITFALL: Function definition in header (no inline)
void function() {  // ⚠️ Defined in header!
    // ...
}

// ✅ SOLUTION: Declare in header, define in source
// header.h
void function();  // ✅ Declaration only

// source.cpp
void function() {  // ✅ Definition in source
    // ...
}

// OR: Use inline
// header.h
inline void function() {  // ✅ Inline definition OK
    // ...
}
```

---

## 16.8 Best Practices Summary

### The Safe Subset

```cpp
// ✅ Always follow these rules:

// 1. Use RAII for resource management
void best_practice_1() {
    std::unique_ptr<int> ptr = std::make_unique<int>(42);
    std::ifstream file("data.txt");
    std::lock_guard<std::mutex> lock(mtx);
}

// 2. Prefer const correctness
class Container {
    std::vector<int> data;
public:
    size_t size() const { return data.size(); }  // ✅ Const
};

// 3. Use explicit constructors
class Widget {
public:
    explicit Widget(int x) {}  // ✅ Explicit
};

// 4. Mark move operations noexcept
class Movable {
public:
    Movable(Movable&&) noexcept = default;  // ✅ noexcept
};

// 5. Use override keyword
class Derived : public Base {
    void foo() override {}  // ✅ Override
};

// 6. Initialize all variables
void init_vars() {
    int x{};  // ✅ Zero-initialized
    int* ptr = nullptr;  // ✅ Initialized
}

// 7. Use range-based for loops
void range_for(const std::vector<int>& vec) {
    for (const auto& val : vec) {  // ✅ No indices
        process(val);
    }
}

// 8. Prefer algorithms over raw loops
void use_algorithms(std::vector<int>& vec) {
    std::sort(vec.begin(), vec.end());  // ✅ Clear intent
}
```

---

## Conclusion

Part 16 covered common C++ pitfalls:

1. **Memory Management** - Leaks, double delete, dangling pointers
2. **Undefined Behavior** - Uninitialized variables, overflow, out-of-bounds
3. **Object Lifetime** - Use after move, iterator invalidation
4. **Const Correctness** - Missing const, const_cast abuse
5. **Type Safety** - Implicit conversions, narrowing
6. **Performance** - Unnecessary copies, inefficient algorithms
7. **Compilation** - Virtual destructors, ODR violations
8. **Best Practices** - Safe subset to follow

**Prevention Checklist:**
- [ ] Use smart pointers, not raw new/delete
- [ ] Initialize all variables
- [ ] Mark functions const when appropriate
- [ ] Use explicit constructors
- [ ] Mark move operations noexcept
- [ ] Add virtual destructors to base classes
- [ ] Check array bounds
- [ ] Use C++ casts, not C-style
- [ ] Prefer const references to copies
- [ ] Use appropriate algorithms

**Next:** Part 17 covers Practical Projects & Case Studies.

---

*This blog post is part of the Complete C++ Mastery series.*
