# ZMK Corne Keyboard Configuration

ZMK firmware for a Corne split keyboard with:
- **Peripherals**: nice!nano v2 (left + right halves)
- **Central/Dongle**: Raytac MDBT50Q-CX-40 USB-C dongle (nRF52840)
- **ZMK Studio** support for live keymap editing

## About the Dongle

The MDBT50Q-CX-40 is a compact USB-C dongle based on the nRF52840. It ships
with a Nordic nRF5 SDK Open DFU bootloader and runs in LDO-only mode (no DC-DC
converter). The Zephyr board definition `raytac_mdbt50q_db_40_nrf52840` targets
the DB-40 dev board which has different GPIO pins and DC-DC support, so
board-specific overrides in `boards/shields/corne_dongle/boards/` are required
to make it work on the CX-40.

## Flashing the Dongle

Flashing is done over USB using the built-in DFU bootloader — no hardware
programmer needed.

### Prerequisites

Install `nrfutil` with the nRF5 SDK tools plugin:

```bash
pip install nrfutil
nrfutil install nrf5sdk-tools
```

### Download the firmware

1. Go to the **Actions** tab in this GitHub repository
2. Open the latest successful build
3. Download the `mdbt50q_dongle` artifact (`.hex` file)

### Flash the firmware

1. **Enter DFU mode**: Hold the button (SW1) on the dongle, then plug it into USB.
   The blue LED should stay on. A `/dev/ttyACM0` device will appear (Linux)
   or a COM port (Windows).

2. **Create the DFU package**:
   ```bash
   nrfutil nrf5sdk-tools pkg generate \
     --hw-version 52 \
     --sd-req 0x0000 \
     --application zmk.hex \
     --application-version 1 \
     firmware.zip
   ```

3. **Flash via USB DFU**:
   ```bash
   nrfutil nrf5sdk-tools dfu usb-serial \
     --package firmware.zip \
     --port /dev/ttyACM0
   ```

4. The dongle will reboot automatically. It should appear as a USB HID device
   (`1d50:615e`).

### ZMK Studio

To use ZMK Studio for live keymap editing, make sure you flashed the
`mdbt50q_dongle` artifact (which includes the `studio-rpc-usb-uart` snippet).
Then open [ZMK Studio](https://zmk.studio) and connect to the dongle.

## Flashing the nice!nano v2 Halves

1. Download `corne_left` or `corne_right` artifact from the GitHub Actions build
2. Double-tap the reset button on the nice!nano to enter the UF2 bootloader
3. Copy the `.uf2` file to the mounted drive

## Hardware Notes

The MDBT50Q-CX-40 dongle is **not** the same as the MDBT50Q-DB-40 dev board.
Key differences handled by the board-specific overlay:

- **Power**: LDO only (no DC-DC converter) — DC-DC must be disabled or the chip crashes
- **LED**: P0.06 (DB-40 uses P0.13)
- **Button**: P1.06 (DB-40 uses P0.18)
- **Flash layout**: Nordic SDK DFU bootloader with app at 0x1000 (not MCUboot at 0xC000)

## Build Artifacts

| Artifact | Description |
|---|---|
| `mdbt50q_dongle` | Dongle firmware with ZMK Studio support |
| `corne_left` | Left half peripheral firmware |
| `corne_right` | Right half peripheral firmware |
| `mdbt50q_reset_settings` | Settings reset firmware for dongle |
| `nano_reset_settings` | Settings reset firmware for nice!nano |
