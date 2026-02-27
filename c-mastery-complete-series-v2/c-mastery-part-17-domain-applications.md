---
title: "Complete C Language Mastery Part 17: Domain-Specific Applications"
date: 2024-02-22 02:00:00 +0000
categories: [C, Programming, Applications, Systems]
tags: [c-language, os-dev, networking, databases, graphics, scientific-computing, compilers]
---

# Complete C Language Mastery Part 17: Domain-Specific Applications

Welcome to Part 17 of the C Language Mastery series. This part explores C in specialized domains: operating system development, network programming, database systems, graphics and games, scientific computing, and compiler development.

**Prerequisites**: Parts 1-16 (complete C mastery including advanced topics)

---

## 17.1 Operating System Development

### Kernel Programming Basics

**Minimal kernel structure:**

```c
/* boot.S - Assembly entry point */
.section .text
.global _start
_start:
    /* Set up stack */
    mov $stack_top, %esp
    call kernel_main
    hlt

.section .bss
.align 16
stack_bottom:
    .skip 16384  /* 16KB stack */
stack_top:

/* kernel.c */
void kernel_main(void) {
    const char *str = "Hello from kernel!";
    char *video_memory = (char *)0xB8000;  /* VGA text mode */
    
    for (int i = 0; str[i] != '\0'; i++) {
        video_memory[i * 2] = str[i];
        video_memory[i * 2 + 1] = 0x07;  /* Light grey on black */
    }
    
    while (1) {
        __asm__("hlt");  /* Halt CPU */
    }
}
```

### Interrupt Handling

```c
/* Interrupt Descriptor Table (IDT) */
struct idt_entry {
    uint16_t offset_low;
    uint16_t selector;
    uint8_t zero;
    uint8_t type_attr;
    uint16_t offset_high;
} __attribute__((packed));

struct idt_ptr {
    uint16_t limit;
    uint32_t base;
} __attribute__((packed));

struct idt_entry idt[256];
struct idt_ptr idtp;

void idt_set_gate(uint8_t num, uint32_t base, uint16_t sel, uint8_t flags) {
    idt[num].offset_low = base & 0xFFFF;
    idt[num].selector = sel;
    idt[num].zero = 0;
    idt[num].type_attr = flags;
    idt[num].offset_high = (base >> 16) & 0xFFFF;
}

void idt_install(void) {
    idtp.limit = (sizeof(struct idt_entry) * 256) - 1;
    idtp.base = (uint32_t)&idt;
    
    memset(&idt, 0, sizeof(struct idt_entry) * 256);
    
    /* Set up ISRs */
    idt_set_gate(0, (uint32_t)isr0, 0x08, 0x8E);
    idt_set_gate(1, (uint32_t)isr1, 0x08, 0x8E);
    /* ... more ISRs ... */
    
    __asm__ volatile("lidt %0" : : "m"(idtp));
}
```

### Memory Management in OS

```c
/* Page frame allocator */
#define PAGE_SIZE 4096
#define MAX_PAGES 1024

uint32_t page_frames[MAX_PAGES / 32];  /* Bitmap */

void set_frame(uint32_t frame_addr) {
    uint32_t frame = frame_addr / PAGE_SIZE;
    uint32_t idx = frame / 32;
    uint32_t off = frame % 32;
    page_frames[idx] |= (1 << off);
}

void clear_frame(uint32_t frame_addr) {
    uint32_t frame = frame_addr / PAGE_SIZE;
    uint32_t idx = frame / 32;
    uint32_t off = frame % 32;
    page_frames[idx] &= ~(1 << off);
}

uint32_t first_free_frame(void) {
    for (uint32_t i = 0; i < MAX_PAGES / 32; i++) {
        if (page_frames[i] != 0xFFFFFFFF) {
            for (int j = 0; j < 32; j++) {
                if (!(page_frames[i] & (1 << j))) {
                    return i * 32 + j;
                }
            }
        }
    }
    return -1;
}
```

---

## 17.2 Network Programming

### TCP Server

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

int create_tcp_server(int port) {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) return -1;
    
    /* Allow address reuse */
    int opt = 1;
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    
    struct sockaddr_in address = {
        .sin_family = AF_INET,
        .sin_addr.s_addr = INADDR_ANY,
        .sin_port = htons(port)
    };
    
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        close(server_fd);
        return -1;
    }
    
    if (listen(server_fd, 10) < 0) {
        close(server_fd);
        return -1;
    }
    
    return server_fd;
}

