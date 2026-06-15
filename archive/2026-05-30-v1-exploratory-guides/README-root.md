# Home Projects — Solution Guides

Each guide compares **multiple implementations** for one capability. Constraints reflected across guides:

- **Wireless-first** where cabling between rooms is not an option (screen share, HA devices)
- **iPhone + Android** native apps preferred for photos and daily control
- **Local storage** for security (2–3 cameras)
- **Home Assistant** as optional hub for lights, TV, audio, and Cast orchestration
- **Existing devices honored:** wall-switch (non-dimmable) ceiling lights with physical control retained, 3 IR-only light zones, Vizio **V4K65M-08** + old **Samsung ARC** audio (IR), Vizio **E55-U** + **Chromecast 4K**

| Guide | Topic |
|-------|--------|
| [solution-similarity-across-projects.md](./solution-similarity-across-projects.md) | Shared stacks, overlap, unified recommendations |
| [device-control-similarity.md](./device-control-similarity.md) | Your existing lights/TVs grouped into 3 control "rails" |
| [personal-media-storage.md](./personal-media-storage.md) | Photos — Immich/Synology, remote mobile apps |
| [home-security-recording.md](./home-security-recording.md) | 2–3 cameras, local NVR |
| [home-assistant-integration.md](./home-assistant-integration.md) | Lights (switch + IR), TV, audio; commercial detection |
| [screen-share-multiple-tvs.md](./screen-share-multiple-tvs.md) | Wireless Cast on your two Vizios, multi-device, remote |
| [unified-remote.md](./unified-remote.md) | DIY battery handheld control deck (buttons/dials/voice/LEDs) |
| [concept-collection.md](./concept-collection.md) | 6 rendered remote concepts — cost breakdowns, pros/cons, feature table |
| [rfid-custom-codes.md](./rfid-custom-codes.md) | Optional: NFC/RFID for scenes (not door focus) |

## Fresh suggestions (second pass — scored)

The [fresh-suggestions/](./fresh-suggestions/) directory distills these guides into **opinionated, scored** recommendations (top pick + alternates per project) rated on **cost, setup/maintenance time, feasibility, scalability, and project similarity**, plus a portfolio build order. Start at [fresh-suggestions/README.md](./fresh-suggestions/README.md).

**Assumptions:** You run your own network, can use Tailscale/VLANs, and are comfortable with Docker, Linux, and consumer smart-home gear.

**Cost notes:** USD, rough 2025–2026 retail; excludes labor and hardware you already own (TVs, phones).
