# Arm Software Engineer / Senior Software Engineer
# C Programming Assessment Question Bank
## Hard → Advanced → Intermediate → Beginner

**Target:** Arm Software Engineer / Senior Software Engineer  
**Assessment duration:** 20 minutes  
**Language:** C  
**Purpose:** Practice question bank based on public candidate experiences, published C assessment examples, and systems-programming fundamentals.

> **Important:** This is a preparation resource, not a confirmed list of Arm's actual questions. The exact assessment content is unknown.

## How to use this bank

- Practise writing code yourself before checking references.
- Compile and run your code where possible.
- Test empty input, boundary values, duplicates, and failure cases.
- For each algorithm, explain time and auxiliary-space complexity.
- For a 20-minute assessment, prioritize correctness and simple, readable solutions.

# Table of Contents

1. Hard — Algorithms and Data Structures
2. Hard — Concurrency and Systems Programming
3. Hard — Debugging and Code Analysis
4. Advanced — Undefined Behavior and Tricky C
5. Advanced — Structures, Unions and Alignment
6. Advanced — Preprocessor and Compilation
7. Advanced — Embedded C and Hardware
8. Intermediate — Memory Management
9. Intermediate — Bit Manipulation
10. Intermediate — Pointers
11. Intermediate — Functions, Storage Classes and Qualifiers
12. Intermediate — Arrays and Strings
13. Beginner — Operators and Control Flow
14. Beginner — C Fundamentals
15. Output Prediction Questions
16. Multiple-Choice Questions
17. Timed Mock Assessment
18. Final Revision Checklist

---

# 1. Hard — Algorithms and Data Structures

## 1.1 Searching and sorting

1. Implement binary search iteratively.
2. Implement binary search recursively.
3. Explain why binary search requires sorted input.
4. Implement insertion sort.
5. Implement merge sort.
6. Implement quicksort and explain its worst-case complexity.
7. Compare bubble sort, insertion sort, merge sort and quicksort.
8. What is algorithmic time complexity?
9. What is auxiliary-space complexity?
10. How would you find the Kth-largest element?
11. How would you merge K sorted arrays?
12. How would you search a sorted array that has been rotated?
13. How would you find the first and last positions of a target in a sorted array?
14. How would you detect a cycle in a linked list?
15. How would you find the middle element of a linked list?

## 1.2 Hard coding problems

### Q1. Two Sum

Given an array and a target, return the indices of two distinct elements whose sum equals the target.

Provide both an O(n²) solution and a faster solution using an appropriate data structure.

### Q2. Maximum Subarray Sum

Given an integer array, find the maximum sum of a contiguous subarray.

Explain Kadane's algorithm and handle arrays containing only negative numbers.

### Q3. First Missing Positive

Find the smallest positive integer missing from an unsorted array.

Target: O(n) time and O(1) auxiliary space.

### Q4. Longest Substring Without Repeating Characters

Find the length of the longest substring containing no repeated characters.

Discuss the character-set assumptions and complexity.

### Q5. Merge Overlapping Intervals

Given a list of intervals, merge all overlapping intervals.

### Q6. Product of Array Except Self

For every element, return the product of all other elements without using division.

Discuss integer overflow.

### Q7. Find the Majority Element

Find an element occurring more than half the time, or report that no majority exists.

### Q8. Rotate a Matrix

Rotate an N × N matrix by 90 degrees in place.

### Q9. Implement a Queue Using Two Stacks

Implement enqueue, dequeue and empty operations.

### Q10. LRU Cache

Design a least-recently-used cache with efficient lookup and update operations.

Discuss how a hash table and doubly linked list work together.

---

# 2. Hard — Concurrency and Systems Programming

## 2.1 Concurrency questions

1. What is a data race?
2. What is a race condition?
3. What is the difference between a data race and a race condition?
4. What is an atomic operation?
5. What are C11 atomic types?
6. What is a mutex?
7. What is a semaphore?
8. What is a spinlock?
9. What is a deadlock?
10. What is livelock?
11. What is starvation?
12. What is the happens-before relationship?
13. What is a memory-ordering constraint?
14. What does `memory_order_relaxed` mean?
15. What is acquire-release synchronization?
16. Why does `volatile` not make a shared counter thread-safe?
17. How can a producer-consumer queue be implemented?
18. What problems occur when a lock-free queue is implemented incorrectly?
19. What is false sharing?
20. Why can concurrent code behave differently across processor architectures?

## 2.2 Systems-level questions

1. What is a process?
2. What is a thread?
3. What is virtual memory?
4. What is a page fault?
5. What is a system call?
6. What is the difference between user mode and kernel mode?
7. What is a context switch?
8. What is a memory-mapped file?
9. What is a shared library?
10. What is dynamic linking?
11. How would you investigate high CPU utilization?
12. How would you debug a segmentation fault?
13. How would you investigate a memory leak that appears only after several hours?
14. How would you debug code that fails only in optimized builds?
15. How would you investigate a multithreaded deadlock?

