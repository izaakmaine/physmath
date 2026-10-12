# Consolidated Adversarial Review
## PM-REP-001 — PhysMath + phys-* Representation Specification v1.0

**Review target:** `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.0.md`  
**Reviewers:** DeepSeek, Gemini, Qwen, Z  
**Objective:** zero architectural blockers and zero ambiguity for the coding agent  
**Reference slice:** complete representation of an equivariance-theorem derivation

---

## 1. Executive verdict

The four reviews independently confirm that the strategic architecture is correct:

- Phase 1 is correctly separated from TOE-Agent, Numerical, provenance, trust, and corpus machinery.
- The physics paper is the unit of research, while its formal mathematical payload is the representation target.
- ARI is correctly separated: Analysis and Interpretation remain human/AI research activity; Representation belongs in PhysMath and `phys-*`.
- PhysMath is theory-neutral.
- `phys-*` carries physical meaning and framework-specific assumptions.
- Multiple physical interpretations of the same mathematical representation are legal.
- The equivariance theorem is the correct Phase-1 vertical slice.
- Representation success is not scientific truth.
- Phys remains frozen.
- Dynamic extension of the PhysMath AST is prohibited.
- Numerical and TOE-Agent remain deferred.

These are **closed and must not be reopened**.

The reviews do, however, identify a finite set of mechanical gaps. The consolidated conclusion is:

> **Do not redesign the architecture. Amend the Phase-1 representation specification once to close the representation contract completely, then move directly to implementation planning.**

The correct closure objective is not “more architectural detail.” It is:

```text
every durable family has an exact identity
every primitive has exact bytes
every ArtifactRef field has an exact target
every transformation has an exact rule and parameter encoding
the equivariance fixture has one prescribed artifact graph
parameter matching has one prescribed rule
framework-specific rules have one prescribed namespace rule
phys-* artifacts have exact canonical schemas
```

---

# 2. Findings that are fully accepted

## A. Durable artifact-family registry — BLOCKER

**Raised by:** DeepSeek, Gemini, Z.

The specification says artifact types must be registered, but does not enumerate every durable family.

### Decision

Add one normative master registry.

Exact Phase-1 families:

```text
physmath/expression/1
physmath/function/1
physmath/function_application/1
physmath/map/1
physmath/tensor/1
physmath/index/1
physmath/collection/1
physmath/operator_application/1
physmath/transformation/1
physmath/relation/1
physmath/equation/1
physmath/equation_set/1
physmath/definition/1
physmath/condition/1
physmath/derivation/1
physmath/theorem/1

<phys-namespace>/framework/1
<phys-namespace>/assumption/1
<phys-namespace>/definition/1
<phys-namespace>/equation/1
<phys-namespace>/theorem/1
<phys-namespace>/interpretation/1
```

No implementation agent may invent another spelling.

Deferred mathematical sorts that have no concrete durable schema in Phase 1 do not acquire invented artifact families.

---

## B. Primitive encoding is incomplete — BLOCKER

**Raised by:** DeepSeek, Gemini, Qwen.

Three primitives/types need closure:

```text
BigInt
ConstantID
closed token enums
```

### Decision: BigInt

Use the earlier settled representation:

```text
BigInt:
    Sign = 0x00 non-negative
           0x01 negative

    MagnitudeLength = UInt32BE(byte count)
    Magnitude       = minimal unsigned big-endian bytes

    Encoding = Sign || MagnitudeLength || Magnitude
```

Rules:

```text
zero = 0x00 || 0x00000000
negative zero invalid
no redundant leading zero bytes
```

### Decision: ConstantID

```go
type ConstantID string
```

Canonical encoding:

```text
String(ConstantID)
```

The closed Phase-1 values are:

```text
math.const.pi
math.const.e
math.const.i
```

### Decision: closed enums

All closed token vocabularies use:

```text
Enum = String(token)
```

