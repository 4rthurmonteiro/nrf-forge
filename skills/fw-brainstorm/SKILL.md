---
name: fw-brainstorm
description: Explores firmware feature requirements through collaborative dialogue before planning. Use when the user says "brainstorm", "quero adicionar", "estou pensando em", "explorar ideia", or describes a new firmware feature or product capability without a plan yet.
argument-hint: feature or idea to explore
---

Explore the requirements of a firmware feature through dialogue, producing a brainstorm document that feeds `/fw-plan`. Do NOT write code in this phase.

<feature>$ARGUMENTS</feature>

If the feature is empty, ask the user what they want to explore.

## Ground rules

- Communicate in the user's language. Never ask the user about C, pointers, or memory — translate every technical decision into product/behavior terms (battery life, range, latency, reliability, cost).
- One question at a time, using AskUserQuestion with concrete options. Stop when you can write the document, not when questions run out.

## Execution flow

1. **Restate** the feature in one paragraph; confirm understanding.
2. **Probe what matters for firmware**, only where unknown:
   - Behavior: what triggers it, what the device does, what the user/cloud sees
   - Data: what is measured/sent, how often, payload size
   - Power: always-on vs duty-cycled; acceptable battery impact
   - Connectivity: BLE role (peripheral/central/both), pairing/bonding, coexistence with NB-IoT gateway traffic
   - Hardware: which sensors/pins/peripherals are involved; DK now vs custom board later
   - Failure modes: what must happen on sensor failure, disconnect, power loss
   - Update path: does this feature affect DFU/OTA?
3. **Sketch 2–3 implementation approaches** in plain language with trade-offs (e.g., "interrupt-driven sampling — better battery, more complex" vs "periodic polling — simpler, costs ~X µA"). Dispatch `ncs-research-agent` if you need current NCS capability facts to make approaches honest.
4. **Recommend one** approach and confirm with the user.
5. **Write the document** to `docs/brainstorm/YYYY-MM-DD-<topic>.md`:
   - Context and goal
   - Requirements (functional, power, connectivity)
   - Approaches considered with trade-offs
   - Chosen approach and rationale
   - Open questions
6. **Handoff**: offer "Seguir para /fw-plan com este documento" as the next step.
