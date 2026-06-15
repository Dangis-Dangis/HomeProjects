# Solutions — Wireless Screen Share

**Problem (P4):** No HDMI or network cables between rooms, apartments, or homes — **wireless only**. Push the same content (primarily YouTube and streaming apps) to multiple TVs and Wi‑Fi devices, sourced from a phone or laptop, and watch the same thing together with family in another state.

**The TVs:**
- **Living room:** Vizio **V4K65M-08** — SmartCast + **Chromecast built-in** + AirPlay 2; audio on an **old Samsung sound system via ARC (IR-controlled, ≈2 remotes)**; an old Roku is present but unused.
- **Bedroom:** Vizio **E55-U** (older, limited SmartCast), mounted above a PC; primary source is a **Chromecast 4K**; audio from TV speakers or an ARC soundbar.

**Typical sources:** YouTube and streaming services via Vizio built-in apps (living room) and the Chromecast 4K (bedroom); occasional Chrome-tab cast.

**Reality check:** wireless multi-TV **mirroring** of arbitrary DRM streaming (e.g. a Netflix tab) to many screens is not officially supported and often violates terms of service if restreamed. Practical paths are **same app/link on each device**, **Cast orchestration**, or **synced playback** where APIs allow (YouTube, Jellyfin SyncPlay).

**Audio note:** both TVs already have Chromecast built-in (living room) / a Chromecast 4K (bedroom), so they are symmetric Cast targets. The wrinkle is that **living-room volume/mute lives on the Samsung system over IR**, not on the TV — so any "play + set volume" macro routes audio through the IR layer (see [solutions-03-home-control.md](./solutions-03-home-control.md)).

---

## Solution A — Automation hub + Google Cast + Vizio SmartCast (multi-TV orchestration)

| | |
|---|---|
| **What it is** | Each TV is a `media_player`: the **living-room V4K65M-08** via Chromecast built-in (`cast`) or `vizio`; the **bedroom Chromecast 4K** via `cast`/`androidtv`. The hub sends the same YouTube URL or `play_media` to both. |
| **Sources** | YouTube (best), Spotify/audio, some DLNA; tab cast initiated from Chrome, then the hub coordinates *additional* TVs for YouTube links. |
| **From either TV** | Both TVs are Chromecast targets — any phone/laptop on Wi‑Fi can cast; the hub can trigger playback without physical HDMI. |
| **Any Wi‑Fi device** | Phones/tablets use the YouTube/Cast client; both TVs are Cast receivers. |
| **Volume** | Bedroom: Cast/CEC to TV/soundbar. **Living room: volume/mute via IR to the Samsung system** (Broadlink), not the Cast volume. |
| **Another state** | Mesh VPN to home + an mDNS relay (or manual-IP cast where supported); the remote user joins Jellyfin SyncPlay or you share a YouTube link — see Solution E. |
| **Cost** | $0 — both TVs already cast; Broadlink ~$35 if you want hub-driven living-room volume. |

**Capabilities**
- Scene: "play URL on living room + bedroom" via Cast.
- Voice optional via a voice assistant tied to the same Cast targets.
- Works with **Vizio built-in** (SmartCast/Chromecast) and the **Chromecast 4K**.

**Tradeoffs**

| Pros | Cons |
|------|------|
| No cables; uses the existing TVs | Not frame-locked; 1–5 s sync drift |
| YouTube works well on both | A tab mirror is still one primary Cast session |
| Central automations | Netflix/Hulu: no multi-TV mirror — launch the app per TV |
| | Living-room volume needs the IR/Broadlink layer |

**Technical fit:** Both TVs already cast; add the hub for one-tap multi-TV YouTube.

---

## Solution B — Manual multi-Cast + Google Home app

| | |
|---|---|
| **What it is** | A phone/laptop casts to one TV; for YouTube, open the same video on the other TVs via app or voice; audio can use a **Speaker Group** (synced audio only). |
| **Cost** | $0. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Zero setup | Tedious for 3+ TVs every time |
| Native Google UX | No true multi-TV video group |

**Technical fit:** Occasional use; not a daily "same game on all TVs."

---

## Solution C — Jellyfin + SyncPlay (owned media + home LAN)

