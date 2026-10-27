# Lesson 05 — Zephyr Message Queues and Communication Between Threads

> **Level:** Beginner • **Platform:** Windows PowerShell + Zephyr + QEMU x86  
> **Source location:** `zephyrproject/zephyr/myapp/lesson05_message_queues/`  
> **Build location:** `zephyrproject/build/lesson05_message_queues/`

## What will we learn?

In Lesson 03, we created two threads. In Lesson 04, we learned how to prevent threads from fighting over shared data using semaphores and mutexes.

**Now we will learn how one thread sends actual information to another thread.**

By the end, you should understand:

1. Why threads need to communicate.
2. What a **message queue** is, using a simple real-world example.
3. What **producer**, **consumer**, **FIFO**, and **blocking** mean.
4. How `K_MSGQ_DEFINE`, `k_msgq_put`, and `k_msgq_get` work.
5. How to build and run a real two-thread QEMU application.
6. What happens when a queue becomes empty or full.
7. How message queues compare with semaphores and mutexes.
8. How **Zephyr, FreeRTOS, and ThreadX** implement queues.
9. Where to read the actual Zephyr implementation.

---

## 1. The problem: How do two threads talk?

Imagine a small robot with two workers:

- **Sensor worker:** Reads the temperature every few moments.
- **Display worker:** Shows the temperature on the screen.

The sensor worker knows the temperature. The display worker does not. How do we send it?

A first idea is to use a global variable:

```c
int temperature;
```

But this raises questions:

- What if the sensor changes it while the display is reading it?
- How does the display know **new** data is ready?
- What if readings arrive faster than the display can process them?

A mutex can protect the variable, and a semaphore can announce that work is ready. But a **message queue** gives us a convenient way to transfer actual data **and** to let a receiver wait until it arrives.

### Imagine a classroom mail box

```text
  Sensor thread                              Display thread
  (producer)                                (consumer)
       |                                          ^
       | puts a temperature                       | takes next temperature
       v                                          |
  +-----------------------------------------------+
  |              MESSAGE QUEUE                    |
  |  [reading #1] [reading #2] [reading #3]      |
  +-----------------------------------------------+

  Oldest waiting message is read first (FIFO).
```

A **producer** creates data. A **consumer** uses data.

A queue is a line of items waiting to be handled.

**FIFO** means *First In, First Out*: the first item placed in the queue is the first queued item removed. If a receiver is already waiting, Zephyr can deliver the message directly to that receiver instead of first storing it in the buffer.

---

## 2. Why not just use a semaphore or mutex?

| Tool | Think of it as | Main job | Carries an actual temperature value? |
|---|---|---|---|
| **Semaphore** | Doorbell | Signal that something happened or count available resources | **No**, not by itself |
| **Mutex** | Key to a room | Protect shared data while one thread uses it | **No**, not by itself |
| **Message queue** | Mailbox containing letters | Move fixed-size data between threads | **Yes** |

Example:

```text
Semaphore: "Hey! A new reading is available!"
Mutex:     "Only one person may edit the reading now."
Queue:     "Here is reading #3: 27 degrees Celsius."
```

Sometimes a shared variable plus synchronization is exactly right. A queue is especially useful when you want **several separate messages** to wait their turn.

---

## 3. How a Zephyr message queue works

A Zephyr message queue has three important properties:

1. **Item size:** How many bytes are in one message?
2. **Capacity:** How many messages can wait in the queue?
3. **Storage:** The memory where waiting messages are kept.

For example:

```text
One message = 8 bytes
Queue capacity = 4 messages
Buffer for queued messages = 8 * 4 = 32 bytes
```

The queue also has kernel bookkeeping. The 32 bytes above describe just the message storage, not total object overhead.

### Queue states

**When the queue has data:**

```text
Queue: [A] [B] [C]
        ^
        Next message read
```

**When the queue is empty:**

```text
Queue: [ empty ]
Consumer calls k_msgq_get(..., K_FOREVER)
Consumer BLOCKS: it waits without wasting CPU time spinning.
```

**When the queue is full:**

```text
Queue: [A] [B] [C] [D]  (capacity = 4)
Producer tries to add E

With K_FOREVER: producer can WAIT for a free slot.
With K_NO_WAIT: returns an error immediately.
```

The scheduler can run other ready threads while the waiting thread is blocked.

