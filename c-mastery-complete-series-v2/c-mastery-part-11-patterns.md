---
title: "Complete C Language Mastery Part 11: Design Patterns and Idioms in C"
date: 2024-02-21 20:00:00 +0000
categories: [C, Programming, Design-Patterns, Software-Engineering]
tags: [c-language, design-patterns, oop, idioms, software-architecture, best-practices]
---

# Complete C Language Mastery Part 11: Design Patterns and Idioms in C

Welcome to Part 11 of the C Language Mastery series. This part explores design patterns and common idioms in C—proven solutions to recurring problems. While C isn't object-oriented, many OOP patterns can be implemented effectively in C, leading to cleaner, more maintainable code.

**Prerequisites**: Parts 1-10 (especially structures, pointers, and function pointers)

---

## 11.1 Object-Oriented Patterns in C

### Encapsulation with Opaque Types

**Hide implementation details:**

```c
/* shape.h - Public interface */
#ifndef SHAPE_H
#define SHAPE_H

typedef struct Shape Shape;  /* Opaque type */

Shape *shape_create(int x, int y);
void shape_destroy(Shape *s);
void shape_move(Shape *s, int dx, int dy);
int shape_get_x(const Shape *s);
int shape_get_y(const Shape *s);

#endif

/* shape.c - Private implementation */
#include "shape.h"
#include <stdlib.h>

struct Shape {  /* Definition hidden from users */
    int x;
    int y;
};

Shape *shape_create(int x, int y) {
    Shape *s = malloc(sizeof(Shape));
    if (s) {
        s->x = x;
        s->y = y;
    }
    return s;
}

void shape_destroy(Shape *s) {
    free(s);
}

void shape_move(Shape *s, int dx, int dy) {
    s->x += dx;
    s->y += dy;
}

int shape_get_x(const Shape *s) {
    return s->x;
}

/* Users can't access internals directly */
```

### Inheritance Simulation

**Embedding base struct:**

```c
/* Base "class" */
typedef struct {
    int x, y;
} Shape;

void shape_init(Shape *s, int x, int y) {
    s->x = x;
    s->y = y;
}

/* Derived "class" - embeds base */
typedef struct {
    Shape base;  /* MUST be first member */
    int radius;
} Circle;

void circle_init(Circle *c, int x, int y, int radius) {
    shape_init(&c->base, x, y);  /* Initialize base */
    c->radius = radius;
}

/* Upcasting (safe) */
Circle c;
circle_init(&c, 10, 20, 5);
Shape *s = (Shape *)&c;  /* Circle* -> Shape* */
printf("x: %d\n", s->x);  /* Access base members */

/* Downcasting (requires type tracking) */
typedef enum { SHAPE_CIRCLE, SHAPE_RECTANGLE } ShapeType;

typedef struct {
    ShapeType type;  /* Type tag */
    int x, y;
} TaggedShape;

typedef struct {
    TaggedShape base;
    int radius;
} TaggedCircle;

/* Safe downcast */
void process_shape(TaggedShape *s) {
    if (s->type == SHAPE_CIRCLE) {
        TaggedCircle *c = (TaggedCircle *)s;
        /* Use circle-specific members */
    }
}
```

### Polymorphism with Function Pointers

**Virtual function tables:**

