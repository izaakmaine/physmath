## Adversarial Review — `plan9.md` (Revised Against `PM-REP-001 v1.2`)

**Context reviewed:** `plan9.md` re-evaluated now that the v1.2 authority (`PM-REP-001 v1.4`) is attached and inspectable. The plan’s header cites `plans/PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md`; the document you provided is `PM-REP-001 v1.4`, which states it **supersedes v1.1 and all earlier PM-REP-001 versions** and is the “IMPLEMENTATION-PLANNING BASELINE — ARCHITECTURALLY CLOSED FOR PHASE-1”. Treating v1.4 as the intended authority.

**Verdict:**  
**NOT CLOSED. FIVE DEMONSTRATED BLOCKERS REMAIN.** (Down from nine.)

The attachment of v1.2 resolves four blockers outright and defuses one more. But it sharpens two existing blockers and introduces one new one, because v1.2 pins decisions that contradict specific statements in `plan9.md` — most consequentially, the “GR tests preserved unchanged” claim.

---

## I. Blocker-by-blocker delta

| Original blocker | v1.2 resolution | Current status |
|---|---|---|
| B1 — v1.2 authority not attached | v1.2 attached and inspected | **RESOLVED** |
| B2 — B1/B2/B3 flagged unresolved | v1.2 §15, §14.2, §16.2 resolve them; plan just needs to adopt | **PARTIALLY RESOLVED** (plan hasn’t adopted) |
| B3 — B3 layout ambiguity | v1.2 §16.2, §25.19 pin nested layout; §26 step 10 mandates refactor of GR fixture | **RESOLVED as text, NEW CONFLICT with plan §6** |
| B4 — Many-frameworks deferral | v1.2 §16.13 and §26 step 12 require it | **STILL OPEN** |
| B5 — Test-only namespace contradiction | v1.2 §5.1 permits any `phys-<domain>`; §25.31 confirms no registry | **RESOLVED** |
| B6 — “No committed replace” vs existing replace | v1.2 §0.6 keeps existing `phys-gr` as substrate; doesn’t address go.mod | **STILL OPEN** |
| B7 — GR tests preserved vs canonical bytes | v1.2 §26 step 10 explicitly mandates **refactoring** the GR protocol fixture | **NEW, SHARPER CONFLICT** |
| B8 — `OperationInvocation` undefined | v1.2 §17.1 defines it | **RESOLVED** |
| B9 — Convention artifact placement | v1.2 §14.2 permits `PhysicalAssumption.Category/Description`; no strict blocker | **DOWNGRADED to residual** |

---

## II. Blocker 1 — Plan still lists B1/B2/B3 as open, but v1.2 resolves them

`plan9.md §11` flags three “ARCHITECTURAL BLOCKERS REQUIRING INPUT”:

- **B1** — Where do Sin / Cos / UnknownFunction / Pow-int go?
- **B2** — Do regime non-vanishing assumptions live as `MathCondition`, `PhysicalAssumption`, or implicit?
- **B3** — Nested `protocol/` layout vs flat `solution/` mirror.

With v1.2 attached, all three are answerable **without** further architectural input:

- **B1** is resolved by v1.2 §15: a `phys-*` package MAY define a local mathematical algebra when PhysMath lacks the abstraction; the local type MUST remain package-local. The plan’s recommendation A is now the specification-mandated route, not an architectural choice.
- **B2** is resolved by v1.2 §14.2 (framework-owned `PhysicalAssumption`), §17.4 (rule-scoped conditions supplied through `OperationInvocation.ConditionRefs`), and §17.10 (`ConditionRefs` explicit, no discharge). The plan’s recommendation A is correct.
- **B3** is resolved by v1.2 §16.2 and §25.19, which pin the nested `<pkg>/{AGENTS.md, manifest.json, protocol/derivation_test.go}` layout. The GR example’s *schema and loading convention* is reused (§1.4); the *file layout* is the new one.

