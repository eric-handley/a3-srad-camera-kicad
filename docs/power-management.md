# Power Management Architecture

## Overview

The SRAD Camera uses the **STPMIC1APQR** power management IC (PMIC) to generate all required voltage rails from a 5V main supply. The PMIC provides:
- 4x Buck converters (high-efficiency switching regulators)
- 6x LDO regulators (low-noise linear regulators)
- Integrated DDR reference voltage and termination
- Sequenced power-up/down control

## Design Strategy

**Cascaded Buck + LDO Architecture:**
- Bucks provide high-efficiency voltage conversion for high-current loads
- LDOs cascade from bucks to provide low-noise power for sensitive analog/mixed-signal components
- Minimizes switching noise on IMU and camera sensor analog rails
- Balances efficiency with signal integrity

---

## Buck Converter Assignments

| Buck      | Vout  | Max Current | Load                       |
|-----------|-------|-------------|----------------------------|
| **BUCK1** | 1.2V  | 1500mA      | RK356x Core                |
| **BUCK1** | 1.2V  | 1500mA      | IMX290 Sensor Digital Core |
| **BUCK2** | 1.35V | 1000mA      | DDR3L RAM (VDD/VDDQ)       |
| **BUCK2** | 1.35V | 1000mA      | REFDDR Input               |
| **BUCK3** | 1.92V | 500mA       | LDO1 Input                 |
| **BUCK3** | 1.92V | 500mA       | LDO2 Input                 |
| **BUCK3** | 1.92V | 500mA       | LDO3 Input                 |
| **BUCK4** | 3.4V  | 2000mA      | STM32U575                  |
| **BUCK4** | 3.4V  | 2000mA      | LDO5 Input                 |
| **BUCK4** | 3.4V  | 2000mA      | Level Shifters (3.3V side) |

**Note:** LDO4 (USB PHY) has auto-switching input and can source from VIN, VBUSOTG, or BSTOUT. It does not load the buck converters directly.

## LDO Regulator Assignments

| LDO      | Vin (Source)  | Vout | Dropout | Load                               | Purpose                      |
|----------|---------------|------|---------|------------------------------------|------------------------------|
| **LDO1** | 1.92V (BUCK3) | 1.8V | ~120mV  | LSM6DSRTR IMU                      | Low-noise "quiet" power      |
| **LDO2** | 1.92V (BUCK3) | 1.8V | ~120mV  | RK356x I/O + Level Shifters (1.8V) | Noise isolation from sensor  |
| **LDO3** | 1.92V (BUCK3) | 1.8V | ~120mV  | IMX290 Sensor I/O (OVDD)           | Low-noise I/O power          |
| **LDO4** | VIN/Auto      | 3.3V | ~45mV   | RK356x USB PHY                     | USB analog transceiver power |
| **LDO5** | 3.4V (BUCK4)  | 2.9V | ~180mV  | IMX290 Sensor Analog (AVDD)        | Low-noise analog power       |
| **LDO6** | 1.92V (BUCK3) | 1.8V | ~120mV  | MicroSD Card (VDD + I/O)           | Dedicated SD card power      |

### Why LDOs When Vin ≈ Vout?

Even when input and output voltages are similar (e.g., 1.92V → 1.8V), LDOs provide critical benefits:

1. **Noise Filtering:**
   - Buck converters have ~50-100mV switching ripple at 2MHz
   - LDOs provide 43-45dB PSRR (Power Supply Rejection Ratio)
   - Essential for IMU (gyro/accel) and camera sensor analog circuits

2. **Isolation:**
   - Separates noisy digital switching loads from sensitive analog circuits
   - Prevents crosstalk between sensor I/O and RK356x I/O

3. **Dropout Voltage:**
   - LDO1/LDO2/LDO3: 120mV dropout → 6% efficiency loss for 94% noise-free power
   - Trade-off: Clean power for sensitive sensors vs. small efficiency loss

