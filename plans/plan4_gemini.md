The rewrite in `plan4.md` (v1.1) successfully resolves the structural deadlocks of the previous iteration. By replacing a vertical package-dependency stack with an orthogonal artifact-reference graph, the architecture moves from forcing package boundaries to carry physical meaning to letting each substrate own its domain with zero static coupling.

The core posture—$\text{PhysMath}\;\bot\;\text{Phys}$ with $\text{phys-*}\rightarrow(\text{PhysMath},\text{Phys})$—unblocks implementation across all repositories.

---

## Architectural Analysis of the Corrections

### 1. Orthogonality of Pure Math and Canonical Trust ($\text{PhysMath}\;\bot\;\text{Phys}$)

Decoupling `PhysMath` and `Phys` at the direct import level eliminates the circular dependency traps that plagued earlier drafts.

* **`PhysMath` Purity:** By prohibiting imports of `Phys` or `phys-*`, `PhysMath` operates exclusively as a pure mathematical engine handling exact AST transformations, canonicalization, De Bruijn variable binding, and mathematical condition tracking.


* **`Phys` Governance:** `Phys` retains a bounded mandate: managing canonical physical formalization, trust assertions, conventions, and ledger provenance without needing to act as a general Computer Algebra System (CAS).



### 2. The Bridge Function of `phys-*` Domain Libraries

Rather than forcing `PhysMath` to understand domain-specific physics or forcing `Phys` to store dynamic theoretical operators, domain libraries (`phys-qm`, `phys-bohm`, `phys-gr`, etc.) explicitly bridge the two:

* `phys-*` packages consume pure mathematical ASTs from `PhysMath` and assign framework-specific physical semantics and interpretation scopes.


* They bind physical metadata using `Phys`'s trust and convention models without attempting to instantiate competing, local metadata engines.


* They compile symbolic equations into lowered, typed computational contracts consumed downstream by the Numerical Layer.



### 3. Containment of Research Exploration in `TOE-Agent`

The new architecture cleans up the "orphaned metadata" dilemma during active research. Candidate parameters, un-curated boundary conditions, provisional gauge fixings, and cross-framework correspondence proposals live strictly inside `TOE-Agent` as `ResearchHypothesis` and `CorrespondenceProposal` artifacts. Un-trusted agent exploration cannot pollute `Phys`'s formal substrate or corrupt `PhysMath`'s algebraic invariants.

### 4. Protocol-Based Formalization vs. One-Shot Dump

Replacing the binary `HandoffPackage` dump with the stateful `FormalizationProposal` protocol reflects how theoretical formalization actually operates:

* The review loop allows explicit formalization outcomes (`ACCEPTED`, `REJECTED`, `REQUEST_REVISION`, `PARTIALLY_ACCEPTED`, `UNREPRESENTABLE`).


* Loss accounting (`FULLY_REPRESENTABLE`, `PARTIALLY_REPRESENTABLE`, `UNREPRESENTABLE`) guarantees no silent loss of mathematical structure when mapping into the frozen `Phys` formal substrate.



---

## Structural Comparison: Version 1.0 vs. Version 1.1

| Architectural Dimension | `plan3.md` (v1.0) | `plan4.md` (v1.1) |
| --- | --- | --- |
| **Dependency Topology** | Vertical stack with dependency exceptions | Strictly acyclic reference graph: $\text{PhysMath}\;\bot\;\text{Phys}$<br> |
| **Extension Sorts** | Dynamic extension sorts in `PhysMath` (causing AST deadlocks) | Mathematical ASTs stay pure; physical traits/contracts live in `phys-*`<br> |
| **Candidate Research Metadata** | Unowned / orphaned during hypothesis exploration | Owned by `TOE-Agent` inside `ResearchHypothesis` records

 |
| **Formalization Mechanism** | One-shot binary `HandoffPackage` transfer | Stateful `FormalizationProposal` protocol with loss tracking

 |
| **Numerical Boundary** | Unspecified lowering path from ASTs to numerical solvers | `phys-*` explicitly emits lowered `ComputationalContract`s to Numerical

 |
| **Replay & Trace Authority** | Monolithic or split replay domains lacking integration | Authority-local traces (`MathTrace`, `Phys` Ledger) linked by artifact refs

 |

---

## Implementation Readiness

`plan4.md` satisfies all structural requirements necessary to begin dispatching coding agents:

1. **Acyclic Module Boundaries:** Go repository definitions (`[github.com/PithomLabs/physmath](https://github.com/PithomLabs/physmath)`, `phys`, `phys-qm`, etc.) can be configured with strict import guardrails without risking cyclic dependencies.


2. **Stable Artifact References:** The `(namespace, artifact_id/digest, schema_version)` specification provides a content-addressable model across independent storage layers.


3. **Phased Rollout Path:** Implementation can proceed strictly in the order specified in Section 44:


1. `PhysMath` mathematical core & canonicalization engine.


2. Artifact/reference codecs (`phys-interop` if needed).


3. `phys-*` external library contract specification.


4. Initial domain libraries (`phys-qm`, `phys-bohm`, `phys-gr`).


5. `Phys` formalization protocol integration & `TOE-Agent` research orchestration.





`plan4.md` is approved as the active architectural baseline.