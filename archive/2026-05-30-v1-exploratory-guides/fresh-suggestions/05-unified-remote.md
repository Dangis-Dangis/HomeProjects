# 05 — Physical Unified Remote (Fresh Suggestion)

**Need:** Battery-powered handheld with **buttons/dials/sliders/switches**, **status LEDs**, **screen**, **voice input**, firing **custom HA actions**; night-viewable with dim/sleep; one–two hands. **Reusable dials** re-mapped by a **Room** (Living Room/Bedroom) → **Type** (RGB/Ceiling) menu context, plus an **Automation** menu (Sync TVs / Night Time / Party Time / Cameras On). You have EE skills, custom wiring, 3D printer, label maker.

Full option list: [../unified-remote.md](../unified-remote.md)

---

## Top pick — ESP32‑S3 + ESPHome custom deck (with push‑to‑talk voice)

A custom board with real tactile controls, WS2812 status LEDs (success/failure feedback), optional OLED, I²S mic for **push‑to‑talk** voice → HA Assist, LiPo + deep sleep. Native HA, fully yours.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$45–95** parts (ESP32‑S3 $10, 3 encoders $5, 2 faders $6, 8 buttons $8, WS2812 $4, OLED $5, I²S mic $4, LiPo+charger $14, printed shell + misc $9). TFT screen +$8. |
| **Setup time** | **2–4 days** (wiring/soldering, ESPHome YAML, enclosure print/iterate, voice + LED feedback). |
| **Maintenance** | **Low–Med** — OTA firmware; recharge weekly (or per-use if voice-heavy); re-map buttons as scenes evolve. |
| **Feasibility** | ★★★★★ — ideal match for your skills/tools; the one risk is **always-on voice battery drain** → use push‑to‑talk. |
| **Scalability** | ★★★★★ — add controls/pages/LEDs in YAML; build multiples for other rooms; same firmware pattern. |

**Voice:** ESPHome Voice Assistant + microWakeWord → **HA Assist** (server-side Whisper/intents). Push‑to‑talk button preserves battery; optionally offload always-listening to a mains-powered voice puck so the handheld sleeps hard.

**Reusable dials:** Room/Type selectors set context; the same 3 encoders then control the chosen target (RGB: brightness/hue/sat; Ceiling: fixture-select + toggle, since they're non-dimmable). A small **screen is now recommended** for the soft dial labels. Automation lives on 4 dedicated RGB buttons (context-independent).

**Status LEDs:** dim idle · blue = sent · **green = success** · **red = failure** · amber = HA unreachable (HA returns the result to the deck).

**Project similarity:** Pure **ESP/ESPHome rail** — same toolchain as the **IR lights** and **RFID readers** (HA #03, RFID #06). It's a **physical front-end for the HA macros** built in #03/#04, and can fire **IR (Broadlink) macros** and **Cast** actions. Voice reuses the **HA Assist** you may already enable.

---

## Alternate A — M5Stack / CYD prototype first

Off-the-shelf ESP32 + screen + battery (M5Dial/Core2 or Cheap Yellow Display). **Cost $20–120; Setup ~a weekend; Feasibility ★★★★★; Scalability ★★★.** Use it to **validate macros + voice flow fast**, then graduate to the custom deck. Fewer real dials/faders.

## Alternate B — ZMK mechanical macropad

Keyboard-grade switches, encoders, OLED, BLE + battery. **Cost $60–200; Feasibility ★★★★ (needs HID→HA bridge); Scalability ★★★★.** Pick only if switch *feel* outranks HA-native simplicity and voice (no native voice).

---

## Verdict

**Prototype on M5Stack/CYD this weekend**, then build the **ESP32‑S3 ESPHome deck** as the keeper once your HA scenes are stable (build order #6). Commit to **push‑to‑talk voice** and a real **success/failure LED loop** — that feedback is what makes a physical remote feel better than a phone. Cheapest high-craft project in the portfolio.
