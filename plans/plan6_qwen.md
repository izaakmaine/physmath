# Adversarial Review of `plan6.md` (PhysMath/Phys/Phys-* Architecture v1.2)

## Executive Verdict

`plan6.md` correctly establishes the conceptual separation between mathematics, physical semantics, trust, and research state. The `PhysMath ⟂ Phys` orthogonality decision is architecturally sound. However, the document contains **seven architectural blockers** that will prevent a coding agent from producing a working implementation. These blockers arise from underspecified cross-module contracts, ambiguous identity mechanisms, and unresolved tensions between the multi-repository ecosystem described in plan6 and the single-repository MVP defined in `specs_v2_3.md`.

The document is a strong conceptual foundation but is not yet implementation-ready.

---

## BLOCKER 1: PhysMath ⟂ Phys Orthogonality Creates an Unresolved Type-Sharing Problem

**The tension:**
- §5.1 states: "PhysMath MUST have no dependency on Phys" and "Phys MUST NOT depend on PhysMath."
- §6 states: "An artifact can be referenced without importing the code that produced it."
- §5.2 states: "When PhysMath needs to interact with Phys artifacts, that interaction occurs in interop/adapter/research integration code."

**The problem:** If PhysMath and Phys share no Go types, no Go imports, and no common type system, then a `phys-*` package that depends on both faces a fundamental composition problem. When `phys-gr` needs to attach a PhysMath mathematical expression to a Phys trust assertion, it cannot do so in a type-safe way because the two type systems are disjoint.

**Concrete example:** `phys-gr` wants to create a `GRMetric` that contains a mathematical tensor (from PhysMath) and a trust assertion (from Phys). How?
- Option A: `phys-gr` imports both and holds two separate Go values. But then composition is manual and unenforced.
- Option B: `phys-gr` serializes the PhysMath artifact to canonical JSON and stores it as a string reference inside the Phys trust assertion. But then type safety is lost.
- Option C: A shared `phys-interop` module provides a common reference type. But §43 says `phys-interop` is optional and "MUST NOT provide physics ontology, mathematics, trust authority."

**Required resolution:** The plan must specify exactly one mechanism for cross-module composition. The most viable option for Go is: **artifact references are opaque string IDs (content-addressed digests), and composition happens at the `phys-*` level through explicit reference fields, not shared Go types.** This must be stated explicitly, not left as "may be content-addressed."

---

## BLOCKER 2: Artifact Identity Mechanism is Underspecified

**The tension:**
- §6 states artifacts have `namespace`, `artifact_id or content_digest`, `schema_version`, `artifact_type`, `canonical_payload`.
- The phrase "artifact_id **or** content_digest" is ambiguous.

**The problem:** A coding agent needs to know exactly which identity mechanism to implement. "Or" is not a specification. Content-addressed identity (digest of canonical payload) and assigned identity (UUID/namespace-qualified name) have fundamentally different properties:
- Content-addressed: identity changes when content changes; no central registry needed; deduplication is automatic.
- Assigned: identity is stable across content changes; requires a registry; deduplication is manual.

**Required resolution:** Pick one. For this architecture, content-addressed identity is the correct choice because:
1. It eliminates the need for a central registry (which would become a hidden authority).
2. It makes artifact integrity self-verifying.
3. It aligns with the existing Phys ledger's hash-chain design (`specs_v2_3.md` §16.17).

The plan must state: **"Artifact identity is content-addressed. The artifact ID is the SHA-256 digest of the canonical payload. The `artifact_id or content_digest` ambiguity is resolved: they are the same thing."**

---

## BLOCKER 3: Relationship Between plan6 Ecosystem and MVP is Unstated

**The tension:**
- `specs_v2_3.md` defines a 39-file single-repository MVP at `github.com/PithomLabs/phys`.
- `plan6.md` §43 describes 9 separate repositories: `physmath`, `phys`, `phys-interop`, `phys-qm`, `phys-bohm`, `phys-gr`, `phys-cst`, `phys-cdt`, `toe-agent`.
- plan6 never states which of these are MVP and which are future.

