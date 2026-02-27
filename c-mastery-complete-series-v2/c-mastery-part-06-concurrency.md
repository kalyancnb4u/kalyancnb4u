---
title: "Complete C Language Mastery Part 6: Concurrency and Multithreading"
date: 2024-02-21 15:00:00 +0000
categories: [C, Programming, Systems, Concurrency]
tags: [c-language, threads, multithreading, concurrency, atomics, mutexes, synchronization, parallel-programming]
---

# Complete C Language Mastery Part 6: Concurrency and Multithreading

Welcome to Part 6 of the C Language Mastery series. This part explores concurrent programming in C, covering both C11 standard threads and POSIX threads (pthreads). Understanding concurrency is essential for writing efficient, scalable applications that leverage modern multi-core processors.

**Prerequisites**: Parts 1-5 (Fundamentals through Standard Library)

**Warning**: Concurrent programming is complex and error-prone. This part assumes careful study and practice.

---

## 6.1 Introduction to Concurrency in C

### Processes vs Threads

**Processes:**
- Independent programs with separate memory space
- Cannot directly access each other's memory
- Communication through IPC (pipes, shared memory, sockets)
- Heavier weight (more overhead)
- Crash isolation (one process crash doesn't affect others)

**Threads:**
- Lightweight execution units within a process
- Share same memory space and resources
- Can directly access shared memory
- Lighter weight (less overhead)
- No isolation (one thread crash can crash entire process)

```c
/* Conceptual visualization:

Process:
+----------------------------------+
| Process Memory Space             |
| +------------------------------+ |
| | Thread 1                     | |
| +------------------------------+ |
| +------------------------------+ |
| | Thread 2                     | |
| +------------------------------+ |
| Shared: Heap, Global variables   |
| Separate: Stack per thread       |
+----------------------------------+
*/
```

### Concurrency Models

**1. Shared Memory (Threads):**
```c
/* Multiple threads access shared data */
int shared_counter = 0;  /* Shared variable */

void *thread_func(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        shared_counter++;  /* ⚠️ Race condition! */
    }
    return NULL;
}
```

**2. Message Passing:**
```c
/* Threads communicate via messages (queues, channels) */
/* No shared mutable state */
/* More common in other languages (Go, Erlang) */
```

### Thread Safety and Reentrancy

**Thread-Safe Function:**
- Can be called by multiple threads simultaneously
- Produces correct results
- May use locks internally

**Reentrant Function:**
- Can be interrupted and called again before completing
- Uses only local variables or caller-provided storage
- No shared mutable state
- All reentrant functions are thread-safe
- Not all thread-safe functions are reentrant

```c
/* ❌ NOT thread-safe (uses static storage) */
char *not_thread_safe(void) {
    static char buffer[100];
    sprintf(buffer, "Result");
    return buffer;  /* Multiple threads will share same buffer */
}

/* ✅ Thread-safe (uses mutex) */
static char buffer[100];
static mtx_t mutex;

char *thread_safe(void) {
    mtx_lock(&mutex);
    sprintf(buffer, "Result");
    char *result = strdup(buffer);
    mtx_unlock(&mutex);
    return result;
}

/* ✅ Reentrant (no shared state) */
char *reentrant(char *buffer, size_t size) {
    snprintf(buffer, size, "Result");
    return buffer;  /* Caller provides storage */
}
```

### Critical Sections

**Critical Section**: Code that accesses shared mutable data.

```c
/* Critical section example */
int shared_counter = 0;
mtx_t counter_mutex;

void increment(void) {
    /* BEGIN CRITICAL SECTION */
    mtx_lock(&counter_mutex);
    shared_counter++;  /* Only one thread at a time */
    mtx_unlock(&counter_mutex);
    /* END CRITICAL SECTION */
}
```

### Race Conditions and Data Races

**Race Condition**: Outcome depends on thread execution order

**Data Race**: Multiple threads access same memory, at least one writes, without synchronization

```c
/* ❌ Data race example */
int x = 0;

void *thread1(void *arg) {
    x = 1;  /* Write */
    return NULL;
}

void *thread2(void *arg) {
    int y = x;  /* Read */
    return NULL;
}

/* Undefined behavior: no synchronization */

/* ✅ Fixed with mutex */
int x = 0;
mtx_t x_mutex;

void *thread1(void *arg) {
    mtx_lock(&x_mutex);
    x = 1;
    mtx_unlock(&x_mutex);
    return NULL;
}

void *thread2(void *arg) {
    mtx_lock(&x_mutex);
    int y = x;
    mtx_unlock(&x_mutex);
    return NULL;
}
```

---

## 6.2 Threads (threads.h) - C11

### Creating and Joining Threads

```c
#include <threads.h>
#include <stdio.h>

/* Thread function signature */
int thread_function(void *arg) {
    int *value = (int *)arg;
    printf("Thread received: %d\n", *value);
    return 0;  /* Return value retrieved by thrd_join */
}

int main(void) {
    thrd_t thread;
    int data = 42;
    
    /* Create thread */
    if (thrd_create(&thread, thread_function, &data) != thrd_success) {
        fprintf(stderr, "Failed to create thread\n");
        return 1;
    }
    
    /* Wait for thread to complete */
    int result;
    if (thrd_join(thread, &result) != thrd_success) {
        fprintf(stderr, "Failed to join thread\n");
        return 1;
    }
    
    printf("Thread returned: %d\n", result);
    return 0;
}
```

### Thread Detachment

```c
/* Detached threads run independently */
/* No need to join, resources cleaned up automatically */

int thread_func(void *arg) {
    /* Do work... */
    return 0;
}

int main(void) {
    thrd_t thread;
    thrd_create(&thread, thread_func, NULL);
    
    /* Detach thread - no join needed */
    thrd_detach(thread);
    
    /* Main can exit, thread continues in background */
    return 0;
}
```

### Thread Identification

```c
/* Get current thread ID */
thrd_t current = thrd_current();

/* Compare thread IDs */
thrd_t t1, t2;
thrd_create(&t1, func1, NULL);
thrd_create(&t2, func2, NULL);

if (thrd_equal(t1, t2)) {
    printf("Same thread\n");
} else {
    printf("Different threads\n");
}
```

### Thread-Local Storage

```c
/* Thread-local variables (each thread has own copy) */
thread_local int tls_variable = 0;

int thread_func(void *arg) {
    tls_variable = (int)(intptr_t)arg;
    printf("Thread %d: tls_variable = %d\n", 
           (int)(intptr_t)arg, tls_variable);
    return 0;
}

int main(void) {
    thrd_t threads[3];
    
    for (int i = 0; i < 3; i++) {
        thrd_create(&threads[i], thread_func, (void *)(intptr_t)i);
    }
    
    for (int i = 0; i < 3; i++) {
        thrd_join(threads[i], NULL);
    }
    
    /* Each thread had its own tls_variable */
    return 0;
}

/* Alternative: Thread-Specific Storage (TSS) */
tss_t key;

void tss_destructor(void *data) {
    free(data);
}

int main(void) {
    tss_create(&key, tss_destructor);
    
    /* In thread */
    int *data = malloc(sizeof(int));
    *data = 42;
    tss_set(key, data);
    
    /* Later in same thread */
    int *retrieved = tss_get(key);
    
    tss_delete(key);
    return 0;
}
```

### Thread Sleep and Yield

```c
#include <time.h>

/* Sleep for specified duration */
void sleep_example(void) {
    struct timespec duration;
    duration.tv_sec = 1;      /* 1 second */
    duration.tv_nsec = 500000000;  /* 500 milliseconds */
    
    struct timespec remaining;
    if (thrd_sleep(&duration, &remaining) < 0) {
        /* Sleep interrupted */
        printf("Sleep interrupted, remaining: %ld.%ld\n",
               remaining.tv_sec, remaining.tv_nsec);
    }
}

/* Yield CPU to other threads */
void yield_example(void) {
    while (condition) {
        /* Do some work */
        
        thrd_yield();  /* Give other threads a chance */
    }
}
```

---

## 6.3 Mutual Exclusion (threads.h) - C11

### Mutex Basics

```c
#include <threads.h>

mtx_t mutex;

void initialize(void) {
    /* Create plain mutex */
    if (mtx_init(&mutex, mtx_plain) != thrd_success) {
        fprintf(stderr, "Mutex initialization failed\n");
    }
}

void critical_section(void) {
    /* Lock mutex */
    mtx_lock(&mutex);
    
    /* Critical section - only one thread at a time */
    shared_data++;
    
    /* Unlock mutex */
    mtx_unlock(&mutex);
}

void cleanup(void) {
    mtx_destroy(&mutex);
}
```

### Mutex Types

```c
/* Plain mutex (default) */
mtx_init(&mutex, mtx_plain);

/* Timed mutex (supports timed locks) */
mtx_init(&mutex, mtx_timed);

/* Recursive mutex (same thread can lock multiple times) */
mtx_init(&mutex, mtx_plain | mtx_recursive);

/* Recursive example */
mtx_t recursive_mutex;
mtx_init(&recursive_mutex, mtx_plain | mtx_recursive);

void recursive_function(int depth) {
    mtx_lock(&recursive_mutex);
    
    if (depth > 0) {
        recursive_function(depth - 1);  /* Can lock again */
    }
    
    mtx_unlock(&recursive_mutex);
}
```

### Try-Lock Operations

```c
/* Non-blocking lock attempt */
void try_lock_example(void) {
    if (mtx_trylock(&mutex) == thrd_success) {
        /* Got the lock */
        shared_data++;
        mtx_unlock(&mutex);
    } else {
        /* Lock not available, do something else */
        printf("Lock busy, trying alternative action\n");
    }
}
```

### Timed Locks

```c
/* Lock with timeout */
void timed_lock_example(void) {
    struct timespec timeout;
    timespec_get(&timeout, TIME_UTC);
    timeout.tv_sec += 1;  /* 1 second timeout */
    
    int result = mtx_timedlock(&mutex, &timeout);
    if (result == thrd_success) {
        /* Got the lock */
        shared_data++;
        mtx_unlock(&mutex);
    } else if (result == thrd_timedout) {
        printf("Lock timeout\n");
    } else {
        printf("Lock error\n");
    }
}
```

### Deadlock Prevention

```c
/* ❌ Deadlock scenario */
mtx_t mutex1, mutex2;

void *thread1(void *arg) {
    mtx_lock(&mutex1);
    /* Context switch here... */
    mtx_lock(&mutex2);  /* Deadlock! */
    /* Work... */
    mtx_unlock(&mutex2);
    mtx_unlock(&mutex1);
    return NULL;
}

void *thread2(void *arg) {
    mtx_lock(&mutex2);
    /* Context switch here... */
    mtx_lock(&mutex1);  /* Deadlock! */
    /* Work... */
    mtx_unlock(&mutex1);
    mtx_unlock(&mutex2);
    return NULL;
}

/* ✅ Prevention: Lock ordering */
void *thread1_fixed(void *arg) {
    mtx_lock(&mutex1);  /* Always lock in same order */
    mtx_lock(&mutex2);
    /* Work... */
    mtx_unlock(&mutex2);
    mtx_unlock(&mutex1);
    return NULL;
}

void *thread2_fixed(void *arg) {
    mtx_lock(&mutex1);  /* Same order */
    mtx_lock(&mutex2);
    /* Work... */
    mtx_unlock(&mutex2);
    mtx_unlock(&mutex1);
    return NULL;
}

/* ✅ Prevention: Try-lock with backoff */
void *thread_with_backoff(void *arg) {
    while (1) {
        mtx_lock(&mutex1);
        
        if (mtx_trylock(&mutex2) == thrd_success) {
            /* Got both locks */
            /* Work... */
            mtx_unlock(&mutex2);
            mtx_unlock(&mutex1);
            break;
        } else {
            /* Couldn't get mutex2, release mutex1 and retry */
            mtx_unlock(&mutex1);
            thrd_yield();
        }
    }
    return NULL;
}
```

---

## 6.4 Condition Variables (threads.h) - C11

### Basic Usage

```c
#include <threads.h>

mtx_t mutex;
cnd_t condition;
int ready = 0;

/* Waiting thread */
int waiter(void *arg) {
    mtx_lock(&mutex);
    
    /* Wait for condition */
    while (!ready) {  /* Always use while, not if */
        cnd_wait(&condition, &mutex);
    }
    
    /* Condition met, mutex reacquired */
    printf("Condition met!\n");
    
    mtx_unlock(&mutex);
    return 0;
}

/* Signaling thread */
int signaler(void *arg) {
    thrd_sleep(&(struct timespec){.tv_sec = 1}, NULL);
    
    mtx_lock(&mutex);
    ready = 1;
    cnd_signal(&condition);  /* Wake one waiting thread */
    mtx_unlock(&mutex);
    
    return 0;
}

int main(void) {
    mtx_init(&mutex, mtx_plain);
    cnd_init(&condition);
    
    thrd_t t1, t2;
    thrd_create(&t1, waiter, NULL);
    thrd_create(&t2, signaler, NULL);
    
    thrd_join(t1, NULL);
    thrd_join(t2, NULL);
    
    cnd_destroy(&condition);
    mtx_destroy(&mutex);
    return 0;
}
```

### Producer-Consumer Pattern

```c
#define BUFFER_SIZE 10

typedef struct {
    int buffer[BUFFER_SIZE];
    int count;
    int in;
    int out;
    mtx_t mutex;
    cnd_t not_full;
    cnd_t not_empty;
} Queue;

void queue_init(Queue *q) {
    q->count = 0;
    q->in = 0;
    q->out = 0;
    mtx_init(&q->mutex, mtx_plain);
    cnd_init(&q->not_full);
    cnd_init(&q->not_empty);
}

void queue_push(Queue *q, int value) {
    mtx_lock(&q->mutex);
    
    /* Wait if buffer full */
    while (q->count == BUFFER_SIZE) {
        cnd_wait(&q->not_full, &q->mutex);
    }
    
    /* Add to buffer */
    q->buffer[q->in] = value;
    q->in = (q->in + 1) % BUFFER_SIZE;
    q->count++;
    
    /* Signal that buffer is not empty */
    cnd_signal(&q->not_empty);
    
    mtx_unlock(&q->mutex);
}

int queue_pop(Queue *q) {
    mtx_lock(&q->mutex);
    
    /* Wait if buffer empty */
    while (q->count == 0) {
        cnd_wait(&q->not_empty, &q->mutex);
    }
    
    /* Remove from buffer */
    int value = q->buffer[q->out];
    q->out = (q->out + 1) % BUFFER_SIZE;
    q->count--;
    
    /* Signal that buffer is not full */
    cnd_signal(&q->not_full);
    
    mtx_unlock(&q->mutex);
    return value;
}

/* Producer thread */
int producer(void *arg) {
    Queue *q = (Queue *)arg;
    for (int i = 0; i < 100; i++) {
        queue_push(q, i);
        printf("Produced: %d\n", i);
    }
    return 0;
}

/* Consumer thread */
int consumer(void *arg) {
    Queue *q = (Queue *)arg;
    for (int i = 0; i < 100; i++) {
        int value = queue_pop(q);
        printf("Consumed: %d\n", value);
    }
    return 0;
}
```

### Spurious Wakeups

```c
/* ❌ Wrong: Using if */
mtx_lock(&mutex);
if (!ready) {
    cnd_wait(&condition, &mutex);  /* May wake spuriously */
}
/* Assumes ready is true - WRONG! */
mtx_unlock(&mutex);

/* ✅ Correct: Using while */
mtx_lock(&mutex);
while (!ready) {  /* Check condition again after wakeup */
    cnd_wait(&condition, &mutex);
}
/* Now ready is guaranteed true */
mtx_unlock(&mutex);
```

### Broadcast vs Signal

```c
/* cnd_signal - Wake one waiting thread */
cnd_signal(&condition);

/* cnd_broadcast - Wake all waiting threads */
cnd_broadcast(&condition);

/* Example: Multiple waiters */
int barrier_count = 0;
int barrier_target = 5;

int barrier_wait(void *arg) {
    mtx_lock(&mutex);
    
    barrier_count++;
    
    if (barrier_count == barrier_target) {
        /* Last thread to arrive */
        barrier_count = 0;
        cnd_broadcast(&condition);  /* Wake all */
    } else {
        /* Wait for all threads */
        while (barrier_count != 0) {
            cnd_wait(&condition, &mutex);
        }
    }
    
    mtx_unlock(&mutex);
    return 0;
}
```

---

## 6.5 Atomics (stdatomic.h) - C11

### Atomic Types

```c
#include <stdatomic.h>

/* Atomic integer types */
atomic_int counter = 0;
atomic_long long_counter = 0;
atomic_bool flag = false;

/* Atomic pointer */
atomic_int *ptr_counter;

/* Generic atomic type */
_Atomic int generic_counter = 0;

/* All basic atomic operations are lock-free on most platforms */
```

### Atomic Operations

```c
/* Basic atomic operations */
atomic_int counter = ATOMIC_VAR_INIT(0);

/* Atomic store */
atomic_store(&counter, 10);

/* Atomic load */
int value = atomic_load(&counter);

/* Atomic exchange (swap) */
int old_value = atomic_exchange(&counter, 20);

/* Atomic compare-and-swap */
int expected = 20;
int desired = 30;
bool success = atomic_compare_exchange_strong(&counter, &expected, desired);
if (success) {
    printf("CAS succeeded\n");
} else {
    printf("CAS failed, counter was %d\n", expected);
}

/* Atomic arithmetic */
atomic_fetch_add(&counter, 5);   /* counter += 5 */
atomic_fetch_sub(&counter, 3);   /* counter -= 3 */
atomic_fetch_or(&counter, 0x01); /* counter |= 0x01 */
atomic_fetch_and(&counter, 0xFE);/* counter &= 0xFE */
atomic_fetch_xor(&counter, 0xFF);/* counter ^= 0xFF */

/* Increment/decrement (returns old value) */
int old = atomic_fetch_add(&counter, 1);  /* counter++ */
int old2 = atomic_fetch_sub(&counter, 1); /* counter-- */
```

### Memory Orders

```c
/* Memory ordering (affects how operations are reordered) */

/* Sequential consistency (default, strongest) */
atomic_store_explicit(&counter, 10, memory_order_seq_cst);
int val = atomic_load_explicit(&counter, memory_order_seq_cst);

/* Acquire-Release (synchronization) */
/* Release: Store, prevents earlier ops from moving after */
atomic_store_explicit(&ready, true, memory_order_release);

/* Acquire: Load, prevents later ops from moving before */
while (!atomic_load_explicit(&ready, memory_order_acquire)) {
    /* Spin */
}

/* Relaxed (no synchronization, only atomicity) */
atomic_fetch_add_explicit(&counter, 1, memory_order_relaxed);

/* Example: Spin lock with acquire-release */
atomic_flag lock = ATOMIC_FLAG_INIT;

void acquire_lock(void) {
    while (atomic_flag_test_and_set_explicit(&lock, memory_order_acquire)) {
        /* Spin */
    }
}

void release_lock(void) {
    atomic_flag_clear_explicit(&lock, memory_order_release);
}
```

### Atomic Flags

```c
/* atomic_flag: Simplest atomic type (lock-free guaranteed) */
atomic_flag flag = ATOMIC_FLAG_INIT;

/* Test and set (returns previous value) */
bool was_set = atomic_flag_test_and_set(&flag);

/* Clear */
atomic_flag_clear(&flag);

/* Simple spinlock using atomic_flag */
typedef struct {
    atomic_flag lock;
} Spinlock;

void spinlock_init(Spinlock *s) {
    atomic_flag_clear(&s->lock);
}

void spinlock_lock(Spinlock *s) {
    while (atomic_flag_test_and_set_explicit(&s->lock, memory_order_acquire)) {
        /* Spin - maybe add pause/yield */
    }
}

void spinlock_unlock(Spinlock *s) {
    atomic_flag_clear_explicit(&s->lock, memory_order_release);
}
```

### Lock-Free Programming

```c
/* Lock-free stack (Treiber stack) */
typedef struct Node {
    int value;
    struct Node *next;
} Node;

typedef struct {
    _Atomic(Node *) head;
} LockFreeStack;

void stack_init(LockFreeStack *stack) {
    atomic_init(&stack->head, NULL);
}

void stack_push(LockFreeStack *stack, int value) {
    Node *new_node = malloc(sizeof(Node));
    new_node->value = value;
    
    Node *old_head;
    do {
        old_head = atomic_load(&stack->head);
        new_node->next = old_head;
    } while (!atomic_compare_exchange_weak(&stack->head, &old_head, new_node));
}

bool stack_pop(LockFreeStack *stack, int *value) {
    Node *old_head;
    Node *new_head;
    
    do {
        old_head = atomic_load(&stack->head);
        if (old_head == NULL) {
            return false;  /* Empty */
        }
        new_head = old_head->next;
    } while (!atomic_compare_exchange_weak(&stack->head, &old_head, new_head));
    
    *value = old_head->value;
    free(old_head);  /* ⚠️ ABA problem possible */
    return true;
}
```

---

## 6.6 POSIX Threads (pthreads)

### pthread_create and pthread_join

```c
#include <pthread.h>
#include <stdio.h>

void *thread_function(void *arg) {
    int *value = (int *)arg;
    printf("Thread received: %d\n", *value);
    return (void *)(intptr_t)(*value * 2);
}

int main(void) {
    pthread_t thread;
    int data = 42;
    
    /* Create thread */
    int result = pthread_create(&thread, NULL, thread_function, &data);
    if (result != 0) {
        fprintf(stderr, "pthread_create failed: %d\n", result);
        return 1;
    }
    
    /* Wait for thread */
    void *return_value;
    pthread_join(thread, &return_value);
    
    printf("Thread returned: %d\n", (int)(intptr_t)return_value);
    return 0;
}

/* Compile: gcc -pthread program.c */
```

### pthread_mutex_t

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

/* Or dynamic initialization */
pthread_mutex_t mutex;
pthread_mutex_init(&mutex, NULL);

/* Lock */
pthread_mutex_lock(&mutex);
/* Critical section */
pthread_mutex_unlock(&mutex);

/* Try lock */
if (pthread_mutex_trylock(&mutex) == 0) {
    /* Got lock */
    pthread_mutex_unlock(&mutex);
}

/* Timed lock */
struct timespec timeout;
clock_gettime(CLOCK_REALTIME, &timeout);
timeout.tv_sec += 1;  /* 1 second timeout */

if (pthread_mutex_timedlock(&mutex, &timeout) == 0) {
    /* Got lock */
    pthread_mutex_unlock(&mutex);
}

/* Cleanup */
pthread_mutex_destroy(&mutex);
```

### pthread_cond_t

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int ready = 0;

/* Waiting thread */
void *waiter(void *arg) {
    pthread_mutex_lock(&mutex);
    
    while (!ready) {
        pthread_cond_wait(&cond, &mutex);
    }
    
    printf("Condition met\n");
    pthread_mutex_unlock(&mutex);
    return NULL;
}

/* Signaling thread */
void *signaler(void *arg) {
    sleep(1);
    
    pthread_mutex_lock(&mutex);
    ready = 1;
    pthread_cond_signal(&cond);  /* Wake one */
    /* pthread_cond_broadcast(&cond); // Wake all */
    pthread_mutex_unlock(&mutex);
    
    return NULL;
}
```

### Read-Write Locks

```c
/* Multiple readers OR one writer */
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

/* Reader */
void *reader(void *arg) {
    pthread_rwlock_rdlock(&rwlock);
    /* Read shared data - multiple readers OK */
    printf("Reading data\n");
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}

/* Writer */
void *writer(void *arg) {
    pthread_rwlock_wrlock(&rwlock);
    /* Write shared data - exclusive access */
    printf("Writing data\n");
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}

/* Try variants */
pthread_rwlock_tryrdlock(&rwlock);
pthread_rwlock_trywrlock(&rwlock);

/* Timed variants */
struct timespec timeout;
clock_gettime(CLOCK_REALTIME, &timeout);
timeout.tv_sec += 1;
pthread_rwlock_timedrdlock(&rwlock, &timeout);
pthread_rwlock_timedwrlock(&rwlock, &timeout);

/* Cleanup */
pthread_rwlock_destroy(&rwlock);
```

### Barriers

```c
/* Synchronization point for multiple threads */
pthread_barrier_t barrier;

/* Initialize for N threads */
pthread_barrier_init(&barrier, NULL, 3);

void *thread_func(void *arg) {
    printf("Thread %ld doing work\n", (long)arg);
    
    /* Wait for all threads */
    pthread_barrier_wait(&barrier);
    
    printf("Thread %ld continuing\n", (long)arg);
    return NULL;
}

int main(void) {
    pthread_t threads[3];
    
    for (long i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, thread_func, (void *)i);
    }
    
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    pthread_barrier_destroy(&barrier);
    return 0;
}
```

### Spinlocks

```c
/* Busy-wait lock (use for very short critical sections) */
pthread_spinlock_t spinlock;

pthread_spin_init(&spinlock, PTHREAD_PROCESS_PRIVATE);

pthread_spin_lock(&spinlock);
/* Very short critical section */
pthread_spin_unlock(&spinlock);

pthread_spin_trylock(&spinlock);

pthread_spin_destroy(&spinlock);

/* ⚠️ Use spinlocks only when:
   - Critical section is very short (few instructions)
   - Lock will be held briefly
   - Preemption is disabled or rare
   
   Otherwise use mutex (yields CPU when waiting)
*/
```

---

## 6.7 Concurrency Patterns

### Thread Pool

```c
#include <pthread.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Task {
    void (*function)(void *);
    void *argument;
    struct Task *next;
} Task;

typedef struct {
    Task *head;
    Task *tail;
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    bool shutdown;
    pthread_t *threads;
    int thread_count;
} ThreadPool;

ThreadPool *pool_create(int num_threads) {
    ThreadPool *pool = malloc(sizeof(ThreadPool));
    pool->head = pool->tail = NULL;
    pool->shutdown = false;
    pool->thread_count = num_threads;
    pool->threads = malloc(sizeof(pthread_t) * num_threads);
    
    pthread_mutex_init(&pool->mutex, NULL);
    pthread_cond_init(&pool->cond, NULL);
    
    for (int i = 0; i < num_threads; i++) {
        pthread_create(&pool->threads[i], NULL, worker_thread, pool);
    }
    
    return pool;
}

void *worker_thread(void *arg) {
    ThreadPool *pool = (ThreadPool *)arg;
    
    while (1) {
        pthread_mutex_lock(&pool->mutex);
        
        /* Wait for task */
        while (pool->head == NULL && !pool->shutdown) {
            pthread_cond_wait(&pool->cond, &pool->mutex);
        }
        
        if (pool->shutdown) {
            pthread_mutex_unlock(&pool->mutex);
            break;
        }
        
        /* Get task */
        Task *task = pool->head;
        pool->head = task->next;
        if (pool->head == NULL) {
            pool->tail = NULL;
        }
        
        pthread_mutex_unlock(&pool->mutex);
        
        /* Execute task */
        task->function(task->argument);
        free(task);
    }
    
    return NULL;
}

void pool_submit(ThreadPool *pool, void (*func)(void *), void *arg) {
    Task *task = malloc(sizeof(Task));
    task->function = func;
    task->argument = arg;
    task->next = NULL;
    
    pthread_mutex_lock(&pool->mutex);
    
    if (pool->tail == NULL) {
        pool->head = pool->tail = task;
    } else {
        pool->tail->next = task;
        pool->tail = task;
    }
    
    pthread_cond_signal(&pool->cond);
    pthread_mutex_unlock(&pool->mutex);
}

void pool_destroy(ThreadPool *pool) {
    pthread_mutex_lock(&pool->mutex);
    pool->shutdown = true;
    pthread_cond_broadcast(&pool->cond);
    pthread_mutex_unlock(&pool->mutex);
    
    for (int i = 0; i < pool->thread_count; i++) {
        pthread_join(pool->threads[i], NULL);
    }
    
    pthread_mutex_destroy(&pool->mutex);
    pthread_cond_destroy(&pool->cond);
    free(pool->threads);
    free(pool);
}
```

### Producer-Consumer Queue (Lock-Free)

```c
/* Lock-free queue using atomics */
typedef struct QueueNode {
    void *data;
    _Atomic(struct QueueNode *) next;
} QueueNode;

typedef struct {
    _Atomic(QueueNode *) head;
    _Atomic(QueueNode *) tail;
} LockFreeQueue;

void queue_init(LockFreeQueue *q) {
    QueueNode *dummy = malloc(sizeof(QueueNode));
    dummy->data = NULL;
    atomic_init(&dummy->next, NULL);
    atomic_init(&q->head, dummy);
    atomic_init(&q->tail, dummy);
}

void queue_push(LockFreeQueue *q, void *data) {
    QueueNode *node = malloc(sizeof(QueueNode));
    node->data = data;
    atomic_init(&node->next, NULL);
    
    QueueNode *tail;
    while (1) {
        tail = atomic_load(&q->tail);
        QueueNode *next = atomic_load(&tail->next);
        
        if (tail == atomic_load(&q->tail)) {
            if (next == NULL) {
                if (atomic_compare_exchange_weak(&tail->next, &next, node)) {
                    break;
                }
            } else {
                atomic_compare_exchange_weak(&q->tail, &tail, next);
            }
        }
    }
    atomic_compare_exchange_weak(&q->tail, &tail, node);
}

void *queue_pop(LockFreeQueue *q) {
    void *data;
    QueueNode *head;
    
    while (1) {
        head = atomic_load(&q->head);
        QueueNode *tail = atomic_load(&q->tail);
        QueueNode *next = atomic_load(&head->next);
        
        if (head == atomic_load(&q->head)) {
            if (head == tail) {
                if (next == NULL) {
                    return NULL;  /* Queue empty */
                }
                atomic_compare_exchange_weak(&q->tail, &tail, next);
            } else {
                data = next->data;
                if (atomic_compare_exchange_weak(&q->head, &head, next)) {
                    break;
                }
            }
        }
    }
    
    free(head);
    return data;
}
```

### Reader-Writer Lock Pattern

```c
/* Custom reader-writer lock using atomics */
typedef struct {
    atomic_int readers;
    atomic_bool writer;
    pthread_mutex_t mutex;
    pthread_cond_t readers_done;
} RWLock;

void rwlock_init(RWLock *lock) {
    atomic_init(&lock->readers, 0);
    atomic_init(&lock->writer, false);
    pthread_mutex_init(&lock->mutex, NULL);
    pthread_cond_init(&lock->readers_done, NULL);
}

void rwlock_read_lock(RWLock *lock) {
    pthread_mutex_lock(&lock->mutex);
    
    while (atomic_load(&lock->writer)) {
        pthread_cond_wait(&lock->readers_done, &lock->mutex);
    }
    
    atomic_fetch_add(&lock->readers, 1);
    pthread_mutex_unlock(&lock->mutex);
}

void rwlock_read_unlock(RWLock *lock) {
    if (atomic_fetch_sub(&lock->readers, 1) == 1) {
        pthread_cond_signal(&lock->readers_done);
    }
}

void rwlock_write_lock(RWLock *lock) {
    pthread_mutex_lock(&lock->mutex);
    
    bool expected = false;
    while (!atomic_compare_exchange_weak(&lock->writer, &expected, true)) {
        expected = false;
        pthread_cond_wait(&lock->readers_done, &lock->mutex);
    }
    
    while (atomic_load(&lock->readers) > 0) {
        pthread_cond_wait(&lock->readers_done, &lock->mutex);
    }
    
    pthread_mutex_unlock(&lock->mutex);
}

void rwlock_write_unlock(RWLock *lock) {
    atomic_store(&lock->writer, false);
    pthread_cond_broadcast(&lock->readers_done);
}
```

### Common Pitfalls and Debugging

**1. Data Races:**
```c
/* ❌ Data race */
int shared = 0;
void *thread(void *arg) {
    shared++;  /* Not atomic! */
    return NULL;
}

/* ✅ Fix with atomic */
atomic_int shared = 0;
void *thread(void *arg) {
    atomic_fetch_add(&shared, 1);
    return NULL;
}

/* ✅ Or with mutex */
pthread_mutex_t mutex;
int shared = 0;
void *thread(void *arg) {
    pthread_mutex_lock(&mutex);
    shared++;
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

**2. Deadlocks:**
```c
/* Prevention strategies:
   1. Lock ordering
   2. Try-lock with backoff
   3. Deadlock detection
   4. Avoid holding multiple locks
*/
```

**3. Race Conditions:**
```c
/* ❌ TOCTOU (Time-of-check-time-of-use) */
if (file_exists(path)) {
    /* File might be deleted here */
    open_file(path);  /* Race! */
}

/* ✅ Atomic operation */
int fd = open(path, O_CREAT | O_EXCL);
```

**Debugging Tools:**
```bash
# ThreadSanitizer (detects data races)
gcc -fsanitize=thread -g program.c -pthread

# Helgrind (Valgrind tool for threads)
valgrind --tool=helgrind ./program

# DRD (Valgrind tool, lighter than Helgrind)
valgrind --tool=drd ./program
```

---

## Best Practices

1. **Minimize shared mutable state**
2. **Use coarse-grained locks initially** (optimize later)
3. **Always use while loops** with condition variables
4. **Consistent lock ordering** to prevent deadlocks
5. **Keep critical sections short**
6. **Prefer high-level synchronization** (condition variables over busy-wait)
7. **Use atomics for simple counters/flags**
8. **Document synchronization requirements**
9. **Test with ThreadSanitizer/Helgrind**
10. **Profile before optimizing** lock granularity

---

## Key Takeaways

**Concurrency Fundamentals:**
- Threads share memory space (lighter than processes)
- Thread-safety requires synchronization
- Race conditions cause undefined behavior
- Critical sections need protection

**C11 Threads (threads.h):**
- thrd_create, thrd_join for thread lifecycle
- mtx_lock, mtx_unlock for mutual exclusion
- cnd_wait, cnd_signal for condition variables
- thread_local for per-thread storage

**Atomics (stdatomic.h):**
- Lock-free programming with atomic operations
- Memory ordering controls reordering
- atomic_flag for simple spinlocks
- CAS for lock-free data structures

**POSIX Threads (pthreads):**
- More features than C11 threads
- pthread_mutex_t for locks
- pthread_rwlock_t for multiple readers
- pthread_barrier_t for synchronization points

**Patterns:**
- Thread pools for task management
- Producer-consumer for data pipelines
- Reader-writer locks for read-heavy workloads
- Lock-free structures for high performance

**Common Issues:**
- Data races (use ThreadSanitizer)
- Deadlocks (lock ordering, timeout)
- Spurious wakeups (always use while with cond_wait)
- ABA problem (in lock-free code)

---

## Part 6 Complete Summary

**Part 6: Concurrency and Multithreading** covered:

### 6.1 Introduction to Concurrency
- Processes vs threads comparison
- Concurrency models (shared memory, message passing)
- Thread safety and reentrancy
- Critical sections and race conditions

### 6.2 Threads (threads.h - C11)
- Creating and joining threads
- Thread detachment
- Thread identification
- Thread-local storage (TLS/TSS)
- Sleep and yield operations

### 6.3 Mutual Exclusion (C11)
- Mutex basics and types
- Try-lock and timed locks
- Deadlock prevention strategies
- Mutex best practices

### 6.4 Condition Variables (C11)
- Wait, signal, and broadcast
- Producer-consumer pattern
- Spurious wakeups handling
- Condition variable patterns

### 6.5 Atomics (stdatomic.h - C11)
- Atomic types and operations
- Memory ordering (seq_cst, acquire, release, relaxed)
- Atomic flags and spinlocks
- Lock-free programming with CAS
- Memory barriers

### 6.6 POSIX Threads (pthreads)
- pthread_create and pthread_join
- pthread_mutex_t for mutual exclusion
- pthread_cond_t for conditions
- Read-write locks (pthread_rwlock_t)
- Barriers (pthread_barrier_t)
- Spinlocks

### 6.7 Concurrency Patterns
- Thread pool implementation
- Lock-free queue
- Custom reader-writer locks
- Common pitfalls and debugging
- Tools (ThreadSanitizer, Helgrind, DRD)

**Series Progress:**
- ✅ Part 1: C Fundamentals & Language Core (30,438 words)
- ✅ Part 2: Memory Management & Pointers (13,312 words)
- ✅ Part 3: Derived Types & Data Structures (6,247 words)
- ✅ Part 4: Advanced C Features (5,361 words)
- ✅ Part 5: C Standard Library (4,542 words)
- ✅ Part 6: Concurrency and Multithreading (5,100+ words)
- **Total: ~65,000 words** across six comprehensive parts

All parts ready for publication on your Jekyll blog with the Chirpy theme!

**Important Note**: Concurrency is complex and error-prone. This part provides foundation and patterns, but real-world concurrent programming requires extensive testing, understanding of platform-specific behaviors, and careful design.
