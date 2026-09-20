# Embedded Communication Protocols Interview Questions

> **Protocols:** UART, I2C, I3C, SPI, CAN, CANopen, Ethernet, Automotive Ethernet, 1-Wire, LIN, Modbus, etc.
> **Target:** M.Tech Graduate + 10 Years Experience
> **Level:** Basic → Intermediate → Advanced → Expert → War Stories

---

# I2C (Inter-Integrated Circuit)

I2C is one of the most commonly used communication buses in embedded systems for connecting a microcontroller to relatively low-speed peripheral devices.

Typical devices:

```text
MCU
 │
 ├── Temperature Sensor
 ├── EEPROM
 ├── RTC
 ├── IMU
 ├── PMIC
 ├── GPIO Expander
 └── Display Controller
```

The main advantage of I2C is that **multiple devices can share the same two communication lines**.

---

# Basic Level

### 1. What is I2C?

**I2C (Inter-Integrated Circuit)** is a synchronous, multi-device serial communication bus that normally uses only two signal lines:

```text
SDA → Serial Data
SCL → Serial Clock
```

A simple I2C bus looks like:

```text
              I2C BUS

          SDA ─────────────────────────
          SCL ─────────────────────────
               │       │       │
              MCU    Sensor   EEPROM
             Master   Slave    Slave
```

Unlike UART, I2C has a **clock line**.

```text
UART:
TX + RX
No clock

I2C:
SDA + SCL
Clock available
```

---

### What does SDA do?

`SDA` stands for:

> **Serial Data**

It carries:

```text
Address bits
Data bits
ACK/NACK
```

---

### What does SCL do?

`SCL` stands for:

> **Serial Clock**

The clock synchronizes data transfer between the devices.

Conceptually:

```text
SCL:  _|‾|_|‾|_|‾|_|‾|_

SDA:  __---____---_____
```

Data is transferred in synchronization with SCL.

---

### Is I2C full-duplex?

No.

I2C is generally **half-duplex / bidirectional on a shared SDA line**.

The same SDA line is used for:

```text
Master → Slave
```

and:

```text
Slave → Master
```

The master controls which direction the transaction takes.

For example:

```text
WRITE:

Master ─── SDA ───> Slave


READ:

Master <── SDA ─── Slave
```

---

### Is I2C multi-master?

Yes.

The I2C specification supports:

```text
Multiple Masters
+
Multiple Slaves
```

Example:

```text
        ┌─────────────┐
        │   I2C BUS   │
        └─────────────┘
          │    │    │
         MCU  MCU  Sensor
        Master Master Slave
```

However, many practical embedded systems use:

```text
1 Master
+
Multiple Slaves
```

because that is simpler.

---

### Why can multiple devices share SDA and SCL?

Because I2C uses **open-drain/open-collector style signaling**.

Devices generally do not actively drive the bus HIGH.

Instead:

```text
Device wants 0 → Pull line LOW

Device wants 1 → Release line
```

External pull-up resistors bring the line HIGH.

This is the foundation of I2C's shared-bus behavior.

---

### Common I2C Speeds

Typical I2C speed modes include:

```text
100 kHz  → Standard-mode
400 kHz  → Fast-mode
1 MHz    → Fast-mode Plus
3.4 MHz  → High-speed mode
```

There are additional specialized modes in the specification, but these are the commonly discussed ones in embedded interviews.

A useful memory trick:

```text
100k → Standard
400k → Fast
1M   → Fast Plus
3.4M → High Speed
```

---

### Where is I2C used?

I2C is commonly used when:

* Many peripherals need to share a bus.
* Data rate requirements are moderate.
* Pin count should be low.

Examples:

```text
I2C → EEPROM
I2C → RTC
I2C → Temperature Sensor
I2C → Pressure Sensor
I2C → IMU
I2C → PMIC
I2C → GPIO Expander
```

---

### I2C vs UART

A useful interview comparison:

| Feature    | UART                      | I2C                      |
| ---------- | ------------------------- | ------------------------ |
| Clock      | No separate clock         | SCL                      |
| Data lines | TX/RX                     | SDA                      |
| Devices    | Usually point-to-point    | Multiple devices         |
| Addressing | Usually no bus addressing | Yes                      |
| Duplex     | Full-duplex               | Shared bidirectional bus |
| Pull-ups   | Not normally required     | Required                 |
| Common use | Debug/modules             | Sensors/EEPROM/ICs       |

---

### Interview Answer

> I2C is a synchronous, serial, shared-bus communication protocol that normally uses two lines: SDA for data and SCL for clock. It supports multiple devices and can support multiple masters. I2C uses open-drain outputs with external pull-up resistors, which allows multiple devices to share the same bus. It is commonly used for sensors, EEPROMs, RTCs, PMICs, and other low-to-moderate-speed peripherals.

---

### 2. What is the I2C hardware or electrical structure?

The basic I2C bus looks like:

```text
                    VCC
                     │
                  [Pull-up]
                     │
SDA ─────────────────┼────────────────────
                     │
                  Devices
                     │
SCL ─────────────────┼────────────────────
                     │
                  [Pull-up]
                     │
                    VCC
```

More conceptually:

```text
              VCC
               │
              Rp
               │
SDA ───────────┼─────────────── SDA
               │
            Device A
            Device B
            Device C


              VCC
               │
              Rp
               │
SCL ───────────┼─────────────── SCL
               │
            Device A
            Device B
            Device C
```

Every device connects to the same:

```text
SDA
SCL
GND
```

---

### Why are pull-up resistors required?

This is one of the most important I2C interview questions.

I2C devices normally use open-drain outputs.

That means a device can:

```text
Pull LOW
```

but does not normally actively drive:

```text
HIGH
```

So where does HIGH come from?

The pull-up resistor.

```text
VCC
 │
Rp
 │
 ├──────── SDA
 │
Device
 │
Can pull LOW
```

Therefore:

```text
Device releases SDA
        ↓
Pull-up makes SDA HIGH
```

and:

```text
Device pulls SDA LOW
        ↓
SDA becomes LOW
```

---

### Why is open-drain useful?

Because multiple devices can safely share the bus.

Imagine:

```text
Device A → releases line
Device B → releases line
```

Then:

```text
SDA = HIGH
```

because of the pull-up.

Now:

```text
Device A → pulls LOW
Device B → releases line
```

Then:

```text
SDA = LOW
```

This creates a **wired-AND / dominant-low behavior**.

A device cannot force the bus HIGH against another device that is pulling it LOW.

That is extremely useful for:

```text
Bus sharing
ACK
Clock stretching
Arbitration
Multi-master operation
```

---

### Why can't we simply use push-pull outputs?

Suppose Device A drives:

```text
SDA = HIGH
```

and Device B drives:

```text
SDA = LOW
```

With push-pull outputs, the two outputs could fight each other electrically.

Conceptually:

```text
HIGH driver ───┐
               ├── SDA
LOW driver  ───┘

       ↓

Potential contention
```

I2C avoids this by having devices pull the bus LOW or release it.

---

### What value should the pull-up resistor have?

You will often hear values such as:

```text
2.2 kΩ
4.7 kΩ
10 kΩ
```

But there is **no universal value such as "4.7 kΩ for 5 V and 10 kΩ for 3.3 V."**

The correct pull-up value depends on:

```text
Bus capacitance
Bus speed
Number of devices
PCB trace length
Device sink-current capability
Required rise time
Supply voltage
```

For many ordinary designs:

```text
4.7 kΩ
```

is a reasonable starting point, but it must be checked against the electrical requirements.

At higher speeds or larger bus capacitance, a stronger pull-up may be needed.

---

### Why does bus capacitance matter?

When the device releases SDA:

```text
LOW → HIGH
```

the pull-up resistor charges the bus capacitance.

So the rising edge is not instantaneous.

Conceptually:

```text
LOW __________
             /
            /
           /
          /________ HIGH
```

A larger:

```text
Rpull-up
```

or:

```text
Bus capacitance
```

makes the rising edge slower.

At higher I2C speeds, slow rise time can cause communication failures.

This is why I2C is not simply:

> "Choose any pull-up resistor."

The resistor and bus capacitance must satisfy the required rise-time limits.

---

### What is clock stretching?

**Clock stretching** allows a slave to temporarily hold SCL LOW when it needs more time.

Normally:

```text
Master generates SCL
```

But a slave can say:

> "Wait, I am not ready yet."

by pulling:

```text
SCL = LOW
```

Conceptually:

```text
Master SCL:

‾|_|‾|_|‾|_|‾|_|‾


Slave stretches:

‾|_|‾|_|________|‾|_
             ↑
       SCL held LOW
```

The master must detect that SCL is still LOW and wait.

Once the slave releases SCL:

```text
SCL → HIGH
```

communication continues.

---

### Why is clock stretching useful?

A slave may need extra time because:

```text
Internal processing
ADC conversion
Sensor measurement
EEPROM operation
Slow internal hardware
```

Instead of losing synchronization, it can temporarily hold the bus.

---

### Is clock stretching supported by every I2C device?

No.

The controller/master and target/slave behavior must support it correctly.

This is especially important with:

* Some newer controllers
* Certain bridge chips
* FPGA implementations
* Bit-banged implementations

Always check the device datasheet/reference manual.

---

### Interview Answer

> I2C uses open-drain outputs and external pull-up resistors on SDA and SCL. Devices pull the lines LOW and release them for HIGH. This allows multiple devices to safely share the bus and enables features such as ACK/NACK, clock stretching, and multi-master arbitration. The pull-up resistor value is selected based on bus capacitance, speed, device sink capability, and required rise time; it is not a fixed value for a given supply voltage.

---

### 3. How does I2C addressing work?

Every I2C target on the bus normally has an address.

The most common format is:

```text
7-bit address
```

For example:

```text
0x50
```

The master uses that address to select a particular device.

Example:

```text
            I2C BUS
               │
       ┌───────┼────────┐
       │       │        │
     0x50    0x68     0x3C
    EEPROM    RTC     Display
```

The master can communicate with one target at a time by putting its address on the bus.

---

### How many 7-bit addresses are possible?

A 7-bit field can represent:

```text
2^7 = 128
```

possible values.

However, not all values are available for ordinary device assignment because certain address ranges are reserved for special purposes.

The commonly quoted number is:

```text
112 usable 7-bit addresses
```

with the remainder reserved.

Do not memorize:

> "All 128 addresses can be freely used."

That is incorrect.

---

### 7-bit Address + R/W Bit

A very common point of confusion is the address byte.

The logical device address is:

```text
7 bits
```

Then the transfer direction is specified by an additional:

```text
R/W bit
```

So the first transmitted byte is conceptually:

```text
┌───────────────┬──────┐
│  7-bit addr   │ R/W  │
└───────────────┴──────┘
       7 bits      1
```

Therefore:

```text
Total = 8 transmitted bits
```

followed by:

```text
9th clock → ACK/NACK
```

---

### What does the R/W bit mean?

```text
R/W = 0 → Write
R/W = 1 → Read
```

Example for device:

```text
7-bit address = 0x50
```

Shift left by one:

```text
0x50 << 1
=
0xA0
```

