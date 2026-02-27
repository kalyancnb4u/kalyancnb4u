---
title: "Complete C Language Mastery Part 3: Derived Types & Data Structures"
date: 2024-02-21 12:00:00 +0000
categories: [C, Programming, Systems]
tags: [c-language, structures, unions, enums, typedef, data-structures, linked-lists, trees, hash-tables]
---

# Complete C Language Mastery Part 3: Derived Types & Data Structures

Welcome to Part 3 of the C Language Mastery series. This part explores C's derived types (structures, unions, enumerations) and demonstrates how to build complex data structures from these primitives. Understanding these concepts is essential for organizing data efficiently and building sophisticated applications.

**Prerequisites**: Part 1 (Fundamentals), Part 2 (Memory & Pointers)

---

## 3.1 Structures (struct)

### Concept Introduction

Structures (structs) are user-defined types that group related variables under a single name. They're fundamental to organizing complex data and are the foundation for object-oriented-like programming in C. Structures enable you to create composite data types that model real-world entities.

### Structure Definition and Declaration

```c
/* Structure definition */
struct Point {
    int x;
    int y;
};

/* Structure variable declaration */
struct Point p1;           /* Declares variable p1 */
struct Point p2, p3;       /* Multiple variables */

/* Definition and declaration combined */
struct Rectangle {
    int width;
    int height;
} rect1, rect2;            /* Declares rect1, rect2 immediately */

/* Anonymous structure (rarely used) */
struct {
    int day;
    int month;
    int year;
} date1, date2;            /* Can't declare more later */

/* Forward declaration (for self-referential structures) */
struct Node;               /* Declaration only */
struct Node {              /* Definition */
    int data;
    struct Node *next;
};
```

### Structure Initialization

```c
struct Point {
    int x;
    int y;
};

/* C89 style - positional initialization */
struct Point p1 = {10, 20};  /* x=10, y=20 */

/* C99+ designated initializers */
struct Point p2 = {.x = 10, .y = 20};      /* Explicit members */
struct Point p3 = {.y = 20, .x = 10};      /* Order doesn't matter */
struct Point p4 = {.x = 10};               /* y=0 (zero-initialized) */

/* Nested structures */
struct Rectangle {
    struct Point top_left;
    struct Point bottom_right;
};

/* C89 style */
struct Rectangle r1 = {{0, 0}, {100, 100}};

/* C99 style */
struct Rectangle r2 = {
    .top_left = {.x = 0, .y = 0},
    .bottom_right = {.x = 100, .y = 100}
};

/* Partial initialization */
struct Point p5 = {0};     /* All members set to 0 */
struct Point p6 = {5};     /* x=5, y=0 */

/* Array of structures */
struct Point points[3] = {
    {0, 0},
    {10, 10},
    {20, 20}
};

/* C99 array with designated initializers */
struct Point sparse[100] = {
    [0] = {.x = 1, .y = 1},
    [50] = {.x = 5, .y = 5},
    [99] = {.x = 9, .y = 9}
};  /* Others zero-initialized */
```

### Member Access

```c
struct Point {
    int x;
    int y;
};

struct Point p = {10, 20};

/* Dot operator (.) for direct access */
p.x = 30;
p.y = 40;
printf("Point: (%d, %d)\n", p.x, p.y);

/* Arrow operator (->) for pointer access */
struct Point *ptr = &p;
ptr->x = 50;              /* Same as: (*ptr).x = 50; */
ptr->y = 60;              /* Same as: (*ptr).y = 60; */

printf("Point: (%d, %d)\n", ptr->x, ptr->y);

/* Nested member access */
struct Rectangle {
    struct Point top_left;
    struct Point bottom_right;
};

struct Rectangle rect;
rect.top_left.x = 0;      /* Nested dot */
rect.top_left.y = 0;

struct Rectangle *rect_ptr = &rect;
rect_ptr->top_left.x = 10;        /* Arrow then dot */
rect_ptr->bottom_right.y = 100;
```

### Nested Structures

```c
/* Example: Employee with embedded address */
struct Address {
    char street[100];
    char city[50];
    char state[3];
    int zip;
};

struct Employee {
    char name[100];
    int id;
    struct Address address;  /* Nested structure */
    double salary;
};

/* Initialization */
struct Employee emp = {
    .name = "John Doe",
    .id = 12345,
    .address = {
        .street = "123 Main St",
        .city = "Springfield",
        .state = "IL",
        .zip = 62701
    },
    .salary = 75000.0
};

/* Access */
printf("Employee: %s\n", emp.name);
printf("City: %s\n", emp.address.city);
printf("ZIP: %d\n", emp.address.zip);

/* Pointer access */
struct Employee *emp_ptr = &emp;
printf("State: %s\n", emp_ptr->address.state);
```

### Padding and Alignment

