# Solutions — Home Security Recording

**Problem (P2):** **Two or three cameras** with **local recording** — no cloud-only dependency. Multi-user remote viewing is a plus, but the fleet stays small.

**Typical load:** 2–3× 1080p H.265 ≈ **2–6 GB/day** each in motion-heavy scenes; **30-day retention** ≈ **120–540 GB** total.

This collection lists every viable option with capabilities, trade-offs, and technical fit, then scores the field and names a recommendation.

---

## Solution A — Frigate on the home server + Wi‑Fi or PoE RTSP cameras

| | |
|---|---|
| **What it is** | Lightweight NVR with optional object detection; stores to local disk/NAS; exposes RTSP/WebRTC via go2rtc. |
| **Cameras** | 2–3× Reolink E1 Pro (Wi‑Fi), Amcrest Wi‑Fi, or a single-run PoE camera to the attic (one cable to a switch only — cameras can still satisfy "no new cross-room cables" if Wi‑Fi). |
| **Local storage** | Recordings directory on the NAS or server SSD/HDD; Frigate retention rules in config. |
| **Multi-user** | Hub dashboards with per-user views; or a shared Frigate UI on LAN/VPN. |
| **Remote** | Mesh VPN to Frigate/hub; Reolink app as a backup if the cameras are Reolink. |
| **Cost** | $0 software; 3× ~$50–90 cameras + optional Coral USB $60; the server is shared with the media stack. |

**Capabilities**
- 24/7 or motion-only; zones; person/package filters with a Coral accelerator.
- Clips + timeline; no subscription.

**Tradeoffs**

| Pros | Cons |
|------|------|
| Scales down perfectly to 2–3 cameras | Initial YAML tuning |
| Strong local retention control | Wi‑Fi cameras need solid 2.4/5 GHz placement |
| Hub integration optional | Frigate UI role-based access is basic |

**Technical fit:** When a home server (Immich/TrueNAS) already runs or is planned, and local AI clips are wanted.

---

## Solution B — Reolink Wi‑Fi cameras + microSD or Reolink Home Hub

| | |
|---|---|
| **What it is** | Standalone cameras record to **on-camera SD** and/or a **Reolink Home Hub** (local NVR box, no cloud required). |
| **Local storage** | SD per camera (64–128 GB) or a Hub with a 2.5" drive. |
| **Multi-user** | Reolink app; shared household login or guest access (model-dependent). |
| **Remote** | Reolink app (P2P optional — disable for LAN/VPN-only if desired). |
| **Cost** | 3× camera $150–270; Home Hub ~$130 + drive. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Fastest path for 2–3 cameras | Weak hub-integration story |
| All wireless to the hub | The Hub is another box |
| Good iOS/Android apps | AI features vary by model |

**Technical fit:** Record locally with minimal homelab; acceptable Reolink-app UX.

---

## Solution C — Blue Iris + RTSP (Windows NVR)

| | |
|---|---|
| **What it is** | A Windows app records RTSP streams from 2–3 cameras to local disk. |
| **Local storage** | Dedicated folder on HDD; Blue Iris retention policies. |
| **Multi-user** | Blue Iris mobile apps; multiple login profiles. |
| **Cost** | ~$70 license + an existing Windows PC. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Polished playback UI | Requires an always-on Windows box |
| Easy for 2–3 streams | Overkill licensing vs. Frigate for a tiny fleet |

**Technical fit:** Prefer a Windows GUI over Docker.

---

## Solution D — Scrypted + HomeKit Secure Video (iPhone-primary)

| | |
|---|---|
| **What it is** | Bridge RTSP cameras into Apple Home; clips in iCloud if subscribed. |
| **Local storage** | **Not fully local** for HKSV history — clips go to iCloud; RTSP is still available locally via Scrypted. |
| **Multi-user** | Apple Home invites. |
| **Cost** | Free Scrypted + an iCloud+ tier. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Best iPhone live view | Android secondary; cloud clip history |
| Fine for 2–3 cameras | Conflicts with "maximally local" if HKSV is primary |

**Technical fit:** An Apple-heavy household that accepts cloud clip storage for short history.

---

## Solution E — Frigate + Automation hub (integrated alerts)

Same hardware as **Solution A**, with the hub as the family-facing layer:

