# Personal Media & Photo Storage (iPhone + Android, Remote)

**Priority:** **Maximal remote accessibility** from **iPhone and Android**, with **native photo-app experience** preferred over web-only admin panels.

**Also:** Central store for home video and personal files; multi-user; background phone backup on Wi‑Fi.

---

## Solution A — TrueNAS SCALE (or N100 Docker) + Immich

| | |
|---|---|
| **What it is** | Self-hosted **Immich**—closest open-source equivalent to Google Photos—with official **iOS and Android apps** (auto-upload, timeline, search, albums, partner sharing). |
| **Multi-user** | Per-user libraries, shared albums, partner link. |
| **Remote** | **Tailscale** or WireGuard; Immich apps connect to `https://immich.home` via VPN or reverse proxy with TLS. |
| **Files/docs** | Add **Nextcloud** (good mobile apps) or use Immich for photos/video only. |
| **Hardware** | N100 mini PC, 16 GB RAM, 2–4 TB storage. |
| **Cost** | $400–900 new build + drives. |

**Capabilities**
- Background upload (iOS/Android), face/search, map, shared libraries.
- No vendor subscription; you control retention and backup.

**Tradeoffs**
| Pros | Cons |
|------|------|
| Best native dual-platform photo UX in self-hosted space | You operate updates/backups |
| Immich apps feel “like Google Photos” | General file sync needs second app (Nextcloud) |
| Strong remote via Tailscale | Initial server setup |

**Technical fit:** Top pick when **native photo apps on both platforms** matter most.

---

## Solution B — Synology NAS + Synology Photos

| | |
|---|---|
| **What it is** | Commercial NAS with **Synology Photos** mobile apps (iOS/Android)—backup, albums, shared space. |
| **Multi-user** | DSM users; Photos shared folders; family invite in app. |
| **Remote** | Synology mobile apps; QuickConnect relay (easy) or VPN (better privacy). |
| **Cost** | $550–1200 unit + drives. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Polished official apps both platforms | Less AI/search depth than Immich |
| Lowest ops for appliance NAS | Hardware premium; vendor tie-in |
| Respectable “native app” feel | QuickConnect is a relay unless VPN |

**Technical fit:** Prefer appliance + official apps over DIY tuning.

---

## Solution C — Immich on Proxmox / existing home server

| | |
|---|---|
| **What it is** | Same Immich apps as A; colocated with Frigate/HA on one Proxmox host. |
| **Remote** | Same Tailscale pattern; one server for photos + security + automation. |
| **Cost** | $0 incremental if hardware exists. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| One box, one VPN for everything | Photo service downtime if host maintenance |
| Immich mobile UX unchanged | VM resource planning |

**Technical fit:** Unified homelab (see [solution-similarity-across-projects.md](./solution-similarity-across-projects.md)).

---

## Solution D — Nextcloud + native Nextcloud apps (files-first)

| | |
|---|---|
| **What it is** | Nextcloud iOS/Android apps for auto-upload camera roll to user folders; gallery view is usable but not Google Photos–class. |
| **Multi-user** | Excellent accounts/shares/quotas. |
| **Remote** | Nextcloud apps + Tailscale. |
| **Cost** | Same hardware as A. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Strong file sync + photos in one stack | Photo browsing weaker than Immich/Synology Photos |
| Single app for docs + camera upload | Family may prefer dedicated photo UX |

**Technical fit:** Photos secondary to document sync.

---

## Solution E — Hybrid: Immich (photos) + iCloud/Google one-way export (optional)

| | |
|---|---|
| **What it is** | Immich as primary self-hosted library; optional export to Apple/Google for household members who insist on platform gallery—**not** required for remote access. |
| **Cost** | Immich stack + optional cloud tiers. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| Satisfies “native” platform galleries too | Duplication; privacy split |

**Technical fit:** Mixed household preferences; usually skip if Immich apps suffice.

---

## Solution F — Cloud hybrid backup (B2/restic) under Immich

| | |
|---|---|
| **What it is** | Immich remains primary; encrypted offsite backup—does not change mobile app UX. |
| **Cost** | ~$6/TB/mo B2 + local hardware. |

**Tradeoffs**
| Pros | Cons |
|------|------|
| DR without second site | Egress on full restore |

---

## Mobile app comparison (what “native photo app preferred” means here)

| Stack | iOS app | Android app | Auto-upload | Timeline/AI | Remote without LAN |
|-------|---------|-------------|-------------|-----------|---------------------|
| **Immich** | Official | Official | ★★★★★ | ★★★★★ | ★★★★★ via VPN/TLS |
| **Synology Photos** | Official | Official | ★★★★ | ★★★★ | ★★★★★ |
| **Nextcloud** | Official | Official | ★★★★ | ★★★ | ★★★★★ |
| **PhotoPrism web** | PWA only | PWA only | ★★ | ★★★★ | ★★★★ |
| **WebDAV + OS gallery** | Files app | Third-party | ★★ | ★★ | ★★★ |

---

## Comparison matrix

| Solution | Native photo apps | iOS + Android parity | Remote access | Multi-user | Typical 4 TB |
|----------|-------------------|----------------------|---------------|------------|--------------|
| A TrueNAS + Immich | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | $700–1100 |
| B Synology Photos | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★ | $900–1400 |
| C Proxmox + Immich | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | $600–1500 |
| D Nextcloud-primary | ★★★ | ★★★★ | ★★★★★ | ★★★★★ | $700–1100 |
| F + offsite backup | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | +$25/mo |

---

## Recommended architecture

```
[iPhone Immich app] ──┐
[Android Immich app]──┼──► Tailscale ──► Immich (+ Postgres, Redis)
                      │                        │
[Remote PC browser] ──┘                        ▼
                                         ZFS pool / photos
                                                │
                                    restic ──► offsite (optional)
```

**Remote setup (both platforms):**
1. Tailscale on phones and server (MagicDNS optional).
2. Immich `EXTERNAL_DOMAIN` / public URL or split-horizon DNS.
3. Enable background upload: Wi‑Fi + charging; test iOS “Local Network” and Android battery exemptions.

**Security:** No public Immich without reverse proxy + auth; prefer VPN-only; separate DB backups.

**Why not PhotoPrism-only:** Strong web UI, but **no first-class native auto-upload apps** on both platforms—conflicts with your photo-app priority.

**Recommended path:** **Immich + Tailscale** on shared home server; add Synology only if you want zero Docker maintenance.
