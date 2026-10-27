# 06 — Zephyr Events, Polling, and Thread Notifications

> **Learning series:** Zephyr RTOS from zero, explained in simple technical language  
> **Target:** QEMU (`qemu_x86`) on Windows PowerShell  
> **Application folder:** `zephyrproject/zephyr/myapp/lesson06_events_polling/`  
> **Build folder:** `zephyrproject/build/lesson06_events_polling/`  
> **Also compared with:** FreeRTOS and Eclipse ThreadX

---

## 1. What will we learn?

By the end of this lesson, you will understand:

1. What an **event** is and why a thread waits for one.
2. How to use **event flags** (bits) to describe several conditions at once.
3. The difference between waiting for **ANY** event and **ALL** events.
4. What **polling** means, and why `k_poll()` is **not** the same as repeatedly checking a variable.
5. How one thread can **notify** another thread that work is ready.
6. The difference between **events, semaphores, message queues, and poll signals**.
7. How the same ideas work in **Zephyr, FreeRTOS, and ThreadX**.
8. How to build and run a **complete QEMU example** using your `myapp/` folder.

**Previous lessons:**

- Lesson 03: Threads and the scheduler
- Lesson 04: Semaphores, mutexes, and race conditions
- Lesson 05: Message queues and data between threads

**Lesson 06 answers the question:** *How can a thread wait for different kinds of work without wasting CPU time?*

---

## 2. Think of a child waiting for school announcements

Imagine a school principal waiting for three things:

- The **bus arrived**.
- The **classroom is ready**.
- The **lunch has arrived**.

The principal does **not** want to stand at the gate checking every second.

Instead, the principal says:

> "Tell me when something important happens. I will work on other things—or rest—until you tell me."

In an RTOS:

| School | RTOS |
|---|---|
| Bus arrives | An event becomes true |
| Teacher rings the bell | Another thread/ISR signals something |
| Principal waits | Thread blocks |
| Bell rings | Scheduler makes a waiting thread ready |
| Principal checks which announcement | Thread reads event flags or poll result |

**Big idea:** A blocked thread is *not* burning CPU cycles checking the same value repeatedly.

---

## 3. What is an event?

An **event** is a piece of information saying **something happened** or **some condition is now true**.

Examples in embedded systems:

- `SENSOR_READY`: sensor has finished initialization.
- `WIFI_CONNECTED`: the device connected to Wi-Fi.
- `DATA_RECEIVED`: information has arrived.
- `MOTOR_STOPPED`: motor reached the stopping point.

Some events are represented by **bits** inside one integer.

### 3.1 One integer can carry many event flags

Zephyr's `k_event` stores a **32-bit set of flags**.

```text
Bit number:  7 6 5 4 3 2 1 0
Value:       0 0 0 0 0 1 0 1
                      |   |
                      |   +-- Sensor is ready (bit 0)
                      +------ Network is ready (bit 2)
```

You can define named bits:

```c
#define EVENT_SENSOR_READY  BIT(0)
#define EVENT_NETWORK_READY BIT(1)
#define EVENT_BUTTON       BIT(2)
```

`BIT(0)` equals `0x01`, `BIT(1)` equals `0x02`, and `BIT(2)` equals `0x04`.

Think of each bit as a small light switch: **0 = OFF**, **1 = ON**.

**Important:** Event flags are not a message queue. Setting the same bit ten times does **not** store ten separate events. The bit simply remains set.

---

## 4. Create an event object: macro or API?

Just like semaphores and message queues, Zephyr supports **static definition** and **runtime initialization**.

### Method A — Static macro

```c
#include <zephyr/kernel.h>

K_EVENT_DEFINE(system_events);
```

- Declares the event object.
- Initializes it for use.
- Great when you already know the object is required.

### Method B — Runtime initialization

```c
#include <zephyr/kernel.h>

struct k_event system_events;

int main(void)
{
    k_event_init(&system_events);
    return 0;
}
```

- Your program supplies the storage.
- You initialize it before the threads use it.

**For the QEMU experiment, we use `K_EVENT_DEFINE()` because it is easier to read.**

---

## 5. Tell threads that an event happened

### 5.1 `k_event_post()` — add flags without removing existing flags

```c
k_event_post(&system_events, EVENT_SENSOR_READY);
```

Later:

```c
k_event_post(&system_events, EVENT_NETWORK_READY);
```

Now both bits are set.

```text
Before posting:  0000
Post SENSOR:     0001
Post NETWORK:    0011    <-- both remembered
```

### 5.2 `k_event_set()` — replace the entire flag value

```c
k_event_set(&system_events, EVENT_SENSOR_READY);
```

This replaces the current set with only the specified bits.

```text
Before: 0011  (sensor and network)
Set:    0001  (sensor only)
```

**Easy rule:**

- `k_event_post()` = **OR/add these flags**.
- `k_event_set()` = **replace the current flags**.

### 5.3 `k_event_clear()` — remove selected flags

```c
k_event_clear(&system_events, EVENT_SENSOR_READY);
```

Only the specified bit is cleared.

