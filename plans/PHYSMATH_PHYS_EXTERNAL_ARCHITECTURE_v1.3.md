# PhysMath / Phys / phys-* / Numerical / TOE-Agent Architecture
## Foundation Contract — v1.3

**Document ID:** PM-ARCH-FOUNDATION-001  
**Version:** 1.3  
**Status:** **ARCHITECTURE SPECIFICATION — PROPOSED FOR FINAL BLOCKER-ONLY AUDIT**  
**Purpose:** Define the cross-project architecture, schemas, identity rules, ownership boundaries, state machines, projection protocol, and numerical boundary so that an implementation agent is not required to invent an architectural decision.

> **This document is the architecture. It is not another closure plan. No architecture-closure document follows v1.3. After the final blocker-only audit described in §34, this document is either frozen or explicitly rejected and amended; an additional closure-plan layer is prohibited.**

---

# 0. Normative language and source authority

The words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are normative.

This document contains three kinds of decisions:

| Lineage | Meaning |
|---|---|
| `INHERITED-PHYS` | Constraint inherited from the frozen Phys authority; v1.3 does not redefine it. |
| `INHERITED-ARCH` | Architectural decision carried forward from the reviewed architecture chain. |
| `V1.3-DECIDED` | New decision made explicitly by this specification to remove remaining implementation ambiguity. |

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
2. this v1.3 architecture for cross-project architecture
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

There is no alternate `artifact_id` mechanism in the v1.3 architecture.

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

There is exactly one normative digest construction.

## 6.1 Canonical payload

Every durable artifact schema defines `CanonicalPayloadBytes`.

These are the exact bytes of the artifact payload after schema-specific canonicalization.

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

There is no second abstract concatenation formula.

## 6.3 Embedded ArtifactRef encoding

When an `ArtifactRef` appears inside another canonical payload, it is encoded as:

```text
Len32(Namespace)
|| UTF8(Namespace)
|| Len32(SchemaVersion)
|| UTF8(SchemaVersion)
|| Len32(ArtifactType)
|| UTF8(ArtifactType)
|| Digest[32]
```

Embedded refs MUST NOT be encoded as:

- JSON objects;
- JSON strings;
- hex strings;
- unframed digest bytes.

Transport JSON MAY display an ArtifactRef in a documented lossless form, but that transport display MUST NOT be substituted into canonical identity bytes.

## 6.4 Canonical collection ordering

Every ordered canonical collection MUST declare its order.

Where the architecture specifies digest ordering, the comparison is lexicographic comparison of the raw 32 digest bytes.

No map iteration order is permitted to affect identity.

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

---

# 7. Canonical primitive encoding

All v1.3 artifact schemas use the following primitive canonical encoding rules before the outer digest framing in §6.2.