| | |
|---|---|
| **Multi-user** | Hub accounts: adults see live + clips; restricted dashboards for guests. |
| **Remote** | Hub companion app + mesh VPN; push notification on person-at-door. |
| **Cost** | Incremental $0 beyond Solution A + a hub box if separate. |

**Technical fit:** Same server as the media/automation projects — one VPN, one app for cameras + lights.

---

## Camera & storage notes (2–3 camera scale)

| Topic | Guidance |
|-------|----------|
| **Wireless** | Wi‑Fi cameras avoid cross-room cabling; use 5 GHz if close to the AP, prefer wired AP backhaul |
| **Protocols** | ONVIF/RTSP; disable cloud upload in camera settings if local-only |
| **Retention** | 14–30 days on 500 GB–1 TB of dedicated space is comfortable |
| **Privacy** | IoT/camera VLAN; block WAN egress on the camera subnet |
| **Multi-user** | One admin NVR; family uses the hub or Reolink viewer, not the shared admin password |

---

## Comparison matrix

| Solution | Local-only | 2–3 cam fit | iOS + Android apps | Hub integration | DIY |
|----------|------------|-------------|--------------------|-----------------|-----|
| A Frigate | ★★★★★ | ★★★★★ | ★★★★ (via hub/web) | ★★★★★ | Medium |
| B Reolink + Hub | ★★★★★ | ★★★★★ | ★★★★★ | ★★ | Low |
| C Blue Iris | ★★★★★ | ★★★★★ | ★★★★★ | ★★★ | Low–Med |
| D Scrypted/HKSV | ★★★ | ★★★★★ | ★★★★★ iOS | ★★★★ | Medium |
| E Frigate + hub | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | Medium |

---

## Recommended path (local, 2–3 cameras)

1. **Cameras:** 2–3× Reolink or Amcrest **Wi‑Fi**, RTSP enabled, cloud off.
2. **NVR:** Frigate on the shared N100/TrueNAS box; recordings dataset `security/recordings`.
3. **Detection:** Skip the Coral for 2–3 cameras — CPU detection is fine at this scale.
4. **Remote:** Mesh VPN; family uses **hub camera cards** or the Reolink app.
5. **Backup:** Optional nightly sync of event clips only — not a full 24/7 mirror.

**Rough budget:** 3× $70 camera + $500 server (shared with media) ≈ **$710** if starting fresh; **~$210** cameras only if the server already exists.

---

## Scored recommendation

**Top pick — Frigate on the same N100 server + 2–3 Wi‑Fi RTSP cameras.** Reuses the media server; local retention; hub-native alerts; no subscription. At 2–3 cameras a Coral accelerator is unnecessary.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$150–350** for 2–3 cameras (Reolink/Amcrest Wi‑Fi $50–90 each); **+$0** compute if it shares the Immich box; optional Coral $60 (skip at this scale). |
| **Setup time** | **0.5–1 day** (cameras, RTSP, Frigate config, retention rules, hub cards). |
| **Maintenance** | **Low–Med** — occasional firmware/zone tuning; storage auto-prunes. |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★★ — grows easily to 6–8 cameras; add a Coral when you do. |

**Reuse:** shares the **N100 server** with Media (recordings live on the same pool, one backup job) and the **automation hub + mesh VPN** for viewing/alerts. The camera VLAN overlaps the IoT VLAN used for ESP/IR devices.

**Alternate A — Reolink Wi‑Fi cameras + Reolink Home Hub** ($280–530; setup 1–2 hrs; maintenance Low; feasibility ★★★★★; scalability ★★★). Pick if you don't want cameras on the server; weak hub story, good native apps.

**Alternate B — Blue Iris (Windows)** (~$70 + PC; setup Low–Med; scalability ★★★★). Polished mobile apps, but adds an always-on Windows box and overkill licensing for 3 cameras. Only if a Windows GUI is preferred.

**Verdict:** **Frigate on the shared server** is the clear winner once the N100 is already being built for Immich — near-zero added compute cost, full local control, and it plugs into the same hub dashboards and mesh VPN set up for everything else. Skip the Coral until you exceed ~4 cameras.

---

## Full dossier — recommended build (Frigate + 2–3 Wi‑Fi RTSP cameras)

### Concept image

![Wireless cameras recording to a local NVR with phone viewing](./assets/concept-security.png)

### Parts list

