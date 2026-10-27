# Lesson 04 — Zephyr Semaphores, Mutexes, and Race Conditions

> **Learning level:** Beginner. Imagine you are explaining the RTOS to a 10-year-old, but keep the real technical names so you can read Zephyr source code later.
>
> **Test platform:** Windows PowerShell + Zephyr + QEMU `qemu_x86` (the same setup used in Lessons 01–03).
>
> **Our project rule:** All learning applications go under **`zephyr/myapp/`**, not beside the `zephyr/` directory. Build output goes under the workspace-level `build/` directory.

## 1. What will we learn?

By the end of this lesson, you should understand:

1. Why two threads can accidentally damage shared data (a **race condition**).
2. Why `counter++` is not automatically a safe operation between threads.
3. How a **mutex** makes sure only one thread uses a shared resource at a time.
4. How a **semaphore** tells a thread that something is ready, or counts available slots.
5. What **blocking**, `K_NO_WAIT`, `K_FOREVER`, and timeouts mean.
6. The difference between **binary** and **counting** semaphores.
7. Why **priority inversion** matters and how mutex priority inheritance helps.
8. Which APIs to use in **Zephyr, FreeRTOS, and ThreadX**.
9. How to build and run **three small experiments on QEMU**.

**One sentence to remember:** A *mutex protects a shared thing*; a *semaphore signals or counts something*.

---

## 2. The school analogy: one notebook and two children

Imagine there is **one notebook** on a classroom desk. Gaurav and Saurav both need to change the number written in it.

The notebook initially says **0**.

- Gaurav reads **0** and plans to write **1**.
- Saurav also reads **0** and plans to write **1**.
- Gaurav writes **1**.
- Saurav writes **1**.

But two children each added one. The correct answer should have been **2**!

```text
Shared counter starts at 0

Gaurav: READ 0 ----------------------- WRITE 1
Saurav:           READ 0 ----- WRITE 1

Final value: 1       Expected value: 2
```

This is a **lost update**, which is one example of a **race condition**.

### What does "race condition" mean?

A race condition happens when a program's correctness depends on the timing or order of actions by different threads or interrupt handlers, without proper synchronization.

It can appear only sometimes, which makes it difficult to debug.

### Why can `counter++` be unsafe?

The C statement looks like one operation:

```c
counter++;
```

But conceptually it can involve separate steps:

```text
1. Read counter from memory
2. Add 1
3. Write the new value back to memory
```

Another thread might execute between those steps. **Do not assume `volatile` fixes this**: `volatile` does not make an increment atomic and does not replace synchronization.

> The exact machine instructions depend on the CPU and compiler. The three steps above are a useful learning model, not a promise about every instruction sequence.

---

## 3. What is a critical section?

A **critical section** is a piece of code that accesses shared data and must not be overlapped unsafely by other code using that same data.

Example:

```c
shared_counter++;
```

If several threads update the same counter, this line belongs in a protected region.

```text
Thread A             SHARED COUNTER             Thread B
    |                      0                       |
    +----- lock ---------->|<------- wait ---------+
    |                 add safely                   |
    +----- unlock -------->|                        |
                           |<-------- lock --------+
                           |                 add safely
                           |<------- unlock -------+
```

A mutex is a common way to provide this protection *between threads*.

---

## 4. Mutex: the key to a room

**Mutex** is short for **mutual exclusion**.

Imagine a small room with only **one key**:

- The child holding the key can enter and use the notebook.
- Other children wait outside.
- The child returns the key when finished.
- The next waiting child can enter.

```text
            [ MUTEX: one key ]
                     |
          +----------+----------+
          |                     |
      Thread A              Thread B
      GETS KEY              MUST WAIT
          |
      changes data
          |
      RETURNS KEY ----------> CAN GET KEY
```

### Zephyr mutex APIs

```c
K_MUTEX_DEFINE(counter_mutex);                   // Make a mutex

k_mutex_lock(&counter_mutex, K_FOREVER);         // Take the key
shared_counter++;                                // Protected work
k_mutex_unlock(&counter_mutex);                 // Return the key
```

`K_FOREVER` means *wait as long as needed* if the mutex is already locked.

**Important mutex rules:**

- A mutex has an **owner**: the thread that locked it must unlock it.
- Zephyr mutexes are **recursive/reentrant**: the owning thread can lock the same mutex multiple times, but must unlock it the same number of times.
- Keep the protected region short so other threads don't wait unnecessarily.
- Zephyr mutexes **must not be used from an interrupt service routine (ISR)**.
- Mutexes are mainly for shared resource ownership, **not** for announcing that new data has arrived.

---

## 5. Semaphore: tokens or a doorbell

Imagine a teacher puts **tokens in a box**. A child must collect a token to use one of several available toys.

