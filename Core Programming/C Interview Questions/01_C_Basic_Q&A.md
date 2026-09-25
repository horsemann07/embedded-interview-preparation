# C Interview Questions — Basic Level Q&A

> **Target:** Entry to Intermediate Level C / Embedded / Application Developer
> **Focus:** Core C fundamentals with interview-level explanations, examples, and practical scenarios

---

## DATA TYPES & VARIABLES

# 1. What are basic data types in C?

## Answer

C provides several fundamental data types:

```text
char
int
float
double
void
```

### Integer types with different signedness and sizes:

```c
char
signed char
unsigned char

short
unsigned short

int
unsigned int

long
unsigned long

long long
unsigned long long
```

### Floating-point types:

```c
float
double
long double
```

### Example:

```c
char c = 'A';
int count = 100;
float temperature = 25.5f;
double voltage = 3.141592;
```

### Why do we need different types?

The type tells the compiler:

1. How much storage is required.
2. How the bits should be interpreted.
3. What operations are valid.
4. What alignment may be required.

### Embedded example:

```c
uint32_t timeout_ms;
uint16_t adc_value;
uint8_t  status;
```

The programmer is communicating the intended data width clearly.

---

# 2. Difference between `signed` and `unsigned`

## Answer

The main difference is how the value range is represented.

### Unsigned integer (N value bits):

```text
0 to 2^N - 1
```

### Signed integer (two's complement):

```text
-2^(N-1) to 2^(N-1)-1
```

### Example with 8-bit types:

```text
unsigned 8-bit:  0 to 255
signed 8-bit:    -128 to +127
```

### Code example:

```c
unsigned char u = 255;
signed char s = 127;
```

### Unsigned wraparound

Unsigned arithmetic is performed modulo one more than the maximum representable value:

```c
uint8_t x = 255;
x++;    // Result is 0
```

### Signed overflow

Signed integer overflow is **undefined behavior** in C:

```c
int x = INT_MAX;
x++;        /* Undefined behavior */
```

### Embedded example:

Unsigned types are often useful for:

```c
uint32_t hardware_register;
uint16_t adc_value;
uint8_t packet_length;
```

because these often represent bit patterns or non-negative quantities.

### Expert answer

> **Unsigned arithmetic has well-defined modulo behavior, while signed overflow is undefined behavior. Signedness also affects comparisons, conversions, shifts, and the interpretation of values, so mixing signed and unsigned types carelessly can create subtle bugs.**

---

# 3. Size of: `char`, `short`, `int`, `long`, `long long`, `float`, `double`

## Answer

A common mistake in interviews is assuming fixed sizes. **They are NOT universally guaranteed by C.**

### Minimum requirements in typical systems:

```text
char       1 byte
short      2 bytes
int        4 bytes
long       4 or 8 bytes
long long  8 bytes
float      commonly 4 bytes (32-bit)
double     commonly 8 bytes (64-bit)
```

### To verify on your system:

```c
#include <stdio.h>

int main(void)
{
    printf("char      = %zu\n", sizeof(char));
    printf("short     = %zu\n", sizeof(short));
    printf("int       = %zu\n", sizeof(int));
    printf("long      = %zu\n", sizeof(long));
    printf("long long = %zu\n", sizeof(long long));
    printf("float     = %zu\n", sizeof(float));
    printf("double    = %zu\n", sizeof(double));
    return 0;
}
```

## Is `int` always 4 bytes?

### Answer

No. C does not require `sizeof(int) == 4`. The implementation decides the representation subject to the standard's requirements.

## What determines datatype size?

### Answer

It depends on:

```text
C implementation
        |
        +--> compiler
        |
        +--> target architecture
        |
        +--> ABI
        |
        +--> compiler options
```

### Example ABIs:

```text
LP64 (64-bit):
long      = 8 bytes
pointer   = 8 bytes
int       = 4 bytes
```

## What is `<stdint.h>`?

### Answer

`<stdint.h>` provides integer types with specified exact widths:

```c
#include <stdint.h>

uint8_t  a;      // 8-bit unsigned
uint16_t b;      // 16-bit unsigned
uint32_t c;      // 32-bit unsigned
uint64_t d;      // 64-bit unsigned
```

## Why use `uint32_t` instead of `unsigned int` in embedded?

### Answer

When communicating with hardware, exact width matters:

```c
uint32_t reg;   // Clearly need 32-bit
unsigned int reg;  // Width is unclear
```

For protocols, registers, and binary formats, `<stdint.h>` types make the intended width explicit.

### Expert answer

> **C does not guarantee that `int` is 4 bytes. When exact integer widths matter, especially for protocols, registers, binary formats, and embedded hardware, `<stdint.h>` types such as `uint32_t` make the intended width explicit when that exact-width type is available.**

