# C Interview Questions — Expert Level Q&A

> **Target:** Guru-Level (15+ Years) C / Systems / Platform Developer
> **Focus:** Memory internals, compiler mechanics, optimization, standards, debugging, architecture, lock-free systems, and deep language semantics

---

## MEMORY MANAGEMENT INTERNALS

# 1. How Does `malloc()` Internally Work?

## Answer

`malloc()` obtains memory from the OS and returns a pointer to usable space.

### Main Strategy: `sbrk` vs `mmap`

#### `sbrk` Approach

```text
Heap grows upward

Initial state:
    +---+
    | Heap end (break)
    +---+

After malloc(100):
    +-------+
    | Data  | <- returned to user
    +-------+
    | Heap end (break)
    +-------+
```

```c
void *malloc(size_t size)
{
    void *p = sbrk(size + METADATA);
    
    /* store metadata at start */
    store_metadata(p, size);
    
    /* return pointer past metadata */
    return (char *)p + METADATA;
}
```

#### `mmap` Approach

For large allocations:

```c
if (size > LARGE_THRESHOLD)
{
    return mmap(NULL, size, PROT_READ | PROT_WRITE,
                MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
}
```

Each large allocation gets its own memory region.

### Free List Management

When `free()` is called:

```c
void free(void *ptr)
{
    struct block *b = get_metadata(ptr);
    
    mark_free(b);
    
    merge_adjacent_free_blocks();
}
```

Maintains a **free list**:

```text
Allocated blocks: [used]
                  [used]
                  [FREE]
                  [used]
                  
Free list: -> [FREE] -> [FREE] -> NULL
```

On next `malloc()`:

```text
Scan free list
    |
    v
Find first block >= size
    |
    v
Split if too large
    |
    v
Mark as allocated
```

---

## Follow-Up: What Is Heap Metadata? How Is It Corrupted?

### Heap Metadata

Stored before or after user data:

```text
User's ptr ------> +--------+
                   | size   | <- metadata
                   +--------+
                   | free?  |
                   +--------+
                   | user   |
                   | data   |
                   | ...    |
                   +--------+
```

Metadata typically contains:

```text
Size of allocation
Free/allocated flag
Pointer to prev block
Pointer to next block
Magic number (for detection)
```

### Corruption Scenarios

#### 1. Buffer Overflow

```c
char buffer[10];

strcpy(buffer, "very long string");
```

Overwrites metadata of adjacent block.

#### 2. Use-After-Free Write

```c
int *p = malloc(100);
free(p);

*p = 999;  /* corrupts metadata */
```

#### 3. Allocation After Overflow

```c
char a[10];
int b[10];

sprintf(a, "%s", very_long_input);  /* overflow */

int *c = malloc(100);  /* c might point into corrupted heap */
```

#### 4. Double-Free

```c
free(p);
free(p);  /* metadata inconsistency */
```

### Detection

Guard bytes:

```c
#define GUARD 0xDEADBEEF

struct block {
    uint32_t guard_before;
    size_t size;
    uint32_t guard_after;
};
```

Check guards:

```c
if (block->guard_before != GUARD ||
    block->guard_after != GUARD)
{
    /* corruption detected */
    abort();
}
```

---

# 2. Difference Between Virtual and Physical Memory

## Answer

### Physical Memory (RAM)

Actual hardware memory:

```text
Address 0x0000
    |
    v
    [byte]
    [byte]
    [byte]
    ...
    [byte]
Address 0xFFFF
```

Available directly only in kernels/bare metal.

### Virtual Memory (User Space)

Abstraction provided by OS + MMU (Memory Management Unit).

```text
Virtual Address    Physical Address
    0x1000    --->    0x5000
    0x1004    --->    0x5004
    0x1008    --->    0x3000 (different page)
    0x100C    --->    0x3004
```

Each process has its own virtual address space.

### Mapping

The **page table** maps virtual to physical:

```text
Virtual Page Table   Physical
+-------+            +-------+
| 0x100 | --+        | 0x500 |
+-------+   |        +-------+
| 0x101 | --+---->   | 0x501 |
+-------+   |        +-------+
| 0x102 | --+        | 0x300 |
+-------+            +-------+
```

### Benefits

```text
Process isolation (one process can't access another's memory)
Larger apparent memory (disk backing)
Memory protection (read/write/execute bits)
Relocation (code can run at any physical address)
Swapping (infrequently used pages to disk)
```

### How It Works

1. CPU issues load from virtual address 0x1234
2. MMU looks up virtual page in page table
3. Page table entry points to physical page 0x5000
4. Actual read happens from physical 0x5234

### TLB (Translation Lookaside Buffer)

For performance, CPUs cache recent translations:

```text
Virtual 0x1234 -> Physical 0x5234 (cached)
```

If TLB miss, page table walk is required (slow).

---

# 3. Paging and Segmentation

## Answer

### Paging

Divides memory into **fixed-size pages** (typically 4 KB).

```text
Virtual memory:
+------+------+------+------+
| Page | Page | Page | Page |
| 0x0  | 0x1  | 0x2  | 0x3  | (4 KB each)
+------+------+------+------+

Physical memory:
+------+------+------+------+
| Page | Page | Page | Page |
| 0x5  | 0x1  | 0x7  | 0x2  |
+------+------+------+------+
```

Pages can be anywhere in physical memory.

#### Advantages

```text
No external fragmentation
Simple allocation
Page table supports sparse memory
Disk swapping efficient (page-sized units)
```

#### Disadvantages

```text
Internal fragmentation (up to 4 KB wasted per allocation)
Page table overhead
TLB misses cause slowdown
```

### Segmentation

Divides memory into **variable-size segments** (code, data, stack, heap).

```text
Segment 0: Code      (0x00000 - 0x10000)
Segment 1: Data      (0x20000 - 0x30000)
Segment 2: Stack     (0x40000 - 0x50000)
Segment 3: Heap      (0x60000 - 0x80000)
```

Segment table:

```text
Segment ID  Base      Limit
0           0x1000    0x10000
1           0x2000    0x10000
2           0x4000    0x10000
3           0x6000    0x20000
```

Access to segment S at offset O:

```text
Physical address = Base[S] + O

If O > Limit[S], fault
```

#### Advantages

```text
Logical organization
Protection (different access rights per segment)
No internal fragmentation
Sparse address spaces
```

#### Disadvantages

```text
External fragmentation
Variable-size segments hard to move
More complex memory management
```

### Modern Systems: Paging + Segmentation

Modern x86/ARM combines both:

```text
Virtual address
    |
    v
Segment table (translate to linear address)
    |
    v
Linear address
    |
    v
Paging (translate to physical address)
    |
    v
Physical address
```

But segmentation is largely legacy; paging is primary.

---

# 4. Memory Corruption Debugging Techniques

## Answer

### 1. AddressSanitizer (ASan)

Compiler instrumentation to detect memory errors:

```bash
gcc -fsanitize=address program.c -o program
./program
```

Detects:

```text
Buffer overflow
Use-after-free
Double-free
Leak detection
```

Output:

```text
==12345==ERROR: AddressSanitizer: heap-buffer-overflow
Address 0x619000000040 at pc 0x0000004cdd4b bp 0x7ffd11ce4900
```

### 2. Valgrind

Dynamic analysis tool:

```bash
valgrind --leak-check=full ./program
```

Detects:

```text
Memory leaks
Invalid reads/writes
Use-after-free
```

Slower but no recompilation needed (for most architectures).

### 3. GDB with Python Scripts

Debug interactively:

```bash
gdb ./program
(gdb) run
(gdb) backtrace
(gdb) frame 0
(gdb) print ptr
(gdb) x/32 ptr  /* examine memory */
```

### 4. Guard Bytes

In embedded systems:

```c
#define GUARD_BYTE 0xCD

void *guarded_malloc(size_t size)
{
    uint8_t *p = malloc(size + 2 * sizeof(int));
    
    *(int *)p = GUARD_BYTE;
    *(int *)(p + size + sizeof(int)) = GUARD_BYTE;
    
    return p + sizeof(int);
}
```

Check periodically:

```c
void check_guards(void *ptr)
{
    uint8_t *p = (uint8_t *)ptr - sizeof(int);
    
    if (*(int *)p != GUARD_BYTE)
        printf("Corruption before!\n");
    
    /* check after similarly */
}
```

### 5. Heap Dump

Print heap state:

```c
void dump_heap(void)
{
    struct heap_block *b = heap_start;
    
    while (b)
    {
        printf("Block %p: size=%zu, free=%d\n",
               b, b->size, b->free);
        b = b->next;
    }
}
```

### 6. Memory Watchpoints (GDB)

```gdb
(gdb) watch *0x7ffffffde000
(gdb) continue
(gdb) /* stops when memory changes */
```

### 7. Sanitizer Coverage

Instrument code to track execution:

```bash
gcc -fsanitize=address -fsanitize-coverage=func program.c
```

Useful for fuzzing.

---

# 5. Detecting Stack Corruption

## Answer

Stack corruption occurs when stack buffer is overflowed, overwriting return address or local variables.

### Runtime Detection Mechanisms

#### 1. Stack Canary

Place a secret value on stack:

```c
void function(void)
{
    uint32_t canary = 0xDEADBEEF;
    
    char buffer[100];
    
    /* buffer overflow might overwrite canary */
    
    if (canary != 0xDEADBEEF)
        abort();  /* corruption detected */
}
```

Many compilers emit this automatically:

```bash
gcc -fstack-protector function.c
```

Compiler places canary:

```text
Stack layout:
    +------------------+
    | return address   |
    +------------------+
    | saved rbp        |
    +------------------+
    | canary           | <- check before return
    +------------------+
    | local variables  |
    +------------------+
```

#### 2. Stack Guard Pages

OS allocates unreadable/unwritable page at stack boundary:

```text
     Guard page (inaccessible)
         |
         v
    +---------+
    | invalid |  <- access = segfault
    +---------+
    +---------+
    | stack   |
    | data    |
    +---------+
```

