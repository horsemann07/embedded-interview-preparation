# 5. REAL-TIME SCHEDULING ALGORITHMS

## BEGINNER

### 191. Implement Rate Monotonic Scheduling Analysis.

**Answer**

Rate Monotonic Scheduling (RMS) is a **fixed-priority preemptive scheduling algorithm** for periodic real-time tasks.

The basic rule is:

> **Shorter period = higher priority.**

For example:

```text
Task    Period    Execution Time    Priority
---------------------------------------------
T1      10 ms        1 ms             Highest
T2      20 ms        2 ms             Medium
T3      50 ms        5 ms             Lowest
```

Because:

```text
10 ms < 20 ms < 50 ms
```

the priority order is:

```text
T1 > T2 > T3
```

The idea is that a task that must run more frequently gets a higher fixed priority.

---

**Counter: Which task has highest priority?**

The task with the **shortest period** has the highest RMS priority.

Example:

```text
T1 = 5 ms
T2 = 10 ms
T3 = 100 ms

Priority:

T1 > T2 > T3
```

Do not confuse period with execution time.

A task can have a short period and still have a larger execution time than another task.

---

**Counter: Can all tasks meet deadlines?**

Not necessarily.

A first check is CPU utilization:

```text
U = Σ(Ci / Ti)
```

where:

```text
Ci = worst-case execution time of task i
Ti = period of task i
```

Example:

```text
T1: C1 = 1 ms, T1 = 10 ms
T2: C2 = 2 ms, T2 = 20 ms
T3: C3 = 3 ms, T3 = 50 ms
```

Then:

```text
U = 1/10 + 2/20 + 3/50

  = 0.10 + 0.10 + 0.06

  = 0.26

  = 26%
```

A utilization of 26% is comfortably below the classical RMS sufficient bound for three tasks.

But:

> Passing the utilization test does not automatically mean that every possible RMS task set is feasible.

For exact feasibility analysis, use **response-time analysis** or another appropriate schedulability test.

---

**Counter: CPU utilization threshold?**

The classical Liu-Layland sufficient bound for `n` independent periodic tasks with RMS is:

```text
U <= n * (2^(1/n) - 1)
```

As `n` becomes large:

```text
U <= ln(2)

  ≈ 0.693
```

So approximately:

```text
69.3%
```

is a guaranteed sufficient bound for the classic assumptions.

For specific task counts:

```text
n = 1  → 100%
n = 2  → 82.8%
n = 3  → 78.0%
n = 4  → 75.7%
n large → 69.3%
```

Important:

```text
U <= bound
```

means the task set is guaranteed schedulable under the classical assumptions.

But:

```text
U > bound
```

does NOT mean failure.

It only means the simple sufficient test is inconclusive.

A task set above the bound may still be schedulable.

---

**A more exact RMS response-time analysis**

For fixed-priority tasks, response-time analysis can calculate the worst-case response time of a task.

For task `i`:

```text
Ri = Ci + Σ ceil(Ri / Tj) * Cj
```

for all higher-priority tasks `j`.

Iteratively:

```text
R(i,0) = Ci

R(i,k+1) =
    Ci +
    Σ ceil(R(i,k) / Tj) * Cj
```

If the final response time satisfies:

```text
Ri <= Di
```

then task `i` meets its deadline `Di`.

This is more informative than utilization alone.

---

**Simple RMS scheduler example**

```c
#include <stdint.h>

/*
 * Periodic tasks.
 *
 * Shortest period gets highest priority.
 */

typedef struct
{
    uint32_t period_ms;
    uint32_t execution_time_ms;
    uint8_t priority;
} Task;

/*
 * Example:
 *
 * T1: 10 ms -> priority 1 (highest)
 * T2: 20 ms -> priority 2
 * T3: 50 ms -> priority 3
 */

static const Task tasks[] =
{
    {10U, 1U, 1U},
    {20U, 2U, 2U},
    {50U, 3U, 3U}
};
```

In a real RTOS, the scheduler would use task priorities and ready states rather than manually searching an array on every tick.

---

**Interview point**

> “RMS assigns fixed priorities based on task periods. The shortest period gets the highest priority. I first calculate utilization as `Σ(C/T)` and can use the Liu-Layland bound as a sufficient test. For exact analysis, I would use response-time analysis and include blocking, context-switch overhead, and other real system costs.”

---

### 192. Preemptive scheduling: task priorities.

**Answer**

In preemptive scheduling, a currently running task can be interrupted when a higher-priority task becomes ready.

