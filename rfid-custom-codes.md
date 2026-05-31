# RFID with Custom Codes & Multiple Devices

**Goals:** Read RFID/NFC tags; map tags to custom payloads (user ID, scene, hex secret, URL); many readers around the home; integrate with Home Assistant for **scenes, lights, and media**—not door unlock (see [home-assistant-integration.md](./home-assistant-integration.md)).

**Constraint:** **No new cable runs**—prefer **phone NFC** or **battery/USB-powered ESP** readers in fixed spots without pulling wire across rooms.

---

## Solution A — ESPHome + RC522 (13.56 MHz-ish MFRC522) / PN532 + MQTT + Home Assistant

| | |
|---|---|
| **What it is** | ESP32 per reader location; ESPHome firmware; `rc522` or `pn532` component; on scan publish UID + optional encoded block to MQTT. |
| **Custom codes** | Store mapping in HA `input_text`/`template` or in tag’s writable sector (MIFARE Classic—weak crypto, OK for home); or encode meaning in UID only. |
| **Multi-device** | Unique `device_id` per ESP; same tag works everywhere; automations filter by `device_id` + `uid`. |
| **Cost** | ~$8 ESP32 + $3–8 reader module × N locations; $5–15 per location assembled. |

**Example payload (MQTT)**
```yaml
topic: rfid/kitchen/scan
payload: {"uid":"A1B2C3D4","name":"mom_unlock","device":"kitchen"}
```

**Capabilities**
- Debounce, cooldown, only fire when UID changes.
- HA automation: `mqtt` trigger → `scene.turn_on` / `light.turn_on` / `media_player.play_media` / notify.
- OTA firmware via ESPHome; no cloud.

**Tradeoffs**
| Pros | Cons |
|------|------|
| Full control; cheapest scale-out | You wire power (USB/PoE) per reader |
| Custom payloads trivial in YAML | MFRC522 range ~3 cm; placement matters |
| Tight HA integration | MIFARE Classic not secure for payments |

**Technical fit:** Default for DIY; you’re fine flashing ESP32.

---

## Solution B — ESPHome + Wiegand reader (125 kHz EM4100 / HID prox)

| | |
|---|---|
| **What it is** | Commercial wall readers (26/34-bit Wiegand) → ESP32 `wiegand` component → MQTT. |
| **Custom codes** | Often **facility + card number** in Wiegand bits; map in HA; or clone tags with known codes. |
| **Cost** | Reader $15–40 + ESP32; tags $1–5 each (TK4100). |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Longer read range; rugged outdoor readers | 125 kHz only; not NFC phone |
| Matches existing access control hardware | Bit length must match reader firmware |

**Technical fit:** Outdoor gate scenarios only if you later need prox range; otherwise prefer Solution E for wireless.

---

## Solution C — Home Assistant + PN532 USB (single desk reader)

| | |
|---|---|
| **What it is** | PN532 USB dongle on HA host or RPi; `pn532` integration or MQTT bridge. |
| **Multi-device** | Poor—one USB reader; use for enrollment/programming tags only. |
| **Cost** | ~$15–25 dongle. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Good for writing NDEF / testing tags | Not scalable to many doors |

**Technical fit:** Tag programming bench, not distributed readers.

---

## Solution D — IP-connected access control (ZKTeco / generic MQTT reader)

| | |
|---|---|
| **What it is** | Standalone RFID controllers with Wiegand in, relay out, Ethernet/Wi‑Fi; push events via HTTP/MQTT. |
| **Custom codes** | Device firmware “card number” + user table; export to HA via webhook. |
| **Cost** | $80–200 per door controller + reader. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Built-in relay for electric strike | Less hackable; vendor firmware |
| Multi-door centralized | Overkill for light switches |

**Technical fit:** Perimeter access with electric strike—outside current home hobby scope unless requirements change.

---

## Solution E — NFC NTAG213/215 stickers + phone (no fixed reader)

| | |
|---|---|
| **What it is** | HA Companion app NFC tag automation; or Tasker; tag encodes URL to trigger webhook. |
| **Custom codes** | NDEF URI `https://ha.example/api/webhook/mom_scene` |
| **Multi-device** | Any phone reads; not fixed readers. |
| **Cost** | Tags $0.50–2; phones you have. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Zero wiring | Requires phone present; not hands-free for kids |
| Easy reprogram | Not multi-reader at door without phone |

---

## Solution F — Zigbee RFID keypad (off-the-shelf)

| | |
|---|---|
| **What it is** | Tuya/Zigbee touch keypads with RFID (vendor-specific). |
| **Custom codes** | Limited—vendor app pairing; expose via Zigbee2MQTT sometimes. |
| **Cost** | $40–80 per keypad. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| No soldering | Opaque payloads; cloud pairing history |

**Technical fit:** Avoid unless you accept vendor lock.

---

## Custom code strategies

| Strategy | Security | Flexibility |
|----------|----------|-------------|
| **UID only** | Low (UID cloneable) | High in HA YAML |
| **HA lookup table** | Medium (obscurity) | Change mapping without reflashing tags |
| **Writable sector secret** | Medium | Per-tag secret bytes; ESP/HA validates |
| **Rolling counter** | Higher | Complex; rare at home |
| **NTAG password** | Medium | Lock tag memory from casual rewrite |

For **sensitive actions**, prefer UID + HA conditions (time, presence) rather than secret sector alone.

---

## Multi-device architecture

```
[ESP kitchen] ──┐
[ESP garage]  ──┼── MQTT (Mosquitto) ── Home Assistant
[ESP office]  ──┘         │
                          ├─ automation: tag + device → action
                          └─ recorder: optional log of scans
```

**Naming:** `rfid/<location>/tag` topics; retain last UID optional.

**Power:** ESP32 USB at each point; or **PoE** with ESP32-PoE board ($25) + VLAN IoT.

---

## Comparison matrix

| Solution | Many readers | Custom payload | HA native | Tag cost | Range |
|----------|--------------|----------------|-----------|----------|-------|
| A ESPHome RC522/PN532 | ★★★★★ | ★★★★★ | ★★★★★ | $1–3 | Short |
| B Wiegand 125kHz | ★★★★★ | ★★★★ | ★★★★★ | $1–5 | Medium |
| C USB PN532 | ★ | ★★★★ | ★★★★ | $1–3 | Short |
| D IP controller | ★★★★ | ★★★ | ★★★ | $2–10 | Medium |
| E Phone NFC | ★★★ (phones) | ★★★★ | ★★★★★ | $1 | Touch |
| F Zigbee pad | ★★★ | ★★ | ★★★ | incl. | Short |

---

## Parts list (3-door ESPHome build)

| Item | Qty | Unit $ | Total |
|------|-----|--------|-------|
| ESP32-WROOM dev board | 3 | $8 | $24 |
| RC522 module | 3 | $4 | $12 |
| USB power / short cable | 3 | $5 | $15 |
| MIFARE Classic 1K tags (optional) | 10 | $1 | $10 |
| **Total** | | | **~$61** |

Add **PN532** (+$3/module) if you need **NFC** (phone emulation, NTAG).

---

## Home Assistant automation sketch

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

1. **Start wireless:** HA Companion **NFC tags** (Solution E) for scenes—zero hardware.  
2. **Fixed readers:** ESPHome + PN532 only where a pad on a shelf makes sense; USB power, no in-wall wiring.  
3. **Enrollment:** HA script maps `uid → scene name` in YAML or `input_select`.  
4. **Security:** IoT VLAN; MQTT auth; use RFID for convenience scenes, not security boundaries.
