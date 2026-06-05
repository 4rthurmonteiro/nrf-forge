---
name: nrf-edge-ai
description: Plans and implements on-device AI on the nRF54LM20B — sound classification, IAQ anomaly detection, and other ML workloads using the Axon NPU, the FLPR coprocessor, or CPU inference. Use when the user says "edge AI", "IA embarcada", "modelo", "NPU", "classificação de som", "machine learning", "inferência".
argument-hint: AI feature or model task
---

Implement Edge AI features on the nRF54LM20B — the product's differentiator.

<input>$ARGUMENTS</input>

## Hardware options on this SoC

1. **Axon NPU @128 MHz** (the LM20B's differentiator vs LM20A): designed for workloads like sound classification and keyword spotting, ~15x faster than M33 inference. **Tooling status is evolving** — the chip reached GA in Q2 2026. ALWAYS dispatch `ncs-research-agent` first to check the current state of Nordic's NPU SDK/compiler support in the installed NCS version before promising NPU execution. Do not write NPU code from memory.
2. **FLPR RISC-V coprocessor @128 MHz**: ideal for always-on sensor preprocessing (PDM audio feature extraction, decimation) feeding either the NPU or the M33; built via sysbuild with the `cpuflpr` target.
3. **Cortex-M33 + CMSIS-NN / TFLite Micro**: the always-available fallback; fine for small models (IAQ anomaly detection, low-rate classifiers).

## Workflow

1. **Define the task as ML**: input signal, window size, classes/outputs, accuracy target, latency target, and power budget per inference (tie into `docs/power-budget.md`).
2. **Data pipeline first**: implement and verify sensor capture (PDM mic, IAQ, lux via nrf-devicetree skill) and a way to export labeled data (USB/BLE/serial dump) before any model talk.
3. **Train externally** (Edge Impulse is well-suited and supports Nordic targets; plain TF/PyTorch + TFLM conversion also works). Track dataset/model versions in `docs/ml/`.
4. **Deploy**: start with M33/TFLM int8 inference to prove the feature end-to-end; migrate hot models to the NPU once research confirms toolchain support — keep the inference call behind a single `src/ml/` API so the backend can swap without touching callers.
5. **Verify on-device**: accuracy spot-check against the offline test set, inference time (log timestamps), RAM/flash footprint (`rom_report`/`ram_report`), and battery impact (nrf-power measurement loop).

## Constraints

- 512 KB RAM / 2 MB RRAM shared with the app, BLE stack, and MCUboot slots — model size budgets are real; quantize to int8 by default.
- Audio features for sound-pressure analytics can stay simple (dB(A) bands) — use ML only where simple DSP fails; record this decision per feature in the brainstorm doc.