### LDO4 for USB PHY

**What is USB PHY?**
- USB PHY (Physical Layer) = analog transceiver circuitry that drives USB D+/D- signals
- Separate from the digital USB controller inside the SoC
- Requires dedicated, clean 3.3V analog power supply
- Typically draws 20-30mA during USB operation

**Why use LDO4?**
- LDO4 is specifically designed for USB PHY applications
- Fixed 3.3V output (cannot be adjusted)
- Auto-switching input (VIN/VBUSOTG/BSTOUT) for flexible power sourcing
- Low dropout (~45mV @ 30mA) provides clean power with minimal loss
- Isolated from other 3.3V rails to prevent USB noise coupling

**RK356x USB PHY Power:**
- The RK356x has integrated USB 2.0/3.0 PHYs requiring dedicated analog supply
- Connect to USB_AVDD33 or similar pin (check RK356x pinout)
- Essential for proper USB signal integrity and compliance

### MicroSD Card Power

**Power Requirements:**
- Modern UHS-I microSD cards support 1.8V signaling for high-speed operation
- VDD (card power): 1.8V nominal (2.7-3.6V range, but 1.8V preferred for UHS-I)
- VDDIO (I/O signaling): 1.8V (matches RK356x I/O voltage)
- Active write current: 100-250mA (depends on card speed/brand)
- Standby current: ~0.2mA

**Why use LDO6 (dedicated rail)?**
- RK356x SDMMC interface operates at 1.8V, so card must match
- Single 1.8V rail for both card VDD and I/O signaling (simplest design)
- UHS-I compatible (required for high-speed 50-104 MB/s operation)
- Dedicated rail prevents SD card transients from affecting RK356x I/O (LDO2)
- STPMIC1 LDO6 max current: 300mA (sufficient for 250mA SD card + margin)
- Sourced from BUCK3 (1.92V) with 120mV dropout to 1.8V

**Why not share LDO2 with RK I/O?**
- LDO2 load: RK I/O (200mA) + level shifters (20mA) = 220mA
- Adding SD card (250mA) would total 470mA, exceeding LDO2's 300mA capacity
- SD card write current spikes could cause voltage droop on RK I/O rail
- Separate rail provides better isolation and reliability

---

## Special Functions

### REFDDR - DDR Reference Voltage

| Function   | Input (Source) | Output | Load       | Description                                  |
|------------|----------------|--------|------------|----------------------------------------------|
| **REFDDR** | 1.35V (BUCK2)  | 0.675V | DDR3L VREF | Generates VDDQ/2 reference voltage for DDR3L |

**How it works:**
- REFDDR divides BUCK2 output (1.35V) by 2
- Provides 0.675V reference to DDR3L memory
- Required for proper DDR3L operation per JEDEC spec

### DDRVTT - DDR Termination (Not Used)

**DDRVTT is NOT used in this design** because the RK356x has built-in **On-Die Termination (ODT)**.

**What DDRVTT would do (if used):**
- Uses LDO3 in special "sink/source" mode
- Provides active VTT termination at 0.675V (VDDQ/2)
- Can source or sink current to maintain VTT = VREF

**Why we don't need it:**
- RK356x has programmable ODT with dynamic PVT compensation
- On-die termination eliminates need for external VTT regulator
- LDO3 is freed up for sensor I/O (1.8V) instead

**If you needed DDRVTT:**
- Configure LDO3 in VTT mode (takes 1.35V from BUCK2, outputs 0.675V)
- Use LDO6 for sensor I/O instead of LDO3
- Current capability: ~100mA sink/source for DDR signal termination

---

## IMX290 Sensor Power Requirements

The IMX290LQR-C camera sensor requires **three** separate voltage rails:

### Voltage Rails:

| Rail             | Function            | Voltage (Nominal) | Voltage Range | PMIC Source                |
|------------------|---------------------|-------------------|---------------|----------------------------|
| **AVDD (VDDHx)** | Analog Power        | 2.9V              | 2.80-3.00V    | LDO5                       |
| **OVDD (VDDMx)** | Interface I/O Power | 1.8V              | 1.70-1.90V    | LDO3                       |
| **DVDD (VDDLx)** | Digital Core Power  | 1.2V              | 1.10-1.30V    | BUCK1 (shared with RK356x) |

### Current Consumption:

**Active Mode (LVDS 8ch, 12-bit, 60fps, 1080p):**

| Rail             | Standard Luminosity | Saturated Luminosity | Maximum    | Notes                   |
|------------------|---------------------|----------------------|------------|-------------------------|
| **AVDD (2.9V)**  | 54 mA               | 111 mA               | 111 mA     | Higher in bright scenes |
| **OVDD (1.8V)**  | 16 mA               | 29 mA                | 29 mA      | I/O interface current   |
| **DVDD (1.2V)**  | 77 mA               | 123 mA               | 214 mA     | Pixel processing        |
| **Total Active** | **147 mA**          | **263 mA**           | **354 mA** | **Total sensor power**  |

**Standby Mode:**

| Rail              | Typical     | Maximum | Notes                   |
|-------------------|-------------|---------|-------------------------|
| **AVDD (2.9V)**   | 0.1 mA      | —       | Nearly off              |
| **OVDD (1.8V)**   | 0.1 mA      | —       | Nearly off              |
| **DVDD (1.2V)**   | 14.0 mA     | 14.0 mA | Serial interface active |
| **Total Standby** | **14.2 mA** | —       | **95% power reduction** |

**Standard vs Saturated Luminosity:**
- **Standard**: 1/3 of sensor saturation (typical indoor/outdoor scenes)
- **Saturated**: Full sensor saturation (very bright scenes - sky, sunlight, etc.)
- **Design guideline**: Use saturated values for worst-case power budget

### Power Budget (Saturated):

| Rail             | Current | Voltage | Power Dissipation |
|------------------|---------|---------|-------------------|
| AVDD (LDO5)      | 111 mA  | 2.9V    | **322 mW**        |
| OVDD (LDO3)      | 29 mA   | 1.8V    | **52 mW**         |
| DVDD (BUCK1)     | 123 mA  | 1.2V    | **148 mW**        |
| **Total Sensor** | —       | —       | **522 mW**        |

---

## Power Budget Summary

### Operating Modes:

The system has distinct power modes with different consumption profiles:

#### Suspend Mode (RK3566 Deep Sleep):
- **RK356x Core (1.2V)**: 80mA (retention mode, PMU active, clocks gated)
- **RK356x I/O (1.8V)**: 20mA (GPIO retention, minimal I/O)
- **DDR3L RAM (1.35V)**: 20mA (self-refresh)
- **MicroSD Card (1.8V)**: 0.2mA (standby)
- **Total Suspend Power**: ~170mW @ 5V input (~35mA)

#### Active Recording Mode (1080p60 H.264/H.265 Encoding):
- **RK356x Core (1.2V)**: 1000mA (CPU + VPU encoder active)
- **RK356x I/O (1.8V)**: 200mA (MIPI CSI, UART, peripherals active)
- **DDR3L RAM (1.35V)**: 300mA (active read/write for video buffering)
- **MicroSD Card (1.8V)**: 250mA (active write - video storage)
- **Camera Sensor**: Active (saturated luminosity, all rails)
- **Total Active Power**: ~3.9W @ 5V input (~780mA)

---

### Total System Power (Active Recording, Saturated):