Use this carefully if multiple threads share one event object. Clearing a bit too early can affect another thread that wants to see it.

---

## 6. Wait for ANY event vs ALL events

### 6.1 ANY: "Wake me when *at least one* thing is ready"

```c
uint32_t result = k_event_wait(
    &system_events,
    EVENT_SENSOR_READY | EVENT_NETWORK_READY,
    false,
    K_FOREVER
);
```

Suppose SENSOR is ready but NETWORK is not:

```text
Wanted:   SENSOR OR NETWORK
Actual:   SENSOR
Result:   Wake up!
```

`k_event_wait()` waits for **any** requested event flag.

### 6.2 ALL: "Wake me only when *everything* is ready"

```c
uint32_t result = k_event_wait_all(
    &system_events,
    EVENT_SENSOR_READY | EVENT_NETWORK_READY,
    false,
    K_FOREVER
);
```

```text
Time 0: SENSOR=0, NETWORK=0  -> WAIT
Time 1: SENSOR=1, NETWORK=0  -> WAIT
Time 2: SENSOR=1, NETWORK=1  -> WAKE UP!
```

This is useful when application startup requires **both** a sensor and a communication interface.

### 6.3 Understand the four parameters

```c
k_event_wait(&system_events, required_bits, false, K_SECONDS(2));
```

| Parameter | Meaning |
|---|---|
| `&system_events` | Which event object? |
| `required_bits` | Which flags do we care about? |
| `false` | **Do not clear all existing event flags before waiting** |
| `K_SECONDS(2)` | Wait up to two seconds |

The `reset` argument is often misunderstood:

- `reset=false`: Keep the current event bits when starting to wait.
- `reset=true`: Clear the event object's stored bits **before** waiting.

It does **not** mean "automatically clear the matching bits after success".

**Return value:** A nonzero bit mask on success, or `0` when the wait times out.

For repeated *consume-and-clear* behavior, look at `k_event_wait_safe()` and `k_event_wait_all_safe()`, which atomically clear received bits. Do not blindly clear shared flags when several threads are waiting.

---

## 7. What is polling?

The word **polling** is used for two different ideas. Be careful!

### 7.1 Bad kind: busy polling

```c
while (!sensor_ready) {
    /* Keep checking again and again */
}
```

The thread can use lots of CPU time doing no useful work. You may also need synchronization to safely read a value changed by another execution context.

**A normal `volatile` variable does not by itself replace proper thread synchronization.**

### 7.2 Better kind: Zephyr `k_poll()`

`k_poll()` means:

> "Kernel, put my thread to sleep until **one of several kernel-object conditions** becomes ready."

For example, one thread waits for:

1. A **semaphore** to become available, **or**
2. A **poll signal** to be raised.

```text
                MAIN THREAD
                     |
                     v
                   k_poll()
                     |
               BLOCK / SLEEP
                     |
         +-----------+-----------+
         |                       |
     Semaphore ready        Signal raised
         |                       |
         +-----------+-----------+
                     |
              Thread becomes READY
                     |
              Check which happened
```

**Important:** Zephyr's `k_poll()` does **not** poll arbitrary global variables and does **not** accept a `k_event` object directly as a poll type. It supports specific kernel-object conditions such as semaphore availability, FIFO/message queue data availability, and poll signals.

---

## 8. `k_poll()` API: step by step

### 8.1 Create a semaphore and a poll signal

```c
K_SEM_DEFINE(data_sem, 0, 1);

struct k_poll_signal stop_signal;
```

Initialize the signal before use:

```c
k_poll_signal_init(&stop_signal);
```

### 8.2 Create a list of things to wait for

```c
struct k_poll_event watched[2];

k_poll_event_init(&watched[0],
                  K_POLL_TYPE_SEM_AVAILABLE,
                  K_POLL_MODE_NOTIFY_ONLY,
                  &data_sem);

k_poll_event_init(&watched[1],
                  K_POLL_TYPE_SIGNAL,
                  K_POLL_MODE_NOTIFY_ONLY,
                  &stop_signal);
```

### 8.3 Wait for any watched condition

```c
int rc = k_poll(watched, 2, K_SECONDS(3));
```

- If one condition becomes ready, `k_poll()` returns.
- If no watched condition occurs before the timeout, it returns an error (typically `-EAGAIN`).
- It is possible for **more than one** condition to be ready when it returns.

### 8.4 Check which condition happened

```c
if (watched[0].state == K_POLL_STATE_SEM_AVAILABLE) {
    /* Semaphore was available when checked */
}

if (watched[1].state == K_POLL_STATE_SIGNALED) {
    /* Signal was raised */
}
```

### 8.5 Important: polling does NOT take ownership

When `k_poll()` reports a semaphore is available, it **does not take the semaphore for you**.

You still need:

```c
if (k_sem_take(&data_sem, K_NO_WAIT) == 0) {
    printk("Semaphore taken!\n");
}
```

If another thread competes for the same semaphore, it might take it first. This is why we check the return value.

### 8.6 How to raise and read a poll signal

Sender:

```c
k_poll_signal_raise(&stop_signal, 1234);
```

Receiver after polling:

```c
unsigned int signaled;
int result;

k_poll_signal_check(&stop_signal, &signaled, &result);

if (signaled != 0U) {
    printk("Signal result = %d\n", result);
}
```

`1234` is just a small integer that the sender chose. It is not a queue of values.

### 8.7 Reusing a poll request

When you poll again, reset the **poll event state**:

```c
watched[0].state = K_POLL_STATE_NOT_READY;
watched[1].state = K_POLL_STATE_NOT_READY;
```

If you already handled a raised poll signal, also reset the **signal object's state**:

```c
k_poll_signal_reset(&stop_signal);
```

Don't reset the signal in a different thread while another thread is handling it: you can create a race and miss a notification.

---

## 9. What does "thread notification" mean in Zephyr?

Imagine Worker A has completed its job and wants to tell Worker B:

> "Wake up! Your work is ready!"

That is a **notification pattern**, not necessarily one specific API.

### 9.1 FreeRTOS has a direct task notification API

FreeRTOS provides `xTaskNotifyGive()`, `ulTaskNotifyTake()`, `xTaskNotify()`, and `xTaskNotifyWait()` (along with related variants).

A notification value/state is associated with a specific task, so many one-to-one signaling patterns need no separate semaphore object.

### 9.2 Zephyr does not have the same one-to-one task-notification API

Zephyr commonly implements the same **behavior** using:

- `k_sem_give()` / `k_sem_take()` for "one worker needs waking".
- `k_event_post()` / `k_event_wait()` for a group of event flags.
- `k_poll_signal_raise()` + `k_poll()` for a signal observed through the polling API.
- `k_msgq_put()` / `k_msgq_get()` when an actual **message** needs to move.

**Do not assume `k_poll_signal` is exactly equivalent to FreeRTOS task notifications.** Their state models, ownership, and supported patterns differ.

### 9.3 Simple Zephyr one-to-one notification using a semaphore

```c
K_SEM_DEFINE(worker_notify, 0, 1);

/* Notifying thread */
k_sem_give(&worker_notify);

/* Waiting thread */
k_sem_take(&worker_notify, K_FOREVER);
```

Think of this as a doorbell:

```text
Thread A                          Thread B
   |                                 |
   |                             k_sem_take()
   |                                 |
   |                           BLOCK / WAIT
   |
   +------ k_sem_give() ------------>|
                                     |
                                WAKE UP
```

A binary semaphore stores availability up to its configured limit; it doesn't store unlimited repeated rings. Use a counting semaphore or queue when every occurrence must be counted or delivered.

---

## 10. Complete QEMU experiment — all three ideas in one application

We'll demonstrate:

1. **Event flags:** main waits for SENSOR and NETWORK readiness.
2. **Notification:** worker gives a semaphore to wake main.
3. **Polling:** main watches a second semaphore **and** a poll signal.
4. **Joining:** main waits for the worker to terminate before printing the final line.

### 10.1 Our project directory (your `myapp/` convention)

**All learning code stays under `zephyr/myapp/`.** We won't edit Zephyr's original `kernel/` files or official samples.

```text
zephyrproject/
├── .venv/
├── zephyr/
│   ├── kernel/
│   ├── samples/
│   └── myapp/
│       └── lesson06_events_polling/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson06_events_polling/    <-- generated firmware
```

**Note:** `myapp/` lives inside the Zephyr Git checkout, so it may show up as untracked files. Keep it out of commits intended for upstream Zephyr.

### 10.2 Step 1 — Create the folder (PowerShell)

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject
.\.venv\Scripts\Activate.ps1
cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson06_events_polling\src | Out-Null
```

PowerShell uses `Activate.ps1`; `activate.bat` is for CMD.

### 10.3 Step 2 — Create `src/main.c`

Save the following code in:

`zephyrproject/zephyr/myapp/lesson06_events_polling/src/main.c`

```c
#include <zephyr/kernel.h>
#include <zephyr/sys/util.h>
#include <errno.h>
#include <stdbool.h>
#include <stdint.h>

/* Two independent conditions stored as event bits */
#define EVENT_SENSOR_READY  BIT(0)
#define EVENT_NETWORK_READY BIT(1)
#define ALL_READY (EVENT_SENSOR_READY | EVENT_NETWORK_READY)

#define WORKER_STACK_SIZE 1024
#define WORKER_PRIORITY   5

/* 1. Event flags: remember which things have become ready */
K_EVENT_DEFINE(system_events);

/* 2. Notification: one worker tells main that a job finished */
K_SEM_DEFINE(job_done_sem, 0, 1);

/* 3. Polling: wait for one of multiple kinds of signals */
K_SEM_DEFINE(poll_data_sem, 0, 1);
static struct k_poll_signal poll_finished_signal;

K_THREAD_STACK_DEFINE(worker_stack, WORKER_STACK_SIZE);
static struct k_thread worker_thread_data;