```text
String(s)      = Len32(UTF8(s)) || UTF8(s)
Bytes(b)       = Len32(len(b)) || b
UInt8(x)       = one byte, unsigned
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

Canonical struct fields appear in the exact declaration order specified by the artifact schema.

For optional fields, the presence marker is always emitted even when absent. There is no implicit default omission.

For arbitrary-precision integers, the sign byte is `0x00` for non-negative and `0x01` for negative, followed by `Len32(magnitude_bytes)` and the minimal big-endian unsigned magnitude bytes. Zero is always encoded as non-negative magnitude zero.

Canonical schemas MUST NOT contain Go maps.

Collections have one of two explicitly declared semantics:

```text
SEQUENCE = preserve declared order
SET      = sort by the canonical ordering specified by that field
```

Where a field is a collection of `ArtifactRef`s and the schema does not explicitly mark the field as a sequence, it is a `SET` and is sorted by the full embedded `ArtifactRef` tuple `(Namespace, ArtifactType, SchemaVersion, Digest)` using lexicographic byte/string order followed by raw digest order.

A field explicitly marked `SEQUENCE` preserves its declared order. The implementation MUST NOT sort such a field merely for determinism.

For optional `ArtifactRef?`, the canonical presence byte from §7 is followed by the embedded-ref encoding only when present.

This rule removes all transport-library and language-runtime dependence from canonical identity.

# 7. Namespace registry and ownership

There is no universal runtime registry.

The following namespace families are reserved by architecture:

| Namespace | Owner | Artifact-type policy |
|---|---|---|
| `physmath` | PhysMath | Types listed in §9 |
| `phys` | frozen Phys | Only frozen referenceable types listed in §8 |
| `phys.qm` | phys-qm | Declared by package contract |
| `phys.bohm` | phys-bohm | Declared by package contract |
| `phys.gr` | phys-gr | Declared by package contract |
| `phys.cst` | phys-cst | Declared by package contract |
| `phys.cdt` | phys-cdt | Declared by package contract |
| `toe` | TOE-Agent | Research/protocol types listed in §18 |
| `numerical` | Numerical | `numerical.contract` in §24 |

`phys-artifact` does not own a semantic namespace.

A new `phys-*` project MUST declare exactly one namespace matching its project identity and MUST publish its artifact-type vocabulary in its package contract.

A new ecosystem namespace requires a v1.3 architecture amendment and cannot be silently invented by an implementation agent.

## 7.1 PhysMath artifact-type mapping

Within namespace `physmath`, the artifact type for a sort is fixed by this table:

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

`math.condition` and `math.trace` are additional meta-artifact types defined in Appendix A and are not mathematical expression sorts. Their payload contracts are fixed by Appendix A.

---

# 8. Frozen Phys artifact references

Phys remains a reference target and does not import `phys-artifact`.

Consumers construct ecosystem `ArtifactRef`s for frozen Phys objects using the frozen Phys canonical bytes.

## 8.1 MVP referenceable Phys artifact type

The v1.3 ecosystem contract defines exactly one Phys artifact type for external reference in MVP:

```text
Namespace    = phys
ArtifactType = object
SchemaVersion = 1
```

Its payload bytes are the exact canonical object bytes defined by the frozen Phys specification for `core.Object`.

The consumer MUST NOT reinterpret, reserialize, normalize, or otherwise modify those canonical object bytes before applying the v1.3 ecosystem digest framing.

The ecosystem digest for a Phys object is not required to equal any internal Phys object hash. It is a new ecosystem identity computed over the exact frozen Phys canonical bytes using the v1.3 framing in §6.2.

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

The v1.3 core sort families are fixed to the following set:

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

The set is **closed for schema version 1**.

A future PhysMath schema version MAY add a sort only through an explicit architecture amendment.

A `phys-*` package MUST NOT inject a new runtime sort into PhysMath v1.

## 9.1 Sort-table cell vocabulary

Each sort's behavioral contract uses only:

```text
N/A
RULE <RuleID>
DEFERRED + CONTAINMENT
```

`yes`, `supported`, `handled`, or equivalent free-form labels are not normative cells.

## 9.2 Expression node kinds

`Expression` is a tagged mathematical union.

MVP node kinds are:

```text
Symbol
Rational
Add
Mul
Neg
Pow
Sqrt
Call
Relation
BranchSet
```

`Symbol` is an Expression node kind, not a separate top-level sort.

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

Exactly one of `ParameterID` and `ConstantID` MAY be present for a bound symbol. Persistent expressions MUST bind every free symbol to an explicit `ParameterID` or `ConstantID`; a bare display name is not a persistent mathematical identity.

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

`Add` and `Mul` contain canonical n-ary child collections.

Canonicalization MUST flatten nested nodes of the same associative kind.

Children MUST be deterministically ordered by canonical expression digest bytes.

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

`Neg` has exactly one expression operand.

A canonicalizer MAY remove double negation only through an explicit normalization rule; construction itself MUST preserve structure until normalization.

## 9.7 Pow

`Pow` contains:

```text
Base
Exponent
```

The PhysMath representation is broader than the frozen Phys projection domain.

For Phys projection, the exponent MUST satisfy the exact frozen Phys `Pow` contract. In MVP this means the exponent is a `Rational` literal.

A non-rational exponent is classified:

```text
ProjectionClass = UNREPRESENTABLE
ReasonCode      = NON_RATIONAL_EXPONENT_IN_POW
```

## 9.8 Sqrt

`Sqrt` contains one radicand expression.

Projection maps `Sqrt(x)` to the frozen Phys `Sqrt(x)` representation exactly.

## 9.9 Call

`Call` contains:

```text
FunctionRef
Args []Expression
```

The function reference is mathematical identity, not a physical semantic label.

Call is not in the strict Phys mirror.

For example, `sin(x)` may be represented as a mathematical call in PhysMath or a theory-local algebra, but it MUST NOT be projected to Phys without an explicit future bridge rule.

## 9.10 Relation

A Relation is a mathematical proposition containing:

```text
Operator
Left
Right
```

with a closed comparison operator vocabulary:

```text
EQ
NE
LT
LTE
GT
GTE
```

Relations are mathematical objects. Their truth is not a physical-trust assertion.

## 9.11 BranchSet

A `BranchSet` contains an ordered set of candidate expressions together with no implicit selection.

Branch selection is performed only by an explicit transformation operation.

No branch is “obviously physical” by naming convention.

---

# 10. Higher mathematical sort contracts

The following sorts are schema families in v1.3. Their exact behavioral surface is fixed by the rule vocabulary in Appendix A; operations outside those rows are deferred rather than invented.

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
    Inputs      []ArtifactRef
    Output      ArtifactRef
    RuleID      string
    Parameters  CanonicalParameters
}

CanonicalParameters is a byte string produced by the named transformation rule. Its schema, field order, and valid values are defined by the `RuleID`. A transformation MUST reject parameters that are not valid for its `RuleID`; there is no open-ended parameter bag.

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

Parameter identity is the mechanism that distinguishes mathematically different variables such as `t` and `k` without putting physical meanings into PhysMath.

## 11.1 ParameterID

```go
type ParameterID [32]byte
```

A `ParameterID` is deterministic.

It is derived from a canonical declaration record:

```text
Namespace
ScopePath
DeclarationOrdinal
```

where:

```text
ScopePath = ordered binder ordinals from the artifact root to the declaration scope
DeclarationOrdinal = zero-based declaration position in that scope
```

`ScopePath` is encoded as a sequence of zero-based binder ordinals. Its canonical bytes are:

```text
UInt32BE(len(ScopePath))
|| UInt32BE(ScopePath[0])
|| ...
|| UInt32BE(ScopePath[n-1])
```

`ScopePathBytes` therefore uses fixed-width 32-bit elements; its length prefix is the number of elements, not the number of encoded bytes.

The complete canonical byte sequence is:

```text
String(Namespace)
|| Bytes(ScopePathBytes)
|| UInt32BE(DeclarationOrdinal)
```

and:

```text
ParameterID = SHA-256(the bytes above)
```

`DisplayName` is deliberately excluded.

## 11.2 Scope rule

Each parameter binding is unique within its declaration path.

Two occurrences refer to the same parameter only when they carry the same `ParameterID`.

Textual spelling does not establish identity.

## 11.3 Alpha-renaming

Renaming:

```text
t -> tau
```

changes display text only.

It MUST NOT change `ParameterID`.

## 11.4 Cross-artifact parameter reuse

A parameter reused across artifacts MUST be referenced by its declaration artifact or stable `ParameterID`.

A new artifact MUST NOT infer parameter equivalence from matching display names.

## 11.5 Free symbols

Ephemeral interactive expressions MAY contain an unbound display symbol. A durable PhysMath artifact MUST NOT: every free symbol MUST be bound to an explicit `ParameterID` or `ConstantID` before persistence or inclusion in a durable transformation trace.

A coding agent MUST NOT generate random UUIDs for mathematical parameter identity.

---

# 12. Differential operators: t-flow and k-flow

PhysMath contains no physical-domain enum such as `TIME` or `RG_SCALE`.

Physical frameworks assign physical roles externally.

## 12.1 Differential operator representation

A differential operator is represented as an `OperatorApplication` whose canonical operator identity is `differentiate` and whose canonical parameters contain:

```text
Target ParameterID
Order  UInt32
```

`Order >= 1`. Multiple derivative applications are represented by repeated applications or by an explicitly canonicalized higher order value; both forms MUST normalize to the same v1 canonical form through the differentiation rule.

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

`MathCondition` is a pure mathematical object.

It is not a Phys assumption and contains no provenance/corpus/trust metadata.

## 14.1 Condition schema

```text
MathCondition {
    Predicate ArtifactRef
}
```

The referenced predicate MUST resolve to a mathematical `Relation` or an explicitly permitted mathematical proposition representation.

## 14.2 TransformResult

Mathematical transformations that can generate conditions return a value-equivalent result containing:

```text
TransformResult {
    Output
    Conditions
    Trace
}
```

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
3. sort by lexicographic Digest[32] order;
4. remove adjacent duplicate digests.
```

