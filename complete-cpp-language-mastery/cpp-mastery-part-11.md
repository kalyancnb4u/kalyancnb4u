---
title: "Complete C++ Mastery Part 11: Performance Optimization"
date: 2024-02-21 06:00:00 +0000
categories: [C++, Programming, Performance]
tags: [cpp, optimization, profiling, performance, cache, simd, benchmarking]
---

# Complete C++ Mastery Part 11: Performance Optimization

## Introduction

Welcome to Part 11 of the Complete C++ Mastery series. Performance is often a primary reason for choosing C++. This part covers the complete spectrum of optimization: profiling tools, algorithmic optimization, compiler optimizations, memory and cache efficiency, data structure selection, SIMD, and benchmarking techniques.

We'll learn to measure first (never optimize without profiling!), identify bottlenecks, apply targeted optimizations, and verify improvements. Understanding performance at the hardware level—CPU cache, branch prediction, memory bandwidth—is crucial for writing truly fast code.

**Prerequisites:** This part assumes knowledge from Parts 1-10, especially memory management, STL, and build systems.

---

## 11.1 Performance Fundamentals

### Measure First, Optimize Later

```cpp
#include <iostream>
#include <chrono>
#include <vector>
#include <algorithm>

// Simple timing utility
class Timer {
    using Clock = std::chrono::high_resolution_clock;
    Clock::time_point start_time;
    const char* name;
    
public:
    Timer(const char* n) : name(n), start_time(Clock::now()) {}
    
    ~Timer() {
        auto end_time = Clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::microseconds>(
            end_time - start_time
        );
        std::cout << name << ": " << duration.count() << " μs\n";
    }
};

void example() {
    Timer t("Vector operation");
    std::vector<int> vec(1000000);
    std::iota(vec.begin(), vec.end(), 0);
    std::sort(vec.begin(), vec.end());
}

int main() {
    example();
    return 0;
}
```

### Understanding Hardware Performance

```cpp
/*
CPU Performance Characteristics (typical modern CPU):

Cache Hierarchy:
- L1 cache: 32-64 KB, ~4 cycles, per core
- L2 cache: 256-512 KB, ~12 cycles, per core
- L3 cache: 8-32 MB, ~40 cycles, shared
- RAM: GB+, ~200 cycles

Cache Line: 64 bytes (16 ints, 8 doubles)

Branch Prediction:
- Correctly predicted: 0 cycles
- Mispredicted: 10-20 cycles

Memory Access Patterns:
- Sequential: ~1 cycle per element (prefetcher)
- Random: ~200 cycles per element (cache miss)

Instruction Level Parallelism (ILP):
- Modern CPUs can execute 4-6 instructions per cycle
- Out-of-order execution
- Superscalar pipelines
*/
```

---

## 11.2 Profiling Tools

### perf (Linux)

```bash
# Collect performance data
perf record -g ./myapp

# View report
perf report

# Detailed statistics
perf stat ./myapp

# Cache statistics
perf stat -e cache-references,cache-misses,instructions,cycles ./myapp

# Branch prediction
perf stat -e branches,branch-misses ./myapp
```

**Example Output:**
```
Performance counter stats for './myapp':

    1,234,567,890  instructions        #  2.34 insn per cycle
      527,891,234  cycles
       12,345,678  cache-references
        1,234,567  cache-misses       # 10.0% of all cache refs
       98,765,432  branches
        4,567,890  branch-misses      # 4.6% of all branches

      0.123456789 seconds time elapsed
```

### gprof

```bash
# Compile with profiling
g++ -pg main.cpp -o myapp

# Run program (generates gmon.out)
./myapp

# View profile
gprof myapp gmon.out > analysis.txt
```

### Valgrind Callgrind

```bash
# Run with callgrind
valgrind --tool=callgrind --dump-instr=yes --collect-jumps=yes ./myapp

# View results
kcachegrind callgrind.out.*

# Or text-based
callgrind_annotate callgrind.out.*
```

### Google Benchmark

