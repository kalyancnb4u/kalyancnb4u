---
title: "Complete C Language Mastery Part 15: Interview Preparation"
date: 2024-02-22 00:00:00 +0000
categories: [C, Programming, Interview, Career]
tags: [c-language, interview-prep, coding-problems, technical-interview, career]
---

# Complete C Language Mastery Part 15: Interview Preparation

Welcome to Part 15 of the C Language Mastery series - the final part! This comprehensive guide prepares you for C programming interviews with common topics, coding problems, conceptual questions, and practical advice.

**Prerequisites**: Parts 1-14 (complete C mastery)

---

## 15.1 Common Interview Topics

### Pointer Manipulation and Arithmetic

**Q: What's the difference between `int *p` and `int* p`?**

A: No functional difference - just style. Both declare a pointer to int. However:
```c
int *p, q;    /* p is pointer, q is int */
int* p, q;    /* Misleading: q is still int, not pointer! */
```

**Q: What does `void (*signal(int sig, void (*func)(int)))(int)` mean?**

A: `signal` is a function that takes an int and a function pointer, and returns a function pointer.
```c
/* Clearer with typedef */
typedef void (*sighandler_t)(int);
sighandler_t signal(int sig, sighandler_t handler);
```

**Q: Pointer arithmetic - what's printed?**
```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;
printf("%d %d %d\n", *p, *(p+2), *++p);
```
A: `1 3 2` (note: `++p` modifies p, evaluation order matters)

### Memory Management

**Q: Find the bug:**
```c
char *get_string(void) {
    char buffer[100] = "Hello";
    return buffer;  /* ❌ Returning pointer to local variable */
}
```
A: Returns dangling pointer. Fix: use malloc or static.

**Q: What's wrong here?**
```c
int *arr = malloc(10);  /* ❌ Wrong size */
```
A: Allocates 10 bytes, not 10 ints. Should be `malloc(10 * sizeof(int))`.

### String Manipulation

**Q: Reverse a string in-place:**
```c
void reverse(char *str) {
    if (!str) return;
    char *end = str + strlen(str) - 1;
    while (str < end) {
        char temp = *str;
        *str++ = *end;
        *end-- = temp;
    }
}
```

**Q: Find the bug:**
```c
char *str = "Hello";
str[0] = 'h';  /* ❌ Modifying string literal - undefined */
```

### Bit Manipulation

**Q: Check if number is power of 2:**
```c
bool is_power_of_2(unsigned int n) {
    return n && !(n & (n - 1));
}
```

**Q: Count set bits:**
```c
int count_bits(unsigned int n) {
    int count = 0;
    while (n) {
        count += n & 1;
        n >>= 1;
    }
    return count;
}

/* Or Brian Kernighan's algorithm */
int count_bits_fast(unsigned int n) {
    int count = 0;
    while (n) {
        n &= n - 1;  /* Clear lowest set bit */
        count++;
    }
    return count;
}
```

---

## 15.2 Coding Problems

### Linked Lists

**Q: Reverse a linked list:**
```c
struct Node {
    int data;
    struct Node *next;
};

struct Node *reverse(struct Node *head) {
    struct Node *prev = NULL;
    struct Node *current = head;
    struct Node *next = NULL;
    
    while (current) {
        next = current->next;
        current->next = prev;
        prev = current;
        current = next;
    }
    
    return prev;
}
```

