# Adversarial Review: `PM-REP-001 v1.0` (Phase-1 Representation Specification) — against `plan8.md` + chain

## Verdict

**The Phase-1/Phase-2 split is correctly executed, and this is the most implementable document in the entire chain — but it is not yet the self-contained contract §28 promises. Three blockers: the durable relation artifact's schema is undefined (two required consumers, and equality semantics hang on it), the rule registry cannot express the one derivation move the equivariance slice exists to demonstrate (application of definitions), and the artifact-family registry is missing entirely (unpinned family strings are invented identity).** Below that, ~12 one-line pins. None of this reopens the architecture — the identity layer, constitutional rules, and slice design are all correct.

---

## 1. plan8-execution audit

| plan8 intent | v1.0 disposition | Verdict |
|---|---|---|
| Separate top-priority spec from secondary systems | §0.3/§0.4 explicit deferral list; §23 boundary; §26 order excludes them | ✅ |
| Paper = research unit; formal payload = representation target; paper not ontology | §1.1 exactly | ✅ |
| ARI: Analysis/Interpretation outside, Representation inside | §1.2 | ✅ |
| Equation-first ("otherwise it's just prose") | §1.3, §11 | ✅ |
| PhysMath theory-neutral | §2.1, §7.3, §9.2, §22 | ✅ |
| "Hundred physicists, hundred answers" | §2.3, §14.7, §16.6, §25.13/25.14 | ✅ |
| Equivariance slice replaces TOE | §16 | ✅ |
| Provenance/trust de-centred | §0.4, §23.3, §22 no-go | ✅ |

The frozen-Phys handling (§0.1, §3), the identity/encoding layer (§4 — including the struct-order/framing-order merger that fixes v1.5's latent discrepancy), and the no-go list (§22) all correctly inherit the chain. §24's fixed/permitted decision lists and the §24.4 gap-classification stop rule are exactly the right handoff machinery.

---

## 2. 🔴 Blockers

### B-1 — The durable `physmath/relation/1` payload schema is undefined

`Equation.RelationRef` (§11.1) and `MathCondition.PredicateRef` (§11.4) both MUST target `physmath/relation/1` — the family is load-bearing twice — but only the Expression-level Relation **node** is defined (§7.10). The durable payload is never specified: node body without the outer tag (v1.5's ruling), or `{Operator, LeftRef, RightRef}` with refs? The choice is not cosmetic: §19.1 defines mathematical equality as *canonical payload bytes identical for the same family* — with inline bodies, structurally identical relations are byte-identical; with refs, equality is inherited from referenced artifacts. The agent must invent a schema plus an equality semantics. **Fix:** one paragraph — durable relation payload = `ComparisonTag || Expression(Left) || Expression(Right)` (node body sans tag), matching v1.5's ruling; the node form and the artifact form are two encodings of one concept (the §9.10-style two-form rule, compressed).

### B-2 — No rule encodes application of a definition — the equivariance derivation's central step is unrepresentable

The slice's derivation (§16.4) must get from the *definitions* of `f`, `ρ_X`, `ρ_Y` to the equivalence of `f(ρ_X(g,x))` and `ρ_Y(g,f(x))`. The registered rules (§11.5: canon, simplify, substitute, rat-normalize, differentiate, select-branch) contain **no function-application rule** — nothing rewrites `Call(f, args)` into the function's body with arguments substituted, and no rule consumes a `Function` artifact at all. Two legitimate fixture designs exist, one of which dead-ends: (a) represent the LHS as a `Call` node — then no rule can ever reduce it, and §25.12's "final step produces the theorem conclusion" is unreachable; (b) apply definitions by substitution against `BodyRef` — expressible today (`InputRefs = [body artifact]`, parameters = substitution entry binding the function's `ParameterID`s), but this pattern is pinned nowhere, and an agent choosing (a) hits §24.4's MATH GAP and reopens the spec. Since the slice is the document's acceptance core, this is a blocker under its own standard. **Fix (pick one, pin it):** add `EXPR-APPLY-001` (rewrite a root `Call(f, e₁…eₙ)` whose `FunctionRef` targets a `Function` with matching `ParameterIDs` by substituting the body — simultaneous, via the §17.4 mechanism; innermost-first for nested calls; parameters are identities, so no capture problem), **or** pin option (b) explicitly: *v1 derivations apply definitions by `EXPR-SUBSTITUTE-001` over the definition's body artifact; `Call` nodes are representational structure and are never rewritten by any v1 rule.* Either is one paragraph; state also that the substitution rule must verify the target `ParameterID` actually occurs as a bound symbol in the input expression (body symbols bind to the function's declared ordinals — the linkage by equal `ParameterID` value should be stated once).

### B-3 — No artifact-family registry; the `assumption` triangle is unresolved