A semaphore has a **count** and a **maximum limit**:

- `k_sem_give()` adds an available token (up to the limit).
- `k_sem_take()` removes a token if one is available.
- If there are no tokens, a thread can **wait (block)**.

A semaphore can also be used as a **doorbell**:

```text
Producer/Interrupt       SEMAPHORE            Consumer thread
       |                 count = 0                   |
       |                                            |
       |                                      k_sem_take()
       |                                      (goes to sleep)
       |                                            |
  New data arrives                                 |
       |                                            |
   k_sem_give() ------------ token ----------------> wakes up
                                                    |
                                               processes data
```

### Binary semaphore

A **binary semaphore** has a maximum count of **1**.

```c
K_SEM_DEFINE(data_ready, 0, 1);
```

Meaning: start with **0** tokens (nothing ready), and allow at most **1** token (a signal is pending).

Useful for: *wake up the worker when an event occurs*.

### Counting semaphore

A **counting semaphore** can hold more than one token.

```c
K_SEM_DEFINE(available_slots, 3, 3);
```

Meaning: start with **3** available slots. Three successful takes would reduce the count to zero; the next take would have to wait or fail.

Useful for: *counting limited resources or pending events*, as long as the counting semantics match your design.

### Semaphore APIs

```c
K_SEM_DEFINE(my_sem, 0, 1);      // Initial count 0, maximum count 1

k_sem_give(&my_sem);             // Signal / add token
k_sem_take(&my_sem, K_FOREVER);  // Wait for token
```

**Unlike a mutex, a semaphore doesn't record a thread owner.** One thread or ISR can give a semaphore and another thread can take it.

> A semaphore stores a **count**, not the actual message payload. To transfer many data items, learn Zephyr queues/FIFOs/message queues in a later lesson.

---

## 6. Mutex vs semaphore: choose the right tool

| Question | Mutex | Semaphore |
|---|---|---|
| Main purpose | Protect one shared resource | Signal an event or count resources |
| Simple analogy | One room key | Doorbell or bucket of tokens |
| Has a thread owner? | **Yes** | **No** |
| Same thread must release? | **Yes** | Not necessarily |
| Can signal from an ISR? | **No** | **Yes**, using `k_sem_give()` |
| Priority inheritance? | **Yes** | **No**, not in the mutex sense |
| Zephyr creation macro | `K_MUTEX_DEFINE()` | `K_SEM_DEFINE()` |
| Take/lock API | `k_mutex_lock()` | `k_sem_take()` |
| Give/unlock API | `k_mutex_unlock()` | `k_sem_give()` |
| Typical use | Shared counter, shared I2C bus | Sensor-ready notification, available-buffer count |

**Beginner rule:** If you are thinking "only one thread may touch this shared thing now," reach for a **mutex**. If you are thinking "wake this thread when an event happens," reach for a **semaphore**.

---

## 7. What does blocking mean?

When a thread **blocks** on an unavailable mutex or semaphore, it does **not** sit inside a busy `while(1)` loop using all the CPU time.

Instead:

1. The kernel marks the thread as not ready to run.
2. The scheduler selects another ready thread.
3. When the mutex becomes available or the semaphore is given, the waiting thread can become ready again.
4. It runs when the scheduler selects it.

```text
Consumer thread:
RUNNING -> k_sem_take(empty) -> BLOCKED -> producer gives -> READY -> RUNNING
```

### Three common timeout choices

```c
k_sem_take(&my_sem, K_NO_WAIT);       // Never wait: succeed or fail immediately
k_sem_take(&my_sem, K_MSEC(100));     // Wait up to about 100 ms
k_sem_take(&my_sem, K_FOREVER);       // Wait indefinitely
```

For mutexes, the corresponding call is `k_mutex_lock()`.

**Always check a return value when using `K_NO_WAIT` or a finite timeout.** For example:

```c
if (k_sem_take(&data_ready, K_MSEC(100)) == 0) {
    printk("Got the signal!\n");
} else {
    printk("Nothing arrived before timeout.\n");
}
```

A zero return value means successful acquisition. A negative value indicates failure, such as a timeout or immediate unavailability.

---

## 8. Our project folder (important!)

All new learning code goes **inside `zephyr/myapp/`**:

```text
zephyrproject/
├── .venv/
├── zephyr/                            <-- Zephyr RTOS repository
│   ├── kernel/
│   ├── drivers/
│   ├── samples/
│   └── myapp/                         <-- OUR learning code
│       └── lesson04_sync/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson04_sync/                 <-- Firmware build output
```

We'll use **one application folder** and try three versions of `src/main.c`, in order:

1. **Experiment A — Race condition:** observe the lost update.
2. **Experiment B — Mutex:** protect the counter and get the right answer.
3. **Experiment C — Semaphore:** pass five values between two threads using signals.

