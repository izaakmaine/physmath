# Adversarial Review of `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.0.md`

## Executive Verdict
Version 1.0 successfully executes the pivot mandated by `plan8.md`: it cleanly separates pure mathematical representation (PhysMath) from physical semantics (phys-*), and correctly targets the **equivariance theorem derivation** as the Phase-1 vertical slice. The artifact identity model, the 16-sort IR, and the formalization protocol are conceptually sound.

However, when subjected to the strict "zero ambiguity for a coding agent" standard, the specification contains **six critical architectural blockers** and **three severe ambiguities**. If handed to a coding agent as-is, the agent will either fail to implement the equivariance reference slice, invent ad-hoc serialization formats, or create an unbridgeable gap between PhysMath expressions and the frozen Phys trust boundary.

Below is the adversarial deconstruction of these flaws and the exact normative patches required to make v1.0 mechanically executable.

---

## ❌ THE 6 CRITICAL BLOCKERS (Must Fix)

### Blocker 1: The Formalization Protocol Integration Gap (The "Kind/Dimension" Void)
**The Flaw:** Section 23 defines the formalization protocol, projecting `physmath/expression/1` artifacts into the frozen Phys hypothesis path (`hypothesis.NewCandidateConcept`). 
**The Reality:** The frozen Phys API strictly requires a `core.Kind` (e.g., `Mass`, `Energy`), a `core.Dimension`, and a `core.AssumptionSet`. A pure PhysMath expression has **none** of these. 
**The Trap:** The coding agent will not know how to bridge this gap. How does a dimensionless mathematical expression become a `core.Object` with dimension $M L^2 T^{-2}$ and kind `Energy`?
**The Fix:** `PhysicalInterpretation` and `PhysicalContract` MUST include explicit mapping fields:
```text
PhysicalContract {
    ...
    TargetPhysKind       string       // e.g., "Energy"
    TargetPhysDimension  Dimension    // exact 7-base rational exponents
    TargetPhysAssumptions []ArtifactRef // refs to MathConditions to be promoted
}
```
Without this, the formalization protocol is unimplementable.

### Blocker 2: Contradiction in `TransformationRuleID` Namespacing
**The Flaw:** Section 15.3 states: *"The Phase-1 closed operation vocabulary is exactly: EXPR-CANON-001... No unregistered RuleID is valid."* But Section 16.2 defines `MathTrace` with a `ProducerNamespace` field, implying that external packages (like `phys-gr`) can record their own local operations.
**The Trap:** If `phys-gr` tries to record a local GR differentiation step, the validator will reject it because its RuleID isn't in the closed PhysMath list.
**The Fix:** Explicitly state that the closed list in §15.3 applies **only** to the `physmath` namespace. The `MathTrace` validation rule must be: `RuleID` is valid if it is registered in the `ProducerNamespace`'s local contract. 

### Blocker 3: Missing Definitional/Logical Expansion Rules for the Reference Slice
**The Flaw:** The Phase-1 target is the derivation of an **equivariance theorem**. Deriving such a theorem inevitably requires steps like "Expand the definition of the Lie derivative" or "Apply the chain rule for group actions."
**The Reality:** The closed `TransformationRuleID` list (§15.3) only contains syntactic/analytic rules (canonicalize, simplify, substitute, differentiate, select branch). There is no rule for **definitional expansion** or **logical substitution**.
**The Fix:** Add `DEFINITION-EXPAND-001` and `LOGICAL-SUBSTITUTE-001` to the closed vocabulary, or explicitly mandate that all definitional expansions must be pre-compiled into `EXPR-SUBSTITUTE-001` operations by the `phys-*` package before recording the trace.

### Blocker 4: Missing Binary Canonical Layouts for phys-* Artifacts
**The Flaw:** Section 28.1 states: *"the canonical payload is a deterministic binary schema-encoding defined by the artifact owner."* Appendix A provides exact binary schemas for PhysMath and Numerical artifacts, but **none** for `PhysicalInterpretation`, `PhysicalContract`, `ProposedInterpretation`, `CorrespondenceProposal`, or `ResearchHypothesis`.
**The Trap:** The coding agent will invent ad-hoc JSON or Go-struct serialization for these, breaking the deterministic digest requirement and making cross-package validation impossible.
**The Fix:** Appendix A MUST include exact binary canonical layouts (field order, primitive encoding, collection ordering) for all `phys-*` and TOE-Agent artifact families, exactly as was done for `ComputationalContract`.

### Blocker 5: The Dual Nature of `Relation` (Artifact vs. Node)
**The Flaw:** Section 9.10 states: *"There are two distinct concepts: Expression node: Relation... Durable artifact: physmath/relation/1"*. Section 14.1 says `MathCondition.Predicate` MUST target `physmath/relation/1`.
**The Trap:** The coding agent will not know whether to create a standalone `Relation` artifact with its own binary payload, or if `physmath/relation/1` is just an `Expression` artifact whose root node happens to be a `Relation` tag.
**The Fix:** Explicitly declare that `physmath/relation/1` is **not** a distinct payload structure. It is an `Expression` artifact (`physmath/expression/1`) whose root node is the `Relation` tag. This prevents duplicate serialization logic.

