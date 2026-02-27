---
title: "Complete C++ Mastery Part 15: Interview Preparation & Common Questions"
date: 2024-02-21 14:00:00 +0000
categories: [C++, Programming, Career]
tags: [cpp, interview, questions, career, preparation, coding-interview]
---

# Complete C++ Mastery Part 15: Interview Preparation & Common Questions

## Introduction

Welcome to Part 15 of the Complete C++ Mastery series. This part prepares you for C++ technical interviews, covering frequently asked questions, coding challenges, system design, and behavioral topics. Whether you're interviewing for junior, mid-level, or senior positions, understanding what interviewers look for is crucial.

We'll cover conceptual questions, coding problems, architecture discussions, and strategies for succeeding in technical interviews.

**Prerequisites:** This part synthesizes knowledge from all previous parts.

---

## 15.1 Core Concepts Questions

### Memory Management

**Q: Explain the difference between stack and heap memory.**

```cpp
void example() {
    // Stack allocation
    int x = 42;              // Fast, automatic cleanup
    int arr[100];            // Fixed size, stack space
    
    // Heap allocation
    int* ptr = new int(42);  // Slower, manual management
    delete ptr;              // Must explicitly delete
    
    // Modern C++: Smart pointers (heap with automatic cleanup)
    auto smart = std::make_unique<int>(42);  // Best of both worlds
}

/*
Stack:
- Fast allocation/deallocation (just move stack pointer)
- Automatic cleanup (LIFO)
- Limited size (~1-8 MB)
- Scope-based lifetime

Heap:
- Slower allocation (need to find free block)
- Manual management (or use smart pointers)
- Large size (limited by system memory)
- Flexible lifetime
*/
```

**Q: What are smart pointers? When would you use each type?**

```cpp
#include <memory>

void smart_pointer_examples() {
    // unique_ptr: Exclusive ownership
    // Use when: Single owner, transfer ownership via move
    auto unique = std::make_unique<int>(42);
    // auto copy = unique;  // ❌ Error: cannot copy
    auto moved = std::move(unique);  // ✅ Transfer ownership
    
    // shared_ptr: Shared ownership (reference counted)
    // Use when: Multiple owners, shared resources
    auto shared1 = std::make_shared<int>(100);
    auto shared2 = shared1;  // Reference count: 2
    // Both can access, destroyed when last shared_ptr dies
    
    // weak_ptr: Non-owning reference to shared_ptr
    // Use when: Break circular references, observer pattern
    std::weak_ptr<int> weak = shared1;
    if (auto locked = weak.lock()) {  // Check if still valid
        // Use locked (which is a shared_ptr)
    }
}

/*
Decision tree:
1. Single owner? → unique_ptr
2. Multiple owners? → shared_ptr
3. Need to observe without owning? → weak_ptr
4. Need to observe raw object? → raw pointer (non-owning)
*/
```

**Q: What is RAII? Give examples.**

```cpp
#include <fstream>
#include <mutex>

// RAII: Resource Acquisition Is Initialization
// Resource lifetime tied to object lifetime

// Example 1: File handling
void file_example() {
    std::ofstream file("data.txt");  // Acquire in constructor
    file << "Hello";
    // File automatically closed in destructor
}

// Example 2: Mutex locking
void thread_safe() {
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);  // Acquire in constructor
    // Critical section
    // Mutex automatically unlocked in destructor
}

// Example 3: Custom RAII class
class Timer {
    std::chrono::steady_clock::time_point start;
    const char* name;
public:
    Timer(const char* n) : name(n), start(std::chrono::steady_clock::now()) {}
    
    ~Timer() {
        auto duration = std::chrono::steady_clock::now() - start;
        std::cout << name << ": " 
                  << std::chrono::duration_cast<std::chrono::milliseconds>(duration).count()
                  << "ms\n";
    }
};

void measured_function() {
    Timer t("measured_function");
    // Function body
    // Timing automatically reported at end
}
```

