# PhysMath / Phys / phys-* Architecture
## Consolidated Review Closure and Coding-Agent Foundation

**Document ID:** PM-ARCH-FOUNDATION-001  
**Version:** 1.2  
**Status:** **ARCHITECTURE CLOSED FOR IMPLEMENTATION PLANNING**  
**Purpose:** Resolve the architectural and structural issues identified by the DeepSeek, Gemini, Qwen, and Z reviews of v1.1, with no remaining architectural ambiguity for coding agents. This document defines the reusable foundation for a large TOE research program; it does not define the TOE's physics.

---

# 0. Executive decision

The four adversarial reviews converge on the same fundamental conclusion:

> **The separation of mathematics, physical semantics, trust/formalization, and research state is correct. The remaining defects were in the joints: dependency direction, in-memory versus serialized composition, artifact identity, projection, trust attachment, protocol state, and extension uniformity.**

The revised architecture is therefore:

```text
PROJECT A — TOE-Agent / Research Program
    owns hypotheses, research state, correspondences,
    attacks, revisions, and formalization proposals

PROJECT B — PhysMath
    owns reusable mathematical structures and exact
    theoretical mathematical transformations

PROJECT C — phys-* External Libraries
    owns reusable physical-domain semantics and contracts

EXISTING Phys
    owns bounded trusted physical formalization,
    canonical physical assumptions/conventions,
    provenance, corpus status, MRC, and trusted ledger

NUMERICAL LAYER
    owns numerical execution and quantitative analysis
```

The relationship is **not a vertical inheritance stack**.

It is an artifact/reference system:

```text
                   ResearchHypothesis
                    /       |       \
                   /        |        \
                  v         v         v
             MathArtifact  PhysicalArtifact  Correspondence
                  |              |
                  |              |
                  v              v
              PhysMath        phys-*
                                   |
                                   v
                                  Phys
```

The arrows above mean **explicit references or contracts**, not inheritance.

The critical dependency rules are:

```text
PhysMath  -X-> Phys
Phys      -X-> PhysMath

phys-*    -> PhysMath
phys-*    -> Phys

TOE-Agent -> public surfaces of PhysMath, Phys, phys-*
Numerical -> Phys + selected phys-*
```

This document supersedes the earlier requirement that PhysMath depend on Phys.

---

# 1. What the four reviews establish

## 1.1 DeepSeek — net-valid findings

DeepSeek identifies the deeper problems as:

1. the relationship was incorrectly modeled as a stack rather than an artifact reference graph;
2. Phys combines a formal physical substrate and a trust authority with different reuse relationships;
3. there was no common border representation/identity mechanism;
4. replay had been artificially described as a fixed number of domains;
5. the `phys-*` family lacked a shared conformance strategy;
6. future trust/curation had been deferred without a concrete architectural direction;
7. growth across multiple projects had no common coordination mechanism.

These findings are adopted where they improve the architecture, with the exception that no separate `PhysTrust` project is required for the current foundation.

DeepSeek's principal architectural insight is retained:

> **Dependency, semantic ownership, trust, and research state are different relationships.**

## 1.2 Gemini — net-valid findings

Gemini reinforces:

- pure mathematical isolation;
- explicit domain contracts in `phys-*`;
- Project A ownership of research state;
- protocol-based formalization rather than one-shot transfer;
- the need for explicit mathematical composition semantics;
- the importance of a downstream numerical lowering boundary.

The proposed unrestricted algebraic trait runtime is **not adopted** as a constitutional requirement. PhysMath remains bounded, typed, versioned, and deterministic.

The Numerical Layer remains outside this foundation.

## 1.3 Qwen — net-valid findings

Qwen identifies four concrete implementation traps:

1. in-memory Go composition versus serialized references;
2. illegal runtime interpretation of an `ACCEPTED` formalization state;
3. the missing mathematical-condition representation;
4. ambiguity caused by optional shared interoperability infrastructure.

These are adopted with explicit decisions below.

## 1.4 Z — net-valid findings

Z identifies the strongest connective-tissue corrections:

- remove PhysMath → Phys dependency;
- establish an explicit mathematical/Phys projection subset;
- use reusable interoperability infrastructure without turning it into an authority;
- use authority-local replay;
- define a future trust/curation trajectory without pretending it exists now;
- establish one ecosystem-wide identity/canonicalization vocabulary;
- coordinate cross-project capability growth.

Z's central finding is adopted:

> **The partition is right; the joints are wrong.**

The exact proposal to make a separate `PhysTrust` project is not required for this foundation because frozen Phys remains the current trust authority.

