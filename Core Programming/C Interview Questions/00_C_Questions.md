# C Interview Questions for 10+ Years Experience

> **Only Core C Language — Structured by Levels**  
> Targeted for Embedded / Application Developer roles  
> Each section includes follow-up/counter questions to probe depth

---

## BASIC LEVEL

### Data Types & Variables

1. What are basic data types in C?
2. Difference between `signed` and `unsigned`.
3. Size of: `char`, `short`, `int`, `long`, `long long`, `float`, `double`
   - Follow-up: What determines datatype size? Is `int` always 4 bytes?
   - Follow-up: What is `stdint.h`? Why use `uint32_t` instead of `unsigned int` in embedded?
4. Difference between declaration and definition.
   - Follow-up: Can you declare a variable multiple times? Define multiple times?
5. Difference between local and global variables.
   - Follow-up: Where is each stored in memory?
   - Follow-up: What is the default value of an uninitialized local variable vs global variable?
6. Scope and lifetime of variables.
7. What are storage classes in C?
8. Difference between `auto`, `register`, `static`, `extern`.
   - Follow-up: What is the scope and lifetime of each?
   - Follow-up: Is `register` still relevant today? Can you take address of a `register` variable?
   - Follow-up: What happens if you declare `extern` variable without defining it anywhere?

### Operators

9. Difference between `i++` and `++i`.
   - Follow-up: Which is more efficient? Does it matter in C?
10. Difference between `=` and `==`.
11. What is short-circuit evaluation?
    - Follow-up: Give a practical use case where short-circuit prevents a crash.
12. Difference between logical and bitwise operators.
    - Follow-up: What is `!!x`? Why is it used?
13. What is operator precedence?
14. What is associativity?
    - Follow-up: What is the output of `a = b = c = 5;`? Why?

### Control Statements

15. Difference between `while` and `do-while`.
    - Follow-up: When is `do-while` preferred? Give embedded use case.
16. Difference between `break` and `continue`.
17. Difference between `goto` and function call.
    - Follow-up: Problems with `goto`. When is `goto` acceptable (e.g., error cleanup in Linux kernel)?

### Arrays

18. What is an array?
19. How is an array stored in memory?
20. Why does array index start from 0?
    - Follow-up: Explain in terms of pointer arithmetic.
21. Can array size change dynamically?
    - Follow-up: What is VLA? Is it safe to use in embedded?
22. Difference between `arr` and `&arr`.
    - Follow-up: What is the type of each? What does pointer arithmetic give for each?
23. What happens on out-of-bounds access?
    - Follow-up: Why doesn't C check bounds? What is buffer overflow?
24. Multi-dimensional arrays memory layout.
    - Follow-up: Row-major order in C. How is `a[i][j]` calculated?

### Strings

25. Difference between `char str[] = "Hello"` and `char *str = "Hello"`.
    - Follow-up: Where is the string literal stored?
    - Follow-up: What happens if you modify `str[0]` in each case?
26. Difference between `strlen()` and `sizeof()`.
27. Why strings end with `'\0'`?
    - Follow-up: What happens if `'\0'` is missing?
28. Difference between `strcpy` and `strncpy`.
    - Follow-up: Does `strncpy` always null-terminate?
29. Common string vulnerabilities.
    - Follow-up: What is buffer overflow via `gets()`? Why was it removed in C11?

### Functions

30. What is function prototype? Why is function declaration needed?
31. Difference between call by value and call by reference.
    - Follow-up: Does C have true call by reference? How does it simulate it?
32. How are arrays passed to functions?
    - Follow-up: Does the function know the array size? Why not?
33. What is recursion?
    - Follow-up: What happens in infinite recursion?
    - Follow-up: Stack memory behavior in recursion.
    - Follow-up: What is tail recursion? Can the compiler optimize it?
34. Advantages/disadvantages of recursion.
    - Follow-up: Why is recursion risky in embedded systems?
35. What are inline functions?
    - Follow-up: Is `inline` a guarantee? What does the compiler actually do?

### Pointers

36. What is a pointer? Why are pointers needed?
37. What is NULL pointer?
    - Follow-up: Is NULL always 0? What does the standard say?
38. What is a dangling pointer?
    - Follow-up: How do you avoid it?
39. What is a wild pointer?
40. What is a void pointer?
    - Follow-up: Can you dereference a void pointer? Can you do arithmetic on it?
41. Pointer arithmetic. Why is pointer arithmetic datatype dependent?
42. Difference between `*p++` and `(*p)++`.
43. Difference between `int *p` and `int (*p)[10]`.
44. Double pointers (pointer to pointer).
    - Follow-up: When do you need double pointers? (e.g., modifying a pointer inside a function)
