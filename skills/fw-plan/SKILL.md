---
name: fw-plan
description: Turns a firmware brainstorm or feature request into a step-by-step implementation plan for an nRF Connect SDK project. Use when the user says "planejar", "faz o plano", "/fw-plan", or after fw-brainstorm produced a document.
argument-hint: brainstorm doc path or feature description
---

Produce an executable implementation plan for `/fw-build`. Do NOT implement anything yet.

<input>$ARGUMENTS</input>

## Execution flow

1. **Load context**: read the brainstorm doc if a path was given; otherwise work from the description. Read the current project (CMakeLists.txt, prj.conf, sysbuild.conf, devicetree overlays, `src/`) to understand what exists.
2. **Research**: dispatch `ncs-research-agent` for every NCS subsystem the feature touches that you are not 100% current on (exact Kconfig symbols, recommended sample, API status in NCS v3.3.0). Never plan from stale memory of the SDK.
3. **Draft the plan** with numbered steps. Every step must state:
   - What changes (files: source modules, `prj.conf`, devicetree overlay, `sysbuild.conf`)
   - Why, in plain language the user can follow
   - Verification: how the step is proven done (build passes, log line appears, BLE characteristic readable, current draw target)
   Cover when relevant: Kconfig additions, devicetree changes, new `src/` modules, thread/work-queue design, memory/stack budget, error handling, and impact on DFU image size (2 MB RRAM total).
4. **Size the plan**: small change → keep ≤5 steps; large feature → split into independently buildable milestones (the firmware must compile and run after every milestone).
5. **Review the plan** against the brainstorm requirements; list risks and rollback notes.
6. **Save** to `docs/plan/YYYY-MM-DD-<topic>.md` and present a summary (not the whole file) to the user.
7. **Handoff**: offer "Executar com /fw-build docs/plan/<file>".

## Constraints

- Target: `nrf54lm20dk/nrf54lm20b/cpuapp`, NCS v3.3.0, sysbuild. Custom board comes later via the nrf-custom-board skill — plans must keep hardware access behind devicetree so the port stays cheap.
- Battery-powered wall device: every plan step that adds wakeups, radio time, or always-on peripherals must state its power cost and mitigation.
- Prefer adapting an official NCS/Zephyr sample over writing subsystem code from scratch.
