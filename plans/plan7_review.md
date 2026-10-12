# Adversarial Review: `plan7.md` (v1.3 authoring program — the closure plan for the closure plan)

## Verdict

**This is the strongest artifact in the chain and it is now authoring-ready: every prior blocker is resolved, every decision I flagged for pinning has been pinned — several more sharply than I prescribed (§4's name-not-resolver and §26's dual ownership are genuine improvements over my recommendations). The residual defect class has narrowed to what it should be at this stage: `ArtifactRef` still lacks a pinned owner, and three state-machine definitions are referenced but never enumerated. Both are authoring-time conditions, not design defects — and plan7 correctly relocates them into the v1.3 contract itself.** The review below discharges my remaining duty: enumerate exactly what must be in the authored v1.3 so that its own §33 meta-rule ("if any required row lacks a normative decision, v1.3 is not closed") can't be failed by an omission nobody notices until the §35 audit.

---

## 1. Disposition audit (all prior findings)

| Prior finding | plan7 resolution | Verdict |
|---|---|---|
| B-1 Phys imports phys-artifact | §2: Phys imports nothing new, reference-target-not-import-source, consumer-computed refs, byte-freeze explicit | 🟢 **Resolved — correctly, and stronger than my fix** (the conceptual distinction "reference without dependency participation" is the right principle to freeze) |
| B-2 DifferentialDomain in PhysMath | §11: enum removed entirely; parameter identity suffices; physical meaning maps externally; optional annotations are opaque metadata never consulted for equality | 🟢 **Resolved — the simplification is correct** and honestly better than my "open tag" version; §12's t→τ invariance law makes the negative test mechanical without any domain vocabulary |
| F-1 PARTIAL unreachable in MVP | §14: zero permitted-loss contracts, vocabulary exists but unreachable | 🟢 Resolved |
| F-2 table cells as rule refs | §9: exact cell vocabulary `N/A / RULE <id> / DEFERRED+CONTAINMENT`; "no redesign, convert v1.2 inventory" | 🟢 Resolved |
| F-3 pre-name mint path | §23: `hypothesis.NewCandidateConcept` pinned *with the confirmation condition* ("provided this is confirmed against the frozen specification") — the right epistemic posture for a document that cannot see the spec | 🟢 Resolved |
| F-4 extension vs dynamic-invention | §13: "closed per schema version, not closed forever" | 🟢 Resolved |
| F-5 one digest formula | §6: single normative framing, "delete any abstract/illustrative formula that differs" | 🟢 Resolved |
| F-6 PARTIALLY_PROJECTABLE vs PARTIALLY_REPRESENTABLE | resolved via §22's state/outcome split; §24 requires typed domains with distinct owners | 🟢 Resolved |
| F-7 ComputationalContract owner | §26: **Numerical owns the schema** (execution boundary artifact), phys-* owns production of valid instances | 🟢 **Resolved — this is the correct answer**, better than the options I offered (it's neither PhysMath metadata nor a phys-* artifact; it's the consumer's input specification, and ownership-by-boundary avoids a cross-module dependency for one schema) |
| F-8 discharge dangling | §18: MVP has NO condition-discharge operation, explicitly | 🟢 Resolved |
| Earlier: supersession entries for new artifacts | §45-style table carried in §37 structure (§22) + §35 self-containment requirement | 🟢 Instructed; verify at authoring |
| Earlier: research-status vocabulary regression | Not explicitly addressed in plan7 | 🟡 Carry into v1.3 (see W-1) |

plan7 also independently lands: `string` SchemaVersion (§5 — correct, Phys uses `"1"` and the GR chain proved the string form); embedded-ref binary encoding (§7 — the GR `StepEnvelope`-class lesson); the closed-cell audit requirement; the MVP/post-MVP scope split (§29 — the nine-repository ambiguity was real); Growth Gate post-MVP (§30); evidence-not-permission for GR (§30, matching the GR chain's own language).

---

## 2. Authoring conditions — what v1.3 must contain for §33's meta-rule to be satisfiable

**W-1 — Fix the four-field sort-table row set before freezing the table.** v1.2's inventory (17 sorts) vs plan7's §9 column list is consistent, but three cross-cutting structures from prior rounds need explicit row treatment or the table self-violates: `Symbol` (§10 resolves as node kind — put it in the Expression row, not its own row); `MathCondition` (not an Expression node — it's the §18 `TransformResult` carrier; give it its own meta-structure contract under §32); `DerivationObligation` (present in v1.2, absent from plan7 — MVP or DEFERRED? decide in the table, don't let it vanish). And §37's structure has no home for the **per-sort MVP/DEFERRED/FORBIDDEN classification table itself** — add it under §8 explicitly.

**W-2 — The three referenced-but-unenumerated state machines must be fully enumerated in v1.3.** §22/§24 commit to typed state machines with entry conditions/transitions/terminals/payloads — but only the formalization one is sketched. The authored v1.3 must contain all of them completely: (a) **Formalization** (DRAFT→SUBMITTED→RESOLVED; five outcomes; revision lineage via `ParentProposalRef`; terminal states explicit); (b) **Research hypothesis** (§35.2's eight values with legal transitions — e.g., can `COUNTEREXAMPLE_FOUND` go back to `PROPOSED`? pin it); (c) **Mathematical operation outcome** (§35.1's five values — is `UNDEFINED` terminal?). If a machine has only one entry path or all-terminal states, say so — "no transitions defined" is a decision, absence is not.

**W-3 — Artifact contract template (§32) needs its five completion rules pinned in v1.3.** The template is the right instrument; four of its thirteen fields have known sharp edges: **RESOLUTION** = "name only; resolution is environment infrastructure; a contract that requires resolution at semantic-layer code time is non-conforming"; **LEGAL STATES/TRANSITIONS** = reference the §21/§22 machines by ID, don't restate; **IDENTITY** = which of the four `ArtifactRef` fields vary within the family (fixed namespace? fixed type?); **POSITIVE/NEGATIVE TEST** = "at least one each, executable, with the §25 Given/When/Then form." Pin these once in §32 so all thirteen artifacts inherit them.

**W-4 — The namespace registry table must be enumerated, not exemplified.** §5/§6 pin authority-owned namespaces; v1.3 needs the actual table with each namespace's owning authority and its artifact-type vocabulary — at minimum `physmath.*`, `phys.*`, one entry per planned phys-* (or the rule for allocating new ones: "new namespace = new phys-* package, declared in its contract"), `toe.*`. Also pin the **Phys namespace contents**: which frozen artifact types are referenceable (`core.Object` canonical JSON? session artifacts? manifests?) — each needs its canonical source named (§10.4 object order, §16.12 step order, §26.2 candidate order), since consumer-computed refs (§2) depend on it.

**W-5 — One coherence check the four reviews haven't run: §26 vs §21.** If Numerical owns the ComputationalContract *schema*, and §21 says no project replicates Phys trust metadata — fine, no collision. But ComputationalContract contains `PhysicalModelRef`/`MathematicalModelRef` — **these must be `ArtifactRef`s** (ecosystem identity, §6's framing) — and the in-process form (§27) materializes them. Say so explicitly in the ComputationalContract contract row, or the one-identity rule develops its first exception.

**W-6 — The audit protocol needs its registry fixed in v1.3.** §34's blocker-evidence rule is right; add: findings register with per-finding classification + the evidence that justified it; and the reviewer set is the four standing agents **plus the project owner as final arbiter** (the chain's REQ-000-03 human-authority principle, inherited from Phys §0.2). The §31 stopping rule then becomes fully mechanical: 4×0 BLOCKER + §33 matrix complete + W-1..W-5 discharged.