```cpp
#include <benchmark/benchmark.h>
#include <vector>
#include <algorithm>

// Benchmark function
static void BM_VectorSort(benchmark::State& state) {
    std::vector<int> vec(state.range(0));
    for (auto _ : state) {
        // Setup
        state.PauseTiming();
        std::iota(vec.begin(), vec.end(), 0);
        std::random_shuffle(vec.begin(), vec.end());
        state.ResumeTiming();
        
        // Actual benchmark
        std::sort(vec.begin(), vec.end());
    }
    state.SetComplexityN(state.range(0));
}

// Register with different sizes
BENCHMARK(BM_VectorSort)->Range(1<<10, 1<<20)->Complexity();

// Custom arguments
static void BM_StringCopy(benchmark::State& state) {
    std::string src(state.range(0), 'x');
    for (auto _ : state) {
        std::string dst = src;
        benchmark::DoNotOptimize(dst);
    }
    state.SetBytesProcessed(state.iterations() * state.range(0));
}
BENCHMARK(BM_StringCopy)->Range(8, 8<<10);

BENCHMARK_MAIN();
```

---

## 11.3 Algorithmic Optimization

### Algorithm Complexity

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <unordered_set>

// ❌ O(n²) - Quadratic
bool has_duplicate_slow(const std::vector<int>& vec) {
    for (size_t i = 0; i < vec.size(); ++i) {
        for (size_t j = i + 1; j < vec.size(); ++j) {
            if (vec[i] == vec[j]) return true;
        }
    }
    return false;
}

// ✅ O(n log n) - Sort first
bool has_duplicate_medium(std::vector<int> vec) {
    std::sort(vec.begin(), vec.end());
    return std::adjacent_find(vec.begin(), vec.end()) != vec.end();
}

// ✅ O(n) - Hash set
bool has_duplicate_fast(const std::vector<int>& vec) {
    std::unordered_set<int> seen;
    for (int val : vec) {
        if (!seen.insert(val).second) return true;
    }
    return false;
}

// Benchmark results for n=10000:
// Slow:   ~50ms   (O(n²))
// Medium: ~0.5ms  (O(n log n))
// Fast:   ~0.2ms  (O(n))
```

### Early Exit Optimization

```cpp
#include <vector>
#include <algorithm>

// ❌ Processes all elements even after finding match
bool contains_slow(const std::vector<int>& vec, int target) {
    bool found = false;
    for (int val : vec) {
        if (val == target) found = true;
    }
    return found;
}

// ✅ Returns immediately on match
bool contains_fast(const std::vector<int>& vec, int target) {
    for (int val : vec) {
        if (val == target) return true;
    }
    return false;
}

// ✅ Or use STL algorithm
bool contains_stl(const std::vector<int>& vec, int target) {
    return std::find(vec.begin(), vec.end(), target) != vec.end();
}
```

### Avoiding Redundant Work

```cpp
#include <string>
#include <vector>

// ❌ Recomputes size every iteration
void process_slow(const std::vector<int>& vec) {
    for (size_t i = 0; i < vec.size(); ++i) {  // size() called n times
        // Process vec[i]
    }
}

// ✅ Cache size
void process_fast(const std::vector<int>& vec) {
    const size_t n = vec.size();  // Called once
    for (size_t i = 0; i < n; ++i) {
        // Process vec[i]
    }
}

// ✅ Or use range-based for loop
void process_best(const std::vector<int>& vec) {
    for (int val : vec) {
        // Process val
    }
}
```

---

## 11.4 Memory Optimization

### Cache-Friendly Data Structures

```cpp
#include <vector>
#include <list>
#include <iostream>
#include <chrono>

// ❌ Cache-unfriendly: list (scattered in memory)
void sum_list(const std::list<int>& data) {
    long long sum = 0;
    for (int val : data) {
        sum += val;
    }
}

// ✅ Cache-friendly: vector (contiguous memory)
void sum_vector(const std::vector<int>& data) {
    long long sum = 0;
    for (int val : data) {
        sum += val;
    }
}

// Benchmark (1M elements):
// List:   ~15ms  (cache misses)
// Vector: ~1ms   (cache hits)
```

### Structure of Arrays (SoA) vs Array of Structures (AoS)

```cpp
#include <vector>

// ❌ Array of Structures (AoS) - poor cache utilization
struct ParticleAoS {
    float x, y, z;      // Position
    float vx, vy, vz;   // Velocity
    float mass;
    int id;
};

void update_aos(std::vector<ParticleAoS>& particles) {
    for (auto& p : particles) {
        p.x += p.vx;  // Loads 32 bytes, uses 8
        p.y += p.vy;  // Poor cache utilization
        p.z += p.vz;
    }
}

// ✅ Structure of Arrays (SoA) - excellent cache utilization
struct ParticlesSoA {
    std::vector<float> x, y, z;
    std::vector<float> vx, vy, vz;
    std::vector<float> mass;
    std::vector<int> id;
};