static void producer_thread(void *p1, void *p2, void *p3)
{
    ARG_UNUSED(p1);
    ARG_UNUSED(p2);
    ARG_UNUSED(p3);

    /* First flag */
    k_sleep(K_MSEC(250));
    printk("WORKER: Sensor is ready!\n");
    k_event_post(&system_events, EVENT_SENSOR_READY);

    /* Second flag */
    k_sleep(K_MSEC(250));
    printk("WORKER: Network is ready!\n");
    k_event_post(&system_events, EVENT_NETWORK_READY);

    /* Simple one-to-one notification */
    k_sleep(K_MSEC(250));
    printk("WORKER: Job completed; notifying main.\n");
    k_sem_give(&job_done_sem);

    /* First condition for the k_poll example */
    k_sleep(K_MSEC(250));
    printk("WORKER: Poll semaphore is now available.\n");
    k_sem_give(&poll_data_sem);

    /* Second condition for the k_poll example */
    k_sleep(K_MSEC(250));
    printk("WORKER: Raising poll signal with value 1234.\n");
    int rc = k_poll_signal_raise(&poll_finished_signal, 1234);
    if (rc != 0) {
        printk("WORKER: Signal raise failed (%d).\n", rc);
    }

    printk("WORKER: Finished.\n");
}

int main(void)
{
    printk("\n=== Lesson 06: Events, Polling, Notifications ===\n");

    /* Poll signal must be initialized BEFORE worker can raise it. */
    k_poll_signal_init(&poll_finished_signal);

    k_thread_create(&worker_thread_data,
                    worker_stack,
                    K_THREAD_STACK_SIZEOF(worker_stack),
                    producer_thread,
                    NULL, NULL, NULL,
                    K_PRIO_PREEMPT(WORKER_PRIORITY),
                    0,
                    K_NO_WAIT);

    /* ---------------------------------------------------------- */
    /* Part A: wait for ANY, then wait for ALL event flags          */
    /* ---------------------------------------------------------- */
    printk("MAIN: Waiting for sensor OR network...\n");

    uint32_t flags = k_event_wait(&system_events,
                                  ALL_READY,
                                  false,
                                  K_SECONDS(3));
    if (flags == 0U) {
        printk("MAIN: Event ANY timed out!\n");
        return 1;
    }

    printk("MAIN: At least one event! Flags = 0x%x\n", flags);

    printk("MAIN: Now waiting for sensor AND network...\n");
    flags = k_event_wait_all(&system_events,
                             ALL_READY,
                             false,
                             K_SECONDS(3));
    if ((flags & ALL_READY) != ALL_READY) {
        printk("MAIN: Event ALL timed out!\n");
        return 1;
    }

    printk("MAIN: Both are ready! Flags = 0x%x\n", flags);

    /* We don't need these startup flags anymore. */
    k_event_clear(&system_events, ALL_READY);

    /* ---------------------------------------------------------- */
    /* Part B: notification by semaphore                            */
    /* ---------------------------------------------------------- */
    printk("MAIN: Waiting for job-complete notification...\n");

    int rc = k_sem_take(&job_done_sem, K_SECONDS(3));
    if (rc != 0) {
        printk("MAIN: Notification wait failed (%d).\n", rc);
        return 1;
    }

    printk("MAIN: Notification received.\n");

    /* ---------------------------------------------------------- */
    /* Part C: k_poll watches TWO kernel-object conditions         */
    /* ---------------------------------------------------------- */
    struct k_poll_event watched[2];

    k_poll_event_init(&watched[0],
                      K_POLL_TYPE_SEM_AVAILABLE,
                      K_POLL_MODE_NOTIFY_ONLY,
                      &poll_data_sem);

    k_poll_event_init(&watched[1],
                      K_POLL_TYPE_SIGNAL,
                      K_POLL_MODE_NOTIFY_ONLY,
                      &poll_finished_signal);

    bool saw_sem = false;
    bool saw_signal = false;

    while (!saw_sem || !saw_signal) {
        printk("MAIN: k_poll() waiting for semaphore OR signal...\n");

        rc = k_poll(watched, ARRAY_SIZE(watched), K_SECONDS(3));
        if (rc != 0) {
            printk("MAIN: k_poll failed/timed out (%d).\n", rc);
            return 1;
        }

        if (watched[0].state == K_POLL_STATE_SEM_AVAILABLE) {
            /* Polling does not consume the semaphore! */
            if (k_sem_take(&poll_data_sem, K_NO_WAIT) == 0) {
                printk("MAIN: Polled semaphore; took its token.\n");
                saw_sem = true;
            }
        }

        if (watched[1].state == K_POLL_STATE_SIGNALED) {
            unsigned int signaled = 0U;
            int result = 0;

            k_poll_signal_check(&poll_finished_signal, &signaled, &result);
            if (signaled != 0U) {
                printk("MAIN: Poll signal received! Value = %d\n", result);
                saw_signal = true;
                k_poll_signal_reset(&poll_finished_signal);
            }
        }

        /* Clear the poll EVENT states before another k_poll(). */
        watched[0].state = K_POLL_STATE_NOT_READY;
        watched[1].state = K_POLL_STATE_NOT_READY;
    }

    /* The worker has returned, but joining confirms termination. */
    rc = k_thread_join(&worker_thread_data, K_SECONDS(3));
    if (rc != 0) {
        printk("MAIN: Worker join failed (%d).\n", rc);
        return 1;
    }

    printk("MAIN: Worker terminated. Lesson 06 complete!\n");
    return 0;
}
```

### 10.4 Step 3 — Create `prj.conf`

Save as `zephyr/myapp/lesson06_events_polling/prj.conf`:

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
CONFIG_EVENTS=y
CONFIG_POLL=y
```

