---
title: "Complete C Language Mastery Part 13: Testing and Debugging"
date: 2024-02-21 22:00:00 +0000
categories: [C, Programming, Testing, Debugging]
tags: [c-language, unit-testing, debugging, gdb, valgrind, tdd, test-frameworks]
---

# Complete C Language Mastery Part 13: Testing and Debugging

Welcome to Part 13 of the C Language Mastery series. This part explores testing methodologies and debugging techniques for C programs. Writing tests and debugging effectively are essential skills for professional C development.

**Prerequisites**: Parts 1-12 (especially build systems and security)

---

## 13.1 Unit Testing

### Testing Philosophy for C

**Why test?**
- Catch bugs early (cheaper to fix)
- Document expected behavior
- Enable refactoring with confidence
- Prevent regressions
- Improve code design (testable code is better code)

**Challenges in C:**
- No built-in testing framework
- Manual memory management complicates cleanup
- Harder to mock/stub than higher-level languages
- Side effects common (file I/O, global state)

### Unit Testing Frameworks

**Check (most popular):**

```c
#include <check.h>

START_TEST(test_addition)
{
    int result = add(2, 3);
    ck_assert_int_eq(result, 5);
}
END_TEST

START_TEST(test_division_by_zero)
{
    int result = divide(10, 0);
    ck_assert_int_eq(result, -1);  // Error value
}
END_TEST

Suite *math_suite(void)
{
    Suite *s = suite_create("Math");
    TCase *tc_core = tcase_create("Core");
    
    tcase_add_test(tc_core, test_addition);
    tcase_add_test(tc_core, test_division_by_zero);
    suite_add_tcase(s, tc_core);
    
    return s;
}

int main(void)
{
    int number_failed;
    Suite *s = math_suite();
    SRunner *sr = srunner_create(s);
    
    srunner_run_all(sr, CK_NORMAL);
    number_failed = srunner_ntests_failed(sr);
    srunner_free(sr);
    
    return (number_failed == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

**Unity (embedded-friendly):**

```c
#include "unity.h"

void setUp(void) {
    // Runs before each test
}

void tearDown(void) {
    // Runs after each test
}

void test_addition(void) {
    TEST_ASSERT_EQUAL_INT(5, add(2, 3));
}

void test_string_copy(void) {
    char dest[10];
    safe_strcpy(dest, "Hello", sizeof(dest));
    TEST_ASSERT_EQUAL_STRING("Hello", dest);
}

int main(void) {
    UNITY_BEGIN();
    RUN_TEST(test_addition);
    RUN_TEST(test_string_copy);
    return UNITY_END();
}
```

**CMocka (for mocking):**

```c
#include <stdarg.h>
#include <stddef.h>
#include <setjmp.h>
#include <cmocka.h>

// Function to test
int database_query(Database *db, const char *query);

// Mock function
int __wrap_database_query(Database *db, const char *query) {
    check_expected(query);
    return mock_type(int);
}

static void test_user_login(void **state) {
    expect_string(__wrap_database_query, query, "SELECT * FROM users");
    will_return(__wrap_database_query, 1);  // Success
    
    int result = login("user", "pass");
    assert_int_equal(result, 0);
}

int main(void) {
    const struct CMUnitTest tests[] = {
        cmocka_unit_test(test_user_login),
    };
    return cmocka_run_group_tests(tests, NULL, NULL);
}
```

### Writing Testable C Code

**Separate business logic from I/O:**

```c
/* ❌ Hard to test - mixed logic and I/O */
void process_file(const char *filename) {
    FILE *fp = fopen(filename, "r");
    // Process file...
    fclose(fp);
}

/* ✅ Testable - separate concerns */
int process_data(const char *data, size_t len, char *output);

void process_file(const char *filename) {
    char *data = read_file(filename);
    char output[1024];
    process_data(data, strlen(data), output);
    write_file("output.txt", output);
    free(data);
}

/* Now can test process_data() without files */
void test_process_data(void) {
    char input[] = "test input";
    char output[1024];
    int result = process_data(input, strlen(input), output);
    assert_equal(result, 0);
    assert_string_equal(output, "expected output");
}
```

**Dependency injection:**

```c
/* ❌ Hard to test - global dependency */
extern Database *global_db;