Then:

```text
Write:
0xA0 | 0 = 0xA0

Read:
0xA0 | 1 = 0xA1
```

So you may see:

```text
0xA0 → Write address byte
0xA1 → Read address byte
```

But remember:

> `0x50` is the **7-bit address**. `0xA0` and `0xA1` are the transmitted address bytes including the R/W bit.

This distinction is a very common interview question.

---

### Important Driver API Warning

Different I2C drivers use different address conventions.

One API may expect:

```text
7-bit address:
0x50
```

while another may expect:

```text
Shifted address:
0xA0
```

Therefore:

> Always check whether the driver API expects the raw 7-bit address or the already-shifted address.

Don't blindly shift an address twice.

---

### 10-bit Addressing

I2C also supports:

```text
10-bit addressing
```

This is useful when the address space needs to be larger.

The transfer format is more complex and uses special address prefix bits.

At interview level, remember:

```text
7-bit = common
10-bit = supported by I2C for larger address space
```

---

### Special Addresses

I2C reserves certain address ranges for special purposes.

For example:

```text
0x00
```

is associated with the **General Call** mechanism.

Certain upper ranges are also reserved.

Therefore:

> Not every 7-bit value should be treated as a normal slave address.

---

### What is General Call?

General Call allows a master to address multiple devices using the reserved general-call address mechanism.

A commonly known use is:

```text
Address = 0x00
```

The exact command behavior depends on whether the slave supports General Call.

---

### Interview Answer

> I2C normally uses a 7-bit target address, followed by a separate R/W bit in the first transmitted byte. `R/W = 0` indicates a write and `R/W = 1` indicates a read. Although 128 values are possible with 7 bits, some values are reserved, leaving fewer addresses available for normal device assignment. I2C also supports 10-bit addressing.

---

### 4. What is the I2C protocol frame?

A typical I2C transaction looks like:

```text
[START]
   ↓
[ADDRESS + R/W]
   ↓
[ACK]
   ↓
[DATA]
   ↓
[ACK]
   ↓
[DATA]
   ↓
[ACK]
   ↓
[STOP]
```

For example:

```text
S
│
├── Address + W
│
├── ACK
│
├── Register
│
├── ACK
│
├── Data
│
├── ACK
│
└── P
```

Where:

```text
S = START
P = STOP
```

---

### What is a START condition?

A START occurs when:

```text
SDA goes HIGH → LOW
```

while:

```text
SCL is HIGH
```

Conceptually:

```text
SCL:  ────────────────
SDA:  ────────┐______
               ↑
             START
```

This tells all devices:

> "A new I2C transaction is beginning."

---

### What is a STOP condition?

A STOP occurs when:

```text
SDA goes LOW → HIGH
```

while:

```text
SCL is HIGH
```

Conceptually:

```text
SCL:  ────────────────
SDA:  _______┐───────
             ↑
            STOP
```

This tells the bus:

> "The current transaction has ended."

---

### Why are START and STOP special?

During normal data transfer:

```text
SDA should normally change while SCL is LOW.
```

START and STOP are exceptions.

```text
START:
SDA changes HIGH → LOW while SCL HIGH

STOP:
SDA changes LOW → HIGH while SCL HIGH
```

This allows devices to recognize bus control conditions.

---

### What is ACK?

After every 8 transmitted bits, the receiver gets a 9th clock bit for ACK/NACK.

For ACK:

```text
Receiver pulls SDA LOW
```

So:

```text
8 data/address bits
        ↓
9th clock
        ↓
ACK = SDA LOW
```

The important point is:

> **The transmitter releases SDA, and the receiver controls the ACK bit.**

---

### What is NACK?

For NACK:

```text
Receiver leaves/releases SDA HIGH
```

So:

```text
ACK  → SDA LOW
NACK → SDA HIGH
```

A NACK can mean different things depending on the phase of the transaction.

For example:

* Addressed target did not respond.
* Target cannot accept more data.
* Master is finished reading.

---

### Why does the master send NACK after the last read byte?

Suppose the master wants to read:

```text
2 bytes
```

The slave sends:

```text
Byte 1
Byte 2
```

After Byte 1, the master sends:

```text
ACK
```

This means:

> "I want more."

After Byte 2, the master sends:

```text
NACK
```

This means:

> "I am finished."

Then the master generates:

```text
STOP
```

So:

```text
Read Byte 1 → ACK
Read Byte 2 → NACK
STOP
```

This is extremely important to remember.

---

### Does the master or slave generate SCL?

Normally:

```text
Master → Generates SCL
```

The master controls the clock.

However, with clock stretching:

```text
Slave → Can hold SCL LOW
```

So the master provides the clock timing, while a slave can temporarily delay progress.

---

### Interview Answer

> An I2C transaction begins with a START condition, followed by the target address and R/W bit. The receiver acknowledges each byte using the ninth clock cycle. Data bytes are then transferred with ACK/NACK after each byte, and the transaction ends with a STOP condition or another START condition. START is SDA falling while SCL is HIGH, and STOP is SDA rising while SCL is HIGH.

---

### 5. How do you transmit data over I2C? – Write Operation

Suppose:

```text
7-bit device address = 0x50
Data = 0xAA
```

The sequence is:

```text
START
  ↓
0x50 + WRITE
  ↓
ACK
  ↓
0xAA
  ↓
ACK
  ↓
STOP
```

Using a simplified software API:

```c
uint8_t addr = 0x50 << 1;
uint8_t data = 0xAA;

i2c_start();

i2c_send_byte(addr | 0);      /* Address + Write */

if (!i2c_receive_ack())
{
    goto error;
}

i2c_send_byte(data);

if (!i2c_receive_ack())
{
    goto error;
}

i2c_stop();
```

---

### What exactly goes onto the bus?

For:

```text
7-bit address = 0x50
```

the transmitted address byte for write is:

```text
0xA0
```

because:

```text
0x50 << 1 = 0xA0
```

and:

```text
R/W = 0
```

So the bus sequence is conceptually:

```text
START
→ 0xA0
→ ACK
→ 0xAA
→ ACK
→ STOP
```

---

### What does the ACK after the address mean?

If the slave pulls SDA LOW during the ACK bit:

```text
ACK
```

it means the addressed device responded.

If the master receives:

```text
NACK
```

possible reasons include:

```text
Wrong address
Device not powered
Device busy
Wrong bus configuration
Device does not support the transaction
```

At senior level, don't treat every NACK as the same error.

---

### Write Multiple Bytes

I2C allows multiple data bytes in the same transaction:

```text
START
 ↓
ADDRESS + W
 ↓
ACK
 ↓
DATA1
 ↓
ACK
 ↓
DATA2
 ↓
ACK
 ↓
DATA3
 ↓
ACK
 ↓
STOP
```

This is often more efficient than sending a separate START/STOP for every byte.

---

### Interview Answer

> For an I2C write, the master generates START, sends the target's 7-bit address with the R/W bit set to write, waits for ACK, and then sends one or more data bytes. Each transmitted byte is followed by an ACK/NACK from the receiver. The master ends the transaction with STOP.

---

### 6. How do you receive data over I2C? – Read Operation

Suppose:

```text
Device = 0x50
```

The master wants to read one byte.

A simplified sequence is:

```text
START
  ↓
0x50 + READ
  ↓
ACK
  ↓
DATA
  ↓
NACK
  ↓
STOP
```

Example:

```c
uint8_t addr = 0x50 << 1;
uint8_t data;

i2c_start();

i2c_send_byte(addr | 1);      /* Address + Read */

if (!i2c_receive_ack())
{
    goto error;
}

data = i2c_receive_byte();

i2c_send_nack();              /* No more data */

i2c_stop();
```

---

### Why does the master send NACK after receiving the last byte?

Because the master is the device controlling the read operation.

When it sends:

```text
ACK
```

it means:

> "Continue, I want another byte."

When it sends:

```text
NACK
```

it means:

> "This was the final byte; stop sending."

Therefore:

```text
Read:
Byte 1 → ACK
Byte 2 → ACK
Byte 3 → NACK
STOP
```

for a three-byte read.

---

### Multi-Byte Read

For example, reading 4 bytes:

```text
START
 ↓
ADDRESS + R
 ↓
ACK
 ↓
DATA1
 ↓
ACK
 ↓
DATA2
 ↓
ACK
 ↓
DATA3
 ↓
ACK
 ↓
DATA4
 ↓
NACK
 ↓
STOP
```

The last byte is normally followed by NACK.

---

### Important Direction Concept

During a read:

```text
Master
  ↓
Controls START and address

Slave
  ↓
Provides data

Master
  ↓
ACK/NACK controls whether more data is wanted
```

This is a very useful mental model.

---

### Interview Answer

> For an I2C read, the master generates START and sends the target address with the R/W bit set to read. The slave then provides the data bytes. The master sends ACK after bytes it wants to continue receiving and sends NACK after the final byte, followed by STOP.

---

### 7. What is an I2C Repeated START condition?

A **Repeated START** is another START condition generated by the master **without first generating a STOP**.

Instead of:

```text
START
 ↓
Transaction
 ↓
STOP
```

the master does:

```text
START
 ↓
Transaction
 ↓
REPEATED START
 ↓
Another transaction
 ↓
STOP
```

Conceptually:

```text
START
  ↓
Address + Write
  ↓
Register Address
  ↓
Repeated START
  ↓
Address + Read
  ↓
Data
  ↓
STOP
```

---

### Why is Repeated START useful?

The most common example is:

> **Read a register from an I2C peripheral.**

Typically, the master first needs to tell the slave:

```text
"I want register 0x10."
```

Then it wants:

```text
"Now give me the contents of register 0x10."
```

The transaction becomes:

```text
START
 ↓
DEVICE + WRITE
 ↓
REGISTER
 ↓
REPEATED START
 ↓
DEVICE + READ
 ↓
DATA
 ↓
NACK
 ↓
STOP
```

---

### Why not simply use STOP and START?

Sometimes you could.

But a repeated START allows the master to keep control of the bus without releasing it with a STOP.

This is useful because another master does not get an opportunity to take the bus between the write and read operation.

It also matches the transaction requirements of many I2C devices.

---

### Important Interview Point

Repeated START is **not a special electrical line**.

It is simply:

```text
Another START condition
```

without a preceding STOP.

---

### Write-Then-Read Example

```text
START
 ↓
0x50 + W
 ↓
ACK
 ↓
Register = 0x10
 ↓
ACK
 ↓
REPEATED START
 ↓
0x50 + R
 ↓
ACK
 ↓
DATA
 ↓
NACK
 ↓
STOP
```

This pattern appears constantly when working with:

```text
Sensors
EEPROMs
RTC
PMICs
IMUs
```

---

### Interview Answer

> A repeated START is a new START condition generated without first sending STOP. It allows the master to change the transfer direction or start a new addressed phase while retaining control of the bus. It is commonly used for register reads, where the master first writes the register address and then performs a read.

---

### 8. How do you read a register from an I2C device?

This is probably the most common practical I2C transaction.

Suppose:

```text
Device Address = 0x50
Register        = 0x10
```

The master wants:

```text
Contents of register 0x10
```

