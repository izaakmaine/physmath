# PhysMath + phys-* Representation Specification
## Phase-1 Implementation Baseline

**Document ID:** PM-REP-001  
**Version:** 1.0  
**Status:** **IMPLEMENTATION BASELINE — TOP PRIORITY**  
**Role:** Normative representation specification for PhysMath and `phys-*` external physics libraries.  
**Supersedes:** The Phase-1 implementation-relevant portions of the cross-project architecture where they conflict with this document.  
**Secondary systems:** TOE-Agent, Numerical, and research/provenance/trust workflow are explicitly deferred and MUST NOT block Phase-1 implementation.

> **Primary purpose.** Define the smallest coherent representation architecture needed to implement PhysMath and `phys-*` libraries around a nontrivial physics-paper outcome: the representation of equations, assumptions, definitions, mathematical transformations, physical interpretation, and a complete derivation of an equivariance theorem.

> **Central rule.** PhysMath represents mathematics. `phys-*` represents physical meaning, assumptions, framework-specific definitions, equations, and theorem/derivation context. Neither layer silently answers the other's question.

---

# 0. Scope and authority

## 0.1 Frozen Phys authority

The existing Phys project remains authoritative for its existing semantics:

```text
Module:              github.com/PithomLabs/phys
Specification:       specs_v2_3.md
Operational freeze:  exact frozen repository state supplied to implementation
```

This specification MUST NOT modify the Phys kernel, dependency graph, or public API.

The frozen Phys authority wins whenever a statement about existing Phys behavior conflicts with this specification. Such a conflict is an architecture issue and MUST NOT be resolved by changing Phys during Phase 1.

## 0.2 Normative language

**MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, and **SHALL NOT** are normative.  
**SHOULD**, **SHOULD NOT**, and **MAY** are advisory.

## 0.3 Phase-1 scope

Phase 1 implements only:

```text
phys-artifact
PhysMath
phys-* external physical-domain libraries
```

The implementation target is representation, not the complete research platform.

## 0.4 Explicitly deferred

The following are separate projects and MUST NOT block Phase-1 implementation:

```text
TOE-Agent research orchestration
Numerical execution
ComputationalContract execution
research lifecycle state machines
formalization workflow
trust/corpus promotion workflow
scientific adjudication
paper-management workflow
human/AI agent orchestration
```

A Phase-1 implementation MUST NOT introduce these concerns merely because a future project will eventually need them.

---

# 1. Research-to-representation boundary

## 1.1 Physics paper is the unit of research

A physics paper is the normal unit of research context.

A paper may contain:

```text
motivation
analysis
hypotheses
interpretation
experimental discussion
prose
mathematical definitions
assumptions
equations
derivations
theorems
```

The Phase-1 representation substrate does **not** model the whole paper.

The representation target is the paper's formal mathematical/physical payload:

```text
definitions
assumptions
mathematical objects
equations
transformations
derivation steps
theorem statements
```

The implementation MUST permit a human or AI research process to extract those objects from a paper without requiring the paper itself to become a PhysMath ontology object.

## 1.2 Analysis / Representation / Interpretation

The Phase-1 boundary is:

```text
ANALYSIS
    humans + AI agents determine what a paper appears to say
    and what hypotheses/theorems may be worth representing

REPRESENTATION
    PhysMath + phys-* encode the resulting mathematical and
    framework-specific structures deterministically

INTERPRETATION
    phys-* declares what physical meaning a framework assigns
    to represented mathematical structures
```

Analysis remains outside PhysMath semantics.

Interpretation is explicit in `phys-*`; it is never inferred from names or mathematical shape.

## 1.3 Equation-first rule

A paper's formal output is represented primarily through:

```text
Equation
EquationSet
Definition
Assumption
Derivation
Theorem
```

Prose may be attached as descriptive metadata by a consuming research system, but prose MUST NOT be required to reconstruct mathematical identity.

---

# 2. Constitutional architecture

## 2.1 PhysMath

PhysMath owns theory-neutral mathematics:

```text
mathematical types
mathematical objects
expressions
functions
maps
operators
relations/equations
conditions
transformations
derivations as mathematical structure
canonicalization
mathematical equality
canonical serialization
```

PhysMath MUST NOT own:

```text
physical interpretation
physical theory names
physical assumptions
experimental truth
research status
scientific preference
paper provenance
trust/corpus status
```

## 2.2 phys-* libraries

A `phys-*` library represents one specific physical framework/domain.

It owns:

```text
physical types
physical assumptions
framework vocabulary
framework-specific definitions
framework-specific equations
framework-specific theorem statements
framework-specific derivation rules
framework-specific interpretation of mathematical objects
framework-specific mappings into PhysMath
```

A `phys-*` library MAY define a local mathematical algebra when PhysMath does not yet contain the required abstraction, but that algebra remains package-local and MUST NOT dynamically extend the PhysMath AST.

## 2.3 Many answers are legal

The architecture deliberately permits:

```text
same PhysMath object
    -> multiple phys-* interpretations
```

No global mechanism chooses one physical interpretation as universally correct.

A consumer that requires physical meaning MUST identify the framework explicitly.

## 2.4 Representation is not truth

The following are representation successes, not scientific truth claims:

```text
canonicalization succeeds
serialization succeeds
derivation replays
equation matches
theorem representation validates
projection succeeds
```

A `phys-*` library represents a framework's claims and rules. It does not prove that the framework is empirically correct.

---

# 3. Dependency boundary

The Phase-1 production dependency graph is:

```text
PhysMath
    -> phys-artifact

phys-*
    -> PhysMath
    -> phys-artifact
    -> Phys (existing public API only, when required)

Phys
    -> no Phase-1 dependency changes
```

There is no dependency from PhysMath to any `phys-*` package.

A `phys-*` package MUST NOT import another `phys-*` package solely to borrow physical meaning. A cross-framework dependency requires an explicit framework contract and is outside the reference implementation.

`phys-artifact` is mechanical infrastructure only.

It MUST NOT own:

```text
physics ontology
math semantics
interpretation
trust
research state
```

---

# 4. Artifact and identity model

## 4.1 Typed values versus durable references

In-memory composition uses concrete typed values:

```text
physmath.Expression
physmath.Function
physmath.Relation
phys-gr.<framework type>
```

A durable cross-object relationship uses `ArtifactRef`.

```text
typed value != ArtifactRef
```

## 4.2 ArtifactRef

The canonical Phase-1 type is:

```go
type ArtifactRef struct {
    Namespace     string
    SchemaVersion string
    ArtifactType  string
    Digest        [32]byte
}
```

Field order is normative.

Identity is exactly:

```text
(Namespace, SchemaVersion, ArtifactType, Digest)
```

The digest alone is not sufficient to identify an artifact family instance.

## 4.3 Canonical digest

For canonical payload `P`:

```text
Header =
    Len32(Namespace UTF-8 bytes)
    || Namespace UTF-8 bytes
    || Len32(SchemaVersion UTF-8 bytes)
    || SchemaVersion UTF-8 bytes
    || Len32(ArtifactType UTF-8 bytes)
    || ArtifactType UTF-8 bytes

Digest = SHA-256(Header || P)
```

There is exactly one Phase-1 artifact digest formula.

## 4.4 Embedded ArtifactRef

An embedded `ArtifactRef` is encoded as:

```text
Len32(Namespace UTF-8 bytes)
|| Namespace UTF-8 bytes
|| Len32(SchemaVersion UTF-8 bytes)
|| SchemaVersion UTF-8 bytes
|| Len32(ArtifactType UTF-8 bytes)
|| ArtifactType UTF-8 bytes
|| Digest[32]
```

No JSON, hexadecimal text, pointer value, or bare digest is canonical embedded-ref material.

## 4.5 Primitive encoding

```text
String(T)      = UInt32BE(byte_length(T)) || UTF-8(T)
Bytes(B)       = UInt32BE(len(B)) || B
UInt8          = 1 byte
UInt16         = 2-byte unsigned big-endian
UInt32         = 4-byte unsigned big-endian
UInt64         = 8-byte unsigned big-endian
Bool           = 0x00 or 0x01
Digest32       = exactly 32 bytes
Optional(T)    = presence byte || T when present
Sequence(T)    = UInt32 count || elements in declared order
Set(T)         = UInt32 count || elements sorted by canonical element bytes
Enum           = canonical String of a closed token
TaggedUnion    = UInt8 discriminator || selected variant payload
```

A `Set` MUST reject duplicate canonical elements.

## 4.6 Floating-point boundary

`Float64Bits` is not permitted in exact PhysMath mathematical identity payloads.

It MAY be used by future numerical schemas.

Phase 1 MUST NOT introduce floating-point approximation into mathematical canonical identity.

---

# 5. Namespace and artifact-family rules

## 5.1 Namespaces

Phase-1 reserved namespaces:

```text
physmath
phys
phys-<domain>
```

Examples:

```text
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
```

`phys-artifact` has no semantic namespace.

## 5.2 Artifact-type form

Artifact types are lowercase path segments separated by `/`.

Examples:

```text
expression
function
relation
equation
assumption
derivation
theorem
framework
physical_object
physical_equation
physical_theorem
```

Artifact types MUST be explicitly registered in the owning package contract.

No implementation may invent an unregistered durable artifact type.

## 5.3 Schema versions

Phase-1 artifact schemas use:

```text
SchemaVersion = "1"
```

A changed canonical payload schema is a new versioned schema. Existing artifacts remain immutable.

---

# 6. PhysMath mathematical IR

## 6.1 Closed core sort inventory

