# MAANG RTOS Interview Questions: Beginner to Advanced

**Target roles:** Embedded Software Engineer, Firmware Engineer, RTOS Engineer, Senior Firmware Engineer  
**Companies to prepare for:** Google, Amazon, Microsoft, and similar product/silicon companies  
**Format:** Interview-style questions, follow-up probes, coding exercises, and real-world debugging scenarios

> **Important:** This is a curated practice bank, not an official question paper and not a claim that Google, Amazon, or Microsoft ask these exact questions. Interview loops vary by team, level, location, and job description. The questions reflect common embedded/firmware interview themes: C/C++, operating systems, scheduling, synchronization, interrupts, debugging, design, and coding. Current Google firmware/embedded role descriptions emphasize C/C++, embedded systems, kernel/device-driver work, debugging, and low-level silicon or platform enablement; reported firmware interviews also include coding and design. See the references at the end.

---

## How to use this question bank

For every question, practise answering in this order:

1. **Define the concept** clearly in 1–2 sentences.
2. **Explain what happens internally**: task state, scheduler decision, interrupt context, memory, or timing.
3. **Give a concrete embedded example** from a sensor, communication driver, motor-control loop, networking stack, or product firmware.
4. **Discuss trade-offs and failure modes**: latency, CPU usage, memory, priority inversion, race conditions, missed deadlines, and testability.
5. **Expect follow-ups.** An interviewer may change one requirement and ask you to revise the design.

### Difficulty guide

- **Beginner:** Explain terms and predict simple behaviour.
- **Intermediate:** Apply the concepts to code and small systems.
- **Advanced:** Analyse timing, concurrency, failure modes, trade-offs, and production debugging.
- **Senior/Staff-style:** Make a defensible design, state assumptions, identify risks, and explain how you would validate it.

# 1. Beginner — RTOS Fundamentals

1. What is an RTOS, and what problem does it solve in an embedded system?
2. What is the difference between a general-purpose operating system and a real-time operating system?
3. Does “real-time” mean “very fast”? Explain the difference between speed and meeting a deadline.
4. What is a hard real-time system? Give one example.
5. What is a soft real-time system? Give one example.
6. What is a firm real-time system, and how is it different from a hard real-time system?
7. What is a task or thread in an RTOS?
8. What is the difference between a task, a process, and a thread?
9. What are the common states of an RTOS task? Explain Ready, Running, Blocked/Waiting, and Suspended.
10. What causes a task to move from Running to Blocked?
11. What is a scheduler?
12. What is a context switch, and what information usually needs to be saved and restored?
13. What is preemptive scheduling?
14. What is cooperative scheduling?
15. Compare preemptive and cooperative scheduling. What are the main trade-offs?
16. What is a task priority? Does a larger numeric value always mean a higher priority?
17. What is a time slice or time quantum?
18. What is the difference between a periodic task and an aperiodic task?
19. What is a sporadic task, and why might its minimum inter-arrival time matter?
20. What are task period, execution time, response time, latency, and deadline?
21. What is jitter? Give an example involving periodic sensor sampling.
22. What is determinism, and why is it important in real-time software?
23. What is the difference between foreground/background super-loop firmware and RTOS-based firmware?
24. When would you choose bare-metal firmware instead of an RTOS?
25. What factors would you consider before introducing an RTOS into an existing firmware project?

# 2. Beginner to Intermediate — Scheduling and Task Behaviour

