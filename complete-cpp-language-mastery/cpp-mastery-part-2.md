---
title: "Complete C++ Mastery Part 2: Object-Oriented Programming & The C++ Object Model"
date: 2024-02-20 12:00:00 +0000
categories: [C++, Programming, OOP]
tags: [cpp, oop, classes, inheritance, polymorphism, object-model, virtual-functions, raii]
---

# Complete C++ Mastery Part 2: Object-Oriented Programming & The C++ Object Model

## Introduction

Welcome to Part 2 of the Complete C++ Mastery series. In this installment, we dive deep into Object-Oriented Programming (OOP) in C++ and explore the underlying object model that makes it all work. Unlike many OOP tutorials that focus solely on syntax, we'll examine the internal mechanisms: how objects are laid out in memory, how virtual functions are implemented, and how the compiler resolves method calls.

C++ provides a unique blend of OOP features with minimal overhead. Understanding the object model—the way classes, inheritance, and polymorphism are implemented at the machine level—empowers you to write efficient, maintainable, and correct object-oriented code.

**Prerequisites:** This part assumes you've completed Part 1 or have equivalent knowledge of C++ fundamentals including types, functions, pointers, and references.

---

## 2.1 Classes and Objects

### Basic Class Definition

A class is a user-defined type that encapsulates data and functions that operate on that data.

```cpp
#include <iostream>
#include <string>

// Class definition
class Person {
    // Private members (default access for class)
    std::string name;
    int age;

public:
    // Constructor
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "Person created: " << name << "\n";
    }
    
    // Destructor
    ~Person() {
        std::cout << "Person destroyed: " << name << "\n";
    }
    
    // Member functions
    void introduce() const {
        std::cout << "Hi, I'm " << name << ", age " << age << "\n";
    }
    
    void celebrate_birthday() {
        ++age;
        std::cout << name << " is now " << age << " years old!\n";
    }
    
    // Getters
    const std::string& get_name() const { return name; }
    int get_age() const { return age; }
    
    // Setters
    void set_age(int a) {
        if (a >= 0 && a <= 150) {  // Validation
            age = a;
        }
    }
};

int main() {
    // Creating objects
    Person alice("Alice", 30);
    Person bob("Bob", 25);
    
    // Calling member functions
    alice.introduce();
    bob.introduce();
    
    alice.celebrate_birthday();
    
    std::cout << "Alice's age: " << alice.get_age() << "\n";
    
    return 0;  // Destructors called automatically
}
```

**Output:**
```
Person created: Alice
Person created: Bob
Hi, I'm Alice, age 30
Hi, I'm Bob, age 25
Alice is now 31 years old!
Alice's age: 31
Person destroyed: Bob
Person destroyed: Alice
```

### Access Specifiers

C++ provides three access levels:

```cpp
class Example {
private:  // Default for class
    int private_data;
    void private_method() {}
    // Accessible only within this class

protected:
    int protected_data;
    void protected_method() {}
    // Accessible in this class and derived classes

public:
    int public_data;
    void public_method() {}
    // Accessible everywhere
};

struct ExampleStruct {
public:  // Default for struct
    int data;  // Public by default
};

int main() {
    Example ex;
    // ex.private_data = 10;  // ❌ Error: private
    // ex.protected_data = 20; // ❌ Error: protected
    ex.public_data = 30;       // ✅ OK: public
    
    return 0;
}
```

**Best Practices:**
```cpp
// ✅ GOOD: Encapsulation with private data, public interface
class BankAccount {
private:
    double balance;  // Encapsulated
    
public:
    BankAccount(double initial_balance) : balance(initial_balance) {}
    
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
    
    double get_balance() const { return balance; }
};

// ❌ BAD: Public data members (no encapsulation)
class BadAccount {
public:
    double balance;  // Anyone can modify directly!
};
```

### Constructors

Constructors initialize objects. C++ provides several types:

**Default Constructor:**
```cpp
class Point {
    int x, y;
    
public:
    // Default constructor
    Point() : x(0), y(0) {
        std::cout << "Default constructor\n";
    }
    
    // Or with default member initializers (C++11)
    // int x = 0;
    // int y = 0;
    // Point() = default;  // Compiler-generated
    
    void print() const {
        std::cout << "(" << x << ", " << y << ")\n";
    }
};

int main() {
    Point p;  // Calls default constructor
    p.print();
    
    return 0;
}
```

**Parameterized Constructor:**
```cpp
class Rectangle {
    int width, height;
    
public:
    // Parameterized constructor
    Rectangle(int w, int h) : width(w), height(h) {
        std::cout << "Rectangle created: " << width << "x" << height << "\n";
    }
    
    int area() const {
        return width * height;
    }
};

int main() {
    Rectangle rect(10, 5);  // Calls parameterized constructor
    std::cout << "Area: " << rect.area() << "\n";
    
    return 0;
}
```

**Constructor Initialization Lists:**
```cpp
class Complex {
    const int real;  // Const members must be initialized
    int& ref;        // Reference members must be initialized
    std::string str; // Complex types benefit from direct initialization
    
public:
    // ✅ GOOD: Member initialization list
    Complex(int r, int& rf, const std::string& s) 
        : real(r), ref(rf), str(s) {
        // Members already initialized
    }
    
    // ❌ BAD: Assignment in constructor body
    // Complex(int r, int& rf, const std::string& s) {
    //     real = r;  // ❌ Error: can't assign to const
    //     ref = rf;  // ❌ Error: reference already bound to garbage
    //     str = s;   // ⚠️ Works but inefficient (default construct + assign)
    // }
};
```

**Initialization Order:**
```cpp
class InitOrder {
    int first;
    int second;
    int third;
    
public:
    // ⚠️ Members initialized in DECLARATION order, not list order!
    InitOrder(int x) 
        : third(x), second(third + 1), first(second + 1) {
        // Actual order: first, second, third
        // first = uninitialized
        // second = uninitialized + 1  (UB!)
        // third = x
    }
    
    // ✅ GOOD: Initialize in declaration order
    InitOrder(int x) 
        : first(x), second(first + 1), third(second + 1) {
        // Now correct: first=x, second=x+1, third=x+2
    }
};
```

**Delegating Constructors (C++11):**
```cpp
class Point {
    int x, y;
    
public:
    // Main constructor
    Point(int x_val, int y_val) : x(x_val), y(y_val) {
        std::cout << "Main constructor\n";
    }
    
    // Delegate to main constructor
    Point() : Point(0, 0) {  // Calls Point(int, int)
        std::cout << "Default constructor (delegating)\n";
    }
    
    Point(int val) : Point(val, val) {  // Calls Point(int, int)
        std::cout << "Single-value constructor (delegating)\n";
    }
};

int main() {
    Point p1;       // Calls delegating default constructor
    Point p2(5);    // Calls delegating single-value constructor
    Point p3(3, 4); // Calls main constructor directly
    
    return 0;
}
```

### Destructors

Destructors clean up resources when objects are destroyed:

```cpp
#include <iostream>

class Resource {
    int* data;
    size_t size;
    
public:
    Resource(size_t n) : size(n) {
        data = new int[size];
        std::cout << "Resource allocated (" << size << " ints)\n";
    }
    
    ~Resource() {
        delete[] data;
        std::cout << "Resource deallocated\n";
    }
    
    void set(size_t index, int value) {
        if (index < size) {
            data[index] = value;
        }
    }
    
    int get(size_t index) const {
        return (index < size) ? data[index] : 0;
    }
};

int main() {
    {
        Resource res(100);
        res.set(0, 42);
        std::cout << "Value: " << res.get(0) << "\n";
        
    } // res goes out of scope, destructor called automatically
    
    std::cout << "After block\n";
    
    return 0;
}
```

**RAII (Resource Acquisition Is Initialization):**
```cpp
#include <fstream>
#include <mutex>

// ✅ GOOD: RAII pattern - resource tied to object lifetime
class FileWriter {
    std::ofstream file;
    
public:
    FileWriter(const std::string& filename) : file(filename) {
        if (!file.is_open()) {
            throw std::runtime_error("Cannot open file");
        }
    }
    
    ~FileWriter() {
        // File automatically closed
        std::cout << "File closed\n";
    }
    
    void write(const std::string& data) {
        file << data;
    }
};

void process_file() {
    FileWriter writer("output.txt");
    writer.write("Hello, RAII!\n");
    
    // Even if exception thrown here, destructor ensures file is closed
    // No manual cleanup needed!
}

// ❌ BAD: Manual resource management
void bad_process_file() {
    std::ofstream file("output.txt");
    
    // Do work...
    
    file.close();  // What if exception thrown before this?
}
```

### Copy Constructor and Copy Assignment

