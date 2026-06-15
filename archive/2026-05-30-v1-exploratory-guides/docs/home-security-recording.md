# Home Security Recording (2–3 Cameras, Local Storage)

**Scope:** **Two or three cameras** with **local recording**—no cloud-only dependency. Multi-user remote viewing is a plus but the fleet stays small.

**Typical load:** 2–3× 1080p H.265 ≈ **2–6 GB/day** each with motion-heavy scenes; **30-day retention** ≈ **120–540 GB** total.

---

## Solution A — Frigate on home server + Wi‑Fi or PoE RTSP cameras

| | |
|---|---|
| **What it is** | Lightweight NVR with optional object detection; stores to local disk/NAS; exposes RTSP/WebRTC via go2rtc. |
| **Cameras** | 2–3× Reolink E1 Pro (Wi‑Fi), Amcrest Wi‑Fi, or single-run PoE to attic (one cable to switch only—cameras can still be “no new cables across rooms” if Wi‑Fi). |
| **Local storage** | Recordings directory on NAS or server SSD/HDD; Frigate retention rules in config. |
| **Multi-user** | Home Assistant dashboards with per-user views; or shared Frigate UI on LAN/VPN. |
| **Remote** | Tailscale to Frigate/HA; Reolink app as backup if cams are Reolink. |
| **Cost** | $0 software; 3× ~$50–90 cams + optional Coral USB $60; server shared with media stack. |

**Capabilities**
- 24/7 or motion-only; zones; person/package filters with Coral.
- Clips + timeline; no subscription.

**Tradeoffs**
| Pros | Cons |
|------|------|
| Scales down perfectly to 2–3 cams | Initial YAML tuning |
| Strong local retention control | Wi‑Fi cams need solid 2.4/5 GHz placement |
| HA integration optional | Frigate UI RBAC is basic |

**Technical fit:** You run or plan a home server (Immich/TrueNAS); want local AI clips.

---

## Solution B — Reolink Wi‑Fi cameras + microSD or Reolink Home Hub

| | |
|---|---|
| **What it is** | Standalone cams record to **on-camera SD** and/or **Reolink Home Hub** (local NVR box, no cloud required). |
| **Local storage** | SD per cam (64–128 GB) or Hub with 2.5" drive. |
| **Multi-user** | Reolink app; shared household login or guest access (model-dependent). |
| **Remote** | Reolink app (P2P optional—disable for LAN/VPN-only if desired). |
| **Cost** | 3× cam $150–270; Home Hub ~$130 + drive. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Fastest path for 2–3 cams | Weak Home Assistant story |
| All wireless to hub | Hub is another box |
| Good iOS/Android apps | AI features vary by model |

**Technical fit:** Record locally with minimal homelab; acceptable Reolink app UX.

---

## Solution C — Blue Iris + RTSP (Windows NVR)

| | |
|---|---|
| **What it is** | Windows app records RTSP streams from 2–3 cams to local disk. |
| **Local storage** | Dedicated folder on HDD; BI retention policies. |
| **Multi-user** | BI mobile apps; multiple login profiles. |
| **Cost** | ~$70 license + existing Windows PC. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Polished playback UI | Windows always-on |
| Easy for 2–3 streams | Overkill license vs Frigate for tiny fleet |

**Technical fit:** Prefer Windows GUI over Docker.

---

## Solution D — Scrypted + HomeKit Secure Video (iPhone-primary)

| | |
|---|---|
| **What it is** | Bridge RTSP cams into Apple Home; clips in iCloud if subscribed. |
| **Local storage** | **Not fully local** for HKSV history—clips in iCloud; RTSP still available locally via Scrypted. |
| **Multi-user** | Apple Home invites. |
| **Cost** | Free Scrypted + iCloud+ tier. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Best iPhone live view | Android secondary; cloud clip history |
| Fine for 2–3 cams | Conflicts with “maximally local” if HKSV is primary |

**Technical fit:** Apple-heavy household OK with cloud clip storage for short history.

---

## Solution E — Frigate + Home Assistant (integrated alerts)

Same hardware as **A**, with HA as the family-facing layer:

| | |
|---|---|
| **Multi-user** | HA accounts: adults see live + clips; restricted dashboards for guests. |
| **Remote** | HA Companion app + Tailscale; push on person at door. |
| **Cost** | Incremental $0 beyond A + HA box if separate. |

**Technical fit:** Same server as media/HA hobby projects—one Tailscale, one app for cameras + lights.

---

## Camera & storage notes (2–3 cam scale)

| Topic | Guidance |
|-------|----------|
| **Wireless** | Wi‑Fi cams avoid cross-apartment cabling; use 5 GHz if close to AP, wired AP backhaul preferred |
| **Protocols** | ONVIF/RTSP; disable cloud upload in cam settings if local-only |
| **Retention** | 14–30 days on 500 GB–1 TB dedicated space is comfortable |
| **Privacy** | IoT/camera VLAN; block WAN egress on cam subnet |
| **Multi-user** | One admin NVR; family uses HA or Reolink viewer, not shared admin password |

---

## Comparison matrix

| Solution | Local-only | 2–3 cam fit | iOS + Android apps | HA integration | DIY |
|----------|------------|-------------|--------------------|----------------|-----|
| A Frigate | ★★★★★ | ★★★★★ | ★★★★ (via HA/web) | ★★★★★ | Medium |
| B Reolink + Hub | ★★★★★ | ★★★★★ | ★★★★★ | ★★ | Low |
| C Blue Iris | ★★★★★ | ★★★★★ | ★★★★★ | ★★★ | Low–Med |
| D Scrypted/HKSV | ★★★ | ★★★★★ | ★★★★★ iOS | ★★★★ | Medium |
| E Frigate + HA | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | Medium |

---

## Recommended path (local, 2–3 cameras, technical)

1. **Cameras:** 2–3× Reolink or Amcrest **Wi‑Fi**, RTSP enabled, cloud off.  
2. **NVR:** Frigate on shared N100/TrueNAS box; recordings dataset `security/recordings`.  
3. **Optional:** Skip Coral for 2–3 cams—CPU detection is fine at this scale.  
4. **Remote:** Tailscale; family uses **HA Companion** camera cards or Reolink app.  
5. **Backup:** Optional nightly sync of event clips only—not full 24/7 mirror.

**Rough budget:** 3× $70 cam + $500 server (shared with media) ≈ **$710** if starting fresh; **~$210** cams only if server exists.
