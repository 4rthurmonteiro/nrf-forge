# Troubleshooting — NCS v3.3.0 / nRF54LM20B

Known classes of problems and fixes. Always read the FIRST error in the output.

## Build

| Symptom | Cause / fix |
|---|---|
| `Board nrf54lm20dk/nrf54lm20b/cpuapp not found` | NCS < 3.3.0 active. Check `west list zephyr` / toolchain env points to v3.3.0 |
| `undefined reference to ...` for a subsystem API | Missing `CONFIG_*` in prj.conf — find the exact symbol with the ncs-research-agent, don't guess |
| Devicetree error `__device_dts_ord_...` or `DT_N_...` undefined | Node disabled or missing in overlay: ensure `status = "okay"` and required properties; check the binding's required props |
| Kconfig warning "attempted to assign CONFIG_X" | Symbol doesn't exist in this NCS version or depends on another symbol — check dependencies with `west build -t menuconfig` |
| Image too big / region RRAM overflow | 2 MB NVM total, shared with MCUboot slots once DFU is on. Check `rom_report` (`west build -t rom_report`) and trim logs/features |
| Partition Manager (`pm_static.yml`) docs found online | PM is deprecated in 3.3.0 — use DTS `fixed-partitions` instead; ignore pre-3.3 tutorials |
| `SB_CONFIG_*` ignored in prj.conf | Sysbuild symbols go in `sysbuild.conf`, not prj.conf |
| Build works but settings/NVS fails at runtime | RRAM, not flash: check storage partition definition and `zephyr,settings-partition` chosen node |

## Flash / debug

| Symptom | Cause / fix |
|---|---|
| `west flash` fails, no emulator found | DK not enumerated or old J-Link. Requires J-Link ≥ 9.24a; check `nrfutil device list` |
| Flash OK but device looks dead | No log backend configured, or app crashed pre-log-init. Try `west flash --erase` (stale data in RRAM), then RTT attach immediately |
| `Readback protection` / AP lock errors | Device lifecycle state locked: recover with `nrfutil device recover` |
| MCUboot: image rejected after DFU enabled | Signature/KMU key mismatch — re-provision keys (see nrf-dfu skill), confirm `west ncs-provision` step ran |
| RTT prints nothing | Wrong RTT block address after rebuild — restart the RTT logger after every flash |

## Runtime faults

- `ZEPHYR FATAL ERROR 2: Stack overflow` → increase the named thread's stack (`CONFIG_MAIN_STACK_SIZE`, `K_THREAD_STACK_DEFINE` size), then have memory-safety-review-agent check for oversized locals.
- `ZEPHYR FATAL ERROR 0: CPU exception` with faulting address near 0 → NULL dereference; map PC via `addr2line -e build/<app>/zephyr/zephyr.elf <addr>`.
- Random hangs with BLE on → check for blocking calls in BT RX context callbacks; offload to a work queue.
- Brownouts on battery → check radio TX peaks vs battery internal resistance; lower TX power before chasing software ghosts.