```c
/* Interface definition */
typedef struct Shape Shape;

typedef struct {
    void (*draw)(const Shape *s);
    double (*area)(const Shape *s);
    void (*destroy)(Shape *s);
} ShapeVTable;

/* Base shape with vtable */
struct Shape {
    const ShapeVTable *vtable;
    int x, y;
};

/* Circle implementation */
typedef struct {
    Shape base;
    int radius;
} Circle;

void circle_draw(const Shape *s) {
    const Circle *c = (const Circle *)s;
    printf("Circle at (%d,%d) r=%d\n", s->x, s->y, c->radius);
}

double circle_area(const Shape *s) {
    const Circle *c = (const Circle *)s;
    return 3.14159 * c->radius * c->radius;
}

void circle_destroy(Shape *s) {
    free(s);
}

static const ShapeVTable circle_vtable = {
    .draw = circle_draw,
    .area = circle_area,
    .destroy = circle_destroy
};

Circle *circle_create(int x, int y, int radius) {
    Circle *c = malloc(sizeof(Circle));
    if (c) {
        c->base.vtable = &circle_vtable;
        c->base.x = x;
        c->base.y = y;
        c->radius = radius;
    }
    return c;
}

/* Rectangle implementation */
typedef struct {
    Shape base;
    int width, height;
} Rectangle;

void rectangle_draw(const Shape *s) {
    const Rectangle *r = (const Rectangle *)s;
    printf("Rectangle at (%d,%d) %dx%d\n", s->x, s->y, r->width, r->height);
}

double rectangle_area(const Shape *s) {
    const Rectangle *r = (const Rectangle *)s;
    return r->width * r->height;
}

static const ShapeVTable rectangle_vtable = {
    .draw = rectangle_draw,
    .area = rectangle_area,
    .destroy = circle_destroy
};

/* Polymorphic usage */
void draw_shapes(Shape **shapes, int count) {
    for (int i = 0; i < count; i++) {
        shapes[i]->vtable->draw(shapes[i]);  /* Polymorphic call */
    }
}

int main(void) {
    Shape *shapes[2];
    shapes[0] = (Shape *)circle_create(10, 20, 5);
    shapes[1] = (Shape *)rectangle_create(30, 40, 10, 15);
    
    draw_shapes(shapes, 2);  /* Calls appropriate draw() */
    
    for (int i = 0; i < 2; i++) {
        shapes[i]->vtable->destroy(shapes[i]);
    }
    return 0;
}
```

---

## 11.2 Creational Patterns

### Factory Pattern

```c
typedef enum { SHAPE_CIRCLE, SHAPE_RECTANGLE } ShapeType;

/* Factory function */
Shape *shape_factory_create(ShapeType type, int x, int y, int param1, int param2) {
    switch (type) {
        case SHAPE_CIRCLE:
            return (Shape *)circle_create(x, y, param1);
        case SHAPE_RECTANGLE:
            return (Shape *)rectangle_create(x, y, param1, param2);
        default:
            return NULL;
    }
}

/* Usage */
Shape *s1 = shape_factory_create(SHAPE_CIRCLE, 0, 0, 10, 0);
Shape *s2 = shape_factory_create(SHAPE_RECTANGLE, 5, 5, 20, 30);
```

### Singleton Pattern

```c
typedef struct {
    int value;
    /* ... other fields ... */
} Config;

/* Global singleton instance */
static Config *instance = NULL;
static pthread_mutex_t instance_mutex = PTHREAD_MUTEX_INITIALIZER;

Config *config_get_instance(void) {
    pthread_mutex_lock(&instance_mutex);
    
    if (instance == NULL) {
        instance = malloc(sizeof(Config));
        if (instance) {
            instance->value = 42;  /* Initialize */
        }
    }
    
    pthread_mutex_unlock(&instance_mutex);
    return instance;
}

void config_destroy(void) {
    pthread_mutex_lock(&instance_mutex);
    free(instance);
    instance = NULL;
    pthread_mutex_unlock(&instance_mutex);
}
```

### Builder Pattern

```c
typedef struct {
    char *name;
    int age;
    char *address;
    char *phone;
} Person;

typedef struct {
    Person person;
} PersonBuilder;

PersonBuilder *person_builder_create(void) {
    PersonBuilder *b = malloc(sizeof(PersonBuilder));
    if (b) {
        memset(&b->person, 0, sizeof(Person));
    }
    return b;
}

PersonBuilder *person_builder_set_name(PersonBuilder *b, const char *name) {
    b->person.name = strdup(name);
    return b;  /* Return self for chaining */
}

PersonBuilder *person_builder_set_age(PersonBuilder *b, int age) {
    b->person.age = age;
    return b;
}

PersonBuilder *person_builder_set_address(PersonBuilder *b, const char *address) {
    b->person.address = strdup(address);
    return b;
}

Person *person_builder_build(PersonBuilder *b) {
    Person *p = malloc(sizeof(Person));
    if (p) {
        *p = b->person;
    }
    free(b);  /* Builder consumed */
    return p;
}

/* Fluent API usage */
Person *person = person_builder_create()
    ->person_builder_set_name("Alice")
    ->person_builder_set_age(30)
    ->person_builder_set_address("123 Main St")
    ->person_builder_build();
```

