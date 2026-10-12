# PhysMath / Phys / Phys-* External Libraries
## Revised Architecture and Implementation Foundation

**Document ID:** PM-ARCH-FOUNDATION-001  
**Version:** 1.1  
**Status:** **Architectural baseline — decision-pinned for implementation planning, subject to independent adversarial review**  
**Purpose:** Define the solid, reusable software foundation on which the TOE research program can be built. This document intentionally focuses on the relationship among **PhysMath**, **Phys**, and **external `phys-*` libraries**. It does not attempt to design the TOE itself.

---

# 0. Executive architectural decision

The previous architecture treated PhysMath, Phys, and `phys-*` as a vertical stack and consequently accumulated special-purpose handoffs, metadata attachment rules, dependency exceptions, and replay distinctions.

The revised architecture changes the fundamental model:

> **PhysMath, Phys, and `phys-*` are separate authorities over different kinds of artifacts. Their relationship is established by explicit references and contracts, not by semantic inheritance.**

The three-project foundation is:

```text
PROJECT A
TOE-Agent / Research Program
    |
    | composes and references artifacts
    v
+-------------------+       +---------------------+
|     PhysMath      |       |     Phys / Trust    |
|  pure mathematics |       | physical formalism  |
|  exact symbolic   |       | + canonical trust   |
+---------+---------+       +----------+----------+
          ^                            ^
          | math references            | physical contracts
          |                            |
          +------------+---------------+
                       |
                 +-----+------+
                 |   phys-*   |
                 |  physical  |
                 |  domains   |
                 +------------+

Numerical Layer
    consumes formalized Phys / phys-* computational contracts
```

The crucial dependency rule is now:

```text
PhysMath  ────────────────X────────────► Phys
Phys      ────────────────X────────────► PhysMath

phys-*   ───────────────► PhysMath
phys-*   ───────────────► Phys

TOE-Agent ──────────────► public surfaces of all required projects

Numerical ─────────────► Phys + selected phys-* only
```

In words:

- **PhysMath does not depend on Phys.**
- **Phys does not depend on PhysMath.**
- **`phys-*` libraries may depend on both.**
- **The TOE-Agent composes them at the research level.**
- **The Numerical Layer consumes formalized physical contracts.**

This is the primary architectural correction.

---

# 1. Why the architecture is changing

The reviews converged on several deeper problems rather than isolated defects.

The old design repeatedly forced us to answer:

```text
How does PhysMath import Phys without becoming physical?

How does Phys hold physics that is richer than its frozen object model?

How does an external package attach metadata without becoming a second trust system?

How do we hand a rich mathematical artifact into a much smaller physical IR?

How many replay systems are there?

Where does a new physical interpretation live?

```

These are symptoms of using package hierarchy to represent relationships that are actually independent.

The revised design separates four concerns:

```text
MATHEMATICS
    what mathematical structure exists

PHYSICAL SEMANTICS
    what physical meaning a framework assigns to that mathematics

TRUST / FORMAL AUTHORITY
    what physical assertions have what provenance/status

RESEARCH STATE
    why an AI agent is combining those artifacts into a hypothesis
```

Each concern has one owner.

---

# 2. Architectural principles

## 2.1 Mathematics is not physics

A mathematical object does not become physical because:

- it has a familiar physics name;
- it appears in a physics paper;
- it is later referenced by a physical theory;
- it participates in a physical derivation.

PhysMath therefore remains mathematics-only.

## 2.2 Physical meaning is not mathematical identity

A mathematical object may have:

- no physical interpretation;
- one physical interpretation;
- multiple framework-specific interpretations.

Those interpretations do not change the mathematical object's identity.

## 2.3 Trust is not physical meaning

A physical package can define a physical object without making it a trusted corpus fact.

Conversely, a trust assertion can refer to a physical artifact without defining the physical meaning of that artifact.

Therefore:

```text
physical semantics != trust status
```

## 2.4 Research state is not either

An AI agent's hypothesis about an object does not redefine the object's mathematics, physical semantics, or trust status.

The hypothesis is a separate research artifact that references them.

## 2.5 References cross boundaries; implementations do not

Cross-project composition happens through:

- stable artifact identifiers;
- schema/version identifiers;
- explicit public contracts;
- explicit mathematical references;
- explicit physical-contract references;
- explicit trust references.

