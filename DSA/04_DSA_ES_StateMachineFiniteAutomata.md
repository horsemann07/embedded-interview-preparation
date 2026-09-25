# 4. STATE MACHINES & FINITE AUTOMATA
## State Machine Interview Pattern

For almost every embedded state-machine question, use this structure in
an interview:

``` text
1. Define the states
        ↓
2. Define the events
        ↓
3. Define transition conditions
        ↓
4. Define entry/exit actions
        ↓
5. Handle invalid events
        ↓
6. Handle timeout/fault
        ↓
7. Define recovery behavior
```

A useful generic model is:

``` text
             EVENT
               |
               v
        +---------------+
        | CURRENT STATE |
        +---------------+
               |
          transition
               |
               v
         +-----------+
         | NEW STATE |
         +-----------+
               |
          entry action
               |
               v
          STATE RUNNING
```


## BEGINNER

#### 146. Simple LED blink state machine (on/off).

**Answer**

I would define two states: `LED_OFF` and `LED_ON`. A periodic timer tick
drives the state machine. When the required time expires, the state
changes from OFF to ON or ON to OFF. The state machine decides the
transition, while the timer provides the timing event.

**Counter: How to implement state transitions?**

Use an enum for the current state and a `switch` statement to define the
behavior and transition of each state.

``` c
#include <stdint.h>

typedef enum
{
    LED_OFF,
    LED_ON
} LedState;

static LedState state = LED_OFF;

void led_task(void)
{
    switch (state)
    {
        case LED_OFF:
            if (timer_expired())
            {
                led_on();
                state = LED_ON;
            }
            break;

        case LED_ON:
            if (timer_expired())
            {
                led_off();
                state = LED_OFF;
            }
            break;

        default:
            state = LED_OFF;
            break;
    }
}
```

**Counter: Timer tick drives transitions?**

Yes. A periodic timer can generate a `TIMEOUT` event. The state machine
checks the event and changes state when the required time has elapsed.

**Counter: Can you reduce states?**

Yes. If the only requirement is to toggle an LED periodically, a
complete state machine may be unnecessary. A simple timer and toggle
operation can be enough.

**Interview point:** Do not use a state machine just because you can.
Use one when different states have different behavior.

------------------------------------------------------------------------

#### 147. Button debounce as state machine.

**Answer**

A mechanical button does not transition cleanly between released and
pressed. It can rapidly switch between 0 and 1 for several milliseconds.
I would use states such as `RELEASED`, `DEBOUNCE_PRESS`, `PRESSED`, and
`DEBOUNCE_RELEASE`. A timer determines whether the button has remained
stable long enough to accept the transition.

``` c
#include <stdint.h>

typedef enum
{
    BUTTON_RELEASED,
    BUTTON_DEBOUNCE_PRESS,
    BUTTON_PRESSED,
    BUTTON_DEBOUNCE_RELEASE
} ButtonState;

static ButtonState state = BUTTON_RELEASED;
static uint32_t debounce_ticks = 0U;

void button_task(bool button_pressed)
{
    switch (state)
    {
        case BUTTON_RELEASED:
            if (button_pressed)
            {
                debounce_ticks = 0U;
                state = BUTTON_DEBOUNCE_PRESS;
            }
            break;

        case BUTTON_DEBOUNCE_PRESS:
            if (!button_pressed)
            {
                state = BUTTON_RELEASED;
            }
            else if (++debounce_ticks >= 20U)
            {
                state = BUTTON_PRESSED;
            }
            break;

        case BUTTON_PRESSED:
            if (!button_pressed)
            {
                debounce_ticks = 0U;
                state = BUTTON_DEBOUNCE_RELEASE;
            }
            break;

        case BUTTON_DEBOUNCE_RELEASE:
            if (button_pressed)
            {
                state = BUTTON_PRESSED;
            }
            else if (++debounce_ticks >= 20U)
            {
                state = BUTTON_RELEASED;
            }
            break;

        default:
            state = BUTTON_RELEASED;
            break;
    }
}
```

**Counter: States: released, pressed, held, released_check?**

Those states can be used, but `HELD` is usually an application-level
state/event rather than a mandatory debounce state. A common design is:

``` text
RELEASED
    |
    | button detected
    v
DEBOUNCE_PRESS
    |
    | stable
    v
PRESSED
    |
    | long duration
    v
HELD
    |
    | release detected
    v
DEBOUNCE_RELEASE
    |
    | stable
    v
RELEASED
```

**Counter: How many timer ticks to debounce?**

A common value is around 20 ms. For a 1 ms periodic task, that means 20
ticks.

However, 20 ms is not universal. The correct value depends on the
physical switch, sampling frequency, and system requirements.