---

# 4. Difference between declaration and definition

## Answer

**Declaration** tells the compiler that something exists and provides its type.

**Definition** actually defines the object and allocates storage (for objects).

### Variable example:

```c
extern int counter;     // Declaration
```

```c
int counter = 100;      // Definition
```

### Function example:

```c
int add(int a, int b);  // Declaration (prototype)
```

```c
int add(int a, int b)   // Definition
{
    return a + b;
}
```

## Can you declare a variable multiple times?

### Answer

Yes:

```c
extern int counter;
extern int counter;    // Allowed
```

Both declarations are compatible.

## Can you define multiple times?

### Answer

No:

```c
int counter = 10;
int counter = 20;     /* Error - multiple definitions */
```

An object/function generally must have one program-wide definition.

### Typical header/source design:

**counter.h:**

```c
#ifndef COUNTER_H
#define COUNTER_H

extern int counter;

#endif
```

**counter.c:**

```c
#include "counter.h"

int counter = 0;  // Single definition
```

**other.c:**

```c
#include "counter.h"

void update(void)
{
    counter++;
}
```

### Expert answer

> **A declaration introduces a name and its type to the compiler. A definition provides the entity itself—such as storage for an object or the function body. Multiple compatible declarations are allowed, but an object/function generally must have one program-wide definition subject to the C linkage rules.**

---

# 5. Difference between local and global variables

## Answer

**Local variable** is declared inside a function:

```c
void function(void)
{
    int x = 10;  // Local to function
}
```

`x` has block scope and commonly resides on the stack.

**Global variable** is declared outside functions:

```c
int counter;  // Global
```

If it has static storage duration, it exists for the entire program execution.

### Memory layout:

```text
.text
.rodata
.data      <- initialized global
.bss       <- uninitialized global
```

## Where is each stored in memory?

### Answer

**Local (automatic):**
- Commonly on the stack
- But compiler may optimize to register or eliminate entirely

**Global/Static:**
- `.data` section if initialized
- `.bss` section if uninitialized

### Default initialization:

**Local automatic variable:**

```c
void f(void)
{
    int x;  // Indeterminate value
}
```

Has an **indeterminate value** (not necessarily zero).

**Global/static objects:**

```c
int x;          // Zero-initialized
static int y;   // Zero-initialized
```

Are initialized to zero if no explicit initializer is provided.

### Expert answer

> **Local/global describes scope, not necessarily physical memory location. Automatic locals commonly use the stack, while static-storage objects commonly reside in data/BSS sections, but optimization can change the physical implementation. Automatic uninitialized objects have indeterminate values; static-storage objects are initialized to zero when no initializer is provided.**

---

# 6. Scope and lifetime of variables

## Answer

These are different concepts.

### Scope:
Answers: "Where can the name be used?"

### Lifetime (storage duration):
Answers: "During what period does the object exist?"

### Example:

```c
void function(void)
{
    int x = 10;
}
```

`x` has:
- **Block scope** (name visible only in function)
- **Automatic storage duration** (exists during function execution)

### Global variable:

```c
int global;
```

has:
- **File scope**
- **Static storage duration** (exists for entire program)

### Static local - the trick question:

```c
void counter(void)
{
    static int count = 0;
    count++;
}
```

`count` has:
- **Block scope** (name only visible inside function)
- **Static storage duration** (value persists between calls!)

### Diagram:

```text
                 VARIABLE
                    |
          +---------+---------+
          |                   |
        Scope              Lifetime
          |                   |
          v                   v
    Where name works     How long object exists
```

### Expert answer

> **Scope describes the visibility of an identifier in source code, while storage duration describes how long the corresponding object exists. A static local is a good example: block scope but static storage duration.**

---

# 7. What are storage classes in C?

## Answer

Common storage-class specifiers:

```c
auto
register
static
extern
```

Important categories:

```text
automatic
static
allocated
thread (in C11+)
```

For traditional interviews, focus on: `auto`, `register`, `static`, `extern`

---

# 8. Difference between `auto`, `register`, `static`, `extern`

## Answer

### `auto`

```c
void f(void)
{
    auto int x = 10;  // Default for block-scope
}
```

**Characteristics:**
- Block scope
- Automatic storage duration
- Default for local variables
- Object created on function entry, destroyed on exit

### `register`

```c
register int counter;
```

**Characteristics:**
- Storage-class hint to the compiler
- Suggests variable may benefit from register storage
- Modern compilers generally ignore and optimize themselves
- **Cannot take address**: `&x` is not allowed

```c
register int x;
&x;     /* Constraint violation */
```

### `static`

#### Static local:

```c
void f(void)
{
    static int count = 0;
    count++;
}
```