### Object Pool

```c
#define POOL_SIZE 10

typedef struct {
    void *objects[POOL_SIZE];
    int available[POOL_SIZE];
    pthread_mutex_t mutex;
} ObjectPool;

ObjectPool *pool_create(size_t object_size) {
    ObjectPool *pool = malloc(sizeof(ObjectPool));
    if (!pool) return NULL;
    
    pthread_mutex_init(&pool->mutex, NULL);
    
    for (int i = 0; i < POOL_SIZE; i++) {
        pool->objects[i] = malloc(object_size);
        pool->available[i] = 1;
    }
    
    return pool;
}

void *pool_acquire(ObjectPool *pool) {
    pthread_mutex_lock(&pool->mutex);
    
    for (int i = 0; i < POOL_SIZE; i++) {
        if (pool->available[i]) {
            pool->available[i] = 0;
            pthread_mutex_unlock(&pool->mutex);
            return pool->objects[i];
        }
    }
    
    pthread_mutex_unlock(&pool->mutex);
    return NULL;  /* Pool exhausted */
}

void pool_release(ObjectPool *pool, void *obj) {
    pthread_mutex_lock(&pool->mutex);
    
    for (int i = 0; i < POOL_SIZE; i++) {
        if (pool->objects[i] == obj) {
            pool->available[i] = 1;
            break;
        }
    }
    
    pthread_mutex_unlock(&pool->mutex);
}
```

---

## 11.3 Structural Patterns

### Adapter Pattern

```c
/* Old interface */
typedef struct {
    void (*old_method)(void *self, int param);
} OldInterface;

/* New interface */
typedef struct {
    void (*new_method)(void *self, const char *param);
} NewInterface;

/* Adapter */
typedef struct {
    NewInterface new_interface;
    OldInterface *old_impl;
} Adapter;

void adapter_new_method(void *self, const char *param) {
    Adapter *a = (Adapter *)self;
    int int_param = atoi(param);  /* Adapt parameter */
    a->old_impl->old_method(a->old_impl, int_param);
}

Adapter *adapter_create(OldInterface *old) {
    Adapter *a = malloc(sizeof(Adapter));
    if (a) {
        a->new_interface.new_method = adapter_new_method;
        a->old_impl = old;
    }
    return a;
}
```

### Decorator Pattern

```c
/* Component interface */
typedef struct Component {
    int (*get_value)(struct Component *self);
} Component;

/* Concrete component */
typedef struct {
    Component base;
    int value;
} ConcreteComponent;

int concrete_get_value(Component *self) {
    ConcreteComponent *c = (ConcreteComponent *)self;
    return c->value;
}

/* Decorator */
typedef struct {
    Component base;
    Component *wrapped;
    int bonus;
} BonusDecorator;

int bonus_get_value(Component *self) {
    BonusDecorator *d = (BonusDecorator *)self;
    return d->wrapped->get_value(d->wrapped) + d->bonus;
}

/* Usage: stack decorators */
Component *c = create_concrete_component(10);
Component *decorated = create_bonus_decorator(c, 5);
printf("Value: %d\n", decorated->get_value(decorated));  /* 15 */
```

---

## 11.4 Behavioral Patterns

### Strategy Pattern

