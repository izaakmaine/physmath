# PM-REP-001 v1.1 — Consolidated Adversarial Review of spec_v2

**Purpose:** Consolidate the four independent v2 reviews into one authoritative closure matrix and determine the exact amendments required to reach the project target:

> **Zero architectural blockers and zero architectural ambiguity for the coding agent.**

**Scope:** This review concerns the Phase-1 PhysMath + `phys-*` representation architecture and its executable reference protocol. It does not reopen the Phase-1/Phase-2 boundary.

**Source reviews read in full:**
- `spec_v2_deepseek.md` — 246 lines
- `spec_v2_gemini.md` — 136 lines
- `spec_v2_qwen.md` — 124 lines
- `spec_v2_z.md` — 92 lines

**Combined review material:** 598 lines.

---

## 1. Executive verdict

`spec_v2` has the correct macro-architecture, and all four reviews independently agree on that point. The Phase-1 / Phase-2 split, PhysMath purity, `phys-*` physical-semantics ownership, many-frameworks principle, rejection of frozen-Phys integration, and deferral of TOE-Agent / Numerical / trust concerns should remain locked.

The reviews do **not** justify another architecture redesign. They identify a finite set of mechanical contract gaps created by the consolidation itself.

However, the four reviews contain two classes of findings:

1. **True blockers / ambiguities** that must be amended in `spec_v2`/`PM-REP-001`.
2. **Duplicate or false-positive findings** where the cited rule is already defined in the base `PM-REP-001 v1.0` document supplied alongside `spec_v2`.

The correct response is therefore **one mechanical amendment**, not another architecture round.

---

## 2. Locked architecture — do not reopen

The following decisions are consistently validated by the reviews and remain frozen:

### 2.1 Responsibilities

```text
PhysMath
    pure mathematical representation

phys-*
    physical-domain/framework semantics

Phys
    frozen trusted physical formalization authority

TOE-Agent
    later research-state / hypothesis / correspondence layer

Numerical
    later numerical execution layer
```

PhysMath does not infer physical meaning. `phys-*` does not redefine PhysMath. The frozen Phys authority is not modified to accommodate this architecture.

### 2.2 Phase separation

Do **not** add to Phase 1:

- frozen-Phys `Kind` / `Dimension` / assumption projection;
- TOE-Agent artifact schemas;
- trust / provenance / corpus machinery;
- a generic `PhysicalContract` abstraction.

Those concerns belong to later specifications. Pulling them into the representation core would violate the separation-of-concerns rule and re-couple layers that are intentionally independent.

### 2.3 Base/specialized split

The `phys-qm` / `phys-bohm` relationship is structurally correct:

```text
phys-qm
   ↑
phys-bohm
```

The base package contains only the minimum assumptions needed by the base framework. Interpretation-specific assumptions belong in the specialized package.

This principle recurses for other framework families (`phys-gr`, `phys-cdt`, `phys-cst`, etc.).

### 2.4 Formalization boundary

The executable protocol used by the existing General Relativity example is the canonical protocol shape for the project:

```text
AGENTS.md
manifest.json
protocol/derivation_test.go
```

For any future theorem, hypothesis, derivation, or physics-paper formalization, prose alone is not formal representation. Formal status is obtained only when the content is represented and exercised through the same protocol model.

This protocol requirement is a project-level invariant and must be incorporated into the next amendment's closure criteria.

---

## 3. Finding disposition summary