The typical sequence is:

```text
START
   ↓
DEVICE ADDRESS + WRITE
   ↓
ACK
   ↓
REGISTER ADDRESS
   ↓
ACK
   ↓
REPEATED START
   ↓
DEVICE ADDRESS + READ
   ↓
ACK
   ↓
DATA
   ↓
NACK
   ↓
STOP
```

---

### Code Example

```c
uint8_t data;

i2c_start();

/* Device address + Write */
i2c_send_byte((0x50 << 1) | 0);

if (!i2c_receive_ack())
{
    goto error;
}

/* Register address */
i2c_send_byte(0x10);

if (!i2c_receive_ack())
{
    goto error;
}

/* Repeated START */
i2c_start();

/* Device address + Read */
i2c_send_byte((0x50 << 1) | 1);

if (!i2c_receive_ack())
{
    goto error;
}

/* Receive register data */
data = i2c_receive_byte();

/* Last byte */
i2c_send_nack();

i2c_stop();
```

---

### Why do we perform a write before a read?

This often confuses beginners.

The first "write" does not necessarily mean:

> "I am writing data into the device."

Instead, the master is often writing the **register address it wants to access**.

So:

```text
WRITE
 ↓
"Select register 0x10"
```

Then:

```text
READ
 ↓
"Give me the contents of register 0x10"
```

This is why register reads are often called:

> **Write register address → Repeated START → Read data**

---

### Example with a Sensor

Suppose a temperature sensor has:

```text
Device address = 0x48
Temperature register = 0x00
```

The transaction could be:

```text
START
 ↓
0x48 + W
 ↓
ACK
 ↓
0x00
 ↓
ACK
 ↓
REPEATED START
 ↓
0x48 + R
 ↓
ACK
 ↓
Temperature MSB
 ↓
ACK
 ↓
Temperature LSB
 ↓
NACK
 ↓
STOP
```

Notice that the number of bytes read depends entirely on the device's register definition.

---

### What if the register is 16-bit?

Some devices use:

```text
8-bit register address
```

Others use:

```text
16-bit register address
```

For example:

```text
Register = 0x1234
```

might be sent as:

```text
0x12
0x34
```

The exact byte order is device-specific.

So always check the peripheral datasheet.

---

### What if there is no register address?

Some I2C devices don't use register-based access at all.

For example, a device may simply define:

```text
START
 ↓
ADDRESS + W
 ↓
DATA
 ↓
STOP
```

or:

```text
START
 ↓
ADDRESS + R
 ↓
DATA
 ↓
STOP
```

Therefore:

> **"Write register address, then read" is a very common I2C pattern, but it is not a mandatory feature of the I2C protocol itself.**

The device's datasheet defines the application-level transaction.

---

# I2C Basic-Level Quick Revision

| Topic            | Key Point                              |
| ---------------- | -------------------------------------- |
| I2C              | Synchronous shared serial bus          |
| Lines            | SDA + SCL                              |
| SDA              | Serial Data                            |
| SCL              | Serial Clock                           |
| Duplex           | Shared bidirectional data              |
| Devices          | Multiple masters/slaves supported      |
| Signaling        | Open-drain / open-collector style      |
| Pull-ups         | Required on SDA/SCL                    |
| Common Speeds    | 100 kHz, 400 kHz, 1 MHz, 3.4 MHz       |
| Address          | Usually 7-bit                          |
| R/W              | `0 = Write`, `1 = Read`                |
| START            | SDA HIGH→LOW while SCL HIGH            |
| STOP             | SDA LOW→HIGH while SCL HIGH            |
| ACK              | Receiver pulls SDA LOW                 |
| NACK             | Receiver releases SDA                  |
| Clock Stretching | Target holds SCL LOW                   |
| Repeated START   | START without preceding STOP           |
| Register Read    | Write register → repeated START → read |

---

# Complete I2C Register Read – Full Waveform

Let's take this real example:

```text
Device Address = 0x50
Register       = 0x10
Read Data      = 0xAB
```

The complete transaction is:

```text
START
  ↓
7-bit Address + WRITE
  ↓
ACK
  ↓
Register Address
  ↓
ACK
  ↓
REPEATED START
  ↓
7-bit Address + READ
  ↓
ACK
  ↓
Data from Slave
  ↓
NACK
  ↓
STOP
```

---

## Complete Transaction – Every Bit

```text
                 ADDRESS + W             REGISTER             ADDRESS + R             DATA
               7 bits + R/W              8 bits              7 bits + R/W            8 bits

              0 1 0 1 0 0 0 0         0 0 0 1 0 0 0 0      0 1 0 1 0 0 0 1       1 0 1 0 1 0 1 1
              └───────┬───────┘        └──────┬──────┘       └───────┬───────┘       └──────┬──────┘
                  0x50 + W                  0x10                  0x50 + R                 0xAB


SCL:

        ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐      ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐      ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐      ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
_______┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘______┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘______┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘______┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘____


SDA:

‾‾‾‾\______________________________________________________________________________________________
     ↑
   START

     1     0     1     0     0     0     0     0     0       0     0     0     1     0     0     0     0     1
     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑       ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑
    A6    A5    A4    A3    A2    A1    A0    R/W   ACK     R7    R6    R5    R4    R3    R2    R1    R0    ACK

                                                        ↓
                                                   REPEATED START

     1     0     1     0     0     0     0     1     0       1     0     1     0     1     0     1     1     1
     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑       ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑
    A6    A5    A4    A3    A2    A1    A0    R/W   ACK     D7    D6    D5    D4    D3    D2    D1    D0   NACK

                                                                                                                  ↑
                                                                                                                 STOP
```

The ASCII waveform above is conceptually showing the complete transaction. The exact electrical edge shape will depend on the bus and pull-up network.

---

# Now Let's Break Every Bit Down

## 1. START

Before communication:

```text
SCL = HIGH
SDA = HIGH
```

Master creates START by:

```text
SDA: HIGH → LOW
SCL: stays HIGH
```

```text
SCL: ‾‾‾‾‾‾‾‾
SDA: ‾‾‾‾\____
          ↑
        START
```

---

# 2. Address + Write

Device address:

```text
0x50
```

Binary 7-bit:

```text
0x50 = 1010000
```

So:

```text
A6 A5 A4 A3 A2 A1 A0

 1  0  1  0  0  0  0
```

Then the R/W bit:

```text
R/W = 0
```

because this is a WRITE operation.

Therefore the 8 transmitted bits are:

```text
1 0 1 0 0 0 0 0
```

or:

```text
10100000
```

which is:

```text
0xA0
```

### Important:

```text
7-bit address = 0x50
Address byte + Write = 0xA0
```

---

# 3. Address ACK

After the 8 address bits, the **9th clock** is the ACK bit.

The slave responds:

```text
ACK = 0
```

So the slave pulls SDA LOW.

```text
Address bits:
1 0 1 0 0 0 0 0

9th bit:
0 = ACK
```

Therefore:

```text
10100000 0
^^^^^^^^ ^ 
Address ACK
```

---

# 4. Register Address

We want register:

```text
0x10
```

Binary:

```text
00010000
```

Therefore the master sends:

```text
R7 R6 R5 R4 R3 R2 R1 R0

0  0  0  1  0  0  0  0
```

Then the slave sends ACK:

```text
0
```

So:

```text
00010000 0
^^^^^^^^ ^
Register ACK
```

---

# 5. Repeated START

The master does **not** send STOP.

Instead:

```text
SDA: HIGH → LOW
while SCL = HIGH
```

This creates a repeated START.

```text
Normal:

START → DATA → STOP


Here:

START → WRITE → REPEATED START → READ → STOP
```

---

# 6. Address + Read

Again the device address is:

```text
0x50
```

7-bit:

```text
1010000
```

But now:

```text
R/W = 1
```

because we want to READ.

Therefore:

```text
1 0 1 0 0 0 0 1
```

which is:

```text
10100001
```

or:

```text
0xA1
```

So:

```text
7-bit address = 0x50

Write byte = 0xA0
Read byte  = 0xA1
```

---

# 7. Read Address ACK

The slave recognizes its address and responds:

```text
ACK = 0
```

Therefore:

```text
10100001 0
^^^^^^^^ ^
Address  ACK
```

---

# 8. Data From Slave

Now the slave sends the requested register data.

We selected:

```text
Register = 0x10
```

Suppose its value is:

```text
0xAB
```

Binary:

```text
10101011
```

Bits:

```text
D7 D6 D5 D4 D3 D2 D1 D0

 1  0  1  0  1  0  1  1
```

So the slave sends:

```text
1 0 1 0 1 0 1 1
```

---

# 9. Master Sends NACK

The master only wanted one byte.

Therefore after receiving `0xAB`, the master sends:

```text
NACK = 1
```

Remember:

```text
ACK  = 0
NACK = 1
```

So:

```text
10101011 1
^^^^^^^^ ^
DATA    NACK
```

This tells the slave:

> "I don't want another byte."

---

# 10. STOP

Finally, the master generates STOP.

```text
SCL = HIGH
SDA: LOW → HIGH
```

```text
SCL: ‾‾‾‾‾‾‾
SDA: ____/‾‾‾
        ↑
       STOP
```

---

# Complete Bit-by-Bit Sequence

Now remember the **entire transaction as one sequence**:

```text
START

10100000 0
^^^^^^^^ ^
ADDR+W  ACK

00010000 0
^^^^^^^^ ^
REGISTER ACK

REPEATED START

10100001 0
^^^^^^^^ ^
ADDR+R  ACK

10101011 1
^^^^^^^^ ^
 DATA   NACK

STOP
```

Or in one line:

```text
START
→ 1 0 1 0 0 0 0 0
→ 0
→ 0 0 0 1 0 0 0 0
→ 0
→ REPEATED START
→ 1 0 1 0 0 0 0 1
→ 0
→ 1 0 1 0 1 0 1 1
→ 1
→ STOP
```

---

# Full Transaction Table

| Step | Bus Bits       | Meaning                   |
| ---- | -------------- | ------------------------- |
| 1    | START          | Master starts transaction |
| 2    | `1010000`      | 7-bit address = `0x50`    |
| 3    | `0`            | Write                     |
| 4    | `0`            | ACK                       |
| 5    | `00010000`     | Register = `0x10`         |
| 6    | `0`            | ACK                       |
| 7    | REPEATED START | Start read phase          |
| 8    | `1010000`      | 7-bit address = `0x50`    |
| 9    | `1`            | Read                      |
| 10   | `0`            | ACK                       |
| 11   | `10101011`     | Data = `0xAB`             |
| 12   | `1`            | NACK                      |
| 13   | STOP           | End transaction           |

---

# The Most Important I2C Waveform to Memorize

For a **register read**, memorize this:

```text
             WRITE                         READ

START
  │
  ▼
[ADDRESS 7-bit][W]
       │
       ▼
      ACK
       │
       ▼
[REGISTER ADDRESS]
       │
       ▼
      ACK
       │
       ▼
REPEATED START
       │
       ▼
[ADDRESS 7-bit][R]
       │
       ▼
      ACK
       │
       ▼
[DATA FROM SLAVE]
       │
       ▼
     NACK
       │
       ▼
      STOP
```

