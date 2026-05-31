# 04 — Wireless Screen Share (Fresh Suggestion)

**Need:** Wireless (no cabling) sharing of YouTube + streaming to your **living-room V4K65M-08** and **bedroom Chromecast 4K**; from phone/laptop; stretch: from either TV, to any Wi‑Fi device, and to family in another state.

Full option list: [../screen-share-multiple-tvs.md](../screen-share-multiple-tvs.md)

---

## Top pick — Home Assistant + Google Cast on both Vizios

Both TVs are already Cast targets (Chromecast built-in on the V4K65M-08, Chromecast 4K in the bedroom). HA scripts the same YouTube URL to both; per-app launch for Netflix/Hulu.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$35–285**: $0 if you only cast (TVs already do it); **+$35 Broadlink** to drive **living-room volume/mute** on the Samsung system; +$120–250 only if HA isn't already built. |
| **Setup time** | **2–4 hrs** once HA + Cast exist (write `play_youtube_everywhere` script, wire LR volume to Broadlink). |
| **Maintenance** | **Low** — re-test Cast after HA/TV firmware updates. |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★★ — add any Cast device as a target; audio sync drifts 1–5 s across rooms. |

**Honest limits:** no true wireless mirror of DRM streaming (Netflix tab) to many screens; plan on **same app/link per device** or **YouTube/`play_media`** orchestration. Living-room volume is **not** the Cast volume — it rides the IR layer to the Samsung system.

**Cross-state:** Tailscale + **Jellyfin SyncPlay** for owned media; shared YouTube link + simultaneous HA trigger for live-ish "watch together."

**Project similarity:** Same **Cast** + **HA** + **Tailscale** as the HA project; **shares the Broadlink IR layer** for living-room audio; the **Unified remote** can expose a "cast to both TVs" button.

---

## Alternate A — Jellyfin SyncPlay (owned media, cross-state)

Best for watch-together of files you own across states. **Cost $0** on the existing server; **Setup low**; **Scalability ★★★** (per-user streams). Complements, doesn't replace, Cast for streaming apps.

## Alternate B — AirPlay + Cast hybrid

Use AirPlay from iPhones (V4K65M-08 supports AirPlay 2) and Cast from Android. **Cost $0**; good for casual phone→TV; **no one-button-all-TVs** without HA.

---

## Verdict

There's almost nothing to buy — this is a **configuration project** layered on the HA + Cast backbone. The single purchase worth making is the **Broadlink** (already justified by the HA project) so the living-room "play" macro can also set Samsung volume. Use **Jellyfin SyncPlay** for the out-of-state watch-together goal.
