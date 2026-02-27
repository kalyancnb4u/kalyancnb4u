---
title: "Complete C Language Mastery Part 5: C Standard Library"
date: 2024-02-21 14:00:00 +0000
categories: [C, Programming, Systems]
tags: [c-language, standard-library, string-handling, math, time, stdlib, stdint, stdbool]
---

# Complete C Language Mastery Part 5: C Standard Library

Welcome to Part 5 of the C Language Mastery series. This part provides comprehensive coverage of the C Standard Library - the rich collection of functions, types, and macros that ship with every conforming C implementation. Mastering the standard library is essential for productive C programming.

**Prerequisites**: Parts 1-4 (Fundamentals, Memory, Derived Types, Advanced Features)

---

## 5.1 Standard Library Overview

### Standard Headers and Portability

The C Standard Library is divided into headers, each providing specific functionality:

**C89/C90 Headers:**
```c
<assert.h>    /* Diagnostics */
<ctype.h>     /* Character handling */
<errno.h>     /* Error codes */
<float.h>     /* Floating-point characteristics */
<limits.h>    /* Implementation limits */
<locale.h>    /* Localization */
<math.h>      /* Mathematics */
<setjmp.h>    /* Non-local jumps */
<signal.h>    /* Signal handling */
<stdarg.h>    /* Variable arguments */
<stddef.h>    /* Standard definitions */
<stdio.h>     /* Input/output */
<stdlib.h>    /* Utility functions */
<string.h>    /* String handling */
<time.h>      /* Date and time */
```

**C99 Additions:**
```c
<complex.h>   /* Complex arithmetic */
<fenv.h>      /* Floating-point environment */
<inttypes.h>  /* Integer type conversions */
<iso646.h>    /* Alternative operators */
<stdbool.h>   /* Boolean type */
<stdint.h>    /* Integer types */
<tgmath.h>    /* Type-generic math */
```

**C11 Additions:**
```c
<stdalign.h>  /* Alignment */
<stdatomic.h> /* Atomic operations */
<stdnoreturn.h> /* Non-returning functions */
<threads.h>   /* Threading */
<uchar.h>     /* Unicode characters */
```

### Freestanding vs Hosted Implementations

**Hosted Implementation** (normal programs):
- Full standard library available
- Requires operating system
- Entry point: `main()`

**Freestanding Implementation** (embedded systems):
- Minimal library required: `<float.h>`, `<iso646.h>`, `<limits.h>`, `<stdalign.h>`, `<stdarg.h>`, `<stdbool.h>`, `<stddef.h>`, `<stdint.h>`, `<stdnoreturn.h>`
- No OS required
- Entry point: implementation-defined

```c
/* Check if hosted */
#if __STDC_HOSTED__ == 1
    /* Hosted implementation */
    #include <stdio.h>
#else
    /* Freestanding implementation */
    /* No stdio available */
#endif
```

### C Standard Library Categories

1. **Input/Output**: `<stdio.h>`
2. **String/Memory**: `<string.h>`
3. **Character Classification**: `<ctype.h>`
4. **Mathematics**: `<math.h>`, `<complex.h>`, `<tgmath.h>`
5. **Utility**: `<stdlib.h>`
6. **Time/Date**: `<time.h>`
7. **Diagnostics**: `<assert.h>`, `<errno.h>`
8. **Types/Limits**: `<stddef.h>`, `<stdint.h>`, `<limits.h>`, `<float.h>`
9. **Variable Arguments**: `<stdarg.h>`
10. **Concurrency**: `<threads.h>`, `<stdatomic.h>` (C11)

---

## 5.2 String Handling (string.h)

### String Manipulation Functions