```c
#include <stddef.h>  /* For offsetof */

/* Poorly aligned structure */
struct Bad {
    char c;      /* 1 byte */
                 /* 3 bytes padding */
    int i;       /* 4 bytes */
    char c2;     /* 1 byte */
                 /* 3 bytes padding */
};  /* Total: 12 bytes */

/* Well aligned structure */
struct Good {
    int i;       /* 4 bytes */
    char c;      /* 1 byte */
    char c2;     /* 1 byte */
                 /* 2 bytes padding */
};  /* Total: 8 bytes (33% smaller!) */

/* Inspect alignment */
void show_alignment(void) {
    printf("sizeof(struct Bad):  %zu\n", sizeof(struct Bad));   /* 12 */
    printf("sizeof(struct Good): %zu\n", sizeof(struct Good));  /* 8 */
    
    printf("\nBad layout:\n");
    printf("  c  offset: %zu\n", offsetof(struct Bad, c));   /* 0 */
    printf("  i  offset: %zu\n", offsetof(struct Bad, i));   /* 4 */
    printf("  c2 offset: %zu\n", offsetof(struct Bad, c2));  /* 8 */
    
    printf("\nGood layout:\n");
    printf("  i  offset: %zu\n", offsetof(struct Good, i));  /* 0 */
    printf("  c  offset: %zu\n", offsetof(struct Good, c));  /* 4 */
    printf("  c2 offset: %zu\n", offsetof(struct Good, c2)); /* 5 */
}

/* Visualizing padding */
/*
struct Bad memory layout:
+---+---+---+---+---+---+---+---+---+---+---+---+
| c |pad|pad|pad|   i (4 bytes) | c2|pad|pad|pad|
+---+---+---+---+---+---+---+---+---+---+---+---+
0   1   2   3   4   5   6   7   8   9   10  11

struct Good memory layout:
+---+---+---+---+---+---+---+---+
|   i (4 bytes) | c | c2|pad|pad|
+---+---+---+---+---+---+---+---+
0   1   2   3   4   5   6   7
*/

/* Alignment rules */
/* - char: 1-byte aligned */
/* - short: 2-byte aligned */
/* - int/float: 4-byte aligned */
/* - double/long long: 8-byte aligned */
/* - pointers: 4 or 8-byte aligned (platform-dependent) */
/* - struct: aligned to largest member */
```

### Bit Fields

```c
/* Bit fields: Specify exact bit width for members */
struct Flags {
    unsigned int is_visible : 1;    /* 1 bit */
    unsigned int is_enabled : 1;    /* 1 bit */
    unsigned int color      : 3;    /* 3 bits (0-7) */
    unsigned int priority   : 4;    /* 4 bits (0-15) */
                                     /* 23 bits padding on 32-bit system */
};  /* Total: 4 bytes (not 4 separate ints!) */

/* Usage */
struct Flags f = {0};
f.is_visible = 1;
f.is_enabled = 0;
f.color = 5;      /* 0-7 */
f.priority = 10;  /* 0-15 */

/* Packed bit fields for hardware registers */
struct StatusRegister {
    unsigned int ready      : 1;    /* bit 0 */
    unsigned int error      : 1;    /* bit 1 */
    unsigned int reserved   : 2;    /* bits 2-3 */
    unsigned int mode       : 2;    /* bits 4-5 */
    unsigned int interrupt  : 1;    /* bit 6 */
    unsigned int busy       : 1;    /* bit 7 */
};

/* ⚠️ Bit field limitations */
/* - Can't take address of bit field */
/* - Layout is implementation-defined */
/* - Signedness of plain 'int' bit fields is implementation-defined */
/* - Use unsigned int for portability */

/* Example: Cannot do this */
struct Flags f;
// unsigned int *ptr = &f.color;  /* ❌ Error: can't take address */

/* Instead, use bitwise operations for more control */
#define FLAG_VISIBLE (1 << 0)
#define FLAG_ENABLED (1 << 1)
#define COLOR_MASK   (7 << 2)
#define COLOR_SHIFT  2

unsigned int flags = 0;
flags |= FLAG_VISIBLE;                      /* Set visible */
flags &= ~FLAG_ENABLED;                     /* Clear enabled */
flags = (flags & ~COLOR_MASK) | (5 << COLOR_SHIFT);  /* Set color to 5 */
```

### Anonymous Structures and Unions (C11)

```c
/* C11: Anonymous structures inside structures */
struct Point3D {
    union {
        struct {
            float x, y, z;
        };  /* Anonymous struct */
        float coords[3];
    };  /* Anonymous union */
};

/* Access members directly (no intermediate name) */
struct Point3D p;
p.x = 1.0f;    /* Direct access */
p.y = 2.0f;
p.z = 3.0f;

/* Also accessible via array */
p.coords[0] = 10.0f;  /* Same as p.x */
p.coords[1] = 20.0f;  /* Same as p.y */

/* Another example */
struct Person {
    char name[50];
    struct {         /* Anonymous struct */
        int day;
        int month;
        int year;
    } birthdate;     /* Named anonymous struct */
};

struct Person person;
strcpy(person.name, "Alice");
person.birthdate.day = 15;    /* Access through birthdate */
person.birthdate.month = 6;
person.birthdate.year = 1990;
```

### Opaque Types and Information Hiding

```c
/* Header file: list.h */
#ifndef LIST_H
#define LIST_H

/* Opaque type - definition hidden */
typedef struct List List;

/* Public interface */
List *list_create(void);
void list_destroy(List *list);
void list_append(List *list, int value);
int list_get(List *list, size_t index);
size_t list_size(List *list);

#endif

/* Implementation file: list.c */
#include "list.h"
#include <stdlib.h>

/* Actual definition (private) */
struct List {
    int *data;
    size_t size;
    size_t capacity;
};

List *list_create(void) {
    List *list = malloc(sizeof(List));
    if (list == NULL) return NULL;
    
    list->size = 0;
    list->capacity = 10;
    list->data = malloc(list->capacity * sizeof(int));
    
    if (list->data == NULL) {
        free(list);
        return NULL;
    }
    
    return list;
}

void list_destroy(List *list) {
    if (list != NULL) {
        free(list->data);
        free(list);
    }
}

/* Benefits of opaque types:
   - Encapsulation: Internal structure hidden
   - ABI stability: Can change implementation without breaking clients
   - Forced interface: Users must use provided functions
*/
```

### Self-Referential Structures