26. In a fixed-priority preemptive RTOS, what happens when a higher-priority task becomes Ready while a lower-priority task is Running?
27. Two tasks have the same priority. How might the scheduler choose between them?
28. A high-priority task never blocks or yields. What can happen to lower-priority tasks?
29. What is starvation? How can it happen in a priority-based scheduler?
30. Can a low-priority task ever delay a high-priority task? Explain at least two ways.
31. What is the difference between task delay and waiting for an event or semaphore?
32. Why is a relative-delay loop sometimes unsuitable for a periodic task?
33. How would you implement a task that must execute every 10 ms without accumulating timing drift?
34. What happens if a task's worst-case execution time exceeds its period?
35. What is worst-case execution time (WCET), and why is average execution time not enough for real-time analysis?
36. How do interrupt execution time and critical sections affect task response time?
37. What is rate-monotonic scheduling (RMS)? What assumption does its classic analysis make about priorities?
38. What is earliest-deadline-first (EDF) scheduling?
39. Compare fixed-priority scheduling with EDF. What are the practical trade-offs on a small MCU?
40. What is CPU utilisation for a set of periodic tasks? How would you calculate it?
41. If three periodic tasks execute for 1 ms, 2 ms, and 3 ms with periods of 5 ms, 10 ms, and 20 ms, respectively, what is their total nominal CPU utilisation?
42. How would you investigate occasional deadline misses when average CPU utilisation is only 35%?
43. What is priority-based round-robin scheduling, and when can it be useful?
44. Why should a task avoid long busy-wait loops?
45. How can logging or diagnostic output unexpectedly change a task's timing behaviour?

# 3. Intermediate — Synchronisation and Inter-Task Communication

46. What is a race condition? Give an embedded example involving a shared counter.
47. What is a critical section? What makes a critical section too long?
48. What is a mutex, and what problem is it designed to solve?
49. What is a semaphore? Explain binary and counting semaphores.
50. Compare a mutex, binary semaphore, counting semaphore, event flag, and message queue.
51. When should a mutex be used instead of a binary semaphore?
52. Why should a mutex generally be owned and released by the same task?
53. What is priority inheritance? Which failure mode does it mitigate?
54. What is priority inversion? Explain it with three tasks of high, medium, and low priority.
55. Does priority inheritance eliminate every possible priority inversion or every deadlock? Explain.
56. What is priority ceiling/protocol-based resource access, at a high level?
57. What is a deadlock? State the conditions commonly associated with deadlock.
58. How would you prevent deadlocks when two tasks need two shared resources?
59. What is a livelock, and how is it different from deadlock?
60. What is a message queue? When is message passing preferable to sharing a global structure?
61. How do queue depth and message size affect RAM consumption and latency?
62. What should a producer do if a queue is full?
63. How would you design a latest-value mailbox for sensor data so stale readings do not build up in a queue?
64. What is a one-to-many event notification? Which RTOS primitive might support it?
65. How can you safely transfer ownership of a buffer between a DMA driver and an application task?
66. Why can `volatile` not, by itself, make a shared variable thread-safe?
67. What is an atomic operation? When might atomics be preferable to a mutex?
68. Is disabling interrupts a universal way to protect shared data? Explain its limitations.
69. What happens if a task waits forever on a mutex while holding another mutex?
70. How would you design a lock-ordering rule for a firmware team?

# 4. Intermediate — Interrupts, ISRs, and Deferred Work

71. What is an interrupt, and what is the typical path from a peripheral event to a task handling the event?
72. What work should normally be done inside an ISR, and what work should be deferred to a task?
73. Why should an ISR avoid blocking calls, sleeping, and long loops?
74. What does it mean for an RTOS API to be “ISR-safe” or callable from interrupt context?
75. Why do some RTOS APIs have separate task-context and ISR-context variants?
76. How would you wake a high-priority task from an ISR?
77. What is interrupt latency? Which hardware and software factors influence it?
78. What is interrupt response time, and how does it differ from interrupt latency?
79. What are interrupt priority levels, and how can interrupt nesting complicate response-time analysis?
80. Explain the difference between interrupt priority and RTOS task priority.
81. What problems can occur if an interrupt has a priority that is incompatible with the RTOS's syscall rules?
82. How would you avoid losing events if a peripheral interrupt fires faster than the consumer task can process events?
83. Compare polling, interrupts, and DMA for a high-rate UART receive path.
84. When might a level-triggered interrupt repeatedly fire? How would you investigate it?
85. How would you debug a system that hard-faults only when a particular interrupt occurs?
86. What is a re-entrant function? Why is re-entrancy relevant to ISRs and task concurrency?
87. What is interrupt storming, and how can it make the whole system appear frozen?
88. How would you measure ISR duration and interrupt-to-task wake-up latency on real hardware?

