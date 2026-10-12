# IMPLEMENTATION PLAN — PhysMath + phys-* v1.2 (PM-REP-001 v1.4) from existing `phys-gr`

> Authority: `plans/PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md` (v1.4, architecturally closed).
> Substrate inspected: `/home/chaschel/Documents/go/phys-gr` (module `github.com/PithomLabs/phys-gr`, go 1.25.7, depends on frozen `github.com/PithomLabs/phys` via `replace => ../phys`).
> Decisions adopted from architect input: (1) multi-module monorepo + `go.work`, DAG `phys-gr -> physmath -> phys-artifact`, no committed `replace`; (2) `phys-gr` is the sole physical owner of the Phase-1 equivariance fixture, no synthetic `phys-equiv`; (3) GR protocol authority is mandatory — reuse `solution/manifest.json` schema/loading and `derivation_test.go` shape exactly, never invent a provisional schema.
> Note: `/home/chaschel/Documents/go/physmath` contains only `plans/` (no code). All implementation work happens in `/home/chaschel/Documents/go/phys-gr`, which becomes the monorepo root.

---

## 1. Current-state assessment (reality, not hypothesis)

### 1.1 Module and packages

```text
module github.com/PithomLabs/phys-gr
  symbolic/     closed 9-kind GRExpr algebra (Symbol/Rational/Add/Mul/Neg/Pow-int/Sin/Cos/UnknownFunction)
  coordinates/  chart ID=spherical-static [t r theta phi]
  tensor/       dense 4D row-major Tensor, Dim=4, Domain=spacetime
  metric/       ansatz diag(-A(r),B(r),r^2,r^2 sin^2theta) + inverse
  connection/   Christoffel (Levi-Civita, not a Tensor by type — good)
  curvature/    Riemann/Ricci/Scalar/Einstein, MTW/-+++/Lambda=0 baked in
  vacuum/       16x R_uv=0 assertions (expr excluded from canonical form)
  limit/        weak-field Phi=-GM/r extraction (Truncate+SplitLinear)
  replay/       GRStep hash chain + ReExecute deterministic dispatch (generic chain, GR op namespace)
  solution/     14-step Schwarzschild Primary + Secondary + FlatCheck + WeakField + SessionCorrespondence
  evidence/     docs only (failure_log, kernel_contact, growth_records)
  docs/         protocol_derivation.md
frozen ../phys (NOT modified): core, ops, session, hypothesis, relativity, mechanics
```

### 1.2 Protocol precedent (found — no PROTOCOL GAP on existence)

`solution/` implements the GR precedent the spec mandates:

| File | Role | Mechanism (must reuse) |
|---|---|---|
| `solution/README.md` | human orientation only | exists-check, never parsed |
| `solution/manifest.json` | machine-readable derivation metadata, **never a premise** | 15 top keys (`schema_version, artifact_type, derivation_id, framework_id, name, entry_points{primary,secondary}, regime, coordinates, conventions{metric_signature,curvature_convention,cosmological_constant}, premises[{id,statement}], assumptions, expected_result{A,B,description}, verification{primary_derivation_trace,primary_stages,secondary_verification}, limitations[], anomalies[], falsification_conditions[]`); pinned values `-+++/MTW/Lambda=0/14 stages/GRStep` |
| `solution/manifest_test.go` | package-contract + manifest validation | `//go:embed manifest.json`, strict typed decode with `DisallowUnknownFields`, no `map[string]any`/reflection/registries, `runtime.Caller`-based sibling resolution, prose-is-not-math check, artifact-presence check |
| `solution/derivation_test.go` (923 lines) | executable derivation + anti-shortcut verification | runs `Primary() -> replay.VerifyChainReplay -> verifyPrimaryTrace`; ordered `expect(op,params)` consumption; firewalls (no `mu` in PRIMARY, `k2` first at stage 12, manifest never imported); M1–M8 mutation harness; PRIMARY/SECONDARY independence test |
| `replay/` | GR-local replay chain | `Step{StepID gr-step-%06d, Operation, InputCanonicals/Hashes, ParamsCanonical, OutputCanonical/Hash, KernelBridgeHash, Prev/CurrentHash}`; closed `gr:*` + `kernel:differentiate/compare` ops |
| `AGENTS.md 8.1` | workflow authority | 4-artifact contract + formal/verification/interpretation/truth distinction |

Structural delta to v1.2 s1.4/s16.2 (flagged in s10, not silently resolved): GR precedent is `solution/{README,manifest.json,manifest_test.go,derivation_test.go}` (flat), while v1.2 prescribes `<pkg>/{AGENTS.md,manifest.json,protocol/derivation_test.go}` (nested `protocol/`). Adaptation must preserve schema/loading/execution semantics and be an explicit, minimal relocation — see Blocker B3.

