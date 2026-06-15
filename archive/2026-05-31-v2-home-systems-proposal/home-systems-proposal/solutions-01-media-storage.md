# Solutions — Media & Photo Storage

**Problem (P1):** A private home for photos, video, and personal files that is **maximally accessible remotely** from **iPhone and Android**, with a **native photo-app experience** preferred over web-only admin panels. Also: a central store for home video and personal files; multi-user; background phone backup on Wi‑Fi.

This collection lists every viable option with capabilities, trade-offs, and technical fit, then scores the field and names a recommendation.

---

## Solution A — TrueNAS SCALE (or N100 Docker) + Immich

| | |
|---|---|
| **What it is** | Self-hosted **Immich** — the closest open-source equivalent to a cloud photo service — with official **iOS and Android apps** (auto-upload, timeline, search, albums, partner sharing). |
| **Multi-user** | Per-user libraries, shared albums, partner link. |
| **Remote** | **Mesh VPN** (Tailscale/WireGuard); Immich apps connect to `https://immich.home` via VPN or a reverse proxy with TLS. |
| **Files/docs** | Add **Nextcloud** (good mobile apps) or use Immich for photos/video only. |
| **Hardware** | N100 mini PC, 16 GB RAM, 2–4 TB storage. |
| **Cost** | $400–900 for a new build + drives. |

**Capabilities**
- Background upload (iOS/Android), face/search, map, shared libraries.
- No vendor subscription; the household controls retention and backup.

**Tradeoffs**

| Pros | Cons |
|------|------|
| Best native dual-platform photo UX in the self-hosted space | You operate updates/backups |
| Immich apps feel like a mainstream cloud gallery | General file sync needs a second app (Nextcloud) |
| Strong remote access via mesh VPN | Initial server setup |

**Technical fit:** Top pick when **native photo apps on both platforms** matter most.

---

## Solution B — Synology NAS + Synology Photos

| | |
|---|---|
| **What it is** | Commercial NAS with **Synology Photos** mobile apps (iOS/Android) — backup, albums, shared space. |
| **Multi-user** | DSM users; Photos shared folders; family invite in-app. |
| **Remote** | Synology mobile apps; QuickConnect relay (easy) or VPN (better privacy). |
| **Cost** | $550–1,200 unit + drives. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Polished official apps on both platforms | Less AI/search depth than Immich |
| Lowest ops for an appliance NAS | Hardware premium; vendor tie-in |
| Respectable native-app feel | QuickConnect is a relay unless you use VPN |

**Technical fit:** Prefer an appliance + official apps over DIY tuning.

---

## Solution C — Immich on Proxmox / a shared home server

| | |
|---|---|
| **What it is** | The same Immich apps as Solution A, colocated with Frigate/hub on one Proxmox host. |
| **Remote** | Same mesh-VPN pattern; one server for photos + security + automation. |
| **Cost** | $0 incremental if the hardware already exists. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| One box, one VPN for everything | Photo-service downtime during host maintenance |
| Immich mobile UX unchanged | VM resource planning |

**Technical fit:** A unified homelab (see [02-shared-architecture.md](./02-shared-architecture.md)).

---

## Solution D — Nextcloud + native Nextcloud apps (files-first)

| | |
|---|---|
| **What it is** | Nextcloud iOS/Android apps auto-upload the camera roll to user folders; the gallery view is usable but not cloud-gallery-class. |
| **Multi-user** | Excellent accounts/shares/quotas. |
| **Remote** | Nextcloud apps + mesh VPN. |
| **Cost** | Same hardware as Solution A. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Strong file sync + photos in one stack | Photo browsing weaker than Immich/Synology Photos |
| Single app for docs + camera upload | Family may prefer a dedicated photo UX |

**Technical fit:** Photos secondary to document sync.

---

## Solution E — Hybrid: Immich (photos) + iCloud/Google one-way export (optional)

| | |
|---|---|
| **What it is** | Immich as the primary self-hosted library; optional export to Apple/Google for household members who insist on a platform gallery — **not** required for remote access. |
| **Cost** | Immich stack + optional cloud tiers. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Satisfies native platform galleries too | Duplication; privacy split |

**Technical fit:** Mixed household preferences; usually skip if the Immich apps suffice.

---

## Solution F — Cloud hybrid backup (B2/restic) under Immich

| | |
|---|---|
| **What it is** | Immich remains primary; encrypted offsite backup — does not change the mobile app UX. |
| **Cost** | ~$6/TB/mo (e.g. Backblaze B2) + local hardware. |