```c
/* Linked list node */
struct Node {
    int data;
    struct Node *next;  /* Pointer to same type */
};

/* Tree node */
struct TreeNode {
    int value;
    struct TreeNode *left;
    struct TreeNode *right;
};

/* Graph node with multiple connections */
struct GraphNode {
    int id;
    struct GraphNode **neighbors;  /* Array of pointers */
    size_t num_neighbors;
};

/* Creating linked list */
struct Node *create_node(int data) {
    struct Node *node = malloc(sizeof(struct Node));
    if (node != NULL) {
        node->data = data;
        node->next = NULL;
    }
    return node;
}

/* Usage */
struct Node *head = create_node(1);
head->next = create_node(2);
head->next->next = create_node(3);

/* Traverse */
struct Node *current = head;
while (current != NULL) {
    printf("%d -> ", current->data);
    current = current->next;
}
printf("NULL\n");
```

### Structure Assignment and Comparison

```c
struct Point {
    int x;
    int y;
};

/* Structure assignment (shallow copy) */
struct Point p1 = {10, 20};
struct Point p2;
p2 = p1;  /* All members copied */

printf("p2: (%d, %d)\n", p2.x, p2.y);  /* (10, 20) */

/* ⚠️ No direct comparison */
// if (p1 == p2) { }  /* ❌ Error: can't compare structures */

/* Must compare members individually */
int points_equal(struct Point a, struct Point b) {
    return a.x == b.x && a.y == b.y;
}

/* Or use memcmp (careful with padding!) */
if (memcmp(&p1, &p2, sizeof(struct Point)) == 0) {
    printf("Equal\n");
}  /* ⚠️ May fail if padding contains garbage */

/* Structure with pointers (shallow vs deep copy) */
struct String {
    char *data;
    size_t length;
};

struct String s1;
s1.data = malloc(100);
strcpy(s1.data, "Hello");
s1.length = 5;

struct String s2 = s1;  /* Shallow copy - both point to same memory! */

/* Deep copy required */
struct String deep_copy(struct String src) {
    struct String dest;
    dest.length = src.length;
    dest.data = malloc(src.length + 1);
    if (dest.data != NULL) {
        strcpy(dest.data, src.data);
    }
    return dest;
}
```

### Passing Structures to Functions

```c
struct Point {
    int x;
    int y;
};

/* Pass by value (copy entire structure) */
void print_point_by_value(struct Point p) {
    printf("Point: (%d, %d)\n", p.x, p.y);
    p.x = 100;  /* Modifies copy, not original */
}

/* Pass by pointer (efficient, can modify) */
void print_point_by_pointer(struct Point *p) {
    printf("Point: (%d, %d)\n", p->x, p->y);
    p->x = 100;  /* Modifies original */
}

/* Pass by const pointer (efficient, read-only) */
void print_point_by_const_pointer(const struct Point *p) {
    printf("Point: (%d, %d)\n", p->x, p->y);
    // p->x = 100;  /* ❌ Error: can't modify const */
}

/* Returning structures */
struct Point create_point(int x, int y) {
    struct Point p = {x, y};
    return p;  /* Returns copy (may be optimized) */
}

/* Usage */
struct Point p1 = {10, 20};
print_point_by_value(p1);
printf("After by_value: (%d, %d)\n", p1.x, p1.y);  /* (10, 20) - unchanged */

print_point_by_pointer(&p1);
printf("After by_pointer: (%d, %d)\n", p1.x, p1.y);  /* (100, 20) - modified */

/* Best practice: Use const pointers for read-only, pointers for modification */
```

### Common Pitfalls

```c
/* 1. Forgetting to initialize */
struct Point p;  /* Members contain garbage */
// printf("%d\n", p.x);  /* Undefined value */

/* Fix: Always initialize */
struct Point p = {0};  /* All members zero */

/* 2. Assuming no padding */
struct S {
    char c;
    int i;
};
/* Don't assume sizeof(struct S) == sizeof(char) + sizeof(int) */

/* 3. memcmp with padding */
struct S s1 = {1, 42};
struct S s2 = {1, 42};
/* Padding may differ! */
// if (memcmp(&s1, &s2, sizeof(struct S)) == 0) { }  /* May fail */

/* Fix: Compare members individually or zero padding */
memset(&s1, 0, sizeof(s1));
s1.c = 1;
s1.i = 42;

/* 4. Dangling pointers in structures */
struct String {
    char *data;
};

struct String s1;
s1.data = malloc(100);
struct String s2 = s1;  /* Both point to same memory */
free(s1.data);
// printf("%s\n", s2.data);  /* ❌ Use after free */
```

### Best Practices

1. **Order members largest to smallest** to minimize padding
2. **Use const pointers** when passing read-only structures
3. **Initialize all structures** explicitly
4. **Use designated initializers** (C99) for clarity
5. **Document structure layout** if size/alignment matters
6. **Use opaque types** for encapsulation
7. **Be careful with padding** in binary I/O
8. **Avoid memcmp** for comparison (padding issues)
9. **Deep copy** structures with pointer members
10. **Use typedef** for cleaner syntax

### Performance Implications

```c
/* Pass by value vs pointer */
struct LargeStruct {
    char data[1000];
};

/* Slow: Copies 1000 bytes */
void process_by_value(struct LargeStruct s) {
    /* ... */
}

/* Fast: Passes 8 bytes (pointer) */
void process_by_pointer(const struct LargeStruct *s) {
    /* ... */
}

/* Structure packing affects cache performance */
/* Well-aligned structures access memory more efficiently */

/* Benchmark */
#include <time.h>

void benchmark(void) {
    struct LargeStruct s;
    clock_t start, end;
    
    /* By value */
    start = clock();
    for (int i = 0; i < 1000000; i++) {
        process_by_value(s);
    }
    end = clock();
    printf("By value: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);
    
    /* By pointer */
    start = clock();
    for (int i = 0; i < 1000000; i++) {
        process_by_pointer(&s);
    }
    end = clock();
    printf("By pointer: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);
}
```

