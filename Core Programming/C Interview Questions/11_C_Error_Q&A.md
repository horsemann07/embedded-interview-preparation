# C Errors, Warnings & Debugging — Complete Q&A Guide

> **Focus**: Compile-time, Link-time, Runtime, Logical, and Embedded-specific errors  
> **Level**: Intermediate → Advanced (debugging strategies and error diagnosis)  
> **Use Case**: Interview preparation for experienced C/Embedded developers

---

## 1. COMPILE-TIME ERRORS

### Syntax Errors

**Q1: What is a compile-time error?**

A compile-time error occurs when the compiler cannot parse or validate source code. The compiler stops compilation and reports the error before any object files are generated.

**Answer:**
```
Compile-time errors happen during:
- Preprocessing (expansion of macros, includes)
- Lexical analysis (tokenization)
- Syntax parsing (grammar validation)
- Semantic analysis (type checking, symbol resolution)

Who catches it: Preprocessor → Compiler

Example Flow:
  Source Code (.c) 
      ↓ (Preprocessor)
    Expanded Code
      ↓ (Compiler Lexer/Parser)
    Syntax/Semantic Check
      ↓ (ERROR found)
    "error: ..." message
    ✗ No object file generated
```

**Follow-up:** Difference between compile-time and link-time error?
- **Compile-time**: Syntax/type/semantic issues, caught by compiler
- **Link-time**: Symbol resolution issues, caught by linker
- **Runtime**: Memory/logic issues, detected during execution

---

**Q2: What is a syntax error? Give 5 examples.**

**Answer:**
```c
// Example 1: Missing semicolon
int x = 10        // ✗ error: expected ';' before 'if'
if (x > 5) { }

// Example 2: Assignment (=) instead of comparison (==)
if (x = 5) { }    // ⚠ Compiles but assigns instead of compares (common bug)

// Example 3: Invalid identifier (starts with number)
int 1var;         // ✗ error: expected '(' or '[' or '.'

// Example 4: Missing expression
int x = ;         // ✗ error: expected expression before ';'

// Example 5: Mismatched brackets
void foo() {      // ✗ error: expected declaration specifier
                  // Missing closing brace somewhere

// Example 6: Wrong operator usage
int y = *;        // ✗ error: expected expression before ';'
```

**Follow-up:** Which is more dangerous — missing semicolon or `=` instead of `==`?
- Missing semicolon: Caught by compiler immediately
- `=` vs `==`: Often compiles successfully but changes logic (sneaky bug!)

---

**Q3: What happens if you forget `#include <stdio.h>` and call `printf()`?**

**Answer:**

In **C89/C90**:
```c
// No #include <stdio.h>
int main() {
    printf("hello");    // ⚠ Warning: implicit function declaration
    return 0;           // Compiles but dangerous!
}

Result:
- Compiler assumes printf returns int (implicit declaration)
- printf call assumes 5 arguments on typical platform
- May work by accident, or crash at runtime
- printf may not have correct prototype, causing ABI mismatch
```

In **C99/C11**:
```c
// Implicit function declarations removed
Result:
- error: implicit declaration of function 'printf'
- Compilation fails
- Forces you to include proper header
```

**Best Practice:**
```c
#include <stdio.h>  // Always include before using standard library functions

int main() {
    printf("hello\n");  // ✓ Correct prototype available
    return 0;
}
```

**Follow-up:** Why did C99 remove implicit function declarations?
- To catch typos in function names (common source of bugs)
- To ensure type safety (function signature must be known)
- To enable better optimization by compiler
- To catch portability issues early

---

### Type Errors

**Q4: What is a type mismatch error?**

**Answer:**
```c
// Example 1: Assigning integer to pointer
int *p = 10;           // ✗ error: invalid initializer (int to pointer)

// Example 2: Assigning string to float
float f = "hello";     // ✗ error: incompatible types in initialization

// Example 3: Wrong pointer type assignment
int arr[10];
float *p = arr;        // ✗ warning: incompatible pointer types

// Example 4: Function return type mismatch
int get_value() {
    return "hello";    // ✗ error: incompatible types in return
}
```

**Severity:** 
- Errors: Compilation fails
- Warnings: Compilation succeeds but undefined behavior likely

**Follow-up:** How to enforce type safety?
```bash
gcc -Wall -Wextra -Werror program.c  # Treat warnings as errors
```

---

**Q5: What is an incompatible pointer type warning?**

**Answer:**
```c
int x = 5;
float *p = &x;         // ⚠ warning: incompatible pointer types

// Why is this dangerous?
// p points to int memory, but dereferencing p treats it as float:
*p = 3.14;             // Writes 4 bytes as float, but x is int representation
                       // Corrupts the integer value
                       // Example: x might become 0x4048f5c3 (3.14 as float bits)
```

**Memory Layout:**
```
&x:  [  int value  ]    (4 bytes on most platforms)
      x86: int = 0x00000005

When accessed as float:
float interpretation of 0x00000005 = ~2.1e-44 (very different!)
```

**Follow-up:** Can this be intentional?

Yes, in some cases (type punning):
```c
// Intentional type punning (dangerous but sometimes necessary)
int x = 0x12345678;
unsigned char *bytes = (unsigned char *)&x;  // Explicit cast

// Access individual bytes (byte order examination)
printf("0x%02X 0x%02X 0x%02X 0x%02X\n", 
       bytes[0], bytes[1], bytes[2], bytes[3]);

// Result on little-endian: 0x78 0x56 0x34 0x12
```

But: Use with explicit cast and comment explaining intent.

---

**Q6: What happens when you pass wrong argument types to a function?**

**Answer:**
```c
// Function prototype (correct)
void process_int(int x);

// Calling with wrong type
int main() {
    float f = 3.14;
    process_int(f);         // ⚠ May compile with warning
    
    char *str = "hello";
    process_int(str);       // ⚠ or ✗ depending on compiler strictness
}

// What happens at runtime?
// float 3.14 passed as int:
//   - Float bits reinterpreted as int bits
//   - Result: garbage value
//   - Example: 0x4048f5c3 → 1079865795 (not 3)

// Pointer passed as int:
//   - Address bits reinterpreted as int
//   - May cause crash if treated as value
```

**Follow-up:** What if there is no prototype?

```c
// No prototype (old C-style, bad practice)
// process_int(f);  // Compiler doesn't know what types to expect
// Argument passed as-is without conversion
// Function receives garbage or wrong value

// Modern solution: Always provide prototypes
void process_int(int x);   // Forward declaration
```

---

### Declaration Errors

**Q7: What is an "undeclared identifier" error?**

**Answer:**
```c
int main() {
    printf("Value: %d\n", value);  // ✗ error: 'value' undeclared
    return 0;
}

// Causes:
// 1. Variable never declared
// 2. Variable declared in different scope
// 3. Typo in variable name
// 4. Missing #include for external symbol
```

**Example — Scope Issue:**
```c
void foo() {
    int x = 10;  // x declared here
}

int main() {
    printf("%d\n", x);  // ✗ error: x not declared in this scope
    return 0;
}
```

**Solution:** Either declare at broader scope or use `extern`:
```c
int x = 10;  // Global

void foo() { }

int main() {
    printf("%d\n", x);  // ✓ Works
}
```

---

**Q8: What is a "redefinition" error?**