Any overflow beyond guard page causes fault.

Embedded systems may lack this (no OS).

#### 3. Instruction Limit

In embedded/RTOS, limit stack per task:

```c
xTaskCreate(task_func, "Task", stack_size, NULL, priority, &handle);
```

Tasks are pre-allocated stack; exceeding causes fault.

---

## Follow-Up: How Do You Implement a Stack Canary?

### Compiler-Generated (GCC)

```bash
gcc -fstack-protector-all program.c
```

Compiler inserts:

```c
void function(void)
{
    uint32_t __stack_chk_guard = <secret>;
    
    char buffer[100];
    
    /* ... */
    
    if (__stack_chk_guard != <secret>)
        __stack_chk_fail();
}
```

### Manual Implementation

```c
#define CANARY_VALUE 0x12345678

void function(void)
{
    uint32_t saved_canary = CANARY_VALUE;
    
    char buffer[100];
    
    /* work with buffer */
    
    if (saved_canary != CANARY_VALUE)
    {
        fprintf(stderr, "Stack corruption detected!\n");
        abort();
    }
}
```

### In Embedded Context

RTOS kernel checks canary at task switch:

```c
void task_switch(Task *current, Task *next)
{
    if (current->stack_canary != EXPECTED_VALUE)
    {
        printf("Stack overflow in task %s\n", current->name);
        system_fault();
    }
    
    switch_to_task(next);
}
```

---

## Follow-Up: How Do You Detect Stack Overflow at Runtime in Embedded?

### Method 1: Watermarking

Before entering userland, mark stack with pattern:

```c
void initialize_stack(Task *task)
{
    uint32_t *stack = (uint32_t *)task->stack_base;
    
    /* Fill entire stack with marker */
    for (int i = 0; i < task->stack_size / 4; i++)
        stack[i] = 0xCCCCCCCC;
}
```

Periodically check:

```c
void check_stack_usage(Task *task)
{
    uint32_t *stack = (uint32_t *)task->stack_base;
    int unused = 0;
    
    while (stack[unused] == 0xCCCCCCCC)
        unused++;
    
    int used = task->stack_size - (unused * 4);
    
    if (used > task->stack_size * 0.9)
        printf("High stack usage: %d/%d\n", used, task->stack_size);
}
```

### Method 2: Canary at Stack Boundary

Place canary at the end:

```c
void initialize_task_stack(Task *task)
{
    uint32_t *stack_top = (uint32_t *)
        ((uint8_t *)task->stack_base + task->stack_size - 4);
    
    *stack_top = STACK_CANARY;
}

void check_stack_overflow(Task *task)
{
    uint32_t *stack_top = (uint32_t *)
        ((uint8_t *)task->stack_base + task->stack_size - 4);
    
    if (*stack_top != STACK_CANARY)
        system_fault("Stack overflow");
}
```

### Method 3: Guard Page (with MMU)

If MMU available:

```c
void setup_task_stack(Task *task)
{
    /* Allocate stack + guard page */
    uint8_t *mem = malloc(STACK_SIZE + PAGE_SIZE);
    
    /* Make guard page inaccessible */
    mprotect(mem, PAGE_SIZE, PROT_NONE);
    
    /* Stack starts after guard */
    task->stack = mem + PAGE_SIZE;
    
    /* Overflow touches guard -> segfault */
}
```

### Method 4: Stack Usage Sampling (No Overhead)

Periodically sample current SP:

```c
void sample_stack_usage(void)
{
    uint32_t sp;
    
    asm("mov %%esp, %0" : "=r" (sp));
    
    int used = stack_base - sp;
    
    if (used > max_observed)
        max_observed = used;
}
```

No guards or canaries; just statistical tracking.

---

# 6. Heap Metadata Corruption

## Answer

Heap metadata is corrupted when:

1. Buffer overflow overwriting adjacent block's metadata
2. Unintended writes to heap structure pointers
3. Corruption of size/free flags
4. Freeing misaligned pointers

### Example Corruption

```c
struct block
{
    size_t size;
    int free;
    struct block *next;
};
```

User buffer overflow:

```c
char buffer[100];

strcpy(buffer, "very very very long string");
```

Overwrites adjacent block's metadata:

```text
Metadata        User buffer
+-----+-----+   +-----+-----+-----+
|size | free|   | str | corrupted
+-----+-----+   +-----+-----+-----+
```

Next `malloc()` or `free()` reads corrupted metadata and may crash.

### Detection Tools

#### 1. Valgrind

```bash
valgrind --track-origins=yes ./program
```

Reports:

```text
Invalid write of 4 bytes
  at 0x...: strcpy (in /lib/x86_64-linux-gnu/libc.so.6)
  by 0x...: main (program.c:15)
```

#### 2. AddressSanitizer

```bash
gcc -fsanitize=address ./program
```

Output:

```text
==12345==ERROR: AddressSanitizer: heap-buffer-overflow on unknown address
```

#### 3. Mudflap (GCC)

```bash
gcc -fmudflap ./program -lmudflap
```

Runtime bounds checking.

#### 4. Custom Guards

```c
#define HEAP_GUARD 0xFEEDBEEF

struct block
{
    uint32_t guard_before;
    size_t size;
    struct block *next;
    uint32_t guard_after;
};

void verify_block(struct block *b)
{
    if (b->guard_before != HEAP_GUARD ||
        b->guard_after != HEAP_GUARD)
    {
        printf("Heap corruption detected!\n");
        abort();
    }
}
```

---

## Follow-Up: What Tools Help? (Valgrind, ASan, Custom Guards)

### Valgrind

**Pros:**

```text
No recompilation
Detects leaks precisely
Memory origin tracking
Large suite of tools
```

**Cons:**

```text
Very slow (10-50x)
Not portable to embedded/baremetal
Large overhead
Needs Linux/Unix
```

**Best for:**

```text
Desktop development
Full system debugging
Comprehensive leak detection
```

### AddressSanitizer (ASan)

**Pros:**

```text
~2x slowdown (not 50x)
Detects buffer overflow at-the-moment
Excellent error messages
Integrated with GCC/Clang
Works on embedded Linux
```

**Cons:**

```text
Requires recompilation
Memory overhead (extra shadow memory)
May not work on baremetal
```

**Best for:**

```text
Continuous integration
Catching memory errors early
Embedded Linux systems
```

### Custom Guards

**Pros:**

```text
No external dependencies
Works on baremetal
Minimal overhead (can tune)
Portable
```

**Cons:**

```text
Manual implementation
Easy to miss corruption patterns
Adds code
```

**Best for:**

```text
Embedded systems
Production code (permanent guards)
Safety-critical applications
```

---

## COMPILER & ELF

# 7. How Does the Compiler Generate an Object File?

## Answer

### Compilation Stages

#### Stage 1: Preprocessing

```bash
input.c --[preprocessor]--> input.i (intermediate)
```

Expands:

```text
#include directives
#define macros
Conditional compilation
```

#### Stage 2: Compilation

```bash
input.i --[compiler]--> input.s (assembly)
```

Translates C to assembly:

```asm
main:
    pushq %rbp
    movq %rsp, %rbp
    movl $10, -4(%rbp)
    ...
```

#### Stage 3: Assembly

```bash
input.s --[assembler]--> input.o (object file)
```

Translates assembly to machine code:

```hex
55 48 89 e5 c7 45 fc 0a 00 00 00
```

#### Stage 4: Linking (separate process)

Multiple `.o` files combined into executable.

### Object File Structure (ELF)

An ELF object file contains:

```text
ELF Header
    |
    v
Program Headers (segments)
    |
    v
Sections
    |
    v
Section Headers Table
    |
    v
Symbol Table
    |
    v
String Table
    |
    v
Relocation Table
```

### Key Sections

- **.text**: Executable code
- **.data**: Initialized data
- **.rodata**: Read-only data (constants)
- **.bss**: Uninitialized data
- **.symtab**: Symbol table
- **.strtab**: String table
- **.reloc**: Relocation entries
- **.debug**: Debug information

### Symbol Table Entry

```c
struct Elf64_Sym {
    uint32_t st_name;     /* offset in string table */
    unsigned char st_info;/* symbol type/binding */
    unsigned char st_other;
    uint16_t st_shndx;    /* section index */
    uint64_t st_value;    /* address or offset */
    uint64_t st_size;     /* size in bytes */
};
```

For function `add()`:

```text
st_name   = offset to "add" in string table
st_info   = STT_FUNC (function)
st_value  = 0x00000000 (to be resolved)
st_size   = 32 (bytes)
```

### Relocation Entries

```c
struct Elf64_Rela {
    uint64_t r_offset;   /* where to relocate */
    uint64_t r_info;     /* relocation type */
    int64_t r_addend;    /* addend */
};
```

Example:

```c
extern int external_var;

void func(void)
{
    int x = external_var;  /* relocation needed */
}
```

Relocation entry:

```text
r_offset  = address of instruction using external_var
r_info    = R_X86_64_GLOB_DAT (global data relocation)
r_addend  = 0
```

Linker fills in actual address.

---

# 8. ELF File Structure

## Answer

Already covered above. Key components:

```text
+--------+
| ELF    |
| Header |
+--------+
|        |
| Sections (.text, .data, .bss, .symtab, etc.)
|        |
+--------+
| Section |
| Header  |
| Table   |
+--------+
```

---

## Follow-Up: Sections vs Segments

### Sections

Logical divisions for the compiler/linker:

```text
.text     (code)
.data     (initialized data)
.rodata   (read-only data)
.bss      (uninitialized)
.symtab   (symbols)
.strtab   (strings)
.debug    (debugging)
.reloc    (relocations)
```

Sections are linked/processed by linker.

### Segments

Divisions for the OS/loader:

```text
Readable segments
Writable segments
Executable segments
```

Example:

```text
Segment 0 (RX):
    .text
    .rodata
    
Segment 1 (RW):
    .data
    .bss
```

