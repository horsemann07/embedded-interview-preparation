# RTOS Concepts Interview Questions
> Generic RTOS — No specific OS  
> M.Tech Graduate + 10 Years Experience  
> Basic → Intermediate → Advanced → Expert → Research Level

---

## SUMMARY TABLE

| Topic | Basic | Intermediate | Advanced | Expert |
|-------|-------|-------------|----------|--------|
| RTOS fundamentals | ★★★★★ | ★★★★★ | ★★★☆☆ | ★★☆☆☆ |
| Task & scheduling | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| Synchronization | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| IPC | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Memory management | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★★ |
| Multicore | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| Safety/Certification | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| Schedulability theory | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ |
| Formal verification | ★☆☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ |
| Real-time comms | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ |

---

## BASIC LEVEL

### Definition & Fundamentals

1. What is an RTOS?
   - Follow-up: What does "real-time" actually mean?
   - Follow-up: Is determinism same as speed?
   - Follow-up: Give 5 examples of RTOS used in industry.

2. Hard real-time vs Soft real-time vs Firm real-time?
   - Follow-up: Give one real-world example of each.
   - Follow-up: Which category does an airbag controller fall into?
   - Follow-up: Which category does video streaming fall into?

3. RTOS vs GPOS (General Purpose OS)?

   | Feature | RTOS | GPOS |
   |---------|------|------|
   | Determinism | Yes | No |
   | Latency | Bounded | Unbounded |
   | Scheduling | Priority-based | Fairness-based |
   | Footprint | Small | Large |
   | Examples | FreeRTOS, VxWorks | Linux, Windows |

   - Follow-up: Is Linux an RTOS?
   - Follow-up: What is PREEMPT_RT patch for Linux?
   - Follow-up: What is Xenomai?

4. What is determinism in RTOS?
   - Follow-up: Difference between deterministic and predictable.
   - Follow-up: What is Worst Case Execution Time (WCET)?
   - Follow-up: How is WCET measured and analyzed?
   - Follow-up: What tools are used for WCET analysis? (aiT, Bound-T)

5. What is jitter?
   - Follow-up: What causes jitter?
   - Follow-up: How do you measure jitter?
   - Follow-up: How do you minimize jitter in ISR latency?

6. What are the main components of an RTOS?
   - Kernel / Scheduler
   - Task Manager
   - Interrupt Handler
   - Memory Manager
   - IPC & Synchronization
   - Timer Services
   - Follow-up: Which component is most critical for real-time guarantees?

7. What is a tick in RTOS?
   - Follow-up: What is tick period and tick rate?
   - Follow-up: Tradeoffs of high tick rate vs low tick rate?
   - Follow-up: What is tickless RTOS?

---

### Tasks

8. What is a task in RTOS?
   - Follow-up: Difference between task, thread, process in RTOS context.
   - Follow-up: What is a Task Control Block (TCB)?
   - Follow-up: What information does TCB hold?

9. What are task states? Draw and explain the state machine.
   ```
   Created → Ready → Running → Blocked → Suspended → Deleted
   ```
   - Follow-up: What causes transition from Running → Blocked?
   - Follow-up: What causes transition from Blocked → Ready?
   - Follow-up: Can a task be in both Blocked and Suspended?

10. What is task priority?
    - Follow-up: Static priority vs dynamic priority.
    - Follow-up: What happens when two tasks have equal priority?
    - Follow-up: What is starvation and how to prevent it?

11. What is task stack?
    - Follow-up: What is stored on task stack?
    - Follow-up: How do you calculate required stack size?
    - Follow-up: What is stack watermark?

---

### Context Switching

12. What is context switching?
    - Follow-up: What is saved and restored in a context switch?
    - Follow-up: What is context switch overhead?
    - Follow-up: How does hardware help context switching? (ARM: PendSV, hardware stacking)

13. What triggers a context switch?
    - Tick interrupt (time slice expired)
    - Higher priority task becomes ready
    - Running task blocks on IPC/delay
    - Running task explicitly yields
    - Follow-up: What is voluntary vs involuntary context switch?

14. Preemptive vs Cooperative scheduling?
    - Follow-up: Pros and cons of each.
    - Follow-up: Which is safer for safety-critical embedded?
    - Follow-up: Can you mix both in one RTOS?