> **Important:** Queue messages are **copied** into/out of a buffer (or directly to a waiting receiver). A queue is not just a pointer to your local variable. If your message structure contains a pointer, only the pointer value is copied—**not** whatever memory it points to.

---

## 4. Three Zephyr APIs to remember

### 4.1 — Create the queue

```c
K_MSGQ_DEFINE(sensor_queue, sizeof(struct sensor_message), 4, 4);
```

| Argument | Meaning |
|---|---|
| `sensor_queue` | Our queue name |
| `sizeof(struct sensor_message)` | Size of one message |
| `4` | Maximum number of queued messages |
| `4` | Storage alignment value used in this example |

Zephyr also provides `K_MSGQ_DEFINE_TYPE(...)` for a queue of a known C type.

### 4.2 — Send a message

```c
int result = k_msgq_put(&sensor_queue, &message, K_FOREVER);
```

- `&sensor_queue` — Which queue?
- `&message` — Address of the data to copy.
- `K_FOREVER` — Wait if there is no room.
- `result == 0` — Success.

### 4.3 — Receive a message

```c
int result = k_msgq_get(&sensor_queue, &received, K_FOREVER);
```

- `&received` — Where the received data should be copied.
- `K_FOREVER` — Wait until a message arrives.
- `result == 0` — Success.

### The three timeout choices

| Timeout | Meaning |
|---|---|
| `K_NO_WAIT` | Try now; do not block. |
| `K_MSEC(500)` | Wait up to 500 ms. |
| `K_FOREVER` | Wait as long as needed. |

On a queue that is full or empty, `K_NO_WAIT` generally returns `-ENOMSG`. A finite wait may return `-EAGAIN` on timeout. Always check the return value when failure matters.

---

## 5. Our hands-on project layout

**All learning code stays under `zephyr/myapp/`.** We will not edit `zephyr/kernel/` or the official Hello World sample.

```text
zephyrproject/
├── .venv/
├── zephyr/                         <-- Zephyr RTOS repository
│   ├── kernel/
│   ├── drivers/
│   ├── samples/
│   └── myapp/                      <-- All our learning examples
│       ├── lesson03_threads/
│       ├── lesson04_sync/
│       └── lesson05_message_queues/
│           ├── CMakeLists.txt
│           ├── prj.conf
│           └── src/
│               └── main.c
└── build/
    └── lesson05_message_queues/   <-- Generated files only
```

> **Git note:** `myapp/` lives inside your Zephyr Git checkout. Keep your learning files separate from upstream Zephyr changes, and don't accidentally include them in Zephyr commits.

### Step 1 — Create the folders

Open **PowerShell**:

```powershell
cd C:\Users\ragha\workspace\personal\zephyrproject
.\.venv\Scripts\Activate.ps1
cd .\zephyr

New-Item -ItemType Directory -Force .\myapp\lesson05_message_queues\src | Out-Null
```

PowerShell uses `Activate.ps1`, not the CMD-specific `activate.bat`.

### Step 2 — Create `main.c`

Save this complete example as:

`C:\Users\ragha\workspace\personal\zephyrproject\zephyr\myapp\lesson05_message_queues\src\main.c`

