---
title: "Complete C++ Mastery Part 12: Modern C++ Best Practices (C++11/14/17/20)"
date: 2024-02-21 08:00:00 +0000
categories: [C++, Programming, Best-Practices]
tags: [cpp, modern-cpp, best-practices, cpp11, cpp14, cpp17, cpp20, guidelines]
---

# Complete C++ Mastery Part 12: Modern C++ Best Practices (C++11/14/17/20)

## Introduction

Welcome to Part 12 of the Complete C++ Mastery series. Modern C++ (C++11 and beyond) transformed the language, making it safer, more expressive, and easier to use. This part consolidates best practices for writing modern C++ code: what to use, what to avoid, and why. We'll explore idiomatic patterns, C++ Core Guidelines, and practical recommendations for each C++ standard.

This is about writing code that's not just correct, but maintainable, safe, and expressive—code that leverages modern C++ features to their fullest.

**Prerequisites:** This part synthesizes knowledge from Parts 1-11, especially modern features introduced throughout.

---

## 12.1 General Best Practices

### Prefer RAII Over Manual Resource Management

```cpp
#include <fstream>
#include <memory>
#include <mutex>

// ❌ BAD: Manual resource management
void bad_example() {
    FILE* file = fopen("data.txt", "r");
    if (!file) return;
    
    // ... process file
    // If exception thrown, file leaks!
    
    fclose(file);
}

// ✅ GOOD: RAII handles cleanup automatically
void good_example() {
    std::ifstream file("data.txt");
    if (!file) return;
    
    // ... process file
    // File automatically closed, even with exceptions
}

// ✅ GOOD: Smart pointers for heap memory
void smart_pointer_example() {
    auto ptr = std::make_unique<int>(42);
    // No delete needed, automatically freed
}

// ✅ GOOD: Lock guards for mutexes
void thread_safe_function() {
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);
    // Critical section
    // Mutex automatically unlocked
}
```

### Use const Correctness

```cpp
#include <vector>
#include <string>

class DataProcessor {
    std::vector<int> data;
    
public:
    // ✅ Mark member functions const when they don't modify state
    size_t size() const { return data.size(); }
    bool empty() const { return data.empty(); }
    
    // ✅ Return const reference for read-only access
    const std::vector<int>& get_data() const { return data; }
    
    // ✅ Take const reference for input parameters
    void process(const std::string& input) {
        // Process input without modifying it
    }
    
    // ✅ Use const for variables that don't change
    void calculate() {
        const double pi = 3.14159;
        const size_t n = data.size();
        // Use pi and n...
    }
};

// ✅ Const correctness enables compiler optimizations
void example(const std::vector<int>& vec) {
    // Compiler knows vec won't be modified
    for (const auto& val : vec) {  // const reference
        std::cout << val << "\n";
    }
}
```

### Prefer Standard Library to Raw Loops

```cpp
#include <algorithm>
#include <numeric>
#include <vector>

// ❌ BAD: Raw loop
int sum_bad(const std::vector<int>& vec) {
    int sum = 0;
    for (size_t i = 0; i < vec.size(); ++i) {
        sum += vec[i];
    }
    return sum;
}

// ✅ GOOD: Use STL algorithm
int sum_good(const std::vector<int>& vec) {
    return std::accumulate(vec.begin(), vec.end(), 0);
}

// ❌ BAD: Manual search
bool contains_bad(const std::vector<int>& vec, int target) {
    for (const auto& val : vec) {
        if (val == target) return true;
    }
    return false;
}

// ✅ GOOD: Use std::find
bool contains_good(const std::vector<int>& vec, int target) {
    return std::find(vec.begin(), vec.end(), target) != vec.end();
}

// ✅ GOOD: Transform with lambda
std::vector<int> double_values(const std::vector<int>& vec) {
    std::vector<int> result;
    std::transform(vec.begin(), vec.end(), std::back_inserter(result),
                   [](int x) { return x * 2; });
    return result;
}
```

---

## 12.2 C++11 Best Practices

### Use auto Judiciously

