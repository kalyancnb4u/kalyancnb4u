---
title: "Complete C Language Mastery Part 4: Advanced C Features"
date: 2024-02-21 13:00:00 +0000
categories: [C, Programming, Systems]
tags: [c-language, file-io, error-handling, compilation, linking, multi-file-projects, build-systems]
---

# Complete C Language Mastery Part 4: Advanced C Features

Welcome to Part 4 of the C Language Mastery series. This part explores advanced C features essential for building real-world applications: file I/O, error handling, command-line processing, compilation/linking, and multi-file project organization.

**Prerequisites**: Parts 1-3 (Fundamentals, Memory & Pointers, Derived Types)

---

## 4.1 File I/O

### Concept Introduction

File I/O (Input/Output) allows programs to read from and write to files on disk, enabling data persistence, logging, configuration, and communication. C provides both high-level buffered I/O through the standard library (stdio.h) and low-level system calls for direct control.

### Standard I/O Library (stdio.h)

```c
#include <stdio.h>

/* Standard streams - automatically opened */
FILE *stdin;   /* Standard input (keyboard) */
FILE *stdout;  /* Standard output (screen) */
FILE *stderr;  /* Standard error (screen, unbuffered) */

/* FILE is an opaque type representing a file stream */
```

### Opening and Closing Files

```c
/* fopen - Open file */
FILE *fopen(const char *filename, const char *mode);

/* Modes:
   "r"   - Read (file must exist)
   "w"   - Write (truncates existing file)
   "a"   - Append (creates if doesn't exist)
   "r+"  - Read and write (file must exist)
   "w+"  - Read and write (truncates)
   "a+"  - Read and append
   
   Binary mode: append 'b' (e.g., "rb", "wb")
*/

/* Basic usage */
FILE *fp = fopen("data.txt", "r");
if (fp == NULL) {
    perror("Error opening file");
    return EXIT_FAILURE;
}

/* Use file... */

/* Always close files */
if (fclose(fp) != 0) {
    perror("Error closing file");
}

/* Example: Read and write */
void file_example(void) {
    /* Write to file */
    FILE *out = fopen("output.txt", "w");
    if (out == NULL) {
        perror("fopen");
        return;
    }
    
    fprintf(out, "Hello, World!\n");
    fprintf(out, "Number: %d\n", 42);
    fclose(out);
    
    /* Read from file */
    FILE *in = fopen("output.txt", "r");
    if (in == NULL) {
        perror("fopen");
        return;
    }
    
    char buffer[256];
    while (fgets(buffer, sizeof(buffer), in) != NULL) {
        printf("%s", buffer);
    }
    fclose(in);
}
```

### Reading and Writing

**Character I/O:**
```c
/* Read/write single character */
int fgetc(FILE *stream);        /* Returns char or EOF */
int fputc(int c, FILE *stream);

int c;
while ((c = fgetc(fp)) != EOF) {
    fputc(c, stdout);  /* Echo to screen */
}

/* Unget character (push back) */
ungetc(c, fp);  /* Put character back into stream */
```

**String I/O:**
```c
/* Read line */
char *fgets(char *str, int n, FILE *stream);

char buffer[256];
if (fgets(buffer, sizeof(buffer), fp) != NULL) {
    /* Remove newline if present */
    size_t len = strlen(buffer);
    if (len > 0 && buffer[len-1] == '\n') {
        buffer[len-1] = '\0';
    }
    printf("Read: %s\n", buffer);
}

/* Write string */
int fputs(const char *str, FILE *stream);

fputs("Hello, World!\n", fp);
```

**Binary I/O:**
```c
/* Read/write blocks of data */
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);

/* Write structure to file */
struct Person {
    char name[50];
    int age;
};

struct Person p = {"Alice", 30};
FILE *fp = fopen("data.bin", "wb");
if (fp != NULL) {
    fwrite(&p, sizeof(struct Person), 1, fp);
    fclose(fp);
}

/* Read structure from file */
struct Person p2;
fp = fopen("data.bin", "rb");
if (fp != NULL) {
    if (fread(&p2, sizeof(struct Person), 1, fp) == 1) {
        printf("Name: %s, Age: %d\n", p2.name, p2.age);
    }
    fclose(fp);
}

/* Write array */
int numbers[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
fp = fopen("numbers.bin", "wb");
fwrite(numbers, sizeof(int), 10, fp);
fclose(fp);

/* Read array */
int nums[10];
fp = fopen("numbers.bin", "rb");
size_t count = fread(nums, sizeof(int), 10, fp);
printf("Read %zu numbers\n", count);
fclose(fp);
```

### Formatted I/O

```c
/* fprintf - Write formatted */
FILE *fp = fopen("log.txt", "w");
fprintf(fp, "User: %s, ID: %d, Score: %.2f\n", "Alice", 123, 95.5);
fclose(fp);

/* fscanf - Read formatted */
fp = fopen("log.txt", "r");
char name[50];
int id;
float score;
if (fscanf(fp, "User: %s, ID: %d, Score: %f", name, &id, &score) == 3) {
    printf("Parsed: %s, %d, %.2f\n", name, id, score);
}
fclose(fp);

/* snprintf - Safe formatted string */
char buffer[100];
snprintf(buffer, sizeof(buffer), "Value: %d", 42);
```

