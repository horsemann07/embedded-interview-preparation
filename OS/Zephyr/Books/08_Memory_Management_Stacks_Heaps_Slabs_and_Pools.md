# Lesson 08 — Zephyr Memory Management: Stacks, Heaps, Memory Slabs, and Memory Pools

> **Level:** Beginner → Intermediate  
> **Environment:** Windows PowerShell + Zephyr 4.4 development tree + QEMU (`qemu_x86`)  
> **Learning application:** `zephyr/myapp/lesson08_memory_management/`  
> **Build results:** `zephyrproject/build/lesson08_memory_management/`  
> **Goal:** Understand where bytes go, who owns them, and how to avoid running out of RAM.

---

## 1. Why do we need memory management?

Imagine your microcontroller is a tiny school. It has a limited number of desks (RAM). Every student (thread or function) needs a place to keep notebooks (data). If you hand out too many desks, or forget to return them, there is no room for anyone else.

**Memory management** means choosing **where data lives**, **how much space it gets**, **how long it stays**, and **when the space can be reused**.

In embedded systems this is especially important: many microcontrollers have far less RAM than a laptop, and a surprise memory failure can stop a real-time task.

### Learning objectives

By the end you should be able to:

1. Explain static memory, stack memory, and dynamic allocation.
2. Use `K_THREAD_STACK_DEFINE()` and recognize the difference between stack size and heap size.
3. Allocate and release memory with `k_malloc()` / `k_free()`.
4. Make an application-owned heap with `K_HEAP_DEFINE()` and use `k_heap_alloc()` / `k_heap_free()`.
5. Make fixed-size blocks with `K_MEM_SLAB_DEFINE()` and use `k_mem_slab_alloc()` / `k_mem_slab_free()`.
6. Explain fragmentation, out-of-memory behavior, and why fixed-block memory is useful in embedded systems.
7. Understand what *memory pool* means in **current** Zephyr (and avoid obsolete examples).
8. Run everything on QEMU and compare Zephyr with FreeRTOS and ThreadX.

---

## 2. First, understand RAM with a picture

A **simplified**, conceptual RAM layout looks like this:

```text
+---------------------------------------------+
| RAM                                         |
|                                             |
|  Global/static data (.data / .bss)          |
|  +---------------------------------------+  |
|  | static int counter;                   |  |
|  +---------------------------------------+  |
|                                             |
|  Thread stacks (separate stack per thread)  |
|  +----------------+  +-----------------+    |
|  | main stack     |  | worker stack    |    |
|  +----------------+  +-----------------+    |
|                                             |
|  Heap regions (when configured)             |
|  +----------------+  +-----------------+    |
|  | system heap    |  | private k_heap  |    |
|  +----------------+  +-----------------+    |
|                                             |
|  Memory slab backing buffer                 |
|  +--------+--------+--------+               |
|  | 64 B   | 64 B   | 64 B   |               |
|  +--------+--------+--------+               |
+---------------------------------------------+
```

This picture is **not an exact address map**. The linker, board, architecture, and configuration decide the actual placement. Some systems have several physical RAM banks, protected regions, or externally attached memory.

### Key vocabulary

| Word | Meaning in simple language |
|---|---|
| RAM | Working memory that holds changing data while the device is powered. |
| Stack | A thread's working area for function calls and local variables. |
| Heap | A memory region that lets code request and return variable-size chunks during execution. |
| Static/global memory | Storage reserved for the lifetime of the application. |
| Memory slab | A box divided into equal-size reusable blocks. |
| Allocation | Asking for a piece of memory. |
| Freeing | Returning allocated memory so it can be used again. |
| Fragmentation | Free bytes are broken into pieces that may be too scattered for a large request. |
| Stack overflow | A thread uses more stack than it was given. |
| Memory leak | Allocated memory is no longer needed but is never returned. |

---

## 3. Static memory — the easiest and most predictable

```c
static uint8_t sensor_buffer[128];
static int sample_count;
```

These objects have storage for the entire life of the program.

**Benefits:** Simple, predictable, usually no allocator failure at runtime.

**Costs:** The reserved space is unavailable for other purposes even when you're not currently using the buffer.

> **Important:** `static` means *static storage duration*, not “stored in Flash”. Writable static variables normally use RAM. Constant objects might be kept in Flash/ROM depending on the platform and toolchain.

### When to use

If your embedded application **always** needs two 128-byte sensor buffers, static allocation is often the simplest choice.

---

## 4. Stack memory — each thread gets its own work table

Consider this function:

```c
void read_sensor(void)
{
    uint8_t sample[32] = {0};
    sample[0] = 42;
    printk("Sample: %u\n", sample[0]);
}
```

`sample` is an automatic local array. On typical embedded targets it uses the **current thread's stack**, unless the compiler optimizes or lays it out differently.

When the function returns, its local storage is no longer valid to access. **Never return a pointer to a local array and use it afterward.**

### Zephyr gives threads their own stacks

```c
#define WORKER_STACK_SIZE 1024

K_THREAD_STACK_DEFINE(worker_stack, WORKER_STACK_SIZE);
static struct k_thread worker_control;
```

- `K_THREAD_STACK_DEFINE()` reserves properly aligned memory for a thread stack.
- `struct k_thread` is a **thread control structure**; it is *not* the stack itself.
- Use `K_THREAD_STACK_SIZEOF(worker_stack)` when passing this stack to `k_thread_create()`. Do **not** assume ordinary `sizeof(worker_stack)` is always the correct value.