---

## INTERMEDIATE LEVEL

### Scheduling Algorithms

1. Priority-Based Preemptive Scheduling
   - Follow-up: What is the highest priority task always guaranteed?
   - Follow-up: What is the problem with pure priority scheduling?

2. Round Robin Scheduling
   - Follow-up: When is round robin applied in RTOS? (equal priority tasks)
   - Follow-up: What is time quantum?

3. Rate Monotonic Scheduling (RMS)
   - Follow-up: What is the Liu & Layland utilization bound?
   - Formula: `U = sum(C_i / T_i) <= n(2^(1/n) - 1)`
   - For n → ∞: `U <= ln(2) ≈ 69.3%`
   - Follow-up: Why is RMS optimal for fixed-priority static systems?
   - Follow-up: What happens when utilization exceeds the bound?

4. Earliest Deadline First (EDF)
   - Follow-up: Is EDF optimal? (Yes — 100% utilization theoretically achievable)
   - Follow-up: Why is EDF rarely used in embedded RTOS?
   - Follow-up: What is the problem with EDF under overload?

5. Least Laxity First (LLF)
   - Follow-up: What is laxity (slack time)?
   - Follow-up: LLF vs EDF — comparison.

6. Deadline Monotonic Scheduling (DMS)
   - Follow-up: When is DMS better than RMS?

7. What is schedulability analysis?
   - Follow-up: Utilization test vs Response Time Analysis (RTA).
   - Follow-up: Which is more accurate?

8. Response Time Analysis
   ```
   R_i = C_i + sum over j∈hp(i) [ ceil(R_i / T_j) * C_j ]
   Solved iteratively until R_i converges or exceeds D_i
   ```
   - Follow-up: What is `hp(i)`? (set of tasks with higher priority than i)
   - Follow-up: What is blocking time term? (from lower priority tasks holding resources)

---

### Priority Inversion

9. What is priority inversion?
   - Classic scenario:
   ```
   L acquires mutex M
   H preempts, tries to acquire M → blocks
   M preempts L (medium priority)
   H waits indefinitely → priority inversion
   ```
   - Follow-up: Mars Pathfinder 1997 — explain the real incident.

10. Priority Inheritance Protocol
    - Follow-up: How does it work step by step?
    - Follow-up: What are its limitations? (chained blocking, complex analysis)

11. Priority Ceiling Protocol (PCP)
    - Follow-up: What is the ceiling of a resource?
    - Follow-up: How does PCP prevent deadlock?
    - Follow-up: Difference between PCP and Immediate Priority Ceiling Protocol (IPCP).

12. Which protocol is used in AUTOSAR OS and why?
    - Follow-up: What is OSEK priority ceiling?

---

### Synchronization Primitives

13. Binary Semaphore
    - Follow-up: Use case: ISR signals a task.
    - Follow-up: What is the problem of using binary semaphore for mutual exclusion?

14. Counting Semaphore
    - Follow-up: Use case: resource pool management.
    - Follow-up: What is semaphore count ceiling?

15. Mutex
    - Follow-up: Difference between mutex and binary semaphore.
    - Follow-up: Does mutex support priority inheritance?
    - Follow-up: What is a recursive mutex?

16. Spinlock
    - Follow-up: When to use spinlock vs semaphore?
    - Follow-up: Why is spinlock wasteful on single-core?
    - Follow-up: When is spinlock correct on multicore?

17. Deadlock
    - Follow-up: Coffman's four necessary conditions.
    - Follow-up: Prevention vs Avoidance vs Detection vs Recovery.
    - Follow-up: How does resource ordering prevent deadlock?

---

### IPC Mechanisms

18. Message Queue
    - Follow-up: Fixed-size vs variable-size messages.
    - Follow-up: Blocking vs non-blocking send/receive.
    - Follow-up: What happens when queue is full?
    - Follow-up: Queue vs shared memory — tradeoffs.

19. Mailbox
    - Follow-up: Difference between mailbox and message queue.

20. Event Flags / Event Groups
    - Follow-up: AND-wait vs OR-wait semantics.
    - Follow-up: Can multiple tasks wait on same event flag?

