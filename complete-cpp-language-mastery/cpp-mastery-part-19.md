---
title: "Complete C++ Mastery Part 19: Advanced Specialized Topics"
date: 2024-02-21 22:00:00 +0000
categories: [C++, Programming, Advanced]
tags: [cpp, advanced, metaprogramming, reflection, coroutines, modules, concepts]
---

# Complete C++ Mastery Part 19: Advanced Specialized Topics

## Introduction

Welcome to Part 19 of the Complete C++ Mastery series. This part explores cutting-edge and specialized C++ topics: advanced template metaprogramming, compile-time computation, reflection proposals, coroutines in depth, modules architecture, and experimental features. These topics push the boundaries of what's possible in C++.

While not needed for everyday programming, these techniques enable library development, domain-specific languages, and high-performance abstractions.

**Prerequisites:** This part assumes mastery of all previous parts, especially templates and advanced features.

---

## 19.1 Advanced Template Metaprogramming

### Compile-Time Computation

```cpp
#include <iostream>
#include <array>

// Compile-time factorial
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

// constexpr function (C++11+)
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

// Compile-time Fibonacci
template<int N>
struct Fib {
    static constexpr int value = Fib<N-1>::value + Fib<N-2>::value;
};

template<> struct Fib<0> { static constexpr int value = 0; };
template<> struct Fib<1> { static constexpr int value = 1; };

// Generate array at compile time
template<size_t... Is>
constexpr auto make_fibonacci_array(std::index_sequence<Is...>) {
    return std::array<int, sizeof...(Is)>{Fib<Is>::value...};
}

int main() {
    // Computed at compile time
    constexpr int fact_5 = Factorial<5>::value;
    constexpr int fact_10 = factorial(10);
    
    // Array computed at compile time
    constexpr auto fib_array = make_fibonacci_array(std::make_index_sequence<10>{});
    
    for (auto val : fib_array) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Type Lists and Manipulation

```cpp
#include <type_traits>
#include <tuple>

// Type list
template<typename... Ts>
struct TypeList {};

// Get size of type list
template<typename List>
struct Length;

template<typename... Ts>
struct Length<TypeList<Ts...>> {
    static constexpr size_t value = sizeof...(Ts);
};

// Get type at index
template<size_t N, typename List>
struct TypeAt;

template<size_t N, typename T, typename... Ts>
struct TypeAt<N, TypeList<T, Ts...>>
    : TypeAt<N - 1, TypeList<Ts...>> {};

template<typename T, typename... Ts>
struct TypeAt<0, TypeList<T, Ts...>> {
    using type = T;
};

// Append type to list
template<typename List, typename T>
struct Append;

template<typename... Ts, typename T>
struct Append<TypeList<Ts...>, T> {
    using type = TypeList<Ts..., T>;
};

// Transform types
template<template<typename> class F, typename List>
struct Transform;

template<template<typename> class F, typename... Ts>
struct Transform<F, TypeList<Ts...>> {
    using type = TypeList<typename F<Ts>::type...>;
};

// Example usage
int main() {
    using MyList = TypeList<int, double, char>;
    
    static_assert(Length<MyList>::value == 3);
    static_assert(std::is_same_v<TypeAt<1, MyList>::type, double>);
    
    using Extended = Append<MyList, float>::type;
    static_assert(Length<Extended>::value == 4);
    
    using Pointers = Transform<std::add_pointer, MyList>::type;
    static_assert(std::is_same_v<TypeAt<0, Pointers>::type, int*>);
    
    return 0;
}
```

### SFINAE and Expression SFINAE

```cpp
#include <iostream>
#include <type_traits>

// Detect if type has begin() method
template<typename T, typename = void>
struct has_begin : std::false_type {};

template<typename T>
struct has_begin<T, std::void_t<decltype(std::declval<T>().begin())>>
    : std::true_type {};

// Detect if type is iterable
template<typename T, typename = void>
struct is_iterable : std::false_type {};

template<typename T>
struct is_iterable<T, std::void_t<
    decltype(std::declval<T>().begin()),
    decltype(std::declval<T>().end())
