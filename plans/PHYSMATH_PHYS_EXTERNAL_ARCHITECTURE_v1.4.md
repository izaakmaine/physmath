# PhysMath / Phys / phys-* / Numerical / TOE-Agent Architecture
## Foundation Contract — v1.4

**Document ID:** PM-ARCH-FOUNDATION-001  
**Version:** 1.4  
**Status:** **ARCHITECTURE SPECIFICATION — PROPOSED FOR FINAL BLOCKER-ONLY AUDIT**  
**Purpose:** Define the cross-project architecture, schemas, identity rules, ownership boundaries, state machines, projection protocol, and numerical boundary so that an implementation agent is not required to invent an architectural decision.

> **This document is the architecture. It is not another closure plan. No architecture-closure document follows v1.4. After the final blocker-only audit described in §34, this document is either frozen or explicitly rejected and amended; an additional closure-plan layer is prohibited.**

---

# 0. Normative language and source authority

The words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative.

This document contains three kinds of decisions:

| Lineage | Meaning |
|---|---|
| `INHERITED-PHYS` | Constraint inherited from the frozen Phys authority; v1.4 does not redefine it. |
| `INHERITED-ARCH` | Architectural decision carried forward from the reviewed architecture chain. |
| `V1.4-DECIDED` | New decision made explicitly by this specification to remove remaining implementation ambiguity. |

## 0.1 Frozen Phys authority

For this architecture, the trusted Phys authority is:

```text
Repository: github.com/PithomLabs/phys
Semantic specification: specs_v2_3.md
Process authority: AGENTS.md
Operational freeze: the frozen 42-file Phys repository state established for the current Phys line
```

The Phys repository, its production dependency graph, its public API surface, its frozen specification, and its frozen file tree are treated as immutable by this architecture.

A run-time implementation record MUST additionally record the exact Git revision used for the frozen Phys reference. The revision identifies which frozen instance was used; it is not a new semantic rule.

## 0.2 Authority precedence

When documents disagree, precedence is:

```text
1. frozen Phys specification for Phys behavior
2. this v1.4 architecture for cross-project architecture
3. implementation plan for execution order and file-level work
4. README/documentation examples
```

A coding agent MUST NOT use an implementation plan to override this architecture or the frozen Phys specification.

---

# 1. Constitutional architecture

The ecosystem consists of distinct projects with distinct semantic responsibilities.

```text
                          PROJECT A
                    TOE-Agent / Research
                   /       |       \\
                  /        |        \\
                 v         v         v
           PhysMath     phys-*      Phys
              |           |           |
              |           |           |
              +-----------+-----------+
                          |
                          v
                     Research graph

                    phys-* / lowering
                          |
                          v
                    Numerical Contract
                          |
                          v
                      Numerical
```

The graph above describes references and contracts, not inheritance.

## 1.1 PhysMath

PhysMath is the pure mathematical substrate.

PhysMath owns:

- mathematical objects and sorts;
- exact symbolic expressions;
- mathematical functions and operators;
- mathematical conditions;
- mathematical transformations;
- mathematical traces;
- mathematical identity and binding;
- deterministic canonicalization;
- bounded exact mathematical operations defined by its schema version.

PhysMath MUST NOT own:

- physical ontology;
- physical interpretation;
- physical regimes;
- physical provenance;
- physical corpus status;
- physical trust authority;
- physical assumptions as epistemic metadata;
- numerical execution.

A mathematical object MUST NOT acquire physical meaning merely because its mathematical structure resembles a structure used by physics.

## 1.2 phys-* external domain libraries

Each `phys-*` package owns a specific physical framework/domain.

A `phys-*` package MAY depend on:

```text
PhysMath
Phys
phys-artifact
Numerical/contract
```

A `phys-*` package MUST NOT:

- modify the Phys kernel;
- create a second trust/corpus authority;
- create a competing assumption/convention authority;
- inject runtime AST sorts into PhysMath;
- infer physical semantics inside PhysMath;
- require a global registry for in-memory operation;
- silently turn hypotheses into trusted Phys corpus items.

## 1.3 Phys

Phys remains the frozen trusted physical formalization substrate.

Phys owns:

- trusted physical objects already defined by its frozen contract;
- physical assumptions and conventions already defined by its frozen contract;
- provenance;
- corpus status;
- MRC;
- trusted derivation/session/ledger mechanisms.

Phys MUST NOT receive new architecture-driven imports, APIs, kernel mutations, corpus-promotion paths, or trust-admission paths under this specification.

## 1.4 TOE-Agent

TOE-Agent owns research state and cross-domain research orchestration.

It owns:

- research hypotheses;
- hypothesis state;
- correspondence proposals;
- proposed physical interpretations;
- formalization proposals and responses;
- research meta-ledger;
- attack/counterexample records;
- revision lineage.

It does not own the semantics of PhysMath, Phys, or any `phys-*` framework.

## 1.5 Numerical

Numerical owns numerical execution.

Numerical owns:

- `ComputationalContract` schema;
- materialized numeric execution plans;
- numerical solvers;
- simulation and quantitative computation;
- numeric execution constraints.

Numerical MAY receive `ArtifactRef`s naming frozen Phys objects as provenance/model references, but resolving those refs is performed by the caller/storage environment, not by the Numerical execution engine.

Numerical MUST NOT:

- parse PhysMath symbolic ASTs directly;
- infer physical semantics;
- add physical assumptions;
- repair malformed physical contracts;
- reinterpret mathematical notation;
- perform trust adjudication.

---

# 2. Scope: target ecosystem versus frozen Phys MVP

The documents in this architecture describe two different scopes and MUST NOT be conflated.

## 2.1 Frozen Phys MVP

The frozen Phys MVP is the existing `github.com/PithomLabs/phys` implementation governed by `specs_v2_3.md`, its process instructions, and its frozen repository state.

No PhysMath, `phys-*`, TOE-Agent, Numerical, or interop implementation is implied by the frozen Phys MVP.

## 2.2 Target ecosystem

The following projects are architectural peers in the target ecosystem:

```text
phys-artifact
PhysMath
Phys
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
TOE-Agent
github.com/PithomLabs/numerical (contract + execution)
```

The first post-MVP external physics stress test remains GR. The target architecture does not require every project above to exist before the first implementation plan.

## 2.3 Growth Gate timing

The Phys Growth Gate is **post-MVP**.

During the frozen Phys MVP:

```text
Phys kernel = frozen
Growth Gate implementation = absent
GR stress evidence = evidence only
GR stress evidence MUST NOT modify Phys
```

After the MVP, an explicit Growth Gate process MAY evaluate whether a reusable, foundational, theory-neutral capability belongs in Phys.

A GR requirement is never by itself sufficient to justify kernel growth.

---

# 3. Dependency graph

The production import graph is:

```text
phys-artifact
      ^
      |
      +-------------------+
      |                   |
   PhysMath            Numerical/contract
      ^                   ^
      |                   |
      +---------+         |
                |         |
              phys-* -----+
                ^
                |
            TOE-Agent
```

The actual architectural rules are:

```text
PhysMath -> phys-artifact
Phys -> no new dependencies
phys-* -> PhysMath
phys-* -> Phys
phys-* -> phys-artifact
phys-* -> Numerical/contract
Numerical/contract -> phys-artifact
Numerical engine -> Numerical/contract
Numerical engine -> no phys-*
Numerical engine -> no PhysMath
TOE-Agent -> PhysMath
TOE-Agent -> Phys
TOE-Agent -> phys-*
TOE-Agent -> phys-artifact
```

`phys-artifact` and `Numerical/contract` are mechanical/schema modules, not semantic authorities.

The `PhysMath ⊥ Phys` invariant remains intact at the domain level: neither imports the other, and Phys remains byte-frozen.

---

# 4. In-memory composition versus persisted references

This distinction is normative.

## 4.1 In-memory composition

Within one process, domain code uses concrete typed Go values.

Examples:

```text
physmath.Expression
phys-gr.GRExpr
phys-gr.Tensor
phys.PhysicalObject / core.Object
numerical/contract.ComputationalContract
```

A typed value MUST NOT require artifact lookup merely to be used in the same process.

## 4.2 Serialized composition

Across persistence or process boundaries, durable cross-project references use `ArtifactRef`.

```text
Typed Go value
    !=
ArtifactRef
```

An `ArtifactRef` is an immutable content-addressed **name**.

It is not:

- a resolver;
- a URL;
- a database handle;
- an implicit network call;
- a semantic authority.

Resolution is an environment concern.

A storage environment MAY implement a resolver, but no global runtime registry is required.

## 4.3 Resolution rule

A consumer that needs a serialized artifact MUST obtain the bytes from an explicit storage environment and verify:

```text
computed ArtifactRef == referenced ArtifactRef
```

before treating the payload as the referenced artifact.

A semantic core MUST NOT silently perform network or registry resolution.

---

# 5. Ecosystem ArtifactRef

`ArtifactRef` has one definition for the ecosystem.

The neutral mechanical owner is:

```text
Go module: github.com/PithomLabs/phys-artifact
Go package: artifact
```

This module contains no physics, mathematics, trust, interpretation, research state, registry policy, or numerical algorithms.

## 5.1 Exact type

```go
type ArtifactRef struct {
    Namespace     string
    ArtifactType  string
    SchemaVersion string
    Digest        [32]byte
}
```

`SchemaVersion` is a string. This matches the frozen Phys object-schema convention and permits non-numeric future schema identifiers.

## 5.2 Identity

Artifact identity is content-addressed.

There is no alternate `artifact_id` mechanism in the v1.4 architecture.

The identity tuple is:

```text
(Namespace, ArtifactType, SchemaVersion, Digest)
```

Two references are equal iff all four values are equal.

The digest alone is not an identity for this architecture.

## 5.3 Namespace grammar

Namespaces MUST be ASCII lowercase and match:

```text
[a-z][a-z0-9._-]{0,127}
```

The namespace is UTF-8 encoded as bytes for canonical framing, but the allowed namespace alphabet is intentionally restricted to ASCII to prevent namespace-equivalence ambiguity.

## 5.4 Artifact-type grammar

Artifact types MUST be ASCII lowercase and match:

```text
[a-z][a-z0-9._-]{0,127}
```

## 5.5 Schema-version grammar

Schema versions MUST be non-empty UTF-8 strings of at most 32 bytes.

No semantic ordering is implied by version strings.

Compatibility MUST be explicit; consumers MUST reject incompatible versions by default.

---

# 6. Canonical identity and byte framing

There is exactly one normative digest construction for every v1.4 durable artifact.

## 6.1 Canonical payload

Every durable artifact schema defines `CanonicalPayloadBytes`.

These are the exact bytes of the artifact payload after schema-specific canonicalization and validation.

Transport encodings are separate from canonical payload bytes.

## 6.2 Normative digest input

Define:

```text
Len32(s)
    = unsigned 32-bit big-endian integer equal to
      len([]byte(s))
```

where `s` is encoded as UTF-8 bytes.

The exact digest input is:

```text
Len32(Namespace)
|| UTF8(Namespace)
|| Len32(SchemaVersion)
|| UTF8(SchemaVersion)
|| Len32(ArtifactType)
|| UTF8(ArtifactType)
|| CanonicalPayloadBytes
```

Then:

```text
Digest = SHA-256(the exact bytes above)
```

There is no second or alternate digest concatenation formula.

The order above is normative even though the Go declaration in §5.1 places `ArtifactType` before `SchemaVersion`. The declaration order of a host-language struct MUST NOT override the explicitly specified canonical framing order.

## 6.3 Embedded ArtifactRef encoding

When an `ArtifactRef` appears inside another canonical payload, it is encoded using this exact field order:

```text
Len32(Namespace)
|| UTF8(Namespace)
|| Len32(SchemaVersion)
|| UTF8(SchemaVersion)
|| Len32(ArtifactType)
|| UTF8(ArtifactType)
|| Digest[32]
```

The embedded-reference encoding is a named primitive of this architecture. Its field order is normative wherever an `ArtifactRef` is embedded; the parent struct's field declaration order governs only the order of the parent fields themselves.

Embedded refs MUST NOT be encoded as:

- JSON objects;
- JSON strings;
- hex strings;
- unframed digest bytes.

Transport JSON MAY display an `ArtifactRef` in a documented lossless form, but that transport display MUST NOT be substituted into canonical identity bytes.

## 6.4 Canonical collection ordering

Every canonical collection MUST explicitly declare `SET` or `SEQUENCE` semantics in its owning schema or in the complete collection table in §7.4.

```text
SEQUENCE = preserve declared order
SET      = sort by the field's declared canonical ordering
```

For an `ArtifactRef` SET, ordering is lexicographic over:

```text
Namespace
ArtifactType
SchemaVersion
Digest[32]
```