### Object-Oriented Programming

**Q: Explain virtual functions and dynamic polymorphism.**

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Shape {
public:
    virtual ~Shape() = default;  // Virtual destructor essential!
    
    virtual double area() const = 0;  // Pure virtual (abstract)
    virtual void draw() const {       // Virtual with default implementation
        std::cout << "Drawing shape\n";
    }
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    
    double area() const override {
        return 3.14159 * radius * radius;
    }
    
    void draw() const override {
        std::cout << "Drawing circle\n";
    }
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double area() const override {
        return width * height;
    }
    
    void draw() const override {
        std::cout << "Drawing rectangle\n";
    }
};

void polymorphism_example() {
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5.0));
    shapes.push_back(std::make_unique<Rectangle>(4.0, 6.0));
    
    // Polymorphic behavior
    for (const auto& shape : shapes) {
        shape->draw();  // Calls appropriate override
        std::cout << "Area: " << shape->area() << "\n";
    }
}

/*
Virtual function mechanism:
1. Each class with virtual functions has vtable (virtual table)
2. Each object has vptr (pointer to vtable)
3. Virtual call: Follow vptr → vtable → function pointer
4. Small overhead: one pointer per object, one indirection per call
*/
```

**Q: What's the difference between override and overload?**

```cpp
class Base {
public:
    virtual void foo(int x) { }      // Virtual function
    void bar(int x) { }              // Non-virtual
};

class Derived : public Base {
public:
    // Override: Same signature, virtual in base
    void foo(int x) override { }     // ✅ Override
    
    // Overload: Different signature
    void bar(double x) { }           // Overload (hides Base::bar)
    using Base::bar;                 // Make Base::bar visible
    
    // void foo(double x) override;  // ❌ Error: doesn't override anything
};

/*
Override:
- Replaces base class virtual function
- Same signature
- Must be virtual in base
- Use 'override' keyword to catch errors

Overload:
- Same function name, different parameters
- Can be in same class or base/derived
- Resolved at compile time
*/
```

### Move Semantics

**Q: Explain lvalue vs rvalue, and move semantics.**

```cpp
#include <utility>
#include <vector>

void lvalue_rvalue_examples() {
    int x = 42;
    
    // Lvalue: Has identity (address)
    int* ptr = &x;  // ✅ Can take address
    
    // Rvalue: Temporary, no identity
    // int* ptr2 = &42;  // ❌ Cannot take address of rvalue
    
    int y = x;       // Copy from lvalue
    int z = x + y;   // Copy from rvalue (temporary result)
}

class Buffer {
    int* data;
    size_t size;
    
public:
    // Constructor
    Buffer(size_t n) : data(new int[n]), size(n) {
        std::cout << "Allocate " << size << " elements\n";
    }
    
    // Destructor
    ~Buffer() {
        delete[] data;
        std::cout << "Deallocate\n";
    }
    
    // Copy constructor (expensive)
    Buffer(const Buffer& other) : size(other.size) {
        std::cout << "Copy " << size << " elements\n";
        data = new int[size];
        std::copy(other.data, other.data + size, data);
    }
    
    // Move constructor (cheap)
    Buffer(Buffer&& other) noexcept
        : data(other.data), size(other.size) {
        std::cout << "Move " << size << " elements\n";
        other.data = nullptr;
        other.size = 0;
    }
    
    // Copy assignment
    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }
    
    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
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

Buffer create_buffer() {
    return Buffer(1000);  // Returned by value
}

void move_semantics_example() {
    Buffer b1(100);
    
    // Copy
    Buffer b2 = b1;  // Copy constructor (expensive)
    
    // Move
    Buffer b3 = std::move(b1);  // Move constructor (cheap)
    // b1 is now in "moved-from" state
    
    // Return value optimization
    Buffer b4 = create_buffer();  // Move (or RVO/NRVO)
}

