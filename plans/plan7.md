I consolidated the four `plan6` reviews against the actual target: **zero architectural blockers and zero architectural invention left to the coding agent**.

The important result is that the reviews are no longer disagreeing about the core architecture. They are finding the same remaining failure mode: **plan6 tried to close architecture by introducing a few new mechanisms whose own ownership and semantics were not yet pinned.**

DeepSeek therefore marks plan6 “not closed,” with eight new blockers introduced by the closure plan itself. plan6_deepseek Gemini identifies seven cross-module contract blockers. plan6_qwen Qwen concentrates on the concrete Go/module and frozen-Phys consequences. plan6_gemini Z independently reaches the same conclusion but identifies the two most important conceptual corrections: **Phys must remain frozen and must not import the new artifact module, and physical differential-domain semantics must not enter PhysMath.** plan6_z

So I would **not** write the implementation plan yet.

I would now produce one final artifact:

> **`PHYSMATH_PHYS_EXTERNAL_ARCHITECTURE_v1.3.md` — the actual closed specification, not another closure plan.**

Then run exactly one blocker-only audit against it.

Here is the consolidated closure plan I would use.

---

# 1. Preserve the architecture that all four reviews now agree on

These become immutable architectural commitments for v1.3:

```text
PhysMath ⊥ Phys
phys-* may depend on both
PhysMath = pure mathematical substrate
phys-* = physical semantics
Phys = frozen trust/formalization authority
Project A / TOE-Agent = research orchestration/state
Numerical = downstream computation
```

Also preserve:

```text
reference-DAG
in-memory typed composition
serialized ArtifactRef references
authority-local replay
no runtime corpus promotion
TrustAssertion = future design only
no dynamic PhysMath AST extensions
t-flow != k-flow
no automatic truth adjudication
```

All four reviews accept this core direction. plan6_deepseek

Do **not** reopen these decisions in v1.3.

---

# 2. Fix the biggest mistake in plan6: Phys remains completely frozen

This is the most important correction.

Plan6 introduced:

```text
Phys -> phys-artifact
```

which is unacceptable because the frozen Phys contract includes the dependency/module surface.

Z makes this explicit: frozen Phys must be a **reference target, never a new-import source**. plan6_z

Therefore v1.3 must state:

```text
Phys
    imports nothing new
    receives no architecture-driven modification
    receives no phys-artifact dependency
    receives no new API
```

The dependency graph is:

```text
phys-artifact ──> mechanical dependency only

PhysMath   ──────> phys-artifact
phys-*     ──────> phys-artifact
TOE-Agent  ──────> phys-artifact

Phys       ──────> no new dependency
```

The important conceptual distinction is:

> **A system may reference Phys artifacts without Phys itself participating in the reference-library dependency.**

Consumers calculate references to frozen Phys artifacts from the frozen Phys canonical representation.

This preserves both the ecosystem reference model and the hard Phys freeze.

---

# 3. Keep `phys-artifact`, but redefine what it is

The reviews exposed that calling it another “authority” is conceptually dangerous.

`phys-artifact` is **not an authority**.

It is a neutral mechanical library.

Its ownership is:

```text
artifact identity mechanics
canonical reference encoding
reference validation
```

It owns **no semantic artifacts**.

It contains no:

```text
physics
mathematics
trust
corpus
interpretation
research state
registry
```

Its existence does not create a sixth semantic authority.

That resolves B-13-1 without compromising the original separation.

---

# 4. Make ArtifactRef a name, not a resolver

This is a critical conceptual cleanup.

DeepSeek correctly observes that saying “ArtifactRef enables resolution” conflicts with the prohibition on a global registry or implicit lookup. plan6_deepseek

Define:

> **`ArtifactRef` is a content-addressed artifact name. It is not a resolver.**

So:

```text
ArtifactRef
    ↓
names artifact
```

but:

```text
Resolver
    ↓
binds name to storage/environment
```

The resolver is outside the semantic cores.

