# Domain — Unified Remote Controls

Hardware and UX rules for the DIY ESPHome deck. Supersedes archive v3 "3 reusable hue/brightness dials" where this doc conflicts.

---

## Design principles

1. **One analog dial: audio volume only** — living room (Samsung via IR macro) and bedroom (Cast/CEC) depending on context. No hue, saturation, or brightness dials.
2. **RGB accent lights** — discrete swatches + 5 brightness levels + programs on the **screen** — see [rgb-lights.md](./rgb-lights.md).
3. **Dedicated hardware buttons** — only for high-frequency, unambiguous actions with **fixed labels**.
4. **Screen-driven actions** — everything else shows its **current function on the backlit display**; physical buttons can be soft-mapped to whatever the screen shows.

---

## Control layout (recommended)

| Control | Type | Function |
|---------|------|----------|
| **Volume dial** | 1× rotary encoder | Audio volume only (context: LR Samsung IR or BR Cast) |
| **Sync Screens** | Dedicated button + screen label | Cast same YouTube/link to both TVs — **keep** |
| **ALL ON** | Dedicated button + screen label | Ceilings on + RGB last state or default warm — household macro |
| **GOODNIGHT** | Dedicated button + screen label | Ceilings off, RGB off, TVs sleep, cameras armed optional |
| **Action row (×2–4)** | Physical buttons **with screen soft labels** | Screen shows current label (e.g. "Cameras On", "Watch TV", "Color +", "Bright +") |
| **ROOM / TYPE** | Buttons or on-screen | Living/Bedroom; RGB / Ceiling |
| **Screen** | 1.9" IPS minimum | Context header, soft button labels, RGB swatch picker, action log |
| **PTT + mic** | Button | Voice → HA Assist |
| **Status LEDs** | WS2812 array | Per-zone + command ack |

**Removed / deprecated**

- ~~Party Time~~ dedicated button — use screen program picker (Rainbow, USA, Christmas) under RGB Type, or a soft action if needed.
- ~~Night Time~~ as duplicate of GOODNIGHT — consolidate.
- ~~3 reusable hue/brightness/sat dials~~ — wrong for IR RGB and user preference.

---

## RGB on the screen (not on dials)

```
┌─────────────────────────────┐
│ ROOM: Living   TYPE: RGB    │
├─────────────────────────────┤
│ Color: ● Blue 2             │  ← prev/next via soft buttons or touch
│ Brightness: ███░░  3/5      │
│ Program: [Static ▼]         │  ← USA / Christmas / Rainbow
├─────────────────────────────┤
│ [Sync] [ALL ON] [GOODNIGHT] │  ← fixed hardware
│ [Act1: Watch TV] [Act2: …]  │  ← labels from screen
└─────────────────────────────┘
```

---

## Ceiling lights (TYPE: Ceiling)

- On/off and fixture/group select on **screen** + soft buttons — not a dial (non-dimmable).

---

## Connectivity (unchanged)

Remote → **Wi-Fi → Home Assistant only** → Cast / Broadlink / switches. See [projects/05-unified-remote/README.md](../../projects/05-unified-remote/README.md).

---

## BOM impact

- **1× encoder** (volume) instead of 3 — simpler PCB, smaller enclosure option.
- **IPS screen required** — soft labels are core UX, not optional.

Wiring: [projects/05-unified-remote/wiring.md](../../projects/05-unified-remote/wiring.md).