| Setting | Why we need it |
|---|---|
| `CONFIG_MULTITHREADING=y` | Main and worker threads can run |
| `CONFIG_PRINTK=y` | Print to the QEMU console |
| `CONFIG_EVENTS=y` | Enable `k_event` objects |
| `CONFIG_POLL=y` | Enable `k_poll()` and poll signal APIs |

### 10.5 Step 4 — Create `CMakeLists.txt`

Save as `zephyr/myapp/lesson06_events_polling/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson06_events_polling)

target_sources(app PRIVATE src/main.c)
```

### 10.6 Step 5 — Build the application

Run from **`zephyrproject/zephyr`**:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson06_events_polling -d ..\build\lesson06_events_polling
```

Here is what each part means:

| Part | Purpose |
|---|---|
| `west build` | Build the firmware |
| `-p always` | Start with a clean build configuration |
| `-b qemu_x86` | Target virtual x86 hardware |
| `.\myapp\lesson06_events_polling` | **Our Lesson 06 source**, inside `zephyr/myapp/` |
| `-d ..\build\lesson06_events_polling` | Keep output in a separate Lesson 06 build directory |

### 10.7 Step 6 — Run in QEMU

```powershell
west build -d ..\build\lesson06_events_polling -t run
```

After it finishes, if QEMU remains open, press **Ctrl+A**, release, then press **X** to exit.

### 10.8 Example console output

The exact times and interleaving may differ, but the important messages should look like:

```text
*** Booting Zephyr OS build ... ***

=== Lesson 06: Events, Polling, Notifications ===
MAIN: Waiting for sensor OR network...
WORKER: Sensor is ready!
MAIN: At least one event! Flags = 0x1
MAIN: Now waiting for sensor AND network...
WORKER: Network is ready!
MAIN: Both are ready! Flags = 0x3
MAIN: Waiting for job-complete notification...
WORKER: Job completed; notifying main.
MAIN: Notification received.
MAIN: k_poll() waiting for semaphore OR signal...
WORKER: Poll semaphore is now available.
MAIN: Polled semaphore; took its token.
MAIN: k_poll() waiting for semaphore OR signal...
WORKER: Raising poll signal with value 1234.
WORKER: Finished.
MAIN: Poll signal received! Value = 1234
MAIN: Worker terminated. Lesson 06 complete!
```

**Why does this matter?** The main thread isn't running a giant `while(1)` loop constantly checking shared flags. It blocks while Zephyr waits for something useful to happen.

---

## 11. Understand the experiment line by line

### Part A: `k_event_wait()` / `k_event_wait_all()`

We want **two startup requirements**.

```text
At ~250 ms: SENSOR_READY posted
            |
            +--> ANY returns

At ~500 ms: NETWORK_READY posted
            |
            +--> ALL returns
```

We passed `reset=false`, so the sensor bit is still set when we start waiting for ALL. If we had cleared the event object before the second wait, the ALL wait could require a *new* sensor event that never arrives.

### Part B: `k_sem_take()` as a notification

```text
MAIN                         WORKER
 |                              |
 |---- k_sem_take()             |
 |     WAIT                     |
 |                              +---- finishes a job
 |<----------------------- k_sem_give()
 |
 +---- continues
```

A dedicated semaphore is a simple way to notify one waiting thread. It is **not** a payload transfer; for real data use a queue or carefully protected shared memory.

### Part C: `k_poll()`

```text
Poll round 1:
    Wait for SEM or SIGNAL
    SEM becomes available
    k_poll() returns
    Main uses k_sem_take()

Poll round 2:
    Re-arm watched states
    Wait for SEM or SIGNAL
    SIGNAL is raised
    k_poll() returns
    Main reads signal result (1234)
