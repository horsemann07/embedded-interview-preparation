# RTOS — From First Principles to Expert Level

## Table of Contents

### 01. Introduction to RTOS
## Table of Contents

### 01. Introduction to RTOS

- [01.1 What is an RTOS?](#011-what-is-an-rtos)
  - [01.1.1 Real-Time Does Not Mean "Very Fast"](#0111-real-time-does-not-mean-very-fast)
  - [01.1.2 Why an Embedded System May Need More Structure](#0112-why-an-embedded-system-may-need-more-structure)
  - [01.1.3 The Basic Purpose of an RTOS](#0113-the-basic-purpose-of-an-rtos)
  - [01.1.4 A Task-Oriented View](#0114-a-task-oriented-view)
  - [01.1.5 The Meaning of "Operating System"](#0115-the-meaning-of-operating-system)

- [01.2 Why Do We Need an RTOS?](#012-why-do-we-need-an-rtos)
  - [01.2.1 The Growth of an Embedded Product](#0121-the-growth-of-an-embedded-product)
  - [01.2.2 Long Operations Can Delay Important Work](#0122-long-operations-can-delay-important-work)
  - [01.2.3 Different Work Has Different Importance](#0123-different-work-has-different-importance)
  - [01.2.4 Waiting Is Also Work](#0124-waiting-is-also-work)
  - [01.2.5 Communication Between Activities](#0125-communication-between-activities)
  - [01.2.6 Timing Requirements Become Easier to Express](#0126-timing-requirements-become-easier-to-express)
  - [01.2.7 Scaling the Software](#0127-scaling-the-software)
  - [01.2.8 Why Not Use an RTOS Everywhere?](#0128-why-not-use-an-rtos-everywhere)
  - [01.2.9 The Real Reason We Need an RTOS](#0129-the-real-reason-we-need-an-rtos)

- [01.3 Bare-Metal vs RTOS](#013-bare-metal-vs-rtos)
  - [01.3.1 RTOS Architecture](#0131-rtos-architecture)
  - [01.3.2 Bare-Metal Execution Model](#0132-bare-metal-execution-model)
  - [01.3.3 RTOS Execution Model](#0133-rtos-execution-model)
  - [01.3.4 Bare-Metal Advantages](#0134-bare-metal-advantages)
  - [01.3.5 Bare-Metal Limitations](#0135-bare-metal-limitations)
  - [01.3.6 RTOS Advantages](#0136-rtos-advantages)
  - [01.3.7 RTOS Costs](#0137-rtos-costs)
  - [01.3.8 Bare-Metal and RTOS Can Coexist](#0138-bare-metal-and-rtos-can-coexist)

- [01.4 General Purpose OS vs RTOS](#014-general-purpose-os-vs-rtos)
  - [01.4.1 Different Design Goals](#0141-different-design-goals)
  - [01.4.2 Throughput vs Predictability](#0142-throughput-vs-predictability)
  - [01.4.3 Example: Desktop Computer](#0143-example-desktop-computer)
  - [01.4.4 Example: Motor Controller](#0144-example-motor-controller)
  - [01.4.5 RTOS Does Not Automatically Mean "Small"](#0145-rtos-does-not-automatically-mean-small)
  - [01.4.6 General-Purpose OS Does Not Mean "Cannot Be Real-Time"](#0146-general-purpose-os-does-not-mean-cannot-be-real-time)
  - [01.4.7 Resource Constraints](#0147-resource-constraints)

- [01.5 Hard, Firm and Soft Real-Time](#015-hard-firm-and-soft-real-time)
  - [01.5.1 Hard Real-Time](#0151-hard-real-time)
  - [01.5.2 Firm Real-Time](#0152-firm-real-time)
  - [01.5.3 Soft Real-Time](#0153-soft-real-time)
  - [01.5.4 Comparison](#0154-comparison)

- [01.6 Determinism](#016-determinism)
  - [01.6.1 Average Time vs Worst-Case Time](#0161-average-time-vs-worst-case-time)
  - [01.6.2 Determinism Is Not "Always Exactly the Same"](#0162-determinism-is-not-always-exactly-the-same)
  - [01.6.3 Sources of Non-Determinism](#0163-sources-of-non-determinism)
  - [01.6.4 Determinism and System Design](#0164-determinism-and-system-design)
  - [01.6.5 Determinism Does Not Mean the System Is Automatically Correct](#0165-determinism-does-not-mean-the-system-is-automatically-correct)

- [01.7 Deadline, Latency and Jitter](#017-deadline-latency-and-jitter)
  - [01.7.1 Deadline](#0171-deadline)
  - [01.7.2 Period and Deadline Are Different](#0172-period-and-deadline-are-different)
  - [01.7.3 Latency](#0173-latency)
  - [01.7.4 Interrupt Latency](#0174-interrupt-latency)
  - [01.7.5 Scheduling Latency](#0175-scheduling-latency)
  - [01.7.6 Jitter](#0176-jitter)
  - [01.7.7 Sources of Jitter](#0177-sources-of-jitter)
  - [01.7.8 Relationship Between Deadline, Latency and Jitter](#0178-relationship-between-deadline-latency-and-jitter)

- [01.8 Real-World RTOS Examples](#018-real-world-rtos-examples)
  - [01.8.1 Automotive Controllers](#0181-automotive-controllers)
  - [01.8.2 Industrial Control](#0182-industrial-control)
  - [01.8.3 Consumer Electronics](#0183-consumer-electronics)
  - [01.8.4 IoT Device](#0184-iot-device)
  - [01.8.5 Robotics](#0185-robotics)
  - [01.8.6 Medical and High-Integrity Systems](#0186-medical-and-high-integrity-systems)
  - [01.8.7 Why the Same RTOS Concept Appears in Different Products](#0187-why-the-same-rtos-concept-appears-in-different-products)

- [01.9 RTOS Architecture](#019-rtos-architecture)
  - [01.9.1 Application Layer](#0191-application-layer)
  - [01.9.2 RTOS Layer](#0192-rtos-layer)
  - [01.9.3 HAL and Drivers](#0193-hal-and-drivers)
  - [01.9.4 BSP](#0194-bsp)
  - [01.9.5 Hardware Layer](#0195-hardware-layer)
  - [01.9.6 Layered View](#0196-layered-view)
  - [01.9.7 Why Layering Matters](#0197-why-layering-matters)

- [01.10 RTOS Mental Model](#0110-rtos-mental-model)
  - [01.10.1 Start With Events](#01101-start-with-events)
  - [01.10.2 Work Has Different Timing Requirements](#01102-work-has-different-timing-requirements)
  - [01.10.3 The CPU Is a Limited Resource](#01103-the-cpu-is-a-limited-resource)
  - [01.10.4 Think in Terms of "Who Needs the CPU?"](#01104-think-in-terms-of-who-needs-the-cpu)
  - [01.10.5 Think in Terms of "Who Is Waiting?"](#01105-think-in-terms-of-who-is-waiting)
  - [01.10.6 Think in Terms of "What Caused the Work?"](#01106-think-in-terms-of-what-caused-the-work)
  - [01.10.7 Think in Terms of Time](#01107-think-in-terms-of-time)
  - [01.10.8 Think in Layers](#01108-think-in-layers)
  - [01.10.9 The Complete RTOS Mental Model](#01109-the-complete-rtos-mental-model)

- [Chapter 01 Summary](#chapter-01-summary)


### 02. RTOS Kernel Fundamentals

- [02.1 What is a Kernel?](#021-what-is-a-kernel)
  - [02.1.1 Why Is a Kernel Necessary?](#0211-why-is-a-kernel-necessary)
  - [02.1.2 Kernel as a Resource Manager](#0212-kernel-as-a-resource-manager)
  - [02.1.3 Kernel Is Not the Entire RTOS](#0213-kernel-is-not-the-entire-rtos)
  - [02.1.4 Application and Kernel Have Different Responsibilities](#0214-application-and-kernel-have-different-responsibilities)
  - [02.1.5 A Factory Analogy](#0215-a-factory-analogy)
  - [02.1.6 Kernel as an Abstraction Layer](#0216-kernel-as-an-abstraction-layer)
  - [02.1.7 Kernel Does Not Mean "Magic"](#0217-kernel-does-not-mean-magic)
  - [02.1.8 Kernel and Hardware Dependence](#0218-kernel-and-hardware-dependence)

- [02.2 Kernel Responsibilities](#022-kernel-responsibilities)
  - [02.2.1 Execution Management](#0221-execution-management)
  - [02.2.2 Scheduling](#0222-scheduling)
  - [02.2.3 Timing Management](#0223-timing-management)
  - [02.2.4 Synchronization](#0224-synchronization)
  - [02.2.5 Inter-Task Communication](#0225-inter-task-communication)
  - [02.2.6 Kernel Object Management](#0226-kernel-object-management)
  - [02.2.7 Interrupt Interaction](#0227-interrupt-interaction)
  - [02.2.8 Memory-Related Responsibilities](#0228-memory-related-responsibilities)
  - [02.2.9 The Kernel as a Central Coordinator](#0229-the-kernel-as-a-central-coordinator)

- [02.3 Kernel Objects](#023-kernel-objects)
  - [02.3.1 Why Kernel Objects Exist](#0231-why-kernel-objects-exist)
  - [02.3.2 A Conceptual Queue Object](#0232-a-conceptual-queue-object)
  - [02.3.3 A Conceptual Task Object](#0233-a-conceptual-task-object)
  - [02.3.4 Kernel Object Lifetime](#0234-kernel-object-lifetime)
  - [02.3.5 Static and Dynamic Kernel Objects](#0235-static-and-dynamic-kernel-objects)

- [02.4 System Calls](#024-system-calls)
  - [02.4.1 Why Do We Need System Calls?](#0241-why-do-we-need-system-calls)
  - [02.4.2 A System Call as a Request](#0242-a-system-call-as-a-request)
  - [02.4.3 Not Every RTOS API Is Necessarily a Hardware Trap](#0243-not-every-rtos-api-is-necessarily-a-hardware-trap)
  - [02.4.4 The System Call Flow](#0244-the-system-call-flow)

- [02.5 Kernel Mode vs Application Mode](#025-kernel-mode-vs-application-mode)
  - [02.5.1 Why Privilege Separation Exists](#0251-why-privilege-separation-exists)
  - [02.5.2 Two-Level Mental Model](#0252-two-level-mental-model)
  - [02.5.3 Privileged vs Unprivileged](#0253-privileged-vs-unprivileged)
  - [02.5.4 What Happens During a Privileged Transition?](#0254-what-happens-during-a-privileged-transition)
  - [02.5.5 Why Many MCUs May Not Use This Separation](#0255-why-many-mcus-may-not-use-this-separation)

- [02.6 Tick Interrupt](#026-tick-interrupt)
  - [02.6.1 Why Is a Tick Needed?](#0261-why-is-a-tick-needed)
  - [02.6.2 Tick Frequency](#0262-tick-frequency)
  - [02.6.3 Tick Frequency and CPU Overhead](#0263-tick-frequency-and-cpu-overhead)
  - [02.6.4 What Happens During a Tick?](#0264-what-happens-during-a-tick)
  - [02.6.5 Tick Is Not the Same as Task Execution](#0265-tick-is-not-the-same-as-task-execution)
  - [02.6.6 Tick and Delay](#0266-tick-and-delay)
  - [02.6.7 Tick Resolution Is Not Guaranteed Execution Precision](#0267-tick-resolution-is-not-guaranteed-execution-precision)

- [02.7 Tickless Operation](#027-tickless-operation)
  - [02.7.1 Why Tickless Operation Exists](#0271-why-tickless-operation-exists)
  - [02.7.2 Traditional Tick](#0272-traditional-tick)
  - [02.7.3 Tickless Idle](#0273-tickless-idle)
  - [02.7.4 Tickless Does Not Mean "No Time"](#0274-tickless-does-not-mean-no-time)
  - [02.7.5 Tickless and Low Power](#0275-tickless-and-low-power)
  - [02.7.6 Tickless Trade-Offs](#0276-tickless-trade-offs)

- [02.8 Kernel Initialization](#028-kernel-initialization)
  - [02.8.1 Reset](#0281-reset)
  - [02.8.2 Startup Code](#0282-startup-code)
  - [02.8.3 Hardware Initialization](#0283-hardware-initialization)
  - [02.8.4 Kernel Data Structures](#0284-kernel-data-structures)
  - [02.8.5 Creating Initial Application Objects](#0285-creating-initial-application-objects)
  - [02.8.6 Starting the Scheduler](#0286-starting-the-scheduler)
  - [02.8.7 What Happens to main()?](#0287-what-happens-to-main)
  - [02.8.8 Kernel Initialization vs Application Initialization](#0288-kernel-initialization-vs-application-initialization)
  - [02.8.9 Why Initialization Order Matters](#0289-why-initialization-order-matters)
  - [02.8.10 A Complete Simplified Startup Flow](#02810-a-complete-simplified-startup-flow)

- [02.9 Putting the Kernel Pieces Together](#029-putting-the-kernel-pieces-together)

- [02.10 A Simple Kernel Mental Model](#0210-a-simple-kernel-mental-model)

- [02.11 Important Distinctions](#0211-important-distinctions)
  - [Kernel vs Application](#kernel-vs-application)
  - [Kernel vs RTOS](#kernel-vs-rtos)
  - [API vs System Call](#api-vs-system-call)
  - [Tick vs Scheduler](#tick-vs-scheduler)
  - [Tick vs Real-Time Precision](#tick-vs-real-time-precision)
  - [Kernel Mode vs RTOS](#kernel-mode-vs-rtos)

- [02.12 One Complete Example](#0212-one-complete-example)

- [02.13 Chapter Summary](#0213-chapter-summary)


### 03. Tasks / Threads

- [03.1 What is a Task?](#031-what-is-a-task)
  - [A task is more than a function](#a-task-is-more-than-a-function)
- [03.2 Task vs Thread](#032-task-vs-thread)
- [03.3 Why Do We Need Tasks?](#033-why-do-we-need-tasks)
- [03.4 How a Task Actually Executes](#034-how-a-task-actually-executes)
- [03.5 A Task Has Its Own Stack](#035-a-task-has-its-own-stack)
- [03.6 Task Lifecycle](#036-task-lifecycle)
- [03.7 Task States](#037-task-states)
  - [READY](#ready)
  - [RUNNING](#running)
  - [BLOCKED](#blocked)
  - [SUSPENDED](#suspended)
- [03.8 Understanding the Difference Between READY and RUNNING](#038-understanding-the-difference-between-ready-and-running)
- [03.9 Task State Transitions](#039-task-state-transitions)
- [03.10 A Real Example of Task States](#0310-a-real-example-of-task-states)
- [03.11 Task Control Block — TCB](#0311-task-control-block--tcb)
- [03.12 Why Does a TCB Exist?](#0312-why-does-a-tcb-exist)
- [03.13 Conceptual TCB Example](#0313-conceptual-tcb-example)
- [03.14 TCB and Scheduler](#0314-tcb-and-scheduler)
- [03.15 Task Stack](#0315-task-stack)
- [03.16 Why Does Every Task Need Its Own Stack?](#0316-why-does-every-task-need-its-own-stack)
- [03.17 Stack Growth Direction](#0317-stack-growth-direction)
- [03.18 Stack Pointer](#0318-stack-pointer)
- [03.19 Context](#0319-context)
- [03.20 Program Counter](#0320-program-counter)
- [03.21 CPU Registers](#0321-cpu-registers)
- [03.22 Context Is More Than Registers](#0322-context-is-more-than-registers)
- [03.23 Context Switching](#0323-context-switching)
- [03.24 Why Context Switching Is Necessary](#0324-why-context-switching-is-necessary)
- [03.25 Context Switch on a Single CPU](#0325-context-switch-on-a-single-cpu)
- [03.26 Context Switching Has a Cost](#0326-context-switching-has-a-cost)
- [03.27 Example of a Context Switch](#0327-example-of-a-context-switch)
- [03.28 Task Creation](#0328-task-creation)
- [03.29 What Happens When a Task Is Created?](#0329-what-happens-when-a-task-is-created)
- [03.30 Initial Task Context](#0330-initial-task-context)
- [03.31 Task Entry Function](#0331-task-entry-function)
- [03.32 What Happens When the Task Function Returns?](#0332-what-happens-when-the-task-function-returns)
- [03.33 Task Deletion](#0333-task-deletion)
- [03.34 Static vs Dynamic Task Creation](#0334-static-vs-dynamic-task-creation)
  - [Static](#static)
  - [Dynamic](#dynamic)
- [03.35 Why Deleting Tasks Can Be Dangerous](#0335-why-deleting-tasks-can-be-dangerous)
- [03.36 Stack Overflow](#0336-stack-overflow)
- [03.37 Why Stack Overflow Can Be Difficult to Debug](#0337-why-stack-overflow-can-be-difficult-to-debug)
- [03.38 Causes of Stack Overflow](#0338-causes-of-stack-overflow)
- [03.39 Recursion and RTOS Stacks](#0339-recursion-and-rtos-stacks)
- [03.40 Stack Monitoring](#0340-stack-monitoring)
- [03.41 Stack Size Is a Design Decision](#0341-stack-size-is-a-design-decision)
- [03.42 Task Context and Stack Are Closely Connected](#0342-task-context-and-stack-are-closely-connected)
- [03.43 Task Context Switching and the Stack Pointer](#0343-task-context-switching-and-the-stack-pointer)
- [03.44 Context Switching and the Program Counter](#0344-context-switching-and-the-program-counter)
- [03.45 Task Design Principles](#0345-task-design-principles)
  - [03.45.1 One Clear Responsibility](#03451-one-clear-responsibility)
  - [03.45.2 Do Not Create One Task for Every Function](#03452-do-not-create-one-task-for-every-function)
  - [03.45.3 Tasks Should Have Well-Defined Timing Behavior](#03453-tasks-should-have-well-defined-timing-behavior)
  - [03.45.4 Prefer Event-Driven Tasks Where Appropriate](#03454-prefer-event-driven-tasks-where-appropriate)
  - [03.45.5 Avoid Long Blocking Inside Important Tasks](#03455-avoid-long-blocking-inside-important-tasks)
  - [03.45.6 Be Careful With Task Priority](#03456-be-careful-with-task-priority)
  - [03.45.7 Minimize Shared State](#03457-minimize-shared-state)
  - [03.45.8 Define Ownership](#03458-define-ownership)
  - [03.45.9 Keep Task Loops Simple](#03459-keep-task-loops-simple)
  - [03.45.10 Tasks Should Not Busy-Wait Unnecessarily](#034510-tasks-should-not-busy-wait-unnecessarily)
  - [03.45.11 Keep Stack Usage Under Control](#034511-keep-stack-usage-under-control)
  - [03.45.12 Think About Failure Behavior](#034512-think-about-failure-behavior)
- [03.46 A Complete Example: Sensor Task](#0346-a-complete-example-sensor-task)
- [03.47 A Complete Example: Communication Task](#0347-a-complete-example-communication-task)
- [03.48 A Complete Example: Motor Control Task](#0348-a-complete-example-motor-control-task)
- [03.49 Multiple Tasks Working Together](#0349-multiple-tasks-working-together)
- [03.50 One Deep Mental Model](#0350-one-deep-mental-model)
  - [Code](#code)
  - [Stack](#stack)
  - [Context](#context)
  - [State](#state)
  - [Kernel metadata](#kernel-metadata)
- [03.51 The Relationship Between Task, TCB and Stack](#0351-the-relationship-between-task-tcb-and-stack)
- [03.52 The Relationship Between Task, Context and Context Switch](#0352-the-relationship-between-task-context-and-context-switch)
- [03.53 Why Task Design Matters More Than API Knowledge](#0353-why-task-design-matters-more-than-api-knowledge)
- [03.54 Chapter Summary](#0354-chapter-summary)

### 04. Scheduling

- [Scheduler](#scheduler)
  - [Scheduling is not the same as context switching](#scheduling-is-not-the-same-as-context-switching)
  - [When does the scheduler get involved?](#when-does-the-scheduler-get-involved)
- [Preemptive Scheduling](#preemptive-scheduling)
  - [Why preemption is useful](#why-preemption-is-useful)
  - [Preemption does not mean "interrupt everything immediately"](#preemption-does-not-mean-interrupt-everything-immediately)
  - [Preemption and blocking](#preemption-and-blocking)
- [Cooperative Scheduling](#cooperative-scheduling)
  - [Why cooperative scheduling can be useful](#why-cooperative-scheduling-can-be-useful)
  - [The danger of cooperative scheduling](#the-danger-of-cooperative-scheduling)
  - [Preemptive vs Cooperative](#preemptive-vs-cooperative)
- [Priority-Based Scheduling](#priority-based-scheduling)
  - [A simple example](#a-simple-example)
  - [Priority describes scheduling importance, not business importance](#priority-describes-scheduling-importance-not-business-importance)
  - [Giving every task high priority](#giving-every-task-high-priority)
- [Round Robin](#round-robin)
  - [Why round robin is useful](#why-round-robin-is-useful)
  - [Round robin does not normally mean equal execution work](#round-robin-does-not-normally-mean-equal-execution-work)
  - [Round robin and priority](#round-robin-and-priority)
- [Time Slicing](#time-slicing)
  - [Why time slicing matters](#why-time-slicing-matters)
  - [Time slicing is different from preemption](#time-slicing-is-different-from-preemption)
  - [Time slicing is not always enabled](#time-slicing-is-not-always-enabled)
- [Priority Inversion](#priority-inversion)
  - [The classic diagram](#the-classic-diagram)
  - [Why this is dangerous](#why-this-is-dangerous)
  - [The deeper lesson](#the-deeper-lesson)
- [Priority Inheritance](#priority-inheritance)
  - [A timeline](#a-timeline)
  - [Important limitation](#important-limitation)
- [Priority Ceiling](#priority-ceiling)
  - [Simple example](#simple-example)
  - [Why priority ceiling is useful](#why-priority-ceiling-is-useful)
- [Scheduling Analysis](#scheduling-analysis)
  - [Why utilization alone is not enough](#why-utilization-alone-is-not-enough)
- [Scheduling Timeline Analysis](#scheduling-timeline-analysis)
- [Worst-Case Execution Time](#worst-case-execution-time)
- [Blocking Time](#blocking-time)
- [Preemption and Scheduling Analysis](#preemption-and-scheduling-analysis)
- [Cooperative Scheduling Analysis](#cooperative-scheduling-analysis)
- [Scheduling Analysis Is About the Whole System](#scheduling-analysis-is-about-the-whole-system)
- [A Practical Scheduling Example](#a-practical-scheduling-example)
- [The Scheduling Mindset](#the-scheduling-mindset)
- [A Single Picture for the Entire Chapter](#a-single-picture-for-the-entire-chapter)
- [Chapter Summary](#chapter-summary)

### 05. Interrupts and RTOS
- [What is an interrupt?](#what-is-an-interrupt)
- [ISR](#isr)
- [ISR vs task](#isr-vs-task)
- [Interrupt latency](#interrupt-latency)
- [Deferred interrupt processing](#deferred-interrupt-processing)
- [ISR-safe APIs](#isr-safe-apis)
- [Nested interrupts](#nested-interrupts)
- [Interrupt priorities](#interrupt-priorities)
- [ISR-to-task communication](#isr-to-task-communication)

### 06. Inter-Task Communication
- [Why tasks need communication](#why-tasks-need-communication)
- [Queues](#queues)
- [Mailboxes](#mailboxes)
- [Message buffers](#message-buffers)
- [Stream buffers](#stream-buffers)
- [Event groups](#event-groups)
- [Notifications](#notifications)
- [Choosing the right mechanism](#choosing-the-right-mechanism)

### 07. Synchronization
- [Race conditions](#race-conditions)
- [Critical sections](#critical-sections)
- [Mutex](#mutex)
- [Binary semaphore](#binary-semaphore)
- [Counting semaphore](#counting-semaphore)
- [Recursive mutex](#recursive-mutex)
- [Spinlocks](#spinlocks)
- [Synchronization patterns](#synchronization-patterns)

### 08. Memory Management
- [Stack vs heap](#stack-vs-heap)
- [Static allocation](#static-allocation)
- [Dynamic allocation](#dynamic-allocation)
- [Memory fragmentation](#memory-fragmentation)
- [Memory pools](#memory-pools)
- [MPU](#mpu)
- [Memory protection](#memory-protection)
- [Zero-copy concepts](#zero-copy-concepts)

### 09. Time Management
- [System tick](#system-tick)
- [Delays](#delays)
- [Periodic tasks](#periodic-tasks)
- [Software timers](#software-timers)
- [Absolute vs relative timing](#absolute-vs-relative-timing)
- [Deadline management](#deadline-management)
- [Jitter](#jitter)
- [Time drift](#time-drift)

### 10. RTOS Internals
- [Scheduler internals](#scheduler-internals)
- [Ready lists](#ready-lists)
- [Blocked lists](#blocked-lists)
- [TCB internals](#tcb-internals)
- [Context switch internals](#context-switch-internals)
- [PendSV](#pendsv)
- [SVC](#svc)
- [SysTick](#systick)
- [Critical section implementation](#critical-section-implementation)

### 11. ARM Cortex-M + RTOS
- [Exception model](#exception-model)
- [NVIC](#nvic)
- [MSP vs PSP](#msp-vs-psp)
- [Handler vs Thread mode](#handler-vs-thread-mode)
- [CONTROL register](#control-register)
- [SVC](#svc)
- [PendSV](#pendsv-1)
- [SysTick](#systick-1)
- [Context switching on Cortex-M](#context-switching-on-cortex-m)

### 12. RTOS Synchronization Problems
- [Race condition](#race-condition)
- [Deadlock](#deadlock)
- [Livelock](#livelock)
- [Starvation](#starvation)
- [Priority inversion](#priority-inversion-1)
- [Priority inheritance](#priority-inheritance-1)
- [Avoiding concurrency bugs](#avoiding-concurrency-bugs)

### 13. Advanced Scheduling
- [Rate Monotonic Scheduling](#rate-monotonic-scheduling)
- [Earliest Deadline First](#earliest-deadline-first)
- [Response-time analysis](#response-time-analysis)
- [WCET](#wcet)
- [CPU utilization](#cpu-utilization)
- [Schedulability](#schedulability)

### 14. Multi-Core RTOS
- [SMP](#smp)
- [AMP](#amp)
- [Core affinity](#core-affinity)
- [Inter-core communication](#inter-core-communication)
- [Cache coherency](#cache-coherency)
- [Spinlocks](#spinlocks-1)
- [Multi-core](#multi-core)

---

# 01. Introduction to RTOS

Real-Time Operating Systems are commonly used in embedded products where software has to perform multiple activities while also meeting timing requirements.

A beginner may initially see an RTOS as a collection of APIs such as task creation, queues, semaphores, timers, and delays. That view is useful for writing code, but it is not enough for understanding how a real embedded system behaves.

A stronger understanding starts much earlier.

Before learning scheduling algorithms, task control blocks, context switching, or synchronization primitives, it is important to understand why an RTOS exists in the first place, what problem it solves, what it does not solve, and where it fits inside an embedded product.

This chapter builds that foundation.

---

# 01.1 What is an RTOS?

RTOS stands for:

```text
Real-Time Operating System
```

An RTOS is an operating system designed for systems where **the timing of software execution is an important part of system behavior**.

A traditional application may mainly care about producing the correct result.

A real-time system usually has two dimensions of correctness:

```text
                 SYSTEM CORRECTNESS
                        |
             +----------+----------+
             |                     |
             v                     v
        Correct Result       Correct Timing
             |                     |
             +----------+----------+
                        |
                        v
                  Correct System
```

Consider a temperature-control system.

The software reads a temperature sensor:

```text
Temperature = 95°C
```

The software correctly detects that the temperature is too high.

It then decides:

```text
Turn ON cooling
```

The calculation is correct.

But suppose the cooling system starts several seconds later.

The software produced the correct result, but it produced it too late.

In this type of system, timing is part of correctness.

That is the basic meaning of **real-time**.

---

## 01.1.1 Real-Time Does Not Mean "Very Fast"

The word **real-time** is often misunderstood.

A real-time system is not necessarily the fastest system.

Instead, the important question is whether the system can behave within its required timing limits.

Consider two implementations.

```text
System A

Execution:
10 ms
11 ms
10 ms
50 ms
12 ms
```

and:

```text
System B

Execution:
15 ms
15 ms
16 ms
15 ms
16 ms
```

System A sometimes finishes faster than System B.

However, System A also has a large variation.

System B is slower in some cases, but its execution is much more predictable.

For many real-time applications, System B may be easier to analyze because its behavior is more stable.

This gives us an important mental model:

```text
General-purpose software:

    "How quickly does it usually finish?"

Real-time software:

    "Can it finish within the required time
     under the conditions that matter?"
```

Real-time engineering therefore places significant importance on:

* timing,
* bounded behavior,
* response time,
* deadlines,
* and worst-case behavior.

---

## 01.1.2 Why an Embedded System May Need More Structure

A very small embedded system can often be implemented using a simple main loop.

For example:

```c
int main(void)
{
    while (1)
    {
        read_temperature();

        if (temperature > 40)
        {
            fan_on();
        }
        else
        {
            fan_off();
        }
    }
}
```

This design is simple.

The processor repeatedly performs the same operations:

```text
      +--------------------+
      | Read Temperature   |
      +---------+----------+
                |
                v
      +--------------------+
      | Compare Threshold  |
      +---------+----------+
                |
          +-----+-----+
          |           |
          v           v
       Fan ON       Fan OFF
          |           |
          +-----+-----+
                |
                v
              Repeat
```

There is nothing inherently wrong with this architecture.

For a small product, it may be the correct architecture.

The problem appears when the product grows.

The same product may eventually need to:

```text
Read temperature
Read humidity
Read pressure
Control a motor
Receive UART commands
Communicate over CAN
Handle Bluetooth
Monitor buttons
Run diagnostics
Detect faults
Store logs
Manage communication timeouts
Update a display
```

Now the software has many different responsibilities.

A large super-loop may look like:

```c
while (1)
{
    read_temperature();
    read_humidity();

    process_uart();
    process_can();
    process_bluetooth();

    check_buttons();

    control_motor();
    control_fan();

    check_faults();

    run_diagnostics();

    update_display();

    save_logs();
}
```

The processor executes these operations sequentially.

```text
read_temperature()
        |
        v
read_humidity()
        |
        v
process_uart()
        |
        v
process_can()
        |
        v
process_bluetooth()
        |
        v
check_buttons()
        |
        v
control_motor()
        |
        v
check_faults()
        |
        v
run_diagnostics()
        |
        v
update_display()
        |
        v
save_logs()
        |
        v
      repeat
```

As the number of responsibilities increases, several problems begin to appear.

The developer must now reason about:

* execution order,
* execution time,
* timing relationships,
* polling frequency,
* long-running functions,
* communication delays,
* interactions between features,
* and future software growth.

This is where an RTOS becomes useful.

---

## 01.1.3 The Basic Purpose of an RTOS

An RTOS provides a structured software environment for managing multiple activities while giving the application mechanisms for controlling:

```text
Execution
Timing
Scheduling
Communication
Synchronization
Resource usage
```

A simplified view is:

```text
+-----------------------------------------+
|              Application                |
|                                         |
|  Sensor    Motor    Communication       |
|  Control   Control  Processing          |
|                                         |
+--------------------+--------------------+
                     |
                     v
+-----------------------------------------+
|                 RTOS                    |
|                                         |
| Scheduling                              |
| Timing                                  |
| Task Management                         |
| Communication                          |
| Synchronization                         |
+--------------------+--------------------+
                     |
                     v
+-----------------------------------------+
|                Hardware                 |
|                                         |
| CPU / RAM / Timers / UART / SPI / CAN  |
+-----------------------------------------+
```

The RTOS becomes the layer that helps organize the execution of application software.

It does not create another CPU.

It does not magically make software instantaneous.

It provides mechanisms for deciding **how software activities should use the available resources**.

---

## 01.1.4 A Task-Oriented View

A complex application can be divided into logical execution activities.

For example:

```text
Application
    |
    +---- Sensor Processing
    |
    +---- Motor Control
    |
    +---- Communication
    |
    +---- Diagnostics
    |
    +---- Logging
```

An RTOS can represent these activities as independently manageable execution units.

The exact terminology differs between operating systems, but in many RTOSes these units are called **tasks** or **threads**.

The important concept for this chapter is not the API.

The important concept is separation of responsibilities.

Instead of having one giant sequence:

```text
A -> B -> C -> D -> E -> F -> G
```

the architecture becomes more structured:

```text
       Sensor Work
            |
       Motor Work
            |
     Communication Work
            |
      Diagnostic Work
            |
        Logging Work
            |
            v
           RTOS
            |
            v
           CPU
```

Later chapters will explain exactly how tasks are represented and scheduled.

For now, the key idea is:

> An RTOS provides a structured way to manage multiple pieces of application work.

---

## 01.1.5 The Meaning of "Operating System"

The operating-system part of RTOS means that the software provides services for managing system resources and application execution.

A microcontroller contains limited resources:

```text
CPU
RAM
Flash
Timers
Interrupts
Communication peripherals
DMA
GPIO
```

Many application components may need to use these resources.

For example:

```text
                 CPU
                  |
       +----------+----------+
       |          |          |
       v          v          v
    Sensor      Motor      Network
    Processing  Control    Processing
```

The operating system provides a structured way to manage this shared environment.

The exact amount of functionality provided depends on the RTOS.

A small RTOS may provide only a kernel and a few synchronization and timing primitives.

A larger real-time platform may provide additional services.

The important thing is that the RTOS provides an organized runtime environment rather than leaving every application component to manage the processor completely on its own.

---

# 01.2 Why Do We Need an RTOS?

An RTOS is not required for every embedded system.

This is an important point.

Using an RTOS simply because a project is an "embedded project" is not good engineering.

A small application may be easier to develop, test, and maintain using bare-metal software.

The need for an RTOS usually appears when the software starts having multiple interacting activities and meaningful timing requirements.

---

## 01.2.1 The Growth of an Embedded Product

Consider a simple device.

### Version 1

The product only reads a sensor.

```text
Sensor
   |
   v
MCU
   |
   v
Display
```

A simple loop may be enough.

```c
while (1)
{
    value = read_sensor();
    display(value);
}
```

Very little software architecture is required.

---

### Version 2

The product now supports:

```text
Sensor
Display
Buttons
UART
```

The software becomes:

```c
while (1)
{
    read_sensor();
    update_display();
    check_buttons();
    process_uart();
}
```

Still manageable.

---

### Version 3

The product grows further:

```text
Sensor
Display
Buttons
UART
BLE
Motor
Diagnostics
Logging
Watchdog
Fault Monitoring
```

Now the main loop becomes:

```c
while (1)
{
    read_sensor();
    update_display();
    check_buttons();
    process_uart();
    process_ble();
    control_motor();
    run_diagnostics();
    monitor_faults();
    update_logs();
}
```

At this point the architecture begins to matter significantly.

The question changes from:

> "Can I write this code?"

to:

> "Can I predict and control how all of these activities interact?"

That is one of the reasons an RTOS becomes valuable.

---

## 01.2.2 Long Operations Can Delay Important Work

Consider:

```c
while (1)
{
    read_sensor();

    process_communication();   // 100 ms

    control_motor();

    check_faults();
}
```

Suppose:

```text
Sensor       → 2 ms
Communication → 100 ms
Motor        → 1 ms
Fault check  → 1 ms
```

The timing looks like:

```text
Time ----------------------------------------------------------->

Sensor
|--2ms--|

Communication
        |-----------------------------100ms---------------------|

Motor
                                                               |-1ms-|

Fault
                                                                   |-1ms-|
```

During the 100 ms communication operation, the motor-control logic does not execute.

If the motor needs to be controlled every 10 ms, the architecture has a problem.

The issue is not that the motor code is wrong.

The issue is that **one activity is preventing another activity from receiving the processor at the time it needs it**.

An RTOS provides mechanisms to separate these activities and schedule them according to their requirements.

---

## 01.2.3 Different Work Has Different Importance

Not all software activities are equally time-sensitive.

Consider a controller with:

```text
Emergency protection
Motor control
Sensor processing
Communication
Display update
Logging
```

Their timing needs may look different:

```text
Emergency handling → Very time-sensitive

Motor control      → Time-sensitive

Sensor processing  → Periodic

Communication      → Event-driven

Display            → Less time-sensitive

Logging            → Background activity
```

A simple sequential loop treats them all as part of the same execution chain.

An RTOS allows the architecture to express different execution requirements.

This becomes particularly useful when:

* one operation is urgent,
* another can wait,
* another must run periodically,
* another should execute only when an event occurs,
* and another should run only when CPU time is available.

---

## 01.2.4 Waiting Is Also Work

A common problem in embedded software is that some activities spend most of their time waiting.

For example:

```text
Wait for UART message
Wait for sensor conversion
Wait for timer
Wait for external event
Wait for network data
```

A poorly designed system may continuously poll:

```c
while (uart_data_available() == false)
{
    /* keep checking */
}
```

The processor is spending CPU cycles asking the same question repeatedly.

An RTOS allows software to be structured so that an activity can wait without continuously consuming the CPU.

Conceptually:

```text
Task
  |
  v
Wait for event
  |
  v
No work required
  |
  v
Wait
  |
  | Event arrives
  v
Continue execution
```

This is one of the major reasons event-driven RTOS applications can be more efficient and easier to reason about.

---

## 01.2.5 Communication Between Activities

Large embedded systems contain multiple logical activities that need to exchange information.

For example:

```text
Sensor Processing
       |
       | sensor data
       v
Motor Control
       |
       | status
       v
Diagnostics
       |
       | fault information
       v
Logging
```

If everything is written inside one giant loop, the communication may happen through global variables and tightly coupled function calls.

As the software grows, that coupling becomes difficult to manage.

An RTOS provides standard mechanisms for communication and synchronization.

The details come later, but conceptually:

```text
Task A
  |
  | Data / Event
  v
+-----------+
| RTOS IPC  |
+-----------+
  |
  v
Task B
```

This creates clearer boundaries between pieces of software.

---

## 01.2.6 Timing Requirements Become Easier to Express

Suppose one operation should occur every 10 ms.

Another should occur every 100 ms.

Another should run whenever a UART message arrives.

Another should execute only when there is background CPU time.

The requirements are fundamentally different:

```text
Motor control
    → Periodic

Diagnostics
    → Periodic but slower

UART processing
    → Event-driven

Logging
    → Background
```

An RTOS provides concepts that map naturally to these types of workloads.

Instead of trying to manually coordinate everything inside one loop, the software architecture can be designed around the behavior of the system.

---

## 01.2.7 Scaling the Software

Embedded products rarely remain at version 1.

Features get added.

For example:

```text
Version 1
    Sensor

Version 2
    Sensor + Display

Version 3
    Sensor + Display + UART

Version 4
    Sensor + Display + UART + BLE

Version 5
    Sensor + Display + UART + BLE + Diagnostics

Version 6
    Sensor + Display + UART + BLE + Diagnostics + OTA
```

A software architecture that is manageable at version 1 can become difficult at version 6.

An RTOS does not automatically solve software scaling, but it provides architectural building blocks that can make a complex application easier to organize.

---

## 01.2.8 Why Not Use an RTOS Everywhere?

An RTOS has costs.

Typical costs include:

```text
RAM usage
Flash usage
CPU overhead
Kernel complexity
Scheduling complexity
Synchronization complexity
Debugging complexity
```

A simple temperature sensor may not need any of this.

For example:

```text
Read sensor
Calculate result
Turn output ON/OFF
Repeat
```

may be better implemented with a simple bare-metal loop.

Therefore:

```text
Small + simple + predictable
        |
        v
Bare-metal may be enough

Complex + concurrent + timing-sensitive
        |
        v
RTOS may be valuable
```

The decision should come from system requirements, not from fashion.

---

## 01.2.9 The Real Reason We Need an RTOS

The deepest reason to use an RTOS is not simply:

```text
"Because there are many functions."
```

It is:

> The system has multiple independent execution requirements that need to coexist in a controlled way.

Those requirements may involve:

* timing,
* responsiveness,
* waiting,
* communication,
* resource sharing,
* isolation of responsibilities,
* and predictable execution.

An RTOS provides a framework for expressing and managing those requirements.

---

# 01.3 Bare-Metal vs RTOS

Bare-metal software means the application runs directly on the hardware without a general-purpose operating-system kernel managing application execution.

A simple architecture is:

```text
+---------------------------+
|       Application         |
|                           |
|       main()              |
|          |                |
|       while(1)            |
+-------------+-------------+
              |
              v
+---------------------------+
|         Hardware          |
| CPU / RAM / Peripherals   |
+---------------------------+
```

The application controls the execution flow directly.

A typical bare-metal application may look like:

```c
int main(void)
{
    system_init();

    while (1)
    {
        read_sensor();
        process_input();
        control_motor();
        update_display();
    }
}
```

The developer controls the order explicitly.

---

## 01.3.1 RTOS Architecture

With an RTOS, an additional software layer exists:

```text
+--------------------------------+
|          Application           |
|                                |
| Task A   Task B   Task C       |
| Task D   Task E                |
+---------------+----------------+
                |
                v
+--------------------------------+
|            RTOS                |
|                                |
| Scheduler                      |
| Timing                         |
| Task Management                |
| Communication                  |
| Synchronization                |
+---------------+----------------+
                |
                v
+--------------------------------+
|           Hardware             |
+--------------------------------+
```

The application no longer directly controls every scheduling decision.

The RTOS provides that infrastructure.

---

## 01.3.2 Bare-Metal Execution Model

Suppose the application has four activities:

```text
A = Sensor
B = Communication
C = Motor
D = Display
```

A simple loop might execute:

```text
A -> B -> C -> D -> A -> B -> C -> D
```

The timing of each activity therefore depends strongly on the total execution time of everything before it.

If B becomes slower, C and D are delayed.

If D becomes slower, A is delayed in the next cycle.

The software is highly connected through execution order.

---

## 01.3.3 RTOS Execution Model

An RTOS can instead organize the activities as separate schedulable units.

```text
Sensor Task
Motor Task
Communication Task
Display Task
```

The scheduler can determine which task should execute at a particular time according to the scheduling model.

Conceptually:

```text
          +-------------+
          | Scheduler   |
          +------+------+
                 |
       +---------+---------+
       |         |         |
       v         v         v
    Sensor     Motor      Comm
     Task       Task      Task
```

This does not mean the tasks all run simultaneously on one CPU.

It means the RTOS manages their access to the CPU.

---

## 01.3.4 Bare-Metal Advantages

Bare-metal software can be extremely efficient.

Advantages may include:

* very low overhead,
* small memory footprint,
* direct control,
* simple startup,
* simple execution flow,
* easy timing analysis for small systems.

For a small application:

```text
while (1)
{
    read_adc();
    calculate();
    update_output();
}
```

an RTOS might add complexity without providing enough benefit.

---

## 01.3.5 Bare-Metal Limitations

As the application grows, limitations can appear.

The main loop can become:

```text
while (1)
{
    A();
    B();
    C();
    D();
    E();
    F();
    G();
    H();
    I();
}
```

Now every function affects every other function's execution timing.

This creates coupling.

Adding one more operation can change the timing of the entire system.

A previously safe design may become unsafe after adding a feature.

---

## 01.3.6 RTOS Advantages

An RTOS can provide:

* structured execution,
* task separation,
* scheduling,
* timing services,
* communication primitives,
* synchronization primitives,
* easier handling of waiting,
* better organization of complex workloads.

A system can be organized according to responsibility:

```text
+------------------+
| Sensor Task      |
+------------------+

+------------------+
| Motor Task       |
+------------------+

+------------------+
| Communication    |
| Task             |
+------------------+

+------------------+
| Diagnostics Task |
+------------------+
```

This separation often makes large software systems easier to understand.

---

## 01.3.7 RTOS Costs

The RTOS approach also introduces additional cost.

Each task may require memory for its execution context and stack.

The kernel itself consumes:

```text
Flash
RAM
CPU time
```

Scheduling also introduces overhead.

The more concurrent activities there are, the more the developer must reason about:

```text
Priority
Timing
Synchronization
Shared resources
Blocking
Starvation
```

Therefore:

> An RTOS solves some architectural problems while introducing another class of engineering problems.

That is an important engineering trade-off.

---

## 01.3.8 Bare-Metal and RTOS Can Coexist

The comparison is not completely binary.

An RTOS application still contains normal C/C++ code and hardware drivers.

Even inside an RTOS task, the application may use:

```c
while (1)
{
    read_sensor();
    process_sensor();
}
```

The difference is that the task itself is now managed by the RTOS.

A real embedded product may therefore contain:

```text
Application
   |
   +-- RTOS tasks
   |
   +-- Interrupt handlers
   |
   +-- Drivers
   |
   +-- Hardware abstraction
   |
   +-- Hardware
```

The RTOS is one architectural layer, not the entire application.

---

# 01.4 General Purpose OS vs RTOS

A General Purpose Operating System, or GP OS, is designed to support a broad range of applications and workloads.

Examples include:

```text
Windows
Linux
macOS
```

These systems are typically designed for rich applications involving:

* user interfaces,
* networking,
* storage,
* multimedia,
* many processes,
* large memory spaces,
* multiple users or applications,
* and general application workloads.

An RTOS is designed around a different set of priorities.

Its key concern is usually the ability to provide **controlled and analyzable timing behavior** for real-time workloads.

---

## 01.4.1 Different Design Goals

A simplified comparison is:

```text
General Purpose OS

    Optimize for:
    - Throughput
    - Fairness
    - User experience
    - Rich features
    - General workloads


RTOS

    Optimize for:
    - Predictable response
    - Timing requirements
    - Bounded behavior
    - Deterministic scheduling
    - Resource constraints
```

This does not mean a GP OS cannot run real-time applications.

It means its default design goals are different.

---

## 01.4.2 Throughput vs Predictability

Suppose a system has many jobs.

A general-purpose operating system may try to keep the system highly utilized and fairly distribute CPU time.

The goal can be:

```text
Finish a large amount of useful work
```

A real-time system may instead care more strongly about:

```text
Make sure this important operation
can respond within its required time
```

This creates different priorities in the operating-system design.

---

## 01.4.3 Example: Desktop Computer

Imagine a desktop computer running:

```text
Browser
Music
File copy
Compiler
Text editor
Background services
```

The user usually does not require a particular application to execute at exactly a specific microsecond.

Small changes in response time are usually acceptable.

The OS therefore has considerable freedom to optimize:

```text
Overall throughput
Responsiveness
Fairness
Resource utilization
```

---

## 01.4.4 Example: Motor Controller

Now consider a motor-control system.

```text
Position sensor
      |
      v
Control calculation
      |
      v
Motor command
```

Suppose the control algorithm is expected to execute every:

```text
10 ms
```

If execution is sometimes delayed by 100 ms because another operation is occupying the CPU, the result may be unacceptable.

The system therefore cares strongly about predictable timing.

---

## 01.4.5 RTOS Does Not Automatically Mean "Small"

Some RTOSes are very small.

Others are more feature-rich.

Therefore:

```text
Small OS  != automatically RTOS
RTOS      != automatically tiny
```

The defining idea is real-time behavior and timing requirements, not simply code size.

---

## 01.4.6 General-Purpose OS Does Not Mean "Cannot Be Real-Time"

This distinction is important.

Modern general-purpose systems can support real-time workloads under appropriate configurations and constraints.

For example, Linux can be configured for real-time workloads.

Therefore the difference should not be stated as:

```text
Linux = not real-time
RTOS  = real-time
```

A more accurate statement is:

```text
Traditional GP OS design
        |
        v
General-purpose workloads

RTOS design
        |
        v
Real-time workloads and
predictable timing requirements
```

The actual behavior depends on the complete system architecture, configuration, hardware, scheduler, drivers, and workload.

---

## 01.4.7 Resource Constraints

Embedded systems often operate with much tighter resources.

For example:

```text
Microcontroller

Flash → a few hundred KB
RAM   → a few hundred KB
CPU   → limited clock speed
Power → battery constrained
```

A desktop computer may have:

```text
GBs of RAM
Large storage
Powerful CPUs
```

A small RTOS can therefore provide a much smaller software environment suited to MCU-class devices.

However, not every RTOS runs only on tiny MCUs. Real-time systems also exist on larger processors.

---

# 01.5 Hard, Firm and Soft Real-Time

Real-time requirements are not all equally strict.

A useful classification is:

```text
Hard Real-Time
Firm Real-Time
Soft Real-Time
```

The distinction is mainly about what happens when a deadline is missed.

---

## 01.5.1 Hard Real-Time

In a hard real-time system, missing a required deadline can constitute a system failure or an unacceptable system condition.

The concept is:

```text
Deadline
   |
   v
+----------------------------+
| Result must be available   |
| within this time limit     |
+----------------------------+
```

If the deadline is missed:

```text
Deadline Missed
       |
       v
Unacceptable behavior
```

Examples can include certain:

* safety-critical control functions,
* protection systems,
* aircraft control functions,
* high-integrity industrial controls.

The exact classification depends on the actual system requirements.

---

## 01.5.2 Firm Real-Time

In a firm real-time system, a result that arrives after its deadline may no longer have useful value.

However, a missed deadline does not necessarily mean complete system failure.

For example:

```text
Result generated
      |
      | useful
      v
+-----------+
| Deadline  |
+-----------+
      |
      | late result
      v
   Discard
```

A late result may simply become obsolete.

The key property is:

```text
Before deadline → useful
After deadline  → no longer useful
```

---

## 01.5.3 Soft Real-Time

In a soft real-time system, late results still have value, but performance may degrade.

For example, consider a video system.

Suppose frames are expected every:

```text
16.67 ms
```

Actual timing might become:

```text
16 ms
18 ms
17 ms
20 ms
16 ms
```

The system can continue operating.

However, increased timing variation may appear to the user as:

```text
Reduced smoothness
Frame drops
Increased latency
```

The deadline is therefore important, but missing it is usually not catastrophic.

---

## 01.5.4 Comparison

| Type | Deadline              | Effect of Missing Deadline   |
| ---- | --------------------- | ---------------------------- |
| Hard | Strict                | May mean system failure      |
| Firm | Important             | Late result may be discarded |
| Soft | Important for quality | Performance degrades         |

The classification is based on system behavior, not simply on the name of the application.

The same broad application category can contain different types of real-time workloads.

---

# 01.6 Determinism

Determinism is one of the most important ideas in real-time engineering.

In simple language:

> A deterministic system behaves within known and bounded timing expectations when the relevant conditions are known.

Consider a function that normally takes:

```text
10 µs
```

Sometimes:

```text
11 µs
```

Sometimes:

```text
12 µs
```

But unexpectedly:

```text
2 ms
```

The average may still be small.

However, the large delay may matter greatly if the function is part of a time-sensitive control path.

---

## 01.6.1 Average Time vs Worst-Case Time

Consider:

```text
Run 1 → 10 µs
Run 2 → 11 µs
Run 3 → 10 µs
Run 4 → 12 µs
Run 5 → 500 µs
```

Average execution time can hide the problem.

A real-time engineer therefore asks:

```text
What is the worst relevant execution time?
```

This is why real-time analysis often focuses on bounds.

---

## 01.6.2 Determinism Is Not "Always Exactly the Same"

Deterministic does not necessarily mean:

```text
Every execution takes exactly 10 µs.
```

Instead, the system may have a known range:

```text
8 µs <= execution time <= 15 µs
```

The useful property is that the behavior is bounded and analyzable.

For example:

```text
Best case  = 8 µs
Typical    = 10 µs
Worst case = 15 µs
```

The worst-case value is important when determining whether a deadline can be met.

---

## 01.6.3 Sources of Non-Determinism

Timing can vary due to many factors:

```text
Interrupt activity
Higher-priority work
Memory behavior
Cache effects
Resource contention
Critical sections
Disabled interrupts
Variable execution paths
Peripheral latency
Driver behavior
Communication delays
```

The more timing-sensitive the system, the more carefully these sources must be understood.

---

## 01.6.4 Determinism and System Design

Suppose a control loop has:

```text
Period = 1 ms
```

and its execution requires:

```text
WCET = 200 µs
```

This provides substantial timing margin before considering other system activity.

Now suppose the same function sometimes requires:

```text
900 µs
```

The design becomes much harder to schedule safely because little time remains for:

```text
Interrupts
Other tasks
Communication
Diagnostics
Kernel overhead
```

Therefore deterministic behavior makes system analysis easier.

---

## 01.6.5 Determinism Does Not Mean the System Is Automatically Correct

An application can be deterministic and still be wrong.

For example:

```text
Task executes every 10 ms
```

but the required period is actually:

```text
5 ms
```

The execution is predictable, but the design requirement is not being met.

Therefore:

```text
Deterministic
       !=
Correct
```

Instead:

```text
Correct requirements
        +
Predictable behavior
        +
Enough timing margin
        =
Good real-time design
```

---

# 01.7 Deadline, Latency and Jitter

These three terms are often used together, but they describe different ideas.

```text
Deadline
    → Latest acceptable completion time

Latency
    → Delay before a response

Jitter
    → Variation in timing
```

Understanding the difference is essential.

---

## 01.7.1 Deadline

A deadline is the latest acceptable time at which an operation or result must be available.

Consider:

```text
Event
  |
  v
Start
  |
  |----------------------|
  |                      |
  |    Allowed time      |
  |                      |
  |----------------------|
                         |
                         v
                      Deadline
```

Suppose:

```text
Deadline = 10 ms
```

Then:

```text
5 ms   → deadline met
8 ms   → deadline met
10 ms  → deadline met
12 ms  → deadline missed
```

The deadline is a requirement.

It is not the same thing as execution time.

---

## 01.7.2 Period and Deadline Are Different

Consider a periodic control task:

```text
Period   = 10 ms
Deadline = 8 ms
```

The task starts every 10 ms, but each execution must complete within 8 ms of its release.

Conceptually:

```text
|<----------- 10 ms ----------->|

Task A
|------ execute ------|
        ^
        |
      <= 8 ms
        |
      Deadline
```

A periodic task can therefore have both a period and a deadline.

These concepts become important when scheduling real-time systems.

---

## 01.7.3 Latency

Latency is the delay between an event and the beginning or completion of the corresponding response, depending on how the latency is defined.

For example:

```text
External event
      |
      | 5 µs
      v
Software response begins
```

The measured latency is:

```text
5 µs
```

In another example:

```text
Event
  |
  | 10 µs
  v
Task starts
  |
  | 20 µs
  v
Task finishes
```

We can distinguish:

```text
Start latency = 10 µs

Execution time = 20 µs

Event-to-completion time = 30 µs
```

The exact definition of latency must always be stated when measuring a system.

---

## 01.7.4 Interrupt Latency

An external peripheral may generate an interrupt.

For example:

```text
UART byte arrives
      |
      v
Interrupt generated
      |
      | latency
      v
ISR begins
```

The time between the interrupt condition and ISR execution is generally referred to as interrupt latency.

Interrupt latency can be affected by:

* interrupt masking,
* higher-priority interrupts,
* current processor state,
* architecture,
* interrupt controller configuration,
* software critical sections.

---

## 01.7.5 Scheduling Latency

Suppose an event makes a task ready.

The task may not execute immediately.

There may be another task currently running.

Conceptually:

```text
Event
  |
  v
Task becomes READY
  |
  | waiting for CPU
  v
Scheduler selects task
  |
  v
Task starts
```

The delay from becoming ready to actually beginning execution is an example of scheduling latency.

This is different from the time required by the task itself.

---

## 01.7.6 Jitter

Jitter is variation around an expected timing value.

Suppose a task should execute every:

```text
10 ms
```

Ideal:

```text
10 ms
10 ms
10 ms
10 ms
10 ms
```

Actual:

```text
9 ms
11 ms
10 ms
12 ms
8 ms
```

The intervals vary.

That variation is timing jitter.

Another way to visualize it:

```text
Expected:

|----10----|----10----|----10----|----10----|


Actual:

|---9---|-----11-----|----10----|------12------|
```

Jitter matters especially in control systems, communication systems, audio/video processing, and any application where periodic timing must remain stable.

---

## 01.7.7 Sources of Jitter

Jitter can be introduced by:

```text
Higher-priority execution
Interrupts
Critical sections
Resource contention
Variable execution time
Memory effects
Communication
Scheduler decisions
```

For example:

```text
Periodic Task
     |
     v
Expected release
     |
     +----------------------+
                            |
                       Unexpected
                       higher-priority
                       work
                            |
                            v
                      Task delayed
```

The task still executes, but not exactly when expected.

---

## 01.7.8 Relationship Between Deadline, Latency and Jitter

These concepts can be understood together.

```text
                Event
                  |
                  v
          +---------------+
          |    Latency    |
          +---------------+
                  |
                  v
            Task executes
                  |
                  | execution time
                  v
              Result
                  |
                  v
              Deadline
```

If the timing changes from run to run:

```text
              Jitter
                |
                v
      Timing varies around
       the expected point
```

A real-time design therefore tries to ensure:

```text
Response latency
      +
Execution time
      +
Other delays
      <
Required deadline
```

with enough margin for the relevant worst-case conditions.

---

# 01.8 Real-World RTOS Examples

RTOS concepts become much easier to understand when connected to real products.

An RTOS can be found in many embedded systems where multiple activities must coexist under timing constraints.

---

## 01.8.1 Automotive Controllers

Consider an automotive electronic control unit.

The controller may receive:

```text
CAN messages
Sensor inputs
Timer events
GPIO events
Diagnostic requests
```

and may control:

```text
Motor
Actuator
Warning output
Communication
```

A simplified software structure could be:

```text
                Automotive ECU
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
   Sensor Task    Control Task    CAN Task
       |              |              |
       +--------------+--------------+
                      |
                      v
                Diagnostics
                      |
                      v
                   Logging
```

Different workloads have different timing requirements.

For example:

```text
Control → time-sensitive

Diagnostics → event-driven

Logging → background

Communication → message-driven
```

An RTOS can provide the mechanisms required to organize these activities.

---

## 01.8.2 Industrial Control

Consider an industrial motor controller.

The system may contain:

```text
Encoder
Temperature sensor
Current sensor
Motor controller
Communication interface
Fault monitor
```

The software flow may be:

```text
Encoder
   |
   v
Position measurement
   |
   v
Control calculation
   |
   v
Motor command
```

At the same time:

```text
Communication
Diagnostics
Temperature monitoring
Fault handling
Logging
```

may also be running.

The control loop has a strong timing requirement.

The logging task may have a much weaker requirement.

An RTOS can help separate these workloads.

---

## 01.8.3 Consumer Electronics

Consider a smart appliance.

It may contain:

```text
Buttons
Display
Sensors
Motor
Bluetooth
Wi-Fi
Audio
Fault detection
```

Possible software activities are:

```text
User interface
Sensor monitoring
Motor control
Communication
Audio processing
Diagnostics
```

Some operations are event-driven.

Some are periodic.

Some are background activities.

This is a natural environment for task-based software architecture.

---

## 01.8.4 IoT Device

Consider a connected sensor.

The software may need to:

```text
Read sensors
Process measurements
Maintain connectivity
Receive commands
Send telemetry
Handle OTA updates
Monitor battery
Store diagnostic information
```

A conceptual architecture is:

```text
              IoT Device
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
    Sensor      Network      Power
    Task        Task         Task
       |           |           |
       +-----------+-----------+
                   |
                   v
               RTOS Kernel
                   |
                   v
                Hardware
```

The network task may spend most of its time waiting.

The sensor task may operate periodically.

The power-management logic may respond to events.

The RTOS provides a structured way to manage these different behaviors.

---

## 01.8.5 Robotics

A robot may contain:

```text
Position sensors
Motors
Encoders
IMU
Communication
Safety monitoring
Navigation
```

The software can become highly concurrent.

For example:

```text
Sensor Acquisition
        |
        v
State Estimation
        |
        v
Control
        |
        v
Actuator
```

At the same time:

```text
Communication
Diagnostics
Safety Monitoring
Logging
```

are also active.

Timing becomes critical because control decisions depend on fresh sensor information.

---

## 01.8.6 Medical and High-Integrity Systems

Some medical and high-integrity systems contain strong timing and reliability requirements.

In such systems, the operating-system choice is only one part of the design.

The complete system may need:

```text
Timing analysis
Fault handling
Memory protection
Testing
Traceability
Certification requirements
Safety mechanisms
```

Therefore an RTOS does not automatically make a system safe.

The entire engineering process matters.

---

## 01.8.7 Why the Same RTOS Concept Appears in Different Products

Although the products are different, the underlying software pattern is often similar:

```text
                Events
                  |
                  v
             Application
                  |
       +----------+----------+
       |          |          |
       v          v          v
    Periodic   Event-driven Background
       |          |          |
       +----------+----------+
                  |
                  v
                 RTOS
                  |
                  v
               Hardware
```

The details change, but the need to organize concurrent activities remains.

---

# 01.9 RTOS Architecture

The architecture of an RTOS-based embedded system can be viewed as multiple software layers.

A common conceptual model is:

```text
+--------------------------------------------------+
|                  Application                     |
|                                                  |
| Sensor | Control | Communication | Diagnostics |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|                    RTOS                          |
|                                                  |
| Scheduler                                        |
| Timing                                           |
| Task Management                                  |
| Communication                                    |
| Synchronization                                  |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|             Hardware Abstraction                 |
|                                                  |
| HAL / BSP / Drivers                              |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|                    Hardware                      |
|                                                  |
| CPU | RAM | Flash | Timer | UART | SPI | I2C    |
+--------------------------------------------------+
```

The exact architecture differs between RTOSes and products, but this layered view is useful.

---

## 01.9.1 Application Layer

The application layer contains the actual product functionality.

For example:

```text
Battery Management
Motor Control
Sensor Processing
Communication Protocol
User Interface
Diagnostics
```

This layer answers:

> What is the product supposed to do?

For example:

```text
Read temperature
Calculate motor speed
Process a CAN command
Detect an over-temperature fault
```

The application should ideally depend on clean interfaces rather than knowing every low-level hardware detail.

---

## 01.9.2 RTOS Layer

The RTOS layer provides operating-system services.

Conceptually:

```text
+--------------------------------+
|            RTOS                |
|                                |
| Scheduling                     |
| Timing                         |
| Task management                |
| Synchronization                |
| Communication                 |
+--------------------------------+
```

This layer answers questions such as:

```text
Which software activity should execute?
How should activities wait?
How should software components communicate?
How should timing services be provided?
```

The RTOS does not know the complete business purpose of the product.

It provides generic mechanisms.

For example, the RTOS does not know:

```text
"This task controls a motorcycle motor."
```

It simply knows:

```text
"Here is a schedulable execution context."
```

The application gives that execution context meaning.

---

## 01.9.3 HAL and Drivers

Below the RTOS is usually some form of hardware abstraction and driver layer.

For example:

```text
Application
     |
     v
RTOS
     |
     v
HAL / Driver
     |
     v
UART Peripheral
```

The application may call:

```c
uart_send(data);
```

The driver handles hardware-specific operations such as:

```text
Register configuration
FIFO access
Interrupt configuration
DMA
Hardware status
Error handling
```

The exact architecture depends on the platform.

---

## 01.9.4 BSP

A Board Support Package, or BSP, is typically the software required to bring up and support a particular board or platform.

It may include:

```text
Clock configuration
Pin configuration
Startup support
Memory configuration
Peripheral initialization
Board-specific drivers
RTOS port integration
```

The exact BSP definition differs between organizations and platforms, but the general purpose is to adapt software to a specific board or hardware platform.

---

## 01.9.5 Hardware Layer

At the bottom are the physical resources:

```text
CPU
RAM
Flash
Timers
UART
SPI
I2C
CAN
GPIO
DMA
ADC
```

The RTOS ultimately executes on a processor and uses hardware mechanisms such as interrupts and timers to provide its services.

---

## 01.9.6 Layered View

The complete flow can therefore be visualized as:

```text
Application
     |
     | Product functionality
     v
RTOS
     |
     | Scheduling / timing / synchronization
     v
HAL / BSP / Drivers
     |
     | Hardware access
     v
MCU / SoC Hardware
```

This separation is valuable because each layer has a different responsibility.

For example:

```text
Application → What should happen?

RTOS        → When/how should software execute?

Driver      → How do we communicate with hardware?

Hardware    → Performs the physical operation.
```

This separation improves maintainability and portability.

---

## 01.9.7 Why Layering Matters

Suppose a product moves from one MCU to another.

If application code directly accesses hardware registers everywhere, the migration can become expensive.

Example:

```c
GPIOA->ODR |= PIN_5;
```

If this kind of code is spread throughout the application, changing the hardware requires changing many application files.

A hardware abstraction can instead provide:

```c
led_on();
```

The application knows about:

```text
LED
```

rather than:

```text
specific MCU register
```

The lower layer can then translate that request to the appropriate hardware implementation.

This is one of the main benefits of architectural separation.

---

# 01.10 RTOS Mental Model

The final section of this chapter brings all the concepts together.

The goal is to build a mental model that can be used when reading RTOS code, debugging a product, or designing a new system.

---

## 01.10.1 Start With Events

Embedded systems are driven by events.

Examples:

```text
UART message arrives
Timer expires
Sensor conversion completes
CAN message arrives
Button is pressed
DMA transfer completes
External interrupt occurs
Periodic work becomes due
```

The system reacts to these events.

A useful conceptual flow is:

```text
                EVENT
                  |
                  v
          Something happens
                  |
                  v
        Software needs to react
                  |
                  v
        Appropriate code executes
```

This is the starting point for understanding an RTOS system.

---

## 01.10.2 Work Has Different Timing Requirements

Not every event needs the same response.

For example:

```text
Emergency event
    |
    v
Very fast response required


Temperature monitoring
    |
    v
Periodic response


Display update
    |
    v
Slower response acceptable


Logging
    |
    v
Background response acceptable
```

An RTOS gives the architecture mechanisms for representing these differences.

---

## 01.10.3 The CPU Is a Limited Resource

Suppose a single CPU is responsible for:

```text
Sensor processing
Motor control
Communication
Diagnostics
Logging
```

The CPU cannot literally execute all of these instructions at the same instant.

Therefore:

```text
Multiple software activities
          |
          v
      Shared CPU
          |
          v
   Controlled execution
```

This is the fundamental problem that scheduling solves.

---

## 01.10.4 Think in Terms of "Who Needs the CPU?"

When analyzing an RTOS system, a very useful question is:

> Which software activity needs the CPU right now?

For example:

```text
                 CPU
                  |
       +----------+----------+
       |          |          |
       v          v          v
    Sensor      Motor      UART
     Task        Task       Task
```

At one moment, the motor-control work may need the CPU.

Later, the communication task may need it.

At another time, the system may have no useful work to perform and can remain idle.

The scheduler is responsible for managing this selection according to the RTOS scheduling model.

---

## 01.10.5 Think in Terms of "Who Is Waiting?"

A task often does not need the CPU continuously.

It may be waiting for:

```text
Time
Data
Event
Hardware completion
Communication
Another software component
```

Conceptually:

```text
Task
 |
 +---- Work
 |
 +---- Wait
 |
 +---- Work
 |
 +---- Wait
 |
 +---- Work
```

This pattern is extremely common in embedded software.

A good RTOS design allows software to wait efficiently rather than wasting CPU cycles.

---

## 01.10.6 Think in Terms of "What Caused the Work?"

When debugging an RTOS application, trace the chain backward.

Suppose a task unexpectedly ran.

Ask:

```text
What caused this work?

        |
        v
What event occurred?

        |
        v
Which software component detected it?

        |
        v
What made the relevant execution path active?

        |
        v
Why did the CPU execute it now?
```

This style of thinking is more powerful than simply looking at one function.

---

## 01.10.7 Think in Terms of Time

For any important operation, think about:

```text
When does it start?

How long can it take?

What can delay it?

When must it finish?

How often does it execute?

How much timing variation is acceptable?
```

These questions connect the architecture to real-time requirements.

A useful picture is:

```text
Event
  |
  v
Response starts
  |
  |<---- latency ---->|
  |
  v
Execution
  |
  |<--- execution --->|
  |
  v
Result
  |
  v
Deadline
```

If timing varies:

```text
           Jitter
              |
              v
      Response timing moves
      earlier or later
```

The system must still remain within its required timing limits.

---

## 01.10.8 Think in Layers

A real embedded system should also be understood vertically.

```text
+----------------------------------+
|          Application             |
|                                  |
| What does the product do?        |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
|              RTOS                |
|                                  |
| How is software execution        |
| managed and coordinated?         |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
|       HAL / BSP / Drivers        |
|                                  |
| How does software access         |
| the hardware?                    |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
|             Hardware             |
+----------------------------------+
```

This gives a second mental model:

```text
Horizontal view:
    Who gets CPU time?

Vertical view:
    Which software layer is responsible?
```

Both views are important.

---

## 01.10.9 The Complete RTOS Mental Model

The entire chapter can be summarized as one flow:

```text
                         EVENT
                           |
                           v
                +-------------------+
                | Event is detected |
                +---------+---------+
                          |
                          v
                +-------------------+
                | Application needs |
                | to perform work   |
                +---------+---------+
                          |
                          v
                +-------------------+
                | RTOS manages      |
                | execution         |
                +---------+---------+
                          |
                          v
                       CPU
                          |
                          v
                +-------------------+
                | Work is performed |
                +---------+---------+
                          |
                          v
                +-------------------+
                | Result / action   |
                +---------+---------+
                          |
                          v
                   Wait for next
                       event
```

Now add timing:

```text
                         EVENT
                           |
                           v
                    Response begins
                           |
                  <--- Latency --->
                           |
                           v
                      Processing
                           |
                  < Execution >
                           |
                           v
                       RESULT
                           |
                  <--- Deadline
```

And timing variation:

```text
Expected:

|------------- period -------------|
              Task


Actual:

|----------- period ------------|
           Task

|---------------- period ----------------|
                 Task
```

The difference between expected and actual timing is where jitter becomes important.

Finally, place everything into the software architecture:

```text
+------------------------------------------------+
|                  APPLICATION                   |
|                                                |
| Sensors | Control | Communication | Logging  |
+--------------------------+---------------------+
                           |
                           v
+------------------------------------------------+
|                     RTOS                       |
|                                                |
| Scheduling | Timing | Coordination            |
+--------------------------+---------------------+
                           |
                           v
+------------------------------------------------+
|                HAL / BSP / Drivers             |
+--------------------------+---------------------+
                           |
                           v
+------------------------------------------------+
|                    HARDWARE                    |
|                                                |
| CPU | RAM | Flash | Timer | UART | SPI | CAN |
+------------------------------------------------+
```

This is the foundation on which the rest of the RTOS book will build.

---

# Chapter 01 Summary

An RTOS is not simply a "small operating system" and not simply a "fast operating system."

It is an operating-system environment designed for systems where software execution has meaningful timing requirements.

The central idea is:

```text
Application has many activities
             |
             v
Those activities share limited resources
             |
             v
Their timing and interaction must be controlled
             |
             v
RTOS provides mechanisms for managing them
```

A small embedded product can often work perfectly well without an RTOS.

As a product becomes more complex, however, the software may need to support:

```text
Multiple activities
Different timing requirements
Event-driven behavior
Periodic processing
Communication
Background work
Resource sharing
```

An RTOS can provide a structured architecture for these requirements.

The most important concepts introduced in this chapter are:

```text
RTOS
 |
 +-- Real-time behavior
 |
 +-- Timing as part of correctness
 |
 +-- Structured execution
 |
 +-- Predictability
 |
 +-- Deadlines
 |
 +-- Latency
 |
 +-- Jitter
 |
 +-- Determinism
 |
 +-- Different real-time classes
 |
 +-- Layered architecture
 |
 +-- Event-driven execution
```

The most useful mental model to carry into the next chapters is:

```text
                 EVENTS
                    |
                    v
              APPLICATION
                    |
                    v
                  RTOS
                    |
                    v
                   CPU
                    |
                    v
                HARDWARE
```

with timing constantly present:

```text
      Event
        |
        v
     Latency
        |
        v
    Processing
        |
        v
     Result
        |
        v
    Deadline
```

And the core engineering question is always:

```text
Can the system perform the required work
at the required time,
under the required conditions,
with enough margin?
```

That question is the foundation of real-time system design.

# 02. RTOS Kernel Fundamentals

The previous chapter introduced the idea of an RTOS and explained why a real-time operating system can be useful in an embedded system.

The next step is to understand the software at the center of the RTOS:

> **The Kernel**

An RTOS application contains application code, drivers, hardware abstraction, and other software components. The kernel sits at the center of the execution environment and provides the fundamental mechanisms required to manage the system.

A useful high-level view is:

```text
+--------------------------------------------------+
|                APPLICATION                       |
|                                                  |
| Sensor | Control | Communication | Diagnostics |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|                 RTOS KERNEL                     |
|                                                  |
| Scheduling     Timing       Synchronization      |
| Communication Task Management  Resource Control |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|             HARDWARE / PROCESSOR                |
|                                                  |
| CPU | RAM | Timers | Interrupts | Peripherals   |
+--------------------------------------------------+
```

The kernel is not the application.

The kernel does not know that a particular task is controlling a motor, processing a temperature sensor, or handling Bluetooth data.

Instead, the kernel provides **generic mechanisms** that applications can use.

The application gives those mechanisms meaning.

For example:

```text
Application:
    "Wake this piece of software when data arrives."

RTOS:
    "I provide a mechanism for waiting and waking."

Hardware:
    "I provide the CPU, timer, memory and interrupts needed
     to make that mechanism work."
```

This chapter develops that model step by step.

---

# 02.1 What is a Kernel?

The **kernel** is the central software component of an operating system.

It sits between the application and the hardware and manages resources that must be shared by different parts of the software.

A simple model is:

```text
Application
     |
     v
+----------------+
|     Kernel     |
+----------------+
     |
     v
Hardware
```

The word **kernel** literally refers to the "core" of the operating system.

It is responsible for providing the mechanisms that allow software to use the system in an organized way.

---

## 02.1.1 Why Is a Kernel Necessary?

Consider a microcontroller with one CPU.

Suppose the application has several pieces of work:

```text
Sensor processing
Communication
Motor control
Diagnostics
Logging
```

All of them ultimately need CPU time.

The processor cannot simply give unlimited CPU time to every piece of software.

Some mechanism must coordinate the execution.

Without a kernel, the application itself may have to manually manage:

```text
Execution order
Timing
Waiting
Communication
Resource sharing
Interrupt interaction
```

For a very small application, that may be completely acceptable.

For a larger application, it becomes difficult to maintain and reason about.

The kernel provides a common place where these operating-system responsibilities can be handled.

---

# 02.1.2 Kernel as a Resource Manager

A useful way to think about the kernel is:

> **The kernel manages shared system resources and provides services to application software.**

The shared resources may include:

```text
CPU
Memory
Timers
Synchronization mechanisms
Communication mechanisms
Other kernel-managed resources
```

Conceptually:

```text
                  KERNEL
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
      CPU           Time       Communication
       |             |             |
       v             v             v
    Execution      Timers       Queues/Events
```

The exact list depends on the RTOS.

A minimal RTOS kernel may provide only a small set of services.

A more feature-rich RTOS may provide many additional facilities.

---

# 02.1.3 Kernel Is Not the Entire RTOS

The terms **RTOS** and **kernel** are often used as if they mean exactly the same thing.

They are related, but they are not necessarily identical.

A useful distinction is:

```text
RTOS
 |
 +-- Kernel
 |
 +-- Device drivers
 |
 +-- Networking
 |
 +-- File system
 |
 +-- Middleware
 |
 +-- Board support
 |
 +-- Other services
```

In some systems, "RTOS" refers mainly to the kernel.

In others, the RTOS distribution includes many additional software components.

The kernel is the **core execution and resource-management component**.

---

# 02.1.4 Application and Kernel Have Different Responsibilities

Consider a motor-control application.

The application is responsible for logic such as:

```text
Read motor feedback
Calculate control value
Generate motor command
```

The kernel is responsible for generic execution services such as:

```text
Manage executable units
Provide timing services
Coordinate access to kernel objects
Handle waiting and wake-up
Select which eligible execution unit runs
```

This separation is important.

The kernel should not contain product-specific logic such as:

```text
if motor_temperature > 100°C
    reduce_motor_speed();
```

That belongs to the application.

The kernel provides generic mechanisms.

---

# 02.1.5 A Factory Analogy

Imagine a factory.

```text
Factory
   |
   +---- Worker A → Assembly
   |
   +---- Worker B → Testing
   |
   +---- Worker C → Packaging
```

The workers perform the actual product-specific work.

The factory management system provides:

```text
Work allocation
Scheduling
Resource management
Timing
Coordination
```

The workers are similar to application execution units.

The management system is similar to the kernel.

The management system does not perform the product's actual business work.

It creates the environment in which that work can happen in an organized way.

---

# 02.1.6 Kernel as an Abstraction Layer

The kernel hides many low-level details behind higher-level interfaces.

For example, an application may request a delay:

```c
osDelay(100);
```

or another RTOS may provide:

```c
vTaskDelay(...);
```

The application does not directly manipulate every timer register to perform the delay.

Instead:

```text
Application
     |
     | delay request
     v
   Kernel
     |
     | manage timing
     v
 Hardware timer / system time
```

The exact implementation differs between RTOSes.

The important concept is abstraction.

---

# 02.1.7 Kernel Does Not Mean "Magic"

The kernel itself is just software.

It executes instructions on the CPU.

It uses:

```text
CPU instructions
RAM
Flash
Interrupts
Timers
Processor features
```

to implement its services.

For example:

```text
Kernel timer service
        |
        v
Hardware timer / clock source
        |
        v
Interrupt
        |
        v
Kernel time update
```

The kernel cannot escape hardware limitations.

If the CPU is overloaded, memory is insufficient, or interrupts are delayed, the kernel must operate within those constraints.

---

# 02.1.8 Kernel and Hardware Dependence

The generic kernel concepts may be portable, but part of the RTOS must understand the processor architecture.

For example:

```text
RTOS generic logic
       |
       v
Architecture-specific layer
       |
       v
CPU
```

The architecture-specific layer may need to understand:

```text
Interrupt mechanism
Processor registers
Privilege modes
Stacking behavior
Timer hardware
Context-switch mechanism
```

This is why an RTOS is typically portable at the API and conceptual level but still contains architecture-specific implementation code.

---

# 02.2 Kernel Responsibilities

The kernel has several responsibilities.

A simplified model is:

```text
                   RTOS KERNEL
                        |
      +-----------------+------------------+
      |                 |                  |
      v                 v                  v
  Execution           Timing          Communication
  Management
      |
      v
  Scheduling
      |
      v
  Resource / object
  management
```

Depending on the RTOS, these responsibilities can include:

* execution management,
* scheduling,
* timing,
* synchronization,
* inter-task communication,
* kernel-object management,
* memory-related services,
* interrupt interaction.

Each responsibility exists because application software needs a controlled way to use a shared processor.

---

## 02.2.1 Execution Management

The kernel needs a way to manage the different units of software that may execute.

Conceptually:

```text
          Application
               |
      +--------+--------+
      |        |        |
      v        v        v
    Task A   Task B   Task C
      \        |        /
       \       |       /
        +------+------+
               |
               v
             Kernel
```

The kernel must know enough about each executable unit to manage it.

It may need information such as:

```text
Current execution state
Priority
Stack information
Timing information
Scheduling-related information
```

The exact internal data structures vary between RTOS implementations.

Later chapters will go much deeper into task management and the Task Control Block.

For this chapter, the important point is:

> The kernel maintains the information required to manage application execution units.

---

# 02.2.2 Scheduling

The kernel is responsible for implementing the RTOS scheduling policy.

At a high level:

```text
Tasks become eligible to run
           |
           v
        Kernel
           |
           v
Scheduling decision
           |
           v
CPU executes selected work
```

Different RTOSes may support different scheduling behaviors.

Examples include:

```text
Priority-based scheduling
Preemption
Time slicing
Cooperative execution
```

The detailed algorithms belong to the scheduling chapter.

Here, remember only the responsibility:

> The kernel controls the rules by which CPU execution is assigned to application work.

---

# 02.2.3 Timing Management

Real-time applications need to reason about time.

The kernel therefore maintains some concept of system time.

This may be based on:

```text
Hardware timer
Periodic interrupt
Free-running counter
Other hardware time source
```

The kernel may use this time information for:

```text
Delays
Timeouts
Periodic operations
Software timers
Time-based wake-up
```

A simplified flow is:

```text
Hardware clock
      |
      v
Timer event
      |
      v
Kernel updates time
      |
      v
Time-based decisions
```

This timing service becomes one of the foundations of real-time execution.

---

# 02.2.4 Synchronization

Multiple software components may need to coordinate.

For example:

```text
Task A
   |
   | needs shared resource
   v
Shared Resource
   ^
   |
Task B
```

The kernel can provide synchronization mechanisms to help control these interactions.

Examples include:

```text
Mutex
Semaphore
Event
Notification
```

The detailed operation of these objects will be covered later.

At the kernel level, the important concept is:

> The kernel provides mechanisms that allow independent software activities to coordinate safely.

---

# 02.2.5 Inter-Task Communication

Software activities often need to exchange data.

For example:

```text
Sensor Task
     |
     | sensor data
     v
Communication Task
```

The kernel may provide communication mechanisms such as:

```text
Queue
Message buffer
Stream buffer
Notification
Event mechanism
```

The application decides what the data means.

The kernel provides the mechanism used to transfer or signal it.

---

# 02.2.6 Kernel Object Management

An RTOS commonly contains objects representing kernel-managed resources.

Examples:

```text
Task
Queue
Semaphore
Mutex
Timer
Event object
```

These objects contain information required by the kernel.

Conceptually:

```text
                 Kernel
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
     Task        Queue       Mutex
    Object      Object       Object
       |           |           |
       +-----------+-----------+
                   |
                   v
              Kernel state
```

The application interacts with these objects through APIs.

---

# 02.2.7 Interrupt Interaction

Embedded systems rely heavily on interrupts.

A peripheral might generate:

```text
UART interrupt
Timer interrupt
GPIO interrupt
DMA interrupt
ADC interrupt
CAN interrupt
```

The kernel may need to interact with interrupts because an interrupt can change the state of the system.

For example:

```text
Peripheral
    |
    v
Interrupt
    |
    v
ISR
    |
    v
Kernel-aware operation
    |
    v
Application becomes eligible to run
```

The exact behavior depends on the RTOS and architecture.

The important point is that interrupts are one of the major bridges between hardware events and kernel-managed software execution.

---

# 02.2.8 Memory-Related Responsibilities

Some RTOSes also provide memory-related services.

Depending on the implementation, this may include:

```text
Dynamic allocation
Fixed memory pools
Stack management
Memory protection
```

However, not every RTOS handles memory in the same way.

Some systems strongly prefer static allocation for predictability.

Others provide multiple allocation strategies.

Therefore:

> Memory management is an RTOS feature, but the exact design depends on the RTOS and application requirements.

---

# 02.2.9 The Kernel as a Central Coordinator

All these responsibilities come together in the kernel:

```text
                         KERNEL
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
    Execution            Timing          Communication
    Management
        |                   |                   |
        v                   v                   v
    Scheduling          Delays              Queues
        |              Timeouts           Messages
        |              Timers             Events
        +-------------------+-------------------+
                            |
                            v
                        Hardware
```

The kernel is therefore best understood as the **central coordination layer** of the RTOS.

---

# 02.3 Kernel Objects

A **kernel object** is a data structure that represents a resource or execution entity managed by the kernel.

Different RTOSes use different names and internal representations, but common concepts include:

```text
Task
Queue
Semaphore
Mutex
Timer
Event group
Message buffer
```

A useful conceptual view is:

```text
+--------------------------------------+
|              RTOS Kernel             |
|                                      |
|  +------+  +------+  +------+        |
|  | Task |  |Queue |  |Mutex |  ...   |
|  +------+  +------+  +------+        |
|                                      |
+--------------------------------------+
```

---

## 02.3.1 Why Kernel Objects Exist

The kernel needs information to manage system resources.

Consider a queue.

The kernel may need to know:

```text
Where is the queue buffer?
How many items can it store?
How many items are currently stored?
Who is waiting for data?
Who is waiting for space?
```

That information must exist somewhere in memory.

A kernel object provides a structure for holding that state.

---

## 02.3.2 A Conceptual Queue Object

Imagine a queue:

```text
+--------------------------------+
|            Queue               |
+--------------------------------+
| Buffer address                 |
| Item size                      |
| Capacity                       |
| Current item count             |
| Read position                  |
| Write position                 |
| Waiting information            |
+--------------------------------+
```

The real implementation may be different.

The purpose is the same:

> Store the state required by the kernel to manage the queue.

---

## 02.3.3 A Conceptual Task Object

A task also requires management information.

Conceptually:

```text
+--------------------------------+
|             Task               |
+--------------------------------+
| Task entry information         |
| Stack information              |
| Priority                       |
| State information              |
| Scheduling information         |
| Other kernel-managed state     |
+--------------------------------+
```

This information is often stored in a structure commonly called a:

> **Task Control Block (TCB)**

The TCB will deserve a dedicated later discussion.

At this stage, understand the general idea:

```text
Task
  |
  v
Kernel needs information about task
  |
  v
Task control data
```

---

## 02.3.4 Kernel Object Lifetime

A kernel object usually has a lifetime.

For example:

```text
Create
  |
  v
Object exists
  |
  v
Application uses object
  |
  v
Object is no longer needed
  |
  v
Delete / release
```

Some systems use static objects.

Others allow dynamic creation and deletion.

Still others support both.

This creates an important design decision around:

```text
Memory usage
Predictability
Lifetime management
Fragmentation
```

---

## 02.3.5 Static and Dynamic Kernel Objects

A kernel object can sometimes be created statically:

```text
Application-provided memory
        |
        v
Kernel object uses known memory
```

Or dynamically:

```text
Application requests object
        |
        v
Kernel allocates memory
        |
        v
Object created
```

Static allocation is often attractive in real-time systems because memory usage can be known ahead of time.

Dynamic allocation provides flexibility but introduces additional concerns.

The detailed memory-management trade-offs belong to the memory chapter.

---

# 02.4 System Calls

A **system call** is a controlled way for application software to request a service from the operating system kernel.

The conceptual model is:

```text
Application
     |
     | System call
     v
Kernel
     |
     v
Hardware / Kernel service
     |
     v
Result returned
     |
     v
Application
```

For example, an application may ask the RTOS to:

```text
Delay execution
Create an object
Send data
Wait for an event
Release a resource
```

The application does not directly manipulate all kernel internals.

It uses an interface provided by the operating system.

---

## 02.4.1 Why Do We Need System Calls?

Suppose application code could directly modify every kernel data structure.

That would be dangerous.

Imagine:

```text
Application
     |
     +---- modifies scheduler state
     |
     +---- modifies another task's state
     |
     +---- changes kernel timer state
```

A single incorrect write could corrupt the operating system.

A controlled interface is therefore useful.

Conceptually:

```text
Application
     |
     | controlled request
     v
+----------------------+
|   Kernel Interface   |
+----------------------+
     |
     v
+----------------------+
|      RTOS Kernel     |
+----------------------+
```

This creates a controlled boundary between application logic and kernel-managed state.

---

## 02.4.2 A System Call as a Request

Imagine:

```c
delay(100);
```

The application is effectively saying:

```text
"I need the kernel to provide a timing service."
```

The kernel may then:

```text
1. Record the timing requirement.
2. Update internal state.
3. Arrange for the application to become eligible again later.
4. Return control according to the RTOS rules.
```

The exact internal path depends on the RTOS.

---

## 02.4.3 Not Every RTOS API Is Necessarily a Hardware Trap

This is an important technical distinction.

The term "system call" is strongly associated with protected operating systems where an application transitions into privileged kernel execution.

In a small embedded RTOS, an API such as:

```c
xQueueSend(...);
```

may simply be a normal function call into kernel code when the application and kernel share the same privilege level.

In another configuration, a privileged transition may be required.

For example:

```text
Application
     |
     | function call
     v
Kernel
```

or:

```text
Application
     |
     | SVC / trap
     v
Privileged kernel
```

Therefore:

> The conceptual idea of a system call is a controlled request for a kernel service. The exact CPU-level mechanism depends on the architecture and RTOS configuration.

This distinction becomes important when moving from basic RTOS usage to protected systems.

---

# 02.4.4 The System Call Flow

A conceptual system-call flow looks like this:

```text
+----------------------+
| Application Code     |
+----------+-----------+
           |
           | Request service
           v
+----------------------+
| System Call API     |
+----------+-----------+
           |
           v
+----------------------+
| Kernel               |
|                      |
| Validate request     |
| Update state         |
| Perform operation    |
+----------+-----------+
           |
           v
+----------------------+
| Result / status      |
+----------------------+
           |
           v
+----------------------+
| Application          |
+----------------------+
```

The kernel remains in control of kernel-managed resources.

---

# 02.5 Kernel Mode vs Application Mode

The distinction between **kernel mode** and **application mode** comes from processor privilege and protection mechanisms.

The basic idea is:

```text
Application code
      |
      | limited access
      v
Application / User Mode

Kernel code
      |
      | privileged access
      v
Kernel / Privileged Mode
```

This separation exists to prevent normal application code from freely modifying critical operating-system state or hardware resources.

However, there is an important embedded-systems detail:

> Not every RTOS deployment uses privilege separation.

Many small microcontroller systems can run application code and kernel code at the same privilege level.

So kernel mode vs application mode is a **capability of the processor and RTOS architecture**, not a requirement that every RTOS must implement in exactly the same way.

---

## 02.5.1 Why Privilege Separation Exists

Consider a computer system.

The kernel controls:

```text
Memory management
CPU scheduling
Hardware access
Interrupt configuration
Critical operating-system state
```

If an ordinary application could execute privileged operations freely, it could accidentally or intentionally damage the operating system.

For example:

```text
Application
    |
    +---- corrupt scheduler data
    |
    +---- disable critical hardware
    |
    +---- modify protected memory
    |
    +---- access protected resources
```

Privilege separation provides a boundary.

---

## 02.5.2 Two-Level Mental Model

A simple mental model is:

```text
+---------------------------+
|     Application Mode      |
|                           |
| Normal application code   |
+-------------+-------------+
              |
              | controlled transition
              v
+---------------------------+
|       Kernel Mode         |
|                           |
| Privileged operations     |
+---------------------------+
              |
              v
           Hardware
```

The application requests a service.

The processor and operating system control how the privileged service is performed.

---

## 02.5.3 Privileged vs Unprivileged

The more general terminology is:

```text
Privileged
Unprivileged
```

rather than simply:

```text
Kernel
Application
```

because processor architectures differ.

For example, an ARM Cortex-M processor provides privilege levels through its processor architecture.

An RTOS can use these features to isolate application code from privileged kernel code when the design supports it.

---

## 02.5.4 What Happens During a Privileged Transition?

Conceptually:

```text
Application
     |
     | Request kernel service
     v
Privilege transition
     |
     v
Kernel
     |
     | perform operation
     v
Return to application
```

On some architectures, a dedicated exception or supervisor instruction is used.

On ARM Cortex-M, the **SVC** mechanism is one important example.

The detailed Cortex-M execution path will be covered later.

For now, remember:

```text
Application requests service
          |
          v
Controlled transition
          |
          v
Privileged kernel operation
```

---

## 02.5.5 Why Many MCUs May Not Use This Separation

Small embedded devices often prioritize:

```text
Low overhead
Small memory footprint
Simple architecture
Fast execution
```

A simple application may run:

```text
Application + Kernel
        |
        v
Same privilege level
```

This can reduce complexity.

The trade-off is reduced protection.

Therefore an RTOS can exist in different protection models:

```text
Simple MCU RTOS
    |
    +-- Kernel + application may share privilege


Protected RTOS design
    |
    +-- Application and kernel use different privilege levels
```

---

# 02.6 Tick Interrupt

Most traditional RTOS implementations maintain a concept of system time.

One common mechanism is a **periodic tick interrupt**.

The basic idea is:

```text
Hardware timer
      |
      | periodic interrupt
      v
RTOS tick handler
      |
      v
Update kernel time
```

For example, suppose the RTOS tick frequency is:

```text
1000 Hz
```

That means:

```text
1000 ticks / second
```

Therefore:

```text
1 tick = 1 ms
```

A simplified timing line becomes:

```text
Time →

|----1ms----|----1ms----|----1ms----|----1ms----|
     ↑            ↑            ↑            ↑
    Tick         Tick         Tick         Tick
```

The periodic interrupt gives the kernel regular opportunities to update time-related state.

---

## 02.6.1 Why Is a Tick Needed?

The kernel needs a time reference for operations such as:

```text
Delay
Timeout
Periodic timing
Software timers
Time measurement
```

Suppose a task requests:

```text
Delay = 100 ms
```

The kernel needs some way to know when that period has elapsed.

A tick-based system can maintain a counter:

```text
Tick count

0
1
2
3
...
100
```

If each tick represents 1 ms:

```text
100 ticks = 100 ms
```

The application can then use the kernel's time base.

---

## 02.6.2 Tick Frequency

The tick frequency is an important RTOS configuration.

Suppose:

```text
Tick frequency = 100 Hz
```

Then:

```text
1 tick = 10 ms
```

If:

```text
Tick frequency = 1000 Hz
```

Then:

```text
1 tick = 1 ms
```

If:

```text
Tick frequency = 10 kHz
```

Then:

```text
1 tick = 100 µs
```

Higher tick frequency gives finer time granularity.

But it also means the processor handles more tick interrupts.

---

## 02.6.3 Tick Frequency and CPU Overhead

Consider:

```text
100 Hz
```

The kernel receives:

```text
100 tick interrupts/second
```

Now consider:

```text
1000 Hz
```

The kernel receives:

```text
1000 tick interrupts/second
```

At:

```text
10 kHz
```

it receives:

```text
10,000 tick interrupts/second
```

Every interrupt consumes CPU time.

Therefore:

```text
Higher tick rate
      |
      +---- Better timing granularity
      |
      +---- More interrupt overhead
```

This is a design trade-off.

---

## 02.6.4 What Happens During a Tick?

A simplified conceptual flow is:

```text
Timer reaches configured interval
           |
           v
      CPU interrupt
           |
           v
      Tick handler
           |
           +---- advance system time
           |
           +---- process time-based events
           |
           +---- update scheduling state
           |
           v
       Return from interrupt
```

The exact operations differ between RTOSes.

A tick handler may do more or less depending on the implementation.

---

## 02.6.5 Tick Is Not the Same as Task Execution

A common misunderstanding is:

```text
Tick = task runs
```

That is not necessarily true.

The tick is fundamentally a time reference.

A tick may cause a scheduling decision, but whether a different task actually executes depends on the current system state and scheduling rules.

Conceptually:

```text
Tick occurs
    |
    v
Kernel processes time
    |
    v
Scheduling state may change
    |
    v
Scheduler may continue the same task
or select another eligible task
```

Therefore:

> A tick is a timing event, not simply a "task switch event."

---

# 02.6.6 Tick and Delay

Suppose:

```c
vTaskDelay(100);
```

and:

```text
1 tick = 1 ms
```

Conceptually:

```text
Current tick = 500

Requested delay = 100 ticks

Wake after approximately:
500 + 100
= 600 ticks
```

Timeline:

```text
Tick
500
 |
 |---- 100 ticks ----|
                     |
                    600
                     |
                     v
                Task eligible
```

The actual API semantics and exact wake-up behavior depend on the RTOS.

The important concept is that the kernel uses its time base to manage delayed execution.

---

## 02.6.7 Tick Resolution Is Not Guaranteed Execution Precision

Suppose:

```text
1 tick = 1 ms
```

That does not automatically mean a task will execute with exactly 1 ms precision.

There may be:

```text
Interrupt latency
Other higher-priority execution
Scheduling effects
Critical sections
Execution time
Hardware behavior
```

Therefore:

```text
Tick resolution
    !=
Guaranteed task response precision
```

This distinction is very important in real-time analysis.

---

# 02.7 Tickless Operation

A traditional tick-based RTOS receives periodic timer interrupts even when the system has very little work to do.

For example:

```text
Tick
Tick
Tick
Tick
Tick
Tick
Tick
Tick
```

This continues even when the CPU could otherwise sleep.

**Tickless operation** changes this behavior.

Instead of generating a periodic tick continuously, the system can sometimes program the timer for the next required event and allow the processor to remain idle until then.

---

## 02.7.1 Why Tickless Operation Exists

Consider a battery-powered device.

Suppose the only active task is:

```text
Wake every 1 second
Read sensor
Send data
Sleep again
```

A traditional 1 ms tick produces:

```text
1000 interrupts during 1 second
```

even though the application may have nothing useful to do during most of that time.

Conceptually:

```text
Active
  |
  v
Do work
  |
  v
Sleep
  |
  |---------------- 1 second ----------------|
  |
  v
Wake
```

Tickless operation can reduce unnecessary timer interrupts during the long idle period.

This can improve power efficiency.

---

## 02.7.2 Traditional Tick

With a traditional tick:

```text
Time →

|--Tick--|--Tick--|--Tick--|--Tick--|--Tick--|
    ↑        ↑        ↑        ↑        ↑
  CPU may wake periodically
```

Even if there is no useful application work, the tick continues.

---

## 02.7.3 Tickless Idle

With tickless idle:

```text
Time →

|---------------------------------------------|
                    Sleep
|---------------------------------------------|
                    |
                    v
                 Wake event
```

Instead of scheduling every periodic tick during idle time, the system can arrange a longer timer interval.

Conceptually:

```text
Current time
     |
     v
Next required event
     |
     v
Program timer for that point
     |
     v
Enter low-power mode
     |
     v
Wake when needed
```

---

## 02.7.4 Tickless Does Not Mean "No Time"

This is important.

Tickless operation does not mean:

```text
The system no longer knows time.
```

The RTOS still needs to maintain logical system time.

Instead, the hardware timer is used more efficiently.

The conceptual model is:

```text
Traditional:

Periodic hardware events
        |
        v
Update time frequently


Tickless:

Program timer to next relevant event
        |
        v
Sleep
        |
        v
Wake
        |
        v
Update elapsed time
```

---

## 02.7.5 Tickless and Low Power

The strongest motivation for tickless operation is often energy efficiency.

A battery-powered device wants to maximize:

```text
Sleep time
```

and minimize:

```text
Unnecessary CPU activity
```

The relationship is:

```text
Less unnecessary wake-up
          |
          v
More time in low-power state
          |
          v
Lower energy consumption
```

This is especially valuable in:

```text
Wearables
Battery sensors
IoT nodes
Remote monitoring devices
Portable electronics
```

---

## 02.7.6 Tickless Trade-Offs

Tickless operation is useful, but it introduces additional complexity.

The system must correctly manage:

```text
Elapsed time
Timer programming
Wake-up events
Clock behavior
Sleep-mode restrictions
Hardware timer limits
```

There can also be platform-specific complications.

For example, the hardware timer used for wake-up may have:

```text
Limited range
Different clock source
Clock drift
Low-power mode restrictions
```

Therefore tickless operation is an optimization that must be designed carefully.

---

# 02.8 Kernel Initialization

Before the RTOS can execute application work, the kernel itself must be initialized.

Initialization is the process of preparing the operating system's internal state and required hardware support.

A simplified boot flow is:

```text
Reset
  |
  v
Startup Code
  |
  v
Hardware Initialization
  |
  v
RTOS Initialization
  |
  v
Create / configure kernel objects
  |
  v
Start Scheduler
  |
  v
Application execution
```

The exact sequence differs between RTOSes and platforms.

---

## 02.8.1 Reset

When the processor resets, execution starts from the platform's defined reset entry point.

Conceptually:

```text
Power-on / Reset
       |
       v
Processor startup
       |
       v
Reset handler
```

The processor has not yet started normal RTOS operation.

The low-level startup environment must first be prepared.

---

## 02.8.2 Startup Code

Startup code typically performs low-level initialization required before normal C/C++ application execution.

Depending on the platform, this may include:

```text
Stack setup
Memory initialization
Vector table setup
Clock initialization
Data section initialization
BSS initialization
Runtime setup
```

The exact responsibilities depend on the compiler, linker, processor, and board.

A simplified flow is:

```text
Reset
  |
  v
Startup code
  |
  +---- Initialize stack
  |
  +---- Initialize memory
  |
  +---- Configure processor
  |
  v
main()
```

---

## 02.8.3 Hardware Initialization

The application may then initialize hardware.

For example:

```c
int main(void)
{
    system_clock_init();
    gpio_init();
    uart_init();
    spi_init();
    timer_init();

    ...
}
```

The exact order matters in some systems.

The RTOS may depend on hardware resources such as:

```text
System clock
Timer
Interrupt controller
Memory
```

Therefore the required hardware must be prepared before the kernel can fully operate.

---

## 02.8.4 Kernel Data Structures

The kernel must initialize its internal state.

Conceptually:

```text
Kernel Startup
      |
      +---- Initialize scheduler state
      |
      +---- Initialize timing state
      |
      +---- Initialize kernel object management
      |
      +---- Initialize required internal structures
      |
      v
Kernel ready
```

These structures allow the kernel to perform its responsibilities once the scheduler starts.

---

## 02.8.5 Creating Initial Application Objects

Before scheduler startup, the application may create or initialize required objects.

Conceptually:

```text
Initialize RTOS
      |
      v
Create application execution units
      |
      v
Create communication objects
      |
      v
Create timing objects
      |
      v
Start scheduler
```

For example:

```text
Create Sensor task
Create Communication task
Create Control task
Create Queue
Create Timer
```

The exact APIs differ between RTOSes.

---

## 02.8.6 Starting the Scheduler

The scheduler is normally started only after the required kernel and application state has been prepared.

Conceptually:

```text
main()
  |
  v
Initialize hardware
  |
  v
Initialize RTOS
  |
  v
Create required objects
  |
  v
Start scheduler
  |
  v
RTOS controls execution
```

After the scheduler starts, the normal application execution model changes.

The CPU is no longer simply following the original `main()` sequence.

Instead, the RTOS controls task execution according to its scheduling rules.

---

## 02.8.7 What Happens to main()?

This depends on the RTOS.

A common embedded pattern is:

```c
int main(void)
{
    hardware_init();

    create_tasks();

    start_scheduler();

    while (1)
    {
        // Normally should not execute if scheduler starts successfully.
    }
}
```

The scheduler takes over normal application execution.

In some systems, `main()` may remain part of the runtime in a different way.

Therefore, do not assume every RTOS uses exactly the same `main()` behavior.

The important concept is:

```text
Before scheduler starts:
    Startup/configuration phase

After scheduler starts:
    RTOS-managed execution phase
```

---

# 02.8.8 Kernel Initialization vs Application Initialization

These are related but different.

### Kernel initialization

Prepares:

```text
RTOS internal state
Timing system
Scheduler structures
Kernel object management
```

### Application initialization

Prepares:

```text
Product-specific tasks
Drivers
Communication interfaces
Sensors
Control software
Buffers
Application state
```

A simplified model is:

```text
                 Initialization
                       |
          +------------+------------+
          |                         |
          v                         v
    Kernel setup               Application setup
          |                         |
          v                         v
    RTOS ready                Product ready
          \                         /
           \                       /
            +---------+-----------+
                      |
                      v
               Start scheduler
```

---

# 02.8.9 Why Initialization Order Matters

Suppose an application attempts to use an RTOS service before the kernel has been initialized.

That creates an invalid execution condition.

Similarly, if a task expects a peripheral to be initialized but the driver has not been initialized yet, the task may fail.

Therefore initialization dependencies must be understood.

For example:

```text
Clock
  |
  v
Timer
  |
  v
RTOS time base
  |
  v
Kernel timing services
```

Or:

```text
UART hardware
      |
      v
UART driver
      |
      v
Communication task
```

The lower layer must generally be ready before the higher layer depends on it.

---

# 02.8.10 A Complete Simplified Startup Flow

A typical conceptual startup sequence can be visualized as:

```text
                    RESET
                      |
                      v
              +---------------+
              | Startup Code  |
              +-------+-------+
                      |
                      v
              +---------------+
              | CPU / Memory  |
              | Initialization|
              +-------+-------+
                      |
                      v
              +---------------+
              | Hardware Init |
              +-------+-------+
                      |
                      v
              +---------------+
              | RTOS Kernel   |
              | Initialization|
              +-------+-------+
                      |
                      v
              +---------------+
              | Create / Init |
              | Kernel Objects|
              +-------+-------+
                      |
                      v
              +---------------+
              | Start Scheduler|
              +-------+-------+
                      |
                      v
              +---------------+
              | Application   |
              | Execution     |
              +---------------+
```

This flow is one of the most useful diagrams to remember from this chapter.

---

# 02.9 Putting the Kernel Pieces Together

We can now combine all eight topics into one picture.

```text
                         APPLICATION
                              |
                     System Call / API
                              |
                              v
+---------------------------------------------------+
|                    RTOS KERNEL                    |
|                                                   |
|  +-------------+     +-------------------------+ |
|  | Kernel      |     | Kernel Objects          | |
|  | Services    |     |                         | |
|  |             |     | Task                    | |
|  | Scheduling  |     | Queue                   | |
|  | Timing      |     | Semaphore               | |
|  | Communication|    | Mutex                   | |
|  | Sync        |     | Timer                   | |
|  +-------------+     +-------------------------+ |
|                                                   |
|          Time base / Tick / Scheduler             |
+---------------------------+-----------------------+
                            |
                            v
                    CPU / Hardware
```

The time base may come from:

```text
Periodic tick
       or
Tickless timer operation
```

Application requests reach the kernel through:

```text
Normal kernel API
       or
Privileged system-call mechanism
```

depending on the architecture and protection model.

---

# 02.10 A Simple Kernel Mental Model

At this point, we can build a very simple mental model of the kernel.

Think of the kernel as a **manager sitting between application software and hardware**.

```text
                 APPLICATION
                      |
        "I need a service."
                      |
                      v
                  +-------+
                  |KERNEL |
                  +---+---+
                      |
           +----------+----------+
           |          |          |
           v          v          v
         CPU        TIME      OBJECTS
           |          |          |
           |          |          |
           +----------+----------+
                      |
                      v
                   HARDWARE
```

The application asks for services.

The kernel manages the state required to provide those services.

The hardware provides the physical resources used by the kernel.

---

# 02.11 Important Distinctions

A strong RTOS engineer should keep several distinctions clear.

## Kernel vs Application

```text
Application
    → Product functionality

Kernel
    → Generic operating-system services
```

---

## Kernel vs RTOS

```text
Kernel
    → Core operating-system component

RTOS
    → Real-time operating-system environment,
      which may include the kernel and additional components
```

---

## API vs System Call

```text
API
    → Interface exposed to application software

System call
    → Controlled request for a kernel service
```

An API call does not necessarily imply a processor privilege transition on every RTOS.

---

## Tick vs Scheduler

```text
Tick
    → Time reference / timer event

Scheduler
    → Decides which eligible execution unit should run
```

A tick may trigger scheduler activity, but the two concepts are not identical.

---

## Tick vs Real-Time Precision

```text
Tick resolution
    ≠
Guaranteed response time
```

The system still has to account for latency, execution time, interrupts, and other delays.

---

## Kernel Mode vs RTOS

```text
RTOS can exist without strict privilege separation.

Privilege separation is an architectural capability,
not a definition of an RTOS.
```

---

# 02.12 One Complete Example

Consider an IoT sensor node.

The product needs to:

```text
Read temperature every 1 second
Send data through Wi-Fi
Monitor battery
Handle incoming commands
Store error information
```

A simplified architecture is:

```text
                       IoT Application
                              |
       +----------------------+----------------------+
       |                      |                      |
       v                      v                      v
 Sensor Processing      Communication           Diagnostics
       |                      |                      |
       +----------------------+----------------------+
                              |
                              v
                         RTOS Kernel
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
         Scheduling         Timing       Communication
             |                |                |
             +----------------+----------------+
                              |
                              v
                           Hardware
```

Now consider the startup.

```text
Reset
  |
  v
Initialize clocks
  |
  v
Initialize memory
  |
  v
Initialize Wi-Fi hardware
  |
  v
Initialize RTOS kernel
  |
  v
Create application execution units
  |
  v
Create communication objects
  |
  v
Start scheduler
  |
  v
Application begins normal RTOS-managed execution
```

Now consider time.

The traditional approach might use a periodic tick:

```text
Tick
Tick
Tick
Tick
Tick
...
```

The kernel uses the time base to manage delays and timers.

If the device spends most of its time sleeping, tickless operation may reduce unnecessary periodic wake-ups:

```text
Wake
  |
  v
Read sensor
  |
  v
Transmit data
  |
  v
Sleep
  |
  |----------- long idle period -----------|
  |
  v
Wake again
```

This one example connects:

```text
Kernel
Kernel objects
System calls / APIs
Privilege
Tick
Tickless operation
Initialization
```

into a single system.

---

# 02.13 Chapter Summary

The RTOS kernel is the central software component that provides the mechanisms required to manage application execution and shared system resources.

The kernel sits between application software and the underlying hardware:

```text
Application
     |
     v
RTOS Kernel
     |
     v
Hardware
```

Its responsibilities commonly include:

```text
Execution management
Scheduling
Timing
Synchronization
Communication
Kernel object management
Interrupt interaction
Memory-related services
```

Kernel objects represent resources or execution entities that the kernel manages.

Examples include:

```text
Task
Queue
Semaphore
Mutex
Timer
Event
```

Applications access kernel services through defined interfaces.

A system call is conceptually a controlled request from application software to the kernel. On some systems this may simply involve a function call, while protected systems may use a privileged transition such as an exception or supervisor mechanism.

Privilege separation allows a system to distinguish between:

```text
Unprivileged application code
        and
Privileged kernel code
```

but not every embedded RTOS uses this separation.

Traditional RTOS timing often uses a periodic tick:

```text
Hardware Timer
      |
      v
Tick Interrupt
      |
      v
Kernel Time Update
```

The tick provides a time reference for operations such as delays and timeouts.

Higher tick frequency provides finer time granularity but increases interrupt overhead.

Tickless operation allows the system to avoid unnecessary periodic tick activity during idle periods:

```text
No useful work
      |
      v
Program next required wake-up
      |
      v
Sleep
      |
      v
Wake when required
```

This can be especially valuable in battery-powered systems.

Finally, kernel initialization prepares the RTOS before normal application execution:

```text
Reset
  |
  v
Startup
  |
  v
Hardware Initialization
  |
  v
Kernel Initialization
  |
  v
Object / Application Initialization
  |
  v
Scheduler Start
  |
  v
RTOS-managed Execution
```

The most important mental model from this chapter is:

```text
                         APPLICATION
                              |
                              v
                         RTOS APIs
                              |
                              v
                    +-------------------+
                    |       KERNEL      |
                    |                   |
                    | Execution         |
                    | Timing            |
                    | Scheduling        |
                    | Communication     |
                    | Synchronization   |
                    +---------+---------+
                              |
                              v
                        CPU / HARDWARE
```

The kernel does not perform the product's actual business logic.

It provides the machinery that allows the product software to execute in a controlled way.

That distinction is fundamental.

Once this foundation is clear, the next chapters can move deeper into the individual execution units managed by the kernel and the mechanisms the kernel uses to schedule and coordinate them.

# 03. Tasks / Threads

A task is one of the most important concepts in an RTOS.

Most of the RTOS features that we will study later—scheduling, priorities, blocking, synchronization, queues, timers, context switching, and even stack management—ultimately make sense only after the idea of a task is clear.

A beginner may think of a task as simply:

```text
"Some function that runs."
```

That is a useful starting point, but it is incomplete.

A task is not just a function.

A function is a piece of code.

A task is a **managed execution unit**.

The difference is important.

A normal function can be called by another function and then return:

```text
main()
  |
  v
foo()
  |
  v
return
  |
  v
main()
```

A task is different.

The RTOS creates an execution environment for it, gives it a stack, maintains information about it, places it into scheduling structures, and controls when it can execute.

A better mental model is:

```text
Task
 |
 +---- Code
 |
 +---- Stack
 |
 +---- CPU context
 |
 +---- Priority
 |
 +---- Current state
 |
 +---- Scheduling information
 |
 +---- Other kernel-managed information
```

This chapter builds that model from the ground up.

---

# 03.1 What is a Task?

Imagine an embedded product with several responsibilities:

```text
Temperature monitoring
Motor control
UART communication
Bluetooth communication
Diagnostics
Logging
Display update
```

If all of these responsibilities are written inside one large loop, the software becomes tightly connected.

An RTOS allows these responsibilities to be separated into different execution units.

For example:

```text
                 Application
                      |
      +---------------+---------------+
      |               |               |
      v               v               v
 Sensor Task      Motor Task      UART Task
      |               |               |
      +---------------+---------------+
                      |
                      v
                RTOS Scheduler
                      |
                      v
                     CPU
```

The important idea is that each task represents a piece of work that the RTOS can manage independently.

A task may:

* perform work continuously,
* perform work periodically,
* wait for an event,
* wait for data,
* sleep for some amount of time,
* or remain inactive until another part of the system wakes it.

A task therefore has a life of its own inside the RTOS.

---

## A task is more than a function

Consider this function:

```c
void read_sensor(void)
{
    int temperature = sensor_read();
}
```

This is simply a function.

It has no independent scheduling identity.

Another function may call it:

```c
while (1)
{
    read_sensor();
    update_display();
}
```

The CPU follows the calling function's execution.

Now consider an RTOS task:

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

This is not just a function being called repeatedly by another function.

The RTOS manages this execution unit.

The task has:

```text
Its own stack
Its own execution state
Its own program location
Its own priority
Its own scheduling information
```

Conceptually:

```text
+----------------------------------+
|           Sensor Task            |
+----------------------------------+
| Task Code                        |
|                                  |
| while(1)                         |
|   read_sensor()                  |
|   delay                          |
+----------------------------------+
| Task Stack                       |
+----------------------------------+
| CPU Context                      |
+----------------------------------+
| Priority / State / Kernel Info  |
+----------------------------------+
```

This is why a task should be thought of as a **complete execution context**, not just a function.

---

# 03.2 Task vs Thread

The words **task** and **thread** are often used interchangeably, but the exact terminology depends on the operating system.

In many RTOSes:

```text
Task ≈ Thread
```

Both represent an independently schedulable flow of execution.

For example:

```text
Task A
Task B
Task C
```

can all be scheduled independently.

In some operating systems, the word "process" has a different meaning and may represent a larger unit containing memory and resources, while threads are execution units inside a process.

In many embedded RTOS environments, there is no need for this large process abstraction.

The application may simply have:

```text
Task A
Task B
Task C
```

with each task having its own stack and CPU state.

Therefore, in this book we will mainly use the word **task**, because that terminology is common in embedded RTOS development.

---

# 03.3 Why Do We Need Tasks?

The main reason for introducing tasks is **separation of execution responsibilities**.

Consider a product containing:

```text
Sensor acquisition
Motor control
UART communication
BLE communication
Fault monitoring
Logging
```

A single loop could look like:

```c
while (1)
{
    sensor_process();
    motor_process();
    uart_process();
    ble_process();
    fault_process();
    logging_process();
}
```

This works only as long as the timing and complexity remain manageable.

Now imagine:

```text
ble_process() = 50 ms
```

During those 50 ms:

```text
motor_process()
fault_process()
```

cannot run if the processor is executing one sequential loop.

With task-based architecture, the responsibilities can be separated:

```text
Sensor Task
Motor Task
UART Task
BLE Task
Fault Task
Logging Task
```

Now the RTOS has an opportunity to manage these execution units according to their individual requirements.

The architecture becomes:

```text
              +----------------+
              |   RTOS Kernel  |
              +--------+-------+
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
  Motor Task       BLE Task       Sensor Task
       |
       +-------------------------------+
```

The task abstraction therefore gives the RTOS something that can be scheduled and managed.

---

# 03.4 How a Task Actually Executes

A very useful way to understand a task is to imagine the CPU executing its instructions one by one.

Suppose the task contains:

```c
void MotorTask(void *argument)
{
    while (1)
    {
        read_motor_sensor();
        calculate_control();
        update_motor_output();

        wait_for_next_cycle();
    }
}
```

The CPU might execute:

```text
read_motor_sensor()
       |
       v
calculate_control()
       |
       v
update_motor_output()
       |
       v
wait
```

When the task reaches the wait operation, it may no longer need the CPU.

The RTOS can then allow another task to execute.

For example:

```text
Motor Task
    |
    v
Do motor calculation
    |
    v
Wait
    |
    v
CPU becomes available


Communication Task
    |
    v
Process UART data
    |
    v
Wait
```

This is one of the core ideas behind task-based execution.

---

# 03.5 A Task Has Its Own Stack

One of the biggest differences between a normal function and an RTOS task is that a task has its own stack.

Consider:

```text
+----------------------+
| Sensor Task Stack    |
+----------------------+

+----------------------+
| Motor Task Stack     |
+----------------------+

+----------------------+
| UART Task Stack      |
+----------------------+
```

Why does every task need a stack?

Because each task can be paused and later resumed.

Suppose Task A is running:

```c
void TaskA(void *arg)
{
    int x;
    int y;

    function1();
}
```

The stack may contain things such as:

```text
Local variable x
Local variable y
Function call information
Saved registers
Temporary values
Return information
```

Now suppose Task A is interrupted by another task.

The RTOS must later restore Task A so that it continues exactly where it stopped.

Its execution state therefore needs a stack associated with that task.

---

# 03.6 Task Lifecycle

A task does not simply:

```text
Create → Run → End
```

In an RTOS, the task usually goes through several phases during its lifetime.

A simplified lifecycle looks like:

```text
                    +---------+
                    | CREATED |
                    +----+----+
                         |
                         v
                    +---------+
                    |  READY  |
                    +----+----+
                         |
                         | Scheduler selects
                         v
                    +---------+
                    | RUNNING |
                    +----+----+
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
           Delay       Wait       Suspend
             |           |           |
             v           v           v
          BLOCKED      BLOCKED    SUSPENDED
             |
             | event/time
             v
           READY
```

The exact lifecycle differs between RTOS implementations, but the general idea is common.

A task can:

```text
Be created
Become ready
Run
Wait
Become blocked
Become ready again
Run again
Be suspended
Resume
Eventually be deleted or terminate
```

Understanding this lifecycle is essential for debugging.

If a task "is not running," the first question should not be:

```text
"Why did my function fail?"
```

The better question is:

```text
"What state is the task in?"
```

That question often leads directly to the problem.

---

# 03.7 Task States

The most common conceptual states are:

```text
READY
RUNNING
BLOCKED
SUSPENDED
```

The exact implementation and naming may differ between RTOSes, but these states form a useful mental model.

---

## READY

A ready task is capable of executing.

It is not blocked.

It may simply be waiting for the scheduler to give it CPU time.

For example:

```text
Task A → READY
Task B → RUNNING
```

Task A is perfectly capable of executing, but Task B currently owns the CPU.

A useful interpretation is:

```text
READY = "I can run."
```

Not:

```text
READY = "I am currently running."
```

---

## RUNNING

A running task is the task that currently has CPU execution on a given core.

For a single-core MCU:

```text
CPU
 |
 +---- Task A RUNNING
```

Task A owns the processor at that moment.

Another task may be ready:

```text
Task A → RUNNING
Task B → READY
Task C → BLOCKED
```

---

## BLOCKED

A blocked task is waiting for something.

Examples include:

```text
Waiting for time
Waiting for queue data
Waiting for semaphore
Waiting for notification
Waiting for event
```

For example:

```c
vTaskDelay(pdMS_TO_TICKS(100));
```

The task effectively says:

```text
"I do not need the CPU for this period."
```

Its state can become:

```text
RUNNING
   |
   v
BLOCKED
   |
   | time expires
   v
READY
```

This is a very important optimization.

A task that has nothing to do should generally not continuously consume CPU cycles.

---

## SUSPENDED

A suspended task is deliberately removed from normal scheduling until it is resumed.

Conceptually:

```text
READY
  |
  v
SUSPENDED
  |
  | resume
  v
READY
```

Suspension is different from blocking.

A blocked task is waiting for a condition such as:

```text
Time
Queue
Semaphore
Event
```

A suspended task is explicitly removed from normal scheduling.

That difference becomes important when debugging task behavior.

---

# 03.8 Understanding the Difference Between READY and RUNNING

This distinction is easy to miss.

Suppose we have:

```text
Task A → Priority 5
Task B → Priority 3
```

Both are ready.

```text
Task A → READY
Task B → READY
```

But the scheduler selects Task A.

Therefore:

```text
Task A → RUNNING
Task B → READY
```

Task B is not blocked.

It is simply waiting for CPU time.

This is why a task can be perfectly healthy and still not be executing.

The mental model is:

```text
READY
  |
  | "I am eligible."
  |
  v
Scheduler chooses me
  |
  v
RUNNING
```

---

# 03.9 Task State Transitions

The transitions are more important than memorizing the state names.

A simplified model is:

```text
                         Create
                           |
                           v
                       +-------+
                       | READY |
                       +---+---+
                           |
                           | Scheduler
                           v
                       +-------+
                       |RUNNING|
                       +---+---+
                           |
               +-----------+-----------+
               |           |           |
               |           |           |
             Delay       Wait      Suspend
               |           |           |
               v           v           v
           +-------+   +-------+  +---------+
           |BLOCKED|   |BLOCKED|  |SUSPENDED|
           +---+---+   +---+---+  +----+----+
               |           |           |
               +-----------+           |
                           |           |
                           +-----+-----+
                                 |
                                 v
                               READY
```

For example:

```text
Task starts
   |
   v
RUNNING
   |
   | waiting for UART
   v
BLOCKED
   |
   | UART data arrives
   v
READY
   |
   | scheduler selects task
   v
RUNNING
```

This simple state-machine model is useful when debugging almost any RTOS system.

---

# 03.10 A Real Example of Task States

Consider three tasks:

```text
Motor Task
Sensor Task
Communication Task
```

At some instant:

```text
Motor Task          → RUNNING
Sensor Task         → BLOCKED
Communication Task  → READY
```

This can mean:

```text
Motor Task
    → currently executing

Sensor Task
    → waiting for next sensor period

Communication Task
    → can execute but is waiting for CPU
```

A little later:

```text
Motor Task
    → BLOCKED waiting for next cycle

Sensor Task
    → READY

Communication Task
    → READY
```

Now the scheduler may select one of the ready tasks according to its scheduling rules.

The state of the system therefore changes continuously.

---

# 03.11 Task Control Block — TCB

The RTOS needs a way to remember everything it needs to know about each task.

That information is commonly stored in a structure called the:

> **Task Control Block**

or:

> **TCB**

The TCB is essentially the kernel's record for a task.

A conceptual TCB looks like:

```text
+--------------------------------------+
|             Task Control Block       |
+--------------------------------------+
| Task state                           |
| Priority                             |
| Stack pointer                        |
| Stack information                    |
| Task entry information               |
| Scheduling information               |
| Task name / identification           |
| Other kernel-managed information     |
+--------------------------------------+
```

The exact fields differ between RTOS implementations.

The principle is the same:

> The kernel needs a record describing the state and resources associated with every task it manages.

---

# 03.12 Why Does a TCB Exist?

Imagine that Task A is running.

The RTOS needs to know:

```text
Where did this task stop?
What stack does it use?
What priority does it have?
What state is it in?
What scheduling information belongs to it?
```

Now imagine switching to Task B.

The kernel needs Task B's information.

Therefore:

```text
Task A
  |
  v
TCB A


Task B
  |
  v
TCB B


Task C
  |
  v
TCB C
```

The kernel maintains these records to manage each task independently.

---

# 03.13 Conceptual TCB Example

Suppose we have:

```text
Motor Task
Priority = 5
```

and:

```text
Logger Task
Priority = 1
```

Conceptually:

```text
TCB Motor
+-------------------------+
| Name: Motor             |
| Priority: 5             |
| State: READY            |
| Stack Pointer: 0x20002000|
| Stack Base: 0x20001000  |
+-------------------------+

TCB Logger
+-------------------------+
| Name: Logger            |
| Priority: 1             |
| State: BLOCKED          |
| Stack Pointer: 0x20004000|
| Stack Base: 0x20003000  |
+-------------------------+
```

The values above are illustrative.

The actual TCB layout depends on the RTOS.

The important concept is:

```text
TCB = Kernel's record for managing a task
```

---

# 03.14 TCB and Scheduler

The TCB is closely related to scheduling.

Suppose:

```text
Task A → Priority 5 → READY
Task B → Priority 3 → READY
Task C → Priority 1 → BLOCKED
```

The scheduler needs information about each task.

Conceptually:

```text
          Scheduler
              |
      +-------+-------+
      |       |       |
      v       v       v
    TCB A   TCB B   TCB C
      |       |       |
      v       v       v
    P5      P3      P1
    READY   READY   BLOCKED
```

The scheduler can use this information to determine which task is eligible to run.

The exact ready-list and scheduler data structures will be studied later.

---

# 03.15 Task Stack

Every task normally requires its own stack.

The stack is memory used during execution for things such as:

```text
Local variables
Function calls
Saved registers
Temporary data
Execution context
```

Consider:

```c
void MotorTask(void *arg)
{
    int speed;
    int current;

    read_sensor();
    calculate_motor(speed, current);
}
```

Variables such as:

```text
speed
current
```

may live on the task's stack depending on compiler optimization and storage class.

Function calls also use stack space.

A simplified stack looks like:

```text
+-------------------------+
| Function information    |
+-------------------------+
| Local variables         |
+-------------------------+
| Saved registers         |
+-------------------------+
| Temporary values        |
+-------------------------+
```

The exact layout depends on the processor ABI, compiler, optimization, and RTOS context-switch mechanism.

---

# 03.16 Why Does Every Task Need Its Own Stack?

Suppose Task A is executing:

```text
Task A
  |
  +-- local variables
  +-- function calls
  +-- temporary state
```

Now Task B starts running.

Task B may need completely different local variables:

```text
Task B
  |
  +-- local variables
  +-- function calls
  +-- temporary state
```

If both tasks used the same stack without proper isolation, their execution state would overwrite each other.

Therefore each task normally receives a separate stack area.

Conceptually:

```text
RAM
 |
 +---------------------------+
 | Task A Stack              |
 +---------------------------+
 | Task B Stack              |
 +---------------------------+
 | Task C Stack              |
 +---------------------------+
```

This is one of the most important memory implications of using an RTOS.

More tasks often mean more stack memory.

---

# 03.17 Stack Growth Direction

A stack may grow upward or downward depending on the processor architecture and ABI.

On many embedded processors, including common ARM Cortex-M configurations, stacks traditionally grow toward lower addresses.

Conceptually:

```text
Higher Address
+----------------------+
| Older stack content  |
+----------------------+
|                      |
|   Used stack space   |
|                      |
+----------------------+
        ^
        |
      SP
        |
        v
    Stack grows
    downward

Lower Address
```

The exact layout is architecture-dependent.

This is important when debugging memory corruption and stack overflow.

---

# 03.18 Stack Pointer

The CPU needs to know where the current task's stack is.

That location is represented by the:

> **Stack Pointer (SP)**

Conceptually:

```text
Task Stack
+----------------------+
| Data                 |
| Data                 |
| Saved state          |
| Local variables      |
+----------------------+
          ^
          |
         SP
```

When the task executes function calls and allocates local variables, the stack pointer changes.

When another task takes the CPU, the RTOS must eventually restore the correct stack pointer for that task.

This is one of the key reasons task context includes stack information.

---

# 03.19 Context

The word **context** means the processor state required for a task to continue execution correctly.

Suppose Task A is running:

```text
Task A
  |
  v
instruction 100
  |
  v
instruction 101
  |
  v
instruction 102
```

Now the RTOS switches to Task B.

When Task A later resumes, it must continue from the correct execution point.

It cannot restart from the beginning.

Therefore the system needs to preserve its execution context.

A simplified context contains:

```text
Program Counter
Stack Pointer
CPU Registers
Processor Status
Other architecture-specific state
```

---

# 03.20 Program Counter

The **Program Counter (PC)** tells the processor which instruction should execute next.

Conceptually:

```text
Code:

instruction 100
instruction 101
instruction 102
instruction 103
```

If Task A is currently around:

```text
instruction 102
```

the saved program counter must allow the task to resume at the appropriate location.

Conceptually:

```text
Task A context
      |
      +---- PC → instruction 102/next instruction
```

The exact point saved depends on the processor's exception and context-switch mechanism.

---

# 03.21 CPU Registers

The processor has registers used while executing code.

For example:

```text
General-purpose registers
Status registers
Stack pointer
Program counter
```

A task may depend on the current values of those registers.

Suppose:

```text
R0 = 10
R1 = 20
R2 = 30
```

Task A is using these values.

If the CPU switches to Task B and Task B changes them, Task A cannot simply resume with the modified values.

Therefore the required register state must be preserved and restored appropriately.

Conceptually:

```text
Task A
  |
  | Save registers
  v
Memory / Task Stack
  |
  | restore later
  v
Task A continues
```

---

# 03.22 Context Is More Than Registers

It is tempting to think:

```text
Context = CPU registers
```

That is an oversimplification.

A task's effective execution state may also involve:

```text
Stack pointer
Program counter
Processor status
Floating-point state, when applicable
RTOS-specific state
```

The exact set depends on:

```text
CPU architecture
Compiler ABI
RTOS implementation
Floating-point configuration
Execution mode
```

Therefore context is best understood as:

> The complete processor state required to stop one task and later continue it correctly.

---

# 03.23 Context Switching

A context switch occurs when execution changes from one task to another.

For example:

```text
Task A
   |
   | currently running
   v
+---------+
| CPU     |
+---------+
   |
   | context switch
   v
Task B
```

Conceptually, the process looks like:

```text
Task A running
      |
      v
Save Task A context
      |
      v
Scheduler selects Task B
      |
      v
Restore Task B context
      |
      v
Task B running
```

This is the basic idea of a context switch.

---

# 03.24 Why Context Switching Is Necessary

Suppose:

```text
Task A
```

is running a motor calculation.

A higher-priority event causes:

```text
Task B
```

to become ready.

The system wants Task B to execute.

But Task A must later continue correctly.

Therefore:

```text
Task A
   |
   +---- preserve state
   |
   v
Task B
   |
   +---- execute
   |
   v
Task A
   |
   +---- restore previous state
   |
   v
Continue
```

Without preserving Task A's state, it would not know where or how to continue.

---

# 03.25 Context Switch on a Single CPU

Suppose there is one CPU.

At time T1:

```text
CPU → Task A
```

At time T2:

```text
CPU → Task B
```

At time T3:

```text
CPU → Task A
```

The timeline is:

```text
Time ------------------------------------------------>

Task A       |================|
Task B                       |=========|
Task A                                  |===========|
```

The CPU is physically executing only one task at each instant.

The context-switch machinery creates the illusion of multiple independent execution streams.

That is the key idea behind multitasking.

---

# 03.26 Context Switching Has a Cost

A context switch is not free.

The CPU must perform work such as:

```text
Save state
Select next task
Restore state
Continue execution
```

Depending on architecture and RTOS:

```text
Registers
Stack information
Scheduler bookkeeping
Floating-point state
Other kernel state
```

may be involved.

Therefore:

```text
More context switches
        |
        v
More CPU overhead
```

This is one reason not to create unnecessary tasks or continuously force task switching.

---

# 03.27 Example of a Context Switch

Consider:

```text
Low Priority Task
```

currently running.

Then:

```text
High Priority Task
```

becomes ready.

Conceptually:

```text
Low Priority Task
       |
       | running
       v
████████████████████
       ^
       |
High Priority Task becomes READY
       |
       v
Context switch
       |
       v
High Priority Task
██████████
       |
       v
Low Priority Task resumes
████████████
```

The low-priority task does not lose its progress.

Its context is preserved so it can continue later.

---

# 03.28 Task Creation

Before a task can execute, the RTOS must create the task's execution environment.

Conceptually, task creation involves:

```text
Task creation request
        |
        v
Allocate / provide task memory
        |
        v
Create task control information
        |
        v
Initialize task stack
        |
        v
Initialize initial context
        |
        v
Place task under RTOS management
```

The exact sequence differs between RTOSes.

---

# 03.29 What Happens When a Task Is Created?

Suppose the application requests:

```c
create_task(MotorTask);
```

Conceptually, the RTOS needs to establish:

```text
Task code address
Task argument
Stack
Stack pointer
Priority
Initial state
Scheduling information
Task control data
```

The resulting memory may look approximately like:

```text
+-----------------------------+
| Motor Task TCB              |
+-----------------------------+
| Motor Task Stack            |
|                             |
|                             |
+-----------------------------+
| Motor Task Code             |
|                             |
+-----------------------------+
```

The task code generally resides in program memory, while the task's stack and control data reside in RAM.

---

# 03.30 Initial Task Context

A newly created task has never executed before.

So what does its CPU context look like?

The RTOS must create an initial context that makes the CPU behave as if the task were already ready to start.

Conceptually:

```text
New Task
   |
   v
Initial Stack Frame
   |
   v
Initial PC → Task entry function
   |
   v
Initial CPU state
```

When the scheduler eventually selects that task, the processor restores this initial context and enters the task function.

This is a subtle but important point:

> The RTOS does not simply "call" the task function in the normal C function-call sense.

It prepares an execution context for the task.

The first time the scheduler starts the task, the CPU enters that prepared context.

---

# 03.31 Task Entry Function

A typical RTOS task function looks like:

```c
void SensorTask(void *argument)
{
    while (1)
    {
        read_sensor();

        wait_for_next_cycle();
    }
}
```

The `argument` parameter allows the same task function to be used with different configuration data.

For example:

```c
SensorConfig config =
{
    .channel = 2,
    .period_ms = 100
};
```

The task can receive:

```c
SensorTask(&config);
```

Conceptually:

```text
Task
 |
 +---- Entry function
 |
 +---- Argument
 |
 +---- Stack
 |
 +---- TCB
```

The RTOS manages the execution around that entry function.

---

# 03.32 What Happens When the Task Function Returns?

Many RTOS task functions are designed not to return.

For example:

```c
void SensorTask(void *argument)
{
    while (1)
    {
        read_sensor();
        delay();
    }
}
```

The function continues indefinitely.

But what if it does return?

```c
void SensorTask(void *argument)
{
    read_sensor();

    return;
}
```

Different RTOSes handle task termination differently.

Some require the task to explicitly delete or terminate itself.

Some provide an idle cleanup mechanism.

Some systems define returning from the task entry as an error or unsupported condition.

Therefore task functions should generally follow the RTOS's expected lifecycle rules.

The important design principle is:

> A task's lifetime must be explicitly understood.

---

# 03.33 Task Deletion

When a task is no longer required, it may be deleted or terminated.

Conceptually:

```text
RUNNING
   |
   v
Terminate
   |
   v
Task removed from scheduling
   |
   v
Resources released
```

For a dynamically created task, this may involve:

```text
Task control data
Stack memory
Kernel bookkeeping
Other task-specific resources
```

The exact behavior depends on how the task was created and how memory is managed.

---

# 03.34 Static vs Dynamic Task Creation

A task may be created using memory that is already provided by the application, or the RTOS may allocate the required memory.

Conceptually:

### Static

```text
Application provides:

+----------------------+
| Task Stack           |
+----------------------+

+----------------------+
| Task Control Memory  |
+----------------------+
```

The memory exists for a known lifetime.

### Dynamic

```text
Task creation request
        |
        v
RTOS memory allocator
        |
        v
Stack + control memory
```

Dynamic creation is flexible.

Static creation can make memory usage more predictable.

This becomes especially important in systems with strict memory and reliability requirements.

---

# 03.35 Why Deleting Tasks Can Be Dangerous

Task deletion sounds simple:

```text
delete task
```

But imagine the task is holding:

```text
A mutex
A peripheral resource
A buffer
A hardware transaction
```

Deleting it immediately may leave the system in an inconsistent state.

For example:

```text
Task A
  |
  +---- owns resource
  |
  +---- gets deleted
           |
           v
      Resource remains
      incorrectly owned
```

Therefore task lifetime should be designed carefully.

In many robust systems, tasks are created during startup and remain alive for the entire product lifetime, blocking when they have no work.

This avoids repeated task creation and deletion.

---

# 03.36 Stack Overflow

Every task has a finite stack.

For example:

```text
Sensor Task → 512 bytes
Motor Task  → 1 KB
Comm Task   → 2 KB
```

The task can safely use only the stack that has been allocated.

If the task requires more, it may write beyond its stack boundary.

That condition is called:

> **Stack overflow**

Conceptually:

```text
+----------------------+
| Stack limit          |
+----------------------+
|                      |
| Valid stack usage    |
|                      |
+----------------------+
| Stack grows          |
| beyond allowed area  |
+----------------------+
        ^
        |
    Overflow
```

Stack overflow can corrupt:

```text
Adjacent memory
Task control data
Other task stacks
Global state
Heap
Kernel data
```

This makes stack overflow one of the most dangerous RTOS memory problems.

---

# 03.37 Why Stack Overflow Can Be Difficult to Debug

A stack overflow may not crash immediately.

Imagine:

```text
Task A Stack
     |
     v
overflow
     |
     v
corrupt memory
     |
     v
program continues
     |
     v
failure happens later
```

The actual crash may happen much later and somewhere else.

For example:

```text
Stack overflow
      |
      v
Corrupt pointer
      |
      v
Normal execution continues
      |
      v
Pointer used 500 ms later
      |
      v
Hard fault / crash
```

This is why stack corruption can be extremely difficult to diagnose.

---

# 03.38 Causes of Stack Overflow

Common causes include:

```text
Too-small stack
Deep function call chains
Large local variables
Large local arrays
Recursion
Large temporary objects
Unexpected execution paths
Library functions requiring significant stack
```

For example:

```c
void Task(void *arg)
{
    uint8_t buffer[2048];
    process_data(buffer);
}
```

If the task has only a 512-byte stack, this is obviously dangerous.

The exact amount of stack required depends on the compiler, architecture, optimization level, called functions, and runtime behavior.

---

# 03.39 Recursion and RTOS Stacks

Recursion can be especially dangerous on small embedded systems.

For example:

```c
void function(void)
{
    function();
}
```

Every recursive call consumes additional stack.

Conceptually:

```text
function()
   |
   +-- function()
         |
         +-- function()
               |
               +-- function()
                     |
                     +-- ...
```

The stack keeps growing until memory is exhausted.

Embedded RTOS applications should therefore use recursion carefully and only when stack requirements are well understood.

---

# 03.40 Stack Monitoring

Good RTOS systems often provide mechanisms to detect or estimate stack usage.

One common technique is to fill unused stack memory with a known pattern.

For example:

```text
Before execution:

AAAAAAAAAAAAAAAAAAAAAAAA

After execution:

AAAAAAAABBBBBBBBBBBBBBBB
        ^
        |
   used region
```

By examining how much of the pattern remains untouched, the system can estimate the **high-water mark**.

The exact technique depends on the RTOS.

This is useful because the stack requirement is difficult to estimate purely by guessing.

---

# 03.41 Stack Size Is a Design Decision

Giving every task a huge stack is not a good solution.

Suppose:

```text
10 tasks × 8 KB
= 80 KB
```

On a microcontroller with only 128 KB RAM, that may be unacceptable.

But giving every task a tiny stack is dangerous.

The engineer therefore needs to balance:

```text
Stack safety
      +
RAM usage
      +
Worst-case execution path
```

A practical process is:

```text
Estimate
   |
   v
Measure
   |
   v
Stress test
   |
   v
Measure high-water mark
   |
   v
Add engineering margin
```

---

# 03.42 Task Context and Stack Are Closely Connected

The concepts of stack and context should not be treated as unrelated.

The stack often contains a significant portion of the saved CPU context.

Conceptually:

```text
Task
 |
 +---- TCB
 |       |
 |       +---- Scheduler state
 |
 +---- Stack
         |
         +---- Saved CPU state
         +---- Local variables
         +---- Function-call state
```

When a task is switched out, part of its processor state may be stored on its stack.

When the task is restored, that state can be recovered.

The exact details depend on the processor and RTOS.

---

# 03.43 Task Context Switching and the Stack Pointer

This is one of the most important low-level relationships.

Suppose:

```text
Task A → Stack A
Task B → Stack B
```

The processor has one active stack pointer for the current execution context.

When switching from A to B:

```text
Before:

SP → Stack A
```

After restoring B:

```text
SP → Stack B
```

Conceptually:

```text
             CPU
              |
              v
        +-----------+
        |    SP     |
        +-----+-----+
              |
       +------+------+
       |             |
       v             v
   Stack A       Stack B
```

The correct stack pointer is therefore fundamental to correct task switching.

If the wrong stack pointer is restored, the processor may interpret the wrong memory as a saved execution context.

That can quickly cause:

```text
Corruption
Hard fault
Unexpected return
Random behavior
```

This is why stack corruption can destroy task execution so thoroughly.

---

# 03.44 Context Switching and the Program Counter

The stack pointer alone is not enough.

The task also needs to resume at the correct code location.

That requires the appropriate program counter state.

Conceptually:

```text
Task A
   |
   +---- SP → Stack A
   |
   +---- PC → Task A instruction


Task B
   |
   +---- SP → Stack B
   |
   +---- PC → Task B instruction
```

When Task A resumes:

```text
Restore A context
       |
       v
SP → Stack A
       |
       v
PC → Task A execution point
       |
       v
Continue
```

This is the essential mechanism behind independent task execution.

---

# 03.45 Task Design Principles

Creating tasks is easy.

Designing good tasks is much harder.

A good task architecture should be driven by **behavior and responsibility**, not by the desire to create many threads.

---

## 03.45.1 One Clear Responsibility

A task should preferably have a clear responsibility.

For example:

```text
Good:

SensorTask
CommunicationTask
MotorControlTask
DiagnosticTask
```

rather than:

```text
EverythingTask
```

containing:

```text
Sensor
Bluetooth
Logging
Motor
Diagnostics
Display
```

A task that owns too many unrelated responsibilities becomes difficult to reason about.

---

# 03.45.2 Do Not Create One Task for Every Function

This is a common beginner mistake.

Suppose you have:

```text
read_sensor()
calculate()
update_display()
```

Creating:

```text
ReadSensorTask
CalculateTask
DisplayTask
```

is not automatically a good design.

Functions and tasks represent different abstractions.

A function is a code unit.

A task is a scheduling unit.

A better design might be:

```text
SensorTask
    |
    +-- read_sensor()
    +-- calculate()
    +-- send result
```

if these operations naturally belong to the same execution flow.

The decision should come from timing, blocking, communication, and ownership requirements.

---

# 03.45.3 Tasks Should Have Well-Defined Timing Behavior

For important real-time tasks, define:

```text
Period
Deadline
Expected execution time
Worst-case execution time
Priority
Blocking behavior
```

For example:

```text
MotorControlTask

Period       = 10 ms
Deadline     = 8 ms
WCET         = 1.5 ms
```

This gives the engineer something measurable to work with.

A task without a timing model is difficult to analyze.

---

# 03.45.4 Prefer Event-Driven Tasks Where Appropriate

Consider a UART task.

Poor design:

```c
while (1)
{
    if (uart_data_available())
    {
        process_uart();
    }
}
```

The task may continuously poll.

A more event-driven architecture is conceptually:

```text
UART
 |
 v
Interrupt / Driver
 |
 v
Signal task
 |
 v
UART Task wakes
 |
 v
Process message
 |
 v
Wait again
```

This reduces unnecessary CPU usage and gives the task a clear execution model.

---

# 03.45.5 Avoid Long Blocking Inside Important Tasks

Suppose a high-priority control task performs:

```c
read_sensor();

perform_long_operation();   // 100 ms

control_motor();
```

Even if the task has high priority, the long operation can prevent other important activities from getting CPU time.

A better architecture may separate the work:

```text
Control Task
     |
     +---- fast control
     |
     +---- request long operation
                |
                v
          Background Task
```

This preserves responsiveness.

---

# 03.45.6 Be Careful With Task Priority

Priority should be based on timing requirements, not simply on how important the software "feels."

For example:

```text
Motor Control      → high
Communication      → medium
Logging            → low
```

may make sense if motor control has the strictest response requirement.

But blindly making every task high priority creates problems.

If:

```text
Task A → High
Task B → High
Task C → High
Task D → High
```

then the priority hierarchy provides little useful information.

Priority should reflect scheduling requirements.

---

# 03.45.7 Minimize Shared State

Tasks become harder to understand when they freely modify the same global variables.

For example:

```c
volatile int state;
```

used by:

```text
Task A
Task B
Task C
ISR
```

can create difficult concurrency problems.

A cleaner design often uses explicit communication:

```text
Task A
  |
  v
Queue / Event / Notification
  |
  v
Task B
```

This makes ownership and data flow easier to understand.

---

# 03.45.8 Define Ownership

A useful task-design question is:

> Who owns this resource?

For example:

```text
UART hardware
      |
      v
Communication Task owns UART
```

Other tasks communicate with it through messages rather than directly touching the peripheral.

This can reduce resource contention.

Conceptually:

```text
Sensor Task
     |
     | request
     v
Communication Task
     |
     v
UART
```

instead of:

```text
Sensor Task -----+
                 |
Logger Task -----+----> UART
                 |
Control Task ----+
```

The second design is more likely to require synchronization and can become difficult to reason about.

---

# 03.45.9 Keep Task Loops Simple

A good task often has a recognizable structure:

```c
void CommunicationTask(void *arg)
{
    while (1)
    {
        wait_for_message();

        process_message();

        send_response();
    }
}
```

The logic is easy to follow:

```text
Wait
  |
  v
Work
  |
  v
Respond
  |
  v
Wait again
```

This event-driven structure often makes RTOS applications easier to debug.

---

# 03.45.10 Tasks Should Not Busy-Wait Unnecessarily

Avoid:

```c
while (!event_ready)
{
    /* waste CPU */
}
```

when the RTOS provides a blocking mechanism appropriate to the problem.

A better model is:

```text
Task
 |
 v
Wait
 |
 v
BLOCKED
 |
 | event arrives
 v
READY
 |
 v
RUNNING
```

This gives the CPU to other work while the task is waiting.

---

# 03.45.11 Keep Stack Usage Under Control

Large local variables can silently increase stack requirements.

Instead of:

```c
void Task(void *arg)
{
    uint8_t huge_buffer[4096];
}
```

consider whether the buffer should instead be:

```text
Static memory
Shared buffer with controlled ownership
Memory pool
Dedicated communication buffer
```

The correct choice depends on the application.

The important point is that task stack is a limited resource.

---

# 03.45.12 Think About Failure Behavior

A robust task design should define what happens when something goes wrong.

For example:

```text
Communication timeout
        |
        v
Retry?
        |
        +---- yes → retry
        |
        +---- no → report fault
```

or:

```text
Sensor failure
     |
     v
Invalid data
     |
     v
Safe state
```

Task design is not only about the normal path.

The error paths also consume CPU, memory, and stack.

---

# 03.46 A Complete Example: Sensor Task

Consider a sensor that should be sampled every 100 ms.

A simple task might be:

```c
void SensorTask(void *argument)
{
    while (1)
    {
        int temperature = sensor_read();

        process_temperature(temperature);

        send_temperature(temperature);

        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

Its behavior is:

```text
RUNNING
   |
   +--> Read sensor
   |
   +--> Process data
   |
   +--> Send result
   |
   v
BLOCKED / WAIT
   |
   | 100 ms
   v
READY
   |
   v
RUNNING
```

The task is therefore:

```text
Periodic
Simple
Self-contained
Easy to reason about
```

---

# 03.47 A Complete Example: Communication Task

Now consider a UART communication task.

It may be event-driven:

```c
void CommunicationTask(void *argument)
{
    while (1)
    {
        message_t msg;

        wait_for_message(&msg);

        process_message(&msg);

        send_response();
    }
}
```

Its behavior becomes:

```text
WAIT
 |
 | message arrives
 v
READY
 |
 v
RUNNING
 |
 +---- process message
 |
 +---- send response
 |
 v
WAIT
```

This task may spend most of its life waiting.

That is perfectly normal.

A task does not need to consume CPU continuously to be useful.

---

# 03.48 A Complete Example: Motor Control Task

A motor-control task may have stronger timing requirements:

```c
void MotorControlTask(void *argument)
{
    while (1)
    {
        sensor_data_t data = read_motor_feedback();

        control_output_t output =
            calculate_control(&data);

        apply_motor_output(&output);

        wait_until_next_cycle();
    }
}
```

The architecture might be:

```text
Motor Feedback
      |
      v
Motor Task
      |
      +---- Read
      |
      +---- Calculate
      |
      +---- Output
      |
      v
Wait for next cycle
```

This task should be designed around its timing requirements.

For example:

```text
Period = 1 ms
Deadline = 500 µs
```

Then everything inside the task has to be analyzed against those constraints.

---

# 03.49 Multiple Tasks Working Together

Consider:

```text
+---------------------+
| Sensor Task         |
+----------+----------+
           |
           | Data
           v
+---------------------+
| Control Task        |
+----------+----------+
           |
           | Command
           v
+---------------------+
| Motor Driver        |
+---------------------+

+---------------------+
| Communication Task  |
+---------------------+

+---------------------+
| Diagnostics Task    |
+---------------------+
```

The RTOS manages the execution of these software units.

The tasks interact through explicit mechanisms.

The resulting system can be understood as:

```text
                  Events
                    |
       +------------+------------+
       |            |            |
       v            v            v
   Sensor Task   Comm Task   Diagnostics
       |
       v
   Control Task
       |
       v
     Output
```

This is much closer to how real embedded products are architected.

---

# 03.50 One Deep Mental Model

A very powerful way to understand a task is to think of it as five things combined:

```text
Task
 |
 +---- Code
 |
 +---- Stack
 |
 +---- Context
 |
 +---- State
 |
 +---- Kernel metadata
```

### Code

What instructions does the task execute?

### Stack

Where does the task keep function-call state, local data, and execution state?

### Context

What CPU state must be preserved so the task can continue?

### State

Is the task:

```text
READY
RUNNING
BLOCKED
SUSPENDED
```

### Kernel metadata

What does the kernel need to know to manage it?

This mental model makes many advanced topics much easier.

---

# 03.51 The Relationship Between Task, TCB and Stack

These three concepts are closely related but are not the same thing.

```text
             TASK
              |
      +-------+-------+
      |               |
      v               v
     TCB             STACK
      |               |
      |               |
      v               v
Kernel metadata   Runtime state
```

The task is the execution abstraction.

The TCB is the kernel's record describing that task.

The stack is memory used by that task during execution.

A simplified memory picture:

```text
RAM
+--------------------------------+
| Task A TCB                     |
+--------------------------------+
| Task A Stack                   |
|                                |
+--------------------------------+

+--------------------------------+
| Task B TCB                     |
+--------------------------------+
| Task B Stack                   |
|                                |
+--------------------------------+
```

This relationship is fundamental to RTOS memory analysis.

---

# 03.52 The Relationship Between Task, Context and Context Switch

The task owns an execution context.

When the CPU changes from one task to another:

```text
Task A
  |
  +---- context saved
  |
  v
Task B
  |
  +---- context restored
  |
  v
Running
```

Later:

```text
Task B
  |
  +---- context saved
  |
  v
Task A
  |
  +---- context restored
  |
  v
Continue Task A
```

The task does not restart from the beginning.

It resumes from its saved execution state.

That is the essence of multitasking.

---

# 03.53 Why Task Design Matters More Than API Knowledge

A developer can memorize:

```c
create_task();
delay_task();
delete_task();
```

but still design a poor RTOS system.

Good RTOS engineering requires understanding:

```text
Why does this need to be a separate task?

What is its timing requirement?

When should it run?

When should it wait?

What should wake it?

Who owns the resources it uses?

How much stack does it need?

What is its worst-case execution time?

What happens when it fails?

What happens if it blocks?

What happens if it runs longer than expected?
```

These are architectural questions.

The API is only the implementation mechanism.

---

# 03.54 Chapter Summary

A task is a **managed unit of execution** inside an RTOS.

It is not simply a function.

A useful model is:

```text
Task
 |
 +---- Code
 +---- Stack
 +---- Context
 +---- State
 +---- Kernel metadata
```

Tasks allow a complex embedded application to separate different responsibilities.

For example:

```text
Sensor Task
Motor Task
Communication Task
Diagnostics Task
Logging Task
```

Each task can have different timing behavior and can spend time either executing or waiting.

A task typically moves through states such as:

```text
            +-------+
            | READY |
            +---+---+
                |
                v
            +-------+
            |RUNNING|
            +---+---+
                |
        +-------+-------+
        |       |       |
      Delay   Event   Suspend
        |       |       |
        v       v       v
     BLOCKED BLOCKED SUSPENDED
        |
        | event/time
        v
      READY
```

The kernel maintains information about every task using a structure commonly called a **Task Control Block**, or TCB.

The task also has its own stack.

The stack is essential because the task must maintain its own runtime state, including function-call information, local data, and saved execution state.

A task's context represents the processor state required to continue execution correctly.

Typical context information includes:

```text
Program Counter
Stack Pointer
CPU Registers
Processor Status
Architecture-specific state
```

When the RTOS changes from one task to another, it performs a context switch:

```text
Task A
  |
  v
Save Context
  |
  v
Select Task B
  |
  v
Restore Context
  |
  v
Task B
```

Task creation establishes the resources and execution environment required for the task.

Task deletion removes it from active RTOS management, but task lifetime must be designed carefully because deleting a task that owns resources can leave the system in an invalid state.

Every task also has a limited stack.

If a task exceeds that stack, stack overflow can corrupt nearby memory and cause failures that may appear much later than the actual overflow.

Therefore stack sizing, monitoring, and testing are important parts of RTOS engineering.

Finally, good task design is about much more than creating many tasks.

A well-designed task should have:

```text
Clear responsibility
Clear timing behavior
Well-defined blocking behavior
Controlled resource ownership
Reasonable stack requirements
Appropriate priority
Defined failure behavior
```

The most useful mental model to carry forward is:

```text
                         TASK
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
         CODE            STACK            TCB
                                            |
                                            v
                                      Kernel State

                           |
                           v
                        CONTEXT
                           |
                           v
                    CPU can pause task
                    and resume it later
```

And the complete execution lifecycle can be visualized as:

```text
                    CREATE
                      |
                      v
                   READY
                      |
                      | scheduler selects
                      v
                  RUNNING
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
        DELAY       WAIT       SUSPEND
          |           |           |
          v           v           v
       BLOCKED     BLOCKED    SUSPENDED
          |           |           |
          +-----------+-----------+
                      |
                      | wake / resume
                      v
                    READY
                      |
                      v
                   RUNNING
                      |
                      v
                  TERMINATE
```

The key idea is simple:

> **A task is a complete, independently managed execution context that the RTOS can start, stop, wait, resume, schedule, and eventually terminate.**

Once this model is clear, the next scheduling concepts become much easier because the scheduler is essentially deciding **which task is eligible to execute and when that task should receive the CPU**.

# 04. Scheduling

Scheduling is the part of an RTOS that decides **which task should use the CPU and when**.

In a single-core microcontroller, this problem is fundamental because only one instruction stream can execute at a time. The system may have many tasks, but the CPU still has to choose which one gets execution.

Consider a small embedded product with:

```text
Sensor Task
Motor Control Task
Communication Task
Diagnostics Task
Logging Task
```

All five tasks may exist in memory at the same time.

But the CPU can execute only one of them at a particular instant.

That creates a basic question:

```text
                MANY TASKS
                    |
                    v
           +----------------+
           |      CPU       |
           +----------------+
                    ^
                    |
               Who runs now?
```

The answer is provided by the **scheduler**.

A scheduler is therefore not just a piece of code that "switches tasks." It is the mechanism that applies the operating system's scheduling rules to decide which eligible task should execute.

---

## Scheduler

The simplest way to understand a scheduler is to imagine several workers waiting to use one machine.

```text
Worker A ──┐
Worker B ──┤
Worker C ──┼──> One Machine
Worker D ──┤
Worker E ──┘
```

Only one worker can use the machine at a time.

Someone must decide:

```text
Who can work?
Who should work?
Who has to wait?
When should the next worker get the machine?
```

The scheduler performs the equivalent job for software tasks.

A simplified RTOS model is:

```text
              TASKS
                |
       +--------+--------+
       |        |        |
       v        v        v
     READY    BLOCKED   SUSPENDED
       |
       |
       v
   SCHEDULER
       |
       v
 SELECTED TASK
       |
       v
      CPU
```

The most important word in this diagram is **READY**.

A blocked task cannot be selected because it is waiting for something.

For example:

```text
Sensor Task
    |
    v
Waiting for next sample
    |
    v
BLOCKED
```

It is not a scheduling candidate until the reason for blocking is satisfied.

A ready task, on the other hand, is eligible to run.

Suppose:

```text
Task A → READY
Task B → READY
Task C → BLOCKED
```

The scheduler only needs to choose between Task A and Task B.

The choice depends on the scheduling policy.

---

### Scheduling is not the same as context switching

These two concepts are closely related, but they are not identical.

The **scheduler** decides:

```text
"Task B should run."
```

The **context-switch mechanism** performs the processor-state transition required to actually move execution from one task to another.

Conceptually:

```text
Task A running
      |
      v
Scheduler decision
      |
      v
Task B selected
      |
      v
Context switch
      |
      v
Task B running
```

So:

```text
Scheduling
    =
Decision

Context switching
    =
Mechanism used to change execution
```

This distinction becomes important when looking at RTOS internals.

---

### When does the scheduler get involved?

The scheduler may need to make a new decision when something changes in the system.

For example:

```text
A task becomes READY
A running task blocks
A delay expires
A synchronization event occurs
A higher-priority task becomes READY
A time slice expires
An interrupt causes application work to become ready
```

A simplified flow is:

```text
             EVENT
               |
               v
       System state changes
               |
               v
       Scheduler evaluates
               |
               v
       Select next task
               |
               v
             CPU
```

The exact trigger depends on the scheduling model and RTOS configuration.

---

# Preemptive Scheduling

Preemptive scheduling means that a running task can be **interrupted by the scheduler and replaced by another eligible task**.

The easiest example is priority-based preemption.

Suppose:

```text
Task A → Priority 2
Task B → Priority 5
```

Task A is currently running.

```text
Time -------------------------------------------------->

Task A
████████████████████████████████████████████
```

Now Task B becomes ready.

```text
Task B becomes READY
          |
          v
     Higher priority
          |
          v
      Preemption
```

The CPU may then execute:

```text
Time -------------------------------------------------->

Task A
███████████

Task B
           █████████████

Task A
                        █████████████████
```

Task A has not finished.

It has simply been interrupted so that Task B can execute.

After Task B finishes or blocks, Task A can continue.

This is one of the defining ideas of preemptive multitasking.

---

### Why preemption is useful

Suppose a low-priority logging task is performing some work:

```text
Logger Task
████████████████████████████████
```

Now a time-sensitive control event occurs:

```text
Motor Control Task
        |
        v
      READY
```

Without preemption, the motor task might have to wait for the logger to finish.

That could create unacceptable response time.

With preemption:

```text
Logger
██████████

Motor Control
          ███████

Logger
                 █████████████
```

The important work gets access to the CPU earlier.

This is especially valuable when different tasks have very different timing requirements.

---

### Preemption does not mean "interrupt everything immediately"

A common oversimplification is:

```text
High priority appears
      ↓
CPU switches instantly
```

Real systems are more complicated.

The actual response can depend on:

```text
Interrupt latency
Critical sections
Interrupt masking
Current CPU state
Scheduler configuration
Architecture
Kernel implementation
```

Therefore, preemptive scheduling provides a mechanism for fast response, but the actual response time still needs to be analyzed.

---

### Preemption and blocking

Preemption is especially useful when combined with blocking.

Imagine three tasks:

```text
Task A → High priority
Task B → Medium priority
Task C → Low priority
```

Task A may spend much of its time waiting:

```text
Task A
  |
  v
Wait for event
  |
  v
BLOCKED
```

While A is blocked, B can run.

When A becomes ready:

```text
Task A
    |
    v
  READY
    |
    v
Preemption
    |
    v
RUNNING
```

This allows the system to use CPU time efficiently.

The important concept is that **priority matters only among tasks that are eligible to run**.

A blocked high-priority task does not consume the CPU.

---

# Cooperative Scheduling

Cooperative scheduling follows a different model.

Here, a running task generally continues executing until it **voluntarily gives up the CPU**, usually by blocking, yielding, or reaching an RTOS-defined scheduling point.

Imagine:

```text
Task A
██████████████████████████
```

Task B becomes ready:

```text
Task B → READY
```

But Task A may continue:

```text
Task A
██████████████████████████
                         |
                         v
                    Task yields
                         |
                         v
Task B
                         ███████████
```

The key difference is that the running task participates in the decision to give up the CPU.

---

### Why cooperative scheduling can be useful

Cooperative scheduling is conceptually simpler in some systems.

If a task runs:

```c
while (1)
{
    do_small_piece_of_work();

    yield();
}
```

the task deliberately gives other tasks opportunities to execute.

This can reduce some scheduling complexity.

It can also be appropriate for systems where:

```text
Tasks are well-behaved
Execution sections are short
Timing requirements are modest
```

---

### The danger of cooperative scheduling

The biggest problem is a badly behaved task.

Suppose:

```c
while (1)
{
    do_long_operation();
}
```

and the task never yields or blocks.

Then other tasks may not get CPU time.

The system can effectively become:

```text
Task A
██████████████████████████████████████████████████
```

while:

```text
Task B → READY
Task C → READY
Task D → READY
```

are all waiting.

This is why cooperative systems depend heavily on task discipline.

A single task that monopolizes the CPU can damage system responsiveness.

---

### Preemptive vs Cooperative

The basic difference can be seen here:

```text
PREEMPTIVE

Task A running
██████████

Task B becomes READY
          |
          v
      Scheduler
          |
          v
Task B runs
██████████


COOPERATIVE

Task A running
██████████████████

Task B becomes READY
          |
          |
          | Task A continues
          |
          v
     Task A yields

Task B
          ████████
```

A useful mental model is:

```text
Preemptive:
    Scheduler can take the CPU away.

Cooperative:
    Task generally must give the CPU away.
```

---

# Priority-Based Scheduling

Priority-based scheduling assigns each task a priority level.

For example:

```text
Priority 5 → Motor Control
Priority 4 → Safety Monitoring
Priority 3 → Communication
Priority 2 → Display
Priority 1 → Logging
```

A common priority-based rule is:

> Among the eligible tasks, the highest-priority task gets the CPU.

The exact priority numbering convention differs between RTOSes. In some systems a larger number means higher priority; in others, a smaller number represents higher priority.

Therefore, never assume the numeric direction without checking the RTOS.

The important concept is the ordering.

---

### A simple example

Suppose:

```text
Task A → Priority 5 → READY
Task B → Priority 3 → READY
Task C → Priority 1 → READY
```

The scheduler sees:

```text
Priority
   ^
   |
  5 |  A   ← selected
  4 |
  3 |  B
  2 |
  1 |  C
   +------------------>
```

Task A is selected.

Now suppose Task A blocks:

```text
Task A → BLOCKED
Task B → READY
Task C → READY
```

Task B now becomes the highest-priority eligible task.

```text
        Scheduler
            |
            v
       Task B runs
```

The task selected by the scheduler therefore changes whenever task eligibility changes.

---

### Priority describes scheduling importance, not business importance

This is a subtle but important design idea.

Suppose:

```text
Logging Task
```

is extremely important from a business perspective.

That does not automatically mean it should be the highest-priority task.

Priority is mainly about **when a task needs CPU access relative to other tasks**.

A better way to think is:

```text
High priority
     |
     v
Needs earlier / more bounded CPU response

Low priority
     |
     v
Can tolerate waiting longer
```

This distinction prevents many poor RTOS designs.

---

### Giving every task high priority

Imagine:

```text
Task A → High
Task B → High
Task C → High
Task D → High
Task E → High
```

Now the priority structure tells us very little.

Priority is useful because it establishes an ordering.

If everything is urgent, then nothing is meaningfully prioritized.

A good priority scheme reflects actual system requirements.

---

# Round Robin

Round-robin scheduling is a way of sharing CPU time among multiple tasks at the same scheduling level.

Imagine three tasks with the same priority:

```text
Task A
Task B
Task C
```

Suppose all three are ready.

A simple round-robin sequence could be:

```text
A → B → C → A → B → C
```

Conceptually:

```text
Time -------------------------------------------------->

Task A     █████
Task B          █████
Task C               █████
Task A                    █████
Task B                         █████
Task C                              █████
```

The idea is to give each eligible task an opportunity to execute.

---

### Why round robin is useful

Suppose three communication-related tasks all have equal priority.

Without time-sharing, one task could potentially consume a large amount of CPU before another gets a chance.

Round robin introduces fairness among comparable tasks.

A useful analogy is a group of students sharing one computer:

```text
Student A → gets a turn
Student B → gets a turn
Student C → gets a turn
Student A → gets another turn
```

Nobody owns the machine forever.

---

### Round robin does not normally mean equal execution work

The CPU gives time, not completed work.

Suppose:

```text
Task A → simple calculation
Task B → heavy calculation
Task C → mostly waiting
```

Giving each a similar CPU opportunity does not mean they perform the same amount of useful work.

Round robin controls CPU access, not the amount of work completed.

---

### Round robin and priority

Round robin is usually discussed among tasks that are eligible and at the same scheduling level, depending on the RTOS.

For example:

```text
Priority 5
    |
    +---- Task A
    +---- Task B
    +---- Task C

Priority 3
    |
    +---- Task D
```

If the highest-priority tasks are all ready, the RTOS may rotate CPU access among A, B, and C.

Task D may not run until the higher-priority tasks block or otherwise become ineligible.

This creates the important relationship:

```text
Priority
    |
    v
Which group gets CPU?

Round Robin
    |
    v
How is CPU shared inside that group?
```

The exact behavior is RTOS-specific.

---

# Time Slicing

Time slicing is closely related to round robin but is not exactly the same concept.

**Time slicing** means that the scheduler may limit how long a task runs before giving another eligible task an opportunity to execute.

For example:

```text
Time slice = 5 ms
```

Suppose two equal-priority tasks are ready:

```text
Task A
Task B
```

A simplified timeline could be:

```text
Time -------------------------------------------------->

0ms        5ms        10ms       15ms       20ms
 |----------|----------|----------|----------|
 Task A       Task B      Task A      Task B
```

Each task receives a CPU slice.

---

### Why time slicing matters

Without time slicing, two equal-priority tasks may not naturally share the processor fairly if the first one does not block or yield.

With time slicing:

```text
Task A
██████████
          |
          | slice expires
          v
Task B
          ██████████
                    |
                    | slice expires
                    v
Task A
```

This improves fairness.

---

### Time slicing is different from preemption

This difference is important.

Preemption generally means:

```text
A task is replaced because another eligible task
has a scheduling reason to run.
```

Time slicing means:

```text
A task's allowed CPU interval expires,
so another eligible task may get a turn.
```

For example:

```text
High-priority task becomes READY
            |
            v
Preemption
```

versus:

```text
Same-priority tasks
       |
       v
Time slice expires
       |
       v
Another same-priority task gets CPU
```

Both can cause task switching, but the reason is different.

---

### Time slicing is not always enabled

Some systems disable time slicing because they want very explicit control over CPU usage.

For example, an RTOS may use:

```text
Priority-based preemption
+
Tasks block explicitly
```

without rotating same-priority tasks automatically.

Therefore, time slicing is a policy choice.

It is not a fundamental requirement of every RTOS.

---

# Priority Inversion

Priority inversion is one of the most important scheduling problems in an RTOS.

It happens when a **high-priority task is indirectly prevented from running because a lower-priority task holds a resource that the high-priority task needs**.

The problem becomes more interesting when a medium-priority task is involved.

Consider three tasks:

```text
High Priority    → H
Medium Priority  → M
Low Priority     → L
```

Suppose:

```text
L owns Mutex
H needs same Mutex
```

The sequence can become:

```text
1. L gets the mutex
2. H becomes ready
3. H tries to get the mutex
4. H must wait because L owns it
5. M becomes ready
6. M runs
7. L is prevented from running
8. H remains blocked
```

The surprising result is:

```text
H is waiting because of L
M is running
```

So a low-priority task has indirectly delayed a high-priority task.

---

## The classic diagram

```text
Priority

High      H
           |
           | waits for mutex
           v
        BLOCKED
           
Medium    M
           |
           | runs
           v
        RUNNING

Low       L
           |
           | owns mutex
           v
        READY / RUNNING
```

The timeline can look like:

```text
Time -------------------------------------------------->

Low Task L
██████████
           \
            \ owns mutex
             \
High Task H    becomes ready
               |
               v
             BLOCKED
               
Medium Task M
               █████████████████
               
Low Task L
                              █████
                              |
                              | releases mutex
                              v
High Task H
                                   ████████
```

This is priority inversion.

---

### Why this is dangerous

Suppose:

```text
High Task deadline = 1 ms
```

but:

```text
Low Task holds resource
Medium Task consumes CPU
```

The high-priority task may be blocked for much longer than expected.

Therefore priority inversion can turn a theoretically high-priority task into a task with poor response time.

This directly affects real-time analysis.

---

### The deeper lesson

Priority alone does not guarantee response.

A task can have the highest priority and still wait because of:

```text
Resource ownership
Synchronization
Interrupts
Critical sections
```

This is why real-time scheduling cannot be understood only by looking at task priorities.

Resource access also matters.

---

# Priority Inheritance

Priority inheritance is one technique used to reduce priority inversion.

The basic idea is:

> If a low-priority task holds a resource required by a high-priority task, temporarily raise the low-priority task's effective priority.

Consider:

```text
High → H
Medium → M
Low → L
```

Suppose L owns a mutex.

H needs the mutex.

Without priority inheritance:

```text
L owns mutex
      |
      v
H blocks
      |
      v
M runs
      |
      v
L waits longer
      |
      v
H waits longer
```

With priority inheritance:

```text
L owns mutex
      |
      v
H blocks on mutex
      |
      v
L inherits H's priority
      |
      v
L runs with high priority
      |
      v
L finishes critical section
      |
      v
L releases mutex
      |
      v
H runs
```

The important behavior is:

```text
H
|
| blocked by resource
v
L temporarily becomes more important
|
v
L releases resource quickly
|
v
H continues
```

---

## A timeline

Without inheritance:

```text
Time -------------------------------------------------->

L        ██████
H               blocked -----------------
M                    █████████████
L                                  ████
H                                       █████
```

With inheritance:

```text
Time -------------------------------------------------->

L        ██████
H               blocked
L                    █████████
                     ^
                     |
              Inherited priority

H                              █████████
```

The goal is not to make the low-priority task permanently high priority.

The elevated priority is temporary.

Once the resource is released and the blocking condition is resolved, the task's original priority can be restored according to the protocol.

---

### Important limitation

Priority inheritance reduces a specific type of priority inversion.

It does not solve every scheduling problem.

It does not automatically fix:

```text
Deadlocks
Starvation
Poor priority assignment
Long critical sections
Bad architecture
```

It is a synchronization protocol, not a universal scheduling solution.

---

# Priority Ceiling

Priority ceiling is another protocol used to control priority inversion and make resource access more predictable.

The core idea is to assign a **ceiling priority** to a shared resource.

That ceiling represents the highest priority of any task that may use the resource.

Imagine:

```text
Mutex A
Used by:
    Task L → Priority 2
    Task H → Priority 5

Ceiling of Mutex A = 5
```

When a task acquires the resource, the scheduling protocol can temporarily treat it according to the resource's ceiling rules.

The purpose is to prevent problematic priority relationships and bound blocking more tightly.

---

## Simple example

Suppose:

```text
Task H → Priority 5
Task M → Priority 3
Task L → Priority 1
```

All three may access:

```text
Shared Resource R
```

Then:

```text
Ceiling(R) = 5
```

When L acquires R:

```text
L
 |
 +---- owns R
 |
 +---- ceiling rules apply
```

The exact behavior depends on the particular priority-ceiling protocol and RTOS implementation.

The general goal is:

```text
Shared resource
      |
      v
Known maximum priority
      |
      v
More controlled blocking
```

---

## Why priority ceiling is useful

Priority inheritance usually reacts after a high-priority task becomes blocked.

Priority ceiling can place restrictions on resource access earlier, preventing some problematic scheduling situations from occurring.

This can improve:

```text
Predictability
Bounded blocking
Deadlock avoidance in some protocols
```

But it is more complex to design.

The engineer must correctly understand:

```text
Which tasks can use the resource?
What is the highest priority among them?
Which ceiling protocol is being used?
What are the locking rules?
```

Therefore priority ceiling is often seen in systems where formal scheduling analysis is important.

---

# Scheduling Analysis

So far, scheduling has been described conceptually.

Real-time engineering eventually requires something more precise:

> **Can the software actually meet its timing requirements?**

This is the purpose of scheduling analysis.

Suppose we have three periodic tasks:

```text
Task A:
Period = 10 ms
Execution = 1 ms
Deadline = 10 ms

Task B:
Period = 20 ms
Execution = 2 ms
Deadline = 20 ms

Task C:
Period = 50 ms
Execution = 5 ms
Deadline = 50 ms
```

At first glance, the system appears manageable.

We can estimate processor utilization:

```text
U = C1/T1 + C2/T2 + C3/T3
```

where:

```text
C = execution time
T = period
```

So:

```text
Task A = 1 / 10 = 0.10
Task B = 2 / 20 = 0.10
Task C = 5 / 50 = 0.10
```

Therefore:

```text
Total utilization = 0.30
                 = 30%
```

This means the periodic task workload consumes approximately 30% of one CPU's capacity under the simplified assumptions.

That is useful information, but it is not enough by itself to prove schedulability.

---

## Why utilization alone is not enough

Imagine:

```text
Task A
Execution = 8 ms
Period    = 10 ms
```

and:

```text
Task B
Execution = 1 ms
Period    = 100 ms
```

Utilization is:

```text
8/10 + 1/100
= 0.8 + 0.01
= 81%
```

That tells us the average CPU demand.

But the real question is:

```text
Can every task finish before its deadline
when tasks interfere with each other?
```

That requires more analysis.

---

# Scheduling Timeline Analysis

Suppose:

```text
High-priority Task H
Execution = 2 ms
Period = 10 ms

Low-priority Task L
Execution = 5 ms
Period = 20 ms
```

A simplified schedule could look like:

```text
Time ------------------------------------------------------>

0       2       7       10      12      20
|-------|-------|--------|-------|-------|

H       H
████████

L       LLLLL
        █████████████

H                       HHH
                        █████

L                               LLLLL
                                █████
```

The engineer can ask:

```text
When does H start?
When does H finish?
How much can L delay H?
Does H meet its deadline?
```

This is the beginning of real scheduling analysis.

---

# Worst-Case Execution Time

Scheduling analysis depends heavily on execution-time assumptions.

If:

```text
Task A typical = 1 ms
```

but:

```text
Task A worst case = 4 ms
```

then analysis based on 1 ms may be unsafe.

Real-time analysis therefore often uses:

```text
WCET
```

or another conservative execution-time bound.

Conceptually:

```text
Execution time
|
|                         Worst case
|                             *
|                           *
|               Typical   *
|                 *      *
|______________*____________________ Time
```

The goal is not to predict exactly what happens every time.

The goal is to understand the maximum relevant demand.

---

# Blocking Time

Scheduling analysis also has to consider blocking.

Suppose a high-priority task needs a mutex:

```text
High Task H
     |
     v
Needs resource
     |
     v
Resource held by Low Task L
```

Then H may be delayed even though H has higher priority.

Therefore response-time analysis may need to consider:

```text
Execution time
+
Higher-priority interference
+
Blocking time
+
Interrupt overhead
+
Scheduler overhead
```

A conceptual formula is:

```text
Response Time
=
Execution
+
Interference
+
Blocking
+
Other relevant overhead
```

The exact mathematical analysis depends on the scheduling model.

---

# Preemption and Scheduling Analysis

Preemptive systems require careful reasoning about how tasks interfere with each other.

Suppose:

```text
Task A → High priority
Task B → Low priority
```

Task B starts first:

```text
B
████████████████████
```

Then A arrives:

```text
A becomes READY
      |
      v
B is preempted
```

The timeline becomes:

```text
B        █████
A             ███████
B                    ███████
```

Task B's execution is split into pieces.

This matters when measuring:

```text
CPU utilization
Execution response
Cache behavior
Resource holding time
```

and when calculating worst-case response times.

---

# Cooperative Scheduling Analysis

Cooperative systems have a different analysis model.

If a task can hold the CPU for a long time before yielding:

```text
Task A
█████████████████████████████
```

then another task may wait that entire interval.

Therefore a cooperative system needs careful upper bounds on:

```text
Maximum non-preemptive execution section
```

This becomes an important part of response-time analysis.

---

# Scheduling Analysis Is About the Whole System

It is not enough to look at one task in isolation.

Consider:

```text
Task execution
    +
Task priority
    +
Task period
    +
Task deadline
    +
Interrupts
    +
Shared resources
    +
Blocking
    +
Kernel overhead
    +
CPU frequency
```

All of these can affect whether the system meets its requirements.

A more complete model is:

```text
                    SYSTEM TIMING
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
    Task Work        Interrupts      Resource Blocking
        |                |                |
        +----------------+----------------+
                         |
                         v
                   Scheduling
                         |
                         v
                    Response Time
                         |
                         v
                      Deadline
```

---

# A Practical Scheduling Example

Consider a motor-control system:

```text
Task                  Period     WCET
------------------------------------------------
Motor Control         1 ms       100 µs
Sensor Processing     5 ms       500 µs
Communication        10 ms       1 ms
Diagnostics          100 ms      2 ms
Logging              100 ms      3 ms
```

The system has one CPU.

The first useful calculation is approximate utilization:

```text
Motor:
0.1 ms / 1 ms = 10%

Sensor:
0.5 ms / 5 ms = 10%

Communication:
1 ms / 10 ms = 10%

Diagnostics:
2 ms / 100 ms = 2%

Logging:
3 ms / 100 ms = 3%
```

Total:

```text
35%
```

That looks comfortable.

But the engineer must still analyze:

```text
Interrupt overhead
Task priority
Blocking
Deadline
Burst behavior
Long critical sections
Communication bursts
Startup behavior
```

For example, if the motor-control task needs a resource that can be locked by the communication task for 400 µs, then the effective worst-case response can be very different from what the simple utilization calculation suggests.

This is why:

```text
CPU utilization
```

and:

```text
Schedulability analysis
```

are related but not identical.

---

# The Scheduling Mindset

A strong RTOS engineer does not look only at:

```text
"Which task has higher priority?"
```

The real questions are deeper:

```text
Which tasks are ready?

Which task is running?

Which task is blocked?

What caused a task to become ready?

Can another task preempt it?

Can it be blocked by a shared resource?

How long can that blocking last?

How frequently does each task execute?

What is the worst-case execution time?

What is the task's deadline?

How much CPU capacity does the system require?

What happens during the worst combination of events?
```

This is the difference between simply knowing how to configure priorities and actually understanding scheduling.

---

# A Single Picture for the Entire Chapter

All of Chapter 04 can be connected through this diagram:

```text
                         TASKS
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          READY         BLOCKED       SUSPENDED
             |
             v
        +-----------+
        | SCHEDULER |
        +-----+-----+
              |
              v
       Scheduling Policy
              |
      +-------+--------+
      |                |
      v                v
 Preemptive        Cooperative
      |
      v
 Priority
      |
      +--------------------+
      |                    |
      v                    v
 Time Slice            Round Robin
      |
      v
 Shared Resources
      |
      v
 Priority Inversion
      |
      +-----------------------+
      |                       |
      v                       v
Priority Inheritance    Priority Ceiling
      |
      v
 Scheduling Analysis
      |
      v
 Can deadlines be met?
```

The entire purpose of scheduling is ultimately:

```text
Multiple tasks
      |
      v
Limited CPU
      |
      v
Scheduling policy
      |
      v
Controlled execution
      |
      v
Predictable response
      |
      v
Meet system deadlines
```

---

# Chapter Summary

Scheduling is the mechanism through which an RTOS decides which eligible task should receive CPU execution.

The scheduler works with the task states introduced in the previous chapter:

```text
READY
RUNNING
BLOCKED
SUSPENDED
```

Only eligible tasks participate in the scheduling decision.

A preemptive scheduler can interrupt a running task when another task becomes eligible and should run according to the scheduling policy.

A cooperative scheduler generally depends on the running task to voluntarily give up the CPU.

Priority-based scheduling orders eligible tasks according to priority.

Round-robin scheduling can distribute CPU time among comparable ready tasks.

Time slicing limits how long an eligible task can continue before another comparable task gets an opportunity to execute.

These mechanisms create useful CPU-sharing behavior, but they also introduce important real-time problems.

Priority inversion occurs when a high-priority task is indirectly delayed by a lower-priority task holding a required resource.

Priority inheritance temporarily raises the resource holder's effective priority so that it can finish the critical section sooner.

Priority ceiling uses a resource's defined ceiling priority to provide stronger control over resource access and blocking.

Finally, scheduling analysis determines whether the complete system can meet its timing requirements.

A useful conceptual model is:

```text
                 SCHEDULING
                     |
          +----------+----------+
          |                     |
          v                     v
      CPU Selection        Timing Analysis
          |                     |
          v                     v
     Task executes         Deadline check
          |                     |
          +----------+----------+
                     |
                     v
               System response
```

The ultimate goal is not simply to make tasks run.

The goal is:

```text
                    System Work
                        |
                        v
                 Scheduling Rules
                        |
                        v
                  CPU Execution
                        |
                        v
                Predictable Timing
                        |
                        v
                 Meet Deadlines
```

A well-designed RTOS scheduler therefore provides the foundation for **responsive, predictable, and analyzable embedded software**.

The next level of understanding is to move from "which task runs?" to **"how do tasks safely communicate and coordinate with each other while they are running?"**
