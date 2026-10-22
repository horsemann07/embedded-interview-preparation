# SPI (Serial Peripheral Interface)

SPI is a **synchronous serial communication interface** commonly used for short-distance, high-speed communication between a microcontroller and peripherals.

Typical devices:

```text
MCU
 │
 ├── SPI Flash
 ├── EEPROM
 ├── ADC
 ├── DAC
 ├── Display Controller
 ├── SD Card
 ├── IMU / Sensor
 └── RF / Communication Module
```

The main advantage of SPI is:

> **Simple protocol + high speed + full-duplex communication.**

The trade-off is that SPI usually needs more pins than I2C.

---

## Basic Level

### 1. What is SPI?

**SPI (Serial Peripheral Interface)** is a **synchronous serial communication interface** in which a controller/master generates the clock and communicates with one or more peripheral/slave devices.

A typical SPI interface uses four signals:

```text
SCLK → Serial Clock
MOSI → Master Out, Slave In
MISO → Master In, Slave Out
CS   → Chip Select
```

A basic connection is:

```text
Master                         Slave

SCLK  -----------------------> SCLK
MOSI  -----------------------> MOSI
MISO  <----------------------- MISO
CS    -----------------------> CS
```

---

### What does SCLK do?

`SCLK` stands for:

> **Serial Clock**

The master/controller generates the clock.

The clock synchronizes the transfer:

```text
SCLK:
__|‾|__|‾|__|‾|__|‾|__
```

Data is shifted and sampled according to the selected SPI mode.

---

### What does MOSI do?

`MOSI` stands for:

> **Master Out, Slave In**

It carries data:

```text
Master → Slave
```

For example:

```text
MCU sends:
0x9A
```

The bits are shifted out on MOSI.

---

### What does MISO do?

`MISO` stands for:

> **Master In, Slave Out**

It carries data:

```text
Slave → Master
```

So during a transfer:

```text
MOSI:
Master → Slave

MISO:
Slave → Master
```

---

### What does CS do?

`CS` stands for:

> **Chip Select**

It is commonly **active LOW**.

When:

```text
CS = LOW
```

the selected slave is enabled for the transaction.

When:

```text
CS = HIGH
```

the slave normally ignores the transaction and releases its MISO output.

For example:

```text
CS = LOW
   ↓
Slave selected

CS = HIGH
   ↓
Slave not selected
```

The exact polarity can be device-specific, but active-low CS is by far the most common.

---

### Is SPI full-duplex?

Yes.

SPI is naturally capable of **full-duplex communication**.

During every clock cycle:

```text
Master → sends one bit on MOSI
Slave  → sends one bit on MISO
```

Therefore:

```text
MOSI  →→→→→
MISO  ←←←←←
SCLK  →→→→→
```

Both directions happen simultaneously.

For example:

```text
Master sends:
0xAA

Slave sends:
0x55
```

At the same time:

```text
TX = 0xAA
RX = 0x55
```

---

### Is SPI a master-only protocol?

Traditional SPI systems are usually built around a **controller/master** that generates the clock and controls slave selection.

However, saying:

> "SPI can only ever have one master"

is too strong.

There are SPI implementations that support multiple controllers/masters, but standard MCU applications most commonly use:

```text
One Master
+
One or more Slaves
```

SPI also does not define a standard device-address field like I2C.

Instead, the master normally selects a device using its:

```text
CS
```

line.

---

### What about SPI speed?

SPI does **not define one universal maximum clock speed**.

The actual maximum depends on:

```text
MCU SPI peripheral
Slave device
PCB layout
Signal integrity
Clock mode
Cable length
Voltage level
```

Many embedded systems use:

```text
1 MHz
5 MHz
10 MHz
20 MHz
40 MHz
```

and some devices support much higher rates.

So instead of memorizing:

> "SPI = 50 MHz"

say:

> **"SPI can support much higher clock rates than typical I2C, but the maximum is device- and system-dependent."**

---

### Where is SPI used?

SPI is commonly used for:

```text
SPI Flash
EEPROM
ADC
DAC
Display
SD card
Sensors
RF modules
External controllers
```

A very common example:

```text
MCU
 │
 └── SPI → External Flash
```

because flash memory often benefits from higher throughput.

---

### SPI vs I2C

| Feature           | SPI                 | I2C                    |
| ----------------- | ------------------- | ---------------------- |
| Clock             | Yes                 | Yes                    |
| Typical lines     | 4                   | 2                      |
| Duplex            | Full-duplex         | Shared bidirectional   |
| Addressing        | Usually CS-based    | Address-based          |
| Speed             | Generally higher    | Generally lower        |
| Pull-up resistors | Normally no         | Yes                    |
| Multiple devices  | Yes                 | Yes                    |
| Main overhead     | More pins           | More protocol overhead |
| Common use        | Flash, display, ADC | Sensors, EEPROM, PMIC  |

---

### Interview Answer

> SPI is a synchronous serial communication interface where a controller generates the clock and communicates with one or more peripheral devices. A typical SPI connection uses SCLK, MOSI, MISO, and chip-select signals. SPI is usually full-duplex, uses no built-in addressing field, and selects devices using CS lines. It is commonly used where higher speed and simple framing are required, such as external Flash, displays, ADCs, DACs, and sensors.

---

### 2. What is the SPI hardware connection?

The basic multi-slave SPI connection looks like this:

```text
                         Slave 1
                      ┌───────────┐
SCLK ─────────────────│ SCLK      │
MOSI ─────────────────│ MOSI      │
MISO ─────────────────│ MISO      │
CS0  ─────────────────│ CS        │
                      └───────────┘

                         Slave 2
                      ┌───────────┐
SCLK ─────────────────│ SCLK      │
MOSI ─────────────────│ MOSI      │
MISO ─────────────────│ MISO      │
CS1  ─────────────────│ CS        │
                      └───────────┘
                          

                         Master
```

The important connection is:

```text
Master SCLK ───────────> All slaves SCLK
Master MOSI ───────────> All slaves MOSI
Master MISO <─────────── All slaves MISO
Master CS0  ───────────> Slave 1 CS
Master CS1  ───────────> Slave 2 CS
```

---

### Why are SCLK and MOSI shared?

The master sends the same clock and MOSI signal to all slaves.

But only the selected slave participates.

For example:

```text
CS1 = HIGH
CS2 = LOW
```

means:

```text
Slave 1 → Not selected
Slave 2 → Selected
```

---

### How is MISO shared safely?

This is a very important interview point.

All slaves can be connected to the same MISO line:

```text
Slave1 MISO ──┐
Slave2 MISO ──┼────> Master MISO
Slave3 MISO ──┘
```

But only the selected slave should actively drive MISO.

The other slaves should place their MISO outputs into:

```text
High-Z / Tri-state
```

Conceptually:

```text
CS = LOW
   ↓
Slave drives MISO

CS = HIGH
   ↓
Slave releases MISO
```

This prevents multiple slaves from driving the same signal simultaneously.

---

### What happens if two slaves are selected at the same time?

Suppose:

```text
CS1 = LOW
CS2 = LOW
```

and both devices drive MISO.

Then:

```text
Slave 1 → drives MISO
Slave 2 → drives MISO
```

The two outputs can conflict.

Possible results:

```text
Corrupted data
Excess current
Signal integrity problems
Potential device stress
```

Therefore:

> **Normally only one SPI slave should be selected at a time when they share MISO.**

---

### Dedicated CS per slave

A common arrangement is:

```text
             Master
          ┌───────────┐
SCLK ─────│           │────────> Slave 1
MOSI ─────│           │────────> Slave 1
MISO <────│           │<──────── Slave 1
CS0  ─────│           │────────> CS1
          │           │
CS1  ─────│           │────────> CS2
          └───────────┘
```

So:

```text
SCLK → shared
MOSI → shared
MISO → shared
CS   → separate
```

This is the most common multi-slave SPI topology.

---

### Can SPI use only one CS for multiple devices?

Yes, but then you need another mechanism to identify/control the devices.

For example:

```text
GPIO-controlled CS
External decoder
Dedicated enable logic
```

A decoder can reduce GPIO usage:

```text
MCU
 ↓
Decoder
 ↓
CS0
CS1
CS2
CS3
```

But the exact arrangement depends on the system.

---

### Another SPI topology: Daisy Chain

Some SPI devices support a daisy-chain arrangement.

Conceptually:

```text
Master
  │
  ├── MOSI ──> Device 1 ──> Device 2 ──> Device 3
  │
  └── MISO <── Device 1 <── Device 2 <── Device 3
```

The data is shifted through multiple devices.

This is device-specific and is not supported by all SPI peripherals.

---

### Interview Answer

> In a typical multi-slave SPI system, SCLK, MOSI, and MISO are shared between devices, while each slave has a separate chip-select line. The master selects one slave by asserting its CS, usually LOW. The selected slave drives MISO while unselected slaves place their MISO outputs in high-impedance mode.

---

### 3. What are SPI modes? – CPOL and CPHA

SPI has four standard clock modes:

```text
Mode 0
Mode 1
Mode 2
Mode 3
```

They are defined by:

```text
CPOL = Clock Polarity
CPHA = Clock Phase
```

---

### What is CPOL?

`CPOL` determines the **idle level of the clock**.

```text
CPOL = 0
SCLK idle = LOW
```

```text
CPOL = 1
SCLK idle = HIGH
```

So:

```text
CPOL = 0:

SCLK
____|‾‾|____|‾‾|____


CPOL = 1:

SCLK
‾‾‾‾|__|‾‾‾‾|__|‾‾‾‾
```

---

### What is CPHA?

`CPHA` determines **which clock edge is used for sampling and which edge is used for changing data**.

This is where people often make mistakes.

A good memory rule is:

```text
CPHA = 0
→ Sample on the first edge
→ Change data on the second edge

CPHA = 1
→ Change data on the first edge
→ Sample on the second edge
```

The exact meaning of "first" and "second" depends on CPOL.

---

### What is the leading edge?

The **leading edge** is the first transition away from the idle clock level.

Therefore:

```text
CPOL = 0
Idle LOW
Leading edge = Rising edge
```

```text
CPOL = 1
Idle HIGH
Leading edge = Falling edge
```

