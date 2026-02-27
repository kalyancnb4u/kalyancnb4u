---
title: "Complete C++ Mastery Part 3: Memory Management & Resource Handling"
date: 2024-02-20 14:00:00 +0000
categories: [C++, Programming, Memory]
tags: [cpp, memory-management, smart-pointers, raii, heap, stack, allocators, resource-handling]
---

# Complete C++ Mastery Part 3: Memory Management & Resource Handling

## Introduction

Welcome to Part 3 of the Complete C++ Mastery series. Memory management is one of the most critical and challenging aspects of C++ programming. Unlike languages with garbage collection, C++ gives you direct control over memory allocation and deallocation—with great power comes great responsibility.

This part explores the full spectrum of memory management in C++: from understanding the stack and heap, to using smart pointers, to implementing custom allocators. We'll dive deep into RAII (Resource Acquisition Is Initialization), examine common memory errors, and learn modern C++ techniques for safe, efficient resource handling.

**Prerequisites:** This part assumes knowledge from Parts 1 and 2, including classes, constructors/destructors, pointers, and references.

---

## 3.1 Dynamic Memory Allocation

### Stack vs Heap Memory

Understanding where objects live is fundamental to C++ memory management.

**Stack Memory:**
```cpp
#include <iostream>

void stack_demo() {
    int x = 42;           // Allocated on stack
    int arr[100];         // Array on stack
    std::string s = "hi"; // String object on stack (data may be on heap)
    
    std::cout << "Address of x: " << &x << "\n";
    std::cout << "Address of arr: " << arr << "\n";
    std::cout << "Address of s: " << &s << "\n";
    
    // All automatically deallocated when function returns
}

int main() {
    stack_demo();
    // x, arr, s no longer exist here
    
    return 0;
}
```

**Stack Characteristics:**
- **Fast allocation**: Just moving stack pointer
- **Automatic deallocation**: LIFO (Last In, First Out)
- **Limited size**: Typically 1-8 MB (system-dependent)
- **Lifetime**: Scope-based
- **Cache-friendly**: Sequential memory access

**Heap Memory:**
```cpp
#include <iostream>

void heap_demo() {
    int* p = new int(42);        // Allocated on heap
    int* arr = new int[100];     // Array on heap
    std::string* s = new std::string("hello");
    
    std::cout << "Value: " << *p << "\n";
    std::cout << "Address of p: " << &p << "\n";      // Pointer on stack
    std::cout << "Points to: " << p << "\n";          // Memory on heap
    
    // Manual deallocation required
    delete p;
    delete[] arr;
    delete s;
}

int main() {
    heap_demo();
    return 0;
}
```

**Heap Characteristics:**
- **Flexible size**: Limited by available RAM
- **Manual management**: Must explicitly deallocate
- **Slower allocation**: Involves memory manager
- **Flexible lifetime**: Controlled by programmer
- **Potential fragmentation**: Non-contiguous allocations

**Memory Layout:**
```
High addresses
+------------------+
|      Stack       | ← Grows downward
|        ↓         |
+------------------+
|                  |
|   Free Space     |
|                  |
+------------------+
|        ↑         |
|      Heap        | ← Grows upward
+------------------+
|   Static Data    |
+------------------+
|   Text (Code)    |
+------------------+
Low addresses
```

### New and Delete Operators

**Basic Usage:**
```cpp
#include <iostream>

int main() {
    // Single object
    int* p = new int;           // Uninitialized
    int* q = new int(42);       // Initialized to 42
    int* r = new int{100};      // C++11 uniform initialization
    
    std::cout << "*q = " << *q << "\n";
    
    delete p;
    delete q;
    delete r;
    
    // Array allocation
    int* arr = new int[10];     // Array of 10 ints
    arr[0] = 5;
    arr[9] = 15;
    
    delete[] arr;  // ⚠️ Must use delete[] for arrays!
    
    return 0;
}
```

**Common Mistakes:**
```cpp
// ❌ BAD: Memory leak
void leak() {
    int* p = new int(42);
    // Forgot to delete p!
}  // Memory leaked

// ❌ BAD: Double delete
void double_delete() {
    int* p = new int(42);
    delete p;
    delete p;  // ⚠️ Undefined behavior!
}

// ❌ BAD: Delete vs delete[] mismatch
void mismatch() {
    int* arr = new int[10];
    delete arr;  // ⚠️ Should be delete[]! Undefined behavior!
}

// ❌ BAD: Use after delete
void use_after_delete() {
    int* p = new int(42);
    delete p;
    std::cout << *p << "\n";  // ⚠️ Undefined behavior!
}

// ❌ BAD: Dangling pointer
int* dangling() {
    int* p = new int(42);
    delete p;
    return p;  // ⚠️ Returns pointer to deleted memory!
}
```

**Correct Pattern:**
```cpp
// ✅ GOOD: RAII wrapper
class IntHolder {
    int* data;
public:
    IntHolder(int value) : data(new int(value)) {}
    
    ~IntHolder() {
        delete data;  // Automatic cleanup
    }
    
    int get() const { return *data; }
    void set(int value) { *data = value; }
};

void correct_usage() {
    IntHolder holder(42);
    std::cout << holder.get() << "\n";
    // No manual delete needed!
}
```

### Placement New

Placement new constructs objects at a specific memory location:

```cpp
#include <iostream>
#include <new>  // For placement new

class Widget {
    int data;
public:
    Widget(int d) : data(d) {
        std::cout << "Widget(" << data << ") constructed\n";
    }
    
    ~Widget() {
        std::cout << "Widget(" << data << ") destroyed\n";
    }
    
    void show() const {
        std::cout << "Widget data: " << data << "\n";
    }
};

int main() {
    // Allocate raw memory
    char buffer[sizeof(Widget)];
    
    // Construct object at specific location
    Widget* w = new (buffer) Widget(42);  // Placement new
    
    w->show();
    
    // Must manually call destructor (no delete!)
    w->~Widget();
    
    // Memory (buffer) automatically deallocated at scope end
    
    return 0;
}
```

