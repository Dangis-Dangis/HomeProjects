# Control Rails

Every existing device maps to **one of three rails**. Pick the rail, not the device.

## Rail 1 — IR (open-loop)

**Devices:** Samsung sound system, 3 IR light zones, TV power fallback.

**Solution:** One **Broadlink RM4** or **ESPHome IR blaster** learns all remotes → Home Assistant.

**Limitation:** No state feedback unless you add power sensors or later swap to Zigbee/WLED.

### IR RGB accent lights (discrete model)

Not addressable RGB. Each zone exposes roughly **8–20 fixed color swatches**, **5 brightness levels**, and named **programs** (e.g. USA, Christmas, Rainbow). HA maps `(swatch, level, program)` → Broadlink IR command.

```mermaid
flowchart LR
  UI[Remote screen or HA] --> Map[Lookup table per zone]
  Map --> BL[Broadlink]
  BL --> Strip[IR RGB controller]
```

Full spec: [docs/domain/rgb-lights.md](../docs/domain/rgb-lights.md).

## Rail 2 — IP / Cast (two-way)

**Devices:** Vizio V4K65M-08 (`vizio` + `cast`), Chromecast 4K (`cast`/`androidtv`).

**Living-room caveat:** Volume/mute is on the **Samsung system (Rail 1)**, not Cast volume.

## Rail 3 — Mains switch (two-way, button preserved)

**Devices:** All ceiling lights — **on/off only** (non-dimmable fixtures).

**Solution:** Lutron Caseta (no-neutral option) or Shelly relay behind existing switch.

---

## Room asymmetry

| | Living room | Bedroom |
|---|-------------|---------|
| Player | SmartCast apps | Chromecast 4K |
| Volume | IR → Samsung | Cast/CEC |
| Commercial skip | Mute only | SponsorBlock/ADB feasible |
| Best macro | TV + Samsung on (Broadlink) | Cast wake |

**Bedroom = experimentation TV.** **Living room = consolidate two remotes.**

---

## Cross-device matrix

| Capability | Ceiling | IR lights | LR TV+audio | BR TV |
|------------|:-------:|:---------:|:-----------:|:-----:|
| Rail 1 IR | ○ | ● | ● | ○ |
| Rail 2 Cast | ✗ | ✗ | ● | ● |
| Rail 3 switch | ● | ✗ | ✗ | ✗ |
| State feedback | ● | ✗* | partial | ● |

\* Close loop with power plug or WLED swap.

Full v1 analysis: [archive/2026-05-30-v1-exploratory-guides/docs/device-control-similarity.md](../archive/2026-05-30-v1-exploratory-guides/docs/device-control-similarity.md).