### 1.3 Math vs physics split (evidence for core/specific)

| Component | Generic math | GR-specific leakage (must move out of core) |
|---|---|---|
| `symbolic/expr.go` 9-kind tree | tree shape, canonical JSON, Equal-by-bytes | `MaxDerivativeOrder=2`, `UnknownFunction` tailored to A(r)/B(r), `ValidSymbolName` bound, "GR-local" labelling |
| `symbolic/normalize.go` | rational folding, flatten, like-term grouping, sort | `trigReduce` (only Sin^2+Cos^2->1 for spherical terms), `ZeroTest` NONZERO-for-symbols (encodes r!=0,A!=0,B!=0,sin!=0 regime), `SplitLinear` (M-1 weak-field only) |
| `symbolic/subst.go, differentiate.go` | structural subst, linearity/Leibniz/Pow/Sin/Cos | zero-order-UF-only targets, no UF-arg rewrite (ansatz invariant), single-arg f(r) assumption, order-2 cap, Symbol-name-only diff var |
| `symbolic/truncate.go` | — (honest header: not general) | entirely weak-field engine (`Pow(1+u,-1)->1-u`, eps-degree-1 only) |
| `tensor/` (Dim=4, single chart) | dense array, offsets, add/contract/raise/lower | `Dim=4`, `Domain=spacetime`, `Zeros->spherical-static`, identical-ChartID-only ops (no covariance), symmetric-metric assumption in `moveIndex`, re-Normalize (regime leak) |
| `metric.Build()` | diagonal inverse check | one ansatz constructor encodes signature+chart+topology+Lambda=0+regularity all at once |
| `connection`, `curvature` | formula loops | torsion-free/metric-compatible unnamed; MTW sign, `R^r_mrn` contraction, no Lambda term — presented as THE Riemann/Ricci/Einstein |
| `solution/schwarzschild.go` | — | pure GR script (14 steps, NonZeroClasses==9, recorded `implication_AB/rA` as byte-identical assertions, not a prover) |
| `replay/` GRStep chain | hash chain, VerifyChain, ReExecute dispatch shape | flat `gr:*` namespace mixing math ops with physics inferences (`curvature`, `vacuum_check`) |

Frozen `phys` boundary: `phys/core`, `phys/ops`, `phys/session`, `phys/hypothesis` are imported by `symbolic` (bridge), `replay`, `solution`. Phase-1 MUST NOT modify `../phys`, MUST NOT add `phys-* -> phys` imports beyond the existing public API surface, and per spec s20.3 no Phase-1 artifact may reference the frozen `phys` namespace.

---

## 2. Target architecture mapping (v1.2 → repo)

```text
/phys-gr (monorepo root, go.work)
  phys-artifact/      module github.com/PithomLabs/phys-artifact
      ArtifactRef{Namespace,SchemaVersion,ArtifactType,Digest} (s4.2, field order normative)
      Digest = SHA256(Header||P), Header per s4.3; embedded-ref encoding s4.4
      primitives s4.5 (String/Bytes/UInt8-64/Bool/Digest32/Optional/Sequence/Set/Enum/TaggedUnion)
      BigInt s4.6 (no negative zero, minimal magnitude); NO Float64Bits in identity (s4.7)
      record/collection rules ss4.8-4.9; BoundParameterRef encoding s4.10
      owns NOTHING semantic (no physics, no math, no trust) — s3
  physmath/           module github.com/PithomLabs/physmath → imports phys-artifact only
      Expression closed 10-node union s7.1 (Symbol/Rational/Add/Mul/Neg/Pow/Sqrt/Call/Relation/BranchSet)
      Symbol/ParameterID/ConstantID ss7.4,8-9; Function/Map/Tensor/Index/Collection/OperatorApplication ss10.1-10.7
      Transformation (durable, standalone) vs DerivationStep (inline) ss10.8-10.9
      Equation/EquationSet/Definition/MathCondition s11; 7 closed rules s11.5/s17
      Derivation/Theorem ss12-13; TransformResult/OperationInvocation s17.1
      canonical codecs + registry + executor + validators; owns NO physics
  phys-gr/ (existing dirs, refactored)  module github.com/PithomLabs/phys-gr → imports physmath, phys-artifact (+ frozen phys public API only)
      FrameworkArtifact + 5 families (assumption/definition/equation/theorem/interpretation) s14
      GR ansatz/conventions/chart/curvature/vacuum/limit/solution as framework-owned semantics
      local algebra ONLY via s15 (package-local, explicit mapping, never extends PhysMath AST/registry)
      protocol fixtures (GR preserved + equivariance added) per s1.4/s16.2/s21.4
```