The Phase-1 architecture recognizes the following mathematical families:

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
Collection
Transformation
Limit
AsymptoticTruncation
DerivationObligation
```

This list is closed for PhysMath schema version `1`.

A new sort requires a new versioned specification. A `phys-*` package MUST NOT inject a runtime sort into PhysMath.

## 6.2 Implementation priority within the sort set

The reference implementation MUST first implement the subset required for the equivariance vertical slice:

```text
Expression
Function
FunctionApplication
Map
OperatorApplication
Relation
Transformation
Tensor
Index
Collection
```

The remaining families remain part of the schema vocabulary but may be `DEFERRED + CONTAINMENT` until a concrete Phase-1 reference slice requires them:

```text
Functional
FunctionalApplication
Operator
Limit
AsymptoticTruncation
DerivationObligation
```

A deferred sort may be stored and validated but MUST NOT silently acquire unimplemented operational semantics.

---

# 7. Expression representation

## 7.1 Closed node set

The Phase-1 Expression tagged union is exactly:

```text
0x01 Symbol
0x02 Rational
0x03 Add
0x04 Mul
0x05 Neg
0x06 Pow
0x07 Sqrt
0x08 Call
0x09 Relation
0x0A BranchSet
```

No other node tag is legal in schema version `1`.

## 7.2 Recursive payload rule

Every child expression is encoded recursively as:

```text
UInt8(NodeTag) || node-specific fields
```

No pointer, runtime type name, JSON representation, or Go memory layout participates in canonical bytes.

## 7.3 Symbol

```text
Symbol {
    DisplayName string
    ParameterID Optional<ParameterID>
    ConstantID  Optional<ConstantID>
}
```

Validation:

```text
durable symbol:
    exactly one of ParameterID or ConstantID is present

ephemeral unbound symbol:
    both are absent

both present:
    always invalid
```

`DisplayName` is presentation only.

PhysMath MUST NOT inspect display names to assign physical meaning.

## 7.4 Rational

```text
Rational {
    Numerator   BigInt
    Denominator BigInt
}
```

The denominator is positive, the fraction is reduced, and zero is represented as `0/1`.

## 7.5 Add / Mul

```text
Add {
    Children []Expression // SEQUENCE before canonical sorting; canonical storage order deterministic
}

Mul {
    Children []Expression // SEQUENCE before canonical sorting; canonical storage order deterministic
}
```

Canonicalization:

```text
canonicalize children recursively
flatten nested same-kind nodes
sort children by canonical encoded child bytes
```

Arity normalization:

```text
Add([])  -> Rational(0/1)
Mul([])  -> Rational(1/1)
Add([x]) -> x
Mul([x]) -> x
```

## 7.6 Neg

```text
Neg {
    Operand Expression
}
```

Construction preserves the node.

`EXPR-SIMPLIFY-001` may apply:

```text
Neg(Neg(x)) -> x
```

## 7.7 Pow

```text
Pow {
    Base     Expression
    Exponent Expression
}
```

PhysMath v1 permits an arbitrary Expression exponent.

A particular `phys-*` framework MAY impose a stricter exponent rule in its own contract.

## 7.8 Sqrt

```text
Sqrt {
    Radicand Expression
}
```

`Sqrt` is a distinct mathematical node. A future normalization may relate it to `Pow`, but Phase-1 canonical identity preserves the explicit node kind.

## 7.9 Call

```text
Call {
    FunctionRef ArtifactRef
    Args        []Expression // SEQUENCE
}
```

`FunctionRef` MUST identify `physmath/function/1`.

Argument order is semantic.

## 7.10 Relation

An Expression-level relation is:

```text
Relation {
    Operator ComparisonOperator
    Left     Expression
    Right    Expression
}
```

Closed comparison vocabulary:

```text
EQ
NEQ
LT
LTE
GT
GTE
```

A relation is a mathematical proposition. It is not a trust assertion.

## 7.11 BranchSet

```text
BranchSet {
    Branches []Expression // SEQUENCE
}
```

Branch order is semantic until an explicit selection transformation occurs.

No branch is selected by name or physical convention.

---

# 8. Parameters and mathematical identity

## 8.1 ParameterID

```go
type ParameterID [32]byte
```

`ParameterID` identifies a mathematical binding within its declaration context.

It is not a global provenance identifier.

## 8.2 Scope path

A parameter declaration is identified by:

```text
ScopePath = ordered lexical binder ordinals from artifact root to declaration scope
ParameterOrdinal = zero-based declaration position within that scope
```

`ScopePathBytes`:

```text
UInt32BE(depth)
|| UInt32BE(scope_0)
|| ...
|| UInt32BE(scope_(depth-1))
```

The outer digest input uses the byte length of those bytes, not the element count a second time.

## 8.3 ParameterID formula

Phase-1 formula:

```text
ParameterID = SHA-256(
    String("physmath")
    || Bytes(ScopePathBytes)
    || UInt32BE(ParameterOrdinal)
)
```

`DisplayName` is excluded.

An artifact's own `ArtifactRef` is excluded. This prevents recursive identity dependence.

## 8.4 Alpha-renaming

Changing:

```text
t -> tau
```

MUST NOT alter ParameterID when the declaration position is unchanged.

## 8.5 Cross-artifact binding

When a parameter is referenced from another durable artifact:

```text
BoundParameterRef {
    DeclaringArtifactRef ArtifactRef
    ParameterID         ParameterID
}
```

The declaration artifact provides the scope context; ParameterID provides the structural parameter identity within that declaration.

A bare ParameterID is insufficient when multiple declaring artifacts are in scope.

## 8.6 t/k rule

PhysMath does not contain physical roles such as:

```text
TIME
RG_SCALE
```

The strings `t` and `k` have no physical significance inside PhysMath.

If a framework declares:

```text
parameter X = physical time
parameter Y = renormalization-group scale
```

that declaration belongs in `phys-*`.

---

# 9. Mathematical constants

## 9.1 ConstantID

Mathematical constants use explicit identities.

Phase-1 built-ins are:

```text
math.const.pi
math.const.e
math.const.i
```

No physical constant such as `c`, `G`, or `hbar` is a PhysMath built-in semantic constant merely because it is common in physics.

## 9.2 Physical constants

A physical constant is represented initially as a mathematical symbol and acquires physical meaning only through a `phys-*` contract.

The same mathematical symbol may therefore have different physical interpretations under different frameworks.

---

# 10. Higher mathematical representations

## 10.1 Function

```text
Function {
    ParameterIDs []ParameterID // SEQUENCE
    BodyRef      ArtifactRef
}
```

`BodyRef` MUST target `physmath/expression/1`.

Parameter order is semantic.

## 10.2 FunctionApplication

```text
FunctionApplication {
    FunctionRef ArtifactRef
    Args        []ArtifactRef // SEQUENCE
}
```

`FunctionRef` MUST target `physmath/function/1`.

## 10.3 Map

```text
Map {
    DomainSort    string
    CodomainSort  string
    DefinitionRef ArtifactRef
}
```

`DefinitionRef` MUST identify an explicit mathematical definition artifact.

## 10.4 Tensor

```text
Tensor {
    Rank          UInt16
    IndexRefs     []ArtifactRef   // SEQUENCE, slot order is semantic
    ComponentRefs []ArtifactRef   // SEQUENCE, canonical row-major order
}
```

For a dense tensor:

```text
len(ComponentRefs) = Dimension^Rank
```

where dimension is provided by the owning tensor/index-space contract.

General sparse tensor semantics are deferred.

## 10.5 Index

```text
Index {
    Label     string
    Variance  IndexVariance
    SpaceRef  Optional<ArtifactRef>
}
```

Closed v1 variance:

```text
COVARIANT
CONTRAVARIANT
UNSPECIFIED
```

Variance remains mathematical structure. It does not itself choose a physical metric or sign convention.

## 10.6 Collection

```text
Collection {
    ElementRefs []ArtifactRef // SET
}
```

Duplicate canonical elements are invalid.

## 10.7 OperatorApplication

```text
OperatorApplication {
    OperatorID  string
    OperatorRef Optional<ArtifactRef>
    Args        []ArtifactRef // SEQUENCE
    Parameters  Bytes
}
```

The Phase-1 reserved operator identity is:

```text
differentiate
```

For `differentiate`:

```text
OperatorRef = absent
Parameters =
    Encode(BoundParameterRef)
    || UInt32BE(Order)