```cpp
#include <vector>
#include <map>
#include <memory>

void auto_guidelines() {
    // ✅ GOOD: Obvious types
    auto vec = std::vector<int>{1, 2, 3};
    auto ptr = std::make_unique<int>(42);
    
    // ✅ GOOD: Complex iterator types
    std::map<std::string, std::vector<int>> map;
    auto it = map.begin();  // Instead of map<string, vector<int>>::iterator
    
    // ✅ GOOD: Template return types
    auto result = some_template_function();
    
    // ❌ BAD: Unclear types
    auto x = get_value();  // What type is x?
    
    // ✅ BETTER: Be explicit when unclear
    int count = get_count();
    std::string name = get_name();
    
    // ✅ GOOD: Range-based for loops
    for (const auto& item : vec) {  // auto is clear here
        // Process item
    }
}
```

### Use nullptr Instead of NULL

```cpp
#include <iostream>

void process(int x) {
    std::cout << "Processing int: " << x << "\n";
}

void process(int* p) {
    std::cout << "Processing pointer\n";
}

void nullptr_example() {
    // ❌ BAD: NULL is ambiguous
    // process(NULL);  // Error: ambiguous call!
    
    // ✅ GOOD: nullptr is unambiguous
    process(nullptr);  // Calls pointer version
    
    // ✅ Checking for null
    int* ptr = nullptr;
    if (ptr == nullptr) {
        std::cout << "Pointer is null\n";
    }
}
```

### Use Override and Final

```cpp
#include <iostream>

class Base {
public:
    virtual ~Base() = default;
    virtual void foo() { std::cout << "Base::foo\n"; }
    virtual void bar() { std::cout << "Base::bar\n"; }
};

class Derived : public Base {
public:
    // ✅ GOOD: override catches typos/signature mismatches
    void foo() override { std::cout << "Derived::foo\n"; }
    
    // ❌ Would be error without override:
    // void foo() const override;  // Error: doesn't override anything
    
    // ✅ GOOD: final prevents further overriding
    void bar() final { std::cout << "Derived::bar\n"; }
};

class FurtherDerived : public Derived {
public:
    // void bar() override;  // ❌ Error: bar is final
};
```

### Use Strongly-Typed Enums

```cpp
// ❌ BAD: Old-style enum
enum Color {
    RED,    // Pollutes namespace
    GREEN,
    BLUE
};

// ✅ GOOD: Scoped enum (C++11)
enum class TrafficLight {
    Red,
    Yellow,
    Green
};

void enum_examples() {
    // ❌ BAD: Name collision
    // enum Color { RED };  // Error: RED already defined
    
    // ✅ GOOD: No collision
    TrafficLight light = TrafficLight::Red;
    
    // ✅ Can't implicitly convert to int
    // int x = TrafficLight::Red;  // Error
    int x = static_cast<int>(TrafficLight::Red);  // Explicit
    
    // ✅ Specify underlying type
    enum class Status : uint8_t {
        OK = 0,
        Error = 1
    };
}
```

### Use Range-Based For Loops

```cpp
#include <vector>
#include <map>

void range_for_examples() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // ✅ GOOD: Simple iteration
    for (const auto& val : vec) {
        std::cout << val << "\n";
    }
    
    // ✅ GOOD: Modification
    for (auto& val : vec) {
        val *= 2;
    }
    
    // ✅ GOOD: With structured bindings (C++17)
    std::map<std::string, int> map = {{"a", 1}, {"b", 2}};
    for (const auto& [key, value] : map) {
        std::cout << key << ": " << value << "\n";
    }
}
```

### Use Lambda Expressions

```cpp
#include <algorithm>
#include <vector>
#include <functional>

void lambda_examples() {
    std::vector<int> vec = {5, 2, 8, 1, 9};
    
    // ✅ GOOD: Sort with lambda comparator
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return a > b;  // Descending order
    });
    
    // ✅ GOOD: Lambda with captures
    int threshold = 5;
    auto count = std::count_if(vec.begin(), vec.end(),
                                [threshold](int x) { return x > threshold; });
    
    // ✅ GOOD: Generic lambda (C++14)
    auto print = [](const auto& x) {
        std::cout << x << "\n";
    };
    print(42);
    print("hello");
    
    // ✅ GOOD: Mutable lambda
    int counter = 0;
    auto increment = [counter]() mutable {
        return ++counter;
    };
}
```

