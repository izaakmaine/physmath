# Adversarial Review: `spec_v2.md` (Consolidated Review of PM-REP-001 v1.0)

## Verdict

**The consolidation is high-quality and correctly refuses to reopen the architecture — the accepted decisions align with the chain's settled mechanisms, and the rejections (S–V) are all justified scope defense. But three of its own decisions are internally incomplete or mutually contradictory, which under the document's own §32.2-style standard are blocker-class: Finding O is adopted without the schema field that makes it representable; Finding P's specialization rule contradicts Finding I's same-namespace rule; and the parameter model's capture-safety depends on a validation rule that is nowhere stated.** Below that, the closure condition (§13) is incomplete relative to the decisions it contains, and one fixture-feasibility hazard needs one sentence to prevent a guaranteed MATH-GAP dead-end.

---

## 1. Disposition of my prior v1.0 findings

| Prior finding | Consolidation disposition | Verdict |
|---|---|---|
| B-1 durable relation schema | E: payload = body sans tag, inline expressions, no refs — equality semantics now decidable (byte identity of structurally identical relations) | 🟢 Resolved |
| B-2 definition-application gap | M+G: `EXPR-APPLY-001` with exact inputs, arity check, simultaneous substitution, no recursive expansion | 🟢 Resolved (one validation gap, see 🔴-3) |
| B-3 registry + assumption triangle | A (master registry) + I (`Theorem.AssumptionRefs → physmath/condition/1`; phys-* gets `<ns>/assumption/1`) | 🟢 Resolved |
| Pin 1 field targets | I — mostly, but two fields remain vague (see 🟠-2) | 🟠 Partial |
| Pin 2 Tensor.Dimension | K: restored with validation | 🟢 |
| Pin 3 Map.DefinitionRef | I: → `physmath/definition/1` | 🟢 |
| Pin 4 EquationSet | J: SEQUENCE | 🟢 |
| Pin 5 BoundParameterRef encoding | D: exact, no wrapper | 🟢 |
| Pin 7 SIMPLIFY list | H: closed list incl. Pow(x,0)-with-condition | 🟢 |
| Pin 8 grammars | **Dropped** — not restated | 🔴-adjacent (🟠-1) |
| Pin 9 result shape | §10: `OperationResult{OutputRef, ConditionRefs SET}`, in-memory | 🟢 |
| Pin 10 InterpretationClass | Silently dropped, no Phase-1 ruling | 🟡 |
| Pin 11 framework rules | O: `(ProducerNamespace, RuleID)` — **schema not amended** | 🔴-1 |
| Pin 12 batch | DerivationRef mandatory ✓; Definition/Interpretation division, ConditionRefs targets, DomainSort values — open | 🟡 |

---

## 2. 🔴 Blocker-class findings in the consolidation's own decisions

### 🔴-1 — Finding O is adopted but unrepresentable: no `ProducerNamespace` field exists

O decides rule identity is the pair `(ProducerNamespace, RuleID)` and states: *"The durable `Transformation` and `DerivationStep` representations must carry the producer namespace."* But neither v1.0 schema has such a field (`Transformation{RuleID, Inputs, Output, Parameters}`; `DerivationStep{Ordinal, RuleID, InputRefs, OutputRef, ConditionRefs, Parameters}`), and the consolidation does not add one. A coding agent must invent the field name, position, and encoding — exactly the ambiguity class this chain exists to kill, and here it is created by an adopted decision. **Fix:** add `ProducerNamespace String` to both schemas (pin position — first field is the natural choice — and note that the five empty-parameter core rules serialize `physmath`). One paragraph.

### 🔴-2 — Finding P contradicts Finding I: cross-framework specialization is unrepresentable

P establishes the base/specialized model (`phys-bohm` may depend on `phys-qm`; interpretation-specific assumptions MUST NOT enter the base). I pins every assumption-reference field to `<same phys-namespace>/assumption/1`. Consequence: a specialized framework **cannot reference the base framework's assumptions** — it must duplicate them into its own namespace (losing the dependency semantics P just introduced), or cross-namespace assumption references must be permitted. Both are legitimate implementations; the document chose neither. Additionally, "depend on the base framework" has no defined mechanism (Go import? artifact reference? a `BaseFrameworkRef` field?). **Fix (pick one, pin):** (a) allow assumption/interpretation refs to base-framework namespaces, with the specialized package's `FrameworkRef` chain recording the dependency; or (b) require re-declaration in the specialized namespace with a provenance-of-inheritance note field. Also pin what "depends on" means mechanically. Without this, P is aspirational prose, not architecture.

