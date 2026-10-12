# Adversarial Review: `plan5.md` (PM-ARCH-FOUNDATION-001 v1.2 — "Architecture Closed")

## Verdict

**All five prior blockers are genuinely resolved — not restated — and several resolutions are better than what I recommended. The architecture is closable. One residual sentence separates this document from fully earning its footer (`Architectural blockers: NONE`): the provenance law of the protocol's session-facing outputs, the exact mistake the GR chain already made and fixed once.** Everything else is boundary-condition bookkeeping the implementation plans must carry. No new structural defects found; the 40-ADR register survives cross-checking without contradiction.

---

## 1. Prior-blocker disposition (verified line-by-line)

| Prior | v1.2 resolution | Verdict |
|---|---|---|
| **B-1** protocol implementer + provenance law | §26: implemented by **Project A integration tooling** using frozen Phys public APIs only, "No Phys kernel modification is implied"; §27: no runtime `AcceptFormalization`, success ≠ `ESTABLISHED`; §28: `PROJECTED_AS_HYPOTHESIS` = "existing hypothesis-grade constraints" | 🟢 **Resolved except one sentence** — see R-1 below |
| **B-2** projection subset undecidable | §23: mirror subset pinned exactly (`Symbol/Rational/Add/Mul/Neg/Pow/Sqrt`, exponent-domain matching frozen `Pow` **exactly**); richer sorts enumerated as non-projectable; §24 classification from the contract, never inferred, never widened; §25 loss report; §44 projection tests | 🟢 **Fully resolved** — the mirror-as-sort-obligation framing is exactly right, and "matching the frozen Pow contract exactly" is the correct way to pin the exponent domain without a dependency |
| **B-3** identity deferred | §5: one `ArtifactRef` scheme, digest = SHA-256 over `Namespace ‖ SchemaVersion ‖ ArtifactType ‖ canonical Payload`, eight enumerated rules, version participates in identity, incompatible versions rejected (§36) | 🟢 **Fully resolved** — the unenforceable proviso is gone; identity ≠ acceptance is now coherent (§5 rule 8 + §36 rule 2) |
| **B-4** phantom TrustAssertion + curation regression | §20: the strong containment invariant restored verbatim in force ("MUST NOT invent a curation system… future Phys specification/Growth Gate concern"); §21: `TrustAssertion = FUTURE DESIGN TARGET`, "No coding agent may implement a new `AcceptTrust` or `AcceptFormalization` path in Phys"; ADR-024/025 | 🟢 **Fully resolved** — §21's explicit prohibition is exactly the wall the GR chain needed |
| **B-5** optional interop kit | §33: per-package conformance **mandatory** (nine proofs); §34: shared module deferred for MVP, mechanical-only if introduced; ADR-032/033/034 | 🟢 **Resolved with a refinement I accept**: capability mandatory, vehicle deferred is *better* than my recommendation — it applies the chain's own L1→L2 logic (no shared infrastructure before evidence of a second consumer) while the mandatory conformance tests prevent the drift class |

---

## 2. Residual findings

### 🟠 R-1 — The protocol still owes one sentence: the provenance law of session-facing outputs

§26's flow terminates at "hypothesis-grade Phys artifact/**session result**" — so session recording is in scope, but v1.2 never states the expected provenance of a session-recorded correspondence. The frozen contamination law (Phys §13.2) makes it deterministic: **`Session.Identify` on hypothesis-grade operands yields `HYPOTHESIS`, never `IDENTIFIED`.** This is not hypothetical risk — it is the exact failure the GR chain hit (V4-2): a pinned test expecting `IDENTIFIED` that could never pass through the public API. Add one sentence to §28: *"Any session-recorded correspondence in the protocol operates on hypothesis-grade operands and therefore yields `HYPOTHESIS`-grade relation artifacts (frozen contamination law); the protocol expects and records this — it is never an error."* In the same paragraph, name the minting path: **`hypothesis.NewCandidateConcept` is the sole public raw mint available to external tooling** (frozen §5.4 path 2), so `ToPhysObject` routes there — content-derived, lazy. Both are frozen-spec consequences, not new architecture; stating them converts the last silent spot in the protocol into a pinned contract.

### 🟠 R-2 — phys-gr's theory-local algebra vs. PhysMath: the one coherence rule the ecosystem now needs