---

# 3. Hard — Debugging and Code Analysis

## 3.1 Debugging scenarios

### Q1. Intermittent crash

A firmware application crashes only after running for several hours. What would you investigate first?

### Q2. Optimization-dependent failure

Code works with `-O0` but fails with `-O2`. What possible causes would you investigate?

### Q3. Memory corruption

A structure's field unexpectedly changes after a function call. How would you identify the source of corruption?

### Q4. Stack overflow

An embedded target resets when a deeply nested function is called. How would you investigate stack usage?

### Q5. Race condition

A shared counter occasionally contains an unexpected value. Explain how to identify and correct the race.

### Q6. Buffer overflow

A UART receive buffer occasionally corrupts adjacent variables. What checks would you add?

### Q7. Memory leak

Heap usage increases slowly during repeated operations. How would you isolate the leak?

### Q8. Hard fault

The processor enters a hard-fault handler. Which registers and diagnostic information would you inspect?

### Q9. Build failure

The source compiles but linking fails with an undefined reference. How would you investigate?

### Q10. Cross-platform failure

Code works on one CPU architecture but fails on another. How would you investigate assumptions about alignment, endianness, integer width and undefined behavior?

---

# 4. Advanced — Undefined Behavior and Tricky C

## 4.1 Undefined behavior questions

1. What is undefined behavior?
2. What is implementation-defined behavior?
3. What is unspecified behavior?
4. What happens when a signed integer overflows?
5. Why is an out-of-bounds array access dangerous?
6. Why is dereferencing an invalid pointer undefined behavior?
7. What is an unsequenced modification?
8. Why should expressions such as `i++ + i++` be avoided?
9. Why is reading an uninitialized automatic variable dangerous?
10. What happens when `printf()` receives a format specifier that does not match its argument?
11. What is a strict-aliasing violation?
12. What is an effective type?
13. Why can casting a pointer to an incompatible type cause problems?
14. What is the role of object alignment?
15. Can a compiler optimize code based on the assumption that undefined behavior never occurs?
16. Why can code that appears to work in a debug build fail under optimization?
17. Why is dereferencing a one-past-the-end pointer invalid?
18. Why can returning a pointer to a local automatic variable be unsafe?
19. What is the problem with modifying a string literal?
20. How can compiler warnings and sanitizers help identify undefined behavior?

## 4.2 Is the code valid?

For each snippet, state whether its behavior is defined, implementation-defined, unspecified or undefined. Explain any assumptions.

### Q1. Signed overflow

```c
#include <limits.h>

int x = INT_MAX;
x = x + 1;
```

### Q2. Out-of-bounds access

```c
int a[3] = {1, 2, 3};
printf("%d", a[3]);
```

### Q3. Returning a local address

```c
int *get_value(void)
{
    int x = 10;
    return &x;
}
```

### Q4. Unsequenced modification

```c
int i = 1;
int x = i++ + i++;
```

### Q5. Modifying a string literal

```c
char *s = "ARM";
s[0] = 'B';
```

### Q6. Invalid shift

```c
unsigned int x = 1;
unsigned int y = x << 32;
```

Assume `unsigned int` is 32 bits.

### Q7. Signed right shift

```c
int x = -8;
int y = x >> 1;
```

Explain why the result is implementation-defined in relevant C versions.

### Q8. Pointer arithmetic

```c
int a[4] = {1, 2, 3, 4};
int *p = a;
int *q = p + 4;
```

Is forming `q` allowed? What operations on `q` are valid?

### Q9. Strict aliasing

```c
float f = 1.0f;
int *p = (int *)&f;
printf("%d", *p);
```

Explain the effective-type and aliasing issue.

### Q10. Format mismatch

```c
int x = 10;
printf("%f", x);
```

Explain the mismatch between the expected and actual variadic argument types.

---

# 5. Advanced — Structures, Unions and Alignment

## 5.1 Structure questions

1. What is a structure in C?
2. What is the difference between a structure and a union?
3. What is structure padding?
4. What is alignment?
5. Why can `sizeof(struct)` be greater than the sum of member sizes?
6. How can member order affect structure size?
7. What is a flexible array member?
8. What is a bit-field?
9. What are the portability limitations of C bit-fields?
10. How do you copy a structure?
11. What problems can occur when comparing structures with `memcmp()`?
12. What is a packed structure?
13. Why can packed structures cause unaligned accesses?
14. Why should structure layout not automatically be treated as a portable wire protocol?
15. How do you design a structure for a hardware register map?

## 5.2 Structure coding problems

### Q1. Predict structure size

