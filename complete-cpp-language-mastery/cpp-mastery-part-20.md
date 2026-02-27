---
title: "Complete C++ Mastery Part 20: FAQ & Troubleshooting"
date: 2024-02-22 00:00:00 +0000
categories: [C++, Programming, Troubleshooting]
tags: [cpp, faq, troubleshooting, debugging, common-issues, solutions]
---

# Complete C++ Mastery Part 20: FAQ & Troubleshooting

## Introduction

Welcome to Part 20 of the Complete C++ Mastery series. This part addresses frequently asked questions and common troubleshooting scenarios. Whether you're encountering cryptic compiler errors, runtime crashes, or performance issues, this guide provides practical solutions.

Organized by problem category for quick reference.

---

## 20.1 Compilation Errors

### Q: "undefined reference to..." / Linker Error

**Problem:**
```bash
/usr/bin/ld: /tmp/main.o: undefined reference to `MyClass::function()'
```

**Causes & Solutions:**

```cpp
// ❌ CAUSE 1: Missing implementation
// header.h
class MyClass {
public:
    void function();  // Declared
};

// ✅ SOLUTION: Add implementation
// source.cpp
#include "header.h"
void MyClass::function() {
    // Implementation
}

// ❌ CAUSE 2: Forgot to link library
g++ main.cpp -o myapp  // Missing -lmylib

// ✅ SOLUTION: Link the library
g++ main.cpp -lmylib -o myapp

// ❌ CAUSE 3: Template function in .cpp file
// header.h
template<typename T>
void process(T value);

// source.cpp (wrong!)
template<typename T>
void process(T value) { /* ... */ }

// ✅ SOLUTION: Define templates in header
template<typename T>
void process(T value) {
    // Implementation in header
}
```

### Q: Template Error Messages Too Long

**Problem:**
```
error: no matching function for call to 'std::vector<int>::push_back(std::string)'
candidate: void std::vector<_Tp, _Alloc>::push_back(const value_type&) [with _Tp = int; _Alloc = std::allocator<int>; value_type = int]
...hundreds of lines...
```

**Solutions:**

```bash
# GCC: Limit template depth in errors
g++ -ftemplate-backtrace-limit=1 main.cpp

# Clang: Better error messages by default
clang++ main.cpp

# Use concepts (C++20) for clearer errors
template<std::integral T>
void process(T value) { }
```

### Q: Cannot Convert 'X' to 'Y'

**Problem:**
```cpp
std::string s = "hello";
int x = s;  // Error: cannot convert std::string to int
```

**Solutions:**

```cpp
// ✅ Use explicit conversion
int x = std::stoi(s);

// ✅ For custom types, provide conversion operator
class MyType {
public:
    explicit operator int() const {
        return value;
    }
private:
    int value;
};
```

---

## 20.2 Runtime Errors

### Q: Segmentation Fault / Access Violation

**Common Causes:**

```cpp
// ❌ CAUSE 1: Null pointer dereference
int* ptr = nullptr;
*ptr = 42;  // SEGFAULT!

// ✅ SOLUTION: Check before dereferencing
if (ptr != nullptr) {
    *ptr = 42;
}

// ❌ CAUSE 2: Dangling pointer
int* ptr;
{
    int x = 42;
    ptr = &x;
}  // x destroyed
*ptr = 100;  // SEGFAULT!

// ✅ SOLUTION: Don't store pointers to locals
int* ptr = new int(42);  // Or use smart pointers

// ❌ CAUSE 3: Array out of bounds
int arr[5];
arr[10] = 42;  // SEGFAULT!

// ✅ SOLUTION: Use std::vector with at()
std::vector<int> vec(5);
vec.at(10) = 42;  // Throws exception instead
```

**Debugging Segfaults:**

```bash
# Enable core dumps
ulimit -c unlimited

# Run with debugger
gdb ./myapp
(gdb) run
# Program crashes...
(gdb) backtrace  # See where it crashed

# Use AddressSanitizer
g++ -fsanitize=address -g main.cpp
./a.out  # Shows exact line of error
```

### Q: Program Hangs / Deadlock

**Common Causes:**

```cpp
// ❌ CAUSE: Deadlock
std::mutex m1, m2;

void thread1() {
    m1.lock();
    std::this_thread::sleep_for(10ms);
    m2.lock();  // Waits for thread2
}

void thread2() {
    m2.lock();
    std::this_thread::sleep_for(10ms);
    m1.lock();  // Waits for thread1
}

// ✅ SOLUTION: Consistent lock ordering
void thread1() {
    std::scoped_lock lock(m1, m2);  // Atomic
}

