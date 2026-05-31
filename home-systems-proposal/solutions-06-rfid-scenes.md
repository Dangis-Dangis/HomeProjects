# Solutions — RFID / NFC Scene Triggers

**Problem (P6):** Read RFID/NFC tags; map tags to custom payloads (user ID, scene, hex secret, URL); place many readers around the home; integrate with the automation hub for **scenes, lights, and media** — **not** door unlock (see [solutions-03-home-control.md](./solutions-03-home-control.md)).

**Constraint:** no new cable runs — prefer **phone NFC** or **battery/USB-powered ESP** readers in fixed spots without pulling wire across rooms.

This collection lists every viable option with capabilities, trade-offs, and technical fit, then scores the field and names a recommendation.

---

## Solution A — ESPHome + RC522 (MFRC522) / PN532 + MQTT + automation hub

| | |
|---|---|
| **What it is** | An ESP32 per reader location; ESPHome firmware; the `rc522` or `pn532` component; on scan, publish the UID + optional encoded block to MQTT. |
| **Custom codes** | Store the mapping in a hub `input_text`/`template`, or in the tag's writable sector (MIFARE Classic — weak crypto, OK for home), or encode meaning in the UID only. |
| **Multi-device** | Unique `device_id` per ESP; the same tag works everywhere; automations filter by `device_id` + `uid`. |
| **Cost** | ~$8 ESP32 + $3–8 reader module × N locations; $5–15 per location assembled. |

**Example payload (MQTT)**
```yaml
topic: rfid/kitchen/scan
payload: {"uid":"A1B2C3D4","name":"mom_unlock","device":"kitchen"}
```

**Capabilities**
- Debounce, cooldown, only fire when the UID changes.
- Hub automation: `mqtt` trigger → `scene.turn_on` / `light.turn_on` / `media_player.play_media` / notify.
- OTA firmware via ESPHome; no cloud.

**Tradeoffs**

| Pros | Cons |
|------|------|
| Full control; cheapest scale-out | You wire power (USB/PoE) per reader |
| Custom payloads trivial in YAML | MFRC522 range ~3 cm; placement matters |
| Tight hub integration | MIFARE Classic is not secure for payments |

**Technical fit:** Default for DIY; flashing an ESP32 is no obstacle.

---

## Solution B — ESPHome + Wiegand reader (125 kHz EM4100 / HID prox)

| | |
|---|---|
| **What it is** | Commercial wall readers (26/34-bit Wiegand) → ESP32 `wiegand` component → MQTT. |
| **Custom codes** | Often **facility + card number** in Wiegand bits; map in the hub; or clone tags with known codes. |
| **Cost** | Reader $15–40 + ESP32; tags $1–5 each (TK4100). |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Longer read range; rugged outdoor readers | 125 kHz only; not NFC phone |
| Matches existing access-control hardware | Bit length must match reader firmware |

**Technical fit:** Outdoor-gate scenarios only if prox range is later needed; otherwise prefer Solution E for wireless.

---

## Solution C — Automation hub + PN532 USB (single desk reader)

| | |
|---|---|
| **What it is** | A PN532 USB dongle on the hub host or an RPi; `pn532` integration or MQTT bridge. |
| **Multi-device** | Poor — one USB reader; use for enrollment/programming tags only. |
| **Cost** | ~$15–25 dongle. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Good for writing NDEF / testing tags | Not scalable to many points |

**Technical fit:** A tag-programming bench, not distributed readers.

---

## Solution D — IP-connected access control (ZKTeco / generic MQTT reader)

| | |
|---|---|
| **What it is** | Standalone RFID controllers with Wiegand in, relay out, Ethernet/Wi‑Fi; push events via HTTP/MQTT. |
| **Custom codes** | Device firmware "card number" + user table; export to the hub via webhook. |
| **Cost** | $80–200 per door controller + reader. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Built-in relay for an electric strike | Less hackable; vendor firmware |
| Multi-door centralized | Overkill for light switches |

**Technical fit:** Perimeter access with an electric strike — outside the current convenience-scene scope unless requirements change.

---

## Solution E — NFC NTAG213/215 stickers + phone (no fixed reader)

| | |
|---|---|
| **What it is** | Hub companion-app NFC tag automation; or Tasker; the tag encodes a URL that triggers a webhook. |
| **Custom codes** | NDEF URI `https://hub.example/api/webhook/mom_scene`. |
| **Multi-device** | Any phone reads; not fixed readers. |
| **Cost** | Tags $0.50–2; phones you already have. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Zero wiring | Requires a phone present; not hands-free for kids |
| Easy to reprogram | Not a multi-reader point without a phone |

---

## Solution F — Zigbee RFID keypad (off-the-shelf)

