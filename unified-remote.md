# Physical Unified Remote (DIY Control Deck)

**Goal:** A handheld, **battery-powered** physical controller that fires **custom actions** (Home Assistant scenes, TV/audio macros, lights) with **tactile switches, dials, sliders, and buttons**, **status LEDs**, optional **screen**, and **voice input**.

**Your constraints & assets:**
- May or may not include a screen (backlit, RGB array, minimal LCD, or combinations).
- **Status LEDs:** power/automation state, action success/failure.
- Physical **switches, dials (rotary encoders/pots), sliders (faders), buttons** are desired.
- **Reusable dials:** a small set of dials whose function changes with the selected **menu context** (not one dedicated dial per function).
- **3D printer + label maker** available; **EE skills + custom wiring** are fine.
- **Night-viewable**, but lights/screens can **dim and sleep**.
- **Battery powered**; fits **one or two hands**; size flexible—**features and cost** matter more.
- **Voice input** wanted.

**Required menus:**
- **Room:** Living Room / Bedroom
- **Type:** RGB Lights / Ceiling Lights
- **Automation:** Sync TVs / Night Time / Party Time / Cameras On

> This project lives mostly on the **ESP / ESPHome rail** and talks to the same **Home Assistant** hub as your lights/TV/audio (see [device-control-similarity.md](./device-control-similarity.md)). The remote is essentially a *physical front-end* for the macros defined in [home-assistant-integration.md](./home-assistant-integration.md).

![Unified remote concept — front view with screen, three reusable dials, four RGB automation buttons, push-to-talk and status strip](./assets/concept-01-front-reusable.png)

> **Visual concepts:** see [concept-collection.md](./concept-collection.md) for six rendered build directions (Pocket OLED, Standard TFT, Mixer, Puck, Game grip, Wood) — each with a component cost breakdown, pros/cons, and a feature comparison table.

---

## Interaction model — reusable dials + menus

The defining idea: a **few physical dials are reused**, and their meaning is set by the current **Room → Type** context shown on the screen. This keeps the device small while controlling many targets. **Automation** is a separate, context-independent list of one-shot scenes.

### Menu tree

```
HOME
├─ ROOM ........... Living Room | Bedroom          (sets WHERE the dials act)
├─ TYPE ........... RGB Lights  | Ceiling Lights   (sets WHAT the dials act on)
│      └─► reusable dials operate on the selected (Room + Type) target
└─ AUTOMATION ..... Sync TVs | Night Time | Party Time | Cameras On
                    (one-tap HA scripts; ignore Room/Type context)
```

On-screen, the menus look like this:

![ROOM menu — Living Room / Bedroom](./assets/concept-03-screen-room-menu.png)

![TYPE menu — RGB Lights / Ceiling Lights (on/off only)](./assets/concept-04-screen-type-menu.png)

![AUTOMATION menu — Sync TVs / Night Time / Party Time / Cameras On with matching backlit buttons](./assets/concept-05-screen-automation-menu.png)

### How "reusable" works

- **Selectors set context:** pick a **Room** and a **Type** (via dedicated buttons, a selector encoder, or on-screen menu). The screen shows the active context and re-labels each dial ("soft labels").
- **Reusable parameter dials** then act on that target. The **same 2–3 encoders** mean different things per context:

| Context (Room + Type) | Dial 1 | Dial 2 | Dial 3 | Dial press |
|-----------------------|--------|--------|--------|-----------|
| **RGB Lights** | Brightness | Hue / color | Saturation / effect | Toggle on/off |
| **Ceiling Lights** *(non-dimmable, on/off only — Rail 3)* | Select fixture/group | — | — | Toggle on/off |

> Note: your **ceiling lights are non-dimmable on/off** (smart switch / Shelly relay — see [device-control-similarity.md](./device-control-similarity.md)). In that context the dials don't dim; Dial 1 scrolls the fixture/group and the press toggles it. RGB strips (IR→Broadlink or swapped to Zigbee/WLED) are the dimmable/color targets.

### Automation actions (context-independent)