### 🔴-3 — Function body validation is unpinned; capture-safety is implicit and an agent may "fix" a non-problem by inventing machinery

`EXPR-APPLY-001` substitutes into `f.BodyRef` "according to `f.ParameterIDs`" with matching by `ParameterID` (N). Nothing validates that the body's symbols are the function's own parameters. A body referencing a foreign `ParameterID` makes the rule's behavior undefined (leave the dangling symbol? reject?) — two legitimate implementations. More importantly: because ParameterIDs are ordinal-derived, *every* Function's *i*-th parameter shares the same value, and one might infer a variable-capture hazard when arguments are spliced into bodies — an agent aware of capture could implement alpha-freshening/renaming, which would break determinism and solve a problem the design has already eliminated. The design is in fact capture-safe: arity matching makes **every** body parameter a substitution target, so no body parameter symbol survives expansion; inserted arguments are spliced wholesale; and within any single Expression artifact, equal `ParameterID` = same parameter (positional discipline). **Fix:** pin (a) *Function artifact validation: every Symbol in `BodyRef` carries a `ParameterID` ∈ `Function.ParameterIDs` or a `ConstantID`; otherwise the Function artifact is invalid*; (b) *within one Expression artifact, `ParameterID` equality means the same parameter; `EXPR-APPLY-001` insertion is capture-free by construction — implementations MUST NOT introduce renaming or freshening.* Two sentences; they convert the trickiest semantics in the document from implicit to mechanical.

---

## 3. 🟠 The fixture graph is not yet "exactly one normative graph"

The closure condition requires it; the current decisions don't deliver it:

1. **g, x, X, Y have no artifact realization.** §16.3 lists them as "required represented objects," but parameters are not artifacts, and no sort represents a domain. L pins the Call structure but not what the `g`/`x` argument *Symbols* bind to. Given C's Function-only binder, the natural pin: *g and x are Symbols bound to the action functions' parameter ordinals (positional discipline — passing the i-th declaration-slot symbol as the i-th argument is identity under expansion), and X/Y are framework-owned definitions/PhysMath Collections.* Pin it, or two fixture authors produce different graphs.
2. **Fixture math must be declared tractable under the closed rules.** The v1 rewrite set has flatten+sort (commutativity/associativity ✓) and rational arithmetic — but **no distributivity** and explicitly no `x+x→2x`. LHS/RHS equality closes only if the concrete `f, ρ_X, ρ_Y` are chosen so (e.g., `f(v)=2·v`, `ρ_X(g,x)=g·x`, `ρ_Y=ρ_X` works). Without a sentence, a fixture author picks forms requiring distributivity, dead-ends at §24.4, and reopens the spec. Pin: *the reference fixture's concrete functions MUST close under `EXPR-CANON-001`/`EXPR-SIMPLIFY-001` alone; no additional rewrite rules may be assumed.*
3. **Relation/embedded-copy consistency:** the durable relation embeds canonical expression bytes while the derivation operates on standalone `expression/1` artifacts — pin that a standalone artifact representing a relation's side MUST be byte-identical to the embedded copy (guaranteed by canonicalization; one line, prevents "which is authoritative" questions).

---

## 4. 🟠 Remaining pins (most violate the document's own closure condition)