**Use Cases for Placement New:**
- Memory pools
- Custom allocators
- Embedded systems with fixed memory
- Constructing objects in shared memory

### Memory Allocation Failures

```cpp
#include <iostream>
#include <new>

void handle_allocation_failure() {
    try {
        // Try to allocate huge amount
        int* huge = new int[1000000000000];  // Likely fails
        delete[] huge;
    } catch (const std::bad_alloc& e) {
        std::cerr << "Allocation failed: " << e.what() << "\n";
    }
}

// Nothrow version (returns nullptr instead of throwing)
void nothrow_allocation() {
    int* p = new (std::nothrow) int[1000000000000];
    
    if (p == nullptr) {
        std::cerr << "Allocation failed\n";
        return;
    }
    
    delete[] p;
}

int main() {
    handle_allocation_failure();
    nothrow_allocation();
    
    return 0;
}
```

### Custom New/Delete Operators

```cpp
#include <iostream>
#include <cstdlib>

// Global custom new/delete
void* operator new(size_t size) {
    std::cout << "Custom new: " << size << " bytes\n";
    void* p = std::malloc(size);
    if (!p) throw std::bad_alloc();
    return p;
}

void operator delete(void* p) noexcept {
    std::cout << "Custom delete\n";
    std::free(p);
}

// Class-specific custom new/delete
class Tracked {
    int data;
    
public:
    Tracked(int d) : data(d) {
        std::cout << "Tracked(" << data << ") constructed\n";
    }
    
    ~Tracked() {
        std::cout << "Tracked(" << data << ") destroyed\n";
    }
    
    // Class-specific new
    void* operator new(size_t size) {
        std::cout << "Tracked::operator new(" << size << ")\n";
        return ::operator new(size);  // Use global new
    }
    
    // Class-specific delete
    void operator delete(void* p) noexcept {
        std::cout << "Tracked::operator delete\n";
        ::operator delete(p);  // Use global delete
    }
};

int main() {
    std::cout << "=== Global new/delete ===\n";
    int* p = new int(42);
    delete p;
    
    std::cout << "\n=== Class-specific new/delete ===\n";
    Tracked* t = new Tracked(100);
    delete t;
    
    return 0;
}
```

---

## 3.2 Pointers and References

### Pointer Fundamentals

```cpp
#include <iostream>

int main() {
    int x = 42;
    int* p = &x;  // p points to x
    
    std::cout << "x = " << x << "\n";
    std::cout << "Address of x: " << &x << "\n";
    std::cout << "p = " << p << "\n";  // Same as &x
    std::cout << "*p = " << *p << "\n";  // Dereference: 42
    
    // Modify through pointer
    *p = 100;
    std::cout << "x after *p = 100: " << x << "\n";  // 100
    
    // Pointer arithmetic
    int arr[] = {10, 20, 30, 40, 50};
    int* ptr = arr;  // Points to first element
    
    std::cout << "\nArray elements:\n";
    for (int i = 0; i < 5; ++i) {
        std::cout << "ptr[" << i << "] = " << *(ptr + i) << "\n";
    }
    
    return 0;
}
```

### Pointer Arithmetic

```cpp
#include <iostream>

int main() {
    int arr[] = {10, 20, 30, 40, 50};
    int* p = arr;
    
    std::cout << "*p = " << *p << "\n";      // 10
    std::cout << "*(p+1) = " << *(p+1) << "\n";  // 20
    std::cout << "*(p+2) = " << *(p+2) << "\n";  // 30
    
    // Increment pointer
    p++;
    std::cout << "*p after p++: " << *p << "\n";  // 20
    
    // Pointer difference
    int* start = &arr[0];
    int* end = &arr[4];
    std::cout << "Distance: " << (end - start) << " elements\n";  // 4
    
    // Pointer comparison
    if (start < end) {
        std::cout << "start comes before end\n";
    }
    
    return 0;
}
```

### Null Pointers

```cpp
#include <iostream>

int main() {
    // C++98 style (avoid)
    int* p1 = NULL;  // NULL is macro for 0
    int* p2 = 0;
    
    // C++11 style (preferred)
    int* p3 = nullptr;  // Proper null pointer literal
    
    // Check for null
    if (p3 == nullptr) {
        std::cout << "p3 is null\n";
    }
    
    // ⚠️ Dereferencing null pointer is UB
    // *p3 = 42;  // Crash!
    
    // ✅ Always check before dereferencing
    if (p3 != nullptr) {
        *p3 = 42;
    } else {
        std::cout << "Cannot dereference null pointer\n";
    }
    
    return 0;
}
```

**Why nullptr is better than NULL:**
```cpp
void func(int x) {
    std::cout << "func(int): " << x << "\n";
}

void func(int* p) {
    std::cout << "func(int*): " << p << "\n";
}

int main() {
    // func(NULL);  // ⚠️ Ambiguous! NULL is 0 (int)
    func(nullptr);  // ✅ Calls func(int*)
    
    return 0;
}
```

### Pointers to Functions

```cpp
#include <iostream>

int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// Function taking function pointer
int apply_operation(int x, int y, int (*operation)(int, int)) {
    return operation(x, y);
}

int main() {
    // Function pointer declaration
    int (*func_ptr)(int, int);
    
    func_ptr = add;
    std::cout << "add(5, 3) = " << func_ptr(5, 3) << "\n";
    
    func_ptr = multiply;
    std::cout << "multiply(5, 3) = " << func_ptr(5, 3) << "\n";
    
    // Using with higher-order function
    std::cout << "apply add: " << apply_operation(10, 5, add) << "\n";
    std::cout << "apply multiply: " << apply_operation(10, 5, multiply) << "\n";
    
    return 0;
}
```

### Pointers to Members

```cpp
#include <iostream>

class Widget {
public:
    int data;
    
    void show() const {
        std::cout << "data = " << data << "\n";
    }
};

int main() {
    Widget w;
    w.data = 42;
    
    // Pointer to member variable
    int Widget::*p_data = &Widget::data;
    w.*p_data = 100;  // Access through object
    std::cout << "w.data = " << w.data << "\n";
    
    Widget* pw = &w;
    pw->*p_data = 200;  // Access through pointer
    std::cout << "w.data = " << w.data << "\n";
    
    // Pointer to member function
    void (Widget::*p_func)() const = &Widget::show;
    (w.*p_func)();    // Call through object
    (pw->*p_func)();  // Call through pointer
    
    return 0;
}
```