### Blocker 6: The "Group Action" Conceptual Mapping for Equivariance
**The Flaw:** Section 16 requires representing the equivariance condition: $f(\rho_X(g, x)) = \rho_Y(g, f(x))$. 
**The Reality:** The IR has `Function`, `Map`, and `FunctionApplication`, but no explicit guidance on how to mathematically type a "group action" ($\rho_X: G \times X \to X$) or a "symmetry transformation".
**The Trap:** The coding agent might invent a new `GroupAction` sort, violating the closed 16-sort rule, or fail to represent the theorem.
**The Fix:** Add a normative note in §16 explicitly mapping the equivariance components to the existing 16 sorts:
- Group $G$ and Domains $X, Y$ $\to$ `SetCollection` or `Map` (Domain/Codomain).
- Group actions $\rho_X, \rho_Y$ $\to$ `Function` with `Domain = G x X`, `Codomain = X`.
- Map $f$ $\to$ `Function` or `Map`.
- Equivariance condition $\to$ `Relation` (Equality) between two `FunctionApplication` nodes.

---

## ⚠️ THE 3 SEVERE AMBIGUITIES (Clarify for Coding Agent)

### Ambiguity 1: `DerivationObligation` vs. `Theorem` Completeness
Section 13.2 says a `Theorem` requires a `DerivationRef`. Section 8 says `DerivationObligation` is `DEFERRED + CONTAINMENT`. 
**Clarification needed:** Explicitly state that in Phase 1, **all** `Theorem` artifacts must reference a **complete** `Derivation`. Partial proofs or unfulfilled obligations are not representable in v1.0.

### Ambiguity 2: `OperatorApplication` Parameters for `differentiate`
Section 13.2 defines the parameters for `DIFFERENTIATE-001` as `Target BoundParameterRef` and `Order UInt32`. 
**Clarification needed:** Explicitly state that `differentiate` in v1.0 is strictly limited to **single-variable, scalar differentiation**. Multi-variable partial derivatives, covariant derivatives, and tensor differentiation are out of scope and must be deferred. This prevents the agent from trying to invent a multi-target parameter schema.

### Ambiguity 3: `PhysicalContract` vs. `PhysicalInterpretation` Overlap
Both artifacts contain `MathRefs`, `RequiredConditions`, and `RequiredConventions`. 
**Clarification needed:** Define the exact boundary. `PhysicalInterpretation` is the **semantic meaning** (what this math *is* in this framework). `PhysicalContract` is the **operational interface** (what this math *requires* to be validly composed with other physical objects). A `PhysicalArtifact` references a `PhysicalContract` for its operational rules, and a `PhysicalInterpretation` for its semantic scope.

---

## 🛠️ REQUIRED PATCHES FOR v1.0

Inject the following addendum into the specification before handoff:

```markdown
ADDENDUM TO PM-REP-001 v1.0 — MECHANICAL CLOSURE PATCHES

PATCH 1 (Formalization Gap): 
Add `TargetPhysKind`, `TargetPhysDimension`, and `TargetPhysAssumptions` 
to the `PhysicalContract` and `PhysicalInterpretation` schemas. These fields 
provide the exact mapping required to construct frozen Phys `core.Object` 
values via `hypothesis.NewCandidateConcept`.

PATCH 2 (RuleID Namespacing): 
Amend §15.3: "The closed vocabulary applies strictly to the `physmath` 
namespace. External producers (e.g., `phys-gr`) MAY define local RuleIDs 
within their own namespace. Validation checks `(ProducerNamespace, RuleID)` 
pairs."

PATCH 3 (Definitional Expansion): 
Add `DEFINITION-EXPAND-001` to the closed `TransformationRuleID` vocabulary 
in §15.3. Parameters: `TargetRef ArtifactRef`, `DefinitionRef ArtifactRef`. 
Output: The expanded expression.

PATCH 4 (Binary Layouts): 
Append exact binary canonical schemas to Appendix A for: 
`PhysicalInterpretation`, `PhysicalContract`, `ProposedInterpretation`, 
`CorrespondenceProposal`, `ResearchHypothesis`, `FormalizationProposal`, 
`FormalizationResponse`. Use the exact primitive encoding rules from §7.

PATCH 5 (Relation Duality): 
Amend §9.10 and §14.1: "`physmath/relation/1` is not a distinct payload 
structure. It is an alias for an `Expression` artifact whose root node 
is the `Relation` tag. `MathCondition.Predicate` targets an `Expression` 
artifact with a `Relation` root."

PATCH 6 (Equivariance Mapping): 
Add §16.5 "Equivariance Component Mapping":
- Group/Domains -> SetCollection/Map
- Actions (rho) -> Function (Domain: G x X, Codomain: X)
- Map (f) -> Function/Map
- Equivariance Equation -> Relation (EQ) between FunctionApplications.

PATCH 7 (Scope Limitations): 
Add §13.4 "v1.0 Differentiation Scope": `DIFFERENTIATE-001` is strictly 
limited to single-variable scalar differentiation. Multi-variable, 
covariant, and tensor derivatives are DEFERRED.
Add §12.6 "v1.0 Theorem Completeness": All `Theorem` artifacts MUST 
reference a complete `Derivation`. `DerivationObligation` is deferred; 
partial proofs are not representable in v1.0.
```

### Final Handoff Status
With these 7 patches applied, **v1.0 transitions from a conceptual blueprint to a mechanically executable contract.** 

The coding agent will have:
1. A complete mapping from math expressions to Phys trust objects.
2. A clear, namespaced rule registry for local and global transformations.
3. Exact binary serialization formats for all physical artifacts.
4. A concrete, unambiguous way to represent the equivariance theorem using the closed 16-sort IR.

**Verdict: CLEARED FOR IMPLEMENTATION PLANNING** once the 7 patches are integrated into the text.