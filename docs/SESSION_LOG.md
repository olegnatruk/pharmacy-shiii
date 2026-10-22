# Session Log (shared)

A dated, **append-only** log of changes that affect **more than one member**: changes to `common/`, contracts, data formats, ADRs, and integration milestones. Each member's own day-to-day work goes in their `<name>/docs/SESSION_LOG.md`.

**Rules**
- Add new entries at the **bottom**. Never rewrite or delete old ones.
- Write in prose: what changed, why, and who is affected. Link the PR, commit, or ADR instead of repeating its contents. This is not a copy of `git log`.
- One entry per change or work session. Keep entries short.

Format:
```
## yyyy-MM-dd — <member> — <short title>
What changed and why. Who needs to do something about it. Links.
```

---

## 2026-10-22 — Setup — Project documentation created
The repository was created with documentation only, and no Java code yet. Contents:
- [README](../README.md), [AGENTS.md](../AGENTS.md), [PROJECT-BRIEF](../PROJECT-BRIEF.md)
- The shared `docs/` folder: architecture and contracts, data formats, features, validation, decisions, conventions, gotchas
- One folder per member (`heather/`, `karlo/`, `cameron/`), plus `common/`, each with agent context docs

Scope decisions ADR-001 to ADR-015 are recorded in [DECISIONS.md](DECISIONS.md), including:
- no branches
- one lot per medicine
- pharmacists see only their own orders
- seeded `admin` account
- deactivate instead of delete

Next: Milestone 0 in the [README](../README.md#build-order).

## 2026-10-22 — Setup — Blueprint, git guide, member prompts
Added three things:
- [BLUEPRINT.md](BLUEPRINT.md): every file, with its owner and milestone. It replaces the folder trees that used to be in ARCHITECTURE §2 and each CONTEXT.md.
- [GIT-GUIDE.md](GIT-GUIDE.md) and a PR template in `.github/`.
- `<member>/docs/PROMPTS.md`: step-by-step agent prompts with the git commands for each step.

Two new files are planned: `karlo/.../ui/MedicineTable.java` and `cameron/.../model/SalesReport.java`. The merge order across members is in GIT-GUIDE §9.

## 2026-10-22 — Setup — Solo mode added
Added a second way to work, so each member can finish their part with one prompt and no waiting ([ADR-016](DECISIONS.md)):
- [SOLO-MODE.md](SOLO-MODE.md): local stand-ins in `stubs/` (gitignored), the `integration` branch, and the integration prompt.
- `<member>/docs/SOLO-PROMPT.md`: one prompt per member, plus git commands.
- [ARCHITECTURE §6](ARCHITECTURE.md#6-cross-member-contracts) now states that menus, services and repositories have public no-argument constructors.

The team picks step mode or solo mode, not both.