### References

**Lvalue References:**
```cpp
#include <iostream>

void swap_ref(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 10;
    int& ref = x;  // Reference to x
    
    std::cout << "x = " << x << "\n";
    ref = 20;
    std::cout << "x after ref = 20: " << x << "\n";
    
    // References must be initialized
    // int& bad_ref;  // ❌ Error: must be initialized
    
    // References cannot be reseated
    int y = 30;
    ref = y;  // ⚠️ Assigns value of y to x (doesn't change ref)
    std::cout << "x = " << x << ", y = " << y << "\n";
    
    // Using references for efficient parameter passing
    int a = 5, b = 10;
    std::cout << "Before swap: a = " << a << ", b = " << b << "\n";
    swap_ref(a, b);
    std::cout << "After swap: a = " << a << ", b = " << b << "\n";
    
    return 0;
}
```

**Rvalue References (C++11):**
```cpp
#include <iostream>
#include <utility>

void process_lvalue(int& x) {
    std::cout << "Lvalue reference: " << x << "\n";
}

void process_rvalue(int&& x) {
    std::cout << "Rvalue reference: " << x << "\n";
}

int main() {
    int x = 42;
    
    process_lvalue(x);         // OK: x is lvalue
    // process_rvalue(x);      // ❌ Error: x is lvalue
    process_rvalue(10);        // OK: 10 is rvalue
    process_rvalue(std::move(x));  // OK: std::move casts to rvalue
    
    return 0;
}
```

### Reference Collapsing Rules

```cpp
// C++11 reference collapsing rules:
// & &   → &
// & &&  → &
// && &  → &
// && && → &&

template<typename T>
void func(T&& param) {  // Universal/forwarding reference
    // If T = int&:   int& &&  → int&   (lvalue ref)
    // If T = int:    int&&    → int&&  (rvalue ref)
}

int main() {
    int x = 42;
    func(x);   // T deduced as int&, param is int&
    func(10);  // T deduced as int, param is int&&
    
    return 0;
}
```

### Dangling Pointers and References

```cpp
#include <iostream>

// ❌ BAD: Returning reference to local
int& bad_function() {
    int local = 42;
    return local;  // ⚠️ Dangling reference!
}

// ❌ BAD: Pointer to deleted memory
int* dangling_pointer() {
    int* p = new int(42);
    delete p;
    return p;  // ⚠️ Dangling pointer!
}

// ✅ GOOD: Return by value
int good_function() {
    int local = 42;
    return local;  // Copy returned
}

// ✅ GOOD: Return reference to member
class Container {
    int data;
public:
    Container(int d) : data(d) {}
    int& get_data() { return data; }  // OK: data lives as long as Container
};

int main() {
    // auto& bad = bad_function();  // ⚠️ UB
    // std::cout << bad << "\n";
    
    int good = good_function();
    std::cout << "good = " << good << "\n";
    
    Container c(100);
    int& ref = c.get_data();
    std::cout << "ref = " << ref << "\n";
    
    return 0;
}
```

### Pointer Safety Best Practices

```cpp
#include <iostream>

class SafePointer {
    int* ptr;
    
public:
    SafePointer(int value) : ptr(new int(value)) {}
    
    ~SafePointer() {
        delete ptr;
        ptr = nullptr;  // ✅ Set to nullptr after delete
    }
    
    int get() const {
        if (ptr == nullptr) {
            throw std::runtime_error("Null pointer access");
        }
        return *ptr;
    }
    
    void set(int value) {
        if (ptr == nullptr) {
            throw std::runtime_error("Null pointer access");
        }
        *ptr = value;
    }
};

int main() {
    SafePointer sp(42);
    std::cout << "Value: " << sp.get() << "\n";
    
    sp.set(100);
    std::cout << "Value: " << sp.get() << "\n";
    
    return 0;
}
```

---

## 3.3 RAII and Smart Pointers

### RAII (Resource Acquisition Is Initialization)

The fundamental principle: tie resource lifetime to object lifetime.

```cpp
#include <iostream>
#include <fstream>
#include <mutex>

// ❌ BAD: Manual resource management
void bad_file_handling() {
    std::ofstream file("output.txt");
    
    if (!file.is_open()) {
        return;  // ⚠️ File not explicitly closed
    }
    
    file << "Hello\n";
    
    // What if exception thrown here?
    
    file.close();  // May never be reached!
}

// ✅ GOOD: RAII automatically manages resource
void good_file_handling() {
    std::ofstream file("output.txt");
    
    if (!file.is_open()) {
        return;  // File automatically closed by destructor
    }
    
    file << "Hello\n";
    
    // File automatically closed at scope end
    // Even if exception thrown!
}

// RAII for mutex
std::mutex mtx;

void protected_function() {
    std::lock_guard<std::mutex> lock(mtx);  // RAII: acquires lock
    
    // Critical section
    std::cout << "Protected code\n";
    
    // Lock automatically released at scope end
}
```

**Custom RAII Wrapper:**
```cpp
#include <iostream>

class FileHandle {
    FILE* file;
    
public:
    explicit FileHandle(const char* filename, const char* mode) 
        : file(std::fopen(filename, mode)) {
        if (!file) {
            throw std::runtime_error("Cannot open file");
        }
        std::cout << "File opened\n";
    }
    
    ~FileHandle() {
        if (file) {
            std::fclose(file);
            std::cout << "File closed\n";
        }
    }
    
    // Delete copy operations
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    
    // Allow move operations
    FileHandle(FileHandle&& other) noexcept : file(other.file) {
        other.file = nullptr;
    }
    
    FileHandle& operator=(FileHandle&& other) noexcept {
        if (this != &other) {
            if (file) {
                std::fclose(file);
            }
            file = other.file;
            other.file = nullptr;
        }
        return *this;
    }
    
    FILE* get() const { return file; }
};

int main() {
    try {
        FileHandle file("test.txt", "w");
        std::fprintf(file.get(), "Hello, RAII!\n");
        
        // File automatically closed when file goes out of scope
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << "\n";
    }
    
    return 0;
}
```

