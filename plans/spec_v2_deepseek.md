## Adversarial Review — `spec_v2.md` (Consolidated Review of PM-REP-001 v1.0)

**Context reviewed:** `spec_v2.md` as a consolidated blocker-closure document intended to be read together with `PM-REP-001 v1.0`. Target: a coding agent, given only the two documents, must not have to invent any architectural decision.

**Verdict:**  
**NOT CLOSED. SEVEN DEMONSTRATED BLOCKERS REMAIN.**

`spec_v2` is the strongest closure document in the series so far. It resolves all nine blockers raised against `PM-REP-001 v1.0` and adds the missing `EXPR-APPLY-001` operation, the master artifact-family registry, `BigInt`, `ConstantID`, closed-enum encoding, `BoundParameterRef` encoding, and target-family maps. It also correctly rejects the proposed Frozen-Phys integration and TOE-Agent contamination.

But it *introduces* the `(ProducerNamespace, RuleID)` rule namespace without adding a field to the durable schemas, leaves rule **input arity** unstated for every rule, doesn’t specify whether `EXPR-SIMPLIFY-001` is recursive, doesn’t pin `EXPR-APPLY-001` boundary behavior, and states a “minimum-assumption rule” that **contradicts** its own `AssumptionRefs` target-family map. In each case, two legitimate implementations remain possible.

---

## I. Blockers resolved from `PM-REP-001 v1.0`

These are now pinned and should not be reopened:

- §A master durable artifact-family registry (all `physmath/*` and `<phys-namespace>/*` families).
- §B `BigInt` sign+length+minimal-magnitude encoding.
- §B `ConstantID = string` with `String()` encoding.
- §B closed-enum encoding as `String(token)`.
- §C `ParameterID` scope reduced to Function-only binding.
- §D `Encode(BoundParameterRef) = EncodeArtifactRef(...) || Digest32(ParameterID)`.
- §E `Relation` node vs `physmath/relation/1` durable payload, without wrapper.
- §F `ComparisonOperator` encoding as `Enum = String(token)`.
- §G exact parameter bytes for all six previous rules plus the new `EXPR-APPLY-001`.
- §H exact `EXPR-CANON-001` and `EXPR-SIMPLIFY-001` semantics, and `RAT-NORMALIZE-001` / `SELECT-BRANCH-001` semantics.
- §I complete target-family map for every `ArtifactRef`-typed field in the schemas.
- §J `EquationSet.EquationRefs` pinned to `SEQUENCE`.
- §K `Tensor.Dimension` restored with `len(ComponentRefs) = Dimension^Rank`.
- §L exact equivariance fixture composition.
- §M `EXPR-APPLY-001` added to the closed PhysMath rule registry.
- §N substitution context requires the declaring artifact in the inputs.
- §O rule identity declared as the pair `(ProducerNamespace, RuleID)`.
- §P `phys-*` family names pinned.
- §Q `PhysicalContract` removed — a simplification, not a merge.
- §R `phys-*` canonical serialization pinned.

---

## II. Demonstrated blockers

Each blocker identifies the section, a concrete counterexample, and the failure consequence.

### Blocker 1 — `ProducerNamespace` is not placed into the `Transformation` or `DerivationStep` schema

- §O declares: “Rule identity is the pair `(ProducerNamespace, RuleID)`” and “The durable `Transformation` and `DerivationStep` representations must carry the producer namespace needed to resolve the RuleID.”
- `PM-REP-001 §10.8` `Transformation` schema has `RuleID TransformationRuleID` and no producer-namespace field.
- `PM-REP-001 §12.2` `DerivationStep` schema has `RuleID TransformationRuleID` and no producer-namespace field.

**Concrete counterexample:** A coding agent must decide whether to add `ProducerNamespace string` as a sibling field, prefix the `RuleID` string (e.g. `phys-gr:EXPAND-001`), or embed it inside a tagged structure. Three legitimate implementations produce three different canonical payloads for the same logical derivation step. Their `Transformation` artifacts are not comparable across implementations.