### One-line memory trick

```text
START
→ Address + W
→ ACK
→ Register
→ ACK
→ Repeated START
→ Address + R
→ ACK
→ Data
→ NACK
→ STOP
```

That is the waveform you should be able to **draw on a whiteboard in an interview**.


# Easy I2C Memory Trick

Remember I2C using:

```text
I2C
 │
 ├── 2 Wires
 │      ├── SDA → Data
 │      └── SCL → Clock
 │
 ├── Pull-ups
 │
 ├── Address
 │
 ├── START
 │
 ├── ACK
 │
 ├── DATA
 │
 └── STOP
```

For a register read:

```text
"Tell me what I want"
        ↓
WRITE register address
        ↓
REPEATED START
        ↓
"Now give it to me"
        ↓
READ data
```

---

# Senior-Level Mental Model

At a 10-year embedded level, think of I2C at three layers:

```text
┌───────────────────────────────┐
│ Application / Device Protocol │
│ Register Map / Commands       │
├───────────────────────────────┤
│ I2C Protocol                  │
│ Address / ACK / START / STOP  │
├───────────────────────────────┤
│ Electrical Layer              │
│ Open Drain / Pull-up / Rise   │
└───────────────────────────────┘
```

For example, when a sensor read fails:

```text
Application
     ↓
Wrong Register?
     ↓
I2C Transaction
     ↓
Wrong Address?
     ↓
ACK/NACK?
     ↓
START/STOP correct?
     ↓
SCL/SDA waveform?
     ↓
Pull-up / Voltage / Rise Time?
```

This is the mindset expected from a senior embedded engineer:

> **Don't treat I2C as only a sequence of bytes. Understand the protocol, the electrical behavior, and the device-specific transaction format together.**

---

# Important I2C Interview Traps

### Trap 1

> "I2C uses 4.7 kΩ pull-up resistors."

Not universally.

Better:

> "I2C requires pull-ups, and the value depends on bus capacitance, speed, rise-time requirements, and device sink capability."

### Trap 2

> "`0xA0` is the I2C address."

Not exactly.

If the device has:

```text
7-bit address = 0x50
```

then:

```text
Write address byte = 0xA0
Read address byte  = 0xA1
```

The actual device address is:

```text
0x50
```

### Trap 3

> "NACK always means an error."

No.

NACK can be normal.

For example:

```text
Last byte of a read
        ↓
Master sends NACK
        ↓
STOP
```

### Trap 4

> "I2C read always requires a register write first."

No.

That is **device-specific**.

Register-based devices commonly use:

```text
Write register
→ Repeated START
→ Read data
```

but the I2C protocol itself does not require registers.

### Trap 5

> "4.7 kΩ always works."

No.

At high speed or with a large capacitive bus, rise time may become too slow.

The pull-up must satisfy the electrical requirements of the bus.

---
# I2C – Intermediate Level

> **Target:** M.Tech Graduate + 10 Years Experience
> **Level:** Intermediate
> **Focus:** Bit-Banging, Clock Stretching, Timeout, STM32, Interrupts, Multi-Byte, Bus Recovery, Slave Mode

---

### 9. How do you implement I2C bit-banging (Software I2C)?

**Bit-banging** means implementing I2C in software using normal GPIO pins instead of using the MCU's dedicated I2C peripheral.

For example:

```text
MCU GPIO
   │
   ├── SDA
   └── SCL
```

The firmware manually generates:

```text
START
STOP
SCL pulses
SDA bits
ACK/NACK
```

This is useful when:

* The MCU has no hardware I2C.
* The required I2C pins are not available.
* You need a custom implementation.
* You need to recover a stuck bus.
* You need special timing behavior.

---

### Important: I2C GPIO must behave like open-drain

This is one of the most important points.

For I2C, the software should generally implement:

```text
Drive LOW
+
Release line
```

rather than:

```text
Drive HIGH
```

Conceptually:

```text
LOW:
GPIO actively pulls line LOW

HIGH:
GPIO is released
      ↓
External pull-up makes line HIGH
```

So a function called:

```c
i2c_scl_high();
```

should ideally mean:

> **Release SCL**, not actively drive it HIGH.

---

### Basic GPIO Implementation

A simplified example:

```c
#define SCL_PORT    GPIOB
#define SCL_PIN     GPIO_PIN_10

#define SDA_PORT    GPIOB
#define SDA_PIN     GPIO_PIN_11
```

SCL LOW:

```c
static void i2c_scl_low(void)
{
    GPIO_ResetBits(SCL_PORT, SCL_PIN);
}
```

SCL HIGH means release:

```c
static void i2c_scl_release(void)
{
    /*
     * Configure GPIO as input/open-drain release.
     * External pull-up brings the line HIGH.
     */
}
```

Reading SCL:

```c
static int i2c_scl_read(void)
{
    return GPIO_ReadInputDataBit(
        SCL_PORT,
        SCL_PIN
    );
}
```

Similarly for SDA:

```c
static void i2c_sda_low(void)
{
    GPIO_ResetBits(SDA_PORT, SDA_PIN);
}

static int i2c_sda_read(void)
{
    return GPIO_ReadInputDataBit(
        SDA_PORT,
        SDA_PIN
    );
}
```

The exact GPIO configuration depends on the STM32 family.

---

### How does bit-banging generate a START?

Recall:

```text
START:
SDA HIGH → LOW
while SCL HIGH
```

So conceptually:

```c
static void i2c_start(void)
{
    i2c_sda_release();
    i2c_scl_release();

    delay_us(5);

    /* SDA HIGH → LOW while SCL HIGH */
    i2c_sda_low();

    delay_us(5);

    /* Take SCL LOW */
    i2c_scl_low();
}
```

Waveform:

```text
SCL:  ‾‾‾‾‾‾‾‾\______

SDA:  ‾‾‾‾\___________
          ↑
        START
```

---

### How does bit-banging generate a STOP?

STOP is:

```text
SDA LOW → HIGH
while SCL HIGH
```

Conceptually:

```c
static void i2c_stop(void)
{
    i2c_sda_low();

    delay_us(5);

    i2c_scl_release();

    /*
     * Wait for SCL to actually become HIGH.
     */

    delay_us(5);

    i2c_sda_release();
}
```

Waveform:

```text
SCL:  ________‾‾‾‾‾‾‾

SDA:  _________/‾‾‾‾‾
             ↑
            STOP
```

---

### How do you send one bit?

For a `1`:

```text
Release SDA
```

For a `0`:

```text
Pull SDA LOW
```

Then generate an SCL pulse.

Conceptually:

```c
static void i2c_write_bit(uint8_t bit)
{
    if (bit)
    {
        i2c_sda_release();
    }
    else
    {
        i2c_sda_low();
    }

    delay_us(2);

    i2c_scl_release();

    /*
     * Important:
     * Wait for actual SCL HIGH here.
     * This handles clock stretching.
     */
    while (i2c_scl_read() == 0)
    {
        /* Wait / timeout */
    }

    delay_us(2);

    i2c_scl_low();
}
```

---

### How do you read one bit?

During a read, the master releases SDA so the slave can control it.

```c
static uint8_t i2c_read_bit(void)
{
    uint8_t bit;

    i2c_sda_release();

    delay_us(2);

    i2c_scl_release();

    while (i2c_scl_read() == 0)
    {
        /* Wait for slave to release SCL */
    }

    delay_us(2);

    bit = i2c_sda_read();

    i2c_scl_low();

    return bit;
}
```

The important concept is:

```text
Master releases SDA
        ↓
Slave controls SDA
        ↓
Master raises SCL
        ↓
Master samples SDA
```

---

### Important limitation of bit-banging

Software I2C is sensitive to:

```text
Interrupt latency
RTOS scheduling
CPU load
GPIO timing
Clock stretching
Critical sections
```

For example:

```text
I2C running
     ↓
Long interrupt occurs
     ↓
SCL/SDA timing disturbed
     ↓
Communication may fail
```

Therefore, hardware I2C is normally preferred when the peripheral is available and meets the system requirements.

---

### Interview Answer

> I2C bit-banging is a software implementation where GPIO pins are used to generate SDA and SCL transitions manually. The firmware implements START, STOP, bit transmission, ACK/NACK, and timing. Because I2C uses open-drain signaling, the software must pull the line LOW or release it rather than actively driving HIGH. A robust implementation must also account for clock stretching and timeouts.

---

### 10. What is I2C clock stretching?

**Clock stretching** allows an I2C target/slave to temporarily hold `SCL` LOW to delay the master.

Normally:

```text
Master generates SCL
```

But the target may need more time.

It can do:

```text
SCL = LOW
```

and keep it LOW until it is ready.

---

### Normal clock

```text
SCL:

____|‾‾|____|‾‾|____|‾‾|____|‾‾|____
```

---

### Clock stretching

Suppose the master wants SCL HIGH:

```text
Master releases SCL
```

but the slave holds it LOW:

```text
SCL:

____|‾‾|____|‾‾|________________|‾‾|____
                         ↑
                  Slave stretches
```

The master must wait.

---

### Why doesn't the master simply assume SCL went HIGH?

Because I2C uses open-drain behavior.

The master releases SCL:

```text
Master → release
```

But the line only becomes HIGH if:

```text
No device is pulling it LOW
```

So the master should check the **actual bus level**.

Conceptually:

```c
i2c_scl_release();

while (i2c_scl_read() == 0)
{
    /* Clock stretching */
}
```

---

### Why would a slave stretch the clock?

Possible reasons include:

```text
Internal processing
Sensor conversion
Data preparation
Slow hardware
Interrupt latency
```

For example:

```text
Master asks:
"Give me sensor result."

Slave:
"Wait, I am still calculating."

        ↓

Slave holds SCL LOW
        ↓
Master waits
```

---

### Important interview point

Clock stretching is **not the master intentionally slowing down the clock**.

It is:

> **The target holding SCL LOW after the controller has released it.**

---

### How should a robust driver handle it?

A good implementation:

```text
Release SCL
   ↓
Read SCL
   ↓
SCL HIGH?
 ┌──────┴──────┐
Yes           No
 ↓             ↓
Continue    Wait / timeout
```

Never write:

```c
while (SCL == 0)
{
}
```

without a timeout in production firmware.

Otherwise a broken device can hang the entire thread forever.

---

### Interview Answer

> Clock stretching is an I2C mechanism where the target holds SCL LOW after the controller releases it, indicating that the target needs more time. The controller must wait until SCL actually becomes HIGH before continuing. A robust driver should always place a timeout around this wait so a faulty device cannot block the bus forever.

---

### 11. What is an I2C timeout?

An **I2C timeout** prevents the software from waiting forever when the bus or target does not respond.

For example:

```c
i2c_scl_release();

while (i2c_scl_read() == 0)
{
    /* Wait */
}
```

This looks simple, but it has a serious problem.

If the slave permanently holds SCL LOW:

```text
while()
{
    forever
}
```

The firmware can become stuck.

---

### Why can an I2C bus get stuck?

Possible reasons:

```text
Slave crashed
MCU reset during transaction
Noise
Incomplete transaction
Clock stretching never released
Power glitch
Peripheral state-machine lockup
```

For example:

```text
MCU sends:

START
ADDRESS
DATA

        ↓

MCU resets unexpectedly

        ↓

Slave still thinks transaction is active
        ↓
SDA may remain LOW
        ↓
Bus stuck
```

---

### What should a timeout protect?

Timeouts are useful around operations such as:

```text
Waiting for SCL HIGH
Waiting for SDA state
Waiting for ACK
Waiting for peripheral flags
Waiting for STOP completion
Waiting for bus idle
```

Example:

```c
#define I2C_TIMEOUT 1000U

static int wait_for_scl_high(void)
{
    uint32_t timeout = I2C_TIMEOUT;

    i2c_scl_release();

    while (i2c_scl_read() == 0)
    {
        if (timeout == 0U)
        {
            return -1;
        }

        timeout--;
    }

    return 0;
}
```

In production code, a time-based timeout using a hardware timer or RTOS tick is usually better than assuming one loop iteration represents a fixed amount of time.

---

### What happens when timeout occurs?

A robust driver should:

```text
Detect timeout
      ↓
Abort current transaction
      ↓
Generate STOP if possible
      ↓
Attempt bus recovery
      ↓
Reset/reinitialize I2C peripheral if needed
      ↓
Return error to application
```

---

### I2C Bus Recovery

A common recovery technique is to manually toggle SCL up to **9 times** while monitoring SDA.

Why 9?

A typical stuck transaction may leave a slave partway through a byte:

```text
8 data bits
+
1 ACK bit
```

Clocking SCL can allow the slave to finish that partial transaction.

Conceptually:

```text
SCL:

_|‾|_|‾|_|‾|_|‾|_|‾|_|‾|_|‾|_|‾|_

 1   2   3   4   5   6   7   8   9
```

After that, the master can attempt to generate a STOP.

---

### Typical Recovery Sequence

```text
Bus stuck
   ↓
Check SCL/SDA
   ↓
Release SDA
   ↓
Toggle SCL up to 9 times
   ↓
Check SDA
   ↓
Generate STOP
   ↓
Check bus idle
   ↓
Reinitialize peripheral if necessary
```

This is one of the most useful real-world I2C recovery techniques.

---

### Important caveat

Clock toggling does not guarantee recovery from every failure.

For example:

```text
SDA physically shorted to GND
```

cannot be fixed by software.

Similarly, a powered-off or damaged device may require:

```text
Power cycle
Hardware reset
Peripheral reset
```

---

### Interview Answer

> An I2C timeout prevents the driver from waiting indefinitely for a bus condition such as SCL becoming HIGH, an ACK, or bus-idle state. When a timeout occurs, the driver should abort the transaction and attempt recovery. A common bus-recovery technique is to release SDA, toggle SCL up to nine times to allow a stuck target to complete its partial byte, then generate a STOP and reinitialize the controller if necessary.

---

### 12. How do you configure I2C on a microcontroller? – STM32 example

A typical STM32 I2C configuration includes:

```text
I2C peripheral clock
GPIO configuration
SDA/SCL alternate function
Pull-ups
Clock speed
Addressing mode
Own address
ACK configuration
I2C peripheral enable
```

A classic STM32 Standard Peripheral Library example is:

```c
I2C_InitTypeDef I2C_InitStructure;

I2C_InitStructure.I2C_ClockSpeed = 100000;

I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;

I2C_InitStructure.I2C_OwnAddress1 = 0x50;

I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;

I2C_InitStructure.I2C_AcknowledgedAddress =
    I2C_AcknowledgedAddress_7bit;

I2C_Init(I2C1, &I2C_InitStructure);

I2C_Cmd(I2C1, ENABLE);
```

---

### What does each setting mean?

#### Clock Speed

```c
I2C_InitStructure.I2C_ClockSpeed = 100000;
```

This configures:

```text
100 kHz
```

which is Standard-mode I2C.

---

#### Mode

```c
I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
```

This selects the I2C peripheral mode.

Some STM32 peripheral generations have additional SMBus-related modes.

---

#### Own Address

```c
I2C_InitStructure.I2C_OwnAddress1 = 0x50;
```

This is important:

> **Own address is the address used when the MCU itself operates as an I2C slave/target.**

It is **not** the address of the external sensor/EEPROM you want to communicate with as controller/master.

For example:

```text
MCU = I2C Controller
Sensor = 0x48
EEPROM = 0x50
```

The controller uses:

```text
0x48
0x50
```

when addressing those devices.

Its own address matters when the MCU is also acting as a target.

---

#### ACK Enable

```c
I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;
```

This enables ACK behavior in the peripheral.

The exact ACK control behavior depends on whether the MCU is transmitting or receiving and on the STM32 peripheral state.

---

#### Addressing Mode

```c
I2C_InitStructure.I2C_AcknowledgedAddress =
    I2C_AcknowledgedAddress_7bit;
```

This selects:

```text
7-bit addressing
```

instead of 10-bit addressing.

---

### GPIO configuration is equally important

I2C configuration isn't complete just because `I2C_Init()` was called.

You also need:

```text
SCL → Correct GPIO
SDA → Correct GPIO
       ↓
Correct alternate function
       ↓
Open-drain configuration
       ↓
Pull-ups
```

Conceptually:

```text
MCU
 │
 ├── SCL ──────────────┐
 │                     │
 └── SDA ──────────────┤
                       │
                    I2C BUS
                       │
                 ┌─────┴─────┐
                 │  Sensor   │
                 └───────────┘
```

---

### Important STM32 version difference

The example above uses the **legacy STM32 Standard Peripheral Library** style.

Modern STM32 families commonly use:

```text
HAL
LL
```

instead.

So a current STM32 project may look more like:

```c
hi2c1.Init.Timing = ...;
hi2c1.Init.OwnAddress1 = 0;
hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
hi2c1.Init.OwnAddress2 = 0;
hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
```

The exact initialization depends heavily on the STM32 family.

For an interview, focus on the concepts rather than memorizing one HAL structure.

---

### Interview Answer

> To configure I2C, I would first configure the I2C peripheral clock and SDA/SCL GPIO alternate functions with the required open-drain behavior and external pull-ups. Then I would configure bus speed, addressing mode, ACK behavior, and own address if the MCU must act as a target. Finally I would enable the peripheral and verify the bus electrically. The exact initialization API differs between STM32 families and between the legacy Standard Peripheral Library, HAL, and LL drivers.

---

### 13. How do you use I2C with interrupts?

Instead of continuously polling the I2C peripheral:

```text
CPU
 ↓
Check status
 ↓
Check status
 ↓
Check status
```

we can use interrupts.

The peripheral generates an interrupt when an important bus event occurs.

Conceptually:

```text
I2C Peripheral
      ↓
Event
      ↓
Interrupt
      ↓
ISR
      ↓
Update I2C state machine
```

---

### STM32 Interrupt Categories

With older STM32 I2C peripherals using the Standard Peripheral Library, you may see:

```text
I2C_IT_EVT
I2C_IT_BUF
I2C_IT_ERR
```

Conceptually:

```text
EVENT
 ↓
START sent
Address phase complete
TX complete
RX event
STOP

BUFFER
 ↓
Data register ready

ERROR
 ↓
Bus error
Arbitration lost
ACK failure
Overrun
Timeout
```

The exact event/error flags vary by STM32 family.

---

### Why is I2C usually implemented as a state machine?

Because an I2C transaction has many sequential steps.

For example:

```text
IDLE
 ↓
START
 ↓
ADDRESS
 ↓
ADDRESS ACK
 ↓
DATA TX
 ↓
DATA ACK
 ↓
RESTART
 ↓
ADDRESS + READ
 ↓
DATA RX
 ↓
NACK
 ↓
STOP
 ↓
COMPLETE
```

An interrupt-driven driver can move from one state to another based on hardware events.

Conceptually:

```c
switch (i2c_state)
{
    case I2C_STATE_START:
        /* handle START */
        break;

    case I2C_STATE_ADDR:
        /* handle address */
        break;

    case I2C_STATE_TX:
        /* send data */
        break;

    case I2C_STATE_RX:
        /* receive data */
        break;

    case I2C_STATE_STOP:
        /* finish transaction */
        break;

    default:
        break;
}
```

---

### Why is a state machine better than a huge ISR?

Because I2C transactions are asynchronous from the CPU's point of view.

The driver may need to:

```text
Wait for hardware
↓
Return from ISR
↓
Wait for next event
↓
Continue transaction
```

A state machine makes this easier to maintain.

---

### What should the ISR do?

Generally:

```text
Read status
   ↓
Identify event/error
   ↓
Move state machine
   ↓
Read/write data register
   ↓
Signal task if transaction completed
```

Avoid:

```text
Long delays
printf()
Large parsing
Blocking waits
```

inside the ISR.

---

### RTOS Architecture

A good architecture is:

```text
Application Task
      ↓
I2C Driver API
      ↓
Start Transaction
      ↓
I2C ISR
      ↓
State Machine
      ↓
Transaction Complete
      ↓
Semaphore/Event
      ↓
Application Task
```

For example:

```text
Task:
i2c_read_register(...)

        ↓

I2C Hardware

        ↓

ISR handles transaction

        ↓

Semaphore given

        ↓

Task wakes up
```

This avoids blocking inside the interrupt context.

---

### Interview Answer

> I2C can be handled using interrupts so the CPU doesn't continuously poll the peripheral. The ISR detects events such as START completion, address acknowledgment, transmit/receive readiness, and error conditions, then advances an I2C state machine. In an RTOS, the ISR can signal the waiting task using a semaphore, event, or notification when the transaction finishes.

---

### 14. How do you perform a multi-byte I2C transaction?

I2C supports sending multiple data bytes in a single transaction.

For example:

```text
Data:
0xBB
0xCC
0xDD
```

The sequence is:

```text
START
 ↓
ADDRESS + WRITE
 ↓
ACK
 ↓
0xBB
 ↓
ACK
 ↓
0xCC
 ↓
ACK
 ↓
0xDD
 ↓
ACK
 ↓
STOP
```

---

### Code Example

```c
i2c_start();

i2c_send_byte((0x50 << 1) | 0);   /* Address + Write */
i2c_receive_ack();

i2c_send_byte(0xBB);
i2c_receive_ack();

i2c_send_byte(0xCC);
i2c_receive_ack();

i2c_send_byte(0xDD);
i2c_receive_ack();

i2c_stop();
```

---

### Why send multiple bytes in one transaction?

Because it is more efficient than:

```text
START
ADDRESS
DATA
STOP

START
ADDRESS
DATA
STOP

START
ADDRESS
DATA
STOP
```

Instead:

```text
START
ADDRESS
DATA1
DATA2
DATA3
STOP
```

This reduces protocol overhead.

---

### Auto-Increment Registers

Many devices support an internal register pointer that automatically increments.

For example:

```text
Start register = 0x10

Write:
0xAA → register 0x10
0xBB → register 0x11
0xCC → register 0x12
```

So:

```text
START
 ↓
ADDRESS + W
 ↓
REGISTER = 0x10
 ↓
ACK
 ↓
0xAA
 ↓
ACK
 ↓
0xBB
 ↓
ACK
 ↓
0xCC
 ↓
ACK
 ↓
STOP
```

But:

> Auto-increment is **device-specific**, not an I2C feature.

---

### What if the slave NACKs one byte?

A robust driver should detect it:

```text
DATA
 ↓
NACK
 ↓
Abort transaction
 ↓
Generate STOP
 ↓
Return error
```

Don't blindly continue transmitting.

---

### Multi-byte Read

For reading several bytes:

```text
START
 ↓
ADDRESS + READ
 ↓
ACK
 ↓
DATA1
 ↓
ACK
 ↓
DATA2
 ↓
ACK
 ↓
DATA3
 ↓
NACK
 ↓
STOP
```

The master ACKs intermediate bytes and NACKs the final byte.

---

### Interview Answer

> A multi-byte I2C transaction sends or receives several bytes between a single START and STOP, with an ACK after each transmitted byte. This reduces bus overhead and is commonly used for register blocks, configuration tables, EEPROM accesses, and sensor data. Whether the device automatically increments its internal register address is defined by the device protocol, not by I2C itself.

---

### 15. How do you detect that the I2C bus is stuck?

A bus is considered stuck when it cannot return to an idle state or a transaction cannot make progress.

For an idle I2C bus:

```text
SDA = HIGH
SCL = HIGH
```

So a common indication is:

```text
SDA = LOW
SCL = HIGH
```

This often means a device is holding SDA LOW.

But don't rely only on this one condition.

Examples:

```text
SDA LOW + SCL HIGH
```

can indicate a slave is waiting to finish a transaction.

Or:

```text
SDA LOW + SCL LOW
```

can indicate that SCL is also being held low.

---

### How do you recover the bus?

A common recovery sequence is:

```text
1. Disable hardware I2C / switch to GPIO if required
2. Release SDA
3. Toggle SCL up to 9 times
4. Check whether SDA becomes HIGH
5. Generate STOP
6. Check that SDA and SCL are HIGH
7. Reinitialize I2C peripheral
```

---

### Why toggle SCL 9 times?

Because a target may be stuck partway through:

```text
8-bit data transfer
+
ACK
```

Clocking SCL gives it an opportunity to advance its internal state machine.

Example:

```text
SCL pulses:

1   2   3   4   5   6   7   8   9
↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑
```

After recovery, the controller can attempt a STOP condition.

---

### Recovery Example

Conceptually:

```c
for (uint8_t i = 0; i < 9; i++)
{
    i2c_scl_release();
    delay_us(5);

    if (i2c_sda_read())
    {
        /*
         * SDA has been released.
         * We may already be recovered.
         */
        break;
    }

    i2c_scl_low();
    delay_us(5);
}
```

Then:

```text
Generate STOP
      ↓
SDA HIGH
SCL HIGH
      ↓
Bus idle?
```

---

### Important Real-World Point

A stuck bus is not always caused by firmware.

Possible causes include:

```text
Slave power loss
MCU reset mid-transaction
Noise
Clock stretching stuck
Hardware fault
Wrong GPIO state
Short circuit
Pull-up problem
```

For example:

```text
SDA physically shorted to GND
```

cannot be solved by repeatedly toggling SCL.

---

### Logic Analyzer / Oscilloscope

When investigating a stuck bus, check:

```text
SDA
SCL
START
STOP
ACK
Clock stretching
```

This can tell you which device is holding the bus.

For example:

```text
SCL = HIGH
SDA = LOW
```

and the logic analyzer shows:

```text
Slave address
partial data
then permanent LOW
```

This provides strong evidence that the target may have been left in the middle of a transaction.

---

### Interview Answer

> An idle I2C bus should have both SDA and SCL HIGH. If one line remains LOW and the controller cannot make further progress, the bus may be stuck. A common recovery mechanism is to release SDA, toggle SCL up to nine times to allow a target to finish a partial transaction, then generate a STOP and verify that both lines return HIGH. If that fails, the peripheral or target may need to be reset or power-cycled.

---

### 16. What is I2C slave mode?

In I2C terminology, a device can act as either:

```text
Controller / Master
```

or:

```text
Target / Slave
```

Older terminology commonly uses:

```text
Master / Slave
```

while newer I2C documentation often prefers:

```text
Controller / Target
```

For interview purposes, understand both terms.

---

### MCU as I2C Slave

Normally you may have:

```text
MCU ──────> Sensor
Master       Slave
```

But the MCU itself can also operate as a slave:

```text
External MCU
     │
     │ I2C
     ▼
Your MCU
Target
```

The external controller initiates the transaction.

Your MCU responds to its address.

---

### What happens when the address matches?

Suppose your MCU has:

```text
Own address = 0x30
```

The controller sends:

```text
START
 ↓
0x30 + WRITE
```

The MCU detects:

```text
Address Match
```

and then:

```text
ACK
```

It can then receive data.

---

### Slave Receive

Example:

```text
External Controller
        ↓
Address + Write
        ↓
     MCU Slave
        ↓
      ACK
        ↓
      Data
        ↓
      ACK
```

The slave receives the bytes and processes them.

---

### Slave Transmit

For a read:

```text
External Controller
        ↓
Address + Read
        ↓
     MCU Slave
        ↓
      ACK
        ↓
       DATA
        ↓
      DATA
        ↓
     Controller ACK/NACK
```

The slave must provide data when requested.

---

### Important: Who controls the transaction?

The controller/master controls:

```text
START
SCL
STOP
Addressing
Read/write direction
```

The target/slave responds.

So:

```text
Master → controls bus
Slave  → responds
```

---

### Typical Slave Events

A microcontroller configured as an I2C slave may need to handle:

```text
Address Match
RX Data Available
TX Request
STOP Detected
NACK
Error
```

For example:

```text
Address Match
      ↓
Determine R/W
   /       \
Write     Read
 ↓          ↓
Receive    Provide
data       data
```

---

### Example Slave Register Protocol

Suppose your MCU acts like a sensor with:

```text
Address = 0x30
Register 0x01 = Status
Register 0x02 = Temperature
```

External master could do:

```text
START
 ↓
0x30 + W
 ↓
ACK
 ↓
0x02
 ↓
ACK
 ↓
REPEATED START
 ↓
0x30 + R
 ↓
ACK
 ↓
Temperature
 ↓
NACK
 ↓
STOP
```

Your MCU's slave firmware would:

```text
Receive address
      ↓
Detect WRITE
      ↓
Receive register = 0x02
      ↓
Store register pointer
      ↓
Detect next READ
      ↓
Return temperature
```

This is exactly how many register-based I2C peripherals behave.

---

### Slave Mode in an RTOS

A possible architecture is:

```text
             I2C Hardware
                  ↓
               ISR/DMA
                  ↓
            Slave Driver
                  ↓
          Register / FIFO
                  ↓
             RTOS Task
                  ↓
            Application
```

The ISR should ideally handle the immediate bus events while the application logic remains outside interrupt context.

---

### Interview Answer

> In I2C slave mode, the MCU responds to an external controller that initiates communication. The slave has an assigned address, acknowledges address matches, receives data during controller writes, and provides data during controller reads. A typical slave implementation handles address-match, receive, transmit, STOP, and error events, often using an interrupt-driven state machine.

---

# I2C Intermediate-Level Quick Revision

| Topic            | Key Point                                                 |
| ---------------- | --------------------------------------------------------- |
| Bit-Banging      | I2C implemented using GPIO in software                    |
| GPIO HIGH        | Normally means release line, not actively drive HIGH      |
| Clock Stretching | Target holds SCL LOW                                      |
| Timeout          | Prevents infinite wait                                    |
| Bus Recovery     | Toggle SCL up to 9 times + STOP                           |
| STM32 I2C        | Configure clock, GPIO, speed, addressing, ACK, peripheral |
| Own Address      | Address of MCU when it acts as an I2C target              |
| Interrupts       | Event/error-driven state machine                          |
| Multi-Byte       | Multiple bytes within one START/STOP transaction          |
| Bus Stuck        | SDA/SCL unable to return to expected idle state           |
| Slave Mode       | MCU responds to an external I2C controller                |

---

# Important I2C Interview Traps

### Trap 1 — Bit-banging HIGH

Don't say:

> "For I2C HIGH, drive GPIO HIGH."

Better:

> "Release the open-drain line and let the pull-up bring it HIGH."

---

### Trap 2 — Clock Stretching

Don't say:

> "Master waits for a fixed delay."

Better:

> "Master releases SCL and waits until the actual SCL line becomes HIGH, with a timeout."

---

### Trap 3 — Timeout

Don't write production code like:

```c
while (SCL == 0)
{
}
```

without a timeout.

A failed target can lock the task forever.

---

### Trap 4 — STM32 Own Address

Don't confuse:

```c
I2C_OwnAddress1
```

with:

```text
Address of the external sensor
```

`OwnAddress1` is the MCU's own address when it participates as a target/slave.

---

### Trap 5 — Bus Recovery

Remember:

```text
Stuck bus
   ↓
Release SDA
   ↓
Clock SCL up to 9 times
   ↓
Generate STOP
   ↓
Reinitialize I2C if needed
```

But this works only for recoverable protocol-state problems; it cannot fix a physical short or permanently powered-off hardware.

---

# Senior-Level I2C Architecture

A production-quality I2C driver often looks like:

```text
                    Application
                         ↓
                 I2C API / Service
                         ↓
                  Transaction Layer
                         ↓
               ┌─────────┴─────────┐
               ↓                   ↓
            Polling              IRQ/DMA
                                   ↓
                             State Machine
                                   ↓
                           I2C Peripheral
                                   ↓
                         SDA / SCL + Pull-ups
                                   ↓
                             I2C Devices
```

And recovery should be designed as part of the driver:

```text
Transaction
    ↓
Success ───────────────────────→ Complete
    │
    ↓
Timeout / NACK / Bus Error
    ↓
Abort Transaction
    ↓
Bus Recovery
    ↓
Reinitialize
    ↓
Retry if Policy Allows
    ↓
Return Status
```

The key senior-level idea is:
# I2C – Advanced Level

> **Target:** M.Tech Graduate + 10 Years Experience
> **Level:** Advanced
> **Focus:** I3C, I2C vs I3C, Debugging, Failure Analysis, Pull-Up Calculation

---

### 17. What is I3C (Improved Inter-Integrated Circuit)?

**I3C** is a newer two-wire serial bus developed by MIPI to combine useful features of **I2C and SPI-style higher-performance communication**.

A simple way to remember it is:

```text
I3C
 ↓
I2C compatibility
+
Higher speed
+
Dynamic addressing
+
In-Band Interrupts
+
Better power efficiency
```

It is mainly aimed at systems with many sensors and other peripherals where I2C bandwidth, addressing, and interrupt-pin requirements become limitations.

---

### I3C bus structure

Like I2C, I3C normally uses:

```text
SDA
SCL
```

So:

```text
              I3C BUS

SDA ─────────────────────────────
SCL ─────────────────────────────
      │        │        │
     MCU     Sensor    Sensor
 Controller  Target    Target
```

