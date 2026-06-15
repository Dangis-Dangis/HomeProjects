# Device Control Similarity (Your Existing Lighting & TVs)

How the control methods for your **specific** hardware overlap. The headline: your devices fall into **three control "rails,"** and most solutions reuse the same rail rather than inventing new ones.

---

## Your inventory

| Device | Location | How it's controlled today | Audio path |
|--------|----------|---------------------------|------------|
| Ceiling lights | Multiple rooms | **Wall switch only**, non-dimmable bulbs, fixtures fixed | — |
| Other lights (×3 zones) | Multiple | **IR remote only, no buttons** (3 separate IR controllers) | — |
| Vizio **V4K65M-08** 65" 4K | Living room | Vizio SmartCast built-in apps; **old Samsung sound system via ARC**, controlled by **IR (≈2 remotes)**; old Roku present, unused | Samsung system (ARC + IR) |
| Vizio **E55-U** 55" 4K | Bedroom (above PC) | **Chromecast 4K** primarily; SmartCast is early/limited on this model | TV speakers, or ARC soundbar |

---

## The three control "rails"

Everything you own reduces to one of three integration rails. Pick the rail, not the device.

### Rail 1 — IR (line-of-sight, open-loop)

| Device on this rail | Why |
|---------------------|-----|
| Samsung sound system (living room) | Only IR; no IP/CEC you can rely on (hence 2 remotes) |
| 3 IR light controllers | IR-only, no physical buttons |
| Vizio TVs (power/volume fallback) | IR works even when IP control is flaky |

**Single solution covers all of them:** one IR strategy — **Broadlink RM4** (Wi‑Fi→IR) or **ESPHome IR blaster** (ESP32 + IR LED) — learns every remote and exposes them to Home Assistant. This is the biggest similarity in your setup: **the Samsung sound bar problem and the "lights with no buttons" problem are the same problem.**

- **Limitation (shared):** IR is **open-loop** — no state feedback. HA "thinks" a light is on but can't confirm. Mitigate with assumed-state + occasional resync, or move that device to Rail 3.
- **Line-of-sight (shared):** one blaster per room/zone with an emitter aimed at each target; budget 1 blaster living room (TV + Samsung), 1–3 for the IR light zones depending on room spread.

### Rail 2 — IP / Cast (stateful, two-way)

| Device on this rail | Integration | What you get |
|---------------------|-------------|--------------|
| Vizio V4K65M-08 (living room) | HA `vizio` (SmartCast API) | Power, volume*, input, app launch, `play_media` |
| Chromecast 4K (bedroom) | HA `cast` / `androidtv` | Play/pause, volume, YouTube URLs, app launch, ADB |
| Roku (unused) | HA `roku` | Optional backup target only |

\***Volume caveat (living room):** the Vizio integration controls the *TV's* volume, but your sound comes from the **Samsung system over ARC**. Unless CEC volume pass-through works (unreliable on old gear — your 2-remote situation confirms it doesn't), **volume/mute must travel Rail 1 (IR to the Samsung system)**, not Rail 2.

### Rail 3 — Mains switching / smart relay (stateful, physical button preserved)

| Device on this rail | Solution | Keeps physical control? |
|---------------------|----------|-------------------------|
| Ceiling lights | Smart **on/off** switch or relay behind switch | Yes (your hard requirement) |

Because the bulbs are **non-dimmable and fixtures are fixed**, this rail is **on/off only** — no dimmers, no smart bulbs (a wall switch cutting power would orphan smart bulbs). Options keep the wall as a working switch.

---

## Similarity map

```
        ┌────────────────────── Home Assistant ──────────────────────┐
        │                                                             │
   Rail 1: IR                Rail 2: IP/Cast              Rail 3: Mains
   (Broadlink/ESPHome)       (media_player)               (smart switch/relay)
        │                          │                            │
   Samsung sound         Vizio SmartCast (LR)            Ceiling lights
   3 IR light zones      Chromecast 4K (BR)              (on/off, button kept)
   TV IR fallback        Roku (backup)
```

**Reuse insight:** A single Broadlink/ESPHome IR layer (Rail 1) simultaneously solves **audio control**, **the IR-only lights**, and **TV power fallback**. A single `media_player` layer (Rail 2) solves **both TVs + screen share**. A single smart-switch standard (Rail 3) solves **all ceiling lights**.

---

## Cross-device solution matrix

| Capability | Ceiling lights | IR lights | Living room TV+audio | Bedroom TV |
|------------|----------------|-----------|----------------------|-----------|
| Rail 1 IR (Broadlink/ESPHome) | ○ | ● | ● (audio + power) | ○ (fallback) |
| Rail 2 IP/Cast | ✗ | ✗ | ● (TV/apps) | ● (Chromecast) |
| Rail 3 smart switch | ● | ✗ | ✗ | ✗ |
| State feedback | ● (Rail 3) | ✗ (IR) | partial | ● (Cast/ADB) |
| Physical button preserved | ● | n/a | n/a | n/a |

● primary · ○ secondary/fallback · ✗ not applicable

---

## Where the rooms diverge (don't copy bedroom plan to living room)

| Concern | Living room (V4K65M-08 + Samsung) | Bedroom (E55-U + Chromecast 4K) |
|---------|-----------------------------------|----------------------------------|
| Primary player | Vizio SmartCast apps | Chromecast with Google TV |
| Volume/mute | **IR → Samsung system** | Cast/CEC to TV or soundbar |
| App launch in HA | `vizio` (model-dependent) | `cast`/`androidtv` (full) |
| Commercial **mute** | Audio detector → IR mute Samsung | ADB / SmartTubeNext |
| Commercial **skip** | Not practical (SmartCast apps) | **Feasible** (Google TV + SponsorBlock/ADB) |
| Best "watch" macro | Broadlink: TV on + Samsung on + input | Cast wake + soundbar via CEC |

**Takeaway:** the **bedroom Chromecast 4K is your experimentation TV** (commercial skip, ADB, SmartTubeNext). The **living room is a "consolidate 2 remotes into 1 HA macro"** problem driven by IR.

---

## Recommended unified control plan

| Rail | Buy | Covers | Rough cost |
|------|-----|--------|-----------|
| 1 — IR | 1× Broadlink RM4 Pro (living room) + 1–2 ESPHome IR nodes for distant IR-light zones | Samsung audio, 3 IR light zones, TV power fallback | $35 + $8–16 |
| 2 — IP/Cast | $0 (use existing Vizio SmartCast + Chromecast 4K) | Both TVs, screen share | $0 |
| 3 — Mains | Smart on/off switches **or** Shelly relays behind existing switches | All ceiling lights | $25–60 per switch |

**Better-than-IR upgrade (optional):** if the "3 IR controllers" drive LED strips, swap each IR receiver for a **Zigbee/WLED controller** (~$20–30) to gain real state + dimming on Rail 3-style stateful control — eliminates the open-loop IR weakness for those lights.

See [home-assistant-integration.md](./home-assistant-integration.md) for full solutions and [screen-share-multiple-tvs.md](./screen-share-multiple-tvs.md) for the TV-specific casting plan.