/*
Key points:
1. Move is just stealing resources
2. Left-hand object in valid but unspecified state
3. noexcept important for performance (vector reallocate)
4. std::move is just a cast to rvalue reference
*/
```

---

## 15.2 Advanced Concepts

### Templates

**Q: Explain template specialization.**

```cpp
#include <iostream>
#include <cstring>

// Primary template
template<typename T>
class Printer {
public:
    void print(const T& value) {
        std::cout << value << "\n";
    }
};

// Full specialization for bool
template<>
class Printer<bool> {
public:
    void print(bool value) {
        std::cout << (value ? "true" : "false") << "\n";
    }
};

// Full specialization for const char*
template<>
class Printer<const char*> {
public:
    void print(const char* value) {
        std::cout << "String: " << value << "\n";
    }
};

// Partial specialization for pointers
template<typename T>
class Printer<T*> {
public:
    void print(T* value) {
        std::cout << "Pointer to: " << *value << "\n";
    }
};

void specialization_example() {
    Printer<int> pi;
    pi.print(42);
    
    Printer<bool> pb;
    pb.print(true);
    
    Printer<const char*> ps;
    ps.print("hello");
    
    int x = 100;
    Printer<int*> pp;
    pp.print(&x);
}
```

**Q: What is SFINAE?**

```cpp
#include <type_traits>
#include <iostream>

// SFINAE: Substitution Failure Is Not An Error

// Example 1: enable_if
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T value) {
    std::cout << "Processing integer: " << value << "\n";
    return value * 2;
}

template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
process(T value) {
    std::cout << "Processing float: " << value << "\n";
    return value * 1.5;
}

// Example 2: Detection idiom
template<typename T, typename = void>
struct has_size : std::false_type {};

template<typename T>
struct has_size<T, std::void_t<decltype(std::declval<T>().size())>>
    : std::true_type {};

void sfinae_examples() {
    process(42);     // Calls integer version
    process(3.14);   // Calls float version
    
    std::cout << "vector has size: " << has_size<std::vector<int>>::value << "\n";
    std::cout << "int has size: " << has_size<int>::value << "\n";
}

/*
SFINAE allows:
- Function overload selection based on type properties
- Compile-time reflection (has_member, is_callable)
- Type traits implementation

C++20 alternative: Concepts (cleaner, better error messages)
*/
```

### Concurrency

**Q: Explain race conditions and how to prevent them.**

```cpp
#include <thread>
#include <mutex>
#include <atomic>
#include <vector>

// ❌ BAD: Race condition
int counter = 0;

void increment_unsafe() {
    for (int i = 0; i < 100000; ++i) {
        ++counter;  // Read-modify-write, not atomic!
    }
}

// ✅ GOOD: Mutex protection
std::mutex mtx;
int counter_safe = 0;

void increment_mutex() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        ++counter_safe;
    }
}

// ✅ BETTER: Atomic
std::atomic<int> counter_atomic{0};

void increment_atomic() {
    for (int i = 0; i < 100000; ++i) {
        ++counter_atomic;  // Atomic operation
    }
}

void race_condition_demo() {
    std::vector<std::thread> threads;
    
    // Unsafe version (race condition)
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment_unsafe);
    }
    for (auto& t : threads) t.join();
    threads.clear();
    std::cout << "Unsafe: " << counter << "\n";  // Wrong result!
    
    // Safe versions
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment_mutex);
    }
    for (auto& t : threads) t.join();
    std::cout << "Mutex: " << counter_safe << "\n";  // Correct
}

/*
Race condition occurs when:
1. Multiple threads access shared data
2. At least one thread modifies data
3. No synchronization

Prevention:
1. Mutex/lock_guard for critical sections
2. Atomic operations for simple counters
3. Lock-free data structures (advanced)
4. Thread-local storage (no sharing)
*/
```

**Q: Explain deadlock and how to avoid it.**

```cpp
#include <thread>
#include <mutex>

