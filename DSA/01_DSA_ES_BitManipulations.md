# Embedded Developer DSA Interview Questions — Resource-Constrained Systems
> ~500 Questions | Beginner → Intermediate → Advanced → Expert
> Topics: Arrays, State Machines, Bit Manipulation, Real-time Algorithms, Memory Management
> ⚠️ No answers — test yourself
> Format: Question → Counter Questions
> Focus: Embedded Systems, IoT, Firmware, Real-time Processing

---

## GRADING REFERENCE FOR EMBEDDED INTERVIEWS

| Level        | Expected Performance                          | Context                 |
| ------------ | --------------------------------------------- | ----------------------- |
| Beginner     | Solve in < 10 mins, explain memory impact     | Basic firmware tasks    |
| Intermediate | Solve in < 20 mins, optimize for RAM/ROM      | Real-time constraints   |
| Advanced     | Solve in < 30 mins, analyze worst-case timing | Critical systems        |
| Expert       | Design + prove correctness in < 45 mins       | Safety-critical systems |

---

## EMBEDDED-SPECIFIC DIFFICULTY MATRIX

| Topic                       | Beginner | Intermediate | Advanced | Expert   |
| --------------------------- | -------- | ------------ | -------- | -------- |
| Bit Manipulation            | Q1–20    | Q21–35       | Q36–45   | Q46–50   |
| Fixed-Size Arrays           | Q51–70   | Q71–85       | Q86–95   | Q96–100  |
| Ring Buffers & Queues       | Q101–115 | Q116–130     | Q131–140 | Q141–145 |
| State Machines              | Q146–160 | Q161–175     | Q176–185 | Q186–190 |
| Real-time Algorithms        | Q191–205 | Q206–220     | Q221–235 | Q236–240 |
| Memory Management           | Q241–255 | Q256–270     | Q271–280 | Q281–285 |
| Interrupt-Safe Patterns     | Q286–300 | Q301–315     | Q316–325 | Q326–330 |
| Sensor/Actuator Logic       | Q331–350 | Q351–365     | Q366–375 | Q376–380 |
| Protocol Parsing            | Q381–400 | Q401–415     | Q416–425 | Q426–430 |
| Power & Timing Optimization | Q431–450 | Q451–465     | Q466–475 | Q476–480 |
| Expert Round                | —        | —            | —        | Q481–500 |

---

*Embedded interviews prioritize: Memory awareness > Theoretical optimality*
*If you can explain WHY this fits in 4KB RAM, not just HOW — you are ready.*

---

## TABLE OF CONTENTS
1. Bit Manipulation & Hardware Registers
2. Fixed-Size Arrays & Stack Operations
3. Ring Buffers & Circular Queues
4. State Machines & Finite Automata
5. Real-time Scheduling Algorithms
6. Memory Management & Pooling
7. Interrupt-Safe Patterns & Atomicity
8. Sensor Data Processing
9. Protocol Parsing (CAN, UART, SPI)
10. Time-Critical Algorithms
11. Low-Power Optimization
12. Firmware Data Structures
13. Hardware Abstraction Layers (HAL)
14. Bootloader & Initialization Sequences
15. Error Handling & Watchdog Logic
16. PWM & Timer Control
17. ADC/DAC Signal Processing
18. Communication Protocols (I2C, SPI, Serial)
19. Cryptography in Embedded Systems
20. Automotive & Industrial Protocols

---

## 1. BIT MANIPULATION & HARDWARE REGISTERS

### BEGINNER

1. Read/Write specific bit in a register without affecting others.
   - Counter: Which bits to preserve?
   - Counter: Can you do it in a single operation?
   - Counter: What about multi-bit fields?
```c++
#include <cstdint>

/*
====================================================
Question 1
Read/Write specific bit or field without affecting others
====================================================

Counter Answer:
1) Which bits to preserve?
   Preserve all bits outside the target mask.

2) Can you do it in a single operation?
   You can write as one expression:
   reg = (reg & ~mask) | newBits;
   (Compiler may still emit multiple machine instructions.)

3) What about multi-bit fields?
   Use width-based mask and same clear-then-insert pattern.
*/

static inline uint32_t readBit(uint32_t reg, uint8_t bitPos)
{
    if (bitPos >= 32) return 0U;
    return (reg >> bitPos) & 1U;
}

static inline uint32_t writeBit(uint32_t reg, uint8_t bitPos, bool bitValue)
{
    if (bitPos >= 32) return reg;

    uint32_t mask = (1U << bitPos);
    reg &= ~mask;                     // clear target bit
    reg |= (uint32_t(bitValue) << bitPos); // insert new bit
    return reg;
}

static inline uint32_t writeField(uint32_t reg, uint32_t fieldValue, uint8_t pos, uint8_t width)
{
    if (pos >= 32 || width == 0 || (pos + width) > 32) return reg;

    uint32_t mask = (width == 32U) ? 0xFFFFFFFFU : ((1U << width) - 1U);
    uint32_t fieldMask = (mask << pos);

    reg &= ~fieldMask;                          // clear field
    reg |= ((fieldValue & mask) << pos);       // insert field
    return reg;
}
```


2. Check if a GPIO pin is set (read bit N).
   - Counter: What is the bit position?
   - Counter: How do you mask other bits?
   - Counter: What if register is volatile?

```c++
#include <cstdint>

/*
====================================================
Question 1
Read/Write specific bit or field without affecting others
====================================================

Counter Answer:
1) Which bits to preserve?
   Preserve all bits outside the target mask.

2) Can you do it in a single operation?
   You can write as one expression:
   reg = (reg & ~mask) | newBits;
   (Compiler may still emit multiple machine instructions.)

3) What about multi-bit fields?
   Use width-based mask and same clear-then-insert pattern.
*/

static inline uint32_t readBit(uint32_t reg, uint8_t bitPos)
{
    if (bitPos >= 32) return 0U;
    return (reg >> bitPos) & 1U;
}

static inline uint32_t writeBit(uint32_t reg, uint8_t bitPos, bool bitValue)
{
    if (bitPos >= 32) return reg;

    uint32_t mask = (1U << bitPos);
    reg &= ~mask;                     // clear target bit
    reg |= (uint32_t(bitValue) << bitPos); // insert new bit
    return reg;
}

static inline uint32_t writeField(uint32_t reg, uint32_t fieldValue, uint8_t pos, uint8_t width)
{
    if (pos >= 32 || width == 0 || (pos + width) > 32) return reg;

    uint32_t mask = (width == 32U) ? 0xFFFFFFFFU : ((1U << width) - 1U);
    uint32_t fieldMask = (mask << pos);

    reg &= ~fieldMask;                          // clear field
    reg |= ((fieldValue & mask) << pos);       // insert field
    return reg;
}
```

3. Set all bits in a register between positions M and N.
   - Counter: Create a mask efficiently?
   - Counter: What if M > N?
   - Counter: Handle shift overflow for large positions?

```c++
#include <cstdint>

/*
====================================================
Question 3
Set all bits from M to N (inclusive)
====================================================

Counter Answer:
1) Efficient mask:
   width = N - M + 1
   mask = ((1U << width) - 1U) << M
2) If M > N, swap or reject. Here we swap.
3) Guard all bounds and width==32 special case to avoid overflow.
*/

static inline uint32_t setBitRange(uint32_t reg, uint8_t m, uint8_t n)
{
    if (m >= 32 || n >= 32) return reg;
    if (m > n) { uint8_t t = m; m = n; n = t; }

    uint8_t width = uint8_t(n - m + 1U);
    uint32_t mask = (width == 32U) ? 0xFFFFFFFFU : (((1U << width) - 1U) << m);
    return reg | mask;
}
```


4. Toggle a specific bit without affecting others.
   - Counter: XOR vs bit shift — which is faster?
   - Counter: What if bit is already set/clear?
   - Counter: Atomic toggle for interrupt-safe code?

```c++
#include <cstdint>

/*
====================================================
Question 4
Toggle one bit
====================================================

Counter Answer:
1) Toggle uses XOR with single-bit mask:
   reg ^= (1U << bit).
   This is typically optimal.
2) If bit is 1 it becomes 0, if 0 it becomes 1.
3) Atomicity is hardware/OS dependent:
   use critical section or atomic register feature if required.
*/

static inline uint32_t toggleBit(uint32_t reg, uint8_t bit)
{
    if (bit >= 32) return reg;
    return reg ^ (1U << bit);
}
```

5. Extract M consecutive bits starting at position N.
   - Counter: Shift and mask order?
   - Counter: Big-endian vs little-endian?
   - Counter: Performance on 8-bit vs 32-bit systems?

```c++
#include <cstdint>

/*
====================================================
Question 5
Extract width bits from position pos
====================================================

Counter Answer:
1) Shift right first, then mask.
2) Endianness does not change bit math once value is in register.
3) Complexity remains O(1). 8-bit MCUs may use more instructions.
*/

static inline uint32_t extractBits(uint32_t value, uint8_t pos, uint8_t width)
{
    if (pos >= 32 || width == 0 || (pos + width) > 32) return 0U;
    uint32_t mask = (width == 32U) ? 0xFFFFFFFFU : ((1U << width) - 1U);
    return (value >> pos) & mask;
}
```


6. Count set bits in a status register.
   - Counter: Population count vs loop method?
   - Counter: When do you use __builtin_popcount()?
   - Counter: Inline assembly optimization?

```c++
#include <cstdint>

/*
====================================================
Question 6
Count set bits
====================================================

Counter Answer:
1) Builtin/popcount instruction is usually fastest.
2) Use __builtin_popcount when compiler supports it.
3) Inline assembly only if profiling proves compiler output is poor.
*/

static inline uint8_t countSetBits(uint32_t x)
{
#if defined(__GNUC__) || defined(__clang__)
    return static_cast<uint8_t>(__builtin_popcount(x));
#else
    uint8_t count = 0;
    while (x)
    {
        x &= (x - 1U); // Brian Kernighan trick
        ++count;
    }
    return count;
#endif
}
```

7. Find position of LSB (least significant bit) that is set.
   - Counter: `n & -n` trick — why does it work?
   - Counter: Performance cost vs loop?
   - Counter: What if no bits are set?

```c++
#include <cstdint>

/*
====================================================
Question 7
Find index of least significant set bit
====================================================

Counter Answer:
1) n & -n isolates lowest set bit in two's complement.
2) ctz builtin is usually faster than loop.
3) If input is zero, return -1.
*/

static inline int8_t lsbIndex(uint32_t x)
{
    if (x == 0U) return -1;
#if defined(__GNUC__) || defined(__clang__)
    return static_cast<int8_t>(__builtin_ctz(x));
#else
    int8_t idx = 0;
    while ((x & 1U) == 0U)
    {
        x >>= 1;
        ++idx;
    }
    return idx;
#endif
}
```


8. Swap two bits in a register without XOR.
   - Counter: Only using AND/OR operations?
   - Counter: What if bits are adjacent?
   - Counter: Cost in cycles on target MCU?

```c++
#include <cstdint>

/*
====================================================
Question 8
Swap two bits using only AND/OR
====================================================

Counter Answer:
1) Yes, read both bits then set/clear using AND/OR.
2) Adjacent bits are handled the same.
3) O(1); exact cycles depend on MCU and compiler.
*/

static inline uint32_t swapBitsNoXor(uint32_t value, uint8_t i, uint8_t j)
{
    if (i >= 32 || j >= 32 || i == j) return value;

    uint32_t bi = (value >> i) & 1U;
    uint32_t bj = (value >> j) & 1U;

    if (bi != bj)
    {
        if (bi) value |=  (1U << j); else value &= ~(1U << j);
        if (bj) value |=  (1U << i); else value &= ~(1U << i);
    }
    return value;
}
```

9. Check if register value is power of 2.
   - Counter: Why does `n & (n-1) == 0` work?
   - Counter: Handle zero and negative numbers?
   - Counter: Real-time constraint: must execute in < 10 cycles?

```c++
#include <cstdint>

/*
====================================================
Question 8
Swap two bits using only AND/OR
====================================================

Counter Answer:
1) Yes, read both bits then set/clear using AND/OR.
2) Adjacent bits are handled the same.
3) O(1); exact cycles depend on MCU and compiler.
*/

static inline uint32_t swapBitsNoXor(uint32_t value, uint8_t i, uint8_t j)
{
    if (i >= 32 || j >= 32 || i == j) return value;

    uint32_t bi = (value >> i) & 1U;
    uint32_t bj = (value >> j) & 1U;

    if (bi != bj)
    {
        if (bi) value |=  (1U << j); else value &= ~(1U << j);
        if (bj) value |=  (1U << i); else value &= ~(1U << i);
    }
    return value;
}
```


10. Clear all bits except the rightmost N bits.
    - Counter: Create mask without loop?
    - Counter: Edge cases: N = 0 or N = 32?
    - Counter: Constant expression for compile-time evaluation?

```c++
#include <cstdint>

/*
====================================================
Question 10
Keep only rightmost N bits
====================================================

Counter Answer:
1) mask = (1U << N) - 1 (with guard for N==32).
2) N==0 => 0. N>=32 => unchanged.
3) If N is constexpr, compiler folds mask at compile time.
*/

static inline uint32_t keepRightmostN(uint32_t value, uint8_t n)
{
    if (n == 0U) return 0U;
    if (n >= 32U) return value;
    return value & ((1U << n) - 1U);
}
```

11. Reverse bit order of a 32-bit register value.
    - Counter: Bit-by-bit vs lookup table approach?
    - Counter: Memory vs speed tradeoff for embedded?
    - Counter: ARM RBIT instruction — when available?


```c++
#include <cstdint>

/*
====================================================
Question 11
Reverse bits in uint32_t
====================================================

Counter Answer:
1) Bit-by-bit is smallest code, table can be faster.
2) Embedded tradeoff: Flash/RAM budget vs speed.
3) On ARM cores with RBIT support, compiler intrinsic is often fastest.
*/

static inline uint32_t reverseBits32(uint32_t x)
{
    x = ((x & 0x55555555U) << 1) | ((x >> 1) & 0x55555555U);
    x = ((x & 0x33333333U) << 2) | ((x >> 2) & 0x33333333U);
    x = ((x & 0x0F0F0F0FU) << 4) | ((x >> 4) & 0x0F0F0F0FU);
    x = ((x & 0x00FF00FFU) << 8) | ((x >> 8) & 0x00FF00FFU);
    x = (x << 16) | (x >> 16);
    return x;
}
```

12. Set bits at positions specified by a mask.
    - Counter: What if mask has non-contiguous bits?
    - Counter: Preserve original bits outside mask?
    - Counter: Atomic operation for multi-register updates?


```c++
#include <cstdint>

/*
====================================================
Question 12
Set masked bits
====================================================

Counter Answer:
1) Non-contiguous masks are naturally supported.
2) OR preserves all bits outside mask.
3) Multi-register atomicity requires lock/critical section/hardware support.
*/

static inline uint32_t setBitsByMask(uint32_t reg, uint32_t mask)
{
    return reg | mask;
}
```
13. Extract sign bit and check if number is negative.
    - Counter: MSB check vs arithmetic right shift?
    - Counter: Portable across signed/unsigned?
    - Counter: Cost difference on DSP vs ARM?

```c++
#include <cstdint>

/*
====================================================
Question 13
Sign bit extraction
====================================================

Counter Answer:
1) Portable approach: cast to uint32_t, then shift.
2) Works clearly with fixed-width types.
3) Usually one shift and compare; tiny constant-time operation.
*/

static inline uint32_t signBit(int32_t x)
{
    return (static_cast<uint32_t>(x) >> 31) & 1U;
}

static inline bool isNegative(int32_t x)
{
    return signBit(x) != 0U;
}
```

14. Interleave bits from two 16-bit values into 32-bit (Morton encoding).
    - Counter: Why used in spatial indexing?
    - Counter: Reverse operation (deinterleave)?
    - Counter: Performance: lookup table vs bit shifts?

```c++
#include <cstdint>

/*
====================================================
Question 14
Morton interleave and deinterleave
====================================================

Counter Answer:
1) Used for Z-order indexing to preserve locality.
2) Reverse operation exists by compacting alternating bits.
3) LUT is faster, shifts use less memory.
*/

static inline uint32_t spread16(uint16_t v)
{
    uint32_t x = v;
    x = (x | (x << 8)) & 0x00FF00FFU;
    x = (x | (x << 4)) & 0x0F0F0F0FU;
    x = (x | (x << 2)) & 0x33333333U;
    x = (x | (x << 1)) & 0x55555555U;
    return x;
}

static inline uint32_t mortonInterleave(uint16_t x, uint16_t y)
{
    return spread16(x) | (spread16(y) << 1);
}

static inline uint16_t compact32(uint32_t x)
{
    x &= 0x55555555U;
    x = (x | (x >> 1)) & 0x33333333U;
    x = (x | (x >> 2)) & 0x0F0F0F0FU;
    x = (x | (x >> 4)) & 0x00FF00FFU;
    x = (x | (x >> 8)) & 0x0000FFFFU;
    return static_cast<uint16_t>(x);
}

static inline void mortonDeinterleave(uint32_t code, uint16_t& x, uint16_t& y)
{
    x = compact32(code);
    y = compact32(code >> 1);
}
```