---

# 2. Constitutional separation

## 2.1 PhysMath

PhysMath is **pure mathematics**.

It asks:

> What mathematical structure is this, and what exact mathematical operations are valid on it?

PhysMath owns:

- mathematical objects;
- mathematical sorts;
- exact symbolic transformations;
- mathematical conditions;
- mathematical functions/functionals;
- mathematical operators;
- tensor/index structures;
- mathematical geometry;
- formal limits/asymptotics;
- mathematical equivalence;
- mathematical execution traces.

PhysMath MUST NOT own:

- physical ontology;
- physical interpretation;
- physical observables;
- physical assumptions as epistemic metadata;
- physical conventions as authoritative metadata;
- physical provenance;
- corpus status;
- MRC;
- trust;
- numerical execution.

## 2.2 phys-* external libraries

A `phys-*` library is a reusable physical-domain implementation.

It asks:

> What physical meaning does this domain assign to these mathematical structures?

It owns:

- physical objects;
- domain contracts;
- physical relationships;
- interpretation scope;
- framework-specific semantics;
- required mathematical references.

It MUST NOT own:

- a competing mathematics engine;
- physical provenance authority;
- corpus status authority;
- MRC/trust authority;
- duplicate assumption/convention systems.

## 2.3 Phys

Phys is the trusted physical formalization authority.

It asks:

> What physical formalization is supported by the trusted substrate, under what canonical physical metadata and derivation rules?

Phys owns:

- supported trusted physical objects;
- physical assumptions;
- physical conventions;
- physical provenance;
- physical corpus status;
- MRC/trust;
- trusted derivation/session/ledger machinery.

Phys MUST NOT become:

- a general mathematics library;
- a general CAS;
- a general theory engine;
- a numerical runtime;
- a dumping ground for every future physical domain.

## 2.4 TOE-Agent

TOE-Agent owns:

- hypotheses;
- hypothesis chains;
- research state;
- correspondence proposals;
- contradiction/counterexample records;
- formalization proposals;
- references to mathematical and physical artifacts.

TOE-Agent does NOT own the mathematical or physical semantics it references.

## 2.5 Numerical Layer

The Numerical Layer owns:

- numerical evaluation;
- simulation;
- numerical solvers;
- optimization;
- fitting;
- Monte Carlo;
- quantitative prediction;
- empirical comparison.

It does not infer missing physical semantics.

---

# 3. The dependency correction

The following is now a hard architectural invariant:

```text
PhysMath has ZERO dependency on Phys.
Phys has ZERO dependency on PhysMath.
```

This is stronger than merely prohibiting semantic leakage.

The pure PhysMath mathematical implementation literally cannot import Phys.

The relationship that requires both is implemented by `phys-*` packages and Project A.

```text
PhysMath
   ▲
   │ math dependency
   │
phys-* 
   │
   │ trust/formalization dependency
   ▼
 Phys
```

Project A composes them externally.

This removes the prior dependency-inversion problem identified across the review cycle. The three-project separation itself remains intact. fileciteturn305file0L15-L25

---

# 4. In-memory composition versus serialized references

This distinction is now explicit.

## 4.1 In-memory

In-memory composition uses ordinary Go types.

A `phys-*` package may import:

```text
PhysMath
Phys
```

and compose concrete types directly.

Example shape:

```go
type PhysicalObject struct {
    Math      physmath.Object
    Contract  PhysicalContract
}
```

The exact Go names are implementation choices.

The semantic rule is not.

An in-memory object MUST NOT require an opaque artifact-ID lookup merely to access another Go object.

## 4.2 Serialized/research graph

Artifact references are used for:

- JSON serialization;
- persistence;
- research-graph storage;
- cross-process communication;
- cross-repository artifact linkage.

They are not a replacement for normal in-memory Go composition.

Therefore:

```text
Go runtime composition
    -> typed Go values

serialized artifact graph
    -> ArtifactRef
```

Qwen's distinction is adopted because otherwise an artifact-reference model can accidentally make Phys behave like a database resolver. fileciteturn305file2L12-L20

---

# 5. Artifact identity

The ecosystem uses one serialized artifact-identity scheme.

The reference is:

```text
ArtifactRef {
    Namespace
    ArtifactType
    SchemaVersion
    Digest
}
```

The digest is:

```text
SHA-256(
    Namespace
    || canonical SchemaVersion
    || canonical ArtifactType
    || canonical Payload
)
```

Rules:

1. canonical bytes are deterministic;
2. schema version participates in identity;
3. artifact type participates in identity;
4. namespace participates in identity;
5. pretty-printing never participates in identity;
6. map iteration order never participates in identity;
7. floating-point serialization is forbidden for exact mathematical artifacts;
8. incompatible schema versions MUST be rejected by consumers unless an explicit compatibility rule exists.

This closes the previously open “artifact_id versus content_digest” ambiguity.

---

# 6. Namespace and registry ownership

There is **no mandatory universal runtime registry**.

Each authority owns its namespace:

```text
physmath.*
phys.*
phys-qm.*
phys-bohm.*
phys-gr.*
phys-cst.*
phys-cdt.*
toe.*
```

A registry/publication system may exist outside the mathematical/physical cores.

Its job is:

```text
store
discover
resolve
publish
```

not:

```text
interpret
trust
adjudicate
```

Artifact resolution is therefore infrastructure, not semantics.

A consumer that cannot resolve an artifact reference MUST fail explicitly.

---

# 7. Canonicalization

Every artifact family has a normative canonical representation.

Canonicalization is defined per artifact family, but follows ecosystem-wide rules:

- schema version explicit;
- type explicit;
- deterministic field ordering;
- deterministic collection ordering;
- stable numeric representation;
- explicit scope/binding identities;
- no presentation formatting;
- no hidden environment state;
- no timestamps unless explicitly part of the artifact's semantics.

Canonicalization is distinct from mathematical equivalence.

---

# 8. PhysMath mathematical IR

PhysMath MUST be multi-sorted.

The exact final core sort set is implementation-pinned before coding, but the semantic categories required by the foundation include:

```text
Expression
Function
FunctionApplication
Functional
FunctionalApplication
Operator
OperatorApplication
Tensor
Index
Map
Relation
Set / Collection
Transformation
Limit
Asymptotic / Truncation
DerivationObligation
```

A sort is not defined merely by its name.

Every sort MUST define:

```text
domain
codomain
construction
application
composition
canonicalization
equality
serialization
hashing
resource limits
```

A coding agent MUST NOT invent composition semantics from names.

This directly addresses the reviewer finding that a vocabulary of sorts is not yet a type system. fileciteturn305file0L55-L59

---

# 9. Mathematical extension protocol

PhysMath's core mathematical IR is closed and versioned.

An extension may add a mathematical sort only through an explicit extension protocol containing:

```text
SortID
SchemaVersion
CanonicalForm
EqualityRule
SerializationRule
HashRule
CompositionRules
TransformationRules
ResourceBounds
```

Runtime self-invention of arbitrary new sorts is forbidden.

This provides extensibility without making canonicalization/replay impossible.

---

# 10. Mathematical conditions

PhysMath explicitly owns mathematical conditions.

A condition is a purely mathematical predicate.

Examples:

```text
x != 0
x > 0
det(g) != 0
r > 2M
domain(f)
boundary_condition
```

Conceptually:

```go
type MathCondition struct {
    Predicate MathExpression
}
```

No physical metadata is present.

No provenance is present.

No trust is present.

No corpus status is present.

No human curation is present.

This resolves the “mathematical precondition vacuum” identified in the review. fileciteturn305file2L30-L37

---

# 11. Mathematical condition versus physical assumption

The same mathematical expression may later acquire physical significance.

For example:

```text
MathCondition:
    r > 2M
```

may later be interpreted as:

```text
physical meaning:
    exterior-region condition
```

and a formalized Phys artifact may attach:

```text
physical assumption / convention / regime metadata
```

The three are separate:

```text
mathematical predicate
    ≠
physical interpretation
    ≠
trust metadata
```

No translation of the mathematical expression itself is required merely to preserve its mathematics.

---

# 12. Physical interpretation artifact

`PhysicalInterpretation` is a **phys-* owned physical-domain artifact**.

It establishes:

```text
this mathematical artifact
is interpreted as
this physical concept
under
this framework/scope
```

Conceptually:

```text
PhysicalInterpretation {
    MathRef
    FrameworkRef
    PhysicalType
    Scope
    RegimeRefs
    RequiredConventionRefs
}
```

It does not contain Phys trust status.

It does not contain corpus status.

It does not redefine the mathematical object.

---

# 13. Research interpretation proposal

Before a physical interpretation is an established reusable `phys-*` contract, Project A may hold a:

```text
ProposedInterpretation
```

as research state.

This is deliberately distinct from `PhysicalInterpretation`.

Therefore:

```text
ProposedInterpretation
    = Project A research claim

PhysicalInterpretation
    = phys-* physical contract
```

