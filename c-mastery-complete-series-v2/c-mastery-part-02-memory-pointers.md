---
title: "Complete C Language Mastery Part 2: Memory Management & Pointers"
date: 2024-02-21 11:00:00 +0000
categories: [C, Programming, Systems]
tags: [c-language, systems-programming, pointers, memory-management, dynamic-allocation, arrays, strings, low-level]
---

# Complete C Language Mastery Part 2: Memory Management & Pointers

Welcome to Part 2 of the C Language Mastery series. This part covers C's most powerful and dangerous feature: direct memory manipulation through pointers. Understanding memory management is what separates competent C programmers from masters of the language. We'll explore everything from fundamental pointer concepts through advanced memory manipulation techniques.

**Prerequisites**: Part 1 (C Fundamentals & Language Core)

---

## 2.1 Understanding Memory

### Concept Introduction

Memory is the workspace where programs store data during execution. Understanding how C organizes and uses memory is fundamental to writing efficient, correct, and safe programs. Unlike high-level languages with automatic memory management, C gives you direct control over memory, which means you must understand memory layout, allocation strategies, and the hardware implications of your choices.

### Memory Layout: The Five Segments

A C program's memory is organized into distinct regions, each serving a specific purpose:

```
High Memory Address (0xFFFFFFFF on 32-bit)
+---------------------------+
|   Command-line arguments  |
|   and environment vars    |
+---------------------------+
|         Stack             |  ← Grows downward
|           ↓               |    (automatic storage)
|                           |
|      (Free space)         |
|                           |
|           ↑               |
|         Heap              |  ← Grows upward
+---------------------------+    (dynamic allocation)
|    Uninitialized Data     |
|        (BSS segment)      |  ← Zero-initialized
+---------------------------+
|     Initialized Data      |  ← Explicitly initialized
|      (Data segment)       |    global/static variables
+---------------------------+
|      Text/Code            |  ← Machine instructions
|      (Read-only)          |    (program code)
+---------------------------+
Low Memory Address (0x00000000)
```

**1. Text/Code Segment:**
```c
/* Machine code stored here */
int add(int a, int b) {  /* Function code in text segment */
    return a + b;
}

/* String literals typically in text or read-only data */
const char *msg = "Hello";  /* "Hello" in read-only memory */

/* Attempting to modify causes crash */
char *str = "Constant";
// str[0] = 'c';  /* ❌ SEGMENTATION FAULT */
```

**2. Initialized Data Segment:**
```c
/* Explicitly initialized global and static variables */
int global_initialized = 42;              /* Data segment */
static int static_initialized = 100;      /* Data segment */

void function(void) {
    static int func_static = 5;           /* Data segment */
}

const int read_only_data = 123;           /* May be in text or data */
```

**3. BSS (Block Started by Symbol) Segment:**
```c
/* Uninitialized global and static variables */
/* Automatically initialized to zero */
int global_uninitialized;                 /* BSS - will be 0 */
static int static_uninitialized;          /* BSS - will be 0 */

void function(void) {
    static int func_static_uninit;        /* BSS - will be 0 */
}

/* BSS doesn't take space in executable file */
/* Operating system zeros it when program loads */
int large_array[1000000];  /* Only size info in executable */
```

**4. Heap Segment:**
```c
/* Dynamically allocated memory */
#include <stdlib.h>

void function(void) {
    /* malloc allocates from heap */
    int *heap_int = malloc(sizeof(int));  /* Heap allocation */
    *heap_int = 42;
    
    /* Heap memory persists until freed */
    free(heap_int);  /* Return to heap */
    
    /* Large allocations always go to heap */
    int *large = malloc(1000000 * sizeof(int));
    free(large);
}
```

**5. Stack Segment:**
```c
/* Automatic storage for local variables */
void function(int param) {  /* param on stack */
    int local = 10;         /* local on stack */
    char buffer[100];       /* buffer on stack */
    
    /* Stack frame created on function entry */
    /* Stack frame destroyed on function exit */
}  /* All local variables destroyed here */
```

### Visualizing Memory Layout with Code

```c
#include <stdio.h>
#include <stdlib.h>

/* Global variables */
int global_init = 100;      /* Data segment */
int global_uninit;          /* BSS segment */
const int global_const = 200;  /* Text or data segment */

/* Static variables */
static int static_global = 300;  /* Data segment */

void print_addresses(void) {
    /* Local variables */
    int local_var = 10;            /* Stack */
    static int static_local = 20;  /* Data segment */
    
    /* Heap allocation */
    int *heap_var = malloc(sizeof(int));
    *heap_var = 30;
    
    /* String literal */
    const char *literal = "Hello";  /* literal on stack, "Hello" in text */
    
    printf("Memory Layout:\n\n");
    
    printf("Text/Code Segment:\n");
    printf("  Function address:    %p\n", (void*)print_addresses);
    printf("  String literal:      %p (%s)\n", (void*)literal, literal);
    
    printf("\nData Segment:\n");
    printf("  global_init:         %p = %d\n", (void*)&global_init, global_init);
    printf("  global_const:        %p = %d\n", (void*)&global_const, global_const);
    printf("  static_global:       %p = %d\n", (void*)&static_global, static_global);
    printf("  static_local:        %p = %d\n", (void*)&static_local, static_local);
    
    printf("\nBSS Segment:\n");
    printf("  global_uninit:       %p = %d\n", (void*)&global_uninit, global_uninit);
    
    printf("\nHeap Segment:\n");
    printf("  heap_var (pointer):  %p\n", (void*)&heap_var);
    printf("  heap_var (data):     %p = %d\n", (void*)heap_var, *heap_var);
    
    printf("\nStack Segment:\n");
    printf("  local_var:           %p = %d\n", (void*)&local_var, local_var);
    printf("  heap_var pointer:    %p\n", (void*)&heap_var);
    
    /* Check ordering */
    printf("\nAddress Ordering (Low to High):\n");
    printf("  Code < Data < BSS < Heap < ... < Stack\n");
    
    free(heap_var);
}

int main(void) {
    print_addresses();
    return 0;
}

/* Typical Output (addresses will vary):
Memory Layout:

Text/Code Segment:
  Function address:    0x400536
  String literal:      0x400724 (Hello)

Data Segment:
  global_init:         0x601040 = 100
  global_const:        0x400720 = 200
  static_global:       0x601044 = 300
  static_local:        0x601048 = 20

BSS Segment:
  global_uninit:       0x60104c = 0

Heap Segment:
  heap_var (pointer):  0x7ffd8c8f3ab8
  heap_var (data):     0x1a4a010 = 30

Stack Segment:
  local_var:           0x7ffd8c8f3ab4 = 10
  heap_var pointer:    0x7ffd8c8f3ab8
*/
```

### Stack vs Heap Memory

**Stack Memory:**

```c
void stack_example(void) {
    /* Fast allocation (just move stack pointer) */
    int arr[1000];     /* 4000 bytes allocated instantly */
    
    /* Automatic deallocation on return */
    /* No memory leaks possible */
}  /* arr automatically freed */

/* Stack characteristics */
// ✅ Very fast allocation/deallocation
// ✅ Automatic lifetime management
// ✅ No fragmentation
// ❌ Limited size (typically 1-8 MB)
// ❌ Fixed size at compile time
// ❌ Cannot return pointers to stack variables
```

**Heap Memory:**

```c
void heap_example(void) {
    /* Slower allocation (find free block) */
    int *arr = malloc(1000 * sizeof(int));
    if (arr == NULL) {
        /* Handle allocation failure */
        return;
    }
    
    /* Manual deallocation required */
    free(arr);  /* Must explicitly free */
}

/* Heap characteristics */
// ✅ Large size (limited by available RAM)
// ✅ Runtime size determination
// ✅ Can return pointers to caller
// ❌ Slower allocation/deallocation
// ❌ Manual memory management required
// ❌ Fragmentation possible
// ❌ Memory leaks if not freed
```

**Comparison:**

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

void benchmark_stack(void) {
    clock_t start = clock();
    
    for (int i = 0; i < 1000000; i++) {
        int arr[100];  /* Stack allocation */
        arr[0] = i;
        /* Automatic cleanup */
    }
    
    clock_t end = clock();
    double time_spent = (double)(end - start) / CLOCKS_PER_SEC;
    printf("Stack: %f seconds\n", time_spent);
}

void benchmark_heap(void) {
    clock_t start = clock();
    
    for (int i = 0; i < 1000000; i++) {
        int *arr = malloc(100 * sizeof(int));  /* Heap allocation */
        if (arr) {
            arr[0] = i;
            free(arr);  /* Manual cleanup */
        }
    }
    
    clock_t end = clock();
    double time_spent = (double)(end - start) / CLOCKS_PER_SEC;
    printf("Heap: %f seconds\n", time_spent);
}

/* Typical output:
   Stack: 0.002 seconds
   Heap:  0.156 seconds
   (Heap is ~78x slower due to allocation overhead)
*/
```

**Stack Overflow Example:**

```c
/* ❌ DANGER: Stack overflow */
void recursive_overflow(int n) {
    int large_array[10000];  /* 40KB per call */
    large_array[0] = n;
    
    recursive_overflow(n + 1);  /* Eventually crashes */
}

/* Stack size is limited (check with: ulimit -s on Linux) */
/* Typical stack size: 1-8 MB */
/* Deep recursion or large local arrays can overflow */

/* ✅ Use heap for large allocations */
void safe_large_allocation(void) {
    int *large_array = malloc(10000 * sizeof(int));
    if (large_array == NULL) {
        fprintf(stderr, "Allocation failed\n");
        return;
    }
    /* Use array... */
    free(large_array);
}
```

### Memory Alignment and Padding

Modern CPUs access memory most efficiently when data is aligned to natural boundaries:

```c
#include <stdio.h>
#include <stddef.h>

/* Poorly aligned structure */
struct BadlyAligned {
    char c;      /* 1 byte */
                 /* 3 bytes padding */
    int i;       /* 4 bytes */
    char c2;     /* 1 byte */
                 /* 3 bytes padding */
};  /* Total: 12 bytes (not 6!) */

/* Well-aligned structure */
struct WellAligned {
    int i;       /* 4 bytes */
    char c;      /* 1 byte */
    char c2;     /* 1 byte */
                 /* 2 bytes padding */
};  /* Total: 8 bytes */

/* Tightly packed structure */
struct Packed {
    char c;      /* 1 byte */
    char c2;     /* 1 byte */
                 /* 2 bytes padding */
    int i;       /* 4 bytes */
};  /* Total: 8 bytes */

void demonstrate_alignment(void) {
    printf("Size of BadlyAligned:  %zu bytes\n", sizeof(struct BadlyAligned));   /* 12 */
    printf("Size of WellAligned:   %zu bytes\n", sizeof(struct WellAligned));    /* 8 */
    printf("Size of Packed:        %zu bytes\n", sizeof(struct Packed));         /* 8 */
    
    /* Check alignment requirements */
    printf("\nAlignment requirements:\n");
    printf("char:    %zu\n", _Alignof(char));     /* 1 */
    printf("short:   %zu\n", _Alignof(short));    /* 2 */
    printf("int:     %zu\n", _Alignof(int));      /* 4 */
    printf("long:    %zu\n", _Alignof(long));     /* 4 or 8 */
    printf("double:  %zu\n", _Alignof(double));   /* 8 */
    printf("pointer: %zu\n", _Alignof(void*));    /* 4 or 8 */
    
    /* Show padding with offsetof */
    printf("\nBadlyAligned offsets:\n");
    printf("c:  offset %zu\n", offsetof(struct BadlyAligned, c));   /* 0 */
    printf("i:  offset %zu\n", offsetof(struct BadlyAligned, i));   /* 4 */
    printf("c2: offset %zu\n", offsetof(struct BadlyAligned, c2));  /* 8 */
}
```

**Visualizing Padding:**

```
struct BadlyAligned:
+------+---+---+---+------+------+------+------+------+---+---+---+
|  c   |pad|pad|pad|  i (4 bytes)      |  c2  |pad|pad|pad|
+------+---+---+---+------+------+------+------+------+---+---+---+
0      1   2   3   4      5      6      7      8      9   10  11

