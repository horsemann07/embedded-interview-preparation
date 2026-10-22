# Embedded Communication Protocols Interview Questions
> UART, I2C, I3C, SPI, CAN, CANopen, Ethernet, Automotive Ethernet, 1-Wire, LIN, Modbus, etc.  
> M.Tech Graduate + 10 Years Experience  
> Basic → Intermediate → Advanced → Expert → War Stories

---

## SPI (Serial Peripheral Interface)

### Basic Level

1. What is SPI?
   - Follow-up: Synchronous serial protocol.
   - Follow-up: 4-wire: SCLK (clock), MOSI (Master Out Slave In), MISO (Master In Slave Out), CS (Chip Select).
   - Follow-up: Master-only protocol (one master, one or more slaves).
   - Follow-up: Full-duplex communication.
   - Follow-up: Speeds: up to 50MHz+ on embedded.
   - Follow-up: Common use: memory (Flash, EEPROM), sensors, display controllers.

2. SPI hardware connection?
   ```
   Master                  Slave1              Slave2
   SCLK  ------+-------+----SCLK             (parallel)
               |       |
   MOSI  ------+-------+----MOSI
               |       |
   MISO  ------+-------+----MISO
               |       |
   CS0   ------+-------+----CS
               
   CS1   ------+-------+----CS
   ```
   - Follow-up: Each slave has dedicated CS line (one master can control multiple slaves).
   - Follow-up: MOSI/MISO shared across all slaves.
   - Follow-up: Only slave with CS = 0 responds.

3. SPI modes (CPHA, CPOL)?
   - Follow-up: CPOL = 0: idle SCL = low, CPOL = 1: idle SCL = high.
   - Follow-up: CPHA = 0: data captured on leading edge, CPHA = 1: data captured on trailing edge.
   - Follow-up: Modes: SPI_MODE_0 (CPOL=0, CPHA=0), ..., SPI_MODE_3 (CPOL=1, CPHA=1).

   | Mode | CPOL | CPHA | Leading Edge | Trailing Edge |
   |------|------|------|--------------|---------------|
   | 0 | 0 | 0 | Capture | Setup |
   | 1 | 0 | 1 | Setup | Capture |
   | 2 | 1 | 0 | Capture | Setup |
   | 3 | 1 | 1 | Setup | Capture |

4. How does SPI transfer work?
   - Follow-up: Shift register — bits shifted in on MOSI, out on MISO, synchronized to SCLK.
   - Follow-up: Full-duplex: master sends byte while slave sends byte back.
   - Follow-up: CS must be held low during transfer, released after.

5. How to transmit/receive over SPI (bit-bang)?
   ```c
   void spi_cs_low(void) {
       GPIO_ResetBits(CS_PORT, CS_PIN);
   }

   void spi_cs_high(void) {
       GPIO_SetBits(CS_PORT, CS_PIN);
   }

   uint8_t spi_transfer(uint8_t byte_out) {
       uint8_t byte_in = 0;
       for (int i = 7; i >= 0; i--) {
           // Setup MOSI
           if (byte_out & (1 << i)) {
               GPIO_SetBits(MOSI_PORT, MOSI_PIN);
           } else {
               GPIO_ResetBits(MOSI_PORT, MOSI_PIN);
           }
           
           // Clock pulse
           GPIO_SetBits(SCLK_PORT, SCLK_PIN);
           delay_ns(10);
           
           // Read MISO
           if (GPIO_ReadInputDataBit(MISO_PORT, MISO_PIN)) {
               byte_in |= (1 << i);
           }
           
           GPIO_ResetBits(SCLK_PORT, SCLK_PIN);
           delay_ns(10);
       }
       return byte_in;
   }

   // Usage
   spi_cs_low();
   uint8_t response = spi_transfer(0xAA);
   spi_cs_high();
   ```

6. How to use SPI peripheral on microcontroller?
   ```c
   SPI_InitTypeDef SPI_InitStructure;
   SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;
   SPI_InitStructure.SPI_Mode = SPI_Mode_Master;
   SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;
   SPI_InitStructure.SPI_CPOL = SPI_CPOL_Low;
   SPI_InitStructure.SPI_CPHA = SPI_CPHA_1Edge;
   SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;  // software CS
   SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_16;
   SPI_Init(SPI1, &SPI_InitStructure);
   SPI_Cmd(SPI1, ENABLE);
   ```