**Copy Constructor:**
```cpp
class Array {
    int* data;
    size_t size;
    
public:
    // Constructor
    Array(size_t n) : size(n), data(new int[n]) {
        std::cout << "Constructor\n";
    }
    
    // Copy constructor
    Array(const Array& other) : size(other.size), data(new int[other.size]) {
        std::cout << "Copy constructor\n";
        std::copy(other.data, other.data + size, data);
    }
    
    // Destructor
    ~Array() {
        delete[] data;
        std::cout << "Destructor\n";
    }
    
    int& operator[](size_t index) { return data[index]; }
    const int& operator[](size_t index) const { return data[index]; }
};

int main() {
    Array arr1(5);
    arr1[0] = 42;
    
    Array arr2 = arr1;  // Copy constructor called
    std::cout << "arr2[0] = " << arr2[0] << "\n";
    
    arr2[0] = 100;
    std::cout << "arr1[0] = " << arr1[0] << "\n";  // Still 42 (deep copy)
    
    return 0;
}
```

**Copy Assignment Operator:**
```cpp
class Array {
    int* data;
    size_t size;
    
public:
    Array(size_t n) : size(n), data(new int[n]) {}
    
    // Copy constructor
    Array(const Array& other) : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    // Copy assignment operator
    Array& operator=(const Array& other) {
        std::cout << "Copy assignment\n";
        
        if (this == &other) {  // Self-assignment check
            return *this;
        }
        
        // Deallocate old memory
        delete[] data;
        
        // Allocate new memory and copy
        size = other.size;
        data = new int[size];
        std::copy(other.data, other.data + size, data);
        
        return *this;
    }
    
    ~Array() {
        delete[] data;
    }
    
    int& operator[](size_t index) { return data[index]; }
};

int main() {
    Array arr1(5);
    arr1[0] = 42;
    
    Array arr2(10);
    arr2 = arr1;  // Copy assignment called
    
    std::cout << "arr2[0] = " << arr2[0] << "\n";  // 42
    
    return 0;
}
```

**Copy-and-Swap Idiom (Exception-Safe):**
```cpp
#include <algorithm>  // For std::swap

class Array {
    int* data;
    size_t size;
    
public:
    Array(size_t n) : size(n), data(new int[n]) {}
    
    Array(const Array& other) : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    // ✅ Exception-safe copy assignment using copy-and-swap
    Array& operator=(Array other) {  // Pass by value (copy made)
        swap(*this, other);  // Swap with temporary
        return *this;        // old data destroyed when 'other' goes out of scope
    }
    
    ~Array() {
        delete[] data;
    }
    
    // Friend swap function
    friend void swap(Array& first, Array& second) noexcept {
        using std::swap;
        swap(first.size, second.size);
        swap(first.data, second.data);
    }
};
```

### Move Constructor and Move Assignment (C++11)

Move semantics enable efficient transfer of resources:

```cpp
#include <iostream>
#include <utility>  // For std::move

class Array {
    int* data;
    size_t size;
    
public:
    // Constructor
    Array(size_t n) : size(n), data(new int[n]) {
        std::cout << "Constructor\n";
    }
    
    // Copy constructor
    Array(const Array& other) : size(other.size), data(new int[other.size]) {
        std::cout << "Copy constructor\n";
        std::copy(other.data, other.data + size, data);
    }
    
    // Move constructor
    Array(Array&& other) noexcept : size(other.size), data(other.data) {
        std::cout << "Move constructor\n";
        
        // Leave 'other' in valid but unspecified state
        other.data = nullptr;
        other.size = 0;
    }
    
    // Copy assignment
    Array& operator=(const Array& other) {
        std::cout << "Copy assignment\n";
        
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }
    
    // Move assignment
    Array& operator=(Array&& other) noexcept {
        std::cout << "Move assignment\n";
        
        if (this != &other) {
            // Release current resources
            delete[] data;
            
            // Steal other's resources
            data = other.data;
            size = other.size;
            
            // Leave 'other' in valid state
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
    
    // Destructor
    ~Array() {
        delete[] data;
        std::cout << "Destructor\n";
    }
    
    size_t get_size() const { return size; }
};

Array create_array(size_t n) {
    Array arr(n);
    return arr;  // Move constructor called (RVO may eliminate it)
}

int main() {
    std::cout << "=== Creating arr1 ===\n";
    Array arr1(10);
    
    std::cout << "\n=== Copy construction ===\n";
    Array arr2 = arr1;  // Copy constructor
    
    std::cout << "\n=== Move construction ===\n";
    Array arr3 = std::move(arr1);  // Move constructor
    std::cout << "arr1 size after move: " << arr1.get_size() << "\n";  // 0
    
    std::cout << "\n=== Move assignment ===\n";
    arr2 = std::move(arr3);  // Move assignment
    
    std::cout << "\n=== Function return (RVO) ===\n";
    Array arr4 = create_array(5);  // Likely no copy/move (RVO)
    
    std::cout << "\n=== End of main ===\n";
    return 0;
}
```

### Rule of Zero, Rule of Three, Rule of Five

**Rule of Zero (Prefer This!):**
```cpp
#include <vector>
#include <string>

// ✅ BEST: Use standard containers, no manual memory management
class Person {
    std::string name;
    std::vector<std::string> hobbies;
    // Compiler-generated special members are correct!
    // No need to write copy/move constructors, assignment, or destructor
    
public:
    Person(const std::string& n) : name(n) {}
    
    void add_hobby(const std::string& hobby) {
        hobbies.push_back(hobby);
    }
};
```

**Rule of Three (C++98):**
If you need to define one of the following, you probably need to define all three:
1. Destructor
2. Copy constructor
3. Copy assignment operator

```cpp
// ⚠️ C++98 style: Manual memory management
class OldStyleArray {
    int* data;
    size_t size;
    
public:
    OldStyleArray(size_t n) : size(n), data(new int[n]) {}
    
    // Must define all three:
    ~OldStyleArray() {
        delete[] data;
    }
    
    OldStyleArray(const OldStyleArray& other) 
        : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    OldStyleArray& operator=(const OldStyleArray& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }
};
```

**Rule of Five (C++11+):**
If you define any of the five, consider defining all:
1. Destructor
2. Copy constructor
3. Copy assignment
4. Move constructor
5. Move assignment

```cpp
// Modern style: Add move semantics
class ModernArray {
    int* data;
    size_t size;
    
public:
    ModernArray(size_t n) : size(n), data(new int[n]) {}
    
    ~ModernArray() {
        delete[] data;
    }
    
    // Copy operations
    ModernArray(const ModernArray& other) 
        : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    ModernArray& operator=(const ModernArray& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }
    
    // Move operations
    ModernArray(ModernArray&& other) noexcept 
        : size(other.size), data(other.data) {
        other.data = nullptr;
        other.size = 0;
    }
    
    ModernArray& operator=(ModernArray&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};
```

### Explicitly Defaulted and Deleted Functions (C++11)

```cpp
class Point {
    int x, y;
    
public:
    Point(int x, int y) : x(x), y(y) {}
    
    // Explicitly request compiler-generated copy constructor
    Point(const Point&) = default;
    
    // Explicitly request compiler-generated copy assignment
    Point& operator=(const Point&) = default;
    
    // Move operations
    Point(Point&&) = default;
    Point& operator=(Point&&) = default;
    
    // Default destructor
    ~Point() = default;
};

class NonCopyable {
public:
    NonCopyable() = default;
    
    // Delete copy operations (prevent copying)
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
    
    // Allow move operations
    NonCopyable(NonCopyable&&) = default;
    NonCopyable& operator=(NonCopyable&&) = default;
};

int main() {
    Point p1(10, 20);
    Point p2 = p1;  // ✅ OK: copy constructor is defaulted
    
    NonCopyable nc1;
    // NonCopyable nc2 = nc1;  // ❌ Error: copy constructor deleted
    NonCopyable nc3 = std::move(nc1);  // ✅ OK: move constructor exists
    
    return 0;
}
```

### Member Initialization

**Default Member Initializers (C++11):**
```cpp
class Config {
    int max_connections = 100;        // Default value
    double timeout = 30.0;
    bool debug_mode = false;
    std::string server_name = "localhost";
    
public:
    // Constructor can override defaults
    Config() = default;  // Uses defaults
    
    Config(int max_conn) : max_connections(max_conn) {
        // timeout, debug_mode, server_name use defaults
    }
    
    Config(int max_conn, double tout) 
        : max_connections(max_conn), timeout(tout) {
        // debug_mode, server_name use defaults
    }
};

int main() {
    Config c1;              // Uses all defaults
    Config c2(200);         // max_connections=200, rest default
    Config c3(200, 60.0);   // max_connections=200, timeout=60.0, rest default
    
    return 0;
}
```

**Aggregate Initialization:**
```cpp
// Aggregate: no user-defined constructors, all public members
struct Point {
    int x;
    int y;
};

struct Rectangle {
    Point top_left;
    Point bottom_right;
};

int main() {
    // Aggregate initialization
    Point p1 = {10, 20};
    Point p2{30, 40};  // C++11 uniform initialization
    
    Rectangle rect = {{0, 0}, {100, 50}};
    
    // Designated initializers (C++20)
    #if __cplusplus >= 202002L
    Point p3 = {.x = 10, .y = 20};
    Rectangle rect2 = {.top_left = {0, 0}, .bottom_right = {100, 50}};
    #endif
    
    return 0;
}
```

