The architecture revision in `plan2.md` (v0.5) successfully resolves the primary structural deadlocks of earlier drafts by establishing a strict **Three-Project Architecture**. Rather than forcing a single Go program or adding an unneeded "Interpretation Layer" into `PhysMath`, responsibility is cleanly partitioned across distinct repositories and authorities.

```text
                        PROJECT A: TOE-AGENT
                      (AI Reasoning & Hypotheses)
                         /        |        \
                        /         |         \
                       v          v          v
                 PROJECT B:    PHYS BASE   PROJECT C:
                  PhysMath    (Frozen)     phys-* suite
                     |            ^            |
                     +------------+------------+
                                  |
                        Formalized Artifacts
                                  |
                                  v
                           NUMERICAL LAYER

```

---

### Key Architectural Corrections in v0.5

1. **Resolution of the AI Exploration Catch-22**
* **Previous Defect:** `PhysMath` could not hold speculative physical ideas without violating its math-pureness rule, while `phys-*` libraries required full package releases.


* **v0.5 Correction:** Project A (TOE-Agent) exclusively owns hypothesis chains, research obligations, attack strategies, and correspondence proposals. `PhysMath` remains strictly math-pure, and `phys-*` libraries supply domain-specific physical contracts. Speculative physical coupling occurs in Project A without modifying `PhysMath` or prematurely releasing Go modules.




2. **Rejection of the "Candidate Physical Object" / "Interpretation Layer"**
* **v0.5 Correction:** Correctly rejects inserting physical interpretation layers into `PhysMath`. Physical semantics belong exclusively to external `phys-*` packages, canonical trust metadata belongs to `Phys`, and research-level conjectures belong to Project A.




3. **Formalization Handoff & Loss Accounting**
* **Previous Defect:** Complex `PhysMath` constructs exported to `Phys` were marked unrepresentable, creating a dead end for numerical simulation.


* **v0.5 Correction:** Blockers B-013 and B-014 define a machine-readable `HandoffPackage` with mandatory export classification (`FULLY_REPRESENTABLE`, `PARTIALLY_REPRESENTABLE`, `UNREPRESENTABLE`) and explicit loss reporting. The Numerical Layer consumes formalized `Phys`/`phys-*` artifacts and cannot silently fill semantic gaps.




4. **Dynamic AI AST Engine vs. Static Go Types**
* **Previous Defect:** Hardcoded Go types prevented AI agents from dynamically generating new mathematical sorts or operators.


* **v0.5 Correction:** `PhysMath` is structured as a dynamic, versioned AST interpreter (B-003) operating over explicit mathematical sorts (`Expression`, `Function`, `Operator`, `Tensor`, etc.). AI agents interface via machine-readable JSON/Protobuf protocols (B-017).




5. **Multi-Flow Dynamic Separation ($t$ vs. $k$) & Joint Dynamics**
* **Previous Defect:** Conflation of physical time evolution ($\partial_t$) and renormalization scale flow ($\partial_k$).


* **v0.5 Correction:** §16 and B-015 enforce parameter identity and role separation to prevent conflating $\partial_t$ and $\partial_k$. The target TOE joint-dynamics equation $\mathcal{D}[\Psi, Q, \mathcal{C}] = 0$ is classified as a Project A research obligation rather than a compiler primitive (B-016).




6. **Reframing Domain Tagging**
* **v0.5 Correction:** Reframes domain tags into generic mathematical types/sorts (`HilbertSpace`, `Manifold`, `Poset`) within `PhysMath`. Dancer/Dance/Stage concepts remain research workload labels in Project A rather than fixed `PhysMath` ontology.





---

### Non-Overlap & Authority Matrix

| Architectural Authority | Primary Responsibilities | Strict Non-Goals & Prohibitions |
| --- | --- | --- |
| **Project A (TOE-Agent)** | Hypothesis workflows, attack strategies, correspondence proposals, handoff manifests.

 | Must not implement a second math engine, physics ontology, or trust plane.

 |
| **Project B (PhysMath)** | Reusable exact mathematical structures, symbolic transformations, AST sorts, and mathematical traces.

 | Must not contain physical ontology, physical regimes, physical provenance, or floating-point execution.

 |
