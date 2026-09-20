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

33. Generate test patterns: alternating bits (0xAAAA, 0x5555).
    - Counter: Why used for hardware testing?
    - Counter: Walking ones/zeros pattern?
    - Counter: Marching bit pattern for memory test?

34. Bit-packing compression: store multiple small values in one register.
    - Counter: What if values have different bit widths?
    - Counter: Alignment and padding overhead?
    - Counter: Endianness considerations?

35. Find Hamming distance (bit difference) between two values.
    - Counter: XOR then popcount?
    - Counter: Application: error detection?
    - Counter: Used in some cyclic codes?

---

### EXPERT

36. Implement Parallel Bit Count (SWAR — SIMD Within A Register).
    - Counter: Process multiple bit counts in parallel?
    - Counter: Algorithm uses shifts and masks cleverly?
    - Counter: When is this worth the complexity?

37. De Bruijn sequence for bit position indexing.
    - Counter: Why used for fast bit position lookup?
    - Counter: Generate De Bruijn sequence for powers of 2?
    - Counter: Space-time tradeoff: 32 bytes table vs log operations?

38. Bit manipulation for fixed-point math.
    - Counter: Multiply and keep fractional part (>>16 for 16.16 fixed)?
    - Counter: Rounding modes: truncate vs round-to-nearest?
    - Counter: Overflow prevention?

39. Gosper's Hack: next permutation of bits.
    - Counter: What does this generate?
    - Counter: Used for efficient subset enumeration?
    - Counter: Real-time constraints: single operation?

40. Low-level hardware test: verify register read-write.
    - Counter: Write patterns and read back?
    - Counter: Handle read-only bits?
    - Counter: Detect stuck-at faults?

41. Register shadowing for protection against corruption.
    - Counter: Maintain shadow copy in RAM?
    - Counter: When to check: every access or periodic?
    - Counter: Recovery strategy?

42. Bit rotation for CAN ID matching (extended vs standard).
    - Counter: Filter match in hardware vs firmware?
    - Counter: Performance: software filtering bottleneck?
    - Counter: Which approach for automotive safety?

43. Detect multiple errors: more than one bit flipped.
    - Counter: Hamming code limitations?
    - Counter: SECDED (single error correct, double error detect)?
    - Counter: Overkill for non-critical data?

44. Optimized XOR-based swap without temp variable.
    - Counter: a ^= b; b ^= a; a ^= b — does it work always?
    - Counter: Atomic operations: is this safe in ISR?
    - Counter: When not to use: performance penalty vs clarity?

45. Bit-level testing framework: verify register behavior.
    - Counter: How to write testable register abstractions?
    - Counter: Mock hardware register interface?
    - Counter: Unit test register bit operations?

---

## 2. FIXED-SIZE ARRAYS & STACK OPERATIONS

### BEGINNER

51. Find maximum in fixed-size array without dynamic allocation.
    - Counter: Stack-allocated array vs heap?
    - Counter: Time complexity: O(n) guaranteed on embedded?
    - Counter: How to handle empty array safely?

52. Reverse array in-place with limited stack space.
    - Counter: Only 4-8 bytes of extra variables allowed?
    - Counter: What if array size is unknown at compile-time?
    - Counter: Verify in-place doesn't cause undefined behavior?

53. Linear search with early termination.
    - Counter: Cache efficiency on Harvard architecture?
    - Counter: Predictable timing for real-time?
    - Counter: Sentinel value optimization?

54. Copy array with size bounds checking.
    - Counter: Prevent buffer overflow?
    - Counter: What if source == destination (overlapping)?
    - Counter: Use memcpy() or manual loop?

55. Find index of maximum value.
    - Counter: Return first or last occurrence?
    - Counter: What if array is empty?
    - Counter: Can you eliminate branching?

56. Check if array contains specific value (presence).
    - Counter: Time limit for real-time system?
    - Counter: Cache-friendly search order?
    - Counter: Early exit optimization?

57. Count occurrences of element in array.
    - Counter: O(n) time, O(1) space always?
    - Counter: Integer overflow: count > 2^32?
    - Counter: How to make this atomic for thread safety?

58. Initialize fixed array with pattern (zeros, ones, alternating).
    - Counter: memset() vs loop vs compiler optimizations?
    - Counter: Volatile vs non-volatile data?
    - Counter: Size-specific optimizations (8/16/32-bit elements)?

