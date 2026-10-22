# C Coding Problems for Arm and Silicon Companies
## Screening and Assessment Practice — Easy to Hard

**Target roles:** Software Engineer, Senior Software Engineer, Firmware Engineer, Embedded C Engineer, Systems Software Engineer  
**Companies to prepare for:** Arm, Qualcomm, Intel, AMD, NVIDIA, NXP, Texas Instruments, Infineon, STMicroelectronics, Renesas, Broadcom, MediaTek, Apple Silicon and similar semiconductor/system companies.

> **Important:** These are original practice problems based on common C, embedded, systems and algorithmic screening themes. They are not claimed to be leaked or verified questions from any specific company's private assessment. Actual questions vary by team, level, location and hiring process.

---

# How to use this question bank

For every problem:

1. Clarify assumptions and input constraints.
2. Write a complete C function with a clear interface.
3. Compile with warnings enabled.
4. Test normal, boundary, invalid and overflow cases.
5. State time and auxiliary-space complexity.
6. Explain why the implementation is safe in C.

Suggested local compiler flags:

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic -Wconversion -Wshadow -O2 solution.c -o solution
```

For supported host platforms, sanitizers can help catch errors:

```bash
gcc -std=c11 -Wall -Wextra -g -fsanitize=address,undefined solution.c -o solution
```

Sanitizer availability depends on the compiler and platform. These tools do not detect every possible bug.

---

# Table of Contents

1. Core C Screening Problems
2. Pointer and Memory Problems
3. Bit Manipulation and Register-Level C
4. Embedded and Firmware Problems
5. C Language Traps and Debugging
6. General Coding Problems
7. Multiple-Choice and Output Prediction
8. Company-Style Practice Sets
9. A 20-Minute Screening Simulation
10. Final Preparation Priorities

---

# Part 1 — Core C Screening Problems

These are the first problems to practise for a short C assessment.

## Problem 1. Closest Temperature to Zero

**Difficulty:** Easy–Medium  
**Skills:** Arrays, comparison, edge cases

Implement:

```c
#include <stddef.h>

int closest_to_zero(const int *values, size_t count);
```

Requirements:

- Return the value closest to zero.
- If positive and negative values are equally close, return the positive value.
- Return `0` for an empty array.
- Define and document behavior for a null pointer with nonzero `count`.
- Avoid overflow when comparing magnitudes, especially for `INT_MIN`.

Examples:

| Input | Expected output |
|---|---:|
| `{-8, -2, 4, 7}` | `-2` |
| `{-5, 5}` | `5` |
| `{0, -2, 4}` | `0` |
| `{}` | `0` |

Follow-up: Can you implement this in O(n) time and O(1) extra space?

## Problem 2. Reverse a String In Place

**Difficulty:** Easy  
**Skills:** Pointers, strings, indexing

Implement:

```c
void reverse_string(char *s);
```

Requirements:

- Reverse a modifiable, null-terminated string in place.
- Handle `NULL` safely.
- Do not allocate another string.
- Explain why passing a string literal is not valid for modification.

Examples:

- `"ARM"` → `"MRA"`
- `""` → `""`
- `"A"` → `"A"`

Follow-up: Implement the function using two pointers instead of indices.

## Problem 3. Count Set Bits

**Difficulty:** Easy–Medium  
**Skills:** Bit manipulation, unsigned arithmetic

Implement:

```c
#include <stdint.h>

