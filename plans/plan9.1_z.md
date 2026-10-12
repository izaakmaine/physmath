# Adversarial Review: `plan9.1` (phys-math + phys-lib sibling-workspace revision) + topology requirements

## Verdict

**The topology revision is correctly absorbed — the phys-math/phys-lib split is a better architecture (three classification destinations, idiomatic Go, the core/specific principle made structural), and nothing in it reopens a settled spec decision. But the revision introduces three blocker-class ambiguities of its own: the new `phys-lib/core` tier's durable-artifact status is unpinned (two readings, one of which silently violates the closed family registry), the Phase-1 GR re-expression scope is undecided (the plan's own sections point both ways, and the choice determines frozen-Phys imports), and the plan contradicts itself on phys-gr imports (shims allowed vs. lint-forbidden).** All three are one-paragraph pins. Plus the usual residue: namespace constants under the module renames, the fixture2 conformance reading, and two carried items. **No redesign; ratify the pins and Phase 0 starts.**

---

## 1. Topology-revision absorption audit

| Requirement / Q&A decision | plan9.1 disposition | Verdict |
|---|---|---|
| PhysMath under `github.com/PithomLabs/phys-math`, pure math | §5 boundary (absolute), §9 map destinations | 🟢 (one namespace pin, P-4) |
| phys-* under `github.com/PithomLabs/phys-lib`, core/specific split | §3 graph, §6/§7 boundaries, D3 | 🟢 |
| Single phys-lib module, packages not sub-modules | §2/§3 (core, gr, internal/conformance/fixture2) | 🟢 — matches the Q&A's Go-guidance reasoning |
| `phys-gr` = refactoring laboratory, preserved byte-untouched | D2, §14 | 🟢 (but see P3) |
| core/specific protocol per complexity principles | §8 record + eligibility questions + default-stays-in-gr | 🟢 — the strongest governance addition; see P-6 for one gap |
| Dependency partial order + prohibitions | §3, CI-vettable list | 🟢 (see P-5 for the BaseFrameworkRefs clause) |
| Empty core is conforming; promotion by evidence | §6, D4, §17 | 🟢 (see P1 — with one unstated consequence) |
| Equivariance owned by phys-lib/gr; fixture2 test-only | §11, §12, D5 | 🟢 |
| Many-frameworks strengthened via core | Deferred correctly (core empty in Phase 1; fixture2 is the Phase-1 mechanism) | 🟢 |

The recursion is genuinely implemented, not just restated: `phys-artifact` (mechanical) → `phys-math` (mathematical common denominator) → `phys-lib/core` (physical common denominator) → `phys-lib/gr` (specific) is the complexity rule made into a dependency graph, with the §8 record as the promotion gate. This is the right shape.

---

## 2. Blockers

### 🔴 P1 — `phys-lib/core`'s durable-artifact status is unpinned, and one reading violates the closed family registry

The spec's durable-family registry is closed: 16 PhysMath families + exactly six per *conforming framework* (which requires a FrameworkArtifact). `phys-lib/core` is deliberately **not** a framework (it is theory-agnostic; publishing a FrameworkArtifact would make it one). So: can a promoted core abstraction (say, a generic chart or a replay-chain helper) own durable artifacts? Two readings: (a) yes, under some namespace — but no legal family exists for it, so this reading forces either an unregistered family (§5.8 violation) or a spec amendment mid-Phase; (b) no — core is in-memory only. The plan never says. An agent promoting the first candidate hits this at the serialization step. **Pin:** *`phys-lib/core` publishes no durable artifact families, no ArtifactRef-bearing schemas, and no FrameworkArtifact in Phase 1; core code operates on PhysMath types in-memory. Durable representation of a promoted abstraction requires either a versioned spec amendment (new family) or placement inside a framework that owns it.* One paragraph; it also usefully constrains promotion (only in-memory abstractions are even eligible), sharpening the §8 gate.

### 🔴 P2 — The Phase-1 GR re-expression scope is undecided; the choice determines frozen-Phys imports and deliverable size