### SDLC Integration

**Requirements:**
- Define data models using structures
- Specify member types and relationships
- Document alignment requirements

**Design:**
- Design structure layout for efficiency
- Plan for opaque types when encapsulation needed
- Consider padding and alignment

**Implementation:**
- Use designated initializers for clarity
- Implement deep copy functions for pointer members
- Create constructors/destructors pattern

**Testing:**
- Test structure initialization
- Verify alignment assumptions
- Test with different compilers
- Check for padding issues in binary I/O

**Deployment:**
- Document structure versioning
- Ensure ABI compatibility
- Profile structure-heavy code

### FAQ

**Q1: When should I use structures vs arrays?**

Use structures when:
- Grouping different types
- Creating named fields
- Modeling real-world entities

Use arrays when:
- Storing same-type elements
- Need indexed access
- Processing collections

**Q2: How do I minimize structure padding?**

Order members from largest to smallest:
```c
struct Optimized {
    double d;   /* 8 bytes */
    void *ptr;  /* 8 bytes (64-bit) */
    int i;      /* 4 bytes */
    short s;    /* 2 bytes */
    char c;     /* 1 byte */
                /* 1 byte padding */
};  /* Total: 24 bytes */
```

**Q3: Can I have a structure with an array of itself?**

No, but you can have pointer to same type:
```c
// struct Node {
//     struct Node children[10];  /* ❌ Infinite size */
// };

struct Node {
    struct Node *children[10];  /* ✅ OK: array of pointers */
};
```

**Q4: What's the difference between . and -> operators?**

- `.` for direct structure access
- `->` for pointer to structure access

```c
struct Point p;
struct Point *ptr = &p;

p.x = 10;     /* Direct access with . */
ptr->x = 10;  /* Pointer access with -> */
              /* Same as: (*ptr).x = 10; */
```

**Q5: Why use opaque types?**

Benefits:
- **Encapsulation**: Hide implementation details
- **ABI stability**: Change internals without breaking clients
- **Maintainability**: Enforce using provided interface
- **Flexibility**: Easy to change implementation

### Interview Questions

**Q1: What is structure padding and why does it exist?**

**Answer:**

**Padding**: Extra bytes added between struct members for alignment.

**Why it exists:**
1. **CPU efficiency**: Most CPUs access aligned data faster
2. **Hardware requirements**: Some architectures crash on misaligned access
3. **Atomic operations**: Alignment required for atomic access

**Example:**
```c
struct Example {
    char c;      /* 1 byte */
                 /* 3 bytes padding (align int to 4-byte boundary) */
    int i;       /* 4 bytes */
    char c2;     /* 1 byte */
                 /* 3 bytes padding (struct size must be multiple of alignment) */
};  /* Total: 12 bytes */

/* Without padding, int 'i' would be at offset 1 (misaligned) */
/* With padding, int 'i' is at offset 4 (aligned) */
```

**Minimize padding:**
```c
struct Optimized {
    int i;       /* 4 bytes */
    char c;      /* 1 byte */
    char c2;     /* 1 byte */
                 /* 2 bytes padding */
};  /* Total: 8 bytes (33% smaller!) */
```

**Q2: Write code to swap two structures without using a temporary variable:**

**Answer:**

```c
struct Point {
    int x;
    int y;
};

void swap_points(struct Point *a, struct Point *b) {
    /* Method 1: XOR swap (only for same type) */
    if (a != b) {  /* Check they're different */
        a->x ^= b->x;
        b->x ^= a->x;
        a->x ^= b->x;
        
        a->y ^= b->y;
        b->y ^= a->y;
        a->y ^= b->y;
    }
}

/* Method 2: Arithmetic swap */
void swap_points_arithmetic(struct Point *a, struct Point *b) {
    if (a != b) {
        a->x = a->x + b->x;
        b->x = a->x - b->x;
        a->x = a->x - b->x;
        
        a->y = a->y + b->y;
        b->y = a->y - b->y;
        a->y = a->y - b->y;
    }
}

/* Method 3: memcpy (most practical, works for any struct) */
void swap_points_memcpy(struct Point *a, struct Point *b) {
    unsigned char temp[sizeof(struct Point)];
    memcpy(temp, a, sizeof(struct Point));
    memcpy(a, b, sizeof(struct Point));
    memcpy(b, temp, sizeof(struct Point));
}

/* Test */
struct Point p1 = {10, 20};
struct Point p2 = {30, 40};
swap_points_memcpy(&p1, &p2);
printf("p1: (%d, %d)\n", p1.x, p1.y);  /* (30, 40) */
printf("p2: (%d, %d)\n", p2.x, p2.y);  /* (10, 20) */
```

**Note**: Using a temporary variable is clearer and recommended in production code!

**Q3: Implement a simple opaque type for a stack:**

**Answer:**

