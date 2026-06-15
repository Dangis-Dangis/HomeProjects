# Shared Architecture

The six capability areas solve different problems but share a small set of building blocks. Committing to one stack early eliminates duplicate operations. This document covers (1) the shared building blocks, (2) the three device-control "rails" that every existing device maps onto, (3) the similarity clusters, and (4) the unified reference architecture.

---

## 1. Shared building blocks

| Building block | Used in | Role |
|----------------|---------|------|
| **Mesh VPN (Tailscale / WireGuard)** | Media, security, screen share (remote), control | Remote access without port-forwarding; treats devices across locations as one logical LAN |
| **N100 / mini-PC home server** | Media (Immich), security (Frigate), optional restream | A single always-on x86 box; Docker or Proxmox |
| **Home Assistant (automation hub)** | Screen-share orchestration, security alerts, lights/TV/audio, commercial experiments | Automation hub; `media_player` for Cast/Vizio |
| **Google Cast ecosystem** | Screen share, TV control | Chromecast built-in (living-room TV), Chromecast 4K (bedroom), Vizio SmartCast |
| **IR layer (Broadlink / ESPHome)** | Audio + IR lights | One blaster covers the Samsung sound system **and** the 3 IR light zones |
| **Smart on/off switch (Caseta / Shelly)** | Lighting | Ceiling lights on/off with the physical button preserved |
| **Immich** | Media server | Native iOS + Android photo apps; background upload |
| **Frigate (or lightweight NVR)** | Security | 2–3 cameras, local disk retention |
| **MQTT** | Hub, optional RFID/scenes | Lightweight event bus for wireless devices |
| **Zigbee / Wi‑Fi IoT** | Hub | Switches, plugs, optional LED-controller swap — no new cable runs |
| **ESP / ESPHome rail** | IR lights, unified remote, RFID readers | One toolchain (ESPHome + MQTT + hub API) for all DIY hardware |
| **Voice assistant (HA Assist)** | Unified remote, optional voice puck | Server-side speech-to-text/intents; push-to-talk from the deck |

---

## 2. The three device-control "rails"

Every existing device reduces to one of three integration rails. The design picks the rail, not the device — which is why one solution often covers several devices at once.

### Rail 1 — IR (line-of-sight, open-loop)

| Device on this rail | Why |
|---------------------|-----|
| Samsung sound system (living room) | IR-only; no reliable IP/CEC (hence the two-remote situation today) |
| 3 IR light controllers | IR-only, no physical buttons |
| Vizio TVs (power/volume fallback) | IR works even when IP control is flaky |

**One solution covers all of them:** a single IR strategy — a **Broadlink RM4** (Wi‑Fi→IR) or an **ESPHome IR blaster** (ESP32 + IR LED) — learns every remote and exposes it to the hub. This is the biggest single similarity in the setup: **the Samsung sound-bar problem and the "lights with no buttons" problem are the same problem.**

- **Shared limitation:** IR is **open-loop** — no state feedback. The hub "thinks" a light is on but cannot confirm it. Mitigate with assumed-state + occasional resync, or move that device to Rail 3.
- **Shared requirement:** line-of-sight, one blaster/emitter per room or zone. Budget 1 blaster for the living room (TV + Samsung) and 1–3 for the IR light zones depending on room spread.

### Rail 2 — IP / Cast (stateful, two-way)

| Device on this rail | Integration | What it provides |
|---------------------|-------------|------------------|
| Vizio V4K65M-08 (living room) | Hub `vizio` (SmartCast API) | Power, volume*, input, app launch, `play_media` |
| Chromecast 4K (bedroom) | Hub `cast` / `androidtv` | Play/pause, volume, YouTube URLs, app launch, ADB |
| Roku (unused) | Hub `roku` | Optional backup target only |

\***Living-room volume caveat:** the Vizio integration controls the *TV's* volume, but the sound comes from the **Samsung system over ARC**. Unless CEC volume pass-through works (unreliable on old gear — the two-remote situation confirms it does not), **volume/mute must travel Rail 1 (IR to the Samsung system)**, not Rail 2.

### Rail 3 — Mains switching / smart relay (stateful, physical button preserved)

| Device on this rail | Solution | Keeps physical control? |
|---------------------|----------|-------------------------|
| Ceiling lights | Smart **on/off** switch or relay behind the switch | Yes (hard requirement) |