59. Find two elements that sum to target.
    - Counter: Sorted array vs unsorted?
    - Counter: O(n) vs O(n log n) space constraint?
    - Counter: Multiple pairs: find all or just one?

60. Shift array elements (rotate/slide) without extra storage.
    - Counter: Left shift by K positions?
    - Counter: Right shift (rotate)?
    - Counter: Minimum number of swaps needed?

---

## 3. RING BUFFERS & CIRCULAR QUEUES

### BEGINNER

101. Implement ring buffer for UART receive.
    - Counter: Fixed array, head, tail pointers?
    - Counter: Full vs empty detection: reserved bit or counter?
    - Counter: Thread-safe without locks?

102. Check if ring buffer is empty.
    - Counter: head == tail condition?
    - Counter: Need to handle wrap-around?
    - Counter: What if not initialized?

103. Calculate space available in ring buffer.
    - Counter: (tail - head - 1) mod size?
    - Counter: Integer overflow risk?
    - Counter: Check before enqueue operation?

104. Enqueue element into ring buffer.
    - Counter: Check full before insert?
    - Counter: Overwrite oldest if full?
    - Counter: Update tail pointer safely?

105. Dequeue element from ring buffer.
    - Counter: Check empty before dequeue?
    - Counter: What if no data available?
    - Counter: Blocking vs non-blocking behavior?

106. Ring buffer for real-time sensor data.
    - Counter: What if samples arrive faster than processing?
    - Counter: Overwrite or drop policy?
    - Counter: Average latency guarantee?

107. Peek at next element without removing.
    - Counter: Useful for protocol parsing?
    - Counter: Can you peek multiple elements ahead?
    - Counter: Thread-safe peeking?

108. Clear/flush ring buffer without deallocation.
    - Counter: Reset head and tail only?
    - Counter: Scrub data for security?
    - Counter: What if currently being read from ISR?

---

### INTERMEDIATE

116. Multi-threaded ring buffer (producer-consumer).
    - Counter: Spinlock vs condition variable?
    - Counter: Priority inversion with multiple producers?
    - Counter: Lock-free implementation possible?

117. Ring buffer with variable-length messages.
    - Counter: How to store message length prefix?
    - Counter: Handle partial message (ISR interrupted)?
    - Counter: Memory waste with padding?

118. Ring buffer for DMA-based UART.
    - Counter: DMA writes to ring buffer directly?
    - Counter: Head pointer: who updates (ISR or main)?
    - Counter: Synchronization between DMA and CPU?

119. Detect overflow in ring buffer (dropped data).
    - Counter: Counter for overflow count?
    - Counter: Set flag and continue, or halt?
    - Counter: Recoverable vs fatal overflow?

120. Ring buffer with timestamp on enqueue.
    - Counter: Include timestamp in each element?
    - Counter: Time synchronization across systems?
    - Counter: Fixed overhead per message?

---

## 4. STATE MACHINES & FINITE AUTOMATA

### BEGINNER

146. Simple LED blink state machine (on/off).
    - Counter: How to implement state transitions?
    - Counter: Timer tick drives transitions?
    - Counter: Can you reduce states?

147. Button debounce as state machine.
    - Counter: States: released, pressed, held, released_check?
    - Counter: How many timer ticks to debounce (e.g., 20ms)?
    - Counter: What if button glitches during hold?

148. Protocol parser state machine (detect packet structure).
    - Counter: States for sync, header, payload, checksum?
    - Counter: Invalid state recovery?
    - Counter: Timeout handling?

149. Traffic light controller (Red → Green → Yellow → Red).
    - Counter: Time in each state?
    - Counter: Emergency vehicle override?
    - Counter: State diagram clarity?

150. Power mode state machine (active/sleep/deep sleep).
    - Counter: Wake-up events trigger transitions?
    - Counter: State entry/exit code for setup?
    - Counter: Energy consumption in each state?

151. Stepper motor control state machine.
    - Counter: States for each coil activation sequence?
    - Counter: Forward and reverse stepping?
    - Counter: Acceleration/deceleration ramp?

152. Temperature controller state machine.
    - Counter: States: heating, cooling, idle?
    - Counter: Hysteresis to prevent oscillation?
    - Counter: Fault detection state?

