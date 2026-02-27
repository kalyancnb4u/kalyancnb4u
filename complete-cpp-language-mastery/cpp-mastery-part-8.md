---
title: "Complete C++ Mastery Part 8: Advanced C++ Features"
date: 2024-02-21 00:00:00 +0000
categories: [C++, Programming, Advanced]
tags: [cpp, advanced, type-deduction, sfinae, attributes, modules, literals]
---

# Complete C++ Mastery Part 8: Advanced C++ Features

## Introduction

Welcome to Part 8 of the Complete C++ Mastery series. Modern C++ (C++11 and beyond) introduced numerous advanced features that enable more expressive, safer, and efficient code. This part explores these sophisticated capabilities: type deduction, move semantics deep dive, SFINAE and metaprogramming techniques, type traits, attributes, user-defined literals, and C++20 modules.

These features push the boundaries of what's possible in C++, enabling library developers to create elegant APIs and application developers to write cleaner, more maintainable code.

**Prerequisites:** This part assumes thorough knowledge from Parts 1-7, especially templates, move semantics, and metaprogramming.

---

## 8.1 Type Deduction and auto

### Basic auto Usage

```cpp
#include <iostream>
#include <vector>
#include <map>

int main() {
    // Basic type deduction
    auto x = 42;              // int
    auto y = 3.14;            // double
    auto s = "hello";         // const char*
    auto str = std::string("world");  // std::string
    
    // Deducing container types
    std::vector<int> vec = {1, 2, 3};
    auto it = vec.begin();    // std::vector<int>::iterator
    
    // Complex type deduction
    std::map<std::string, std::vector<int>> map;
    auto pair = map.insert({"key", {1, 2, 3}});
    // std::pair<std::map<...>::iterator, bool>
    
    // Function return type
    auto lambda = [](int x) { return x * 2; };
    auto result = lambda(5);  // int
    
    return 0;
}
```

### auto Rules and Nuances

```cpp
#include <iostream>

int main() {
    int x = 42;
    const int cx = x;
    const int& rx = x;
    
    // auto drops const and references
    auto a1 = cx;   // int (const dropped)
    auto a2 = rx;   // int (const and reference dropped)
    
    // Keep const with explicit const
    const auto a3 = cx;   // const int
    
    // Keep reference with &
    auto& a4 = rx;        // const int& (preserves const)
    auto& a5 = x;         // int&
    
    // Universal reference
    auto&& a6 = x;        // int& (lvalue)
    auto&& a7 = 42;       // int&& (rvalue)
    
    // Pointer deduction
    int* p = &x;
    auto a8 = p;          // int*
    auto* a9 = p;         // int*
    
    return 0;
}
```

### decltype

```cpp
#include <iostream>
#include <vector>

int func() { return 42; }

int main() {
    int x = 10;
    
    // decltype deduces exact type
    decltype(x) y = 20;           // int
    decltype((x)) z = x;          // int& (note extra parentheses)
    decltype(func()) result = 0;  // int
    
    // decltype with containers
    std::vector<int> vec = {1, 2, 3};
    decltype(vec)::value_type val = 5;  // int
    
    // decltype(auto) - preserves references and const (C++14)
    const int cx = 42;
    decltype(auto) a1 = cx;       // const int
    decltype(auto) a2 = (cx);     // const int&
    
    return 0;
}
```

### Trailing Return Type

```cpp
#include <iostream>
#include <vector>

// C++11: Trailing return type
template<typename Container>
auto get_first(Container& c) -> decltype(c[0]) {
    return c[0];
}

// C++14: Return type deduction
template<typename Container>
auto get_second(Container& c) {
    return c[1];
}

// C++14: decltype(auto) preserves references
template<typename Container>
decltype(auto) get_third(Container& c) {
    return c[2];  // Returns reference if c[2] is reference
}

int main() {
    std::vector<int> vec = {10, 20, 30, 40};
    
    auto val1 = get_first(vec);   // int (copy)
    auto val2 = get_second(vec);  // int (copy)
    auto& val3 = get_third(vec);  // int& (reference)
    
    val3 = 99;
    std::cout << vec[2] << "\n";  // 99 (modified through reference)
    
    return 0;
}
```