15. Gray code to binary conversion.
    - Counter: What is Gray code used for?
    - Counter: Why preferred in hardware counters?
    - Counter: Conversion algorithm: XOR tricks?

```c++
#include <cstdint>

/*
====================================================
Question 15
Gray to binary conversion
====================================================

Counter Answer:
1) Used in encoders and asynchronous boundaries.
2) Only one bit changes between adjacent values, reducing transition glitches.
3) Prefix XOR method converts Gray to binary in O(number of bits).
*/

static inline uint32_t grayToBinary(uint32_t gray)
{
    uint32_t bin = gray;
    while (gray >>= 1U)
    {
        bin ^= gray;
    }
    return bin;
}
```

---

### INTERMEDIATE

16. Implement bit field operations with multiple fields in a register.
    - Counter: How do you define field layout?
    - Counter: Endianness handling?
    - Counter: Does compiler optimize bitfield access?

```c++
/*
#include <cstdint>

====================================================
Question 16
Bit field operations with multiple fields in a register
====================================================

Answer:
- Each field is defined by a bit position and width.
- Use masks to clear the target field without disturbing other bits.
- Preserve all bits outside the field using:
    reg = (reg & ~mask) | (value << pos);
- Endianness is only relevant when reading raw bytes from memory.
- Compilers often optimize these operations well, but explicit masks are safest in embedded code.
*/

static inline uint32_t fieldMask(uint8_t pos, uint8_t width)
{
    if (width == 0U || pos >= 32U || (pos + width) > 32U)
        return 0U;

    return ((1U << width) - 1U) << pos;
}

static inline uint32_t readField(uint32_t reg, uint8_t pos, uint8_t width)
{
    return (reg & fieldMask(pos, width)) >> pos;
}

static inline uint32_t writeField(uint32_t reg, uint8_t pos, uint8_t width, uint32_t value)
{
    uint32_t mask = fieldMask(pos, width);
    return (reg & ~mask) | ((value << pos) & mask);
}
```

17. Parity bit calculation for error detection.
    - Counter: Even vs odd parity?
    - Counter: Hardware vs software implementation?
    - Counter: Real-time: must complete in 1 cycle?

```c++
#include <cstdint>

/*
====================================================
Question 17
Parity bit calculation for error detection
====================================================

Answer:
- Even parity means total number of set bits including parity is even.
- Odd parity means total number of set bits including parity is odd.
- Hardware parity is faster and cheaper in communication modules.
- Software parity is used where no hardware support exists.
- For strict real-time timing, hardware parity or optimized bitwise logic is preferred.
*/

static inline uint8_t evenParityBit(uint32_t x)
{
    uint8_t parity = 0U;

    while (x != 0U)
    {
        parity ^= (x & 1U);
        x >>= 1U;
    }

    return parity ^ 1U;   // make total count even
}

```


18. Find highest set bit (most significant bit) position.
    - Counter: Performance on 8/16/32-bit systems?
    - Counter: __builtin_clz() usage and portability?
    - Counter: If no bits set, return value convention?

```c++
#include <cstdint>

/*
====================================================
Question 18
Find highest set bit position
====================================================

Answer:
- This means locate the position of the most significant set bit.
- Use compiler builtins like __builtin_clz() when available.
- On 8/16/32-bit MCUs, width matters and the algorithm must match the CPU width.
- If no bit is set, return -1 as a standard convention.
*/

static inline int msbIndex(uint32_t x)
{
    if (x == 0U) return -1;

#if defined(__GNUC__) || defined(__clang__)
    return 31 - __builtin_clz(x);
#else
    int pos = -1;
    while (x != 0U)
    {
        x >>= 1U;
        ++pos;
    }
    return pos;
#endif
}
```

19. Rotate bits left/right by N positions.
    - Counter: Circular shift vs linear shift?
    - Counter: Is there a CPU instruction (ROR/ROL)?
    - Counter: Optimize for arbitrary rotation count?

```c++
#include <cstdint>

/*
====================================================
Question 19
Rotate bits left/right by N positions
====================================================

Answer:
- Rotation is a circular shift.
- Left rotate wraps bits from MSB back to LSB.
- Right rotate wraps bits from LSB back to MSB.
- CPU instructions like ROL/ROR may exist on some architectures.
- For arbitrary N, mask the shift count to avoid undefined behavior.
*/

static inline uint32_t rotl32(uint32_t value, uint8_t n)
{
    n &= 31U;
    return (value << n) | (value >> ((32U - n) & 31U));
}

static inline uint32_t rotr32(uint32_t value, uint8_t n)
{
    n &= 31U;
    return (value >> n) | (value << ((32U - n) & 31U));
}
```


20. Create register mask for clearing N bits at position M.
    - Counter: Without creating intermediate large values?
    - Counter: Avoid shift overflow on 8-bit MCUs?
    - Counter: Compile-time vs runtime mask creation?

```c++
#include <cstdint>

/*
====================================================
Question 20
Create register mask for clearing N bits at position M
====================================================

Answer:
- Build the bit mask using:
    mask = ((1U << N) - 1U) << M
- Then clear field bits with:
    reg = reg & ~mask;
- Guard for N == 0 and N == 32 to avoid invalid shifts.
- For fixed registers, compile-time masks are better than runtime masks.
*/

static inline uint32_t clearBitsAt(uint32_t reg, uint8_t pos, uint8_t width)
{
    if (width == 0U || pos >= 32U || (pos + width) > 32U)
        return reg;

    uint32_t mask = (width == 32U) ? 0xFFFFFFFFU : (((1U << width) - 1U) << pos);
    return reg & ~mask;
}
```


21. Unpack BCD (Binary Coded Decimal) from register.
    - Counter: Why BCD used in real-time clocks?
    - Counter: Little-endian BCD — how to handle?
    - Counter: Validate BCD: check digit ranges?

```cpp
#include <cstdint>

/*
### Counter 1: Why is BCD used in real-time clocks?
BCD is useful because each decimal digit is stored as a nibble (4 bits).
This makes it very easy to display time and date on 7-segment displays and LCDs.

Example:
    // RTC register stores 23:59:59 in BCD
    // 0x23 means 23 decimal, 0x59 means 59 decimal
    uint8_t rtc_min = 0x59; // 59 minutes

### Counter 2: Little-endian BCD — how to handle it?
If the register is little-endian, the lower nibble is the least significant digit.
So a value like 0x47 means:
    tens = 4
    ones = 7
    decimal = 47

### Counter 3: Validate BCD digit ranges?
Each nibble must be between 0 and 9.
If a nibble is greater than 9, the data is invalid.

Example:
    uint8_t value = 0x1A; // invalid: lower nibble A = 10
    // must reject or handle as error
*/

#include <stdint.h>

static inline uint8_t unpack_bcd_byte(uint8_t reg)
{
    uint8_t tens = (reg >> 4) & 0x0F;
    uint8_t ones = reg & 0x0F;

    if (tens > 9U || ones > 9U)
        return 0U; // invalid BCD: return safe value

    return (uint8_t)((tens * 10U) + ones);
}
```
22. Pack decimal digits into BCD register.
    - Counter: Validate input before packing?
    - Counter: Handle negative numbers?
    - Counter: Optimize for multiple digit packing?


```c++
#include <cstdint>

/*
### Counter 1: Validate input before packing?
Yes. For BCD, every digit must be 0..9.
If invalid digits are passed, the output may become corrupted.

Example:
    // 42 -> BCD = 0x42
    // 4 in upper nibble, 2 in lower nibble

### Counter 2: Handle negative numbers?
For signed values, use a sign bit or separate sign handling.
Example:
    -12  -> sign = negative, magnitude = 12
    BCD of 12 = 0x12
    final value = sign marker + 0x12

### Counter 3: Optimize for multiple digit packing?
Use bit shifts and nibble masks.
This avoids expensive decimal conversion and is efficient on microcontrollers.
*/

static inline uint8_t pack_decimal_digit(uint8_t digit)
{
    if (digit > 9U)
        return 0U; // invalid digit, safe default
    return digit;
}

static inline uint8_t pack_bcd_byte(uint8_t value)
{
    uint8_t tens = value / 10U;
    uint8_t ones = value % 10U;

    return (uint8_t)((tens << 4) | ones);
}

/*
Example:
    uint8_t packed = pack_bcd_byte(42); // packed = 0x42
    // 42 decimal -> 0100 0010 in BCD
*/

```
23. Bit-banging protocol: transmit byte serially.
    - Counter: LSB or MSB first?
    - Counter: Timing constraints for specific baud rate?
    - Counter: Interrupt vs polling driven?

```c++
#include <cstdint>

/*
### Counter 1: LSB or MSB first?
Depends on the protocol.
UART usually sends LSB first, but some protocols send MSB first.
You must match the protocol exactly.

Example:
    byte = 0b11000010
    LSB-first transmission:
        0, 1, 0, 0, 0, 0, 1, 1
    MSB-first transmission:
        1, 1, 0, 0, 0, 0, 1, 0

### Counter 2: Timing constraints for specific baud rate?
Bit-banging must produce exact delays.
Example:
    baud = 9600
    bit period = 1 / 9600 ≈ 104 us
    Each bit must be held for ~104 us.

### Counter 3: Interrupt vs polling?
Polling is simple but blocks CPU.
Interrupt-driven bit-banging is better for responsiveness.
But hardware UART is preferred when available.
*/

static inline void tx_pin(uint8_t bit)
{
    // Example only: write to GPIO output
    // GPIOA->ODR = (bit != 0U) ? 1U : 0U;
}

static inline void bitbang_tx_byte(uint8_t byte, uint8_t lsb_first)
{
    // Start bit
    tx_pin(0U);

    if (lsb_first)
    {
        for (uint8_t i = 0U; i < 8U; ++i)
        {
            tx_pin((byte >> i) & 1U);
        }
    }
    else
    {
        for (int8_t i = 7; i >= 0; --i)
        {
            tx_pin((byte >> i) & 1U);
        }
    }

    // Stop bit
    tx_pin(1U);
}

/*
Example:
    uint8_t data = 0x5A;
    bitbang_tx_byte(data, 1);  // LSB-first
*/
```

24. Implement a circular bit buffer (1 bit per element).
    - Counter: Memory savings vs access overhead?
    - Counter: When is this practical?
    - Counter: Atomic read/write safety?

```c++
#include <cstdint>

/*
### Counter 1: Memory savings vs access overhead?
This uses 1 bit per slot, so it is very memory-efficient.
But reading/writing is slower because you must manipulate bit masks.

Example:
    64-bit bit buffer can store 64 flags in 8 bytes.
    A byte array would cost 64 bytes.

### Counter 2: When is it practical?
Good for:
- event flags
- GPIO state history
- status tracking
- compressed bitset storage

### Counter 3: Atomic read/write safety?
If accessed from multiple contexts (ISR and main), use:
- atomic operations
- critical sections
- or disable interrupts while reading/writing
*/

#define BITBUF_SIZE 64U

typedef struct
{
    uint8_t data[(BITBUF_SIZE + 7U) / 8U];
    uint16_t head;
    uint16_t tail;
    uint16_t size;
} BitBuffer;

static inline bool bitbuf_empty(const BitBuffer* b)
{
    return b->head == b->tail;
}

static inline void bitbuf_write(BitBuffer* b, bool value)
{
    uint16_t index = b->tail;
    uint16_t byte_index = index >> 3U;
    uint16_t bit_index = index & 7U;

    if (value)
        b->data[byte_index] |= (uint8_t)(1U << bit_index);
    else
        b->data[byte_index] &= (uint8_t)~(1U << bit_index);

    b->tail = (b->tail + 1U) % b->size;
}

static inline bool bitbuf_read(BitBuffer* b)
{
    if (bitbuf_empty(b))
        return false;

    uint16_t index = b->head;
    uint16_t byte_index = index >> 3U;
    uint16_t bit_index = index & 7U;

    bool value = ((b->data[byte_index] >> bit_index) & 1U) != 0U;
    b->head = (b->head + 1U) % b->size;
    return value;
}
```

25. Detect transition: check if bit changed from last read.
    - Counter: Edge detection: rising vs falling?
    - Counter: Debounce consideration for GPIO?
    - Counter: Multiple bit transition detection simultaneously?

```c++
#include <cstdint>

/*
### Counter 1: Rising vs falling edge?
Rising edge = old bit was 0, new bit is 1.
Falling edge = old bit was 1, new bit is 0.

Example:
    previous = 0 -> current = 1  => rising edge
    previous = 1 -> current = 0  => falling edge

### Counter 2: Debounce consideration for GPIO?
Mechanical buttons bounce.
So you usually wait 10-50 ms before accepting the transition.
This prevents false triggers.

Example:
    if (current_bit != last_bit)
    {
        // wait debounce time
        // then accept edge
    }

### Counter 3: Multiple bit transition detection?
Use XOR:
    changed = previous_reg ^ current_reg;
This marks exactly which bits changed.

Example:
    prev = 0b00110100
    curr = 0b00100110
    changed = 0b00010010
*/

static inline bool rising_edge(uint8_t prev, uint8_t curr)
{
    return (prev == 0U) && (curr == 1U);
}

static inline bool falling_edge(uint8_t prev, uint8_t curr)
{
    return (prev == 1U) && (curr == 0U);
}

static inline uint32_t detect_changed_bits(uint32_t prev_reg, uint32_t curr_reg)
{
    return prev_reg ^ curr_reg;
}

/*
Example:
    uint32_t prev = 0b00110100;
    uint32_t curr = 0b00100110;
    uint32_t changed = detect_changed_bits(prev, curr);
    // changed bits are where transitions occurred
*/
```
---------

### ADVANCED

26. Two's complement arithmetic without CPU support.
    - Counter: How to simulate subtraction using addition?
    - Counter: Overflow detection?
    - Counter: When needed on low-end MCUs?


```c++
/* 
   Problem:
   How can we perform:

       A - B

   using addition only?

   Two's complement gives:

       A - B = A + (~B + 1)

   Because:

       -B = ~B + 1

   Example:

       5 - 3

       Binary:
           5 = 00000101
           3 = 00000011

       NOT 3:
           11111100

       +1:
           11111101

       Add:

           00000101
         + 11111101
         ----------
           00000010

       Result = 2

   This is the mathematical basis used by modern binary
   arithmetic hardware.

   On most modern CPUs, subtraction is already directly
   supported, so this software emulation is mainly useful
   for understanding arithmetic or unusual restricted
   environments.
*/

/*
   API/FUNCTION: subtract_using_addition()

   It performs subtraction using:

       a + (~b + 1)

   uint8_t is used here to clearly demonstrate 8-bit arithmetic.

   Important:
   The C integer promotion rules mean calculations involving
   uint8_t may actually be performed as int.

   Therefore, explicit casts are useful when we want
   deliberate 8-bit behavior.
*/
static uint8_t subtract_using_addition(uint8_t a, uint8_t b)
{
    uint8_t negative_b;
    uint8_t result;

    /*
       Step 1:
       Invert all bits of b.

       Example:
           b = 00000011
           ~b = 11111100
    */
    negative_b = (uint8_t)(~b);

    /*
       Step 2:
       Add 1.

       ~b + 1 = two's complement representation of -b.
    */
    negative_b = (uint8_t)(negative_b + 1U);

    /*
       Step 3:
       Add a and -b.

           a - b
             =
           a + (-b)
    */
    result = (uint8_t)(a + negative_b);

    return result;
}

/*
   OVERFLOW DETECTION:

   Unsigned overflow is easy to detect.

   Example with uint8_t:

       250 + 10 = 260

   But uint8_t can represent only:

       0 ... 255

   Therefore result wraps around:

       260 mod 256 = 4

   One way to detect overflow:

       result < original operand

   Another robust method is to perform the operation
   in a wider integer type and compare with the limit.
*/

/*
   API/FUNCTION: add_u8_with_overflow()

   Returns:
       1 -> overflow occurred
       0 -> no overflow

   result points to the location where the final 8-bit
   result will be written.
*/
static int add_u8_with_overflow(uint8_t a,
                                uint8_t b,
                                uint8_t *result)
{
    uint16_t temporary;

    /*
       uint16_t is wide enough to represent:

           0 ... 510

       which covers every possible uint8_t + uint8_t result.
    */
    temporary = (uint16_t)a + (uint16_t)b;

    /*
       Keep only the lower 8 bits.
    */
    *result = (uint8_t)temporary;

    /*
       If temporary is greater than 255,
       the uint8_t result could not contain the full value.
    */
    return (temporary > 255U);
}
```
27. Saturating arithmetic: prevent overflow/underflow.
    - Counter: Signed vs unsigned saturation?
    - Counter: Is there a CPU instruction (SSAT/USAT)?
    - Counter: Performance cost vs brute force?