>> : std::true_type {};

// Function enabled only for iterables
template<typename T>
std::enable_if_t<is_iterable<T>::value, size_t>
count_elements(const T& container) {
    return std::distance(container.begin(), container.end());
}

// C++20: Concepts version (cleaner)
#if __cplusplus >= 202002L
template<typename T>
concept Iterable = requires(T t) {
    { t.begin() } -> std::same_as<typename T::iterator>;
    { t.end() } -> std::same_as<typename T::iterator>;
};

template<Iterable T>
size_t count_elements_modern(const T& container) {
    return std::distance(container.begin(), container.end());
}
#endif

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    std::cout << "has_begin: " << has_begin<std::vector<int>>::value << "\n";
    std::cout << "is_iterable: " << is_iterable<std::vector<int>>::value << "\n";
    std::cout << "Elements: " << count_elements(vec) << "\n";
    
    return 0;
}
```

---

## 19.2 Coroutines (C++20)

### Generator Pattern

```cpp
#include <coroutine>
#include <iostream>
#include <exception>

template<typename T>
class Generator {
public:
    struct promise_type {
        T current_value;
        std::exception_ptr exception;
        
        Generator get_return_object() {
            return Generator{
                std::coroutine_handle<promise_type>::from_promise(*this)
            };
        }
        
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        
        std::suspend_always yield_value(T value) {
            current_value = value;
            return {};
        }
        
        void return_void() {}
        
        void unhandled_exception() {
            exception = std::current_exception();
        }
    };
    
    struct iterator {
        std::coroutine_handle<promise_type> handle;
        
        iterator& operator++() {
            handle.resume();
            return *this;
        }
        
        bool operator==(std::default_sentinel_t) const {
            return handle.done();
        }
        
        T operator*() const {
            return handle.promise().current_value;
        }
    };
    
    iterator begin() {
        handle.resume();
        return iterator{handle};
    }
    
    std::default_sentinel_t end() { return {}; }
    
    ~Generator() {
        if (handle) handle.destroy();
    }
    
private:
    explicit Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    std::coroutine_handle<promise_type> handle;
};

// Example: Generate Fibonacci sequence
Generator<int> fibonacci(int count) {
    int a = 0, b = 1;
    for (int i = 0; i < count; ++i) {
        co_yield a;
        auto tmp = a;
        a = b;
        b = tmp + b;
    }
}

