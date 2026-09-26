

```
RTOS — From First Principles to Expert Level
│
├── 01. Introduction to RTOS
│   ├── What is an RTOS?
│   ├── Why do we need an RTOS?
│   ├── Bare-metal vs RTOS
│   ├── General Purpose OS vs RTOS
│   ├── Hard, Firm and Soft Real-Time
│   ├── Determinism
│   ├── Deadline, Latency and Jitter
│   ├── Real-world RTOS examples
│   ├── RTOS architecture
│   └── RTOS mental model
│
├── 02. RTOS Kernel Fundamentals
│   ├── What is a Kernel?
│   ├── Kernel responsibilities
│   ├── Kernel objects
│   ├── System calls
│   ├── Kernel mode vs application mode
│   ├── Tick interrupt
│   ├── Tickless operation
│   └── Kernel initialization
│
├── 03. Tasks / Threads
│   ├── What is a task?
│   ├── Task lifecycle
│   ├── Task states
│   ├── TCB
│   ├── Task stack
│   ├── Context
│   ├── Context switching
│   ├── Task creation/deletion
│   ├── Stack overflow
│   └── Task design principles
│
├── 04. Scheduling
│   ├── Scheduler
│   ├── Preemptive scheduling
│   ├── Cooperative scheduling
│   ├── Priority-based scheduling
│   ├── Round robin
│   ├── Time slicing
│   ├── Priority inversion
│   ├── Priority inheritance
│   ├── Priority ceiling
│   └── Scheduling analysis
│
├── 05. Interrupts and RTOS
│   ├── What is an interrupt?
│   ├── ISR
│   ├── ISR vs task
│   ├── Interrupt latency
│   ├── Deferred interrupt processing
│   ├── ISR-safe APIs
│   ├── Nested interrupts
│   ├── Interrupt priorities
│   └── ISR-to-task communication
│
├── 06. Inter-Task Communication
│   ├── Why tasks need communication
│   ├── Queues
│   ├── Mailboxes
│   ├── Message buffers
│   ├── Stream buffers
│   ├── Event groups
│   ├── Notifications
│   └── Choosing the right mechanism
│
├── 07. Synchronization
│   ├── Race conditions
│   ├── Critical sections
│   ├── Mutex
│   ├── Binary semaphore
│   ├── Counting semaphore
│   ├── Recursive mutex
│   ├── Spinlocks
│   └── Synchronization patterns
│
├── 08. Memory Management
│   ├── Stack vs heap
│   ├── Static allocation
│   ├── Dynamic allocation
│   ├── Memory fragmentation
│   ├── Memory pools
│   ├── MPU
│   ├── Memory protection
│   └── Zero-copy concepts
│
├── 09. Time Management
│   ├── System tick
│   ├── Delays
│   ├── Periodic tasks
│   ├── Software timers
│   ├── Absolute vs relative timing
│   ├── Deadline management
│   ├── Jitter
│   └── Time drift
│
├── 10. RTOS Internals
│   ├── Scheduler internals
│   ├── Ready lists
│   ├── Blocked lists
│   ├── TCB internals
│   ├── Context switch internals
│   ├── PendSV
│   ├── SVC
│   ├── SysTick
│   └── Critical section implementation
│
├── 11. ARM Cortex-M + RTOS
│   ├── Exception model
│   ├── NVIC
│   ├── MSP vs PSP
│   ├── Handler vs Thread mode
│   ├── CONTROL register
│   ├── SVC
│   ├── PendSV
│   ├── SysTick
│   └── Context switching on Cortex-M
│
├── 12. RTOS Synchronization Problems
│   ├── Race condition
│   ├── Deadlock
│   ├── Livelock
│   ├── Starvation
│   ├── Priority inversion
│   ├── Priority inheritance
│   └── Avoiding concurrency bugs
│
├── 13. Advanced Scheduling
│   ├── Rate Monotonic Scheduling
│   ├── Earliest Deadline First
│   ├── Response-time analysis
│   ├── WCET
│   ├── CPU utilization
│   └── Schedulability
│
├── 14. Multi-Core RTOS
│   ├── SMP
│   ├── AMP
│   ├── Core affinity
│   ├── Inter-core communication
│   ├── Cache coherency
│   ├── Spinlocks
│   └── Multi-core
```

# RTOS — From Fundamentals to Advanced Embedded Systems

## Chapter 1 — Introduction to RTOS

---

## 1.1 Introduction

An embedded system is usually built to perform a specific job. A simple system may read a sensor, process the value, and control an output. As the product becomes more complex, the software may need to handle communication, user input, motor control, diagnostics, logging, timers, networking, and several other activities at the same time.

