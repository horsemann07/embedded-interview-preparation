# Embedded Communication Protocols Interview Questions
> UART, I2C, I3C, SPI, CAN, CANopen, Ethernet, Automotive Ethernet, 1-Wire, LIN, Modbus, etc.  
> M.Tech Graduate + 10 Years Experience  
> Basic → Intermediate → Advanced → Expert → War Stories


## I2C (Inter-Integrated Circuit)

### Basic Level

1. What is I2C?
   - Follow-up: 2-wire serial bus: SDA (data), SCL (clock).
   - Follow-up: Multi-master, multi-slave capability.
   - Follow-up: Open-drain outputs (pull-up resistors required).
   - Follow-up: Speeds: 100kHz (standard), 400kHz (fast), 1MHz (fast-plus).
   - Follow-up: Common use: sensors, EEPROMs, RTC, power management ICs.

2. I2C hardware (electrical)?
   ```
   Master                 Slave
   SDA ----+----[Rp]----+---- SDA
           |            |
   SCL ----+----[Rp]----+---- SCL
           |            |
           +---- GND ----+
   ```
   - Follow-up: Pull-up resistors: typically 4.7kΩ on 5V, 10kΩ on 3.3V.
   - Follow-up: Why open-drain? (allows multi-master, wire-OR logic)
   - Follow-up: What is clock stretching? (slave pulls SCL low to pause)

3. I2C addressing?
   - Follow-up: 7-bit address (128 possible, 112 usable) or 10-bit address.
   - Follow-up: Address format: `<7-bit address> <R/W bit>`.
   - Follow-up: R/W bit: 0 = write, 1 = read.
   - Follow-up: Special addresses: 0x00 (general call), 0x78–0x7F (reserved).

4. I2C protocol frame?
   ```
   [START] [Address + R/W] [ACK] [Data Byte] [ACK] ... [STOP]
   ```
   - START: SDA goes low while SCL high.
   - Address 7-bit: MSB first.
   - ACK: slave pulls SDA low during 9th clock.
   - NACK: slave releases SDA (master can detect missing slave).
   - STOP: SDA goes high while SCL high.

5. How to transmit over I2C (write)?
   ```c
   // Write 1 byte to address 0x50
   uint8_t addr = 0x50 << 1;  // 7-bit addr, shift for I2C format
   uint8_t data = 0xAA;
   
   i2c_start();
   i2c_send_byte(addr | 0);  // address + write bit
   if (!i2c_receive_ack()) goto error;
   i2c_send_byte(data);
   if (!i2c_receive_ack()) goto error;
   i2c_stop();
   ```

6. How to receive over I2C (read)?
   ```c
   // Read 1 byte from address 0x50
   uint8_t addr = 0x50 << 1;
   uint8_t data;
   
   i2c_start();
   i2c_send_byte(addr | 1);  // address + read bit
   if (!i2c_receive_ack()) goto error;
   data = i2c_receive_byte();
   i2c_send_nack();  // no more data
   i2c_stop();
   ```

7. I2C repeated START condition?
   - Follow-up: Master can restart without releasing bus (no STOP).
   - Follow-up: Allows write-then-read without releasing bus.
   - Follow-up: Used in: read from address register, then read data.

8. How to read register from I2C device?
   ```c
   // Read register 0x10 from device 0x50
   uint8_t data;
   
   i2c_start();
   i2c_send_byte(0x50 << 1 | 0);  // address + write
   i2c_receive_ack();
   i2c_send_byte(0x10);            // register address
   i2c_receive_ack();
   
   // Repeated START
   i2c_start();
   i2c_send_byte(0x50 << 1 | 1);  // address + read
   i2c_receive_ack();
   data = i2c_receive_byte();
   i2c_send_nack();
   i2c_stop();
   ```

---

### Intermediate Level

9. How to implement I2C bit-banging (software I2C)?
   ```c
   #define SCL_PORT  GPIOB
   #define SCL_PIN   GPIO_PIN_10
   #define SDA_PORT  GPIOB
   #define SDA_PIN   GPIO_PIN_11

   void i2c_scl_low(void) {
       GPIO_ResetBits(SCL_PORT, SCL_PIN);  // drive low
   }

   void i2c_scl_high(void) {
       GPIO_SetBits(SCL_PORT, SCL_PIN);   // release (pull-up)
   }

   int i2c_scl_read(void) {
       return GPIO_ReadInputDataBit(SCL_PORT, SCL_PIN);
   }

   // similar for SDA
   ```
   - Follow-up: Must handle clock stretching (wait if slave holds SCL low).
   - Follow-up: Timing delays between SCL/SDA changes for clock stretching.