The plan points both ways: §9's map row says `solution/ + replay GR semantics` are "**re-expressed in phys-lib/gr** — REPLACE (re-implement, not move)… new protocol pkgs… Ph5–6," while §6 says "What remains GR-specific: `solution/` package **verbatim**" and Phase 6 covers only the *equivariance* package. Two legitimate implementations: (a) phys-lib/gr delivers framework 6 families + GR-local algebra + equivariance fixture, with the Schwarzschild protocol remaining executable **only in phys-gr** (canonical reference); (b) additionally a re-implemented Schwarzschild protocol package in phys-lib/gr — which drags in the full GR-local algebra port (trig/UF/order-2/trigReduce/ZeroTest), a second 15-key manifest instantiation, the M1–M8 harness, and — because `solution/`'s SessionCorrespondence used frozen Phys — the **first frozen-Phys import decision** for the new modules. The agent cannot size Phase 5 or know whether `phys-lib` imports `phys`. **Pin (recommend (a)):** *Phase-1 phys-lib/gr deliverable = FrameworkArtifact + six families + GR-local theory-local algebra + equivariance fixture. The Schwarzschild protocol's canonical executable home remains `phys-gr/solution/` (preserved, untouched); its re-expression into phys-lib is a future, separately scoped phase.* Under (a): **Phase-1 new modules have zero frozen-Phys imports and zero `phys`-namespace artifact references** — which also resolves the wiring question below and honors the chain's no-frozen-namespace-references invariant. If the architect wants (b), it must be scoped explicitly (session-correspondence → Phys import; manifest instantiation; harness port) before Phase 5.

### 🔴 P3 — §14 and §15 contradict each other on phys-gr imports

§14: "Temporary shims for `github.com/PithomLabs/phys-gr` imports allowed only where technically useful." §15: "Workspace drift (new modules depending on phys-gr) → import-path lint in CI." A shim **is** a new-module import of phys-gr; and D2 (byte-untouched phys-gr) removes the old adapter mechanism that made shims possible (adapters previously lived *inside* phys-gr). As written, one section licenses what the other forbids. **Pin:** *Phase-1 new modules (phys-artifact, phys-math, phys-lib) MUST NOT import `github.com/PithomLabs/phys-gr` under any circumstance; validation against legacy behavior is via ported golden vectors (copied data), never via imports. The §14 shim clause is deleted; the §15 lint has no exception.* One sentence each.

---

## 3. Pins