Thus duplicate generated conditions produce one canonical condition entry.

Tree traversal order MUST NOT determine final condition ordering.

---

# 15. MathArtifact and shared mathematical types

## 15.1 MathArtifact

A durable mathematical artifact is an artifact-family-specific payload under namespace `physmath`. Its common logical form is:

```text
MathArtifact {
    SortID       string
    PayloadBytes []byte
}
```

`SortID` MUST be one of the 17 v1 sort IDs in §9. `PayloadBytes` are the canonical bytes of that sort's schema. `ArtifactType` identifies the artifact family; `SortID` MUST agree with it.

The `ArtifactRef` is not stored inside `PayloadBytes`. It is derived from the payload and schema metadata.

## 15.2 MathTypeRef

```text
MathTypeRef = string
```

Valid v1 values are exactly the 17 sort IDs in §9. `MathTypeRef` is a structural mathematical type name; it has no physical semantics.

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

There is no generic `CompatibilityRules` engine in v1.3.

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
    ProposalRevision      uint32
    ParentProposalRef     ArtifactRef?
    ProducerNamespace     string
    SourceArtifactRefs    []ArtifactRef   // SEQUENCE
    RequestedTarget       string
    ProjectionPolicy      string
    RequiredFrameworkRefs []ArtifactRef
}

