# 3. RING BUFFERS & CIRCULAR QUEUES

## BEGINNER

### 101. Implement ring buffer for UART receive.

**Answer**

A ring buffer, also called a circular buffer, is commonly used for UART reception because the producer and consumer can operate independently.

A typical design contains:

```text
Fixed array
    +
head index
    +
tail index
```

For UART:

```text
UART ISR / DMA
      ↓
    WRITE
      ↓
  Ring Buffer
      ↓
    READ
      ↓
Main task
```

The UART interrupt receives bytes and inserts them into the buffer. The application removes bytes when it is ready.

A simple single-producer/single-consumer implementation is:

```c
#include <stdint.h>
#include <stdbool.h>
#include <stddef.h>

#define RING_SIZE 128U

typedef struct
{
    uint8_t buffer[RING_SIZE];

    /*
     * head = position where producer writes next.
     */
    volatile size_t head;

    /*
     * tail = position where consumer reads next.
     */
    volatile size_t tail;

} RingBuffer;

static void ring_init(RingBuffer *rb)
{
    rb->head = 0U;
    rb->tail = 0U;
}

static bool ring_is_empty(const RingBuffer *rb)
{
    return rb->head == rb->tail;
}

static bool ring_is_full(const RingBuffer *rb)
{
    /*
     * One slot is intentionally left unused.
     *
     * If head is one position behind tail, the buffer is full.
     */
    size_t next_head = rb->head + 1U;

    if (next_head >= RING_SIZE)
    {
        next_head = 0U;
    }

    return next_head == rb->tail;
}

static bool ring_put(RingBuffer *rb, uint8_t value)
{
    if (ring_is_full(rb))
    {
        return false;
    }

    rb->buffer[rb->head] = value;

    rb->head++;

    if (rb->head >= RING_SIZE)
    {
        rb->head = 0U;
    }

    return true;
}

static bool ring_get(RingBuffer *rb, uint8_t *value)
{
    if (ring_is_empty(rb))
    {
        return false;
    }

    *value = rb->buffer[rb->tail];

    rb->tail++;

    if (rb->tail >= RING_SIZE)
    {
        rb->tail = 0U;
    }

    return true;
}
```

**Counter: Fixed array, head, tail pointers?**

Yes. The simplest ring buffer uses:

```text
buffer[]
head
tail
size/capacity
```

`head` identifies where new data is written.

`tail` identifies where the oldest unread data is stored.

In many implementations they are indices rather than actual pointers.

**Counter: Full vs empty detection: reserved bit or counter?**

Several approaches are possible.

**Approach 1: Reserve one slot**

```text
empty:
    head == tail

full:
    next(head) == tail
```

For an array of 128 elements, only 127 elements can be stored.

This is simple and avoids maintaining an extra count.

**Approach 2: Maintain a count**

```c
size_t count;
```

Then:

```text
empty:
    count == 0

full:
    count == RING_SIZE
```

This allows all 128 slots to be used, but `count` becomes shared state that must be updated safely if producer and consumer execute concurrently.

**Approach 3: Wider monotonic counters**

Some lock-free designs use continuously increasing producer/consumer counters and calculate occupancy using unsigned arithmetic.

**Counter: Thread-safe without locks?**

For a single producer and a single consumer, a ring buffer can often be implemented lock-free if:

```text
Producer owns head
Consumer owns tail
```

and the memory-ordering requirements of the target architecture are respected.

For example:

```text
Producer:
    write data
    publish head

Consumer:
    observe head
    read data
    publish tail
```

On modern multicore systems, memory barriers/atomic operations may be required.

On a simple MCU with a single core and carefully designed ISR/main interaction, interrupt atomicity and variable width still need to be considered.

Do not claim that every `volatile` ring buffer is automatically thread-safe.

---

### 102. Check if ring buffer is empty.

**Answer**

In the common one-empty-slot design:

```c
static bool ring_is_empty(const RingBuffer *rb)
{
    return rb->head == rb->tail;
}
```

The buffer is empty when there is no unread element.

```text
head == tail
    ↓
EMPTY
```

**Counter: head == tail condition?**

Yes, for the standard reserved-slot implementation.

**Counter: Need to handle wrap-around?**