---

## 8.2 Move Semantics Deep Dive

### Perfect Forwarding

```cpp
#include <iostream>
#include <utility>

void process(int& x) {
    std::cout << "Lvalue: " << x << "\n";
}

void process(int&& x) {
    std::cout << "Rvalue: " << x << "\n";
}

// Perfect forwarding wrapper
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}

int main() {
    int x = 42;
    
    wrapper(x);    // Forwards as lvalue
    wrapper(100);  // Forwards as rvalue
    
    return 0;
}
```

### Universal References vs Rvalue References

```cpp
#include <iostream>
#include <vector>

// Rvalue reference: specific type
void func1(std::vector<int>&& v) {
    std::cout << "Rvalue reference to vector\n";
}

// Universal reference: template with T&&
template<typename T>
void func2(T&& v) {
    std::cout << "Universal reference\n";
}

// Universal reference: auto&&
void func3() {
    auto&& x = 42;  // Universal reference
}

int main() {
    std::vector<int> vec;
    
    // func1(vec);  // ❌ Error: lvalue cannot bind to rvalue reference
    func1(std::move(vec));  // ✅ OK
    
    func2(vec);          // ✅ OK: T deduced as std::vector<int>&
    func2(std::move(vec)); // ✅ OK: T deduced as std::vector<int>
    
    return 0;
}
```

### Reference Collapsing Rules

```cpp
/*
Reference Collapsing Rules:
& &   → &
& &&  → &
&& &  → &
&& && → &&
*/

template<typename T>
void func(T&& param) {
    // If T = int&:   T&& = int& && → int&
    // If T = int:    T&& = int&&
}

int main() {
    int x = 42;
    
    func(x);   // T = int&,  param is int&
    func(10);  // T = int,   param is int&&
    
    return 0;
}
```

### Move-Only Types

```cpp
#include <iostream>
#include <memory>
#include <thread>

class MoveOnly {
    std::unique_ptr<int> data;
    
public:
    MoveOnly(int value) : data(std::make_unique<int>(value)) {}
    
    // Delete copy operations
    MoveOnly(const MoveOnly&) = delete;
    MoveOnly& operator=(const MoveOnly&) = delete;
    
    // Default move operations
    MoveOnly(MoveOnly&&) = default;
    MoveOnly& operator=(MoveOnly&&) = default;
    
    int get_value() const { return *data; }
};

MoveOnly create_move_only() {
    return MoveOnly(42);
}

int main() {
    // Move construction
    MoveOnly obj1(10);
    MoveOnly obj2 = std::move(obj1);
    
    // Move from function return
    MoveOnly obj3 = create_move_only();
    
    // Store in container (moves)
    std::vector<MoveOnly> vec;
    vec.push_back(MoveOnly(100));
    vec.push_back(std::move(obj2));
    
    // Thread with move-only lambda capture
    std::thread t([obj = std::move(obj3)]() {
        std::cout << "Value: " << obj.get_value() << "\n";
    });
    t.join();
    
    return 0;
}
```

### std::move and std::forward Implementation

```cpp
#include <type_traits>

// Simplified std::move implementation
template<typename T>
typename std::remove_reference<T>::type&& move(T&& arg) noexcept {
    return static_cast<typename std::remove_reference<T>::type&&>(arg);
}

// Simplified std::forward implementation
template<typename T>
T&& forward(typename std::remove_reference<T>::type& arg) noexcept {
    return static_cast<T&&>(arg);
}

template<typename T>
T&& forward(typename std::remove_reference<T>::type&& arg) noexcept {
    static_assert(!std::is_lvalue_reference<T>::value,
                  "Cannot forward rvalue as lvalue");
    return static_cast<T&&>(arg);
}
```

---

## 8.3 SFINAE and Type Traits

### Substitution Failure Is Not An Error

