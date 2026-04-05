# CRSP documentation map

This folder mixes **normative spec**, **empirical notes**, and **analysis**. Use this file to choose the right document.

| Document | Layer | Audience | Notes |
| --- | --- | --- | --- |
| [`specs.md`](specs.md) | **Normative** | Implementers, legal/tech reviewers | Single source of truth for architecture, walls, tiers, thresholds (version in header). |
| [`crsp-field-notes.md`](crsp-field-notes.md) | **Empirical** | Protocol authors | What broke or was missing in a real run; informed spec v0.4. |
| [`crsp-ip-measurement-analysis.md`](crsp-ip-measurement-analysis.md) | **Analysis** | Authors, tooling designers | Auditor metrics, rename evasion, high-level copyright framing (not legal advice). |

**Local archive (not in git):** The repo root [`archive/`](../archive/) directory is listed in [`.gitignore`](../.gitignore). Use it for disposable snapshots (e.g. `specs.bak`). Clones do not include it; it is **not** a source of truth.

**Outside `docs/` but part of the same story**

| Location | Layer | Notes |
| --- | --- | --- |
| [README.md](../README.md) | **Project** | Vision, status, repo layout, pointers here. |
| [`.claude/skills/crsp/SKILL.md`](../.claude/skills/crsp/SKILL.md) | **Operational** | Claude Code Supervisor playbook; should track `specs.md`. |
| [`.claude/skills/crsp/references/templates.md`](../.claude/skills/crsp/references/templates.md) | **Operational** | Artifact templates aligned with the spec. |
| [`src/crsp-v0-3/SKILL.md`](../src/crsp-v0-3/SKILL.md) | **Historical** | v0.3-era skill snapshot; superseded by `.claude/skills/crsp/`. See [`src/crsp-v0-3/README.md`](../src/crsp-v0-3/README.md). |

**Curation principles**

1. **Normative changes** → edit `specs.md` first, then align skill/templates and bump spec version when the contract changes.
2. **Lessons from runs** → `crsp-field-notes.md` or new dated note; fold stable conclusions into `specs.md`.
3. **Deep dives** (metrics, law, tooling comparisons) → separate `docs/crsp-*.md` analysis files; link from here and optionally one line in `specs.md` §7 if it drives work.