### Binary vs Text Mode

```c
/* Text mode (default) */
/* - Newline translation (LF <-> CRLF on Windows) */
/* - May truncate at Ctrl-Z on Windows */
FILE *text = fopen("file.txt", "r");

/* Binary mode */
/* - No translation */
/* - Exact byte-for-byte I/O */
FILE *binary = fopen("file.bin", "rb");

/* Why it matters: */
/* Text mode on Windows: */
/*   Writing '\n' -> stored as '\r\n' (2 bytes) */
/*   Reading '\r\n' -> returned as '\n' (1 byte) */

/* Binary mode: */
/*   Writing '\n' -> stored as '\n' (1 byte) */
/*   Reading '\r\n' -> returned as '\r\n' (2 bytes) */

/* Always use binary mode for non-text files */
FILE *image = fopen("photo.jpg", "rb");
FILE *data = fopen("data.bin", "wb");
```

### File Positioning

```c
/* Get current position */
long ftell(FILE *stream);

/* Set position */
int fseek(FILE *stream, long offset, int whence);
/* whence: SEEK_SET (beginning), SEEK_CUR (current), SEEK_END (end) */

/* Reset to beginning */
void rewind(FILE *stream);

/* Example: Get file size */
long get_file_size(FILE *fp) {
    long current = ftell(fp);  /* Save position */
    fseek(fp, 0, SEEK_END);    /* Go to end */
    long size = ftell(fp);     /* Get position (size) */
    fseek(fp, current, SEEK_SET);  /* Restore position */
    return size;
}

/* Random access example */
FILE *fp = fopen("data.bin", "r+b");
fseek(fp, 10 * sizeof(int), SEEK_SET);  /* Skip to 11th int */
int value = 42;
fwrite(&value, sizeof(int), 1, fp);  /* Overwrite */
fclose(fp);
```

### Error Handling

```c
/* Check for errors */
int ferror(FILE *stream);  /* Returns non-zero if error occurred */
int feof(FILE *stream);    /* Returns non-zero if EOF reached */
void clearerr(FILE *stream);  /* Clear error and EOF flags */

/* Example */
FILE *fp = fopen("data.txt", "r");
char buffer[256];

while (fgets(buffer, sizeof(buffer), fp) != NULL) {
    printf("%s", buffer);
}

/* Why did loop end? */
if (ferror(fp)) {
    fprintf(stderr, "Error reading file\n");
} else if (feof(fp)) {
    printf("Reached end of file\n");
}

fclose(fp);

/* perror - Print error message */
FILE *fp = fopen("missing.txt", "r");
if (fp == NULL) {
    perror("fopen");  /* Prints: "fopen: No such file or directory" */
    return EXIT_FAILURE;
}
```

### Low-Level File I/O (POSIX)

```c
#include <fcntl.h>
#include <unistd.h>

/* Open file - returns file descriptor */
int fd = open("file.txt", O_RDONLY);
if (fd == -1) {
    perror("open");
    return EXIT_FAILURE;
}

/* Read */
char buffer[1024];
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));

/* Write */
const char *msg = "Hello";
ssize_t bytes_written = write(fd, msg, strlen(msg));

/* Close */
close(fd);

/* Flags for open: */
/* O_RDONLY, O_WRONLY, O_RDWR - access mode */
/* O_CREAT - create if doesn't exist */
/* O_TRUNC - truncate to zero length */
/* O_APPEND - append mode */

/* Create file with permissions */
int fd = open("newfile.txt", O_CREAT | O_WRONLY, 0644);
```

### Complete File I/O Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Copy file */
int copy_file(const char *src, const char *dest) {
    FILE *in = fopen(src, "rb");
    if (in == NULL) {
        perror("Source file");
        return -1;
    }
    
    FILE *out = fopen(dest, "wb");
    if (out == NULL) {
        perror("Destination file");
        fclose(in);
        return -1;
    }
    
    char buffer[4096];
    size_t bytes;
    
    while ((bytes = fread(buffer, 1, sizeof(buffer), in)) > 0) {
        if (fwrite(buffer, 1, bytes, out) != bytes) {
            fprintf(stderr, "Write error\n");
            fclose(in);
            fclose(out);
            return -1;
        }
    }
    
    if (ferror(in)) {
        fprintf(stderr, "Read error\n");
        fclose(in);
        fclose(out);
        return -1;
    }
    
    fclose(in);
    fclose(out);
    return 0;
}

