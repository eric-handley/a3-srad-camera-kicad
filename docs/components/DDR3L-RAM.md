# DDR3L RAM (MT41K256M16TW)

## Overview
Micron MT41K256M16TW-107 DDR3L SDRAM
- 4Gb (512MB) x16 configuration
- DDR3L-1866 (933 MHz, 1866 MT/s max)
- Low voltage operation (1.35V vs 1.5V DDR3)
- Temperature range: 0°C to +85°C (extended to +95°C in self-refresh with SRT)

## Power Requirements

### VDD Rail (Core Supply)
- **Voltage:** 1.35V (typical), range 1.283V - 1.45V
- **Standby Current (Self-Refresh IDD6):** 12 mA (max)
- **Record Current:**
  - Active Standby (IDD3N): 40 mA (typical), 42 mA (max)
  - Burst Read (IDD4R): 165 mA (typical), 175 mA (max)
  - Burst Write (IDD4W): 165 mA (typical), 175 mA (max)
  - One Bank Active (IDD1): 84 mA (typical), 87 mA (max)
  - Burst Refresh (IDD5B): 180 mA (typical), 185 mA (max)

### VDDQ Rail (I/O Supply)
- **Voltage:** 1.35V (typical), range 1.283V - 1.45V
- **Current:** Included in VDD measurements above
- **Note:** VDDQ is isolated on-device for improved noise immunity

### VREFCA Rail (Command/Address Reference)
- **Voltage:** 0.675V (0.5 × VDD), range 0.49 × VDD to 0.51 × VDD
- **Current:** <1 mA (reference input, high impedance)
- **Note:** Must be maintained during all modes including self-refresh

### VREFDQ Rail (Data Reference)
- **Voltage:** 0.675V (0.5 × VDD), range 0.49 × VDD to 0.51 × VDD
- **Current:** <1 mA (reference input, high impedance)
- **Note:** Required during active operation, not required during self-refresh

### VTT Rail (Termination - System Level)
- **Voltage:** 0.675V (0.5 × VDDQ)
- **Current:** N/A (not required - RK3566 uses On-Die Termination)

## Total Power Consumption

### Standby Mode (Self-Refresh)
- **Total Current:** 12 mA @ 1.35V
- **Total Power:** ~16 mW
- **Operating Conditions:** CKE LOW, external clock off, all banks idle

### Extended Temperature Self-Refresh (IDD6ET)
- **Total Current:** 16 mA @ 1.35V
- **Total Power:** ~22 mW
- **Operating Conditions:** Temperature 85°C to 95°C with SRT enabled

### Record Mode (Mixed Read/Write at DDR3L-1866)
- **Typical Current:** 80-100 mA @ 1.35V (average for video encoding workload)
- **Peak Current:** 180-185 mA @ 1.35V (during burst refresh)
- **Typical Power:** 110-135 mW (average)
- **Peak Power:** 240-250 mW

## Operating Modes Summary

| Mode                         | Current (mA) | Power (mW) | Description                         |
|------------------------------|--------------|------------|-------------------------------------|
| Self-Refresh                 | 12           | 16         | Lowest power, data retained         |
| Self-Refresh (Extended Temp) | 16           | 22         | 85°C-95°C with SRT                  |
| Active Standby               | 40-42        | 54-57      | All banks precharged, clocks active |
| One Bank Active              | 84-87        | 113-117    | Single bank active operation        |
| Burst Read                   | 165-175      | 223-236    | Continuous read operations          |
| Burst Write                  | 165-175      | 223-236    | Continuous write operations         |
| Burst Refresh                | 180-185      | 243-250    | Peak current during refresh         |

## Notes
- All current values are for worst-case conditions (max ratings)
- Die revision N specifications shown; Die revision P has lower power consumption
- Current specifications vary with data rate (faster speeds = higher current)
- Video encoding workload typically has mixed read/write with average current well below peak
- Memory power consumption correlates with:
  - Memory bandwidth utilization
  - Access pattern (sequential vs random)
  - Refresh rate requirements
- Self-refresh current increases with temperature (use SRT above 85°C)
- VDD and VDDQ must track each other (±50mV maximum delta)

## Design Considerations
- Low-noise power supply critical for reliable operation at DDR3L-1866
- Place decoupling capacitors close to device power pins per datasheet recommendations
- Consider temperature effects on self-refresh current
- VDD and VDDQ must track each other (±50mV maximum delta)

## Datasheet Reference
`/home/erichandley/UVR/srad-camera/datasheets/html/MT41K256M16TW-html.html`