We replace `main.c` for each experiment (save older versions somewhere separately if you want to keep all three). We do **not** edit Zephyr's kernel or official samples.

### Step 1 — Open PowerShell and create the app

From any PowerShell prompt:

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject
.\.venv\Scripts\Activate.ps1
cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson04_sync\src | Out-Null
```

> Use `Activate.ps1` in PowerShell, **not** `activate.bat`. The virtual environment lives one level above `zephyr/`.

### Step 2 — Create `prj.conf`

File: `zephyr/myapp/lesson04_sync/prj.conf`

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
```

- `CONFIG_MULTITHREADING=y`: lets our application run multiple threads.
- `CONFIG_PRINTK=y`: enables simple terminal printing.

We don't need to enable time slicing. Our threads explicitly block or sleep, giving the scheduler opportunities to switch.

### Step 3 — Create `CMakeLists.txt`

File: `zephyr/myapp/lesson04_sync/CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson04_sync)

target_sources(app PRIVATE src/main.c)
```

### Step 4 — Build and run (reuse for every experiment)

From `zephyrproject/zephyr`:

```powershell
west build -b qemu_x86 .\myapp\lesson04_sync -d ..\build\lesson04_sync
west build -d ..\build\lesson04_sync -t run
```

After replacing `main.c`, build again with the same command. If build configuration needs to be reset:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson04_sync -d ..\build\lesson04_sync
```

To exit QEMU when it remains running: press **Ctrl+A**, release, then press **X**. The exact boot banner depends on the installed Zephyr version.

---

## 9. Experiment A — See a race condition happen

### Goal

Make two threads both read **0**, then each write back **1**. We deliberately force the unsafe order so the lost update is predictable on QEMU. This demonstrates a real lost-update problem, without relying on luck or random CPU timing.

### `zephyr/myapp/lesson04_sync/src/main.c` — Experiment A

```c
/**
 * @file main.c
 * @brief Experiment A: Demonstrating a lost-update race condition.
 *
 * @details
 * Two threads increment a shared counter without a mutex.
 * Semaphores force both threads to read the same value
 * before either thread writes.
 *
 * Expected counter = 2
 * Actual counter   = 1 (lost update)
 */

#include <zephyr/kernel.h>

/* Thread configuration. */
#define STACK_SIZE 1024

/* Shared variable accessed without mutex protection. */
static int shared_counter = 0;

/* Synchronization semaphores. */
K_SEM_DEFINE(both_have_read, 0, 2);
K_SEM_DEFINE(allow_write, 0, 2);
K_SEM_DEFINE(workers_done, 0, 2);

/* Static function declarations. */
static void unsafe_increment(const char *name);
static void worker_a(void *a, void *b, void *c);
static void worker_b(void *a, void *b, void *c);

/* Create two worker threads with equal priority. */
K_THREAD_DEFINE(Gaurav_tid, STACK_SIZE, worker_a,
                NULL, NULL, NULL, K_PRIO_PREEMPT(5), 0, 0);

K_THREAD_DEFINE(Saurav_tid, STACK_SIZE, worker_b,
                NULL, NULL, NULL, K_PRIO_PREEMPT(5), 0, 0);

/**
 * @brief Main thread controls worker execution.
 */
int main(void)
{
    printk("\n--- Experiment A: LOST UPDATE ---\n");

    /* Wait until both workers read the shared counter. */
    k_sem_take(&both_have_read, K_FOREVER);
    k_sem_take(&both_have_read, K_FOREVER);

    printk("Both read the OLD value. Now let them write.\n");

    /* Allow both workers to write their previously read values. */
    k_sem_give(&allow_write);
    k_sem_give(&allow_write);

    /* Wait until both workers finish writing. */
    k_sem_take(&workers_done, K_FOREVER);
    k_sem_take(&workers_done, K_FOREVER);

    printk("Final counter = %d, expected = 2\n", shared_counter);

    return 0;
}

/**
 * @brief Demonstrates a non-atomic read-modify-write operation.
 *
 * Both threads read the counter before either writes.
 * Consequently, one increment is lost.
 */
static void unsafe_increment(const char *name)
{
    /* Read the current shared value. */
    int old_value = shared_counter;

    printk("%s read counter = %d\n", name, old_value);

    /* Notify main that this worker has completed its read. */
    k_sem_give(&both_have_read);

    /* Wait until main permits the write operation. */
    k_sem_take(&allow_write, K_FOREVER);

    /* Write using the old value, causing a lost update. */
    shared_counter = old_value + 1;

    printk("%s wrote counter = %d\n", name, shared_counter);

    /* Notify main that this worker has finished. */
    k_sem_give(&workers_done);
}

/**
 * @brief Gaurav worker thread.
 */
