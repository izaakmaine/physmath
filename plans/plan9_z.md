# Adversarial Review: PM-REP-001 v1.4 (visible text) × plan9 — Re-verification

## Verdict

**The attached pair is unchanged from the prior round; this review re-grounds the prior dispositions against the spec's visible normative text (§0–§13.3) line by line. Result: GO confirmed.** All three blocker resolutions (A/A/A) are now grounded in *visible* spec text rather than chain inheritance; the visible text resolves seven of the prior round's pins; two genuinely new items emerge (one Phase-8 scoping rule, one fixture-determinism verification); the carried items are unchanged. **No new architectural blockers. The plan can proceed to Phase 0 once the architect ratifies B1-A/B2-A/B3-A — which the visible spec now supports directly.**

---

## 1. Blocker re-adjudication against the visible spec

| Blocker | Prior disposition | Visible-spec grounding | Verdict |
|---|---|---|---|
| **B1** node-set | Resolution A (framework-local trig/UF) | §7.1 closes the 10-node set ✓; §7.8 Pow exponent is an arbitrary Expression ✓; **§2.2 visibly authorizes the resolution**: "A `phys-*` library MAY define a local mathematical algebra… package-local… MUST NOT dynamically extend the PhysMath AST or durable PhysMath family registry" | 🟢 **A ratified** — grounding is now constitutional text, not the truncated §15 |
| **B2** regime placement | Resolution A (framework assumptions; step conditions only where rule contracts demand) | §12.5 requires explicit conditions for *generated* conditions only; §5.7 provides both carriers (`DerivationStep.ConditionRefs → physmath/condition/1`; framework assumption families) — the two-layer structure is visible and A matches it | 🟢 **A ratified** |
| **B3** protocol layout | Resolution A (nested layout, exact manifest schema/loading, documented equivalence) | §1.4 prescribes the nested shape *and* mandates "the exact schema and loading convention of the existing GR example" *and* permits "reused **or refactored**" — the three directives are jointly satisfiable only under A; the PROTOCOL GAP trigger is field-level ("without inventing a new semantic package-protocol field"), exactly how plan9 §6 scopes it | 🟢 **A ratified** — the tension is spec-internal, and A honors all three directives |

One corrigendum to B3's record: since §1.4 permits *refactoring* the GR package, resolution A does not even require the layout-equivalence note to be framed as a deviation — it is a sanctioned refactoring of shape with preserved semantics. Keep the note anyway (plan risk register already tracks the two-shape drift).

---

## 2. Prior findings resolved in the visible text

| Prior item | Visible resolution |
|---|---|
| DisplayName in golden vectors (prior P2) | §7.4: DisplayName is "presentation-only with respect to mathematical meaning, but it IS identity-bearing" — golden bytes depend on it; plan's display-pinning is mandatory, confirmed |
| §8.3 validation + framing (prior F3/P) | §8.3 now requires `Function.ParameterIDs[i] == DeriveParameterID(i)` and locks the framing ("distinct mathematical-binding digest domain… MUST NOT be harmonized") — resolved in text |
| Two validation regimes (prior pin) | §8.8 states them explicitly |
| Operator vocabulary (prior pin) | §10.7: executable vocabulary is exactly `{differentiate}`; non-absent `OperatorRef` invalid |
| ProducerNamespace (prior 🔴-1) | §10.8 + §12.2 — field present in both schemas, canonical identity |
| RAT-NORMALIZE family (prior P3) | §5.7: `Transformation.Output → physmath/expression/1` for **every** rule — RAT-NORMALIZE operates on expression artifacts containing a single Rational node; the phantom `rational` family is dead |
| EquationSet uniqueness (prior pin) | §11.2: deliberate uniqueness-on-SEQUENCE, never sorted |
| Standalone-expression encoding (prior pin) | §7.2: no wrapper; same encoding embedded and standalone |
| Conclusion byte-identity | §13.3 confirmed present (visible fragment) |
| plan9's §1 inventory duty | §0.6 **makes it a spec requirement** — plan9 §1/§4/§5 discharge it |

Also confirmed: §0.5 explicitly establishes the plan-first handoff and the right to report demonstrated blockers — plan9's B1–B3 reports are procedurally sanctioned, and §0.6's mapping categories (phys-artifact / PhysMath core / phys-gr-specific / protocol / legacy) are exactly plan9 §4's classification columns.

---

## 3. New findings

### 🟠 NF-1 — Phase 8's durable-reference scope is narrower than the plan states

§5.7 closes every framework `MathematicalRefs`/`EquationRef` target to the 16 PhysMath families. Consequences the plan's Phase 8 must absorb as one paragraph:

