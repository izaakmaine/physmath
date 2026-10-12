# Adversarial Review of `spec_v2.md` (Consolidated Review of PM-REP-001)

## Executive Verdict
`spec_v2.md` successfully closes the macro-architecture. The separation of PhysMath, `phys-*`, and the frozen Phys authority is now mechanically sound, and the rejection of Phase-2 scope creep (TOE-Agent, Numerical, Trust) is correct. 

However, when subjected to the strict "zero ambiguity for the coding agent" standard, **`spec_v2.md` contains 5 critical micro-blockers and 2 semantic gaps**. If handed to a coding agent as-is, the agent will either invent architecture to fill the vague `ArtifactRef` targets, create duplicate in-memory structs due to naming collisions, or fail to serialize the `Relation` artifact correctly.

Below is the exact deconstruction of these flaws and the normative patches required to make the document 100% implementation-ready.

---

## ❌ THE 5 CRITICAL BLOCKERS (Must Fix)

### Blocker 1: The In-Memory Result Naming Collision
**The Flaw:** Section 3G defines the ephemeral operation return type as `TransformResult { Output, Conditions, Trace }`. However, Section 10 (Residual Clarifications) defines it as `OperationResult { OutputRef, ConditionRefs SET }`. 
**The Trap:** The coding agent will implement two different structs, or guess which fields are required, breaking the uniform transformation pipeline.
**The Fix:** Unify the name and schema. Standardize on `TransformResult` and explicitly define its fields:
```text
TransformResult {
    OutputRef     ArtifactRef
    ConditionRefs []ArtifactRef  // SET
    TraceRef      ArtifactRef?   // Optional, for durable tracking
}
```
*Rule:* Delete all references to `OperationResult`.

### Blocker 2: `DerivationStep` vs `Transformation` Artifact Overlap
**The Flaw:** Both `DerivationStep` (inside a `Derivation`) and the durable `Transformation` artifact (Section 10.8) contain `RuleID`, `Inputs`, `Output`, and `Parameters`. The spec does not explain *when* to use which.
**The Trap:** The agent will either make `Derivation` reference `Transformation` artifacts (creating a massive graph of tiny artifacts) or duplicate the schema, violating the "no silent loss" rule.
**The Fix:** Explicitly define their distinct use cases:
*   **`DerivationStep`**: Used *only* inline within a `Derivation` artifact to prove a theorem. It does not have its own `ArtifactRef`.
*   **`Transformation`**: A standalone, durable, content-addressed artifact used to record a single, isolated mathematical operation (e.g., a canonicalization step outside a formal proof). 
*Rule:* Add a "Usage Boundary" note to Section 10.

### Blocker 3: Vague `ArtifactRef` Targets for Phase 1
**The Flaw:** Section 4I leaves several `ArtifactRef` targets open-ended with phrases like *"phase-1 mathematical durable artifacts explicitly admitted by the collection contract"* or *"framework-owned artifacts explicitly registered"*.
**The Trap:** The coding agent cannot guess what is "explicitly admitted." It will either halt or invent a generic registry.
**The Fix:** Pin every vague target to an exact Phase 1 family:
*   `Collection.ElementRefs` $\to$ `physmath/expression/1` (MVP only)
*   `Definition.DefinedRef` $\to$ `physmath/function/1` OR `physmath/expression/1`
*   `Definition.DefiningRef` $\to$ `physmath/expression/1`
*   `PhysicalEquation.ScopeRefs` $\to$ `<phys-namespace>/assumption/1`
*   `Theorem.PremiseRefs` $\to$ `physmath/function/1` OR `physmath/expression/1` (Required for the equivariance fixture to reference the definitions of $f, \rho_X, \rho_Y$).

### Blocker 4: Missing `PhysicalAssumption.ConditionRefs` Target
**The Flaw:** PM-REP-001 Section 14.2 defines `PhysicalAssumption` with a `ConditionRefs` field, but Section 4I of `spec_v2.md` completely omits the target mapping for it.
**The Trap:** The agent won't know what physical assumptions point to.
**The Fix:** Add to Section 4I:
*   `PhysicalAssumption.ConditionRefs` $\to$ `physmath/condition/1`