```c
#include <zephyr/kernel.h>
#include <stdint.h>

#define STACK_SIZE   1024
#define PRODUCER_PRIO 5
#define CONSUMER_PRIO 4
#define READING_COUNT 6

/* One complete item to send through the queue. */
struct sensor_message {
    uint32_t sequence;
    int32_t temperature_c;
};

/* Each item has the size of our struct. Four may wait in the buffer. */
K_MSGQ_DEFINE(sensor_queue, sizeof(struct sensor_message), 4, 4);

/* Each worker needs its own stack and thread object. */
K_THREAD_STACK_DEFINE(producer_stack, STACK_SIZE);
K_THREAD_STACK_DEFINE(consumer_stack, STACK_SIZE);

static struct k_thread producer_thread_data;
static struct k_thread consumer_thread_data;

static void producer_thread(void *p1, void *p2, void *p3)
{
    ARG_UNUSED(p1);
    ARG_UNUSED(p2);
    ARG_UNUSED(p3);

    for (uint32_t i = 1; i <= READING_COUNT; i++) {
        struct sensor_message message = {
            .sequence = i,
            .temperature_c = 24 + (int32_t)i
        };

        printk("PRODUCER: send reading %u = %d C\n",
               (unsigned int)message.sequence,
               (int)message.temperature_c);

        int ret = k_msgq_put(&sensor_queue, &message, K_FOREVER);
        if (ret != 0) {
            printk("PRODUCER: send failed (%d)\n", ret);
            return;
        }

        k_sleep(K_MSEC(300));
    }

    printk("PRODUCER: finished\n");
}

static void consumer_thread(void *p1, void *p2, void *p3)
{
    ARG_UNUSED(p1);
    ARG_UNUSED(p2);
    ARG_UNUSED(p3);

    for (int i = 0; i < READING_COUNT; i++) {
        struct sensor_message received;

        int ret = k_msgq_get(&sensor_queue, &received, K_FOREVER);
        if (ret != 0) {
            printk("CONSUMER: receive failed (%d)\n", ret);
            return;
        }

        printk("CONSUMER: got reading %u = %d C\n",
               (unsigned int)received.sequence,
               (int)received.temperature_c);
    }

    printk("CONSUMER: finished\n");
}

int main(void)
{
    printk("\n=== Lesson 05: Zephyr Message Queues ===\n");
    printk("One thread creates readings; another receives them.\n");

    k_tid_t producer_id = k_thread_create(
        &producer_thread_data,
        producer_stack,
        K_THREAD_STACK_SIZEOF(producer_stack),
        producer_thread,
        NULL, NULL, NULL,
        PRODUCER_PRIO, 0, K_NO_WAIT);

    k_tid_t consumer_id = k_thread_create(
        &consumer_thread_data,
        consumer_stack,
        K_THREAD_STACK_SIZEOF(consumer_stack),
        consumer_thread,
        NULL, NULL, NULL,
        CONSUMER_PRIO, 0, K_NO_WAIT);

    /* Wait until both worker threads have returned. */
    (void)k_thread_join(producer_id, K_FOREVER);
    (void)k_thread_join(consumer_id, K_FOREVER);

    printk("Lesson 05 complete: all workers stopped.\n");
    return 0;
}
```

**What this program does:**

1. `main()` starts two workers.
2. The **consumer** waits for a message.
3. The **producer** creates six fake sensor readings, one every 300 ms.
4. Each reading travels through the queue.
5. The consumer prints each reading.
6. `main()` waits for both threads to finish.

The consumer has priority `4` and producer `5`. **In Zephyr, the smaller preemptive priority number is higher priority.** Since the consumer normally waits on an empty queue, it does not monopolize the CPU.

### Step 3 — Create `prj.conf`

Save as `zephyr/myapp/lesson05_message_queues/prj.conf`:

```ini
CONFIG_MULTITHREADING=y
CONFIG_PRINTK=y
```

There is no extra Kconfig switch required just to use standard Zephyr message queues.

### Step 4 — Create `CMakeLists.txt`

Save as `zephyr/myapp/lesson05_message_queues/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

project(lesson05_message_queues)

target_sources(app PRIVATE src/main.c)
```

### Step 5 — Build for QEMU

Run **from** `zephyrproject/zephyr`:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson05_message_queues -d ..\build\lesson05_message_queues
```

We use `-p always` for the first build here because you previously had an old lesson appearing in a new lesson's build folder. It forces a fresh configuration.

**Understand the paths:**

| Argument | Means |
|---|---|
| `-p always` | Start with a clean build configuration. |
| `-b qemu_x86` | Build for the QEMU x86 board. |
| `.\myapp\lesson05_message_queues` | **The Lesson 05 source**, under `zephyr/myapp/`. |
| `-d ..\build\lesson05_message_queues` | Keep generated files outside the source folder. |

### Step 6 — Run QEMU

```powershell
west build -d ..\build\lesson05_message_queues -t run
```

Expected **example** output (exact boot banner and line timing may differ):

```text
SeaBIOS (...)
Booting from ROM..
*** Booting Zephyr OS build ... ***