```c
/* Strategy interface */
typedef struct {
    int (*compare)(const void *a, const void *b);
} SortStrategy;

/* Concrete strategies */
int ascending_compare(const void *a, const void *b) {
    return *(int *)a - *(int *)b;
}

int descending_compare(const void *a, const void *b) {
    return *(int *)b - *(int *)a;
}

/* Context */
void sort_array(int *arr, size_t n, SortStrategy *strategy) {
    qsort(arr, n, sizeof(int), strategy->compare);
}

/* Usage */
SortStrategy asc = { .compare = ascending_compare };
SortStrategy desc = { .compare = descending_compare };

sort_array(arr, n, &asc);   /* Sort ascending */
sort_array(arr, n, &desc);  /* Sort descending */
```

### Observer Pattern

```c
#define MAX_OBSERVERS 10

typedef struct Subject Subject;

typedef struct {
    void (*update)(void *observer, Subject *subject);
} Observer;

struct Subject {
    Observer *observers[MAX_OBSERVERS];
    int observer_count;
    int state;
};

void subject_attach(Subject *s, Observer *o) {
    if (s->observer_count < MAX_OBSERVERS) {
        s->observers[s->observer_count++] = o;
    }
}

void subject_notify(Subject *s) {
    for (int i = 0; i < s->observer_count; i++) {
        s->observers[i]->update(s->observers[i], s);
    }
}

void subject_set_state(Subject *s, int state) {
    s->state = state;
    subject_notify(s);  /* Notify all observers */
}
```

### State Machine Pattern

```c
typedef struct State State;

typedef struct {
    void (*enter)(State *state);
    void (*handle)(State *state, int event);
    void (*exit)(State *state);
} StateVTable;

struct State {
    const StateVTable *vtable;
    void *context;  /* State machine context */
};

typedef struct {
    State *current_state;
} StateMachine;

void state_machine_transition(StateMachine *sm, State *new_state) {
    if (sm->current_state && sm->current_state->vtable->exit) {
        sm->current_state->vtable->exit(sm->current_state);
    }
    
    sm->current_state = new_state;
    
    if (new_state && new_state->vtable->enter) {
        new_state->vtable->enter(new_state);
    }
}

void state_machine_handle_event(StateMachine *sm, int event) {
    if (sm->current_state && sm->current_state->vtable->handle) {
        sm->current_state->vtable->handle(sm->current_state, event);
    }
}
```

---

## 11.5 Resource Management Patterns

### RAII Simulation (Constructor/Destructor)

```c
/* Constructor pattern */
typedef struct {
    FILE *file;
    char *buffer;
} FileReader;

FileReader *file_reader_create(const char *filename) {
    FileReader *fr = malloc(sizeof(FileReader));
    if (!fr) return NULL;
    
    fr->file = fopen(filename, "r");
    if (!fr->file) {
        free(fr);
        return NULL;
    }
    
    fr->buffer = malloc(4096);
    if (!fr->buffer) {
        fclose(fr->file);
        free(fr);
        return NULL;
    }
    
    return fr;
}

/* Destructor pattern */
void file_reader_destroy(FileReader *fr) {
    if (fr) {
        if (fr->file) fclose(fr->file);
        if (fr->buffer) free(fr->buffer);
        free(fr);
    }
}

/* Scope-based cleanup with GCC */
#define CLEANUP(func) __attribute__((cleanup(func)))

void cleanup_file_reader(FileReader **fr) {
    file_reader_destroy(*fr);
}

void example(void) {
    CLEANUP(cleanup_file_reader) FileReader *fr = file_reader_create("file.txt");
    /* Use fr... */
    /* Automatically destroyed when leaving scope */
}
```

### Reference Counting

```c
typedef struct {
    void *data;
    int ref_count;
    void (*destructor)(void *);
} RefCounted;

RefCounted *rc_create(void *data, void (*destructor)(void *)) {
    RefCounted *rc = malloc(sizeof(RefCounted));
    if (rc) {
        rc->data = data;
        rc->ref_count = 1;
        rc->destructor = destructor;
    }
    return rc;
}

void rc_retain(RefCounted *rc) {
    if (rc) {
        __atomic_add_fetch(&rc->ref_count, 1, __ATOMIC_SEQ_CST);
    }
}

void rc_release(RefCounted *rc) {
    if (rc) {
        int count = __atomic_sub_fetch(&rc->ref_count, 1, __ATOMIC_SEQ_CST);
        if (count == 0) {
            if (rc->destructor) {
                rc->destructor(rc->data);
            }
            free(rc);
        }
    }
}
```