| ID | Consolidated finding | Disposition | Required action |
|---|---|---|---|
| B01 | ProducerNamespace missing from durable rule records | **BLOCKER** | Add field to `Transformation` and `DerivationStep` |
| B02 | Rule input arity not closed | **BLOCKER** | Close arity + per-position families + invalid-arity behavior for all 7 rules |
| B03 | Simplify/canon recursion policy not closed | **BLOCKER** | Define traversal, local rewrite loop, idempotence/fixpoint behavior |
| B04 | EXPR-APPLY applicability boundary not closed | **BLOCKER** | Root-only predicate, function-ref equality, exact failure behavior |
| B05 | Cross-framework assumption references conflict with base/specialized architecture | **BLOCKER** | Add explicit framework dependency closure and permit refs into declared dependencies |
| B06 | Operation result type/name inconsistent | **BLOCKER** | Standardize on `TransformResult`; remove `OperationResult`; define exact shape |
| B07 | FrameworkVersion value/grammar not explicit | **AMBIGUITY** | Make it opaque package-owned string with `String()` encoding; fixture pins exact value |
| B08 | General ArtifactRef targets remain vague in some fields | **BLOCKER** | Publish closed target-family table |
| B09 | PhysicalAssumption.ConditionRefs target omitted | **BLOCKER** | Pin to `physmath/condition/1`, SET |
| B10 | Relation value-vs-reference encoding can be misread | **AMBIGUITY** | Add explicit value-type serialization warning |
| B11 | Expression inline-vs-durable boundary unclear in edge cases | **AMBIGUITY** | State exact boundary and nested-call strategy |
| B12 | Function body binding/capture semantics under-specified | **BLOCKER** | Validate body symbols and forbid freshening |
| B13 | EXPR-SUBSTITUTE input arrangement / recursive matching not closed | **AMBIGUITY** | Pin input layout, declaring-function validation, and recursive traversal |
| B14 | DIFFERENTIATE declaring-context asymmetry | **AMBIGUITY** | Require declaring Function input and exact ref match |
| B15 | SELECT-BRANCH input shape / condition behavior | **AMBIGUITY** | One expression input rooted at BranchSet; no generated condition |
| B16 | RAT-NORMALIZE input family/root | **AMBIGUITY** | One `physmath/expression/1` rooted at Rational |
| B17 | Deferred-sort durability wording conflicts with closed family registry | **AMBIGUITY** | Deferred sorts are in-memory-only in Phase 1 |
| B18 | Phase-1 executable RuleID registry must be explicitly exhaustive | **AMBIGUITY** | Lock exactly 7 runtime rules; reject others |
| B19 | Fixture graph does not specify g/x/X/Y representation | **BLOCKER for reference fixture** | Pin fixture-local representation without adding a new sort |
| B20 | Fixture may require algebra not in the closed rule set | **BLOCKER for reference fixture** | Pin concrete functions to close by canonicalization/simplification only |
| B21 | Relation embedded sides vs standalone expressions need identity rule | **AMBIGUITY** | Require byte identity between canonical subtree and standalone expression payload |
| B22 | Grammars dropped from consolidation | **AMBIGUITY** | Restore settled namespace/type/version/identifier grammars |
| B23 | PhysicalDefinition vs PhysicalInterpretation boundary under-defined | **AMBIGUITY** | Add one explicit division-of-labor sentence |
| B24 | InterpretationClass removal not explicitly resolved | **AMBIGUITY** | State all Phase-1 physical artifacts are framework-scoped; no neutral physical claims |
| B25 | DomainSort/CodomainSort values not closed | **AMBIGUITY** | Restrict to registered Phase-1 sort IDs |
| B26 | Golden vectors do not fully pin the reference derivation sequence | **TEST/CONFORMANCE GAP** | Pin exact reference sequence and fixture inputs |
| B27 | DerivationStep vs Transformation lifecycle overlap | **AMBIGUITY** | Inline proof step vs standalone durable operation boundary |
| B28 | Generic Bytes/Optional/Set/Sequence serialization | **NOT A NEW BLOCKER** | Already defined in PM-REP-001 v1.0; spec_v2 should explicitly inherit, not re-invent |
| B29 | phys-* field layouts | **NOT A NEW BLOCKER** | Struct schemas already exist in PM-REP-001 v1.0; spec_v2 must only amend/extend explicitly |
| B30 | Derivation/Transformation byte layout | **NOT A NEW BLOCKER AFTER B01** | Existing field-order + primitive encoding rules apply; add amended ProducerNamespace field to the normative order |
| B31 | Additional conclusion-production risk in equivariance proof | **VERIFY / CLOSE** | Reference protocol test must prove the theorem conclusion is actually produced/validated; add explicit acceptance criterion |

---

## 4. Exact normative resolutions

The following are the decisions the next amendment should state. They are deliberately written as implementation decisions rather than suggestions.

### B01 — ProducerNamespace becomes a real field

Rule identity is:

```text
(ProducerNamespace, RuleID)
```

Therefore the durable schemas MUST carry both values.

```text
Transformation {
    ProducerNamespace string
    RuleID            TransformationRuleID
    Inputs            []ArtifactRef // SEQUENCE
    Output            ArtifactRef
    Parameters        Bytes
}
```

```text
DerivationStep {
    Ordinal        UInt32
    ProducerNamespace string
    RuleID         TransformationRuleID
    InputRefs      []ArtifactRef // SEQUENCE
    OutputRef      ArtifactRef
    ConditionRefs  []ArtifactRef // SET
    Parameters     Bytes
}
```

Canonical field order is exactly the declaration order above.

Phase-1 core rules use:

```text
ProducerNamespace = "physmath"
```

Future framework-owned rules may use the producer's namespace, but no new framework-owned executable rule enters the Phase-1 runtime registry unless a future version explicitly registers it.

---

### B02 — Rule input arity is closed

For every registered Phase-1 rule, the rule contract MUST declare exact arity, input family for every positional slot, and invalid-arity behavior.

The Phase-1 rule table is:

| Rule | Inputs | Input families / root constraints | Output | Invalid arity |
|---|---:|---|---|---|
| `EXPR-CANON-001` | exactly 1 | `physmath/expression/1` | `physmath/expression/1` | fail; no output |
| `EXPR-SIMPLIFY-001` | exactly 1 | `physmath/expression/1` | `physmath/expression/1` | fail; no output |
| `EXPR-SUBSTITUTE-001` | `1 + D` | input 0 = `physmath/expression/1`; inputs 1..D = distinct declaring `physmath/function/1` refs appearing in substitution targets | `physmath/expression/1` | fail; no output |
| `RAT-NORMALIZE-001` | exactly 1 | `physmath/expression/1`, root node MUST be `Rational` | `physmath/expression/1` | fail; no output |
| `DIFFERENTIATE-001` | exactly 2 | input 0 = `physmath/expression/1`; input 1 = declaring `physmath/function/1` | `physmath/expression/1` | fail; no output |
| `SELECT-BRANCH-001` | exactly 1 | `physmath/expression/1`, root node MUST be `BranchSet` | `physmath/expression/1` | fail; no output |
| `EXPR-APPLY-001` | exactly 2 | input 0 = `physmath/expression/1`, root node MUST be `Call`; input 1 = `physmath/function/1` | `physmath/expression/1` | fail; no output |

