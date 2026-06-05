---
name: nrf-ble
description: Implements Bluetooth LE features in nRF Connect SDK — peripheral (GATT services, advertising, pairing for a config app) and central (scanning and connecting to fog devices), including dual-role operation. Use when the user says "BLE", "bluetooth", "GATT", "advertising", "conectar com outro dispositivo", "app de configuração".
argument-hint: BLE feature to implement
---

Implement BLE on the nRF54LM20B (SoftDevice Controller, BLE 6.0 capable, Channel Sounding available).

<input>$ARGUMENTS</input>

## Project context

This device is dual-role: **peripheral** for a phone config app, **central** (and/or scanner) toward other fog devices, while NB-IoT handles cloud traffic out-of-band. Design GATT and connection topology accordingly.

## Implementation rules

- Base configs: `CONFIG_BT=y`, plus `CONFIG_BT_PERIPHERAL=y` / `CONFIG_BT_CENTRAL=y` as needed; set `CONFIG_BT_MAX_CONN` for the fog fan-out; name via `CONFIG_BT_DEVICE_NAME`.
- Start from an official sample and adapt: `nrf/samples/bluetooth/peripheral_lbs` (custom GATT service pattern), `peripheral_uart` (NUS data pipe), `central_uart` (scan + connect + GATT client). Dispatch `ncs-research-agent` to confirm current sample APIs before coding.
- Custom GATT services via `BT_GATT_SERVICE_DEFINE` with 128-bit UUIDs; notifications for telemetry, write-with-response for configuration; validate ALL incoming write lengths/values (memory-safety-review-agent will check).
- Callbacks from the BT stack must not block: hand data to a work queue or message queue immediately.
- Security: for the config app, require LE Secure Connections pairing (`CONFIG_BT_SMP=y`, bonding) before exposing config characteristics; protect characteristics with `BT_GATT_PERM_*_ENCRYPT` (or `_AUTHEN`).
- Power: advertising interval is a battery knob — slow advertising (1–2 s) when idle, fast only during commissioning windows. Connection intervals for fog links should be as long as latency requirements allow.
- Coexistence: if 802.15.4 or heavy scanning is added later, revisit scheduler load; document chosen intervals in the plan doc.

## Execution flow

1. Clarify role(s), data model (characteristics: name, type, size, direction, security), and lifecycle (when advertising starts/stops).
2. Plan via fw-plan for anything beyond one service; small additions can be implemented directly.
3. Implement in a dedicated `src/ble.c` module exposing init + send APIs; keep GATT tables and bt callbacks internal.
4. Build, flash, verify end-to-end with a phone (nRF Connect for Mobile app) or a second DK: advertising visible, connect, read/write/notify each characteristic.
5. Run the quality gate (both review agents) — BLE input handling is the most security-sensitive surface of this product.