struct WellAligned:
+------+------+------+------+------+------+---+---+
|  i (4 bytes)              |  c   |  c2  |pad|pad|
+------+------+------+------+------+------+---+---+
0      1      2      3      4      5      6   7

struct Packed:
+------+------+---+---+------+------+------+------+
|  c   |  c2  |pad|pad|  i (4 bytes)      |
+------+------+---+---+------+------+------+------+
0      1      2   3   4      5      6      7
```

**Manual Packing (Compiler-Specific):**

```c
/* GCC/Clang: Pack to 1-byte alignment */
struct __attribute__((packed)) NoPadding {
    char c;
    int i;
    char c2;
};  /* Total: 6 bytes */

/* MSVC */
#pragma pack(push, 1)
struct NoPadding2 {
    char c;
    int i;
    char c2;
};
#pragma pack(pop)

/* ⚠️ Warning: Packed structures may cause:
   - Slower access (unaligned reads)
   - Crashes on some architectures (ARM, SPARC)
   - Use only when necessary (binary formats, network protocols)
*/
```

### Data Structure Layout

Understanding how structures are laid out in memory:

```c
#include <stdio.h>
#include <stddef.h>

struct Point {
    int x;
    int y;
};

struct Rectangle {
    struct Point top_left;
    struct Point bottom_right;
};

void structure_layout(void) {
    struct Rectangle rect;
    
    printf("Rectangle layout:\n");
    printf("  sizeof(Rectangle): %zu\n", sizeof(struct Rectangle));  /* 16 */
    printf("  Offset of top_left:     %zu\n", offsetof(struct Rectangle, top_left));
    printf("  Offset of bottom_right: %zu\n", offsetof(struct Rectangle, bottom_right));
    
    /* Nested structure layout */
    printf("\nMemory addresses:\n");
    printf("  rect:              %p\n", (void*)&rect);
    printf("  rect.top_left:     %p\n", (void*)&rect.top_left);
    printf("  rect.top_left.x:   %p\n", (void*)&rect.top_left.x);
    printf("  rect.top_left.y:   %p\n", (void*)&rect.top_left.y);
    printf("  rect.bottom_right: %p\n", (void*)&rect.bottom_right);
}

/* Array layout */
int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    
    printf("Array layout:\n");
    for (int i = 0; i < 5; i++) {
        printf("  arr[%d] at %p = %d\n", i, (void*)&arr[i], arr[i]);
    }
    
    /* Arrays are contiguous in memory */
    printf("\nElement spacing: %zu bytes\n", 
           (char*)&arr[1] - (char*)&arr[0]);  /* sizeof(int) */
    
    structure_layout();
    return 0;
}
```

### Endianness

Byte order for multi-byte values:

```c
#include <stdio.h>
#include <stdint.h>

void check_endianness(void) {
    uint32_t value = 0x01020304;
    uint8_t *bytes = (uint8_t*)&value;
    
    printf("Value: 0x%08X\n", value);
    printf("Byte layout in memory:\n");
    for (int i = 0; i < 4; i++) {
        printf("  Byte %d: 0x%02X\n", i, bytes[i]);
    }
    
    if (bytes[0] == 0x04) {
        printf("System is LITTLE-ENDIAN\n");
        printf("  (Least significant byte first)\n");
        printf("  Memory: [04] [03] [02] [01]\n");
    } else if (bytes[0] == 0x01) {
        printf("System is BIG-ENDIAN\n");
        printf("  (Most significant byte first)\n");
        printf("  Memory: [01] [02] [03] [04]\n");
    }
}

/* Byte order conversion (network byte order is big-endian) */
#include <arpa/inet.h>  /* POSIX */

uint32_t host_to_network(uint32_t host_long) {
    return htonl(host_long);  /* Host TO Network Long */
}

uint32_t network_to_host(uint32_t net_long) {
    return ntohl(net_long);  /* Network TO Host Long */
}

/* Manual conversion */
uint32_t swap_endian_32(uint32_t value) {
    return ((value & 0xFF000000) >> 24) |
           ((value & 0x00FF0000) >>  8) |
           ((value & 0x0000FF00) <<  8) |
           ((value & 0x000000FF) << 24);
}
```

### Virtual Memory and Address Spaces

Modern operating systems use virtual memory:

```c
/* Each process has its own virtual address space */
/* Physical memory is abstracted away */

void virtual_memory_demo(void) {
    /* Two processes can have same virtual address */
    /* But they map to different physical memory */
    
    static int global = 42;
    printf("Virtual address of global: %p\n", (void*)&global);
    
    /* This address is the same across program runs (ASLR disabled) */
    /* But refers to different physical RAM */
}

/* Address Space Layout Randomization (ASLR) */
/* Security feature that randomizes memory layout */
/* Makes exploits harder */

int main(void) {
    /* Run multiple times - addresses change (ASLR enabled) */
    int *heap = malloc(sizeof(int));
    int stack;
    
    printf("Heap address: %p\n", (void*)heap);
    printf("Stack address: %p\n", (void*)&stack);
    
    free(heap);
    return 0;
}
```

### Memory-Mapped I/O

In embedded systems and device drivers, memory addresses map to hardware:

```c
/* Embedded systems example */
#define GPIO_BASE_ADDRESS ((volatile uint32_t*)0x40020000)
#define GPIO_DATA_REGISTER (GPIO_BASE_ADDRESS + 0)
#define GPIO_DIRECTION_REGISTER (GPIO_BASE_ADDRESS + 1)

void configure_gpio(void) {
    /* Writing to memory address controls hardware */
    *GPIO_DIRECTION_REGISTER = 0xFF;  /* Set all pins as output */
    *GPIO_DATA_REGISTER = 0x55;       /* Turn on alternating LEDs */
    
    /* volatile ensures compiler doesn't optimize away accesses */
}

/* Memory-mapped file I/O (POSIX) */
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>

void mmap_example(const char *filename) {
    int fd = open(filename, O_RDONLY);
    if (fd == -1) return;
    
    off_t file_size = lseek(fd, 0, SEEK_END);
    
    /* Map file into memory */
    char *data = mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (data == MAP_FAILED) {
        close(fd);
        return;
    }
    
    /* Access file content like array */
    printf("First byte: %c\n", data[0]);
    
    /* Unmap and close */
    munmap(data, file_size);
    close(fd);
}
```

### Common Pitfalls

**1. Assuming memory layout:**
```c
/* ❌ Undefined: relying on specific layout */
struct S {
    int a;
    int b;
};

void bad_assumption(struct S *s) {
    /* Assumes a and b are contiguous - usually true but not guaranteed */
    int *arr = &s->a;
    int value = arr[1];  /* Might be s->b, might not be */
}
```

**2. Stack/heap confusion:**
```c
/* ❌ Returning stack address */
int *get_value(void) {
    int local = 42;
    return &local;  /* UNDEFINED BEHAVIOR */
}

/* ✅ Return heap or static */
int *get_value_correct(void) {
    static int value = 42;
    return &value;  /* OK: static lifetime */
}
```

**3. Assuming zero-initialization:**
```c
void function(void) {
    int uninit;  /* ❌ Contains garbage */
    printf("%d\n", uninit);  /* UNDEFINED BEHAVIOR */
    
    static int static_var;  /* ✅ Automatically zero */
    printf("%d\n", static_var);  /* OK: prints 0 */
}
```

### Best Practices

1. **Understand where data lives** (stack, heap, data, BSS, text)
2. **Never return addresses of local variables**
3. **Initialize all variables** before use
4. **Organize struct members** to minimize padding
5. **Use alignment correctly** for performance
6. **Be aware of endianness** in binary formats
7. **Use appropriate allocation** (stack for small/temporary, heap for large/persistent)
8. **Understand your platform's memory model**
9. **Check stack size** before large local allocations
10. **Profile memory usage** in production

### Performance Implications

**Cache Locality:**
```c
/* Cache-friendly: sequential access */
for (int i = 0; i < size; i++) {
    sum += array[i];  /* Fast: uses cache line */
}

/* Cache-unfriendly: strided access */
for (int i = 0; i < size; i += 64) {
    sum += array[i];  /* Slower: cache misses */
}

/* Structure of Arrays (SoA) vs Array of Structures (AoS) */
/* AoS: Less cache-friendly */
struct Point { int x, y, z; };
struct Point points[1000];

for (int i = 0; i < 1000; i++) {
    sum += points[i].x;  /* Loads y, z too (wasted cache) */
}

/* SoA: More cache-friendly */
struct Points {
    int x[1000];
    int y[1000];
    int z[1000];
} points;

for (int i = 0; i < 1000; i++) {
    sum += points.x[i];  /* Only loads x values */
}
```

### SDLC Integration

**Requirements:**
- Define memory constraints (embedded: limited RAM)
- Specify lifetime requirements
- Document allocation strategies

**Design:**
- Plan memory allocation patterns
- Design for cache efficiency
- Consider memory fragmentation

**Implementation:**
- Choose appropriate memory regions
- Implement allocation/deallocation correctly
- Add debug instrumentation

**Testing:**
- Test memory usage under load
- Check for leaks (Valgrind)
- Verify alignment assumptions
- Test on target platform

**Deployment:**
- Profile memory usage
- Monitor for leaks
- Optimize hot paths

### FAQ

**Q1: How do I know how much stack/heap space I have?**

Stack: Typically 1-8 MB, check with:
```bash
ulimit -s  # Linux/macOS (in KB)
```

Heap: Limited by available RAM and OS limits
```c
/* Test heap exhaustion */
size_t total = 0;
while (1) {
    void *p = malloc(1024 * 1024);  /* 1 MB */
    if (p == NULL) {
        printf("Out of heap at %zu MB\n", total);
        break;
    }
    total++;
}
```

**Q2: Why does my structure have unexpected size?**

Due to padding/alignment:
```c
struct S {
    char c;   /* 1 byte */
    /* 3 bytes padding */
    int i;    /* 4 bytes */
};  /* Total: 8 bytes, not 5 */

/* Use offsetof to inspect */
printf("c at: %zu\n", offsetof(struct S, c));
printf("i at: %zu\n", offsetof(struct S, i));
```

**Q3: When should I allocate on stack vs heap?**

Stack when:
- Small data (< few KB)
- Short lifetime (function scope)
- Size known at compile time

Heap when:
- Large data
- Needs to outlive function
- Size determined at runtime

**Q4: What is the BSS segment and why does it exist?**

BSS stores uninitialized global/static variables. Instead of storing zeros in the executable file, the OS just allocates and zeros the memory when loading. This reduces executable size.

**Q5: How does virtual memory help security?**

- **Isolation**: Each process has separate address space
- **ASLR**: Randomizes layout, makes exploits harder
- **Protection**: Pages can be read-only, no-execute
- **Detection**: Access violations caught immediately

### Interview Questions

**Q1: Explain the difference between stack and heap memory. When would you use each?**

**Answer:**

**Stack:**
- Automatic storage duration
- LIFO (Last In, First Out) allocation
- Fast (just move stack pointer)
- Limited size (typically 1-8 MB)
- Automatic cleanup
- Cannot return pointers to stack memory

**Heap:**
- Manual allocation (malloc/free)
- Random access
- Slower (find free block, manage metadata)
- Large size (limited by RAM)
- Manual cleanup required
- Can return pointers to caller

**Use Stack for:**
- Small, temporary data
- Fixed-size data known at compile time
- Data not returned from function

**Use Heap for:**
- Large allocations
- Variable-size data (runtime determined)
- Data that outlives function
- Shared data structures

**Example:**
```c
void example(void) {
    /* Stack: small, temporary */
    int buffer[1024];
    
    /* Heap: large, returned to caller */
    int *large_array = malloc(1000000 * sizeof(int));
    /* ... use array ... */
    free(large_array);
}
```

**Q2: Draw the memory layout of this program and explain what happens:**

```c
int global = 10;
static int file_static = 20;