The equality itself does not require explicit wrap handling because both indexes wrap to the beginning of the array.

Example:

```text
head = 127
head + 1 → 0
```

The implementation must ensure the index remains within the buffer range.

**Counter: What if not initialized?**

Uninitialized `head` and `tail` can point to arbitrary positions.

That can make the buffer appear:

```text
empty
```

or:

```text
non-empty
```

incorrectly.

Always initialize:

```c
rb->head = 0U;
rb->tail = 0U;
```

during startup or reset.

For safety-critical code, initialization should also be validated if corrupted startup state is a concern.

**Interview point:**

> “In the standard reserved-slot ring buffer, empty is `head == tail`. Both indexes must be initialized and wrapped safely.”

---

### 103. Calculate space available in ring buffer.

**Answer**

For a ring buffer that deliberately leaves one slot unused, the available space is:

```text
capacity - 1 - current_count
```

A common index-based formula is:

```text
free =
    (tail - head - 1 + SIZE) % SIZE
```

For power-of-two sizes, the implementation can be simplified using a mask.

Example:

```c
#include <stdint.h>
#include <stddef.h>

#define RING_SIZE 128U

static size_t ring_free_space(size_t head, size_t tail)
{
    /*
     * One slot is reserved to distinguish full from empty.
     */
    if (tail > head)
    {
        return tail - head - 1U;
    }

    return RING_SIZE - head + tail - 1U;
}
```

**Counter: `(tail - head - 1) mod size`?**

Yes, conceptually:

```text
free = (tail - head - 1 + size) % size
```

The `+ size` avoids a negative intermediate in ordinary signed arithmetic.

Using explicit branches can be clearer in embedded C and may avoid an expensive modulo operation on targets without fast division.

**Counter: Integer overflow risk?**

If using fixed-size indexes and ordinary array indexes:

```c
size_t
```

with explicit wrap handling, the risk is low.

If using continuously increasing counters, unsigned arithmetic is useful because it naturally wraps modulo `2^N`, but the implementation must obey the range assumptions of the design.

Do not mix signed and unsigned arithmetic carelessly.

**Counter: Check before enqueue operation?**

Yes.

Before inserting an item:

```text
Check available space
    ↓
If enough space → enqueue
Else → overflow policy
```

For a single byte:

```text
if (!ring_is_full())
    enqueue();
else
    handle_overflow();
```

For a multi-byte message, check that the entire message fits before writing it if the API requires atomic message insertion.

**Interview point:**

> “Space calculation depends on the chosen full/empty representation. I should avoid arithmetic that can overflow or become negative and should check capacity before writing.”

---

### 104. Enqueue element into ring buffer.

**Answer**

Enqueue means adding a new element at the producer/head position.

The standard order is:

```text
1. Check full
2. Write data
3. Advance head
```

Example:

```c
static bool ring_put(RingBuffer *rb, uint8_t value)
{
    /*
     * Do not overwrite unread data in this version.
     */
    size_t next_head = rb->head + 1U;

    if (next_head >= RING_SIZE)
    {
        next_head = 0U;
    }

    if (next_head == rb->tail)
    {
        /*
         * No free space.
         */
        return false;
    }

    /*
     * Store the value first.
     */
    rb->buffer[rb->head] = value;

    /*
     * Publish the new head only after the data is stored.
     */
    rb->head = next_head;

    return true;
}
```

**Counter: Check full before insert?**

Yes.

Otherwise the producer may overwrite unread data.

**Counter: Overwrite oldest if full?**

There are two valid policies.

**Policy 1: Reject new data**

```text
FULL
 ↓
drop new item
```

Useful when old data is more valuable or loss must be reported.

**Policy 2: Overwrite oldest data**

```text
FULL
 ↓
discard oldest
 ↓
store new item
```

Useful for telemetry or "latest value matters" data.

Example:

```c
if (next_head == rb->tail)
{
    /*
     * Drop oldest item by moving tail forward.
     */
    rb->tail++;

    if (rb->tail >= RING_SIZE)
    {
        rb->tail = 0U;
    }
}
```

This changes the semantics of the buffer, so it should be explicit in the API.

**Counter: Update tail pointer safely?**

