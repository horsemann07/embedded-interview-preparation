# SPI (Serial Peripheral Interface) — Study Notes
## Protocol Basics, Data Transfer, Modes, Daisy Chaining, Pros/Cons, and I²C Comparison

> **Purpose:** These notes explain the main subjects covered in the EmbeTronicX SPI basics article in original wording, with additional diagrams and practical clarifications. They are not a verbatim copy.
>
> **Reference article:** [SPI – Serial Peripheral Interface Protocol Basics — EmbeTronicX](https://embetronicx.com/tutorials/tech_devices/spi-serial-peripheral-interface-protocol-basics/)  
> Article last updated by the publisher on July 15, 2023.

---

## 1. What is SPI?

**SPI (Serial Peripheral Interface)** is a synchronous serial communication interface commonly used by a microcontroller to exchange data with nearby peripherals. Typical devices include sensors, ADCs, DACs, displays, external flash memory, EEPROMs, real-time clocks, and SD-card interfaces.

- **Serial** means the bits travel one after another over a data line.
- **Synchronous** means the devices use a shared clock signal to time the transfer.
- SPI is commonly organized around a **controller/master** that supplies the clock and selects a **peripheral/slave**.
- SPI is usually **full-duplex** because the controller and peripheral have separate data lines for sending and receiving.
- SPI commonly supports high clock rates, but the maximum reliable speed depends on the controller, peripheral, board layout, wire length, and timing specifications.
- SPI is an interface family rather than one complete, universally fixed protocol. Devices can differ in word length, command format, bit order, chip-select behavior, and supported modes.

SPI was developed at Motorola in the 1980s and became widely used in embedded systems.

### A simple analogy

Think of the controller as the person coordinating a conversation. It supplies the beat (clock), chooses which peripheral should listen (chip select), and exchanges one bit at a time. The selected peripheral sends bits back at the same time.

---

## 2. SPI signals and wiring

A conventional four-wire SPI bus uses four signal types.

| Signal | Common name | Direction from controller's perspective | Purpose |
|---|---|---|---|
| Clock | `SCLK` / `SCK` | Controller → peripheral | Provides timing for each bit |
| Controller data out | `MOSI` | Controller → peripheral | Carries bits sent to the peripheral |
| Controller data in | `MISO` | Peripheral → controller | Carries bits returned to the controller |
| Chip select | `CS` / `SS` / `NSS` | Controller → peripheral | Selects the peripheral that should participate |

A peripheral-only device may label the data pins from its own perspective: `SDI` can mean serial data in and `SDO` serial data out. Always confirm pin names and directions in the device datasheet.

### Diagram: one controller and one peripheral

```text
                 SPI Controller                  SPI Peripheral
              +------------------+            +------------------+
MOSI -------->| Data OUT          |----------->| Data IN          |
MISO <--------| Data IN           |<-----------| Data OUT         |
SCLK -------->| Clock OUT         |----------->| Clock IN         |
CS   -------->| Peripheral select |----------->| Chip select      |
              +------------------+            +------------------+
```

`CS` is commonly **active-low**: the controller pulls it low to select the peripheral and returns it high when the transaction is over. This is common, but the device datasheet is authoritative.

### Multiple peripherals on one SPI bus

With the usual shared-bus arrangement, the clock and data lines are shared. Each peripheral normally gets its own chip-select signal.

```text
                         +-------------------+
              MOSI ----->|                   |---- MOSI shared ----+
              MISO <-----|   SPI Controller  |<--- MISO shared ----+----+
              SCLK ----->|                   |---- SCLK shared ----+    |
                         +-------------------+                         |
                            |       |       |                          |
                           CS0     CS1     CS2                         |
                            |       |       |                          |
                       +----v--+ +--v---+ +--v---+                     |
                       |Device0| |Device1| |Device2|<-------------------+
                       +-------+ +------+ +-------+
```

Only the intended peripheral should drive the shared MISO line during a transfer. Many peripherals place MISO in a high-impedance state when not selected, but this behavior must be confirmed for each device.

Adding peripherals can require additional chip-select GPIOs. An external decoder can reduce the number of controller pins needed for selection. Daisy chaining is another option, but only if the peripherals support it.

### Can SPI use fewer than four wires?

Some implementations use three-wire or bidirectional-data variants. Other pin-saving options may be possible in specific designs:

- With a single peripheral, the chip-select pin may sometimes be tied to its active level, **if the datasheet permits it**. Some devices require chip-select transitions to recognize commands.
- If the peripheral only receives data and never needs to return data, the MISO connection may not be needed.
- A decoder/demultiplexer can create multiple chip-select signals from fewer GPIOs.
- Daisy-chain mode can allow multiple compatible devices to share a chip-select line.

Do not assume that a connection-saving technique works for every SPI peripheral.

---

## 3. How an SPI transfer works

The controller configures the clock rate and mode, places the target peripheral in the selected state, and generates clock pulses. On each clock cycle, bits move in both directions:

- Controller sends a bit on MOSI.
- Peripheral reads that bit.
- Peripheral sends a bit on MISO.
- Controller reads that bit.

The same clock coordinates both directions. Therefore, even if the controller only wants to read data, it normally must generate clock pulses and transmit some value on MOSI (often a dummy byte). Likewise, a write-only operation still receives bits on MISO, even if the software ignores them.

SPI transfers are often organized into 8-bit bytes, but supported word sizes and framing are implementation-dependent. Data is frequently sent most-significant-bit first (MSB-first), although some devices allow least-significant-bit first (LSB-first).

### Diagram: simultaneous transfer

```text
Controller MOSI:   [ bit 7 ][ bit 6 ][ bit 5 ] ... [ bit 0 ]  --->
Peripheral MISO:   [ bit 7 ][ bit 6 ][ bit 5 ] ... [ bit 0 ]  <---

                    One clock pulse per transferred bit
```

### Typical transaction sequence

```text
1. Configure clock speed, CPOL, CPHA, and bit order
2. Set up the transmit/receive buffers
3. Assert the target peripheral's CS signal
4. Generate clock pulses and shift data in/out
5. Wait for the transfer to complete
6. Deassert CS as required by the device
7. Check status and interpret the received bytes
```

Some devices require command bytes, an address, dummy cycles, turnaround time, or multiple phases while CS stays asserted. Those rules come from the peripheral's datasheet, not from SPI alone.

### Shift-register concept

A simple way to picture the transfer is to imagine one shift register in the controller and one in the peripheral. With every active transfer edge, one bit leaves each register while a bit arrives from the other device. After eight clock cycles in an 8-bit transfer, each side may have exchanged one byte.

```text
Controller shift register  <---- bit exchange ---->  Peripheral shift register
          [ b7 ... b0 ]                             [ b7 ... b0 ]
```

This is a conceptual model. Real controller and peripheral hardware can include FIFOs, command engines, DMA, configurable word sizes, and other details.

---

## 4. SPI clock polarity and phase (CPOL and CPHA)

The controller and peripheral must agree on how the clock idles and which clock edge captures each data bit. If they use different settings, the receiver may sample data at the wrong moment and read incorrect bits.

Two settings determine the common SPI modes:

- **CPOL (Clock Polarity):** Defines the clock's idle level.
  - `CPOL = 0`: clock idles low.
  - `CPOL = 1`: clock idles high.
- **CPHA (Clock Phase):** Defines whether sampling occurs on the first or second clock edge of a cycle.
  - `CPHA = 0`: sample on the leading edge.
  - `CPHA = 1`: sample on the trailing edge.

The **leading edge** is the first transition away from the idle level. The **trailing edge** is the second transition, returning toward the idle level.

| SPI mode | CPOL | CPHA | Clock idle level | Sample edge | Other edge |
|---|---:|---:|---|---|---|
| Mode 0 | 0 | 0 | Low | First / rising | Falling |
| Mode 1 | 0 | 1 | Low | Second / falling | Rising |
| Mode 2 | 1 | 0 | High | First / falling | Rising |
| Mode 3 | 1 | 1 | High | Second / rising | Falling |

A common conceptual rule is: **CPHA determines whether the leading or trailing edge samples data; the other edge is used to shift or prepare the next bit.** Exact setup/hold timing still depends on the device specifications.

### Original timing sketches

The following sketches show the idle clock level and sample edge conceptually. They are not scale-accurate timing diagrams.

#### Mode 0 — CPOL = 0, CPHA = 0

- Clock idles low.
- The first edge is rising and is used to sample.
- Data should be ready before the rising sample edge; the opposite edge can be used to change data.

```text
SCLK  idle low:  ____/‾‾\____/‾‾\____
                    ^       ^
                  sample  sample
```

#### Mode 1 — CPOL = 0, CPHA = 1

- Clock idles low.
- The first edge is rising; data is shifted/prepared on this edge.
- Sampling occurs on the falling edge.

```text
SCLK  idle low:  ____/‾‾\____/‾‾\____
                    shift   shift
                       ^       ^
                    sample  sample
                    (falling edges)
```

#### Mode 2 — CPOL = 1, CPHA = 0

- Clock idles high.
- The first edge is falling and is used to sample.
- Data should be ready before the falling sample edge.

```text
SCLK  idle high: ‾‾‾‾\____/‾‾‾‾\____/‾‾
                     ^       ^
                   sample  sample
```

#### Mode 3 — CPOL = 1, CPHA = 1

- Clock idles high.
- The first edge is falling and is used to shift/prepare data.
- Sampling occurs on the rising edge.

```text
SCLK  idle high: ‾‾‾‾\____/‾‾‾‾\____/‾‾
                     shift   shift
                        ^       ^
                     sample  sample
                     (rising edges)
```

**Practical debugging tip:** If SPI communication returns incorrect or unstable data, confirm the peripheral's required SPI mode first. Also verify clock frequency, bit order, chip-select timing, voltage levels, pin muxing, wiring, and setup/hold requirements.

---

## 5. Daisy-chain SPI

In a conventional multi-peripheral bus, the controller uses separate chip-select signals to choose devices. In a **daisy chain**, compatible peripherals are connected in a sequence so that the serial output of one feeds the serial input of the next. The devices commonly share one chip-select line and a clock.

```text
Controller MOSI
      |
      v
+-------------+   serial out   +-------------+   serial out   +-------------+
| Peripheral 1|--------------->| Peripheral 2|--------------->| Peripheral 3|
| SI      SO  |                | SI      SO  |                | SI      SO  |
+-------------+                +-------------+                +-------------+
      ^                               ^                              |
      |________________ shared CS and SCLK _________________________|
                                                                     |
Controller MISO <----------------------------------------------------+
```

Pin names like `SI` and `SO` are illustrative. Actual devices may use different signal names. The key point is that data passes from one device to the next.

### Conceptual example with three 8-bit devices

Suppose a controller wants to load these values:

- Peripheral 1: `0xAA`
- Peripheral 2: `0xBB`
- Peripheral 3: `0xCC`

With a three-device 8-bit chain, the controller may need to clock three bytes (24 clock pulses) to move a new set of data through all three shift registers. The ordering of transmitted bytes depends on the device chain and its internal shift behavior, so the sequence must be checked against the device datasheet.

```text
Controller transmits 24 bits
        |
        v
[ byte for farthest device ][ byte for middle device ][ byte for nearest device ]
        |
        v
Data shifts through the chain as clock pulses arrive
```

This diagram describes a common shift-register style of chain; some peripherals latch outputs only on a separate latch/update event.

### Benefits and trade-offs of daisy chaining

**Potential benefits**
- Can reduce the number of individual chip-select GPIOs.
- Can be convenient for compatible shift-register-based peripherals or LED drivers.
- All devices can be updated using one shared selection signal in supported designs.

**Trade-offs**
- Every device must support a compatible chain mode.
- More clock cycles are usually needed as the chain gets longer.
- A failure or framing problem can affect data flowing through later devices.
- The byte order, chain length, latch behavior, and timing must be understood precisely.

Daisy chaining is **not** a universal SPI feature. Confirm support in every target device's datasheet before designing around it.

---

## 6. Advantages of SPI

- **High throughput:** SPI can often operate at higher clock rates than common I²C configurations, depending on device limits and board design.
- **Full-duplex data path:** MOSI and MISO allow data to travel in both directions during the same clock cycles.
- **Low protocol overhead:** Basic SPI framing does not require standard start bits, stop bits, or address bytes. A peripheral's command format may still add overhead.
- **Simple hardware model:** Basic peripherals can use shift registers and straightforward control logic.
- **Flexible data widths:** Transfers are not inherently limited to 8-bit values, although supported sizes vary by controller and peripheral.
- **Multiple peripherals:** Multiple devices can share clock and data lines, with chip-select signals or supported alternatives used for selection.

---

## 7. Disadvantages and limitations of SPI

- **Pin count:** The common four-wire arrangement needs clock, MOSI, MISO, and chip select. Each additional peripheral may need another CS signal unless a decoder or supported chain method is used.
- **No standard acknowledgment:** SPI itself does not define a universal ACK/NACK mechanism to confirm that a peripheral received a command correctly.
- **No built-in universal error checking:** CRC or other error detection must be provided by the device protocol or application if required.
- **Controller-managed traffic:** In a typical arrangement, the controller initiates transfers and supplies the clock. Peripherals cannot simply send bytes whenever they want without some system-specific mechanism.
- **No universal device-address format:** Selection is normally done through CS signals, not by sending an I²C-style address.
- **Short-distance board-level use:** SPI is most commonly used between devices on the same PCB or over short connections. Signal integrity becomes more difficult as wires get longer or clock rates increase.
- **Implementation differences:** Word sizes, mode support, chip-select behavior, command structure, and daisy-chain support vary among devices.

Multi-controller SPI arrangements can be built in some systems, but arbitration and bus ownership are not handled by one universally adopted SPI mechanism in the way many people expect from a complete bus protocol. A conventional embedded SPI design usually has one controller managing the bus.

---

## 8. Typical SPI applications

SPI is commonly used to connect microcontrollers or processors to peripherals such as:

- **Sensors:** Temperature, pressure, inertial sensors, touch controllers, and other measurement devices.
- **Data converters:** ADCs (analog-to-digital converters) and DACs (digital-to-analog converters).
- **Displays and drivers:** LCD controllers, OLED controllers, LED drivers, and related interface chips.
- **Memory:** Serial flash and EEPROM devices.
- **Timekeeping:** Real-time clock (RTC) peripherals.
- **Storage:** SD and MMC cards in SPI mode when the card and host support it.
- **Communication modules:** Some Ethernet controllers, wireless modules, and other interface chips use SPI as the host connection.
- **Control devices:** Audio codecs, digital potentiometers, and other programmable ICs.

SPI is the host-to-peripheral link in these examples; it does not mean that the peripheral's own higher-level communication protocol is SPI.

---

## 9. SPI vs I²C

SPI and I²C are both synchronous serial interfaces, but they make different trade-offs.

| Feature | SPI (typical arrangement) | I²C |
|---|---|---|
| Clock | Separate clock line driven by controller | Separate clock line driven by controller, with bus rules for multiple controllers |
| Data signals | Usually separate MOSI and MISO lines | Usually one shared bidirectional SDA line |
| Common signal count | 4 signals for one peripheral; additional CS lines may be needed | 2 bus signals: SCL and SDA |
| Duplex behavior | Usually full-duplex | Normally half-duplex on the shared data line |
| Device selection | Usually a separate CS signal per peripheral or an alternative such as a decoder/chain | Address field selects the target |
| Acknowledgment | No standard universal ACK/NACK | ACK/NACK is part of the bus transfer format |
| Throughput | Often higher for short local transfers, subject to device limits | Often lower than SPI, but depends on I²C speed mode and devices |
| Multi-controller support | Possible in some implementations, but not uniformly handled by a universal arbitration scheme | The I²C specification defines multi-controller arbitration |
| Hardware overhead | Simple streaming transfer model; selection pins add board/GPIO costs | Address, acknowledgment, and start/stop conditions add bus overhead |
| Common use | Fast links to nearby peripherals where extra signals are acceptable | Multiple slower peripherals sharing two wires is a common use |

### When might you choose SPI?

SPI may be a good choice when a peripheral requires high throughput, full-duplex transfers, or a simple clocked data stream, and the extra wiring is acceptable.

### When might you choose I²C?

I²C may be a good choice when several relatively low- or moderate-speed peripherals should share a two-wire bus and devices provide unique addresses.

Neither is always better. Use the target peripheral's supported interfaces, timing requirements, electrical characteristics, board layout, and system-level needs to make the choice.

---

## 10. Practical checklist for debugging SPI

When SPI does not work, check the following in a systematic order:

1. **Power and ground:** Confirm correct supply voltage and a common ground where required.
2. **Pin mapping:** Verify the MCU pin mux and that MOSI, MISO, SCLK, and CS are connected to the correct peripheral pins.
3. **Controller/peripheral roles:** Ensure one side supplies the clock and the other is configured to receive it.
4. **SPI mode:** Match CPOL and CPHA to the peripheral datasheet.
5. **Clock frequency:** Start at a conservative supported rate and increase only after communication is reliable.
6. **Bit order and word size:** Check MSB-first/LSB-first settings and the expected transfer width.
7. **Chip-select behavior:** Confirm active polarity, setup/hold time, whether CS must remain asserted across a command, and when it should return inactive.
8. **Read transactions:** Remember that the controller must generate clock pulses to receive data; it may need to transmit dummy bytes.
9. **MISO bus sharing:** Ensure unselected devices release MISO if the bus is shared.
10. **Data interpretation:** Check whether the received bytes include status, command responses, dummy bytes, or data that must be assembled into a larger value.
11. **Electrical quality:** If transfers fail at higher speeds, inspect signal integrity, edge rate, trace length, grounding, and timing with an oscilloscope or logic analyzer.
12. **Peripheral state:** Some devices need power-up delay, reset release, initialization commands, or a specific CS transition before responding.

### Example fault-isolation flow

```text
No valid SPI response
         |
         v
Check power, ground, pin mux, and wiring
         |
         v
Can you observe SCLK and CS?
      /       \
    No         Yes
    |           |
Check MCU       Is SPI mode and clock rate
configuration   correct for the peripheral?
                /               \
              No                 Yes
              |                   |
        Correct settings     Decode MOSI/MISO
                                  |
                                  v
                         Check command format,
                         dummy cycles, CS timing,
                         peripheral status, and
                         electrical signal quality
```

---

## Key takeaways

- SPI is a synchronous, commonly full-duplex interface for short-distance communication between a controller and peripherals.
- The main signals are SCLK, MOSI, MISO, and CS/SS.
- CPOL sets the idle clock level; CPHA selects which clock edge is used for sampling.
- A mismatch in SPI mode, clock rate, wiring, or CS timing is a common cause of communication failures.
- Daisy chaining can save chip-select signals, but only when the devices support it and the extra shift cycles are acceptable.
- SPI generally offers low framing overhead and high throughput, while I²C generally saves pins and provides addressing and acknowledgment.
- Always treat the exact peripheral datasheet as the final authority for supported modes, electrical limits, command sequences, timing, and chain behavior.

---

## Source

The source article that inspired these study notes is:

[EmbeTronicX — SPI: Serial Peripheral Interface Protocol Basics](https://embetronicx.com/tutorials/tech_devices/spi-serial-peripheral-interface-protocol-basics/)

These notes are an original educational summary with newly written explanations and diagrams, rather than a verbatim reproduction of the article.