```

The `k_poll_signal_raise()` result is an integer attached to one signal state; it does not store a history of multiple signal values.

### Why `k_thread_join()` at the end?

You already learned this in Lesson 03:

```c
k_thread_join(&worker_thread_data, K_SECONDS(3));
```

It means: **"Wait for the worker thread to fully terminate."**

A semaphore or event can mean "work completed," but `k_thread_join()` specifically synchronizes with **thread termination**.

---

## 12. Macros vs initialization APIs (quick reference)

| Object | Define statically | Initialize existing object |
|---|---|---|
| Zephyr event | `K_EVENT_DEFINE()` | `k_event_init()` |
| Semaphore | `K_SEM_DEFINE()` | `k_sem_init()` |
| Mutex | `K_MUTEX_DEFINE()` | `k_mutex_init()` |
| Message queue | `K_MSGQ_DEFINE()` | `k_msgq_init()` |
| Poll signal | `struct k_poll_signal` + `k_poll_signal_init()` | `k_poll_signal_init()` |
| Poll event entry | `K_POLL_EVENT_STATIC_INITIALIZER()` | `k_poll_event_init()` |

**Key detail:** A plain `struct k_poll_signal` declaration is **not** the same as a fully initialized signal. Call `k_poll_signal_init()` (or use the documented static initializer pattern) before use.

---

## 13. Events vs semaphores vs message queues vs poll signals

| What you need | Best starting choice | Simple meaning |
|---|---|---|
| One thread wakes another | Semaphore | "Doorbell!" |
| Wait until several conditions are ready | Event flags | "All the checkboxes are ticked." |
| Carry structured data from one thread to another | Message queue | "Put a letter in a mailbox." |
| Protect shared data from simultaneous updates | Mutex | "Only one person holds the key." |
| Wait for several supported kernel-object conditions | `k_poll()` | "Tell me when ANY of these becomes ready." |
| Send a result-bearing signal observed through `k_poll()` | Poll signal | "A special bell with a small number." |
| Confirm a thread has finished executing | `k_thread_join()` | "Wait until the worker has gone home." |

**Remember:**

- **Event flag:** remembers *a condition bit*, not a count of how many times it happened.
- **Semaphore:** can count permits up to its configured limit.
- **Message queue:** stores actual messages.
- **Poll signal:** stays in a signaled state until reset; it isn't a general-purpose message FIFO.
- **Mutex:** used for ownership of a critical section, not for notifying another thread.

---

## 14. Zephyr vs FreeRTOS vs ThreadX

### 14.1 Side-by-side API comparison

| Feature | FreeRTOS | Eclipse ThreadX | Zephyr |
|---|---|---|---|
| Group of event bits | Event Groups | Event Flags Groups | `k_event` |
| Create event flags | `xEventGroupCreate()` / static variant | `tx_event_flags_create()` | `K_EVENT_DEFINE()` / `k_event_init()` |
| Set/add event flags | `xEventGroupSetBits()` | `tx_event_flags_set(..., TX_OR)` | `k_event_post()` |
| Wait for ANY | `xEventGroupWaitBits(..., pdFALSE, ...)` | `tx_event_flags_get(..., TX_OR, ...)` | `k_event_wait()` |
| Wait for ALL | `xEventGroupWaitBits(..., pdTRUE, ...)` | `tx_event_flags_get(..., TX_AND, ...)` | `k_event_wait_all()` |
| Single-thread notification | Direct task notifications | Commonly semaphores or event flags | Commonly semaphores, events, or poll signals |
| Dedicated task-notification API | `xTaskNotifyGive()` / `ulTaskNotifyTake()` and related calls | No directly matching FreeRTOS-style task-notification API | No directly matching FreeRTOS-style task-notification API |
| Wait on several types of kernel objects together | No exact `k_poll()` equivalent in basic task API; queue sets cover certain queues/semaphores | No direct equivalent of the same general API | `k_poll()` |
| Signal usable with polling | Depends on chosen FreeRTOS object/pattern | Event flags/semaphore patterns | `k_poll_signal_raise()` |
| Wait for thread termination | No standard `join` API | No standard `join` API | `k_thread_join()` |

**Important caveat:** FreeRTOS queue sets, ThreadX event flags, and Zephyr `k_poll()` solve some similar problems, but they are **not interchangeable** and have different supported object types and behavior.

### 14.2 FreeRTOS event-group idea

**Illustrative FreeRTOS snippet:**

```c
EventBits_t bits = xEventGroupWaitBits(
    event_group,
    SENSOR_BIT | NETWORK_BIT,
    pdFALSE,      /* Don't clear on exit */
    pdTRUE,       /* Wait for ALL bits */
    portMAX_DELAY
);
```

In FreeRTOS, the `pdTRUE` parameter shown above for `xWaitForAllBits` means wait for **all** requested bits.

FreeRTOS also has direct notifications for a specific task:

```c
/* Task A (sender) */
xTaskNotifyGive(task_b_handle);

/* Task B (receiver) */
ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
```

The meaning of notification counts/values depends on the chosen FreeRTOS notification API and action.

### 14.3 ThreadX event-flags idea

**Illustrative ThreadX snippet:**

```c
ULONG actual_flags;

/* Sender: add one flag */
tx_event_flags_set(&flags_group, SENSOR_BIT, TX_OR);

/* Receiver: wait for BOTH flags */
tx_event_flags_get(&flags_group,
                   SENSOR_BIT | NETWORK_BIT,
                   TX_AND,
                   &actual_flags,
                   TX_WAIT_FOREVER);
```

With `TX_AND_CLEAR` or `TX_OR_CLEAR`, ThreadX can automatically clear matching flags after a successful get. Be careful with shared event groups, just as in Zephyr.

### 14.4 Zephyr version of the same idea

```c
/* Sender: add a bit */
k_event_post(&system_events, EVENT_SENSOR_READY);

