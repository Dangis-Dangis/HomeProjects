# Home Assistant — Wireless Home Hobby (Lights, TV, Audio)

**Constraints & existing hardware:**
- **No new cable runs** across rooms—prefer Zigbee, Wi‑Fi, IR blasters, Cast, and smart-TV APIs.
- **Ceiling lights:** wall-switch controlled, **non-dimmable** bulbs, fixtures cannot be changed. **Replacing the switch is acceptable** as long as **physical switch control still works**.
- **Other lights (×3 zones):** controlled only by **IR controllers with no physical buttons**.
- **Living room TV:** Vizio **V4K65M-08** (SmartCast, Chromecast built-in); audio via **old Samsung sound system over ARC, controlled by IR (≈2 remotes)**; old Roku present but unused.
- **Bedroom TV:** Vizio **E55-U** (older), mounted above a PC; primary source **Chromecast 4K**; audio from TV speakers or an ARC soundbar.

**Focus:** light control, TV control, audio control—not door locks.

**Stretch goals:** detect commercials → **mute**; detect commercials → **fast-forward** where feasible.

> Control rails recap (see [device-control-similarity.md](./device-control-similarity.md)): **Rail 1 = IR** (Samsung audio + IR lights), **Rail 2 = IP/Cast** (both TVs), **Rail 3 = smart switch** (ceiling lights, on/off, button preserved).

---

## Lighting

### Ceiling lights — Solution L1: Smart on/off switch replacement (recommended)

| | |
|---|---|
| **What it is** | Replace the dumb wall switch with a **smart on/off switch** (no dimmer, since bulbs are non-dimmable). Paddle still physically switches the load, and reports state to HA. |
| **Picks** | **Lutron Caseta** switch (+ Pico, works **without neutral**, own hub, rock-solid); **Inovelli/Zooz Zigbee** on/off (neutral required); **Shelly Plus 1** relay behind the existing switch (keeps your exact switch). |
| **Physical control** | Preserved—paddle or original toggle still works if HA/hub is down. |
| **Cost** | Caseta switch ~$60 + hub ~$80 (or Pico-only); Zigbee switch ~$35–45; Shelly ~$25 each. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Real two-way state; survives HA outage | Requires touching mains wiring |
| On/off matches non-dimmable bulbs | Most need a neutral (Caseta excepted) |
| Shelly keeps your existing switch faceplate | Box depth for Shelly behind switch |

**Why not smart bulbs:** fixtures are fixed and the wall switch cuts power—smart bulbs would be orphaned whenever someone flips the switch. Avoid.

### Ceiling lights — Solution L2: Shelly relay behind switch (cheapest, switch unchanged)

| | |
|---|---|
| **What it is** | **Shelly Plus 1** wired behind the existing switch in "detached/edge" mode; dumb switch becomes a stateful smart load. |
| **Cost** | ~$25 per switch; needs neutral + box space. |

**Tradeoffs:** keeps the exact look/feel of current switches and physical control; per-box wiring work and tight boxes are the catch.

### IR lights (3 zones) — Solution L3: Broadlink / ESPHome IR (reuse existing controllers)

| | |
|---|---|
| **What it is** | Learn the 3 IR remotes into a **Broadlink RM4 Pro** or **ESPHome IR transmitter** (ESP32 + IR LED). HA gets on/off/scene buttons; no hardware swap. |
| **Cost** | Broadlink ~$35; ESPHome node ~$8 each. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Zero rewiring; uses what you have | **Open-loop**—no state feedback |
| Same blaster also drives Samsung audio | Line-of-sight per zone |

### IR lights (3 zones) — Solution L4: Replace IR receiver with Zigbee/WLED (best state)

| | |
|---|---|
| **What it is** | If these are LED strips, swap each IR controller module for a **Zigbee** (Gledopto/Sonoff) or **WLED (ESP)** controller—true on/off/dim/color with feedback. |
| **Cost** | $20–30 per zone. |

**Tradeoffs:** eliminates the open-loop IR weakness and adds dimming/color; requires opening the strip's controller and matching voltage (usually 12/24 V).

---

## TVs & audio

### Living room (V4K65M-08 + Samsung ARC) — Solution T1: Vizio SmartCast + Broadlink for audio