unsigned int count_set_bits(uint32_t value);
```

Examples:

- `0` → `0`
- `13` (`1101b`) → `3`
- `255` → `8`

Follow-up: Compare a loop over all 32 bit positions with Brian Kernighan's algorithm.

## Problem 4. Set, Clear, Toggle and Read a Bit

**Difficulty:** Easy–Medium  
**Skills:** Bit masks, integer widths

Implement four functions that set, clear, toggle and read bit `n` of a `uint32_t`.

Requirements:

- Reject or otherwise handle `n >= 32`.
- Do not shift by an invalid amount.
- Preserve all unrelated bits.
- Explain why unsigned types are appropriate.

## Problem 5. Second-Largest Distinct Element

**Difficulty:** Easy–Medium  
**Skills:** Arrays, edge cases

Implement a function that returns the second-largest **distinct** value.

Examples:

- `{5, 2, 9, 7, 9}` → `7`
- `{4, 4, 4}` → no second distinct value
- `{-8, -2, -5}` → `-5`

Choose a return convention, such as `bool` plus an output parameter. Do not use a sentinel that could also be a valid answer.

Follow-up: Achieve O(n) time and O(1) auxiliary space.

## Problem 6. Find the Missing Number

**Difficulty:** Easy–Medium  
**Skills:** Arrays, arithmetic, XOR

An array contains `n` distinct values from the range `0` through `n`, with one value missing. Find the missing value.

Examples:

- `{3, 0, 1}` → `2`
- `{0, 1}` → `2`
- `{1}` → `0`

Follow-ups:

- Solve using XOR.
- Solve using a sum formula.
- Explain overflow risks and how to avoid them.

## Problem 7. Binary Search

**Difficulty:** Easy–Medium  
**Skills:** Arrays, algorithms

Implement iterative binary search over a sorted integer array.

Requirements:

- Return the index if found.
- Return a documented not-found result otherwise.
- Avoid overflow in midpoint calculation.
- Explain the sorted-input requirement.

Follow-up: Find the first occurrence when duplicates exist.

## Problem 8. Find the First Non-Repeating Character

**Difficulty:** Medium  
**Skills:** Strings, counting

Given a string, return the first character that appears exactly once.

Example: `"swiss"` → `'w'`

State whether the input is ASCII, extended bytes or Unicode. For a basic C exercise, assume byte-oriented ASCII input.

Follow-up: Implement without a large frequency array.

---

# Part 2 — Pointer and Memory Problems

These are important for low-level firmware and systems programming.

## Problem 9. Implement `strlen()`

Implement a custom version of `strlen()`.

Requirements:

- Use pointer traversal.
- Do not access memory beyond the terminating null character.
- Explain why the function requires a valid null-terminated string.

## Problem 10. Implement `strcmp()`

Implement a custom version of `strcmp()`.

Requirements:

- Compare as `unsigned char` values.
- Return a negative value, zero or a positive value according to lexical ordering.
- Do not assume the result must be exactly `-1`, `0` or `1`.

## Problem 11. Safe Bounded String Copy

Design a function that copies a string into a destination buffer of known capacity.

Requirements:

- Never write past the destination capacity.
- Specify whether the destination is null-terminated on truncation.
- Handle zero capacity and null pointers according to a documented contract.
- Return enough information for the caller to detect truncation.

## Problem 12. Swap Two Integers Using Pointers

Implement:

```c
void swap_int(int *a, int *b);
```

Discuss what should happen if either pointer is null or both point to the same object.

## Problem 13. Pointer-Based Array Reversal

Reverse an integer array using two pointers.

Requirements:

- O(n) time.
- O(1) auxiliary space.
- Handle empty and one-element arrays.
- Do not dereference an invalid or one-past-the-end pointer.

## Problem 14. Safe Dynamic Array Allocation

Write a function that allocates `n` integers and returns the result through an output pointer.

Requirements:

- Detect allocation failure.
- Check whether `n * sizeof(int)` can overflow `size_t`.
- Define ownership: who must call `free()`?
- Do not lose an existing allocation when resizing fails.

## Problem 15. Find and Fix a Use-After-Free

Identify and correct the bug:

```c
int *p = malloc(sizeof(*p));

