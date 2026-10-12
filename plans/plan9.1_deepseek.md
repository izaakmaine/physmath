## Adversarial Review — `plan9.1.md` (Revised Implementation Plan, `phys-math` + `phys-lib`)

**Context reviewed:** `plan9.1.md` against `plan9_notes.md` (the requirement that triggered the revision), `PM-REP-001 v1.4` (the attached specification), and the target: a coding agent, given the plan and its authority, must not have to invent any architectural decision.

**Verdict:**  
**NOT CLOSED. FOUR DEMONSTRATED BLOCKERS REMAIN.**

`plan9.1.md` correctly absorbs the `phys-math` / `phys-lib` / `core` structural requirement, introduces the core/specific promotion protocol (§8), preserves the four-artifact protocol reuse discipline (§10), and correctly demotes `phys-gr` from final architecture to refactoring laboratory (§14). The Go package-name and single-module rationale is idiomatic and consistent with `plan9_notes.md`. The revised module graph (§3), dependency directions, and quality gate (§18) are materially stronger than plan9.md.

The four remaining blockers are specific and each meets the review standard (contradiction, two legitimate implementations, missing schema/validation, or self-declared unresolved architectural decision).

---

## I. What `plan9.1.md` gets right

Preserve through any revision:

- §1 locked architectural facts (sibling workspace; `phys-gr` preserved; `core/` empty start; `phys-lib/gr` owns the equivariance fixture; `fixture2` test-only; GR protocol files reused).
- §3 target module graph with the four-level partial order `phys-artifact → phys-math → phys-lib/core → phys-lib/<framework>` and explicit prohibition of reverse/lateral edges.
- §4 `phys-artifact` strictly mechanical with a vocabulary boundary test.
- §5 `phys-math` purity rule and 7-rule `(ProducerNamespace, RuleID)` lock.
- §6 `phys-lib/core` empty start with mandatory §8 promotion record and “useful to us is not enough”.
- §7 `phys-lib/gr` owning all GR semantics; Sin/Cos/regime logic flagged as GR-local under §15 theory-local algebra.
- §8 core/specific classification protocol with the eligibility questions and the required record shape.
- §9 refactoring map with the three-destination classification (`PHYS-MATH | PHYS-LIB/CORE | PHYS-LIB/GR` plus `SPLIT/REPLACE/DELETE/DEFER`) and per-row interface column.
- §10 GR protocol migration strategy: reuse actual files as authority, nested layout for the new fixture, PROTOCOL GAP discipline.
- §11 equivariance fixture plan aligning with v1.4 §16.4–§16.11.
- §12 second-framework `fixture2` at `phys-lib/internal/conformance/fixture2/`.
- §13 layered test matrix and contamination-priority list.
- §14 `phys-gr` preservation + port-not-move; `replace` removal gate as a multi-condition gate.
- §15 risk register including core-as-dumping-ground and workspace drift.
- §18 dependency-ordered Phase 0–10 with a final quality gate.

---

## II. Demonstrated blockers

### Blocker 1 — B1 and B2 are still self-declared open

- §16 lists two “ARCHITECTURAL BLOCKER” entries: B1 (Sin/Cos node-set placement) and B2 (regime non-vanishing placement).
- Each entry ends “Recommended: A” but with no decision.
- §18 closing line: “Awaiting architectural input on B1/B2 (or approval to proceed under the recommendations) before Phase 0 execution.”
- The plan thereby declares itself not implementation-authorizable.

**Concrete counterexample:** A coding agent receives `plan9.1.md`. Phase 0’s first task is inventory. It then needs to proceed to Phase 5 (GR extraction) where B1 determines whether Sin/Cos go into `phys-lib/gr` as theory-local algebra (option A) or must be deferred (option B). Without a decision, the coding agent cannot proceed to Phase 5 without guessing.

**Failure consequence:** Phases 5–8 stall, or the coding agent invents the resolution — violating v1.4 §24.2, which forbids the coding agent from deciding whether the equivariance fixture may use trig, whether local algebra may extend PhysMath, and whether the GR-local trig becomes a permanent `phys-lib/gr` concept.