---

## 12.3 C++14 Best Practices

### Use std::make_unique

```cpp
#include <memory>

class Widget {
public:
    Widget(int x, double y) {}
};

void make_unique_example() {
    // ❌ BAD: Direct new
    std::unique_ptr<Widget> ptr1(new Widget(1, 2.0));
    
    // ✅ GOOD: make_unique (C++14)
    auto ptr2 = std::make_unique<Widget>(1, 2.0);
    
    // Benefits:
    // 1. Exception safe
    // 2. No repeated type name
    // 3. Consistent with make_shared
}
```

### Use Generic Lambdas

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// ✅ Generic lambda (C++14)
auto print = [](const auto& value) {
    std::cout << value << "\n";
};

void generic_lambda_example() {
    print(42);
    print(3.14);
    print("hello");
    
    // ✅ In algorithms
    std::vector<int> vec = {1, 2, 3};
    std::for_each(vec.begin(), vec.end(), [](auto x) {
        std::cout << x * 2 << "\n";
    });
}
```

### Use Binary Literals and Digit Separators

```cpp
void cpp14_literals() {
    // ✅ Binary literals
    int flags = 0b1010'1100;  // Binary with digit separator
    
    // ✅ Digit separators for readability
    int million = 1'000'000;
    long long large = 9'223'372'036'854'775'807;
    double pi = 3.141'592'653'589'793;
}
```

---

## 12.4 C++17 Best Practices

### Use Structured Bindings

```cpp
#include <map>
#include <tuple>
#include <array>

struct Point { int x, y; };

void structured_bindings() {
    // ✅ With tuples
    std::tuple<int, double, std::string> t = {42, 3.14, "hello"};
    auto [i, d, s] = t;
    
    // ✅ With pairs
    std::map<std::string, int> map = {{"age", 30}};
    for (const auto& [key, value] : map) {
        std::cout << key << ": " << value << "\n";
    }
    
    // ✅ With structs
    Point p{10, 20};
    auto [x, y] = p;
    
    // ✅ With arrays
    std::array<int, 3> arr = {1, 2, 3};
    auto [a, b, c] = arr;
}
```

### Use std::optional

```cpp
#include <optional>
#include <string>

// ✅ Return optional instead of special value or exception
std::optional<int> find_index(const std::vector<int>& vec, int target) {
    auto it = std::find(vec.begin(), vec.end(), target);
    if (it != vec.end()) {
        return std::distance(vec.begin(), it);
    }
    return std::nullopt;
}

void optional_example() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    if (auto idx = find_index(vec, 3)) {
        std::cout << "Found at index: " << *idx << "\n";
    } else {
        std::cout << "Not found\n";
    }
    
    // ✅ Value or default
    int index = find_index(vec, 10).value_or(-1);
}
```

### Use std::string_view

```cpp
#include <string_view>
#include <iostream>

// ✅ GOOD: string_view for read-only strings (no copy)
void process(std::string_view sv) {
    std::cout << sv << "\n";
}

void string_view_example() {
    std::string s = "hello";
    const char* cstr = "world";
    
    process(s);     // No copy
    process(cstr);  // No copy
    process("literal");  // No copy
    
    // ⚠️ Warning: Ensure lifetime
    std::string_view sv;
    {
        std::string temp = "temporary";
        sv = temp;  // ⚠️ Dangerous!
    }
    // sv now points to destroyed string!
}
```

### Use if/switch with Initializer

```cpp
#include <map>
#include <mutex>

void if_with_init() {
    std::map<std::string, int> map = {{"key", 42}};
    
    // ✅ C++17: if with initializer
    if (auto it = map.find("key"); it != map.end()) {
        std::cout << "Found: " << it->second << "\n";
    }
    // it is out of scope here
    
    // ✅ With lock_guard
    std::mutex mtx;
    if (std::lock_guard lock(mtx); true) {
        // Critical section
    }
}
```

### Use constexpr if

```cpp
#include <type_traits>

template<typename T>
auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 1.5;
    } else {
        return value;
    }
}
```

---

## 12.5 C++20 Best Practices

### Use Concepts

```cpp
#include <concepts>
#include <iostream>