# 5. Intermediate — Memory, Stacks, and Resource Limits

89. What is each task's stack used for?
90. What is the difference between stack and heap memory in embedded firmware?
91. Why does each RTOS task need its own stack?
92. What can cause a task stack overflow? What symptoms might appear before a crash?
93. How would you estimate a safe task stack size instead of choosing an arbitrary number?
94. What are stack high-water marks, guard regions, and stack canaries?
95. Why can recursion be risky in a constrained RTOS task?
96. What is heap fragmentation? Why can a long-running product fail even when enough free heap appears to remain?
97. What are the advantages and disadvantages of dynamic memory allocation after the scheduler starts?
98. Compare fixed-size memory pools, object pools, and general-purpose heap allocation.
99. How can passing a pointer to a task create a use-after-free or lifetime bug?
100. What is false sharing, and is it usually a primary concern on small MCUs? Explain when it could matter.
101. What is cache coherency, and why can DMA expose cache-coherency bugs on some SoCs?
102. How would you debug memory corruption that appears to originate in a task unrelated to the actual bug?
103. What is the difference between memory safety and thread safety?
104. How can a task's local variable become invalid when its address is passed to another task?

# 6. Advanced — Real-Time Analysis and Scheduling Design

105. How would you determine whether a proposed task set can meet its deadlines?
106. Explain the components of response-time analysis for a fixed-priority task, including execution time, higher-priority interference, and blocking.
107. How do blocking time and non-preemptive sections affect schedulability?
108. Why can an interrupt-driven design still fail to meet a task deadline?
109. A control task runs every 1 ms, a communication task every 5 ms, and a logging task every 20 ms. How would you assign priorities and validate the assignment?
110. A high-priority task meets its own deadline but causes two lower-priority tasks to miss theirs. How would you diagnose and resolve the conflict?
111. How would you handle an occasional long operation that does not fit inside the normal periodic task's timing budget?
112. What is a sporadic server or budget-based execution model, and what problem does it help control?
113. How do release jitter and execution-time variation affect response-time analysis?
114. What is overload behaviour in a real-time system? How should a robust design degrade under overload?
115. How would you prioritise control, communication, diagnostics, and logging in a resource-constrained device?
116. When is dropping old data safer than processing every queued sample?
117. What is temporal isolation, and how can a design prevent one subsystem from consuming all CPU time?
118. How would you prove or provide evidence that a deadline is met across worst-case operating conditions?

# 7. Advanced — FreeRTOS and RTOS Implementation Details

> These questions use FreeRTOS-style concepts where useful. Translate to the APIs and configuration of the RTOS used by the target role; names and semantics vary between kernels.

119. Explain the relationship between `xTaskCreate`, task priority, task stack depth, and task parameters in FreeRTOS.
120. What is the difference between `vTaskDelay()` and `vTaskDelayUntil()`? When would you choose each?
121. What does `xHigherPriorityTaskWoken` indicate in common FreeRTOS ISR APIs, and why may a yield be requested before leaving the ISR?
122. Why is `FromISR` required for certain FreeRTOS APIs when called from an ISR?
123. What is the difference between a binary semaphore and a direct-to-task notification in FreeRTOS? What trade-offs matter?
124. How do FreeRTOS mutexes implement priority inheritance, and what limitations should a developer understand?
125. What is the idle task responsible for? What can go wrong if the idle task is starved?
126. What is the tick interrupt used for? What changes when tickless idle is enabled?
127. What is the trade-off between tick frequency, timing resolution, interrupt overhead, and power consumption?
128. How would you configure and use stack-overflow checking and malloc-failure hooks in FreeRTOS?
129. What is a FreeRTOS software timer, and why should a timer callback not perform a long blocking operation?
130. What role does the timer service task play, and how can its priority affect timer callback latency?
131. How would you find out why a task is not running even though it was created successfully?
132. How would you distinguish a scheduler-start problem from a task blocked forever or a task starved by higher priorities?
133. What are the implications of using floating-point instructions in tasks on an MCU with a floating-point unit?
134. Which FreeRTOS configuration options would you inspect first when investigating unexpected memory usage or scheduling behaviour?

