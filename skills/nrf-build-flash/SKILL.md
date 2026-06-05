---
name: nrf-build-flash
description: Builds, flashes, and debugs nRF Connect SDK firmware — west commands, toolchain environment, RTT/UART logs, and troubleshooting of build and flash errors. Use when the user says "compila", "builda", "flash", "grava", "não compila", "deu erro no build", or asks to see device logs.
argument-hint: optional action (build | flash | logs | erro <message>)
---

Operate the build/flash/debug loop for the project. Target: `nrf54lm20dk/nrf54lm20b/cpuapp`, NCS v3.3.0, sysbuild.

<input>$ARGUMENTS</input>

## Environment

All `west` commands must run inside the NCS toolchain environment. If commands fail with "west not found" or wrong toolchain:

```bash
nrfutil sdk-manager toolchain launch --ncs-version v3.3.0 --shell   # Linux/macOS
```

(macOS SDK lives in `/opt/nordic/ncs/`, Linux `~/ncs`.) J-Link ≥ v9.24a is required.

## Core commands

```bash
west build -b nrf54lm20dk/nrf54lm20b/cpuapp --sysbuild   # build
west build                                                # incremental rebuild
west build -t pristine                                    # clean
west flash                                                # program via J-Link (DK onboard)
west flash --erase                                        # full chip erase + program
```

Logs: RTT via `JLinkRTTLogger`/`west attach`, or UART console on the DK VCOM port (115200 8N1). Prefer RTT on this project (logging configs already in prj.conf).

## Debug workflow (the user never reads C — you do)

1. Reproduce: build or boot, capture the FULL error/log output.
2. For build errors: read the first real error (not the cascade), open the file/Kconfig involved, fix root cause. Check `references/troubleshooting.md` for known nRF54L/NCS 3.3.0 issues before guessing.
3. For runtime faults: grab the fault dump from RTT (`ZEPHYR FATAL ERROR`, faulting address, thread), correlate with `build/<app>/zephyr/zephyr.map` and `addr2line` to find the source line, then fix and re-run the memory-safety-review-agent on the fixed file.
4. Explain to the user what was wrong and what changed, in product terms.
5. If stuck after two root-cause attempts, dispatch `ncs-research-agent` with the exact error text.

## Verification habit

After any flash, always confirm the boot banner and absence of error/warning logs before declaring success.