1. **No physmath `Function` may be created for `A(r)`, `B(r)`** — §8.6/§10.1 require a body; unknown functions have none. (Sharpens prior P8/F6 into a concrete prohibition.)
2. **Trig-bearing expressions have no §5.7-legal target** — `r²sin²θ` is not expressible; the ansatz equation cannot be durably referenced as a whole.
3. **Derivative atoms (`A'`, `B''`) are representable only as opaque schematic Symbols** — and `DIFFERENTIATE-001`'s closed rules would treat them as constants, so no PhysMath replay may ever re-derive them (GR computation stays in local algebra / `solution/`, untouched — consistent).
4. **The `sin≠0` regularity assumption has no expressible predicate**; convention-type assumptions (`-+++`, MTW) are not relation-expressible at all. Pin: convention/regularity assumptions publish empty `ConditionRefs` SETs with package-owned `Description` (identity-bearing per §14.2's pattern), and mathematical-predicate assumptions (`r > 0`, `Λ = 0`) carry expressible relations.

Pin: *Phase 8 publishes framework artifacts only for the PhysMath-expressible subset; GR-local mathematical content lives in package code and protocol tests, never in invented families.* Without it, an agent will either invent a family (violating §5.8) or force-fit unexpressible content.

### 🟠 NF-2 — Schematic `ParameterID` values are unpinned in visible text — and they determine the fixture's golden ArtifactRefs

§8.8 permits standalone ParameterID-bearing schematic symbols; §8.3's derivation is defined only for Function-declaration ordinals. The staged Calls (`Call(ρ_X, g, x)`) require durable `g`/`x` symbols (§7.4 durable rule), so their ParameterID *values* enter every step output's digest — and §25.34's cross-implementation determinism depends on them. **Verify §16.5 pins the schematic-value derivation** (the natural rule: schematic symbols carry the ordinal-derived value of the parameter slot they occupy). If §16.5 is silent, the implementation plan must pin it as a fixture rule before Phase 10 — plan9's golden-ref pinning (Phase 10(e)) makes *this* fixture deterministic, but the spec-level rule is what makes the *next* fixture reproducible.

### 🟢 NF-3 — `ρ_X` and `ρ_Y` are necessarily the same PhysMath artifact — document it as a designed consequence

I traced the slice: with `f(v)=2v`, `ρ(g,v)=g·v`, the pinned 6-step graph, and the closed rewrite set, `ρ_X` and `ρ_Y` have identical canonical payloads (§7.4 makes DisplayName identity-bearing, so divergent displays would produce distinct artifacts — but then §13.3's conclusion byte-identity gate fails, since `CANON` does not remove `Neg(Neg(…))` and no distributivity rule exists to reconcile divergent bodies). **The degeneracy is forced by the spec's own rules and the role distinction correctly lives at the framework level** — which is the architecture's math/meaning split demonstrated, not a defect. Pin one sentence so an implementer doesn't "fix" it by diverging display names and breaking the conclusion gate. (Also confirmed by trace: Steps 2 and 5 produce byte-identical outputs → same `ArtifactRef`, making §13.3's gate a direct ref comparison; the plan's `2*g*x` is display shorthand — canonical child order is byte-sorted, so golden vectors must record canonical order, not presentation order.)

---

## 4. Carried items (unchanged)

- **§12.4's "registered for the applicable runtime"** — the physmath-validator-vs-framework-delegation question for non-`physmath` derivation steps remains defined only in invisible §17.11/§24.5. Non-blocking for Phase 1 (all fixture steps are `physmath`; `solution/` runs on the local chain); carry the one-line pin.
- **Invisible sections cited as contract** (§14.8/§14.9 dependency closure, §16 fixture, §17 rule contracts, §25 suite): the spec remains truncated at §13.3. §0.5's plan-first handoff legitimizes plan9's citations, and the visible text corroborates the cited pattern everywhere the two overlap. Verify at the Phase-5/7/9 checkpoints.

---

## 5. Verified correct in the visible text (no action)

§0.5/§0.6 (closure basis + substrate mandate); §1.4's protocol rule with the field-level PROTOCOL GAP trigger; §4 identity/encoding layer (framing-order merger, embedded refs, BigInt, Float64Bits boundary); §5.1's syntactic-namespace/self-ownership model (resolves the earlier registry question for Phase 1); §5.6's 16-family registry with the six deferred sorts explicitly in-memory-only; §5.7's complete target map; §6.3's operation lock; §7's node set and per-node rules; §8's full parameter story (formula, ordinal-consistency validation, capture-safety, schematic boundary, t/k rule, framing lock); §10's higher-representation schemas with the fixture-uses-Calls ruling (§10.2) that plan9's forbidden-list mirrors; §10.9's Transformation/DerivationStep lifecycle split; §11's support artifacts with `EXPR-APPLY-001` registered and `Definition` tied to it (§11.3); §12's derivation validity, same-run availability, and cache prohibition; §13's mandatory `DerivationRef`.

---

## Bottom line

**GO.** The plan9 → PM-REP-001 v1.4 pair now satisfies the review target to the extent the visible text allows: the three blockers are ratified with visible-spec grounding (B1 via §2.2/§7.1, B2 via §12.5/§5.7, B3 via §1.4), seven prior pins are confirmed resolved in v1.4's text, and the two new items are one-paragraph pins (Phase-8 durable-reference scope; schematic ParameterID derivation), not decisions. The remaining unverifiable surface (§14.9, §16, §17, §25) is exactly the surface §0.5's plan-first handoff assigns to the implementation plan's cited contract — verify it at the phase checkpoints the plan already defines. Ratify the three resolutions, absorb NF-1/NF-2, record NF-3 as a designed property, and Phase 0 can start.