```c
struct Example {
    char a;
    int b;
    char c;
};
```

Explain the padding and why the exact size depends on the implementation's alignment rules.

### Q2. Reorder members

Reorder the members to reduce padding where possible.

### Q3. Implement a linked-list node

Create a structure that contains an integer and a pointer to the next node.

### Q4. Deep-copy a structure

Implement a deep copy for a structure that owns a dynamically allocated string.

### Q5. Design a packet structure

Represent a packet containing a header, length and payload. Explain why explicit serialization is safer than transmitting the raw structure bytes.

---

# 6. Advanced — Preprocessor and Compilation

## 6.1 Preprocessor questions

1. What is a macro?
2. What is the difference between a macro and a function?
3. Why should macro parameters be parenthesized?
4. Why should the complete macro expansion often be parenthesized?
5. What is a function-like macro?
6. What is a variadic macro?
7. What is conditional compilation?
8. What are `#ifdef`, `#ifndef`, `#if` and `#elif`?
9. What is an include guard?
10. What is the difference between `#define` and `const`?
11. What is macro stringification?
12. What is token pasting?
13. What is the difference between an object file and a static library?
14. What is the difference between a static library and a shared library?
15. What is a linker error?
16. What is a multiple-definition error?
17. What is a symbol table?
18. What is relocation?
19. What is the difference between a debug build and an optimized build?
20. What does link-time optimization do?

### Macro trap

```c
#define SQUARE(x) x * x

int result = SQUARE(2 + 3);
```

Predict the result and fix the macro.

### Safer macro

```c
#define SQUARE(x) ((x) * (x))
```

Why can this still be unsafe when passed an expression with side effects, such as `i++`?

---

# 7. Advanced — Embedded C and Hardware

## 7.1 Embedded C questions

1. What is memory-mapped I/O?
2. Why are hardware registers commonly declared `volatile`?
3. What is the difference between a hardware register and a normal RAM variable?
4. What is the difference between a polling-based driver and an interrupt-driven driver?
5. What is DMA?
6. What are the alignment requirements for DMA buffers?
7. Why might cache maintenance be required for DMA on a cache-enabled processor?
8. What is an interrupt service routine?
9. Why should an ISR generally execute quickly?
10. Why are blocking operations inappropriate in many ISRs?
11. What is interrupt latency?
12. What is the difference between interrupt masking and disabling a peripheral interrupt?
13. What is a critical section?
14. What are atomic operations?
15. What is the difference between `volatile` and `_Atomic`?
16. How would you safely communicate between an ISR and a task?
17. What is a race condition?
18. What is a priority inversion?
19. What is the difference between a watchdog timer and a software timeout?
20. How would you debug a hard fault or unexpected reset?

## 7.2 Embedded coding problems

### Q1. Register bit update

Given a register pointer, set bit 3 without changing other bits.

Explain the assumptions required for a read-modify-write operation.

### Q2. Clear a status bit

Clear bit 5 of a register. Discuss whether the register has ordinary read/write semantics or special write-one-to-clear behavior.

### Q3. Extract a field

Extract bits 8 through 11 from a 32-bit register.

### Q4. Circular buffer

Implement a fixed-size ring buffer with initialization, push, pop, and full/empty detection.

Define whether the implementation reserves one slot or maintains a separate count.

### Q5. UART receive buffer

Design a buffer that receives bytes from an interrupt handler and is consumed by a foreground task.

Discuss concurrency, overflow and synchronization.

### Q6. Timeout handling

Implement a timeout check using an unsigned tick counter that can wrap around.

State assumptions about the maximum interval and counter width.

### Q7. Endianness

Write code to determine the machine's byte order, then explain why explicit serialization is preferred for network protocols.

### Q8. Checksum

Implement a simple byte-sum checksum over a buffer. Discuss what additional properties a CRC provides.

### Q9. Register field insertion

Insert a value into bits 8–11 of a register while preserving other bits.

### Q10. Power-of-two alignment

Align a buffer size or address to a power-of-two boundary, with overflow checks.

---

# 8. Intermediate — Memory Management

## 8.1 Memory questions

1. What is the difference between stack and heap memory?
2. What is static storage duration?
3. What is dynamic memory allocation?
4. What does `malloc()` return?
5. What is the difference between `malloc()` and `calloc()`?
6. What does `realloc()` do?
7. Why should the result of `realloc()` normally be stored in a temporary pointer?
8. What happens if `malloc()` fails?
9. What does `free(NULL)` do?
10. What happens when memory is freed twice?
11. What is a memory leak?
12. What is a use-after-free error?
13. What is a buffer overflow?
14. What is the difference between shallow copy and deep copy?
15. What happens when an allocated pointer is overwritten before freeing the allocation?
16. Why should allocation-size arithmetic be checked for overflow?
17. What is ownership of dynamically allocated memory?
18. Why must an allocated object not be accessed after `free()`?
19. What is fragmentation?
20. What memory-allocation concerns are especially important in embedded systems?