void update_soa(ParticlesSoA& particles) {
    const size_t n = particles.x.size();
    for (size_t i = 0; i < n; ++i) {
        particles.x[i] += particles.vx[i];  // Sequential access
        particles.y[i] += particles.vy[i];  // Perfect cache utilization
        particles.z[i] += particles.vz[i];
    }
}

// Benchmark (1M particles):
// AoS: ~20ms
// SoA: ~5ms (4x faster!)
```

### Memory Alignment

```cpp
#include <iostream>
#include <cstddef>

// ❌ Misaligned (poor performance)
struct BadAlign {
    char a;      // 1 byte
    double b;    // 8 bytes (needs 8-byte alignment)
    char c;      // 1 byte
};  // sizeof: 24 bytes (padding added)

// ✅ Aligned (better performance)
struct GoodAlign {
    double b;    // 8 bytes
    char a;      // 1 byte
    char c;      // 1 byte
};  // sizeof: 16 bytes

// ✅ Explicit alignment
struct alignas(64) CacheLineAligned {  // 64-byte cache line
    int data[16];
};

int main() {
    std::cout << "BadAlign: " << sizeof(BadAlign) << " bytes\n";
    std::cout << "GoodAlign: " << sizeof(GoodAlign) << " bytes\n";
    std::cout << "CacheLineAligned: " << sizeof(CacheLineAligned) << " bytes\n";
    
    return 0;
}
```

### Avoiding False Sharing

```cpp
#include <thread>
#include <atomic>
#include <vector>

// ❌ False sharing (slow)
struct BadCounters {
    std::atomic<long> counter1;
    std::atomic<long> counter2;
};  // Both on same cache line!

void bad_increment(BadCounters& c) {
    std::vector<std::thread> threads;
    
    threads.emplace_back([&] {
        for (int i = 0; i < 10000000; ++i) {
            c.counter1++;  // Cache line bouncing
        }
    });
    
    threads.emplace_back([&] {
        for (int i = 0; i < 10000000; ++i) {
            c.counter2++;  // Cache line bouncing
        }
    });
    
    for (auto& t : threads) t.join();
}

// ✅ Avoid false sharing (fast)
struct GoodCounters {
    alignas(64) std::atomic<long> counter1;  // Own cache line
    alignas(64) std::atomic<long> counter2;  // Own cache line
};

// Benchmark:
// Bad:  ~500ms (false sharing)
// Good: ~100ms (no false sharing)
```

---

## 11.5 Compiler Optimizations

### Understanding Optimization Levels

```cpp
// Example code
int sum(const int* data, size_t n) {
    int result = 0;
    for (size_t i = 0; i < n; ++i) {
        result += data[i];
    }
    return result;
}

/*
Compilation with different optimization levels:

-O0 (no optimization):
- Straightforward translation
- Easy debugging
- Slowest execution

-O1 (basic optimization):
- Common subexpression elimination
- Constant folding
- Dead code elimination

-O2 (recommended for release):
- All -O1 optimizations
- Loop unrolling
- Function inlining
- Instruction scheduling

-O3 (aggressive):
- All -O2 optimizations
- Vectorization
- More aggressive inlining
- May increase binary size

-Os (optimize for size):
- Similar to -O2 but prioritizes size
- Good for embedded systems

-Ofast (fastest, non-standard):
- All -O3 optimizations
- Ignores strict IEEE floating point
*/
```

### Link-Time Optimization (LTO)

```bash
# Compile with LTO
g++ -O3 -flto file1.cpp file2.cpp -o myapp

# CMake
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
```

```cpp
// LTO enables cross-translation-unit optimizations

// file1.cpp
int compute(int x) {
    return x * x;
}

// file2.cpp
extern int compute(int x);

int main() {
    return compute(5);  // LTO can inline this across files!
}
```

### Profile-Guided Optimization (PGO)

```bash
# Step 1: Compile with instrumentation
g++ -O3 -fprofile-generate main.cpp -o myapp

# Step 2: Run with typical workload
./myapp < typical_input.txt

# Step 3: Recompile with profile data
g++ -O3 -fprofile-use main.cpp -o myapp_optimized
```

### Helping the Compiler

```cpp
// Restrict keyword (pointer aliasing)
void add_arrays_slow(int* a, int* b, int* c, size_t n) {
    for (size_t i = 0; i < n; ++i) {
        c[i] = a[i] + b[i];  // Compiler can't assume no aliasing
    }
}

