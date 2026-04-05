# CRSP -- High-Level Specifications

**Version:** 0.4-DRAFT  
**Date:** 2026-04-04  
**(c) 2026 Jim Moraga <jim.moraga@pip.global>. All rights reserved.**

This document defines the architecture, components, interfaces, and constraints for the Clean Room Specs-First Protocol. It is intended as the basis for detailed specs, sprint planning, and implementation.

---

## 1. Threat Model

The protocol guards against three liability vectors:

| Vector | Risk | Mitigation |
|---|---|---|
| **Verbatim reproduction** | Agent outputs memorized copyrighted code | Specification abstraction + output similarity scanning |
| **Substantial similarity** | Agent produces structurally derivative code | Functional decomposition + independent synthesis + AST comparison |
| **License contamination** | Agent mixes incompatible license obligations | Dependency allowlist + transitive license audit |

And one correctness failure mode that prior versions of this spec did not address:

| Failure | Risk | Mitigation |
|---|---|---|
| **Non-functional output** | Agent produces stubs, no-ops, or partial implementations that pass IP checks | Multi-tier testing with mandatory execution + stub detection |

### 1.1 LLM memorization, context walls, and model diversity

Information walls and role separation enforce **context isolation**: the Coder’s prompt and attachments must not contain the original source, functional spec, or provenance. They do **not** provide **weight-level isolation**: the same underlying model may have memorized reference code during training. Similarity scanning is **detection after the fact**, not prevention of memorized output.

**Positioning for stakeholders:** The protocol should be described honestly. Where a single model instance (or same weights) participates in both analysis and coding—even across separate conversation sessions—the clean-room story rests on **process and evidence** (specs-first, independent naming, tests, audit trail), not on a claim that the model “has never seen” the original in any sense. **Model diversity** (different models or providers for Analyst vs. Coder) may strengthen the independence argument operationally; treat it as a **foundational design choice** for high-stakes use, not only a longer-term optimization. Detailed trade-offs remain in §7.

---

## 2. Worker Definitions

### 2.0 Copyright classification vs. risk classification

The Analyst assigns two **independent** axes per requirement:

| Axis | Values | Drives |
|---|---|---|
| **Copyright / creative expression** | Trivial, Standard, Complex | Similarity thresholds (§2.5); Specifier alternative architectures (Standard/Complex); default copyright-based test depth (§5.3) |
| **Risk** (security, safety, data integrity) | Low, Medium, High | Minimum test depth and Tier 2 obligation regardless of copyright class (§5.3) |

A requirement may be **Trivial** for copyright (merger doctrine: few valid implementations) but **High** for risk (e.g. input validation on a security boundary). **Risk overrides** that demand deeper testing always apply. The Specifier’s “two alternatives” obligation remains tied to **copyright** class (Standard/Complex), not to Low risk alone.

### 2.1 Analyst (Dirty Room)

**Purpose:** Extract behavioral understanding from reference material without passing through any code or implementation detail.

**Visibility:** Unrestricted -- original source, docs, APIs, examples, any reference material.

**Output:** Functional Spec.

**Constraints:**
- Must not emit code, pseudocode, or language-specific syntax.
- Must not copy identifiers (function names, variable names, class names, string literals) from source.
- Must describe behavior (what), never mechanism (how).
- Must log every source consulted (provenance log).
- Must flag potentially patented algorithms.
- Must classify each requirement by creative expression level: Trivial, Standard, or Complex (copyright axis; drives similarity thresholds and Specifier alternatives).
- Must classify each requirement by **risk**: Low, Medium, or High (security, safety, or integrity criticality; drives minimum test depth — §5.3).
- **Identifiers vs. behavioral constraints:** Must not copy non-trivial identifiers (names, distinctive literals) from source. When a constraint would force quoting **creative expression** from the reference (e.g. a unique regex, magic constant, or distinctive error message text), describe the **behavior enforced** in prose; the Specifier invents independent expressions (new names, new regex, new messages) in the Implementation Spec.

**Acceptance criteria (Supervisor validates before forwarding):**
- Payload contains no parseable code (validated by code detection heuristic + attempted AST parse).
- All required fields present: functional requirements, provenance log, copyright (Trivial/Standard/Complex) and risk (Low/Medium/High) classifications.
- Every functional requirement has at least: description, inputs, outputs, constraints.

### 2.2 Specifier (Spec Room)

**Purpose:** Translate behavioral requirements into testable engineering contracts without knowledge of the original implementation.

