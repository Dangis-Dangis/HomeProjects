---
name: home-projects
description: >-
  Connected home design repo (HomeProjects): Immich, Frigate, Home Assistant,
  IR RGB accent lights, Vizio TVs, unified ESPHome remote. Use when editing
  needs/, architecture/, projects/, docs/, assets/, or discussing household
  automation, discrete IR RGB colors, volume-only remote dial, repo consistency,
  or archiving superseded docs.
---

# HomeProjects

Privacy-respecting, wireless-first connected home — **six capabilities**, one shared backbone.

## Before editing

1. [docs/agent-guide.md](../../../docs/agent-guide.md) — **read → archive → write from scratch**
2. [needs/constraints-and-inventory.md](../../../needs/constraints-and-inventory.md)
3. [docs/domain/rgb-lights.md](../../../docs/domain/rgb-lights.md), [docs/domain/unified-remote.md](../../../docs/domain/unified-remote.md)
4. [docs/documentation-standards.md](../../../docs/documentation-standards.md)

## Consistency policy (required)

The **repo root must stay fully consistent** for a first-time external reader.

When changing one area:

1. **Grep the live tree** (not `archive/`) for stale references.
2. **Update every live doc** that mentions the topic — or **move superseded files into** `archive/YYYY-MM-DD-vN-name/`.
3. **Prefer rewrite from scratch** over incremental patches that leave old paragraphs.
4. **Archive grows** by adding dated folders; **do not edit old archive files in place** to match new specs.

Run the checklist in `docs/agent-guide.md` before finishing.

## Hard rules

- **Live surface:** `README.md`, `needs/`, `architecture/`, `portfolio/`, `projects/`, `docs/`, `assets/`
- **Embed images** from `assets/` when they exist
- **IR RGB:** discrete swatches (8–20) + 5 brightness levels + programs — not HSV/RGB 0–255
- **Remote:** one **volume dial**; fixed **Sync Screens / ALL ON / GOODNIGHT**; screen soft labels for other actions; no Party Time hardware button
- **GPIO table** in `projects/05-unified-remote/wiring.md` beats wiring images

## Layout

| Path | Role |
|------|------|
| Root | Current truth — must not contradict itself |
| `archive/` | Historical generations — link only, append new folders |

## Stack (short)

Tailscale + N100 (Immich + Frigate) + Home Assistant + Broadlink + Caseta + Cast.

[portfolio/build-order-and-costs.md](../../../portfolio/build-order-and-costs.md)
