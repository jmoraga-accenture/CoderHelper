# CRSP artifact templates

Use these structures when producing CRSP outputs. Adjust headings to match project conventions.

---

## Functional Spec (Analyst → Specifier)

```markdown
# Functional Spec

## Task summary
[Behavioral goal in plain language — no implementation]

## Requirements
### R1 — [short name]
- **Description:** …
- **Inputs:** …
- **Outputs:** …
- **Constraints:** … (prose behavior — do not copy distinctive regexes, magic constants, or message text from reference)
- **Copyright class:** Trivial | Standard | Complex
- **Risk:** Low | Medium | High
- **Patent note:** None | [flag + brief reason]

### R2 — …

## Provenance log
| Source | Type (doc, code, API, etc.) | Notes |
| --- | --- | --- |

## Out of scope
…
```

---

## Implementation Spec (Specifier → Coder / Tester)

```markdown
# Implementation Spec

## Conventions
- **Target language / version:** …
- **Naming prefix / package:** … (independent of any reference codebase)

## Architecture
### Alternatives considered (required for Standard/Complex *copyright* contracts)
1. **Option A:** …
2. **Option B:** …
**Selected:** … **Rationale:** …

## Dependency allowlist
| Package / module | Version pin | License (SPDX) | Notes |
| --- | --- | --- | --- |

## Contracts

### C1 — [interface or module name]
- **Preconditions:** …
- **Postconditions:** …
- **Invariants:** …
- **Error contract:** …
- **Performance (if any):** …
- **Test cases (≥3):**
  1. Input: … → Expected: …
  2. Input: … → Expected: …
  3. Input: … → Expected: …
- **Test strategy:** Tiers 0–3; property generators: … (respect risk floor from Functional Spec)
- **Trace to functional reqs:** R…, R…

### C2 — …
```

---

## CRSP-GENERATED file header

Place at top of each generated source file (adjust comment syntax for language).

**`spec-hash`:** SHA-256 hash of the **entire** Implementation Spec document in UTF-8, then take the **first 12 hexadecimal characters** (lowercase). Supervisor records the full 64-char digest in the audit log if needed (`docs/specs.md` section 6.5).

```
CRSP-GENERATED
protocol-version: 0.4-DRAFT
spec-hash: <12 hex chars, SHA-256 full Impl Spec UTF-8>
generated-at: <ISO-8601 UTC>
```

---

## Deliverable checklist (Supervisor → Coder)

```markdown
# Deliverable checklist (closed set)

Submission is invalid if any row is missing at gate time.

| # | Path / artifact | Type (source, test, config, …) | Required by |
| --- | --- | --- | --- |
| 1 | … | … | Impl Spec C… |

**Coder sign-off:** [ ] All rows present before final submission.
```

---

## Traceability matrix (Tester)

```markdown
# Traceability matrix

| Req | Contract | Test IDs | Code location | Tier 0 | Tier 1 | Tier 2 | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |

## Summary
- Total requirements: N
- Covered (≥1 passing test): …
- Fully tested (all applicable tiers): …
- Untested: … (must be zero to pass)
- Fully tested ratio: … (target ≥ 0.9 unless configured otherwise)
- BLOCKED (escalation): …
```

---

## Compliance report (Auditor)

```markdown
# CRSP compliance report

## Status
Passed | Failed | Escalated | Conditional (human override per section 4.3)

## Implementation limitations
- Reference isolation mechanism: …
- Same model / provider for multiple roles: Yes/No — narrative impact (see specs section 1.1)
- Prompt lock: best-effort context isolation only / other: …

## Part A — Quality
- Lint: …
- Types (strict): …
- Complexity flags: …
- Security notes: …
- Documentation vs contracts: …

## Part B — IP
- Token similarity by requirement/copyright class: …
- AST similarity: …
- Identifier collisions (non-trivial): …
- License audit (tool output attached): …
- Patent flags: …

## Rework directives (wall-safe)
- To Specifier (abstract only): …
- To Coder (quality only): …
```

---

## Clarification request (Coder → Specifier, Wall 2 reverse)

```markdown
# Spec clarification

- **Contract ID:** C#
- **Ambiguity:** [What the Implementation Spec does not define — spec wording only, no runtime or code]
- **Blocking:** Yes | No
```

---

## Spec gap report (Tester → Specifier, Wall 3 reverse)

```markdown
# Spec gap

- **Contract ID:** C#
- **Gap:** [What behavior is undefined or contradictory in the Implementation Spec]
- **Suggested spec addition (behavioral only):** …
```

(Do not include Coder code snippets or “the implementation returns X” observations.)
