# Debugging Interview Questions
> C Language + Embedded Systems + RTOS  
> M.Tech Graduate + 10 Years Experience  
> Basic → Intermediate → Advanced → Expert → War Stories

---

## BASIC LEVEL

------------------------------------------------------------------------

## 1. What is debugging?

Debugging is the process of finding the reason a program behaves
incorrectly and then correcting that reason. A program may crash, give a
wrong result, respond too slowly, get stuck in a loop, or behave
differently when several tasks run at the same time. Debugging is not
just changing code until the problem disappears. A good engineer
collects evidence, forms a hypothesis, tests it, identifies the root
cause, applies a fix, and verifies that the fix works without creating
another problem.

For example, suppose an embedded device should turn on a motor when a
sensor value exceeds 500, but the motor never starts. Debugging may
involve checking whether the sensor value is read correctly, whether the
comparison is correct, whether the motor-control function is called,
whether the GPIO is configured correctly, and whether the hardware
driver is working.

A typical debugging process is:

``` text
Observe the problem
        |
        v
Reproduce the problem
        |
        v
Collect evidence and narrow the scope
        |
        v
Find the root cause
        |
        v
Apply a small, targeted fix
        |
        v
Verify the fix and run regression tests
```

### Follow-up: What is the difference between debugging and testing?

**Testing** checks whether software behaves as expected for selected
inputs and conditions. It is intended to reveal failures. **Debugging**
begins when a failure or suspicious behavior needs an explanation; it
investigates the cause and helps correct it.

Example: A test sends a sensor value of 600 and observes that the motor
does not start. That is testing. Inspecting the sensor reading,
condition, GPIO state, and driver call to discover that the code uses
`>` incorrectly in a boundary case---or that the pin is not configured
as an output---is debugging.

Testing and debugging support each other. After debugging and fixing the
issue, tests should be run again to confirm the fix and check that
existing behavior still works.

### Follow-up: What is the difference between debugging and profiling?

**Debugging** investigates incorrect behavior. **Profiling** measures
how a program uses resources such as CPU time, memory, stack, or
execution time. Profiling helps identify performance bottlenecks, while
debugging helps explain faults. They can overlap: a profiler may reveal
that one task consumes too much CPU, and debugging may then explain why
that task runs continuously.

Example: If a task takes 20 ms when it should take 2 ms, profiling can
identify which function consumes most of the time. Debugging can then
reveal that the function repeatedly polls a peripheral because a status
flag never changes.

### Follow-up: What is a bug vs defect vs fault vs failure?

These terms are sometimes used differently across companies and
standards, so explain the distinction clearly rather than claiming every
organization uses exactly the same definitions.

-   **Bug:** An informal term for a problem in software or its behavior.
-   **Defect:** A flaw in a requirement, design, code, or other work
    product. For example, a wrong comparison in the source code.
-   **Fault:** The underlying incorrect condition that can cause a
    failure. In software discussions, it may refer to the defect or
    faulty logic that is present in the system.
-   **Failure:** The externally observable inability of the system to
    perform a required function.

Example: The code has a defect that converts a temperature incorrectly
(fault). When the temperature rises, the controller reports a safe value
and fails to switch off the heater (failure). The team may casually call
the entire issue a bug.

------------------------------------------------------------------------

## 2. What are the types of bugs?

Bugs can be grouped by where they occur or how they appear. These
categories can overlap: a memory bug can cause a runtime crash, and a
concurrency bug can create a timing-dependent failure.

### 2.1 Syntax bugs

A syntax bug occurs when code does not follow the grammar of the C
language. The compiler usually reports it before the program runs.

``` c
int count = 10
printf("%d\n", count);  // Missing semicolon above
```

The compiler normally reports an error near the missing semicolon. Fix
the reported line and also inspect nearby code, because the actual
mistake can sometimes be just before the line identified by the
compiler.

### 2.2 Logical bugs

A logical bug occurs when the program compiles and runs, but the code
does the wrong thing.

``` c
if (temperature < 80) {
    turn_heater_off();  // Possibly wrong logic for the intended requirement
}
```

If the requirement is to switch the heater off when temperature reaches
80 degrees or more, the condition is wrong. The compiler cannot usually
determine the intended business or control logic. A requirement-based
test and careful review are needed.

### 2.3 Runtime bugs

A runtime bug appears while the program is executing. Examples include
dividing by zero, accessing an invalid pointer, overflowing a stack, or
calling a function with invalid state. In embedded systems, a runtime
bug may cause a HardFault, reset, hang, or corrupted output.

``` c
int *ptr = NULL;
*ptr = 10;  // Invalid memory access; behavior is undefined
```

Use the debugger, fault registers, logs, assertions, and input checks to
locate the cause. Not every runtime bug produces a clean crash; some
silently corrupt data.

### 2.4 Memory bugs

Memory bugs involve incorrect use of memory. Common examples include:

-   **Buffer overflow:** Writing beyond the allocated size of an array.
-   **Out-of-bounds read:** Reading beyond an array's valid elements.
-   **Use-after-free:** Using memory after it has been released in
    systems that support dynamic allocation.
-   **Uninitialized data:** Reading a variable before assigning a valid
    value.
-   **Invalid pointer:** Dereferencing a null, corrupted, or otherwise
    invalid pointer.
-   **Stack overflow:** Using more stack space than a task or program
    has available.
-   **Memory leak:** Allocating memory but failing to release it when it
    is no longer needed.

Example:

``` c
int values[4] = {0, 1, 2, 3};
values[4] = 99;  // Out of bounds: valid indexes are 0 through 3
```

This is dangerous even if the device does not crash immediately. The
write may corrupt another variable, a control structure, or task data.
Use bounds checks, static analysis, compiler warnings, memory protection
where available, and stack-watermark monitoring.

### 2.5 Timing bugs

A timing bug occurs when behavior depends on when an event happens, how
long an operation takes, or whether a deadline is met. Examples include
missing a sensor sample, responding too late to an interrupt, or using a
delay that is too short for a peripheral.

A timing bug may disappear when a debugger pauses the CPU or when extra
logging is added. This makes timing behavior especially important in
embedded systems.