45. Pointer to function.
46. Array of pointers vs pointer to array.

### Dynamic Memory Allocation

47. What is static memory allocation?
48. What is dynamic memory allocation?
49. Difference between stack and heap.
    - Follow-up: Which is faster? Why?
50. Explain `malloc`, `calloc`, `realloc`, `free`.
51. Difference between `malloc` and `calloc`.
52. What happens when `malloc` fails?
    - Follow-up: How should you handle it in embedded systems?
53. What is a memory leak?
    - Follow-up: How do you detect it?
54. Double free problem.
    - Follow-up: What actually happens internally?
55. Use-after-free issue.
56. Heap fragmentation.
    - Follow-up: How do you handle it in long-running embedded systems?

### Structures & Unions

57. What is a structure?
58. What is a union?
59. Difference between structure and union.
    - Follow-up: Give a real embedded use case for union (e.g., register overlay, protocol parsing).
60. Why is union memory shared?
61. Nested structures.
62. Array inside structure.
63. Pointer to structure.
64. Structure padding and alignment.
    - Follow-up: How is structure size calculated?
    - Follow-up: Calculate size:
    ```c
    struct test {
        char a;
        int b;
        char c;
    };
    ```
    - Follow-up: How to disable padding? (`#pragma pack`, `__attribute__((packed))`)
    - Follow-up: Why is padding important for performance?
65. Bit fields in structures.
    - Follow-up: Is bit-field ordering guaranteed across compilers?
    - Follow-up: When should you NOT use bit-fields?

### Typecasting

66. Implicit typecasting (type promotion).
67. Explicit typecasting.
68. Data truncation problem.
69. Why is typecasting dangerous sometimes?
    - Follow-up: What are the dangers of casting `int*` to `char*`?

---

## INTERMEDIATE LEVEL

### Memory Layout

1. Explain memory layout of a C program:
   - Text segment
   - Data segment (initialized)
   - BSS segment (uninitialized)
   - Heap
   - Stack
   - Follow-up: Where are global variables, static variables, string literals, local variables stored?
   - Follow-up: Difference between stack overflow and heap overflow.
   - Follow-up: What grows upward and what grows downward?

### Compilation Process

2. Explain the full compilation process:
   - Preprocessing
   - Compilation
   - Assembly
   - Linking
   - Follow-up: What is an object file?
   - Follow-up: What is an executable?
   - Follow-up: What is a symbol table?
   - Follow-up: Static vs dynamic linking — pros and cons.
   - Follow-up: Role of the linker. What errors does the linker catch?
   - Follow-up: Difference between `.o` file and `.a`/`.lib` file.

### Preprocessor & Macros

3. What is the preprocessor?
4. What is a macro? Function-like macros. Multi-line macros.
   - Follow-up: What is the problem with `#define SQUARE(x) x*x`? How to fix?
   - Follow-up: What are `#` (stringification) and `##` (token pasting)?
5. Macro vs inline function — pros and cons.
6. Macro vs `const` — when to use which?
   - Follow-up: Can `const` replace macro in all cases?
7. Advantages/disadvantages of macros.
8. Conditional compilation (`#ifdef`, `#ifndef`, `#if`, `#elif`, `#else`, `#endif`).
   - Follow-up: How is it used for platform-specific code?
9. Why are macros dangerous?
   - Follow-up: What are variadic macros (`__VA_ARGS__`)?

### Header Files

10. Why are header files used?
11. What is a header guard?
    - Follow-up: `#ifndef` vs `#pragma once` — pros and cons.
12. What happens without a header guard?
13. Difference between `#include "a.h"` and `#include <a.h>`.

### Static Keyword

14. Static local variable.
15. Static global variable.
16. Static function.
    - Follow-up: What is the lifetime of a static variable?
    - Follow-up: Why is `static` used in embedded systems?
    - Follow-up: How does `static` help in encapsulation in C?
    - Follow-up: "You said `static` limits scope to a file — how exactly does the linker enforce this?"

### Const & Volatile

17. What is `const`?
18. What is `volatile`?
    - Follow-up: Why is `volatile` important for hardware registers?
    - Follow-up: Why is `volatile` NOT enough for thread safety?
19. Difference between:
    ```c
    const int *p;      // pointer to const int
    int *const p;      // const pointer to int
    const int *const p; // const pointer to const int
    ```
