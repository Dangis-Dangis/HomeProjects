# Solutions — Lights, TV & Audio Control

**Problem (P3):** Wireless control of **ceiling lights** (wall-switch, non-dimmable, keep physical control), **3 IR light zones**, the **living-room Vizio V4K65M-08 + Samsung ARC (IR)**, and the **bedroom E55-U + Chromecast 4K** — under one automation hub. *(Stretch: commercial mute/skip.)*

**Constraints:** no new cable runs (prefer Zigbee, Wi‑Fi, IR blasters, Cast, smart-TV APIs); ceiling fixtures are fixed and non-dimmable; replacing a switch is acceptable only if the paddle still works; focus is light/TV/audio — **not** door locks.

> The device-control "rails" referenced throughout are detailed in [02-shared-architecture.md](./02-shared-architecture.md): **Rail 1 = IR** (Samsung audio + IR lights), **Rail 2 = IP/Cast** (both TVs), **Rail 3 = smart switch** (ceiling lights, on/off, button preserved).

---

## Lighting

### Ceiling lights — Solution L1: Smart on/off switch replacement (recommended)

| | |
|---|---|
| **What it is** | Replace the dumb wall switch with a **smart on/off switch** (no dimmer, since the bulbs are non-dimmable). The paddle still physically switches the load and reports state to the hub. |
| **Picks** | **Lutron Caseta** switch (+ Pico, works **without a neutral**, own hub, rock-solid); **Inovelli/Zooz Zigbee** on/off (neutral required); **Shelly Plus 1** relay behind the existing switch (keeps the exact switch). |
| **Physical control** | Preserved — the paddle or original toggle still works if the hub/hub-bridge is down. |
| **Cost** | Caseta switch ~$60 + hub ~$80 (or Pico-only); Zigbee switch ~$35–45; Shelly ~$25 each. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Real two-way state; survives a hub outage | Requires touching mains wiring |
| On/off matches non-dimmable bulbs | Most need a neutral (Caseta excepted) |
| Shelly keeps the existing switch faceplate | Box depth for Shelly behind the switch |

**Why not smart bulbs:** fixtures are fixed and the wall switch cuts power — smart bulbs would be orphaned whenever someone flips the switch. Avoid.

### Ceiling lights — Solution L2: Shelly relay behind the switch (cheapest, switch unchanged)

| | |
|---|---|
| **What it is** | A **Shelly Plus 1** wired behind the existing switch in detached/edge mode; the dumb switch becomes a stateful smart load. |
| **Cost** | ~$25 per switch; needs neutral + box space. |

**Tradeoffs:** keeps the exact look/feel of the current switches and physical control; per-box wiring work and tight boxes are the catch.

### IR lights (3 zones) — Solution L3: Broadlink / ESPHome IR (reuse existing controllers)

| | |
|---|---|
| **What it is** | Learn the 3 IR remotes into a **Broadlink RM4 Pro** or an **ESPHome IR transmitter** (ESP32 + IR LED). The hub gets on/off/scene buttons; no hardware swap. |
| **Cost** | Broadlink ~$35; ESPHome node ~$8 each. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Zero rewiring; uses what's already there | **Open-loop** — no state feedback |
| The same blaster also drives Samsung audio | Line-of-sight per zone |

### IR lights (3 zones) — Solution L4: Replace IR receiver with Zigbee/WLED (best state)

| | |
|---|---|
| **What it is** | If these are LED strips, swap each IR controller module for a **Zigbee** (Gledopto/Sonoff) or **WLED (ESP)** controller — true on/off/dim/color with feedback. |
| **Cost** | $20–30 per zone. |

**Tradeoffs:** eliminates the open-loop IR weakness and adds dimming/color; requires opening the strip's controller and matching voltage (usually 12/24 V).

---

## TVs & audio

### Living room (V4K65M-08 + Samsung ARC) — Solution T1: Vizio SmartCast + Broadlink for audio

| | |
|---|---|
| **What it is** | The hub `vizio` integration controls the **TV** (power, input, app launch, `play_media`); a **Broadlink** (Rail 1) controls the **Samsung sound system** (power, volume, mute) because audio rides ARC and the Samsung gear is IR-only. |
| **Consolidates** | The **2 remotes → 1 "Watch TV" macro**: TV on → correct input → Samsung system on → volume preset. |
| **Cost** | $0 for the Vizio side (SmartCast over IP); Broadlink ~$35 (shared with the IR lights). |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Fixes the 2-remote annoyance | Volume/mute is open-loop IR (no exact level readback) |
| Uses the TV's existing IP control | App-launch support varies by SmartCast firmware |
| Roku stays as an optional backup target | CEC volume pass-through is unreliable on old Samsung — don't depend on it |