**Exit condition:** Convert §16 from “blockers awaiting input” to “decisions adopted”:
- B1 → A: Sin/Cos/UF-shape remain in `phys-lib/gr` via §15 theory-local algebra; equivariance slice does not use them.
- B2 → A: regime non-vanishing conditions are framework-level `PhysicalAssumption` artifacts in `phys-lib/gr`; rule-scoped conditions (e.g. `Pow(x,0)`) travel via `OperationInvocation.ConditionRefs` per v1.4 §17.1 and §17.4.
Or amend v1.4 to accommodate the alternative.

The recommendations are already in the plan. Adopting them closes the blockers without further architectural input. The plan already says this is a legitimate option: “or approval to proceed under the recommendations.”

---

### Blocker 2 — “Preserve `phys-gr` byte-identical” contradicts v1.4 §26 step 10

- `plan9.1.md` §14: “Preserve: `phys-gr` repository byte-identical (regression baseline + migration source; its suite keeps passing independently).”
- `plan9.1.md` §3 diagram: “`phys-gr/` (preserved, untouched; removed from go.work at completion).”
- v1.4 §26 implementation order step 10: “Refactor the GR protocol fixture into the canonical package pattern.”
- v1.4 §1.4 requires the canonical package pattern (`AGENTS.md + manifest.json + protocol/derivation_test.go`) and states the existing GR example is “the protocol precedent” that must be “reused or refactored rather than replaced by a newly invented package protocol.”
- v1.4 §16.2: “No alternative ‘equivariance fixture format’ is introduced. The fixture MUST follow the same structural and operational pattern used by the existing General Relativity example.”

The GR example’s existing layout is flat: `solution/{README.md, manifest.json, manifest_test.go, derivation_test.go}`. The canonical pattern is nested: `<pkg>/{AGENTS.md, manifest.json, protocol/derivation_test.go}`.

Two legitimate readings:

1. **“Refactor” = the fresh `phys-lib/gr` fixture becomes the canonical-pattern GR fixture; the original `phys-gr/solution/` stays as archive.** Under this reading, plan9.1 §14 is fine; v1.4 §26 step 10 is satisfied by the new `phys-lib/gr` package’s nested fixture.
2. **“Refactor” = `phys-gr/solution/` itself must be rewritten into the nested layout.** Under this reading, plan9.1 §14 (“byte-identical”) is a direct contradiction.

The plan does not state which reading it adopts. Its §10 says “new fixture uses the canonical v1.2 nested layout” — implying the new `phys-lib/gr` fixture only — but §14 says “phys-gr preserved byte-identical”.

**Concrete counterexample:** Phase 6 in §18 says “GR protocol migration (equivariance pkg in nested layout; GR-schema reuse)”. Phase 5 says “GR semantics port”. There is no phase in §18 that refactors `phys-gr/solution/`. If the reading is (2), Phase 6 must also refactor `solution/`, contradicting §14. If the reading is (1), the plan should say so.

**Failure consequence:** Either the coding agent violates §14 by refactoring `phys-gr/solution/`, or it produces a Phase-1 implementation that fails v1.4 §26 step 10 as literally read.

**Exit condition:** Add an explicit statement in §14 and §16: “The GR protocol fixture is refactored into the canonical nested package pattern as the new `phys-lib/gr/.../protocol/derivation_test.go`; the original `phys-gr/solution/` is preserved unchanged as historical reference and compatibility anchor. v1.4 §26 step 10 is satisfied by the new `phys-lib/gr` fixture.” Or, if the intent is (2), amend §14 and drop the byte-identical claim.

---

### Blocker 3 — Phase 4 (core extraction) precedes Phase 5 (gr extraction) with nothing to classify

- §18 Phase 4: “`phys-lib/core` candidate extraction (module + EMPTY core; classify per §8; promote ONLY with records; default stays in gr)”.
- §8 requires classification to be based on “Known/current consumers”, “Potential future consumers”, “Theory-specific assumptions”, etc.
- §18 Phase 5: “`phys-lib/gr` extraction + refactoring (framework 6 families, GR semantics port, GR-local math via §15, B1/B2 rulings applied)”.