| | |
|---|---|
| **What it is** | A personal video library on the home server; Jellyfin clients on Fire TV, Chromecast (cast from phone), Android TV, mobile; **SyncPlay** keeps viewers roughly aligned. |
| **Sources** | Your files — not live YouTube/Netflix unless you own the file. |
| **Any Wi‑Fi device** | Jellyfin apps everywhere. |
| **Another state** | Mesh VPN + Jellyfin remote; invite to a SyncPlay session. |
| **Cost** | $0 software on the existing NAS/server. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Good cross-state watch-together for owned media | Not for arbitrary streaming apps |
| Multi-user accounts | Needs a server you maintain |

**Technical fit:** Movie nights from a ripped/owned library; complements Cast for streaming apps.

---

## Solution D — AirPlay 2 + Chromecast hybrid (phone/laptop sources)

| | |
|---|---|
| **What it is** | iPhone → AirPlay to a compatible TV; Android/Chrome → Cast; mix ecosystems room by room. |
| **Multi-TV** | AirPlay 2 can target multiple **audio** outputs; video multi-target is still limited — often one video + multi-room audio. |
| **Cost** | Built into many Vizio/Samsung/LG models; or Apple TV / Chromecast dongles. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Excellent phone → TV path | iOS and Android need different flows |
| No cables | No unified "one button, all TVs" without the hub |

**Technical fit:** A mixed iPhone + Android household casting from personal devices.

---

## Solution E — Remote viewing / another state (mesh VPN + link sync)

| | |
|---|---|
| **What it is** | **Same content, not a pixel mirror:** (1) share a YouTube/stream link; (2) **Jellyfin SyncPlay** or **Teleparty** for supported services; (3) mesh VPN into the home LAN so a remote laptop acts as if local for Cast (advanced, flaky). |
| **Phone/laptop source** | The broadcaster picks content; a hub webhook or simple app sends the URL to a home script that starts all home TVs. |
| **Cost** | Mesh VPN free tier; Teleparty free. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Works across states without restreaming DRM | Not a true mirror of an arbitrary tab |
| Jellyfin SyncPlay is reliable for files | Cast over VPN needs mDNS help |

**Technical fit:** Family in another city watching the same YouTube or an owned movie together.

---

## Solution F — Self-hosted HLS restream (laptop/OBS → Wi‑Fi clients)

| | |
|---|---|
| **What it is** | A laptop captures a window/tab (OBS) → RTMP/HLS on the home server → TVs/phones pull via VLC/Kodi/browser on the same Wi‑Fi. |
| **Sources** | Non-DRM or content you're allowed to capture; **not** a workaround for Netflix DRM. |
| **Another state** | Mesh VPN + HLS URL; higher latency (5–15 s). |
| **Cost** | $0 software; CPU load on the laptop/server. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| One stream, many Wi‑Fi clients | Latency; DRM blocked |
| Flexible | Overkill for YouTube (use Cast instead) |

**Technical fit:** Presentations, sports from an OTA tuner, internal streams — not primary streaming apps.

---

## Solution G — Stremio + WebTorrent / community add-ons (niche)

| | |
|---|---|
| **What it is** | Stremio on each device; some setups support sync viewing; heavy dependency on add-ons. |
| **Cost** | Free. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Single catalog UI | Legal/quality varies; not for mainstream family streaming |

**Technical fit:** Optional; not recommended as the primary household stack.

---

## Stretch-goal mapping

| Goal | Best solutions | Notes |
|------|----------------|-------|
| From either TV (Chromecast/Vizio) | A, D | TVs are Cast targets; the phone initiates |
| To any Wi‑Fi device | A, C, D | Cast + Jellyfin + mobile apps |
| Phone or laptop source | A, B, D | Chrome cast tab → one TV; YouTube URL → hub multi-play |
| Another state | C, E | SyncPlay, shared links, mesh VPN; avoid DRM restream |

---

## Comparison matrix

