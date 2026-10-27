# 09 — Zephyr Interrupts, ISRs, and Deferred Processing

> **Beginner-friendly Zephyr RTOS series**  
> Target: **Windows PowerShell + Zephyr 4.4-based source tree + QEMU `qemu_x86`**  
> Source code location: **`zephyrproject/zephyr/myapp/lesson09_interrupts/`**  
> Build files location: **`zephyrproject/build/lesson09_interrupts/`**

---

## 1. What will we learn?

By the end of this lesson, you should understand:

1. Why interrupts exist, and why an interrupt is different from a normal thread.
2. What **ISR** (Interrupt Service Routine) means.
3. Why long-running code inside an ISR is a problem.
4. What **deferred processing** (moving work from an ISR into a thread) means.
5. How Zephyr uses semaphores and workqueues to hand work to threads.
6. How to test interrupt-context behavior on **QEMU**, without any physical board.
7. How Zephyr, FreeRTOS, and ThreadX approach ISR-related work.
8. Which Zephyr source files and APIs to study next.

**Big idea:** *An interrupt says, “Something urgent happened!” The ISR notices it quickly. A thread does the heavier work afterward.*

---

## 2. What is an interrupt? (Explain it like I'm 10)

Imagine you are doing homework. Suddenly, the doorbell rings.

- **Without interrupts:** You keep walking to the door every few seconds to ask, “Did anyone ring?” That wastes your time.
- **With interrupts:** You continue doing homework. When someone rings the bell, you pause, see who arrived, and then return to homework.

In an embedded system:

- A **UART** might interrupt when a byte arrives.
- A **timer** might interrupt when its countdown expires.
- A **GPIO** might interrupt when a button is pressed.
- A **DMA controller** might interrupt after a transfer completes.

```text
CPU running an ordinary thread
           |
           |  Hardware event happens
           v
      INTERRUPT!
           |
           v
    ISR runs briefly
           |
           v
  Return to thread scheduling
```

An interrupt does **not** mean a new thread was created. An ISR runs in a special interrupt execution context; a thread is scheduled by the RTOS.

---

## 3. ISR versus thread

| Question | Interrupt service routine (ISR) | Thread |
|---|---|---|
| Who starts it? | A hardware/interrupt event | Zephyr scheduler |
| Typical work | Acknowledge hardware, capture a small amount of data, signal a thread | Process data, format logs, run application logic |
| Can it use `k_sleep()`? | **No** | **Yes** |
| Can it block on `k_sem_take(..., K_FOREVER)`? | **No** | **Yes** |
| Can it signal a semaphore? | **Yes**, `k_sem_give()` | Yes |
| Can it submit work to a workqueue? | **Yes**, `k_work_submit()` | Yes |
| Can it be preempted? | Depends on interrupt priorities and architecture | Depends on thread scheduling |

**Easy rule: Never call a blocking/sleeping API in an ISR.** An ISR must finish promptly so that the system can handle other urgent events.

> Some APIs permit ISR use only with `K_NO_WAIT`, and some are forbidden altogether. Always read that API's documentation; do not assume every `k_...` function is ISR-safe.

---

## 4. Why should ISRs be short?

Suppose a UART receives data every millisecond. But its ISR spends 20 milliseconds printing messages or processing a large packet.

While that ISR is busy, other interrupts can be delayed (depending on priority/masking), and scheduling latency can increase. You might lose data or miss deadlines.

**Bad design:**

```text
UART interrupt
    |
    +-- Read received byte
    +-- Parse whole protocol
    +-- Print a giant report
    +-- Perform long calculations
    +-- Return much later
```

**Better design:**

```text
UART interrupt
    |
    +-- Read/store urgent data
    +-- Signal a thread or submit work
    +-- Return quickly
                    |
                    v
            Worker thread
                    |
                    +-- Parse data
                    +-- Process message
                    +-- Print results
```

This is called **deferred processing** or **interrupt offloading**.

---

## 5. Two common Zephyr designs

### Design A — ISR signals a dedicated thread

```text
Interrupt -> ISR -> k_sem_give() -> Worker wakes -> Handles data
```

Use this when your application has a dedicated worker or when you must **count individual events**.

```c
/* ISR context */
k_sem_give(&my_sem);

/* Thread context */
k_sem_take(&my_sem, K_FOREVER);
/* Process the event */
```

A counting semaphore can retain multiple outstanding signals up to its configured limit. It does not carry event payloads. For payloads, consider a queue or other buffer.

### Design B — ISR submits work to a workqueue