**Failure consequence:** `Derivation` and `Theorem` artifacts are non-portable. Cross-implementation replay fails at the very first rule lookup. Golden vectors for the equivariance fixture cannot be standardized.

**Exit condition:** Publish the exact updated schema for `Transformation` and `DerivationStep` including the producer-namespace field, its type, and its canonical position.

---

### Blocker 2 — Rule input arity is never pinned

- §G lists parameter bytes and (for some rules) mentions `Input[0]` and `Input[1]`.
- No section states how many inputs each rule takes.
- `Transformation.Inputs` and `DerivationStep.InputRefs` are `SEQUENCE` of `ArtifactRef`, so syntactically any count is legal.
- Rules like `EXPR-CANON-001`, `EXPR-SIMPLIFY-001`, `RAT-NORMALIZE-001`, `SELECT-BRANCH-001` never declare their input shape.

**Concrete counterexample:** A coding agent implements `EXPR-CANON-001`. Does it require exactly one input, or does it allow zero, or does it allow multiple and canonicalize each independently? Two implementations accept different artifact graphs. A third accepts multiple inputs and folds them into one output; a fourth rejects multiple inputs with `TRANSFORMATION_FAILED`.

**Failure consequence:** Derivation replay divergence across implementations; a `DerivationStep` produced by one implementation cannot be validated by another even though both conform to §G.

**Exit condition:** For every rule in the registry, declare `Inputs` arity, `Inputs[i]` target family, and the behavior when arity is violated.

---

### Blocker 3 — Recursion policy for `EXPR-SIMPLIFY-001` is not stated

- §H lists the closed rewrite rules: `Neg(Neg(x)) -> x`, `Add(..., 0, ...) -> Add(...)`, `Mul(..., 1, ...) -> Mul(...)`, `Mul(..., 0, ...) -> Rational(0/1)`, rational arithmetic canonicalization, `Pow(x, 1) -> x`, and the conditioned `Pow(x, 0)` rule.
- §H does not state whether these rules are applied at the root only or recursively to sub-expressions.
- §H does not state whether the rewrite is iterated to fixpoint.

**Concrete counterexample:** Input `Add([Mul([Rational(1/1), x]), Rational(0/1)])`. One implementation applies rules only at the root: it becomes `Mul([Rational(1/1), x])` (unary unwrap), then `x` (unary unwrap) — two steps, non-recursive is still ambiguous. Another implementation applies recursively in one step. A third applies only at the root and stops at `Add([Mul([1,x]), 0])` because the addend-zero removal is applied only if the root Add has more than two children. All three are consistent with the text.

**Failure consequence:** The same input produces different outputs across implementations; `ArtifactRef`s diverge; the canonical golden vectors cannot be standardized.

**Exit condition:** Declare whether each rule is applied at the root only, recursively, or to a fixpoint; declare the traversal order for recursive application.

---

### Blocker 4 — `EXPR-APPLY-001` boundary behavior is unspecified

- §G for `EXPR-APPLY-001`:
  - `Input[0] = Expression containing a root Call`
  - `Input[1] = the Function artifact referenced by that Call`
  - “Function arity must equal Call argument count”
  - “nested Calls are not recursively expanded”
- Not stated: what happens when `Input[0]` is not a Call at the root; whether “root Call” means the outermost node must be a Call, or that a Call must be reachable; whether `Call.FunctionRef` is dereferenced or compared by equality to `Input[1]`’s `ArtifactRef`; whether the comparison occurs before or after validation of other inputs.

**Concrete counterexample:** A coding agent encounters `Add([Call(f, x), y])` as `Input[0]`. Is the rule applicable (because a Call exists somewhere) or rejected (because the root is not a Call)? Two implementations legitimately disagree. A third implementation expands the nested `Call(f, x)` even though §G says “nested Calls are not recursively expanded”, because the Call is now at the root of a sub-expression.

**Failure consequence:** Derivation with `EXPR-APPLY-001` is non-deterministic across implementations; the equivariance fixture’s canonical artifact graph is not reproducible.