# 8. Advanced — Embedded C/C++ Questions That Commonly Pair with RTOS

135. Explain why `volatile` is important for memory-mapped registers but insufficient for inter-task synchronisation.
136. What can go wrong if an ISR and a task share a non-atomic multi-byte variable on a processor that cannot access it atomically?
137. What is undefined behaviour in C, and how can it undermine low-level firmware assumptions?
138. Explain `static`, `const`, `volatile`, and `extern` using embedded examples.
139. What are alignment and padding in C structures? How can they affect memory-mapped data, communication packets, and DMA buffers?
140. What is the difference between a data race and a logical race condition?
141. What is the difference between atomicity, visibility, and ordering in concurrent code?
142. When are memory barriers or compiler barriers needed in low-level firmware?
143. What is a function pointer, and how could it support a driver callback or hardware abstraction layer?
144. How would you design a thread-safe ring buffer in C for one producer and one consumer?
145. What assumptions must hold for a lock-free single-producer/single-consumer ring buffer to be correct?
146. How would you safely share a 64-bit timestamp between a 32-bit MCU ISR and a task?
147. Why can a `printf()` call be unsafe, slow, or deadlock-prone in an ISR or multithreaded firmware?
148. What are the trade-offs between RAII-based locking in C++ and explicit lock/unlock APIs in C?
149. How can exceptions, static initialisation, and heap allocation affect deterministic behaviour in embedded C++?
150. How would you unit-test a scheduler-independent state machine that normally runs inside an RTOS task?

# 9. Scenario-Based Questions — Debugging Real Product Failures

For each scenario, explain your **first measurements**, **hypotheses**, **instrumentation**, **fix options**, and **regression tests**. Do not jump straight to changing task priorities.

151. A device works for hours, then freezes. The watchdog resets it, but the logs show no obvious fault. How would you isolate the root cause?
152. A 1 ms control task occasionally takes 2.5 ms to complete. How would you identify where the extra time is spent?
153. UART data is lost only when BLE traffic increases. How would you investigate task priorities, queueing, ISR load, DMA, and buffer ownership?
154. A high-priority task misses deadlines only when debug logging is enabled. What are your leading hypotheses?
155. A low-priority task holds a mutex, a medium-priority task runs continuously, and a high-priority task blocks on that mutex. Explain the schedule and your fix.
156. Two tasks occasionally corrupt the same configuration structure. How would you find every access path and make the update safe?
157. A task's stack high-water mark looks healthy during normal use, but a rare command causes a crash. What would you inspect?
158. A timer callback sometimes runs 20 ms late. How would you separate tick delay, timer-service-task starvation, and callback execution time?
159. The system reports a queue full condition once a day. How would you decide whether to increase queue depth, improve service rate, drop stale messages, or redesign the pipeline?
160. After enabling DMA, a peripheral driver reads stale data from a buffer. What cache, alignment, ownership, and memory-ordering questions would you ask?
161. A device misses deadlines only at high temperature or low supply voltage. How could hardware timing, clock changes, retries, and WCET be involved?
162. A watchdog fires during flash erase/write operations. How would you reason about interrupt masking, execution stalls, scheduling, and watchdog design?
163. After adding a new high-priority task, system latency becomes worse despite low CPU utilisation. What could explain this?
164. A deadlock occurs once in several days. What lock-order, timeout, trace, and fault-capture measures would you add?
165. The device sometimes fails to enter low-power mode. How would you identify tasks, timers, interrupts, DMA, and peripheral states that prevent sleep?
166. An ISR is short in source code but takes unexpectedly long on target. What hardware- and compiler-level factors could explain it?
167. A binary semaphore occasionally loses an event when two events occur close together. Is the primitive appropriate? What alternatives would you consider?
168. A firmware update introduces a timing regression but all functional tests pass. How would you build a timing-regression test strategy?

