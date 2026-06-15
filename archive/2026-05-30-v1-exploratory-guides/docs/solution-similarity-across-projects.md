# Solution Similarity Across Projects

How the recommended stacks overlap, share infrastructure, and trade the same constraints (wireless-only, iPhone + Android, local storage, remote access).

> For control of your **specific existing devices** (wall-switch ceiling lights, 3 IR light zones, Vizio V4K65M-08 + Samsung ARC, Vizio E55-U + Chromecast 4K), see the companion doc [device-control-similarity.md](./device-control-similarity.md), which groups them into three reusable control "rails."

---

## Shared building blocks

Several solutions reuse the same core components. Picking one stack early reduces duplicate ops.

| Building block | Used in | Role |
|----------------|---------|------|
| **Tailscale / WireGuard mesh** | Media, security, screen share (remote), HA | Remote access without port-forward; cross-state as one logical LAN |
| **N100 / mini PC home server** | Media (Immich), security (Frigate), optional restream | Single always-on x86 box; Docker or Proxmox |
| **Home Assistant** | Screen share orchestration, security alerts, lights/TV/audio, commercial experiments | Automation hub; `media_player` for Cast/Vizio |
| **Google Cast ecosystem** | Screen share, HA TV control | Chromecast built-in (V4K65M-08), Chromecast 4K, Vizio SmartCast |
| **IR layer (Broadlink/ESPHome)** | HA audio + IR lights | One blaster covers Samsung ARC system **and** the 3 IR light zones |
| **Smart on/off switch (Caseta/Shelly)** | HA lighting | Ceiling lights on/off with physical button preserved |
| **Immich** | Media server | Native iOS + Android photo apps; background upload |
| **Frigate (or lightweight NVR)** | Security | 2–3 cameras, local disk retention |
| **MQTT** | HA, optional RFID/scenes | Lightweight event bus for wireless devices |
| **Zigbee / Wi‑Fi IoT** | HA hobby | Switches, plugs, optional LED-controller swap—no new cable runs |
| **ESP / ESPHome rail** | IR lights, unified remote, RFID readers | One toolchain (ESPHome + MQTT + HA API) for all DIY hardware |
| **HA Assist (voice)** | Unified remote, optional voice puck | Server-side STT/intents; push‑to‑talk from the deck |

---

## Similarity clusters

Solutions that solve different *problems* but share the same *architecture*.

### Cluster 1 — “Tailscale + self-hosted services”

| Project | Primary service | Remote client |
|---------|-----------------|---------------|
| Media | Immich (+ optional Nextcloud) | Immich app, Nextcloud app |
| Security | Frigate + go2rtc or Reolink local | HA app, Frigate UI, Reolink app |
| Screen share (remote) | Cast at home via VPN*, or SyncPlay/Jellyfin | Phone/laptop on mesh |

\*Cast discovery over VPN often needs an mDNS repeater (e.g. Avahi reflector on router, or **tailscale serve** patterns). Same pain point as reaching Immich without public exposure—solved once for all services.

**Similarity:** One VPN/mesh config unlocks remote photo library, camera playback, and (with tuning) home Cast targets.

---

### Cluster 2 — “Home Assistant as AV conductor (wireless)”

| Project | HA role |
|---------|---------|
| Screen share | Script: play same YouTube URL on both TVs' `media_player` (Cast) |
| TV / audio hobby | Macros: TV on + correct input + Samsung audio on (IR); group Cast speakers |
| Commercial stretch | Bedroom Chromecast: SponsorBlock/ADB skip; living room: IR mute on Samsung |
| Security | Person detected → pause TV or flash lights (optional) |

**Similarity:** Both TVs are **Cast** targets (`media_player`)—no HDMI wiring. The only asymmetry is **living-room volume/mute, which rides the IR layer to the Samsung system**, not the Cast volume.

---

### Cluster 3 — “Phone-first mobile apps (iOS + Android)”

| Project | Best native-app fit |
|---------|---------------------|
| Media | **Immich** (both platforms); Synology Photos (both, weaker AI) |
| Security | Reolink app (both); HA Companion (both); Frigate via HA or web |
| Screen share | YouTube app + Cast; Google Home app; Vizio Mobile |
| HA hobby | HA Companion (both); vendor apps (Hue, Sonos) as backup |

**Similarity:** Prefer stacks with **first-party or Immich-level** mobile apps rather than web-only admin UIs for daily use.