unless a schema explicitly identifies itself as a binary tagged union.

Therefore:

```text
ComparisonOperator -> String(token)
IndexVariance      -> String(token)
```

There is no competing UInt8 ordinal representation for these enums.

---

## C. ParameterID scope rules — BLOCKER

**Raised by:** Gemini.

The current declaration-position model is correct, but the scope/declaration procedure is not sufficiently operational.

### Decision

Phase-1 supports **Function parameter binding** as the canonical binding mechanism.

For a `Function`:

```text
ScopePath = empty sequence
ParameterOrdinal = zero-based position in Function.ParameterIDs
```

Therefore:

```text
depth = 0
ScopePathBytes = UInt32BE(0)
```

Nested lexical binders are not part of the Phase-1 binder vocabulary and are deferred rather than invented.

The canonical formula remains:

```text
ParameterID =
    SHA-256(
        String("physmath")
        || Bytes(ScopePathBytes)
        || UInt32BE(ParameterOrdinal)
    )
```

Display names never participate.

This preserves the intentional property that `ParameterID` is **local mathematical binding identity**, not global provenance identity.

Cross-artifact identity is explicit through:

```text
BoundParameterRef {
    DeclaringArtifactRef ArtifactRef
    ParameterID         ParameterID
}
```

The declaring artifact provides context; the ParameterID identifies the declaration slot.

---

## D. BoundParameterRef encoding — BLOCKER

**Raised by:** DeepSeek, Gemini, Z.

### Decision

Canonical encoding is exactly:

```text
Encode(BoundParameterRef)
    =
    EncodeArtifactRef(DeclaringArtifactRef)
    || Digest32(ParameterID)
```

No `Bytes()` wrapper surrounds the 32-byte ParameterID.

This is the only canonical representation.

---

## E. Relation representation — BLOCKER

**Raised by:** DeepSeek, Qwen, Z.

Both reviewers correctly identified the dangerous duality between the Expression `Relation` node and durable `physmath/relation/1`.

### Decision

Keep both because both are useful, but define their relationship precisely.

### Expression node

```text
Relation {
    Operator ComparisonOperator
    Left     Expression
    Right    Expression
}
```

Serialized as:

```text
0x09
|| Enum(Operator)
|| CanonicalExpression(Left)
|| CanonicalExpression(Right)
```

### Durable relation artifact

```text
physmath/relation/1
```

Its payload is exactly the relation body **without** the Expression node tag:

```text
Enum(Operator)
|| CanonicalExpression(Left)
|| CanonicalExpression(Right)
```

There is no wrapper and no referenced left/right artifacts.

Therefore the two forms express the same mathematical relation:

```text
embedded relation
    = node tag + relation body

durable relation
    = relation body
```

`Equation.RelationRef` and `MathCondition.PredicateRef` target the durable family:

```text
physmath/relation/1
```

---

## F. ComparisonOperator encoding — BLOCKER

This is part of the Relation issue but deserves a separate explicit lock.

### Decision

The closed vocabulary is:

```text
EQ
NEQ
LT
LTE
GT
GTE
```

Canonical representation:

```text
Enum = String(token)
```

Example:

```text
EQ
-> UInt32BE(2) || UTF8("EQ")
```

---

# 3. Transformation rules and derivation

## G. Rule parameter bytes — BLOCKER

**Raised by:** DeepSeek, Gemini.

Every registered transformation must have exact parameters.

### Closed Phase-1 PhysMath rules

```text
EXPR-CANON-001
EXPR-SIMPLIFY-001
EXPR-SUBSTITUTE-001
RAT-NORMALIZE-001
DIFFERENTIATE-001
SELECT-BRANCH-001
EXPR-APPLY-001
```

`PROJECT-STRICT-001` remains outside Phase 1.

### Exact parameter schemas

#### EXPR-CANON-001

```text
Parameters = empty byte sequence
```