### SDLC Integration: Classes and Objects

**Requirements Phase:**
- Identify entities that should be classes
- Define public interfaces (methods)
- Specify encapsulation requirements
- Document resource ownership

**Design Phase:**
```cpp
// Design class with clear responsibilities (Single Responsibility Principle)

// ✅ GOOD: Single responsibility - manages database connection
class DatabaseConnection {
private:
    std::string connection_string;
    bool connected;
    
public:
    explicit DatabaseConnection(const std::string& conn_str);
    ~DatabaseConnection();  // RAII: closes connection
    
    bool connect();
    void disconnect();
    bool is_connected() const;
    
    // Prevent copying (resource should not be duplicated)
    DatabaseConnection(const DatabaseConnection&) = delete;
    DatabaseConnection& operator=(const DatabaseConnection&) = delete;
    
    // Allow moving (transfer ownership)
    DatabaseConnection(DatabaseConnection&&) noexcept = default;
    DatabaseConnection& operator=(DatabaseConnection&&) noexcept = default;
};

// ❌ BAD: Too many responsibilities
class DatabaseConnectionAndLogger {
    // Manages connection AND logging - violates SRP
};
```

**Implementation Phase:**
```cpp
// Implement with proper error handling
DatabaseConnection::DatabaseConnection(const std::string& conn_str)
    : connection_string(conn_str), connected(false) {
    if (conn_str.empty()) {
        throw std::invalid_argument("Connection string cannot be empty");
    }
}

DatabaseConnection::~DatabaseConnection() {
    if (connected) {
        disconnect();
    }
}

bool DatabaseConnection::connect() {
    if (!connected) {
        // Attempt connection...
        connected = true;
        return true;
    }
    return false;
}

void DatabaseConnection::disconnect() {
    if (connected) {
        // Close connection...
        connected = false;
    }
}
```

**Testing Phase:**
```cpp
#include <cassert>

void test_database_connection() {
    // Test construction
    DatabaseConnection db("server=localhost");
    assert(!db.is_connected());
    
    // Test connection
    assert(db.connect());
    assert(db.is_connected());
    
    // Test disconnection
    db.disconnect();
    assert(!db.is_connected());
    
    // Test RAII (destructor closes connection)
    {
        DatabaseConnection temp_db("server=test");
        temp_db.connect();
    }  // Destructor should close connection
    
    // Test invalid construction
    try {
        DatabaseConnection bad_db("");
        assert(false);  // Should not reach here
    } catch (const std::invalid_argument&) {
        // Expected
    }
}
```

**Code Review Checklist:**
- [ ] All data members private or protected
- [ ] Const-correctness for member functions
- [ ] RAII for resource management
- [ ] Rule of Zero, Three, or Five followed
- [ ] Move operations marked noexcept when possible
- [ ] Self-assignment handled in assignment operators
- [ ] Initialization lists used for member initialization
- [ ] Virtual destructor if class is intended as base class

### Common Pitfalls

**1. Shallow Copy Problem:**
```cpp
// ❌ BAD: Compiler-generated copy does shallow copy
class BadArray {
public:
    int* data;
    size_t size;
    
    BadArray(size_t n) : size(n), data(new int[n]) {}
    
    ~BadArray() {
        delete[] data;
    }
    
    // No explicit copy constructor - compiler generates shallow copy!
};

int main() {
    BadArray arr1(5);
    arr1.data[0] = 42;
    
    BadArray arr2 = arr1;  // Shallow copy: both point to same memory!
    arr2.data[0] = 100;    // Modifies arr1's data too!
    
    // When destructors run: double-free! (both try to delete same memory)
    return 0;
}

// ✅ GOOD: Explicit deep copy constructor
class GoodArray {
    int* data;
    size_t size;
    
public:
    GoodArray(size_t n) : size(n), data(new int[n]) {}
    
    // Deep copy constructor
    GoodArray(const GoodArray& other) 
        : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    ~GoodArray() {
        delete[] data;
    }
};
```

**2. Self-Assignment Bug:**
```cpp
// ❌ BAD: Doesn't handle self-assignment
class BadAssignment {
    int* data;
    
public:
    BadAssignment& operator=(const BadAssignment& other) {
        delete[] data;  // Oops! If this == &other, we just deleted other's data too!
        data = new int[10];
        // Copy from other.data... but it's already deleted!
        return *this;
    }
};

// ✅ GOOD: Check for self-assignment
class GoodAssignment {
    int* data;
    
public:
    GoodAssignment& operator=(const GoodAssignment& other) {
        if (this == &other) {  // Self-assignment check
            return *this;
        }
        
        delete[] data;
        data = new int[10];
        // Safe to copy from other
        return *this;
    }
};
```

**3. Forgetting to Return *this in Assignment:**
```cpp
// ❌ BAD: Return type is void
class BadChaining {
public:
    void operator=(const BadChaining& other) {  // void return!
        // ...
    }
};

int main() {
    BadChaining a, b, c;
    // a = b = c;  // ❌ Error: can't chain assignments
}

// ✅ GOOD: Return reference to *this
class GoodChaining {
public:
    GoodChaining& operator=(const GoodChaining& other) {
        // ...
        return *this;
    }
};

int main() {
    GoodChaining a, b, c;
    a = b = c;  // ✅ OK: chains correctly
}
```

### Performance Implications

**Construction Cost:**
```cpp
class Expensive {
    std::vector<int> large_vector;
    std::string long_string;
    
public:
    // ❌ Inefficient: Default-construct then assign
    Expensive(size_t n, const std::string& s) {
        large_vector.resize(n);  // Default construct, then resize
        long_string = s;          // Default construct, then assign
    }
    
    // ✅ Efficient: Direct initialization in initializer list
    Expensive(size_t n, const std::string& s) 
        : large_vector(n), long_string(s) {
        // Members constructed directly with correct values
    }
};
```

**Move vs Copy:**
```cpp
#include <chrono>
#include <vector>

std::vector<int> create_large_vector() {
    std::vector<int> vec(1000000, 42);
    return vec;  // Move (or RVO)
}

int main() {
    using namespace std::chrono;
    
    // Copy: expensive
    auto start = high_resolution_clock::now();
    std::vector<int> vec1(1000000, 42);
    std::vector<int> vec2 = vec1;  // Copy: ~4ms
    auto end = high_resolution_clock::now();
    std::cout << "Copy: " << duration_cast<microseconds>(end - start).count() << "μs\n";
    
    // Move: cheap
    start = high_resolution_clock::now();
    std::vector<int> vec3 = std::move(vec1);  // Move: ~0.1μs
    end = high_resolution_clock::now();
    std::cout << "Move: " << duration_cast<microseconds>(end - start).count() << "μs\n";
    
    return 0;
}
```

### Frequently Asked Questions

**Q1: What's the difference between `struct` and `class` in C++?**

**A:** The only difference is **default access level**:
- `class`: members are **private** by default
- `struct`: members are **public** by default

```cpp
class ClassExample {
    int x;  // private by default
};

struct StructExample {
    int x;  // public by default
};

// Functionally equivalent:
class ExplicitClass {
public:
    int x;
};

struct ExplicitStruct {
    int x;
};
```

**Convention:** Use `struct` for plain data aggregates (POD types), `class` for types with invariants and behavior.

**Related Concepts:** Access specifiers, POD types, aggregate initialization

---

**Q2: When should I use initializer lists vs assignment in constructor body?**

**A:** **Always prefer initializer lists** when possible:

**Reasons:**
1. **Performance**: Direct initialization vs default-construct-then-assign
2. **Correctness**: Only way to initialize const and reference members
3. **Base class initialization**: Required for base class constructors

```cpp
class Example {
    const int constant;
    int& reference;
    std::string str;
    
public:
    // ✅ REQUIRED: Initializer list for const and reference
    Example(int c, int& r, const std::string& s) 
        : constant(c), reference(r), str(s) {
    }
    
    // ❌ Won't compile
    // Example(int c, int& r, const std::string& s) {
    //     constant = c;  // Error: can't assign to const
    //     reference = r; // Error: reference must be initialized
    //     str = s;       // Works but less efficient
    // }
};
```

**When constructor body is necessary:**
- Complex initialization logic
- Error handling with try-catch
- Calling functions that might throw

**Related Concepts:** RAII, const correctness, references

---

**Q3: Why do move constructors and move assignment need to be `noexcept`?**

**A:** Move operations should be `noexcept` because:

1. **Strong exception guarantee**: STL containers can only use move operations for reallocation if they're `noexcept`. Otherwise, they fall back to copy (expensive!).

2. **Performance**: Compiler can generate better code when it knows operations won't throw.

3. **Correctness**: If a move throws halfway through, object state may be inconsistent.

```cpp
class Resource {
    int* data;
    
public:
    // ✅ GOOD: noexcept move operations
    Resource(Resource&& other) noexcept 
        : data(other.data) {
        other.data = nullptr;  // No throw
    }
    
    Resource& operator=(Resource&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            other.data = nullptr;  // No throw
        }
        return *this;
    }
};

// Impact on std::vector:
std::vector<Resource> vec;
vec.push_back(Resource());  // If move is noexcept: uses move on reallocation
                            // If move can throw: uses copy on reallocation (slow!)
```

