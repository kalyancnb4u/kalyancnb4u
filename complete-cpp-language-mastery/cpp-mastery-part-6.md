---
title: "Complete C++ Mastery Part 6: Concurrency & Multithreading"
date: 2024-02-20 20:00:00 +0000
categories: [C++, Programming, Concurrency]
tags: [cpp, multithreading, concurrency, threads, mutex, atomic, async, parallel]
---

# Complete C++ Mastery Part 6: Concurrency & Multithreading

## Introduction

Welcome to Part 6 of the Complete C++ Mastery series. Modern applications demand concurrency to fully utilize multi-core processors and handle multiple tasks simultaneously. C++11 introduced a standardized threading model, making concurrent programming portable across platforms. This part explores the complete concurrency toolkit: threads, synchronization primitives, atomic operations, the memory model, and high-level abstractions.

Concurrency is challenging—data races, deadlocks, and race conditions can introduce subtle, hard-to-debug bugs. This part emphasizes correct concurrent programming, common pitfalls, and best practices for writing safe, efficient parallel code.

**Prerequisites:** This part assumes knowledge from Parts 1-5, especially RAII, move semantics, templates, and STL.

---

## 6.1 Threads and Basic Synchronization

### Creating and Managing Threads

```cpp
#include <iostream>
#include <thread>

void hello() {
    std::cout << "Hello from thread " 
              << std::this_thread::get_id() << "\n";
}

void count_to(int n) {
    for (int i = 0; i < n; ++i) {
        std::cout << "Count: " << i << "\n";
    }
}

int main() {
    // Create thread with function
    std::thread t1(hello);
    
    // Create thread with function and arguments
    std::thread t2(count_to, 5);
    
    // Create thread with lambda
    std::thread t3([]() {
        std::cout << "Lambda thread\n";
    });
    
    // Must join or detach before destruction
    t1.join();  // Wait for t1 to finish
    t2.join();
    t3.join();
    
    std::cout << "Main thread finished\n";
    
    return 0;
}
```

**Thread Lifecycle:**
```cpp
#include <iostream>
#include <thread>
#include <chrono>

void task(int id) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Task " << id << " completed\n";
}

int main() {
    std::thread t(task, 1);
    
    // Check if joinable
    if (t.joinable()) {
        std::cout << "Thread is joinable\n";
    }
    
    // Get thread ID
    std::cout << "Thread ID: " << t.get_id() << "\n";
    
    // Get hardware concurrency
    std::cout << "Hardware threads: " 
              << std::thread::hardware_concurrency() << "\n";
    
    // Join thread
    t.join();
    
    // After join, no longer joinable
    if (!t.joinable()) {
        std::cout << "Thread is no longer joinable\n";
    }
    
    return 0;
}
```

**Detached Threads:**
```cpp
#include <iostream>
#include <thread>
#include <chrono>

void background_task() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Background task completed\n";
}

int main() {
    std::thread t(background_task);
    
    // Detach thread (runs independently)
    t.detach();
    
    // Main thread doesn't wait for detached thread
    std::cout << "Main thread continuing...\n";
    
    // ⚠️ If main exits before detached thread finishes,
    // detached thread is terminated!
    std::this_thread::sleep_for(std::chrono::seconds(3));
    
    return 0;
}
```

**RAII Thread Wrapper:**
```cpp
#include <iostream>
#include <thread>

class ThreadGuard {
    std::thread& t;
    
public:
    explicit ThreadGuard(std::thread& t_) : t(t_) {}
    
    ~ThreadGuard() {
        if (t.joinable()) {
            t.join();
        }
    }
    
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;
};

void task() {
    std::cout << "Task executed\n";
}

int main() {
    std::thread t(task);
    ThreadGuard guard(t);
    
    // Even if exception thrown, destructor joins thread
    // throw std::runtime_error("error");
    
    return 0;  // Thread automatically joined
}
```

### Race Conditions

```cpp
#include <iostream>
#include <thread>
#include <vector>

// ❌ BAD: Race condition
int counter = 0;

void increment_unsafe() {
    for (int i = 0; i < 100000; ++i) {
        ++counter;  // ⚠️ Not atomic! Race condition!
    }
}

int main() {
    std::vector<std::thread> threads;
    
    for (int i = 0; i < 10; ++i) {
        threads.push_back(std::thread(increment_unsafe));
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    // Expected: 1000000, Actual: varies (race condition)
    std::cout << "Counter: " << counter << "\n";
    
    return 0;
}
```