void handle_client(int client_fd) {
    char buffer[1024];
    ssize_t n;
    
    while ((n = read(client_fd, buffer, sizeof(buffer) - 1)) > 0) {
        buffer[n] = '\0';
        printf("Received: %s\n", buffer);
        write(client_fd, buffer, n);  /* Echo back */
    }
    
    close(client_fd);
}
```

### Asynchronous I/O with epoll

```c
#include <sys/epoll.h>

int epoll_fd = epoll_create1(0);
struct epoll_event event, events[MAX_EVENTS];

/* Add server socket to epoll */
event.events = EPOLLIN;
event.data.fd = server_fd;
epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &event);

while (1) {
    int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
    
    for (int i = 0; i < nfds; i++) {
        if (events[i].data.fd == server_fd) {
            /* New connection */
            int client_fd = accept(server_fd, NULL, NULL);
            
            event.events = EPOLLIN | EPOLLET;  /* Edge-triggered */
            event.data.fd = client_fd;
            epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &event);
        } else {
            /* Data from client */
            handle_client(events[i].data.fd);
        }
    }
}
```

### HTTP Server

```c
void handle_http_request(int client_fd) {
    char buffer[4096];
    ssize_t n = read(client_fd, buffer, sizeof(buffer) - 1);
    buffer[n] = '\0';
    
    /* Parse HTTP request */
    char method[16], path[256], version[16];
    sscanf(buffer, "%s %s %s", method, path, version);
    
    /* Simple response */
    const char *response = 
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "Content-Length: 13\r\n"
        "\r\n"
        "Hello, World!";
    
    write(client_fd, response, strlen(response));
    close(client_fd);
}
```

---

## 17.3 Database Systems

### Embedded Database (SQLite Style)

```c
/* Simple key-value store */
typedef struct {
    char key[256];
    char value[1024];
} Record;

typedef struct {
    FILE *fp;
    int num_records;
} Database;

Database *db_open(const char *filename) {
    Database *db = malloc(sizeof(Database));
    db->fp = fopen(filename, "r+b");
    
    if (!db->fp) {
        db->fp = fopen(filename, "w+b");
    }
    
    fseek(db->fp, 0, SEEK_END);
    db->num_records = ftell(db->fp) / sizeof(Record);
    
    return db;
}

int db_insert(Database *db, const char *key, const char *value) {
    Record rec;
    strncpy(rec.key, key, sizeof(rec.key) - 1);
    strncpy(rec.value, value, sizeof(rec.value) - 1);
    
    fseek(db->fp, 0, SEEK_END);
    fwrite(&rec, sizeof(Record), 1, db->fp);
    fflush(db->fp);
    
    db->num_records++;
    return 0;
}

int db_get(Database *db, const char *key, char *value) {
    Record rec;
    
    fseek(db->fp, 0, SEEK_SET);
    while (fread(&rec, sizeof(Record), 1, db->fp) == 1) {
        if (strcmp(rec.key, key) == 0) {
            strcpy(value, rec.value);
            return 0;
        }
    }
    
    return -1;  /* Not found */
}
```

### B-Tree Implementation

```c
#define ORDER 4

typedef struct BTreeNode {
    int keys[ORDER - 1];
    struct BTreeNode *children[ORDER];
    int num_keys;
    bool is_leaf;
} BTreeNode;

BTreeNode *btree_create_node(bool is_leaf) {
    BTreeNode *node = calloc(1, sizeof(BTreeNode));
    node->is_leaf = is_leaf;
    return node;
}

void btree_insert_nonfull(BTreeNode *node, int key);
void btree_split_child(BTreeNode *parent, int i);

void btree_insert(BTreeNode **root, int key) {
    if ((*root)->num_keys == ORDER - 1) {
        BTreeNode *new_root = btree_create_node(false);
        new_root->children[0] = *root;
        btree_split_child(new_root, 0);
        *root = new_root;
    }
    btree_insert_nonfull(*root, key);
}
```

---

## 17.4 Graphics and Game Programming

### OpenGL with C

```c
#include <GL/gl.h>
#include <GLFW/glfw3.h>

void render(void) {
    glClear(GL_COLOR_BUFFER_BIT);
    
    /* Draw a triangle */
    glBegin(GL_TRIANGLES);
    glColor3f(1.0f, 0.0f, 0.0f);
    glVertex2f(0.0f, 0.5f);
    glColor3f(0.0f, 1.0f, 0.0f);
    glVertex2f(-0.5f, -0.5f);
    glColor3f(0.0f, 0.0f, 1.0f);
    glVertex2f(0.5f, -0.5f);
    glEnd();
}

