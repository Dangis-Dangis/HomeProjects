# HomeProjects — Documentation & Agent Guide

Human-readable specs and **AI-facing guidance** for this repository. Cursor **Auto** should treat these as source of truth before editing project docs.

| Doc | Audience | Purpose |
|-----|----------|---------|
| [agent-guide.md](./agent-guide.md) | Cursor Auto / you | **Read → archive → write from scratch**; consistency checklist |
| [documentation-standards.md](./documentation-standards.md) | Authors + AI | Embedded images, diagrams, live-tree consistency |
| [domain/rgb-lights.md](./domain/rgb-lights.md) | Authors + AI | IR RGB strips (discrete swatches, 5 levels) |
| [domain/unified-remote.md](./domain/unified-remote.md) | Authors + AI | Volume dial, screen actions, fixed scene buttons |
| [prompts/common-tasks.md](./prompts/common-tasks.md) | You | Copy-paste prompts including supersession workflow |

**Machine-readable hooks**

| Location | Role |
|----------|------|
| [.cursor/skills/home-projects/SKILL.md](../.cursor/skills/home-projects/SKILL.md) | Project skill — domain + consistency policy |
| [.cursor/rules/home-projects.mdc](../.cursor/rules/home-projects.mdc) | Always-on rule — points here |
| [AGENTS.md](../AGENTS.md) | Root agent entry |

**Archive:** append new dated folders when superseding live content. Do not edit old archive generations in place to match new specs unless explicitly asked.
