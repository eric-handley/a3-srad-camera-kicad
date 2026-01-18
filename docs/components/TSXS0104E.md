# TSXS0104E Level Translator

## Overview
Texas Instruments TSXS0104E
- 4-bit bidirectional voltage-level translator
- Auto-direction sensing (no direction control pin)
- Push-pull and open-drain modes
- Dual supply operation

## Power Requirements

### VCCA Rail (A-Port Supply)
- **Voltage Range:** 1.65V - 3.6V
- **Standby Current:** <2.4 µA (typ) @ 3.6V
- **Active Current:** Part of total ICCA + ICCB (see below)
- **Function:** Powers A-side I/O buffers

### VCCB Rail (B-Port Supply)
- **Voltage Range:** 2.3V - 5.5V
- **Standby Current:** <12 µA @ 2.3V - 5.5V
- **Active Current:** Part of total ICCA + ICCB (see below)
- **Function:** Powers B-side I/O buffers
- **Constraint:** VCCA ≤ VCCB (required)

### Total Supply Current
- **ICCA + ICCB Combined:** 14.4 µA (max) @ VI = VO = Open
- **Active Mode (I/O switching):** Additional dynamic current depends on frequency and load

## Electrical Characteristics

### Input/Output Current
- **Input Current (II):** -1 µA (typ), ±2 µA (max)
- **Output Leakage (IOZ):** -1 µA (typ), ±2 µA (max) when OE = VIL (disabled)
- **Continuous Output Current:** -50 mA to +50 mA

### Voltage Levels
- **VIH (High-level input):** VCCI - 0.2V to VCCI - 0.4V (varies by port and voltage)
- **VIL (Low-level input):** 0V to 0.15V
- **VOH (High-level output):** ≥ VCCA × 0.8 or VCCB × 0.8
- **VOL (Low-level output):** ≤ 0.4V

### Capacitance
- **OE Pin:** 2.5 pF (typ), 3.5 pF (max)
- **A Port I/Os:** 5 pF (typ), 6.5 pF (max)
- **B Port I/Os:** 12 pF (typ), 16.5 pF (max)

## Performance Specifications

### Data Rate
- **Push-Pull Mode:** 24 Mbps (maximum)
- **Open-Drain Mode:** 2 Mbps (maximum)

### Timing
- **Minimum Pulse Duration:**
  - Push-pull: 41 ns
  - Open-drain: 500 ns

## Absolute Maximum Ratings
- **Input Clamp Current (IIK):** -50 mA (when VI < 0)
- **Output Clamp Current (IOK):** -50 mA (when VO < 0)
- **Continuous Output Current:** -50 mA to +50 mA
- **VCC/GND Current:** -100 mA to +100 mA

## Operating Conditions
- **Temperature Range:** -40°C to +85°C
- **VCCA:** 1.65V - 3.6V
- **VCCB:** 2.3V - 5.5V (must be ≥ VCCA)

## Features
- **Channels:** 4-bit bidirectional
- **Auto-Direction Sensing:** No direction control required
- **Output Enable:** Active-high OE pin
- **Modes:** Push-pull or open-drain operation
- **Low Power:** <15 µA quiescent current (both supplies combined)

## Pin Configuration
- **A1-A4:** A-side I/O pins (VCCA voltage level)
- **B1-B4:** B-side I/O pins (VCCB voltage level)
- **OE:** Output enable (active high)
- **VCCA:** A-side supply voltage
- **VCCB:** B-side supply voltage
- **GND:** Ground

## Power Consumption Summary

### Standby Mode (OE = High, No Switching)
- **VCCA Current:** <2.4 µA
- **VCCB Current:** <12 µA
- **Total:** <14.4 µA

### Active Mode
- **Static:** Same as standby
- **Dynamic:** Depends on switching frequency, load capacitance, and voltage levels

## Datasheet Reference
`/home/erichandley/UVR/srad-camera/datasheets/html/TSXS0104E-html.html`