```text
Interrupt -> ISR -> k_work_submit() -> Workqueue thread -> Handler
```

Use this for short, non-urgent deferred jobs when a dedicated thread would be unnecessary.

```c
/* ISR context */
k_work_submit(&my_work);

/* System workqueue thread context */
void my_work_handler(struct k_work *work)
{
    /* Process outside interrupt context */
}
```

**Important:** Submitting the *same work item* several times while it is already queued **does not guarantee one separate handler run per event**. Work items are not counting queues. Use a semaphore/message queue/counter to preserve events if every occurrence matters.

---

## 6. Practical QEMU experiment: A timer callback behaves like an ISR

Our QEMU board does not provide a physical button or UART connection for this tutorial. Instead, we use a **Zephyr software timer whose expiry callback executes in the system clock interrupt handler**.

This is a genuine interrupt-context demonstration, **not** a pretend interrupt triggered from `main()`.

### Experiment plan

1. `main()` starts a periodic timer.
2. When it expires, the expiry callback runs in interrupt context.
3. The callback performs **only fast operations**: it counts the event, signals a semaphore, and submits a work item.
4. The system workqueue thread prints a message outside ISR context.
5. `main()` receives five semaphore notifications, stops the timer, and waits for the pending work item to finish.

```text
                   Periodic timer expires
                              |
                              v
                      TIMER EXPIRY ISR
                       /           \
                      /             \
                 k_sem_give()   k_work_submit()
                    |                |
                    v                v
               main() wakes     System workqueue
                    |                |
              Counts event       Work handler prints
```

### Project directory

```text
zephyrproject/
├── zephyr/
│   ├── kernel/                      # Zephyr source (don't edit)
│   └── myapp/
│       └── lesson09_interrupts/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson09_interrupts/         # Auto-generated
```

### Step 1 — Create the directory

In **Windows PowerShell**:

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject
.\.venv\Scripts\Activate.ps1
cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson09_interrupts\src | Out-Null
```

### Step 2 — Create `src/main.c`

Save as:

`zephyr/myapp/lesson09_interrupts/src/main.c`

```c
#include <zephyr/kernel.h>
#include <zephyr/sys/atomic.h>
#include <errno.h>

#define TOTAL_EVENTS 5

/* One event signal per timer expiry, up to 10 queued signals. */
K_SEM_DEFINE(event_sem, 0, 10);

static atomic_t expiry_count;
static atomic_t work_count;

/* This function runs in the system workqueue THREAD, not in an ISR. */
static void deferred_handler(struct k_work *work)
{
    ARG_UNUSED(work);

    int handled = atomic_inc(&work_count) + 1;

    printk("[WORKQUEUE] handler call %d | in ISR? %s\n",
           handled,
           k_is_in_isr() ? "YES" : "NO");
}

K_WORK_DEFINE(deferred_work, deferred_handler);

/* Timer expiry callbacks run in interrupt context. */
static void timer_expiry_handler(struct k_timer *timer)
{
    ARG_UNUSED(timer);

    atomic_inc(&expiry_count);

    /* ISR-safe: signal the main thread. */
    k_sem_give(&event_sem);

    /* ISR-safe: ask the system workqueue thread to do work later. */
    (void)k_work_submit(&deferred_work);

    /* No k_sleep(), blocking wait, or long processing here. */
}

K_TIMER_DEFINE(event_timer, timer_expiry_handler, NULL);

