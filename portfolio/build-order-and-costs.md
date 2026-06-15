# Build Order & Costs

## Recommended order (dependency-aware)

| Step | What | Time | Unlocks |
|------|------|------|---------|
| 1 | **HA + Tailscale** | 1–2 days | Everything else |
| 2 | **N100 + Immich** | 1 day | Daily photo value; server for Frigate |
| 3 | **Lighting + IR + TV macros** | ~1 week | Caseta, Broadlink, Cast scenes |
| 4 | **Frigate + 2–3 Wi‑Fi cams** | 0.5–1 day | Security on existing server |
| 5 | **Screen share scripts** | 2–4 hrs | Multi-TV YouTube |
| 6 | **Unified remote** | 2–4 days | Physical front-end (after macros exist) |
| 7 | **RFID/NFC** | 1–4 hrs | Optional convenience |

> Build the remote **last** so every button maps to a real scene.

---

## Cost summary (recommended picks)

| Project | Parts | Setup | Maintenance |
|---------|-------|-------|-------------|
| 01 Media | $500–1,100 | 1–2 days | Med (updates, drives) |
| 02 Security | $150–350 | 0.5–1 day | Low–Med |
| 03 Control | $350–700 | 1–3 days | Med |
| 04 Screen share | $35–285* | 2–4 hrs | Low |
| 05 Remote | $45–90 | 2–4 days | Low–Med |
| 06 RFID | $0–60 | 1–4 hrs | Low |

\* Mostly $0 if HA exists; +$35 Broadlink if not already bought for P3.

**Portfolio total:** ~**$1,100–2,600** (excluding TVs, phones, PCs you own).

Optional: **+$25/mo** B2 offsite backup for photos.

---

## Similarity matrix

| Block | Media | Sec | Control | Share | Remote | RFID |
|-------|:-----:|:---:|:-------:|:-----:|:------:|:----:|
| Tailscale | ● | ● | ● | ● | ● | ○ |
| HA | ○ | ● | ● | ● | ● | ● |
| N100 server | ● | ● | ○ | ○ | ✗ | ✗ |
| ESPHome | ✗ | ✗ | ● | ✗ | ● | ● |
| Broadlink | ✗ | ✗ | ● | ● | ● | ✗ |
| Cast | ✗ | ✗ | ● | ● | ○ | ✗ |

● primary · ○ supporting · ✗ n/a

---

## Scoring rubric (for comparing options)

| Dimension | Scale |
|-----------|-------|
| Cost | USD parts |
| Setup | hours / days to household-usable |
| Maintenance | Low / Med / High |
| Feasibility | ★1–5 (you: ★5 on most) |
| Scalability | ★1–5 |

Archived scored picks: [archive/2026-05-30-v1-exploratory-guides/fresh-suggestions/](../archive/2026-05-30-v1-exploratory-guides/fresh-suggestions/).

Full dossiers: [archive/2026-05-31-v2-home-systems-proposal/home-systems-proposal/](../archive/2026-05-31-v2-home-systems-proposal/home-systems-proposal/).