**Related Concepts:** Exception safety, strong guarantee, std::move_if_noexcept

---

### Interview Questions

**Question 1: Explain the Rule of Zero, Rule of Three, and Rule of Five. When should each be applied?**

**Difficulty:** Mid-Level/Senior

**Answer:**

**Rule of Zero (Preferred):**
Don't define any special member functions. Let the compiler generate them correctly by using standard containers and RAII wrappers.

```cpp
// ✅ BEST: Rule of Zero
class Person {
    std::string name;
    std::vector<std::string> addresses;
    std::unique_ptr<Details> details;
    // No special members needed!
};
```

**When to use:** Always, unless you're managing raw resources.

**Rule of Three (C++98):**
If you define one of these, define all three:
1. Destructor
2. Copy constructor
3. Copy assignment operator

```cpp
// Managing raw resource → Rule of Three
class LegacyArray {
    int* data;
    size_t size;
    
public:
    ~LegacyArray() { delete[] data; }
    LegacyArray(const LegacyArray& other) { /* deep copy */ }
    LegacyArray& operator=(const LegacyArray& other) { /* deep copy */ }
};
```

**When to use:** C++98/03 code or when interfacing with C APIs requiring manual memory management.

**Rule of Five (C++11+):**
If you define any special member, define all five:
1. Destructor
2. Copy constructor
3. Copy assignment
4. Move constructor
5. Move assignment

```cpp
// Managing raw resource + move semantics → Rule of Five
class ModernArray {
    int* data;
    size_t size;
    
public:
    ~ModernArray() { delete[] data; }
    
    // Copy operations
    ModernArray(const ModernArray& other) { /* deep copy */ }
    ModernArray& operator=(const ModernArray& other) { /* deep copy */ }
    
    // Move operations
    ModernArray(ModernArray&& other) noexcept { /* steal resources */ }
    ModernArray& operator=(ModernArray&& other) noexcept { /* steal resources */ }
};
```

**When to use:** C++11+ code managing raw resources where performance matters.

**Decision tree:**
```
Are you managing a raw resource (raw pointer, file handle, etc.)?
├─ No → Rule of Zero (use std::vector, std::unique_ptr, etc.)
└─ Yes → Are you writing C++11+ code?
    ├─ No → Rule of Three
    └─ Yes → Rule of Five
```

**Why This Matters:** Following these rules prevents resource leaks, double-frees, and undefined behavior. Most modern C++ code should use Rule of Zero.

**Follow-up Questions:**
* What happens if you only define destructor and copy constructor but not copy assignment?
* Can you mix defaulted and user-defined special members?
* What is the "Rule of Four and a Half"?

---

**Question 2: What is the copy-and-swap idiom? Why is it exception-safe?**

**Difficulty:** Senior

**Answer:**

The copy-and-swap idiom is a technique for implementing the assignment operator that provides strong exception safety.

**Traditional approach (not exception-safe):**
```cpp
Array& operator=(const Array& other) {
    if (this != &other) {
        delete[] data;  // ⚠️ If new throws, object is in invalid state!
        size = other.size;
        data = new int[size];  // Might throw
        std::copy(other.data, other.data + size, data);
    }
    return *this;
}
```

**Copy-and-swap idiom:**
```cpp
class Array {
    int* data;
    size_t size;
    
public:
    // Copy constructor (must be exception-safe)
    Array(const Array& other) 
        : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    // Assignment via copy-and-swap
    Array& operator=(Array other) {  // Pass by value: copy made
        swap(*this, other);           // Swap (no throw)
        return *this;                 // old data destroyed when 'other' destructs
    }
    
    // Friend swap (no throw)
    friend void swap(Array& first, Array& second) noexcept {
        using std::swap;
        swap(first.size, second.size);
        swap(first.data, second.data);
    }
    
    ~Array() { delete[] data; }
};
```

**Why it's exception-safe:**
1. Copy is made in the parameter (pass-by-value)
2. If copy throws, `*this` is unchanged (strong guarantee)
3. Swap operation is no-throw
4. Old resources released when temporary destructs

**Benefits:**
- Exception-safe (strong guarantee)
- Handles self-assignment automatically (copy creates different object)
- Consistent with move assignment (same implementation!)

**Unified with move:**
```cpp
Array& operator=(Array other) {  // Takes by value
    swap(*this, other);
    return *this;
}

// Works for both:
Array a(10), b(20);
a = b;              // Copy: b copied to parameter, then swapped
a = Array(30);      // Move: temporary moved to parameter, then swapped
```

**Why This Matters:** Demonstrates deep understanding of exception safety, resource management, and modern C++ idioms. Copy-and-swap is a fundamental technique in robust C++ design.

**Follow-up Questions:**
* What's the performance trade-off of copy-and-swap?
* How does move assignment fit into this idiom?
* What is the strong exception safety guarantee?

---

**Question 3: What is member initialization order and why does it matter?**

**Difficulty:** Mid-Level

**Answer:**

Members are initialized in the **order they are declared in the class**, not the order in the initializer list.

**Problem:**
```cpp
class BadOrder {
    int second;
    int first;
    
public:
    // ⚠️ Bug: initialization order doesn't match list order!
    BadOrder(int x) : first(x), second(first + 1) {
        // Actual initialization order:
        // 1. second = first + 1  (but first is uninitialized!)
        // 2. first = x
        // Result: second has undefined value
    }
};
```

**Correct:**
```cpp
class GoodOrder {
    int first;   // Declared first
    int second;  // Declared second
    
public:
    GoodOrder(int x) : first(x), second(first + 1) {
        // Initialization order matches declaration:
        // 1. first = x
        // 2. second = first + 1
        // Result: correct
    }
};
```

**Why declaration order matters:**
1. **Memory layout**: Members are laid out in memory in declaration order
2. **Destruction order**: Destructors run in reverse declaration order
3. **Consistency**: Same order for all constructors

**Compiler warnings:**
Most compilers warn about initialization order mismatches:
```bash
$ g++ -Wall -Wextra bad_order.cpp
warning: 'BadOrder::first' will be initialized after
warning:   'int BadOrder::second'
```

**Best practice:**
```cpp
// ✅ GOOD: Initialize in declaration order
class Config {
    int port;         // 1st
    std::string host; // 2nd
    int timeout;      // 3rd
    
public:
    Config(int p, const std::string& h, int t)
        : port(p), host(h), timeout(t) {  // Same order as declaration
    }
};
```

**Why This Matters:** Unintended initialization order is a subtle bug that can cause undefined behavior. Understanding this prevents hard-to-debug issues.

**Follow-up Questions:**
* What happens if base class members and derived class members have dependencies?
* How do static members fit into initialization order?
* What is the "static initialization order fiasco"?

---

### Key Takeaways

✅ Classes encapsulate data and behavior; always make data members private

✅ Use initializer lists for member initialization (required for const/reference members)

✅ Follow Rule of Zero when possible (use standard containers and RAII)

✅ If managing raw resources, follow Rule of Three (C++98) or Rule of Five (C++11+)

✅ Move operations should be marked `noexcept` for optimal STL container performance

✅ RAII ties resource lifetime to object lifetime (acquire in constructor, release in destructor)

✅ Copy-and-swap idiom provides exception-safe assignment

✅ Members initialize in declaration order, not initializer list order

✅ Use `= default` and `= delete` to explicitly control special member functions

✅ Pass large objects by const reference; return large objects by value (trust RVO)

✅ Self-assignment check is critical in assignment operators

---

[Due to length constraints, this is approximately 40% of Part 2. The complete document would continue with sections 2.2-2.6 covering The C++ Object Model, Inheritance and Polymorphism, Special Member Functions, Operator Overloading, and Static Members and Friends, totaling 18,000-20,000 words as specified.]

Would you like me to continue with the remaining sections (2.2-2.6)?

## 2.2 The C++ Object Model

### Memory Layout of Objects

Understanding how objects are laid out in memory is crucial for performance optimization and low-level programming.

**Simple Class Layout:**
```cpp
#include <iostream>

class Simple {
    int a;      // 4 bytes
    char b;     // 1 byte
    int c;      // 4 bytes
};

int main() {
    std::cout << "sizeof(Simple): " << sizeof(Simple) << "\n";  // Likely 12, not 9
    std::cout << "alignof(Simple): " << alignof(Simple) << "\n"; // Likely 4
    
    Simple s;
    std::cout << "Address of a: " << &s.a << "\n";
    std::cout << "Address of b: " << &s.b << "\n";
    std::cout << "Address of c: " << &s.c << "\n";
    
    return 0;
}
```

**Memory Layout:**
```
+---+---+---+---+---+---+---+---+---+---+---+---+
| a | a | a | a | b |pad|pad|pad| c | c | c | c |
+---+---+---+---+---+---+---+---+---+---+---+---+
0   1   2   3   4   5   6   7   8   9   10  11
```

**Padding and Alignment:** Compilers insert padding to align members to their natural boundaries for performance.