Example: Firmware reads a sensor 5 ms after starting a conversion, but
the sensor requires 10 ms. The returned value may be stale or invalid.
The fix is to follow the peripheral's timing requirements and use a
suitable completion flag, interrupt, or timer instead of guessing with a
delay.

### 2.6 Concurrency bugs

A concurrency bug occurs when multiple tasks, threads, or interrupt
handlers access shared state in an unsafe or unexpected order. Common
examples are race conditions, deadlocks, missed signals, and priority
inversion.

``` c
volatile int counter = 0;

/* Task A */ counter++;
/* Task B */ counter++;
```

Even if `counter` is declared `volatile`, the increment is not
necessarily atomic. Each increment may involve reading, adding, and
writing. The two tasks can interfere, causing an update to be lost.
Protect shared data with an appropriate mutex, critical section, atomic
operation, or ownership design.

### Follow-up: Which type of bug is hardest to debug, and why?

There is no single answer for every system, but **intermittent timing
and concurrency bugs** are often among the hardest. They may depend on
exact scheduling, interrupt timing, compiler optimization, hardware
state, or rare input sequences. Adding a breakpoint or log can change
the timing and make the bug disappear.

A strong debugging approach is to: - Record timestamps and event order
with low-overhead instrumentation. - Capture task states, interrupt
activity, and relevant shared variables. - Reproduce the issue under
controlled stress. - Reduce the system to the smallest case that still
shows the failure. - Avoid changing several things at once. - Verify the
suspected root cause with a targeted experiment.

Memory corruption can be equally difficult because the visible crash may
occur long after the original invalid write.

### Follow-up: Which bugs does the compiler catch, and which does it miss?

A compiler can catch many syntax errors, type mismatches, undeclared
identifiers, and some suspicious constructs. With suitable warning
levels and static-analysis tools, it can also flag some uninitialized
variables, unreachable code, suspicious conversions, and possible array
or pointer problems.

A compiler generally cannot reliably infer whether the application logic
matches the requirement, whether a sensor is wired correctly, whether a
deadline is met, or whether a particular task interleaving causes a
race. Some undefined behavior and memory mistakes may also escape
detection.

Enable warnings such as `-Wall -Wextra` when supported by the compiler,
treat important warnings seriously, use static analysis, and combine
these tools with testing and hardware-level validation. Exact warning
options differ by compiler.

------------------------------------------------------------------------

## 3. What is the first step when debugging an unknown crash?

The first step is to **preserve the evidence and establish what actually
happened**. Do not immediately change code or reset the device
repeatedly without recording useful information. Note the exact
symptoms, the firmware build, the input or operating conditions, the
time of occurrence, and any available fault registers, logs, reset-cause
flags, or crash dumps.

For a microcontroller, useful first checks include: - Was the reset
caused by a watchdog, power-on reset, software reset, or fault? - Is the
program counter (PC) or fault status available? - Did a HardFault, bus
fault, memory fault, or usage fault occur? - What were the stack pointer
(SP), link register (LR), and relevant registers? - Was the system
handling an interrupt or switching tasks? - Did the failure follow a
recent code, configuration, or hardware change?

### Follow-up: Explain Reproduce → Isolate → Identify → Fix → Verify

1.  **Reproduce:** Find a reliable sequence of steps or conditions that
    causes the problem.
2.  **Isolate:** Narrow the problem to a module, function, task,
    interrupt, input, or hardware interface.
3.  **Identify:** Use evidence to determine the root cause rather than
    just the visible symptom.
4.  **Fix:** Make a focused change that addresses the cause.
5.  **Verify:** Repeat the failing case, run related tests, and check
    that the change has not introduced regressions.

Example: A device sometimes resets when transmitting a large message.
Reproduce the reset with large messages, isolate the transmission path,
inspect the stack and buffer bounds, identify an undersized buffer,
increase or correctly validate the buffer capacity, and test both
boundary-size and normal messages.

### Follow-up: What if you cannot reproduce the bug?

An intermittent bug still deserves a structured investigation. Collect
more evidence in the field and preserve the system state before it is
lost.

-   Add low-overhead event tracing or a circular log buffer in RAM.
-   Record reset causes, fault registers, timestamps, task IDs, and key
    state transitions.
-   Capture input values and configuration that preceded the issue.
-   Increase stress testing or vary timing and task load in a controlled
    way.
-   Compare the failing and working firmware builds.
-   Check for stack exhaustion, data races, memory corruption, power
    instability, and peripheral timing.
-   Use a watchdog or fault handler to preserve diagnostic information
    before reset, where safe and supported.

Avoid relying only on `printf`, because printing can change timing. If a
bug occurs only in optimized builds, check optimization-sensitive
undefined behavior, data races, uninitialized variables, and assumptions
about `volatile`.

------------------------------------------------------------------------

## 4. What is a breakpoint?

A breakpoint is a debugging mechanism that pauses program execution at a
selected instruction or location so the engineer can inspect the system.
At a breakpoint, a debugger may let you examine variables, registers,
memory, the call stack, and the current instruction. On a
microcontroller, pausing the CPU may also affect peripherals,
interrupts, watchdogs, and real-time behavior, so a breakpoint is not
always harmless.

Example: Place a breakpoint at `process_sensor_data()` and run the
firmware. When execution reaches that function, inspect the sensor value
and the state used by the control logic.

### Follow-up: Hardware breakpoint vs software breakpoint?

**Hardware breakpoint:** Uses debug hardware in the processor to detect
execution at a particular address. It normally does not need to replace
the instruction in program memory. The number of available hardware
breakpoint comparators is limited.

**Software breakpoint:** Often works by replacing an instruction with a
special breakpoint instruction (such as `BKPT` on ARM). This may be easy
to use in RAM, but changing flash contents can be limited or require
debugger support. Exact behavior depends on the architecture and debug
probe.

On Cortex-M systems, debugger support and the target's debug resources
determine which breakpoint type is used. Hardware breakpoints are
valuable when debugging code in flash or when modifying program memory
is undesirable.

### Follow-up: How many hardware breakpoints does ARM Cortex-M4 support?

Do not treat **six** as a universal Cortex-M4 guarantee. The Cortex-M4
architecture uses Flash Patch and Breakpoint (FPB) hardware, but the
number of instruction breakpoint comparators is implementation-dependent
and should be checked in the specific device's documentation or through
the debug tool. Some devices expose six instruction breakpoint
comparators; others may differ.