void function(void) {
    static int func_static = 30;
    int local = 40;
    int *heap = malloc(sizeof(int));
    *heap = 50;
}
```

**Answer:**

```
Memory Layout:

Text Segment:
  - function() code

Data Segment (Initialized):
  - global = 10
  - file_static = 20
  - func_static = 30

Heap Segment:
  - *heap → memory containing 50

Stack Segment (during function() call):
  - local = 40
  - heap (pointer variable)
```

**Execution:**
1. `global` and `file_static` in data segment (program start)
2. Call `function()`: stack frame created
3. `func_static` in data segment (initialized once)
4. `local` created on stack
5. `heap` (pointer) created on stack
6. `malloc()` allocates memory on heap
7. Store 50 at heap location
8. Return: stack frame destroyed (local, heap pointer gone)
9. ❌ Memory leak: heap memory not freed

**Q3: Why does this code crash? How do you fix it?**

```c
char *get_string(void) {
    char str[] = "Hello";
    return str;
}

int main(void) {
    char *s = get_string();
    printf("%s\n", s);  /* Crashes or prints garbage */
    return 0;
}
```

**Answer:**

**Problem:** Returning pointer to stack memory. `str` is destroyed when `get_string()` returns.

**Fixes:**

```c
/* Fix 1: Return string literal */
char *get_string(void) {
    return "Hello";  /* String literal in text segment */
}

/* Fix 2: Use static */
char *get_string(void) {
    static char str[] = "Hello";
    return str;  /* static has program lifetime */
}

/* Fix 3: Allocate on heap */
char *get_string(void) {
    char *str = malloc(6);  /* 5 chars + null */
    if (str == NULL) return NULL;
    strcpy(str, "Hello");
    return str;  /* Caller must free() */
}

/* Fix 4: Use caller-provided buffer */
void get_string(char *buffer, size_t size) {
    snprintf(buffer, size, "Hello");
}
```

**Q4: Explain memory alignment. What is the size of these structures and why?**

```c
struct A {
    char c;
    int i;
    char c2;
};

struct B {
    int i;
    char c;
    char c2;
};
```

**Answer:**

**Alignment:** Data types should be stored at addresses that are multiples of their size for efficient CPU access.

**struct A:**
```
+------+---+---+---+------+------+------+------+------+---+---+---+
|  c   |pad|pad|pad|  i (4 bytes)      |  c2  |pad|pad|pad|
+------+---+---+---+------+------+------+------+------+---+---+---+
Offset: 0   1   2   3   4   5   6   7   8   9   10  11

Size: 12 bytes
- c at offset 0 (1 byte)
- 3 bytes padding (align int to 4-byte boundary)
- i at offset 4 (4 bytes)
- c2 at offset 8 (1 byte)
- 3 bytes padding (struct size must be multiple of largest alignment)
```

**struct B:**
```
+------+------+------+------+------+------+---+---+
|  i (4 bytes)              |  c   |  c2  |pad|pad|
+------+------+------+------+------+------+---+---+
Offset: 0   1   2   3   4   5   6   7

Size: 8 bytes (33% smaller!)
- i at offset 0 (4 bytes)
- c at offset 4 (1 byte)
- c2 at offset 5 (1 byte)
- 2 bytes padding
```

**Best Practice:** Order struct members from largest to smallest to minimize padding.

**Q5: Implement a function to detect system endianness at runtime:**

**Answer:**

```c
#include <stdio.h>
#include <stdint.h>

typedef enum {
    LITTLE_ENDIAN_SYS,
    BIG_ENDIAN_SYS,
    UNKNOWN_ENDIAN
} endian_type;

endian_type detect_endianness(void) {
    uint32_t test = 0x01020304;
    uint8_t *bytes = (uint8_t*)&test;
    
    if (bytes[0] == 0x04) {
        return LITTLE_ENDIAN_SYS;  /* LSB first */
    } else if (bytes[0] == 0x01) {
        return BIG_ENDIAN_SYS;     /* MSB first */
    } else {
        return UNKNOWN_ENDIAN;
    }
}

/* Alternative using union */
endian_type detect_endianness_union(void) {
    union {
        uint32_t i;
        uint8_t c[4];
    } test = { .i = 0x01020304 };
    
    if (test.c[0] == 0x04) return LITTLE_ENDIAN_SYS;
    if (test.c[0] == 0x01) return BIG_ENDIAN_SYS;
    return UNKNOWN_ENDIAN;
}

/* Usage */
int main(void) {
    endian_type endian = detect_endianness();
    
    switch (endian) {
        case LITTLE_ENDIAN_SYS:
            printf("Little-endian (x86, x86-64, most ARM)\n");
            break;
        case BIG_ENDIAN_SYS:
            printf("Big-endian (network byte order, some RISC)\n");
            break;
        default:
            printf("Unknown endianness\n");
            break;
    }
    
    return 0;
}
```

**Explanation:**
- Store multi-byte value (0x01020304)
- Check first byte
- Little-endian: 0x04 (least significant byte first)
- Big-endian: 0x01 (most significant byte first)

### Key Takeaways

- C programs use five memory segments: text, data, BSS, heap, stack
- Stack is fast, automatic, limited; heap is flexible, manual, larger
- Global/static variables go in data or BSS segments
- Memory alignment affects structure size and performance
- Padding is added to align struct members
- Endianness matters for binary I/O and network protocols
- Virtual memory provides isolation and security
- Never return pointers to stack variables
- Understand memory layout for debugging and optimization
- Profile memory usage to avoid leaks and optimize performance

---

## 2.2 Pointers Fundamentals

### Concept Introduction

A pointer is a variable that stores a memory address. Pointers are C's most powerful and dangerous feature, enabling direct memory manipulation, dynamic data structures, and efficient parameter passing. Mastering pointers is essential for systems programming, as they provide the low-level control that makes C suitable for operating systems, embedded systems, and performance-critical applications.

### What is a Pointer?

```c
/* A pointer stores a memory address */
int value = 42;        /* Regular variable: stores 42 */
int *ptr = &value;     /* Pointer: stores address of value */

/* Visualization:
   Memory Address    Content
   0x1000           42        (value)
   0x2000           0x1000    (ptr - points to value)
*/

printf("value = %d\n", value);              /* 42 */
printf("address of value = %p\n", (void*)&value);  /* 0x1000 (example) */
printf("ptr = %p\n", (void*)ptr);           /* 0x1000 (same address) */
printf("*ptr = %d\n", *ptr);                /* 42 (dereference to get value) */
```

### Pointer Declaration and Initialization

```c
/* Declaration syntax */
int *ptr1;              /* Pointer to int */
char *ptr2;             /* Pointer to char */
double *ptr3;           /* Pointer to double */

/* Multiple pointers */
int *p1, *p2, *p3;      /* Three pointers to int */
int *p4, p5, *p6;       /* p4 and p6 are pointers, p5 is int */

/* ⚠️ Common mistake */
int* ptr7, ptr8;        /* Only ptr7 is pointer, ptr8 is int! */
/* Better style: put * with variable name */

/* Initialization */
int x = 10;
int *ptr = &x;          /* Initialize with address */
int *null_ptr = NULL;   /* Initialize to NULL */

/* ❌ Uninitialized pointer - DANGEROUS */
int *bad_ptr;           /* Contains garbage address */
*bad_ptr = 5;           /* UNDEFINED BEHAVIOR - likely crash */

/* ✅ Always initialize */
int *good_ptr = NULL;   /* Safe: explicitly no valid address */
```

### Address-of Operator (&)

```c
/* & returns the address of a variable */
int var = 100;
printf("Value: %d\n", var);           /* 100 */
printf("Address: %p\n", (void*)&var); /* 0x... */

/* Can only use & with lvalues (things with addresses) */
int *ptr1 = &var;       /* ✅ OK */
// int *ptr2 = &42;     /* ❌ Error: can't take address of literal */
// int *ptr3 = &(x+y);  /* ❌ Error: can't take address of expression */

/* Arrays and functions decay to pointers automatically */
int arr[10];
int *ptr_arr = arr;     /* arr decays to &arr[0] */
// int *ptr_arr2 = &arr; /* Type mismatch (int (*)[10] vs int*) */
```

### Dereference Operator (*)

```c
/* * accesses the value at the pointed address */
int value = 50;
int *ptr = &value;

printf("%d\n", *ptr);   /* 50 - dereference to get value */
*ptr = 100;             /* Modify value through pointer */
printf("%d\n", value);  /* 100 - original changed */

/* Dereferencing NULL is undefined behavior */
int *null_ptr = NULL;
// int x = *null_ptr;   /* ❌ SEGMENTATION FAULT */

/* ✅ Always check before dereferencing */
if (ptr != NULL) {
    int x = *ptr;       /* Safe */
}

/* Multiple levels of indirection */
int val = 10;
int *p1 = &val;         /* Pointer to int */
int **p2 = &p1;         /* Pointer to pointer to int */

printf("%d\n", val);    /* 10 */
printf("%d\n", *p1);    /* 10 */
printf("%d\n", **p2);   /* 10 */
```

### NULL Pointers

```c
#include <stddef.h>  /* For NULL definition */

/* NULL is a null pointer constant */
int *ptr = NULL;     /* Points to nothing */

/* Checking for NULL */
if (ptr == NULL) {
    printf("Pointer is null\n");
}

/* Common pattern: initialize to NULL */
int *p = NULL;
/* ... */
if (condition) {
    p = malloc(sizeof(int));
}
/* ... */
if (p != NULL) {
    *p = 42;
    free(p);
    p = NULL;  /* Set to NULL after freeing */
}

/* NULL vs 0 vs nullptr (C++ concept not in C) */
int *p1 = NULL;      /* Preferred in C */
int *p2 = 0;         /* Also works (0 converts to null pointer) */
// int *p3 = nullptr; /* ❌ C++ only, not available in C */
```

### Void Pointers

```c
/* void* can point to any data type */
void *generic_ptr;

int i = 10;
double d = 3.14;

generic_ptr = &i;    /* Points to int */
generic_ptr = &d;    /* Now points to double (no cast needed) */

/* ⚠️ Cannot dereference void* directly */
// int x = *generic_ptr;  /* ❌ Error: void* has no size */

/* ✅ Must cast before dereferencing */
int x = *(int*)generic_ptr;

/* Common use: generic functions */
void swap(void *a, void *b, size_t size) {
    unsigned char temp[size];
    memcpy(temp, a, size);
    memcpy(a, b, size);
    memcpy(b, temp, size);
}

int x = 5, y = 10;
swap(&x, &y, sizeof(int));
printf("%d %d\n", x, y);  /* 10 5 */

/* malloc returns void* */
int *ptr = malloc(sizeof(int));  /* Implicit conversion from void* */
```

### Pointer Arithmetic

```c
/* Pointer arithmetic operates on elements, not bytes */
int arr[] = {10, 20, 30, 40, 50};
int *ptr = arr;  /* Points to arr[0] */

printf("%d\n", *ptr);      /* 10 */
ptr++;                      /* Move to next int */
printf("%d\n", *ptr);      /* 20 */
ptr += 2;                   /* Move 2 ints forward */
printf("%d\n", *ptr);      /* 40 */

/* Pointer arithmetic accounts for size */
int *p1 = arr;
int *p2 = p1 + 1;
printf("Difference in bytes: %ld\n", (char*)p2 - (char*)p1);  /* 4 */

/* Valid operations */
ptr = arr;
ptr + 3;        /* ✅ Add integer to pointer */
ptr - 1;        /* ✅ Subtract integer from pointer */
ptr++;          /* ✅ Increment */
ptr--;          /* ✅ Decrement */

int *p = &arr[4];
ptrdiff_t diff = p - ptr;  /* ✅ Subtract two pointers (result: 4) */