When executable is loaded:

```text
Segment 0 -> read, execute only (PROT_READ | PROT_EXEC)
Segment 1 -> read, write (PROT_READ | PROT_WRITE)
```

### Key Difference

```text
Sections: compiler/linker view
Segments: runtime/loader view
```

Linker combines sections into segments.

---

# 9. Relocation Process

## Answer

**Relocation** updates symbolic references to actual addresses.

### Example

```c
/* file1.c */
extern int global_var;

void func(void)
{
    global_var = 10;
}
```

```c
/* file2.c */
int global_var = 0;
```

### Compilation

file1.c compiles to:

```asm
mov $0xDEADBEEF, %eax
movl %eax, <global_var>
```

But `<global_var>` is unknown at compile time.

Object file contains:

```text
Instruction: mov $0xDEADBEEF, (offset)
Relocation:  type=R_X86_64_GLOB_DAT, offset=...
Symbol:      global_var (external)
```

### Linking

Linker:

1. Collects all symbols:

```text
Symbol: global_var from file2.o at address 0x601000
```

2. Updates all references:

```asm
mov $0xDEADBEEF, %eax
movl %eax, 0x601000  <- RELOCATED
```

Final executable has concrete addresses.

### Relocation Types

- **R_X86_64_PC32**: PC-relative 32-bit
- **R_X86_64_GLOB_DAT**: Global data
- **R_X86_64_JUMP_SLOT**: Dynamic linking
- Many others depending on architecture

---

# 10. Dynamic Loader Role

## Answer

The **dynamic loader** (ld.so) loads shared libraries at runtime.

### Process

```bash
$ ./executable
```

1. Kernel loads program header
2. Kernel transfers control to dynamic loader
3. Loader reads DYNAMIC segment:

```text
NEEDED: libc.so.6
NEEDED: libm.so.6
```

4. Loader searches for libraries in standard paths
5. Loader `mmap()`s libraries into memory
6. Loader performs relocations for PLT/GOT
7. Loader calls library constructors
8. Loader jumps to program entry point

### PLT and GOT

**Procedure Linkage Table (PLT)**: For functions

```asm
plt_entry:
    jmp *got_entry
    ...
```

**Global Offset Table (GOT)**: Contains actual addresses

```text
got[0] = address of printf
got[1] = address of malloc
got[2] = address of free
```

On first call to `printf()`:

```asm
call printf@plt
    |
    v
jmp *got[0]
    |
    v
lazy binding: resolve printf, fill GOT[0]
    |
    v
jump to printf
```

Subsequent calls:

```asm
call printf@plt
    |
    v
jmp *got[0]
    |
    v
goto printf (already resolved)
```

### Symbol Binding

**Lazy binding** (default):

```text
Symbol resolved on first call
Faster startup
```

**Eager binding**:

```bash
LD_BIND_NOW=1 ./executable
```

All symbols resolved upfront.

---

# 11. Position Independent Code (PIC)

## Answer

**PIC** means code can execute at any memory address without modification.

### Non-PIC (Old Approach)

```asm
mov $0x401000, %eax  /* hardcoded address */
```

If loaded at 0x400000 instead of 0x401000, instruction is wrong.

### PIC (Modern)

```asm
mov rip, %rax
add $0x1000, %rax    /* relative to current position */
mov (%rax), %eax
```

Address is calculated relative to instruction pointer.

### Compiler Generation

```bash
gcc -fPIC file.c  # Position-independent code
gcc -fpic file.c  # Small PIC (if possible)
```

### Benefits

- Shared libraries (libc.so) can be loaded at any address
- Address Space Layout Randomization (ASLR) security
- Multiple process instances of same library at different addresses

### PLT/GOT for Function Calls

To call external function:

**Without PIC:**

```asm
call 0x401000  /* hardcoded address */
```

**With PIC:**

```asm
call printf@plt  /* relative to GOT */
```

---

## Follow-Up: Why Is PIC Needed for Shared Libraries?

When multiple programs use libc.so:

```text
Program A loaded at 0x400000
    |
    libc.so loaded at 0x500000
    
Program B loaded at 0x600000
    |
    libc.so loaded at 0x700000  (different address!)
```

If libc.so has hardcoded addresses from when compiled, it won't work at 0x700000.

PIC makes the library work at any address:

```text
mov $code_section_offset, %rax
add $base_address, %rax
```

When loaded at 0x700000, automatic offset provides correct absolute address.

---

## UNDEFINED BEHAVIOR DEEP DIVE

# 12. Strict Aliasing Optimization Issue

## Answer

**Strict aliasing** is a rule that an object can only be accessed via a pointer of compatible type.

Example:

```c
int x = 10;

int *ip = &x;
float *fp = (float *)&x;

*fp = 3.14;  /* VIOLATION */

printf("%d\n", x);  /* x changed? */
```

Compiler assumes:

```text
int pointer
    |
    v
accesses int

float pointer
    |
    v
accesses float

They cannot be the same object
```

Result: Compiler optimizes assuming `x` is still 10.

Output:

```text
10
```

(Not the modified value.)

### Why Compiler Does This

Optimization example:

```c
void func(int *ip, float *fp)
{
    *ip = 10;
    float f = *fp;
    printf("%d\n", *ip);
}
```

Compiler sees:

```text
ip and fp are different types
Therefore, assuming strict aliasing, they don't overlap
Therefore, *ip assignment doesn't affect *fp
Therefore, printf uses cached value (10)
```

Optimized code:

```asm
mov $10, (%rdi)
movss (%rsi), %xmm0
mov $10, %eax
printf("%d\n", %eax)  /* uses cached 10, not re-read */
```

But if ip and fp actually point to same object, the cache is stale.

---

## Follow-Up: How Does Type-Punning via Union Differ from Pointer Cast?

### Pointer Cast (Violates Strict Aliasing)

```c
int x = 10;

float *fp = (float *)&x;

*fp = 3.14;  /* UNDEFINED BEHAVIOR */
```

Violates strict aliasing rule.

### Union (Defined Behavior)

```c
union Value
{
    int i;
    float f;
};

union Value v;

v.i = 10;
v.f = 3.14;
```

The standard permits this.

Accessing different members of a union is defined.

### Key Difference

Union is designed for this; pointer cast is not.

```text
Union: "intentional type punning"
Cast:  "accidental type punning"
```

Compiler trusts union; does not optimize away.

### Example with Cast

```c
int x = 10;

float f = *(float *)&x;

printf("x = %d, f = %f\n", x, f);
```

With `-fstrict-aliasing` (default in -O2):

```text
Output: x = 10, f = <garbage>
(x is not re-read; cached value used)
```

Without `-fstrict-aliasing`:

```bash
gcc -fno-strict-aliasing program.c
```

```text
Output: x = 10, f = <IEEE representation of bit pattern>
(x is re-read)
```

---

# 13. Integer Overflow UB

## Answer

Signed integer overflow is undefined behavior.

```c
int x = INT_MAX;

x++;  /* UB */
```

The compiler can:

```text
Wrap (like unsigned)
Saturate
Trap
Do anything
```

Unsigned integer overflow is defined (wraps):

```c
unsigned int x = UINT_MAX;

x++;  /* defined: wraps to 0 */
```

### Why the Difference?

Signed: Compiler assumes no overflow for optimization

Unsigned: Modular arithmetic is well-defined

### Optimization Example

```c
if (x + 1 < x)
{
    printf("Overflow!\n");
}
```

Compiler might reason:

```text
Signed overflow is UB
Therefore, x + 1 < x is impossible
Therefore, eliminate the branch
```

Optimized code:

```asm
/* branch removed entirely */
```

### Real Impact

```c
int count = read_count_from_sensor();

if (count > 0 && count < 1000000)
{
    count++;
    process(count);
}
```

Compiler might assume:

```text
count is positive and < 1000000
count + 1 cannot overflow (count < INT_MAX - 1)
```

If sensor somehow returns INT_MAX, it's UB.

---

## Follow-Up: Why Is Signed Overflow UB but Unsigned Overflow Defined?

### Unsigned Arithmetic

Mathematically defined as modular arithmetic (mod 2^N).

```text
UINT_MAX + 1 == 0
```

This is standard behavior for machine integers.

### Signed Arithmetic

No standard definition for overflow in most architectures.

Traditionally:

```text
x86: wrap
ARM: undefined
MIPS: undefined
```

Languages (C, C++) said:

```text
Overflow is UB
Let compiler optimize based on that
```

This permits aggressive optimizations:

```text
a + b always positive?
(if a, b are positive and no overflow)
```

---

# 14. Dangling Reference Behavior

## Answer

A **dangling reference** is a pointer to an object that has been deleted/freed.

```c
int *get_pointer(void)
{
    int x = 10;

    return &x;  /* dangling: x goes out of scope */
}

int main(void)
{
    int *p = get_pointer();

    printf("%d\n", *p);  /* use-after-free / dangling pointer */
}
```

Behavior is **undefined**.

### What Can Happen

```text
Access the freed/invalid memory location
Read garbage
Crash (segfault)
Appear to work (if memory not reused)
Cause intermittent failures
```

### Example

```c
void task_a(void)
{
    int local = 100;
    int *ptr = &local;
    
    task_b(ptr);  /* pass dangling pointer */
}

void task_b(int *ptr)
{
    printf("%d\n", *ptr);  /* might crash, might read garbage */
}
```

### Detection

**Valgrind:**

```bash
valgrind --track-origins=yes ./program
```

Reports:

```text
Use of uninitialised value of size 4
Invalid read of size 4
Address 0x... is 0 bytes inside a block of size ... free'd
```

**ASan:**

```bash
gcc -fsanitize=address ./program
```

Reports:

```text
use-after-poison
```

---

# 15. Misaligned Access Issue

## Answer

Accessing data at incorrect alignment can cause:

```text
Crashes (some architectures)
Slowdown (read-modify-write)
Unpredictable behavior
```

### Example