## 8.2 Memory coding problems

1. Dynamically allocate an integer array of N elements.
2. Check for allocation failure.
3. Initialize every allocated element.
4. Resize an allocated array safely using `realloc()`.
5. Write a function that frees all memory it owns.
6. Find and fix a memory leak in a given program.
7. Find and fix a use-after-free bug.
8. Find and fix a double-free bug.
9. Implement a deep copy of a structure containing a dynamically allocated string.
10. Write a bounded append function for a fixed-size character buffer.

### Bug-finding question

Find the bug:

```c
#include <stdio.h>
#include <stdlib.h>

int *p = malloc(10 * sizeof(*p));

if (p == NULL)
    return 1;

free(p);

printf("%d\n", p[0]);
```

Explain the lifetime violation and correct the program.

---

# 9. Intermediate — Bit Manipulation

## 9.1 Bitwise questions

1. What is a bit mask?
2. How do you set bit N?
3. How do you clear bit N?
4. How do you toggle bit N?
5. How do you read bit N?
6. How do you set a group of bits?
7. How do you clear a selected group of bits?
8. How do you extract a field from a register?
9. How do you insert a field into a register without disturbing other bits?
10. What is the difference between logical and arithmetic right shift?
11. What happens when a shift count is negative or greater than or equal to the width of the promoted left operand?
12. Why should unsigned types usually be preferred for bit manipulation?
13. Why can shifting a signed integer into or beyond its sign bit be problematic?
14. How can you determine whether a number is a power of two?
15. How can you count set bits efficiently?
16. How do you find the lowest set bit?
17. How do you clear the lowest set bit?
18. How can you swap two values using XOR, and why is a temporary variable often clearer?
19. How do you reverse the bits of an integer?
20. How do you check whether two integers have opposite signs without relying on implementation-defined signed representations?

## 9.2 Bit-manipulation coding problems

### Q1. Set a bit

Implement a function to set bit N safely for an unsigned 32-bit integer.

### Q2. Clear a bit

Implement a function that clears bit N.

### Q3. Toggle a bit

Implement a function that toggles bit N.

### Q4. Read a bit

Return 1 if bit N is set, otherwise return 0.

### Q5. Count set bits

Input: `13`  
Binary: `1101`  
Output: `3`

### Q6. Check power of two

Return true for 1, 2, 4, 8, 16 and false for 0 and 12.

### Q7. Find the lowest set bit

Given an unsigned integer, return the value of its lowest set bit.

### Q8. Clear the lowest set bit

Implement:

```c
x = x & (x - 1);
```

Explain why it works.

### Q9. Reverse all bits

Reverse the bits of a 32-bit unsigned integer.

### Q10. Swap upper and lower nibbles

For an 8-bit value, exchange the upper four bits with the lower four bits.

### Q11. Check whether a bit field is set

Given a register value and a mask, determine whether all masked bits are set.

### Q12. Extract a field

Extract `width` bits starting at bit `position`. Consider how to avoid undefined shifts and handle full-width fields.

### Q13. Insert a field

Insert a value into a bit field without modifying unrelated bits.

### Q14. Align an address

Align an unsigned address upwards to a power-of-two boundary. Handle overflow correctly.

### Q15. Find the unique element

In an array where every other integer appears twice and exactly one integer appears once, find the unique integer using XOR.

---

# 10. Intermediate — Pointers

## 10.1 Pointer fundamentals

1. What is a pointer?
2. What is a null pointer?
3. What is a wild pointer?
4. What is a dangling pointer?
5. What is an uninitialized pointer?
6. What is pointer arithmetic?
7. Why does `p + 1` advance by the size of the pointed-to type?
8. What happens when you dereference a null pointer?
9. What is a void pointer?
10. Why must a `void *` be converted to an appropriate object-pointer type before dereferencing it in standard C?
11. What is a pointer to a pointer?
12. What is a function pointer?
13. What is a pointer to an array?
14. What is an array of pointers?
15. What is the difference between `int *p`, `int **p` and `int (*p)[10]`?
16. What is the difference between passing a pointer by value and modifying the caller's pointer through a pointer-to-pointer?
17. What is the difference between pointer equality and comparing pointed-to values?
18. When is pointer arithmetic valid?
19. Why is subtracting pointers to unrelated objects undefined behavior?
20. Why must a pointer not be dereferenced after its object lifetime ends?

## 10.2 Pointer output questions

### Q1

```c
int x = 10;
int *p = &x;

*p = 20;

printf("%d", x);
```

### Q2

```c
int a[] = {10, 20, 30};
int *p = a;

printf("%d", *(p + 1));
```