| Button / menu item | Fires (HA) | Touches |
|--------------------|-----------|---------|
| **Sync TVs** | `play_media`/cast same source to both Vizios; set LR volume via Broadlink | Screen share (#04) |
| **Night Time** | Lights off/dim, lower audio, arm cameras, screens→sleep | HA + lights + security |
| **Party Time** | RGB scene + color loop, audio group on, brighten | RGB lights + audio |
| **Cameras On** | Enable Frigate recording / arm detection + notify | Security (#02) |

Automation is best mapped to **4 dedicated backlit buttons** (instant, glanceable RGB state) rather than buried in a menu—so the reusable dials stay focused on Room/Type adjustment.

### Recommended control allocation (Solution A)

![Annotated layout — screen, ROOM/TYPE selectors, three reusable encoders, four automation buttons, push-to-talk, status strip, USB-C](./assets/concept-02-annotated-layout.png)

| Physical control | Role |
|------------------|------|
| **ROOM** button (or selector encoder) | Cycle Living Room ↔ Bedroom |
| **TYPE** button | Cycle RGB ↔ Ceiling |
| **3× reusable encoders** | Context-driven (table above); press to toggle |
| **4× RGB buttons** | Automation: Sync TVs, Night, Party, Cameras |
| **Push-to-talk button + mic** | Voice fallback for anything not on a dial |
| **WS2812 status strip** | Power, "sent", success/failure, HA-reachable |

Because the dials are now **soft-labeled**, a **screen becomes recommended (not optional)** for this model — a 1.3" OLED is enough; a small IPS TFT is nicer for color/room icons.

---

## Connectivity & feedback architecture

### Principle: the remote is a thin Wi-Fi client

The remote **never talks to TVs or lights directly**. It speaks **one protocol — Wi-Fi to Home Assistant** (ESPHome native API) — and HA fans out to each device on its native rail. This keeps firmware simple and is what makes real feedback possible.

```
Remote ──Wi-Fi (ESPHome native API, TCP/acked)──► Home Assistant ──┬─ IP/Cast ──► Vizio V4K65M-08, Chromecast 4K
                                                                   ├─ Wi-Fi→IR ─► Broadlink ──IR──► Samsung audio, 3 IR light zones
                                                                   └─ Zigbee/Wi-Fi ► ceiling smart switches / WLED
```

### How it reaches each target

| Target | Transport | Path | Notes |
|--------|-----------|------|-------|
| **V4K65M-08** (living room) | **Wi-Fi / IP** | Remote→HA `vizio` + `cast` | No BT, no IR needed for the TV itself |
| **Chromecast 4K** (bedroom) | **Wi-Fi / IP** | Remote→HA `cast` + `androidtv` (ADB) | App launch, play, volume |
| **Samsung sound system** | **IR** | Remote→HA→Broadlink RM4→IR | Volume/mute live here (ARC), not on the TV |
| **3 IR light zones** | **IR** | Remote→HA→Broadlink / ESPHome IR node→IR | Line-of-sight per zone |
| **Ceiling lights** | **Zigbee / Wi-Fi** | Remote→HA→smart switch / Shelly | Two-way; on/off |

- **Skip Bluetooth.** Neither the Vizio nor Chromecast exposes useful BT control; BT is one-host, short-range, pairing-fragile. Wi-Fi/IP is strictly better.
- **Keep IR centralized on the Broadlink**, not on the remote. (Optional on-board IR LED only as an *offline* fallback — open-loop and duplicates the blaster.)

### Feedback — two layers

| Layer | Always available? | Drives |
|-------|-------------------|--------|
| **1. Command delivery** | **Yes** — ESPHome native API is TCP and **acknowledged** | blue = sent · green = script ran · red = failed/timeout · amber = HA unreachable |
| **2. Actual device state** | **Only on two-way rails** | True values imported from HA entities |

- **Two-way devices** (Cast TVs, Vizio API, Zigbee switches, WLED): HA holds real state; the deck **imports it** via ESPHome `homeassistant` sensor/`text_sensor` and shows true values. Closed loop.
- **One-way IR devices** (Samsung audio, IR strips): IR is **fire-and-forget**. To truly close the loop, **add a sensor**:
  - **Power-monitoring smart plug** on the Samsung system / IR-light supply → watts rise = actually on.
  - Or **swap the IR strip controller to Zigbee/WLED** for native state (best).
  - Otherwise HA tracks **assumed/optimistic** state — signal it as "assumed," not "confirmed."

### Expanded status-LED array (one LED per zone)

Dedicate **one addressable RGB LED per tracked entity** for at-a-glance state. For **open-loop IR devices, back the LED with a power sensor** so it reflects *measured reality*.

| LED | Tracks | Off / Idle | On / OK | Problem |
|-----|--------|-----------|---------|---------|
| LR RGB | living room strip | dim | color of strip | red = unreachable |
| Ceiling | ceiling switch | dim | white | — |
| Samsung audio | power-sense plug | dim | green | amber = assumed (no sensor) |
| Cameras | Frigate armed | dim | green | red = recording error |
| Sync | TVs casting | dim | blue | amber = one TV only |
| HA link | API connection | — | green | amber/red = offline |

### On-screen action descriptions (toast + log)

The screen shows **transient per-step results** so IR failures aren't silent:

```
Night Time
  ✓ 6 lights off
  ✓ cameras armed
  ⚠ Samsung audio (assumed — no sensor)
```

**Division of labor:** LEDs = persistent at-a-glance zone state; screen = transient narrative + values + error detail.

**Battery caveat:** more always-on RGB costs runtime — keep LEDs **dim, low brightness, and auto-sleeping with the screen**. A handful at low duty is negligible; dozens at full brightness will hurt.

### Feedback sketch (ESPHome ↔ HA)

```yaml
# Import a real HA entity state so the deck reflects truth (two-way devices)
text_sensor:
  - platform: homeassistant
    id: lr_rgb_state
    entity_id: light.living_room_rgb     # "on"/"off" + attributes

# For IR devices, import a power-sensing plug instead of trusting the IR send
sensor:
  - platform: homeassistant
    id: samsung_watts
    entity_id: sensor.samsung_audio_power # >5 W ⇒ actually on
```

HA pushes these to the deck; an on-device automation maps them to the per-zone LEDs and the on-screen log.

---

## Solution A — ESP32-S3 + ESPHome custom deck (recommended)

| | |
|---|---|
| **What it is** | A custom board (perfboard → custom PCB) around an **ESP32‑S3**: mechanical/arcade **buttons**, **rotary encoders** (dials), **slide pots** (faders), toggle **switches**, **WS2812 RGB** status LEDs, optional **OLED/TFT**, **I²S mic** (INMP441) for voice, **LiPo + charger**. Runs **ESPHome**; native Home Assistant API. |
| **Voice** | ESPHome **Voice Assistant** with on-device **microWakeWord**; audio streams to **HA Assist** (Whisper STT + intents). **Push‑to‑talk** button strongly recommended to protect battery. |
| **Custom actions** | Each control maps to an ESPHome automation → HA service call (scene, script, `media_player`, IR/Broadlink). **Reusable dials** read the current Room/Type context (global vars) and call the matching service. |
| **Status LEDs** | WS2812: a "command" strip (sent/success/failure/HA-link) **plus a per-zone LED array** (one LED per tracked entity — LR RGB, ceiling, Samsung audio, cameras, sync) driven by real HA state / power sensors. See *Connectivity & feedback*. |
| **Screen** | **Recommended** for this build (soft dial labels + context): 1.3" OLED (low power) or small IPS TFT for room/type icons and live values. |
| **Battery** | 18650 or LiPo pouch 2000–3500 mAh; **deep sleep** wake-on-touch/button; weeks of standby, ~1–2 days if always-listening. |
| **Cost** | **~$45–95** in parts (see BOM). |

**Capabilities**
- Fully local control via HA API/MQTT; OTA firmware.
- **Reusable encoders** re-mapped by Room/Type context; faders for fine levels; toggles/buttons for modes and Automation.
- Per-button RGB + on-screen soft labels; label maker for static legends (ROOM, TYPE, automation names).

**Tradeoffs**
| Pros | Cons |
|------|------|
| Native HA, no cloud; cheap; infinitely customizable | You design power/sleep + wiring |
| Reuses your EE + 3D printing skills | Always-on voice wake word drains battery → use push-to-talk |
| Status feedback loop (success/fail LEDs) | ESPHome voice is good but server-side STT needs HA Assist set up |

**Technical fit:** Best balance of cost, control, and HA-native integration. **Primary recommendation.**

---

## Solution B — M5Stack modular (Core2 / Dial / Cardputer + units)

| | |
|---|---|
| **What it is** | Off-the-shelf **ESP32** maker modules with **screen + battery** built in: **M5Dial** (rotary + round LCD), **Core2** (touch LCD), **Cardputer** (keyboard); add mic/IR/button units. Flash **ESPHome** or M5 firmware. |
| **Voice** | M5 mic unit + ESPHome voice, or echo-style modules. |
| **Cost** | **~$30–120** depending on modules. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Battery/screen/enclosure already solved | Less tactile (few real dials/faders) |
| Fast to working prototype | Fixed form factor; fewer physical controls |
| ESPHome-compatible | Modules add up in cost |

**Technical fit:** Fast path / prototype before committing to a custom PCB; or if you want a screen-first device with minimal soldering.

---

## Solution C — Stream Deck + Companion (desk reference, not your spec)

| | |
|---|---|
| **What it is** | Elgato Stream Deck (LCD keys) + **Bitfocus Companion** → HA. |
| **Cost** | **$80–250**. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Polished LCD buttons; reliable | **USB-wired, no battery**, **no voice**, no dials/faders |
| Zero firmware work | Doesn't meet battery/voice/tactile goals |

**Technical fit:** A wired control surface for a desk—listed for completeness; **fails your battery/voice/handheld requirements.**

---

## Solution D — FreeTouchDeck / ESP32 "Cheap Yellow Display" (touch-first)

| | |
|---|---|
| **What it is** | ESP32 + TFT touchscreen (CYD ~$15) running **FreeTouchDeck**/ESPHome; soft buttons + a few hardware buttons/encoders bolted on. |
| **Cost** | **~$20–50**. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Cheapest screen-based deck | Touch ≠ tactile dials/faders |
| Good for icon grids | Battery/power must be added |

**Technical fit:** Touch-centric, low cost; combine with a few real encoders for the tactile feel.

---

## Solution E — QMK/ZMK macropad (mechanical-keyboard rail)

| | |
|---|---|
| **What it is** | A custom **mechanical** controller using **ZMK** (wireless BLE) or **QMK** (USB): Cherry-MX switches, rotary encoders, OLED, per-key RGB. ZMK gives **battery + BLE**. Bridges to HA via a host/BLE-HID receiver or companion. |
| **Cost** | **~$60–200** (nice switches/PCB). |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Best mechanical feel; mature RGB/OLED/encoder support | HID-centric → needs a bridge to reach HA cleanly |
| ZMK = great battery life | No native voice; faders/sliders less common |

**Technical fit:** If keyboard-grade switch feel matters most and you accept a HID→HA bridge.

---

## Solution F — Raspberry Pi handheld (max compute / offline voice)

| | |
|---|---|
| **What it is** | Pi Zero 2 W / Pi 4 + tou/screen + GPIO controls; can run **offline** wake word + local STT, full apps. |
| **Cost** | **~$60–150**. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Offline voice possible; powerful | Worse battery life, bulkier, hotter |
| Run anything (browser, RTSP preview) | Overkill for sending HA commands |

**Technical fit:** Only if you want on-device/offline voice or a rich screen UI in-hand.

---

## Voice input options (any solution)

| Approach | Where STT runs | Battery impact | Notes |
|----------|----------------|----------------|-------|
| **ESPHome VA + microWakeWord → HA Assist** | HA server (Whisper) | Push‑to‑talk: low / Always‑on: high | Best HA-native path (Solution A/B/D) |
| **HA Voice PE / Atom Echo as separate puck** | HA server | n/a (own power) | Offload voice to a mains-powered satellite; remote stays button-only |
| **On-device offline (Pi + whisper.cpp)** | On device | High | Only Solution F |

**Recommendation:** **push‑to‑talk** (a dedicated button starts streaming) — preserves battery and avoids false wakes. Consider a **separate always-listening voice puck** for the room so the handheld can sleep aggressively.

---

## Reference BOM — Solution A (rich tactile deck)

![Exploded view — 3D-printed faceplate, display, ESP32-S3 PCB, encoders, WS2812 strip, mic, LiPo, TP4056 charger, bottom shell](./assets/concept-11-exploded-internals.png)

| Part | Qty | Unit $ | Total |
|------|-----|--------|-------|
| ESP32‑S3 dev board (PSRAM) | 1 | $10 | $10 |
| Rotary encoders w/ push | 3 | $1.50 | $5 |
| Slide potentiometers (faders) | 2 | $3 | $6 |
| Mechanical/arcade buttons | 8 | $1 | $8 |
| Toggle/rocker switches | 3 | $1 | $3 |
| WS2812 RGB (command strip + per-zone array) | 18–24 px | — | $5 |
| Power-sensing smart plug *(external/optional — closes IR loop)* | 1–2 | $12 | +$12–24 |
| 1.3" OLED (SSD1306/SH1106) | 1 | $5 | $5 |
| INMP441 I²S mic | 1 | $4 | $4 |
| LiPo 3500 mAh + TP4056 + boost | 1 | $14 | $14 |
| 3D-printed shell + filament | — | — | $4 |
| Misc (wire, headers, screws) | — | — | $5 |
| **Total (handheld)** | | | **~$69** |

The **power-sensing plug(s)** live in HA (not in the handheld) and are **optional** — they exist to close the feedback loop on the one-way IR devices (Samsung audio, IR strips). Add a small **IPS TFT** (+$8) instead of OLED for color labels; subtract faders/encoders to cut cost toward ~$45.

---

## Wiring & GPIO pinout (Solution A, ESP32‑S3‑WROOM‑1)

> This is the **authoritative** mapping (the generated diagrams are visual references only). Pins chosen to avoid strapping pins (0, 3, 45, 46), USB (19/20), UART debug (43/44), and octal-PSRAM pins (26–32 on N16R8 modules).

### Pin map

| Function | Signal | GPIO | Notes |
|----------|--------|------|-------|
| **Encoder 1** | A / B / SW | 4 / 5 / 6 | `INPUT_PULLUP`; SW = push |
| **Encoder 2** | A / B / SW | 7 / 15 / 16 | |
| **Encoder 3** | A / B / SW | 17 / 18 / 8 | |
| **ROOM button** | SW | 9 | pull-up |
| **TYPE button** | SW | 10 | pull-up |
| **Automation btns ×4** | SW | 11 / 12 / 13 / 14 | Sync / Night / Party / Cameras |
| **Push-to-talk** | SW | 21 | wakes mic stream |
| **WS2812 (command + zone array)** | DIN | 38 | 5 V data; level-shift if noisy |
| **INMP441 mic (I²S)** | BCLK / WS / SD | 40 / 41 / 42 | SEL→GND (left ch) |
| **Display (ST7789 SPI)** | SCLK / MOSI / CS / DC / RST / BLK | 36 / 35 / 34 / 33 / 47 / 48 | OLED users: swap to I²C SDA=8 / SCL=9 and free encoder pins |
| **Battery sense** | ADC | 1 | ADC1_CH0 via 2×100 kΩ divider |

### Power subsystem

```
USB-C ──► TP4056/IP5306 charger ──► LiPo 3.7V 3500 mAh ──► 3.3 V LDO/buck ──► ESP32-S3 3V3
                                          │
                                          └► ADC divider ──► GPIO1 (battery %)
WS2812 + INMP441 run on 3V3 (or 5V boost for LED strip; common GND with ESP32)
```

- **Common ground** across ESP32, encoders, buttons, mic, LEDs, charger.
- WS2812 are happiest at 5 V; at 3.3 V data they usually work for a short run — add a **74AHCT125 level shifter** if you see flicker on the longer zone array.
- Use **deep sleep + wake-on-interrupt** (PTT/ROOM button on an RTC-capable pin) for battery life.

### Logical wiring overview

```
                 ┌───────────────────────────┐
   3× Encoders ──┤ GPIO4-8,15-18             │
 ROOM/TYPE btn ──┤ GPIO9,10                  │
 4× Auto btns  ──┤ GPIO11-14                 │
   PTT button  ──┤ GPIO21          ESP32-S3  │── Wi-Fi ──► Home Assistant
  WS2812 array ──┤ GPIO38 (DIN)              │
   INMP441 mic ──┤ GPIO40-42 (I²S)           │
  ST7789 TFT   ──┤ GPIO33-36,47,48 (SPI)     │
  Batt divider ──┤ GPIO1 (ADC)               │
                 └──────────┬────────────────┘
                            │ 3V3 / GND
            USB-C ─► TP4056 ─► LiPo ─► LDO/buck
```

### Generated diagrams

**Whole-device overview** — all peripherals around the ESP32‑S3:

![Wiring overview — ESP32-S3 with encoders, buttons, display, mic, WS2812, TP4056 and LiPo](./assets/wiring-01-system-overview.png)

**GPIO pinout map** (visual reference):

![ESP32-S3 GPIO pinout map by function](./assets/wiring-02-gpio-pinout.png)

**Power subsystem** — USB-C → TP4056 → LiPo → 3V3 with ADC battery divider:

![Power subsystem wiring: USB-C, TP4056 charger, LiPo, 3.3V regulator, battery sense divider](./assets/wiring-03-power-subsystem.png)

**Inputs detail** — encoders + buttons on shared 3V3/GND buses with pull-ups:

![Input wiring detail: three rotary encoders and six buttons on shared power and ground rails](./assets/wiring-04-inputs-detail.png)

> Diagrams are **visual references**; if a pin label differs, the **pin map table above is authoritative** (it avoids strapping/USB/PSRAM pins).

---

## Reusable-dial context (ESPHome sketch)

```yaml
# Two globals hold the current menu context
globals:
  - id: room          # 0 = Living Room, 1 = Bedroom
    type: int
    restore_value: yes
  - id: light_type    # 0 = RGB, 1 = Ceiling
    type: int

# ROOM / TYPE selector buttons cycle the context and refresh the screen
binary_sensor:
  - platform: gpio
    pin: GPIO5
    name: "Sel Room"
    on_press:
      - lambda: 'id(room) = (id(room) + 1) % 2;'
      - component.update: deck_display

  - platform: gpio
    pin: GPIO6
    name: "Sel Type"
    on_press:
      - lambda: 'id(light_type) = (id(light_type) + 1) % 2;'
      - component.update: deck_display

# Reusable Dial 1 = Brightness/Hue (RGB) or fixture-select (Ceiling)
sensor:
  - platform: rotary_encoder
    id: dial1
    pin_a: GPIO7
    pin_b: GPIO8
    on_clockwise:
      - homeassistant.event:
          event: esphome.deck_dial
          data:
            dial: "1"
            dir: "up"
            room: !lambda 'return id(room);'
            type: !lambda 'return id(light_type);'
```

HA then routes `esphome.deck_dial` with a `choose:` on `room`/`type` to the right target (RGB `light.turn_on` brightness/hue vs. ceiling group/fixture toggle). The screen (`deck_display`) renders the live soft labels for each dial.

---

## Status-LED + success/failure pattern (ESPHome ↔ HA)

```yaml
# ESPHome: button press → HA event; HA replies with result → LED feedback
binary_sensor:
  - platform: gpio
    pin: GPIO4
    name: "Deck Btn SyncTVs"
    on_press:
      - homeassistant.event:
          event: esphome.deck_action
          data: {action: "sync_tvs"}   # Automation buttons ignore Room/Type

# In HA: script runs, then sets a text/RGB helper the deck subscribes to:
# light.turn_on deck_status → green flash on success, red on failure (choose/try)
```

LED states: **dim idle**, **blue pulse = sent**, **green = success**, **red = failure/timeout**, **amber = HA unreachable**.

---

## Comparison matrix

| Solution | Tactile (dials/faders) | Screen | Voice | Battery-native | HA-native | Build effort | Cost |
|----------|------------------------|--------|-------|----------------|-----------|--------------|------|
| A ESP32‑S3 ESPHome | ★★★★★ | optional | ★★★★ (PTT) | ★★★★ | ★★★★★ | High | $45–95 |
| B M5Stack | ★★ | ★★★★★ | ★★★★ | ★★★★★ | ★★★★ | Low | $30–120 |
| C Stream Deck | ✗ | ★★★★★ | ✗ | ✗ | ★★★★ | Very low | $80–250 |
| D FreeTouchDeck/CYD | ★★ | ★★★★ | ★★★ | ★★ | ★★★★ | Low–Med | $20–50 |
| E QMK/ZMK | ★★★★★ | ★★★ | ✗ | ★★★★ (ZMK) | ★★ (bridge) | Med–High | $60–200 |
| F Raspberry Pi | ★★★ | ★★★★ | ★★★★★ (offline) | ★★ | ★★★ | Med | $60–150 |

---

## Recommended path

1. **Prototype on M5Stack/CYD** (Solution B/D) to validate the **Room/Type context + reusable dials** and voice flow quickly (~a weekend); the screen makes this model easy to test before soldering.
2. **Build Solution A** as the keeper: ESP32‑S3 + **3 reusable encoders** + **ROOM/TYPE selector buttons** + **4 Automation RGB buttons** + push‑to‑talk + WS2812 + OLED/TFT + I²S mic + 3500 mAh LiPo, 3D-printed shell, label-maker legends.
3. **Voice = push‑to‑talk** to HA Assist; optionally add a mains-powered **voice puck** so the handheld sleeps.
4. **Power discipline:** deep sleep + wake-on-interrupt; USB-C charging; LED/OLED auto-dim schedule for night.
5. **Feedback loop:** wire success/failure RGB from HA script results so every action confirms physically.

**Maintenance:** OTA ESPHome updates; recharge weekly (or per-use if voice-heavy); re-map buttons in YAML as your scenes evolve.