```c
struct __attribute__((packed)) Packed
{
    char c;      /* 1 byte */
    int i;       /* 4 bytes, misaligned */
};

struct Packed p;

p.i = 1000;  /* misaligned write */
```

On Cortex-M0: fault
On x86: works (slow)
On ARM (with alignment): fault

### UB Consideration

Is misaligned access UB?

```text
C standard: implementation-defined
Most platforms: UB
```

### Detection

**ASan:**

```bash
gcc -fsanitize=address -fsanitize=alignment
```

**Manual check:**

```c
if ((uintptr_t)ptr % alignment != 0)
{
    printf("Misaligned access!\n");
}
```

---

# 16. Volatile Optimization Edge Cases

## Answer

`volatile` prevents the compiler from optimizing away accesses, but has limitations.

### What `volatile` Prevents

Compiler cannot:

```text
Cache in register
Eliminate redundant accesses
Reorder volatile operations
```

### Example

```c
volatile int status = 0;

status = 1;
status = 1;  /* not eliminated */

int x = status;
int y = status;  /* both reads executed */
```

### What `volatile` Does NOT Prevent

**Atomic operations:** `volatile` is not atomic.

```c
volatile int counter = 0;

counter++;  /* NOT atomic; still UB in multithreaded context */
```

**Hardware memory barriers:** `volatile` doesn't guarantee CPU barriers.

```c
volatile int a, b;

a = 1;
b = 2;
```

CPU might reorder these on weakly-ordered architectures (ARM).

**Compiler barriers:** `volatile` provides some.

```c
asm volatile("" ::: "memory");
```

is stronger than `volatile int x`.

### Real-World Use

Hardware registers:

```c
volatile uint32_t *uart_status = (volatile uint32_t *)0x40000000;

if (*uart_status & RX_READY)
{
    printf("Data ready\n");
}
```

Without `volatile`:

```text
Compiler might: "status is constant, skip read"
```

With `volatile`:

```text
Compiler: must read from address every time
```

---

## Follow-Up: When Does `volatile` NOT Prevent Reordering?

### `volatile` Provides Compiler Barriers

```c
volatile int a = 0, b = 0;

a = 1;
b = 2;
```

Compiler guarantees:

```text
a = 1 executes before b = 2
```

But CPU might reorder (on ARM, PowerPC, etc.).

### Solution: Hardware Barriers

```c
volatile int a = 0, b = 0;

a = 1;
asm volatile("dsb" ::: "memory");  /* data synchronization barrier */
b = 2;
```

or use atomics:

```c
_Atomic int a = 0, b = 0;

atomic_store(&a, 1);
atomic_store(&b, 2);
```

Atomics automatically use hardware barriers.

### Modern Approach

In multi-threaded code:

```text
volatile: WRONG
_Atomic: CORRECT
```

`volatile` is only for:

```text
Hardware registers
Signal handlers (partially)
Device I/O
```

---

## C LIBRARY API DESIGN

# 17. How to Design a Scalable C Library?

## Answer

A scalable library must:

### 1. Define Clear API

Export only public symbols:

```c
/* public API */
int lib_init(void);
void lib_cleanup(void);
int lib_process(const char *input);

/* internal (static) */
static int internal_helper(void);
```

Use prefix for namespacing:

```text
lib_* for public
_lib_* for semi-public
(no prefix) for internal
```

### 2. Opaque Pointers (PIMPL)

Hide implementation details:

```c
/* library.h */

typedef struct LibContext LibContext;

LibContext *lib_create(void);
void lib_destroy(LibContext *ctx);
int lib_process(LibContext *ctx, const char *input);
```

```c
/* library.c */

struct LibContext
{
    int internal_state;
    void *buffer;
    /* ... */
};
```

Users cannot access members; only library can.

### 3. Version Management

```c
#define LIB_VERSION_MAJOR 2
#define LIB_VERSION_MINOR 1

int lib_get_version(int *major, int *minor);
```

### 4. Error Handling

Consistent error codes:

```c
typedef enum
{
    LIB_OK = 0,
    LIB_ERROR_INVALID_PARAM = -1,
    LIB_ERROR_ALLOC = -2,
    LIB_ERROR_STATE = -3,
} LibError;

LibError lib_process(const char *input);
```

### 5. Configuration

```c
typedef struct
{
    size_t buffer_size;
    int timeout;
    int flags;
} LibConfig;

int lib_init_ex(const LibConfig *config);
```

### 6. Thread-Safety

Document if library is reentrant:

```c
/**
 * Thread-safe: Yes
 * Multiple contexts can operate concurrently
 */
LibContext *lib_create(void);

/**
 * Not reentrant
 * Can only be called from one thread at a time
 */
int global_lib_process(const char *input);
```

### 7. Memory Management

Decide who allocates:

```c
/* Library allocates */
LibContext *ctx = lib_create();
lib_destroy(ctx);

/* Caller allocates */
LibContext ctx;
lib_init(&ctx);
lib_cleanup(&ctx);
```

### 8. Documentation

```c
/**
 * Process input data
 *
 * @param ctx: valid LibContext from lib_create()
 * @param input: non-NULL string
 * @return: LIB_OK on success, error code on failure
 *
 * Thread-safe: Yes
 * Reentrant: No (ctx is modified)
 *
 * Example:
 *   LibContext *ctx = lib_create();
 *   lib_process(ctx, "data");
 *   lib_destroy(ctx);
 */
int lib_process(LibContext *ctx, const char *input);
```

---

# 18. Opaque Pointers (PIMPL Pattern in C)

## Answer

**PIMPL** (Pointer to Implementation) hides internal details.

### Pattern

Header file:

```c
/* mylib.h */

typedef struct MyContext MyContext;  /* opaque */

MyContext *ctx_create(void);
void ctx_destroy(MyContext *ctx);
void ctx_set_value(MyContext *ctx, int value);
int ctx_get_value(MyContext *ctx);
```

Implementation file:

```c
/* mylib.c */

struct MyContext
{
    int value;
    int internal_flag;
    uint8_t buffer[1024];
};

MyContext *ctx_create(void)
{
    MyContext *ctx = malloc(sizeof(MyContext));
    ctx->value = 0;
    return ctx;
}

void ctx_destroy(MyContext *ctx)
{
    free(ctx);
}

void ctx_set_value(MyContext *ctx, int value)
{
    ctx->value = value;
}

int ctx_get_value(MyContext *ctx)
{
    return ctx->value;
}
```

User code:

```c
MyContext *ctx = ctx_create();

ctx_set_value(ctx, 42);  /* only way to access */

printf("%d\n", ctx_get_value(ctx));

ctx_destroy(ctx);
```

---

## Follow-Up: How Does This Achieve Encapsulation?

### Without PIMPL

```c
/* public header */

struct MyContext
{
    int value;
    int internal_flag;
    uint8_t buffer[1024];
};
```

Users access directly:

```c
ctx->value = 100;  /* direct access */
```

If implementation changes structure:

```c
struct MyContext
{
    int value;
    int new_field;  /* added */
    /* internal_flag removed */
    uint8_t buffer[512];  /* shrunk */
};
```

All user code breaks. ABI compatibility lost.

### With PIMPL

Users don't see structure. Only functions matter.

If implementation changes:

```c
struct MyContext  /* in .c file, not public */
{
    int value;
    int new_field;
    uint8_t buffer[512];
};
```

No user code breaks.

### ABI Stability

```text
PIMPL: Library can change internal structure
       Users recompile, code still works

Direct access: Library changing structure
               breaks every user's compiled code
```

---

# 19. ABI Compatibility

## Answer

**ABI** (Application Binary Interface) is the contract between library and calling code.

### What Breaks ABI

1. **Changing function signature**

```c
/* Old */
int lib_process(const char *input);

/* New */
int lib_process(const char *input, int flags);
```

Caller passes 1 argument, callee expects 2. Crash.

2. **Changing struct layout**

```c
/* Old */
struct Config
{
    int a;
    int b;
};

/* New */
struct Config
{
    int a;
    int x;  /* renamed */
    int b;
};
```

3. **Adding/removing symbols**

```bash
# Old libfoo exports foo_process()
# New libfoo removes foo_process()
# Existing binary calls foo_process() -> undefined symbol error
```

4. **Changing return type**

```c
/* Old */
int lib_get_version(void);

/* New */
uint64_t lib_get_version(void);
```

Caller expects return in eax/rax; stack layout changes.

---

## Follow-Up: What Breaks ABI? How Do You Maintain Backward Compatibility?

### Maintaining Compatibility

#### 1. Use Opaque Pointers

Internal changes don't break ABI:

```c
typedef struct LibContext LibContext;

LibContext *lib_create(void);
```

#### 2. Extend Structures via New Functions

Instead of changing struct:

```c
/* Old function */
int lib_init(const char *config_file);

/* New function (for new config) */
int lib_init_ex(const LibContext_Ex *config);
```

#### 3. Keep Old Functions

```c
/* Keep old (deprecated) */
int lib_process(const char *input);

/* Add new (enhanced) */
int lib_process_v2(const char *input, int flags);
```

#### 4. Use Version Symbols

```bash
nm libfoo.so | grep version
```

Export version-specific symbols:

```c
int lib_process_v1(void) { /* old implementation */ }
int lib_process_v2(void) { /* new implementation */ }

int lib_process(void) __attribute__((alias("lib_process_v2")));
```

#### 5. Versioned Linker Script

```linker
LIBFOO_1.0 {
    lib_process;
    lib_init;
};

LIBFOO_1.1 {
    lib_process_v2;
};
```

#### 6. Check ABI Compatibility

Tool:

```bash
abi-dumper libfoo_old.so -o libfoo_old.dump
abi-dumper libfoo_new.so -o libfoo_new.dump
abi-compliance-checker libfoo_old.dump libfoo_new.dump
```

Reports breaking changes.

---

# 20. Forward Declarations

## Answer

Forward declaration tells compiler "this exists elsewhere".