### Q3

```c
int a[] = {10, 20, 30};
int *p = a;

printf("%d", *p++);
```

### Q4

```c
int a[] = {10, 20, 30};
int *p = a;

printf("%d", (*p)++);
printf("%d", a[0]);
```

### Q5

```c
int x = 10;
int *p = &x;
int **pp = &p;

**pp = 50;

printf("%d", x);
```

### Q6

```c
int a[] = {1, 2, 3, 4};
int *p = &a[1];

printf("%d", p[-1]);
```

### Q7

```c
int a[] = {10, 20, 30};
int *p = a;

printf("%d", *(p + 3));
```

Explain why forming a one-past-the-end pointer is allowed but dereferencing it is not.

### Q8

```c
int x = 10;
const int *p = &x;
```

Can you write `*p = 20`? Why or why not?

### Q9

```c
int x = 10;
int *const p = &x;
```

Can you assign a different address to `p`? Can you modify `*p`?

### Q10

```c
int x = 10;
const int *const p = &x;
```

What operations are prohibited through `p`?

## 10.3 Pointer coding problems

1. Swap two integers using pointers.
2. Reverse an array using two pointers.
3. Implement `strlen()` using pointer traversal.
4. Find the maximum element using pointers.
5. Implement a safe bounded string-copy function.
6. Return multiple values from a function using output pointers.
7. Implement a function that dynamically allocates an array and reports allocation failure.
8. Write a function that allocates an object and transfers ownership to the caller.
9. Explain and fix a double-free bug.
10. Explain and fix a use-after-free bug.
11. Implement a function that modifies a caller's pointer using `T **`.
12. Implement a function pointer callback for an array-processing function.

---

# 11. Intermediate — Functions, Storage Classes and Qualifiers

## 11.1 `const`, `volatile`, `static` and `extern`

1. What does `const` mean in C?
2. What does `volatile` mean?
3. Why does `volatile` not guarantee atomicity?
4. Why does `volatile` not provide thread synchronization?
5. When should a memory-mapped hardware register be declared `volatile`?
6. Can a `const volatile` object be modified by hardware?
7. What is the purpose of `static` at file scope?
8. What is the purpose of `static` inside a function?
9. What does `extern` mean?
10. What is the difference between internal and external linkage?
11. What is the difference between a declaration and a definition of an external object?
12. What is the difference between storage duration and scope?
13. What is the lifetime of a local automatic variable?
14. What is the lifetime of a static-storage-duration object?
15. What is the difference between `const int *p` and `int *const p`?
16. What is the meaning of `const int *const p`?
17. Can a `static` local variable retain its value between function calls?
18. Does declaring a variable `static` automatically make it thread-safe?
19. What problems can occur when a header file defines a non-`static` global variable?
20. How would you organize global variables across several C source files?

### Coding question: static variable

Predict the output:

```c
#include <stdio.h>

void counter(void)
{
    static int count = 0;
    count++;
    printf("%d ", count);
}

int main(void)
{
    counter();
    counter();
    counter();
}
```

### Coding question: external linkage

Explain the difference between:

```c
extern int g_count;
```

and:

```c
static int g_count;
```

Discuss what happens when each is placed at file scope.

---

# 12. Intermediate — Arrays and Strings

## 12.1 Array questions

1. What is an array?
2. How are array elements stored in memory?
3. What is the relationship between an array name and a pointer?
4. Does an array name always behave as a pointer?
5. What is an out-of-bounds array access?
6. What is the difference between `int a[10]` and `int *p`?
7. How do you determine the number of elements in a local array?
8. Why does `sizeof(array) / sizeof(array[0])` fail when used on an array parameter?
9. What is a multidimensional array?
10. How are two-dimensional arrays laid out in memory in C?
11. What is the difference between a fixed-size array and dynamically allocated memory?
12. How do you pass an array to a function?
13. How do you initialize an array?
14. What happens to elements omitted from an aggregate initializer?
15. What is the difference between a character array and a string?

## 12.2 Array coding problems

### Q1. Find the maximum element

Input: `{3, 7, 2, 9, 4}`  
Output: `9`

### Q2. Find the minimum element

Return the smallest element in an integer array.

### Q3. Find the second-largest distinct element

Input: `{5, 2, 9, 7, 9}`  
Output: `7`

Handle arrays containing only duplicate values.

### Q4. Reverse an array in place

Input: `{1, 2, 3, 4, 5}`  
Output: `{5, 4, 3, 2, 1}`

### Q5. Find the sum of array elements

Write an implementation and discuss integer overflow.

### Q6. Search for an element

Implement linear search and return the element's index, or a documented not-found value.

### Q7. Find duplicate elements

Identify duplicate values without using a second array.

### Q8. Remove duplicates

