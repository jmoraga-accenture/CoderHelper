---
name: crsp-v0-3
description: >-
  Applies CRSP v0.3 (Clean Room Specs-First Protocol): dirty/spec/clean rooms,
  Wall 1 and Wall 2, supervisor FSM with locked workers and audit log, five
  workers, four-tier testing, traceability, IP plus quality audit, rework
  routing and budgets. Use for clean-room code generation, API reimplementation,
  replacing proprietary deps, FuncSpec/ImplSpec/compliance artifacts, or when
  the user mentions CRSP, walls, Analyst/Specifier/Coder/Tester/Auditor, or
  minimized copyright or training-data risk.
---

# CRSP v0.3 — Clean Room Specs-First Protocol

**Vision:** Reduce copyright and IP liability when agents reimplement behavior, replace dependencies, or target API compatibility — by separating **knowledge of existing code** from **writing new code** via typed IPC and supervisor routing.

Authoritative detail: `README.md` (vision, boundaries, limitations) and `docs/specs.md` (lifecycle, FSM, validators, evidence). This file is the agent playbook.

**Not legal advice.** The protocol supports an evidentiary story for independent creation; engage IP counsel for high-stakes work.

## Rooms and walls

```
DIRTY ROOM          SPEC ROOM           CLEAN ROOM
(sees source)       (specs only)        (ImplSpec + code only)

  ANALYST ─FuncSpec─> SPECIFIER ─ImplSpec─> CODER
                         │                    │
                         └──── specs ────────> TESTER ──> AUDITOR
                              (Tester: ImplSpec + code; Auditor: all artifacts for compare)
```

| Wall | Boundary | Crosses | Must NOT cross |
|------|-----------|---------|----------------|
| **Wall 1** | Analyst → Specifier | `FunctionalSpec` only | Code, copied identifiers, implementation hints |
| **Wall 2** | Specifier → Coder | `ImplementationSpec` only | Provenance log, references to original source |

Enforcement: Supervisor validates payloads before forward routing.

## Workers (sees → produces → constraint)

| Worker | Sees | Produces | Key constraint |
|--------|------|----------|----------------|
| **Analyst** | Source, docs, APIs, references | FuncSpec (behavioral, code-free) | MUST NOT emit code or copy identifiers |
| **Specifier** | FuncSpec only | ImplSpec (contracts, tests, strategies) | MUST NOT see source; propose alternative approaches where needed |
| **Coder** | ImplSpec only | Source + tests (TDD / RGR) | MUST NOT see source, FuncSpec, or provenance |
| **Tester** | ImplSpec + Coder output | Test results + traceability matrix | MUST execute code; detect stubs |
| **Auditor** | All artifacts (for comparison) | Compliance report (IP + quality) | Quality (Tier 3) separate from similarity/license |

Supervisor: FSM orchestration, wall rules, loop budgets, escalation to human.

## Supervisor duties (operational)

- Spawn workers with **locked system prompts** (workers do not relax their own constraints).
- **Validate** every message against routing rules and payload schemas before forwarding.
- Track loop counters vs budgets; transition FSM on pass/fail/rework.
- Maintain an **append-only audit log** of IPC for traceability.
- May expose the protocol as a **single callable tool/skill** to a parent agent.

## Goals (what v0.3 fixes vs naive pipelines)

- **Upward queries** at every wall; Supervisor mediates.
- **Red/Green/Refactor** inside phases (Coder per contract; Tester tier gates).
- **Four-tier testing** with numeric gates and stub detection — not vague “add tests.”
- **Auditor:** functional/code quality **and** IP compliance.
- **Traceability:** requirement → contract → tests → code, with coverage gates.

## Channels and message types

**Forward:** Analyst → Specifier → Coder → Tester → Auditor.

**Upward (mediated):** Specifier→Analyst `CLARIFY_FUNC_SPEC`; Coder→Specifier `CLARIFY_IMPL_SPEC`; Tester→Specifier `CLARIFY_CONTRACT`; Auditor→Specifier `CLARIFY_INTENT`; response `CLARIFICATION_RESP`.

**Rework:** `REWORK_CODE`, `REWORK_SPEC`, `REWORK_FUNC_SPEC` (rare). **Terminal:** `ESCALATION` → human.

**Forward payload names (enum alignment):** `functional_spec`, `implementation_spec`, `code_output`, `test_results`, `compliance_report` — see `docs/specs.md` §3.4.

Ask upstream about **spec meaning** only; never leak Coder implementation (algorithms, structures, dirty-side detail) through clarify or rework to the clean side.

## Rework routing (critical)

| Source | Signal | Route |
|--------|--------|--------|
| Tester | Code bug | Coder — facts only, **no fix suggestions** |
| Tester | Spec gap / undefined behavior | Specifier, then re-implement |
| Auditor | Quality fail | Coder (refactor, same spec) |
| Auditor | IP / similarity fail | Specifier (abstract re-spec) — **never** direct to Coder with similarity detail |

IP rule: Coder must not learn what to “avoid.” Specifier applies abstract divergence (e.g. different algorithmic family for contract Cx) without original snippets.

## Testing tiers and gates

