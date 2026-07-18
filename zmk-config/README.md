# Vessel — ZMK starter config

A starting point for the Vessel trackball on the **Seeed XIAO nRF52840 (Sense Plus)**.
It builds as a **shield** (`vessel`) on the `seeeduino_xiao_ble` board.

## Layout

```
zmk-config/
  build.yaml                      # GitHub Actions build matrix
  config/
    west.yml                      # pulls ZMK + the PMW3610 driver module
    vessel.conf                   # feature flags (BLE, sleep, pointing, PMW3610)
    vessel.keymap                 # keymap + scroll binding
    boards/shields/vessel/
      vessel.overlay              # pins: buttons, scroll encoder, trackball SPI
      vessel.zmk.yml  Kconfig.*   # shield metadata
```

## How to build

1. Push this to a GitHub repo (or use ZMK's `zmk-config` template and drop these in).
2. GitHub Actions builds `vessel` on `seeeduino_xiao_ble` → download `zmk.uf2`.
3. Double-tap the XIAO reset to enter the UF2 bootloader, drag `zmk.uf2` on.

## Pin map (already filled in)

| Function | XIAO pin | in overlay |
|---|---|---|
| Click M1 / M2 | D0 / D1 | `kscan0` |
| Scroll hall A / B | D2 / D3 | `scroll` (EC11) |
| Haptics I²C (DRV2605L) | D4 / D5 | *(not wired in FW yet — see below)* |
| LED DIN | D6 | *(commented, optional)* |
| PMW3610 MOT / SCK / nCS / SDIO | D7 / D8 / D9 / D10 | `&spi1` + `trackball` |

## What's turnkey vs. what you finish

**Works as-is:** BLE, deep-sleep, the two mouse-click buttons, and the scroll
ring (mapped to volume by default).

**Add the module, then it works:** the **trackball**. `west.yml` already pulls
[`badjeff/zmk-pmw3610-driver`](https://github.com/badjeff/zmk-pmw3610-driver);
the overlay node uses `pixart,pmw3610-alt`. After the first flash, tweak
`swap-xy; invert-x; invert-y;` in the overlay if the cursor moves the wrong way.

**You fill in (marked `TODO`):**
- **Extra buttons 01–04** — they live on the 9 Sense-Plus **castellated pads**,
  which aren't in the standard XIAO pin map. Look up their `P0.xx/P1.xx` numbers
  on the Seeed pinout, add them to `kscan0` `input-gpios`, and add one binding
  each to the keymap (in the same order).
- **3-wire SPI** — PMW3610 has one bidirectional data line (SDIO). The overlay
  parks `MISO` on an unused pin (P0.06) so Zephyr is happy; confirm against the
  driver README and your wiring.
- **Scroll → mouse wheel** instead of volume: swap the `sensor-bindings` line.

**Needs custom work (not in a starter):**
- **Haptics (DRV2605L)** — ZMK has no stock "buzz on keypress" behavior. Wire it
  to D4/D5 (I²C) and drive it from a custom behavior/module.
- **Mic push-to-talk / IMU tilt** — see the Build & Wiring Guide. PTT is easiest
  as a button that sends your OS dictation hotkey; real onboard-mic streaming and
  IMU gestures are custom firmware.

## Dongle

The starter connects the Vessel **straight to your computer over BLE** — no dongle
needed. Your second XIAO can later become a USB "dongle" (a BLE central), which is
a separate advanced setup; a stub is commented in `build.yaml`.
