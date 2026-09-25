# Expert-Level Real-Time Systems Interview Guide
## Schedulability Theory • Real-Time Communication • Research-Level Topics
### Questions 1–18 | Senior / Staff Embedded-RTOS Preparation

---

## How to answer these in an interview

For expert-level RTOS questions, do not stop at a definition.

A strong senior answer usually has this structure:

1. **Definition** — what the concept means.
2. **Model / assumption** — under what assumptions the result is true.
3. **Equation / mechanism** — the core technical idea.
4. **Why it matters in embedded systems** — timing, memory, latency, jitter, safety.
5. **Limitation / trade-off** — where the textbook result stops being enough.
6. **Practical example** — how you would use it on a real MCU/SoC/RTOS.

A very strong sentence pattern is:

> “Under the classical assumptions, X is guaranteed. In a real embedded system, I would additionally account for Y and Z.”

That sentence shows the interviewer you understand both theory and engineering.

---

# EXPERT LEVEL

# Schedulability Theory

---

## 1. Liu & Layland 1973 paper — what did it prove?

### Interview answer

The 1973 paper by C. L. Liu and James W. Layland established some of the foundations of classical uniprocessor real-time scheduling theory.

For periodic independent tasks on a single processor, under the classical assumptions, it showed two especially important results:

1. **Rate Monotonic (RM)** is an optimal **fixed-priority** assignment for the task model they considered.
2. **Earliest Deadline First (EDF)** can achieve full processor utilization for the same idealized uniprocessor model.

The paper therefore created the basic theoretical split:

```text
Fixed Priority:
    Rate Monotonic
        |
        +--> simple / static priority
        +--> strong analytical properties
        +--> utilization sufficient bound < 100%

Dynamic Priority:
    EDF
        |
        +--> priority changes with deadline
        +--> can schedule task sets up to 100% utilization
           under the classical idealized model
```

The paper's abstract explicitly states that the optimum fixed-priority scheduler has an upper utilization bound approaching about 70% for large task sets, while dynamic deadline-based priority can achieve full utilization in the model studied.

### Classical assumptions

You should mention the assumptions because the results are not universal:

- single processor
- independent periodic tasks
- fixed execution time
- no resource blocking
- no context-switch overhead in the mathematical model
- no release jitter
- typically implicit deadlines (`D = T`)
- jobs are preemptible
- no precedence constraints

Real embedded systems violate several of these.

### Rate Monotonic bound

For `n` periodic tasks:

\[
U = \sum_{i=1}^{n}\frac{C_i}{T_i}
\]

The classical sufficient RM utilization bound is:

\[
U \le n(2^{1/n}-1)
\]

As `n → ∞`:

\[
U_{bound} \rightarrow \ln(2) \approx 0.693
\]

So the famous “69.3%” is a **sufficient guarantee**, not the maximum CPU utilization RM can ever handle.

A task set can have:

```text
U = 0.80
```

and still be schedulable under RM.

Passing the bound means:

```text
Definitely schedulable under the assumptions.
```

Failing the bound means:

```text
Unknown from this test.
```

It does **not** mean unschedulable.

### Why this paper still matters

Modern systems use more sophisticated analysis, but the paper gives the conceptual foundation for:

- fixed-priority scheduling
- utilization-based schedulability tests
- EDF
- deadline-driven scheduling
- later response-time analysis

### Senior-level answer

> “Liu and Layland gave us the classical theoretical foundation for fixed-priority and deadline-driven real-time scheduling on a uniprocessor. The RM bound is a sufficient condition, while EDF is utilization-optimal for the classical model. The important engineering point is that these proofs depend on assumptions that real systems violate, so practical analysis adds blocking, jitter, overhead, cache effects, and sometimes communication delays.”

### Counter-question

**Q: Is the famous 69.3% bound the maximum utilization of RM?**

No.

It is only a **sufficient** utilization bound. Exact response-time analysis can prove schedulability above that bound for many task sets.

### Source note

Liu and Layland, *Scheduling Algorithms for Multiprogramming in a Hard-Real-Time Environment*, Journal of the ACM, 1973. DOI: `10.1145/321738.321743`.

---

## 2. What is the hyperbolic bound?

### Interview answer

The **hyperbolic bound** is a tighter utilization-based sufficient test for fixed-priority / Rate Monotonic scheduling than the classical Liu-Layland utilization bound.

For tasks with utilization:

\[
U_i = \frac{C_i}{T_i}
\]

the hyperbolic test is:

\[
\boxed{
\prod_{i=1}^{n}(1+U_i) \le 2
}
\]

If that inequality is true, the task set is guaranteed schedulable under the corresponding classical assumptions.

### Why is it tighter?

The classical bound compresses the entire task set into a single number:

\[
\sum U_i
\]

The hyperbolic test keeps more information about the **distribution** of utilization.

For example, these two systems can have the same total utilization:

```text
System A:
U = {0.50, 0.10, 0.05}

System B:
U = {0.25, 0.20, 0.20}
```

A simple sum treats them identically:

```text
U_total = 0.65
```

But the product:

\[
(1+U_1)(1+U_2)(1+U_3)
\]

is different.

That is why the hyperbolic test can reject fewer task sets than the classic bound.

### Example

Suppose:

```text
U1 = 0.50
U2 = 0.25
U3 = 0.10
```

Then:

\[
(1.5)(1.25)(1.10)=2.0625
\]

So the hyperbolic test does not certify the set.

But that still does **not** prove the set is unschedulable.

Again:

```text
Passed test    -> guaranteed schedulable
Failed test    -> inconclusive
```

### Practical importance

Hyperbolic bound is useful when:

- you want a fast admission test
- tasks have different utilizations
- exact response-time analysis is more expensive
- you are doing task-set synthesis or quick feasibility screening

For a real embedded product, I would normally use:

```text
Fast screening
    ↓
Utilization / hyperbolic test
    ↓
Exact or tighter response-time analysis
    ↓
Measurement / validation
```

### Senior-level nuance

The hyperbolic bound is still a **sufficient test**, not a complete schedulability oracle.

Also, once you introduce:

- blocking
- release jitter
- arbitrary deadlines
- execution-time overhead
- multiprocessors
- shared caches

you need a more appropriate analysis model.

### Counter-question

**Q: Why not always use the hyperbolic test instead of the 69.3% bound?**

Because it is still only a sufficient test. It can tell you quickly that a task set is definitely schedulable, but it cannot certify every schedulable task set.

For exact engineering decisions I would use response-time analysis or another analysis matched to the actual scheduling model.

---