# 10. Design Questions — Typical Senior Firmware/RTOS Discussion

169. Design an RTOS architecture for a battery-powered sensor that samples every 10 ms, logs data, and transmits over BLE.
170. Design a UART receive subsystem that handles bursts without losing frames while keeping ISR execution short.
171. Design a driver API that can be called from both task context and interrupt context. What should the API contract say?
172. Design a multi-producer event pipeline that delivers events to one consumer in priority order.
173. Design a watchdog strategy for a multi-task system. How can it detect a task that is alive but making no progress?
174. Design health monitoring for task stack usage, CPU load, queue depth, deadline misses, and fault history.
175. Design a safe shutdown/restart sequence when several tasks own peripherals and buffers.
176. How would you make a task restartable after a subsystem failure without rebooting the entire device?
177. How would you design a firmware architecture that supports both FreeRTOS on an MCU and Linux on a more powerful SoC?
178. Where would you draw the boundary between a hardware driver, BSP, RTOS abstraction, middleware, and application logic?
179. How would you structure a codebase so that drivers can be unit-tested on a host PC without real hardware?
180. How would you validate deterministic timing across multiple hardware revisions and compiler optimisation levels?
181. How should the design change if the system gains a second core or uses an AMP/SMP architecture?
182. What extra problems appear when tasks on different cores share data or use shared interrupts?
183. How would you make failure handling deterministic when a communication task crashes or stops responding?
184. How would you decide between polling, interrupt-driven I/O, DMA, and a dedicated worker task for a new peripheral?

# 11. Coding Exercises — Practise Writing C/C++ Without an IDE

For each exercise, clarify assumptions, write clean code, discuss complexity, and identify concurrency or timing limitations.

185. Write a function that sets, clears, toggles, and reads a selected bit in a 32-bit register value.
186. Implement a fixed-size circular buffer in C with `push`, `pop`, `is_empty`, and `is_full` operations.
187. Extend the circular buffer to support one ISR producer and one task consumer. State the assumptions that make the implementation safe.
188. Write a function that calculates elapsed time correctly when a 32-bit tick counter wraps around.
189. Implement a periodic task loop that maintains a stable 10 ms period and reports deadline misses.
190. Write a bounded event queue using a static array, without dynamic memory allocation.
191. Implement a timeout-based wait loop using a monotonic tick source. Explain how it behaves when the counter wraps.
192. Write a producer-consumer example using a queue or semaphore. Identify all shared data and synchronisation points.
193. Implement a task-safe update for a configuration structure containing several related fields, ensuring a reader never observes a partially updated configuration.
194. Given a trace of task start/end times, calculate execution time, response time, jitter, and deadline misses.
195. Given a priority table and blocking intervals, identify which task can miss its deadline and explain why.
196. Review a code sample that uses `volatile` on a shared counter and explain why it can still contain a race.
197. Design a small state machine for a UART driver with Idle, Receiving, FrameReady, Error, and Recovery states.
198. Implement a watchdog heartbeat design in which each critical task reports progress and the supervisor checks stale heartbeats.
199. Write a memory-pool allocator for a fixed number of equal-sized objects and explain how double-free or invalid pointers would be handled.
200. Given a deadlock-prone two-lock example, rewrite it using a consistent lock order or a different ownership model.

# 12. Mock Interview Follow-Up Probes

These follow-ups test depth. Pick a question from an earlier section and answer these as if the interviewer is challenging your first solution.