using UTF-8 byte order for the three strings and raw-byte lexicographic order for the digest.

No map iteration order is permitted to influence canonical identity.

## 6.5 Canonical payload restrictions

Exact symbolic artifacts MUST NOT use floating-point values as their canonical scalar representation.

Canonical payloads MUST NOT depend on:

- locale;
- time zone;
- runtime pointer values;
- Go map iteration order;
- memory addresses;
- process IDs;
- timestamps not explicitly part of the schema;
- presentation whitespace.

A durable artifact MUST validate its complete canonical payload before computing its `ArtifactRef`.

# 7. Canonical primitive encoding, namespace registry, and schema vocabulary

All v1.4 artifact schemas use the following primitive canonical encoding rules before the outer digest framing in §6.2.

```text
String(s)      = Len32(UTF8(s)) || UTF8(s)
Bytes(b)       = Len32(len(b)) || b
UInt8(x)       = one byte, unsigned
UInt16(x)      = two-byte unsigned big-endian
UInt32(x)      = four-byte unsigned big-endian
UInt64(x)      = eight-byte unsigned big-endian
Bool(false)    = 0x00
Bool(true)     = 0x01
Array(xs)      = UInt32(len(xs)) || Canonical(x0) || ... || Canonical(xn-1)
Optional(nil)  = 0x00
Optional(v)    = 0x01 || Canonical(v)
Digest32(d)    = exactly 32 raw bytes
```

Signed integer encodings, when present, use two's-complement big-endian with the fixed width stated by the field type. Arbitrary-precision integers use a sign byte followed by a length-prefixed magnitude byte string.

For arbitrary-precision integers, the sign byte is `0x00` for non-negative and `0x01` for negative, followed by `Len32(magnitude_bytes)` and the minimal big-endian unsigned magnitude bytes. Zero is always encoded as non-negative magnitude zero.

`Float64Bits` is canonicalized as exactly 8 bytes containing the IEEE-754 binary64 bit pattern in big-endian byte order. Canonical payloads MUST reject NaN and ±Inf. `Float64Bits` is permitted only in explicit numerical-execution schemas and never in exact PhysMath symbolic payloads.

Semantic enums are encoded as `String(enum-token)` using their exact closed vocabulary. A binary `UInt8` discriminator is used only where a schema is an explicit tagged union or compact operator/comparison tag and the schema names the tag values.

Canonical struct fields appear in their declared schema order.

For optional fields, the presence marker is always emitted even when absent. There is no implicit default omission.

Canonical schemas MUST NOT contain Go maps.

## 7.1 Namespace registry and ownership

There is no universal runtime registry.

The following namespace families are reserved by architecture:

| Namespace | Owner | Artifact-type policy |
|---|---|---|
| `physmath` | PhysMath | Exact vocabulary in §7.3 |
| `phys` | frozen Phys | Only `object`/`1` in MVP |
| `phys.qm` | phys-qm | Package-owned vocabulary |
| `phys.bohm` | phys-bohm | Package-owned vocabulary |
| `phys.gr` | phys-gr | Package-owned vocabulary |
| `phys.cst` | phys-cst | Package-owned vocabulary |
| `phys.cdt` | phys-cdt | Package-owned vocabulary |
| `toe` | TOE-Agent | Exact vocabulary in §7.3 |
| `numerical` | Numerical | `contract`/`1` |
|

`phys-artifact` does not own a semantic namespace.

A new `phys-*` project MUST declare exactly one namespace matching its project identity and MUST publish its artifact-type vocabulary in its package contract.

A new ecosystem namespace requires a v1.4 architecture amendment and cannot be silently invented by an implementation agent.

## 7.2 Canonical collection rules

The following collection semantics are normative. A schema that contradicts this table is invalid.

### SEQUENCE

```text
Function.ParameterIDs
Call.Args
FunctionApplication.Args
OperatorApplication.Args
Tensor.Entries
Tensor.IndexRefs
Transformation.Inputs
CorrespondenceProposal.SourceRefs
CorrespondenceProposal.TargetRefs
FormalizationProposal.SourceArtifactRefs
FormalizationResponse.ProjectedRefs
MathTrace.InputRefs
ResearchStep.InputRefs
DerivationObligation.Premises
DerivationObligation.RequiredRules
```

`OperatorApplication.Args` preserves argument position and therefore MUST NOT be sorted.

`Tensor.IndexRefs[i]` corresponds to tensor mode/slot `i`.

### SET

```text
SetCollection.Members
PhysicalInterpretation.MathRefs
PhysicalInterpretation.RequiredConditions
PhysicalContract.MathRefs
PhysicalContract.RequiredConditions
PhysicalArtifact.MathRefs
PhysicalArtifact.RequiredConditions
ProposedInterpretation.SourceMathRefs
ProposedInterpretation.CorrespondenceRefs
ProposedInterpretation.EvidenceRefs
CorrespondenceProposal.MappingArtifactRefs
CorrespondenceProposal.FrameworkRefs
ResearchHypothesis.ClaimRefs
ResearchHypothesis.ConstraintRefs
ResearchHypothesis.CorrespondenceRefs
ResearchHypothesis.ProposedInterpretationRefs
ResearchHypothesis.CounterexampleRefs
ResearchHypothesis.InconsistencyRefs
FormalizationProposal.RequiredFrameworkRefs
MathTrace.ConditionRefs
```

For numerical contracts:

```text
ComputationalContract.Inputs           = SET, unique and sorted by InputBinding.Name
ComputationalContract.Parameters       = SET, unique and sorted by NumericParameter.Name
ComputationalContract.RequestedOutputs = SET, unique and sorted by OutputRequest.Name
```

The sort key for each is the UTF-8 byte order of its `Name`. Duplicate names are invalid.

`RequiredConventions` is sorted by canonical key `(Namespace, Key, Value)` and duplicate tuples are invalid.

`ValidationReport.Checks` is a SET keyed by `Code`; duplicate codes are invalid.

A SET rejects duplicate members unless the owning schema explicitly defines a semantic deduplication operation. Mathematical condition finalization is such an explicit operation; general canonical serialization is not permitted to silently deduplicate.

## 7.3 Exact durable artifact vocabulary

The following identity registry is normative for v1.4. A durable artifact family MUST use the exact `Namespace`, `ArtifactType`, and `SchemaVersion` listed here unless it is a package-owned extensible family explicitly delegated by this table.

### PhysMath

```text
physmath/expression/1
physmath/function/1
physmath/function_application/1
physmath/functional/1
physmath/functional_application/1
physmath/operator/1
physmath/operator_application/1
physmath/tensor/1
physmath/index/1
physmath/map/1
physmath/relation/1
physmath/set_collection/1
physmath/transformation/1
physmath/limit/1
physmath/asymptotic_truncation/1
physmath/derivation_obligation/1
physmath/condition/1
physmath/trace/1
```

### Frozen Phys

```text
phys/object/1
```

No other external Phys artifact type exists in v1.4 MVP.

### TOE-Agent

```text
toe/proposed_interpretation/1
toe/correspondence_proposal/1
toe/research_hypothesis/1
toe/validation_report/1
toe/projection_loss_report/1
toe/formalization_proposal/1
toe/formalization_response/1
toe/research_step/1
```

### Numerical

```text
numerical/contract/1
```

### phys-* framework packages

Each framework package declares exactly:

```text
<framework-namespace>/interpretation/1
<framework-namespace>/contract/1
<framework-namespace>/artifact/1
```

The package-owned `artifact` payload schema is defined by that framework's contract. No global physical-payload registry is required.

`ArtifactRef` itself is a naming primitive and is not a separately hashable semantic artifact family.

## 7.4 PhysMath artifact-type mapping

Within namespace `physmath`, the artifact type for each of the 16 v1 mathematical sorts is fixed by this table:

| Sort | ArtifactType |
|---|---|
| Expression | `expression` |
| Function | `function` |
| FunctionApplication | `function_application` |
| Functional | `functional` |
| FunctionalApplication | `functional_application` |
| Operator | `operator` |
| OperatorApplication | `operator_application` |
| Tensor | `tensor` |
| Index | `index` |
| Map | `map` |
| Relation | `relation` |
| SetCollection | `set_collection` |
| Transformation | `transformation` |
| Limit | `limit` |
| AsymptoticTruncation | `asymptotic_truncation` |
| DerivationObligation | `derivation_obligation` |

`condition` and `trace` are meta-artifact types, not mathematical expression sorts.

# 8. Frozen Phys artifact references

Phys remains a reference target and does not import `phys-artifact`.

Consumers construct ecosystem `ArtifactRef`s for frozen Phys objects using the frozen Phys canonical bytes.

## 8.1 MVP referenceable Phys artifact type

The v1.4 ecosystem contract defines exactly one Phys artifact type for external reference in MVP:

```text
Namespace    = phys
ArtifactType = object
SchemaVersion = 1
```

Its payload bytes are the exact canonical object bytes defined by the frozen Phys specification for `core.Object`.

The consumer MUST NOT reinterpret, reserialize, normalize, or otherwise modify those canonical object bytes before applying the v1.4 ecosystem digest framing.

The ecosystem digest for a Phys object is not required to equal any internal Phys object hash. It is a new ecosystem identity computed over the exact frozen Phys canonical bytes using the v1.4 framing in §6.2.

## 8.2 Non-object Phys artifacts

Phys session, ledger, manifest, and other internal structures remain externally referenceable only if a future Phys specification explicitly defines an ecosystem reference contract for them.

An implementation agent MUST NOT invent an external ArtifactRef type for an internal Phys structure merely because it is serializable.

## 8.3 Frozen Phys schema-version distinction

The frozen Phys specification version and a Phys artifact schema version are different concepts.

For the referenceable Phys object in this architecture:

```text
Phys specification authority = specs_v2_3.md
Phys object schema version   = "1"
```

The specification version MUST NOT be substituted for the artifact schema version.

---

# 9. PhysMath mathematical IR

PhysMath is multi-sorted and statically typed.

The v1.4 core sort families are fixed to exactly **16** values:

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
SetCollection
Transformation
Limit
AsymptoticTruncation
DerivationObligation
```

The set is closed for schema version `1`. A future PhysMath schema version MAY add a sort only through an explicit architecture amendment.

A `phys-*` package MUST NOT inject a new runtime sort into PhysMath v1.

## 9.1 Sort-table cell vocabulary

Rule-family references in the normative sort matrix resolve to the exact concrete RuleIDs in Appendix B. Wildcard notation such as `EXPR-*` is descriptive only and MUST NOT appear as a serialized `RuleID` value.

## 9.2 Expression node kinds and binary encoding

`Expression` is a tagged mathematical union. MVP node kinds are:

| ExprTag | NodeKind |
|---|---|
| `0x01` | Symbol |
| `0x02` | Rational |
| `0x03` | Add |
| `0x04` | Mul |
| `0x05` | Neg |
| `0x06` | Pow |
| `0x07` | Sqrt |
| `0x08` | Call |
| `0x09` | Relation |
| `0x0A` | BranchSet |

Canonical expression encoding is:

```text
Expression = ExprTag || node-specific canonical fields
```

The exact fields are:

```text
Symbol:
    ExprTag(0x01)
    || Optional(ParameterID)
    || Optional(ConstantID)
    || String(DisplayName)

Rational:
    ExprTag(0x02)
    || BigInt(Numerator)
    || BigInt(Denominator)

Add:
    ExprTag(0x03)
    || Array(Expression)

Mul:
    ExprTag(0x04)
    || Array(Expression)

Neg:
    ExprTag(0x05)
    || Expression(Operand)

Pow:
    ExprTag(0x06)
    || Expression(Base)
    || Expression(Exponent)

Sqrt:
    ExprTag(0x07)
    || Expression(Radicand)

Call:
    ExprTag(0x08)
    || EmbeddedArtifactRef(FunctionRef)
    || Array(Expression(Args))

Relation:
    ExprTag(0x09)
    || ComparisonTag
    || Expression(Left)
    || Expression(Right)

BranchSet:
    ExprTag(0x0A)
    || Array(Expression(Candidates))