```c
#include <string.h>

/* String length */
size_t strlen(const char *s);

char str[] = "Hello";
printf("Length: %zu\n", strlen(str));  /* 5 */

/* String copy */
char *strcpy(char *dest, const char *src);
char *strncpy(char *dest, const char *src, size_t n);

char dest[20];
strcpy(dest, "Hello");           /* ⚠️ No bounds check */
strncpy(dest, "Hello", sizeof(dest));  /* Safer */
dest[sizeof(dest)-1] = '\0';    /* Ensure null termination */

/* String concatenation */
char *strcat(char *dest, const char *src);
char *strncat(char *dest, const char *src, size_t n);

char str[20] = "Hello";
strcat(str, " World");          /* ⚠️ No bounds check */
strncat(str, " World", sizeof(str) - strlen(str) - 1);  /* Safer */

/* String comparison */
int strcmp(const char *s1, const char *s2);
int strncmp(const char *s1, const char *s2, size_t n);

if (strcmp(str1, str2) == 0) {
    printf("Strings are equal\n");
}
/* Returns: <0 if s1 < s2, 0 if equal, >0 if s1 > s2 */

/* String search */
char *strchr(const char *s, int c);      /* First occurrence */
char *strrchr(const char *s, int c);     /* Last occurrence */
char *strstr(const char *haystack, const char *needle);

char *pos = strchr("Hello World", 'W');  /* Returns pointer to 'W' */
char *found = strstr("Hello World", "World");  /* Returns pointer to "World" */

/* String prefix/suffix */
size_t strspn(const char *s, const char *accept);    /* Span of accept chars */
size_t strcspn(const char *s, const char *reject);   /* Span of non-reject chars */
char *strpbrk(const char *s, const char *accept);    /* First accept char */
```

### Memory Functions

```c
/* Memory copy */
void *memcpy(void *dest, const void *src, size_t n);
void *memmove(void *dest, const void *src, size_t n);

int arr1[5] = {1, 2, 3, 4, 5};
int arr2[5];
memcpy(arr2, arr1, sizeof(arr1));  /* Copy entire array */

/* memcpy vs memmove */
char str[] = "Hello";
memcpy(str + 2, str, 3);    /* ❌ Undefined if regions overlap */
memmove(str + 2, str, 3);   /* ✅ OK: handles overlapping regions */

/* Memory set */
void *memset(void *s, int c, size_t n);

char buffer[100];
memset(buffer, 0, sizeof(buffer));      /* Zero out buffer */
memset(buffer, 'A', sizeof(buffer));    /* Fill with 'A' */

/* Memory comparison */
int memcmp(const void *s1, const void *s2, size_t n);

int arr1[5] = {1, 2, 3, 4, 5};
int arr2[5] = {1, 2, 3, 4, 5};
if (memcmp(arr1, arr2, sizeof(arr1)) == 0) {
    printf("Arrays are equal\n");
}

/* Memory search */
void *memchr(const void *s, int c, size_t n);

int arr[] = {1, 2, 3, 4, 5};
int *pos = memchr(arr, 3, sizeof(arr));  /* Find 3 in array */
```

### Safe String Functions (C11 Annex K)

```c
#ifdef __STDC_LIB_EXT1__
/* Bounds-checking interfaces */

errno_t strcpy_s(char *dest, rsize_t destsz, const char *src);
errno_t strcat_s(char *dest, rsize_t destsz, const char *src);
errno_t strncpy_s(char *dest, rsize_t destsz, const char *src, rsize_t count);

char dest[20];
if (strcpy_s(dest, sizeof(dest), "Hello") != 0) {
    fprintf(stderr, "strcpy_s failed\n");
}

/* Benefits:
   - Runtime bounds checking
   - Return error on overflow
   - Null-terminate on truncation
*/

/* Note: Not widely implemented
   - MSVC has them
   - GCC/Clang typically don't
   - Check with __STDC_LIB_EXT1__
*/
#endif
```

### String Function Pitfalls

```c
/* 1. strncpy doesn't always null-terminate */
char dest[5];
strncpy(dest, "Hello", sizeof(dest));  /* ❌ No null terminator! */
dest[sizeof(dest)-1] = '\0';           /* ✅ Must add manually */

/* 2. strncat parameter is different */
char str[20] = "Hello";
strncat(str, " World", sizeof(str));  /* ❌ Wrong! */
strncat(str, " World", sizeof(str) - strlen(str) - 1);  /* ✅ Correct */

/* 3. Comparing strings with == */
char *s1 = "Hello";
char *s2 = "Hello";
if (s1 == s2) { }  /* ❌ Compares pointers, not content */
if (strcmp(s1, s2) == 0) { }  /* ✅ Compares content */

/* 4. Modifying string literals */
char *str = "Hello";
str[0] = 'h';  /* ❌ Undefined behavior */

char str[] = "Hello";
str[0] = 'h';  /* ✅ OK: modifiable array */
```

### String Tokenization