**Characteristics:**
- Block scope
- Static storage duration
- Value persists across function calls
- Initialized only once

#### Static file-scope:

```c
static int internal_counter;
```

**Characteristics:**
- **Internal linkage** (only visible within this translation unit)
- Static storage duration

### `extern`

```c
extern int counter;
```

**Characteristics:**
- Declares an external object/function
- The object is defined elsewhere
- Declaration only, not a definition

### Example with extern:

**file1.c:**

```c
int counter = 10;
```

**file2.c:**

```c
extern int counter;

void f(void)
{
    counter++;
}
```

### What if `extern` is declared but never defined?

**Answer:**

Compilation may succeed, but the linker will report an undefined reference:

```text
Compiler -> source accepted
   |
   v
Linker -> "Where is counter?"
   |
   v
undefined reference error
```

### Storage-class summary table:

| Keyword             | Typical scope | Storage duration                        | Key point                               |
| ------------------- | ------------- | --------------------------------------- | --------------------------------------- |
| `auto`              | Block         | Automatic                               | Default local behavior                  |
| `register`          | Block         | Automatic                               | Address cannot be taken                 |
| `static` local      | Block         | Static                                  | Value persists between calls            |
| `static` file scope | File          | Static                                  | Internal linkage                        |
| `extern`            | Depends       | Usually static storage                  | Declaration of external object/function |

### Expert answer

> **`auto` and `register` apply to block-scope automatic objects, `static` can provide static storage duration or internal linkage depending on context, and `extern` declares an entity whose definition is provided elsewhere. `register` is mostly a legacy optimization hint today, and its address cannot be taken.**

---

## OPERATORS

# 9. Difference between `i++` and `++i`

## Answer

### Post-increment (`i++`):

```c
int i = 5;
int x = i++;
```

**Process:**
1. Use old value: `x = 5`
2. Increment: `i = 6`

### Pre-increment (`++i`):

```c
int i = 5;
int x = ++i;
```

**Process:**
1. Increment: `i = 6`
2. Use new value: `x = 6`

### Diagram:

```text
i++              ++i
use old value -> increment
     |               |
     v               v
   i = 5         i = 6
     |               |
     v               v
increment        use new value
     |               |
     v               v
   i = 6
```

## Which is more efficient?

### Answer

For a scalar integer in C, modern compilers can generally generate equally efficient code when the resulting value is not used differently.

### Standalone statements:

```c
i++;     /* Generally same efficiency */
++i;     /* As modern compiler optimizations */
```

### Expert answer

> **The difference is semantic: `i++` yields the old value, while `++i` yields the incremented value. For ordinary scalar integers, modern compilers generally optimize either form equally when the surrounding semantics permit it.**

---

# 10. Difference between `=` and `==`

## Answer

### `=` is assignment:

```c
x = 10;     // Put 10 into x
```

### `==` is equality comparison:

```c
x == 10     // Is x equal to 10? (returns 0 or 1)
```

## Common bug:

```c
if (x = 10)    /* Assigns 10, evaluates to true */
{
    ...
}
```

Should be:

```c
if (x == 10)   /* Compares x to 10 */
{
    ...
}
```

## Embedded example:

```c
if (status == UART_READY)  /* Correct comparison */
{
    transmit();
}
```

---

# 11. What is short-circuit evaluation?

## Answer

C's logical operators `&&` and `||` can avoid evaluating their right-hand operand.

### `&&` (AND) behavior:

For `A && B`, if A is false, B is not evaluated.

**Example:**

```c
if (ptr != NULL && ptr->value == 10)
{
    ...
}
```

If `ptr == NULL`, the second condition is skipped, preventing NULL dereference.

### `||` (OR) behavior:

For `A || B`, if A is true, B is not evaluated.

**Example:**

```c
if (error || retry_count > 3)
{
    ...
}
```

If `error` is true, `retry_count` is not checked.

### Important note:

Short-circuiting applies to `&&` and `||` only. Bitwise operators `&` and `|` do NOT short-circuit.

### Expert answer

> **Logical `&&` and `||` evaluate left-to-right and may skip the right operand. This is particularly useful for guarding unsafe operations such as pointer dereferences or bounds-dependent accesses.**

---

# 12. Difference between logical and bitwise operators

## Answer

### Logical operators:

```c
&&   (AND)
||   (OR)
!    (NOT)
```

Operate on logical truth values (0 or 1):

```c
if (a && b)
```

### Bitwise operators:

```c
&    (AND)
|    (OR)
^    (XOR)
~    (NOT)
<<   (left shift)
>>   (right shift)
```

Operate on individual bits:

**Example:**

```c
uint8_t status = 0x05;   // Binary: 0000 0101

if (status & 0x01)       // Check bit 0
{
    // Bit 0 is set
}
```