```cpp
#include <iostream>
#include <type_traits>

// SFINAE: Only enabled for integral types
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T value) {
    std::cout << "Processing integral: " << value << "\n";
    return value * 2;
}

// SFINAE: Only enabled for floating-point types
template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
process(T value) {
    std::cout << "Processing float: " << value << "\n";
    return value * 1.5;
}

int main() {
    std::cout << process(42) << "\n";      // Calls integral version
    std::cout << process(3.14) << "\n";    // Calls floating-point version
    // process("hello");  // ❌ Error: no matching function
    
    return 0;
}
```

### Detection Idiom

```cpp
#include <iostream>
#include <type_traits>

// Detect if type has size() method
template<typename T, typename = void>
struct has_size_method : std::false_type {};

template<typename T>
struct has_size_method<T, std::void_t<decltype(std::declval<T>().size())>>
    : std::true_type {};

int main() {
    std::cout << "vector has size(): " 
              << has_size_method<std::vector<int>>::value << "\n";  // true
    std::cout << "int has size(): " 
              << has_size_method<int>::value << "\n";  // false
    
    return 0;
}
```

### if constexpr (C++17)

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
auto get_value(T t) {
    if constexpr (std::is_pointer_v<T>) {
        return *t;  // Dereference pointer
    } else {
        return t;   // Return as-is
    }
}

template<typename T>
void process(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "Integer: " << value << "\n";
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "Float: " << value << "\n";
    } else if constexpr (std::is_pointer_v<T>) {
        std::cout << "Pointer to: " << *value << "\n";
    } else {
        std::cout << "Other type\n";
    }
}

int main() {
    int x = 42;
    int* p = &x;
    
    std::cout << get_value(x) << "\n";   // 42
    std::cout << get_value(p) << "\n";   // 42
    
    process(10);
    process(3.14);
    process(p);
    
    return 0;
}
```

### Type Traits

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void analyze_type() {
    std::cout << "Type analysis:\n";
    std::cout << "  is_integral: " << std::is_integral_v<T> << "\n";
    std::cout << "  is_floating_point: " << std::is_floating_point_v<T> << "\n";
    std::cout << "  is_pointer: " << std::is_pointer_v<T> << "\n";
    std::cout << "  is_reference: " << std::is_reference_v<T> << "\n";
    std::cout << "  is_const: " << std::is_const_v<T> << "\n";
    std::cout << "  is_trivial: " << std::is_trivial_v<T> << "\n";
    std::cout << "  is_standard_layout: " << std::is_standard_layout_v<T> << "\n";
}

int main() {
    analyze_type<int>();
    std::cout << "\n";
    analyze_type<const double*>();
    
    return 0;
}
```

### Custom Type Traits

```cpp
#include <iostream>
#include <type_traits>

// Check if type is a container (has begin/end)
template<typename T, typename = void>
struct is_container : std::false_type {};

template<typename T>
struct is_container<T, std::void_t<
    decltype(std::declval<T>().begin()),
    decltype(std::declval<T>().end())
>> : std::true_type {};

// Helper variable template (C++17)
template<typename T>
inline constexpr bool is_container_v = is_container<T>::value;

int main() {
    std::cout << "vector is container: " 
              << is_container_v<std::vector<int>> << "\n";  // true
    std::cout << "int is container: " 
              << is_container_v<int> << "\n";  // false
    
    return 0;
}
```

---

## 8.4 Attributes (C++11/14/17/20)

### Standard Attributes

```cpp
#include <iostream>

// [[noreturn]] - Function never returns
[[noreturn]] void fatal_error(const char* msg) {
    std::cerr << "Fatal: " << msg << "\n";
    std::exit(1);
}

// [[deprecated]] - Mark as deprecated
[[deprecated("Use new_function instead")]]
void old_function() {
    std::cout << "Old function\n";
}

void new_function() {
    std::cout << "New function\n";
}

// [[nodiscard]] - Warn if return value ignored (C++17)
[[nodiscard]] int compute() {
    return 42;
}

// [[maybe_unused]] - Suppress unused warnings (C++17)
void function([[maybe_unused]] int param) {
    // param might not be used in all code paths
}

// [[fallthrough]] - Intentional switch fallthrough (C++17)
void process(int value) {
    switch (value) {
        case 1:
            std::cout << "One\n";
            [[fallthrough]];
        case 2:
            std::cout << "One or Two\n";
            break;
        default:
            std::cout << "Other\n";
    }
}

int main() {
    // old_function();  // Warning: deprecated
    new_function();
    
    compute();  // Warning: nodiscard return value ignored
    int result = compute();  // OK
    
    process(1);
    
    return 0;
}
```