For `EXPR-SUBSTITUTE-001`, `D` is the number of distinct declaring-function artifact refs referenced by the substitution targets. The declaring-function refs appear after the target expression in canonical ArtifactRef byte order. At least one substitution entry is required.

---

### B03 — Canonicalization and simplification traversal are deterministic

`EXPR-CANON-001`:

1. Traverse expression nodes recursively in post-order.
2. Canonicalize every child before canonicalizing the parent.
3. Flatten nested `Add` and `Mul` nodes.
4. Sort commutative children by lexicographic canonical expression bytes.
5. Normalize unary arity where the existing rule permits it.
6. The operation is idempotent: applying it twice produces the same artifact as once.
7. A second global fixpoint pass is not required.

`EXPR-SIMPLIFY-001`:

1. Traverse recursively in post-order.
2. Simplify children first.
3. At each node, repeatedly apply the closed rewrite list until the node is locally stable.
4. Rewrite priority is the order in the normative rule list.
5. After each rewrite, restart rule selection at the top of the priority list for that node.
6. No rewrite outside the closed list is permitted.
7. The operation is deterministic and idempotent.

`EXPR-SIMPLIFY-001` does not add distributivity, factorization, `x+x -> 2x`, theorem proving, or hidden algebraic inference.

This keeps the core small while eliminating traversal-dependent results.

---

### B04 — EXPR-APPLY-001 is root-only

Applicability predicate:

```text
Inputs length = 2
Input[0] is a physmath/expression/1 artifact
Input[0] root node is Call
Input[1] is a physmath/function/1 artifact
Input[0].Call.FunctionRef == Input[1].ArtifactRef
Function.ParameterIDs length == Call.Args length
```

All predicates MUST be checked before rewriting.

If any predicate fails, the operation fails and produces no output artifact.

There is **no NodePath** parameter in Phase 1.

A Call nested inside another expression is not expanded by `EXPR-APPLY-001`. The caller must isolate the nested Call as a top-level `physmath/expression/1` artifact, apply the rule, and then rebuild the enclosing expression.

Nested Calls inside the selected function body are not recursively expanded by the same rule invocation.

---

### B05 — Base/specialized framework dependencies are explicit

The base/specialized split remains locked.

A `FrameworkArtifact` gains an explicit dependency field:

```text
FrameworkArtifact {
    FrameworkID        string
    PackageNamespace   string
    FrameworkVersion   string
    BaseFrameworkRefs  []ArtifactRef // SET
}
```

`BaseFrameworkRefs` targets `<phys-namespace>/framework/1`.

Rules:

1. A framework MUST NOT reference itself.
2. Dependency edges MUST be acyclic.
3. A specialized framework MAY reference assumptions and interpretations owned by itself or by a framework in the transitive dependency closure of its `BaseFrameworkRefs`.
4. It MUST NOT reference unrelated framework namespaces.
5. Duplicate re-declaration of inherited base assumptions is not required.
6. "Depends on" therefore means both package/deployment dependency and artifact-level declared framework dependency represented by `BaseFrameworkRefs`.

Physical assumption/interpretation reference fields therefore target:

```text
<same phys-namespace>/assumption/1
OR
<framework dependency closure namespace>/assumption/1
```

and, where applicable:

```text
<same phys-namespace>/interpretation/1
OR
<framework dependency closure namespace>/interpretation/1
```

This preserves the core/specific split: common assumptions remain in the base; specialized assumptions remain in the specialization.

---

### B06 — One in-memory operation result type

Standardize on:

```text
TransformResult {
    OutputRef     ArtifactRef
    ConditionRefs []ArtifactRef // SET
}
```

`TransformResult` is an in-memory return value only. It is not a durable artifact and has no ArtifactRef of its own.

`ConditionRefs` is represented as a canonical SET in semantic order, i.e. sorted by canonical ArtifactRef bytes.

There is **no `OperationResult` type**.

There is **no `TraceRef` field** in the Phase-1 operation result. Durable derivation tracing is represented by `Derivation` / `DerivationStep` or a standalone `Transformation` artifact. This keeps operation execution separate from trace persistence.

Implementations MUST treat the result as immutable after construction.

---

### B07 — FrameworkVersion is an opaque package-owned value

`FrameworkVersion` is not a global semantic version ontology.

Canonical rule:

```text
FrameworkVersion = non-empty String
encoding = String(FrameworkVersion)
```

The value is opaque to PhysMath. A framework package chooses it; it is part of the framework artifact's canonical payload.

The reference fixture MUST pin the exact framework-version value in its fixture data. Two implementations using different framework versions are correctly producing different framework artifacts; this is not a serialization ambiguity.

---

### B08/B09/B25 — Exact ArtifactRef target-family table

The next amendment MUST publish the following closed Phase-1 target map.

#### PhysMath