§5.2: "Artifact types MUST be explicitly registered… No implementation may invent an unregistered durable artifact type" — then the document registers almost none. Pinned somewhere: `physmath/relation/1`, `/function/1`, `/expression/1`, `/condition/1`, `/derivation/1`, `/theorem/1`, `<ns>/framework/1`. **Unpinned:** `equation`, `equation_set`, `definition` (§11 defines the schemas, not the family strings), and all five phys-* kinds (`PhysicalAssumption/PhysicalDefinition/PhysicalEquation/PhysicalTheorem/PhysicalInterpretation`). Family strings are serialization identity — an agent inventing `"physmath/equations/1"` vs `"physmath/equation/1"` breaks cross-implementation digest compatibility, which §25.1 tests. Additionally, a triangle is left open: §1.3 lists **Assumption** as a Phase-1 representation unit; §5.2 lists `assumption` as an example type; but §11 defines no PhysMath assumption artifact, and `Theorem.AssumptionRefs` (§13.1) has no pinned target. **Fix:** one registry table (v1.5 §7.3's shape) listing every Phase-1 family — and pin the triangle: *mathematical assumptions are `physmath/condition/1` artifacts; there is no separate `physmath/assumption/1` in v1* (or define one — but decide).

---

## 3. 🟠 Pins the agent would otherwise decide (one line each)

1. **Theorem/Derivation field targets:** `Theorem.PremiseRefs`/`ConclusionRef`, `Derivation.PremiseRefs`/`ResultRefs` — pin families (conditions vs expressions vs relations). Same for `FunctionApplication.Args`/`OperatorApplication.Args` → `physmath/expression/1`.
2. **Tensor.Dimension was removed** (v1.5 had it) and replaced by "the owning tensor/index-space contract" — no such contract exists anywhere. Restore `Dimension uint16` or define the index-space family; without it, the `Dimension^Rank` count check (§10.4) is unimplementable.
3. **`Map.DefinitionRef`** — "an explicit mathematical definition artifact": pin `physmath/definition/1` (new reading) or `physmath/expression/1` (v1.5's pin).
4. **`EquationSet.EquationRefs`** — "SEQUENCE when…; SET only when…" is usage-dependent; a schema field has one semantics. Pin `SEQUENCE`.
5. **`Encode(BoundParameterRef)`** (§10.7) is an undefined composite — pin `EmbeddedArtifactRef(DeclaringArtifactRef) || Digest32(ParameterID)`.
6. **§3's chain formatting** (`phys-* → PhysMath → phys-artifact → Phys`) can be misread as `phys-artifact → Phys`. Replace with the flat edge list; phys-artifact imports nothing.
7. **`EXPR-SIMPLIFY-001`'s closed rewrite list is not restated** (v1.5 §15.5 had it; here only `Neg(Neg(x))` and the condition hook appear). Restating is required by §28's self-sufficiency — "non-conflicting inheritance" is not a mechanism an agent can execute.
8. **Grammars partially lost:** namespace/type/version grammars and the v1.5 free-form identifier grammar (AssumptionID, FrameworkID, OperatorID, …) are not restated; canonical digests depend on them. Five lines.
9. **Operation result shape:** §17.3 implies operations carry generated conditions, but no result type is defined (v1.5's `TransformResult` is gone with the trace redesign). Pin minimally: every transformation returns the output artifact plus the generated condition set.
10. **InterpretationClass is silently gone** (v1.5's closed two-value guard). Either restore the field or state the Phase-1 ruling: *all phys-* physical artifacts are framework-scoped by construction; no neutral claims exist in Phase 1.*
11. **Framework-owned transformation rules** (§2.2 claims phys-* owns them; §17.2's hook references them; §20.3 assumes `phys-gr` local RuleIDs) have no representation — no schema, no namespace rule, no registration path. Define the pattern (`<ns>/transformation/1`, namespaced RuleIDs) or explicitly defer with one sentence — currently §2.2 promises what the spec doesn't provide.
12. **Small:** `Theorem.DerivationRef` is a non-Optional field but §13.2 says "when the theorem claims an explicit derivational basis" — pin mandatory; `PhysicalDefinition` vs `PhysicalInterpretation` division of labor (one sentence); `PhysicalAssumption.ConditionRefs` closed target list; conventions (v1.5's `ConventionRequirement`, e.g. `metric.signature`) have no Phase-1 home — state the deferral or the representative-as-assumption pattern; `DomainSort`/`CodomainSort` valid values = the §6.1 sort IDs; Phys references in Phase-1 (`phys/object/1` inherited? out of scope until projection returns?) — one line.

---

## 4. Verified correct (no action)

The ArtifactRef layer is faithfully carried and internally consistent (declaration order now equals framing order — the v1.5 warning is obsolete by construction). Parameter identity (§8) is the settled local-identity design with the correct framing note; `BoundParameterRef` is unchanged. Symbol binding rules (§7.3), arity normalization (§7.5), Pow/Sqrt distinction (§7.7–7.8), the reserved `differentiate` operator with summed-Order canonical form (§10.7), the six-rule registry with `PROJECT-STRICT-001` correctly deferred (§11.5), derivation structural validity with the step-closure rule and no-hidden-inference (§12.4–12.5), Theorem ≠ truth (§13.2), multi-framework coexistence with the contamination and ambiguity tests (§14.7, §16.6, §25.13–25.15), theory-local rules (§15), equality as payload identity with the theorem-equivalence caveat (§19), per-family golden vectors (§20.4), conformance negative tests (§21.1), the no-go list (§22), and the §24/§28 handoff machinery — all correct, mutually consistent, and faithful to both plan8 and the chain. The `Operator`-sort-deferred/`OperatorApplication`-MVP split remains coherent via the reserved tag.

---

## Bottom line

**The strategic pivot (plan8's split, equation-first, equivariance slice, deferred secondary systems) is executed correctly, and the representation core is freeze-quality.** What stands between this document and its own §28 handoff test is finite and mechanical: pin the durable relation schema (B-1), close the definition-application gap (B-2 — the one finding that touches the acceptance slice's executability), emit the family registry and settle the assumption triangle (B-3), and sweep the twelve pins — every fix is a paragraph with precedent somewhere in the chain, and several are verbatim restorations of v1.5 text that the rewrite dropped. Do that, and this becomes the first specification in the sequence that a coding agent can implement start-to-finish without inventing a single identity, ownership, or representation rule — which is exactly what plan8 asked for, and exactly what the equivariance fixture will then be able to prove.