```c
/* strtok - Not thread-safe */
char *strtok(char *str, const char *delim);

char str[] = "one,two,three";
char *token = strtok(str, ",");
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, ",");  /* NULL continues from last position */
}

/* strtok_r - Thread-safe (POSIX) */
char *strtok_r(char *str, const char *delim, char **saveptr);

char str[] = "one,two,three";
char *saveptr;
char *token = strtok_r(str, ",", &saveptr);
while (token != NULL) {
    printf("%s\n", token);
    token = strtok_r(NULL, ",", &saveptr);
}

/* Note: strtok modifies the string (inserts '\0') */
```

---

## 5.3 Character Handling (ctype.h)

### Character Classification

```c
#include <ctype.h>

/* Classification functions (return non-zero if true) */
int isalpha(int c);   /* Letter */
int isdigit(int c);   /* Digit 0-9 */
int isalnum(int c);   /* Alphanumeric */
int isspace(int c);   /* Whitespace */
int isupper(int c);   /* Uppercase letter */
int islower(int c);   /* Lowercase letter */
int isprint(int c);   /* Printable character */
int ispunct(int c);   /* Punctuation */
int isxdigit(int c);  /* Hexadecimal digit */
int iscntrl(int c);   /* Control character */
int isgraph(int c);   /* Graphic character (visible) */

/* Examples */
char ch = 'A';
if (isalpha(ch)) {
    printf("'%c' is a letter\n", ch);
}

if (isdigit('5')) {
    printf("'5' is a digit\n");
}

/* Process string */
const char *str = "Hello123";
for (size_t i = 0; str[i]; i++) {
    if (isalpha(str[i])) {
        printf("%c is alpha\n", str[i]);
    } else if (isdigit(str[i])) {
        printf("%c is digit\n", str[i]);
    }
}
```

### Character Conversion

```c
/* Case conversion */
int toupper(int c);
int tolower(int c);

char ch = 'a';
printf("%c\n", toupper(ch));  /* 'A' */

/* Convert string to uppercase */
void to_upper(char *str) {
    while (*str) {
        *str = toupper((unsigned char)*str);
        str++;
    }
}

char str[] = "hello";
to_upper(str);
printf("%s\n", str);  /* "HELLO" */

/* ⚠️ Cast to unsigned char for safety */
/* Negative char values can cause issues */
char c = -1;
toupper(c);                    /* ❌ Undefined if c < 0 */
toupper((unsigned char)c);     /* ✅ Safe */
```

### Locale Considerations

```c
#include <locale.h>

/* ctype functions are locale-aware */
setlocale(LC_ALL, "en_US.UTF-8");

/* isalpha() behavior depends on locale */
/* In C locale: only A-Z, a-z */
/* In UTF-8 locale: includes accented characters */
```

---

## 5.4 Mathematics (math.h)

### Mathematical Functions

```c
#include <math.h>

/* Basic operations */
double fabs(double x);        /* Absolute value */
double fmod(double x, double y);  /* Floating-point remainder */
double remainder(double x, double y);
double fmax(double x, double y);  /* Maximum */
double fmin(double x, double y);  /* Minimum */

/* Power and roots */
double pow(double x, double y);   /* x^y */
double sqrt(double x);            /* Square root */
double cbrt(double x);            /* Cube root */
double hypot(double x, double y); /* sqrt(x^2 + y^2) */

/* Rounding */
double ceil(double x);     /* Round up */
double floor(double x);    /* Round down */
double trunc(double x);    /* Round toward zero */
double round(double x);    /* Round to nearest */

printf("%f\n", ceil(3.2));   /* 4.0 */
printf("%f\n", floor(3.8));  /* 3.0 */
printf("%f\n", round(3.5));  /* 4.0 */
```

### Trigonometric Functions

```c
/* Basic trig (angles in radians) */
double sin(double x);
double cos(double x);
double tan(double x);

/* Inverse trig */
double asin(double x);
double acos(double x);
double atan(double x);
double atan2(double y, double x);  /* Better for full circle */

/* Hyperbolic */
double sinh(double x);
double cosh(double x);
double tanh(double x);

/* Example: Convert degrees to radians */
#define PI 3.14159265358979323846
#define DEG_TO_RAD(deg) ((deg) * PI / 180.0)

double angle_deg = 45.0;
double angle_rad = DEG_TO_RAD(angle_deg);
printf("sin(45°) = %f\n", sin(angle_rad));
```

### Exponential and Logarithmic