## What is `!!x`?

### Answer

Double negation normalizes a value to exactly 0 or 1.

**Example:**

```c
int x = 25;

!!x         // First: !25 = 0, then: !0 = 1
```

### Use case:

```c
uint8_t is_ready = !!(status & READY_BIT);
```

Guarantees the result is 0 or 1, not just any non-zero value.

### Expert answer

> **Logical operators reason about truth values and provide short-circuit behavior; bitwise operators manipulate individual bits. `!!x` is commonly used to normalize any scalar truth value into exactly `0` or `1`.**

---

# 13. What is operator precedence?

## Answer

Operator precedence determines which operators bind more strongly.

**Example:**

```c
int x = 2 + 3 * 4;
```

Multiplication has higher precedence than addition, so:

```text
2 + (3 * 4) = 14
```

NOT `(2 + 3) * 4 = 20`

## Use parentheses for clarity:

```c
result = a + (b * c);   /* More readable */
```

## Common embedded trap:

```c
if (x & 1 == 0)    /* WRONG: equality has higher precedence */
```

This is parsed as: `x & (1 == 0)`, not `(x & 1) == 0`

**Correct:**

```c
if ((x & 1) == 0)   /* Correct */
```

### Expert rule

> **When bitwise operations and comparisons are mixed, always use parentheses.**

---

# 14. What is associativity?

## Answer

Associativity determines how operators of the same precedence are grouped.

**Example:**

```c
a = b = c = 5;
```

Assignment is **right-associative**, so:

```text
a = (b = (c = 5))
```

**Evaluation:**
1. `c = 5`
2. `b = 5`
3. `a = 5`

**Result:** `a = 5`, `b = 5`, `c = 5`

## Precedence vs Associativity:

### Precedence:
Answers: "Which operator binds first?"

### Associativity:
Answers: "How are same-precedence operators grouped?"

### Diagram:

```text
Expression
   |
   +--> precedence (operator priority)
   |
   +--> associativity (grouping direction)
```

### Expert answer

> **Precedence determines which operators bind more strongly. Associativity determines grouping when operators at the same precedence level occur together. Assignment operators are right-associative, so `a = b = c = 5` becomes `a = (b = (c = 5))`.**

---

## CONTROL STATEMENTS

# 15. Difference between `while` and `do-while`

## Answer

### `while` loop:

Condition is tested **before** the body.

```c
while (condition)
{
    work();
}
```

Body may execute:
- 0 times
- 1 time
- many times

### `do-while` loop:

Body executes first, then condition is checked.

```c
do
{
    work();
} while (condition);
```

Body **always executes at least once**.

## Embedded use case:

Hardware initialization that must happen at least once:

```c
do
{
    status = read_status();
} while (!(status & READY_BIT));
```

Menu/command processing:

```c
do
{
    command = get_command();
    process_command(command);
} while (command != EXIT);
```

### Expert answer

> **`while` checks the condition before entering the body, so zero iterations are possible. `do-while` executes the body first and checks afterward, so at least one iteration occurs.**

---

# 16. Difference between `break` and `continue`

## Answer

### `break`:

Terminates the nearest loop or `switch` completely.

```c
for (int i = 0; i < 10; i++)
{
    if (i == 5)
        break;      /* Exit loop immediately */
}
```

### `continue`:

Skips the remainder of the current loop iteration and proceeds to the next.

```c
for (int i = 0; i < 10; i++)
{
    if (i == 5)
        continue;   /* Skip i=5, continue with i=6 */

    process(i);
}
```

### Diagram:

```text
             LOOP
               |
        +------+------+
        |             |
      break        continue
        |             |
        v             v
     exit loop   next iteration
```

### Expert answer

> **`break` exits the nearest loop or switch completely. `continue` skips the remaining body of the current loop iteration and proceeds with the next iteration.**

---

# 17. Difference between `goto` and function call

## Answer

### Function call:

Transfers control to another function:

```c
foo();
```

**Flow:**
```text
caller -> foo() -> return -> caller continues
```

### `goto`:

Unconditional jump to a label within the same function:

```c
goto cleanup;

...

cleanup:
    free(buffer);
```

## Why can `goto` be problematic?

Poorly designed `goto` creates "spaghetti" control flow that's hard to follow.

## When is `goto` acceptable?

**One legitimate use: centralized error cleanup**

```c
int process(void)
{
    char *buffer = NULL;
    FILE *file = NULL;

    buffer = malloc(1024);
    if (buffer == NULL)
        goto cleanup;

    file = open_file();
    if (file == NULL)
        goto cleanup_buffer;

    /* Work... */

    close_file(file);

cleanup_buffer:
    free(buffer);

cleanup:
    return -1;
}
```

