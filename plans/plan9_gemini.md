# Phase-1 Implementation Plan: PhysMath & `phys-*` Architecture (PM-REP-001 v1.2)

This project-specific implementation plan establishes the concrete refactoring path from the existing `phys-gr` codebase to the normative architecture specified in PM-REP-001 v1.2. The primary deliverable is the formal implementation of `phys-artifact`, the `PhysMath` core library, the refactored `phys-gr` domain package, and the executable Equivariance Theorem protocol slice.

---

## 1. Inventory of Existing `phys-gr` Code and Responsibilities

An audit of the existing `phys-gr` implementation identifies five primary areas of responsibility that currently conflate theoretical mathematics, physical semantics, and canonical serialization:

| Module / File Component | Existing Responsibilities | Target Architectural Layer |
| --- | --- | --- |
| `gr/ast/` | Expression AST nodes, basic symbol types, equation representations, tensor indices. | Conflates math expressions with GR metric conventions. Split into `PhysMath` core IR and `phys-gr` domain types. |
| `gr/hash/` | Digest generation for expressions and AST nodes. | Uses ad-hoc hashing. Migrate to `phys-artifact` for canonical header/payload framing. |
| `gr/algebra/` | Canonicalization, simplification, substitution, and partial differentiation. | Conflates generic math rules with GR-specific metric contractions. Extract generic rules to `PhysMath`; retain metric algebra in `phys-gr`. |
| `gr/physics/` | Einstein tensor definitions, stress-energy momentum, spacetime dimension, geodesics. | Pure domain semantics. Retain in `phys-gr` under the `phys-gr/definition/1` and `phys-gr/equation/1` families. |
| `gr/protocol/` | Test harnesses, manifest parsing, and derivation step verification. | Packaging/protocol harness. Standardize under `protocol/derivation_test.go` and `manifest.json`. |

---

## 2. Code Mapping and Architectural Allocation Matrix

Existing code and new components are partitioned across the four primary Phase-1 boundaries according to the core/specific separation rule:

```
+-----------------------------------------------------------------------------------+
|                                  PHYS-ARTIFACT                                    |
|  - Canonical Primitive Encoders (UInt, String, Bytes, Optional, Sequence, Set)   |
|  - ArtifactRef Digest Formula: SHA-256(Len32(NS)||NS||...||Payload)               |
|  - BigInt Canonical Serialization                                                 |
+-----------------------------------------------------------------------------------+
                                          ^
                                          |
+-----------------------------------------------------------------------------------+
|                                    PHYSMATH                                       |
|  - 16 Closed Math Sorts & 10 Core Node Types (Symbol, Rational, Add, Mul, etc.)  |
|  - ParameterID Derivation: SHA-256("physmath" || ScopePath || ParameterOrdinal)   |
|  - 16 Durable Families (physmath/expression/1, physmath/function/1, etc.)         |
|  - 7 Closed Transformation Rules (EXPR-CANON-001, EXPR-APPLY-001, etc.)           |
+-----------------------------------------------------------------------------------+
                                          ^
                                          |
+-----------------------------------------------------------------------------------+
|                                    PHYS-GR                                        |
|  - Framework Identity: phys-gr/framework/1                                        |
|  - Domain Families: assumption/1, definition/1, equation/1, theorem/1, etc.       |
|  - Physical Interpretations & Spacetime Metric Assumptions                        |
+-----------------------------------------------------------------------------------+
                                          ^
                                          |
+-----------------------------------------------------------------------------------+
|                                PROTOCOL SLICE                                     |
|  - AGENTS.md, manifest.json, protocol/derivation_test.go                          |
|  - Equivariance Theorem Reference Derivation Replay                              |
+-----------------------------------------------------------------------------------+

```

---

## 3. Core Common Denominator (`phys-artifact` & `PhysMath`)

### 3.1 `phys-artifact` Infrastructure

`phys-artifact` provides binary serialization framing and content addressing.

* **ArtifactRef Formulation**:
```go
type ArtifactRef struct {
    Namespace     string
    SchemaVersion string
    ArtifactType  string
    Digest        [32]byte
}

```


* **Digest Rule**:

$$\text{Header} = \text{Len32}(\text{NS}) \parallel \text{NS} \parallel \text{Len32}(\text{Ver}) \parallel \text{Ver} \parallel \text{Len32}(\text{Type}) \parallel \text{Type}$$


$$\text{Digest} = \text{SHA-256}(\text{Header} \parallel \text{Payload})$$


