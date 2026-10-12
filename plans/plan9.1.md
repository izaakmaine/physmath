# Revised Implementation Plan — `phys-math` + `phys-lib` (PM-REP-001 v1.2, sibling-workspace revision)

Supersedes the `physmath`/`phys-gr`-module layout of the prior plan. The PhysMath mathematics, artifact/identity model, 7-rule execution lock, and equivariance derivation content are unchanged; what changes is the **Go library topology, the refactor destination, and the core/specific promotion discipline**.

---

## 1. Authority and scope

Authoritative: **PM-REP-001 v1.2** (architecturally closed for Phase 1). This plan does not restate or redesign it. Scope is representation plus executable reference protocol only: `phys-artifact`, `phys-math`, `phys-lib` (core + gr), equivariance fixture, conformance suite. Deferred: TOE-Agent, Numerical/ComputationalContract, state machines, search/orchestration, trust/corpus/adjudication, paper workflow, embeddings, autoformalization, frozen-Phys projection, PhysicalContract ontology, generic NodePath selectors, sparse tensors, deferred-sort execution (§6.2), production `qm`/`bohm`/`cst`/`cdt` packages.

Locked architectural facts: (a) sibling workspace, not nesting; (b) `phys-gr` preserved byte-untouched as historical artifact; (c) `phys-lib/core` starts empty; (d) `phys-lib/gr` owns the equivariance fixture, no `phys-equiv`; (e) second-framework fixture is a test-only in-module package; (f) GR protocol files are reused, never reinvented.

## 2. Repository/current-state inventory (verified)

```text
/home/chaschel/Documents/go/            <- workspace root
  go.work                                (go 1.25.7; use ./phys ./phys-gr)
  phys/                                  (frozen; module github.com/PithomLabs/phys; DO NOT MODIFY)
  phys-gr/                               (migration source; module github.com/PithomLabs/phys-gr;
                                          go 1.25.7; replace phys => ../phys [transitional])
  phys-math/                             (exists, EMPTY — greenfield)
  phys-lib/                              (exists, EMPTY — greenfield)
  phys-artifact/                         (DOES NOT EXIST — create in Phase 1)
```

`phys-gr` contents (inspected, preserved): `symbolic/` (9-kind GRExpr + Normalize/Diff/Subst/ZeroTest/Truncate), `coordinates/` (spherical-static), `tensor/` (dense 4D), `metric/` (ansatz), `connection/` (Christoffel), `curvature/` (MTW/-+++/Lambda=0), `vacuum/` (16 assertions), `limit/` (weak-field), `replay/` (GRStep chain + ReExecute), `solution/` (14-step Schwarzschild + 15-key `manifest.json` + strict `manifest_test.go` + 923-line `derivation_test.go` with M1–M8 harness), `AGENTS.md §8.1`, `evidence/`, `docs/`. Frozen `phys` import surface used: `core, ops, session, hypothesis`.

## 3. Target module graph

```text
workspace (/home/chaschel/Documents/go/)
  phys-artifact/   module github.com/PithomLabs/phys-artifact   (CREATE)
  phys-math/       module github.com/PithomLabs/phys-math       (exists empty)
  phys-lib/        module github.com/PithomLabs/phys-lib         (exists empty)
    core/          (starts EMPTY)
    gr/            (canonical GR library)
    internal/conformance/fixture2/   (test-only second framework)
  go.work          (extend: use ./phys-artifact ./phys-math ./phys-lib)
  phys/            (frozen, untouched)
  phys-gr/         (preserved, untouched; removed from go.work at completion)
```

Dependency direction (enforced by module + import discipline):

```text
phys-lib/gr  →  phys-lib/core  →  phys-math  →  phys-artifact
phys-lib/gr  →  phys-math | phys-artifact  (direct use permitted, never reverse)
```

Prohibited (CI-vettable): `phys-artifact → {phys-math, phys-lib/*}`; `phys-math → phys-lib/*`; `phys-lib/core → phys-lib/{gr,qm,…}`; lateral `phys-lib/gr ↔ phys-lib/qm` except via the spec's explicit `BaseFrameworkRefs` mechanism. `go.work` is local wiring only; no committed `replace` directives in the new modules. Legacy `replace => ../phys` removal gate (§14): remove only after workspace wiring resolves frozen Phys correctly **and** the final graph contains no path through `phys-gr`.

## 4. `phys-artifact` boundary