=== Lesson 05: Zephyr Message Queues ===
One thread creates readings; another receives them.
PRODUCER: send reading 1 = 25 C
CONSUMER: got reading 1 = 25 C
PRODUCER: send reading 2 = 26 C
CONSUMER: got reading 2 = 26 C
PRODUCER: send reading 3 = 27 C
CONSUMER: got reading 3 = 27 C
PRODUCER: send reading 4 = 28 C
CONSUMER: got reading 4 = 28 C
PRODUCER: send reading 5 = 29 C
CONSUMER: got reading 5 = 29 C
PRODUCER: send reading 6 = 30 C
CONSUMER: got reading 6 = 30 C
CONSUMER: finished
PRODUCER: finished
Lesson 05 complete: all workers stopped.
```

These two threads run concurrently under the scheduler. Some print messages may appear closer together or in a slightly different interleaving. The **message sequence** should remain in FIFO order for this single producer/consumer example.

QEMU often exits automatically when this simple application is finished; if it remains running, use **Ctrl+A, release, X**.

---

## 6. Trace a message through the code

Let's trace reading #1:

```text
PRODUCER thread
   |
   | struct sensor_message message = {1, 25};
   |
   | k_msgq_put(&sensor_queue, &message, K_FOREVER)
   v
+-----------------------------------+
|     Zephyr message queue          |
|     Reading #1: 25 C              |
+-----------------------------------+
   |
   | k_msgq_get(&sensor_queue, &received, K_FOREVER)
   v
CONSUMER thread
   |
   +--> printk("reading 1 = 25 C")
```

**Watch the important names:**

- `message`: Producer's **local variable**.
- `sensor_queue`: Shared RTOS **queue object**.
- `received`: Consumer's **local variable**.

The sender and receiver do not directly read and write the same local variable. The queue moves a **copy of the message**.

---

## 7. What happens if the queue is empty?

The consumer does:

```c
k_msgq_get(&sensor_queue, &received, K_FOREVER);
```

If nothing has been sent:

1. Zephyr marks the consumer as **waiting**.
2. The scheduler runs another ready thread.
3. When the producer sends a message, the consumer becomes ready.
4. The scheduler may run the consumer, depending on thread priorities.

**It is not a busy loop.** That is important for CPU usage and power consumption.

---

## 8. What happens if the queue is full?

Suppose capacity is four messages but the producer creates six before the consumer gets to process them:

```text
[1] [2] [3] [4]  <-- FULL
Producer tries message [5]
```

If the producer uses:

```c
k_msgq_put(&sensor_queue, &message, K_FOREVER);
```

It may **block** until room appears. Nothing is silently discarded by that call simply because the queue is full.

But if the producer uses:

```c
int ret = k_msgq_put(&sensor_queue, &message, K_NO_WAIT);

if (ret != 0) {
    printk("Queue full: reading was not sent\n");
}
```

It returns immediately when it cannot enqueue. **You decide what to do with unsent data:** retry, report an error, or intentionally drop it.

### Optional Experiment A — Cause a full queue

Try these controlled changes to `main.c`:

1. Change `k_sleep(K_MSEC(300))` in the producer to `k_sleep(K_MSEC(1))`.
2. Inside the consumer, after its `printk`, add `k_sleep(K_MSEC(600));`.
3. Replace the producer's `K_FOREVER` with `K_NO_WAIT` and print when `ret != 0`.
4. Rebuild with the same pristine build command.

**Observation:** Some producer sends may fail because the consumer is too slow and the four available queue slots fill. With this modification, the consumer loop still expects six successfully received messages and could wait forever after a dropped message. For a safe complete experiment, **also keep `K_FOREVER` instead of `K_NO_WAIT`** if you want to preserve the original loop termination, or redesign the protocol to notify the consumer how many messages actually succeeded.

**Recommended beginner variant:** Change only the producer delay to 1 ms and add the consumer's 600 ms delay. Leave `K_FOREVER` unchanged. Now the producer waits when the queue is full and all six messages are still delivered.

---

## 9. What happens if the receiving thread waits too long?

You can set a timeout:

```c
int ret = k_msgq_get(&sensor_queue, &received, K_MSEC(500));