```

The tag values are closed. An unknown tag MUST be rejected.

`Array(Expression)` is a SEQUENCE. `BranchSet.Candidates` is a SEQUENCE and carries no implicit branch selection.

### ComparisonTag

`ComparisonTag` is a compact tagged operator vocabulary, not an open string enum:

```text
0x01 EQ
0x02 NE
0x03 LT
0x04 LTE
0x05 GT
0x06 GTE
```

An unknown tag MUST be rejected.

`Relation.Operator` is therefore a comparison tag and is not an `ArtifactRef` to the top-level `Operator` sort.

The v1 Phys projection mirror contains only:

```text
Symbol
Rational
Add
Mul
Neg
Pow
Sqrt
```

The node kinds `Call`, `Relation`, and `BranchSet` are not projectable through the strict MVP mirror.

## 9.3 Symbol

A `Symbol` contains:

```text
ParameterID?        optional bound mathematical parameter identity
ConstantID?         optional mathematical constant identifier
DisplayName         presentation label only
```

Exactly one of `ParameterID` and `ConstantID` MUST be present in a durable symbolic expression. An ephemeral interactive symbol MAY be unbound until persistence validation.

`DisplayName` MUST NOT participate in mathematical identity when a stable declaration/parameter identity exists.

PhysMath MUST NOT infer physical meaning from `DisplayName`.

## 9.4 Rational

```text
Numerator   signed arbitrary-precision integer
Denominator positive arbitrary-precision integer
```

Canonical form requires:

```text
Denominator > 0
gcd(|Numerator|, Denominator) = 1
0 => Numerator = 0, Denominator = 1
```

No floating-point approximation is a substitute for an exact rational.

## 9.5 Add and Mul

`Add` and `Mul` contain canonical n-ary child SEQUENCEs. Canonicalization MUST flatten nested nodes of the same associative kind and then sort children by canonical child Expression digest under the v1.4 identity framing.

For `Add`:

```text
n = 0 -> Rational(0,1)
n = 1 -> child
n >= 2 -> Add(children)
```

For `Mul`:

```text
n = 0 -> Rational(1,1)
n = 1 -> child
n >= 2 -> Mul(children)
```

Thus empty and single-child associative nodes do not survive canonicalization.

## 9.6 Neg

`Neg` has exactly one Expression operand.

A canonicalizer MUST apply the explicit double-negation normalization rule `EXPR-005`; construction itself does not silently rewrite the node before the transformation/canonicalization boundary.

## 9.7 Pow

`Pow` contains:

```text
Base     Expression
Exponent Expression
```

The PhysMath representation is broader than the frozen Phys projection domain.

For Phys projection, the exponent MUST be a literal `Rational` expression satisfying the exact frozen Phys `Pow` contract.

A non-rational exponent is classified:

```text
ProjectionClass = UNREPRESENTABLE
ReasonCode      = NON_RATIONAL_EXPONENT_IN_POW
```

## 9.8 Sqrt

`Sqrt` contains one radicand Expression.

Projection maps `Sqrt(x)` to the frozen Phys `Sqrt(x)` representation exactly.

## 9.9 Call

`Call` contains:

```text
FunctionRef ArtifactRef
Args        []Expression   // SEQUENCE
```

The function reference is mathematical identity, not a physical semantic label.

Call is not in the strict Phys mirror.

## 9.10 Relation

A Relation is both:

1. a top-level mathematical sort with artifact identity `physmath/relation/1`; and
2. an Expression node kind encoded with `ExprTag = 0x09`.

A top-level Relation artifact's canonical payload is exactly the Relation node body without the outer Expression tag:

```text
ComparisonTag
|| Expression(Left)
|| Expression(Right)
```

An embedded Relation node uses the same body preceded by `ExprTag(0x09)`.

The two representations are semantically aligned but are distinct artifact families and therefore MUST have different `ArtifactRef`s:

```text
physmath/relation/1
!=
physmath/expression/1
```

`MathCondition.Predicate` MUST reference `physmath/relation/1` in MVP.

## 9.11 BranchSet

A `BranchSet` contains an ordered sequence of candidate Expressions together with no implicit selection.

Branch selection is performed only by an explicit transformation operation governed by a known v1 RuleID. No branch is “obviously physical” by naming convention.

# 10. Higher mathematical sort contracts

The following sorts are schema families in v1.4. Their exact behavioral surface is fixed by the rule vocabulary in Appendix A; operations outside those rows are deferred rather than invented.

## 10.1 Function

```text
Function {
    ParameterIDs []ParameterID
    Domain        MathTypeRef
    Codomain      MathTypeRef
    Body          Expression
}
```

A function body references parameter identities, not display names.

## 10.2 FunctionApplication

```text
FunctionApplication {
    FunctionRef ArtifactRef
    Args        []Expression
}
```

The referenced function MUST resolve to a compatible `Function` artifact.

## 10.3 Functional

A `Functional` is a typed map from a function space to a scalar/math-space value.

MVP canonical representation:

```text
Functional {
    DomainFunctionType MathTypeRef
    Codomain           MathTypeRef
    Definition         ArtifactRef
}
```

The definition artifact is explicit; no implicit closure or executable callback is permitted in canonical data.

## 10.4 FunctionalApplication

```text
FunctionalApplication {
    FunctionalRef ArtifactRef
    ArgumentRef   ArtifactRef
}
```

## 10.5 Operator

An Operator is a mathematical transformation over one or more typed operands.

```text
Operator {
    OperatorID string
    Domain     MathTypeRef
    Codomain   MathTypeRef
    Definition ArtifactRef
}
```

Operator names alone do not determine physical semantics.

## 10.6 OperatorApplication

```text
OperatorApplication {
    OperatorRef ArtifactRef
    Args        []ArtifactRef
}
```

The special differential operator rules are defined in §12.

## 10.7 Tensor

```text
Tensor {
    Rank      uint8
    Dimension uint16
    Entries   []Expression
    IndexRefs []ArtifactRef
}
```

The number of entries MUST equal `Dimension^Rank` for a dense representation.

Sparse tensor encoding is a future schema version and MUST NOT be invented in v1.

## 10.8 Index

```text
Index {
    SymbolID  string
    Variance  Variance
    Slot      uint32
}
```

`Variance` is a mathematical structural property and does not itself imply a physical metric convention.

## 10.9 Map

```text
Map {
    Domain     MathTypeRef
    Codomain   MathTypeRef
    Definition ArtifactRef
}
```

## 10.10 SetCollection

```text
SetCollection {
    ElementType MathTypeRef
    Members     []ArtifactRef
}
```

Canonical ordering is member digest order.

## 10.11 Transformation

```text
Transformation {
    Inputs      []ArtifactRef   // SEQUENCE
    Output      ArtifactRef
    RuleID      string
    Parameters  CanonicalParameters
}

`Inputs` is a SEQUENCE because operand position may be semantically significant.

`RuleID` is a closed exact identifier from the concrete v1 RuleID registry in Appendix B. Wildcard family names such as `EXPR-*` are not legal serialized values. A transformation MUST reject a RuleID that is not listed in Appendix B.

`CanonicalParameters` is a byte string produced by the named transformation rule. Its schema, field order, and valid values are defined by the exact RuleID. A transformation MUST reject parameters that are not valid for its RuleID; there is no open-ended parameter bag.

A Transformation artifact records what transformation occurred; it does not itself constitute a proof of physical validity.

## 10.12 Limit

```text
Limit {
    ExpressionRef ArtifactRef
    Variable      ParameterID
    Target        Expression
    Direction     LimitDirection
}
```

The bounded limit engine defined by the implementation specification may operate only where an explicit rule exists.

## 10.13 AsymptoticTruncation

```text
AsymptoticTruncation {
    ExpressionRef ArtifactRef
    Parameter      ParameterID
    Order          uint32
    Direction      LimitDirection
}
```

The truncation operation MUST use an explicit finite-order rule and MUST NOT silently become a general series engine.

## 10.14 DerivationObligation

```text
DerivationObligation {
    Premises      []ArtifactRef
    Target        ArtifactRef
    RequiredRules []string
    FailurePolicy FailurePolicy
}
```

An obligation is a structured research/mathematical requirement, not a trust assertion.

---

# 11. Mathematical parameter identity and scope

PhysMath parameter identity is structural mathematical identity, not global provenance identity.

## 11.1 ParameterID

```go
type ParameterID [32]byte
```

For PhysMath v1:

```text
ParameterID = SHA-256(the exact bytes below)

String("physmath")
|| ScopePathBytes
|| UInt32BE(DeclarationOrdinal)
```

`ScopePath` is an ordered sequence of zero-based binder ordinals. Its canonical bytes are:

```text
UInt32BE(number_of_scope_components)
|| UInt32BE(ScopePath[0])
|| ...
|| UInt32BE(ScopePath[n-1])
```

There is exactly one scope-path count prefix. There is **no outer `Bytes(...)` wrapper** around `ScopePathBytes`.

In v1, the only schema-defined binder is `Function.ParameterIDs`. Its declaration scope is the Function artifact itself, whose root scope path is empty:

```text
ScopePath = []
```

The declaration ordinal of a Function parameter is its zero-based index in `Function.ParameterIDs`.

`Limit.Variable` and `AsymptoticTruncation.Parameter` reference an already-declared parameter; they do not create a new binder.

`DisplayName` is excluded from the derivation. When a `ParameterID` is serialized as a field, it uses `Digest32(ParameterID)` from §7.

## 11.2 Scope rule

A declaration is identified structurally by its declaration scope and declaration ordinal.

Within a single Function declaration scope:

```text
parameter 0 != parameter 1
```

because their declaration ordinals differ.

Two independent artifacts MAY contain structurally equal parameter identities when their alpha-equivalent declaration coordinates are equal. This is intentional and MUST NOT be treated as provenance equivalence.

A `ParameterID` alone does not identify the artifact in which a declaration occurred.

## 11.3 Alpha-renaming

Renaming:

```text
t -> tau
```

changes presentation text only.

It MUST NOT change `ParameterID`.

## 11.4 Cross-artifact parameter reuse

Cross-artifact references to an existing parameter MUST retain the declaration-artifact context explicitly at the composition layer.

A consumer MUST NOT infer that two independently authored artifacts share one parameter merely because their `ParameterID` values are equal.

When a parameter is intentionally shared across artifacts, the durable research or composition artifact MUST retain the declaring artifact `ArtifactRef` separately from the `ParameterID`.

No recursive `ParameterID -> declaring ArtifactRef -> ParameterID` identity construction is permitted.

When an operation contains a `ParameterID` target, the operation MUST resolve that identity against its explicit mathematical input/declaration context. If the operation has no unique declaration context for the target identity, the operation MUST reject the request rather than infer one.

## 11.5 Free symbols

Ephemeral interactive expressions MAY contain an unbound display symbol. A durable PhysMath artifact MUST NOT: every free symbol MUST be bound to an explicit `ParameterID` or `ConstantID` before persistence or inclusion in a durable transformation trace.

A coding agent MUST NOT generate random UUIDs for persistent mathematical parameter identity.

# 12. Differential operators: t-flow and k-flow

PhysMath contains no physical-domain enum such as `TIME` or `RG_SCALE`.

Physical frameworks assign physical roles externally.

## 12.1 Differential operator representation

A differential operator is represented as an `OperatorApplication` whose canonical operator identity is `differentiate` and whose canonical parameters contain:

```text
Target ParameterID
Order  UInt32
```

`Order >= 1`. Multiple derivative applications MUST canonicalize to one higher-order differentiation form with the summed `Order` value. Repeated equivalent applications therefore have one v1 canonical representation. The canonical parameter field order is `Target ParameterID` followed by `Order UInt32`.

The target is a mathematical parameter identity, not a display string and not a physical role tag.

No closed physical-domain tag exists inside PhysMath v1.

## 12.2 Invariance law

Variable renaming MUST preserve the target `ParameterID`.

Therefore:

```text
∂/∂t
```

and:

```text
∂/∂k
```

are distinct whenever their target parameters have distinct identities.

## 12.3 Physical role assignment

A physical framework MAY declare:

```text
ParameterID X corresponds to physical time
ParameterID Y corresponds to RG scale
```

but this declaration lives in a `phys-*` physical interpretation/contract artifact.

PhysMath does not inspect that declaration to perform mathematical equality.

## 12.4 Forbidden inference

The following is non-conforming:

```text
Symbol name == "t" -> physical time
Symbol name == "k" -> RG scale
```

This is prohibited even in helper code whose only input is a PhysMath expression.

---

# 13. Pure mathematical constants and physical constants

Exact mathematics must not be forced into floating-point representation merely because a constant is not rational.

## 13.1 Mathematical constants

Standard mathematical constants MAY be represented as ordinary `Symbol`/`ConstantID` values, for example:

```text
math.const.pi
math.const.e
```

These are exact symbolic identities until an explicitly defined mathematical rewrite applies.

No fixed rational approximation is permitted as a substitute for an exact mathematical constant.

## 13.2 Physical constants

Constants such as `c`, `hbar`, `G`, or a framework-specific coupling constant are **not PhysMath physical semantics**.

Inside PhysMath they are ordinary symbols whose physical meaning is supplied externally by `phys-*` contracts.

A PhysMath canonicalizer MUST NOT special-case physical symbol names.

## 13.3 Numerical lowering

Numerical lowering MAY resolve exact mathematical or physical constants to numerical values only when the owning mathematical/physical contract supplies an explicit deterministic mapping.

Such a lookup is not part of PhysMath core semantics.

---

# 14. Mathematical conditions