21. Pipes
    - Follow-up: Byte stream vs message-based pipes.

22. Signals
    - Follow-up: POSIX signals in real-time context.

23. Shared Memory
    - Follow-up: Why shared memory needs synchronization.
    - Follow-up: When is shared memory + mutex better than message queue?

---

### Memory Management

24. Static vs Dynamic memory allocation in RTOS
    - Follow-up: Why is dynamic allocation avoided in safety-critical RTOS?
    - Follow-up: MISRA-C Rule 21.3 — no dynamic memory.

25. Memory Pool Allocator
    - Follow-up: Fixed-size block pool vs variable-size pool.
    - Follow-up: How does it prevent fragmentation?
    - Follow-up: Implement a simple fixed-size memory pool.

26. Memory Fragmentation
    - Follow-up: Internal vs external fragmentation.
    - Follow-up: How do you detect and measure fragmentation?

27. Memory Protection Unit (MPU)
    - Follow-up: How does RTOS use MPU for task isolation?
    - Follow-up: What faults does MPU generate on violation?

---

### Interrupts in RTOS

28. Relationship between ISR and RTOS tasks
    - Follow-up: Can RTOS API be called from ISR?
    - Follow-up: What is deferred interrupt processing?

29. Interrupt Latency
    - Follow-up: Components of interrupt latency in RTOS.
    - Follow-up: How does RTOS critical section affect interrupt latency?

30. Interrupt Nesting
    - Follow-up: How does priority affect ISR nesting?
    - Follow-up: What is non-maskable interrupt (NMI) and its RTOS impact?

---

## ADVANCED LEVEL

### Kernel Internals

1. How does a preemptive RTOS kernel work internally?
   - Ready list structure (priority bitmap / sorted list)
   - Scheduler invocation points
   - Context switch implementation
   - Follow-up: How does O(1) scheduler work?

2. What is a microkernel vs monolithic kernel in RTOS?
   - Follow-up: Examples: QNX (microkernel), VxWorks (monolithic).
   - Follow-up: Tradeoffs in embedded context.

3. What is a nanokernel?
   - Follow-up: Examples: INTEGRITY RTOS.

4. What is an exokernel?
   - Follow-up: Research context — how it differs from microkernel.

5. What is a hypervisor in embedded RTOS?
   - Type 1 (bare metal) vs Type 2.
   - Follow-up: PikeOS, INTEGRITY, Xen on ARM, Jailhouse.
   - Follow-up: How does hypervisor provide temporal and spatial isolation?

---

### Time Management

6. What is a monotonic clock? Why is it important in RTOS?
7. What is clock drift and how is it compensated?
8. What is time synchronization in distributed real-time systems?
   - Follow-up: IEEE 1588 Precision Time Protocol (PTP).
   - Follow-up: Time-Triggered Protocol (TTP).
9. What is Time-Division Multiple Access (TDMA) scheduling?
10. What is Time-Triggered Architecture (TTA)?
    - Follow-up: FlexRay, TT-Ethernet in automotive RTOS context.

---

### Mixed Criticality Systems

11. What is a Mixed Criticality System (MCS)?
    - Follow-up: Vestal model.
    - Follow-up: High-criticality vs low-criticality tasks.
    - Follow-up: What happens when high-criticality mode is triggered?

12. How do you achieve temporal isolation between criticality levels?
    - Follow-up: Hypervisor partitioning.
    - Follow-up: Time partitioning (time windows per partition).

13. What is AUTOSAR OS mixed criticality support?

---

### Multicore RTOS

14. SMP RTOS (Symmetric Multiprocessing)
    - Follow-up: How does scheduler distribute tasks across cores?
    - Follow-up: Global queue vs per-core queue scheduling.
    - Follow-up: Cache coherency problem.
    - Follow-up: False sharing — what it is and how to prevent.

15. AMP RTOS (Asymmetric Multiprocessing)
    - Follow-up: Each core runs its own RTOS instance.
    - Follow-up: Use case: Cortex-M + Cortex-A on same SoC.
    - Follow-up: Inter-core communication via shared memory + mailbox.