### std::mutex - Mutual Exclusion

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

std::mutex mtx;
int counter = 0;

void increment_safe() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();
        ++counter;
        mtx.unlock();
    }
}

// ✅ BETTER: RAII with lock_guard
void increment_raii() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        ++counter;
        // Automatically unlocked when lock goes out of scope
    }
}

int main() {
    std::vector<std::thread> threads;
    
    for (int i = 0; i < 10; ++i) {
        threads.push_back(std::thread(increment_raii));
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "Counter: " << counter << "\n";  // Always 1000000
    
    return 0;
}
```

**std::lock_guard vs std::unique_lock:**
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;

void use_lock_guard() {
    std::lock_guard<std::mutex> lock(mtx);
    // Simple RAII lock
    // Cannot unlock early
    // Cannot transfer ownership
}

void use_unique_lock() {
    std::unique_lock<std::mutex> lock(mtx);
    // More flexible than lock_guard
    
    // Can unlock early
    lock.unlock();
    
    // Can relock
    lock.lock();
    
    // Can transfer ownership
    std::unique_lock<std::mutex> lock2 = std::move(lock);
    
    // Deferred locking
    std::unique_lock<std::mutex> lock3(mtx, std::defer_lock);
    // ... do work without lock
    lock3.lock();  // Lock when needed
}
```

### Deadlock

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx1, mtx2;

// ❌ BAD: Potential deadlock
void thread1_deadlock() {
    mtx1.lock();
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    mtx2.lock();  // ⚠️ May deadlock if thread2 holds mtx2
    
    std::cout << "Thread 1\n";
    
    mtx2.unlock();
    mtx1.unlock();
}

void thread2_deadlock() {
    mtx2.lock();
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    mtx1.lock();  // ⚠️ May deadlock if thread1 holds mtx1
    
    std::cout << "Thread 2\n";
    
    mtx1.unlock();
    mtx2.unlock();
}

// ✅ GOOD: Avoid deadlock with std::lock
void thread1_safe() {
    std::lock(mtx1, mtx2);  // Atomic lock of multiple mutexes
    std::lock_guard<std::mutex> lock1(mtx1, std::adopt_lock);
    std::lock_guard<std::mutex> lock2(mtx2, std::adopt_lock);
    
    std::cout << "Thread 1 (safe)\n";
}

void thread2_safe() {
    std::lock(mtx1, mtx2);
    std::lock_guard<std::mutex> lock1(mtx1, std::adopt_lock);
    std::lock_guard<std::mutex> lock2(mtx2, std::adopt_lock);
    
    std::cout << "Thread 2 (safe)\n";
}

// ✅ BETTER: C++17 scoped_lock
void thread_scoped_lock() {
    std::scoped_lock lock(mtx1, mtx2);  // RAII for multiple mutexes
    std::cout << "Thread (scoped_lock)\n";
}
```

### std::recursive_mutex

Allows same thread to lock multiple times:

```cpp
#include <iostream>
#include <mutex>
#include <thread>

class Counter {
    std::recursive_mutex mtx;
    int count = 0;
    
public:
    void increment() {
        std::lock_guard<std::recursive_mutex> lock(mtx);
        ++count;
    }
    
    void increment_twice() {
        std::lock_guard<std::recursive_mutex> lock(mtx);
        increment();  // OK: Can lock again in same thread
        increment();
    }
    
    int get_count() {
        std::lock_guard<std::recursive_mutex> lock(mtx);
        return count;
    }
};

int main() {
    Counter counter;
    
    std::thread t([&counter]() {
        counter.increment_twice();
    });
    
    t.join();
    
    std::cout << "Count: " << counter.get_count() << "\n";
    
    return 0;
}
```

### std::timed_mutex

Attempt to acquire lock with timeout:

```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

std::timed_mutex tmtx;

void worker(int id) {
    // Try to lock for 100ms
    if (tmtx.try_lock_for(std::chrono::milliseconds(100))) {
        std::cout << "Thread " << id << " acquired lock\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(200));
        tmtx.unlock();
    } else {
        std::cout << "Thread " << id << " timeout\n";
    }
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);
    
    t1.join();
    t2.join();
    
    return 0;
}
```

---

## 6.2 Condition Variables

### std::condition_variable

Allows threads to wait for specific conditions:

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> queue;
bool finished = false;

void producer() {
    for (int i = 0; i < 10; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        
        {
            std::lock_guard<std::mutex> lock(mtx);
            queue.push(i);
            std::cout << "Produced: " << i << "\n";
        }
        
        cv.notify_one();  // Wake up one waiting thread
    }
    
    {
        std::lock_guard<std::mutex> lock(mtx);
        finished = true;
    }
    cv.notify_all();  // Wake up all waiting threads
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        
        // Wait until queue has data or production finished
        cv.wait(lock, []() {
            return !queue.empty() || finished;
        });
        
        if (!queue.empty()) {
            int value = queue.front();
            queue.pop();
            lock.unlock();
            
            std::cout << "Consumer " << id << " consumed: " << value << "\n";
        } else if (finished) {
            break;
        }
    }
}

int main() {
    std::thread prod(producer);
    std::thread cons1(consumer, 1);
    std::thread cons2(consumer, 2);
    
    prod.join();
    cons1.join();
    cons2.join();
    
    return 0;
}
```

