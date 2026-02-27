---
title: "Complete C++ Mastery Part 4: Templates & Generic Programming"
date: 2024-02-20 16:00:00 +0000
categories: [C++, Programming, Templates]
tags: [cpp, templates, generic-programming, metaprogramming, concepts, variadic-templates, sfinae]
---

# Complete C++ Mastery Part 4: Templates & Generic Programming

## Introduction

Welcome to Part 4 of the Complete C++ Mastery series. Templates are one of C++'s most powerful features, enabling generic programming—the ability to write code that works with any type while maintaining type safety and performance. Unlike runtime polymorphism (virtual functions), templates provide compile-time polymorphism with zero runtime overhead.

This part explores the full spectrum of template programming: from basic function and class templates, through advanced metaprogramming techniques, to modern C++20 concepts. We'll understand how templates work under the hood, master SFINAE, and learn to write generic, reusable code that rivals the STL in quality.

**Prerequisites:** This part assumes knowledge from Parts 1-3, including functions, classes, pointers, references, and the compilation process.

---

## 4.1 Function Templates

### Basic Function Template Syntax

Function templates define a family of functions parameterized by types.

```cpp
#include <iostream>

// Function template
template<typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

// Alternative: using 'class' keyword (identical to 'typename')
template<class T>
T min(T a, T b) {
    return (a < b) ? a : b;
}

int main() {
    // Template instantiation with int
    std::cout << "max(10, 20) = " << max(10, 20) << "\n";
    
    // Template instantiation with double
    std::cout << "max(3.14, 2.71) = " << max(3.14, 2.71) << "\n";
    
    // Template instantiation with string
    std::cout << "max(\"abc\", \"def\") = " << max("abc", "def") << "\n";
    
    return 0;
}
```

**How Templates Work:**
```cpp
// When you write:
max(10, 20);

// Compiler generates (instantiates):
int max(int a, int b) {
    return (a > b) ? a : b;
}

// For each unique type, a separate function is instantiated
max(3.14, 2.71);  // Generates: double max(double, double)
```

### Template Type Deduction

The compiler deduces template arguments from function call arguments:

```cpp
#include <iostream>

template<typename T>
void print(T value) {
    std::cout << "Value: " << value << ", Type size: " << sizeof(T) << "\n";
}

int main() {
    print(42);        // T deduced as int
    print(3.14);      // T deduced as double
    print("hello");   // T deduced as const char*
    
    // Explicit template argument
    print<long>(42);  // T explicitly specified as long
    
    return 0;
}
```

**Type Deduction Rules:**
```cpp
template<typename T>
void func1(T param);

template<typename T>
void func2(T& param);

template<typename T>
void func3(const T& param);

int x = 42;
const int cx = x;
const int& rx = x;

// func1: pass by value (copies, drops const and references)
func1(x);   // T = int
func1(cx);  // T = int (const dropped)
func1(rx);  // T = int (reference and const dropped)

// func2: pass by reference
func2(x);   // T = int, param type is int&
func2(cx);  // T = const int, param type is const int&
func2(rx);  // T = const int, param type is const int&

// func3: pass by const reference
func3(x);   // T = int, param type is const int&
func3(cx);  // T = int, param type is const int&
func3(rx);  // T = int, param type is const int&
```

### Multiple Template Parameters

```cpp
#include <iostream>

template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {  // Trailing return type
    return a + b;
}

// C++14: auto return type deduction
template<typename T, typename U>
auto multiply(T a, U b) {
    return a * b;
}

int main() {
    std::cout << add(5, 3.14) << "\n";       // int + double = double
    std::cout << multiply(2, 3.5) << "\n";   // int * double = double
    
    return 0;
}
```

### Explicit Template Arguments

```cpp
#include <iostream>
#include <vector>

template<typename T>
T create_default() {
    return T();  // Default construction
}

template<typename Container, typename Value>
void fill_container(Container& c, const Value& value, size_t count) {
    for (size_t i = 0; i < count; ++i) {
        c.push_back(value);
    }
}

int main() {
    // Must specify template argument (no deduction possible)
    int x = create_default<int>();
    std::string s = create_default<std::string>();
    
    std::cout << "x = " << x << "\n";
    std::cout << "s = \"" << s << "\"\n";
    
    // Partial explicit specification
    std::vector<int> vec;
    fill_container<std::vector<int>>(vec, 42, 5);  // Can deduce Value
    
    for (int val : vec) {
        std::cout << val << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Template Specialization

**Full Specialization:**
```cpp
#include <iostream>
#include <cstring>

// Primary template
template<typename T>
bool is_equal(T a, T b) {
    return a == b;
}

// Full specialization for const char*
template<>
bool is_equal<const char*>(const char* a, const char* b) {
    return std::strcmp(a, b) == 0;
}

int main() {
    std::cout << is_equal(10, 10) << "\n";          // Uses primary template
    std::cout << is_equal(3.14, 3.14) << "\n";      // Uses primary template
    std::cout << is_equal("hello", "hello") << "\n"; // Uses specialization
    
    return 0;
}
```

### Function Template Overloading

```cpp
#include <iostream>