MVP constraint: every `SourceArtifactRefs` entry MUST reference a `physmath.expression` artifact. A physical artifact or theory-local artifact is not directly projected into Phys by the MVP formalization protocol.
```

MVP pins:

```text
RequestedTarget  = "phys.hypothesis_object"
ProjectionPolicy = "STRICT_MIRROR_ONLY"
```

No other values are valid in MVP.

## 19.2 FormalizationResponse

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

For MVP, `ProjectedRefs` contains exactly one `phys/object/1` reference only when `Outcome = PROJECTED_AS_HYPOTHESIS`; all other outcomes contain zero projected references. Because MVP permits zero information-loss projections, `PARTIALLY_PROJECTABLE` is unreachable in MVP but remains reserved for a future versioned projection policy.
```

The response is immutable and content-addressed.

## 19.3 Proposal state

Every persisted proposal has:

```text
ProposalState
ProposalOutcome
```

Legal `ProposalState` values:

```text
DRAFT
SUBMITTED
RESOLVED
```

A proposal MAY only transition:

```text
DRAFT -> SUBMITTED
SUBMITTED -> RESOLVED
```

`RESOLVED` is terminal.

`ProposalOutcome` is one of:

```text
REQUEST_REVISION
REJECTED
PROJECTABLE
PARTIALLY_PROJECTABLE
UNREPRESENTABLE
PROJECTED_AS_HYPOTHESIS
```

For an MVP successful formalization path, the required terminal pair is:

```text
(RESOLVED, PROJECTED_AS_HYPOTHESIS)
```

For a failed revision path:

```text
(RESOLVED, REQUEST_REVISION)
```

## 19.4 Revision lineage

When an evaluation returns `REQUEST_REVISION`:

```text
parent proposal -> (RESOLVED, REQUEST_REVISION)
new proposal   -> DRAFT with ParentProposalRef = parent ArtifactRef
```

A resolved proposal MUST NOT be resubmitted or mutated.