### std::unique_ptr - Exclusive Ownership

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Widget {
    int id;
    
public:
    Widget(int i) : id(i) {
        std::cout << "Widget " << id << " constructed\n";
    }
    
    ~Widget() {
        std::cout << "Widget " << id << " destroyed\n";
    }
    
    void show() const {
        std::cout << "Widget " << id << "\n";
    }
};

int main() {
    std::cout << "=== Creating unique_ptr ===\n";
    std::unique_ptr<Widget> ptr1(new Widget(1));
    
    // ✅ Better: use make_unique (C++14)
    auto ptr2 = std::make_unique<Widget>(2);
    
    ptr1->show();
    ptr2->show();
    
    // Cannot copy unique_ptr
    // std::unique_ptr<Widget> ptr3 = ptr1;  // ❌ Error
    
    // Can move unique_ptr (transfers ownership)
    std::cout << "\n=== Moving unique_ptr ===\n";
    std::unique_ptr<Widget> ptr3 = std::move(ptr1);
    
    if (ptr1 == nullptr) {
        std::cout << "ptr1 is now null\n";
    }
    
    ptr3->show();
    
    // Unique_ptr with arrays
    std::cout << "\n=== unique_ptr with arrays ===\n";
    std::unique_ptr<int[]> arr(new int[5]{1, 2, 3, 4, 5});
    
    // Or with make_unique (C++14)
    auto arr2 = std::make_unique<int[]>(5);
    
    for (int i = 0; i < 5; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << "\n";
    
    // Using with containers
    std::cout << "\n=== unique_ptr in vector ===\n";
    std::vector<std::unique_ptr<Widget>> widgets;
    widgets.push_back(std::make_unique<Widget>(10));
    widgets.push_back(std::make_unique<Widget>(20));
    
    for (const auto& w : widgets) {
        w->show();
    }
    
    std::cout << "\n=== End of main ===\n";
    return 0;
}
```

**Output:**
```
=== Creating unique_ptr ===
Widget 1 constructed
Widget 2 constructed
Widget 1
Widget 2

=== Moving unique_ptr ===
ptr1 is now null
Widget 1

=== unique_ptr with arrays ===
1 2 3 4 5 

=== unique_ptr in vector ===
Widget 10 constructed
Widget 20 constructed
Widget 10
Widget 20

=== End of main ===
Widget 20 destroyed
Widget 10 destroyed
Widget 2 destroyed
Widget 1 destroyed
```

### std::shared_ptr - Shared Ownership

```cpp
#include <iostream>
#include <memory>

class Resource {
    int id;
    
public:
    Resource(int i) : id(i) {
        std::cout << "Resource " << id << " created\n";
    }
    
    ~Resource() {
        std::cout << "Resource " << id << " destroyed\n";
    }
    
    void show() const {
        std::cout << "Resource " << id << "\n";
    }
};

int main() {
    std::cout << "=== Creating shared_ptr ===\n";
    
    // Create shared_ptr
    std::shared_ptr<Resource> sp1 = std::make_shared<Resource>(1);
    std::cout << "Reference count: " << sp1.use_count() << "\n";
    
    {
        // Copy shared_ptr (increments reference count)
        std::shared_ptr<Resource> sp2 = sp1;
        std::cout << "Reference count: " << sp1.use_count() << "\n";
        
        sp1->show();
        sp2->show();
        
        // Create another shared_ptr
        std::shared_ptr<Resource> sp3 = sp1;
        std::cout << "Reference count: " << sp1.use_count() << "\n";
        
    }  // sp2 and sp3 destroyed, ref count decremented
    
    std::cout << "Reference count after block: " << sp1.use_count() << "\n";
    
    std::cout << "\n=== End of main ===\n";
    return 0;
}
```

**Output:**
```
=== Creating shared_ptr ===
Resource 1 created
Reference count: 1
Reference count: 2
Resource 1
Resource 1
Reference count: 3
Reference count after block: 1

=== End of main ===
Resource 1 destroyed
```

**Reference Counting Internals:**
```cpp
#include <iostream>
#include <memory>

class Tracked {
    int id;
    
public:
    Tracked(int i) : id(i) {
        std::cout << "Tracked " << id << " constructed\n";
    }
    
    ~Tracked() {
        std::cout << "Tracked " << id << " destroyed\n";
    }
    
    void show() const {
        std::cout << "Tracked " << id << "\n";
    }
};

int main() {
    auto sp1 = std::make_shared<Tracked>(1);
    
    std::cout << "sp1 use_count: " << sp1.use_count() << "\n";
    std::cout << "sp1.get(): " << sp1.get() << "\n";
    
    auto sp2 = sp1;  // Copy: share ownership
    std::cout << "After copy - sp1 use_count: " << sp1.use_count() << "\n";
    std::cout << "sp2.get(): " << sp2.get() << "\n";  // Same address
    
    auto sp3 = std::move(sp1);  // Move: transfer ownership
    std::cout << "After move - sp3 use_count: " << sp3.use_count() << "\n";
    std::cout << "sp1 use_count: " << sp1.use_count() << "\n";  // 0
    
    if (sp1 == nullptr) {
        std::cout << "sp1 is null after move\n";
    }
    
    return 0;
}
```

### std::weak_ptr - Breaking Circular References

```cpp
#include <iostream>
#include <memory>

class Node {
public:
    int data;
    std::shared_ptr<Node> next;  // Strong reference
    std::weak_ptr<Node> prev;    // Weak reference (breaks cycle)
    
    Node(int d) : data(d) {
        std::cout << "Node " << data << " created\n";
    }
    
    ~Node() {
        std::cout << "Node " << data << " destroyed\n";
    }
};

int main() {
    std::cout << "=== Creating doubly-linked list ===\n";
    
    auto node1 = std::make_shared<Node>(1);
    auto node2 = std::make_shared<Node>(2);
    
    // Create circular reference (with weak_ptr)
    node1->next = node2;
    node2->prev = node1;  // Weak reference breaks cycle
    
    std::cout << "node1 use_count: " << node1.use_count() << "\n";  // 1
    std::cout << "node2 use_count: " << node2.use_count() << "\n";  // 2
    
    // Using weak_ptr
    if (auto prev = node2->prev.lock()) {  // Convert to shared_ptr
        std::cout << "node2's prev data: " << prev->data << "\n";
    }
    
    std::cout << "\n=== End of main ===\n";
    return 0;
}
```

**Output:**
```
=== Creating doubly-linked list ===
Node 1 created
Node 2 created
node1 use_count: 1
node2 use_count: 2
node2's prev data: 1

=== End of main ===
Node 2 destroyed
Node 1 destroyed
```

**Circular Reference Problem:**
```cpp
#include <iostream>
#include <memory>

class BadNode {
public:
    int data;
    std::shared_ptr<BadNode> next;
    std::shared_ptr<BadNode> prev;  // ❌ Strong reference causes leak!
    
    BadNode(int d) : data(d) {
        std::cout << "BadNode " << data << " created\n";
    }
    
    ~BadNode() {
        std::cout << "BadNode " << data << " destroyed\n";
    }
};

int main() {
    std::cout << "=== Creating circular reference ===\n";
    
    {
        auto node1 = std::make_shared<BadNode>(1);
        auto node2 = std::make_shared<BadNode>(2);
        
        node1->next = node2;
        node2->prev = node1;  // ❌ Circular: both keep each other alive!
        
        std::cout << "node1 use_count: " << node1.use_count() << "\n";  // 2
        std::cout << "node2 use_count: " << node2.use_count() << "\n";  // 2
    }
    
    std::cout << "After block - nodes leaked!\n";
    // ⚠️ Destructors never called - memory leak!
    
    return 0;
}
```

**Output:**
```
=== Creating circular reference ===
BadNode 1 created
BadNode 2 created
node1 use_count: 2
node2 use_count: 2
After block - nodes leaked!
```

### make_unique and make_shared

**Why prefer make_shared/make_unique:**

```cpp
#include <memory>

class Widget {
public:
    Widget(int x, double y) {}
};

// ❌ Less efficient: two allocations
std::shared_ptr<Widget> sp1(new Widget(10, 3.14));
// Allocation 1: Widget object
// Allocation 2: Control block (ref count)

// ✅ More efficient: single allocation
auto sp2 = std::make_shared<Widget>(10, 3.14);
// Single allocation: Widget + control block together

// ✅ Exception-safe
void process(std::shared_ptr<Widget> w, int value);

// ❌ Potential memory leak
process(std::shared_ptr<Widget>(new Widget(10, 3.14)), risky_function());
// If risky_function() throws after new but before shared_ptr constructed,
// Widget leaks!

// ✅ Exception-safe
process(std::make_shared<Widget>(10, 3.14), risky_function());
// make_shared is a single expression
```

### Custom Deleters

```cpp
#include <iostream>
#include <memory>
#include <cstdio>

// Custom deleter for FILE*
void file_deleter(FILE* f) {
    if (f) {
        std::cout << "Closing file\n";
        std::fclose(f);
    }
}

int main() {
    // unique_ptr with custom deleter
    {
        std::unique_ptr<FILE, decltype(&file_deleter)> 
            file(std::fopen("test.txt", "w"), file_deleter);
        
        if (file) {
            std::fprintf(file.get(), "Hello!\n");
        }
        
        // File automatically closed by custom deleter
    }
    
    // shared_ptr with custom deleter
    {
        std::shared_ptr<FILE> file(
            std::fopen("test2.txt", "w"),
            file_deleter
        );
        
        if (file) {
            std::fprintf(file.get(), "Hello again!\n");
        }
        
        // File automatically closed
    }
    
    // Lambda as custom deleter
    {
        auto file = std::shared_ptr<FILE>(
            std::fopen("test3.txt", "w"),
            [](FILE* f) {
                if (f) {
                    std::cout << "Lambda closing file\n";
                    std::fclose(f);
                }
            }
        );
        
        if (file) {
            std::fprintf(file.get(), "Lambda says hello!\n");
        }
    }
    
    return 0;
}
```

### Converting Between Smart Pointers

```cpp
#include <iostream>
#include <memory>

class Base {
public:
    virtual ~Base() = default;
    virtual void show() const { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void show() const override { std::cout << "Derived\n"; }
};

int main() {
    // unique_ptr to shared_ptr
    std::unique_ptr<Base> up = std::make_unique<Derived>();
    std::shared_ptr<Base> sp = std::move(up);  // Transfer ownership
    
    sp->show();
    
    // shared_ptr downcast with dynamic_pointer_cast
    std::shared_ptr<Derived> derived = std::dynamic_pointer_cast<Derived>(sp);
    
    if (derived) {
        std::cout << "Downcast successful\n";
        derived->show();
    }
    
    // static_pointer_cast (no runtime check)
    std::shared_ptr<Derived> derived2 = std::static_pointer_cast<Derived>(sp);
    
    // const_pointer_cast
    std::shared_ptr<const Base> const_sp = sp;
    std::shared_ptr<Base> non_const_sp = std::const_pointer_cast<Base>(const_sp);
    
    return 0;
}
```

---

## 3.4 Memory Management Best Practices

### Avoiding Memory Leaks

**Common Leak Scenarios:**
```cpp
#include <iostream>

// ❌ Leak: Exception before delete
void leak1() {
    int* p = new int(42);
    
    risky_operation();  // If throws, p never deleted
    
    delete p;
}

// ✅ Fixed: RAII
void no_leak1() {
    auto p = std::make_unique<int>(42);
    
    risky_operation();  // Exception-safe: p automatically deleted
}

// ❌ Leak: Early return
void leak2(bool condition) {
    int* p = new int(42);
    
    if (condition) {
        return;  // Forgot to delete!
    }
    
    delete p;
}

// ✅ Fixed: RAII
void no_leak2(bool condition) {
    auto p = std::make_unique<int>(42);
    
    if (condition) {
        return;  // p automatically deleted
    }
}

// ❌ Leak: Lost pointer
void leak3() {
    int* p = new int(42);
    p = new int(100);  // Lost original allocation!
    delete p;  // Only deletes second allocation
}

// ✅ Fixed: Proper cleanup
void no_leak3() {
    auto p = std::make_unique<int>(42);
    p = std::make_unique<int>(100);  // Old value automatically deleted
}
```

### Double-Free and Use-After-Free Errors

```cpp
#include <iostream>

// ❌ Double-free
void double_free() {
    int* p = new int(42);
    delete p;
    delete p;  // ⚠️ Undefined behavior!
}

// ✅ Set to nullptr after delete
void safe_delete() {
    int* p = new int(42);
    delete p;
    p = nullptr;  // Safe to delete again
    delete p;     // No-op (safe)
}

// ❌ Use-after-free
void use_after_free() {
    int* p = new int(42);
    delete p;
    std::cout << *p << "\n";  // ⚠️ Undefined behavior!
}

// ✅ RAII prevents use-after-free
void raii_safe() {
    auto p = std::make_unique<int>(42);
    // Cannot accidentally use after scope
}
```

### Ownership Semantics

**Clear Ownership Models:**
```cpp
#include <memory>
#include <vector>

class Document {
    std::string content;
    
public:
    Document(const std::string& c) : content(c) {}
    const std::string& get_content() const { return content; }
};

// ✅ Exclusive ownership: unique_ptr
std::unique_ptr<Document> create_document(const std::string& content) {
    return std::make_unique<Document>(content);
}

// ✅ Shared ownership: shared_ptr
class Library {
    std::vector<std::shared_ptr<Document>> documents;
    
public:
    void add_document(std::shared_ptr<Document> doc) {
        documents.push_back(doc);
    }
    
    std::shared_ptr<Document> get_document(size_t index) {
        return documents[index];  // Share ownership
    }
};

// ✅ Non-owning access: raw pointer or reference
void print_document(const Document* doc) {  // Or const Document&
    if (doc) {
        std::cout << doc->get_content() << "\n";
    }
}

int main() {
    // Exclusive ownership
    auto doc1 = create_document("Hello");
    
    // Share ownership
    Library library;
    auto doc2 = std::make_shared<Document>("World");
    library.add_document(doc2);
    
    // Non-owning access
    print_document(doc1.get());
    print_document(doc2.get());
    
    return 0;
}
```

### Move Semantics for Efficient Resource Transfer

```cpp
#include <iostream>
#include <vector>
#include <memory>

class Buffer {
    size_t size;
    std::unique_ptr<char[]> data;
    
public:
    // Constructor
    Buffer(size_t s) : size(s), data(std::make_unique<char[]>(s)) {
        std::cout << "Buffer(" << size << ") constructed\n";
    }
    
    // Move constructor
    Buffer(Buffer&& other) noexcept 
        : size(other.size), data(std::move(other.data)) {
        std::cout << "Buffer moved\n";
        other.size = 0;
    }
    
    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            size = other.size;
            data = std::move(other.data);
            other.size = 0;
            std::cout << "Buffer move-assigned\n";
        }
        return *this;
    }
    
    // Delete copy operations (unique ownership)
    Buffer(const Buffer&) = delete;
    Buffer& operator=(const Buffer&) = delete;
    
    ~Buffer() {
        std::cout << "Buffer(" << size << ") destroyed\n";
    }
    
    size_t get_size() const { return size; }
};