Remove duplicate elements from a sorted array in place and return the new logical length.

### Q9. Find a missing number

Given N distinct numbers from the range 1 through N+1, find the missing number.

Discuss overflow risks when using the arithmetic-series formula.

### Q10. Move zeros to the end

Input: `{0, 1, 0, 3, 12}`  
Output: `{1, 3, 12, 0, 0}`

Preserve the relative order of nonzero elements.

### Q11. Rotate an array

Rotate an array left or right by K positions.

### Q12. Merge two sorted arrays

Produce a single sorted output array.

### Q13. Find two numbers with a target sum

Given an array and target, determine whether two distinct elements sum to the target.

### Q14. Find the intersection of two arrays

Return elements common to both arrays.

### Q15. Find the closest temperature to zero

Input: `{-8, -2, 4, 7}`  
Output: `-2`

Tie rule: if `-5` and `5` are equally close to zero, return `5`.

If the array is empty, return `0`.

Consider integer limits when comparing absolute values.

## 12.3 String questions

1. What is a null-terminated string?
2. Why does a C string require a terminating `'\0'`?
3. What is the difference between `'\0'` and `'0'`?
4. What is the difference between `strlen()` and `sizeof()`?
5. What is the difference between `strcpy()` and `strncpy()`?
6. Why can `strcpy()` cause a buffer overflow?
7. What is the difference between a string literal and a writable character array?
8. Why is modifying a string literal undefined behavior?
9. What does `strcmp()` return?
10. How do you compare two strings correctly?
11. What is the difference between `char s[] = "ARM";` and `char *s = "ARM";`?
12. How can you reverse a string without allocating another array?
13. What are common string-related buffer overflow vulnerabilities?
14. How should a function report that an output buffer is too small?
15. Why is `gets()` unsafe and why was it removed from modern C standards?

## 12.4 String coding problems

1. Reverse a string in place. Input: `"ARM"`; output: `"MRA"`.
2. Check a palindrome string. Input: `"level"`; output: `true`.
3. Count vowels and consonants in a string containing alphabetic characters.
4. Count occurrences of a character. Input: `"embedded"`, character `'e'`; output: `3`.
5. Implement a custom version of `strlen()`.
6. Implement a bounded string-copy function with clearly defined null-termination behavior.
7. Implement a custom version of `strcmp()`.
8. Remove spaces. Input: `"Arm Software Engineer"`; output: `"ArmSoftwareEngineer"`.
9. Find the first non-repeating character. Input: `"swiss"`; output: `'w'`.
10. Check whether two strings are anagrams. Input: `"listen"`, `"silent"`; output: `true`.
11. Reverse words in a sentence. Input: `"Arm C Programming"`; output: `"Programming C Arm"`.
12. Find the longest common prefix. Input: `{"flower", "flow", "flight"}`; output: `"fl"`.

---

# 13. Beginner — Operators and Control Flow

## 13.1 Operator questions

1. What is the difference between `=` and `==`?
2. What is the difference between `&` and `&&`?
3. What is the difference between `|` and `||`?
4. What is the difference between `!` and `~`?
5. What is the difference between `++i` and `i++`?
6. What is operator precedence?
7. What is operator associativity?
8. What is short-circuit evaluation?
9. What is the conditional operator `?:`?
10. What is the difference between `while`, `do-while` and `for` loops?
11. What does `break` do?
12. What does `continue` do?
13. What is the purpose of `switch`?
14. Can a `switch` statement operate on floating-point values?
15. What happens if a `switch` case does not have a `break`?
16. What is an infinite loop?
17. What is the difference between logical and bitwise operations?
18. What happens when an expression mixes signed and unsigned integers?
19. What is integer promotion?
20. Why should complex expressions with multiple increments be avoided?

## 13.2 Output prediction

Predict the output before running each program.

### Q1

```c
int x = 5;
printf("%d", x++);
```

### Q2

```c
int x = 5;
printf("%d", ++x);
```

### Q3

```c
int x = 5;
printf("%d", x & (x - 1));
```

### Q4

```c
int x = 10;

if (x = 0)
    printf("A");
else
    printf("B");
```

Explain the difference between assignment and comparison.

### Q5

```c
int a = 5;
int b = 2;

printf("%d", a / b);
```

### Q6

```c
printf("%d", 5 % 2);
```

### Q7

```c
int x = 0;
printf("%d", x && (10 / x));
```

Explain short-circuit evaluation.

### Q8

```c
int x = 1;
printf("%d", x << 3);
```

### Q9

```c
int x = 10;
printf("%d", x > 5 ? 100 : 200);
```

### Q10

```c
unsigned int x = 1;
printf("%u", x - 2);
```

Explain why the result is not a negative integer.

---

# 14. Beginner — C Fundamentals

## 14.1 Basic C questions