1. **Grammars must be restated.** Namespace/type/version grammars and the free-form identifier grammar (AssumptionID, FrameworkID, OperatorID, …) were settled in v1.5 and dropped in the v1.0→consolidation pass. Without them, digests are not computable from the document — failing §28's self-sufficiency. Restore verbatim.
2. **Two Finding-I cells are vague, violating "every ArtifactRef field has an exact target":** `Derivation.PremiseRefs` / `Theorem.PremiseRefs` ("explicit mathematical support artifacts or expressions") and `Derivation.ResultRefs` — pin exact family sets (e.g., `physmath/{expression,relation,equation,condition,definition}/1`). Same for `Collection.ElementRefs` ("admitted by the collection contract" — the contract doesn't exist; pin the admitted set).
3. **`PhysicalAssumption.ConditionRefs` is missing from Finding I's table** — pin (`physmath/condition/1`, SET).
4. **DIFFERENTIATE-001 asymmetry:** SUBSTITUTE requires the declaring artifact among inputs; DIFFERENTIATE embeds a `BoundParameterRef` in parameters with no input requirement. Pin: matching is by `ParameterID` within `Input[0]`; state whether the embedded `DeclaringArtifactRef` must be among inputs or is validated-informational.
5. **§6.2 vs Registry A:** deferred sorts "may be stored and validated" — but A gives them no durable family. Align: deferred sorts are in-memory-only in Phase 1 (no durable artifact family exists to store them into).
6. **Per-rule input families:** H gives inputs for four rules; SUBSTITUTE and DIFFERENTIATE input families unpinned. One line each.
7. **SELECT-BRANCH-001 generates no condition** — correct (a selection ordinal is not a math predicate), but pin the sentence so the GR chain's `selected_branch/*` precedent isn't imported, which would violate `physmath/condition/1`'s relation-only purity.
8. **§10 batch:** pin `DomainSort`/`CodomainSort` valid values (= §6.1 sort IDs); one sentence on PhysicalDefinition-vs-PhysicalInterpretation division of labor (both survive Finding Q; their overlap was never delimited); one-line Phase-1 ruling replacing InterpretationClass (*all Phase-1 physical artifacts are framework-scoped by construction via `FrameworkRef`; no interpretation-neutral claims exist in Phase 1*).

---

## 5. Rejections endorsed (verified against the chain)

- **S (frozen-Phys Kind/Dimension/assumption integration):** correctly rejected — the hypothesis-mint bridge is a projection-era concern; importing it into Phase-1 representation would re-couple the substrate to trust semantics exactly as plan8 removed them. `Representation ≠ trust admission` is the right wall.
- **T (TOE-Agent binary layouts), U (trust/provenance machinery):** correctly deferred; Phase-1 carries references only.
- **V (PhysicalContract):** correctly resolved by deletion rather than merging — Phase-1's five package-owned families plus package-local validation is the minimal honest surface.

The rejections are not evasions: each names the later specification that owns the concern, keeping the door open without blocking Phase 1.

---

## 6. Closure condition (§13) must be extended

As listed, §13's checklist would pass while the following remain open — extend it with: ProducerNamespace fields in Transformation/DerivationStep; base/specialized assumption-reference rule (P↔I reconciliation); Function body-validation + no-freshening rule; fixture parameter-declaration graph + tractability constraint; exact PremiseRefs/ResultRefs/Collection targets; PhysicalAssumption.ConditionRefs; restated grammars; DIFFERENTIATE input rule; deferred-sort storage alignment. A checklist that certifies a document containing its own unresolved decisions is the failure mode this chain has repeatedly caught — the §34-style audit will find them, so find them first.

---

## 7. Verified correct (no action)

The framing/struct-order merger (§4.2–4.3) is consistent and single-source; BigInt/ConstantID/enum decisions match the chain's settled encodings, with the tagged-union-vs-String-enum boundary correctly drawn (node tags binary, comparison/variance as String); the two-form Relation rule with inline bodies is the right compression of v1.5's ruling; the parameter model (C, D, N) is the settled local-identity design correctly operationalized for Phase-1's single-binder reality; the rule-semantics section (G+H) is the first in the chain where every transformation has closed parameters, closed rewrites, and fail-closed condition handling — including the excellent `Pow(x,0)` condition-generation rule; the equivariance structure (L) exercises exactly the machinery that matters and invents no sorts; the minimum-assumption/base-specialization principle (P, modulo 🔴-2) is the correct structural form of the hundred-physicists principle; and the rejection block correctly keeps Phase-1 clean of the entire Phase-2 surface.

---

## Bottom line

**Endorse the consolidation's strategy: amend once, mechanically, then implement.** The 15 accepted blockers are the right 15, the 4 rejections are the right 4, and no accepted decision reopens architecture. What remains is the consolidation completing *its own decisions*: add the ProducerNamespace field (🔴-1), reconcile P with I (🔴-2), pin body validation and the no-freshening rule (🔴-3), finish the fixture graph including the tractability constraint (§3), and sweep the pins — then extend §13 so the certification covers what the decisions actually created. Every fix is a paragraph; none reopens a single settled decision. With that amendment, PM-REP-001 becomes the first specification in this entire chain that satisfies its own handoff test — and the equivariance fixture becomes a demonstration rather than a discovery process.