**Optimized Layout:**
```cpp
class Optimized {
    int a;      // 4 bytes
    int c;      // 4 bytes  (moved next to a)
    char b;     // 1 byte
    // 3 bytes padding at end
};
// sizeof(Optimized) = 12 (same as Simple)

class Better {
    char b;     // 1 byte
    // 3 bytes padding
    int a;      // 4 bytes
    int c;      // 4 bytes
};
// sizeof(Better) = 12 (still padded)

// Best: group small members
class Best {
    int a;      // 4 bytes
    int c;      // 4 bytes
    char b;     // 1 byte
    char d;     // 1 byte
    char e;     // 1 byte
    char f;     // 1 byte
};
// sizeof(Best) = 12 (no wasted padding!)
```


### Padding and Alignment Control

```cpp
#include <iostream>

// Natural alignment
struct Natural {
    char a;     // 1 byte
    // 3 bytes padding
    int b;      // 4 bytes
    char c;     // 1 byte
    // 3 bytes padding
};
// sizeof(Natural) = 12 bytes

// Packed structure (platform-specific)
#pragma pack(push, 1)
struct Packed {
    char a;     // 1 byte
    int b;      // 4 bytes
    char c;     // 1 byte
};
#pragma pack(pop)
// sizeof(Packed) = 6 bytes (no padding!)

// C++11 alignas
struct alignas(16) Aligned {
    char data[10];
};
// sizeof(Aligned) = 16 (aligned to 16-byte boundary)

int main() {
    std::cout << "sizeof(Natural): " << sizeof(Natural) << "\n";
    std::cout << "sizeof(Packed): " << sizeof(Packed) << "\n";
    std::cout << "sizeof(Aligned): " << sizeof(Aligned) << "\n";
    std::cout << "alignof(Aligned): " << alignof(Aligned) << "\n";
    
    return 0;
}
```

**Performance Impact:**
```cpp
// Aligned access: fast (single CPU instruction)
struct Aligned {
    int data;  // 4-byte aligned
};

// Misaligned access: slower (multiple instructions, or hardware exception)
#pragma pack(push, 1)
struct Misaligned {
    char c;
    int data;  // Not 4-byte aligned!
};
#pragma pack(pop)

// Modern CPUs handle misaligned access but with performance penalty
```

### Empty Base Optimization (EBO)

```cpp
#include <iostream>

class Empty {};  // sizeof(Empty) = 1 (cannot be 0)

class HasEmpty {
    Empty e;
    int data;
};
// sizeof(HasEmpty) = 8 (Empty takes space + padding)

// Empty Base Optimization
class DerivedFromEmpty : Empty {
    int data;
};
// sizeof(DerivedFromEmpty) = 4 (Empty base takes no space!)

// Practical use: Policy-based design, stateless allocators
template<typename T, typename Allocator = std::allocator<T>>
class Vector : private Allocator {  // EBO applies
    T* data;
    size_t size;
    // Allocator takes no space if empty!
};

int main() {
    std::cout << "sizeof(Empty): " << sizeof(Empty) << "\n";
    std::cout << "sizeof(HasEmpty): " << sizeof(HasEmpty) << "\n";
    std::cout << "sizeof(DerivedFromEmpty): " << sizeof(DerivedFromEmpty) << "\n";
    
    return 0;
}
```

### POD and Standard-Layout Types

**POD (Plain Old Data) - C++03 concept:**
```cpp
// POD type requirements (C++03):
// - Trivial default constructor
// - Trivial copy/move operations
// - Trivial destructor
// - All non-static members are POD
// - No virtual functions or virtual base classes

struct POD {
    int x;
    double y;
    // Trivial operations (compiler-generated)
};

// POD properties:
// - Can use memcpy
// - Compatible with C
// - Can be initialized with brace initialization
```

**Standard-Layout Types (C++11):**
```cpp
#include <type_traits>

// Standard-layout: predictable memory layout
struct StandardLayout {
    int x;
    double y;
    
    // Can have constructors/destructor (unlike POD)
    StandardLayout() : x(0), y(0.0) {}
};

// All members must have same access specifier
struct NotStandardLayout {
private:
    int x;
public:
    int y;  // Different access: not standard-layout
};

int main() {
    std::cout << std::boolalpha;
    std::cout << "POD is_trivial: " 
              << std::is_trivial<POD>::value << "\n";
    std::cout << "POD is_standard_layout: " 
              << std::is_standard_layout<POD>::value << "\n";
    std::cout << "StandardLayout is_standard_layout: " 
              << std::is_standard_layout<StandardLayout>::value << "\n";
    
    return 0;
}
```

### Object Lifetime

```cpp
#include <iostream>

class Tracker {
    int id;
public:
    Tracker(int i) : id(i) {
        std::cout << "Tracker " << id << " constructed\n";
    }
    
    ~Tracker() {
        std::cout << "Tracker " << id << " destroyed\n";
    }
};

int main() {
    std::cout << "=== Automatic storage ===\n";
    {
        Tracker t1(1);  // Created
        Tracker t2(2);
    }  // Destroyed in reverse order: t2, t1
    
    std::cout << "\n=== Dynamic storage ===\n";
    Tracker* p = new Tracker(3);
    delete p;  // Manual destruction
    
    std::cout << "\n=== Array ===\n";
    {
        Tracker arr[3] = {Tracker(4), Tracker(5), Tracker(6)};
        // Created in order: 4, 5, 6
    }  // Destroyed in reverse: 6, 5, 4
    
    std::cout << "\n=== End ===\n";
    return 0;
}
```

---

## 2.3 Inheritance and Polymorphism

### Single Inheritance

```cpp
#include <iostream>
#include <string>

// Base class
class Animal {
protected:
    std::string name;
    
public:
    Animal(const std::string& n) : name(n) {
        std::cout << "Animal constructor: " << name << "\n";
    }
    
    virtual ~Animal() {  // Virtual destructor (important!)
        std::cout << "Animal destructor: " << name << "\n";
    }
    
    void eat() const {
        std::cout << name << " is eating\n";
    }
    
    virtual void make_sound() const {
        std::cout << name << " makes a sound\n";
    }
};

// Derived class
class Dog : public Animal {
    std::string breed;
    
public:
    Dog(const std::string& n, const std::string& b) 
        : Animal(n), breed(b) {
        std::cout << "Dog constructor: " << name << "\n";
    }
    
    ~Dog() {
        std::cout << "Dog destructor: " << name << "\n";
    }
    
    // Override virtual function
    void make_sound() const override {  // override keyword (C++11)
        std::cout << name << " barks: Woof!\n";
    }
    
    void fetch() const {
        std::cout << name << " fetches the ball\n";
    }
};

int main() {
    std::cout << "=== Creating Dog ===\n";
    Dog dog("Buddy", "Golden Retriever");
    
    std::cout << "\n=== Calling methods ===\n";
    dog.eat();         // Inherited from Animal
    dog.make_sound();  // Overridden in Dog
    dog.fetch();       // Dog-specific
    
    std::cout << "\n=== Polymorphism ===\n";
    Animal* ptr = &dog;  // Pointer to base
    ptr->make_sound();   // Calls Dog::make_sound() (dynamic dispatch)
    
    std::cout << "\n=== End ===\n";
    return 0;
}
```

**Output:**
```
=== Creating Dog ===
Animal constructor: Buddy
Dog constructor: Buddy

=== Calling methods ===
Buddy is eating
Buddy barks: Woof!
Buddy fetches the ball

=== Polymorphism ===
Buddy barks: Woof!

=== End ===
Dog destructor: Buddy
Animal destructor: Buddy
```

### Virtual Functions and Dynamic Dispatch

**How Virtual Functions Work:**
```cpp
class Base {
public:
    virtual void func() { std::cout << "Base::func\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void func() override { std::cout << "Derived::func\n"; }
};

int main() {
    Base* b = new Derived();
    b->func();  // Calls Derived::func() (dynamic dispatch)
    delete b;
    
    return 0;
}
```

**Virtual Table (vtable) and Virtual Pointer (vptr):**

```
Base object memory layout:
+--------+
| vptr   | -----> [Base vtable]
+--------+          | &Base::func     |
| data   |          | &Base::~Base    |
+--------+

Derived object memory layout:
+--------+
| vptr   | -----> [Derived vtable]
+--------+          | &Derived::func     |
| Base   |          | &Derived::~Derived |
| data   |
+--------+
| Der.   |
| data   |
+--------+
```

**Virtual Function Call Process:**
1. Access object's vptr
2. Index into vtable
3. Call function through pointer

```cpp
// Conceptual translation of polymorphic call
Base* ptr = new Derived();
ptr->func();  

// Becomes (conceptually):
// (*(ptr->vptr[0]))();  // vptr[0] points to func
```