This rule applies equally to successful, rejected, and revision-requested proposals.

## 19.5 Formalization ownership

Project A owns proposal/research state.

The implementation of the formalization protocol is integration tooling using only the existing public Phys API and existing frozen Phys contracts.

No new Phys API is introduced by this architecture.

A reviewer suggestion is not evidence that an API exists. The implementation plan MUST name only APIs actually present in the frozen Phys authority.

## 19.6 Phys hypothesis minting

When confirmed against the frozen Phys specification, the raw public hypothesis-grade mint path is:

```text
hypothesis.NewCandidateConcept
```

External tooling MUST NOT invent a new Phys mint path.

The resulting content is hypothesis-grade under the frozen Phys rules and is not corpus promotion.

If the frozen Phys specification revision supplied to the implementation plan does not expose this exact path, the implementation plan MUST stop rather than inventing an alternative API.

---

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

This rule is inherited from the frozen Phys authority, not created by v1.3:

> A session-recorded correspondence operating on hypothesis-grade operands remains hypothesis-grade under the frozen Phys contamination law.

Therefore a hypothesis-grade formalization path MUST NOT expect an `IDENTIFIED` or `ESTABLISHED` result merely because the mapping was successfully recorded.

A protocol implementation MUST use the exact contamination behavior of the frozen Phys specification.

## 21.4 TrustAssertion

`TrustAssertion` is a future design target only.

It is not an existing v1.3 runtime artifact, not an accepted currency, and not an invitation to add:

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

```text
MathTrace {
    ProducerNamespace    string
    OperationID           string
    OperationSchemaVersion string
    InputRefs             []ArtifactRef
    OutputRef             ArtifactRef
    ParameterBytes        []byte
    ConditionRefs         []ArtifactRef
}
```

`MathObjectRef` is not a separate type in v1.3. Durable mathematical references use `ArtifactRef`.

## 23.2 Authority replay

Each producer owns replay of its own operation vocabulary.

```text
PhysMath -> replays PhysMath operations
phys-gr  -> replays phys-gr operations
Phys     -> replays Phys operations
```

No universal replay engine is required.

## 23.3 Research meta-ledger

TOE-Agent may maintain a research meta-ledger:

```text
ResearchStep {
    Operation       string
    InputRefs       []ArtifactRef
    OutputRef       ArtifactRef
    ProducerNamespace string
}
```

The meta-ledger replays the **research graph**, not the internal semantics of every authority.

TOE-Agent MUST NOT attempt to replay Phys or phys-* internal operation implementations merely because it references their outputs.

---

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

MVP supports:

```text
Rational
Float64Bits
```

`Float64Bits` is a tagged 64-bit IEEE-754 representation stored canonically as an unsigned 64-bit value in big-endian byte order.

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

For the mechanics package specifically, the object-level relativity keys are not present:

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
ProposalOutcome
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

For all schemas introduced by v1.3, the canonical payload is a deterministic binary schema-encoding defined by the artifact owner.

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

| Field | Required v1.3 rule |
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

The thirteen-field template is inherited by every artifact family listed in Appendix D.

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

v1.3 supersedes the corresponding reviewed decisions in the preceding architecture chain where they conflict with the normative rules above.

The supersession table is part of the architecture, not merely historical notes.

