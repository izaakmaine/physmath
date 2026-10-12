## Adversarial Review — `plan4.md` v1.1

**Context reviewed:** `plan4.md` against `background.md`, `TOE_gpt.md`, `AGENTS.md`, `README.md`, and the prior reviews.

**Verdict:**  
**NOT YET IMPLEMENTATION-AUTHORIZED.**  
The architectural *reasoning* is now correct. The reference-DAG model, the `PhysMath ⊥ Phys` separation, the split of identity/interpretation/trust, and the protocol-not-artifact framing of formalization are all genuine corrections. The document has moved from “wrong mental model” to “right mental model.”

But it repeats the exact pattern the previous reviews flagged: **it names the artifact classes and the invariants without specifying their schemas, identity rules, composition rules, or tests.** §48 states the completion criterion as *“No architectural decision is left for the coding agent to invent.”* That criterion is not met. A coding agent handed this document would still have to invent the reference type, the artifact identity mechanism, the interpretation ownership rule, the trust-assertion attachment path, the formalization message schema, the state machines, and the numerical contract.

Below are the architectural blockers that remain, plus new ones created by the reference-DAG model itself.

---

## I. Real improvements worth preserving

Before listing the blockers, credit where credit is due. These are correct and should survive any revision:

- `PhysMath ⊥ Phys` at the direct dependency level (§0, §5). This removes the entire class of “how does mathematics import physics without becoming physics” problems.
- No universal envelope (§8). The four-way split of `Artifact`, `Reference`, `Interpretation`, `TrustAssertion`, `ResearchHypothesis` is the right decomposition.
- `TrustAssertion` as a first-class artifact referencing a subject (§11). This makes trust a property of *a relationship between an authority and an artifact*, not a property baked into every artifact.
- `PhysicalInterpretation` as a first-class artifact (§12). This cleanly dissolves the wavefunction problem — the same math object has multiple interpretations, none of which owns it.
- Formalization as a protocol with typed outcomes (§31–§32). This is the correct shape.
- Replay authority being local, with no fixed number of replay domains (§28). This correctly decouples replay from layer hierarchy.
- Explicit rejection of a permanent Interpretation Layer, a Candidate Physical Object type, and Phys kernel growth for TOE convenience (§42).

The architecture is now sound. The remaining work is *mechanical closure*.

---

## II. The persistent pattern: named artifacts, unspecified schemas

§6 declares the universal primitive is a “referenceable artifact.” The document then defines:

- `MathArtifact` (§9)
- `PhysicalArtifact` (§10)
- `TrustAssertion` (§11)
- `PhysicalInterpretation` (§12)
- `FormalizationProposal` / `AcceptedPhysicalArtifact` (§31)
- `CorrespondenceProposal` (§30)
- `ResearchHypothesis` (§29)
- `MathTrace` (§28)
- `ArtifactRef` (§5.2)

None of these has a specified schema. Each has field names or a short list of concerns; none has field types, canonical form, identity rules, or validation. §48 claims the coding agent will not have to invent decisions, but the coding agent will in fact have to invent every one of the above.

This is the same gap that survived plan2 and plan3. It is not a nitpick; it is the reason the document cannot yet be implemented.

---

## III. Blockers that persist from prior reviews

Each of these was previously listed. Each remains open. None has been closed by v1.1.

### B-003 / IR composition rules (§22–§23)

§22 lists 16 sorts. §23 requires each sort to define domain, codomain, construction, application, composition, canonical form, equality, serialization, hashing, and resource limits. It does not specify these. A coding agent must invent them, which §48 forbids.

The specific unresolved case: `Function`, `Functional`, `Operator` are named as peers but distinguished only by behavior. Without composition rules, they cannot be implemented.

### B-004 / Canonicalization (§6)

The document says identity may be content-addressed “provided the final implementation satisfies the deterministic identity requirements.” But it does not specify:
- which sorts are content-addressed and which are name-addressed;
- what the canonical form of a `MathArtifact` is;
- what the canonical form of a `PhysicalArtifact` is;
- how `namespace` is scoped;
- whether `artifact_id` and `content_digest` are alternatives or complements.