| Solution | YouTube multi-TV | Streaming apps | Tab/laptop | Cross-state | No new cables |
|----------|------------------|----------------|------------|-------------|---------------|
| A Hub + Cast/Vizio | ★★★★★ | ★★★ (per device) | ★★★ | ★★★ | ★★★★★ |
| B Google Home manual | ★★★ | ★★★ | ★★★★ | ★★ | ★★★★★ |
| C Jellyfin SyncPlay | ★ | ★★ (owned) | ★★ | ★★★★★ | ★★★★★ |
| D AirPlay + Cast | ★★★★ | ★★★ | ★★★★ | ★★ | ★★★★★ |
| E Mesh VPN + links | ★★★★ | ★★ | ★★★ | ★★★★ | ★★★★★ |
| F HLS restream | ★★ | ★ | ★★★★ | ★★★ | ★★★★★ |

---

## Cost example (the two TVs, wireless)

| Item | Cost |
|------|------|
| Living-room V4K65M-08 (Chromecast built-in, existing) | $0 |
| Bedroom Chromecast 4K (existing) | $0 |
| Broadlink RM4 (hub-driven living-room volume/mute) | $35 |
| Automation hub (if not already present) | $120–250 |
| **Total incremental** | **$35–285** |

---

## Recommended path

1. **Receivers are already standardized:** Chromecast built-in (V4K65M-08) + Chromecast 4K (bedroom); put them on the **same Google account**.
2. **Hub script** `play_youtube_everywhere(url)` → `media_player.play_media` on both Cast entities.
3. **Living-room audio:** route volume/mute through **Broadlink → Samsung system** (IR), since ARC audio is not on the TV's Cast volume.
4. **Streaming apps:** accept per-TV launch for Netflix/Hulu (Vizio apps in the living room, Chromecast in the bedroom).
5. **Remote family:** Jellyfin SyncPlay for owned files; a YouTube **shared link + simultaneous hub trigger**, or Teleparty where supported.
6. **Do not plan on** a wireless HDMI matrix — cable-free means Cast/API orchestration, not a bit-identical mirror.

**Hub integration:** scenes tie screen share to lights/audio, and the **bedroom Chromecast 4K** doubles as the commercial-skip test bed (see [solutions-03-home-control.md](./solutions-03-home-control.md)).

---

## Scored recommendation

**Top pick — Automation hub + Google Cast on both Vizios.** Both TVs are already Cast targets (Chromecast built-in on the V4K65M-08, Chromecast 4K in the bedroom). The hub scripts the same YouTube URL to both; per-app launch for Netflix/Hulu.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$35–285**: $0 if you only cast (the TVs already do it); **+$35 Broadlink** to drive **living-room volume/mute** on the Samsung system; +$120–250 only if the hub isn't already built. |
| **Setup time** | **2–4 hrs** once the hub + Cast exist (write `play_youtube_everywhere`, wire living-room volume to Broadlink). |
| **Maintenance** | **Low** — re-test Cast after hub/TV firmware updates. |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★★ — add any Cast device as a target; audio sync drifts 1–5 s across rooms. |

**Honest limits:** no true wireless mirror of DRM streaming to many screens; plan on **same app/link per device** or **YouTube/`play_media`** orchestration. Living-room volume is **not** the Cast volume — it rides the IR layer to the Samsung system.

**Cross-state:** mesh VPN + **Jellyfin SyncPlay** for owned media; a shared YouTube link + simultaneous hub trigger for live-ish "watch together."

**Reuse:** same **Cast + hub + mesh VPN** as Home control; **shares the Broadlink IR layer** for living-room audio; the **Unified remote** can expose a "cast to both TVs" button.

**Alternate A — Jellyfin SyncPlay** (owned media, cross-state). Best for watch-together of owned files across states. Cost $0 on the existing server; setup Low; scalability ★★★ (per-user streams). Complements, doesn't replace, Cast for streaming apps.

**Alternate B — AirPlay + Cast hybrid.** AirPlay from iPhones (the V4K65M-08 supports AirPlay 2) and Cast from Android. Cost $0; good for casual phone→TV; **no one-button-all-TVs** without the hub.

**Verdict:** There's almost nothing to buy — this is a **configuration project** layered on the hub + Cast backbone. The single purchase worth making is the **Broadlink** (already justified by Home control) so the living-room "play" macro can also set Samsung volume. Use **Jellyfin SyncPlay** for the out-of-state watch-together goal.

---

## Full dossier — recommended build (Hub + Google Cast on both TVs)

### Concept image

![Two TVs showing the same content, cast wirelessly from a phone](./assets/concept-screen-share.png)