| | |
|---|---|
| **What it is** | HA `vizio` integration controls the **TV** (power, input, app launch, `play_media`); a **Broadlink** (Rail 1) controls the **Samsung sound system** (power, volume, mute) because audio rides ARC and the Samsung gear is IR-only. |
| **Consolidates** | Your **2 remotes → 1 HA "Watch TV" macro**: TV on → correct input → Samsung system on → volume preset. |
| **Cost** | $0 for the Vizio side (SmartCast over IP); Broadlink ~$35 (shared with IR lights). |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Fixes the 2-remote annoyance | Volume/mute is open-loop IR (no exact level readback) |
| Uses TV's existing IP control | App launch support varies by SmartCast firmware |
| Roku stays as optional backup target | CEC volume pass-through unreliable on old Samsung—don't depend on it |

### Bedroom (E55-U + Chromecast 4K) — Solution T2: Chromecast/`androidtv` first

| | |
|---|---|
| **What it is** | Drive the **Chromecast 4K** via `cast` (+ `androidtv`/ADB) as the primary player; the E55-U's early SmartCast is limited, so treat the TV mostly as a display + (optional) IR power. |
| **Audio** | TV speakers or ARC soundbar—CEC volume usually works here; fall back to Broadlink if not. |
| **Cost** | $0 (existing Chromecast). |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Full play/volume/app control via Cast/ADB | E55-U SmartCast API support is spotty—lean on Chromecast |
| This is your **commercial-skip test bed** | Above-PC mount: confirm IR/CEC reach if adding blaster |

### Solution T3: HA hub options (where this all runs)

| Option | What | Cost |
|--------|------|------|
| **HAOS** on N100/RPi 4 | Supervisor + add-ons (Mosquitto, Zigbee2MQTT) | $120–250 |
| **HA Container** on shared server | Colocate with Immich/Frigate | $0 incremental |
| **HA Green/Yellow** | Appliance with Zigbee/Thread radio | $99–199 |

---

## Control patterns (mapped to your gear)

| Target | Rail | Integration | HA can… |
|--------|------|-------------|---------|
| Ceiling lights | 3 (mains) | Caseta/Zigbee/Shelly | On/off + state, physical button intact |
| IR light zones | 1 (IR) | Broadlink/ESPHome | On/off/scene (no feedback) or full if swapped (L4) |
| Living room TV | 2 (IP) | `vizio` | Power, input, app, play_media |
| Samsung audio | 1 (IR) | Broadlink | Power, volume, mute |
| Bedroom Chromecast | 2 (IP) | `cast`/`androidtv` | Play, volume, YouTube URL, ADB |
| Bedroom TV | 1/2 | `vizio` (limited)/IR | Power; rely on Chromecast for apps |

**Screen-share tie-in:** HA sends the same YouTube URL to both TVs' `media_player` targets—your wireless multi-TV layer (see [screen-share-multiple-tvs.md](./screen-share-multiple-tvs.md)).

---

## Stretch goal: commercial detection → mute / skip

Streaming apps expose no "ad playing" API. Feasibility differs **by room** because the players differ.

### Bedroom (Chromecast 4K = Google TV) — best chance

| Approach | Mute | Skip | How |
|----------|------|------|-----|
| **SmartTubeNext + SponsorBlock** (YouTube) | ● | ● | Sideload on Google TV; skips sponsor/ad segments |
| **ADB ad detection** (`androidtv`) | ● | ● | HA reads "Ad" UI / sends `KEYCODE_MEDIA_SKIP` |

### Living room (Vizio SmartCast apps) — mute only, via IR

| Approach | Mute | Skip | How |
|----------|------|------|-----|
| **Audio-loudness/fingerprint detector** (ESP32 mic) → **Broadlink mute** on Samsung | ● | ○ | Mic node publishes `ad_start`/`ad_end` to MQTT; HA fires IR mute |
| Manual "mute" button on HA dashboard | ● | ○ | One-tap mute of Samsung via Broadlink |

### Live TV / antenna / cable (any room)

| Approach | Mute | Skip | How |
|----------|------|------|-----|
| **Channels DVR / Plex DVR** + commercial markers | ○ | ●●● | Time-shifted skip; needs OTA tuner (HDHomeRun) |

**Reality:** reliable **skip** = YouTube (SponsorBlock) or DVR time-shift. Live SmartCast streaming apps → plan for **auto-mute** (IR to Samsung in the living room), not skip.