```c
/* Exponential */
double exp(double x);      /* e^x */
double exp2(double x);     /* 2^x */
double expm1(double x);    /* e^x - 1 (accurate for small x) */

/* Logarithmic */
double log(double x);      /* Natural log (ln) */
double log10(double x);    /* Base-10 log */
double log2(double x);     /* Base-2 log */
double log1p(double x);    /* ln(1 + x) (accurate for small x) */

/* Example */
double result = exp(log(5.0));  /* Should be 5.0 */
```

### Floating-Point Environment (fenv.h)

```c
#include <fenv.h>

/* Rounding modes */
fesetround(FE_TONEAREST);   /* Round to nearest (default) */
fesetround(FE_DOWNWARD);    /* Round down */
fesetround(FE_UPWARD);      /* Round up */
fesetround(FE_TOWARDZERO);  /* Truncate */

/* Exception handling */
feclearexcept(FE_ALL_EXCEPT);
double result = sqrt(-1.0);  /* Invalid operation */
if (fetestexcept(FE_INVALID)) {
    printf("Invalid operation\n");
}
```

### Special Math Functions (C99)

```c
/* Classification */
int isnan(double x);       /* Is NaN */
int isinf(double x);       /* Is infinity */
int isfinite(double x);    /* Is finite */
int isnormal(double x);    /* Is normal */

double x = sqrt(-1.0);
if (isnan(x)) {
    printf("Result is NaN\n");
}

/* Special values */
double inf = INFINITY;
double nan = NAN;

/* Comparison with NaN */
if (x != x) {  /* True only for NaN */
    printf("x is NaN\n");
}
```

---

## 5.5 Time and Date (time.h)

### Time Representation

```c
#include <time.h>

/* time_t - Seconds since epoch (usually 1970-01-01) */
time_t now = time(NULL);
printf("Seconds since epoch: %ld\n", (long)now);

/* struct tm - Broken-down time */
struct tm {
    int tm_sec;    /* Seconds (0-60) */
    int tm_min;    /* Minutes (0-59) */
    int tm_hour;   /* Hours (0-23) */
    int tm_mday;   /* Day of month (1-31) */
    int tm_mon;    /* Month (0-11) */
    int tm_year;   /* Years since 1900 */
    int tm_wday;   /* Day of week (0-6, Sunday=0) */
    int tm_yday;   /* Day of year (0-365) */
    int tm_isdst;  /* Daylight saving time flag */
};
```

### Time Manipulation

```c
/* Get current time */
time_t now = time(NULL);

/* Convert to broken-down time */
struct tm *local = localtime(&now);    /* Local time */
struct tm *utc = gmtime(&now);         /* UTC time */

printf("Local: %d-%02d-%02d %02d:%02d:%02d\n",
       local->tm_year + 1900, local->tm_mon + 1, local->tm_mday,
       local->tm_hour, local->tm_min, local->tm_sec);

/* Convert back to time_t */
time_t timestamp = mktime(local);

/* Time difference */
time_t start = time(NULL);
/* Do work... */
time_t end = time(NULL);
double elapsed = difftime(end, start);
printf("Elapsed: %.2f seconds\n", elapsed);
```

### Formatting Time

```c
/* strftime - Format time */
char buffer[100];
struct tm *timeinfo = localtime(&now);

strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", timeinfo);
printf("Formatted: %s\n", buffer);  /* "2024-02-21 14:30:45" */

/* Common format specifiers:
   %Y - Year (4 digits)
   %m - Month (01-12)
   %d - Day (01-31)
   %H - Hour (00-23)
   %M - Minute (00-59)
   %S - Second (00-59)
   %A - Weekday name
   %B - Month name
   %c - Complete date/time
   %x - Date
   %X - Time
*/

strftime(buffer, sizeof(buffer), "%A, %B %d, %Y", timeinfo);
printf("%s\n", buffer);  /* "Wednesday, February 21, 2024" */
```

### High-Resolution Timers

```c
/* C11: timespec_get */
#if __STDC_VERSION__ >= 201112L
struct timespec ts;
timespec_get(&ts, TIME_UTC);
printf("Seconds: %ld, Nanoseconds: %ld\n", ts.tv_sec, ts.tv_nsec);
#endif

/* POSIX: clock_gettime (more precise) */
#ifdef _POSIX_C_SOURCE
struct timespec start, end;
clock_gettime(CLOCK_MONOTONIC, &start);
/* Do work... */
clock_gettime(CLOCK_MONOTONIC, &end);

double elapsed = (end.tv_sec - start.tv_sec) +
                 (end.tv_nsec - start.tv_nsec) / 1e9;
printf("Elapsed: %.9f seconds\n", elapsed);
#endif

/* Clock for CPU time */
clock_t start_clock = clock();
/* Do work... */
clock_t end_clock = clock();
double cpu_time = (double)(end_clock - start_clock) / CLOCKS_PER_SEC;
printf("CPU time: %.3f seconds\n", cpu_time);
```

