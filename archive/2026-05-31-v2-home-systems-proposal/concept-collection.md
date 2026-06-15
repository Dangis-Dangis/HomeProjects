# Unified Remote — Concept Collection

Six **buildable** design directions for the [unified remote](./unified-remote.md), each implementing the current spec: **reusable dials** re-mapped by a **Room → Type** menu context, an **Automation** set (Sync TVs / Night / Party / Cameras On), **status LEDs** (power / success / failure), **push-to-talk voice**, and **battery** power.

Each concept lists a **component cost breakdown**, **advantages / disadvantages**, and the page ends with a **feature comparison table**.

**Common assumptions:** ESP32‑S3 + ESPHome, Home Assistant API, USB‑C LiPo charging, 3D-printed enclosure, label-maker legends. Prices are rough 2025–2026 hobby-quantity USD; exclude tools and filament you already own.

---

## 1. Pocket OLED (budget / one-handed)

![Pocket OLED concept](./assets/collection-1-pocket.png)

Smallest, cheapest build: a monochrome OLED, **2 reusable dials**, and the 4 automation buttons. Context shown as text ("ROOM: Living  TYPE: RGB").

**Component cost breakdown**

| Component | Part | Cost |
|-----------|------|------|
| MCU | ESP32‑S3 mini board | $7 |
| Screen | 0.96" OLED (SSD1306) | $4 |
| Reusable dials | 2× rotary encoder w/ push | $3 |
| Selectors | ROOM/TYPE tactile buttons | $1 |
| Automation | 4× tactile + 4× WS2812 | $4 |
| Status LED | WS2812 strip (6 px) | $2 |
| Voice | INMP441 I²S mic | $4 |
| Power | LiPo 1200 mAh + TP4056 | $9 |
| Enclosure | 3D print + hardware | $6 |
| **Total** | | **~$40** |

**Advantages**
- Cheapest and fastest to build; truly pocketable / one-handed.
- Low power → longest standby on a small cell.
- Great first prototype to validate the Room/Type flow.

**Disadvantages**
- Monochrome text only — no color swatches or icons.
- Only 2 dials → tighter mapping (e.g., brightness + hue, no separate saturation).
- Cramped layout; label-maker tags do a lot of work.

---

## 2. Standard TFT deck (recommended)

![Standard TFT deck concept](./assets/collection-2-standard.png)

The balanced build and the one [unified-remote.md](./unified-remote.md) Solution A targets: a **1.9" IPS** screen with soft dial labels, **3 reusable encoders**, ROOM/TYPE selectors, 4 RGB automation buttons, push-to-talk.

**Component cost breakdown**

| Component | Part | Cost |
|-----------|------|------|
| MCU | ESP32‑S3 (8 MB PSRAM) | $10 |
| Screen | 1.9" IPS TFT (ST7789) | $9 |
| Reusable dials | 3× rotary encoder w/ push | $5 |
| Selectors | ROOM/TYPE buttons | $1 |
| Automation | 4× RGB arcade buttons | $6 |
| Status LED | WS2812 strip (12 px) | $4 |
| Voice | INMP441 I²S mic | $4 |
| Power | LiPo 3500 mAh + TP4056 + boost | $14 |
| Enclosure | 3D print + hardware | $9 |
| **Total** | | **~$62** |

**Advantages**
- Color soft-labels make reusable dials self-explanatory.
- 3 dials cleanly cover RGB brightness/hue/saturation.
- Comfortable two-hand ergonomics; best all-round value.

**Disadvantages**
- More wiring than the pocket build.
- No faders for very fine level control.
- Mid battery; always-on voice still needs push-to-talk.

---

## 3. Mixer console (max controls)

![Mixer console concept](./assets/collection-3-mixer.png)

Widescreen "control desk": adds **2 slide faders** alongside 3 encoders and extra selector buttons. For when you want a dedicated physical level for everything.

**Component cost breakdown**

| Component | Part | Cost |
|-----------|------|------|
| MCU | ESP32‑S3 (8 MB PSRAM) | $10 |
| Screen | 2.4" IPS TFT | $12 |
| Reusable dials | 3× rotary encoder | $5 |
| Faders | 2× 60 mm slide pots | $8 |
| Selectors | 6–8 buttons (rooms/types/scenes) | $8 |
| Automation | 4× RGB buttons | $6 |
| Status LED | WS2812 strip (16 px) | $5 |
| Voice | INMP441 I²S mic | $4 |
| Power | LiPo 5000 mAh + charger + boost | $18 |
| Enclosure | larger 3D print + hardware | $12 |
| **Total** | | **~$88** |

**Advantages**
- Most simultaneous controls; faders give instant absolute levels.
- Big battery → long runtime even with the larger screen.
- Excellent for power-users / multi-room "scene mixing."

**Disadvantages**
- Largest, least portable; lives on a coffee table or desk.
- More parts = more soldering and more to go wrong.
- Highest cost in the set.

---

## 4. Thermostat puck (single-dial)

![Thermostat puck concept](./assets/collection-4-puck.png)

Circular, **one large reusable dial** with a round color screen and an RGB rim. Minimalist; spin to adjust, edge buttons for automations.

**Component cost breakdown**