**Counter: What if the button glitches during hold?**

If the button briefly glitches during `PRESSED`, do not immediately
treat it as a real release. Enter `DEBOUNCE_RELEASE`. If the signal
becomes pressed again before the debounce interval expires, return to
`PRESSED`.

**Interview point:** Debouncing means accepting a transition only after
the input remains stable for the required time.

------------------------------------------------------------------------

#### 148. Protocol parser state machine (detect packet structure).

**Answer**

A protocol parser is a classic state-machine application. I would define
states such as `WAIT_SYNC`, `READ_HEADER`, `READ_PAYLOAD`, and
`CHECKSUM`. Each received byte acts as an event that may cause a state
transition.

``` c
#include <stdint.h>

typedef enum
{
    WAIT_SYNC,
    READ_HEADER,
    READ_PAYLOAD,
    CHECKSUM
} ParserState;

static ParserState state = WAIT_SYNC;

void parser_process_byte(uint8_t byte)
{
    switch (state)
    {
        case WAIT_SYNC:
            if (byte == 0xAAU)
            {
                state = READ_HEADER;
            }
            break;

        case READ_HEADER:
            /*
             * Read header fields here.
             * Transition when the complete header is received.
             */
            if (header_complete())
            {
                state = READ_PAYLOAD;
            }
            break;

        case READ_PAYLOAD:
            /*
             * Store payload bytes.
             */
            if (payload_complete())
            {
                state = CHECKSUM;
            }
            break;

        case CHECKSUM:
            if (checksum_valid(byte))
            {
                packet_ready();
            }

            /*
             * Whether valid or invalid, return to a known
             * synchronization state.
             */
            state = WAIT_SYNC;
            break;

        default:
            state = WAIT_SYNC;
            break;
    }
}
```

**Counter: States for sync, header, payload, checksum?**

Yes. This is a natural state decomposition:

``` text
WAIT_SYNC
    ↓
READ_HEADER
    ↓
READ_PAYLOAD
    ↓
CHECKSUM
    ↓
PACKET_READY
```

**Counter: Invalid state recovery?**

If invalid data is received, discard the current packet and return to
`WAIT_SYNC`. Depending on the protocol, the invalid byte can also be
checked to see whether it is a possible new synchronization byte.

**Counter: Timeout handling?**

Each parser state can have a timeout. If the expected next byte does not
arrive within the allowed time, reset the parser.

``` text
READ_PAYLOAD
      |
      | timeout
      v
WAIT_SYNC
```

**Interview point:** A robust protocol parser must handle valid packets,
invalid bytes, timeouts, buffer overflow, and resynchronization.

------------------------------------------------------------------------

#### 149. Traffic light controller (Red → Green → Yellow → Red).

**Answer**

I would represent each traffic-light phase as a state: `RED`, `GREEN`,
and `YELLOW`. A timer controls how long each state remains active.

``` c
#include <stdint.h>

typedef enum
{
    TRAFFIC_RED,
    TRAFFIC_GREEN,
    TRAFFIC_YELLOW
} TrafficState;

static TrafficState state = TRAFFIC_RED;

void traffic_task(void)
{
    switch (state)
    {
        case TRAFFIC_RED:
            set_red_light();

            if (timer_expired())
            {
                state = TRAFFIC_GREEN;
            }
            break;

        case TRAFFIC_GREEN:
            set_green_light();

            if (timer_expired())
            {
                state = TRAFFIC_YELLOW;
            }
            break;

        case TRAFFIC_YELLOW:
            set_yellow_light();

            if (timer_expired())
            {
                state = TRAFFIC_RED;
            }
            break;

        default:
            state = TRAFFIC_RED;
            break;
    }
}
```

**Counter: Time in each state?**

Each state can have a different timeout.

Example:

``` text
RED    = 30 seconds
GREEN  = 25 seconds
YELLOW = 5 seconds
```

The actual values should come from the system requirements.

**Counter: Emergency vehicle override?**

Treat the emergency condition as an event. The system can transition to
a dedicated `EMERGENCY` state or another safety-defined sequence.

``` text
GREEN
  |
  | emergency event
  v
EMERGENCY
```

After the emergency condition is handled, the controller can return to
the normal sequence according to the system design.

**Counter: State diagram clarity?**

Every state should clearly define:

``` text
Entry action
State behavior
Transition condition
Next state
Fault behavior
```

**Interview point:** A state diagram should make every valid transition
and important fault/override path easy to understand.

------------------------------------------------------------------------

#### 150. Power mode state machine (active/sleep/deep sleep).

**Answer**