| Part | Qty | Unit $ | Total |
|------|-----|--------|-------|
| Reolink/Amcrest Wi‑Fi RTSP camera (1080p–4 MP, ONVIF) | 3 | $70 | $210 |
| 1 TB dedicated HDD/SSD for `security/recordings` (or space on the existing pool) | 1 | $45 | $45 |
| (Optional) Google Coral USB accelerator — only beyond ~4 cams | 0 | $60 | $0 |
| Existing N100 server + mesh VPN + hub | — | $0 | $0 |
| **Total (3 cams, shared server)** | | | **~$255** |

Frigate, go2rtc, and the hub integration are free and open-source.

### Cost example

- **Cameras only (server exists):** 3× $70 + 1 TB disk $45 ≈ **$255**.
- **Fresh (no server yet):** above + N100 server $230 (shared with photos) ≈ **$485**.
- **2-camera minimal:** 2× $70 + reuse existing pool space ≈ **$140**.
- Optional Coral **$60** is deferred until the fleet exceeds ~4 cameras.

### Data path

```
[Cam 1] [Cam 2] [Cam 3]   (RTSP/ONVIF, cloud upload disabled in-camera)
   └───────┼───────┘
           ▼  RTSP over Wi-Fi (camera VLAN)
        go2rtc ──► Frigate (CPU object detection: person/car)
           │              │
           │              ├─ event clips + 24/7 → security/recordings/ (local disk)
           │              └─ detections → Home Assistant → push to phones
           ▼
   Live view / playback ── Mesh VPN ──► [phones, away from home]
```

### General wiring / topology diagram

```
   Internet
      │
  ┌───┴────┐
  │ Router │
  └───┬────┘
      │ VLAN trunk
 ┌────┴───────────────┐
 │  Wi-Fi AP (SSIDs)   │
 │  • main   • IoT/cam │── camera VLAN: WAN egress BLOCKED
 └──┬────┬────┬────────┘
  cam1 cam2 cam3   (PoE optional via 1 in-room run to a small PoE switch)
      │ RTSP
 ┌────┴───────┐
 │ N100 server│  Frigate + recordings disk (shared ZFS pool)
 └────────────┘
```

### Effort to create

| Phase | Effort |
|-------|--------|
| Mount/place cameras, join IoT VLAN, enable RTSP, disable cloud | 2–3 hrs |
| Deploy Frigate, add camera streams via go2rtc | 1–2 hrs |
| Tune zones / motion masks / retention rules (YAML) | 1–2 hrs |
| Hub camera cards + person-at-door notifications | 30–60 min |
| **Total** | **~0.5–1 day** |

### Maintenance cost

- **Electricity:** cameras ~3–5 W each ≈ **$5–12/yr** total; the NVR runs on the already-on server (no incremental compute cost).
- **Storage:** 2–3 cams ≈ **60–180 GB/mo**; auto-prunes at the retention limit — **$0/mo** ongoing.
- **Time:** occasional firmware updates + zone re-tuning ≈ **15–30 min/quarter**.
- **Subscriptions:** none.

### Cost feasibility & scalability

- **Marginal cost ≈ $50–90 per camera** plus a little disk; no per-camera license.
- **Detection cliff:** CPU detection on the N100 is comfortable to **~4 cameras**; beyond that add a **Coral USB ($60, one-time)** rather than a bigger CPU.
- **Storage scales linearly** with camera count × retention; a single larger disk swap covers growth cheaply.
- **Takeaway:** very low entry and near-zero marginal cost up to ~4 cameras; a single $60 step unlocks 8+.

### Potential risks & downsides

| Risk | Mitigation |
|------|-----------|
| **Wi‑Fi camera dropouts / 2.4 GHz congestion** | Strong AP placement, wired AP backhaul, or one PoE run for the farthest camera |
| **All recordings are local — fire/theft loses them** | Optional nightly sync of *event clips only* to the photo offsite backup |
| **Camera firmware phoning home** | Camera VLAN with WAN egress blocked; disable P2P/cloud in-camera |
| **Retention sized wrong → premature pruning** | Size disk for desired days; alert at 80% usage |
| **YAML misconfig misses events** | Verify zones with test walks; keep 24/7 record as a safety net early on |
| **A camera doubles as a privacy risk indoors** | Place intentionally; document what each camera sees |