Example:

```text
Task A = low priority
Task B = high priority
```

If Task A is running:

```text
Task A running
      |
      | Task B becomes READY
      v
Context switch
      |
      v
Task B runs
      |
      | B finishes/blocks
      v
Task A resumes
```

The scheduler chooses the highest-priority ready task.

---

**Counter: How to switch context between tasks?**

A context switch saves the CPU context of the current task and restores the context of the next task.

Typical context includes:

```text
CPU registers
Program counter
Stack pointer
Status/control register
Floating-point registers, if used
Other architecture-specific state
```

Conceptually:

```text
Current task
    ↓
Save registers to Task A stack/TCB
    ↓
Select Task B
    ↓
Restore Task B registers
    ↓
Return to Task B
```

A common implementation uses:

```text
Task Control Block (TCB)
+
Task stack
+
Scheduler
```

The exact context-switch mechanism depends on the CPU architecture and RTOS.

---

**Counter: What if two tasks have the same priority?**

This depends on the scheduler.

Common possibilities are:

**Round-robin among equal-priority tasks**

```text
A → B → A → B
```

using a time slice.

Or:

**FIFO / cooperative behavior within the priority level**

A task may continue until it blocks, yields, or is otherwise preempted.

An RTOS documentation/specification should define the exact behavior.

---

**Counter: Priority inversion risk?**

Yes.

Priority inversion happens when a high-priority task is indirectly blocked by a lower-priority task holding a shared resource.

Example:

```text
High priority H
       |
       | needs mutex
       v
     BLOCKED
       |
       | mutex owned by
       v
Low priority L
```

Now suppose a medium-priority task M runs:

```text
H waiting for L
L waiting/preempted
M runs
```

The high-priority task can be delayed by the medium-priority task even though H has a higher priority.

Common solutions include:

```text
Priority inheritance
Priority ceiling
Short critical sections
Careful resource design
```

**Interview point**

> “Preemptive scheduling improves responsiveness to high-priority work, but I must account for context-switch overhead, critical sections, blocking, and priority inversion.”

---

### 193. Round-robin scheduling with time quanta.

**Answer**

Round-robin scheduling gives each ready task of the same scheduling class a fixed time slice, or **quantum**.

Example:

```text
Quantum = 5 ms
```

If three tasks are ready:

```text
Task A → 5 ms
Task B → 5 ms
Task C → 5 ms
Task A → 5 ms
...
```

The scheduler rotates through the ready tasks.

---

**Counter: Fixed time slice per task?**

Typically yes.

Example:

```text
Quantum = 10 ms
```

A task may execute for up to that amount before the scheduler rotates to another eligible task.

The actual behavior depends on the RTOS.

---

**Counter: How to handle blocked/waiting tasks?**

A blocked task should not consume CPU time.

Typical states:

```text
READY
RUNNING
BLOCKED
```

If Task B waits for I/O:

```text
A running
B blocks
C runs
A runs
...
```

When the event occurs:

```text
I/O complete
    ↓
B becomes READY
    ↓
B rejoins the scheduling queue
```

---

**Counter: Fairness vs responsiveness?**

Smaller quantum:

```text
+ Better responsiveness
+ More frequent sharing
- More context switches
- More scheduling overhead
```

Larger quantum:

```text
+ Lower context-switch overhead
+ Better cache locality in some systems
- Longer waiting time for other tasks
```

So quantum size is a tradeoff.

For real-time systems, plain round-robin is usually not enough by itself for guaranteeing deadline behavior. Priority-based or deadline-based scheduling is usually more appropriate when hard deadlines matter.

---

**Example**

```c
#include <stdint.h>

typedef struct
{
    uint8_t id;
    uint8_t ready;
} Task;

#define TIME_QUANTUM_MS 5U

static uint32_t time_slice = TIME_QUANTUM_MS;

void scheduler_tick(void)
{
    /*
     * Decrement the current task's time slice.
     */
    if (time_slice > 0U)
    {
        time_slice--;
    }

    if (time_slice == 0U)
    {
        /*
         * Move to the next READY task.
         */
        schedule_next_ready_task();

        /*
         * Restart the quantum.
         */
        time_slice = TIME_QUANTUM_MS;
    }
}
```

**Interview point**

> “Round-robin improves fairness between eligible tasks, but the time quantum is a tradeoff between responsiveness and context-switch overhead. Blocked tasks leave the ready queue and rejoin only when their blocking condition is satisfied.”

---