Because the bulbs are **non-dimmable and fixtures are fixed**, this rail is **on/off only** — no dimmers, no smart bulbs (a wall switch cutting power would orphan smart bulbs). Every option keeps the wall as a working switch.

### Rail map

```
        ┌────────────────────── Automation hub ───────────────────────┐
        │                                                              │
   Rail 1: IR                Rail 2: IP/Cast              Rail 3: Mains
   (Broadlink/ESPHome)       (media_player)               (smart switch/relay)
        │                          │                            │
   Samsung sound         Vizio SmartCast (LR)            Ceiling lights
   3 IR light zones      Chromecast 4K (BR)              (on/off, button kept)
   TV IR fallback        Roku (backup)
```

**Reuse insight:** a single IR layer (Rail 1) simultaneously solves **audio control**, the **IR-only lights**, and **TV power fallback**. A single `media_player` layer (Rail 2) solves **both TVs + screen share**. A single smart-switch standard (Rail 3) solves **all ceiling lights**.

### Cross-device control matrix

| Capability | Ceiling lights | IR lights | Living-room TV+audio | Bedroom TV |
|------------|:--------------:|:---------:|:--------------------:|:----------:|
| Rail 1 IR (Broadlink/ESPHome) | ○ | ● | ● (audio + power) | ○ (fallback) |
| Rail 2 IP/Cast | ✗ | ✗ | ● (TV/apps) | ● (Chromecast) |
| Rail 3 smart switch | ● | ✗ | ✗ | ✗ |
| State feedback | ● (Rail 3) | ✗ (IR) | partial | ● (Cast/ADB) |
| Physical button preserved | ● | n/a | n/a | n/a |

● primary · ○ secondary/fallback · ✗ not applicable

### Where the rooms diverge

| Concern | Living room (V4K65M-08 + Samsung) | Bedroom (E55-U + Chromecast 4K) |
|---------|-----------------------------------|----------------------------------|
| Primary player | Vizio SmartCast apps | Chromecast with Google TV |
| Volume/mute | **IR → Samsung system** | Cast/CEC to TV or soundbar |
| App launch in hub | `vizio` (model-dependent) | `cast`/`androidtv` (full) |
| Commercial **mute** | Audio detector → IR mute Samsung | ADB / SmartTubeNext |
| Commercial **skip** | Not practical (SmartCast apps) | **Feasible** (Google TV + SponsorBlock/ADB) |
| Best "watch" macro | TV on + Samsung on + input (Broadlink) | Cast wake + soundbar via CEC |

**Takeaway:** the **bedroom Chromecast 4K is the experimentation TV** (commercial skip, ADB, SmartTubeNext). The **living room is a "consolidate two remotes into one hub macro"** problem driven by IR.

---

## 3. Similarity clusters

Solutions that solve different *problems* but share the same *architecture*.

### Cluster 1 — "Mesh VPN + self-hosted services"

| Capability | Primary service | Remote client |
|------------|-----------------|---------------|
| Media | Immich (+ optional Nextcloud) | Immich app, Nextcloud app |
| Security | Frigate + go2rtc, or Reolink local | Hub app, Frigate UI, Reolink app |
| Screen share (remote) | Cast at home via VPN*, or SyncPlay/Jellyfin | Phone/laptop on the mesh |

\*Cast discovery over VPN often needs an mDNS repeater (an Avahi reflector on the router, or `tailscale serve` patterns). This is the same pain point as reaching the photo library without public exposure — solved once for all services.

**Similarity:** one VPN/mesh config unlocks the remote photo library, camera playback, and (with tuning) home Cast targets.

### Cluster 2 — "Automation hub as AV conductor (wireless)"

| Capability | Hub role |
|------------|----------|
| Screen share | Script: play the same YouTube URL on both TVs' `media_player` (Cast) |
| TV / audio | Macros: TV on + correct input + Samsung audio on (IR); group Cast speakers |
| Commercial stretch | Bedroom Chromecast: SponsorBlock/ADB skip; living room: IR mute on Samsung |
| Security | Person detected → pause TV or flash lights (optional) |

**Similarity:** both TVs are **Cast** targets — no HDMI wiring. The only asymmetry is **living-room volume/mute, which rides the IR layer to the Samsung system**, not the Cast volume.

### Cluster 3 — "Phone-first mobile apps (iOS + Android)"

| Capability | Best native-app fit |
|------------|---------------------|
| Media | **Immich** (both platforms); Synology Photos (both, weaker AI) |
| Security | Reolink app (both); hub companion (both); Frigate via hub or web |
| Screen share | YouTube app + Cast; Google Home app; Vizio Mobile |
| Control | Hub companion (both); vendor apps (Hue, Sonos) as backup |