Buffer create_buffer(size_t size) {
    Buffer buf(size);
    return buf;  // Move, not copy
}

int main() {
    std::cout << "=== Creating buffer ===\n";
    Buffer buf1(1024);
    
    std::cout << "\n=== Moving buffer ===\n";
    Buffer buf2 = std::move(buf1);
    std::cout << "buf1 size: " << buf1.get_size() << "\n";
    std::cout << "buf2 size: " << buf2.get_size() << "\n";
    
    std::cout << "\n=== Function return ===\n";
    Buffer buf3 = create_buffer(2048);
    
    std::cout << "\n=== Using in vector ===\n";
    std::vector<Buffer> buffers;
    buffers.push_back(create_buffer(512));
    buffers.push_back(create_buffer(1024));
    
    std::cout << "\n=== End ===\n";
    return 0;
}
```

### Copy Elision and RVO/NRVO

```cpp
#include <iostream>

class Expensive {
    int* data;
    
public:
    Expensive(int value) : data(new int(value)) {
        std::cout << "Expensive constructed\n";
    }
    
    Expensive(const Expensive& other) : data(new int(*other.data)) {
        std::cout << "Expensive copied\n";
    }
    
    Expensive(Expensive&& other) noexcept : data(other.data) {
        other.data = nullptr;
        std::cout << "Expensive moved\n";
    }
    