void add_arrays_fast(int* __restrict a, int* __restrict b, 
                     int* __restrict c, size_t n) {
    for (size_t i = 0; i < n; ++i) {
        c[i] = a[i] + b[i];  // Compiler knows no aliasing
    }
}

// Likely/unlikely (branch prediction hints)
bool process(int value) {
    if (__builtin_expect(value > 0, 1)) {  // Likely
        return true;
    }
    return false;
}

// C++20: [[likely]] and [[unlikely]]
bool process_modern(int value) {
    if (value > 0) [[likely]] {
        return true;
    }
    return false;
}

// Pure functions (no side effects)
__attribute__((const))
int square(int x) {
    return x * x;  // Compiler can optimize calls
}
```

---

## 11.6 Loop Optimization

### Loop Unrolling

```cpp
// ❌ Normal loop
void sum_normal(const int* data, size_t n, int& result) {
    for (size_t i = 0; i < n; ++i) {
        result += data[i];
    }
}

// ✅ Manual unrolling
void sum_unrolled(const int* data, size_t n, int& result) {
    size_t i = 0;
    for (; i + 4 <= n; i += 4) {
        result += data[i];
        result += data[i + 1];
        result += data[i + 2];
        result += data[i + 3];
    }
    // Handle remainder
    for (; i < n; ++i) {
        result += data[i];
    }
}

// Compiler auto-unrolling (with optimization flags)
void sum_auto(const int* data, size_t n, int& result) {
    #pragma GCC unroll 4
    for (size_t i = 0; i < n; ++i) {
        result += data[i];
    }
}
```

### Loop Fusion

```cpp
#include <vector>

// ❌ Two separate loops
void process_separate(std::vector<int>& a, std::vector<int>& b) {
    for (size_t i = 0; i < a.size(); ++i) {
        a[i] *= 2;
    }
    for (size_t i = 0; i < b.size(); ++i) {
        b[i] += 1;
    }
}

// ✅ Fused loop (if arrays are same size)
void process_fused(std::vector<int>& a, std::vector<int>& b) {
    for (size_t i = 0; i < a.size(); ++i) {
        a[i] *= 2;
        b[i] += 1;
    }
}
```

### Loop Invariant Code Motion

```cpp
#include <cmath>

// ❌ Recomputes expensive operation every iteration
void process_slow(double* data, size_t n, double x) {
    for (size_t i = 0; i < n; ++i) {
        data[i] = data[i] * std::sqrt(x * x + 1.0);  // sqrt computed n times
    }
}

// ✅ Move invariant computation outside loop
void process_fast(double* data, size_t n, double x) {
    const double factor = std::sqrt(x * x + 1.0);  // Computed once
    for (size_t i = 0; i < n; ++i) {
        data[i] = data[i] * factor;
    }
}
```

---

## 11.7 SIMD and Vectorization

### Auto-Vectorization

```cpp
#include <vector>

// Compiler can auto-vectorize this
void add_vectors(const float* a, const float* b, float* c, size_t n) {
    #pragma GCC ivdep  // Tell compiler no dependencies
    for (size_t i = 0; i < n; ++i) {
        c[i] = a[i] + b[i];  // Can process 4-8 at once with SIMD
    }
}

// Check if vectorized:
// g++ -O3 -ftree-vectorize -fopt-info-vec main.cpp
```

### Explicit SIMD (SSE/AVX)

```cpp
#include <immintrin.h>  // SSE/AVX intrinsics

// Scalar version
void add_scalar(const float* a, const float* b, float* c, size_t n) {
    for (size_t i = 0; i < n; ++i) {
        c[i] = a[i] + b[i];
    }
}

// SSE version (4 floats at once)
void add_sse(const float* a, const float* b, float* c, size_t n) {
    size_t i = 0;
    for (; i + 4 <= n; i += 4) {
        __m128 va = _mm_loadu_ps(&a[i]);
        __m128 vb = _mm_loadu_ps(&b[i]);
        __m128 vc = _mm_add_ps(va, vb);
        _mm_storeu_ps(&c[i], vc);
    }
    // Handle remainder
    for (; i < n; ++i) {
        c[i] = a[i] + b[i];
    }
}

// AVX version (8 floats at once)
void add_avx(const float* a, const float* b, float* c, size_t n) {
    size_t i = 0;
    for (; i + 8 <= n; i += 8) {
        __m256 va = _mm256_loadu_ps(&a[i]);
        __m256 vb = _mm256_loadu_ps(&b[i]);
        __m256 vc = _mm256_add_ps(va, vb);
        _mm256_storeu_ps(&c[i], vc);
    }
    // Handle remainder
    for (; i < n; ++i) {
        c[i] = a[i] + b[i];
    }
}