This eliminates the overlap between interpretation and correspondence artifacts.

---

# 14. Correspondence proposal

`CorrespondenceProposal` belongs to Project A.

It means:

> A research agent proposes a relationship between already identifiable mathematical/physical artifacts.

It does NOT mean:

> these two artifacts are physically identical.

A correspondence can therefore state:

```text
A maps to B
A approximates B
A may reproduce B in regime R
A and B are conjectured to be related
```

without creating identity.

This also preserves the TOE's freedom to investigate multiple competing relationships.

---

# 15. Multi-interpretation rule

One mathematical artifact may have multiple physical interpretations.

Example:

```text
MathWaveFunction
    │
    ├── phys-qm interpretation
    └── phys-bohm interpretation
```

Each interpretation is separately namespaced and scoped.

No interpretation overwrites the mathematics.

No interpretation automatically supersedes another.

Conflicting interpretations are not a PhysMath error.

They become a research/physical-contract issue governed by their declared frameworks.

---

# 16. Interpretation ambiguity rule

Every physical object MUST explicitly declare:

```text
INTERPRETATION_NEUTRAL
```

or:

```text
FRAMEWORK_SPECIFIC
```

A framework-specific object MUST NOT masquerade as interpretation-neutral.

The existence of multiple interpretations is not an architectural defect.

The architectural defect is **silent interpretation**.

---

# 17. Physical-object extension contract

An external `phys-*` package MUST define:

```text
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

The package may own domain-specific physical semantics.

It may not own the canonical trust machinery.

---

# 18. Physical versus mathematical object

The classification is semantic:

| Concept | Owner |
|---|---|
| Hilbert space | PhysMath |
| Vector space | PhysMath |
| Function | PhysMath |
| Operator | PhysMath |
| Wavefunction as mathematical function | PhysMath |
| Quantum state as physical object | phys-qm |
| Bohmian wavefunction as physical object | phys-bohm |
| Bohmian configuration | phys-bohm |
| Physical spacetime | appropriate phys-* |
| Physical metric | appropriate phys-* |
| Physical assumption | Phys |
| Physical provenance | Phys |
| Corpus status | Phys |

The word used in the type name never determines ownership.

---

# 19. Trust metadata ownership

There is one canonical physical trust/metadata model.

Phys owns:

```text
Assumption
Convention
Provenance
CorpusStatus
MRC
TrustedSession/Ledger
```

PhysMath does not replicate these.

phys-* does not replicate these.

TOE-Agent does not replicate these.

A `phys-*` package may declare required physical metadata, but the actual authoritative metadata remains in Phys.

---

# 20. Frozen Phys constraint and external-domain trust

The frozen Phys MVP does not provide a general runtime curation mechanism for every future `phys-*` domain.

Therefore this rule is explicit:

> **External-domain physical objects remain external and do not automatically become trusted Phys corpus objects.**

A `phys-*` package MUST NOT invent a curation system.

A future curation/admission mechanism is a future Phys specification/Growth Gate concern.

For the current architecture, external physical-domain definitions can be:

```text
reusable
formalized
tested
referenced
```

without pretending that they are already trusted corpus facts.

The previously strongest version of this rule is restored as a hard containment invariant.

---

# 21. TrustAssertion is a future design target

`TrustAssertion` is a useful architectural concept, but the current frozen Phys MVP does not provide a standalone runtime trust-assertion object.

Therefore:

```text
TrustAssertion
    = FUTURE DESIGN TARGET
```

not:

```text
TrustAssertion
    = existing Phys runtime API
```

No coding agent may implement a new `AcceptTrust` or `AcceptFormalization` path in Phys to satisfy this architecture.

Any future standalone trust-artifact mechanism requires a separate Phys specification/Growth Gate decision.

This directly closes the review finding that the earlier document accidentally implied an existing Phys artifact that does not exist. fileciteturn305file3L39-L45

---

# 22. External physical objects and current Phys trust

Because current Phys trust can only be created through its existing supported object/session mechanisms:

```text
external phys-* object
       |
       +-- may be used in research
       |
       +-- may participate in correspondence
       |
       +-- may supply projectable scalar/quantity fragments
       |
       +-- may produce hypothesis-grade Phys artifacts
       |
       `-- does not become trusted corpus automatically
```

This is intentionally conservative.

---

# 23. PhysMath → Phys projection

The PhysMath/Phys boundary has one explicit mathematical projection subset.

`PhysMath.Expression` MUST provide a semantic mirror for the frozen Phys quantity-expression subset used for interoperability.