201. What assumptions are you making, and which assumption would change your design the most?
202. Can you draw the task states and timeline before and after the event?
203. What is the worst-case behaviour, not just the normal behaviour?
204. Can you solve it without dynamic memory allocation?
205. What happens if the event occurs twice before the consumer runs?
206. What happens if the queue is full, the mutex is unavailable, or the interrupt arrives during a critical section?
207. How would your design behave if the clock rate changes during low-power operation?
208. Which metrics would you instrument in production, and how would you avoid the instrumentation changing the timing?
209. How would you reproduce this bug if it occurs only once per day on customer hardware?
210. What test would fail if your proposed fix were incorrect?
211. What trade-off did you choose, and what would make you choose the other option?
212. How would the design change on a multicore SoC, under Linux, or with a DMA-capable peripheral?

# 13. Company-Style Preparation Focus

These are **preparation emphases**, not guarantees about the content of a particular interview loop.

## Google-style embedded/firmware preparation

Prioritise strong C/C++, bit manipulation, data structures and algorithms, OS fundamentals, low-level debugging, and the ability to reason about kernel/driver or SoC interactions. Be ready to explain a design from hardware event to ISR, scheduling decision, data path, and end-to-end latency. Practise writing correct code without depending on an IDE.

## Amazon-style embedded/firmware preparation

Prioritise coding correctness, practical firmware design, debugging, system design, ownership of past projects, trade-offs, and clear examples of resolving difficult production issues. Practise explaining how you measured success, handled a failure, and prevented recurrence. Review the Leadership Principles and prepare genuine STAR-format examples.

## Microsoft-style embedded/firmware preparation

Prioritise C/C++ fundamentals, debugging and problem-solving, OS/threading concepts, system design, testing, collaboration, and the requirements of the target hardware/product team. Do not neglect general coding and data structures just because the job title says “embedded.”

## Senior-level expectation across all three

A senior candidate is usually expected to do more than define a semaphore or name an RTOS API. Explain why a design is correct, where it can fail, how to measure it, how to debug it on hardware, and how to test it so the problem does not return.

# 14. Suggested Preparation Order

- [ ] **Stage 1:** Questions 1–25 — RTOS fundamentals.
- [ ] **Stage 2:** Questions 26–70 — scheduling, IPC, synchronisation, and priority inversion.
- [ ] **Stage 3:** Questions 71–104 — interrupts, memory, stacks, and buffers.
- [ ] **Stage 4:** Questions 105–150 — real-time analysis, FreeRTOS, and embedded C/C++.
- [ ] **Stage 5:** Questions 151–184 — debugging scenarios and architecture design.
- [ ] **Stage 6:** Questions 185–200 — coding exercises.
- [ ] **Stage 7:** Questions 201–212 — follow-up probes and mock interviews.

For a senior firmware role, do not spend all your preparation time memorising definitions. Practise at least one scenario and one coding problem aloud every day, and be able to support each design decision with timing, memory, concurrency, and testing considerations.

# References and Scope Notes

The question bank is original practice material informed by common embedded interview topics and public role/interview descriptions. It is not an official company question list.

- [Google Careers — Software Engineer, Kernel and Firmware, Silicon Engineering](https://www.google.com/about/careers/applications/jobs/results/115719346323366598-software-engineer-kernel-and-firmware-silicon-engineering)
- [Google Careers — Software Engineer, Firmware](https://www.google.com/about/careers/applications/jobs/results/110574479087674054-software-engineer-firmware)
- [Reported Amazon Senior Firmware Engineer interview experience](https://www.jointaro.com/interviews/companies/amazon/experiences/senior-firmware-engineer-india-may-1-2024-no-offer-positive-e8536f7b/)
- [Reported embedded interview discussion covering Microsoft/Amazon technical rounds](https://leetcode.com/discuss/post/974067/nda-microsoft-amazon-senior-software-engineer-embedded-software-india-offers/)