`phys-gr` **exists**, with its own `GRExpr` algebra (GR chain D19–D21). v1.2 is silent on how an existing theory-local algebra relates to the new PhysMath `Expression` sort — and the TOE integration plan will have to compose both. Pin one paragraph: *theory-local algebras are permitted and remain package-local; PhysMath is the destination for cross-theory reuse; a theory-local algebra may project into PhysMath's mirror subset via its own deterministic mapping; no forced migration in MVP; movement follows the L1→L2 promotion rules (named second consumer, worked example).* Without this, the first phys-* implementation plan must decide it — violating §47.

### 🟡 Boundary conditions to record (not blockers; one line each)

1. **Protocol transitions are mechanical.** §24 ("never inferred") + §27 ("human curation outside this runtime process") imply it, but state it: `REQUEST_REVISION`/`REJECTED` are triggered by explicit rules over proposal content (non-projectable components, missing required refs/contracts), never discretionary mid-process. The specific trigger table belongs to the integration plan.
2. **The sort-set deferral is owned and sequenced.** §8's "exact final core sort set is implementation-pinned before coding" is legitimate — but the CLOSED status is conditional on the PhysMath implementation plan pinning (a) the exact set, (b) the ten-field per-sort contract for each. Record that condition where the deferral is stated.
3. **The research-status vocabulary shrank.** `DERIVATION_FAILED`, `PHYSICAL_CONTRACT_MISSING/CONTRADICTED` (v1.0 §31) are gone from §35.2. Defensible if contract-completeness is enforced by the readiness gate rather than a status value — but say so, or the handoff gate's failure mode loses its name.
4. **`UNREPRESENTABLE` dual-use residue.** It now appears as projection classification (§24) and formalization outcome (§35.3). The meanings are aligned (both = "cannot map to Phys"), and the owning schemas disambiguate — one acknowledgment line under §40's discipline closes it.
5. **§4.1's illustration vs. §32's rule.** The example `Math physmath.Object` gestures at a universal type that §32 forbids (`no universal Artifact interface{}`). Reword the illustration to a concrete sort (e.g., a `MathArtifact` handle or `physmath.Expression`) so the example can't be read as licensing the antipattern.
6. **Duplicate projection code until consolidation.** With the shared module deferred, each consumer implements the (identical, deterministic) projection independently. Accepted trade — but the implementation plans should note the future consolidation point explicitly.

---

## 3. What v1.2 gets right (verified, no action)

**§45's supersession table** is exactly the required bookkeeping — six decisions with explicit previous→current pairs and the "no earlier closed decision simultaneously active" clause; I cross-checked it against plan3's ADR-003/006/018/020/025 and plan4's interop optionality — coverage is complete. **The 40-ADR register** survives adversarial pairing: the risky combinations (ADR-004 dependency vs ADR-024 no-trust; ADR-019 established interpretation vs ADR-020 proposed interpretation; ADR-002 zero-dependency vs ADR-026 mirror obligation — which is a specification-level duty, not an import) are all coherent. **§4's in-memory vs. serialized split** (adopted from Qwen) is the correct fix for the database-resolver antipattern, and the rule that in-memory objects must not require artifact-ID lookup is a genuinely useful mechanical boundary. **§12/§13's split** of `PhysicalInterpretation` (phys-* contract) from `ProposedInterpretation` (Project A research state) resolves an overlap no prior round had fully named. **§38's t/k restoration** — the author caught their own regression; the mandatory adversarial-test framing is retained. **§44** is the most complete test inventory in the document chain — dependency, identity, ownership, condition, interpretation, projection, formalization (including "no runtime AcceptFormalization"), replay, and numerical-boundary suites, each mechanically checkable. **§34's interop decision** and **§41's ledger-as-lifecycle-guidance** both match the structural review's intent. The frozen Phys is never contradicted: §20/§21/§22/§27 are the cleanest expression yet of "limitations are boundaries, not defects."

---

## Bottom line

**Authorize — with R-1 applied before any implementation plan is written.** The v1.1 → v1.2 delta is what closure actually looks like: every deferred proviso became an enumerated rule, every implied artifact was either pinned or explicitly labeled future, and the one strategic reversal (⊥) was executed with a correct enforcement argument and honest supersession. The two 🟠 items are paragraphs, not decisions: R-1 imports a lesson this project already paid for once (state the contamination law and name the sole mint path), R-2 prevents the first phys-* implementation plan from having to invent the theory-local-algebra relationship. With those, plus the six one-line boundary conditions folded into the implementation plans' inputs, §48's authorization is earned — and the next artifact in this chain, for the first time across the entire review history, can be exactly what the footer promises: a project-specific implementation plan with nothing architectural left to invent.