**Tradeoffs**

| Pros | Cons |
|------|------|
| Disaster recovery without a second site | Egress cost on a full restore |

---

## Mobile-app comparison (what "native photo app preferred" means here)

| Stack | iOS app | Android app | Auto-upload | Timeline/AI | Remote without LAN |
|-------|---------|-------------|-------------|-------------|---------------------|
| **Immich** | Official | Official | ★★★★★ | ★★★★★ | ★★★★★ via VPN/TLS |
| **Synology Photos** | Official | Official | ★★★★ | ★★★★ | ★★★★★ |
| **Nextcloud** | Official | Official | ★★★★ | ★★★ | ★★★★★ |
| **PhotoPrism web** | PWA only | PWA only | ★★ | ★★★★ | ★★★★ |
| **WebDAV + OS gallery** | Files app | Third-party | ★★ | ★★ | ★★★ |

---

## Comparison matrix

| Solution | Native photo apps | iOS + Android parity | Remote access | Multi-user | Typical 4 TB |
|----------|-------------------|----------------------|---------------|------------|--------------|
| A TrueNAS + Immich | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | $700–1,100 |
| B Synology Photos | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★ | $900–1,400 |
| C Proxmox + Immich | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | $600–1,500 |
| D Nextcloud-primary | ★★★ | ★★★★ | ★★★★★ | ★★★★★ | $700–1,100 |
| F + offsite backup | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | +$25/mo |

---

## Recommended architecture

```
[iPhone Immich app] ──┐
[Android Immich app]──┼──► Mesh VPN ──► Immich (+ Postgres, Redis)
                      │                        │
[Remote PC browser] ──┘                        ▼
                                         ZFS pool / photos
                                                │
                                    restic ──► offsite (optional)
```

**Remote setup (both platforms):**
1. Mesh VPN on phones and server (MagicDNS optional).
2. Immich `EXTERNAL_DOMAIN` / public URL, or split-horizon DNS.
3. Enable background upload: Wi‑Fi + charging; test iOS "Local Network" permission and Android battery exemptions.

**Security:** No public Immich without a reverse proxy + auth; prefer VPN-only; keep separate DB backups.

**Why not PhotoPrism-only:** strong web UI, but **no first-class native auto-upload apps** on both platforms — which conflicts with the photo-app priority.

---

## Scored recommendation

**Top pick — Immich on N100/TrueNAS + mesh VPN.** The only self-hosted stack with **first-class native photo apps on both platforms** plus background upload. Add Nextcloud only if document sync is needed.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$500–1,100** built fresh: N100 mini PC $200–350, 16 GB RAM, 2× 4 TB ($160–300), case/PSU; or $0 software on an existing box. +$25/mo if B2 offsite is added. |
| **Setup time** | **1–2 days** (TrueNAS/Docker, Immich, mesh VPN, per-phone app enrollment). |
| **Maintenance** | **Med** — monthly container updates, watch DB backups, drive swap every 3–5 years. |
| **Feasibility** | ★★★★★ — squarely within the household's skill set. |
| **Scalability** | ★★★★★ — add drives/users freely; ZFS datasets per person. |

**Reuse:** shares the **N100 server** with Security (Frigate) and the **mesh VPN** with everything. One backup job (restic/kopia) covers `photos/` + `security/recordings/`.

**Alternate A — Synology + Synology Photos** ($900–1,400; setup 2–4 hrs; maintenance Low; feasibility ★★★★★; scalability ★★★★). Pick if you'd rather not run Docker; loses Immich-level search and the shared-server reuse with Frigate.

**Alternate B — Immich as a Proxmox VM** (same apps, colocated with Frigate + hub; **$0 incremental** if building the server anyway; scalability ★★★★★; maintenance Med–High). Best for a single box covering the whole portfolio.

**Verdict:** Build **Immich on the N100** as backbone item #2 (after the hub). It delivers the highest daily value and becomes the shared server for Security. Add the **mesh VPN** once and reuse it everywhere; add **B2 offsite** only when the library is worth insuring.

---

## Full dossier — recommended build (Immich on N100 + mesh VPN)

### Concept image

![Local self-hosted photo server syncing to phones](./assets/concept-media-storage.png)

### Parts list