// ✅ Define concept
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

// ✅ Use concept in function
template<Numeric T>
T square(T value) {
    return value * value;
}

// ✅ Abbreviated function template
void print(std::integral auto value) {
    std::cout << "Integer: " << value << "\n";
}

// ✅ Multiple constraints
template<typename T>
requires std::integral<T> && std::signed_integral<T>
T absolute(T value) {
    return value < 0 ? -value : value;
}
```

### Use Ranges

```cpp
#include <ranges>
#include <vector>
#include <iostream>

void ranges_example() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // ✅ Ranges: composable, lazy evaluation
    auto even_doubled = vec
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * 2; });
    
    for (int val : even_doubled) {
        std::cout << val << " ";  // 4 8 12 16 20
    }
}
```

### Use Three-Way Comparison (Spaceship Operator)

```cpp
#include <compare>

struct Point {
    int x, y;
    
    // ✅ C++20: Auto-generate all comparison operators
    auto operator<=>(const Point&) const = default;
};

void spaceship_example() {
    Point p1{1, 2};
    Point p2{1, 3};
    
    bool eq = (p1 == p2);
    bool ne = (p1 != p2);
    bool lt = (p1 < p2);
    bool le = (p1 <= p2);
    bool gt = (p1 > p2);
    bool ge = (p1 >= p2);
    // All operators available from single <=> definition!
}
```

### Use designated initializers

```cpp
struct Config {
    int width = 800;
    int height = 600;
    bool fullscreen = false;
    int fps = 60;
};

void designated_init() {
    // ✅ C++20: Designated initializers
    Config cfg {
        .width = 1920,
        .height = 1080,
        .fullscreen = true
        // fps uses default value
    };
}
```

---

## 12.6 Common Antipatterns to Avoid

### Don't Use Raw Pointers for Ownership

```cpp
// ❌ BAD: Raw pointer ownership unclear
class Bad {
    int* data;
public:
    Bad() : data(new int(42)) {}
    ~Bad() { delete data; }
    // Missing copy/move operations!
};

// ✅ GOOD: Smart pointer makes ownership clear
class Good {
    std::unique_ptr<int> data;
public:
    Good() : data(std::make_unique<int>(42)) {}
    // Compiler generates correct copy/move
};
```

### Don't Use C-Style Casts

```cpp
// ❌ BAD: C-style cast (dangerous)
double d = 3.14;
int* p = (int*)&d;  // Unsafe!

// ✅ GOOD: Use C++ casts
auto i = static_cast<int>(d);           // Safe conversion
auto ptr = reinterpret_cast<int*>(&d); // Explicit intent
const int x = 42;
auto mutable_x = const_cast<int&>(x);   // Explicit removal of const
```

### Don't Use new/delete Directly

```cpp
// ❌ BAD: Manual memory management
void bad() {
    int* ptr = new int(42);
    // ... code that might throw
    delete ptr;  // Might never execute!
}

// ✅ GOOD: RAII with smart pointers
void good() {
    auto ptr = std::make_unique<int>(42);
    // ... code that might throw
    // Automatic cleanup
}

// ✅ GOOD: Use containers
void better() {
    std::vector<int> vec;
    vec.push_back(42);
    // No manual memory management
}
```

### Don't Ignore Return Values

```cpp
#include <optional>

std::optional<int> compute() {
    return 42;
}

// ❌ BAD: Ignored return value
void bad() {
    compute();  // Warning: ignoring return value
}

// ✅ GOOD: Check return value
void good() {
    if (auto result = compute()) {
        std::cout << *result << "\n";
    }
}