16. Task Affinity
    - Follow-up: Why pin safety-critical tasks to dedicated core?
    - Follow-up: How does task migration affect cache performance?

17. Inter-core Synchronization
    - Follow-up: Hardware spinlock on multicore SoC.
    - Follow-up: OpenAMP framework.
    - Follow-up: RPMsg protocol.

---

### Power Management

18. How does RTOS support power management?
    - Follow-up: Idle task and sleep modes.
    - Follow-up: Tickless idle — how does it work?
    - Follow-up: Dynamic Voltage and Frequency Scaling (DVFS).
    - Follow-up: How to wake from deep sleep and resume RTOS correctly?

19. Tradeoff between tick rate and power consumption.

---

### Safety & Certification

20. What is IEC 61508?
    - Follow-up: Safety Integrity Levels (SIL 1–4).
    - Follow-up: What does SIL certification mean for RTOS?

21. What is ISO 26262?
    - Follow-up: Automotive Safety Integrity Levels (ASIL A–D).
    - Follow-up: ASIL decomposition.

22. What is DO-178C?
    - Follow-up: Design Assurance Levels (DAL A–E) for aviation software.

23. What is IEC 62443?
    - Follow-up: Industrial cybersecurity standard.

24. What makes an RTOS certifiable?
    - Follow-up: SafeRTOS vs FreeRTOS.
    - Follow-up: Code coverage requirements (MC/DC for DO-178C Level A).

25. What is formal verification?
    - Follow-up: seL4 microkernel — world's first formally verified OS kernel.
    - Follow-up: Why is formal verification rare but critical in aerospace?

---

### Debugging Real-Time Systems

26. Challenges in debugging RTOS applications.
    - Heisenberg effect — debugger changes timing.
    - Race conditions disappear under debug.
    - Stack overflow hard to detect post-mortem.

27. Non-intrusive debugging techniques.
    - Trace buffers (circular log in RAM).
    - JTAG trace (ETM — Embedded Trace Macrocell).
    - Logic analyzer on GPIO toggle.
    - Follow-up: SEGGER SystemView — how does it instrument RTOS?

28. How do you test a real-time system?
    - Timing analysis under worst-case load.
    - Stress testing: maximum task load injection.
    - Fault injection testing.
    - Hardware-in-the-loop (HIL) testing.

---

## EXPERT LEVEL

### Schedulability Theory

1. Liu & Layland 1973 paper — what did it prove?
2. What is hyperbolic bound? (tighter than utilization bound)
3. What is blocking analysis in fixed-priority scheduling?
   - Follow-up: How is blocking term added to RTA?
4. What is the difference between feasibility and schedulability?
5. What is an optimal scheduler?
   - Follow-up: EDF is optimal for preemptive uniprocessor — proof sketch.
6. What is the multiprocessor scheduling problem? Why is it harder?
   - Follow-up: Dhall effect.
   - Follow-up: Partitioned vs global vs semi-partitioned scheduling.
7. What is Pfair scheduling?
8. What is G-EDF and its limitations on multiprocessor?

---

### Real-Time Communication

9. What is CAN bus real-time scheduling?
   - Follow-up: CAN message priority = frame ID.
   - Follow-up: CAN scheduling analysis (Tindell et al.).
10. What is FlexRay and why is it more deterministic than CAN?
11. What is Time-Sensitive Networking (TSN)?
    - Follow-up: IEEE 802.1Qbv time-aware shaper.
12. What is ARINC 653?
    - Follow-up: Partitioned scheduling in avionics.
    - Follow-up: Time and space partitioning.

---

### Research-Level Questions

13. What is the Cyber-Physical System (CPS) and its RTOS challenges?
14. What is probabilistic WCET (pWCET)?
    - Follow-up: Why is deterministic WCET hard on modern out-of-order CPUs?
15. What is the impact of hardware caches on real-time analysis?
    - Follow-up: Cache-Related Preemption Delay (CRPD).
16. What is scratchpad memory vs cache in real-time systems?
17. What is RTOS virtualization overhead analysis?
18. What is the role of RTOS in autonomous vehicles (AUTOSAR Adaptive)?
    - Follow-up: POSIX PSE51 profile.
    - Follow-up: Service-oriented architecture on RTOS.

---