```

Repeated differentiation MUST normalize to one `differentiate` application with summed order wherever the rule applies.

## 10.8 Transformation

```text
Transformation {
    RuleID     TransformationRuleID
    Inputs     []ArtifactRef // SEQUENCE
    Output     ArtifactRef
    Parameters Bytes
}
```

Transformation records a mathematical operation. It is not itself a proof of physical validity.

---

# 11. Mathematical support artifacts

These are durable artifacts, not new mathematical sorts.

## 11.1 Equation

An equation artifact packages a relation for durable reference:

```text
Equation {
    RelationRef ArtifactRef
}
```

`RelationRef` MUST target `physmath/relation/1`.

An Equation represents a mathematical equality or other explicit relation; it does not assert physical truth.

## 11.2 EquationSet

```text
EquationSet {
    EquationRefs []ArtifactRef // SEQUENCE when used as a derivation sequence; SET only when order is explicitly irrelevant
}
```

For paper-level formal output, equation order MUST be preserved when the order carries derivation meaning.

## 11.3 Definition

```text
Definition {
    DefinedRef  ArtifactRef
    DefiningRef ArtifactRef
}
```

A definition is a mathematical declaration that one represented object is defined by another represented object.

## 11.4 MathCondition

```text
MathCondition {
    PredicateRef ArtifactRef
}
```

Family:

```text
physmath/condition/1
```

`PredicateRef` MUST target `physmath/relation/1`.

MathCondition contains no physical interpretation or trust metadata.

## 11.5 TransformationRuleID

The Phase-1 closed operation vocabulary is:

```text
EXPR-CANON-001
EXPR-SIMPLIFY-001
EXPR-SUBSTITUTE-001
RAT-NORMALIZE-001
DIFFERENTIATE-001
SELECT-BRANCH-001
```

`PROJECT-STRICT-001` is deliberately outside the Phase-1 core representation set; projection into frozen Phys is a separate integration concern.

No unregistered RuleID is valid.

---

# 12. Derivation representation

## 12.1 Purpose

A `Derivation` represents how a mathematical result was obtained from explicitly identified premises and operations.

It is not a universal proof assistant.

It records representation-level derivation structure sufficient for deterministic replay of the operations that the owning implementation supports.

## 12.2 DerivationStep

```text
DerivationStep {
    Ordinal       UInt32
    RuleID        TransformationRuleID
    InputRefs     []ArtifactRef
    OutputRef     ArtifactRef
    ConditionRefs []ArtifactRef
    Parameters    Bytes
}
```

Rules:

```text
Ordinal      = unique sequential position starting at 0
InputRefs    = SEQUENCE
ConditionRefs = SET
Parameters   = exact canonical rule parameters
```

A step may refer only to:

```text
an imported prerequisite artifact
or an output from an earlier step
```

unless the rule contract explicitly permits an external reference.

## 12.3 Derivation

```text
Derivation {
    PremiseRefs []ArtifactRef // SET
    Steps       []DerivationStep // SEQUENCE
    ResultRefs  []ArtifactRef // SET
}
```

Family:

```text
physmath/derivation/1
```

`Steps` preserve order.

`ResultRefs` identify the derivation result(s).

## 12.4 Derivation validity

A derivation is structurally valid when:

```text
all references resolve
step ordinals are contiguous and unique
all step rule IDs are registered
all step parameters satisfy their rule schema
all same-run inputs are available
all generated conditions are explicit
final ResultRefs are produced by a step or declared premise
```

Structural validity MUST NOT be interpreted as scientific truth.

## 12.5 No hidden inference

A derivation implementation MUST NOT silently insert:

```text
unstated assumption
unstated equation
unstated theorem
physical interpretation
```

If an operation requires a condition, the condition MUST appear explicitly in `ConditionRefs` or the operation MUST fail.

---

# 13. Theorem representation

## 13.1 Theorem

A theorem artifact represents a formal mathematical theorem statement plus its derivation representation.

```text
Theorem {
    PremiseRefs    []ArtifactRef // SET
    ConclusionRef  ArtifactRef
    AssumptionRefs []ArtifactRef // SET
    DerivationRef  ArtifactRef
}
```

Family:

```text
physmath/theorem/1
```

## 13.2 Theorem meaning

`Theorem` means:

```text
this artifact represents a theorem claim
```

It does not mean:

```text
the theorem is true in nature
```

A complete theorem representation requires a represented derivation when the theorem claims an explicit derivational basis.

## 13.3 Theorem versus physical theorem

PhysMath theorem:

```text
mathematical statement + mathematical derivation representation
```

`phys-*` physical theorem:

```text
physical assumptions + framework interpretation
+ mathematical theorem/equations
+ framework-specific derivation context
```

The physical theorem remains owned by `phys-*`.

---

# 14. phys-* framework contract

## 14.1 Framework identity

Each `phys-*` package defines a framework artifact:

```text
FrameworkArtifact {
    FrameworkID      string
    PackageNamespace string
    FrameworkVersion string
}
```

Family:

```text
<phys-namespace>/framework/1
```

One framework version has exactly one framework identity artifact.

## 14.2 Physical assumptions

A framework defines package-owned assumptions:

```text
PhysicalAssumption {
    AssumptionID   string
    ConditionRefs  []ArtifactRef // SET
    Category       string
    Description    string
}
```

The architecture imposes no universal physical assumption vocabulary beyond the package contract.

`ConditionRefs` must point to mathematical conditions or other explicitly allowed mathematical support artifacts.

`Category` is a framework-owned semantic label.

It is not a global ontology key.

## 14.3 PhysicalDefinition

A framework may define physical meaning for a mathematical object:

```text
PhysicalDefinition {
    DefinitionID      string
    FrameworkRef       ArtifactRef
    PhysicalType       string
    MathematicalRefs   []ArtifactRef // SET
    AssumptionRefs     []ArtifactRef // SET
    DefinitionEquation Optional<ArtifactRef>
}
```

The framework decides the meaning of `PhysicalType`.

PhysMath does not interpret it.

## 14.4 PhysicalEquation

```text
PhysicalEquation {
    EquationID     string
    FrameworkRef   ArtifactRef
    EquationRef    ArtifactRef
    AssumptionRefs []ArtifactRef // SET
    ScopeRefs      []ArtifactRef // SET
}
```

The mathematical equation remains a PhysMath artifact.

The physical equation adds framework meaning and assumptions.

## 14.5 PhysicalTheorem

```text
PhysicalTheorem {
    TheoremID       string
    FrameworkRef    ArtifactRef
    MathematicalTheoremRef ArtifactRef
    AssumptionRefs  []ArtifactRef // SET
    InterpretationRefs []ArtifactRef // SET
}
```

The derivation itself remains represented by the referenced mathematical theorem/derivation unless the framework requires a distinct framework-specific derivation object.

## 14.6 Physical interpretation

A framework-specific interpretation is explicit:

```text
PhysicalInterpretation {
    InterpretationID string
    FrameworkRef     ArtifactRef
    PhysicalType     string
    MathematicalRefs []ArtifactRef // SET
    AssumptionRefs   []ArtifactRef // SET
}
```

There is no global interpretation registry.

## 14.7 Framework disagreement

Two packages MAY define:

```text
same mathematical expression
same mathematical equation
same theorem structure
```

with different physical meaning or assumptions.

This is conforming behavior, not an identity conflict.

The framework identity is part of the physical artifact's semantic context.

---

# 15. Theory-local algebra rule

## 15.1 When local algebra is permitted

A `phys-*` library MAY introduce a local mathematical type when:

```text
required by the physical framework
AND
not yet expressible using existing PhysMath types
```

## 15.2 Locality

The local type MUST remain inside the package:

```text
phys-foo.LocalExpr
```

It MUST NOT be added dynamically to PhysMath's tagged union.

## 15.3 Mapping to PhysMath

A package MAY define:

```text
LocalExpr
    -> explicit deterministic mapping
    -> PhysMath representation