In an interview, a careful answer is: "The number is
implementation-dependent. I would check the MCU reference manual or
debugger's reported resources instead of assuming every Cortex-M4 has
six."

### Follow-up: What is a watchpoint (data breakpoint)?

A watchpoint monitors access to a selected memory address or range.
Depending on the processor and debugger, it can halt execution when the
memory is read, written, or accessed in a specified way. It is useful
when a variable changes unexpectedly and you do not know which code is
changing it.

Example: A variable `motor_state` unexpectedly becomes zero. Set a write
watchpoint on its address. When a write occurs, the debugger stops at
the instruction responsible, making it easier to find the unexpected
modification.

Watchpoints are limited in number and supported access sizes. Some tools
offer software watchpoints, but these can be slower or less reliable
than hardware watchpoints.

------------------------------------------------------------------------

## 5. What is single stepping?

Single stepping means executing the program one instruction or
source-level step at a time while using a debugger to inspect what
happens. It helps you follow control flow and see how variables and
registers change. Depending on the debugger settings, one source-level
step may correspond to several machine instructions.

### Follow-up: Step over vs step into vs step out?

-   **Step into:** Executes the current line and enters a called
    function, allowing you to debug inside it.
-   **Step over:** Executes the current line, including a function call,
    without stopping inside that function; execution pauses at the next
    source line in the current function.
-   **Step out:** Continues until the current function returns to its
    caller.

Example:

``` c
int read_sensor(void)
{
    return 42;
}

int main(void)
{
    int value = read_sensor();
    value++;
}
```

If the debugger is on `int value = read_sensor();`, **step into** enters
`read_sensor()`, **step over** runs the call and pauses at `value++`,
and **step out** from inside `read_sensor()` runs until the function
returns.

### Follow-up: Why can single stepping change timing-related bug behavior?

Single stepping slows the program dramatically. While execution is
paused, interrupts may still run or may be masked depending on the
debugger and target settings. Peripheral timers may continue,
communication devices may keep operating, and watchdogs may expire. As a
result, a race condition may disappear, a timeout may occur, or a
real-time deadline may be missed.

For timing-sensitive problems, use trace, GPIO timing markers,
logic-analyzer captures, hardware timers, or a low-overhead circular
buffer instead of relying only on single stepping.

------------------------------------------------------------------------

## 6. What is a watchdog timer?

A watchdog timer is a hardware or software timer used to detect when the
system is no longer making acceptable progress. Firmware must service,
refresh, or "feed" the watchdog according to the configured rules. If
the watchdog is not serviced in time, it triggers a defined action,
commonly a reset or interrupt.

A watchdog is useful for recovering from certain hangs, deadlocks, or
unexpected control-flow failures. It does not prove that the software is
correct: a program can continue feeding the watchdog while doing the
wrong work.

Example:

``` text
Firmware performs useful work
          |
          v
Health checks pass
          |
          v
Watchdog is serviced
          |
          +---- Repeat

If useful progress stops and the watchdog is not serviced:
          |
          v
Watchdog expires -> interrupt or reset
```

### Follow-up: How does it help detect system hangs?

If firmware becomes stuck in an infinite loop, deadlocks, or stops
reaching the code that services the watchdog, the timer expires. The
device can then reset or enter a recovery path instead of remaining
stuck indefinitely.

The watchdog should be serviced only after meaningful health checks. In
an RTOS, a central supervisor may check that important tasks have
reported progress before refreshing the hardware watchdog. If any
critical task stops progressing, the supervisor should withhold the
refresh.

A poor design refreshes the watchdog from a periodic interrupt
regardless of application health. That interrupt may continue running
even while the main application is deadlocked, preventing the watchdog
from detecting the failure.

### Follow-up: What is a window watchdog vs a basic watchdog?

A **basic watchdog** generally requires servicing before a timeout. If
servicing does not happen in time, the watchdog expires.

A **window watchdog** restricts servicing to an allowed time window.
Servicing too early or too late can trigger a fault, depending on the
implementation. This helps detect both missed deadlines and unexpected
execution patterns, such as code running too quickly through a loop.

The exact behavior and whether early servicing causes an immediate reset
depend on the MCU's watchdog implementation. Always check the reference
manual.

### Follow-up: What should you do in a watchdog reset handler?

First, distinguish between a watchdog interrupt handler and startup code
that runs after a watchdog-caused reset. Many watchdogs reset the
processor without giving firmware a normal handler opportunity. The
design must account for that.

Useful practices include: - Read and preserve the reset-cause flags
early during startup, before they are cleared. - Store a small, robust
diagnostic record in retained RAM or nonvolatile storage if the platform
supports it. - Record a reset counter and relevant fault information,
while avoiding excessive flash writes that could wear out storage. -
Initialize hardware and software into a safe state after reset. -
Determine whether the failure is recurring and provide a controlled
recovery or safe mode when needed. - Avoid immediately feeding the
watchdog forever without checking whether the system has recovered. -
Keep interrupt or fault handlers short and safe; do not perform complex
logging or blocking operations in a context that cannot support them.

For safety-critical systems, recovery behavior should be defined by the
safety requirements. A watchdog reset is evidence of a problem to
investigate, not proof that the root cause has been fixed.

------------------------------------------------------------------------

## 7. What is printf debugging?

`printf` debugging means adding print statements to display variable
values, program flow, error codes, and state transitions. It is a simple
way to observe what the firmware is doing without stopping execution at
breakpoints.

Example:

``` c
printf("temperature=%d, state=%d\n", temperature, state);
```

The output may be sent through a UART serial port, a debugger console, a
semihosting channel, or another configured output path. The method is
useful for simple applications and early investigation, but the output
path and formatting can consume significant time and memory.

### Follow-up: What are the advantages and disadvantages?

**Advantages** - Easy to add and understand. - Shows the values observed
while the program is running. - Can help investigate failures that are
difficult to reproduce manually. - Can be retained in a log or captured
by a host computer. - Often requires less debugger expertise than
advanced trace tools.

**Disadvantages** - Printing can change the timing and hide race
conditions. - UART output can be slow, especially at low baud rates. -
Formatting functions can increase code size and stack usage. - Printing
from an interrupt can cause long interrupt latency or deadlock if the
output path blocks. - Logs may overflow buffers or be lost when the
system resets. - Sensitive information can be exposed if logs are not
controlled. - Excessive logging can make a real-time system miss
deadlines.

