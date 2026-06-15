# Constraints & Device Inventory

Binding requirements and the hardware the design integrates — not replaces.

---

## Household goals

Six capability areas, all **local-first**, **wireless-first**, **iPhone + Android** for daily use:

1. **Media & photos** — private library, native mobile apps, remote access
2. **Security** — 2–3 cameras, local recording, multi-user viewing
3. **Home control** — lights, TV, audio under one hub
4. **Screen share** — same content on multiple TVs/devices without cabling
5. **Unified remote** — battery DIY deck with tactile controls + voice
6. **RFID scenes** — tap-to-run convenience (not door locks)

---

## Existing device inventory

| Device | Location | Control today | Audio |
|--------|----------|---------------|-------|
| Ceiling lights | Multiple rooms | Wall switch; **non-dimmable** fixed fixtures | — |
| Accent lights (×3) | Multiple | **IR only**, no buttons (3 IR controllers); **discrete color swatches** (≈8–20 per zone), **5 brightness levels**, programs (USA, Christmas, Rainbow) — not 0–255 RGB | — |
| Vizio **V4K65M-08** 65" | Living room | SmartCast + Chromecast built-in; **Samsung ARC via IR (≈2 remotes)** | Samsung system |
| Vizio **E55-U** 55" | Bedroom (above PC) | **Chromecast 4K** primary | TV or ARC soundbar |
| iPhone(s) + Android | — | Daily clients | — |

---

## Hard constraints

| Constraint | Implication |
|------------|-------------|
| **Wireless-first** | No cross-room cable runs; Wi‑Fi, Zigbee, IR, Cast |
| **iOS + Android parity** | Native apps for photos, cameras, control — not web-only |
| **Local storage** | Photos + recordings on disk; cloud = optional encrypted backup only |
| **Physical light switches** | Ceiling switches must still work if hub is down |
| **Non-dimmable ceilings** | On/off only — no smart bulbs (switch would orphan them) |
| **Honor existing AV & IR RGB** | Integrate Vizios, Samsung IR audio, Chromecast 4K; model RGB as **IR preset indices**, not continuous hue |
| **Multi-user** | Adults + kids; appropriate access levels |
| **Remote access** | Mesh VPN — no public internet exposure of services |

---

## Household capabilities

- Own network; can deploy **Tailscale/WireGuard** and **VLANs**
- Comfortable with **Docker, Linux**, consumer smart-home gear
- **EE skills**, custom wiring, **3D printer**, **label maker** — ESPHome DIY is welcome

---

## Stretch goals

| Goal | Project |
|------|---------|
| Commercial **mute** / **skip** where feasible | 03-home-control |
| Cast from either TV; any Wi‑Fi device; cross-state watch-together | 04-screen-share |
| Voice on handheld remote | 05-unified-remote |
| Encrypted offsite backup | 01-media-storage |

---

## Explicit non-goals

- Door locks / access control (RFID = scenes only)
- Wireless HDMI matrix / DRM mirror to many screens
- Frame-locked multi-TV sync (1–5 s drift OK)
- Public port-forwarding of self-hosted services
