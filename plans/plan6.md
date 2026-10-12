Yes. The next artifact should be a **final architecture-closure amendment**, not the coding-agent implementation plan yet.

The four reviews are converging on the same underlying problem: the architecture is conceptually correct, but several boundaries are still described as names and intentions rather than **mechanically checkable contracts**. DeepSeek is explicit that the current document still leaves the coding agent to invent schemas, ownership, identity, validation, and state transitions. plan5_deepseek Z reaches the more permissive conclusion that the architecture is closable, but still requires the provenance law and the theory-local-algebra/PhysMath relationship before implementation planning. plan5_z

So I would do one disciplined pass called:

# Final Architecture Closure Plan — v1.3

The objective is not “make the architecture nicer.”

The objective is:

> **A competent coding agent, given v1.3 + the frozen Phys specification, must not have to choose among two architecturally legitimate implementations.**

That gives us a hard stopping condition.

---

# 1. Freeze the load-bearing architecture first

Do not reopen these decisions during v1.3.

| Decision | v1.3 treatment |
|---|---|
| PhysMath has no dependency on Phys | **LOCK** |
| Phys has no dependency on PhysMath | **LOCK** |
| `phys-*` may depend on both | **LOCK** |
| Phys owns its existing trust/corpus authority | **LOCK** |
| `phys-*` owns physical interpretation | **LOCK** |
| Project A owns research state | **LOCK** |
| TrustAssertion is future design only | **LOCK** |
| No runtime corpus-promotion API | **LOCK** |
| Reference-DAG is the cross-project model | **LOCK** |
| In-memory typed values vs serialized references | **LOCK** |
| Replay is authority-local | **LOCK** |
| PhysMath AST is closed | **LOCK** |
| `phys-*` does not inject AST node types into PhysMath | **LOCK** |
| t-flow and k-flow remain distinct | **LOCK** |
| No automatic scientific truth adjudication | **LOCK** |
| `phys-interop` runtime remains deferred | **LOCK** |
| Per-`phys-*` conformance is mandatory | **LOCK** |

This preserves the architectural core that all four reviews accepted. plan5_deepseek

---

# 2. Solve the ArtifactRef ownership problem once

This is the first actual closure decision.

The current architecture requires one ecosystem-wide reference identity while forbidding PhysMath↔Phys dependency and deferring `phys-interop`. That leaves no legitimate owner. Gemini identifies this exact Go-module deadlock. plan5_gemini

## Decision

Create a **tiny, neutral, zero-domain module** solely for artifact identity.

For example:

```text
github.com/PithomLabs/phys-artifact
```

It is not Phys.

It is not PhysMath.

It is not `phys-interop`.

It contains only:

```text
ArtifactRef
canonical identity/framing rules
schema/version identifiers
artifact-reference errors
```

It contains:

```text
NO physics
NO symbolic mathematics
NO trust
NO registry
NO interpretation
NO adapters
```

Dependency shape becomes:

```text
             phys-artifact
             /     |      \
            /      |       \
      PhysMath     Phys     phys-*
                         \
                        TOE-Agent
```

while preserving:

```text
PhysMath ⊥ Phys
```

### Closure rule

Every ecosystem-wide serialized reference uses the same `ArtifactRef` definition.

No project defines its own incompatible `ArtifactRef`.

---

# 3. Pin ArtifactRef identity completely

The old `artifact_id OR content_digest` language must disappear. DeepSeek explicitly identifies this contradiction. plan5_deepseek

Use exactly:

```go
type ArtifactRef struct {
    Namespace    string
    ArtifactType string
    SchemaVersion uint32
    Digest       [32]byte
}
```

Identity equality is:

```text
Namespace
+ ArtifactType
+ SchemaVersion
+ Digest
```

all four must match.

Digest alone is **not** sufficient identity.

## Canonical identity law

```text
Digest =
SHA-256(
    Namespace
    || SchemaVersion
    || ArtifactType
    || CanonicalPayload
)
```

but the byte framing must be explicit.

Do not permit:

```text
"a" + "bc"
```

to be equivalent to:

```text
"ab" + "c"
```

The review specifically caught this. plan5_gemini

Use fixed binary framing:

```text
Len32(namespace) || namespace
|| Version32(schema_version)
|| Len32(artifact_type) || artifact_type
|| canonical_payload
```

where all integers are unsigned big-endian.

## Important second lock

The `ArtifactRef` itself is **not included recursively in the canonical payload**.

Instead:

```text
canonical payload
      ↓
digest
      ↓
ArtifactRef
```

That eliminates recursive identity ambiguity.

---

# 4. Separate canonical identity encoding from transport encoding

The review correctly rejects “just use encoding/json” as a sufficient canonicalization specification. plan5_deepseek

v1.3 should say:

> JSON is an interchange format, not automatically the identity format.

For every durable artifact schema, specify:

```text
Canonical payload:
    exact bytes used for identity

Transport representation:
    JSON or other external encoding used for interchange
```

For MVP, the simplest permitted canonical form is:

```text
deterministic schema-owned representation
```

with:

```text
no unordered maps
no floating-point values
no unspecified field order
no implicit omission
no implementation-defined formatting
```

Every artifact schema gets at least one canonicalization test vector.

That removes cross-language and cross-version digest ambiguity.

---

# 5. Pin the distinction between in-memory composition and serialized references

This needs to be stated as a law rather than an explanation.

## In memory

Use ordinary typed Go values:

```text
phys-gr Tensor
phys-gr Metric
PhysMath Expression
Phys Object
```

No registry lookup is required just to compose these values.

## Across persistence/process boundaries

Use:

```text
ArtifactRef
```

for durable references.

Therefore:

```text
typed Go object ≠ ArtifactRef
```

An `ArtifactRef` does not magically become a Go object.

Resolution requires an explicit resolver owned by the consuming/storage environment.

There is:

```text
NO mandatory global registry
NO universal runtime catalog
NO implicit network lookup
```

This closes N-1 and R-4 together.

---

# 6. Pin the PhysMath IR instead of deferring it

This is mandatory.

The current language saying “final sort set will be pinned before coding” is incompatible with claiming architecture is already closed. DeepSeek identifies that contradiction directly. plan5_deepseek

The v1.3 architecture must contain a normative table:

```text
Sort ID | MVP? | Child structure | Payload | Canonicalization |
        |      |                 |         | Equality
        |      |                 |         | Substitution
        |      |                 |         | Differentiation
        |      |                 |         | Normalization
        |      |                 |         | Serialization
        |      |                 |         | Projection
```

For every current PhysMath sort.

Then classify each as:

```text
MVP
DEFERRED
FORBIDDEN
```

The existing v1.2 sort inventory should be preserved rather than redesigned during this pass.

The implementation plan then implements that frozen table; it does not decide it.

---

# 7. Make the Phys projection subset a strict semantic mirror

The reviewers agree that the projection subset should exactly mirror frozen Phys:

```text
Symbol
Rational
Add
Mul
Neg
Pow
Sqrt
```

and nothing else is implicitly added.

Z specifically approves the “mirror-as-sort-obligation” formulation. plan5_z

Pin:

```text
PhysMath subset
        ↕ exact compatibility
Frozen Phys expression subset
```

The bridge must fail closed.

No heuristic conversion.

No semantic guess.

No automatic widening.

---

# 8. Pin the Pow exponent rule

The reviewer found a concrete structural mismatch: PhysMath can represent broader powers than frozen Phys.

Qwen's resolution is precise and should be adopted. plan5_gemini

The projection rule becomes:

```text
PhysMath.Pow(base, exponent)
```

is projectable only when `exponent` satisfies the exact frozen Phys `Pow` exponent contract.

Otherwise:

```text
ProjectionClass = UNREPRESENTABLE
Reason = NON_RATIONAL_EXPONENT_IN_POW
```

The source PhysMath expression is untouched.

This is a good example of the standard v1.3 rule:

> **Never repair an external mathematical object silently to make it fit the trusted substrate.**

---

# 9. Define the projection classification algorithm compositionally

This closes N-3.

Exactly three classes:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

with these rules:

### FULLY

All semantically required structure is representable and no required information is lost.

### PARTIALLY

A projection is explicitly allowed by the contract to discard information, and the discarded information is enumerated in a `LossReport`.

### UNREPRESENTABLE