### Bedroom (E55-U + Chromecast 4K) — Solution T2: Chromecast/`androidtv` first

| | |
|---|---|
| **What it is** | Drive the **Chromecast 4K** via `cast` (+ `androidtv`/ADB) as the primary player; the E55-U's early SmartCast is limited, so treat the TV mostly as a display + (optional) IR power. |
| **Audio** | TV speakers or an ARC soundbar — CEC volume usually works here; fall back to Broadlink if not. |
| **Cost** | $0 (existing Chromecast). |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Full play/volume/app control via Cast/ADB | E55-U SmartCast API support is spotty — lean on the Chromecast |
| This is the **commercial-skip test bed** | Above-PC mount: confirm IR/CEC reach if adding a blaster |

### Solution T3: Hub host options (where this all runs)

| Option | What | Cost |
|--------|------|------|
| **HAOS** on N100/RPi 4 | Supervisor + add-ons (Mosquitto, Zigbee2MQTT) | $120–250 |
| **Hub container** on the shared server | Colocate with Immich/Frigate | $0 incremental |
| **HA Green/Yellow** | Appliance with a Zigbee/Thread radio | $99–199 |

---

## Control patterns (mapped to the existing gear)

| Target | Rail | Integration | The hub can… |
|--------|------|-------------|--------------|
| Ceiling lights | 3 (mains) | Caseta/Zigbee/Shelly | On/off + state, physical button intact |
| IR light zones | 1 (IR) | Broadlink/ESPHome | On/off/scene (no feedback) or full if swapped (L4) |
| Living-room TV | 2 (IP) | `vizio` | Power, input, app, play_media |
| Samsung audio | 1 (IR) | Broadlink | Power, volume, mute |
| Bedroom Chromecast | 2 (IP) | `cast`/`androidtv` | Play, volume, YouTube URL, ADB |
| Bedroom TV | 1/2 | `vizio` (limited)/IR | Power; rely on the Chromecast for apps |

**Screen-share tie-in:** the hub sends the same YouTube URL to both TVs' `media_player` targets — the wireless multi-TV layer (see [solutions-04-screen-share.md](./solutions-04-screen-share.md)).

---

## Stretch goal: commercial detection → mute / skip

Streaming apps expose no "ad playing" API. Feasibility differs **by room** because the players differ.

### Bedroom (Chromecast 4K = Google TV) — best chance

| Approach | Mute | Skip | How |
|----------|------|------|-----|
| **SmartTubeNext + SponsorBlock** (YouTube) | ● | ● | Sideload on Google TV; skips sponsor/ad segments |
| **ADB ad detection** (`androidtv`) | ● | ● | The hub reads the "Ad" UI / sends `KEYCODE_MEDIA_SKIP` |

### Living room (Vizio SmartCast apps) — mute only, via IR

| Approach | Mute | Skip | How |
|----------|------|------|-----|
| **Audio-loudness/fingerprint detector** (ESP32 mic) → **Broadlink mute** on Samsung | ● | ○ | A mic node publishes `ad_start`/`ad_end` to MQTT; the hub fires an IR mute |
| Manual "mute" button on the hub dashboard | ● | ○ | One-tap mute of Samsung via Broadlink |

### Live TV / antenna / cable (any room)

| Approach | Mute | Skip | How |
|----------|------|------|-----|
| **Channels DVR / Plex DVR** + commercial markers | ○ | ●●● | Time-shifted skip; needs an OTA tuner (HDHomeRun) |

**Reality:** reliable **skip** = YouTube (SponsorBlock) or DVR time-shift. Live SmartCast streaming apps → plan for **auto-mute** (IR to Samsung in the living room), not skip.

### Example automation (living-room mute via IR on an ad signal)

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

