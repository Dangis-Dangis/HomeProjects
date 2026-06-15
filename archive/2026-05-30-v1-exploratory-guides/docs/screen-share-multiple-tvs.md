# Wireless Screen Share (Multiple TVs & Devices)

**Constraints:** No HDMI or network cables between rooms, apartments, or homes—**wireless only**.

**Your TVs:**
- **Living room:** Vizio **V4K65M-08** — SmartCast + **Chromecast built-in** + AirPlay 2; audio on an **old Samsung sound system via ARC (IR-controlled, ≈2 remotes)**; old Roku present but unused.
- **Bedroom:** Vizio **E55-U** (older, limited SmartCast), mounted above a PC; primary source is a **Chromecast 4K**; audio from TV speakers or an ARC soundbar.

**Typical sources:** YouTube and streaming services via **Vizio built-in apps** (living room) and **Chromecast 4K** (bedroom); occasional Chrome tab cast.

**Stretch goals:**
- Start playback from **either TV** (Chromecast built-in / Vizio SmartCast)
- Cast to **any device on Wi‑Fi** (TVs, phones, tablets)
- Source from **phone or laptop**
- **Share with someone in another state** (synced or same content remotely)

**Reality check:** Wireless multi-TV **mirroring** of arbitrary DRM streaming (Netflix tab) to many screens is not officially supported and often violates ToS if restreamed. Practical paths: **same app/link on each device**, **Cast orchestration**, or **synced playback** where APIs allow (YouTube, Jellyfin SyncPlay).

**Audio note:** Both TVs already have **Chromecast built-in** (LR) / a **Chromecast 4K** (BR), so they're symmetric Cast targets. The wrinkle is **living-room volume/mute lives on the Samsung system over IR**, not on the TV—so any "play + set volume" macro routes audio through the IR layer (see [home-assistant-integration.md](./home-assistant-integration.md) and [device-control-similarity.md](./device-control-similarity.md)).

---

## Solution A — Home Assistant + Google Cast + Vizio SmartCast (multi-TV orchestration)

| | |
|---|---|
| **What it is** | Each TV is a `media_player`: the **living-room V4K65M-08** via Chromecast-built-in (`cast`) or `vizio`; the **bedroom Chromecast 4K** via `cast`/`androidtv`. HA sends the same YouTube URL or `play_media` to both. |
| **Sources** | YouTube (best), Spotify/audio, some DLNA; tab cast initiated from Chrome, then HA coordinates *additional* TVs for YouTube links. |
| **From either TV** | Both TVs are Chromecast targets—any phone/laptop on Wi‑Fi can cast; HA can trigger playback without physical HDMI. |
| **Any Wi‑Fi device** | Phones/tablets use YouTube/Cast client; both TVs are Cast receivers. |
| **Volume** | Bedroom: Cast/CEC to TV/soundbar. **Living room: volume/mute via IR to the Samsung system** (Broadlink), not the Cast volume. |
| **Another state** | Tailscale to home + mDNS relay (or manual IP cast where supported); remote user joins Jellyfin SyncPlay or you share a YouTube link—see Solution E. |
| **Cost** | $0—both TVs already cast; Broadlink ~$35 if you want HA-driven living-room volume. |

**Capabilities**
- Scene: “Play URL on living room + bedroom” via Cast.
- Voice optional via Google Assistant tied to the same Cast targets.
- Works with **Vizio built-in** (SmartCast/Chromecast) and the **Chromecast 4K**.

**Tradeoffs**
| Pros | Cons |
|------|------|
| No cables; uses your existing TVs | Not frame-locked; 1–5 s sync drift |
| YouTube works well on both | Tab mirror is still one primary Cast session |
| Central automations | Netflix/Hulu: no multi-TV mirror—launch app per TV |
| | LR volume needs the IR/Broadlink layer |

**Technical fit:** You already cast on both TVs; add HA for one-tap multi-TV YouTube.

---

## Solution B — Manual multi-Cast + Google Home app

| | |
|---|---|
| **What it is** | Phone/laptop casts to one TV; for YouTube, open same video on other TVs via app or voice; audio can use **Speaker Group** (synced audio only). |
| **Cost** | $0. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Zero setup | Tedious for 3+ TVs every time |
| Native Google UX | No true multi-TV video group |

**Technical fit:** Occasional use; not daily “same game on all TVs.”

---

## Solution C — Jellyfin + SyncPlay (owned media + home LAN)

| | |
|---|---|
| **What it is** | Personal video library on home server; Jellyfin clients on Fire TV, Chromecast (cast from phone), Android TV, mobile; **SyncPlay** keeps viewers roughly aligned. |
| **Sources** | Your files—not live YouTube/Netflix unless you own the file. |
| **Any Wi‑Fi device** | Jellyfin apps everywhere. |
| **Another state** | Tailscale + Jellyfin remote; invite to SyncPlay session. |
| **Cost** | $0 software on existing NAS/server. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Good cross-state watch-together for owned media | Not for arbitrary streaming apps |
| Multi-user accounts | Needs server you maintain |

**Technical fit:** Movie nights from ripped/owned library; complements Cast for streaming apps.

---

## Solution D — AirPlay 2 + Chromecast hybrid (phone/laptop sources)