There is no:

```text
global registry
implicit network lookup
universal resolver
mandatory database
```

A storage environment may implement a resolver.

That is an infrastructure decision, not a semantic one.

This distinction should appear directly in the normative architecture.

---

# 5. Pin ArtifactRef identity and types

Do not retain:

```text
artifact_id OR content_digest
```

That is permanently eliminated.

Use content addressing only.

I recommend:

```go
type ArtifactRef struct {
    Namespace     string
    ArtifactType  string
    SchemaVersion string
    Digest        [32]byte
}
```

I deliberately prefer `string` for `SchemaVersion` rather than `uint32`, because frozen Phys already uses `"1"` and future schemas need not be constrained to an integer representation.

Identity is:

```text
(Namespace, ArtifactType, SchemaVersion, Digest)
```

not digest alone.

Two refs are equal iff all four fields are equal.

---

# 6. There must be exactly one digest framing rule

Delete any abstract/illustrative digest formula that differs from the normative one.

Use only:

```text
Len32(Namespace)
|| Namespace
|| Len32(SchemaVersion)
|| SchemaVersion
|| Len32(ArtifactType)
|| ArtifactType
|| CanonicalPayload
```

all encoded as:

```text
UTF-8 strings
unsigned 32-bit big-endian lengths
```

then:

```text
SHA-256(...)
```

This resolves the framing ambiguity identified by both Gemini/Qwen. plan6_gemini

No raw concatenation.

No implementation-defined encoding.

No JSON-dependent digest.

---

# 7. Define embedded ArtifactRef encoding separately

Gemini correctly found a second-level problem: even once top-level identity is fixed, the architecture had not said how an `ArtifactRef` appearing **inside another artifact** is encoded. plan6_gemini

Pin:

```text
EncodeArtifactRef =
    Len32(Namespace)
    || Namespace
    || Len32(SchemaVersion)
    || SchemaVersion
    || Len32(ArtifactType)
    || ArtifactType
    || Digest[32]
```

Embedded references are never encoded as:

```text
JSON
hex strings
bare digest bytes
```

within canonical payloads.

That gives one representation everywhere.

---

# 8. Canonical payload and transport format must remain separate

The architecture should explicitly distinguish:

```text
CanonicalPayloadBytes
```

from:

```text
TransportEncoding
```

JSON can remain an interoperability representation.

But:

> **`encoding/json` by itself is not the canonical identity specification.**

Every durable artifact schema must define deterministic canonical bytes.

No:

```text
map ordering dependence
float formatting dependence
implicit field omission
locale dependence
native Go representation dependence
```

---

# 9. Freeze the PhysMath IR in v1.3 itself

This is non-negotiable.

DeepSeek correctly identified the contradiction in saying “the sort set will be pinned before coding” while simultaneously declaring architecture closed. plan6_deepseek

Therefore v1.3 must contain the **actual normative PhysMath sort table**.

For every sort, specify:

```text
SortID
MVP / DEFERRED / FORBIDDEN
fields
child structure
canonicalization
equality
substitution
differentiation
normalization
serialization
projection behavior
```

Do not redesign the mathematical inventory during this pass.

Take the existing v1.2 inventory and convert it into the normative table.

A cell cannot say merely:

```text
yes
supported
normalizes
```

It must say:

```text
N/A
RULE <identifier>
DEFERRED + CONTAINMENT
```

Z correctly identifies this as a key way to prevent ambiguity simply being moved into table cells. plan6_z

---

# 10. Clarify what is an IR sort versus a meta-level result

This resolves one of the subtle inconsistencies caught by DeepSeek.

Pin:

```text
Symbol
```

as an **Expression node kind**, matching the frozen Phys expression model.

It is not a separate top-level mathematical artifact sort.

Likewise:

```text
TransformResult
ConditionSet
TransformationTrace
LossReport
ValidationReport
```

are **operation/result/meta-level structures**, not new mathematical expression node kinds.

