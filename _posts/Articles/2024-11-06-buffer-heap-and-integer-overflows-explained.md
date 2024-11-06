---
title: Buffer Overflow, Heap Overflow, Integer Overflow - Vulnerabilities Explained
date: 2024-11-06
categories: [Articles]
tags: [buffer, heap, integer, overflow]
img_path: /assets/img/
description: Overview of buffer, heap, and integer overflows, their risks, and how to prevent them with secure coding practices.
---

In the world of software security, vulnerabilities such as buffer overflows, heap overflows, and integer overflows are some of the most dangerous issues that can lead to unexpected behavior, crashes, and potential exploits. Each of these vulnerabilities stems from improper handling of memory, and they can be leveraged by attackers to take control of a system, execute arbitrary code, or cause a denial of service. In this article, we will explain each of these vulnerabilities with examples.

## Buffer Overflow

### What is a Buffer Overflow

A buffer overflow occurs when data exceeds the boundary of a memory buffer, which is a fixed-size region of memory used to store data. When the data goes beyond the allocated buffer space, it can overwrite adjacent memory locations. This can corrupt data, crash the program, or even allow an attacker to inject malicious code into the program’s memory.

### Example of a Vulnerable Program

Let's take an example of a simple C program that reads user input into a fixed-size buffer:

```c
#include <stdio.h>
#include <string.h>

void vulnerable_function(char *input) {
    char buffer[16];
    strcpy(buffer, input);  // No bounds checking, risky operation
    printf("Input was: %s\n", buffer);
}

int main() {
    char input[100];
    printf("Enter a string: ");
    fgets(input, 100, stdin);
    vulnerable_function(input);
    return 0;
}
```

In the example above, the function `vulnerable_function()` copies user input into a fixed-size buffer of 16 bytes (16 chars) using the `strcpy()` function. However, `strcpy()` does not check the size of the input, meaning that if the user provides more than 16 bytes of input, it will overflow the buffer and overwrite adjacent memory.

Let's run this code!

First with legit input:

```shell
$ ./buffer
Enter a string: abcdef
Input was: abcdef
```

Then with too long input:

```shell
$ ./buffer
Enter a string: abcdefghijklmnopqrstuvwxyz
Input was: abcdefghijklmnopqrstuvwxyz

*** stack smashing detected ***: terminated
Aborted (core dumped)
```

We can see that when the input is longer than the allocated memory space it crashes the application!

### Mitigation

To avoid the buffer overflow vulnerability follow these rules:

1. Use safe string manipulation functions such as `strncpy()` or `snprintf()` that allow you to specify the maximum size of the buffer
2. Limit input size by explicitly checking the length of the user input
3. Avoid using fixed-size buffers wherever possible, but if you must, always ensure they are large enough and properly bounded

Here is a secured version of the previous code:

```c
#include <stdio.h>
#include <string.h>

#define MAX_INPUT_SIZE 16  // Define the maximum input size

void secure_function(char *input) {
    char buffer[MAX_INPUT_SIZE + 1];  // Make sure there's room for the null terminator

    // Use snprintf() to prevent overflow, specifying the buffer size
    snprintf(buffer, sizeof(buffer), "%s", input);
    printf("Input was: %s\n", buffer);
}

int main() {
    char input[MAX_INPUT_SIZE + 1];  // Ensure the input buffer can hold the maximum size

    printf("Enter a string (max %d characters): ", MAX_INPUT_SIZE);

    // Use fgets() to read input safely, and ensure we don't read more than MAX_INPUT_SIZE characters
    if (fgets(input, sizeof(input), stdin) != NULL) {
        // Remove the trailing newline character if present
        input[strcspn(input, "\n")] = '\0';
        secure_function(input);
    } else {
        printf("Input error!\n");
    }

    return 0;
}
```

Let's run it again to verify:

```shell
$ ./no_buffer
Enter a string (max 16 characters): abcdef
Input was: abcdef
```

Then with too long input:

```shell
$ ./no_buffer
Enter a string (max 16 characters): abcdefghijklmnopqrstuvwxyz
Input was: abcdefghijklmnop
```

The input is now safely truncated and crashes are avoided!

## Heap Overflow

### What is a Heap Overflow

A heap overflow occurs when data exceeds the bounds of a memory buffer that resides in the heap area. Unlike buffer overflows, which are often associated with stack memory, heap overflows target dynamically allocated memory that is managed by functions like malloc() and calloc(). When data is written beyond the allocated heap buffer, it can corrupt adjacent memory on the heap, which may contain critical structures like metadata for memory management or function pointers. This can lead to arbitrary code execution, program crashes, or even privilege escalation.

### Example of a Vulnerable Program