| | |
|---|---|
| **What it is** | iPhone → AirPlay to compatible TV; Android/Chrome → Cast; mix ecosystems room by room. |
| **Multi-TV** | AirPlay 2 can target multiple **audio** outputs; video multi-target still limited—often one video + multi-room audio. |
| **Cost** | Built into many Vizio/Samsung/LG models; or Apple TV / Chromecast dongles. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Excellent phone → TV path | iOS and Android need different flows |
| No cables | No unified “one button all TVs” without HA |

**Technical fit:** Mixed iPhone + Android household casting from personal devices.

---

## Solution E — Remote viewing / another state (Tailscale + link sync)

| | |
|---|---|
| **What it is** | **Same content, not pixel mirror:** (1) Share YouTube/stream link; (2) **Jellyfin SyncPlay** or **Teleparty** for supported services; (3) Tailscale into home LAN so remote laptop acts as if local for Cast (advanced, flaky). |
| **Phone/laptop source** | Broadcaster picks content; HA webhook or simple app sends URL to home script that starts all home TVs. |
| **Cost** | Tailscale free tier; Teleparty free. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Works across states without restreaming DRM | Not true mirror of arbitrary tab |
| Jellyfin SyncPlay is reliable for files | Cast over VPN needs mDNS help |

**Technical fit:** Family in another city watching same YouTube or owned movie together.

---

## Solution F — Self-hosted HLS restream (laptop/obs → Wi‑Fi clients)

| | |
|---|---|
| **What it is** | Laptop captures window/tab (OBS) → RTMP/HLS on home server → TVs/phones pull via VLC/Kodi/browser on same Wi‑Fi. |
| **Sources** | Non-DRM or content you’re allowed to capture; **not** a workaround for Netflix DRM. |
| **Another state** | Tailscale + HLS URL; higher latency (5–15 s). |
| **Cost** | $0 software; CPU load on laptop/server. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| One stream many clients on Wi‑Fi | Latency; DRM blocked |
| Flexible | Overkill for YouTube (use Cast instead) |

**Technical fit:** Presentations, sports from OTA tuner, internal streams—not primary streaming apps.

---

## Solution G — Stremio + WebTorrent / community addons (niche)

| | |
|---|---|
| **What it is** | Stremio on each device; some setups support sync viewing; heavy dependency on addons. |
| **Cost** | Free. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Single catalog UI | Legal/quality varies; not for mainstream family Netflix |

**Technical fit:** Optional; not recommended as primary household stack.

---

## Stretch goal mapping

| Goal | Best solutions | Notes |
|------|----------------|-------|
| From either TV (Chromecast/Vizio) | A, D | TVs are Cast targets; phone initiates |
| To any Wi‑Fi device | A, C, D | Cast + Jellyfin + mobile apps |
| Phone or laptop source | A, B, D | Chrome cast tab → one TV; YouTube URL → HA multi-play |
| Another state | C, E | SyncPlay, shared links, Tailscale; avoid DRM restream |

---

## Comparison matrix

| Solution | YouTube multi-TV | Streaming apps | Tab/laptop | Cross-state | No new cables |
|----------|------------------|----------------|------------|-------------|---------------|
| A HA + Cast/Vizio | ★★★★★ | ★★★ (per device) | ★★★ | ★★★ | ★★★★★ |
| B Google Home manual | ★★★ | ★★★ | ★★★★ | ★★ | ★★★★★ |
| C Jellyfin SyncPlay | ★ | ★★ (owned) | ★★ | ★★★★★ | ★★★★★ |
| D AirPlay + Cast | ★★★★ | ★★★ | ★★★★ | ★★ | ★★★★★ |
| E Tailscale + links | ★★★★ | ★★ | ★★★ | ★★★★ | ★★★★★ |
| F HLS restream | ★★ | ★ | ★★★★ | ★★★ | ★★★★★ |

---

## Cost example (your 2 TVs, wireless)

| Item | Cost |
|------|------|
| Living room V4K65M-08 (Chromecast built-in, existing) | $0 |
| Bedroom Chromecast 4K (existing) | $0 |
| Broadlink RM4 (HA-driven living-room volume/mute) | $35 |
| Home Assistant (if not already) | $120–250 |
| **Total incremental** | **$35–285** |

---

## Recommended path

1. **Receivers are already standardized:** Chromecast built-in (V4K65M-08) + Chromecast 4K (bedroom); put them on the **same Google account**.  
2. **HA script** `play_youtube_everywhere(url)` → `media_player.play_media` on both Cast entities.  
3. **Living-room audio:** route volume/mute through **Broadlink → Samsung system** (IR), since ARC audio isn't on the TV's Cast volume.  
4. **Streaming apps:** accept per-TV launch for Netflix/Hulu (Vizio apps in LR, Chromecast in BR).  
5. **Remote family:** Jellyfin SyncPlay for owned files; YouTube **shared link + simultaneous HA trigger** or Teleparty where supported.  
6. **Do not plan on** a wireless HDMI matrix—cable-free means Cast/API orchestration, not bit-identical mirror.

**HA integration:** Scenes tie screen share to lights/audio, and the **bedroom Chromecast 4K** doubles as the commercial-skip test bed (see [home-assistant-integration.md](./home-assistant-integration.md)).