This is important because otherwise “closed AST” and “new result type” get conflated.

---

# 11. Keep PhysMath mathematically pure: remove DifferentialDomain from the core

This is the second critical correction.

Plan6's:

```text
COORDINATE
TIME
RG_SCALE
```

enum is not acceptable in pure PhysMath.

Z correctly identifies that this smuggles physical ontology into the mathematical substrate. plan6_z

The clean solution is simpler than the proposed enum:

### Mathematical identity

A differential operator targets a **parameter identity**.

Thus:

```text
parameter A != parameter B
```

is enough to distinguish the two variables.

### Physical meaning

Physical systems may interpret those identities:

```text
parameter identity t
    -> physical time

parameter identity k
    -> RG scale
```

but that mapping belongs to `phys-*` / Project A.

Therefore PhysMath does not know:

```text
TIME
RG_SCALE
```

at all.

This is a much better embodiment of the “notation is not semantics” principle.

Optional external annotations may exist, but PhysMath treats them as opaque metadata and never uses them as mathematical equality rules.

---

# 12. Pin the t/k invariance law

The enforcement mechanism becomes:

```text
Differential operator
    =
    target parameter identity
```

The identity is immutable.

Renaming a display label:

```text
t -> τ
```

does not change parameter identity.

Therefore:

```text
∂/∂t
```

cannot silently become:

```text
∂/∂k
```

because a parameter's mathematical identity has changed.

This makes the negative test mechanical without injecting physics into PhysMath.

---

# 13. Keep theory-local algebras, but never inject them into PhysMath

For example:

```text
phys-gr::GRExpr
```

may exist.

But:

```text
GRExpr
```

is not a PhysMath extension node.

Instead:

```text
GRExpr
    ↓
explicit deterministic mapping
    ↓
PhysMath Expression
```

when appropriate.

No automatic plugin registry.

No runtime extension dispatch.

No PhysMath import of `phys-gr`.

Z explicitly confirms this part of plan6 was correct. plan6_z

The right phrase is:

> **PhysMath is closed per schema version, not closed forever.**

A future version may add a mathematically justified sort through an explicit versioned architecture change.

---

# 14. Make projection completely deterministic

Projection has three outcomes:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

Use this exact rule:

```text
FULL
    all required structure representable
    and no information loss

PARTIAL
    only when a versioned projection contract
    explicitly permits a known information loss

UNREPRESENTABLE
    otherwise
```

And the crucial MVP lock:

> **MVP defines zero permitted-loss contracts.**

Therefore:

```text
PARTIALLY_REPRESENTABLE
```

exists in the vocabulary but is unreachable in MVP.

Future contracts may explicitly introduce permitted loss.

That closes F-1 from Z and avoids a silent “some children projected, therefore parent partial” policy. plan6_z

---

# 15. Composition must be recursive, not heuristic

For a composite:

```text
parent FULL
    iff all required children FULL
    and parent semantics FULL

parent PARTIAL
    iff an explicit permitted-loss rule applies

otherwise
    UNREPRESENTABLE
```

A projectable child never makes an unprojectable parent projectable.

No coding agent decides this case-by-case.

---

# 16. Pin the Pow rule

The projection subset remains:

```text
Symbol
Rational
Add
Mul
Neg
Pow
Sqrt
```

but its exponent domain must match frozen Phys exactly.

Therefore:

```text
Pow(base, exponent)
```

is projectable only if the exponent meets the frozen Phys exponent contract.

For an unsupported symbolic exponent:

```text
classification = UNREPRESENTABLE
reason =
    NON_RATIONAL_EXPONENT_IN_POW
```

Do not simplify or rewrite the original PhysMath expression to force projection.

Qwen explicitly identified this concrete hazard. plan5_gemini

---

# 17. Pin associative projection

Gemini catches another legitimate implementation choice: PhysMath may use n-ary `Add`/`Mul` while Phys uses binary trees. plan6_gemini