```c++
/*
Normal arithmetic wraps around.

   Example:

       uint8_t x = 250;
       x = x + 10;

   Normal result:

       4

   This may be dangerous in control systems.

   Saturating arithmetic instead gives:

       255

   because the value is "clamped" to the maximum.

   Similarly:

       5 - 10

   normally wraps for unsigned arithmetic.

   Saturation gives:

       0

   Common embedded applications:
   - Digital signal processing
   - Audio
   - Motor control
   - Sensor processing
   - Fixed-point calculations
   - Image processing

   CPU support:

   Some ARM processors provide dedicated saturating
   instructions, such as SSAT and USAT on applicable
   architectures/instruction sets.

   Compiler intrinsics or DSP libraries may expose them.

   Software implementation is more portable.
*/

/*
   API/FUNCTION: saturating_add_u8()

   Maximum possible value of uint8_t = 255.

   Algorithm:
       1. Add using a wider type.
       2. If >255, return 255.
       3. Otherwise return calculated value.
*/
static uint8_t saturating_add_u8(uint8_t a, uint8_t b)
{
    uint16_t result;

    result = (uint16_t)a + (uint16_t)b;

    if (result > 255U)
    {
        return UINT8_MAX;
    }

    return (uint8_t)result;
}

/*
   API/FUNCTION: saturating_sub_u8()

   Algorithm:
       If b > a:
           result would become negative.

       Since uint8_t cannot represent negative values,
       clamp to 0.

       Otherwise perform normal subtraction.
*/
static uint8_t saturating_sub_u8(uint8_t a, uint8_t b)
{
    if (b > a)
    {
        return 0U;
    }

    return (uint8_t)(a - b);
}

/*
   SIGNED SATURATION:

   int8_t range:

       -128 ... +127

   Example:

       120 + 20 = 140

   But 140 cannot fit in int8_t.

   Saturating result:

       +127
*/
static int8_t saturating_add_s8(int8_t a, int8_t b)
{
    int16_t result;

    result = (int16_t)a + (int16_t)b;

    if (result > INT8_MAX)
    {
        return INT8_MAX;
    }

    if (result < INT8_MIN)
    {
        return INT8_MIN;
    }

    return (int8_t)result;
}
```

28. Implement barrel shifter functionality in software.
    - Counter: Any variable shift count?
    - Counter: Performance vs dedicated hardware?
    - Counter: Used in protocol bit alignment?

```c++

/* 
   Barrel shifter:

   A barrel shifter can shift or rotate a word by an arbitrary
   number of positions in one hardware operation.

   Example:

       value = 0x12345678

       shift left by 4:

       0x23456780

   In C, we can use:

       value << count
       value >> count

   Rotation is different from ordinary shifting.

   Rotation moves the bits that leave one end back into
   the other end.

   Example 8-bit:

       10010001

   Rotate left by 1:

       00100011

   Important embedded use:
   - Protocol bit alignment
   - Cryptography
   - CRC operations
   - Hash functions
   - Register manipulation
   - Bitstream parsing
*/

/*
   API/FUNCTION: rotate_left_32()

   This implements a 32-bit left rotation.

   Formula:

       (value << shift) |
       (value >> (32 - shift))

   Why OR?

   The first shift moves bits left.

   The second shift takes the bits that would otherwise
   be lost and moves them to the lower positions.

   IMPORTANT:
   Shifting by exactly 32 is undefined in C for uint32_t.

   Therefore:

       shift %= 32

   and special handling for shift == 0 are used.
*/
static uint32_t rotate_left_32(uint32_t value, uint32_t shift)
{
    shift %= 32U;

    if (shift == 0U)
    {
        return value;
    }

    return (value << shift) | (value >> (32U - shift));
}

/*
   API/FUNCTION: rotate_right_32()

   Same concept, but the rotation direction is reversed.
*/
static uint32_t rotate_right_32(uint32_t value, uint32_t shift)
{
    shift %= 32U;

    if (shift == 0U)
    {
        return value;
    }

    return (value >> shift) | (value << (32U - shift));
}

/*
   Hardware vs software:

   If the CPU has a rotate instruction, compiler optimizers may
   recognize these C expressions and generate that instruction.

   Therefore, writing portable C does not automatically mean
   poor performance.

   Always inspect generated assembly when performance matters.
*/
```



29. Extract nibbles (4-bit) from register and process.
    - Counter: Why work with nibbles instead of bytes?
    - Counter: Common in BCD and data encoding?
    - Counter: Iterate or unroll loop?

```c++
#include <cstdint>

/*
====================================================
Question 29
Extract and process nibbles from a register
====================================================

Counter Answer:
1) A nibble represents one hexadecimal digit and packs two small values into
   one byte. This is useful when the hardware register or protocol defines
   fields that are four bits wide.
2) BCD stores one decimal digit per nibble. Nibbles are also common in packed
   status fields, hexadecimal text, and compact device protocols.
3) Iterate when register width is variable or code size matters. Unroll a
   fixed, short loop only after measuring a timing benefit on the target.
*/

static inline uint8_t extractNibble(uint32_t reg, uint8_t index)
{
    if (index >= 8U)
        return 0U;

    return static_cast<uint8_t>((reg >> (index * 4U)) & 0x0FU);
}

// Example processing: return the sum of all eight nibbles in a 32-bit register.
static inline uint8_t sumNibbles32(uint32_t reg)
{
    uint8_t sum = 0U;

    for (uint8_t index = 0U; index < 8U; ++index)
        sum = static_cast<uint8_t>(sum + extractNibble(reg, index));

    return sum;
}
```

30. Bit reversal lookup table vs algorithmic reversal.
    - Counter: Memory usage: 256 vs 65536 entry table?
    - Counter: Speed difference on target hardware?
    - Counter: Which is better for embedded?

```c++
#include <cstdint>

/*
====================================================
Question 30
Bit reversal: lookup table versus algorithm
====================================================

Counter Answer:
1) A 256-entry byte-reversal table occupies 256 bytes when its entries are
   uint8_t. A 65536-entry 16-bit reversal table needs at least 128 KiB, so it
   is rarely appropriate for a small MCU.
2) A table replaces shifts and masks with memory reads. It can be faster from
   fast flash/cache, but slow flash or cache misses can make the algorithm
   competitive. Benchmark on the actual target.
3) Prefer a CPU instruction/intrinsic when available. Otherwise, a shift-mask
   algorithm is a good default; use a 256-entry flash table for a measured
   hot path when flash is available.
*/

static inline uint8_t reverseBits8(uint8_t value)
{
    value = static_cast<uint8_t>(((value & 0x55U) << 1U) | ((value >> 1U) & 0x55U));
    value = static_cast<uint8_t>(((value & 0x33U) << 2U) | ((value >> 2U) & 0x33U));
    return static_cast<uint8_t>((value << 4U) | (value >> 4U));
}

// A 256-entry const uint8_t table can replace reverseBits8() in a hot path.
static inline uint32_t reverseBits32Algorithmic(uint32_t value)
{
    value = ((value & 0x55555555U) << 1U) | ((value >> 1U) & 0x55555555U);
    value = ((value & 0x33333333U) << 2U) | ((value >> 2U) & 0x33333333U);
    value = ((value & 0x0F0F0F0FU) << 4U) | ((value >> 4U) & 0x0F0F0F0FU);
    value = ((value & 0x00FF00FFU) << 8U) | ((value >> 8U) & 0x00FF00FFU);
    return (value << 16U) | (value >> 16U);
}
```

31. CRC bit-by-bit calculation using XOR.
    - Counter: Polynomial selection?
    - Counter: Lookup table optimization (pre-computed CRC)?
    - Counter: Real-time: must fit in ISR time budget?

```c++
#include <cstddef>
#include <cstdint>

/*
====================================================
Question 31
CRC-8 bit-by-bit calculation using XOR
====================================================

This is the non-reflected CRC-8/ATM form: poly=0x07, init=0x00, xorout=0x00.

Counter Answer:
1) Select the complete CRC definition required by the protocol: width,
   polynomial, initial value, input/output reflection, and final XOR. Using
   only the right polynomial is not sufficient for interoperability.
2) A 256-entry table reduces the work from eight bit iterations per byte to
   one lookup and XOR. It trades flash for speed; hardware CRC may be best.
3) Include worst-case message length in the ISR time budget. If it does not
   fit, process incrementally outside the ISR, use DMA/hardware CRC, or use a
   table after measuring its execution time on the target.
*/

static inline uint8_t crc8Atm(const uint8_t *data, size_t length)
{
    uint8_t crc = 0x00U;

    while (length-- != 0U)
    {
        crc ^= *data++;

        for (uint8_t bit = 0U; bit < 8U; ++bit)
        {
            crc = (crc & 0x80U) != 0U
                ? static_cast<uint8_t>((crc << 1U) ^ 0x07U)
                : static_cast<uint8_t>(crc << 1U);
        }
    }

    return crc;
}
```

32. Implement endianness conversion functions.
    - Counter: When necessary: network byte order?
    - Counter: __bswap32() availability and portability?
    - Counter: Multi-byte structure endianness handling?

```c++
#include <stdint.h>

/*
====================================================
Question 32
Implement endianness conversion functions.

Counter:
1. When necessary: network byte order?
2. __bswap32() availability and portability?
3. Multi-byte structure endianness handling?

Answers:
1. Network byte order is traditionally big-endian.
   Convert explicitly when exchanging multi-byte values
   between systems or protocols.

2. __bswap32() is compiler/platform dependent.
   A manual implementation is more portable in embedded C.

3. Do not directly transmit/copy a C structure when its
   multi-byte fields have protocol-defined endianness.
   Serialize each field explicitly.
*/

static uint16_t swap16(uint16_t value)
{
    return (uint16_t)((value >> 8U) |
                      (value << 8U));
}

static uint32_t swap32(uint32_t value)
{
    return ((value >> 24U) & 0x000000FFUL) |
           ((value >> 8U)  & 0x0000FF00UL) |
           ((value << 8U)  & 0x00FF0000UL) |
           ((value << 24U) & 0xFF000000UL);
}

/* Convert host value to big-endian representation */
static uint32_t host_to_be32(uint32_t value)
{
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
    return swap32(value);
#else
    return value;
#endif
}

/* Convert big-endian representation to host value */
static uint32_t be32_to_host(uint32_t value)
{
    return host_to_be32(value);
}
```

33. Generate test patterns: alternating bits (0xAAAA, 0x5555).
    - Counter: Why used for hardware testing?
    - Counter: Walking ones/zeros pattern?
    - Counter: Marching bit pattern for memory test?

```c++
#include <stdint.h>

/*
====================================================
Question 33
Generate test patterns: alternating bits (0xAAAA, 0x5555).

Counter:
1. Why used for hardware testing?
2. Walking ones/zeros pattern?
3. Marching bit pattern for memory test?

Answers:
1. 0xAAAA and 0x5555 exercise alternating 1/0 data lines
   and can help detect certain data-line and coupling faults.

2. Walking-1 tests one bit as 1 while other bits are 0.
   Walking-0 does the opposite. They help detect stuck-at
   and data-line faults.

3. March tests apply sequences of write/read operations while
   moving through memory addresses. They are commonly used
   to detect memory faults such as stuck-at and transition faults.
*/

static uint16_t generate_alternating_aaaa(void)
{
    return 0xAAAAU;
}

static uint16_t generate_alternating_5555(void)
{
    return 0x5555U;
}

static uint32_t walking_one(uint8_t position)
{
    return (uint32_t)1U << position;
}

static uint32_t walking_zero(uint8_t position)
{
    return ~((uint32_t)1U << position);
}
```

34. Bit-packing compression: store multiple small values in one register.
    - Counter: What if values have different bit widths?
    - Counter: Alignment and padding overhead?
    - Counter: Endianness considerations?


```c++
#include <stdint.h>

/*
====================================================
Question 34
Bit-packing compression: store multiple small values in one register.

Counter:
1. What if values have different bit widths?
2. Alignment and padding overhead?
3. Endianness considerations?

Answers:
1. Assign each value a fixed number of bits and shift it
   to its defined position. Different fields can have
   different widths.

2. Bit-packed data can reduce storage, but the containing
   object may still have alignment requirements. Protocol
   serialization should be handled explicitly.

3. Endianness matters when the packed value is transmitted
   as multiple bytes. Define the byte order in the protocol
   and serialize explicitly.
*/

/*
 * Pack:
 *   value_a = 5 bits
 *   value_b = 3 bits
 *   value_c = 8 bits
 *
 * Total = 16 bits
 */
static uint16_t pack_values(uint8_t value_a,
                            uint8_t value_b,
                            uint8_t value_c)
{
    uint16_t packed = 0U;

    packed |= (uint16_t)(value_a & 0x1FU);
    packed |= (uint16_t)(value_b & 0x07U) << 5U;
    packed |= (uint16_t)value_c << 8U;

    return packed;
}

static void unpack_values(uint16_t packed,
                          uint8_t *value_a,
                          uint8_t *value_b,
                          uint8_t *value_c)
{
    *value_a = (uint8_t)(packed & 0x1FU);
    *value_b = (uint8_t)((packed >> 5U) & 0x07U);
    *value_c = (uint8_t)((packed >> 8U) & 0xFFU);
}
```

#### 35. Find Hamming distance (bit difference) between two values.
    - Counter: XOR then popcount?
    - Counter: Application: error detection?
    - Counter: Used in some cyclic codes?

