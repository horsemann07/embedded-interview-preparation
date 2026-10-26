# Lesson 02 — How Zephyr Boots and Reaches `main()`

**Zephyr RTOS Internals • Beginner Level • QEMU x86**

You have already run your first Zephyr application successfully:

```text
SeaBIOS (version rel-1.16.3...)
Booting from ROM..
*** Booting Zephyr OS build v4.4.0-16104-g64437be51c30 ***
Hello World! qemu_x86/atom
```

Now comes the interesting question:

**You only wrote `main()`. Who started the CPU, initialized the kernel, created the main thread, and finally called your function?**

Unlike a typical FreeRTOS application, you didn't call `vTaskStartScheduler()`.

Zephyr handled that automatically.

---

## 1. Understand booting like a 10-year-old

Imagine you're opening a school.

Before students can begin studying:

1. Somebody unlocks the school gate.
2. The lights and electricity are switched on.
3. Classrooms and desks are prepared.
4. Teachers are assigned to their classes.
5. The principal announces that classes can start.
6. Students finally begin their lessons.

Your embedded computer follows a similar sequence.

| School analogy | Zephyr equivalent |
|---|---|
| Unlock school gate | CPU starts executing |
| Switch on electricity | CPU and low-level hardware setup |
| Prepare classrooms | Kernel and device initialization |
| Assign teachers | Create initial threads |
| Start school timetable | Scheduler begins running threads |
| Students begin studying | Your `main()` executes |

The important point is that **`main()` is not the first code that runs on the processor.**

---

## 2. The complete Zephyr boot flow

Here is the simplified flow for your QEMU x86 application.

```text
┌───────────────────────────────────────────┐
│ 1. QEMU starts                            │
│ Creates a virtual x86 computer            │
└───────────────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│ 2. SeaBIOS boots                          │
│ Provides virtual PC firmware              │
└───────────────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│ 3. x86 startup                            │
│ Prepares CPU and initial memory           │
└───────────────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│ 4. z_cstart()                             │
│ Starts common Zephyr kernel initialization│
└───────────────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│ 5. prepare_multithreading()               │
│ Sets up main thread and scheduler data    │
└───────────────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│ 6. bg_thread_main()                       │
│ Finishes kernel and device initialization │
└───────────────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│ 7. main()                                 │
│ Runs your Hello World application         │
└───────────────────────────────────────────┘
```

These are the important stages, not every internal function call. On a normal multithreaded Zephyr build, `z_cstart()` sets up the main thread, and that thread performs the remaining initialization before calling your application's `main()`.

---

## 3. Stage 1 — QEMU and CPU startup

When you execute:

```powershell
west build -d ..\build\hello_world_qemu -t run
```

West invokes the build system, which launches QEMU with your compiled Zephyr ELF image.

Your machine looks like this:

```text
Windows PC
   |
   +-- PowerShell
          |
          +-- West / CMake / Ninja
                  |
                  +-- QEMU
                        |
                        +-- Virtual x86 CPU
                        +-- Virtual RAM
                        +-- Virtual peripherals
                        |
                        +-- Zephyr firmware
```

QEMU emulates a computer. The virtual CPU executes your Zephyr firmware's machine instructions, not Windows application instructions.

For the x86 platform, early architecture code prepares the CPU for C execution. On 32-bit x86, this can include entering protected mode, preparing segment registers, setting up a stack, and preparing memory sections.

### What is a stack?

A stack is a region of RAM used for temporary function data, return addresses, and local variables.

For example:

```c
void hello(void)
{
    int number = 10;
}
```

The CPU needs a working stack before ordinary C functions can execute correctly.

---

## 4. Stage 2 — `z_cstart()` begins Zephyr kernel initialization

The first important common kernel function to study is:

**`z_cstart()`**

Find it in your existing Zephyr source:

```text
zephyr/
└── kernel/
    └── init.c
```

Open it from your current PowerShell terminal:

```powershell
notepad .\kernel\init.c
```

Search for:

```c
void z_cstart(void)
```

This function is one of Zephyr's key boot entry points.

Conceptually, it performs work like:

```c
/* Simplified illustration, not exact source code */

void z_cstart(void)
{
    /* Very early initialization */
    run_early_initialization();

    /* CPU architecture initialization */
    initialize_architecture();

    /* Hardware and early kernel setup */
    initialize_early_devices();

    /* Create and enter the kernel main thread */
    prepare_and_switch_to_main_thread();
}
```