// ptr + ptr;   /* ❌ Can't add two pointers */
// ptr * 2;     /* ❌ Can't multiply pointers */
// ptr / 2;     /* ❌ Can't divide pointers */

/* Accessing array elements via pointer arithmetic */
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;

printf("%d\n", p[0]);    /* 1 - same as *p */
printf("%d\n", p[2]);    /* 3 - same as *(p + 2) */
printf("%d\n", 2[p]);    /* 3 - same as p[2]! (weird but valid) */

/* Traversing array with pointers */
int sum = 0;
for (int *p = arr; p < arr + 5; p++) {
    sum += *p;
}
printf("Sum: %d\n", sum);  /* 15 */
```

### Pointer Comparison

```c
int arr[10];
int *p1 = &arr[3];
int *p2 = &arr[7];

/* Comparing pointers */
if (p1 < p2) {
    printf("p1 points to earlier element\n");
}

/* Valid comparisons */
if (p1 == p2) { }   /* ✅ Equality */
if (p1 != p2) { }   /* ✅ Inequality */
if (p1 < p2) { }    /* ✅ Less than (same array/object) */
if (p1 <= p2) { }   /* ✅ Less than or equal */

/* Comparing to NULL */
if (ptr == NULL) { }  /* ✅ Always valid */
if (ptr) { }          /* ✅ Same as ptr != NULL */
if (!ptr) { }         /* ✅ Same as ptr == NULL */

/* ⚠️ Comparing pointers from different arrays is undefined */
int arr1[10], arr2[10];
int *p1 = &arr1[0];
int *p2 = &arr2[0];
// if (p1 < p2) { }   /* ❌ UNDEFINED BEHAVIOR */

/* ✅ Can compare for equality though */
if (p1 == p2) { }   /* OK: will be false */
```

### Pointer to Pointer

```c
/* Pointer to pointer (double indirection) */
int value = 42;
int *ptr = &value;      /* Pointer to int */
int **pptr = &ptr;      /* Pointer to pointer to int */

printf("%d\n", value);  /* 42 */
printf("%d\n", *ptr);   /* 42 */
printf("%d\n", **pptr); /* 42 */

/* Modifying through double pointer */
**pptr = 100;
printf("%d\n", value);  /* 100 */

/* Use case: modifying pointer in function */
void allocate_memory(int **ptr) {
    *ptr = malloc(sizeof(int));
    if (*ptr != NULL) {
        **ptr = 42;
    }
}

int *p = NULL;
allocate_memory(&p);  /* Pass address of pointer */
if (p != NULL) {
    printf("%d\n", *p);  /* 42 */
    free(p);
}

/* Dynamic 2D array */
int rows = 3, cols = 4;
int **matrix = malloc(rows * sizeof(int*));
for (int i = 0; i < rows; i++) {
    matrix[i] = malloc(cols * sizeof(int));
}

/* Access elements */
matrix[1][2] = 42;

/* Free memory */
for (int i = 0; i < rows; i++) {
    free(matrix[i]);
}
free(matrix);
```

### Array-Pointer Relationship

```c
/* Arrays and pointers are closely related but NOT identical */

int arr[5] = {1, 2, 3, 4, 5};
int *ptr = arr;  /* arr decays to pointer to first element */

/* Equivalences */
arr[i]  ≡  *(arr + i)  ≡  *(ptr + i)  ≡  ptr[i]

/* arr and &arr */
printf("%p\n", (void*)arr);     /* Address of first element */
printf("%p\n", (void*)&arr);    /* Address of array (same value, different type) */
printf("%p\n", (void*)&arr[0]); /* Same as arr */

/* arr and ptr are NOT the same */
sizeof(arr);  /* Size of entire array (5 * sizeof(int) = 20) */
sizeof(ptr);  /* Size of pointer (4 or 8 bytes) */

/* arr is not modifiable */
// arr++;      /* ❌ Error: array is not a variable */
ptr++;         /* ✅ OK: pointer is a variable */

/* Passing array to function */
void func(int arr[]) {  /* Actually: int *arr */
    /* arr has decayed to pointer */
    sizeof(arr);  /* Size of pointer, not array! */
}

/* Multi-dimensional arrays */
int matrix[3][4];
int (*p)[4] = matrix;  /* Pointer to array of 4 ints */

/* Access */
matrix[1][2]  ≡  *(*(matrix + 1) + 2)  ≡  p[1][2]
```

### Pointer Size and Representation

```c
#include <stdint.h>

void pointer_info(void) {
    int x = 10;
    int *ptr = &x;
    
    printf("Pointer size: %zu bytes\n", sizeof(ptr));
    printf("Pointer value (address): %p\n", (void*)ptr);
    printf("Pointed value: %d\n", *ptr);
    
    /* Pointer size is platform-dependent */
    #if UINTPTR_MAX == 0xFFFFFFFF
        printf("32-bit pointers\n");
    #elif UINTPTR_MAX == 0xFFFFFFFFFFFFFFFF
        printf("64-bit pointers\n");
    #endif
    
    /* Convert pointer to integer (for hashing, etc.) */
    uintptr_t addr = (uintptr_t)ptr;
    printf("Address as integer: %ju\n", (uintmax_t)addr);
    
    /* Convert back to pointer */
    int *ptr2 = (int*)addr;
    printf("Reconstructed pointer: %p\n", (void*)ptr2);
}
```

### Common Pitfalls

**1. Uninitialized pointers:**
```c
/* ❌ DANGER */
int *ptr;        /* Contains garbage address */
*ptr = 10;       /* Writes to random memory - crash or corruption */

/* ✅ Initialize */
int *ptr = NULL;
```

**2. Dereferencing NULL:**
```c
/* ❌ CRASH */
int *ptr = NULL;
*ptr = 10;       /* Segmentation fault */

/* ✅ Check first */
if (ptr != NULL) {
    *ptr = 10;
}
```

**3. Dangling pointers:**
```c
/* ❌ Use after free */
int *ptr = malloc(sizeof(int));
free(ptr);
*ptr = 10;       /* Undefined behavior */

/* ✅ Set to NULL after free */
free(ptr);
ptr = NULL;
```

**4. Lost pointer:**
```c
/* ❌ Memory leak */
int *ptr = malloc(100 * sizeof(int));
ptr = NULL;      /* Lost reference - memory leaked */

/* ✅ Free before reassigning */
free(ptr);
ptr = NULL;
```

**5. Pointer arithmetic out of bounds:**
```c
int arr[5];
int *ptr = arr;
ptr += 10;       /* Points beyond array */
*ptr = 42;       /* ❌ Undefined behavior */
```

### Best Practices

1. **Always initialize pointers** (at least to NULL)
2. **Check for NULL** before dereferencing
3. **Set pointers to NULL** after freeing
4. **Use const** when pointer shouldn't modify data
5. **Understand pointer arithmetic** (element-based, not byte-based)
6. **Be careful with pointer lifetime** (don't return stack addresses)
7. **Use sizeof correctly** (sizeof(ptr) vs sizeof(*ptr))
8. **Validate pointer before use** in defensive code
9. **Avoid void*** unless necessary** (loses type safety)
10. **Document pointer ownership** (who allocates, who frees)

### Performance Implications

```c
/* Pointer vs array indexing */
/* Modern compilers optimize both equally */

/* Array indexing */
for (int i = 0; i < n; i++) {
    sum += arr[i];
}

/* Pointer arithmetic (traditional "faster") */
int *end = arr + n;
for (int *p = arr; p < end; p++) {
    sum += *p;
}

/* Both compile to same machine code with optimization */

/* Cache considerations */
/* Sequential pointer access is cache-friendly */
for (int *p = arr; p < arr + n; p++) {
    *p *= 2;  /* Good: sequential access */
}

/* Strided access is cache-unfriendly */
for (int *p = arr; p < arr + n; p += 64) {
    *p *= 2;  /* Bad: many cache misses */
}
```

### SDLC Integration

**Requirements:**
- Define data ownership and lifetime
- Specify pointer usage patterns
- Document null-ability

**Design:**
- Design clear ownership semantics
- Plan pointer validation strategy
- Consider using smart pointer patterns

**Implementation:**
- Initialize all pointers
- Check returns from allocation
- Use const for read-only access

**Testing:**
- Test with NULL pointers
- Test boundary conditions
- Use sanitizers (ASan) to detect errors
- Test with Valgrind for memory issues

**Deployment:**
- Enable runtime checks in debug builds
- Profile pointer-heavy code
- Monitor for crashes

### FAQ

**Q1: What's the difference between `int *p` and `int* p`?**

No difference - just style preference:
```c
int *p;   /* Emphasizes p is a pointer */
int* p;   /* Emphasizes int* is a type */

/* Be careful with multiple declarations */
int* p, q;  /* Only p is pointer! */
int *p, *q; /* Both are pointers */
```

**Q2: Can I use pointers to access hardware registers?**

Yes, common in embedded systems:
```c
#define REGISTER_ADDRESS ((volatile uint32_t*)0x40000000)
*REGISTER_ADDRESS = 0x12345678;  /* Write to hardware */
```

**Q3: Why is `ptr++` different from `ptr += 1`?**

They're the same for pointer arithmetic:
```c
int arr[5];
int *ptr = arr;

ptr++;      /* Move to next element */
ptr += 1;   /* Same effect */

/* Both move by sizeof(*ptr) bytes */
```

**Q4: How do I swap two pointers?**

```c
void swap_pointers(int **a, int **b) {
    int *temp = *a;
    *a = *b;
    *b = temp;
}

int x = 10, y = 20;
int *p1 = &x, *p2 = &y;
swap_pointers(&p1, &p2);
/* Now p1 points to y, p2 points to x */
```

**Q5: What's the difference between `ptr[i]` and `*(ptr + i)`?**

They're identical - `ptr[i]` is syntactic sugar:
```c
int arr[5];
int *ptr = arr;

ptr[2]      ≡  *(ptr + 2)
2[ptr]      ≡  *(2 + ptr)  /* Legal but confusing! */
```

### Interview Questions

**Q1: Explain pointer arithmetic. If `int *ptr = arr` and `ptr++`, how many bytes does ptr move?**

**Answer:**

Pointer arithmetic is **element-based**, not byte-based. When you increment a pointer, it moves by `sizeof(*ptr)` bytes.

```c
int arr[5];
int *ptr = arr;

printf("ptr points to: %p\n", (void*)ptr);
ptr++;  /* Move to next int */
printf("ptr now points to: %p\n", (void*)ptr);
/* Difference: sizeof(int) bytes (typically 4) */

/* For different types */
char *cp = (char*)arr;
cp++;  /* Moves 1 byte (sizeof(char)) */

short *sp = (short*)arr;
sp++;  /* Moves 2 bytes (sizeof(short)) */