/* Read entire file into memory */
char *read_file(const char *filename, size_t *size) {
    FILE *fp = fopen(filename, "rb");
    if (fp == NULL) {
        return NULL;
    }
    
    /* Get file size */
    fseek(fp, 0, SEEK_END);
    long fsize = ftell(fp);
    rewind(fp);
    
    /* Allocate buffer */
    char *buffer = malloc(fsize + 1);
    if (buffer == NULL) {
        fclose(fp);
        return NULL;
    }
    
    /* Read file */
    size_t bytes_read = fread(buffer, 1, fsize, fp);
    buffer[bytes_read] = '\0';
    
    fclose(fp);
    
    if (size != NULL) {
        *size = bytes_read;
    }
    
    return buffer;
}
```

### Common Pitfalls

```c
/* 1. Forgetting to close file */
FILE *fp = fopen("file.txt", "r");
/* Use file... */
/* Forgot fclose(fp); - resource leak */

/* 2. Not checking fopen return */
FILE *fp = fopen("missing.txt", "r");
fgets(buffer, sizeof(buffer), fp);  /* ❌ Crash if fp is NULL */

/* 3. Wrong mode (truncates file) */
FILE *fp = fopen("important.txt", "w");  /* ❌ Deletes all content! */

/* 4. Buffer overflow with gets */
gets(buffer);  /* ❌ NEVER USE - no bounds check */
fgets(buffer, sizeof(buffer), stdin);  /* ✅ Use this instead */

/* 5. Text mode for binary data */
FILE *fp = fopen("image.png", "r");  /* ❌ Should be "rb" */

/* 6. Not handling EOF correctly */
while (!feof(fp)) {  /* ❌ Wrong - feof only true after read fails */
    fgetc(fp);
}

/* ✅ Correct */
int c;
while ((c = fgetc(fp)) != EOF) {
    /* Process c */
}
```

### Best Practices

1. **Always check fopen return value**
2. **Always close files** (use RAII pattern)
3. **Use binary mode** for binary data
4. **Check read/write return values**
5. **Use fgets** instead of gets
6. **Flush output** if needed (fflush)
7. **Handle errors** properly (ferror, errno)
8. **Use buffering** wisely (setvbuf)
9. **Avoid mixing stdio and low-level I/O** on same file
10. **Test with large files** and edge cases

### Performance Implications

```c
/* Buffering significantly improves performance */
/* Default buffer size: usually 4096-8192 bytes */

/* Unbuffered (slow) */
for (int i = 0; i < 1000000; i++) {
    fputc('a', fp);  /* 1 million system calls */
}

/* Buffered (fast) */
char buffer[4096];
memset(buffer, 'a', sizeof(buffer));
for (int i = 0; i < 1000000 / 4096; i++) {
    fwrite(buffer, 1, sizeof(buffer), fp);  /* ~244 system calls */
}

/* Custom buffering */
char my_buffer[65536];
setvbuf(fp, my_buffer, _IOFBF, sizeof(my_buffer));

/* Buffering modes:
   _IONBF - Unbuffered
   _IOLBF - Line buffered (stdout typically)
   _IOFBF - Fully buffered (files typically)
*/
```

### Key Takeaways

- Use stdio.h for high-level buffered I/O
- Always check fopen return value (NULL on error)
- Close files with fclose when done
- Use binary mode ("rb", "wb") for non-text files
- fgets for reading lines (safer than gets)
- fread/fwrite for binary data
- fprintf/fscanf for formatted I/O
- Check ferror and feof to determine why read failed
- Use fseek/ftell for random access
- Low-level I/O (open/read/write) for more control

---

## 4.2 Error Handling

### Return Codes Convention

```c
/* Success = 0, Error = non-zero (negative or positive) */
int process_data(const char *filename) {
    if (filename == NULL) {
        return -1;  /* Invalid argument */
    }
    
    FILE *fp = fopen(filename, "r");
    if (fp == NULL) {
        return -2;  /* File open error */
    }
    
    /* Process... */
    
    fclose(fp);
    return 0;  /* Success */
}

/* Check return value */
int result = process_data("data.txt");
if (result != 0) {
    fprintf(stderr, "Error: %d\n", result);
}

/* Define error codes */
#define SUCCESS 0
#define ERR_INVALID_ARG -1
#define ERR_FILE_OPEN -2
#define ERR_NO_MEMORY -3
```

### errno and perror

```c
#include <errno.h>
#include <string.h>

/* errno - Global error code set by system calls */
FILE *fp = fopen("missing.txt", "r");
if (fp == NULL) {
    printf("errno: %d\n", errno);
    printf("Error: %s\n", strerror(errno));
}

/* perror - Print error message */
fp = fopen("missing.txt", "r");
if (fp == NULL) {
    perror("fopen");  /* Prints: "fopen: No such file or directory" */
}

/* Common errno values:
   ENOENT - No such file or directory
   EACCES - Permission denied
   ENOMEM - Out of memory
   EINVAL - Invalid argument
*/

/* Check specific errors */
fp = fopen("file.txt", "r");
if (fp == NULL) {
    if (errno == ENOENT) {
        fprintf(stderr, "File does not exist\n");
    } else if (errno == EACCES) {
        fprintf(stderr, "Permission denied\n");
    } else {
        perror("fopen");
    }
}
```

### Error Handling Patterns

**Pattern 1: Return error codes**
```c
typedef enum {
    STATUS_OK = 0,
    STATUS_ERROR_IO = -1,
    STATUS_ERROR_MEMORY = -2,
    STATUS_ERROR_INVALID = -3
} Status;