A better approach is to log concise events, use timestamps or sequence
numbers, and buffer data for later transmission when possible.
Compile-time log levels can help disable verbose output in production
builds.

### Follow-up: When is printf debugging NOT acceptable in embedded systems?

Avoid or tightly control `printf` when it can violate timing, safety, or
resource constraints, such as:

-   Inside a high-priority interrupt or a time-critical ISR.
-   In a hard real-time path where blocking output can miss a deadline.
-   In a low-memory device where formatting adds too much code or stack
    use.
-   When the output driver takes a mutex that may already be held by the
    interrupted task.
-   In low-power code where output prevents the system from sleeping.
-   In a fault handler where normal drivers, locks, or memory may
    already be corrupted.
-   In production systems where logs may expose secrets or sensitive
    operational data.

For these cases, consider a fixed-size binary event buffer, a GPIO
timing pulse, SWO/ITM trace where supported, or a dedicated trace
system. If logging is necessary, make it non-blocking and safe for the
execution context.

### Follow-up: What is semihosting?

Semihosting allows embedded firmware to request services from a host
computer through the debugger. Examples include writing text to a debug
console or performing certain file operations. The debugger intercepts
special instructions or exceptions and carries out the requested host
operation.

Semihosting is convenient during development, but it can be very slow
and may halt the target while the debugger handles the request. It often
depends on an attached debugger and is usually unsuitable for
timing-critical production firmware. If the debugger is absent, behavior
depends on the target's semihosting configuration and exception
handling.

### Follow-up: What is ITM (Instrumentation Trace Macrocell), and why can it be better than UART printf?

The **Instrumentation Trace Macrocell (ITM)** is an ARM CoreSight debug
component available on supported Cortex-M implementations. Firmware can
write instrumentation data to ITM stimulus ports, and a compatible debug
probe can capture that data through a supported trace path, often Serial
Wire Output (SWO).

Compared with UART-based `printf`, ITM/SWO can provide a convenient
debug output channel without using the same UART peripheral needed by
the application. It can also support timestamped or structured trace
workflows, depending on the core, probe, and tools. It may have lower
overhead than a blocking UART print, but it is not automatically free or
non-blocking.

Important limitations: - ITM and SWO support depends on the specific
MCU, debug probe, clock configuration, and development tools. - Trace
bandwidth is limited; excessive data can be lost or blocked depending on
configuration. - Instrumentation still consumes CPU time and can affect
timing. - ITM is a debug/trace facility, not a universal replacement for
production logging. - Unlike a simple UART, it may require a compatible
debugger and trace configuration.

**Interview summary:** "UART `printf` is widely available but can be
slow and intrusive. Semihosting routes output through the debugger and
is often very slow. ITM/SWO can provide lower-overhead trace on
supported Cortex-M systems, but its bandwidth and hardware/tool support
must be checked."

------------------------------------------------------------------------

## Quick Revision Table

  -----------------------------------------------------------------------
  Topic                   Main idea               Common caution
  ----------------------- ----------------------- -----------------------
  Debugging               Find and fix the root   Do not confuse a
                          cause of incorrect      symptom with the cause
                          behavior                

  Testing                 Check behavior against  Tests reveal failures;
                          expected results        they do not
                                                  automatically explain
                                                  them

  Profiling               Measure resource usage  A slow function is not
                          and performance         necessarily incorrect

  Syntax bug              Code violates language  Usually caught by the
                          grammar                 compiler

  Logical bug             Code runs but does the  Requires
                          wrong thing             requirements-based
                                                  reasoning and tests

  Memory bug              Invalid or unsafe       Corruption may appear
                          memory use              far from the original
                                                  mistake

  Timing bug              Behavior depends on     Debugger pauses and
                          timing or deadlines     logs can change
                                                  behavior

  Concurrency bug         Tasks/interrupts        `volatile` alone does
                          interact unsafely       not make shared updates
                                                  atomic

  Breakpoint              Pause at an             Limited resources; can
                          instruction/location    alter real-time
                                                  behavior

  Watchpoint              Stop when a memory      Hardware watchpoints
                          location is accessed    are limited

  Single stepping         Execute incrementally   Often unsuitable for
                          under debugger control  timing investigations

  Watchdog                Detect lack of expected Must be serviced based
                          progress                on health, not blindly

  `printf` debugging      Print values and events Can block, use stack,
                                                  and alter timing

  Semihosting             Send requests through   Usually slow and
                          the host debugger       debugger-dependent

  ITM/SWO                 Hardware-assisted       Requires compatible
                          instrumentation trace   hardware and
                                                  configuration
  -----------------------------------------------------------------------


---

### Basic Tools

8. What is GDB?
   - Follow-up: Basic GDB commands:
   ```bash
   gdb ./program          # start
   run                    # run
   break main             # breakpoint at function
   break file.c:42        # breakpoint at line
   next                   # step over
   step                   # step into
   finish                 # step out
   print x                # print variable
   info locals            # all local variables
   backtrace              # call stack
   continue               # continue execution
   watch x                # watchpoint on variable
   info breakpoints       # list breakpoints
   delete 1               # delete breakpoint 1
   quit                   # exit
   ```

9. What is a core dump?
   - Follow-up: How to enable core dump on Linux?
   ```bash
   ulimit -c unlimited
   ./program        # crashes → generates core
   gdb program core # analyze
   ```
   - Follow-up: What information is in a core dump?
   - Follow-up: How to read backtrace from core dump?

10. What is `valgrind`?
    - Follow-up: What errors does it detect?
    - Follow-up: Command: `valgrind --leak-check=full ./program`
    - Follow-up: Performance overhead of valgrind? (~10-50x slower)

---

## INTERMEDIATE LEVEL

### C Debugging

1. How do you debug a segmentation fault?
   - Step 1: Compile with `-g` flag.
   - Step 2: Run under GDB or get core dump.
   - Step 3: `backtrace` to find crash location.
   - Step 4: `print` variables to find bad pointer.
   - Follow-up: Common causes:
     - NULL pointer dereference
     - Dangling pointer
     - Buffer overflow
     - Stack overflow
     - Writing to read-only memory (string literal)

