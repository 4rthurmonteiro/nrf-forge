---
name: fw-review
description: Runs an on-demand quality review of firmware code — memory safety and Zephyr/NCS best practices — without building a feature. Use when the user says "revisa o código", "review", "tá seguro isso?", or before a release.
argument-hint: path or scope to review (default: uncommitted changes, else src/)
---

Run the firmware quality gate standalone.

<scope>$ARGUMENTS</scope>

## Execution flow

1. **Determine scope**: the given path; otherwise uncommitted git changes; otherwise the whole `src/` tree plus `prj.conf` and overlays.
2. **Dispatch in parallel**:
   - `memory-safety-review-agent`
   - `zephyr-review-agent`
3. **Consolidate**: merge both reports, deduplicate, order by severity. Save to `docs/reviews/YYYY-MM-DD-consolidated.md`.
4. **Report to the user in plain language**: overall verdict, the critical issues described by their consequence ("o buffer do BLE pode estourar e travar o dispositivo" — not pointer jargon), and ask whether to apply the fixes.
5. **If the user approves fixes**: apply them, rebuild (`west build`), and re-run the agents until PASS.