**Cost of Virtual Functions:**
```cpp
#include <chrono>
#include <iostream>

class NonVirtual {
public:
    int compute(int x) { return x * 2; }
};

class Virtual {
public:
    virtual int compute(int x) { return x * 2; }
    virtual ~Virtual() = default;
};

int main() {
    using namespace std::chrono;
    
    NonVirtual nv;
    Virtual v;
    int result = 0;
    
    // Non-virtual calls (can be inlined)
    auto start = high_resolution_clock::now();
    for (int i = 0; i < 100000000; ++i) {
        result += nv.compute(i);
    }
    auto end = high_resolution_clock::now();
    std::cout << "Non-virtual: " 
              << duration_cast<milliseconds>(end - start).count() << "ms\n";
    
    // Virtual calls (cannot be inlined, indirect)
    start = high_resolution_clock::now();
    for (int i = 0; i < 100000000; ++i) {
        result += v.compute(i);
    }
    end = high_resolution_clock::now();
    std::cout << "Virtual: " 
              << duration_cast<milliseconds>(end - start).count() << "ms\n";
    
    return 0;
}
```

### Abstract Classes and Pure Virtual Functions

```cpp
#include <iostream>
#include <vector>
#include <memory>

// Abstract base class (interface)
class Shape {
public:
    virtual ~Shape() = default;
    
    // Pure virtual functions
    virtual double area() const = 0;
    virtual double perimeter() const = 0;
    virtual void draw() const = 0;
    
    // Can have non-virtual functions
    void print_info() const {
        std::cout << "Area: " << area() << ", Perimeter: " << perimeter() << "\n";
    }
};

class Circle : public Shape {
    double radius;
    
public:
    Circle(double r) : radius(r) {}
    
    double area() const override {
        return 3.14159 * radius * radius;
    }
    
    double perimeter() const override {
        return 2 * 3.14159 * radius;
    }
    
    void draw() const override {
        std::cout << "Drawing circle with radius " << radius << "\n";
    }
};

class Rectangle : public Shape {
    double width, height;
    
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double area() const override {
        return width * height;
    }
    
    double perimeter() const override {
        return 2 * (width + height);
    }
    
    void draw() const override {
        std::cout << "Drawing rectangle " << width << "x" << height << "\n";
    }
};

int main() {
    // Cannot instantiate abstract class
    // Shape s;  // ❌ Error: cannot instantiate abstract class
    
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5.0));
    shapes.push_back(std::make_unique<Rectangle>(4.0, 6.0));
    
    for (const auto& shape : shapes) {
        shape->draw();
        shape->print_info();
        std::cout << "\n";
    }
    
    return 0;
}
```

### Multiple Inheritance

```cpp
#include <iostream>

class Flyable {
public:
    virtual void fly() {
        std::cout << "Flying\n";
    }
    virtual ~Flyable() = default;
};

class Swimmable {
public:
    virtual void swim() {
        std::cout << "Swimming\n";
    }
    virtual ~Swimmable() = default;
};

// Multiple inheritance
class Duck : public Flyable, public Swimmable {
public:
    void fly() override {
        std::cout << "Duck flies\n";
    }
    
    void swim() override {
        std::cout << "Duck swims\n";
    }
    
    void quack() {
        std::cout << "Quack!\n";
    }
};

int main() {
    Duck duck;
    duck.fly();
    duck.swim();
    duck.quack();
    
    // Can use as either base class
    Flyable* f = &duck;
    f->fly();
    
    Swimmable* s = &duck;
    s->swim();
    
    return 0;
}
```

**Diamond Problem:**
```cpp
#include <iostream>

class Animal {
public:
    void eat() { std::cout << "Animal eats\n"; }
};

// Diamond inheritance hierarchy:
//      Animal
//      /    \
//   Bird    Fish
//      \    /
//      FlyingFish

class Bird : public Animal {
public:
    void fly() { std::cout << "Bird flies\n"; }
};

class Fish : public Animal {
public:
    void swim() { std::cout << "Fish swims\n"; }
};

class FlyingFish : public Bird, public Fish {
public:
    // Problem: inherits two copies of Animal!
    // eat() is ambiguous
};

int main() {
    FlyingFish ff;
    // ff.eat();  // ❌ Error: ambiguous - which Animal::eat()?
    ff.Bird::eat();  // ✅ Specify which base
    ff.Fish::eat();  // ✅ Specify which base
    
    return 0;
}
```

**Virtual Inheritance (Solution to Diamond Problem):**
```cpp
class Animal {
public:
    void eat() { std::cout << "Animal eats\n"; }
    virtual ~Animal() = default;
};

// Virtual inheritance
class Bird : public virtual Animal {
public:
    void fly() { std::cout << "Bird flies\n"; }
};

class Fish : public virtual Animal {
public:
    void swim() { std::cout << "Fish swims\n"; }
};

class FlyingFish : public Bird, public Fish {
public:
    // Now only ONE copy of Animal
};

int main() {
    FlyingFish ff;
    ff.eat();  // ✅ OK: unambiguous (single Animal)
    ff.fly();
    ff.swim();
    
    return 0;
}
```

### Override and Final Keywords (C++11)

```cpp
class Base {
public:
    virtual void func1() {}
    virtual void func2() const {}
    virtual void func3() {}
};

class Derived : public Base {
public:
    // override: ensures we're actually overriding
    void func1() override {}  // ✅ OK
    
    // void func2() override {}  // ❌ Error: not const (not overriding)
    void func2() const override {}  // ✅ OK
    
    // Prevent further overriding
    void func3() final override {}  // Cannot be overridden in further derived classes
};

// final class: cannot be inherited
class FinalClass final {
    // ...
};

// class CannotDerive : public FinalClass {};  // ❌ Error: cannot derive from final class

class FurtherDerived : public Derived {
public:
    // void func3() override {}  // ❌ Error: func3 is final
};
```

### Covariant Return Types

```cpp
class Base {
public:
    virtual Base* clone() const {
        return new Base(*this);
    }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    // Covariant return type: Derived* instead of Base*
    Derived* clone() const override {
        return new Derived(*this);
    }
};

int main() {
    Base* b = new Derived();
    Derived* d = b->clone();  // Returns Derived* (no cast needed)
    
    delete d;
    delete b;
    return 0;
}
```

---

## 2.4 Special Member Functions Deep Dive

### Virtual Destructors

**Why Virtual Destructors Matter:**
```cpp
#include <iostream>

class Base {
public:
    Base() { std::cout << "Base constructor\n"; }
    
    // ❌ BAD: Non-virtual destructor
    ~Base() { std::cout << "Base destructor\n"; }
};

class Derived : public Base {
    int* data;
    
public:
    Derived() : data(new int[100]) {
        std::cout << "Derived constructor\n";
    }
    
    ~Derived() {
        std::cout << "Derived destructor\n";
        delete[] data;  // Clean up
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr;  // ⚠️ Only calls Base::~Base()! Memory leak!
    // Derived destructor never called: data not freed!
    
    return 0;
}
```

**Correct Implementation:**
```cpp
class Base {
public:
    Base() { std::cout << "Base constructor\n"; }
    
    // ✅ GOOD: Virtual destructor
    virtual ~Base() { std::cout << "Base destructor\n"; }
};

class Derived : public Base {
    int* data;
    
public:
    Derived() : data(new int[100]) {
        std::cout << "Derived constructor\n";
    }
    
    ~Derived() override {  // override optional but recommended
        std::cout << "Derived destructor\n";
        delete[] data;
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr;  // ✅ Calls Derived::~Derived(), then Base::~Base()
    // Proper cleanup!
    
    return 0;
}
```

**Rule:** Always make destructors virtual in base classes intended for polymorphic use.

### Constructor/Destructor Calling Virtual Functions

```cpp
#include <iostream>

class Base {
public:
    Base() {
        std::cout << "Base constructor calling virtual func\n";
        func();  // ⚠️ Calls Base::func(), NOT Derived::func()!
    }
    
    virtual ~Base() {
        std::cout << "Base destructor calling virtual func\n";
        func();  // ⚠️ Calls Base::func()
    }
    
    virtual void func() {
        std::cout << "Base::func()\n";
    }
};

class Derived : public Base {
public:
    Derived() {
        std::cout << "Derived constructor\n";
    }
    
    ~Derived() {
        std::cout << "Derived destructor\n";
    }
    
    void func() override {
        std::cout << "Derived::func()\n";
    }
};

int main() {
    std::cout << "=== Construction ===\n";
    Derived d;
    
    std::cout << "\n=== Normal call ===\n";
    d.func();  // Calls Derived::func()
    
    std::cout << "\n=== Destruction ===\n";
    return 0;
}
```

**Output:**
```
=== Construction ===
Base constructor calling virtual func
Base::func()           ← NOT Derived::func()!
Derived constructor

=== Normal call ===
Derived::func()

=== Destruction ===
Derived destructor
Base destructor calling virtual func
Base::func()           ← NOT Derived::func()!
```

**Why:** During construction, Derived part doesn't exist yet. During destruction, Derived part already destroyed.

---

## 2.5 Operator Overloading

### Arithmetic Operators

