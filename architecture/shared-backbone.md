# Shared Backbone

Build these once; every project reuses them.

```
            ┌──────────── Tailscale / WireGuard ────────────┐
            │                                              │
   ┌────────┴─────────┐         ┌──────────────┐         ┌─┴──────────────┐
   │  N100 server      │◄───────│  Home         │◄────────│  ESPHome / IR  │
   │  Immich + Frigate │         │  Assistant    │         │  Broadlink,    │
   └───────────────────┘         └──────┬───────┘         │  ESP32 deck    │
                                         │                 └────────────────┘
                                         ▼
                    Cast (both TVs) · Caseta switches · Cast audio groups
```

| Building block | Projects | Payoff |
|----------------|----------|--------|
| **Tailscale** | 1–5 | One remote-access config |
| **N100 server** | 1, 2 | Photos + NVR; one backup job |
| **Home Assistant** | 2–6 | Hub for control, cast, alerts |
| **Google Cast** | 3, 4 | Both TVs as `media_player` |
| **Broadlink IR** | 3, 4, 5 | Samsung audio + IR lights |
| **Caseta/Shelly** | 3 | Ceiling lights, button kept |
| **Immich** | 1 | Native iOS + Android photo apps |
| **Frigate** | 2 | Local AI clips (optional Coral later) |
| **ESPHome + MQTT** | 3, 5, 6 | One DIY toolchain |

**Biggest wins:** one server (media + security); one hub + ESPHome + IR (control + remote + RFID); one VPN (everything remote).

See [control-rails.md](./control-rails.md) for how existing devices map onto IR / Cast / mains.