`MathCondition` is a pure mathematical meta-artifact.

It is not a Phys assumption and contains no provenance/corpus/trust metadata.

## 14.1 Condition schema

`MathCondition` has the exact durable identity:

```text
Namespace     = physmath
ArtifactType  = condition
SchemaVersion = 1
```

Its canonical payload is:

```text
EmbeddedArtifactRef(Predicate)
```

and:

```text
Predicate MUST reference physmath/relation/1
```

The `MathCondition` artifact itself is the unit whose digest is called the **condition digest** elsewhere in this architecture.

## 14.2 TransformResult

Mathematical transformation operations return a single uniform result type:

```text
TransformResult {
    Output     Expression
    Conditions []MathCondition   // SEQUENCE
    Trace      MathTrace?
}
```

Pure node constructors are not transformation operations and MAY return their constructed typed value directly.

A transformation that cannot generate conditions returns an empty `Conditions` sequence.

This uniformity is for composable mathematical transformations; it does not force raw AST construction APIs through a result wrapper.

The AST nodes themselves remain immutable and do not carry mutable side-condition fields.

## 14.3 Condition accumulation

Transformations MUST accumulate generated conditions rather than discard them.

A later transformation MAY add more conditions.

MVP has **no condition-discharge operation**.

Condition discharge is a future, explicit operation and cannot be inferred from a boolean simplification.

## 14.4 Condition canonicalization

Condition finalization performs:

```text
1. canonicalize each condition;
2. compute its condition digest;
3. sort by raw Digest[32] order;
4. remove adjacent duplicate digests.
```

Thus duplicate generated conditions MAY be semantically collapsed only by this explicit finalization operation.

Tree traversal order MUST NOT determine final condition ordering.

# 15. MathArtifact and shared mathematical types

## 15.1 MathArtifact

A durable mathematical artifact is an artifact-family-specific payload under namespace `physmath`. Its common logical form is:

```text
MathArtifact {
    SortID       string
    PayloadBytes []byte
}
```

`SortID` MUST be one of the 16 v1 sort IDs in §9. `PayloadBytes` are the canonical bytes of that sort's schema. `ArtifactType` identifies the artifact family; `SortID` MUST agree with it.

The `ArtifactRef` is not stored inside `PayloadBytes`. It is derived from the payload and schema metadata.

## 15.2 MathTypeRef

```text
MathTypeRef = string
```

Valid v1 values are exactly the 16 sort IDs in §9. `MathTypeRef` is a structural mathematical type name; it has no physical semantics.

## 15.3 LimitDirection

```text
FROM_LEFT
FROM_RIGHT
TWO_SIDED
PLUS_INFINITY
MINUS_INFINITY
```

## 15.4 Variance

```text
COVARIANT
CONTRAVARIANT
```

## 15.5 FailurePolicy

```text
REJECT
RECORD_ONLY
```

These shared definitions are part of PhysMath v1 and may not be redefined by `phys-*`.

# 16. Physical interpretation and physical contracts

Physical interpretation is always explicit.

## 16.1 PhysicalInterpretation

A `PhysicalInterpretation` is owned by a specific `phys-*` framework.

Canonical fields:

```text
PhysicalInterpretation {
    InterpretationID      string
    FrameworkRef          ArtifactRef
    PhysicalType          string
    MathRefs              []ArtifactRef
    RequiredConditions    []ArtifactRef
    RequiredConventions   []ConventionRequirement
    Scope                 ScopeSpec
}
```

No trust, provenance, corpus status, or curation fields are embedded in this artifact.

## 16.2 ScopeSpec

```text
ScopeSpec {
    RegimeID      string
    CoordinateSpec string
    ValidityNotes string
}
```

The framework owns the meaning of its scope strings.

`ValidityNotes` is descriptive metadata and MUST NOT be treated as an executable rule.

## 16.3 ConventionRequirement

```text
ConventionRequirement {
    Namespace string
    Key       string
    Value     string
}
```

A convention requirement is a requirement declared by the owning framework. It is not a second convention authority.

## 16.4 PhysicalContract

The physical contract for a framework-owned object is:

```text
PhysicalContract {
    ExtensionID          string
    PhysicalType         string
    FrameworkRef         ArtifactRef
    MathRefs             []ArtifactRef
    RequiredConditions   []ArtifactRef
    RequiredConventions  []ConventionRequirement
    InterpretationClass  string
}
```

There is no generic `CompatibilityRules` engine in v1.4.

Validation is performed by the owning framework against this declarative contract.

## 16.5 PhysicalArtifact

A durable physical artifact produced by a `phys-*` package has package-owned payload fields plus the common contract references:

```text
PhysicalArtifactPayload {
    PhysicalType        string
    FrameworkRef        ArtifactRef
    ContractRef         ArtifactRef
    MathRefs            []ArtifactRef
    RequiredConditions  []ArtifactRef
    RequiredConventions []ConventionRequirement
    PayloadBytes        []byte
}
```

`PayloadBytes` are canonical bytes for the package-owned physical fields. The package MUST define that payload schema in its own contract.

## 16.6 ProposedInterpretation

A research proposal for an interpretation is a TOE-Agent artifact, not a physical contract.

```text
ProposedInterpretation {
    ProposalID            string
    SourceMathRefs        []ArtifactRef
    ProposedPhysicalType  string
    ProposedFrameworkRef  ArtifactRef?
    CorrespondenceRefs    []ArtifactRef
    EvidenceRefs          []ArtifactRef
    Rationale             string
}
```

A proposal MUST NOT be treated as a physical interpretation merely because its fields resemble `PhysicalInterpretation`.

## 16.7 No promotion operation

There is no universal:

```text
Promote()
Approve()
Accept()
```

operation between `ProposedInterpretation` and `PhysicalInterpretation`.

A `phys-*` framework may publish its own `PhysicalInterpretation` artifact independently.

---

# 17. Multiple interpretations

The same mathematical artifact MAY have multiple framework-specific interpretations.

Example:

```text
MathArtifact M
   |
   +--> PhysicalInterpretation under framework A
   |
   +--> PhysicalInterpretation under framework B
```

There is no global winner.

A consumer MUST explicitly select the framework interpretation required by its contract.

Two frameworks MAY interpret the same math artifact differently without one silently superseding the other.

A conflict between physical interpretations is handled by the affected physical frameworks and/or Project A as research state; it is not resolved by PhysMath and is not adjudicated by a universal registry.

---

# 18. Research artifacts owned by TOE-Agent

TOE-Agent durable artifacts are content-addressed under namespace `toe`.

## 18.1 CorrespondenceProposal

```text
CorrespondenceProposal {
    ProposalVersion      uint32
    SourceRefs           []ArtifactRef
    TargetRefs           []ArtifactRef
    MappingType          string
    MappingArtifactRefs  []ArtifactRef
    FrameworkRefs        []ArtifactRef
    Rationale            string
}
```

A correspondence is a research mapping, not a physical identity.

## 18.2 ResearchHypothesis

```text
ResearchHypothesis {
    ClaimRefs             []ArtifactRef
    ConstraintRefs        []ArtifactRef
    CorrespondenceRefs   []ArtifactRef
    ProposedInterpretationRefs []ArtifactRef
    Status                ResearchState
    PreviousStateRef      ArtifactRef?
    CounterexampleRefs    []ArtifactRef
    InconsistencyRefs     []ArtifactRef
}
```

Persistent state changes create a new content-addressed artifact. Existing hypothesis artifacts are never mutated.

## 18.3 Research state vocabulary

The exact vocabulary is:

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

## 18.4 Research state transitions

Legal transitions are:

```text
PROPOSED
    -> UNDER_ATTACK
    -> REVISION_REQUIRED

UNDER_ATTACK
    -> REVISION_REQUIRED
    -> COUNTEREXAMPLE_FOUND
    -> INCONSISTENT
    -> AWAITING_FORMALIZATION

COUNTEREXAMPLE_FOUND
    -> REVISION_REQUIRED
    -> INCONSISTENT

INCONSISTENT
    -> REVISION_REQUIRED
    -> CLOSED

REVISION_REQUIRED
    -> PROPOSED

AWAITING_FORMALIZATION
    -> FORMALIZED
    -> REVISION_REQUIRED

FORMALIZED
    -> UNDER_ATTACK
    -> CLOSED

CLOSED
    -> terminal
```

There are no unspecified transitions.

`FORMALIZED` does not mean `TRUE`, `ESTABLISHED`, or `TRUSTED`.

`CLOSED` means the research workflow has been intentionally closed, not that the claim is scientifically true.

## 18.5 Mathematical operation outcomes

The mathematical operation outcome is owned by PhysMath and is a terminal result vocabulary for one invocation, not a TOE-Agent state machine.

The mathematical operation vocabulary is:

```text
SUCCESS
UNDEFINED
TRANSFORMATION_FAILED
OUT_OF_SCOPE
RESOURCE_LIMIT
```

These are terminal outcomes of an individual operation invocation, not a persistent state machine. A future invocation starts anew from its inputs.

---

# 19. Formalization protocol

Formalization is a deterministic protocol for attempting to project a research artifact into the frozen Phys expression/object vocabulary.

It does not create scientific truth and does not modify Phys.

## 19.1 FormalizationProposal

Canonical schema:

```text
FormalizationProposal {
    ProposalRevision       uint32
    ParentProposalRef      ArtifactRef?
    ProducerNamespace      string
    SourceArtifactRefs     []ArtifactRef   // SEQUENCE
    RequestedTarget        string
    ProjectionPolicy       string
    RequiredFrameworkRefs  []ArtifactRef  // SET
    ProposalState           ProposalState
    PreviousStateRef       ArtifactRef?
}
```

MVP exact values:

```text
RequestedTarget  = "phys/object/1"
ProjectionPolicy = "STRICT_MIRROR_ONLY"
```

Every `SourceArtifactRefs` entry MUST reference a `physmath/expression/1` artifact. A physical artifact or theory-local artifact is not directly projected into Phys by the MVP formalization protocol.

## 19.2 FormalizationResponse

```text
FormalizationResponse {
    ProposalRef        ArtifactRef
    ProposalRevision   uint32
    Outcome             FormalizationOutcome
    ProjectionClass     ProjectionClass
    ProjectedRefs       []ArtifactRef   // SEQUENCE; one element at most in MVP
    LossReport          ProjectionLossReport
    ValidationReport    ValidationReport
    ProducerNamespace   string
}
```

For MVP, `ProjectedRefs` contains exactly one `phys/object/1` reference only when `Outcome = PROJECTED_AS_HYPOTHESIS`; all other v1 outcomes contain zero projected references.

The response is immutable and content-addressed.

`ProposalRef` MUST identify the exact `SUBMITTED` or `RESOLVED` proposal artifact that was evaluated; a response that closes a proposal MUST point to the exact immutable `RESOLVED` proposal-state artifact.

## 19.3 Proposal state

`ProposalState` is a separate persistent lifecycle from `FormalizationOutcome`.

Legal `ProposalState` values:

```text
DRAFT
SUBMITTED
RESOLVED
```

A proposal revision MAY only transition:

```text
DRAFT -> SUBMITTED
SUBMITTED -> RESOLVED
```

`RESOLVED` is terminal.

Every state transition creates a new immutable `FormalizationProposal` artifact.

`PreviousStateRef` references the immediately preceding state artifact of the **same proposal revision**.

`ProposalRevision` is unchanged by a state transition. A new revision increments `ProposalRevision` and starts at `DRAFT`.

## 19.4 Revision lineage

When an evaluation returns `REQUEST_REVISION`:

```text
parent proposal -> (RESOLVED, REQUEST_REVISION)
new proposal   -> DRAFT with ParentProposalRef = parent ArtifactRef
```

`ParentProposalRef` links revisions. `PreviousStateRef` links lifecycle states within one revision.

A resolved proposal MUST NOT be resubmitted or mutated.

## 19.5 Formalization outcome vocabulary

The exact v1 `FormalizationOutcome` vocabulary is:

```text
REQUEST_REVISION
REJECTED
UNREPRESENTABLE
PROJECTED_AS_HYPOTHESIS
```

`PROJECTABLE` and `PARTIALLY_PROJECTABLE` are not valid v1 response outcomes. They are future vocabulary only and MUST NOT be serialized in schema version 1.

Meanings:

```text
REQUEST_REVISION
    proposal requires a new revision

REJECTED
    terminal protocol refusal; no projected Phys object is produced

UNREPRESENTABLE
    source cannot be represented under the v1 projection contract

PROJECTED_AS_HYPOTHESIS
    strict projection succeeded and the frozen Phys hypothesis-grade
    minting path succeeded
```

Formalization outcome does **not** automatically transition `ResearchHypothesis.Status`.

If research state changes as a consequence of formalization, TOE-Agent MUST explicitly create a new `ResearchHypothesis` artifact and state transition.