The mirror subset is:

```text
Symbol
Rational
Add
Mul
Neg
Pow
Sqrt
```

with the exponent-domain rules matching the frozen Phys `Pow` contract exactly.

The meaning of “mirror” is:

```text
same mathematical structure
same canonical semantics
same exactness expectations
deterministic projection
```

It does NOT mean the entire PhysMath IR is restricted to this subset.

All richer mathematical sorts remain outside this projection unless an explicit future projection contract is added.

Examples of normally non-projectable structures include:

```text
Function
Functional
Operator
Tensor
Manifold
Connection
Curvature
General transformation
```

They remain valid PhysMath artifacts.

Their inability to fit the frozen Phys substrate is an explicit representation boundary, not a mathematical failure.

---

# 24. Projection classification

Every proposed projection is classified:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

Classification is based on the explicit projection contract.

It is never inferred from object names.

It is never silently widened.

---

# 25. Projection loss report

A partial projection MUST report:

```text
source_ref
projected_ref
classification
represented_fields
omitted_fields
omitted_conditions
omitted_approximations
unresolved_obligations
reason
```

An unrepresentable object MUST remain unmodified in its source authority.

No information is silently discarded.

---

# 26. Formalization protocol

Formalization is a **protocol**, not a one-shot object transfer.

The protocol is implemented by Project A integration tooling using:

- PhysMath public artifacts;
- `phys-*` public contracts;
- frozen Phys public APIs.

No Phys kernel modification is implied.

The protocol is:

```text
FormalizationProposal
        |
        v
Projectability review
        |
  +-----+---------+
  |               |
  v               v
RequestRevision  Reject
  |
  v
revised proposal
  |
  v
projectable subset
  |
  v
existing Phys public APIs
  |
  v
hypothesis-grade Phys artifact/session result
```

---

# 27. No runtime ACCEPTED corpus state

There is **no runtime `AcceptFormalization()` API** in the current architecture.

A successful formalization attempt means:

```text
the projectable fragment was successfully formalized
```

It does NOT mean:

```text
corpus status became ESTABLISHED
```

External-domain additions do not acquire trusted corpus status at runtime.

Human curation remains outside this runtime process and follows the applicable Phys specification.

Qwen's warning about the frozen Phys MVP is adopted as a hard implementation constraint. fileciteturn305file2L21-L28

---

# 28. Formalization outcomes

The protocol uses:

```text
REQUEST_REVISION
REJECTED
PROJECTABLE
PARTIALLY_PROJECTABLE
UNREPRESENTABLE
PROJECTED_AS_HYPOTHESIS
```

These are **protocol/research outcomes**, not Phys corpus statuses.

`PROJECTED_AS_HYPOTHESIS` means only that the Phys public machinery successfully represented the selected fragment with its existing hypothesis-grade constraints.

---

# 29. Replay architecture

Replay is authority-local.

Examples:

```text
PhysMath MathTrace
TOE-Agent ResearchTrace
phys-gr domain trace
phys-bohm domain trace
Phys Session Ledger
```

There is no fixed global count.

Each trace declares:

```text
producer
schema_version
operation_vocabulary
input identity model
output identity model
replay authority
```

A trace may reference artifacts from another authority.

It does not thereby acquire authority to replay those artifacts.

This corrects the earlier artificial “two replay domains” model. fileciteturn305file3L49-L55

---

# 30. Mathematical trace

PhysMath traces are research/computational records.

They may contain:

```text
MathObjectRefs
OperationIDs
Parameters
IntermediateRefs
ResultRefs
Conditions
Failures
Counterexamples
Obligations
```

They MUST NOT contain:

```text
Phys provenance
Phys corpus status
Phys trust decisions
Empirical confirmation
```

---

# 31. Research trace

TOE-Agent research traces may contain:

```text
hypothesis transitions
candidate interpretations
correspondence proposals
attack results
revision history
formalization proposals
artifact references
```

They are not mathematical traces and not Phys ledgers.

---

# 32. Artifact-reference versus in-memory rule

This distinction is mandatory:

```text
ArtifactRef:
    serialization/interchange identity

Go object:
    in-memory composition
```

A coding agent MUST NOT create a universal:

```go
type Artifact interface{}
```

merely to unify all projects.

Typed domain structs remain the primary in-memory representation.

Serialized artifacts use `ArtifactRef`.

---

# 33. phys-* family conformance

The physical-extension family shares a common conformance specification.

Every `phys-*` package MUST prove:

```text
no import of Phys internals
no duplicate trust metadata
no duplicate provenance
no duplicate corpus status
explicit interpretation scope
deterministic serialized identity
valid MathRefs
valid PhysicalContract
no silent semantic widening
```

This conformance requirement is mandatory.

---

# 34. phys-interop MVP decision

A separate `phys-interop` runtime package is **not required in the initial foundation implementation**.

For MVP:

```text
in-memory composition
    -> direct typed imports

serialization
    -> standard encoding/json + package-owned schemas

conformance
    -> mandatory tests per phys-* package
```

A shared `phys-interop` module may be introduced later, but only as mechanical infrastructure.

It MUST NOT own:

```text
math semantics
physics semantics
trust
interpretation policy
research state
```

The capability is therefore mandatory; the specific shared module is deferred.

This combines the valid part of Z's shared-kit proposal with Qwen's warning about optional architecture. fileciteturn305file3L39-L45 fileciteturn305file2L39-L46

---

# 35. Status vocabulary

The system has separate vocabularies.

## 35.1 PhysMath mathematical outcomes

```text
SUCCESS
UNDEFINED
TRANSFORMATION_FAILED
OUT_OF_SCOPE
RESOURCE_LIMIT
```

## 35.2 TOE-Agent research states

```text
PROPOSED
UNDER_ATTACK
REVISION_REQUIRED
COUNTEREXAMPLE_FOUND
INCONSISTENT
AWAITING_FORMALIZATION
FORMALIZED
CLOSED
```

## 35.3 Formalization outcomes

```text
REQUEST_REVISION
REJECTED
PROJECTABLE
PARTIALLY_PROJECTABLE
UNREPRESENTABLE
PROJECTED_AS_HYPOTHESIS
```

## 35.4 Phys corpus/trust status

Defined only by the frozen Phys specification.

No other vocabulary reuses Phys corpus values to mean something else.

---

# 36. Version compatibility

A consumer MUST:

1. accept the exact supported schema version;
2. use an explicitly declared backward/forward compatibility rule if one exists;
3. reject incompatible versions;
4. never silently reinterpret an unknown field or schema version as an old schema.

This closes the version-compatibility residual identified in the review. fileciteturn305file3L49-L55

---

# 37. Naming and namespace discipline

No cross-project name may silently shadow a semantically distinct operation or artifact.

For example, a package MUST NOT rely on:

```text
Diff
Limit
Subst
```

being understood as equivalent to Phys's:

```text
Differentiate
Limit
Substitute
```

without an explicit mapping.

Names are presentation.

Stable IDs and contracts carry semantics.

---

# 38. Time versus RG scale

The foundation MUST preserve the distinction:

```text
t = physical time parameter
k = renormalization/coarse-graining parameter
```

PhysMath models both as mathematical parameters.

Physical roles are supplied by physical contracts.

The architecture must support:

```text
∂/∂t
```

and:

```text
∂/∂k
```

as different operations.

This requirement was accidentally dropped in the previous architecture revision and is restored.

---

# 39. Numerical contract

A future computational contract is produced downstream from formalized physics.

Conceptually:

```text
ComputationalContract {
    schema_version
    physical_model_refs
    mathematical_input_refs
    equations
    domain
    parameters
    numerical_method_requirements
    approximation_policy
    expected_outputs
}
```

The Numerical Layer consumes this.

It does not create physical meaning.

It does not replace missing assumptions.

The exact numerical IR and lowering mechanism are outside this foundation and remain a separate project concern.

---

# 40. Growth routing

When a capability is missing:

```text
Is it pure mathematics?
    |
    +-- YES --> PhysMath

Is it physical-domain semantics?
    |
    +-- YES --> phys-*

Is it a generic trusted physical primitive?
    |
    +-- YES --> Phys Growth Gate

Otherwise
    |
    +-- remain external
```

A TOE difficulty is insufficient by itself to justify kernel growth.

---

# 41. Capability coordination

A cross-project capability ledger is recommended for the broader TOE program.

It records:

```text
capability
requesting project
required project(s)
dependency order
blocking evidence
target release
```

It does not own semantics.

This is optional governance tooling rather than a runtime architecture requirement.

Z's capability-ledger insight is retained as lifecycle guidance rather than a new semantic subsystem. fileciteturn303file0L21-L23

---

# 42. TOE foundation boundary

The TOE is a large research program.

This architecture therefore specifies only what is necessary for the TOE to build upon the foundation.

It does not encode:

```text
Dancer
Dance
Stage
```

as PhysMath or Phys types.

The TOE-Agent may use those as research concepts and labels.

