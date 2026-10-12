# Adversarial Review: `plan.md` (PhysMath PM-ARCH-REQ-001 v0.4) vs. `background.md` + `TOE_gpt.md` — Architectural Blockers Only

## Verdict

**The three-layer architecture (PhysMath → Phys → Numerical) survives adversarial review intact — no blocker requires reopening the frozen Phys, the constitution, or the layer division itself. But the plan's own rules halt implementation today:** by §2 ("a coding-agent implementation is non-conforming if it requires the coding agent to invent an answer to an unresolved blocker") and §26 (no implementation authorization while a blocker is OPEN), the document — which is explicitly a blocker-closure instrument — leaves **seven architectural blockers open, three of them at the PhysMath↔Phys boundary where the entire trust story lives**. The boundary is defined *epistemically* (who owns what) but not *mechanically* (how anything crosses it). All seven are decision-pins of exactly the kind the GR chain executed in one revision pass (D19–D40); none is a design flaw.

**Scope caveat:** §26 is truncated at B-001 — the blocker register, the document's central instrument, cannot be checked for coverage. This review therefore supplies the consolidated register for merging into §26.

---

## What the plan gets right (verified, foundation for the blocker list)

The plan absorbs all four major TOE repairs at the architectural level: **t-flow ≠ k-flow** (§12 implements the reality-check's flow-separation correction directly); **no notation-inferred semantics** (§6/§7 — which also kills index-theorem-style inference at the representation layer); **Stage-agnosticism** (the plan commits to no spacetime ontology, consistent with CST≠CDT); **the missing coupling law representable as an open obligation** (§16/REQ-PM-CON-016 — the D[Ψ,Q,C]=0 gap is a first-class outcome, not an error to hide). The metadata monopoly (§19.7's anti-fragmentation table) is exactly right and correctly extends REQ-PM-CON-028; the wavefunction split (§4.2's "critical consequence") is the correct call; the responsibility matrix is nearly contradiction-free against the frozen Phys facts (external modules → public API only, no `internal/kernel`, Growth-Gate route for kernel needs, corpus status never computed). §19.11's routing tree is the GR Growth Gate generalized correctly.

---

## Architectural blockers

### 🔴 PB-1 — The bridge is mandated but has no mechanism, signature, or subset definition

REQ-PM-CON-012 requires "an explicit, auditable bridge"; §19.8 defines the handoff *package contents* and §19.9 the *classification* — but the bridge **API** is undefined: what functions, what the export artifact is, how IDs are derived, lazy vs. eager, and above all *which fragment of PhysMath's IR maps to `core.Expr` at all*. The frozen Phys makes this forced and narrow: `core.Expr` is a closed 10-node set (no functions, operators, tensors, manifolds, sets, binders); the **only** public raw mint for an external module is `hypothesis.NewCandidateConcept` (forced `HYPOTHESIS`/`NONE`); ops take `core.Object`. The GR chain spent three revisions pinning exactly this (D22: `ToCoreExpr`/`FromCoreExpr`/`ToCoreObject`, lazy, content-derived ID over pinned payload, mirror-subset filter, assumption-bridgeability rule) — plan.md must generalize that contract, including the honest expectation that the mirror subset is a *small* fraction of PhysMath's IR.

### 🔴 PB-2 — Extension physical objects cannot be trusted Phys objects; the metadata-population mechanism and handoff provenance law are undefined

REQ-PM-CON-027/028 promise "one trust-metadata model; extensions consume and populate it." The frozen Phys provides **no mechanism** for this: fixed constructors are closed (mechanics/relativity only), manifests are frozen to two frameworks, and an external library can mint only hypothesis-grade objects — permanently `HYPOTHESIS`/`NONE`, with no human-curation pathway for new domains (a phys-qm manifest would require extending frozen Phys; the Growth Gate route exists but is not a curation instrument). Either the plan accepts that **all external-domain physics is permanently hypothesis-grade until a defined curation instrument exists** (and names it), or it defines extension-owned curation records that Phys validators consume without Phys modification. It must pick — the choice is the trust story of the whole ecosystem.

Compounding it: **the plan never states the provenance law of handoff artifacts.** Every bridge export is `HYPOTHESIS` by construction; therefore any `Session.Identify` recorded at a TOE interface yields `HYPOTHESIS`, never `IDENTIFIED` (frozen §13.2 contamination law). This exact wall was hit and fixed in the GR chain (V4-2). §10's tier-4 equality ("physical identification") must be pinned as a **recorded correspondence proposal** in PhysMath — the minting of `IDENTIFIED` stays exclusively with `Session.Identify` (MRC-006), with expected outcome `HYPOTHESIS` on hypothesis-grade operands. Also carry forward the **two-replay-domains** decision explicitly (GR D11/D32): a PhysMath derivation trace can never replay inside a Phys session; the Phys ledger records only the bridge-projectable fragment + correspondence identifications, with the full trace hashed and referenced.

### 🔴 PB-3 — Non-quantity ontology has no home in the Phys object model

The frozen object model is `Kind` (18 closed values) + `Dimension` (7 SI bases) + `Expr` (10 nodes). A causal set, a Bohmian configuration `Q`, a wavefunctional `Ψ[Q]`, an operator algebra — none has a `Kind`/`Dimension` representation, and forcing them into dimensionless `Expression` objects is precisely the semantic violence REQ-PM-CON-018 and §4.2 forbid. Yet §18 requires physical contracts for exactly these objects (phys-cst, phys-bohm), and REQ-PM-CON-028 requires them to use the Phys metadata model. **The plan must define the extension-artifact shape**: extension-owned types that *reference* Phys metadata records (sidecar model — the GR precedent, where the tensor layer stayed theory-local with its own hashing and only scalar fragments bridged), vs. forced `core.Object` wrapping. This decision determines what §18's Stage/Dancer/Dance physical contracts even are.

### 🔴 PB-4 — Higher-order structure typing is a self-declared open blocker (§13)

§13 states verbatim: "A major architectural blocker is preventing everything from collapsing into a scalar expression tree… The implementation plan MUST therefore resolve how higher-order mathematical objects are typed and composed before coding begins." By the plan's own §2 rule, implementation is unauthorized while this is open. The Expression/Function/Operator/Transformation sort system, composition rules, and binder discipline are the PhysMath-internal architecture; §12's typed flows and §14's scopes both depend on it. Resolve (closed core sorts + composition contracts) or record deferral-with-containment in §26 — silently carrying it is not an option the document permits itself. (Related: §25's proposed `assumptions/relations` package name contradicts §11's own vocabulary and invites the REQ-PM-CON-025 violation — rename to `conditions/relations`.)

### 🟠 PB-5 — Open-vs-closed IR and the extension protocol are undecided, and §22 presupposes the answer

§8.2/§8.3 commit to "multiple related sorts" and "domain packages add semantics" — via what protocol? An unregistered open IR makes all nine of §22's canonicalization questions ("MUST answer before implementation") unanswerable: canonical form, hashing, and replay require a closed, versioned sort set with per-sort canonicalization obligations. The GR chain demonstrated the cost of getting this wrong (node-set closure + STOP conditions is what made canonicalization/replay pinnable). Pin: closed core sorts + sealed, versioned extension-sort registration with mandatory canonical-form contracts — or explicit deferral with containment.

### 🟠 PB-6 — The Numerical-Layer exclusion is unenforceable as drawn

The graph has `phys-* ──► PhysMath` (permitted) and `Numerical ──► Phys + phys-*`; therefore PhysMath runtime state is **transitively inside** the numerical layer's dependency graph, while REQ-PM-CON-014 bans consuming "raw PhysMath runtime state." Worse, §4.3 requires extension physical objects to be "tied to explicit PhysMath mathematical objects" — so phys-bohm artifacts *embed* PhysMath structure by design. The `-X->` arrow is thus a direct-import ban, not a semantic guarantee. Define the consumable surface: extension artifacts expose only formalized Phys-side projections (+ opaque math payloads), or the ban is re-scoped to direct imports with review-enforced semantic discipline. State which; it determines the phys-* artifact API (and interlocks with PB-3).

### 🟠 PB-7 — The TOE acceptance workload has no execution contract or bridgeability expectation

§18 names seven test dimensions but no phases, artifacts, golden states, or success criteria — against the established house style (GR-0…GR-8 tables). More importantly, honest expectation-setting is missing: the Dance sector (functional analysis, RG flows, ε-expansions) and most of the Stage sector are **UNREPRESENTABLE at the Phys bridge** (frozen `Expr` cannot hold them; approximation objects have no Phys node — the GR Newtonian limit stayed theory-local for exactly this reason). The acceptance test must define what "formalization handoff success" means for a predominantly UNREPRESENTABLE workload (scalar/quantity fragments cross; structural artifacts certified PhysMath-side with sidecar metadata; correspondence identifications recorded in Phys sessions), or the stress test can neither pass nor fail meaningfully.

---

## Secondary blockers / dependencies (register entries, one line each)

- **§22's nine canonicalization questions** are acknowledged-open and gate every hashing/replay promise; close them immediately after PB-5 (the GR F/D32/§26 pipeline is the template).
- **PhysMath ↔ theory-local algebra relationship** unstated: phys-gr's `GRExpr`-style algebras, PhysMath, and the L1→L2 promotion rules need one coherence sentence (subsume via promotion, migrate, or coexist).
- **Naming discipline**: D31's kernel-name-non-shadowing rule must extend to PhysMath vs. Phys (`Diff` vs `Differentiate`, `Subst` vs `Substitute`, PhysMath limit ops vs frozen `Limit`).
- **Constitutional amendment procedure** absent: REQ-PM-CON-010 requires supporting future domains "without changes to the constitution," but a LOCKED document with no amendment rule eventually forces silent violation; add the append-only versioning discipline the Phys chain already uses.

---

## Checked and NOT blockers (verified)

The exactness model (§9) does not conflict with Phys's series/asymptotics bans — those are Phys-internal non-goals, and PhysMath approximation objects are explicitly mathematical (§9.3) with export handled by §19.9's approximation-status field. The five-tier equality (§10) is architectural but resolves into PB-2's identification-authority pin. Dependency acyclicity holds (diamond, no cycle); §19.4's no-`internal/kernel` rule matches REQ-004-03; §19.11's routing preserves the Growth Gate; §20.3's result direction is consistent with REQ-013-04/05; §5's non-overlap rules and §19.7's metadata monopoly contain no exception that leaks. The dual `19.7` numbering and other editorial items are, per instruction, out of scope.

---

## Bottom line

**No architectural blockers to the architecture — seven architectural blockers to implementing it**, all concentrated in one place: the plan defines *who owns what* impeccably but not *how anything crosses the boundaries it draws* (PB-1/2/3), plus the PhysMath-internal sort system it correctly flags as its own deepest open question (PB-4) and the extension/IR protocol its canonicalization chapter silently presupposes (PB-5). Every one is a decision-pin with a proven precedent in the GR chain (bridge contract D22–D24, two replay domains D11/D32, artifact-shape sidecar model, provenance-law pinning V4-2, closed-node STOP discipline D19). Merge this register into §26, resolve the boundary cluster first — it is the trust story — and the plan meets its own authorization rule: an agent implementing PhysMath would face only mathematical difficulty, evidence, and human adjudication, never an architectural invention.