| Earlier decision | v1.3 decision | Lineage |
|---|---|---|
| PhysMath directly depending on Phys | Neither imports the other | `INHERITED-ARCH` |
| Phys artifact identity = id or digest | Content-addressed ArtifactRef only | `V1.3-DECIDED` |
| ArtifactRef unresolved owner | `github.com/PithomLabs/phys-artifact/artifact` | `V1.3-DECIDED` |
| ArtifactRef as resolver | ArtifactRef is a name; resolution is environmental | `V1.3-DECIDED` |
| Phys imports artifact-reference module | Phys remains frozen and imports nothing new | `V1.3-DECIDED` |
| Closed physical differential enum in PhysMath | Parameter identity only; physical role external | `V1.3-DECIDED` |
| Projection mirror not fully pinned | Exact seven-node mirror + compositional rule | `INHERITED-ARCH` + `V1.3-DECIDED` |
| Partial projection permitted implicitly | MVP zero permitted-loss contracts | `V1.3-DECIDED` |
| Generic compatibility rules | Narrow declarative PhysicalContract | `INHERITED-ARCH` |
| ProposedInterpretation promotion | Permanently distinct artifacts | `INHERITED-ARCH` |
| Formalization as conceptual transfer | Typed proposal/response protocol | `V1.3-DECIDED` |
| Hypothesis ingestion API may be invented | Frozen Phys public APIs only | `INHERITED-PHYS` |
| Global replay | Authority-local replay + TOE meta-ledger | `INHERITED-ARCH` |
| Numerical symbolic parsing | phys-* lowering + Numerical contract | `V1.3-DECIDED` |
| Growth Gate potentially active during MVP | Post-MVP only | `V1.3-DECIDED` |

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
| PhysMath sort set | §9 | PhysMath | 17 closed v1 sort families |
| Expression nodes | §9.2 | PhysMath | 10 node kinds |
| Parameter identity | §11 | PhysMath | deterministic `ParameterID` |
| t/k separation | §12 | PhysMath + phys-* meaning | parameter identity |
| MathCondition | §14 | PhysMath | condition schema |
| PhysicalInterpretation | §16 | phys-* | interpretation schema |
| ProposedInterpretation | §16.6 | TOE-Agent | proposal schema |
| PhysicalContract | §16.4 | phys-* | declarative schema |
| Multiple interpretations | §17 | Consumer/framework | explicit framework selection |
| Correspondence | §18.1 | TOE-Agent | proposal schema |
| Research state | §18.3-18.4 | TOE-Agent | closed state machine |
| Formalization proposal | §19.1 | TOE-Agent | typed schema |
| Formalization response | §19.2 | integration tooling | typed schema |
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

**Given** the frozen Phys repository.  
**When** the architecture integration is applied.  
**Then** no new Phys import, file, API, kernel object, or dependency is created.  
**Failure:** any architecture-driven mutation of Phys.

## 33.3 Artifact identity

**Given** identical canonical payload bytes and identical namespace/type/version.  
**When** digesting twice.  
**Then** the digest is byte-identical.  
**Failure:** different digest.

## 33.4 Artifact framing

**Given** namespace values `a` and `ab` with distinct following fields.  
**When** canonical bytes are framed.  
**Then** no concatenation ambiguity exists.  
**Failure:** two distinct tuples produce the same framed header bytes.

## 33.5 Embedded ArtifactRef

**Given** a parent artifact containing an ArtifactRef.  
**When** canonicalizing the parent.  
**Then** the embedded ref uses the exact §6.3 encoding.  
**Failure:** JSON, hex, or bare digest representation appears in canonical payload.

## 33.6 Parameter rename

**Given** a differential operator targeting parameter identity P with display label `t`.  
**When** the label changes to `tau`.  
**Then** target identity remains P.  
**Failure:** parameter identity changes solely because display text changes.

## 33.7 t/k distinction

**Given** two parameters P and Q with P != Q.  
**When** differential operators target P and Q.  
**Then** the operators remain structurally distinct.  
**Failure:** symbol spelling alone causes equality.

## 33.8 Mathematical purity

**Given** a PhysMath expression containing a symbol named `c`.  
**When** the mathematical engine processes the symbol.  
**Then** it applies no physical constant semantics.  
**Failure:** mathematical behavior changes because the spelling is `c`.

## 33.9 Projection Pow

**Given** `Pow(x, y)` where y is a symbolic expression.  
**When** strict projection runs.  
**Then** result is `UNREPRESENTABLE` with `NON_RATIONAL_EXPONENT_IN_POW`.  
**Failure:** projection succeeds or rewrites the source silently.

## 33.10 Projection associativity

**Given** canonical `Add([a,b,c])`.  
**When** projected.  
**Then** the output is `Add(Add(a,b),c)` using canonical child order.  
**Failure:** output tree differs across runs.