if (p != NULL) {
    *p = 42;
    free(p);
    printf("%d\n", *p);
}
```

Explain why assigning `NULL` to `p` after freeing it can help prevent accidental reuse, but does not fix other aliases to the allocation.

## Problem 16. Linked-List Cycle Detection

Implement Floyd's tortoise-and-hare algorithm to detect a cycle in a singly linked list.

Follow-ups:

- Find the cycle's entry node.
- Find the cycle length.
- Explain time and space complexity.
- State assumptions about valid node pointers.

## Problem 17. Reverse a Linked List

Reverse a singly linked list iteratively.

Requirements:

- O(n) time and O(1) auxiliary space.
- Handle an empty list.
- Explain how to avoid losing the remainder of the list while changing links.

## Problem 18. Remove a Node from a Linked List

Remove the first node containing a target value.

Requirements:

- Handle removal of the head.
- Handle a missing target.
- Free memory only if the function owns the node allocation.
- Define how the caller receives the updated head pointer.

---

# Part 3 — Bit Manipulation and Register-Level C

## Problem 19. Power of Two

Determine whether a `uint32_t` value is a power of two.

Examples:

- `1` → true
- `8` → true
- `0` → false
- `12` → false

Follow-up: Explain why zero must be handled separately.

## Problem 20. Find the Lowest Set Bit

Given a nonzero unsigned integer, return the value of its lowest set bit.

Example: `value = 40` (`101000b`) → `8` (`001000b`).

Define the result for zero.

## Problem 21. Reverse Bits

Reverse all 32 bits of a `uint32_t`.

Follow-up: Compare a simple 32-iteration loop with a lookup-table approach.

## Problem 22. Extract a Register Field

Implement a function that extracts `width` bits beginning at `position` from a 32-bit register value.

Requirements:

- Validate `position` and `width`.
- Handle extraction of the full 32-bit width without shifting by 32.
- Use unsigned types.
- Define how invalid parameters are reported.

## Problem 23. Insert a Register Field

Insert a field into a 32-bit register value without changing other bits.

Requirements:

- Reject a field value that does not fit in `width` bits, or document truncation.
- Avoid invalid shift counts.
- Explain the mask-generation edge cases.

## Problem 24. Align an Address Upwards

Given an unsigned address and a nonzero power-of-two alignment, calculate the next aligned address.

Requirements:

- Detect invalid alignment.
- Check arithmetic overflow.
- Explain why the common expression `(x + alignment - 1) & ~(alignment - 1)` needs validation.

## Problem 25. Detect Endianness

Write a small C program that detects byte order on a host system.

Follow-up: Explain why production network and file formats should use explicit byte serialization instead of assuming host endianness.

## Problem 26. Count Bit Differences

Given two `uint32_t` values, count how many bit positions differ.

Example: `a = 10`, `b = 7` → `4`.

Hint: XOR identifies differing bit positions.

## Problem 27. Unique Number in an Array

Every element appears twice except one. Find the unique element using XOR.

Follow-ups:

- What if every other value appears three times?
- What if two values appear once and all others appear twice?
- What assumptions about integer representation and input multiplicities are required?

## Problem 28. Circular Bit Rotation

Implement rotate-left and rotate-right operations for a 32-bit unsigned integer.

Requirements:

- Handle a rotation count of zero.
- Normalize counts greater than 31.
- Avoid shifting by the full width.

---

# Part 4 — Embedded and Firmware Problems

## Problem 29. Circular Buffer

Implement a fixed-size byte ring buffer with:

```c
bool ring_push(struct ring_buffer *rb, uint8_t value);
bool ring_pop(struct ring_buffer *rb, uint8_t *value);
```

Define the structure and implement initialization, full detection and empty detection.

Discuss:

- Whether one slot is reserved.
- How overflow is reported.
- Whether the implementation is thread-safe.
- How an ISR-to-task use case changes the design.

## Problem 30. UART Receive Buffer

Design a UART receive buffer where an interrupt handler writes bytes and a task reads them.

Discuss:

- Producer and consumer ownership.
- Buffer overflow policy.
- Atomicity of index updates on the target.
- Memory ordering and critical sections.
- Why `volatile` alone is not a general synchronization solution.

## Problem 31. Register Read-Modify-Write

Set bit 3 in a memory-mapped register without changing other bits.

Explain the assumptions required for read-modify-write to be safe.

Follow-up: How does the solution change for write-one-to-clear status registers?

## Problem 32. Wrap-Safe Timer Timeout

Implement a timeout check using an unsigned tick counter that wraps around.

Requirements:

- Define the tick-counter width.
- Use modular subtraction appropriately.
- State the maximum supported timeout interval.
- Avoid comparing wrapped counters using naïve greater-than logic.

## Problem 33. Debounce a Digital Input

Design a function that filters a noisy button input.

Requirements:

- Use a periodic tick or timestamp.
- Avoid blocking delays.
- Explain the difference between time-based debounce and requiring N identical samples.

## Problem 34. Parse a Binary Packet

Implement a parser for a byte buffer containing:

- A fixed header
- A length field
- A payload
- A checksum

Requirements:

- Validate the buffer length before every read.
- Handle byte order explicitly.
- Reject oversized payload lengths.
- Avoid unaligned typed-pointer casts.
- Do not assume the incoming data is trustworthy.

## Problem 35. Checksum and CRC

Implement an 8-bit additive checksum.

Follow-up: Explain why a CRC detects many error patterns that a simple sum may miss.

## Problem 36. DMA Buffer Design

Describe how you would design a DMA receive buffer for a cache-enabled MCU or application processor.

Discuss:

- Alignment.
- Cache coherency.
- Ownership transitions.
- Buffer length and bounds checks.
- Completion interrupts and error handling.

This is a design question; exact steps depend on the processor, DMA engine and cache architecture.

## Problem 37. Safe MMIO Access

Design a register-access layer for a peripheral.

Discuss:

- Fixed-width integer types.
- `volatile` access.
- Register access width.
- Read side effects.
- Write-one-to-clear fields.
- Hardware barriers where required by the architecture.
- Why portable C alone does not specify every hardware ordering requirement.

## Problem 38. ISR-to-Task Event Transfer

Design a minimal event queue for an interrupt handler and a task.

State whether there is one producer and one consumer. Explain how the design changes with multiple producers or consumers.

---

# Part 5 — C Language Traps and Debugging

## Problem 39. Signed Overflow

Explain why this is not a portable way to detect signed overflow:

```c
int result = a + b;