## 3. What is blocking analysis in fixed-priority scheduling?

### Interview answer

**Blocking** is the time a task spends unable to execute because a lower-priority task is holding a resource or otherwise prevents the higher-priority task from progressing.

The classic cause is a non-preemptable critical section protected by a mutex or resource lock.

Consider:

```text
High priority:    H
Medium priority: M
Low priority:    L
```

If `L` locks a resource needed by `H`:

```text
L:   [LOCK ===== critical section ===== UNLOCK]
H:          READY → BLOCKED
```

Even though `H` has higher priority, it cannot enter the critical section until `L` releases it.

That waiting interval is **blocking**.

### Why blocking matters

Classical simple response-time analysis often starts from:

\[
R_i = C_i + \sum_{j\in hp(i)}
\left\lceil\frac{R_i}{T_j}\right\rceil C_j
\]

But in a real system a task can also be blocked by lower-priority critical sections.

So we add:

\[
\boxed{
R_i =
C_i +
B_i +
\sum_{j\in hp(i)}
\left\lceil
\frac{R_i}{T_j}
\right\rceil C_j
}
\]

where:

- `C_i` = task execution time
- `B_i` = maximum blocking time from lower-priority tasks
- `T_j` = period of higher-priority task `j`
- `C_j` = execution time of higher-priority task `j`

### Why priority inheritance is not the same thing as removing blocking

Priority inheritance mainly prevents **unbounded priority inversion caused by unrelated medium-priority work**.

It does **not** make the resource critical section disappear.

Example:

```text
L owns mutex
H waits for mutex
L inherits H's priority
L runs faster relative to M
L releases mutex
H runs
```

`H` still waits for the critical section.

The objective is to bound the blocking duration.

### Priority ceiling

Protocols such as:

- Priority Inheritance Protocol
- Priority Ceiling Protocol
- Immediate Priority Ceiling Protocol

help make that blocking analyzable and prevent problematic forms of priority inversion/deadlock.

### Senior-level answer

> “Blocking analysis adds the worst lower-priority resource-holding time into the response-time equation. Priority inheritance controls priority inversion, while ceiling protocols can also bound blocking and help prevent deadlock. In a safety or hard-real-time system I would explicitly account for every shared resource in the response-time budget rather than treating mutex time as an implementation detail.”

### Counter-question

**Q: Can a task be blocked more than once by one lower-priority task?**

Under classical priority ceiling/resource protocols the blocking can often be tightly bounded, often to one lower-priority critical section in the model. The exact bound depends on the protocol and implementation.

Do not casually claim “one time” for every RTOS mutex.

---

## 4. What is the difference between feasibility and schedulability?

### Feasibility

A task system is **feasible** if there exists **some valid schedule** that meets all timing constraints.

Think:

> “Is there any possible way to schedule this system successfully?”

### Schedulability

A task set is **schedulable by algorithm A** if algorithm A can produce a schedule that meets all the deadlines.

Think:

> “Will this specific scheduler successfully schedule it?”

### Example

Suppose:

```text
Task set = T
```

There may exist a valid schedule under EDF.

Therefore:

```text
T is feasible
T is EDF-schedulable
```

But a poor fixed-priority assignment might fail:

```text
T is feasible
T is not schedulable under that priority assignment
```

### Important relationship

For a given scheduling model:

```text
Feasible
   |
   +-- scheduler A succeeds
   |       -> schedulable by A
   |
   +-- scheduler A fails
           -> does NOT automatically mean infeasible
```

If the scheduler is **optimal** for the model, however:

```text
scheduler fails
      ↓
no feasible schedule exists
```

That is one reason optimality matters.

### Why interviewers ask this

It tests whether you understand the difference between:

- properties of the task set
- properties of a scheduling algorithm
- sufficient tests
- exact tests

### Senior-level answer

> “Feasibility is existential: there exists at least one valid schedule. Schedulability is algorithm-relative: a particular scheduling policy can or cannot realize that schedule. A schedulability test can also be sufficient rather than exact, so failing a test does not necessarily mean the task set is infeasible.”

### Counter-question

**Q: What does a sufficient schedulability test tell you?**

Passing gives a guarantee.

Failing gives no conclusion.

---

## 5. What is an optimal scheduler?

### Interview answer

A scheduler is **optimal for a scheduling model** if:

> Whenever a feasible schedule exists under that model, the scheduler can also produce a feasible schedule.

Formally:

```text
If any scheduler can meet all deadlines
        ↓
optimal scheduler also meets all deadlines
```

### EDF on a preemptive uniprocessor

EDF is optimal under the classical model of:

- one processor
- independent jobs
- preemptions allowed
- known release times
- deadlines
- no unusual resource constraints
- appropriate execution model

### Proof sketch — exchange argument

Suppose there exists a feasible schedule `S`, but at some time `t` it runs job `A` even though another available job `B` has an earlier deadline.

So:

```text
deadline(B) < deadline(A)
```

EDF would prefer `B`.

In the feasible schedule:

```text
... A ... B ...
```

Swap the execution portions so that:

```text
... B ... A ...
```

Because `B` has the earlier deadline, moving `B` earlier cannot make `B` miss its deadline.

And moving `A` later does not hurt its deadline because `A` had the later deadline.

By repeatedly performing these local exchanges, a feasible schedule can be transformed into one that always executes the ready job with the earliest deadline.

Therefore EDF can succeed whenever a feasible schedule exists.

### Why this breaks in multiprocessors

On multiple processors, EDF is not optimal in the same simple sense.

You have:

- multiple simultaneous execution slots
- migration
- partitioning
- processor affinity
- cache effects
- more complex interference

Global EDF has strong properties, but it does not inherit the simple uniprocessor optimality result.

### Senior-level nuance

“EDF is optimal” is incomplete.

The correct statement is:

> “EDF is optimal for preemptive uniprocessor scheduling under the classical independent-job model.”

Always state the model.

---

## 6. What is the multiprocessor scheduling problem? Why is it harder?

### Interview answer

On a multiprocessor system, multiple tasks must share multiple CPUs while satisfying timing constraints.

The problem becomes harder because you now have:

- several processors
- possible task migration
- load balancing
- affinity
- cache locality
- global vs partitioned decisions
- synchronization
- shared memory buses
- interference on memory and peripherals

### Uniprocessor mental model

One task runs:

```text
CPU
 |
 +-- task A
 +-- task B
 +-- task C
```

### Multiprocessor

Now:

```text
CPU0             CPU1             CPU2
 |                |                |
 A/B              C/D              E/F
```

The scheduler has two problems:

1. **Which task should execute?**
2. **On which CPU should it execute?**

That second question creates much of the complexity.

### Partitioned scheduling

Each task is permanently assigned to one CPU.

```text
CPU0: A B C
CPU1: D E
CPU2: F G
```

Advantages:

- simpler local scheduling
- no migration cost
- better cache locality
- easier analysis

Disadvantage:

- bin-packing problem
- one CPU can be overloaded while another is idle

Example:

```text
CPU0 utilization = 95%
CPU1 utilization = 45%
```

Total utilization looks reasonable, but the partition is bad.

### Global scheduling

Tasks are placed in a global ready queue and can migrate.

```text
          Global Ready Queue
             |
       +-----+-----+
       |     |     |
      CPU0  CPU1  CPU2
```

Advantages:

- better load balancing
- flexible migration

Costs:

- migration overhead
- cache disruption
- synchronization
- global scheduler contention
- more difficult analysis

### Semi-partitioned scheduling

Most tasks stay assigned to a processor, but selected tasks may migrate or be split in a controlled way.

The objective is to get a compromise:

```text
partitioning simplicity
        +
limited migration
        =
better utilization
```

---

### Follow-up: What is the Dhall effect?

The **Dhall effect** shows that global EDF on multiprocessors can behave surprisingly badly in the worst case.

The important idea is:

> A global EDF scheduler can have extremely low worst-case utilization at which it fails, even when an optimal schedule exists.

As the processor count grows, specially constructed task sets can make the utilization at failure approach arbitrarily small relative to the number of processors.

This is not the normal performance of EDF.

It is a **worst-case adversarial construction** that exposes the weakness of applying a uniprocessor intuition directly to multiprocessor systems.

### Why it matters

It explains why:

```text
EDF is optimal on 1 CPU
```

does **not** imply:

```text
EDF is optimal on N CPUs
```

### Counter-question

**Q: Which is easier to analyze, global or partitioned scheduling?**

Usually partitioned scheduling because each processor can often be analyzed as a separate uniprocessor.

The trade-off is potentially worse processor utilization because assignment becomes a bin-packing problem.

---

## 7. What is Pfair scheduling?

### Interview answer

**Pfair** means **Proportionate Fair** scheduling.

It was designed for multiprocessor real-time systems where the scheduler tries to provide each task its required share of processor time with very small deviation from the ideal fractional allocation.

The key trick is:

> Divide a task into small quantum-sized subtasks and assign each subtask deadlines so that the task receives its required processor share over time.

Suppose a task needs:

```text
50% CPU
```

With two processors and fixed slots, the ideal behavior might look like:

```text
time:    1 2 3 4 5 6 7 8
task A:  X . X . X . X .
```

Pfair tries to keep actual service close to this ideal proportion.

### Why use it?

Pfair is attractive theoretically because it can provide strong multiprocessor schedulability properties under its task model.

The classic literature studies it as a way to optimally schedule certain periodic task systems on multiprocessors.

### Why is it expensive?

Because the scheduler must reason about many small subjobs/quantum boundaries.

That can increase:

- scheduler overhead
- context-switch frequency
- state management
- implementation complexity

This is why Pfair is much more important academically than as a default scheduler in ordinary embedded RTOS products.

### Senior-level answer

> “Pfair converts each periodic task into quantum-sized subtasks with release times and deadlines so that processor service remains proportionate to the task's required utilization. It gives powerful theoretical properties on multiprocessors, but the fine-grained scheduling overhead makes it much less common in practical small-footprint RTOS designs.”

### Counter-question

**Q: Is Pfair the same as Round Robin?**

No.

Round Robin gives equal-turn service among runnable tasks.

Pfair tries to enforce each task's **required fractional processor share** with timing guarantees.

---

## 8. What is G-EDF and what are its limitations on multiprocessor systems?

### Interview answer

**G-EDF** means **Global Earliest Deadline First**.

At each scheduling decision:

```text
Select up to M READY jobs
with the earliest absolute deadlines
and run them on M processors.
```

Example with 2 CPUs:

```text
Ready jobs:

J1 deadline = 20
J2 deadline = 14
J3 deadline = 17
J4 deadline = 30

Run:
CPU0 -> J2
CPU1 -> J3
```

because 14 and 17 are the earliest deadlines.

### Why “global”?

Because runnable tasks can be selected for any processor.

Tasks may migrate:

```text
CPU0: A → B → A
CPU1: C → A → D
```

### Benefits

- natural load balancing
- no static partition assignment
- dynamic response to deadlines
- good average utilization in many workloads

### Limitations

The major limitation is that multiprocessor EDF is not optimal.

Analysis becomes complicated because of:

- migration
- carry-in jobs
- parallel interference
- cache effects
- memory-bus contention
- execution costs of migration
- non-uniform processor effects
- synchronization

Many G-EDF schedulability tests are sufficient rather than exact. This is a major difference from the elegant uniprocessor EDF result.

### Senior-level insight

For real embedded products, you often do not choose G-EDF just because it has good theoretical utilization.

You also ask:

```text
Do I need migration?
What is the cache architecture?
Can I bound scheduler overhead?
Can I certify the system?
Do tasks have affinity?
Are some functions safety-critical?
```

A slightly less flexible scheduler can sometimes be far easier to analyze and certify.

### Counter-question

**Q: Why does migration hurt real-time predictability?**

Migration can cause:

- cold caches
- TLB effects
- scheduler overhead
- synchronization traffic
- different CPU-local resources
- non-deterministic memory behavior

So the logical scheduling decision may be correct while the actual execution time becomes harder to bound.

---

# Real-Time Communication

---

## 9. What is CAN bus real-time scheduling?

### Interview answer

CAN uses **priority-based non-preemptive bus arbitration**.

The message identifier determines arbitration priority:

```text
Lower numerical CAN ID
        ↓
Higher CAN priority
```

During arbitration, dominant and recessive bits allow the highest-priority frame to win without corrupting the winning frame.

This gives CAN a very useful real-time property:

> A high-priority message can gain bus access without waiting for lower-priority messages that have not started transmitting.

However, CAN frames are **non-preemptive once transmission begins**.

So a high-priority frame can still be blocked by a lower-priority frame that already owns the bus.

### Scheduling analogy

Think:

```text
High priority = emergency vehicle
Low priority  = truck
```

If the truck has not entered the intersection:

```text
Emergency vehicle wins.
```

If the truck is already crossing:

```text
Emergency vehicle waits until it finishes.
```

That is the source of the blocking term.

### Follow-up: CAN message priority = frame ID