### Blocker 5: The `Relation` Value-Type vs Reference-Type Trap
**The Flaw:** Section 2E states that the durable `physmath/relation/1` payload is `Enum(Operator) || CanonicalExpression(Left) || CanonicalExpression(Right)`. 
**The Trap:** Because almost every other artifact uses `ArtifactRef` to point to its children, the coding agent will instinctively make `Left` and `Right` into `ArtifactRef`s pointing to `Expression` artifacts. This violates the spec and breaks the canonical byte layout.
**The Fix:** Add a massive, explicit warning to Section 2E and Section 4I:
> **CRITICAL SERIALIZATION RULE:** `physmath/relation/1` is a **value-type** artifact. It embeds the full canonical expression bytes *inline*. It does NOT use `ArtifactRef`s for its operands. (Contrast this with `Equation.RelationRef`, which *is* a reference-type artifact pointing to the relation).

---

## ⚠️ THE 2 SEMANTIC GAPS (Must Clarify)

### Gap 1: `EXPR-APPLY-001` Free Variable Handling
**The Issue:** Section 3G defines `EXPR-APPLY-001` as simultaneous substitution of arguments into a `Function` body. But what if the `Function` body contains a `Symbol` with a `ParameterID` that is *not* in the function's `ParameterIDs` list?
**The Fix:** Explicitly state: "Symbols in the function body with `ParameterID`s not present in the `Function.ParameterIDs` list are treated as free variables. They remain completely untouched during the `EXPR-APPLY-001` substitution."

### Gap 2: The Phase 1 Execution Registry is Not Closed
**The Issue:** Section 3G lists the transformation rules, but doesn't explicitly forbid the agent from inventing new ones for deferred sorts.
**The Fix:** Add a hard lock to Section 3G:
> **PHASE 1 EXECUTION LOCK:** The ONLY valid `TransformationRuleID`s for Phase 1 runtime execution are: `EXPR-CANON-001`, `EXPR-SIMPLIFY-001`, `EXPR-SUBSTITUTE-001`, `RAT-NORMALIZE-001`, `DIFFERENTIATE-001`, `SELECT-BRANCH-001`, `EXPR-APPLY-001`. Any other `RuleID` MUST be rejected during deserialization. Deferred sorts (Tensor, Functional, etc.) have no executable rules in Phase 1.

---

## 🛠️ REQUIRED PATCHES FOR HANDOFF

Inject the following addendum into `spec_v2.md` before handing it to the coding agent:

```markdown
ADDENDUM TO spec_v2.md — MECHANICAL CLOSURE PATCHES

PATCH 1 (Naming Unification): 
Delete all references to `OperationResult`. The ephemeral in-memory 
return type for all PhysMath operations is exactly `TransformResult`, 
containing `OutputRef`, `ConditionRefs` (SET), and optional `TraceRef`.

PATCH 2 (Artifact Usage Boundary): 
`DerivationStep` is an inline record used ONLY within a `Derivation` 
artifact. `Transformation` is a standalone, durable, content-addressed 
artifact used for isolated operations. They share schema fields but 
have distinct lifecycle and identity rules.

PATCH 3 (Pinned ArtifactRef Targets): 
The following vague targets are now strictly pinned for Phase 1:
- Collection.ElementRefs -> physmath/expression/1
- Definition.DefinedRef -> physmath/function/1 OR physmath/expression/1
- Definition.DefiningRef -> physmath/expression/1
- PhysicalEquation.ScopeRefs -> <phys-namespace>/assumption/1
- Theorem.PremiseRefs -> physmath/function/1 OR physmath/expression/1
- PhysicalAssumption.ConditionRefs -> physmath/condition/1

PATCH 4 (Relation Serialization Warning): 
`physmath/relation/1` is a VALUE-TYPE artifact. Its `Left` and `Right` 
fields are embedded canonical expression bytes, NOT ArtifactRefs. 
Do not use reference indirection for relation operands.

PATCH 5 (Free Variable Rule): 
Under `EXPR-APPLY-001`, any `Symbol` in the function body whose 
`ParameterID` is not in the function's `ParameterIDs` list is a free 
variable and remains untouched during substitution.

PATCH 6 (Phase 1 RuleID Lock): 
The Phase 1 executable RuleID registry is EXHAUSTIVELY CLOSED to the 
7 rules listed in Section 3G. Deserialization MUST reject any unknown 
RuleID. Deferred sorts have no executable rules in Phase 1.
```

### Final Handoff Status
With these 6 patches applied, `spec_v2.md` transitions from a "conceptually closed architecture" to a **mechanically executable contract**. 

The coding agent will have:
1. Zero naming collisions for in-memory types.
2. Clear boundaries between inline proof steps and durable operation records.
3. Exact, unguessable `ArtifactRef` targets for every single field.
4. Explicit serialization rules to prevent the `Relation` value-type trap.
5. A closed, locked registry for Phase 1 execution.

**Verdict: CLEARED FOR IMPLEMENTATION PLANNING.**