Strictly mechanical: `ArtifactRef` (normative field order), §4.3 digest, §4.4 embedded encoding, §4.5 primitives (Set sort+dedupe, Optional/Sequence framing), §4.6 BigInt (reject −0/leading zeros), §5.1–5.4 grammars. No math vocabulary, no physics vocabulary, no protocol semantics, no generic utilities. Boundary test: a test asserting the module's public API surface contains no identifiers matching math/physics vocabularies (`Expression`, `Function`, `Framework`, `Assumption`, `Theorem`, `Derivation`, `Tensor`, …) and imports only stdlib — fails the build on violation.

## 5. `phys-math` boundary (absolute)

Pure mathematical objects and representation: Expression 10-node union (§7.1), Symbol/ParameterID/ConstantID (§§7.4, 8–9), Function/Map/Tensor/Index/Collection/OperatorApplication (§10), Transformation vs DerivationStep lifecycle (§§10.8–10.9), Equation/EquationSet/Definition/MathCondition (§11), 7-rule executor (§17), Derivation/Theorem + replay (§§12–13). MUST NOT contain: physical objects/assumptions/interpretations, theory names, GR/QM semantics, physical dimensions-as-meaning, conventions, trust/corpus semantics, theory-specific transforms. Enforcement: the mathematician test — every exported API must be explainable without naming a physical theory — plus a vocabulary test rejecting physics identifiers, and the `(ProducerNamespace,RuleID)` 7-rule lock (§§17.11, 24.5). Canonical bytes live here; transport JSON is debug-only (§20.2).

## 6. `phys-lib/core` boundary (starts empty)

Theory-agnostic physical common denominator — for abstractions whose meaning is independent of any particular theory. Candidate destinations are **not** pre-seeded (no chart, no replay-chain seeding). Promotion requires the §8 record with demonstrated multi-framework stability; "useful to us" is insufficient. If `core/` is still empty (or near-empty) at Phase-1 completion, that is a conforming outcome, not a gap. Forbid `common`/`util`/`shared`/catch-all packages. Sub-packaging inside `core/` follows Go guidance (focused, client-facing abstractions with meaningful names), derived from actual consumer analysis, never invented upfront.

## 7. `phys-lib/<specific>` boundary

`phys-lib/gr` owns all GR semantics: ansatz, chart, MTW sign, −+++ signature, Λ=0, regularity assumptions (as explicit `PhysicalAssumption` artifacts, not code comments), weak-field reduction, GR-local math (Sin/Cos, UnknownFunction-shape needs, order-2 caps, trigReduce, regime ZeroTest — via the §15 theory-local mechanism, never as PhysMath nodes). `qm`/`bohm`/`cst`/`cdt` are **not** built in Phase 1 (deferred). The test-only `internal/conformance/fixture2` asserts its own distinct `phys-*` test namespace, publishes a minimal FrameworkArtifact, and must never become production API or gain a dependency from `core/` or `gr/`.

## 8. Core/specific classification protocol

For every significant `phys-gr` abstraction, classify exactly one of `PHYS-MATH | PHYS-LIB/CORE | PHYS-LIB/GR | SPLIT | REPLACE | DELETE | DEFER`. For `SPLIT`, name the exact interface (types, function signatures, ownership of each side). For promotion to `phys-lib/core`, record:

```text
Component: / Current location: / Candidate destination:
Why it is theory-agnostic: / Known/current consumers: / Potential future consumers:
Theory-specific assumptions: / Stable interface: / Dependencies:
Why it does not belong in phys-math: / Why it does not belong in phys-lib/gr:
```

Eligibility questions: meaning independent of theory? stable across frameworks? no theory-specific assumptions in API? reduces duplication / establishes reusable interface? documentable without naming a theory? Any theory-dependence → specific; partial → split; proven independence → core. Guiding application per decision: Coupling (math vs physics; core-physics vs theory-physics; identity vs packaging; representation vs execution → separate), Cohesion (smallest stable unifying interface, no convenience boundaries), Core/specific (smallest true common denominator downward, differences outward).

## 9. Full `phys-gr` refactoring map (source → new home; `phys-gr` itself unchanged)