### Why not simply `uint8_t stack[1024]`?

Thread stacks may require special alignment, space for hardware guards, or architecture/user-mode metadata. Zephyr's stack macros know the rules for the target.

### A stack is not a heap

```text
Thread stack:                  Heap:
  Local variables              Requested allocations
  Function call frames         Can outlive the allocating function
  Mostly automatic lifetime    Must be released with correct API
  Stack overflow is dangerous  Allocation may return NULL
```

> **Practical rule:** Do not put huge local arrays in a small thread stack. Use a static buffer, a slab, or a heap when appropriate.

### Example: thread creation and its stack

```c
K_THREAD_STACK_DEFINE(worker_stack, 1024);
static struct k_thread worker_data;

void worker_fn(void *a, void *b, void *c)
{
    uint8_t local_bytes[32] = {0};
    local_bytes[0] = 7;
    printk("worker local: %u\n", local_bytes[0]);
}

void create_worker(void)
{
    k_thread_create(&worker_data,
                    worker_stack,
                    K_THREAD_STACK_SIZEOF(worker_stack),
                    worker_fn,
                    NULL, NULL, NULL,
                    5, 0, K_NO_WAIT);
}
```

**What if we give too small a stack?** A stack overflow can corrupt data or cause a fatal error. Hardware/software stack protection can help detect problems; it is not a substitute for choosing a sufficient stack size.

**What if we give too big a stack?** Valuable RAM may be wasted.

---

## 5. Heap memory — borrow any suitably sized box

Imagine a store where you can ask for 24 bytes now and 200 bytes later. The heap manages a shared collection of variable-size regions.

A heap allocator needs to track which parts are used and which parts are free.

```text
Heap before:
[               available memory                ]

Allocate 64 bytes:
[ used 64 ][             available               ]

Allocate 128 bytes:
[ used 64 ][ used 128 ][     available            ]

Free first allocation:
[ free 64 ][ used 128 ][     available            ]
```

### The Zephyr system heap

```c
void *ptr = k_malloc(100);

if (ptr != NULL) {
    /* use up to 100 requested bytes */
    k_free(ptr);
} else {
    printk("Allocation failed\n");
}
```

The system heap is configured with:

```ini
CONFIG_HEAP_MEM_POOL_SIZE=2048
```