153. Encryption key derivation state machine.
    - Counter: States for key generation steps?
    - Counter: Ensure all states must execute in order?
    - Counter: Prevent skip to final state?

154. Communication protocol handshake (ACK/NACK/RETRY).
    - Counter: States for waiting, transmission, acknowledgment?
    - Counter: Retry counter and timeout?
    - Counter: Recovery from unexpected message?

155. File system state machine (unmounted/mounted/busy).
    - Counter: State transitions on mount/unmount/error?
    - Counter: What happens in invalid state transitions?
    - Counter: Stuck state detection?

---

### INTERMEDIATE

161. Mealy vs Moore state machine: which to use?
    - Counter: Output on transition vs state?
    - Counter: Performance difference?
    - Counter: Which is easier for protocol parsing?

162. Hierarchical state machine (nested states).
    - Counter: Parent state for common transitions?
    - Counter: Entry/exit actions for super-states?
    - Counter: Implementation without recursion?

163. Guard conditions: execute transition only if condition true.
    - Counter: Prevent invalid transitions?
    - Counter: What if guard fails in critical section?
    - Counter: Can you have multiple guards per transition?

164. Timeout-based state transitions.
    - Counter: How long in each state before auto-transition?
    - Counter: Timer management for multiple states?
    - Counter: Reset timer on entering state?

165. Parallel state machines (multiple independent FSMs).
    - Counter: Synchronize outputs from multiple FSMs?
    - Counter: Resource contention between state machines?
    - Counter: Timing guarantee across all FSMs?

166. Event-driven state machine vs time-driven.
    - Counter: Interrupt on event or polled at fixed rate?
    - Counter: Responsiveness vs determinism?
    - Counter: Which is safer for safety-critical?

167. State history: remember previous state for recovery.
    - Counter: Return to last valid state on error?
    - Counter: What if error occurred in middle of transition?
    - Counter: Persistent history (in EEPROM)?

168. Conditional state entry (entry guards).
    - Counter: Check preconditions before entering state?
    - Counter: Reject entry and stay in previous state?
    - Counter: Log rejected transitions?

---

## 5. REAL-TIME SCHEDULING ALGORITHMS

### BEGINNER

191. Implement rate monotonic scheduling analysis.
    - Counter: Which task has highest priority (shortest period)?
    - Counter: Can all tasks meet deadlines?
    - Counter: CPU utilization threshold?

192. Preemptive scheduling: task priorities.
    - Counter: How to switch context between tasks?
    - Counter: What if two tasks have same priority?
    - Counter: Priority inversion risk?

193. Round-robin scheduling with time quanta.
    - Counter: Fixed time slice per task?
    - Counter: How to handle blocked (waiting) tasks?
    - Counter: Fairness vs responsiveness?

194. Deadline calculation: current time + offset.
    - Counter: 32-bit timer overflow handling?
    - Counter: Precision: millisecond or microsecond?
    - Counter: Atomic time read?

195. Task activation: trigger task at specific time.
    - Counter: Early vs late activation penalty?
    - Counter: Can task be activated multiple times?
    - Counter: Queue of pending activations?

---

### INTERMEDIATE

206. Priority ceiling protocol to prevent priority inversion.
    - Counter: Boost task priority during critical section?
    - Counter: How to determine ceiling priority?
    - Counter: Restore priority after release?

207. Earliest deadline first (EDF) scheduling.
    - Counter: Which task runs next (nearest deadline)?
    - Counter: Optimal for uniprocessor systems?
    - Counter: Implementation complexity vs RMS?

208. Processor frequency scaling based on load.
    - Counter: Lower frequency = lower power?
    - Counter: When to scale up/down (hysteresis)?
    - Counter: Deadline guarantee maintained?

---

## 6. MEMORY MANAGEMENT & POOLING

### BEGINNER

241. Static memory pool: pre-allocate fixed-size blocks.
    - Counter: Why use pool instead of malloc?
    - Counter: How many blocks needed?
    - Counter: What if pool exhausted?

242. Free list management in memory pool.
    - Counter: Linked list of free blocks?
    - Counter: Allocation: pop from free list?
    - Counter: Deallocation: push back to free list?

243. Check available memory without fragmentation.
    - Counter: Only possibility if using fixed-size blocks?
    - Counter: What if some blocks in use?
    - Counter: Percentage free calculation?