**The problem:** A coding agent reading both documents will not know whether to implement one repository or nine. The MVP implements only `phys`. The other eight repositories are future work. But plan6 presents them all at the same level.

**Required resolution:** plan6 must explicitly state:
1. The MVP scope is the `phys` repository only (implementing `specs_v2_3.md`).
2. PhysMath, `phys-*`, `toe-agent`, and `phys-interop` are post-MVP projects.
3. The GR stress test (from the GR plans) is the first post-MVP project and will establish the pattern for `phys-*` development.
4. The plan6 architecture is the **target ecosystem**, not the MVP deliverable.

---

## BLOCKER 4: `phys-interop` Scope is Self-Contradictory

**The tension:**
- §35 says `phys-interop` "MAY be introduced" and may provide "artifact references, schema utilities, canonical-reference helpers, extension conformance tests, interop codecs, import-boundary checks."
- §35 also says it "MUST NOT provide physics ontology, mathematics, trust authority, corpus authority, research state, interpretation policy."
- §36 says it "does not turn MathArtifact into PhysicalTruth" and "does not choose QM interpretation."

**The problem:** If `phys-interop` cannot provide ontology, mathematics, trust, or interpretation, and it cannot turn math into physics, then what are the "interop codecs" actually encoding/decoding? The only thing left is canonical serialization format, which is already specified by each project's own canonical JSON rules.

**Required resolution:** Either:
- **Option A (preferred for MVP):** Eliminate `phys-interop` entirely. Each project defines its own canonical format. Cross-project references use content-addressed digests of canonical JSON. No shared infrastructure needed.
- **Option B (post-MVP):** `phys-interop` provides only schema validation utilities and cross-project conformance tests. It is a test/CI tool, not a runtime dependency.

The plan must pick one and state it explicitly.

---

## BLOCKER 5: The Wavefunction Rule Creates an Unresolved Composition Gap

**The tension:**
- §13 states: "Wavefunction as mathematics → PhysMath; QuantumState as physical object → phys-qm; BohmianWaveFunction → phys-bohm."
- §5.1 states PhysMath and Phys share no types.
- §15 states `phys-*` packages declare `math_refs[]` referencing PhysMath artifacts.

**The problem:** When `phys-bohm` creates a `BohmianWaveFunction`, it needs to:
1. Reference a PhysMath mathematical wavefunction (for computation).
2. Attach Phys trust metadata (for provenance).
3. Perform mathematical operations on the wavefunction (using PhysMath machinery).

If PhysMath and Phys share no types, step 3 requires the Bohmian package to convert between PhysMath's mathematical representation and its own physical representation on every operation. This is expensive and error-prone.

**Required resolution:** The plan must specify the **reference resolution mechanism**:
- `phys-bohm.BohmianWaveFunction` holds a `PhysMathArtifactRef` (content digest) and a `PhysTrustRef` (content digest).
- Mathematical operations are performed by PhysMath, which returns a new artifact with a new digest.
- `phys-bohm` updates its reference to point to the new artifact.
- The mathematical object is never deserialized into a Go type shared between modules.

This must be stated explicitly.

---

## BLOCKER 6: The Numerical Layer Contract is Undefined

**The tension:**
- §34 states the Numerical Layer "consumes formalized physical model + physical computational contract + explicit numerical parameters."
- §34 says it "does not infer physics, invent assumptions, repair malformed contracts, reinterpret mathematical notation, convert an unformalized hypothesis into a physical model."

**The problem:** What is a "physical computational contract"? What is its schema? What does the Numerical Layer receive as input? The plan describes what the Numerical Layer must NOT do but never specifies what it DOES receive.