The libraries instead provide reusable primitives such as:

```text
PhysMath:
    Hilbert spaces
    functions
    operators
    tensors
    geometry
    discrete structures
    formal asymptotics
    transformations

phys-*:
    quantum physical contracts
    Bohmian physical contracts
    GR contracts
    CST/CDT contracts
```

The exact physical theory remains Project A's research problem.

---

# 43. TOE integration test philosophy

The TOE is a stress test of the foundation, not its specification.

Acceptance should test:

```text
multiple mathematical domains compose
multiple physical frameworks compose
one mathematical object may have multiple interpretations
cross-framework correspondences remain conjectural
rich mathematical objects remain valid even when Phys cannot project them
at least one meaningful physical fragment can enter existing Phys machinery
partial/unrepresentable cases fail honestly
t-flow and k-flow remain distinct
```

The foundation passes when it enables the research program without becoming the research program.

---

# 44. Adversarial tests required before implementation-plan handoff

## Dependency tests

```text
PhysMath core -> no Phys
PhysMath core -> no phys-*
Phys -> no PhysMath
phys-* -> PhysMath allowed
phys-* -> Phys allowed
TOE-Agent -> public surfaces only
```

## Identity tests

```text
same artifact bytes -> same digest
different schema version -> different identity
pretty-print change -> same identity
field-order change -> same identity
```

## Ownership tests

```text
HilbertSpace -> PhysMath
WaveFunction-as-math -> PhysMath
BohmianConfiguration -> phys-bohm
Assumption -> Phys
Provenance -> Phys
CorpusStatus -> Phys
```

## Condition tests

```text
MathCondition(x != 0) contains no Phys metadata
physical assumption cannot exist in PhysMath
```

## Interpretation tests

```text
one MathArtifact -> multiple scoped interpretations
framework-specific object cannot masquerade as neutral
CorrespondenceProposal != PhysicalInterpretation
```

## Projection tests

```text
supported expression -> FULLY_REPRESENTABLE
rich non-projectable structure -> UNREPRESENTABLE
partial projection -> mandatory loss report
no silent coercion
```

## Formalization tests

```text
REQUEST_REVISION -> no Phys corpus promotion
PROJECTABLE -> hypothesis-grade Phys result
no runtime AcceptFormalization
```

## Replay tests

```text
MathTrace replays in PhysMath
Phys Ledger replays in Phys
one authority cannot claim replay of another authority's operations
```

## Numerical boundary tests

```text
missing physical assumption -> computation rejected
raw research state -> not a numerical input contract
```

---

# 45. Supersession of previous architecture decisions

This document explicitly supersedes the following previous decisions:

```text
Previous:
    PhysMath -> Phys

Current:
    PhysMath ⊥ Phys
```

```text
Previous:
    universal HandoffPackage

Current:
    FormalizationProposal + FormalizationResponse protocol
```

```text
Previous:
    universal PhysicalArtifactEnvelope

Current:
    independent Artifact / Reference / Interpretation / Trust concepts
```

```text
Previous:
    fixed two replay domains

Current:
    authority-local replay domains
```

```text
Previous:
    standalone TrustAssertion implied as existing Phys runtime object

Current:
    TrustAssertion is future design target only
```

```text
Previous:
    optional phys-interop implementation

Current:
    conformance mandatory; shared runtime module deferred
```

No earlier “closed” decision may be interpreted as simultaneously active where this section explicitly supersedes it.

---

# 46. Final decision register