1. What is the difference between C and C++?
2. What are the basic data types in C?
3. What is the difference between `char`, `int`, `float` and `double`?
4. What are the typical sizes of `char`, `short`, `int`, `long` and `long long`?
5. Why is `sizeof(char)` always 1 in C?
6. Is `sizeof(int)` always 4 bytes?
7. What is the difference between declaration and definition?
8. What is the difference between initialization and assignment?
9. What is the purpose of `#include`?
10. What is the purpose of the `main()` function?
11. What is the difference between `printf()` and `scanf()`?
12. What are format specifiers such as `%d`, `%u`, `%x`, `%c`, `%s` and `%p`?
13. What is the difference between signed and unsigned integers?
14. What is integer overflow?
15. What is the difference between implicit and explicit type conversion?
16. What is the difference between a compiler, linker and preprocessor?
17. What is the difference between source code, object code and executable code?
18. What is a header file?
19. Why should header files use include guards?
20. What is the difference between a compile-time error and a runtime error?

## 14.2 Basic coding problems

### Q1. Print numbers from 1 to N

Write a function that prints all integers from 1 through N.

### Q2. Check even or odd

Write a function that determines whether an integer is even or odd.

### Q3. Find the largest of three numbers

Given three integers, return the largest value.

### Q4. Calculate factorial

Calculate the factorial of N using a loop. Consider N = 0 and negative input.

### Q5. Reverse an integer

Input: `1234`; output: `4321`.

What should happen if the input contains trailing zeros or the result overflows an integer?

### Q6. Check a palindrome number

Input: `121`; output: `true`.

### Q7. Check prime number

Write a function to determine whether an integer is prime.

### Q8. Calculate Fibonacci numbers

Generate the first N Fibonacci numbers. Discuss iterative versus recursive implementations.

### Q9. Find the sum of digits

Input: `572`; output: `14`.

### Q10. Swap two variables

Swap two integers using a temporary variable. Explain why this is usually preferable to clever arithmetic-based swapping.

---

# 15. Output Prediction Questions — Consolidated Revision

For each snippet, determine the output or explain why the behavior is not defined by the C language.

## Q1. Array size

```c
int a[10];

printf("%zu\n", sizeof(a) / sizeof(a[0]));
```

## Q2. Pointer size

```c
int a[10];
int *p = a;

printf("%zu %zu\n", sizeof(a), sizeof(p));
```

## Q3. Array parameter

```c
void print_size(int a[])
{
    printf("%zu\n", sizeof(a));
}
```

Why does this not print the full array size?

## Q4. Character array

```c
char s[] = "ARM";

printf("%zu\n", sizeof(s));
printf("%zu\n", strlen(s));
```

## Q5. Static local variable

```c
void test(void)
{
    static int x = 0;
    printf("%d ", ++x);
}
```

Call `test()` three times. Predict the output.

## Q6. Unsigned underflow

```c
unsigned int x = 0;
x--;

printf("%u\n", x);
```

Explain unsigned arithmetic modulo one more than the maximum representable value.

## Q7. Signed and unsigned comparison

```c
int a = -1;
unsigned int b = 1;

printf("%d\n", a < b);
```

Explain the usual arithmetic conversions and the implementation's integer-width assumptions.

## Q8. Short-circuit evaluation

```c
int x = 0;

if (x != 0 && 10 / x > 1)
    printf("A");
else
    printf("B");
```

## Q9. Pointer increment

```c
int a[] = {1, 2, 3};
int *p = a;

printf("%d ", *p++);
printf("%d\n", *p);
```

## Q10. Invalid pointer access

```c
int a[] = {1, 2, 3};
int *p = a + 3;

printf("%d\n", *p);
```

## Q11. Macro expansion

```c
#define SQUARE(x) x * x

printf("%d\n", SQUARE(2 + 3));
```

Explain the macro expansion and fix the definition.

## Q12. Evaluation order

```c
int i = 1;
printf("%d %d\n", i++, i++);
```

Explain why you must not rely on a particular order or result.

## Q13. Const pointer

```c
int x = 10;
int y = 20;

int *const p = &x;
*p = 30;

/* p = &y; */
```

Which statements are valid?

## Q14. Pointer to const

```c
int x = 10;
const int *p = &x;
```

Can `p` point to `y`? Can `*p` be assigned a new value through this pointer?

## Q15. Signed shift

```c
int x = -8;
printf("%d\n", x >> 1);
```

Discuss the implementation-defined behavior.

---

# 16. Multiple-Choice Questions

## Q1. What does `volatile` guarantee?

A. Atomic access  
B. Thread safety  
C. Compiler handling of volatile accesses as specified by the language  
D. Automatic placement in RAM

## Q2. What is the usual result of `x & (x - 1)`?