**Condition Variable Spurious Wakeups:**
```cpp
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void wait_for_signal() {
    std::unique_lock<std::mutex> lock(mtx);
    
    // ❌ BAD: Doesn't handle spurious wakeups
    cv.wait(lock);
    if (ready) {
        // Process...
    }
    
    // ✅ GOOD: Predicate handles spurious wakeups
    cv.wait(lock, []() { return ready; });
    // Process...
}
```

### Producer-Consumer Pattern

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>

template<typename T>
class ThreadSafeQueue {
    std::queue<T> queue;
    std::mutex mtx;
    std::condition_variable cv;
    
public:
    void push(T value) {
        std::lock_guard<std::mutex> lock(mtx);
        queue.push(std::move(value));
        cv.notify_one();
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
    
    void wait_and_pop(T& value) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return !queue.empty(); });
        value = std::move(queue.front());
        queue.pop();
    }
    
    bool empty() const {
        std::lock_guard<std::mutex> lock(mtx);
        return queue.empty();
    }
};

int main() {
    ThreadSafeQueue<int> queue;
    
    // Producer
    std::thread producer([&queue]() {
        for (int i = 0; i < 10; ++i) {
            queue.push(i);
            std::cout << "Produced: " << i << "\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    });
    
    // Consumer
    std::thread consumer([&queue]() {
        for (int i = 0; i < 10; ++i) {
            int value;
            queue.wait_and_pop(value);
            std::cout << "Consumed: " << value << "\n";
        }
    });
    
    producer.join();
    consumer.join();
    
    return 0;
}
```

---

## 6.3 Atomic Operations

### std::atomic

Lock-free atomic operations:

```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <vector>

std::atomic<int> counter(0);

void increment_atomic() {
    for (int i = 0; i < 100000; ++i) {
        ++counter;  // Atomic increment
    }
}

int main() {
    std::vector<std::thread> threads;
    
    for (int i = 0; i < 10; ++i) {
        threads.push_back(std::thread(increment_atomic));
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "Counter: " << counter << "\n";  // Always 1000000
    
    return 0;
}
```

**Atomic Operations:**
```cpp
#include <iostream>
#include <atomic>

int main() {
    std::atomic<int> x(0);
    
    // Atomic operations
    x = 5;                    // Atomic store
    int val = x;              // Atomic load
    ++x;                      // Atomic increment
    x += 10;                  // Atomic add
    
    // Compare-and-swap
    int expected = 15;
    bool success = x.compare_exchange_strong(expected, 20);
    // If x == expected, set x = 20 and return true
    // Otherwise, set expected = x and return false
    
    std::cout << "CAS success: " << success << "\n";
    std::cout << "Value: " << x << "\n";
    
    // Fetch-and-modify operations
    int old = x.fetch_add(5);  // Returns old value, adds 5
    std::cout << "Old: " << old << ", New: " << x << "\n";
    
    // Check if lock-free
    std::cout << "Is lock-free: " << x.is_lock_free() << "\n";
    
    return 0;
}
```

**Atomic Flags (Spinlock):**
```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <vector>

class Spinlock {
    std::atomic_flag flag = ATOMIC_FLAG_INIT;
    
public:
    void lock() {
        while (flag.test_and_set(std::memory_order_acquire)) {
            // Spin until flag is clear
        }
    }
    
    void unlock() {
        flag.clear(std::memory_order_release);
    }
};

Spinlock spinlock;
int counter = 0;

void increment_with_spinlock() {
    for (int i = 0; i < 100000; ++i) {
        spinlock.lock();
        ++counter;
        spinlock.unlock();
    }
}

int main() {
    std::vector<std::thread> threads;
    
    for (int i = 0; i < 10; ++i) {
        threads.push_back(std::thread(increment_with_spinlock));
    }
    
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "Counter: " << counter << "\n";
    
    return 0;
}
```

### Memory Ordering

```cpp
#include <iostream>
#include <atomic>
#include <thread>

std::atomic<int> x(0), y(0);
int r1, r2;

void thread1() {
    x.store(1, std::memory_order_relaxed);
    r1 = y.load(std::memory_order_relaxed);
}

void thread2() {
    y.store(1, std::memory_order_relaxed);
    r2 = x.load(std::memory_order_relaxed);
}

int main() {
    std::thread t1(thread1);
    std::thread t2(thread2);
    
    t1.join();
    t2.join();
    
    // With relaxed ordering, both r1 and r2 can be 0!
    std::cout << "r1: " << r1 << ", r2: " << r2 << "\n";
    
    return 0;
}
```

**Memory Orders:**
```cpp
// memory_order_relaxed: No synchronization, only atomicity
std::atomic<int> x;
x.store(1, std::memory_order_relaxed);
int val = x.load(std::memory_order_relaxed);

// memory_order_acquire/release: Synchronizes with release
std::atomic<int> flag;
// Thread 1
data = 42;
flag.store(1, std::memory_order_release);
// Thread 2
while (flag.load(std::memory_order_acquire) == 0);
assert(data == 42);  // Guaranteed

// memory_order_seq_cst: Sequential consistency (default, safest)
x.store(1, std::memory_order_seq_cst);
int val2 = x.load(std::memory_order_seq_cst);

// memory_order_consume: Like acquire but weaker (rarely used)
// memory_order_acq_rel: Both acquire and release
// Used with compare_exchange operations
```

---

## 6.4 High-Level Concurrency

### std::async and std::future

```cpp
#include <iostream>
#include <future>
#include <chrono>

int compute(int x) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

int main() {
    // Launch async task
    std::future<int> result = std::async(std::launch::async, compute, 10);
    
    // Do other work while computation runs
    std::cout << "Computing...\n";
    
    // Get result (blocks until ready)
    int value = result.get();
    std::cout << "Result: " << value << "\n";
    
    return 0;
}
```

**Launch Policies:**
```cpp
#include <iostream>
#include <future>

int task() {
    return 42;
}

int main() {
    // async: Guaranteed to run asynchronously
    auto f1 = std::async(std::launch::async, task);
    
    // deferred: Run synchronously when result requested
    auto f2 = std::async(std::launch::deferred, task);
    
    // async | deferred: Implementation chooses
    auto f3 = std::async(std::launch::async | std::launch::deferred, task);
    
    // Default: async | deferred
    auto f4 = std::async(task);
    
    std::cout << "Results: " << f1.get() << " " 
              << f2.get() << " " << f3.get() << "\n";
    
    return 0;
}
```

**Multiple Async Tasks:**
```cpp
#include <iostream>
#include <future>
#include <vector>

int compute(int x) {
    return x * x;
}

int main() {
    std::vector<std::future<int>> futures;
    
    // Launch multiple tasks
    for (int i = 1; i <= 10; ++i) {
        futures.push_back(std::async(std::launch::async, compute, i));
    }
    
    // Collect results
    for (auto& f : futures) {
        std::cout << f.get() << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### std::promise

Manually set future value:

```cpp
#include <iostream>
#include <future>
#include <thread>

void compute_promise(std::promise<int> prom, int x) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    prom.set_value(x * x);
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    
    std::thread t(compute_promise, std::move(prom), 10);
    
    std::cout << "Waiting for result...\n";
    int result = fut.get();
    std::cout << "Result: " << result << "\n";
    
    t.join();
    
    return 0;
}
```

**Exception Propagation:**
```cpp
#include <iostream>
#include <future>
#include <stdexcept>

void task_with_exception(std::promise<int> prom) {
    try {
        throw std::runtime_error("Task failed");
    } catch (...) {
        prom.set_exception(std::current_exception());
    }
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    
    std::thread t(task_with_exception, std::move(prom));
    
    try {
        int result = fut.get();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << "\n";
    }
    
    t.join();
    
    return 0;
}
```

### std::packaged_task

Wrapper for callable objects that produces future:

```cpp
#include <iostream>
#include <future>
#include <thread>

int compute(int x) {
    return x * x;
}

int main() {
    // Create packaged_task
    std::packaged_task<int(int)> task(compute);
    std::future<int> result = task.get_future();
    
    // Run in separate thread
    std::thread t(std::move(task), 10);
    
    // Get result
    std::cout << "Result: " << result.get() << "\n";
    
    t.join();
    
    return 0;
}
```

---

## 6.5 Thread-Local Storage

```cpp
#include <iostream>
#include <thread>

thread_local int tls_counter = 0;

void increment() {
    for (int i = 0; i < 5; ++i) {
        ++tls_counter;
        std::cout << "Thread " << std::this_thread::get_id() 
                  << ": " << tls_counter << "\n";
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    
    t1.join();
    t2.join();
    
    // Each thread has its own copy
    std::cout << "Main thread: " << tls_counter << "\n";  // 0
    
    return 0;
}
```

---

## 6.6 Parallel Algorithms (C++17)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <execution>
#include <chrono>

int main() {
    std::vector<int> vec(10000000);
    std::iota(vec.begin(), vec.end(), 1);
    
    // Sequential
    auto start = std::chrono::high_resolution_clock::now();
    std::sort(vec.begin(), vec.end());
    auto end = std::chrono::high_resolution_clock::now();
    auto seq_time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    
    // Parallel
    std::random_shuffle(vec.begin(), vec.end());
    start = std::chrono::high_resolution_clock::now();
    std::sort(std::execution::par, vec.begin(), vec.end());
    end = std::chrono::high_resolution_clock::now();
    auto par_time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    
    std::cout << "Sequential: " << seq_time.count() << "ms\n";
    std::cout << "Parallel: " << par_time.count() << "ms\n";
    
    return 0;
}
```

**Execution Policies:**
```cpp
#include <algorithm>
#include <execution>
#include <vector>

std::vector<int> vec = {3, 1, 4, 1, 5, 9, 2, 6};

// sequenced_policy: Sequential execution
std::sort(std::execution::seq, vec.begin(), vec.end());

// parallel_policy: Parallel execution
std::sort(std::execution::par, vec.begin(), vec.end());

// parallel_unsequenced_policy: Parallel + vectorized
std::sort(std::execution::par_unseq, vec.begin(), vec.end());

// unsequenced_policy: Vectorized (C++20)
// std::sort(std::execution::unseq, vec.begin(), vec.end());
```

[Due to length constraints, this is approximately 60% of Part 6. Would you like me to continue with the remaining sections on common patterns, best practices, and a comprehensive example?]

---

## 6.7 Common Concurrency Patterns

### Thread Pool

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>

class ThreadPool {
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop = false;
    
public:
    ThreadPool(size_t num_threads) {
        for (size_t i = 0; i < num_threads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    
                    {
                        std::unique_lock<std::mutex> lock(queue_mutex);
                        condition.wait(lock, [this] {
                            return stop || !tasks.empty();
                        });
                        
                        if (stop && tasks.empty()) {
                            return;
                        }
                        
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    
                    task();
                }
            });
        }
    }
    
    template<typename F, typename... Args>
    auto enqueue(F&& f, Args&&... args) 
        -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
        std::future<return_type> result = task->get_future();
        
        {
            std::lock_guard<std::mutex> lock(queue_mutex);
            if (stop) {
                throw std::runtime_error("enqueue on stopped ThreadPool");
            }
            tasks.emplace([task]() { (*task)(); });
        }
        
        condition.notify_one();
        return result;
    }
    
    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(queue_mutex);
            stop = true;
        }
        
        condition.notify_all();
        
        for (std::thread& worker : workers) {
            worker.join();
        }
    }
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;
    
    for (int i = 0; i < 8; ++i) {
        results.emplace_back(pool.enqueue([i] {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            std::cout << "Task " << i << " completed\n";
            return i * i;
        }));
    }
    
    for (auto& result : results) {
        std::cout << "Result: " << result.get() << "\n";
    }
    
    return 0;
}
```

### Reader-Writer Lock

```cpp
#include <iostream>
#include <thread>
#include <shared_mutex>
#include <vector>

class SharedData {
    mutable std::shared_mutex mutex;
    int value = 0;
    
public:
    // Multiple readers can acquire shared lock simultaneously
    int read() const {
        std::shared_lock<std::shared_mutex> lock(mutex);
        return value;
    }
    
    // Only one writer can acquire exclusive lock
    void write(int new_value) {
        std::unique_lock<std::shared_mutex> lock(mutex);
        value = new_value;
    }
};

int main() {
    SharedData data;
    
    // Multiple reader threads
    std::vector<std::thread> readers;
    for (int i = 0; i < 5; ++i) {
        readers.emplace_back([&data, i] {
            for (int j = 0; j < 10; ++j) {
                int val = data.read();
                std::cout << "Reader " << i << " read: " << val << "\n";
                std::this_thread::sleep_for(std::chrono::milliseconds(100));
            }
        });
    }
    
    // Writer thread
    std::thread writer([&data] {
        for (int i = 0; i < 10; ++i) {
            data.write(i);
            std::cout << "Writer wrote: " << i << "\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(200));
        }
    });
    
    for (auto& t : readers) {
        t.join();
    }
    writer.join();
    
    return 0;
}
```

### Double-Checked Locking (Singleton)

```cpp
#include <iostream>
#include <mutex>
#include <atomic>

class Singleton {
    static std::atomic<Singleton*> instance;
    static std::mutex mtx;
    
    Singleton() {
        std::cout << "Singleton created\n";
    }
    
public:
    // Double-checked locking pattern
    static Singleton* get_instance() {
        Singleton* tmp = instance.load(std::memory_order_acquire);
        if (tmp == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            tmp = instance.load(std::memory_order_relaxed);
            if (tmp == nullptr) {
                tmp = new Singleton();
                instance.store(tmp, std::memory_order_release);
            }
        }
        return tmp;
    }
    
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};

std::atomic<Singleton*> Singleton::instance{nullptr};
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

---

## 6.8 Common Pitfalls and Best Practices

### Data Races

```cpp
// ❌ BAD: Data race
int shared_value = 0;

void thread1() { shared_value = 1; }
void thread2() { shared_value = 2; }

// ✅ GOOD: Protected by mutex
std::mutex mtx;
int protected_value = 0;

void safe_thread1() {
    std::lock_guard<std::mutex> lock(mtx);
    protected_value = 1;
}

void safe_thread2() {
    std::lock_guard<std::mutex> lock(mtx);
    protected_value = 2;
}

// ✅ BETTER: Use atomic
std::atomic<int> atomic_value{0};

void atomic_thread1() { atomic_value = 1; }
void atomic_thread2() { atomic_value = 2; }
```

### Deadlock Prevention

```cpp
#include <mutex>
#include <thread>

std::mutex m1, m2;

// ❌ BAD: Can deadlock
void bad_func1() {
    std::lock_guard<std::mutex> lock1(m1);
    std::lock_guard<std::mutex> lock2(m2);
}

void bad_func2() {
    std::lock_guard<std::mutex> lock2(m2);
    std::lock_guard<std::mutex> lock1(m1);
}

// ✅ GOOD: Consistent lock order
void good_func1() {
    std::lock_guard<std::mutex> lock1(m1);
    std::lock_guard<std::mutex> lock2(m2);
}

void good_func2() {
    std::lock_guard<std::mutex> lock1(m1);
    std::lock_guard<std::mutex> lock2(m2);
}

// ✅ BETTER: std::lock or scoped_lock
void best_func() {
    std::scoped_lock lock(m1, m2);  // C++17
}
```

### False Sharing

```cpp
#include <thread>
#include <vector>
#include <atomic>

// ❌ BAD: False sharing (counters on same cache line)
struct BadCounters {
    std::atomic<int> counter1;
    std::atomic<int> counter2;
};

// ✅ GOOD: Pad to separate cache lines
struct alignas(64) GoodCounters {
    std::atomic<int> counter1;
    char padding[60];  // Ensure different cache lines
    std::atomic<int> counter2;
};

// Modern alternative: C++17 hardware_destructive_interference_size
#include <new>

struct ModernCounters {
    alignas(std::hardware_destructive_interference_size)
    std::atomic<int> counter1;
    
    alignas(std::hardware_destructive_interference_size)
    std::atomic<int> counter2;
};
```

### Thread Exceptions

```cpp
#include <iostream>
#include <thread>
#include <exception>

void throwing_function() {
    throw std::runtime_error("Thread exception");
}

int main() {
    try {
        std::thread t(throwing_function);
        t.join();
    } catch (const std::exception& e) {
        // ⚠️ Won't catch! Exception thrown in different thread
        std::cout << "Caught: " << e.what() << "\n";
    }
    
    // ✅ Proper exception handling
    std::exception_ptr exception;
    
    std::thread t([&exception] {
        try {
            throw std::runtime_error("Thread exception");
        } catch (...) {
            exception = std::current_exception();
        }
    });
    
    t.join();
    
    if (exception) {
        try {
            std::rethrow_exception(exception);
        } catch (const std::exception& e) {
            std::cout << "Caught: " << e.what() << "\n";
        }
    }
    
    return 0;
}
```

### Best Practices Checklist

```cpp
// ✅ Use RAII for locks (lock_guard, unique_lock, scoped_lock)
{
    std::lock_guard<std::mutex> lock(mtx);
    // Critical section
}

// ✅ Prefer atomic operations over mutexes for simple counters
std::atomic<int> counter{0};
++counter;  // No mutex needed

// ✅ Use condition variables for waiting, not busy loops
std::condition_variable cv;
std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock, []{ return ready; });

// ✅ Avoid nested locks when possible
// If unavoidable, use consistent ordering or std::lock

// ✅ Keep critical sections small
{
    std::lock_guard<std::mutex> lock(mtx);
    // Minimal work here
}
// Expensive work outside lock

// ✅ Use thread_local for thread-specific data
thread_local int thread_id = 0;

// ✅ Prefer high-level abstractions (async, future, parallel algorithms)
auto result = std::async([]{ return compute(); });

// ✅ Test concurrent code thoroughly
// Use thread sanitizers, stress tests, different thread counts
```

---

## Comprehensive Example: Concurrent Task Scheduler

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <chrono>
#include <atomic>

class TaskScheduler {
public:
    using Task = std::function<void()>;
    
    enum class Priority {
        LOW,
        NORMAL,
        HIGH
    };
    
private:
    struct QueuedTask {
        Task task;
        Priority priority;
        std::chrono::steady_clock::time_point scheduled_time;
        
        bool operator<(const QueuedTask& other) const {
            if (priority != other.priority) {
                return priority < other.priority;
            }
            return scheduled_time > other.scheduled_time;
        }
    };
    
    std::priority_queue<QueuedTask> task_queue;
    std::vector<std::thread> workers;
    std::mutex queue_mutex;
    std::condition_variable condition;
    std::atomic<bool> stop{false};
    std::atomic<int> active_tasks{0};
    
public:
    explicit TaskScheduler(size_t num_threads) {
        for (size_t i = 0; i < num_threads; ++i) {
            workers.emplace_back([this] { worker_thread(); });
        }
    }
    
    void schedule(Task task, Priority priority = Priority::NORMAL,
                  std::chrono::milliseconds delay = std::chrono::milliseconds(0)) {
        auto scheduled_time = std::chrono::steady_clock::now() + delay;
        
        {
            std::lock_guard<std::mutex> lock(queue_mutex);
            task_queue.push({std::move(task), priority, scheduled_time});
        }
        
        condition.notify_one();
    }
    
    void wait_all() {
        std::unique_lock<std::mutex> lock(queue_mutex);
        condition.wait(lock, [this] {
            return task_queue.empty() && active_tasks == 0;
        });
    }
    
    ~TaskScheduler() {
        stop = true;
        condition.notify_all();
        
        for (auto& worker : workers) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }
    
private:
    void worker_thread() {
        while (!stop) {
            QueuedTask queued_task;
            
            {
                std::unique_lock<std::mutex> lock(queue_mutex);
                
                condition.wait(lock, [this] {
                    return stop || !task_queue.empty();
                });
                
                if (stop && task_queue.empty()) {
                    return;
                }
                
                auto now = std::chrono::steady_clock::now();
                if (task_queue.top().scheduled_time > now) {
                    condition.wait_until(lock, task_queue.top().scheduled_time);
                    continue;
                }
                
                queued_task = std::move(const_cast<QueuedTask&>(task_queue.top()));
                task_queue.pop();
                ++active_tasks;
            }
            
            // Execute task
            try {
                queued_task.task();
            } catch (const std::exception& e) {
                std::cerr << "Task exception: " << e.what() << "\n";
            }
            
            --active_tasks;
            condition.notify_all();
        }
    }
};

int main() {
    TaskScheduler scheduler(4);
    
    std::cout << "=== Scheduling Tasks ===\n";
    
    // High priority task
    scheduler.schedule([] {
        std::cout << "[HIGH] Critical task executing\n";
    }, TaskScheduler::Priority::HIGH);
    
    // Normal priority tasks
    for (int i = 0; i < 5; ++i) {
        scheduler.schedule([i] {
            std::cout << "[NORMAL] Task " << i << " executing\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }, TaskScheduler::Priority::NORMAL);
    }
    
    // Low priority task
    scheduler.schedule([] {
        std::cout << "[LOW] Background task executing\n";
    }, TaskScheduler::Priority::LOW);
    
    // Delayed task
    scheduler.schedule([] {
        std::cout << "[DELAYED] Task executing after delay\n";
    }, TaskScheduler::Priority::NORMAL, std::chrono::seconds(2));
    
    std::cout << "=== Waiting for completion ===\n";
    scheduler.wait_all();
    
    std::cout << "=== All tasks completed ===\n";
    
    return 0;
}
```

---

## Conclusion

Part 6 has provided comprehensive coverage of concurrency and multithreading in C++:

1. **Threads and Synchronization** - Creating threads, mutexes, lock guards, deadlock prevention
2. **Condition Variables** - Waiting for events, producer-consumer pattern
3. **Atomic Operations** - Lock-free programming, memory ordering
4. **High-Level Abstractions** - async, future, promise, packaged_task
5. **Thread-Local Storage** - Per-thread data
6. **Parallel Algorithms** - C++17 execution policies
7. **Common Patterns** - Thread pools, reader-writer locks, singletons
8. **Best Practices** - Avoiding data races, deadlocks, false sharing

**Key Principles:**
- **RAII for Locks**: Always use lock_guard, unique_lock, or scoped_lock
- **Prefer High-Level**: Use async, future, parallel algorithms when possible
- **Avoid Deadlocks**: Consistent lock ordering, std::lock, scoped_lock
- **Use Atomics**: For simple counters and flags
- **Small Critical Sections**: Minimize time holding locks
- **Test Thoroughly**: Use sanitizers, stress tests

**Common Mistakes to Avoid:**
```cpp
// ❌ Forgetting to join or detach threads
std::thread t(task);
// t destructor terminates program!

// ❌ Data race on shared variable
int counter = 0;
// Multiple threads incrementing without synchronization

// ❌ Deadlock from inconsistent lock order
lock(m1); lock(m2);  // Thread 1
lock(m2); lock(m1);  // Thread 2

// ❌ Spurious wakeup not handled
cv.wait(lock);  // Should use predicate!

// ❌ Exceptions crossing thread boundaries
// Exceptions in threads don't propagate to main
```

**Concurrency Checklist:**
- [ ] All shared data protected (mutex/atomic)
- [ ] No data races (use thread sanitizer)
- [ ] No deadlocks (consistent ordering)
- [ ] Threads always joined or detached
- [ ] Condition variables use predicates
- [ ] Critical sections minimized
- [ ] Move semantics used for thread arguments
- [ ] Exception handling in threads
- [ ] False sharing avoided (padding/alignment)
- [ ] Thread-safety documented

**Performance Considerations:**
- Mutex overhead: 20-50 ns per lock/unlock
- Atomic operations: 1-10 ns
- Context switch: 1-10 μs
- Cache line: 64 bytes (avoid false sharing)
- Thread creation: 10-100 μs

**Next in the series:** Part 7 will explore Exception Handling & Error Management, covering exception safety guarantees, RAII, and modern error handling techniques.

---

## References and Further Reading

- **"C++ Concurrency in Action"** by Anthony Williams
- **C++ Memory Model**: https://en.cppreference.com/w/cpp/language/memory_model
- **Thread Sanitizer**: https://clang.llvm.org/docs/ThreadSanitizer.html
- **C++ Core Guidelines**: Concurrency section
- **cppreference.com**: Threading and atomic operations

---

*This blog post is part of the Complete C++ Mastery series. Continue to Part 7 for Exception Handling & Error Management.*
