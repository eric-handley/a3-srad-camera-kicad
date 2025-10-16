# Hardware Design Notes

## STM32 Selection

**Selected: STM32U575RGT6**

- 1MB Flash, 192KB RAM, 64-pin LQFP (10×10mm), ~$4
- 6x USART (hw flow control), 4x I2C, CAN FD
- Ultra-low power: 19.5 µA/MHz run, 16 nA standby

## RK3566 Selection

**Selected: RK3566**

- Quad-core Cortex-A55, 15.5×14.4mm BGA, ~$14
- 1080p60 H.264/H.265 encoding, 8MP ISP
- External DDR3L/DDR4/LPDDR4 required

## PMIC Selection

**Selected: STPMIC1APQR**

- 4x Buck (1.5A, 1.5A, 500mA, 2A), 6x LDO (150-300mA)
- I²C programmable, power sequencing
- 5×6mm WFQFN44, ~$3

## IMU Selection

**Selected: LSM6DSRTR**

- 6-axis (accel + gyro), I²C/SPI
- ±2/±4/±8/±16 g accel, ±125/±250/±500/±1000/±2000 dps gyro
- 3×3mm LGA, low power

## RK3566 Voltage Domains

All VCCIO domains (1-7) and PMUIO2 are configurable: **1.8V or 3.3V**

- **PMUIO0**: Fixed 1.8V
- **PMUIO1**: Fixed 3.3V
- **PMUIO2**: Configurable (hosts UART2_M0 debug, I2C0, I2C1)

## Camera I/O Voltage

Most MIPI cameras (IMX219, IMX290) use **1.8V I/O** (IOVDD).

Set PMUIO2 to **1.8V** for camera I2C compatibility.

## STM32 <-> RK3566 UART

**Configuration**: UART with RTS/CTS (4-wire)
- TX, RX, RTS, CTS

**Voltage mismatch**: STM32 (3.3V) <-> RK3566 (1.8V on PMUIO2)

## Level Shifter Selection

**Selected: TXS0104EPWR**

- 4-channel bidirectional, auto-direction detection
- VCCA: 1.65-3.6V, VCCB: 2.3-5.5V
- 24 Mbps data rate, push-pull outputs
- TSSOP-14, -40°C to +85°C

**Solution**: TXS0104EPWR for STM32 (3.3V) <-> RK3566 (1.8V) UART