This reserves a system heap of the configured size (subject to Zephyr's configuration rules and subsystem minimum requirements). It does **not** mean *every* heap in Zephyr is 2048 bytes. A private `k_heap` has its own storage.

### Important behavior

- `k_malloc(n)` returns a pointer to allocated memory or `NULL` on failure.
- `k_free(ptr)` returns it to the **system heap**.
- `k_malloc()` **does not automatically zero** your bytes. Initialize them before reading.
- Memory must remain allocated as long as some thread or driver needs it.
- Do not call `k_free()` on stack memory, static arrays, or slab blocks.

### Easy memory-leak example

**Wrong:**

```c
void bad_example(void)
{
    void *ptr = k_malloc(64);
    /* Oops: ptr is forgotten without k_free(ptr). */
}
```

**Correct:**

```c
void good_example(void)
{
    void *ptr = k_malloc(64);
    if (ptr == NULL) {
        return;
    }

    /* Use ptr. */
    k_free(ptr);
}
```

Real applications often free memory in a different function. That is okay, **if ownership and lifetime are clear**.

---

## 6. Dedicated heaps — your own reserved memory shop

Zephyr also lets you create a heap belonging to your application or subsystem:

```c
K_HEAP_DEFINE(my_heap, 2048);
```

Then:

```c
void *p = k_heap_alloc(&my_heap, 128, K_NO_WAIT);

if (p != NULL) {
    /* use memory */
    k_heap_free(&my_heap, p);
}
```

### Three useful timeout choices

| Timeout | What it means |
|---|---|
| `K_NO_WAIT` | Return immediately if sufficient memory is not available. |
| `K_MSEC(100)` | A thread can wait for freed memory, up to about 100 ms. |
| `K_FOREVER` | A thread can wait indefinitely for space to become available. |

Do **not** wait forever if nobody will ever return memory to that heap.

### Macro vs runtime initialization

| Method | API |
|---|---|
| Define heap with storage at build time | `K_HEAP_DEFINE(name, bytes)` |
| Initialize heap over existing memory at runtime | `k_heap_init(&heap, buffer, size)` |
| Allocate | `k_heap_alloc(&heap, bytes, timeout)` |
| Free | `k_heap_free(&heap, pointer)` |

**Never mix allocators:** memory returned by `k_heap_alloc(&my_heap, ...)` must be released with `k_heap_free(&my_heap, ...)`, **not** `k_free()`.

### System heap vs private heap

```text
k_malloc() -----------------------> Zephyr system heap
k_free()   <----------------------- same system heap

k_heap_alloc(&my_heap, ...) ------> your my_heap
k_heap_free(&my_heap, ptr) <------- same my_heap
```

### Low-level `sys_heap`

Zephyr also has `sys_heap`, an allocator without the normal `k_heap` thread synchronization. It is useful in specialized code, but callers must serialize access to the same heap themselves. For beginners and normal multi-threaded programs, start with **`k_heap`**.

---

## 7. Memory slabs — fixed-size reusable LEGO bricks

A slab is like a box containing three identical LEGO bricks. Each brick is always the same size.

```text
K_MEM_SLAB_DEFINE(packet_slab, 64, 3, 8)

+-----------+-----------+-----------+
| Block 0   | Block 1   | Block 2   |
| 64 bytes  | 64 bytes  | 64 bytes  |
+-----------+-----------+-----------+

Request #1 -> block
Request #2 -> block
Request #3 -> block
Request #4 -> FAIL or WAIT (no free block)

Free one block -> a later request can succeed
```

### Define one statically

```c
K_MEM_SLAB_DEFINE(packet_slab, 64, 3, 8);
```

Parameters:

| Parameter | Value | Meaning |
|---|---:|---|
| Name | `packet_slab` | Slab object name. |
| Block size | `64` | Each block holds 64 bytes. |
| Block count | `3` | Up to three allocations simultaneously. |
| Alignment | `8` | Each block is aligned to an 8-byte boundary. |

On 32-bit x86, `8` is a valid alignment, and 64 is divisible by 8. Block size and alignment must satisfy the API's architecture requirements.

### Allocate and free

```c
void *block = NULL;
int ret = k_mem_slab_alloc(&packet_slab, &block, K_NO_WAIT);

if (ret == 0) {
    /* block points at one 64-byte region */
    k_mem_slab_free(&packet_slab, block);
}
```

**Notice the pointer styles:**

- `k_mem_slab_alloc(&slab, &block, ...)` accepts the **address of the pointer** and writes the block address into it.
- `k_mem_slab_free(&slab, block)` receives the **block pointer itself**.

### Why use a slab?

**Advantages**

- Same block size every time: predictable amount of memory per allocation.
- Fast allocation/release for fixed-size objects.
- No variable-size *external fragmentation* inside this slab.
- An exhausted slab can make threads wait until a block is freed.

**Limitations**

- A request for 20 bytes still consumes one entire 64-byte block (internal waste).
- You cannot request one 200-byte block from a slab whose blocks are 64 bytes.
- You must choose block size and count ahead of time for a particular slab.
- If all blocks are used, allocation fails immediately or waits, depending on timeout.

### Example use cases

Sensor records, message descriptors, network packet metadata, fixed-size command objects, and other equal-size items.

### Runtime initialization version

You can also initialize an existing slab object over properly aligned storage using `k_mem_slab_init()`. The storage must remain alive and obey block-size/alignment rules. The macro version is easier for Lesson 08.

---

## 8. Memory pools: be careful with old tutorials!

The phrase **memory pool** often means *a reserved region of memory managed by an allocator*. But it has been used for **different Zephyr APIs over time**.

### Historical Zephyr API (do NOT use for this lesson)

Older articles may show:

```c
/* Historical/obsolete example: DO NOT COPY into Zephyr 4.4 */
K_MEM_POOL_DEFINE(...);
k_mem_pool_alloc(...);
k_mem_pool_free(...);
```

The old `k_mem_pool` / `sys_mem_pool` families were deprecated in the Zephyr 2.x era and replaced by `k_heap` / `sys_heap`. They should **not** be presented as the modern memory-management API.

### What should you use in modern Zephyr?

| Need | Preferred building block |
|---|---|
| Fixed-size object blocks | `k_mem_slab` |
| Variable-size allocations from a dedicated region | `k_heap` |
| Small dynamic allocations from the common system region | `k_malloc()` / `k_free()` |
| Unsynchronized low-level heap logic under your own locking | `sys_heap` |
| Multiple physical heap regions with custom selection | `sys_multi_heap` / shared multi-heap facilities |
| Allocate one or more fixed-size blocks (possibly scatter-gather) | `sys_mem_blocks` |

**Two different uses of “pool”:**

- `CONFIG_HEAP_MEM_POOL_SIZE` is still a valid **configuration name** for Zephyr's **system heap**.
- That does **not** mean the old `K_MEM_POOL_DEFINE()` API is still the right API.

### Tiny introduction to `sys_mem_blocks`

A memory-block allocator tracks fixed-size blocks and can hand out multiple blocks in one operation. It uses separate allocation bookkeeping, which is useful in some memory/DMA designs. This is an **advanced topic**; the QEMU experiment below stays with `k_mem_slab` and `k_heap`.

---

## 9. Stack vs heap vs slab vs static allocation

| Feature | Static/global | Thread stack | Heap (`k_heap`) | Slab (`k_mem_slab`) |
|---|---|---|---|---|
| Size chosen | Build time | Stack sized at creation | Request at runtime | Fixed block size, selected at definition |
| Lifetime | Whole program | Function scope / thread execution | Until freed | Until freed |
| Variable-size requests | No | Local objects vary, but stack limit fixed | Yes | No — one fixed block per allocation |
| Can fail at allocation time | Not normally | Overflow may occur during use | Yes | Yes |
| External fragmentation risk | No allocator involved | No allocator involved | Possible; allocator mitigates it | No variable-size external fragmentation |
| Typical use | Permanent buffers | Local variables | Variable-size payloads | Fixed-size packets/objects |
| Release method | None | Automatic when scope ends | `k_heap_free()` | `k_mem_slab_free()` |

**Note:** The table's heap release method describes **dedicated heaps**. The **system heap** uses `k_free()`.

### Beginner decision tree

```text
Need storage?
 |
 +-- Always the same buffer for the whole application?
 |       +-- static/global buffer
 |
 +-- Small temporary local variable inside a function?
 |       +-- stack (if it safely fits)
 |
 +-- Many equally sized objects borrowed and returned?
 |       +-- memory slab
 |
 +-- Different-size allocations at runtime?
         +-- system heap or dedicated k_heap
```

---

# 10. Hands-on QEMU Experiment — all four memory concepts

This is a **complete, runnable Zephyr application**. It deliberately:

1. Reads a global static buffer.
2. Creates one worker with a dedicated stack.
3. Allocates all blocks from a slab and shows a fourth request failing.
4. Releases a block and successfully allocates again.
5. Allocates and releases variable-sized chunks from a private heap.
6. Uses `k_malloc()` / `k_free()` from the system heap.
7. Waits for worker termination using `k_thread_join()`.

The experiment does **not** deliberately overflow a stack or corrupt memory.

## Step 1 — Project layout

All learning code stays under **your chosen folder**: `zephyr/myapp/`.

```text
zephyrproject/
├── .venv/
├── zephyr/                           <-- Zephyr source checkout
│   ├── kernel/
│   ├── drivers/
│   └── myapp/                        <-- All your learning apps
│       └── lesson08_memory_management/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson08_memory_management/   <-- Generated firmware
```

> `myapp/` is inside the Zephyr Git checkout because that is how you chose to organize your lessons. Avoid accidentally adding learning files to upstream Zephyr commits.

## Step 2 — Create the folder using PowerShell

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject
.\.venv\Scripts\Activate.ps1
cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson08_memory_management\src | Out-Null
```

**Reminder:** PowerShell activates with `Activate.ps1`, not `activate.bat`.

## Step 3 — Create `src/main.c`

Path:

```text
C:\Users\ragha\workspace\personal\zephyrproject\zephyr\myapp\lesson08_memory_management\src\main.c
```

Paste this **whole program**:

```c
#include <zephyr/kernel.h>
#include <errno.h>
#include <stdint.h>
#include <string.h>

#define WORKER_STACK_SIZE 1024
#define SLAB_BLOCK_SIZE    64
#define SLAB_BLOCK_COUNT    3

/* 1) Static/global memory: exists for the whole program. */
static uint8_t global_buffer[64];

/* 2) A correctly defined thread stack and control structure. */
K_THREAD_STACK_DEFINE(worker_stack, WORKER_STACK_SIZE);
static struct k_thread worker_data;

/* 3) Three reusable 64-byte blocks, each aligned to 8 bytes. */
K_MEM_SLAB_DEFINE(packet_slab,
                  SLAB_BLOCK_SIZE,
                  SLAB_BLOCK_COUNT,
                  8);

/* 4) A separate, application-owned heap. */
K_HEAP_DEFINE(private_heap, 2048);

static void demo_slab(void)
{
    void *blocks[SLAB_BLOCK_COUNT] = {NULL};
    void *extra = NULL;
    int rc;

    printk("\n[SLAB] Three fixed 64-byte blocks\n");

    for (int i = 0; i < SLAB_BLOCK_COUNT; i++) {
        rc = k_mem_slab_alloc(&packet_slab, &blocks[i], K_NO_WAIT);

        if (rc != 0) {
            printk("[SLAB] Block %d: unexpected failure (%d)\n", i + 1, rc);
            goto cleanup;
        }

        memset(blocks[i], i + 1, SLAB_BLOCK_SIZE);
        printk("[SLAB] Allocated block %d\n", i + 1);
    }

    /* All three blocks are in use: do not wait for a fourth. */
    rc = k_mem_slab_alloc(&packet_slab, &extra, K_NO_WAIT);
    if (rc == -ENOMEM) {
        printk("[SLAB] Fourth allocation failed: slab is full (expected)\n");
    } else if (rc == 0) {
        printk("[SLAB] Unexpected fourth allocation success\n");
        k_mem_slab_free(&packet_slab, extra);
    } else {
        printk("[SLAB] Fourth allocation returned %d\n", rc);
    }

    /* Return one block, then borrow another. */
    k_mem_slab_free(&packet_slab, blocks[0]);
    blocks[0] = NULL;
    printk("[SLAB] Freed one block\n");

    rc = k_mem_slab_alloc(&packet_slab, &extra, K_NO_WAIT);
    if (rc == 0) {
        printk("[SLAB] Allocation after free succeeded\n");
        k_mem_slab_free(&packet_slab, extra);
    } else {
        printk("[SLAB] Retry failed (%d)\n", rc);
    }

cleanup:
    for (int i = 0; i < SLAB_BLOCK_COUNT; i++) {
        if (blocks[i] != NULL) {
            k_mem_slab_free(&packet_slab, blocks[i]);
        }
    }
}

static void demo_private_heap(void)
{
    printk("\n[HEAP] Allocate different-size chunks\n");

    uint8_t *small = k_heap_alloc(&private_heap, 64, K_NO_WAIT);
    uint8_t *large = k_heap_alloc(&private_heap, 192, K_NO_WAIT);

    if (small != NULL) {
        memset(small, 0x11, 64);
        printk("[HEAP] Allocated 64 bytes\n");
    } else {
        printk("[HEAP] 64-byte allocation failed\n");
    }

    if (large != NULL) {
        memset(large, 0x22, 192);
        printk("[HEAP] Allocated 192 bytes\n");
    } else {
        printk("[HEAP] 192-byte allocation failed\n");
    }

    if (small != NULL) {
        k_heap_free(&private_heap, small);
        printk("[HEAP] Freed 64 bytes\n");
    }
    if (large != NULL) {
        k_heap_free(&private_heap, large);
        printk("[HEAP] Freed 192 bytes\n");
    }
}

static void demo_system_heap(void)
{
    printk("\n[SYSTEM HEAP] k_malloc() / k_free()\n");

    uint8_t *p = k_malloc(128);
    if (p == NULL) {
        printk("[SYSTEM HEAP] Allocation failed\n");
        return;
    }

    memset(p, 0xAB, 128);
    printk("[SYSTEM HEAP] Allocated 128 bytes\n");
    k_free(p);
    printk("[SYSTEM HEAP] Freed 128 bytes\n");
}

static void worker_fn(void *p1, void *p2, void *p3)
{
    /* This automatic local data belongs to this worker's stack. */
    uint8_t local_buffer[32] = {0};
    local_buffer[0] = 42;

    printk("\n[STACK] Worker running with its own stack\n");
    printk("[STACK] Local byte = %u\n", (unsigned int)local_buffer[0]);
    k_sleep(K_MSEC(20));
    printk("[STACK] Worker finished\n");
}

int main(void)
{
    printk("\n=== Lesson 08: Zephyr Memory Management ===\n");

    global_buffer[0] = 7;
    printk("[STATIC] Global byte = %u\n",
           (unsigned int)global_buffer[0]);

    demo_slab();
    demo_private_heap();
    demo_system_heap();

    printk("\n[THREAD] Creating worker...\n");
    k_tid_t id = k_thread_create(&worker_data,
                                 worker_stack,
                                 K_THREAD_STACK_SIZEOF(worker_stack),
                                 worker_fn,
                                 NULL, NULL, NULL,
                                 5, 0, K_NO_WAIT);

    if (id == NULL) {
        printk("[THREAD] Thread creation failed\n");
        return 0;
    }

    int rc = k_thread_join(id, K_FOREVER);
    printk("[THREAD] Join result = %d\n", rc);
    printk("\nLesson 08 complete!\n");

    return 0;
}
```

**Why this example is safe for beginners:** It checks allocations before use, frees every successfully borrowed block, uses `K_NO_WAIT` for deliberate out-of-memory tests, and waits for the worker before announcing completion.

> `k_thread_create()` is normally expected to return a valid thread ID when given valid statically defined arguments; the null check above is a defensive illustration, not a complete error-reporting mechanism for every invalid setup. Incorrect thread setup can be a kernel programming error rather than a recoverable `NULL` return.

## Step 4 — Create `prj.conf`

Path: `zephyr/myapp/lesson08_memory_management/prj.conf`

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
CONFIG_HEAP_MEM_POOL_SIZE=2048
CONFIG_MAIN_STACK_SIZE=2048
```

| Setting | Why it is here |
|---|---|
| `CONFIG_MULTITHREADING=y` | Enable normal thread scheduling. |
| `CONFIG_PRINTK=y` | Print easy-to-read QEMU console messages. |
| `CONFIG_HEAP_MEM_POOL_SIZE=2048` | Reserve system heap space for `k_malloc()`. |
| `CONFIG_MAIN_STACK_SIZE=2048` | Give the main thread comfortable space for this lesson. |

**Important:** The private heap is already defined separately with `K_HEAP_DEFINE(private_heap, 2048)` in `main.c`.

## Step 5 — Create `CMakeLists.txt`

Path: `zephyr/myapp/lesson08_memory_management/CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson08_memory_management)

target_sources(app PRIVATE src/main.c)
```

## Step 6 — Build the firmware

From `zephyrproject/zephyr` in your **activated PowerShell**:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson08_memory_management -d ..\build\lesson08_memory_management
```

`-p always` creates a clean build so the build directory won't accidentally continue pointing at another lesson (you encountered this when Lesson 04 ran Lesson 03).

## Step 7 — Run on QEMU

```powershell
west build -d ..\build\lesson08_memory_management -t run
```

To exit an interactive QEMU console: **Ctrl+A**, release, then **X**.

### Expected output (representative, not exact timing)

```text
SeaBIOS (...)
Booting from ROM..
*** Booting Zephyr OS build ... ***

=== Lesson 08: Zephyr Memory Management ===
[STATIC] Global byte = 7

[SLAB] Three fixed 64-byte blocks
[SLAB] Allocated block 1
[SLAB] Allocated block 2
[SLAB] Allocated block 3
[SLAB] Fourth allocation failed: slab is full (expected)
[SLAB] Freed one block
[SLAB] Allocation after free succeeded

[HEAP] Allocate different-size chunks
[HEAP] Allocated 64 bytes
[HEAP] Allocated 192 bytes
[HEAP] Freed 64 bytes
[HEAP] Freed 192 bytes

[SYSTEM HEAP] k_malloc() / k_free()
[SYSTEM HEAP] Allocated 128 bytes
[SYSTEM HEAP] Freed 128 bytes

[THREAD] Creating worker...

[STACK] Worker running with its own stack
[STACK] Local byte = 42
[STACK] Worker finished
[THREAD] Join result = 0

Lesson 08 complete!
```

The exact boot banner and thread scheduling/timing depend on your Zephyr build.

---

## 11. What happens inside the allocator?

### Slab: a free list of whole blocks

```text
Initially:  Block A -> Block B -> Block C -> end
Allocate:   Give caller Block A
            Free list now B -> C -> end
Free A:     Return A to the free list
```

Zephyr tracks whole equal-size blocks. It does not split a block into smaller variable-size allocations.

### Heap: metadata + free regions

```text
Heap initially: [                  free                ]
Alloc 64:       [ used ][          free                 ]
Alloc 128:      [ used ][ used ][   free                 ]
Free first:     [ free ][ used ][   free                 ]
```

A heap tracks region sizes and may combine adjacent freed regions. Some free memory may be unusable for a particular large request if it is too fragmented or allocator bookkeeping takes space.

### Memory ownership rule

```text
Who allocated it?                  Who must release it?
-----------------------------------------------------------------
k_malloc()                         k_free()
k_heap_alloc(&private_heap, ...)   k_heap_free(&private_heap, ...)
k_mem_slab_alloc(&slab, ...)       k_mem_slab_free(&slab, ...)
Local stack buffer                 Nobody manually frees it
Static/global array               Nobody manually frees it
```

**Do not free a pointer twice.** **Do not use a pointer after freeing it.** Either mistake can corrupt a running embedded system.

---

## 12. Why fragmentation matters

Imagine a heap with 1000 free bytes spread across ten little gaps. Your program requests 300 **contiguous** bytes.

Even though 1000 bytes are free in total, the allocator might not find a 300-byte continuous region, so the request may fail.

```text
[100 free][used][100 free][used][100 free][used] ...

Total free memory: maybe large
Largest individual available gap: small
```

**Slabs solve this particular problem by using equal-size blocks**, but they can waste space *inside* each block. One storage strategy isn't best for every need.

### External vs internal fragmentation

- **External fragmentation:** Free space is separated into holes between active allocations.
- **Internal fragmentation:** You reserve a 64-byte slab block for a 20-byte record, leaving 44 bytes unused inside that block.

### Rules of thumb for real-time code

1. Prefer static allocation when object counts and sizes are known.
2. Consider slabs for many same-size packets or records.
3. Keep heap allocations out of your most timing-critical operations when possible.
4. Set clear owners and lifetimes for dynamically allocated objects.
5. Check failures and plan graceful fallback.
6. Measure RAM and stack usage on the actual target, not just QEMU.

---

## 13. Macro-based vs API-based allocation

| Object | Define at build time | Initialize existing storage at runtime | Allocate / get | Free / return |
|---|---|---|---|---|
| Thread stack | `K_THREAD_STACK_DEFINE()` | `k_thread_stack_alloc()` is a separate dynamic stack allocator | Passed to `k_thread_create()` | Dynamic stacks use `k_thread_stack_free()` after safe termination |
| System heap | Kconfig (`CONFIG_HEAP_MEM_POOL_SIZE`) | Managed by the system | `k_malloc()` | `k_free()` |
| Dedicated heap | `K_HEAP_DEFINE()` | `k_heap_init()` | `k_heap_alloc()` | `k_heap_free()` |
| Memory slab | `K_MEM_SLAB_DEFINE()` | `k_mem_slab_init()` | `k_mem_slab_alloc()` | `k_mem_slab_free()` |

**Careful:** `K_THREAD_STACK_DEFINE()` defines a *stack region*, not a generic malloc/free heap. The dynamic thread-stack allocator has additional architecture and configuration requirements.

---

## 14. FreeRTOS vs ThreadX vs Zephyr — memory management

| Feature | FreeRTOS | ThreadX | Zephyr |
|---|---|---|---|
| Task/thread stack | Per-task stack assigned by creation APIs | Caller provides stack storage during thread creation | Stack macros (`K_THREAD_STACK_DEFINE`) or supported dynamic stack APIs |
| Dynamic general-purpose allocation | `pvPortMalloc()` / `vPortFree()`; behavior depends on heap scheme | Usually application-managed pools or C library allocator | `k_malloc()` / `k_free()` system heap |
| Variable-size dedicated allocator | FreeRTOS heap implementation or application allocator | **Byte pool** (`tx_byte_pool_create`, `tx_byte_allocate`, `tx_byte_release`) | `K_HEAP_DEFINE`, `k_heap_alloc`, `k_heap_free` |
| Fixed-size allocator | Static buffers or application allocator (no direct standard Zephyr-style slab object) | **Block pool** (`tx_block_pool_create`, `tx_block_allocate`, `tx_block_release`) | `K_MEM_SLAB_DEFINE`, `k_mem_slab_alloc`, `k_mem_slab_free` |
| Static task/thread creation | `xTaskCreateStatic()` | Object + stack memory supplied by caller | `K_THREAD_DEFINE()` or `k_thread_create()` with defined stack |
| Common beginner failure | Missing `vPortFree()` or insufficient task stack | Byte/block pool exhausted or incorrect buffer lifetime | `NULL`, slab `-ENOMEM`, insufficient thread stack |
| Fragmentation consideration | Depends on selected heap strategy | Byte pool variable-size allocations can fragment | Heap can fragment; slabs avoid variable-size external fragmentation |

### ThreadX: two different pool types

**ThreadX block pool = equal-sized blocks** (conceptually similar to a Zephyr slab).

```c
/* ThreadX-style API names (illustrative, not Zephyr code) */
tx_block_pool_create(...);
tx_block_allocate(...);
tx_block_release(...);
```

**ThreadX byte pool = requests for different sizes** (conceptually similar to a heap).

```c
/* ThreadX-style API names (illustrative, not Zephyr code) */
tx_byte_pool_create(...);
tx_byte_allocate(...);
tx_byte_release(...);
```

### FreeRTOS: know the configured heap scheme

Common FreeRTOS projects choose an allocator implementation such as `heap_4.c` or `heap_5.c`. `pvPortMalloc()` and `vPortFree()` behavior depends on that choice, so don't assume every FreeRTOS application uses the same heap arrangement.

**Easy memory trick:**

```text
FreeRTOS:  task stack + selected heap_N allocator
ThreadX:   thread stack + byte pool / block pool
Zephyr:    thread stack + system/private heap + fixed-block slab
```

These are **conceptual mappings**, not drop-in API replacements.

---

## 15. Common bugs and how to fix them

| Problem | Possible reason | What to check |
|---|---|---|
| `k_malloc()` returns `NULL` | System heap missing, too small, or exhausted | `CONFIG_HEAP_MEM_POOL_SIZE`, available RAM, previous allocations |
| `k_heap_alloc()` returns `NULL` | No sufficiently large region is available before timeout | Heap size, outstanding allocations, fragmentation, timeout |
| `k_mem_slab_alloc()` returns `-ENOMEM` with `K_NO_WAIT` | No free slab block | Block count, releases, duplicate ownership |
| Slab allocation wait times out | No block becomes free in time | Threads that should free blocks, timeout |
| Hard fault / stack overflow | Thread uses too much stack or memory was corrupted | Stack sizes, recursion, large local arrays, instrumentation |
| Heap corruption | Double free, freeing to wrong heap, or writing out of bounds | Match alloc/free pairs; use valid buffer lengths |
| Wrong lesson starts in QEMU | Existing CMake cache references old application | Pristine build with correct application path |
| Compile error about old memory pool macros | Tutorial uses removed `k_mem_pool` API | Replace with `k_heap` or `k_mem_slab` as appropriate |

### Diagnostic commands from your `zephyr/` directory

```powershell
# Confirm the example source exists
Get-Content .\myapp\lesson08_memory_management\src\main.c

# Check relevant generated config after build
Select-String -Path ..\build\lesson08_memory_management\zephyr\.config -Pattern 'CONFIG_HEAP_MEM_POOL_SIZE','CONFIG_MAIN_STACK_SIZE'

# Inspect the linker memory map when present
Get-Item ..\build\lesson08_memory_management\zephyr\zephyr.map

# Confirm which application CMake built
Select-String -Path ..\build\lesson08_memory_management\CMakeCache.txt -Pattern 'APPLICATION_SOURCE_DIR'
```

---

## 16. Optional experiment — measure stack usage

Zephyr's **Thread Analyzer** can help estimate per-thread stack usage. Enable it using an **optional** overlay or by adding Kconfig options to `prj.conf`, for example:

```ini
CONFIG_THREAD_ANALYZER=y
CONFIG_THREAD_ANALYZER_USE_PRINTK=y
CONFIG_THREAD_ANALYZER_AUTO=y
CONFIG_THREAD_ANALYZER_AUTO_INTERVAL=5
```

The analyzer may select or require additional kernel configuration automatically. Run a **pristine rebuild** after changing Kconfig.

> This Lesson 08 demo finishes quickly. With an automatic 5-second reporting interval you may not see a report before it ends. To investigate stack usage, make the worker run longer or use the on-demand analyzer API in a separate experiment. Stack numbers from QEMU are not a substitute for measurements on your eventual STM32 hardware.

Other useful documentation topics: `CONFIG_INIT_STACKS`, `CONFIG_THREAD_STACK_INFO`, and `k_thread_stack_space_get()`.

### Optional challenge: measure manually

1. Build the application with sufficient instrumentation.
2. Run the analyzer while threads are alive.
3. Increase local buffer sizes *gradually*, without intentionally overflowing the stack.
4. Compare the observed stack usage and configured stack sizes.
5. Restore the original example afterward.

---

## 17. Kernel source tour — follow the real implementation

These locations are useful for exploring your Zephyr checkout. File organization can change with version.

| Concept | Suggested place to explore |
|---|---|
| Memory slab internals | `zephyr/kernel/mem_slab.c` |
| Kernel heap interface | Search for `k_heap_alloc` in `kernel/` and `include/zephyr/` |
| Low-level heap | Search for `sys_heap` in `lib/heap/` and `include/zephyr/sys/` |
| Thread stack definitions | `zephyr/include/zephyr/kernel/thread_stack.h` |
| Thread creation | `zephyr/kernel/thread.c` |
| Heap and slab API declarations | `zephyr/include/zephyr/kernel.h` and related headers |
| Memory management docs | `zephyr/doc/kernel/memory_management/` |

### PowerShell search commands

From inside `zephyrproject/zephyr`:

```powershell
Select-String -Path .\kernel\mem_slab.c -Pattern 'k_mem_slab_alloc','k_mem_slab_free'
Select-String -Path .\include\zephyr\kernel.h -Pattern 'K_MEM_SLAB_DEFINE','K_HEAP_DEFINE'
Get-ChildItem .\lib\heap -Recurse -File | Select-Object -ExpandProperty FullName
```

If a symbol has moved to a different header in your checkout, VS Code's **Go to Definition** is a useful way to track it.

---

## 18. Practice exercises

### Exercise A — change slab capacity

Change:

```c
#define SLAB_BLOCK_COUNT 3
```

to:

```c
#define SLAB_BLOCK_COUNT 5
```

**Question:** How many blocks must be allocated before the next request fails? (**Answer:** five blocks can succeed; the sixth fails with `K_NO_WAIT`.)

Note: the test program's loop uses `SLAB_BLOCK_COUNT`, so the demonstration updates automatically. The log heading still says “Three fixed 64-byte blocks”; update that message as well.

### Exercise B — choose a larger heap request

Change the private heap request from `192` to `512` bytes, and adjust the matching `memset()` length and print strings. Check allocation results and remember to free the pointer.

### Exercise C — try a different system heap size

Change:

```ini
CONFIG_HEAP_MEM_POOL_SIZE=2048
```

to a smaller value such as `256`, then perform a pristine rebuild. Investigate whether the 128-byte `k_malloc()` request succeeds. Heap bookkeeping and any subsystem minimum requirements affect how much is actually usable.

**Don't assume the exact failure point merely by comparing request bytes to configured heap bytes.**

### Exercise D — ownership question

```c
void *p = k_heap_alloc(&private_heap, 64, K_NO_WAIT);
```

Which is the correct release operation?

- `k_free(p)` — **incorrect**
- `k_mem_slab_free(&packet_slab, p)` — **incorrect**
- `k_heap_free(&private_heap, p)` — **correct**

### Exercise E — design thinking

You need a queue of 20 sensor descriptors, each exactly 48 bytes. Would you start with a heap or a slab?

**Suggested answer:** A slab is a strong candidate for predictable fixed-size storage. Decide block alignment, size (possibly rounded up), and how many descriptors can be outstanding at once. A fixed array may be even simpler if you never need to borrow/release elements.

---

## 19. Mini quiz (answers below)

1. Does every Zephyr thread have a separate stack?
2. Does `k_malloc()` get memory from your `K_HEAP_DEFINE(my_heap, ...)` object?
3. Can a 64-byte slab return one 200-byte block?
4. What does `K_NO_WAIT` mean when the slab is exhausted?
5. Why is forgetting `k_free()` a problem?
6. Which API releases a `k_heap_alloc()` allocation?
7. Should you copy `K_MEM_POOL_DEFINE()` from an old blog into modern Zephyr?
8. Does `K_THREAD_STACK_DEFINE()` create a general-purpose heap?

<details>
<summary>Answers</summary>

1. Each normal thread has its own stack storage; how stacks are provisioned depends on thread type and configuration.
2. No. It uses the system heap, while `my_heap` is a separate heap object.
3. No. It returns one fixed-size block per allocation.
4. Return immediately with a failure such as `-ENOMEM`.
5. The allocation remains occupied, potentially exhausting memory over time.
6. `k_heap_free(&that_same_heap, ptr)`.
7. No; the historical pool API was replaced by modern heap facilities.
8. No. It defines a properly formed thread stack region.

</details>

---

## 20. One-page revision sheet

```text
MEMORY IN ZEPHYR — REMEMBER THIS

STATIC / GLOBAL
    Always present, simple, fixed-size

STACK
    One working area per thread
    K_THREAD_STACK_DEFINE(...)
    Use K_THREAD_STACK_SIZEOF(...) with k_thread_create(...)

SYSTEM HEAP
    CONFIG_HEAP_MEM_POOL_SIZE=...
    k_malloc(...) -> k_free(...)

PRIVATE HEAP
    K_HEAP_DEFINE(my_heap, bytes)
    k_heap_alloc(&my_heap, size, timeout)
    k_heap_free(&my_heap, ptr)

MEMORY SLAB
    K_MEM_SLAB_DEFINE(slab, block_size, count, align)
    k_mem_slab_alloc(&slab, &ptr, timeout)
    k_mem_slab_free(&slab, ptr)

OLD MEMORY POOL MACROS
    Do not use historical K_MEM_POOL_DEFINE in Zephyr 4.4

RULE #1: Check allocation results.
RULE #2: Free with the matching allocator.
RULE #3: Never use memory after freeing it.
RULE #4: Measure stack usage on your real target.
```

---

## 21. Reference documentation

Official Zephyr documentation:

- [Memory Management overview](https://docs.zephyrproject.org/latest/kernel/memory_management/index.html)
- [Memory Heaps](https://docs.zephyrproject.org/latest/kernel/memory_management/heap.html)
- [Memory Slabs](https://docs.zephyrproject.org/latest/kernel/memory_management/slabs.html)
- [Memory Blocks Allocator](https://docs.zephyrproject.org/latest/kernel/memory_management/sys_mem_blocks.html)
- [Thread stacks and thread creation](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html)
- [Thread Analyzer](https://docs.zephyrproject.org/latest/services/debugging/thread-analyzer.html)
- [Heap API reference](https://docs.zephyrproject.org/latest/doxygen/html/group__heap__apis.html)
- [Memory slab API reference](https://docs.zephyrproject.org/latest/doxygen/html/group__mem__slab__apis.html)

Historical API note:

- [Zephyr 2.4 release notes (deprecation of the older memory-pool APIs)](https://docs.zephyrproject.org/2.4.0/releases/release-notes-2.4.html)

---

## 22. Where we go next

**Lesson 09 — Zephyr Interrupts, ISRs, and Deferred Processing**

We'll learn how an interrupt interrupts normal thread execution; what is safe to do in an ISR; and how to signal a semaphore or submit work to a workqueue instead of doing heavy work inside interrupt context.

**Before moving on:** Build this lesson in QEMU, check that the fourth slab allocation fails as expected, and verify it succeeds after releasing one block. That single experiment demonstrates why allocation, exhaustion, and release matter.