20. Can a `const` variable be modified? (via pointer casting — UB)
21. Can a variable be both `const` and `volatile`? Give an example.
    - Follow-up: "Show me the assembly difference with and without `volatile`."

### Advanced Pointers

22. Function pointers.
    - Follow-up: How do you implement a callback mechanism in C?
    - Follow-up: How do you pass context/user data along with callback?
23. Difference between `int (*fp)(int)` and `int *fp(int)`.
24. Self-referential structures (linked lists).
25. Pointer aliasing.
26. Strict aliasing rule.
    - Follow-up: What optimization does the compiler do based on strict aliasing?
    - Follow-up: How can violating it cause bugs?
27. Pointer arithmetic on `void*` — standard vs GCC extension.

### File Handling

28. File opening modes (`r`, `w`, `a`, `r+`, `w+`, `a+`, `rb`, `wb`).
29. Difference between text and binary files.
30. `fread` vs `fscanf`. `fwrite` vs `fprintf`.
31. `fseek`, `ftell`, `rewind`.
32. Buffering in file handling (`setbuf`, `setvbuf`).

### Bitwise Operations

33. How do you set, clear, toggle, and check a bit?
    - Follow-up: Write a macro to set the nth bit.
    - Follow-up: How do you implement a bitmask-based flag system?
34. How do you count set bits in an integer?
35. Swap two numbers without temporary variable using XOR.
    - Follow-up: Why does XOR swap fail when both variables point to same memory?

### Endianness

36. What is endianness? Big-endian vs little-endian.
    - Follow-up: Write code to detect endianness at runtime.
    - Follow-up: How do you convert between big-endian and little-endian?
    - Follow-up: "You mentioned struct padding — what if I need to send this struct over UART byte by byte?"

---

## ADVANCED LEVEL

### Memory & Performance

1. Alignment requirements on different architectures.
   - Follow-up: What happens on unaligned access on Cortex-M0 vs Cortex-M4?
2. Packed structures — when and why.
   - Follow-up: Performance penalty of packed structures.
3. Cache line alignment.
   - Follow-up: What is false sharing? How do you avoid it?
4. Heap fragmentation handling.
   - Follow-up: Memory pool allocator design.
5. Stack optimization techniques.
6. Custom memory allocator design.
   - Follow-up: First-fit, best-fit, pool allocator strategies.
   - Follow-up: Why is dynamic allocation avoided in safety-critical embedded systems?

### Undefined Behavior

7. What is undefined behavior?
8. Give 5+ examples of UB.
   ```c
   i = i++;                    // UB
   int *p = NULL; *p = 5;     // UB
   int a[5]; a[10] = 1;       // UB
   signed overflow             // UB
   use after free              // UB
   ```
9. Why does the compiler behave unpredictably with UB?
10. What are sequence points?
11. Difference between UB, unspecified behavior, and implementation-defined behavior.
    - Follow-up: How do you write defensive code to avoid UB?

### Advanced Compilation

12. Compiler optimization levels (`-O0`, `-O1`, `-O2`, `-O3`, `-Os`).
13. Dead code elimination.
14. Loop unrolling.
15. Inline expansion.
16. Link-time optimization (LTO).
17. Symbol visibility (`__attribute__((visibility("hidden")))`, `static`).

### Concurrency Concepts in C

18. Race condition.
19. Critical section.
20. Atomic operation.
    - Follow-up: What are `_Atomic` types in C11?
21. Reentrancy.
    - Follow-up: Difference between re-entrant and thread-safe.
    - Follow-up: Is `printf` re-entrant? Why not?
    - Follow-up: How do you make a function re-entrant?
22. What are memory barriers and why are they needed?
    - Follow-up: Compiler barrier vs hardware barrier.

### Advanced Structures

23. Flexible array member.
    ```c
    struct packet {
        int length;
        char data[];  // flexible array member
    };
    ```
24. Anonymous structures/unions (C11).
25. `offsetof` macro — how does it work internally?
26. `container_of` macro (Linux kernel).
    - Follow-up: How can you use it for intrusive linked lists?

### Function Pointer Advanced

27. State machine using function pointers.
    - Follow-up: Function pointer table approach vs switch-case — pros and cons.
28. Jump tables / dispatch tables.
29. Callback architecture.
30. Plugin architecture in C.

### Linker Scripts & Sections

31. What are linker scripts? What do they control?
    - Follow-up: How do you place a variable/function at a specific memory address?
    - Follow-up: What are `.text`, `.data`, `.bss`, `.rodata` sections?