**Counterexample:** A coding agent receiving `plan9.md` sees three “BLOCKERS REQUIRING INPUT” and cannot know whether to halt for architectural input or proceed. If it proceeds, it must guess at each.

**Failure consequence:** The plan spends its first iteration on architectural questions that v1.2 has already answered, delaying implementation.

**Exit condition:** Replace §11 with a §11 resolutions section: adopt B1=A (local algebra under v1.2 §15), B2=A (framework-level `PhysicalAssumption`), B3=A (nested layout per v1.2 §16.2). No architectural input required.

---

## III. Blocker 2 — “GR tests preserved unchanged” contradicts v1.2 §26 step 10

`plan9.md §6`:

> GR tests migrate by *preservation*, not rewrite: `solution/` suite runs unchanged throughout Phases 1-8 as the compatibility anchor.

`plan9.md §8`:

> Backward-compatible refactor: `solution/` behavior and tests …

`plan9.md §5` Phase 8:

> keep `solution/` untouched; ...

But `v1.2 §26 step 10` says the implementation order explicitly includes:

> Refactor the GR protocol fixture into the canonical package pattern.

And `v1.2 §24.0` step 10 of the plan deliverables:

> the exact path to the executable equivariance protocol fixture.

And `v1.2 §25.19` requires the fixture to be packaged as `AGENTS.md + manifest.json + protocol/derivation_test.go`.

And `v1.2 §16.2` says “No alternative ‘equivariance fixture format’ is introduced” and “… MUSE follow the same structural and operational pattern used by the existing General Relativity example.”

The GR example’s existing `solution/` layout is flat (`README.md`, `manifest.json`, `manifest_test.go`, `derivation_test.go`), not nested (`AGENTS.md`, `manifest.json`, `protocol/derivation_test.go`). v1.2 does not say “future packages follow a different shape than the GR example”; it says future packages follow the same pattern, and the pattern is the one this spec names.

**Counterexample:** The coding agent reaches Phase 9. `plan9.md §6` says leave `solution/` untouched; `v1.2 §26 step 10` says refactor the GR fixture into the canonical package pattern. Two contradictory instructions.

**Failure consequence:** The coding agent either (a) leaves `solution/` flat and produces a Phase-1 fixture that does not match v1.2 §16.2/§25.19 acceptance; or (b) refactors `solution/` and violates the plan’s compatibility-anchor guarantee.

**Exit condition:** Decide which of these is correct and state it in the plan:

1. Refactor `solution/` to the nested layout and update its tests — consistent with v1.2 §26 step 10, breaks “GR tests preserved unchanged”.
2. Keep `solution/` flat and argue the “pattern” is schema/loading only, not directory shape — inconsistent with v1.2 §16.2’s wording, would require a spec amendment.
3. Add `<pkg>/{AGENTS.md, manifest.json, protocol/derivation_test.go}` for the equivariance fixture as a sibling of `solution/` and accept two directory shapes during Phase 1 — inconsistent with v1.2 §16.2’s “no alternative format”.

The plan currently implies (2) but the spec text supports (1).

---

## IV. Blocker 3 — Many-frameworks fixture still deferred but v1.2 requires it

`plan9.md §5` Phase 9:

> Many-frameworks stays a *deferred conformance test* (second interpretation added later, not a synthetic `phys-equiv` now).

`plan9.md §5`:

> Explicitly deferred ... second-framework fixture beyond test harness.

`v1.2 §16.13`:

> The implementation suite **MUST include** two physical framework fixtures that interpret the same or overlapping mathematical representation differently.

`v1.2 §25.22` lists this as an acceptance test.

`v1.2 §26` implementation order includes:

> 12. second phys-* framework/interpretation fixture

**Counterexample:** Phase 11 of the plan is “Conformance + vectors + negatives”. It does not add a second framework fixture. v1.2 §26 step 12 requires one.

**Failure consequence:** The Phase-1 implementation fails v1.2 §16.13 acceptance; the many-frameworks principle is not actually tested.