### Example automation (living-room mute via IR on ad signal)

```yaml
automation:
  - alias: Mute Samsung when ad detector fires
    trigger:
      - platform: mqtt
        topic: livingroom/ad_detector
        payload: "ad_start"
    action:
      - service: remote.send_command
        target:
          entity_id: remote.living_room_broadlink
        data:
          device: samsung_audio
          command: mute
  - alias: Unmute Samsung when content resumes
    trigger:
      - platform: mqtt
        topic: livingroom/ad_detector
        payload: "ad_end"
    action:
      - service: remote.send_command
        target:
          entity_id: remote.living_room_broadlink
        data:
          device: samsung_audio
          command: mute
```

(Samsung mute is a toggle on most IR remotes—track state in an `input_boolean` so you don't double-toggle.)

---

## Multi-user household

```
HA users (adults / kids)
    ├─ Dashboard: lights + scenes + TV/audio macros
    ├─ No door/unlock entities
    └─ Notify: optional Frigate clips (if security stack present)
```

- **Kids:** ceiling/IR lights + limited scenes.
- **Adults:** full TV/audio macros, commercial-mute toggle.

---

## Radio & device choices

| Protocol | Use here | Notes |
|----------|----------|-------|
| **Lutron Clear Connect** | Ceiling switches (Caseta) | No-neutral option; own hub; very reliable |
| **Zigbee** | Switches, optional LED swap | Mesh via mains devices |
| **Wi‑Fi** | Shelly relay, Broadlink IR | IoT VLAN |
| **IR (via Wi‑Fi blaster)** | Samsung audio, 3 IR light zones | Open-loop; line-of-sight |
| **Cast / IP** | Both TVs | Primary for Vizio/Chromecast |

---

## Cost scenarios

| Profile | Hardware | Notes |
|---------|----------|-------|
| Starter | $150 HA box + 1 Broadlink ($35) + 3 smart switches ($120) | Living room audio + ceiling lights + IR lights |
| + Bedroom AV | + $0 (existing Chromecast) | Commercial-skip experiments |
| LED upgrade | + $20–30 × IR zone (Zigbee/WLED) | Replaces open-loop IR with stateful control |

---

## Comparison matrix

| Solution | Ceiling lights | IR lights | LR TV+audio | BR TV | Commercial | Physical button kept |
|----------|----------------|-----------|-------------|-------|------------|----------------------|
| L1 Smart switch | ★★★★★ | — | — | — | — | ★★★★★ |
| L2 Shelly relay | ★★★★★ | — | — | — | — | ★★★★★ |
| L3 Broadlink IR | — | ★★★★ | (audio) | (pwr) | mute | n/a |
| L4 Zigbee/WLED swap | — | ★★★★★ | — | — | — | n/a |
| T1 Vizio+Broadlink | — | — | ★★★★★ | — | mute | n/a |
| T2 Chromecast/ADB | — | — | — | ★★★★★ | mute+skip | n/a |

---

## Recommended path

1. **HA hub:** HAOS on N100 (or Container on your home server) + **Zigbee** coordinator.
2. **Ceiling lights:** **Lutron Caseta** on/off switches (no-neutral, keeps physical paddle) — or **Shelly Plus 1** behind switches if you want the existing faceplates and have neutral.
3. **IR layer:** one **Broadlink RM4 Pro** in the living room learns the **Samsung sound system** + (optionally) the TV; add **ESPHome IR nodes** for the 3 IR light zones not in line-of-sight. Optionally upgrade IR strips to **Zigbee/WLED** later.
4. **Living room TV:** `vizio` integration for the V4K65M-08; **Broadlink for volume/mute** on the Samsung system; build a **"Watch TV" macro** to replace the 2-remote dance.
5. **Bedroom TV:** drive the **Chromecast 4K** via `cast`/`androidtv`; make it your **commercial-skip test bed** (SmartTubeNext + SponsorBlock, ADB).
6. **Commercial stretch:** SponsorBlock/ADB on the bedroom Google TV; **MQTT audio detector → Broadlink mute** for living-room SmartCast apps; Channels DVR only if you add an OTA tuner.
7. **Remote:** Tailscale + HA Companion on iPhone and Android.

**Upgrade discipline:** monthly HA updates; re-test `vizio` and Cast after upgrades.