The major difference is that I3C can use **push-pull signaling for much of its high-speed data transfer**, while retaining open-drain behavior for certain bus-management phases and for compatibility with legacy I2C targets.

---

### I3C speed

Be careful with the commonly quoted numbers.

For standard **I3C SDR mode**, the base raw bit rate is:

```text
12.5 Mbps
```

with about:

```text
11 Mbps
```

of real data rate at a 12.5 MHz clock, according to the current MIPI FAQ.

Higher performance is available through optional HDR modes. For example, MIPI documents up to:

```text
33.3 Mbps raw
≈ 30 Mbps real data
```

for certain HDR modes. I3C v1.1.1 and later also define multi-lane options that can raise aggregate throughput much further.

So don't memorize:

```text
"I3C = 25 MHz"
```

as the standard I3C speed.

A better interview answer is:

> "I3C SDR provides a 12.5 Mbps raw data rate, with optional HDR and later multi-lane modes providing higher throughput."

---

### Why is I3C faster than I2C?

Traditional I2C relies heavily on:

```text
Open-drain signaling
+
Pull-up resistors
```

The pull-up resistor makes the HIGH transition relatively slow because the bus capacitance must charge through the resistor.

I3C uses:

```text
Push-pull signaling
```

for much of its high-speed operation.

Conceptually:

```text
I2C:

LOW  → actively drive
HIGH → resistor pulls up

I3C:

LOW  → actively drive
HIGH → actively drive
```

This produces much faster transitions and allows much higher data rates.

MIPI specifies that SCL on an I3C bus is driven push-pull by the controller, while SDA switches between push-pull and open-drain modes depending on the type of transaction.

---

### Dynamic Address Assignment

Traditional I2C normally depends on devices having fixed/static addresses:

```text
Sensor A → 0x48
Sensor B → 0x68
Sensor C → 0x3C
```

I3C supports **Dynamic Address Assignment (DAA)**.

The controller discovers devices and assigns them dynamic 7-bit addresses during bus initialization.

Conceptually:

```text
I3C Controller
      ↓
Discover Targets
      ↓
Identify each Target
      ↓
Assign Dynamic Addresses
      ↓
Normal Communication
```

MIPI uses a Provisioned ID for this process, and the controller can assign a dynamic 7-bit address to each target.

---

### Does I3C completely eliminate static addresses?

No.

This is an important interview correction.

An I3C target **may also have a static address**. Dynamic addressing is the normal mechanism for managing active I3C targets, but static addressing can still exist for compatibility and other use cases.

---

### What is In-Band Interrupt – IBI?

Traditional I2C generally requires an external GPIO if a sensor wants to notify the MCU:

```text
Sensor
  │
  ├── I2C SDA
  ├── I2C SCL
  └── INT GPIO ──────> MCU
```

With I3C, a target can request attention **through the I3C bus itself**.

This is called:

> **In-Band Interrupt (IBI)**

Conceptually:

```text
Sensor
   │
   │ IBI
   ↓
I3C Controller
```

This can reduce the need for separate interrupt GPIOs, which is particularly useful when a system has many sensors.

---

### Important: I3C does not use I2C-style clock stretching

This is a major difference.

In I2C:

```text
Target can hold SCL LOW
```

In I3C:

```text
Target cannot normally stretch SCL
```

The I3C controller manages SCL, and clock stretching is not allowed on an I3C bus.

This is one of the reasons I3C can provide much more deterministic high-speed timing.

---

### Is I3C backward compatible with I2C?

Yes, but be precise.

I3C can coexist with many legacy I2C **target** devices, but there are restrictions. For example, MIPI specifies compatibility conditions such as support for the required spike filtering and the legacy target not using clock stretching on the I3C bus. Legacy I2C Fast-mode/Fast-mode Plus targets are the main compatibility targets; not every arbitrary I2C device can simply be connected to an I3C bus.

Also:

> A legacy I2C controller cannot simply share an I3C bus as another controller.

---

### Interview Answer

> I3C is a MIPI two-wire bus designed as a higher-performance evolution of I2C. It maintains compatibility with many legacy I2C targets while adding higher-speed push-pull signaling, dynamic address assignment, in-band interrupts, and other bus-management features. Standard I3C SDR provides 12.5 Mbps raw throughput, with optional HDR and newer multi-lane modes providing higher performance. Unlike I2C, normal clock stretching by targets is not allowed on an I3C bus.

---

### 18. I3C vs I2C

The easiest way to compare them is:

| Feature                     | I2C                                           | I3C                                                                    |
| --------------------------- | --------------------------------------------- | ---------------------------------------------------------------------- |
| Bus lines                   | SDA + SCL                                     | SDA + SCL                                                              |
| Normal signaling            | Open-drain with pull-ups                      | Mixed open-drain + push-pull                                           |
| Typical standard speed      | 100 kHz                                       | 12.5 Mbps SDR raw                                                      |
| Higher-speed modes          | Up to 1 MHz for Fm+; other legacy modes exist | HDR modes and newer multi-lane options                                 |
| Addressing                  | Static 7-bit or 10-bit                        | Dynamic 7-bit addressing, with static-address capability also possible |
| In-Band Interrupt           | No                                            | Yes                                                                    |
| Clock stretching            | Supported                                     | Not allowed for normal I3C target operation                            |
| Legacy I2C targets          | Native                                        | Many can coexist with restrictions                                     |
| External pull-up dependence | Important                                     | Mainly needed for open-drain phases / legacy I2C compatibility         |
| Main use                    | Sensors, EEPROM, RTC, PMIC                    | High-speed sensor hubs and dense peripheral systems                    |
| Power efficiency            | Good                                          | Designed for higher performance with efficient bus operation           |

I3C was specifically designed to combine higher performance and lower pin overhead while preserving a migration path from I2C.

---

### When would you choose I3C instead of I2C?

I3C becomes attractive when you have:

```text
Many sensors
+
High sensor data rate
+
Limited GPIOs
+
Need for interrupt signaling
+
Need for dynamic device management
```

For example:

```text
                    MCU
                     │
                    I3C
       ┌─────────────┼─────────────┐
       │             │             │
      IMU          Gyro        Temperature
```

Instead of having:

```text
I2C
+
Many interrupt GPIOs
```

you can use IBI for supported I3C targets.

---

### When is I2C still perfectly reasonable?

I3C is not automatically better for every peripheral.

For:

```text
EEPROM
RTC
Simple sensor
GPIO expander
PMIC
Low-speed peripheral
```

I2C may be completely sufficient.

The decision should be based on:

```text
Bandwidth
Power
Pin count
Device availability
Cost
Legacy compatibility
Software support
```

---

### Interview Answer

> I2C is simpler and widely supported, but it is limited by open-drain signaling, static addressing, and the need for separate interrupt GPIOs in many systems. I3C uses the same two-wire concept but adds higher-speed push-pull transfer, dynamic addressing, in-band interrupts, and improved system management. I would choose I3C when the system has many sensors or needs higher bandwidth and fewer sideband GPIOs; I2C remains a practical choice for simpler low-speed peripherals.

---

### 19. How do you debug I2C issues?

The most important rule is:

> **Don't debug I2C only from firmware logs. Look at the actual SDA and SCL waveforms.**

A systematic process is much faster.

---

### Step 1 – Check the Physical Bus

First verify:

```text
SDA connected correctly
SCL connected correctly
Common GND
Correct supply voltage
Pull-up resistors present
```

At idle:

```text
SDA = HIGH
SCL = HIGH
```

If they are LOW before any transaction starts, investigate the bus immediately.

---

### Step 2 – Check Pull-Ups

Verify:

```text
Pull-up value
Supply voltage
Bus capacitance
Rise time
```

A resistor that is too large produces a slow rising edge:

```text
LOW ______
         /
        /
       /________ HIGH
```

A resistor that is too small causes:

```text
Higher LOW-level current
Higher power dissipation
```

So pull-up selection is a trade-off.

---

### Step 3 – Use a Logic Analyzer

A logic analyzer is usually the first tool I would use.

Check:

```text
START
Address
R/W bit
ACK/NACK
Register
Data
Repeated START
STOP
```

For example:

```text
START
 ↓
0x50 + W
 ↓
ACK?
 ↓
0x10
 ↓
ACK?
 ↓
Repeated START
 ↓
0x50 + R
 ↓
ACK?
 ↓
DATA
 ↓
NACK
 ↓
STOP
```

A logic analyzer can quickly answer:

> "Is the master actually sending what I think the firmware is sending?"

---

### Step 4 – Check the Address

A very common problem is address confusion.

Suppose the device datasheet says:

```text
7-bit address = 0x50
```

but the driver expects the shifted address:

```text
0xA0
```

or vice versa.

You may end up effectively doing:

```text
Wrong address
     ↓
No ACK
```

Always verify what the MCU driver API expects.

---

### Step 5 – Check ACK/NACK

If the slave does not ACK the address:

```text
Address
   ↓
NACK
```

possible reasons include:

```text
Wrong address
Device not powered
Device held in reset
Wrong bus voltage
Wrong pins
Device busy
Device not actually present
```

If the address is ACKed but a later data byte is NACKed:

```text
Address → ACK
Register → NACK
```

then the problem may be in the device-specific protocol rather than basic bus connectivity.

---

### Step 6 – Use an Oscilloscope

A logic analyzer tells you:

```text
"What digital protocol happened?"
```

An oscilloscope tells you:

```text
"What did the electrical signal actually look like?"
```

Use the oscilloscope to inspect:

```text
Rise time
Fall time
Noise
Ringing
Overshoot
Undershoot
Voltage levels
```

This is especially important when:

```text
I2C works at 100 kHz
but fails at 400 kHz
```

That often points toward an electrical/timing problem.

---

### Step 7 – Check Clock Stretching

If communication occasionally hangs:

```text
SCL LOW
```

check whether:

```text
Target is stretching
```

or:

```text
Target is permanently holding SCL LOW
```

The logic analyzer can make this very obvious.

---

### Step 8 – Test With Known-Good Hardware

A very effective debugging technique is:

```text
Known-good Master
+
Known-good Slave
```

Then:

```text
Your Master + Known-good Slave
```

Then:

```text
Known-good Master + Your Slave
```

This isolates whether the problem is:

```text
Master
Slave
Physical Bus
```

---

### Senior-Level I2C Debug Flow

```text
No Communication
      ↓
Are SDA/SCL physically connected?
      ↓
Are both HIGH when idle?
      ↓
Are pull-ups correct?
      ↓
START generated?
      ↓
Correct address?
      ↓
ACK received?
      ↓
Correct register/data?
      ↓
Repeated START correct?
      ↓
STOP generated?
      ↓
Timing / rise time okay?
```

---

### Interview Answer

> I debug I2C from the physical layer upward. First I check SDA/SCL wiring, voltage, common ground, and pull-ups. Then I use a logic analyzer to verify START, address, R/W, ACK/NACK, data, repeated START, and STOP. If the digital sequence looks correct but communication is still unreliable, I use an oscilloscope to check rise time, noise, and signal integrity. For intermittent problems, I also check clock stretching, bus capacitance, and compare against a known-good master or target.

