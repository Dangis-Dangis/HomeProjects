# Documentation Standards

Every **live** project doc (`projects/`, `architecture/`, `needs/`, `portfolio/`) should be readable as a **visual spec**, not a wall of text.

**Consistency:** When one area changes, grep the **live** tree and update every affected doc — or move superseded files to `archive/`. See [agent-guide.md](./agent-guide.md).

---

## Required visual elements

| Element | When | How |
|---------|------|-----|
| **Hero / concept image** | Each `projects/*/README.md` | `![alt](./../../assets/heroes/concept-*.png)` or `collection-*.png` |
| **Flowchart** | Data path, agent workflow, connectivity | Mermaid in fenced ` ```mermaid ` block |
| **Architecture diagram** | Shared backbone, control rails | ASCII or Mermaid |
| **Wiring diagram** | Any hardware project (remote, ESP) | Embed `assets/wiring-*.png` + **authoritative pin table** in markdown |
| **UI state mockups** | Menus, screen labels | Embed `assets/concept-*-screen-*.png` where available |

**Rule:** If a PNG exists in `assets/` for the topic, **embed it** — don't only link the filename.

---

## Image paths

| From | Path pattern |
|------|----------------|
| `projects/0N-*/README.md` | `../../assets/...` |
| `projects/05-unified-remote/*.md` | `../../assets/...` |
| `architecture/`, `needs/`, `portfolio/` | `../assets/...` |
| `docs/` | `../assets/...` |

Inventory: [assets/README.md](../assets/README.md). Canonical archive copies: `archive/2026-06-14-v3-remote-and-assets/assets/`.

---

## Mermaid conventions

- Use `flowchart TD` or `LR` for pipelines (remote → HA → device).
- Use `sequenceDiagram` for feedback loops (button press → HA → LED ack).
- Keep node labels short; put detail in prose below the diagram.

**Example — remote command path:**

```mermaid
sequenceDiagram
  participant R as Remote
  participant HA as Home Assistant
  participant B as Broadlink
  participant L as IR RGB strip
  R->>HA: Wi-Fi command (color index, brightness level)
  HA->>B: IR code for preset
  B->>L: IR burst
  HA-->>R: ack + optional state
```

---

## Markdown structure per project

```markdown
# Title
![Hero](../../assets/...)

**Problem:** link to needs/problems.md

## Recommended
(table: cost, setup, feasibility)

## How it works
(diagram + short prose)

## Deep dive
(links to archive only — live README must still stand alone)
```

---

## Consistency (live tree only)

| Do | Don't |
|----|-------|
| Rewrite affected live files **from scratch** when the control model changes | Patch one paragraph and leave contradictory sections above |
| Grep live paths for stale terms after a change | Assume one file is the only source of truth |
| Move superseded live docs → `archive/YYYY-MM-DD-vN-name/` | Edit old archive files to match new specs |
| Label legacy concept art when UI changed | Present archive UX as current without a disclaimer |

---

## What not to do

- Bare filename tables without embedded images when PNGs exist.
- Continuous 0–255 RGB / HSV language for **IR accent strips** (wrong model).
- Leaving the repo root inconsistent for a first-time external reader.
- Placeholder images or lorem — use existing `assets/` or note `[image TBD]` with generation prompt in a comment block for the author only (avoid in user-facing prose).

---

## After generating new images

1. Save to `assets/` with descriptive name (`concept-`, `wiring-`, `heroes/`).
2. Embed in the relevant markdown immediately.
3. Add one line to `assets/README.md` inventory table.
