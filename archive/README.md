# Archive Index

Historical generations of this repo. **The live spec is at the repo root** (`needs/`, `architecture/`, `portfolio/`, `projects/`, `docs/`).

Archive **grows** when superseded live content is moved here — add dated folders; **do not rewrite old archive files** to match new specs (that erases history).

| Folder | Date | What it is |
|--------|------|------------|
| [2026-05-30-v1-exploratory-guides](./2026-05-30-v1-exploratory-guides/) | 2026-05-30 | First pass: per-topic solution comparisons (multiple options each), cross-project similarity docs, scored `fresh-suggestions/` |
| [2026-05-31-v2-home-systems-proposal](./2026-05-31-v2-home-systems-proposal/) | 2026-05-31 | Consolidated proposal: needs/constraints, problem statements, shared architecture, six solution dossiers with concept heroes; `concept-collection.md` (6 remote build directions) |
| [2026-06-14-v3-remote-and-assets](./2026-06-14-v3-remote-and-assets/) | 2026-06-14 | Unified remote deep-dive (3 reusable dials, Party/Night automation buttons, GPIO pinout, wiring) + **full image set** (20 concept renders, 6 collection renders, 4 wiring diagrams, 3 early concepts) |
| [2026-06-15-v4-superseded-remote-assets](./2026-06-15-v4-superseded-remote-assets/) | 2026-06-15 | Pre-regeneration copies of live `assets/` remote PNGs (still 3-dial / Party UX) before 2026-06-15 image regen |

## Adding a new archive generation

When live docs are superseded wholesale:

1. Create `archive/YYYY-MM-DD-vN-short-name/`
2. Move superseded files (or copies) into it
3. Add a one-paragraph `README.md` in that folder describing what was replaced
4. Update this index table
5. Rewrite live files at root from scratch

## Lineage

```
v1 exploratory  →  v2 proposal  →  v3 remote + assets  →  live root (2026-06-15+)  →  v4 superseded PNGs
     (many options)   (dossiers)      (3-dial remote UX)      (current spec)            (pre-regen art)
```

## Image locations

| Images | Path |
|--------|------|
| All concept + wiring PNGs (33 files) | `2026-06-14-v3-remote-and-assets/assets/` |
| Per-project hero images + collection duplicates | `2026-05-31-v2-home-systems-proposal/home-systems-proposal/assets/` |

Working copies for live docs: [../assets/](../assets/).

Some archived markdown may reference `./assets/` from an old root layout. Use the table above if a link breaks.

## Git history

- `03bf877` — Initial commit (2026-05-31)
- `9c07b32` — Updated images (2026-06-14)

Live root overhaul: 2026-06-15.

