# Adversarial Review: plan9.2 — Implementation Plan (`phys-artifact` + `phys-math` + `phys-lib`)

## Verdict

**GO — conditional on one must-pin cluster and a handful of one-liners. No redesign-class blockers.** The plan correctly absorbs every finding from the plan9.1 review into explicit decisions (D1–D17), resolves the three blockers exactly along the recommended lines, and — notably — exercises real adversarial discipline by *rejecting* three reviewer-introduced inventions (R1/R3/R4) with spec citations rather than adopting them. What remains is one scope wobble that will stop the coding agent mid-Phase-5 unless pinned (the plan's own §7 map and its Phase-5 task list point in different directions, and the expressibility wall guarantees the stop), plus the usual one-line residue. Nothing reopens the spec, the topology, or any settled decision.

---

## 1. Prior-findings disposition (all verified)

| Prior finding | plan9.2 resolution | Verdict |
|---|---|---|
| 🔴 P1 core durable-artifact status | **D5**: in-memory only; no FrameworkArtifact, no durable families, no ArtifactRef-bearing schemas, no namespace | 🟢 Resolved |
| 🔴 P2 Phase-1 GR scope | **D7**: framework + six families + GR-local algebra + fixture; Schwarzschild stays canonical in `phys-gr/solution/`; **zero frozen-Phys imports**, zero frozen-namespace references | 🟢 Resolved (recommendation (a) adopted; R5/F5 record the boundary) |
| 🔴 P3 shims vs. lint contradiction | **D4**: no phys-gr imports under any circumstance, no shim exception; validation via copied golden vectors only | 🟢 Resolved — and stronger than recommended (shim clause deleted outright) |
| 🟠 P-4 namespace/module decoupling | **D12**: durable `physmath` and `phys-gr` namespaces unchanged despite Go renames; `phys-fixture2` pinned | 🟢 |
| 🟠 P-5 BaseFrameworkRefs | **D13**: data dependency only; zero inter-framework Go imports in Phase 1 | 🟢 |
| 🟠 P-6 fixture2 conformance | **R4**: adopts the *literal* reading — fully conforming minimal framework, all six families (§11 confirms) | 🟢 Resolved by explicit choice, not left open |
| 🟠 P-7 frozen-Phys wiring | **R5/F5/D17**: go.work use-directives for any future import; no fabricated versions | 🟢 |
| 🟠 P-8 schematic ParameterIDs | **D10 + Phase 7/§10**: `rho_X = rho_Y = [PID_0, PID_1]`, `f = [PID_0]`, every durable free symbol carries its declaring-slot PID; no bare `NewSymbol("g")` | 🟢 Resolved with exact values — the fixture is now reproducible |
| 🟠 NF-1 durable-reference scope | **Partially**: D15 (closed Category enum; empty `ConditionRefs` for non-expressible) — but see 🔴-class finding below for the definitions/equations half | 🟠 Incomplete |
| 🟢 NF-3 `rho_X ≡ rho_Y` coincidence | Phase 7: "canonical-coincidence documented as intentional — distinction lives in interpretation" | 🟢 Absorbed verbatim |
| 🟡 P-9 assumption expressibility | **D15** | 🟢 (one wording pin, P-C below) |
| 🟡 P-10(a) version-label | §1: recorded, operative rule = file content governs, single name adopted | 🟢 |
| 🟡 P-10(b–d) | **D6** (Phase-4 mechanism-only); **D16** (mathematician test = review, scans = CI); §7/§6 cross-ref — see P-D | 🟢 / 🟠 |

---

## 2. The must-pin: Phase-5 GR re-expression scope collides with the expressibility wall

Two plan sections disagree, and one reading is partially unimplementable:

- **§7 map** (metric/connection/curvature/vacuum row): "formula loops re-expressed on phys-math," CLASS `PHYS-LIB/GR`, SEQ Ph5; replay row: "`curvature`/`vacuum_check` as ops → gr-side protocol adapter."
- **D7 + Phase 5 task list + F1**: Phase-1 deliverable is framework + six families + GR-local algebra + fixture; the Schwarzschild protocol and its computation pipeline are deferred; Phase 5's tasks never mention re-implementing Christoffel/Ricci computation.

Meanwhile the expressibility wall (established across the GR chain and reflected in §5.7/§8.6/D1/D9) makes part of the map impossible as written: the ansatz `diag(−A(r), B(r), r², r²sin²θ)` **cannot** be durably expressed in PhysMath families — `sin` requires a `physmath/function/1` artifact with a body (none can exist), `A(r)`/`A′(r)`/`B″(r)` are local-algebra content, and `Tensor.ComponentRefs → physmath/expression/1` forbids populating metric components. So "re-express the metric as PhysicalDefinition/PhysicalEquation over physmath artifacts" fails the moment the agent populates `MathematicalRefs`.

An agent executing Phase 5 literally hits a guaranteed MATH GAP mid-phase — exactly the outcome seven rounds of pinning were meant to prevent. **Pin (one paragraph, recommended text):**

> *Phase-1 `phys-lib/gr` implements: the six framework families populated **only with PhysMath-expressible content** (e.g., the Λ=0 and r>0 relations, coordinate/index artifacts, empty-`ConditionRefs` convention assumptions per D15); the disjoint GR-local algebra (D9) with the capabilities the fixture and framework validation require; and the D2 bridge. The GR computation pipeline (Christoffel/Ricci/vacuum/weak-field evaluation) is **not** re-implemented in Phase 1 — it operates on the disjoint gr-local algebra in `phys-gr` and its re-expression is F1-deferred. §7's "formula loops re-expressed on phys-math" is corrected to "re-expressible fragments project to phys-math; the pipeline itself is F1-deferred." `solution/` (preserved) remains the sole executable home of the full GR derivation. Phase-5 "ported GR vectors" cover framework-artifact serialization, GR-local algebra behavior on copied vectors, and projection success/failure — not pipeline results.*

One paragraph; converts a guaranteed stop into a bounded deliverable.

---

## 3. One-line pins

- **P-A — D11 "spec-authorized generic ops" vs. §6.3 operation lock.** §6.3 permits exactly the seven rules; tensor arithmetic (contract/raise/lower) is *not* spec-authorized for Phase-1 phys-math. As written, D11/§7 ("generic validation/ops") could be read as authorizing tensor ops in phys-math. Pin: *Phase-1 phys-math Tensor = construction, validation, canonical serialization only; no tensor operations* (gr-local computation over in-memory types remains gr-side, per the §2 pin).
- **P-C — D15 authority wording.** "Extension requires a spec change" — the Category vocabulary is framework-owned per §14.2 ("no universal vocabulary beyond the package contract"); extension is a *phys-gr package-contract* decision recorded at plan level, not a spec amendment. One clause; prevents both agent-whim extensions and an unnecessary spec-touch procedure.
- **P-E — condition materialization.** D2's bridge and §5.7 require rule-generated conditions to become durable `physmath/condition/1` artifacts, while Phase 3's TransformResult carries a "supplied set." Pin: *the executor never creates durable artifacts; conditions are materialized by the artifact author (fixture/framework code) before referencing; `TransformResult.ConditionRefs` reference pre-materialized conditions* (this is also what makes Pow-0 fail-closed coherent).
- **P-F — core-emptiness test vs. record template.** If the §6 record template is a Go type in `package core`, the Phase-4 "core-empty assertion" needs an exemption. Pin: the template is documentation, not an exported core symbol.
- **P-G — gate [18] vs. §6.** Gate [18] demands records "for every §7 row"; §6 requires records for *promotions* (and boundary names for SPLITs, which §7 already carries). Harmonize: full records for `PHYS-LIB/CORE` candidates; classification + named boundary for SPLIT; classification suffices for the rest.
- **P-H — inventory hygiene.** The plan file lives under `physmath/plans/` while §2/§4 declare `phys-math` "exists empty" — reconcile the directory spelling (legacy `physmath` vs. module `phys-math`) and the emptiness claim ("empty of Go code; contains plans/"). Also the status header ("closed for implementation planning") describes a document that *is* the plan — retitle "closed for implementation execution."

---

## 4. Rejection endorsements (the plan's best adversarial moves)

- **R1** — the chain-rule `Derivative(f,i)` node is correctly refused: it would violate §7.1's closed node set and §6.3's lock; analytic derivative substitution via APPLY-001 body expansion or gr-local rewrites is the conforming route.
- **R3** — colon-joined rule IDs demoted to transport/error-key form; canonical identity stays the `(ProducerNamespace, RuleID)` pair with `ErrUnknownRuleID` fail-closed. Kills a real canonical-bytes hazard.
- **R4** — adopting the *literal* §5.1 conformance reading (six families, minimal content) over the convenient subset reading is the right call: it makes fixture2 a true conformance instrument and pre-empts the "subset suffices" drift.
- **R5** — honestly recording that the plan is *stricter* than the spec's permission (phys-* → Phys "when required") rather than silently exercising or silently banning it.
- **D16** — the semantic-vs-substring vocabulary distinction is load-bearing: a naive substring ban on "Function"/"Dimension" would flag phys-math's own API. Naming the mathematician test as review-criterion and the scans as CI gates resolves the enforceability question correctly.

---

## 5. Verified correct (no action)

Four-tier topology with single-module phys-lib and package-level core/specific split; D3/D4 phys-gr preservation enforced mechanically (gate [7] checksum oracle, gate [8] import scan — the strongest immutability instrument in the chain); Phase-0 checksum records for both frozen trees (G1/G3); the §7 map's per-row common-denominator/specific-difference columns with "move is never sufficient — deliberate re-expression"; Phase 3's inline restatement of the rule contracts (traversal policies, fail-closed Pow-0, root-only APPLY, sorted simultaneous SUBSTITUTE, summed-Order DIFFERENTIATE, R1 exclusion); §10's fixture data pinned to executable precision (displays, PID assignments, staged-call linkage `LHS_1.Args[0] == Step0 bytes`, final byte-equality gate, per-step golden refs); the 6-step graph verified by trace — both sides reach identical canonical bytes, Steps 2/5 produce the same `ArtifactRef` (making §13.3's gate a direct ref comparison), and the APPLY arity checks hold at every call site; fixture2 under `internal/` (compiler-enforced isolation, gate [17]); D14's honest single-implementation determinism scope; §12's explicit mechanical-vs-review gate classification; the quality gate's "empty core is a PASS"; and the handoff checklist's forbidden list, which matches every ruled-out item in the chain. Nothing in the plan touches PhysMath purity, the frozen Phys boundary, the identity layer, the closed rule registry, or the equivariance contract.

---

## Bottom line

**Ratify and proceed — after the Phase-5 scope pin.** plan9.2 does what a closure plan must: converts every prior finding into a numbered decision, rejects reviewer inventions with citations, enforces preservation mechanically, and leaves no D-level decision with two legitimate readings — except the one its own §7 map creates. Apply the §2 pin (Phase-1 GR = declarative framework + disjoint algebra + fixture; pipeline deferred; durable content = PhysMath-expressible only), the six one-liners, and the plan satisfies its own final handoff claim. At that point the equivariance fixture against a deliberately empty `phys-lib/core` is exactly what plan9.1 promised: the first executable demonstration that the common denominator lives in the lower tiers and the differences live in `gr` — proven, not asserted.