/* Receiver: wait for BOTH bits */
k_event_wait_all(&system_events,
                 EVENT_SENSOR_READY | EVENT_NETWORK_READY,
                 false,
                 K_FOREVER);
```

### 14.5 Easy way to remember

```text
FreeRTOS:
  "Event Groups for many bits;
   direct task notifications for a specific task."

ThreadX:
  "Event Flags Groups for bits;
   semaphores and event flags for coordination."

Zephyr:
  "k_event for bits;
   k_sem for a simple wake-up;
   k_poll for waiting on several supported objects."
```

---

## 15. Timeouts, return values, and possible mistakes

### 15.1 Event wait timeout

```c
uint32_t bits = k_event_wait(&system_events, EVENT_SENSOR_READY,
                             false, K_MSEC(500));

if (bits == 0U) {
    printk("No sensor event within 500 ms\n");
}
```

`k_event_wait()` returns a **bit mask**, not a typical zero-or-error integer.

### 15.2 Poll timeout

```c
int rc = k_poll(watched, 2, K_MSEC(500));
if (rc == -EAGAIN) {
    printk("Neither condition became ready within 500 ms\n");
}
```

`k_poll()` returns an **integer status**, so check it before trusting poll-event states.

### 15.3 Semaphore wait timeout

```c
int rc = k_sem_take(&job_done_sem, K_MSEC(500));
if (rc != 0) {
    printk("No notification yet\n");
}
```

### 15.4 Five common beginner bugs

1. **Thinking `k_event_wait()` clears bits on success.** It doesn't by default.
2. **Using `reset=true` and accidentally deleting an event that arrived earlier.**
3. **Forgetting `CONFIG_EVENTS=y` or `CONFIG_POLL=y`.** The build may fail or the API may be unavailable.
4. **Using `k_poll()` but forgetting `k_sem_take()` afterward.** Polling notices availability; it doesn't consume the token.
5. **Reusing `k_poll()` without resetting its entry states and a handled signal.** You may see an immediate wake-up based on stale state.

### 15.5 ISR rule

- You can **post/set events** from an ISR in Zephyr.
- You can **give** a semaphore from an ISR.
- A blocking event wait or semaphore take is **not** suitable inside an ISR.
- `k_poll()` is a **thread-side wait**, not an ISR-side blocking operation.

Keep interrupt handlers short. Signal a thread, then let that thread do larger work.

---

## 16. Where is the implementation in Zephyr source code?

Explore these files in your existing Zephyr checkout:

| File | What you'll find |
|---|---|
| `include/zephyr/kernel.h` | Public declarations and many kernel object macros |
| `kernel/events.c` | Event flag implementation |
| `kernel/poll.c` | `k_poll()` and poll-signal implementation |
| `kernel/sem.c` | Semaphore give/take implementation |
| `kernel/sched.c` | Scheduler behavior when threads block/wake |
| `kernel/thread.c` | Thread management |
| `kernel/Kconfig` | Kernel feature switches including `CONFIG_POLL` and `CONFIG_EVENTS` |

Try these PowerShell commands while in `zephyrproject/zephyr`:

```powershell
code .\kernel\events.c
code .\kernel\poll.c
code .\kernel\sem.c
```

If you want to search for the important API implementations:

```powershell
Select-String -Path .\kernel\events.c -Pattern 'k_event_post','k_event_wait'
Select-String -Path .\kernel\poll.c -Pattern 'k_poll_signal_raise','k_poll'
Select-String -Path .\kernel\sem.c -Pattern 'k_sem_give','k_sem_take'
```

The names and precise locations of internal helpers may vary by Zephyr version. Start from the public API and follow the implementation for the **version actually checked out in your workspace**.

### What the scheduler does

```text
Main calls k_event_wait()
        |
        v
Main cannot continue yet
        |
        v
Zephyr BLOCKS main
        |
        v
Another ready thread executes
        |
        v
Worker calls k_event_post()
        |
        v
Wait condition is satisfied
        |
        v
Main becomes READY
        |
        v
Scheduler eventually runs main
```

This is why RTOS synchronization is efficient: the waiting thread gives the CPU to other useful work.

---

## 17. Practical exercises (change one thing at a time)

### Exercise A — Swap ANY for ALL

Change:

```c
k_event_wait(&system_events, ALL_READY, false, K_SECONDS(3));
```

to:

```c
k_event_wait_all(&system_events, ALL_READY, false, K_SECONDS(3));
```

**Question:** Does the first message now wait until the second worker event?

### Exercise B — Make the event wait timeout

Change the first wait to `K_MSEC(100)` while the worker waits 250 ms before posting its first event.

**Question:** What does the return value mean when it is zero?

### Exercise C — Understand `reset=true`

In the *second* event wait (ALL), change `false` to `true`.

**Question:** Why might the first sensor-ready bit be lost, causing the wait to time out?

Then restore `false`.

### Exercise D — Inspect poll results

Add prints to display:

```c
printk("poll states: sem=%u signal=%u\n",
       watched[0].state,
       watched[1].state);