Here's an example of a C program with a vulnerability in heap memory allocation:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void vulnerable_heap_function(char *input) {
    char *buffer = (char *)malloc(16);  // Allocate 16 bytes on the heap
    if (buffer == NULL) {
        printf("Memory allocation failed!\n");
        exit(1);
    }
    strcpy(buffer, input);  // No bounds check
    printf("Input was: %s\n", buffer);
    free(buffer);  // Free the allocated memory
}

int main() {
    char input[100];
    printf("Enter a string: ");
    fgets(input, 100, stdin);
    vulnerable_heap_function(input);
    return 0;
}
```

### Mitigation

To prevent heap overflows, follow these best practices:

1. Use safe memory management functions: Instead of `strcpy()`, use `strncpy()` or `snprintf()` to specify a maximum buffer size
2. Validate input size: Check that input data does not exceed the size of allocated memory
3. Implement bounds-checking libraries: Libraries like `libsafe` and other runtime memory management tools can provide extra protection against overflow attacks

Here's a safer version of the above program:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_HEAP_INPUT_SIZE 16

void secure_heap_function(char *input) {
    char *buffer = (char *)malloc(MAX_HEAP_INPUT_SIZE + 1);  // Allocate space for the null terminator
    if (buffer == NULL) {
        printf("Memory allocation failed!\n");
        exit(1);
    }
    strncpy(buffer, input, MAX_HEAP_INPUT_SIZE);
    buffer[MAX_HEAP_INPUT_SIZE] = '\0';  // Ensure the string is null-terminated
    printf("Input was: %s\n", buffer);
    free(buffer);
}

int main() {
    char input[100];
    printf("Enter a string (max %d characters): ", MAX_HEAP_INPUT_SIZE);
    fgets(input, 100, stdin);
    input[strcspn(input, "\n")] = '\0';  // Remove trailing newline
    secure_heap_function(input);
    return 0;
}
```

## Integer Overflow

### What is an Integer Overflow

An integer overflow happens when an arithmetic operation results in a value outside the allowed range for a data type. For instance, if an int type can hold values from -2,147,483,648 to 2,147,483,647, adding 1 to 2,147,483,647 will cause an overflow and may wrap around to the minimum value. This unintended behavior can lead to logic errors, memory corruption, or even security vulnerabilities if the overflowed value is used in memory allocation or other critical calculations.

### Example of a Vulnerable Program

Here’s a simple example where integer overflow could lead to unexpected behavior:

```c
#include <stdio.h>
#include <stdlib.h>

void allocate_buffer(int size) {
    char *buffer;
    if (size > 0) {
        buffer = (char *)malloc(size);
        if (buffer == NULL) {
            printf("Memory allocation failed!\n");
            return;
        }
        printf("Buffer allocated with size: %d\n", size);
        free(buffer);
    } else {
        printf("Invalid buffer size!\n");
    }
}

int main() {
    int size = 2147483647;  // Max value for a signed 32-bit integer
    size += 1;  // Causes an integer overflow
    allocate_buffer(size);
    return 0;
}
```

In this example, size overflows to a negative value due to the addition, which could cause unexpected behavior in malloc() and potentially lead to a security vulnerability.

### Mitigation

To mitigate integer overflows, follow these guidelines:

1. Check for overflows: Before performing arithmetic operations, verify if the result will fit within the type's range
2. Use safer integer libraries: Libraries like `SafeInt` provide functions to detect overflows in arithmetic operations
3. Choose appropriate data types: Use data types with a large enough range or with built-in overflow protection if available

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

void allocate_buffer(int size) {
    char *buffer;
    if (size > 0 && size < INT_MAX) {
        buffer = (char *)malloc(size);
        if (buffer == NULL) {
            printf("Memory allocation failed!\n");
            return;
        }
        printf("Buffer allocated with size: %d\n", size);
        free(buffer);
    } else {
        printf("Invalid buffer size!\n");
    }
}

int main() {
    int size = INT_MAX;  // Set to max value to test overflow
    if (size + 1 > INT_MAX) {
        printf("Integer overflow detected! Adjusting size.\n");
        size = INT_MAX;
    } else {
        size += 1;
    }
    allocate_buffer(size);
    return 0;
}
```

Now, the program detects and prevents overflow, safeguarding against unintended behavior.

## Conclusion

Understanding buffer, heap, and integer overflows is crucial for anyone working in software security. Each of these vulnerabilities originates from improper memory or data handling, and, if exploited, can lead to severe consequences such as arbitrary code execution, system crashes, or data corruption. By knowing how these overflows work and implementing proper safeguards, developers can better protect their applications against such attacks. Techniques like safe memory management, rigorous bounds checking, and using modern compiler defenses play a vital role in mitigating these risks.