**Required resolution:** The plan must define a `ComputationalContract` schema:
```
ComputationalContract {
    contract_id: content_digest
    physical_model_ref: PhysArtifactRef
    mathematical_model_ref: PhysMathArtifactRef
    parameters: {name: value, ...}
    requested_outputs: [observable_name, ...]
    numerical_method: string
    precision_requirements: {tolerance, ...}
}
```
Without this, the Numerical Layer boundary is a hand-wave, not an architectural decision.

---

## BLOCKER 7: The Growth Gate Interacts Ambiguously with the MVP Freeze

**The tension:**
- `specs_v2_3.md` §41 states the Phys kernel is frozen. No modifications permitted.
- `plan6.md` §18 describes a Growth Gate for Phys kernel evolution.
- The GR plans describe GR as a "kernel stress test" that may produce Growth Gate evidence.

**The problem:** If the Phys kernel is frozen for the MVP, the Growth Gate is a post-MVP mechanism. But the GR plans treat the Growth Gate as if it's active during the GR stress test. A coding agent will not know whether to implement the Growth Gate now or defer it.

**Required resolution:** The plan must state:
1. During MVP: Phys kernel is frozen. No Growth Gate exists. GR stress test evidence is recorded but does not trigger kernel changes.
2. Post-MVP: Growth Gate is activated. Evidence from GR (and other `phys-*` projects) is evaluated against the Growth Gate criteria.
3. The Growth Gate criteria in §18 are post-MVP evaluation criteria, not MVP implementation requirements.

---

## Additional Findings (Non-Blocking but Important)

### Finding A: `Manifold` and `Metric` in PhysMath are Ambiguous
§4.1 lists `Manifold`, `Metric`, `Connection`, `Curvature` as allowed PhysMath contents. But these are deeply tied to physics. The plan says "In ambiguous cases, a neutral mathematical name is preferred" but doesn't resolve the ambiguity for these specific cases. A `Metric` in PhysMath is a Riemannian/pseudo-Riemannian metric — a mathematical object, yes, but one whose primary use is physical. The plan should acknowledge this and state that PhysMath's `Metric` is the pure mathematical structure (a symmetric bilinear form on a tangent bundle) without physical interpretation.

### Finding B: The `phys-*` Admission Criteria are Underspecified
§17 lists 8 criteria for admitting a new `phys-*` package but doesn't specify who evaluates them or what the evaluation process looks like. For the MVP, this is fine (no `phys-*` packages exist). For post-MVP, the plan should specify that admission is a human governance decision, not an automated check.

### Finding C: The "No-Go Architectures" List Should Include One More Entry
§42 lists 14 rejected architectures. It should also explicitly reject: **"PhysMath implicitly inferring physical meaning from mathematical structure."** This is the core Reality-First principle and should be stated as a hard architectural prohibition, not just a philosophical guideline.

---

## Required Amendments Summary

| # | Blocker | Required Amendment |
|---|---------|-------------------|
| 1 | Type-sharing across PhysMath ⟂ Phys | Specify: composition via opaque content-addressed references at `phys-*` level; no shared Go types |
| 2 | Artifact identity ambiguity | Resolve "id or digest" → content-addressed digest only |
| 3 | MVP vs ecosystem scope | Explicitly state MVP = `phys` only; rest is post-MVP |
| 4 | `phys-interop` contradiction | Eliminate for MVP; post-MVP = test/CI tool only |
| 5 | Wavefunction composition gap | Specify reference resolution mechanism for cross-module math operations |
| 6 | Numerical Layer contract | Define `ComputationalContract` schema |
| 7 | Growth Gate vs MVP freeze | State Growth Gate is post-MVP; MVP kernel is frozen |

---

## Final Verdict

`plan6.md` is a strong conceptual architecture that correctly identifies the separation of concerns. However, it is **not yet implementation-ready** because the cross-module contracts are underspecified. The seven blockers above must be resolved before a coding agent can produce a working implementation.

**Status: ARCHITECTURE SOUND, CONTRACTS INCOMPLETE. Seven blockers must be resolved before implementation authorization.**