### Parts list

| Part | Qty | Unit $ | Total |
|------|-----|--------|-------|
| Living-room V4K65M-08 (Chromecast built-in) | 1 | $0 (owned) | $0 |
| Bedroom Chromecast 4K | 1 | $0 (owned) | $0 |
| Broadlink RM4 Pro (living-room volume/mute on Samsung) | 1 | $35 | $35 |
| Home Assistant hub | 1 | $0 (shared) | $0 |
| (Optional) extra Chromecast/Google TV dongle per new TV target | 0 | $30–50 | $0 |
| **Total incremental (uses existing gear)** | | | **~$35** |

The Broadlink is typically already purchased for Home control, making this effectively a **$0 software/config** project.

### Cost example

- **Pure config (Broadlink already owned):** **$0**.
- **Add living-room volume control:** **+$35** Broadlink RM4.
- **Add a non-Cast TV as a target:** **+$30–50** Chromecast/Google TV dongle.
- **If the hub doesn't exist yet:** +$99–250 (but that's the shared backbone, not specific to screen share).

### Data path

```
[phone / laptop]                         [remote family, another state]
   │ cast (YouTube/Chrome tab)               │ Mesh VPN + Jellyfin SyncPlay / shared link
   ▼                                         ▼
Home Assistant  ── play_media (same URL) ──► Living-room V4K65M-08 (Chromecast built-in)
   │                                    └──► Bedroom Chromecast 4K
   │
   └─ living-room volume/mute ─► Broadlink RM4 ──IR──► Samsung sound system (ARC)
                                   (audio is NOT on the Cast volume)
```

### General wiring / topology diagram

```
            Internet ── Mesh VPN ──► remote viewers
               │
           ┌───┴────┐
           │ Router │
           └───┬────┘
        Wi-Fi  │  (same Google account on both Cast devices)
   ┌───────────┼────────────┬───────────────┐
[phone/      [V4K65M-08]   [Chromecast 4K]  [Home Assistant]
 laptop]      Cast rx        Cast rx          orchestrates
                │                              │
          Samsung audio ◄──IR── Broadlink RM4 ◄┘  (LR volume path)
          (ARC, no cable to TV Cast volume)
```

No A/V cabling between rooms — everything rides Wi‑Fi/Cast; the only "wire" is the IR line-of-sight from the Broadlink to the Samsung system.

### Effort to create

| Phase | Effort |
|-------|--------|
| Put both Cast devices on one Google account; confirm hub sees them | 30 min |
| Write `play_youtube_everywhere(url)` script (`media_player.play_media` ×2) | 1 hr |
| Wire living-room volume/mute to Broadlink in the play macro | 30–60 min |
| (Optional) Jellyfin SyncPlay for cross-state watch-together | 1 hr |
| **Total** | **~2–4 hrs** |

### Maintenance cost

- **Electricity:** none incremental (devices already powered).
- **Time:** re-test Cast after a TV/hub firmware update ≈ **10–15 min/quarter**.
- **Subscriptions:** none (YouTube/own media; optional Teleparty is free).

### Cost feasibility & scalability

- **Marginal cost to add a TV/screen:** **$0** if it has Chromecast built-in, else **~$30–50** for a dongle.
- **No per-stream or per-device fees**; scaling is just adding Cast entities to the hub script.
- **Soft limit:** audio sync drifts **1–5 s** across rooms and grows with more targets — fine for "same content," not for frame-locked sync.
- **Takeaway:** essentially free to scale to any number of Cast targets; the only cost is an optional dongle for a non-Cast screen.

### Potential risks & downsides

| Risk | Mitigation |
|------|-----------|
| **No true DRM mirror (Netflix/Hulu) to many screens** | Launch the app per TV; reserve multi-cast for YouTube/`play_media` |
| **1–5 s audio drift between rooms** | Accept for "same content"; use a Cast speaker group where tighter audio sync matters |
| **Cast discovery fails over VPN (cross-state)** | Use Jellyfin SyncPlay / shared links rather than casting remote; mDNS reflector only if needed |
| **Living-room volume isn't the Cast volume** | Always route LR volume/mute through Broadlink → Samsung |
| **Google account/firmware changes break casting** | Keep both devices updated and on one account; re-test after updates |