For Classical CAN:

```text
11-bit standard identifier
```

or an extended identifier:

```text
29-bit identifier
```

Lower numerical identifier corresponds to higher arbitration priority because dominant bits overwrite recessive bits.

For real-time analysis, you therefore assign IDs consistently with message criticality/deadline requirements.

### CAN response-time analysis

A simplified fixed-priority response-time equation is:

\[
R_i =
B_i + C_i +
\sum_{j\in hp(i)}
\left\lceil
\frac{R_i}{T_j}
\right\rceil C_j
\]

where:

- `C_i` = transmission time of message `i`
- `B_i` = blocking from one lower-priority frame already in transmission
- `hp(i)` = higher-priority messages
- `T_j` = period / minimum inter-arrival time
- `R_i` = worst-case response time from queueing to successful reception

A more realistic CAN analysis also considers factors such as:

- release jitter
- frame-format overhead
- bit stuffing
- propagation/physical effects
- retransmissions / error handling
- multiple messages released close together

The classic Tindell, Burns and Wellings work developed a response-time analysis to bound CAN message latency, which is one of the foundational results for hard real-time CAN analysis.

### Why CAN works well for control

CAN combines:

```text
Priority arbitration
        +
bounded frame transmission time
        +
short messages
        +
fault detection
```

That makes it attractive for distributed control.

### Senior-level answer

> “I treat CAN as a fixed-priority non-preemptive communication scheduler. Message IDs define priority, higher-priority messages interfere with a lower-priority message, and one already-transmitting lower-priority frame can cause blocking. For hard-real-time analysis I compute worst-case response time iteratively and include protocol overhead and error-related effects where required.”

### Counter-question

**Q: Can a lower-priority CAN frame preempt a higher-priority frame?**

No.

Arbitration happens before or while resolving access, but once a frame has won and transmission has begun, it is not preempted by another CAN frame.

That is why the blocking term exists.

### Key reference

K. Tindell, A. Burns, A. J. Wellings, *Calculating Controller Area Network (CAN) Message Response Times*, Control Engineering Practice, 1995, DOI `10.1016/0967-0661(95)00112-8`.

---

## 10. What is FlexRay and why is it more deterministic than CAN?

### Interview answer

FlexRay was designed for time-triggered and high-predictability automotive communication.

A FlexRay communication cycle can contain:

```text
+----------------+
| Static Segment |
+----------------+
| Dynamic Segment|  (optional)
+----------------+
| Symbol Window  |  (optional)
+----------------+
| NIT            |
+----------------+
```

The **static segment** uses time-triggered slots.

That means the bus schedule says in advance:

```text
slot 1 -> Node A
slot 2 -> Node B
slot 3 -> Node C
...
```

### Why this is more deterministic than CAN

CAN is:

```text
event-triggered
priority-based
contention/arbitration driven
```

FlexRay's static segment is:

```text
time-triggered
scheduled
slot based
```

Therefore the transmission opportunity is known ahead of time.

### CAN vs FlexRay

```text
CAN
 |
 +-- Priority based
 +-- Event driven
 +-- Non-preemptive
 +-- Variable waiting time
 +-- Strong real-time behavior with analysis

FlexRay
 |
 +-- TDMA/static schedule
 +-- Synchronized communication
 +-- Planned transmission slots
 +-- More predictable timing
```

### Important nuance

Do not say:

> “FlexRay is always deterministic.”

The **static segment** is highly deterministic.

FlexRay also has an optional **dynamic segment** for event-driven traffic, so not every transmission in the system has identical timing predictability.

### Why use a dynamic segment?

Some events are not naturally periodic.

FlexRay can therefore provide a hybrid model:

```text
Static segment
    -> deterministic periodic traffic

Dynamic segment
    -> event-driven traffic
```

The dynamic segment has a bounded cycle allocation so it does not destroy the timing structure of the static segment.

### Senior-level answer

> “FlexRay gives stronger temporal determinism than CAN mainly through its time-triggered static segment and synchronized communication schedule. CAN's arbitration-based access is priority deterministic but the actual waiting time depends on current bus occupancy. FlexRay trades some flexibility and schedule-management complexity for more predictable transmission timing.”

---

## 11. What is Time-Sensitive Networking (TSN)?

### Interview answer

**TSN** is a family of IEEE 802.1 Ethernet standards designed to provide predictable communication for time-sensitive applications.

The objective is to make Ethernet suitable for traffic that cares about:

- bounded latency
- low jitter
- synchronization
- bounded loss
- deterministic delivery

Examples include:

- industrial control
- robotics
- automotive networking
- audio/video
- distributed control systems

### Why ordinary Ethernet is not enough

Traditional Ethernet is optimized for flexible data networking.

Real-time control may need:

```text
Sensor sample
     ↓ 1 ms
network transmission
     ↓ 1 ms
controller
     ↓ 1 ms
actuator
```

You need a bound, not merely a good average.

### Follow-up: IEEE 802.1Qbv Time-Aware Shaper

802.1Qbv defines scheduled queue transmission.

Think of each switch output queue having a gate:

```text
Time --->

Queue A:  OPEN OPEN CLOSED CLOSED OPEN
Queue B:  CLOSED OPEN OPEN OPEN CLOSED
Queue C:  CLOSED CLOSED OPEN CLOSED CLOSED
```

The gate schedule determines which traffic class is allowed to transmit in each time window.

This is called the **Time-Aware Shaper (TAS)**.

IEEE describes 802.1Qbv as providing time-aware queue-draining procedures so bridges and end stations can schedule frame transmission based on synchronized time.

### Why this helps

Instead of:

```text
Best effort:
"send when possible"
```

you get:

```text
scheduled:
"send within this configured time window"
```

That gives a much stronger basis for latency and jitter analysis.

### But Qbv alone is not the whole TSN story

A strong answer mentions the ecosystem:

- **802.1AS** — time synchronization
- **802.1Qbv** — scheduled traffic / TAS
- **802.1Qbu / 802.3br** — frame preemption
- **802.1Qci** — per-stream filtering/policing
- other TSN standards for redundancy, shaping, and resource management

The exact set depends on the system architecture.

### Senior-level example

In an automotive Ethernet design:

```text
Camera / radar
     ↓
TSN Ethernet
     ↓
Central compute
     ↓
Control network
```

A schedule can reserve precise transmission windows for hard time-sensitive traffic while allowing less-critical traffic to use other bandwidth.

### Counter-question

**Q: Is TSN simply “Ethernet with a higher priority”?**

No.

Priority queues alone do not give deterministic timing.

TSN combines:

```text
time synchronization
+
scheduled transmission
+
traffic shaping
+
admission/configuration
+
often redundancy/monitoring
```

to make timing behavior analyzable.

---

## 12. What is ARINC 653?

### Interview answer

ARINC 653 is a standard used in avionics for **partitioning and scheduling applications with strong temporal and spatial isolation**.

The central idea is:

```text
One computing platform
        |
        +-- Partition A
        +-- Partition B
        +-- Partition C
        +-- Partition D
```

Each partition gets:

- allocated CPU time
- protected memory/resources
- controlled communication interfaces

### Follow-up: Partitioned scheduling in avionics

The system often has a repeating major frame:

```text
|---- Major Frame ----|

| A | B | A | C | B | D |
```

The scheduling table determines when each partition may execute.

The FAA description of ARINC 653 explains that partitions can be mapped to applications and scheduled in a repetitive sequence using a scheduling table.

### Follow-up: Time partitioning

Time partitioning means one partition cannot consume another partition's allocated execution window.

Example:

```text
0 ms ------------------------------- 100 ms

Partition A: [0 -------- 20]
Partition B:             [20 ---- 50]
Partition C:                      [50 ----- 80]
Partition A:                               [80 --- 100]
```

Even if `A` is busy, it cannot simply take all of `B`'s slot.

This provides **temporal isolation**.

### Follow-up: Space partitioning

Space partitioning means memory used by one partition is protected from unauthorized access by another partition.

Typical mechanisms include:

- MMU/MPU
- memory protection regions
- controlled inter-partition communication

### Why this is powerful for certification

Suppose:

```text
Partition A = flight-control function
Partition B = display
Partition C = logging
```

A failure in logging should ideally not corrupt the flight-control memory or steal its CPU budget.

That gives a much easier safety argument:

```text
fault containment
        ↓
bounded interference
        ↓
easier verification
```

### Senior-level answer

> “ARINC 653 provides a partitioned execution model with temporal and spatial isolation. The OS executes partitions according to a configured major-frame schedule, while memory protection prevents one partition from freely accessing another partition's memory. The key benefit is fault containment and predictable resource allocation, which is extremely valuable in avionics certification.”

### Counter-question

**Q: Is ARINC 653 just an RTOS scheduler?**

No.

It defines a broader application execution and partitioning model, including:

- partition management
- scheduling
- communication
- health monitoring concepts
- protected execution environments

---

# Research-Level Questions

---

## 13. What is a Cyber-Physical System (CPS) and what are its RTOS challenges?

### Interview answer

A **Cyber-Physical System** tightly integrates:

```text
Physical world
   ↕
Sensors
   ↓
Computation
   ↓
Networking
   ↓
Control logic
   ↓
Actuators
   ↓
Physical world
```

Examples:

- autonomous vehicles
- drones
- industrial robots
- medical devices
- smart grids

The key difference from an ordinary software system is that **time affects physical behavior**.

A result that arrives too late may be as bad as an incorrect result.

### CPS timing chain

Consider an autonomous braking function:

```text
Sensor
  ↓
sampling
  ↓
network
  ↓
perception
  ↓
decision
  ↓
control task
  ↓
CAN/Ethernet
  ↓
brake actuator
```

The deadline is end-to-end.

It is not enough to say:

```text
RTOS task = 2 ms
```

You must consider:

```text
sensor delay
+ queueing
+ communication
+ CPU execution
+ synchronization
+ actuator delay
```

### RTOS challenges

#### 1. End-to-end timing

A sensor-to-actuator chain needs a bounded latency.

#### 2. Jitter

Periodic control algorithms may assume relatively stable sampling intervals.

Large jitter can affect:

- control stability
- estimation
- synchronization

#### 3. Mixed criticality

A system may simultaneously run:

```text
ASIL / safety function
+
camera processing
+
infotainment
+
diagnostics
+
network services
```

Isolation becomes critical.

#### 4. Distributed coordination

Multiple ECUs/cores need:

- clock synchronization
- deterministic communication
- consistent data ownership

#### 5. Fault handling

A real CPS must react to:

- sensor failure
- communication loss
- corrupted data
- watchdog events
- timing violations

#### 6. Cybersecurity

In CPS, cybersecurity can become a safety issue.

For example:

```text
malicious network packet
      ↓
wrong control input
      ↓
physical consequence
```

### Senior-level answer

> “The RTOS in a CPS is not just a fast scheduler. It is part of a closed control loop. I care about end-to-end latency, sampling jitter, synchronization, communication determinism, fault containment, mixed-criticality isolation, and security. The timing property to analyze is often the complete sensor-to-actuator chain rather than an individual task.”

---

## 14. What is probabilistic WCET (pWCET)?

### Interview answer

Traditional WCET asks:

> “What is the safe upper bound on execution time?”

Probabilistic WCET asks:

> “What execution time bound is exceeded only with a specified very small probability?”

Example:

```text
pWCET = 80 us
P(X > 80 us) <= 10^-9
```

The exact statistical meaning depends on the analysis assumptions and confidence model.

### Why use pWCET?

Modern hardware can be extremely difficult to model exactly because of:

- caches
- branch prediction
- speculation
- out-of-order execution
- DRAM behavior
- shared resources
- dynamic frequency behavior

Measurement-based probabilistic timing analysis tries to infer extreme execution-time behavior from repeated measurements.

A common mathematical tool is **Extreme Value Theory (EVT)**.

### Important distinction

Deterministic WCET:

```text
Execution time <= W
```

under the proven model.

pWCET:

```text
Probability(execution time > W) <= p
```

under statistical/model assumptions.

That is fundamentally different.

### EVT intuition

Suppose measured execution times look like:

```text
81 79 83 80 84 82 86 81 89 ...
```

You are interested in the **tail**, not the average.

EVT tries to model rare extreme behavior.

The goal is to estimate a high quantile / exceedance probability rather than simply using:

```text
maximum observed sample
```

because the maximum of a finite test set is not a proof of the true upper tail.

### Why pWCET is not magic

A senior engineer should immediately mention assumptions:

- representative measurements
- independence / suitable dependence treatment
- stable execution-time distribution
- coverage of relevant program paths and input classes
- valid statistical model
- sufficient sample quality

If your measurements miss a behavior:

```text
statistics cannot discover what was never observed
```

### Counter-question

**Q: Can pWCET replace deterministic WCET in every safety system?**

No.

The acceptability depends on:

- safety standard
- assurance case
- analysis method
- tool qualification
- statistical assumptions
- system architecture