A very small embedded system can often handle these operations using a simple `while(1)` loop. As the number of activities increases, however, the software becomes harder to organize and harder to reason about.

#### Example: From a Simple Loop to a Complex Embedded System

Imagine a small temperature-monitoring device.

At first, the system has only one job:

1. Read the temperature sensor.
2. Check whether the temperature is too high.
3. Turn ON a fan if required.

A simple implementation might look like this:

```c
while (1)
{
    temperature = read_temperature();

    if (temperature > 40)
    {
        fan_on();
    }
    else
    {
        fan_off();
    }
}

This works well because the system has very few responsibilities.

Now imagine the product becomes more advanced. The same system must also:

Read temperature and humidity sensors
Control a fan and motor
Receive commands through UART
Send data over Bluetooth
Monitor buttons
Detect faults
Store error logs
Run periodic diagnostics
Handle communication timeouts

The main loop could start looking like this:

This is where a **Real-Time Operating System**, commonly called an **RTOS**, becomes useful.

An RTOS provides a structured environment for managing multiple activities while controlling:

* when software executes,
* how the CPU is shared,
* how tasks communicate,
* how tasks synchronize,
* how software responds to interrupts,
* and how timing requirements are handled.

The important point is that an RTOS is not simply a smaller version of Windows or Linux.

The primary concern of an RTOS is **controlled and predictable execution**.

---

# 1.2 Understanding the Term RTOS

The name can be divided into three parts:

```text
Real + Time + Operating System
```

The term **Operating System** refers to software that manages system resources and provides services to application software.

The term **Real-Time** means that the timing of an operation is part of the system's correctness.

Consider a temperature monitoring system.

Suppose the system detects an over-temperature condition and correctly calculates that the temperature is too high. If the system takes several seconds to react, the calculation may be correct, but the system has still failed its purpose.

The result must therefore be both:

```text
Correct result
      +
Correct timing
      =
Correct real-time behavior
```

This idea is fundamental to real-time systems.

---

# 1.3 Real-Time Does Not Mean Fast

A common misunderstanding is that an RTOS is simply an operating system that makes software execute faster.

That is not the correct mental model.

Consider two systems.

### System A

```text
Execution time:

10 ms
11 ms
10 ms
50 ms
12 ms
```

### System B

```text
Execution time:

15 ms
15 ms
16 ms
15 ms
16 ms
```

System A may sometimes finish earlier than System B, but its execution time has a large variation.

System B is slower in some cases, but its behavior is much more predictable.

For a real-time system, this predictability can be more important than average execution speed.

A useful mental model is:

```text
General application:
       "How fast can I finish?"

Real-time system:
       "Can I finish within the required time,
        including the worst-case condition?"
```

---

# 1.4 Time as Part of Correctness

In normal software, the following may be considered a correct result:

```text
10 + 20 = 30
```

In a real-time control system, correctness may look more like:

```text
Sensor event
     ↓
Processing
     ↓
Control decision
     ↓
Actuator command
```

The control decision might be correct only when it is produced within a specified deadline.

For example:

```text
Event occurs
│
├─────────────────────── Deadline
│                           │
│                           ↓
↓                           ✕
Start                       Too late
```

If the result arrives after the deadline, the software may no longer be useful even though the computation itself was correct.

Therefore:

> In a real-time system, **when the result is produced** can be as important as **what the result contains**.

---

# 1.5 A Simple Embedded System Without an RTOS

Consider a small embedded device containing:

* a temperature sensor,
* a button,
* a motor,
* UART communication,
* and a display.

A simple bare-metal implementation could look like this:

```c
int main(void)
{
    hardware_init();

    while (1)
    {
        read_temperature();
        check_button();
        control_motor();
        process_uart();
        update_display();
    }
}
```

At first, this design is easy to understand.

The CPU continuously executes the same sequence:

```text
                +----------------------+
                |      Main Loop       |
                +----------+-----------+
                           |
             +-------------+-------------+
             |             |             |
             ↓             ↓             ↓
      Read Temperature  Check Button  Motor Control
             |
             ↓
        UART Processing
             |
             ↓
        Update Display
             |
             +----------------------+
                      Repeat
```

For a small system this can be completely appropriate.

The difficulty appears when the software grows.

---

# 1.6 The Problem With a Large Main Loop

Suppose the execution time of each operation is approximately:

```text
Temperature reading    →  2 ms
Button processing      →  1 ms
Motor control          →  1 ms
UART processing        →  2 ms
Display update          → 20 ms
```

The loop can take roughly:

```text
2 + 1 + 1 + 2 + 20 = 26 ms
```

Now imagine a button event occurs immediately after the button-processing code has finished.

The program may have to execute the remaining operations before checking the button again.

```text
Button event
     │
     │
     ↓