**P-4 (🟠) — Namespace constants survive the module renames — state it.** The artifact namespace remains exactly `physmath` (it is baked into §8.3's ParameterID formula and all 16 family IDs), while the Go module is `github.com/PithomLabs/phys-math`; the GR framework namespace remains `phys-gr` (FrameworkArtifact `PackageNamespace = "phys-gr"`) while the Go package is `phys-lib/gr`; fixture2's namespace is pinned (e.g., `phys-fixture2`). Hyphenated module paths and hyphen-free namespaces are different identifier spaces — one paragraph preventing the "helpful harmonization" that would invalidate every persisted ParameterID (same class as the §8.3 framing lock the spec already mandates).

**P-5 (🟠) — `BaseFrameworkRefs` is a data-level dependency only.** §3's "lateral phys-lib/gr ↔ phys-lib/qm except via the spec's explicit BaseFrameworkRefs mechanism" can be misread as permitting lateral Go imports when a BaseFrameworkRef exists. Pin: BaseFrameworkRefs creates artifact-reference (data) dependencies only; **no Go import between framework packages exists in Phase 1 regardless of BaseFrameworkRefs**; a future base-package code dependency is a spec amendment.

**P-6 (🟠) — fixture2 vs. the spec's conformance definition.** Spec §5.1: a namespace "becomes a conforming framework namespace when its package publishes a valid FrameworkArtifact **and implements the six framework-owned families**." plan9.1's fixture2 publishes three. Pin the reading: *conformance requires a valid FrameworkArtifact plus implementation of the families the package actually uses; the six-family set is the permitted vocabulary, not a mandatory population* — otherwise the plan's own fixture fails the spec's literal conformance test.

**P-7 (🟠) — Frozen-Phys wiring mechanics.** §14's gate requires frozen Phys "referenced by version (not local path) in new modules" — likely unsatisfiable for an unpublished module (no proxy resolution without a tagged release), and moot under P2-(a) (zero Phase-1 imports). Pin: *workspace `use ./phys` wiring is the sanctioned resolution mechanism for any future new-module Phys import; the freeze record supplies the revision; version-pinned references apply only if/when modules are published.*

**P-8 (🟡, carried) — Schematic ParameterID values.** §11 defers to spec §16.5 ("schematic `g,x` per §16.5"), but the values determine every golden ArtifactRef and §16.5 is not visible in the packet. Pin in the plan if the spec is silent: *schematic symbols carry the ordinal-derived ParameterID of the parameter slot they occupy.*

**P-9 (🟡, carried) — GR framework-artifact mathematical scope.** Under B2-A: convention-type assumptions (`-+++`, MTW) and non-expressible regularity (`sin≠0`) publish **empty `ConditionRefs`** with package-owned identity-bearing `Description`; only PhysMath-expressible predicates (`r > 0`, `Λ = 0`) carry relations; unknown-function/derivative-atom content never becomes a PhysMath `Function` (§8.6 would reject it). One paragraph in Phase 5.

**P-10 (🟡) — Hygiene batch.** (a) Authority-label mismatch: the spec file is `..._v1.2.md` with header `Version: 1.4`, and the plan cites "v1.2"; cite once by Document ID + file and fix the label at the next spec touch. (b) Phase 4 is classification-record-only (consumers don't exist yet); no promotion occurs in Phase 1 — one sentence so the agent doesn't force the §8 gate early. (c) The "mathematician test" is a review criterion; the mechanical gates are the vocabulary/import tests — say which is CI-enforceable. (d) §9's `REPLACE` row and §6's "verbatim" row should cross-reference the P2 ruling once the scope is pinned.

---

## 4. B1/B2 — procedural close-out available

Both were ratified last round against visible spec text, and plan9.1's recommendations match: **B1-A** (framework-local trig/UF under the theory-local mechanism; equivariance uses no trig) is grounded in §2.2's visible authorization ("MAY define a local mathematical algebra… package-local… MUST NOT dynamically extend"); **B2-A** (framework-level assumptions; step conditions only where rule contracts demand) matches §12.5's generated-conditions-only rule plus §5.7's two-carrier structure. **D1 = B3-A** is already decided. The "awaiting architectural input" status can close: ratify A/A, record them as spec-referenced resolutions (not plan preferences), and Phase 0 is unblocked.

---

## 5. Verified correct (no action)

The four-tier topology with module-boundary enforcement and the CI-vettable prohibition list; phys-artifact's boundary test (vocabulary + import scan — mechanically checkable); phys-math's absolute boundary with the `(ProducerNamespace, RuleID)` lock carried; the §8 record format (component/consumers/theory-dependence/why-not-elsewhere) — the right evidence instrument, and "useful to us is insufficient" is the correct anti-dumping-ground rule; the §9 map's per-row common-denominator/specific-difference columns (the complexity principles made operational); port-fresh with phys-gr byte-untouched (stronger isolation than the prior adapter plan — at the cost of making P3's shim clause an orphan, which is exactly why it must go); protocol migration with schema-identical reuse and the armed PROTOCOL GAP rule; the 6-step equivariance slice with staged calls, linkage checks, forbidden-list, and the `ρ_X`/`ρ_Y` single-artifact consequence carried forward implicitly; fixture2 as test-only internal package; the risk register mapping each complexity failure mode to a concrete test; the quality gate; and the explicit deferral list. Nothing in the new topology touches PhysMath purity, the frozen Phys boundary, the identity layer, the rule registry, or the equivariance contract — all carried intact.

---

## Bottom line

**GO — conditional on three one-paragraph pins.** The phys-math/phys-lib topology is the correct materialization of the recursion and is absorbed without reopening anything. P1 (core is in-memory-only), P2 (Phase-1 GR scope = framework + algebra + fixture; Schwarzschild stays canonical in phys-gr; zero frozen-Phys imports), and P3 (no phys-gr imports; shims deleted) are decisions the plan must make, not discoveries for the agent — each currently has two legitimate readings, which is precisely the §32.2-class condition this chain treats as blocking. Apply P1–P3, the eight pins, ratify B1-A/B2-A as spec-grounded resolutions, and the plan is execution-ready: Phase 0 can verify inventory, Phase 1 can create phys-artifact, and the first real test of the three-tier common-denominator architecture becomes the equivariance fixture running against a deliberately empty `phys-lib/core` — which, as the plan itself says, is a conforming outcome, not a gap.