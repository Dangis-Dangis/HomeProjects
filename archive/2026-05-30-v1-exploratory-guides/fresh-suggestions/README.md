# Fresh Suggestions — Opinionated, Scored Recommendations

This directory is a **second pass** over the exploratory guides in the parent folder. Where those files list *every* option, these files commit to a **top pick + 1–2 alternates per project** and score each on a consistent rubric, then tie the whole portfolio together.

**What changed vs. the original guides:** same constraints (wireless-first, iPhone + Android, local storage, no cross-room cabling, your specific Vizio + Samsung + IR-light + wall-switch hardware), now distilled into "here's what to actually build, in what order, and what it costs in money and time."

---

## Scoring rubric (used in every file)

| Dimension | Scale | Meaning |
|-----------|-------|---------|
| **Cost range** | USD | Devices + realistic combos; excludes gear you already own (TVs, phones, PCs) |
| **Setup time** | hours / days | Initial build to "working for the household" |
| **Maintenance** | Low / Med / High | Ongoing effort (updates, recharges, drive swaps) |
| **Feasibility** | ★1–5 | Given your skills (technical, EE, 3D print, custom wiring) — 5 = trivial for you |
| **Scalability** | ★1–5 | How gracefully it grows (more cams/TVs/remotes/storage/users) |

Plus a **Project similarity** note: which shared building blocks it reuses.

---

## The portfolio at a glance

| # | Project | Top pick | Cost (top pick) | Setup | Feasibility | Scalability |
|---|---------|----------|-----------------|-------|-------------|-------------|
| 1 | [Media storage](./01-media-storage.md) | TrueNAS/N100 + Immich + Tailscale | $500–1,100 | 1–2 days | ★★★★★ | ★★★★★ |
| 2 | [Security](./02-security-recording.md) | Frigate on same server + 2–3 Wi‑Fi cams | $150–350 (+server) | 0.5–1 day | ★★★★★ | ★★★★ |
| 3 | [HA lights/TV/audio](./03-home-assistant.md) | HAOS + Caseta + Broadlink + Cast | $350–700 | 1–3 days | ★★★★★ | ★★★★★ |
| 4 | [Screen share](./04-screen-share.md) | HA + Cast on both Vizios | $35–285 | 2–4 hrs | ★★★★★ | ★★★★ |
| 5 | [Unified remote](./05-unified-remote.md) | ESP32‑S3 ESPHome deck | $45–95 | 2–4 days | ★★★★★ | ★★★★★ |
| 6 | [RFID scenes](./06-rfid-scenes.md) | Phone NFC + a few ESP readers | $0–60 | 1–4 hrs | ★★★★★ | ★★★★ |

**Whole-portfolio hardware (fresh-built, excluding TVs/phones):** roughly **$1,100–2,600** depending on storage size, camera count, and how elaborate the remote and lighting get.

---

## Shared backbone (build once, reused everywhere)

```
            ┌──────────── Tailscale (all phones, laptops, server) ────────────┐
            │                                                                  │
   ┌────────┴─────────┐         ┌───────────────────┐         ┌───────────────┴───────┐
   │  N100 server      │◄───────│  Home Assistant    │◄────────│  ESPHome/IR rail      │
   │  Immich + Frigate │         │  (hub + Assist)    │         │  Broadlink, ESP32     │
   └───────────────────┘         └─────────┬─────────┘         │  deck, mic, sensors   │
                                            │                   └───────────────────────┘
                                            ▼
                          Cast (V4K65M-08, Chromecast 4K) · Caseta switches · Sonos/Cast audio
```

| Backbone piece | Projects that reuse it | Build-once payoff |
|----------------|------------------------|-------------------|
| **Tailscale mesh** | 1, 2, 3, 4, 5 | One remote-access config for photos, cameras, HA, remote casting |
| **N100/TrueNAS server** | 1, 2 | Immich + Frigate share CPU/disk; one backup job |
| **Home Assistant** | 2, 3, 4, 5, 6 | The hub every control/automation talks to |
| **ESP/ESPHome rail** | 3 (IR lights), 5 (remote), 6 (readers) | Same toolchain (ESPHome + MQTT + HA API) for all DIY hardware |
| **Google Cast** | 3, 4 | Both Vizios are Cast targets; one casting pattern |
| **IR layer (Broadlink)** | 3 (Samsung audio + IR lights), 4 (LR volume), 5 (deck can fire IR macros) | One blaster, many consumers |

---

## Recommended build order (dependency-aware)

1. **Home Assistant hub + Tailscale** — everything else plugs into it. *(1–2 days)*
2. **N100 server + Immich** — highest daily value (photos), and the box also hosts Frigate later. *(1 day)*
3. **Lighting + IR + TV macros (HA)** — Caseta switches, Broadlink for Samsung/IR lights, Cast for TVs. *(spread over a week)*
4. **Security (Frigate)** — adds onto the existing server; 2–3 Wi‑Fi cams. *(half day)*
5. **Screen share** — mostly config once Cast + HA exist. *(an afternoon)*
6. **Unified remote** — the capstone; prototype on M5Stack, then build the ESP32‑S3 deck once your macros are stable. *(2–4 days of fun)*
7. **RFID scenes** — optional sugar; phone NFC first, ESP readers if you enjoy it. *(an hour)*

> Rationale: the remote (5) is most satisfying **after** the HA scenes/macros it triggers (3, 4) already exist, so each physical control maps to something real.

---

## Cross-project similarity matrix

| Capability | Media | Security | HA | Screen share | Remote | RFID |
|------------|:-----:|:--------:|:--:|:------------:|:------:|:----:|
| Tailscale | ● | ● | ● | ● | ● | ○ |
| Home Assistant | ○ | ● | ● | ● | ● | ● |
| N100/TrueNAS server | ● | ● | ○ | ○ | ✗ | ✗ |
| ESPHome / ESP32 | ✗ | ✗ | ● | ✗ | ● | ● |
| IR (Broadlink) | ✗ | ✗ | ● | ● | ● | ✗ |
| Google Cast | ✗ | ✗ | ● | ● | ○ | ✗ |
| HA Assist (voice) | ✗ | ✗ | ● | ○ | ● | ✗ |

● primary · ○ supporting · ✗ n/a

**Biggest reuse wins:** (a) one **server** does media + security; (b) one **HA + ESPHome + IR** toolchain does lights, TV/audio, the remote, and RFID; (c) one **Tailscale** config makes all of it remote.

See each numbered file for the scored breakdown.
