## Adversarial Review — `plan2.md` v0.5

**Context reviewed:** `plan2.md` against `background.md`, `TOE_gpt.md`, `AGENTS.md`, `README.md`.

**Verdict:**  
**NOT READY FOR IMPLEMENTATION PLAN.**  
The three-project separation is a major architectural improvement. It correctly resolves several prior review findings by moving responsibilities to the right project instead of adding new layers. But v0.5 remains a design-review draft: all blockers `B-001`–`B-020` are still open, and its own §36 checklist is unsatisfied. By its own G2 and final status, implementation planning is not authorized.

---

## What v0.5 gets right

- Separates **Project A (TOE AI research)**, **Project B (PhysMath)**, **Project C (`phys-*`)**, plus frozen **Phys** and **Numerical Layer**.
- Correctly rejects a permanent “Candidate Physical Object” type in PhysMath.
- Correctly rejects Dancer/Dance/Stage as PhysMath ontology.
- Correctly keeps Phys as the single trust/metadata authority.
- Correctly keeps numerical computation outside PhysMath.
- Correctly treats the TOE as a research program, not a monolithic Go program.
- Correctly reframes Qwen’s “domain tagging” as generic mathematical typing.
- Correctly refuses to introduce a new “Interpretation Layer.”

These are genuine improvements.

---

## Remaining architectural blockers

All blockers below are still `OPEN`. They are not minor; each prevents a coding agent from implementing without inventing architecture.

### B-001 — Three-project ownership and dependency graph

Still open. The exact repositories, modules, packages, and import rules for Project A, PhysMath, `phys-*`, Phys, and Numerical are not pinned. The diagrams in §0 and §6.1 are conceptual, not mechanically enforceable. Without this, coding agents will invent cross-project dependencies.

### B-002 — PhysMath core versus Phys interoperability

Still open. The document retains `PhysMath -> Phys` as a locked project-level dependency, but also says pure mathematical packages must not import physical-domain packages. That is reconcilable only with an explicit interop package boundary. No such package graph or import test is defined.

### B-003 — PhysMath IR/sort system

Still open. Core sorts for expressions, functions, functionals, operators, tensors, maps, relations, and obligations are not pinned. Extension protocol is not defined. Without this, no mathematical object can be implemented deterministically.

### B-004 — Canonicalization, equality, hashing

Still open. No canonical form, no deterministic serialization, no distinction between structural identity and mathematical equivalence is pinned. This is foundational for replay and handoff.

### B-005 — Mathematical conditions and binding/scope

Still open. How mathematical conditions are represented without becoming Phys metadata is unresolved. Scope, binding, and precondition semantics are not pinned.

### B-006 — Mathematical versus physical dimensions

Still open. PhysMath must manipulate symbols like `c`, `ℏ`, `G`, but must not own physical units. The mathematical dimension/index model versus Phys physical-unit semantics is not separated.

### B-007 — Higher-order mathematical operations

Still open. The TOE requires functions, functionals, operators, tensors, maps. The type system must prevent collapse into scalar expression trees. Operation contracts are not pinned.

### B-008 — Formal limits/asymptotics

Still open. The capability is required for RG/coarse-graining work, but exact representation of limits, truncations, and asymptotic order is not pinned.

### B-009 — Mathematical trace versus Phys ledger

Still open. PhysMath trace and Phys trusted ledger are declared different replay domains, but the handoff reference between them is not defined. A Phys ledger must not claim to replay arbitrary PhysMath traces.

### B-010 — Cross-framework correspondence artifacts

Still open. Project A owns research-level correspondences, but no artifact schema exists. Without it, correspondences can silently become identities.

### B-011 — External Phys-library object contract

Still open. This is critical. Frozen Phys has a constrained object/quantity model. The TOE requires physical objects like `BohmianConfiguration`, `CausalSet`, `PhysicalSpacetime`. How `phys-*` defines these objects outside the frozen Phys base is not specified. If Phys cannot represent them, they remain `UNREPRESENTABLE`; that must be an explicit, acceptable outcome, not an accident.

### B-012 — Phys metadata attachment to external objects

Still open. How assumptions, conventions, provenance, and trust attach to external physical objects without duplicating metadata systems is unresolved. This is the metadata monopoly enforcement point.

### B-013 — Formalization bridge

Still open. The exact API/schema crossing from Project A + PhysMath + `phys-*` into Phys is not defined. Without it, handoff is prose.

