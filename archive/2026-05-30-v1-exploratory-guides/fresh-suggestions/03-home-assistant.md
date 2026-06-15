# 03 — Home Assistant: Lights, TV, Audio (Fresh Suggestion)

**Need:** Wireless control of **ceiling lights** (wall-switch, non-dimmable, keep physical control), **3 IR light zones**, **living-room Vizio V4K65M-08 + Samsung ARC (IR)**, **bedroom E55-U + Chromecast 4K**; commercial mute/skip stretch.

Full option list: [../home-assistant-integration.md](../home-assistant-integration.md) · Control rails: [../device-control-similarity.md](../device-control-similarity.md)

---

## Top pick — HAOS + Lutron Caseta (ceiling) + Broadlink (IR) + Cast (TVs)

One hub, three control rails, each matched to the hardware you already have.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$350–700**: HA box $120–250; Caseta hub $80 + on/off switches $60 ea (or Shelly Plus 1 $25 behind existing switches); Broadlink RM4 $35; Zigbee coordinator $30; optional Zigbee/WLED swap for IR strips $20–30/zone. |
| **Setup time** | **1–3 days** (HA + Zigbee, switch wiring, learn IR remotes, build TV/audio macros). |
| **Maintenance** | **Med** — monthly HA updates; re-test `vizio`/Cast after upgrades; IR is open-loop so occasional resync. |
| **Feasibility** | ★★★★★ — mains wiring + IR learning are easy for you. |
| **Scalability** | ★★★★★ — add switches/zones/scenes indefinitely; rails generalize. |

**The three rails (reused across the portfolio):**
1. **Mains switch (Caseta/Shelly)** → ceiling lights, on/off, physical paddle preserved.
2. **IR (Broadlink/ESPHome)** → Samsung audio **and** the 3 IR light zones (same blaster) + LR volume/mute.
3. **IP/Cast** → V4K65M-08 via `vizio`, Chromecast 4K via `cast`/`androidtv`.

**Commercial stretch:** bedroom Chromecast 4K = test bed (SmartTubeNext + SponsorBlock, ADB skip); living room = **audio-detector → Broadlink mute** on the Samsung system (mute only, not skip).

**Project similarity:** This is the **hub** for Security alerts, Screen share, the Unified remote, and RFID scenes. The **IR rail** is shared by Screen share (LR volume) and the Remote (can fire IR macros). **Tailscale** + HA Companion give remote control on both phone OSes.

---

## Alternate A — HA Container on the N100/Proxmox

Folds HA onto the media/security server (no separate box, **$0 incremental**). **Maintenance Med–High** (no Supervisor add-ons; run Mosquitto/Zigbee2MQTT as compose). Best if minimizing devices; slightly more ops.

## Alternate B — Shelly-everywhere (Wi‑Fi-first, no Zigbee)

Shelly relays behind every switch + Broadlink for IR; skip a Zigbee coordinator. **Cost similar; Feasibility ★★★★★; Scalability ★★★★** (Wi‑Fi device count on one AP is the limit). Good if you prefer Wi‑Fi/HTTP over Zigbee mesh.

---

## Verdict

Build **HAOS first** (backbone #1). Do **Caseta** switches for the cleanest no-neutral install that keeps physical control, or **Shelly** if you want to keep existing faceplates and have neutrals. One **Broadlink** solves Samsung audio + IR lights at once. Treat the **bedroom Chromecast** as the commercial-skip playground; accept **mute-only** in the living room.
