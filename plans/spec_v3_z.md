# Adversarial Review: `PM-REP-001 v1.1` (Phase-1 Representation Specification, "Closed for Phase-1 Implementation")

## Verdict

**This is the document the chain has been driving toward — every blocker from the consolidation is resolved, the two 🔴 schema gaps are fixed exactly as prescribed, the capture-safety and body-validation rules are pinned, the equivariance graph is executable, and the §0.5 closure basis plus the §1.4 GR-protocol requirement turn the last "future work" handwaves into contract. What remains is not architectural: it is the residue of an incomplete paste (the document truncates inside §14.6), plus a small set of pins concentrated in the areas the rewrite *added* — §17, §24, and the fixture data.** Under the document's own standard ("no ambiguity for the coding agent"), the truncation itself is a blocker-class delivery defect: §17 (executable protocol + fixtures) and §24 (implementation contract) are the two sections a coding agent will live in, and they are not visible. Everything visible that was reviewable is reviewed below.

---

## 1. Disposition of prior findings (v1.0 → consolidation → v1.1)

| Prior | v1.1 resolution | Verdict |
|---|---|---|
| B-1 durable relation payload | §7.11/§7.2: node body sans tag; same encoding embedded and standalone (§7.2's "same canonical expression encoding" also resolves my byte-identity pin) | 🟢 |
| B-2 definition application | §11.5 `EXPR-APPLY-001` registered; §8.6/§8.7 body validation + no-freshening; §11.3 ties Definitions to it | 🟢 |
| B-3 registry + assumption triangle | §5.6 complete registry (16 PhysMath + 6 per phys-*); assumptions = `physmath/condition/1`, no `physmath/assumption/1` | 🟢 |
| 🔴-1 ProducerNamespace unrepresentable | §10.8/§12.2 field added to both schemas, position pinned, canonical identity declared | 🟢 |
| 🔴-2 P↔I cross-framework contradiction | §5.7 "dependency closure" + §14.8/§14.9 mechanism (truncated — verify); §14.1 `BaseFrameworkRefs` | 🟠 Verify remainder |
| 🔴-3 body validation + capture | §8.6/§8.7 exactly as prescribed, plus the positional-slot clarification | 🟢 |
| Fixture graph unification | §10.2 explicit ruling (Call not durable FunctionApplication); §13.3 conclusion-bytes rule; §12.6 same-run availability | 🟢 (tractability constraint — verify §17) |
| Grammars restated | §5.1–5.5 complete, incl. FrameworkVersion-as-opaque-string + fixture pin | 🟢 |
| TransformResult | Single result type — §15 region, truncated; §14.5 references it | 🟠 Verify |
| InterpretationClass | Dropped; §14.3's definition/interpretation division of labor added | 🟢 Acceptable — verify no residual in remainder |
| Tensor.Dimension | §10.4 restored with validation | 🟢 |
| EquationSet SEQUENCE | §11.2 | 🟢 |
| BoundParameterRef encoding | §4.10 | 🟢 |
| SIMPLIFY rewrites, rule semantics, differentiate rules | §11.5/§17 region — **truncated** | 🟠 Verify |
| DomainSort values, ConditionRefs targets, scope ruling, misc. batch | §10.3, §5.7, §14.x — all landed | 🟢 |

New and correct: §1.4/§1.5 (formalization = protocol execution; prose-only status — the strongest anti-drift instrument in the chain, directly imported from the GR precedent), §2.5 (complexity heuristics as permanent architecture), §5.7 (the complete target-family table — the single most valuable addition), §6.3's operation lock, §10.9's Transformation-vs-DerivationStep lifecycle split, §12.6's cache-discovery prohibition, §13.3's byte-identity conclusion rule, §14.2's Description-is-identity-bearing pin.

---

## 2. 🔴 Delivery blocker: the document is truncated and the closed sections are the load-bearing ones

The paste ends mid-§14.6 (`PhysicalInterpretation` targets, first list). Missing entirely: **§14.8/§14.9** (the framework dependency mechanism — load-bearing for §5.7's "dependency closure" and §3's dependency rule), **§15** (theory-local algebras), **§16** (the equivariance reference slice — the document's acceptance core), **§17** (the executable protocol, rule semantics, fixtures — including the v1.0 §17.1–17.4 content the closure promised to complete), **§18–§21** (canonicalization/equality/replay/golden vectors/conformance), **§22** (no-go list), **§23** (secondary boundary), **§24** (implementation contract — fixed vs. permitted decisions, stop rule), **§25** (acceptance suite A–S), **§26** (implementation order), **§27–28** (invariants, handoff rule).

Two of these are blocker-class by the document's own standard:

1. **§14.8/§14.9 undefined** → §5.7's "dependency closure" and §3's "only through §14.8/§14.9" are **dangling references inside normative target rules**. A coding agent validating `PhysicalEquation.AssumptionRefs` against an assumption in the base framework cannot determine validity. Pin at minimum: dependency closure = reflexive closure of `BaseFrameworkRefs` over published framework artifacts; acyclicity enforced; base assumption reuse permitted, base *modification* never (an assumption artifact is immutable and framework-owned; a specialized framework references it, never redefines it).
2. **§17 incomplete** → the seven rule contracts (input families per rule, EXPR-SIMPLIFY-001's closed rewrite list, EXPR-APPLY-001's exact inputs and arity rule, DIFFERENTIATE-001's closed derivative rules, the tractability constraint on the fixture's concrete functions) are the document's executable core. Without them, §11.5's registry is a list of names.

**Requirement for the audit:** supply the complete document; verify the remainder against the checklist in §4 below.

---

## 3. Findings in the visible text

### 🟠 F-1 — The rule-parameter schema for `EXPR-APPLY-001` is underdetermined by its own §10.8 rule

§10.8: "`Parameters` are governed by the exact rule contract in §17" and "For Phase-1 PhysMath transformations… Output family = physmath/expression/1." Two visible inconsistencies: (a) §10.8 says the output family is `physmath/expression/1` **for every Phase-1 executable rule** — but `RAT-NORMALIZE-001`'s input/output is a Rational *artifact* (v1.0 §15.5) and `SELECT-BRANCH-001` selects a branch expression (fine). If RAT-NORMALIZE operates on a standalone `physmath/expression/1` containing a single Rational node, all is consistent — but that's an inference, not a pin. State it: *RAT-NORMALIZE-001's input and output are `physmath/expression/1` artifacts whose payload is a single canonical Rational node.* (b) Whether `Transformation.Parameters` is empty (EXPR-CANON/SIMPLIFY/APPLY) or carries entries (SUBSTITUTE — sorted-entry encoding; DIFFERENTIATE — `Encode(BoundParameterRef)‖UInt32BE(Order)`; SELECT-BRANCH — `UInt32BE(ordinal)`) must be enumerated per rule in §17 — visible text cannot confirm.

### 🟠 F-2 — §12.4's "producer/rule pairs are registered for the applicable runtime" needs the Phase-1 disambiguation

With `(ProducerNamespace, RuleID)` identity and framework-local rules permitted (§2.2), a Derivation may legitimately contain non-`physmath` steps. Pin: *Phase-1 PhysMath validation accepts only `physmath`-producer steps; framework-local steps are legal in a Derivation's data model and are validated by the owning framework (§20.3's authority-local replay) — the PhysMath structural validator MUST reject an unregistered producer namespace rather than guess.* Without this, "registered for the applicable runtime" is two readings (reject vs. delegate).

### 🟠 F-3 — §13.3's byte-identity rule needs its scope pinned

"The relation's embedded left/right expression bytes MUST equal the canonical payload bytes of the corresponding final derivation result expression artifacts used as conclusion sides." Pin two things: (a) the *correspondence rule* — which result is "the" left side (e.g., derivation order / the step whose output is first referenced by the conclusion construction); (b) that this is a **construction-time validation duty** (the tooling that assembles Theorem from Derivation checks it), not a global invariant the validator re-derives. As written, an agent could implement it as a cross-artifact scan.

### 🟠 F-4 — §8.3's hash-input framing differs from §4.3's — verify intent, then pin

§4.3's framing: `Len32(N)‖N‖Len32(V)‖V‖Len32(T)‖T‖P` (three length-prefixed strings). §8.3's ParameterID input: `String("physmath")‖Bytes(ScopePathBytes)‖UInt32BE(ordinal)` = `Len32‖bytes‖Len32‖bytes‖4 bytes`. These are different framings — fine (different digest domains), but the document must say ParameterID is **a distinct digest domain, not an ArtifactRef digest and not reusable as one**, and that the double length-prefixing (`String` around the namespace, `Bytes` around the scope path) is intentional. One paragraph prevents a future "harmonize the framings" refactor that would invalidate every persisted ParameterID.

### 🟡 Small pins (one line each)

1. **§7.4 durable-symbol binding vs. schematic standalone symbols (§8.8):** state explicitly that the §7.4 validation ("exactly one of…") applies to *Function bodies and operation-referenced contexts*, while standalone schematic expressions are validated by §8.8's rule — two validation regimes, deliberately.
2. **§10.7 `OperatorID` grammar:** free-form identifier grammar (§5.4) applies; Phase-1 vocabulary is exactly `{differentiate}`; a non-`differentiate` ID with absent `OperatorRef` is invalid — one sentence.
3. **§14.2 `Category`:** open string confirmed identity-bearing (it's in the canonical payload) — consistent with §14.2's Description ruling; no action, but the §5.4 identifier-grammar applicability to `Category`/`Description` should be stated (they're prose — likely exempt; say so).
4. **§11.2 duplicate-reference invalidity:** "Duplicate references are invalid in Phase 1" — good; confirm this is a SET-style uniqueness check on a SEQUENCE-typed field (deliberate hybrid) with one clause.
5. **Phys references in Phase-1:** §5.7 contains no `phys/*` targets and §5.6 registers none — consistent with projection deferral (§11.5 `PROJECT-STRICT-001` outside Phase 1). State the one-liner: *no Phase-1 artifact references namespace `phys`; the frozen Phys boundary activates only in a future projection specification* — prevents an agent from "helpfully" adding a Phys back-reference.
6. **§0.5 "closed decisions" vs. §24:** the closure-basis list is authoritative; ensure §24's permitted-decisions list is consistent (visible text can't confirm).

---

## 4. Completion checklist for the truncated remainder (audit requirements)

1. **§14.8/§14.9** — framework dependency mechanism: closure definition, acyclicity, base-reuse/base-modification rule, validation algorithm for the §5.7 "dependency closure" targets.
2. **§15** — theory-local algebras (locality, explicit deterministic mapping, no AST extension, no forced migration).
3. **§16** — the equivariance slice: required artifact list, derivation shape, framework fixture minimums, ambiguity test — updated for v1.1's pinned rulings (Call-not-FunctionApplication, conclusion byte-identity, concrete function choices satisfying F-1's tractability pin).
4. **§17** — the executable protocol: per-rule input families (SUBSTITUTE: Function + expression inputs, declaring artifact present; DIFFERENTIATE: single expression input + embedded BoundParameterRef target); per-rule `Parameters` encoding; SIMPLIFY's closed rewrite list (Neg(Neg), identity elements, zero-product, rational arithmetic, Pow(x,0)-with-explicit-condition, Pow(x,1)); APPLY's arity check and non-recursion clause; **the tractability constraint** (*fixture functions MUST close under CANON+SIMPLIFY alone*); TransformResult definition; condition-generation rules.
5. **§18–§20** — canonicalization determinism, equality-as-payload, SET/SEQUENCE table confirmation, replay authority-locality, golden-vector requirements (per-family + fixture), conformance negative tests (the eight from v1.0 §21.1 + framework-disagreement test).
6. **§22–§28** — no-go list (including the §1.2 physical-inference prohibition and §2.5 heuristics), secondary-system boundary, implementation contract (fixed vs. permitted — aligned with §0.5), acceptance suite A–S, implementation order (with phys-artifact first), invariants, handoff rule.
7. **§5.7 hygiene:** confirm `PhysicalDefinition.MathematicalRefs` and `PhysicalInterpretation.MathematicalRefs` list *all 16* families (v1.0's finding; the visible §14.3 list shows 16 — verify §14.6's truncated list matches), and that `EquationSet.EquationRefs` appears (visible ✓).

---

## 5. Verified correct (no action)

The identity/encoding layer is faithfully carried with the framing-order merger (§4.2–4.4, §4.8) and the settled BigInt (§4.6); §5.6's registry is complete and the deferred-sort in-memory-only ruling closes the v1.0 storage question correctly; §6.3's operation lock kills the "vocabulary implies capability" inference; §7.2's no-wrapper standalone-expression rule with the same-encoding guarantee; §8's complete parameter story (local slot identity, explicit §8.3 formula, capture-safety, schematic standalone symbols, t/k rule); §9's closed ConstantIDs; §10's per-family schemas with the Function-body validation tie-in; §10.8/§10.9's ProducerNamespace + lifecycle split; §11's support artifacts with EXPR-APPLY as the sole definitional-application path; §12's derivation validity with same-run availability and cache prohibition; §13's mandatory DerivationRef and byte-identity conclusion; §14's framework contract with BaseFrameworkRefs, identity-bearing Description, and the definition/interpretation division — all correct, mutually consistent, and faithful to every settled chain decision. The GR-protocol formalization rule (§1.4) is the right inheritance: it imports the chain's hardest-won lesson (executable derivation tests, not prose) as a constitutional requirement for all future formalization packages.

---

## Bottom line

**The architecture and representation contract are closed; the document is not yet deliverable.** One delivery blocker (truncation hiding §14.8/§14.9 and §17 — both load-bearing), four pins in the visible text (F-1 through F-4, each one paragraph), six one-line hygiene items, and a seven-item checklist for the remainder. None of it reopens a settled decision — the fixes are restorations of v1.0 content the rewrite dropped, completions of the consolidation's own adopted decisions, and explicitations of rules the visible text already implies. Complete the document, apply the pins, and run the blocker-only audit: for the first time in this chain, the audit's job will be confirmation rather than discovery — which is exactly what "CLOSED FOR PHASE-1 IMPLEMENTATION" must mean before a coding agent is handed the document.