---

## 5.6 Utility Functions (stdlib.h)

### Memory Allocation (covered in Part 2)

```c
void *malloc(size_t size);
void *calloc(size_t nmemb, size_t size);
void *realloc(void *ptr, size_t size);
void free(void *ptr);
```

### Random Number Generation

```c
#include <stdlib.h>
#include <time.h>

/* Seed the generator */
srand(time(NULL));

/* Generate random number */
int random = rand();  /* 0 to RAND_MAX */

/* Random in range [0, n) */
int random_range = rand() % n;

/* Random in range [min, max] */
int random_between = min + rand() % (max - min + 1);

/* Random float [0.0, 1.0) */
double random_float = (double)rand() / RAND_MAX;

/* ⚠️ rand() is not cryptographically secure */
/* For security, use OS-specific functions */
```

### String Conversion

```c
/* Simple conversion (no error checking) */
int atoi(const char *str);
long atol(const char *str);
long long atoll(const char *str);
double atof(const char *str);

int value = atoi("123");  /* 123 */

/* Safe conversion with error checking */
long strtol(const char *str, char **endptr, int base);
unsigned long strtoul(const char *str, char **endptr, int base);
long long strtoll(const char *str, char **endptr, int base);
double strtod(const char *str, char **endptr);

/* Example */
const char *str = "123abc";
char *endptr;
long value = strtol(str, &endptr, 10);

if (*endptr != '\0') {
    printf("Conversion stopped at: %s\n", endptr);  /* "abc" */
}

/* Check for errors */
errno = 0;
long result = strtol(str, &endptr, 10);
if (errno == ERANGE) {
    printf("Overflow/underflow occurred\n");
}
```

### Searching and Sorting

```c
/* Binary search */
void *bsearch(const void *key, const void *base, size_t nmemb,
              size_t size, int (*compar)(const void *, const void *));

int compare_int(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}

int arr[] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
int key = 5;
int *result = bsearch(&key, arr, 9, sizeof(int), compare_int);
if (result != NULL) {
    printf("Found: %d\n", *result);
}

/* Quick sort */
void qsort(void *base, size_t nmemb, size_t size,
           int (*compar)(const void *, const void *));

int arr[] = {5, 2, 8, 1, 9};
qsort(arr, 5, sizeof(int), compare_int);
/* Now sorted: {1, 2, 5, 8, 9} */
```

### Program Control

```c
/* Exit program */
void exit(int status);    /* Normal termination */
void _Exit(int status);   /* Immediate termination (C99) */
void abort(void);         /* Abnormal termination */

exit(EXIT_SUCCESS);  /* 0 */
exit(EXIT_FAILURE);  /* Non-zero */

/* Register cleanup functions */
int atexit(void (*func)(void));

void cleanup(void) {
    printf("Cleaning up...\n");
}

atexit(cleanup);  /* Called when exit() is called */
```

### Environment Variables

```c
/* Get environment variable */
char *getenv(const char *name);

char *path = getenv("PATH");
if (path != NULL) {
    printf("PATH: %s\n", path);
}

/* Common variables: PATH, HOME, USER, TEMP */
```

---

## 5.7 Variable Arguments (stdarg.h)

### Variadic Function Basics

```c
#include <stdarg.h>

/* Variadic function components:
   - va_list: Type to hold argument list
   - va_start: Initialize argument list
   - va_arg: Get next argument
   - va_end: Cleanup argument list
*/

/* Simple example: Sum variable number of integers */
int sum(int count, ...) {
    va_list args;
    va_start(args, count);  /* Initialize after last fixed parameter */
    
    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);  /* Get next int */
    }
    
    va_end(args);  /* Cleanup */
    return total;
}

/* Usage */
int result = sum(5, 10, 20, 30, 40, 50);  /* 150 */
```

### Implementing Variadic Functions

