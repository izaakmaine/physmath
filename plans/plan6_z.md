# Adversarial Review: `plan6.md` (Final Architecture Closure Plan → v1.3)

## Verdict

**This is the right closure program — it converts the correct joint-level decisions (identity framing, IR table, projection algorithm, provenance law, no-new-Phys-API, state/outcome split) into authored artifacts, and its meta-process (§30 blocker-only audit, §31 stopping rule) is the first genuinely checkable termination condition in the chain. But the plan contains two blocker-class defects of its own that would corrupt v1.3 if authored as written: the phys-artifact dependency diagram quietly violates the most protected invariant in the entire project chain (frozen Phys), and the `DifferentialDomain` enum smuggles physical ontology into PhysMath in direct contradiction of the constitution and of plan5 §38.** Both have cheap, precedent-backed fixes. A further set of decisions must be pinned *during* authoring or the ambiguity migrates into v1.3's table cells. Scope caveat: the four underlying reviews (plan5_deepseek/gemini/qwen/z) and the N-/R- finding IDs are not in this packet; their summaries are taken at face value.

---

## 1. Disposition against prior findings

| Prior | plan6 resolution | Verdict |
|---|---|---|
| R-1 provenance law | §20 normative rule, verbatim in the required direction ("remains hypothesis-grade… never treated as a failure… never corpus promotion") + §25 trust test | 🟢 Resolved |
| R-2 theory-local algebras | §13: permitted, package-local, deterministic mapping into PhysMath only within the frozen sort set, no forced migration, no plugin AST | 🟢 Resolved |
| v1.2 §8's "pinned before coding" deferral | §6: normative per-sort table, MVP/DEFERRED/FORBIDDEN classification; §47-consistency restored | 🟢 Correct move |
| In-memory vs serialized | §4/§5/§32 as law with resolver rules | 🟢 Resolved |
| Identity proviso | §3: exact struct, framed digest, recursion lock, four-field equality | 🟢 Resolved (one internal wrinkle, F-5) |
| Status-vocabulary dual use | §18 state/outcome split + §24 typed state machines | 🟢 Resolved (one naming near-collision, F-6) |
| Numerical data-flow mismatch | §23 lowering chain, Numerical never parses PhysMath | 🟢 Resolved (ownership pin owed, F-7) |
| Optional interop kit | §34: conformance mandatory, module deferred — with one new twist, see 🔴-1 | 🟢 With a contradiction to fix |

---

## 2. 🔴 Blockers to fix before authoring v1.3

### B-1 — §2's dependency shape makes **Phys** depend on phys-artifact. That requires modifying frozen Phys — the one thing this entire chain never does.

The §2 diagram lists `Phys` among phys-artifact's dependents. Phys is byte-frozen (specs_v2_3; the GR chain's operational freeze; zero code/spec/tree/module changes). Adding a `require` plus imports to Phys violates it — and a conforming agent following the diagram either commits that violation or halts. The GR chain already solved this exact shape: **frozen Phys is a reference *target*, never a new-import source** (phys-gr computed hashes over frozen canonical bytes externally).

**Fix (pin in v1.3):**
- Phys imports **nothing new**. Phys is not a phys-artifact consumer.
- Phys's internal identity is already fully specified by its frozen contract (canonical object JSON §10.4, hashes §9.2/§16, `schema_version:"1"`). Ecosystem references *to* Phys artifacts are `ArtifactRef{Namespace:"phys", …}` **computed by consumers** (phys-*, TOE-Agent) over the frozen-spec canonical bytes.
- Dependency edges: `PhysMath → phys-artifact`, `phys-* → phys-artifact`, `TOE-Agent → phys-artifact`; `Phys → ∅`.
- Add **"Phys remains byte-frozen; zero new imports"** to the §1 lock table — its absence is exactly what let this slip through.

Consequential note: PhysMath's dependency surface becomes `{stdlib, phys-artifact}` — state it explicitly; a neutral identity module does not violate mathematics-only, but the surface should be declared.

