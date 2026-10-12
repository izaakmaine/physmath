# Adversarial Review of `plan9.2.md` Against `PM-REP-001 v1.1` Closures

## Executive Verdict
`plan9.2.md` is a masterful refactoring and execution map. It correctly identifies the core/specific split, preserves the `phys-gr` legacy substrate, and locks the module topology. 

However, when read through the lens of the **31 hard mechanical closures** established in `PM-REP-001 v1.1` (the Consolidated Adversarial Review), `plan9.2.md` suffers from a specific handoff risk: **it relies too heavily on "see spec §X" references for the most dangerous execution contracts.** 

A coding agent reading `plan9.2.md` will know *where* to put the code, but might invent the wrong *mechanical shape* for the 7 Phase-1 rules, the equivariance derivation sequence, or the relation payload unless the v1.1 closures are explicitly inlined into the plan's phase descriptions.

Below are the **6 critical gaps** where `plan9.2.md` must be patched to align perfectly with `PM-REP-001 v1.1`, followed by the exact addendum to inject.

---

## ❌ THE 6 CRITICAL GAPS (Mapped to v1.1 Closures)

### Gap 1: Rule Input Arity and Declaring Context (Phase 3 vs. v1.1 B02, B13, B14)
**The Flaw:** `plan9.2.md` Phase 3 mentions "exact §17.2 arity/family table" and "DIFFERENTIATE single-target", but it does not explicitly state that **`DIFFERENTIATE-001` and `EXPR-SUBSTITUTE-001` require the declaring `Function` artifact as an explicit input**.
**The Risk:** The coding agent might implement `DIFFERENTIATE` as a 1-input operation (just the Expression) and try to resolve the parameter context implicitly, violating v1.1 B14. Similarly, they might implement `SUBSTITUTE` without forcing the declaring Functions into Inputs `1..D` (v1.1 B13).
**The Fix:** Explicitly lock the 2-input and (1+D)-input contracts in Phase 3.

### Gap 2: TransformResult Condition Threading (Phase 3 vs. v1.1 B06)
**The Flaw:** `plan9.2.md` Phase 3 states "conditions = supplied set, never discharged". 
**The Risk:** The agent might invent a mechanism where `EXPR-SIMPLIFY-001` or `EXPR-CANON-001` silently generates or drops conditions. v1.1 B06 explicitly mandates that Phase-1 rules generate *zero* new conditions; the output `ConditionRefs` must exactly equal the supplied input `ConditionRefs`.
**The Fix:** Explicitly lock the "no condition generation" rule for all 7 Phase-1 operations in Phase 3.

### Gap 3: Equivariance Exact 6-Step Sequence (Phase 7 vs. v1.1 B26)
**The Flaw:** `plan9.2.md` Phase 7 summarizes the derivation as "steps APPLY, APPLY, CANON, APPLY, APPLY, CANON → 2·g·x both sides".
**The Risk:** The agent might reorder the staged calls or use `EXPR-SIMPLIFY-001` instead of `EXPR-CANON-001`. v1.1 §16.9 (B26) mandates an *exact, normative 6-step sequence* (Step 0: inner rho_X, Step 1: outer f, Step 2: CANON LHS, Step 3: inner f, Step 4: outer rho_Y, Step 5: CANON RHS). 
**The Fix:** Explicitly bind Phase 7 to the exact v1.1 §16.9 normative sequence.

### Gap 4: Relation Value-Type Encoding (Phase 7 vs. v1.1 B10, B21)
**The Flaw:** `plan9.2.md` Phase 7 specifies the relation as `EQ||canon(LHS)||canon(RHS)`.
**The Risk:** The agent might wrap this in an `Expression` node tag (0x09) or use `ArtifactRef` indirection for the left/right sides. v1.1 B10/B21 explicitly mandates that the durable `physmath/relation/1` payload is *exactly* the relation body without the outer Expression tag, and the embedded bytes must be byte-for-byte identical to standalone expression artifacts.
**The Fix:** Explicitly forbid the outer Expression tag and mandate byte-identity with standalone artifacts in Phase 7.

### Gap 5: Deferred Sorts Containment (Phase 2 vs. v1.1 B17)
**The Flaw:** `plan9.2.md` Phase 2 lists the 10-node AST and mentions `OperatorApplication`.
**The Risk:** The agent might see `Functional`, `Limit`, or `AsymptoticTruncation` in the v1.0 sort vocabulary and create durable artifact families for them (e.g., `physmath/limit/1`). v1.1 B17 explicitly mandates that deferred sorts are **in-memory only** and have **no Phase-1 durable family**.
**The Fix:** Explicitly forbid the creation of durable families for deferred sorts in Phase 2.