```

The mapping MUST document:

```text
source type
target type
conditions for mapping
information preserved
information rejected
canonical output
```

No implicit fallback mapping is permitted.

---

# 16. Equivariance theorem reference slice

## 16.1 Why this is the Phase-1 target

The reference implementation target is the representation of an equivariance theorem derivation because it requires the architecture to exercise:

```text
mathematical objects
parameters
functions
function application
relations/equations
conditions
assumptions
transformations
derivation sequencing
theorem representation
physical framework interpretation
```

It is therefore a vertical-slice test of the architecture rather than a special feature of any one physical theory.

## 16.2 Generic equivariance structure

The representation MUST be capable of expressing the generic mathematical form:

```text
Given:
    a group/action parameter g
    a source action rho_X
    a target action rho_Y
    a map f : X -> Y

Equivariance:

    f(rho_X(g, x)) = rho_Y(g, f(x))
```

Equivalent composition form MAY be represented when the selected mathematical constructs support it.

This is a mathematical representation pattern, not a commitment to any particular physical interpretation.

## 16.3 Required represented objects

The reference slice MUST contain explicit artifacts for:

```text
source domain X
source action rho_X
target domain Y
target action rho_Y
map f
parameter g
argument x
left-hand-side expression
right-hand-side expression
equivariance relation
assumptions, when required
theorem statement
derivation
```

The exact physical realization of X, Y, rho_X, rho_Y, and f is selected by the concrete `phys-*` test fixture.

## 16.4 Required derivation shape

The reference slice MUST demonstrate a complete derivation path of the form:

```text
premises / definitions
        |
        v