10. What is I2C clock stretching?
    - Follow-up: Slave pulls SCL low to pause master.
    - Follow-up: Master detects SCL = 0, waits until slave releases.
    - Follow-up: Slave uses this to slow down communication if needed.
    - Follow-up: How to implement: after releasing SCL, wait for it to go high.

11. What is I2C timeout?
    - Follow-up: If slave never releases SCL, bus deadlock.
    - Follow-up: Implementation: count clock cycles, reset if timeout.
    - Follow-up: Bus recovery: master toggles clock, then STOP.

12. I2C on microcontroller (STM32 example)?
    ```c
    I2C_InitTypeDef I2C_InitStructure;
    I2C_InitStructure.I2C_ClockSpeed = 100000;
    I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
    I2C_InitStructure.I2C_OwnAddress1 = 0x50;
    I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;
    I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
    I2C_Init(I2C1, &I2C_InitStructure);
    I2C_Cmd(I2C1, ENABLE);
    ```

13. How to use I2C with interrupts?
    - Follow-up: Events: `I2C_IT_EVT` (events), `I2C_IT_BUF` (buffer), `I2C_IT_ERR` (errors).
    - Follow-up: Interrupt handler decodes event type and responds.

14. I2C multi-byte transaction?
    ```c
    // Write 3 bytes to device
    i2c_start();
    i2c_send_byte(0x50 << 1);
    i2c_receive_ack();
    i2c_send_byte(0xBB);
    i2c_receive_ack();
    i2c_send_byte(0xCC);
    i2c_receive_ack();
    i2c_send_byte(0xDD);
    i2c_receive_ack();
    i2c_stop();
    ```

15. How to detect I2C bus stuck?
    - Follow-up: SDA = low, SCL = high (slave holding SDA).
    - Follow-up: Recovery: master toggles SCL up to 9 times (clocking out stuck byte), then STOP.

16. What is I2C slave mode?
    - Follow-up: Microcontroller as I2C slave (device).
    - Follow-up: Interrupts on: address match, RX data, TX request.
    - Follow-up: Slave must ACK received bytes, provide data when read.

---

### Advanced Level

17. What is I3C (Improved Inter-Integrated Circuit)?
    - Follow-up: I2C successor, higher speed, backward compatible.
    - Follow-up: Speeds: 12.5MHz (typical), up to 25MHz (5MHz I2C equivalent standard).
    - Follow-up: Open-drain push-pull (OD-PP) for higher speed.
    - Follow-up: Dynamic address assignment replaces static 7-bit address.
    - Follow-up: In-Band Interrupts (IBI) — slave can interrupt master.

18. I3C vs I2C?
    | Feature | I2C | I3C |
    |---------|-----|-----|
    | Speed | 100–1000 kHz | 2.7–25 MHz |
    | Addresses | Static 7/10-bit | Dynamic |
    | Interrupts | No | Yes (IBI) |
    | Backward compat | N/A | Yes (I2C legacy devices) |
    | Power | Low | Higher |

    - Follow-up: When to use I3C? (sensor hubs, high-speed data collection)

19. How to debug I2C issues?
    - Follow-up: Logic analyzer to verify START, STOP, ACK/NACK, data waveforms.
    - Follow-up: Oscilloscope to check signal levels, noise, rise time.
    - Follow-up: Check pull-up resistor values.
    - Follow-up: Verify clock stretching behavior.
    - Follow-up: Test with known-good master and slave.

20. What causes I2C failure?
    - Incorrect pull-up resistor values
    - No pull-up resistors
    - Bus capacitance too high (cable too long, many devices)
    - Slave not ACKing (wrong address, slave not responding)
    - Master not recognizing slave address
    - Clock stretching not implemented
    - Noise on lines

21. How to calculate I2C pull-up resistor?
    - Follow-up: Constraint: rise time < 1000ns (for 100kHz), < 300ns (for 400kHz).
    - Follow-up: Formula: `Rp < 0.8473 * (1000ns - tr) / C_total`
    - Follow-up: Typical: 4.7kΩ (5V), 10kΩ (3.3V).
    - Follow-up: Too low: more power, faster rise. Too high: slower rise (timing violation).

---