```cpp
#include <iostream>

class Complex {
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Member function overload
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    Complex operator-(const Complex& other) const {
        return Complex(real - other.real, imag - other.imag);
    }
    
    Complex operator*(const Complex& other) const {
        return Complex(
            real * other.real - imag * other.imag,
            real * other.imag + imag * other.real
        );
    }
    
    // Unary minus
    Complex operator-() const {
        return Complex(-real, -imag);
    }
    
    void print() const {
        std::cout << real << " + " << imag << "i\n";
    }
};

int main() {
    Complex a(3, 4);
    Complex b(1, 2);
    
    Complex sum = a + b;
    Complex diff = a - b;
    Complex product = a * b;
    Complex neg = -a;
    
    sum.print();      // 4 + 6i
    diff.print();     // 2 + 2i
    product.print();  // -5 + 10i
    neg.print();      // -3 + -4i
    
    return 0;
}
```

### Comparison Operators

**Traditional Approach (Pre-C++20):**
```cpp
class Version {
    int major, minor, patch;
    
public:
    Version(int maj, int min, int pat) 
        : major(maj), minor(min), patch(pat) {}
    
    // Need to define all six comparison operators
    bool operator==(const Version& other) const {
        return major == other.major && 
               minor == other.minor && 
               patch == other.patch;
    }
    
    bool operator!=(const Version& other) const {
        return !(*this == other);
    }
    
    bool operator<(const Version& other) const {
        if (major != other.major) return major < other.major;
        if (minor != other.minor) return minor < other.minor;
        return patch < other.patch;
    }
    
    bool operator>(const Version& other) const {
        return other < *this;
    }
    
    bool operator<=(const Version& other) const {
        return !(other < *this);
    }
    
    bool operator>=(const Version& other) const {
        return !(*this < other);
    }
};
```

**C++20 Spaceship Operator (<=>):**
```cpp
#include <compare>

class Version {
    int major, minor, patch;
    
public:
    Version(int maj, int min, int pat) 
        : major(maj), minor(min), patch(pat) {}
    
    // One operator generates all six comparison operators!
    auto operator<=>(const Version& other) const = default;
    
    // Also need equality (can be defaulted)
    bool operator==(const Version& other) const = default;
};

int main() {
    Version v1(1, 2, 3);
    Version v2(1, 2, 4);
    
    bool eq = (v1 == v2);   // false
    bool ne = (v1 != v2);   // true
    bool lt = (v1 < v2);    // true
    bool gt = (v1 > v2);    // false
    bool le = (v1 <= v2);   // true
    bool ge = (v1 >= v2);   // false
    
    return 0;
}
```

### Stream Insertion and Extraction Operators

```cpp
#include <iostream>

class Point {
    int x, y;
    
public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}
    
    // Friend functions for stream operators
    friend std::ostream& operator<<(std::ostream& os, const Point& p);
    friend std::istream& operator>>(std::istream& is, Point& p);
};

// Output operator
std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";
    return os;  // Enable chaining
}

// Input operator
std::istream& operator>>(std::istream& is, Point& p) {
    char lparen, comma, rparen;
    is >> lparen >> p.x >> comma >> p.y >> rparen;
    
    if (lparen != '(' || comma != ',' || rparen != ')') {
        is.setstate(std::ios::failbit);
    }
    
    return is;
}

int main() {
    Point p1(3, 4);
    std::cout << "p1 = " << p1 << "\n";  // Output: p1 = (3, 4)
    
    Point p2;
    std::cout << "Enter point (x, y): ";
    std::cin >> p2;
    std::cout << "p2 = " << p2 << "\n";
    
    return 0;
}
```

### Subscript Operator

```cpp
#include <iostream>
#include <stdexcept>

class Array {
    int* data;
    size_t size;
    
public:
    Array(size_t n) : size(n), data(new int[n]()) {}
    
    ~Array() { delete[] data; }
    
    // Non-const subscript (allows modification)
    int& operator[](size_t index) {
        if (index >= size) {
            throw std::out_of_range("Index out of bounds");
        }
        return data[index];
    }
    
    // Const subscript (read-only)
    const int& operator[](size_t index) const {
        if (index >= size) {
            throw std::out_of_range("Index out of bounds");
        }
        return data[index];
    }
    
    size_t get_size() const { return size; }
};

int main() {
    Array arr(5);
    
    // Using non-const operator[]
    for (size_t i = 0; i < arr.get_size(); ++i) {
        arr[i] = i * 10;
    }
    
    // Using const operator[]
    const Array& carr = arr;
    for (size_t i = 0; i < carr.get_size(); ++i) {
        std::cout << carr[i] << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Function Call Operator

```cpp
#include <iostream>

class Multiplier {
    int factor;
    
public:
    Multiplier(int f) : factor(f) {}
    
    // Function call operator (makes object callable)
    int operator()(int x) const {
        return x * factor;
    }
};

// Lambda equivalent
auto multiplier_lambda = [factor = 2](int x) { return x * factor; };

int main() {
    Multiplier times_three(3);
    
    std::cout << times_three(10) << "\n";  // 30
    std::cout << times_three(5) << "\n";   // 15
    
    // Functors are useful with STL algorithms
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::transform(vec.begin(), vec.end(), vec.begin(), times_three);
    
    for (int val : vec) {
        std::cout << val << " ";  // 3 6 9 12 15
    }
    std::cout << "\n";
    
    return 0;
}
```

### Conversion Operators

```cpp
#include <iostream>
#include <string>

class Fraction {
    int numerator, denominator;
    
public:
    Fraction(int num, int denom = 1) 
        : numerator(num), denominator(denom) {}
    
    // Implicit conversion to double
    operator double() const {
        return static_cast<double>(numerator) / denominator;
    }
    
    // Explicit conversion to string (C++11)
    explicit operator std::string() const {
        return std::to_string(numerator) + "/" + std::to_string(denominator);
    }
    
    // Explicit conversion to bool
    explicit operator bool() const {
        return numerator != 0;
    }
};

int main() {
    Fraction f(3, 4);
    
    // Implicit conversion to double
    double d = f;  // OK: implicit
    std::cout << "As double: " << d << "\n";
    
    // Explicit conversion to string
    // std::string s = f;  // ❌ Error: explicit conversion
    std::string s = static_cast<std::string>(f);  // ✅ OK
    std::cout << "As string: " << s << "\n";
    
    // Explicit conversion to bool (in boolean context)
    if (f) {  // OK: contextual conversion
        std::cout << "Fraction is non-zero\n";
    }
    
    return 0;
}
```

### Member vs Non-Member Operators

**Guidelines:**
- **Member functions:** `=`, `[]`, `()`, `->`
- **Non-member functions:** Symmetric operators (`+`, `-`, `*`, `/`, `==`, `<`, etc.)
- **Friend functions:** When need access to private members

```cpp
class Complex {
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // Member: this is always left operand
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }
    
    // Friend non-member: symmetric, allows implicit conversions on left
    friend Complex operator+(Complex lhs, const Complex& rhs) {
        lhs += rhs;  // Reuse +=
        return lhs;
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {
        os << c.real << " + " << c.imag << "i";
        return os;
    }
};

int main() {
    Complex a(3, 4);
    Complex b(1, 2);
    
    // Both work due to non-member operator+
    Complex c1 = a + b;      // OK
    Complex c2 = 5.0 + a;    // OK: 5.0 converted to Complex(5.0, 0)
    Complex c3 = a + 5.0;    // OK
    
    std::cout << c1 << "\n";
    std::cout << c2 << "\n";
    std::cout << c3 << "\n";
    
    return 0;
}
```

---

## 2.6 Static Members and Friend Functions

### Static Member Variables

```cpp
#include <iostream>

class Counter {
    static int count;  // Declaration
    int id;
    
public:
    Counter() : id(++count) {
        std::cout << "Counter " << id << " created (total: " << count << ")\n";
    }
    
    ~Counter() {
        --count;
        std::cout << "Counter " << id << " destroyed (remaining: " << count << ")\n";
    }
    
    static int get_count() {  // Static member function
        return count;
    }
    
    int get_id() const { return id; }
};

// Definition (must be outside class)
int Counter::count = 0;

int main() {
    std::cout << "Initial count: " << Counter::get_count() << "\n\n";
    
    {
        Counter c1;
        Counter c2;
        std::cout << "Count: " << Counter::get_count() << "\n\n";
        
        {
            Counter c3;
            std::cout << "Count: " << Counter::get_count() << "\n\n";
        }
        
        std::cout << "After c3 destroyed: " << Counter::get_count() << "\n\n";
    }
    
    std::cout << "Final count: " << Counter::get_count() << "\n";
    
    return 0;
}
```

### Static Member Functions

```cpp
class Math {
public:
    // Static functions don't have 'this' pointer
    static int add(int a, int b) {
        return a + b;
    }
    
    static int multiply(int a, int b) {
        return a * b;
    }
    
    // Cannot access non-static members
    // static void invalid() {
    //     data = 10;  // ❌ Error: no 'this' pointer
    // }
    
private:
    int data;  // Non-static member
};

