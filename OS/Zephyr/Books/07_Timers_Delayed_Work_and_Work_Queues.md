# 07 — Zephyr Timers, Delayed Work, and Work Queues

> **Beginner-friendly Zephyr RTOS lesson**  
> Target: **Windows PowerShell + Zephyr 4.4 development tree + QEMU x86**  
> Learning application: **`zephyr/myapp/lesson07_timers_workqueues/`**  
> We will also compare **FreeRTOS, ThreadX, and Zephyr**.

---

## 1. What will you learn?

By the end of this lesson, you will know:

1. Why embedded software needs timers.
2. The difference between **sleep**, **timer**, **work item**, **workqueue**, and **delayed work**.
3. How to create a **one-shot** or **periodic** timer.
4. Why Zephyr timer expiry callbacks must be very short.
5. How to ask another thread to do work using `k_work_submit()`.
6. How to perform a job **later** using `k_work_schedule()` or `k_work_reschedule()`.
7. How to build and run everything on **QEMU**.
8. How the concepts compare with FreeRTOS and ThreadX.

No physical microcontroller is required.

---

## 2. Think of an alarm clock and a helper

Imagine you are studying at home:

- An **alarm clock** rings every 30 minutes: that is a **periodic timer**.
- A **one-time alarm** reminds you about a meeting: that is a **one-shot timer**.
- You ask a **helper** to wash dishes, so you can continue studying: that is a **workqueue**.
- You tell the helper, “Wash the dishes in 10 minutes”: that is **delayed work**.

In a tiny computer, the “helper” is usually another **thread**, not another CPU.

**Important idea:** A timer tells you *when* something should happen. A workqueue gives you a *thread context* in which to do that work.

---

## 3. Start with the difference: `k_sleep()` vs `k_timer`

### `k_sleep()` — the calling thread waits

```c
printk("Before sleep\n");
k_sleep(K_MSEC(500));
printk("After sleep\n");
```

The current thread becomes **not ready to run** for roughly 500 ms. Other ready threads can use the CPU.

### `k_timer` — the clock runs independently of your application thread

You start a timer and continue executing:

```c
k_timer_start(&my_timer, K_MSEC(500), K_NO_WAIT);
printk("I did not wait here!\n");
```

After the timeout, the timer can invoke its expiry callback or record its expiration for another thread to observe.

| Question | `k_sleep()` | `k_timer` |
|---|---|---|
| What is it? | A way to delay the calling thread | A kernel timer object |
| Does the calling thread wait? | Yes | Not just from starting it |
| Can it repeat automatically? | Only if your thread loops | Yes, set a nonzero period |
| Can it notify another thread? | Not by itself | Yes, through a callback or timer status |
| Typical use | Poll a sensor every so often | Generate timeout/periodic notifications |

> Neither mechanism promises perfectly exact wall-clock timing. Interrupt latency, tick resolution, QEMU emulation, and scheduling can affect when your application handles a timeout.

---

## 4. One-shot and periodic timers

A Zephyr timer has **two timing arguments**:

```c
k_timer_start(&my_timer, duration, period);
```

- **`duration`**: when the **first** expiration occurs.
- **`period`**: interval between later expirations.

### One-shot (rings once)

```c
k_timer_start(&my_timer, K_SECONDS(2), K_NO_WAIT);
```

```text
Time: 0s ----- 1s ----- 2s ----- 3s
      START             RING        (finished)
```

### Periodic (rings repeatedly)

```c
k_timer_start(&my_timer, K_MSEC(500), K_MSEC(500));
```

```text
Time: 0ms --- 500 --- 1000 --- 1500 --- 2000
      START    RING     RING     RING     RING
```

Stop it with:

```c
k_timer_stop(&my_timer);
```

**Do not confuse** a periodic timer with a thread that calls `k_sleep()` in a loop. Both can produce periodic behavior, but they use different mechanisms.

---

## 5. Macro creation and runtime initialization

You asked about Zephyr's `K_*_DEFINE()` family in the previous lesson. Timers and work objects have it too.