Dependency enforcement is by **module boundary** (architect's decision): `phys-gr -> physmath -> phys-artifact`, no reverse edges; framework-to-framework only via `BaseFrameworkRefs` (ss14.8-14.9); graph acyclic (s3). `go.work` wires the three modules locally; no committed `replace` directives.

Family registry (s5.6): 16 PhysMath families + per-framework 6 (`<ns>/{framework,assumption,definition,equation,theorem,interpretation}/1`). Target-family table s5.7 is enforced by validators, not by convention.

---

## 3. Package/module plan (owns / does-not-own / depends)

| Module.package | Purpose | Owns | Does NOT own | Depends on | Depended on by |
|---|---|---|---|---|---|
| `phys-artifact` (root) | mechanical identity + codecs | ArtifactRef, digest, primitives, BigInt, Set/Sequence sort, grammar checks (ss5.1-5.4) | any math/physics semantics | stdlib only | physmath, phys-gr |
| `physmath/ast` | in-memory IR + constructors | Expression nodes, Function/Map/Tensor/Index/Collection/OperatorApplication types, validation | canonical bytes, execution | phys-artifact | physmath/codec, rules, derivation |
| `physmath/codec` | canonical serialization | field-order encoders per family, golden vectors | transport JSON (debug only) | ast, phys-artifact | tests, phys-gr (via physmath API) |
| `physmath/identity` | ParameterID/BoundParameterRef/ConstantID | DeriveParameterID (s8.3), ordinal consistency, capture-safety | physical roles (t/k) | ast, phys-artifact | rules, derivation |
| `physmath/rules` | 7-rule executor | rule table s17.2, traversal policies ss17.3-17.9, TransformResult, condition threading s17.10 | framework-local rules (reject as data-only) | ast, codec, identity | derivation, protocol tests |
| `physmath/derivation` | Derivation/Theorem artifacts + replay | step ordering, same-run availability s12.6, conclusion-side byte identity ss13.3/16.10-11 | truth/provenance | rules, codec | phys-gr fixtures |
| `phys-gr/framework` (new) | GR as conforming phys-* | FrameworkArtifact, 6 families, BaseFrameworkRefs (empty in Phase 1), conformance vectors s14.11 | math identity | physmath, phys-artifact | phys-gr/* |
| `phys-gr/{metric,connection,curvature,vacuum,limit,coordinates}` | GR semantics (refactored) | ansatz, conventions, chart, MTW Ricci, Lambda=0 Einstein, weak-field | canonical codecs (delegate to physmath) | framework, physmath | solution, fixtures |
| `phys-gr/symbolic,tensor` (shrinking) | legacy compat only | thin adapters over physmath during migration | new semantics (frozen after cutover) | physmath | legacy tests until deleted |
| `phys-gr/solution` (preserved) | GR-8 Schwarzschild protocol | existing 14-step derivation + manifest/tests untouched | equivariance content | existing | CI |
| `phys-gr/equivariance/` (new fixture pkg) | v1.2 s16 vertical slice | fixture functions f/rho_X/rho_Y, 6-step derivation, PhysicalTheorem, protocol test | second framework (deferred) | framework, physmath | protocol test |

Kernel-growth guard: `phys-artifact` stays <= ArtifactRef+codecs+digest+grammars; any PR adding physics/math vocabulary there is rejected. `physmath` rejects unknown `(ProducerNamespace,RuleID)` at deserialization and execution (ss17.11, 24.5).

---

## 4. phys-gr refactoring map

| CURRENT | TARGET | ACTION | RATIONALE | DEPENDENCIES | RISKS | TEST IMPACT |
|---|---|---|---|---|---|---|
| `symbolic/expr.go` tree + canonical JSON | `physmath/ast+codec` (ss7,10) | **SPLIT**: generic tree shape -> physmath; `Sin/Cos/UnknownFunction/MaxOrder` -> phys-gr local algebra (s15) or DELETE | core/specific: only true common denominator moves to core | phys-artifact | node-set mismatch (Blocker B1) | golden vectors must change (bytes incompatible by design) |
| `symbolic/normalize.go` Normalize core | `EXPR-CANON-001 + EXPR-SIMPLIFY-001 + RAT-NORMALIZE-001` (ss17.3-17.5) | **SPLIT**: flatten/sort/fold/idempotent rational rules -> physmath; `trigReduce`, regime-ZeroTest, `SplitLinear` -> phys-gr or DELETE | regime zero-test is physics, not math | identity, codec | silent behavior change if callers relied on NONZERO | idempotence + s25.8/25.9/25.27 vectors |
| `symbolic/differentiate.go` | `DIFFERENTIATE-001` (s17.9) | **MOVE** generic cases; **REMAIN** GR cap (order<=2, Symbol-only var) as caller-side constraint | closed derivative rules are core; caps are workload bounds | identity (BoundParameterRef) | Pow side-conditions need explicit MathCondition plumbing | s25.14 negative tests |
| `symbolic/subst.go` | `EXPR-SUBSTITUTE-001 + EXPR-APPLY-001` (ss17.7-17.8) | **SPLIT**: structural recursion -> physmath with declaring-Function inputs + sorted-params + ParameterID-match; ansatz pinning stays in phys-gr | spec rule contracts are stricter than current impl | identity, codec | duplicate-ParameterID + missing-declaring-Function rejections are new failures | s25.12/25.13/25.28 |
| `symbolic/truncate.go`, `limit/`, `SplitLinear` | phys-gr local (s15) | **REMAIN IN PHYS-GR** (rename to `WeakFieldTruncate` eventually) | single-rule weak-field engine, not general math | framework | name implies generality — rename, don't generalize | existing weakfield tests preserved |
| `tensor/` (Dim=4, single chart) | `physmath` Tensor/Index/Collection (ss10.4-10.6) + phys-gr chart binding | **SPLIT**: rank/slots/row-major/SET-vs-SEQUENCE -> physmath; `Dim=4/Domain/chart-equality/metric-symmetry` -> phys-gr framework data | parametric core, instantiated periphery | codec | sparse deferred; dense-only `len==Dim^Rank` check | tensor golden + negative vectors |
| `metric/connection/curvature/vacuum` | `phys-gr/framework` + 6 families (s14) | **REMAIN IN PHYS-GR**, re-expressed as PhysicalDefinition/Equation/Assumption/Interpretation over physmath artifacts | GR-specific assumptions by definition | physmath | MTW/-+++/Lambda=0 must become explicit assumption refs, not comments | GR solution tests unchanged |
| `replay/` GRStep chain | `physmath/derivation` + `Transformation` (ss10.8,12) + phys-gr protocol adapter | **SPLIT**: hash-chain/VerifyChain/ReExecute shape -> physmath derivation replay; `gr:*` op namespace + `curvature/vacuum_check` inferences -> phys-gr | transformation-vs-step lifecycle boundary s10.9 | rules, codec | flat op level mixes math+physics — separate during port | replay tests migrate to derivation-replay tests (s25.17) |
| `coordinates` | phys-gr framework data + `physmath/collection/1` SpaceRef | **REMAIN IN PHYS-GR** | chart is framework-owned | framework | — | conformance vectors |
| `solution/` (Schwarzschild) | preserved GR protocol package | **REMAIN**, untouched (regression anchor) | protocol precedent + behavior preservation | — | — | must keep passing throughout |
| `evidence/, docs/` | docs | **DEFER** (no code action) | out of Phase-1 scope | — | — | none |

Classification summary: MOVE TO PHYSMATH — expression core, canonical rewriters, closed differentiator/substitutor/applier, tensor scaffolding, replay chain shape; MOVE TO PHYS-ARTIFACT — ArtifactRef/digest/primitive codecs/grammars (none exist yet — greenfield); REMAIN IN PHYS-GR — ansatz, conventions, curvature, vacuum, weak-field, coordinates, solution; SPLIT — expr/normalize/subst/tensor/replay as above; DELETE/REPLACE — regime-ZeroTest-as-math, generic-sounding weak-field names, `gr:*` flat op namespace; DEFER — evidence/docs, TOE/Numerical/trust, sparse tensors, functional/operator/limit sorts (s6.2).

---

## 5. Implementation phases (dependency-ordered, agent-executable)

**Phase 0 — Inventory lock (done in this plan; verify before coding).**
Goal: confirm substrate locations. Tasks: `ls` phys-gr packages; record `solution/manifest.json` keys; record frozen `../phys` import surface used (`core,ops,session,hypothesis`); pin `go.work` root at `/home/chaschel/Documents/go/phys-gr`. Verification: package map (s1.1) reproduces. Exit: no code changes.

**Phase 1 — `phys-artifact` module (greenfield, ss4-5).**
Files: `phys-artifact/{artifactref,digest,primitives,bigint,grammar}.go`. Tasks: implement ArtifactRef struct (normative field order), s4.3 digest, s4.4 embedded encoding, s4.5 primitives (incl. Set sort+dedupe, Optional/Sequence framing), s4.6 BigInt (reject -0/leading zeros), namespace/artifact-type/schema-version grammars. Tests: s25.1-25.3 vectors + framing-ambiguity negatives. Exit: `go test ./...` in module passes; no math/physics imports.

**Phase 2 — PhysMath AST + codecs (expression slice, ss7-9).**
Tasks: 10-node union with tags 0x01-0x0A; Symbol validation (durable exactly-one-of ParameterID/ConstantID; ephemeral unbound allowed); Rational reduced/denominator-positive; Add/Mul flatten+sort+arity-normalize; Neg/Pow/Sqrt/Call/Relation/BranchSet per spec; durable `expression/1` payload = tag||recursive. Tests: s25.8 Add/Mul order, s25.16 relation embedding, Sqrt-vs-Pow-1/2 distinctness. Exit: golden bytes stable; idempotent canon.

**Phase 3 — Identity (ParameterID/BoundParameterRef/constants, s8-9).**
Tasks: `DeriveParameterID` = SHA256(`String("physmath")||Bytes(ScopePathBytes)||UInt32BE(ordinal)`, depth-0); ordinal-consistency validator; Function-body binding rule s8.6 (two validation regimes: strict bodies vs schematic standalone s8.8); capture-safety (no freshening); ConstantID closed `{pi,e,i}`; t/k neutrality test s25.7. Exit: ss25.4-25.6, 25.30 pass.

**Phase 4 — Higher reps (Function/Map/Tensor/Index/Collection/OperatorApplication, s10).**
Tasks: family payloads with s5.7 target validation; Map sorts restricted to s6.1 identifiers; Tensor dense checks; Index variance closed; Collection SET dedupe; OperatorApplication: OperatorRef must be absent, OperatorID must be `differentiate`, Parameters = BoundParameterRef||Order. Exit: per-family golden + negative vectors.

**Phase 5 — Rule registry + executor (s17).**
Tasks: `OperationInvocation/TransformResult` types (in-memory, immutable result, conditions = supplied set, no discharge); exact arity/family table s17.2 (fail on mismatch even if interpretable); traversal policies ss17.3-17.9 (post-order, priority-restart simplify, all-Rational-only folding s25.27, Pow(x,0)-needs-`NEQ(x,0)` fail-closed s25.26, SELECT ordinal range, APPLY root-only + staged-call discipline, SUBSTITUTE sorted-inputs/sorted-entries/distinct-ParameterIDs, DIFFERENTIATE single-target + nonnegative-int exponent); `(ProducerNamespace,RuleID)` gate + 7-rule lock ss17.11/24.5. Exit: ss25.9-25.15, 25.26-25.29 pass.

**Phase 6 — Derivation + Theorem (ss12-13) + EquationSet/Definition/Condition (s11).**
Tasks: inline DerivationStep (ordinal 0..n, same-run availability s12.6, ConditionRefs SET), Derivation SET/SEQUENCE rules, Theorem mandatory DerivationRef + conclusion byte-identity s13.3/s16.10 (construction-time check, no repository search); EquationSet SEQUENCE-with-uniqueness s11.2. Exit: ss25.17-25.18 replay determinism + completeness.

**Phase 7 — `phys-gr/framework` (first conforming phys-*, s14).**
Tasks: FrameworkArtifact (`PackageNamespace=phys-gr`), 6 families with s5.7 target validation incl. dependency-closure checks; Category/Description/PhysicalType as opaque framework strings; canonical serialization s14.11 + per-family positive/negative vectors. Exit: ss25.22-25.24, 25.31 + s21.1 negatives (10 listed) pass.

**Phase 8 — GR semantics port (metric/connection/curvature/vacuum/limit/coordinates → framework-owned).**
Tasks: re-express ansatz as PhysicalDefinition+PhysicalEquation with explicit assumption refs (MTW, -+++, Lambda=0, r>0, regularity) — converting code comments into artifact refs; keep `solution/` untouched; add adapters so legacy `symbolic/tensor` tests still pass. Verification: `go test ./...` legacy suite green (compatibility anchor). Exit: GR behavior preserved; new framework artifacts validate.

**Phase 9 — Protocol migration (GR preserved; equivariance added per s16 + architect decision).**
Tasks: keep `solution/` 4-artifact package exactly (no schema change); create `phys-gr/equivariance/{AGENTS.md,manifest.json,protocol/derivation_test.go}` reusing the `solution/` schema/loading/execution shape (go:embed, strict decode, ordered-expect consumption, firewalls, mutation harness); `manifest.json` is metadata only (never a premise; s25.25/PROTOCOL GAP rule). Physical owner = `phys-gr` (ss16.12-16.13, one FrameworkArtifact + >=1 assumption + >=1 definition/interpretation + PhysicalTheorem). Many-frameworks stays a *deferred conformance test* (second interpretation added later, not a synthetic `phys-equiv` now). Exit: ss25.19, 25.25 pass; GR suite still green.

**Phase 10 — Equivariance vertical slice (s16.4-16.11, exact).**
Tasks (in order): (a) Functions `f(v)=2v` (1 param, display `v`), `rho_X/rho_Y(g,v)=g*v` (2 params, displays `g,v`); schematic `g,x` per s16.5; (b) staged Calls `LHS_0/LHS_1/RHS_0/RHS_1` s16.9 + linkage checks (LHS_1 arg == Step0 output, RHS_1 arg2 == Step3 output); (c) 6 steps exactly (0 APPLY inner-rho_X -> `g*x`; 1 APPLY staged-LHS_1 -> `2*(g*x)`; 2 CANON -> `2*g*x`; 3 APPLY inner-f -> `2*x`; 4 APPLY staged-RHS_1 -> `g*(2*x)`; 5 CANON -> `2*g*x`); final-bytes equality; (d) relation `EQ||canon(LHS)||canon(RHS)` s16.10, Equation wrapper, Theorem + PhysicalTheorem; (e) pin golden ArtifactRefs per step + negative vectors. Forbidden: Map-for-f, durable FunctionApplication for LHS/RHS, distributivity, GroupAction sort. Exit: ss25.20-25.21, 25.32, 25.34 pass; deterministic replay across implementations.

**Phase 11 — Conformance + vectors + negatives (ss20.5, 21, 25).**
Tasks: per-family golden/negative vectors (physmath 16 + phys-gr 6 + 7 rule-param encodings); s21.1 12 negatives; base/specialized closure test pattern (s21.3, using a test-only derived namespace — not a shipped framework); cross-implementation determinism note. Exit: full suite green in all three modules + legacy GR suite.

Must-implement-now: Phases 1-11 above. Explicitly deferred (s0.4/ss22-23): TOE-Agent, Numerical/ComputationalContract, state machines, search/orchestration, trust/corpus/adjudication, paper workflow, embeddings, autoformalization, frozen-Phys projection, PhysicalContract ontology, generic NodePath selectors, sparse tensors, Functional/Operator/Limit/AsymptoticTruncation/DerivationObligation execution, second-framework fixture beyond test harness.

---

## 6. Protocol migration (detail)

What remains GR-specific: `solution/` package verbatim (Schwarzschild content, MTW/-+++/Lambda=0 pins, 14-stage trace, M1-M8 harness, `GRStep` chain). What becomes generic: the *shape* — go:embed manifest, strict typed decode, artifact-presence + exact-value tests, executable derivation test with ordered expectations + firewalls + mutation negatives, replay VerifyChain+ReExecute. `manifest.json` handling: equivariance manifest copies the 15-key schema and loading convention; values change (derivation_id, premises, expected_result = equivariance sides, verification.stages = 6); no new semantic manifest field — if the 6-step staged-call linkage cannot be expressed without one, stop and report PROTOCOL GAP (s1.4). `protocol/derivation_test.go` structure: mirror `solution/derivation_test.go` (setup fixtures -> ordered step consumption -> linkage checks -> conclusion byte-identity -> independence note -> mutation negatives scoped to the 6 steps). Equivariance fixture fits as a second protocol package beside `solution/`, sharing only the pattern. GR tests migrate by *preservation*, not rewrite: `solution/` suite runs unchanged throughout Phases 1-8 as the compatibility anchor; new tests are additive.

---

## 7. Equivariance vertical slice (end-to-end order)

```text
phys-artifact (digest/codecs)
  → physmath AST (Call/Function/Symbol-ParameterID per s16.5, displays v/g,v/x pinned)
  → phys-gr framework artifacts (FrameworkArtifact + assumptions + definition/interpretation + PhysicalTheorem)
  → fixture setup (Functions + staged LHS_1/RHS_1 Calls + expected g*x / 2*x intermediates)
  → derivation (6 steps, s16.9; linkage validation before replay)
  → protocol/derivation_test.go (replay each step via (physmath,RuleID), assert golden ArtifactRefs)
  → theorem construction (relation embeds final LHS/RHS bytes s16.10; byte-equality gate s16.11)
  → deterministic replay (second run / independent impl reproduces golden refs s25.34)
```

---

## 8. Compatibility and migration strategy

Backward-compatible refactor: `solution/` behavior and tests; `metric/connection/curvature/vacuum/limit` observable GR results; `replay.VerifyChain` semantics (re-expressed, not weakened). Behavioral change required by v1.2: canonical bytes (new framing — old GRExpr JSON is not canonical identity material); ZeroTest regime semantics -> explicit MathConditions; `UnknownFunction` -> Function+Call; flat `gr:*` ops -> 7-rule `(physmath,RuleID)` pairs; vacuum `implication_*` comments -> explicit derivation steps or declared assumptions. Temporary migration compatibility: thin `symbolic/tensor` adapters + `ToCoreExpr/FromCoreExpr`-style bridges (cf. existing `bridge.go`) kept until Phase 8 exit, then removed. Obsolete: regime-ZeroTest-as-math, `Truncate`/`SplitLinear` generic names (rename), `NonZeroClasses==9` as core assertion (fixture-local check), `symbolic` Sin/Cos-as-core (framework-local or dropped if unused by equivariance).

---

## 9. Test strategy (levels; architecture-contamination first)

Unit (constructors/validators/codecs per family) -> serialization (golden bytes per family + framing-ambiguity negatives) -> identity/hashing (digest formula, ParameterID alpha-rename/ordinal-consistency, BoundParameterRef) -> rule execution (7 rules incl. arity/family/condition gates) -> derivation replay (ordering, same-run availability, condition threading) -> phys-* conformance (s21: namespace/framework/assumption/interpretation/no-AST-extension/no-trust-duplication + 12 negatives + disagreement + base/specialized) -> protocol (artifact-presence, exact-manifest-values, firewall/mutation harness for both packages) -> golden vectors (per-family + per-rule-params + 6-step equivariance pins) -> negative tests (duplicate SET, unregistered family/RuleID, missing refs, invalid binding, discharged conditions, self/cyclic framework deps, unrelated-assumption refs) -> cross-implementation determinism (s25.34: same fixture -> identical refs). Contamination-priority tests: display-name-physics (s25.7), dynamic-AST-extension rejection (s25.24), PhysMath-leakage (physics vocabulary in core) and GR-leakage (convention-dependent behavior without assumption refs).

---

## 10. Complexity/risk register

| Risk | Prevention |
|---|---|
| Coupling: regime physics (ZeroTest r!=0, MTW/Lambda=0) hides in math core | explicit MathCondition refs + assumption artifacts; fail-closed rules; s25.10/25.26 tests |
| Cohesion: Transformation vs DerivationStep split confuses implementers | lifecycle docs + distinct types; test that Derivation is not a graph of Transformation artifacts (s10.9) |
| Core/specific leakage GR->PhysMath (Sin/Cos/UF/order-2/Dim=4/trigReduce) | module-boundary review gate; s15 locality tests (s25.24); adapters deleted after cutover |
| Core/specific leakage PhysMath->physics (t/k roles, interpretation in canon) | s25.7 neutrality test; codec review (DisplayName identity-bearing but meaningless) |
| Serialization: JSON-as-identity confusion (legacy GRExpr JSON vs canonical bytes) | codec separation; transport-vs-canonical tests; Equal-by-canonical-bytes only |
| Semantic ambiguity: staged-call linkage looking like hidden rewrite | pre-constructed Calls + byte-linkage checks as fixture validation (s16.9, s25.32) |
| Protocol drift: flat `solution/` vs nested `protocol/` layouts diverging | single pattern doc; both packages share harness conventions; manifest-schema diff test |
| Identity: ParameterID ordinal collision across Functions | BoundParameterRef everywhere cross-artifact; duplicate-ParameterID substitution rejection (s25.28) |

---

## 11. Architectural blockers requiring input

```text
ARCHITECTURAL BLOCKER B1 — Expression node-set conflict

Decision needed:
How are A(r), B(r), r^2 sin^2theta represented under the closed 10-node set?

Evidence:
phys-gr/symbolic/expr.go (UnknownFunction, Sin, Cos, Pow-int) vs spec ss7.1, 10.2, 16.4
(Function+inline-Call mandated; Sin/Cos/Sqrt status unclear; Pow exponent is arbitrary
Expression in spec but int-only in code).

Conflict with v1.2:
s7.1 closes the node set (no UnknownFunction/Sin/Cos); s16.4 mandates f/rho as
physmath/function/1 with inline Call nodes; no trig sort exists.

Why existing decisions are insufficient:
The spec never says where Sin/Cos go (core node? ConstantID? framework-local algebra
under s15? deleted as out-of-slice?) nor how the Schwarzschild r^2 sin^2theta term is
handled when the equivariance slice needs no trig at all.

Possible resolutions:
A. Framework-local trig: keep Sin/Cos in phys-gr under s15 with explicit mapping;
   equivariance slice uses only Mul (no trig), GR package retains trig locally.
B. Promote trig to a future versioned spec amendment (defer; GR metric keeps legacy
   symbolic::Sin/Cos via adapter until amended).

Recommended resolution:
A (keeps Phase-1 core closed; GR keeps what it needs locally).

Impact:
Chooses what the GR-port in Phase 8 is allowed to keep vs must delete; affects codec
scope and whether legacy metric tests need adapters.
```

```text
ARCHITECTURAL BLOCKER B2 — Regime non-vanishing (r,A,B,sin != 0) placement

Decision needed:
Do r!=0 / A!=0 / B!=0 / sintheta!=0 live as explicit MathConditions + PhysicalAssumptions,
or does any implicit nonzero rule survive in PhysMath?

Evidence:
symbolic/normalize.go ZeroTest + normalizeMul "regime-owned nonzero" comment;
tensor/metric/connection/curvature depend on it; spec ss12.5, 17.4, 17.10 (fail-closed,
explicit conditions, no silent discharge).

Conflict with v1.2:
Spec requires the Pow(x,0)->1 rule to fail without an explicit NEQ(x,0) condition and
forbids silent discharge, but never enumerates which regime conditions the GR port
must thread.

Why existing decisions are insufficient:
Two readings both conform: (i) every GR derivation step carries explicit condition
refs; (ii) framework assumptions cover regularity once at the PhysicalAssumption
level. The choice changes DerivationStep ConditionRefs content and validator strictness.

Possible resolutions:
A. Framework-level: regularity assumptions as phys-gr PhysicalAssumptions referenced
   by PhysicalTheorem/Equations; derivation steps carry conditions only where a rule
   contract demands (e.g. Pow-0).
B. Step-level: every affected derivation step carries explicit MathCondition refs.

Recommended resolution:
A (matches ss14.2/14.4 assumption ownership; avoids conditioning pure canon steps).

Impact:
Determines Phase 8 validator behavior and what the equivariance slice (which needs no
nonzero rule) must demonstrate about conditions.
```

```text
ARCHITECTURAL BLOCKER B3 — Protocol package layout (flat solution/ vs nested protocol/)

Decision needed:
Does the equivariance fixture use <pkg>/{AGENTS.md,manifest.json,protocol/derivation_test.go}
verbatim (s16.2), or mirror the actual GR precedent solution/{manifest.json,...} flat layout?

Evidence:
solution/{README.md,manifest.json,manifest_test.go,derivation_test.go} + AGENTS.md s8.1
vs spec ss1.4, 16.2, 21.4, 25.19 (AGENTS.md + manifest.json + protocol/derivation_test.go).

Conflict with v1.2:
s1.4 says manifest "MUST use the exact schema and loading convention of the existing
GR example" and forbids inventing a competing schema — but the directory shape itself
differs (flat vs protocol/ subdir; README vs AGENTS.md at package level).

Why existing decisions are insufficient:
"Reuse exactly" is satisfiable for schema/loading/execution semantics under either
layout, but file placement affects the s25.19/s25.25 conformance check literally read.

Possible resolutions:
A. Nested layout with identical schema/loading semantics (go:embed, strict decode,
   15 keys) + a layout-equivalence note recorded in the fixture AGENTS.md.
B. Flat mirror of solution/ exactly (deviates from s16.2 text but byte-matches precedent).

Recommended resolution:
A (honors both the letter of s16.2 and the substance of s1.4; minimal, documented).

Impact:
Fixes Phase 9 file creation paths and the protocol-conformance test's expected paths.
If rejected, adopt B and record the s16.2 deviation explicitly.
```

Non-blockers decided in-plan (no input needed): Go file/package naming, error-type names, fixture organization, CI, caching/storage, codec strategy, benchmarks (s24.3).

---

## 12. Final implementation sequence

```text
0. Verify substrate paths (phys-gr packages, solution/ protocol files, ../phys import surface)
1. phys-artifact module (ArtifactRef/digest/primitives/BigInt/grammars + vectors)
2. physmath AST + codecs (10-node union, Eq/Set/Sequence semantics)
3. Identity (ParameterID/BoundParameterRef/ConstantID, binding regimes, capture-safety)
4. Higher reps (Function/Map/Tensor/Index/Collection/OperatorApplication + target table)
5. 7-rule registry + executor (arity/family/params/traversal/conditions/fail-closed)
6. Derivation + Theorem (+EquationSet/Definition/Condition) + replay determinism
7. phys-gr/framework (6 families, closure validation, conformance vectors)
8. GR semantics port (assumptions made explicit; solution/ untouched; adapters temporary)
9. Protocol migration (equivariance pkg mirroring GR shape; layout per B3 ruling)
10. Equivariance 6-step slice (setup -> staged Calls -> replay -> relation/theorem -> goldens)
11. Full conformance + golden/negative/cross-impl suites; delete adapters; green CI in all modules
```

Deferred (explicit): TOE-Agent, Numerical/ComputationalContract, state machines, search/orchestration, trust/corpus/adjudication, paper workflow, embeddings, autoformalization, frozen-Phys projection/kind system, PhysicalContract ontology, generic NodePath selectors, sparse tensors, deferred sorts' execution, second shipped framework (test-harness only in Phase 1).

**Stop rule (s24.4):** any MATH/PHYS/FROZEN-PHYS/INTEGRATION/PROTOCOL gap found during Phases 1-11 is reported in B1-B3 format with evidence and competing readings; no silent invention of identity, interpretation, serialization, derivation, protocol, namespace, condition, or ownership rules.