if (result < a) {
    /* overflow? */
}
```

Design a safe addition check for signed integers.

## Problem 40. Signed/Unsigned Comparison

Predict and explain:

```c
int a = -1;
unsigned int b = 1;

printf("%d\n", a < b);
```

Explain the usual arithmetic conversions and why mixed signed/unsigned comparisons deserve caution.

## Problem 41. Macro Side Effects

Review:

```c
#define SQUARE(x) ((x) * (x))

int result = SQUARE(i++);
```

Explain why the macro evaluates its argument more than once and propose a safer alternative for a C project.

## Problem 42. Array Parameter Size

Explain why this function cannot determine the caller's array length using `sizeof`:

```c
void process(int values[])
{
    printf("%zu\n", sizeof(values));
}
```

Redesign the function interface.

## Problem 43. Structure Padding

Given:

```c
struct Example {
    char a;
    int b;
    char c;
};
```

Explain why the structure may contain padding and why its exact size is implementation-dependent.

Reorder the fields to reduce padding where possible.

## Problem 44. Strict Aliasing

Explain the risks in:

```c
float f = 1.0f;
int *p = (int *)&f;
printf("%d\n", *p);
```

Describe a portable way to copy the object representation into an integer object when the chosen integer type is large enough, and explain why the resulting bit pattern is still representation-dependent.

## Problem 45. Optimization-Dependent Bug

A program works with `-O0` but fails with `-O2`.

Create a debugging checklist covering:

- Undefined behavior.
- Uninitialized memory.
- Data races.
- Strict-aliasing violations.
- Incorrect `volatile` assumptions.
- Missing compiler or linker configuration.
- Stack corruption.

## Problem 46. Format Specifier Safety

Find the problem in each line:

```c
size_t n = 100;
printf("%d\n", n);

int x = 10;
printf("%f\n", x);