### 194. Deadline calculation: current time + offset.

**Answer**

A common deadline calculation is:

```text
deadline = current_time + offset
```

Example:

```text
current_time = 1000 ms
offset       = 200 ms

deadline     = 1200 ms
```

In embedded systems, however, timer overflow must be handled carefully.

---

**Counter: 32-bit timer overflow handling?**

Suppose a 32-bit timer reaches:

```text
0xFFFFFFFF
```

and wraps to:

```text
0x00000000
```

Do not compare deadlines using naive ordering such as:

```c
if (now >= deadline)
```

because wraparound can break the comparison.

A common technique is unsigned modular subtraction:

```c
static bool deadline_expired(uint32_t now,
                             uint32_t deadline)
{
    return (int32_t)(now - deadline) >= 0;
}
```

Another common embedded approach is to store the start time and test elapsed time:

```c
static bool timeout_expired(uint32_t now,
                            uint32_t start,
                            uint32_t timeout)
{
    return (uint32_t)(now - start) >= timeout;
}
```

This works correctly across a single timer wrap as long as the interval being measured is within the valid range of the timer arithmetic.

---

**Example**

Suppose:

```text
start = 0xFFFFFFF0
timeout = 20
```

The timer wraps:

```text
0xFFFFFFF0
...
0xFFFFFFFF
0x00000000
0x00000001
...
```

Unsigned subtraction still gives the correct elapsed time modulo `2^32`.

---

**Counter: Precision: millisecond or microsecond?**

Choose the time unit based on system requirements.

Milliseconds are common for:

```text
Human interface
Slow periodic tasks
Simple timeout supervision
```

Microseconds may be required for:

```text
Motor control
Communication timing
High-rate sampling
Short real-time deadlines
```

Higher precision increases:

```text
Timer frequency
Interrupt/scheduling overhead
Counter management requirements
```

So use only the precision you actually need.

---

**Counter: Atomic time read?**

Yes, this matters when reading a multi-byte timer or software time counter that can change asynchronously.

For example, on a 32-bit MCU, a 32-bit timer read may be naturally atomic.

On an 8-bit MCU, reading a 32-bit software counter may require multiple instructions.

An interrupt could occur halfway through the read.

Solutions may include:

```text
Disable interrupts briefly
Use an atomic access mechanism
Use hardware-supported atomic timer capture
Read high/low parts using a validated sequence
```

The correct method depends on the architecture.

---

**Example**

```c
#include <stdint.h>

static bool timeout_expired(uint32_t now,
                            uint32_t start,
                            uint32_t timeout)
{
    /*
     * Unsigned subtraction naturally wraps modulo 2^32.
     */
    return (uint32_t)(now - start) >= timeout;
}
```

**Interview point**

> “For embedded deadlines I prefer wrap-safe unsigned subtraction rather than naive timestamp comparisons. I also make sure the time read is atomic on the target architecture.”

---

### 195. Task activation: trigger task at specific time.

**Answer**

Task activation means making a task ready at a defined time.

Examples:

```text
Run diagnostics at 1000 ms
Run communication task every 10 ms
Run sensor processing at 500 us intervals
```

A common periodic model is:

```text
Activation 1
    ↓
Execute
    ↓
Activation 2
    ↓
Execute
    ↓
Activation 3
    ↓
Execute
```

---

**Counter: Early vs late activation penalty?**

This depends on the task.

Early activation can be a problem if the task depends on data that should not yet be available.

Late activation can be a problem if the task has a deadline.

Example:

```text
Required activation = 100 ms
Actual activation = 108 ms
```

Jitter:

```text
+8 ms
```

In real-time systems, activation jitter should be defined and bounded.

---

**Counter: Can a task be activated multiple times?**

Yes, and this is an important design question.

Suppose a periodic task is triggered again before its previous instance has completed.

Possible policies include:

```text
1. Coalesce activations
   → one pending activation only

2. Queue every activation
   → each activation becomes a pending instance

3. Reject/drop activation
   → if an instance is already pending/running

4. Overrun handling
   → report that execution time exceeded the period
```

The correct policy depends on the task semantics.

---

**Counter: Queue of pending activations?**

Yes.

A queue can be used when each activation represents meaningful work.

Example:

```text
Timer events
   ↓
Activation Queue
   ↓
Task Scheduler
   ↓
Task execution
```

But an unbounded activation queue is dangerous.

If:

```text
arrival rate > service rate
```

the queue can grow continuously.

