---
name: crsp
description: >-
  Clean Room Specs-First Protocol (CRSP): behavioral specs before code, strict
  information walls between analyst/specifier/coder/tester, TDD, tiered testing
  with stub detection, traceability, and IP-quality audit. Use for clean-room
  reimplementation, API-compatible rewrites, minimizing copyright risk, or when
  the user names CRSP, clean room, or specs-first protocol.
argument-hint: "[task description] [paths to reference material]"
---

# CRSP — Clean Room Specs-First Protocol

Normative detail: `docs/specs.md` (repo root; from this skill: `../../../docs/specs.md`). Field learnings: `docs/crsp-field-notes.md`. This skill is the **operational playbook** for Claude acting as **Supervisor** and sequential workers.

## Dual mandate (non-negotiable)

1. **Correctness:** Behavior matches contracts; tests run and pass; no stubs masquerading as implementations; **every enumerated deliverable** present (section 5.5).
2. **Independent creation:** No worker that **writes code** may see original source, functional spec, or provenance. Enforce **phase isolation**, **payload rules**, and **physical isolation** of reference material from the Coder (see below).

## Context vs. weights (framing)

Walls enforce **context isolation**, not **weight-level** clean room: the same model may have memorized reference code. Be explicit in compliance narratives. Prefer **different models/providers** for Analyst vs. Coder when stakes are high (`docs/specs.md` section 1.1).

## When to apply

- Reimplementing or replacing existing code (proprietary, competitor, or unknown license).
- Targeting behavioral or API compatibility without copying expression.
- User explicitly requests clean room, CRSP, or specs-first with IP caution.

**Not legal advice.** Document limitations (isolation mechanism, same-model use, tool gaps) in the compliance report.

---

## Threat model (summary)

| Vector | Mitigation |
| --- | --- |
| Verbatim / memorized code | Spec abstraction; auditor similarity checks |
| Substantial similarity | Decomposition; independent naming; AST-style comparison where tooling exists |
| License contamination | Allowlist + **tool-generated** transitive license listing for Auditor |
| Non-functional output | Pre-failing stub gate (when possible), execution, stub detection, traceability |
| Reference reachable on disk | **Physical isolation** — Coder workspace must not include reference paths |

---

## Physical isolation (Supervisor)

Before Coder work: ensure reference code is **not** in the Coder’s editable tree (separate repo, worktree without reference dirs, or excluded paths). Log the mechanism in the audit trail. “Just don’t open it” is not sufficient.

---

## Worker roles (what each phase may know)

| Role | May see | Must produce |
| --- | --- | --- |
| **Analyst** (dirty) | Sanitized reference (strip HTML/MD comments where feasible); full material for analysis | **Functional Spec** — behavior only; prose constraints instead of copied regex/constants/messages; **copyright** class (Trivial/Standard/Complex) **and** **risk** (Low/Medium/High) per requirement; provenance; patent flags |
| **Specifier** | Functional Spec only | **Implementation Spec** — contracts, tests, errors, allowlist, **no provenance**; ≥2 alternatives + choice for Standard/Complex **copyright**; independent names |
| **Coder** | Implementation Spec only | Code + tests (TDD); allowlist only; **CRSP-GENERATED** headers (`spec-hash` = SHA-256 of full Impl Spec UTF-8, **first 12 hex chars** — `docs/specs.md` section 6.5) |
| **Tester** | Impl Spec + Coder output | Executed tests, extra tests, stub check, **traceability matrix**; failures without fix hints; **not** the same principal as Specifier when avoidable |
| **Auditor** | Everything + **CLI license/dependency output** from Supervisor | **Compliance report** (quality + IP + **implementation limitations**); wall-safe rework |

---

## Information walls (enforce on every handoff)

### Wall 1 — Analyst ↔ Specifier

- **Forward:** Functional Spec + classifications. **Block:** code, pseudocode, source identifiers, provenance beyond “sources consulted,” implementation hints.
- **Reverse:** Clarifications about behavior/scope only.

### Wall 2 — Specifier ↔ Coder

- **Forward:** Implementation Spec (selected architecture only). **Block:** functional spec, provenance, any original reference.
- **Reverse:** Only **contract ID** + **spec ambiguity** — no implementation or runtime narrative.

**Reject (do not forward) Wall 2 reverse if the payload includes:** code fences or snippets; “I implemented / currently returns / throws”; “when I run / the test passes / I get …”; or similar leak patterns (`docs/specs.md` section 3). Allowed: “Contract C3 does not define behavior when `folder` is empty.”

