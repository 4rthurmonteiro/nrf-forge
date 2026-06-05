---
name: nrf-devicetree
description: Edits devicetree overlays and wires sensors and peripherals (I2C, SPI, UART, ADC, PWM, GPIO) into Zephyr on the nRF54LM20B. Use when the user says "conectar sensor", "ligar no pino", "devicetree", "overlay", "I2C", "SPI", "ADC", or mentions a specific sensor part number.
argument-hint: peripheral or sensor to wire up
---

Wire hardware into the firmware via devicetree, keeping all hardware knowledge out of the C code.

<input>$ARGUMENTS</input>

## Principles

- All changes go in the app overlay (`boards/nrf54lm20dk_nrf54lm20b_cpuapp.overlay`), never in SDK board files. This is what makes the future custom-board port cheap.
- If Zephyr has a driver for the sensor (check `zephyr/drivers/sensor/` and the compatible in `zephyr/dts/bindings/sensor/`), use it with the sensor API (`sensor_sample_fetch`/`sensor_channel_get`) — no register-level code. If no driver exists, say so and propose: closest existing driver, or a minimal custom driver module (then plan it via fw-plan).
- nRF54L pin specifics: pins are `P0.x`, `P1.x`, `P2.x`; fast peripherals are restricted to specific ports — verify the instance/pin combination in the nRF54LM20B datasheet or DK schematic before assigning. Every peripheral needs a `pinctrl` configuration.
- After any overlay change: full rebuild and check the generated `build/<app>/zephyr/zephyr.dts` to confirm the node resolved as intended.

## Execution flow

1. Identify the bus, part number, voltage, and intended pins (ask the user only for what the schematic/DK can't answer).
2. Check driver availability (dispatch `ncs-research-agent` if unsure of compatible string or driver status in NCS 3.3.0).
3. Write the overlay node — see `references/overlay-examples.md` for I2C sensor, SPI device, ADC channel, PWM, and GPIO patterns with pinctrl.
4. Enable needed Kconfig (`CONFIG_I2C`, `CONFIG_SPI`, `CONFIG_ADC`, `CONFIG_SENSOR`, driver-specific symbols) in prj.conf.
5. Add/extend a `src/` module that exposes a clean function like `sensors_read_iaq()` — devicetree macros stay inside that module.
6. Build, flash if possible, verify with a log line printing a real reading. A sensor is not "integrated" until a plausible value appears in the log.