| Object | Define it statically | Initialize an existing object at runtime |
|---|---|---|
| Timer | `K_TIMER_DEFINE(timer, expiry, stop)` | `k_timer_init(&timer, expiry, stop)` |
| Ordinary work | `K_WORK_DEFINE(work, handler)` | `k_work_init(&work, handler)` |
| Delayed work | `K_WORK_DELAYABLE_DEFINE(work, handler)` | `k_work_init_delayable(&work, handler)` |
| Custom workqueue | Provide a `struct k_work_q`, thread stack, initialize and start it | `k_work_queue_init()` + `k_work_queue_start()` |

**Static definition example:**

```c
static void alarm_callback(struct k_timer *timer)
{
    ARG_UNUSED(timer);
    /* Short, interrupt-safe action */
}

K_TIMER_DEFINE(alarm, alarm_callback, NULL);
```

**Runtime initialization example:**

```c
static struct k_timer alarm;

int main(void)
{
    k_timer_init(&alarm, alarm_callback, NULL);
    k_timer_start(&alarm, K_SECONDS(1), K_NO_WAIT);
    return 0;
}
```

> Initializing an object at runtime is **not the same** as automatically allocating that object's memory. In these examples, the C variable already provides the storage.

---

## 6. Critical rule: Zephyr timer expiry callbacks run in interrupt context

Look at this:

```c
static void timer_expiry(struct k_timer *timer)
{
    ARG_UNUSED(timer);
    /* Called from the system clock interrupt handler */
}
```

An **interrupt** is like a very urgent knock on the CPU's door. Keep interrupt work short so normal processing can resume quickly.

**Safe pattern:** Signal a thread or submit lightweight work.

```c
K_SEM_DEFINE(timer_signal, 0, 1);

static void timer_expiry(struct k_timer *timer)
{
    ARG_UNUSED(timer);
    k_sem_give(&timer_signal);   /* Allowed from ISR context */
}
```

**Do not do this inside the expiry callback:**

```c
/* WRONG IDEA — do not block from a timer expiry callback */
k_sleep(K_SECONDS(1));
k_sem_take(&timer_signal, K_FOREVER);
```

An ISR cannot block waiting for another thread. Also avoid lengthy sensor calculations, slow flash writes, or networking inside the timer expiry callback.

**Remember:** Zephyr's `k_timer` expiry callback runs in **system clock interrupt context**. This is different from the typical FreeRTOS software timer callback, which runs in a **timer service task**.

---

## 7. What is a workqueue?

A workqueue is a **queue of jobs** handled by a dedicated worker **thread**.

```text
Your main thread or an ISR
           |
           | k_work_submit()
           v
    +--------------------+
    | Workqueue          |
    | [Job A] [Job B]    |
    +--------------------+
              |
              v
    Workqueue's thread
              |
              v
      Calls job handler
```

You do not necessarily need to create a new thread for every small job.

Zephyr offers a **system workqueue**, a shared workqueue useful for relatively short deferred operations.

### Work item example

```c
static void led_work_handler(struct k_work *work)
{
    ARG_UNUSED(work);
    printk("The helper thread is doing the job!\n");
}

K_WORK_DEFINE(led_work, led_work_handler);

/* Called from a thread or supported ISR context */
k_work_submit(&led_work);
```

The call to `k_work_submit()` requests processing; the handler runs when the workqueue thread gets CPU time. **Submission is not the same as immediately finishing the job.**

### Why use a workqueue?

- Keep interrupt handlers short.
- Avoid starting a separate thread for every tiny operation.
- Let one dedicated thread perform multiple deferred jobs.

### Why not put everything on the system workqueue?

A workqueue processes jobs using its worker thread. A **slow or blocking handler** can delay other work items. Use a separate thread or custom workqueue for lengthy/blocking jobs when appropriate.

> Important detail: Repeatedly submitting the **same work item while it is already queued** does not necessarily create multiple copies. A work item is **not** a message queue and should not be used to count every event. Use a counting semaphore or message queue when every event/message matters.

---

## 8. What is delayed work?

Delayed work means **submit this job to a workqueue after a delay**.

```c
static void reminder_handler(struct k_work *work)
{
    ARG_UNUSED(work);
    printk("Reminder fired in a workqueue thread!\n");
}

K_WORK_DELAYABLE_DEFINE(reminder, reminder_handler);

/* The handler is eligible to run after about 1 second */
k_work_schedule(&reminder, K_SECONDS(1));
```

