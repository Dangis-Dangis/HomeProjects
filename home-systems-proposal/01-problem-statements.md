# Problem Statements

The work decomposes into six discrete problem statements. Each is independently buildable and maps to one solution collection, yet all six draw on the same shared backbone described in [02-shared-architecture.md](./02-shared-architecture.md). Each statement below names the problem, the success criteria, the binding constraints, and the explicit out-of-scope boundary.

---

## P1 — Private, remotely accessible media & photo storage

**Problem.** The household needs a private home for photos, video, and personal files that replaces a cloud photo service without sacrificing the daily mobile experience.

**Success criteria**
- Photos and video are stored on **local disk**, owned and backed up by the household.
- **Native iOS and Android apps** provide auto-upload, timeline, search, albums, and sharing — not a web-only admin panel.
- The library is **reachable remotely** from both phone platforms without exposing the server to the public internet.
- **Multiple users** have their own libraries plus shared albums.

**Binding constraints.** iPhone + Android parity; local storage / no subscription lock-in; remote access via mesh VPN.

**Out of scope.** General-purpose office document collaboration beyond basic file sync.

→ [solutions-01-media-storage.md](./solutions-01-media-storage.md)

---

## P2 — Local-recording home security for a small camera fleet

**Problem.** The household wants 2–3 cameras recording continuously and/or on motion to **local storage**, with multiple members able to view live and recorded footage remotely.

**Success criteria**
- 2–3 cameras record to local disk with a sensible retention window (≈14–30 days).
- No mandatory cloud subscription for recording or playback.
- **Multi-user remote viewing** from both phone platforms.
- Optional object/person detection to cut down on noise.

**Binding constraints.** Wireless-first (Wi‑Fi cameras to avoid cross-room cabling); local storage; remote access via mesh VPN; camera traffic isolated on its own VLAN.

**Out of scope.** Large-fleet (8+ camera) commercial NVR deployments; cloud-only camera ecosystems.

→ [solutions-02-security-recording.md](./solutions-02-security-recording.md)

---

## P3 — Unified, automatable control of existing lights, TVs, and audio

**Problem.** Control is fragmented across wall switches, three separate IR light remotes, two TV ecosystems, and an IR-only Samsung sound system that today needs roughly two remotes. The household wants this consolidated under one automation hub while honoring the existing hardware and keeping physical light switches working.

**Success criteria**
- Ceiling lights gain hub control and state feedback **while the wall switch still physically works**.
- The three IR light zones are controllable from the hub.
- The living-room "watch TV" experience collapses **two remotes into one macro** (TV on → correct input → Samsung audio on → volume).
- The bedroom Chromecast TV is fully controllable (play, volume, app launch).
- *(Stretch)* Commercials are **muted** automatically where detectable, and **skipped** where the player allows.

**Binding constraints.** Wireless-first; non-dimmable on/off ceiling fixtures; preserve physical switch control; integrate the Vizio TVs, Samsung IR audio, Chromecast 4K, and 3 IR light controllers as-is.

**Out of scope.** Door locks and any security-boundary automation.

→ [solutions-03-home-control.md](./solutions-03-home-control.md)

---

## P4 — Wireless screen sharing across TVs and devices

**Problem.** The household wants to push the same content (primarily YouTube and streaming apps) to multiple TVs and Wi‑Fi devices without cabling, sourced from a phone or laptop, and to watch the same thing together with family in another state.

**Success criteria**
- One action plays the **same YouTube/content on both TVs**.
- Any phone/laptop on Wi‑Fi can **cast to either TV**.
- Casting works **to any Wi‑Fi device** (TVs, phones, tablets) as a client.
- **Cross-state watch-together** is possible for owned media and for shareable links.
- Living-room **volume routes correctly to the Samsung system** during a "play" macro.

**Binding constraints.** Wireless-only (no HDMI/network cabling between rooms); the living-room volume lives on the IR layer, not the Cast volume.

**Out of scope.** Bit-perfect mirroring of arbitrary DRM streams to many screens; frame-locked sync.

→ [solutions-04-screen-share.md](./solutions-04-screen-share.md)

---

## P5 — A battery-powered unified physical remote

**Problem.** The household wants a handheld, battery-powered control deck that fires the most common automation actions with real tactile controls, status feedback, an optional screen, and voice — a physical front-end for the hub macros.

**Success criteria**
- **Reusable dials** whose function changes with a **Room → Type** menu context (Living Room/Bedroom × RGB/Ceiling).
- A context-independent **Automation** set: Sync TVs, Night Time, Party Time, Cameras On.
- **Status LEDs** for power, command sent, success/failure, and hub reachability.
- **Push-to-talk voice** as a fallback for anything not on a control.
- **Battery powered**, night-viewable, with dim/sleep; one- or two-hand operation.
- Physical controls map to **confirmed** results (closed-loop feedback wherever the device rail allows).

**Binding constraints.** Talks only to the hub over Wi‑Fi (the hub fans out to each device); battery life favors push-to-talk over always-listening; non-dimmable ceiling context means fixture-select + toggle rather than dimming.

**Out of scope.** Direct device control bypassing the hub; always-on cloud voice.

→ [solutions-05-unified-remote.md](./solutions-05-unified-remote.md)

---

## P6 — Tap-to-trigger RFID/NFC convenience scenes

**Problem.** The household wants to tap a tag to run a scene (lights, media, automations) at natural spots around the home, using phones and/or low-cost fixed readers — strictly for convenience, never as a security boundary.

**Success criteria**
- A tag maps to a **friendly scene name** in the hub and fires the same scenes the remote does.
- Works from **phones** (NFC) and from optional **fixed ESP readers** without new wiring.
- Mapping changes (tag → scene) require **no re-flashing of tags**.

**Binding constraints.** Wireless-friendly (phone NFC or USB/battery-powered ESP readers); RFID used for convenience only; readers isolated on the IoT VLAN.

**Out of scope.** Door unlock, electric strikes, payments, or any access-control use.

→ [solutions-06-rfid-scenes.md](./solutions-06-rfid-scenes.md)

---

## How the problems relate

The six problems are independent in *outcome* but heavily shared in *architecture*. P1 and P2 share a server. P3, P4, P5, and P6 all share the automation hub. P3, P5, and P6 share the ESPHome/IR DIY toolchain. P3 and P4 share Google Cast and the IR layer. All of them share the mesh VPN for remote access. The next document maps these overlaps in detail.