If overwrite-on-full is used, the producer modifies `tail`, which can break a simple single-producer/single-consumer ownership model.

Therefore:

> In a lock-free SPSC design, it is usually cleaner for the producer to own only `head` and the consumer to own only `tail`.

If overwrite semantics require both sides to modify indexes, use appropriate synchronization.

**Interview point:**

> “The normal enqueue sequence is check capacity, write data, then publish the new head. If the buffer is full, the design must explicitly choose between dropping the new item and overwriting old data.”

---

### 105. Dequeue element from ring buffer.

**Answer**

Dequeue means removing the oldest unread element.

The standard order is:

```text
1. Check empty
2. Read data
3. Advance tail
```

Example:

```c
static bool ring_get(RingBuffer *rb, uint8_t *value)
{
    if (rb->head == rb->tail)
    {
        /*
         * Nothing available.
         */
        return false;
    }

    /*
     * Read oldest data.
     */
    *value = rb->buffer[rb->tail];

    /*
     * Move to next element.
     */
    rb->tail++;

    if (rb->tail >= RING_SIZE)
    {
        rb->tail = 0U;
    }

    return true;
}
```

**Counter: Check empty before dequeue?**

Yes.

Without checking:

```text
tail == head
```

the consumer may read stale or invalid data.

**Counter: What if no data available?**

The API must define the behavior.

Common choices:

```text
return false
return error code
return "no data" status
block until data arrives
```

For ISR-oriented ring buffers, non-blocking behavior is usually preferred.

**Counter: Blocking vs non-blocking behavior?**

**Non-blocking:**

```text
No data
   ↓
Return immediately
```

Useful in:

```text
ISR
main loop
high-priority task
polling code
```

**Blocking:**

```text
No data
   ↓
Task waits
   ↓
Wake when data arrives
```

Useful in an RTOS when waiting is expected and does not violate timing requirements.

**Interview point:**

> “For a UART ISR ring buffer, I normally use non-blocking enqueue/dequeue. A task can use a blocking API at a higher abstraction level if the RTOS and timing requirements allow it.”

---

### 106. Ring buffer for real-time sensor data.

**Answer**

A ring buffer is useful when sensor production and processing happen at different rates.

Example:

```text
Sensor:
    1 sample / 1 ms

Processing:
    sometimes takes 1-2 ms
```

The buffer absorbs short-term bursts.

```text
Sensor ISR
    ↓
Ring buffer
    ↓
Processing task
```

**Counter: What if samples arrive faster than processing?**

If the average production rate is higher than the average consumption rate, the buffer will eventually fill.

Example:

```text
Producer = 100 samples/s
Consumer = 80 samples/s
```

The backlog grows continuously.

No finite ring buffer can solve a permanent rate mismatch.

Eventually:

```text
FULL
```

You need one or more of:

```text
Faster processing
Data decimation
Larger buffer
Dropping policy
Backpressure
Lower sensor rate
Dedicated processing
```

**Counter: Overwrite or drop policy?**

Depends on the data.

**Latest sample is most important:**

```text
overwrite oldest
```

Typical for:

```text
telemetry
control display
latest sensor state
```

**Every sample is important:**

```text
do not overwrite silently
```

Instead:

```text
drop with diagnostic
backpressure if possible
increase processing capacity
```

**Counter: Average latency guarantee?**

Average latency is not enough for a hard real-time system.

You should consider:

```text
Worst-case latency
Worst-case queue depth
Maximum burst size
Worst-case processing time
Deadline
```

A ring buffer can absorb finite bursts, but it cannot hide a permanent overload.

A useful sizing idea is:

```text
Required buffer capacity
    >=
maximum burst arrivals
    -
minimum guaranteed processing
```

The exact equation depends on timing assumptions.

**Interview point:**

> “For real-time sensor data, I size the buffer from worst-case burst behavior and service latency, not from average rates alone. I also explicitly define whether old or new samples are dropped on overflow.”

---

### 107. Peek at next element without removing.

**Answer**

Peek means reading the next element without moving `tail`.

Example:

```c
static bool ring_peek(const RingBuffer *rb, uint8_t *value)
{
    if (rb->head == rb->tail)
    {
        return false;
    }

    *value = rb->buffer[rb->tail];

    /*
     * tail is NOT modified.
     */
    return true;
}
```

**Counter: Useful for protocol parsing?**

Yes.

Suppose a protocol starts with:

```text
[SYNC][LENGTH][PAYLOAD][CRC]
```

Before consuming bytes, a parser may need to inspect:

```text
SYNC
LENGTH
```

to know whether the complete message is already available.

Peek allows inspection without removing bytes prematurely.

**Counter: Can you peek multiple elements ahead?**

Yes.

For a ring of size `RING_SIZE`:

```c
static bool ring_peek_at(const RingBuffer *rb,
                         size_t offset,
                         uint8_t *value)
{
    size_t available;

    if (rb->head >= rb->tail)
    {
        available = rb->head - rb->tail;
    }
    else
    {
        available = RING_SIZE - rb->tail + rb->head;
    }

    if (offset >= available)
    {
        return false;
    }

    size_t index = rb->tail + offset;

    if (index >= RING_SIZE)
    {
        index -= RING_SIZE;
    }

    *value = rb->buffer[index];

    return true;
}
```

This allows:

```text
peek(0) → next byte
peek(1) → one byte ahead
peek(2) → two bytes ahead
```

**Counter: Thread-safe peeking?**

The same producer/consumer concurrency rules apply.

In an SPSC design, the consumer can safely inspect data that the producer has already published, provided the memory ordering is correct.

Do not inspect an element before the producer has committed/published it.

**Interview point:**

> “Peek is especially useful for packet parsers because it lets me inspect headers and lengths without consuming data until I know the complete message is available.”

---

### 108. Clear/flush ring buffer without deallocation.

**Answer**

For a normal ring buffer, clearing it does not require deallocating memory.

The simplest operation is:

```c
static void ring_clear(RingBuffer *rb)
{
    rb->head = 0U;
    rb->tail = 0U;
}
```

If both indexes are reset consistently, the buffer becomes empty.

**Counter: Reset head and tail only?**

Usually yes.

There is no need to erase every array element because those values are considered invalid once the indexes are reset.

This is:

```text
O(1)
```

instead of:

```text
O(N)
```

for clearing every byte.

**Counter: Scrub data for security?**

If the buffer contains sensitive data, simply resetting head/tail does not erase the old bytes from RAM.

For sensitive data:

```c
static void ring_secure_clear(RingBuffer *rb)
{
    for (size_t i = 0U; i < RING_SIZE; ++i)
    {
        rb->buffer[i] = 0U;
    }

    rb->head = 0U;
    rb->tail = 0U;
}
```

For security-sensitive applications, the compiler must not optimize away required memory erasure; use an appropriate secure-zeroing primitive for the platform.

**Counter: What if currently being read from ISR?**

This is a concurrency problem.

If an ISR can enqueue or dequeue while another context clears the buffer, simply resetting both indexes can race with the ISR.

Possible approaches include:

```text
Disable the relevant interrupt briefly
Use a lock if appropriate
Use a dedicated synchronization protocol
Perform clear from the owning context
```

For an ISR/main-loop SPSC buffer, the simplest safe design is often to define exactly which context owns which index and perform flush in a controlled critical section.

**Interview point:**

> “Normal flush is O(1): reset the producer and consumer indices. But if the buffer is shared with an ISR or another thread, the reset itself must be synchronized.”

---

## INTERMEDIATE

### 116. Multi-threaded ring buffer (producer-consumer).

**Answer**

A multi-threaded ring buffer has one or more producers and one or more consumers.

The concurrency problem is more complex than a simple SPSC ring.

Example:

```text
Producer A ──┐
             ├──> Ring Buffer ──> Consumer A
Producer B ──┘                  └> Consumer B
```

With multiple producers, two producers could attempt to write the same slot simultaneously.

With multiple consumers, two consumers could attempt to remove the same element.

Therefore synchronization is required unless a carefully designed lock-free multi-producer/multi-consumer algorithm is used.

**Counter: Spinlock vs condition variable?**

A **spinlock** keeps the CPU busy while waiting.

```text
Thread A owns lock
Thread B spins
Thread B keeps checking
```