For periodic real-time tasks, a bounded queue or a single-pending-activation model is often safer.

---

**Example periodic activation**

```c
#include <stdint.h>

static uint32_t next_release;

void scheduler_init(uint32_t now)
{
    next_release = now + 10U;
}

void scheduler_tick(uint32_t now)
{
    if ((uint32_t)(now - next_release) < 0x80000000U)
    {
        /*
         * Task release point reached/passed.
         */
        activate_task();

        /*
         * Schedule next release.
         */
        next_release += 10U;
    }
}
```

In real systems, a mature RTOS timer/tick mechanism is usually preferable to implementing scheduling logic directly in application code.

**Interview point**

> “Task activation is about deciding when a task becomes ready. I must define activation jitter, what happens if activations overlap, whether activations queue or coalesce, and how overruns are detected.”

---

## INTERMEDIATE

### 206. Priority ceiling protocol to prevent priority inversion.

**Answer**

The **Priority Ceiling Protocol (PCP)** is a synchronization protocol used to control priority inversion when multiple tasks share protected resources.

The fundamental idea is to assign each resource a **ceiling priority**.

For a resource `R`:

```text
Ceiling(R) =
highest priority of any task that may lock R
```

Example:

```text
Resource R1:

High task H can use R1
Low task L can use R1

Therefore:

Ceiling(R1) = priority(H)
```

The exact priority-number convention varies by RTOS:

```text
Smaller number = higher priority
```

or:

```text
Larger number = higher priority
```

Always state the convention before discussing numeric priority values.

---

**Counter: Boost task priority during critical section?**

In the commonly used **Immediate Priority Ceiling Protocol (IPCP/Immediate Ceiling Priority Protocol)**, when a task locks a resource, its active priority is raised to that resource's ceiling.

Example:

```text
Task L
  |
  | locks Resource R
  v
priority raised to ceiling(R)
  |
  | critical section
  v
release R
  |
  v
restore priority
```

This prevents medium-priority tasks from preempting the low-priority task while it holds a resource needed by a higher-priority task.

Be aware that terminology varies: classic PCP and immediate-ceiling variants have different blocking rules and priority behavior.

---

**Counter: How to determine ceiling priority?**

For each resource:

```text
Ceiling(resource) =
highest priority among all tasks that can lock it
```

Example:

```text
Task H = priority 1
Task M = priority 2
Task L = priority 3

Resource R can be used by H and L

Ceiling(R) = priority 1
```

---

**Counter: Restore priority after release?**

Yes.

When the task releases the resource, its effective priority should return according to the applicable protocol.

For a simple immediate-ceiling implementation:

```text
Lock
 ↓
Boost priority
 ↓
Critical section
 ↓
Unlock
 ↓
Restore previous/effective priority
```

With nested resources, restoration must account for other resources still held.

---

**Why ceiling protocols are useful**

They can provide:

```text
Bounded blocking
Reduced priority inversion
Prevention of some deadlock scenarios
More predictable real-time behavior
```

This predictability is especially valuable in real-time systems.

---

**Simple conceptual example**

```text
H = high-priority task
M = medium-priority task
L = low-priority task

Without protection:

H → blocked by L
M → preempts L
H waits for L

With ceiling:

L locks R
L priority → ceiling(R)
M cannot preempt L
L finishes
L releases R
H continues
```

**Interview point**

> “A priority ceiling protocol assigns every shared resource a ceiling based on the highest-priority task that may use it. The protocol limits priority inversion and provides a bounded blocking behavior that is useful for real-time schedulability.”

---

### 207. Earliest Deadline First (EDF) scheduling.

**Answer**

EDF is a **dynamic-priority scheduling algorithm**.

Unlike RMS, priorities are not fixed.

The scheduler chooses:

> **The ready task with the earliest absolute deadline.**

Example:

```text
Task A deadline = 100 ms
Task B deadline = 80 ms
Task C deadline = 120 ms

Run order:

B → A → C
```

because:

```text
80 < 100 < 120
```

---

**Counter: Which task runs next?**

The ready task with the nearest/earliest deadline.

Example:

```c
if (task_a.deadline < task_b.deadline)
{
    run_task_a();
}
else
{
    run_task_b();
}
```

A real scheduler needs an efficient ready queue rather than repeatedly scanning every task.

Possible data structures:

```text
Sorted list
Heap
Priority queue
Balanced tree
RTOS-specific ready structure
```

---

**Counter: Optimal for uniprocessor systems?**

