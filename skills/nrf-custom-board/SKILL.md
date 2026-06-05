---
name: nrf-custom-board
description: Creates and maintains a custom Zephyr board definition for the product's own PCB based on the nRF54LM20B, migrating from the nRF54LM20 DK. Use when the user says "placa custom", "nossa placa", "board definition", "portar para o hardware final", "bring-up".
argument-hint: board name or bring-up task
---

Port the firmware from the DK to the product's custom nRF54LM20B board.

<input>$ARGUMENTS</input>

## Preconditions

- Custom board schematic/pinout available (ask the user to provide it — PDF or net list).
- Firmware so far kept hardware access in devicetree overlays (enforced by the other skills) — verify this before starting; fix violations first.

## Board definition structure

Create `boards/<vendor>/<board_name>/` in the app repo (out-of-tree board, registered via `CMakeLists`/`board_root`):

```
boards/<vendor>/<board_name>/
├── board.yml                       # name, vendor, soc: nrf54lm20b
├── Kconfig.<board_name>
├── <board_name>_defconfig
├── <board_name>.dts                # the real hardware description
└── <board_name>-pinctrl.dtsi
```

Base the DTS on the DK's (`zephyr/boards/nordic/nrf54lm20dk/`), importing the shared SoC dtsi (`nrf54lm20_a_b.dtsi` lineage), then: correct pinctrl for every routed peripheral, crystal configuration (HFXO/LFXO per BOM — wrong LFXO config breaks BLE timing), regulators/PMIC nodes, and delete DK-only nodes (buttons/LEDs not on the product).

Dispatch `ncs-research-agent` for the current out-of-tree board how-to in NCS 3.3.0 before generating files — board.yml schema details have changed across versions.

## Bring-up sequence (one subsystem at a time, in this order)

1. Power rails verified by the user with a multimeter BEFORE first flash.
2. Minimal image (logging only) → boot banner over RTT (SWD must work first).
3. Clocks: confirm LFXO/HFXO startup (BLE will not work otherwise).
4. GPIO/LED heartbeat → buses (I2C scan each address, SPI loopback) → each sensor → ADC → BLE radio (advertising visible) → DFU path.
5. Keep `west build -b nrf54lm20dk/nrf54lm20b/cpuapp` working in CI/dev: both targets build from the same source, differing only in devicetree.

Record every bring-up finding in `docs/bringup-<board>.md` — hardware bugs found here feed the next PCB revision.