---

### 20. What causes I2C failure?

I2C failures usually fall into four categories:

```text
Configuration
Electrical
Protocol
Software
```

---

#### 1. No Pull-Up Resistors

If SDA/SCL use open-drain signaling and there is no suitable pull-up:

```text
Device releases line
        ↓
Nobody drives HIGH
        ↓
Line remains LOW / undefined
        ↓
Communication fails
```

---

#### 2. Pull-Up Value Too High

Example:

```text
Large Rp
+
Large bus capacitance
        ↓
Slow rising edge
        ↓
Timing violation
        ↓
Communication failure
```

This becomes especially important at:

```text
400 kHz
1 MHz
```

---

#### 3. Pull-Up Value Too Low

A very small resistor makes the line rise quickly, but when a device pulls LOW:

```text
VCC
 │
Rp
 │
Device pulls LOW
```

the sink current becomes large.

That can cause:

```text
Higher power
Excess device current
Possible violation of IOL specification
```

Therefore, both minimum and maximum resistor values matter.

---

#### 4. Bus Capacitance Too High

Capacitance comes from:

```text
PCB traces
Connectors
Device pins
Cables
Multiple devices
Level shifters
```

Higher capacitance means slower rising edges.

So:

```text
Long cable
+
Many devices
=
Higher Cbus
```

and therefore:

```text
Slower rise time
```

---

#### 5. Slave Not ACKing

Possible reasons:

```text
Wrong address
Device not powered
Wrong register protocol
Device in reset
Device busy
Wrong voltage
Wrong pin configuration
```

---

#### 6. Master Does Not Recognize the Slave

This may actually be a software/configuration problem:

```text
Wrong address format
7-bit vs shifted address confusion
Wrong GPIO alternate function
Wrong I2C peripheral instance
Wrong bus speed
```

---

#### 7. Clock Stretching Not Supported Correctly

If the target stretches SCL but the controller driver does not correctly handle it:

```text
Target pulls SCL LOW
        ↓
Controller assumes clock HIGH
        ↓
Timing breaks
```

In a hardware I2C implementation, verify that the MCU peripheral supports the target's clock-stretching behavior.

---

#### 8. Noise

Noise may produce:

```text
False edges
Corrupted bits
Unexpected START/STOP
NACK
Bus lockup
```

Possible sources:

```text
Motor
DC/DC converter
High-current switching
Poor grounding
Long traces
Crosstalk
EMI
```

---

#### 9. Software State-Machine Bug

For an interrupt-driven driver:

```text
START
 ↓
Address
 ↓
ACK
 ↓
Data
```

If the state machine handles one event incorrectly:

```text
Wrong state
   ↓
Wrong register access
   ↓
Invalid bus sequence
```

The waveform may look unusual even though the physical hardware is fine.

---

### 21. How do you calculate the I2C pull-up resistor?

This is a very important senior-level I2C question.

The resistor must satisfy **two constraints**:

```text
Rp must not be too large
Rp must not be too small
```

---

## Maximum Pull-Up Resistance

The maximum resistor is mainly limited by:

```text
Required rise time
+
Total bus capacitance
```

The standard RC relationship is:

```text id="k9p3b7"
tr = 0.8473 × Rp × Cb
```

Therefore:

```text id="0p1r3x"
Rp(max) = tr(max) / (0.8473 × Cb)
```

This is the key formula to remember.

---

### Example

Suppose:

```text
Bus speed      = 100 kHz
Required rise  = 1000 ns
Bus capacitance = 100 pF
```

Then:

```text
Rp(max)
=
1000 ns / (0.8473 × 100 pF)
```

Approximately:

```text
Rp(max) ≈ 11.8 kΩ
```

So a resistor below roughly:

```text
11.8 kΩ
```

would satisfy the rise-time constraint in this simplified calculation.

A practical standard value might then be selected after checking the minimum-resistance constraint and device datasheets.

---

## Minimum Pull-Up Resistance

The resistor also cannot be too small.

When a device pulls SDA/SCL LOW, it must sink the resulting current.

The lower limit is approximately:

```text id="lwx5fd"
Rp(min) =
(VCC - VOL(max)) / IOL(max)
```

where:

```text
VCC      = Bus supply
VOL(max) = Maximum allowed LOW voltage
IOL(max) = Maximum LOW-level sink current
```

This is important because a very small pull-up resistor causes excessive current when the line is LOW.

---

### Therefore the valid resistor range is:

```text id="wqg0l9"
Rp(min) ≤ Rp ≤ Rp(max)
```

Graphically:

```text
Too Low                    Valid                  Too High
│----------------------------│------------------------│

High current          Correct rise time       Slow rise time
More power             Good signal            Timing violation
```

---

### What determines `Cb`?

`Cb` is the **total bus capacitance**.

It includes contributions from:

```text
MCU pin
Sensor pins
EEPROM pins
PCB traces
Connector
Cable
Level shifter
Other devices
```

Conceptually:

```text
             C1
             │
SDA ─────────┼──── C2 ──── C3 ──── C4
             │
            Cbus
```

You need to consider the combined capacitance, not just one device's pin capacitance.

---

### I2C Rise-Time Limits

Common I2C rise-time limits include approximately:

```text
Standard-mode       1000 ns
Fast-mode            300 ns
Fast-mode Plus       120 ns
```

So as the bus gets faster:

```text
Higher speed
     ↓
Smaller allowed rise time
     ↓
Smaller maximum Rp
```

The exact limits should be checked against the relevant I2C specification/device requirements.

---

### Why is "4.7 kΩ" not a universal answer?

You often hear:

```text
"Use 4.7 kΩ."
```

That's a common practical starting point, but it is **not a universal rule**.

For example:

```text
Small board
+
Low capacitance
+
100 kHz
```

might work perfectly with a relatively weak pull-up.

But:

```text
Long bus
+
Many devices
+
400 kHz
```

may require a stronger pull-up.

Conversely, making the resistor too small can create excessive sink current.

Therefore:

> **Calculate the allowable range and then choose a standard resistor value.**

TI and NXP both document the same basic RC relationship and the need to consider both rise time and sink-current limits.

---

### Why does a stronger pull-up make the signal rise faster?

Because:

```text
Smaller R
+
Same C
=
Smaller RC time constant
```

Therefore:

```text
Smaller R
    ↓
Faster rise
```

But:

```text
Smaller R
    ↓
More current when LOW
    ↓
Higher power
```

So there is a trade-off.

---

### Example: Why 10 kΩ can fail at higher speed

Suppose:

```text
Rp = 10 kΩ
Cbus = large
```

Then:

```text
tr = 0.8473 × Rp × Cbus
```

can become large.

At:

```text
100 kHz
```

the system may still work.

At:

```text
400 kHz
```

the same rise time might violate the tighter timing requirement.

This explains a very common real-world failure:

> "I2C works at 100 kHz but fails at 400 kHz."

The pull-up and bus capacitance are among the first things to investigate.

---

## Senior-Level I2C Pull-Up Design Flow

```text
Know Bus Speed
      ↓
Determine Maximum Rise Time
      ↓
Estimate / Measure Cbus
      ↓
Calculate Rp(max)
      ↓
Calculate Rp(min)
      ↓
Choose Standard Resistor
      ↓
Check Device IOL / VOL
      ↓
Measure Actual Rise Time
      ↓
Validate at Temperature / Voltage Corners
```

For a production design, don't stop at the calculation.

Measure the actual bus waveform with an oscilloscope.

---

# Advanced I2C Quick Revision

| Topic                | Key Point                                                          |
| -------------------- | ------------------------------------------------------------------ |
| I3C                  | Higher-performance evolution of I2C                                |
| I3C SDR              | 12.5 Mbps raw                                                      |
| I3C HDR              | Higher data-rate modes available                                   |
| Dynamic Address      | Controller assigns addresses during bus initialization             |
| IBI                  | Target can request attention through the bus                       |
| I3C Signaling        | Push-pull for much of high-speed traffic; open-drain phases remain |
| I3C Clock Stretching | Not allowed in normal I3C operation                                |
| I2C Debug            | Logic analyzer + oscilloscope                                      |
| First Debug Check    | SDA/SCL idle HIGH                                                  |
| ACK Failure          | Check address, power, wiring, configuration                        |
| Bus Capacitance      | Higher C → slower rise time                                        |
| Pull-Up Too High     | Slow rise time                                                     |
| Pull-Up Too Low      | Excess LOW-level current                                           |
| `Rp(max)`            | `tr(max) / (0.8473 × Cb)`                                          |
| `Rp(min)`            | `(VCC - VOL(max)) / IOL(max)`                                      |
| Valid Range          | `Rp(min) ≤ Rp ≤ Rp(max)`                                           |

---

# Important Advanced Interview Traps

### Trap 1 — I3C = 25 MHz

Don't memorize this.

Better:

```text
I3C SDR
≈ 12.5 Mbps raw
≈ 11 Mbps real data
```

with higher-rate HDR and multi-lane options available in newer versions.

---

### Trap 2 — I3C is exactly the same as faster I2C

No.

I3C changes the signaling model and bus management substantially:

```text
Dynamic Addressing
IBI
Push-Pull High-Speed Transfer
No Target Clock Stretching
CCC Commands
```

---

### Trap 3 — Every I2C device works on I3C

No.

Many legacy I2C targets can coexist, but there are compatibility requirements and restrictions.

---

### Trap 4 — Pull-up resistor has one fixed value

No.

The correct resistor depends on:

```text
VCC
VOL
IOL
Bus capacitance
Required rise time
Bus speed
```

---

### Trap 5 — Only calculate `Rp(max)`

A good hardware answer calculates **both**:

```text
Rp(min)
+
Rp(max)
```

because:

```text
Too low → current problem
Too high → rise-time problem
```

---

# Senior-Level Mental Model

For an experienced embedded engineer, think about I2C as:

```text
                 I2C SYSTEM
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
   Protocol       Electrical    Firmware
       │             │             │
 START/STOP       Pull-ups       Driver
 ACK/NACK         Cbus           ISR/DMA
 Address          Rise time      State machine
 Read/Write       Voltage       Timeout
       │             │             │
       └─────────────┼─────────────┘
                     ↓
                 Real Hardware
```

When debugging:

```text
No ACK
  ↓
Check address
  ↓
Check power
  ↓
Check SDA/SCL
  ↓
Check pull-ups
  ↓
Check waveform
  ↓
Check firmware state machine
```

When debugging a high-speed bus:

```text
100 kHz works
400 kHz fails
      ↓
Check:
      ↓
Rise time
Bus capacitance
Pull-up value
Signal integrity
Clock timing
```

And when discussing I3C:

```text
I2C
 ↓
Static addressing
Open-drain
External interrupt GPIO
Slower bus

I3C
 ↓
Dynamic addressing
Push-pull high-speed transfer
IBI
Better system integration
Legacy I2C target coexistence
```

> **The senior-level skill is not memorizing protocol definitions. It is being able to connect the protocol behavior to the actual electrical waveform, peripheral hardware, firmware state machine, and system-level failure mode.**