The Linux kernel uses this structured cleanup style for error paths.

### Comparison:

| Aspect                | `goto`                     | Function call           |
| --------------------- | -------------------------- | ----------------------- |
| Scope                 | Within same function       | To another function     |
| Control flow          | Can hurt readability       | Encourages modularity   |
| Best use              | Structured cleanup/errors  | Reusable behavior       |
| Stack behavior        | No new frame typically     | Creates call/return     |

### Expert answer

> **A function call transfers control to another function and normally returns to the caller. `goto` transfers control to a label within the same function. `goto` should not be used for arbitrary spaghetti control flow, but structured cleanup/error paths are a legitimate systems-programming use.**

---

## ARRAYS

# 18. What is an array?

## Answer

An array is a contiguous collection of elements of the same type.

**Example:**

```c
int a[5];
```

**Memory layout:**

```text
a

+------+------+------+------+------+
| a[0] | a[1] | a[2] | a[3] | a[4] |
+------+------+------+------+------+
```

Elements are stored contiguously in memory.

## Why is this useful?

Fast access using index notation:

```c
a[0], a[1], a[2], ...
```

Pointer arithmetic:

```c
a[i] is equivalent to *(a + i)
```

---

# 19. How is an array stored in memory?

## Answer

**Example:**

```c
int a[4] = {10, 20, 30, 40};
```

If `sizeof(int) = 4` and array starts at `0x1000`:

```text
0x1000 -> a[0] = 10
0x1004 -> a[1] = 20
0x1008 -> a[2] = 30
0x100C -> a[3] = 40
```

**Diagram:**

```text
array
 |
 v
+------+------+------+------+
|  10  |  20  |  30  |  40  |
+------+------+------+------+
 0x1000  0x1004 0x1008 0x100C
```

Elements occupy **contiguous memory** addresses separated by the size of each element.

---

# 20. Why does array index start from 0?

## Answer

Because C defines:

```c
a[i]
```

in terms of:

```c
*(a + i)
```

For the first element, `a[0]` becomes `*(a + 0)` which is `*a` (base address).

For the second element, `a[1]` becomes `*(a + 1)` (base + one element).

### Pointer arithmetic:

```c
a[i]  means  base + (i * sizeof(element_type))
```

**Example:**

```c
int a[5];

a[3]        // base + (3 * sizeof(int))
            // = base + 12 (if sizeof(int) = 4)
```

### Expert answer

> **C defines `a[i]` as `*(a + i)`. Therefore index zero means no offset from the first element, and pointer arithmetic automatically scales the offset by the size of the pointed-to type.**

---

# 21. Can array size change dynamically?

## Answer

A normal fixed-size array cannot be resized:

```c
int a[10];

a = another_size;     /* Not allowed */
```

### Dynamic allocation:

You can allocate storage at runtime:

```c
int *a = malloc(n * sizeof(*a));

if (a != NULL)
{
    /* Use a as array */
}

free(a);
```

### Variable Length Arrays (VLA) in C99:

```c
void process(size_t n)
{
    int a[n];   /* Size determined at runtime */

    ...
}
```

## Is VLA safe in embedded?

### Answer

Depends on constraints. Concerns:

```text
Runtime size
     |
     v
Stack allocation
     |
     v
Potentially large stack usage
```

In embedded systems, stack is often constrained. For deterministic software, embedded developers prefer:

- Fixed-size buffers
- Static allocation
- Memory pools
- Carefully controlled dynamic allocation

### Expert answer

> **A normal array object's size cannot be changed after creation. Runtime-sized storage can be obtained with dynamic allocation, while C99 VLAs provide runtime-sized automatic arrays. VLAs require careful stack analysis in embedded systems because their storage is typically automatic and the maximum size may be difficult to bound.**

---

# 22. Difference between `arr` and `&arr`

## Answer

Consider:

```c
int arr[5];
```

### `arr`:

In most expressions, `arr` undergoes **array-to-pointer conversion**.

Resulting type: `int *` (pointer to int)

Conceptually points to: `&arr[0]` (first element)

### `&arr`:

The address-of operator. Type: `int (*)[5]` (pointer to array of 5 ints)

Points to: The entire array as a unit

### Pointer arithmetic difference:

```c
arr + 1      /* Advances by sizeof(int) */

&arr + 1     /* Advances by sizeof(entire array) */
             /* = 5 * sizeof(int) */
```

### Diagram:

```text
arr          &arr
 |            |
 v            v
+----+----+   +-----------------------------------+
| 0  | 1  |   | entire array of 5 ints          |
+----+----+   +-----------------------------------+
^
|
Points to first element
```

### Expert answer