“or content_digest” is a disjunction the coding agent must resolve.

### B-011 / `PhysicalArtifact` schema (§10)

The document lists:
```
artifact_identity
extension_id
physical_type
math_refs[]
physical_contract
```
with no types, no validation, no canonical form. In particular:
- What is a `math_ref`? A tuple? A digest? A namespace-qualified identifier?
- What is `physical_contract`? A free-form struct? A reference? An inline schema?
- Where is `physical_contract` stored, and who validates it?

### B-012 / `TrustAssertion` attachment (§11)

The document shows a trust assertion above a physical artifact. It does not specify:
- how a `TrustAssertion` is created (Phys API? Curation workflow? Both?);
- how a `subject_ref` identifies a `phys-*` object that Phys's frozen substrate cannot represent;
- whether `TrustAssertion` can reference a PhysMath artifact;
- how a client discovers the trust assertions that apply to a given artifact.

The last point is critical: without an index or lookup contract, “trust is separate” becomes “trust is unfindable.”

### B-013 / Formalization message schema (§31–§32)

The document gives a state-transition diagram and a list of outcomes. It does not give:
- the schema of `FormalizationProposal`;
- the schema of the Phys response (`Accept`, `Reject`, `RequestRevision`);
- the schema of `AcceptedPhysicalArtifact`;
- the state machine’s transition rules.

A diagram is not a protocol.

### B-014 / Loss accounting (§33)

`FULLY_REPRESENTABLE`, `PARTIALLY_REPRESENTABLE`, `UNREPRESENTABLE` are named. The loss report is not specified:
- what a “loss” is structurally;
- who verifies the loss report is complete;
- how a `PARTIALLY_REPRESENTABLE` item records which conditions, approximations, or obligations were dropped.

### B-015 / `t` vs `k` typing (§19 in v0.5; absent in v1.1)

The prior plan stated the `t`/`k` distinction as a mandatory adversarial test. v1.1 does not mention it. This is a regression. The reference-DAG model makes it easier to represent, but the *requirement* has been dropped, not just the wording.

### B-016 / TOE joint-dynamics acceptance

v1.1 says less about the TOE than v0.5 did. §37–§38 say the foundation must permit the TOE workload without committing to it. That is correct architecturally, but the acceptance criterion that previously existed in v0.5 §30–§31 (one FULLY and one PARTIALLY/UNREPRESENTABLE path) has not been carried forward as a formal contract. It is not clear whether v1.1 retains this.

### B-017 / AI-agent protocol

§34 says schemas are “versioned.” No schema is defined. §39 gives a workflow, not a message format. §22/§24/§29/§30 give field names.

### B-018 / Numerical handoff contract

§34 says the Numerical Layer consumes a “physical computational contract.” That contract is not defined. Who produces it? What fields does it carry? How is completeness validated?

### B-019 / Growth Gate procedure (§18)

Eight criteria are listed. No procedure, no evidence requirement, no decision record. “The TOE is not by itself sufficient evidence” is a rule, not an enforcement.

### B-020 / Resource contracts (§41)

The bounded quantities are listed. No defaults, no units, no contract, no `ResourceLimit` error type.

---

## IV. New blockers created by the reference-DAG model

The v1.1 model introduces new structural questions that the earlier versions did not have.

### N-1. The reference type is the new universal primitive, and it is undefined

Every cross-artifact relationship in the document (`math_refs[]`, `subject_ref`, `framework_ref`, `physical_contract_ref`, `mapping_refs[]`, `TrustAssertion.subject_ref`, `PhysicalInterpretation.math_ref`, `ArtifactRef`) relies on a reference type that is never defined.

The document says a reference is “namespace + digest” informally (§5.2, §6), but never commits. The shape of the reference is the single most load-bearing decision in the architecture, and it is left to the coding agent.

**Exit condition:** a concrete `ArtifactRef` schema with namespace, identity mechanism, and resolution contract.

### N-2. Artifact location and registry ownership

If `MathArtifact`, `PhysicalArtifact`, `TrustAssertion`, `PhysicalInterpretation`, `CorrespondenceProposal`, and `ResearchHypothesis` are all artifacts, where do they live?