32. What are weak symbols and strong symbols?
    - Follow-up: What is `__attribute__((weak))`? Use case in embedded?
    - Follow-up: How do you override a default handler?

### Interrupt Handling in C

33. How do you handle interrupt service routines (ISR) in C?
    - Follow-up: What should you NOT do inside an ISR?
    - Follow-up: How do you share data between ISR and main loop safely?
    - Follow-up: What is interrupt latency and how do you minimize it?
    - Follow-up: "You said ISR should be short — what if you need to process a large buffer received via DMA?"

---

## EXPERT LEVEL

### Expert Memory Questions

1. How does `malloc` internally work? (`sbrk`, `mmap`, free list)
   - Follow-up: What is heap metadata? How is it corrupted?
2. Difference between virtual and physical memory.
3. Paging and segmentation.
4. Memory corruption debugging techniques.
5. Detecting stack corruption.
   - Follow-up: How do you implement a stack canary?
   - Follow-up: How do you detect stack overflow at runtime in embedded?
6. Heap metadata corruption.
   - Follow-up: What tools help? (Valgrind, ASan, custom guards)

### Expert Compiler Questions

7. How does the compiler generate an object file?
8. ELF file structure.
   - Follow-up: Sections vs segments.
9. Relocation process.
10. Dynamic loader role.
11. Position Independent Code (PIC).
    - Follow-up: Why is PIC needed for shared libraries?

### Expert Undefined Behavior

12. Strict aliasing optimization issue.
    - Follow-up: How does type-punning via union differ from pointer cast?
13. Integer overflow UB.
    - Follow-up: Why is signed overflow UB but unsigned overflow defined?
14. Dangling reference behavior.
15. Misaligned access issue.
16. Volatile optimization edge cases.
    - Follow-up: When does `volatile` NOT prevent reordering?

### Expert API Design in C

17. How to design a scalable C library?
18. Opaque pointers (PIMPL pattern in C).
    - Follow-up: How does this achieve encapsulation?
19. ABI compatibility.
    - Follow-up: What breaks ABI? How do you maintain backward compatibility?
20. Forward declarations.
21. Encapsulation in C.
22. Error handling design patterns in C.
    - Follow-up: Return codes vs errno vs callback — tradeoffs.

### Expert Debugging Questions

23. Debug a segmentation fault — systematic approach.
24. Analyze a core dump.
25. Detect memory leak without Valgrind.
26. Diagnose a random crash that only occurs in optimized builds.
27. Detect stack overflow without OS support.
28. Debug optimized code — challenges and techniques.

### C Standards

29. Differences between C89, C99, C11, and C17.
    - Follow-up: What features from C99/C11 are commonly used in embedded?
    - Follow-up: What is `_Generic` in C11?
    - Follow-up: What is `_Static_assert`?
30. `restrict` keyword (C99).
    - Follow-up: How does it help compiler optimization?
    - Follow-up: What happens if you violate the `restrict` contract?

### Object-Oriented Design in C

31. How do you implement object-oriented design patterns in C?
    - Follow-up: Polymorphism using function pointers and structs.
    - Follow-up: How does the Linux kernel use this pattern (e.g., `file_operations`)?
    - Follow-up: Inheritance via struct embedding.

### Lock-Free Programming

32. What is a lock-free ring buffer?
    - Follow-up: How do you implement it with `volatile` and memory barriers?
33. Compare-and-swap (CAS) operations.
34. ABA problem.

---

## EXPERT-LEVEL TRICK QUESTIONS

1. What is the output?
   ```c
   printf("%d %d %d", i++, i++, i++);
   ```
   **Answer:** Undefined behavior — unsequenced modification.

2. Difference between:
   ```c
   char *p = "hello";   // pointer to string literal (read-only)
   char p[] = "hello";  // mutable array on stack
   ```

3. Can `main()` return `void`?
   - **Answer:** Not per the standard. It must return `int`.

4. Why is `sizeof(char)` always 1?
   - **Answer:** By definition in the standard. `char` is the unit of measurement.

5. Is NULL always 0?
   - **Answer:** Null pointer constant is `0` or `(void*)0`, but the representation in memory may not be all-zeros (rare architectures).

6. Why is an array name not modifiable (not an lvalue)?

7. Why does `sizeof` not evaluate its expression?
   ```c
   int x = 5;
   printf("%zu", sizeof(x++));  // x is still 5
   ```
   - **Answer:** `sizeof` is evaluated at compile time (except VLAs).

8. Why is `free(NULL)` safe?
   - **Answer:** The standard guarantees it's a no-op.