int main(void) {
    if (!glfwInit()) return -1;
    
    GLFWwindow *window = glfwCreateWindow(640, 480, "OpenGL", NULL, NULL);
    glfwMakeContextCurrent(window);
    
    while (!glfwWindowShouldClose(window)) {
        render();
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    
    glfwTerminate();
    return 0;
}
```

### Game Loop Pattern

```c
typedef struct {
    float x, y;
    float vx, vy;
} Entity;

void update(Entity *entity, float dt) {
    entity->x += entity->vx * dt;
    entity->y += entity->vy * dt;
}

int main(void) {
    Entity player = {0, 0, 0, 0};
    
    uint64_t last_time = get_time_ns();
    
    while (running) {
        uint64_t current_time = get_time_ns();
        float dt = (current_time - last_time) / 1e9f;
        last_time = current_time;
        
        /* Input */
        handle_input(&player);
        
        /* Update */
        update(&player, dt);
        
        /* Render */
        render(&player);
        
        /* Cap framerate */
        sleep_until_next_frame();
    }
    
    return 0;
}
```

---

## 17.5 Scientific Computing

### Matrix Operations (BLAS-like)

```c
/* Matrix-vector multiplication */
void gemv(int m, int n, double alpha, double *A, double *x, 
          double beta, double *y) {
    for (int i = 0; i < m; i++) {
        double sum = 0.0;
        for (int j = 0; j < n; j++) {
            sum += A[i * n + j] * x[j];
        }
        y[i] = alpha * sum + beta * y[i];
    }
}

/* Matrix-matrix multiplication */
void gemm(int m, int n, int k, double alpha, double *A, double *B,
          double beta, double *C) {
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            double sum = 0.0;
            for (int p = 0; p < k; p++) {
                sum += A[i * k + p] * B[p * n + j];
            }
            C[i * n + j] = alpha * sum + beta * C[i * n + j];
        }
    }
}
```

### FFT Implementation

```c
#include <complex.h>
#include <math.h>

void fft(double complex *x, int n) {
    if (n <= 1) return;
    
    /* Divide */
    double complex *even = malloc(n / 2 * sizeof(double complex));
    double complex *odd = malloc(n / 2 * sizeof(double complex));
    
    for (int i = 0; i < n / 2; i++) {
        even[i] = x[i * 2];
        odd[i] = x[i * 2 + 1];
    }
    
    /* Conquer */
    fft(even, n / 2);
    fft(odd, n / 2);
    
    /* Combine */
    for (int k = 0; k < n / 2; k++) {
        double complex t = cexp(-2.0 * I * M_PI * k / n) * odd[k];
        x[k] = even[k] + t;
        x[k + n / 2] = even[k] - t;
    }
    
    free(even);
    free(odd);
}
```

### OpenMP Parallelization

```c
#include <omp.h>

void parallel_sum(double *array, int n, double *result) {
    double sum = 0.0;
    
    #pragma omp parallel for reduction(+:sum)
    for (int i = 0; i < n; i++) {
        sum += array[i];
    }
    
    *result = sum;
}

void matrix_multiply_parallel(double *A, double *B, double *C, int n) {
    #pragma omp parallel for collapse(2)
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            double sum = 0.0;
            for (int k = 0; k < n; k++) {
                sum += A[i * n + k] * B[k * n + j];
            }
            C[i * n + j] = sum;
        }
    }
}
```

---

## 17.6 Compiler and Interpreter Development

### Writing a Lexer

```c
typedef enum {
    TOKEN_NUMBER,
    TOKEN_PLUS,
    TOKEN_MINUS,
    TOKEN_STAR,
    TOKEN_SLASH,
    TOKEN_LPAREN,
    TOKEN_RPAREN,
    TOKEN_EOF
} TokenType;

typedef struct {
    TokenType type;
    int value;
} Token;

typedef struct {
    const char *input;
    int pos;
} Lexer;

void lexer_init(Lexer *lex, const char *input) {
    lex->input = input;
    lex->pos = 0;
}

