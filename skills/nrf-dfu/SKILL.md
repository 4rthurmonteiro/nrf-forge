---
name: nrf-dfu
description: Sets up secure firmware updates (DFU/OTA) with MCUboot on the nRF54LM20B — image signing, KMU key provisioning, DFU over BLE (SMP) or USB, and devicetree partitioning. Use when the user says "DFU", "OTA", "atualização de firmware", "bootloader", "MCUboot", "assinar imagem".
argument-hint: DFU transport or task (ble | usb | keys | partitions)
---

Add or operate secure DFU on the nRF54LM20B. NCS v3.3.0 fully supports DFU on this SoC.

<input>$ARGUMENTS</input>

## Architecture decisions (confirm with the user once, record in docs/plan/)

1. **Layout**: classic dual-slot (swap; safest, halves usable space within 2 MB RRAM) vs single-slot "firmware loader" (more app space, update mode requires entering the loader). Default recommendation: dual-slot while the image is < ~800 KB.
2. **Transport**: SMP over BLE (phone/fog updates — natural for this product) and/or USB MCUmgr (bench updates, `SB_CONFIG_FIRMWARE_LOADER_IMAGE_USB_MCUMGR` in single-slot mode); serial recovery as fallback.
3. **Keys**: ED25519 signatures, hardware-accelerated by CRACEN, public key provisioned to the KMU. NEVER commit private keys — keep them outside the repo and document the storage location.

## Implementation outline

- Enable in `sysbuild.conf`: `SB_CONFIG_BOOTLOADER_MCUBOOT=y` plus signature/key configs; app-side `prj.conf`: SMP server (`CONFIG_MCUMGR=y`, transport configs `CONFIG_MCUMGR_TRANSPORT_BT=y` for BLE).
- Partitions are devicetree `fixed-partitions` in NCS 3.3.0 (Partition Manager is deprecated — ignore older tutorials); verify slot sizes against `west build -t rom_report`.
- Key provisioning to KMU uses the `west ncs-provision` flow during production flash; dispatch `ncs-research-agent` for the exact current command sequence before executing — this changed across NCS versions and is the most error-prone step.
- Version every image via the `VERSION` file; MCUboot rejects downgrades when anti-rollback is on.

## Verification (mandatory, on hardware)

1. Build → flash full image → confirm boot.
2. Bump version, build update image (`build/<app>/zephyr/zephyr.signed.bin`).
3. Deliver via chosen transport (nRF Connect Device Manager app for BLE) → confirm swap, new version in boot log.
4. Test the failure path: corrupt/unsigned image must be rejected; power-loss mid-update must roll back.
DFU is not "done" until the failure paths pass.