### Wall 3 — Specifier ↔ Tester (via Supervisor)

- **Forward:** Same Impl Spec as Coder + Coder output. **Block:** original source, functional spec, provenance.
- **Reverse:** **Spec gaps** only — **block** code snippets and “the code does X.”

### Auditor outputs (wall-constrained)

- **IP failure → Specifier only** (abstract constraints; no similarity detail to Coder).
- **Quality failure → Coder** (lint/type/complexity; no original source).

---

## Supervisor checklist

1. **Inputs:** task, reference materials, target language, allowed licenses (default MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause), budgets/overrides.
2. **Sanitize** reference for Analyst (comments/odd blocks) where feasible; log gaps.
3. **Isolate** reference from Coder workspace; record how.
4. **Analyst** → validate Functional Spec (no parseable code; copyright + **risk** per requirement; behavioral vs. copied creative expression).
5. **Specifier** → validate Impl Spec (no provenance; ≥3 tests + error contract per interface; alternatives for Standard/Complex copyright; allowlist).
6. **Deliverable checklist** — closed list of every source + test + agreed file; **reject** submission if any item missing (`docs/specs.md` section 5.5).
7. **Pre-failing stub gate (recommended):** baseline with **N** failing tests; Coder confirms **N** failures before implementation; accept only when runner hits zero failures (`docs/specs.md` section 5.0).
8. **Coder** (TDD, allowlist, headers) → **Tester** (execute; stub detection; tiers; traceability).
9. Run **license/dependency CLI**; pass output to **Auditor**.
10. **Auditor** (quality + IP). **Partial delivery default:** all-or-not-passed; on budget exhaust, escalate with `BLOCKED` contracts and human summary (`docs/specs.md` section 4.3).
11. **Deliver:** code + tests, matrix, compliance report (including **context-isolation / same-model** caveats), audit summary, status.

Templates: [references/templates.md](references/templates.md).

---

## Analyst — hard rules

- **What**, not **how**; no language-specific syntax.
- No non-trivial copied **identifiers** or **creative literals** (unique regex, magic constants, distinctive messages). Describe enforced behavior in prose; Specifier reinvents expression.
- **Copyright class** and **risk class** per requirement (orthogonal).
- Provenance log; patent flags.

---

## Specifier — hard rules

- New names for all interfaces; no guessing originals.
- Preconditions, postconditions, invariants, error contract, **≥3** tests per contract; test strategy; performance when relevant; allowlist with SPDX ids.

---

## Coder — TDD and walls

- **Red / Green / Refactor**; confirm real failures (or pre-fail baseline, section 5.0).
- No external source search; allowlist only.
- Clarifications: **contract IDs** + spec gaps only — **never** implementation/runtime (Wall 2).
- **CRSP-GENERATED** header on every file (`docs/specs.md` section 6.5).

---

## Tester — execution and independence

- Execute code; stub heuristics at Tier 0; extra tests T1/T2; traceability; no fix hints in failure reports.
- Prefer **different session/principal** from Specifier for independence.

**Tiers:** 0 (spec + stubs) → 1 (functional) → 2 (property) → 3 (quality). Sequential gates.

**Test depth:** Apply the **stricter** of copyright-driven defaults and **risk** floors (`docs/specs.md` section 5.3). High-risk requirements get full depth even if copyright-Trivial.

**Traceability:** Untested requirements = failure; fully tested ratio ≥ **0.9** unless configured.

---

## Auditor — quality and IP

**Quality:** Lint/type strict; complexity flags; docs vs contracts; basic security.

**IP:** Token/AST/identifiers; **license audit using Supervisor-supplied tool output** (LLM-only transitive guessing is not enough).

**Similarity thresholds** (ignore for **Trivial** copyright): Standard — flag token >15%, AST >0.7; Complex — flag >8%, AST >0.5.

---

## Rework routing (critical)

| Trigger | Route to |
| --- | --- |
| Implementation bug | Coder |
| Spec gap | Specifier, then Coder |
| IP similarity | **Specifier only** |
| Quality | Coder |

Cap loops per `docs/specs.md` sections 4.2 and 6.3; escalate with diagnostics.

---

## Philosophy

- Specs first; walls are infrastructure; **risk** can override shallow testing even when copyright is Trivial; pragmatic on merger doctrine for **similarity** only.

---

## Open implementation gaps

Framework, sandbox, similarity engines: `docs/specs.md` section 7. Until automated, use best-effort tools and **state limitations** in the compliance report.