The **trailing edge** is the second transition, returning toward the idle level.

So:

```text
CPOL = 0:
Leading  = Rising
Trailing = Falling

CPOL = 1:
Leading  = Falling
Trailing = Rising
```

---

### SPI Modes

| Mode | CPOL | CPHA | Idle Clock | Sample Edge        | Change Edge        |
| ---- | ---: | ---: | ---------- | ------------------ | ------------------ |
| 0    |    0 |    0 | LOW        | Leading / Rising   | Trailing / Falling |
| 1    |    0 |    1 | LOW        | Trailing / Falling | Leading / Rising   |
| 2    |    1 |    0 | HIGH       | Leading / Falling  | Trailing / Rising  |
| 3    |    1 |    1 | HIGH       | Trailing / Rising  | Leading / Falling  |

This is the table worth remembering.

---

### SPI Mode 0

```text
CPOL = 0
CPHA = 0
```

Clock idles LOW.

The first edge is rising.

Data is sampled on the rising edge.

```text
SCLK:
____|‾‾|____|‾‾|____
    ↑       ↑
  Sample  Sample
```

---

### SPI Mode 1

```text
CPOL = 0
CPHA = 1
```

Clock idles LOW.

Data changes on the first edge.

Data is sampled on the second edge.

```text
SCLK:
____|‾‾|____|‾‾|____
    ↑       ↑
  Change   Sample
```

---

### SPI Mode 2

```text
CPOL = 1
CPHA = 0
```

Clock idles HIGH.

The first edge is falling.

Data is sampled on the falling edge.

```text
SCLK:
‾‾‾‾|__|‾‾‾‾|__|‾‾‾
     ↑         ↑
   Sample    Sample
```

---

### SPI Mode 3

```text
CPOL = 1
CPHA = 1
```

Clock idles HIGH.

Data changes on the first edge.

Data is sampled on the second edge.

```text
SCLK:
‾‾‾‾|__|‾‾‾‾|__|‾‾‾
     ↑         ↑
   Change    Sample
```

---

### Why must SPI mode match between master and slave?

Suppose:

```text
Master = Mode 0
Slave  = Mode 3
```

The two devices disagree about:

```text
Clock idle level
Sampling edge
Data-changing edge
```

The result can be:

```text
Wrong bits
Bit shift
Corrupted bytes
```

The communication may even look almost correct on a logic analyzer but contain incorrect data.

Therefore:

> Always check the slave datasheet for its required CPOL/CPHA mode.

---

### Interview Answer

> SPI has four clock modes defined by CPOL and CPHA. CPOL determines the clock's idle level, while CPHA determines which edge is used to sample data and which edge is used to change data. Mode 0 is CPOL=0, CPHA=0; Mode 1 is 0,1; Mode 2 is 1,0; and Mode 3 is 1,1. Master and slave must use compatible timing.

---

### 4. How does an SPI transfer work?

SPI uses **shift registers** to transfer data.

Suppose the master wants to send:

```text
0xAA
```

and the slave sends:

```text
0x55
```

The master loads:

```text
Master TX shift register = 0xAA
```

The slave loads:

```text
Slave TX shift register = 0x55
```

Then the master generates clock pulses.

For every clock:

```text
One bit goes from Master → Slave
One bit goes from Slave → Master
```

Therefore:

```text
MOSI = shift out
MISO = shift in
```

and at the same time:

```text
Master receives slave bits
Slave receives master bits
```

---

### Example: One-byte full-duplex transfer

```text
Master sends:
10101010

Slave sends:
01010101
```

After 8 clock pulses:

```text
Master received:
01010101 = 0x55

Slave received:
10101010 = 0xAA
```

So:

```text
MASTER:
TX = 0xAA
RX = 0x55

SLAVE:
TX = 0x55
RX = 0xAA
```

---

### What does CS do during a transfer?

Normally:

```text
CS → LOW
```

before the transaction starts.

Then:

```text
Clock pulses
```

occur while CS remains asserted.

Finally:

```text
CS → HIGH
```

to end the transaction.

Conceptually:

```text
CS:
‾‾‾‾\____________________/‾‾‾‾
      ↑                  ↑
    START               END

SCLK:
       |‾|_|‾|_|‾|_|‾|_|‾|
```

---

### Why must CS remain asserted?

Many SPI devices interpret CS as the transaction boundary.

For example, an SPI Flash command may require:

```text
CS LOW
Command
Address
Data
CS HIGH
```

If CS is released too early:

```text
CS LOW
Command
CS HIGH
CS LOW
Address
```

the device may treat these as separate transactions.

Therefore:

> **CS timing is part of the device-specific SPI protocol.**

---

### Is every SPI transfer read/write at the same time?

At the electrical level, yes:

```text
One clock
→ MOSI bit transferred
→ MISO bit transferred
```

But the application protocol may use only one direction.

For example, a display write may use:

```text
MOSI → meaningful
MISO → ignored
```

Similarly, a sensor command may send dummy bytes from the master simply to generate clock pulses while the slave returns data.

---

### Why does SPI sometimes send dummy bytes during a read?

This is an important interview question.

Suppose the master wants to read data from a sensor.

The slave cannot send data unless there are clock pulses.

So the master sends a dummy byte:

```text
Master:
0x00
```

while the slave returns:

```text
Actual sensor data
```

Conceptually:

```text
MOSI:
00000000
   ↓
Dummy data

MISO:
10101011
   ↓
Real data
```

Therefore:

> **SPI read operations often require the master to transmit dummy data to generate the clock.**

---

### Interview Answer

> SPI transfers data using shift registers and a clock generated by the master. On each clock cycle, one bit is shifted from master to slave over MOSI and one bit from slave to master over MISO, so SPI is naturally full-duplex. CS normally remains asserted throughout the device-defined transaction. During reads, the master often sends dummy bytes because it must generate clock pulses for the slave to return data.

---

### 5. How do you transmit and receive over SPI using bit-banging?

**Bit-banging** means implementing SPI in software using GPIO instead of the MCU's dedicated SPI peripheral.

The firmware manually generates:

```text
CS
SCLK
MOSI
```

and reads:

```text
MISO
```

---

### Basic CS Functions

```c
void spi_cs_low(void)
{
    GPIO_ResetBits(CS_PORT, CS_PIN);
}

void spi_cs_high(void)
{
    GPIO_SetBits(CS_PORT, CS_PIN);
}
```

This assumes:

```text
CS = Active LOW
```

---

### Bit-Banging Transfer

A simple Mode 0, MSB-first example:

```c
uint8_t spi_transfer(uint8_t byte_out)
{
    uint8_t byte_in = 0;

    for (int i = 7; i >= 0; i--)
    {
        /* Setup MOSI before sampling edge */
        if (byte_out & (1U << i))
        {
            GPIO_SetBits(MOSI_PORT, MOSI_PIN);
        }
        else
        {
            GPIO_ResetBits(MOSI_PORT, MOSI_PIN);
        }

        /*
         * Mode 0:
         * CPOL = 0
         * Data is sampled on rising edge
         */
        GPIO_SetBits(SCLK_PORT, SCLK_PIN);

        delay_ns(10);

        if (GPIO_ReadInputDataBit(MISO_PORT, MISO_PIN))
        {
            byte_in |= (1U << i);
        }

        /*
         * Return clock LOW
         */
        GPIO_ResetBits(SCLK_PORT, SCLK_PIN);

        delay_ns(10);
    }

    return byte_in;
}
```

Usage:

```c
spi_cs_low();

uint8_t response = spi_transfer(0xAA);

spi_cs_high();
```

---

### What is happening bit by bit?

Suppose:

```text
byte_out = 0xAA
```

Binary:

```text
10101010
```

The master sends:

```text
Bit 7 → 1
Bit 6 → 0
Bit 5 → 1
Bit 4 → 0
Bit 3 → 1
Bit 2 → 0
Bit 1 → 1
Bit 0 → 0
```

For every bit:

```text
1. Set MOSI
2. Generate sampling clock edge
3. Read MISO
4. Complete clock cycle
```

---

### Important: Bit-banging must match the SPI mode

The above code is effectively describing:

```text
Mode 0
MSB first
```

If the device expects:

```text
Mode 3
LSB first
```

the implementation must change accordingly.

For example:

```text
CPOL
CPHA
Bit order
CS polarity
```

all affect the implementation.

---

### Bit-banging disadvantages

Software SPI consumes CPU time.

For example:

```text
CPU
 ↓
GPIO write
 ↓
Delay
 ↓
GPIO read
 ↓
GPIO write
 ↓
Delay
 ↓
Repeat
```

At high speed this becomes expensive.

Other disadvantages:

```text
Interrupt sensitivity
Timing jitter
Lower throughput
CPU overhead
Harder RTOS integration
```

Hardware SPI is usually preferable when available.

---

### Interview Answer

> SPI bit-banging uses GPIO to manually generate CS, SCLK, and MOSI while reading MISO. The firmware must implement the selected CPOL/CPHA mode, bit order, and timing requirements. It is useful for simple or unusual interfaces but consumes CPU time and is generally less efficient and less deterministic than using the MCU's hardware SPI peripheral.

---

### 6. How do you configure SPI on a microcontroller?

A classic STM32 Standard Peripheral Library example is:

```c
SPI_InitTypeDef SPI_InitStructure;

SPI_InitStructure.SPI_Direction =
    SPI_Direction_2Lines_FullDuplex;

SPI_InitStructure.SPI_Mode =
    SPI_Mode_Master;

SPI_InitStructure.SPI_DataSize =
    SPI_DataSize_8b;

SPI_InitStructure.SPI_CPOL =
    SPI_CPOL_Low;

SPI_InitStructure.SPI_CPHA =
    SPI_CPHA_1Edge;

SPI_InitStructure.SPI_NSS =
    SPI_NSS_Soft;

SPI_InitStructure.SPI_BaudRatePrescaler =
    SPI_BaudRatePrescaler_16;

SPI_Init(SPI1, &SPI_InitStructure);

SPI_Cmd(SPI1, ENABLE);
```

This configures a typical:

```text
Master
8-bit
Full-duplex
Mode 0
Software-controlled NSS/CS
Prescaler = 16
```

---