> **`arr` normally converts to `int *`, pointing to the first element. `&arr` is a pointer to the entire array, with type `int (*)[5]`. Consequently, `arr + 1` advances one `int`, while `&arr + 1` advances one entire array of five `int`s.**

---

# 23. What happens on out-of-bounds access?

## Answer

**Example:**

```c
int a[5];

a[5] = 100;     /* Invalid - outside array */
```

Valid indices: 0, 1, 2, 3, 4

Using out-of-bounds pointer/access results in **undefined behavior**.

## Why doesn't C check bounds?

C was designed as a low-level systems language with minimal runtime overhead.

There is no automatic bounds metadata with every array.

The compiler cannot reliably know that runtime `i` is within intended bounds.

## What is buffer overflow?

**Buffer overflow** occurs when a program writes beyond intended storage:

```c
char buffer[8];

buffer[8] = 'A';    /* Outside valid range */
```

**Memory:**

```text
buffer
+----+----+----+----+----+----+----+----+
| 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  |
+----+----+----+----+----+----+----+----+
                                      ^
                                  last valid

buffer[8]
    |
    v
outside object
```

### Consequences:

Can corrupt:
- Neighboring variables
- Stack frames
- Heap metadata
- Function pointers
- Control data

### Expert answer

> **C does not automatically perform array bounds checks. Accessing outside an array's valid range can produce undefined behavior. A buffer overflow occurs when an operation exceeds the intended storage region and may corrupt adjacent state or control data.**

---

# 24. Multi-dimensional arrays and row-major layout

## Answer

**Example:**

```c
int a[2][3];    /* 2 rows, 3 columns */
```

C stores elements in **row-major order**:

```text
a[0][0]
a[0][1]
a[0][2]
a[1][0]
a[1][1]
a[1][2]
```

**Diagram:**

```text
Column:   0    1    2

Row 0:  [ ]  [ ]  [ ]

Row 1:  [ ]  [ ]  [ ]

Memory:
+-----+-----+-----+-----+-----+-----+
|0,0  |0,1  |0,2  |1,0  |1,1  |1,2  |
+-----+-----+-----+-----+-----+-----+
```

## How is `a[i][j]` calculated?

**Formula:**

```text
address = base + ((i * num_columns) + j) * sizeof(element)
```

**Example for `a[1][2]`:**

```text
address = base + ((1 * 3) + 2) * sizeof(int)
        = base + 5 * sizeof(int)
```

So `a[1][2]` is the 6th element in memory.

## Why does C need the second dimension?

The compiler needs to know how large each row is to calculate the offset:

```c
void process(int a[][3], size_t rows)
{
    /* Compiler knows each row = 3 ints */
}
```

### Expert answer

> **C uses row-major layout for multidimensional arrays. `int a[2][3]` is an array of two arrays, each containing three `int`s, so the memory order is `a[0][0]`, `a[0][1]`, `a[0][2]`, `a[1][0]`, `a[1][1]`, `a[1][2]`.**

---

## STRINGS

# 25. Difference between `char str[] = "Hello"` and `char *str = "Hello"`

## Answer

These look similar but create different objects.

### `char str[] = "Hello"`:

Creates an actual character **array**:

```text
str
 |
 v
+---+---+---+---+---+----+
| H | e | l | l | o | \0 |
+---+---+---+---+---+----+
```

Contains a **copy** of the characters. Modifiable:

```c
str[0] = 'J';   /* Valid - now "Jello" */
```

### `char *str = "Hello"`:

Creates a **pointer** to a string literal:

```text
str
 |
 v
+---+---+---+---+---+----+
| H | e | l | l | o | \0 |
+---+---+---+---+---+----+
     (string literal)
```

String literal has static storage and is non-modifiable:

```c
str[0] = 'J';   /* Undefined behavior */
```

Better:

```c
const char *str = "Hello";  /* Explicit intent */
```

## Where is the string literal stored?

String literals have **static storage duration**.

On typical systems, placed in read-only section:

```text
.rodata
```

On embedded systems, often in flash/ROM.

### Important answer:

Don't say: "String literals are always in ROM."

Instead: **String literals have static storage duration. Implementations commonly place them in read-only memory, but the C language itself does not prescribe a physical section.**

---

# 26. Difference between `strlen()` and `sizeof()`

## Answer

These are completely different operations.

### `strlen()`:

Counts characters before the terminating `'\0'` at **runtime**:

```c
char str[] = "Hello";

strlen(str)     /* Returns 5 */
```

```text
H e l l o \0
\________/
  strlen = 5 (not including \0)
```

### `sizeof()`:

Returns the storage size of object/type in bytes:

```c
char str[] = "Hello";

sizeof(str)     /* Returns 6 (includes \0) */
```

```text
H e l l o \0
\___________/
  sizeof = 6 (includes \0)
```