```c++
#include <stdint.h>

/*
===========================================================
Question 35:
Find Hamming Distance (bit difference) between two values
===========================================================

WHAT IS HAMMING DISTANCE?
-------------------------

Hamming distance between two binary values is the number of
bit positions where the two values are different.

Example:

    a = 0b10110100
    b = 0b10011100

Compare bit by bit:

    a = 1 0 1 1 0 1 0 0
    b = 1 0 0 1 1 1 0 0
        -----------------
        0 0 1 0 1 0 0 0

There are TWO different bit positions.

Therefore:

    Hamming Distance = 2


-----------------------------------------------------------
STEP 1: XOR THE TWO VALUES
-----------------------------------------------------------

XOR has the following truth table:

    A   B   A ^ B
    -------------
    0   0     0
    0   1     1
    1   0     1
    1   1     0

Important observation:

    XOR = 1 -> bits are different
    XOR = 0 -> bits are the same

Therefore, if we XOR two values:

    diff = a ^ b;

the resulting value contains 1s exactly at the positions
where a and b are different.


Example:

    a = 10110100
    b = 10011100

    a ^ b
    -------
      00101000

There are two 1s.

Therefore:

    Hamming Distance = number of 1s in (a ^ b)


-----------------------------------------------------------
STEP 2: COUNT THE NUMBER OF 1s
-----------------------------------------------------------

After XOR:

    diff = a ^ b;

we need to count how many bits are set to 1.

This operation is commonly called:

    POPCOUNT
    Population Count
    Bit Count
    Number of Set Bits

For maximum portability in basic embedded C, we can
implement popcount ourselves.


-----------------------------------------------------------
METHOD 1: SIMPLE BIT-BY-BIT METHOD
-----------------------------------------------------------

For a uint32_t value:

    1. Check the least-significant bit.
    2. If it is 1, increment count.
    3. Shift the value right by one bit.
    4. Repeat until the value becomes zero.

Example:

    diff = 00101000

Iteration:

    00101000 -> LSB = 0 -> count = 0
    00010100 -> LSB = 0 -> count = 0
    00001010 -> LSB = 0 -> count = 0
    00000101 -> LSB = 1 -> count = 1
    00000010 -> LSB = 0 -> count = 1
    00000001 -> LSB = 1 -> count = 2
    00000000 -> stop

Final:

    count = 2


-----------------------------------------------------------
C IMPLEMENTATION
-----------------------------------------------------------
*/

uint32_t hamming_distance(uint32_t a, uint32_t b)
{
    /*
     * XOR identifies the positions where the two values
     * are different.
     *
     * Same bits:
     *
     *     0 ^ 0 = 0
     *     1 ^ 1 = 0
     *
     * Different bits:
     *
     *     0 ^ 1 = 1
     *     1 ^ 0 = 1
     */
    uint32_t diff = a ^ b;

    /*
     * This variable stores the number of different bits.
     */
    uint32_t count = 0;

    /*
     * Process every set bit in 'diff'.
     */
    while (diff != 0U)
    {
        /*
         * Check the least-significant bit.
         *
         * diff & 1U:
         *
         *     0 -> LSB is 0
         *     1 -> LSB is 1
         */
        count += diff & 1U;

        /*
         * Shift right by one position.
         *
         * This moves the next bit into the LSB position.
         */
        diff >>= 1U;
    }

    /*
     * Number of 1s in (a ^ b) is exactly the Hamming
     * distance between a and b.
     */
    return count;
}


/*
===========================================================
Example
===========================================================
*/

void example(void)
{
    uint32_t a = 0xB4U;
    uint32_t b = 0x9CU;

    /*
     * Binary representation:

         a = 0xB4 = 1011 0100
         b = 0x9C = 1001 1100

         XOR
             1011 0100
           ^ 1001 1100
           ------------
             0010 1000

         0010 1000 contains two 1s.

         Therefore:

             Hamming distance = 2
    */

    uint32_t distance = hamming_distance(a, b);

    /*
     * distance = 2
     */
    (void)distance;
}


/*
===========================================================
METHOD 2: BRIAN KERNIGHAN'S ALGORITHM
===========================================================

The previous implementation checks every bit.

For a 32-bit integer, it can require up to 32 iterations.

There is a more efficient technique:

    diff = diff & (diff - 1);

This operation removes the LOWEST SET BIT from diff.

Example:

    diff     = 00101000
    diff - 1 = 00100111

    00101000
  & 00100111
  ----------
    00100000

One '1' has been removed.

Therefore, every iteration corresponds to ONE SET BIT.

If there are only a few set bits, this can be much faster.
*/

uint32_t hamming_distance_fast(uint32_t a, uint32_t b)
{
    /*
     * XOR gives us the bits that are different.
     */
    uint32_t diff = a ^ b;

    uint32_t count = 0;

    /*
     * Each iteration removes one set bit.
     *
     * Therefore, the number of iterations equals the number
     * of set bits.
     */
    while (diff != 0U)
    {
        /*
         * Remove the lowest set bit.
         *
         * Example:
         *
         *     diff     = 10110000
         *     diff - 1 = 10101111
         *
         *     10110000
         *   & 10101111
         *   -----------
         *     10100000
         *
         * One set bit has disappeared.
         */
        diff &= (diff - 1U);

        count++;
    }

    return count;
}


/*
===========================================================
METHOD 3: COMPILER / CPU POPCOUNT
===========================================================

Many processors provide a hardware instruction for counting
set bits.

Depending on the compiler and target architecture, we may
have a built-in function such as:

    __builtin_popcount()

Example:

    uint32_t distance = __builtin_popcount(a ^ b);

This is extremely concise.

However, compiler-specific built-ins are not always suitable
when writing highly portable embedded C.

The exact generated instruction should also be checked when
performance is important.

For example, on a target CPU that supports a hardware
population-count instruction, the compiler may generate a
single or small number of instructions.

On another target, it may generate software code.


===========================================================
IMPORTANT INTERVIEW ANSWER
===========================================================

Question:

    How do you find the Hamming distance between two values?

Good answer:

    "I XOR the two values because XOR produces 1 at every
     bit position where the values differ. Then I count the
     number of set bits in the XOR result using popcount."


In short:

    Hamming Distance(a, b) = POPCOUNT(a ^ b)


===========================================================
SPECIAL CASES
===========================================================

CASE 1: a == b

Example:

    a = 10101010
    b = 10101010

    a ^ b = 00000000

    popcount = 0

Therefore:

    Hamming distance = 0

No bits are different.


CASE 2: Completely different bits

For an 8-bit value:

    a = 00000000
    b = 11111111

    a ^ b = 11111111

    popcount = 8

Therefore:

    Hamming distance = 8


CASE 3: One bit changed

    a = 10110100
    b = 10100100

    a ^ b = 00010000

Only one bit is different.

Therefore:

    Hamming distance = 1


===========================================================
APPLICATION: ERROR DETECTION
===========================================================

Hamming distance is an important concept in error-detecting
and error-correcting codes.

Suppose a communication system sends:

    101100

During transmission, one bit changes:

    101000

The receiver can compare the received data with another
known/reference value and calculate the Hamming distance.

Here:

    101100
    101000
    ------
    000100

Hamming distance = 1

This tells us that one bit position differs.

IMPORTANT:

Hamming distance by itself does NOT automatically mean that
we can detect or correct an error.

The coding scheme must be designed with a sufficient minimum
Hamming distance.

For a code with minimum Hamming distance 'd':

    It can detect up to:

        d - 1

    bit errors.

And it can correct up to:

        floor((d - 1) / 2)

    bit errors.

Example:

    Minimum distance = 3

    Error detection:
        3 - 1 = 2 errors

    Error correction:
        floor((3 - 1) / 2) = 1 error


===========================================================
IS IT USED IN CYCLIC CODES?
===========================================================

Yes, Hamming distance is relevant to coding theory and is
used when analyzing error-detecting and error-correcting
codes, including cyclic codes.

But be careful with the wording in an interview.

Hamming distance is NOT a special operation belonging only
to cyclic codes.

It is a general concept in coding theory.

For example, it can be used when discussing:

    - Hamming codes
    - Cyclic codes
    - Linear block codes
    - Error-detecting codes
    - Error-correcting codes

For cyclic codes, the minimum Hamming distance helps
determine how many errors the code can detect or correct.


===========================================================
EMBEDDED SYSTEM EXAMPLE
===========================================================

Imagine an embedded system receives a 32-bit status word:

    received_status

and wants to compare it with a previous status:

    previous_status

We can calculate:

    changed_bits = received_status ^ previous_status;

Then:

    number_of_changed_bits =
        popcount(changed_bits);

This can tell us how many individual flags/bits changed
between two status words.


Example:

    previous = 0b00001101
    current  = 0b00011101

    previous ^ current
             = 00010000

Only one bit changed.

Therefore:

    Hamming distance = 1


===========================================================
TIME COMPLEXITY
===========================================================

Simple bit-by-bit implementation:

    while (diff != 0)
        diff >>= 1;

Worst case:

    O(N)

where N is the number of bits.

For uint32_t:

    maximum = 32 iterations


Brian Kernighan's algorithm:

    while (diff != 0)
        diff &= diff - 1;

Complexity:

    O(K)

where K = number of set bits.

For example:

    diff = 00000000000000000000000000010001

Only two bits are set.

K = 2

So the loop executes only twice instead of potentially
checking all 32 bit positions.


===========================================================
FINAL INTERVIEW SUMMARY
===========================================================

The cleanest answer is:

    uint32_t hamming_distance(uint32_t a, uint32_t b)
    {
        uint32_t diff = a ^ b;
        uint32_t count = 0;

        while (diff != 0U)
        {
            diff &= (diff - 1U);
            count++;
        }

        return count;
    }


Remember this one line:

    Hamming Distance = POPCOUNT(A XOR B)


Why XOR?
    -> Finds different bits.

Why POPCOUNT?
    -> Counts those different bits.

Where useful?
    -> Comparing bit patterns, detecting changes, and
       analyzing error-detecting/error-correcting codes.

Key coding-theory point:
    -> Minimum Hamming distance determines the error
       detection/correction capability of a code.
*/
```

---

### EXPERT

#### 36. Implement Parallel Bit Count (SWAR — SIMD Within A Register).
    - Counter: Process multiple bit counts in parallel?
    - Counter: Algorithm uses shifts and masks cleverly?
    - Counter: When is this worth the complexity?

```c++
#include <stdint.h>

/*
===========================================================
Question 36
Implement Parallel Bit Count
SWAR = SIMD Within A Register
===========================================================

GOAL
----

We want to count the number of 1-bits in a 32-bit integer.

Example:

    value = 10110010...

A normal approach checks one bit at a time.

SWAR takes a different approach:

    Instead of processing one bit at a time,
    we process GROUPS of bits in parallel inside
    the same 32-bit register.

This is why it is called:

    SWAR
    = SIMD Within A Register


===========================================================
WHAT DOES "PARALLEL" MEAN HERE?
===========================================================

Suppose we have:

    1011 0010 1101 0001

We can think of this 32-bit register as multiple smaller
fields.

SWAR performs operations on these fields simultaneously
using normal integer instructions.

There is no actual SIMD/vector register involved.

We are simply using:

    - bitwise AND
    - shifts
    - subtraction
    - addition

to make multiple groups behave independently.


===========================================================
WHY NOT JUST USE __builtin_popcount()?
===========================================================

In modern C/C++ code, you might simply write:

    __builtin_popcount(value);

This is usually preferable when available.

The compiler knows the target CPU and may generate:

    - hardware POPCOUNT instruction
    - optimized software implementation
    - other target-specific instructions

SWAR is useful when:

    1. Hardware POPCNT is unavailable.
    2. We want a portable branchless algorithm.
    3. We care about predictable execution.
    4. We are working in low-level/embedded code.
    5. We want to understand bit-manipulation techniques.

For ordinary application code, SWAR can be unnecessarily
complicated.


===========================================================
THE ALGORITHM
===========================================================

The complete algorithm is:

    value = value - ((value >> 1U) & 0x55555555UL);

    value = (value & 0x33333333UL) +
            ((value >> 2U) & 0x33333333UL);

    value = (value + (value >> 4U)) & 0x0F0F0F0FUL;

    value = value + (value >> 8U);

    value = value + (value >> 16U);

    return value & 0x3FU;


It looks complicated.

But each line has a very specific purpose.

The algorithm progressively combines bit counts:

    individual bits
          ↓
    2-bit groups
          ↓
    4-bit groups
          ↓
    8-bit groups
          ↓
    16-bit groups
          ↓
    32-bit total


===========================================================
STEP 1
===========================================================

    value = value - ((value >> 1U) & 0x55555555UL);


This is the most confusing line.

The mask:

    0x55555555

in binary is:

    01010101010101010101010101010101


Why?

Because we want to work with pairs of bits:

    [bit1 bit0]
    [bit3 bit2]
    [bit5 bit4]
    ...


The mask selects the lower bit of every 2-bit group.


Example:

    0x55555555

    0101 0101 0101 0101
    0101 0101 0101 0101


===========================================================
WHY SHIFT RIGHT BY 1?
===========================================================

Consider one 2-bit group:

    00
    01
    10
    11

The number of 1s in each group is:

    00 -> 0
    01 -> 1
    10 -> 1
    11 -> 2


After:

    value >> 1

the upper bit of each pair moves into the lower position.

Then:

    & 0x55555555

keeps only that relevant bit.

The subtraction effectively converts each 2-bit group into
the number of 1s contained in that group.

So after STEP 1:

    Every 2-bit field contains its own population count.

Conceptually:

    Original:

        xx xx xx xx xx xx...

    After STEP 1:

        00 01 10 01 00 10...

    Each 2-bit field now represents a value from:

        0 to 2


===========================================================
STEP 2
===========================================================

    value = (value & 0x33333333UL) +
            ((value >> 2U) & 0x33333333UL);


The mask:

    0x33333333

in binary:

    00110011001100110011001100110011


Now we are working with:

    4-bit groups


The previous step produced counts inside 2-bit groups.

Now we combine two adjacent 2-bit counts.

For example:

    [count][count]

becomes:

    [total count]

A 4-bit group can contain up to four 1s.

Therefore its count can represent:

    0 to 4


After STEP 2:

    Every 4-bit field contains the number of 1s
    in those four original bits.


Conceptually:

    Before:

        [2-bit count][2-bit count]

    After:

        [4-bit total]


===========================================================
STEP 3
===========================================================

    value = (value + (value >> 4U)) & 0x0F0F0F0FUL;


Mask:

    0x0F0F0F0F

Binary:

    00001111 00001111 00001111 00001111


Now we combine:

    4-bit groups

into:

    8-bit groups


Each 8-bit group can contain:

    0 to 8 ones

which fits inside 4 bits.


For example:

    10110101

contains:

    6 ones

After the previous operations, the corresponding 8-bit
field will eventually contain:

    00000110

which represents decimal 6.


After STEP 3:

    Each byte contains the number of 1s in that byte.


So conceptually:

    [byte count] [byte count] [byte count] [byte count]

Each byte contains a value:

    0 to 8


===========================================================
STEP 4
===========================================================

    value = value + (value >> 8U);


Now we want to combine:

    8-bit counts

into:

    16-bit counts.


Suppose we have:

    [count byte 3]
    [count byte 2]
    [count byte 1]
    [count byte 0]

After shifting by 8:

    [   0   ]
    [count byte 3]
    [count byte 2]
    [count byte 1]


Adding them allows the lower fields to accumulate
the counts from neighboring bytes.


After this step, the lower 16-bit area contains
the count of the corresponding 16-bit portion.


===========================================================
STEP 5
===========================================================

    value = value + (value >> 16U);


Same idea again.

Now we combine:

    16-bit counts

into:

    32-bit total count.


After this operation, the lower portion of the register
contains the total number of set bits in the original
32-bit value.


For a uint32_t, the maximum number of 1s is:

    32

So the final answer only needs 6 bits:

    32 decimal = 100000 binary

Therefore 6 bits are enough to represent 0..32.


===========================================================
STEP 6
===========================================================

    return value & 0x3FU;


Why:

    0x3F ?

Because:

    0x3F = binary 00111111

which keeps 6 bits.

A 32-bit value can contain at most:

    32 set bits

and:

    32 = 100000₂

which requires 6 bits.


Therefore:

    value & 0x3F

extracts the final population count.


===========================================================
COMPLETE FUNCTION
===========================================================
*/

static uint32_t popcount_swar32(uint32_t value)
{
    /*
     * STEP 1
     *
     * Count bits inside every 2-bit group.
     *
     * 0x55555555 =
     *
     *     01010101010101010101010101010101
     *
     * This isolates alternating bits.
     */
    value = value - ((value >> 1U) & 0x55555555UL);

    /*
     * STEP 2
     *
     * Combine two 2-bit counts into one 4-bit count.
     *
     * 0x33333333 =
     *
     *     00110011001100110011001100110011
     *
     * Each 4-bit field now contains the number of 1s
     * in the corresponding original 4 bits.
     */
    value = (value & 0x33333333UL) +
            ((value >> 2U) & 0x33333333UL);

    /*
     * STEP 3
     *
     * Combine 4-bit counts into 8-bit counts.
     *
     * 0x0F0F0F0F =
     *
     *     00001111 00001111 00001111 00001111
     *
     * Each byte now contains the number of 1s in
     * that original byte.
     */
    value = (value + (value >> 4U)) & 0x0F0F0F0FUL;

    /*
     * STEP 4
     *
     * Combine counts from neighboring bytes.
     *
     * 8-bit groups -> 16-bit groups.
     */
    value = value + (value >> 8U);

    /*
     * STEP 5
     *
     * Combine the two 16-bit groups.
     *
     * 16-bit groups -> final 32-bit count.
     */
    value = value + (value >> 16U);

    /*
     * STEP 6
     *
     * Keep only the lower 6 bits.
     *
     * Maximum popcount for uint32_t = 32.
     *
     * 32 requires 6 bits.
     *
     * 0x3F = 00111111
     */
    return value & 0x3FU;
}


/*
===========================================================
EXAMPLE
===========================================================
*/

void example(void)
{
    uint32_t value = 0xF0F0F0F0U;

    /*
     * Binary:

        F0 = 11110000

        Therefore:

        0xF0F0F0F0

        = 11110000 11110000 11110000 11110000


     * Each byte contains four 1s.
     *
     * There are four bytes:
     *
     *     4 + 4 + 4 + 4 = 16
     *
     * Therefore:
     *
     *     popcount = 16
     */

    uint32_t count = popcount_swar32(value);

    /*
     * count = 16
     */

    (void)count;
}


/*
===========================================================
ANOTHER SIMPLE EXAMPLE
===========================================================

Input:

    value = 0x0000000F

Binary:

    00000000 00000000 00000000 00001111

There are four 1s.

Therefore:

    popcount_swar32(0x0000000F) = 4


-----------------------------------------------------------
ALL BITS SET
-----------------------------------------------------------

Input:

    value = 0xFFFFFFFF

Binary:

    11111111 11111111 11111111 11111111

There are 32 ones.

Therefore:

    popcount_swar32(0xFFFFFFFF) = 32


-----------------------------------------------------------
ZERO
-----------------------------------------------------------

Input:

    value = 0x00000000

There are no 1s.

Therefore:

    popcount_swar32(0) = 0


===========================================================
WHY IS THIS CALLED "SIMD WITHIN A REGISTER"?
===========================================================

Traditional SIMD:

    One instruction
        ↓
    Multiple independent data elements
        ↓
    Vector/SIMD register

SWAR:

    One normal CPU register
        ↓
    Divide it conceptually into fields
        ↓
    Perform operations on multiple fields simultaneously


For example:

    32-bit register

    +--------+--------+--------+--------+
    | 8 bits | 8 bits | 8 bits | 8 bits |
    +--------+--------+--------+--------+

The CPU sees one uint32_t.

Our algorithm treats it as multiple smaller fields.


===========================================================
WHY THE MASKS LOOK "MAGICAL"
===========================================================

The important masks are:

    0x55555555
    0x33333333
    0x0F0F0F0F
    0x0000003F


They correspond to progressively larger groups:

    0x55555555
         ↓
    2-bit grouping

    0x33333333
         ↓
    4-bit grouping

    0x0F0F0F0F
         ↓
    8-bit grouping

    0x0000003F
         ↓
    final 6-bit result


Remember the progression:

       1-bit
         ↓
       2-bit
         ↓
       4-bit
         ↓
       8-bit
         ↓
      16-bit
         ↓
      32-bit


This is the key idea behind the algorithm.


===========================================================
COMPARISON WITH BRIAN KERNIGHAN
===========================================================

Brian Kernighan:

    while (value != 0)
    {
        value &= value - 1;
        count++;
    }


SWAR:

    Fixed sequence of shifts,
    masks and additions.


Kernighan complexity:

    O(number of set bits)


SWAR:

    Fixed number of operations for uint32_t.

Therefore SWAR provides predictable work regardless of
how many bits are set.


===========================================================
WHEN IS SWAR WORTH USING?
===========================================================

SWAR can make sense when:

    - Hardware popcount is unavailable.
    - Performance matters.
    - The operation happens frequently.
    - You want branchless code.
    - You need predictable execution.
    - You are working close to the hardware.


But don't automatically use it everywhere.

For normal C/C++ application code:

    __builtin_popcount(value)

is generally easier to understand.

For embedded code:

    uint32_t popcount_swar32(uint32_t value)

can be useful when:

    - compiler support is limited,
    - target hardware has no POPCNT,
    - timing matters,
    - code size/performance has been measured.


===========================================================
INTERVIEW QUESTION
===========================================================

Interviewer:

    "Explain this code."

Do NOT just say:

    "It's a popcount algorithm."

A stronger answer is:

    "This is a SWAR population-count algorithm. It
     progressively accumulates the number of set bits
     inside 2-bit, 4-bit, 8-bit and larger groups using
     shifts, masks and additions. Because multiple fields
     are processed simultaneously inside one register,
     it avoids checking each bit individually."


===========================================================
IMPORTANT EMBEDDED INTERVIEW POINT
===========================================================

If the interviewer asks:

    "Would you use this in production?"

Don't automatically say YES.

A good answer:

    "It depends on the target. If the compiler provides
     an efficient popcount instruction for the MCU/CPU,
     I would generally use the compiler builtin and verify
     the generated assembly. If hardware popcount isn't
     available and profiling shows popcount is performance
     critical, a SWAR implementation can be appropriate."


That last part is important.

In embedded development, "clever bit manipulation" is not
automatically better.

Measure:

    execution time
    code size
    compiler output
    target architecture
    maintainability


before choosing the implementation.


===========================================================
ONE-LINE MEMORY TRICK
===========================================================

Remember:

    SWAR POPCOUNT

        1 bit
          ↓
        2 bits
          ↓
        4 bits
          ↓
        8 bits
          ↓
        16 bits
          ↓
        32 bits
          ↓
        TOTAL


And the main masks:

    0x55555555 -> 2-bit stage
    0x33333333 -> 4-bit stage
    0x0F0F0F0F -> 8-bit stage
    0x3F       -> final answer


===========================================================
FINAL FORMULA
===========================================================

For a 32-bit value:

    value = value - ((value >> 1) & 0x55555555);

    value = (value & 0x33333333)
          + ((value >> 2) & 0x33333333);

    value = (value + (value >> 4))
          & 0x0F0F0F0F;

    value += value >> 8;
    value += value >> 16;

    result = value & 0x3F;


The important concept is NOT memorizing the magic numbers.

Understand the progression:

    count small groups
          ↓
    combine groups
          ↓
    combine larger groups
          ↓
    obtain total popcount
*/
```