```c
typedef struct Node Node;  /* forward declaration */

Node *create_node(void);
```

User doesn't need definition of `Node`.

### Benefits

- Avoid circular dependencies
- Faster compilation (don't include full headers)
- PIMPL pattern (definition hidden)

### Limitation

Can't access members:

```c
Node *n = create_node();

/* invalid: */
printf("%d\n", n->value);

/* valid: */
int value = node_get_value(n);
```

---

# 21. Encapsulation in C

## Answer

C provides mechanisms (not as strong as C++):

### 1. Static Functions

```c
/* file.c */

static int internal_helper(void)
{
    return 42;
}

int public_function(void)
{
    return internal_helper();
}
```

`internal_helper` is private to the file.

### 2. Static Variables

```c
static int counter = 0;

void increment(void)
{
    counter++;
}
```

`counter` is module-scoped, not global.

### 3. Opaque Pointers

```c
typedef struct Impl Impl;

Impl *create(void);
void destroy(Impl *);
```

### 4. Const Qualification

```c
void modify_object(const Object *obj);
```

Caller cannot modify via this pointer.

### 5. Naming Conventions

```c
void public_api(void);
void _internal_api(void);
static void private_api(void);
```

### Limitations

Unlike C++, no access specifiers (public/private).

Relies on:

```text
Convention
Documentation
Static/extern keywords
```

---

# 22. Error Handling Design Patterns in C

## Answer

### Pattern 1: Return Codes

```c
int lib_process(const char *input)
{
    if (!input)
        return -1;
    
    if (allocation_failed())
        return -2;
    
    return 0;  /* success */
}
```

**Pros:**

```text
Simple
No global state
Easy to handle
```

**Cons:**

```text
Limited information (only int)
Easy to ignore
Caller must remember error codes
```

### Pattern 2: errno

```c
#include <errno.h>

int lib_process(const char *input)
{
    if (!input)
    {
        errno = EINVAL;
        return -1;
    }
    
    return 0;
}

/* caller */
int result = lib_process(input);
if (result < 0)
    perror("lib_process");
```

**Pros:**

```text
Standard mechanism
Rich error values
Portable
```

**Cons:**

```text
Global state (not thread-safe without __thread)
Side effects
Can be overwritten
```

### Pattern 3: Callback / Out Parameter

```c
typedef void (*ErrorCallback)(const char *msg, int code);

int lib_process(const char *input, ErrorCallback on_error)
{
    if (!input)
    {
        on_error("Invalid input", 1);
        return -1;
    }
    
    return 0;
}
```

**Pros:**

```text
Flexible error handling
Can log, recover, or propagate
Caller controls response
```

**Cons:**

```text
More verbose
Callback overhead
```

### Pattern 4: Exception-like (Setjmp/Longjmp)

```c
jmp_buf err_buf;

int lib_process(const char *input)
{
    if (!input)
        longjmp(err_buf, 1);
    
    return 0;
}

int main(void)
{
    if (setjmp(err_buf) != 0)
    {
        printf("Error occurred\n");
        return 1;
    }
    
    lib_process(NULL);
}
```

**Pros:**

```text
Non-local jumps for error handling
```

**Cons:**

```text
Messy stack unwinding
Hard to resource cleanup (no RAII)
Not recommended for normal use
```

---

## Follow-Up: Return Codes vs errno vs Callback — Tradeoffs

| Strategy  | Return Code | errno | Callback |
| --------- | ----------- | ----- | -------- |
| Simplicity | Simple     | Moderate | Complex |
| Thread-safe | Yes | No (unless `__thread`) | Yes |
| Info detail | Limited | Rich | Flexible |
| Ignoreability | Easy to ignore | Medium | Hard (forced) |
| Performance | Fast | Fast | Slight overhead (call) |
| Embedded | Preferred | Acceptable | For special cases |
| Legacy | Common | Very common | Less common |

**Best practice for embedded:**

Return codes + optional callback for logging:

```c
int lib_process(const char *input, ErrorCallback on_error)
{
    if (!input)
    {
        if (on_error)
            on_error("Invalid input", 1);
        return 1;
    }
    
    return 0;
}
```

Caller can ignore callback; still gets return code.

---

## ADVANCED DEBUGGING

# 23. Debug a Segmentation Fault — Systematic Approach

## Answer

Segfault: invalid memory access.

### Step 1: Reproduce

```bash
./program
```

If it crashes: good, we can debug.

If sporadic:

```bash
while true; do ./program; done
```

or:

```bash
gdb ./program
(gdb) run
(gdb) run
(gdb) run
```

### Step 2: Get Backtrace

```bash
gdb ./program
(gdb) run
# (crashes)
(gdb) backtrace
#0  0x00401234 in crash_function() at file.c:50
#1  0x00401345 in caller() at file.c:100
#2  0x00401456 in main() at file.c:200
```

### Step 3: Examine Crash Location

```bash
(gdb) frame 0
(gdb) list
(gdb) print ptr
(gdb) print *ptr
```

Look for:

```text
NULL pointer dereference
Invalid pointer
Out-of-bounds array access
Use-after-free
```

### Step 4: Check Data

```bash
(gdb) print ptr
$1 = (int *) 0x0
(gdb) print array
$2 = {10, 20, 30, ... }
```

### Step 5: Enable Core Dumps

```bash
ulimit -c unlimited
./program
# segfault generates core file

gdb ./program core
```

### Step 6: Use ASan

```bash
gcc -fsanitize=address program.c -o program
./program
```

Output:

```text
==12345==ERROR: AddressSanitizer: SEGV on unknown address 0x0
```

### Step 7: Use Valgrind

```bash
valgrind ./program
```

Output:

```text
Invalid write of 4 bytes
  at 0x...: crash_function (file.c:50)
```

---

# 24. Analyze a Core Dump

## Answer

Core dump is a snapshot of process memory at crash time.

### Enable Core Dumps

```bash
ulimit -c unlimited
```

### Trigger Crash

```bash
./program
# segfault
# generates core, core.123 (or similar)
```

### Analyze with GDB

```bash
gdb ./program core
(gdb) backtrace
(gdb) frame 0
(gdb) print <variables>
(gdb) x/32x 0x<address>  /* examine memory */
```

### Extract Information

```bash
file core
```

Shows:

```text
core.123: ELF 64-bit LSB core file, x86-64, version 1 (SYSV)
```

### Disassemble

```bash
(gdb) disassemble crash_function
```

shows:

```asm
0x00401234 <+0>: mov %eax, (%rdi)
0x00401237 <+3>: ret
```

### Check Memory Mapping

```bash
(gdb) info proc mappings
```

Shows:

```text
Start Addr End Addr Size Offset Perms objfile
0x400000 0x401000 0x1000 0 r-xp /path/to/program
0x600000 0x601000 0x1000 0x1000 r--p /path/to/program
```

---

# 25. Detect Memory Leak Without Valgrind

## Answer

### Method 1: Watermarking Allocations

```c
struct Alloc
{
    size_t size;
    const char *file;
    int line;
    uint32_t guard;
};

#define my_malloc(size) my_malloc_impl(size, __FILE__, __LINE__)

void *my_malloc_impl(size_t size, const char *file, int line)
{
    struct Alloc *a = malloc(sizeof(struct Alloc) + size);
    
    a->size = size;
    a->file = file;
    a->line = line;
    a->guard = 0xDEADBEEF;
    
    return a + 1;
}

void my_free(void *ptr)
{
    struct Alloc *a = (struct Alloc *)ptr - 1;
    
    if (a->guard != 0xDEADBEEF)
        fprintf(stderr, "Double-free or corruption\n");
    
    free(a);
}
```

Track allocations:

```c
struct AllocationList
{
    struct Alloc *list;
    size_t count;
};

void track_allocation(void *ptr)
{
    /* add to list */
}

void report_leaks(void)
{
    for (int i = 0; i < alloc_list.count; i++)
    {
        printf("Leak at %s:%d (%zu bytes)\n",
               alloc_list.list[i].file,
               alloc_list.list[i].line,
               alloc_list.list[i].size);
    }
}
```

### Method 2: Memory Statistics

```c
static size_t total_allocated = 0;
static size_t total_freed = 0;

void *my_malloc(size_t size)
{
    total_allocated += size;
    return malloc(size);
}

void my_free(void *ptr)
{
    /* need to track size somehow */
    free(ptr);
}

void print_memory_status(void)
{
    printf("Allocated: %zu\n", total_allocated);
    printf("Freed: %zu\n", total_freed);
    printf("Outstanding: %zu\n", total_allocated - total_freed);
}
```

### Method 3: Dump Allocations at Exit

```c
#include <atexit.h>

void report_at_exit(void)
{
    printf("Total allocations: %d\n", total_allocs);
    
    /* if total_freed < total_allocs, there are leaks */
}

int main(void)
{
    atexit(report_at_exit);
    
    /* program runs */
    
    return 0;
}
```

### Method 4: System Monitoring (Linux)

```bash
ps aux | grep program
# Shows RSS (resident set size)

watch -n 1 'ps aux | grep program'
# Monitor memory growth
```

### Method 5: Backtrace on Allocation

```c
#include <execinfo.h>

void *my_malloc(size_t size)
{
    void *ptrs[10];
    int n = backtrace(ptrs, 10);
    
    /* store backtrace with allocation */
    
    return malloc(size);
}
```

Print leaks with stack traces.

---

# 26. Diagnose a Random Crash in Optimized Builds

## Answer

Crash occurs at `-O2` but not `-O0`: likely UB or optimization-exposed bug.

### Step 1: Reproduce

```bash
gcc -O0 program.c -o program_no_opt
./program_no_opt
# (works)

gcc -O2 program.c -o program_opt
./program_opt
# (crashes)
```

### Step 2: Disable Optimizations One by One

```bash
gcc -O2 -fno-aggressive-loop-optimizations program.c
gcc -O2 -fno-tree-slp-vectorize program.c
gcc -O2 -fno-inline program.c
```

Find which optimization causes it.

### Step 3: Check for UB

Common patterns:

```c
int *p;  /* uninitialized */
if (p)
    *p = 10;  /* UB: p might be dereferenced */

int x = *y;  /* y uninitialized */

int a = INT_MAX;
a++;  /* signed overflow UB */
```

Run with:

```bash
gcc -fsanitize=undefined program.c
./program
```

### Step 4: Check for Race Conditions

```bash
gcc -fsanitize=thread program.c -pthread
./program
```

### Step 5: Use GCC UBSan

```bash
gcc -fsanitize=undefined -fsanitize-coverage=func program.c
./program
```

### Step 6: Binary Search

Narrow down crash location:

```bash
gcc -O2 -finstrument-functions program.c
```

Instrument each function; find where it crashes.

### Step 7: Examine Assembly

```bash
gcc -O2 -S program.c
cat program.s  /* look for suspicious patterns */
```

### Step 8: Use `-Wall -Wextra -Wpedantic`

```bash
gcc -Wall -Wextra -Wpedantic -O2 program.c
```

May warn about UB.

---

# 27. Detect Stack Overflow Without OS Support

## Answer

In bare-metal/embedded, no OS guard pages.

### Method 1: Canary at Stack Base

```c
#define STACK_CANARY 0xDEADBEEF

extern char _stack_start;  /* from linker script */

void init_stack_check(void)
{
    uint32_t *canary = (uint32_t *)&_stack_start;
    *canary = STACK_CANARY;
}

void check_stack(void)
{
    uint32_t *canary = (uint32_t *)&_stack_start;
    
    if (*canary != STACK_CANARY)
    {
        printf("STACK OVERFLOW!\n");
        system_reset();
    }
}
```

Periodically call `check_stack()` (e.g., in timer interrupt).

### Method 2: Watermark Stack

```c
void watermark_stack(void)
{
    extern char _stack_start, _stack_end;
    
    uint8_t *p = (uint8_t *)&_stack_start;
    
    while (p < (uint8_t *)&_stack_end)
    {
        *p = 0xCC;
        p++;
    }
}

void check_watermark(void)
{
    extern char _stack_start;
    
    uint8_t *p = (uint8_t *)&_stack_start;
    int unused = 0;
    
    while (*p == 0xCC)
    {
        unused++;
        p++;
    }
    
    int used = stack_size - unused;
    
    if (used > stack_size * 0.9)
        printf("Stack at %d%%\n", 100 * used / stack_size);
}
```

### Method 3: Static Stack Analysis

Compile with:

```bash
gcc -fstack-usage program.c
```

Generates `program.su` showing per-function stack usage.

Manually compute worst-case stack depth.

### Method 4: Measure Current SP

```c
uint32_t get_sp(void)
{
    uint32_t sp;
    
    asm("mov %%esp, %0" : "=r" (sp));
    
    return sp;
}

void log_stack_depth(void)
{
    static uint32_t max_sp = 0;
    
    uint32_t current_sp = get_sp();
    
    if (current_sp < max_sp)
        max_sp = current_sp;
    
    int depth = stack_base - max_sp;
    
    printf("Max stack used: %d\n", depth);
}
```

Call periodically to track peak usage.

---

# 28. Debug Optimized Code — Challenges and Techniques

## Answer

Optimized code is hard to debug:

### Challenges

1. **Variables optimized away**

```c
int x = 10;

printf("x = %d\n", x);
```

With optimization, `x` may not exist in registers/memory.

```bash
(gdb) print x
$1 = <optimized out>
```

2. **Instructions reordered**

Execution order doesn't match source.

3. **Inlined functions**

No frame for inlined function.

4. **Variable values incorrect**

Compiler may assume certain value based on UB.

### Techniques

#### 1. Compile with Debugging Symbols

```bash
gcc -O2 -g program.c
```

`-g` adds debug info; `-O2` still optimizes.

#### 2. Use `-Og`

```bash
gcc -Og program.c
```

Limited optimization that preserves debuggability.

#### 3. Set Watchpoints

```bash
(gdb) watch x
(gdb) continue
# (stops when x changes)
```

Even if optimized away, can set breakpoint before read.

#### 4. Examine Assembly

```bash
gcc -O2 -S program.c
cat program.s
```

Understand what instructions are generated.

#### 5. Use Reverse Debugging

Some GDB versions support reverse execution:

```bash
(gdb) set record full
(gdb) run
# (program runs, recorded)
(gdb) reverse-step  # execute backward
```

#### 6. Step Instruction-Level

```bash
(gdb) stepi
(gdb) nexti
```

Execute one assembly instruction at a time.

#### 7. Examine Memory Directly

```bash
(gdb) x/32x 0x<address>  # examine 32 words at address
```

---

## C STANDARDS

# 29. Differences Between C89, C99, C11, and C17

## Answer

### C89 (ANSI C)

Original standard (1989).

Key features:

```text
Basic types (int, char, float, double)
Pointers
Arrays
Functions
Structs, unions, enums
```

Limitations:

```text
No inline
No const/volatile
No void * (used char *)
No prototypes required
Implicit int return
```

### C99

Key additions:

```text
inline keyword
const/volatile
Variable-length arrays (VLAs)
Designated initializers
restrict keyword
_Bool type
long long
uint32_t, etc. (stdint.h)
snprintf, vsnprintf
```

Example:

```c
int array[n];  /* VLA (variable-length array) */

struct Point p = {.x = 10, .y = 20};  /* designated init */

int * restrict ptr;  /* restrict pointer */
```

### C11

Key additions:

```text
_Alignof, _Alignas
_Generic (type-generic macros)
_Static_assert
_Atomic types
_Pragma
Unicode support
Multithreading support
Anonymous structs/unions
```

Example:

```c
_Static_assert(sizeof(int) == 4, "int must be 4 bytes");

_Generic(x,
    int: handle_int,
    float: handle_float,
    default: handle_default
)(x);
```

### C17 (C18)

Minimal changes (defect fixes):

```text
Removed gets() function
Clarified undefined behavior
Removed _Noreturn from stdnoreturn.h
```

Essentially C11 with bug fixes.

### Adoption in Embedded

```text
C89: Legacy systems, limited toolchains
C99: Common in embedded Linux
C11: Modern embedded, RTOS
C17: Latest systems
```

---

## Follow-Up: What Features from C99/C11 Are Commonly Used in Embedded?

### C99

```c
/* Variable-length arrays */
void process_array(int n)
{
    int array[n];  /* allocate based on n */
}

/* Designated initializers */
struct Config cfg = {
    .mode = MODE_FAST,
    .timeout = 100,
};

/* uint32_t and friends */
uint32_t value;
int32_t signed_value;
```

### C11

```c
/* _Static_assert for compile-time checks */
_Static_assert(sizeof(int) == 4, "int size mismatch");

/* _Alignas for alignment */
char __attribute__((aligned(16))) aligned_buffer[256];

/* _Atomic for thread safety */
_Atomic int flag = 0;
atomic_store(&flag, 1);
```

### Less Common (But Useful)

```c
/* _Generic for type-generic macros */
#define MIN(a, b) _Generic((a), \
    int: min_int, \
    float: min_float \
)(a, b)

/* Anonymous structs (C11) */
struct Message {
    struct {
        uint8_t type;
        uint16_t length;
    };  /* no name, members directly accessible */
    uint8_t payload[256];
};
```

---

## Follow-Up: What Is `_Generic` in C11?

### Purpose

Implement type-generic functions (like C++ templates).

### Example

```c
#define ABS(x) _Generic((x), \
    int: abs, \
    long: labs, \
    long long: llabs, \
    float: fabsf, \
    double: fabs, \
    long double: fabsl \
)(x)

int a = -10;
double b = -3.14;

printf("%d\n", ABS(a));      /* calls abs() */
printf("%f\n", ABS(b));      /* calls fabs() */
```

### How It Works

Compiler determines type of expression at compile time.

Selects corresponding implementation.

---

## Follow-Up: What Is `_Static_assert`?

### Purpose

Assert at compile time (not runtime).

### Example

```c
_Static_assert(sizeof(int) == 4, "int must be 4 bytes");

_Static_assert(BUFFER_SIZE > 0, "buffer size must be positive");

_Static_assert(offsetof(struct Packet, payload) == 8,
               "payload offset incorrect");
```

If assertion fails, compilation stops with message.

### Use Case

Verify platform assumptions:

```c
_Static_assert(sizeof(uint32_t) == 4, "uint32_t broken");

_Static_assert(sizeof(int) >= 2, "int too small");
```

---

## Follow-Up: `restrict` Keyword (C99)

### Purpose

Tell compiler pointer doesn't alias with other pointers.

### Example

```c
void add_arrays(int *restrict a, int *restrict b, int *restrict c)
{
    for (int i = 0; i < 100; i++)
    {
        a[i] = b[i] + c[i];
    }
}
```

Compiler assumes:

```text
*a, *b, *c don't overlap
Can optimize: cache b[i], c[i] without worrying
```

### Without restrict

```c
void add_arrays(int *a, int *b, int *c)
{
    for (int i = 0; i < 100; i++)
    {
        a[i] = b[i] + c[i];
    }
}
```

Compiler must assume:

```text
b and a might overlap
c and a might overlap
Must re-read b[i], c[i] every iteration
```

### Violation

```c
int x[10], y[10];

add_arrays(x, x, y);  /* violates restrict contract */
```

Behavior is undefined.

---

# 30. `restrict` Keyword — How Does It Help Compiler Optimization?

## Answer

### Alias Analysis

Without `restrict`, pointers might alias:

```c
void func(int *a, int *b)
{
    a[0] = 10;
    printf("%d\n", b[0]);  /* could be 10 if a == b */
}
```

Compiler must assume worst case (aliasing).

With `restrict`:

```c
void func(int * restrict a, int * restrict b)
{
    a[0] = 10;
    printf("%d\n", b[0]);  /* definitely not 10 if a != b */
}
```

Compiler assumes no aliasing. Can optimize.

### Optimization Example

```c
void vec_add(float * restrict result,
             float * restrict a,
             float * restrict b)
{
    for (int i = 0; i < 4; i++)
        result[i] = a[i] + b[i];
}
```

**Without restrict:**

```asm
loop:
    load a[i]
    load b[i]
    add
    store result[i]
    (might interfere with a or b, re-read)
```

**With restrict:**

```asm
load all a[0..3]
load all b[0..3]
add all pairs
store all results[0..3]
```

Vectorization possible.

### Follow-Up: What Happens If You Violate the `restrict` Contract?

Behavior is **undefined**.

```c
int x[5];

void func(int * restrict a, int * restrict b)
{
    a[0] = 10;
    b[0] = 20;
}

func(x, x);  /* VIOLATES restrict: a and b are the same */
```

Compiler assumes no aliasing.

May generate code where:

```text
a[0] = 10
(b[0] cached as 10, compiler assumes not modified)
printf("%d\n", b[0]);  /* prints 10, not 20 */
```

---

# OBJECT-ORIENTED DESIGN IN C

# 31. How Do You Implement Object-Oriented Design Patterns in C?

## Answer

C doesn't have built-in OOP, but can be simulated.

### Encapsulation

Use opaque pointers:

```c
typedef struct Object Object;

Object *object_create(void);
void object_destroy(Object *);
void object_method(Object *, int arg);
```

### Inheritance via Struct Embedding

```c
struct Animal
{
    int age;
    const char *name;
};

struct Dog
{
    struct Animal base;  /* inherit */
    char breed[50];
};

void print_animal(struct Animal *a)
{
    printf("%s: %d\n", a->name, a->age);
}

struct Dog dog = {
    .base = {.age = 5, .name = "Rex"},
    .breed = "Labrador"
};

print_animal(&dog.base);  /* works */
```

### Polymorphism via Function Pointers

```c
struct Shape
{
    void (*draw)(struct Shape *);
    void (*destroy)(struct Shape *);
};

struct Circle
{
    struct Shape base;
    int radius;
};

void circle_draw(struct Shape *s)
{
    struct Circle *c = (struct Circle *)s;
    printf("Drawing circle: r=%d\n", c->radius);
}

struct Circle c = {
    .base = {.draw = circle_draw, .destroy = free},
    .radius = 10
};

c.base.draw(&c.base);  /* polymorphic call */
```

### Method Pointers with Context

```c
struct Button
{
    int x, y;
    void (*on_click)(struct Button *);
};

void button_click(struct Button *b)
{
    if (b->on_click)
        b->on_click(b);
}
```

### Virtual Table (vtable)

```c
struct VirtualTable
{
    void (*draw)(void *);
    void (*move)(void *, int, int);
};

struct Object
{
    struct VirtualTable *vtable;
};

struct Circle
{
    struct Object obj;
    int x, y, radius;
};

struct VirtualTable circle_vtable = {
    .draw = circle_draw,
    .move = circle_move,
};

struct Circle c = {
    .obj.vtable = &circle_vtable,
    .radius = 10
};

c.obj.vtable->draw(&c);  /* virtual call */
```

---

## Follow-Up: Polymorphism Using Function Pointers and Structs

### Example: Shape Hierarchy

```c
struct Shape
{
    void (*draw)(struct Shape *);
    void (*destroy)(struct Shape *);
};

void draw_shape(struct Shape *s)
{
    if (s && s->draw)
        s->draw(s);
}

void destroy_shape(struct Shape *s)
{
    if (s && s->destroy)
        s->destroy(s);
}
```

Implementations:

```c
struct Circle
{
    struct Shape shape;
    int radius;
};

void circle_draw(struct Shape *s)
{
    struct Circle *c = (struct Circle *)s;
    printf("Circle r=%d\n", c->radius);
}

void circle_destroy(struct Shape *s)
{
    free(s);
}

struct Circle *circle_new(int r)
{
    struct Circle *c = malloc(sizeof(*c));
    c->shape.draw = circle_draw;
    c->shape.destroy = circle_destroy;
    c->radius = r;
    return c;
}
```

Usage:

```c
struct Shape *shapes[] = {
    (struct Shape *)circle_new(5),
    (struct Shape *)rect_new(10, 20),
};

for (int i = 0; i < 2; i++)
    draw_shape(shapes[i]);
```

---

## Follow-Up: How Does Linux Kernel Use This Pattern?

### file_operations Structure

Kernel defines vtable for file types:

```c
struct file_operations
{
    int (*open)(struct inode *, struct file *);
    ssize_t (*read)(struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write)(struct file *, const char __user *, size_t, loff_t *);
    int (*close)(struct inode *, struct file *);
};
```

Different file types implement:

```c
/* regular file */
static const struct file_operations regular_fops = {
    .open = regular_open,
    .read = regular_read,
    .write = regular_write,
};

/* character device */
static const struct file_operations cdev_fops = {
    .open = cdev_open,
    .read = cdev_read,
};
```

Kernel calls generically:

```c
if (file->f_op->read)
    file->f_op->read(file, buf, count, offset);
```

Different implementations invoked polymorphically.

---

## Follow-Up: Inheritance via Struct Embedding

### Container Principle

```c
struct Base
{
    int x;
};

struct Derived
{
    struct Base base;
    int y;
};

struct Derived d = {.base.x = 1, .y = 2};

/* cast to parent */
struct Base *b = &d.base;

printf("%d\n", b->x);  /* works */
```

### Reverse: container_of

Recover derived from base:

```c
struct Derived *d = container_of(b, struct Derived, base);

printf("%d\n", d->y);
```

### Inheritance Hierarchy

```c
struct Animal { const char *name; };
struct Mammal { struct Animal animal; int fur_color; };
struct Dog { struct Mammal mammal; char breed[50]; };

struct Dog dog = {...};

/* cast up hierarchy */
struct Animal *a = &dog.mammal.animal;
```

---

## LOCK-FREE PROGRAMMING

# 32. What Is a Lock-Free Ring Buffer?

## Answer

A **lock-free ring buffer** allows concurrent reads/writes without mutexes.

### Structure

```c
struct RingBuffer
{
    uint8_t data[BUFFER_SIZE];
    volatile int head;
    volatile int tail;
};
```

Or with atomics:

```c
struct RingBuffer
{
    uint8_t data[BUFFER_SIZE];
    _Atomic int head;
    _Atomic int tail;
};
```

### Producer

```c
void rb_enqueue(struct RingBuffer *rb, uint8_t value)
{
    int next_tail = (rb->tail + 1) % BUFFER_SIZE;
    
    while (next_tail == rb->head)
    {
        /* buffer full, wait */
    }
    
    rb->data[rb->tail] = value;
    rb->tail = next_tail;
}
```

### Consumer

```c
uint8_t rb_dequeue(struct RingBuffer *rb)
{
    while (rb->head == rb->tail)
    {
        /* buffer empty, wait */
    }
    
    uint8_t value = rb->data[rb->head];
    rb->head = (rb->head + 1) % BUFFER_SIZE;
    
    return value;
}
```

### Why It's Lock-Free

- No mutexes
- Producer writes to one location (tail)
- Consumer reads from one location (head)
- No conflict if indices don't overlap

### Limitation: Size Must Be Power of 2

```c
#define BUFFER_SIZE 256  /* power of 2 */
```

Allows modulo via bitwise AND:

```c
rb->tail = (rb->tail + 1) & (BUFFER_SIZE - 1);
```

---

## Follow-Up: How Do You Implement It with `volatile` and Memory Barriers?

### Using volatile

```c
struct RingBuffer
{
    uint8_t data[256];
    volatile int head;
    volatile int tail;
};
```

`volatile` ensures:

```text
Compiler doesn't cache head/tail
Reads/writes happen in order
```

But CPU might still reorder (on ARM, PowerPC).

### Using Memory Barriers

```c
void rb_enqueue(struct RingBuffer *rb, uint8_t value)
{
    int tail = rb->tail;
    int next_tail = (tail + 1) & 0xFF;
    
    while (next_tail == rb->head)
    {
        /* wait */
    }
    
    rb->data[tail] = value;
    
    /* compiler + hardware barrier */
    asm volatile("dmb" ::: "memory");
    
    rb->tail = next_tail;
}

uint8_t rb_dequeue(struct RingBuffer *rb)
{
    while (rb->head == rb->tail)
    {
        /* wait */
    }
    
    uint8_t value = rb->data[rb->head];
    
    /* ensure data is read before advancing head */
    asm volatile("dmb" ::: "memory");
    
    rb->head = (rb->head + 1) & 0xFF;
    
    return value;
}
```

### Using Atomics (Preferred)

```c
struct RingBuffer
{
    uint8_t data[256];
    _Atomic int head;
    _Atomic int tail;
};

void rb_enqueue(struct RingBuffer *rb, uint8_t value)
{
    int tail = atomic_load_explicit(&rb->tail, memory_order_relaxed);
    int next_tail = (tail + 1) & 0xFF;
    
    while (next_tail == atomic_load_explicit(&rb->head, memory_order_acquire))
    {
        /* wait */
    }
    
    rb->data[tail] = value;
    
    atomic_store_explicit(&rb->tail, next_tail, memory_order_release);
}

uint8_t rb_dequeue(struct RingBuffer *rb)
{
    int head = atomic_load_explicit(&rb->head, memory_order_acquire);
    
    while (head == atomic_load_explicit(&rb->tail, memory_order_acquire))
    {
        /* wait */
    }
    
    uint8_t value = rb->data[head];
    
    atomic_store_explicit(&rb->head, (head + 1) & 0xFF, memory_order_release);
    
    return value;
}
```

Benefits:

```text
Explicit memory ordering
Portable across architectures
Self-documenting
```

---

# 33. Compare-and-Swap (CAS) Operations

## Answer

**Compare-and-Swap (CAS)** atomically:

1. Compare a memory location to an expected value
2. If equal, swap with new value
3. Return whether swap succeeded

### x86 Instruction

```asm
cmpxchg dest, source  /* compare and exchange */
```

### C Atomic Version

```c
_Atomic int counter = 0;

int old = 5;
int new = 10;

_Bool success = atomic_compare_exchange_strong(&counter, &old, new);

if (success)
    printf("Swapped from %d to %d\n", old, new);
else
    printf("Value was %d, not 5\n", old);
```

### Typical Use: Lock-Free Increment

```c
_Atomic int count = 0;

void increment(void)
{
    int old, new;
    
    do
    {
        old = atomic_load(&count);
        new = old + 1;
    } while (!atomic_compare_exchange_weak(&count, &old, new));
}
```

### Loop Pattern

```c
while (!cas(&counter, current, current + 1))
{
    current = load(&counter);
}
```

Try to increment; if failed, retry with new value.

---

# 34. ABA Problem

## Answer

**ABA Problem**: A memory location changes from A to B back to A, fooling CAS logic.

### Example

Two threads:

```text
Thread 1                    Thread 2

Load ptr = A
                            Load ptr = A
                            cas(ptr, A, B)  succeeds
                            cas(ptr, B, A)  succeeds
                            (back to A, but different object!)

cas(ptr, A, C)  succeeds
(thinks A is still the original)
```

But the original object at A may have been freed.

### Concrete Example

```c
struct Node
{
    int value;
    struct Node *next;
};

_Atomic(struct Node *) head = NULL;

void pop(void)
{
    struct Node *first;
    
    do
    {
        first = atomic_load(&head);
        if (!first)
            return;
    } while (!atomic_compare_exchange_weak(&head, &first, first->next));
    
    free(first);  /* ABA PROBLEM: first might have been pushed back */
}
```

Timeline:

```text
Thread 1: Load head = A
Thread 2: Load head = A
Thread 2: CAS(head, A, A->next) succeeds
Thread 2: free(A)
Thread 1: CAS(head, A, A->next) succeeds!
          Now head points to A->next, but A was freed
```

### Solutions

#### 1. Versioned Pointers

Include version counter:

```c
struct VersionedPtr
{
    struct Node *ptr;
    uint64_t version;
};

_Atomic(struct VersionedPtr) head = {NULL, 0};
```

After swap back to same address, version differs.

#### 2. Epoch-Based Reclamation

Don't free immediately; wait for safe point.

#### 3. Hazard Pointers

Threads announce which pointers they're using; safe to free only if no hazards.

#### 4. RCU (Read-Copy-Update)

Readers traverse safely; writers update with versioning.

### Modern Approach

Use lock-free library providing safe CAS, not raw CAS.

---

# 🔥 EXPERT-LEVEL TRICK QUESTIONS

# Trick Question 1

What is the output?

```c
int i = 0;
printf("%d %d %d", i++, i++, i++);
```

## Answer

**Undefined behavior.**

The order of evaluation of function arguments is unspecified.

Possible outputs:

```text
0 1 2
0 2 1
2 1 0
... (many other combinations)
```

All are valid.

---

# Trick Question 2

Difference between:

```c
char *p = "hello";
char p[] = "hello";
```

## Answer

```c
char *p = "hello";
```

- `p` is a pointer to a string literal (read-only)
- String literal is in `.rodata` section
- `p` is on stack (or in static storage)
- Cannot modify: `p[0] = 'H';` is UB

```c
char p[] = "hello";
```

- `p` is an array on the stack
- String is copied from `.rodata` to stack
- Can modify: `p[0] = 'H';` works

---

# Trick Question 3

Can `main()` return `void`?

## Answer

**No**, not per the standard.

```c
int main(void)
{
    return 0;
}
```

is required.

(Some compilers allow `void main()` as extension, but it's not standard.)

---

# Trick Question 4

Why is `sizeof(char)` always 1?

## Answer

By definition in the C standard.

From section 6.5.3.4:

> "... the unit for the size of objects of type `char`"

`sizeof(char)` is defined to be 1 by the language, not by hardware.

---

# Trick Question 5

Is NULL always 0?

## Answer

**No.**

Null pointer constant (`0` or `(void *)0`) is semantically NULL.

But in memory, the bit representation might not be all zeros.

Example (rare):

```text
On some architectures, NULL might be 0xFFFFFFFF in memory
But logically it's NULL, compared as 0
```

Standard says:

> "An integer constant expression with the value 0, or such an expression cast to type void *, is called a null pointer constant."

The representation is implementation-defined; the semantics are fixed.

---

# Trick Question 6

Why is an array name not modifiable?

## Answer

Array name decays to a pointer to the first element, which is an **rvalue** (non-modifiable lvalue).

```c
int a[10];

a = NULL;  /* ERROR: a is not an lvalue */

int *p = a;  /* OK: implicit decay */
```

From standard:

> "Except when it is the operand of the sizeof operator or the unary & operator, an lvalue that has type 'array of type' is converted to an expression with type 'pointer to type' that points to the initial element of the array object"

The pointer itself cannot be reassigned.

---

# Trick Question 7

Why does `sizeof` not evaluate its expression?

## Answer

`sizeof` is evaluated at compile-time (except VLAs in C99).

```c
int x = 5;

printf("%zu", sizeof(x++));  /* x is still 5 */
```

The compiler doesn't execute `x++`; it only checks the type of `x`.

---

# Trick Question 8

Why is `free(NULL)` safe?

## Answer

The C standard (section 7.22.3.3) guarantees:

> "If the pointer argument to free is a null pointer, no action occurs."

Safe to call multiple times with NULL.

```c
free(NULL);  /* no-op */
free(NULL);  /* safe */
```

---

# Trick Question 9

Can `malloc` return the same address again?

## Answer

**Yes.**

After `free()`, that memory can be reused.

```c
int *p1 = malloc(100);
free(p1);

int *p2 = malloc(100);
// p2 might equal p1 (same address reused)
```

Common pattern in pool allocators and standard implementations.

---

# Trick Question 10

Why is recursion risky in embedded systems?

## Answer

```text
Limited stack (maybe 2 KB on MCU)
No virtual memory (can't spill to disk)
Unpredictable stack depth
Hard to prove max depth
Crash if stack exceeded
```

Example:

```c
void traverse(struct Node *node)
{
    if (!node)
        return;
    
    traverse(node->left);   /* stack frame */
    traverse(node->right);  /* another frame */
}
```

For a deep tree, can overflow stack.

Safe approach:

```c
/* iterative with explicit stack */
struct Stack stack;
push(&stack, root);

while (!is_empty(&stack))
{
    struct Node *n = pop(&stack);
    process(n);
    if (n->left)
        push(&stack, n->left);
    if (n->right)
        push(&stack, n->right);
}
```

---

# Trick Question 11

What is the output?

```c
int a = 1, b = 2;
int c = a+++b;
```

## Answer

```text
c = 3
a = 2
b = 2
```

Parsed as:

```c
c = (a++) + b;
```

First, `a++ + b` evaluates to `1 + 2 = 3`.

Then, `a` is incremented to 2.

So `c = 3`, `a = 2`.

---

# Trick Question 12

What does this declare?

```c
int (*(*fp)(int))[10];
```

## Answer

`fp` is a **pointer to a function taking `int` and returning pointer to array of 10 `int`s**.

Breaking it down:

```text
(*(*fp)(int))   = "something" that takes int
                  and returns array pointer

[10]            = array of 10 ints

*               = pointer to that array

*fp             = pointer to a function

So: fp is pointer to (function returning pointer to array)
```

Example usage:

```c
int arr[10] = {0, 1, 2, ...};

int (*func(int x))[10]
{
    return &arr;
}

int (*(*fp)(int))[10] = func;

int (*p)[10] = (*fp)(5);  /* call via fp */
printf("%d\n", (*p)[0]);  /* access array */
```

---

# Trick Question 13

"You said `malloc` returns `void*` — why don't we cast it in C but we must in C++?"

## Answer

In C:

```c
int *p = malloc(100);  /* implicit void* -> int* conversion OK */
```

Standard allows implicit conversion from `void*` to any pointer type.

In C++:

```cpp
int *p = malloc(100);  /* ERROR: no implicit void* conversion */
int *p = (int *)malloc(100);  /* explicit cast required */
```

C++ requires explicit cast for type safety.

Reason:

```text
C: void* is generic pointer, trusted to convert
C++: void* is less trusted, forces explicit cast
```

---

# Trick Question 14

"You said `free(NULL)` is safe — what does the standard say about `realloc(NULL, size)`?"

## Answer

From the standard:

> "`realloc(NULL, size)` is equivalent to `malloc(size)`"

```c
void *p = realloc(NULL, 100);  /* same as malloc(100) */
```

So yes, `realloc(NULL, size)` is safe and works.

---

# Trick Question 15

What is the difference between `sizeof` and `_Alignof`?

## Answer

```c
sizeof(type)
```

Returns the size in bytes.

```c
_Alignof(type)
```

Returns the alignment requirement in bytes.

Example:

```c
struct Test
{
    char c;     /* 1 byte, 1-byte aligned */
    int i;      /* 4 bytes, 4-byte aligned */
};

printf("Size: %zu\n", sizeof(struct Test));     /* 8 (with padding) */
printf("Align: %zu\n", _Alignof(struct Test));  /* 4 (int's alignment) */
```

Structure alignment is determined by its largest member's alignment.

---

# END OF EXPERT-LEVEL C INTERVIEW QUESTIONS

These represent the deepest knowledge expected of a system-level C engineer with 15+ years of experience.

Key takeaways:

```text
Know the standards (C89, C99, C11, C17)
Understand undefined behavior deeply
Master memory management
Debug complex issues systematically
Design scalable APIs
Use atomics and barriers correctly
Think in systems, not just functions
Always profile and measure
```

Good luck with expert-level interviews!