Status read_config(const char *filename, Config *config) {
    if (filename == NULL || config == NULL) {
        return STATUS_ERROR_INVALID;
    }
    
    FILE *fp = fopen(filename, "r");
    if (fp == NULL) {
        return STATUS_ERROR_IO;
    }
    
    /* Read config... */
    
    fclose(fp);
    return STATUS_OK;
}
```

**Pattern 2: Error via output parameter**
```c
int *allocate_array(size_t size, int *error) {
    if (size == 0) {
        if (error) *error = EINVAL;
        return NULL;
    }
    
    int *arr = malloc(size * sizeof(int));
    if (arr == NULL) {
        if (error) *error = ENOMEM;
        return NULL;
    }
    
    if (error) *error = 0;
    return arr;
}

/* Usage */
int error;
int *arr = allocate_array(100, &error);
if (arr == NULL) {
    fprintf(stderr, "Error: %s\n", strerror(error));
}
```

**Pattern 3: Global error state**
```c
static int last_error = 0;

void set_error(int error) {
    last_error = error;
}

int get_last_error(void) {
    return last_error;
}

const char *get_error_string(void) {
    switch (last_error) {
        case 0: return "No error";
        case -1: return "Invalid argument";
        case -2: return "Out of memory";
        default: return "Unknown error";
    }
}
```

### Assert and Debugging

```c
#include <assert.h>

/* assert - Check condition, abort if false */
void process_array(int *arr, size_t size) {
    assert(arr != NULL);  /* Debug check */
    assert(size > 0);
    
    /* Process array... */
}

/* Assertions disabled in release builds */
/* Compile with -DNDEBUG to disable */

/* Static assertions (C11) */
_Static_assert(sizeof(int) >= 4, "int must be at least 32 bits");

/* Custom assertion */
#define REQUIRE(cond, msg) \
    do { \
        if (!(cond)) { \
            fprintf(stderr, "Assertion failed: %s at %s:%d\n", \
                    msg, __FILE__, __LINE__); \
            abort(); \
        } \
    } while(0)

REQUIRE(ptr != NULL, "Pointer must not be NULL");
```

### Error Propagation

```c
/* Propagate errors up call stack */
int level3(void) {
    /* Error occurs */
    return -1;
}

int level2(void) {
    int result = level3();
    if (result != 0) {
        /* Log error */
        fprintf(stderr, "Error in level3\n");
        return result;  /* Propagate */
    }
    return 0;
}

int level1(void) {
    int result = level2();
    if (result != 0) {
        fprintf(stderr, "Error in level2\n");
        return result;  /* Propagate */
    }
    return 0;
}

/* Cleanup on error */
int function_with_cleanup(void) {
    FILE *fp = NULL;
    char *buffer = NULL;
    int result = -1;
    
    fp = fopen("file.txt", "r");
    if (fp == NULL) {
        goto cleanup;
    }
    
    buffer = malloc(1024);
    if (buffer == NULL) {
        goto cleanup;
    }
    
    /* Process... */
    result = 0;
    
cleanup:
    if (buffer) free(buffer);
    if (fp) fclose(fp);
    return result;
}
```

### setjmp/longjmp

```c
#include <setjmp.h>

jmp_buf jump_buffer;

void error_handler(void) {
    fprintf(stderr, "Error occurred, unwinding stack\n");
    longjmp(jump_buffer, 1);  /* Jump back to setjmp */
}

void deep_function(void) {
    /* Some error condition */
    error_handler();  /* Unwind stack */
}

int main(void) {
    if (setjmp(jump_buffer) == 0) {
        /* Normal execution */
        printf("Calling deep_function\n");
        deep_function();
        printf("This won't print\n");
    } else {
        /* Jumped here from longjmp */
        printf("Caught error\n");
    }
    return 0;
}

/* ⚠️ Warning: setjmp/longjmp is dangerous */
/* - Bypasses destructors */
/* - Can leak resources */
/* - Variables may have unexpected values */
/* - Use only for truly exceptional cases */
```

### Signal Handling

```c
#include <signal.h>

void signal_handler(int sig) {
    if (sig == SIGINT) {
        printf("\nCaught Ctrl+C\n");
        exit(0);
    }
}

int main(void) {
    /* Register signal handler */
    signal(SIGINT, signal_handler);
    
    while (1) {
        printf("Running... (Ctrl+C to stop)\n");
        sleep(1);
    }
    
    return 0;
}