```c
/* stack.h */
#ifndef STACK_H
#define STACK_H

#include <stddef.h>
#include <stdbool.h>

typedef struct Stack Stack;

Stack *stack_create(size_t capacity);
void stack_destroy(Stack *stack);
bool stack_push(Stack *stack, int value);
bool stack_pop(Stack *stack, int *value);
bool stack_peek(const Stack *stack, int *value);
bool stack_is_empty(const Stack *stack);
bool stack_is_full(const Stack *stack);
size_t stack_size(const Stack *stack);

#endif

/* stack.c */
#include "stack.h"
#include <stdlib.h>

struct Stack {
    int *data;
    size_t top;
    size_t capacity;
};

Stack *stack_create(size_t capacity) {
    Stack *stack = malloc(sizeof(Stack));
    if (stack == NULL) return NULL;
    
    stack->data = malloc(capacity * sizeof(int));
    if (stack->data == NULL) {
        free(stack);
        return NULL;
    }
    
    stack->top = 0;
    stack->capacity = capacity;
    return stack;
}

void stack_destroy(Stack *stack) {
    if (stack != NULL) {
        free(stack->data);
        free(stack);
    }
}

bool stack_push(Stack *stack, int value) {
    if (stack == NULL || stack->top >= stack->capacity) {
        return false;
    }
    stack->data[stack->top++] = value;
    return true;
}

bool stack_pop(Stack *stack, int *value) {
    if (stack == NULL || stack->top == 0) {
        return false;
    }
    if (value != NULL) {
        *value = stack->data[--stack->top];
    }
    return true;
}

bool stack_is_empty(const Stack *stack) {
    return stack == NULL || stack->top == 0;
}

size_t stack_size(const Stack *stack) {
    return stack != NULL ? stack->top : 0;
}
```

**Q4: What's wrong with this code?**

```c
struct Node {
    int data;
    struct Node next;
};
```

**Answer:**

**Problem:** Structure contains itself by value, creating infinite size.

```c
sizeof(struct Node) = sizeof(int) + sizeof(struct Node)
                   = sizeof(int) + sizeof(int) + sizeof(struct Node)
                   = ∞ (infinite recursion)
```

**Fix:** Use pointer to same type:

```c
struct Node {
    int data;
    struct Node *next;  /* ✅ Pointer has fixed size */
};

sizeof(struct Node) = sizeof(int) + sizeof(struct Node*)
                   = 4 + 8 = 12 bytes (typical 64-bit system)
```

**Q5: How would you implement a deep copy of this structure?**

```c
struct Person {
    char *name;
    int age;
    struct Person *spouse;
};
```

**Answer:**

```c
struct Person *person_deep_copy(const struct Person *src) {
    if (src == NULL) return NULL;
    
    struct Person *dest = malloc(sizeof(struct Person));
    if (dest == NULL) return NULL;
    
    /* Copy age (simple value) */
    dest->age = src->age;
    
    /* Deep copy name (string) */
    if (src->name != NULL) {
        dest->name = malloc(strlen(src->name) + 1);
        if (dest->name == NULL) {
            free(dest);
            return NULL;
        }
        strcpy(dest->name, src->name);
    } else {
        dest->name = NULL;
    }
    
    /* Handle spouse (prevent infinite recursion) */
    /* Option 1: Shallow copy (just copy pointer) */
    dest->spouse = src->spouse;
    
    /* Option 2: NULL out circular reference */
    // dest->spouse = NULL;
    
    /* Option 3: Track copied nodes (complex, needed for graphs) */
    /* Would require hash table to track src->dest mappings */
    
    return dest;
}

void person_deep_free(struct Person *p) {
    if (p != NULL) {
        free(p->name);
        /* Don't free spouse (shared reference or NULL) */
        free(p);
    }
}
```

### Key Takeaways

- Structures group related variables under single name
- Use designated initializers (C99) for clear initialization
- Order members largest-to-smallest to minimize padding
- Use `.` for direct access, `->` for pointer access
- Pass large structures by pointer for efficiency
- Structures cannot contain themselves, only pointers to same type
- Opaque types provide encapsulation
- Bit fields save space but have portability concerns
- Deep copy required for structures with pointer members
- Use offsetof to inspect memory layout
- Padding affects structure size and comparison

---

## 3.2 Unions

### Concept Introduction

A union is a special data type that allows storing different data types in the same memory location. Unlike structures where each member has its own memory, all union members share the same memory space. Only one member can hold a value at any time, and the union's size is determined by its largest member.

### Union Declaration and Usage

```c
/* Union declaration */
union Data {
    int i;
    float f;
    char str[20];
};

/* Union variable */
union Data data;

/* Accessing members */
data.i = 10;
printf("data.i = %d\n", data.i);

data.f = 3.14f;  /* Overwrites the int value */
printf("data.f = %f\n", data.f);
// printf("%d\n", data.i);  /* ❌ Undefined: no longer valid */

strcpy(data.str, "Hello");
printf("data.str = %s\n", data.str);
```

### Memory Layout of Unions

```c
union Number {
    int i;      /* 4 bytes */
    float f;    /* 4 bytes */
    double d;   /* 8 bytes */
};

printf("sizeof(union Number) = %zu\n", sizeof(union Number));  /* 8 bytes */

/* All members start at offset 0 */
union Number num;
printf("Address of i: %p\n", (void*)&num.i);
printf("Address of f: %p\n", (void*)&num.f);
printf("Address of d: %p\n", (void*)&num.d);
/* All print same address */

/* Memory visualization:
   +---+---+---+---+---+---+---+---+
   |   |   |   |   |   |   |   |   |  8 bytes total
   +---+---+---+---+---+---+---+---+
   ^int/float (4 bytes)
   ^double (8 bytes)
*/
```

### Type Punning with Unions