### Gap 6: Capture-Safety and No Freshening (Phase 3 vs. v1.1 B12)
**The Flaw:** `plan9.2.md` Phase 3 mentions "staged-call discipline" for `EXPR-APPLY-001`.
**The Risk:** The agent might implement a generic capture-avoidance engine (alpha-renaming/freshening) when expanding function bodies. v1.1 B12 explicitly forbids this: `EXPR-APPLY-001` is capture-free by construction because Function bodies are validated to only contain declared parameters. No freshening is permitted.
**The Fix:** Explicitly ban alpha-freshening in Phase 3.

---

## 🛠️ REQUIRED AMENDMENTS TO `plan9.2.md`

Inject the following section into `plan9.2.md` (e.g., as **Section 18: v1.1 Mechanical Execution Locks**) to close the gaps for the coding agent:

```markdown
18. v1.1 Mechanical Execution Locks (Coding Agent Constraints)
The following mechanical decisions are locked per PM-REP-001 v1.1 closures. 
The coding agent MUST NOT invent alternative execution semantics for these contracts.

LOCK 1: Rule Input Arity & Declaring Context (v1.1 B02, B13, B14)
- DIFFERENTIATE-001 requires EXACTLY 2 inputs: Input[0] = Expression, 
  Input[1] = declaring physmath/function/1. Matching is by ParameterID.
- EXPR-SUBSTITUTE-001 requires 1 + D inputs: Input[0] = target Expression; 
  Inputs[1..D] = distinct declaring physmath/function/1 artifacts referenced 
  by the substitution targets.
- EXPR-APPLY-001 requires EXACTLY 2 inputs: Input[0] = Expression rooted 
  at Call; Input[1] = the referenced physmath/function/1.

LOCK 2: TransformResult Condition Threading (v1.1 B06)
- TransformResult is strictly { OutputRef, ConditionRefs SET }.
- Phase-1 rules generate ZERO new mathematical conditions. 
- For all 7 rules, the output ConditionRefs MUST exactly equal the supplied 
  input ConditionRefs. No silent discharge, no implicit generation.

LOCK 3: Equivariance Exact 6-Step Sequence (v1.1 B26)
- The equivariance fixture MUST execute the exact normative 6-step sequence 
  defined in v1.1 §16.9:
  Step 0: EXPR-APPLY-001 on inner rho_X(g,x) -> g*x
  Step 1: EXPR-APPLY-001 on outer f(g*x) -> 2*(g*x)
  Step 2: EXPR-CANON-001 on LHS -> 2*g*x
  Step 3: EXPR-APPLY-001 on inner f(x) -> 2*x
  Step 4: EXPR-APPLY-001 on outer rho_Y(g, 2*x) -> g*(2*x)
  Step 5: EXPR-CANON-001 on RHS -> 2*g*x
- Do not reorder these steps. Do not use EXPR-SIMPLIFY-001 where CANON is mandated.

LOCK 4: Relation Value-Type Encoding (v1.1 B10, B21)
- The durable physmath/relation/1 payload is EXACTLY: 
  Enum(Operator) || CanonicalExpression(Left) || CanonicalExpression(Right).
- There is NO outer Expression node tag (0x09) wrapping the durable relation.
- The embedded Left/Right expression bytes MUST be byte-for-byte identical 
  to the canonical payload of the standalone physmath/expression/1 artifacts 
  produced by the final derivation steps.

LOCK 5: Deferred Sorts Containment (v1.1 B17)
- The following sorts are IN-MEMORY ONLY in Phase 1 and have NO durable 
  artifact family: Functional, FunctionalApplication, Operator, Limit, 
  AsymptoticTruncation, DerivationObligation.
- Do not create physmath/functional/1, physmath/limit/1, etc.

LOCK 6: Capture-Safety & No Freshening (v1.1 B12)
- EXPR-APPLY-001 is capture-free by construction. 
- Implementations MUST NOT alpha-freshen, rename, or mutate ParameterIDs 
  during function body expansion. No fresh-variable allocator is permitted.
```

### Final Handoff Status
With these 6 locks injected, `plan9.2.md` transitions from a "brilliant refactoring map" to a **mechanically unambiguous execution contract**. 

The coding agent now has:
1. Exact input arrangements for the 7 rules (no guessing context).
2. Explicit condition threading (no silent discharge).
3. The exact 6-step equivariance sequence (no reordered proofs).
4. The exact relation byte layout (no hidden tags or indirection).
5. Strict containment of deferred sorts (no invented families).
6. Explicit capture-safety (no invented freshening engines).

**Verdict: CLEARED FOR IMPLEMENTATION.** The architecture is fully closed, the refactoring path is verified against the legacy `phys-gr` substrate, and the mechanical contracts are locked. Hand off to the coding agent.