void *p = NULL;
printf("%p\n", p);
```

Correct the format specifiers and explain variadic argument type requirements.

## Problem 47. Integer Parsing

Implement a parser that converts a decimal string to an integer.

Requirements:

- Handle leading whitespace and optional sign if supported.
- Reject invalid characters according to a documented policy.
- Detect overflow before performing the overflowing operation.
- Define behavior for empty input and null pointers.

## Problem 48. Safe Array Indexing

Implement a function that reads `values[index]` only if the index is valid.

Explain why checking `index <= count` is incorrect when `count` is the number of elements.

---

# Part 6 — General Coding Problems

These problems are not specific to any company, but are useful for technical screening.

## Problem 49. Move Zeros to the End

Input: `{0, 1, 0, 3, 12}`  
Output: `{1, 3, 12, 0, 0}`

Preserve the relative order of nonzero elements.

## Problem 50. Merge Two Sorted Arrays

Merge two sorted arrays into one sorted result.

Explain complexity and whether the output buffer can be reused safely.

## Problem 51. Maximum Subarray Sum

Find the maximum sum of a contiguous subarray.

Discuss all-negative inputs and integer overflow.

## Problem 52. First Missing Positive

Find the smallest missing positive integer in an unsorted array.

Target: O(n) time and O(1) extra space.

## Problem 53. Longest Substring Without Repeating Characters

Find the length of the longest substring without repeated characters.

State the input character-set assumption.

## Problem 54. Merge Intervals

Merge overlapping intervals and return the merged list.

Discuss sorting, allocation and output capacity.

## Problem 55. Find the Majority Element

Find an element occurring more than half the time, if one exists.

Explain Boyer–Moore voting and why a verification pass is needed when existence is not guaranteed.

## Problem 56. Detect a Linked-List Cycle

Use Floyd's algorithm. Then extend the solution to find the cycle entry.

## Problem 57. Implement a Queue Using Two Stacks

Implement enqueue, dequeue and empty operations. Explain amortized complexity.

## Problem 58. Implement a Fixed-Size Memory Pool

Design a memory pool for fixed-size blocks.

Discuss allocation failure, alignment, free-list corruption and concurrency.

## Problem 59. Implement a Simple LRU Cache

Design a cache with O(1) average lookup and update using a hash table and doubly linked list.

Explain ownership and allocation-failure handling.

## Problem 60. Bounded Log Formatting

Design a function that formats a message into a fixed-size output buffer.

Requirements:

- Use a bounded formatting function.
- Handle truncation.
- Avoid overlapping source and destination buffers.
- Define what happens when the destination capacity is zero.

---

# Part 7 — Multiple-Choice and Output Prediction

## Question 61. `volatile`

What does `volatile` guarantee?

A. Atomic access  
B. Thread safety  
C. Compiler handling of volatile accesses as specified by the language  
D. Automatic placement in RAM

## Question 62. Clear the Lowest Set Bit

What does this expression do for an unsigned integer `x`?

```c
x & (x - 1)
```

A. Sets the lowest set bit  
B. Clears the lowest set bit  
C. Reverses all bits  
D. Shifts the value right

## Question 63. Allocation Failure

What does `malloc()` return on failure?

A. An address of zero-filled memory  
B. A pointer to static memory  
C. A null pointer  
D. An uninitialized pointer that must be freed

## Question 64. Pointer to Const

Which statement is correct for `const int *p`?

A. The pointer cannot be reassigned  
B. The pointed-to value cannot be modified through `p`  
C. The pointer is always null  
D. The pointed-to object can never change through any alias

## Question 65. Undefined Behavior

Which operation is undefined behavior?

A. Computing `sizeof(int)`  
B. Calling `free(NULL)`  
C. Dereferencing a null pointer  
D. Comparing an integer with zero

## Question 66. Pointer Increment

Predict the output:

```c
int a[] = {10, 20, 30};
int *p = a;

printf("%d ", *p++);
printf("%d\n", *p);
```

## Question 67. Short-Circuit Evaluation

What is printed?

```c
int x = 0;

if (x != 0 && 10 / x > 1)
    puts("A");
else
    puts("B");