A power-management state machine can represent modes such as `ACTIVE`,
`SLEEP`, and `DEEP_SLEEP`. Events such as inactivity, timeout, GPIO
interrupts, communication activity, or RTC events trigger transitions.

``` c
#include <stdint.h>

typedef enum
{
    POWER_ACTIVE,
    POWER_SLEEP,
    POWER_DEEP_SLEEP
} PowerState;

static PowerState state = POWER_ACTIVE;

void power_task(void)
{
    switch (state)
    {
        case POWER_ACTIVE:
            if (inactivity_timeout())
            {
                prepare_sleep();
                state = POWER_SLEEP;
            }
            break;

        case POWER_SLEEP:
            if (wake_event())
            {
                restore_from_sleep();
                state = POWER_ACTIVE;
            }
            else if (deep_sleep_timeout())
            {
                prepare_deep_sleep();
                state = POWER_DEEP_SLEEP;
            }
            break;

        case POWER_DEEP_SLEEP:
            if (wake_event())
            {
                restore_from_deep_sleep();
                state = POWER_ACTIVE;
            }
            break;

        default:
            state = POWER_ACTIVE;
            break;
    }
}
```

**Counter: Wake-up events trigger transitions?**

Yes. Typical wake-up sources include:

``` text
GPIO interrupt
Timer/RTC
CAN/UART activity
External interrupt
Sensor interrupt
```

**Counter: State entry/exit code for setup?**

Yes. Entry/exit actions are important.

For example:

``` c
void enter_sleep(void)
{
    disable_unused_peripherals();
    configure_wakeup_sources();
    reduce_clock_frequency();
}

void exit_sleep(void)
{
    restore_clock();
    restore_peripherals();
}
```

**Counter: Energy consumption in each state?**

Generally:

``` text
ACTIVE       → highest power
SLEEP        → lower power
DEEP_SLEEP   → lowest power
```

Actual consumption depends on the MCU, clocks, RAM retention,
peripherals, and wake-up sources.

**Interview point:** Power management is a tradeoff between power
consumption and wake-up latency.

------------------------------------------------------------------------

#### 151. Stepper motor control state machine.

**Answer**

A stepper motor can be controlled using states representing the coil
excitation sequence. The state changes according to the requested
direction and step timing.

``` c
#include <stdint.h>

typedef enum
{
    STEP_0,
    STEP_1,
    STEP_2,
    STEP_3
} StepState;

static StepState state = STEP_0;

void stepper_step(bool forward)
{
    if (forward)
    {
        switch (state)
        {
            case STEP_0:
                energize_coils(0);
                state = STEP_1;
                break;

            case STEP_1:
                energize_coils(1);
                state = STEP_2;
                break;

            case STEP_2:
                energize_coils(2);
                state = STEP_3;
                break;

            case STEP_3:
                energize_coils(3);
                state = STEP_0;
                break;

            default:
                state = STEP_0;
                break;
        }
    }
    else
    {
        switch (state)
        {
            case STEP_0:
                energize_coils(0);
                state = STEP_3;
                break;

            case STEP_1:
                energize_coils(1);
                state = STEP_0;
                break;

            case STEP_2:
                energize_coils(2);
                state = STEP_1;
                break;

            case STEP_3:
                energize_coils(3);
                state = STEP_2;
                break;

            default:
                state = STEP_0;
                break;
        }
    }
}
```

**Counter: States for each coil activation sequence?**

Yes. Each state can correspond to a coil excitation pattern.

Example:

``` text
STEP_0 → Coil A
STEP_1 → Coil B
STEP_2 → Coil A'
STEP_3 → Coil B'
```

The exact sequence depends on the motor and whether full-step,
half-step, or microstepping is used.

**Counter: Forward and reverse stepping?**

Forward moves through the sequence in one direction:

``` text
0 → 1 → 2 → 3 → 0
```

Reverse moves through it in the opposite direction:

``` text
0 → 3 → 2 → 1 → 0
```

**Counter: Acceleration/deceleration ramp?**

A stepper should not necessarily jump directly to maximum step
frequency.

A typical profile is:

``` text
Start slowly
    ↓
Increase step frequency
    ↓
Run at target speed
    ↓
Decrease step frequency
    ↓
Stop
```

This prevents missed steps and improves motion control.

**Interview point:** The state machine controls the excitation sequence;
a timer/motion-control layer determines when the next step occurs.

------------------------------------------------------------------------

#### 152. Temperature controller state machine.

**Answer**

I would typically use states such as `IDLE`, `HEATING`, `COOLING`, and
`FAULT`. Temperature thresholds control the transitions.