std::mutex m1, m2;

// ❌ BAD: Potential deadlock
void thread1_bad() {
    m1.lock();
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    m2.lock();  // ⚠️ Deadlock if thread2 holds m2!
    // Work...
    m2.unlock();
    m1.unlock();
}

void thread2_bad() {
    m2.lock();
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    m1.lock();  // ⚠️ Deadlock if thread1 holds m1!
    // Work...
    m1.unlock();
    m2.unlock();
}

// ✅ GOOD: Consistent lock order
void thread1_good() {
    m1.lock();
    m2.lock();
    // Work...
    m2.unlock();
    m1.unlock();
}

void thread2_good() {
    m1.lock();  // Same order as thread1
    m2.lock();
    // Work...
    m2.unlock();
    m1.unlock();
}

// ✅ BETTER: std::lock or scoped_lock
void thread_better() {
    std::scoped_lock lock(m1, m2);  // C++17: atomic lock
    // Work...
}

/*
Deadlock conditions (all must be true):
1. Mutual exclusion (resources can't be shared)
2. Hold and wait (holding one, waiting for another)
3. No preemption (can't force release)
4. Circular wait (cycle in wait graph)

Prevention strategies:
1. Consistent lock ordering
2. std::lock / scoped_lock (atomic multi-lock)
3. Try-lock with timeout
4. Lock hierarchies
*/
```

---

## 15.3 Coding Problems

### Problem 1: Implement std::unique_ptr

```cpp
template<typename T>
class UniquePtr {
    T* ptr;
    
public:
    // Constructor
    explicit UniquePtr(T* p = nullptr) : ptr(p) {}
    
    // Destructor
    ~UniquePtr() {
        delete ptr;
    }
    
    // Move constructor
    UniquePtr(UniquePtr&& other) noexcept : ptr(other.ptr) {
        other.ptr = nullptr;
    }
    
    // Move assignment
    UniquePtr& operator=(UniquePtr&& other) noexcept {
        if (this != &other) {
            delete ptr;
            ptr = other.ptr;
            other.ptr = nullptr;
        }
        return *this;
    }
    
    // Delete copy operations
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
    
    // Dereference
    T& operator*() const { return *ptr; }
    T* operator->() const { return ptr; }
    
    // Get raw pointer
    T* get() const { return ptr; }
    
    // Release ownership
    T* release() {
        T* temp = ptr;
        ptr = nullptr;
        return temp;
    }
    
    // Reset
    void reset(T* p = nullptr) {
        delete ptr;
        ptr = p;
    }
    
    // Bool conversion
    explicit operator bool() const {
        return ptr != nullptr;
    }
};
```

### Problem 2: Thread-Safe Singleton

```cpp
#include <mutex>

class Singleton {
    static Singleton* instance;
    static std::mutex mtx;
    
    // Private constructor
    Singleton() {}
    
public:
    // Delete copy/move
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    // Double-checked locking
    static Singleton* get_instance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;

// ✅ BETTER: C++11 thread-safe local static
class ModernSingleton {
    ModernSingleton() {}
    
public:
    static ModernSingleton& get_instance() {
        static ModernSingleton instance;  // Thread-safe in C++11
        return instance;
    }
    
    ModernSingleton(const ModernSingleton&) = delete;
    ModernSingleton& operator=(const ModernSingleton&) = delete;
};
```

### Problem 3: LRU Cache

```cpp
#include <unordered_map>
#include <list>

class LRUCache {
    int capacity;
    std::list<std::pair<int, int>> items;  // key-value pairs
    std::unordered_map<int, std::list<std::pair<int, int>>::iterator> cache;
    
public:
    LRUCache(int cap) : capacity(cap) {}
    
    int get(int key) {
        auto it = cache.find(key);
        if (it == cache.end()) {
            return -1;  // Not found
        }
        
        // Move to front (most recently used)
        items.splice(items.begin(), items, it->second);
        return it->second->second;
    }
    