| ID | Architectural decision | Status |
|---|---|---|
| ADR-001 | PhysMath, Phys, phys-*, and TOE-Agent are separately owned projects. | CLOSED |
| ADR-002 | PhysMath has zero dependency on Phys. | CLOSED |
| ADR-003 | Phys has zero dependency on PhysMath. | CLOSED |
| ADR-004 | phys-* may depend on PhysMath and Phys. | CLOSED |
| ADR-005 | TOE-Agent owns research state and hypothesis chains. | CLOSED |
| ADR-006 | PhysMath owns mathematics only. | CLOSED |
| ADR-007 | phys-* owns physical-domain semantics. | CLOSED |
| ADR-008 | Phys owns physical trust/formalization metadata. | CLOSED |
| ADR-009 | Numerical Layer owns numerical computation. | CLOSED |
| ADR-010 | In-memory composition uses concrete Go types. | CLOSED |
| ADR-011 | Serialized cross-project references use ArtifactRef. | CLOSED |
| ADR-012 | Artifact identity uses deterministic SHA-256 canonical bytes. | CLOSED |
| ADR-013 | Namespaces are authority-owned; no universal runtime registry is required. | CLOSED |
| ADR-014 | Canonicalization is per artifact family under common deterministic rules. | CLOSED |
| ADR-015 | PhysMath uses a multi-sorted mathematical IR. | CLOSED |
| ADR-016 | Mathematical extension sorts require explicit versioned contracts. | CLOSED |
| ADR-017 | PhysMath owns MathCondition. | CLOSED |
| ADR-018 | Phys owns physical Assumption. | CLOSED |
| ADR-019 | PhysicalInterpretation is phys-* owned. | CLOSED |
| ADR-020 | ProposedInterpretation is Project A research state. | CLOSED |
| ADR-021 | CorrespondenceProposal is Project A research state. | CLOSED |
| ADR-022 | One mathematical artifact may have multiple physical interpretations. | CLOSED |
| ADR-023 | Interpretation scope must be explicit. | CLOSED |
| ADR-024 | External physical objects do not automatically become trusted Phys corpus objects. | CLOSED |
| ADR-025 | Standalone TrustAssertion is a future Phys design target, not MVP runtime API. | CLOSED |
| ADR-026 | Math/Phys projection subset is explicit and bounded. | CLOSED |
| ADR-027 | Rich non-projectable mathematics remains valid PhysMath. | CLOSED |
| ADR-028 | Projection loss is explicit and auditable. | CLOSED |
| ADR-029 | Formalization is a protocol, not a one-shot transfer artifact. | CLOSED |
| ADR-030 | No runtime AcceptFormalization/corpus-promotion API. | CLOSED |
| ADR-031 | Replay is authority-local. | CLOSED |
| ADR-032 | phys-* conformance rules are mandatory. | CLOSED |
| ADR-033 | No separate phys-interop runtime module is required for MVP. | CLOSED |
| ADR-034 | Shared interop infrastructure, if introduced later, is mechanical only. | CLOSED |
| ADR-035 | Status vocabularies are owner-specific and non-overlapping. | CLOSED |
| ADR-036 | Schema incompatibility must be rejected explicitly. | CLOSED |
| ADR-037 | t-flow and k-flow remain distinct. | CLOSED |
| ADR-038 | Numerical semantics are downstream and cannot repair missing physics. | CLOSED |
| ADR-039 | Growth Gate remains the only route to Phys kernel expansion. | CLOSED |
| ADR-040 | TOE physics is not encoded into the library foundations. | CLOSED |

---

# 47. Coding-agent no-invention rule

A coding agent implementing from this document MUST NOT invent architecture for:

```text
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

Implementation choices remain open only where they do not alter these semantics.

Examples:

```text
private helper names
private algorithms
file organization
test organization
serialization implementation details
performance optimization
```

---

# 48. Implementation-plan authorization

The architecture is implementation-plan ready when this document and the frozen Phys specification are supplied together.

The coding-agent plan must produce:

```text
PhysMath implementation plan
    +
phys-* extension implementation plan
    +
TOE-Agent integration plan
```

It must NOT produce:

```text
Phys kernel redesign
```

unless a separate Growth Gate decision exists.

The architecture is intended to let implementation work proceed without re-opening semantic ownership.

---

# 49. Final architectural invariant

The complete system is:

\[
\boxed{
\text{Research State}
+
\text{Mathematical Structure}
+
\text{Physical Interpretation}
+
\text{Physical Trust}
+
\text{Numerical Computation}
}
\]

with explicit ownership:

\[
\boxed{
\begin{aligned}
\text{TOE-Agent} &\rightarrow \text{research} \\
\text{PhysMath} &\rightarrow \text{mathematics} \\
\text{phys-*} &\rightarrow \text{physical semantics} \\
\text{Phys} &\rightarrow \text{trusted physical formalization} \\
\text{Numerical} &\rightarrow \text{computation}
\end{aligned}}
\]

and explicit dependency:

\[
\boxed{
\text{PhysMath} \not\rightarrow \text{Phys},
\qquad
\text{Phys} \not\rightarrow \text{PhysMath},
\qquad
\text{phys-*}\rightarrow(\text{PhysMath},\text{Phys})
}
\]

The foundation is not itself the TOE.

It is the reusable substrate that allows a large TOE research program to be built, extended, challenged, formalized, and eventually computed without collapsing mathematics, physics, trust, research state, or numerics into one system.

**Architecture status: CLOSED.**

**Architectural blockers: NONE.**

**Coding-agent architectural invention: FORBIDDEN.**

**Next artifact: project-specific implementation plan.**