| Subsystem      | Voltage   | Current | Power     | Source            |
|----------------|-----------|---------|-----------|-------------------|
| RK356x Core    | 1.2V      | ~1000mA | 1200mW    | BUCK1             |
| RK356x I/O     | 1.8V      | ~200mA  | 360mW     | LDO2 ← BUCK3      |
| RK356x USB PHY | 3.3V      | ~30mA   | 99mW      | LDO4 (Auto)       |
| DDR3L RAM      | 1.35V     | ~300mA  | 405mW     | BUCK2             |
| MicroSD Card   | 1.8V      | ~250mA  | 450mW     | LDO2 ← BUCK3      |
| STM32U575      | 3.4V      | ~100mA  | 340mW     | BUCK4             |
| IMU (LSM6DSR)  | 1.8V      | ~5mA    | 9mW       | LDO1 ← BUCK3      |
| Sensor Analog  | 2.9V      | 111mA   | 322mW     | LDO5 ← BUCK4      |
| Sensor I/O     | 1.8V      | 29mA    | 52mW      | LDO3 ← BUCK3      |
| Sensor Digital | 1.2V      | 123mA   | 148mW     | BUCK1             |
| Level Shifters | 1.8V/3.3V | ~20mA   | 60mW      | LDO2/BUCK4        |
| **Total**      | —         | —       | **~3.4W** | **From 5V input** |

### Input Power (from 5V supply):

Assuming average buck efficiency of 85-90%:
- **Total Output Power**: ~3.4W
- **Input Power @ 85% eff**: ~4.0W
- **Input Current @ 5V**: ~800mA

**Worst-case with all rails at max + margins: ~5W / 1A @ 5V**

---

## Design Rationale

### Why BUCK3 @ 1.92V Instead of 1.8V?

**Problem**: LDOs need headroom (dropout voltage) to regulate properly
- LDO1 needs: 1.8V + 0.12V = **1.92V minimum**
- LDO2 needs: 1.8V + 0.12V = **1.92V minimum**
- LDO3 needs: 1.8V + 0.12V = **1.92V minimum**

**Solution**: Set BUCK3 to 1.92V and use LDOs for all 1.8V loads
- **Benefit**: All 1.8V rails get clean, isolated power
- **Trade-off**: ~6% efficiency loss per LDO vs direct buck connection
- **Result**: Better signal integrity for IMU, camera sensor, and RK I/O with reduced crosstalk

### Why BUCK4 @ 3.4V Instead of 3.3V?

**Reason**: LDO5 dropout headroom (sensor analog power)
- LDO5 needs: 2.9V + 0.18V = **3.08V minimum** → Set to 3.4V for margin
- STM32U575 spec: 1.71V - 3.6V → **3.4V is within spec**
- Level shifters typically: 3.0V - 3.6V → **3.4V is within spec**

**Benefit**: Clean 2.9V analog power for camera sensor with good efficiency (88%)

### Why Share BUCK1 for RK356x Core + Sensor Digital?

**Reasoning**:
- Both loads require 1.2V nominal
- BUCK1 capacity: 1500mA
- Combined load: ~1125mA max (75% capacity)
- Sensor digital is **not** noise-sensitive (pure digital logic)
- No benefit to separate rails or LDO filtering

**Benefit**: Efficient use of PMIC resources, no additional components needed

---

## Buck Voltage Selection Summary

| Buck      | Nominal Voltage | Actual Setting | Reason for Offset                    |
|-----------|-----------------|----------------|--------------------------------------|
| **BUCK1** | 1.2V            | **1.2V**       | Direct to loads (no LDOs)            |
| **BUCK2** | 1.35V           | **1.35V**      | Direct to DDR + REFDDR               |
| **BUCK3** | 1.8V            | **1.92V**      | +120mV for LDO1/LDO2/LDO3 dropout    |
| **BUCK4** | 3.3V            | **3.4V**       | +180mV for LDO5 dropout + STM margin |

---

## References

- **STPMIC1 Datasheet**: DS12792 Rev 10, STMicroelectronics
- **IMX290LQR-C Datasheet**: Sony Semiconductor Solutions
- **DDR3L JESD79-3F**: JEDEC Standard for DDR3L SDRAM
