## MUST HAVE

- Must be able to use either 3.5V or 5V as power source
- At least 6x LDOs capable of low noise power delivery from at least 0.9V to 2.9V. More highly preferred, wider range highly preferred.
- LDOs must be capable of using at LEAST as low as 1.25V as a power source
- At LEAST 1 high efficiency buck converter with max current of MINIMUM 2000mA, MUST be capable of delivering as low as 0.9V (with headroom)
- At LEAST 5x buck converters total
- One LDO must have a relatively low dropout requirement to deliver at least 400mA 

## HIGH PRIORITY

- Must be less than $3 USD (some flexibility for extremely good options)
- Must be available from at least one major distributor:
    - Preferred: JLCPCB
- Must be I2C programmable (voltage levels, etc)
- At least one LDO must be able to deliver 0.675V (VREF for DDR3L @ 1.35V)
- Each LDO must have a relatively low dropout requirement to deliver at least 200mA (not necessarily all of them, at least 3 or 4 though)
- Most buck converters should have at least 1000mA capacity
- Most buck converters should have reliably high efficiency across current ranges

## MEDIUM PRIORITY

- LDOs not tied together to same power source, e.g. LDO 1,2,3 not all tied to VCC5
- Reasonably small package size, preferably less than 10x10mm
- Buck converters should maintain >60% efficiency down to 50mA loads