#### 37. De Bruijn sequence for bit position indexing.
    - Counter: Why used for fast bit position lookup?
    - Counter: Generate De Bruijn sequence for powers of 2?
    - Counter: Space-time tradeoff: 32 bytes table vs log operations?

```c++
#include <stdint.h>

/*
===========================================================
Question 37
De Bruijn Sequence for Bit Position Indexing
===========================================================

GOAL
----

Given a 32-bit value containing exactly ONE set bit:

    00000000 00000000 00000000 00001000

we want to find the position of that bit.

Here:

    bit position = 3

because:

    1 << 3 = 0x00000008


The straightforward approach is:

    while ((value >>= 1U) != 0U)
    {
        position++;
    }

But this requires multiple operations.

A De Bruijn sequence provides a clever way to convert
a power-of-two value into a small table index.

The basic idea is:

    isolate lowest set bit
            ↓
    multiply by De Bruijn constant
            ↓
    shift
            ↓
    use result as lookup-table index
            ↓
    obtain bit position


===========================================================
FIRST: WHAT IS A POWER OF TWO BIT?
===========================================================

A value with exactly one bit set is a power of two.

Examples:

    00000001 -> bit 0
    00000010 -> bit 1
    00000100 -> bit 2
    00001000 -> bit 3
    00010000 -> bit 4

Numerically:

    1 << 0 = 1
    1 << 1 = 2
    1 << 2 = 4
    1 << 3 = 8
    1 << 4 = 16


The problem is:

    Given 1, 2, 4, 8, 16, ...
    how can we quickly determine the exponent?

For example:

    8 = 2^3

Answer:

    position = 3


===========================================================
THE DE BRUIJN IDEA
===========================================================

A De Bruijn sequence has a special property:

    Every possible N-bit pattern appears exactly once
    within a particular cyclic sequence.

For the 32-bit bit-position trick, we use a carefully
chosen constant:

    0x077CB531U


This constant has the property that after multiplying
a single set bit by it and extracting selected bits,
we get a unique value for every possible bit position.


===========================================================
WHY MULTIPLICATION?
===========================================================

This is the clever part.

Suppose:

    value = 1 << position

We multiply:

    value * 0x077CB531


Because 'value' contains only one set bit, multiplication
effectively shifts the De Bruijn pattern according to the
position of that bit.

Then:

    result >> 27

extracts 5 bits.

Why 5 bits?

Because:

    2^5 = 32

We have 32 possible bit positions:

    0 ... 31

Therefore we need a table with:

    32 entries.


===========================================================
THE COMPLETE FUNCTION
===========================================================
*/

static uint32_t bit_position_debruijn(uint32_t value)
{
    /*
     * De Bruijn lookup table.
     *
     * There are 32 possible bit positions for uint32_t:
     *
     *     0 through 31
     *
     * Therefore the table has 32 entries.
     *
     * The index generated by the multiplication and shift
     * tells us which entry to use.
     */
    static const uint8_t table[32] =
    {
         0,  1, 28,  2,
        29, 14, 24,  3,
        30, 22, 20, 15,
        25, 17,  4,  8,
        31, 27, 13, 23,
        21, 19, 16,  7,
        26, 12, 18,  6,
        11,  5, 10,  9
    };

    /*
     * IMPORTANT:
     *
     * This algorithm assumes that 'value' contains exactly
     * ONE set bit.
     *
     * Valid examples:
     *
     *     0x00000001
     *     0x00000002
     *     0x00000004
     *     0x00000008
     *     ...
     *     0x80000000
     *
     * If value contains multiple set bits, the result will
     * not represent the position of a single bit correctly.
     */

    /*
     * Multiply the single-bit value by the De Bruijn
     * constant.
     *
     *     0x077CB531
     *
     * The multiplication encodes the original bit position
     * into the upper 5 bits of the result.
     */
    uint32_t index =
        (value * 0x077CB531U) >> 27U;

    /*
     * The resulting 5-bit index is between:
     *
     *     0 and 31
     *
     * Use it to access the lookup table.
     *
     * The table converts the encoded index into the actual
     * bit position.
     */
    return table[index];
}


/*
===========================================================
EXAMPLE 1
===========================================================

Suppose:

    value = 0x00000008

Binary:

    00000000 00000000 00000000 00001000

Only bit 3 is set.

Therefore:

    expected position = 3


Conceptually:

    value
       |
       v
    0x00000008
       |
       | multiply
       v
    value * 0x077CB531
       |
       | >> 27
       v
    lookup index
       |
       v
    table[index]
       |
       v
       3


===========================================================
EXAMPLE 2
===========================================================

    value = 0x00000001

Binary:

    00000000 00000000 00000000 00000001

The set bit is at:

    position 0

The De Bruijn calculation generates an index that points
to:

    table[index] = 0


===========================================================
EXAMPLE 3
===========================================================

    value = 0x00000010

Binary:

    00000000 00000000 00000000 00010000

The set bit is at:

    position 4

The lookup table returns:

    4


===========================================================
EXAMPLE 4
===========================================================

    value = 0x80000000

Binary:

    10000000 00000000 00000000 00000000

The set bit is at:

    position 31

The De Bruijn calculation produces an index that maps to:

    table[index] = 31


===========================================================
WHY IS THIS FAST?
===========================================================

Compare with a simple loop.

LOOP METHOD:

    position = 0;

    while ((value & 1U) == 0U)
    {
        value >>= 1U;
        position++;
    }

If bit 31 is set:

    31 iterations

are required.

The De Bruijn method uses approximately:

    1 multiplication
    1 shift
    1 table lookup

plus the required surrounding operations.

So it provides a fixed-cost method.

This was especially useful on processors where:

    - hardware CLZ/CTZ was unavailable
    - multiplication was reasonably fast
    - predictable execution was desirable


===========================================================
IMPORTANT:
THE TABLE IS NOT MAGIC
===========================================================

The table:

    0, 1, 28, 2, ...

was generated specifically for the chosen De Bruijn
constant:

    0x077CB531

The constant and table MUST MATCH.

You cannot change the constant and keep the same table.

The relationship is:

    De Bruijn constant
            +
    shift amount
            +
    lookup table
            =
    valid bit-position algorithm


===========================================================
HOW ARE THE 32 TABLE VALUES CREATED?
===========================================================

For every possible bit position:

    position = 0 ... 31

we calculate:

    ((1U << position) * 0x077CB531U) >> 27


This gives a unique 5-bit index.

Then we store:

    table[index] = position


Conceptually:

    for position = 0 to 31

        index =
            ((1U << position)
             * 0x077CB531U) >> 27;

        table[index] = position;


The resulting table is then hard-coded.

This is why the table looks random.

It is not random.

It is simply the reverse mapping of the generated indices.


===========================================================
WHY 5 BITS?
===========================================================

For a 32-bit integer:

    Number of possible bit positions = 32

We need to represent:

    0 through 31

Number of bits required:

    log2(32) = 5


Therefore:

    >> 27

extracts the top 5 bits from the 32-bit multiplication
result.

Because:

    32 - 5 = 27


So:

    product >> 27

gives us a value between:

    0 and 31


which can directly index the 32-element table.


===========================================================
LOWEST SET BIT VS HIGHEST SET BIT
===========================================================

Be careful here.

There are two common questions:

    1. Find position of the lowest set bit.
    2. Find position of the highest set bit.


For LOWEST set bit:

    value & -value

or, using unsigned arithmetic:

    value & (~value + 1U)


Example:

    value = 10110000

    lowest set bit:

        00010000


Then apply the De Bruijn lookup.

For HIGHEST set bit, a different preprocessing step is
normally used, often involving:

    CLZ
    bit spreading
    or another lookup technique.


===========================================================
COMBINING LOWEST-BIT ISOLATION WITH DE BRUIJN
===========================================================

If the input can contain multiple set bits:

    value = 10110100


We can first isolate the lowest set bit:

    value & (~value + 1U)

Result:

    00000100


Now the result contains exactly one set bit.

Then:

    De Bruijn lookup

returns:

    position = 2


Therefore, a more general helper can be:

    uint32_t position_of_lowest_set_bit(uint32_t value)
    {
        uint32_t bit = value & (~value + 1U);

        return bit_position_debruijn(bit);
    }


IMPORTANT:

    value == 0

must be handled separately.

There is no set bit position when the value is zero.


===========================================================
ZERO CASE
===========================================================

For:

    value = 0

there is no set bit.

Therefore this function:

    bit_position_debruijn(0)

should NOT be called unless your API explicitly defines
what zero means.

A safe wrapper is:


*/

static int32_t lowest_set_bit_position(uint32_t value)
{
    /*
     * Zero has no set-bit position.
     */
    if (value == 0U)
    {
        return -1;
    }

    /*
     * Isolate the lowest set bit.
     *
     * Example:
     *
     *     value = 10110100
     *
     *     isolated = 00000100
     */
    uint32_t isolated = value & (~value + 1U);

    /*
     * Now exactly one bit is set.
     */
    return (int32_t)bit_position_debruijn(isolated);
}


/*
===========================================================
SPACE-TIME TRADEOFF
===========================================================

The question asks:

    "32 bytes table vs log operations?"

This is an important embedded consideration.


OPTION 1:
LOOP / SHIFT METHOD

Example:

    position = 0;

    while ((value & 1U) == 0U)
    {
        value >>= 1U;
        position++;
    }

Advantages:

    - Very simple
    - Almost no extra memory
    - Easy to understand

Disadvantage:

    - Execution time depends on bit position
    - Worst case can require many iterations


===========================================================
OPTION 2:
DE BRUIJN
===========================================================

Uses:

    multiplication
    shift
    lookup table


Advantages:

    - Fixed number of operations
    - Very fast on suitable CPUs
    - Small lookup table
    - No loop required


Disadvantages:

    - More difficult to understand
    - Requires lookup table
    - Requires multiplication
    - Constant/table must match
    - Potential flash/cache access considerations


For uint32_t:

    table = 32 entries

If using uint8_t:

    32 bytes


So the memory cost is tiny on many systems.


===========================================================
OPTION 3:
HARDWARE CLZ / CTZ
===========================================================

Modern CPUs often provide instructions for:

    CLZ = Count Leading Zeros
    CTZ = Count Trailing Zeros


For lowest set bit:

    CTZ(value)

directly gives the position of the lowest set bit.

For example:

    value = 00010000

    CTZ(value) = 4


If the compiler provides:

    __builtin_ctz(value)

the compiler may generate a hardware instruction.

This can be preferable to manually implementing De Bruijn.


IMPORTANT:

    __builtin_ctz(0)

is undefined behavior in GCC/Clang.

Therefore check:

    if (value == 0U)

before calling it.


===========================================================
WHEN IS DE BRUIJN WORTH THE COMPLEXITY?
===========================================================

Use it when:

    - You need fast bit-position lookup.
    - Hardware CTZ/CLZ isn't available.
    - The compiler isn't generating a good instruction.
    - The operation is performance-critical.
    - A small lookup table is acceptable.
    - You need predictable execution.


It may NOT be worth it when:

    - Code readability is more important.
    - The operation happens rarely.
    - Hardware CTZ is available.
    - Compiler builtin generates optimal assembly.
    - Flash/cache behavior makes table lookup undesirable.


===========================================================
INTERVIEW QUESTION
===========================================================

Interviewer:

    "Why use a De Bruijn sequence?"

Good answer:

    "For a value containing a single set bit, a De Bruijn
     constant can encode the bit position into a small
     lookup-table index. We multiply the value by the
     constant, shift to extract a unique 5-bit index, and
     use a 32-entry table to recover the bit position.
     This gives a fixed number of operations instead of
     scanning the bits one by one."


===========================================================
INTERVIEW FOLLOW-UP:
"WHY 32 TABLE ENTRIES?"
===========================================================

Answer:

    "Because uint32_t has 32 possible bit positions,
     numbered 0 through 31. Five bits are sufficient to
     represent 32 different indices, so the multiplication
     result is shifted by 27 bits to obtain a 5-bit index."


===========================================================
INTERVIEW FOLLOW-UP:
"CAN YOU USE IT WITH MULTIPLE SET BITS?"
===========================================================

Answer:

    "The basic De Bruijn lookup expects a value containing
     exactly one set bit. If the original value contains
     multiple bits, I first isolate the lowest set bit using
     value & -value, then apply the De Bruijn lookup."


For unsigned C:

    uint32_t bit = value & (~value + 1U);


===========================================================
INTERVIEW FOLLOW-UP:
"WHAT IF VALUE IS ZERO?"
===========================================================

Answer:

    "Zero has no set-bit position, so I need to define a
     special return value or handle zero before performing
     the lookup."


For example:

    return -1;


if the API uses a signed return value.


===========================================================
KEY CONCEPT
===========================================================

The entire trick can be remembered as:

    ONE SET BIT
         |
         v
    MULTIPLY
         |
         v
    DE BRUIJN CONSTANT
         |
         v
    SHIFT
         |
         v
    5-BIT INDEX
         |
         v
    32-ENTRY TABLE
         |
         v
    BIT POSITION


===========================================================
MOST IMPORTANT FORMULA
===========================================================

For uint32_t:

    index =
        (single_bit * 0x077CB531U) >> 27;


Then:

    position = table[index];


The important condition is:

    single_bit contains exactly ONE set bit.


===========================================================
FINAL MEMORY TRICK
===========================================================

Question:
    "How does De Bruijn find bit position quickly?"

Remember:

    ISOLATE
       ↓
    MULTIPLY
       ↓
    SHIFT
       ↓
    LOOKUP


And remember the tradeoff:

    LOOP
      ↓
    less memory
    but variable execution time

    DE BRUIJN
      ↓
    tiny table
    + fixed operations
    + fast

    HARDWARE CTZ
      ↓
    usually simplest/fastest when available


For modern embedded development, don't blindly choose
De Bruijn.

First check:

    1. Does the MCU have CLZ/CTZ?
    2. Does the compiler expose it?
    3. What assembly does the compiler generate?
    4. Is this operation actually performance-critical?

Then choose the simplest implementation that meets the
measured performance requirement.
*/
```
#### 38. Bit manipulation for fixed-point math.
    - Counter: Multiply and keep fractional part (>>16 for 16.16 fixed)?
    - Counter: Rounding modes: truncate vs round-to-nearest?
    - Counter: Overflow prevention?