/* Common signals:
   SIGINT  - Interrupt (Ctrl+C)
   SIGTERM - Termination request
   SIGSEGV - Segmentation fault
   SIGABRT - Abort signal
*/
```

### Best Practices

1. **Check all function returns** that can fail
2. **Use errno** for system call errors
3. **Provide meaningful error messages**
4. **Clean up resources** on all paths (including errors)
5. **Use goto** for cleanup (acceptable pattern)
6. **Don't ignore errors** silently
7. **Document error codes** in API
8. **Use asserts** for debugging
9. **Fail fast** on unrecoverable errors
10. **Log errors** appropriately

### Key Takeaways

- Return 0 for success, non-zero for errors
- errno provides system error codes
- perror/strerror for error messages
- Always clean up on error paths
- Use goto for cleanup (acceptable)
- assert for debug checks only
- setjmp/longjmp for exception-like handling (use sparingly)
- Signal handlers for asynchronous events
- Document error handling in API
- Test error paths thoroughly

---

## 4.3 Command-Line Processing

### Parsing Command-Line Arguments

```c
/* Basic argument handling */
int main(int argc, char *argv[]) {
    printf("Program name: %s\n", argv[0]);
    printf("Arguments:\n");
    
    for (int i = 1; i < argc; i++) {
        printf("  argv[%d] = %s\n", i, argv[i]);
    }
    
    return 0;
}

/* Manual parsing */
int main(int argc, char *argv[]) {
    int verbose = 0;
    const char *output_file = NULL;
    
    for (int i = 1; i < argc; i++) {
        if (strcmp(argv[i], "-v") == 0 || 
            strcmp(argv[i], "--verbose") == 0) {
            verbose = 1;
        } else if (strcmp(argv[i], "-o") == 0) {
            if (i + 1 < argc) {
                output_file = argv[++i];
            } else {
                fprintf(stderr, "Error: -o requires argument\n");
                return 1;
            }
        } else {
            fprintf(stderr, "Unknown option: %s\n", argv[i]);
            return 1;
        }
    }
    
    printf("Verbose: %d\n", verbose);
    printf("Output: %s\n", output_file ? output_file : "(none)");
    
    return 0;
}
```

### getopt (POSIX)

```c
#include <unistd.h>
#include <getopt.h>

int main(int argc, char *argv[]) {
    int opt;
    int verbose = 0;
    const char *output = NULL;
    
    /* Parse options */
    while ((opt = getopt(argc, argv, "vo:h")) != -1) {
        switch (opt) {
            case 'v':
                verbose = 1;
                break;
            case 'o':
                output = optarg;  /* Option argument */
                break;
            case 'h':
                printf("Usage: %s [-v] [-o output] files...\n", argv[0]);
                return 0;
            case '?':  /* Unknown option */
                return 1;
        }
    }
    
    /* Non-option arguments */
    for (int i = optind; i < argc; i++) {
        printf("File: %s\n", argv[i]);
    }
    
    return 0;
}

/* Option string format:
   "abc"    - Options -a, -b, -c
   "a:bc"   - Option -a requires argument
   "a::bc"  - Option -a has optional argument
*/
```

### Long Options (getopt_long)

```c
#include <getopt.h>

int main(int argc, char *argv[]) {
    static struct option long_options[] = {
        {"verbose", no_argument,       0, 'v'},
        {"output",  required_argument, 0, 'o'},
        {"help",    no_argument,       0, 'h'},
        {0, 0, 0, 0}  /* Terminator */
    };
    
    int opt;
    int option_index = 0;
    
    while ((opt = getopt_long(argc, argv, "vo:h", 
                              long_options, &option_index)) != -1) {
        switch (opt) {
            case 'v':
                printf("Verbose mode\n");
                break;
            case 'o':
                printf("Output: %s\n", optarg);
                break;
            case 'h':
                printf("Help message\n");
                return 0;
        }
    }
    
    return 0;
}

/* Usage:
   program -v --output=file.txt
   program --verbose -o file.txt
*/
```

### Argument Validation

```c
#include <limits.h>
#include <stdlib.h>

/* Safe string to int conversion */
int parse_int(const char *str, int *value) {
    char *endptr;
    errno = 0;
    long result = strtol(str, &endptr, 10);
    
    if (errno == ERANGE || result < INT_MIN || result > INT_MAX) {
        return -1;  /* Out of range */
    }
    
    if (*endptr != '\0') {
        return -1;  /* Invalid characters */
    }
    
    *value = (int)result;
    return 0;
}

/* Validation example */
int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <number>\n", argv[0]);
        return 1;
    }
    
    int value;
    if (parse_int(argv[1], &value) != 0) {
        fprintf(stderr, "Error: '%s' is not a valid integer\n", argv[1]);
        return 1;
    }
    
    printf("Value: %d\n", value);
    return 0;
}
```

### Help and Usage Messages

```c
void print_usage(const char *program_name) {
    printf("Usage: %s [OPTIONS] FILES...\n", program_name);
    printf("\nOptions:\n");
    printf("  -v, --verbose      Enable verbose output\n");
    printf("  -o, --output FILE  Write output to FILE\n");
    printf("  -c, --count N      Set count to N\n");
    printf("  -h, --help         Show this help message\n");
    printf("\nExamples:\n");
    printf("  %s -v file1.txt file2.txt\n", program_name);
    printf("  %s --output=result.txt input.txt\n", program_name);
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        print_usage(argv[0]);
        return 1;
    }
    
    /* Parse arguments... */
    
    return 0;
}
```

### Key Takeaways

- argc/argv provide command-line arguments
- getopt for POSIX-style option parsing
- getopt_long for long options (--verbose)
- Validate all inputs
- Provide clear usage messages
- Handle errors gracefully

---

## 4.4 Compilation and Linking

### Compilation Process Stages

```
Source Code (.c)
     ↓