```c
/* printf-style function */
void my_printf(const char *format, ...) {
    va_list args;
    va_start(args, format);
    
    vprintf(format, args);  /* Pass va_list to v* function */
    
    va_end(args);
}

/* Custom formatted output */
void log_message(const char *level, const char *format, ...) {
    printf("[%s] ", level);
    
    va_list args;
    va_start(args, format);
    vprintf(format, args);
    va_end(args);
    
    printf("\n");
}

/* Usage */
log_message("INFO", "User %s logged in from %s", "alice", "192.168.1.1");
log_message("ERROR", "Failed to open file: %s", "data.txt");
```

### Type Safety Considerations

```c
/* ⚠️ Type safety issues with variadic functions */

/* No type checking - caller must match types */
int bad_sum(int count, ...) {
    va_list args;
    va_start(args, count);
    
    int total = 0;
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);  /* Assumes int */
    }
    
    va_end(args);
    return total;
}

/* ❌ Undefined behavior */
bad_sum(2, 1.5, 2.5);  /* Passed double, expected int */

/* ✅ Type-safe alternative: Use format string */
void process(const char *format, ...) {
    va_list args;
    va_start(args, format);
    
    for (const char *p = format; *p; p++) {
        if (*p == 'i') {
            int value = va_arg(args, int);
            printf("int: %d\n", value);
        } else if (*p == 'd') {
            double value = va_arg(args, double);
            printf("double: %f\n", value);
        }
    }
    
    va_end(args);
}

/* Usage */
process("iid", 42, 100, 3.14);
```

### printf-Style Format Functions

```c
/* Variadic wrapper for vprintf */
void debug_print(const char *file, int line, const char *format, ...) {
    fprintf(stderr, "[%s:%d] ", file, line);
    
    va_list args;
    va_start(args, format);
    vfprintf(stderr, format, args);
    va_end(args);
    
    fprintf(stderr, "\n");
}

/* Macro for convenient use */
#define DEBUG(fmt, ...) debug_print(__FILE__, __LINE__, fmt, ##__VA_ARGS__)

/* Usage */
DEBUG("Value: %d", 42);
/* Output: [file.c:10] Value: 42 */

/* String formatting */
char *format_string(const char *format, ...) {
    va_list args;
    va_start(args, format);
    
    /* Determine size needed */
    va_list args_copy;
    va_copy(args_copy, args);
    int size = vsnprintf(NULL, 0, format, args_copy) + 1;
    va_end(args_copy);
    
    /* Allocate and format */
    char *result = malloc(size);
    if (result != NULL) {
        vsnprintf(result, size, format, args);
    }
    
    va_end(args);
    return result;
}

char *msg = format_string("Hello %s, you have %d messages", "Alice", 5);
printf("%s\n", msg);
free(msg);
```

---

## 5.8 Type Generic Macros (tgmath.h) - C99

### Generic Math Functions

```c
#include <tgmath.h>

/* tgmath.h provides type-generic macros */
/* Automatically selects float/double/long double version */

float f = 2.0f;
double d = 2.0;
long double ld = 2.0L;

/* These all work - macro selects correct function */
sqrt(f);    /* Calls sqrtf() */
sqrt(d);    /* Calls sqrt() */
sqrt(ld);   /* Calls sqrtl() */

/* Type-generic functions available:
   sqrt, pow, sin, cos, tan, asin, acos, atan, atan2,
   sinh, cosh, tanh, exp, log, log10, fabs, ceil, floor,
   and many more...
*/

/* Example: Works with any floating-point type */
#define SQUARE(x) ((x) * (x))
#define DISTANCE(x1, y1, x2, y2) \
    sqrt(SQUARE((x2)-(x1)) + SQUARE((y2)-(y1)))

float distance_f = DISTANCE(0.0f, 0.0f, 3.0f, 4.0f);  /* sqrtf */
double distance_d = DISTANCE(0.0, 0.0, 3.0, 4.0);     /* sqrt */
```

### Implementation Using _Generic (C11)

```c
/* C11 _Generic for type-generic programming */
#define abs_value(x) _Generic((x), \
    int: abs(x), \
    long: labs(x), \
    long long: llabs(x), \
    float: fabsf(x), \
    double: fabs(x), \
    long double: fabsl(x) \
)

int i = -42;
double d = -3.14;

printf("%d\n", abs_value(i));   /* Calls abs() */
printf("%f\n", abs_value(d));   /* Calls fabs() */

/* Custom type-generic macro */
#define max(a, b) _Generic((a), \
    int: max_int, \
    double: max_double \
)(a, b)

int max_int(int a, int b) { return a > b ? a : b; }
double max_double(double a, double b) { return a > b ? a : b; }

int mi = max(5, 3);        /* Calls max_int */
double md = max(5.0, 3.0); /* Calls max_double */
```

