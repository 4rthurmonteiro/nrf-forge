---
name: nrf-power
description: Optimizes battery life and power management — sleep states, peripheral suspension, duty cycling, battery charging via USB-C, and current measurement. Use when the user says "bateria", "consumo", "low power", "duração", "carregamento", "dormir", "economia de energia".
argument-hint: power concern to address
---

Optimize power for a battery-powered wall device with USB-C charging.

<input>$ARGUMENTS</input>

## Power model first, code second

Before optimizing, build/refresh a simple budget table in `docs/power-budget.md`: per subsystem (CPU active, each sensor, BLE adv/conn, NB-IoT modem TX, sleep floor) × duty cycle → average µA → battery life estimate. Update it after every measurement. Decisions come from this table, not intuition.

## Technique checklist (nRF54L / Zephyr)

- **Idle is automatic**: with no work pending, Zephyr enters low-power idle. The job is removing things that prevent it: busy loops, short periodic timers, log flushing, always-on peripherals.
- **Peripheral suspension**: `CONFIG_PM_DEVICE=y` (+ `CONFIG_PM_DEVICE_RUNTIME=y`); suspend I2C/SPI/UART/sensor devices between sampling windows; pinctrl `sleep` states on every bus.
- **Sensor duty cycle**: batch reads in one wake window; prefer sensor-side FIFOs/interrupts over polling.
- **Radios are the budget killers**: BLE advertising interval slow when idle; long connection intervals + slave latency on fog links; coordinate NB-IoT modem windows (PSM/eDRX on the modem side) so the device sleeps between uplinks.
- **Logging**: RTT logging keeps debug cheap, but disable/raise log level for release builds — UART consoles cost power.
- **Console/UART off** in release: `CONFIG_SERIAL=n` if unused.
- **Charging**: battery charging + fuel gauge belongs to a PMIC (e.g., Nordic nPM1300 — has NCS drivers and a fuel-gauge sample); charging logic lives in the PMIC, firmware only reads status/SoC over I2C. If the custom board uses another charger IC, wire via nrf-devicetree.

## Measurement loop (mandatory for any power claim)

1. Measure baseline with a Power Profiler Kit II (PPK2) in source-meter mode on the DK (current measurement jumpers per DK docs).
2. Apply ONE change.
3. Re-measure, record in `docs/power-budget.md` (before/after µA).
Never report a power improvement without numbers. If no PPK2 is available, state estimates as estimates.

## Verification habit

After any feature lands (fw-build step 5), check the sleep floor hasn't regressed: a single forgotten 10 ms timer can triple average current.