Required semantics cannot be expressed in frozen Phys.

Most importantly:

```text
projectable(child) ≠ automatically projectable(parent)
```

For a composite:

```text
parent is FULL
    iff all required children are FULL
    and parent's own semantics are representable

parent is PARTIAL
    iff contract explicitly permits loss
    and every loss is declared

otherwise UNREPRESENTABLE
```

No coding-agent discretion remains.

---

# 10. Do not create a universal Projectable interface

One reviewer suggests a universal:

```go
ProjectToPhys()
```

interface. plan5_qwen

I would **not** adopt that.

It recreates the universal interface problem the architecture is trying to avoid.

Instead, v1.3 should establish:

> Any package claiming that one of its artifacts is projectable MUST provide a deterministic projection procedure conforming to the formalization contract.

The API can remain typed and package-local.

The architecture specifies the **protocol contract**, not one universal Go interface.

This distinction matters.

---

# 11. Close the PhysMath condition model

Math conditions remain pure mathematics.

No trust metadata.

No physical interpretation.

No corpus status.

The underlying issue is that symbolic transformations can generate preconditions and those cannot be silently discarded. Qwen identifies this directly. plan5_gemini

Use an immutable result concept:

```go
type TransformResult struct {
    Output     Expr
    Conditions []MathCondition
    Trace      TransformationTrace
}
```

The architecture should require:

```text
conditions are explicit
conditions are immutable
conditions are deterministically ordered
conditions are accumulated
conditions are never silently discarded
```

Thus:

```text
x / x → 1
```

cannot silently erase:

```text
x ≠ 0
```

No general theorem prover is introduced.

Condition discharge occurs only through an explicitly defined operation.

This keeps the math system pure while making symbolic side conditions auditable.

---

# 12. Make t and k structurally distinct

The current architecture saying PhysMath “supports” separate derivatives is insufficient.

Qwen correctly identifies that string names such as `"t"` and `"k"` are not stable semantic identities. plan5_gemini

Use an explicit differential-domain tag.

Conceptually:

```text
DifferentialDomain =
    COORDINATE
    TIME
    RG_SCALE
```

A differential operator contains:

```text
target variable identity
+
differential domain
```

Therefore:

```text
∂/∂t
```

and:

```text
∂/∂k
```

remain distinct even if names are changed or expressions are transformed.

The mandatory negative test must prove:

```text
a time derivative cannot be rewritten as an RG derivative
simply by changing a symbol name.
```

---

# 13. Allow theory-local algebras without creating a PhysMath plugin AST

This is the other issue Z explicitly asks to close. plan5_z

Pin:

```text
phys-* may define theory-local mathematical algebras.
```

For example:

```text
phys-gr::GRExpr
```

remains `phys-gr`-owned.

It does **not** become a new PhysMath AST sort.

When cross-theory reuse becomes justified:

```text
theory-local expression
        ↓
deterministic mapping
        ↓
PhysMath Expression
```

only where the mapped structure belongs to the frozen PhysMath sort set.

No forced migration in MVP.

No dynamic trait/AST extension registry.

No `phys-gr` import into PhysMath.

---

# 14. Remove “promotion” ambiguity between proposed and physical interpretation

Rather than defining a mysterious promotion mechanism, make these two artifacts permanently distinct.

## `ProposedInterpretation`

Owned by Project A.

Meaning:

```text
research claim about how a mathematical object
might be understood physically
```

Not authoritative.

## `PhysicalInterpretation`

Owned by the relevant `phys-*` framework.

Meaning:

```text
framework-owned physical semantics
```

No generic:

```text
Promote()
Accept()
Approve()
```

operation exists.

Instead:

```text
ProposedInterpretation
        |
        | optional explicit relationship
        v
PhysicalInterpretation ArtifactRef
```

A phys-* framework may publish a PhysicalInterpretation independently.

This eliminates the entire “who promotes whom?” problem rather than building a promotion workflow.

---

# 15. Simplify PhysicalContract instead of creating a generic rule engine

The review correctly asks what `CompatibilityRules` actually means. plan5_deepseek

The better architectural move is to **remove generic compatibility-rule machinery**.

Make the physical contract declarative and narrow:

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