```text
Function.BodyRef
    -> physmath/expression/1

FunctionApplication.FunctionRef
    -> physmath/function/1

FunctionApplication.Args[*]
    -> physmath/expression/1

Map.DefinitionRef
    -> physmath/definition/1

Tensor.IndexRefs[*]
    -> physmath/index/1

Tensor.ComponentRefs[*]
    -> physmath/expression/1

Index.SpaceRef
    -> physmath/collection/1

Collection.ElementRefs[*]
    -> physmath/expression/1

OperatorApplication.Args[*]
    -> physmath/expression/1

OperatorApplication.OperatorRef
    -> only a future explicitly registered operator family; absent for DIFFERENTIATE-001

Equation.RelationRef
    -> physmath/relation/1

MathCondition.PredicateRef
    -> physmath/relation/1

Derivation.PremiseRefs[*]
    -> one of the explicitly admitted mathematical support families
       {expression, function, relation, equation, definition, condition}/1

Derivation.ResultRefs[*]
    -> one of
       {expression, function, relation, equation, definition}/1

Theorem.PremiseRefs[*]
    -> one of
       {expression, function, relation, equation, definition, condition}/1

Theorem.ConclusionRef
    -> physmath/relation/1

Theorem.AssumptionRefs[*]
    -> physmath/condition/1

Theorem.DerivationRef
    -> physmath/derivation/1

Definition.DefinedRef
    -> physmath/function/1 OR physmath/expression/1

Definition.DefiningRef
    -> physmath/expression/1
```

The allowed-family union above is closed for Phase 1. There is no phrase such as “explicitly admitted by a later collection contract.”

#### phys-*

```text
FrameworkArtifact
    BaseFrameworkRefs[*] -> <phys-namespace>/framework/1

PhysicalAssumption.ConditionRefs[*]
    -> physmath/condition/1

PhysicalDefinition.FrameworkRef
    -> same package's <phys-namespace>/framework/1

PhysicalDefinition.MathematicalRefs[*]
    -> registered PhysMath mathematical families

PhysicalDefinition.AssumptionRefs[*]
    -> same framework or declared dependency-closure assumption families

PhysicalDefinition.DefinitionEquation
    -> physmath/equation/1 (optional)

PhysicalEquation.FrameworkRef
    -> same package's framework/1

PhysicalEquation.EquationRef
    -> physmath/equation/1

PhysicalEquation.AssumptionRefs[*]
    -> same framework or declared dependency-closure assumption families

PhysicalEquation.ScopeRefs[*]
    -> same framework or declared dependency-closure assumption families

PhysicalTheorem.FrameworkRef
    -> same package's framework/1

PhysicalTheorem.MathematicalTheoremRef
    -> physmath/theorem/1

PhysicalTheorem.AssumptionRefs[*]
    -> same framework or declared dependency-closure assumption families

PhysicalTheorem.InterpretationRefs[*]
    -> same framework or declared dependency-closure interpretation families

PhysicalInterpretation.FrameworkRef
    -> same package's framework/1

PhysicalInterpretation.MathematicalRefs[*]
    -> registered PhysMath mathematical families

PhysicalInterpretation.AssumptionRefs[*]
    -> same framework or declared dependency-closure assumption families
```

Optionality MUST be preserved exactly from the base schema; the target-family map does not change whether a field is optional.

---

### B10 — Relation is a value-type durable artifact

The durable `physmath/relation/1` payload is:

```text
Enum(Operator)
|| CanonicalExpression(Left)
|| CanonicalExpression(Right)
```

The left and right expressions are embedded canonical expression bytes.

They are **not ArtifactRefs**.

The outer relation itself is referenceable through `ArtifactRef` from `Equation`, `MathCondition`, and `Theorem.ConclusionRef`.

The `Expression` node form with the `Relation` node tag and the durable `physmath/relation/1` payload are two encodings of the same mathematical relation concept; the durable family omits the outer expression node tag.

---

### B11 — Exact inline/durable Expression boundary

An expression tree is always recursively inlined as expression-node bytes inside the artifact that owns that tree.

A durable `physmath/expression/1` artifact is the canonical top-level serialization of one complete expression tree.

Therefore:

```text
embedded expression subtree
        ==
canonical payload of a standalone physmath/expression/1
```

when both represent the same subtree.

Transformation inputs and outputs reference top-level expression artifacts; they do not reference individual nodes inside a tree.

Nested expression transformation is therefore performed by isolating the required subtree as a top-level expression artifact, transforming it, and rebuilding the parent expression. This is preferred to adding a generic node-path rewrite facility to Phase 1 because it keeps the transformation algebra small and explicit.

---

### B12 — Function bodies are closed and require no freshening

For every `Function` artifact:

```text
Every Symbol in BodyRef MUST have:
    ParameterID ∈ Function.ParameterIDs
OR
    ConstantID present

A durable Function body with an unbound Symbol is invalid.
```

Within one expression artifact, equal `ParameterID` values denote the same positional parameter identity.

`EXPR-APPLY-001` is capture-free by construction because every bound function-body ParameterID is among the function's declared parameters and every declared parameter has a corresponding argument.

Implementations MUST NOT alpha-freshen, rename, or otherwise mutate ParameterIDs during `EXPR-APPLY-001`.

