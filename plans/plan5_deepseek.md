## Adversarial Review — `plan5.md` v1.2

**Context reviewed:** `plan5.md` against the prior reviews, `background.md`, `TOE_gpt.md`, `AGENTS.md`, `README.md`.

**Verdict:**  
**NOT YET CLOSED FOR IMPLEMENTATION PLANNING.**  
The document claims `ARCHITECTURE CLOSED FOR IMPLEMENTATION PLANNING` and `Architectural blockers: NONE`. Neither claim is supported by the document’s contents. Twelve architectural decisions remain unresolved, and each violates §47’s own no-invention rule. A coding agent handed this document would still have to invent architecture, which §47 explicitly forbids.

The reference-DAG model, `PhysMath ⊥ Phys`, the trust/interpretation separation, and the protocol framing of formalization are all correct and should be preserved. The failure mode is the one that has persisted since plan2: **the document names artifact classes and boundaries but does not pin their schemas, ownership, or composition rules.**

---

## The persistent pattern

§6 declares:

> Every durable artifact MUST have: `namespace`, `artifact_id` or `content_digest`, `schema_version`, `artifact_type`, `canonical_payload`.

The document then defines or names:

- `ArtifactRef` (§5)
- `MathArtifact` (§9 in v1.1, subsumed into §6 here)
- `PhysicalArtifact` (§17)
- `PhysicalInterpretation` (§12)
- `ProposedInterpretation` (§13)
- `CorrespondenceProposal` (§14)
- `MathTrace` (§30)
- `ResearchHypothesis` (§35.2 states)
- `FormalizationProposal` / `FormalizationResponse` (§26, §28)
- `ComputationalContract` (§39)
- `MathObjectRef` (§30)
- Status vocabularies (§35)

None of these has a specified schema. Each has field names or a list of concerns; none has field types, canonical form, identity rules, resolution rules, or validation. This is the same gap that survived plan2, plan3, and plan4.

---

## Architectural blockers

### N-1. `ArtifactRef` has no owning module, and §5/§6 are inconsistent

`ArtifactRef` is the universal cross-boundary identity mechanism. But:

- `PhysMath ⊥ Phys` (ADR-002, ADR-003) — neither project can define a shared type that the other imports.
- `phys-interop` is deferred (ADR-033).
- No other module is named as the owner of `ArtifactRef`.

So where does it live? If each `phys-*` package defines its own, cross-project references are incompatible. If one shared module owns it, that module is a new authority — which contradicts ADR-033 and §34.

Second, §5 says identity is:

```
SHA-256(Namespace || SchemaVersion || ArtifactType || Payload)
```

but §6 says a durable artifact may have `artifact_id` **or** `content_digest`. §5 does not mention `artifact_id`. The two sections disagree on whether artifacts are content-addressed, name-addressed, or both.

Third, `ArtifactRef = {Namespace, ArtifactType, SchemaVersion, Digest}` redundantly carries the identity prefix that §5 says is folded into the digest. Whether two `ArtifactRef`s with the same `Digest` but different `Namespace` are equal is undefined.

**Exit condition:** declare the owning module of `ArtifactRef`, resolve `artifact_id` vs `content_digest`, and specify whether the tuple or the digest is the identity.

---

### N-2. The PhysMath IR is admitted to be unpinned (§8)

§8 says:

> The exact final core sort set is implementation-pinned before coding.

Then §46 declares `CLOSED`, and §47 forbids coding-agent invention of IR ownership. These three statements cannot all hold.

§23 offers further confusion by defining the Phys projection subset as seven node types (`Symbol, Rational, Add, Mul, Neg, Pow, Sqrt`). Is that seven-type subset the MVP IR? Or is the sixteen-sort list in §8 the MVP IR? The document does not say.

**Exit condition:** pin the MVP sort set explicitly and state whether the additional sorts are v0.2+ or out of scope.

---

### N-3. Partial projection classification is undefined (§23–§25)

§24 says classification is based on “the explicit projection contract.” §23 defines the projection subset. But the rule for a composite is not stated:

- A `Tensor` whose entries are all projectable — is it `FULLY`, `PARTIALLY`, or `UNREPRESENTABLE`?
- A `Function` whose body is projectable but whose domain/codomain are not — same question.
- A `Transformation` whose steps are individually projectable — same question.

Without a rule, a coding agent must invent it. §47 forbids this.

**Exit condition:** a compositional classification rule: for each sort, how does projectability of children combine into projectability of the parent?

---

### N-4. `ProposedInterpretation → PhysicalInterpretation` promotion is unspecified (§13)

§13 says Project A may hold a `ProposedInterpretation` before it becomes a `phys-*` `PhysicalInterpretation`. It does not say:

- who promotes;
- what validation the promotion requires;
- what happens when a `phys-*` package disagrees with a proposed interpretation;
- whether a `phys-*` package can unilaterally reject;
- whether a proposed interpretation can be consumed by a numerical layer before promotion.

Without this, `ProposedInterpretation` and `PhysicalInterpretation` are indistinguishable in practice — which defeats the whole point of §13.

**Exit condition:** a promotion contract naming the authority, the required validation, and the rejection path.

---

### N-5. Formalization protocol has no messages (§26, §28)

§26 gives a flow diagram. §28 lists outcomes. Neither specifies:

- the schema of `FormalizationProposal`;
- the schema of `FormalizationResponse`;
- the state machine of transitions;
- what `REQUEST_REVISION` actually contains;
- who validates projectability;
- whether the same proposal can be retried after revision or must be re-created;
- which parties issue which outcomes.

§44 says “Formalization tests” exist, but there is nothing to test without schemas.

**Exit condition:** concrete schemas for the proposal and the response, plus a state machine, plus a validator specification.

---

### N-6. `t`/`k` distinction is supported, not enforced (§38)

§38 says PhysMath “models both as mathematical parameters” and “must support `∂/∂t` and `∂/∂k` as different operations.” Support is not enforcement. Both are symbols. Nothing in §8 or §17 requires the IR to prevent conflation.

§44 lists a `t`/`k` test but gives no criterion for what would fail. A test without a failure criterion is not an enforcement.

**Exit condition:** an explicit representation in the IR that distinguishes parameters by role, plus a concrete negative test that rejects the conflation.

---

### N-7. `PhysicalContract` has no schema (§17)

§17 lists:

```
ExtensionID
SchemaVersion
PhysicalType
PhysicalContract
MathRefs
FrameworkScope
InterpretationClass
RequiredConditions
RequiredConventions
CompatibilityRules
```

No types. Specifically:

- What is `PhysicalContract`? A document reference? An inline struct? A closure?
- What is `CompatibilityRules`? A set of predicates? A version range? A hand-written check?
- Who validates `MathRefs` against actual PhysMath artifacts?
- Who validates `RequiredConventions` against the Phys canonical convention model?

**Exit condition:** concrete type for `PhysicalContract` and a definition of `CompatibilityRules`, plus a validation rule.

---

### N-8. §48's “frozen Phys specification” is undefined

§48 says implementation is authorized when “this document and the frozen Phys specification are supplied together.” The document does not name:

- which file(s) constitute the frozen Phys spec;
- the spec’s version identifier;
- whether the spec is `README.md`, `AGENTS.md`, `specs_v2_3.md`, or all three;
- how the coding agent knows it has the right version.

Without this, §48’s authorization condition cannot be checked.

**Exit condition:** pin the spec identifier and version.

---

### N-9. Status vocabularies are lists of strings, not types (§35, §44)

§35 lists four vocabularies. None is defined as a data type. Specifically:

- Are they enums? Structs with payloads?
- Do they serialize as strings, integers, or tagged unions?
- What are the legal transitions?
- Is `AWAITING_FORMALIZATION` reachable from `REVISION_REQUIRED`?
- Can `COUNTEREXAMPLE_FOUND` transition to `UNDER_ATTACK` again?
- Is `CLOSED` terminal?
- What data does each state carry? (e.g., `COUNTEREXAMPLE_FOUND` presumably carries the counterexample; the document does not say.)

Without a state machine, research workflows cannot be implemented or tested.

**Exit condition:** data type definitions and transition rules for each vocabulary.

---

### N-10. The no-invention rule is unsatisfiable from the document (§47)