```text
main()                          System workqueue
  |
  | schedule after 1000 ms
  v
[timeout is pending]
  |
  | 1000 ms elapses
  +--------------------------> [reminder queued]
                                  |
                                  v
                              reminder_handler()
```

Unlike the timer expiry callback, the delayed work **handler runs in a workqueue thread**, so it can use thread-context APIs (but should not block the shared queue unnecessarily).

### `k_work_schedule()` vs `k_work_reschedule()`

Both are useful, but they have different deadline policies.

| API | Simple meaning |
|---|---|
| `k_work_schedule(&work, K_SECONDS(1))` | Schedule an idle delayed item; if it is already scheduled, keep the existing deadline. |
| `k_work_reschedule(&work, K_SECONDS(1))` | Reset the pending delay/deadline so the job is scheduled relative to this call. |

**Example: a button bounce**

- A button produces several rapid events.
- You want to act **only after no new event has arrived for 200 ms**.
- Each new event can call `k_work_reschedule(&button_work, K_MSEC(200))`.
- Once activity stops, the work handler runs.

This technique is called **debouncing**.

---

## 9. Timer vs ordinary work vs delayed work

| Feature | Kernel timer | Normal work item | Delayed work |
|---|---|---|---|
| Main purpose | Notice a time expiration | Run a job in a worker thread | Run a job in a worker thread later |
| Key creation macro | `K_TIMER_DEFINE` | `K_WORK_DEFINE` | `K_WORK_DELAYABLE_DEFINE` |
| Start API | `k_timer_start()` | `k_work_submit()` | `k_work_schedule()` |
| Handler context | **Interrupt** | **Workqueue thread** | **Workqueue thread** |
| Periodic automatically? | Yes, if period set | No | No; schedule again if required |
| Can handler sleep? | **No** | Technically can, but avoid blocking shared worker | Same caution |
| Common use | Periodic tick/alarm | Deferred processing | Debounce, retry, timeout processing |

A handy design is:

```text
Timer expiry (interrupt)
          |
          | k_sem_give() or k_work_submit()
          v
Application thread / workqueue thread
          |
          v
Do heavier processing safely
```

---

# 10. Hands-on QEMU application

We will demonstrate **three experiments in one program**:

1. A periodic timer wakes `main()` three times.
2. An ordinary work item runs inside the system workqueue.
3. A delayed work item runs after a timeout; then we reschedule it to demonstrate resetting the deadline.

This example uses semaphores to **confirm that the work handlers finished**, instead of guessing by sleeping for an arbitrary amount of time.

## 10.1 Folder structure — inside your `myapp/`

**All your learning applications remain inside `zephyr/myapp/`.** We do **not** edit the core Zephyr source.

```text
zephyrproject/
├── .venv/
├── zephyr/
│   ├── kernel/                     <-- Zephyr's original kernel code
│   ├── samples/                    <-- Official samples
│   └── myapp/                      <-- YOUR learning applications
│       ├── lesson03_threads/
│       ├── lesson04_sync/
│       ├── lesson05_message_queues/
│       ├── lesson06_events_polling/
│       └── lesson07_timers_workqueues/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson07_timers_workqueues/ <-- Generated build output
```

> Since `myapp/` is inside the Zephyr Git checkout, make sure your local learning files are not accidentally included in upstream Zephyr commits.

## 10.2 Step 1 — Create the folder (Windows PowerShell)

From a new PowerShell window:

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject
.\.venv\Scripts\Activate.ps1
cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson07_timers_workqueues\src | Out-Null
```

Your terminal should be inside:

```text
C:\Users\ragha\workspace\personal\zephyrproject\zephyr
```

## 10.3 Step 2 — Create `src/main.c`

Save at:

```text
zephyr/myapp/lesson07_timers_workqueues/src/main.c
```

**Complete, runnable program:**

```c
#include <zephyr/kernel.h>
#include <errno.h>

/* The timer wakes main(). Maximum 3 notifications can accumulate. */
K_SEM_DEFINE(timer_sem, 0, 3);

/* The work handlers tell main() when they have finished. */
K_SEM_DEFINE(normal_work_done, 0, 1);
K_SEM_DEFINE(delayed_work_done, 0, 1);

/* ----------------------------------------------------------
 * 1. TIMER CALLBACK: This runs in interrupt context.
 * ---------------------------------------------------------- */