**Q: Detect cycle in linked list (Floyd's algorithm):**
```c
bool has_cycle(struct Node *head) {
    struct Node *slow = head;
    struct Node *fast = head;
    
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    
    return false;
}
```

**Q: Find middle element:**
```c
struct Node *find_middle(struct Node *head) {
    struct Node *slow = head;
    struct Node *fast = head;
    
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    
    return slow;
}
```

### Arrays

**Q: Find two numbers that sum to target:**
```c
bool two_sum(int *arr, int n, int target, int *idx1, int *idx2) {
    /* O(n) solution using hash table would be better */
    /* O(n²) naive solution: */
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (arr[i] + arr[j] == target) {
                *idx1 = i;
                *idx2 = j;
                return true;
            }
        }
    }
    return false;
}
```

**Q: Remove duplicates from sorted array (in-place):**
```c
int remove_duplicates(int *arr, int n) {
    if (n == 0) return 0;
    
    int j = 0;
    for (int i = 1; i < n; i++) {
        if (arr[i] != arr[j]) {
            j++;
            arr[j] = arr[i];
        }
    }
    
    return j + 1;
}
```

### Binary Trees

**Q: Tree traversals:**
```c
struct TreeNode {
    int value;
    struct TreeNode *left, *right;
};

/* Inorder: Left, Root, Right */
void inorder(struct TreeNode *root) {
    if (!root) return;
    inorder(root->left);
    printf("%d ", root->value);
    inorder(root->right);
}

/* Preorder: Root, Left, Right */
void preorder(struct TreeNode *root) {
    if (!root) return;
    printf("%d ", root->value);
    preorder(root->left);
    preorder(root->right);
}

/* Postorder: Left, Right, Root */
void postorder(struct TreeNode *root) {
    if (!root) return;
    postorder(root->left);
    postorder(root->right);
    printf("%d ", root->value);
}
```

**Q: Check if binary tree is BST:**
```c
bool is_bst_util(struct TreeNode *node, int min, int max) {
    if (!node) return true;
    if (node->value <= min || node->value >= max) return false;
    return is_bst_util(node->left, min, node->value) &&
           is_bst_util(node->right, node->value, max);
}

bool is_bst(struct TreeNode *root) {
    return is_bst_util(root, INT_MIN, INT_MAX);
}
```

---

## 15.3 Conceptual Questions

### Fundamental Differences

**Q: malloc vs calloc?**

A: 
- `malloc(size)`: Allocates uninitialized memory
- `calloc(n, size)`: Allocates zero-initialized memory for n elements
- `calloc` checks for overflow, slightly slower

**Q: Stack vs Heap?**

| Feature | Stack | Heap |
|---------|-------|------|
| Speed | Fast (pointer bump) | Slower (fragmentation) |
| Size | Limited (1-8 MB) | Large (GB) |
| Lifetime | Automatic | Manual (malloc/free) |
| Management | Compiler | Programmer |
| Allocation | LIFO order | Any order |

**Q: Pass by value vs reference in C?**

A: C is always pass-by-value. To modify caller's data, pass pointer (simulates pass-by-reference):
```c
void modify(int *p) {
    *p = 100;  /* Modifies caller's variable */
}

int x = 5;
modify(&x);  /* x is now 100 */
```

### Advanced Concepts

**Q: When to use volatile?**

A:
1. Hardware registers: `volatile uint32_t *GPIO = 0x40020000;`
2. Variables modified by ISRs: `volatile int flag;`
3. Variables modified by other threads (but prefer atomics)
4. Memory-mapped I/O

**Q: Explain const correctness:**

```c
const int *p;        /* Pointer to const int - can't modify *p */
int const *p;        /* Same as above */
int *const p;        /* Const pointer - can't modify p */
const int *const p;  /* Const pointer to const int */
```

**Q: Examples of undefined behavior?**

A:
1. Dereferencing NULL pointer
2. Buffer overflow
3. Using uninitialized variables
4. Signed integer overflow
5. Modifying string literals
6. Accessing freed memory
7. Shifting by negative or >= width

---

## 15.4 Systems Programming Questions

**Q: Process vs Thread?**

| Feature | Process | Thread |
|---------|---------|--------|
| Memory | Separate | Shared |
| Creation | Expensive | Cheap |
| Communication | IPC | Shared memory |
| Crash | Isolated | Affects all threads |
| Context switch | Slower | Faster |

**Q: Explain fork():**

```c
pid_t pid = fork();
if (pid == 0) {
    /* Child process */
    printf("Child PID: %d\n", getpid());
} else if (pid > 0) {
    /* Parent process */
    printf("Parent PID: %d, Child PID: %d\n", getpid(), pid);
} else {
    /* Error */
    perror("fork");
}
```

**Q: IPC mechanisms?**

A:
1. **Pipes**: Unidirectional, related processes
2. **FIFOs**: Named pipes, unrelated processes
3. **Message Queues**: Structured messages
4. **Shared Memory**: Fastest, requires synchronization
5. **Sockets**: Network or local, bidirectional
6. **Signals**: Asynchronous notifications

---

## 15.5 Behavioral and Design Questions

### Code Review

**Q: What's wrong with this code?**

```c
/* Multiple issues to identify */
void process_file(char *filename) {
    FILE *fp = fopen(filename, "r");
    char buffer[100];
    
    while (fgets(buffer, sizeof(buffer), fp)) {
        printf("%s", buffer);
    }
    
    fclose(fp);
}
```

Issues:
1. No NULL check on fopen
2. No error handling
3. Fixed buffer size (potential truncation)
4. No return value check on fgets

**Fixed:**
```c
int process_file(const char *filename) {
    FILE *fp = fopen(filename, "r");
    if (!fp) {
        perror("fopen");
        return -1;
    }
    
    char buffer[1024];
    while (fgets(buffer, sizeof(buffer), fp)) {
        if (fputs(buffer, stdout) == EOF) {
            fclose(fp);
            return -1;
        }
    }
    
    if (ferror(fp)) {
        fclose(fp);
        return -1;
    }
    
    fclose(fp);
    return 0;
}
```

### Performance Optimization

**Q: How would you optimize this?**

```c
/* O(n²) */
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        result += array[i] * array[j];
    }
}
```

A: Can be reduced to O(n):
```c
int sum = 0;
for (int i = 0; i < n; i++) {
    sum += array[i];
}
result = sum * sum;
```

---

## 15.6 Mock Interview Questions (Top 50)

### Must-Know Questions

1. **Implement strstr()** - Find substring
2. **Check palindrome** - With pointers
3. **Implement atoi()** - String to integer
4. **Binary search** - Recursive and iterative
5. **Merge sorted arrays** - In-place if possible
6. **Implement memcpy()** - Handle overlapping
7. **Find nth Fibonacci** - Recursive and iterative
8. **Check if linked list has cycle** - Floyd's algorithm
9. **Reverse words in string** - In-place
10. **Implement queue using stacks** - Two stacks

### Memory Management

11. **Detect memory leaks** - Code review
12. **Implement malloc()** - Basic version
13. **Fix double-free bug** - Debug scenario
14. **Memory pool implementation** - Fixed-size blocks
15. **Smart pointer in C** - Reference counting

### Bit Manipulation

16. **Swap without temp variable** - XOR trick
17. **Find missing number** - XOR all
18. **Check power of 4** - Bit pattern
19. **Reverse bits** - 32-bit integer
20. **Single number** - All others appear twice

### System Design

21. **Design LRU cache** - Hash + doubly linked list
22. **Thread-safe queue** - Producer-consumer
23. **Memory allocator** - Buddy system
24. **Log file rotation** - Size/time based
25. **Rate limiter** - Token bucket

### Common Pitfalls

26. **Explain this bug:** `if (x = 5)` - Assignment not comparison
27. **Why crashes?** `free(ptr); *ptr = 10;` - Use after free
28. **Race condition?** Multi-threaded counter - Need mutex
29. **Buffer overflow?** `gets(buffer)` - Use fgets
30. **Memory leak?** Forgot to free in error path

---

## Interview Tips

### Before Interview

1. **Review fundamentals**: Pointers, memory, strings
2. **Practice coding**: LeetCode, HackerRank
3. **Study complexity**: Big-O for common operations
4. **Know the company**: What C do they use?
5. **Prepare questions**: Ask about their C codebase

### During Interview

1. **Clarify requirements** before coding
2. **Think out loud** - explain your approach
3. **Start with brute force** then optimize
4. **Test edge cases**: NULL, empty, large inputs
5. **Discuss trade-offs**: Time vs space
6. **Write clean code**: Meaningful names, comments
7. **Check for bugs**: Walk through your code

### Common Mistakes

1. Not checking malloc return value
2. Forgetting null terminator in strings
3. Off-by-one errors in loops
4. Integer overflow in calculations
5. Not freeing allocated memory
6. Dereferencing before NULL check
7. Modifying string literals

### Time Management

- **5 min**: Understand problem
- **5 min**: Discuss approach
- **20 min**: Code solution
- **5 min**: Test and debug
- **5 min**: Discuss improvements

---

## Key Takeaways

**Technical Skills:**
- Master pointers and memory management
- Know common data structures cold
- Practice bit manipulation
- Understand systems concepts
- Be fluent in complexity analysis

**Problem Solving:**
- Clarify before coding
- Start simple, then optimize
- Test thoroughly
- Handle edge cases
- Explain trade-offs

**Communication:**
- Think out loud
- Explain your reasoning
- Ask clarifying questions
- Discuss alternative approaches
- Be honest about what you don't know

**Preparation:**
- Practice 100+ problems
- Review each part of this series
- Do mock interviews
- Time yourself
- Get comfortable with whiteboard/online editors

---

## Final Summary

This concludes the **Complete C Language Mastery** series! Over 15 comprehensive parts, we've covered everything from basic syntax to advanced interview preparation.

**Series Complete:**
- ✅ Parts 1-14: Complete C mastery (~84,600 words)
- ✅ Part 15: Interview Preparation (~2,200 words)
- **Total: ~86,800 words** across fifteen comprehensive parts

You now have a complete, professional-grade education in C programming. Good luck with your interviews and career!

---

## 🎓 Congratulations!

You've completed the **Complete C Language Mastery** series - one of the most comprehensive C programming resources available. You're now equipped with:

- Deep understanding of C fundamentals
- Systems programming expertise
- Performance optimization skills
- Security best practices
- Professional development workflows
- Interview readiness

**Thank you for this journey through C programming!**

All 15 parts are ready for publication on your Jekyll blog with the Chirpy theme.
