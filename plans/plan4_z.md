# Adversarial Review: `plan4.md` (PM-ARCH-FOUNDATION-001 v1.1)

## Verdict

**The dependency-level architecture is now correct and genuinely coherent — PhysMath ⊥ Phys, adopted explicitly and well-justified, resolves the tension that generated three review cycles. The artifact/reference model (envelope removal, interpretation and trust as separate artifact classes, replay authority-local) is a real improvement, not a reshuffle. But the document inherits the exact pattern I flagged in the GR chain's v3: a completion checklist claiming "no architectural decision is left for the coding agent to invent" while three load-bearing joints remain decision-deferral language.** The blocker class has moved from *dependency direction* (solved) to *crossing mechanics* (open): the formalization protocol has no implementer and no provenance law, the projection subset is still undefined, and identity is "may be content-addressed, provided…" — a proviso whose requirements are never enumerated. Plus one regression: the strongest honesty statement in the chain (plan3 §29's curation deferral) was diluted, and a new implied Phys artifact (`TrustAssertion`) needs to be labeled design-target vs. existing mechanism.

---

## 1. Adoption audit vs. the structural review (D1–D7)

| Recommendation | v1.1 disposition | Verdict |
|---|---|---|
| D1 cut PhysMath→Phys | §0/§5/§42/§47, supersedes the prior rule explicitly, §5.1 enforcement rationale correct | ✅ Adopted — the central correction, correctly made |
| D2 Expression mirror/projection contract | **Absent.** §22/§23 pin sort behavior generally; nothing binds PhysMath's `Expression` sort to a `core.Expr` projection contract | ❌ Open — see B-2 |
| D3 phys-interop as shared kit | §35–36 present, non-semantic, correct exclusions — but **optional ("MAY")** and the bridge triple is not among its named capabilities | 🟠 Half — see B-5 |
| D4 replay per-authority | §28: "no fixed number of replay domains," per-trace declarations, digest-only cross-refs | ✅ Adopted — fixes the false "two domains" count |
| D5 trust ladder / named target | §11 `TrustAssertion` is a design target in spirit, but **unlabeled as future**; the explicit curation deferral (plan3 §29/ADR-007) is gone | 🟠 Regression risk — see B-4 |
| D6 meta-model chapter | Partial: §40 status vocabularies separated (good); identity/versioning/canonical/naming not consolidated | 🟠 Partial — see B-3 |
| D7 capability ledger | Not adopted | Acceptable omission; note for the lifecycle |

---

## 2. Blockers

### 🔴 B-1 — The formalization protocol has no implementer and no provenance law; `ACCEPTED` is unreachable as written

§31/§32/§39 route `FormalizationProposal` through "**Phys** review / validation," and §39 step 8 says "**Phys** validates/projectizes the physical subset." Frozen Phys contains no such function — validation of external proposals, projectization, and a revision-cycle protocol are all Phys growth, obtainable only through the Growth Gate that this document itself defers to (§18). Meanwhile the only mints frozen Phys offers external code are `hypothesis.NewCandidateConcept` (forced `HYPOTHESIS`/`NONE`) and session artifacts. Consequence: a coding agent implementing §31 either (a) waits for Phys capabilities that cannot exist, or (b) writes tests expecting `ACCEPTED` to yield corpus-trusted artifacts — which the contamination law makes impossible. This is the V4-2 wall with new plaster.

**Pin (one paragraph):** the protocol is *implemented by Project A / phys-interop tooling calling frozen Phys public APIs only*; `ACCEPTED` concretely means: the proposal's projectable fragment minted successfully as hypothesis-grade objects (forced `HYPOTHESIS`/`NONE`), losses fully accounted per §33, optional session recording of correspondence relations (expected provenance `HYPOTHESIS`, never `IDENTIFIED`, on hypothesis-grade operands); `REJECTED`/`REQUEST_REVISION` are tooling outcomes, not Phys corpus judgments. Phys's role is exactly what it already is: ops validation, MRC, session semantics. Nothing in Phys grows.

### 🔴 B-2 — The projection subset is still undefined, so §33's classification is undecidable

§1 lists "How do we hand a rich mathematical artifact into a much smaller physical IR?" as a symptom the redesign resolves. It is relocated (§31, §33), not resolved: `FULLY` vs `PARTIALLY_REPRESENTABLE` cannot be computed without defining what maps. §39's "projectizes" names a function that exists nowhere. Adopt the mirror contract at sort definition: **PhysMath's `Expression` sort is specified as a deliberate semantic mirror of the Phys quantity-expression subset (`Symbol/Rational/Add/Mul/Neg/Pow/Sqrt`) with a pinned exponent-domain mapping** — projection then becomes near-identity, classification becomes decidable against a documented subset, and the non-bridgeable sorts (Function, Operator, Tensor, Manifold, …) get a crisp, uniform `UNREPRESENTABLE` boundary instead of per-export negotiation. This was the single highest-leverage item in the structural review and it is the one conspicuously missing.

### 🔴 B-3 — Identity is deferred, not pinned (R-2 residual)