static void periodic_expiry(struct k_timer *timer)
{
    ARG_UNUSED(timer);

    /* Short and ISR-safe: do not sleep here! */
    k_sem_give(&timer_sem);
}

K_TIMER_DEFINE(periodic_timer, periodic_expiry, NULL);

/* ----------------------------------------------------------
 * 2. NORMAL WORK: This runs in the system workqueue thread.
 * ---------------------------------------------------------- */
static void normal_work_handler(struct k_work *work)
{
    ARG_UNUSED(work);

    printk("[WORK] Ordinary work executed in a worker thread\n");
    k_sem_give(&normal_work_done);
}

K_WORK_DEFINE(normal_work, normal_work_handler);

/* ----------------------------------------------------------
 * 3. DELAYED WORK: Also runs in a workqueue thread.
 * ---------------------------------------------------------- */
static void delayed_work_handler(struct k_work *work)
{
    ARG_UNUSED(work);

    printk("[DELAYED] Scheduled work executed\n");
    k_sem_give(&delayed_work_done);
}

K_WORK_DELAYABLE_DEFINE(delayed_work, delayed_work_handler);

int main(void)
{
    int ret;

    printk("\n=== Lesson 07: Timers and Workqueues ===\n");

    /* EXPERIMENT A: Periodic timer */
    printk("\n[A] Starting timer: every 300 ms\n");

    k_timer_start(&periodic_timer, K_MSEC(300), K_MSEC(300));

    for (int i = 1; i <= 3; i++) {
        k_sem_take(&timer_sem, K_FOREVER);
        printk("[TIMER] main() received tick %d/3\n", i);
    }

    k_timer_stop(&periodic_timer);
    printk("[A] Periodic timer stopped\n");

    /* EXPERIMENT B: Submit work now */
    printk("\n[B] Submitting ordinary work\n");

    ret = k_work_submit(&normal_work);
    if (ret < 0) {
        printk("ERROR: k_work_submit returned %d\n", ret);
        return 1;
    }

    k_sem_take(&normal_work_done, K_FOREVER);
    printk("[B] Main knows normal work is finished\n");

    /* EXPERIMENT C1: Delay work by 700 ms */
    printk("\n[C1] Scheduling work after 700 ms\n");

    ret = k_work_schedule(&delayed_work, K_MSEC(700));
    if (ret < 0) {
        printk("ERROR: k_work_schedule returned %d\n", ret);
        return 1;
    }

    k_sem_take(&delayed_work_done, K_FOREVER);
    printk("[C1] Main knows delayed work is finished\n");

    /* EXPERIMENT C2: Start delay, then reset its deadline */
    printk("\n[C2] Schedule after 1000 ms; reset after 200 ms\n");

    ret = k_work_schedule(&delayed_work, K_MSEC(1000));
    if (ret < 0) {
        printk("ERROR: second schedule returned %d\n", ret);
        return 1;
    }

    k_sleep(K_MSEC(200));
    printk("[C2] Rescheduling: 700 ms from NOW\n");

    ret = k_work_reschedule(&delayed_work, K_MSEC(700));
    if (ret < 0) {
        printk("ERROR: reschedule returned %d\n", ret);
        return 1;
    }

    k_sem_take(&delayed_work_done, K_FOREVER);
    printk("[C2] Rescheduled work is finished\n");

    printk("\nLesson 07 complete!\n");
    return 0;
}
```

### Read the program like a story

- **Timer:** Every ~300 ms an interrupt-safe callback gives `timer_sem`.
- **Main thread:** Waits for three notifications using `k_sem_take()`.
- **Normal work:** Main submits a job; a system workqueue thread prints the message and signals completion.
- **Delayed work:** Main schedules the job for later and waits for its completion.
- **Rescheduling:** Main moves a new delayed deadline, then waits again.

This is a **finite experiment**; after the messages finish, `main()` returns.

## 10.4 Step 3 — Create `prj.conf`

Save at:

```text
zephyr/myapp/lesson07_timers_workqueues/prj.conf
```

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
CONFIG_SYSTEM_WORKQUEUE=y
CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=2048
```

- `CONFIG_MULTITHREADING=y`: enable normal RTOS threads.
- `CONFIG_PRINTK=y`: use `printk()` for terminal output.
- `CONFIG_SYSTEM_WORKQUEUE=y`: provide Zephyr's system workqueue.
- `CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=2048`: give the system workqueue a stack suitable for this basic demonstration.