```c
/* Access bit representation of float */
union FloatBits {
    float f;
    uint32_t bits;
};

union FloatBits fb;
fb.f = 3.14f;

printf("Float: %f\n", fb.f);
printf("Bits: 0x%08X\n", fb.bits);

/* IEEE 754 decomposition */
uint32_t sign = (fb.bits >> 31) & 1;
uint32_t exponent = (fb.bits >> 23) & 0xFF;
uint32_t mantissa = fb.bits & 0x7FFFFF;

printf("Sign: %u, Exponent: %u, Mantissa: 0x%06X\n", 
       sign, exponent, mantissa);

/* ⚠️ Note: Accessing different union member than written
   is implementation-defined but widely supported */
```

### Tagged Unions (Discriminated Unions)

```c
/* Type tag enum */
enum DataType {
    TYPE_INT,
    TYPE_FLOAT,
    TYPE_STRING
};

/* Tagged union */
struct Variant {
    enum DataType type;  /* Tag indicates active member */
    union {
        int i;
        float f;
        char *str;
    } data;
};

/* Create variant */
struct Variant create_int(int value) {
    struct Variant v;
    v.type = TYPE_INT;
    v.data.i = value;
    return v;
}

struct Variant create_float(float value) {
    struct Variant v;
    v.type = TYPE_FLOAT;
    v.data.f = value;
    return v;
}

/* Use variant */
void print_variant(struct Variant v) {
    switch (v.type) {
        case TYPE_INT:
            printf("Int: %d\n", v.data.i);
            break;
        case TYPE_FLOAT:
            printf("Float: %f\n", v.data.f);
            break;
        case TYPE_STRING:
            printf("String: %s\n", v.data.str);
            break;
    }
}
```

### Endianness Detection

```c
union EndianTest {
    uint32_t value;
    uint8_t bytes[4];
};

int is_little_endian(void) {
    union EndianTest test;
    test.value = 0x01020304;
    
    return test.bytes[0] == 0x04;  /* Little-endian if LSB first */
}

/* Usage */
if (is_little_endian()) {
    printf("System is little-endian\n");
} else {
    printf("System is big-endian\n");
}
```

### Use Cases and Pitfalls

**Use Cases:**
1. Memory-efficient variant types
2. Type punning (bit manipulation)
3. Hardware register mapping
4. Protocol implementations

**Pitfalls:**
```c
/* ❌ Accessing wrong member */
union Data d;
d.i = 42;
printf("%f\n", d.f);  /* Undefined value */

/* ❌ Forgetting to track active member */
union Data d;
d.i = 10;
/* ... code ... */
printf("%f\n", d.f);  /* Which member is active? */

/* ✅ Always use tagged unions for safety */
```

---

## 3.3 Enumerations (enum)

### Enumeration Declaration

```c
/* Basic enum */
enum Color {
    RED,      /* 0 */
    GREEN,    /* 1 */
    BLUE      /* 2 */
};

/* With explicit values */
enum Status {
    OK = 0,
    ERROR = -1,
    PENDING = 1
};

/* Non-sequential values */
enum Flags {
    FLAG_NONE = 0,
    FLAG_READ = 1,
    FLAG_WRITE = 2,
    FLAG_EXECUTE = 4
};

/* Using previous value */
enum Month {
    JAN = 1,
    FEB,      /* 2 */
    MAR,      /* 3 */
    APR,      /* 4 */
    DEC = 12
};
```

### Enumeration Constants

```c
/* Enums are compile-time constants */
enum Size {
    SMALL = 10,
    MEDIUM = 20,
    LARGE = 30
};

/* Can use in array sizes */
int buffer[LARGE];

/* Can use in switch */
enum Color c = RED;
switch (c) {
    case RED:   printf("Red\n"); break;
    case GREEN: printf("Green\n"); break;
    case BLUE:  printf("Blue\n"); break;
}

/* Bit flags pattern */
enum Permission {
    PERM_READ    = 1 << 0,  /* 0b001 */
    PERM_WRITE   = 1 << 1,  /* 0b010 */
    PERM_EXECUTE = 1 << 2   /* 0b100 */
};

unsigned int perms = PERM_READ | PERM_WRITE;
if (perms & PERM_READ) {
    printf("Has read permission\n");
}
```

### Underlying Integer Type

```c
/* Enums are integers */
enum Color c = RED;
int i = c;  /* OK: implicit conversion */

printf("%d\n", RED);    /* 0 */
printf("%d\n", GREEN);  /* 1 */

/* Size is implementation-defined (usually int) */
printf("sizeof(enum Color) = %zu\n", sizeof(enum Color));

/* Can have negative values */
enum Temp {
    FREEZING = -10,
    COLD = 0,
    WARM = 20
};
```

### Enum vs #define

```c
/* #define */
#define COLOR_RED 0
#define COLOR_GREEN 1
#define COLOR_BLUE 2

/* enum */
enum Color {
    RED = 0,
    GREEN = 1,
    BLUE = 2
};

/* Differences:
   1. Scope: #define is global, enum can be scoped
   2. Type: enum has type, #define doesn't
   3. Debugging: enum names visible in debugger
   4. Namespace: enum constants share namespace
*/

/* ✅ Prefer enum for related constants */
```

### Enum Scope Rules

```c
/* C89/C90: Enums share namespace */
enum A { X = 1 };
enum B { X = 2 };  /* ❌ Error: redefinition of X */

/* Workaround: Use prefixes */
enum A { A_X = 1, A_Y = 2 };
enum B { B_X = 1, B_Y = 2 };

/* Or use typedef */
typedef enum { X = 1, Y = 2 } TypeA;
typedef enum { X = 3, Y = 4 } TypeB;  /* ❌ Still conflicts */

/* Better: descriptive names */
typedef enum {
    COLOR_RED,
    COLOR_GREEN,
    COLOR_BLUE
} Color;
```