### C++20 Attributes

```cpp
// [[likely]] / [[unlikely]] - Branch prediction hints (C++20)
int classify(int x) {
    if (x > 0) [[likely]] {
        return 1;
    } else [[unlikely]] {
        return -1;
    }
}

// [[no_unique_address]] - Optimize empty member size (C++20)
struct Empty {};

struct WithoutAttribute {
    Empty e;
    int x;
};  // sizeof might be 8 (4 for int + 4 for Empty with padding)

struct WithAttribute {
    [[no_unique_address]] Empty e;
    int x;
};  // sizeof is 4 (Empty takes no space)
```

---

## 8.5 User-Defined Literals

### Built-in Literal Operators

```cpp
#include <iostream>
#include <chrono>
#include <string>
#include <complex>

int main() {
    using namespace std::literals;
    
    // String literals
    auto s1 = "hello"s;        // std::string
    auto s2 = u8"hello"s;      // std::u8string (C++20)
    
    // Chrono literals
    auto duration1 = 100ms;    // std::chrono::milliseconds
    auto duration2 = 5s;       // std::chrono::seconds
    auto duration3 = 2min;     // std::chrono::minutes
    auto duration4 = 1h;       // std::chrono::hours
    
    // Complex number literals
    auto c = 3.0 + 4.0i;       // std::complex<double>
    
    std::cout << "String: " << s1 << "\n";
    std::cout << "Duration: " << duration1.count() << "ms\n";
    std::cout << "Complex: " << c << "\n";
    
    return 0;
}
```

### Custom Literal Operators

```cpp
#include <iostream>
#include <cstddef>

// Cooked form (for numeric types)
constexpr long double operator""_kg(long double value) {
    return value;
}

constexpr long double operator""_lb(long double value) {
    return value * 0.453592;  // Convert pounds to kg
}

// Raw form (for string-like types)
constexpr size_t operator""_KB(unsigned long long value) {
    return value * 1024;
}

constexpr size_t operator""_MB(unsigned long long value) {
    return value * 1024 * 1024;
}

// String literal operator
std::string operator""_reverse(const char* str, size_t len) {
    return std::string(str, str + len).reverse();
}

int main() {
    auto weight1 = 100.0_kg;
    auto weight2 = 220.0_lb;
    
    std::cout << "100 kg = " << weight1 << " kg\n";
    std::cout << "220 lb = " << weight2 << " kg\n";
    
    auto size1 = 5_KB;
    auto size2 = 10_MB;
    
    std::cout << "5 KB = " << size1 << " bytes\n";
    std::cout << "10 MB = " << size2 << " bytes\n";
    
    return 0;
}
```

---

## 8.6 C++20 Modules (Preview)

### Traditional Header-Based Approach

```cpp
// math.h (traditional header)
#ifndef MATH_H
#define MATH_H

namespace math {
    int add(int a, int b);
    int multiply(int a, int b);
}

#endif

// math.cpp
#include "math.h"

namespace math {
    int add(int a, int b) { return a + b; }
    int multiply(int a, int b) { return a * b; }
}

// main.cpp
#include "math.h"
#include <iostream>

int main() {
    std::cout << math::add(5, 3) << "\n";
}
```

### Module-Based Approach (C++20)

