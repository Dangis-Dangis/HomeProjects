# Needs & Constraints

This document frames everything that follows. It captures the household's goals, the equipment already in place, and the non-negotiable constraints that every proposed solution must respect. The problem statements in [01-problem-statements.md](./01-problem-statements.md) and the solution collections all trace back to the requirements stated here.

---

## 1. Household goals

The household wants a **privacy-respecting, locally controlled, wireless-first** connected home covering six capability areas:

1. **Personal media & photo storage** — a private replacement for cloud photo services, accessible remotely from both iPhone and Android with native photo-app experiences.
2. **Home security recording** — a small camera fleet (2–3 cameras) recording to local storage, viewable by multiple household members.
3. **Lighting, TV, and audio control** — unified, automatable control of existing lights, televisions, and audio under one hub.
4. **Wireless screen sharing** — casting the same content to multiple TVs and devices without cabling, including watch-together with family in another location.
5. **A unified physical remote** — a battery-powered, handheld DIY control deck for the most common actions, with tactile controls, status feedback, and voice.
6. **RFID/NFC scene triggers** — tap-to-run convenience scenes (explicitly **not** door locks or access control).

---

## 2. Existing device inventory

Solutions are designed around the equipment already owned. The household prefers to integrate this hardware rather than replace it.

| Device | Location | How it is controlled today | Audio path |
|--------|----------|----------------------------|------------|
| Ceiling lights | Multiple rooms | **Wall switch only**; non-dimmable bulbs; fixtures are fixed | — |
| Accent lights (×3 zones) | Multiple | **IR remote only, no physical buttons** (3 separate IR controllers) | — |
| Vizio **V4K65M-08** 65" 4K | Living room | Vizio SmartCast built-in apps + Chromecast built-in + AirPlay 2; sound through an **old Samsung sound system over ARC, controlled by IR (≈2 remotes)**; an old Roku is present but unused | Samsung system (ARC + IR) |
| Vizio **E55-U** 55" 4K | Bedroom (above a PC) | **Chromecast 4K** is the primary source; the TV's own SmartCast is early/limited on this model | TV speakers, or an ARC soundbar |
| iPhone(s) + Android phone(s) | — | Daily-use clients for photos and control | — |

---

## 3. Hard constraints

These apply across all six capability areas.

| Constraint | What it means for the design |
|------------|------------------------------|
| **Wireless-first** | No new cable runs between rooms, apartments, or homes. Prefer Wi‑Fi, Zigbee, IR blasters, Cast, and smart-TV APIs. A single in-room cable to a switch or AP is acceptable; cross-room pulls are not. |
| **iPhone + Android parity** | Daily-use apps (photos, control, camera viewing) must have first-class native apps on **both** platforms, not web-only admin panels. |
| **Local storage / no cloud lock-in** | Photos and camera recordings live on local disk with no mandatory subscription. Cloud is acceptable only as optional encrypted offsite backup. |
| **Preserve physical light control** | Ceiling-light wall switches must keep working as physical switches even if the hub is down. Replacing a switch is fine **only** if the paddle/toggle still cuts the load. |
| **Non-dimmable ceiling fixtures** | Ceiling lights are on/off only — no dimmers and no smart bulbs (a switched-off circuit would orphan smart bulbs). |
| **Honor existing AV gear** | The two Vizio TVs, the Samsung IR sound system, the Chromecast 4K, and the 3 IR light controllers are integrated as-is. |
| **Multi-user** | Multiple household members (adults and kids) use photos, cameras, and control with appropriate access levels. |
| **Remote access** | Photos, cameras, and control must be reachable away from home without exposing services directly to the public internet. |

---

## 4. Capabilities the household has available

These shape which solutions are realistic and where DIY is preferred over off-the-shelf.

- Runs its own network; can deploy a **mesh VPN** (Tailscale/WireGuard) and/or **VLANs**.
- Comfortable with **Docker, Linux**, and consumer smart-home gear.
- Has **electronics skills, custom wiring**, a **3D printer**, and a **label maker** — DIY hardware (ESP32/ESPHome) is a welcome path, not a barrier.

---

## 5. Stretch goals

Desirable but secondary; each is addressed where feasible in the relevant solution collection.

| Stretch goal | Where addressed |
|--------------|-----------------|
| Detect commercials → **mute** | [solutions-03-home-control.md](./solutions-03-home-control.md) |
| Detect commercials → **fast-forward/skip** where feasible | [solutions-03-home-control.md](./solutions-03-home-control.md) |
| Start playback from **either TV** | [solutions-04-screen-share.md](./solutions-04-screen-share.md) |
| Cast to **any device on Wi‑Fi** (TVs, phones, tablets) | [solutions-04-screen-share.md](./solutions-04-screen-share.md) |
| **Watch-together with family in another state** | [solutions-04-screen-share.md](./solutions-04-screen-share.md) |
| **Voice input** on the handheld remote | [solutions-05-unified-remote.md](./solutions-05-unified-remote.md) |
| Optional **encrypted offsite backup** | [solutions-01-media-storage.md](./solutions-01-media-storage.md) |

---

## 6. Explicit non-goals

Stating these prevents over-building.

- **No door locks / electric strikes / access control.** RFID is for convenience scenes only.
- **No wireless HDMI matrix / bit-perfect mirroring** of arbitrary DRM streaming across screens — this is not reliably possible; screen share uses Cast/API orchestration instead.
- **No frame-locked multi-TV sync** — a 1–5 second drift between rooms is acceptable.
- **No public internet exposure** of self-hosted services — remote access is via the mesh VPN.

These needs and constraints recur across every solution collection. The shared building blocks that satisfy them are detailed in [02-shared-architecture.md](./02-shared-architecture.md).