Therefore:

```text
PhysMath canonicalization
    =
    flatten associative Add/Mul
    +
    deterministic child ordering
```

Projection becomes:

```text
[a,b,c]
    ->
Add(Add(a,b),c)
```

using the canonical order.

Thus the projection is unique.

---

# 18. Close MathCondition semantics

Use:

```text
TransformResult {
    Output
    Conditions
    Trace
}
```

with all three immutable.

Conditions are accumulated during transformation.

They are never attached as mutable fields to AST nodes.

They are never discarded.

They are canonically sorted by:

```text
canonical condition digest bytes
```

before the result is finalized.

This directly closes Qwen's condition-ordering concern. plan6_gemini

MVP also explicitly has:

```text
NO condition-discharge operation
```

Conditions accumulate.

Discharge is a future separately specified operation.

That prevents another dangling abstraction.

---

# 19. Separate ProposedInterpretation from PhysicalInterpretation permanently

No promotion API.

No:

```text
Promote()
Approve()
Accept()
```

Instead:

```text
ProposedInterpretation
    = Project A research artifact

PhysicalInterpretation
    = phys-* owned framework contract
```

The former may point toward the latter.

It never becomes the latter automatically.

A physical framework can independently publish its interpretation.

This eliminates the promotion ambiguity completely rather than inventing a governance workflow.

---

# 20. PhysicalContract needs a narrow declarative schema

Do not resurrect generic `CompatibilityRules`.

Use a concrete contract such as:

```text
ExtensionID
SchemaVersion
PhysicalType
FrameworkRef
MathRefs
RequiredConditions
RequiredConventions
InterpretationClass
```

Validation is mechanical:

```text
references resolve
required structures exist
required conditions exist
required conventions exist
framework identity matches
schema compatibility holds
```

No generic predicate language.

No general theorem prover.

No generic physics rule engine.

That was one of the simplifications correctly preserved from plan6. plan6_z

---

# 21. Formalization gets actual typed messages

The current field-list approach is insufficient.

Define at least:

```text
FormalizationProposal
FormalizationResponse
ProjectionLossReport
ValidationReport
```

with every field having:

```text
concrete type
required/optional status
canonical encoding
identity rule
validation rule
```

For example:

```text
FormalizationProposal
    ProposalVersion
    ParentProposalRef?
    ProducerNamespace
    SourceArtifactRefs[]
    RequestedTarget
    ProjectionPolicy
    RequiredFrameworkRefs[]
```

Pin MVP:

```text
ProjectionPolicy =
    STRICT_MIRROR_ONLY
```

`RequestedTarget` must be a typed target reference, not a free-form package name.

---

# 22. Define the formalization state machine

Separate:

```text
ProposalState
```

from:

```text
ProposalOutcome
```

For MVP:

```text
DRAFT
    -> SUBMITTED
        -> RESOLVED
```

Outcome describes the resolution:

```text
PROJECTED_AS_HYPOTHESIS
PARTIALLY_PROJECTABLE
UNREPRESENTABLE
REJECTED
REQUEST_REVISION
```

For:

```text
REQUEST_REVISION
```

the original proposal becomes:

```text
RESOLVED
```

and the revised proposal contains:

```text
ParentProposalRef
```

pointing to the resolved parent.

No mutation of historical proposals.

Gemini identifies this exact missing lineage rule. plan6_gemini

---

# 23. Explicitly pin the Phys formalization path

Do not invent:

```text
IngestHypothesisExpression()
```

just because a reviewer proposed it.

The architecture should instead state:

> The implementation plan must use only the exact existing public Phys APIs enumerated from the frozen Phys specification.

The architecture may pin the known public mint authority:

```text
hypothesis.NewCandidateConcept
```

as the raw hypothesis-grade mint path, provided this is confirmed against the frozen specification.

External tooling cannot introduce:

```text
AcceptFormalization()
AcceptTrust()
PromoteCorpus()
```

This preserves the “no new Phys API” rule.