long *lp = (long*)arr;
lp++;  /* Moves 4 or 8 bytes (sizeof(long)) */
```

**The formula: `ptr + n` = `(char*)ptr + n * sizeof(*ptr)`**

**Q2: What's wrong with this code?**

```c
int *create_array(void) {
    int arr[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    return arr;
}
```

**Answer:**

**Problem:** Returning pointer to local array. `arr` is on the stack and destroyed when function returns.

**What happens:**
```c
int *ptr = create_array();
printf("%d\n", ptr[0]);  /* Undefined behavior - accessing freed memory */
```

**Fixes:**

```c
/* Fix 1: Allocate on heap */
int *create_array(void) {
    int *arr = malloc(10 * sizeof(int));
    if (arr == NULL) return NULL;
    for (int i = 0; i < 10; i++) {
        arr[i] = i + 1;
    }
    return arr;  /* Caller must free() */
}

/* Fix 2: Use static (but only one copy exists) */
int *create_array(void) {
    static int arr[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    return arr;
}

/* Fix 3: Caller provides buffer */
void fill_array(int *arr, size_t size) {
    for (size_t i = 0; i < size && i < 10; i++) {
        arr[i] = i + 1;
    }
}
```

**Q3: What is the output?**

```c
int arr[] = {10, 20, 30, 40, 50};
int *ptr = &arr[2];
printf("%d\n", ptr[-1]);
printf("%d\n", ptr[2]);
```

**Answer:**

Output:
```
20
50
```

**Explanation:**
- `ptr` points to `arr[2]` (value 30)
- `ptr[-1]` = `*(ptr - 1)` = `arr[1]` = 20
- `ptr[2]` = `*(ptr + 2)` = `arr[4]` = 50

Negative indices are legal if within bounds!

**Q4: Implement a function to reverse an array using pointers:**

**Answer:**

```c
void reverse_array(int *arr, size_t size) {
    int *left = arr;
    int *right = arr + size - 1;
    
    while (left < right) {
        /* Swap */
        int temp = *left;
        *left = *right;
        *right = temp;
        
        /* Move pointers */
        left++;
        right--;
    }
}

/* Test */
int main(void) {
    int arr[] = {1, 2, 3, 4, 5};
    size_t size = sizeof(arr) / sizeof(arr[0]);
    
    reverse_array(arr, size);
    
    for (size_t i = 0; i < size; i++) {
        printf("%d ", arr[i]);  /* 5 4 3 2 1 */
    }
    printf("\n");
    
    return 0;
}
```

**Q5: Explain the difference between these declarations:**

```c
const int *p1;
int const *p2;
int *const p3;
const int *const p4;
```

**Answer:**

```c
/* 1 & 2: Pointer to const int (both equivalent) */
const int *p1;
int const *p2;
/* Can change pointer, can't modify pointed value */
int x = 10, y = 20;
p1 = &x;    /* ✅ OK */
p1 = &y;    /* ✅ OK */
// *p1 = 30; /* ❌ Error: can't modify through p1 */

/* 3: Const pointer to int */
int *const p3 = &x;
/* Can't change pointer, can modify pointed value */
*p3 = 30;   /* ✅ OK */
// p3 = &y; /* ❌ Error: can't change pointer */

/* 4: Const pointer to const int */
const int *const p4 = &x;
/* Can't change pointer, can't modify pointed value */
// *p4 = 30; /* ❌ Error */
// p4 = &y;  /* ❌ Error */
```

**Reading rule:** Read right-to-left:
- `const int *p` → p is pointer to const int
- `int *const p` → p is const pointer to int
- `const int *const p` → p is const pointer to const int

### Key Takeaways

- Pointers store memory addresses
- `&` gets address, `*` dereferences (gets value)
- Always initialize pointers (at least to NULL)
- NULL pointer represents "no address"
- void* can point to any type (generic pointer)
- Pointer arithmetic is element-based, not byte-based
- Pointers can be compared (within same array/object)
- Pointer to pointer enables multi-level indirection
- Arrays decay to pointers when passed to functions
- sizeof(array) ≠ sizeof(pointer)
- Check for NULL before dereferencing
- Set pointers to NULL after freeing
- Understand pointer ownership and lifetime

---

## 2.3 Dynamic Memory Allocation

### Concept Introduction

Dynamic memory allocation allows programs to request memory at runtime from the heap. Unlike stack allocation (automatic storage), heap memory persists until explicitly freed, enabling flexible data structures whose size isn't known at compile time. Understanding dynamic memory is crucial for building complex data structures, managing resources efficiently, and avoiding memory leaks.

### malloc - Memory Allocation

```c
#include <stdlib.h>

/* malloc allocates uninitialized memory */
void *malloc(size_t size);

/* Basic usage */
int *ptr = malloc(sizeof(int));
if (ptr == NULL) {
    fprintf(stderr, "Memory allocation failed\n");
    exit(EXIT_FAILURE);
}
*ptr = 42;
free(ptr);  /* Always free when done */

/* Allocating arrays */
size_t n = 100;
int *arr = malloc(n * sizeof(int));
if (arr == NULL) {
    /* Handle error */
    return -1;
}

/* Use array */
for (size_t i = 0; i < n; i++) {
    arr[i] = i * 2;
}

free(arr);  /* Free when done */

/* Allocating structures */
struct Person {
    char name[50];
    int age;
};

struct Person *p = malloc(sizeof(struct Person));
if (p != NULL) {
    strcpy(p->name, "Alice");
    p->age = 30;
    free(p);
}

/* ✅ Best practice: use sizeof on variable, not type */
int *safe = malloc(sizeof(*safe));  /* Type-safe, doesn't break if type changes */
```

**malloc Characteristics:**
- Returns void* (implicitly converts to any pointer type in C)
- Returns NULL if allocation fails
- Memory is uninitialized (contains garbage)
- Allocated from heap
- Must be freed with free()

### calloc - Contiguous Allocation

```c
/* calloc allocates and zero-initializes memory */
void *calloc(size_t nmemb, size_t size);

/* Allocate array of 100 ints, all initialized to 0 */
int *arr = calloc(100, sizeof(int));
if (arr == NULL) {
    /* Handle error */
    return -1;
}

/* All elements are 0 */
for (int i = 0; i < 100; i++) {
    printf("%d ", arr[i]);  /* Prints: 0 0 0 ... */
}

free(arr);

/* calloc vs malloc + memset */
int *m1 = malloc(100 * sizeof(int));
memset(m1, 0, 100 * sizeof(int));  /* Same as calloc, but slower */

int *m2 = calloc(100, sizeof(int));  /* More efficient */

/* ⚠️ calloc can detect overflow */
size_t huge = SIZE_MAX;
int *overflow = calloc(huge, sizeof(int));  /* Returns NULL */

/* malloc might overflow before check */
int *bad = malloc(huge * sizeof(int));  /* Overflow in multiplication */
```

**calloc vs malloc:**

| Feature | malloc | calloc |
|---------|--------|--------|
| Parameters | Size in bytes | Number and size of elements |
| Initialization | Uninitialized | Zero-initialized |
| Overflow check | No | Yes (in multiplication) |
| Speed | Faster | Slightly slower (zeroing) |
| Use case | Performance-critical | When zero-init needed |

### realloc - Resize Allocation

```c
/* realloc changes size of previously allocated memory */
void *realloc(void *ptr, size_t size);

/* Start with small array */
size_t capacity = 10;
int *arr = malloc(capacity * sizeof(int));
if (arr == NULL) return -1;

/* Fill array */
for (size_t i = 0; i < capacity; i++) {
    arr[i] = i;
}

/* Need more space - resize */
capacity = 20;
int *temp = realloc(arr, capacity * sizeof(int));
if (temp == NULL) {
    /* Realloc failed - original arr still valid */
    free(arr);
    return -1;
}
arr = temp;  /* Update pointer */

/* Old data preserved, can use new space */
for (size_t i = 10; i < capacity; i++) {
    arr[i] = i;
}

free(arr);

/* realloc behaviors */
// realloc(NULL, size)      → Same as malloc(size)
// realloc(ptr, 0)          → Same as free(ptr), returns NULL (C standard)
//                             ⚠️ Some implementations return non-NULL - use free() instead
// realloc(ptr, new_size)   → May move data to new location
```

**realloc Patterns:**

```c
/* ✅ Correct: Check return value */
int *new_ptr = realloc(old_ptr, new_size);
if (new_ptr == NULL) {
    /* old_ptr still valid, free it */
    free(old_ptr);
    return -1;
}
old_ptr = new_ptr;

/* ❌ Wrong: Overwrite original pointer */
old_ptr = realloc(old_ptr, new_size);  /* If fails, old_ptr lost - memory leak */

/* Growing array pattern */
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} DynamicArray;

void array_append(DynamicArray *arr, int value) {
    if (arr->size >= arr->capacity) {
        /* Grow array (double capacity) */
        size_t new_cap = arr->capacity * 2;
        int *new_data = realloc(arr->data, new_cap * sizeof(int));
        if (new_data == NULL) {
            return;  /* Handle error */
        }
        arr->data = new_data;
        arr->capacity = new_cap;
    }
    arr->data[arr->size++] = value;
}
```

### free - Deallocation

```c
/* free returns memory to heap */
void free(void *ptr);

/* Basic usage */
int *ptr = malloc(sizeof(int));
*ptr = 42;
free(ptr);  /* Return memory */

/* After free, pointer becomes invalid (dangling) */
// *ptr = 100;  /* ❌ UNDEFINED BEHAVIOR */

/* ✅ Set to NULL after free */
free(ptr);
ptr = NULL;

/* Freeing NULL is safe (no-op) */
free(NULL);  /* OK, does nothing */

/* ⚠️ Common errors */
// double free
int *p = malloc(sizeof(int));
free(p);
free(p);  /* ❌ UNDEFINED BEHAVIOR - crash or corruption */

// use after free
int *q = malloc(sizeof(int));
*q = 10;
free(q);
printf("%d\n", *q);  /* ❌ UNDEFINED BEHAVIOR */

// freeing stack memory
int x = 10;
free(&x);  /* ❌ UNDEFINED BEHAVIOR */

// freeing part of allocation
int *arr = malloc(10 * sizeof(int));
free(&arr[5]);  /* ❌ UNDEFINED BEHAVIOR - must free original pointer */
```

### Memory Allocation Strategies

**1. Power-of-Two Allocation:**
```c
/* Round up to next power of 2 */
size_t next_power_of_2(size_t n) {
    n--;
    n |= n >> 1;
    n |= n >> 2;
    n |= n >> 4;
    n |= n >> 8;
    n |= n >> 16;
    #if SIZE_MAX > 0xFFFFFFFF
    n |= n >> 32;
    #endif
    return n + 1;
}

/* Use for buffer sizes */
size_t capacity = next_power_of_2(requested_size);
char *buffer = malloc(capacity);
```

**2. Exponential Growth:**
```c
/* Double capacity when full */
if (size >= capacity) {
    capacity = capacity ? capacity * 2 : 1;
    data = realloc(data, capacity * sizeof(*data));
}
```

**3. Fixed Block Sizes:**
```c
/* Reduce fragmentation in allocators */
#define SMALL_BLOCK 64
#define MEDIUM_BLOCK 256
#define LARGE_BLOCK 1024

void *allocate_sized(size_t size) {
    if (size <= SMALL_BLOCK) return malloc(SMALL_BLOCK);
    if (size <= MEDIUM_BLOCK) return malloc(MEDIUM_BLOCK);
    if (size <= LARGE_BLOCK) return malloc(LARGE_BLOCK);
    return malloc(size);
}
```

### Memory Leaks and Detection

**Memory Leak Examples:**

```c
/* ❌ Leak 1: Lost pointer */
void leak1(void) {
    int *ptr = malloc(100 * sizeof(int));
    ptr = NULL;  /* Lost reference - memory leaked */
}

/* ❌ Leak 2: Forgot to free */
void leak2(void) {
    int *ptr = malloc(100 * sizeof(int));
    /* Use ptr... */
    /* Function returns without freeing */
}

/* ❌ Leak 3: Exception/early return */
void leak3(void) {
    int *ptr = malloc(100 * sizeof(int));
    if (error_condition) {
        return;  /* Forgot to free */
    }
    free(ptr);
}

/* ✅ Fix: Always free on all paths */
void no_leak(void) {
    int *ptr = malloc(100 * sizeof(int));
    if (ptr == NULL) return;
    
    if (error_condition) {
        free(ptr);
        return;
    }
    
    /* Normal path */
    free(ptr);
}
```

**Leak Detection Tools:**

```bash
# Valgrind (Linux/macOS)
valgrind --leak-check=full ./program

# AddressSanitizer (GCC/Clang)
gcc -fsanitize=address -g program.c -o program
./program

# LeakSanitizer (part of ASan)
gcc -fsanitize=leak -g program.c -o program

# Visual Studio Memory Leak Detection (Windows)
# Add at start of main():
#ifdef _DEBUG
    _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
#endif
```

### Double-Free Vulnerabilities

```c
/* ❌ DANGER: Double-free */
int *ptr = malloc(sizeof(int));
free(ptr);
free(ptr);  /* CRASH or heap corruption */

/* Why it's dangerous:
   1. First free returns memory to heap
   2. Second free corrupts heap metadata
   3. Can lead to arbitrary code execution (security vulnerability)
*/

/* ✅ Prevention: Set to NULL after free */
free(ptr);
ptr = NULL;
free(ptr);  /* Safe: freeing NULL is no-op */

/* ✅ Wrapper function */
#define SAFE_FREE(p) do { free(p); (p) = NULL; } while(0)

int *ptr = malloc(sizeof(int));
SAFE_FREE(ptr);
SAFE_FREE(ptr);  /* Safe: second call does nothing */
```

### Use-After-Free Bugs

```c
/* ❌ DANGER: Use after free */
int *ptr = malloc(sizeof(int));
*ptr = 42;
free(ptr);
printf("%d\n", *ptr);  /* UNDEFINED BEHAVIOR */
*ptr = 100;            /* UNDEFINED BEHAVIOR - heap corruption */

/* Why it's dangerous:
   1. Memory may be reallocated for other use
   2. Reading gives wrong data
   3. Writing corrupts other data or heap metadata
   4. Security vulnerability (can be exploited)
*/

/* ✅ Prevention strategies */

/* 1. Set to NULL immediately */
free(ptr);
ptr = NULL;

/* 2. Use debug allocator that detects use-after-free */
#ifdef DEBUG
void *debug_malloc(size_t size) {
    void *p = malloc(size);
    /* Track allocation */
    return p;
}

void debug_free(void *p) {
    /* Check if already freed */
    free(p);
    /* Mark as freed */
}
#else
#define debug_malloc malloc
#define debug_free free
#endif

/* 3. Fill freed memory with pattern (debug builds) */
#ifdef DEBUG
void safe_free(void *ptr, size_t size) {
    memset(ptr, 0xDD, size);  /* Fill with 0xDD */
    free(ptr);
}
#endif
```

### Memory Alignment with aligned_alloc (C11)

```c
#include <stdlib.h>

/* aligned_alloc: Allocate with specific alignment */
void *aligned_alloc(size_t alignment, size_t size);

/* Allocate 1024 bytes aligned to 64-byte boundary */
void *ptr = aligned_alloc(64, 1024);
if (ptr == NULL) {
    /* Handle error */
}

/* Requirements:
   - alignment must be power of 2
   - size must be multiple of alignment
*/

/* Use case: SIMD operations */
#include <immintrin.h>  /* SSE/AVX */

/* Allocate aligned for AVX (256-bit = 32 bytes) */
float *data = aligned_alloc(32, 1024 * sizeof(float));
if (data != NULL) {
    /* Can use AVX instructions efficiently */
    __m256 vec = _mm256_load_ps(data);  /* Aligned load */
    free(data);
}

/* Checking alignment */
int is_aligned(void *ptr, size_t alignment) {
    return ((uintptr_t)ptr % alignment) == 0;
}
```

### Custom Allocators

```c
/* Simple memory pool allocator */
typedef struct {
    void *pool;
    size_t block_size;
    size_t num_blocks;
    size_t next_free;
} MemoryPool;

MemoryPool *pool_create(size_t block_size, size_t num_blocks) {
    MemoryPool *pool = malloc(sizeof(MemoryPool));
    if (pool == NULL) return NULL;
    
    pool->pool = malloc(block_size * num_blocks);
    if (pool->pool == NULL) {
        free(pool);
        return NULL;
    }
    
    pool->block_size = block_size;
    pool->num_blocks = num_blocks;
    pool->next_free = 0;
    
    return pool;
}

void *pool_alloc(MemoryPool *pool) {
    if (pool->next_free >= pool->num_blocks) {
        return NULL;  /* Pool exhausted */
    }
    
    void *ptr = (char*)pool->pool + (pool->next_free * pool->block_size);
    pool->next_free++;
    return ptr;
}

void pool_destroy(MemoryPool *pool) {
    free(pool->pool);
    free(pool);
}

/* Usage */
MemoryPool *pool = pool_create(sizeof(int), 1000);
int *numbers[100];

for (int i = 0; i < 100; i++) {
    numbers[i] = pool_alloc(pool);
    *numbers[i] = i;
}

pool_destroy(pool);  /* Free all at once */
```

### Stack vs Heap Trade-offs

**Stack Allocation:**
```c
void stack_example(void) {
    int buffer[1000];  /* Fast, automatic cleanup */
    /* ... use buffer ... */
}  /* Automatically freed */

/* ✅ Pros: Fast, automatic, no leaks */
/* ❌ Cons: Limited size, fixed at compile-time */
```

**Heap Allocation:**
```c
void heap_example(size_t n) {
    int *buffer = malloc(n * sizeof(int));  /* Flexible size */
    if (buffer == NULL) return;
    /* ... use buffer ... */
    free(buffer);  /* Manual cleanup */
}

/* ✅ Pros: Large size, runtime size, persists */
/* ❌ Cons: Slower, manual management, leak risk */
```

**Comparison:**

| Aspect | Stack | Heap |
|--------|-------|------|
| Speed | Very fast | Slower |
| Size | Small (KB-MB) | Large (GB) |
| Lifetime | Function scope | Until freed |
| Management | Automatic | Manual |
| Fragmentation | None | Possible |
| Overhead | None | Metadata |
| Risk | Overflow | Leaks, corruption |

### Common Pitfalls

**1. Not checking return value:**
```c
/* ❌ */
int *ptr = malloc(1000000000000);  /* Likely fails */
*ptr = 42;  /* CRASH - NULL dereference */

/* ✅ */
int *ptr = malloc(size);
if (ptr == NULL) {
    /* Handle error */
    return -1;
}
```

**2. Incorrect size calculation:**
```c
/* ❌ */
int *arr = malloc(n);  /* Allocates n bytes, not n ints */

/* ✅ */
int *arr = malloc(n * sizeof(int));
/* Even better: */
int *arr = malloc(n * sizeof(*arr));
```

**3. Freeing same pointer twice:**
```c
/* ❌ */
free(ptr);
free(ptr);  /* Double-free */

/* ✅ */
free(ptr);
ptr = NULL;
```

**4. Memory leak in loop:**
```c
/* ❌ */
for (int i = 0; i < 1000; i++) {
    int *temp = malloc(1000);
    /* Forgot to free */
}

/* ✅ */
for (int i = 0; i < 1000; i++) {
    int *temp = malloc(1000);
    /* Use temp */
    free(temp);
}
```

**5. Returning stack address instead of heap:**
```c
/* ❌ */
int *get_array(void) {
    int arr[10];
    return arr;  /* Returns stack address */
}

/* ✅ */
int *get_array(void) {
    int *arr = malloc(10 * sizeof(int));
    return arr;  /* Caller must free */
}
```

### Best Practices

1. **Always check malloc/calloc/realloc returns** for NULL
2. **Use sizeof(*ptr)** instead of sizeof(type)
3. **Initialize pointers to NULL** before allocation
4. **Set pointers to NULL after free**
5. **Match every allocation with deallocation**
6. **Free in reverse order of allocation** (for dependencies)
7. **Use tools** (Valgrind, ASan) to detect leaks
8. **Document ownership** (who allocates, who frees)
9. **Consider RAII patterns** for C (cleanup functions)
10. **Profile memory usage** in production

### Performance Implications

**Allocation overhead:**
```c
/* Slow: Many small allocations */
for (int i = 0; i < 10000; i++) {
    int *p = malloc(sizeof(int));
    *p = i;
    free(p);
}

/* Fast: One large allocation */
int *arr = malloc(10000 * sizeof(int));
for (int i = 0; i < 10000; i++) {
    arr[i] = i;
}
free(arr);

/* Allocation overhead: ~100-1000 CPU cycles */
/* Access overhead: ~1-10 CPU cycles */
```

**Memory pooling:**
```c
/* Custom allocator for fixed-size objects */
struct Node {
    int data;
    struct Node *next;
};

/* Pre-allocate pool */
struct Node *pool = malloc(10000 * sizeof(struct Node));
size_t pool_index = 0;

struct Node *allocate_node(void) {
    if (pool_index >= 10000) return NULL;
    return &pool[pool_index++];
}

/* Much faster than repeated malloc/free */
```

### SDLC Integration

**Requirements:**
- Define memory constraints
- Specify allocation patterns
- Document lifetime requirements

**Design:**
- Plan ownership semantics
- Design for memory efficiency
- Consider custom allocators

**Implementation:**
- Check all allocations
- Document who frees what
- Use consistent patterns

**Testing:**
- Test allocation failures
- Check for leaks (Valgrind)
- Test with sanitizers (ASan)
- Stress test with large allocations

**Deployment:**
- Profile memory usage
- Monitor for leaks
- Set resource limits
- Log allocation failures

### FAQ

**Q1: When should I use malloc vs calloc?**

Use malloc when:
- Performance critical
- You'll initialize all values anyway
- Memory content doesn't matter

Use calloc when:
- Need zero-initialization
- Allocating structures with pointers (safe defaults)
- Want overflow protection

**Q2: Why does malloc return void*?**

void* is generic pointer type that converts to any pointer type. This allows:
```c
int *ip = malloc(sizeof(int));  /* No cast needed in C */
char *cp = malloc(100);         /* Works for any type */
```

**Q3: What happens if I don't free memory?**

Memory leak - allocated memory not returned to system:
- Long-running programs exhaust memory
- OS reclaims memory when process exits
- In embedded systems, may never be reclaimed

**Q4: Can I use realloc to shrink memory?**

Yes:
```c
int *arr = malloc(1000 * sizeof(int));
/* Only need 100 now */
int *smaller = realloc(arr, 100 * sizeof(int));
if (smaller != NULL) {
    arr = smaller;  /* Shrunk successfully */
}
```

**Q5: How much overhead does malloc add?**

Typically 8-16 bytes per allocation for metadata:
```c
sizeof(allocation) = requested_size + overhead
```
Many small allocations waste memory. Use pools or allocate in bulk.

### Interview Questions

**Q1: What's the difference between malloc and calloc?**

**Answer:**

```c
/* malloc: Allocates uninitialized memory */
int *m = malloc(10 * sizeof(int));
/* Memory contains garbage values */

/* calloc: Allocates zero-initialized memory */
int *c = calloc(10, sizeof(int));
/* All elements are 0 */

/* Syntax difference */
malloc(total_bytes);
calloc(num_elements, size_per_element);

/* Performance */
/* malloc: Faster (no initialization) */
/* calloc: Slightly slower (zeros memory) */

/* Overflow protection */
size_t huge = SIZE_MAX;
malloc(huge * sizeof(int));   /* May overflow before allocation */
calloc(huge, sizeof(int));    /* Detects overflow, returns NULL */
```

**Q2: What's wrong with this code?**

```c
void process_data(void) {
    int *data = malloc(1000 * sizeof(int));
    /* Process data... */
}
```

**Answer:**

**Problems:**
1. Doesn't check if malloc succeeded
2. Doesn't free memory (memory leak)

**Fixed:**
```c
void process_data(void) {
    int *data = malloc(1000 * sizeof(int));
    if (data == NULL) {
        fprintf(stderr, "Allocation failed\n");
        return;
    }
    
    /* Process data... */
    
    free(data);  /* Free before return */
}
```

**Q3: Implement a dynamic array that grows automatically:**

**Answer:**

```c
typedef struct {
    int *data;
    size_t size;      /* Current number of elements */
    size_t capacity;  /* Total allocated space */
} DynamicArray;

DynamicArray *array_create(void) {
    DynamicArray *arr = malloc(sizeof(DynamicArray));
    if (arr == NULL) return NULL;
    
    arr->capacity = 4;  /* Initial capacity */
    arr->size = 0;
    arr->data = malloc(arr->capacity * sizeof(int));
    
    if (arr->data == NULL) {
        free(arr);
        return NULL;
    }
    
    return arr;
}

int array_append(DynamicArray *arr, int value) {
    if (arr == NULL) return -1;
    
    /* Grow if needed */
    if (arr->size >= arr->capacity) {
        size_t new_capacity = arr->capacity * 2;
        int *new_data = realloc(arr->data, new_capacity * sizeof(int));
        
        if (new_data == NULL) {
            return -1;  /* Allocation failed */
        }
        
        arr->data = new_data;
        arr->capacity = new_capacity;
    }
    
    arr->data[arr->size++] = value;
    return 0;
}

void array_destroy(DynamicArray *arr) {
    if (arr != NULL) {
        free(arr->data);
        free(arr);
    }
}

/* Usage */
int main(void) {
    DynamicArray *arr = array_create();
    if (arr == NULL) return 1;
    
    for (int i = 0; i < 100; i++) {
        array_append(arr, i);
    }
    
    printf("Size: %zu, Capacity: %zu\n", arr->size, arr->capacity);
    
    array_destroy(arr);
    return 0;
}
```

**Q4: What is a double-free? Why is it dangerous?**

**Answer:**

**Double-free:** Calling free() twice on the same pointer.

```c
int *ptr = malloc(sizeof(int));
free(ptr);
free(ptr);  /* ❌ Double-free */
```

**Why it's dangerous:**

1. **Heap corruption:**
   - free() updates heap metadata
   - Second free corrupts this metadata
   - Subsequent allocations may fail or return wrong memory

2. **Security vulnerability:**
   - Can be exploited for arbitrary code execution
   - Attacker can control heap state
   - Common in CVE (security vulnerabilities)

3. **Unpredictable behavior:**
   - May crash immediately
   - May cause corruption later
   - Difficult to debug (effects appear elsewhere)

**Prevention:**
```c
/* Method 1: Set to NULL */
free(ptr);
ptr = NULL;
free(ptr);  /* Safe: no-op */

/* Method 2: Wrapper macro */
#define SAFE_FREE(p) do { \
    free(p); \
    (p) = NULL; \
} while(0)