```c++
#include <stdint.h>
#include <limits.h>

/*
===========================================================
Question 38
Bit Manipulation for Fixed-Point Math
===========================================================

WHAT IS FIXED-POINT MATH?
-------------------------

Fixed-point arithmetic represents a fractional number using
an integer.

Instead of storing:

    12.75

directly as a floating-point number, we can store it as an
integer with an implied decimal/binary point.

For a 16.16 fixed-point format:

    16 bits = integer part
    16 bits = fractional part

Total:

    32 bits


Example:

    12.75

Integer part:

    12

Fractional part:

    0.75


Multiply the real value by:

    2^16 = 65536

Therefore:

    12.75 * 65536 = 835584

So we store:

    835584

as an integer.

When interpreting the integer as 16.16 fixed point:

    835584 / 65536 = 12.75


===========================================================
WHY USE FIXED-POINT?
===========================================================

Fixed-point is useful in embedded systems when:

    - Floating-point hardware is unavailable.
    - Deterministic execution time is important.
    - Memory is limited.
    - We need predictable arithmetic.
    - We want to avoid floating-point overhead.

Common applications:

    - Motor control
    - Audio processing
    - Sensor processing
    - DSP
    - Graphics
    - PID controllers
    - Robotics
    - Automotive control


===========================================================
16.16 REPRESENTATION
===========================================================

A 32-bit 16.16 value looks like:

    +----------------+----------------+
    | integer (16)   | fraction (16)  |
    +----------------+----------------+

Example:

    5.25


    5.25 * 65536
    = 344064


In hexadecimal:

    5.25 = 0x0005.4000

Conceptually:

    0000 0000 0000 0101 0100 0000 0000 0000
    <------ integer ------><--- fraction --->


===========================================================
CONVERSION: FLOAT -> FIXED
===========================================================

For 16.16:

    fixed = real_value * 65536


Example:

    2.5

    2.5 * 65536
    = 163840

So:

    fixed = 163840


===========================================================
CONVERSION: FIXED -> INTEGER
===========================================================

Suppose:

    fixed = 2.75 * 65536

To remove the fractional portion:

    integer = fixed >> 16


Why?

Because dividing by:

    2^16

is equivalent to:

    >> 16


Example:

    fixed = 2.75 * 65536
          = 180224

Then:

    180224 >> 16
          = 2

The fractional .75 is discarded.


===========================================================
TRUNCATION VS ROUNDING
===========================================================

This is an important interview question.

Simply doing:

    integer = fixed >> 16;

performs truncation.

Example:

    12.75 -> 12

The .75 is discarded.


If we want round-to-nearest instead:

    integer = (fixed + 32768) >> 16;


Why 32768?

Because:

    2^15 = 32768

which is half of:

    2^16 = 65536


Therefore:

    + 0.5

is added before shifting.


Example:

    12.75

    12.75 * 65536
    = 835584

Add half:

    835584 + 32768
    = 868352

Then:

    868352 >> 16
    = 13

So:

    12.75 -> 13


For positive values, this gives the familiar
round-to-nearest behavior.


===========================================================
FIXED-POINT MULTIPLICATION
===========================================================

This is where the bit manipulation becomes particularly
important.

Suppose we have two 16.16 numbers:

    A
    B

Each represents:

    real_A = A / 2^16
    real_B = B / 2^16


Normal integer multiplication gives:

    A * B


But the result now has:

    32 fractional bits


because:

    2^16 * 2^16 = 2^32


We need to shift right by 16 to get back to 16.16:

    result = (A * B) >> 16


===========================================================
EXAMPLE
===========================================================

Suppose:

    A = 2.5

    B = 4.0


16.16 representations:

    A = 2.5 * 65536
      = 163840

    B = 4.0 * 65536
      = 262144


Multiply:

    A * B
    = 163840 * 262144


The result contains 32 fractional bits.

Therefore:

    result = (A * B) >> 16


The final result represents:

    10.0


===========================================================
IMPORTANT OVERFLOW ISSUE
===========================================================

This is one of the biggest traps.

If A and B are both uint32_t:

    A * B

can require up to 64 bits.

If we perform:

    uint32_t result = (A * B) >> 16;

the multiplication may overflow BEFORE the shift.


Therefore, use a wider intermediate type when available:

    uint64_t result =
        ((uint64_t)A * (uint64_t)B) >> 16;


This is extremely important in embedded code.


===========================================================
SIGNED FIXED-POINT
===========================================================

For signed fixed-point values:

    int32_t

can be used.

Example:

    -2.5

would be represented using two's complement.

Multiplication should use a wider signed intermediate:

    int64_t product =
        (int64_t)A * (int64_t)B;

Then:

    int32_t result =
        (int32_t)(product >> 16);


However, overflow still needs to be considered before
converting back to int32_t.


===========================================================
FIXED-POINT ADDITION
===========================================================

Addition is much simpler.

If:

    A = 2.5
    B = 3.25

both use the same 16.16 format:

    result = A + B;


No shift is required.

Why?

Because both values have the same scaling factor:

    2^16


Therefore:

    A + B

already has the correct fixed-point format.


===========================================================
FIXED-POINT SUBTRACTION
===========================================================

Similarly:

    result = A - B;


No scaling conversion is necessary as long as both operands
use the same fixed-point format.


===========================================================
FIXED-POINT DIVISION
===========================================================

Division requires the opposite scaling adjustment.

For:

    result = A / B

we want the result to remain in 16.16 format.

Therefore:

    result = (A << 16) / B;


The left shift restores the required fractional scaling.


Example:

    A = 10.0
    B = 4.0

Expected:

    result = 2.5


Conceptually:

    (A << 16) / B


Again, a wider intermediate may be required to avoid
overflow.


===========================================================
OVERFLOW PREVENTION
===========================================================

Three common strategies:


1. USE A WIDER INTERMEDIATE

For multiplication:

    uint64_t product =
        (uint64_t)A * B;


This prevents intermediate multiplication overflow.


2. CHECK RANGE BEFORE OPERATION

Before adding:

    A + B

check whether the result can fit in the destination type.


3. SATURATION

Instead of allowing:

    MAX + 1

to wrap around to zero, clamp the result:

    MAX


This is called saturated arithmetic.

It is common in DSP and control systems.


===========================================================
WRAPAROUND VS SATURATION
===========================================================

Suppose uint8_t maximum is:

    255


Normal unsigned arithmetic:

    255 + 1

wraps:

    0


Saturated arithmetic:

    255 + 1

becomes:

    255


In signal processing, saturation is often preferable because
wraparound can create a very large unexpected error.


===========================================================
ROUNDING DURING MULTIPLICATION
===========================================================

Simple:

    result = product >> 16;


This truncates.


For positive values, round-to-nearest can be:

    result = (product + (1ULL << 15)) >> 16;


Because:

    1 << 15

is half of the scaling factor.


For signed values, however, rounding requires more care.

You should not blindly add the same positive offset to a
signed two's-complement value and assume symmetric rounding.

The required behavior depends on the chosen rounding mode.


===========================================================
FIXED-POINT EXAMPLE FUNCTION
===========================================================
*/

static int32_t fixed16_mul(int32_t a, int32_t b)
{
    /*
     * Use int64_t so the multiplication does not overflow
     * during the intermediate calculation.
     */
    int64_t product = (int64_t)a * (int64_t)b;

    /*
     * Both inputs have 16 fractional bits.
     *
     * Multiplication produces 32 fractional bits.
     *
     * Shift right by 16 to return to 16.16 format.
     */
    return (int32_t)(product >> 16);
}


/*
===========================================================
KEY FIXED-POINT RULES
===========================================================

For Q16.16:

    Addition:
        A + B

    Subtraction:
        A - B

    Multiplication:
        (A * B) >> 16

    Division:
        (A << 16) / B

    Fixed -> integer:
        value >> 16

    Integer -> fixed:
        integer << 16


IMPORTANT:

Use wider intermediate values whenever multiplication or
shifting can overflow the original type.


===========================================================
INTERVIEW SUMMARY
===========================================================

Question:

    "How does fixed-point math use bit manipulation?"

Answer:

    "Fixed-point stores a scaled integer with an implied
     binary point. For Q16.16, the lower 16 bits represent
     the fractional part. Multiplication produces an extra
     16 fractional bits, so we shift the product right by
     16. Shifts can replace multiplication or division by
     powers of two, making fixed-point arithmetic efficient
     on systems without fast floating-point hardware."

*/
```

#### 39. Gosper's Hack: next permutation of bits.
    - Counter: What does this generate?
    - Counter: Used for efficient subset enumeration?
    - Counter: Real-time constraints: single operation?


```c++
#include <stdint.h>

/*
===========================================================
Question 39
Gosper's Hack: Next Permutation of Bits
===========================================================

GOAL
----

Generate the next larger integer containing the SAME number
of set bits.

For example, suppose we want all 8-bit patterns containing
exactly 3 set bits.

Start:

    00000111

Next:

    00001011

Next:

    00001101

Next:

    00001110

Next:

    00010011

...

Every value contains exactly:

    3 ones


===========================================================
WHY IS THIS USEFUL?
===========================================================

This is useful when enumerating combinations.

Suppose we have:

    N = 8 elements

and want every combination containing:

    K = 3 elements.


We can represent a combination as a bitmask.

Example:

    00010110

Set bits:

    bit 1
    bit 2
    bit 4

represent the selected elements.


The next combination can be generated directly without
checking every integer.

This is useful in:

    - Subset enumeration
    - Combination generation
    - Dynamic programming
    - Search algorithms
    - Combinatorial algorithms
    - Bitmask-based algorithms


===========================================================
THE GOSPER FORMULA
===========================================================

The classic unsigned implementation is:


    uint32_t c = x & -x;
    uint32_t r = x + c;
    x = (((r ^ x) >> 2U) / c) | r;


The result is the next larger value with the same number
of set bits.


===========================================================
STEP 1
===========================================================

    c = x & -x;


This isolates the lowest set bit.

For unsigned C, it is often clearer to write:

    c = x & (~x + 1U);


Example:

    x = 00101100


Lowest set bit:

    c = 00000100


So:

    c = 4


===========================================================
STEP 2
===========================================================

    r = x + c;


We add the lowest set bit to the original value.

Example:

    x = 00101100
    c = 00000100

    x + c
    -------
    r = 00110000


This moves the pattern toward the next combination.


===========================================================
STEP 3
===========================================================

    r ^ x


Using:

    x = 00101100
    r = 00110000

we get:

    r ^ x
    -------
    00011100


This identifies the bits that changed during the addition.


===========================================================
STEP 4
===========================================================

    (r ^ x) >> 2


Shift right by 2:

    00011100
          >>
    00000111


The purpose is to prepare the remaining set bits so that
they can be packed into the lowest available positions.


===========================================================
STEP 5
===========================================================

    (...) / c


We divide by the isolated lowest set bit.

Since c is a power of two:

    c = 1
    c = 2
    c = 4
    c = 8
    ...


division by c is equivalent to a right shift by the
corresponding number of bits.

Conceptually this compresses the remaining set bits.


===========================================================
STEP 6
===========================================================

    ... | r


Finally we OR the packed lower bits with r.

This produces the next larger number containing exactly
the same number of set bits.


===========================================================
COMPLETE FUNCTION
===========================================================
*/

static uint32_t gosper_next(uint32_t x)
{
    /*
     * Isolate the lowest set bit.
     *
     * Example:
     *
     *     x = 00101100
     *
     *     c = 00000100
     */
    uint32_t c = x & (~x + 1U);

    /*
     * Move the lowest set-bit boundary upward.
     */
    uint32_t r = x + c;

    /*
     * Rearrange the remaining lower bits so that they are
     * placed in the lowest possible positions.
     *
     * The result is the next larger integer with the same
     * number of set bits.
     */
    return (((r ^ x) >> 2U) / c) | r;
}


/*
===========================================================
EXAMPLE
===========================================================

Suppose:

    x = 00101100

Number of set bits:

    3

because:

    00101100
       ^^^
       3 ones


Apply Gosper's Hack.

Next result:

    00110001

Count the bits:

    00110001
       ^^^
       3 ones


And:

    00110001 > 00101100


So it is the next larger 3-bit combination.


===========================================================
ENUMERATING ALL COMBINATIONS
===========================================================

Suppose:

    N = 8

and:

    K = 3


The first mask with 3 bits set is:

    00000111


Then repeatedly:

    x = gosper_next(x);


We get:

    00000111
    00001011
    00001101
    00001110
    00010011
    00010101
    00010110
    00011001
    ...


Every mask contains exactly:

    3 set bits.


===========================================================
WHY IS THIS BETTER THAN CHECKING EVERY INTEGER?
===========================================================

Suppose we want all 8-bit numbers containing exactly
3 set bits.

There are:

    C(8,3)

combinations.

That is:

    8! / (3! * 5!)
    = 56


A naive approach could iterate through:

    0
    1
    2
    ...
    255

and test:

    popcount(x) == 3


That checks all 256 possible values.

Gosper's Hack jumps directly:

    combination
        ↓
    next combination
        ↓
    next combination
        ↓
    ...


It avoids examining values that do not contain the required
number of set bits.


===========================================================
COMBINATION COUNT
===========================================================

For N bits and K set bits, the number of combinations is:

    C(N,K)

or:

    N! / (K!(N-K)!)


For example:

    N = 8
    K = 3

    C(8,3) = 56


Gosper's Hack generates those 56 masks directly.


===========================================================
IMPORTANT EDGE CASE
===========================================================

The formula assumes:

    x != 0


Why?

Because:

    c = x & -x


would become zero for:

    x = 0


and then:

    / c

would divide by zero.


Therefore the caller must handle zero.


===========================================================
ANOTHER IMPORTANT EDGE CASE
===========================================================

Suppose we have 8 bits and:

    x = 11100000


This is already the LAST 8-bit combination containing
three set bits.

There is no next combination within 8 bits.

If we use a 32-bit integer, Gosper's Hack may produce a
larger value outside the intended N-bit range.

Therefore, when enumerating N-bit combinations, we need to
check whether the next value exceeds the N-bit limit.


===========================================================
EXAMPLE COMBINATION ENUMERATOR
===========================================================
*/

static void enumerate_combinations(void)
{
    /*
     * Example:
     *
     * N = 8 bits
     * K = 3 set bits
     *
     * First combination:
     *
     *     00000111
     */
    uint32_t x = 0x07U;

    /*
     * Highest allowed 8-bit value.
     */
    const uint32_t limit = 0x100U;

    while (x < limit)
    {
        /*
         * 'x' represents one combination.
         *
         * Example:
         *
         *     00000111
         *     00001011
         *     00001101
         *     ...
         */

        /*
         * Generate the next combination.
         */
        uint32_t next = gosper_next(x);

        /*
         * If next exceeds our N-bit range, stop.
         */
        if (next >= limit)
        {
            break;
        }

        x = next;
    }
}


/*
===========================================================
WHY DOES IT PRESERVE THE NUMBER OF SET BITS?
===========================================================

This is the clever part.

Suppose:

    x = 00101110

The lower portion has a pattern of:

    ...01110

The algorithm:

    1. Finds the lowest set bit.
    2. Moves a boundary bit upward.
    3. Rearranges the remaining set bits into the lowest
       possible positions.

Therefore:

    Original:
        00101110

    Next:
        00110101

Both contain the same number of 1s.

The result is also the smallest integer greater than x
having that same number of set bits.


===========================================================
"NEXT PERMUTATION" MEANING
===========================================================

The term "permutation" here can be slightly confusing.

We are not randomly permuting all bits.

Instead, if we interpret:

    1 = selected
    0 = not selected

then a bitmask represents a combination.

Example:

    00101100

could mean:

    select element 2
    select element 3
    select element 5

Gosper's Hack generates the next combination in numerical
order.


===========================================================
REAL-TIME CONSIDERATION
===========================================================

The question asks:

    "Real-time constraints: single operation?"


Be careful here.

Gosper's Hack is a very small fixed sequence of arithmetic
and bitwise operations.

It does NOT mean:

    "one CPU instruction"


It means:

    "one compact algorithmic step"


The implementation contains:

    - subtraction/negation
    - AND
    - addition
    - XOR
    - shift
    - division
    - OR


The division can be particularly important.

Depending on the target MCU:

    division may be cheap,
    or
    division may be expensive.


So you should NOT claim:

    "Gosper's Hack is always constant-time and extremely
     fast."


Instead:

    "The algorithm uses a fixed number of high-level
     operations, but actual execution time depends on the
     target architecture, especially the division."


===========================================================
CAN THE DIVISION BE OPTIMIZED?
===========================================================

Yes.

Recall:

    c = x & -x


c is always a power of two.

For example:

    1
    2
    4
    8
    16
    ...


Division by a power of two can be replaced with a shift.

For example:

    value / 8

is equivalent to:

    value >> 3


However, the shift amount must be determined from c.

A compiler may already recognize and optimize this pattern,
depending on the code and target.


===========================================================
SPACE REQUIREMENT
===========================================================

Unlike the De Bruijn technique:

    Gosper's Hack does NOT require a lookup table.


It uses:

    O(1) extra space.


This makes it attractive when memory is constrained.


===========================================================
WHEN IS GOSPER'S HACK USEFUL?
===========================================================

Useful for:

    - Enumerating combinations
    - Bitmask dynamic programming
    - Subset search
    - Hardware configuration combinations
    - Scheduling combinations
    - Competitive programming
    - Combinatorial algorithms


Example:

Suppose 16 sensors exist but exactly 4 sensors must be
selected.

Instead of generating all:

    2^16 = 65536

masks and filtering them,

we can generate only:

    C(16,4)

valid masks.


C(16,4) = 1820


Gosper's Hack can jump directly from one valid 4-bit mask
to the next.


===========================================================
INTERVIEW QUESTION
===========================================================

Interviewer:

    "What does Gosper's Hack do?"

Good answer:

    "It generates the next larger integer having the same
     number of set bits. It is useful for efficiently
     enumerating combinations represented as bitmasks."


===========================================================
FOLLOW-UP:
"WHY IS THAT USEFUL?"
===========================================================

Answer:

    "If I need all combinations of N elements choosing K,
     I can represent each combination as an N-bit mask with
     K set bits. Gosper's Hack generates only those valid
     masks instead of scanning every possible N-bit value."


===========================================================
FOLLOW-UP:
"IS IT ONE CPU INSTRUCTION?"
===========================================================

Answer:

    "No. It's a compact sequence of bitwise and arithmetic
     operations. Its actual execution time depends on the
     processor and compiler. In particular, division can be
     relatively expensive on some MCUs."


===========================================================
FOLLOW-UP:
"WHAT IS THE SPACE COMPLEXITY?"
===========================================================

Answer:

    "O(1) extra space. It doesn't require a lookup table."


===========================================================
FOLLOW-UP:
"WHAT IS THE BIG ADVANTAGE?"
===========================================================

The important point is:

    DON'T ENUMERATE INVALID MASKS.


Instead of:

    00000000
    00000001
    00000010
    00000011
    ...
    and checking popcount,


Gosper generates:

    00000111
    00001011
    00001101
    00001110
    ...

only the masks containing the required number of set bits.


===========================================================
FINAL MEMORY TRICK
===========================================================

Gosper's Hack:

    x
    ↓
    isolate lowest set bit
    ↓
    add it to x
    ↓
    identify changed bits
    ↓
    pack remaining bits
    ↓
    next combination


Classic formula:

    c = x & -x;
    r = x + c;

    next = (((r ^ x) >> 2) / c) | r;


The key sentence to remember:

    "Gosper's Hack generates the next larger bitmask with
     the same population count."


===========================================================
Q38 vs Q39
===========================================================

Q38 — Fixed Point:

    Problem:
        Represent fractional numbers without floating point.

    Main tools:
        shifts
        multiplication
        wider intermediate
        rounding
        saturation


Q39 — Gosper's Hack:

    Problem:
        Generate combinations represented by bitmasks.

    Main tools:
        isolate lowest bit
        addition
        XOR
        shift
        division
        OR


Q38 is mainly about:

    NUMERICAL REPRESENTATION


Q39 is mainly about:

    COMBINATORIAL ENUMERATION
*/
```