``` c
#include <stdint.h>

typedef enum
{
    TEMP_IDLE,
    TEMP_HEATING,
    TEMP_COOLING,
    TEMP_FAULT
} TemperatureState;

static TemperatureState state = TEMP_IDLE;

void temperature_task(int32_t temperature)
{
    switch (state)
    {
        case TEMP_IDLE:
            heater_off();
            cooler_off();

            if (sensor_fault())
            {
                state = TEMP_FAULT;
            }
            else if (temperature < 20)
            {
                state = TEMP_HEATING;
            }
            else if (temperature > 30)
            {
                state = TEMP_COOLING;
            }
            break;

        case TEMP_HEATING:
            heater_on();
            cooler_off();

            if (sensor_fault())
            {
                state = TEMP_FAULT;
            }
            else if (temperature > 22)
            {
                state = TEMP_IDLE;
            }
            break;

        case TEMP_COOLING:
            heater_off();
            cooler_on();

            if (sensor_fault())
            {
                state = TEMP_FAULT;
            }
            else if (temperature < 28)
            {
                state = TEMP_IDLE;
            }
            break;

        case TEMP_FAULT:
            heater_off();
            cooler_off();
            report_temperature_fault();
            break;

        default:
            state = TEMP_FAULT;
            break;
    }
}
```

**Counter: States: heating, cooling, idle?**

Yes. A fault state should also be considered because sensor or control
failures need safe handling.

**Counter: Hysteresis to prevent oscillation?**

Yes. Hysteresis prevents rapid ON/OFF switching around a threshold.

For example:

``` text
Heater ON  < 20°C
Heater OFF > 22°C
```

Without hysteresis:

``` text
ON  at 20°C
OFF at 20°C
```

Small sensor fluctuations could cause repeated switching.

**Counter: Fault detection state?**

Yes. Examples include:

``` text
Sensor disconnected
Sensor out of range
ADC failure
Over-temperature
Invalid sensor value
```

A fault should normally force the actuator into a safe condition.

**Interview point:** Temperature control is not only about thresholds;
it also needs hysteresis and fault handling.

------------------------------------------------------------------------

#### 153. Encryption key derivation state machine.

**Answer**

A key-derivation process can be represented as sequential states when
operations are asynchronous, hardware-assisted, or need explicit
verification.

``` c
typedef enum
{
    KEY_START,
    KEY_LOAD_INPUT,
    KEY_DERIVE,
    KEY_VERIFY,
    KEY_READY,
    KEY_ERROR
} KeyState;

static KeyState state = KEY_START;

void key_task(void)
{
    switch (state)
    {
        case KEY_START:
            initialize_key_operation();
            state = KEY_LOAD_INPUT;
            break;

        case KEY_LOAD_INPUT:
            if (load_key_input())
            {
                state = KEY_DERIVE;
            }
            else
            {
                state = KEY_ERROR;
            }
            break;

        case KEY_DERIVE:
            start_key_derivation();

            if (key_derivation_complete())
            {
                state = KEY_VERIFY;
            }
            break;

        case KEY_VERIFY:
            if (verify_derived_key())
            {
                state = KEY_READY;
            }
            else
            {
                state = KEY_ERROR;
            }
            break;

        case KEY_READY:
            use_key();
            break;

        case KEY_ERROR:
            handle_key_error();
            break;

        default:
            state = KEY_ERROR;
            break;
    }
}
```

**Counter: States for key generation steps?**

Yes, when the process consists of multiple sequential or asynchronous
steps.

**Counter: Ensure all states execute in order?**

If the cryptographic process requires a specific sequence, transitions
should only occur when the previous operation has successfully
completed.

**Counter: Prevent skip to final state?**

Only transition to `KEY_READY` after all required operations and
verification succeed.

``` text
START
  ↓
LOAD
  ↓
DERIVE
  ↓
VERIFY
  ↓
KEY_READY
```

**Interview point:** Do not create custom cryptographic algorithms. Use
established cryptographic primitives and vetted implementations.

------------------------------------------------------------------------

#### 154. Communication protocol handshake (ACK/NACK/RETRY).

**Answer**

A communication handshake can be modeled with states such as `IDLE`,
`TRANSMIT`, `WAIT_ACK`, `RETRY`, and `ERROR`. A timeout prevents the
system from waiting forever for a response.