### Best Practices

```c
/* 1. Use enums for related constants */
enum Direction { NORTH, SOUTH, EAST, WEST };

/* 2. Provide default/invalid value */
enum Status {
    STATUS_INVALID = -1,
    STATUS_OK = 0,
    STATUS_ERROR = 1
};

/* 3. Count values */
enum Color {
    RED,
    GREEN,
    BLUE,
    COLOR_COUNT  /* Useful for array sizes */
};
int color_names[COLOR_COUNT];

/* 4. Use typedef for convenience */
typedef enum {
    CMD_START,
    CMD_STOP,
    CMD_PAUSE
} Command;

Command cmd = CMD_START;  /* No need for 'enum' keyword */

/* 5. Document values if non-obvious */
enum ErrorCode {
    ERR_NONE = 0,      /* No error */
    ERR_NOMEM = -1,    /* Out of memory */
    ERR_IO = -2        /* I/O error */
};
```

---

## 3.4 Typedef

### Creating Type Aliases

```c
/* Basic typedef */
typedef int Age;
typedef unsigned long long uint64;

Age person_age = 30;
uint64 large_number = 1000000000000ULL;

/* typedef with structures */
typedef struct {
    int x;
    int y;
} Point;

Point p = {10, 20};  /* No need for 'struct' keyword */

/* typedef with existing struct */
struct Rectangle {
    int width;
    int height;
};
typedef struct Rectangle Rect;

Rect r = {100, 50};
```

### Typedef with Structures and Unions

```c
/* Anonymous struct with typedef */
typedef struct {
    char name[50];
    int age;
} Person;

/* Named struct with typedef */
typedef struct Node {
    int data;
    struct Node *next;  /* Must use struct keyword here */
} Node;

/* Union typedef */
typedef union {
    int i;
    float f;
} Number;

/* Usage */
Person p = {"Alice", 30};
Node *list = NULL;
Number n;
n.f = 3.14f;
```

### Typedef with Function Pointers

```c
/* Without typedef - complex */
int (*callback)(int, int);

/* With typedef - cleaner */
typedef int (*BinaryOp)(int, int);

BinaryOp operation;

/* Function pointer typedef for callbacks */
typedef void (*EventHandler)(void *data);

void on_click(void *data) {
    printf("Clicked: %s\n", (char*)data);
}

EventHandler handler = on_click;
handler("Button1");

/* Array of function pointers */
typedef int (*MathFunc)(int, int);

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

MathFunc operations[] = {add, sub};
int result = operations[0](10, 5);  /* 15 */
```

### Opaque Types

```c
/* Header file: stack.h */
typedef struct Stack Stack;  /* Forward declaration */

Stack *stack_create(void);
void stack_destroy(Stack *s);
void stack_push(Stack *s, int value);

/* Implementation file: stack.c */
struct Stack {  /* Actual definition */
    int *data;
    size_t size;
    size_t capacity;
};

/* Users can't access internals */
```

### Naming Conventions

```c
/* Common conventions:
   - CamelCase for types: Point, Rectangle
   - snake_case with _t suffix: point_t, rectangle_t
   - ALL_CAPS for constants (not types)
*/

/* POSIX-style with _t suffix */
typedef struct point point_t;
typedef unsigned int uint_t;

/* Avoid name conflicts */
/* ❌ Bad: conflicts with common names */
typedef int string;
typedef float real;

/* ✅ Good: descriptive, unique names */
typedef char *String;
typedef double Real;

/* Pointer typedef naming */
typedef struct Node Node;
typedef Node *NodePtr;  /* _Ptr suffix for pointer types */
```

### Typedef Best Practices

1. **Use for complex types** (function pointers, long declarations)
2. **Don't hide pointers** (controversial - make intent clear)
3. **Use for platform abstraction**
4. **Consistent naming** across project
5. **Document typedef'd types**

```c
/* ✅ Good uses */
typedef uint32_t (*HashFunction)(const void *key);
typedef struct {
    int real;
    int imag;
} Complex;

/* ⚠️ Debatable: hiding pointers */
typedef struct Node *NodePtr;
NodePtr node;  /* Is this a pointer? Not obvious */

/* ✅ Better: explicit pointer */
typedef struct Node Node;
Node *node;  /* Clearly a pointer */
```

---

## 3.5 Common Data Structures in C

### Linked Lists

**Singly Linked List:**

```c
typedef struct Node {
    int data;
    struct Node *next;
} Node;

/* Create node */
Node *node_create(int data) {
    Node *node = malloc(sizeof(Node));
    if (node != NULL) {
        node->data = data;
        node->next = NULL;
    }
    return node;
}

/* Insert at head */
void list_insert_head(Node **head, int data) {
    Node *new_node = node_create(data);
    if (new_node != NULL) {
        new_node->next = *head;
        *head = new_node;
    }
}

/* Delete list */
void list_destroy(Node *head) {
    while (head != NULL) {
        Node *temp = head;
        head = head->next;
        free(temp);
    }
}

/* Search */
Node *list_find(Node *head, int value) {
    while (head != NULL) {
        if (head->data == value) {
            return head;
        }
        head = head->next;
    }
    return NULL;
}
```

**Doubly Linked List:**

```c
typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

/* Insert after node */
void dlist_insert_after(DNode *node, int data) {
    if (node == NULL) return;
    
    DNode *new_node = malloc(sizeof(DNode));
    if (new_node == NULL) return;
    
    new_node->data = data;
    new_node->next = node->next;
    new_node->prev = node;
    
    if (node->next != NULL) {
        node->next->prev = new_node;
    }
    node->next = new_node;
}
```