Under the classical assumptions for **preemptive, independent tasks on a uniprocessor**, EDF is optimal in the sense that if a feasible schedule exists under an online work-conserving scheduler, EDF can schedule the task set.

For the classic case of periodic tasks with:

```text
deadline = period
```

the utilization feasibility condition is:

```text
U <= 1
```

This is much less conservative than the classical RMS utilization bound.

However, real embedded systems are not always ideal:

```text
Blocking
Context-switch cost
Interrupts
Release jitter
Shared resources
Non-preemptive sections
Task overhead
Multiprocessor behavior
```

all affect actual schedulability.

---

**Counter: Implementation complexity vs RMS?**

RMS:

```text
Fixed priorities
Simple priority assignment
Simple ready-queue management
Predictable
```

EDF:

```text
Dynamic priorities
Deadline tracking
Dynamic ready ordering
More scheduler overhead
More implementation complexity
```

Conceptually:

```text
RMS:
Period determines priority

EDF:
Deadline determines priority
```

---

**EDF example**

```c
typedef struct
{
    uint32_t deadline;
    uint32_t execution_time;
    uint8_t ready;
} Task;

static int find_earliest_deadline(Task *tasks,
                                  uint32_t count)
{
    int selected = -1;

    for (uint32_t i = 0U; i < count; ++i)
    {
        if (!tasks[i].ready)
        {
            continue;
        }

        if ((selected < 0) ||
            (tasks[i].deadline <
             tasks[selected].deadline))
        {
            selected = (int)i;
        }
    }

    return selected;
}
```

This is simple but not necessarily efficient enough for a high-performance scheduler.

**Interview point**

> “EDF dynamically prioritizes the ready task with the earliest deadline. For the classical preemptive uniprocessor model, EDF is optimal and can fully utilize up to 100% CPU in the ideal implicit-deadline case. The tradeoff is higher scheduler complexity compared with fixed-priority RMS.”

---

### 208. Processor frequency scaling based on load.

**Answer**

Processor frequency scaling means changing CPU frequency based on workload.

This is commonly associated with:

```text
DVFS
Dynamic Voltage and Frequency Scaling
```

Basic idea:

```text
Low workload
    ↓
Lower frequency
    ↓
Lower energy/power use

High workload
    ↓
Higher frequency
    ↓
More computational capacity
```

However, frequency and voltage are usually related. Dynamic power is approximately proportional to:

```text
P_dynamic ∝ C × V^2 × f
```

where:

```text
C = effective switched capacitance
V = voltage
f = frequency
```

So reducing voltage can have a large effect on dynamic power.

---

**Counter: Lower frequency = lower power?**

Usually it reduces **dynamic power**, but the complete picture is more complicated.

Reasons include:

```text
Dynamic power
Leakage power
Voltage scaling
Time spent running
Peripheral clocks
Memory power
Regulator efficiency
```

If you reduce frequency without reducing voltage, the instantaneous dynamic power often falls, but execution takes longer.

Energy consumption is:

```text
Energy = Power × Time
```

Therefore:

> Lower frequency does not automatically guarantee lower total energy for every workload.

---

**Counter: When to scale up/down?**

A controller can monitor CPU load and change the frequency based on thresholds.

Example:

```text
CPU load > 80%
    ↓
Increase frequency

CPU load < 30%
    ↓
Decrease frequency
```

But using a single threshold can cause oscillation.

Example:

```text
79% → scale down
81% → scale up
79% → scale down
81% → scale up
...
```

Use **hysteresis**.

Example:

```text
Scale UP:
    load > 80%

Scale DOWN:
    load < 40%
```

Now the CPU must move through a wider range before changing frequency again.

---

**Counter: Deadline guarantee maintained?**

This is the most important real-time question.

You cannot reduce CPU frequency simply because average CPU load is low.

You must verify that the chosen frequency still meets:

```text
Worst-case execution time
Deadlines
Periodic release rates
Interrupt latency
Communication timing
Scheduler overhead
```

If a task requires:

```text
WCET = 2 ms
Deadline = 5 ms
```

and frequency scaling doubles its execution time:

```text
new WCET = 4 ms
```

it may still meet its deadline.

But if:

```text
new WCET = 7 ms
```

the task misses its deadline.

Therefore a DVFS controller should maintain a **real-time performance floor**.

---

**Simple load-based controller**

