---
name: nrf-new-project
description: Scaffolds a new freestanding nRF Connect SDK v3.3.0 application for the nRF54LM20B with sysbuild, proper structure, and a first successful build. Use when the user says "criar projeto", "novo firmware", "scaffold", "começar o projeto do zero".
argument-hint: project name and short purpose
---

Create a new NCS v3.3.0 application targeting `nrf54lm20dk/nrf54lm20b/cpuapp`.

<input>$ARGUMENTS</input>

## Execution flow

1. **Confirm basics** if unknown: project name (kebab-case), location, and which features go in the first build (recommend: boot + logging only; features come later via fw-brainstorm → fw-plan → fw-build).
2. **Generate the structure** from `references/project-template.md` (exact file contents there):

```
<name>/
├── CMakeLists.txt
├── prj.conf
├── sysbuild.conf
├── VERSION
├── boards/
│   └── nrf54lm20dk_nrf54lm20b_cpuapp.overlay
├── src/
│   └── main.c
└── docs/            (brainstorm/, plan/, reviews/ — workflow artifacts)
```

3. **First build**: inside the NCS toolchain environment, run
   `west build -b nrf54lm20dk/nrf54lm20b/cpuapp --sysbuild` from the app directory.
   Fix any environment issues using the nrf-build-flash skill. A scaffold is not done until it compiles clean.
4. **Optional flash**: if the DK is connected, `west flash` and confirm the boot log over RTT/UART.
5. **Set up git**: `git init`, a Zephyr-appropriate `.gitignore` (`build/`, `.vscode/` private settings), first commit.
6. **Report** in plain language: what was created, build result, and suggest `/fw-brainstorm` for the first feature.

## Rules

- Freestanding app (outside the SDK tree) using the toolchain-managed NCS v3.3.0 workspace — do not `west init` a new workspace unless the user asks.
- sysbuild always (`--sysbuild`), so MCUboot/DFU can be added later without restructuring.
- Devicetree overlays in `boards/`, never editing SDK board files — this keeps the path open for the future custom board (nrf-custom-board skill).
- No Partition Manager (deprecated in NCS 3.3.0); when partitions become necessary use DTS `fixed-partitions`.