244. Initialize memory pool at startup.
    - Counter: Compile-time vs runtime initialization?
    - Counter: Verify no usage before initialization?
    - Counter: Link all free blocks together?

245. Detect memory corruption: guard bytes around blocks.
    - Counter: Canary values before/after data?
    - Counter: Check on allocation/deallocation?
    - Counter: Where to store metadata?

---

## 7. INTERRUPT-SAFE PATTERNS

### BEGINNER

286. Atomic read of shared variable (volatile).
    - Counter: Read-modify-write race condition?
    - Counter: Use volatile keyword?
    - Counter: Disable interrupts vs atomic instructions?

287. Increment shared counter in ISR and main.
    - Counter: Is ++ atomic on 8-bit MCU?
    - Counter: Disable IRQ during increment in main?
    - Counter: Performance cost of synchronization?

288. Queue access from ISR and main task.
    - Counter: Enqueue in ISR, dequeue in main?
    - Counter: No locks allowed (lock-free)?
    - Counter: What if queue full/empty?

289. Flag set in ISR, checked in main.
    - Counter: Volatile flag necessary?
    - Counter: Can main loop check immediately after ISR?
    - Counter: Memory barrier needed?

290. Critical section protection without locks.
    - Counter: Disable interrupts locally?
    - Counter: Nested interrupt disabling (save/restore)?
    - Counter: Timing impact on ISR latency?

---

## 8. SENSOR DATA PROCESSING

### BEGINNER

331. Read ADC value and convert to physical units.
    - Counter: Scaling factor calculation?
    - Counter: 10-bit vs 12-bit ADC?
    - Counter: Temperature sensor correction?

332. Moving average filter for noisy sensor.
    - Counter: Window size choice?
    - Counter: Fixed-size buffer for last N samples?
    - Counter: Integer arithmetic to avoid floating point?

333. Median filter for outlier rejection.
    - Counter: Why better than mean for outliers?
    - Counter: Sort 3-5 values to find median?
    - Counter: Implementation without sorting library?

334. Detect sensor fault (value out of range).
    - Counter: Min/max thresholds?
    - Counter: Consecutive fault readings trigger alarm?
    - Counter: Hysteresis to prevent noise-triggered alarms?

335. Calibration offset correction.
    - Counter: Stored in EEPROM?
    - Counter: Apply offset to raw reading?
    - Counter: Recalibration procedure?

---

## 9. PROTOCOL PARSING

### BEGINNER

381. Parse UART packet: detect start/stop bytes.
    - Counter: Framing error handling?
    - Counter: Maximum packet size?
    - Counter: Timeout if start detected but no stop?

382. CRC-8 checksum validation.
    - Counter: Polynomial for CRC-8?
    - Counter: Bit-by-bit vs lookup table?
    - Counter: Performance critical?

383. Extract field from CAN message (11-bit ID, 8-byte data).
    - Counter: Bit shifting for field extraction?
    - Counter: Byte order in CAN (big-endian)?
    - Counter: Validate ID range?

384. Decode variable-length encoded message.
    - Counter: Length prefix indicates payload size?
    - Counter: Handle fragmentation across multiple buffers?
    - Counter: Memory-efficient assembly?

385. State machine for Modbus packet parsing.
    - Counter: Slave ID, function code, data, CRC?
    - Counter: Validate before processing?
    - Counter: Timeout if incomplete?

---

## 10. TIME-CRITICAL ALGORITHMS

### BEGINNER

431. Implement timer ISR callback without context switch.
    - Counter: Minimal time in ISR?
    - Counter: Defer work to main loop?
    - Counter: Flag vs callback function pointer?

432. Measure execution time of critical code section.
    - Counter: Use timer counter at start/end?
    - Counter: Account for timer wraparound?
    - Counter: Overhead of time measurement itself?

433. Schedule periodic task with millisecond precision.
    - Counter: Can timer ISR guarantee < 1ms jitter?
    - Counter: What if OS scheduler not available?
    - Counter: Busy-wait vs interrupt-driven?

434. Prevent stack overflow from deep recursion.
    - Counter: Check stack pointer before recursion?
    - Counter: Iterative approach preferable?
    - Counter: Maximum recursion depth calculation?

435. Implement watchdog timer reset logic.
    - Counter: Where to call reset function (in ISR or main)?
    - Counter: If reset not called, system reboots?
    - Counter: Window watchdog (both too fast and too slow reset)?