SAFE_FREE(ptr);
SAFE_FREE(ptr);  /* Safe */

/* Method 3: Use tools */
/* Valgrind, ASan detect double-frees */
```

**Q5: Explain memory fragmentation and how to avoid it:**

**Answer:**

**Memory fragmentation:** Heap has enough total free memory but no contiguous block large enough for allocation.

**Example:**
```
Initial: [Free: 1000 bytes]

After allocations:
[Used:100][Used:100][Used:100][Used:100][Used:100]

After freeing alternating blocks:
[Used:100][Free:100][Used:100][Free:100][Used:100][Free:100]

Total free: 300 bytes
Largest contiguous: 100 bytes
Request for 200 bytes: FAILS (fragmented)
```

**Types:**

1. **External fragmentation:** Free space split into small blocks
2. **Internal fragmentation:** Allocated block larger than needed

**Avoidance strategies:**

```c
/* 1. Memory pools (fixed-size blocks) */
typedef struct Block {
    struct Block *next;
} Block;

typedef struct {
    Block *free_list;
    size_t block_size;
} Pool;

/* 2. Arena allocator (allocate once, free all at once) */
typedef struct Arena {
    char *memory;
    size_t used;
    size_t capacity;
} Arena;

void *arena_alloc(Arena *arena, size_t size) {
    if (arena->used + size > arena->capacity) {
        return NULL;
    }
    void *ptr = arena->memory + arena->used;
    arena->used += size;
    return ptr;
}

