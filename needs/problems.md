# Problem Statements

Six independently buildable problems. All share the [shared backbone](../architecture/shared-backbone.md).

---

## P1 — Media & photo storage

**Problem:** Replace cloud photo service with a private library usable daily from iPhone and Android.

**Success:** Local disk; native auto-upload apps both platforms; remote via VPN; multi-user libraries + shared albums.

**Out of scope:** Heavy office/document collaboration.

→ [projects/01-media-storage/](../projects/01-media-storage/)

---

## P2 — Security recording (2–3 cameras)

**Problem:** Small camera fleet, local retention, remote viewing for multiple users.

**Success:** 2–3 cams; 14–30 day local retention; no subscription; optional person detection.

**Out of scope:** 8+ cam commercial NVR; cloud-only cameras.

→ [projects/02-security/](../projects/02-security/)

---

## P3 — Lights, TV, and audio control

**Problem:** Fragmented control (wall switches, 3 IR light remotes, 2 TV setups, 2-remotes-for-Samsung) → one hub.

**Success:** Ceiling lights hub-controlled with physical switch preserved; IR lights controllable; living-room watch macro; bedroom Chromecast full control; *(stretch)* commercial mute/skip.

**Out of scope:** Door locks.

→ [projects/03-home-control/](../projects/03-home-control/)

---

## P4 — Wireless screen share

**Problem:** Same YouTube/streaming on multiple TVs/devices without cabling; watch-together remotely.

**Success:** One action → both TVs; phone/laptop cast; cross-state for owned media + shareable links; LR volume via Samsung IR.

**Out of scope:** DRM tab mirror; frame-perfect sync.

→ [projects/04-screen-share/](../projects/04-screen-share/)

---

## P5 — Unified physical remote

**Problem:** Handheld battery deck for common actions — tactile controls, status feedback, voice.

**Success:** Reusable **screen** context (Room/Type); **one volume dial**; discrete RGB picker; fixed **Sync Screens / ALL ON / GOODNIGHT**; soft-labeled action buttons; status feedback + voice (PTT).

**Binding constraints.** Volume is the only analog dial; RGB is discrete swatches — see [docs/domain/rgb-lights.md](../docs/domain/rgb-lights.md).

**Out of scope:** Replacing the phone as primary UI; hue/saturation dials; dedicated Party Time button.

→ [projects/05-unified-remote/](../projects/05-unified-remote/)

---

## P6 — RFID / NFC scenes

**Problem:** Tap-to-trigger scenes without door security use.

**Success:** Phone NFC in minutes; optional fixed ESP readers; maps to existing hub scenes.

**Out of scope:** Perimeter access control.

→ [projects/06-rfid-scenes/](../projects/06-rfid-scenes/)