#### 40. Low-level hardware test: verify register read-write.
    - Counter: Write patterns and read back?
    - Counter: Handle read-only bits?
    - Counter: Detect stuck-at faults?


```c++
#include <stdint.h>
#include <stdbool.h>

/*
===========================================================
Question 40
Low-Level Hardware Test: Verify Register Read/Write
===========================================================

GOAL
----

At the hardware bring-up stage, we often need to verify that
a peripheral register can actually be written and read back
correctly.

Basic idea:

    1. Write a known test pattern.
    2. Read the register.
    3. Compare the read value with the expected value.
    4. Report any mismatch.


Example:

    register = 0xAAAAAAAA

Write:

    10101010 10101010 10101010 10101010

Read it back.

If we get:

    10101010 10101010 10101010 10101010

the tested bits behave as expected.


===========================================================
IMPORTANT:
DO NOT BLINDLY WRITE 0xFFFFFFFF
===========================================================

A hardware register is not necessarily a normal RAM variable.

Different bits may have different behavior:

    RW  -> Read/Write
    RO  -> Read Only
    WO  -> Write Only
    W1C -> Write 1 to Clear
    RC  -> Read to Clear
    Reserved -> Must not be modified


Therefore, a register test must use the hardware reference
manual to determine which bits are actually safe to test.


===========================================================
TESTABLE BIT MASK
===========================================================

Suppose:

    Register = 32 bits

but only these bits are Read/Write:

    bits 0-3
    bits 8-15

Then:

    RW_MASK =
        0x0000FF0F


We should only compare those bits.


===========================================================
FUNCTION
===========================================================
*/

static bool verify_register_rw(volatile uint32_t *reg,
                                uint32_t rw_mask)
{
    /*
     * Test pattern 1:
     *
     * Alternating bits.
     *
     * This helps expose certain stuck-at and coupling
     * problems.
     */
    const uint32_t pattern1 = 0xAAAAAAAAU;

    /*
     * Test pattern 2:
     *
     * Opposite alternating pattern.
     */
    const uint32_t pattern2 = 0x55555555U;

    /*
     * Write only the bits that are documented as RW.
     *
     * Important:
     *
     * This assumes the register's existing value can safely
     * be preserved for bits outside rw_mask.
     *
     * The exact write mechanism depends on the peripheral.
     */
    uint32_t original = *reg;

    /*
     * Write pattern 1 into the RW bits.
     */
    *reg = (original & ~rw_mask) |
           (pattern1 & rw_mask);

    /*
     * Read back.
     *
     * Only compare the bits that are expected to be RW.
     */
    uint32_t readback = *reg;

    if ((readback & rw_mask) != (pattern1 & rw_mask))
    {
        return false;
    }

    /*
     * Write the opposite pattern.
     *
     * Using more than one pattern is important.
     */
    *reg = (original & ~rw_mask) |
           (pattern2 & rw_mask);

    /*
     * Read back again.
     */
    readback = *reg;

    if ((readback & rw_mask) != (pattern2 & rw_mask))
    {
        return false;
    }

    /*
     * Register behaved correctly for the tested patterns.
     */
    return true;
}


/*
===========================================================
WHY USE MULTIPLE PATTERNS?
===========================================================

Suppose a bit is stuck at:

    0

If we only write:

    00000000

we may never detect the problem.

Similarly, if a bit is stuck at:

    1

writing all 1s won't expose it.

Therefore use complementary patterns such as:

    0xAAAAAAAA
    0x55555555

These contain alternating 0/1 values.


===========================================================
STUCK-AT FAULT
===========================================================

A stuck-at fault means a bit behaves as though it is
permanently:

    stuck at 0

or:

    stuck at 1


Example:

Expected:

    1010

Actual:

    1000

The difference indicates that one tested bit did not
behave as expected.

IMPORTANT:

A read/write test can detect many stuck-at faults, but
it does NOT prove that the hardware is completely fault-free.

Other possible problems include:

    - Coupling faults
    - Timing faults
    - Bus faults
    - Address decoding faults
    - Peripheral-specific behavior


===========================================================
INTERVIEW ANSWER
===========================================================

"How would you verify a hardware register?"

Answer:

    "I would first identify the register's access type and
     writable bit mask from the datasheet/reference manual.
     Then I would write known patterns such as 0xAAAAAAAA
     and 0x55555555 to the RW bits, read the register back,
     mask the expected bits, and compare. I would avoid
     writing reserved, read-only, W1C, or other
     side-effect bits."
*/
```

#### 41. Register shadowing for protection against corruption.
    - Counter: Maintain shadow copy in RAM?
    - Counter: When to check: every access or periodic?
    - Counter: Recovery strategy?

```c++
#include <stdint.h>
#include <stdbool.h>

/*
===========================================================
Question 41
Register Shadowing for Protection Against Corruption
===========================================================

GOAL
----

A critical hardware configuration register may be corrupted
because of:

    - Software bugs
    - Memory corruption
    - Unexpected writes
    - EMI/noise
    - Hardware faults
    - Erroneous peripheral behavior


A common defensive technique is REGISTER SHADOWING.


===========================================================
WHAT IS REGISTER SHADOWING?
===========================================================

Maintain two copies:

    Hardware register
            |
            +---- expected configuration
                   stored in RAM

Example:

    HW register:
        0x00001234

    Shadow:
        0x00001234


Periodically compare them.

If:

    hardware != shadow

then something has changed unexpectedly.


===========================================================
BASIC IMPLEMENTATION
===========================================================
*/

typedef struct
{
    /*
     * Expected configuration value.
     */
    uint32_t shadow;

    /*
     * Mask of bits that are expected to remain constant.
     */
    uint32_t mask;

} RegisterShadow;


/*
===========================================================
INITIALIZATION
===========================================================
*/

static void shadow_init(RegisterShadow *shadow,
                        volatile uint32_t *reg,
                        uint32_t mask)
{
    /*
     * Read the current hardware configuration.
     */
    shadow->shadow = *reg;

    /*
     * Remember which bits are protected.
     */
    shadow->mask = mask;
}


/*
===========================================================
CHECK FUNCTION
===========================================================
*/

static bool shadow_check(const RegisterShadow *shadow,
                         volatile uint32_t *reg)
{
    /*
     * Read current hardware value.
     */
    uint32_t actual = *reg;

    /*
     * Compare only protected bits.
     */
    return ((actual ^ shadow->shadow) &
            shadow->mask) == 0U;
}


/*
===========================================================
RECOVERY
===========================================================

Detection alone may not be enough.

Suppose:

    shadow = 0x00001234

but hardware reads:

    0x00001034

We detected corruption.

Possible recovery:

    1. Log the fault.
    2. Rewrite the expected register value.
    3. Verify it again.
    4. Escalate if it cannot be restored.


Example:
===========================================================
*/

static bool shadow_recover(RegisterShadow *shadow,
                           volatile uint32_t *reg)
{
    /*
     * Rewrite the expected configuration.
     */
    uint32_t current = *reg;

    uint32_t repaired =
        (current & ~shadow->mask) |
        (shadow->shadow & shadow->mask);

    *reg = repaired;

    /*
     * Verify the repair.
     */
    return shadow_check(shadow, reg);
}


/*
===========================================================
WHEN SHOULD WE CHECK?
===========================================================

OPTION 1:
EVERY ACCESS

Advantages:

    - Very fast detection
    - Tight protection

Disadvantages:

    - More CPU overhead
    - More code around every access


OPTION 2:
PERIODIC CHECK

For example:

    every 10 ms
    every 100 ms
    every scheduler cycle


Advantages:

    - Lower overhead
    - Easy to integrate into periodic monitoring


Disadvantage:

    - Corruption may exist between checks.


OPTION 3:
EVENT-BASED

Check after:

    - Interrupt
    - DMA activity
    - Peripheral reset
    - Mode transition
    - Safety-critical operation


===========================================================
RECOVERY STRATEGY
===========================================================

Detection:

    HW != SHADOW

Then:

    1. Detect
    2. Record diagnostic
    3. Rewrite
    4. Read back
    5. If still incorrect -> escalate


Possible escalation:

    - Retry
    - Disable peripheral
    - Enter safe state
    - Trigger system reset
    - Report diagnostic fault


The correct recovery depends heavily on the safety
requirements of the system.


===========================================================
IMPORTANT LIMITATION
===========================================================

A shadow copy in RAM is NOT automatically independent.

Suppose RAM itself is corrupted:

    shadow = corrupted

Then:

    hardware == corrupted shadow

and the comparison may incorrectly report success.


For safety-critical systems, additional mechanisms may
be needed:

    - ECC-protected RAM
    - CRC
    - Redundant storage
    - Independent monitoring
    - Hardware safety mechanisms


===========================================================
INTERVIEW ANSWER
===========================================================

"Register shadowing keeps an expected copy of a critical
configuration register and compares the hardware register
against it periodically or at important checkpoints. If a
mismatch occurs, the software can attempt to restore the
expected value, verify it, and escalate to a safe state if
recovery fails."
*/
```

#### 42. Bit rotation for CAN ID matching (extended vs standard).
    - Counter: Filter match in hardware vs firmware?
    - Counter: Performance: software filtering bottleneck?
    - Counter: Which approach for automotive safety?

```c++
#include <stdint.h>
#include <stdbool.h>

/*
===========================================================
Question 42
Bit Rotation for CAN ID Matching
===========================================================

IMPORTANT:

CAN filtering normally does NOT require rotating the CAN ID.

The important concepts are:

    - Standard CAN ID: 11 bits
    - Extended CAN ID: 29 bits
    - Hardware acceptance filters
    - Mask-based filtering
    - List/ID filtering
    - Firmware filtering


Bit rotation may be useful for custom software processing,
hashing, or lookup schemes, but it is not inherently part
of CAN ID matching.


===========================================================
STANDARD VS EXTENDED CAN
===========================================================

Standard CAN:

    11-bit identifier

Range:

    0x000 to 0x7FF


Extended CAN:

    29-bit identifier

Range:

    0x00000000 to 0x1FFFFFFF


The receiver needs to know whether the received frame is
standard or extended.


===========================================================
MASK-BASED MATCHING
===========================================================

A common software filter is:

    (received_id & mask) == (expected_id & mask)


Example:

    expected = 0x120
    mask     = 0x7F0


Then only selected bits participate in the comparison.


===========================================================
SOFTWARE FILTER EXAMPLE
===========================================================
*/

static bool can_id_match(uint32_t received_id,
                         uint32_t expected_id,
                         uint32_t mask)
{
    /*
     * Only bits where mask = 1 are compared.
     *
     * mask = 0 -> don't care
     * mask = 1 -> must match
     */
    return (received_id & mask) ==
           (expected_id & mask);
}


/*
===========================================================
STANDARD CAN EXAMPLE
===========================================================
*/

static bool standard_can_filter(uint16_t received_id)
{
    /*
     * Example:
     *
     * Accept IDs from:
     *
     *     0x120 to 0x12F
     *
     * The lower 4 bits are variable.
     */

    const uint16_t expected = 0x120U;
    const uint16_t mask     = 0x7F0U;

    return (received_id & mask) ==
           (expected & mask);
}


/*
===========================================================
HARDWARE FILTER VS FIRMWARE FILTER
===========================================================

Hardware filtering:

    CAN controller
          |
          | only accepted frames
          v
       CPU/ISR


Firmware filtering:

    CAN controller
          |
          | all received frames
          v
       CPU/ISR
          |
          v
      software filter


Hardware filtering is usually preferable when the
peripheral supports the required filtering.


Why?

Because rejected frames can be discarded before consuming
CPU resources.


===========================================================
PERFORMANCE BOTTLENECK
===========================================================

Suppose the CAN bus is busy.

If hardware accepts every frame:

    CAN frame
       ↓
    interrupt
       ↓
    CPU
       ↓
    software ID check
       ↓
    discard


The CPU has already paid the cost of:

    - Interrupt handling
    - Register reads
    - Buffer management
    - Context processing


If hardware filtering rejects the frame:

    CAN frame
       ↓
    hardware filter
       ↓
    rejected


CPU work is reduced.


===========================================================
WHERE WOULD BIT ROTATION FIT?
===========================================================

A custom firmware implementation might use rotation as part
of a hash:

    hash = rotate_left(id, n) ^ id;


This could map IDs into buckets.

However, this is NOT the same as CAN acceptance filtering.

If deterministic exact filtering is required, a mask/list
filter is generally more appropriate.


===========================================================
AUTOMOTIVE SAFETY
===========================================================

For safety-related automotive systems, the architecture
should not simply be:

    "Use hardware because it's faster."

The correct approach depends on:

    - Safety requirements
    - MCU/CAN controller capabilities
    - Diagnostic mechanisms
    - Freedom from interference
    - Worst-case CPU load
    - Required response time
    - Fault containment


Hardware filtering can reduce unnecessary CPU load.

Firmware validation can provide additional diagnostics.

In some designs, both can be used:

    Hardware filter
          ↓
    reduce traffic
          ↓
    Firmware validation
          ↓
    application


For safety-critical systems, the exact mechanism should be
derived from the system safety requirements rather than
assuming one approach is universally correct.


===========================================================
INTERVIEW ANSWER
===========================================================

"If the CAN controller supports suitable acceptance filters,
I would normally filter as early as possible in hardware to
avoid unnecessary CPU and interrupt load. Firmware filtering
is useful when filtering rules are dynamic or more complex.
For safety-related systems, I would consider worst-case
latency, diagnostics, fault handling, and the safety
requirements rather than choosing solely on performance."
*/
```

#### 43. Detect multiple errors: more than one bit flipped.
    - Counter: Hamming code limitations?
    - Counter: SECDED (single error correct, double error detect)?
    - Counter: Overkill for non-critical data?