**Note:** The snippet above is only for explanation. Those illustrative helper names should not be copied into your project as real Zephyr APIs.

The real source uses functions such as `z_sys_init_run_level()`, `arch_kernel_init()`, `prepare_multithreading()`, and `switch_to_main_thread()`.

### What does `z_` mean?

You'll notice many Zephyr internal functions begin with `z_`:

```c
z_cstart()
z_sched_init()
z_setup_new_thread()
z_ready_thread()
```

The `z_` prefix is commonly used for internal Zephyr implementation symbols.

These aren't usually functions your application should call directly.

---

## 5. Stage 3 — Zephyr creates its first threads

Inside `kernel/init.c`, search for:

```c
prepare_multithreading()
```

You will find code similar to:

```c
z_sched_init();

stack_ptr = z_setup_new_thread(
    &z_main_thread,
    z_main_stack,
    K_THREAD_STACK_SIZEOF(z_main_stack),
    bg_thread_main,
    NULL, NULL, NULL,
    CONFIG_MAIN_THREAD_PRIORITY,
    K_ESSENTIAL,
    "main"
);

z_mark_thread_as_not_sleeping(&z_main_thread);
z_ready_thread(&z_main_thread);
```

This is a **simplified example of the initialization logic**, not guaranteed to match your exact Zephyr revision character-for-character.

Let's understand it.

| Code | Simple meaning |
|---|---|
| `z_sched_init()` | Prepare the scheduling system. |
| `z_main_thread` | Data structure describing the main thread. |
| `z_main_stack` | Memory for the main thread's stack. |
| `z_setup_new_thread()` | Prepare the thread so the CPU can execute it. |
| `bg_thread_main` | Function that the initial main thread starts executing. |
| `z_ready_thread()` | Mark the thread as ready to run. |

### What is a thread?

Think of a thread as one worker in a factory.

A microcontroller can have several software workers:

- Thread A reads a sensor.
- Thread B processes sensor data.
- Thread C sends data over UART.

On a single CPU, the scheduler decides which ready worker gets to use the CPU.

**Zephyr automatically creates a main thread and an idle thread.** The idle thread executes when no other thread is ready to run.

---

## 6. Stage 4 — The CPU switches into the main thread

Find this function in `kernel/init.c`:

```c
switch_to_main_thread()
```

This is where Zephyr transfers execution from its special startup context into the prepared main thread.

A **context switch** means the CPU stops using one execution context and begins using another. It involves CPU state such as registers and the stack pointer.

The first startup-to-main transition is special because the startup context isn't an ordinary application thread.

Zephyr uses architecture-specific mechanisms for this step.

---

## 7. Stage 5 — `bg_thread_main()` prepares everything else

Now search for:

```c
bg_thread_main()
```

This function continues system initialization within a real thread.

It runs the later initialization stages, including driver and application-level initialization.

Zephyr has several initialization levels:

| Level | Meaning |
|---|---|
| `EARLY` | Very early setup. |
| `PRE_KERNEL_1` | Low-level initialization before normal kernel services are available. |
| `PRE_KERNEL_2` | More initialization before normal kernel services are available. |
| `POST_KERNEL` | Initialization after kernel services become usable. |
| `APPLICATION` | Final initialization before application `main()`. |
| `SMP` | Multiprocessor-specific initialization, if enabled. |

For example, a hardware driver may need clocks or interrupt support initialized before it can operate. These stages help enforce the correct ordering.

### Finally, Zephyr calls your `main()`

Inside `bg_thread_main()`, near the end of the function, you'll find the application entry call.

For a build without `CONFIG_BOOTARGS`, it looks like:

```c
extern int main(void);

(void)main();
```

That's the connection we've been looking for.

**Your application `main()` is called by Zephyr's kernel initialization code!**

---

## 8. Hands-on experiment: Explore the real source

Let's prove the boot path on your installation instead of just reading about it.

You're currently working inside:

```text
C:\Users\ragha\workspace\personal\zephyrproject\zephyr
```

Run this in PowerShell:

```powershell
Select-String -Path .\kernel\init.c -Pattern `
"void z_cstart", `
"prepare_multithreading", `
"switch_to_main_thread", `
"bg_thread_main", `
"\(void\)main"
```

This searches the actual kernel source and shows matching line numbers.

Next, open the file in VS Code if the `code` command is available:

```powershell
code .\kernel\init.c
```

Use `Ctrl+F` to follow these functions in order.

### Boot code reading checklist

- [ ] 1. `z_cstart()`
- [ ] 2. `prepare_multithreading()`
- [ ] 3. `z_setup_new_thread()`
- [ ] 4. `switch_to_main_thread()`
- [ ] 5. `bg_thread_main()`
- [ ] 6. `main()`

---

## 9. Small practical experiment: Print the main thread information

Instead of editing Zephyr's original sample, create our own application in a new folder.

From your current `zephyr/` directory:

```powershell
New-Item -ItemType Directory -Force ..\my_first_app\src
```

Create these three files:

### `my_first_app/src/main.c`

```c
#include <zephyr/kernel.h>

int main(void)
{
    printk("\n--- Zephyr Boot Experiment ---\n");

    k_tid_t current = k_current_get();

    printk("Application main() started!\n");
    printk("Current thread: %p\n", (void *)current);
    printk("System uptime: %lld ms\n",
           (long long)k_uptime_get());

    printk("Hello from Zephyr main thread!\n");

    return 0;
}
```

Notice `k_current_get()` retrieves the thread currently executing. In this case, it is Zephyr's main thread.

### `my_first_app/prj.conf`

```ini
CONFIG_PRINTK=y
CONFIG_MULTITHREADING=y
```

### `my_first_app/CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(my_first_app)

target_sources(app PRIVATE src/main.c)
```

Build it:

```powershell
west build -b qemu_x86 ..\my_first_app -d ..\build\boot_experiment
```

Run it:

```powershell
west build -d ..\build\boot_experiment -t run
```

Expected output will look similar to:

```text
*** Booting Zephyr OS build ... ***

--- Zephyr Boot Experiment ---
Application main() started!
Current thread: 0x...
System uptime: ... ms
Hello from Zephyr main thread!
```

The thread address and uptime values will depend on the actual execution.

This demonstrates that your `main()` runs with a thread context already established.

---

## 10. What happens after `main()` returns?

Your Hello World application prints once and returns.

Does that automatically stop Zephyr?

**No.**

When `main()` returns normally, Zephyr allows the main thread to terminate. Other ready threads can continue running. When nothing else is ready, the idle thread runs.

That's another important difference between a normal desktop program and an RTOS application.

For a continuously running embedded application, you might instead write:

```c
int main(void)
{
    while (1) {
        printk("Main thread running\n");
        k_sleep(K_SECONDS(1));
    }

    return 0;
}
```

Here, the main thread prints, sleeps for a second, and repeats. While it sleeps, the CPU can execute other ready threads.

---

## 11. Zephyr versus FreeRTOS startup

| Stage | Typical FreeRTOS application | Zephyr |
|---|---|---|
| Hardware startup | Startup code and board initialization | Architecture, SoC, and board startup |
| Kernel setup | Application performs required setup | Zephyr initializes its kernel |
| First application task | Created by application | Main thread created automatically |
| Start scheduler | Usually `vTaskStartScheduler()` | Started automatically during kernel boot |
| Application entry | Commonly `main()` creates tasks | Zephyr invokes `main()` inside its main thread |
| Idle task/thread | Created by kernel when scheduler starts | Created by Zephyr during initialization |

One important nuance: Zephyr can also be built without normal multithreading support, but our explanation is for the standard multithreaded build you're learning with.

---

## What you've learned

You now know that the important part of the Zephyr boot process is:

```text
CPU startup
    |
    v
z_cstart()
    |
    v
Prepare scheduler and main thread
    |
    v
Switch to main thread
    |
    v
bg_thread_main()
    |
    v
Your main()
```

**The most important source file for this lesson is `zephyr/kernel/init.c`.**

The next natural lesson is **03 — Zephyr Thread Creation, Scheduling, and Context Switching**. We'll create two threads in QEMU, observe how the scheduler switches between them, compare `k_thread_create()` with FreeRTOS `xTaskCreate()`, and trace the actual scheduler implementation in `kernel/sched.c`.

---

## Official references (for further reading)

- [Zephyr kernel documentation](https://docs.zephyrproject.org/latest/kernel/index.html)
- [Zephyr thread documentation](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html)
- [Zephyr system initialization documentation](https://docs.zephyrproject.org/latest/kernel/drivers/index.html#initialization-levels)
- [Zephyr GitHub source: `kernel/init.c`](https://github.com/zephyrproject-rtos/zephyr/blob/main/kernel/init.c)

> **Note:** Zephyr internals can change across releases. For this lesson, use your local `zephyr/kernel/init.c` as the final reference for the exact source code in your checked-out version.
