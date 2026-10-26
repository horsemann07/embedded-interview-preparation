# Lesson 03 — Zephyr Thread Creation, Scheduling, and Context Switching

> **Beginner-friendly Zephyr RTOS learning series**  
> **Platform:** Windows PowerShell + Zephyr + QEMU (`qemu_x86`)  
> **Your tested setup:** Zephyr `v4.4.0-16104-g64437be51c30`; QEMU target `qemu_x86/atom`  
> **Previous lesson:** 02 — How Zephyr boots and reaches `main()`

---

## What will we learn?

In Lesson 02, we saw how Zephyr starts and eventually calls your `main()` function.

Now let's answer a bigger question:

**How can one tiny processor look after multiple jobs at once?**

By the end of this lesson, you will understand:

1. What a **thread** is and why embedded systems use threads.
2. How to **create two threads** in Zephyr.
3. What the **scheduler** does and how **priority** works.
4. What **running**, **ready**, and **waiting** mean.
5. How **context switching** lets one CPU share its time.
6. How Zephyr's thread APIs compare with **FreeRTOS** and **ThreadX**.
7. Where the relevant implementation lives in **Zephyr's source code**.
8. How to **build and run** a hands-on experiment in QEMU.

Everything is explained in simple words, then linked to the real C code.

---

## 1. A thread is like a worker

Imagine a small robot. It has three jobs:

- **Worker A:** Read the temperature sensor.
- **Worker B:** Blink the LED.
- **Worker C:** Send a message over UART.

Instead of putting all three jobs in one giant `while (1)` loop, an RTOS lets us write them as separate **threads**.

```text
                  TINY ROBOT
                      |
                  One CPU
                      |
                  Scheduler
             +--------+--------+
             |        |        |
         Thread A Thread B Thread C
          Sensor    LED       UART
```

A **thread** is a sequence of instructions that the operating system can schedule independently.

**Important:** With one CPU core, only one ordinary thread executes on that core at a given instant. It just switches between them so quickly that the robot can make progress on multiple jobs. A multicore system can genuinely run threads on different cores at the same time.

### An everyday analogy

Imagine you are doing homework and cooking noodles:

1. Read one page of homework.
2. Put the noodles in boiling water.
3. While the noodles cook, do more homework.
4. Check the noodles when the timer rings.

You didn't grow another pair of hands. You simply changed which job you were doing. The **scheduler** is the part of the RTOS that decides which job gets the CPU next.

---

## 2. Every Zephyr thread needs a few things

Think of a thread as a worker who needs:

| Thread part                | Child-friendly meaning                 | Real Zephyr name                        |
| -------------------------- | -------------------------------------- | --------------------------------------- |
| Entry function             | The worker's instruction sheet         | `void worker(void *, void *, void *)`   |
| Stack                      | The worker's personal scratch notebook | `K_THREAD_STACK_DEFINE(...)`            |
| Thread control block (TCB) | The worker's ID card / status record   | `struct k_thread`                       |
| Priority                   | How urgently the worker gets the CPU   | `K_PRIO_PREEMPT(n)` or `K_PRIO_COOP(n)` |
| Start setting              | Start now or later?                    | `K_NO_WAIT`, `K_FOREVER`, etc.          |

### What is a stack?

A **stack** is a memory area used by a thread while calling functions. It holds information such as return addresses and some temporary values.

```text
Thread A stack          Thread B stack
+---------------+       +---------------+
| return info   |       | return info   |
| local data    |       | local data    |
| saved context |       | saved context |
+---------------+       +---------------+
```

Each thread has its **own stack**. But threads normally share the application's global variables and memory address space. That is why shared data may need protection with mutexes or other synchronization tools.

### What is a TCB?

`struct k_thread` stores bookkeeping information the kernel uses about a thread: scheduling state, priority, stack-related details, and more.

**Simple memory rule:**

- Stack = **where the worker keeps temporary notes**.
- TCB = **where the principal keeps the worker's record**.
- Thread function = **the job the worker performs**.

---

## 3. Creating a thread: understand each line

Here is a small example of how one thread is created:

```c
#include <zephyr/kernel.h>

#define STACK_SIZE 1024

K_THREAD_STACK_DEFINE(worker_stack, STACK_SIZE);
static struct k_thread worker_data;

static void worker(void *p1, void *p2, void *p3)
{
    ARG_UNUSED(p1);
    ARG_UNUSED(p2);
    ARG_UNUSED(p3);

    while (1) {
        printk("Worker is running!\n");
        k_sleep(K_MSEC(1000));
    }
}

int main(void)
{
    k_thread_create(&worker_data,
                    worker_stack,
                    K_THREAD_STACK_SIZEOF(worker_stack),
                    worker,
                    NULL, NULL, NULL,
                    K_PRIO_PREEMPT(5),
                    0,
                    K_NO_WAIT);

    return 0;
}
```

### Decode `k_thread_create()` like LEGO blocks

```c
k_thread_create(
    &worker_data,                         // Thread's record (TCB)
    worker_stack,                         // Thread's own stack
    K_THREAD_STACK_SIZEOF(worker_stack),  // Usable stack size
    worker,                               // Which function to execute
    NULL, NULL, NULL,                     // Up to 3 optional arguments
    K_PRIO_PREEMPT(5),                    // Scheduling priority
    0,                                    // Thread options
    K_NO_WAIT                             // Make ready immediately
);
```

**Note:** `K_NO_WAIT` means the new thread becomes eligible to run immediately. It does **not** guarantee that it will run before the current thread. The scheduler still chooses according to priority.

`k_thread_create()` returns a thread ID of type `k_tid_t` (a handle you can later use for joining or managing the thread).

Zephyr also has **`K_THREAD_DEFINE()`**, which creates a thread, stack, and control block using a convenient static definition. We'll focus on `k_thread_create()` here because it helps you see the individual pieces.

---

## 4. What exactly does a scheduler do?

The **scheduler** chooses which ready thread can use the CPU.

Think of it as a school principal calling the next student into the classroom.

```text
           READY THREADS
        +-------------------+
        | Thread A: prio 5  |
        | Thread B: prio 7  |
        | Thread C: prio 9  |
        +---------+---------+
                  |
                  v
         ZEPHYR SCHEDULER
                  |
                  v
            CPU runs A
```

In this example, priority **5** is more urgent than 7 or 9.

### A rule that is easy to forget

**In Zephyr, a smaller priority number means a higher scheduling priority.**

| Priority number | Relative urgency                                              |
| --------------- | ------------------------------------------------------------- |
| `-2`            | Cooperative priority; very high in ordinary priority ordering |
| `0`             | Higher than preemptible priority 5                            |
| `5`             | Higher than 7                                                 |
| `7`             | Lower than 5                                                  |
| `9`             | Lower than 7                                                  |