### What does each field mean?

#### Direction

```c
SPI_Direction_2Lines_FullDuplex
```

Means:

```text
MOSI + MISO
```

and full-duplex operation.

---

#### Master Mode

```c
SPI_Mode_Master
```

The MCU generates:

```text
SCLK
```

and controls the transaction.

---

#### Data Size

```c
SPI_DataSize_8b
```

Each SPI frame unit is:

```text
8 bits
```

Some peripherals also support:

```text
16-bit
```

or other sizes.

---

#### CPOL

```c
SPI_CPOL_Low
```

means:

```text
Clock idle = LOW
```

---

#### CPHA

```c
SPI_CPHA_1Edge
```

means sampling occurs on the first edge for the particular STM32 convention, corresponding to:

```text
CPHA = 0
```

Together:

```text
CPOL = 0
CPHA = 0
```

gives:

```text
SPI Mode 0
```

---

#### NSS

```c
SPI_NSS_Soft
```

means the SPI peripheral's NSS handling is controlled in software.

In many MCU applications:

```text
GPIO → CS
```

is manually controlled:

```c
CS_LOW();
SPI_Transfer(...);
CS_HIGH();
```

This is useful because many SPI devices need precise CS control across a complete command sequence.

---

#### Baud Rate Prescaler

```c
SPI_BaudRatePrescaler_16
```

The SPI clock is derived from the peripheral clock using the selected prescaler.

Conceptually:

```text
SPI Clock
≈
Peripheral Clock / Prescaler
```

The exact clock-divider behavior depends on the MCU.

For example, if:

```text
Peripheral Clock = 80 MHz
Prescaler = 16
```

then approximately:

```text
SPI Clock = 80 MHz / 16
          = 5 MHz
```

Always verify the exact peripheral clock feeding the SPI instance.

---

### What else must be configured?

Just like UART/I2C, SPI initialization is not complete with `SPI_Init()` alone.

You also need to configure:

```text
Peripheral clock
GPIO clock
SCLK pin
MOSI pin
MISO pin
GPIO alternate function
CS GPIO
SPI peripheral enable
```

Conceptually:

```text
System Clock
     ↓
SPI Peripheral Clock
     ↓
SPI Configuration
     ↓
GPIO Alternate Function
     ↓
CS GPIO
```

---

### Important: CS may not be automatically controlled

A common senior-level point:

```text
SPI_NSS = Software
```

does not necessarily mean the peripheral automatically toggles your external CS pin.

Often the firmware must explicitly do:

```c
CS_LOW();

SPI_Transfer(...);

CS_HIGH();
```

The exact behavior depends on the MCU SPI peripheral.

---

### Interview Answer

> To configure SPI, I would configure the peripheral clock and GPIO alternate functions, then set master/slave mode, data width, full- or half-duplex direction, CPOL, CPHA, bit order if supported, clock prescaler, and chip-select handling. I would also verify the slave datasheet for the required SPI mode, maximum clock frequency, CS timing, and command format.

---

### 7. How do you transfer data using the SPI peripheral?

SPI is fundamentally a **transmit-and-receive operation**.

When the master sends data, the hardware also receives whatever appears on MISO during the clock cycles.

A polling example is:

```c
/* Wait until transmit register is empty */
while (SPI_GetFlagStatus(
           SPI1,
           SPI_FLAG_TXE) == RESET)
{
}

SPI_SendData(SPI1, 0xAA);

/* Wait until received data is available */
while (SPI_GetFlagStatus(
           SPI1,
           SPI_FLAG_RXNE) == RESET)
{
}

uint8_t response =
    (uint8_t)SPI_ReceiveData(SPI1);
```

---

### What is TXE?

`TXE` means:

> **Transmit Buffer/Shift Register is ready for another write**, depending on the exact SPI peripheral implementation.

In many STM32 peripherals, TXE means the transmit data register is empty and can accept more data.

So:

```text
TXE = 1
    ↓
Software can write next data
```

---

### What is RXNE?

`RXNE` means:

> **Receive data is available.**

So:

```text
RXNE = 1
    ↓
Received data can be read
```

---

### Full-Duplex Transfer

The important point is that a single SPI transfer usually does both directions simultaneously.

For example:

```c
while (SPI_GetFlagStatus(
           SPI1,
           SPI_FLAG_TXE) == RESET)
{
}

SPI_SendData(SPI1, 0xAA);

while (SPI_GetFlagStatus(
           SPI1,
           SPI_FLAG_RXNE) == RESET)
{
}

uint8_t response =
    (uint8_t)SPI_ReceiveData(SPI1);
```

Conceptually:

```text
Master sends 0xAA
        ↓
8 clock pulses
        ↓
Master receives whatever slave sent
```

---

### Important SPI Read Concept

Suppose the slave contains:

```text
Temperature = 0x3C
```

The master may need to send:

```text
Command = READ
```

and then send a dummy byte:

```text
0x00
```

to generate the clock.

Example:

```c
SPI_Transfer(READ_COMMAND);
SPI_Transfer(0x00);     /* Dummy byte */

data = SPI_ReadReceivedByte();
```

The second transfer generates the clocks during which the slave sends:

```text
0x3C
```

---

### A Better Transfer API

Instead of separate transmit and receive functions, a common SPI driver API is:

```c
uint8_t SPI_Transfer(uint8_t tx)
{
    while (SPI_GetFlagStatus(
               SPI1,
               SPI_FLAG_TXE) == RESET)
    {
    }

    SPI_SendData(SPI1, tx);

    while (SPI_GetFlagStatus(
               SPI1,
               SPI_FLAG_RXNE) == RESET)
    {
    }

    return (uint8_t)SPI_ReceiveData(SPI1);
}
```

Then:

```c
uint8_t rx;

CS_LOW();

rx = SPI_Transfer(0xAA);

CS_HIGH();
```

This makes the full-duplex nature much clearer:

```text
Input:
TX byte

Output:
RX byte
```

---

### Multi-Byte Transfer

Suppose we want to send:

```text
0x9A
0x10
0x20
0x30
```

The transaction may be:

```c
CS_LOW();

SPI_Transfer(0x9A);
SPI_Transfer(0x10);
SPI_Transfer(0x20);
SPI_Transfer(0x30);

CS_HIGH();
```

The exact meaning depends on the slave protocol.

For example:

```text
0x9A → Command
0x10 → Register
0x20 → Data
0x30 → Data
```

The SPI bus itself does not define those meanings.

---

### Why must CS usually remain LOW for all bytes?

Because many SPI peripherals define one complete command as:

```text
CS LOW
Command
Address
Data
CS HIGH
```

If you toggle CS between every byte:

```text
CS LOW
Command
CS HIGH

CS LOW
Address
CS HIGH
```

the slave may interpret them as separate commands.

Therefore:

> **CS timing is part of the peripheral's protocol specification.**

---

### What happens if you forget to read RX data?

Because SPI is full-duplex, every transmitted byte produces a received byte at the hardware level.

If firmware repeatedly transmits without handling RX properly, the receive buffer may eventually produce:

```text
Overrun / overflow
```

depending on the peripheral.

This is a common mistake in SPI drivers.

Even if the application doesn't care about MISO, the software may still need to read or otherwise correctly handle the received data path.

---

### Polling vs Interrupt vs DMA

SPI can be handled using:

```text
Polling
Interrupt
DMA
```

For small transfers:

```text
Polling
```

may be simple enough.

For larger transfers:

```text
DMA
```

is often preferable.

For example:

```text
RAM TX Buffer
     ↓
    DMA
     ↓
   SPI TX
```

and:

```text
SPI RX
   ↓
  DMA
   ↓
RAM RX Buffer
```

This allows large SPI transfers with lower CPU overhead.

---

### Interview Answer

> SPI transfer is inherently full-duplex. Writing a byte to the transmit register causes the SPI hardware to generate clock pulses and simultaneously receive a byte from MISO. The firmware usually waits for TX readiness and then for RX data availability, while keeping CS asserted for the complete device-defined transaction. For larger transfers, DMA can move data between RAM and SPI with much lower CPU overhead.

---

# SPI Basic-Level Quick Revision

| Topic           | Key Point                                     |
| --------------- | --------------------------------------------- |
| SPI             | Synchronous serial interface                  |
| Typical Signals | SCLK, MOSI, MISO, CS                          |
| SCLK            | Clock generated by controller/master          |
| MOSI            | Master Out, Slave In                          |
| MISO            | Master In, Slave Out                          |
| CS              | Selects the target, commonly active LOW       |
| Duplex          | Full-duplex                                   |
| Addressing      | Usually no address field; CS selects device   |
| Multiple Slaves | Shared SCLK/MOSI/MISO + separate CS           |
| MISO            | Unselected slaves should normally be High-Z   |
| Speed           | No universal maximum; device-dependent        |
| CPOL            | Clock idle level                              |
| CPHA            | Sampling/change timing relative to clock edge |
| Mode 0          | CPOL=0, CPHA=0                                |
| Mode 1          | CPOL=0, CPHA=1                                |
| Mode 2          | CPOL=1, CPHA=0                                |
| Mode 3          | CPOL=1, CPHA=1                                |
| Full-Duplex     | One bit sent and one bit received per clock   |
| Read            | Often requires dummy TX bytes                 |
| TXE             | TX register/buffer ready                      |
| RXNE            | RX data available                             |
| DMA             | Useful for large/high-speed transfers         |

---

# Easy SPI Memory Trick

Remember:

```text
SPI
│
├── SCLK → Clock
├── MOSI → Master → Slave
├── MISO → Slave → Master
└── CS   → Select Slave
```

For multiple devices:

```text
SCLK ─────────── Shared
MOSI ─────────── Shared
MISO ─────────── Shared
CS0  ─────────── Slave 0
CS1  ─────────── Slave 1
CS2  ─────────── Slave 2
```

For SPI modes:

```text
CPOL → Where is clock when idle?

CPHA → Which edge samples data?
```

And:

```text
Mode 0 → 00
Mode 1 → 01
Mode 2 → 10
Mode 3 → 11
```

---

# Important SPI Interview Traps

### Trap 1 – "SPI maximum speed is 50 MHz"

Not universally.

Better:

> SPI has no single protocol-wide maximum clock frequency. The usable speed is determined by the MCU, slave device, board, signal integrity, and electrical constraints.

---

### Trap 2 – "SPI has device addresses like I2C"

Normally, no.

SPI usually uses:

```text
CS0 → Device 0
CS1 → Device 1
CS2 → Device 2
```

rather than putting an address field on the bus.

---

### Trap 3 – "MISO can always be connected together"

Only if unselected devices properly release MISO.

```text
Selected slave → Drives MISO
Unselected slave → High-Z
```

Otherwise two outputs may fight.

---

### Trap 4 – "Read means only MISO is active"

Not quite.

SPI needs clock pulses.

Therefore the master often sends:

```text
Dummy byte
```

on MOSI to generate those clocks while receiving meaningful data on MISO.

---

### Trap 5 – "CS is part of the SPI protocol in exactly the same way everywhere"

Not quite.

SPI defines the basic serial signaling concept, but many details such as:

```text
Command format
Address format
CS timing
CS polarity
Transaction length
```

are often defined by the **specific slave device**.

---

### Trap 6 – "CPOL and CPHA are the same thing"

No.

```text
CPOL
↓
Clock idle level

CPHA
↓
Sampling / changing edge relationship
```

---

# Senior-Level Mental Model

For a real embedded SPI driver, think in layers:

```text
┌──────────────────────────────┐
│ Device Protocol              │
│ Command / Address / Data     │
├──────────────────────────────┤
│ SPI Driver                   │
│ CS / Polling / IRQ / DMA     │
├──────────────────────────────┤
│ SPI Peripheral               │
│ CPOL / CPHA / Clock / FIFO   │
├──────────────────────────────┤
│ GPIO / Electrical Interface  │
│ SCLK / MOSI / MISO / CS      │
└──────────────────────────────┘
```

When debugging:

```text
No response
   ↓
CS correct?
   ↓
SPI mode correct?
   ↓
Clock frequency valid?
   ↓
MOSI waveform correct?
   ↓
MISO responding?
   ↓
Command/address correct?
   ↓
CS timing correct?
   ↓
Read RX register correctly?
```

A strong senior-level answer should connect:

```text
SPI configuration
      ↓
CPOL / CPHA
      ↓
Clock timing
      ↓
CS behavior
      ↓
Device command protocol
      ↓
FIFO / Interrupt / DMA
      ↓
Actual waveform on the logic analyzer
```

> **The key idea: SPI defines the data-transfer mechanism, but the actual command protocol is usually defined by the specific peripheral connected to the SPI bus.**

---

## Intermediate Level

### 8. How do you use SPI with DMA?

**DMA (Direct Memory Access)** allows the SPI peripheral to transfer data between the SPI hardware and memory without the CPU handling every individual byte.

Without DMA:

```text
RAM
 ↓
CPU
 ↓
SPI
 ↓
Transmit

SPI
 ↓
CPU
 ↓
RAM
 ↓
Receive
```

With DMA:

```text
TX:

RAM Buffer
    ↓
   DMA
    ↓
SPI TX
    ↓
MOSI


RX:

MISO
 ↓
SPI RX
 ↓
DMA
 ↓
RAM Buffer
```

The CPU mainly:

```text
Configure DMA
Start transfer
Wait for completion/event
Process received data
```

---

### Why is DMA useful for SPI?

Suppose you need to transfer:

```text
4096 bytes
```

With polling:

```text
CPU
 ↓
Check SPI
 ↓
Write byte
 ↓
Wait
 ↓
Read byte
 ↓
Repeat thousands of times
```

With interrupts:

```text
SPI
 ↓
Interrupt
 ↓
CPU
 ↓
Handle byte
 ↓
Interrupt
 ↓
CPU
```

With DMA:

```text
RAM
 ↓
DMA
 ↓
SPI
```

The CPU is mostly free to perform other work.

---

### SPI DMA TX

For transmission:

```text
uint8_t tx_buffer[1024];
```

DMA is configured roughly as:

```text
Memory → SPI Data Register
```

Conceptually:

```text
RAM:
+----+----+----+----+----+
| A1 | A2 | A3 | A4 | A5 |
+----+----+----+----+----+
        ↓
       DMA
        ↓
   SPI TX Register
        ↓
       SCLK
        ↓
       MOSI
```

---

### SPI DMA RX

For reception:

```text
SPI RX Register → DMA → RAM
```

Conceptually:

```text
MISO
 ↓
SPI RX Register
 ↓
DMA
 ↓
+----+----+----+----+
| B1 | B2 | B3 | B4 |
+----+----+----+----+
```

---

### Why do we normally use both TX and RX DMA for SPI?

Because SPI is full-duplex.

When the master transmits:

```text
MOSI → Slave
```

the SPI peripheral is also receiving:

```text
MISO → Master
```

So for a large full-duplex transfer:

```text
TX RAM ──DMA──> SPI ──MOSI──> Slave
RX RAM <──DMA── SPI <──MISO── Slave
```

This allows both directions to be handled efficiently.

---

### What happens when DMA transfer completes?

The DMA controller can generate an interrupt:

```text
DMA Transfer
     ↓
Transfer Complete
     ↓
DMA Interrupt
     ↓
ISR
     ↓
Signal application/task
```

In an RTOS:

```text
DMA complete
     ↓
ISR
     ↓
Task notification / semaphore
     ↓
Application task wakes up
```

This is usually preferable to doing the entire processing inside the ISR.

---

### Important: DMA does not remove CS handling

Suppose a device requires:

```text
CS LOW
Command
Address
Data
CS HIGH
```

You still need to control CS correctly.

For example:

```c
CS_LOW();

SPI_DMA_Transfer(
    tx_buffer,
    rx_buffer,
    transfer_length
);

WaitForDMADone();

CS_HIGH();
```

But be careful:

> Some SPI peripherals have hardware NSS/CS support, while others require software-controlled GPIO. The exact timing must satisfy the slave datasheet.

---

### Important DMA issue: cache coherency

On MCUs with data/instruction caches, DMA buffers can require cache maintenance.

For example:

```text
CPU writes TX buffer
        ↓
Data still in cache
        ↓
DMA reads RAM
        ↓
DMA sees old data
```

Similarly:

```text
DMA writes RX buffer
        ↓
CPU cache contains old data
        ↓
CPU reads stale data
```

Depending on the MCU architecture, you may need:

```text
Cache clean
Cache invalidate
Non-cacheable DMA memory
```

This is a common senior-level DMA interview topic.

---

### SPI DMA – Typical Flow

```text
Application
    ↓
Prepare TX/RX buffers
    ↓
Configure SPI
    ↓
Configure DMA
    ↓
CS LOW
    ↓
Start DMA
    ↓
SPI transfer
    ↓
DMA Transfer Complete
    ↓
CS HIGH
    ↓
Notify Task
    ↓
Process RX Data
```

---

### Interview Answer

> SPI with DMA is used to reduce CPU overhead during large transfers. TX DMA moves data from memory to the SPI transmit register, while RX DMA moves received data from the SPI receive register to memory. Since SPI is full-duplex, both channels can operate simultaneously. A transfer-complete interrupt can notify the CPU when the DMA transaction finishes. CS handling, buffer ownership, and cache coherency must also be considered on real systems.

---

### 9. How do you implement an SPI multi-byte transaction?

SPI itself does not define commands such as:

```text
READ
WRITE
REGISTER
ADDRESS
```

Those are normally defined by the specific SPI peripheral.

For example, suppose a device defines:

```text
0x03 = READ command
0x10 = Register/address
```

and we want to read two bytes.

The transaction could be:

```text
CS LOW
   ↓
0x03          → READ command
   ↓
0x10          → Address
   ↓
0xFF          → Dummy byte / generate clock
   ↓
0xFF          → Dummy byte / generate clock
   ↓
CS HIGH
```

---

### Code Example

```c
uint8_t cmd  = 0x03;
uint8_t addr = 0x10;

uint8_t data_high;
uint8_t data_low;

spi_cs_low();

spi_transfer(cmd);
spi_transfer(addr);

data_high = spi_transfer(0xFF);
data_low  = spi_transfer(0xFF);

spi_cs_high();
```

The important point is:

```text
Master sends 0xFF
        ↓
Slave returns data on MISO
```

The value `0xFF` is not necessarily meaningful to the slave.

It is often simply a **dummy byte used to generate eight clock pulses**.

---

### Full Waveform Concept

```text
CS:
‾‾‾‾\_______________________________/‾‾‾‾
     ↑                               ↑
   Start                             End

MOSI:

[ 0x03 ] [ 0x10 ] [ 0xFF ] [ 0xFF ]

MISO:

[ XXXX ] [ XXXX ] [DATA_H] [DATA_L]
```

Where:

```text
XXXX
```

means the received value during command/address transmission may be:

```text
Don't care
Status
Previous data
Undefined
```

depending on the slave.

---

### Why must CS remain LOW?

Because many devices treat:

```text
CS LOW
```

as:

> "This is one continuous transaction."

So:

```text
CS LOW
Command
Address
Data
CS HIGH
```

is different from:

```text
CS LOW
Command
CS HIGH

CS LOW
Address
CS HIGH
```

The second sequence may not be interpreted correctly by the slave.

---

### Multi-byte Write

A write transaction could be:

```text
CS LOW
   ↓
WRITE COMMAND
   ↓
ADDRESS
   ↓
DATA1
   ↓
DATA2
   ↓
DATA3
   ↓
CS HIGH
```

For example:

```c
spi_cs_low();

spi_transfer(0x02);   /* WRITE */
spi_transfer(0x10);   /* Address */

spi_transfer(0xAA);
spi_transfer(0xBB);
spi_transfer(0xCC);

spi_cs_high();
```

Again, the command values are device-specific.

---

### Important Senior-Level Point

SPI defines:

```text
Clock
Data
Chip Select
```

but does **not** universally define:

```text
Read command
Write command
Register address
Dummy cycles
Data length
```

These are part of the **slave device protocol**.

This distinction is especially important when working with:

```text
SPI Flash
Display controllers
ADC
Sensors
DAC
```

---

### Interview Answer