## 19.6 Formalization ownership

TOE-Agent owns proposal and research state.

The formalization protocol implementation is integration tooling using only the existing public Phys API and existing frozen Phys contracts.

No new Phys API is introduced by this architecture.

A reviewer suggestion is not evidence that an API exists. The implementation plan MUST name only APIs actually present in the frozen Phys authority.

## 19.7 Phys hypothesis minting

When confirmed against the frozen Phys specification, the raw public hypothesis-grade mint path is:

```text
hypothesis.NewCandidateConcept
```

External tooling MUST NOT invent a new Phys mint path.

The resulting content is hypothesis-grade under the frozen Phys rules and is not corpus promotion.

If the frozen Phys specification revision supplied to the implementation plan does not expose this exact path, the implementation plan MUST stop rather than inventing an alternative API.

# 20. Projection contract

Projection is a typed, deterministic lowering from a richer mathematical representation to the frozen Phys subset.

## 20.1 Projection subset

The exact mirror is:

```text
Symbol
Rational
Add
Mul
Neg
Pow
Sqrt
```

No other expression node is implicitly projectable.

## 20.2 Projection classes

Exactly:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

## 20.3 Compositional rule

For a composite node:

```text
FULLY_REPRESENTABLE
    iff every semantically required child is fully representable
    and the parent itself is representable
    and no required information is lost.
```

```text
PARTIALLY_REPRESENTABLE
    iff an explicit versioned projection contract permits loss
    and every permitted loss is recorded.
```

```text
UNREPRESENTABLE
    otherwise.
```

In particular:

```text
projectable(child) != projectable(parent)
```

## 20.4 MVP permitted-loss policy

MVP defines **zero permitted-loss contracts**.

Therefore:

```text
PARTIALLY_REPRESENTABLE
```

is part of the architectural vocabulary but is unreachable in MVP projection.

Future permitted-loss behavior requires a versioned projection-contract amendment.

## 20.5 Pow projection

A `Pow(base, exponent)` projects only when `exponent` is a literal `Rational` satisfying the frozen Phys exponent contract.

Otherwise:

```text
UNREPRESENTABLE
NON_RATIONAL_EXPONENT_IN_POW
```

The source expression MUST remain unchanged.

## 20.6 Associative projection

Before projection, `Add`/`Mul` are canonicalized into sorted n-ary form.

For `n >= 2`, projection constructs a left-associative binary Phys tree:

```text
Add([a,b,c])
    -> Add(Add(a,b),c)
```

where `[a,b,c]` is already canonicalized.

`n=0` and `n=1` do not reach projection because canonicalization eliminates them.

## 20.7 Projection loss report

```text
ProjectionLossReport {
    Classification ProjectionClass
    Reasons        []ProjectionReason
}

ProjectionReason {
    Code        string
    NodeKind    string
    Detail      string
}
```

MVP reason-code vocabulary is closed to:

```text
NON_RATIONAL_EXPONENT_IN_POW
UNSUPPORTED_NODE_KIND
CHILD_UNREPRESENTABLE
PHYSICAL_SEMANTICS_REQUIRED
PROJECTION_POLICY_FORBIDS_LOSS
```

A coding agent MUST NOT invent a new projection reason code in MVP.

---

# 21. Phys trust boundary and provenance law

Phys trust metadata is not replicated by `phys-*`.

The current Phys trust authority remains authoritative for its own objects and sessions.

## 21.1 No duplicate trust system

A `phys-*` package MUST NOT introduce competing versions of:

```text
provenance
corpus status
trust status
assumption authority
MRC
```

Domain-specific physical types are permitted; a second trust ontology is not.

## 21.2 External objects

Physical structures that cannot be represented by frozen Phys remain external `phys-*` artifacts.

They MUST NOT be forced into fake Phys kinds merely to obtain trust metadata.

## 21.3 Hypothesis contamination law

This rule is inherited from the frozen Phys authority, not created by v1.4:

> A session-recorded correspondence operating on hypothesis-grade operands remains hypothesis-grade under the frozen Phys contamination law.

Therefore a hypothesis-grade formalization path MUST NOT expect an `IDENTIFIED` or `ESTABLISHED` result merely because the mapping was successfully recorded.

A protocol implementation MUST use the exact contamination behavior of the frozen Phys specification.

## 21.4 TrustAssertion

`TrustAssertion` is a future design target only.

It is not an existing v1.4 runtime artifact, not an accepted currency, and not an invitation to add:

```text
AcceptTrust
AcceptFormalization
PromoteToEstablished
```

to Phys.

Any future standalone trust/curation system requires a separate architecture and Growth Gate decision.

---

# 22. Theory-local mathematics

A `phys-*` package MAY own a theory-local mathematical algebra.

For example:

```text
phys-gr.GRExpr
```

is allowed to remain package-local.

## 22.1 No PhysMath plugin AST

Theory-local algebra MUST NOT become a runtime extension node in PhysMath v1.

There is no dynamic trait registry, universal visitor interface, or runtime “unknown mathematical node” mechanism.

## 22.2 Mapping into PhysMath

A theory-local algebra MAY provide an explicit deterministic mapping into PhysMath when the mapped structure belongs to the closed PhysMath sort set.

For example:

```text
GRExpr subset
    -> PhysMath Expression
```

The mapping is package-owned and deterministic.

No forced migration of theory-local algebras into PhysMath is required in MVP.

## 22.3 Extension rule

“Closed” means **closed per schema version**.

A future PhysMath version MAY add a new mathematically justified sort through an explicit versioned architecture amendment.

No implementation agent may add a new sort without such a versioned decision.

---

# 23. MathTrace and replay

Replay is authority-local.

## 23.1 MathTrace

`MathTrace` is a durable PhysMath/authority-local operation artifact under:

```text
physmath/trace/1
```

Its exact schema is:

```text
MathTrace {
    ProducerNamespace       string
    OperationID              string
    OperationSchemaVersion  string
    InputRefs               []ArtifactRef   // SEQUENCE
    OutputRef               ArtifactRef
    ParameterBytes          []byte
    ConditionRefs           []ArtifactRef   // SET
}
```

`MathTrace` records enough information for the producing authority to deterministically replay its own operation vocabulary. It MUST NOT be interpreted as a research state transition.

## 23.2 Authority replay

Each producer owns replay of its own operation vocabulary:

```text
PhysMath -> replays PhysMath operations
phys-gr  -> replays phys-gr operations
Phys     -> replays Phys operations
```

No universal replay engine is required.

`ArtifactRef` inputs are references to durable objects, not implicit network operations.

## 23.3 Research meta-ledger

TOE-Agent owns a separate durable research-graph artifact:

```text
ResearchStep {
    Operation          string
    InputRefs          []ArtifactRef   // SEQUENCE
    OutputRef          ArtifactRef
    ProducerNamespace  string
}
```

Its exact identity is:

```text
toe/research_step/1
```

`ResearchStep` records macro-level research transitions between durable artifacts. It does not duplicate the internal operation trace of another authority.

TOE-Agent MUST NOT attempt to replay Phys or phys-* internal operation implementations merely because it references their outputs.

PhysMath operations do not automatically mint `ResearchStep` artifacts; TOE-Agent explicitly constructs them when recording research progression.

# 24. Numerical boundary

The Numerical boundary is downstream of physical semantics.

## 24.1 ComputationalContract ownership

The schema is owned by the Numerical project in the schema package:

```text
github.com/PithomLabs/numerical/contract
```

The package contains only execution-boundary schema definitions and mechanical encoding support.

It MUST NOT contain physical interpretation or trust logic.

`phys-*` produces valid `ComputationalContract` instances using this schema.

The Numerical engine consumes them.

## 24.2 Dependency rule

```text
phys-* -> numerical/contract
numerical engine -> numerical/contract
numerical engine -X-> phys-*
numerical engine -X-> PhysMath
```

There is no import cycle.

## 24.3 ComputationalContract schema

```text
ComputationalContract {
    PhysicalModelRef     ArtifactRef
    MathematicalModelRef ArtifactRef
    Inputs               []InputBinding
    Parameters           []NumericParameter
    RequestedOutputs     []OutputRequest
    NumericalMethod      string
    Precision            PrecisionRequirement
    ExecutionLimits      ExecutionLimits
    ProvenanceRefs       []ArtifactRef
}
```

### InputBinding

```text
InputBinding {
    Name        string
    ElementType string
    Shape       []uint64
    Data        []NumericScalar
}
```

### NumericParameter

```text
NumericParameter {
    Name  string
    Value NumericScalar
}
```

### NumericScalar

MVP supports exactly two variants. `NumericScalar` is an explicit tagged union:

```text
0x01 || Canonical(Rational)
0x02 || UInt64BE(Float64Bits)
```

`Float64Bits` is the IEEE-754 binary64 bit pattern stored canonically as an unsigned 64-bit value in big-endian byte order. NaN and ±Inf MUST be rejected. An unknown variant tag MUST be rejected.

### PrecisionRequirement

```text
PrecisionRequirement {
    AbsoluteTolerance NumericScalar?
    RelativeTolerance NumericScalar?
    MaxIterations     uint64?
}
```

### OutputRequest

```text
OutputRequest {
    Name string
}
```

### ExecutionLimits

```text
ExecutionLimits {
    MaxSteps      uint64?
    MaxElements   uint64?
    MaxWallUnits  uint64?
}
```

The exact resource-limit values are configuration parameters, not semantic rules.

A `ComputationalContract` contains already-materialized numeric values. In v1.4 there is no ecosystem-wide constant-to-number registry. A producing `phys-*` package or other contract owner MUST deterministically resolve any required exact mathematical or physical constant before constructing the numerical contract. Numerical MUST NOT infer, look up, reinterpret, or recompute symbolic constants.

## 24.4 In-process execution

When `phys-*` hands a `ComputationalContract` to Numerical in the same process, the execution structure MUST contain fully populated typed input arrays.

Numerical MUST NOT perform ArtifactRef lookup to execute a contract.

ArtifactRefs on the contract are immutable provenance/identity tags.

## 24.5 Persisted execution

For persisted/cross-process contracts, the storage environment resolves the required references and materializes the execution inputs before invoking Numerical.

The Numerical engine remains local and deterministic.

## 24.6 Float restriction

`Float64Bits` is permitted only in `ComputationalContract` numeric payloads and explicitly numeric execution artifacts.

It is forbidden in exact PhysMath AST payloads and in research/interpretation identity fields.

---

# 25. phys-* conformance contract

Every `phys-*` package MUST provide the following architectural proofs/tests.

## 25.1 Dependency proof

The package imports:

```text
PhysMath
Phys
phys-artifact
(optional) Numerical/contract
```

and does not import:

```text
TOE-Agent research implementation
other phys-* semantic packages unless a separately declared framework dependency exists
Numerical engine
```

Any cross-phys package dependency MUST be explicitly declared by that framework's contract rather than created ad hoc.

## 25.2 Semantic ownership proof

The package MUST demonstrate that physical interpretation is attached through its own `PhysicalInterpretation` and `PhysicalContract` artifacts.

## 25.3 Trust-boundary proof

The package MUST NOT declare an alternative provenance/corpus/trust vocabulary.

## 25.4 Cross-package assumption proof

Physical-domain assumptions MUST remain scoped to the owning framework and must not contaminate unrelated frameworks.

The following mechanics-specific condition is an inherited frozen-Phys check, not a universal rule for all `phys-*` packages. For the mechanics package specifically, the object-level relativity keys are not present:

```text
rest_frame
rest_mass_nonnegative
speed_of_light_positive
```

Mechanics has zero object-level assumption keys in this cross-domain contract.

## 25.5 Projection proof

Any artifact claiming projection support MUST provide a deterministic projection operation conforming to §20.

No universal `Projectable` interface is required.

## 25.6 Replay proof

Every durable operation trace has enough information for the owning package to replay its own deterministic operation.

## 25.7 Canonicalization proof

Every durable artifact has a deterministic canonical payload and golden digest vector.

## 25.8 Negative AST scan proof

The following are statically tested at source level:

- `ops` does not branch on physical symbol names such as `c`, `m`, `v`, `E`, or `p` to select mathematical semantics;
- Phys kernel code contains only the explicitly frozen closed-world physics identifiers allowed by the frozen specification.

These are source-structure tests, not runtime expression tests.

## 25.9 Mechanics-specific negative proof

A mutation such as silently attaching `rest_mass_nonnegative` to a mechanics constructor MUST be rejected by the conformance suite.

## 25.10 Hypothesis constructor proof

A hypothesis constructor MUST remain hypothesis-grade and MUST NOT be included in an ESTABLISHED-yielding constructor allowlist.

The mechanics/relativity established-constructor allowlists remain exactly those defined by the frozen Phys corpus contract; a hypothesis concept is never included merely because it is constructible.