**Answer:**
```c
// Same translation unit (same .c file)
int x = 5;
int x = 10;    // ✗ error: redefinition of 'x'
```

**Difference: Redefinition vs Redeclaration**

| Term | Meaning | Allowed? |
|------|---------|----------|
| **Redeclaration** | Declaring same symbol again | ✓ Yes (extern declarations) |
| **Redefinition** | Defining same symbol again | ✗ No (only one definition per translation unit) |

**Example:**
```c
// Redeclaration (✓ OK)
extern int x;   // Declare that x exists elsewhere
extern int x;   // Declare again (OK)
int x = 5;      // Define once

// Redefinition (✗ ERROR)
int x = 5;
int x = 10;     // ERROR: x already defined
```

**Header Guard Problem:**
```c
// Without header guards (in header.h)
int x = 5;

// In main.c
#include "header.h"
#include "header.h"  // Included twice! 

// Result: error: redefinition of 'x' (if included twice in same file)
```

**Solution: Header Guards**
```c
#ifndef HEADER_H
#define HEADER_H

int x = 5;  // Only defined once

#endif
```

---

**Q9: What is a "conflicting types" error?**

**Answer:**
```c
// Declaration says int, but definition is float
int get_value();        // Forward declaration

float get_value() {     // ✗ error: conflicting types for 'get_value'
    return 3.14;
}
```

**When does this happen?**

```c
// Scenario 1: Different return types
int foo();
double foo() { return 1.5; }  // ✗ Conflict

// Scenario 2: Different parameter types
void bar(int x);
void bar(float x) { }         // ✗ Conflict

// Scenario 3: Extern vs definition mismatch
extern int count;             // Declare as int
count = 3.14;                 // But used as float
int count;                    // Definition as int (conflict caught)
```

---

### Preprocessor Errors

**Q10: What is the `#error` directive? When do you use it?**

**Answer:**
```c
// Compile-time configuration validation
#ifndef CONFIG_BOARD
#error "CONFIG_BOARD must be defined"
#endif

// Example: Enforcing required compile flags
#ifndef DEBUG
#error "DEBUG flag not set. Compile with -DDEBUG"
#endif

// Example: Portability check
#if !defined(__GNUC__)
#error "This code requires GCC compiler"
#endif
```

**Use Cases:**

1. **Configuration Validation** - Ensure required macros are set
2. **Platform Checks** - Verify architecture/OS compatibility
3. **Compiler Checks** - Enforce specific compiler or version
4. **Feature Checks** - Ensure required C standard version

---

**Q11: What happens with circular `#include`?**

**Answer:**
```c
// file1.h
#ifndef FILE1_H
#define FILE1_H
#include "file2.h"  // Includes file2

struct A {
    struct B *b;    // Forward reference to B
};
#endif

// file2.h
#ifndef FILE2_H
#define FILE2_H
#include "file1.h"  // Includes file1 (circular!)

struct B {
    struct A *a;    // Forward reference to A
};
#endif
```

**Without Header Guards:**
```
Compilation:
  file1.h → includes file2.h
    file2.h → includes file1.h
      file1.h → includes file2.h (infinite loop!)
      
Result: Preprocessor expands infinitely, out of memory
```

**With Header Guards:**
```
  file1.h → #ifndef FILE1_H, #define FILE1_H
  includes file2.h
    file2.h → #ifndef FILE2_H, #define FILE2_H
    includes file1.h
      file1.h → #ifndef FILE1_H (already defined, skip)
    Rest of file2.h
  Rest of file1.h

Result: ✓ Works correctly
```

**Best Practice:**
```c
#ifndef FILE_NAME_H
#define FILE_NAME_H

// ... content ...

#endif
```

---

**Q12: What is a "macro redefined" warning?**

**Answer:**
```c
// First definition
#define MAX 100

// Later redefinition with different value
#define MAX 200         // ⚠ warning: "MAX" redefined

// Compiler compilation:
#define MAGIC 0x42
#define MAGIC 0x42      // ⚠ Same value but still warned (usually)
```

**How to silence:**
```c
// Undefine before redefining
#define MAX 100
#undef MAX
#define MAX 200         // ✓ OK now

// Or use #ifdef guard
#ifndef MAX
#define MAX 200
#endif
```

**Embedded consideration:** Use compiler flag to treat as error:
```bash
gcc -Werror=redefine program.c  # Fail on macro redefinition
```

---

**Q13: What happens if `#include` file is not found?**

**Answer:**
```c
#include "nonexistent.h"   // ✗ error: file not found
#include <missing_lib.h>   // ✗ error: file not found
```

**Error Types:**

| Include Type | Search Path | Error |
|---|---|---|
| `#include "file.h"` | Current dir first, then system | fatal error: 'file.h' file not found |
| `#include <file.h>` | System include paths only | fatal error: 'file.h' file not found |

**Debugging:**
```bash
# Check where compiler looks for includes
gcc -v -E empty.c 2>&1 | grep include

# Or list include search paths
gcc -Xpreprocessor -v -E empty.c

# Explicitly add include path
gcc -I/path/to/includes program.c
```

---

## 2. LINK-TIME ERRORS

**Q14: What is a link-time error?**

**Answer:**

Link-time errors occur during the linking phase, after all object files are compiled. The linker tries to resolve external symbols and combine object files.

```
Compilation Phase:
  foo.c → foo.o ✓
  bar.c → bar.o ✓
  
Linking Phase:
  foo.o + bar.o + libraries → program
  (Symbol resolution, relocation)
  ✗ Linking fails
```

**Key Difference:**
```
Compile-time: Syntax/type errors within single file detected by compiler
Link-time: Cross-file symbol resolution errors detected by linker
Runtime: Program runs but crashes or produces wrong output
```

---

**Q15: What is an "undefined reference" error?**

**Answer:**
```
undefined reference to `foo'
collect2: error: ld returned 1 exit status
```

**Cause:** Symbol `foo` is used (called) but never defined/implemented.

**Example:**
```c
// main.c
extern void foo();

int main() {
    foo();              // Call foo
    return 0;
}

// MISSING: foo() definition
// bar.c doesn't exist or doesn't have foo() implementation
```

**Compile Flow:**
```
main.c → main.o (✓ compiles, foo marked as "undefined external")
bar.c → bar.o  (✓ compiles, foo not found anywhere)

ld main.o bar.o
    → Looking for symbol 'foo' in all object files
    → Not found in main.o
    → Not found in bar.o
    → Not found in linked libraries
    ✗ error: undefined reference to 'foo'
```

**Common Causes:**

1. **Missing source file** - foo() defined in missing file
```bash
gcc main.c                 # Only compiles main.c
gcc main.c bar.c          # ✓ Includes foo() from bar.c
```

2. **Typo in function name** - foo() defined but called as Foo()
```c
void foo() { }
Foo();  // ✗ Undefined reference to 'Foo' (case-sensitive)
```

3. **Missing library** - foo() in external library
```bash
gcc main.c -lmath  # Link with math library (-l flag)
# If library not found: ✗ ld: cannot find -lmath
```

4. **Symbol name mangling (C++ issue)** - but in C just wrong scope
```c
static void helper() { }  // static = not visible outside file
// Another file calls helper() → undefined reference
```

---

**Q16: What is a "multiple definition" error?**

**Answer:**
```
multiple definition of `x'
/tmp/cc... first defined here
```