If `phys-lib/gr` does not yet exist, there is no concrete code to classify. Phase 4’s classification step is either a no-op or must operate on the *original* `phys-gr` — but plan9.1 §9’s refactoring map is a *plan*, not a classification of live code, and the plan says nothing about doing the classification before extraction.

**Concrete counterexample:** A coding agent reaches Phase 4. It has `phys-artifact`, `phys-math`, and an empty `phys-lib/` with no `gr/`. It must “classify per §8” but has nothing to classify. Either it invents classifications based on guesses, or it fails to proceed, or it reads “classify” as “set up the classification framework” — a reading the plan does not support.

**Failure consequence:** Ambiguous execution of Phase 4; risk of premature `core` promotion based on guesses rather than evidence; risk of the coding agent implementing Phase 4 as a no-op and never revisiting core classification.

**Exit condition:** Either:
- Merge Phase 4 into Phase 5 or move it after Phase 5 so classification operates on extracted code, or
- Explicitly state Phase 4 is “create `phys-lib/` module and empty `core/` package; classification framework setup only; actual promotions occur during or after Phase 5”.

The plan’s own §6 (“core starts empty; promotion requires the §8 record with demonstrated multi-framework stability”) implies classification must follow extraction, not precede it.

---

### Blocker 4 — `phys-lib/gr` re-implementation scope vs the `solution/` semantic anchor is not pinned

- §14: “`solution/` behavior is the semantic anchor for the GR port.”
- §9 (row for `solution/`): “preserved in `phys-gr` (reference); re-expressed in `phys-lib/gr` | REPLACE (re-implement, not move)”.
- §14: “Port, don’t move: new canonical code is written fresh in the sibling modules against the spec, validated against ported vectors.”

Neither §9, §14, nor §18 specifies:
- whether the fresh `phys-lib/gr` implementation is required to reproduce `solution/`’s Schwarzschild derivation end-to-end in Phase 1, or only the equivariance fixture;
- what “ported vectors” concretely are (intermediate outputs? hash chains? `expect(op,params)` sequences?);
- whether any divergence between `phys-lib/gr` and `solution/` is acceptable — and if so, under what condition;
- whether the 14-step Schwarzschild derivation is inside Phase-1 scope.

**Concrete counterexample:** A coding agent reaches Phase 5. §18 says “GR semantics port”. It must decide whether to port `metric/`, `connection/`, `curvature/`, `vacuum/`, `limit/` (yes, per §9) AND the 14-step Schwarzschild solution (ambiguous). §9’s row for `solution/` says “re-expressed in `phys-lib/gr`” — that suggests yes. But §17 deferred-work list does not mention “port the full Schwarzschild derivation” — its absence implies it might be out of Phase 1. The plan is silent.

**Failure consequence:** Substantially different Phase-1 scopes; the coding agent either over-builds (porting the full Schwarzschild derivation when only the equivariance slice was required) or under-builds (only the equivariance slice, leaving the port as a stub that fails §14’s “validated against ported vectors” criterion).

**Exit condition:** Explicitly state one of:
- `phys-lib/gr` Phase-1 scope includes the full Schwarzschild derivation port, with `solution/` as the semantic anchor and ported vectors defined (e.g. each of the 14 stage outputs, the `A/B` weak-field extraction, the 16 `R_uv=0` assertions).
- `phys-lib/gr` Phase-1 scope is limited to the framework artifacts (§14 family) plus the equivariance fixture; the Schwarzschild port is deferred.

---

## III. Residual issues

Not blockers under the strict test, but a coding agent will encounter them.

- **`fixture2` namespace not pinned.** §12 says “grammar-valid, e.g. `phys-fixture2`”. v1.4 §25.31 accepts any grammar-valid `phys-<domain>` publishing a valid FrameworkArtifact, so this is not architectural — but v1.4 §25.34’s cross-implementation determinism requires a fixed namespace for the fixture2 golden vectors. Pin the exact string.