#### EXPR-SIMPLIFY-001

```text
Parameters = empty byte sequence
```

#### RAT-NORMALIZE-001

```text
Parameters = empty byte sequence
```

#### SELECT-BRANCH-001

```text
Parameters =
    UInt32BE(BranchOrdinal)
```

#### DIFFERENTIATE-001

```text
Parameters =
    Encode(BoundParameterRef)
    || UInt32BE(Order)
```

`Order >= 1`.

Phase-1 implementation scope:

```text
scalar expressions
one target ParameterID
ordinary partial differentiation with respect to that target
```

Covariant differentiation, tensor differentiation, multi-target derivative operators, and physical derivative semantics are deferred.

#### EXPR-SUBSTITUTE-001

The parameter payload is:

```text
SubstitutionCount = UInt32BE(N)

for each sorted entry:
    Encode(BoundParameterRef)
    || EncodeArtifactRef(ReplacementExpression)
```

Entries are a SET sorted by canonical entry bytes.

Rules:

```text
substitution is simultaneous
duplicate targets invalid
replacement must target physmath/expression/1
each BoundParameterRef.DeclaringArtifactRef must appear among the rule inputs
the target ParameterID must be declared by that declaring artifact
matching inside the expression is by ParameterID
display name is never consulted
```

#### EXPR-APPLY-001

This is the missing reference-slice operation.

```text
Parameters = empty byte sequence
```

Input requirements:

```text
Input[0] = Expression containing a root Call
Input[1] = the Function artifact referenced by that Call
```

The rule:

```text
Call(f, e1, ..., en)
    ->
simultaneous substitution of e1 ... en
into f.BodyRef according to f.ParameterIDs
```

Rules:

```text
Function arity must equal Call argument count
parameter substitution is simultaneous
parameter identity is structural
the Function artifact explicitly supplied as input must match Call.FunctionRef
nested Calls are not recursively expanded by the same invocation
a separate EXPR-APPLY-001 step is required for each additional expansion
```

This avoids adding a hidden theorem prover or a second definition-expansion language.

---

## H. Transformation semantics are incomplete — BLOCKER

**Raised by:** Gemini.

The rule names alone are insufficient.

### Decision: exact Phase-1 canonicalization

`EXPR-CANON-001` performs only:

```text
recursive child canonicalization
flatten nested Add
flatten nested Mul
canonical child ordering
arity normalization
```

It does **not** combine algebraically equal terms merely because they are equivalent.

In particular:

```text
x + x
```

does not silently become:

```text
2*x
```

under canonicalization.

### Exact Phase-1 simplification

`EXPR-SIMPLIFY-001` contains the closed structural rules:

```text
Neg(Neg(x)) -> x

Add(..., 0, ...) -> Add(...)
Mul(..., 1, ...) -> Mul(...)

Mul(..., 0, ...) -> Rational(0/1)

Rational arithmetic -> canonical Rational

Pow(x, 1) -> x
```

For:

```text
Pow(x, 0)
```

the rewrite to `1` is permitted only with an explicit `MathCondition` proving:

```text
x != 0
```

Otherwise the operation fails rather than silently assuming the condition.

No general theorem proving occurs.

### RAT-NORMALIZE-001

Input must be a Rational artifact.

It returns the unique reduced rational satisfying:

```text
denominator > 0
gcd(abs(numerator), denominator) = 1
zero -> 0/1
```

### SELECT-BRANCH-001

Input must be a BranchSet.

The branch ordinal is zero-based and must be in range.

The output is exactly the selected Expression artifact.

---

# 4. Artifact references and schemas

## I. Target families for ArtifactRef fields — BLOCKER

**Raised by:** DeepSeek, Gemini, Z.

Every `ArtifactRef` field must be closed.

The normative mapping is:

```text
Function.BodyRef
    -> physmath/expression/1

FunctionApplication.FunctionRef
    -> physmath/function/1

FunctionApplication.Args
    -> physmath/expression/1

Map.DefinitionRef
    -> physmath/definition/1

Tensor.IndexRefs
    -> physmath/index/1

Tensor.ComponentRefs
    -> physmath/expression/1

Index.SpaceRef
    -> physmath/collection/1

Collection.ElementRefs
    -> phase-1 mathematical durable artifacts explicitly admitted by the
       collection contract; MVP equivariance fixture uses
       physmath/expression/1

OperatorApplication.OperatorRef
    -> absent for differentiate; otherwise a registered future operator family

OperatorApplication.Args
    -> physmath/expression/1

Transformation.Inputs
    -> schema-declared producer artifacts

Transformation.Output
    -> schema-declared producer artifact

Equation.RelationRef
    -> physmath/relation/1

Definition.DefinedRef
    -> phase-1 mathematical artifact family declared by the Definition

Definition.DefiningRef
    -> phase-1 mathematical artifact family declared by the Definition

MathCondition.PredicateRef
    -> physmath/relation/1

Derivation.PremiseRefs
    -> explicit mathematical support artifacts or expressions

Derivation.ResultRefs
    -> explicit mathematical artifacts produced by the derivation

Theorem.PremiseRefs
    -> explicit mathematical support artifacts

Theorem.ConclusionRef
    -> physmath/relation/1

Theorem.AssumptionRefs
    -> physmath/condition/1

Theorem.DerivationRef
    -> physmath/derivation/1

PhysicalDefinition.FrameworkRef
    -> <phys-namespace>/framework/1

PhysicalDefinition.MathematicalRefs
    -> PhysMath mathematical artifact families

PhysicalDefinition.AssumptionRefs
    -> <same phys-namespace>/assumption/1

PhysicalDefinition.DefinitionEquation
    -> physmath/equation/1

PhysicalEquation.FrameworkRef
    -> <phys-namespace>/framework/1

PhysicalEquation.EquationRef
    -> physmath/equation/1

PhysicalEquation.AssumptionRefs
    -> <same phys-namespace>/assumption/1

PhysicalEquation.ScopeRefs
    -> framework-owned artifacts explicitly registered by that package

PhysicalTheorem.FrameworkRef
    -> <phys-namespace>/framework/1

PhysicalTheorem.MathematicalTheoremRef
    -> physmath/theorem/1

PhysicalTheorem.AssumptionRefs
    -> <same phys-namespace>/assumption/1

PhysicalTheorem.InterpretationRefs
    -> <same phys-namespace>/interpretation/1

PhysicalInterpretation.FrameworkRef
    -> <phys-namespace>/framework/1

PhysicalInterpretation.MathematicalRefs
    -> PhysMath mathematical artifact families

PhysicalInterpretation.AssumptionRefs
    -> <same phys-namespace>/assumption/1
```

A schema containing an ArtifactRef without one of these declared target rules is not closed.

---

## J. EquationSet ordering — PIN

`EquationSet.EquationRefs` is:

```text
SEQUENCE
```

in Phase 1.

Equation order is semantically meaningful because a paper's equation sequence may represent derivation order.

A set variant is not used in the Phase-1 schema.

---

## K. Tensor dimension ambiguity — PIN

The Phase-1 tensor schema must restore an explicit:

```text
Dimension UInt16
```

field.

Thus:

```text
Tensor {
    Rank          UInt16
    Dimension     UInt16
    IndexRefs     []ArtifactRef
    ComponentRefs []ArtifactRef
}
```

Validation:

```text
len(IndexRefs) = Rank
dense:
len(ComponentRefs) = Dimension^Rank
```

Sparse tensor semantics remain deferred.

---

# 5. Equivariance reference slice

## L. Exact structural representation — BLOCKER

**Raised by:** Gemini.

The reference fixture must use one prescribed representation rather than allowing Function, Map, and FunctionApplication alternatives.

### Decision