[1] Preprocessing (#include, #define)
     ↓
Preprocessed Code (.i)
     ↓
[2] Compilation (C to Assembly)
     ↓
Assembly Code (.s)
     ↓
[3] Assembly (Assembly to Machine Code)
     ↓
Object File (.o)
     ↓
[4] Linking (Combine Objects + Libraries)
     ↓
Executable
```

**Step-by-step example:**

```bash
# 1. Preprocessing only
gcc -E program.c -o program.i

# 2. Compile to assembly
gcc -S program.c -o program.s

# 3. Assemble to object file
gcc -c program.c -o program.o

# 4. Link to create executable
gcc program.o -o program

# All in one step
gcc program.c -o program
```

### Object Files

```c
/* math.c */
int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

/* Compile to object file */
// gcc -c math.c -o math.o

/* Object file contains:
   - Compiled machine code
   - Symbol table (function names, etc.)
   - Relocation information
   - Not executable (has unresolved symbols)
*/

/* Inspect symbols */
// nm math.o
// objdump -t math.o
```

### Static Libraries (.a)

```bash
# Create library from object files
gcc -c math.c -o math.o
gcc -c util.c -o util.o
ar rcs libmymath.a math.o util.o

# Use library
gcc main.c -L. -lmymath -o program

# Flags:
# -L. : Add current directory to library search path
# -lmymath : Link with libmymath.a

# Library is statically linked (copied into executable)
```

**Creating and using static library:**

```c
/* math.h */
#ifndef MATH_H
#define MATH_H
int add(int a, int b);
int multiply(int a, int b);
#endif

/* math.c */
#include "math.h"
int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

/* main.c */
#include <stdio.h>
#include "math.h"

int main(void) {
    printf("5 + 3 = %d\n", add(5, 3));
    printf("5 * 3 = %d\n", multiply(5, 3));
    return 0;
}
```

```bash
# Build
gcc -c math.c -o math.o
ar rcs libmath.a math.o
gcc main.c -L. -lmath -o program
./program
```

### Shared Libraries (.so, .dll, .dylib)

```bash
# Create shared library (Linux)
gcc -fPIC -c math.c -o math.o
gcc -shared math.o -o libmath.so

# Use shared library
gcc main.c -L. -lmath -o program

# Run (need to set LD_LIBRARY_PATH)
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
./program

# Or install to system location
sudo cp libmath.so /usr/local/lib
sudo ldconfig
```

**Position-Independent Code (PIC):**

```c
/* PIC code can load at any memory address */
/* Required for shared libraries */
/* -fPIC flag enables PIC generation */

/* Without PIC: Absolute addressing */
mov eax, [0x12345678]  /* Fixed address */

/* With PIC: Relative addressing */
mov eax, [rip + offset]  /* Position-independent */
```

### Symbol Visibility

```c
/* Default: All symbols exported */
int public_function(void) {
    return 42;
}

/* Hidden: Not visible outside library */
__attribute__((visibility("hidden")))
int internal_function(void) {
    return 123;
}

/* Static: File scope only */
static int file_local(void) {
    return 456;
}

/* Control visibility at compile time */
// gcc -fvisibility=hidden ...  /* Hide by default */
```

### Versioning Shared Libraries

```bash
# Version numbering: libname.so.major.minor.patch
# Example: libmath.so.1.2.3

# Create versioned library
gcc -shared -Wl,-soname,libmath.so.1 math.o -o libmath.so.1.2.3

# Create symlinks
ln -s libmath.so.1.2.3 libmath.so.1
ln -s libmath.so.1 libmath.so

# Application links with: -lmath (finds libmath.so)
# At runtime, uses: libmath.so.1 (soname)
# Actual file: libmath.so.1.2.3
```

### Dynamic Loading

```c
#include <dlfcn.h>

/* Load library at runtime */
void *handle = dlopen("./libmath.so", RTLD_LAZY);
if (handle == NULL) {
    fprintf(stderr, "Error: %s\n", dlerror());
    return 1;
}

/* Get function pointer */
typedef int (*add_func)(int, int);
add_func add = (add_func)dlsym(handle, "add");
if (add == NULL) {
    fprintf(stderr, "Error: %s\n", dlerror());
    dlclose(handle);
    return 1;
}

/* Use function */
int result = add(5, 3);
printf("Result: %d\n", result);

/* Close library */
dlclose(handle);
```

### Link-Time Optimization (LTO)

```bash
# LTO enables optimizations across translation units
gcc -flto -c file1.c -o file1.o
gcc -flto -c file2.c -o file2.o
gcc -flto file1.o file2.o -o program

# Whole program optimization
gcc -flto -O3 file1.c file2.c -o program
```

### Key Takeaways

- Compilation: preprocessing → compilation → assembly → linking
- Object files contain compiled code (not executable)
- Static libraries (.a): Linked at compile time, copied into executable
- Shared libraries (.so): Linked at runtime, shared across programs
- PIC required for shared libraries
- dlopen/dlsym for runtime loading
- LTO for cross-file optimization

---

## 4.5 Multi-file Projects

### Project Organization

```
project/
├── include/           # Public headers
│   └── mylib/
│       ├── api.h
│       └── types.h
├── src/              # Source files
│   ├── api.c
│   ├── internal.c
│   └── internal.h    # Private headers
├── tests/            # Test files
│   └── test_api.c
├── docs/             # Documentation
├── build/            # Build artifacts (generated)
├── Makefile          # Build script
└── README.md         # Project description
```

### Header File Best Practices

```c
/* mylib.h - Public interface */
#ifndef MYLIB_H
#define MYLIB_H

#include <stddef.h>  /* For size_t */

/* Forward declarations */
typedef struct MyStruct MyStruct;

/* Function declarations */
MyStruct *mylib_create(size_t size);
void mylib_destroy(MyStruct *obj);
int mylib_process(MyStruct *obj, int value);

#endif /* MYLIB_H */

/* mylib_internal.h - Private interface */
#ifndef MYLIB_INTERNAL_H
#define MYLIB_INTERNAL_H

#include "mylib.h"

/* Actual structure definition (hidden from users) */
struct MyStruct {
    int *data;
    size_t size;
};

/* Internal functions */
int internal_helper(MyStruct *obj);

#endif
```

### Include Guards

```c
/* Traditional include guards */
#ifndef HEADER_H
#define HEADER_H

/* Header content */

#endif /* HEADER_H */

/* Modern: #pragma once (non-standard but widely supported) */
#pragma once

/* Header content */

/* Naming conventions for guards:
   PROJECT_MODULE_FILE_H
   Example: MYLIB_UTIL_STRING_H
*/
```

### Forward Declarations

```c
/* Avoid circular dependencies with forward declarations */

/* a.h */
#ifndef A_H
#define A_H

/* Forward declare B */
typedef struct B B;

typedef struct A {
    B *b_ptr;  /* Pointer to B */
} A;

void a_function(A *a);

#endif

/* b.h */
#ifndef B_H
#define B_H

/* Forward declare A */
typedef struct A A;

typedef struct B {
    A *a_ptr;  /* Pointer to A */
} B;

void b_function(B *b);

#endif
```

### Circular Dependencies

```c
/* ❌ Circular dependency problem */
/* file1.h includes file2.h */
/* file2.h includes file1.h */

/* ✅ Solution: Forward declarations */
/* file1.h */
struct File2Type;  /* Forward declaration */

struct File1Type {
    struct File2Type *ptr;
};

/* file2.h */
struct File1Type;  /* Forward declaration */

struct File2Type {
    struct File1Type *ptr;
};
```

### Translation Unit Boundaries

```c
/* Translation unit = .c file + all #included headers */

/* util.c */
#include "util.h"

/* This .c file is one translation unit */
static int helper(void) {  /* File-scope only */
    return 42;
}

int public_function(void) {  /* External linkage */
    return helper();
}

/* Each .c file compiles independently */
/* Linker combines them later */
```

### One Definition Rule (ODR)

```c
/* ✅ Declaration (can appear multiple times) */
extern int global_var;
int function(void);

/* ❌ Definition (must appear exactly once across all files) */
int global_var = 42;
int function(void) { return 0; }

/* ❌ ODR violation */
/* file1.c */
int global_var = 42;

/* file2.c */
int global_var = 100;  /* ❌ Multiple definitions */

/* ✅ Correct approach */
/* header.h */
extern int global_var;  /* Declaration */

/* one_file.c */
int global_var = 42;  /* Single definition */
```

### Dependency Management

```c
/* Minimize header dependencies */

/* ❌ Bad: header includes everything */
/* widget.h */
#include "window.h"
#include "graphics.h"
#include "events.h"

/* ✅ Good: minimal includes */
/* widget.h */
struct Window;  /* Forward declaration */
struct Graphics;

typedef struct Widget Widget;

Widget *widget_create(struct Window *w);
void widget_draw(Widget *w, struct Graphics *g);

/* widget.c */
#include "widget.h"
#include "window.h"     /* Full include only in .c */
#include "graphics.h"
```

### Build System Example (Makefile)

```makefile
# Makefile for multi-file project

CC = gcc
CFLAGS = -Wall -Wextra -std=c11 -Iinclude
LDFLAGS = 
LIBS = 

# Directories
SRCDIR = src
INCDIR = include
BUILDDIR = build
TARGET = bin/program

# Source files
SOURCES = $(wildcard $(SRCDIR)/*.c)
OBJECTS = $(patsubst $(SRCDIR)/%.c,$(BUILDDIR)/%.o,$(SOURCES))

# Default target
all: $(TARGET)

# Link
$(TARGET): $(OBJECTS)
	@mkdir -p $(dir $@)
	$(CC) $(LDFLAGS) -o $@ $^ $(LIBS)

# Compile
$(BUILDDIR)/%.o: $(SRCDIR)/%.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c -o $@ $<

# Clean
clean:
	rm -rf $(BUILDDIR) $(TARGET)

# Rebuild
rebuild: clean all

.PHONY: all clean rebuild
```

### Complete Multi-File Example

**Project structure:**
```
calculator/
├── include/
│   └── calc.h
├── src/
│   ├── main.c
│   ├── operations.c
│   └── parser.c
└── Makefile
```

**calc.h:**
```c
#ifndef CALC_H
#define CALC_H

int add(int a, int b);
int subtract(int a, int b);
int parse_expression(const char *expr);

#endif
```

**operations.c:**
```c
#include "calc.h"

int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}
```

**parser.c:**
```c
#include "calc.h"
#include <stdio.h>
#include <string.h>

int parse_expression(const char *expr) {
    /* Simple parser implementation */
    int a, b;
    char op;
    
    if (sscanf(expr, "%d %c %d", &a, &op, &b) != 3) {
        return 0;
    }
    
    switch (op) {
        case '+': return add(a, b);
        case '-': return subtract(a, b);
        default: return 0;
    }
}
```

**main.c:**
```c
#include <stdio.h>
#include "calc.h"

int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("Usage: %s '<expression>'\n", argv[0]);
        return 1;
    }
    
    int result = parse_expression(argv[1]);
    printf("Result: %d\n", result);
    
    return 0;
}
```

**Makefile:**
```makefile
CC = gcc
CFLAGS = -Wall -Iinclude
TARGET = calculator

