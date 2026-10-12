## 1. Critical Architectural Blockers & Execution Traps

### 1.1 `EXPR-APPLY-001` Root Constraint vs. Enclosing AST Rebuilding Gap

* **The Specification Constraint (§17.7):** `EXPR-APPLY-001` mandates that `Input[0]` MUST have a root node of tag `0x08 Call`. It explicitly forbids applying function expansion to nested `Call` nodes inside `Add`, `Mul`, `Pow`, `Relation`, or outer `Call` nodes without isolating the subtree.
* **The Derivation Execution Trap (§16.9):** In the Equivariance Theorem derivation:
* Step 0 applies `rho_X(g, x) -> g * x`.
* Step 1 applies `f(rho_X(g, x)) -> 2 * (g * x)`.
* For Step 1 to execute `EXPR-APPLY-001`, `Input[0]` must be an expression artifact representing `Call(f, [g * x])`.


* **Architectural Gap:** Phase 1 contains exactly seven executable rules (§11.5, §24.5). There is **no** rule for "AST sub-node replacement" or "rebuilding enclosing Call nodes".
* **Resolution for Coding Agent:** Intermediate Call artifacts (such as `Call(f, [g * x])` for Step 1 and `Call(rho_Y, [g, 2 * x])` for Step 4) cannot be derived via rule transformations from Step 0/Step 3 outputs. They MUST be pre-constructed during setup and supplied directly as elements of `Derivation.PremiseRefs` (or protocol fixtures) per §12.6 and §16.9.

### 1.2 `Symbol.DisplayName` Serialization vs. Alpha-Equivalence

* **The Specification Schema (§7.4, §4.8):** `Symbol` is defined as `{ DisplayName string, ParameterID Optional<ParameterID>, ConstantID Optional<ConstantID> }`. Following the record encoding rule (§4.8), canonical payload bytes for `Symbol` begin with `String(DisplayName)`.
* **The Alpha-Renaming Rule (§8.4, §25.4):** Changing `t -> tau` does not change `ParameterID`.
* **Architectural Ambiguity:** While `ParameterID` remains invariant, changing `DisplayName` **does** alter the canonical payload bytes of the `Symbol` expression node, producing a different `physmath/expression/1` digest (`ArtifactRef`).
* **Resolution for Coding Agent:** Do not conflate parameter binding equivalence with `ArtifactRef` digest equality. Two `Symbol` nodes with identical `ParameterID`s but different `DisplayName`s are semantically equivalent parameters within a `Function` scope, but they yield distinct `ArtifactRef` digests.

### 1.3 Commutative Child Sorting (`Add`/`Mul`) vs. Human Display Prose

* **The Sorting Rule (§17.3, §18.1):** Children in `Add` and `Mul` nodes MUST be sorted strictly by the lexicographical byte comparison of their canonical expression encodings.
* **The Byte Order Trap:** The expression tag for `Symbol` is `0x01`, while `Rational` is `0x02`. Because `0x01 < 0x02`, `Symbol` nodes (e.g., `g`, `x`) precede `Rational` nodes (e.g., `2/1`) in canonical payload bytes.
* **Resolution for Coding Agent:** In code, the canonical AST for `2 * g * x` must store children in byte-sorted order: `[ Symbol(g), Symbol(x), Rational(2/1) ]`. Prose representations in §16.7–16.9 (writing `2 * g * x`) are display formatting only and MUST NOT override raw byte sorting (§1.3, §18.1).

---

## 2. Serialization & Binary Encoding Spec Ambiguities

### 2.1 Double-Framing of `Parameters` in Rule Records

* **Schema Definition (§10.8, §12.2):** Both `Transformation` and `DerivationStep` define `Parameters` as type `Bytes`.
* **Primitive Encoding Rule (§4.5):** `Bytes(B)` is encoded as `UInt32BE(len(B)) || B`.
* **Encoding Matrix for Rules:**

| Rule | Parameter Payload `B` | Encoded `Parameters` Field Bytes |
| --- | --- | --- |
| `EXPR-CANON-001`, `EXPR-SIMPLIFY-001`, `RAT-NORMALIZE-001`, `EXPR-APPLY-001` | `[]byte{}` (0 bytes) | `0x00000000` (4 bytes) |
| `SELECT-BRANCH-001` | `UInt32BE(BranchOrdinal)` (4 bytes) | `0x00000004` || `UInt32BE(BranchOrdinal)` (8 bytes) |
| `DIFFERENTIATE-001` | `Encode(BoundParameterRef)` || `UInt32BE(Order)` (97 bytes) | `0x00000061` || `97 bytes payload` (101 bytes) |
| `EXPR-SUBSTITUTE-001` | `UInt32BE(N)` || `Entries...` | `UInt32BE(len(B))` || `B` |

### 2.2 Precise Byte Stream Construction for `ParameterID`

* **Formula (§8.3):** `ParameterID = SHA-256(String("physmath") || Bytes(ScopePathBytes) || UInt32BE(ParameterOrdinal))`.
* **Phase-1 Expansion (Depth 0):**
1. `String("physmath")` $\rightarrow$ `0x00000008` || `"physmath"` (12 bytes)
2. `ScopePathBytes` = `UInt32BE(0)` (4 bytes) $\rightarrow$ `Bytes(ScopePathBytes)` = `0x00000004` || `0x00000000` (8 bytes)
3. `UInt32BE(ParameterOrdinal)` $\rightarrow$ 4 bytes big-endian integer


* **Total Hashing Buffer:** Exactly 24 bytes passed to SHA-256.

### 2.3 `physmath/relation/1` Payload vs. `Expression` Tag `0x09`