**Similarity:** prefer stacks with **first-party or Immich-level** mobile apps over web-only admin UIs for daily use.

### Cluster 4 — "Local disk, no cloud subscription"

| Capability | Local storage |
|------------|---------------|
| Media | ZFS/NAS dataset for the Immich library |
| Security | NVR recordings on the same pool; 2–3 cams ≈ 60–180 GB/mo |
| Screen share | Usually none (streaming); optional Jellyfin for owned files on the same NAS |

**Similarity:** one NAS pool with datasets `photos/` and `security/recordings/` simplifies backup (one restic/kopia job).

### Capability × building-block matrix

| Capability | Media | Security | Screen share | Control |
|------------|:-----:|:--------:|:------------:|:-------:|
| Mesh VPN remote | ● | ● | ● | ● |
| Automation hub | ○ | ● | ● | ● |
| Google Cast / Vizio | ○ | ○ | ● | ● |
| Immich | ● | ○ | ○ | ○ |
| Frigate / local NVR | ○ | ● | ○ | ● |
| Zigbee / Wi‑Fi lights | ○ | ○ | ○ | ● |
| Jellyfin | ○ | ○ | ● | ○ |
| Native iOS + Android app | ● | ● | ● | ● |

● primary fit · ○ optional / supporting

---

## 4. Unified reference architecture

These choices maximize reuse for the stated constraints.

```
                    ┌─────────────────────────────────┐
                    │  Mesh VPN (phones, laptops,     │
                    │  server, optional hub)          │
                    └───────────────┬─────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        ▼                           ▼                           ▼
 ┌──────────────┐           ┌──────────────┐           ┌──────────────┐
 │ Immich       │           │ Frigate      │           │ Automation   │
 │ (photos)     │           │ 2–3 RTSP cams│           │ hub          │
 └──────────────┘           └──────────────┘           └──────┬───────┘
        │                           │                          │
        └───────────────┬───────────┘                          │
                        ▼                                      │
                 TrueNAS / N100                                │
                 (single server)                               │
                        │                                      │
                        └──────────► Cast / Vizio / Hue / Sonos
                                   (wireless only)
```

| Layer | Pick | Why it overlaps |
|-------|------|-----------------|
| Server | N100 16 GB + 2–4 TB | Runs Immich + Frigate for 2–3 cams comfortably |
| Remote | Mesh VPN | Same URL pattern for Immich, hub, and Frigate |
| Photos | Immich | Best dual-native photo apps |
| Cameras | 2–3× Wi‑Fi or PoE RTSP → Frigate | Local retention; hub alerts |
| AV | Hub + Cast + Vizio SmartCast | Multi-TV YouTube; no cables |
| Lights/audio | Zigbee (IKEA/Aqara) + Sonos or Cast groups | Wireless; hub scenes |

**Rough combined hardware (excluding owned TVs):** $500–900 server/storage + $120–250 hub box + $150–360 cameras + $50–150 smart switches/plugs.

### Where solutions intentionally diverge (not interchangeable)

| Need | Do not reuse from another capability | Purpose-built instead |
|------|--------------------------------------|------------------------|
| Mirror one TV's app to another | Media/Jellyfin | No reliable wireless HDMI mirror; use the same app on each TV, or Jellyfin for owned files |
| Frame-perfect sync on multiple TVs | Security NVR timing | Cast/hub scripts; accept 1–5 s drift |
| Commercial skip on streaming apps | Frigate motion AI | Ad handling needs the audio/TV automation stack |
| Cloud-gallery replacement UX | Reolink app | Immich or Synology Photos |

### Decision shortcuts

| If you prioritize… | Favor | Deprioritize |
|--------------------|-------|--------------|
| Best remote photo apps | Immich + mesh VPN | Folder-only sync without an app |
| Simplest 3-cam local record | Reolink Wi‑Fi + SD or small NVR | UniFi, 8-channel kits |
| Wireless multi-TV YouTube | Hub + multi-Cast + Vizio SmartCast | HDMI matrix/splitter |
| One automation hub | HAOS + Zigbee | Standalone vendor hubs only |
| Minimal boxes | Proxmox: Immich + Frigate + hub VMs | Separate appliance per role |

Each solution collection that follows lists the full set of options, costs, and trade-offs for its capability area.