```c
#include <stdint.h>

typedef enum
{
    FREQ_LOW,
    FREQ_MEDIUM,
    FREQ_HIGH
} CpuFrequency;

static CpuFrequency current_frequency = FREQ_MEDIUM;

void update_cpu_frequency(uint32_t cpu_load_percent)
{
    /*
     * Hysteresis:
     *
     * Increase frequency only above 80%.
     * Decrease only below 40%.
     */
    if (cpu_load_percent > 80U)
    {
        if (current_frequency != FREQ_HIGH)
        {
            set_cpu_frequency(FREQ_HIGH);
            current_frequency = FREQ_HIGH;
        }
    }
    else if (cpu_load_percent < 40U)
    {
        if (current_frequency != FREQ_LOW)
        {
            set_cpu_frequency(FREQ_LOW);
            current_frequency = FREQ_LOW;
        }
    }
    else
    {
        /*
         * Keep the current setting.
         */
    }
}
```

A production implementation needs to consider transition latency, voltage sequencing, clock-tree constraints, peripherals, timer recalibration, and real-time deadlines.

---

**Frequency scaling and real-time scheduling**

Think about it this way:

```text
Scheduler
    ↓
Task timing requirements
    ↓
Minimum required CPU capacity
    ↓
DVFS controller
    ↓
Allowed frequency range
```

The frequency controller should not operate independently of the real-time scheduler.

A useful design is:

```text
Required workload
       ↓
Deadline analysis
       ↓
Minimum safe frequency
       ↓
Select frequency
       ↓
Monitor actual execution/load
```

---

**Interview point**

> “Frequency scaling can reduce dynamic power, especially when voltage can also be reduced, but in a real-time system the lower frequency must still satisfy worst-case execution times and deadlines. I would use hysteresis to avoid oscillation and enforce a minimum frequency based on real-time constraints rather than using average CPU load alone.”

---

# REAL-TIME SCHEDULING — INTERVIEW CHEAT SHEET

## RMS

```text
Type:
    Fixed priority

Priority:
    Shorter period = higher priority

Basic utilization:
    U = Σ(C/T)

Classical sufficient bound:
    U <= n(2^(1/n) - 1)

Large n:
    ≈ 69.3%

Exact analysis:
    Response-time analysis
```

---

## Preemptive Scheduling

```text
Higher-priority task becomes READY
            ↓
       Context switch
            ↓
     Higher-priority task
            ↓
     Blocks/completes
            ↓
     Previous task resumes
```

Main concerns:

```text
Context-switch overhead
Priority inversion
Blocking
Interrupt latency
Critical sections
```

---

## Round Robin

```text
Task A → quantum
Task B → quantum
Task C → quantum
Task A → quantum
...
```

Smaller quantum:

```text
+ Responsiveness
- More context switches
```

Larger quantum:

```text
+ Less scheduling overhead
- Worse response for other tasks
```

---

## Deadline Calculation

Prefer wrap-safe elapsed-time checks:

```c
(uint32_t)(now - start) >= timeout
```

Avoid assuming:

```c
now >= deadline
```

will remain correct across timer wraparound.

---

## Task Activation

Define:

```text
When does activation occur?
How much jitter is allowed?
Can activations overlap?
Queue or coalesce?
What happens on overrun?
```

---

## Priority Ceiling

```text
Resource
   ↓
Highest priority of tasks that may lock it
   ↓
Ceiling priority
```

Purpose:

```text
Bound priority inversion
Reduce blocking unpredictability
Improve real-time analysis
```

---

## EDF

```text
Dynamic priority

Earliest absolute deadline
            ↓
         RUN NEXT
```

Classical ideal implicit-deadline uniprocessor condition:

```text
U <= 1
```

Tradeoff:

```text
More flexible utilization
      vs
More scheduler complexity
```

---

## DVFS / Frequency Scaling

```text
Load ↑
  ↓
Frequency ↑
  ↓
Execution capacity ↑

Load ↓
  ↓
Frequency ↓
  ↓
Dynamic power ↓
```

But always verify:

```text
WCET
Deadlines
Interrupt latency
Transition overhead
Voltage/frequency constraints
```

---

# Strong Interview Answer for the Whole Topic

> **“For real-time scheduling, I first identify the task model, including period, WCET, deadline, priority, blocking, and release behavior. For fixed-priority systems I can use RMS and response-time analysis. For dynamic-priority systems I can use EDF. I also need to account for context-switch overhead, synchronization and priority inversion, timer wraparound, activation jitter, and shared-resource blocking. If I apply frequency scaling, I must ensure that the selected frequency still provides enough CPU capacity to meet all worst-case deadlines.”**