| Tier | Owner | Gate |
|------|--------|------|
| 0 | Tester | Contract tests from ImplSpec; stub detection; **100% pass** |
| 1 | Tester | Generated boundaries, negatives, interactions, round-trips; **100% pass** |
| 2 | Tester | Properties from `test_strategy.tier_2_properties`; 100–1000 examples, shrink on fail |
| 3 | Auditor | Lint (zero errors), types (strict, zero errors), complexity/docs/security/maintainability per `docs/specs.md` §4.5 / §7.5 |

**Tier 0 failure classes:** `STUB_DETECTED`, `WRONG_OUTPUT`, `EXCEPTION`, `TIMEOUT`, `SIGNATURE_MISMATCH`.

**Stub heuristic (conceptual):** `NotImplementedError`; identical output across ≥3 diverse inputs; outputs only zero-values.

## Traceability matrix

After Tier 2, map each `requirement_id` → `contract_id` → tier test ids → `code_refs`. **Gates:** `requirements_untested` = 0; `requirements_fully_tested / requirements_total` ≥ 0.9 (configurable); partials flagged for human review.

## FSM (essential path)

`INIT` → `ANALYZING` → `SPEC_REVIEW` ↔ `SPECIFYING` → `SPEC_APPROVAL` (optional) → `IMPLEMENTING` → `TESTING_T0` → `TESTING_T1` → `TESTING_T2` → `QUALITY_CHECK` → `IP_AUDIT` → `PASSED`. Rework states: `CODE_REWORK` (often restart T0), `SPEC_REWORK`, `IP_REWORK`. Budget exceeded → `FAILED_ESCALATE`. Full table: `docs/specs.md` §6.

## Loop budgets (defaults)

`max_code_rework_per_contract` 3; `max_code_rework_total` 10; `max_spec_rework` 3; `max_ip_rework` 2; `max_clarifications_per_wall` 5; `max_total_llm_calls` 50; `max_wall_clock_seconds` 600.

## Artifact contents (checklists)

**FunctionalSpec (Analyst):** Behavioral FRs (inputs, outputs, constraints, edges), NFRs, **provenance log** (sources consulted), **patent_flags**, **complexity class per requirement** (`TRIVIAL` / `STANDARD` / `COMPLEX`).

**ImplementationSpec (Specifier):** Interfaces with **independently chosen names**; behavioral contracts (pre/post, invariants); test cases per contract; `test_strategy` (tiers, property defs, generators); `error_contracts`; `performance_contracts` where relevant; **dependency allowlist** with licenses; alternative architectures for non-trivial choices.

**Compliance report (Auditor):** Token/AST similarity per module, identifier collisions, license audit, patent flags, quality metrics, overall pass/fail/conditional.

## Role reminders

**Analyst:** Code-free output; complexity feeds Auditor thresholds.

**Specifier:** Specifier test cases are a **floor** — Tester must exceed them at Tier 1.

**Coder:** Per-contract RED → GREEN → REFACTOR; report contract status; no skeleton-then-fill.

**Tester:** Run code; include `execution_log`, per-test `duration_ms` and `actual_output`; Tier 2 bundle includes traceability. Do not coach the Coder in failure payloads.

**Auditor:** Tier 3 mandatory vs advisory per spec; IP thresholds strictest for `COMPLEX`, relaxed for `TRIVIAL` (merger doctrine). Patent: flag, do not auto-reject.

## Rework / IP payload rules

IP divergence: no original source snippets; no line references into original (`line N`, `L123`, `:42:` patterns). Rework must cite `contract_ref` / `contract_refs`. See `docs/specs.md` §9.2.

## Legal basis (summary)

Independent creation from functional spec aligns with clean-room precedent (*NEC v. Intel*, *Sega v. Accolade*, *Sony v. Connectix*). Idea vs expression (*Baker v. Selden*, 17 U.S.C. §102(b), *Oracle v. Google*). Similarity framing: AFC (*Altai*); merger doctrine for trivial expressions (*Altai*, *Lotus v. Borland*). Evidence ledger in spec §10 expands citations.

## Known limitations (README + spec)

- **Model weights:** Protocol controls prompts, not training data; similarity phase is detection, not prevention. Prefer models with training transparency for strongest story.
- **Merger / trivial code:** Near-identical output can still be independent; complexity + relaxed thresholds help; humans judge edge cases.
- **Cost:** Minimum ~5 LLM calls per full pass; **10–30+** realistic with rework — profile real tasks for envelopes.
- **Sandbox:** Tester executes untrusted generated code — needs isolated runtime (Docker, Firecracker, WASM) and limits; security design is separate from spec prose.
- **Open questions:** Similarity tooling choice (`copydetect`, `jplag`, commercial); **model diversity** (Analyst vs Coder different LLMs?); **prompt injection** in reference material; partial delivery vs all-or-nothing; robust **performance** verification; framework binding (LangGraph vs bare API vs custom).

## Open implementation items (spec §11)

Sandbox hardening; noisy O()-style performance checks; partial success policy; cost model calibration; failure message minimalism so Tester is not a Coder crutch; prefer explicit FSM + typed envelopes over ad hoc sequencing.