static void worker_a(void *a, void *b, void *c)
{
    ARG_UNUSED(a);
    ARG_UNUSED(b);
    ARG_UNUSED(c);

    unsafe_increment("Gaurav");
}

/**
 * @brief Saurav worker thread.
 */
static void worker_b(void *a, void *b, void *c)
{
    ARG_UNUSED(a);
    ARG_UNUSED(b);
    ARG_UNUSED(c);

    unsafe_increment("Saurav");
}
```

### Expected output (order may vary)

```text
*** Booting Zephyr OS build ... ***

--- Experiment A: LOST UPDATE ---
Gaurav read counter = 0
Saurav read counter = 0
Both read the OLD value. Now let them write.
Gaurav wrote counter = 1
Saurav wrote counter = 1
Final counter = 1, expected = 2
```

**Why does this happen?** Both workers used the same *old* snapshot. Synchronizing when they start writing does **not** fix the missing protection between *read* and *write*.

> **Important technical detail:** The example intentionally uses semaphores to **force an unsafe interleaving**. It is a teaching demonstration of a lost update, not a claim that every unsynchronized `counter++` must always fail. In real C programs, unsynchronized concurrent access to shared data can also be undefined behavior under the C memory model; use proper synchronization in production.

### Diagram

```text
Gaurav                        Saurav                        Main
  |                           |                           |
read 0                     read 0                         |
  |                           |                           |
signal read                signal read                    |
  |                           |                           |
WAIT <----------------------WAIT------------------ sees both reads
  |                           |                           |
  +-------- both allowed to write ------------------------+
  |                           |                           |
write 1                     write 1                       |
  |                           |                           |
  +------------ both signal completion ------------------>|
                                                     final = 1
```

---

## 10. Experiment B — Fix the shared counter using a mutex

### Goal

Two threads each add **5** to the same shared counter. If we use the mutex correctly, the expected final answer is **10**.

### Replace `zephyr/myapp/lesson04_sync/src/main.c` with this code

```c
/**
 * @file main.c
 * @brief Experiment B: Thread synchronization using Mutex and Semaphore.
 *
 * @details
 * Two threads increment a shared counter five times each.
 *
 * - Mutex protects the shared counter from race conditions.
 * - Semaphore notifies main when both workers finish.
 *
 * Expected final counter = 10.
 */

#include <zephyr/kernel.h>

/* Thread configuration. */
#define STACK_SIZE               1024
#define INCREMENTS_PER_THREAD    5

/* Shared resource accessed by both threads. */
static int shared_counter = 0;

/* Mutex protects shared_counter. */
K_MUTEX_DEFINE(counter_mutex);

/* Semaphore tracks completion of both worker threads. */
K_SEM_DEFINE(workers_done, 0, 2);

/* Static function declarations. */
static void count_five_times(const char *name);
static void worker_a(void *a, void *b, void *c);
static void worker_b(void *a, void *b, void *c);

/* Create two worker threads with equal priority. */
K_THREAD_DEFINE(Gaurav_tid, STACK_SIZE, worker_a,
                NULL, NULL, NULL, K_PRIO_PREEMPT(5), 0, 0);

K_THREAD_DEFINE(Saurav_tid, STACK_SIZE, worker_b,
                NULL, NULL, NULL, K_PRIO_PREEMPT(5), 0, 0);

/**
 * @brief Main thread waits for both workers to finish.
 */
int main(void)
{
    printk("\n--- Experiment B: MUTEX ---\n");

    /* Wait for completion signals from both workers. */
    k_sem_take(&workers_done, K_FOREVER);
    k_sem_take(&workers_done, K_FOREVER);

    /* Both workers have completed their increments. */
    printk("Final counter = %d, expected = 10\n", shared_counter);

    return 0;
}

/**
 * @brief Safely increment the shared counter five times.
 *
 * @param name Name of the worker thread.
 */
static void count_five_times(const char *name)
{
    for (int i = 0; i < INCREMENTS_PER_THREAD; ++i)
    {
        /* Acquire mutex before accessing the shared resource. */
        k_mutex_lock(&counter_mutex, K_FOREVER);

        /* Begin critical section. */
        shared_counter++;

        printk("%s: shared_counter = %d\n",
               name, shared_counter);
        /* End critical section. */

        /* Release mutex so another thread can access the counter. */
        k_mutex_unlock(&counter_mutex);

        /* Allow other ready threads to execute. */
        k_sleep(K_MSEC(20));
    }

    /* Notify main that this worker has completed. */
    k_sem_give(&workers_done);
}

/**
 * @brief Worker A (Gaurav) thread entry.
 */
static void worker_a(void *a, void *b, void *c)
{
    ARG_UNUSED(a);
    ARG_UNUSED(b);
    ARG_UNUSED(c);

    count_five_times("Gaurav");
}