§47 forbids coding-agent invention of architecture for:

```
dependency direction
artifact identity
namespace ownership
in-memory versus serialized composition
IR ownership
mathematical conditions
physical interpretation ownership
trust ownership
projection subset
formalization state
replay authority
phys-* conformance
status vocabularies
numerical boundary
Growth Gate routing
```

Of these, at least the following are still undefined in the document:

- `artifact identity` (N-1)
- `namespace ownership` (N-1)
- `IR ownership` (N-2)
- `physical interpretation ownership` (N-4)
- `projection subset` (N-3)
- `formalization state` (N-5)
- `status vocabularies` (N-9)

The rule and the document are in direct contradiction. Either the document must be extended to specify these, or §47 must be relaxed for the listed concerns.

---

### N-11. Project A's role in formalization is under-specified (§26)

§26 says the formalization protocol is “implemented by Project A integration tooling.” But §2.4 says Project A owns research state and does **not** own physical semantics.

Two questions follow:

1. Does Project A orchestrate calls to Phys’s public API? If so, Project A must know enough about Phys to drive the protocol — but §2.4 says Project A does not own Phys semantics.
2. What prevents Project A from bypassing the protocol and directly invoking Phys APIs to fabricate a Phys artifact? §47 forbids coding-agent invention, but there is no architectural enforcement.

**Exit condition:** an explicit statement that the formalization protocol is a Phys-owned or `phys-interop`-owned driver, and a mechanism that prevents Project A from manufacturing Phys artifacts without going through it.

---

### N-12. §45 supersession is unanchored

§45 says “This document explicitly supersedes the following previous decisions.” It lists six prior decisions but does not cite the documents or versions containing them.

A coding agent cannot verify supersession without knowing which prior documents exist. If the coding agent is handed `plan3.md` and `plan5.md` together, it cannot tell which decisions are still active.

**Exit condition:** cite document IDs and versions for every superseded decision.

---

## Additional residual issues worth naming

These are smaller but still structural:

### R-1. `MathObjectRef` is used in §30 but never defined

§30 says a `MathTrace` contains `MathObjectRefs`. §5 defines `ArtifactRef`. Are they the same type? Different scopes? If different, the schema must be specified.

### R-2. §44's tests are names, not test specifications

The “Adversarial tests required before implementation-plan handoff” (§44) are named but not specified. For example:

- “PhysMath core → no Phys” is a policy; the actual check (package-graph test? import lint?) is not specified.
- “one MathArtifact → multiple scoped interpretations” has no criterion for what “scoped” means in a test.

Without specified tests, the acceptance criterion is not checkable.

### R-3. §15’s multi-interpretation rule has no conflict-resolution mechanism

§15 says conflicting interpretations are “a research/physical-contract issue governed by their declared frameworks.” It does not say:

- whether two `phys-*` packages can claim the same `(FrameworkRef, PhysicalType)` pair for the same MathArtifact;
- whether one supersedes the other;
- what a consumer does when both resolve.

This matters for the TOE’s Dancer↔Stage, Dance↔Stage, and Dancer↔Dance interfaces, which cross `phys-qm`, `phys-bohm`, and `phys-cst`/`phys-cdt`.

### R-4. §32 and §17 are in tension

§32 says “typed domain structs remain the primary in-memory representation; serialized artifacts use `ArtifactRef`.” §17 says a `PhysicalArtifact` has `MathRefs`, `RequiredConditions`, `RequiredConventions`. Are these Go values in-memory and `ArtifactRef`s in serialized form? The dual representation is not specified.

### R-5. §34 “standard encoding/json” is a decision, not a specification

Serialization is an architectural concern under §47 (“artifact identity”). §34 delegates it to “standard encoding/json + package-owned schemas,” but JSON has no canonical form. Two Go programs using `encoding/json` on the same struct can produce different bytes (map iteration, float formatting, unicode escaping). This contradicts §7’s determinism requirements.

Either the canonicalization rule must constrain JSON output, or the serialization format must be pinned.

### R-6. §23’s projection subset is stated but not justified