``` c
#include <stdint.h>

typedef enum
{
    COMM_IDLE,
    COMM_TRANSMIT,
    COMM_WAIT_ACK,
    COMM_ERROR
} CommState;

static CommState state = COMM_IDLE;
static uint8_t retry_count = 0U;

#define MAX_RETRIES 3U

void communication_task(void)
{
    switch (state)
    {
        case COMM_IDLE:
            if (message_ready())
            {
                retry_count = 0U;
                state = COMM_TRANSMIT;
            }
            break;

        case COMM_TRANSMIT:
            transmit_message();
            start_ack_timeout();
            state = COMM_WAIT_ACK;
            break;

        case COMM_WAIT_ACK:
            if (ack_received())
            {
                state = COMM_IDLE;
            }
            else if (nack_received() || ack_timeout())
            {
                if (retry_count < MAX_RETRIES)
                {
                    retry_count++;
                    state = COMM_TRANSMIT;
                }
                else
                {
                    state = COMM_ERROR;
                }
            }
            break;

        case COMM_ERROR:
            report_communication_error();
            state = COMM_IDLE;
            break;

        default:
            state = COMM_ERROR;
            break;
    }
}
```

**Counter: States for waiting, transmission, acknowledgment?**

Yes. A typical sequence is:

``` text
IDLE
 ↓
TRANSMIT
 ↓
WAIT_ACK
 ↓
ACK → IDLE
```

**Counter: Retry counter and timeout?**

Yes. If ACK is not received before the timeout, increment the retry
counter and retransmit until `MAX_RETRIES` is reached.

**Counter: Recovery from unexpected message?**

The protocol should define the behavior. Possible actions include:

``` text
Ignore
Send NACK
Reset handshake
Return to IDLE
Report protocol error
```

**Interview point:** A robust communication state machine must handle
ACK, NACK, timeout, unexpected messages, retry exhaustion, and
communication failure.

------------------------------------------------------------------------

#### 155. File system state machine (unmounted/mounted/busy).

**Answer**

A file-system state machine can represent whether the filesystem is
unavailable, mounted, busy, or in an error state.

``` c
#include <stdint.h>

typedef enum
{
    FS_UNMOUNTED,
    FS_MOUNTED,
    FS_BUSY,
    FS_ERROR
} FileSystemState;

static FileSystemState state = FS_UNMOUNTED;

void filesystem_task(void)
{
    switch (state)
    {
        case FS_UNMOUNTED:
            if (mount_filesystem())
            {
                state = FS_MOUNTED;
            }
            else
            {
                state = FS_ERROR;
            }
            break;

        case FS_MOUNTED:
            if (file_operation_requested())
            {
                state = FS_BUSY;
            }
            else if (unmount_requested())
            {
                if (!unmount_filesystem())
                {
                    state = FS_ERROR;
                }
                else
                {
                    state = FS_UNMOUNTED;
                }
            }
            break;

        case FS_BUSY:
            if (operation_complete())
            {
                state = FS_MOUNTED;
            }
            else if (operation_timeout())
            {
                state = FS_ERROR;
            }
            break;

        case FS_ERROR:
            handle_filesystem_error();
            break;

        default:
            state = FS_ERROR;
            break;
    }
}
```

**Counter: State transitions on mount/unmount/error?**

Typical transitions are:

``` text
UNMOUNTED
    |
    | mount success
    v
MOUNTED
    |
    | file operation
    v
BUSY
    |
    | complete
    v
MOUNTED
```

An error from any important operation can transition to `ERROR`.

**Counter: What happens in invalid state transitions?**

An operation that is not valid for the current state should be rejected.

For example:

``` text
UNMOUNTED → read_file()
```

should return an error such as:

``` c
FS_NOT_MOUNTED
```

rather than attempting the operation.

**Counter: Stuck state detection?**

Use an operation timeout or watchdog.

For example:

``` text
BUSY
  |
  | operation timeout
  v
ERROR
```

This prevents the system from remaining in `BUSY` forever.

**Interview point:** A robust filesystem state machine should define
valid transitions, reject invalid operations, and have timeout/error
recovery.

------------------------------------------------------------------------

## INTERMEDIATE

## Intermediate State Machine Interview Pattern

For intermediate-level FSM questions, answer using this structure:

```text
1. Define the states
        ↓
2. Define events
        ↓
3. Define transition conditions
        ↓
4. Define guards
        ↓
5. Define entry/exit actions
        ↓
6. Define timeout behavior
        ↓
7. Define invalid-event handling
        ↓
8. Define fault/recovery behavior
        ↓
9. Consider concurrency/resource ownership
        ↓
10. Consider timing and testability
```

### Key concepts to remember

```text
Mealy
    → Output depends on State + Input

Moore
    → Output depends on State

Hierarchical FSM
    → Parent states handle common behavior

Guard
    → Transition allowed only when condition is true

Timeout
    → Transition after defined time

Parallel FSM
    → Multiple independent FSMs run together

Event-driven
    → Event causes execution/transition

Time-driven
    → Periodic tick causes execution

State history
    → Previous state retained for diagnostics/recovery

Entry guard
    → Preconditions checked before entering a state
```

### Strong interview closing line