* **Expression Node Tag `0x09` (§7.11):** Encoded as `0x09` || `Enum(Operator)` || `CanonicalExpression(Left)` || `CanonicalExpression(Right)`.
* **Durable Artifact `physmath/relation/1` (§19.2, §25.16):** Encoded as `Enum(Operator)` || `CanonicalExpression(Left)` || `CanonicalExpression(Right)` (without the `0x09` tag).
* **Operand Encoding:** `Left` and `Right` are embedded canonical expression trees (`UInt8(tag) || payload`), **never** `ArtifactRef` indirections.

---

## 3. Rule Specification & Arity Edge Cases

### 3.1 `EXPR-SUBSTITUTE-001` Strict Sorting & Validation Contracts

* **Input Arity (§17.2, §17.8):** Requires `1 + D` inputs ($D \ge 1$).
* `Input[0]`: Target `physmath/expression/1`.
* `Inputs[1..D]`: Declaring `physmath/function/1` artifacts.


* **Ordering Requirements:**
1. `Inputs[1..D]` MUST be sorted lexicographically by their canonical `ArtifactRef` byte representation.
2. The parameter entries in `Parameters` MUST be sorted lexicographically by their canonical `Entry` bytes (`Encode(BoundParameterRef) || EncodeArtifactRef(ReplacementExpression)`).


* **Validation Failure Conditions:**
* Arity $< 2$.
* `Inputs[1..D]` unsorted.
* Parameter entries unsorted.
* A `BoundParameterRef` references a function not present in `Inputs[1..D]`.
* A target `ParameterID` is not declared by the specified `Function`.



### 3.2 `EXPR-SIMPLIFY-001` Discharged Condition Validation

* **Rule Behavior (§17.4):** Simplification of `Pow(x, 0) -> 1/1` MUST fail unless an explicit `MathCondition` representing $x \neq 0$ is present in the step's `ConditionRefs` or enclosing context.
* **Failure Handling:** The simplifier MUST return an error and `nil` output reference if $x \neq 0$ is missing. It MUST NOT automatically generate or assume the condition.

---

## 4. Graph Linkage & Target Family Contradictions

### 4.1 Target Family Matrix Summary

The coding agent MUST enforce the following closed target validation map during artifact deserialization (§5.7):

```
+------------------------------------+---------------------------------------------------+
| Struct Field                       | Valid Target Families                             |
+------------------------------------+---------------------------------------------------+
| Function.BodyRef                   | physmath/expression/1                             |
| FunctionApplication.FunctionRef    | physmath/function/1                               |
| Map.DefinitionRef                  | physmath/definition/1                             |
| Equation.RelationRef               | physmath/relation/1                               |
| EquationSet.EquationRefs[*]        | physmath/equation/1                               |
| Definition.DefinedRef              | physmath/function/1 OR physmath/expression/1      |
| Definition.DefiningRef             | physmath/expression/1                             |
| MathCondition.PredicateRef         | physmath/relation/1                               |
| PhysicalAssumption.ConditionRefs[*] | physmath/condition/1                              |
| PhysicalEquation.EquationRef       | physmath/equation/1                               |
| PhysicalTheorem.MathTheoremRef     | physmath/theorem/1                                |
+------------------------------------+---------------------------------------------------+

```

### 4.2 Framework Dependency Closure Rules (§14.9, §21.3)

* **Validation Restrictions:**
1. `FrameworkArtifact.BaseFrameworkRefs` must form an acyclic directed graph. Self-references or cycles MUST trigger a validation error.
2. A physical artifact in namespace `phys-A` may reference an assumption in `phys-B` **only** if `phys-B` is in the transitive `BaseFrameworkRefs` closure of `phys-A`.



---

## 5. Coding Agent Implementation Blueprint

To ensure complete implementation without architectural blockers, structure the Go codebase according to the following module layout and execution flow:

```
physmath/
├── artifact/         # phys-artifact: Digest, ArtifactRef, Primitive Codecs (§4)
├── ir/               # PhysMath IR: AST Nodes, Sort Vocabulary, Canonical Codecs (§6-10)
├── rules/            # 7 Phase-1 Rule Implementations & Evaluator (§11, §17)
├── derivation/       # DerivationStep, Derivation, Theorem Verification (§12-13)
├── phys/             # phys-* Base Framework & Physical Schemas (§14)
└── test/protocol/    # Executable GR & Equivariance Protocol Tests (§16, §25)

```

### Step-by-Step Implementation Flow

1. **`phys-artifact` Primitive Layer:** Implement `String`, `Bytes`, `Optional`, `Sequence`, `Set`, `BigInt`, and `ArtifactRef` header/digest algorithms (§4.2–4.10).
2. **PhysMath AST & Serialization:** Implement the 10 core expression nodes (§7.1) and serialization handlers. Enforce child sorting for `Add` and `Mul` nodes (§18.1).
3. **Parameter Binding Engine:** Implement `ParameterID` hashing (§8.3) and `BoundParameterRef` encoding (§4.10).
4. **Transform Rule Engine:** Implement the 7 closed Phase-1 rules (§17) returning `TransformResult{OutputRef, ConditionRefs}` (§17.1). Reject all unregistered `(ProducerNamespace, RuleID)` pairs (§24.5).
5. **Derivation Replay:** Implement `DerivationStep` verification (§12.2–12.6) checking step sequence, availability of inputs, and condition tracking.
6. **`phys-*` Physical Layer:** Implement framework identity, assumption, definition, equation, theorem, and interpretation schemas (§14), enforcing transitive `BaseFrameworkRefs` lookup (§14.9).
7. **Equivariance Protocol Fixture:** Construct the protocol runner in `protocol/derivation_test.go` exercising the 6 derivation steps (§16.9) and verifying equality of output `ArtifactRef`s.