| Component | Part | Cost |
|-----------|------|------|
| MCU | ESP32‑S3 | $10 |
| Screen | 1.28" round IPS (GC9A01) | $9 |
| Reusable dial | 1× quality rotary encoder | $6 |
| Selectors | ROOM/TYPE + 4 edge buttons | $4 |
| Status LED | WS2812 ring (16–24 px) | $6 |
| Voice | INMP441 I²S mic | $4 |
| Power | LiPo 2000 mAh + TP4056 | $10 |
| Enclosure | round 3D print + hardware | $8 |
| **Total** | | **~$57** |

**Advantages**
- Cleanest aesthetic; sits nicely on a side table.
- One-dial interaction is intuitive (Nest-like).
- RGB rim is a beautiful glanceable status indicator.

**Disadvantages**
- Single dial → more menu-stepping to reach each parameter.
- Round screen wastes corner space; layout is fiddly.
- Less "tactile variety" (no faders/toggles).

---

## 5. Game grip (ergonomic, two-thumb)

![Game-controller grip concept](./assets/collection-5-grip.png)

Controller form with a **dial under each thumb**, shoulder ROOM/TYPE buttons, and face buttons for automations. Best in-hand ergonomics for couch use.

**Component cost breakdown**

| Component | Part | Cost |
|-----------|------|------|
| MCU | ESP32‑S3 | $10 |
| Screen | 3.0" IPS TFT | $14 |
| Reusable dials | 2× thumb rotary encoders | $4 |
| Selectors | 2 shoulder + 4 face buttons | $5 |
| Status LED | WS2812 light bar (10 px) | $4 |
| Voice | INMP441 I²S mic | $4 |
| Power | 4000 mAh LiPo + charger + boost | $16 |
| Enclosure | ergonomic 3D print (more filament) | $14 |
| **Total** | | **~$71** |

**Advantages**
- Most comfortable for long couch sessions; natural thumb reach.
- Big screen + light bar; great for the "Sync TVs / movie" use case.
- Buttons + dials within reach without looking.

**Disadvantages**
- Bulkiest enclosure; most print time and iteration to get grips right.
- Only 2 dials; relies on menu context switching.
- Heaviest to hold one-handed.

---

## 6. Premium wood (showpiece)

![Premium wood concept](./assets/collection-6-wood.png)

A display-worthy build: walnut shell, brushed-aluminum faceplate, machined knobs. Same electronics as the standard deck, elevated finish.

**Component cost breakdown**

| Component | Part | Cost |
|-----------|------|------|
| MCU | ESP32‑S3 (8 MB PSRAM) | $10 |
| Screen | 2.4" IPS TFT | $12 |
| Reusable dials | 3× encoder + machined alu knobs | $14 |
| Selectors | ROOM/TYPE buttons | $2 |
| Automation | 4× RGB buttons | $6 |
| Status LED | WS2812 strip (12 px) | $4 |
| Voice | INMP441 I²S mic | $4 |
| Power | LiPo 3500 mAh + charger + boost | $14 |
| Enclosure | walnut blank + finish + 3D parts | $20 |
| Misc | hardware, threaded inserts | $6 |
| **Total** | | **~$92** |

**Advantages**
- Looks like a finished consumer product; living-room friendly.
- Premium knob feel; metal faceplate is durable.
- Same proven electronics as Concept 2 (low technical risk).

**Disadvantages**
- Most expensive; woodworking adds time and tools.
- Wood/metal complicate antenna placement — keep ESP32 antenna near a non-metal window.
- Repairs/mods are harder once finished.

---

## Feature comparison

| Feature | 1 Pocket | 2 Standard ⭐ | 3 Mixer | 4 Puck | 5 Grip | 6 Wood |
|---------|:--------:|:------------:|:-------:|:------:|:------:|:------:|
| **Cost (parts)** | ~$40 | ~$62 | ~$88 | ~$57 | ~$71 | ~$92 |
| **Screen** | 0.96" OLED mono | 1.9" IPS | 2.4" IPS | 1.28" round | 3.0" IPS | 2.4" IPS |
| **Reusable dials** | 2 | 3 | 3 | 1 (large) | 2 (thumb) | 3 |
| **Faders** | – | – | 2 | – | – | – |
| **Automation buttons** | 4 | 4 | 4 | 4 (edge) | 4 (face) | 4 |
| **Voice (PTT)** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Status LEDs** | 6 px | 12 px | 16 px | ring | bar | 12 px |
| **Battery** | 1200 mAh | 3500 mAh | 5000 mAh | 2000 mAh | 4000 mAh | 3500 mAh |
| **Est. standby** | weeks | ~2–3 wk | ~3–4 wk | ~2 wk | ~3 wk | ~2–3 wk |
| **Ergonomics** | 1-hand | 2-hand | desk | table | 2-hand couch | 2-hand/desk |
| **Build effort** | Low | Medium | High | Medium | High | High |
| **Portability** | ★★★★★ | ★★★★ | ★★ | ★★★★ | ★★★ | ★★★ |
| **Tactile variety** | ★★ | ★★★★ | ★★★★★ | ★★ | ★★★ | ★★★★ |
| **Aesthetic / WAF** | ★★ | ★★★ | ★★★ | ★★★★★ | ★★★ | ★★★★★ |
| **Best for** | First build / EDC | Daily driver | Power user | Side table | Couch / TV | Showpiece |

⭐ = recommended starting point. A common path: prototype on **Concept 1**, build **Concept 2** as the daily driver, then re-shell as **Concept 6** if you want the showpiece finish.

Runtime estimates assume push-to-talk voice and deep-sleep with wake-on-interrupt; always-listening wake word reduces these to ~1–2 days regardless of concept.