Advantages:

```text
Very low waiting latency for very short critical sections
No scheduler sleep/wake transition
```

Disadvantages:

```text
Burns CPU
Can be harmful on low-power systems
Can be terrible for lower-priority tasks
```

A **condition variable** or semaphore allows a thread to sleep until data or space becomes available.

```text
Consumer waits
    ↓
Producer adds data
    ↓
Signal condition
    ↓
Consumer wakes
```

This is often more appropriate for tasks that may wait for non-trivial periods.

**Counter: Priority inversion with multiple producers?**

Yes.

Suppose:

```text
High-priority producer → wants lock
Low-priority producer  → owns lock
Medium-priority task   → runs
```

The high-priority producer can be indirectly delayed.

Use appropriate protocols such as:

```text
Priority inheritance mutex
Priority ceiling
Short critical sections
Lock-free design where justified
```

**Counter: Lock-free implementation possible?**

Yes, but it is significantly more complex.

SPSC queues are comparatively straightforward to make lock-free because:

```text
One owner of head
One owner of tail
```

MPSC/MPMC queues require careful use of:

```text
Atomic operations
Memory ordering
CAS
ABA considerations in some algorithms
Correct publication/consumption ordering
```

Do not write a "lock-free" queue without understanding the target memory model.

**Interview point:**

> “For SPSC, a lock-free ring buffer is relatively simple. For MPMC, I need real atomic synchronization and memory-ordering guarantees; otherwise I would use a proven queue primitive rather than inventing one.”

---

### 117. Ring buffer with variable-length messages.

**Answer**

A byte ring buffer stores a continuous stream, but variable-length messages need framing.

A common method is:

```text
[LENGTH][PAYLOAD]
```

Example:

```text
[0x05][H][E][L][L][O]
```

The length tells the consumer how many bytes belong to the message.

A message can therefore be represented as:

```text
+--------+----------------------+
| Length | Payload              |
+--------+----------------------+
```

**Counter: How to store message length prefix?**

For example:

```c
typedef struct
{
    uint16_t length;
    uint8_t  data[256];
} Message;
```

Or directly inside a byte ring:

```text
byte 0,1 → message length
byte 2... → message payload
```

If the length itself may straddle the physical end of the ring, the implementation must correctly handle wrap-around.

A safer approach for many systems is to reserve enough metadata space and use helper functions that perform wrapped reads/writes.

**Counter: Handle partial message (ISR interrupted)?**

The producer must publish the message only after the complete message has been written.

For example:

```text
Reserve space
    ↓
Write length
    ↓
Write payload
    ↓
Publish message
```

Do not advance the public producer index before the entire message is valid if consumers are allowed to read concurrently.

Otherwise the consumer could see:

```text
length = 100
payload = only 40 bytes written
```

This is an incomplete message.

Another approach is a commit marker:

```text
[LENGTH][PAYLOAD][COMMIT]
```

The exact design depends on the concurrency model.

**Counter: Memory waste with padding?**

If the next message does not fit contiguously before the physical end of the ring, some designs insert padding or wrap markers.

Example:

```text
[end of buffer]
[unused padding]
[message starts at index 0]
```

This wastes some capacity but simplifies message handling.

Alternative designs split the message across the boundary, which uses memory more efficiently but makes implementation more complex.

**Interview point:**

> “For variable-length messages I need framing, atomic message publication, and a defined wrap-around strategy. I cannot let the consumer observe a partially written message.”

---

### 118. Ring buffer for DMA-based UART.

**Answer**

A DMA-based UART can write received bytes directly into RAM, reducing CPU interrupt overhead.

A common architecture is:

```text
UART RX
   ↓
DMA
   ↓
RAM ring buffer
   ↓
CPU/parser
```

The DMA hardware acts as the producer.

**Counter: DMA writes to ring buffer directly?**

Yes.

Many MCUs support circular DMA mode.

Example concept:

```text
DMA destination:
    buffer[0 ... N-1]

DMA writes:
    0 → 1 → 2 → ... → N-1 → 0 → ...
```

The CPU determines how much new data has arrived by comparing the DMA's current write position with the software's consumed position.

**Counter: Head pointer: who updates (ISR or main)?**

