# 02 — Home Security Recording (Fresh Suggestion)

**Need:** **2–3 cameras**, **local storage**, multi-user remote viewing.

Full option list: [../home-security-recording.md](../home-security-recording.md)

---

## Top pick — Frigate on the same N100 server + 2–3 Wi‑Fi RTSP cameras

Reuses the media server; local retention; HA-native alerts; no subscription. At 2–3 cams you don't even need a Coral.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$150–350** for 2–3 cameras (Reolink/Amcrest Wi‑Fi $50–90 each); **+$0** compute if it shares the Immich box; optional Coral $60 (skip at this scale). |
| **Setup time** | **0.5–1 day** (cameras, RTSP, Frigate config, retention rules, HA cards). |
| **Maintenance** | **Low–Med** — occasional firmware/zone tuning; storage auto-prunes. |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★★ — easily grows to 6–8 cams; add Coral when you do. |

**Project similarity:** Shares the **N100 server** with Media (recordings live on the same pool, one backup job) and **Home Assistant + Tailscale** for viewing/alerts. Camera VLAN overlaps the IoT VLAN you'll use for ESP/IR devices.

---

## Alternate A — Reolink Wi‑Fi cams + Reolink Home Hub (no homelab)

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$280–530** (cams + hub + drive). |
| **Setup time** | **1–2 hrs.** |
| **Maintenance** | **Low.** |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★ (Reolink ecosystem). |

Pick if you don't want cameras on your server. Weak HA story; good native iOS/Android apps.

---

## Alternate B — Blue Iris (Windows)

Polished mobile apps, but adds a Windows always-on box and overkill licensing for 3 cams. **Cost ~$70 + PC; Setup low–med; Scalability ★★★★.** Only if you prefer a Windows GUI.

---

## Verdict

**Frigate on the shared server** is the clear winner once you're already building the N100 for Immich — near-zero added compute cost, full local control, and it plugs into the same HA dashboards and Tailscale remote you set up for everything else. Skip Coral until you exceed ~4 cameras.