/**
 * @brief Worker B (Saurav) thread entry.
 */
static void worker_b(void *a, void *b, void *c)
{
    ARG_UNUSED(a);
    ARG_UNUSED(b);
    ARG_UNUSED(c);

    count_five_times("Saurav");
}
```

### Build and run again

```powershell
west build -b qemu_x86 .\myapp\lesson04_sync -d ..\build\lesson04_sync
west build -d ..\build\lesson04_sync -t run
```

### Expected behavior

You should see **10 increment messages**, in an order decided by the scheduler, followed by:

```text
--- Experiment B: MUTEX ---
Gaurav: shared_counter = 1
Saurav: shared_counter = 2
Gaurav: shared_counter = 3
Saurav: shared_counter = 4
Gaurav: shared_counter = 5
Saurav: shared_counter = 6
Gaurav: shared_counter = 7
Saurav: shared_counter = 8
Gaurav: shared_counter = 9
Saurav: shared_counter = 10
Final counter = 10, expected = 10
```

**Why does this work?** A worker must successfully lock the mutex before it reads/modifies/writes the shared value. The second worker waits until the first unlocks it.

**What is the semaphore doing here?** `workers_done` is *not* protecting `shared_counter`; it only informs `main()` that both workers have completed. That's a good real-world example of using a **mutex and semaphore for different jobs**.

> To keep the output easy to read, `printk()` is inside the mutex-protected region. In production, try to keep locks short; usually capture the new value while locked, unlock, then log appropriately.

---

## 11. Experiment C — Producer/consumer using semaphores

### Goal

Imagine there is **one mailbox** between two threads:

- **Producer** puts a number in the mailbox.
- **Consumer** reads that number.
- Producer must **not overwrite** a number before Consumer has read it.

We will use **two binary semaphores**:

1. `slot_free`: starts at 1 — the empty mailbox can accept one value.
2. `item_ready`: starts at 0 — there is initially nothing to read.

```text
    PRODUCER                 ONE-SLOT MAILBOX                 CONSUMER
        |                          |                               |
  take(slot_free)                  |                               |
  write value ------------------->|                               |
  give(item_ready) ---------------+------------------------------> wake
        |                          |                             read
  wait for free slot <------------+----------------------- give(slot_free)
        |
  write next value ...
```

### Replace `zephyr/myapp/lesson04_sync/src/main.c` with this code

```c
#include <zephyr/kernel.h>

#define STACK_SIZE 1024
#define ITEM_COUNT 5

static int mailbox_value = 0;

/* One empty slot exists initially; no item exists initially. */
K_SEM_DEFINE(slot_free, 1, 1);
K_SEM_DEFINE(item_ready, 0, 1);
K_SEM_DEFINE(workers_done, 0, 2);

static void producer(void *a, void *b, void *c)
{
    ARG_UNUSED(a);
    ARG_UNUSED(b);
    ARG_UNUSED(c);

    for (int i = 1; i <= ITEM_COUNT; ++i) {
        /* Wait until the mailbox has space. */
        k_sem_take(&slot_free, K_FOREVER);

        mailbox_value = i * 10;
        printk("Producer placed: %d\n", mailbox_value);

        /* Tell the consumer a new item exists. */
        k_sem_give(&item_ready);
        k_sleep(K_MSEC(20));
    }

    k_sem_give(&workers_done);
}

static void consumer(void *a, void *b, void *c)
{
    ARG_UNUSED(a);
    ARG_UNUSED(b);
    ARG_UNUSED(c);

    for (int i = 0; i < ITEM_COUNT; ++i) {
        /* Wait for something to arrive. */
        k_sem_take(&item_ready, K_FOREVER);

        int value = mailbox_value;
        printk("Consumer received: %d\n", value);

        /* Now the producer may reuse the one-slot mailbox. */
        k_sem_give(&slot_free);
        k_sleep(K_MSEC(50));
    }

    k_sem_give(&workers_done);
}

K_THREAD_DEFINE(producer_tid, STACK_SIZE, producer,
                NULL, NULL, NULL, K_PRIO_PREEMPT(5), 0, 0);
K_THREAD_DEFINE(consumer_tid, STACK_SIZE, consumer,
                NULL, NULL, NULL, K_PRIO_PREEMPT(5), 0, 0);

