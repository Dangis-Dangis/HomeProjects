# Connected Home Systems — Proposal

A complete, self-contained design proposal for a privacy-respecting, wireless-first connected home built on open, self-hosted components. This document set frames the household's needs and constraints, breaks the work into discrete problem statements, and presents a fully researched collection of solutions for each — with costs, trade-offs, wiring, and reference configurations.

The proposal is organized so that one shared backbone (a small home server, a local automation hub, a private mesh network, and a single DIY hardware toolchain) serves six capability areas. Building the backbone once unlocks every capability with minimal duplicate effort.

---

## How to read this proposal

| Document | Purpose |
|----------|---------|
| [00-needs-and-constraints.md](./00-needs-and-constraints.md) | The household's goals, the existing device inventory, and the hard constraints every solution must respect. |
| [01-problem-statements.md](./01-problem-statements.md) | The work decomposed into six discrete, independently buildable problem statements. |
| [02-shared-architecture.md](./02-shared-architecture.md) | The shared backbone, the three device-control "rails," similarity clusters, and the unified reference architecture. |
| [solutions-01-media-storage.md](./solutions-01-media-storage.md) | Photos/video/files: remote-accessible self-hosted storage with native mobile apps. |
| [solutions-02-security-recording.md](./solutions-02-security-recording.md) | 2–3 cameras with local recording and remote viewing. |
| [solutions-03-home-control.md](./solutions-03-home-control.md) | Lights (mains + IR), TVs, and audio under one automation hub. |
| [solutions-04-screen-share.md](./solutions-04-screen-share.md) | Wireless multi-TV / multi-device casting, including cross-state watch-together. |
| [solutions-05-unified-remote.md](./solutions-05-unified-remote.md) | A battery-powered DIY control deck (dials/buttons/LEDs/voice) plus six build concepts. |
| [solutions-06-rfid-scenes.md](./solutions-06-rfid-scenes.md) | Tap-to-trigger NFC/RFID scenes (convenience, not access control). |

**Conventions used throughout:** Costs are in USD at rough current retail and exclude labor and equipment the household already owns (TVs, phones, PCs). Each solution collection lists every viable option with its pros, cons, and technical fit, then names a recommended choice.

**Each solution file ends with a full dossier for the recommended build**, covering: a concept image, an itemized parts list, a worked cost example, the data path, a general wiring/topology diagram, the effort to create, the maintenance cost (money and time), cost feasibility & scalability, and the potential risks & downsides.

---

## Scope at a glance

| # | Capability | Recommended approach | Cost (recommended) | Setup | Feasibility | Scalability |
|---|------------|----------------------|--------------------|-------|-------------|-------------|
| 1 | Media & photo storage | Self-hosted Immich on an N100/TrueNAS server + mesh VPN | $500–1,100 | 1–2 days | ★★★★★ | ★★★★★ |
| 2 | Security recording | Frigate on the same server + 2–3 Wi‑Fi RTSP cameras | $150–350 (+server) | 0.5–1 day | ★★★★★ | ★★★★ |
| 3 | Lights / TV / audio | Automation hub + Lutron Caseta + Broadlink IR + Cast | $350–700 | 1–3 days | ★★★★★ | ★★★★★ |
| 4 | Wireless screen share | Automation hub + Google Cast on both TVs | $35–285 | 2–4 hrs | ★★★★★ | ★★★★ |
| 5 | Unified DIY remote | ESP32‑S3 + ESPHome custom deck | $45–95 | 2–4 days | ★★★★★ | ★★★★★ |
| 6 | RFID / NFC scenes | Phone NFC first, add ESP readers as desired | $0–60 | 1–4 hrs | ★★★★★ | ★★★★ |

**Whole-portfolio hardware (built fresh, excluding TVs/phones):** roughly **$1,100–2,600**, depending on storage capacity, camera count, and how elaborate the remote and lighting become.

---

## The shared backbone (build once, reused everywhere)