    void put(int key, int value) {
        auto it = cache.find(key);
        
        if (it != cache.end()) {
            // Update existing
            it->second->second = value;
            items.splice(items.begin(), items, it->second);
        } else {
            // Insert new
            if (items.size() >= capacity) {
                // Remove least recently used
                int old_key = items.back().first;
                items.pop_back();
                cache.erase(old_key);
            }
            
            items.push_front({key, value});
            cache[key] = items.begin();
        }
    }
};
```

---

## 15.4 System Design Questions

### Question: Design a thread-safe queue

```cpp
#include <queue>
#include <mutex>
#include <condition_variable>
#include <optional>

template<typename T>
class ThreadSafeQueue {
    std::queue<T> queue;
    mutable std::mutex mtx;
    std::condition_variable cv;
    bool finished = false;
    
public:
    void push(T value) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            queue.push(std::move(value));
        }
        cv.notify_one();
    }
    
    std::optional<T> pop() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this] { return !queue.empty() || finished; });
        
        if (queue.empty()) {
            return std::nullopt;  // Queue finished and empty
        }
        
        T value = std::move(queue.front());
        queue.pop();
        return value;
    }
    
    bool try_pop(T& value) {
        std::lock_guard<std::mutex> lock(mtx);
        if (queue.empty()) {
            return false;
        }
        
        value = std::move(queue.front());
        queue.pop();
        return true;
    }
    
    void finish() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            finished = true;
        }
        cv.notify_all();
    }
    
    size_t size() const {
        std::lock_guard<std::mutex> lock(mtx);
        return queue.size();
    }
};
```

---

## 15.5 Interview Strategies

### Approach to Coding Problems

```
1. Clarify requirements
   - Input/output format
   - Edge cases
   - Constraints (time/space)

2. Think out loud
   - Explain approach before coding
   - Discuss trade-offs
   - Consider alternatives

3. Start with brute force
   - Simple solution first
   - Then optimize

4. Test your code
   - Normal cases
   - Edge cases
   - Error conditions

5. Analyze complexity
   - Time: O(?)
   - Space: O(?)
```

### Common Pitfalls to Avoid

```cpp
// ❌ Don't forget edge cases
int divide(int a, int b) {
    return a / b;  // ⚠️ What if b == 0?
}

// ✅ Handle edge cases
int divide_safe(int a, int b) {
    if (b == 0) {
        throw std::invalid_argument("Division by zero");
    }
    return a / b;
}

// ❌ Don't ignore const correctness
class Container {
    std::vector<int> data;
public:
    int size() { return data.size(); }  // ⚠️ Should be const
};

// ✅ Use const
class Container {
    std::vector<int> data;
public:
    size_t size() const { return data.size(); }
};

// ❌ Don't leak resources
void bad_function() {
    int* ptr = new int(42);
    // ... might throw
    delete ptr;  // ⚠️ Might not execute
}

// ✅ Use RAII
void good_function() {
    auto ptr = std::make_unique<int>(42);
    // Automatic cleanup
}
```

---

## Conclusion

Part 15 covered C++ interview preparation:

1. **Core Concepts** - Memory, OOP, move semantics
2. **Advanced Topics** - Templates, SFINAE, concurrency
3. **Coding Problems** - unique_ptr, singleton, LRU cache
4. **System Design** - Thread-safe queue
5. **Strategies** - Problem-solving approach

**Interview Checklist:**
- [ ] Understand memory management
- [ ] Know smart pointers deeply
- [ ] Explain virtual functions
- [ ] Implement move semantics
- [ ] Handle concurrency correctly
- [ ] Write exception-safe code
- [ ] Analyze complexity
- [ ] Test edge cases

**Next:** Part 16 covers Common Pitfalls.

---

*This blog post is part of the Complete C++ Mastery series.*
