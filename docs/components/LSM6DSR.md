# LSM6DSR IMU (Inertial Measurement Unit)

## Overview
STMicroelectronics LSM6DSRTR 6-Axis IMU
- 3-axis accelerometer + 3-axis gyroscope
- I2C/SPI digital interfaces
- High-performance and normal operating modes
- ODR up to 416 Hz (gyro) and 833 Hz (accel)

## Power Requirements

### VDD Rail (Core Supply)
- **Voltage:** 1.8V (nominal), range 1.71V - 3.6V
- **Standby Current (Power-Down Mode):** 3 µA (typical)
- **Record Current (104 Hz High-Performance Mode):** 1.2 mA (typical)
  - Accelerometer only: 360 µA (typical)
  - Gyroscope only: ~840 µA (calculated)
  - Combined: 1.2 mA (typical)

### VDDIO Rail (I/O Supply)
- **Voltage:** 1.62V - 3.6V (can match VDD or be independent)
- **Standby Current:** Included in VDD standby (3 µA total)
- **Record Current:** Minimal, included in VDD consumption

## Operating Modes

### Power-Down Mode
- **VDD Current:** 3 µA (typical)
- **Description:** Both accelerometer and gyroscope powered down
- **Use case:** System standby

### High-Performance Mode @ 104 Hz
- **VDD Current:** 1.2 mA (typical) for both sensors
- **Description:** Maximum accuracy, ODR-independent current in HP mode
- **Configuration:**
  - Accelerometer: 104 Hz ODR, high-performance mode
  - Gyroscope: 104 Hz ODR, high-performance mode
  - Interface: I2C @ 400 kHz
- **Use case:** Active IMU data logging during flight/recording

### Normal Mode @ 208 Hz (Reference)
- **VDD Current:** 0.7 mA (typical) for both sensors
- **Note:** At 104 Hz, normal mode would consume less, but exact value not specified in datasheet

## Current Consumption Breakdown

### By Sensor (High-Performance Mode)
| Component          | Current (Typical) | Voltage | Power   |
|--------------------|-------------------|---------|---------|
| Accelerometer only | 360 µA            | 1.8V    | 0.65 mW |
| Gyroscope only     | ~840 µA           | 1.8V    | 1.51 mW |
| Both sensors       | 1.2 mA            | 1.8V    | 2.16 mW |
| Power-down         | 3 µA              | 1.8V    | 5.4 µW  |

### By Operating Mode
| Mode              | VDD Current | VDD Power |
|-------------------|-------------|-----------|
| Power-Down        | 3 µA        | 5.4 µW    |
| Accel HP @ 104 Hz | 360 µA      | 0.65 mW   |
| Gyro HP @ 104 Hz  | 840 µA      | 1.51 mW   |
| Both HP @ 104 Hz  | 1.2 mA      | 2.16 mW   |

## Total Power Consumption

### Standby Mode
- **Total Current:** 3 µA @ 1.8V
- **Total Power:** ~5 µW

### Record Mode (104 Hz High-Performance)
- **Total Current:** 1.2 mA @ 1.8V
- **Total Power:** ~2.2 mW

## Interface Configuration

### I2C Communication
- **Clock Speed:** Up to 400 kHz (Fast mode), 1 MHz (Fast mode Plus)
- **Address:** 0x6A or 0x6B (configurable via SA0 pin)
- **Current:** Included in VDD consumption, minimal additional overhead

### Pins
- **SCL/SDA:** I2C interface
- **INT1/INT2:** Programmable interrupt outputs
- **SA0:** I2C address selection
- **CS:** Chip select (tie high for I2C mode, tie low for SPI mode)

## Timing and Performance

### Turn-On Time
- **Power-down to active:** 35 ms (typical)
- **Description:** Time required to stabilize after power-up or mode change

### Data Rate
- **ODR:** 104 Hz for both accelerometer and gyroscope
- **Data throughput:** ~208 samples/second (combined)

## Design Considerations

### Low-Noise Power Supply
- **Reason:** IMU requires clean, low-noise power for accurate measurements
- **Recommendation:** Use LDO regulator for best performance

### Power Sequencing
- **VDD and VDDIO:** No specific sequencing required
- **Power-up delay:** Allow 35 ms stabilization before first I2C transaction

### Thermal Considerations
- **Operating temperature:** -40°C to +85°C
- **Test conditions:** Specifications at VDD = 1.8V, T = 25°C
- **Note:** Current consumption may vary with temperature

## Datasheet Notes
- **All current values are TYPICAL only** - Datasheet does not provide maximum specifications
- **Test conditions:** VDD = 1.8V, T = 25°C unless otherwise noted
- **High-performance mode:** ODR-independent current (same at 12.5 Hz or 416 Hz)
- **I2C interface current:** Not separately specified, included in main VDD consumption

## Features
- Embedded temperature sensor
- Programmable interrupts (INT1/INT2)
- FIFO buffer (up to 4 kB)
- Always-on experience with low power consumption
- Selectable full-scale acceleration ranges: ±2/±4/±8/±16 g
- Selectable full-scale angular rates: ±125/±250/±500/±1000/±2000 dps

## Datasheet Reference
`/home/erichandley/UVR/srad-camera/datasheets/html/LSM6DSR-html.html`