SRCS = src/main.c src/operations.c src/parser.c
OBJS = $(SRCS:.c=.o)

$(TARGET): $(OBJS)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

.PHONY: clean
```

### Key Takeaways

- Organize code by module, not file type
- Public headers in include/, private in src/
- Use include guards or #pragma once
- Forward declarations avoid circular dependencies
- One definition per variable/function across all files
- Minimize header dependencies
- Use build systems (Make, CMake) for automation
- Test code modularly
- Document public interfaces
- Keep translation units focused

---

## Part 4 Complete Summary

**Part 4: Advanced C Features** covered essential topics for building real-world C applications:

### 4.1 File I/O
- Standard I/O library (stdio.h) with FILE pointers
- Opening/closing files (fopen, fclose)
- Reading/writing (fread, fwrite, fgets, fputs, fgetc, fputc)
- Formatted I/O (fprintf, fscanf)
- Binary vs text mode differences
- File positioning (fseek, ftell, rewind)
- Error handling (ferror, feof, clearerr)
- Standard streams (stdin, stdout, stderr)
- Low-level POSIX I/O (open, read, write, close)
- Complete file copy and read examples
- Performance implications of buffering

### 4.2 Error Handling
- Return code conventions (0 = success)
- errno and perror for system errors
- Error handling patterns (return codes, output parameters, global state)
- Assert for debugging (debug-only checks)
- Error propagation strategies
- setjmp/longjmp for exception-like handling
- Signal handling for asynchronous events
- Cleanup patterns with goto
- Best practices for robust error handling

### 4.3 Command-Line Processing
- argc/argv basics
- Manual argument parsing
- getopt for POSIX-style options
- getopt_long for long options (--verbose)
- Argument validation and conversion
- Help and usage message patterns
- Real-world CLI examples

### 4.4 Compilation and Linking
- Four-stage compilation process (preprocess, compile, assemble, link)
- Object files (.o) and symbol tables
- Static libraries (.a) - compile-time linking
- Shared libraries (.so, .dll, .dylib) - runtime linking
- Position-independent code (PIC) for shared libraries
- Symbol visibility control
- Library versioning strategies
- Dynamic loading with dlopen/dlsym
- Link-time optimization (LTO)
- Build process examples

### 4.5 Multi-file Projects
- Project directory organization
- Header file best practices (public vs private)
- Include guards and #pragma once
- Forward declarations to break circular dependencies
- Translation unit concepts
- One Definition Rule (ODR)
- Dependency management strategies
- Complete multi-file project example
- Makefile for build automation

**Key Themes:**
- Proper resource management (files, memory, errors)
- Error handling throughout the stack
- Modular project organization
- Build process understanding
- Professional C development practices

**Next:** Part 5 will cover the C Standard Library in depth.

**Series Progress:**
- ✅ Part 1: C Fundamentals & Language Core (30,438 words)
- ✅ Part 2: Memory Management & Pointers (13,312 words)
- ✅ Part 3: Derived Types & Data Structures (6,247 words)
- ✅ Part 4: Advanced C Features (6,200+ words)
- **Total: ~56,000 words** across four comprehensive parts

All parts ready for publication on your Jekyll blog with the Chirpy theme!