- **§13 cross-implementation determinism test unspecified.** v1.4 §25.34 requires two independent implementations to produce identical golden `ArtifactRef`s. A single coding agent cannot verify this. Say whether this is (a) a single-implementation fixture-replay determinism check, (b) a formal acceptance criterion deferred to an orchestration layer, or (c) verified by an explicit second-implementation requirement in the plan.

- **`phys-lib/gr → phys-lib/core` dependency when core is empty.** §3 shows the dependency direction; §6 says core may remain empty at Phase-1 completion. If `gr` has no code depending on `core`, the arrow in §3 is aspirational, not enforced. Clarify that the direction is a *permitted* direction, not a *required* import.

- **§3 does not include `phys-lib/* → phys`.** v1.4 §3 permits `phys-* → Phys (existing public API only, when required)`. The plan’s dependency diagram omits this edge, which may be read as prohibiting it. Add it or explicitly say it is permitted under v1.4 §3.

- **§4 / §5 vocabulary tests may be over-strict.** `phys-artifact` “contains no identifiers matching math/physics vocabularies (`Expression`, `Function`, `Framework`, ...)” — but many non-semantic helper names could match (e.g. `FunctionRef`, `HashFunction`, `FrameworkVersionString`). Similarly `phys-math` forbids “physical dimensions-as-meaning” without distinguishing mathematical dimension (rank, size) from physical dimension (meter, second). The v1.4 §10.4 `Tensor.Dimension UInt16` field is mathematical. State the distinction explicitly.

- **§5 “conventions” forbiddance.** `phys-math` MUST NOT contain “conventions”. But v1.4 §7.11 has `ComparisonOperator` values `EQ, NEQ, LT, LTE, GT, GTE` — are these “conventions”? They are mathematical comparison relations. Clarify that mathematical operators are not forbidden by the “conventions” rule.

- **Version/tag strategy for the three new modules.** `phys-artifact` is greenfield; `phys-math`/`phys-lib` are sibling modules; go.work is local. What version string do `phys-artifact`, `phys-math`, `phys-lib` declare in their `go.mod`? What is the tag strategy when the workspace is published? Not pinned.

- **`phys-lib/gr` framework artifacts and `solution/manifest.json` conventions.** `solution/manifest.json` has `conventions{metric_signature, curvature_convention, cosmological_constant}`. v1.4 §14.2 allows `PhysicalAssumption { AssumptionID, ConditionRefs, Category, Description }`. Phase 5 must decide whether `MTW`, `-+++`, `Λ=0` become `PhysicalAssumption` artifacts with a `Category` value, or something else. §9’s row for `metric/.../curvature/vacuum` says “as explicit `PhysicalAssumption` artifacts, not code comments” — good — but does not pin the `Category` vocabulary. This is a residual because v1.4 permits open `Category` strings, but a plan-level choice would simplify Phase 5.

- **`phys-gr` removal from `go.work` at completion.** §3 says “removed from go.work at completion”, §14 gate is “no active dependency path through phys-gr”. Who verifies, how, and does the workspace still build against the frozen Phys afterwards? Not pinned. Add a CI verification step.

- **B1 B2 “or approval to proceed”** — process question. If the plan proceeds under its own recommendations without explicit approval, does that count as “adopted”? State the adoption rule explicitly: e.g., “If the architect does not respond within N days, option A is adopted as the plan’s decision.” This is a process, not an architecture, decision, but it determines whether the plan starts in a closed or open state.

- **v1.4 §26 step 9 “Refactor/extract phys-gr into the first concrete `phys-*` implementation”** — the plan interprets this as porting to `phys-lib/gr` (§9). Good. But v1.4 §26 step 10’s separate “Refactor the GR protocol fixture into the canonical package pattern” is not clearly addressed (see Blocker 2).

---

## IV. Path to closure

To meet the target:

1. **Blocker 1.** Adopt B1 = A and B2 = A. Convert §16 from “blockers requiring input” to “decisions adopted”. Remove the §18 closing line “Awaiting architectural input on B1/B2… before Phase 0 execution.”
2. **Blocker 2.** State the adopted reading of v1.4 §26 step 10: either the new `phys-lib/gr` fixture is the refactored GR fixture (original `phys-gr/solution/` preserved as archive), or `phys-gr/solution/` must be refactored (which contradicts §14). The current plan implies the former but does not say so.
3. **Blocker 3.** Reorder Phase 4 to follow Phase 5, or explicitly redefine Phase 4 as “create module and empty package; classification framework setup only”.
4. **Blocker 4.** Pin `phys-lib/gr`’s Phase-1 scope with respect to the Schwarzschild derivation and the concrete “ported vectors” definition.

Then apply the residuals as one-line clarifications, and update §18’s quality gate accordingly.

None of these reopens the strategic architecture. v1.4 has settled semantics; the plan must be brought into alignment with v1.4 and its own internal statements.

---

## V. Final verdict

```text
REVIEW: plan9.1.md against PM-REP-001 v1.4

Sibling workspace / module graph:               PASS
phys-gr as refactoring laboratory:              PASS
phys-artifact strictly mechanical:              PASS
phys-math purity:                               PASS
phys-lib/core empty start + promotion protocol: PASS
phys-lib/gr GR-specific ownership:              PASS
Core/specific classification protocol (§8):     PASS
Refactoring map (§9):                           PASS
GR protocol migration strategy (§10):           PARTIAL (Blocker 2)
Equivariance fixture plan (§11):                PASS
Second-framework conformance plan (§12):        PASS
Test matrix (§13):                              PASS as a set
Compatibility/migration strategy (§14):         PARTIAL (Blocker 2, Blocker 4)
Complexity/risk register (§15):                 PASS
Decided D1–D5 (§16):                            PASS
Deferred work (§17):                            PASS
Execution sequence (§18):                       PARTIAL (Blocker 3)
Quality gate (§18 closing):                     PASS as a checklist

B1 (Sin/Cos placement):                         OPEN (Blocker 1)
B2 (regime non-vanishing placement):            OPEN (Blocker 1)
GR fixture refactor vs preserve:                OPEN (Blocker 2)
Phase 4 before Phase 5 sequencing:              OPEN (Blocker 3)
phys-lib/gr Phase-1 scope vs solution/:         OPEN (Blocker 4)

FINAL STATUS: NOT CLOSED. FOUR DEMONSTRATED BLOCKERS.
```

---

## VI. Assessment

`plan9.1.md` correctly absorbs the `plan9_notes.md` requirement and moves the ecosystem to a more idiomatic Go topology. It does so without re-architecting the mathematical representation: `phys-math` is a rename-and-relocate of `physmath`, `phys-lib` is a single module with packages, `phys-gr` becomes the refactoring laboratory, and the recursive core/specific split is now explicit at three levels (`phys-math` = math denominator, `phys-lib/core` = physical denominator, `phys-lib/<framework>` = framework-specific).

The four remaining blockers are all one-decision fixes:

1. Adopt the plan’s own recommendations for B1 and B2.
2. State the reading of v1.4 §26 step 10 that preserves §14’s byte-identical `phys-gr` guarantee.
3. Reorder Phase 4/5 or redefine Phase 4 as scaffolding.
4. Pin the Phase-1 scope of `phys-lib/gr` relative to `solution/`.

Two observations for the wider effort:

- The plan’s own §18 closing line — “Awaiting architectural input on B1/B2 (or approval to proceed under the recommendations) before Phase 0 execution” — is the correct pattern for a coding agent following v1.4 §24.4’s stop rule. The plan is honestly reporting its own open state. This is exactly what v1.4 §24.0 requires.
- The three-level core/specific split (`math → physical-core → framework`) is more than a Go module layout choice. It is the concrete realization of the plan’s own “Architecture of Complexity” principle (plan9_notes.md and v1.4 §2.5). The plan should treat this as a first-class architectural property and its quality gate should require that no classification decision reduces to “we put it here because it was convenient.”

The cycle is close to closure. Adopting the four decisions above — all of which the plan itself already favors — would satisfy v1.4 §28’s handoff rule and allow the coding agent to begin Phase 0 without inventing an architectural decision.