Why these seven node types? They correspond to Phys’s existing `Expr` node kinds. That is reasonable, but the document does not say “these correspond exactly to Phys’s seven `Expr` kinds and no others,” so a coding agent cannot verify that the projection subset matches the frozen Phys substrate.

---

## What plan5 gets right

These decisions should be defended against rollback in any revision:

- `PhysMath ⊥ Phys` at the direct dependency level (ADR-002, ADR-003).
- Reference-DAG model rather than a vertical stack.
- Artifacts, references, interpretations, trust assertions, and research hypotheses as independent classes.
- `TrustAssertion` explicitly demoted to a future design target rather than an assumed existing Phys runtime type (ADR-025).
- No runtime `AcceptFormalization` / corpus-promotion API (ADR-030).
- Replay authority local to the producing system (ADR-031).
- `phys-interop` deferred, conformance mandatory (ADR-033).
- `t`/`k` distinction restored.
- Explicit Math vs Phys vs Trust vs Research vs Numerical vocabulary separation.
- No-Go list (§42) and supersession list (§45) as explicit architectural commitments.

These are the load-bearing decisions the whole ecosystem depends on.

---

## Path to actual closure

The document is not one small step away from closure. It is one schema-pinning pass away. The concrete work is:

For each artifact class:

```
Schema:        concrete field types
Identity:      how content-addressing works
Canonical form: serialization bytes
Composition:   what it may reference and how refs resolve
Ownership:     which module/authority owns the type
Validation:    what makes an instance legal
Tests:         at least one positive and one negative
```

Applied to:

- `ArtifactRef` (N-1, R-1)
- `MathArtifact` / `Expression` / node types (N-2)
- `PhysicalArtifact` / `PhysicalContract` (N-7, R-4)
- `PhysicalInterpretation` (N-4, R-3)
- `ProposedInterpretation` (N-4)
- `CorrespondenceProposal` (R-3)
- `TrustAssertion` (marked future, no schema needed now)
- `MathTrace` (R-1)
- `ResearchHypothesis` (N-9)
- `FormalizationProposal` / `FormalizationResponse` (N-5)
- `ComputationalContract` (N/A — explicitly out of scope)
- Status vocabularies (N-9)
- `ArtifactRef` owning module (N-1)
- Frozen Phys spec identifier (N-8)
- Supersession citations (N-12)

Once each of those has a schema, an ownership, and a test, §47 can honestly claim its no-invention rule.

---

## Final verdict

```text
REVIEW: plan5.md v1.2

Architectural model:                     PASS
Reference-DAG mental model:              PASS
PhysMath ⊥ Phys separation:              PASS
Trust as future design target:           PASS
Interpretation as phys-* artifact:       PASS
Formalization as protocol:               PASS
No runtime corpus promotion:             PASS
Authority-local replay:                  PASS
Ownership table:                         PASS
Supersession list:                       PASS
No-Go list:                              PASS

ArtifactRef ownership and identity:      OPEN (N-1)
PhysMath IR pinning:                     OPEN (N-2)
Partial projection rule:                 OPEN (N-3)
Interpretation promotion:                OPEN (N-4)
Formalization message schemas:           OPEN (N-5)
t/k enforcement mechanism:               OPEN (N-6)
PhysicalContract schema:                 OPEN (N-7)
Frozen Phys spec identifier:             OPEN (N-8)
Status state machines:                   OPEN (N-9)
No-invention rule satisfiability:        OPEN (N-10)
Project A formalization role:            OPEN (N-11)
Supersession anchoring:                  OPEN (N-12)

Residual:
  MathObjectRef definition:              OPEN (R-1)
  Test specifications:                   OPEN (R-2)
  Multi-interpretation conflicts:        OPEN (R-3)
  In-memory vs serialized duality:       OPEN (R-4)
  JSON canonicalization:                 OPEN (R-5)
  Projection subset justification:       OPEN (R-6)

FINAL STATUS: NOT CLOSED FOR IMPLEMENTATION PLANNING
```

The architecture is correct. The document is not. §47’s no-invention rule is the right commitment — but it cannot be satisfied by declaring it. It can only be satisfied by pinning the schemas, types, and tests that make the rule checkable. Until then, plan5.md is an accurate architectural manifesto with unimplemented interfaces.