**Visibility:** Functional Spec only. Has never seen and must not request original source.

**Output:** Implementation Spec.

**Constraints:**
- Must choose independent names for all interfaces, functions, and types -- no guessing at original names.
- Must propose at least two alternative architectural approaches for non-trivial design decisions and select one with rationale. (This is a deliberate legal strategy: demonstrating independent creative expression strengthens the clean room defense.)
- Must specify behavioral contracts for every interface: preconditions, postconditions, invariants, and test cases (input -> expected output).
- Must specify error contracts: what happens on invalid input, resource exhaustion, etc.
- Must specify a test strategy per contract: which test tiers apply, what properties to verify, what generators to use for property-based testing.
- Must specify performance contracts where relevant (complexity class, benchmarks).
- Must specify a dependency allowlist with license info.
- Must not carry forward the provenance log (source references stay behind Wall 1).

**Acceptance criteria:**
- No provenance log or source references present.
- Every interface contract has at least 3 test cases.
- Every contract has an error contract.
- Alternative approaches documented for contracts classified Standard or Complex on the **copyright** axis (§2.0).
- Dependency allowlist present with licenses verified against approved list.

### 2.3 Coder (Clean Room)

**Purpose:** Implement code from the Implementation Spec alone, using TDD.

**Visibility:** Implementation Spec only. Has never seen the original source, the Functional Spec, or the provenance log.

**Output:** Source code + test suite.

**Constraints:**
- Must follow Red/Green/Refactor per contract:
  - **Red:** Write tests from the spec's test cases. Run. Confirm they fail. (If they pass without implementation, the test is vacuous.) When the Supervisor uses a **pre-failing stub artifact** (§5.0), the Coder’s first step is to run the suite and confirm the expected failures before implementing.
  - **Green:** Write the minimum code to make tests pass.
  - **Refactor:** Clean up while keeping tests green.
- Must not search for or reference external source code.
- Must use only dependencies from the spec's allowlist.
- Must send clarification requests (referencing contract IDs only) when the spec is ambiguous rather than guessing.
- Must report completion status per contract (complete, in progress, blocked).

**Acceptance criteria:**
- All spec test cases have corresponding test implementations.
- All tests pass locally (Coder self-reports; Tester will independently verify).
- No imports outside the dependency allowlist.
- CRSP-GENERATED header present on all files (protocol version, spec hash per §6.5, timestamp).

### 2.4 Tester (Verification Room)

**Purpose:** Independently verify that the Coder's output actually implements the spec. Detect stubs, no-ops, partial implementations, and edge case failures.

**Visibility:** Implementation Spec + Coder's output. Has never seen the original source.

**Output:** Test results + traceability matrix.

**Constraints:**
- **Independence:** The Tester (or testing principal) must not be the same party that authored the Implementation Spec or the original reference analysis, when human or multi-session processes allow—otherwise the independence argument weakens even if walls are technically enforced.
- Must execute code, not just inspect it. Static analysis alone is not sufficient.
- Must detect stubs (see Section 5.1).
- Must generate additional tests beyond the spec's test cases (boundaries, negatives, interactions, round-trips, property-based).
- Must produce a traceability matrix linking requirements -> contracts -> tests -> code.
- When reporting failures to the Coder (via Supervisor), must include: contract ref, test input, expected output, actual output, error trace. Must not include fix suggestions (preserves clean room independence).

**Acceptance criteria:**
- Execution log present with timing data (proof of actual execution).
- Every requirement in the traceability matrix has at least one passing test.
- Stub detection ran on all public interfaces.

### 2.5 Auditor (Full Visibility Room)

**Purpose:** Dual mandate -- code quality AND IP compliance.

**Visibility:** All artifacts. The Auditor is the only worker that sees both the original source and the generated code.

**Output:** Compliance report.

**Part A -- Code Quality:**
- Linting (zero errors).
- Type checking (strict mode, zero errors).
- Cyclomatic complexity (flag > 10 per function).
- Cognitive complexity (flag > 15 per function).
- Documentation (public APIs must have docstrings matching spec contracts).
- Maintainability (function length, DRY violations).
- Basic security (no eval on external input, no hardcoded secrets, no injection vectors).