int main(void)
{
    printk("\n--- Experiment C: SEMAPHORE HANDOFF ---\n");

    k_sem_take(&workers_done, K_FOREVER);
    k_sem_take(&workers_done, K_FOREVER);

    printk("All %d items transferred successfully.\n", ITEM_COUNT);
    return 0;
}
```

### Build and run again

```powershell
west build -b qemu_x86 .\myapp\lesson04_sync -d ..\build\lesson04_sync
west build -d ..\build\lesson04_sync -t run
```

### Expected output (interleaving may vary)

```text
--- Experiment C: SEMAPHORE HANDOFF ---
Producer placed: 10
Consumer received: 10
Producer placed: 20
Consumer received: 20
Producer placed: 30
Consumer received: 30
Producer placed: 40
Consumer received: 40
Producer placed: 50
Consumer received: 50
All 5 items transferred successfully.
```

**Why use two semaphores?** One says *there is a filled mailbox*; the other says *the mailbox is empty again*. This prevents overwriting unread data.

**Why not just use one semaphore?** If the producer could write many times before the consumer reads, one shared `int` would lose older values. Counting notifications alone does not magically store each item.

For a real multi-item producer/consumer application, a queue (for example, `k_msgq`) is usually a better design. That's a topic for a later lesson.

---

## 12. Priority inversion — when the important child waits

Imagine three threads:

| Thread | Zephyr priority | What it does |
|---|---:|---|
| HIGH | 3 | Wants access to shared resource immediately |
| MEDIUM | 5 | Does unrelated CPU work |
| LOW | 7 | Currently owns the resource mutex |

**Zephyr rule:** A **smaller** priority number means a **higher** scheduling priority.

Without a helpful mechanism:

1. LOW locks the mutex.
2. HIGH becomes ready and tries to lock it, so HIGH blocks.
3. MEDIUM becomes ready and may keep running ahead of LOW.
4. LOW cannot finish and release the mutex promptly.
5. HIGH keeps waiting even though MEDIUM doesn't use the mutex!

This is called **priority inversion**.

```text
LOW owns mutex
    |
HIGH wants mutex -> BLOCKED
    |
MEDIUM runs ahead of LOW
    |
LOW cannot finish quickly -> HIGH waits longer
```

### Mutex priority inheritance

Zephyr mutexes implement **priority inheritance**. When a higher-priority thread waits for a mutex owned by a lower-priority thread, the owning thread can **temporarily inherit a higher effective priority** so it can run and release the lock sooner.

```text
LOW owns mutex
HIGH blocks on mutex
      |
LOW temporarily inherits HIGH's priority
      |
LOW runs, finishes protected work, unlocks
      |
HIGH can acquire mutex and continue
```

**Two cautions:**

- Priority inheritance helps with mutex-related priority inversion; it is not a fix for every scheduling delay or every deadlock.
- A semaphore is **not** a substitute for a mutex when you need ownership and priority inheritance.

---

## 13. Deadlock — two children each hold one key

Imagine two rooms with two different keys:

```text
Thread A: locks MUTEX_1 -> waits for MUTEX_2
Thread B: locks MUTEX_2 -> waits for MUTEX_1

Neither thread can continue.
```

This is a **deadlock**.

### Easy ways to reduce deadlocks

- Always acquire multiple mutexes in a **consistent order** across all threads.
- Unlock everything you lock; make error-handling paths do the same.
- Avoid waiting forever for unrelated operations while holding a mutex.
- Use timeouts where they fit the design.
- Keep critical sections short.

A mutex protects your shared resource, but careless locking can still freeze the application.

---

## 14. Interrupts (ISRs): which one can you use?

**ISR** means **Interrupt Service Routine** — code that runs when hardware needs quick attention.

Example: a UART interrupt announces that new bytes arrived.

**Zephyr rules:**

- `k_mutex_lock()` / `k_mutex_unlock()` are for **thread context**, not ISR context.
- An ISR **can call `k_sem_give()`** to wake a waiting thread.
- If an ISR calls `k_sem_take()`, it must use **`K_NO_WAIT`**; an ISR cannot sleep or wait for a future event.
- Keep ISRs short; let a normal worker thread do longer processing.

```c
/* Conceptual split, not a full interrupt registration example. */

K_SEM_DEFINE(rx_ready, 0, 1);

void uart_irq_handler(const void *arg)
{
    ARG_UNUSED(arg);
    /* Acknowledge/handle the actual hardware interrupt as required. */
    k_sem_give(&rx_ready);
}