**W-7 — Smaller authoring pins:** (a) §2's "consumers calculate references to frozen Phys artifacts from the frozen canonical representation" — name the computation once: `Digest = SHA-256(Len32 framing over the frozen canonical bytes exactly as the frozen spec defines them)`, so consumers don't re-derive framing for Phys artifacts; (b) §7's "never JSON/hex/digest-bytes" for embedded refs — add the positive statement of what a *transport* document may use (transport JSON may show refs in a documented lossless display form; canonical payload never does); (c) §28 — Float64Bits belongs only in ComputationalContract payloads; one sentence barring it from every other artifact family; (d) §16's reason codes — pin the reason vocabulary (`NON_RATIONAL_EXPONENT_IN_POW`, plus the closed list per node kind from the projection contract) so loss reports are typed; (e) §23's "RequestedTarget must be a typed target reference" — enumerate the MVP targets (only `phys.hypothesis_object`, presumably — one value, pinned); (f) §5's namespace format rule (ASCII? charset? length caps?) — one line, since digests depend on it.

---

## 3. One process risk worth naming (not a blocker)

§35 requires the audit to run on "v1.3 + frozen Phys specification" alone. That is correct for closure — but the audit reviewers will re-derive context the chain has already paid for (GR D22–D24, V4-2, the N-/R- lineage). Keep the **decision registers of prior documents archived and referenced as non-normative lineage** in v1.3's supersession section, so the audit can *check* provenance of rules without *re-deciding* them. The distinction plan7 already draws (§24: "new architecture vs constraint inherited from frozen authority") should extend to lineage: inherited-from-prior-round vs inherited-from-frozen-Phys vs new-in-v1.3 — three tags in the supersession table.

---

## 4. What plan7 gets right that deserves explicit confirmation

- **§4's name-not-resolver** is the conceptual unlock the whole ArtifactRef thread needed — it dissolves the Gemini deadlock without a registry and without semantic authority.
- **§26's dual ownership** (schema by consumer, instances by producer) is the correct resolution of the last genuine ownership question in the chain.
- **§29/§30's scope split** fixes a real ambiguity that has silently infected five rounds of review (reviewers auditing ecosystem documents against MVP constraints and vice versa).
- **§34's evidence-based arbitration** with the (1)–(4) demonstration rule and "no majority vote" is the right termination mechanism — it makes the review falsifiable rather than diplomatic.
- **§36's promoted prohibition** ("physical meaning never inferred from mathematical structure" on the No-Go list) is the correct constitutionalization of the deepest lesson in the chain.
- **The closing judgment is correct**: "the architecture itself is not collapsing; the closure mechanism was one level too abstract." The chain's endgame is now: author v1.3 per §37's structure with W-1..W-7 discharged → run §35's audit → freeze → write the deliberately boring implementation plan.

---

## Bottom line

**plan7 is approved as the authoring program for v1.3 — no architectural blockers remain in it.** Its two critical corrections (Phys byte-freeze; DifferentialDomain removal) are exactly right, its resolutions of every F-/R-/B- item are verified, and its two genuine improvements (name-not-resolver, contract-schema dual ownership) are adoptable as-is. The document that gets frozen must be checked against **W-1 through W-7** — the enumerated sort table with all three meta-structures placed, three complete state machines, the template's completion rules, the namespace/registry table, the §26/§21 coherence line, the audit registry, and the half-dozen one-line pins — because §33's meta-rule is only as strong as the completeness of the row list it governs. Author v1.3, run the blocker-only audit, freeze on 4×0, and then — finally — the implementation plan can be as boring as this chain has spent seven documents earning the right to make it.