### Stacks

```c
typedef struct {
    int *data;
    size_t top;
    size_t capacity;
} Stack;

Stack *stack_create(size_t capacity) {
    Stack *s = malloc(sizeof(Stack));
    if (s == NULL) return NULL;
    
    s->data = malloc(capacity * sizeof(int));
    if (s->data == NULL) {
        free(s);
        return NULL;
    }
    
    s->top = 0;
    s->capacity = capacity;
    return s;
}

int stack_push(Stack *s, int value) {
    if (s->top >= s->capacity) return -1;
    s->data[s->top++] = value;
    return 0;
}

int stack_pop(Stack *s, int *value) {
    if (s->top == 0) return -1;
    if (value != NULL) {
        *value = s->data[--s->top];
    }
    return 0;
}
```

### Queues

```c
typedef struct {
    int *data;
    size_t front;
    size_t rear;
    size_t size;
    size_t capacity;
} Queue;

Queue *queue_create(size_t capacity) {
    Queue *q = malloc(sizeof(Queue));
    if (q == NULL) return NULL;
    
    q->data = malloc(capacity * sizeof(int));
    if (q->data == NULL) {
        free(q);
        return NULL;
    }
    
    q->front = q->rear = q->size = 0;
    q->capacity = capacity;
    return q;
}

int queue_enqueue(Queue *q, int value) {
    if (q->size >= q->capacity) return -1;
    
    q->data[q->rear] = value;
    q->rear = (q->rear + 1) % q->capacity;
    q->size++;
    return 0;
}

int queue_dequeue(Queue *q, int *value) {
    if (q->size == 0) return -1;
    
    if (value != NULL) {
        *value = q->data[q->front];
    }
    q->front = (q->front + 1) % q->capacity;
    q->size--;
    return 0;
}
```

### Binary Trees

```c
typedef struct TreeNode {
    int value;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

TreeNode *tree_create_node(int value) {
    TreeNode *node = malloc(sizeof(TreeNode));
    if (node != NULL) {
        node->value = value;
        node->left = node->right = NULL;
    }
    return node;
}

/* BST insertion */
TreeNode *bst_insert(TreeNode *root, int value) {
    if (root == NULL) {
        return tree_create_node(value);
    }
    
    if (value < root->value) {
        root->left = bst_insert(root->left, value);
    } else if (value > root->value) {
        root->right = bst_insert(root->right, value);
    }
    
    return root;
}

/* Inorder traversal */
void tree_inorder(TreeNode *root) {
    if (root != NULL) {
        tree_inorder(root->left);
        printf("%d ", root->value);
        tree_inorder(root->right);
    }
}
```

### Hash Tables

```c
#define TABLE_SIZE 100

typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;  /* Collision handling */
} Entry;

typedef struct {
    Entry *buckets[TABLE_SIZE];
} HashTable;

unsigned int hash(const char *key) {
    unsigned int h = 0;
    while (*key) {
        h = h * 31 + *key++;
    }
    return h % TABLE_SIZE;
}

void ht_insert(HashTable *ht, const char *key, int value) {
    unsigned int index = hash(key);
    Entry *entry = malloc(sizeof(Entry));
    
    entry->key = strdup(key);
    entry->value = value;
    entry->next = ht->buckets[index];
    ht->buckets[index] = entry;
}

int ht_get(HashTable *ht, const char *key, int *value) {
    unsigned int index = hash(key);
    Entry *entry = ht->buckets[index];
    
    while (entry != NULL) {
        if (strcmp(entry->key, key) == 0) {
            if (value != NULL) {
                *value = entry->value;
            }
            return 0;
        }
        entry = entry->next;
    }
    return -1;
}
```

### Key Takeaways - Part 3

**Structures:**
- Group related data under single name
- Use designated initializers (C99)
- Order members largest-to-smallest (minimize padding)
- Use opaque types for encapsulation
- Pass by pointer for efficiency

**Unions:**
- Share memory among members
- Use tagged unions for type safety
- Good for variant types and type punning
- Only one member valid at a time

**Enumerations:**
- Named integer constants
- Better than #define for related values
- Type-safe in modern C
- Use for state machines, options, flags

**Typedef:**
- Create type aliases
- Simplify complex declarations
- Use for function pointers
- Enable opaque types
- Follow naming conventions

**Data Structures:**
- Build complex structures from basic types
- Linked lists for dynamic insertion
- Stacks for LIFO, queues for FIFO
- Trees for hierarchical data
- Hash tables for fast lookup
- Always handle memory carefully

---

## Part 3 Complete Summary

**Part 3: Derived Types & Data Structures** covered:

### 3.1 Structures
- Structure definition, initialization, and access
- Padding and alignment optimization
- Bit fields for compact storage
- Anonymous structures (C11)
- Opaque types for encapsulation
- Self-referential structures

### 3.2 Unions
- Union memory layout and usage
- Type punning techniques
- Tagged unions for type safety
- Endianness detection
- Use cases and pitfalls

### 3.3 Enumerations
- Enum declaration and constants
- Underlying integer types
- Enum vs #define comparison
- Scope rules and best practices
- Bit flag patterns

### 3.4 Typedef
- Type aliases and simplification
- Function pointer typedefs
- Opaque type implementation
- Naming conventions
- Best practices

### 3.5 Common Data Structures
- Linked lists (singly, doubly)
- Stacks and queues
- Binary search trees
- Hash tables
- Implementation patterns
- Memory management strategies

**Next:** Part 4 will cover Advanced C Features including file I/O, error handling, command-line processing, compilation/linking, and multi-file projects.