2. How do you debug a memory leak in C?
   - Follow-up: Valgrind `--leak-check=full`
   - Follow-up: AddressSanitizer (ASan)
   - Follow-up: Manual tracking: wrapper around malloc/free with logging
   - Follow-up: What is LSAN (LeakSanitizer)?

3. What is AddressSanitizer (ASan)?
   ```bash
   gcc -fsanitize=address -g program.c -o program
   ./program
   ```
   - Follow-up: What bugs does ASan detect?
     - Heap buffer overflow
     - Stack buffer overflow
     - Use after free
     - Use after return
     - Double free
     - Memory leak (with LSAN)
   - Follow-up: Performance overhead? (~2x slower)
   - Follow-up: ASan vs Valgrind — when to use which?

4. What is UndefinedBehaviorSanitizer (UBSan)?
   ```bash
   gcc -fsanitize=undefined -g program.c -o program
   ```
   - Follow-up: What does UBSan detect?
     - Signed integer overflow
     - Null pointer dereference
     - Misaligned access
     - Out-of-bounds array access
     - Invalid enum value
   - Follow-up: Can you combine ASan + UBSan?
   ```bash
   gcc -fsanitize=address,undefined -g program.c -o program
   ```

5. What is ThreadSanitizer (TSan)?
   ```bash
   gcc -fsanitize=thread -g program.c -o program
   ```
   - Follow-up: What does TSan detect?
   - Follow-up: Why can't ASan and TSan be used together?

6. How do you debug infinite loop?
   - Follow-up: Attach GDB to running process: `gdb -p <pid>`
   - Follow-up: `Ctrl+C` in GDB to pause, then `backtrace`.
   - Follow-up: How to detect infinite loop without debugger in embedded?

7. How do you debug stack corruption?
   - Follow-up: Stack canary (`-fstack-protector-strong`).
   - Follow-up: What is the canary value? How is it checked?
   - Follow-up: Manual fill pattern approach.
   - Follow-up: How does compiler detect smashed stack canary?

8. How do you debug uninitialized variable bugs?
   - Follow-up: `-Wuninitialized` compiler warning.
   - Follow-up: MemorySanitizer (MSan):
   ```bash
   clang -fsanitize=memory -g program.c -o program
   ```
   - Follow-up: Valgrind memcheck — uninitialized value use.

9. How do you debug format string bugs?
   - Follow-up: `-Wformat` compiler warning.
   - Follow-up: `-Wformat-security` for dangerous format strings.

10. How do you use `objdump` for debugging?
    ```bash
    objdump -d program          # disassemble
    objdump -S program          # interleave source and assembly
    objdump -t program          # symbol table
    objdump -x program          # all headers
    ```

11. How do you use `nm` for debugging?
    ```bash
    nm program                  # list symbols
    nm -u program               # undefined symbols (unresolved)
    nm -C program               # demangle C++ names
    ```

12. How do you use `readelf`?
    ```bash
    readelf -h program          # ELF header
    readelf -S program          # sections
    readelf -s program          # symbol table
    readelf -l program          # segments (program headers)
    readelf -d program          # dynamic section
    ```

13. How do you use `strace`?
    ```bash
    strace ./program            # trace system calls
    strace -e open,read ./prog  # filter specific syscalls
    strace -p <pid>             # attach to running process
    ```

14. How do you use `ltrace`?
    ```bash
    ltrace ./program            # trace library calls
    ```

15. What is `addr2line`?
    ```bash
    addr2line -e program 0x08001234   # convert address to file:line
    ```
    - Follow-up: How to use crash address from embedded fault handler?

---

### Embedded Debugging

16. What debugging interfaces are available in embedded systems?
    - JTAG (Joint Test Action Group)
    - SWD (Serial Wire Debug — ARM specific, 2-pin)
    - UART (printf debugging)
    - ITM / SWO (Serial Wire Output)
    - ETM (Embedded Trace Macrocell — instruction trace)
    - Follow-up: JTAG vs SWD — differences and use cases?
    - Follow-up: How many pins does JTAG use vs SWD?

17. What is OpenOCD?
    - Follow-up: OpenOCD + GDB workflow:
    ```bash
    openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
    # in another terminal:
    arm-none-eabi-gdb firmware.elf
    (gdb) target remote :3333
    (gdb) monitor reset halt
    (gdb) load
    (gdb) continue
    ```

18. What is SEGGER J-Link and J-Trace?
    - Follow-up: J-Link vs ST-Link vs CMSIS-DAP?
    - Follow-up: What is J-Trace used for? (instruction trace via ETM)

19. What is ITM (Instrumentation Trace Macrocell)?
    - Follow-up: How does `ITM_SendChar()` work for printf via SWO?
    - Follow-up: Why is ITM better than UART debugging?
      - No UART peripheral needed
      - Non-blocking, timestamped
      - Can be decoded by debugger
    - Follow-up: What is SWO (Serial Wire Output) pin?

20. How do you debug without a debugger in embedded?
    - GPIO toggle + oscilloscope/logic analyzer
    - LED blink pattern (error codes)
    - UART printf
    - Error code stored in non-volatile memory / backup registers
    - Watchdog reset counter in RTC backup register
    - Follow-up: What is the limitation of UART printf debugging in ISR?

21. What is a logic analyzer? When do you use it?
    - Follow-up: Logic analyzer vs oscilloscope — when to use each?
    - Follow-up: Protocol decoding: SPI, I2C, UART, CAN on logic analyzer.

22. What is a boundary scan (JTAG IEEE 1149.1)?
    - Follow-up: How is it used for PCB-level debugging?

23. What is ELF file and why do you need it for debugging?
    - Follow-up: `.elf` vs `.bin` vs `.hex` — what is in each?
    - Follow-up: Why is debug info (`-g`) stripped in production firmware?

24. How do you debug a hard fault on ARM Cortex-M?
    - Follow-up: Hard fault registers:
    ```c
    SCB->HFSR   // Hard Fault Status Register
    SCB->CFSR   // Configurable Fault Status Register
    SCB->MMFAR  // MemManage Fault Address Register
    SCB->BFAR   // Bus Fault Address Register
    ```
    - Follow-up: Decode stacked PC from fault handler:
    ```c
    void HardFault_Handler(void) {
        uint32_t *sp = (uint32_t*)__get_MSP();
        uint32_t pc  = sp[6];  // stacked PC
        uint32_t lr  = sp[5];  // stacked LR
        // log and reset
    }
    ```
    - Follow-up: What is BFAR and MMFAR? When are they valid?