**Cause:** Symbol `x` is defined in more than one object file.

```c
// file1.c
int x = 5;          // Definition of x

// file2.c
int x = 10;         // Another definition of x!

// Compile:
gcc file1.c file2.c
✗ error: multiple definition of 'x'
```

**Why defining a variable in header causes this:**

```c
// config.h
int MAX_SIZE = 100;   // Definition (not declaration!)

// file1.c
#include "config.h"   // Includes: int MAX_SIZE = 100;
                      // → Creates object file1.o with MAX_SIZE

// file2.c
#include "config.h"   // Includes: int MAX_SIZE = 100;
                      // → Creates object file2.o with MAX_SIZE

// gcc file1.c file2.c
✗ error: multiple definition of 'MAX_SIZE'
```

**Solution 1: Use `extern` and define once**
```c
// config.h
extern int MAX_SIZE;    // Declaration only (no initialization)

// config.c
int MAX_SIZE = 100;     // Definition in one place

// file1.c, file2.c
#include "config.h"     // ✓ All reference same MAX_SIZE
```

**Solution 2: Use `static` for file-local scope**
```c
// config.h
static int MAX_SIZE = 100;  // static = private to each file that includes it
                            // Each file gets its own copy (OK for constants)
```

**Solution 3: Use `#define` for compile-time constants**
```c
// config.h
#define MAX_SIZE 100    // Preprocessor - no symbol in object file
```

---

**Q17: How does `static` and `extern` fix multiple definition errors?**

**Answer:**

```c
// header.h

// Option 1: Static (file-local scope)
static int value = 5;   // Each .c file that includes this gets its own copy
                        // Not visible to linker

// Option 2: Extern + define in one file
extern int value;       // Declaration: "exists somewhere"
                        // No definition, no symbol in this object file

// In one .c file:
// global.c
int value = 5;          // Definition: creates symbol in object file

// option 3: static inline function (combined definition + local)
static inline void helper() {
    // Each file gets its own copy of this function
    // Linker doesn't unify them
}
```

**Comparison Table:**

| Approach | Header | Implementation | Visible to Linker |
|---|---|---|---|
| `extern int x;` | Declaration | Define once in .c | Yes (unified) |
| `static int x;` | Definition | Included in each .c | No (private copy per file) |
| `#define X` | Define | Preprocessor | No (not a symbol) |

---

**Q18: What is a "duplicate symbol" error in static libraries?**

**Answer:**

Static libraries (`.a` files on Unix, `.lib` on Windows) contain multiple object files.

```
mylib.a contains:
  ├─ foo.o (defines foo, helper)
  ├─ bar.o (defines bar, helper)  ← duplicate 'helper' symbol!
  └─ util.o (defines util)
```

**When linking:**
```bash
gcc main.o -lmylib
✗ error: duplicate symbol 'helper'
  first defined in foo.o
  second defined in bar.o
```

**Solutions:**

1. **Rename conflicting functions**
```c
// foo.c
static void foo_helper() { }  // Unique name

// bar.c
static void bar_helper() { }  // Unique name
```

2. **Make them static (file-local)**
```c
// foo.c
static void helper() { }  // Local to foo.c

// bar.c
static void helper() { }  // Local to bar.c (different symbol)
```

3. **Use namespace prefix**
```c
void foo__helper() { }    // Clear which module it belongs to
void bar__helper() { }
```

---

**Q19: What happens when library linking order is wrong?**

**Answer:**

```bash
# Correct order (library after object that uses it)
gcc main.o -lm
✓ Works

# Wrong order (library before object)
gcc -lm main.o
✗ May fail: undefined reference to sqrt
# (gcc processes left-to-right, math library exhausted before main.o examined)
```

**Why does order matter with static libraries?**

The linker processes files left-to-right, adding symbols to a symbol table:

```
gcc -lm main.o        # libm processed first, all its symbols added
                      # main.o processed next, looks for sqrt in table
                      # sqrt not yet encountered, not in table
                      # ✗ Undefined reference

gcc main.o -lm        # main.o processed, sqrt marked as "undefined external"
                      # libm processed, sqrt found in libm
                      # ✓ Symbol resolved
```

**Modern linkers (GNU ld with `--as-needed`):**
```bash
# Sometimes handles reordering automatically
gcc -Wl,--as-needed main.o -lm
```

**Best practice:** List libraries after objects that use them:
```bash
gcc main.o util.o -lm -lc  # Libraries at end
```

---

**Q20: What is a weak symbol? How does it affect linking?**

**Answer:**

A **weak symbol** is a symbol that the linker can ignore if a strong definition exists elsewhere.

```c
// Original implementation
__attribute__((weak)) void handler(void) {
    printf("Default handler\n");
}

// In platform-specific code, override with strong definition
void handler(void) {
    printf("ARM-specific handler\n");
}

// Linker chooses the strong definition over weak
```

**Linking behavior:**
```
If two definitions exist:
  1 Strong + 1 Weak → Linker uses Strong
  2 Strong + 0 Weak → Error: multiple definition
  0 Strong + 2 Weak → Linker uses one (arbitrary, usually first)
  0 Strong + 1 Weak → Linker uses the Weak
  No definition       → Undefined reference error
```

**Real-world use (ARM interrupt handlers):**
```c
// In core library (default weak handler)
__attribute__((weak)) void UART_IRQHandler(void) {
    while(1);  // Hang (default behavior)
}

// In application code (override with strong definition)
void UART_IRQHandler(void) {
    // Custom interrupt handling
    receive_byte();
}

// Linker: "Strong definition in app, ignore weak in library" → Uses app version
```

**Advantages:**
- Provides default implementations
- Allows optional overrides
- Avoids multiple definition errors

**Disadvantages:**
- Less type-safe than explicit function pointers
- Can hide errors if wrong function is called
- Linker behavior may be non-obvious

---

## 3. RUNTIME ERRORS

### Segmentation Fault

**Q21: What is a segmentation fault?**

**Answer:**

A segmentation fault (segfault, `SIGSEGV`) is a runtime error where the CPU tries to access memory that:
1. Doesn't exist
2. Isn't mapped to the process
3. The process doesn't have permission to access

**Hardware/OS Level:**
```
CPU attempts to read/write memory at address
→ MMU (Memory Management Unit) checks page tables
→ Address not mapped OR permission violation
→ CPU raises exception (segmentation fault signal)
→ OS terminates process
→ Core dump generated (if enabled)
```

**Difference from other crashes:**

| Error | Cause | Signal |
|---|---|---|
| Segmentation Fault | Invalid memory access | SIGSEGV (11) |
| Bus Error | Misaligned access (some CPUs) | SIGBUS (7) |
| Abort | Explicit call to abort() | SIGABRT (6) |
| Illegal Instruction | Invalid CPU instruction | SIGILL (4) |

---

**Q22: Common causes of segfault:**

**Answer:**