if (ret == 0) {
    printk("Got a message\n");
} else {
    printk("No message arrived in time (error %d)\n", ret);
}
```

Unlike `K_FOREVER`, a finite wait allows the thread to continue and handle an error when no message arrives in time.

---

## 10. Do queues replace mutexes?

**Not completely.**

Message queues can reduce the need to share mutable application data: the consumer gets a copy of a reading, not access to the producer's local variable.

But a mutex is still useful for protecting **shared resources** such as a global configuration structure, display driver state, or a peripheral shared by multiple threads.

Simple design tip: **Pass messages between threads when possible. Protect genuinely shared state when necessary.**

---

## 11. Memory usage: Beware of large messages

Zephyr's message queue copies a fixed-size message. If one message is 1000 bytes and capacity is 10, the queue's message storage alone needs 10,000 bytes.

For tiny MCUs, that can be a lot of RAM.

```text
Small sensor reading (8 bytes) x 4 entries = 32 bytes
Large data packet (1000 bytes) x 10 entries = 10,000 bytes
```

For large payloads, more advanced designs often pass **pointers to carefully managed buffers** rather than copying the entire payload. But remember: if you queue a pointer, the referenced memory must remain valid until the receiver finishes with it. That involves ownership/lifetime rules, which we will study later.

---

## 12. Can an interrupt (ISR) use a queue?

**Yes, with an important limitation.** Zephyr allows queue send and receive operations from an ISR, but the ISR **must not wait** for space or data.

For example, an ISR can attempt:

```c
(void)k_msgq_put(&sensor_queue, &message, K_NO_WAIT);
```

Check the return value in real code to detect a full queue. The ISR should be short and fast; a worker thread can process the message later. Do **not** use `K_FOREVER` or a blocking timeout from an ISR.

Our QEMU example uses only threads, not an ISR.

---

## 13. Zephyr vs FreeRTOS vs ThreadX — Message Queues

All three RTOSes support queues for passing data. The names and configuration details differ.

| Feature | **Zephyr** | **FreeRTOS** | **ThreadX** |
|---|---|---|---|
| Main queue concept | Fixed-size message queue | Queue of fixed-size items | Queue of fixed-size messages |
| Create | `K_MSGQ_DEFINE()` or `k_msgq_init()` | `xQueueCreate()` | `tx_queue_create()` |
| Send | `k_msgq_put()` | `xQueueSend()` | `tx_queue_send()` |
| Receive | `k_msgq_get()` | `xQueueReceive()` | `tx_queue_receive()` |
| Waiting forever | `K_FOREVER` | `portMAX_DELAY` (requires appropriate config for indefinite blocking) | `TX_WAIT_FOREVER` |
| Do not wait | `K_NO_WAIT` | `0` ticks | `TX_NO_WAIT` |
| Copy messages? | Yes | Yes | Yes |
| ISR support | Nonblocking `k_msgq_put/get` | ISR-specific APIs, e.g. `xQueueSendFromISR()` | Queue APIs depend on allowed calling context; no blocking from ISR |
| Message size | Configured in **bytes** | Configured in **bytes** | Configured in **ULONG words**, with a version/configuration-dependent maximum (traditionally 1–16 words) |

### 13.1 — The same idea in Zephyr

```c
struct sensor_message msg = { .sequence = 1, .temperature_c = 25 };
k_msgq_put(&sensor_queue, &msg, K_FOREVER);
```

### 13.2 — The same idea in FreeRTOS

```c
/* Conceptual fragment: queue handle must have been created. */
QueueHandle_t q = xQueueCreate(4, sizeof(struct sensor_message));
struct sensor_message msg = { .sequence = 1, .temperature_c = 25 };
xQueueSend(q, &msg, portMAX_DELAY);
```

The usual FreeRTOS receive API is `xQueueReceive(q, &received, portMAX_DELAY)`.

### 13.3 — The same idea in ThreadX

```c
/* Conceptual fragment: create the queue and its backing storage first. */
ULONG reading = 25;
tx_queue_send(&my_queue, &reading, TX_WAIT_FOREVER);
```

The corresponding receive API is `tx_queue_receive(&my_queue, &reading, TX_WAIT_FOREVER)`.

**ThreadX detail:** Its classic queue API expresses the item size in `ULONG` units. In current ThreadX versions, the maximum can be controlled using `TX_QUEUE_MESSAGE_MAX_SIZE` (the default retains the traditional 16-`ULONG` limit). Don't assume it accepts an arbitrary byte size like Zephyr or FreeRTOS.

### 13.4 — Easy way to remember

```text
Zephyr:   k_msgq_put()         / k_msgq_get()
FreeRTOS: xQueueSend()         / xQueueReceive()
ThreadX:  tx_queue_send()      / tx_queue_receive()
```

**They all solve the same problem: Move information safely between independent tasks/threads.**

---

## 14. Find the real implementation in Zephyr

You can examine the kernel's message queue implementation in your own checkout:

```text
zephyr/
├── kernel/
│   └── msg_q.c                    <-- Message queue implementation
└── include/
    └── zephyr/
        └── kernel.h               <-- Application-facing kernel APIs