It does not happen by importing internal implementation types across semantic boundaries.

---

# 3. Ownership model

| Authority | Owns | Does not own |
|---|---|---|
| **PhysMath** | mathematical structures, exact symbolic transformations, mathematical conditions, mathematical traces | physical ontology, physical assumptions, physical provenance, corpus status, numerical execution |
| **Phys** | canonical physical formalization, physical assumptions, conventions, provenance, corpus status, MRC/trust, trusted physical ledger | general mathematics, every possible future physical domain, numerical execution |
| **phys-* ** | domain-specific physical objects, physical relationships, framework semantics, interpretation scope | mathematical engine, duplicate trust systems, universal truth |
| **TOE-Agent** | hypotheses, research state, correspondence proposals, attack strategy, orchestration, handoff proposals | mathematical engine, physical ontology, Phys trust authority |
| **Numerical Layer** | numerical evaluation and empirical/computational analysis | physical semantics, mathematical reinterpretation, trust adjudication |

This table is normative.

---

# 4. Project boundaries

## 4.1 PhysMath

PhysMath is a reusable mathematical library ecosystem.

Its central question is:

> **What mathematical structure is this, and what mathematically valid transformations can be performed on it?**

PhysMath may contain:

```text
Expression
Function
Functional
Operator
Tensor
Index
Map
Relation
Set
Manifold
Metric
Connection
Curvature
Limit
Asymptotic structure
Transformation
```

The exact set is determined by the PhysMath IR specification.

PhysMath MUST NOT define:

```text
QuantumState
BohmianConfiguration
PhysicalSpacetime
Observable
PhysicalRegime
CorpusStatus
PhysicalProvenance
```

unless those names are purely mathematical objects and are explicitly documented as such. In ambiguous cases, a neutral mathematical name is preferred.

---

## 4.2 Phys

Phys is the trusted physical foundation.

Its question is:

> **What physical object or formal physical assertion is being represented, under what assumptions, conventions, provenance, and trust status?**

Phys remains bounded.

It must not become the universal home for every physical theory.

The frozen Phys implementation remains authoritative for its existing capabilities.

Where future internal evolution is needed, the conceptual concerns should remain separable:

```text
physical formalization
    +
physical trust/metadata
```

without requiring Phys to absorb all mathematical functionality.

---

## 4.3 phys-*

A `phys-*` package is an external physical-domain library.

Its question is:

> **What physical meaning does this domain assign to these mathematical structures?**

Examples:

```text
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
```

The package may contain:

```text
QuantumState
BohmianConfiguration
BohmianWaveFunction
GRMetric
CausalSet
CDTGeometry
GuidanceLaw
FieldEquation
```

where the corresponding physical semantics are explicitly declared.

`phys-*` remains external.

Its existence does not imply:

```text
trusted corpus
empirical truth
Phys kernel membership
```

---

# 5. The key correction: PhysMath has no upward dependency on Phys

This decision supersedes the earlier architectural rule that PhysMath must depend on Phys.

## 5.1 PhysMath dependency rule

The PhysMath mathematical core MUST have no dependency on Phys or `phys-*`.

Conceptually:

```text
PhysMath
    |
    X
    |
 Phys / phys-*
```

This gives the strongest possible enforcement of:

> **PhysMath is pure mathematics.**

A coding agent cannot accidentally use `Phys.Assumption`, `Phys.Provenance`, `Phys.Dimension`, or a physical-domain constructor because the pure mathematical package cannot import them.

## 5.2 Interoperability is outside mathematical semantics

When PhysMath needs to interact with Phys artifacts, that interaction occurs in:

```text
interop / adapter / research integration code
```

which is not part of the mathematical semantics.

That code may hold opaque references such as:

```text
ArtifactRef
PhysArtifactRef
SchemaRef
```

but does not reinterpret the mathematical object.

---

# 6. Artifact model

The universal architectural primitive is a **referenceable artifact**, not an envelope.

Every durable artifact MUST have:

```text
namespace
artifact_id or content_digest
schema_version
artifact_type
canonical_payload
```

The exact identity mechanism may be content-addressed, provided the final implementation satisfies the deterministic identity requirements.

The important invariant is:

> **An artifact can be referenced without importing the code that produced it.**

---

# 7. Artifact relationships

The ecosystem therefore resembles a reference DAG:

```text
                     ResearchHypothesis
                       /      |       \
                      /       |        \
                     v        v         v
              MathArtifact  PhysicalArtifact  Correspondence
                   |                |
                   |                |
                   v                v
             PhysMath          phys-* contract
                                      |
                                      v
                                Phys metadata /
                                trusted assertion
```

These arrows mean “references.”

They do not mean inheritance.

---

# 8. No universal envelope

The earlier `PhysicalArtifactEnvelope` concept is replaced.

An envelope attempted to combine:

```text
identity
payload
math references
physical interpretation
trust references
```

into one universal structure.

That is unnecessary coupling.

Instead:

```text
Artifact
Reference
Interpretation
TrustAssertion
ResearchHypothesis
```

are separate artifact classes.

A physical artifact may reference a mathematical artifact.

A trust assertion may reference a physical artifact.

A research hypothesis may reference all of them.

Each retains independent ownership.

---

# 9. Mathematical artifact

A mathematical artifact is owned by PhysMath.

Conceptually:

```text
MathArtifact {
    artifact_identity
    math_schema_version
    sort
    canonical_payload
}
```

It may be referenced by:

```text
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
TOE-Agent
Numerical tooling
```

without becoming physical.

---

# 10. Physical artifact

A physical artifact is owned by a `phys-*` domain package or, where already supported, the Phys base.

Conceptually:

```text
PhysicalArtifact {
    artifact_identity
    extension_id
    physical_type
    math_refs[]
    physical_contract
}
```

Its physical semantics are owned by the extension.

Its trust status is not intrinsic to the object merely because the object exists.

---

# 11. Trust assertion

Trust is a separate Phys-controlled artifact.

Conceptually:

```text
TrustAssertion {
    subject_ref
    provenance
    corpus_status
    assumptions_ref
    conventions_ref
    authority
}
```

This means:

```text
physical artifact
       ^
       |
trust assertion
```

rather than embedding trust state inside every extension's object implementation.

Phys remains the authoritative owner.

---

# 12. Physical interpretation

A physical interpretation connects mathematics to physics.

Conceptually:

```text
PhysicalInterpretation {
    math_ref
    framework_ref
    physical_contract_ref
    scope
    regime
    interpretation_class
}
```

The relationship is explicit.

It does not modify the mathematical artifact.

For example:

```text
MathWaveFunction
      |
      +----> QM interpretation
      |
      +----> Bohmian interpretation
      |
      +----> another explicitly scoped interpretation
```

This resolves the wavefunction problem cleanly.

---

# 13. Wavefunction rule

The following rule is normative:

```text
Wavefunction as mathematics
    -> PhysMath

QuantumState as physical object
    -> phys-qm

BohmianWaveFunction as framework-specific physical object
    -> phys-bohm
```

The mathematical object is reusable.

The physical interpretations are not conflated.

No layer infers physical ontology from the word `WaveFunction`.

---

# 14. Mathematical conditions versus physical assumptions

A condition such as:

```text
r > 2M
```

is a mathematical artifact in PhysMath.

Its use as:

```text
Schwarzschild exterior-region condition
```

is a physical interpretation supplied by a GR contract.

Its standing as:

```text
ASSUMED
ESTABLISHED
CONTESTED
```

is Phys trust metadata.

Therefore a single mathematical artifact can participate in multiple physical contexts without duplication.

---

# 15. External physical-library contract

Every `phys-*` library MUST declare:

```text
extension_id
schema_version
public physical types
physical relationships
mathematical references
interpretation scope
regime requirements
required conventions
required Phys metadata
compatibility rules
```

The extension MUST NOT create a competing metadata system.

---

# 16. Interpretation classification

Every framework-specific physical type MUST be explicitly classified.

Allowed classes:

```text
INTERPRETATION_NEUTRAL
FRAMEWORK_SPECIFIC
```

No implicit third state.

If an object cannot be described without choosing an interpretation, it must be framework-specific.

This prevents:

```text
phys-bohm.BohmianConfiguration
```

from masquerading as a universal quantum concept.

---

# 17. External-library admission criteria

