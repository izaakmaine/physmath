The architecture defined in `AGENTS.md` and `README.md` establishes a deterministic, closed-AST symbolic physics compiler rather than an open-ended Computer Algebra System (CAS), dynamic runtime compiler, or numerical simulator.

The five architectural concerns previously raised are resolved by the system's explicit non-goals, hard package boundaries, and closed mathematical primitives.

---

### Re-evaluation of Architectural Blockers

| Previously Identified Blocker | Specification Mechanism | Architectural Status & Resolution |
| --- | --- | --- |
| **1. AI Exploration vs. Package Boundaries** | `hypothesis/` package, `HYPOTHESIS` provenance, and transitively propagated contamination law (`AGENTS.md` §2, §6, §9).

 | **Resolved.** Speculative physical concepts do not require runtime Go package compilation. Concepts are minted via `hypothesis.NewCandidateConcept` with `HYPOTHESIS` provenance. Pure transformations (`ops.Apply`) transitively propagate `HYPOTHESIS` status to outputs without altering `internal/kernel` or violating domain boundaries.

 |
| **2. Formalization & Numerical Execution Bottleneck** | Explicit Non-Goals & Scope (`README.md` Scope; `AGENTS.md` §5).

 | **Resolved.** Numerical evaluation, simulation, and floating-point computations are explicitly deferred non-goals. The compiler functions strictly as a formal symbolic substrate and provenance ledger. External tools or human researchers evaluate empirical validity and numerical outputs.

 |
| **3. Go Static Compilation vs. Dynamic AI Interface** | Closed 10-node `core.Expr` AST and closed 12-operation `ops.Apply` dispatch (`README.md` Scope; `AGENTS.md` §5).

 | **Resolved.** The AI agent does not generate or compile Go source code at runtime. AI agents manipulate machine-readable JSON representations built from 10 fixed expression node kinds (`Symbol`, `Call`, `Add`, `Pow`, etc.) and apply 12 canonical operations.

 |
| **4. Multi-Flow Dynamic Representation Gap ($\partial_t$ vs. $\partial_k$)** | Bounded framework scope (`relativity/`, `mechanics/`) (`AGENTS.md` §9, §10).

 | **Resolved.** `relativity/` is strictly bounded to Special Relativity in flat Minkowski spacetime with signature $-+++$. General Relativity, quantum gravity, and multi-flow differential geometry exceed MVP scope. Speculative frameworks reside in `hypothesis/` built from standard AST primitives.

 |
| **5. Undecidability & Equivalence Paradox** | Deterministic SHA-256 canonicalization and `Session.Identify` firewall (MRC-006) (`AGENTS.md` §5; `README.md` Authority boundaries).

 | **Resolved.** The system does not attempt open-ended general equivalence proving. Identity relies on canonical JSON/SHA-256 structures and explicit `Simplify`/`Substitute` passes. Non-trivial mathematical equivalences are asserted explicitly through `Session.Identify` with strict dimension/kind checks and audit log justifications.

 |

---

### Key Operational Guarantees

#### 1. Independent Status Axes

Objects carry two independent status axes that must not be conflated:

* **Provenance Status:** `DEFINED`, `POSTULATED`, `DERIVED`, `IDENTIFIED`, `APPROXIMATED` (reserved), or `HYPOTHESIS`. Computed deterministically by pure operations.


* **Corpus Status:** `NONE`, `ESTABLISHED`, `CONTESTED`, `SUPERSEDED`, or `FALSIFIED`. Human-curated manifest metadata; never altered by formal derivation or code execution.



#### 2. Closed Operation Set

Mathematical transformations are strictly restricted to 12 pure operations evaluated at exact positional indices:


$$\text{Ops} \in \{\text{add}, \text{subtract}, \text{multiply}, \text{divide}, \text{pow}, \text{simplify}, \text{substitute}, \text{differentiate}, \text{limit}, \text{compare}, \text{solve}, \text{select\_branch}\}$$


`ops` functions are session-free and pure. Only `session.Session` can commit ledgers, invoke `Identify`, or seal a `ResearchCandidate`.

#### 3. Candidate Containment Firewall (MRC-008)

Hypotheses can never auto-promote to established status. Any derivation step relying on a hypothesis that presents a trusted provenance status, a non-`NONE` corpus status, or an untracked hypothesis field triggers a `CandidateContainmentError`. No promotion APIs (`PROMOTE`, `TRUST`, `APPROVE`) exist in the codebase.

#### 4. Import & Dependency Hierarchy

```text
internal/kernel  <-- Theory-neutral formal substrate (Expr, Object, Mint authority)[cite: 32, 33]
       ^
     core        <-- Public API aliases & manifests[cite: 32, 33]
       ^
      ops        <-- Pure transformation functions (12 ops only)[cite: 32, 33]
       ^
    session      <-- Ledger replay, Identify, and ResearchCandidate sealing[cite: 32, 33]

mechanics / relativity  --> Import core + kernel (never session or ops directly)[cite: 32, 33]
hypothesis             --> Imports core + kernel (never domain frameworks at build time)[cite: 32, 33]

```

---

### Execution Guidance for AI Reasoning

1. **Reason from Premises & Assumptions:** Equations are bound to explicit regimes, metric signatures (e.g., $-+++$), and dimensional constraints. Assumptions must be tracked and merged across every step—never dropped or introduced implicitly.


2. **Derive Targets directly:** Stored target objects (such as `MassEnergyRelation`) must never be used as derivation premises for their own derivation.


3. **Handle Branch Choices Explicitly:** Branch selections (`select_branch`) require explicit constraints (e.g., non-negativity constraints) recorded as structured assumptions.