## 25.11 Generated-key allowlist

For the MVP operation vocabulary, operation-generated Phys assumption keys are exactly:

```text
denominator/<hash>
selected_branch/<hash>
```

No third generated assumption key is permitted without a frozen architecture/specification amendment.

---

# 26. Status vocabulary separation

The following are different types and MUST remain different:

```text
MathOperationOutcome
ResearchState
ProposalState
FormalizationOutcome
ProjectionClass
PhysCorpusStatus
ObstacleClass
```

The words:

```text
UNREPRESENTABLE
PARTIALLY_PROJECTABLE
PARTIALLY_REPRESENTABLE
```

may appear in different typed domains only when their owning schema makes the distinction explicit.

A string in one vocabulary MUST NOT be treated as an enum value in another vocabulary.

---

# 27. Version compatibility

Every durable artifact is versioned by `SchemaVersion`.

## 27.1 Compatibility rule

A consumer MUST:

```text
accept exact supported schema version
OR
apply an explicitly declared compatibility adapter
OR
reject
```

It MUST NOT guess compatibility from field similarity.

## 27.2 No implicit migration

A schema migration is a new artifact.

It has its own:

```text
ArtifactRef
SchemaVersion
CanonicalPayload
Digest
```

The original artifact remains immutable.

## 27.3 Framework compatibility

A `PhysicalInterpretation` or `PhysicalContract` MUST declare its framework identity explicitly.

Version compatibility does not imply semantic equivalence.

---

# 28. Canonical serialization policy

Canonical representation and transport are distinct.

## 28.1 Canonical representation

For all schemas introduced by v1.4, the canonical payload is a deterministic binary schema-encoding defined by the artifact owner. Core primitive and shared-reference encodings are fixed by §§6–7; artifact owners may define only schema-specific field composition on top of those primitives.

The encoding MUST have:

```text
fixed field order
explicit length framing for variable-length fields
explicit collection counts
explicit integer widths
explicit UTF-8 encoding
explicit optional-field presence markers
```

No implementation may use Go's native memory layout as canonical bytes.

## 28.2 Transport

JSON MAY be used for interchange and debugging.

Transport JSON is not canonical identity material unless a future schema explicitly defines a normative canonical JSON byte representation.

## 28.3 Golden vectors

Every durable artifact family MUST ship at least:

```text
one canonical payload test vector
one expected ArtifactRef digest
one negative vector
```

---

# 29. Artifact contract template

Every durable artifact family is fully specified by this template:

| Field | Required v1.4 rule |
|---|---|
| Owner | Named project/package authority. |
| Namespace | Exact durable namespace and allocation rule. |
| Schema | Concrete field types and legal values. |
| Identity | ArtifactRef tuple and identity inputs. |
| Canonical form | Exact payload byte construction. |
| Serialization | Canonical + permitted transport forms. |
| Composition | Allowed child/reference relationships. |
| Resolution | Name resolution is environment infrastructure; semantic code does not require a registry. |
| Validation | Explicit legal-instance conditions. |
| Legal states | State vocabulary or “immutable/no state”. |
| Legal transitions | Explicit list or “none”. |
| Positive test | At least one executable Given/When/Then test. |
| Negative test | At least one executable Given/When/Then rejection test. |

Artifact identity within a family is content-addressed. The artifact body MUST NOT carry a self-referential digest that is part of its own canonical payload.

The thirteen-field template is inherited by every durable artifact family listed in Appendix D. A schema may also be embedded as a field inside another artifact; embedded schemas inherit the same primitive encoding rules but do not acquire a second ArtifactRef unless explicitly listed as a durable family in §7.3.

---

# 30. No-Go architectures

The following are explicitly forbidden:

1. PhysMath importing Phys.
2. Phys importing PhysMath.
3. Phys modifying its frozen dependency graph to satisfy this architecture.
4. A universal semantic `Artifact` interface.
5. A global artifact registry required by semantic cores.
6. An ArtifactRef functioning as an implicit resolver/network handle.
7. Dynamic PhysMath runtime AST extension.
8. `phys-*` injecting new PhysMath sorts.
9. PhysMath inferring physical meaning from mathematical structure.
10. `phys-*` creating a second trust/corpus/provenance authority.
11. Runtime `AcceptFormalization`, `AcceptTrust`, or corpus-promotion APIs in Phys.
12. A generic theorem prover hidden inside projection/compatibility machinery.
13. A generic compatibility-rule engine replacing framework-owned validation.
14. A global replay engine spanning all authorities.
15. Numerical parsing arbitrary PhysMath symbolic ASTs.
16. Numerical repairing missing physical semantics.
17. Random UUIDs for persistent mathematical parameter identity.
18. Physical operator tags such as `TIME` or `RG_SCALE` inside PhysMath core.
19. Floating-point exactness shortcuts in PhysMath canonical ASTs.
20. Silent partial projection with undeclared information loss.
21. Silent physical interpretation of a `ProposedInterpretation`.
22. Silent artifact migration across incompatible schema versions.

---

# 31. Supersession and lineage

v1.4 supersedes the corresponding reviewed decisions in the preceding architecture chain where they conflict with the normative rules above.

The supersession table is part of the architecture, not merely historical notes.

| Earlier decision | v1.4 decision | Lineage |
|---|---|---|
| PhysMath directly depending on Phys | Neither imports the other | `INHERITED-ARCH` |
| Phys artifact identity = id or digest | Content-addressed ArtifactRef only | `V1.4-DECIDED` |
| ArtifactRef unresolved owner | `github.com/PithomLabs/phys-artifact/artifact` | `V1.4-DECIDED` |
| ArtifactRef as resolver | ArtifactRef is a name; resolution is environmental | `V1.4-DECIDED` |
| Phys imports artifact-reference module | Phys remains frozen and imports nothing new | `V1.4-DECIDED` |
| Closed physical differential enum in PhysMath | Parameter identity only; physical role external | `V1.4-DECIDED` |
| Projection mirror not fully pinned | Exact seven-node mirror + compositional rule | `INHERITED-ARCH` + `V1.4-DECIDED` |
| Partial projection permitted implicitly | MVP zero permitted-loss contracts | `V1.4-DECIDED` |
| Generic compatibility rules | Narrow declarative PhysicalContract | `INHERITED-ARCH` |
| ProposedInterpretation promotion | Permanently distinct artifacts | `INHERITED-ARCH` |
| Formalization as conceptual transfer | Typed proposal/response protocol | `V1.4-DECIDED` |
| Hypothesis ingestion API may be invented | Frozen Phys public APIs only | `INHERITED-PHYS` |
| Global replay | Authority-local replay + TOE meta-ledger | `INHERITED-ARCH` |
| Numerical symbolic parsing | phys-* lowering + Numerical contract | `V1.4-DECIDED` |
| Growth Gate potentially active during MVP | Post-MVP only | `V1.4-DECIDED` |

No superseded architecture remains simultaneously active.

---

# 32. No-Invention Matrix

The following table is the final architectural completeness matrix.

| Concern | Normative decision | Owner | Concrete contract |
|---|---|---|---|
| Dependency direction | §3 | Architecture | Dependency rules |
| ArtifactRef ownership | §5 | phys-artifact | `ArtifactRef` |
| Artifact identity | §5 | phys-artifact | Four-field tuple |
| Digest framing | §6 | phys-artifact | Exact Len32 framing |
| Embedded refs | §6.3 | phys-artifact | Exact embedded encoding |
| Namespace ownership | §7 | Each producer | Namespace registry |
| Phys references | §8 | Consumer | `phys/object/1` |
| In-memory composition | §4 | Each implementation | Typed Go values |
| Serialized composition | §4 | Storage environment | `ArtifactRef` |
| PhysMath sort set | §9 | PhysMath | 16 closed v1 sort families |
| Expression nodes | §9.2 | PhysMath | 10 node kinds |
| Parameter identity | §11 | PhysMath | deterministic `ParameterID` |
| t/k separation | §12 | PhysMath + phys-* meaning | parameter identity |
| MathCondition | §14 | PhysMath | `physmath/condition/1` + canonical predicate ref |
| PhysicalInterpretation | §16 | phys-* | interpretation schema |
| ProposedInterpretation | §16.6 | TOE-Agent | proposal schema |
| PhysicalContract | §16.4 | phys-* | declarative schema |
| Multiple interpretations | §17 | Consumer/framework | explicit framework selection |
| Correspondence | §18.1 | TOE-Agent | proposal schema |
| Research state | §18.3-18.4 | TOE-Agent | closed state machine |
| Formalization proposal | §19.1 | TOE-Agent | typed schema + persistent ProposalState |
| Formalization response | §19.2 | integration tooling | typed schema + exact outcome vocabulary |
| Projection class | §20.2 | Projection protocol | closed vocabulary |
| Projection loss | §20.7 | Projection protocol | typed reasons |
| Phys trust | §21 | Phys | frozen authority |
| Hypothesis contamination | §21.3 | Frozen Phys | inherited law |
| Theory-local algebra | §22 | phys-* | package-local mapping |
| MathTrace | §23 | producer | authority-local trace |
| Research meta-ledger | §23.3 | TOE-Agent | reference graph |
| ComputationalContract | §24 | Numerical contract package | typed schema |
| Numerical handoff | §24.4 | phys-* / Numerical | in-memory materialization |
| Status vocabularies | §26 | respective owner | separate types |
| Compatibility | §27 | artifact owner | explicit version rules |
| Canonical serialization | §28 | artifact owner | deterministic bytes |
| Artifact completion | §29 | each producer | 13-field template |
| No-Go architecture | §30 | architecture | closed prohibition list |
| Growth Gate timing | §2.3 | Phys governance | post-MVP |

**Closure rule:** if any required concern above lacks a normative decision, this document is not closed.

---

# 33. Acceptance properties

Every architectural acceptance property uses the form:

```text
Given
When
Then
Failure condition
```

## 33.1 Dependency

**Given** the PhysMath module graph.  
**When** dependencies are audited.  
**Then** no production dependency reaches Phys.  
**Failure:** any production import path reaches Phys.

## 33.2 Frozen Phys

**Given** the frozen Phys repository and exact revision recorded by the implementation.  
**When** the architecture integration is applied.  
**Then** no new Phys import, file, API, kernel object, dependency, trust path, or corpus-promotion path is created.  
**Failure:** any architecture-driven mutation of Phys.

## 33.3 Sort inventory

**Given** a PhysMath `SortID`.  
**When** the v1 validator checks it.  
**Then** exactly the 16 IDs in §9 are accepted and any 17th candidate is rejected.  
**Failure:** validator accepts or requires a sort outside the closed 16-value set.

## 33.4 Expression tags

**Given** any of the ten v1 Expression node kinds.  
**When** canonical serialization runs.  
**Then** the exact tag and field order in §9.2 are emitted.  
**Failure:** alternate tags, field order, or string-name discriminators are accepted.

## 33.5 Artifact identity and framing

**Given** identical canonical payload bytes and identical namespace/type/version.  
**When** digesting twice.  
**Then** the digest is byte-identical.  
**Failure:** different digest.

**Given** framed strings such as `"a" + "bc"` versus `"ab" + "c"`.  
**When** the identity framing is computed.  
**Then** the length-prefixed framing distinguishes them.  
**Failure:** alternate unframed concatenation yields the same canonical header bytes.

## 33.6 Embedded ArtifactRef

**Given** a parent artifact containing an ArtifactRef.  
**When** canonicalizing the parent.  
**Then** the embedded ref uses §6.3 ordering `Namespace, SchemaVersion, ArtifactType, Digest`.  
**Failure:** host-language struct declaration order or JSON representation changes embedded-ref bytes.

## 33.7 Enum and tagged-union encoding

**Given** a semantic enum such as `ResearchState`.  
**When** serialized.  
**Then** it is encoded as `String(enum-token)`.  
**Failure:** implementation-specific ordinal bytes are accepted as an equivalent canonical form.

**Given** `NumericScalar`.  
**When** serialized.  
**Then** it begins with exactly `0x01` for Rational or `0x02` for Float64Bits.  
**Failure:** a variant is emitted without a discriminator.

## 33.8 Collection semantics

**Given** a SET containing the same members in different source orders.  
**When** canonicalized.  
**Then** canonical identity is identical.  
**Failure:** SET insertion order changes the digest.

**Given** a SEQUENCE containing the same members in two different orders.  
**When** canonicalized.  
**Then** identity differs when order is semantically significant.  
**Failure:** canonicalization silently sorts a SEQUENCE.

**Given** a SET with duplicate members.  
**When** validated.  
**Then** the artifact is rejected unless its schema explicitly defines semantic deduplication.  
**Failure:** the generic serializer silently deduplicates.

## 33.9 Parameter identity