The Phase-1 equivariance fixture uses:

```text
Function
    for f, rho_X, rho_Y

Expression/Call
    for applications

Relation
    for the equality

Equation
    for durable equation packaging

Derivation
    for the transformation sequence

Theorem
    for the final theorem
```

`Map` and durable `FunctionApplication` remain valid general Phase-1 artifacts but are **not used in the canonical equivariance fixture**.

The exact mathematical expression is:

```text
f(rho_X(g,x)) = rho_Y(g,f(x))
```

Represented as:

```text
Call(f,
    Call(rho_X, g, x)
)
```

and:

```text
Call(rho_Y,
    g,
    Call(f, x)
)
```

wrapped in the durable:

```text
physmath/relation/1
```

with:

```text
Operator = EQ
Left     = first expression
Right    = second expression
```

The corresponding `Equation` references that relation.

The final `Theorem` references:

```text
PremiseRefs
ConclusionRef
AssumptionRefs
DerivationRef
```

The reference slice does not introduce a `GroupAction` sort.

`G`, `X`, and `Y` remain mathematical/framework-specific objects represented through ordinary PhysMath structures and/or `phys-*` definitions. No new core sort is invented.

---

# 6. Definition application and substitution

## M. Missing definition application — BLOCKER

**Raised by:** Gemini, Z, Qwen.

### Decision

Add:

```text
EXPR-APPLY-001
```

to the Phase-1 PhysMath RuleID registry.

This is preferable to making a coding agent encode function expansion indirectly through arbitrary substitution steps.

The reference derivation can therefore explicitly show:

```text
Call
  |
  | EXPR-APPLY-001
  v
expanded Expression
```

The operation is deterministic and does not prove anything beyond the declared Function definition.

No `DEFINITION-EXPAND-001` is needed.

---

## N. Parameter substitution context — BLOCKER

**Raised by:** Gemini.

### Decision

A substitution step that consumes a `BoundParameterRef` must include its `DeclaringArtifactRef` among its explicit input artifacts.

Therefore the agent never has to guess which declaration context owns a `ParameterID`.

Inside the referenced Function body:

```text
Symbol.ParameterID == BoundParameterRef.ParameterID
```

is the matching rule.

Outside a known declaration context:

```text
ambiguous ParameterID -> reject
```

Display names are never used as a fallback.

---

# 7. Rule namespace

## O. RuleID scope — BLOCKER

**Raised by:** Qwen.

The Phase-1 spec correctly wants a closed PhysMath rule vocabulary, but the architecture also permits framework-local physical derivation rules.

### Decision

Rule identity is the pair:

```text
(ProducerNamespace, RuleID)
```

For core PhysMath operations:

```text
ProducerNamespace = physmath
```

For a framework-local operation:

```text
ProducerNamespace = phys-gr
```

or another registered framework namespace.

A framework-local RuleID cannot collide with a PhysMath RuleID.

The durable `Transformation` and `DerivationStep` representations must carry the producer namespace needed to resolve the RuleID.

The Phase-1 `physmath` registry itself remains closed.

No dynamic PhysMath AST extension follows from local rule registration.

---

# 8. phys-* architecture

## P. Exact physical artifact families — BLOCKER

**Raised by:** DeepSeek, Gemini, Z.

Every `phys-*` package must use:

```text
<namespace>/framework/1
<namespace>/assumption/1
<namespace>/definition/1
<namespace>/equation/1
<namespace>/theorem/1
<namespace>/interpretation/1
```

The package owns the semantics of those records.

### Minimum-assumption rule

A base framework package contains only assumptions necessary to define the base framework.

A specialized framework may depend on the base framework:

```text
phys-qm
   ↑
phys-bohm
```

but not the reverse.

Interpretation-specific assumptions MUST NOT be placed in the base framework merely because one implementation currently uses them.

This is the structural implementation of:

```text
hundred physicists -> hundred legitimate answers
```

A base framework is reusable by multiple interpretations.

---

## Q. PhysicalInterpretation vs PhysicalContract — RESOLVE BY SIMPLIFICATION

The reviews repeatedly notice overlapping physical metadata.

### Decision

Do **not** introduce another Phase-1 `PhysicalContract` system.

Phase 1 uses:

```text
PhysicalDefinition
PhysicalEquation
PhysicalTheorem
PhysicalInterpretation
PhysicalAssumption
```

The physical package's executable validation logic remains package-local Go code.

No generic compatibility engine is introduced.

This keeps Phase 1 focused on representation rather than a second trust/contract framework.

---

## R. Exact phys-* canonical serialization — BLOCKER

**Raised by:** Gemini, Qwen.

All durable `phys-*` artifacts use:

```text
fixed field order
String / Bytes / UInt* / Optional / SET / SEQUENCE
exact collection semantics
exact UTF-8
```

and the master ArtifactRef digest framing.

Each physical artifact family must have:

```text
canonical serialization vector
expected ArtifactRef
negative vector
```

No ad-hoc JSON serialization is allowed as canonical identity.

---

# 9. Findings explicitly rejected as Phase-1 blockers

These recommendations should **not** be applied.

## S. Frozen Phys Kind / Dimension / AssumptionSet integration — REJECT

**Raised by:** Qwen.

The proposed addition of:

```text
TargetPhysKind
TargetPhysDimension
TargetPhysAssumptions
```

is not a Phase-1 requirement.

The current Phase-1 contract deliberately does not implement the TOE/frozen-Phys formalization pipeline.

The reference target is representation of an equivariance derivation, not construction of a trusted Phys object.

Therefore:

```text
PhysMath representation
    !=
Phys trust admission
```

The proposed `hypothesis.NewCandidateConcept` bridge belongs to a later integration specification.

Do not contaminate Phase-1 with it.

---

## T. Binary layouts for TOE-Agent research artifacts — REJECT

**Raised by:** Qwen.

TOE-Agent is explicitly deferred.

No Phase-1 implementation should define:

```text
ResearchHypothesis
CorrespondenceProposal
FormalizationProposal
FormalizationResponse
```

or their canonical serialization.

Those belong to the later research/integration specification.

---

## U. Global trust/provenance/corpus machinery — REJECT

The reviewers' concerns about trust are valid in the larger system, but they are intentionally outside this Phase-1 contract.

Phase 1 carries only the references necessary to represent mathematical and framework relationships.

Scientific truth, corpus admission, human approval, and provenance policy remain deferred.

---

## V. Merging PhysicalInterpretation into a PhysicalContract — REJECT

The semantic meaning of a framework artifact and the operational validity rules of a generic contract system are different concerns.

However, because Phase 1 does not need a generic PhysicalContract engine at all, the correct action is **simplification**, not merging.

---

# 10. Residual clarifications that become explicit locks

These are not separate architectural redesigns; they are one-line closures.

### Theorem completeness

In Phase 1:

```text
Theorem.DerivationRef is mandatory.
```

A theorem with an asserted derivational basis cannot omit the derivation.

`DerivationObligation` is deferred.

---

### DerivationStep order

`Derivation.Steps` list order is authoritative.

`Ordinal` must equal the zero-based list position.

No independent ordering interpretation exists.

---

### Transformation result

Every PhysMath operation returns:

```text
OperationResult {
    OutputRef
    ConditionRefs SET
}
```

The result type is in-memory only.

No hidden condition discharge.

---

### PhysMath vs phys-* mathematical structures

A framework-local mathematical type may remain local when the existing PhysMath vocabulary is insufficient.

The mapping to PhysMath must be explicit and deterministic.

No dynamic AST extension.

---

### Representation does not imply equivalence

Two representations may be structurally different while being mathematically equivalent.

PhysMath identity remains canonical-payload identity.