### B-2 — §12's `DifferentialDomain{COORDINATE, TIME, RG_SCALE}` puts physical ontology inside PhysMath core.

`RG_SCALE` is unambiguously physical semantics; `TIME` as a physical flow is likewise. plan6 §12 therefore contradicts (a) the constitution (PhysMath owns no physical ontology/regimes), and (b) plan5 §38, which plan6 itself restores: *"Physical roles such as t = physical time, k = renormalization scale are supplied by physical contracts."* The closed enum recreates in miniature the exact Dancer/Dance/Stage-ontology violation the architecture bans.

**Fix (and it is a simplification, per §29's own philosophy):**
- The distinguisher is **parameter identity**, already pinned (v1.2 §13 scope/binding; §24): distinct parameters are distinct identities; free parameters are not alpha-renamable; renaming display never changes identity. The mandatory negative test passes on identity alone.
- Derivative operators carry `(target parameter identity, optional domain tag)` where the tag is **open, caller-supplied annotation data** — PhysMath core defines *no* values; `TIME`/`RG_SCALE`, if the TOE wants them, are declared by phys-* contracts or Project A. Tags are preserved through transformations and never consulted for equality/composition semantics.
- If a closed enum is insisted upon, it lives at the phys-*/TOE-Agent level, never in PhysMath.

---

## 3. Decisions that must be pinned during v1.3 authoring (else ambiguity migrates into the cells)

**F-1 — The permitted-loss set for `PARTIAL` is empty and undefined (§9).** The compositional rule turns on "contract explicitly permits loss" — but no contract defines any permitted loss. As written, one agent classifies a `FunctionApplication` with a projectable argument as `PARTIAL` (keeping the argument), another as `UNREPRESENTABLE` (the function is required). Pin: **MVP has zero permitted-loss contracts; `PARTIAL` is defined but unreachable in MVP; permitted losses are added only by versioned projection-contract amendments.** Also define "required children" as "all children, except those covered by a permitted-loss contract."

**F-2 — §6's table cells need rule references, not checkmarks.** For 15–16 sorts × ~10 behavioral columns, a cell saying "normalize: yes" is the same ambiguity relocated. Each cell must be exactly one of: `N/A` / `RULE <id>` (pinned in a v1.3 appendix, GR-Diff-table style) / `DEFERRED + containment`. Also resolve the Symbol conflict: plan6 says "preserve the v1.2 inventory" (no Symbol sort) while §7's mirror list begins with Symbol. Pin: **Symbol is an Expression node kind (as in Phys), not a standalone sort; the mirror subset names Expression's node kinds.**

**F-3 — §19 should pre-name the mint path now.** "The implementation plan must enumerate the exact existing public Phys APIs" is right, but the enumeration is already known and tiny: **`hypothesis.NewCandidateConcept` is the sole public raw mint (forced `HYPOTHESIS`/`NONE`)**; `ops.Apply`/twelve ops; `session.Session` actions; `core` constructors/inspectors. Pinning the mint path in v1.3 costs one sentence and closes the last place an agent might assume an ingest API exists.

**F-4 — §29's "no dynamic AST extension mechanism" must not be read as revoking ADR-010.** The *static, versioned* sort-extension protocol (explicit contract, no runtime invention) stays; what's forbidden is dynamic runtime invention. One clarifying line, since §1 locks "PhysMath AST is closed" while ADR-010 permits versioned extension — "closed per version" is the reconciled reading.

**F-5 — Exactly one digest formula must survive into v1.3.** §3 states an abstract law (`Namespace‖SchemaVersion‖ArtifactType‖payload`) and then the framed law (`Len32(namespace)‖namespace‖Version32…`) in different field orders; the framed one is the pin, and the abstract one should be deleted or marked illustrative. Related: frozen Phys versions are strings (`"1"`), `ArtifactRef.SchemaVersion` is `uint32` — pin the mapping for Phys-targeted refs.

**F-6 — `PARTIALLY_PROJECTABLE` (protocol outcome) vs `PARTIALLY_REPRESENTABLE` (projection class)** are distinct typed domains per §24's own discipline — add the one-line acknowledgment so the near-collision doesn't undo the earlier `UNREPRESENTABLE` fix.

**F-7 — `ComputationalContract` ownership (§23).** Producer: phys-* lowering; consumer: Numerical. Pin the schema home and add it to §34's artifact families.

**F-8 — Smaller pins:** §11 — state that **MVP has no condition-discharge operation** (conditions accumulate; discharge is a future explicitly-defined operation), else "explicitly defined operation" dangles; §17 — pin `ProjectionPolicy` values (`STRICT_MIRROR_ONLY` for MVP) and the `ValidationReport` field set (= §15's checks); §21 — embedded `ArtifactRef`s serialize canonically as their four components in fixed order (no recursion issue, but say it); §31 — the stopping rule cites `N-1…N-12`/`R-1…R-6` from reviews not in the packet — inline the list into v1.3 so the rule is checkable from the frozen document itself; §27 — the supersession table must gain entries for: phys-artifact creation (amends ADR-033/034), state/outcome split (refines v1.2 §28), lowering chain (refines v1.2 §39), and domain-tag policy (whatever B-2 resolves to); research statuses — v1.2's vocabulary dropped `DERIVATION_FAILED`/`PHYSICAL_CONTRACT_MISSING/CONTRADICTED`; either restore them or state where those failures live (obstacle class / response `ValidationReport`) — the readiness gate needs a nameable failure mode.

---

## 4. Verified correct (no action)

The phys-artifact concept itself (neutral identity module, no semantics, closure rule against incompatible redefinitions) is the correct resolution of the Gemini deadlock — only its Phys edge is wrong (B-1). §3's framing fix (`"a"+"bc" ≠ "ab"+"c"`) and the no-recursive-identity lock are exactly the GR D22 lesson generalized. §9's compositional classification with "projectable(child) ≠ projectable(parent)" is deterministic and correct. §10's rejection of a universal `Projectable` interface is right and consistent with the no-universal-envelope lineage. §14 dissolving promotion (permanently distinct artifacts, no `Promote()`/`Accept()`/`Approve()`) is cleaner than any workflow design and consistent with ADR-019/020. §15's narrow declarative contract with mechanical validation is the right anti-rule-engine call. §19's "a reviewer-suggested API is not evidence the frozen API contains it" is an important discipline. §22's meta-ledger (research replay ≠ authority replay) completes the authority-local model correctly. §23's lowering chain fixes the genuine data-flow mismatch. §25's Given/When/Then format with failure conditions makes the acceptance suite executable rather than nominal. §26's authority tuple should record: module `github.com/PithomLabs/phys`, spec `specs_v2_3.md`, the 42-file operational freeze record, commit pinned at run time. §28's "if a row has no normative decision, v1.3 is not closed" converts the recurring complaint into a mechanical completeness test — the best governance instrument in the document chain. §30's blocker-only prompt with the (a)–(d) finding taxonomy and §31's 4×0-BLOCKER rule is a real stopping condition; one hardening: any finding that *demonstrates* two legitimate implementations is a BLOCKER by definition regardless of the reviewer's label, so classification cannot be gamed.

---

## Bottom line

**plan6 is the correct final pass — and it is one revision away from being executable as written.** Fix B-1 (Phys is a reference target, never a phys-artifact importer; add the byte-freeze lock to §1) and B-2 (parameter identity + open caller-supplied tags; no physical enum in PhysMath core), pin F-1 through F-8 during authoring, and v1.3 can honestly claim what v1.2 could not: every architectural noun converted into an owned, typed, canonical, validated contract, with the §30/§31 protocol providing the first termination condition in this chain that does not depend on reviewer taste. The implementation plan that follows should then be as boring as §31 promises — sequence, files, mandated interfaces, tests, golden vectors — because, for the first time, every decision it would otherwise have to make has already been made, checked, and frozen.