| CURRENT (phys-gr/) | TARGET | CLASS | COMMON DENOMINATOR | SPECIFIC DIFFERENCE | TARGET INTERFACE | DEPS | TEST IMPACT | SEQ |
|---|---|---|---|---|---|---|---|---|
| `symbolic/expr.go` tree shape, canonical-JSON discipline, Equal-by-bytes | `phys-math` AST+codec | SPLIT | closed tree mechanics, recursive tag encoding, byte-equality | 9-kind set itself (Sin/Cos/UF/Pow-int), `MaxDerivativeOrder`, `ValidSymbolName` bound | `phys-math` constructors/validators/codec; `phys-lib/gr` local algebra (§15) | phys-artifact | new golden bytes (incompatible by design) | Ph2 |
| `symbolic/normalize.go` flatten/sort/fold, fixpoint, idempotence | `phys-math` EXPR-CANON/SIMPLIFY/RAT-NORMALIZE | SPLIT | commutative normalization, all-Rational folding, §7.6 arity rules | `trigReduce`, regime-ZeroTest, `SplitLinear` | rule executor + condition threading | identity, codec | §§25.8/25.9/25.27 idempotence vectors | Ph3 |
| `symbolic/differentiate.go` linearity/Leibniz/Pow/Sin/Cos | `phys-math` DIFFERENTIATE-001 | SPLIT | closed derivative rules | order≤2 cap, Symbol-name-only var, no-chain rule | BoundParameterRef + Order params | identity | §25.14 negatives | Ph3 |
| `symbolic/subst.go` structural recursion | `phys-math` SUBSTITUTE/APPLY-001 | SPLIT | simultaneous ParameterID substitution, sorted inputs/entries | zero-order-UF-only targets, no UF-arg rewrite, single-arg f(r) | declaring-Function inputs, staged-call discipline | identity, codec | §§25.12/25.13/25.28 | Ph3 |
| `symbolic/truncate.go`, `limit/`, `SplitLinear` | `phys-lib/gr` (rename `WeakFieldTruncate`) | PHYS-LIB/GR | — | single-rule Schwarzschild O(eps) engine | framework-local function, explicit mapping doc | framework | legacy weakfield tests migrate with it | Ph5 |
| `tensor/` offsets/validation/arithmetic shape | `phys-math` Tensor/Index/Collection | SPLIT | rank/slots/row-major, SET-vs-SEQUENCE, dense checks | `Dim=4`, `Domain=spacetime`, ChartID-equality ops, symmetric-metric assume, re-Normalize | parametric core types; chart binding in `gr` | codec | tensor goldens + negatives | Ph2–3 |
| `metric/connection/curvature/vacuum/coordinates` | `phys-lib/gr` framework artifacts | PHYS-LIB/GR | formula loops (re-implemented against phys-math) | ansatz, MTW, −+++, Λ=0, chart, regularity, contraction choice | 6 families + assumption refs | phys-math, core (if any) | GR regression via ported vectors | Ph5 |
| `replay/` chain+dispatch shape | `phys-math` derivation replay | SPLIT | hash-chain, VerifyChain, ReExecute dispatch pattern | `gr:*` namespace, `curvature`/`vacuum_check` as ops | Transformation vs DerivationStep types | rules, codec | replay → derivation-replay tests §25.17 | Ph3 |
| `coordinates` chart data | `phys-lib/gr` (+ `collection/1` SpaceRef) | PHYS-LIB/GR | — | spherical-static chart | framework data | framework | conformance vectors | Ph5 |
| `solution/` + `replay` GR semantics | preserved in `phys-gr` (reference); re-expressed in `phys-lib/gr` | REPLACE (re-implement, not move) | derivation pattern | all Schwarzschild content | new protocol pkgs | — | original suite stays green; new suite additive | Ph5–6 |
| `evidence/`, `docs/` | — | DEFER | — | — | — | — | none | — |
| chart ops, generic replay helpers | `phys-lib/core` | DEFER-DECISION (default: stays in `gr`) | only if §8 record proves theory-independence | — | — | — | — | Ph4 |

Default for anything ambiguous: **keep it in `phys-lib/gr`**. Promotion to `core` or `phys-math` requires the §8 record; absence of evidence keeps the abstraction specific.

## 10. GR protocol migration strategy

Reuse the actual files as authority: `solution/manifest.json` (15-key schema — copy exactly, change values), `manifest_test.go` (go:embed, strict typed decode, `DisallowUnknownFields`, no reflection, `runtime.Caller` siblings, prose-is-not-math, artifact-presence), `derivation_test.go` (ordered `expect(op,params)` consumption, firewalls, M1–M8-style mutation harness, independence test), `replay/` chain semantics, `AGENTS.md §8.1` 4-artifact contract. Packaging refactor (decided, not a blocker): new fixture uses the canonical v1.2 nested layout `<pkg>/{AGENTS.md,manifest.json,protocol/derivation_test.go}` with **identical schema/loading/execution semantics**; record the flat→nested equivalence in the fixture `AGENTS.md`. `manifest.json` stays metadata-only (never a premise). If any equivariance need cannot be expressed without a new semantic manifest field → stop, report PROTOCOL GAP, do not improvise.