25. What are the types of faults on ARM Cortex-M?
    - Hard Fault (escalated or direct)
    - MemManage Fault (MPU violation, null pointer)
    - Bus Fault (invalid memory access, unaligned)
    - Usage Fault (undefined instruction, divide by zero if enabled)
    - Follow-up: How to enable divide-by-zero and unaligned access trapping?
    ```c
    SCB->CCR |= SCB_CCR_DIV_0_TRP_Msk;
    SCB->CCR |= SCB_CCR_UNALIGN_TRP_Msk;
    ```

26. How do you debug stack overflow in embedded?
    - Follow-up: Fill stack with known pattern (0xDEADBEEF) at startup.
    - Follow-up: Check pattern integrity periodically or in fault handler.
    - Follow-up: MPU stack guard region — generates MemManage fault on overflow.
    - Follow-up: Linker symbol approach to check stack boundaries.

27. How do you debug a watchdog reset?
    - Follow-up: Save reset cause in backup register before reset.
    - Follow-up: Log last known good state to EEPROM/Flash.
    - Follow-up: Add task profiling to identify which task caused hang.

28. What is semi-hosting in ARM embedded?
    - Follow-up: How does it work? (SVC call intercepted by debugger)
    - Follow-up: Why is it not suitable for production?
    - Follow-up: Alternatives: ITM, UART, RTT.

29. What is SEGGER RTT (Real-Time Transfer)?
    - Follow-up: How does RTT work? (shared ring buffer in RAM, debugger reads via JTAG)
    - Follow-up: Why is RTT better than semihosting and UART?
      - No UART peripheral needed
      - Non-blocking (no CPU stall)
      - Works without debug connection for post-mortem
    - Follow-up: RTT Viewer vs SystemView.

---

## ADVANCED LEVEL

### Advanced C Debugging

1. How do you debug optimized code (`-O2`, `-O3`)?
   - Follow-up: Variables may be in registers, not on stack.
   - Follow-up: Inlined functions — no symbol in backtrace.
   - Follow-up: Reordered instructions — source line mapping unreliable.
   - Follow-up: How to force no optimization for specific function:
   ```c
   __attribute__((optimize("O0"))) void critical_debug_func(void) { }
   ```

2. How do you debug intermittent / non-reproducible bugs?
   - Follow-up: Enable all sanitizers in debug build.
   - Follow-up: Add assertions at boundaries.
   - Follow-up: Use circular log buffer — always-on lightweight logging.
   - Follow-up: Stress test: run millions of iterations.
   - Follow-up: What is the Heisenbug? (bug disappears when observed)

3. What is a Heisenbug?
   - Follow-up: Common causes in embedded:
     - Timing changed by debug output
     - Optimizer removes "unnecessary" code with side effects
     - Debugger changes memory layout (padding)
   - Follow-up: How to debug Heisenbug without disturbing it?

4. How do you debug strict aliasing violations?
   ```bash
   gcc -fno-strict-aliasing  # disable for debugging
   gcc -Wstrict-aliasing=2   # warn about violations
   ```
   - Follow-up: Why does UB from aliasing only manifest at `-O2`?

5. How do you debug linker issues?
   ```bash
   ld -Map output.map        # generate map file
   arm-none-eabi-size firmware.elf  # section sizes
   ```
   - Follow-up: How to find what is consuming flash/RAM?
   - Follow-up: How to use map file to find symbol location?
   - Follow-up: Linker garbage collection: `--gc-sections` + `-ffunction-sections -fdata-sections`.

6. How do you debug startup code issues?
   - Follow-up: Issues: `.data` copy from flash to RAM not done.
   - Follow-up: `.bss` not zeroed.
   - Follow-up: Static constructors not called.
   - Follow-up: How to verify startup with memory view in debugger?

7. What is post-mortem debugging?
   - Follow-up: Crash log saved to non-volatile storage.
   - Follow-up: Coredump saved to external Flash/EEPROM.
   - Follow-up: How to reconstruct backtrace from saved stack?

8. How do you use GDB scripting for automated debugging?
   ```python
   # GDB Python script
   import gdb
   gdb.execute("break main")
   gdb.execute("run")
   frame = gdb.selected_frame()
   print(frame.read_var("my_variable"))
   ```

---

### Embedded Advanced Debugging

9. What is instruction trace? What is ETM?
   - Follow-up: ETM records every instruction executed.
   - Follow-up: How does J-Trace capture ETM trace?
   - Follow-up: Use case: find exact instruction that caused fault.
   - Follow-up: Difference between ETM and MTB (Micro Trace Buffer).

10. What is MTB (Micro Trace Buffer)?
    - Follow-up: Available on Cortex-M0+, M23, M33.
    - Follow-up: Small on-chip SRAM used as circular trace buffer.
    - Follow-up: How to read MTB buffer after crash.

11. How do you debug DMA issues?
    - Follow-up: Cache coherency — data written by DMA not visible to CPU.
    - Follow-up: Cache invalidation before DMA read, cache clean before DMA write.
    - Follow-up: Buffer alignment requirements.
    - Follow-up: DMA transfer complete but data corrupt — what to check?

12. How do you debug clock/timing issues?
    - Follow-up: Oscilloscope on GPIO toggle to measure task period.
    - Follow-up: Logic analyzer to verify SPI/I2C timing.
    - Follow-up: How to verify interrupt latency?

13. How do you debug flash write/erase issues?
    - Follow-up: Verify write enable, erase before write, verify after write.
    - Follow-up: What is write protection issue?
    - Follow-up: How to debug CRC mismatch after firmware update?

14. How do you debug power issues?
    - Follow-up: Current measurement with bench power supply ammeter.
    - Follow-up: Power profiler kit (Nordic PPK2, Otii Arc).
    - Follow-up: How to find unexpected current draw in sleep mode?

