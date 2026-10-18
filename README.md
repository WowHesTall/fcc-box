# DroneDD — DJI Mini 3 Pro FCC/CE mode switch

Session-scoped FCC toggle for the DJI Mini 3 Pro (WM162) over the aircraft's USB DUML channel. The RC Pro screen flips to FCC live; an aircraft power-cycle reverts. Confirmed on Mini 3 Pro; other DJI models unconfirmed.

## How it works

The aircraft's air-link module (9,0) accepts DUML writes over USB bulk (interface 4, endpoints 04/85). Writing the AU channel-set frames moves the negotiated radio region onto the FCC-class channel set and the link (and the RC screen) follows. All state is RAM — nothing persistent.

- AU channel group ×2 (`0x07/0x30` → (9,0)) — the live switch
- OFDM power limits (`0x09/0x27` → (9,0), regs 0x48/0x63 = FFFF) — persistence across RC restarts
- Channel map (`0x07/0x18` → (7,0)) + channel flag (`0x07/0x19` → (9,0), ACK echoes "AU")

## Usage

1. RC Pro + drone powered on and linked (screen shows CE).
2. Drone on USB (data cable).
3. `python scripts/drone_fcc.py`
4. Watch the RC screen flip to FCC. Revert: power-cycle the aircraft.

## Dependencies

- Windows, 64-bit Python, pyusb, libusb0 (Zadig driver on the drone's vendor USB interfaces)
- [dji-firmware-tools](https://github.com/o-gs/dji-firmware-tools) cloned to `tools/dji-firmware-tools` (frame builder + CRCs)

## Scripts

- `scripts/drone_fcc.py` — drone-side FCC toggle (the Mod Box sequence)
- `scripts/ac_pwr27.py` — power-limit registers only, with VtConfig readback
- `scripts/fcc_replay.py` — RC-side path: FreeFCC profile over the RC Pro's local DUML proxy (TCP 40009 via adb forward)
- `scripts/duml_get.py` / `duml_set.py` / `duml_capture.py` / `sdr_snapshot.py` — DUML probe/capture harness
- `poc/fccbox/` — ESP32-S3 USB-host PoC (same six frames, standalone board)

## Findings

- LBT (`0x09/0x2d`) does not flip with the mode on WM162 — not a CE/FCC oracle
- VtConfig (`0x09/0x21`) byte 3 varies per link session — negotiation state, not the mode word
- The RC Pro runs a DUML proxy on TCP 40009; sender identity matters (MOBILE_APP (2,4) gets answers where PC (10,0) does not)

## Legal

Bench/chamber only. Exceeding a region's certified TX power may breach national radio regulations. Session-scoped by design — the aircraft forgets on reboot.

## Credits

- [FreeFCC](https://github.com/doesthings/FreeFCC) — the frame profile source
- [dji-firmware-tools](https://github.com/o-gs/dji-firmware-tools) — DUML protocol tooling and dissector
- [DJI-FCC-HACK](https://github.com/M4TH1EU/DJI-FCC-HACK) — original N1 payload behind the 0x27 power-limit step