int main() {
    for (int value : fibonacci(10)) {
        std::cout << value << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Async/Await Pattern

```cpp
#include <coroutine>
#include <future>
#include <thread>

template<typename T>
class Task {
public:
    struct promise_type {
        T value;
        std::exception_ptr exception;
        
        Task get_return_object() {
            return Task{
                std::coroutine_handle<promise_type>::from_promise(*this)
            };
        }
        
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        
        void return_value(T v) {
            value = v;
        }
        
        void unhandled_exception() {
            exception = std::current_exception();
        }
    };
    
    T get() {
        handle.resume();
        if (handle.promise().exception) {
            std::rethrow_exception(handle.promise().exception);
        }
        return handle.promise().value;
    }
    
    ~Task() {
        if (handle) handle.destroy();
    }
    
private:
    explicit Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    std::coroutine_handle<promise_type> handle;
};

// Async operation
Task<int> compute_async() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    co_return 42;
}

// Async composition
Task<int> compute_sum() {
    auto a = co_await compute_async();
    auto b = co_await compute_async();
    co_return a + b;
}
```

---

## 19.3 Modules (C++20)

### Module Basics

```cpp
// math.ixx (module interface)
export module math;

import std;

export namespace math {
    // Exported functions
    export int add(int a, int b) {
        return a + b;
    }
    
    export int multiply(int a, int b) {
        return a * b;
    }
    
    // Internal implementation (not exported)
    namespace internal {
        int helper(int x) {
            return x * 2;
        }
    }
    
    export class Calculator {
        int value = 0;
    public:
        void set(int v) { value = v; }
        int get() const { return value; }
        
        int compute(int x) {
            return internal::helper(x);
        }
    };
}

// Advanced: Module partitions
export module math:basic;

export int add(int a, int b);
export int subtract(int a, int b);

// math_advanced.ixx
export module math:advanced;

export double power(double base, int exp);
export double sqrt(double x);

// math.ixx (primary module interface)
export module math;

export import :basic;
export import :advanced;
```

### Module Usage

```cpp
// main.cpp
import math;
import std;

int main() {
    int result = math::add(5, 3);
    std::println("Result: {}", result);
    
    math::Calculator calc;
    calc.set(10);
    std::println("Calculated: {}", calc.compute(5));
    
    return 0;
}
```

### Benefits of Modules

```cpp
/*
Modules vs Headers:

HEADERS:
- Parsed every translation unit
- Include guards needed
- Macros leak
- Order-dependent
- Slow compilation

MODULES:
- Parsed once
- No guards needed
- No macro leakage
- Order-independent
- Fast compilation

Compilation time improvements:
- 10-50% faster compilation
- 90% faster incremental builds
- Better IDE performance
*/
```

---

## 19.4 Reflection (Proposed)

### Current State: Static Reflection

```cpp
// Available in C++20 via libraries (not standard)
#include <iostream>
#include <string_view>
#include <array>

// Minimal reflection using constexpr
template<typename T>
constexpr std::string_view type_name() {
    std::string_view name = __PRETTY_FUNCTION__;
    // Parse compiler-specific output
    return name;
}

// Member introspection (manual)
struct Person {
    std::string name;
    int age;
    
    // Manual reflection data
    static constexpr size_t field_count = 2;
    
    template<size_t N>
    auto& get_field() {
        if constexpr (N == 0) return name;
        else if constexpr (N == 1) return age;
    }
    
    template<size_t N>
    static constexpr std::string_view field_name() {
        if constexpr (N == 0) return "name";
        else if constexpr (N == 1) return "age";
    }
};

template<typename T, size_t... Is>
void print_fields_impl(const T& obj, std::index_sequence<Is...>) {
    ((std::cout << T::template field_name<Is>() << ": " 
                << obj.template get_field<Is>() << "\n"), ...);
}

template<typename T>
void print_fields(const T& obj) {
    print_fields_impl(obj, std::make_index_sequence<T::field_count>{});
}

int main() {
    Person p{"Alice", 30};
    print_fields(p);
    
    return 0;
}
```

### Future: Static Reflection Proposal

```cpp
// C++26 or later (proposed)
#include <meta>

// Automatic reflection
template<typename T>
void print_object(const T& obj) {
    constexpr auto members = std::meta::members_of(^T);
    
    [: members :] >> [&]<auto member> {
        std::cout << std::meta::name_of(member) << ": "
                  << obj.[:member:] << "\n";
    };
}

// Automatic serialization
template<typename T>
std::string to_json(const T& obj) {
    std::string result = "{";
    constexpr auto members = std::meta::members_of(^T);
    
    bool first = true;
    [: members :] >> [&]<auto member> {
        if (!first) result += ",";
        result += "\"" + std::string(std::meta::name_of(member)) + "\":";
        result += to_json_value(obj.[:member:]);
        first = false;
    };
    
    result += "}";
    return result;
}
```

---

## 19.5 Compile-Time Regex

```cpp
#include <iostream>
#include <string_view>
#include <array>

// Compile-time string
template<size_t N>
struct CompileString {
    char data[N];
    
    constexpr CompileString(const char (&str)[N]) {
        for (size_t i = 0; i < N; ++i) {
            data[i] = str[i];
        }
    }
    
    constexpr size_t size() const { return N - 1; }
};

// Simple compile-time regex matcher
template<CompileString Pattern>
constexpr bool matches(std::string_view str) {
    // Simplified: just checks if string starts with pattern
    if (str.size() < Pattern.size()) return false;
    
    for (size_t i = 0; i < Pattern.size(); ++i) {
        if (str[i] != Pattern.data[i]) return false;
    }
    
    return true;
}

int main() {
    constexpr bool match1 = matches<"hello">("hello world");  // true
    constexpr bool match2 = matches<"hello">("hi there");     // false
    
    std::cout << "Match 1: " << match1 << "\n";
    std::cout << "Match 2: " << match2 << "\n";
    
    return 0;
}
```

---

## 19.6 Expression Templates

```cpp
#include <iostream>
#include <vector>

// Lazy evaluation with expression templates
template<typename E>
class VecExpression {
    const E& expr;
public:
    VecExpression(const E& e) : expr(e) {}
    
    auto operator[](size_t i) const {
        return expr[i];
    }
    
    size_t size() const { return expr.size(); }
};

// Concrete vector
class Vec {
    std::vector<double> data;
public:
    Vec(size_t n) : data(n) {}
    Vec(std::initializer_list<double> init) : data(init) {}
    
    double operator[](size_t i) const { return data[i]; }
    double& operator[](size_t i) { return data[i]; }
    size_t size() const { return data.size(); }
    
    // Assignment from expression
    template<typename E>
    Vec& operator=(const VecExpression<E>& expr) {
        for (size_t i = 0; i < size(); ++i) {
            data[i] = expr[i];
        }
        return *this;
    }
};

// Addition expression
template<typename L, typename R>
class VecAdd {
    const L& left;
    const R& right;
public:
    VecAdd(const L& l, const R& r) : left(l), right(r) {}
    
    auto operator[](size_t i) const {
        return left[i] + right[i];
    }
    
    size_t size() const { return left.size(); }
};

// Operator overloads
template<typename L, typename R>
VecExpression<VecAdd<L, R>> operator+(const L& l, const R& r) {
    return VecExpression<VecAdd<L, R>>(VecAdd<L, R>(l, r));
}

int main() {
    Vec a = {1, 2, 3};
    Vec b = {4, 5, 6};
    Vec c = {7, 8, 9};
    Vec result(3);
    
    // Single loop, no temporaries!
    result = a + b + c;
    
    for (size_t i = 0; i < result.size(); ++i) {
        std::cout << result[i] << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

---

## 19.7 Constexpr Everything (C++20)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// Constexpr vector operations (C++20)
constexpr int compute_at_compile_time() {
    std::vector<int> vec = {5, 2, 8, 1, 9};
    
    std::sort(vec.begin(), vec.end());
    
    int sum = 0;
    for (int val : vec) {
        sum += val;
    }
    
    return sum;
}

// Constexpr virtual functions (C++20)
struct Base {
    constexpr virtual int compute() const { return 1; }
    constexpr virtual ~Base() = default;
};

struct Derived : Base {
    constexpr int compute() const override { return 2; }
};

constexpr int test_virtual() {
    Derived d;
    const Base* b = &d;
    return b->compute();
}

int main() {
    constexpr int result = compute_at_compile_time();
    constexpr int virt = test_virtual();
    
    std::cout << "Computed at compile time: " << result << "\n";
    std::cout << "Virtual call result: " << virt << "\n";
    
    return 0;
}
```

---

## Conclusion

Part 19 explored advanced C++ topics:

1. **Advanced Metaprogramming** - Type lists, compile-time computation
2. **Coroutines** - Generators, async/await
3. **Modules** - Modern code organization
4. **Reflection** - Current state and future proposals
5. **Compile-Time Regex** - Constexpr string processing
6. **Expression Templates** - Lazy evaluation
7. **Constexpr Everything** - C++20 constexpr expansion

**Key Insights:**
- Metaprogramming enables zero-overhead abstractions
- Coroutines simplify async code
- Modules revolutionize compilation
- Reflection coming in future standards
- Constexpr keeps expanding

**When to Use:**
- Library development
- Performance-critical code
- Domain-specific languages
- Generic programming

**Advanced Features:**
- Not needed for everyday code
- Essential for library authors
- Enable elegant APIs
- Compiler-assisted optimization

**Next:** Part 20 covers FAQ & Troubleshooting.

---

*This blog post is part of the Complete C++ Mastery series.*