> A multi-byte SPI transaction usually keeps CS asserted while the master sends a device-specific command, address, and data. During reads, the master often sends dummy bytes to generate the clocks required for the slave to return data. The exact command and address format is defined by the peripheral datasheet, not by the SPI bus itself.

---

### 10. What is Quad SPI (QSPI)?

**Quad SPI**, commonly called **QSPI**, extends the SPI interface so that data can be transferred using **four data lines in parallel**.

Traditional SPI normally uses:

```text
MOSI
MISO
```

for a total of:

```text
1 data bit per clock in each direction
```

Quad SPI can use:

```text
IO0
IO1
IO2
IO3
```

so that up to:

```text
4 bits per clock
```

can be transferred during quad-data phases.

---

### Traditional SPI

```text
            1 bit
Master ─────────────> Slave

MOSI
```

### Quad SPI

```text
              4 bits
Master =================> Flash

IO0
IO1
IO2
IO3
```

Conceptually:

```text
SPI:
Clock → 1 data bit/clock

Quad:
Clock → 4 data bits/clock
```

This is why, under the same clock and suitable protocol conditions, quad transfer can provide approximately **4× the data throughput of single-bit transfer during the quad-data phase**.

It does not mean the entire transaction is always exactly four times faster because commands, addresses, dummy cycles, and device protocol overhead still exist.

---

### Typical QSPI Signals

A common QSPI Flash interface looks like:

```text
MCU                     Flash

SCLK  ----------------> CLK
CS    ----------------> CS

IO0   <---------------> IO0
IO1   <---------------> IO1
IO2   <---------------> IO2
IO3   <---------------> IO3
```

The exact signal naming can vary.

In a conventional 1-bit SPI configuration, some of these pins have roles such as:

```text
IO0 → MOSI
IO1 → MISO
```

and the additional lines become active during quad operation.

---

### Why is QSPI commonly used for Flash?

Flash memory often needs to transfer a large amount of data.

For example:

```text
Firmware
Images
Fonts
Audio
Application data
```

Single-bit SPI can become a bandwidth bottleneck.

QSPI allows:

```text
More bits/clock
   ↓
Higher read throughput
```

while still using relatively few pins compared with a parallel memory bus.

---

### Can QSPI start in normal SPI mode?

Yes, many QSPI Flash devices support a protocol where the device can initially communicate using conventional SPI-style commands and then enter quad-data operations.

However:

> The exact command sequence and enabling mechanism are device-specific.

Some devices require configuration bits or specific commands before quad operation is enabled.

---

### Is QSPI always exactly 4× faster?

No.

A better answer is:

> "Quad data transfer provides up to four bits per clock during quad phases, but the overall speed improvement depends on command overhead, address cycles, dummy cycles, clock frequency, and the memory device."

---

### QSPI vs SPI

| Feature             | Standard SPI                | QSPI                      |
| ------------------- | --------------------------- | ------------------------- |
| Data lines          | 1 TX + 1 RX                 | 4 bidirectional I/O lines |
| Data per clock      | Up to 1 bit per data lane   | Up to 4 bits              |
| Typical use         | Sensors, controllers, Flash | Mainly high-speed Flash   |
| Throughput          | Lower                       | Higher                    |
| Complexity          | Lower                       | Higher                    |
| Common architecture | MOSI/MISO                   | IO0–IO3                   |

---

### Interview Answer

> QSPI is an extension of SPI that uses four data I/O lines to transfer up to four bits per clock during quad-data phases. It is widely used with external NOR Flash to improve read and write throughput without requiring a wide parallel memory bus. The overall throughput is not necessarily exactly four times higher because commands, addresses, and dummy cycles still consume clock cycles.

---

### 11. What is Dual SPI (DSPI)?

**Dual SPI** uses **two data lines** to transfer data in parallel.

Instead of:

```text
1 bit/clock
```

it can transfer:

```text
2 bits/clock
```

during dual-data phases.

Conceptually:

```text
Standard SPI:

IO0 → 1 bit


Dual SPI:

IO0 → bit 0
IO1 → bit 1
```

So:

```text
Clock
  ↓
2 bits transferred
```

---

### Where is Dual SPI used?

Dual-SPI modes are commonly found in:

```text
NOR Flash
Memory interfaces
Some high-speed peripherals
```

The exact protocol is device-specific.

---

### Is Dual SPI exactly 2× faster?

Not necessarily.

Just like QSPI:

```text
2 bits/clock
```

does not mean:

```text
Entire transaction = exactly 2× faster
```

because there can be:

```text
Command
Address
Dummy cycles
Wait states
```

Therefore:

> Dual SPI can provide approximately twice the data width during dual-data phases, but overall throughput depends on the complete transaction.

---

### SPI vs Dual SPI vs Quad SPI

```text
Standard SPI
    ↓
1 data bit / clock

Dual SPI
    ↓
2 data bits / clock

Quad SPI
    ↓
4 data bits / clock
```

This is the easiest way to remember them.

---

### Interview Answer

> Dual SPI is a wider SPI transfer mode that uses two data lines in parallel, allowing up to two bits per clock during dual-data phases. It is commonly used with Flash memories. The actual throughput improvement depends on the device protocol and the amount of command, address, and dummy-cycle overhead.

---

### 12. What is SPI Slave Mode?

In slave mode, the microcontroller does **not generate the SPI clock**.

Instead:

```text
External Master
      ↓
Generates SCLK
      ↓
Selects MCU using CS
      ↓
MCU acts as SPI Slave
```

The connection is:

```text
External Master                 MCU Slave

SCLK  ----------------------->  SCLK
MOSI  ----------------------->  MOSI
MISO  <-----------------------  MISO
CS    ----------------------->  CS
```

---

### What does the slave do?

The slave waits for:

```text
CS asserted
```

and then uses:

```text
SCLK
```

provided by the master to shift data.

The slave cannot decide:

```text
When to generate SCLK
```

because:

> **The master owns the clock.**

---

### Typical Slave Sequence

```text
CS HIGH
   ↓
Idle

CS LOW
   ↓
Transaction starts

SCLK pulses
   ↓
Receive MOSI
Transmit MISO
   ↓
CS HIGH
   ↓
Transaction ends
```

---

### How does the slave know when to prepare TX data?

This depends on the slave protocol.

The master may first send:

```text
Command
```

and then the slave needs to return data.

The slave firmware must prepare the transmit data **before the master clocks it out**.

This creates an important design challenge:

```text
Master clocks immediately
        ↓
Slave must already have data ready
```

For real-time slave designs, preloading TX FIFO/registers is often important.

---

### Typical SPI Slave Interrupt Events

Depending on the MCU:

```text
CS asserted
RX data available
TX register empty
RX FIFO threshold
DMA complete
Overrun
```

But SPI hardware generally does not have a universal "CS interrupt" feature across all MCUs.

Some MCUs provide hardware support; others require a GPIO interrupt for CS.

---

### SPI Slave and DMA

DMA is very useful when the MCU acts as a high-speed SPI slave.

For example:

```text
Master
  ↓
SPI SCLK
  ↓
MCU SPI Peripheral
  ↓
DMA
  ↓
RAM
```

TX:

```text
RAM
 ↓
DMA
 ↓
SPI TX
 ↓
MISO
```

RX:

```text
MOSI
 ↓
SPI RX
 ↓
DMA
 ↓
RAM
```

This reduces CPU response-time requirements.

---

### Important Slave-Mode Challenge

Suppose the master starts clocking data immediately after CS goes LOW.

The slave has very little time to respond.

Therefore, you need to consider:

```text
CS-to-first-clock delay
TX preload time
DMA startup latency
ISR latency
FIFO depth
Clock frequency
```

This is a very good senior-level interview topic.

---

### What happens if the slave cannot keep up?

Possible problems:

```text
RX overflow
TX underrun
Corrupted data
Missed first byte
Protocol desynchronization
```

The exact error flags depend on the MCU.

---

### Interview Answer

> In SPI slave mode, the external master controls CS and SCLK. The MCU detects its selection, receives data on MOSI, and transmits data on MISO according to the master's clock. For high-speed slave operation, FIFO and DMA are often used because the slave must respond within the master's timing. Preloading TX data and handling CS timing are especially important.

---

### 13. How do you debug SPI?

The best way to debug SPI is:

> **Look at the actual CS, SCLK, MOSI, and MISO waveforms.**

Do not rely only on software logs.

---

### Step 1 – Check Wiring

Verify:

```text
Master SCLK → Slave SCLK
Master MOSI → Slave MOSI
Master MISO ← Slave MISO
Master CS   → Slave CS
GND         → GND
```

Common mistakes:

```text
MOSI/MISO swapped
Wrong CS pin
Missing GND
Wrong peripheral pins
```

---

### Step 2 – Check SPI Mode

Verify:

```text
CPOL
CPHA
```

against the slave datasheet.

For example:

```text
Slave requires:
Mode 3
```

but firmware uses:

```text
Mode 0
```

Result:

```text
Wrong sampling edge
      ↓
Corrupted data
```

---

### Step 3 – Check Clock Frequency

Suppose the slave supports:

```text
Maximum = 20 MHz
```

but your MCU sends:

```text
40 MHz
```

The communication may fail.

The result can be:

```text
Intermittent errors
Wrong data
No response
```

Always check the peripheral's maximum SPI clock under the actual voltage and operating conditions.

---

### Step 4 – Check CS Timing

Verify:

```text
CS goes LOW
      ↓
Required setup time
      ↓
Clock starts
      ↓
Transaction
      ↓
Last clock completes
      ↓
Required hold time
      ↓
CS goes HIGH
```

Different devices have different requirements.

---

### Step 5 – Use a Logic Analyzer

Check:

```text
CS
SCLK
MOSI
MISO
```

A logic analyzer can decode:

```text
SPI Mode
Clock frequency
Bit order
TX bytes
RX bytes
CS boundaries
```

For example, you might discover:

```text
Firmware expected:
03 10 FF FF

Actual:
03 08 FF FF
```

Then the problem is immediately narrowed to the transmitted command/address rather than signal integrity.

---

### Step 6 – Use an Oscilloscope

Use an oscilloscope when the digital protocol looks correct but the electrical signal is suspicious.

Check:

```text
Rise time
Fall time
Overshoot
Undershoot
Ringing
Noise
Voltage levels
Clock duty cycle
```

For higher-speed SPI, signal integrity becomes increasingly important.

---

### Step 7 – Check Setup/Hold Time

The slave needs enough time for:

```text
Data setup
Data hold
```

relative to the sampling clock edge.

Conceptually:

```text
Data:
─────── stable ─────────────
        ↑
        │
      Sample

Clock:
________|‾‾‾‾‾
        ↑
     Sample edge
```

If data changes too close to the sampling edge:

```text
Timing violation
      ↓
Wrong bit
```

---

### Step 8 – Reduce Clock Speed

This is a useful diagnostic technique.

Suppose:

```text
40 MHz → fails
20 MHz → works
10 MHz → works
```

That strongly suggests a timing/electrical margin problem.

Then investigate:

```text
Signal integrity
Clock rise/fall time
Setup/hold time
PCB routing
Driver strength
Termination
```

---

### Step 9 – Check Software RX Handling

Because SPI is full-duplex, verify that firmware is reading RX data.

Check:

```text
RX FIFO
RXNE
Overrun
DMA status
Buffer size
```

A transmit-only application may still generate received data internally.

---

### Interview Answer

> I would debug SPI from the physical layer upward. First I verify wiring, CS, clock mode, clock frequency, and GPIO configuration. Then I use a logic analyzer to verify CS, SCLK, MOSI, MISO, command bytes, bit order, and timing. If the digital sequence looks correct but communication is unreliable, I use an oscilloscope to inspect signal integrity, rise/fall time, ringing, noise, and setup/hold margin. I would also check RX overrun, DMA status, and buffer handling.

---

### 14. What causes SPI failure?

SPI failures can come from:

```text
Configuration
Timing
CS handling
Software
Electrical
Signal integrity
```

---

### 1. Wrong CPOL/CPHA Mode

Example:

```text
Master = Mode 0
Slave  = Mode 3
```

The devices sample data on different clock edges.

Result:

```text
Wrong bits
Corrupted data
```

---

### 2. Clock Speed Too High

Example:

```text
Slave maximum = 10 MHz
Master sends   = 20 MHz
```

The slave may not meet:

```text
Setup time
Hold time
Propagation time
```

Result:

```text
Unreliable communication
```

---

### 3. CS Toggles During a Transaction

Suppose the slave expects:

```text
CS LOW
Command
Address
Data
CS HIGH
```

but firmware does:

```text
CS LOW
Command
CS HIGH

CS LOW
Address
Data
CS HIGH
```

The slave may interpret this incorrectly.

---

### 4. Wrong CS Polarity

The firmware may assume:

```text
CS LOW = selected
```

while a specific external interface may use a different arrangement.

Always check the device requirements.

---

### 5. Data Setup/Hold Violation

The master changes MOSI too close to the slave sampling edge.

Example:

```text
MOSI changes
      ↓
    too close
      ↓
Sample edge
```

Result:

```text
Unstable / incorrect bit
```

This becomes especially important at high clock rates.

---

### 6. Signal Integrity Problems

At higher speeds, problems such as:

```text
Ringing
Overshoot
Undershoot
Crosstalk
Noise
Slow rise/fall
Ground bounce
```

can corrupt the clock or data.

---

### 7. Poor PCB Routing

SPI signals can be sensitive to:

```text
Long traces
Stubs
Poor return path
Aggressive neighboring signals
Connector reflections
```

A design that works on a short bench connection may fail on the final PCB.

---

### 8. Floating or Incorrectly Controlled Lines

Unlike I2C, SPI does not generally require pull-up resistors on all data lines.

But some SPI devices may require specific biasing or known states for:

```text
CS
Reset
Enable
```

A floating CS can cause random device selection.

---

### 9. MISO Contention

If two slaves drive MISO simultaneously:

```text
Slave 1 → HIGH
Slave 2 → LOW
```

you can get:

```text
Bus contention
Corrupted data
Excess current
```

Check that unselected devices properly release MISO.

---

### 10. RX Overrun / TX Underrun

In high-speed transfers:

```text
CPU too slow
      ↓
RX FIFO fills
      ↓
Overrun
```

For slave mode:

```text
Master clocks data
      ↓
Slave has no TX data ready
      ↓
TX underrun
```

DMA and FIFO can help.

---

### 11. Wrong Bit Order

Some devices use:

```text
MSB first
```

others may use:

```text
LSB first
```

If the wrong order is configured:

```text
Expected:
10110010

Received/interpreted:
01001101
```

the bytes can appear bit-reversed.

Always check the device datasheet.

---

### 12. Wrong Command/Address

The SPI electrical transfer may be perfect while the device still does not respond because the application protocol is wrong.

For example:

```text
Expected:
03 10 FF FF

Sent:
02 10 FF FF
```

The waveform looks good, but the device receives the wrong command.

This is why:

> **"SPI waveform is clean" does not automatically mean "SPI communication is correct."**

You must also verify the device command protocol.

---

### SPI Failure Debugging Flow

```text
SPI Failure
    ↓
Check Power / GND
    ↓
Check CS
    ↓
Check CPOL / CPHA
    ↓
Check Clock Frequency
    ↓
Check Bit Order
    ↓
Check Command / Address
    ↓
Check MOSI / MISO
    ↓
Check RX / TX FIFO
    ↓
Check DMA
    ↓
Check Setup / Hold
    ↓
Check Signal Integrity
```

---

# SPI Intermediate-Level Quick Revision

| Topic                  | Key Point                                                       |
| ---------------------- | --------------------------------------------------------------- |
| DMA                    | Moves SPI data between peripheral and RAM with low CPU overhead |
| TX DMA                 | RAM → SPI TX                                                    |
| RX DMA                 | SPI RX → RAM                                                    |
| DMA Complete           | Interrupt/event after transfer                                  |
| Multi-byte Transaction | CS remains asserted across the device-defined sequence          |
| Read                   | Often uses dummy TX bytes                                       |
| QSPI                   | Up to 4 data bits/clock in quad phases                          |
| Dual SPI               | Up to 2 data bits/clock in dual phases                          |
| SPI Slave              | External master provides CS and SCLK                            |
| Slave Challenge        | Must have TX data ready in time                                 |
| Debugging              | Logic analyzer + oscilloscope                                   |
| Common Failure         | Wrong CPOL/CPHA                                                 |
| High-Speed Failure     | Setup/hold or signal integrity                                  |
| MISO Conflict          | Multiple slaves driving at once                                 |
| RX Overrun             | CPU/DMA fails to consume received data                          |

---

# Easy Memory Trick

For DMA:

```text
TX:
RAM → DMA → SPI → MOSI

RX:
MISO → SPI → DMA → RAM
```

For QSPI:

```text
SPI   → 1 bit
Dual  → 2 bits
Quad  → 4 bits
```

For debugging:

```text
CS
 ↓
Clock
 ↓
Mode
 ↓
MOSI
 ↓
MISO
 ↓
Command
 ↓
Timing
 ↓
Signal Integrity
```

For SPI failure:

```text
Wrong Mode
Wrong Speed
Wrong CS
Wrong Command
Wrong Bit Order
Bad Signal
```

---

# Senior-Level Mental Model

A production SPI architecture often looks like:

```text
                    Application
                         ↓
                  Device Driver
                         ↓
              ┌──────────┴──────────┐
              ↓                     ↓
           Polling               DMA/IRQ
                                    ↓
                              SPI Peripheral
                                    ↓
                         ┌──────────┼──────────┐
                         ↓          ↓          ↓
                        SCLK       MOSI       MISO
                         │          │          │
                         └──────────┼──────────┘
                                    ↓
                                    CS
                                    ↓
                              SPI Peripheral
```

For a real transaction:

```text
CS LOW
   ↓
Command
   ↓
Address
   ↓
Dummy / Data
   ↓
CS HIGH
```

For debugging:

```text
Software says:
"Send 03 10 FF FF"

        ↓

Logic Analyzer:
"Did the pins actually send
03 10 FF FF?"

        ↓

Oscilloscope:
"Was the electrical waveform
clean enough for the device?"

        ↓

Device Datasheet:
"Was the command/timing
actually valid?"
```

> **The senior-level SPI skill is being able to move from a driver API to the actual clock/data waveform and then to the slave's command protocol. A clean waveform alone is not enough; the mode, CS timing, command sequence, and electrical timing must all be correct.**

---

## Advanced Level

### 15. How do you handle variable-length SPI transactions?

SPI itself transfers data based on **clock pulses**, so the master decides how many clock cycles to generate.

Different SPI devices may have transactions such as:

```text
Command
+
Address
+
Dummy Cycles
+
Variable Number of Data Bytes
```

For example:

```text id="xqoz8f"
Transaction A:

Command = 1 byte
Address = 1 byte
Data    = 2 bytes
```

while another transaction may be:

```text id="32pu9s"
Transaction B:

Command = 1 byte
Address = 3 bytes
Data    = 64 bytes
```

The driver therefore needs to handle different transfer lengths.

---

### Example: Write Transaction

```text id="a5prj4"
CS LOW
   ↓
Command     → 1 byte
   ↓
Address     → 2 bytes
   ↓
Data        → 4 bytes
   ↓
CS HIGH
```

Total clocks:

```text id="8mqil7"
1 + 2 + 4 = 7 bytes
```

---

### Example: Read Transaction

Suppose the device requires:

```text id="0y8m6d"
Command     = 1 byte
Address     = 2 bytes
Dummy       = 1 byte
Data        = 16 bytes
```

The master must generate:

```text id="24ke0h"
1 + 2 + 1 + 16
=
20 byte-times of clock
```

Conceptually:

```text id="4nny2x"
MOSI:

[CMD] [ADDR] [ADDR] [DUMMY] [DUMMY] [DUMMY] ...
                                  ↑
                            Data being read

MISO:

[--]  [--]    [--]    [--]   [D0] [D1] [D2] ...
```

The dummy bytes are simply there to generate clock pulses.

---

### Why are dummy bytes required?

SPI does not have a separate "read clock."

The clock is generated by the master.

So if the master wants to receive:

```text id="u52ikf"
8 bits
```

it must generate:

```text id="z2xj3o"
8 clock pulses
```