**Exit condition:** Declare the exact applicability predicate: whether the root node must be a Call, what happens otherwise (which `OperationOutcome`), and whether the `Call.FunctionRef` must byte-equal `Input[1]`’s `ArtifactRef`.

---

### Blocker 5 — phys-* cross-namespace assumption references are contradictory

- §P: “A specialized framework may depend on the base framework: `phys-qm ← phys-bohm`.”
- §P: “Interpretation-specific assumptions MUST NOT be placed in the base framework.”
- §I: `PhysicalEquation.AssumptionRefs -> <same phys-namespace>/assumption/1`.
- §I: `PhysicalTheorem.AssumptionRefs -> <same phys-namespace>/assumption/1`.
- §I: `PhysicalDefinition.AssumptionRefs -> <same phys-namespace>/assumption/1`.
- §I: `PhysicalInterpretation.AssumptionRefs -> <same phys-namespace>/assumption/1`.

If `phys-bohm` is a specialized framework built on top of `phys-qm`, and it depends on `phys-qm`’s assumptions, then `phys-bohm`-owned physical artifacts must be able to reference `phys-qm/assumption/1`. The target-family map forbids this. Either the “minimum-assumption rule” cannot be applied, or the target-family map is wrong.

**Concrete counterexample:** `phys-bohm` declares `PhysicalEquation` for a guidance law that reuses the `phys-qm` Hilbert-space assumption. Its `AssumptionRefs` must point to `phys-bohm/assumption/1` — which duplicates the assumption — or it must break the family rule.

**Failure consequence:** Duplicate assumption artifacts across namespaces; the “hundred physicists, hundred answers” principle is undermined because `phys-bohm` is forced to re-declare `phys-qm` assumptions instead of referring to them.

**Exit condition:** Either amend the target-family rule to permit `<any-ancestor-namespace>/assumption/1` (with an explicit dependency declaration), or amend §P to require full assumption re-declaration in each specialized framework, and state which one is canonical.

---

### Blocker 6 — `OperationResult` schema is not pinned

- §10 residual locks: “Every PhysMath operation returns: `OperationResult { OutputRef, ConditionRefs SET }`.”
- No type declaration is given for `OutputRef` (is it `ArtifactRef`?) or for `ConditionRefs` (is it `[]ArtifactRef` or `[]MathCondition`?).
- §10 says it is “in-memory only”, but does not state whether it is a struct, a tuple, or an interface.
- §10 also says “No hidden condition discharge”, but does not say whether the result is mutable.

**Concrete counterexample:** A coding agent defines `OperationResult { OutputRef ArtifactRef; ConditionRefs []ArtifactRef }`. Another defines `OperationResult { OutputRef Expression; ConditionRefs []MathCondition }`. Both implement every rule in §G–§H. Their callers cannot interoperate.

**Failure consequence:** Public PhysMath API is not portable across implementations; the equivariance fixture cannot be exercised consistently against either API.

**Exit condition:** Publish the exact Go declaration of `OperationResult`, including field types and mutability.

---

### Blocker 7 — `FrameworkVersion` canonical form is unspecified

- `PM-REP-001 §14.1`: `FrameworkArtifact { FrameworkID, PackageNamespace, FrameworkVersion string }`.
- `spec_v2 §P`: `<namespace>/framework/1`.
- The family ID uses `1` as `SchemaVersion`. Whether `FrameworkVersion` is `"1"`, `"1.0"`, `"v1.0.0"`, or an arbitrary string is not stated.

**Concrete counterexample:** A coding agent must populate `FrameworkVersion` for `phys-qm`’s framework artifact. One writes `"1"`. Another writes `"2026-01"`. A third writes `"1.0.0"`. All satisfy every rule in `spec_v2`. Their `phys-qm/framework/1` artifacts diverge in bytes; different downstream references cannot be matched.

**Failure condition:** Cross-implementation framework identity is not portable; the §16.6 “many frameworks” acceptance test cannot be enforced because the framework artifacts disagree.