This is a core simplification: do not invent closure objects, fresh-variable machinery, or capture-avoidance infrastructure in Phase 1.

---

### B13 — EXPR-SUBSTITUTE exact input arrangement

Parameters are encoded as:

```text
UInt32BE(N)
|| Entry_0
|| ...
|| Entry_(N-1)
```

Entries are sorted by their exact canonical entry bytes.

Each entry is:

```text
Encode(BoundParameterRef)
|| EncodeArtifactRef(ReplacementExpression)
```

For `EXPR-SUBSTITUTE-001`:

```text
Input[0] = target Expression artifact
Input[1..D] = declaring Function artifacts referenced by targets
```

The declaring Function refs are distinct and sorted by canonical ArtifactRef bytes.

Each target:

1. MUST reference a declaring Function among `Input[1..D]`.
2. MUST name a ParameterID declared by that Function.
3. MUST match expression Symbols by ParameterID.
4. Is applied recursively through every expression node child.
5. Is simultaneous; replacement expressions are not recursively re-substituted into each other during the same operation.

Duplicate targets are invalid.

---

### B14 — DIFFERENTIATE exact declaring context

`DIFFERENTIATE-001` has exactly two inputs:

```text
Input[0] = Expression
Input[1] = declaring Function
```

Parameters:

```text
Encode(BoundParameterRef)
|| UInt32BE(Order)
```

Validation:

```text
BoundParameterRef.DeclaringArtifactRef == Input[1].ArtifactRef
Target ParameterID is declared by Input[1]
Order >= 1
```

Matching within the expression is by ParameterID.

This removes the asymmetry between substitution and differentiation and ensures differentiation always has an explicit declaration context.

---

### B15 — SELECT-BRANCH-001

Exactly one input:

```text
physmath/expression/1
```

whose root node is `BranchSet`.

Parameters:

```text
UInt32BE(BranchOrdinal)
```

`BranchOrdinal` is zero-based and MUST be in range.

No condition is generated by this rule.

Branch selection is an explicit structural choice, not a mathematical predicate. It therefore MUST NOT inherit a hidden `selected_branch/*` condition mechanism from another system.

---

### B16 — RAT-NORMALIZE-001

Exactly one input:

```text
physmath/expression/1
```

whose root node is `Rational`.

Output is another `physmath/expression/1` artifact containing the normalized Rational node.

Normalization preserves exact value, makes denominator positive, reduces by GCD, and represents zero with denominator one.

---

### B17 — Deferred sorts have no Phase-1 durable family

The Phase-1 durable family registry is closed.

A sort listed for later support but lacking a Phase-1 durable family is:

```text
in-memory only
```

in Phase 1.

It MUST NOT be serialized into an invented artifact family, and it has no Phase-1 executable transformation rules.

This aligns the sort list, family registry, and execution registry without adding new schemas prematurely.

---

### B18 — Phase-1 executable RuleID registry is exhaustive

The only executable Phase-1 `(ProducerNamespace, RuleID)` pairs are:

```text
(physmath, EXPR-CANON-001)
(physmath, EXPR-SIMPLIFY-001)
(physmath, EXPR-SUBSTITUTE-001)
(physmath, RAT-NORMALIZE-001)
(physmath, DIFFERENTIATE-001)
(physmath, SELECT-BRANCH-001)
(physmath, EXPR-APPLY-001)
```

Unknown pairs MUST be rejected during validation/deserialization.

No deferred sort gets an executable rule merely because an agent believes one would be useful.

Framework-local rules are a future extension point, not part of the Phase-1 runtime registry.

---

## 5. Equivariance reference fixture — one normative graph

The reference fixture must not merely be an abstract equation in the specification. It is an executable protocol fixture following the project-standard repository shape:

```text
<fixture>/
    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go
```

The `AGENTS.md` and `manifest.json` follow the same role and structural pattern used by the General Relativity example. `protocol/derivation_test.go` is the executable proof/replay test.

### 5.1 Mathematical realization

Use:

```text
f(v)      = 2 * v
rho_X(g,v) = g * v
rho_Y(g,v) = g * v
```

This makes:

```text
f(rho_X(g,x))
    = 2 * (g * x)
    = 2 * g * x

rho_Y(g,f(x))
    = g * (2 * x)
    = 2 * g * x
```

The two sides close using the Phase-1 canonicalization/simplification rules only. No distributivity or extra theorem-proving rule is required.

### 5.2 Representation choices

```text
f, rho_X, rho_Y
    -> physmath/function/1

LHS / RHS
    -> physmath/expression/1

Equivariance relation
    -> physmath/relation/1

Equation
    -> physmath/equation/1

Theorem
    -> physmath/theorem/1

Physical interpretation
    -> concrete phys-* artifacts
```

Do not create a `GroupAction` sort.

Do not create a new mathematical domain sort merely for `G`, `X`, or `Y`.

### 5.3 Fixture variables g and x

Phase 1 does not introduce a general top-level quantifier/binder system.

For the reference fixture, `g` and `x` are **schematic parameter symbols supplied by the action-function declaration context**, not new global PhysMath constants and not a new binder sort.

The fixture defines:

```text
rho_X.ParameterIDs = [PID_g, PID_x]
rho_Y.ParameterIDs = [PID_g, PID_x]

g = Symbol(ParameterID = PID_g)
x = Symbol(ParameterID = PID_x)
``

where `PID_g` is the ParameterID of declaration slot zero and `PID_x` is the ParameterID of declaration slot one. Because Phase-1 ParameterID derivation is positional and alpha-renaming invariant, `rho_X` and `rho_Y` deliberately use the same slot structure for these schematic arguments.

A Symbol's `ParameterID` is interpreted through the immediately relevant Function/application context; it is not a global variable identity. The fixture MUST include the declaring action Functions among the derivation premises so the parameter context is explicit.

This is a **schematic executable theorem fixture**, not a Phase-1 general quantifier system. General top-level quantified theorem binding remains deferred.

### 5.4 X and Y

`X` and `Y` are represented by framework-owned definitions that point to ordinary registered PhysMath mathematical structures (for example `physmath/collection/1`) rather than by a new domain sort.

They are framework-semantic names, not new PhysMath ontology.

### 5.5 Normative derivation sequence

The fixture MUST use a fixed derivation sequence rather than allowing the coding agent to invent any equivalent proof.

A conforming reference sequence is:

```text
1. Construct inner Call rho_X(g,x).
2. Apply EXPR-APPLY-001 to rho_X.
3. Construct outer Call f(result-of-step-2).
4. Apply EXPR-APPLY-001 to f.
5. EXPR-CANON-001 on LHS result.
6. Construct Call f(x).
7. Apply EXPR-APPLY-001 to f.
8. Construct Call rho_Y(g,result-of-step-7).
9. Apply EXPR-APPLY-001 to rho_Y.
10. EXPR-CANON-001 on RHS result.
11. Validate that LHS and RHS canonical expression bytes are identical.
12. Validate the durable equivariance relation built from those two canonical expressions.
13. Validate the Equation wrapper.
14. Validate the PhysMath Theorem and its Derivation.
15. Validate the concrete phys-* PhysicalTheorem referencing the mathematical theorem.
```

The test fixture MUST pin exact input references and expected outputs for every step.

### 5.6 Relation-side identity

For the relation at step 12:

```text
Relation.Left  = canonical bytes of LHS expression artifact
Relation.Right = canonical bytes of RHS expression artifact
```

No alternate representation is permitted for the reference fixture.

### 5.7 No hidden equality theorem

The reference fixture proves the equivalence by canonical identity of the final expressions under the closed transformation rules. The fixture MUST NOT rely on an unregistered algebraic equality rule.

If the implementation requires an explicit durable relation-construction operation, that operation must be separately specified; the reference test itself MUST demonstrate that the final relation is directly constructible from the canonical LHS/RHS expressions and does not depend on an undocumented inference step.

---

## 6. DerivationStep vs Transformation

These two structures share rule-execution fields but have different lifecycle roles.

### DerivationStep

```text
inline record
inside physmath/derivation/1
no independent ArtifactRef
ordered by Ordinal
```

It exists to record a step in a theorem derivation.

### Transformation

```text
standalone durable artifact
content-addressed
may record one isolated mathematical operation
has an independent ArtifactRef
```

It is not automatically part of a proof merely because it exists.

A `Derivation` does not become a graph of tiny Transformation artifacts. `DerivationStep` is the proof-trace representation.

---

## 7. Generic serialization findings — disposition

Two reviews raised generic `Bytes(T)`, `Optional<T>`, `SEQUENCE<T>`, `SET<T>`, and `ArtifactRef` framing as blockers.

These are valid concerns in isolation but are **not independent new blockers** because the supplied `PM-REP-001 v1.0` already defines them normatively:

```text
String(T)   = UInt32BE(byte_length(T)) || UTF-8(T)
Bytes(B)    = UInt32BE(len(B)) || B
Optional(T) = presence byte || T when present
Sequence(T) = UInt32 count || elements in declared order
Set(T)      = UInt32 count || elements sorted by canonical element bytes
Enum        = canonical String of a closed token
Digest32    = exactly 32 bytes
```

and defines:

```text
ArtifactRef
    field order
    identity tuple
    digest header
    embedded-ref encoding
```

Therefore the correct patch is **not to duplicate these definitions**. The v2 consolidation should explicitly state that they are inherited unchanged from the base specification and that the amendment only adds the fields/rules identified here.

This resolves the reviewer concern while preserving one authoritative encoding source.

---

## 8. phys-* schema findings — disposition

The `FrameworkArtifact`, `PhysicalAssumption`, `PhysicalDefinition`, `PhysicalEquation`, `PhysicalTheorem`, and `PhysicalInterpretation` field layouts already exist in PM-REP-001 v1.0.

The actual remaining issue is not absence of schemas but incomplete amendment-level target maps and dependency semantics.

The next spec amendment therefore:

1. keeps the existing field layouts;
2. adds `BaseFrameworkRefs` to `FrameworkArtifact`;
3. pins all target families;
4. pins SET/SEQUENCE semantics;
5. keeps exact generic encoding inherited from the base spec.

This avoids duplicating an already-defined schema while removing every implementation choice that remains open.

---

## 9. PhysicalDefinition vs PhysicalInterpretation

The boundary is:

```text
PhysicalDefinition
    = what mathematical object/equation is taken to mean
      as a physical object in the framework

