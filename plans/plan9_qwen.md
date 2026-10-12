# Adversarial Review of `plan9.md` Against `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md`

## Executive Verdict

`plan9.md` (GR stress test) and `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md` (PhysMath representation spec) are **complementary but operate at different architectural levels**. plan9.md operates against frozen Phys (specs_v2_3.md); v1.2 defines the PhysMath + phys-* representation layer. They are not contradictory, but their **relationship is undefined**, which creates implementation ambiguity.

After careful cross-referencing, I identify **3 architectural blockers** and **2 ambiguities** that must be resolved before a coding agent can execute without making architectural decisions.

---

## BLOCKER 1: Undefined Scope Relationship Between plan9.md and v1.2

**The Problem:**
plan9.md and v1.2 operate at different architectural levels but their relationship is never defined.

- v1.2 defines: PhysMath + phys-* representation (Phase 1)
- plan9.md defines: GR stress test against frozen Phys (separate workload)

A coding agent will ask: *"Do I implement PhysMath first, then GR? Does GR use PhysMath expressions or `core.Expr`? Are these sequential or parallel?"*

**Evidence from the documents:**
- plan9.md says: *"scalar rational algebra uses `core.Expr`"* — meaning frozen Phys, not PhysMath
- plan9.md says: *"GR-specific objects... should initially be Level-1 Go structures, not `core.Object` instances"*
- v1.2 says: phys-* packages depend on PhysMath + Phys
- But plan9.md's GR uses `core.Expr` (frozen Phys), not PhysMath expressions

**The Ambiguity:** When plan9.md says GR uses `core.Expr` for "scalar rational algebra," does this mean:
- (a) GR uses frozen Phys `core.Expr` directly (bypassing PhysMath), or
- (b) GR uses PhysMath expressions that happen to be backed by `core.Expr`?

**Required Resolution:**
plan9.md must explicitly state:
> "The GR stress test operates against frozen Phys (`core.Expr`, `core.Object`) as a hostile workload. It does not use PhysMath expressions. PhysMath integration is a post-GR Phase-2 concern. GR-local mathematical structures (trig, tensors, differentiator) are Level-1 Go structures that hold `core.Expr` components for scalar subexpressions only."

---

## BLOCKER 2: GR-Local Operations vs v1.2 Phase-1 Rules

**The Problem:**
v1.2 defines exactly 7 executable Phase-1 rules (§11.5, §17):
`EXPR-CANON-001, EXPR-SIMPLIFY-001, EXPR-SUBSTITUTE-001, RAT-NORMALIZE-001, DIFFERENTIATE-001, SELECT-BRANCH-001, EXPR-APPLY-001`

plan9.md requires GR-local operations that are **not** in this set:
- GR-local differentiator (GR-3b): handles `r⁻¹`, `r⁻²`, `sin(θ)`, `cos(θ)`
- GR-local `Normalize` / `ZeroTest` (§9 of plan9.md)
- GR-local weak-field reduction (GR-8)
- GR-local trig: `sin(θ)`, `cos(θ)`

v1.2 §15.4 says framework-local transformations are allowed "only under a future versioned execution contract." But plan9.md requires these operations in Phase 1 (GR-3b, GR-8).

**The Ambiguity:** Are GR-local operations Phase-1 or post-Phase-1? If Phase-1, they need versioned contracts. If post-Phase-1, the GR workload can't complete.

**Required Resolution:**
plan9.md must explicitly state:
> "GR-local operations (differentiator, trig, Normalize, ZeroTest, weak-field reduction) are Level-1 framework-local operations under §15.4 of the PhysMath representation spec. They operate on GR-local expression types (`GRExpr`), not PhysMath expressions. They are governed by GR-local versioned contracts, not PhysMath Phase-1 rules. The GR-local rule namespace is `phys-gr` (ProducerNamespace = `phys-gr`)."

---

## BLOCKER 3: GR-8 Newtonian Limit Operation Undefined

**The Problem:**
plan9.md §9 says GR-8 includes "weak-field / Newtonian correspondence" using "GR-local weak-field reduction." But:
- v1.2's Phase-1 rules don't include a weak-field reduction rule
- plan9.md doesn't define what this operation does mechanically
- plan9.md says "GR-local weak-field reduction" but doesn't specify the mathematical procedure

A coding agent will ask: *"What does 'weak-field reduction' mean mechanically? What are the inputs, outputs, and transformation rules?"*