## 33.11 Empty/single associative nodes

**Given** `Add([])` or `Add([a])`.  
**When** canonicalized.  
**Then** results are `0` and `a` respectively.  
**Failure:** multiple canonical forms remain.

## 33.12 Condition ordering

**Given** generated conditions in arbitrary traversal order.  
**When** finalized.  
**Then** conditions are sorted by digest and deduplicated.  
**Failure:** equivalent transformations yield different condition order or duplicate entries.

## 33.13 Formalization revision

**Given** a submitted proposal receiving `REQUEST_REVISION`.  
**When** a new revision is created.  
**Then** the parent is `(RESOLVED, REQUEST_REVISION)` and the child carries `ParentProposalRef`.  
**Failure:** parent remains mutable/submittable.

## 33.14 Provenance

**Given** hypothesis-grade Phys operands.  
**When** a session-recorded correspondence is created through the frozen Phys API.  
**Then** the resulting relation remains hypothesis-grade under the frozen contamination law.  
**Failure:** protocol code assumes corpus promotion.

## 33.15 Theory-local AST

**Given** a new `phys-gr` mathematical constructor.  
**When** it is added.  
**Then** it remains local unless an explicit PhysMath schema extension exists.  
**Failure:** PhysMath runtime accepts it as an unregistered extension sort.

## 33.16 Numerical handoff

**Given** an in-process `ComputationalContract`.  
**When** Numerical executes it.  
**Then** all execution arrays are already materialized.  
**Failure:** Numerical performs implicit ArtifactRef lookup.

## 33.17 Numerical semantics

**Given** a malformed physical contract.  
**When** Numerical receives it.  
**Then** validation fails or execution is rejected.  
**Failure:** Numerical invents missing physical assumptions.

## 33.18 Status separation

**Given** `ProjectionClass.UNREPRESENTABLE`.  
**When** a formalization response is constructed.  
**Then** its outcome type is a separately typed `FormalizationOutcome`.  
**Failure:** string comparison permits accidental cross-vocabulary casting.

## 33.19 Hypothesis constructor

**Given** `hypothesis.NewCandidateConcept`.  
**When** corpus-status validation runs.  
**Then** it is classified as hypothesis-grade and never as an ESTABLISHED manifest item.  
**Failure:** it appears in the ESTABLISHED constructor allowlist.

## 33.20 Generated assumption keys

**Given** an MVP operation result.  
**When** output assumption keys are checked.  
**Then** generated keys are a subset of input keys plus:

```text
denominator/<hash>
selected_branch/<hash>
```

**Failure:** any other operation-generated key appears.

---

# 34. Final blocker-only audit protocol

The final audit is not another architecture-design round.

## 34.1 Review question

Each reviewer MUST answer:

> Can a competent coding agent, given only v1.3 and the frozen Phys authority identified in §0.1, still choose between two architecturally legitimate implementations?

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

Once v1.3 is frozen, the implementation plan has one responsibility:

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

> **Implement v1.3 exactly. When the implementation encounters a condition not covered by v1.3, do not invent an architectural rule; stop and surface the exact missing contract.**

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

The following schemas are the concrete v1.3 contract. Each schema version is `"1"` unless explicitly inherited otherwise.

## A.1 `physmath.expression`

```text
ExpressionArtifactPayload {
    SortID       string        // exactly "Expression"
    PayloadBytes []byte        // canonical Expression encoding
}
```

The expression encoding is the recursive tagged union defined in §9.2–§9.11.

## A.2 `physmath.condition`

```text
MathCondition {
    Predicate ArtifactRef
}
```

`Predicate` MUST reference a mathematical proposition artifact.

## A.3 `physmath.trace`

```text
MathTrace {
    ProducerNamespace      string
    OperationID            string
    OperationSchemaVersion string
    InputRefs              []ArtifactRef   // SET
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
    MappingArtifactRefs   []ArtifactRef
    FrameworkRefs         []ArtifactRef
    Rationale             string
}
```

All `*Refs` fields are SETs unless an implementation-specific mapping contract explicitly declares sequence significance.