In a DMA design, the DMA engine owns the actual hardware write position.

Software derives a logical head from:

```text
DMA current position
```

or updates a software head when handling DMA interrupts/events.

The consumer usually owns `tail`.

Conceptually:

```text
DMA hardware
    ↓
produces bytes
    ↓
software calculates head
    ↓
consumer reads until tail catches head
```

**Counter: Synchronization between DMA and CPU?**

This is critical.

The CPU must not read data that DMA has not finished writing.

Possible mechanisms include:

```text
DMA half-transfer interrupt
DMA transfer-complete interrupt
UART idle-line detection
DMA current-address polling
Memory barriers where required
Cache maintenance on cached systems
```

On systems with data cache, DMA/CPU cache coherency must be explicitly handled.

For example:

```text
DMA writes RAM
CPU cache still contains old data
CPU reads stale bytes
```

The solution may involve cache invalidation or a non-cacheable DMA region, depending on the architecture.

**Interview point:**

> “With DMA UART reception, DMA owns the producer side. Software tracks the consumer index and derives how much data has arrived from DMA's current position. I also need to consider synchronization, cache coherency, DMA wrap-around, and frame-boundary detection.”

---

### 119. Detect overflow in ring buffer (dropped data).

**Answer**

A ring buffer overflow occurs when the producer wants to add data but there is no free space.

A robust design should make overflow visible rather than silently losing data.

Example:

```c
typedef struct
{
    uint8_t buffer[RING_SIZE];

    size_t head;
    size_t tail;

    uint32_t overflow_count;

    bool overflowed;

} RingBuffer;
```

When full:

```c
if (ring_is_full(rb))
{
    rb->overflow_count++;
    rb->overflowed = true;

    return false;
}
```

**Counter: Counter for overflow count?**

Yes. A counter is useful for diagnostics:

```text
overflow_count = number of observed overflow events
```

This is valuable because the system may continue operating while the diagnostic shows that data was lost.

**Counter: Set flag and continue, or halt?**

Depends on the criticality of the data.

For telemetry:

```text
set flag
increment counter
drop/overwrite according to policy
continue
```

For safety-critical control data:

```text
overflow may be a fault
→ enter degraded mode
→ stop actuator
→ report diagnostic
```

The reaction should come from system requirements.

**Counter: Recoverable vs fatal overflow?**

**Recoverable:**

If losing one or more samples is acceptable:

```text
detect
log
continue
```

**Fatal:**

If every message is required:

```text
detect
enter fault state
stop or reset subsystem
```

**Important distinction**

An overflow counter tells you that data was lost.

It does not recover the lost data.

Recovery requires either:

```text
retransmission
reacquisition
resynchronization
upstream backpressure
```

depending on the system.

**Interview point:**

> “I would always make overflow observable through a counter/status flag. Whether the system continues or enters a fault state depends on whether lost data is recoverable and what the safety requirements are.”

---

### 120. Ring buffer with timestamp on enqueue.

**Answer**

A timestamp can be stored together with each ring-buffer element.

Instead of:

```text
uint8_t data
```

the ring can store:

```c
typedef struct
{
    uint32_t timestamp;
    uint8_t  data;

} TimestampedByte;
```

Example:

```c
#define RING_SIZE 128U

typedef struct
{
    TimestampedByte buffer[RING_SIZE];

    size_t head;
    size_t tail;

} TimestampRing;
```

During enqueue:

```c
static bool timestamp_ring_put(TimestampRing *rb,
                               uint8_t data,
                               uint32_t timestamp)
{
    size_t next_head = rb->head + 1U;

    if (next_head >= RING_SIZE)
    {
        next_head = 0U;
    }

    if (next_head == rb->tail)
    {
        return false;
    }

    rb->buffer[rb->head].timestamp = timestamp;
    rb->buffer[rb->head].data = data;

    rb->head = next_head;

    return true;
}
```

**Counter: Include timestamp in each element?**

Yes, if each sample/message needs an accurate arrival time.

But storing a timestamp for every byte can be wasteful for high-rate UART data.

A more efficient design may timestamp:

```text
each message
```

instead of:

```text
each byte
```

