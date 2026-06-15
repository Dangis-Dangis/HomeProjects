# Connected Home Systems

A privacy-respecting, **wireless-first**, self-hosted connected home — six capabilities on one shared backbone.

| Doc | Purpose |
|-----|---------|
| [needs/constraints-and-inventory.md](./needs/constraints-and-inventory.md) | Goals, existing hardware, hard constraints |
| [needs/problems.md](./needs/problems.md) | Six problem statements |
| [architecture/shared-backbone.md](./architecture/shared-backbone.md) | Build-once components reused everywhere |
| [architecture/control-rails.md](./architecture/control-rails.md) | TVs, lights, and audio → IR / Cast / mains |
| [portfolio/build-order-and-costs.md](./portfolio/build-order-and-costs.md) | Dependency order, costs, similarity matrix |

**Contributing / AI:** [docs/](./docs/) · [AGENTS.md](./AGENTS.md)

---

## Projects

| # | Project | Recommended | Parts |
|---|---------|-------------|-------|
| 1 | [Media & photos](./projects/01-media-storage/) | Immich on N100 + Tailscale | $500–1,100 |
| 2 | [Security (2–3 cams)](./projects/02-security/) | Frigate on same server + Wi‑Fi RTSP | $150–350 |
| 3 | [Lights / TV / audio](./projects/03-home-control/) | HA + Caseta + Broadlink + Cast | $350–700 |
| 4 | [Wireless screen share](./projects/04-screen-share/) | HA + Cast on both Vizios | $35–285 |
| 5 | [Unified DIY remote](./projects/05-unified-remote/) | ESP32‑S3 ESPHome deck | $45–90 |
| 6 | [RFID / NFC scenes](./projects/06-rfid-scenes/) | Phone NFC; ESP readers optional | $0–60 |

**Portfolio hardware (fresh build, excl. TVs/phones):** ~**$1,100–2,600**

Each project README is the live spec for that capability. Domain rules for IR RGB and the unified remote: [docs/domain/](./docs/domain/).

---

## Assets

Concept renders and wiring diagrams: [assets/](./assets/) (heroes in `assets/heroes/`).

---

## Archive

Earlier generations (multi-option guides, full proposal dossiers, pre-overhaul remote spec) live under [archive/](./archive/). The repo root is the **current** spec; archive is historical reference only.

| Generation | Date | Contents |
|------------|------|----------|
| [v1 exploratory](./archive/2026-05-30-v1-exploratory-guides/) | 2026-05-30 | Multi-option guides |
| [v2 proposal](./archive/2026-05-31-v2-home-systems-proposal/) | 2026-05-31 | Integrated proposal + dossiers |
| [v3 remote + assets](./archive/2026-06-14-v3-remote-and-assets/) | 2026-06-14 | Remote deep-dive + 33 PNGs |

Index: [archive/README.md](./archive/README.md).