void rx_worker(void)
{
    while (1) {
        k_sem_take(&rx_ready, K_FOREVER);
        /* Process UART data here in thread context. */
    }
}
```

> This snippet illustrates the **signal/wakeup relationship only**. Actual UART ISR setup needs the correct driver, interrupt status handling, and board configuration.

---

## 15. Zephyr vs FreeRTOS vs ThreadX

All three RTOSes offer synchronization tools. Their names and a few behaviors differ.

| Feature | Zephyr | FreeRTOS | ThreadX |
|---|---|---|---|
| Create mutex | `K_MUTEX_DEFINE()` / `k_mutex_init()` | `xSemaphoreCreateMutex()` | `tx_mutex_create()` |
| Lock mutex | `k_mutex_lock()` | `xSemaphoreTake()` | `tx_mutex_get()` |
| Unlock mutex | `k_mutex_unlock()` | `xSemaphoreGive()` | `tx_mutex_put()` |
| Create binary semaphore | `K_SEM_DEFINE(sem, 0, 1)` | `xSemaphoreCreateBinary()` | `tx_semaphore_create(..., 0)` |
| Take semaphore | `k_sem_take()` | `xSemaphoreTake()` | `tx_semaphore_get()` |
| Give semaphore | `k_sem_give()` | `xSemaphoreGive()` | `tx_semaphore_put()` |
| Timeout style | `K_NO_WAIT`, `K_MSEC(n)`, `K_FOREVER` | `0`, tick count, `portMAX_DELAY` (configuration-dependent) | `TX_NO_WAIT`, tick count, `TX_WAIT_FOREVER` |
| Mutex priority inheritance | Built in | Built in for mutex objects | Optional via `TX_INHERIT` when creating a mutex |
| Mutex ownership | Owner must unlock | Owner task must give | Owner thread must put |
| Mutex from ISR | **No** | **No** | **No** |
| Semaphore give from ISR | `k_sem_give()` | `xSemaphoreGiveFromISR()` | `tx_semaphore_put()` (subject to ThreadX API context rules) |

### The same idea in three RTOSes

**Zephyr**

```c
K_MUTEX_DEFINE(lock);
k_mutex_lock(&lock, K_FOREVER);
shared_counter++;
k_mutex_unlock(&lock);
```

**FreeRTOS** (assuming `lock` was created successfully)

```c
SemaphoreHandle_t lock = xSemaphoreCreateMutex();

if (xSemaphoreTake(lock, portMAX_DELAY) == pdTRUE) {
    shared_counter++;
    xSemaphoreGive(lock);
}
```

**ThreadX** (assuming the mutex object has been successfully created)

```c
TX_MUTEX lock;
/* At initialization time: tx_mutex_create(&lock, "lock", TX_INHERIT); */

if (tx_mutex_get(&lock, TX_WAIT_FOREVER) == TX_SUCCESS) {
    shared_counter++;
    tx_mutex_put(&lock);
}
```

### One extra ThreadX detail

ThreadX also has **Preemption-Threshold**, which you studied in Lesson 03. That is a **scheduling feature**; it is **not a replacement for mutex protection** of shared data. For mutex priority inheritance in ThreadX, use the mutex option `TX_INHERIT` appropriately.

### Same core ideas, different names

```text
Protect shared resource:
  Zephyr:   k_mutex_lock()   / k_mutex_unlock()
  FreeRTOS: xSemaphoreTake() / xSemaphoreGive()  [with a mutex handle]
  ThreadX:  tx_mutex_get()    / tx_mutex_put()

Signal or count events:
  Zephyr:   k_sem_give()        -> k_sem_take()
  FreeRTOS: xSemaphoreGive()    -> xSemaphoreTake()
  ThreadX:  tx_semaphore_put()  -> tx_semaphore_get()
```

> FreeRTOS uses its **semaphore API names even when the object is a mutex**, which can be confusing at first. A FreeRTOS mutex and a FreeRTOS binary semaphore are not the same object internally: mutexes have ownership and priority inheritance behavior.

---

## 16. Which Zephyr source files should we inspect?

From `zephyrproject/zephyr/`, inspect:

| Path | Why it matters |
|---|---|
| `include/zephyr/kernel.h` | Public declarations and API entry points |
| `kernel/sem.c` | Semaphore implementation |
| `kernel/mutex.c` | Mutex behavior, ownership, and priority inheritance |
| `kernel/sched.c` | Scheduler's handling of ready and waiting threads |
| `kernel/thread.c` | Thread management |
| `myapp/lesson04_sync/src/main.c` | **Your** learning application |

Useful PowerShell commands:

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject\zephyr

code .\kernel\sem.c
code .\kernel\mutex.c
code .\kernel\sched.c

Select-String -Path .\kernel\sem.c -Pattern "k_sem_take", "k_sem_give"
Select-String -Path .\kernel\mutex.c -Pattern "k_mutex_lock", "k_mutex_unlock"
```

If `code` isn't on your PATH, use `notepad` or open these files in VS Code's Explorer.

**Study tip:** First understand the public API, then follow how the kernel removes a waiting thread from the scheduler and wakes it again.

---

## 17. Common beginner mistakes