§6: "may be content-addressed, **provided the final implementation satisfies the deterministic identity requirements**" — which are never enumerated. An unenforceable proviso is deferral. §48's "Mathematical artifacts have stable identities" checkbox is untestable as written, and every cross-project reference in the document (`math_refs`, `subject_ref`, `source_ref`, trace digests) dangles on it. Pin: **one content-addressed scheme ecosystem-wide** — `namespace/sort/<hex SHA-256 of canonical bytes>`, references carrying `schema_version` + digest, digests as the only cross-project replay claim. One paragraph; kills the three-parallel-meta-models class (P4/P6 of the structural review) permanently.

### 🟠 B-4 — `TrustAssertion` implies a Phys artifact type that does not exist; the curation deferral regressed

§11 presents `TrustAssertion{subject_ref, provenance, corpus_status, …}` as a Phys-controlled artifact class. Frozen Phys expresses trust **only embedded in minted objects** (provenance/corpus fields); there is no standalone assertable artifact, and external code cannot mint one. Unlabeled, a coding agent will treat §11 as an existing mechanism. Additionally, plan3 §29's explicit containment — *"until a future Phys specification defines admission, new-domain objects remain external and are never automatically trusted; no extension may invent a curation mechanism"* — the strongest honesty statement in the chain — has been diluted to §10's "existence does not imply trusted corpus." Restore the deferral-with-containment verbatim, and label §11 as **design target**: interim trust evidence for external artifacts lives as non-authoritative attestation records (phys-interop sidecar), with the `TrustAssertion`/CurationRecord as the named future Growth Gate candidate — the trust *ladder*, not a phantom present-tense mechanism.

### 🟠 B-5 — phys-interop "MAY" defeats its own stated purpose

§35 justifies the kit as existing "to avoid five independent implementations of the same mechanical boundary rules" — then makes it optional. If optional, the first two extensions roll their own, and the drift class returns. Pin: the **boundary conformance suite is mandatory for every phys-* package** (import-boundary, no-duplicate-metadata, envelope/interpretation-classification, deterministic-identity checks); the shared kit is the sanctioned implementation vehicle but a package may satisfy conformance with equivalent local tests. Also add one clause to §36: codecs may *depend on* both canonical forms (that is translation, not "providing mathematics").

---

## 3. Secondary items (same pass; none architectural)

1. **Supersession bookkeeping:** plan3 v1.0 is marked "ARCHITECTURE CLOSED" with 34 ADRs; v1.1 overturns ADR-003 and replaces ADR-020 (`HandoffPackage` → protocol, §31) and the envelope (ADR-006). Two "closed" registers must not coexist — add an explicit supersession list naming which v1.0 ADRs are amended/retired, or the ambiguity class this chain exists to kill re-enters through documentation.
2. **Status-string residual:** `UNREPRESENTABLE` appears in both the §33 classification and the §32 formalization outcome. Typed fields + owning schema disambiguate; one acknowledgment line (the §40 discipline applied to its own edge case).
3. **Naming discipline absent:** the no-shadowing rule (kernel `Differentiate` vs local `Diff` precedent) should extend across all five authorities — one line in §46.
4. **§44 step 7 wording:** "Phys formalization integration" reads as Phys changes; retitle "formalization tooling integration (no Phys changes)."
5. **Version-compatibility rule:** artifacts carry `schema_version` but no consumption rule ("consumer MUST reject incompatible versions") — one line.
6. **Capability ledger (D7)** not adopted — acceptable; note it as a lifecycle option, not a gap.

---

## 4. Verified correct (no action)

The ⊥ decision's justification chain (§1 symptoms → §5 enforcement) is the right argument and now mechanically testable; §5.2 correctly relocates interop out of mathematical semantics; **envelope removal is a genuine improvement** — decoupling identity/payload/interpretation/trust into separate artifact classes solves the wavefunction multiplicity (§12–13) more cleanly than any envelope could; §7's reference-DAG with "arrows mean references, not inheritance" is the correct primitive; replay authority-locality with declared vocabularies (§28) supersedes the false two-domain count; the four status vocabularies (§40) each have one owner; Growth Gate routing intact with the TOE explicitly powerless (§18); the No-Go list now includes the old dependency itself (§42) — correct self-consistency; Numerical boundary (§34) unchanged and sound; implementation order (§44) establishes foundations before workload; the TOE is kept out of the libraries entirely, per instruction, with §37–38 correctly scoping it as consumer.

---

## Bottom line

**Adopt this revision — it fixes the relationship at the level where previous cycles kept failing (dependency, ownership, artifact classes) and does so with correctly chosen mechanisms.** What separates it from "no architectural decision left to invent" is the same finite pin set the GR chain closed in one pass: **B-1** (protocol implementer + provenance law of `ACCEPTED`), **B-2** (projection contract pinned at the `Expression` sort), **B-3** (one content-addressed identity scheme, enumerated), **B-4** (restore the curation deferral; label `TrustAssertion` as design target), **B-5** (mandatory conformance, optional kit) — plus the supersession list and the five one-liners. All are paragraphs with house precedent; none reopens the ⊥ decision, the artifact model, or the frozen Phys. Apply them and §48's checklist becomes a claim the document has earned — and the next artifact in this chain can, as promised, be an implementation plan rather than another architecture round.