## 10.5 Step 4 — Create `CMakeLists.txt`

Save at:

```text
zephyr/myapp/lesson07_timers_workqueues/CMakeLists.txt
```

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson07_timers_workqueues)

target_sources(app PRIVATE src/main.c)
```

## 10.6 Step 5 — Build firmware on QEMU

Run from **`zephyrproject/zephyr`**:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson07_timers_workqueues -d ..\build\lesson07_timers_workqueues
```

The `-p always` option ensures a **pristine configuration**, so you do not accidentally reuse another lesson's cached application path.

## 10.7 Step 6 — Run QEMU

```powershell
west build -d ..\build\lesson07_timers_workqueues -t run
```

You should see output **similar to**:

```text
SeaBIOS (version ...)
Booting from ROM..
*** Booting Zephyr OS build ... ***

=== Lesson 07: Timers and Workqueues ===

[A] Starting timer: every 300 ms
[TIMER] main() received tick 1/3
[TIMER] main() received tick 2/3
[TIMER] main() received tick 3/3
[A] Periodic timer stopped

[B] Submitting ordinary work
[WORK] Ordinary work executed in a worker thread
[B] Main knows normal work is finished

[C1] Scheduling work after 700 ms
[DELAYED] Scheduled work executed
[C1] Main knows delayed work is finished

[C2] Schedule after 1000 ms; reset after 200 ms
[C2] Rescheduling: 700 ms from NOW
[DELAYED] Scheduled work executed
[C2] Rescheduled work is finished

Lesson 07 complete!
```

**The delays are approximate**, and the SeaBIOS/Zephyr banner may differ on your installation.

QEMU may exit automatically after the application finishes depending on the board configuration. If it stays open, press **Ctrl+A**, release, then **X** to exit the QEMU console.

### If the old lesson still runs

Inspect the build cache:

```powershell
Select-String -Path ..\build\lesson07_timers_workqueues\CMakeCache.txt -Pattern "APPLICATION_SOURCE_DIR"
```

The application path should point to:

```text
.../zephyr/myapp/lesson07_timers_workqueues
```

Re-run the **pristine build** command above if it points elsewhere.

---

## 11. A fourth experiment: `k_timer_status_sync()`

Not every timer needs an expiry callback. A thread can wait directly for timer expiration:

```c
K_TIMER_DEFINE(wait_timer, NULL, NULL);

int main(void)
{
    k_timer_start(&wait_timer, K_MSEC(500), K_NO_WAIT);

    printk("Waiting for timer...\n");
    uint32_t expirations = k_timer_status_sync(&wait_timer);
    printk("Timer expired %u time(s)\n", (unsigned int)expirations);

    return 0;
}
```

This is a **separate mini-example**, not an extra `main()` to paste into the main experiment. `k_timer_status_sync()` may block **the calling thread**, and it must not be used from an ISR. It returns the expiration count since it was last read (or zero if the timer was stopped before expiration).

---

## 12. Creating your own workqueue (advanced beginner)

The system workqueue is shared. What if you have a special job that should not delay unrelated system work?

You can define your own queue:

```c
#define MY_WORKQUEUE_STACK_SIZE 1024
#define MY_WORKQUEUE_PRIORITY   5

K_THREAD_STACK_DEFINE(my_wq_stack, MY_WORKQUEUE_STACK_SIZE);
static struct k_work_q my_wq;

/* Called once, from a suitable thread during initialization */
k_work_queue_init(&my_wq);
k_work_queue_start(&my_wq,
                   my_wq_stack,
                   K_THREAD_STACK_SIZEOF(my_wq_stack),
                   MY_WORKQUEUE_PRIORITY,
                   NULL);

/* Submit an already initialized work item to this queue */
k_work_submit_to_queue(&my_wq, &normal_work);
```

This snippet demonstrates the **APIs** and is not meant to be pasted at file scope: put the initialization and submission calls inside a function, after `normal_work` has been defined or initialized.

A custom workqueue uses its **own thread and stack**, which costs extra RAM. For short learning examples, using the system workqueue is simpler.

---

## 13. FreeRTOS vs ThreadX vs Zephyr

All three RTOSes offer ways to do things after a timeout. The APIs and execution contexts differ.