---

## 11. LOW-POWER OPTIMIZATION

### BEGINNER

436. Configure MCU sleep mode (idle vs sleep vs deep sleep).
    - Counter: Which peripherals disabled in each mode?
    - Counter: Wake-up latency?
    - Counter: Power consumption reduction factor?

437. Reduce power consumption by lowering clock frequency.
    - Counter: Performance impact?
    - Counter: Maintain real-time guarantees?
    - Counter: Dynamic voltage and frequency scaling (DVFS)?

438. Disable unused peripherals (timers, ADC, UART).
    - Counter: Power savings per peripheral?
    - Counter: Re-enable before use?
    - Counter: Initialization overhead?

439. Schedule intensive computation in low-power period.
    - Counter: WiFi not transmitting?
    - Counter: Batch processing vs continuous?
    - Counter: User interaction requirements?

440. Implement low-power sensor polling.
    - Counter: Read sensor every N seconds?
    - Counter: Wake from sleep, read, sleep again?
    - Counter: Wake-up latency acceptable?

---

## 12. FIRMWARE DATA STRUCTURES

### BEGINNER

441. Implement linked list in fixed memory block.
    - Counter: Array of nodes with next pointers?
    - Counter: No dynamic allocation?
    - Counter: Performance vs tree?

442. Circular doubly-linked list for cache LRU eviction.
    - Counter: Move accessed node to front?
    - Counter: Evict tail node on full?
    - Counter: Pointer overhead?

443. Priority queue using array (binary heap).
    - Counter: Parent at index n/2, children at 2n and 2n+1?
    - Counter: Heapify operations (up/down)?
    - Counter: Insert and extract minimum?

444. Hash table with linear probing (no dynamic memory).
    - Counter: Fixed size table?
    - Counter: Collision resolution?
    - Counter: Load factor threshold?

445. Bitmap for presence tracking (is element in set).
    - Counter: One bit per element?
    - Counter: Space-efficient set membership?
    - Counter: Iterate over present elements?

---

## 13. HARDWARE ABSTRACTION LAYERS (HAL)

### INTERMEDIATE

326. GPIO abstraction: read/write pin value.
    - Counter: Handle different MCU pin layouts?
    - Counter: Inline vs function call for speed?
    - Counter: Register memory-mapped or SPI-accessible?

327. Timer HAL: configure prescaler and reload value.
    - Counter: Different timer frequencies on different MCUs?
    - Counter: Prevent off-by-one errors in calculations?
    - Counter: Support different timer modes?

328. UART HAL: initialize, transmit, receive.
    - Counter: Baud rate generation formula?
    - Counter: Interrupt vs polling driven?
    - Counter: Flow control (hardware or software)?

329. ADC HAL: start conversion, read result.
    - Counter: Single shot vs continuous conversion?
    - Counter: Channel selection and sequencing?
    - Counter: Sample-and-hold time?

330. PWM HAL: set frequency and duty cycle.
    - Counter: Frequency vs period register?
    - Counter: Dead time insertion for H-bridge?
    - Counter: Complementary PWM channels?

---

## 14. BOOTLOADER & INITIALIZATION

### INTERMEDIATE

336. Bootloader recovery: detect valid firmware image.
    - Counter: CRC or signature validation?
    - Counter: Rollback if current firmware corrupted?
    - Counter: Over-the-air update mechanism?

337. Initialize all peripherals at startup.
    - Counter: Order of initialization matters?
    - Counter: Dependencies between systems?
    - Counter: Self-test before declaring ready?

338. Memory layout: distinguish code, data, BSS.
    - Counter: Linker script definition?
    - Counter: Initialized vs uninitialized data?
    - Counter: Copy code from flash to RAM?

339. Vector table setup for interrupt handlers.
    - Counter: Each interrupt has entry in table?
    - Counter: Weak vs strong symbols for handlers?
    - Counter: Interrupt number to function mapping?

340. Clock configuration: set system frequency.
    - Counter: PLL lock time before use?
    - Counter: Divider chain calculations?
    - Counter: Check oscillator stability?

---

## 15. ERROR HANDLING & WATCHDOG

### INTERMEDIATE

341. Implement fault handler for CPU exceptions.
    - Counter: Hard fault vs mem management fault?
    - Counter: Extract PC/LR for post-mortem debugging?
    - Counter: Reboot or graceful shutdown?