### Comparison:

```c
char str[] = "Hello";

strlen(str);   // 5
sizeof(str);   // 6
```

### Important pointer case:

```c
char *p = "Hello";

sizeof(p);      /* Size of pointer: typically 4 or 8 */
strlen(p);      /* Still 5 (runtime string length) */
```

### Expert answer

> **`strlen` operates on a null-terminated string at runtime and counts characters before `'\0'`. `sizeof` operates on a type/object and gives its storage size in C bytes; for an array it includes the terminating `'\0'` when present.**

---

# 27. Why do strings end with `'\0'`?

## Answer

C does not have a built-in string type. A C string is conventionally:

> A contiguous sequence of characters terminated by a null character.

**Example:**

```c
char str[] = "Hello";
```

**Memory:**

```text
H e l l o \0
```

## Why is the terminator needed?

Functions need to know where the string ends:

```c
strlen(str)     /* Searches for '\0' */
strcpy(dst, src)/* Copies until '\0' */
strcmp(a, b)    /* Compares until '\0' */
printf("%s")    /* Prints until '\0' */
```

**Conceptually:**

```c
strlen(str)
    |
    v
read character
    |
    v
is it '\0'?
   / \
 yes  no
  |    |
stop  continue
```

## What if `'\0'` is missing?

**Example:**

```c
char buffer[5] = {'H', 'e', 'l', 'l', 'o'};  /* No \0 */

printf("%s", buffer);   /* Undefined behavior */
```

`printf("%s")` continues reading memory looking for zero byte.

### Consequences:

- Garbage output
- Crash
- Memory disclosure
- Out-of-bounds reads

### Real-world embedded scenario:

```text
UART packet received
     |
     v
5 bytes received into buffer[5]
     |
     v
printf("%s", buffer)    /* Not a valid C string! */
     |
     v
Reads beyond buffer looking for \0
```

### Important distinction:

A character array is NOT automatically a C string:

```c
char data[5];   /* Just an array, not a string */
```

It becomes a valid C string only when `'\0'` exists within the object.

### Expert answer

> **The null terminator provides the boundary for C's null-terminated string convention. Without it, string APIs cannot know where the string ends and may read beyond the object's bounds, causing undefined behavior.**

---

# 28. Difference between `strcpy()` and `strncpy()`

## Answer

### `strcpy()`:

```c
strcpy(destination, source);
```

Copies characters from source including terminating `'\0'`.

**Example:**

```c
char dst[10];

strcpy(dst, "Hello");     /* Copies "Hello\0" */
```

Result:

```text
H e l l o \0
```

### Major danger:

`strcpy()` does not know destination size. This is dangerous:

```c
char dst[4];

strcpy(dst, "Hello");     /* Buffer overflow! */
```

Destination cannot hold 6 bytes (5 chars + \0).

Result: **undefined behavior, possible memory corruption**

### `strncpy()`:

```c
strncpy(dst, src, n);
```

Copies at most `n` characters.

**Important trap:** `strncpy()` does NOT always null-terminate!

**Example:**

```c
char dst[5];

strncpy(dst, "Hello", sizeof(dst));
```

Source has 5 non-null characters. All 5 positions filled. No room for `'\0'`.

Result: `dst` is **not** a valid null-terminated C string!

### Safer pattern:

```c
char dst[5];

strncpy(dst, src, sizeof(dst) - 1);
dst[sizeof(dst) - 1] = '\0';    /* Force termination */
```

Now guaranteed valid C string:

```text
+---+---+---+---+----+
| H | e | l | l | \0 |
+---+---+---+---+----+
```

### Expert answer

> **`strcpy` copies through the source's terminating null character but performs no destination-size checking. `strncpy` limits the number of characters copied, but it does not guarantee null termination when the source length is at least the specified count.**

---

# 29. Common string vulnerabilities

## Answer

Major C string problems:

```text
1. Buffer overflow (exceeds destination)
2. Missing null termination
3. Out-of-bounds read
4. Use-after-free
5. Double free
6. Format-string vulnerabilities
7. Integer overflow in allocation size
8. Truncation bugs
```

## Buffer overflow example:

```c
char buffer[8];

strcpy(buffer, input);      /* If input is long... */
```

Extra bytes write beyond `buffer`:

```text
buffer
+---+---+---+---+---+---+---+---+
|   |   |   |   |   |   |   |   |
+---+---+---+---+---+---+---+---+
                                  |
                                  v
                        adjacent memory corrupted
```

May overwrite adjacent objects, stack frames, or control data.

## Why is `gets()` dangerous?

```c
char buffer[10];

gets(buffer);       /* No size limit! */
```

Cannot know when to stop before overflow.