A new physical concept belongs in an external `phys-*` library when:

1. it asserts physical meaning;
2. the meaning is not already covered by the frozen Phys base;
3. it has an explicit mathematical representation;
4. its interpretation scope is declared;
5. its relevant assumptions/conventions can reference the Phys metadata model;
6. it is independently testable;
7. it is reusable beyond one private derivation where appropriate;
8. it does not require modifying Phys merely for convenience.

---

# 18. Phys Growth Gate

The default route is:

```text
new capability
     |
     +---- mathematical? --------> PhysMath
     |
     +---- domain physics? ------> phys-*
     |
     +---- foundational/trust? ---> Phys Growth Gate
```

A request for Phys kernel growth requires independent evidence that the capability is:

- foundational;
- theory-neutral within scope;
- reusable across domains;
- not properly housed in PhysMath or `phys-*`;
- representable without corrupting existing Phys invariants;
- required by more than one isolated workload.

The TOE is not by itself sufficient evidence for kernel growth.

---

# 19. Composability

The architecture is designed for composition.

A theory may combine:

```text
PhysMath mathematics A
+
PhysMath mathematics B
+
phys-qm
+
phys-bohm
+
phys-gr
+
Phys trust metadata
```

without any one package becoming the owner of all of them.

Composition occurs through references and contracts.

---

# 20. Extensibility

New mathematics:

```text
add PhysMath capability
```

without requiring a physics-package rewrite.

New physical theory:

```text
add phys-* package
```

without requiring PhysMath changes.

New research program:

```text
add Project A research artifacts
```

without requiring either foundation to know the research program exists.

New trusted-corpus capability:

```text
use Phys Growth Gate
```

without turning external libraries into Phys core.

---

# 21. Reusability

A reusable mathematical object should be usable by unrelated physical theories.

For example:

```text
HilbertSpace
Operator
Tensor
Manifold
Functional
DifferentialOperator
```

must not be owned by one physical interpretation.

Likewise, a physical contract should be reusable across research programs without embedding one hypothesis chain.

A TOE-specific hypothesis belongs in Project A, not in PhysMath.

---

# 22. PhysMath IR

PhysMath MUST use a multi-sorted mathematical IR.

At minimum, the specification must distinguish the behavioral categories required by theoretical mathematics:

```text
Expression
Function
Functional
FunctionApplication
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
Derivation Obligation
```

The final core sort set is closed and versioned.

The purpose is not to enumerate every branch of mathematics.

The purpose is to make the mathematical structures needed by the supported workloads explicit and composable.

---

# 23. Composition is specified by behavior

A sort name alone is not a type system.

Every supported mathematical sort MUST define:

```text
domain
codomain
construction
application rules
composition rules
canonical form
equality semantics
serialization
hashing
resource limits
```

For example:

```text
f : A -> B
x : A

f(x) : B
```

must be a mechanically defined valid composition.

Likewise, invalid combinations must fail explicitly.

---

# 24. Symbol identity and scope

Display names never determine mathematical identity.

The architecture MUST explicitly model:

- scope;
- binding;
- local variables;
- bound variables;
- dummy variables;
- index variables;
- function arguments;
- generated identities;
- namespace ownership.

Canonical identity must survive renaming and formatting changes where the mathematical semantics require equivalence.

---

# 25. Exactness

PhysMath is exact.

Allowed:

```text
integer
rational
symbolic
exact finite structures
formal limits
formal asymptotics
exact algebraic transformation
```

Not allowed:

```text
floating-point execution
numerical integration
numerical solving
simulation
Monte Carlo
optimization
fitting
empirical analysis
```

This is theoretical mathematical physics, not numerical physics.

---

# 26. Asymptotics

PhysMath may represent:

```text
lim_{ε -> 0} F(ε)

F(ε) = a0 + a1 ε + O(ε^2)
```

as exact mathematics.

The physical assertion that a particular parameter corresponds to a particular physical regime belongs to the relevant `phys-*` contract.

This preserves the mathematical/physical boundary.

---

# 27. Equality model

PhysMath MUST distinguish:

```text
structural identity
definitional equality
mathematical equivalence
approximate/asymptotic equivalence
physical identification
```

These are separate relations.

A physical identification may reference mathematical equivalence, but cannot be inferred automatically from it.