PhysicalInterpretation
    = the framework-specific interpretive context
      in which that meaning is assigned
```

A `PhysicalDefinition` can state the framework's concrete physical meaning of a mathematical structure.

A `PhysicalInterpretation` captures the framework's interpretive association to mathematical structures and assumptions.

Neither changes the mathematical identity of the underlying PhysMath artifact.

---

## 10. InterpretationClass disposition

`InterpretationClass` is not restored as a global field.

Phase-1 ruling:

> **All Phase-1 physical artifacts are framework-scoped by construction through `FrameworkRef`; Phase 1 contains no interpretation-neutral physical claims.**

This is sufficient and avoids adding another global ontology key.

---

## 11. Category disposition

`PhysicalAssumption.Category` remains:

```text
string
```

It is a framework-owned semantic label, not a global ontology key.

Its value is encoded using the generic `String()` encoding already defined by PM-REP-001.

No global closed category registry is introduced in Phase 1.

---

## 12. DomainSort / CodomainSort

`Map.DomainSort` and `Map.CodomainSort` are String values whose content MUST be one of the registered Phase-1 PhysMath sort identifiers.

They MUST NOT silently introduce a new sort.

A new value outside the registered sort set is invalid for schema version `1`.

---

## 13. Acceptance and golden-vector requirements

The closure suite must be expanded so the checklist cannot certify an internally incomplete specification.

### 13.1 Serialization

Golden vectors are required for:

```text
ArtifactRef
ParameterID
BoundParameterRef
Relation
Transformation
DerivationStep
Derivation
Theorem
all six phys-* families
TransformResult semantic construction where applicable
```

### 13.2 Rule vectors

One positive and one negative vector is required for each of the seven Phase-1 rules.

Negative vectors MUST include:

```text
wrong arity
wrong input family
invalid root node when applicable
unknown rule pair
invalid parameters
missing declaring context where required
out-of-range branch
```

### 13.3 Parameter/capture vectors

Test:

```text
alpha-renamed Function parameters
invalid foreign ParameterID in Function body
EXPR-APPLY-001 exact argument count
EXPR-APPLY-001 mismatched FunctionRef
EXPR-SUBSTITUTE recursive matching
DIFFERENTIATE declaring-ref mismatch
```

### 13.4 Framework dependency vectors

At minimum:

```text
phys-qm assumption referenced by phys-qm artifact      -> valid
phys-bohm base phys-qm assumption referenced           -> valid
phys-bohm private assumption referenced                -> valid
phys-qm referencing phys-bohm assumption               -> invalid
unrelated phys-cst assumption referenced by phys-bohm  -> invalid
cyclic framework dependency                            -> invalid
```

### 13.5 Many-framework vector

Two framework fixtures may interpret the same mathematical representation differently.

Expected:

```text
same PhysMath artifact identity
separate FrameworkArtifact identities
separate physical assumptions
separate interpretations
no global winner
```

### 13.6 Exact equivariance protocol vector

The acceptance test MUST assert the repository protocol structure:

```text
AGENTS.md present
manifest.json present
protocol/derivation_test.go present
```

and must execute the pinned derivation sequence rather than accepting any mathematically equivalent derivation.

---

## 14. Additional closure point: theorem conclusion must be executable

The v1.0 acceptance wording says the final derivation step produces the theorem conclusion. The review set did not explicitly resolve how the durable relation conclusion is created from the final LHS/RHS expression artifacts.

The next amendment MUST make this concrete in one of two ways:

### Preferred Phase-1 approach

Treat relation construction as a deterministic packaging operation performed by the reference fixture after canonical LHS/RHS equality has been established. It is not an extra theorem-proving rule and does not add a new executable RuleID.

The theorem derivation therefore proves the equality by producing equal canonical expressions; the `physmath/relation/1` artifact packages that equality for durable theorem reference.

### Required acceptance condition

`protocol/derivation_test.go` MUST fail if the theorem conclusion relation cannot be constructed deterministically from the final canonical LHS/RHS expression artifacts.

This prevents an undocumented “equality inference” operation from appearing in the implementation.

If a future implementation instead chooses to model relation construction as an explicit transformation, that must be a future versioned rule addition; it is not silently introduced into Phase 1.

---

## 15. Complexity-architecture audit

Every resolution above should be checked against the three complexity principles.

### Coupling

We remove coupling by keeping:

```text
math ≠ physical semantics
representation ≠ trust
operation result ≠ durable trace
proof step ≠ standalone transformation artifact
Phase 1 ≠ later research machinery
```

### Cohesion

We unify only where an explicit interface is necessary:

```text
ProducerNamespace + RuleID
framework + dependency closure
protocol + theorem derivation
relation + canonical expression sides
```

### Core / specific split

The core contains:

```text
parameter identity
expression representation
canonical transformations
exact derivation mechanics
artifact identity
```

Specific packages contain:

```text
physical assumptions
physical interpretations
framework dependencies
framework-specific meaning
```

No framework-specific concept is promoted into the PhysMath AST merely because it is useful to one theory.

---

## 16. Final closure checklist

`PM-REP-001` + the v2.1 amendment are CLOSED only when every item below is true:

### Architecture

- [ ] Phase-1 / Phase-2 boundary unchanged.
- [ ] PhysMath remains physical-meaning neutral.
- [ ] `phys-*` owns framework semantics.
- [ ] Frozen Phys remains unmodified.
- [ ] Base/specialized framework split is explicit.

### Identity / serialization

- [ ] Generic encoding rules explicitly inherited from PM-REP-001 v1.0.
- [ ] Artifact-family registry complete.
- [ ] All ArtifactRef targets closed.
- [ ] Relation inline payload closed.
- [ ] ParameterID closed.
- [ ] BoundParameterRef closed.
- [ ] All amended field orders explicit.

### Rules

- [ ] ProducerNamespace is physically present in durable rule records.
- [ ] Rule registry is exhaustive.
- [ ] Every rule has exact arity.
- [ ] Every rule has exact input families.
- [ ] Every rule has exact parameters.
- [ ] Invalid application behavior is fail-closed.
- [ ] Canonicalization traversal is deterministic.
- [ ] Simplification traversal is deterministic.
- [ ] EXPR-APPLY boundary is root-only.
- [ ] EXPR-SUBSTITUTE matching is recursive and context-valid.
- [ ] DIFFERENTIATE has explicit declaring context.
- [ ] SELECT-BRANCH has exact input/root/ordinal semantics.
- [ ] RAT-NORMALIZE has exact root semantics.

### Binding

- [ ] Function bodies are closed.
- [ ] Foreign ParameterIDs invalidate a Function body.
- [ ] No alpha-freshening occurs during application.
- [ ] Fixture g/x representation is explicit.

### phys-*

- [ ] Framework dependency closure is explicit.
- [ ] Base assumptions can be referenced by dependent specialized frameworks.
- [ ] Unrelated framework references fail.
- [ ] PhysicalDefinition / PhysicalInterpretation division is explicit.
- [ ] InterpretationClass disposition is explicit.
- [ ] Category is intentionally open.
- [ ] DomainSort / CodomainSort are closed to registered sort IDs.

### Reference protocol

- [ ] GR-style `AGENTS.md` structure is required.
- [ ] GR-style `manifest.json` structure is required.
- [ ] GR-style `protocol/derivation_test.go` structure is required.
- [ ] Equivariance fixture has one normative derivation sequence.
- [ ] Fixture uses no unregistered algebra.
- [ ] Theorem conclusion relation is deterministically constructible.
- [ ] Two-framework interpretation test passes.

### Golden vectors / implementation readiness

- [ ] Positive vectors exist for every durable family.
- [ ] Negative vectors exist for every rule and target-family violation class.
- [ ] Independent implementation can reproduce the same ArtifactRefs.
- [ ] Coding agent can implement without inventing an architectural decision.

---

## 17. Final disposition of the four reviews

### DeepSeek

**Accepted:** all seven newly demonstrated blockers; residual target-family, parameter, grammar, fixture, and golden-vector concerns.

**Refined:** `FrameworkVersion` is treated as an opaque package-owned String rather than a globally standardized version ontology.

**Refined:** `TransformResult` replaces `OperationResult`, but the proposed `TraceRef` is rejected to preserve operation/trace separation.

### Gemini

**Accepted:** AST boundary, relation value-type warning, operation/derivation distinction, and free-variable/capture issues.

**Reclassified:** generic `Bytes`, `Optional`, `SET`, `SEQUENCE`, `ArtifactRef`, phys-* field-layout, and derivation byte-layout concerns are not independent blockers because PM-REP-001 v1.0 already defines the underlying canonical encodings and field schemas.

**Rejected for Phase 1:** `NodePath` parameter in EXPR-APPLY-001; root-only explicit application plus subtree isolation keeps the core smaller.

### Qwen

**Accepted:** result-name collision, usage boundary, target-family closure, relation warning, exhaustive runtime RuleID lock.

**Refined:** its “free variable remains untouched” proposal is rejected. A durable Function body must instead be closed over its declared parameters or constants; this avoids dangling bindings and makes EXPR-APPLY capture-free.

### Z

**Accepted:** ProducerNamespace field, framework dependency contradiction, function-body validation, fixture graph, tractability, relation-copy identity, grammar restoration, deferred-sort alignment, DIFFERENTIATE context, and SELECT-BRANCH no-condition rule.

**Refined:** framework dependencies are represented explicitly in `FrameworkArtifact.BaseFrameworkRefs`, not left as an informal Go-package relationship.

---

## 18. Final conclusion

The four reviews converge on the same fundamental result:

```text
macro-architecture: CLOSED
mechanical contract: NOT YET CLOSED
architecture redesign: NOT REQUIRED
```

The remaining work is a **single specification amendment** that completes decisions already made rather than introducing new conceptual machinery.

The correct next artifact is therefore:

> **PM-REP-001 v1.1 / spec_v2.1 — Mechanical Closure Amendment**

It should incorporate the exact resolutions in this document, then undergo one final blocker-only audit whose sole question is:

> **Can a coding agent, given PM-REP-001 v1.0, this amendment, the frozen Phys authority, and the GR-style reference protocol, implement the system without inventing any architectural decision?**

The desired answer is **yes**.

After that audit, stop architecture iteration and move to implementation.