```c
// 1. NULL pointer dereference (most common)
int *p = NULL;
*p = 10;                   // ✗ SIGSEGV
int x = *p;                // ✗ SIGSEGV

// 2. Dangling pointer (use after free)
int *p = malloc(sizeof(int));
free(p);
*p = 10;                   // ✗ Accessing freed memory (may crash, may not)

// 3. Stack buffer overflow
char buf[10];
strcpy(buf, "This is way too long for the buffer");  // Writes past stack
                                                      // ✗ Corrupts stack, segfault

// 4. Heap buffer overflow
char *p = malloc(10);
strcpy(p, "This is way too long");  // Writes past heap
                                     // ✗ Corrupts heap metadata

// 5. Writing to string literal
char *str = "hello";
str[0] = 'H';              // ✗ Attempting to write to read-only memory

// 6. Invalid array access
int arr[5];
arr[10] = 42;              // ✗ Writing past array bounds

// 7. Stack overflow (infinite recursion)
void infinite() {
    infinite();            // ✗ Stack grows until it hits guard page
}

// 8. Accessing unmapped memory region
int *p = (int *)0x99999999;  // Random address
*p = 5;                        // ✗ Likely not mapped
```

---

**Q23: What is `SIGSEGV` signal?**

**Answer:**

`SIGSEGV` (Signal: SEGmentation Violation) is raised by the OS when a segmentation fault occurs.

**Can you catch it?**

```c
#include <signal.h>
#include <stdio.h>

void segfault_handler(int sig) {
    printf("Caught segfault, signal: %d\n", sig);
    // Attempt cleanup
    exit(1);
}

int main() {
    signal(SIGSEGV, segfault_handler);  // Register handler
    
    int *p = NULL;
    *p = 10;  // Segfault occurs, handler called
    
    return 0;
}

// Output:
// Caught segfault, signal: 11
```

**Should you catch it?**

**Bad idea (generally):**
- Memory state is corrupted
- Can't reliably continue execution
- May mask the underlying bug
- Cleanup may cause more crashes

**Acceptable cases:**
- Logging/debugging before terminating
- Cleanup critical resources (close files, flush logs)
- Post-mortem analysis

**Better approach:**
- Fix the bug causing segfault
- Use tools (Valgrind, ASan) to detect issues before runtime

---

**Q24: What is a core dump? How do you analyze it?**

**Answer:**

A **core dump** is a file containing the process memory at the moment of crash.

**Enable core dumps:**
```bash
ulimit -c unlimited  # Allow unlimited core dump size
# or
ulimit -c 1024000    # Limit to 1 GB

# Verify:
ulimit -c            # Should print unlimited or size
```

**Reproduce crash and analyze:**
```bash
gcc -g program.c -o program  # Compile with debug symbols (-g)

./program              # Crashes
# Generates: core or core.12345 (depending on OS)

gdb program core       # Open debugger with core dump

(gdb) bt              # Print backtrace
#0  0x00000000 in ?? ()
#1  0x08048401 in foo ()
#2  0x08048415 in main ()

(gdb) frame 1         # Select frame
(gdb) info locals     # Print local variables
(gdb) print var       # Print specific variable
(gdb) quit
```

**What info does core dump contain?**

```
- Full memory contents (heap, stack, data, etc.)
- CPU registers at crash time
- Process state (open files, signals, etc.)
- Call stack (return addresses of functions)
```

**Read backtrace from core dump:**

```bash
gdb ./program ./core

(gdb) bt full
#0  crash_function () at program.c:45
        local_var = 42
#1  0x08048401 in caller () at program.c:50
        x = 0x0
#2  0x08048415 in main () at program.c:55

# Shows exactly which line caused the crash
```

---

### Bus Error

**Q25: What is a bus error (`SIGBUS`)?**

**Answer:**

A **bus error** occurs when the CPU detects a hardware-level access problem:
- Misaligned memory access (on strict alignment architectures)
- Accessing non-existent memory bus
- DMA access violation
- Parity error in memory

**Difference between segfault and bus error:**

| Error | Cause | Architecture | 
|---|---|---|
| Segmentation Fault | Invalid virtual address | Most systems |
| Bus Error | Hardware access violation | ARM, MIPS (strict alignment) |

```c
// Misaligned access (SIGBUS on ARM Cortex-M0)
int x = 5;
int *p = (int *)((char *)&x + 1);  // Point to misaligned address
*p = 10;                            // ✗ SIGBUS on ARM (not x86)
```

**On which architectures?**

- **x86/x64**: Usually tolerates misalignment (slower but works)
- **ARM Cortex-M0/M1**: Strict alignment, SIGBUS on misaligned access
- **ARM Cortex-M3+**: Configurable alignment checking
- **MIPS**: Strict alignment

---

### Memory Errors

**Q26: What is a memory leak?**

**Answer:**

A **memory leak** occurs when allocated memory is never freed, making it inaccessible but unavailable for reuse.

```c
void leak_example() {
    int *p = malloc(1024);
    printf("Allocated %p\n", p);
    // Function ends, p goes out of scope
    // Memory still allocated but no way to free it
    // ✗ Memory leak: 1024 bytes lost
}

int main() {
    for (int i = 0; i < 1000000; i++) {
        leak_example();  // Leaks 1024 bytes per iteration
                         // Total: 1 GB leaked
    }
    return 0;
}
```

**How to detect it:**

1. **Valgrind** (Linux)
```bash
valgrind --leak-check=full ./program
```

2. **AddressSanitizer (ASan)**
```bash
gcc -fsanitize=address program.c -o program
./program
```

3. **Manual tracking** (embedded systems)
```c
#define MAX_ALLOCATIONS 1000
struct {
    void *ptr;
    size_t size;
} allocations[MAX_ALLOCATIONS];

void *tracked_malloc(size_t size) {
    void *p = malloc(size);
    // Record in allocations table
    return p;
}

void print_leaks() {
    // Print all unfreed allocations
}
```

**Why critical in embedded:**

- Long-running systems (medical devices, routers)
- Limited memory (microcontrollers)
- Can't restart frequently
- Leak eventually causes system failure

---

**Q27: What is double free?**

**Answer:**

**Double free**: Calling `free()` twice on the same pointer.

```c
int *p = malloc(sizeof(int));
free(p);
free(p);              // ✗ Double free — undefined behavior
```

**What actually happens internally?**

```
After free(p):
  Heap metadata updated: [p's memory block marked as free]
  
After free(p) again:
  Heap allocator tries to mark same block as free
  May corrupt heap metadata
  Next malloc() may return overlapping blocks
  Writes to one "allocated" block corrupt another
  Segfault or silent corruption
```

**Prevention:**
```c
int *p = malloc(sizeof(int));
free(p);
p = NULL;            // ✓ Mark as NULL after free

free(p);             // Free NULL is safe (no-op)
// Output: Nothing (free checks for NULL)
```

**Safe free macro (Linux kernel style):**
```c
#define FREE(p) do { if (p) { free(p); (p) = NULL; } } while(0)

FREE(p);
FREE(p);  // ✓ Safe, no crash
```

---

**Q28: What is use-after-free?**

**Answer:**

**Use-after-free**: Accessing memory after it has been freed.

```c
int *p = malloc(sizeof(int));
*p = 42;

free(p);              // Memory returned to allocator

*p = 100;             // ✗ Use-after-free — UB
int x = *p;           // ✗ Reading freed memory

p->field = 5;         // ✗ Accessing freed struct
```

**Why might it "work" sometimes?**

```
After free():
- Memory block marked as free in heap
- Physical memory not overwritten
- Next allocation might not use same block yet
- Read: May get old data (wrong but doesn't crash yet)
- Write: May corrupt next allocation or heap metadata

Later:
- Next malloc() reuses freed memory
- Writes corrupt the new allocation
- Crash happens in seemingly unrelated code
- Hard to debug (crash far from root cause)
```