A. Sets the lowest set bit  
B. Clears the lowest set bit  
C. Reverses all bits  
D. Shifts the value right

## Q3. What does `malloc()` return on failure?

A. An address of zero-filled memory  
B. A pointer to static memory  
C. A null pointer  
D. An uninitialized pointer that must be freed

## Q4. What is the main difference between `const int *p` and `int *const p`?

A. They are identical  
B. The first prevents reassignment of `p`  
C. The second prevents reassignment of `p`  
D. Both prevent all modifications of the pointed-to object

## Q5. Which statement about arrays is correct?

A. Arrays and pointers are identical types  
B. Arrays can always be assigned to other arrays  
C. An array expression often converts to a pointer to its first element  
D. `sizeof` always reports pointer size for an array

## Q6. Which operation is undefined behavior?

A. Comparing two valid pointers to the same object  
B. Dereferencing a null pointer  
C. Computing `sizeof(int)`  
D. Calling `free(NULL)`

## Q7. Which statement about `free()` is correct?

A. It sets the pointer variable to NULL automatically  
B. It makes the allocated object's lifetime end  
C. It can safely be called twice on the same allocation  
D. It clears all allocated bytes to zero

## Q8. What does `static` on a file-scope object generally provide?

A. Automatic storage duration  
B. Internal linkage  
C. Thread synchronization  
D. Dynamic allocation

## Q9. Why is `volatile` commonly used for hardware registers?

A. It guarantees atomic access  
B. It prevents all CPU reordering  
C. It requires the compiler to respect volatile accesses as specified  
D. It guarantees the hardware register is valid

## Q10. Which is the best description of undefined behavior?

A. The compiler must always print an error  
B. The program always crashes  
C. The C standard imposes no requirements on the behavior  
D. The result is always zero

---

# 17. Timed Mock Assessment — 20 Minutes

This mock is a practice exercise, not a reproduction of Arm's actual test.

## Part A — C fundamentals (5 minutes)

### Question 1

What is the difference between:

```c
const int *p;
```

and:

```c
int *const p;
```

### Question 2

What is the output?

```c
unsigned int x = 8;
printf("%u\n", x & (x - 1));
```

### Question 3

Why does `sizeof(arr)` produce a different kind of result from `sizeof(ptr)`?

### Question 4

What is the difference between `volatile` and `_Atomic`?

## Part B — Coding (10 minutes)

### Question 5: Closest to zero

Implement:

```c
#include <stddef.h>

int closest_to_zero(const int *a, size_t n);
```

Requirements:

- Return `0` when the array is empty.
- Return the element closest to zero.
- If two elements have the same absolute value, prefer the positive element.
- Avoid undefined behavior when comparing extreme integer values.
- Document the behavior when the input pointer is NULL.

### Question 6: Count set bits

Implement:

```c
#include <stdint.h>

unsigned int count_bits(uint32_t x);
```

Return the number of set bits in a 32-bit unsigned integer.

## Part C — Debugging (5 minutes)

### Question 7

Identify the bug:

```c
int *allocate_value(void)
{
    int x = 42;
    return &x;
}
```

### Question 8

Explain why the following code is unsafe:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *p = malloc(sizeof(*p));

    if (p != NULL) {
        *p = 10;
        free(p);
    }

    printf("%d\n", *p);
    return 0;
}
```

### Question 9

What should you do if a program passes tests with no optimization but fails with compiler optimizations enabled?

---

# 18. Final Revision Checklist

Before taking the assessment, confirm that you can:

- [ ] Write a C function without syntax errors.
- [ ] Use pointers and arrays correctly.
- [ ] Reverse a string in place.
- [ ] Find the second-largest distinct element.
- [ ] Solve the closest-to-zero problem.
- [ ] Set, clear, toggle and test bits.
- [ ] Count set bits efficiently.
- [ ] Explain `const`, `volatile`, `static` and `extern`.
- [ ] Distinguish arrays from pointers.
- [ ] Explain `sizeof` and string length.
- [ ] Identify a dangling pointer and a memory leak.
- [ ] Recognize undefined behavior.
- [ ] Handle empty input and boundary conditions.
- [ ] Compile and run code using the available assessment environment.
- [ ] Explain the time and space complexity of a solution.

## Recommended priority for the 20-minute screening

1. Pointers and strings
2. Arrays and basic algorithms
3. Bit manipulation
4. C fundamentals and output prediction
5. Memory safety and undefined behavior
6. Embedded-specific topics

Advanced concurrency, lock-free algorithms and kernel internals are useful for the broader Senior Software Engineer role, but they should not displace practice on core C fundamentals.

**Preparation rule:** Write the code yourself, compile it, test normal and boundary inputs, and explain why it works. Recognizing a solution is not the same as being able to produce one under a 20-minute time limit.