    ~Expensive() {
        std::cout << "Expensive destroyed\n";
        delete data;
    }
    
    int get_value() const { return data ? *data : 0; }
};

// RVO: Return Value Optimization
Expensive create_expensive(int value) {
    Expensive obj(value);
    return obj;  // RVO: no copy or move!
}

// Guaranteed copy elision (C++17)
Expensive guaranteed_elision(int value) {
    return Expensive(value);  // Prvalue: guaranteed no copy/move
}

int main() {
    std::cout << "=== RVO ===\n";
    Expensive e1 = create_expensive(42);
    
    std::cout << "\n=== Guaranteed elision (C++17) ===\n";
    Expensive e2 = guaranteed_elision(100);
    
    std::cout << "\n=== End ===\n";
    return 0;
}
```

**Output (with optimizations):**
```
=== RVO ===
Expensive constructed

=== Guaranteed elision (C++17) ===
Expensive constructed

=== End ===
Expensive destroyed
Expensive destroyed
```

[Note: This is approximately 50% of Part 3. Would you like me to continue with sections 3.5 covering debugging memory issues, custom allocators, and PMR?]

---

## 3.5 Debugging Memory Issues

### Memory Debugging Tools

**Valgrind - Memory Error Detection:**
```bash
# Compile with debug symbols
g++ -g -std=c++17 program.cpp -o program