---

### Cluster 4 — “Local disk, no cloud subscription”

| Project | Local storage |
|---------|---------------|
| Media | ZFS/NAS dataset for Immich library |
| Security | NVR recordings on same pool or NVR disk; 2–3 cams ≈ 60–180 GB/mo |
| Screen share | Usually none (streaming); optional Jellyfin for owned files on same NAS |

**Similarity:** One NAS pool with datasets `photos/`, `security/recordings/` simplifies backup (restic/kopia once).

---

## Cross-project matrix

| Capability | Media | Security | Screen share | HA hobby |
|------------|-------|----------|--------------|----------|
| Tailscale remote | ● | ● | ● | ● |
| Home Assistant | ○ | ● | ● | ● |
| Google Cast / Vizio | ○ | ○ | ● | ● |
| Immich | ● | ○ | ○ | ○ |
| Frigate / local NVR | ○ | ● | ○ | ● |
| Zigbee / Wi‑Fi lights | ○ | ○ | ○ | ● |
| Jellyfin | ○ | ○ | ● | ○ |
| Native iOS + Android app | ● | ● | ● | ● |

● = primary fit · ○ = optional / supporting

---

## Recommended unified stack (high similarity)

These choices maximize reuse for your constraints:

```
                    ┌─────────────────────────────────┐
                    │  Tailscale (phones, laptops,    │
                    │  server, optional HA)           │
                    └───────────────┬─────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        ▼                           ▼                           ▼
 ┌──────────────┐           ┌──────────────┐           ┌──────────────┐
 │ Immich       │           │ Frigate      │           │ Home         │
 │ (photos)     │           │ 2–3 RTSP cams│           │ Assistant    │
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
| Remote | Tailscale | Same URL pattern for Immich, HA, Frigate |
| Photos | Immich | Best dual-native photo apps |
| Cameras | 2–3× Wi‑Fi or PoE RTSP → Frigate | Local retention; HA alerts |
| AV | HA + Cast + Vizio SmartCast | Multi-TV YouTube; no cables |
| Lights/audio | Zigbee (IKEA/Aqara) + Sonos or Cast groups | Wireless; HA scenes |

**Rough combined hardware (excluding TVs you own):** $500–900 server/storage + $120–250 HA box + $150–360 cameras + $50–150 smart bulbs/plugs.

---

## Where solutions diverge (not interchangeable)

| Need | Do not reuse from other project | Purpose-built instead |
|------|--------------------------------|------------------------|
| Mirror Netflix from one TV to another | Media/Jellyfin | No reliable wireless HDMI mirror; use same app on each TV or Jellyfin for owned rips |
| Frame-perfect sync on 3 TVs | Security NVR timing | Cast/HA scripts; accept 1–5 s drift |
| Commercial skip on streaming apps | Frigate motion AI | Ad detection needs audio/TV automation stack (see HA guide) |
| Google Photos replacement UX | Reolink app | Immich or Synology Photos |

---

## Stretch goals: cross-project paths

| Stretch goal | Projects involved | Similar tooling |
|--------------|-------------------|-----------------|
| Stream from either TV (Chromecast/Vizio) | Screen share + HA | Same Cast entities; `media_player` source selection |
| Stream to any Wi‑Fi device | Screen share | Cast/AirPlay/Jellyfin clients on tablets, phones, TVs |
| Share with device in another state | All | Tailscale + Immich remote upload/view; Jellyfin SyncPlay; YouTube link + HA sync script at home |
| Mute on commercials | HA + screen share | HA `media_player.volume_mute` driven by ad detector (YouTube: SponsorBlock; live TV: Channels DVR) |
| Fast-forward commercials | HA + screen share | Practical for OTA/cable (Channels DVR); streaming apps mostly mute-only |

---

## Decision shortcuts

| If you prioritize… | Favor | Deprioritize |
|--------------------|-------|--------------|
| Best photo apps remote | Immich + Tailscale | Folder-only sync without Immich app |
| Simplest 3-cam local record | Reolink Wi‑Fi + SD or small NVR | UniFi, 8-channel kits |
| Wireless multi-TV YouTube | HA + multi Cast + Vizio SmartCast | HDMI matrix/splitter |
| One automation hub | HAOS + Zigbee | Standalone vendor hubs only |
| Minimal boxes | Proxmox: Immich + Frigate + HA VMs | Separate appliance per role |

See individual project guides for full solution lists and costs.