## A.9 ResearchHypothesis

```text
ResearchHypothesis {
    ClaimRefs                    []ArtifactRef
    ConstraintRefs               []ArtifactRef
    CorrespondenceRefs           []ArtifactRef
    ProposedInterpretationRefs   []ArtifactRef
    Status                       ResearchState
    PreviousStateRef             ArtifactRef?
    CounterexampleRefs           []ArtifactRef
    InconsistencyRefs            []ArtifactRef
}
```

All reference collections are SETs. `PreviousStateRef` is the only required history edge.

## A.10 FormalizationProposal

```text
FormalizationProposal {
    ProposalRevision      uint32
    ParentProposalRef     ArtifactRef?
    ProducerNamespace     string
    SourceArtifactRefs    []ArtifactRef   // SEQUENCE
    RequestedTarget       string
    ProjectionPolicy      string
    RequiredFrameworkRefs []ArtifactRef
}
```

MVP exact values:

```text
RequestedTarget  = "phys.hypothesis_object"
ProjectionPolicy = "STRICT_MIRROR_ONLY"
SourceArtifactRefs -> physmath.expression only
```

`RequiredFrameworkRefs` are a SET of explicit `ArtifactRef`s and MAY be empty.

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

Checks are a SET keyed by `Code`. Duplicate codes are invalid. The v1.3 check-code vocabulary is exactly:

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
```

The in-process form contains materialized `Data`; the `ArtifactRef`s are identity/provenance metadata and are not execution-time lookup instructions.

# Appendix B — Rule catalogue

The rule IDs used by the sort matrix are aliases for the normative sections above. They are identifiers, not separate behavior.

| Rule family | Normative source |
|---|---|
| `ID-001` | §§5–7, identity and canonical encoding |
| `EXPR-*` | §9.2–§9.11, Expression node contracts |
| `FN-*` | §10.1, Function |
| `FA-*` | §10.2, FunctionApplication |
| `FNL-*` | §10.3, Functional |
| `FNLA-*` | §10.4, FunctionalApplication |
| `OP-*` | §10.5, Operator |
| `OPA-*` | §10.6 and §12, OperatorApplication |
| `TEN-*` | §10.7, Tensor |
| `IDX-*` | §10.8, Index |
| `MAP-*` | §10.9, Map |
| `REL-*` | §9.10 and §14, Relation/condition semantics |
| `SET-*` | §10.10, SetCollection |
| `TR-*` | §10.11 and §14, Transformation |
| `LIM-*` | §10.12, Limit |
| `ASY-*` | §10.13, AsymptoticTruncation |
| `DO-*` | §10.14, DerivationObligation |
| `PROJ-*` | §20, projection contract |

The implementation agent MUST resolve a rule-family identifier to the cited normative section rather than inventing alternate behavior.

# Appendix C — PhysMath v1 sort contract matrix

The matrix is normative. Rule-family identifiers are defined in Appendix B and resolve to the cited normative sections.

| Sort | Construct | Canonicalize | Equality | Substitute | Serialize | Project |
|---|---|---|---|---|---|---|
| Expression | `RULE EXPR-001` | `RULE EXPR-002` | `RULE EXPR-003` | `RULE EXPR-004` | `RULE ID-001` | `RULE PROJ-001` |
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

The matrix fixes the v1.3 sort inventory and prevents an implementation plan from silently adding another core sort.

---

# Appendix D — Durable artifact family checklist

Every family below requires all thirteen fields from §29.

```text
ArtifactRef
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
[ ] v1.3 is the sole current architecture document
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
[ ] final four-agent blocker-only audit complete
```

---

# Appendix H — Closure status

This document is the v1.3 architecture specification, not a planning document. Its normative decisions are complete by design; the remaining procedural step is the final blocker-only audit in §34. A finding that demonstrates a genuine unresolved architectural choice reopens this document at that exact decision. A request for additional implementation convenience, stylistic preference, or future capability does not reopen it.

No v1.4 closure plan is permitted.

**END OF PM-ARCH-FOUNDATION-001 v1.3**