**Detection:**
```bash
valgrind --tool=memcheck ./program
AddressSanitizer with -fsanitize=address
```

**Prevention:**
```c
int *p = malloc(sizeof(int));
free(p);
p = NULL;  // Prevent accidental reuse

*p = 100;  // Now crashes immediately (NULL dereference)
           // Much easier to debug than silent corruption
```

---

**Q29: What is heap corruption?**

**Answer:**

**Heap corruption**: Corrupting heap metadata or allocated blocks, leading to allocator malfunction.

**Mechanism:**

```
Heap Layout:
  [Metadata] [Block A] [Metadata] [Block B] [Metadata] [Block C]
     freed              freed              allocated

If Block A overflows:
  [Metadata] [Block A overflow → corrupts Metadata] [Block B metadata corrupted]
                          ↓
                    Heap allocator confused
                    
Next malloc():
  Allocator tries to read corrupted metadata
  Makes wrong decisions about free space
  Returns overlapping blocks
  Multiple pointers to same memory → Corruption
```

**Example:**
```c
char *a = malloc(10);
char *b = malloc(10);

strcpy(a, "This is way too long and overflows into b's metadata");

free(a);
free(b);  // ✗ Tries to free corrupted block, heap fails
          // May segfault or corrupt subsequent allocations

malloc(5); // ✗ May return invalid pointer
```

**How to detect:**

```bash
valgrind --leak-check=full ./program
  # Shows: Invalid write of 10 bytes in block of 10 bytes

AddressSanitizer:
  gcc -fsanitize=address program.c
  # Runtime detection of buffer overflow
  
# Manual (embedded):
  - Fill heap with pattern
  - Check pattern before freeing
  - Use memory pool with fixed sizes (no fragmentation)
```

---

**Q30: What is buffer overflow?**

**Answer:**

**Buffer overflow**: Writing past the end of allocated buffer.

```c
// Stack buffer overflow
char buf[10];
strcpy(buf, "this is way too long for the buffer");  // Writes past buf
                                                       // Corrupts stack

// Heap buffer overflow  
char *buf = malloc(10);
strcpy(buf, "this is way too long for the buffer");  // Writes past heap
                                                       // Corrupts heap metadata
```

**Stack buffer overflow example:**
```
Memory Layout:
  [buf(10)] [local_var] [return address]
  ← 10 →   ← 4 →       ← 4 →

strcpy(buf, long_string):
  buf[0] = 't'
  buf[1] = 'h'
  ...
  buf[9] = 'r'
  buf[10] = 'e'  // ✗ Overflow! Writes to local_var
  buf[11] = 'e'  // ✗ Overflow! Overwrites return address!
  ...
  
When function returns:
  pop return address from stack
  ✗ Return address corrupted
  CPU jumps to invalid address
  SIGSEGV or SIGILL
```

**Types:**

1. **Stack buffer overflow** - Overwrite locals, return address
2. **Heap buffer overflow** - Overwrite next block, heap metadata

**Prevention:**

```c
// Bad: unbounded copy
strcpy(dest, src);

// Good: bounded copy
strncpy(dest, src, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';  // Ensure null termination

// Better: use safer functions
snprintf(dest, sizeof(dest), "%s", src);
```

**Compiler protections:**

```bash
gcc -fstack-protector program.c     # Stack canary
gcc -fstack-protector-all program.c # Canary on all functions
gcc -D_FORTIFY_SOURCE=2 program.c   # Runtime checks on str functions
```

---

**Q31: What is stack overflow?**

**Answer:**

**Stack overflow**: Exhausting stack memory, usually through infinite/deep recursion.

```c
// Infinite recursion
void infinite() {
    infinite();              // ✗ Stack grows indefinitely
}

// Deep recursion
int fibonacci(int n) {
    return fibonacci(n-1) + fibonacci(n-2);  // ✗ Exponential call stack
}

// Large local arrays
void bad() {
    char huge[1000000];  // ✗ Allocates on stack, may overflow
}
```

**Stack Exhaustion Process:**

```
Stack grows:
  [unused]
  [empty]
  [empty]
  [empty]  ← SP (stack pointer)
  
Recursion adds frames:
  [frame N]
  [frame N-1]
  [frame N-2]
  ...
  [frame 1]
  [frame 0]  ← SP
  
When SP reaches guard page (if present):
  MMU: "Access to guard page"
  ✗ SIGSEGV or SIGBUS
  Process terminates
```

**How to detect (embedded without OS):**

```c
// Fill stack with pattern
extern char _stack_top;
void fill_stack() {
    volatile char *p = &_stack_top;
    while ((char *)&p > _stack_top - STACK_SIZE) {
        *p = 0xAA;
        p--;
    }
}

// Check pattern before stack corruption
int check_stack_usage() {
    volatile char *p = &_stack_top;
    int depth = 0;
    while (*p == 0xAA && depth < STACK_SIZE) {
        p--;
        depth++;
    }
    return depth;  // Returns how much stack was used
}
```

**Default stack size:**

```bash
ulimit -s          # Print current stack size (usually 8 MB)
ulimit -s 1024     # Set to 1 MB
```

**Can you change it?**

```bash
# At compile time:
gcc program.c -Wl,--stack-size=0x10000  # 64 KB stack

# At runtime:
ulimit -s 2048     # Set to 2 MB before running
./program
```

---

### Arithmetic Errors

**Q32: What happens on division by zero?**

**Answer:**

**Integer division by zero:**
```c
int x = 10 / 0;      // ✗ Undefined behavior (UB)
                     // Usually: SIGFPE (Floating Point Exception)
                     // But not guaranteed

// May cause:
// - CPU exception → SIGFPE signal
// - Undefined result → random value
// - Hang
```

**Floating-point division by zero:**
```c
float f = 10.0 / 0.0;     // ✗ Results in +Infinity (IEEE 754)
double d = 0.0 / 0.0;     // ✗ Results in NaN (Not a Number)

printf("%f\n", f);  // Prints: inf
printf("%f\n", d);  // Prints: -nan
```

**What is `SIGFPE`?**

`SIGFPE` (Signal: Floating Point Exception) is raised on arithmetic errors:
- Integer division by zero
- Integer overflow (on some systems)
- Floating-point exceptions (on some systems)

```c
#include <signal.h>

void fpe_handler(int sig) {
    printf("Caught SIGFPE\n");
    exit(1);
}

int main() {
    signal(SIGFPE, fpe_handler);
    int x = 10 / 0;  // Raises SIGFPE
    return 0;
}
```

**Difference: Integer vs Floating-point:**

| Type | Division by 0 | Result |
|---|---|---|
| Integer | 10 / 0 | UB (usually SIGFPE) |
| Float | 10.0 / 0.0 | +Infinity (IEEE 754) |
| Double | 10.0 / 0.0 | +Infinity (IEEE 754) |
| 0.0 / 0.0 | NaN | Not a Number |

---

**Q33: What is integer overflow?**

**Answer:**

**Integer overflow**: Result exceeds maximum representable value.