void thread2() {
    std::scoped_lock lock(m1, m2);  // Same order
}
```

**Debugging Hangs:**

```bash
# Attach debugger to running process
gdb -p <pid>
(gdb) thread apply all bt  # Backtrace all threads

# Use ThreadSanitizer
g++ -fsanitize=thread main.cpp
```

### Q: Memory Corruption

**Symptoms:**
- Random crashes
- Incorrect values
- Heap corruption errors

**Common Causes:**

```cpp
// ❌ Buffer overflow
char buffer[10];
strcpy(buffer, "This is a very long string");  // Overflow!

// ✅ Use safe functions
char buffer[10];
strncpy(buffer, str, sizeof(buffer) - 1);
buffer[9] = '\0';

// ✅ Or use std::string
std::string s = "Any length is fine";

// ❌ Use after free
int* ptr = new int(42);
delete ptr;
*ptr = 100;  // Undefined behavior!

// ✅ Use smart pointers
auto ptr = std::make_unique<int>(42);
// Automatic cleanup
```

**Detection:**

```bash
# Valgrind memcheck
valgrind --leak-check=full ./myapp

# AddressSanitizer
g++ -fsanitize=address -g main.cpp
```

---

## 20.3 Performance Issues

### Q: Code Runs Slowly

**Diagnostic Steps:**

```bash
# 1. Profile first!
perf record ./myapp
perf report

# 2. Check compilation flags
g++ -O0 main.cpp  # ❌ Debug build
g++ -O3 main.cpp  # ✅ Release build

# 3. Look for algorithmic issues
# O(n²) when O(n log n) possible?
```

**Common Issues:**

```cpp
// ❌ Unnecessary copies
void process(std::vector<int> vec) {  // Copy!
    for (auto item : vec) {  // Another copy!
        // ...
    }
}

// ✅ Use references
void process(const std::vector<int>& vec) {
    for (const auto& item : vec) {
        // ...
    }
}

// ❌ Wrong data structure
std::vector<int> numbers;
// Linear search O(n)
auto it = std::find(numbers.begin(), numbers.end(), target);

// ✅ Use hash table for lookups
std::unordered_set<int> numbers;
// O(1) lookup
auto it = numbers.find(target);
```

### Q: High Memory Usage

**Diagnostic:**

```bash
# Memory profiling
valgrind --tool=massif ./myapp
ms_print massif.out.12345

# Check for leaks
valgrind --leak-check=full ./myapp
```

**Common Issues:**

```cpp
// ❌ Memory leak
void function() {
    int* data = new int[1000];
    // Forgot to delete!
}

// ✅ Use RAII
void function() {
    auto data = std::make_unique<int[]>(1000);
    // Automatic cleanup
}

// ❌ Unbounded growth
std::vector<int> cache;
while (true) {
    cache.push_back(get_data());  // Never shrinks!
}

// ✅ Implement size limit
if (cache.size() > MAX_SIZE) {
    cache.erase(cache.begin());
}
```

---

## 20.4 Build System Issues

### Q: CMake Cannot Find Package

**Problem:**
```
CMake Error: Could not find package Boost
```

**Solutions:**

```cmake
# Set package search path
set(CMAKE_PREFIX_PATH "/path/to/boost")
find_package(Boost REQUIRED)

# Or use environment variable
export CMAKE_PREFIX_PATH=/path/to/boost

# Specify version
find_package(Boost 1.70 REQUIRED)