construct LHS
        |
        v
apply explicit transformation rules
        |
        v
construct RHS
        |
        v
establish equality relation
        |
        v
equivariance theorem
```

The derivation MAY branch and rejoin. A single linear chain is not required.

## 16.5 Framework interpretation

A `phys-*` fixture MUST add at least:

```text
one framework artifact
at least one physical assumption
at least one physical definition or interpretation
one physical theorem object referencing the mathematical theorem
```

The fixture MUST NOT modify PhysMath to encode the physical interpretation.

## 16.6 Required ambiguity test

The implementation suite MUST include two framework fixtures that interpret the same or overlapping mathematical representation differently.

Expected result:

```text
PhysMath identity remains unchanged
framework artifacts remain distinct
physical interpretations remain distinct
no global winner is inferred
```

This is the concrete test for the “many physicists, many answers” principle.

---

# 17. Operation semantics for the reference slice

## 17.1 Canonical operations

The reference implementation MUST support at least:

```text
canonicalize Expression
simplify Expression
substitute parameters
normalize rationals
differentiate a supported Expression
select a BranchSet branch
record a derivation step
```

## 17.2 Differentiate

The closed mathematical derivative rules required by the reference slice are:

```text
d(Rational)/dp = 0

d(Symbol(target))/dp = 1

d(Symbol(other parameter))/dp = 0

d(Add(xs))/dp = Add(d(x1), ..., d(xn))
d(Mul(xs))/dp = sum_i(product_except_i(x) * d(x_i))
d(Neg(x))/dp = Neg(d(x))
d(Pow(b,n))/dp = n * Pow(b,n-1) * d(b)
```

The `Pow` rule applies only where `n` is a nonnegative integer Rational and all required side conditions are satisfied.

`Call`, `Relation`, and `BranchSet` differentiation is out of scope unless a concrete `phys-*` implementation defines an explicit mathematical extension outside the Phase-1 core rule set.

## 17.3 Condition generation

If an operation requires a mathematical condition:

```text
operation result
    -> explicit MathCondition
    -> ConditionRefs