int main() {
    // Call without object instance
    int sum = Math::add(5, 3);
    int product = Math::multiply(4, 7);
    
    std::cout << "Sum: " << sum << "\n";
    std::cout << "Product: " << product << "\n";
    
    return 0;
}
```

### Singleton Pattern with Static Members

```cpp
#include <iostream>
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mutex_;
    
    // Private constructor
    Singleton() {
        std::cout << "Singleton created\n";
    }
    
    // Delete copy operations
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
public:
    // Thread-safe lazy initialization
    static Singleton* get_instance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mutex_);
            if (instance == nullptr) {  // Double-checked locking
                instance = new Singleton();
            }
        }
        return instance;
    }
    
    void do_something() {
        std::cout << "Singleton doing something\n";
    }
};

// Definition
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex_;

// Modern C++11 alternative: Meyer's Singleton (thread-safe by standard)
class ModernSingleton {
private:
    ModernSingleton() {
        std::cout << "ModernSingleton created\n";
    }
    
public:
    static ModernSingleton& get_instance() {
        static ModernSingleton instance;  // Thread-safe in C++11
        return instance;
    }
    
    // Delete copy operations
    ModernSingleton(const ModernSingleton&) = delete;
    ModernSingleton& operator=(const ModernSingleton&) = delete;
    
    void do_something() {
        std::cout << "ModernSingleton doing something\n";
    }
};

int main() {
    // Old style
    Singleton* s1 = Singleton::get_instance();
    Singleton* s2 = Singleton::get_instance();
    std::cout << "Same instance? " << (s1 == s2) << "\n\n";
    
    // Modern style (preferred)
    ModernSingleton& ms1 = ModernSingleton::get_instance();
    ModernSingleton& ms2 = ModernSingleton::get_instance();
    std::cout << "Same instance? " << (&ms1 == &ms2) << "\n";
    
    return 0;
}
```

### Friend Functions

```cpp
#include <iostream>

class Box {
private:
    double width, height, depth;
    
public:
    Box(double w, double h, double d) : width(w), height(h), depth(d) {}
    
    // Friend function declaration
    friend double get_volume(const Box& box);
    friend class BoxPrinter;  // Friend class
};

// Friend function definition (can access private members)
double get_volume(const Box& box) {
    return box.width * box.height * box.depth;
}

// Friend class (can access private members)
class BoxPrinter {
public:
    static void print(const Box& box) {
        std::cout << "Dimensions: " << box.width << " x " 
                  << box.height << " x " << box.depth << "\n";
    }
};

int main() {
    Box box(3.0, 4.0, 5.0);
    
    std::cout << "Volume: " << get_volume(box) << "\n";
    BoxPrinter::print(box);
    
    return 0;
}
```

**When to Use Friends:**
```cpp
// ✅ GOOD use: Stream operators
class Complex {
    double real, imag;
    
public:
    Complex(double r, double i) : real(r), imag(i) {}
    
    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {
        os << c.real << " + " << c.imag << "i";
        return os;
    }
};

// ✅ GOOD use: Symmetric operators
class Fraction {
    int num, den;
    
public:
    Fraction(int n, int d) : num(n), den(d) {}
    
    friend Fraction operator+(const Fraction& lhs, const Fraction& rhs) {
        return Fraction(lhs.num * rhs.den + rhs.num * lhs.den,
                       lhs.den * rhs.den);
    }
};

// ❌ BAD use: Breaking encapsulation
class BadEncapsulation {
private:
    int secret_data;
    
    // Too many friends = poor encapsulation
    friend class Friend1;
    friend class Friend2;
    friend class Friend3;
    // ... many friends accessing private data
};

// ✅ BETTER: Provide proper interface
class GoodEncapsulation {
private:
    int data;
    
public:
    int get_data() const { return data; }
    void set_data(int d) { data = d; }
    // Controlled access through interface
};
```

---

## Comprehensive Example: Putting It All Together

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>

// Abstract base class
class Employee {
protected:
    std::string name;
    int id;
    static int next_id;
    
public:
    Employee(const std::string& n) : name(n), id(next_id++) {}
    
    virtual ~Employee() = default;
    
    // Pure virtual functions
    virtual double calculate_salary() const = 0;
    virtual void print_details() const = 0;
    
    // Common interface
    int get_id() const { return id; }
    const std::string& get_name() const { return name; }
    
    // Static member function
    static int get_employee_count() { return next_id; }
};

int Employee::next_id = 1000;

// Derived class: Hourly worker
class HourlyEmployee : public Employee {
    double hourly_rate;
    double hours_worked;
    
public:
    HourlyEmployee(const std::string& n, double rate) 
        : Employee(n), hourly_rate(rate), hours_worked(0) {}
    
    void add_hours(double hours) {
        hours_worked += hours;
    }
    
    double calculate_salary() const override {
        return hourly_rate * hours_worked;
    }
    
    void print_details() const override {
        std::cout << "Hourly Employee: " << name << " (ID: " << id << ")\n";
        std::cout << "  Rate: $" << hourly_rate << "/hour\n";
        std::cout << "  Hours: " << hours_worked << "\n";
        std::cout << "  Salary: $" << calculate_salary() << "\n";
    }
};

// Derived class: Salaried worker
class SalariedEmployee : public Employee {
    double annual_salary;
    
public:
    SalariedEmployee(const std::string& n, double salary)
        : Employee(n), annual_salary(salary) {}
    
    double calculate_salary() const override {
        return annual_salary / 12;  // Monthly salary
    }
    
    void print_details() const override {
        std::cout << "Salaried Employee: " << name << " (ID: " << id << ")\n";
        std::cout << "  Annual Salary: $" << annual_salary << "\n";
        std::cout << "  Monthly Salary: $" << calculate_salary() << "\n";
    }
};

// Derived class: Manager (multiple responsibilities)
class Manager : public SalariedEmployee {
    std::vector<Employee*> team;
    double bonus_rate;
    
public:
    Manager(const std::string& n, double salary, double bonus)
        : SalariedEmployee(n, salary), bonus_rate(bonus) {}
    
    void add_team_member(Employee* emp) {
        team.push_back(emp);
    }
    
    double calculate_salary() const override {
        double base = SalariedEmployee::calculate_salary();
        double team_bonus = team.size() * bonus_rate;
        return base + team_bonus;
    }
    
    void print_details() const override {
        std::cout << "Manager: " << name << " (ID: " << id << ")\n";
        std::cout << "  Base Monthly: $" << SalariedEmployee::calculate_salary() << "\n";
        std::cout << "  Team Size: " << team.size() << "\n";
        std::cout << "  Total Salary: $" << calculate_salary() << "\n";
    }
};

int main() {
    std::vector<std::unique_ptr<Employee>> employees;
    
    // Create employees
    auto hourly = std::make_unique<HourlyEmployee>("Alice", 25.0);
    hourly->add_hours(40);
    employees.push_back(std::move(hourly));
    
    employees.push_back(std::make_unique<SalariedEmployee>("Bob", 60000));
    
    auto manager = std::make_unique<Manager>("Charlie", 80000, 500);
    employees.push_back(std::move(manager));
    
    // Polymorphic iteration
    std::cout << "=== Employee Details ===\n\n";
    double total_payroll = 0;
    
    for (const auto& emp : employees) {
        emp->print_details();
        total_payroll += emp->calculate_salary();
        std::cout << "\n";
    }
    
    std::cout << "Total Monthly Payroll: $" << total_payroll << "\n";
    std::cout << "Total Employees: " << Employee::get_employee_count() - 1000 << "\n";
    
    return 0;
}
```

---

## Conclusion

Part 2 has covered the essential aspects of Object-Oriented Programming in C++ and the underlying object model:

1. **Classes and Objects** - Encapsulation, constructors, destructors, RAII
2. **The C++ Object Model** - Memory layout, padding, alignment, object lifetime
3. **Inheritance and Polymorphism** - Single, multiple, virtual inheritance, vtables
4. **Special Member Functions** - Rule of Zero/Three/Five, virtual destructors
5. **Operator Overloading** - Custom operators, member vs non-member
6. **Static Members and Friends** - Class-level data and controlled access

These OOP concepts form the backbone of modern C++ design. Understanding how the compiler implements these features—from memory layout to virtual function dispatch—enables you to write efficient, maintainable code.

**Key Principles to Remember:**
- Prefer composition over inheritance
- Use RAII for resource management
- Follow the Rule of Zero when possible
- Make destructors virtual in polymorphic base classes
- Use smart pointers for dynamic polymorphism
- Understand the cost of virtual functions

**Next in the series:** Part 3 will explore Memory Management & Resource Handling in depth, covering dynamic allocation, smart pointers, custom allocators, and advanced RAII techniques.

---

## References and Further Reading

- **C++ Core Guidelines**: https://isocpp.github.io/CppCoreGuidelines/
- **"Effective C++"** by Scott Meyers - Essential OOP patterns
- **"More Effective C++"** by Scott Meyers - Advanced OOP techniques
- **"Inside the C++ Object Model"** by Stanley Lippman - Deep dive into implementation
- **cppreference.com** - Comprehensive language reference

---

*This blog post is part of the Complete C++ Mastery series. Continue to Part 3 for in-depth coverage of Memory Management & Resource Handling.*
