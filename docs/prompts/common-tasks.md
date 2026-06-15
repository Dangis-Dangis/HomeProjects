# Common Task Prompts

Copy-paste starters for Cursor (Auto or pinned agent). Adjust bracketed parts.

**Default policy:** [agent-guide.md](../agent-guide.md) — read → archive → write from scratch; root must stay consistent.

---

## Supersede a requirement (remote, RGB, architecture)

```
Read docs/agent-guide.md and run the consistency checklist.
Requirement change: [describe].

1. Grep the live tree (exclude archive/) for stale references.
2. If whole live files are obsolete, move them to archive/YYYY-MM-DD-vN-[name]/ with a short README.
3. Rewrite every affected live doc from scratch: needs/, architecture/, portfolio/, projects/*, docs/domain/*, root README.md.
4. Do not edit old archive files in place.
5. Embed assets per documentation-standards.md.
```

---

## Update a project recommendation

```
Read docs/agent-guide.md and needs/constraints-and-inventory.md.
Update projects/[NN-name]/ and grep the live tree for anything that still contradicts [change].
Rewrite affected files wholly — don't add 10 lines to one README.
Embed relevant images from assets/ per docs/documentation-standards.md.
```

---

## Extend RGB IR catalog

```
Read docs/domain/rgb-lights.md.
I learned these IR codes for zone [1/2/3]: [list swatches and brightness steps].
Update the per-zone table and HA mapping in projects/03-home-control/.
Grep live docs for any RGB control language that still implies HSV 0-255.
```

---

## Remote hardware change

```
Read docs/domain/unified-remote.md and projects/05-unified-remote/.
[Describe change]. Keep volume-only dial; fixed Sync Screens / ALL ON / GOODNIGHT.
Rewrite wiring.md GPIO table if pins change; grep live tree for encoder/dial counts.
Embed wiring diagrams from assets/; note legacy images if they show 3 encoders.
```

---

## Add a concept image to docs

```
Generate [description] and save to assets/concept-[name].png.
Embed it in projects/[path].md with alt text.
Update assets/README.md inventory.
If the image supersedes old UX, label legacy art in concepts.md.
```

---

## Promote content from archive

```
Compare archive/2026-05-31-v2-home-systems-proposal/home-systems-proposal/solutions-[NN].md
with projects/[NN]/README.md.
Port [specific section] into a rewritten live README that stands alone.
Respect docs/domain/* — do not resurrect Party button, 3 dials, or HSV RGB.
```

---

## Full consistency audit

```
Audit the live tree (exclude archive/) per docs/agent-guide.md checklist:
- root README, needs/, architecture/, portfolio/, projects/*, docs/domain/*
- grep for Party Time, Night Time, 3 reusable, hue/sat dials, HSV 0-255
- cost table alignment across README and portfolio
- documentation-standards.md: heroes, diagrams, embedded images
Fix by rewriting inconsistent live files, not patching archive.
```