int main(void)
{
    printk("\n=== Lesson 09: Interrupts and Deferred Processing ===\n");
    printk("Starting periodic timer; five events expected.\n");

    atomic_set(&expiry_count, 0);
    atomic_set(&work_count, 0);

    /* First expiry after 200 ms; subsequent expiries 500 ms apart. */
    k_timer_start(&event_timer, K_MSEC(200), K_MSEC(500));

    for (int i = 1; i <= TOTAL_EVENTS; i++) {
        int ret = k_sem_take(&event_sem, K_SECONDS(5));

        if (ret != 0) {
            printk("[MAIN] Timed out waiting for event %d: %d\n",
                   i, ret);
            break;
        }

        printk("[MAIN] received timer event %d | in ISR? %s\n",
               i,
               k_is_in_isr() ? "YES" : "NO");
    }

    /* Stop future timer expiries before returning from main(). */
    k_timer_stop(&event_timer);

    /* Wait for the last submitted work to finish, if still active. */
    static struct k_work_sync sync;
    (void)k_work_flush(&deferred_work, &sync);

    printk("Timer expiries: %d\n", atomic_get(&expiry_count));
    printk("Work handler calls: %d\n", atomic_get(&work_count));
    printk("Lesson 09 finished.\n");

    return 0;
}
```

**What to notice in this code:**

- `K_TIMER_DEFINE()` creates the timer statically.
- `K_SEM_DEFINE()` creates a counting semaphore statically.
- `K_WORK_DEFINE()` creates a work item statically.
- `k_sem_give()` signals that an event happened.
- `k_work_submit()` requests deferred work in a **thread**.
- `k_is_in_isr()` helps demonstrate the difference between interrupt and thread contexts.
- `atomic_inc()` lets the example update its counters from different contexts without an ordinary data race.
- `k_work_flush()` lets the main thread wait for outstanding work. Do **not** call `k_work_flush()` from the timer ISR.

> **Note:** The number of work handler calls need not equal the number of timer expiries in every schedule. Repeated submissions of a queued work item can be coalesced. The semaphore is what counts the timer events in this example.

### Step 3 — Create `prj.conf`

Save as `zephyr/myapp/lesson09_interrupts/prj.conf`:

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=1024
```

`CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE` reserves a modest amount of stack for the shared workqueue thread. If your build already has an adequate default stack, it may not be necessary to override it.

### Step 4 — Create `CMakeLists.txt`

Save as `zephyr/myapp/lesson09_interrupts/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson09_interrupts)

target_sources(app PRIVATE src/main.c)
```

### Step 5 — Build (pristine)

Run **from** `zephyrproject/zephyr/`:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson09_interrupts -d ..\build\lesson09_interrupts
```

### Step 6 — Run on QEMU

```powershell
west build -d ..\build\lesson09_interrupts -t run
```

### Expected output (illustrative)

```text
*** Booting Zephyr OS build ... ***

=== Lesson 09: Interrupts and Deferred Processing ===
Starting periodic timer; five events expected.
[MAIN] received timer event 1 | in ISR? NO
[WORKQUEUE] handler call 1 | in ISR? NO
[MAIN] received timer event 2 | in ISR? NO
[WORKQUEUE] handler call 2 | in ISR? NO
...
Timer expiries: 5
Work handler calls: 5
Lesson 09 finished.
```

**The order can vary**, and the number of work handler calls can be lower than the number of timer expiries because work submissions can be merged while pending. The key observation is: **the work handler and `main()` are not executing in ISR context**.

To exit QEMU, press **Ctrl+A**, release, then press **X**, if QEMU remains running. A completed sample may cause the runner to exit on its own.

---

## 7. What if we put `k_sleep()` inside the timer callback?

**Do not do this:**

```c
static void timer_expiry_handler(struct k_timer *timer)
{
    /* WRONG: Timer expiry callback executes in interrupt context. */
    k_sleep(K_SECONDS(1));
}
```

Why? `k_sleep()` must suspend a **thread**, but the timer expiry callback is running inside an **interrupt handler**, not a schedulable thread. That breaks the rule against blocking in ISRs.

Correct approach: signal a thread or submit work; the thread can sleep if its design requires it.

---

## 8. What about an actual GPIO/UART interrupt?

On real hardware, a driver can register a device ISR associated with a physical IRQ. The basic idea looks like this:

```text
Physical peripheral raises IRQ
             |
             v
Hardware interrupt controller
             |
             v
   Zephyr interrupt handler
             |
             v
Clear/acknowledge device condition
             |
             v
Signal semaphore or submit work
             |
             v
       Worker thread