```

Depending on the Zephyr version, the actual declarations may be in headers included by `kernel.h`. Search by symbol instead of guessing line numbers:

```powershell
# Run from zephyrproject/zephyr
Select-String -Path .\kernel\msg_q.c -Pattern "k_msgq_put", "k_msgq_get"

# Search public API/header files
Get-ChildItem .\include\zephyr -Recurse -Filter *.h |
    Select-String -Pattern "K_MSGQ_DEFINE", "k_msgq_put"
```

### What to look for in `kernel/msg_q.c`

1. How the kernel knows the queue is full or empty.
2. How a message is copied.
3. How waiting threads become ready again.
4. How the scheduler may switch threads after a send/receive.

**Don't worry if the implementation looks difficult.** Start by recognizing the API names you already used in `main.c`.

---

## 15. If QEMU accidentally runs Lesson 03 or 04 again

This happened once already, so always select the **Lesson 05 source directory** when you configure the build:

```powershell
west build -p always -b qemu_x86 .\myapp\lesson05_message_queues -d ..\build\lesson05_message_queues
west build -d ..\build\lesson05_message_queues -t run
```

Verify the selected source directory:

```powershell
Select-String -Path ..\build\lesson05_message_queues\CMakeCache.txt -Pattern "APPLICATION_SOURCE_DIR", "CMAKE_HOME_DIRECTORY"
```

The configured source path should end in:

```text
zephyr/myapp/lesson05_message_queues
```

If the firmware unexpectedly prints `Lesson 03`, inspect the current `main.c` and the build cache. **Changing only the build directory name does not change the application's source.**

---

## 16. Mini quiz

Try to answer before opening the explanations.

**Q1.** What is a producer?  
**A.** A thread that creates or sends data.

**Q2.** What is a consumer?  
**A.** A thread that takes and uses the data.

**Q3.** What happens when `k_msgq_get(..., K_FOREVER)` runs on an empty queue?  
**A.** The receiving thread blocks until a message arrives (unless an unusual event such as a purge affects it).

**Q4.** Does a queue replace every mutex?  
**A.** No. A mutex is still useful for protecting shared state/resources.

**Q5.** What if the queue is full and `k_msgq_put(..., K_NO_WAIT)` is called?  
**A.** The call returns an error instead of waiting.

**Q6.** Does queueing a pointer copy the memory it points to?  
**A.** No. It copies the pointer's value; the sender and receiver must manage the pointed-to memory safely.

**Q7.** What are the equivalent API pairs in the three RTOSes?  
**A.** Zephyr `k_msgq_put/get`, FreeRTOS `xQueueSend/Receive`, ThreadX `tx_queue_send/receive`.

---

## 17. Lesson summary

```text
LESSON 03: THREADS
   Multiple workers can run under a scheduler.

LESSON 04: SEMAPHORES / MUTEXES
   Coordinate workers and protect shared state.

LESSON 05: MESSAGE QUEUES
   Move actual data between workers.
```

**One sentence to remember:** *A semaphore rings the bell, a mutex guards the door, and a message queue delivers the letter.*

### Useful official reading

- [Zephyr: Message Queues](https://docs.zephyrproject.org/latest/kernel/services/data_passing/message_queues.html)
- [Zephyr: Message Queue APIs](https://docs.zephyrproject.org/latest/doxygen/html/group__msgq__apis.html)
- [FreeRTOS: Website](https://www.freertos.org/)
- [Eclipse ThreadX: Queue documentation](https://github.com/eclipse-threadx/rtos-docs/blob/main/rtos-docs/threadx/chapter3.md)
- [Eclipse ThreadX: Current queue message-size configuration](https://github.com/eclipse-threadx/threadx/blob/master/common/inc/tx_api.h)

**Next topic suggestion:** Lesson 06 — Events, Polling, and Thread Notifications: how threads wake up when something important happens.