9. Can `malloc` return the same address again?
   - **Answer:** Yes, after `free()` that memory can be reused.

10. Why is recursion risky in embedded systems?
    - **Answer:** Limited stack, no virtual memory, hard to predict max depth.

11. What is the output?
    ```c
    int a = 1, b = 2;
    int c = a+++b;  // parsed as (a++) + b = 1 + 2 = 3, then a becomes 2
    ```

12. What does this do?
    ```c
    int (*(*fp)(int))[10];
    ```
    **Answer:** `fp` is a pointer to a function taking `int` and returning pointer to array of 10 `int`s.

13. "You said `malloc` returns `void*` — why don't we cast it in C but we must in C++?"

14. "You said `free(NULL)` is safe — what does the standard say about `realloc(NULL, size)`?"
    - **Answer:** Equivalent to `malloc(size)`.

15. What is the difference between `sizeof` and `_Alignof`?

---

## CODING QUESTIONS (Whiteboard / Live)

1. Implement `memcpy`, `memset`, `strlen`, `strcmp` from scratch.
   - Follow-up: Handle overlapping memory in `memcpy` (i.e., implement `memmove`).
2. Reverse a string in-place.
3. Implement a circular buffer (ring buffer) with read/write APIs.
4. Write a bit-manipulation function to count set bits (Brian Kernighan's algorithm).
5. Implement a singly linked list with insert, delete, search, and reverse.
6. Write a generic swap macro that works for any data type.
7. Implement `atoi` (string to integer) with error handling.
8. Write a program to detect endianness and swap bytes of a 32-bit integer.
9. Implement a simple state machine (e.g., traffic light controller, UART protocol parser).
10. Write an ISR-safe FIFO queue using `volatile` and atomic operations.
11. Implement `container_of` macro from scratch.
12. Write a memory pool allocator for fixed-size blocks.
13. Implement a function to detect if a linked list has a cycle (Floyd's algorithm).
14. Write a program to print memory layout of a structure showing padding bytes.

---

## MOST IMPORTANT TOPICS FOR 10+ YEARS C INTERVIEW

If an interviewer sees 10 years experience, they expect **strong** knowledge in:

| Priority | Topic |
|----------|-------|
| ★★★★★ | Memory layout & management |
| ★★★★★ | Pointers (all levels) |
| ★★★★★ | Function pointers & callbacks |
| ★★★★★ | Dynamic memory (malloc/free internals) |
| ★★★★★ | Structures, padding & alignment |
| ★★★★★ | Undefined behavior |
| ★★★★☆ | Compilation & linking process |
| ★★★★☆ | Macros & preprocessor |
| ★★★★☆ | `const` / `volatile` / `static` |
| ★★★★☆ | Optimization awareness |
| ★★★☆☆ | Multithreading basics |
| ★★★★★ | Debugging techniques |
| ★★★★☆ | API design in C |
| ★★★★☆ | Performance optimization |
| ★★★★★ | Memory corruption analysis |
| ★★★★☆ | Compiler behavior understanding |
| ★★★☆☆ | C standards (C99/C11 features) |
| ★★★★☆ | Embedded-specific: ISR, volatile, linker scripts |

---

## DEEP DIVE / COUNTER QUESTIONS

These are follow-up questions an interviewer uses to probe real depth:

1. "You said `static` limits scope to a file — how exactly does the linker enforce this?"
2. "You said `volatile` prevents optimization — show me the assembly difference with and without `volatile`."
3. "You said arrays decay to pointers — does `sizeof` also see the decayed pointer?"
4. "You mentioned structure padding — what if I need to send this struct over UART byte by byte?"
5. "You said `malloc` returns `void*` — why don't we cast it in C but we must in C++?"
6. "You said `const` is better than macro — can `const` replace macro in all cases?"
7. "You said function pointers are used for callbacks — how do you pass context/user data?"
8. "You said `free(NULL)` is safe — what does the standard say about `realloc(NULL, size)`?"
9. "You said ISR should be short — what if you need to process a large buffer received via DMA?"
10. "You said you avoid dynamic allocation in embedded — how do you handle variable-size data?"
11. "You said `sizeof` is compile-time — when is it runtime?"
12. "You said signed overflow is UB — why does the compiler not just wrap it like unsigned?"
13. "You said strict aliasing helps optimization — show me an example where it breaks code."
14. "You said packed structures solve network byte ordering — what is the performance cost?"
15. "You said `inline` is a hint — when does the compiler ignore it?"

---

*Document created for interview preparation — covers Basic to Expert level C programming questions with depth-probing follow-ups.*