---

# 28. Replay model

Replay is authority-local.

A project may maintain a trace if it owns executable transformations.

Examples:

```text
PhysMath MathTrace
Phys trusted Ledger
phys-gr domain trace
TOE-Agent research trace
```

There is no fixed number of replay domains.

Each trace declares:

```text
producer
schema version
operation vocabulary
input/output identity model
replay authority
```

Cross-project references identify source artifacts.

They do not claim cross-project replayability.

---

# 29. Research state versus mathematical trace

The TOE-Agent owns the hypothesis chain.

PhysMath owns mathematical transformation traces.

Therefore:

```text
ResearchHypothesis
    references
        MathTrace
        PhysicalArtifact
        CorrespondenceProposal
        TrustAssertion
```

rather than embedding the mathematical trace inside the hypothesis as if the research project owned the mathematics.

---

# 30. Research correspondence

The TOE-Agent may propose:

```text
CorrespondenceProposal
```

between artifacts owned by different mathematical or physical frameworks.

A correspondence proposal specifies:

```text
source refs
target refs
mapping
scope
regime
direction
exactness
obligations
status
```

It does not create physical identity.

This is the correct place for cross-framework research claims.

---

# 31. Formalization is a protocol, not a universal translation object

The earlier one-shot `HandoffPackage` model is replaced conceptually by:

```text
FormalizationProposal
        |
        v
Phys review / validation
        |
   +----+---------+
   |              |
Accept        RequestRevision
   |              |
   |              +-------> revised proposal
   |
   v
AcceptedPhysicalArtifact
```

The proposal is a machine-readable artifact.

The review cycle is the protocol.

---

# 32. Formalization outcomes

The protocol must support:

```text
ACCEPTED
REJECTED
REQUEST_REVISION
PARTIALLY_ACCEPTED
UNREPRESENTABLE
```

These are formalization outcomes.

They are not corpus-status values.

They do not replace Phys's trust metadata.

---

# 33. No silent loss

When a rich PhysMath artifact cannot fit the available Phys formal substrate:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

must be explicit.

A partial mapping must identify what is not represented.

An unrepresentable artifact remains valid research material.

The inability of frozen Phys to represent it is not a defect in PhysMath.

---

# 34. Numerical boundary

The Numerical Layer consumes:

```text
formalized physical model
+
physical computational contract
+
explicit numerical parameters
```

It does not become another semantics engine.

It must not:

- infer physics;
- invent assumptions;
- repair malformed contracts;
- reinterpret mathematical notation;
- convert an unformalized hypothesis into a physical model.

Numerical execution is downstream.

---

# 35. Shared interoperability infrastructure

A shared `phys-interop` package MAY be introduced as a **non-semantic infrastructure module**.

If used, it may provide:

```text
artifact references
schema utilities
canonical-reference helpers
extension conformance tests
interop codecs
import-boundary checks
```

It MUST NOT provide:

```text
physics ontology
mathematics
trust authority
corpus authority
research state
interpretation policy
```

This shared kit exists to avoid five independent implementations of the same mechanical boundary rules.

It is infrastructure, not a new conceptual layer.

---

# 36. What phys-interop does NOT do

It does not turn:

```text
MathArtifact
```

into:

```text
PhysicalTruth
```

It does not choose:

```text
QM interpretation
Bohmian interpretation
GR interpretation
```

It does not attach corpus status by itself.

It does not override Phys.

It does not become a hidden fifth authority.

---

# 37. TOE foundation contract

This architecture deliberately says very little about the physics of the TOE.

It only guarantees that the TOE research program can build upon reusable foundations.

A TOE workload may therefore reference:

```text
PhysMath:
    Hilbert spaces
    functionals
    operators
    geometry
    discrete structures
    differential operators
    asymptotics

phys-*:
    quantum physical contracts
    Bohmian physical contracts
    GR physical contracts
    CST/CDT physical contracts

Phys:
    physical assumptions
    conventions
    provenance
    trust
    formal derivation
```

The Dancer/Dance/Stage model remains entirely outside the PhysMath ontology.

The TOE research program decides its actual physical content.

---

# 38. TOE interfaces remain research problems

The foundation must permit Project A to investigate relationships such as:

```text
Dancer ↔ Stage
Dance  ↔ Stage
Dancer ↔ Dance
```

without requiring PhysMath to know those names.

The software foundation only needs to guarantee:

```text
mathematical structure
+
physical interpretation
+
explicit correspondence
+
explicit assumptions
+
formalization pathway
```

The actual TOE coupling law remains a research question.

---

# 39. AI-agent workflow

A research agent can therefore work like this:

```text
1. Select mathematical structures
        ↓
2. Compose them in PhysMath
        ↓
3. Derive and attack consequences
        ↓
4. Select physical contracts from phys-*
        ↓
5. Propose physical correspondences
        ↓
6. Record failures / counterexamples
        ↓
7. Produce FormalizationProposal
        ↓
8. Phys validates/projectizes the physical subset
        ↓
9. Numerical Layer evaluates computational consequences
```

The agent never needs to modify PhysMath merely because its hypothesis belongs to a new physical domain.

---

# 40. Status vocabularies

The ecosystem deliberately separates:

## Mathematical status

Owned by PhysMath operations/research traces.

Examples:

```text
VALID_TRANSFORMATION
TRANSFORMATION_FAILED
UNDEFINED
OUT_OF_SCOPE
RESOURCE_LIMIT
```

## Research status

Owned by TOE-Agent.

Examples:

```text
PROPOSED
UNDER_ATTACK
COUNTEREXAMPLE_FOUND
INCONSISTENT
AWAITING_FORMALIZATION
FORMALIZED
```

## Formalization outcome

Owned by the Phys formalization protocol.

Examples:

```text
ACCEPTED
REJECTED
REQUEST_REVISION
PARTIALLY_ACCEPTED
UNREPRESENTABLE
```

## Corpus/trust status

Owned by Phys.

Examples are defined by the frozen Phys contract.

No vocabulary is reused to mean two different things.

---

# 41. Security and containment

All AI-generated structures are untrusted input.

The implementation must bound:

```text
expression depth
object count
tensor size
recursion
operation count
trace size
serialization size
memory
```

Failures must not corrupt:

```text
PhysMath artifacts
research traces
Phys metadata
Phys ledgers
accepted physical artifacts
```

---

# 42. No-go architectures

The following are rejected:

```text
PhysMath -> Phys dependency
Phys -> PhysMath dependency
PhysMath physical ontology
phys-* duplicate trust metadata
universal artifact envelope
one-shot handoff as the only boundary mechanism
Dancer/Dance/Stage in PhysMath core
automatic physical interpretation from notation
automatic physical identity from mathematical equivalence
automatic hypothesis promotion
numerical execution in PhysMath
TOE-specific code in PhysMath core
Phys kernel growth for domain convenience
single monolithic TOE application
```

---

# 43. Implementation structure

The implementation should be developed as distinct repositories/modules.

Conceptually:

```text
github.com/PithomLabs/physmath
github.com/PithomLabs/phys
github.com/PithomLabs/phys-interop       optional shared infrastructure
github.com/PithomLabs/phys-qm
github.com/PithomLabs/phys-bohm
github.com/PithomLabs/phys-gr
github.com/PithomLabs/phys-cst
github.com/PithomLabs/phys-cdt
github.com/PithomLabs/toe-agent           research program
```

These are architectural examples.

Exact repository names are implementation decisions unless separately locked.

The dependency rules are not optional.

---

# 44. Implementation order

The recommended order is:

```text
1. PhysMath mathematical core
2. PhysMath canonicalization / identity
3. PhysMath mathematical traces
4. shared artifact/reference protocol
5. Phys external-library contract
6. first phys-* library
7. Phys formalization integration
8. TOE-Agent research artifacts
9. additional phys-* packages
10. Numerical consumer contracts
```

This order deliberately establishes reusable foundations before attempting the full TOE workload.

---

# 45. Acceptance strategy

The architecture should be tested in three independent dimensions.

## A. PhysMath acceptance

Can it support reusable theoretical mathematics?

## B. phys-* acceptance

Can independent physical domains reuse the same mathematical foundation and Phys trust model without semantic collisions?

## C. TOE-Agent acceptance

Can the research program compose those pieces into large multi-framework hypotheses without any foundation becoming TOE-specific?

The TOE is therefore an **integration stress test**, not the specification of the libraries.