void save_user(User *user) {
    database_insert(global_db, user);
}

/* ✅ Testable - inject dependency */
void save_user(Database *db, User *user) {
    database_insert(db, user);
}

/* In tests, pass a mock database */
void test_save_user(void) {
    MockDatabase mock_db;
    User user = {.name = "Alice", .age = 30};
    save_user((Database *)&mock_db, &user);
    assert(mock_db.insert_called);
}
```

### Test Fixtures and Setup/Teardown

```c
/* Check framework example */
static int *data;

void setup(void) {
    data = malloc(sizeof(int) * 100);
    for (int i = 0; i < 100; i++) {
        data[i] = i;
    }
}

void teardown(void) {
    free(data);
}

START_TEST(test_array_sum)
{
    int sum = array_sum(data, 100);
    ck_assert_int_eq(sum, 4950);
}
END_TEST

Suite *array_suite(void) {
    Suite *s = suite_create("Array");
    TCase *tc = tcase_create("Core");
    
    tcase_add_checked_fixture(tc, setup, teardown);
    tcase_add_test(tc, test_array_sum);
    
    suite_add_tcase(s, tc);
    return s;
}
```

### Test Coverage

**Compile with coverage:**

```bash
gcc -fprofile-arcs -ftest-coverage -o test test.c
./test
gcov test.c
```

**Generate HTML report:**

```bash
lcov --capture --directory . --output-file coverage.info
genhtml coverage.info --output-directory coverage
```

**Aim for high coverage:**
- 80%+ line coverage (good)
- 90%+ line coverage (excellent)
- 100% coverage doesn't guarantee no bugs
- Focus on covering critical paths

### Test-Driven Development (TDD)

**Red-Green-Refactor cycle:**

```c
/* 1. RED - Write failing test */
void test_validate_email(void) {
    assert_true(validate_email("user@example.com"));
    assert_false(validate_email("invalid"));
}

/* 2. GREEN - Write minimal code to pass */
bool validate_email(const char *email) {
    return strchr(email, '@') != NULL;
}

/* 3. REFACTOR - Improve implementation */
bool validate_email(const char *email) {
    if (!email) return false;
    const char *at = strchr(email, '@');
    if (!at || at == email) return false;
    if (!strchr(at, '.')) return false;
    return true;
}
```

---

## 13.2 Integration and System Testing

### Integration Testing

**Test components together:**

```c
/* Integration test: database + business logic */
void test_user_registration_integration(void) {
    Database *db = database_create(":memory:");  // SQLite in-memory
    
    // Register user
    User user = {.name = "Alice", .email = "alice@example.com"};
    int result = register_user(db, &user);
    assert_equal(result, 0);
    
    // Verify user was saved
    User *retrieved = get_user_by_email(db, "alice@example.com");
    assert_not_null(retrieved);
    assert_string_equal(retrieved->name, "Alice");
    
    user_free(retrieved);
    database_destroy(db);
}
```

### System Testing

**Test entire program:**

```bash
#!/bin/bash
# system_test.sh

# Setup
./program --init-db test.db

# Test case 1: Add user
output=$(./program --add-user "Alice" "alice@example.com")
if [[ "$output" != *"User added successfully"* ]]; then
    echo "FAIL: Add user"
    exit 1
fi

# Test case 2: List users
output=$(./program --list-users)
if [[ "$output" != *"Alice"* ]]; then
    echo "FAIL: List users"
    exit 1
fi

echo "All tests passed"
```

### Automated Testing

**CMake with CTest:**

```cmake
enable_testing()

add_executable(test_math tests/test_math.c)
target_link_libraries(test_math mylib check)
add_test(NAME MathTests COMMAND test_math)

add_executable(test_string tests/test_string.c)
target_link_libraries(test_string mylib check)
add_test(NAME StringTests COMMAND test_string)