The simplest way is to transmit a dummy byte:

```text id="bsx3ov"
MOSI = 0x00
```

or:

```text id="0xFF"
```

while the slave sends meaningful data on MISO.

---

### Variable TX and RX Lengths

This needs careful wording.

Because SPI is synchronous, the number of receive bits is fundamentally tied to the number of clock cycles generated by the transmitter.

So you normally cannot have:

```text id="l2gqhz"
RX = 16 bytes
```

with:

```text id="x0vfzi"
Only 8 bytes worth of clocks
```

There are not enough clock pulses.

Instead, the TX side may send:

```text id="3zkg6a"
Command
Address
Dummy bytes
```

while the RX side captures:

```text id="xgbi9d"
Meaningful response bytes
```

Therefore, conceptually:

```text id="fr4m9m"
TX:
[CMD][ADDR][DUMMY][DUMMY]

RX:
[-- ][ -- ][DATA ][DATA ]
```

---

### How does DMA handle this?

You can use DMA to transfer a larger TX stream while simultaneously collecting RX data.

Example:

```text id="vh4v0e"
TX Buffer:

+------+-------+-------+--------+
| CMD  | ADDR  | DUMMY | DUMMY  |
+------+-------+-------+--------+

RX Buffer:

+------+------+--------+--------+
| IGN  | IGN  | DATA0  | DATA1  |
+------+------+--------+--------+
```

The DMA engine transfers based on the generated SPI clocks.

Some MCU peripherals/DMA controllers allow flexible source/destination configurations, but the exact TX/RX length behavior is **MCU-specific**.

Do not assume that every SPI peripheral can independently run arbitrary TX and RX DMA lengths.

---

### Common Driver Design

A useful API can separate the phases:

```c
spi_transaction(
    tx_cmd,
    cmd_len,
    tx_data,
    tx_len,
    rx_data,
    rx_len
);
```

Internally:

```text id="qv55x1"
CS LOW
   ↓
Send Command
   ↓
Send Address
   ↓
Send Dummy Cycles
   ↓
Receive Data
   ↓
CS HIGH
```

This is often much cleaner than treating every transaction as simply:

```text
spi_transfer(buffer, length)
```

---

### Variable-Length Example

Suppose a Flash read requires:

```text id="g3wxez"
Command = 0x03
Address = 0x123456
Data = 4 bytes
```

The transfer is:

```text id="2sj0lu"
CS LOW

TX:
03 12 34 56 FF FF FF FF

RX:
-- -- -- -- D0 D1 D2 D3

CS HIGH
```

The master generated:

```text id="h2fh5v"
8 bytes × 8 clocks
=
64 clock pulses
```

while only the last four received bytes may be meaningful.

---

### Senior-Level Consideration: CS Must Cover the Required Phases

A variable-length transaction does not mean you can arbitrarily toggle CS between phases.

For example, if the device requires:

```text id="3i8z3c"
CS LOW
Command
Address
Dummy
Data
CS HIGH
```

then the driver must keep CS asserted throughout.

The correct abstraction is:

```text id="c8y4p0"
Transaction Boundary
        =
      CS LOW
        ↓
All required phases
        ↓
      CS HIGH
```

---

### Interview Answer

> Variable-length SPI transactions are handled by generating exactly the number of clock cycles required by the device's command protocol. Read operations often include command, address, dummy cycles, and then variable-length data. Since SPI needs clock pulses to receive data, the master sends dummy bytes when it needs to clock data from the slave. DMA can be used to handle the transfer efficiently, but the exact TX/RX DMA capabilities depend on the MCU SPI and DMA hardware.

---

### 16. What is SPI bus contention?

**Bus contention** occurs when two or more devices try to drive the same electrical line in incompatible states at the same time.

There are two important situations to distinguish:

```text id="hr68wp"
1. Multiple SPI controllers/masters
2. Multiple SPI slaves driving a shared MISO line
```

---

### Multiple Masters

Unlike I2C, ordinary SPI does **not define a standard built-in multi-master arbitration mechanism**.

For example:

```text id="l2aaw1"
Master A ─────┐
              ├──── SPI BUS
Master B ─────┘
```

If both controllers try to drive:

```text id="g86s0z"
SCLK
MOSI
CS
```

at the same time:

```text id="kqkq4y"
Master A → HIGH
Master B → LOW
```

you can get:

```text id="vyh6ss"
Signal corruption
Electrical contention
Possible excess current
```

Therefore:

> **Standard SPI should not be treated as inherently multi-master safe.**

A multi-master architecture requires additional system-level arbitration or external hardware.

---

### Why is I2C different?

I2C was designed with:

```text id="j0m9i2"
Open-drain signaling
+
Bus arbitration
```

SPI generally does not provide the same standardized arbitration mechanism.

So:

```text id="h0xkqy"
I2C:
Multi-master arbitration built into protocol

SPI:
Multi-master requires additional design
```

---

### Bus Contention with Multiple Slaves

This is actually a very common SPI problem.

Suppose:

```text id="rld7y9"
Slave 1 MISO ──┐
Slave 2 MISO ──┼──> Master MISO
Slave 3 MISO ──┘
```

Normally:

```text id="vytv0o"
Selected slave → drives MISO
Other slaves  → High-Z
```

But if Slave 1 and Slave 2 both drive MISO:

```text id="29l7nh"
Slave 1 → HIGH
Slave 2 → LOW
```

you get contention.

---

### How do you prevent slave-side contention?

Use independent CS lines:

```text id="m9qv1g"
CS0 → Slave 0
CS1 → Slave 1
CS2 → Slave 2
```

and guarantee:

```text id="g34lhk"
Only ONE CS asserted
at a time
```

Conceptually:

```text id="o2z8px"
CS0 = LOW
CS1 = HIGH
CS2 = HIGH

→ Slave 0 active
```

Then:

```text id="d1uy5e"
CS0 = HIGH
CS1 = LOW
CS2 = HIGH

→ Slave 1 active
```

---

### What if an unselected slave does not release MISO?

This is a real hardware issue.

Some devices may:

```text id="0m3q5r"
Continue driving MISO
```

even when CS is inactive, depending on their implementation.

Possible solutions include:

```text id="6g64l4"
External tri-state buffer
Dedicated SPI bus
Different CS topology
Hardware mux
Device-specific configuration
```

Always check the slave datasheet for:

> **MISO behavior when CS is inactive.**

---

### How do you debug bus contention?

Use an oscilloscope or logic analyzer and check:

```text id="8s6m3k"
CS lines
MISO waveform
SCLK
```

If you see:

```text id="1dtio9"
MISO signal looks distorted
```

especially when changing between slaves, investigate whether more than one device is driving the line.

---

### Interview Answer

> SPI does not provide standardized built-in multi-master arbitration like I2C. Multiple masters therefore require external or system-level arbitration. In the more common multi-slave case, SCLK, MOSI, and MISO are shared, and only the selected slave should drive MISO. Bus contention occurs if multiple devices actively drive the same line at the same time. Proper CS management and confirming that inactive slaves release MISO are essential.

---

### 17. How do you implement multi-slave SPI?

The most common SPI multi-slave topology is:

```text id="g3ov0s"
              Master
                 │
       ┌─────────┼─────────┐
       │         │         │
      SCLK      MOSI      MISO
       │         │         │
       ├─────────┼─────────┤
       │         │         │
    Slave 1   Slave 2   Slave 3
       │         │         │
      CS0       CS1       CS2
```

So:

```text id="c8g1wi"
SCLK → Shared
MOSI → Shared
MISO → Shared
CS   → One dedicated line per slave
```

---

### Selecting Slave 1

```text id="r2qehh"
CS0 = LOW
CS1 = HIGH
CS2 = HIGH
```

Only Slave 1 participates.

```text id="z9h4el"
Master
   │
   ├── SCLK ─────────> All slaves
   ├── MOSI ─────────> All slaves
   ├── MISO <───────── Selected slave
   │
   └── CS0 = LOW
```

---

### Selecting Slave 2

```text id="7c0a6x"
CS0 = HIGH
CS1 = LOW
CS2 = HIGH
```

Now Slave 2 is selected.

---

### Why separate CS lines?

Because SPI does not normally send a device address on the bus.

Instead:

```text id="j2h84t"
CS0 → Device 0
CS1 → Device 1
CS2 → Device 2
```

The CS line effectively identifies which device should participate.

---

### Software Example

```c id="5x7j2n"
void spi_select_device(uint8_t device)
{
    /* First deselect all slaves */
    CS0_HIGH();
    CS1_HIGH();
    CS2_HIGH();

    switch (device)
    {
        case 0:
            CS0_LOW();
            break;

        case 1:
            CS1_LOW();
            break;

        case 2:
            CS2_LOW();
            break;

        default:
            break;
    }
}
```

Then:

```c id="n4h1kh"
spi_select_device(1);

spi_transfer(0x9A);
spi_transfer(0x10);

spi_deselect_all();
```

---

### Important CS Timing

A robust driver should ensure:

```text id="8a0w0v"
Deselect all
      ↓
Select target
      ↓
Small setup delay if required
      ↓
Transfer
      ↓
Wait for final clock to complete
      ↓
Deselect target
```

The delay and hold time are device-specific.

---

### Daisy-Chain SPI

Another topology is:

> **Daisy-chain SPI**

Instead of separate CS lines for every device, data can pass through the devices in series.

Conceptually:

```text id="8a9c8c"
                 ┌─────────┐
MOSI ───────────>│ Slave 1 │
                 └────┬────┘
                      │
                      ↓
                 ┌─────────┐
                 │ Slave 2 │
                 └────┬────┘
                      │
                      ↓
                 ┌─────────┐
                 │ Slave 3 │
                 └────┬────┘
                      │
                      ↓
                    MISO
                      │
                      ↓
                    Master
```

The output of one device feeds the input of the next.

---

### How does Daisy Chain work?

Suppose each slave has an 8-bit shift register.

Three slaves:

```text id="6v3h0p"
Slave 1 = 0x11
Slave 2 = 0x22
Slave 3 = 0x33
```

The master may need to send:

```text id="j2qz2k"
24 clock cycles
```

to shift data through all three devices.

The data moves:

```text id="1w7rfd"
Master
   ↓
Slave 1
   ↓
Slave 2
   ↓
Slave 3
   ↓
Master
```

So the total transfer length depends on the number of devices and their shift-register sizes.

---

### Advantage of Daisy Chain

It can reduce the number of CS/control lines.

Instead of:

```text id="0m7w3o"
3 devices
+
3 CS lines
```

you may have:

```text id="q9y2vn"
1 shared CS
```

depending on the device architecture.

---

### Disadvantage of Daisy Chain

Latency increases because data must pass through the chain.

Also:

```text id="xwl8k1"
Not every SPI device supports daisy chaining.
```

It is a device-specific feature.

If one device behaves incorrectly or is powered down in an incompatible way, the entire chain may be affected.

---

### Dedicated CS vs Daisy Chain

| Feature             | Separate CS    | Daisy Chain            |
| ------------------- | -------------- | ---------------------- |
| CS lines            | One per device | Potentially shared     |
| Data path           | Parallel       | Serial through devices |
| Latency             | Lower          | Higher                 |
| Device support      | Common         | Device-specific        |
| Software complexity | Simpler        | More complex           |
| Failure isolation   | Better         | More difficult         |

---

### Important Senior-Level Point

Do not assume:

> "SPI always requires one CS per slave."

That is the most common arrangement, but not the only possible architecture.

Other options include:

```text id="u2r8j9"
Dedicated CS
Daisy Chain
CS decoder
External mux
```

---

### Interview Answer

> The usual multi-slave SPI architecture shares SCLK, MOSI, and MISO while giving each slave a dedicated CS line. The master asserts only the target device's CS and keeps the others inactive, ensuring only one slave drives MISO. Some devices also support daisy-chain operation, where one device's serial output feeds the next device and the master transfers a longer combined shift-register sequence.

---

### 18. What are SPI timing constraints?

SPI timing constraints define **when data must be valid relative to the clock** so that the receiver can sample it reliably.

The most important parameters are:

```text id="g6w8z6"
Setup Time
Hold Time
Clock-to-Output Delay
Maximum Clock Frequency
CS Setup Time
CS Hold Time
```

---

### Setup Time

**Setup time** is the minimum time the data must be stable **before the sampling clock edge**.

Conceptually:

```text id="y7p0v3"
MOSI:

        ───────────────
              stable
        <---- tSU ---->

SCLK:

________________|‾‾‾‾
                ↑
            Sample edge
```

The data must arrive early enough before the sampling edge.

If:

```text id="w2l0v7"
t_setup < required value
```

the receiver may sample the wrong value.

---

### Hold Time

**Hold time** is the minimum time the data must remain stable **after the sampling clock edge**.

```text id="7e0v9m"
MOSI:

─────────|────────────
         <--- tH --->

SCLK:

_________|‾‾‾‾‾‾‾‾
         ↑
      Sample edge
```

If the transmitter changes data too quickly after the edge:

```text id="s4x8be"
Hold-time violation
       ↓
Potential wrong sample
```

---

### Clock-to-Output Delay

For a slave transmitting on MISO, the output does not necessarily change at the exact clock edge.

There can be propagation delay:

```text id="sdw4r3"
SCLK edge
   ↓
internal logic
   ↓
MISO changes
```

This is often specified as:

```text id="a2x3p4"
tCO
```

or:

```text id="3f4h5j"
Clock-to-output delay
```

Example:

```text id="j9k2p8"
SCLK edge
       ↓
       |---- tCO ----|
                    MISO changes
```

The master must sample at an appropriate point after the slave's output becomes valid.

---

### Maximum SPI Clock

The device datasheet may specify:

```text id="m7n8b9"
fSCLK(max)
```

For example:

```text id="2b3c4d"
Maximum SPI clock = 20 MHz
```

The master should operate at:

```text id="0e1f2g"
≤ 20 MHz
```

subject to all other timing and system constraints.

---

### CS Setup Time

Some devices require CS to be LOW for a minimum time **before the first clock edge**.

Conceptually:

```text id="h3i4j5"
CS:

────────\______________
         ↑
      CS active


SCLK:

               |‾|_|‾|
               ↑
             First edge


CS setup:
<--------->
```

This gives the device time to recognize the beginning of the transaction.

---

### CS Hold Time

Similarly, some devices require CS to remain active for a minimum time after the final clock edge.

```text id="k6l7m8"
SCLK:
_|‾|_|‾|_|‾|

              ↑
         Last clock edge

CS:
________________/‾‾‾‾
                ↑
        Wait required time
```

If CS rises too early, the final bit may not be correctly recognized.

---

### Why does CPOL/CPHA matter for timing?

Because CPOL/CPHA determine:

```text id="n9o0p1"
Which edge samples data
Which edge changes data
```

Therefore:

```text id="q2r3s4"
Wrong CPOL/CPHA
      ↓
Wrong sampling edge
      ↓
Setup/Hold violation
      ↓
Corrupted data
```

---

### Example of a Timing Problem

Suppose the slave requires:

```text id="t5u6v7"
Setup time = 5 ns
Hold time  = 2 ns
```

but the master changes data too close to the sampling edge:

```text id="w8x9y0"
Setup = 1 ns
```

Then:

```text id="z1a2b3"
Required = 5 ns
Actual   = 1 ns

       ↓

Timing violation
```

The communication may:

```text id="c4d5e6"
Work at low speed
Fail at high speed
```

This is a very common real-world symptom.

---

### Why does SPI sometimes work at 1 MHz but fail at 20 MHz?

Because increasing clock frequency reduces the available timing margin.

At:

```text id="f7g8h9"
1 MHz
```

there is a lot of time per bit.

At:

```text id="i0j1k2"
20 MHz
```

there is much less time.

Therefore:

```text id="l3m4n5"
Clock increases
      ↓
Bit period decreases
      ↓
Less timing margin
      ↓
Setup/Hold violations become more likely
```

Signal integrity problems also become more significant.

---

### Timing Diagram

A simplified SPI timing relationship is:

```text id="o6p7q8"
             Setup       Hold
              <---->     <-->
MOSI:  -------[ VALID DATA ]---------
                        

SCLK:  _____________|‾‾‾‾‾‾‾
                     ↑
                 Sample edge
```

The exact diagram depends on:

```text id="r9s0t1"
SPI Mode
Device datasheet
Sampling direction
```

---

### Where do you get these timing values?

From the **slave device datasheet**.

Look for parameters such as:

```text id="u2v3w4"
tSU
tH
tCO
tCS
tCH
fSCLK
```

The exact notation differs between manufacturers.

Never assume generic SPI timing values.

---

### Senior-Level SPI Timing Debug

Suppose:

```text id="x5y6z7"
SPI works at 10 MHz
SPI fails at 25 MHz
```

A good investigation is:

```text id="a8b9c0"
Check maximum slave clock
        ↓
Check CPOL / CPHA
        ↓
Check setup time
        ↓
Check hold time
        ↓
Check clock-to-output delay
        ↓
Check CS setup/hold
        ↓
Check signal integrity
```

Use:

```text id="d1e2f3"
Logic Analyzer
+
Oscilloscope
```

to compare the actual waveform against the datasheet timing requirements.

---

### Interview Answer

> SPI timing is defined by the relationship between SCLK, MOSI, MISO, and CS. The important parameters include data setup time before the sampling edge, hold time after the edge, slave clock-to-output delay, maximum SCLK frequency, and CS setup/hold requirements. These values are device-specific and must be checked in the slave datasheet. Increasing SPI frequency reduces timing margin, so a link that works at a low frequency can fail at a higher frequency due to timing or signal-integrity violations.

---

# SPI Advanced-Level Quick Revision

| Topic                    | Key Point                                            |
| ------------------------ | ---------------------------------------------------- |
| Variable-Length Transfer | Master generates the required number of clock cycles |
| Dummy Byte               | Used to generate clocks while receiving data         |
| TX/RX Length             | Coupled through generated clock cycles               |
| DMA                      | Efficient for large variable-length transfers        |
| Bus Contention           | Multiple devices drive the same line simultaneously  |
| Multi-Master SPI         | Requires external/system-level arbitration           |
| Shared MISO              | Only selected slave should drive it                  |
| Separate CS              | Most common multi-slave architecture                 |
| Daisy Chain              | Data shifts through multiple devices                 |
| Setup Time               | Data stable before sampling edge                     |
| Hold Time                | Data remains stable after sampling edge              |
| Clock-to-Output          | Delay from clock edge to slave output becoming valid |
| CS Setup                 | Time CS must be active before clock                  |
| CS Hold                  | Time CS must stay active after clock                 |
| Max SCLK                 | Defined by device/system limitations                 |

---

# Easy Memory Trick

### Variable-Length SPI

```text
Command
   ↓
Address
   ↓
Dummy
   ↓
Data
```

Remember:

> **Need data from slave? Generate clocks from master.**

---

### Multi-Slave SPI

```text
SCLK → Shared
MOSI → Shared
MISO → Shared
CS   → Dedicated
```

Remember:

> **One CS LOW → One slave active.**

---

### SPI Timing

```text
Before edge → Setup
At edge     → Sample
After edge  → Hold
```

And:

```text
SCLK Edge
   ↓
tCO
   ↓
MISO becomes valid
```

---

# Senior-Level Mental Model

For advanced SPI debugging, think of the complete timing chain:

```text
                 CS
                  │
                  ↓
             Transaction
                  │
          ┌───────┴───────┐
          ↓               ↓
        SCLK             MOSI
          ↓               ↓
     Sampling Edge    Data Setup
          │               │
          └───────┬───────┘
                  ↓
                MISO
                  ↓
              tCO Delay
                  ↓
            Master Samples
```

Then connect it to the device requirements:

```text
Driver
  ↓
SPI Peripheral
  ↓
CPOL / CPHA
  ↓
SCLK Frequency
  ↓
CS Timing
  ↓
Setup / Hold
  ↓
Signal Integrity
  ↓
Slave Device
```

> **At the advanced level, SPI debugging becomes a timing problem as much as a protocol problem. A transaction can contain the correct command bytes and still fail because the clock edge, CS timing, setup/hold margin, or slave output delay is outside the device's specification.**