**Exit condition:** Pin the exact format of `FrameworkVersion` (a closed token like the artifact `SchemaVersion`, or an explicit schema).

---

## III. Residual issues

Not demonstrated blockers, but a coding agent will encounter them.

- **`Derivation.PremiseRefs` and `Derivation.ResultRefs` target family not closed.** §I says “explicit mathematical support artifacts or expressions”. That is a description, not a family registry. A coding agent must decide whether “expressions” means the durable `physmath/expression/1` artifact or an in-memory expression.
- **`Transformation.Inputs` target family not closed.** §I says “schema-declared producer artifacts”. A coding agent must decide whether this admits arbitrary `physmath/*` families or only those explicitly permitted by each rule.
- **`EquationSet.EquationRefs` SEQUENCE pinned, but no rule states whether duplicates are allowed.** A repeated equation reference may be legitimate (a derivation may cite the same equation twice) or invalid (a paper’s equation list is unique). Not stated.
- **`Encode(BoundParameterRef)` is defined only for use inside rule parameters.** Whether the same encoding is used when a `BoundParameterRef` appears elsewhere (e.g., in a future durable artifact) is not stated.
- **`EXPR-SUBSTITUTE-001` declaring-artifact inputs are in `SET` or `SEQUENCE` order?** §G says “each `BoundParameterRef.DeclaringArtifactRef` must appear among the rule inputs”, but does not specify which position, nor whether the list of declaring artifacts is a `SET` or a `SEQUENCE`.
- **`EXPR-SUBSTITUTE-001` matching inside the target expression.** §G says “matching inside the expression is by ParameterID”. Does that mean the expression’s `Symbol`s carry `ParameterID = target.ParameterID`? Yes, per `PM-REP-001 §7.3`. But what about `Call.Args` and nested expressions — does substitution recurse through all expression node kinds? Presumably yes, but not stated.
- **`SELECT-BRANCH-001` additional input validation.** §H says “Input must be a BranchSet”. Does the rule require exactly one input, of family `physmath/expression/1` whose root is a `BranchSet`? Or a `Collection` of branches? Not stated.
- **`EXPR-SIMPLIFY-001` `Pow(x, 0)` case.** §H says “the rewrite to `1` is permitted only with an explicit `MathCondition` proving `x != 0`. Otherwise the operation fails”. Two failure modes remain: `UNDEFINED` vs `OUT_OF_SCOPE`. Not stated which outcome is returned.
- **`RAT-NORMALIZE-001` input.** §H says “Input must be a Rational artifact”. `Rational` is an Expression node kind, not a durable artifact family. Does the rule take `physmath/expression/1` whose root is `Rational`, or a distinct durable rational family? `spec_v2 §A` does not register a rational family.
- **Golden vector reproducibility for the equivariance fixture.** §L pins the fixture’s shape, but the exact derivation chain (which RuleIDs, in which order) is left to the coding agent. Two conforming implementations will produce two different fixtures and therefore two different golden vectors. This is not an architectural blocker, but it is a test-design gap; if the intent is to fix the fixture so cross-implementation comparison is possible, the exact sequence of RuleIDs and inputs must be pinned.
- **`PhysicalAssumption.Category`** is `string` in `PM-REP-001 §14.2`. Whether it is a closed token or free-form is unstated; §2 of `PM-REP-001` implies it is framework-owned, so permissive is consistent. Not a blocker, but the review should state that it is intentionally open.
- **`PhysicalDefinition.DefinitionEquation`** uses `Optional<ArtifactRef>` in `PM-REP-001 §14.3` but `spec_v2 §I` does not distinguish Optional from required in the target-family map.

---

## IV. Path to closure

To satisfy the target, the amendment must resolve the seven blockers above. Concretely:

1. **Blocker 1.** Publish updated `Transformation` and `DerivationStep` schemas with `ProducerNamespace string` (or an explicitly chosen alternative), including canonical field position.
2. **Blocker 2.** For each rule in the registry, declare `Inputs` arity, `Inputs[i]` target family, and violation behavior.
3. **Blocker 3.** Declare recursion policy for `EXPR-SIMPLIFY-001` and `EXPR-CANON-001`.
4. **Blocker 4.** Declare the exact applicability predicate for `EXPR-APPLY-001`, including the failure outcome when the predicate is not met.
5. **Blocker 5.** Reconcile the minimum-assumption rule with the `<same phys-namespace>` target-family rule, one way or the other.
6. **Blocker 6.** Publish the exact Go declaration of `OperationResult`.
7. **Blocker 7.** Pin `FrameworkVersion` canonical form.

Then address the residual issues as one-line clarifications: rational-input family for `RAT-NORMALIZE-001`, `SELECT-BRANCH-001` input shape, `EXPR-SIMPLIFY-001` `Pow(x,0)` outcome class, `Derivation.PremiseRefs` / `Transformation.Inputs` family restrictions, `BoundParameterRef` general encoding, `EXPR-SUBSTITUTE-001` declaring-artifact input layout.

Finally, add golden vectors for each new pin so the equivariance fixture’s derivation becomes cross-implementation comparable.

---

## V. Final verdict

```text
REVIEW: spec_v2 (Consolidated review of PM-REP-001 v1.0)

Strategic architecture:                          PASS
Phase-1 / Phase-2 separation:                    PASS
Paper-as-research-unit:                          PASS
ARI boundary:                                    PASS
PhysMath purity:                                 PASS
phys-* physical-semantics ownership:             PASS
Many-frameworks principle:                       PASS
Artifact-family registry:                        PASS
BigInt encoding:                                 PASS
ConstantID encoding:                             PASS
Enum encoding:                                   PASS
ParameterID scope rules:                         PASS
BoundParameterRef encoding:                      PASS
Relation node vs durable payload:                PASS
ComparisonOperator encoding:                     PASS
Rule parameter bytes (six+one rules):            PASS
Rule semantics (canonicalize/simplify/etc.):     PASS
ArtifactRef target families:                     PASS
EquationSet ordering:                            PASS
Tensor.Dimension:                                PASS
Equivariance fixture composition:                PASS
EXPR-APPLY-001 existence:                        PASS
Substitution declaring-artifact rule:            PASS
RuleID namespace pair decision:                  PARTIAL (Blocker 1)
phys-* family names:                             PASS
PhysicalContract removed:                        PASS
phys-* serialization rule:                       PASS
Frozen-Phys integration rejected:                PASS
TOE-Agent artifacts rejected:                    PASS
Trust/provenance deferred:                       PASS

Blocker 1  ProducerNamespace schema field:       OPEN
Blocker 2  Rule Input arity:                     OPEN
Blocker 3  EXPR-SIMPLIFY recursion policy:       OPEN
Blocker 4  EXPR-APPLY-001 boundary behavior:     OPEN
Blocker 5  phys-* cross-namespace assumptions:  OPEN
Blocker 6  OperationResult schema:               OPEN
Blocker 7  FrameworkVersion canonical form:      OPEN

FINAL STATUS: NOT CLOSED. SEVEN DEMONSTRATED BLOCKERS.
```

---

## VI. Assessment

`spec_v2` is the *closest* document in the entire series to the target. Its strategic decisions are correct; it explicitly rejects the right Phase-2 contaminants; it registers the correct families; it pins most of the byte-level encodings that a coding agent needs.

The seven remaining blockers are contract-pinning issues, not architecture redesigns. None of them requires revisiting the Phase-1/Phase-2 split, the ARI framing, the phys-* ownership rule, or the many-frameworks principle. All can be resolved in a single `spec_v2.1` amendment.

If those seven are resolved — plus the residual clarifications — `spec_v2` (as amended into `PM-REP-001 v1.1`) would satisfy its own closure condition and the coding agent could proceed without inventing any mathematical-identity rule, serialization rule, transformation rule, derivation rule, or cross-layer ownership rule.

The cycle is now genuinely one amendment away from closure, not another architecture round.