```cpp
// math.ixx (module interface)
export module math;

export namespace math {
    int add(int a, int b);
    int multiply(int a, int b);
}

// math.cpp (module implementation)
module math;

namespace math {
    int add(int a, int b) { return a + b; }
    int multiply(int a, int b) { return a * b; }
}

// main.cpp (module usage)
import math;
import std;  // Import standard library

int main() {
    std::cout << math::add(5, 3) << "\n";
    return 0;
}
```

### Module Benefits

```cpp
/*
Advantages of Modules:

1. Faster Compilation:
   - Parsed once, not every translation unit
   - No need for include guards
   - Reduced preprocessing overhead

2. Better Encapsulation:
   - Only exported names are visible
   - Implementation details truly hidden
   - No macro leakage

3. Order Independence:
   - No header order issues
   - No circular dependency problems

4. Isolation:
   - Macros don't leak between modules
   - Better namespace isolation

Example compilation time improvement:
- Headers: Reparse in every file (~1s per file)
- Modules: Parse once (~1s total)
*/
```

---

## Comprehensive Example: Generic Library

```cpp
#include <iostream>
#include <vector>
#include <type_traits>
#include <optional>

// Custom type traits
template<typename T, typename = void>
struct has_print : std::false_type {};

template<typename T>
struct has_print<T, std::void_t<decltype(std::declval<T>().print())>>
    : std::true_type {};

template<typename T>
inline constexpr bool has_print_v = has_print<T>::value;

// SFINAE-based function selection
template<typename T>
std::enable_if_t<has_print_v<T>> display(const T& obj) {
    obj.print();
}

template<typename T>
std::enable_if_t<!has_print_v<T>> display(const T& obj) {
    std::cout << obj << "\n";
}

// Perfect forwarding wrapper
template<typename Func, typename... Args>
decltype(auto) measure_call(Func&& func, Args&&... args) {
    std::cout << "Calling function...\n";
    return std::forward<Func>(func)(std::forward<Args>(args)...);
}

// Custom literal
constexpr size_t operator""_percent(unsigned long long value) {
    return value;
}

// Class with print method
class Widget {
    int value;
public:
    Widget(int v) : value(v) {}
    void print() const {
        std::cout << "Widget(" << value << ")\n";
    }
};

int main() {
    // Type trait detection
    std::cout << "Widget has print: " << has_print_v<Widget> << "\n";
    std::cout << "int has print: " << has_print_v<int> << "\n";
    
    // SFINAE-based dispatch
    Widget w(42);
    display(w);   // Calls print()
    display(100); // Uses operator<<
    
    // Perfect forwarding
    auto lambda = [](int x, int y) { return x + y; };
    int result = measure_call(lambda, 5, 3);
    std::cout << "Result: " << result << "\n";
    
    // User-defined literal
    auto percent = 75_percent;
    std::cout << "Percentage: " << percent << "%\n";
    
    return 0;
}
```

---

## Conclusion

Part 8 has covered advanced C++ features:

1. **Type Deduction** - auto, decltype, decltype(auto), trailing returns
2. **Move Semantics** - Perfect forwarding, universal references, reference collapsing
3. **SFINAE** - Detection idiom, if constexpr, type traits
4. **Attributes** - noreturn, deprecated, nodiscard, likely/unlikely
5. **User-Defined Literals** - Custom operators for domain-specific types
6. **Modules** (C++20) - Modern alternative to headers

**Key Principles:**
- **Type Safety**: Compile-time type checking
- **Zero Overhead**: No runtime cost for abstractions
- **Expressiveness**: Cleaner, more readable code
- **Modularity**: Better compilation model

**Best Practices:**
- Use auto for complex types
- Mark move operations noexcept
- Use if constexpr for compile-time branches
- Apply [[nodiscard]] to important return values
- Prefer modules over headers (C++20+)

**Next in the series:** Part 9 will explore Design Patterns & Software Architecture in C++.

---

## References

- **"Effective Modern C++"** by Scott Meyers
- **"C++ Move Semantics"** by Nicolai Josuttis
- **C++20 Modules**: https://en.cppreference.com/w/cpp/language/modules
- **C++ Core Guidelines**: Modern features

---

*This blog post is part of the Complete C++ Mastery series.*