```

## Question 68. Static Local

What is printed after calling `counter()` three times?

```c
void counter(void)
{
    static int n;
    printf("%d ", ++n);
}
```

## Question 69. Array Size

For a local array `int a[10]`, what is the result of:

```c
sizeof(a) / sizeof(a[0])
```

## Question 70. Invalid Shift

Assume `unsigned int` is 32 bits. Explain the problem with:

```c
unsigned int x = 1;
unsigned int y = x << 32;
```

---

# Part 8 — Company-Style Practice Sets

These are suggested practice mixes, not claims about actual company question sets.

## Set A — Arm / CPU Architecture / Systems Software

**Suggested duration:** 20–30 minutes

1. Reverse a string in place using pointers.
2. Count set bits in a 32-bit value.
3. Find the closest temperature to zero with tie-breaking rules.
4. Explain `volatile` versus atomic operations.
5. Diagnose an out-of-bounds access.
6. Explain array decay and `sizeof`.
7. Design a bounded ring buffer.

**Focus:** C correctness, pointers, low-level data representation, boundary handling and systems reasoning.

## Set B — Qualcomm / Mobile SoC / Firmware

**Suggested duration:** 30–45 minutes

1. Extract and insert register bit fields.
2. Implement a circular buffer.
3. Parse a binary packet safely.
4. Find a memory leak in a repeated operation.
5. Design an ISR-to-task data path.
6. Explain DMA/cache coherency assumptions.
7. Diagnose a failure that appears only under optimization.

**Focus:** Embedded C, device interaction, memory safety and concurrency.

## Set C — Intel / AMD / CPU Platform Software

**Suggested duration:** 30–45 minutes

1. Implement binary search and discuss boundary conditions.
2. Find the first and last occurrence of a value in a sorted array.
3. Explain alignment and structure padding.
4. Debug strict-aliasing and undefined-behavior issues.
5. Explain cache-line false sharing.
6. Discuss atomics and memory ordering.
7. Diagnose a segmentation fault or optimization-dependent failure.

**Focus:** Systems programming, data structures, memory model and debugging.

## Set D — NVIDIA / GPU / Performance-Oriented Software

**Suggested duration:** 30–45 minutes

1. Implement a reduction that sums an integer array.
2. Explain integer overflow and accumulation width.
3. Design a bounded producer-consumer queue.
4. Compare O(n²) and O(n log n) solutions for a data-processing task.
5. Explain data alignment and memory access patterns.
6. Identify a data race in shared-state code.
7. Measure and reason about a performance regression.

**Focus:** Algorithmic reasoning, memory access, parallelism and performance analysis. Specific GPU roles may additionally require CUDA or other GPU-specific knowledge.

## Set E — MCU Vendors: NXP / TI / Infineon / ST / Renesas

**Suggested duration:** 30–45 minutes

1. Implement register bit manipulation.
2. Build a fixed-size ring buffer.
3. Implement a wrap-safe timeout.
4. Design an interrupt-driven UART receive path.
5. Explain `volatile`, critical sections and atomics.
6. Diagnose a hard fault.
7. Parse a bounded packet without unaligned accesses.

**Focus:** Bare-metal C, hardware registers, interrupts, timing and defensive coding.

---

# Part 9 — A 20-Minute Screening Simulation

Try this without using notes.

## Minutes 0–4: C fundamentals

1. Explain `const int *p` versus `int *const p`.
2. Predict `sizeof(array) / sizeof(array[0])` for a local array.
3. Explain why `volatile` does not guarantee thread safety.

## Minutes 4–12: Coding

4. Implement `count_set_bits(uint32_t value)`.
5. Implement `closest_to_zero()` with positive-value tie-breaking.
6. Explain how your solution handles empty input and `INT_MIN`.

## Minutes 12–17: Pointer and memory safety

7. Find the bug in a function returning the address of a local variable.
8. Explain use-after-free.
9. Explain why dereferencing a one-past-the-end pointer is invalid.

## Minutes 17–20: Review

10. Compile mentally or with a compiler.
11. Check return values and boundary conditions.
12. State time and auxiliary-space complexity.
13. Remove undefined behavior and unjustified assumptions.

---

# Part 10 — Final Preparation Priorities

For a short C screening, practise in this order:

1. **Strings and pointers:** reversal, `strlen`, `strcmp`, pointer arithmetic.
2. **Arrays:** closest to zero, second-largest distinct value, missing number, binary search.
3. **Bit manipulation:** set/clear/toggle/test, count bits, power of two, bit fields.
4. **C semantics:** `sizeof`, `const`, `volatile`, integer conversions, storage duration.
5. **Memory safety:** bounds, allocation failure, dangling pointers, leaks, overflow.
6. **Embedded C:** register access, ring buffers, ISR/task data transfer, wrap-safe timeouts.
7. **Harder systems topics:** atomics, memory ordering, alignment, strict aliasing and optimization-dependent bugs.

## Assessment discipline

- Read the complete problem statement before coding.
- Ask what should happen for invalid or empty input.
- Avoid clever tricks when a straightforward solution is safer.
- Never assume that passing visible examples proves correctness.
- Compile with warnings enabled and inspect every warning.
- Do not use outside assistance during the actual assessment if the assessment rules prohibit it.

**Best next step:** Solve Problems 1–8, then Problems 19–24, under a timer. These give you a concentrated practice set for the C fundamentals most relevant to short screening tests.