// ✅ Use [[nodiscard]] attribute
[[nodiscard]] std::optional<int> important_compute() {
    return 42;
}
```

---

## 12.7 C++ Core Guidelines Summary

### Resource Management
- **R.1**: Use RAII for resource management
- **R.3**: Use `unique_ptr` for exclusive ownership
- **R.4**: Use `shared_ptr` for shared ownership
- **R.5**: Prefer scoped objects, don't heap allocate unnecessarily

### Functions
- **F.1**: Package meaningful operations as functions
- **F.3**: Keep functions short and simple
- **F.7**: For general use, take parameters by `const&`
- **F.15**: Prefer simple and conventional parameter passing

### Classes
- **C.2**: Use class if any member is non-public
- **C.4**: Make a function a member only if it needs direct access
- **C.21**: Use `= default` for default operations
- **C.45**: Don't define a default constructor that only initializes members

### Enums
- **Enum.3**: Prefer `enum class` over plain `enum`

### Expressions
- **ES.5**: Keep scopes small
- **ES.10**: Declare one name per declaration
- **ES.23**: Prefer the `{}` initializer syntax

### Performance
- **Per.1**: Don't optimize without reason
- **Per.2**: Don't optimize prematurely
- **Per.3**: Don't optimize something that's not performance critical

---

## Comprehensive Example: Modern C++ Style

```cpp
#include <iostream>
#include <vector>
#include <memory>
#include <optional>
#include <string_view>
#include <algorithm>
#include <ranges>

// ✅ Modern C++ class with all best practices
class DataProcessor {
public:
    // ✅ Use concepts (C++20)
    template<std::ranges::range R>
    explicit DataProcessor(R&& data)
        : values(std::ranges::begin(data), std::ranges::end(data)) {}
    
    // ✅ Use string_view for read-only strings
    void set_name(std::string_view name) {
        this->name = std::string(name);
    }
    
    // ✅ Use optional for nullable returns
    [[nodiscard]] std::optional<int> find_max() const {
        if (values.empty()) return std::nullopt;
        return *std::max_element(values.begin(), values.end());
    }
    
    // ✅ Use ranges (C++20)
    auto get_filtered(int threshold) const {
        return values
            | std::views::filter([threshold](int x) { return x > threshold; })
            | std::views::transform([](int x) { return x * 2; });
    }
    
    // ✅ Const correctness
    [[nodiscard]] size_t size() const noexcept { return values.size(); }
    [[nodiscard]] bool empty() const noexcept { return values.empty(); }
    
private:
    std::vector<int> values;
    std::string name;
};

// ✅ Factory function with deduction guide
auto create_processor(const std::vector<int>& data) {
    return std::make_unique<DataProcessor>(data);
}

int main() {
    // ✅ Use auto with clear types
    auto processor = create_processor({1, 2, 3, 4, 5, 6, 7, 8, 9, 10});
    
    // ✅ Structured bindings would go here if returning pairs/tuples
    
    // ✅ Use optional
    if (auto max = processor->find_max()) {
        std::cout << "Max: " << *max << "\n";
    }
    
    // ✅ Range-based for with structured binding
    for (auto value : processor->get_filtered(5)) {
        std::cout << value << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

---

## Conclusion

Part 12 consolidated modern C++ best practices:

1. **General Principles** - RAII, const correctness, prefer STL
2. **C++11** - auto, nullptr, override, enum class, lambdas
3. **C++14** - make_unique, generic lambdas
4. **C++17** - structured bindings, optional, string_view
5. **C++20** - concepts, ranges, spaceship operator
6. **Antipatterns** - What to avoid
7. **Core Guidelines** - Industry standards

**Key Takeaways:**
- Use RAII for all resource management
- Prefer smart pointers over raw pointers
- Use const correctness everywhere
- Leverage STL algorithms over raw loops
- Use modern features: auto, lambdas, ranges
- Apply Core Guidelines consistently

**Modern C++ Checklist:**
- [ ] Use smart pointers (unique_ptr, shared_ptr)
- [ ] Mark functions const, noexcept, override
- [ ] Use auto for complex types
- [ ] Prefer enum class over enum
- [ ] Use structured bindings (C++17)
- [ ] Use optional instead of null checks
- [ ] Apply concepts for constraints (C++20)
- [ ] Use ranges for composable operations (C++20)

**Next in the series:** Part 13 will cover Domain-Specific Topics.

---

## References

- **C++ Core Guidelines**: https://isocpp.github.io/CppCoreGuidelines/
- **"Effective Modern C++"** by Scott Meyers
- **cppreference.com**: Modern C++ features

---

*This blog post is part of the Complete C++ Mastery series.*