**Given** two parameters in the same Function at declaration ordinals 0 and 1.  
**When** ParameterIDs are computed.  
**Then** they differ.  
**Failure:** two same-scope declarations collide.

**Given** a parameter whose display label changes from `t` to `tau`.  
**When** identity is recomputed.  
**Then** ParameterID is unchanged.  
**Failure:** display-name change alters identity.

**Given** two independent alpha-equivalent Function declarations in different artifacts.  
**When** ParameterIDs are computed.  
**Then** equal structural ParameterIDs MAY result, and this does not imply shared provenance.  
**Failure:** implementation assumes ParameterID equality alone identifies the declaration artifact.

## 33.10 Differential t/k distinction

**Given** distinct parameters P and Q within one declaration scope.  
**When** `differentiate` targets P versus Q.  
**Then** the resulting operator applications are structurally distinct.  
**Failure:** display strings such as `t` and `k` are used as the mathematical identity mechanism.

## 33.11 Mathematical purity

**Given** a PhysMath expression containing a symbol named `c`.  
**When** the mathematical engine processes the symbol.  
**Then** no physical constant semantics are inferred from the spelling.  
**Failure:** mathematical behavior changes because the spelling is `c`.

## 33.12 Relation dual role

**Given** a top-level Relation and an embedded Relation with identical semantic content.  
**When** serialized.  
**Then** the relation-body encoding is identical but the ArtifactRefs differ because the outer artifact families are `physmath/relation/1` and `physmath/expression/1`.  
**Failure:** both roles collapse into one artifact type or digest.

## 33.13 Projection

**Given** `Pow(x, y)` where `y` is a symbolic Expression.  
**When** strict projection runs.  
**Then** result is `UNREPRESENTABLE` with `NON_RATIONAL_EXPONENT_IN_POW`.  
**Failure:** projection succeeds or rewrites the source silently.

**Given** an `Add` with three canonical children.  
**When** projected.  
**Then** output is the deterministic left-associative Phys tree using canonical child order.  
**Failure:** output association or child ordering varies.

**Given** source Symbol binding metadata not represented by Phys.  
**When** projection runs.  
**Then** metadata is ignored as non-semantic strengthening unless its loss would merge distinct source semantics.  
**Failure:** every bound source symbol is incorrectly classified unrepresentable solely because Phys stores display names.

## 33.14 Condition finalization

**Given** equivalent generated conditions in arbitrary traversal order.  
**When** finalization runs.  
**Then** each condition becomes `physmath/condition/1`, is digested, sorted by raw digest, and adjacent duplicates are removed.  
**Failure:** traversal order determines final condition order.

## 33.15 Formalization target

**Given** a v1 formalization proposal.  
**When** `RequestedTarget` is validated.  
**Then** only `phys/object/1` is accepted.  
**Failure:** `phys/object/1` or another invented type is accepted.

## 33.16 Formalization proposal persistence

**Given** a DRAFT proposal.  
**When** submitted.  
**Then** a new immutable proposal artifact with `ProposalState = SUBMITTED` and `PreviousStateRef` to the DRAFT artifact is created.  
**Failure:** the original proposal is mutated.

**Given** a SUBMITTED proposal.  
**When** resolved.  
**Then** a new immutable proposal artifact with `ProposalState = RESOLVED` references the SUBMITTED state through `PreviousStateRef`.  
**Failure:** the proposal transitions in place.

## 33.17 Formalization outcome separation

**Given** a formalization response with `Outcome = REJECTED`.  
**When** the research ledger is updated.  
**Then** no automatic `ResearchState` transition occurs; any research-state change is represented by a new explicit `ResearchHypothesis` artifact.  
**Failure:** response outcome is treated as a ResearchState enum value.

## 33.18 Formalization revision lineage

**Given** a resolved proposal with `REQUEST_REVISION`.  
**When** the next revision is created.  
**Then** `ParentProposalRef` points to the prior revision while the new proposal begins in `DRAFT`.  
**Failure:** `ParentProposalRef` is reused for same-revision state transitions.

## 33.19 Phys minting

**Given** `hypothesis.NewCandidateConcept` exists in the frozen Phys revision.  
**When** formalization succeeds.  
**Then** the output is hypothesis-grade and never treated as an established corpus object.  
**Failure:** formalization code promotes the object into the trusted corpus.

## 33.20 Trace boundary

**Given** a PhysMath MathTrace and a TOE-Agent ResearchStep referring to the same output.  
**When** replay is requested.  
**Then** PhysMath replays its own operation; TOE-Agent replays only the research graph.  
**Failure:** TOE-Agent requires or implements another authority's internal replay engine.

## 33.21 Numerical handoff

**Given** an in-process ComputationalContract.  
**When** Numerical executes it.  
**Then** all numeric arrays are already materialized and Numerical performs no ArtifactRef lookup.  
**Failure:** execution depends on implicit resolution.

## 33.22 Numerical constant boundary

**Given** an exact mathematical or physical constant required by a numerical model.  
**When** the ComputationalContract is constructed.  
**Then** the producer deterministically materializes the chosen numeric value.  
**Failure:** Numerical invents or performs a symbolic constant mapping.

## 33.23 Float validation

**Given** a `Float64Bits` value.  
**When** the canonical payload is validated.  
**Then** only finite IEEE-754 binary64 values are accepted and bytes are big-endian.  
**Failure:** NaN, ±Inf, or implementation-dependent byte order is accepted.

## 33.24 Frozen trust boundary

**Given** a hypothesis-grade object.  
**When** it enters the formalization or correspondence protocol.  
**Then** the protocol does not create or mutate Phys trust/corpus authority.  
**Failure:** integration code invents a trust-promotion API.

## 33.25 No-Invention completeness

**Given** any architecture concern listed in the No-Invention Matrix.  
**When** implementation begins.  
**Then** the concern has an exact normative decision, owner, schema, validation rule, and acceptance property.  
**Failure:** the coding agent must choose between two architecturally legitimate implementations.

# 34. Final blocker-only audit protocol

The final audit is not another architecture-design round. v1.4 is intended to contain the architecture decisions needed for implementation; the audit tests whether any demonstrated blocker remains.

## 34.1 Review question

Each reviewer MUST answer:

> Can a competent coding agent, given only v1.4 and the frozen Phys authority identified in §0.1, still choose between two architecturally legitimate implementations?

## 34.2 Valid blocker definition

A finding is a `BLOCKER` only when it demonstrates at least one:

```text
1. contradiction between two normative rules;
2. two legitimate implementation choices remain;
3. required schema/owner/identity/state/validation is absent;
4. the implementation agent must invent architecture.
```

The finding MUST identify:

```text
exact section
concrete counterexample
failure consequence
```

A preference or optional improvement is not a blocker.

## 34.3 Reviewer set

The standing adversarial reviewers are:

```text
DeepSeek
Gemini
Qwen
Z
```

The project owner is the final human arbiter for disputed blocker classification, consistent with the human-authority principle inherited from the frozen Phys governance model.

## 34.4 Arbitration

There is no majority-vote dismissal of a demonstrated blocker.

If any reviewer demonstrates a valid blocker under §34.2, the architecture remains open until the specific contradiction/choice is resolved.

A reviewer cannot indefinitely block closure by assertion alone; the required counterexample/evidence test is mandatory.

## 34.5 Closure condition

Architecture may be frozen only when all are true:

```text
DeepSeek = 0 demonstrated BLOCKER
Gemini   = 0 demonstrated BLOCKER
Qwen     = 0 demonstrated BLOCKER
Z        = 0 demonstrated BLOCKER

No-Invention Matrix complete
All artifact templates complete
All state machines explicit
All cross-project owners explicit
All canonical identity rules explicit
All required acceptance properties present
```

After these conditions are met, no new closure-plan document is permitted.

---

# 35. Implementation-plan boundary

Once v1.4 is frozen, the implementation plan has one responsibility:

```text
HOW to implement the already-decided architecture
```

It MUST NOT decide:

```text
whether PhysMath imports Phys
who owns ArtifactRef
what ArtifactRef means
whether artifacts are content-addressed
how t/k are distinguished
what the projection subset is
whether Phys can be modified
who owns PhysicalInterpretation
whether formalization promotes trust
who owns ComputationalContract
whether Numerical parses PhysMath
```

The implementation plan MAY decide only:

```text
repository sequencing
file/package layout within the frozen boundaries
implementation order
test fixture organization
CI wiring
golden vector files
migration sequencing where the architecture explicitly permits it
```

The final coding-agent instruction is:

> **Implement v1.4 exactly. When the implementation encounters a condition not covered by v1.4, do not invent an architectural rule; stop and surface the exact missing contract.**

---

# 36. Final architectural invariant

The ecosystem is coherent only when this distinction remains intact:

```text
MATHEMATICS
    asks what mathematical structure and operation are valid

PHYSICAL SEMANTICS
    asks what physical meaning a framework assigns to that structure

TRUST / FORMALIZATION
    records what the trusted Phys substrate formally supports

RESEARCH
    records what a human research program proposes, attacks, revises,
    and attempts to formalize

NUMERICAL
    computes quantitative consequences of already-defined contracts
```

No layer may silently answer the question belonging to another layer.

The architecture is therefore:

```text
PhysMath ⊥ Phys

phys-* = physical meaning over mathematics + trusted references

TOE-Agent = research state over explicit artifacts/contracts

Numerical = computation over explicit executable contracts

ArtifactRef = content-addressed name, never implicit resolution

Physical interpretation = explicit, framework-owned

Trust = never duplicated

Replay = authority-local

Projection = deterministic and closed-world

Mathematical identity = explicit, not textual guesswork

Physical meaning = never inferred from mathematical structure
```

---

# Appendix A — Normative artifact schemas

The following schemas are the concrete v1.4 contract. Each schema version is `"1"` unless explicitly inherited otherwise.

## A.1 `physmath.expression`

```text
ExpressionArtifactPayload {
    SortID       string        // exactly "Expression"
    PayloadBytes []byte        // canonical Expression encoding
}
```

The expression encoding is the recursive tagged union defined in §9.2–§9.11.

## A.2 `physmath.condition`

Identity:

```text
Namespace     = physmath
ArtifactType  = condition
SchemaVersion = 1
```

Canonical schema:

```text
MathCondition {
    Predicate ArtifactRef   // MUST reference physmath/relation/1
}
```

Canonical payload is exactly `EmbeddedArtifactRef(Predicate)`.

## A.3 `physmath.trace`

```text
MathTrace {
    ProducerNamespace      string
    OperationID            string
    OperationSchemaVersion string
    InputRefs              []ArtifactRef   // SEQUENCE
    OutputRef              ArtifactRef
    ParameterBytes         []byte
    ConditionRefs          []ArtifactRef   // SET
}
```

## A.4 `phys.<framework>` PhysicalInterpretation

```text
PhysicalInterpretation {
    InterpretationID     string
    FrameworkRef         ArtifactRef
    PhysicalType         string
    MathRefs             []ArtifactRef
    RequiredConditions   []ArtifactRef
    RequiredConventions  []ConventionRequirement
    Scope                ScopeSpec
}
```

`MathRefs` and `RequiredConditions` are SETs. `RequiredConventions` is sorted by `(Namespace, Key, Value)`.

## A.5 PhysicalContract

```text
PhysicalContract {
    ExtensionID          string
    PhysicalType         string
    FrameworkRef         ArtifactRef
    MathRefs             []ArtifactRef
    RequiredConditions   []ArtifactRef
    RequiredConventions  []ConventionRequirement
    InterpretationClass  string
}
```

## A.6 PhysicalArtifact

```text
PhysicalArtifactPayload {
    PhysicalType         string
    FrameworkRef         ArtifactRef
    ContractRef          ArtifactRef
    MathRefs             []ArtifactRef
    RequiredConditions   []ArtifactRef
    RequiredConventions  []ConventionRequirement
    PayloadBytes         []byte
}
```

## A.7 ProposedInterpretation

```text
ProposedInterpretation {
    ProposalID             string
    SourceMathRefs         []ArtifactRef
    ProposedPhysicalType   string
    ProposedFrameworkRef   ArtifactRef?
    CorrespondenceRefs     []ArtifactRef
    EvidenceRefs           []ArtifactRef
    Rationale              string
}
```

## A.8 CorrespondenceProposal

```text
CorrespondenceProposal {
    ProposalVersion       uint32
    SourceRefs            []ArtifactRef   // SEQUENCE
    TargetRefs            []ArtifactRef   // SEQUENCE
    MappingType           string
    MappingArtifactRefs   []ArtifactRef   // SET
    FrameworkRefs         []ArtifactRef   // SET
    Rationale             string
}
```

The field annotations above are normative. `SourceRefs` and `TargetRefs` preserve declared order because correspondence mappings may be positional. `MappingArtifactRefs` and `FrameworkRefs` are SETs sorted by the embedded `ArtifactRef` ordering rule. There is no blanket rule that converts sequence fields into SETs.