+----+------------------------------------------------+
| Button processing already completed                 |
|                                                    |
| Motor control          1 ms                        |
| UART processing        2 ms                        |
| Display update        20 ms                        |
+----------------------------------------------------+
                         │
                         ↓
                 Button checked again
```

The system may therefore react much later than expected.

As software grows, the main loop becomes increasingly difficult to manage.

---

# 1.7 Introducing Tasks

An RTOS allows the application to divide work into separate **tasks**.

For example:

```text
                Application
                     │
        +------------+------------+
        │            │            │
        ↓            ↓            ↓
   Sensor Task   Motor Task    UART Task
        │            │            │
        └------------+------------┘
                     │
                     ↓
                RTOS Scheduler
                     │
                     ↓
                    CPU
```

Each task represents a logical unit of work.

For example:

```text
Sensor Task
    → Read sensors periodically

Motor Task
    → Execute motor control logic

Communication Task
    → Process incoming messages

Display Task
    → Update the display

Logging Task
    → Store diagnostic information
```

The important difference is that the application is no longer responsible for manually executing every operation in one fixed sequence.

The RTOS scheduler decides which eligible task should run.

---

# 1.8 Tasks as Independent Units of Work

A useful way to think about a task is as a worker responsible for one particular type of work.

Imagine a factory:

```text
                 Factory
                    │
      +-------------+-------------+
      │             │             │
      ↓             ↓             ↓
   Worker A      Worker B      Worker C
   Sensors        Motor         Display
```

There may be several workers, but the factory may have limited machines.

Similarly, an MCU may have only one CPU core:

```text
        Task A
        Task B
        Task C
        Task D
           │
           ↓
        One CPU
```

The tasks do not actually execute simultaneously on a single CPU.

Instead, the RTOS manages the CPU so that the tasks execute according to their scheduling rules.

On a multi-core processor, several tasks can genuinely execute in parallel, but the scheduling principles still apply.

---

# 1.9 The Scheduler

The **scheduler** is the part of the RTOS responsible for selecting which task should use the CPU.

A simplified model looks like this:

```text
                 Tasks
                   │
       +-----------+-----------+
       │           │           │
       ↓           ↓           ↓
    READY       BLOCKED     SUSPENDED
       │
       ↓
   Scheduler
       │
       ↓
Selected Task
       │
       ↓
      CPU
```

A task that is ready to execute competes for CPU time with other ready tasks.

The scheduler uses the RTOS scheduling policy to make the decision.

A common policy is priority-based scheduling.

For example:

```text
Priority 5   → Motor control
Priority 4   → Communication
Priority 2   → Display
Priority 1   → Logging
```

If all four tasks are ready, a priority-based scheduler may select the highest-priority eligible task.

The exact behavior depends on the RTOS configuration and scheduling model.

---

# 1.10 Task States

A task does not remain in the CPU continuously.

A typical RTOS maintains task states such as:

```text
                  +---------+
                  |  READY  |
                  +----+----+
                       |
                       | Scheduler selects
                       ↓
                  +---------+
                  | RUNNING |
                  +----+----+
                       |
              +--------+--------+
              |        |        |
              ↓        ↓        ↓
            Delay    Wait     Suspend
              |        |        |
              ↓        ↓        ↓
          +-------+  +-------+  +-----------+
          |BLOCKED|  |BLOCKED|  | SUSPENDED |
          +-------+  +-------+  +-----------+