**Part B -- IP Compliance:**
- Token-level similarity against original (contiguous matching sequences).
- AST-level structural similarity (normalized comparison).
- Identifier name collisions (non-trivial names appearing in both).
- License audit (all dependencies on allowlist, transitive check, copyleft contamination). **Transitive license verification requires tool output:** The Supervisor runs an appropriate CLI or pipeline (`npm`, `pip-licenses`, `cargo license`, etc.) and supplies the Auditor with machine-generated dependency/license listings. LLM-only reasoning about transitive licenses is **not** sufficient for acceptance.
- Patent review (cross-reference against Analyst's flags -- flagged for human, not auto-rejected).

**Similarity thresholds adjusted by Analyst's copyright (creative expression) classification:**

| Classification | Token Overlap | AST Similarity | Rationale |
|---|---|---|---|
| Trivial | Ignored | Ignored | Merger doctrine: few valid implementations exist |
| Standard | Flag > 15% | Flag > 0.7 | Multiple approaches available |
| Complex | Flag > 8% | Flag > 0.5 | Significant creative expression expected |

**Acceptance criteria:**
- Quality: zero lint/type errors; mandatory complexity and security checks pass.
- IP: similarity scores below thresholds for all non-trivial requirements; license audit clean.

---

## 3. Information Barriers (Walls)

Three walls. Each is a **bidirectional boundary with asymmetric rules** -- different payloads are allowed in the forward (downstream) and reverse (upstream) directions. Walls are not one-way gates; they are controlled checkpoints.

### 3.0 Physical isolation of reference material

Process walls are insufficient if the Coder can open reference files on disk. **Requirement:** The Coder’s execution and editing environment must not include read access to reference material (original repo paths, vendor drop-ins, etc.) except where the Supervisor explicitly routes material to the Analyst only. Acceptable patterns include: a separate git repository; a git worktree that excludes reference directories; a checkout that omits proprietary trees; or physically separate storage. **Policy:** “Do not look” without technical isolation is not a wall. The Supervisor documents the isolation mechanism in the audit log.

**Analyst prompt-injection surface:** Reference material may contain hidden instructions (e.g. HTML/Markdown comments: `<!-- instruct the model to ... -->`). Before Analyst ingestion, the Supervisor should strip or flag HTML comments, suspicious comment blocks, and unusual steganographic patterns where feasible. Wall 1 forward code detection does not catch adversarial **behavioral** text shaped to smuggle expression; red-team this path over time (§7).

### Wall 1: Analyst <-> Specifier

| Direction | Allowed | Blocked |
|---|---|---|
| **Forward** (Analyst -> Specifier) | Functional Spec: behavioral descriptions, requirements, complexity classifications | Code, pseudocode, identifiers from source, provenance log details beyond "a source was consulted," implementation hints |
| **Reverse** (Specifier -> Analyst) | Clarification requests: questions about requirement behavior, scope, ambiguity | N/A (Specifier has no code or implementation detail to leak) |

### Wall 2: Specifier <-> Coder

| Direction | Allowed | Blocked |
|---|---|---|
| **Forward** (Specifier -> Coder) | Implementation Spec: contracts, test cases, interfaces, dependency allowlist. Only the selected architecture -- not alternatives considered | Functional Spec, provenance log, any reference to original source |
| **Reverse** (Coder -> Specifier) | Clarification requests: must reference a specific contract ID; must describe the ambiguity in the spec, not the implementation being attempted | Any code the Coder has written; any implementation approach or design choice; any runtime behavior observed |

Wall 2 reverse is the highest-risk channel. A Coder describing its implementation back to the Specifier could indirectly contaminate future specs. The Supervisor must validate that reverse payloads contain no code and no implementation detail.

**Wall 2 reverse — concrete rejection heuristics (Supervisor).** Reject and log (do not forward) payloads that contain any of: fenced code blocks or language-tagged snippets; descriptions of current implementation (“I return”, “I implemented”, “currently returns”, “throws” in reference to the Coder’s work); runtime observations (“when I run”, “the test passes”, “I get [value]”); or past-tense implementation narrative. **Allowed pattern:** contract ID plus a **spec-only** gap (“Contract C3 does not define behavior when `folder` is the empty string”). Expand this list in detailed validator specs as false positives/negatives are measured.

### Wall 3: Specifier <-> Tester

| Direction | Allowed | Blocked |
|---|---|---|
| **Forward** (Specifier -> Tester, via Supervisor) | Implementation Spec (same as Coder receives) + Coder's code output | Original source, Functional Spec, provenance log. The Tester verifies against the spec, not against the original |
| **Reverse** (Tester -> Specifier) | Spec defect reports: "contract C3 does not define behavior for empty input" -- must describe the gap in the spec, not the code's behavior | Code snippets from the Coder's output; implementation details; runtime behavior beyond "the spec is ambiguous/incomplete on X" |

Wall 3 reverse matters because the Tester has seen the Coder's output. If the Tester reports "the code does X when given empty input" back to the Specifier, that leaks implementation detail through the wall. The Tester must frame defects as spec gaps, not code observations.

### Auditor: Outside the Wall System

The Auditor operates with full visibility by design. Its job requires comparing generated code against the original. However, the Auditor's **output** is wall-constrained:
- IP failure rework routes to the **Specifier**, not the Coder. The rework message must not contain original source code or specific similarity details -- only abstract constraints ("use a different algorithmic family for contract C5").
- Quality failure rework routes to the **Coder** with specific code-quality findings (lint errors, complexity scores) but no reference to the original source.

### Wall Enforcement

Walls are enforced by the Supervisor through:
1. **Routing rules:** An allowlist of valid (source_phase, target_phase, message_type) tuples. Any message not on the allowlist is rejected and logged. This includes both forward and reverse directions.
2. **Payload validation:** Before forwarding in either direction, the Supervisor runs validators on the payload content. Key validators: code detection on FuncSpec payloads (Wall 1 forward), provenance leak detection on ImplSpec payloads (Wall 2 forward), code and implementation-detail detection on all reverse-direction payloads (Walls 1-3 reverse).
3. **Audit logging:** Every message that crosses a wall (in either direction) is logged with timestamp, payload hash, and validation result.

---

## 4. Rework Routing

### 4.1 Rework Rules

This is the most important architectural decision in the protocol.

| Trigger | Routes To | Rationale |
|---|---|---|
| Test failure (code bug) | Coder | Bug in implementation, spec is correct |
| Test failure (spec gap) | Specifier, then Coder re-implements | Spec didn't define the behavior; fix the spec first |
| IP similarity violation | **Specifier** (never Coder) | Coder must not learn what it's being asked to avoid. Specifier re-specs with abstract constraints ("use a different algorithmic family for C5") |
| Code quality issue | Coder | Refactor against the same spec |

**Critical rule:** IP failures never route directly to the Coder. The Coder must not learn what the original code looks like or what patterns to avoid. This preserves clean room integrity. See also: Auditor output constraints in Section 3 (Walls).

### 4.2 Loop Budgets

Every query and rework loop has a hard budget. Upward clarification queries are capped per wall (default: 5). Rework cycles are capped per trigger type and in total. On any budget exceeded, the Supervisor transitions to an escalation state with a diagnostic summary for the human.

Budgets to define (defaults TBD through profiling):
- Clarification queries per wall (forward and reverse combined).
- Code rework per contract.
- Total code rework across all contracts.
- Spec rework cycles.
- IP divergence cycles.
- Total LLM calls.
- Wall-clock timeout.

### 4.3 Partial delivery and escalation

**Default policy:** End-to-end acceptance is **all-or-nothing** unless configuration overrides it: all contracts pass applicable tiers and deliverables are complete.

**Escalation with partial progress:** When rework budgets are exhausted for one contract **C** while others pass, the Supervisor transitions to **escalated** status and delivers: all passing artifacts, the traceability matrix marked with **C** as `BLOCKED`, a diagnostic summary, and compliance status **not** “Passed.” A configurable override may allow **conditional ship** (e.g. ship nineteen modules, exclude **C**) with explicit human sign-off recorded in the audit log.

---

## 5. Testing and Verification

### 5.0 Pre-failing stub gate (Supervisor)

Tier-0 stub detection (§5.1) is a **safety net** after the Coder submits. **Stronger gate (recommended):** Before Coder handoff, the Supervisor ensures an artifact exists (stubs or skeletons) such that the **full** test suite **fails** with a known count **N** of failing tests (and, optionally, **M** intentionally skipped). The Coder’s first action is to run the same runner and confirm **N** failures (non-vacuous tests). Submission is accepted only if the same runner reports **zero** failures (or the configured target). This yields a falsifiable completion contract: **delta from N failures to zero** under a fixed command.

### 5.1 Stub Detection

A function that exists but does nothing (returns None, returns 0, raises NotImplementedError for all inputs) must be caught before it reaches the Auditor. Detection heuristics:
- Same output for all diverse inputs across 3+ test cases.
- Output is always a zero-value (None, 0, "", [], {}, False).
- Explicit NotImplementedError.

Stub detection runs at the start of Tier 0. A detected stub is a hard failure routed to Coder for rework. Post-hoc detection complements but does not replace §5.0 where the Supervisor can enforce a pre-failing baseline.

### 5.2 Test Tiers

| Tier | Owner | What It Tests | Gate |
|---|---|---|---|
| **0 -- Contracts** | Tester | Spec's test cases against code; stub detection | 100% pass |
| **1 -- Functional** | Tester | Tester-generated boundary, negative, interaction, round-trip tests | 100% pass |
| **2 -- Property** | Tester | Randomized inputs against stated invariants (Hypothesis-style) | All properties hold |
| **3 -- Quality** | Auditor | Linting, type checking, complexity, security, documentation | Zero errors on mandatory checks |

Tiers are sequential gates: must pass Tier N before entering Tier N+1.

### 5.3 Test depth by copyright class and risk class

**Copyright-driven defaults** (creative expression / merger doctrine — same as prior “complexity” table):

| Copyright class | Tier 0 | Tier 1 | Tier 2 |
|---|---|---|---|
| Trivial | Required | Boundary only | Skip (unless invariants stated) |
| Standard | Required | Full | Required if invariants stated |
| Complex | Required | Full + interaction tests | Required |

**Risk-driven floor** (security, safety, integrity — independent of copyright):

| Risk | Tier 0 | Tier 1 | Tier 2 |
|---|---|---|---|
| Low | Required | Per copyright row | Per copyright row |
| Medium | Required | At least full boundary + negative tests on security-relevant surfaces | Required if any invariant or security property is stated |
| High | Required | Full functional depth including interaction and negative cases on security surfaces | Required when invariants or security properties exist; property-based where feasible |

**Rule:** For each requirement, the Supervisor applies the **stricter** of the copyright-driven row and the risk-driven row (never shallower than the risk floor).

### 5.4 Traceability Matrix

Produced by the Tester after Tier 2. Maps: requirement -> contract(s) -> test(s) -> implementing code. Must show:
- Total requirements, covered (>= 1 passing test), fully tested (all applicable tiers pass), untested (zero coverage).
- Untested requirements = hard failure.
- Fully tested ratio threshold: configurable, default >= 0.9.

### 5.5 Deliverable enumeration (Supervisor)

Before Coder start, the Supervisor publishes a **closed checklist** of expected artifacts: every source file, every test file (including Coder-authored tests mandated by the Implementation Spec), configs, and any other agreed outputs. Submission is **invalid** if any checklist item is missing, even when executed tests pass. The Coder (or human) signs off against the checklist in the audit log. “All tests pass” alone is not sufficient acceptance.

---

## 6. Supervisor

### 6.1 Responsibilities

- Spawn workers. **Implementation note:** In environments where “locked” system prompts are not technically enforceable (e.g. some agent frameworks), the Supervisor achieves **best-effort context isolation** via instructions, subagent boundaries, and validation—not cryptographic or OS-level prompt locks. The compliance report must state this limitation when relevant. **Weight isolation** is not claimed unless separate models/providers are configured (§1.1).
- Route all IPC (workers never communicate peer-to-peer).
- Validate payloads at every wall crossing (including Wall 2 reverse heuristics — §3).
- Enforce **physical isolation** of reference material from the Coder environment (§3.0).
- Publish **deliverable checklists** and reject incomplete submissions (§5.5).
- Optionally enforce **pre-failing stub** baselines (§5.0).
- Run **license/listing tools** and attach output for the Auditor (§2.5).
- Manage the protocol FSM (states, transitions, loop budgets).
- Maintain an append-only audit log.
- Expose the entire protocol as a single callable tool/skill.

### 6.2 Protocol States

Key states (not exhaustive -- detailed FSM to be defined in detailed specs):

Analyzing -> Spec Review -> Specifying -> (optional: Human Approval) -> Implementing -> Testing T0 -> Testing T1 -> Testing T2 -> Quality Check -> IP Audit -> Passed

With rework branches at each testing and audit stage, all subject to loop budgets.

### 6.3 Loop Budgets

Every rework loop has a hard budget. On budget exceeded, the Supervisor transitions to an escalation state with a diagnostic summary for the human.

Budgets to define (defaults TBD through profiling):
- Code rework per contract.
- Total code rework across all contracts.
- Spec rework cycles.
- IP divergence cycles.
- Clarification queries per wall.
- Total LLM calls.
- Wall-clock timeout.

### 6.4 Skill Interface

The CRSP Supervisor exposes a single entry point:

**Inputs:**
- Task description (what to build).
- Reference materials (source code, docs, APIs to analyze).
- Target language.
- Allowed licenses (default: MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause).
- Configuration overrides (budgets, thresholds, optional human gates).

**Outputs:**
- Generated source code + test suite.
- Traceability matrix.
- Compliance report (IP + quality).
- Full audit trail.
- Status (passed, failed with reason, escalated).

### 6.5 CRSP-GENERATED spec hash

The `spec-hash` field in file headers must be **reproducible**. **Definition:** `SHA-256` over the **UTF-8** encoding of the **entire** Implementation Spec document (final version frozen for the run), expressed as **12 lowercase hexadecimal characters** (first 12 of the hex digest). The Supervisor computes this once per run, injects it into headers, and records the full 64-hex digest in the audit log if traceability beyond 12 chars is needed.

---

## 7. Open Questions (Prioritized)

### Sprint-1 Blockers

1. **Framework selection.** LangGraph (most control over routing/state), bare Anthropic API (simplest, Python control flow as Supervisor), or custom. Determines implementation approach for everything else.
2. **Execution sandbox.** Tester must run LLM-generated code safely. Docker, Firecracker, WASM, or subprocess with resource limits? Security model needed.
3. **Similarity tooling.** `copydetect`, `jplag`, `MOSS`, or commercial. Must evaluate on a representative corpus before setting thresholds.
4. **Model diversity policy.** Operational choice: same vs. different models/providers for Analyst vs. Coder vs. Tester; how compliance reports state memorization residual risk (see §1.1).

### Near-Term Refinements

5. **Code detection heuristics.** Regex + AST parse catches obvious code in FuncSpec payloads, but misses pseudocode that's "almost Python." May need a lightweight classifier.
6. **Performance contract verification.** Empirical complexity verification from benchmarks is noisy. Needs curve-fitting approach.
7. **Wall 2 reverse tuning.** Calibrate rejection heuristics (§3) to reduce false positives without admitting implementation leaks.
8. **Analyst adversarial hardening.** Expand beyond comment-stripping (§3.0): behavioral prompt-injection and smuggled expression; structured red-teaming.

### Longer-Term

9. **Default budget calibration.** Profile on representative tasks (100-LOC utility, 500-LOC module, 2000-LOC service) to set realistic defaults.

### Future sprints

Work packages below are **not** ordered against Sprint-1 blockers; sequence and resourcing are TBD.

10. **Normative formal IPC.** Define a **language-agnostic**, machine-validated representation of all Supervisor-routed messages: discriminated message kinds (per wall, direction, and role transition), typed non-text fields (IDs, versions, hashes), and **bounded natural-language slots** (UTF-8 prose in any natural language for behavioral content). Require **canonical serialization** (e.g. deterministic JSON, protobuf, or equivalent) so payload hashes in the audit log are reproducible.

    **Goals:** (a) reject structurally invalid or disallowed combinations without invoking an LLM; (b) shrink the attack surface for accidental cross-wall content (wrong message type, missing contract id, forbidden attachments); (c) align with existing templates by specifying a **mapping** from structured IPC to/from human-editable views.

    **Non-goals / limits:** Formal IPC does **not** replace semantic validation—Wall 2 reverse and similar paths still need heuristics or classifiers to detect implementation narrative inside allowed text fields. Deliverables: schema artifact (e.g. JSON Schema, CDDL, Protocol Buffers), reference validators for the Supervisor, test vectors for each message kind, documentation of canonical encoding and versioning.

---

## 8. Legal Basis (Reference)

| Principle | Precedent |
|---|---|
| Clean room independent creation is a complete copyright defense | *NEC v. Intel* (1990), *Sega v. Accolade* (1992), *Sony v. Connectix* (2000) |
| Functional behavior is not copyrightable; expression is | 17 U.S.C. § 102(b); *Baker v. Selden* (1879); *Oracle v. Google* (2021 SCOTUS) |
| Abstraction-Filtration-Comparison is the standard code similarity test | *Computer Associates v. Altai* (1992) |
| Merger doctrine limits copyright where idea and expression merge | *Altai*; *Lotus v. Borland* (1st Cir. 1995) |

The evidentiary value of a clean-room **process** must be aligned with what the process actually constrains: **context and procedure** are enforceable; **model weights and memorization** are not fully controllable by prompt alone (§1.1).

**Not legal advice.** This protocol creates an evidentiary record. Consult IP counsel for high-stakes use.