* **Primitive Encoders**: Strict big-endian integers (`UInt8`, `UInt16`, `UInt32`, `UInt64`), `String` (length-prefixed UTF-8), `Bytes`, `Optional` ($0x00$ or $0x01 \parallel \text{Payload}$), `Sequence` (length-prefixed ordered items), and `Set` (length-prefixed, deduplicated, sorted bytewise lexicographically).
* **BigInt Canonical Form**: `Sign` ($0x00$ for non-negative, $0x01$ for negative) $\parallel$ `UInt32BE(MagLen)` $\parallel$ Minimal unsigned big-endian bytes (no redundant leading zero bytes; zero is $0x00 \parallel \text{UInt32BE}(0)$).

### 3.2 `PhysMath` Core IR & AST

`PhysMath` owns theory-neutral mathematical constructs across 16 closed durable families:

1. **AST Node Set**: `Symbol` ($0x01$), `Rational` ($0x02$), `Add` ($0x03$), `Mul` ($0x04$), `Neg` ($0x05$), `Pow` ($0x06$), `Sqrt` ($0x07$), `Call` ($0x08$), `Relation` ($0x09$), and `BranchSet` ($0x0A$).
2. **ParameterID Formula**:

$$\text{ParameterID} = \text{SHA-256}(\text{String}("physmath") \parallel \text{Bytes}(\text{UInt32BE}(0)) \parallel \text{UInt32BE}(\text{ParameterOrdinal}))$$


3. **BoundParameterRef Encoding**:

$$\text{Encode}(\text{BoundParameterRef}) = \text{EncodeArtifactRef}(\text{DeclaringArtifactRef}) \parallel \text{Digest32}(\text{ParameterID})$$


4. **Function Validation**: Every durable `Function` must satisfy `Function.ParameterIDs[i] == DeriveParameterID(i)` for all parameter slots, and all body symbols must be either declared parameters or valid constants.

### 3.3 Core Transformation Rule Executors

`PhysMath` provides execution logic for the seven closed core rules:

* `EXPR-CANON-001`: Post-order traversal, child canonicalization, `Add`/`Mul` flattening, commutative sorting by canonical bytes, and identity normalization.
* `EXPR-SIMPLIFY-001`: Double negation elimination (`Neg(Neg(x)) -> x`), identity pruning (`Add(x, 0) -> x`, `Mul(x, 1) -> x`), zero multiplication (`Mul(x, 0) -> 0`), and complete rational child reduction. `Pow(x, 0) -> 1` requires an explicit `NEQ(x, 0)` condition in the context.
* `EXPR-SUBSTITUTE-001`: Simultaneous ParameterID substitution given input expression, target functions, and sorted `BoundParameterRef` maps.
* `RAT-NORMALIZE-001`: Normalizes `Rational` nodes to coprime numerators and positive denominators.
* `DIFFERENTIATE-001`: Partial differentiation over scalar expression trees with respect to a bound parameter slot.
* `SELECT-BRANCH-001`: Extracts a zero-based branch from a `BranchSet` root node.
* `EXPR-APPLY-001`: Expands a top-level `Call` node by substituting arguments into the target `Function` body.

---

## 4. Framework-Specific Responsibilities (`phys-gr`)

`phys-gr` is refactored into a conforming physical framework package using the `phys-gr` namespace.

### 4.1 Framework Artifacts

* **Framework Identifier**: `phys-gr/framework/1`
```go
FrameworkArtifact{
    FrameworkID:      "general-relativity",
    PackageNamespace: "phys-gr",
    FrameworkVersion: "1.0.0",
    BaseFrameworkRefs: [],
}

```


* **Physical Assumptions (`phys-gr/assumption/1`)**: Pseudo-Riemannian manifold structure, metric signature $(-,+,+,+)$, torsion-free Levi-Civita connection, and stress-energy conservation ($\nabla_\mu T^{\mu\nu} = 0$).
* **Physical Definitions (`phys-gr/definition/1`)**: Metric tensor $g_{\mu\nu}$, Ricci tensor $R_{\mu\nu}$, and scalar curvature $R$.
* **Physical Equations (`phys-gr/equation/1`)**: Einstein Field Equations ($G_{\mu\nu} + \Lambda g_{\mu\nu} = \frac{8\pi G}{c^4} T_{\mu\nu}$) mapping to underlying `physmath/equation/1` instances.
* **Physical Interpretations (`phys-gr/interpretation/1`)**: Spacetime geometry interpretation of metric tensors and energy-momentum distribution of matter fields.

---

## 5. Refactoring & Migration Phasing