```

The implementation MUST NOT silently discharge the condition.

## 17.4 Simultaneous substitution

Substitution is simultaneous.

A substitution parameter set contains:

```text
Target BoundParameterRef
Replacement ArtifactRef -> physmath/expression/1
```

Duplicate targets are invalid.

---

# 18. Canonicalization and deterministic behavior

## 18.1 Canonical child order

For commutative mathematical collections such as `Add`, `Mul`, and declared sets:

```text
sort by lexicographic canonical encoded bytes
```

No runtime map iteration order may affect identity.

## 18.2 Sequence semantics

A `SEQUENCE` is used when order itself is meaningful.

Examples:

```text
function parameters
function arguments
tensor index slots
derivation steps
branch alternatives
operation inputs when positional meaning exists
```

## 18.3 Set semantics

A `SET` is used when order carries no mathematical or framework meaning.

Examples:

```text
assumption references
premises
physical interpretation references
framework-independent evidence references
```

The schema MUST explicitly state `SET` or `SEQUENCE` for every collection field.

## 18.4 Canonicalization is not theorem proving

Canonicalization may make structurally equivalent mathematical representations identical.

It MUST NOT solve arbitrary equations, prove theorems, infer physical meaning, or invent assumptions.

---

# 19. Equality rules

## 19.1 Structural equality

Canonical mathematical equality means:

```text
canonical payload bytes are identical
```

for the same mathematical artifact family and schema.

## 19.2 Physical equivalence

Physical equivalence is framework-specific.

PhysMath MUST NOT define:

```text
GR equivalence
QM interpretive equivalence
Bohmian equivalence
```

## 19.3 Theorem equality

Two theorem artifacts are equal only when their canonical theorem payloads are equal.

Two mathematically equivalent theorems may nevertheless be different artifacts if they use different canonical theorem structures; equivalence reasoning is not automatically identity normalization.

---

# 20. Serialization and replay

## 20.1 Canonical serialization

Every durable Phase-1 artifact MUST have:

```text
fixed field order
explicit lengths
explicit optional presence bytes
explicit sequence/set semantics
exact integer widths
UTF-8 for strings
closed enum values
```

## 20.2 Transport

JSON MAY be used for debugging or interchange.

Transport JSON MUST NOT be used as canonical identity material unless a future schema explicitly declares it canonical.

## 20.3 Replay

A mathematical transformation can be replayed by the authority that owns its RuleID.

For example:

```text
PhysMath RuleID -> PhysMath replay
phys-gr local RuleID -> phys-gr replay
```

Replay authority is local.

A consumer MUST NOT claim to replay an unknown framework operation solely because it can resolve its output ArtifactRef.

## 20.4 Golden vectors

Every durable Phase-1 artifact family MUST provide:

```text
one canonical serialization vector
one expected ArtifactRef digest
one invalid/negative vector
```

The equivariance reference fixture MUST additionally provide golden vectors for:

```text
the representative mathematical equations
the final theorem
the derivation
at least one physical framework interpretation
```

---

# 21. phys-* conformance contract

Every `phys-*` package implementing this specification MUST demonstrate:

```text
namespace ownership
framework identity
concrete physical artifact schemas
ArtifactRef compatibility
canonical serialization
explicit assumptions
explicit mathematical references
explicit framework interpretation
no dynamic PhysMath AST extension
no duplicate Phys trust/corpus authority
```

## 21.1 Required negative tests

At minimum:

```text
physical meaning inferred from a PhysMath display name -> reject
unregistered physical artifact type -> reject
unregistered RuleID -> reject
missing required mathematical reference -> reject
duplicate SET element -> reject
invalid ParameterID binding -> reject
implicit condition discharge -> reject
dynamic PhysMath AST extension attempt -> reject
```

## 21.2 Framework disagreement test

Two framework packages MUST be able to attach different physical definitions to the same PhysMath representation without mutating the PhysMath artifact.

---

# 22. Explicit Phase-1 no-go list

The Phase-1 implementation MUST NOT introduce:

```text
TOE-specific ontology into PhysMath
Dancer/Dance/Stage as PhysMath types
physical-role enums such as TIME or RG_SCALE in PhysMath
Numerical execution into PhysMath
Numerical dependencies into the core PhysMath package
research state machines into PhysMath
paper/citation databases into PhysMath
trust/corpus/provenance ontology into PhysMath
universal Artifact interfaces
global semantic artifact registry
dynamic PhysMath AST extension
physical interpretation inferred from symbol names
scientific truth flags attached to mathematical artifacts
automatic theorem proving hidden in canonicalization
automatic assumption generation without explicit rule contracts
```

---

# 23. Secondary-system boundary

## 23.1 TOE-Agent

TOE-Agent may later consume:

```text
PhysMath artifacts
phys-* artifacts
paper references
hypotheses
correspondences
research states
```

None of those research workflow concerns belongs in this Phase-1 implementation contract.

## 23.2 Numerical

Numerical later consumes explicit executable numerical contracts produced by an upstream lowering process.

Numerical is not part of PhysMath's representation core and MUST NOT parse arbitrary PhysMath ASTs in Phase 1.

## 23.3 Provenance and trust

Phase 1 preserves only the minimum identity/reference information necessary to locate the represented object.

It does not define:

```text
scientific trust
corpus promotion
established/hypothesis status
human approval
provenance policy
```

Such metadata can be layered later without changing the mathematical identity of an artifact.

---

# 24. Implementation contract for the coding agent

## 24.1 Primary objective

Implement:

```text
PhysMath
    +
phys-* reference framework
    -> complete equivariance-theorem representation