## 11. Equivariance implementation plan (owned by `phys-lib/gr`)

Functions `f(v)=2v` (1 param, display `v`), `rho_X/rho_Y(g,v)=g·v` (displays `g,v`); schematic `g,x` per §16.5. Staged Calls `LHS_0/LHS_1/RHS_0/RHS_1` with byte-linkage checks. Exact 6 steps: 0 APPLY inner-`rho_X`→`g·x`; 1 APPLY staged-`LHS_1`→`2·(g·x)`; 2 CANON→`2·g·x`; 3 APPLY inner-`f`→`2·x`; 4 APPLY staged-`RHS_1`→`g·(2·x)`; 5 CANON→`2·g·x`. Relation `EQ||canon(LHS)||canon(RHS)`, Equation wrapper, Theorem + PhysicalTheorem (≥1 assumption + ≥1 definition/interpretation + FrameworkArtifact). Pin golden ArtifactRefs per step + negatives. Forbidden: Map-for-f, durable FunctionApplication for LHS/RHS, distributivity, GroupAction sort, trig. Deterministic replay across implementations (§25.34).

## 12. Second-framework conformance plan

`phys-lib/internal/conformance/fixture2/`: test-only package asserting a distinct namespace (grammar-valid, e.g. `phys-fixture2`), minimal FrameworkArtifact + ≥1 assumption + ≥1 interpretation over the **same** equivariance PhysMath artifacts with **different** physical meaning. Asserts: PhysMath identity unchanged, framework artifacts distinct, no global winner, unrelated-assumption references rejected, cyclic/self framework deps rejected (§§21.1–21.3, 25.22–25.23). Never imported by production code; never versioned as product.

## 13. Test and acceptance matrix

Levels: unit → serialization (golden + framing-ambiguity) → identity/hashing (digest, alpha-rename, ordinal consistency, BoundParameterRef) → rule execution (arity/family/condition gates, fail-closed Pow-0, all-Rational folding) → derivation replay (ordering, same-run availability, condition threading) → phys-* conformance (§21 + 12 negatives + disagreement + base/specialized via fixture2) → protocol (artifact-presence, exact-manifest, firewall/mutation for both GR-reference and equivariance) → golden vectors (all families + 7 rule-params + 6-step pins) → cross-implementation determinism. Contamination-priority: t/k neutrality (§25.7), dynamic-AST-extension rejection (§25.24), PhysMath-vocabulary and `core`-purity tests (new: `core` public API must be explainable without naming a theory), GR-leakage tests (convention-dependent behavior without assumption refs fails).

## 14. Compatibility/migration strategy

Preserve: `phys-gr` repository byte-identical (regression baseline + migration source; its suite keeps passing independently). Port, don't move: new canonical code is written fresh in the sibling modules against the spec, validated against ported vectors. `solution/` behavior is the semantic anchor for the GR port. Temporary shims for `github.com/PithomLabs/phys-gr` imports allowed only where technically useful, never as architecture, never receiving new functionality. `replace => ../phys` removal gate: (1) `go.work` extended and resolving, (2) frozen Phys untouched and referenced by version (not local path) in new modules, (3) no active dependency path through `phys-gr`. Obsolete on arrival: regime-ZeroTest-as-math, generic weak-field names, `gr:*` flat ops, GRExpr-JSON-as-identity.

## 15. Complexity/risk register

Core-as-dumping-ground (promotion without evidence) → §8 record gate + empty-start + "default stays in gr" rule. GR leakage into `phys-math` (trig/UF/caps/dims) → §15 locality + vocabulary tests + B1 ruling. Physics leakage into `phys-math` (t/k roles, interpretation in canon) → mathematician test + §25.7. `core` theory-contamination (chart/replay smuggling GR assumptions) → eligibility questions + theory-name-vocabulary test on `core`. Serialization confusion (legacy GRExpr JSON vs canonical bytes) → codec separation, Equal-by-canonical-bytes. Protocol drift (flat vs nested) → equivalence note + schema-diff test. Staged-call ambiguity → pre-constructed Calls + linkage checks (§25.32). ParameterID cross-artifact collision → BoundParameterRef + duplicate-target rejection (§25.28). Workspace drift (new modules depending on `phys-gr`) → import-path lint in CI.