```
 Phase A: Infrastructure
 +-------------------------------+
 | Build phys-artifact primitives|
 | Impl ArtifactRef & Digest     |
 | Standardize BigInt Encoding   |
 +-------------------------------+
                 |
                 v
 Phase B: PhysMath AST & Families
 +-------------------------------+
 | Impl 10 AST Node Types        |
 | Impl ParameterID Engine       |
 | Register 16 Durable Families  |
 +-------------------------------+
                 |
                 v
 Phase C: Transformation Rules
 +-------------------------------+
 | Impl 7 Core Rule Executors    |
 | Build Deterministic Replay    |
 | Implement Derivation Validation|
 +-------------------------------+
                 |
                 v
 Phase D: phys-gr Refactoring
 +-------------------------------+
 | Isolate GR Physics Semantics  |
 | Register 6 phys-gr Families   |
 | Decouple Math from Physics    |
 +-------------------------------+
                 |
                 v
 Phase E: Equivariance Slice
 +-------------------------------+
 | Construct Equivariance Fixture|
 | Build 6-Step Derivation       |
 | Execute Protocol Harness      |
 +-------------------------------+

```

### Phase A: `phys-artifact` Foundations

1. Create `phys-artifact` package.
2. Implement primitive coders (`UInt`, `String`, `Bytes`, `Optional`, `Sequence`, `Set`).
3. Implement `ArtifactRef` serialization and canonical digest formula.
4. Implement `BigInt` binary encoder and validator.

### Phase B: `PhysMath` Core IR

1. Implement 10 AST node expressions and canonical tree encoders.
2. Implement `ParameterID` derivation and validation logic.
3. Define structs and serializers for the 16 durable PhysMath families.
4. Port existing AST nodes from `phys-gr` into `PhysMath`, stripping physical metadata.

### Phase C: Rule Engine & Transformations

1. Implement `OperationInvocation` and `TransformResult` execution contracts.
2. Implement rule engines for `EXPR-CANON-001`, `EXPR-SIMPLIFY-001`, `EXPR-SUBSTITUTE-001`, `RAT-NORMALIZE-001`, `DIFFERENTIATE-001`, `SELECT-BRANCH-001`, and `EXPR-APPLY-001`.
3. Implement `DerivationStep` validator and derivation replay harness.

### Phase D: `phys-gr` Extraction & Domain Alignment

1. Refactor `phys-gr` to depend on `PhysMath` and `phys-artifact`.
2. Package GR physical assumptions, definitions, field equations, and interpretations into `phys-gr/*` families.
3. Ensure no local GR type extends `PhysMath` AST dynamically.

### Phase E: Equivariance Protocol Slice

1. Construct the reference equivariance functions and expressions.
2. Build the 6-step derivation sequence.
3. Wire the protocol runner in `protocol/derivation_test.go` and validate `manifest.json`.

---

## 6. Test Suite and Verification Strategy

The test suite directly maps to the 32 acceptance tests in §25 of PM-REP-001 v1.2:

```
                  +-----------------------------------+
                  |   Phase-1 Acceptance Test Suite   |
                  +-----------------------------------+
                                    |
        +---------------------------+---------------------------+
        |                                                       |
        v                                                       v
+-------------------------------+               +-------------------------------+
|     Identity & Framing        |               |      Rule & Transformation    |
| - 25.1: ArtifactRef Identity  |               | - 25.9: Recursion & Canonical |
| - 25.2: Primitive Framing     |               | - 25.11: Arity & Failure      |
| - 25.3: BigInt Boundaries     |               | - 25.12: EXPR-APPLY Root Only |
| - 25.4: Alpha-Renaming Ident  |               | - 25.13: Substitution Scope   |
| - 25.6: Function Closure      |               | - 25.15: Branch Selection     |
| - 25.30: Parameter Ordinals   |               | - 25.26: Explicit Conditions  |
+-------------------------------+               +-------------------------------+
        |                                                       |
        +---------------------------+---------------------------+
                                    |
        +---------------------------+---------------------------+
        |                                                       |
        v                                                       v
+-------------------------------+               +-------------------------------+
|   Domain & Framework Isolation|               |    Equivariance Protocol      |
| - 25.7: Symbol Neutrality     |               | - 25.19: Protocol Structure   |
| - 25.22: Multi-Framework      |               | - 25.20: 6-Step Derivation    |
| - 25.23: Framework Dependency |               | - 25.21: Relation Conclusion  |
| - 25.24: No Dynamic Extension |               | - 25.32: Staged Call Linkage  |
+-------------------------------+               +-------------------------------+

```