Input like `ABCDEFGHIJKLMNOPQRSTUVWXYZ` exceeds buffer[10].

### Consequences:

- Stack corruption
- Function pointer corruption
- Return address corruption
- Potential code execution

**Removed from C standard in C11** because it cannot be made safe.

### Safer alternatives:

```c
fgets(buffer, sizeof(buffer), stdin);

uart_receive(buffer, sizeof(buffer));
```

### Embedded real-world scenario:

```text
UART packet:
    payload_length = 32 bytes
         |
         v
copy into:
    buffer[16]
         |
         v
No validation!
         |
         v
32-byte copy into 16-byte buffer
         |
         v
Buffer overflow + memory corruption
```

**Correct approach:**

```c
if (payload_length <= sizeof(buffer))
{
    copy_data(buffer, data, payload_length);
}
else
{
    /* Error: payload too large */
}
```

### Expert answer

> **The major C string risks are missing terminators, destination-size violations, out-of-bounds reads/writes, truncation, and unsafe formatting. `gets()` is especially dangerous because its interface provides no destination size and it was removed from C11.**

---

## Summary Table

| Question | Key Answer |
|----------|-----------|
| 1. Basic data types | `char`, integer types, floating types, `void`; exact representation is implementation-dependent |
| 2. Signed vs unsigned | Unsigned arithmetic is modulo; signed overflow is UB |
| 3. Type sizes | `int` is not universally 4 bytes; use `<stdint.h>` for exact-width types |
| 4. Declaration vs definition | Declaration introduces; definition provides the entity |
| 5. Local vs global | Scope/storage duration differ; don't equate local strictly with stack |
| 6. Scope vs lifetime | Scope = name visibility; lifetime = object existence |
| 7. Storage classes | `auto`, `register`, `static`, `extern`; understand linkage/storage duration |
| 8. `auto/register/static/extern` | Automatic, register hint, static lifetime/linkage, external declaration |
| 9. `i++` vs `++i` | Old value vs new value; modern compilers optimize equally for scalars |
| 10. `=` vs `==` | Assignment vs equality comparison |
| 11. Short circuit | `&&` and `||` may skip right operand for safety |
| 12. Logical vs bitwise | Truth-value operations vs bit manipulation; `!!x` normalizes to 0 or 1 |
| 13. Precedence | Determines operator binding strength |
| 14. Associativity | Determines grouping for same-precedence operators |
| 15. `while` vs `do-while` | Zero-or-more vs at-least-one iteration |
| 16. `break` vs `continue` | Exit loop vs skip current iteration |
| 17. `goto` vs function call | Local jump vs function transfer; cleanup `goto` can be valid |
| 18. Array | Contiguous elements of same type |
| 19. Array memory | Elements in contiguous storage |
| 20. Index starts at 0 | `a[i]` is `*(a+i)` due to pointer arithmetic |
| 21. Dynamic array | Use dynamic allocation or VLA; consider embedded stack constraints |
| 22. `arr` vs `&arr` | `int *` vs `int (*)[N]`; different pointer arithmetic |
| 23. Out-of-bounds | Results in undefined behavior |
| 24. Multidimensional array | C uses row-major layout; dimension needed for offset calculation |
| 25. `char[]` vs `char *` | Array copy vs pointer to read-only string literal |
| 26. `strlen` vs `sizeof` | Runtime string length vs object/type storage size |
| 27. `'\0'` terminator | Marks end of C string for all string APIs |
| 28. `strcpy` vs `strncpy` | Unbounded vs bounded; `strncpy` may not terminate |
| 29. String vulnerabilities | Overflow, missing terminator, OOB access, format-string bugs, `gets()` removed |

---

## Interview Mental Model

For a basic C question at interview level, use this framework:

```text
                 QUESTION
                    |
                    v
             What does C standard say?
                    |
                    v
          What does typical compiler do?
                    |
                    v
       What behavior is GUARANTEED?
                    |
          +---------+---------+
          |                   |
          v                   v
       Defined             Undefined
       behavior            Behavior
          |                   |
          v                   v
  Explain details      Explain why UB
                       & risks
```

### Strong interview answer:

Instead of: "It prints garbage."

Say: **"The access is outside the valid element range, so this is undefined behavior. The implementation may appear to read adjacent memory, but that behavior is not guaranteed by the C standard."**

---

## Key Concepts to Connect

```text
TYPE          →  size, alignment, representation, operations
OBJECT        →  storage duration, lifetime, scope, linkage
POINTER       →  target type, arithmetic, lifetime, alignment
ARRAY         →  contiguous storage, conversion, sizeof exception
STRING        →  char array, '\0' terminator, length, capacity, safety
```

At interview level, distinguish what C **guarantees** from what's merely common on a particular compiler/ABI/OS/CPU.