```c
int x = INT_MAX;     // 2147483647
x = x + 1;           // ✗ Signed overflow — UNDEFINED BEHAVIOR

unsigned int u = UINT_MAX;  // 4294967295
u = u + 1;           // ✗ Result wraps to 0 (defined behavior)
```

**Why is signed overflow UB?**

```
Compiler reasoning:
  "Signed overflow can't happen in a correct program"
  "If it does, anything can happen"
  
Compiler may:
- Generate code that assumes it won't happen
- Remove overflow check
- Remove entire code block
- Generate wrong code

Example:
  int x = INT_MAX;
  if (x + 1 > x) {     // Always true? (compiler thinks)
      printf("Yes");
  }
  // Compiler might remove the condition entirely
  // Or generate incorrect assembly
```

**Why is unsigned overflow defined?**

```
Unsigned integers guaranteed to wrap (modulo arithmetic):
  unsigned int u = UINT_MAX;  // 2^32 - 1
  u = u + 1;                  // Result: 0 (wraps around)
  
This is DEFINED and PORTABLE (unlike signed overflow)
```

**How to safely check for overflow:**

```c
// Check before overflow
int safe_add(int a, int b, int *result) {
    if (a > 0 && b > INT_MAX - a) {
        return -1;  // Would overflow
    }
    if (a < 0 && b < INT_MIN - a) {
        return -1;  // Would underflow
    }
    *result = a + b;
    return 0;  // OK
}
```

**Better approach (use wider type):**
```c
long add_checked(int a, int b) {
    long result = (long)a + (long)b;
    if (result > INT_MAX || result < INT_MIN) {
        return LLONG_MAX;  // Error value
    }
    return result;
}
```

---

**Q34: What is floating-point precision error?**

**Answer:**

**Precision error**: Floating-point arithmetic doesn't represent all numbers exactly.

```c
float f = 0.1 + 0.2;
if (f == 0.3) {          // ✗ May be false!
    printf("Equal\n");
} else {
    printf("Not equal\n");  // Usually prints this
}

printf("%.20f\n", f);    // Prints: 0.30000001192092896
printf("%.20f\n", 0.3);  // Prints: 0.29999999999999999
```

**Why should you never use `==` with floats?**

Floating-point numbers are approximations:
- 0.1 cannot be represented exactly in binary
- 0.2 cannot be represented exactly
- Sum of approximations ≠ Exact 0.3

```
Binary: 0.1 = 0.00011001100110011... (repeating)
Stores:  0.09999999... or 0.10000001... (rounded)

0.1 + 0.2 = (0.0999... + 0.1999...) ≠ exactly 0.3
```

**Epsilon comparison (safe):**

```c
#define EPSILON 1e-6  // Tolerance

int almost_equal(float a, float b) {
    return fabs(a - b) < EPSILON;
}

// Safe comparison:
if (almost_equal(f, 0.3)) {
    printf("Close enough\n");
}
```

**Avoid floats when possible:**

```c
// Instead of:
float percent = 33.33;

// Use:
int percent = 3333;  // Represents 33.33%
printf("%.2f%%\n", percent / 100.0);
```

---

### Format String Errors

**Q35: What happens with format string mismatch?**

**Answer:**

**Format string mismatch**: Format specifier type doesn't match argument type.

```c
int x = 5;
printf("%s", x);        // ✗ UB: expects string, gets int
printf("%d", 3.14);     // ✗ UB: expects int, gets double

// What happens:
// Printf reads from stack as if argument is correct type
// Interprets bit patterns incorrectly
// May print garbage, crash, or worse
```

**Example:**
```c
int x = 5;
float f = 3.14;

printf("%s %d\n", x, f);
// printf reads:
//   First arg as string: tries to dereference 5 as pointer
//   ✗ Likely segfault

printf("%d %f\n", x, f);
// printf reads:
//   First arg (int): prints 5 ✓
//   Second arg (float): reads from stack/registers, misinterprets
//   Prints garbage like 0.000000
```

**What is format string vulnerability/attack?**

**Dangerous code:**
```c
char *user_input = get_input();
printf(user_input);     // ✗✗✗ DANGEROUS
```

**Attack scenario:**
```
Input: "%x %x %x %x"
Output: Prints 4 values from stack (leaking memory)

Input: "%s"
Output: Tries to dereference stack value as pointer, read memory

Input: "%n"
Output: Writes to stack location (format strings support write!)
        Can overwrite return address → arbitrary code execution
```

**Safe code:**
```c
char *user_input = get_input();
printf("%s\n", user_input);     // ✓ Safe: explicit format
```

**Compiler protection:**
```bash
gcc -Wformat -Wformat-security program.c  # Warn about format vulnerabilities
```

---

## 4. LOGICAL ERRORS

**Q36: What is a logical error?**

**Answer:**

**Logical error**: Program runs without crashing but produces wrong results.

```c
// Calculate average
int sum = 0;
for (int i = 1; i <= 10; i++) {   // ✗ Starts at 1, not 0
    sum += i;
}
int avg = sum / 10;  // Result off by 5.5

// Expected: avg = 5.5
// Actual: avg = 5 (and only 10 numbers summed instead of [0..9])
```

**Why is it the hardest to debug?**

- No compiler error or warning
- No crash signal
- Program runs to completion
- Wrong output discovered by:
  - Human testing
  - Customer complaint
  - Unit test failure
- Requires understanding intended behavior
- Not mechanical (like syntax errors)

---

**Q37: Common logical errors:**

**Answer:**

```c
// 1. Using = instead of ==
if (x = 5) {           // ✓ Assigns 5 to x, evaluates to true
    printf("Yes\n");
}
// Intended: if (x == 5)

// 2. Off-by-one error
for (int i = 0; i <= n; i++) {  // Processes n+1 elements
    printf("%d ", arr[i]);
}
// Intended: i < n

// 3. Wrong operator precedence
int result = 10 + 5 * 2;  // = 20, not 30
// int result = 10 + (5 * 2) vs (10 + 5) * 2

// 4. Missing break in switch
switch(x) {
case 1:
    printf("One\n");
    // Missing: break;
case 2:
    printf("Two\n");
    break;
}
// If x=1: prints "One\n" and "Two\n" (fall-through)

// 5. Infinite loop (unintentional)
while (x < 10) {
    printf("%d\n", x);
    // Missing: x++
}
// x never changes, loop never ends

// 6. Wrong pointer arithmetic
int arr[10];
int *p = arr + 5;   // Points to arr[5]
p++;                // Points to arr[6], not arr[5] + 1 byte!
                    // (pointer arithmetic is scaled by element size)

// 7. Signed/unsigned comparison bug
unsigned int a = 5;
int b = -1;
if (a > b) {        // ✗ b promoted to unsigned
    printf("Yes\n");  // b becomes UINT_MAX (4294967295)
}
```

**Why is signed/unsigned comparison dangerous?**

```
Signed -1: 0xFFFFFFFF in two's complement
Unsigned: 4294967295 (literal interpretation)

When mixed in comparison:
  -1 < 5 (signed arithmetic) = true
  4294967295 > 5 (unsigned interpretation) = true
  
Different results depending on how compiler handles comparison!
```

---

## 5. DEBUGGING TOOLS & TECHNIQUES

**Q38: How do you use `gdb` for debugging C programs?**

**Answer:**

**GDB (GNU Debugger)** - Interactive debugger for C programs.