**Exit condition:** Add Phase 12 to `plan9.md` to include a second conforming `phys-*` framework that publishes a valid `<ns>/framework/1` and the six families. Per v1.2 §5.1 and §25.31, any namespace matching `phys-<domain>` that publishes a valid FrameworkArtifact is legitimate — the fixture’s namespace choice is an implementation detail, not an architectural blocker.

---

## V. Blocker 4 — “No committed `replace`” vs existing `replace` still unresolved

`plan9.md` header:

> (1) multi-module monorepo + `go.work`, DAG `phys-gr -> physmath -> phys-artifact`, no committed `replace`.

`plan9.md §1.1`:

> module `github.com/PithomLabs/phys-gr`, go 1.25.7, depends on frozen `github.com/PithomLabs/phys` via `replace => ../phys`.

v1.2 §0.6 confirms `phys-gr` remains the refactoring substrate; §3 confirms the DAG. v1.2 does not address go.mod mechanics.

**Counterexample:** The coding agent sets up `go.work`. The plan says no committed `replace`, but the existing `phys-gr` module has a committed `replace => ../phys`. Two choices:

- Keep the existing replace and interpret “no committed replace” as applying to new modules only.
- Remove the existing replace and rely solely on `go.work` for local wiring.

Each is consistent with the plan as written.

**Failure consequence:** If the coding agent removes the existing replace and `go.work` does not cover the frozen `../phys`, the build breaks against the frozen authority. If the coding agent keeps it, the plan’s own rule is violated.

**Exit condition:** Clarify whether “no committed replace” applies to all modules (and the existing `replace => ../phys` is removed with `go.work` providing the wiring) or to new modules only (and the existing replace is preserved).

---

## VI. Residual issues (not blockers under the target)

- **Convention artifact placement (B9 in prior review).** v1.2 §14.2 permits `PhysicalAssumption { AssumptionID, ConditionRefs, Category, Description }` where `Category` is an open string and `Description` is explanatory package-owned content. `MTW`, `-+++`, `Lambda=0` can be modeled as `PhysicalAssumption` artifacts with `Category = "convention"` and empty `ConditionRefs`. The plan should state this explicitly in Phase 8 so the coding agent does not need to decide. Not a blocker because v1.2 permits it; residual because the plan doesn’t say it.

- **`OperationInvocation` adoption.** v1.2 §17.1 pins the type. The plan does not name it; the coding agent would infer it from v1.2. A one-line reference to §17.1 resolves this.

- **“`ConditionRefs` do not count toward rule input arity.”** v1.2 §17.2 and §17.1 state this. The plan’s §10 risk register mentions “condition threading” but does not restate the arity rule. A one-line cross-reference helps.

- **`FrameworkVersion` value pinning.** v1.2 §5.5 says the fixture MUST pin its exact framework-version value in fixture data. The plan does not mention this. Minor.

- **Cross-implementation determinism test (v1.2 §25.34).** The plan’s §9 mentions “cross-implementation determinism” but a single coding agent cannot verify two implementations produce identical golden ArtifactRefs. The plan should either flag this as an orchestration-level acceptance criterion or state that it is verified by the fixture’s internal replay determinism (single-implementation) plus the fixture spec being sufficient. Not a blocker per se, but a residual.

- **Phase-8 GR conventions vs `PhysicalEquation.ScopeRefs`.** v1.2 §14.4 permits `ScopeRefs` to target `<owning ns>/assumption/1` or the dependency closure. The plan’s Phase 8 says “re-express ansatz as PhysicalDefinition+PhysicalEquation with explicit assumption refs”. A one-line note on which field carries MTW/-+++/Lambda=0 would tighten this.

---

## VII. Path to closure

To satisfy the target, resolve the four remaining blockers and apply the residuals.