# Run with: ctest --output-on-failure
```

---

## 13.3 Debugging Techniques

### Printf Debugging

**Strategic placement:**

```c
void complex_function(int *arr, size_t n) {
    printf("DEBUG: Entering complex_function, n=%zu\n", n);
    
    for (size_t i = 0; i < n; i++) {
        printf("DEBUG: i=%zu, arr[i]=%d\n", i, arr[i]);
        
        if (arr[i] < 0) {
            printf("DEBUG: Found negative value at i=%zu\n", i);
            return;
        }
    }
    
    printf("DEBUG: Exiting complex_function normally\n");
}
```

**Debug macros:**

```c
#ifdef DEBUG
    #define DEBUG_PRINT(fmt, ...) \
        fprintf(stderr, "DEBUG %s:%d: " fmt "\n", \
                __FILE__, __LINE__, ##__VA_ARGS__)
#else
    #define DEBUG_PRINT(fmt, ...) do {} while(0)
#endif

// Usage
DEBUG_PRINT("Value: %d", x);  // Only prints if DEBUG defined
```

### GDB Basics

**Compile with debug symbols:**

```bash
gcc -g program.c -o program
gdb ./program
```

**Essential GDB commands:**

```gdb
(gdb) break main          # Set breakpoint at main
(gdb) break file.c:42     # Set breakpoint at line
(gdb) run arg1 arg2       # Run program with arguments
(gdb) next                # Step over (n)
(gdb) step                # Step into (s)
(gdb) continue            # Continue execution (c)
(gdb) print variable      # Print variable value (p)
(gdb) print *pointer      # Dereference pointer
(gdb) backtrace           # Show call stack (bt)
(gdb) frame 2             # Switch to stack frame 2
(gdb) info locals         # Show local variables
(gdb) info args           # Show function arguments
(gdb) quit                # Exit GDB
```

### Breakpoints and Watchpoints

```gdb
# Conditional breakpoint
(gdb) break main if argc > 2

# Watchpoint (break when variable changes)
(gdb) watch my_variable
(gdb) watch *0x12345678  # Watch memory address

# Break on function call
(gdb) break malloc

# Temporary breakpoint (delete after hit)
(gdb) tbreak function_name

# List breakpoints
(gdb) info breakpoints

# Delete breakpoint
(gdb) delete 1
```

### Examining Memory

```gdb
# Examine memory
(gdb) x/10i $pc          # 10 instructions at program counter
(gdb) x/10x 0x12345678   # 10 hex words at address
(gdb) x/10s array        # 10 strings
(gdb) x/10c buffer       # 10 chars

# Examine array
(gdb) print array@10     # Print first 10 elements

# Examine structure
(gdb) print *my_struct
(gdb) print my_struct->field
```

### Core Dumps

**Enable core dumps:**

```bash
ulimit -c unlimited  # Allow core dumps
./program
# Program crashes
gdb ./program core   # Debug with core dump
```

**Analyze core dump:**

```gdb
(gdb) backtrace       # Where did it crash?
(gdb) frame 0         # Go to crash frame
(gdb) print variable  # Examine variables
(gdb) info registers  # Check registers
```

---

## 13.4 Advanced Debugging

### Reverse Debugging with rr

```bash
# Record execution
rr record ./program

# Replay with reverse debugging
rr replay

(rr) continue             # Run forward
(rr) reverse-continue     # Run backward!
(rr) reverse-step         # Step backward
(rr) reverse-next         # Step backward over function
```

**Find when variable changed:**

```gdb
(rr) watch variable
(rr) reverse-continue  # Go back to when it changed
```

### Conditional Breakpoints

```gdb
# Break when condition is true
(gdb) break function if x > 100

# Break on Nth hit
(gdb) break function
(gdb) ignore 1 99  # Ignore first 99 hits of breakpoint 1

# Commands on breakpoint hit
(gdb) break function
(gdb) commands 1
> print x
> continue
> end
```

### Scripting GDB

**GDB script (.gdbinit):**

```python
# .gdbinit
define print_array
    set $i = 0
    while $i < $arg1
        print $arg0[$i]
        set $i = $i + 1
    end
end

# Usage: print_array my_array 10
```

**Python in GDB:**

```python
# gdb_script.py
import gdb

class PrintSizeCommand(gdb.Command):
    def __init__(self):
        super().__init__("print-size", gdb.COMMAND_USER)
    
    def invoke(self, arg, from_tty):
        val = gdb.parse_and_eval(arg)
        print(f"Size of {arg}: {val.type.sizeof} bytes")

PrintSizeCommand()
```

```gdb
(gdb) source gdb_script.py
(gdb) print-size my_struct
```

### Debugging Multithreaded Programs

```gdb
# Show all threads
(gdb) info threads

# Switch to thread
(gdb) thread 2

# Apply command to all threads
(gdb) thread apply all backtrace

# Break in specific thread
(gdb) break function thread 3

# Scheduler locking
(gdb) set scheduler-locking on   # Only current thread runs
(gdb) set scheduler-locking off  # All threads run
```

---

## 13.5 Debugging Tools

### Valgrind Suite

**Memcheck (memory errors):**

```bash
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         ./program
```

**Output:**

```
==12345== Invalid write of size 4
==12345==    at 0x400530: main (program.c:10)
==12345==  Address 0x520004c is 0 bytes after a block of size 40 alloc'd

==12345== 100 bytes in 1 blocks are definitely lost
==12345==    at 0x400530: malloc (vg_replace_malloc.c:380)
==12345==    by 0x400540: main (program.c:5)
```

**Cachegrind (cache profiling):**

```bash
valgrind --tool=cachegrind ./program
cg_annotate cachegrind.out.<pid>
```

**Callgrind (call graph):**

```bash
valgrind --tool=callgrind ./program
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>  # GUI
```

### Sanitizers

**AddressSanitizer:**

```bash
gcc -fsanitize=address -g program.c
./program
# Detects: buffer overflows, use-after-free, double-free
```

**ThreadSanitizer:**

```bash
gcc -fsanitize=thread -g program.c
./program
# Detects: data races
```

**UndefinedBehaviorSanitizer:**

```bash
gcc -fsanitize=undefined -g program.c
./program
# Detects: integer overflow, null dereference, etc.
```

### strace and ltrace

**System call tracing:**

```bash
strace ./program
strace -e open,read,write ./program  # Filter syscalls
strace -c ./program                   # Summary statistics
```

**Library call tracing:**

```bash
ltrace ./program
ltrace -e malloc+free ./program  # Filter functions
```

### Visual Debuggers

**VS Code (launch.json):**

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/program",
            "args": [],
            "cwd": "${workspaceFolder}",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "text": "-enable-pretty-printing"
                }
            ]
        }
    ]
}
```

---

## Best Practices

**Testing:**
1. Write tests for all public interfaces
2. Test edge cases and error conditions
3. Use fixtures for complex setup
4. Aim for 80%+ code coverage
5. Run tests in CI/CD
6. Practice TDD for critical code
7. Mock external dependencies

**Debugging:**
1. Compile with `-g` during development
2. Use sanitizers (ASan, TSan, UBSan)
3. Enable compiler warnings (`-Wall -Wextra`)
4. Use Valgrind for memory issues
5. Learn GDB/LLDB thoroughly
6. Write debug macros for conditional logging
7. Reproduce bugs in tests before fixing

---

## Key Takeaways

**Unit Testing:**
- Frameworks: Check (popular), Unity (embedded), CMocka (mocking)
- Write testable code (separate logic from I/O)
- Use fixtures for setup/teardown
- Track coverage (aim for 80%+)
- Practice TDD for critical features

**Debugging Techniques:**
- Printf debugging: quick and simple
- GDB: powerful, essential to learn
- Breakpoints, watchpoints, conditional breaks
- Core dumps: post-mortem analysis
- Reverse debugging with rr

**Debugging Tools:**
- Valgrind: memory errors and leaks
- Sanitizers: fast runtime checking
- strace/ltrace: system/library call tracing
- Visual debuggers: VS Code, CLion

---

## Part 13 Summary

Testing and debugging are essential skills for professional C development. This part covered modern testing frameworks, comprehensive debugging techniques, and powerful debugging tools.

**Series Progress:**
- ✅ Parts 1-12: Complete C mastery (~80,565 words)
- ✅ Part 13: Testing and Debugging (~2,400 words)
- **Total: ~82,965 words** across thirteen comprehensive parts

All parts ready for publication!