| | |
|---|---|
| **What it is** | Tuya/Zigbee touch keypads with RFID (vendor-specific). |
| **Custom codes** | Limited — vendor-app pairing; expose via Zigbee2MQTT sometimes. |
| **Cost** | $40–80 per keypad. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| No soldering | Opaque payloads; cloud pairing history |

**Technical fit:** Avoid unless you accept vendor lock-in.

---

## Custom-code strategies

| Strategy | Security | Flexibility |
|----------|----------|-------------|
| **UID only** | Low (UID cloneable) | High in hub YAML |
| **Hub lookup table** | Medium (obscurity) | Change mapping without reflashing tags |
| **Writable sector secret** | Medium | Per-tag secret bytes; ESP/hub validates |
| **Rolling counter** | Higher | Complex; rare at home |
| **NTAG password** | Medium | Lock tag memory from casual rewrite |

For **sensitive actions**, prefer UID + hub conditions (time, presence) rather than a secret sector alone.

---

## Multi-device architecture

```
[ESP kitchen] ──┐
[ESP garage]  ──┼── MQTT (Mosquitto) ── Automation hub
[ESP office]  ──┘         │
                          ├─ automation: tag + device → action
                          └─ recorder: optional log of scans
```

**Naming:** `rfid/<location>/tag` topics; retain last UID optional.
**Power:** ESP32 USB at each point; or **PoE** with an ESP32-PoE board ($25) + IoT VLAN.

---

## Comparison matrix

| Solution | Many readers | Custom payload | Hub native | Tag cost | Range |
|----------|--------------|----------------|-----------|----------|-------|
| A ESPHome RC522/PN532 | ★★★★★ | ★★★★★ | ★★★★★ | $1–3 | Short |
| B Wiegand 125 kHz | ★★★★★ | ★★★★ | ★★★★★ | $1–5 | Medium |
| C USB PN532 | ★ | ★★★★ | ★★★★ | $1–3 | Short |
| D IP controller | ★★★★ | ★★★ | ★★★ | $2–10 | Medium |
| E Phone NFC | ★★★ (phones) | ★★★★ | ★★★★★ | $1 | Touch |
| F Zigbee pad | ★★★ | ★★ | ★★★ | incl. | Short |

---

## Parts list (3-point ESPHome build)

| Item | Qty | Unit $ | Total |
|------|-----|--------|-------|
| ESP32-WROOM dev board | 3 | $8 | $24 |
| RC522 module | 3 | $4 | $12 |
| USB power / short cable | 3 | $5 | $15 |
| MIFARE Classic 1K tags (optional) | 10 | $1 | $10 |
| **Total** | | | **~$61** |

Add **PN532** (+$3/module) if you need **NFC** (phone emulation, NTAG).

---

## Automation sketch (hub)

```yaml
automation:
  - alias: RFID movie scene
    trigger:
      - platform: mqtt
        topic: rfid/living/scan
    condition:
      - condition: template
        value_template: "{{ trigger.payload_json.uid == 'A1B2C3D4' }}"
    action:
      - service: scene.turn_on
        target:
          entity_id: scene.movie_night
```

Extend with `choose` on `payload_json.name` for custom symbolic codes.

---

## Recommended path

1. **Start wireless:** hub companion **NFC tags** (Solution E) for scenes — zero hardware.
2. **Fixed readers:** ESPHome + PN532 only where a pad on a shelf makes sense; USB power, no in-wall wiring.
3. **Enrollment:** a hub script maps `uid → scene name` in YAML or an `input_select`.
4. **Security:** IoT VLAN; MQTT auth; use RFID for convenience scenes, not security boundaries.

---

## Scored recommendation

**Top pick — Phone NFC first, add a few ESP readers if you enjoy it.** Start with **zero hardware** (hub companion NFC tags), then sprinkle **ESPHome PN532** readers where a fixed tap point is nice (entry shelf, nightstand, desk).

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$0–60**: NTAG stickers $0.50–2 each (phone-tap = $0 hardware); ESP32 + PN532 ~$12/reader if you add fixed pads. |
| **Setup time** | **1–4 hrs** (NFC tags instant; ESP readers ~30 min each + enrollment). |
| **Maintenance** | **Low** — occasional re-map of UID → scene. |
| **Feasibility** | ★★★★★. |
| **Scalability** | ★★★★ — many readers on one MQTT broker; phone tags unlimited. |

**Mapping:** UID → friendly scene name in the hub; a tap fires `scene.turn_on` / `media_player.play_media` (e.g. a "movie night" tag dims the Caseta lights, casts to the V4K65M-08, and sets Samsung volume via Broadlink).

**Reuse:** the same **ESP/ESPHome + MQTT + hub** rail as the **Unified remote** and the **IR lights** — identical toolchain, just a different input. Triggers the same **hub scenes** as the remote; think of NFC tags as "single-button remotes."

**Alternate — ESPHome PN532 readers everywhere (no phone needed).** Fixed 13.56 MHz pads in several rooms. Cost ~$12/reader; feasibility ★★★★★; scalability ★★★★. Choose if you want hands-free taps for household members without phones.