For safety-critical products, the engineering team must establish that the method is suitable for the applicable assurance process.

---

### Follow-up: Why is deterministic WCET hard on modern out-of-order CPUs?

Modern CPUs perform many things that are wonderful for average performance but difficult for timing analysis:

```text
Pipeline
Branch prediction
Speculative execution
Out-of-order execution
Caches
Prefetchers
DRAM controllers
Dynamic frequency
Shared last-level caches
SMT
```

For a simple in-order MCU:

```text
instruction -> known pipeline -> predictable memory
```

For a high-performance CPU:

```text
instruction
   ↓
branch prediction
   ↓
speculation
   ↓
OOO scheduling
   ↓
cache/TLB state
   ↓
shared resources
```

The execution time depends heavily on hardware history and microarchitectural state.

That does not make WCET impossible, but it makes the analysis substantially harder.

---

## 15. What is the impact of hardware caches on real-time analysis?

### Interview answer

Caches improve average performance but make timing **state-dependent**.

Suppose:

```text
RAM access = 100 ns
Cache hit = 5 ns
```

The same load instruction can therefore have dramatically different timing.

### Why that matters

For WCET, you need to reason about:

- cache hits
- cache misses
- replacement
- cache state at task release
- preemption effects
- shared cache interference

### Task execution without preemption

```text
Task A
  ↓
cache state gradually changes
  ↓
Task A completes
```

Now add a preemption:

```text
Task A
  ↓
PREEMPT
  ↓
Task B fills cache
  ↓
RESUME A
  ↓
A suffers cache misses
```

This is **cache-related preemption delay (CRPD)**.

### Follow-up: Cache-Related Preemption Delay (CRPD)

CRPD is the extra time required when a preempted task resumes and must reload cache blocks that were evicted by the preempting task.

Basic idea:

```text
Before preemption:

Cache:
[A1][A2][A3][A4]

Task B runs:

Cache:
[B1][B2][B3][A4]

A resumes:
A1 -> MISS
A2 -> MISS
A3 -> MISS
```

Those extra misses increase the response time of `A`.

### How does it enter analysis?

A generalized response-time equation can be represented as:

\[
R_i =
C_i + B_i +
\sum_{j\in hp(i)}
\left\lceil\frac{R_i}{T_j}\right\rceil
(C_j + \Delta_{i,j})
\]

where:

- `C_i` = normal execution
- `B_i` = blocking
- `C_j` = higher-priority task execution
- `Δ_i,j` = cache-related interference associated with a preemption

The exact CRPD expression is more sophisticated than one constant and depends on:

- useful cache blocks
- evicting cache blocks
- cache architecture
- replacement policy
- preemption points

### Why CRPD can be significant

A task may have a small nominal `C` but become much more expensive when repeatedly preempted.

That means:

```text
CPU utilization analysis alone
        ↓
can be too optimistic
```

### Senior-level answer

> “Caches turn WCET from a pure instruction-count problem into a microarchitectural-state problem. In preemptive systems, I have to include cache-related preemption delay because the preempting task can evict useful blocks of the preempted task. On multicore systems, shared-cache and memory-bus interference make the problem even more difficult.”

### Counter-question

**Q: Would disabling the cache solve everything?**

It can simplify timing analysis, but often at the cost of much higher execution time.

You still have:

- pipeline effects
- memory wait states
- DRAM variability
- bus contention

Also, simply disabling cache does not automatically produce a perfect timing model.

---

## 16. What is scratchpad memory vs cache in real-time systems?

### Scratchpad memory

A **scratchpad memory (SPM)** is a small on-chip memory controlled by software or compiler-managed placement.

```text
CPU
 |
 +---- SPM  <-- software decides what lives here
 |
 +---- RAM
```

### Cache

A cache is transparent hardware-managed storage.

```text
CPU
 |
 +---- Cache  <-- hardware decides contents
 |
 +---- RAM
```

### Key difference

```text
Cache:
    automatic
    transparent
    fast average performance
    harder timing analysis

Scratchpad:
    explicit control
    predictable placement
    easier timing analysis
    programmer/compiler responsibility
```

### Why scratchpad is attractive for hard real-time

Suppose you place a critical control loop in SPM:

```text
SPM:
  control_loop()
  sensor_filter()
  safety_state_machine()
```

You know approximately where the code/data are.

There is no hardware replacement event like:

```text
"Oops, another task evicted my critical block."
```

That makes execution timing easier to bound.

### Trade-off

The cost is software complexity.

You must decide:

```text
What goes into SPM?
When is it copied?
Who owns the memory?
How is data moved?
How large should each region be?
```

### Common embedded equivalent

On many MCUs, concepts such as:

- tightly coupled memory
- ITCM
- DTCM
- explicitly mapped on-chip RAM

serve a similar purpose from a timing perspective.

### Senior-level answer

> “Cache optimizes average access time transparently, but its dynamic state complicates WCET. Scratchpad is software-managed and therefore more predictable. For a hard real-time path, I can intentionally place critical code/data in predictable memory and keep less timing-sensitive work in cacheable memory.”

### Counter-question

**Q: Is scratchpad always faster than cache?**

Not necessarily.

Cache may produce excellent average performance.

The reason to choose scratchpad is often **predictability**, not average speed.

---

## 17. What is RTOS virtualization overhead analysis?

### Interview answer

Virtualization introduces another software layer between the guest RTOS and the hardware.

```text
Application
    ↓
Guest RTOS
    ↓
Virtual machine
    ↓
Hypervisor
    ↓
Hardware
```

The real-time question is:

> “How much additional worst-case latency and execution time does this layer add?”

### Sources of virtualization overhead

#### 1. VM exits / traps

A guest operation can cause a transition to hypervisor mode.

```text
Guest
  ↓
trap
  ↓
Hypervisor
  ↓
return to guest
```

Each transition costs time.

#### 2. Virtual interrupt injection

A hardware interrupt may need to be converted into a virtual interrupt.

That introduces latency.

#### 3. Virtual timer handling

The guest believes it owns a timer, but the hypervisor controls physical timing resources.

#### 4. vCPU scheduling

A guest can be runnable but not currently receiving a physical CPU.

```text
Guest wants CPU
     ↓
vCPU descheduled
     ↓
deadline delay
```

#### 5. Memory translation

Virtualized memory can create additional:

- TLB behavior
- page-table walks
- translation overhead

depending on architecture.

#### 6. Shared resources

Several guests may contend for:

- CPU
- memory bandwidth
- cache
- DMA
- network
- storage

### Simple timing model

A basic engineering model could be:

\[
C_{virtualized}
=
C_{native}
+
O_{entry/exit}
+
O_{interrupt}
+
O_{scheduler}
+
O_{I/O}
+
O_{cache}
\]

For rigorous analysis, however, overhead should not blindly be treated as one constant.

The frequency and timing of virtualization events matters.

### Better analysis

For hard real-time systems, I would characterize:

```text
Native WCET
+
worst-case hypervisor overhead
+
VM scheduling delay
+
resource interference
+
interrupt virtualization delay
```

and determine whether the hypervisor gives:

- CPU budgets
- temporal partitioning
- bounded interrupt latency
- memory isolation
- static scheduling or reservations

### Why measurement alone is dangerous

Suppose you measure:

```text
VM exit = 2 us
```

That is useful.

But the real system could experience:

```text
VM exit
+ cache eviction
+ interrupt
+ memory contention
+ hypervisor scheduling delay
```

So:

```text
average virtualization overhead
≠
worst-case real-time overhead
```

### Senior-level answer

> “I analyze virtualization as an additional execution and interference layer. I quantify VM exits, interrupt injection, vCPU scheduling delay, I/O virtualization, cache/TLB effects and shared-resource contention, then include the resulting bounds in response-time analysis. For hard real-time use I prefer a hypervisor that provides explicit temporal and spatial isolation rather than relying only on average overhead measurements.”

### Research-level nuance

Virtualization can also be useful for real-time systems because it enables:

```text
Safety-critical guest
       +
Linux guest
       +
service/infotainment guest
```

on one SoC.

The benefit is consolidation.

The cost is the need to prove that the isolation mechanisms themselves are temporally bounded.

---

## 18. What is the role of RTOS in autonomous vehicles (AUTOSAR Adaptive)?

### Interview answer

AUTOSAR Adaptive targets high-performance automotive ECUs and is designed for systems that need more dynamic software behavior than classic microcontroller-oriented AUTOSAR.

The current AUTOSAR Adaptive architecture contains:

```text
Adaptive Applications
        ↓
       ARA
        ↓
Functional Clusters
        ↓
POSIX OS
        ↓
Hardware / Virtual Machine
```

AUTOSAR's current Adaptive documentation describes the platform as implementing the AUTOSAR Runtime for Adaptive Applications (ARA), with service and API interfaces and support for high-performance ECUs. The current AUTOSAR R25-11 documentation also describes a POSIX PSE51-based operating-system interface for Adaptive Applications.

### Why this matters for autonomous vehicles

An autonomous-driving ECU may need:

- multicore CPUs
- large RAM
- Ethernet
- camera/radar/lidar processing
- service-oriented communication
- dynamic configuration
- software updates
- multiple processes
- security services

That is much closer to a high-performance computing platform than a simple 32-bit MCU.

### Follow-up: POSIX PSE51 profile

PSE51 is the **single-process profile** of POSIX designed for a real-time embedded environment.

In AUTOSAR Adaptive, the OS interface is based on this profile.

It provides APIs for things such as:

- threads
- scheduling attributes
- synchronization
- message passing
- real-time signals
- high-resolution timers
- clocks

AUTOSAR's documentation explicitly describes its Operating System Interface as providing functionality for Adaptive Applications using POSIX PSE51.

### Important nuance

PSE51 is not equivalent to “full desktop Linux POSIX.”

It is a restricted profile aimed at embedded real-time applications.

### Follow-up: Service-oriented architecture on RTOS

Instead of tightly coupling applications through direct function calls:

```text
App A
  |
  +----> App B
```

service-oriented communication looks more like:

```text
             Service
               |
       +-------+-------+
       |       |       |
     Client  Client  Client
```

A service exposes:

- methods
- events
- fields

Clients can discover and consume the service.

AUTOSAR Adaptive uses `ara::com` as a communication API model, with deployments that can use network bindings such as:

- SOME/IP
- DDS

### Example

Imagine:

```text
Camera Service
      ↓
Object Detection Service
      ↓
Path Planning Service
      ↓
Vehicle Control Service
```

The applications are connected by defined service interfaces rather than hard-coded ECU-local assumptions.

### Why this is useful for autonomous vehicles

It supports:

- modular software
- service reuse
- distributed deployment
- dynamic startup/stop
- software updates
- scalable compute architectures

AUTOSAR's current Adaptive Platform documentation describes services as potentially distributed over the in-car network and describes dynamic linking of services and clients during runtime.

### But where does the RTOS fit?

The RTOS/OS is still responsible for the low-level execution behavior:

```text
CPU scheduling
thread creation
interrupt handling
timers
memory protection
IPC
synchronization
I/O
```

The service layer adds a higher abstraction:

```text
"Give me object detections"
```

instead of:

```text
"Read buffer address 0x..."
```

The RTOS and platform therefore provide the execution foundation on which service-oriented applications run.

### Very important interview nuance

Do not say:

> “AUTOSAR Adaptive is just an RTOS.”

It is a **software platform architecture** built over an OS/POSIX environment.

Likewise, do not say:

> “AUTOSAR Adaptive always means Linux.”

A particular Adaptive implementation can run over a suitable POSIX-based operating environment, potentially including specialized or safety-oriented OS/hypervisor configurations.

### Senior-level answer

> “For autonomous-driving ECUs, AUTOSAR Adaptive gives a service-oriented software platform over a POSIX-based execution environment. The OS provides threads, scheduling, synchronization, timing and isolation; Adaptive adds execution management, communication, services, security and other functional clusters. The design supports multicore/high-performance hardware and distributed services, which is much closer to the architecture of a centralized automotive computer than a classic MCU ECU.”

### Counter-question

**Q: Why not use AUTOSAR Classic for all autonomous-driving software?**

The two platforms target different classes of systems.

Classic is strongly oriented toward:

- statically configured
- resource-constrained
- deterministic microcontroller ECUs

Adaptive is aimed at:

- high-performance ECUs
- POSIX-based systems
- dynamic applications
- service-oriented communication
- more complex software lifecycle requirements

Modern vehicles can use both.

---

# Expert Interview Cheat Sheet

## 1. Scheduling theory

```text
Liu & Layland
    ↓
RM fixed priority
EDF dynamic priority
    ↓
Utilization tests
    ↓
Exact response-time analysis
    ↓
Blocking / jitter / cache / overhead
```

## 2. Key equations

### Utilization

\[
U = \sum_i \frac{C_i}{T_i}
\]

### Liu-Layland RM sufficient bound

\[
U \le n(2^{1/n}-1)
\]

### Hyperbolic bound