---

# 24. Pin the provenance law with its source

DeepSeek correctly objects to stating the contamination law as though v1.3 itself creates it. plan6_deepseek

Therefore v1.3 should say:

> The protocol relies on the frozen Phys contamination law, and the relevant frozen Phys section is explicitly cited.

Then:

```text
hypothesis-grade inputs
    +
session correspondence
    =
HYPOTHESIS-grade result
```

never:

```text
IDENTIFIED
ESTABLISHED
```

The architecture does not create that rule; it **references the rule already frozen in Phys**.

This is exactly the kind of distinction we need throughout the document:

```text
new architecture
vs
constraint inherited from frozen authority
```

---

# 25. Unify MathTrace without pretending replay is global

`MathTrace` is authority-local.

A research ledger is different.

Thus:

```text
Authority Trace
    = exact internal derivation record
```

while:

```text
Research Ledger
    = cross-authority graph of artifact transitions
```

The research ledger can record:

```text
PhysMath artifact A
    →
phys-gr artifact B
    →
Phys candidate C
```

but it does not replay the internal operations of those authorities.

This is the correct resolution of the “cross-module replay illusion.” plan6_z

---

# 26. Give Numerical a clean contract boundary

The reviews correctly insist that this cannot remain a hand-wave.

I would make the decision:

> **Numerical owns the `ComputationalContract` schema because it is the execution boundary contract, while `phys-*` owns production of valid instances from physical semantics.**

That does **not** give Numerical authority over physics.

Numerical defines:

```text
what executable numeric input looks like
```

not:

```text
what the physical model means
```

So:

```text
PhysMath
     ↓
phys-* lowering
     ↓
Numerical-owned ComputationalContract
     ↓
Numerical
```

`ComputationalContract` contains only execution-level information, such as:

```text
ContractRef
PhysicalModelRef
MathematicalModelRef
parameter data
input layout
requested outputs
numerical method
precision requirements
execution constraints
```

No physical assumption inference.

No reinterpretation.

No symbolic parsing.

---

# 27. Distinguish in-process and persisted ComputationalContract handoff

Gemini correctly catches this missing piece. plan6_gemini

### Same process

The contract handed to Numerical contains fully materialized typed arrays / execution structures.

Numerical performs:

```text
NO ArtifactRef lookup
NO network resolution
```

### Persisted / cross-process

ArtifactRefs may exist for provenance.

The caller/storage environment resolves and materializes the necessary data before the numerical execution boundary.

Therefore Numerical itself remains deterministic and local.

---

# 28. Solve floating-point canonicalization once

Qwen identifies the problem created by saying “no floats anywhere” while inevitably needing numeric approximations downstream. plan6_gemini

The clean rule is:

```text
PhysMath exact mathematics:
    no floating-point AST values

Physical symbolic parameters:
    exact rational whenever exactness is intended

Numerical contracts:
    may use Float64Bits
```

Canonical numerical floating-point representation:

```text
uint64
=
IEEE-754 float64 bit pattern
```

serialized big-endian.

Reject:

```text
NaN
±Inf
locale-dependent formatting
decimal text formatting
native memory layout
```

unless a future contract explicitly specifies them.

This separates **exact symbolic mathematics** from **numerical approximation** rather than pretending they are the same representation.

---

# 29. Clarify the MVP/post-MVP boundary

Qwen correctly finds that the architecture and the frozen Phys MVP were being discussed at different scopes. plan6_qwen

The v1.3 document should explicitly say:

```text
Frozen Phys MVP
    = governed by specs_v2_3.md and its operational freeze

v1.3 ecosystem architecture
    = target architecture for PhysMath / phys-* / Project A / Numerical integration

No implementation plan may infer that all ecosystem repositories
must be implemented as part of the frozen Phys MVP.
```

This removes the nine-repositories-versus-one-repository ambiguity.

---

# 30. Growth Gate is post-MVP only

No ambiguity here:

```text
MVP:
    Phys frozen
    no Growth Gate implementation
    GR stress evidence does not modify Phys

POST-MVP:
    Growth Gate evaluates evidence
    potential kernel evolution follows explicit governance
```

Qwen identifies this as an important mismatch between the frozen MVP and the later ecosystem. plan6_qwen

The GR stress test is evidence.

It is not permission to modify the kernel.

---

# 31. `phys-interop` stays deferred

Do not allow `phys-interop` to become a dumping ground.

MVP:

```text
NO shared runtime interop module
```

But every `phys-*` package must satisfy the conformance specification.

Future `phys-interop`, if introduced, may contain only:

```text
reference helpers
codecs
schema utilities
conformance tests
mechanical boundary checks
```

and nothing involving:

```text
physics
math semantics
trust
interpretation
corpus authority
research policy
```

This is one of the areas where the four reviews largely converge. plan6_z

---

# 32. Add the full artifact contract template

This is the key mechanism that ends the recurring review loop.

Every durable artifact specification must answer exactly:

```text
OWNER
SCHEMA
IDENTITY
CANONICAL FORM
SERIALIZATION
REFERENCES
RESOLUTION
VALIDATION
LEGAL STATES
LEGAL TRANSITIONS
POSITIVE TEST
NEGATIVE TEST
```

Apply this to:

```text
ArtifactRef
MathArtifact
PhysicalArtifact
PhysicalInterpretation
ProposedInterpretation
CorrespondenceProposal
MathTrace
ResearchHypothesis
FormalizationProposal
FormalizationResponse
ProjectionLossReport
ValidationReport
ComputationalContract
```

TrustAssertion is excluded because it remains:

```text
FUTURE DESIGN TARGET
```

as already established.

DeepSeek's core closure checklist is exactly this shape. plan6_deepseek

---

# 33. Add the No-Invention Matrix as a normative completeness check

The matrix should explicitly contain the entire architectural decision set.

At minimum:

```text
Dependency direction
ArtifactRef owner
ArtifactRef identity
Digest framing
Embedded-ref encoding
Canonical encoding
Namespace ownership
In-memory composition
Serialized composition
PhysMath sort ownership
PhysMath sort set
MathCondition semantics
Parameter identity
Projection subset
Projection classification
Partial-loss policy
Physical interpretation ownership
PhysicalContract
Trust boundary
Formalization states
Formalization outcomes
Formalization ownership
Provenance law
Replay authority
Research ledger
Numerical contract
Numerical handoff
phys-* conformance
Frozen Phys authority
Growth Gate phase
MVP scope
Supersession
```

And the meta-rule:

> **If any required row lacks a normative decision, v1.3 is not closed.**

This was correctly recognized by Z as one of the strongest governance mechanisms in the chain. plan6_z

---

# 34. Make reviewer arbitration evidence-based, not majority-based

I would **not** use majority vote.

That would allow a genuine blocker to be dismissed 3–1.

Nor would I let one reviewer block indefinitely.

Use an objective rule:

A finding is a **BLOCKER** only when the reviewer demonstrates at least one of:

```text
1. direct contradiction between two normative rules

2. two architecturally legitimate implementations remain possible

3. required owner/schema/identity/state/validation is absent

4. coding agent would have to invent an architectural decision
```

and provides:

```text
exact section
concrete counterexample
demonstrated consequence
```

A reviewer assertion without such evidence is:

```text
CLARIFICATION
PREFERENCE
or IMPLEMENTATION DETAIL
```

This gives you a real termination condition without making review either democratic or arbitrary.

---

# 35. Make the final review itself self-contained

Do not require reviewers to remember:

```text
plan3
plan4
plan5
plan6
```

to determine whether v1.3 is closed.

v1.3 should contain its own:

```text
supersession table
closed decision list
no-go list
no-invention matrix
acceptance properties
authority references
```

The reviewer only needs:

```text
v1.3
+
frozen Phys specification
```