### B-014 — Handoff loss accounting

Still open. `FULLY/PARTIALLY/UNREPRESENTABLE` classification is required, but the loss report schema is not pinned. A partial export must not silently become trusted.

### B-015 — `t`-flow / `k`-flow typing

Still open. The plan correctly says these must be distinct, but no formal parameter identity model or negative test is defined.

### B-016 — TOE joint-dynamics acceptance contract

Still open. The TOE requires a candidate for  
\[
\mathcal D[\Psi,Q,\mathcal C]=0
\]  
but Project A acceptance artifacts and the mathematical/physical package interfaces for formulating that candidate are not pinned.

### B-017 — AI-agent machine-readable research protocol

Still open. Versioned schemas for hypothesis, trace, and handoff are not defined. Without them, AI agents will revert to prose as semantic state.

### B-018 — Numerical handoff boundary

Still open. The exact artifacts the Numerical Layer may consume are not pinned. Raw PhysMath research state must not become authoritative physical meaning.

### B-019 — Phys Growth Gate and extension boundary

Still open. The routing rule for when a missing capability stays external versus when it justifies Phys kernel growth is described but not mechanized.

### B-020 — Resource and adversarial testing

Still open. Deterministic resource limits, failure classes, and adversarial tests for malformed research inputs are not pinned.

---

## New or sharpened blockers introduced by the three-project split

### 1. Project A interface contract

Project A is now a separate project. How does it invoke PhysMath? Does it import PhysMath as a Go library, or exchange machine-readable schemas? Does it run as a Go program, a service, or an agent orchestration layer? Its API to PhysMath, Phys, and `phys-*` is not defined. This should be folded into B-001 and B-017.

### 2. Minimal Phys-representable artifact for TOE

The TOE is the first major workload. The plan includes Phys formalization in the lifecycle. But frozen Phys may not represent central TOE objects. What is the minimal Phys-representable artifact the TOE must produce to pass acceptance? If the answer is “none,” then Phys is not part of the TOE acceptance test, which contradicts §27.4 and §30. This needs an explicit decision.

### 3. External `phys-*` composability

The plan says `phys-*` libraries may depend on Phys and PhysMath. But how do they compose with each other? The TOE requires Dancer ↔ Stage, Dance ↔ Stage, Dancer ↔ Dance. These interfaces cross `phys-qm`, `phys-bohm`, `phys-cst`, `phys-cdt`. A cross-extension interface schema is missing.

### 4. Project A research state vs PhysMath trace

Project A owns hypothesis chains. PhysMath owns mathematical traces. How are they linked? How does a hypothesis reference a mathematical trace without importing PhysMath trust semantics? This is not fully covered by B-009 or B-017.

---

## Required next actions

1. **Do not authorize implementation planning.** The document itself says this.
2. **Close B-001 through B-020** in the decision register. Each must have a stable decision entry with rationale, affected packages, and tests.
3. **Add the new/sharpened blockers** listed above to the register.
4. **Populate the decision register** required by the earlier plan’s §34. v0.5 does not include it.
5. **Re-run adversarial review** after blockers are closed or explicitly deferred with containment.

---

## Final status

```text
REVIEW: plan2.md v0.5

Three-project separation: PASS
Phys trust metadata monopoly: PASS as intent
PhysMath mathematics-only boundary: PASS as intent
External phys-* boundary: PASS as intent
Numerical Layer separation: PASS as intent

Mechanically enforceable dependency graph: OPEN (B-001, B-002)
PhysMath IR/sorts/canonicalization: OPEN (B-003, B-004)
Condition/scope/dimension models: OPEN (B-005, B-006)
Higher-order math/asymptotics: OPEN (B-007, B-008)
Trace/replay/handoff: OPEN (B-009, B-013, B-014)
Cross-framework/TOE acceptance: OPEN (B-010, B-016)
External object/metadata contract: OPEN (B-011, B-012)
t/k flow typing: OPEN (B-015)
AI-agent protocol: OPEN (B-017)
Numerical handoff: OPEN (B-018)
Growth Gate routing: OPEN (B-019)
Resource/adversarial testing: OPEN (B-020)

FINAL STATUS: NOT READY FOR IMPLEMENTATION PLAN
```

The architecture is improving, but the boundary is still partly prose. The three-project split is the right direction. The remaining work is to pin the interfaces, schemas, and invariants that make the separation mechanically real.