// Benchmark (1M elements):
// Scalar: ~2ms
// SSE:    ~0.5ms (4x faster)
// AVX:    ~0.25ms (8x faster)
```

---

## 11.8 Comprehensive Optimization Example

```cpp
#include <vector>
#include <algorithm>
#include <numeric>
#include <random>
#include <chrono>
#include <iostream>

// Image blur example

// ❌ Slow version: Poor cache utilization, repeated computation
void blur_slow(const std::vector<float>& src, std::vector<float>& dst,
               int width, int height) {
    for (int y = 1; y < height - 1; ++y) {
        for (int x = 1; x < width - 1; ++x) {
            float sum = 0;
            for (int dy = -1; dy <= 1; ++dy) {
                for (int dx = -1; dx <= 1; ++dx) {
                    sum += src[(y + dy) * width + (x + dx)];
                }
            }
            dst[y * width + x] = sum / 9.0f;
        }
    }
}

// ✅ Fast version: Better cache utilization, loop unrolling
void blur_fast(const std::vector<float>& src, std::vector<float>& dst,
               int width, int height) {
    // Separable filter: blur horizontally then vertically
    std::vector<float> temp(width * height);
    
    // Horizontal pass
    for (int y = 0; y < height; ++y) {
        for (int x = 1; x < width - 1; ++x) {
            int idx = y * width + x;
            temp[idx] = (src[idx - 1] + src[idx] + src[idx + 1]) / 3.0f;
        }
    }
    
    // Vertical pass
    for (int y = 1; y < height - 1; ++y) {
        for (int x = 0; x < width; ++x) {
            int idx = y * width + x;
            dst[idx] = (temp[idx - width] + temp[idx] + temp[idx + width]) / 3.0f;
        }
    }
}

// Benchmark
void benchmark_blur() {
    const int width = 1920;
    const int height = 1080;
    std::vector<float> src(width * height);
    std::vector<float> dst(width * height);
    
    // Initialize with random data
    std::generate(src.begin(), src.end(), []() { return std::rand() / float(RAND_MAX); });
    
    // Slow version
    auto start = std::chrono::high_resolution_clock::now();
    blur_slow(src, dst, width, height);
    auto end = std::chrono::high_resolution_clock::now();
    auto slow_ms = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();
    
    // Fast version
    start = std::chrono::high_resolution_clock::now();
    blur_fast(src, dst, width, height);
    end = std::chrono::high_resolution_clock::now();
    auto fast_ms = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();
    
    std::cout << "Slow: " << slow_ms << "ms\n";
    std::cout << "Fast: " << fast_ms << "ms\n";
    std::cout << "Speedup: " << (float)slow_ms / fast_ms << "x\n";
}

int main() {
    benchmark_blur();
    return 0;
}

// Typical results:
// Slow: 150ms
// Fast: 20ms
// Speedup: 7.5x
```

---

## Conclusion

Part 11 covered performance optimization comprehensively:

1. **Fundamentals** - Measure first, understand hardware
2. **Profiling** - perf, gprof, Valgrind, Google Benchmark
3. **Algorithmic** - Complexity, early exit, avoid redundant work
4. **Memory** - Cache-friendly structures, SoA vs AoS, alignment
5. **Compiler** - Optimization levels, LTO, PGO
6. **Loops** - Unrolling, fusion, invariant code motion
7. **SIMD** - Auto-vectorization, explicit intrinsics

**Key Principles:**
- **Measure First**: Profile before optimizing
- **Algorithmic First**: O(n) beats optimized O(n²)
- **Cache Matters**: Memory access patterns critical
- **Help the Compiler**: Use const, restrict, likely/unlikely
- **SIMD**: 4-8x speedup for data-parallel code

**Optimization Checklist:**
- [ ] Profile to find hotspots
- [ ] Choose optimal algorithm (O(n) complexity)
- [ ] Use appropriate data structures
- [ ] Make code cache-friendly
- [ ] Enable compiler optimizations (-O3, LTO)
- [ ] Consider SIMD for hot loops
- [ ] Verify improvements with benchmarks

**Next in the series:** Part 12 will cover Modern C++ Best Practices.

---

## References

- **"Optimizing C++"** by Agner Fog
- **Intel Optimization Manual**
- **Google Benchmark**: https://github.com/google/benchmark

---

*This blog post is part of the Complete C++ Mastery series.*