# Run with Valgrind
valgrind --leak-check=full --show-leak-kinds=all ./program

# Track origins of uninitialized values
valgrind --track-origins=yes ./program
```

**Example Program with Memory Errors:**
```cpp
#include <iostream>

void memory_errors() {
    // 1. Memory leak
    int* leak = new int(42);
    // Forgot to delete!
    
    // 2. Use after free
    int* p = new int(100);
    delete p;
    std::cout << *p << "\n";  // ⚠️ Use after free
    
    // 3. Double free
    int* q = new int(200);
    delete q;
    delete q;  // ⚠️ Double free
    
    // 4. Invalid read
    int arr[10];
    std::cout << arr[100] << "\n";  // ⚠️ Out of bounds
}

int main() {
    memory_errors();
    return 0;
}
```

**AddressSanitizer (ASan):**
```bash
# Compile with AddressSanitizer
g++ -g -fsanitize=address -std=c++17 program.cpp -o program

# Run (ASan detects errors at runtime)
./program
```

**ASan Example:**
```cpp
#include <iostream>

int main() {
    int* arr = new int[10];
    
    // ASan detects heap-buffer-overflow
    std::cout << arr[10] << "\n";  // ⚠️ Out of bounds
    
    delete[] arr;
    
    // ASan detects heap-use-after-free
    std::cout << arr[0] << "\n";  // ⚠️ Use after free
    
    return 0;
}
```

**MemorySanitizer (MSan) - Uninitialized Memory:**
```bash
# Compile with MemorySanitizer (Clang only)
clang++ -g -fsanitize=memory -std=c++17 program.cpp -o program

./program
```

**LeakSanitizer (LSan) - Memory Leaks:**
```bash
# Compile with LeakSanitizer
g++ -g -fsanitize=leak -std=c++17 program.cpp -o program

./program
```

### Static Analysis Tools

**Clang-Tidy:**
```bash
# Run clang-tidy
clang-tidy program.cpp -- -std=c++17

# With specific checks
clang-tidy program.cpp -checks='clang-analyzer-*,bugprone-*' -- -std=c++17
```

**Cppcheck:**
```bash
# Run cppcheck
cppcheck --enable=all --std=c++17 program.cpp
```

### Runtime Assertions

```cpp
#include <cassert>
#include <iostream>

class Buffer {
    char* data;
    size_t size;
    
public:
    Buffer(size_t s) : size(s), data(new char[s]) {
        assert(s > 0 && "Buffer size must be positive");
        assert(data != nullptr && "Allocation failed");
    }
    
    ~Buffer() {
        delete[] data;
    }
    
    char& operator[](size_t index) {
        assert(index < size && "Index out of bounds");
        return data[index];
    }
    
    size_t get_size() const { return size; }
};

int main() {
    Buffer buf(10);
    
    buf[0] = 'A';     // OK
    // buf[10] = 'B'; // ⚠️ Assertion fails in debug build
    
    return 0;
}
```

**Static Assertions (C++11):**
```cpp
template<typename T>
class Container {
    static_assert(sizeof(T) <= 1024, "Type too large for Container");
    T data;
};

int main() {
    Container<int> c1;  // OK
    // Container<char[2000]> c2;  // ❌ Compile error
    
    return 0;
}
```

### Debug vs Release Builds

```cpp
#include <iostream>

// Debug-only code
#ifndef NDEBUG
    #define DEBUG_LOG(msg) std::cout << "[DEBUG] " << msg << "\n"
#else
    #define DEBUG_LOG(msg)
#endif

class TrackedResource {
    int id;
    
public:
    TrackedResource(int i) : id(i) {
        DEBUG_LOG("Resource " << id << " acquired");
    }
    
    ~TrackedResource() {
        DEBUG_LOG("Resource " << id << " released");
    }
    
    void use() {
        DEBUG_LOG("Using resource " << id);
        // Actual work...
    }
};

int main() {
    TrackedResource res(1);
    res.use();
    
    // Debug messages only in debug builds
    
    return 0;
}
```

### Memory Profiling

**Custom Memory Tracking:**
```cpp
#include <iostream>
#include <unordered_map>
#include <cstdlib>
#include <mutex>

class MemoryTracker {
    struct AllocationInfo {
        size_t size;
        const char* file;
        int line;
    };
    
    std::unordered_map<void*, AllocationInfo> allocations;
    std::mutex mtx;
    size_t total_allocated = 0;
    size_t total_freed = 0;
    
public:
    void track_allocation(void* ptr, size_t size, const char* file, int line) {
        std::lock_guard<std::mutex> lock(mtx);
        allocations[ptr] = {size, file, line};
        total_allocated += size;
    }
    
    void track_deallocation(void* ptr) {
        std::lock_guard<std::mutex> lock(mtx);
        auto it = allocations.find(ptr);
        if (it != allocations.end()) {
            total_freed += it->second.size;
            allocations.erase(it);
        }
    }
    
    void report() {
        std::lock_guard<std::mutex> lock(mtx);
        std::cout << "\n=== Memory Report ===\n";
        std::cout << "Total allocated: " << total_allocated << " bytes\n";
        std::cout << "Total freed: " << total_freed << " bytes\n";
        std::cout << "Currently leaked: " << (total_allocated - total_freed) << " bytes\n";
        
        if (!allocations.empty()) {
            std::cout << "\nLeaked allocations:\n";
            for (const auto& [ptr, info] : allocations) {
                std::cout << "  " << info.size << " bytes at " 
                          << info.file << ":" << info.line << "\n";
            }
        }
    }
    
    static MemoryTracker& instance() {
        static MemoryTracker tracker;
        return tracker;
    }
};

// Override global new/delete for tracking
void* operator new(size_t size, const char* file, int line) {
    void* ptr = std::malloc(size);
    MemoryTracker::instance().track_allocation(ptr, size, file, line);
    return ptr;
}

void operator delete(void* ptr) noexcept {
    MemoryTracker::instance().track_deallocation(ptr);
    std::free(ptr);
}

#define NEW new(__FILE__, __LINE__)