Validation is:

```text
all required references resolve
required math sorts match
required conditions are present
required conventions are present
framework identity matches
schema compatibility rules are satisfied
```

Domain-specific physics validation remains in the owning `phys-*`.

Do not create:

```text
generic predicate language
generic theorem prover
generic compatibility engine
```

This is tighter and much more aligned with the rest of the architecture.

---

# 16. Make the trust boundary explicit

The reviews are very clear on this.

`phys-*` may define domain semantics, but it does **not** create another trust/corpus system.

That means:

```text
Phys trust metadata
    = Phys authority

phys-* domain types
    = physical meaning

Project A research metadata
    = research workflow
```

Do not interpret this as “phys-* cannot define any enums.”

The prohibition should specifically be:

> `phys-*` MUST NOT create a second vocabulary for provenance, corpus status, trust, or assumption authority that competes with Phys.

Domain-specific physical types may exist locally.

If a domain object cannot be represented by frozen Phys:

```text
keep it external
```

Do not force it into a fake Phys `Kind`.

That resolves the closed-enum deadlock identified by Gemini. plan5_qwen

---

# 17. Turn formalization into an actual protocol

This is the largest remaining implementation ambiguity.

Define concrete schemas for:

```text
FormalizationProposal
FormalizationResponse
ProjectionLossReport
```

## Proposal

At minimum:

```text
ProposalRef
ProducerNamespace
ProposalVersion
SourceArtifactRefs
RequestedTarget
ProjectionPolicy
RequiredFrameworkRefs
```

## Response

```text
ProposalRef
ProposalVersion
Outcome
ProjectedArtifactRefs
LossReport
ValidationReport
ProducerNamespace
```

The response must state exactly what happened.

No ambiguous:

```text
accepted
processed
done
```

language.

Use explicit outcomes.

---

# 18. Separate protocol state from protocol outcome

This will also fix the status-vocabulary problem.

Do not make:

```text
REQUEST_REVISION
UNREPRESENTABLE
PROJECTED_AS_HYPOTHESIS
```

all competing “states.”

Use:

```text
ProposalState
ProposalOutcome
```

For example:

```text
ProposalState:
    DRAFT
    SUBMITTED
    RESOLVED
```

Then:

```text
ProposalOutcome:
    PROJECTED_AS_HYPOTHESIS
    PARTIALLY_PROJECTABLE
    UNREPRESENTABLE
    REJECTED
    REQUEST_REVISION
```

`REQUEST_REVISION` creates a new proposal revision rather than mutating an old proposal.

That eliminates the transition ambiguity identified by DeepSeek. plan5_deepseek

---

# 19. Do not invent a new Phys ingestion API

This is critical.

The reviews contain proposed API names such as:

```text
session.IngestHypothesisExpression()
```

but a reviewer suggestion is not evidence that the frozen Phys API contains such a method. Gemini's suggestion should therefore be treated as a checklist item, not copied blindly. plan5_gemini

The v1.3 rule should be:

> The formalization implementation plan must enumerate the exact already-existing public Phys APIs used for hypothesis-grade projection. No new Phys API may be introduced to support the architecture.

The implementation agent must inspect the frozen specification and use only those APIs.

---

# 20. Explicitly pin the provenance law

This is Z's one remaining substantive closure item. plan5_z

Add this as a normative rule:

> Any session-recorded correspondence operating on hypothesis-grade operands remains hypothesis-grade under the frozen Phys contamination law. The formalization protocol expects and records this result; it is never treated as a failure and never interpreted as corpus promotion.

This is essential because otherwise the protocol boundary quietly creates a trust inconsistency.

---

# 21. Define MathTrace once and eliminate MathObjectRef

DeepSeek correctly notes that `MathObjectRef` is used without definition. plan5_deepseek

Do not create another reference type.

Replace it with:

```text
ArtifactRef
```

for durable traces.

For purely in-memory operations:

```text
typed Go values
```

For durable traces:

```text
ArtifactRef
```

`MathTrace` therefore becomes an authority-local trace containing:

```text
OperationID
ProducerNamespace
OperationSchemaVersion
Input ArtifactRefs
Output ArtifactRef
Condition refs
Canonical parameters
```

