# 06 — RFID / NFC Scene Triggers (Fresh Suggestion)

**Need (revised scope):** Tap-to-trigger **scenes/lights/media** — **not** door locks. Wireless-friendly; you'll happily build ESP readers if it's fun.

Full option list: [../rfid-custom-codes.md](../rfid-custom-codes.md)

---

## Top pick — Phone NFC first, add a few ESP readers if you enjoy it

Start with **zero hardware** (HA Companion NFC tags), then sprinkle **ESPHome PN532** readers where a fixed tap point is nice (entry shelf, nightstand, desk).

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$0–60**: NTAG stickers $0.50–2 each (phone-tap = $0 hardware); ESP32 + PN532 ~$12/reader if you add fixed pads. |
| **Setup time** | **1–4 hrs** (NFC tags instant; ESP readers ~30 min each + enrollment). |
| **Maintenance** | **Low** — occasional re-map of UID → scene. |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★★ — many readers on one MQTT broker; phone tags unlimited. |

**Mapping:** UID → friendly scene name in HA; tap fires `scene.turn_on` / `media_player.play_media` (e.g., "movie night" tag dims Caseta lights, casts to the V4K65M-08, sets Samsung volume via Broadlink).

**Project similarity:** Same **ESP/ESPHome + MQTT + HA** rail as the **Unified remote (#05)** and **IR lights (#03)** — identical toolchain, just a different input. Triggers the same **HA scenes** as the remote; think of NFC tags as "single-button remotes."

---

## Alternate — ESPHome PN532 readers everywhere (no phone needed)

Fixed 13.56 MHz pads in several rooms. **Cost ~$12/reader; Feasibility ★★★★★; Scalability ★★★★.** Choose if you want hands-free taps for household members without phones.

---

## Verdict

Lowest-priority, highest-whimsy item. Do **phone NFC** in an hour to prove the flow, then add **one or two ESP PN532 pads** at natural tap points. It reuses everything from #03/#05, so marginal effort is tiny. Keep RFID for **convenience scenes**, never security boundaries.