Token lexer_next_token(Lexer *lex) {
    while (isspace(lex->input[lex->pos])) {
        lex->pos++;
    }
    
    if (lex->input[lex->pos] == '\0') {
        return (Token){TOKEN_EOF, 0};
    }
    
    if (isdigit(lex->input[lex->pos])) {
        int value = 0;
        while (isdigit(lex->input[lex->pos])) {
            value = value * 10 + (lex->input[lex->pos] - '0');
            lex->pos++;
        }
        return (Token){TOKEN_NUMBER, value};
    }
    
    switch (lex->input[lex->pos++]) {
        case '+': return (Token){TOKEN_PLUS, 0};
        case '-': return (Token){TOKEN_MINUS, 0};
        case '*': return (Token){TOKEN_STAR, 0};
        case '/': return (Token){TOKEN_SLASH, 0};
        case '(': return (Token){TOKEN_LPAREN, 0};
        case ')': return (Token){TOKEN_RPAREN, 0};
    }
    
    return (Token){TOKEN_EOF, 0};
}
```

### Recursive Descent Parser

```c
/* Grammar:
   expr   -> term (('+' | '-') term)*
   term   -> factor (('*' | '/') factor)*
   factor -> NUMBER | '(' expr ')'
*/

int parse_expr(Lexer *lex);

int parse_factor(Lexer *lex) {
    Token tok = lexer_next_token(lex);
    
    if (tok.type == TOKEN_NUMBER) {
        return tok.value;
    }
    
    if (tok.type == TOKEN_LPAREN) {
        int result = parse_expr(lex);
        lexer_next_token(lex);  /* Consume ')' */
        return result;
    }
    
    return 0;
}

int parse_term(Lexer *lex) {
    int result = parse_factor(lex);
    
    while (1) {
        Token tok = lexer_peek(lex);
        if (tok.type != TOKEN_STAR && tok.type != TOKEN_SLASH) {
            break;
        }
        lexer_next_token(lex);
        
        int right = parse_factor(lex);
        result = (tok.type == TOKEN_STAR) ? result * right : result / right;
    }
    
    return result;
}

int parse_expr(Lexer *lex) {
    int result = parse_term(lex);
    
    while (1) {
        Token tok = lexer_peek(lex);
        if (tok.type != TOKEN_PLUS && tok.type != TOKEN_MINUS) {
            break;
        }
        lexer_next_token(lex);
        
        int right = parse_term(lex);
        result = (tok.type == TOKEN_PLUS) ? result + right : result - right;
    }
    
    return result;
}
```

### Virtual Machine

```c
typedef enum {
    OP_PUSH,
    OP_ADD,
    OP_SUB,
    OP_MUL,
    OP_DIV,
    OP_HALT
} OpCode;

typedef struct {
    int stack[256];
    int sp;
    uint8_t *code;
    int pc;
} VM;

void vm_init(VM *vm, uint8_t *code) {
    vm->sp = 0;
    vm->pc = 0;
    vm->code = code;
}

void vm_run(VM *vm) {
    while (1) {
        switch (vm->code[vm->pc++]) {
            case OP_PUSH:
                vm->stack[vm->sp++] = vm->code[vm->pc++];
                break;
            case OP_ADD:
                vm->sp--;
                vm->stack[vm->sp - 1] += vm->stack[vm->sp];
                break;
            case OP_SUB:
                vm->sp--;
                vm->stack[vm->sp - 1] -= vm->stack[vm->sp];
                break;
            case OP_MUL:
                vm->sp--;
                vm->stack[vm->sp - 1] *= vm->stack[vm->sp];
                break;
            case OP_DIV:
                vm->sp--;
                vm->stack[vm->sp - 1] /= vm->stack[vm->sp];
                break;
            case OP_HALT:
                return;
        }
    }
}
```

---

## Key Takeaways

**OS Development:**
- Direct hardware control
- No standard library
- Kernel vs userspace
- Interrupt handling critical

**Network Programming:**
- Sockets are fundamental
- Async I/O for scalability
- HTTP is text-based protocol
- Error handling essential

**Database Systems:**
- Simple key-value stores
- B-trees for indexing
- Transaction handling
- ACID properties

**Graphics/Games:**
- Game loop pattern
- Fixed timestep updates
- OpenGL for 3D graphics
- Entity-component systems

**Scientific Computing:**
- Linear algebra operations
- FFT for signal processing
- OpenMP for parallelism
- BLAS/LAPACK libraries

**Compiler Development:**
- Lexer → Parser → Codegen
- AST representation
- Virtual machines
- Bytecode execution

---

## Part 17 Summary

C excels in specialized domains where performance and control matter. This part demonstrated C's versatility across operating systems, networking, databases, graphics, scientific computing, and language implementation.

**Series Progress:**
- ✅ Parts 1-16: Complete mastery (~89,060 words)
- ✅ Part 17: Domain-Specific Applications (~2,100 words)
- **Total: ~91,160 words** across seventeen comprehensive parts

All parts ready for publication!