- Is there one registry or one per authority?
- Who owns the namespace of each artifact class?
- How does an artifact get discovered by a consumer?

The document says artifacts are "referenceable" but never says "registered." Without a registry contract, references cannot be resolved.

**Exit condition:** a registry contract specifying who owns which namespace and how artifacts are discovered.

### N-3. `PhysicalInterpretation` ownership is unstated

§12 introduces `PhysicalInterpretation` as a first-class artifact. But §3's ownership table does not list it. It is not clear whether:

- `phys-*` owns it (framework-owned interpretation);
- Phys owns it (curation-level interpretation);
- TOE-Agent owns it (research-level interpretation);
- multiple owners can create independent interpretations of the same math artifact.

If multiple owners can, the interpretation is a framework-specific artifact and must be namespace-qualified accordingly. If only one can, the ownership must be stated.

**Exit condition:** explicit ownership and namespace rule for `PhysicalInterpretation`.

### N-4. `TrustAssertion` subject range

§11 shows a trust assertion over a “physical artifact.” The document does not say whether `TrustAssertion.subject_ref` may reference:

- a `MathArtifact` (PhysMath-owned);
- a `PhysicalArtifact` from a `phys-*` extension;
- a `PhysicalInterpretation`;
- a `CorrespondenceProposal`.

If trust is meant to annotate any of these, the subject range must be declared. If it is only for physical artifacts, the reason must be stated.

**Exit condition:** explicit subject range and rationale.

### N-5. `phys-*` → Phys trust reference contract

The dependency graph says `phys-* → Phys`. But the artifact model says trust is a separate `TrustAssertion` owned by Phys. So how does a `phys-*` artifact acquire a trust assertion?

Two possible mechanisms exist:
1. `phys-*` requests a trust assertion; Phys issues one and stores it.
2. Phys crawls `phys-*` artifacts and attaches trust assertions on its own authority.

The document does not choose. Without a choice, the enforcement is unspecified.

**Exit condition:** the direction of trust-attachment requests.

### N-6. Phys referencing external artifacts

§11 implies Phys must be able to reference a `phys-*` artifact as a subject. But Phys's frozen object model is bounded. The document does not address:

- whether Phys's `TrustAssertion` schema can carry a reference to an artifact Phys itself cannot represent;
- if so, what its validation rules are;
- if not, what the trust boundary actually covers.

This is the same tension as prior plans, resurfacing at the trust layer.

**Exit condition:** an explicit rule for how Phys trust-asserts on artifacts outside its own formal model.

### N-7. `phys-interop` optionality

§35 introduces a shared infrastructure module but says it “MAY be introduced.” If the boundary rules require shared mechanics (reference resolution, schema utilities, conformance tests, import-boundary checks), then it should be mandatory. If it isn't required, then the boundary rules must be implementable independently by each `phys-*` package.

The two cases produce different architectures. “MAY” leaves the choice to the coding agent, which §48 forbids.

**Exit condition:** decide whether `phys-interop` is mandatory, and if so, which modules depend on it.

### N-8. Status transitions and state machines

§40 lists four status vocabularies. None has a transition rule. `AWAITING_FORMALIZATION` vs `FORMALIZED` vs `PARTIALLY_ACCEPTED` overlap. `UNDER_ATTACK` vs `COUNTEREXAMPLE_FOUND` vs `INCONSISTENT` are not ordered. A research agent cannot decide from §40 alone whether a hypothesis can return to active research from `COUNTEREXAMPLE_FOUND`, or whether `FORMALIZED` is terminal.

**Exit condition:** state machine per vocabulary.

### N-9. `CorrespondenceProposal` vs `PhysicalInterpretation` overlap

A `CorrespondenceProposal` (§30) states a proposed relationship between artifacts. A `PhysicalInterpretation` (§12) states that a math artifact means a physical thing in a framework.

Are these the same kind of artifact viewed from different scopes? Can a correspondence proposal generate a physical interpretation? Can a physical interpretation imply a correspondence proposal?