# Check available components
find_package(Boost COMPONENTS filesystem system)
```

### Q: Header Not Found

**Problem:**
```
fatal error: myheader.h: No such file or directory
```

**Solutions:**

```cmake
# Add include directory
target_include_directories(myapp PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# Or in compilation
g++ -I./include main.cpp
```

### Q: Undefined Symbols When Linking

**Problem:**
```
undefined reference to `boost::filesystem::path::path()'
```

**Solutions:**

```cmake
# Link libraries
find_package(Boost REQUIRED COMPONENTS filesystem)
target_link_libraries(myapp Boost::filesystem)

# Or directly
g++ main.cpp -lboost_filesystem
```

---

## 20.5 Modern C++ Questions

### Q: When to Use auto?

**Guidelines:**

```cpp
// ✅ Use auto for complex types
auto it = map.begin();  // Instead of map<K,V>::iterator

// ✅ Use auto with templates
auto ptr = std::make_unique<Widget>();

// ✅ Use auto in range-for
for (const auto& item : container) { }

// ❌ Don't use when type is unclear
auto x = get_value();  // What type is x?

// ✅ Be explicit for clarity
int count = get_count();
```

### Q: unique_ptr vs shared_ptr?

**Decision Tree:**

```cpp
// Single owner? → unique_ptr
auto ptr = std::make_unique<Widget>();

// Multiple owners? → shared_ptr
auto ptr = std::make_shared<Widget>();
auto ptr2 = ptr;  // Both own it

// Observer without ownership? → weak_ptr
std::weak_ptr<Widget> observer = ptr;

// Non-owning reference? → raw pointer
Widget* ref = ptr.get();
```

### Q: When to Use move()?

**Guidelines:**

```cpp
// ✅ Transferring ownership
std::unique_ptr<int> ptr1 = std::make_unique<int>(42);
std::unique_ptr<int> ptr2 = std::move(ptr1);

// ✅ Last use of expensive object
std::vector<int> vec = create_large_vector();
process(std::move(vec));  // vec not used after

// ❌ Don't move in return statements
std::vector<int> create() {
    std::vector<int> result;
    return std::move(result);  // ❌ Prevents RVO!
}

// ✅ Let compiler optimize
std::vector<int> create() {
    std::vector<int> result;
    return result;  // ✅ RVO or move
}
```

---

## 20.6 Debugging Techniques

### Print Debugging

```cpp
// Quick and dirty
std::cout << "Debug: x = " << x << "\n";

// Better: Conditional compilation
#ifdef DEBUG
    std::cout << "Debug: " << var << "\n";
#endif

// Best: Use logging library
#include <spdlog/spdlog.h>
spdlog::debug("Value: {}", x);
```

### GDB Techniques

```bash
# Print STL containers
(gdb) print vector
# Use pretty-printing
(gdb) set print pretty on

# Conditional breakpoints
(gdb) break function if x > 100

# Watch variable changes
(gdb) watch variable

# Examine memory
(gdb) x/10x address
```

### Assert Usage

```cpp
#include <cassert>

void process(int* ptr) {
    assert(ptr != nullptr);  // Debug only
    // ...
}

// Runtime assertions
if (ptr == nullptr) {
    throw std::invalid_argument("Null pointer");
}
```

---

## 20.7 Cross-Platform Issues

### Q: Code Works on Linux but Not Windows

**Common Issues:**

```cpp
// ❌ Path separators
std::string path = "data/file.txt";  // Works on Unix
std::string path = "data\\file.txt"; // Works on Windows

// ✅ Use std::filesystem (C++17)
#include <filesystem>
namespace fs = std::filesystem;
auto path = fs::path("data") / "file.txt";

// ❌ Case-sensitive file systems
// Linux: file.txt ≠ File.txt
// Windows: file.txt = File.txt

// ✅ Use consistent naming
```

### Q: Different Compiler Behavior

```cpp
// Some compilers are more strict
// Use consistent warning flags
g++ -Wall -Wextra -pedantic main.cpp
clang++ -Wall -Wextra -pedantic main.cpp
```

---

## 20.8 Quick Reference

### Common Error Messages

| Error | Meaning | Solution |
|-------|---------|----------|
| `undefined reference` | Missing implementation | Add .cpp or link library |
| `no matching function` | Wrong arguments | Check function signature |
| `cannot convert` | Type mismatch | Add explicit conversion |
| `segmentation fault` | Invalid memory access | Check pointers/bounds |
| `pure virtual function` | Missing override | Implement virtual function |
| `ambiguous call` | Multiple matches | Make call explicit |

### Debugging Checklist

- [ ] Compiled with `-g` for debug symbols?
- [ ] Tried with AddressSanitizer?
- [ ] Checked with Valgrind?
- [ ] Added print statements?
- [ ] Used debugger (GDB/LLDB)?
- [ ] Checked recent changes?
- [ ] Simplified to minimal example?

---

## Conclusion

Part 20 covered practical troubleshooting:

1. **Compilation Errors** - Linker, templates, conversions
2. **Runtime Errors** - Segfaults, hangs, corruption
3. **Performance** - Slow code, memory usage
4. **Build Systems** - CMake, dependencies
5. **Modern C++** - auto, smart pointers, move
6. **Debugging** - Techniques and tools
7. **Cross-Platform** - Portability issues

**Problem-Solving Process:**
1. Reproduce the issue
2. Isolate to minimal example
3. Use appropriate tool (debugger, sanitizer, profiler)
4. Fix root cause, not symptom
5. Add test to prevent regression

**Essential Tools:**
- **Debugger**: GDB or LLDB
- **Sanitizers**: AddressSanitizer, ThreadSanitizer
- **Profiler**: perf or Valgrind
- **Static Analyzer**: clang-tidy

**Next:** Part 21 - Master Cheat Sheet (Final Part!)

---

*This blog post is part of the Complete C++ Mastery series.*