void arena_reset(Arena *arena) {
    arena->used = 0;  /* "Free" all at once */
}

/* 3. Buddy allocator (power-of-2 sizes) */
/* Reduces external fragmentation */

/* 4. Compact allocation (allocate similar lifetimes together) */
/* Objects freed together reduce fragmentation */
```

### Key Takeaways

- malloc allocates uninitialized memory from heap
- calloc allocates zero-initialized memory
- realloc resizes previously allocated memory
- free returns memory to heap
- Always check allocation return values for NULL
- Set pointers to NULL after freeing
- Match every allocation with deallocation
- Memory leaks occur when allocated memory isn't freed
- Double-free and use-after-free are serious bugs
- Use tools (Valgrind, ASan) to detect memory errors
- Custom allocators can improve performance
- Stack is faster but limited; heap is flexible but manual
- Document ownership and lifetime clearly

---

## 2.4 Arrays and Strings

### Array Declaration and Initialization

```c
/* Fixed-size array declaration */
int numbers[5];  /* 5 uninitialized ints */
int values[10] = {0};  /* All elements 0 */
int data[5] = {1, 2, 3, 4, 5};  /* Explicit initialization */
int partial[10] = {1, 2, 3};  /* {1, 2, 3, 0, 0, 0, 0, 0, 0, 0} */

/* Size inferred from initializer */
int auto_size[] = {10, 20, 30, 40};  /* Size = 4 */

/* C99: Designated initializers */
int sparse[100] = {[0] = 1, [50] = 5, [99] = 9};  /* Others 0 */
int matrix[3][3] = {
    [0][0] = 1, [1][1] = 5, [2][2] = 9  /* Diagonal */
};

/* Calculating array size */
int arr[10];
size_t size = sizeof(arr) / sizeof(arr[0]);  /* 10 */
```

### Multi-Dimensional Arrays

```c
/* 2D array */
int matrix[3][4];  /* 3 rows, 4 columns */

/* Initialization */
int grid[3][3] = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

/* Flattened initialization */
int grid2[3][3] = {1, 2, 3, 4, 5, 6, 7, 8, 9};

/* Accessing elements */
matrix[1][2] = 42;  /* Row 1, Column 2 */

/* Iterating */
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 4; j++) {
        matrix[i][j] = i * 4 + j;
    }
}

/* Memory layout: row-major order */
/* int arr[2][3] stored as: arr[0][0], arr[0][1], arr[0][2], arr[1][0], arr[1][1], arr[1][2] */

/* Pointer to array */
int (*ptr)[4] = matrix;  /* Points to array of 4 ints */
ptr[0][1] = 10;  /* Same as matrix[0][1] */
```

### Array Decay to Pointers

```c
int arr[10];

/* Array name decays to pointer to first element */
int *ptr = arr;  /* Same as: int *ptr = &arr[0]; */

/* But arr and &arr are different */
printf("%p\n", (void*)arr);   /* Address of first element */
printf("%p\n", (void*)&arr);  /* Address of array (same value, different type) */

/* Pointer arithmetic */
ptr + 1;  /* Points to arr[1] */
arr + 1;  /* Same */

/* sizeof difference */
sizeof(arr);  /* Size of entire array (10 * sizeof(int)) */
sizeof(ptr);  /* Size of pointer (4 or 8 bytes) */

/* Array passed to function decays to pointer */
void func(int arr[]) {  /* Same as: void func(int *arr) */
    /* arr is pointer here, not array */
    sizeof(arr);  /* Size of pointer, NOT array! */
}

/* Must pass size separately */
void process(int *arr, size_t size) {
    for (size_t i = 0; i < size; i++) {
        arr[i] *= 2;
    }
}

int nums[5] = {1, 2, 3, 4, 5};
process(nums, sizeof(nums) / sizeof(nums[0]));
```

### Pointer vs Array Notation

```c
int arr[5] = {10, 20, 30, 40, 50};
int *ptr = arr;

/* Equivalent access methods */
arr[2]  ≡  *(arr + 2)  ≡  ptr[2]  ≡  *(ptr + 2)  ≡  2[arr]  /* Weird but legal! */

/* Difference: arr is not modifiable */
// arr = ptr;  /* ❌ Error: array is not a variable */
ptr = arr;     /* ✅ OK: pointer is a variable */

// arr++;      /* ❌ Error */
ptr++;         /* ✅ OK */
```

### Dynamic Arrays

```c
/* Allocate array at runtime */
size_t n = 100;
int *arr = malloc(n * sizeof(int));
if (arr == NULL) {
    return -1;
}

/* Use like regular array */
for (size_t i = 0; i < n; i++) {
    arr[i] = i * 2;
}

/* Resize if needed */
size_t new_n = 200;
int *new_arr = realloc(arr, new_n * sizeof(int));
if (new_arr != NULL) {
    arr = new_arr;
}

free(arr);

/* Variable-Length Arrays (VLAs) - C99, optional in C11+ */
void func(int n) {
    int vla[n];  /* Size determined at runtime */
    for (int i = 0; i < n; i++) {
        vla[i] = i;
    }
    /* Automatically freed when function returns */
}

/* ⚠️ VLAs on stack can overflow */
/* ⚠️ Not all compilers support VLAs */
```

### String Fundamentals

```c
/* Strings are null-terminated character arrays */
char str1[] = "Hello";  /* {'H', 'e', 'l', 'l', 'o', '\0'} */
char str2[] = {'H', 'e', 'l', 'l', 'o', '\0'};  /* Same */
char str3[10] = "Hello";  /* Remaining bytes are 0 */

/* String literal */
char *str4 = "Hello";  /* Points to read-only memory */
// str4[0] = 'h';  /* ❌ UNDEFINED BEHAVIOR */

/* Modifiable string */
char str5[] = "Hello";  /* Copy in array */
str5[0] = 'h';  /* ✅ OK */

/* String length */
#include <string.h>
size_t len = strlen("Hello");  /* 5 (doesn't count '\0') */

/* Always null-terminate manually created strings */
char manual[6];
manual[0] = 'H';
manual[1] = 'e';
manual[2] = 'l';
manual[3] = 'l';
manual[4] = 'o';
manual[5] = '\0';  /* Required! */
```

### String Library Functions

```c
#include <string.h>

/* Copying */
char dest[100];
strcpy(dest, "Hello");  /* ⚠️ Unsafe: no bounds check */
strncpy(dest, "Hello", sizeof(dest) - 1);  /* Safer */
dest[sizeof(dest) - 1] = '\0';  /* Ensure null termination */

/* Concatenation */
char str[100] = "Hello";
strcat(str, " World");  /* ⚠️ Unsafe */
strncat(str, " World", sizeof(str) - strlen(str) - 1);  /* Safer */

/* Comparison */
if (strcmp(str1, str2) == 0) {
    printf("Strings are equal\n");
}
int cmp = strncmp(str1, str2, 5);  /* Compare first 5 chars */

/* Searching */
char *pos = strchr("Hello", 'l');  /* Find first 'l' */
char *last = strrchr("Hello", 'l');  /* Find last 'l' */
char *sub = strstr("Hello World", "World");  /* Find substring */

/* Length */
size_t len = strlen("Hello");  /* 5 */

/* Safe alternatives (C11 Annex K, if available) */
#ifdef __STDC_LIB_EXT1__
strcpy_s(dest, sizeof(dest), "Hello");
strcat_s(dest, sizeof(dest), " World");
#endif
```

### String Manipulation Pitfalls

```c
/* ❌ Buffer overflow */
char small[5];
strcpy(small, "This is too long");  /* BUFFER OVERFLOW */

/* ❌ No null terminator */
char str[5];
strncpy(str, "Hello", 5);  /* May not null-terminate if source is >= size */
/* str might not be null-terminated! */

/* ✅ Safe pattern */
char str[10];
strncpy(str, source, sizeof(str) - 1);
str[sizeof(str) - 1] = '\0';

/* ❌ Modifying string literal */
char *str = "Hello";
str[0] = 'h';  /* UNDEFINED BEHAVIOR */

/* ✅ Use array for modifiable strings */
char str[] = "Hello";
str[0] = 'h';  /* OK */
```

### Buffer Overflows

```c
/* Classic buffer overflow vulnerability */
void vulnerable(void) {
    char buffer[10];
    printf("Enter name: ");
    gets(buffer);  /* ❌ NEVER USE gets() - no bounds check */
}

/* ✅ Safe alternatives */
void safe(void) {
    char buffer[10];
    
    /* fgets: reads at most n-1 characters */
    if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
        /* Remove newline if present */
        size_t len = strlen(buffer);
        if (len > 0 && buffer[len-1] == '\n') {
            buffer[len-1] = '\0';
        }
    }
    
    /* scanf with width specifier */
    scanf("%9s", buffer);  /* Reads at most 9 chars + '\0' */
}