## A.9 ResearchHypothesis

```text
ResearchHypothesis {
    ClaimRefs                    []ArtifactRef   // SET
    ConstraintRefs               []ArtifactRef   // SET
    CorrespondenceRefs           []ArtifactRef   // SET
    ProposedInterpretationRefs   []ArtifactRef   // SET
    Status                       ResearchState
    PreviousStateRef             ArtifactRef?
    CounterexampleRefs           []ArtifactRef   // SET
    InconsistencyRefs            []ArtifactRef   // SET
}
```

All reference collections are SETs. `PreviousStateRef` is the only history edge.

## A.10 FormalizationProposal

```text
FormalizationProposal {
    ProposalRevision       uint32
    ParentProposalRef      ArtifactRef?
    ProducerNamespace      string
    SourceArtifactRefs     []ArtifactRef   // SEQUENCE
    RequestedTarget        string
    ProjectionPolicy       string
    RequiredFrameworkRefs  []ArtifactRef   // SET
    ProposalState           ProposalState
    PreviousStateRef        ArtifactRef?
}
```

MVP exact values:

```text
RequestedTarget  = "phys/object/1"
ProjectionPolicy = "STRICT_MIRROR_ONLY"
SourceArtifactRefs -> physmath/expression/1 only
```

The first DRAFT of a revision has `PreviousStateRef = nil`. A state transition creates a new immutable proposal artifact and sets `PreviousStateRef` to the immediately preceding state artifact of the same revision. A new revision increments `ProposalRevision`, sets `ParentProposalRef` to the prior revision artifact, and starts at DRAFT.

## A.11 ValidationReport

```text
ValidationReport {
    Checks []ValidationCheck
}

ValidationCheck {
    Code    string
    Passed  bool
    Detail  string
}
```

Checks are a SET keyed by `Code`. Duplicate codes are invalid. The v1.4 check-code vocabulary is exactly:

```text
SOURCE_REF_RESOLVED
SOURCE_SCHEMA_SUPPORTED
REQUESTED_TARGET_SUPPORTED
PROJECTION_POLICY_SUPPORTED
PROJECTION_SUCCESS
LOSS_POLICY_SATISFIED
PHYS_API_CONFIRMED
HYPOTHESIS_MINT_SUCCESS
PROVENANCE_CONFORMING
```

## A.12 ProjectionLossReport

```text
ProjectionLossReport {
    Classification ProjectionClass
    Reasons        []ProjectionReason
}

ProjectionReason {
    Code     string
    NodeKind string
    Detail   string
}
```

Reasons are sorted by `(Code, NodeKind, Detail)`.

## A.13 FormalizationResponse

```text
FormalizationResponse {
    ProposalRef        ArtifactRef
    ProposalRevision   uint32
    Outcome            FormalizationOutcome
    ProjectionClass    ProjectionClass
    ProjectedRefs      []ArtifactRef   // SEQUENCE; one element at most in MVP
    LossReport         ProjectionLossReport
    ValidationReport   ValidationReport
    ProducerNamespace  string
}
```

`Outcome` is exactly one of:

```text
REQUEST_REVISION
REJECTED
UNREPRESENTABLE
PROJECTED_AS_HYPOTHESIS
```

## A.14 ResearchStep

```text
ResearchStep {
    Operation          string
    InputRefs          []ArtifactRef
    OutputRef          ArtifactRef
    ProducerNamespace  string
}
```

`InputRefs` preserves declared research-step sequence; it is not sorted because order can carry research provenance meaning.

## A.15 Numerical ComputationalContract

The authoritative schema is owned by `github.com/PithomLabs/numerical/contract`, namespace `numerical`, artifact type `contract`, schema version `1`. Its exact payload is:

```text
ComputationalContract {
    PhysicalModelRef     ArtifactRef
    MathematicalModelRef ArtifactRef
    Inputs               []InputBinding
    Parameters           []NumericParameter
    RequestedOutputs     []OutputRequest
    NumericalMethod      string
    Precision            PrecisionRequirement
    ExecutionLimits      ExecutionLimits
    ProvenanceRefs       []ArtifactRef
}

InputBinding {
    Name        string
    ElementType string
    Shape       []uint64
    Data        []NumericScalar
}

NumericParameter {
    Name  string
    Value NumericScalar
}

OutputRequest {
    Name string
}

PrecisionRequirement {
    AbsoluteTolerance NumericScalar?
    RelativeTolerance NumericScalar?
    MaxIterations     uint64?
}

ExecutionLimits {
    MaxSteps      uint64?
    MaxElements   uint64?
    MaxWallUnits  uint64?
}

NumericScalar = Rational | Float64Bits
Float64Bits    = UInt64 IEEE-754 binary64 bit pattern

Canonical `NumericScalar` encoding is exactly:

0x01 || Canonical(Rational)
0x02 || UInt64BE(Float64Bits)


The in-process form contains materialized `Data`; the `ArtifactRef`s are identity/provenance metadata and are not execution-time lookup instructions.

# Appendix B — Exact RuleID and operator catalogue

Rule IDs are concrete serialized identifiers. The family labels below are documentation groupings only.

## B.1 Exact RuleIDs

The v1.4 rule registry is exactly the following set:

```text
ID-001
EXPR-001
EXPR-002
EXPR-003
EXPR-004
EXPR-005
FN-001
FN-002
FN-003
FA-001
FA-002
FA-003
FNL-001
FNL-002
FNL-003
FNLA-001
FNLA-002
FNLA-003
OP-001
OP-002
OP-003
OPA-001
OPA-002
OPA-003
OPA-004
TEN-001
TEN-002
TEN-003
IDX-001
IDX-002
IDX-003
MAP-001
MAP-002
MAP-003
REL-001
REL-002
REL-003
REL-004
SET-001
SET-002
SET-003
TR-001
TR-002
TR-003
LIM-001
LIM-002
LIM-003
LIM-004
ASY-001
ASY-002
ASY-003
DO-001
DO-002
DO-003
PROJ-001
```

`EXPR-005` is the explicit double-negation normalization subrule invoked by §9.6 and is governed by the Expression canonicalization rule. It is not a new sort or a new operation.

Every other RuleID above appears as a concrete identifier in the normative sort matrix or identity/projection machinery. A wildcard such as `EXPR-*` is documentation only and MUST NOT be serialized as a RuleID.

## B.2 PhysMath OperatorID registry

The closed PhysMath OperatorID vocabulary in v1.4 is exactly:

```text
differentiate
```

Its namespace is `physmath`. No second PhysMath operator vocabulary is permitted.
# Appendix C — PhysMath v1 sort contract matrix

The matrix is normative. Rule-family identifiers are defined in Appendix B and resolve to the cited normative sections.

| Sort | Construct | Canonicalize | Equality | Substitute | Serialize | Project |
|---|---|---|---|---|---|---|
| Expression | `RULE EXPR-001` | `RULE EXPR-002` (includes `EXPR-005`) | `RULE EXPR-003` | `RULE EXPR-004` | `RULE ID-001` | `RULE PROJ-001` |
| Function | `RULE FN-001` | `RULE FN-002` | `RULE FN-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| FunctionApplication | `RULE FA-001` | `RULE FA-002` | `RULE FA-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Functional | `RULE FNL-001` | `RULE FNL-002` | `RULE FNL-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| FunctionalApplication | `RULE FNLA-001` | `RULE FNLA-002` | `RULE FNLA-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Operator | `RULE OP-001` | `RULE OP-002` | `RULE OP-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| OperatorApplication | `RULE OPA-001` | `RULE OPA-002` | `RULE OPA-003` | `RULE OPA-004` | `RULE ID-001` | `UNREPRESENTABLE` |
| Tensor | `RULE TEN-001` | `RULE TEN-002` | `RULE TEN-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Index | `RULE IDX-001` | `RULE IDX-002` | `RULE IDX-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Map | `RULE MAP-001` | `RULE MAP-002` | `RULE MAP-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Relation | `RULE REL-001` | `RULE REL-002` | `RULE REL-003` | `RULE REL-004` | `RULE ID-001` | `UNREPRESENTABLE` |
| SetCollection | `RULE SET-001` | `RULE SET-002` | `RULE SET-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Transformation | `RULE TR-001` | `RULE TR-002` | `RULE TR-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| Limit | `RULE LIM-001` | `RULE LIM-002` | `RULE LIM-003` | `RULE LIM-004` | `RULE ID-001` | `UNREPRESENTABLE` |
| AsymptoticTruncation | `RULE ASY-001` | `RULE ASY-002` | `RULE ASY-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |
| DerivationObligation | `RULE DO-001` | `RULE DO-002` | `RULE DO-003` | `DEFERRED + CONTAINMENT` | `RULE ID-001` | `UNREPRESENTABLE` |

The matrix fixes the v1.4 sort inventory and prevents an implementation plan from silently adding another core sort.

---

# Appendix D — Durable artifact family checklist

Every durable family below requires all thirteen fields from §29.

```text
MathArtifact
PhysicalArtifact
PhysicalInterpretation
PhysicalContract
ProposedInterpretation
CorrespondenceProposal
ResearchHypothesis
FormalizationProposal
FormalizationResponse
ProjectionLossReport
ValidationReport
MathTrace
ResearchStep
ComputationalContract
```

`TrustAssertion` is deliberately excluded from the current artifact family set because it is a future design target.

---

# Appendix E — Closed MVP operation-generated assumption keys

The only operation-generated assumption-key forms are:

```text
denominator/<hash>
selected_branch/<hash>
```

Generation ownership:

```text
Divide      -> denominator/<hash>
SelectBranch -> selected_branch/<hash>
```

No other MVP operation creates a new assumption key.

---

# Appendix F — Frozen Phys constructor/trust guard

The following invariant is normative for integration tooling:

```text
Manifest-backed established object
    -> established only through frozen Phys corpus contract

Hypothesis concept
    -> HYPOTHESIS / NONE

External phys-* object
    -> external framework semantics
    -> not silently established Phys corpus
```

A coding agent MUST NOT construct an apparently established Phys object by calling a convenience constructor whose corpus status is not guaranteed by the frozen Phys specification.

---

# Appendix G — Final handoff checklist

Before the implementation plan is accepted, verify mechanically:

```text
[ ] v1.4 is the sole current architecture document
[ ] frozen Phys authority is identified
[ ] Phys remains byte-frozen
[ ] PhysMath ⊥ Phys
[ ] ArtifactRef has one owner
[ ] ArtifactRef identity is content-addressed
[ ] one digest framing rule exists
[ ] one embedded-ref encoding exists
[ ] namespaces are enumerated
[ ] Phys referenceable artifact type is pinned
[ ] PhysMath sort set is pinned
[ ] Expression node set is pinned
[ ] ParameterID construction is pinned
[ ] t/k distinction is identity-based
[ ] MathCondition lifecycle is pinned
[ ] condition ordering/deduplication is pinned
[ ] PhysicalInterpretation ownership is pinned
[ ] ProposedInterpretation ownership is pinned
[ ] PhysicalContract schema is pinned
[ ] formalization proposal schema is pinned
[ ] formalization response schema is pinned
[ ] proposal state machine is pinned
[ ] research state machine is pinned
[ ] operation outcomes are explicitly terminal
[ ] projection subset is pinned
[ ] projection compositional rule is pinned
[ ] partial projection policy is pinned
[ ] projection reason vocabulary is pinned
[ ] provenance law is inherited from frozen Phys
[ ] no new Phys API is invented
[ ] theory-local algebras remain package-local
[ ] MathTrace uses ArtifactRef
[ ] replay remains authority-local
[ ] ComputationalContract owner is pinned
[ ] Numerical dependency edge is pinned
[ ] Numerical does not parse PhysMath
[ ] Growth Gate is post-MVP
[ ] phys-interop is not an MVP runtime dependency
[ ] status vocabularies are separate
[ ] canonical serialization rules are pinned
[ ] No-Invention Matrix complete
[ ] acceptance properties executable
[ ] final four-agent blocker-only audit complete with zero demonstrated blockers
```

---

# Appendix H — Closure status

This document is the v1.4 architecture specification, not a closure-plan layer. It supersedes v1.3 where the two conflict and incorporates the consolidated blocker review decisions.

The remaining procedural step is the final blocker-only audit in §34. A finding that demonstrates a genuine unresolved architectural choice reopens v1.4 at that exact decision. A request for implementation convenience, stylistic preference, or future capability does not reopen the architecture.

No separate closure-plan document is permitted. Once the §34 closure condition is met, v1.4 is frozen and the implementation plan may describe only implementation sequencing and repository mechanics within these locked boundaries.

**END OF PM-ARCH-FOUNDATION-001 v1.4**