---

## 5.9 Boolean Type (stdbool.h) - C99

### bool, true, false

```c
#include <stdbool.h>

/* Defines:
   - bool: Boolean type (actually _Bool)
   - true: Boolean true (1)
   - false: Boolean false (0)
*/

bool is_valid = true;
bool is_error = false;

/* Use in functions */
bool is_even(int n) {
    return n % 2 == 0;
}

if (is_even(42)) {
    printf("42 is even\n");
}

/* Conditional expressions */
bool result = (x > 0) && (y < 100);

/* Boolean in structures */
struct Flags {
    bool is_active;
    bool is_visible;
    bool is_enabled;
};
```

### Boolean Semantics

```c
/* _Bool is an integer type (0 or 1) */
_Bool b1 = 42;   /* Stores 1 (any non-zero becomes 1) */
_Bool b2 = 0;    /* Stores 0 */

printf("%d %d\n", b1, b2);  /* 1 0 */

/* bool is typedef for _Bool */
bool b3 = 42;    /* Same behavior */

/* Comparison */
if (b1 == true) {
    printf("b1 is true\n");
}

/* Use in expressions */
int count = 0;
count += is_valid;  /* Adds 1 if true, 0 if false */

/* Array of booleans */
bool flags[10] = {false};  /* All false */
flags[0] = true;
flags[1] = true;
```

---

## 5.10 Integer Types (stdint.h) - C99

### Fixed-Width Integer Types

```c
#include <stdint.h>

/* Exact-width types (may not be available on all platforms) */
int8_t   i8;    /* Exactly 8 bits, signed */
uint8_t  u8;    /* Exactly 8 bits, unsigned */
int16_t  i16;   /* Exactly 16 bits, signed */
uint16_t u16;   /* Exactly 16 bits, unsigned */
int32_t  i32;   /* Exactly 32 bits, signed */
uint32_t u32;   /* Exactly 32 bits, unsigned */
int64_t  i64;   /* Exactly 64 bits, signed */
uint64_t u64;   /* Exactly 64 bits, unsigned */

/* Example */
uint32_t file_signature = 0x12345678;
int16_t temperature = -40;
```

### Minimum-Width Integer Types

```c
/* At least N bits (guaranteed available) */
int_least8_t   il8;    /* At least 8 bits */
uint_least8_t  ul8;
int_least16_t  il16;   /* At least 16 bits */
uint_least16_t ul16;
int_least32_t  il32;   /* At least 32 bits */
uint_least32_t ul32;
int_least64_t  il64;   /* At least 64 bits */
uint_least64_t ul64;

/* Use when exact size not critical */
int_least16_t count;  /* Guaranteed at least 16 bits */
```

### Fastest Integer Types

```c
/* Fastest type with at least N bits */
int_fast8_t   if8;
uint_fast8_t  uf8;
int_fast16_t  if16;
uint_fast16_t uf16;
int_fast32_t  if32;
uint_fast32_t uf32;
int_fast64_t  if64;
uint_fast64_t uf64;

/* Use for performance-critical code */
void process_array(int_fast32_t *arr, size_t size) {
    for (size_t i = 0; i < size; i++) {
        int_fast32_t value = arr[i];
        /* Process value... */
    }
}
```

### Pointer-Sized Integers

```c
/* Integer types that can hold a pointer */
intptr_t  iptr;   /* Signed */
uintptr_t uptr;   /* Unsigned */

/* Use for pointer arithmetic or conversions */
int *ptr = malloc(sizeof(int));
uintptr_t addr = (uintptr_t)ptr;
printf("Address: 0x%lx\n", (unsigned long)addr);

/* Convert back */
int *ptr2 = (int*)addr;

/* Maximum-width integers */
intmax_t  imax;   /* Largest signed integer type */
uintmax_t umax;   /* Largest unsigned integer type */

/* Use for maximum range */
intmax_t factorial(int n) {
    intmax_t result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}
```

### Integer Limits Macros