Theorem equivalence is not automatic normalization.

---

# 11. Consolidated implementation target

The coding agent should now have exactly this vertical slice:

```text
Physics paper
    |
    | analysis / interpretation by humans or AI
    v
formal mathematical content
    |
    v
PhysMath
    |
    +-- Function f
    +-- Function rho_X
    +-- Function rho_Y
    +-- parameters g, x
    +-- Call expressions
    +-- Relation / Equation
    +-- Conditions
    +-- Transformation rules
    +-- Derivation
    +-- Theorem
    |
    v
phys-* framework
    |
    +-- framework identity
    +-- minimum physical assumptions
    +-- physical definitions
    +-- physical equation
    +-- physical theorem
    +-- physical interpretation
    |
    v
complete equivariance-theorem representation
```

Then the second fixture:

```text
same PhysMath representation
        |
        +---- phys-qm
        |
        +---- interpretation-specific phys-* package
```

must demonstrate:

```text
PhysMath artifact identity unchanged
framework identities distinct
assumption sets distinct
physical interpretations distinct
no global winner
no physical contamination of PhysMath
```

---

# 12. Final consolidated blocker count

After deduplication across the four reviews:

```text
TRUE BLOCKERS

1.  Missing artifact-family registry
2.  Missing BigInt encoding
3.  Missing ConstantID encoding
4.  Missing closed-enum encoding
5.  Incomplete ParameterID scope rules
6.  Undefined BoundParameterRef encoding
7.  Undefined durable Relation payload
8.  Undefined ArtifactRef target families
9.  Incomplete transformation parameter schemas
10. Missing exact transformation rewrite semantics
11. Missing definition/application rule
12. Ambiguous substitution context/matching
13. Ambiguous framework-local RuleID namespace
14. Ambiguous canonical equivariance artifact graph
15. Missing exact phys-* family registration/serialization
```

These are all mechanical and finite.

The Qwen “Frozen Phys kind/dimension/assumption” finding and all TOE-Agent/trust/provenance formalization findings are explicitly **not Phase-1 blockers**.

---

# 13. Closure condition

The specification can be declared implementation-ready when:

```text
[ ] master artifact-family registry is complete
[ ] every primitive has one canonical encoding
[ ] every enum has one canonical encoding
[ ] ParameterID generation is deterministic
[ ] BoundParameterRef encoding is pinned
[ ] Relation node and durable relation payload are pinned
[ ] every ArtifactRef field has a target family
[ ] every Phase-1 RuleID has exact parameter bytes
[ ] every Phase-1 RuleID has exact semantics
[ ] EXPR-APPLY-001 exists
[ ] substitution matching is deterministic
[ ] framework-local RuleIDs are namespace-qualified
[ ] equivariance fixture has exactly one normative artifact graph
[ ] phys-* families are registered
[ ] phys-* serialization is canonical
[ ] minimum-assumption/specialization rule is explicit
[ ] Phase-2 systems remain excluded
[ ] all golden vectors exist
[ ] positive and negative tests exist
[ ] four-reviewer blocker audit reports zero demonstrated blockers
```

---

## Final verdict

**The strategic architecture is CLOSED. The representation contract is not yet closed, but the remaining closure work is finite and mechanical.**

Do **not** start another architecture redesign.

The next amendment should incorporate the locks above into `PM-REP-001`, then the project should immediately move to the implementation plan.

The key architectural principle remains:

```text
PhysMath
    = mathematical representation

phys-* base framework
    = minimum assumptions needed to define that framework

specialized phys-*
    = framework + additional interpretation/model/ontology assumptions

Analysis + Interpretation
    = humans + AI agents

Representation
    = PhysMath + phys-*

Numerical + TOE-Agent + provenance/trust
    = later
```

That is the smallest architecture that can realistically get the equivariance implementation started without sacrificing the “hundred physicists, hundred answers” principle.