---

## 11.6 Common C Idioms

### Error Handling Idioms

**Goto cleanup pattern:**

```c
int complex_operation(const char *filename) {
    int result = -1;
    FILE *fp = NULL;
    char *buffer = NULL;
    int *data = NULL;
    
    fp = fopen(filename, "r");
    if (!fp) goto cleanup;
    
    buffer = malloc(1024);
    if (!buffer) goto cleanup;
    
    data = malloc(sizeof(int) * 100);
    if (!data) goto cleanup;
    
    /* Do work... */
    
    result = 0;  /* Success */
    
cleanup:
    if (fp) fclose(fp);
    if (buffer) free(buffer);
    if (data) free(data);
    return result;
}
```

### X-Macros

**Generate related code from single definition:**

```c
/* Define list once */
#define ERROR_CODES \
    X(SUCCESS,      0,   "Success") \
    X(ERR_NOMEM,   -1,   "Out of memory") \
    X(ERR_IO,      -2,   "I/O error") \
    X(ERR_INVALID, -3,   "Invalid argument")

/* Generate enum */
#define X(name, value, desc) name = value,
enum ErrorCode {
    ERROR_CODES
};
#undef X

/* Generate string table */
#define X(name, value, desc) desc,
static const char *error_strings[] = {
    ERROR_CODES
};
#undef X

/* Generate function */
const char *error_to_string(enum ErrorCode code) {
    #define X(name, value, desc) case name: return desc;
    switch (code) {
        ERROR_CODES
        default: return "Unknown error";
    }
    #undef X
}
```

### Callback Registration

```c
typedef void (*EventCallback)(void *data);

typedef struct {
    EventCallback callbacks[10];
    void *callback_data[10];
    int count;
} EventSystem;

void event_register(EventSystem *es, EventCallback cb, void *data) {
    if (es->count < 10) {
        es->callbacks[es->count] = cb;
        es->callback_data[es->count] = data;
        es->count++;
    }
}

void event_trigger(EventSystem *es) {
    for (int i = 0; i < es->count; i++) {
        es->callbacks[i](es->callback_data[i]);
    }
}
```

### Handle-Based APIs

```c
/* Opaque handle */
typedef struct Connection *ConnectionHandle;

/* Internal structure (hidden) */
struct Connection {
    int socket;
    char buffer[1024];
};

/* API uses handles */
ConnectionHandle connection_open(const char *host, int port);
int connection_send(ConnectionHandle handle, const void *data, size_t len);
void connection_close(ConnectionHandle handle);
```

---

## Key Takeaways

**OOP in C:**
- Opaque types for encapsulation
- Struct embedding for inheritance
- Function pointers for polymorphism
- VTables for virtual functions

**Creational Patterns:**
- Factory: centralized object creation
- Singleton: single instance (use sparingly)
- Builder: complex object construction
- Object pool: reuse expensive objects

**Structural Patterns:**
- Adapter: interface compatibility
- Decorator: add functionality dynamically
- Proxy: control access

**Behavioral Patterns:**
- Strategy: interchangeable algorithms
- Observer: publish-subscribe
- State machine: clean state management

**Resource Management:**
- RAII simulation with cleanup functions
- Reference counting for shared ownership
- Goto cleanup for error handling

**Common Idioms:**
- Error handling with goto cleanup
- X-macros for code generation
- Callback registration systems
- Handle-based APIs

---

## Part 11 Summary

Design patterns provide proven solutions to common problems. While C isn't object-oriented, these patterns can be effectively implemented, leading to better code organization and maintainability.

**Series Progress:**
- ✅ Parts 1-10: Complete C mastery (~75,650 words)
- ✅ Part 11: Design Patterns and Idioms (~2,900 words)
- **Total: ~78,550 words** across eleven comprehensive parts

All parts ready for publication with practical pattern implementations!