| Mistake | Why it hurts | Better approach |
|---|---|---|
| Using `volatile` as a lock | Does not make multiple operations atomic | Use mutexes or appropriate atomics |
| Forgetting `k_mutex_unlock()` | Other threads may wait forever | Pair every successful lock with an unlock |
| Sleeping for a long time with a mutex held | Other threads wait longer | Make the critical section short |
| Taking a mutex in an ISR | Mutex API isn't designed for ISRs | Give a semaphore or use an ISR-safe mechanism |
| Using one binary semaphore to "store" multiple messages | Semaphore remembers a count, not payloads | Use a message queue/FIFO |
| Assuming `k_sem_give()` always adds another count | Count stops at configured maximum | Choose a suitable limit and design |
| Using `K_FOREVER` without thinking about failure modes | Could block forever on a forgotten signal | Use timeouts when necessary |
| Thinking a mutex protects unrelated accesses automatically | Every competing access must follow the same locking rule | Lock around **all** accesses to shared state |
| Confusing priority values across RTOSes | FreeRTOS and Zephyr use opposite numeric conventions | FreeRTOS: larger = higher; Zephyr/ThreadX: smaller = higher |

---

## 18. Quick revision quiz

Try answering before looking at the solutions.

1. Gaurav and Saurav both increase the same counter. What problem can happen?
2. Which Zephyr object is best to protect a shared I2C device: a mutex or semaphore?
3. Which object is a good choice to notify a worker thread when a sensor interrupt occurs?
4. What is the difference between `K_NO_WAIT` and `K_FOREVER`?
5. Can Zephyr code lock a mutex directly inside an ISR?
6. Why might a low-priority thread temporarily run at a higher effective priority?
7. Do semaphores store the actual sensor readings?
8. If both workers add five while holding the same mutex, what final count should we expect from a zero initial count?

<details>
<summary>Show answers</summary>

1. A race condition, such as a lost update.
2. A mutex (for thread-based exclusive ownership).
3. A semaphore (given from ISR, taken by worker thread).
4. `K_NO_WAIT` doesn't block; `K_FOREVER` can wait indefinitely.
5. No.
6. Mutex priority inheritance can help resolve priority inversion.
7. No, use a suitable data transport such as `k_msgq` for stored messages.
8. Ten.

</details>

---

## 19. Lesson 04 checklist

- [ ] I understand how a lost update can happen.
- [ ] I can explain **mutex = one key**.
- [ ] I can explain **semaphore = tokens/doorbell**.
- [ ] I created `zephyr/myapp/lesson04_sync/`.
- [ ] I built and ran Experiment A (final count **1**, not 2).
- [ ] I built and ran Experiment B (final count **10**).
- [ ] I built and ran Experiment C (all **5** values transferred).
- [ ] I understand the purpose of `K_FOREVER` and `K_NO_WAIT`.
- [ ] I know why mutexes are not for ISRs.
- [ ] I know the Zephyr, FreeRTOS, and ThreadX API names.
- [ ] I opened `kernel/sem.c` and `kernel/mutex.c`.

### Mini challenge

After successfully running Experiment B:

1. Change `INCREMENTS_PER_THREAD` from `5` to `10`.
2. Update the expected final number from `10` to `20`.
3. Rebuild and run QEMU.
4. Explain why the mutex still makes the total correct.

Then, in Experiment C, change `ITEM_COUNT` from `5` to `8` and check that all eight values arrive in order.

---

## 20. The big picture

```text
                      MULTIPLE THREADS
                             |
             +---------------+----------------+
             |                                |
     Share one resource                Need to signal work
             |                                |
           MUTEX                          SEMAPHORE
             |                                |
       lock -> work -> unlock        give -> wake -> take
             |                                |
      No unsafe overlap               Event/resource count
             |                                |
     e.g. shared counter                e.g. sensor-ready
```

**What you should remember most:**

- A **race condition** is a bug caused by uncontrolled ordering of shared operations.
- A **mutex** protects shared data by giving exclusive access to one owning thread.
- A **semaphore** helps threads wait for events or count resources.
- Proper synchronization lets the scheduler switch threads **without breaking your data**.

### Next lesson

**Lesson 05 — Zephyr Queues, FIFOs, and Message Passing:** Instead of just telling a thread *something is ready*, we'll send the actual data between threads safely.

---

## Official references

- [Zephyr documentation — Semaphores](https://docs.zephyrproject.org/latest/kernel/services/synchronization/semaphores.html)
- [Zephyr documentation — Mutexes](https://docs.zephyrproject.org/latest/kernel/services/synchronization/mutexes.html)
- [Zephyr documentation — Threads](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html)
- [Zephyr documentation — Kernel services](https://docs.zephyrproject.org/latest/kernel/services/index.html)
- [FreeRTOS documentation](https://www.freertos.org/Documentation/)
- [Eclipse ThreadX documentation](https://github.com/eclipse-threadx/rtos-docs-asciidoc)

> The commands, directory paths, and source examples are written for your **Windows PowerShell + `qemu_x86`** learning workflow. Some lower-level implementation details depend on the exact Zephyr revision and CPU architecture.