```

**Question:** Which condition wakes `k_poll()` first?

### Exercise E — Send a different signal value

Change:

```c
k_poll_signal_raise(&poll_finished_signal, 1234);
```

to:

```c
k_poll_signal_raise(&poll_finished_signal, 99);
```

**Question:** Where does the receiver get the number `99`?

### Exercise F — Look at the build configuration

After building:

```powershell
Select-String -Path ..\build\lesson06_events_polling\zephyr\.config -Pattern 'CONFIG_EVENTS=','CONFIG_POLL='
```

Expected settings include `CONFIG_EVENTS=y` and `CONFIG_POLL=y`.

---

## 18. Troubleshooting on your Windows + QEMU setup

### Problem: "Lesson 05 is still running when I try Lesson 06"

You saw a similar issue when moving from Lesson 03 to Lesson 04.

**Use a pristine build and explicitly point to the correct Lesson 06 source folder:**

```powershell
west build -p always -b qemu_x86 .\myapp\lesson06_events_polling -d ..\build\lesson06_events_polling
west build -d ..\build\lesson06_events_polling -t run
```

Check the application source directory recorded in the cache:

```powershell
Select-String -Path ..\build\lesson06_events_polling\CMakeCache.txt -Pattern 'APPLICATION_SOURCE_DIR','CMAKE_HOME_DIRECTORY'
```

It should correspond to:

`C:/Users/ragha/workspace/personal/zephyrproject/zephyr/myapp/lesson06_events_polling`

### Problem: `CONFIG_EVENTS` or `CONFIG_POLL` seems missing

Verify `prj.conf` is in the **application root**, next to `CMakeLists.txt`, and rebuild with `-p always`.

### Problem: QEMU fails before printing Zephyr output

That is usually a QEMU/runtime environment problem, not a semaphore or event bug. Your earlier Windows QEMU dependency fix may need to be applied to the current terminal session.

### Problem: Different message order

Thread scheduling, QEMU timing, and console output can affect exact interleaving. Focus on **which conditions must happen before others**, not exact millisecond times.

---

## 19. Mini quiz

Try answering before checking the solutions:

1. Does setting the same event bit five times mean five messages are waiting?
2. What is the difference between `k_event_wait()` and `k_event_wait_all()`?
3. Does `k_poll()` use a busy loop?
4. Does `k_poll()` automatically take a semaphore?
5. Is a Zephyr `k_poll_signal` identical to a FreeRTOS direct task notification?
6. When is a message queue better than an event flag?
7. Why must you reset the poll event state before polling again?

**Answers:**

1. No: the bit is either set or clear.
2. ANY requested bits versus ALL requested bits.
3. No: it blocks the thread until a supported condition occurs or the wait times out.
4. No: use `k_sem_take()` afterward and check success.
5. No: they are different APIs with different semantics.
6. When actual data/payloads must be delivered.
7. To avoid treating a previously handled readiness condition as new.

---

## 20. Summary — what to remember

```text
                         ZEPHYR SYNCHRONIZATION
                                   |
              +--------------------+--------------------+
              |                    |                    |
           k_event               k_sem                k_poll
              |                    |                    |
       Event flag bits        Signal/token         Wait for multiple
       ANY / ALL waits        one thread           supported conditions
              |                    |                    |
       k_event_post()        k_sem_give()          Semaphore available?
       k_event_wait()        k_sem_take()          Poll signal raised?
       k_event_wait_all()                          Queue has data?
```

**One-sentence memory trick:**

- **Events:** "Which conditions happened?"
- **Semaphores:** "You may continue now."
- **Queues:** "Here is your data."
- **Polling:** "Wake me when any of these supported things is ready."
- **Thread joining:** "Tell me when the worker has completely terminated."

### Build/run quick reference

From `C:\Users\ragha\workspace\personal\zephyrproject\zephyr`:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson06_events_polling -d ..\build\lesson06_events_polling
west build -d ..\build\lesson06_events_polling -t run
```

---

## 21. Sources and further reading

These are authoritative references for the concepts and APIs used in this lesson:

- [Zephyr — Events](https://docs.zephyrproject.org/latest/kernel/services/synchronization/events.html)
- [Zephyr — Event API reference](https://docs.zephyrproject.org/latest/doxygen/html/group__event__apis.html)
- [Zephyr — Polling API](https://docs.zephyrproject.org/latest/kernel/services/polling.html)
- [Zephyr — Poll API reference](https://docs.zephyrproject.org/latest/doxygen/html/group__poll__apis.html)
- [Zephyr — Thread lifecycle and joining](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html)
- [Eclipse ThreadX — Event flags](https://github.com/eclipse-threadx/rtos-docs/blob/main/rtos-docs/threadx/chapter3.md)

Documentation marked `latest` can change after your installed version (`v4.4.0-16104-g64437be51c30`). If something differs, inspect the headers and kernel source in your own Zephyr checkout first.

**Next possible lesson:** `07_Timers_Delayed_Work_and_Work_Queues.md` — timers, callbacks, and handing work from interrupts to threads.