1. **Blocker 1.** Replace plan §11 with a resolutions section adopting B1=A, B2=A, B3=A (all mandated by v1.2).
2. **Blocker 2.** Decide: refactor `solution/` to the nested layout per v1.2 §26 step 10, or amend v1.2 §16.2/§26 step 10 to permit the flat layout. Plan currently says “preserve”, spec says “refactor”.
3. **Blocker 3.** Add Phase 12 (second framework fixture) with a `phys-*` namespace that publishes a valid `<ns>/framework/1` per v1.2 §16.13/§25.22/§26 step 12.
4. **Blocker 4.** Clarify “no committed replace” against the existing `replace => ../phys`.
5. **Residuals.** Add one-line notes on conventions placement (§14.2 `PhysicalAssumption`), `OperationInvocation` usage (§17.1), `ConditionRefs` arity exemption (§17.2), `FrameworkVersion` fixture pinning (§5.5), and cross-implementation determinism (§25.34).

None of these reopens the strategic architecture. v1.2 has settled the semantics; the plan must be brought into alignment with v1.2 and its own internal statements.

---

## VIII. Final verdict

```text
REVIEW: plan9.md against PM-REP-001 v1.4 (a.k.a. "v1.2")

Authority attached:                             RESOLVED (was Blocker 1)
Substrate inventory:                            PASS
Protocol precedent identified:                  PASS
Math/physics split assessment:                  PASS
Target architecture:                            PASS
Module/package plan:                            PASS
Refactoring map:                                PASS
Phase decomposition:                            PARTIAL (Blocker 3)
Protocol migration:                             FAIL (Blocker 2)
Equivariance vertical slice:                    PASS
Compatibility strategy:                         FAIL (Blocker 2, Blocker 4)
Test strategy:                                  PASS as a set
Risk register:                                  PASS
Complexity invariants:                          PASS

B1/B2/B3 flagged blockers:                      RESOLVED by v1.2 §15/§14.2/§16.2
                                                 (plan hasn't adopted; Blocker 1)
B3 layout:                                      RESOLVED textually;
                                                 conflicts with plan §6
Test-only namespace:                            RESOLVED by v1.2 §5.1 / §25.31
OperationInvocation vocabulary:                 RESOLVED by v1.2 §17.1
GR tests preserved vs canonical bytes:          NEW BLOCKER (Blocker 2)
Many-frameworks deferral:                       STILL OPEN (Blocker 3)
“No committed replace” vs existing replace:     STILL OPEN (Blocker 4)
Convention artifact placement:                  DOWNGRADED to residual

Blocker 1  B1/B2/B3 resolutions not adopted:     OPEN
Blocker 2  GR fixture refactor vs preserve:      OPEN
Blocker 3  Second framework fixture:             OPEN
Blocker 4  replace semantics:                    OPEN

FINAL STATUS: NOT CLOSED. FOUR DEMONSTRATED BLOCKERS.
```

---

## IX. Assessment

`plan9.md` remains the most operationally useful document in the series, and the attachment of v1.2 raises its authority substantially. Four of the nine original blockers are resolved outright; one is downgraded to a residual; three remain; and one new blocker arises because v1.2 pins a behavior (`§26 step 10` refactoring the GR fixture) that `plan9.md` explicitly denies.

The remaining work is short and specific. Adopt v1.2’s answers to B1/B2/B3, decide whether to refactor `solution/` or keep it flat (and, if the latter, amend v1.2 §16.2/§26 step 10 in a follow-up), add the second framework fixture to Phase 12, and clarify the `replace` semantics. The residuals are one-line clarifications.

Two observations worth recording for the wider effort:

- **The plan-first handoff is now working.** v1.2 §24.0 requires the coding agent’s first deliverable to be a plan, not code. `plan9.md` is exactly that plan. This is the correct pattern and should be preserved.
- **`plan9.md §11` was written before v1.2 was complete.** It treats B1/B2/B3 as architectural blockers when v1.2 has since settled them. The fix is to convert §11 from “blockers requiring input” to “resolutions adopted from v1.2” — no new architectural decisions required.

The cycle is now within reach of closure: adopt v1.2’s answers, resolve the GR-fixture refactor question, add the second framework fixture, and clarify the replace rule. That is one amendment, not another plan.