**Compile with debug symbols:**
```bash
gcc -g program.c -o program  # -g includes debug info (DWARF format)
# Don't strip with -s, keep symbols
```

**Basic GDB commands:**

```bash
gdb program                # Start debugger

# Breakpoints:
(gdb) break main           # Set breakpoint at main()
(gdb) break func:10        # Set at line 10 in func
(gdb) break *.c:10         # By file:line
(gdb) break function.c:45
(gdb) info break           # List all breakpoints
(gdb) delete 1             # Delete breakpoint 1
(gdb) disable 1            # Disable (keep for later)

# Execution:
(gdb) run                  # Start execution
(gdb) run arg1 arg2        # Run with arguments
(gdb) continue             # Resume after breakpoint
(gdb) step                 # Step into function
(gdb) next                 # Step over function
(gdb) step 5               # Step 5 times
(gdb) finish               # Run until return

# Inspection:
(gdb) backtrace            # Print call stack
(gdb) frame 0              # Select frame
(gdb) info locals          # Print local variables
(gdb) info args            # Print arguments
(gdb) print var            # Print variable value
(gdb) print &var           # Print address of var
(gdb) print *ptr           # Dereference pointer
(gdb) print arr[10]        # Array element
(gdb) print sizeof(int)    # Expression evaluation

# Watchpoints:
(gdb) watch x              # Break when x changes
(gdb) watch *ptr           # Break when *ptr changes
(gdb) info watch           # List watchpoints

# Quit:
(gdb) quit
```

**Example session:**
```bash
$ gdb ./crash_program
(gdb) break main
Breakpoint 1 at 0x8048401
(gdb) run
Starting program: /crash_program

Breakpoint 1, main () at program.c:10
10      int x = 5;
(gdb) step
11      int *p = NULL;
(gdb) step
12      *p = 10;          ← This will crash
(gdb) print p
$1 = (int *) 0x0
(gdb) continue
Segmentation fault

(gdb) backtrace
#0  0x08048416 in main () at program.c:12
```

---

**Q39: What is Valgrind? What errors does it detect?**

**Answer:**

**Valgrind** - Dynamic analysis tool for memory debugging and profiling.

**Installation:**
```bash
apt install valgrind  # Ubuntu/Debian
brew install valgrind # macOS
```

**Compile with debug symbols:**
```bash
gcc -g program.c -o program
```

**Run with Valgrind:**
```bash
valgrind --leak-check=full ./program
valgrind --leak-check=full --show-leak-kinds=all ./program
valgrind --track-origins=yes ./program  # Track uninitialized value origins
```

**Errors detected:**

```
1. Memory leak
   "40 bytes in 1 blocks are definitely lost in loss record"
   
2. Invalid read/write
   "Invalid read of 4 bytes"
   "Invalid write of 8 bytes"
   
3. Use-after-free
   "Use of uninitialised value of size 8"
   "Address 0x... is 0 bytes inside a block of 4 bytes free'd"
   
4. Uninitialized value
   "Uninitialised byte(s)"
   "The value just read is supposed to be initialised"
```

**Example:**
```c
// memleak.c
int main() {
    int *p = malloc(100);
    printf("Allocated %p\n", p);
    return 0;  // p never freed
}

$ gcc -g memleak.c -o memleak
$ valgrind --leak-check=full ./memleak

==12345== 100 bytes in 1 blocks are definitely lost in loss record
==12345== at 0x...: malloc
==12345== by 0x...: main (memleak.c:3)
```

---

**Q40: What is AddressSanitizer (ASan)?**

**Answer:**

**AddressSanitizer** - Compiler instrumentation for runtime memory error detection (faster than Valgrind).

**Enable ASan:**
```bash
gcc -fsanitize=address program.c -o program
# or
gcc -fsanitize=address -g program.c -o program  # Debug symbols
```

**Run:**
```bash
./program
```

**Detects:**
- Buffer overflow/underflow
- Use-after-free
- Double free
- Memory leak
- Misaligned access

**Example:**
```c
// overflow.c
int main() {
    int arr[10];
    arr[10] = 5;  // Buffer overflow
    return 0;
}

$ gcc -fsanitize=address overflow.c -o overflow
$ ./overflow

=================================================================
==12345==ERROR: AddressSanitizer: stack-buffer-overflow on unknown write
    Write of size 4 at addr 0x7fff... at pc 0x... in main
    Address 0x7fff... is located in stack of thread T0 at offset 40 in frame
    main at overflow.c
    
    This frame has 1 object(s):
    [32, 72) 'arr' <== Memory access at offset 40 is out-of-bounds
==================================================================
```

**ASan vs Valgrind:**

| Feature | ASan | Valgrind |
|---------|------|----------|
| Speed | 2-8x slowdown | 10-100x slowdown |
| Compile | Instrumentation (-fsanitize) | No compilation change |
| Errors | Buffer over/underflow | Comprehensive |
| Platforms | Linux, macOS, Windows | Linux, macOS, Solaris |

---

**Q41: What is UndefinedBehaviorSanitizer (UBSan)?**

**Answer:**

**UBSan** - Detects undefined behavior at runtime.

**Enable:**
```bash
gcc -fsanitize=undefined program.c -o program
```

**Detects:**
- Integer overflow (signed)
- Division by zero
- Shift out of bounds
- Misaligned access
- Null pointer dereference
- Out-of-bounds array access
- Load of invalid value

**Example:**
```c
// ubsan.c
int main() {
    int x = INT_MAX;
    x = x + 1;  // Signed overflow
    return 0;
}

$ gcc -fsanitize=undefined ubsan.c -o ubsan
$ ./ubsan

ubsan.c:4:9: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
```

---

## 6. EMBEDDED-SPECIFIC RUNTIME ERRORS

**Q42: Watchdog timeout — what causes it?**

**Answer:**

**Watchdog timer** - Hardware timer that resets the system if not "pet" (reset) periodically.

**Common causes:**

```c
// 1. Infinite loop in main loop
int main() {
    while (1) {
        // Watchdog pet code missing!
        // or very rarely reached
    }
}

// 2. Blocking call in main loop
int main() {
    while (1) {
        blocking_i2c_read();  // Takes 10 seconds, watchdog reset at 5 seconds
        // Never reaches watchdog pet
    }
}

// 3. Infinite loop in ISR
void interrupt_handler() {
    while (1) {  // ✗ ISR hangs
        // Main loop can't run
        // Watchdog not pet
        // System resets
    }
}

// 4. Priority inversion in RTOS
// High-priority task waits on low-priority task
// Watchdog task can't run
// System resets
```

**Solution:**

```c
// Define watchdog interval (e.g., 5 seconds)
#define WATCHDOG_INTERVAL_MS 5000

// Enable watchdog
void init_watchdog() {
    WDT->CTRL = 0x01;  // Enable watchdog
}

// Pet watchdog regularly
void pet_watchdog() {
    WDT->RST = 0xA5;  // Special pattern to reset timer
}

// In main loop
int main() {
    init_watchdog();
    
    while (1) {
        pet_watchdog();  // ✓ Reset timer
        
        // Do work (must complete in < WATCHDOG_INTERVAL_MS)
        process_sensor_data();
        
        delay_ms(100);
    }
}
```

---

**Q43: Hard fault on ARM Cortex-M — common causes.**

**Answer:**