int main() {
    int* p1 = NEW int(42);
    int* p2 = NEW int[10];
    
    delete p1;
    // Forgot to delete[] p2 - will be detected!
    
    MemoryTracker::instance().report();
    
    return 0;
}
```

---

## Comprehensive Example: Smart Resource Manager

```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>
#include <algorithm>

// Abstract resource interface
class Resource {
public:
    virtual ~Resource() = default;
    virtual std::string get_info() const = 0;
    virtual void use() = 0;
};

// Concrete resource: File
class FileResource : public Resource {
    std::string filename;
    
public:
    explicit FileResource(const std::string& name) : filename(name) {
        std::cout << "Opening file: " << filename << "\n";
    }
    
    ~FileResource() override {
        std::cout << "Closing file: " << filename << "\n";
    }
    
    std::string get_info() const override {
        return "File: " + filename;
    }
    
    void use() override {
        std::cout << "Reading from " << filename << "\n";
    }
};

// Concrete resource: Network connection
class NetworkResource : public Resource {
    std::string address;
    
public:
    explicit NetworkResource(const std::string& addr) : address(addr) {
        std::cout << "Connecting to: " << address << "\n";
    }
    
    ~NetworkResource() override {
        std::cout << "Disconnecting from: " << address << "\n";
    }
    
    std::string get_info() const override {
        return "Network: " + address;
    }
    
    void use() override {
        std::cout << "Sending data to " << address << "\n";
    }
};

// Resource manager with smart pointers
class ResourceManager {
    std::vector<std::shared_ptr<Resource>> resources;
    
public:
    void add_resource(std::shared_ptr<Resource> res) {
        resources.push_back(res);
        std::cout << "Added resource: " << res->get_info() << "\n";
    }
    
    void use_all_resources() {
        std::cout << "\n=== Using all resources ===\n";
        for (auto& res : resources) {
            res->use();
        }
    }
    
    void list_resources() const {
        std::cout << "\n=== Resource list ===\n";
        for (size_t i = 0; i < resources.size(); ++i) {
            std::cout << i << ": " << resources[i]->get_info() 
                      << " (refs: " << resources[i].use_count() << ")\n";
        }
    }
    
    std::shared_ptr<Resource> get_resource(size_t index) {
        if (index < resources.size()) {
            return resources[index];
        }
        return nullptr;
    }
    
    void remove_resource(size_t index) {
        if (index < resources.size()) {
            std::cout << "Removing: " << resources[index]->get_info() << "\n";
            resources.erase(resources.begin() + index);
        }
    }
    
    ~ResourceManager() {
        std::cout << "\n=== ResourceManager cleanup ===\n";
    }
};

int main() {
    std::cout << "=== Creating ResourceManager ===\n";
    ResourceManager manager;
    
    std::cout << "\n=== Adding resources ===\n";
    manager.add_resource(std::make_shared<FileResource>("data.txt"));
    manager.add_resource(std::make_shared<NetworkResource>("192.168.1.1"));
    manager.add_resource(std::make_shared<FileResource>("config.ini"));
    
    manager.list_resources();
    manager.use_all_resources();
    
    std::cout << "\n=== Sharing resource ===\n";
    {
        auto shared_res = manager.get_resource(0);
        std::cout << "Shared: " << shared_res->get_info() 
                  << " (refs: " << shared_res.use_count() << ")\n";
        
        manager.list_resources();
    }  // shared_res goes out of scope
    
    std::cout << "\n=== After shared resource released ===\n";
    manager.list_resources();
    
    std::cout << "\n=== Removing resource ===\n";
    manager.remove_resource(1);
    
    manager.list_resources();
    
    std::cout << "\n=== End of main ===\n";
    return 0;
}
```

---

## Conclusion

Part 3 has provided comprehensive coverage of memory management and resource handling in C++:

1. **Dynamic Memory Allocation** - Stack vs heap, new/delete, placement new
2. **Pointers and References** - Fundamentals, arithmetic, null pointers, function pointers
3. **RAII and Smart Pointers** - unique_ptr, shared_ptr, weak_ptr, custom deleters
4. **Memory Management Best Practices** - Avoiding leaks, ownership semantics, move semantics
5. **Debugging Memory Issues** - Valgrind, sanitizers, static analysis, profiling

**Key Principles:**
- **RAII**: Tie resource lifetime to object lifetime
- **Smart Pointers**: Use unique_ptr by default, shared_ptr when needed
- **Ownership**: Make ownership explicit and clear
- **Rule of Zero**: Prefer standard containers over manual memory management
- **Move Semantics**: Enable efficient resource transfer
- **Tools**: Use sanitizers and static analyzers in development

**Common Patterns:**
```cpp
// ✅ Exclusive ownership
std::unique_ptr<Resource> resource = std::make_unique<Resource>();

// ✅ Shared ownership
std::shared_ptr<Resource> shared = std::make_shared<Resource>();

// ✅ Non-owning reference
void process(const Resource* res);  // Or Resource&

// ✅ Breaking cycles
std::weak_ptr<Resource> weak_ref = shared;
```

**Memory Safety Checklist:**
- [ ] Use RAII for all resources
- [ ] Prefer smart pointers over raw pointers
- [ ] Use make_unique/make_shared
- [ ] Mark move operations noexcept
- [ ] Check for null before dereferencing
- [ ] Enable sanitizers in development
- [ ] Run static analysis regularly
- [ ] Profile memory usage for large applications
- [ ] Use const correctness
- [ ] Follow the Rule of Zero when possible

**Next in the series:** Part 4 will explore Templates & Generic Programming, covering function templates, class templates, template metaprogramming, and C++20 concepts.

---

## References and Further Reading

- **C++ Core Guidelines**: Memory management section
- **"Effective Modern C++"** by Scott Meyers - Smart pointers, move semantics
- **Valgrind User Manual**: http://valgrind.org/docs/manual/
- **AddressSanitizer**: https://github.com/google/sanitizers/wiki/AddressSanitizer
- **cppreference.com**: Memory management reference

---

*This blog post is part of the Complete C++ Mastery series. Continue to Part 4 for Templates & Generic Programming.*