```

The names and exact implementation can differ between RTOSes, but the general concept is common.

### READY

The task can run but is currently not using the CPU.

### RUNNING

The scheduler has selected the task and it is currently executing.

### BLOCKED

The task is waiting for something.

Examples:

* a delay to expire,
* a queue message,
* a semaphore,
* a notification,
* an event.

### SUSPENDED

The task has explicitly been removed from normal scheduling until it is resumed.

---

# 1.11 Blocking and Why It Matters

Blocking is one of the most important RTOS concepts.

Consider a sensor task:

```c
void SensorTask(void *argument)
{
    while (1)
    {
        read_sensor();

        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

After executing `read_sensor()`, the task does not need to continuously consume CPU time for the next 100 ms.

It can enter a blocked state.

```text
Sensor Task

RUNNING
   │
   │ read_sensor()
   ↓
Delay 100 ms
   │
   ↓
BLOCKED
   │
   │ 100 ms expires
   ↓
READY
   │
   ↓
RUNNING
```

While the sensor task is blocked, another task can use the CPU.

This is much better than continuously polling:

```c
while (timer_not_expired())
{
    /* keep checking */
}
```

Continuous polling can waste CPU cycles.

Blocking allows the RTOS to use the CPU for useful work elsewhere.

---

# 1.12 Event-Driven Execution

Many RTOS systems are naturally event-driven.

The basic pattern is:

```text
Event
  │
  ↓
Wake the appropriate task
  │
  ↓
Task executes
  │
  ↓
Task waits again
```

For example:

```text
UART data arrives
       │
       ↓
      ISR
       │
       ↓
Queue / Notification
       │
       ↓
Communication Task
       │
       ↓
Process message
       │
       ↓
Wait for next message
```

This architecture is extremely common in embedded systems.

---

# 1.13 Preemption

One of the major features of a preemptive RTOS is **preemption**.

Preemption allows a currently running task to be interrupted when another task becomes eligible to run according to the scheduling policy.

Consider:

```text
Low Priority Task
████████████████████████████████
             ↑
             │
       High Priority Task
       becomes READY
```

The scheduler may perform:

```text
Low Priority Task
        │
        │ preempted
        ↓
   Scheduler
        │
        ↓
High Priority Task
        │
        │ executes
        ↓
Low Priority Task resumes
```

Conceptually:

```text
CPU timeline

|------ Low ------|-- High --|------ Low ------|
                  ↑
               Preemption
```

This allows urgent work to receive CPU time without waiting for a lower-priority task to finish voluntarily.

---

# 1.14 Context Switching

When the RTOS changes from one task to another, the processor must preserve enough CPU state for the original task to continue correctly later.

This operation is called a **context switch**.

A simplified view is:

```text
             Task A running
                   │
                   ↓
            Save Task A context
                   │
                   ↓
               Scheduler
                   │
                   ↓
            Restore Task B context
                   │
                   ↓
             Task B running
```

The context can include processor state such as:

```text
Registers
Program Counter
Stack Pointer
Processor status
```

The exact registers and mechanism depend on the architecture.

On ARM Cortex-M systems, context switching is closely related to mechanisms such as:

```text
PendSV
SVC
SysTick
PSP
MSP
```

These topics will be covered later in depth.

---

# 1.15 Context Switching Has a Cost

A context switch is useful, but it is not free.

The processor may need to:

1. save the current task's context,
2. update scheduler data structures,
3. select another task,
4. restore the next task's context,
5. continue execution.

Therefore excessive task switching can increase CPU overhead.

A good RTOS design balances:

```text
Responsiveness
        +
Task separation
        +
CPU overhead
        +
RAM usage
```

More tasks are not automatically better.

---

# 1.16 Real-Time Requirements

A real-time system usually has timing requirements.

Suppose a motor control operation must complete within 100 µs.

```text
Event
  │
  ↓
+------------------------------------+
|       Maximum allowed time         |
|              100 µs                |
+------------------------------------+
                                    │
                                    ↓
                                 Deadline
```

Possible results:

```text
80 µs   → within deadline
95 µs   → within deadline
100 µs  → deadline met
120 µs  → deadline missed
```

The system designer therefore needs to understand not only the normal execution time, but also the **worst-case behavior**.

---

# 1.17 Hard, Firm and Soft Real-Time

Real-time systems are commonly discussed in three broad categories.

## Hard Real-Time

Missing a deadline may mean the system is considered to have failed.

Examples can include certain:

* safety-critical control systems,
* aircraft control functions,
* protection systems,
* high-integrity industrial systems.

The essential idea is:

```text
Deadline missed
      ↓
System failure or unacceptable behavior
```

---

## Firm Real-Time

A late result may no longer be useful, but an occasional missed deadline does not necessarily mean total system failure.

A simplified example is a data-processing result that becomes obsolete after a specific point in time.

```text
Useful result
│
├──────────────── Deadline
│
└───────────────┐
                │
                ↓
            Result late
            → Discard
```

---

## Soft Real-Time

Late results reduce performance or quality, but the system can continue functioning.

Examples include some:

* audio systems,
* video systems,
* user-interface systems,
* communication workloads.

For example:

```text
Expected frame interval → 16 ms

Actual:
16 ms
17 ms
18 ms
16 ms
```

A little variation may be acceptable.

---

# 1.18 Deadline

A **deadline** is the latest acceptable time for an operation to complete.

Consider:

```text
Event
  │
  ↓
Start
  │
  ├───────────────────────────┤
  │                           │
  │        allowed time       │
  │                           │
  └───────────────────────────┘
                              ↓
                           Deadline
```

The system designer must know whether the operation can meet that deadline under the required conditions.

This is where scheduling analysis, interrupt latency, execution time, and resource contention become important.

---

# 1.19 Latency

Latency describes the delay between an event and a corresponding response.

For example:

```text
Interrupt generated
        │
        │ 5 µs
        ↓
ISR starts
```

The interrupt latency in this simplified example is 5 µs.

Real systems have several different latency measurements:

```text
Interrupt latency
Scheduling latency
Context-switch latency
Task response latency
Communication latency
```

Latency is not the same as execution time.

For example:

```text
Event
  │
  │ 10 µs
  ↓
Task starts
  │
  │ 20 µs
  ↓
Task completes
```

Here:

```text
Response latency ≈ 10 µs
Execution time   ≈ 20 µs
```

The total response from the event to completion would be approximately:

```text
30 µs
```

The exact definitions depend on the measurement being made.

---

# 1.20 Jitter

Jitter describes variation in timing.

Suppose a periodic task is intended to execute every 10 ms.

Ideal behavior:

```text
|----10ms----|----10ms----|----10ms----|----10ms----|
```

Actual behavior might be:

```text
|---9---|-----11-----|----10----|------12------|
```

The interval changes from one execution to another.

That variation is timing jitter.

Jitter can come from many sources:

* interrupts,
* higher-priority tasks,
* critical sections,
* disabled interrupts,
* resource contention,
* cache behavior,
* communication activity,
* scheduler behavior.

For some applications, small jitter is acceptable.

For other control systems, jitter must be tightly controlled.

---

# 1.21 Determinism

A real-time system should have behavior that can be reasoned about within known bounds.

This property is often described as **determinism**.

Suppose a function normally takes:

```text
10 µs
```

but in a particular situation it may take:

```text
500 µs
```

The average execution time may look good, but the worst case may be unacceptable.

A real-time designer therefore studies:

```text
Best case
Typical case
Worst case
```

For example:

```text
Execution time

Best case       = 8 µs
Typical         = 10 µs
Worst case      = 15 µs
```

The worst-case value is especially important when verifying deadline requirements.

---

# 1.22 WCET

**WCET** stands for:

> Worst-Case Execution Time

WCET represents the maximum execution time that a piece of software is expected to require under defined conditions.

Consider:

```text
Function A

Best case   → 8 µs
Typical     → 10 µs
Worst case  → 15 µs
```

If a control task has only 20 µs available, a 15 µs WCET may be acceptable.

But if the same operation has:

```text
WCET = 40 µs
```

then the design must be reconsidered.

WCET analysis becomes increasingly important in high-reliability and safety-oriented systems.

---

# 1.23 Bare-Metal and RTOS Architecture

The fundamental difference can be visualized as follows.

### Bare-Metal

```text
+-------------------------+
|      Application        |
|                         |
|  Main Loop              |
|   ├─ Task A             |
|   ├─ Task B             |
|   ├─ Task C             |
|   └─ Task D             |
+------------+------------+
             |
             ↓
          Hardware
```

The application directly controls the execution flow.

### RTOS

```text
+--------------------------------+
|          Application           |
|                                |
| Task A   Task B   Task C       |
| Task D   Task E                |
+---------------+----------------+
                |
                ↓
+--------------------------------+
|          RTOS Kernel           |
|                                |
| Scheduler                      |
| Task Management                |
| Queue                          |
| Semaphore                      |
| Mutex                          |
| Timer                          |
| Event Management               |
+---------------+----------------+
                |
                ↓
+--------------------------------+
|            Hardware            |
| CPU / RAM / Timer / UART / SPI |
+--------------------------------+
```

The RTOS introduces a kernel layer that manages common operating-system services.

---

# 1.24 RTOS Kernel Responsibilities

The kernel typically provides several fundamental services.

```text
                 RTOS Kernel
                      │
       +--------------+--------------+
       │              │              │
       ↓              ↓              ↓
  Scheduling     Synchronization  Communication
       │              │              │
       ↓              ↓              ↓
     Tasks        Mutex/Semaphore    Queue
       │
       ↓
     Timing
       │
       ↓
    Delay/Timer
```

Depending on the RTOS, other services may include:

* memory management,
* event handling,
* software timers,
* message passing,
* thread-local storage,
* memory protection,
* networking,
* power management,
* device management.

Different RTOS implementations provide different feature sets.

---

# 1.25 Communication Between Tasks

Multiple tasks often need to exchange information.

For example:

```text
Sensor Task
     │
     │ temperature data
     ↓
+-----------+
|   Queue   |
+-----------+
     │
     ↓
Control Task
```

The queue acts as a communication mechanism.

Similarly, a task may wait for an event:

```text
Communication Task
        │
        ↓
   Wait for data
        │
        ↓
     BLOCKED
        │
        │ data arrives
        ↓
      READY
        │
        ↓
     RUNNING
```

Common RTOS communication and synchronization mechanisms include:

* queues,
* semaphores,
* mutexes,
* task notifications,
* event groups,
* message buffers,
* stream buffers.

Each mechanism solves a somewhat different problem.

---

# 1.26 Synchronization

Concurrency creates another important problem: multiple execution contexts may access the same resource.

Consider:

```c
int counter = 0;
```

Two tasks execute:

```c
counter++;
```

A simple increment can conceptually involve:

```text
READ
  ↓
MODIFY
  ↓
WRITE
```

The following sequence is possible:

```text
Task A                  Task B
------                  ------

READ counter = 0
                        READ counter = 0

increment
                        increment

WRITE 1
                        WRITE 1
```

The final value becomes:

```text
counter = 1
```

instead of:

```text
counter = 2
```

This type of bug is called a **race condition**.

RTOS systems therefore require proper synchronization.

Common mechanisms include:

```text
Mutex
Semaphore
Critical Section
Atomic Operation
Spinlock
```

The correct choice depends on the problem being solved.

---

# 1.27 Interrupts and RTOS

Embedded systems are event-driven at the hardware level.

A peripheral may generate an interrupt when:

* UART data arrives,
* a timer expires,
* SPI transfer completes,
* a GPIO changes state,
* a DMA transfer finishes,
* an ADC conversion completes.

A common architecture is:

```text
Peripheral
    │
    ↓
Interrupt
    │
    ↓
   ISR
    │
    ↓
Queue / Notification / Event
    │
    ↓
Task
```

The ISR should generally perform only the urgent work required at interrupt level.

Longer processing can often be deferred to a task.

For example:

```text
UART Interrupt
      │
      ↓
Receive byte
      │
      ↓
Place data into buffer
      │
      ↓
Notify task
      │
      ↓
UART Processing Task
      │
      ↓
Process complete message
```

This design helps keep interrupt execution short and allows the RTOS scheduler to manage the larger amount of processing.

---

# 1.28 RTOS Does Not Eliminate Software Complexity

Introducing an RTOS does not automatically solve system-design problems.

It introduces its own set of concerns:

```text
Task scheduling
Race conditions
Priority inversion
Deadlock
Starvation
Stack overflow
Memory fragmentation
Interrupt latency
Context-switch overhead
Resource contention
```

Therefore an RTOS should be viewed as an **engineering framework**, not a magic solution.

A poorly designed RTOS application can be harder to debug than a well-designed bare-metal application.

---

# 1.29 Task Granularity

One of the important architectural decisions is deciding how much functionality belongs inside one task.

A system can be designed with:

```text
Few large tasks
```

or:

```text
Many small tasks
```

Both approaches have advantages and disadvantages.

### Few large tasks

```text
+ Simpler scheduling
+ Lower task overhead
+ Less RAM usage

- More responsibility per task
- More blocking interactions
- Larger task logic
```

### Many small tasks

```text
+ Clear separation of responsibilities
+ Independent scheduling
+ Easier isolation of some workloads

- More RAM
- More context switches
- More synchronization
- More scheduling complexity
```

A good architecture chooses task boundaries based on system behavior rather than simply creating one task for every function.

---

# 1.30 Priority

Tasks are often assigned priorities.

For example:

```text
Priority 5 → Motor control
Priority 4 → Safety monitoring
Priority 3 → Communication
Priority 2 → Display
Priority 1 → Logging
```

A high-priority task generally gets preference over lower-priority tasks when the scheduler must choose between ready tasks.

However, assigning priorities is an architectural decision.

A high priority does not automatically mean:

```text
"More important because this task is important."
```

Priority should be connected to timing and scheduling requirements.

For example:

```text
Deadline-sensitive task
        ↓
Needs bounded response
        ↓
Higher scheduling priority may be appropriate
```

Poor priority assignment can lead to:

* starvation,
* priority inversion,
* unacceptable response time,
* missed deadlines.

These topics deserve separate treatment and will be covered later.

---

# 1.31 Example: Automotive ECU

Consider a simplified automotive ECU.

The hardware may include:

```text
CAN
UART
SPI
I2C
ADC
Timers
GPIO
DMA
```

The software may contain:

```text
CAN Reception Task
Sensor Task
Control Task
Diagnostic Task
Logging Task
Communication Task
```

A simplified architecture can look like:

```text
                 CAN Hardware
                      │
                      ↓
                   CAN ISR
                      │
                      ↓
                  CAN Queue
                      │
                      ↓
                CAN Task
                      │
                      ↓
              Control Decision
                      │
                      ↓
               Control Task
                      │
                      ↓
                  Actuator
```

At the same time:

```text
Sensor Task ───────→ Control Task

Diagnostic Task ───→ Diagnostic Manager

Logging Task ──────→ Storage

Communication Task → External Interface
```

The RTOS provides the mechanisms that allow these activities to coexist in a controlled way.

---

# 1.32 Example: Sensor Acquisition System

Consider a sensor that produces data every 10 ms.

The desired flow is:

```text
Sensor
  │
  ↓
Timer / Interrupt
  │
  ↓
Acquire Data
  │
  ↓
Process Data
  │
  ↓
Store / Transmit
```

A possible RTOS implementation is:

```text
+----------------------+
| Sensor Task          |
|                      |
| Read ADC             |
| Filter data          |
| Validate data        |
+----------+-----------+
           |
           ↓
       Queue
           |
           ↓
+----------------------+
| Processing Task      |
|                      |
| Analyze data         |
| Generate result      |
+----------+-----------+
           |
           ↓
      Communication
```

The important design question is not simply whether the code works.

The engineer must also determine:

```text
Period
Deadline
Worst-case execution time
Task priority
Communication latency
Interrupt latency
Queue capacity
CPU utilization
```

That is where real-time engineering begins.

---

# 1.33 RTOS and CPU Utilization

The CPU has a limited amount of processing capacity.

Suppose tasks require:

```text
Task A → 20% CPU
Task B → 15% CPU
Task C → 10% CPU
Task D → 5% CPU
```

Then approximately:

```text
Total = 50%
```

The remaining capacity may be used for:

* lower-priority work,
* background processing,
* idle time,
* power management,
* future feature growth.

As CPU utilization increases, scheduling becomes more difficult.

A system running permanently close to 100% CPU utilization leaves very little room for:

* unexpected workload,
* interrupts,
* timing variation,
* software growth,
* diagnostic activities.

Therefore CPU utilization is an important part of RTOS design.

---

# 1.34 RTOS and Memory

An RTOS also consumes memory.

A task usually requires a stack.

For example:

```text
Task A → 1 KB stack
Task B → 2 KB stack
Task C → 1 KB stack
Task D → 4 KB stack
```

Then the application may additionally require:

```text
Task Control Blocks
Queues
Semaphores
Timers
Buffers
Kernel objects
Application data
```

A conceptual memory layout might be:

```text
+-------------------------+
| Application Data        |
+-------------------------+
| Heap / Memory Pools     |
+-------------------------+
| Queue Buffers           |
+-------------------------+
| Task D Stack            |
+-------------------------+
| Task C Stack            |
+-------------------------+
| Task B Stack            |
+-------------------------+
| Task A Stack            |
+-------------------------+
| Kernel Data             |
+-------------------------+
```

On small MCUs, memory planning is often as important as CPU scheduling.

---

# 1.35 Why RTOS Knowledge Requires More Than API Knowledge

It is possible to write RTOS applications by memorizing APIs:

```c
xTaskCreate(...);
xQueueSend(...);
xSemaphoreTake(...);
vTaskDelay(...);
```

But API knowledge alone is not enough for advanced embedded development.

A strong RTOS engineer should understand what happens underneath:

```text
API
 ↓
Kernel
 ↓
Scheduler
 ↓
Task state
 ↓
Context switch
 ↓
CPU registers
 ↓
Interrupt / exception mechanism
 ↓
Hardware
```

For example, knowing how to call `vTaskDelay()` is useful.

Understanding what happens to the task after that call is more valuable.

Conceptually:

```text
vTaskDelay()
     │
     ↓
Task leaves RUNNING state
     │
     ↓
Task becomes BLOCKED
     │
     ↓
Scheduler selects another READY task
     │
     ↓
CPU executes another task
```

That deeper mental model is what this book will focus on.

---

# 1.36 The Complete RTOS Mental Model

A useful high-level model is:

```text
                         HARDWARE
                            │
                            │ interrupt / event
                            ↓
                      +-----------+
                      |    ISR    |
                      +-----+-----+
                            │
                            │ notify / queue
                            ↓
                      +-----------+
                      |   TASK    |
                      +-----+-----+
                            │
                            │ execute
                            ↓
                      +-----------+
                      | Scheduler|
                      +-----+-----+
                            │
                            ↓
                           CPU
```

And from the task perspective:

```text
             +----------------+
             |     READY      |
             +-------+--------+
                     |
                     ↓
             +----------------+
             |    RUNNING     |
             +-------+--------+
                     |
          +----------+----------+
          |          |          |
          ↓          ↓          ↓
       Delay       Queue      Event
          |          |          |
          +----------+----------+
                     │
                     ↓
                 BLOCKED
                     │
                     │ event occurs
                     ↓
                   READY
```

This simple model explains a surprisingly large portion of RTOS behavior.

---

# 1.37 Important Design Principles

Several principles should remain in mind throughout RTOS development.

### Keep time requirements explicit

Every important real-time operation should have clearly understood timing expectations.

### Keep tasks focused

A task should have a clear responsibility rather than becoming a second "main loop" containing unrelated functionality.

### Prefer blocking over unnecessary polling

When a task has nothing to do, allow it to block instead of wasting CPU time repeatedly checking a condition.

### Keep interrupt handlers short

Interrupt handlers should generally perform urgent interrupt-level work and defer larger processing to tasks where appropriate.

### Protect shared resources

Whenever multiple execution contexts can access the same resource, concurrency must be considered.

### Design for the worst case

Average execution time is not enough for important real-time paths.

### Treat RAM as a design resource

Every task, queue, buffer, and synchronization object consumes memory.

---

# 1.38 Common Mistakes in Beginner RTOS Designs

### Creating a task for every function

Not every function deserves its own task.

Functions are software units.

Tasks are scheduling units.

These are different concepts.

---

### Using delays as synchronization

Code such as:

```c
vTaskDelay(pdMS_TO_TICKS(100));
check_status();
```

does not guarantee that the required event happened during those 100 ms.

Synchronization should be based on actual events when possible.

For example:

```text
Queue
Semaphore
Notification
Event
```

is generally a better model.

---

### Giving everything high priority

If every task is high priority, priority loses its meaning.

A priority hierarchy is useful only when priorities represent meaningful scheduling requirements.

---

### Doing too much work inside an ISR

Large amounts of processing inside an ISR can increase:

```text
Interrupt latency
Scheduling latency
Jitter
```

and can prevent other interrupts from being handled promptly.

---

### Ignoring stack usage

A task can work correctly for weeks and then fail when an unusual execution path requires more stack.

Stack monitoring and measurement are therefore important.

---

# 1.39 RTOS as an Engineering Model

The most useful way to think about an RTOS is not as a collection of APIs.

Think of it as a model for organizing concurrent work.

The application can be understood as:

```text
                Events
                  │
                  ↓
              Tasks wake
                  │
                  ↓
              Scheduler
                  │
                  ↓
              CPU executes
                  │
                  ↓
           Task completes work
                  │
                  ↓
              Task waits
                  │
                  ↓
          Next event arrives
                  │
                  └───────────────→ Repeat
```

Once this model becomes clear, APIs become much easier to understand.

---

# 1.40 Chapter Summary

An RTOS provides a structured way to manage multiple activities in a resource-constrained embedded system.

Its purpose is not simply to increase execution speed.

The important concepts introduced in this chapter are:

```text
Real-Time
    ↓
Timing is part of correctness

Task
    ↓
Unit of schedulable work

Scheduler
    ↓
Chooses which eligible task runs

READY
    ↓
Task can run

RUNNING
    ↓
Task currently owns the CPU

BLOCKED
    ↓
Task is waiting for time or an event

Preemption
    ↓
A task can be interrupted by another eligible task

Context Switch
    ↓
CPU state changes from one task to another

Deadline
    ↓
Latest acceptable completion time

Latency
    ↓
Delay between an event and response

Jitter
    ↓
Variation in timing

Determinism
    ↓
Predictable behavior within known limits

WCET
    ↓
Worst-case execution time

Synchronization
    ↓
Controls access to shared resources

Communication
    ↓
Allows tasks to exchange information
```

The most important mental model to carry forward is:

```text
                  EVENT
                    │
                    ↓
                   ISR
                    │
                    ↓
             Queue / Notification
                    │
                    ↓
              Task becomes READY
                    │
                    ↓
                Scheduler
                    │
                    ↓
                   CPU
                    │
                    ↓
               Task executes
                    │
                    ↓
             Task blocks/waits
                    │
                    └──────────→ Next event
```

This model will become the foundation for everything that follows.

---

# Chapter 2 — RTOS Kernel Fundamentals

The next chapter moves below the application level and starts looking at what the RTOS kernel actually manages.

Topics will include:

* Kernel architecture
* Kernel data structures
* Task Control Block
* Task stacks
* Ready lists
* Blocked lists
* Scheduler data structures
* System tick
* Kernel tick handling
* Context-switch path
* Critical sections
* Interrupt interaction
* Kernel startup
* Scheduler startup
* Idle task
* Tickless operation
* Kernel objects

The objective is to move from simply **using an RTOS** to understanding **how the RTOS works internally**.