```

## 24.2 Architectural decisions already fixed

The coding agent MUST NOT decide any of the following:

```text
whether PhysMath knows physics
whether phys-* may define physical meaning
whether multiple frameworks may coexist
what ArtifactRef means
what canonical digest framing is
whether symbol names define parameter identity
whether physical roles belong in PhysMath
whether physical assumptions belong in phys-*
whether local framework algebra may dynamically extend PhysMath
whether TOE-Agent is required
whether Numerical is required
```

These are fixed by this specification.

## 24.3 Permitted implementation decisions

The implementation plan MAY decide only:

```text
Go package/file structure
repository layout
constructor/helper naming where schema semantics are unchanged
error type naming where semantics are unchanged
test fixture organization
CI configuration
local cache/storage implementation
codec implementation strategy
benchmark harnesses
```

## 24.4 Stop rule

When implementation encounters behavior not defined here, the coding agent MUST NOT invent a semantic rule.

It must classify the gap as:

```text
MATH GAP        -> PhysMath specification issue
PHYS GAP        -> phys-* contract issue
FROZEN-PHYS GAP -> inspect frozen Phys authority; do not modify Phys
INTEGRATION GAP -> defer to secondary-system specification
```

Only a demonstrated contradiction or missing representation decision reopens this specification.

---

# 25. Reference implementation acceptance suite

## 25.1 Identity

**Given** identical canonical payloads and identical namespace/type/version.  
**When** ArtifactRef is computed twice.  
**Then** digests are identical.

## 25.2 Framing

**Given** two distinct field concatenations that would collide under raw concatenation.  
**When** encoded using Phase-1 framing.  
**Then** their canonical byte streams remain distinct.

## 25.3 Parameter alpha-renaming

**Given** the same declaration structure with display name `t` versus `tau`.  
**When** ParameterID is computed.  
**Then** ParameterID is identical.

## 25.4 Parameter separation

**Given** two independently declared parameters at different declaration positions.  
**When** ParameterIDs are computed.  
**Then** their identities differ.

## 25.5 Symbol physical neutrality

**Given** symbols named `t`, `k`, `c`, `G`, and `psi`.  
**When** PhysMath canonicalization runs.  
**Then** no physical role is attached from spelling alone.

## 25.6 Add/Mul normalization

**Given** nested Add/Mul structures with different insertion order.  
**When** canonicalized.  
**Then** equivalent commutative structures have one canonical child order.

## 25.7 Condition preservation

**Given** an operation requiring `x != 0`.  
**When** the operation executes.  
**Then** the condition is explicit in the result/derivation and is not silently removed.

## 25.8 Rule registry

**Given** an unknown TransformationRuleID.  
**When** deserialized.  
**Then** validation rejects it.

## 25.9 Derivation determinism

**Given** the same premises, ordered derivation steps, rule IDs, parameters, and conditions.  
**When** serialized and replayed.  
**Then** the canonical result references are reproducible.

## 25.10 Theorem completeness

**Given** a theorem with a claimed derivation.  
**When** validated.  
**Then** all premise, conclusion, assumption, and derivation references are present and type-correct.

## 25.11 Equivariance representation

**Given** the generic equivariance structure:

```text
f(rho_X(g,x)) = rho_Y(g,f(x))
```

**When** represented.  
**Then** both sides, the relation, the relevant functions/actions, and the final theorem are represented as typed artifacts.

## 25.12 Equivariance derivation

**Given** a concrete equivariance theorem fixture.  
**When** its derivation is replayed.  
**Then** every step resolves to explicit inputs, rule IDs, parameters, and conditions, and the final step produces the theorem conclusion.

## 25.13 Many-framework interpretation

**Given** the same mathematical theorem artifact.  
**When** two independent `phys-*` frameworks attach different physical assumptions or meanings.  
**Then** the PhysMath theorem identity remains unchanged while the framework artifacts remain distinct.

## 25.14 Framework contamination

**Given** a physical assumption defined by framework A.  
**When** framework B consumes the same mathematical artifact without explicitly importing A's contract.  
**Then** A's physical assumption is not silently attached to B.

## 25.15 Dynamic extension prohibition

**Given** a framework-local mathematical node not registered in PhysMath v1.  
**When** a package attempts to inject it into the PhysMath runtime AST.  
**Then** the operation is rejected; the package must keep the type local or define an explicit future PhysMath mapping.

---

# 26. Implementation order

The implementation sequence is intentionally short.

```text
1. phys-artifact
      |
      v
2. PhysMath core identity + canonical codecs
      |
      v
3. Expression + Symbol + Rational + Add/Mul + Neg + Pow + Sqrt + Call + Relation
      |
      v
4. ParameterID + BoundParameterRef + constants
      |
      v
5. Function + FunctionApplication + Map + Tensor + Index + Collection
      |
      v
6. Transformation registry + condition handling
      |
      v
7. Derivation + Theorem artifacts
      |
      v
8. one concrete phys-* framework
      |
      v
9. equivariance theorem reference fixture
      |
      v
10. second phys-* interpretation fixture
      |
      v
11. canonical vectors + negative tests + conformance suite
```

TOE-Agent, Numerical, and research/trust layers do not appear in this sequence.

---

# 27. Architecture invariants to preserve permanently

```text
PhysMath = mathematical representation

phys-* = framework-specific physical representation

Paper = research context, not PhysMath ontology

Equation = primary formal representation unit

Derivation = explicit ordered transformation structure

Theorem = represented mathematical claim + derivation

Physical assumption = phys-* owned

Physical interpretation = phys-* owned

Mathematical identity = explicit structural identity

Symbol spelling != physical meaning

Multiple physical interpretations may coexist

No dynamic PhysMath AST extension

Phys remains frozen

Numerical is downstream and deferred

TOE-Agent is downstream and deferred

Trust/provenance/corpus is downstream and deferred

Representation success != scientific truth
```

---

# 28. Final handoff rule

This specification is complete enough to begin implementation when the coding agent can satisfy the following statement:

> **Given only this specification, the frozen Phys authority where explicitly referenced, and a concrete equivariance-theorem fixture, a competent Go developer can implement PhysMath and a `phys-*` framework without inventing any mathematical identity rule, physical interpretation rule, serialization rule, derivation representation rule, or cross-layer ownership rule.**

The next artifact is a project-specific implementation plan.

No secondary architecture work is required to begin Phase-1 coding.

**END OF PM-REP-001 v1.0**
