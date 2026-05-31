# 01 — Media & Photo Storage (Fresh Suggestion)

**Need:** Photos/video/files, maximally accessible from **iPhone + Android remotely**, **native photo apps**, multi-user.

Full option list: [../personal-media-storage.md](../personal-media-storage.md)

---

## Top pick — Immich on N100/TrueNAS + Tailscale

The only self-hosted stack with **first-class native photo apps on both platforms** plus background upload. Add Nextcloud only if you need document sync.

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$500–1,100** built fresh: N100 mini PC $200–350, 16 GB RAM, 2× 4 TB ($160–300), case/PSU; or $0 software on an existing box. +$25/mo if you add B2 offsite. |
| **Setup time** | **1–2 days** (TrueNAS/Docker, Immich, Tailscale, app enrollment per phone). |
| **Maintenance** | **Med** — monthly container updates, watch DB backups, drive swap every 3–5 yrs. |
| **Feasibility** | ★★★★★ — squarely in your wheelhouse. |
| **Scalability** | ★★★★★ — add drives/users freely; ZFS datasets per person. |

**Project similarity:** Shares the **N100 server** with Security (Frigate) and **Tailscale** with everything. One backup job (restic/kopia) covers `photos/` + `security/recordings/`.

---

## Alternate A — Synology + Synology Photos

| Dimension | Assessment |
|-----------|------------|
| **Cost range** | **$900–1,400** (unit + drives). |
| **Setup time** | **2–4 hrs.** |
| **Maintenance** | **Low.** |
| **Feasibility** | ★★★★★ (easy). |
| **Scalability** | ★★★★ (bay-limited; vendor tie-in). |

Pick if you'd rather not run Docker. Loses Immich-level search and shared-server reuse with Frigate.

---

## Alternate B — Immich as a VM on a Proxmox all-in-one

Same apps, colocated with Frigate + HA on one host. **Cost $0 incremental** if you build the server anyway; **Scalability ★★★★★**; **Maintenance Med–High** (you run the hypervisor). Best if you want a single box for the whole portfolio.

---

## Verdict

Build **Immich on the N100** as backbone item #2 (after HA). It delivers the highest daily value and becomes the shared server for Security. Add **Tailscale** once, reuse everywhere; add **B2 offsite** only when the library is worth insuring.