| **Project C (`phys-*` Suite)** | Framework-specific physical objects (`phys-qm`, `phys-bohm`, `phys-gr`, `phys-cst`, `phys-cdt`) and domain contracts.

 | Must not duplicate `PhysMath` engine or implement package-local trust/provenance metadata.

 |
| **Phys Substrate (Frozen Base)** | Canonical physical metadata, assumptions, conventions, provenance, corpus status, and `Session` derivation authority.

 | Must not expand kernel code merely to accommodate complex `PhysMath` structures.

 |
| **Numerical Layer** | Floating-point evaluation, PDE/ODE integration, Monte Carlo, and empirical comparisons.

 | Must not interpret raw research state or override semantic/formalization gaps.

 |

---

### Critical Path: Closing the 20 Decision Blockers (Gate G2)

Before proceeding to Gate G3 (Architecture) and Gate G5 (Implementation Planning), all 20 decision-level blockers must be formally closed:

#### 1. Repository & Package Boundaries

* **B-001 (Package Graph):** Establish mechanically verifiable import paths proving `PhysMath` core cannot import physical packages.


* **B-002 (Interop Isolation):** Confine `PhysMath` $\rightarrow$ `Phys` imports strictly to designated interoperability/adaptation packages.


* **B-011 & B-012 (External Object & Metadata Contracts):** Define standard Go struct shapes for non-quantity physical objects in `phys-*` and enforce canonical attachment to `Phys` metadata without duplicate trust systems.


* **B-019 (Growth Gate Routing):** Mandate external routing for new domain features before requesting `Phys` kernel additions.



#### 2. Core Mathematical IR & Exact Symbolic Engine

* **B-003 & B-004 (IR Sorts & Canonicalization):** Pin closed core IR sorts (`Expression`, `Function`, `Functional`, `Operator`, `Tensor`, etc.), deterministic serialization, and distinction between structural identity and mathematical equivalence.


* **B-005 & B-006 (Conditions & Mathematical Dimensions):** Formulate mathematical condition scoping ($x > 0$) separate from physical assumptions, and decouple tensor/manifold dimensions from physical SI units.


* **B-007 & B-008 (Higher-Order Ops & Asymptotics):** Define exact bounded transformations for variational operators ($\frac{\delta}{\delta \mathcal{C}}$) and formal asymptotic expansions ($F(\varepsilon) = a_0 + a_1 \varepsilon + \mathcal{O}(\varepsilon^2)$).



#### 3. Formalization Bridge & Handoff Protocols

* **B-009 (Replay Domains):** Define boundaries separating non-projectable `PhysMath` mathematical execution traces from trusted `Phys` derivation ledgers.


* **B-010 (Correspondence Artifacts):** Pin schema for Project A conjectures connecting distinct frameworks (e.g., CST $\leftrightarrow$ CDT).


* **B-013 & B-014 (Handoff Schema & Loss Accounting):** Define JSON schema for `HandoffPackage` and enforce `FULLY`/`PARTIALLY`/`UNREPRESENTABLE` classification.


* **B-018 (Numerical Handoff Surface):** Restrict Numerical Layer consumption strictly to formalized `Phys`/`phys-*` artifacts.



#### 4. TOE Workload & AI Agent Protocol

* **B-015 ($t$ vs. $k$ Flow Typing):** Implement explicit parameter role types preventing conflation of $\partial_t$ and $\partial_k$.


* **B-016 (TOE Acceptance Contract):** Specify Project A interfaces to express candidate joint dynamics $\mathcal{D}[\Psi, Q, \mathcal{C}] = 0$ as a research obligation.


* **B-017 (AI Research Protocol):** Define versioned schemas for AI hypothesis creation, attack logging, and handoff execution.


* **B-020 (Adversarial & Resource Testing):** Define depth, memory, and recursion limits to prevent pathological mathematical inputs from exhausting resources.



---

### Status Verification

Document version 0.5 (`plan2.md`) is **DESIGN-REVIEW / NOT YET IMPLEMENTATION-AUTHORIZED**. Implementation planning is authorized only when all 21 items on the authorization checklist (§36) are verified and signed off at Gate G2.