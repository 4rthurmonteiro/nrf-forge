---
name: fw-build
description: Executes a firmware implementation plan — writes the C code, builds with west after each step, fixes errors, and runs quality review agents before declaring done. Use when the user says "/fw-build", "implementa", "executa o plano", or "bora codar".
argument-hint: plan doc path (or feature description for small changes)
---

Execute an implementation plan end to end. The user does not read or write C — you own all code.

<input>$ARGUMENTS</input>

## Ground rules

- Explain each step's outcome in plain language (what the device now does), never by narrating pointer mechanics. Show C code only if the user explicitly asks.
- The firmware must compile after every step: run the project's build command (typically `west build -b nrf54lm20dk/nrf54lm20b/cpuapp --sysbuild` from the app directory, inside the NCS toolchain environment) and fix all errors and warnings before moving on. Treat warnings as defects.
- Hardware access only through devicetree APIs; configuration only through Kconfig/devicetree — never edit SDK files.

## Execution flow

1. **Load the plan** (or write a micro-plan inline for trivial changes). Confirm scope with the user if the plan has open questions.
2. **Implement step by step**: code → build → verify per the step's verification criteria → mark the step done. On build errors, read the full error, fix root cause; if the error is obscure, consult the nrf-build-flash skill's troubleshooting reference or dispatch `ncs-research-agent`.
3. **Flash & smoke-test** when hardware is available: `west flash`, then check boot via RTT/UART logs (see nrf-build-flash skill). If no hardware is attached, say so and list the manual test steps for the user.
4. **Quality gate** — mandatory, in parallel:
   - `memory-safety-review-agent` on all changed C files
   - `zephyr-review-agent` on the changeset
   Fix every critical and major finding, rebuild, and re-run the failing agent until PASS.
5. **Report**: summarize what the device now does, the verification evidence (build output, log lines), review verdicts, and remaining risks. Update the plan doc with completion status.

## Key principles

- Small steps, always buildable. No "big bang" commits of 10 files.
- When a step proves harder than planned, stop and revise the plan with the user instead of improvising architecture.
- Battery first: after the feature works, confirm no new busy-waits, tight timers, or always-on peripherals slipped in.