```

Zephyr has mechanisms such as `IRQ_CONNECT()` and architecture-specific interrupt controller support. A *real* IRQ example needs a known device, interrupt number, priority, and board configuration. We are **not** inventing these for QEMU.

For most application development, use the appropriate **driver API**, such as GPIO callback registration or UART asynchronous callbacks, instead of modifying low-level interrupt vectors manually.

---

## 9. `irq_lock()` is not a mutex

A mutex protects shared data between threads and may cause a thread to wait.

`irq_lock()` is different: on the **current CPU**, it masks regular interrupts as defined by the architecture, until the interrupt-lock state is restored. It does **not** serve as a general cross-CPU synchronization mechanism, and it does not block hardware events from occurring.

```c
unsigned int key = irq_lock();
/* Extremely short critical section; do not sleep here. */
irq_unlock(key);
```

**Beginner guidance:** Prefer mutexes for ordinary thread-to-thread shared state, atomic operations for simple counters/flags where appropriate, and IRQ locking only when genuinely necessary. Never hold interrupts masked during lengthy calculations or printing.

---

## 10. Zephyr vs FreeRTOS vs ThreadX — Interrupt handling

| Feature | Zephyr | FreeRTOS | ThreadX |
|---|---|---|---|
| Interrupt handling | Architecture/driver ISR registration, including `IRQ_CONNECT()` | MCU vector table and port/HAL integration | MCU vector table and ThreadX port integration |
| Signal a waiting thread | `k_sem_give()` (ISR-safe) | `xSemaphoreGiveFromISR()` | `tx_semaphore_put()` is usable from ISR where permitted by API rules |
| Queue data from ISR | `k_msgq_put(..., K_NO_WAIT)` | `xQueueSendFromISR()` | Check permitted ThreadX service and ISR restrictions; use an ISR-safe handoff |
| Defer work to a worker | `k_work_submit()` | Typically wake/notify a task (`vTaskNotifyGiveFromISR()` etc.) | Typically signal a thread using a semaphore/event flags |
| ISR blocking/sleep | Not allowed | Not allowed | Not allowed |
| ISR-specific API naming | Often same API, with context restrictions | Many APIs use `...FromISR()` | Service-specific rules (not always separate ISR suffix) |
| Generic application ISR rule | Keep short; defer heavy work | Keep short; defer heavy work | Keep short; defer heavy work |

**Important:** Not every FreeRTOS/ThreadX API is legal inside an ISR. FreeRTOS also imposes interrupt-priority rules for calling `...FromISR()` APIs, depending on MCU port/configuration. ThreadX has its own documented permitted-services list and port-specific constraints.

### Small examples (conceptual)

**Zephyr:**

```c
/* In ISR context */
k_sem_give(&ready_sem);
```

**FreeRTOS:**

```c
/* In ISR context; task-woken flag may request a context switch. */
BaseType_t higher_woken = pdFALSE;
xSemaphoreGiveFromISR(ready_sem, &higher_woken);
portYIELD_FROM_ISR(higher_woken); /* Port-specific macro */
```

**ThreadX:**

```c
/* In ISR context, when permitted by the ThreadX API/port. */
tx_semaphore_put(&ready_sem);
```

The syntax differs, but the idea is the same: **notify a waiting thread, then leave the ISR quickly**.

---

## 11. Common mistakes

| Mistake | Why it is a problem | Better approach |
|---|---|---|
| `k_sleep()` in timer expiry callback | Timer expiry callback runs in ISR context | Signal a thread or submit work |
| Waiting forever on a semaphore from ISR | ISRs cannot block | Use `k_sem_give()` in ISR, `k_sem_take()` in thread |
| Parsing large messages inside ISR | Increases interrupt latency | Push data into a buffer/queue and defer processing |
| Assuming one `k_work_submit()` = one run | Pending submissions can coalesce | Use a semaphore, queue, or counter to retain occurrences |
| Heavy processing in shared workqueue | Delays other workqueue jobs | Use a dedicated worker thread/workqueue |
| Holding `irq_lock()` too long | Increases interrupt latency | Keep critical sections extremely short |
| Reusing lesson 08 build directory | Runs the wrong executable | Use `-p always` and a separate lesson 09 build folder |

---

## 12. Explore the Zephyr source code

From your PowerShell terminal in `zephyrproject/zephyr`:

```powershell
# Find timer-related sources and structures
Get-ChildItem .\kernel -Filter '*timer*'

# Explore interrupt headers and architecture code
Get-ChildItem .\include\zephyr\irq* -ErrorAction SilentlyContinue
Get-ChildItem .\arch\x86

# Search ISR-related APIs in headers
Get-ChildItem .\include\zephyr -Recurse -Filter '*.h' |
    Select-String -Pattern 'IRQ_CONNECT\(' -List |
    Select-Object -First 10 Path, LineNumber