> **“When designing an embedded state machine, I don't just define the normal state transitions. I also define guards, timeouts, invalid-event behavior, resource ownership, fault recovery, and worst-case timing. That makes the FSM deterministic, testable, and robust.”**


#### 161. Mealy vs Moore state machine: which to use?

**Answer**

A **Mealy machine** produces outputs based on both the current state and the current input/event.

A **Moore machine** produces outputs based only on the current state.

The main difference is:

```text
Mealy:
    Output = f(State, Input)

Moore:
    Output = f(State)
```

Example:

```text
Mealy:

STATE_WAIT + RX_BYTE
        ↓
    output/event

Moore:

STATE_RX
   ↓
output depends only on STATE_RX
```

For embedded systems, the choice depends on the behavior you need.

**Counter: Output on transition vs state?**

In a Mealy machine, an output can change immediately as a result of an input/event while remaining in or transitioning through a state.

In a Moore machine, the output is associated with the current state, so changing the output generally means entering a different state.

Example:

```text
Mealy:
WAIT + VALID_RX → SEND_ACK

Moore:
WAIT → ACK_STATE
       |
       +--> ACK output is active in ACK_STATE
```

**Counter: Performance difference?**

There is usually no meaningful inherent performance advantage between Mealy and Moore.

The actual cost depends on:

- Number of states
- Number of transitions
- Amount of work done in each state
- Implementation
- Compiler optimization
- Target CPU

A Mealy machine can sometimes react to an event without adding a separate output state, which may reduce state count. A Moore machine may require extra states for explicit outputs.

**Counter: Which is easier for protocol parsing?**

Both can work, but **Mealy-style logic is often convenient for protocol parsing** because received bytes/events directly affect transitions and actions.

Example:

```text
WAIT_SYNC + SYNC_BYTE
        ↓
READ_HEADER

WAIT_ACK + ACK
        ↓
TRANSFER_COMPLETE
```

However, for safety-critical or highly deterministic designs, Moore-style outputs can be easier to reason about because outputs are tied directly to well-defined states.

**Interview point:**

> Mealy outputs depend on state + input, while Moore outputs depend only on state. I would choose based on clarity, timing requirements, number of states, and how the outputs need to behave.

---

#### 162. Hierarchical state machine (nested states).

**Answer**

A hierarchical state machine, or **HSM**, allows states to contain substates.

Instead of creating many independent states for every combination of behavior, common behavior can be placed in a **parent/super-state**.

Example:

```text
SYSTEM
├── NORMAL
│   ├── IDLE
│   └── RUNNING
└── ERROR
    ├── RECOVERABLE
    └── FATAL
```

Suppose both `IDLE` and `RUNNING` should respond to a global `SHUTDOWN` event.

Instead of implementing:

```text
IDLE + SHUTDOWN
RUNNING + SHUTDOWN
```

separately, the parent state `NORMAL` can handle the common transition.

**Counter: Parent state for common transitions?**

Yes.

The child state is checked first. If the child does not handle the event, the parent can handle it.

Conceptually:

```text
Event
  ↓
Current child state
  |
  | not handled
  v
Parent state
  |
  | not handled
  v
Higher parent/default handler
```

This avoids duplicating common transition logic.

**Counter: Entry/exit actions for super-states?**

Yes.

A super-state can perform common entry/exit behavior.

For example:

```text
Enter NORMAL
    ↓
enable normal-mode resources

Enter IDLE
    ↓
configure idle behavior
```

When leaving the hierarchy:

```text
Leave IDLE
    ↓
Leave NORMAL
    ↓
disable normal-mode resources
```

Exact ordering should be explicitly defined by the HSM implementation.

**Counter: Implementation without recursion?**

Yes.

You do not need recursive function calls.

A simple implementation can use:

- Explicit parent pointers
- State tables
- Iterative event dispatch
- Parent-state lookup

Example:

```c
typedef struct State
{
    const struct State *parent;

    void (*entry)(void);
    void (*exit)(void);
    bool (*handle_event)(int event);

} State;
```

Event dispatch can walk upward iteratively:

```c
const State *state = current_state;

while (state != NULL)
{
    if (state->handle_event(event))
    {
        break;
    }

    state = state->parent;
}
```

**Interview point:**

> Hierarchical state machines reduce state explosion by allowing common transitions and behavior to be handled in parent states.

---

#### 163. Guard conditions: execute transition only if condition true.

**Answer**

A guard condition is a boolean condition that must be true before a transition is allowed.

Conceptually:

```text
CURRENT_STATE
      |
      | EVENT + guard == true
      v
 NEXT_STATE
```

Example:

```text
WAIT_FOR_START
      |
      | START + system_ready == true
      v
RUNNING
```