| Feature | FreeRTOS | ThreadX | Zephyr |
|---|---|---|---|
| Sleep current task/thread | `vTaskDelay()` | `tx_thread_sleep()` | `k_sleep()` |
| Create software timer | `xTimerCreate()` | `tx_timer_create()` | `K_TIMER_DEFINE()` / `k_timer_init()` |
| Start timer | `xTimerStart()` | `tx_timer_activate()` | `k_timer_start()` |
| Stop timer | `xTimerStop()` | `tx_timer_deactivate()` | `k_timer_stop()` |
| Timer callback context | Timer service/daemon **task** | Normally a system **timer thread** (configuration-dependent) | System clock **interrupt** |
| Ready-made kernel workqueue API | No direct Zephyr `k_work` equivalent; use tasks/queues/notifications | No direct Zephyr `k_work` equivalent; use threads/queues/semaphores | `k_work` + workqueue thread |
| Submit deferred processing | Signal a dedicated task, e.g. task notification or queue | Signal a dedicated thread, e.g. semaphore or queue | `k_work_submit()` |
| Delayed work item | Commonly implemented with a software timer plus task signaling | Commonly timer plus worker thread signaling | `k_work_schedule()` / `k_work_reschedule()` |
| Periodic timer | Auto-reload timer | Nonzero reschedule ticks | Nonzero period |

### Very important callback difference

**FreeRTOS:** Software timer callback executes in the timer service **task**. It should **not block**, because other timer commands/callbacks share that task.

**ThreadX:** Timer expiration callbacks normally execute in the ThreadX **timer thread**; some configurations can use interrupt-based processing. Keep callbacks short, and follow the rules for your configuration.

**Zephyr:** `k_timer` expiry callback executes in the system clock **ISR**. It **must not use blocking APIs**. Hand off heavier work to a thread or workqueue.

### Typical FreeRTOS example

```c
/* Illustrative sketch — not a full standalone application */
TimerHandle_t timer = xTimerCreate(
    "alarm",
    pdMS_TO_TICKS(500),
    pdTRUE,        /* auto-reload */
    NULL,
    my_timer_callback);

xTimerStart(timer, 0);
```

### Typical ThreadX example

```c
/* Illustrative sketch — assume timer and callback are declared */
tx_timer_create(&timer,
                "alarm",
                my_timer_callback,
                0,
                50,      /* initial ticks */
                50,      /* repeat ticks */
                TX_AUTO_ACTIVATE);
```

For ThreadX, the duration is in **timer ticks**, not automatically milliseconds.

### Typical Zephyr example

```c
K_TIMER_DEFINE(alarm_timer, alarm_expiry, NULL);

k_timer_start(&alarm_timer, K_MSEC(500), K_MSEC(500));
```

---

## 14. Frequent beginner mistakes

### Mistake 1: Calling a blocking API inside a Zephyr timer callback

**Wrong:** `k_sleep(K_MSEC(100))` in `k_timer` expiry callback.

**Better:** Give a semaphore or submit a work item; let a thread handle the job.

### Mistake 2: Thinking `k_work_submit()` creates a new thread

It **does not**. It places work on an existing workqueue, whose thread executes the handler.

### Mistake 3: Thinking `k_work_schedule()` runs exactly on the deadline

The timeout makes the work **eligible to be queued**; the worker thread runs it when scheduled. Another work item may already be using the worker.

### Mistake 4: Assuming every submission will be counted

A single work item may already be queued. Submitting the same item again is not the same as adding another message. Use `k_msgq` or a counting semaphore when every occurrence must be preserved.

### Mistake 5: Blocking the system workqueue for a long time

Long sleeps or waits in shared work handlers delay other system work. Move heavyweight/blocking operations to an independent thread or custom queue when necessary.

### Mistake 6: Confusing scheduling and rescheduling

`k_work_schedule()` usually keeps an existing pending deadline. `k_work_reschedule()` resets it.

### Mistake 7: Editing Lesson 07 but running Lesson 03

Always check that both the **application source path** and **build directory** refer to Lesson 07; a build directory alone does not select the application.

---

## 15. Explore the Zephyr source code

Open these files in your local Zephyr checkout:

| File | What to look for |
|---|---|
| `kernel/timer.c` | Timer start, stop, expiry behavior |
| `kernel/timeout.c` | Internal timeout handling |
| `kernel/work.c` | Workqueue submission and work handlers |
| `include/zephyr/kernel.h` | Public timer and workqueue APIs/types |
| `kernel/sched.c` | How runnable threads are scheduled |
| `kernel/sem.c` | How `k_sem_give()` wakes a waiting thread |

From `zephyrproject/zephyr`:

```powershell
code .\kernel\timer.c
code .\kernel\work.c
code .\kernel\timeout.c
```

Try searching source symbols:

```powershell
Select-String -Path .\kernel\timer.c -Pattern "k_timer_start", "k_timer_stop"
Select-String -Path .\kernel\work.c -Pattern "k_work_submit", "k_work_reschedule"
```

**Try to trace this chain:**

```text
System clock tick / timeout
          |
          v
Timer expiry callback (ISR)
          |
          | k_sem_give(&timer_sem)
          v
Main becomes ready
          |
          v
Scheduler runs main
          |
          v
main() prints the tick message
```

And for delayed work:

```text
main() calls k_work_schedule()
          |
          v
Kernel waits for delayed-work timeout
          |
          v
Work item goes onto system workqueue
          |
          v
Workqueue thread calls delayed_work_handler()
          |
          | k_sem_give(&delayed_work_done)
          v
main() continues
```

---

## 16. Practice challenges

1. **Change the timer period:** Replace `K_MSEC(300)` with `K_MSEC(1000)`. What changes?
2. **One-shot timer:** Change the period to `K_NO_WAIT`. How many notifications arrive? Remember to adjust the `for` loop so main does not wait forever.
3. **Show timing:** Print `k_uptime_get()` before and after delayed work.
4. **Debounce simulation:** Call `k_work_reschedule()` several times 100 ms apart with a 300 ms delay. When does the handler finally execute?
5. **Experiment with priorities:** Study `CONFIG_SYSTEM_WORKQUEUE_PRIORITY` and observe how thread priority changes responsiveness.
6. **Avoid lost events:** Think about when a message queue is better than one reusable work item.

### Interview questions

- What is the difference between `k_sleep()` and `k_timer_start()`?
- Why must you not sleep inside a timer expiry callback?
- Does `k_work_submit()` immediately execute a handler?
- Does `k_work_schedule()` delay the calling thread?
- What is the difference between `k_work_schedule()` and `k_work_reschedule()`?
- How is a Zephyr timer callback different from a FreeRTOS software timer callback?
- Why does a custom workqueue use extra RAM?

---

## 17. Quick revision sheet

```text
k_sleep(K_MSEC(100))
  -> Block this thread for about 100 ms.

K_TIMER_DEFINE() + k_timer_start()
  -> Schedule one-shot or repeated timer expiration.
  -> Timer expiry callback runs in ISR context.

K_WORK_DEFINE() + k_work_submit()
  -> Ask a workqueue thread to execute a function.

K_WORK_DELAYABLE_DEFINE() + k_work_schedule()
  -> Execute that function later in a workqueue thread.

k_work_reschedule()
  -> Change/reset the deadline for delayed work.

k_sem_give() inside timer expiry
  -> Interrupt-safe way to notify a thread.
```

**One sentence to remember:** A **timer tells you the time has arrived**; a **workqueue supplies a thread to do the job**.

---

## 18. References (official documentation)

These are useful for checking APIs against your installed Zephyr version:

- [Zephyr: Timers](https://docs.zephyrproject.org/latest/kernel/services/timing/timers.html)
- [Zephyr: Workqueue Threads](https://docs.zephyrproject.org/latest/kernel/services/threads/workqueue.html)
- [Zephyr: Timer APIs](https://docs.zephyrproject.org/latest/doxygen/html/group__timer__apis.html)
- [Zephyr: Work Queue APIs](https://docs.zephyrproject.org/latest/doxygen/html/group__workqueue__apis.html)
- [Zephyr: Kernel Timing](https://docs.zephyrproject.org/latest/kernel/services/timing/clocks.html)
- [Eclipse ThreadX API header](https://github.com/eclipse-threadx/threadx/blob/master/common/inc/tx_api.h)

---

**Suggested next lesson:** **08 — Zephyr Memory Management (Stacks, Heaps, Memory Slabs, and Memory Pools)**.