or use one timestamp plus a known sample period.

**Counter: Time synchronization across systems?**

If timestamps come from different devices, their clocks may not be synchronized.

Possible problems:

```text
Clock offset
Clock drift
Different timer resolutions
Reset/reboot
Wrap-around
```

If comparing timestamps across systems, use an appropriate synchronization mechanism such as:

```text
PTP
synchronized RTC
CAN/network time synchronization
application-level timestamp correction
```

The correct method depends on the network/system.

**Counter: Fixed overhead per message?**

Yes.

For example:

```text
uint8_t data      = 1 byte

uint32_t timestamp = 4 bytes

Total = 5 bytes
```

The metadata can dominate memory for small messages.

For a 128-entry buffer:

```text
128 × 1 byte  = 128 bytes

128 × 5 bytes = 640 bytes
```

Therefore timestamp resolution and granularity should be chosen based on the requirement.

**Alternative: timestamp once per message**

```c
typedef struct
{
    uint32_t timestamp;
    uint16_t length;
    uint8_t  payload[PAYLOAD_MAX];

} TimestampedMessage;
```

This is often more memory-efficient for packet-oriented systems.

**Interview point:**

> “Timestamping is useful when I need event timing or latency measurement, but I would choose timestamp granularity carefully because timestamp metadata increases memory usage.”

---

# RING BUFFER — INTERVIEW CHEAT SHEET

## Basic Ring Buffer

```text
             +---------------------+
             |     RING BUFFER     |
             |                     |
Producer --->| head          tail  |---> Consumer
             |                     |
             +---------------------+
```

Typical ownership:

```text
Producer → head
Consumer → tail
```

---

## Empty

For the reserved-slot design:

```c
head == tail
```

---

## Full

```c
next(head) == tail
```

One slot is sacrificed so that:

```text
head == tail
```

can uniquely mean empty.

---

## Enqueue

```text
1. Check full
2. Write data
3. Advance/publish head
```

---

## Dequeue

```text
1. Check empty
2. Read data
3. Advance tail
```

---

## Space

Reserved-slot design:

```text
free = (tail - head - 1 + size) % size
```

---

## Overflow policies

```text
DROP NEW
    ↓
Preserve old data

OVERWRITE OLD
    ↓
Preserve latest data
```

Choose based on application semantics.

---

## SPSC

Single Producer / Single Consumer:

```text
ISR/DMA ─────> Ring Buffer ─────> Main/Task
     owns head                 owns tail
```

This is the easiest case for a lock-free design.

---

## MPMC

Multiple Producer / Multiple Consumer:

```text
Producer A ─┐
Producer B ─┼──> Queue ──> Consumer A
Producer C ─┘             Consumer B
```

Requires more sophisticated synchronization.

---

## DMA UART Ring Buffer

```text
UART
  ↓
DMA
  ↓
RAM circular buffer
  ↓
CPU determines new-data region
  ↓
Parser
```

Important concerns:

```text
DMA position
Cache coherency
Memory ordering
Idle-line detection
Half/full transfer events
Wrap-around
```

---

## Real-Time Sensor Buffer

Always consider:

```text
Producer rate
Consumer rate
Maximum burst
Worst-case processing time
Maximum acceptable latency
Overflow policy
```

A bigger buffer can absorb bursts, but it cannot solve permanent overload.

---

## Overflow Detection

Use:

```c
overflow_count++;
overflow_flag = true;
```

Then choose:

```text
Continue
Drop
Overwrite
Backpressure
Fault
```

based on requirements.

---

## Timestamped Ring

Basic element:

```c
typedef struct
{
    uint32_t timestamp;
    uint8_t data;
} Item;
```

But timestamp every byte only when the requirement justifies the memory cost.

---

# Strong Interview Answer for Ring Buffers

> **“A ring buffer is a fixed-size FIFO that uses head and tail indexes and wraps around at the buffer boundary. For a simple SPSC design, the producer owns head and the consumer owns tail, which can allow a lock-free implementation with proper memory ordering. I always define the full/empty policy, overflow behavior, concurrency model, and timing requirements. For DMA or ISR-driven data, I also account for synchronization, cache coherency where applicable, and worst-case burst behavior.”**