**Required Resolution:**
plan9.md must define the weak-field reduction operation mechanically:
> "GR-8 Newtonian limit procedure:
> 1. Start with Schwarzschild metric `g_tt = -(1 - 2μ/r)` where `μ = GM/c²`
> 2. Introduce `ε = GM/(c²r)` as expansion parameter
> 3. Truncate `g_tt` to first order in `ε`: `g_tt ≈ -(1 + 2Φ/c²)` where `Φ = -GM/r`
> 4. Verify `a_r = -GM/r²` from geodesic equation in weak-field limit
> This is a GR-local symbolic truncation, not a PhysMath `Limit` operation. It operates on GR-local `GRExpr` types."

---

## AMBIGUITY 1: GR Artifact Identity Mechanism

**The Problem:**
v1.2 defines `ArtifactRef = (Namespace, SchemaVersion, ArtifactType, Digest)` with canonical digest framing (§4.2-4.4). plan9.md mentions "GR artifact hashing" (§20 of plan9.md: "GR artifact hashes recorded") but doesn't define the identity mechanism.

A coding agent will ask: *"Do GR artifacts use v1.2's ArtifactRef? Do they have their own identity scheme? What is the canonical digest framing for GR artifacts?"*

**Required Resolution:**
plan9.md must state:
> "GR artifacts use their own canonical identity scheme under namespace `phys-gr`. GR artifact identity follows the same digest framing as v1.2 §4.3: `SHA-256(Len32(Namespace) || Namespace || Len32(SchemaVersion) || SchemaVersion || Len32(ArtifactType) || ArtifactType || CanonicalPayload)`. GR artifacts use `Namespace = phys-gr`, `SchemaVersion = 1`. GR artifact types include `grexpr`, `tensor`, `metric`, `connection`, `curvature`, `grstep`."

---

## AMBIGUITY 2: GR-8 Session Segment vs v1.2 Session Model

**The Problem:**
plan9.md mentions "GR-8 session recording segment" using `phys.Session` (frozen Phys sessions from specs_v2_3.md). But v1.2 doesn't define how GR interacts with Phys sessions. v1.2's session model is PhysMath-internal.

A coding agent will ask: *"Does GR-8 create a `phys.Session`? Does it record steps in a Phys ledger? Or does it use GR-local replay?"*

**Required Resolution:**
plan9.md must clarify:
> "GR-8 session segment uses frozen Phys `session.Session` for the final correspondence segment only (identifying `k2 = -2μ` as a physical constant correspondence). This creates a `KindRelation` / `IDENTIFIED` / `NONE` relation artifact in the Phys ledger. All other GR operations (GR-0 through GR-7) use GR-local replay (`GRStep` chain) and do not create Phys ledger entries."

---

## Summary of Required Resolutions

| # | Type | Issue | Resolution |
|---|---|---|---|
| B1 | Blocker | Scope relationship between plan9.md and v1.2 undefined | Explicitly state GR stress test uses frozen Phys, not PhysMath; PhysMath is Phase-2 |
| B2 | Blocker | GR-local operations vs v1.2 Phase-1 rules | Explicitly state GR-local ops are Level-1 under §15.4, namespace `phys-gr` |
| B3 | Blocker | GR-8 Newtonian limit operation undefined | Define mechanical procedure for weak-field truncation |
| A1 | Ambiguity | GR artifact identity mechanism undefined | Define GR artifact identity using v1.2 digest framing |
| A2 | Ambiguity | GR-8 session segment vs v1.2 session model | Clarify Phys session use is limited to final correspondence segment |

---

## What plan9.md Gets Right

1. **GR as hostile workload** — correctly treats GR as a stress test, not a kernel feature
2. **Separate module** — `phys-gr` as separate module depending one-way on frozen Phys
3. **GR-local structures** — tensors, trig, differentiator as Level-1 Go structures, not `core.Object`
4. **PASS0 pins** — correctly tests frozen Phys behavior before GR workload
5. **C1-C8 concern ledger** — correctly separates architectural concerns from growth findings
6. **Silent-wrongness testing** — correctly identifies the dangerous "accepted and wrong" outcome
7. **Growth Gate** — correctly requires evidence before kernel changes
8. **No kernel modification** — correctly preserves frozen Phys throughout

---

## Final Verdict

**3 blockers + 2 ambiguities remain.** All are resolvable with explicit textual clarifications in plan9.md. No architectural redesign is needed. The plan's core architecture is sound; it just needs explicit boundary definitions between the GR stress test and the PhysMath representation spec.

**After resolving B1-B3 and A1-A2, the plan is cleared for coding agent handoff.**