342. Watchdog timer to detect firmware hang.
    - Counter: How often to reset watchdog?
    - Counter: Timeout value appropriate?
    - Counter: Independent from main clock?

343. Assert macro for development debugging.
    - Counter: Disable in production?
    - Counter: Message string storage?
    - Counter: Performance cost?

344. Error recovery: restart subsystem on failure.
    - Counter: Detect fault condition?
    - Counter: Graceful shutdown vs immediate reset?
    - Counter: Preserve error logs before reset?

345. Logging critical events to flash (ring buffer).
    - Counter: Limited flash write cycles?
    - Counter: Ring buffer to limit size?
    - Counter: Timestamp each log entry?

---

## 16. PWM & TIMER CONTROL

### INTERMEDIATE

346. Generate PWM signal at specific frequency and duty.
    - Counter: Frequency precision requirement?
    - Counter: Duty cycle resolution (8-bit vs 16-bit)?
    - Counter: Synchronize multiple PWM channels?

347. Motor speed control using PWM duty cycle feedback.
    - Counter: Proportional control?
    - Counter: Integrator windup avoidance?
    - Counter: Speed ramp for smooth acceleration?

348. Measure input frequency using timer capture.
    - Counter: Rising vs falling edge capture?
    - Counter: Overflow handling for slow signals?
    - Counter: Frequency calculation from timer ticks?

349. Input capture for measuring pulse width.
    - Counter: Start on rising edge, stop on falling?
    - Counter: Handle consecutive pulses?
    - Counter: Minimum/maximum pulse width validation?

350. Output compare mode for precise timing.
    - Counter: Generate pulse train at exact intervals?
    - Counter: Update compare value in ISR?
    - Counter: Phase-correct PWM?

---

## 17. ADC/DAC SIGNAL PROCESSING

### INTERMEDIATE

351. Oversample ADC and decimate to reduce noise.
    - Counter: 4x oversample reduces noise by sqrt(4)?
    - Counter: Digital decimation filter?
    - Counter: Computational cost?

352. Successive approximation ADC emulation in firmware.
    - Counter: Binary search for value?
    - Counter: DAC output vs input comparison?
    - Counter: Conversion time?

353. Thermocouple cold junction compensation.
    - Counter: Reference temperature sensor needed?
    - Counter: Polynomial or lookup table for linearization?
    - Counter: Accuracy impact?

354. DAC output smoothing with filter capacitor.
    - Counter: RC filter time constant?
    - Counter: Settling time before stable reading?
    - Counter: Update rate vs filtering?

355. Simultaneous sampling of multiple ADC channels.
    - Counter: Sample order?
    - Counter: Time between channels?
    - Counter: Cross-talk between channels?

---

## 18. COMMUNICATION PROTOCOLS

### INTERMEDIATE

356. I2C multi-byte read with repeated start condition.
    - Counter: Write address, read data, read more data?
    - Counter: ACK/NAK for each byte?
    - Counter: Stop or repeated start?

357. SPI master-slave communication.
    - Counter: CPOL and CPHA modes?
    - Counter: Chip select timing?
    - Counter: Full-duplex simultaneous TX/RX?

358. CAN message filtering and routing.
    - Counter: Accept specific IDs or ranges?
    - Counter: Masking for ID patterns?
    - Counter: Queue for processing?

359. Modbus protocol: master request, slave response.
    - Counter: Slave address, function code, CRC?
    - Counter: RTU vs ASCII mode?
    - Counter: Exception response?

360. LIN protocol: master task scheduling.
    - Counter: Fixed frame headers?
    - Counter: Slave transmit response?
    - Counter: Timeout handling?

---

## 19. CRYPTOGRAPHY IN EMBEDDED

### INTERMEDIATE

361. AES encryption in ECB mode (fixed 16-byte blocks).
    - Counter: Key expansion setup?
    - Counter: S-box lookup?
    - Counter: Why ECB unsafe (patterns visible)?

362. SHA-256 hash for firmware integrity.
    - Counter: Block processing on limited RAM?
    - Counter: Message padding?
    - Counter: Compare hash safely (timing attack)?

363. HMAC-SHA256 for authenticated messages.
    - Counter: Key derivation?
    - Counter: Inner and outer hash?
    - Counter: Time-constant comparison?