```
            ┌──────────── Mesh VPN (all phones, laptops, server) ─────────────┐
            │                                                                  │
   ┌────────┴─────────┐         ┌───────────────────┐         ┌───────────────┴───────┐
   │  N100 server      │◄───────│  Automation hub    │◄────────│  ESPHome / IR rail    │
   │  Immich + Frigate │         │  (hub + voice)     │         │  Broadlink, ESP32     │
   └───────────────────┘         └─────────┬─────────┘         │  deck, mic, sensors   │
                                            │                   └───────────────────────┘
                                            ▼
            Cast (living-room + bedroom TVs) · smart switches · Cast/Sonos audio groups
```

| Backbone piece | Capabilities that reuse it | Build-once payoff |
|----------------|----------------------------|-------------------|
| **Mesh VPN (Tailscale/WireGuard)** | 1, 2, 3, 4, 5 | One remote-access config for photos, cameras, hub, and remote casting |
| **N100 / TrueNAS server** | 1, 2 | Photos and camera NVR share CPU/disk; a single backup job covers both |
| **Automation hub (Home Assistant)** | 2, 3, 4, 5, 6 | The hub every control and automation talks to |
| **ESP / ESPHome rail** | 3 (IR lights), 5 (remote), 6 (readers) | One toolchain — ESPHome + MQTT + hub API — for all DIY hardware |
| **Google Cast** | 3, 4 | Both TVs are Cast targets; one casting pattern serves control and screen share |
| **IR layer (Broadlink)** | 3 (audio + IR lights), 4 (volume), 5 (deck IR macros) | One blaster, many consumers |

**Biggest reuse wins:** (a) one **server** does media + security; (b) one **hub + ESPHome + IR** toolchain does lights, TV/audio, the remote, and RFID; (c) one **mesh VPN** makes all of it securely remote.

---

## Recommended build order (dependency-aware)

1. **Automation hub + mesh VPN** — everything else plugs into it. *(1–2 days)*
2. **N100 server + Immich (photos)** — highest daily value; the box also hosts the camera NVR later. *(1 day)*
3. **Lighting + IR + TV macros** — smart switches, the IR blaster for audio/IR-lights, Cast for the TVs. *(spread over a week)*
4. **Security (Frigate)** — adds onto the existing server; 2–3 Wi‑Fi cameras. *(half day)*
5. **Screen share** — mostly configuration once Cast + hub exist. *(an afternoon)*
6. **Unified remote** — the capstone; prototype on an off-the-shelf module, then build the custom deck once the macros it triggers are stable. *(2–4 days)*
7. **RFID scenes** — optional convenience; phone NFC first, ESP readers if desired. *(an hour)*

> Rationale: the remote (6) is most satisfying **after** the automation scenes/macros it fires (3, 4) already exist, so each physical control maps to something real.

---

## Cross-capability similarity matrix

| Building block | Media | Security | Control | Screen share | Remote | RFID |
|----------------|:-----:|:--------:|:-------:|:------------:|:------:|:----:|
| Mesh VPN | ● | ● | ● | ● | ● | ○ |
| Automation hub | ○ | ● | ● | ● | ● | ● |
| N100 / TrueNAS server | ● | ● | ○ | ○ | ✗ | ✗ |
| ESPHome / ESP32 | ✗ | ✗ | ● | ✗ | ● | ● |
| IR (Broadlink) | ✗ | ✗ | ● | ● | ● | ✗ |
| Google Cast | ✗ | ✗ | ● | ● | ○ | ✗ |
| Voice assistant | ✗ | ✗ | ● | ○ | ● | ✗ |

● primary · ○ supporting · ✗ not applicable

---

## Operating assumptions

- The household runs its own network and can deploy a mesh VPN and/or VLANs.
- Comfortable with Docker, Linux, consumer smart-home gear, basic electronics, custom wiring, a 3D printer, and a label maker.
- Local-first and subscription-free are preferred; cloud is used only for optional offsite backup.