// Template version
template<typename T>
void print(T value) {
    std::cout << "Template: " << value << "\n";
}

// Overload for pointer
template<typename T>
void print(T* value) {
    std::cout << "Template (pointer): " << *value << "\n";
}

// Non-template overload (preferred if matches)
void print(int value) {
    std::cout << "Non-template: " << value << "\n";
}

int main() {
    int x = 42;
    
    print(x);      // Calls non-template (exact match preferred)
    print(3.14);   // Calls template<double>
    print(&x);     // Calls template<int*> (pointer overload)
    
    return 0;
}
```

### SFINAE (Substitution Failure Is Not An Error)

SFINAE is the principle that allows template substitution failures without causing compilation errors.

```cpp
#include <iostream>
#include <type_traits>

// Only enabled for integral types
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T value) {
    std::cout << "Processing integral: " << value << "\n";
    return value * 2;
}

// Only enabled for floating-point types
template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
process(T value) {
    std::cout << "Processing floating-point: " << value << "\n";
    return value * 1.5;
}

int main() {
    std::cout << process(42) << "\n";      // Calls integral version
    std::cout << process(3.14) << "\n";    // Calls floating-point version
    // process("hello");  // ❌ Error: no matching function
    
    return 0;
}
```

**Modern Alternative (C++17): if constexpr**
```cpp
#include <iostream>
#include <type_traits>

template<typename T>
T process(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "Processing integral: " << value << "\n";
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "Processing floating-point: " << value << "\n";
        return value * 1.5;
    } else {
        std::cout << "Processing other type\n";
        return value;
    }
}

int main() {
    std::cout << process(42) << "\n";
    std::cout << process(3.14) << "\n";
    
    return 0;
}
```

---

## 4.2 Class Templates

### Basic Class Template

```cpp
#include <iostream>

template<typename T>
class Box {
    T value;
    
public:
    Box(T v) : value(v) {}
    
    T get() const { return value; }
    void set(T v) { value = v; }
    
    void print() const {
        std::cout << "Box contains: " << value << "\n";
    }
};

int main() {
    Box<int> int_box(42);
    int_box.print();
    
    Box<std::string> str_box("Hello");
    str_box.print();
    
    Box<double> double_box(3.14);
    double_box.print();
    
    return 0;
}
```

### Template Parameters

**Type Parameters:**
```cpp
template<typename T, typename U>
class Pair {
    T first;
    U second;
    
public:
    Pair(T f, U s) : first(f), second(s) {}
    
    T get_first() const { return first; }
    U get_second() const { return second; }
};
```

**Non-Type Parameters:**
```cpp
#include <iostream>
#include <array>

// Non-type template parameter (compile-time constant)
template<typename T, size_t N>
class Array {
    T data[N];
    
public:
    constexpr size_t size() const { return N; }
    
    T& operator[](size_t index) {
        return data[index];
    }
    
    const T& operator[](size_t index) const {
        return data[index];
    }
};