```c
/* Limits for exact-width types */
INT8_MIN    /* -128 */
INT8_MAX    /* 127 */
UINT8_MAX   /* 255 */
INT16_MIN   /* -32768 */
INT16_MAX   /* 32767 */
UINT16_MAX  /* 65535 */
INT32_MIN   /* -2147483648 */
INT32_MAX   /* 2147483647 */
UINT32_MAX  /* 4294967295 */
INT64_MIN   /* -9223372036854775808 */
INT64_MAX   /* 9223372036854775807 */
UINT64_MAX  /* 18446744073709551615 */

/* Check overflow */
if (value > INT32_MAX / 2) {
    printf("Value too large, would overflow\n");
}

/* Format macros for printf */
#include <inttypes.h>

uint32_t value = 0x12345678;
printf("Value: %" PRIu32 "\n", value);
printf("Hex: %" PRIx32 "\n", value);

int64_t large = 1234567890123LL;
printf("Large: %" PRId64 "\n", large);

/* Scan macros for scanf */
uint32_t input;
scanf("%" SCNu32, &input);
```

---

## Part 5 Complete Summary

**Part 5: C Standard Library** provided comprehensive coverage of:

### 5.1 Standard Library Overview
- Standard headers organization (C89, C99, C11)
- Freestanding vs hosted implementations
- Library categories and functionality

### 5.2 String Handling (string.h)
- String manipulation: strlen, strcpy, strcat, strcmp
- Memory functions: memcpy, memmove, memset, memcmp
- Safe string functions (C11 Annex K)
- String function pitfalls (strncpy, strncat)
- String tokenization: strtok, strtok_r

### 5.3 Character Handling (ctype.h)
- Classification functions: isalpha, isdigit, isalnum, isspace
- Conversion functions: toupper, tolower
- Locale considerations
- Proper casting to unsigned char

### 5.4 Mathematics (math.h)
- Basic operations: fabs, pow, sqrt, fmod
- Trigonometric functions: sin, cos, tan, asin, acos, atan
- Exponential/logarithmic: exp, log, log10
- Rounding: ceil, floor, round, trunc
- Floating-point environment (fenv.h)
- Special functions: isnan, isinf, isfinite

### 5.5 Time and Date (time.h)
- Time representation: time_t, struct tm
- Time manipulation: localtime, gmtime, mktime
- Formatting: strftime
- High-resolution timers: timespec_get, clock_gettime
- CPU time measurement: clock

### 5.6 Utility Functions (stdlib.h)
- Memory allocation: malloc, calloc, realloc, free
- Random numbers: srand, rand
- String conversion: atoi, strtol, strtod
- Searching/sorting: bsearch, qsort
- Program control: exit, abort, atexit
- Environment: getenv

### 5.7 Variable Arguments (stdarg.h)
- Variadic function basics: va_list, va_start, va_arg, va_end
- Implementing printf-style functions
- Type safety considerations
- Format string patterns

### 5.8 Type Generic Macros (tgmath.h)
- Generic math functions (C99)
- Type-generic programming
- _Generic keyword (C11)
- Custom type-generic macros

### 5.9 Boolean Type (stdbool.h)
- bool type definition (C99)
- true and false constants
- Boolean semantics
- Usage in functions and structures

### 5.10 Integer Types (stdint.h)
- Fixed-width types: int8_t, uint32_t, int64_t
- Minimum-width types: int_least16_t
- Fastest types: int_fast32_t
- Pointer-sized: intptr_t, uintptr_t
- Maximum-width: intmax_t, uintmax_t
- Limit macros and format specifiers

**Key Themes:**
- Portable programming with standard library
- Type safety and proper usage
- Performance considerations
- Error handling patterns
- Platform-independent code

**Best Practices:**
- Use fixed-width types for binary formats
- Check return values
- Initialize all variables
- Use safe string functions when available
- Cast to unsigned char for ctype functions
- Use stdint.h for portable integer code
- Prefer type-safe alternatives when possible

**Series Progress:**
- ✅ Part 1: C Fundamentals & Language Core (30,438 words)
- ✅ Part 2: Memory Management & Pointers (13,312 words)
- ✅ Part 3: Derived Types & Data Structures (6,247 words)
- ✅ Part 4: Advanced C Features (5,361 words)
- ✅ Part 5: C Standard Library (5,200+ words)
- **Total: ~60,500 words** across five comprehensive parts

All parts ready for publication on your Jekyll blog with the Chirpy theme!

**What's Next:**
According to the generated prompt, Part 6 would cover Concurrency & Multithreading, Part 7 would cover Best Practices & Design Patterns, and subsequent parts would dive into specialized topics like embedded systems, networking, and more.