# Find workqueue implementation
Get-ChildItem .\kernel -Filter '*work*'
```

Key files/directories to investigate:

- **`include/zephyr/kernel.h`** — Public RTOS APIs and included kernel declarations.
- **`include/zephyr/irq.h`** — Interrupt-related API declarations (as present in your checkout).
- **`kernel/timeout.c`**, **`kernel/timer.c`** — Timeouts and timer handling.
- **`kernel/work.c`** — Workqueue implementation.
- **`arch/x86/`** — Architecture-specific CPU and interrupt support.
- **`drivers/`** — Actual device drivers and their ISR integration.

These help you move from the API usage to the kernel internals.

---

## 13. Try these exercises

### Exercise A — Change the timer period

Change:

```c
k_timer_start(&event_timer, K_MSEC(200), K_MSEC(500));
```

to:

```c
k_timer_start(&event_timer, K_MSEC(100), K_MSEC(1000));
```

Observe how event spacing changes. Timing is approximate in QEMU; it isn't a hard real-time measurement.

### Exercise B — Count faster interrupts

Reduce the period to `K_MSEC(10)` and increase `TOTAL_EVENTS` to 20. Watch whether work handler calls always match timer expiries. Explain why they might not.

### Exercise C — Compare semaphore and workqueue

Answer: If you **must** handle every incoming UART byte, is repeatedly submitting **one** work item enough? Why might a bounded message queue or ring buffer be more suitable?

### Exercise D — Explain ISR context

Why does `k_is_in_isr()` print `NO` inside our work handler even though the work was submitted by a timer interrupt?

---

## 14. Mini quiz

1. What does ISR stand for?
2. Can an ISR call `k_sleep()`?
3. Which API can signal a semaphore from Zephyr ISR context?
4. Which API submits a work item to the system workqueue?
5. Does a workqueue handler execute in a thread context or ISR context?
6. Will submitting the same pending work item five times guarantee five executions?
7. What is the difference between `irq_lock()` and `k_mutex_lock()`?
8. What is the FreeRTOS equivalent of an ISR-safe semaphore give?
9. Why should a UART receive interrupt handler be short?
10. Why is a timer expiry callback suitable for our QEMU demonstration?

<details>
<summary>Answers</summary>

1. Interrupt Service Routine.
2. No.
3. `k_sem_give()`.
4. `k_work_submit()`.
5. Thread context.
6. No; submissions while pending can coalesce.
7. IRQ masking on the current CPU versus thread-oriented mutual exclusion.
8. `xSemaphoreGiveFromISR()`.
9. To reduce latency and avoid delaying other important work.
10. Zephyr invokes expiry callbacks in the system clock interrupt handler, so it demonstrates ISR context without needing external hardware.

</details>

---

## 15. When do the beginner RTOS topics finish? Learning roadmap

You have completed the reading path through **Lesson 08**. With this lesson, you reach **Lesson 09**.

| Lesson | Topic | Phase |
|---|---|---|
| 01 | Introduction to Zephyr | Core fundamentals |
| 02 | Boot process and `main()` | Core fundamentals |
| 03 | Threads, priorities, and scheduling | Core fundamentals |
| 04 | Semaphores, mutexes, and race conditions | Core fundamentals |
| 05 | Message queues and inter-thread communication | Core fundamentals |
| 06 | Events, polling, and notifications | Core fundamentals |
| 07 | Timers, delayed work, and workqueues | Core fundamentals |
| 08 | Stacks, heaps, and memory slabs | Core fundamentals |
| **09** | **Interrupts, ISRs, and deferred processing** | **Core fundamentals** |
| **10** | **Kconfig, Devicetree, and build system** | **Core fundamentals finish** |
| 11 | Device model and GPIO driver | Hardware/application development |
| 12 | UART, SPI, and I²C drivers | Hardware/application development |
| 13 | Debugging, logging, and fault handling | Debugging |
| 14 | Custom board and STM32 bring-up | Hardware integration |
| 15 | Final small embedded project | Putting everything together |

**This is our suggested learning plan, not an official Zephyr certification curriculum.** There is no fixed end to Zephyr: the kernel basics are a milestone, while networking, Bluetooth, storage, security, USB, power management, testing, and upstream contribution can be learned later depending on your goals.

### What comes next?

**Lesson 10 — Kconfig, Devicetree, and Zephyr Build System.** It explains how Zephyr knows which code, drivers, and hardware to include in the firmware. After Lesson 10, we'll switch from mostly RTOS/kernel features to hardware drivers and real embedded board development.

---

## 16. Official references

1. [Zephyr: Interrupts](https://docs.zephyrproject.org/latest/kernel/services/interrupts.html)
2. [Zephyr: Workqueue Threads](https://docs.zephyrproject.org/latest/kernel/services/threads/workqueue.html)
3. [Zephyr: Timers](https://docs.zephyrproject.org/latest/kernel/services/timing/timers.html)
4. [Zephyr: Kernel Services](https://docs.zephyrproject.org/latest/kernel/services/index.html)

---

**Remember:** ISR = *notice urgently*. Worker thread = *do the longer job safely*. Keeping those responsibilities separate is one of the most useful embedded RTOS design habits.
