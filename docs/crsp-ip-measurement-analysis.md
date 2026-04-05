# CRSP: Measurement of Copied Content — Analysis

**Date:** 2026-04-04  
**Scope:** How IP similarity is structured in the current spec, whether rename-only evasion is effective, and how that relates to copyright doctrine (high level, not legal advice).

**Normative source:** [`specs.md`](specs.md) (Auditor, threat model, legal reference).

---

## 1. How “copied content” is structured in CRSP today

At the **architecture** level, the threat model separates **verbatim reproduction** from **substantial similarity** and assigns post-hoc measurement to the **Auditor**. The spec states that similarity scanning is **detection after the fact**, not prevention of memorized or copied output (`specs.md` section 1.1).

The **Auditor Part B — IP compliance** defines three quantitative signals plus **classification-dependent thresholds**:

| Signal | Role in spec |
| --- | --- |
| **Token-level similarity** | Contiguous matching sequences against the original |
| **AST-level structural similarity** | Normalized comparison (definition TBD at implementation time) |
| **Identifier name collisions** | Non-trivial names appearing in both works |

Thresholds (token overlap and AST similarity) depend on the Analyst’s **copyright / creative expression** class (Trivial, Standard, Complex). **Trivial** requirements ignore token and AST gates (merger doctrine: few valid implementations).

**Gaps:** The spec does not yet normatively define tokenization, AST normalization (e.g. α-renaming, literal handling, control-flow granularity), window sizes for contiguous tokens, or how multiple scores are composed. Tooling and calibration are open work (`specs.md` section 7).

---

## 2. Would renaming all variables and functions “fool” the measurements?

**Summary:** Partially — it depends which metric is relied on and how it is implemented.

| Signal | Effect of rename-only evasion |
| --- | --- |
| **Identifier collisions** | Largely **neutralizes** this check if every non-trivial shared identifier is renamed. As a **primary** defense against copying, this metric is **easy to game**. |
| **Token-level contiguous sequences** | **May still trigger** on copied **comments**, **string literals**, **boilerplate order**, or **large pasted blocks** with minor edits — or **miss** short or generic sequences. Highly **tool-dependent**. |
| **AST similarity (properly normalized)** | If normalization **abstracts identifiers** (as it should for structural comparison), renaming **does not** by itself fix **isomorphic** or near-isomorphic structure. Weak tools, small programs, or highly idiomatic code increase **false negatives**. |

**Conclusion:** Rename-only is a weak attack against a **serious structural** pipeline; it is a **strong** attack against **identifier-overlap-as-primary-signal** designs. CRSP should treat identifier collision as a **cheap tripwire**, not the backbone of IP compliance.

---

## 3. Would rename-only “fly” in a copyright case?

**Not legal advice.**

U.S. software copyright analysis commonly uses **abstraction–filtration–comparison** (see `specs.md` section 8, *Computer Associates v. Altai*). The court compares **protected expression** after filtering ideas, methods, scènes à faire, merger-limited expression, etc. **Substantial similarity** is **not** equivalent to “matching variable names.” **Structure, sequence, and organization** of what remains can still support a finding of infringement **even after renaming**.

**Clean room** and **independent creation** are **factual and process** defenses. Automated similarity scores are **evidentiary aids**, not a substitute for the legal test. Low scores on tooling do not guarantee non-infringement; high scores on weak metrics do not guarantee infringement without filtration and context.

---

## 4. Implications for future spec and tooling work

1. **Normatively specify** AST normalization and token rules so “rename everything” is not the critical failure mode of CRSP’s own checks.
2. **Weight** identifier overlap appropriately — useful signal, **not** sufficient alone.
3. **Align** automated comparison with the **spirit** of filtration (e.g. separate handling of literals, comments, and generic idioms) even if full legal filtration remains human-driven for Complex cases.
4. **Document** in compliance reports that metrics are **heuristic** and **calibrated**, not dispositive for legal outcomes.

---

## 5. Related documents

- [`specs.md`](specs.md) — Auditor Part B, thresholds, section 8 legal basis  
- [`crsp-field-notes.md`](crsp-field-notes.md) — Operational lessons from a hands-on run  