364. Random number generation for nonce/IV.
    - Counter: Seed from entropy source (ADC, thermal)?
    - Counter: Periodically reseed?
    - Counter: Cryptographic vs pseudo-random?

365. Elliptic curve signatures on embedded.
    - Counter: ECDSA signing?
    - Counter: Large number arithmetic?
    - Counter: Flash/RAM usage?

---

## 20. AUTOMOTIVE & INDUSTRIAL PROTOCOLS

### INTERMEDIATE

366. OBD-II parameter reading via CAN.
    - Counter: Mode and PID request?
    - Counter: Multi-frame response?
    - Counter: Unit conversion (km/h, etc)?

367. J1939 protocol for heavy-duty vehicles.
    - Counter: 29-bit identifier structure?
    - Counter: Priority and addressing?
    - Counter: Message assembly across CAN frames?

368. Time-triggered architecture (AUTOSAR).
    - Counter: Fixed-time scheduling?
    - Counter: Deterministic behavior?
    - Counter: Synchronization across nodes?

369. Fail-operational design: graceful degradation.
    - Counter: Detect failures?
    - Counter: Reduce functionality vs safe state?
    - Counter: Minimize downtime?

370. Safety-critical firmware: MISRA C compliance.
    - Counter: Forbidden language features?
    - Counter: Tool verification (static analysis)?
    - Counter: Code review checklist?

---

## 21-30. ADVANCED TOPICS (Level Advanced/Expert)

371-400: Advanced optimization techniques
- Function inlining for performance-critical code
- Loop unrolling vs code size explosion
- SIMD instruction usage (SSE, NEON)
- Cache line alignment for better performance
- Branch prediction and pipeline efficiency
- Compiler optimization levels (-O2, -O3, -Os)
- Hardware prefetching awareness
- Memory access patterns for cache efficiency

401-430: Real-time system design
- Preemption latency analysis
- Context switch overhead
- Interrupt nesting depth limits
- Task synchronization without priority inversion
- Bounded execution time proof
- Predictability vs optimality tradeoffs
- Multi-core synchronization primitives
- Lock-free data structure design

431-460: Safety and reliability
- Defensive programming: validate all inputs
- Graceful shutdown sequences
- Recovery procedures after failures
- Checksum/CRC for corruption detection
- EEPROM write cycle management
- Flash wear leveling algorithms
- Redundancy for critical systems
- Self-healing firmware mechanisms

461-500: Expert challenges
- Design a minimal RTOS kernel (task scheduling, context switch)
- Implement lock-free queue for producer-consumer
- Create memory profiler for embedded system
- Design robust bootloader with rollback capability
- Implement custom allocator optimized for fixed pools
- Multi-threaded filesystem on flash memory
- Real-time garbage collector with bounded pause
- Cryptographic protocol implementation (TLS subset)
- Debug protocol (GDB stub) implementation
- Kernel module for custom hardware driver

---

*Embedded developer interviews prioritize: Making it work in 4KB of RAM > theoretical complexity*
*If you understand WHEN to use assembly vs C — and can prove it matters — you are ready.*

---

## TESTING & VALIDATION STRATEGIES

### For All Difficulty Levels

- **Unit testing without framework**: How to test code in isolation?
- **Hardware-in-the-loop simulation**: Real hardware behavior mocking?
- **Timing analysis tools**: Profiling with oscilloscope/logic analyzer?
- **Code coverage metrics**: gcov for embedded?
- **Memory leak detection**: Limited tools on embedded?
- **Static analysis**: cppcheck, splint for common errors?
- **Fuzzing**: Generate invalid inputs to find crashes?
- **Regression testing**: Detect unintended changes?

---

## INTERVIEW TIPS FOR EMBEDDED DEVELOPERS

1. **Trade-offs are key**: No perfect solution, discuss what you're optimizing for
2. **Measure, don't guess**: Use actual hardware data
3. **Know your tools**: Compiler flags, linker scripts, debugger commands
4. **Edge cases matter**: What happens at wraparound, underflow, interrupt collision?
5. **Worst-case analysis**: Not average case or best case
6. **Architecture awareness**: Different on ARM Cortex-M, x86, RISC-V
7. **Real constraints**: 64KB flash, 8KB RAM is common
8. **Show your work**: Draw register layouts, timing diagrams, state machines