int main() {
    Array<int, 5> arr;
    
    for (size_t i = 0; i < arr.size(); ++i) {
        arr[i] = i * 10;
    }
    
    for (size_t i = 0; i < arr.size(); ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

**Template Template Parameters:**
```cpp
#include <iostream>
#include <vector>
#include <list>

// Template template parameter: Container must itself be a template
template<typename T, template<typename, typename> class Container>
class Stack {
    Container<T, std::allocator<T>> data;
    
public:
    void push(const T& value) {
        data.push_back(value);
    }
    
    T pop() {
        T value = data.back();
        data.pop_back();
        return value;
    }
    
    bool empty() const {
        return data.empty();
    }
};

int main() {
    Stack<int, std::vector> vec_stack;
    vec_stack.push(10);
    vec_stack.push(20);
    
    Stack<int, std::list> list_stack;
    list_stack.push(30);
    list_stack.push(40);
    
    std::cout << "vec_stack: " << vec_stack.pop() << " " << vec_stack.pop() << "\n";
    std::cout << "list_stack: " << list_stack.pop() << " " << list_stack.pop() << "\n";
    
    return 0;
}
```

### Member Templates

```cpp
#include <iostream>

template<typename T>
class Container {
    T value;
    
public:
    Container(T v) : value(v) {}
    
    // Member template (templated member function)
    template<typename U>
    Container(const Container<U>& other) : value(static_cast<T>(other.get())) {
        std::cout << "Converting constructor\n";
    }
    
    T get() const { return value; }
    
    // Member template function
    template<typename U>
    void set_from(const Container<U>& other) {
        value = static_cast<T>(other.get());
    }
};

int main() {
    Container<int> int_container(42);
    
    // Uses member template constructor
    Container<double> double_container(int_container);
    
    std::cout << "double_container: " << double_container.get() << "\n";
    
    return 0;
}
```

### Class Template Specialization

**Full Specialization:**
```cpp
#include <iostream>

// Primary template
template<typename T>
class Printer {
public:
    void print(const T& value) {
        std::cout << "Generic: " << value << "\n";
    }
};

// Full specialization for bool
template<>
class Printer<bool> {
public:
    void print(const bool& value) {
        std::cout << "Bool: " << (value ? "true" : "false") << "\n";
    }
};

// Full specialization for const char*
template<>
class Printer<const char*> {
public:
    void print(const char* const& value) {
        std::cout << "String: \"" << value << "\"\n";
    }
};

int main() {
    Printer<int> int_printer;
    int_printer.print(42);
    
    Printer<bool> bool_printer;
    bool_printer.print(true);
    
    Printer<const char*> str_printer;
    str_printer.print("hello");
    
    return 0;
}
```

**Partial Specialization:**
```cpp
#include <iostream>

// Primary template
template<typename T, typename U>
class Pair {
public:
    void print() {
        std::cout << "Generic Pair\n";
    }
};

// Partial specialization: both types are the same
template<typename T>
class Pair<T, T> {
public:
    void print() {
        std::cout << "Pair with same types\n";
    }
};

// Partial specialization: second type is pointer
template<typename T, typename U>
class Pair<T, U*> {
public:
    void print() {
        std::cout << "Pair with pointer as second type\n";
    }
};

int main() {
    Pair<int, double> p1;
    p1.print();  // Generic Pair
    
    Pair<int, int> p2;
    p2.print();  // Pair with same types
    
    Pair<int, double*> p3;
    p3.print();  // Pair with pointer as second type
    
    return 0;
}
```

### Class Template Argument Deduction (CTAD) - C++17

```cpp
#include <iostream>
#include <vector>

template<typename T>
class Container {
    T value;
    
public:
    Container(T v) : value(v) {}
    
    T get() const { return value; }
};

// Deduction guide
template<typename T>
Container(T) -> Container<T>;

int main() {
    // C++17: Template arguments deduced from constructor arguments
    Container c1(42);        // Container<int>
    Container c2(3.14);      // Container<double>
    Container c3("hello");   // Container<const char*>
    
    std::cout << c1.get() << "\n";
    std::cout << c2.get() << "\n";
    std::cout << c3.get() << "\n";
    
    // STL containers also support CTAD
    std::vector v{1, 2, 3, 4, 5};  // std::vector<int>
    
    return 0;
}
```

### Default Template Arguments

```cpp
#include <iostream>
#include <memory>

template<typename T, typename Allocator = std::allocator<T>>
class Vector {
    T* data;
    size_t size;
    Allocator alloc;
    
public:
    Vector(size_t n = 0) : size(n), data(alloc.allocate(n)) {}
    
    ~Vector() {
        alloc.deallocate(data, size);
    }
    
    T& operator[](size_t index) { return data[index]; }
};

int main() {
    Vector<int> v1(10);           // Uses default allocator
    Vector<int, std::allocator<int>> v2(10);  // Explicit allocator
    
    return 0;
}
```

---

## 4.3 Template Metaprogramming

### Compile-Time Computation

Template metaprogramming performs computations at compile time.

**Factorial Example:**
```cpp
#include <iostream>

// Template metaprogram for factorial
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

// Base case (specialization)
template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

int main() {
    // Computed at compile time
    constexpr int fact5 = Factorial<5>::value;
    std::cout << "5! = " << fact5 << "\n";  // 120
    
    // Can use in array size (proves it's compile-time)
    int array[Factorial<4>::value];  // Array of size 24
    std::cout << "Array size: " << sizeof(array) / sizeof(int) << "\n";
    
    return 0;
}
```

**Fibonacci Example:**
```cpp
#include <iostream>

template<int N>
struct Fibonacci {
    static constexpr int value = Fibonacci<N - 1>::value + Fibonacci<N - 2>::value;
};

template<>
struct Fibonacci<0> {
    static constexpr int value = 0;
};

template<>
struct Fibonacci<1> {
    static constexpr int value = 1;
};

int main() {
    std::cout << "Fib(10) = " << Fibonacci<10>::value << "\n";  // 55
    
    return 0;
}
```

### Type Traits and Type Manipulation

**Standard Type Traits:**
```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void check_type() {
    std::cout << "Type properties:\n";
    std::cout << "  is_integral: " << std::is_integral<T>::value << "\n";
    std::cout << "  is_floating_point: " << std::is_floating_point<T>::value << "\n";
    std::cout << "  is_pointer: " << std::is_pointer<T>::value << "\n";
    std::cout << "  is_const: " << std::is_const<T>::value << "\n";
    std::cout << "  is_reference: " << std::is_reference<T>::value << "\n";
}

int main() {
    check_type<int>();
    std::cout << "\n";
    check_type<const double*>();
    
    return 0;
}
```

**Type Transformations:**
```cpp
#include <iostream>
#include <type_traits>

int main() {
    // Remove const
    using T1 = const int;
    using T2 = std::remove_const<T1>::type;  // int
    
    // Add pointer
    using T3 = std::add_pointer<int>::type;  // int*
    
    // Remove reference
    using T4 = std::remove_reference<int&>::type;  // int
    
    // Check results
    std::cout << std::is_const<T2>::value << "\n";  // false
    std::cout << std::is_pointer<T3>::value << "\n";  // true
    std::cout << std::is_reference<T4>::value << "\n";  // false
    
    return 0;
}
```

**Custom Type Traits:**
```cpp
#include <iostream>

// Custom type trait to check if type has a nested 'value_type'
template<typename T, typename = void>
struct has_value_type : std::false_type {};

template<typename T>
struct has_value_type<T, std::void_t<typename T::value_type>> : std::true_type {};

// Usage
#include <vector>
#include <string>

int main() {
    std::cout << has_value_type<std::vector<int>>::value << "\n";  // true
    std::cout << has_value_type<int>::value << "\n";  // false
    std::cout << has_value_type<std::string>::value << "\n";  // true
    
    return 0;
}
```

### Tag Dispatching

Use types as tags to select different implementations at compile time.

```cpp
#include <iostream>
#include <iterator>

// Tags for different iterator categories
struct input_iterator_tag {};
struct random_access_iterator_tag : input_iterator_tag {};

// Implementation for input iterators (slow)
template<typename Iterator>
void advance_impl(Iterator& it, int n, input_iterator_tag) {
    std::cout << "Using slow advance (input iterator)\n";
    while (n--) {
        ++it;
    }
}

// Implementation for random access iterators (fast)
template<typename Iterator>
void advance_impl(Iterator& it, int n, random_access_iterator_tag) {
    std::cout << "Using fast advance (random access iterator)\n";
    it += n;
}

// Dispatch function
template<typename Iterator>
void advance(Iterator& it, int n) {
    advance_impl(it, n, typename std::iterator_traits<Iterator>::iterator_category{});
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    auto it = vec.begin();
    advance(it, 3);  // Uses random access version
    std::cout << "Value: " << *it << "\n";
    
    return 0;
}
```

### SFINAE and enable_if

```cpp
#include <iostream>
#include <type_traits>

// Only enabled for arithmetic types
template<typename T>
typename std::enable_if<std::is_arithmetic<T>::value, T>::type
square(T value) {
    return value * value;
}

// C++14 shorthand
template<typename T>
std::enable_if_t<std::is_arithmetic<T>, T>
cube(T value) {
    return value * value * value;
}

int main() {
    std::cout << square(5) << "\n";      // OK: int
    std::cout << square(3.14) << "\n";   // OK: double
    // square("hello");  // ❌ Error: not arithmetic
    
    std::cout << cube(2) << "\n";
    std::cout << cube(2.5) << "\n";
    
    return 0;
}
```

### constexpr if (C++17)

Simplifies many metaprogramming tasks:

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
auto get_value(T t) {
    if constexpr (std::is_pointer_v<T>) {
        return *t;  // Dereference if pointer
    } else {
        return t;   // Return as-is
    }
}

template<typename T>
void print_info(T value) {
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
    
    print_info(10);
    print_info(3.14);
    print_info(p);
    
    return 0;
}
```

---

## 4.4 Advanced Template Techniques

### Perfect Forwarding

Perfect forwarding preserves the value category (lvalue/rvalue) of arguments.

```cpp
#include <iostream>
#include <utility>

void process(int& x) {
    std::cout << "Lvalue reference: " << x << "\n";
}

void process(int&& x) {
    std::cout << "Rvalue reference: " << x << "\n";
}

// Perfect forwarding wrapper
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}

int main() {
    int x = 42;
    
    wrapper(x);   // Forwards as lvalue
    wrapper(10);  // Forwards as rvalue
    
    return 0;
}
```

**Universal/Forwarding References:**
```cpp
#include <iostream>

template<typename T>
void func(T&& param) {  // Universal reference (not rvalue reference!)
    // T&& is universal reference in template context
}

int main() {
    int x = 42;
    
    func(x);   // T deduced as int&, param is int&
    func(10);  // T deduced as int, param is int&&
    
    return 0;
}
```

### Expression Templates

Delay computation to optimize complex expressions:

```cpp
#include <iostream>
#include <vector>

// Simple vector expression template
template<typename E>
class VecExpression {
public:
    double operator[](size_t i) const {
        return static_cast<const E&>(*this)[i];
    }
    
    size_t size() const {
        return static_cast<const E&>(*this).size();
    }
};

class Vec : public VecExpression<Vec> {
    std::vector<double> data;
    
public:
    Vec(size_t n) : data(n) {}
    
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

// Expression template for addition
template<typename E1, typename E2>
class VecSum : public VecExpression<VecSum<E1, E2>> {
    const E1& u;
    const E2& v;
    
public:
    VecSum(const E1& u, const E2& v) : u(u), v(v) {}
    
    double operator[](size_t i) const {
        return u[i] + v[i];
    }
    
    size_t size() const { return u.size(); }
};

// Operator overload
template<typename E1, typename E2>
VecSum<E1, E2> operator+(const VecExpression<E1>& u, const VecExpression<E2>& v) {
    return VecSum<E1, E2>(static_cast<const E1&>(u), static_cast<const E2&>(v));
}

int main() {
    Vec a(3), b(3), c(3);
    
    for (size_t i = 0; i < 3; ++i) {
        a[i] = i;
        b[i] = i * 2;
    }
    
    // Expression template: no temporary vectors created!
    c = a + b + a;  // Computed in single loop
    
    for (size_t i = 0; i < 3; ++i) {
        std::cout << c[i] << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Policy-Based Design

```cpp
#include <iostream>

// Policies
template<typename T>
class SingleThreaded {
public:
    void lock() {}
    void unlock() {}
};

template<typename T>
class MultiThreaded {
    std::mutex mtx;
public:
    void lock() { mtx.lock(); }
    void unlock() { mtx.unlock(); }
};

// Class using policy
template<typename T, template<typename> class ThreadingPolicy = SingleThreaded>
class Container : private ThreadingPolicy<Container<T>> {
    std::vector<T> data;
    
public:
    void add(const T& value) {
        this->lock();
        data.push_back(value);
        this->unlock();
    }
    
    size_t size() const {
        return data.size();
    }
};

int main() {
    Container<int, SingleThreaded> st_container;
    st_container.add(42);
    
    Container<int, MultiThreaded> mt_container;
    mt_container.add(100);
    
    std::cout << "Single-threaded: " << st_container.size() << "\n";
    std::cout << "Multi-threaded: " << mt_container.size() << "\n";
    
    return 0;
}
```

### Curiously Recurring Template Pattern (CRTP)

```cpp
#include <iostream>

// CRTP base class
template<typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
    
    void common_function() {
        std::cout << "Common function\n";
    }
};

class Derived1 : public Base<Derived1> {
public:
    void implementation() {
        std::cout << "Derived1 implementation\n";
    }
};

class Derived2 : public Base<Derived2> {
public:
    void implementation() {
        std::cout << "Derived2 implementation\n";
    }
};

// Static polymorphism (no virtual functions)
template<typename T>
void use_base(Base<T>& obj) {
    obj.interface();
    obj.common_function();
}

int main() {
    Derived1 d1;
    Derived2 d2;
    
    use_base(d1);
    use_base(d2);
    
    return 0;
}
```

[Due to length constraints, this is approximately 60% of Part 4. Would you like me to continue with sections 4.5 (C++20 Concepts) and 4.6 (Variadic Templates)?]

---

## 4.5 C++20 Concepts

### Introduction to Concepts

Concepts provide compile-time constraints on template parameters, making template code more readable and producing better error messages.

**Before Concepts (SFINAE):**
```cpp
// C++17 and earlier: Complex SFINAE
template<typename T>
std::enable_if_t<std::is_integral_v<T>, T>
increment(T value) {
    return value + 1;
}
```

**With Concepts (C++20):**
```cpp
// C++20: Clear and readable
template<std::integral T>
T increment(T value) {
    return value + 1;
}

// Or using requires clause
template<typename T>
requires std::integral<T>
T decrement(T value) {
    return value - 1;
}
```

### Defining Concepts

```cpp
#include <concepts>
#include <iostream>

// Custom concept: type must be comparable
template<typename T>
concept Comparable = requires(T a, T b) {
    { a < b } -> std::convertible_to<bool>;
    { a > b } -> std::convertible_to<bool>;
    { a == b } -> std::convertible_to<bool>;
};

// Function using custom concept
template<Comparable T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << max(10, 20) << "\n";
    std::cout << max(3.14, 2.71) << "\n";
    
    return 0;
}
```

### Standard Concepts

C++20 provides many standard concepts in `<concepts>`:

```cpp
#include <concepts>
#include <iostream>

// Using standard concepts
template<std::integral T>
T square(T value) {
    return value * value;
}

template<std::floating_point T>
T half(T value) {
    return value / 2;
}

template<typename T>
requires std::copyable<T> && std::default_initializable<T>
class Container {
    T data;
public:
    Container() = default;
    Container(const T& d) : data(d) {}
};

int main() {
    std::cout << square(5) << "\n";
    std::cout << half(3.14) << "\n";
    
    Container<int> c1;
    Container<std::string> c2("hello");
    
    return 0;
}
```

**Common Standard Concepts:**
```cpp
// Type categories
std::integral<T>
std::floating_point<T>
std::signed_integral<T>
std::unsigned_integral<T>

// Object concepts
std::movable<T>
std::copyable<T>
std::semiregular<T>
std::regular<T>

// Callable concepts
std::invocable<F, Args...>
std::predicate<F, Args...>

// Comparison concepts
std::equality_comparable<T>
std::totally_ordered<T>

// Iterator concepts
std::input_iterator<I>
std::forward_iterator<I>
std::random_access_iterator<I>
```

### Requires Clauses and Expressions

```cpp
#include <concepts>
#include <iostream>

// Requires clause
template<typename T>
requires std::integral<T>
T add(T a, T b) {
    return a + b;
}

// Requires expression with multiple requirements
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;  // a + b must return T
    { a - b } -> std::same_as<T>;
};

// Using custom concept
template<Addable T>
T subtract(T a, T b) {
    return a - b;
}

// Complex requirements
template<typename T>
concept Container = requires(T c) {
    typename T::value_type;           // Must have nested type
    typename T::iterator;
    { c.begin() } -> std::same_as<typename T::iterator>;
    { c.end() } -> std::same_as<typename T::iterator>;
    { c.size() } -> std::convertible_to<std::size_t>;
};

template<Container C>
void print_container(const C& container) {
    for (const auto& elem : container) {
        std::cout << elem << " ";
    }
    std::cout << "\n";
}

int main() {
    std::cout << add(5, 3) << "\n";
    std::cout << subtract(10, 4) << "\n";
    
    std::vector<int> vec = {1, 2, 3, 4, 5};
    print_container(vec);
    
    return 0;
}
```

### Concept Composition

```cpp
#include <concepts>
#include <iostream>

// Compose concepts with && and ||
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<typename T>
concept SignedNumeric = Numeric<T> && std::signed_integral<T>;

// Using composed concepts
template<Numeric T>
T multiply(T a, T b) {
    return a * b;
}

template<SignedNumeric T>
T absolute(T value) {
    return (value < 0) ? -value : value;
}

int main() {
    std::cout << multiply(5, 3) << "\n";
    std::cout << multiply(2.5, 4.0) << "\n";
    
    std::cout << absolute(-10) << "\n";
    
    return 0;
}
```

### Abbreviated Function Templates (C++20)

```cpp
#include <concepts>
#include <iostream>

// Abbreviated function template (auto parameters)
auto add(auto a, auto b) {
    return a + b;
}

// With concepts
auto multiply(std::integral auto a, std::integral auto b) {
    return a * b;
}

// Multiple constraints
void process(std::integral auto x, std::floating_point auto y) {
    std::cout << "Integer: " << x << ", Float: " << y << "\n";
}

int main() {
    std::cout << add(5, 3) << "\n";
    std::cout << add(2.5, 1.5) << "\n";
    
    std::cout << multiply(4, 5) << "\n";
    
    process(10, 3.14);
    
    return 0;
}
```

### Subsumption

Concepts can be ordered by subsumption (one concept is more specific than another):

```cpp
#include <concepts>
#include <iostream>

template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<typename T>
concept Integral = std::integral<T>;

// More specific overload (subsumed by Integral)
void process(Integral auto value) {
    std::cout << "Processing integral: " << value << "\n";
}

// Less specific overload
void process(Numeric auto value) {
    std::cout << "Processing numeric: " << value << "\n";
}

int main() {
    process(42);     // Calls Integral version (more specific)
    process(3.14);   // Calls Numeric version
    
    return 0;
}
```

---

## 4.6 Variadic Templates and Parameter Packs

### Basic Variadic Templates

Variadic templates accept variable number of template arguments.

```cpp
#include <iostream>

// Base case (no arguments)
void print() {
    std::cout << "\n";
}

// Recursive case
template<typename T, typename... Args>
void print(T first, Args... rest) {
    std::cout << first << " ";
    print(rest...);  // Recursive call with remaining arguments
}

int main() {
    print(1, 2, 3, 4, 5);
    print("Hello", "World", 42, 3.14);
    
    return 0;
}
```

### Parameter Pack Expansion

```cpp
#include <iostream>

// Sizeof... operator: number of arguments
template<typename... Args>
void count_args(Args... args) {
    std::cout << "Number of arguments: " << sizeof...(Args) << "\n";
}

// Expanding parameter pack in function call
template<typename... Args>
void forward_to_print(Args... args) {
    print(args...);  // Expands to: print(arg1, arg2, arg3, ...)
}

int main() {
    count_args(1, 2, 3);           // 3
    count_args("a", "b", "c", "d"); // 4
    
    forward_to_print(10, 20, 30);
    
    return 0;
}
```

### Fold Expressions (C++17)

Fold expressions simplify variadic template operations:

```cpp
#include <iostream>

// Sum using fold expression
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // Unary right fold: (arg1 + (arg2 + (arg3 + ...)))
}

// Product
template<typename... Args>
auto product(Args... args) {
    return (args * ...);
}

// Logical AND
template<typename... Args>
bool all_true(Args... args) {
    return (args && ...);
}

// Print with fold
template<typename... Args>
void print_fold(Args... args) {
    (std::cout << ... << args) << "\n";  // Left fold
}

int main() {
    std::cout << "Sum: " << sum(1, 2, 3, 4, 5) << "\n";        // 15
    std::cout << "Product: " << product(2, 3, 4) << "\n";      // 24
    std::cout << "All true: " << all_true(true, true, false) << "\n"; // 0
    
    print_fold("Hello", " ", "World", " ", 42);
    
    return 0;
}
```

**Fold Expression Types:**
```cpp
// Unary right fold: (E op ...)
(args + ...)  // arg1 + (arg2 + (arg3 + ...))

// Unary left fold: (... op E)
(... + args)  // ((... + arg1) + arg2) + arg3

// Binary right fold: (E op ... op I)
(args + ... + 0)  // arg1 + (arg2 + (arg3 + 0))

// Binary left fold: (I op ... op E)
(0 + ... + args)  // ((0 + arg1) + arg2) + arg3
```

### Variadic Class Templates

```cpp
#include <iostream>

// Tuple-like class
template<typename... Types>
class Tuple;

// Base case: empty tuple
template<>
class Tuple<> {};

// Recursive case
template<typename Head, typename... Tail>
class Tuple<Head, Tail...> : private Tuple<Tail...> {
    Head value;
    
public:
    Tuple(Head h, Tail... t) : Tuple<Tail...>(t...), value(h) {}
    
    Head get_head() const { return value; }
    
    Tuple<Tail...>& get_tail() {
        return *this;
    }
};

int main() {
    Tuple<int, double, std::string> t(42, 3.14, "hello");
    
    std::cout << t.get_head() << "\n";  // 42
    
    return 0;
}
```

### Perfect Forwarding with Variadic Templates

```cpp
#include <iostream>
#include <utility>
#include <memory>

// Factory function using perfect forwarding
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

class Widget {
    int x, y;
    
public:
    Widget(int x, int y) : x(x), y(y) {
        std::cout << "Widget(" << x << ", " << y << ")\n";
    }
};

// Variadic template with perfect forwarding
template<typename Func, typename... Args>
auto call_function(Func&& func, Args&&... args) {
    std::cout << "Calling function with " << sizeof...(Args) << " arguments\n";
    return std::forward<Func>(func)(std::forward<Args>(args)...);
}

int main() {
    auto widget = make_unique<Widget>(10, 20);
    
    auto lambda = [](int a, int b, int c) {
        return a + b + c;
    };
    
    int result = call_function(lambda, 1, 2, 3);
    std::cout << "Result: " << result << "\n";
    
    return 0;
}
```

### Variadic Template Examples

**Type-Safe printf:**
```cpp
#include <iostream>

void printf_impl(const char* format) {
    std::cout << format;
}

template<typename T, typename... Args>
void printf_impl(const char* format, T value, Args... args) {
    while (*format) {
        if (*format == '%') {
            std::cout << value;
            printf_impl(format + 1, args...);
            return;
        }
        std::cout << *format++;
    }
}

template<typename... Args>
void printf(const char* format, Args... args) {
    printf_impl(format, args...);
    std::cout << "\n";
}

int main() {
    printf("Hello %!", "World");
    printf("The answer is %", 42);
    printf("% + % = %", 2, 3, 5);
    
    return 0;
}
```

**Compile-Time Type List:**
```cpp
#include <iostream>
#include <type_traits>

// Type list
template<typename... Types>
struct TypeList {};

// Check if type is in list
template<typename T, typename List>
struct Contains;

template<typename T>
struct Contains<T, TypeList<>> : std::false_type {};

template<typename T, typename Head, typename... Tail>
struct Contains<T, TypeList<Head, Tail...>> 
    : Contains<T, TypeList<Tail...>> {};

template<typename T, typename... Tail>
struct Contains<T, TypeList<T, Tail...>> : std::true_type {};

int main() {
    using MyTypes = TypeList<int, double, std::string>;
    
    std::cout << Contains<int, MyTypes>::value << "\n";      // 1
    std::cout << Contains<float, MyTypes>::value << "\n";    // 0
    std::cout << Contains<std::string, MyTypes>::value << "\n"; // 1
    
    return 0;
}
```

---

## Comprehensive Example: Generic Container

```cpp
#include <iostream>
#include <memory>
#include <concepts>
#include <stdexcept>

// Concept for types that can be stored in container
template<typename T>
concept Storable = std::copyable<T> || std::movable<T>;

// Generic container using modern C++ features
template<Storable T>
class Container {
    std::unique_ptr<T[]> data;
    size_t capacity;
    size_t count;
    
    void ensure_capacity(size_t new_capacity) {
        if (new_capacity > capacity) {
            auto new_data = std::make_unique<T[]>(new_capacity);
            for (size_t i = 0; i < count; ++i) {
                new_data[i] = std::move(data[i]);
            }
            data = std::move(new_data);
            capacity = new_capacity;
        }
    }
    
public:
    Container() : data(nullptr), capacity(0), count(0) {}
    
    explicit Container(size_t initial_capacity) 
        : data(std::make_unique<T[]>(initial_capacity)),
          capacity(initial_capacity),
          count(0) {}
    
    // Variadic template constructor
    template<typename... Args>
    void emplace_back(Args&&... args) {
        if (count >= capacity) {
            ensure_capacity(capacity == 0 ? 1 : capacity * 2);
        }
        data[count++] = T(std::forward<Args>(args)...);
    }
    
    void push_back(const T& value) requires std::copyable<T> {
        if (count >= capacity) {
            ensure_capacity(capacity == 0 ? 1 : capacity * 2);
        }
        data[count++] = value;
    }
    
    void push_back(T&& value) {
        if (count >= capacity) {
            ensure_capacity(capacity == 0 ? 1 : capacity * 2);
        }
        data[count++] = std::move(value);
    }
    
    T& operator[](size_t index) {
        if (index >= count) {
            throw std::out_of_range("Index out of bounds");
        }
        return data[index];
    }
    
    const T& operator[](size_t index) const {
        if (index >= count) {
            throw std::out_of_range("Index out of bounds");
        }
        return data[index];
    }
    
    size_t size() const { return count; }
    bool empty() const { return count == 0; }
    
    // Iterator support
    T* begin() { return data.get(); }
    T* end() { return data.get() + count; }
    const T* begin() const { return data.get(); }
    const T* end() const { return data.get() + count; }
};

// Generic print function using concepts
template<typename C>
requires requires(C c) {
    { c.begin() };
    { c.end() };
}
void print_container(const C& container) {
    std::cout << "[ ";
    for (const auto& elem : container) {
        std::cout << elem << " ";
    }
    std::cout << "]\n";
}

int main() {
    std::cout << "=== Integer Container ===\n";
    Container<int> int_container;
    int_container.push_back(10);
    int_container.push_back(20);
    int_container.emplace_back(30);
    print_container(int_container);
    
    std::cout << "\n=== String Container ===\n";
    Container<std::string> str_container(4);
    str_container.push_back("Hello");
    str_container.push_back("World");
    str_container.emplace_back(5, '!');  // Construct string with 5 '!'
    print_container(str_container);
    
    std::cout << "\n=== Accessing Elements ===\n";
    for (size_t i = 0; i < int_container.size(); ++i) {
        std::cout << "int_container[" << i << "] = " 
                  << int_container[i] << "\n";
    }
    
    return 0;
}
```

---

## Conclusion

Part 4 has provided comprehensive coverage of templates and generic programming in C++:

1. **Function Templates** - Type deduction, specialization, overloading, SFINAE
2. **Class Templates** - Parameters, specialization, CTAD, default arguments
3. **Template Metaprogramming** - Compile-time computation, type traits, tag dispatching
4. **Advanced Techniques** - Perfect forwarding, expression templates, CRTP
5. **C++20 Concepts** - Constraints, standard concepts, requires clauses
6. **Variadic Templates** - Parameter packs, fold expressions, perfect forwarding

**Key Principles:**
- **Type Safety**: Templates are type-checked at compile time
- **Zero Overhead**: Template code generates efficient machine code
- **Code Reuse**: Write once, works with any type
- **Compile-Time Computation**: Move work from runtime to compile time
- **Concepts**: Make constraints explicit and improve error messages

**Common Patterns:**
```cpp
// Basic function template
template<typename T>
T max(T a, T b);

// Class template with concepts
template<std::integral T>
class Container;

// Variadic template with fold expression
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);
}

// Perfect forwarding
template<typename T, typename... Args>
auto make(Args&&... args) {
    return T(std::forward<Args>(args)...);
}
```

**Best Practices:**
- Use concepts (C++20) instead of SFINAE when possible
- Prefer `constexpr if` over tag dispatching in C++17+
- Use fold expressions for variadic templates in C++17+
- Always use `std::forward` with universal references
- Document template requirements clearly
- Provide clear error messages (concepts help!)
- Test templates with various types

**Template Checklist:**
- [ ] Use concepts to constrain templates (C++20)
- [ ] Prefer `if constexpr` over SFINAE (C++17)
- [ ] Use fold expressions for variadic operations (C++17)
- [ ] Perfect forward with `std::forward`
- [ ] Document type requirements
- [ ] Test with multiple types
- [ ] Consider compilation time impact
- [ ] Use CTAD when appropriate (C++17)

**Next in the series:** Part 5 will explore the Standard Template Library (STL), covering containers, algorithms, iterators, and utilities in depth.

---

## References and Further Reading

- **C++20 Concepts**: https://en.cppreference.com/w/cpp/language/constraints
- **"C++ Templates: The Complete Guide"** by Vandevoorde, Josuttis, Gregor
- **"Modern C++ Design"** by Andrei Alexandrescu - Advanced template techniques
- **C++ Core Guidelines**: Templates section
- **cppreference.com**: Comprehensive template reference

---

*This blog post is part of the Complete C++ Mastery series. Continue to Part 5 for comprehensive coverage of the Standard Template Library.*