The document does not distinguish them beyond field lists. In practice this will cause duplicate schemas.

**Exit condition:** clear scope separation between “an interpretation exists” and “a correspondence is proposed.”

### N-10. Multi-framework interpretation of one math artifact

The wavefunction example (§13) says one math object can have QM, Bohmian, and “another explicitly scoped interpretation.” But §12's `PhysicalInterpretation` schema does not show how multiple interpretations of the same math object coexist in the artifact registry. In particular:

- who arbitrates when two interpretations claim the same scope;
- whether a math object can accumulate an unbounded number of interpretations;
- whether an interpretation can reference another interpretation.

**Exit condition:** coexistence rule and registry behavior.

---

## V. What v1.1 gets right that should be defended

To make the review actionable, these are the load-bearing decisions that must not be rolled back during revision:

- The `PhysMath ⊥ Phys` separation.
- Artifacts as first-class, references as the cross-boundary mechanism.
- Trust as an artifact referencing a subject, not a field of every object.
- Interpretation as a first-class artifact.
- Formalization as a protocol.
- Replay authority local to the producing system.
- Rejection of a universal envelope, an Interpretation Layer, and a Candidate Physical Object layer.
- Ownership table (§3) as normative.

The revision should build on these, not reopen them.

---

## VI. Final verdict

```text
REVIEW: plan4.md v1.1

Architectural model:                     PASS
Reference-DAG mental model:              PASS
PhysMath ⊥ Phys separation:              PASS
Trust as separate artifact:              PASS
Interpretation as separate artifact:     PASS
Formalization as protocol:               PASS
No universal envelope:                   PASS
Ownership table:                         PASS
No-Go list:                              PASS

Artifact schemas:                        OPEN (every artifact class)
Reference type schema:                   OPEN (N-1)
Registry ownership:                      OPEN (N-2)
Interpretation ownership:                OPEN (N-3)
Trust subject range:                     OPEN (N-4)
Trust attachment direction:              OPEN (N-5)
Phys referencing external artifacts:     OPEN (N-6)
phys-interop mandatory or optional:      OPEN (N-7)
Status state machines:                   OPEN (N-8)
Correspondence vs interpretation:        OPEN (N-9)
Multi-interpretation coexistence:        OPEN (N-10)
IR composition rules:                    OPEN (B-003)
Canonicalization rules:                  OPEN (B-004)
Loss accounting:                         OPEN (B-014)
t/k typing requirement:                  REGRESSED (B-015)
TOE acceptance contract:                 REGRESSED (B-016)
AI protocol schemas:                     OPEN (B-017)
Numerical contract:                      OPEN (B-018)
Growth Gate procedure:                   OPEN (B-019)
Resource contracts:                      OPEN (B-020)

FINAL STATUS: NOT READY FOR IMPLEMENTATION PLAN
```

---

## VII. What would move this to actually closed

v1.1's biggest opportunity is to stop describing artifact classes and start defining them. A minimal but real closure would require, per artifact class:

```
Schema:        concrete field types
Identity:      how content-addressing works for this class
Canonical form: how to serialize and hash
Composition:   what it may reference, and how references resolve
Registry:      where it lives and who owns its namespace
Validation:    what makes an instance legal
Tests:         at least one positive and one negative test
```

Applied to:

- `ArtifactRef` (N-1)
- `MathArtifact` (B-003, B-004)
- `PhysicalArtifact` (B-011)
- `TrustAssertion` (B-012, N-4, N-5, N-6)
- `PhysicalInterpretation` (N-3, N-9, N-10)
- `CorrespondenceProposal` (N-9)
- `ResearchHypothesis` (B-017)
- `FormalizationProposal` / `AcceptedPhysicalArtifact` (B-013, B-014)
- `ComputationalContract` (B-018)
- Status state machines (N-8)
- Growth Gate procedure (B-019)
- Resource limits (B-020)

That is the concrete closure work. It is not architectural reasoning; it is mechanical specification. Once those twelve items each have a schema and a test, §48's criterion can honestly be claimed.

Until then, v1.1 is the correct architecture with unimplemented interfaces.