* **Identity & Encoding Tests**: Verify `ArtifactRef` digest stability (§25.1), unambiguous primitive framing (§25.2), `BigInt` minimal length checks (§25.3), parameter alpha-renaming neutrality (§25.4), and parameter ordinal consistency (§25.30).
* **Rule Execution Tests**: Validate strict arity enforcement (§25.11), root-only constraint for `EXPR-APPLY-001` (§25.12), missing substitution context rejection (§25.13), branch selection bounds (§25.15), explicit condition requirements for `Pow(x,0)` (§25.26), and strict rational arithmetic triggering (§25.27).
* **Framework & Isolation Tests**: Verify symbol physical neutrality (§25.7), multi-framework interpretation coexistence (§25.22), framework dependency closure rules (§25.23), rejection of dynamic AST extensions (§25.24), and namespace grammar validation (§25.31).

---

## 7. Executable Equivariance Protocol Fixture Path

The reference equivariance theorem slice exercises the full mathematical and physical representation stack.

### 7.1 Protocol Package Layout

```text
equivariance-fixture/
├── AGENTS.md
├── manifest.json
└── protocol/
    └── derivation_test.go

```

### 7.2 Core Mathematical Setup

* **Target Actions & Maps**:
* $f(v) = 2 \cdot v$ (Parameter: $v \to \text{PID}_0$)
* $\rho_X(g, v) = g \cdot v$ (Parameters: $g \to \text{PID}_0$, $v \to \text{PID}_1$)
* $\rho_Y(g, v) = g \cdot v$ (Parameters: $g \to \text{PID}_0$, $v \to \text{PID}_1$)


* **Schematic Symbols**:
* $g = \text{Symbol}(\text{ParameterID}=\text{PID}_0, \text{DisplayName}="g")$
* $x = \text{Symbol}(\text{ParameterID}=\text{PID}_1, \text{DisplayName}="x")$



### 7.3 Staged Calls & 6-Step Derivation Sequence

The derivation requires pre-constructed staged outer call expressions:

* $\text{LHS}_0 = \text{Call}(f, [\text{Call}(\rho_X, [g, x])])$
* $\text{LHS}_1 = \text{Call}(f, [g \cdot x])$
* $\text{RHS}_0 = \text{Call}(\rho_Y, [g, \text{Call}(f, [x])])$
* $\text{RHS}_1 = \text{Call}(\rho_Y, [g, 2 \cdot x])$

```text
Step 0: EXPR-APPLY-001  on Call(rho_X, [g, x])         -> g * x
Step 1: EXPR-APPLY-001  on LHS_1 = Call(f, [g * x])    -> 2 * (g * x)
        (Verify LHS_1 argument bytes == Step 0 output bytes)
Step 2: EXPR-CANON-001  on 2 * (g * x)                 -> 2 * g * x

Step 3: EXPR-APPLY-001  on Call(f, [x])                -> 2 * x
Step 4: EXPR-APPLY-001  on RHS_1 = Call(rho_Y, [g, 2*x]) -> g * (2 * x)
        (Verify RHS_1 second-argument bytes == Step 3 output bytes)
Step 5: EXPR-CANON-001  on g * (2 * x)                 -> 2 * g * x

```

### 7.4 Conclusion Relation Verification

The protocol verifies that the canonical bytes of Step 2 output ($2 \cdot g \cdot x$) and Step 5 output ($2 \cdot g \cdot x$) are identical. It then wraps the output in a durable `physmath/relation/1` ($EQ$), links it to a `physmath/equation/1`, and binds it as the `ConclusionRef` of the `physmath/theorem/1` artifact.

---

## 8. Architectural Gap & Blocker Audit (§24.4 Stop-Rule Assessment)

An audit of the PM-REP-001 v1.2 specification against the implementation plan confirms that all architectural decisions are closed:

| Gap Category | Status | Assessment & Ruling |
| --- | --- | --- |
| **MATH GAP** | Closed | All 16 sorts, 10 AST nodes, and 7 core transformation rules are fully specified with deterministic parameters and execution semantics. |
| **PHYS GAP** | Closed | `phys-*` physical framework contracts, six durable families, and base/specialized framework dependency closure rules are fully defined. |
| **FROZEN-PHYS GAP** | Closed | Frozen `phys` (`[github.com/PithomLabs/phys](https://github.com/PithomLabs/phys)`) is untouched; no modifications or dependency changes are made to the frozen kernel. |
| **INTEGRATION GAP** | Closed | Secondary systems (TOE-Agent, Numerical, Trust/Corpus promotion) are explicitly deferred and do not block Phase 1. |
| **PROTOCOL GAP** | Closed | Protocol packaging strictly reuses `AGENTS.md`, `manifest.json`, and `protocol/derivation_test.go` from the General Relativity precedent. |

No architectural blockers exist. The implementation plan is ready for immediate execution.