## 16. Architectural blocker log

```text
ARCHITECTURAL BLOCKER B1 — Expression node-set conflict (CARRIED OVER)
Decision needed: where do Sin/Cos (r²sin²θ) and A(r)/B(r)-shaped needs live?
Evidence: phys-gr/symbolic/expr.go vs §§7.1, 10.2, 16.4. Conflict: closed 10-node set
has no trig/UnknownFunction; Pow is arbitrary-Expression in spec, int-only in code.
Why insufficient: spec never assigns trig a home. Options: A. framework-local trig
under §15 in phys-lib/gr (equivariance needs none); B. future spec amendment.
Recommended: A. Impact: Phase 5 GR-port scope; legacy metric tests keep adapters.
```

```text
ARCHITECTURAL BLOCKER B2 — Regime non-vanishing placement (CARRIED OVER)
Decision needed: framework-level PhysicalAssumptions vs per-step MathConditions for
r,A,B,sinθ ≠ 0? Evidence: normalize.go ZeroTest + dependents vs §§12.5, 17.4, 17.10.
Options: A. framework-level assumptions; steps carry conditions only where rule
contracts demand (Pow-0). B. per-step conditions everywhere. Recommended: A.
Impact: Phase 5 validator strictness; equivariance slice unaffected (no nonzero rule).
```

```text
DECIDED D1 — Protocol layout: nested <pkg>/{AGENTS.md,manifest.json,protocol/derivation_test.go}
with GR-identical schema/loading/execution semantics + equivalence note.
DECIDED D2 — Sibling workspace; phys-gr preserved untouched. DECIDED D3 — phys-lib single
module; core/gr/etc. are packages. DECIDED D4 — core starts empty; promotion by evidence.
DECIDED D5 — fixture2 lives in phys-lib/internal/conformance/fixture2/, test-only.
```

## 17. Deferred work

Production `qm`/`bohm`/`cst`/`cdt` packages; sparse tensors; deferred-sort execution; TOE-Agent/Numerical/trust/provenance/state machines/search/embeddings/autoformalization; frozen-Phys projection; PhysicalContract ontology; generic NodePath selectors; any `core` promotion lacking a §8 record.

## 18. Final dependency-ordered execution sequence

```text
Phase 0  Repository/authority/protocol inventory (verify §2; confirm phys-artifact/ creation path;
         extend go.work; NO code yet; NO changes to phys-gr or phys/)
Phase 1  phys-artifact (CREATE module; ArtifactRef/digest/primitives/BigInt/grammars;
         boundary test; §25.1–25.3 vectors)
Phase 2  phys-math representation + identity (AST, 10-node codec, ParameterID/BoundParameterRef/
         ConstantID, Function/Map/Tensor/Index/Collection/OperatorApplication, target table §5.7)
Phase 3  phys-math rules + derivation (7-rule executor, TransformResult, Derivation/Theorem,
         replay determinism; split replay-chain shape here)
Phase 4  phys-lib/core candidate extraction (module + EMPTY core; classify per §8;
         promote ONLY with records; default stays in gr)
Phase 5  phys-lib/gr extraction + refactoring (framework 6 families, GR semantics port,
         GR-local math via §15, B1/B2 rulings applied)
Phase 6  GR protocol migration (equivariance pkg in nested layout; GR-schema reuse;
         PROTOCOL GAP rule armed)
Phase 7  Equivariance vertical slice (§11 exact 6 steps → theorem → goldens)
Phase 8  Second-framework conformance fixture (fixture2 + disagreement/closure tests)
Phase 9  Golden vectors / negatives / determinism (full §§20.5/21/25 suite)
Phase 10 Adapter cleanup + architecture conformance (remove shims, replace-removal gate,
         vocabulary/lint gates, quality-gate checklist green)
```

Quality gate (all must hold before coding is declared complete): PhysMath has no physical concepts; `core` has no theory-specific semantics (and every `core` item has a §8 record); `gr` owns GR semantics; no reverse dependency; core/specific decisions explicitly justified; no util/common dumping ground; `phys-gr` inspected and preserved untouched; GR protocol files reused with no provisional schema; equivariance owned by `phys-lib/gr`; second-framework conformance included; frozen Phys untouched; `replace` removal gated; unsupported GR math kept out of PhysMath; formalization remains executable; blockers B1/B2 surfaced, not guessed.

No production code is written by this plan. Awaiting architectural input on B1/B2 (or approval to proceed under the recommendations) before Phase 0 execution.
