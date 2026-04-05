# CRSP -- Clean Room Specs-First Protocol

> A reusable agent skill for generating code with minimized copyright/IP liability and verified functional correctness, built on the clean room design doctrine adapted for LLM-based multi-agent systems.

**Status:** Draft spec — Claude Code skill and normative specs evolving in-repo  
**Date:** 2026-04-04  
**(c) 2026 Jim Moraga <jim.moraga@pip.global>. All rights reserved.**

---

## Why This Exists

When an AI agent generates code that reimplements existing functionality -- replacing a proprietary dependency, targeting API compatibility, or rewriting a competitor's feature -- there is material risk of copyright infringement through verbatim reproduction, substantial similarity, or license contamination.

The **clean room design doctrine** provides a proven legal defense: if the person (or agent) writing the code has never seen the original and works only from a behavioral specification, the result is an independent creation. This doctrine has held up in court for decades (*NEC v. Intel*, *Sega v. Accolade*, *Sony v. Connectix*, among others).

CRSP adapts this doctrine for multi-agent AI systems. The core insight is that LLM agents make clean room separation both easier (you can strictly control each worker's context window) and harder (the underlying model may have memorized the very code you're trying to avoid). CRSP addresses both sides.

## The Dual Mandate

This protocol guarantees two things:

1. **The code works.** It satisfies behavioral contracts, passes multi-tier testing, and traces back to requirements.
2. **The code is independently created.** No worker that writes code has ever seen the original source, and there is an auditable evidentiary trail proving it.

Most existing approaches focus on one or the other. CRSP treats functional correctness and IP independence as equally weighted, non-negotiable outcomes.

## Philosophy

- **Specs first, always.** No code is written until behavioral contracts and test cases exist. The specification IS the product -- code is just its expression.
- **Walls are infrastructure, not policy.** Information barriers between workers are enforced by the Supervisor through payload validation and routing rules, not by asking agents to behave. If it's not enforced, it's not a wall.
- **Testing is not verification -- it's the definition of done.** A stub that passes similarity checks is not an implementation. Code must be executed against specs to count as complete.
- **Iterative by design.** Rework loops, upward queries, and budget controls are architectural features, not failure modes.
- **Pragmatic, not paranoid.** Trivial functions where only one reasonable implementation exists (merger doctrine) should not burn cycles trying to look different. The protocol adjusts its strictness based on the creative expression actually at stake.
- **Copyright class ≠ risk class.** A requirement can be Trivial for similarity thresholds yet High for security; test depth follows the stricter of the two (`docs/specs.md` sections 2.0 and 5.3).

## Formal IPC language (planned direction)

Worker handoffs today are described procedurally (walls, templates, heuristics). A **normative formal language for IPC**—a closed set of message kinds, typed fields, and **canonical serialization** (for hashing and audit)—would harden the Supervisor: malformed payloads and many wrong-shape failures are rejected **before** relying on natural-language discipline alone. **Natural-language specs stay first-class:** behavioral text lives in designated UTF-8 fields (any natural language); the formal layer carries structure, IDs, and routing metadata. That combination is **language-agnostic** for prose while remaining machine-checkable.

Formal schemas are **strong on structure and routing**; they **do not fully solve** semantic leaks (e.g. implementation detail smuggled inside an allowed string field). CRSP would keep **additional validators** (denylists, classifiers, optional human gates) on top. Roadmap item: future sprint in [`docs/specs.md`](docs/specs.md) (section 7, Future sprints).

## Project Approach

This is an iterative, Agile project. **Normative architecture and constraints** live in [`docs/specs.md`](docs/specs.md). **Operational playbook for Claude Code** lives in [`.claude/skills/crsp/SKILL.md`](.claude/skills/crsp/SKILL.md) (invoke `/crsp`). Hands-on gaps and lessons from a real run are captured in [`docs/crsp-field-notes.md`](docs/crsp-field-notes.md). A **documentation index** (what each file is for) is in [`docs/README.md`](docs/README.md). Detailed design, schemas, and a full Supervisor implementation will emerge through sprint work.

## Repository Structure

```
CoderHelper/
├── README.md                 # This file
├── archive/                  # Local-only snapshots (gitignored); e.g. old specs.bak
├── docs/
│   ├── README.md                       # Index: which doc to read when
│   ├── specs.md                        # High-level specifications (normative)
│   ├── crsp-field-notes.md             # Operational learnings from a hands-on run
│   └── crsp-ip-measurement-analysis.md # IP similarity structure, rename evasion, legal framing
└── .claude/
    └── skills/
        └── crsp/
            ├── SKILL.md      # Claude Code skill (Supervisor playbook)
            └── references/
                └── templates.md
```

Planned **future** layout (supervisor code, workers, validators) remains TBD; see `docs/specs.md` section 7.

## Current Status

- [x] Problem framing and legal basis
- [x] High-level architecture (workers, walls, channels, loops)
- [x] Testing protocol (4 tiers, dual mandate)
- [x] High-level specs ([`docs/specs.md`](docs/specs.md), v0.4-DRAFT)
- [x] Claude Code skill + templates ([`.claude/skills/crsp/`](.claude/skills/crsp/))
- [x] Field notes from a hands-on variant ([`docs/crsp-field-notes.md`](docs/crsp-field-notes.md))
- [ ] Detailed specs per component
- [ ] Framework selection (LangGraph vs. bare API vs. custom)
- [ ] Execution sandbox design
- [ ] Prototype (happy path, single simple function)
- [ ] Similarity tooling evaluation
- [ ] Cost/latency profiling
- [ ] Normative formal IPC schema + validators (future sprint; `docs/specs.md` section 7)

## Known Limitations

1. **LLM training data.** CRSP controls the prompt, not the model weights. The Coder's LLM may have memorized the original. Similarity scanning (Auditor) is detection after the fact, not prevention. **Context walls do not equal weight isolation** — see `docs/specs.md` section 1.1.
2. **Physical isolation.** If reference code remains on the same filesystem as the Coder, policy (“don’t look”) is weak; the spec requires technical isolation (`docs/specs.md` section 3.0).
3. **Not legal advice.** The protocol creates an evidentiary record of independent creation. It reduces risk. Consult IP counsel for high-stakes engagements.
4. **Cost.** Minimum 5 LLM calls per run, realistically 10-30+ with rework. Needs profiling.

## License

(c) 2026 Jim Moraga <jim.moraga@pip.global>. All rights reserved.