**Hard Fault** - Highest-priority CPU exception, usually unrecoverable.

**Common causes:**

```c
// 1. Misaligned memory access (some Cortex-M)
int *p = (int *)0x00000001;  // Misaligned to 1-byte boundary
*p = 5;                       // ✗ Hard fault on strict-alignment CPUs

// 2. Access to unmapped memory region
int *p = (int *)0xDEADBEEF;   // Random address
*p = 5;                        // ✗ Hard fault

// 3. Invalid instruction
asm("undefined_opcode");  // ✗ Hard fault

// 4. Write to read-only memory
char *code = (char *)0x08000000;  // Code section (Flash)
code[0] = 'X';                    // ✗ Hard fault (MPU protection)

// 5. Stack overflow hitting MPU boundary
void stack_overflow() {
    char huge[10000];  // Allocate on stack
    stack_overflow();  // Recurse → stack grows
                       // Hits MPU guard page
                       // ✗ Hard fault
}

// 6. Null pointer dereference (if MPU present)
int *p = NULL;
*p = 5;  // ✗ Hard fault (depends on MPU configuration)
```

**How to decode the hard fault register:**

```c
// ARM Cortex-M Hard Fault Status Register (HFSR)
void HardFault_Handler() {
    uint32_t hfsr = SCB->HFSR;
    
    if (hfsr & 0x80000000) {
        printf("Debug Event\n");
    }
    if (hfsr & 0x40000000) {
        printf("FORCED: Escalated fault\n");
        // Check CFSR for details
        uint32_t cfsr = SCB->CFSR;
        if (cfsr & 0x00000001) printf("  IACCVIOL: Instruction access violation\n");
        if (cfsr & 0x00000100) printf("  DACCVIOL: Data access violation\n");
        if (cfsr & 0x00010000) printf("  UNDEFINSTR: Undefined instruction\n");
    }
    if (hfsr & 0x00000002) {
        printf("VECTTBL: Vector table error\n");
    }
    
    while(1);  // Hang
}
```

**What is a bus fault, usage fault, memory management fault?**

| Fault | Cause | Severity |
|-------|-------|----------|
| Memory Management | MPU violation, exec from non-exec region | Configurable |
| Bus Fault | Misaligned access, invalid address space | Configurable |
| Usage Fault | Invalid instruction, division by zero | Configurable |
| Hard Fault | Unhandled fault, escalation from above | Not maskable |

---

## 7. ERROR HANDLING PATTERNS IN C

**Q44: Return code pattern:**

**Answer:**

```c
// Traditional error handling
int read_file(const char *filename, char *buffer, size_t size) {
    FILE *f = fopen(filename, "r");
    if (f == NULL) {
        return -1;  // Error code
    }
    
    size_t n = fread(buffer, 1, size, f);
    fclose(f);
    return (int)n;  // 0 or positive = success, negative = error
}

// Usage
char buf[1024];
int result = read_file("data.txt", buf, sizeof(buf));
if (result < 0) {
    printf("Error reading file\n");
} else {
    printf("Read %d bytes\n", result);
}
```

**Drawbacks:**

```c
// 1. Multiple error codes to track
// 2. Easy to forget to check
// 3. Lost information (can't return both data and error code)

// Workaround for #3:
int get_value(int *output) {
    if (calculation_failed()) {
        return -1;  // Error code
    }
    *output = result;
    return 0;  // Success
}

// Usage:
int value;
if (get_value(&value) != 0) {
    printf("Error\n");
} else {
    printf("Got %d\n", value);
}
```

---

**Q45: `errno` mechanism:**

**Answer:**

**errno** - Global variable containing last error code from standard library.

```c
#include <errno.h>
#include <stdio.h>
#include <string.h>

FILE *f = fopen("nonexistent.txt", "r");
if (f == NULL) {
    printf("Error: %s\n", strerror(errno));  // Print human-readable error
    // Or: perror("fopen");  // Prints "fopen: No such file or directory"
}
```

**Is `errno` thread-safe?**

Yes, in POSIX systems. Each thread has its own errno.

```c
#include <errno.h>
#include <pthread.h>

void *thread_func(void *arg) {
    FILE *f = fopen("missing.txt", "r");
    if (f == NULL) {
        int my_errno = errno;  // Each thread has own errno
        printf("Thread error: %d\n", my_errno);
    }
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, thread_func, NULL);
    pthread_create(&t2, NULL, thread_func, NULL);
    // ...
}
```

**Caveat:** errno is only valid immediately after error. Multiple calls may change it:

```c
FILE *f = fopen("bad", "r");
if (f == NULL) {
    printf("%d\n", errno);  // Some error code
    fopen("bad2", "r");     // Another failed open
    printf("%d\n", errno);  // Different error (from fopen above)
}
```

---

**Q46: `goto` for cleanup:**

**Answer:**

**Goto for error cleanup** - Used in Linux kernel to avoid deeply nested if/else.

```c
// Without goto (nested, hard to read)
int foo() {
    int *a = malloc(100);
    if (!a) {
        return -1;
    }

    int *b = malloc(200);
    if (!b) {
        free(a);
        return -1;
    }

    int *c = malloc(300);
    if (!c) {
        free(b);
        free(a);
        return -1;
    }

    // Success path
    // Process data...
    free(c);
    free(b);
    free(a);
    return 0;
}

// With goto (flattened, easier to read)
int foo() {
    int *a = malloc(100);
    if (!a) goto fail_a;

    int *b = malloc(200);
    if (!b) goto fail_b;

    int *c = malloc(300);
    if (!c) goto fail_c;

    // Success path
    // Process data...

    free(c);
    free(b);
    free(a);
    return 0;

fail_c:
    free(b);
fail_b:
    free(a);
fail_a:
    return -1;
}
```

**Why is this pattern used in Linux kernel?**

- Reduces indentation nesting
- Makes cleanup order clear
- Less error-prone (can't forget cleanup)
- Performance (single jump vs multiple condition checks)

**Alternatives to `goto`:**

```c
// Exception-style with setjmp/longjmp (not recommended)
// Callback-based error handling (function pointers)
// Defer-style (like Go) with macro cleanup stacks
```

---

## 8. TRICK / OUTPUT QUESTIONS

**Q47: What happens here?**
```c
char *p = "hello";
p[0] = 'H';
```

**Answer:** 
Segmentation fault or undefined behavior. String literals are stored in read-only memory (`.rodata` section). Attempting to write triggers a protection violation.

---

**Q48: What happens here?**
```c
int x;
if (x) {
    printf("true\n");
}
```

**Answer:**
Undefined behavior. `x` is uninitialized — contains garbage value from stack. The `if` will evaluate unpredictably.

---

**Q49: What happens here?**
```c
int a = -1;
unsigned int b = 1;
if (a < b) printf("less");
else printf("greater or equal");
```

**Answer:**
Prints "greater or equal". `-1` is promoted to unsigned, becoming `UINT_MAX` (~4 billion), which is greater than 1.

---

**Q50: What happens here?**
```c
char buf[5];
sprintf(buf, "%s", "Hello, World!");
```

**Answer:**
Buffer overflow. `sprintf` writes 14 bytes into 5-byte buffer, corrupting stack. Use `snprintf(buf, 5, ...)` instead.

---

*Complete C error reference for embedded systems interviews — covers all error categories from syntax to debugging strategies.*