/* Preventing overflows in string operations */
void safe_concat(char *dest, size_t dest_size, const char *src) {
    size_t dest_len = strlen(dest);
    size_t src_len = strlen(src);
    
    if (dest_len + src_len + 1 <= dest_size) {
        strcat(dest, src);  /* Safe */
    } else {
        /* Handle error */
    }
}
```

### Bounds Checking

```c
/* Manual bounds checking */
void safe_copy(char *dest, size_t dest_size, const char *src) {
    size_t src_len = strlen(src);
    
    if (src_len >= dest_size) {
        /* Truncate */
        strncpy(dest, src, dest_size - 1);
        dest[dest_size - 1] = '\0';
    } else {
        strcpy(dest, src);
    }
}

/* Range checking for array access */
int safe_get(int *arr, size_t size, size_t index) {
    if (index >= size) {
        fprintf(stderr, "Index %zu out of bounds (size %zu)\n", index, size);
        return 0;  /* Or error value */
    }
    return arr[index];
}

void safe_set(int *arr, size_t size, size_t index, int value) {
    if (index < size) {
        arr[index] = value;
    } else {
        fprintf(stderr, "Index out of bounds\n");
    }
}
```

### Interview Question - Arrays and Strings

**Q: Implement a function to reverse a string in place:**

```c
void reverse_string(char *str) {
    if (str == NULL) return;
    
    size_t len = strlen(str);
    char *left = str;
    char *right = str + len - 1;
    
    while (left < right) {
        /* Swap */
        char temp = *left;
        *left = *right;
        *right = temp;
        
        left++;
        right--;
    }
}

/* Test */
char str[] = "Hello";
reverse_string(str);
printf("%s\n", str);  /* "olleH" */
```

---

## 2.5 Advanced Pointer Concepts

### Pointer to Functions

```c
/* Function pointer declaration */
int (*func_ptr)(int, int);  /* Pointer to function taking two ints, returning int */

/* Assigning function to pointer */
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }

func_ptr = add;  /* Function name decays to pointer */
func_ptr = &add;  /* Explicit (equivalent) */

/* Calling through function pointer */
int result = func_ptr(5, 3);  /* Direct call */
int result2 = (*func_ptr)(5, 3);  /* Explicit dereference (equivalent) */
```

### Arrays of Function Pointers

```c
/* Calculator using function pointer array */
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }
int divide(int a, int b) { return b != 0 ? a / b : 0; }

/* Array of function pointers */
int (*operations[4])(int, int) = {add, sub, mul, divide};

/* Menu-driven calculator */
printf("0=Add, 1=Sub, 2=Mul, 3=Div\n");
int choice = 0;  /* User input */
int result = operations[choice](10, 5);

/* Typedef for readability */
typedef int (*binary_op_t)(int, int);
binary_op_t ops[] = {add, sub, mul, divide};
```

### Callback Mechanisms

```c
/* Generic sort with comparison callback */
void bubble_sort(int *arr, size_t size, int (*compare)(int, int)) {
    for (size_t i = 0; i < size - 1; i++) {
        for (size_t j = 0; j < size - i - 1; j++) {
            if (compare(arr[j], arr[j+1]) > 0) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

/* Comparison functions */
int ascending(int a, int b) { return a - b; }
int descending(int a, int b) { return b - a; }

/* Usage */
int arr[] = {5, 2, 8, 1, 9};
bubble_sort(arr, 5, ascending);  /* Sort ascending */
bubble_sort(arr, 5, descending);  /* Sort descending */

/* Event handling pattern */
typedef void (*event_handler_t)(void *data);

void on_click(void *data) {
    printf("Clicked: %s\n", (char*)data);
}

void register_callback(event_handler_t handler, void *data) {
    /* Store handler for later use */
    handler(data);
}
```

### Const Pointers and Pointers to Const

```c
/* Four combinations */

/* 1. Pointer to const data */
const int *p1;
int const *p1_alt;  /* Same */
int value = 10;
p1 = &value;   /* ✅ Can change pointer */
// *p1 = 20;   /* ❌ Can't modify data */

/* 2. Const pointer to data */
int *const p2 = &value;
*p2 = 20;      /* ✅ Can modify data */
// p2 = &other;/* ❌ Can't change pointer */

/* 3. Const pointer to const data */
const int *const p3 = &value;
// *p3 = 20;   /* ❌ Can't modify data */
// p3 = &other;/* ❌ Can't change pointer */

/* 4. Pointer to data (both mutable) */
int *p4 = &value;
*p4 = 20;      /* ✅ Can modify data */
p4 = &other;   /* ✅ Can change pointer */

/* Practical use: const parameters */
void print_array(const int *arr, size_t size) {
    for (size_t i = 0; i < size; i++) {
        printf("%d ", arr[i]);  /* Read only */
        // arr[i] = 0;  /* ❌ Compile error */
    }
}
```

### Volatile Pointers

```c
/* volatile: Tells compiler value may change unexpectedly */
volatile int *hardware_register = (volatile int*)0x40000000;

/* Prevents optimization */
int x = *hardware_register;  /* Always reads from memory */
int y = *hardware_register;  /* Reads again (not cached) */

/* Without volatile, compiler might optimize to: */
/* int x = *hardware_register; */
/* int y = x;  // Reuse cached value */

/* Use cases */
/* 1. Memory-mapped I/O */
volatile uint32_t *status_reg = (volatile uint32_t*)0x40000000;
while (*status_reg & 0x01) {  /* Wait for bit to clear */
    /* Must read each time */
}

/* 2. Variables modified by ISR */
volatile int flag = 0;

void interrupt_handler(void) {
    flag = 1;  /* Modified by interrupt */
}

void main_loop(void) {
    while (flag == 0) {  /* Check repeatedly */
        /* Compiler won't optimize away check */
    }
}

/* 3. Multi-threaded shared data (but use atomics instead in modern code) */
```

### Restrict Pointers (C99)

```c
/* restrict: Promises pointer is only way to access object */
void copy_array(int *restrict dest, const int *restrict src, size_t n) {
    for (size_t i = 0; i < n; i++) {
        dest[i] = src[i];  /* Compiler knows dest and src don't overlap */
    }
}

/* Enables optimization */
/* Without restrict, compiler must reload src[i] each iteration */
/* With restrict, compiler can optimize more aggressively */

/* Standard library example */
void *memcpy(void *restrict dest, const void *restrict src, size_t n);
/* Promises dest and src don't overlap (undefined if they do) */

void *memmove(void *dest, const void *src, size_t n);
/* Handles overlapping regions (no restrict) */
```

### Pointer Aliasing and Strict Aliasing Rule

```c
/* Strict aliasing rule: Access object through compatible pointer type only */

/* ❌ Violates strict aliasing */
float f = 3.14f;
int i = *(int*)&f;  /* UNDEFINED BEHAVIOR */

/* Compiler assumes float* and int* don't alias */
/* May reorder operations incorrectly */

/* ✅ Exceptions to strict aliasing */
/* 1. char* can alias anything */
float f = 3.14f;
unsigned char *bytes = (unsigned char*)&f;
for (size_t i = 0; i < sizeof(f); i++) {
    printf("%02x ", bytes[i]);  /* OK: byte access */
}

/* 2. Union type-punning (implementation-defined but common) */
union FloatInt {
    float f;
    uint32_t i;
};

union FloatInt fi;
fi.f = 3.14f;
uint32_t bits = fi.i;  /* Access bit pattern */

/* 3. memcpy (always safe) */
float f = 3.14f;
uint32_t i;
memcpy(&i, &f, sizeof(f));  /* Safe */
```

### Common Pointer Errors and Debugging

```c
/* 1. Null pointer dereference */
int *ptr = NULL;
// *ptr = 10;  /* ❌ CRASH */

/* Fix: Always check */
if (ptr != NULL) {
    *ptr = 10;
}

/* 2. Dangling pointer */
int *ptr = malloc(sizeof(int));
free(ptr);
// *ptr = 10;  /* ❌ Use after free */

/* Fix: Set to NULL */
free(ptr);
ptr = NULL;

/* 3. Uninitialized pointer */
int *ptr;  /* Contains garbage */
// *ptr = 10;  /* ❌ UNDEFINED BEHAVIOR */

/* Fix: Initialize */
int *ptr = NULL;

/* 4. Lost pointer (memory leak) */
int *ptr = malloc(100);
ptr = malloc(200);  /* ❌ Lost first allocation */

/* Fix: Free before reassigning */
free(ptr);
ptr = malloc(200);

/* 5. Returning stack address */
int *func(void) {
    int local = 42;
    return &local;  /* ❌ Returns stack address */
}

/* Fix: Use static, heap, or caller-provided buffer */
```

**Debugging Tools:**

```bash
# Valgrind (memory errors, leaks)
valgrind --leak-check=full --show-leak-kinds=all ./program

# AddressSanitizer (fast runtime checks)
gcc -fsanitize=address -g program.c
./program

# UndefinedBehaviorSanitizer
gcc -fsanitize=undefined -g program.c

# MemorySanitizer (uninitialized memory)
clang -fsanitize=memory -g program.c

# GDB (debugger)
gdb ./program
(gdb) break main
(gdb) run
(gdb) print ptr
(gdb) print *ptr
```

### Key Takeaways - Part 2

**Memory Management:**
- C programs use five memory segments: text, data, BSS, heap, stack
- Stack is fast, automatic, limited; heap is flexible, manual, larger
- Always check malloc/calloc/realloc returns
- Match every malloc with free
- Set pointers to NULL after freeing

**Pointers:**
- Pointers store memory addresses
- Always initialize pointers
- Check for NULL before dereferencing
- Understand pointer arithmetic (element-based)
- Arrays decay to pointers when passed to functions

**Dynamic Allocation:**
- malloc: uninitialized memory
- calloc: zero-initialized memory
- realloc: resize allocation
- free: return memory to heap
- Use tools to detect leaks and errors

**Arrays and Strings:**
- Arrays are contiguous memory blocks
- Strings are null-terminated char arrays
- Always bounds-check array access
- Use safe string functions (strncpy, strncat)
- Beware buffer overflows

**Advanced Concepts:**
- Function pointers enable callbacks
- const protects data from modification
- volatile prevents optimization
- restrict enables optimization
- Understand strict aliasing rule
- Use debugging tools (Valgrind, ASan)

---

## Part 2 Complete Summary

**Part 2: Memory Management & Pointers** has provided comprehensive coverage of:

### 2.1 Understanding Memory
- Memory segment layout and purpose
- Stack vs heap allocation strategies
- Memory alignment and padding optimization
- Endianness detection and handling
- Virtual memory concepts
- 5 FAQ + 5 interview questions

### 2.2 Pointers Fundamentals
- Pointer declaration, initialization, operations
- NULL pointers and void pointers
- Pointer arithmetic and comparison
- Multi-level indirection
- Array-pointer relationship
- 5 FAQ + 5 interview questions

### 2.3 Dynamic Memory Allocation
- malloc, calloc, realloc, free functions
- Memory leak prevention
- Double-free and use-after-free bugs
- Custom allocators and memory pools
- Performance optimization
- 5 FAQ + 5 interview questions

### 2.4 Arrays and Strings
- Array declaration and initialization
- Multi-dimensional arrays
- Dynamic arrays and VLAs
- String manipulation and safety
- Buffer overflow prevention
- Bounds checking techniques

### 2.5 Advanced Pointer Concepts
- Function pointers and callbacks
- Const and volatile qualifiers
- Restrict keyword optimization
- Strict aliasing rule
- Common errors and debugging
- Tools (Valgrind, ASan, GDB)

**Total Coverage:** Deep dive into C's memory model with practical examples, security considerations, performance implications, and production-ready patterns throughout.

**Next:** Part 3 will cover Derived Types & Data Structures (structures, unions, enumerations, typedef, and implementing common data structures in C).