This makes trace persistence compatible with the ecosystem identity system without creating another reference ontology.

---

# 22. Define the TOE-Agent research ledger as a meta-ledger

Gemini's “cross-module replay illusion” is important, but the correct interpretation is not that the architecture needs one global replay engine. plan5_qwen

The rule is:

```text
PhysMath replays PhysMath operations.
phys-* replays phys-* operations.
Phys replays Phys operations.
```

Project A can maintain a **research ledger** whose steps refer to those outputs:

```text
ResearchStep {
    Operation
    InputRefs
    OutputRef
    ProducerNamespace
}
```

It does not replay internal operations from another authority.

Therefore:

```text
research replay
    = replay the research graph

authority replay
    = replay the authority's actual computation
```

Those are different contracts.

---

# 23. Fix the Numerical boundary

Qwen correctly catches a dependency/data-flow mismatch: the Numerical layer is said not to consume PhysMath directly, while `ComputationalContract` refers to mathematical objects. plan5_gemini

Make the boundary:

```text
PhysMath
   ↓
phys-* domain-specific lowering
   ↓
ComputationalContract / execution representation
   ↓
Numerical
```

Numerical:

```text
does NOT parse PhysMath expressions
does NOT invent symbolic lowering
does NOT infer physical semantics
does NOT repair formalization
```

`ComputationalContract` is therefore an execution boundary artifact, not another symbolic-math representation.

Source references may be retained for provenance, but Numerical does not consume them as symbolic input.

---

# 24. Turn status vocabularies into real typed state machines

This directly addresses N-9.

Keep the vocabulary already established by v1.2, but for each status set add:

```text
type
legal values
entry conditions
allowed transitions
terminal states
required payload
forbidden transitions
```

Do not introduce new statuses merely to make implementation convenient.

Also preserve the distinction between:

```text
Research status
Formalization outcome
Projection class
Phys corpus status
Obstacle class
```

Never use one enum for another.

This also resolves the `UNREPRESENTABLE` dual-use residue: same word, different typed domains. Z specifically calls for this acknowledgement. plan5_z

---

# 25. Replace “named tests” with executable acceptance properties

The current test inventory is good, but DeepSeek correctly notes that names like “PhysMath → no Phys” are not sufficient acceptance criteria. plan5_deepseek

Every architectural test in v1.3 gets:

```text
Given
When
Then
Failure condition
```

Examples:

### Dependency

```text
Given PhysMath module graph
When dependency audit runs
Then no Phys import exists
Failure = any direct or indirect production dependency on Phys
```

### Identity

```text
Given identical canonical payload
When encoded twice
Then digest is byte-identical
Failure = different digest
```

### t/k

```text
Given ∂/∂t and ∂/∂k
When identifiers are transformed
Then domain identities remain distinct
Failure = substitution by spelling alone changes role
```

### Projection

```text
Given Pow(x, y)
When y is non-Rational
Then classification = UNREPRESENTABLE
Failure = projection succeeds
```

### Trust

```text
Given hypothesis-grade inputs
When formalization records correspondence
Then resulting relation remains HYPOTHESIS
Failure = IDENTIFIED/ESTABLISHED appears
```

### Physical interpretation

```text
Given one MathArtifact
When two phys-* frameworks interpret it
Then both interpretations may coexist
Failure = silent global winner/supersession
```

---

# 26. Pin the frozen Phys authority

The architecture currently says “the frozen Phys specification” without an exact identifier. DeepSeek flags this. plan5_deepseek

v1.3 must contain an explicit authority tuple:

```text
Phys repository/module
Frozen specification identifier
Frozen specification version
Required repository revision/commit
```

Also separate:

```text
semantic authority
process instructions
descriptive documentation
```

so the coding agent knows which source controls architectural interpretation.

---

# 27. Make supersession machine-checkable

The supersession list is already good conceptually. Z confirms that. plan5_z

But each entry should contain:

```text
Previous document ID
Previous version
Previous ADR/decision
Current ADR
Effective version
Reason
```

Then explicitly require:

```text
no two superseded decisions may both be marked active
```

This closes N-12.

---

# 28. Add the final “no invention” matrix

This should become the central handoff mechanism.

Create a table with these columns:

```text
Architectural concern
Normative decision
Owner
Concrete schema/type
Validation rule
Acceptance test
Source ADR
```

At minimum:

```text
Dependency direction
Artifact identity
ArtifactRef ownership
Canonical encoding
Namespace ownership
In-memory vs serialized composition
PhysMath sort set
MathCondition lifecycle
Projection subset
Projection classification
Physical interpretation ownership
Physical contract
Trust boundary
Formalization state
Formalization outcome
Provenance law
Research ledger
Replay authority
phys-* conformance
Numerical boundary
Frozen Phys authority
Supersession
```

The rule is then:

> **If a row has no normative decision, v1.3 is not closed.**

That turns the review problem into a mechanical completeness test.

---

# 29. Do one deliberate simplification pass

This is important because several reviewer proposals would solve ambiguity by adding machinery.

Do **not** accept complexity merely to close a reviewer comment.

The v1.3 simplification rules should be:

```text
No universal Artifact interface
No universal Projectable interface
No generic compatibility-rule language
No global registry
No generic theorem prover
No dynamic AST extension mechanism
No runtime trust promotion
No global replay engine
No second assumption/corpus system
No symbolic engine inside Numerical
```

The best architectural response to an undefined abstraction is often:

> **remove the abstraction rather than define a complicated version of it.**

That is how we avoid solving ambiguity by creating a larger architecture.

---

# 30. Final AI review protocol

After v1.3 is written, send **the identical v1.3 document** to all four agents again.

But change the review prompt.

Do not ask:

> “Improve this architecture.”

Ask:

> **“Perform a blocker-only closure audit. Do not redesign or propose optional improvements. Find only cases where (a) two legitimate implementations remain possible, (b) a stated architectural rule contradicts another rule, (c) a required type/schema/owner/transition/identity rule is missing, or (d) a coding agent would have to invent an architectural decision. For every finding, identify the exact section and give a concrete counterexample demonstrating the ambiguity.”**

Then consolidate their responses using this exact classification:

```text
BLOCKER
    requires architecture change

CONTRACT GAP
    missing concrete schema/validation/test

CLARIFICATION
    wording only, no architectural decision

IMPLEMENTATION DETAIL
    belongs in coding plan

PREFERENCE
    not a blocker
```

Only the first two reopen v1.3.

---

# 31. The final stopping rule

Do **not** require the AIs to agree that the architecture is elegant.

Require this:

```text
DeepSeek: 0 BLOCKER
Gemini:   0 BLOCKER
Qwen:     0 BLOCKER
Z:        0 BLOCKER
```

and:

```text
all prior findings N-1...N-12 resolved
all residual findings R-1...R-6 resolved
no new contradiction
no unresolved ownership
no unresolved schema
no unresolved state transition
no unresolved identity rule
no unresolved dependency
no unresolved authority
```

Then freeze:

```text
PHYSMATH_PHYS_EXTERNAL_ARCHITECTURE_v1.3
STATUS: FROZEN
```

Only then write:

```text
IMPLEMENTATION_PLAN.md
```

---

# What the implementation plan should *then* contain

At that point the implementation plan becomes deliberately boring.

It should no longer contain architectural decisions.

Its job is:

```text
repository sequence
package/file changes
interfaces already mandated by v1.3
implementation order
migration/compatibility steps
tests
fixtures
golden vectors
CI checks
documentation updates
definition of done
```

The coding agent should be able to read:

```text
v1.3 = WHAT MUST EXIST
implementation plan = HOW WE BUILD IT
```

not:

```text
implementation plan = WHAT THE ARCHITECTURE SHOULD BE
```

That distinction is the key to finally ending the review loop.

## The one-line strategy

**Do one final v1.3 pass that converts every remaining architectural noun into an owned, typed, canonical, validated contract; remove abstractions that cannot be pinned cleanly; have all four AIs perform a blocker-only closure audit; freeze; then write the implementation plan.**

That directly addresses DeepSeek's requirement that every artifact have schema, identity, canonical form, composition, ownership, validation and tests. plan5_deepseek It also incorporates Z's two remaining substantive requests—provenance law and theory-local algebra—and preserves the architectural resolutions the reviewers already accepted. plan5_z