The available priority range depends on Kconfig settings. You cannot assume every priority number is enabled on every build.

### Two kinds of priorities

**Preemptible threads:**

```c
K_PRIO_PREEMPT(5)
```

- Have **non-negative** priority values.
- Can be interrupted by a higher-priority ready thread at scheduling points.
- Are what we'll use in the QEMU experiment.

**Cooperative threads:**

```c
K_PRIO_COOP(1)
```

- Have **negative** priority values.
- Normally keep the CPU until they **yield**, **sleep**, or otherwise **block**.
- Can still be interrupted by hardware interrupts. Special kernel features also have additional rules, so don't think of cooperative mode as disabling interrupts.

For now, start with **preemptible threads**. They're easier to learn with.

---

## 5. Ready, running, and waiting: the thread lifecycle

Threads do not all run constantly.

```text
            Create / Start
                   |
                   v
                [READY] <---------------------+
                   |                          |
           Scheduler selects                 |
                   |                          |
                   v                          |
               [RUNNING]                      |
                 /   \                        |
                /     \                       |
       sleep / wait   preempt / yield --------+
              |
              v
           [WAITING]
              |
     Time expires / event arrives
              |
              +------------------------------> READY

RUNNING -> [TERMINATED]  (thread function returns)
```

This diagram is a **learning model**, not a literal dump of Zephyr's internal state flags. In Zephyr's documentation, **ready** is a thread state, while **running** means a ready thread is currently selected on the CPU.

| Concept           | Easy explanation                             |
| ----------------- | -------------------------------------------- |
| Ready             | I have work and can run if selected.         |
| Running           | I am currently using the CPU.                |
| Waiting / blocked | I can't run until something happens.         |
| Sleeping          | I am waiting for a timer.                    |
| Suspended         | I have been explicitly paused until resumed. |
| Terminated        | My thread function finished or was ended.    |

### What does `k_sleep()` really do?

```c
k_sleep(K_MSEC(500));
```

It means **"I don't need the CPU for approximately 500 milliseconds."**

The thread becomes unready for that sleep interval. The scheduler is then free to run another ready thread or the idle thread. When the timeout expires, the sleeping thread becomes ready again.

**Very important:** `k_sleep()` does **not** mean the CPU is busy waiting for 500 ms. The CPU is available for other work.

---

## 6. Context switching: the CPU changes workers

Let's say Thread A is running, but it calls `k_sleep()`.

The CPU must stop A and start another ready thread, B.

How can A later resume **exactly where it stopped**?

Zephyr performs a **context switch**.

Imagine putting a bookmark in one book before opening another book.

```text
Before:
  CPU ---> Thread A (running)
           Thread B (ready)

Thread A calls k_sleep():
  1. Save needed CPU state for A
  2. Mark A unready (sleeping)
  3. Scheduler selects B
  4. Restore B's saved CPU state

After:
  CPU ---> Thread B (running)
           Thread A (sleeping)

Later, timer expires for A:
  A becomes ready again
  If scheduling rules allow, CPU switches back to A
```

### What gets saved?

The exact details depend on the CPU architecture, but context switching generally involves information such as:

- The **stack pointer** (where the thread's stack is currently being used).
- CPU **registers** that must be preserved.
- The **instruction/execution position** needed to resume correctly.
- Architecture-specific status or extended context when required.

The CPU does **not** normally copy a thread's entire stack into another stack every time. Each thread already has its own stack. The kernel/CPU arrangement preserves enough state to resume the thread using that stack.

### Is an interrupt the same thing as a context switch?

**No.** An interrupt runs an interrupt service routine (ISR). After the ISR, Zephyr may decide that a different thread should run. An interrupt can create a **reschedule point**, but an ISR itself is not an ordinary application thread.

### Who does which part?

```text
Your application             kernel/thread.c
    |                              |
    +-- k_thread_create() -------->+  prepare thread
    |
    +-- k_sleep() --------------------+
                                      v
                                kernel/sched.c
                                  choose next
                                      |
                                      v
                                arch/x86/... 
                                save / restore
                                processor context
```

The **scheduler policy** is largely common kernel code. The low-level **CPU register switch** is architecture-specific.

---

## 7. Our real QEMU experiment: two workers

We will create a little "factory" with two workers:

- **HIGH worker**: priority **5**, prints once every **400 ms**.
- **LOW worker**: priority **7**, prints once every **650 ms**.

Both will print **six times** and finish. The main thread will wait until both are done.

The goal is to watch how both jobs make progress on one emulated CPU.

## Project Directory

All our Zephyr learning applications will be created inside `zephyr/myapp/`.

We will **not modify Zephyr's original kernel source code** or official samples.

```text
zephyrproject/
├── .venv/
├── zephyr/                         <-- Zephyr RTOS source
│   ├── arch/
│   ├── boards/
│   ├── drivers/
│   ├── include/
│   ├── kernel/
│   ├── samples/
│   └── myapp/                      <-- Our learning applications
│       └── lesson03_threads/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson03_threads/           <-- Generated build files
```

> **Note:** `myapp/` is inside the Zephyr Git repository. Keep your learning code separate from official Zephyr changes and avoid accidentally committing it upstream.

---

### Step 1 — Create the Folder

Open **PowerShell**, activate your Python virtual environment, and navigate to the Zephyr directory.

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject

.\.venv\Scripts\Activate.ps1

cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson03_threads\src | Out-Null
```

Your application directory is now:

```text
zephyrproject/zephyr/myapp/lesson03_threads/
```

> **PowerShell reminder:** Use `Activate.ps1`, not `activate.bat`. Your `.venv` folder is located in the workspace root.

---

### Step 2 — Create `main.c`

Save the Lesson 03 two-thread application code in:

```text
C:\Users\ragha\workspace\personal\zephyrproject\zephyr\myapp\lesson03_threads\src\main.c
```
```c
/**
 * @file main.c
 * @brief Lesson 03: Thread creation and priority-based scheduling in Zephyr RTOS.
 *
 * @details
 * This example demonstrates the following Zephyr RTOS concepts:
 *
 * 1. Creating threads dynamically using k_thread_create().
 * 2. Allocating dedicated stacks for individual threads.
 * 3. Assigning different preemptive priorities to threads.
 * 4. Understanding priority-based preemptive scheduling.
 * 5. Suspending thread execution temporarily using k_sleep().
 * 6. Waiting for thread completion using k_thread_join().
 *
 * Thread configuration:
 *
 *   High-priority thread:
 *     - Priority       : 5
 *     - Execution count: 6
 *     - Sleep interval : 400 ms
 *
 *   Low-priority thread:
 *     - Priority       : 7
 *     - Execution count: 6
 *     - Sleep interval : 650 ms
 *
 * Important:
 *   In Zephyr, a lower numerical priority value indicates
 *   a higher scheduling priority.
 *
 *   Therefore:
 *
 *       Priority 5 > Priority 7
 *
 *   Both threads execute concurrently from the application's
 *   perspective, with the Zephyr scheduler deciding which
 *   ready thread receives CPU time.
 *
 *   When a thread calls k_sleep(), it becomes unready,
 *   allowing other ready threads to execute.
 */

#include <zephyr/kernel.h>
#include <zephyr/sys/printk.h>

/* --------------------------------------------------------------------------
 * Thread Configuration
 * -------------------------------------------------------------------------- */

/**
 * @brief Stack size allocated to each worker thread.
 *
 * Each thread receives a dedicated stack of 1024 bytes.
 *
 * The stack stores:
 *   - Local variables
 *   - Function call information
 *   - Saved execution context
 *   - Other stack-based data
 *
 * Note:
 *   Stack requirements depend on the application and architecture.
 *   For production applications, verify actual stack usage.
 */
#define STACK_SIZE       1024

/**
 * @brief Thread priorities.
 *
 * Zephyr preemptive priorities use non-negative values.
 * A smaller numerical value represents a higher priority.
 *
 * HIGH_PRIORITY = 5
 * LOW_PRIORITY  = 7
 *
 * Consequently, the high-priority thread executes before
 * the low-priority thread when both are ready to run,
 * assuming no higher-priority thread is executing.
 */
#define HIGH_PRIORITY    5
#define LOW_PRIORITY     7

/* --------------------------------------------------------------------------
 * Thread Stack Allocation
 * -------------------------------------------------------------------------- */

/**
 * @brief Allocate a dedicated stack for the high-priority thread.
 *
 * K_THREAD_STACK_DEFINE() allocates stack memory with the
 * alignment and architecture-specific requirements expected
 * by the Zephyr kernel.
 */
K_THREAD_STACK_DEFINE(high_stack, STACK_SIZE);

/**
 * @brief Allocate a dedicated stack for the low-priority thread.
 *
 * Every thread needs its own stack so that its execution
 * context remains independent of other threads.
 */
K_THREAD_STACK_DEFINE(low_stack, STACK_SIZE);

/* --------------------------------------------------------------------------
 * Thread Control Blocks
 * -------------------------------------------------------------------------- */

/**
 * @brief Thread control blocks used by the Zephyr kernel.
 *
 * struct k_thread contains the internal information required
 * by the scheduler to manage a thread, including its state,
 * scheduling information, and stack context.
 *
 * One control block is required for each thread.
 */
static struct k_thread high_thread_data;
static struct k_thread low_thread_data;

/* --------------------------------------------------------------------------
 * Static Function Declarations
 * -------------------------------------------------------------------------- */

static void high_worker(void *p1, void *p2, void *p3);
static void low_worker(void *p1, void *p2, void *p3);

/* --------------------------------------------------------------------------
 * Main Application
 * -------------------------------------------------------------------------- */

/**
 * @brief Main application entry point.
 *
 * @details
 * In Zephyr RTOS, main() executes within a kernel-managed thread.
 *
 * This function:
 *   1. Creates the high-priority worker thread.
 *   2. Creates the low-priority worker thread.
 *   3. Waits for both worker threads to finish.
 *   4. Prints a completion message.
 *
 * @return 0 when the example completes.
 */
int main(void)
{
    printk("\n=== Lesson 03: Zephyr Threads ===\n");
    printk("main() is already running inside a Zephyr thread.\n");

    /*
     * Create the high-priority worker thread.
     *
     * Parameters:
     *
     * 1. &high_thread_data
     *      Pointer to the thread control block.
     *
     * 2. high_stack
     *      Memory allocated for the thread stack.
     *
     * 3. K_THREAD_STACK_SIZEOF(high_stack)
     *      Usable stack size, accounting for Zephyr's
     *      architecture-specific stack requirements.
     *
     * 4. high_worker
     *      Function executed by the new thread.
     *
     * 5-7. NULL
     *      Three optional arguments passed to the thread.
     *
     * 8. K_PRIO_PREEMPT(HIGH_PRIORITY)
     *      Assigns preemptive priority 5.
     *
     * 9. 0
     *      No special thread options.
     *
     * 10. K_NO_WAIT
     *      Makes the thread ready immediately.
     *
     * Note:
     *   The newly created thread may begin executing
     *   before k_thread_create() returns, depending
     *   on scheduling priorities.
     */
    k_tid_t high_id = k_thread_create(
        &high_thread_data,
        high_stack,
        K_THREAD_STACK_SIZEOF(high_stack),
        high_worker,
        NULL, NULL, NULL,
        K_PRIO_PREEMPT(HIGH_PRIORITY),
        0,
        K_NO_WAIT
    );

    /*
     * Create the low-priority worker thread.
     *
     * This thread has priority 7, which is lower
     * than the high-priority worker's priority 5.
     *
     * When both worker threads are ready,
     * the scheduler selects the high-priority worker.
     *
     * However, when the high-priority worker sleeps,
     * the low-priority worker may execute.
     */
    k_tid_t low_id = k_thread_create(
        &low_thread_data,
        low_stack,
        K_THREAD_STACK_SIZEOF(low_stack),
        low_worker,
        NULL, NULL, NULL,
        K_PRIO_PREEMPT(LOW_PRIORITY),
        0,
        K_NO_WAIT
    );

    printk("Two worker threads created. Main will wait for them.\n");

    /*
     * Wait until the high-priority worker terminates.
     *
     * k_thread_join() blocks the calling thread until
     * the specified thread completes or the timeout expires.
     *
     * K_FOREVER means there is no timeout.
     *
     * While main is blocked, the scheduler continues
     * executing other ready threads.
     *
     * Important:
     *   Waiting for the high-priority thread does not
     *   prevent the low-priority thread from executing.
     */
    (void)k_thread_join(high_id, K_FOREVER);

    /*
     * Wait until the low-priority worker terminates.
     *
     * If this thread has already completed,
     * k_thread_join() returns immediately.
     */
    (void)k_thread_join(low_id, K_FOREVER);

    /*
     * Both worker threads have terminated.
     * Main can now safely print the completion message.
     */
    printk("Both workers finished. Lesson 03 complete!\n");

    return 0;
}

/* --------------------------------------------------------------------------
 * Static Helper Functions
 * -------------------------------------------------------------------------- */

/**
 * @brief High-priority worker thread entry function.
 *
 * @param p1 First optional thread argument (unused).
 * @param p2 Second optional thread argument (unused).
 * @param p3 Third optional thread argument (unused).
 *
 * @details
 * This thread:
 *
 *   1. Prints its current execution count.
 *   2. Sleeps for 400 milliseconds.
 *   3. Repeats the operation six times.
 *   4. Terminates after completing its work.
 *
 * During k_sleep(), the thread transitions into a
 * sleeping state and is not eligible for scheduling.
 *
 * Once the sleep duration expires, it becomes ready.
 *
 * If this thread has the highest priority among
 * ready threads, the scheduler selects it for execution.
 *
 * Note:
 *   k_sleep() specifies a minimum suspension interval;
 *   the actual time before the thread executes again
 *   can be longer because of scheduling and tick timing.
 */
static void high_worker(void *p1, void *p2, void *p3)
{
    /* Explicitly mark unused arguments. */
    ARG_UNUSED(p1);
    ARG_UNUSED(p2);
    ARG_UNUSED(p3);

    for (int i = 1; i <= 6; i++)
    {
        /*
         * k_uptime_get_32() returns the elapsed time
         * since system boot, in milliseconds.
         *
         * This helps observe thread execution timing.
         */
        printk("[%u ms] HIGH priority worker: job %d/6\n",
               k_uptime_get_32(), i);

        /*
         * Suspend this thread for at least 400 ms.
         *
         * The CPU can execute other ready threads
         * while this thread is sleeping.
         */
        k_sleep(K_MSEC(400));
    }

    /*
     * Returning from a Zephyr thread entry function
     * terminates the thread.
     */
    printk("HIGH priority worker finished.\n");
}

/**
 * @brief Low-priority worker thread entry function.
 *
 * @param p1 First optional thread argument (unused).
 * @param p2 Second optional thread argument (unused).
 * @param p3 Third optional thread argument (unused).
 *
 * @details
 * This thread:
 *
 *   1. Prints its current execution count.
 *   2. Sleeps for 650 milliseconds.
 *   3. Repeats the operation six times.
 *   4. Terminates after completing its work.
 *
 * Because this thread has a lower priority,
 * it executes when no higher-priority ready thread
 * prevents it from running.
 *
 * If the high-priority worker becomes ready while
 * this thread is executing, the high-priority worker
 * can preempt it.
 */
static void low_worker(void *p1, void *p2, void *p3)
{
    /* Explicitly mark unused arguments. */
    ARG_UNUSED(p1);
    ARG_UNUSED(p2);
    ARG_UNUSED(p3);

    for (int i = 1; i <= 6; i++)
    {
        /*
         * Print the current system uptime
         * and worker iteration number.
         */
        printk("[%u ms] LOW  priority worker: job %d/6\n",
               k_uptime_get_32(), i);

        /*
         * Suspend this thread for at least 650 ms.
         *
         * Other ready threads can execute during
         * this interval.
         */
        k_sleep(K_MSEC(650));
    }

    /*
     * The thread terminates automatically when
     * its entry function returns.
     */
    printk("LOW priority worker finished.\n");
}
```

### Step 3 — Create `prj.conf`

Save this file as:

`zephyr/myapp/lesson03_threads/prj.conf`

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
```

**Explanation:**

- `CONFIG_MULTITHREADING=y`: Enable normal thread scheduling.
- `CONFIG_PRINTK=y`: Enable simple terminal messages using `printk()`.

**We don't need to enable time slicing for this experiment.**

Our threads explicitly sleep using `k_sleep()`. While one thread sleeps, the scheduler can select another ready thread.

---

### Step 4 — Create `CMakeLists.txt`

Save this file as:

`zephyr/myapp/lesson03_threads/CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson03_threads)

target_sources(app PRIVATE src/main.c)
```

**Explanation:**

| Command                     | Purpose                                  |
| --------------------------- | ---------------------------------------- |
| `cmake_minimum_required()`  | Specifies the minimum CMake version.     |
| `find_package(Zephyr ...)`  | Finds the Zephyr build system.           |
| `project(lesson03_threads)` | Defines our application project.         |
| `target_sources()`          | Adds `src/main.c` to the firmware build. |

---

### Step 5 — Build the Firmware

From the `zephyrproject/zephyr` directory, execute:

```powershell
west build -b qemu_x86 .\myapp\lesson03_threads -d ..\build\lesson03_threads
```

**Command explanation:**

| Parameter                      | Meaning                                  |
| ------------------------------ | ---------------------------------------- |
| `west build`                   | Compile the Zephyr application.          |
| `-b qemu_x86`                  | Use the QEMU x86 virtual board.          |
| `.\myapp\lesson03_threads`     | Location of our application source code. |
| `-d ..\build\lesson03_threads` | Location for generated build files.      |

After a successful build, the firmware will be generated under:

```text
zephyrproject/
└── build/
    └── lesson03_threads/
        └── zephyr/
            └── zephyr.elf
```

#### Perform a Pristine Rebuild

If you change the board, configuration, or need to reset the previous build, run:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson03_threads -d ..\build\lesson03_threads
```

This rebuilds the application with a fresh build configuration.

---

### Step 6 — Run on QEMU

Once the build succeeds, run:

```powershell
west build -d ..\build\lesson03_threads -t run
```

QEMU will start the Zephyr firmware and display output from your threads.

To exit QEMU:

1. Press `Ctrl+A`.
2. Release the keys.
3. Press `X`.

---

### Future Learning Applications

All future lessons will be organized inside `myapp/`:

```text
zephyrproject/
└── zephyr/
    └── myapp/
        ├── lesson03_threads/
        │   ├── CMakeLists.txt
        │   ├── prj.conf
        │   └── src/
        │       └── main.c
        │
        ├── lesson04_semaphores/
        ├── lesson05_mutex/
        ├── lesson06_queues/
        └── ...
```

Each application will contain its own source code, configuration, and CMake files.

**Important:** The Zephyr RTOS source remains shared across all our applications. We do not need to copy the kernel, drivers, or architecture code into every lesson.

### Expected output (illustrative)

```text
*** Booting Zephyr OS build ... ***

=== Lesson 03: Zephyr Threads ===
main() is already running inside a Zephyr thread.
Two worker threads created. Main will wait for them.
[0 ms] HIGH priority worker: job 1/6
[0 ms] LOW  priority worker: job 1/6
[400 ms] HIGH priority worker: job 2/6
[650 ms] LOW  priority worker: job 2/6
[800 ms] HIGH priority worker: job 3/6
[1200 ms] HIGH priority worker: job 4/6
[1300 ms] LOW  priority worker: job 3/6
...
HIGH priority worker finished.
...
LOW priority worker finished.
Both workers finished. Lesson 03 complete!
```

**Actual timestamps, first-run timing, and print order may differ** because they depend on QEMU timing and scheduling. Don't expect the exact numbers above.

If QEMU stays running after the final message, use **Ctrl+A**, release the keys, then press **X** to exit (when using QEMU's `-nographic` console).

**Windows note:** If you open a fresh PowerShell window and encounter the QEMU DLL problem again, remember that temporary `PATH` changes made in an earlier terminal do not survive into a new terminal.

---

## 8. Let's trace what our experiment actually did

### A — Zephyr already created the main thread

From Lesson 02:

```text
CPU startup -> z_cstart() -> main thread -> bg_thread_main() -> main()
```

So `main()` is **not** running outside the RTOS. It is already part of a Zephyr thread.

### B — `main()` created two more threads

```text
main thread
   |
   +-- k_thread_create(HIGH, priority 5)
   |
   +-- k_thread_create(LOW,  priority 7)
   |
   +-- k_thread_join(HIGH, K_FOREVER)
           |
           +-- main waits
                   |
                   +-- scheduler runs ready workers
```

The main thread normally has a higher scheduling priority than our workers on this standard QEMU build, so it can create both workers before blocking in the join. You can inspect the actual main-thread priority in your resolved build configuration rather than assuming that every board/configuration uses the same setting.

### C — The scheduler starts HIGH first

When both workers are ready, **priority 5 wins over priority 7**.

```text
Ready:  HIGH (5), LOW (7)
Choose: HIGH (5)
```

### D — HIGH sleeps

```text
HIGH executes k_sleep(K_MSEC(400))

HIGH: waiting
LOW:  ready

CPU can now run LOW.
```

### E — HIGH wakes up

After the timeout, HIGH becomes ready. If LOW is executing and HIGH outranks it, the scheduler can preempt LOW at the appropriate scheduling point.

### F — Main waits for completion

```c
k_thread_join(high_id, K_FOREVER);
k_thread_join(low_id, K_FOREVER);
```

This means: **"Don't continue until this worker has finished."** If the target thread hasn't finished, main waits and other ready threads get CPU time.

Our worker entry functions return after six print iterations. That terminates those worker threads, so the joins can finish.

---

## 9. The most important timing picture

The following is a **conceptual** timeline, not an exact oscilloscope capture.

```text
Time --------->

HIGH (prio 5):  RUN | sleep | RUN | sleep | RUN | ...
LOW  (prio 7):      RUN | sleep | RUN | sleep | ...
MAIN:          create threads | wait ........... | done

CPU:           MAIN -> HIGH -> LOW -> ... -> MAIN
```

Three very useful lessons are hiding in this diagram:

1. **Sleeping is not wasting CPU time.** When a thread sleeps, it is not occupying the CPU.
2. **Higher priority is not the same as running all the time.** A high-priority thread that is blocked cannot run.
3. **Print order alone is not a perfect proof of every context switch.** Many scheduling decisions happen without printing anything.

---

## 10. Try three small experiments

Change only one thing at a time, then rebuild and run using the commands from Step 5 and Step 6.

### Experiment A — Reverse priorities

Change:

```c
#define HIGH_PRIORITY 7
#define LOW_PRIORITY  5
```

**Predict:** When both threads are ready, the thread whose priority is now **5** gets chosen first.

**Learn:** Names like `high_worker` are just labels. The scheduler uses the **priority number**, not the function name.

### Experiment B — Give both workers equal priority

Change:

```c
#define HIGH_PRIORITY 5
#define LOW_PRIORITY  5
```

**Predict:** Both get chances to run because they call `k_sleep()`. Don't assume they will print in a perfect A-B-A-B pattern; wake-up times differ.

**Learn:** Equal-priority threads don't necessarily alternate automatically after every line of C code. Optional time slicing and explicit yielding affect how equal-priority ready threads share the CPU.

### Experiment C — Change the sleep times

Try:

```c
/* HIGH worker */
k_sleep(K_MSEC(1000));

/* LOW worker */
k_sleep(K_MSEC(200));
```

**Predict:** You will usually see several LOW messages between HIGH messages, even though HIGH has the higher priority.

**Learn:** The scheduler only compares threads that are **ready**. A high-priority sleeping thread isn't ready.

---

## 11. `k_sleep()` versus `k_yield()` versus time slicing

These three ideas sound similar, but they do different jobs.

| Mechanism              | Meaning                                                                  | Can lower-priority threads run?                                                                                  |
| ---------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `k_sleep(K_MSEC(500))` | "I am unavailable for about 500 ms."                                     | Yes, while the caller is sleeping.                                                                               |
| `k_yield()`            | "Let the scheduler consider another ready thread."                       | Not merely because of the yield: the yielding thread stays ready, so lower-priority threads generally don't win. |
| Time slicing           | "Share CPU intervals among eligible equal-priority preemptible threads." | It is primarily for sharing among equal-priority ready threads, not granting priority to lower-priority threads. |

### Mini example: `k_yield()`

```c
while (1) {
    do_a_small_piece_of_work();
    k_yield();
}
```

This is **not the same as sleeping**. If there is no ready thread of equal or higher priority to take the CPU, the same thread can continue.

### What about time slicing?

In some Zephyr configurations, the scheduler can automatically share time between eligible threads with the same priority. This is controlled by settings such as `CONFIG_TIMESLICING`.

We intentionally did not enable it in our first experiment because **sleeping and waking already demonstrate scheduling clearly**. We'll study time slicing separately later.

---

## 12. Cooperative versus preemptive scheduling

Imagine one child is holding the TV remote.

**Preemptive:** If a higher-priority job becomes ready, the scheduler can take the CPU away from the current preemptible thread.

**Cooperative:** The currently running cooperative thread normally keeps going until it decides to yield, sleep, or block.

```text
Preemptive example:
LOW runs -> HIGH wakes -> scheduler can switch to HIGH

Cooperative example:
COOP runs -> another ordinary thread becomes ready
           -> COOP generally continues until it yields/blocks
```

Cooperative threads are useful for certain predictable workloads but can also cause long delays if they never yield or block. Hardware interrupts still work unless interrupts have been explicitly masked.

**Beginner rule:** For your first QEMU programs, use `K_PRIO_PREEMPT(...)` and include `k_sleep()` where a thread has nothing useful to do.


### 12.1 — How FreeRTOS Uses Preemptive and Cooperative Scheduling

FreeRTOS supports **both scheduling methods**, just like Zephyr.

But there is one important difference in how they are configured.

#### 1. FreeRTOS Preemptive Scheduling

This is the **default and most commonly used scheduling mode** in FreeRTOS.

Inside `FreeRTOSConfig.h`:

```c
#define configUSE_PREEMPTION 1
#define configUSE_TIME_SLICING 1
```

**How it works:**

Imagine two tasks:

- Task A: Priority 3 (HIGH)
- Task B: Priority 1 (LOW)

```text
Time ---------------------------------------->

LOW TASK:   [RUNNING] ----STOP---- [RUNNING]
                         |
HIGH TASK:             [RUNNING]
                         |
                    HIGH wakes up
```

1. LOW starts executing because HIGH is sleeping.
2. HIGH wakes up.
3. FreeRTOS checks which task has higher priority.
4. HIGH has greater priority, so the scheduler switches to HIGH.
5. LOW waits until it can run again.

**Important:** In FreeRTOS, a larger priority number means a higher priority.

#### 2. FreeRTOS Cooperative Scheduling

Change the configuration:

```c
#define configUSE_PREEMPTION 0
```

Now automatic task preemption is disabled.

Example:

```text
LOW TASK:  [RUNNING-------------][STOP]
                                |
HIGH TASK:                      [RUNNING]
                                |
                         LOW yields/blocks
```

Even if HIGH becomes ready while LOW is running, LOW normally continues until it voluntarily gives up the CPU.

LOW can do this using:

```c
taskYIELD();
```

Or by blocking:

```c
vTaskDelay(pdMS_TO_TICKS(100));
```

**Simple meaning:**

- `taskYIELD()` — Ask the scheduler to select another ready task.
- `vTaskDelay()` — Sleep for some time, allowing another ready task to execute.

An interrupt can also explicitly request a context switch in supported FreeRTOS ports.

---

### 12.2 — How Zephyr Handles Both Scheduling Methods

Zephyr uses **thread priorities** to select the scheduling class.

```c
K_PRIO_PREEMPT(5)  // Preemptive thread

K_PRIO_COOP(1)     // Cooperative thread
```

For example:

```c
K_THREAD_DEFINE(thread_a, 1024,
                thread_a_fn,
                NULL, NULL, NULL,
                K_PRIO_PREEMPT(5), 0, 0);

K_THREAD_DEFINE(thread_b, 1024,
                thread_b_fn,
                NULL, NULL, NULL,
                K_PRIO_COOP(1), 0, 0);
```

Here:

- Thread A uses preemptive scheduling.
- Thread B uses cooperative scheduling.
- Both exist inside the same Zephyr application.

A running cooperative thread normally keeps the CPU until it yields, sleeps, or blocks.

**Zephyr priority rule:** Smaller numerical values mean higher priority. Cooperative priority values are negative, so cooperative threads outrank preemptive threads.

---

### 12.3 — How ThreadX Handles Preemptive and Cooperative Scheduling

ThreadX (now Eclipse ThreadX) uses **priority-based preemptive scheduling** by default.

Unlike FreeRTOS, ThreadX doesn't normally switch the entire RTOS between preemptive and cooperative modes.

Instead, ThreadX provides:

1. Preemptive scheduling
2. Cooperative yielding between equal-priority threads
3. Preemption-Threshold
4. Optional time slicing for each thread

#### 1. ThreadX Preemptive Scheduling

Imagine two threads:

- Thread A: Priority 5 (HIGH)
- Thread B: Priority 20 (LOW)

**Remember:** In ThreadX, a smaller priority number means higher priority.

```text
Time ---------------------------------->

LOW:  [RUNNING]          [RUNNING]
               \        /
HIGH:           [RUNNING]
                  |
               HIGH wakes
```

When HIGH becomes ready, ThreadX can immediately preempt LOW.

This is similar to FreeRTOS and Zephyr preemptive scheduling.

#### 2. ThreadX Cooperative Scheduling

ThreadX supports cooperative sharing between threads of the same priority.

Example:

```c
tx_thread_relinquish();
```

This tells ThreadX:

"I am ready to give another thread of my priority a chance to run."

You can also block the current thread:

```c
tx_thread_sleep(100);
```

This suspends the thread for 100 timer ticks.

**Important:** Cooperative yielding does not disable normal higher-priority preemption.

#### 3. ThreadX Preemption-Threshold

This is a special ThreadX feature.

Imagine three threads:

| Thread | Priority |
|---|---|
| A | 5 (HIGH) |
| B | 15 (MEDIUM) |
| C | 20 (LOW) |

Normally, B and A can both preempt C.

But suppose Thread C is performing an important operation and does not want Thread B to interrupt it.

ThreadX allows C to set a **preemption-threshold of 15**.

```text
Thread C:
    Priority  = 20
    Threshold = 15

Thread A (Priority 5)
         |
         +-- CAN preempt C

Thread B (Priority 15)
         |
         +-- CANNOT preempt C
```

Only threads with a priority number **smaller than 15** can preempt C.

Example API:

```c
UINT old_threshold;

tx_thread_preemption_change(
    &thread_c,
    15,
    &old_threshold
);
```

Later, restore the previous threshold:

```c
tx_thread_preemption_change(
    &thread_c,
    old_threshold,
    &old_threshold
);
```

This protects the running thread from a selected range of higher-priority threads.

It is not exactly the same as Zephyr's cooperative-thread scheduling class.

**Note:** Preemption-Threshold affects thread scheduling, not hardware interrupt delivery. It also disables time slicing for the protected thread.

#### 4. ThreadX Time Slicing

ThreadX allows you to configure a time slice for each thread.

For example, two threads of priority 10 can share CPU time.

```text
Time ---------------------------------->

Thread A: [RUN]       [RUN]
Thread B:      [RUN]       [RUN]
```

A thread's time slice is configured during thread creation and can be changed later.

```c
tx_thread_time_slice_change(
    &thread_a,
    10,
    &old_time_slice
);
```

Here, `10` means 10 ThreadX timer ticks.

Set `TX_NO_TIME_SLICE` to disable time slicing for that thread.

---

### 12.4 — FreeRTOS vs ThreadX vs Zephyr

| Feature | FreeRTOS | ThreadX | Zephyr |
|---|---|---|---|
| Preemptive scheduling | Yes | Yes | Yes |
| Cooperative behavior | Global scheduling mode | Yielding and Preemption-Threshold | Cooperative thread priorities |
| Scheduling configuration | `FreeRTOSConfig.h` | Thread creation parameters and APIs | Kconfig and thread priorities |
| Priority numbering | Larger = higher | Smaller = higher | Smaller = higher |
| Yield API | `taskYIELD()` | `tx_thread_relinquish()` | `k_yield()` |
| Sleep API | `vTaskDelay()` | `tx_thread_sleep()` | `k_sleep()` |
| Time slicing | Global configuration | Per thread | Configurable, including priority-based eligibility |
| Preemption-Threshold | No equivalent built-in feature | Yes | No direct equivalent |
| Mixed cooperative and preemptive scheduling classes | Not normally | No separate Zephyr-style classes | Yes |

### 12.5 — Easy Way to Remember

**FreeRTOS:**

"Choose whether the scheduler can preempt tasks."

```text
configUSE_PREEMPTION = 1
          |
          +-- Preemptive

configUSE_PREEMPTION = 0
          |
          +-- Cooperative
```

**ThreadX:**

"Use preemptive scheduling, but a thread can control which priorities are allowed to preempt it."

```text
Thread C: Priority 20, Threshold 15
          |
          +-- Priority 5  : CAN preempt
          |
          +-- Priority 10 : CAN preempt
          |
          +-- Priority 15 : CANNOT preempt
```

**Zephyr:**

"Choose the scheduling class using the thread's priority."

```text
Zephyr Application
       |
       +-- Thread A: K_PRIO_PREEMPT(5)
       |
       +-- Thread B: K_PRIO_COOP(1)
       |
       +-- Both exist together
```

### 12.6 — FreeRTOS vs ThreadX vs Zephyr Comparison

| Feature | FreeRTOS | ThreadX | Zephyr |
|---|---|---|---|
| Preemptive scheduling | Supported | Supported | Supported |
| Cooperative scheduling | Supported through global configuration | Cooperative yielding; no separate cooperative scheduling class | Supported per thread |
| Default approach | Preemptive | Priority-based preemptive | Preemptive threads for normal application work |
| Scheduling configuration | `FreeRTOSConfig.h` | Thread creation parameters and APIs | Kconfig and thread priorities |
| Preemption setting | `configUSE_PREEMPTION=1` | Preemptive by default | `K_PRIO_PREEMPT()` |
| Cooperative setting | `configUSE_PREEMPTION=0` | No global cooperative mode; use yielding or Preemption-Threshold for controlled behavior | `K_PRIO_COOP()` |
| Yield CPU | `taskYIELD()` | `tx_thread_relinquish()` | `k_yield()` |
| Sleep/block | `vTaskDelay()` | `tx_thread_sleep()` | `k_sleep()` |
| Time slicing | `configUSE_TIME_SLICING` | Configurable per thread | `CONFIG_TIMESLICING` |
| Preemption-Threshold | Not built in | Supported | No direct equivalent |
| Mixed preemptive/cooperative tasks | Not as separate per-task scheduling classes in the standard single-core scheduler | No separate cooperative and preemptive classes | Yes |
| Higher-priority number means | Higher number = higher priority | Lower number = higher priority | Lower number = higher priority |

**Easy way to remember:**

- **FreeRTOS:** Choose preemptive or cooperative scheduling globally.
- **ThreadX:** Uses preemptive scheduling with optional Preemption-Threshold to control which higher-priority threads may interrupt a running thread.
- **Zephyr:** Supports cooperative and preemptive threads together using different priority classes.

**Important:** Time slicing and preemption are different. Preemption lets a higher-priority thread interrupt a lower-priority thread. Time slicing allows eligible equal-priority threads to share CPU time.

### Final Understanding

- **FreeRTOS:** Simple global choice between preemptive and cooperative scheduling.
- **ThreadX:** Preemptive scheduling with optional per-thread Preemption-Threshold and time slicing.
- **Zephyr:** Separate cooperative and preemptive priority classes in one application.

All three are real-time operating systems. They simply provide different ways to control which thread gets the CPU.

### 12.7 — Which Should We Use?

**For most beginner and normal RTOS applications, start with preemptive scheduling.**

Why?

1. Important tasks can respond quickly.
2. High-priority tasks don't have to wait for lower-priority tasks to voluntarily yield.
3. It works well with interrupts and event-driven applications.
4. It is commonly used in embedded systems.

Use cooperative scheduling when you specifically need a thread to run without ordinary thread preemption during a controlled operation.

**Warning:** A cooperative thread that never yields, sleeps, or blocks can prevent other threads from running on that CPU.


## 13. Zephyr vs FreeRTOS vs ThreadX

All three systems can create threads/tasks and schedule them. They are **alternative RTOS choices**, not a ladder where Zephyr automatically makes every application faster.

| Concept                | Zephyr                                | FreeRTOS                                | ThreadX                                  |
| ---------------------- | ------------------------------------- | --------------------------------------- | ---------------------------------------- |
| Unit of scheduled work | Thread                                | Task                                    | Thread                                   |
| Create a worker        | `k_thread_create()`                   | `xTaskCreate()` / `xTaskCreateStatic()` | `tx_thread_create()`                     |
| Delay a worker         | `k_sleep()`                           | `vTaskDelay()`                          | `tx_thread_sleep()`                      |
| Voluntarily yield      | `k_yield()`                           | `taskYIELD()`                           | `tx_thread_relinquish()`                 |
| Priority direction     | **Smaller number = higher**           | **Larger number = higher**              | **Smaller number = higher**              |
| Start scheduling       | Normally automatic during Zephyr boot | Typically call `vTaskStartScheduler()`  | Managed by ThreadX startup / entry setup |
| Own stack per worker   | Yes                                   | Yes                                     | Yes                                      |

**Watch out when moving from FreeRTOS!** A FreeRTOS task at priority 5 normally outranks a task at priority 2. In Zephyr, a preemptible thread at priority **2** outranks priority **5**.

### Why does Zephyr look more complicated?

A minimal FreeRTOS project often exposes fewer files and fewer layers. Zephyr includes a larger integrated build/configuration system, hardware descriptions, drivers, optional subsystems, and tools. This can take longer to learn, but can help when developing applications across different boards or requiring more integrated features.

**None of the RTOSes magically creates extra CPU power.** Threads make it easier to organize jobs and respond to important events.

---

## 14. Let's open Zephyr's real source code

Your checkout is at:

`C:\Users\ragha\workspace\personal\zephyrproject\zephyr`

These are the most useful starting locations:

| Path                             | Why you should open it                                                        |
| -------------------------------- | ----------------------------------------------------------------------------- |
| `include/zephyr/kernel.h`        | Public thread and scheduling API declarations.                                |
| `include/zephyr/kernel/thread.h` | Thread-related types and definitions.                                         |
| `kernel/thread.c`                | Thread creation, preparation, and lifecycle implementation.                   |
| `kernel/sched.c`                 | Ready queue, scheduler decisions, rescheduling.                               |
| `kernel/sleep.c`                 | Sleep implementation (check your checkout).                                   |
| `kernel/timeout.c`               | Timeout handling used by timed waits.                                         |
| `kernel/init.c`                  | Initial main thread setup, from Lesson 02.                                    |
| `arch/x86/`                      | Architecture-specific CPU context switching support for your QEMU x86 target. |
| `arch/arm/`                      | Relevant architecture code later when you work with Cortex-M hardware.        |

**Important:** Exact internal functions, filenames, and low-level assembly may change between Zephyr versions or architecture configurations. Use your checked-out source as the final reference.

### Search for thread creation

From PowerShell inside `zephyr/`:

```powershell
Select-String -Path .\kernel\thread.c -Pattern "k_thread_create", "z_setup_new_thread"
```

### Search for scheduling

```powershell
Select-String -Path .\kernel\sched.c -Pattern "z_ready_thread", "reschedule", "z_swap"
```

### Search for sleep logic

```powershell
Select-String -Path .\kernel\sleep.c -Pattern "k_sleep", "z_impl_k_sleep"
```

### Open the source in VS Code

```powershell
code .\kernel\thread.c
code .\kernel\sched.c
code .\arch\x86
```

Don't try to understand all the source code in one day. Follow this small path:

```text
Application calls k_thread_create()
    |
    v
kernel/thread.c prepares the thread
    |
    v
kernel/sched.c makes scheduling decisions
    |
    v
arch/x86 handles required CPU-specific switch work
    |
    v
The chosen worker function begins/resumes executing
```

This is an **overview**, not an exact call graph of every function.

---

## 15. Common beginner mistakes

### Mistake 1 — "Priority 10 is higher than 5"

Not in Zephyr. **5 is higher priority than 10** (within the preemptible range).

### Mistake 2 — "Creating a thread means it runs right now"

It means the thread is created and, with `K_NO_WAIT`, becomes ready immediately. It still needs the scheduler to choose it.

### Mistake 3 — "`k_sleep()` freezes the whole system"

No. It sleeps the **calling thread**, not all threads.

### Mistake 4 — "All threads share one stack"

No. Each thread has its own stack area.

### Mistake 5 — "If I see two alternating print messages, I proved exactly two context switches"

No. Printing shows observable work; context switches may also occur around waits, interrupts, and other scheduling events.

### Mistake 6 — "One CPU executes two threads simultaneously"

One CPU core runs one thread at an instant (interrupts aside). Rapid switching is **concurrency**; true simultaneous execution is possible with multiple cores.

### Mistake 7 — "It's fine for a high-priority thread to spin forever"

A high-priority thread that never sleeps, waits, or yields can prevent lower-priority threads from running. Design thread loops to block while idle and use synchronization mechanisms to wait for work.

---

## 16. Useful troubleshooting commands

From the `zephyr/` directory:

```powershell
# Is West available?
west --version

# Did CMake record our target board?
Select-String -Path ..\build\lesson03_threads\zephyr\.config -Pattern "CONFIG_BOARD", "CONFIG_MULTITHREADING"

# Force a clean configuration/build
west build -p always -b qemu_x86 ..\lesson03_threads -d ..\build\lesson03_threads

# Run the existing output
west build -d ..\build\lesson03_threads -t run
```

If `west` is not found, activate the virtual environment from the workspace root. If QEMU fails with Windows error `0xC0000135`, the executable may be missing a DLL dependency on your machine; check the working QEMU/MinGW setup you used for Lesson 01 before trying to change the application code.

If a source compile error happens, check that the three files were saved at the paths above and that the CMake `target_sources` path is `src/main.c`.

---

## 17. Quick self-test (answer before peeking)

1. What three items do you need to create a normal Zephyr thread?
2. Which thread gets CPU time first if both are ready: priority 3 or priority 8?
3. Does `k_sleep()` stop every thread or only the calling thread?
4. What do we mean by **context switching**?
5. Does `K_NO_WAIT` guarantee immediate CPU execution?
6. When a high-priority thread sleeps, can a low-priority thread run?
7. Why do we call `k_thread_join()` in our program?
8. Is an ISR itself an ordinary thread?

<details>
<summary>Answers</summary>

1. Entry function, stack, and thread control block (plus priority/options when creating it).
2. Priority **3**, because smaller numbers have higher priority in Zephyr.
3. Only the calling thread.
4. Saving/restoring enough CPU and thread state to stop one thread and resume another.
5. No. It makes the new thread ready; the scheduler decides when it executes.
6. Yes, if the lower-priority thread is ready.
7. So main waits until both workers terminate.
8. No. An ISR is interrupt-handling code.

</details>

---

## 18. The whole lesson in one picture

```text
           ZEPHYR STARTS
                |
                v
        MAIN THREAD RUNS
                |
                v
       k_thread_create()
          /           \
         v             v
   HIGH WORKER      LOW WORKER
   priority 5        priority 7
         \             /
          \           /
           v         v
          READY THREADS
                |
                v
             SCHEDULER
                |
         choose best ready
                |
                v
             ONE CPU
                |
      +---------+---------+
      |                   |
   RUN HIGH             RUN LOW
      |                   |
    sleeps               sleeps
      |                   |
      +----- other thread gets CPU
                |
                v
        context switches
        happen as needed
                |
                v
        both workers finish
                |
                v
           main() exits
```

### The five facts to remember

1. **A thread is one independent job.**
2. **Every thread needs its own stack and kernel bookkeeping.**
3. **The scheduler chooses among ready threads.**
4. **In Zephyr, smaller priority numbers have higher priority.**
5. **Context switching lets a CPU save one job and resume another.**

### Next lesson

**Lesson 04 — Zephyr Semaphores, Mutexes, and Race Conditions**

We'll create two threads that share data, intentionally show why unprotected shared data is dangerous, and then fix it using Zephyr's synchronization APIs.

---

## Official references

These are the primary references behind this lesson. For exact details, always compare with the version checked out on your own machine.

- [Zephyr — Threads](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html)
- [Zephyr — Scheduling](https://docs.zephyrproject.org/latest/kernel/services/scheduling/index.html)
- [Zephyr — Thread APIs](https://docs.zephyrproject.org/latest/doxygen/html/group__thread__apis.html)
- [Zephyr source — `kernel/thread.c`](https://github.com/zephyrproject-rtos/zephyr/blob/main/kernel/thread.c)
- [Zephyr source — `kernel/sched.c`](https://github.com/zephyrproject-rtos/zephyr/blob/main/kernel/sched.c)
- [Zephyr architecture porting — context switching](https://github.com/zephyrproject-rtos/zephyr/blob/main/doc/hardware/porting/arch.rst)

*Prepared as Lesson 03 of a beginner-friendly Zephyr RTOS series.*