**Verdict:** The lowest-priority, highest-whimsy item. Do **phone NFC** in an hour to prove the flow, then add **one or two ESP PN532 pads** at natural tap points. It reuses everything from Home control and the Unified remote, so marginal effort is tiny. Keep RFID for **convenience scenes**, never security boundaries.

---

## Full dossier — recommended build (Phone NFC + a few ESPHome PN532 pads)

### Concept image

![Tapping a phone on an NFC wall pad to trigger a lighting scene](./assets/concept-rfid.png)

### Parts list

| Part | Qty | Unit $ | Total |
|------|-----|--------|-------|
| NTAG213/215 NFC stickers (phone-tap, zero fixed hardware) | 10 | $1 | $10 |
| ESP32 dev board (per fixed reader location) | 2 | $8 | $16 |
| PN532 NFC module (13.56 MHz; phone + NTAG capable) | 2 | $7 | $14 |
| USB power adapter + short cable per reader | 2 | $5 | $10 |
| (Optional) small 3D-printed pad enclosure | 2 | $1 | $2 |
| **Total (phone NFC + 2 fixed pads)** | | | **~$52** |

ESPHome, MQTT (Mosquitto), and the hub integration are free.

### Cost example

- **Phone-only:** a pack of NTAG stickers ≈ **$10** total, **$0** fixed hardware.
- **+1 fixed pad:** ≈ **$22** (ESP32 + PN532 + power).
- **3-pad build (RC522, no phone emulation needed):** 3× ESP32 $24 + 3× RC522 $12 + power $15 + tags $10 ≈ **$61** (see *Parts list* table earlier in this document).

### Data path

```
[phone tap on NTAG]          [tap on fixed ESP pad]
   │ NDEF URI                   │ scan → publish
   ▼                            ▼
HA Companion webhook       ESPHome (rc522/pn532) ──► MQTT (Mosquitto)
   └──────────────┬─────────────────────────────────┘
                  ▼
          Home Assistant automation (uid/name + device_id → action)
                  ▼
        scene.turn_on / light.turn_on / media_player.play_media / notify
```

### General wiring / topology diagram

Fixed ESP reader (PN532 over I²C; USB-powered, no in-wall wiring):

```
        ┌──────────── ESP32 dev board ────────────┐
  3V3 ──┤ VCC                                       │
  GND ──┤ GND                                       │── Wi-Fi ──► MQTT ──► Home Assistant
GPIO21 ─┤ SDA ─► PN532 SDA                          │
GPIO22 ─┤ SCL ─► PN532 SCL   (PN532 DIP switch = I²C)│
        └───────────────────────────────────────────┘
   USB-C 5V ─► onboard regulator ─► 3V3   (wall adapter; place pad on a shelf)
```

Multi-pad topology:

```
[ESP kitchen] ┐
[ESP bedside] ┼─ MQTT (Mosquitto) ─ Home Assistant ─ tag+device → scene
[ESP entry]   ┘
```

### Effort to create

| Phase | Effort |
|-------|--------|
| Phone NFC: write NDEF webhook tags, map to scenes | ~1 hr |
| Flash ESPHome (rc522/pn532) per pad; join Wi‑Fi/MQTT | ~30 min/pad |
| Enrollment: map `uid → scene name` in hub (`input_select`/YAML) | 15–30 min |
| **Total** | **~1–4 hrs** |

### Maintenance cost

- **Electricity:** each ESP pad ~0.5–1 W ≈ **$1–2/yr** each; phone tags use no power.
- **Time:** occasional re-map of a UID → scene ≈ **a few minutes, rarely**.
- **Parts:** none recurring; tags are durable.
- **Subscriptions:** none.

### Cost feasibility & scalability

- **Marginal cost per fixed pad ≈ $12–22**; phone tags are **~$1 each** and unlimited.
- **No per-reader fee;** many readers share one MQTT broker and the existing hub.
- **Scales by adding `device_id`s;** the same tag works at every pad. No central controller cost.
- **Takeaway:** start at ~$10, grow a few dollars per tap point — the cheapest add-on in the portfolio.

### Potential risks & downsides

| Risk | Mitigation |
|------|-----------|
| **UID is cloneable — not a security primitive** | Use RFID for convenience scenes only; gate sensitive actions with hub conditions (time/presence) |
| **Phone-tap requires a phone present** | Add fixed ESP pads for hands-free taps (kids/guests) |
| **MFRC522 range ~3 cm; placement-sensitive** | Mount pads at a natural tap height; mark the sweet spot |
| **Reader/tag spoofing on the IoT network** | IoT VLAN + MQTT auth; never wire RFID to locks/strikes |
| **Vendor Zigbee keypads leak pairing history to cloud** | Prefer the ESPHome/phone path; avoid cloud-paired keypads |