If `system_ready == false`, the transition does not occur.

**Counter: Prevent invalid transitions?**

Yes.

Guards are useful for enforcing preconditions.

Example:

```c
if (state == RUNNING &&
    motor_ready &&
    temperature_ok)
{
    state = ACTIVE;
}
```

This prevents the system from entering a state when required conditions are not satisfied.

**Counter: What if guard fails in critical section?**

Do not assume that simply checking a variable makes the operation safe.

If the guard depends on data that can change asynchronously, the required concurrency protection must be considered.

For example:

```text
Read condition
    ↓
Condition changes
    ↓
Perform transition
```

can cause a race condition.

Depending on the system, you may need:

- Atomic access
- Interrupt protection
- Mutex/lock
- Snapshot of relevant inputs
- Execution within a critical section

But critical sections should be kept short.

**Counter: Can you have multiple guards per transition?**

Yes.

For example:

```text
EVENT
AND
system_ready
AND
temperature_ok
AND
voltage_ok
        ↓
NEXT_STATE
```

Code example:

```c
if (start_event &&
    system_ready &&
    temperature_ok &&
    voltage_ok)
{
    state = RUNNING;
}
```

**Interview point:**

> A guard is a precondition for a transition. Guards help prevent illegal state changes, but shared data must still be handled safely.

---

#### 164. Timeout-based state transitions.

**Answer**

A timeout transition automatically moves the FSM to another state if the current state has been active for too long or for a defined amount of time.

Example:

```text
WAIT_ACK
   |
   | timeout = 100 ms
   v
RETRY
```

**Counter: How long in each state before auto-transition?**

Each state can define its own timeout.

Example:

```text
WAIT_ACK   → 100 ms
INITIALIZE → 2 s
CALIBRATE  → 500 ms
```

The timeout values should come from system requirements and measured worst-case behavior, not arbitrary guesses.

**Counter: Timer management for multiple states?**

A common approach is to record a timestamp when entering the state.

Example:

```c
static uint32_t state_enter_time;

void enter_state(uint32_t now)
{
    state_enter_time = now;
}

bool state_timeout_expired(uint32_t now,
                           uint32_t timeout)
{
    return (uint32_t)(now - state_enter_time) >= timeout;
}
```

This allows one system timer to serve many FSM states instead of creating a separate hardware timer for each state.

Using unsigned arithmetic this way also handles timer-counter wraparound correctly, provided the timeout interval is within the valid range of the counter arithmetic.

**Counter: Reset timer on entering state?**

Yes. Normally the state's timeout reference should be reset whenever entering the state.

Example:

```text
STATE_A
  |
  | transition
  v
STATE_B
  |
  | enter STATE_B
  v
record entry timestamp
```

**Interview point:**

> A practical embedded design often uses one system tick and records the entry timestamp for each timed state instead of creating separate hardware timers.

---

#### 165. Parallel state machines (multiple independent FSMs).

**Answer**

Parallel state machines are useful when several parts of a system operate independently.

For example:

```text
Main scheduler
    |
    +---- Communication FSM
    |
    +---- Motor FSM
    |
    +---- Power FSM
    |
    +---- Diagnostic FSM
```

Each FSM manages its own state and events.

**Counter: Synchronize outputs from multiple FSMs?**

Yes.

Suppose:

```text
Motor FSM      → requests motor ON
Safety FSM     → says motor NOT allowed
```

The final output must be resolved by a defined priority or arbitration rule.

For example:

```text
Safety permission
        AND
Motor request
        ↓
Actual motor command
```

This is safer than allowing two FSMs to directly control the same hardware resource independently.

**Counter: Resource contention between state machines?**

This is an important issue.

If two FSMs share:

- UART
- SPI
- DMA
- actuator
- buffer
- hardware peripheral

then ownership must be defined.

Possible mechanisms include:

- Resource manager
- Queue
- Mutex
- Arbitration
- Single owner FSM

**Counter: Timing guarantee across all FSMs?**

The scheduler must ensure that the combined execution time fits the available CPU budget.

Example:

```text
Task/FSM 1 = 100 us
Task/FSM 2 = 150 us
Task/FSM 3 = 200 us

Total = 450 us
```

If they must all execute within a 1 ms cycle, the budget needs to include interrupt and scheduling overhead as well.

For real-time systems, use worst-case execution time rather than average execution time.

**Interview point:**

> Parallel FSMs are useful for independent behavior, but shared resources and timing must be explicitly coordinated.

---

#### 166. Event-driven state machine vs time-driven.

**Answer**

An **event-driven FSM** changes state when an event occurs.

A **time-driven FSM** is evaluated periodically, usually from a timer tick.

Event-driven:

```text
UART RX event
    ↓
FSM executes
```

Time-driven:

```text
1 ms tick
    ↓
FSM executes
```

**Counter: Interrupt on event or polled at fixed rate?**

It depends on the application.

Event-driven systems can use:

- Interrupts
- Queues
- Message notifications
- Callbacks

Time-driven systems can use:

- Periodic scheduler
- Timer tick
- RTOS task
- Main loop

**Counter: Responsiveness vs determinism?**

Event-driven:

```text
+ Fast reaction to events
+ Can avoid unnecessary polling
- More asynchronous behavior
- Concurrency can be more difficult
```

Time-driven:

```text
+ Predictable execution points
+ Easier periodic behavior
+ Often easier to analyze
- Response can be delayed until the next tick
- May perform unnecessary checks
```

**Counter: Which is safer for safety-critical?**

There is no universal answer.

Safety-critical design should be based on the system's timing, safety, diagnostic, and certification requirements.

A time-driven design can make scheduling and timing analysis easier in some systems.

An event-driven design may be appropriate when immediate reaction to asynchronous events is required.

Many real systems use a hybrid architecture:

```text
Interrupt/event
      ↓
Queue/event flag
      ↓
Deterministic task
      ↓
FSM
```

**Interview point:**

> Safety comes from the overall architecture, timing analysis, fault handling, and verification—not simply from choosing event-driven or time-driven.

---

#### 167. State history: remember previous state for recovery.

**Answer**

State history means storing the previous valid state so the system can use it for diagnostics or recovery.

Example:

```c
typedef enum
{
    STATE_IDLE,
    STATE_RUNNING,
    STATE_ERROR
} State;

static State current_state = STATE_IDLE;
static State previous_state = STATE_IDLE;

void transition_to(State new_state)
{
    previous_state = current_state;
    current_state = new_state;
}
```

**Counter: Return to last valid state on error?**

Sometimes, but not automatically.

Example:

```text
RUNNING
   |
   | temporary fault
   v
RECOVERY
   |
   | successful recovery
   v
RUNNING
```

However, if the fault may have compromised the previous state, returning directly to it can be unsafe.

A recovery policy should explicitly determine whether to:

- Retry
- Return to previous state
- Go to a safe state
- Reinitialize
- Reset the system

**Counter: What if error occurred in middle of transition?**

This is why entry and exit operations should be designed carefully.

Avoid having transitions that leave hardware partially configured.

A useful approach is:

```text
Prepare
  ↓
Validate
  ↓
Commit new state
```

or use an explicit `RECOVERY`/`ERROR` state when a transition can fail midway.

**Counter: Persistent history (EEPROM)?**

It can be used when information must survive reset or power loss, but do not write persistent memory on every transition unnecessarily.

EEPROM/Flash has:

- Limited write endurance
- Write latency
- Power-failure considerations

Persistent history is more appropriate for significant events such as:

```text
Last reset reason
Critical fault code
Boot failure information
Recovery attempt count
```

rather than every normal state transition.

**Interview point:**

> Previous-state tracking is useful for diagnostics and recovery, but recovery should return to a previous state only when that state is still known to be safe.

---

#### 168. Conditional state entry (entry guards).

**Answer**

An entry guard checks whether the system is allowed to enter a state before the transition is completed.

Example:

```text
READY
  |
  | START
  |
  | guard:
  | motor_ready &&
  | temperature_ok
  v
RUNNING
```

If the guard fails, the system remains in the current state or follows an explicitly defined alternate transition.

**Counter: Check preconditions before entering state?**

Yes.

Typical preconditions could include:

```text
Hardware initialized
Sensor valid
Voltage within range
Communication available
Safety interlock active
Resource acquired
```

**Counter: Reject entry and stay in previous state?**

Yes, that is a common behavior.

Example:

```c
if (motor_ready && temperature_ok)
{
    state = RUNNING;
}
else
{
    state = READY;
}
```

However, for some conditions it may be better to transition to a dedicated `FAULT` or `WAITING_FOR_RESOURCE` state rather than silently remain in the previous state.

**Counter: Log rejected transitions?**

For important or safety-relevant transitions, logging rejected transitions can be useful for diagnostics.

Example:

```c
if (!motor_ready)
{
    log_event(EVENT_START_REJECTED);
}
```

But avoid excessive logging if the condition can happen continuously, because it can:

- Increase CPU load
- Fill diagnostic buffers
- Increase flash wear if persisted
- Hide more important events

Rate-limited or event-on-change logging is often better.

**Interview point:**

> An entry guard enforces the preconditions of a state. If the guard fails, the FSM should follow an explicitly defined alternative path and, where appropriate, record the reason.

---