Z specifically identifies the need to make the authority tuple explicit: module, `specs_v2_3.md`, operational freeze, and pinned revision. plan6_z

---

# 36. Add the “physical meaning never inferred” prohibition

Qwen's additional finding is worth promoting into a hard rule.

The No-Go list should explicitly reject:

> **PhysMath inferring physical meaning from mathematical structure.**

Thus:

```text
Metric
Connection
Curvature
Manifold
Wavefunction
etc.
```

can exist mathematically without acquiring a physical interpretation merely because their mathematical structure resembles something used in physics.

Physical interpretation must arrive through:

```text
PhysicalInterpretation
phys-*
framework contract
```

This is the deepest architectural safeguard against the original Dancer/Dance/Stage category error.

Qwen explicitly calls for this as a hard architectural prohibition. plan6_qwen

---

# 37. The final v1.3 document structure should therefore be

Not another “plan.”

Use this structure:

```text
1. Status / Authority
2. Scope
3. Constitutional invariants
4. Dependency graph
5. Artifact identity
6. Canonical encoding
7. In-memory vs serialized composition
8. PhysMath IR
9. MathCondition
10. Theory-local mathematics
11. PhysicalInterpretation
12. PhysicalContract
13. Trust boundary
14. Projection contract
15. Formalization protocol
16. Provenance law
17. Research ledger / replay
18. Numerical boundary
19. phys-* conformance
20. Growth Gate / MVP boundary
21. Status state machines
22. Supersession
23. No-Go architectures
24. No-Invention Matrix
25. Acceptance Properties
26. Closure Criterion
```

That becomes the actual architecture contract.

---

# 38. The implementation plan comes only after this

Once v1.3 survives the final audit, there should be a sharp transition:

```text
v1.3 architecture
        ↓
       FROZEN
        ↓
implementation plan
        ↓
coding agent
```

The implementation plan should **not contain architecture decisions**.

It should say things like:

```text
create package X
implement type Y exactly as v1.3
add canonical vector Z
add positive test A
add negative test B
run dependency check C
```

It should never say:

```text
decide whether ArtifactRef is content-addressed
decide whether PhysMath owns the type
decide how projection should behave
decide whether Numerical resolves refs
```

Those questions must already be dead.

---

# Final consolidated verdict

The four reviews are actually giving us a very useful signal:

**The architecture itself is not collapsing. The closure mechanism was one level too abstract.**

The remaining work is now finite and concrete.

The critical corrections are:

```text
Phys remains byte-frozen and imports nothing new
+
phys-artifact is neutral mechanical infrastructure, not an authority
+
ArtifactRef is a content-addressed name, not a resolver
+
exact canonical framing is pinned
+
embedded refs use one canonical encoding
+
PhysMath IR is explicitly frozen in v1.3
+
DifferentialDomain physical enum is removed
+
t/k are distinguished by mathematical parameter identity
+
projection is compositional and deterministic
+
MVP permits zero information-loss projections
+
Formalization has typed messages and states
+
provenance law is explicitly inherited from frozen Phys
+
ComputationalContract has one owner
+
Numerical receives materialized execution data
+
Growth Gate is post-MVP
+
theory-local algebras remain external to PhysMath
+
all artifact classes receive schema/identity/canonicalization/validation/test contracts
+
reviewers can block only with a demonstrated architectural counterexample
```

DeepSeek's latest verdict is particularly useful here: it considers plan6 a good closure method but says exactly that its new decisions must themselves be pinned before v1.3 can be frozen. plan6_deepseek Z independently says the same thing in different terms: fix the Phys dependency edge, remove physical ontology from `DifferentialDomain`, then pin the remaining contract details and the architecture can finally be frozen. plan6_z

**So this is the point where I would stop adding “closure plans.” The next document should be the actual v1.3 architecture specification implementing the decisions above. After that, one blocker-only four-agent audit; then freeze and proceed to the coding-agent implementation plan.**