\[
\prod_i(1+U_i)\le2
\]

### Fixed-priority RTA

\[
R_i =
C_i + B_i +
\sum_{j\in hp(i)}
\left\lceil\frac{R_i}{T_j}\right\rceil C_j
\]

with additional terms for jitter, release overhead, CRPD, etc., when required.

---

# 3. Multiprocessor vocabulary

```text
Partitioned
    = assign tasks to CPUs
    = no ordinary migration

Global
    = one global scheduling decision
    = tasks may migrate

Semi-partitioned
    = mostly fixed placement
    = controlled migration/splitting

Pfair
    = fine-grained proportional fairness
    = strong theory
    = higher scheduler overhead

G-EDF
    = earliest deadlines globally
    = good flexibility
    = not uniprocessor-optimal
```

---

# 4. Communication vocabulary

```text
CAN
    priority/arbitration
    event driven
    non-preemptive frames
    response-time analysis

FlexRay
    scheduled static segment
    TDMA
    synchronized cycle
    deterministic timing

TSN
    Ethernet + time-aware mechanisms
    802.1AS time synchronization
    802.1Qbv time-aware shaper
    engineered latency/jitter

ARINC 653
    time partitioning
    space partitioning
    major frame
    fault containment
```

---

# 5. Research-level vocabulary

```text
WCET
    absolute worst-case upper bound

pWCET
    probabilistic upper bound
    tail / exceedance probability

Cache
    high average performance
    harder timing analysis

CRPD
    cache damage caused by preemption

Scratchpad
    software-controlled
    predictable
    explicit management

Virtualization
    isolation + consolidation
    but adds timing/interference layers

CPS
    physical process
      ↕
    computation + communication + control

AUTOSAR Adaptive
    high-performance automotive computing
    POSIX PSE51
    service-oriented communication
    multicore / distributed applications
```

---

# Senior-level mistakes to avoid

## Mistake 1: “69.3% means RM cannot use more than 69.3% CPU.”

Wrong.

It is a sufficient bound, not the maximum achievable utilization.

---

## Mistake 2: “EDF is optimal everywhere.”

Wrong.

The classical optimality result is for **preemptive uniprocessor scheduling under specific assumptions**.

---

## Mistake 3: “Priority inheritance removes blocking.”

Wrong.

It controls priority inversion; it does not remove the resource critical section itself.

---

## Mistake 4: “Global EDF is the multiprocessor version of optimal EDF.”

Wrong.

Multiprocessor scheduling has fundamentally different complexity and interference properties.

---

## Mistake 5: “CAN is deterministic because it has priorities.”

Too simplistic.

CAN has analyzable bounded response time, but waiting depends on arbitration and blocking by an already-transmitting lower-priority frame.

---

## Mistake 6: “TSN means Ethernet is now automatically deterministic.”

Wrong.

Determinism comes from the engineered combination of time synchronization, scheduling, shaping, resource configuration, and bounded interference.

---

## Mistake 7: “Maximum observed execution time = WCET.”

Wrong.

A finite test campaign cannot prove that no longer execution path exists.

---

## Mistake 8: “Cache makes the system unpredictable.”

Too extreme.

Cache makes timing **harder to analyze**, not necessarily impossible to bound.

---

## Mistake 9: “Scratchpad is faster than cache.”

The real advantage is **predictability and software control**, not universally higher average speed.

---

## Mistake 10: “AUTOSAR Adaptive is an RTOS.”

Not exactly.

It is a platform architecture over a POSIX-based execution environment.

---

# How to sound like a 10-year embedded engineer

For difficult questions, avoid giving only textbook definitions.

Use this pattern:

> “The classical theorem says X under assumptions A, B and C. In an actual embedded product, I would additionally account for D, E and F.”

Example:

> “Classical EDF is optimal on a preemptive uniprocessor. But in my product I would not stop there; I would account for interrupt latency, blocking, cache behavior, timer resolution and execution-time margin before declaring the system schedulable.”

That one sentence immediately moves the answer from:

```text
student answer
```

to:

```text
experienced embedded engineer answer
```

---

# Reference Notes

1. C. L. Liu and J. W. Layland, “Scheduling Algorithms for Multiprogramming in a Hard-Real-Time Environment,” Journal of the ACM, 1973. DOI: `10.1145/321738.321743`.
2. E. Bini, G. C. Buttazzo and G. M. Buttazzo-related work on hyperbolic schedulability bounds / schedulability-test analysis.
3. J. H. Anderson and A. Srinivasan, work on Pfair / ERfair multiprocessor scheduling.
4. K. Tindell, A. Burns, A. J. Wellings, “Calculating Controller Area Network (CAN) Message Response Times,” Control Engineering Practice, 1995. DOI: `10.1016/0967-0661(95)00112-8`.
5. IEEE 802.1Qbv, “Enhancements for Scheduled Traffic,” IEEE 802.1 Working Group.
6. FAA material describing ARINC 653 partition scheduling and partition isolation.
7. AUTOSAR Adaptive Platform R25-11 documentation, including Operating System Interface / POSIX PSE51 and `ara::com`.
8. Research literature on cache-related preemption delay and WCET-aware scratchpad memory.
9. Research literature on probabilistic WCET / MBPTA and Extreme Value Theory.

---

# Final 30-second revision

If the interviewer asks these topics rapidly, remember:

```text
Liu-Layland
    -> RM bound + EDF foundation

Hyperbolic
    -> product(1 + Ui) <= 2

Blocking
    -> add Bi to RTA

Feasible
    -> any valid schedule exists

Optimal
    -> succeeds whenever any schedule can

Multiprocessor
    -> migration + load balancing + cache/interference
    -> Dhall effect
    -> partitioned/global/semi-partitioned

Pfair
    -> proportional service in fine-grained quanta

G-EDF
    -> global earliest deadlines
    -> not uniprocessor-optimal

CAN
    -> ID = priority
    -> non-preemptive
    -> blocking + higher-priority interference

FlexRay
    -> TDMA/static slots
    -> predictable cycle

TSN
    -> Ethernet + synchronized scheduled traffic
    -> 802.1Qbv TAS

ARINC 653
    -> temporal + spatial partitioning

CPS
    -> timing is part of physical correctness

pWCET
    -> probabilistic tail bound

Cache
    -> performance vs predictability
    -> CRPD

Scratchpad
    -> software-managed predictable memory

Virtualization
    -> consolidation + isolation
    -> extra timing/interference layer

AUTOSAR Adaptive
    -> POSIX PSE51
    -> services
    -> ara::com
    -> high-performance automotive ECUs
```