---

# 46. Architectural invariants for coding agents

A coding agent must be able to implement any project without answering these questions itself.

### Ownership

```text
Math -> PhysMath
Physics -> phys-* / Phys
Trust -> Phys
Research -> TOE-Agent
Numerics -> Numerical
```

### Dependencies

```text
PhysMath -X-> Phys
Phys -X-> PhysMath
phys-* -> PhysMath + Phys
TOE-Agent -> public surfaces
Numerical -> Phys + phys-*
```

### Metadata

```text
Assumption -> Phys
Convention -> Phys
Provenance -> Phys
CorpusStatus -> Phys
MRC -> Phys
Trust -> Phys
```

### Interpretation

```text
physical meaning -> phys-* / Phys
mathematical meaning -> PhysMath
```

### Research

```text
hypothesis -> TOE-Agent
```

### Computation

```text
numerical execution -> Numerical Layer
```

---

# 47. Final architectural model

The final foundation is:

```text
                       TOE-Agent
                    research / AI
                 /       |        \
                /        |         \
               v         v          v

        +-----------+  +-----------+  +-----------+
        | PhysMath  |  |  phys-*   |  |   Phys    |
        | Mathematics|  | Physical  |  | Trust +   |
        |            |  | Semantics |  | Formalize |
        +-----+------+  +-----+-----+  +-----+-----+
              \              |              /
               \             |             /
                +------------+------------+
                     explicit references

                           |
                           v
                    Numerical Layer
```

The semantic separation is:

\[
\boxed{
\text{Mathematics}
\neq
\text{Physical Meaning}
\neq
\text{Trust}
\neq
\text{Research State}
\neq
\text{Numerical Computation}
}
\]

The implementation relationship is:

\[
\boxed{
\text{PhysMath}
\;\;\bot\;\;
\text{Phys}
}
\]

for direct dependency, while:

\[
\boxed{
\text{phys-*}\rightarrow
(\text{PhysMath},\text{Phys})
}
\]

and:

\[
\boxed{
\text{TOE-Agent}\rightarrow
(\text{PhysMath},\text{phys-*},\text{Phys})
}
\]

This gives the TOE research program a foundation that is:

\[
\boxed{\textbf{composable + extensible + reusable}}
\]

without forcing the foundations to become the TOE.

---

# 48. Coding-agent completion criterion

An implementation plan derived from this document is conforming only when:

```text
[ ] PhysMath is physically agnostic.
[ ] PhysMath has no dependency on Phys.
[ ] Phys has no dependency on PhysMath.
[ ] phys-* can reuse both without cycles.
[ ] Mathematical artifacts have stable identities.
[ ] Physical artifacts reference mathematics explicitly.
[ ] Trust assertions reference artifacts rather than redefining them.
[ ] No package-local trust system exists in phys-*.
[ ] Mathematical conditions remain mathematical until physically interpreted.
[ ] Physical interpretation is explicit.
[ ] Mathematical equality cannot silently become physical identity.
[ ] Replay authority is local to each producing system.
[ ] Formalization is an iterative protocol, not a one-shot dump.
[ ] Partial/unrepresentable formalization is explicit.
[ ] TOE-specific ontology is absent from PhysMath.
[ ] Numerical computation is absent from PhysMath and Phys.
[ ] Shared interoperability infrastructure, if implemented, owns no semantics.
[ ] Phys Growth Gate remains the only route to kernel growth.
[ ] Project A owns research state and hypothesis composition.
[ ] No architectural decision is left for the coding agent to invent.
```

---

# 49. Final principle

The foundation should not attempt to make all of physics one program.

It should make **different kinds of knowledge composable without confusing their ownership**.

> **PhysMath provides the mathematical language of structure and transformation. `phys-*` packages provide explicit physical interpretations. Phys provides trusted physical formalization and canonical trust metadata. The TOE-Agent composes those resources into a research program. Numerical systems compute consequences only after formalization.**

That is the foundation on which a large TOE research program can grow without turning PhysMath into physics, Phys into a universal CAS, or the TOE into a monolithic Go application.

**End state: architectural responsibilities are explicit, dependencies are acyclic, semantic ownership is non-overlapping, and future TOE work can expand through composition rather than core entanglement.**