15. How do you debug communication protocol issues?
    - I2C: Check ACK/NACK, clock stretching, address conflicts.
    - SPI: Check CPOL/CPHA, CS timing, bit order.
    - UART: Check baud rate, parity, stop bits.
    - CAN: Check bit timing, termination, error frames.
    - Follow-up: How to use logic analyzer protocol decoder?

---

### RTOS Debugging

16. What are common RTOS bugs?
    - Stack overflow
    - Priority inversion
    - Deadlock
    - Race condition (ISR vs task shared data)
    - Missing `FromISR` API
    - Calling blocking API from ISR
    - Timer callback too long
    - Memory leak in heap (deleted tasks not cleaned up)
    - Follow-up: Which is hardest to debug?

17. How do you debug stack overflow in FreeRTOS?
    - Follow-up: `configCHECK_FOR_STACK_OVERFLOW 2` — fill pattern method.
    - Follow-up: `vApplicationStackOverflowHook()` callback.
    - Follow-up: `uxTaskGetStackHighWaterMark()` for each task.
    - Follow-up: Systematic approach: double all task stacks, reduce until stable.

18. How do you debug deadlock in RTOS?
    - Follow-up: All tasks in Blocked state — nothing in Ready.
    - Follow-up: `vTaskList()` to see all task states.
    - Follow-up: Identify which resource each task is waiting for.
    - Follow-up: Draw resource allocation graph — find cycle.

19. How do you debug priority inversion in RTOS?
    - Follow-up: High priority task permanently blocked.
    - Follow-up: Use RTOS trace to see task switches.
    - Follow-up: SystemView timeline visualization.

20. How do you debug race condition between ISR and task?
    - Follow-up: Missing `volatile` on shared variable.
    - Follow-up: Non-atomic read-modify-write on shared variable.
    - Follow-up: Missing critical section.
    - Follow-up: How to reproduce: inject artificial delay in critical section.

21. What is SEGGER SystemView and how does it help?
    - Follow-up: Records: task switches, ISR entry/exit, semaphore take/give, queue send/receive.
    - Follow-up: Visualizes timeline of all RTOS events.
    - Follow-up: How does it timestamp? (DWT cycle counter)
    - Follow-up: Overhead of SystemView? (~0.5µs per event)

22. What is Percepio Tracealyzer?
    - Follow-up: Snapshot mode vs streaming mode.
    - Follow-up: What additional analysis does it provide vs SystemView?

23. How do you debug heap fragmentation in FreeRTOS?
    - Follow-up: `xPortGetFreeHeapSize()` vs `xPortGetMinimumEverFreeHeapSize()`.
    - Follow-up: Add malloc failed hook:
    ```c
    void vApplicationMallocFailedHook(void) {
        // log free heap size
        // log task that caused failure
    }
    ```

24. How do you debug incorrect task timing in RTOS?
    - Follow-up: Use `vTaskDelayUntil()` instead of `vTaskDelay()`.
    - Follow-up: GPIO toggle + oscilloscope to measure actual period.
    - Follow-up: DWT cycle counter for microsecond timing measurement.

25. How do you add non-intrusive logging to RTOS?
    - Follow-up: Circular buffer in RAM — log to buffer, dump via UART when idle.
    - Follow-up: SEGGER RTT — zero-copy ring buffer read by debugger.
    - Follow-up: ITM trace channel output.

---

## EXPERT LEVEL

### Expert C Debugging

1. How do you debug miscompilation?
   - Follow-up: Suspect compiler bug when bug only appears at `-O2`/`-O3`.
   - Follow-up: Bisect compiler versions.
   - Follow-up: Inspect generated assembly.
   - Follow-up: Report to compiler team with minimal reproducer.

2. How do you debug ABI mismatch issues?
   - Follow-up: Mixing compiled objects with different ABI settings.
   - Follow-up: Soft-float vs hard-float ABI on ARM.
   - Follow-up: How to check ABI of an object file?
   ```bash
   arm-none-eabi-readelf -A firmware.elf
   ```

3. How do you find and debug subtle undefined behavior?
   - Follow-up: UBSan catches many at runtime.
   - Follow-up: Static analysis tools: Coverity, PC-lint, cppcheck, Clang static analyzer.
   - Follow-up: `__builtin_expect` and how it interacts with UB elimination.

4. What is delta debugging?
   - Follow-up: Systematically reducing test case to minimal reproducer.
   - Follow-up: Binary search over commits to find regression (git bisect).

5. What is `git bisect`?
   ```bash
   git bisect start
   git bisect bad HEAD
   git bisect good v1.0
   git bisect run ./test.sh  # automated
   ```
   - Follow-up: How to bisect a flaky/intermittent bug?

6. How do you debug position independent code issues?
   - Follow-up: GOT/PLT corruption.
   - Follow-up: Relocations in embedded firmware.

7. How do you debug flash memory corruption?
   - Follow-up: CRC check over flash regions periodically.
   - Follow-up: Write protection bits.
   - Follow-up: Flash endurance — unexpected erase cycle wear.

---

### Expert Embedded Debugging

8. How do you recover from a completely unresponsive embedded target?
   - Follow-up: JTAG connect and halt: `monitor reset halt`.
   - Follow-up: Read PC and registers to find where it's stuck.
   - Follow-up: If JTAG also fails — check power, clock, reset line with oscilloscope.

9. How do you debug a boot failure (blank screen, no UART output)?
   - Follow-up: Systematic checklist:
     1. Power supply voltage correct?
     2. Clock source stable? (crystal, PLL lock)
     3. Correct boot mode pins?
     4. Startup code running? (toggle LED in very first line)
     5. `.data` initialized, `.bss` zeroed?
     6. Stack pointer correct?
     7. Interrupt vector table at correct address?

10. How do you debug firmware update (OTA/DFU) failures?
    - Follow-up: CRC mismatch — verify transfer integrity.
    - Follow-up: Flash write error — verify erase before write.
    - Follow-up: Wrong jump address after update.
    - Follow-up: Bootloader stack and application stack conflict.

11. How do you debug EMI/EMC related embedded issues?
    - Follow-up: Intermittent resets caused by external interference.
    - Follow-up: Power supply decoupling.
    - Follow-up: GPIO input filtering.
    - Follow-up: How to distinguish EMI bug from software bug?