```c++
#include <stdint.h>

/*
===========================================================
Question 43
Detect Multiple Bit Errors
===========================================================

The important concept here is:

    Hamming distance


A code's minimum Hamming distance determines how many
errors it can detect or correct.


===========================================================
HAMMING DISTANCE
===========================================================

If two valid codewords differ in at least:

    d

bit positions,

then the code has minimum Hamming distance:

    d


A code with minimum distance d can detect:

    d - 1

errors.


It can correct:

    floor((d - 1) / 2)

errors.


===========================================================
EXAMPLE
===========================================================

Suppose:

    d = 3

Then:

    Detection capability:
        2 errors

    Correction capability:
        1 error


This is the basis of:

    SEC

Single Error Correction


===========================================================
SECDED
===========================================================

SECDED means:

    Single Error Correct
    Double Error Detect


A classic SECDED code adds an additional parity bit to a
single-error-correcting code.


It can:

    - Correct one-bit error
    - Detect two-bit error


But it does NOT mean:

    "Detect any number of errors."


Three or more errors can produce more complicated failure
modes depending on the code.


===========================================================
WHY CAN'T SIMPLE PARITY HANDLE MULTIPLE ERRORS?
===========================================================

Suppose data is:

    1011001

Add one parity bit.

If one bit flips:

    parity changes

so we can detect an odd number of errors.

But if two bits flip:

    parity may remain unchanged.


Example:

Original:

    1011001

Two bits flipped:

    1110001

The parity can remain the same.


Therefore:

    Simple parity
        ↓
    detects many single-bit errors
        ↓
    but cannot reliably detect all multi-bit errors.


===========================================================
SECDED CONCEPT
===========================================================

A SECDED code generally has:

    Hamming code
        +
    overall parity bit


The Hamming portion provides syndrome information.

The overall parity helps distinguish:

    one-bit error
    two-bit error


Conceptually:

    syndrome = 0
    overall parity = correct

        -> no detected error


    syndrome != 0
    overall parity indicates odd error

        -> single-bit error
        -> correct it


    syndrome != 0
    overall parity indicates even error

        -> multi-bit error
        -> detect, but don't blindly correct


The exact syndrome interpretation depends on the
implementation.


===========================================================
WHY IS "MULTIPLE ERROR DETECTION" HARD?
===========================================================

A code must have sufficient minimum Hamming distance.

For example:

    d = 2

can detect:

    1 error


but cannot correct a single error.


For:

    d = 3

we can:

    detect 2
    correct 1


For:

    d = 4

we can:

    detect 3
    correct 1


and:

    floor((4 - 1) / 2)
    = 1


===========================================================
APPLICATIONS
===========================================================

Error-detecting/error-correcting codes are common in:

    - ECC RAM
    - Flash memories
    - Communication systems
    - Storage
    - Network links
    - Automotive safety systems
    - Aerospace systems


===========================================================
IS ECC OVERKILL?
===========================================================

It depends on the data.

For:

    temporary debug data

full ECC may be unnecessary.


For:

    safety-critical configuration
    control parameters
    memory containing important state


stronger protection may be justified.


The decision should consider:

    - Consequence of corruption
    - Fault model
    - Required diagnostic coverage
    - Hardware support
    - Performance
    - Memory overhead


===========================================================
INTERVIEW ANSWER
===========================================================

"If I need to handle multiple bit errors, I first look at
the required fault model and the code's minimum Hamming
distance. SECDED can correct a single-bit error and detect
a double-bit error, but it doesn't guarantee correction or
detection of arbitrary multi-bit errors. For safety-critical
data, the required protection should come from the system
fault analysis rather than automatically adding ECC."
*/
```

#### 44. Optimized XOR-based swap without temp variable.
    - Counter: a ^= b; b ^= a; a ^= b — does it work always?
    - Counter: Atomic operations: is this safe in ISR?
    - Counter: When not to use: performance penalty vs clarity?
```c++
#include <stdint.h>

/*
===========================================================
Question 44
XOR Swap
===========================================================

Classic XOR swap:

    a ^= b;
    b ^= a;
    a ^= b;


The result:

    a and b are exchanged.


===========================================================
WHY DOES IT WORK?
===========================================================

XOR has an important property:

    x ^ x = 0

and:

    x ^ 0 = x


Suppose:

    a = A
    b = B


STEP 1:

    a = a ^ b

Now:

    a = A ^ B
    b = B


STEP 2:

    b = b ^ a

    b = B ^ (A ^ B)

Rearrange:

    b = B ^ B ^ A

    b = 0 ^ A

    b = A


STEP 3:

    a = a ^ b

    a = (A ^ B) ^ A

    a = B


Therefore:

    a = B
    b = A


===========================================================
IMPLEMENTATION
===========================================================
*/

static void xor_swap(uint32_t *a, uint32_t *b)
{
    /*
     * IMPORTANT:
     *
     * This only works correctly if a and b point to
     * DIFFERENT storage locations.
     */

    *a ^= *b;
    *b ^= *a;
    *a ^= *b;
}


/*
===========================================================
THE ALIASING PROBLEM
===========================================================

This is the most important counter-question.

What happens if:

    a == b


Example:

    uint32_t x = 10;

    xor_swap(&x, &x);


STEP 1:

    x ^= x;

Therefore:

    x = 0


STEP 2:

    x ^= x;

Still:

    x = 0


STEP 3:

    x ^= x;

Still:

    x = 0


The original value is destroyed.


Therefore:

    XOR swap does NOT work safely when both references
    refer to the same storage location.


===========================================================
NORMAL TEMPORARY VARIABLE
===========================================================
*/

static void normal_swap(uint32_t *a, uint32_t *b)
{
    /*
     * This is much easier to understand.
     */
    uint32_t temp = *a;

    *a = *b;
    *b = temp;
}


/*
===========================================================
WHICH SHOULD YOU USE?
===========================================================

In normal production C code:

    temp swap

is usually preferable.


Why?

    - Easier to read
    - Easier to maintain
    - Compiler can optimize it
    - Avoids aliasing issue
    - No practical reason to avoid a register temporary


Modern compilers are very good at register allocation.

The temporary variable does NOT necessarily mean an
expensive memory access.


===========================================================
IS XOR SWAP ATOMIC?
===========================================================

NO.


This:

    a ^= b;
    b ^= a;
    a ^= b;


is three separate operations.


An interrupt can occur between them.

An ISR or another concurrent execution context could observe
a partially modified state.


Therefore XOR swap should NOT be considered atomic.


If shared data must be protected, use the appropriate
synchronization/atomic mechanism.


===========================================================
ISR EXAMPLE
===========================================================

Suppose:

    main code:
        a ^= b;
        b ^= a;
        a ^= b;


and an ISR reads:

    a
    b


The ISR could execute after the first instruction.

It may observe:

    a = A ^ B
    b = B


This is an intermediate state, not the intended final state.


Therefore XOR swap does not provide atomicity.


===========================================================
PERFORMANCE
===========================================================

XOR swap requires:

    3 XOR operations


Normal swap can be:

    load a
    load b
    store a
    store b


The compiler may optimize a temporary-based swap extremely
well.

Therefore:

    "XOR swap uses fewer variables"

does NOT automatically mean:

    "XOR swap is faster."


===========================================================
INTERVIEW ANSWER
===========================================================

"XOR swap works because XOR is reversible: applying the same
XOR operation again recovers the original value. However,
it fails when both operands alias the same memory location,
and the three operations are not atomic. In production code
I'd normally use a temporary variable because it is clearer
and modern compilers optimize it well."
*/
```

#### 45. Bit-level testing framework: verify register behavior.
    - Counter: How to write testable register abstractions?
    - Counter: Mock hardware register interface?
    - Counter: Unit test register bit operations?

```c++
#include <stdint.h>
#include <stdbool.h>

/*
===========================================================
Question 45
Bit-Level Testing Framework
===========================================================

GOAL
----

We want to test code that manipulates hardware registers
without requiring real hardware for every unit test.


The key technique:

    Separate register logic
            from
    hardware access


Instead of scattering:

    HW_REGISTER |= BIT_X;


throughout the application, create a small abstraction.


===========================================================
PROBLEM WITH DIRECT REGISTER ACCESS
===========================================================

Code like:

    REG->CTRL |= (1U << 4);

is tightly coupled to hardware.


Unit testing it on a PC is difficult because:

    - REG may not exist.
    - Memory-mapped addresses are invalid.
    - Hardware behavior cannot easily be simulated.


Instead, separate:

    "What should the register contain?"

from:

    "How do I physically access the register?"


===========================================================
REGISTER INTERFACE
===========================================================
*/

typedef struct
{
    /*
     * Function used to read the register.
     */
    uint32_t (*read)(void *context);

    /*
     * Function used to write the register.
     */
    void (*write)(void *context, uint32_t value);

    /*
     * Object containing the hardware/mock state.
     */
    void *context;

} RegisterInterface;


/*
===========================================================
GENERIC BIT OPERATIONS
===========================================================
*/

static void register_set_bits(RegisterInterface *reg,
                               uint32_t mask)
{
    /*
     * Read current value.
     */
    uint32_t value = reg->read(reg->context);

    /*
     * Set selected bits.
     */
    value |= mask;

    /*
     * Write modified value.
     */
    reg->write(reg->context, value);
}


static void register_clear_bits(RegisterInterface *reg,
                                 uint32_t mask)
{
    uint32_t value = reg->read(reg->context);

    /*
     * Clear selected bits.
     */
    value &= ~mask;

    reg->write(reg->context, value);
}


static void register_update_bits(RegisterInterface *reg,
                                  uint32_t mask,
                                  uint32_t value)
{
    uint32_t current = reg->read(reg->context);

    /*
     * Clear the bits represented by mask.
     */
    current &= ~mask;

    /*
     * Insert the new value.
     */
    current |= (value & mask);

    reg->write(reg->context, current);
}


/*
===========================================================
WHY IS THIS TESTABLE?
===========================================================

We can now replace the real hardware with a mock register.

Instead of:

    actual MCU register

we use:

    uint32_t mock_register;


The production code doesn't need to know the difference.


===========================================================
MOCK REGISTER
===========================================================
*/

typedef struct
{
    uint32_t value;

} MockRegister;


/*
===========================================================
MOCK READ
===========================================================
*/

static uint32_t mock_read(void *context)
{
    MockRegister *mock = (MockRegister *)context;

    return mock->value;
}


/*
===========================================================
MOCK WRITE
===========================================================
*/

static void mock_write(void *context, uint32_t value)
{
    MockRegister *mock = (MockRegister *)context;

    mock->value = value;
}


/*
===========================================================
UNIT TEST EXAMPLE
===========================================================
*/

static bool test_set_bits(void)
{
    /*
     * Create fake hardware register.
     */
    MockRegister mock =
    {
        .value = 0x00000000U
    };

    /*
     * Connect the mock to our generic register interface.
     */
    RegisterInterface reg =
    {
        .read = mock_read,
        .write = mock_write,
        .context = &mock
    };

    /*
     * Set bit 3.
     */
    register_set_bits(&reg, (1U << 3U));

    /*
     * Expected:
     *
     *     00001000
     */
    return mock.value == 0x00000008U;
}


/*
===========================================================
UNIT TEST: CLEAR BITS
===========================================================
*/

static bool test_clear_bits(void)
{
    MockRegister mock =
    {
        .value = 0x000000FFU
    };

    RegisterInterface reg =
    {
        .read = mock_read,
        .write = mock_write,
        .context = &mock
    };

    /*
     * Clear bits 4-7.
     *
     * Mask:
     *
     *     11110000
     */
    register_clear_bits(&reg, 0xF0U);

    /*
     * Expected:
     *
     *     00001111
     */
    return mock.value == 0x0FU;
}


/*
===========================================================
UNIT TEST: UPDATE FIELD
===========================================================

Suppose a register contains:

    bits 4-7 = MODE


We want:

    MODE = 5

Mask:

    0xF0


Value:

    5 << 4

    = 0x50


===========================================================
*/

static bool test_update_field(void)
{
    MockRegister mock =
    {
        .value = 0x00U
    };

    RegisterInterface reg =
    {
        .read = mock_read,
        .write = mock_write,
        .context = &mock
    };

    /*
     * Update bits 4-7 to 5.
     */
    register_update_bits(
        &reg,
        0xF0U,
        0x50U
    );

    /*
     * Expected:
     *
     *     0101 0000
     *
     * = 0x50
     */
    return mock.value == 0x50U;
}


/*
===========================================================
TESTING RESERVED BITS
===========================================================

Suppose:

    bits 0-3  = MODE
    bits 4-7  = SPEED
    bits 8-31 = RESERVED


We should NOT allow generic code to modify reserved bits.


Define:

    VALID_MASK = 0x000000FF


Tests can verify:

    only valid bits changed.


===========================================================
TESTING READ-ONLY BITS
===========================================================

A mock can simulate hardware behavior.

Example:

    Register:

        bits 0-3 = RW
        bits 4-7 = RO


The mock write function can preserve RO bits.


Conceptually:


    new_value =
        (written_value & RW_MASK)
        |
        (old_value & RO_MASK);


This allows unit tests to verify that software does not
accidentally assume all bits are writable.


===========================================================
REGISTER TEST CATEGORIES
===========================================================

A good register test suite should test:

    1. Reset value

    2. Set individual bit

    3. Clear individual bit

    4. Set multiple bits

    5. Clear multiple bits

    6. Field update

    7. Field boundary values

    8. Reserved bits

    9. Read-only bits

    10. Write-one-to-clear behavior

    11. Write-one-to-set behavior

    12. Invalid values

    13. Reset/reinitialization behavior


===========================================================
BIT TEST EXAMPLE
===========================================================
*/

static bool test_individual_bits(void)
{
    MockRegister mock =
    {
        .value = 0U
    };

    RegisterInterface reg =
    {
        .read = mock_read,
        .write = mock_write,
        .context = &mock
    };

    /*
     * Test every bit position.
     */
    for (uint32_t bit = 0U; bit < 32U; ++bit)
    {
        uint32_t mask = 1UL << bit;

        /*
         * Clear register first.
         */
        mock.value = 0U;

        /*
         * Set one bit.
         */
        register_set_bits(&reg, mask);

        /*
         * Exactly that bit should be set.
         */
        if (mock.value != mask)
        {
            return false;
        }

        /*
         * Clear that bit.
         */
        register_clear_bits(&reg, mask);

        /*
         * Register should return to zero.
         */
        if (mock.value != 0U)
        {
            return false;
        }
    }

    return true;
}


/*
===========================================================
WHY MOCK HARDWARE?
===========================================================

Without a mock:

    Unit test
       ↓
    real MCU
       ↓
    real register
       ↓
    hardware required


With a mock:

    Unit test
       ↓
    fake register
       ↓
    deterministic result


Advantages:

    - Fast tests
    - Runs on PC/CI
    - No hardware required
    - Repeatable
    - Easy fault injection


===========================================================
FAULT INJECTION
===========================================================

A good mock can intentionally simulate bad hardware.


Example:

    read returns unexpected value.

Or:

    write silently fails.

Or:

    one bit is stuck at zero.


This allows testing recovery code without physically
breaking hardware.


===========================================================
IMPORTANT EMBEDDED ARCHITECTURE
===========================================================

A useful architecture is:


Application
    |
    v
Register abstraction
    |
    +------------------+
    |                  |
    v                  v
Real hardware       Mock hardware
    |                  |
    v                  v
MCU register        RAM variable


The application logic stays unchanged.


===========================================================
INTERVIEW ANSWER
===========================================================

"How would you unit-test register bit manipulation?"

Answer:

    "I would abstract register reads and writes behind a
     small interface. Production uses the memory-mapped
     register, while unit tests inject a mock register
     implementation. Then I can test set, clear, field
     update, reset values, reserved bits and special access
     semantics entirely on the host."


===========================================================
IMPORTANT DISTINCTION
===========================================================

UNIT TEST:

    Tests software logic.

    Example:

        mask operation is correct.


HARDWARE TEST:

    Tests actual hardware behavior.

    Example:

        writing a register actually changes the intended
        hardware bits.


INTEGRATION TEST:

    Tests software + actual peripheral together.


You generally need all three at different stages.


===========================================================
FINAL TESTING PYRAMID
===========================================================

                System Test
                     |
             Integration Test
                     |
              Hardware Test
                     |
                 Unit Test
                     |
              Static Analysis


Unit tests:

    Fast
    Many
    Run in CI


Hardware tests:

    Slower
    Require target hardware


System tests:

    Slowest
    Validate end-to-end behavior


===========================================================
FINAL INTERVIEW MEMORY TRICK
===========================================================

For register testing:

    ABSTRACT
       ↓
    MOCK
       ↓
    TEST
       ↓
    INTEGRATE
       ↓
    VERIFY ON HARDWARE


And for individual bit operations:

    SET:
        reg |= mask;

    CLEAR:
        reg &= ~mask;

    TOGGLE:
        reg ^= mask;

    TEST:
        if (reg & mask)


For a field:

    CLEAR FIELD:
        reg &= ~mask;

    INSERT FIELD:
        reg |= (value << shift) & mask;
*/
```
---