| Part | Qty | Unit $ | Total |
|------|-----|--------|-------|
| N100 mini PC, 16 GB RAM, 512 GB NVMe boot (e.g. Beelink/Topton class) | 1 | $230 | $230 |
| 4 TB 3.5" NAS HDD (WD Red Plus / Seagate IronWolf) | 2 | $90 | $180 |
| USB 3 / SATA enclosure or bays (if the mini PC lacks 2× 3.5") | 1 | $45 | $45 |
| Mini UPS (DC, keeps the box up through brief outages) | 1 | $60 | $60 |
| Gigabit switch port + Cat6 patch (to existing router) | 1 | $8 | $8 |
| **Total (8 TB raw / 4 TB usable mirror)** | | | **~$523** |

Software (Immich, Postgres, Redis, Tailscale, restic) is free and open-source.

### Cost example

- **Starter (mirror, ~4 TB usable):** N100 16 GB $230 + 2× 4 TB $180 + enclosure $45 + UPS $60 ≈ **$515**.
- **Family (mirror, ~8 TB usable):** same box + 2× 8 TB ($340) ≈ **$675**.
- **Insured (family + offsite):** add Backblaze B2 at ~$6/TB/mo → ~$48/mo for 8 TB used (typically far less early on; you pay for stored GB).
- **$0 incremental** if Immich runs as a container on a server already built for the portfolio.

### Data path

```
[iPhone Immich app] ┐
[Android Immich app]┤
[Remote browser]    ┘
        │ background upload (Wi-Fi + charging) / browse
        ▼
   Mesh VPN (WireGuard via Tailscale) — no public exposure
        ▼
   Immich (web + ML) ── Postgres (metadata) + Redis (jobs)
        │
        ▼
   ZFS mirror dataset  photos/   (originals + generated thumbs)
        │
        └── restic (nightly, encrypted) ──► Backblaze B2 (optional offsite)
```

### General wiring / topology diagram

```
                 Internet
                    │
              ┌─────┴─────┐
              │  Router    │ (mesh VPN exit node optional)
              └─────┬─────┘
        Gigabit LAN  │
   ┌─────────────────┼───────────────────┐
   │                 │                    │
[Wi-Fi AP]      [N100 server]        [Admin PC]
   │             │   │   │
 phones      NVMe  HDD1 HDD2  ← ZFS mirror (photos/)
              boot  └───┬───┘
                     mini UPS ── wall outlet
```

### Effort to create

| Phase | Effort |
|-------|--------|
| Assemble box, install TrueNAS/Debian, create ZFS mirror | 2–3 hrs |
| Deploy Immich (Docker/compose), Postgres, Redis | 1–2 hrs |
| Mesh VPN on server + each phone; test remote | 1 hr |
| Per-phone app enrollment + background-upload permissions | 30 min/phone |
| Initial library import + face/CLIP indexing (runs unattended) | hours, hands-off |
| **Total active effort** | **~1–2 days** |

### Maintenance cost

- **Electricity:** ~10–18 W idle ≈ 90–160 kWh/yr ≈ **$15–30/yr** at typical US rates.
- **Time:** monthly container update + glance at backup logs ≈ **20–30 min/mo**; verify a test restore quarterly.
- **Parts:** plan a drive replacement every **3–5 yrs** (~$90 each); UPS battery ~5 yr.
- **Subscriptions:** none required; optional B2 offsite ~**$6/TB/mo** of data stored.

### Cost feasibility & scalability

- **Marginal storage cost is the main lever:** ~**$20–25 per usable TB** (mirrored) — add capacity by swapping to larger drives or adding a vdev; no software cost.
- **Compute is effectively free to scale:** the N100 handles many users and the ML jobs; adding users costs $0.
- **Cliff to watch:** moving from a 2-drive mirror to a larger pool may need a roomier case/HBA (a one-time ~$80–150 step). Offsite backup cost scales linearly with *used* GB, so it stays small until the library is large.
- **Takeaway:** low entry (~$500), gentle linear growth, no per-seat or per-GB vendor fees — the cheapest long-run path for a growing private library.

### Potential risks & downsides

| Risk | Mitigation |
|------|-----------|
| **A mirror is redundancy, not a backup** | Keep restic → B2 (or a second local copy); test restores |
| **Postgres corruption loses all metadata/albums** | Scheduled DB dumps separate from the photo dataset |
| **iOS background upload throttling** | Grant "Local Network" + disable Low Power during upload; verify periodically |
| **Single-box outage = no photos until fixed** | UPS for brief outages; keep install notes for fast rebuild |
| **Immich is fast-moving software** | Pin versions; read release notes before updating; monthly cadence |
| **Drive failure / bit rot** | ZFS scrubs monthly; SMART alerts; spare drive on hand |