7. How to transfer over SPI peripheral?
   ```c
   // Transmit
   while (SPI_GetFlagStatus(SPI1, SPI_FLAG_TXE) == RESET);
   SPI_SendData(SPI1, 0xAA);

   // Receive
   while (SPI_GetFlagStatus(SPI1, SPI_FLAG_RXNE) == RESET);
   uint8_t data = SPI_ReceiveData(SPI1);

   // Full-duplex transfer
   while (SPI_GetFlagStatus(SPI1, SPI_FLAG_TXE) == RESET);
   SPI_SendData(SPI1, 0xAA);
   while (SPI_GetFlagStatus(SPI1, SPI_FLAG_RXNE) == RESET);
   uint8_t response = SPI_ReceiveData(SPI1);
   ```

---

### Intermediate Level

8. How to use SPI with DMA?
   - Follow-up: DMA copies data to TX FIFO, copies RX FIFO to memory.
   - Follow-up: Interrupt when transfer complete.
   - Follow-up: Much faster than polled or interrupt-driven approach.

9. SPI transaction protocol (multiple bytes)?
   ```c
   // Read register 0x10 from SPI device
   uint8_t cmd = 0x03;        // read command
   uint8_t addr = 0x10;       // register address
   uint8_t data_out[2] = {0}; // don't care during read

   spi_cs_low();
   spi_transfer(cmd);   // send command
   spi_transfer(addr);  // send address
   data_out[0] = spi_transfer(0xFF);  // receive high byte
   data_out[1] = spi_transfer(0xFF);  // receive low byte
   spi_cs_high();
   ```

10. What is quad SPI (QSPI)?
    - Follow-up: 4-line data transfer instead of 1 (MOSI + MISO).
    - Follow-up: 4x faster than standard SPI (if all 4 lines used).
    - Follow-up: Typically used for Flash memory.
    - Follow-up: Backward compatible: can start in SPI mode, switch to QSPI.

11. What is dual SPI (DSPI)?
    - Follow-up: 2-line data transfer.
    - Follow-up: 2x faster than standard SPI.

12. SPI slave mode?
    - Follow-up: Microcontroller as SPI slave.
    - Follow-up: Must respond to CS and SCLK from master.
    - Follow-up: Interrupt when CS = 0, data received, or data requested.

13. How to debug SPI?
    - Follow-up: Logic analyzer to verify CS, SCLK, MOSI, MISO waveforms.
    - Follow-up: Oscilloscope for signal integrity, noise, timing.
    - Follow-up: Check clock polarity and phase match device datasheet.

14. What causes SPI failure?
    - Wrong clock mode (CPOL/CPHA)
    - Clock speed too high
    - CS toggling during transfer
    - Data setup/hold time violation
    - Improper termination (floating lines)
    - Capacitive coupling, noise

---

### Advanced Level

15. How to handle variable-length SPI transactions?
    - Follow-up: Some devices require different TX/RX lengths.
    - Follow-up: Send dummy bytes if need to clock out data without sending.
    - Follow-up: Use DMA with different lengths for TX and RX.

16. What is SPI bus contention?
    - Follow-up: Multiple masters driving same line.
    - Follow-up: SPI is not multi-master safe (unlike I2C).
    - Follow-up: Careful CS management required.

17. How to implement multi-slave SPI?
    - Follow-up: Each slave has dedicated CS line.
    - Follow-up: Pull CS low for target slave, high for others.
    - Follow-up: Daisy-chain: slaves connected in series, data shifts through.
    ```
    Master MOSI -----> Slave1 MOSI (output shift register)
                            |
                       Slave2 MOSI (input to shift register)
                            |
                       Slave3 MOSI ...
    ```

18. SPI timing constraints?
    - Follow-up: Setup time: data valid before clock edge.
    - Follow-up: Hold time: data valid after clock edge.
    - Follow-up: Clock to output delay: how long after clock before slave drives MISO.
    - Follow-up: All constraints in device datasheet.

---