12. How do you do remote debugging in production embedded systems?
    - Follow-up: UART command shell for runtime inspection.
    - Follow-up: Diagnostic mode triggered by GPIO or special command.
    - Follow-up: Crash log uplink over communication interface.
    - Follow-up: What is a "black box" approach in embedded?

---

### Expert RTOS Debugging

13. How do you debug a task that intermittently misses its deadline?
    - Follow-up: Trace with SystemView to see what preempts it.
    - Follow-up: Measure worst-case execution time.
    - Follow-up: Check for shared resource contention (mutex wait time).

14. How do you debug ISR that causes system instability?
    - Follow-up: ISR too long — measure with GPIO toggle + oscilloscope.
    - Follow-up: ISR calling non-ISR-safe RTOS API.
    - Follow-up: ISR stack overflow.
    - Follow-up: Interrupt storm — peripheral generating too many interrupts.

15. How do you debug memory leak in long-running RTOS system?
    - Follow-up: Log `xPortGetFreeHeapSize()` periodically.
    - Follow-up: Log heap size before and after each allocation.
    - Follow-up: Custom malloc/free wrappers with caller tracking.

16. How do you debug multicore RTOS issues?
    - Follow-up: Cache coherency — CPU0 writes, CPU1 reads stale cache line.
    - Follow-up: Missing memory barrier between cores.
    - Follow-up: Spinlock fairness — one core never acquires lock.

---

## DEBUGGING STRATEGY FRAMEWORK

### Systematic Debugging Process

```
1. REPRODUCE
   └─ Find minimal, consistent reproduction steps
   └─ If intermittent: add logging, stress test

2. ISOLATE
   └─ Binary search: which subsystem?
   └─ Simplify: remove code until bug disappears
   └─ git bisect: which commit introduced it?

3. IDENTIFY ROOT CAUSE
   └─ Use tools: GDB, sanitizers, static analysis
   └─ Read all relevant code, not just crash site
   └─ Question all assumptions

4. FIX
   └─ Fix root cause, not symptom
   └─ Understand why fix works

5. VERIFY
   └─ Reproduce original scenario → fixed
   └─ Run full regression
   └─ Check for similar bugs elsewhere

6. PREVENT RECURRENCE
   └─ Add assertion / test for this case
   └─ Update documentation / coding guidelines
```

---

## TOOLS QUICK REFERENCE TABLE

| Tool | Purpose | Platform |
|------|---------|----------|
| GDB | Debugger | Linux/Embedded |
| OpenOCD | JTAG/SWD server | Embedded |
| J-Link GDB Server | JTAG/SWD server | Embedded |
| Valgrind | Memory errors, leak | Linux |
| ASan | Memory errors | Linux/Embedded |
| UBSan | Undefined behavior | Linux |
| TSan | Race conditions | Linux |
| MSan | Uninitialized reads | Linux |
| SEGGER SystemView | RTOS trace | Embedded RTOS |
| Percepio Tracealyzer | RTOS trace | Embedded RTOS |
| SEGGER RTT | Non-intrusive logging | Embedded |
| ITM/SWO | Printf trace | ARM Cortex-M |
| J-Trace | Instruction trace (ETM) | ARM |
| Coverity | Static analysis | C/C++ |
| PC-lint | Static analysis | C/C++ |
| Cppcheck | Static analysis | C/C++ |
| Clang-tidy | Linting | C/C++ |
| objdump | Binary analysis | All |
| nm | Symbol inspection | All |
| readelf | ELF inspection | Linux/Embedded |
| addr2line | Address → source | All |
| strace | Syscall trace | Linux |
| ltrace | Library trace | Linux |
| Logic analyzer | Signal debug | Embedded HW |
| Oscilloscope | Timing/signal | Embedded HW |

---

## TRICK / DEEP COUNTER QUESTIONS

1. "You said ASan catches buffer overflow — can it catch overflow of a global array?"
   - **Answer:** Yes — ASan instruments global, stack, and heap buffers.

2. "You said printf debugging is bad in ISR — why exactly?"
   - **Answer:** printf uses locks (not ISR-safe), may block, changes timing.

3. "You said hardware breakpoints don't affect timing — is this always true?"
   - **Answer:** Not always — some peripherals sensitive to halted CPU (watchdog, communication timeouts).

4. "You said `volatile` fixes race condition — is that true?"
   - **Answer:** No — `volatile` only prevents compiler optimization; does not provide atomicity or memory ordering.

5. "You said watchdog detects hangs — what if the watchdog kick itself is inside the hang?"
   - **Answer:** Task-based watchdog monitoring — each task has its own token that it must refresh.

6. "You said stack overflow detection method 2 (fill pattern) is reliable — what can fool it?"
   - **Answer:** A single large stack frame that jumps over the sentinel region.

7. "How do you debug a bug that only happens on customer hardware, not your dev board?"
   - **Answer:** Hardware differences (clock speed, memory, PCB noise), enable logging, ship debug build.

8. "You said `addr2line` converts crash address to source line — what if binary is stripped?"
   - **Answer:** Keep unstripped ELF with debug symbols separate; strip only production binary.

9. "You said RTT is non-blocking — what happens when RTT buffer is full?"
   - **Answer:** Configurable: block, skip, or overwrite oldest data.

10. "You said SystemView overhead is ~0.5µs — is this acceptable for a 10µs ISR?"
    - **Answer:** No — 5% overhead, may mask timing bugs. Disable trace for tight ISRs.

---

## SUMMARY TABLE

| Debugging Area | Basic | Intermediate | Advanced | Expert |
|----------------|-------|-------------|----------|--------|
| GDB usage | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Sanitizers (ASan/UBSan) | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★☆ |
| Core dump analysis | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★★★ |
| Embedded JTAG/SWD | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★★ |
| Hard fault analysis | ★★☆☆☆ | ★★★★☆ | ★★★★★ | ★★★★★ |
| RTOS trace (SystemView) | ★★☆☆☆ | ★★★★☆ | ★★★★★ | ★★★★★ |
| Instruction trace (ETM) | ★☆☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ |
| Static analysis | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★★★ |
| Post-mortem debug | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| Optimized code debug | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| Multicore debug | ★☆☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★★ |
| Intermittent bug debug | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |

---

*This document covers debugging from fundamentals to expert level across C, Embedded Systems, and RTOS.*