(Samsung mute is a toggle on most IR remotes — track state in an `input_boolean` so you don't double-toggle.)

---

## Multi-user household

```
Hub users (adults / kids)
    ├─ Dashboard: lights + scenes + TV/audio macros
    ├─ No door/unlock entities
    └─ Notify: optional Frigate clips (if the security stack is present)
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
| Starter | $150 hub box + 1 Broadlink ($35) + 3 smart switches ($120) | Living-room audio + ceiling lights + IR lights |
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

1. **Hub:** HAOS on an N100 (or a container on the home server) + a **Zigbee** coordinator.
2. **Ceiling lights:** **Lutron Caseta** on/off switches (no-neutral, keeps the physical paddle) — or **Shelly Plus 1** behind switches if you want the existing faceplates and have a neutral.
3. **IR layer:** one **Broadlink RM4 Pro** in the living room learns the **Samsung sound system** + (optionally) the TV; add **ESPHome IR nodes** for the 3 IR light zones that aren't in line-of-sight. Optionally upgrade IR strips to **Zigbee/WLED** later.
4. **Living-room TV:** the `vizio` integration for the V4K65M-08; **Broadlink for volume/mute** on the Samsung system; build a **"Watch TV" macro** to replace the 2-remote dance.
5. **Bedroom TV:** drive the **Chromecast 4K** via `cast`/`androidtv`; make it the **commercial-skip test bed** (SmartTubeNext + SponsorBlock, ADB).
6. **Commercial stretch:** SponsorBlock/ADB on the bedroom Google TV; an **MQTT audio detector → Broadlink mute** for living-room SmartCast apps; Channels DVR only if you add an OTA tuner.
7. **Remote:** mesh VPN + hub companion on iPhone and Android.

**Upgrade discipline:** monthly hub updates; re-test `vizio` and Cast after upgrades.

---

## Scored recommendation

**Top pick — HAOS + Lutron Caseta (ceiling) + Broadlink (IR) + Cast (TVs).** One hub, three control rails, each matched to the hardware already in place.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$350–700**: hub box $120–250; Caseta hub $80 + on/off switches $60 ea (or Shelly Plus 1 $25 behind existing switches); Broadlink RM4 $35; Zigbee coordinator $30; optional Zigbee/WLED swap for IR strips $20–30/zone. |
| **Setup time** | **1–3 days** (hub + Zigbee, switch wiring, learn IR remotes, build TV/audio macros). |
| **Maintenance** | **Med** — monthly hub updates; re-test `vizio`/Cast after upgrades; IR is open-loop, so occasional resync. |
| **Feasibility** | ★★★★★ — mains wiring + IR learning are easy for this household. |
| **Scalability** | ★★★★★ — add switches/zones/scenes indefinitely; the rails generalize. |

**The three rails (reused across the portfolio):**
1. **Mains switch (Caseta/Shelly)** → ceiling lights, on/off, physical paddle preserved.
2. **IR (Broadlink/ESPHome)** → Samsung audio **and** the 3 IR light zones (same blaster) + living-room volume/mute.
3. **IP/Cast** → V4K65M-08 via `vizio`, Chromecast 4K via `cast`/`androidtv`.

**Commercial stretch:** the bedroom Chromecast 4K = test bed (SmartTubeNext + SponsorBlock, ADB skip); the living room = **audio-detector → Broadlink mute** on the Samsung system (mute only, not skip).

**Reuse:** this is the **hub** for Security alerts, Screen share, the Unified remote, and RFID scenes. The **IR rail** is shared by Screen share (living-room volume) and the Remote (which can fire IR macros). The **mesh VPN** + hub companion give remote control on both phone platforms.

**Alternate A — Hub container on the N100/Proxmox.** Folds the hub onto the media/security server (no separate box, **$0 incremental**). Maintenance Med–High (no Supervisor add-ons; run Mosquitto/Zigbee2MQTT as compose). Best if minimizing devices.

**Alternate B — Shelly-everywhere (Wi‑Fi-first, no Zigbee).** Shelly relays behind every switch + Broadlink for IR; skip a Zigbee coordinator. Cost similar; feasibility ★★★★★; scalability ★★★★ (Wi‑Fi device count on one AP is the limit). Good if you prefer Wi‑Fi/HTTP over a Zigbee mesh.

**Verdict:** Build the **hub first** (backbone #1). Do **Caseta** switches for the cleanest no-neutral install that keeps physical control, or **Shelly** if you want to keep existing faceplates and have neutrals. One **Broadlink** solves Samsung audio + IR lights at once. Treat the **bedroom Chromecast** as the commercial-skip playground; accept **mute-only** in the living room.

---

## Full dossier — recommended build (HAOS + Caseta + Broadlink + Cast)

### Concept image

![One hub controlling a switch, IR blaster/TV, and RGB strip over wireless rails](./assets/concept-home-control.png)

### Parts list

| Part | Qty | Unit $ | Total |
|------|-----|--------|-------|
| Hub host — Home Assistant Green (or N100 running HAOS) | 1 | $99 | $99 |
| Zigbee/Thread coordinator (Home Assistant SkyConnect / SLZB-06) | 1 | $30 | $30 |
| Lutron Caseta smart bridge | 1 | $80 | $80 |
| Lutron Caseta on/off switch (no-neutral) + Pico | 3 | $60 | $180 |
| Broadlink RM4 Pro (Wi‑Fi → IR) | 1 | $35 | $35 |
| ESPHome IR node (ESP32 + IR LED + 5 V supply) for distant IR zones | 2 | $10 | $20 |
| (Optional) Zigbee power-monitoring plug — closes the IR loop on Samsung audio | 1 | $13 | $13 |
| **Total (3 ceiling switches + full IR coverage)** | | | **~$457** |

Home Assistant, ESPHome, and all integrations are free.

### Cost example

- **Starter:** HA Green $99 + Caseta bridge $80 + 1 switch $60 + Broadlink $35 + Zigbee stick $30 ≈ **$304** (living-room audio + 1 ceiling zone + IR lights).
- **Recommended (3 ceiling zones + distant IR):** ≈ **$457** (table above).
- **Shelly variant (keep faceplates, have neutral):** swap Caseta bridge + switches for 3× Shelly Plus 1 ($25) = save the $80 bridge → ≈ **$224** for the lighting portion.

### Data path

```
[phone / hub dashboard / unified remote]
        │ Wi-Fi (hub API / companion)
        ▼
   Home Assistant ──┬─ Rail 2 IP/Cast ──► Vizio V4K65M-08 (vizio), Chromecast 4K (cast/androidtv)
                    │
                    ├─ Rail 1 IR ─► Broadlink RM4 ──IR──► Samsung audio (vol/mute), TV power
                    │            └─► ESPHome IR node ─IR─► distant IR light zones
                    │
                    └─ Rail 3 Zigbee/Clear-Connect ─► Caseta/Shelly switch ─► ceiling lights
        ▲
   (state back) power-monitoring plug ─► HA confirms Samsung "actually on"
```

### General wiring / topology diagram

Smart on/off switch at the ceiling-light box (load wired through the switch so the paddle still cuts power):

```
  Line (hot) ──────────────► [ L ]
                              │  Lutron Caseta / Shelly Plus 1
  Neutral ───────────────────► [ N ]   (Caseta works WITHOUT neutral;
                              │          Shelly REQUIRES neutral)
  Load to ceiling fixture ◄─── [ LOAD ]
  Ground ────────────────────► [ G ]
        paddle/toggle = physical on/off, reports state to hub
```

ESPHome IR node (for a distant IR light zone):

```
  ESP32 dev board
    5V ─────► IR LED driver (NPN/MOSFET) ─► IR LED ──► aimed at the zone receiver
    GPIO ───► transistor base/gate (current-limit resistor)
    GND ────► common
    USB-C 5V ─► wall adapter (no in-wall wiring)
```

Broadlink RM4 Pro sits in open line-of-sight of the Samsung system + TV; it is mains-USB powered and joins over Wi‑Fi (no wiring).

### Effort to create

| Phase | Effort |
|-------|--------|
| Install HAOS, pair Zigbee coordinator, set up mesh VPN + companion | 3–4 hrs |
| Replace/insert ceiling switches (power off at breaker; per box) | 30–45 min/switch |
| Learn Samsung + IR-light remotes into Broadlink; flash ESPHome IR nodes | 2–3 hrs |
| Build "Watch TV" macro + scenes; wire commercial-mute automation | 2–3 hrs |
| **Total** | **~1–3 days** (spread over a week is comfortable) |

### Maintenance cost

- **Electricity:** hub ~3–6 W + bridge/blaster trivial ≈ **$8–15/yr**.
- **Time:** monthly HA update + re-test `vizio`/Cast after TV firmware ≈ **30 min/mo**; occasional IR resync (open-loop).
- **Parts:** essentially none; switches/bridge are long-lived. Pico/remote coin cells every few years (~$2).
- **Subscriptions:** none.

### Cost feasibility & scalability

- **Per-zone marginal cost:** ceiling switch **~$60** (Caseta) or **~$25** (Shelly); distant IR zone **~$10** (ESPHome node); IR-strip → Zigbee/WLED upgrade **$20–30/zone**.
- **Shared infrastructure is one-time:** the hub, Zigbee coordinator, Caseta bridge, and Broadlink are bought once and then amortized across every new device.
- **No per-device fees**; Zigbee mesh strengthens (not weakens) as mains-powered devices are added.
- **Takeaway:** moderate one-time backbone (~$150–250), then cheap linear per-zone growth — very cost-feasible to expand room by room.

### Potential risks & downsides

| Risk | Mitigation |
|------|-----------|
| **Mains wiring (shock/fire)** | Kill the breaker, verify dead, follow code; use Caseta no-neutral where boxes lack neutral |
| **IR is open-loop (no confirmation)** | Back Samsung audio with a power-monitoring plug; or swap IR strips to Zigbee/WLED |
| **CEC volume pass-through unreliable on old Samsung** | Route living-room volume via IR (Broadlink), never depend on CEC |
| **Vizio SmartCast app-launch varies by firmware** | Treat app launch as best-effort; lean on Cast for the bedroom |
| **Hub outage** | Caseta/Shelly keep the physical paddle working; Caseta bridge runs locally |
| **Zigbee vs. Wi‑Fi channel interference** | Pick a Zigbee channel away from Wi‑Fi; or use the Shelly Wi‑Fi-only variant |
