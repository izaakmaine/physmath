# Implementation Plan 9.2 — `phys-artifact` + `phys-math` + `phys-lib`

```text
STATUS: ARCHITECTURALLY CLOSED FOR IMPLEMENTATION PLANNING
```

> **Note on artifact location.** This file is the implementation-plan artifact. It lives
> under `physmath/plans/` as a planning document only. The canonical Go code does **not**
> belong under a `physmath/` repository. Canonical implementation lives in the sibling
> workspace `/home/chaschel/Documents/go/` as `phys-artifact/`, `phys-math/`, `phys-lib/`,
> with `phys/` frozen and `phys-gr/` preserved as historical reference.

---

## 1. Executive architectural status

The topology is settled and is not redesigned here: four-tier sibling workspace
(`phys-artifact` → `phys-math` → `phys-lib/core` → `phys-lib/<framework>`), single-module
`phys-lib`, byte-untouched `phys-gr`, empty in-memory-only `core`, equivariance owned by
`phys-lib/gr`, test-only `fixture2`, GR protocol files reused verbatim in semantics.

Four adversarial reviews of `plan9.1.md` (`plan9.1_deepseek.md`, `plan9.1_gemini.md`,
`plan9.1_qwen.md`, `plan9.1_z.md`) concur: the architecture is sound; the remaining work
is closure, not redesign. Every blocker and residual they identified is resolved below as
an **adopted decision**, an **explicit deferral with a defined boundary**, or an
**explicit ruling-out**. There are zero open architectural blockers, zero
"recommended / probably / e.g. / TBD / awaiting input" wordings on implementation-critical
decisions, and no coding-phase architectural invention is authorized.

**Version-label hygiene (recorded, not redesigned).** The operative specification file is
`plans/PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md`. Its header reads `Version: 1.2`
(Document ID `PM-REP-001`) while body text (e.g. §0.5, §24.0) refers to "v1.4". Operative
rule: the file's normative content governs as written; "PM-REP-001 v1.2" names the
specification throughout this plan; the header/body label mismatch is documentation
hygiene to be fixed at the next spec touch. No semantics are altered on account of it.
(Review texts citing "v1.4" refer to this same file.)

---

## 2. Explicit adopted decisions

```text
D1  B1 (trig / GR-local algebra): ADOPTED A. Sin, Cos, UnknownFunction-shape needs,
    trig simplification, regime-aware zero testing, GR derivative caps, and all GR-local
    symbolic behavior live in phys-lib/gr as theory-local algebra (§15 mechanism).
    phys-math is never extended; the 10-node union is closed.
D2  B2 (regime conditions): ADOPTED A. Framework-level GR regularity/non-vanishing
    requirements are PhysicalAssumption artifacts. DerivationStep carries MathCondition
    only when a closed phys-math rule contract demands it. Bridge (§7): rule returns
    TransformResult bearing the MathCondition → gr captures the reference → gr publishes
    a PhysicalAssumption referencing it → PhysicalTheorem/PhysicalEquation references
    the assumption. No condition is silently discharged.
D3  phys-gr: byte-untouched historical artifact. Never edited, moved, renamed, deleted,
    truncated, refactored in place, or extended. Refactoring means fresh canonical
    implementation in phys-lib/gr using phys-gr as read-only source.
D4  No new module imports github.com/PithomLabs/phys-gr under any circumstance.
    No shim exception. Legacy validation uses copied golden vectors, never imports.
D5  phys-lib/core starts empty and is IN-MEMORY ONLY in Phase 1: no FrameworkArtifact,
    no durable families, no ArtifactRef-bearing schemas, no core artifact namespace.
    Promotion requires a §6 record; default keeps abstractions in phys-lib/gr.
D6  Phase 4 is scaffolding + classification-mechanism only. Actual classification and
    any promotion happen during Phase 5 against extracted code.
D7  Phase-1 phys-lib/gr scope: FrameworkArtifact + six framework families +
    GR-local algebra + equivariance fixture + required tests. The Schwarzschild
    protocol's canonical executable home remains phys-gr/solution/ (untouched, deferred
    re-expression). Consequence: Phase-1 new modules have ZERO frozen-phys imports and
    reference no frozen namespace.
D8  New GR fixture uses nested layout <fixture>/{AGENTS.md,manifest.json,
    protocol/derivation_test.go} with GR-identical schema/loading/execution semantics.
    Original phys-gr/solution/ is the preserved reference. Spec §26-step-10-style
    "refactor into canonical pattern" is satisfied by the new fixture, not by touching
    phys-gr/solution/.
D9  GR-local expression type is a DISJOINT Go type from phys-math.Expression. No shared
    interface, no tagged-union extension. Sole bridge: explicit deterministic projection
    of the representable subset, failing with ErrUnrepresentableProjection otherwise.
D10 Equivariance symbols: Function artifacts for f, rho_X, rho_Y are mandatory; every
    durable free symbol carries the exact ordinal-derived ParameterID of its declaring
    slot. No bare NewSymbol("g")/NewSymbol("x") in durable artifacts.
D11 Tensor split: phys-math owns rank, mathematical dimension/size, index variance,
    dense representation, generic shape validation, spec-authorized generic ops.
    phys-lib/gr owns ChartID compatibility, metric signature, symmetry semantics,
    connection-vs-tensor distinction, GR contraction/validation semantics.
D12 Namespace separation: durable namespace physmath is UNCHANGED (baked into §8.3
    ParameterID framing and all 16 family IDs) despite Go module
    github.com/PithomLabs/phys-math. GR durable framework namespace is phys-gr
    (FrameworkArtifact PackageNamespace = "phys-gr") despite Go package phys-lib/gr.
    fixture2 durable namespace is phys-fixture2 (pinned, invariant across runs).
D13 BaseFrameworkRefs is artifact/data dependency only. It never authorizes a Go import
    between framework packages. No inter-framework Go imports exist in Phase 1.
D14 Phase-1 determinism acceptance: deterministic serialization, identity, replay, and
    repeated-replay golden-ArtifactRef equality within the single implementation.
    True two-implementation equivalence is deferred until a second implementation exists.
D15 Category vocabulary for GR PhysicalAssumption is the closed enum
    {convention, regularity, model}. Any other value is rejected; extension requires a
    spec change. Convention-type assumptions (signature, curvature convention, Λ choice)
    and non-PhysMath-expressible regularity publish EMPTY ConditionRefs with
    identity-bearing package-owned Description; only PhysMath-expressible predicates
    travel via MathCondition/ConditionRefs.
D16 Vocabulary tests are semantic/mechanical (whole identifiers, exported-API semantics,
    imports, declared schema meaning), never naive substring traps. Mathematical
    Dimension/rank/variance, comparison operators, and mathematical Function are
    permitted; physical meaning is what is banned. The mathematician test is a REVIEW
    criterion; import/vocabulary/API scans are the MECHANICAL CI gates.
D17 go.work is local development wiring only. No release/tag/publication mechanics are
    invented in Phase 1. No replace directives in new modules.
```

### Resolutions against review/spec conflicts (recorded explicitly)

```text
R1  Gemini chain-rule invention (Call → Derivative(f,i) node): REJECTED. The spec
    (§17.9) places Call differentiation out of scope for the Phase-1 core runtime.
    DIFFERENTIATE-001 implements exactly the closed scalar rules in §17.9; analytic
    derivative substitution happens via EXPR-APPLY-001 body expansion or gr-local
    rewrites. No new node, no Derivative artifact, no hidden operator.
R2  Fixture directory pin: the equivariance fixture lives at
    phys-lib/gr/equivariance/{AGENTS.md,manifest.json,protocol/derivation_test.go}
    (dedicated fixture package; gr root holds the library, not fixture metadata).
R3  Rule identity: canonical identity is the PAIR (ProducerNamespace, RuleID).
    Colon-joined "physmath:EXPR-CANON-001" is transport/debug/error-key form only,
    never canonical bytes. Unknown pairs fail closed with ErrUnknownRuleID.
R4  fixture2 conformance reading: fixture2 is a FULLY CONFORMING MINIMAL framework —
    valid FrameworkArtifact plus minimal implementations of all six framework-owned
    families. (Adopts the literal §5.1 reading; rejects the "subset suffices" shortcut.
    Minimal content, full shape.)
R5  Spec §3 permits phys-* → Phys "when required"; THIS PLAN prohibits all Phase-1
    new-module Phys imports (D7). The permission is not exercised; any future use is a
    separately scoped decision using go.work use-directives, never a fabricated version.
R6  Dependency arrows are PERMITTED directions, not required imports. gr need not
    import core while core is empty; the prohibition runs the other way.
```

## 3. Explicit deferred decisions (with boundaries)

```text
F1  Schwarzschild re-expression in phys-lib/gr: deferred to a separately scoped future
    phase. Boundary: semantic anchor remains phys-gr/solution/; no Phase-1 code depends
    on the re-expression existing.
F2  Production qm/bohm/cst/cdt packages: deferred. Boundary: no directories, no stubs,
    no placeholder namespaces beyond the closed-registry rule itself.
F3  Any phys-lib/core promotion lacking a §6 record: deferred by default into gr.
    Boundary: empty core is conforming; promotion happens only on evidence.
F4  True cross-implementation equivalence: deferred until a second implementation
    exists. Boundary: Phase 1 proves single-implementation determinism (D14).
F5  Frozen-Phys integration of new modules: deferred. Boundary: zero imports, zero
    namespace references in Phase 1 (D7/R5).
F6  Publication/tag/version strategy: deferred. Boundary: module paths declared in
    go.mod; go.work is local-only (D17).
F7  Sparse tensors; deferred-sort execution (Functional/Operator/Limit/
    AsymptoticTruncation/DerivationObligation): deferred per spec §6.2. Boundary:
    in-memory vocabulary only, no durable families, no invented execution.
```

Ruled out (never to be reopened in Phase 1): new PhysMath families or nodes; generic
PhysicalContract ontology; phys-equiv or any synthetic production framework; deferred
theory modules; core populated for convenience; second protocol/manifest schema;
dynamic AST extension; silent condition discharge; NodePath selectors.

---

## 4. Final repository/module/package topology

```text
/home/chaschel/Documents/go/              <- workspace root
  go.work                                   (extend: use ./phys ./phys-artifact ./phys-math ./phys-lib;
                                             remove ./phys-gr only at Phase-10 gate)
  phys/                                     FROZEN — DO NOT MODIFY (module github.com/PithomLabs/phys)
  phys-gr/                                  HISTORICAL — DO NOT MODIFY (module
                                            github.com/PithomLabs/phys-gr; preserved byte-for-byte)
  phys-artifact/  CREATE                    module github.com/PithomLabs/phys-artifact
  phys-math/      exists empty               module github.com/PithomLabs/phys-math
  phys-lib/       exists empty               module github.com/PithomLabs/phys-lib (SINGLE module)
    core/                                     starts EMPTY (package core; future sub-packages
                                              only with meaningful names — never util/common)
    gr/                                       canonical GR library
      equivariance/{AGENTS.md,manifest.json,protocol/derivation_test.go}
    internal/conformance/fixture2/            test-only; durable namespace phys-fixture2;
      {AGENTS.md,manifest.json,protocol/derivation_test.go}
      never imported by production code; never a dependency of core or gr
```

## 5. Dependency rules (mechanical, CI-enforced)

Permitted partial order: `phys-lib/<fw> → phys-lib/core → phys-math → phys-artifact`,
plus direct `phys-lib/gr → phys-math | phys-artifact` where the spec requires it.
Prohibited: `phys-artifact → anything`; `phys-math → phys-lib/*`;
`phys-lib/core → framework packages`; any framework→framework Go import (D13);
any new module → `github.com/PithomLabs/phys-gr` (D4); any new module → frozen `phys`
in Phase 1 (D7). Gates: import scanner (`go list` closed-world check), `go mod graph`
acyclicity, `go test ./...` per module. `go.work` wiring only; zero `replace` in new
`go.mod` files.

## 6. Core/specific classification protocol

Every significant `phys-gr` abstraction gets exactly one classification:
`PHYS-MATH | PHYS-LIB/CORE | PHYS-LIB/GR | SPLIT | REPLACE | DELETE | DEFER`.
SPLIT names the exact type/function/interface boundary, ownership of both sides, and
dependency direction. Promotion to `core` requires the full record:

```text
Component: / Current location: / Candidate destination:
Why it is theory-agnostic: / Known/current consumers: / Potential future consumers:
Theory-specific assumptions: / Stable interface: / Dependencies:
Why it does not belong in phys-math: / Why it does not belong in phys-lib/gr:
```

Eligibility: theory-independent meaning; cross-framework-stable semantics; no
theory-specific assumptions in the API; meaningful reusable interface; demonstrable
multi-framework consumer evidence; documentable without naming a theory. "Useful to us"
is insufficient. Ambiguity defaults to `phys-lib/gr`. Applied per decision through
Coupling (separate conflated concerns), Cohesion (smallest stable unifying interface),
Core/specific (smallest true denominator downward, differences outward).

## 7. Full refactoring map

| CURRENT (phys-gr/, read-only) | TARGET | CLASS | COMMON DENOMINATOR | SPECIFIC DIFFERENCE | TARGET INTERFACE | DEPS | TEST IMPACT | SEQ |
|---|---|---|---|---|---|---|---|---|
| `symbolic/expr.go` tree mechanics, canonical-JSON discipline, byte-equality | `phys-math` AST+codec | SPLIT | closed tree mechanics, recursive tag encoding, byte-equality | 9-kind set (Sin/Cos/UF/Pow-int), `MaxDerivativeOrder`, `ValidSymbolName` bound | `phys-math` ctors/validators/codec; disjoint `gr.GRExpr` + `ToPhysMath()` projection w/ `ErrUnrepresentableProjection` (D9) | phys-artifact | new golden bytes (incompatible by design) | Ph2 |
| `symbolic/normalize.go` flatten/sort/fold, fixpoint, idempotence | `phys-math` EXPR-CANON/SIMPLIFY/RAT-NORMALIZE | SPLIT | commutative normalization, all-Rational-only folding, §7.6 arity | `trigReduce`, regime-ZeroTest, `SplitLinear` → gr-local | rule executor + condition threading (D2 bridge) | identity, codec | §§25.8/25.9/25.27 vectors | Ph3 |
| `symbolic/differentiate.go` linearity/Leibniz/Pow | `phys-math` DIFFERENTIATE-001 | SPLIT | closed §17.9 scalar rules (NO Call differentiation per R1) | order≤2 cap, Symbol-name-only var, no-chain rule → gr-local | BoundParameterRef + Order params | identity | §25.14 negatives | Ph3 |
| `symbolic/subst.go` structural recursion | `phys-math` SUBSTITUTE/APPLY-001 | SPLIT | simultaneous ParameterID substitution, sorted inputs/entries, distinct targets | zero-order-UF-only targets, no UF-arg rewrite, single-arg f(r) → gr-local | declaring-Function inputs, staged-call discipline | identity, codec | §§25.12/25.13/25.28 | Ph3 |
| `symbolic/truncate.go`, `limit/`, `SplitLinear` | `phys-lib/gr` (`WeakFieldTruncate`) | PHYS-LIB/GR | — | single-rule Schwarzschild O(eps) engine | framework-local fn + explicit mapping doc | framework | ported weakfield vectors | Ph5 |
| `tensor/` offsets/validation/arithmetic | `phys-math` Tensor/Index/Collection | SPLIT | rank, math dimension, variance, dense layout, generic validation/ops | `Dim=4`, `Domain=spacetime`, ChartID-equality, symmetric-metric assume, re-Normalize → gr validation layer (D11) | parametric core types; chart/signature/connection semantics in gr | codec | tensor goldens + negatives | Ph2–3 |
| `metric/connection/curvature/vacuum/coordinates` | `phys-lib/gr` framework artifacts | PHYS-LIB/GR | formula loops re-expressed on phys-math | ansatz, MTW, −+++, Λ=0, chart, regularity, contraction choice | six families + assumption refs (Category ∈ {convention,regularity,model}, D15) | phys-math | GR regression via ported vectors | Ph5 |
| `replay/` chain+dispatch shape | `phys-math` derivation replay | SPLIT | hash-chain, VerifyChain, ReExecute dispatch pattern | `gr:*` namespace, `curvature`/`vacuum_check` as ops → gr-side protocol adapter | Transformation (durable) vs DerivationStep (inline) | rules, codec | derivation-replay tests §25.17 | Ph3 |
| `coordinates` chart data | `phys-lib/gr` (+ `collection/1` SpaceRef) | PHYS-LIB/GR | — | spherical-static chart | framework data | framework | conformance vectors | Ph5 |
| `solution/` semantics | preserved reference; NOT re-expressed in Phase 1 | DEFER (F1) | derivation pattern only | all Schwarzschild content | none in Phase 1 | — | original suite green independently | — |
| `evidence/`, `docs/` | — | DEFER | — | — | — | — | none | — |
| chart ops, generic replay helpers | `phys-lib/core` candidacy | DEFER-DECISION (default stays in gr) | only with §6 record proving theory-independence | — | — | — | — | Ph4–5 |

"Move" is never sufficient: every row is a deliberate re-expression against the locked
architecture, validated by ported golden vectors (copied data, D4).

## 8. Phase-by-phase implementation plan

```text
Phase 0  Authority + inventory + workspace verification. Verify §4 topology, record the
         15-key manifest schema from phys-gr/solution/, record frozen-phys import surface
         (informational only). Extend go.work (retain ./phys-gr until Phase-10 gate).
         NO modifications to phys/ or phys-gr/. Exit: inventory checklist signed.
Phase 1  phys-artifact (CREATE module). ArtifactRef (normative order), §4.3 digest, §4.4
         embedded encoding, §4.5 primitives (Set sort+dedupe; Optional/Sequence framing),
         §4.6 BigInt, §5.1–5.4 grammars. Semantic boundary test (D16) + §§25.1–25.3
         vectors. Exit: go test ./... green; stdlib-only imports proven.
Phase 2  phys-math representation + identity. 10-node AST (tags 0x01–0x0A), Symbol regimes
         (durable exactly-one-of; ephemeral unbound allowed), reduced Rationals, Add/Mul
         flatten+sort+arity, Function/Map/Tensor/Index/Collection/OperatorApplication
         (OperatorRef absent; OperatorID == "differentiate"), ParameterID
         SHA-256(String("physmath")||Bytes(ScopePath)||UInt32BE(ordinal)), empty ScopePath
         = EncodeBytes([]) = 0x00000000, ordinal-consistency validator, capture-safety
         (no freshening), ConstantID {pi,e,i}, §5.7 target-family validators. Exit:
         per-family goldens + negatives; t/k neutrality §25.7.
Phase 3  phys-math rules + derivation. OperationInvocation/TransformResult (immutable;
         conditions = supplied set, never discharged); exact §17.2 arity/family table
         (mismatch fails even if interpretable); traversal policies §§17.3–17.9
         (post-order; priority-restart simplify; all-Rational-only folding; Pow(x,0) needs
         exact NEQ(x,0) — fail closed; SELECT range-checked; APPLY root-Call-only +
         staged-call discipline; SUBSTITUTE sorted inputs/entries + distinct ParameterIDs;
         DIFFERENTIATE single-target, nonneg-int exponent, NO Call differentiation — R1);
         (ProducerNamespace,RuleID) lock + ErrUnknownRuleID (R3); inline DerivationStep
         (ordinals 0..n, same-run availability, ConditionRefs SET); Derivation/
         Theorem with mandatory DerivationRef + conclusion byte-identity
         (construction-time check, no repository search); EquationSet
         SEQUENCE-with-uniqueness. Exit: §§25.9–25.18, 25.26–25.29.
Phase 4  phys-lib module + EMPTY core scaffolding + classification-record mechanism.
         package core with zero domain abstractions; record template committed; NO
         promotions (D6). Exit: core-empty assertion test green.
Phase 5  phys-lib/gr implementation. FrameworkArtifact (PackageNamespace "phys-gr",
         BaseFrameworkRefs empty), six families with §5.7 closure validation, Category
         enum enforcement (D15), GR-local algebra as disjoint types + explicit fallible
         projection (D1/D9), tensor semantic-validation layer (D11), D2 bridge mechanics,
         §8 classification executed per abstraction (promotions only with records).
         Exit: framework goldens/negatives; §21.1 negatives; ported GR vectors.
Phase 6  Protocol fixture (nested layout, R2 path) reusing GR schema/loading/execution
         semantics verbatim (go:embed, strict decode, DisallowUnknownFields,
         artifact-presence, ordered expectations, firewalls). PROTOCOL GAP rule armed:
         any inexpressible need stops work and is reported, never improvised. Exit:
         §§25.19, 25.25.
Phase 7  Equivariance six-step slice (§11 of the mandate: Functions f/rho_X/rho_Y;
         staged LHS_0/LHS_1/RHS_0/RHS_1 + linkage checks; steps
         APPLY,APPLY,CANON,APPLY,APPLY,CANON → 2·g·x both sides; relation
         EQ||canon(LHS)||canon(RHS); Equation; Theorem + PhysicalTheorem; golden refs
         pinned; rho_X/rho_Y canonical-coincidence documented as intentional —
         distinction lives in interpretation). Forbidden list enforced. Exit:
         §§25.20–25.21, 25.32, 25.34(single-impl reading, D14).
Phase 8  fixture2 (phys-fixture2, fully conforming minimal framework — R4) at
         internal/conformance/fixture2 with identical nested protocol shape. Proves:
         distinct identity, unchanged PhysMath identity, no global winner,
         unrelated-assumption rejection, self/cyclic dependency rejection,
         disagreement without PhysMath mutation. Isolation test: no production import
         path reaches fixture2. Exit: §§25.22–25.23, 21.2–21.3 patterns.
Phase 9  Full suite: per-family goldens/negatives (phys-math 16 + gr 6 + 7 rule-params),
         §21.1 twelve negatives, contamination tests (D16), staged-call linkage,
         repeated-replay determinism, architecture-of-complexity record audit. Exit:
         everything green in all three modules + legacy phys-gr suite independently green.
Phase 10 Migration gate (mechanical): grep/import scan + go list + go mod graph +
         go test ./... prove zero phys-gr and zero phys imports in new modules; remove
         phys-gr from go.work; legacy phys-gr remains physically present and untouched;
         final architecture audit against the §15 quality gate. Exit: handoff checklist.
```

## 9. Exact protocol migration plan

Copy (never reinterpret): 15-key manifest schema (values change: derivation_id,
premises, expected_result = equivariance sides, verification stages = 6); go:embed
loading; strict typed decode with unknown-field rejection; README/AGENTS role split
(human orientation vs machine metadata); artifact-presence + exact-value tests;
prose-is-not-math check; ordered step consumption; firewalls (fixture-local analogues:
staged-call linkage, no-skip, no-substitution-of-targets); mutation negatives scoped
to the 6 steps; manifest-never-a-premise (reverse firewall). Flat→nested equivalence
recorded in fixture AGENTS.md. manifest.json remains metadata-only.

## 10. Exact equivariance fixture plan

Per §8-Phase-7 above plus: display names pinned exactly (`v` for f slot; `g,v` for
action slots; schematic `g,x` — identity-bearing, fixture-versioned); Function
ParameterIDs `rho_X = rho_Y = [PID_0, PID_1]` positionally (schematic sharing by
design, §16.5); `f = [PID_0]` capture-safe via whole-slot substitution; pre-step
linkage validation (`LHS_1.Args[0] == Step0 bytes`, `RHS_1.Args[1] == Step3 bytes`);
final byte-equality gate before relation construction (§16.11); golden ArtifactRefs for
every step input/output, the relation, equation, theorem, and all six gr-side artifacts.

## 11. Exact fixture2 conformance plan

Namespace `phys-fixture2` (D12). Minimal FrameworkArtifact (`FrameworkID`
"fixture2", `FrameworkVersion` pinned opaque string, empty BaseFrameworkRefs); ≥1
assumption with a DIFFERENT Category/Description over the same MathCondition family;
≥1 interpretation with a different PhysicalType over the same PhysMath refs; minimal
definition/equation/theorem completing the six families (R4); own nested protocol
package replaying a trivial derivation to prove harness reuse. Negative battery:
cross-namespace assumption reference without edge → reject; self-reference → reject;
injected cycle → reject; dynamic AST extension attempt → reject; display-name-physics
inference → reject.

## 12. Acceptance/test matrix (20 gates)

```text
[1] dependency graph (import scan + mod graph) [2] namespace separation (durable vs Go)
[3] artifact purity (semantic boundary test) [4] math purity (vocab/import/API scan)
[5] core emptiness / promotion records [6] GR-locality (disjointness + projection-failure
test) [7] phys-gr immutability (checksum-oracle in CI comparing against recorded tree hash)
[8] zero phys-gr imports [9] zero phys imports [10] ParameterID determinism (vectors +
alpha-rename + ordinal-consistency) [11] canonical encoding (framing-ambiguity + BigInt
edge vectors) [12] rule-ID closure (ErrUnknownRuleID + arity/family matrices)
[13] step target-family validation [14] condition threading (fail-closed Pow-0; bridge
capture test) [15] staged-call linkage [16] six-step golden refs [17] fixture2 isolation
(no production import path; distinct artifacts) [18] classification records present for
every §7 row [19] protocol-manifest compliance (schema-diff vs GR original)
[20] deterministic replay (N consecutive replays → identical refs)
```

Gates 1–9, 12, 19 are fully mechanical CI; 4/16/18 pair mechanical scans with the
mathematician/architect REVIEW criteria (D16).

## 13. Migration gates

G1 (Phase 0): inventory checklist; workspace resolves; phys/ and phys-gr checksums
recorded as immutability oracles. G2 (Phase 5): ported GR vectors pass; §8 records
complete. G3 (Phase 10): zero-import proofs; phys-gr removed from go.work; legacy tree
hash unchanged; frozen phys untouched (its own tree hash unchanged). `replace => ../phys`
inside phys-gr is legacy-internal and is never copied into new modules.

## 14. Risks and failure modes

Core-as-dumping-ground → record gate + empty-start + default-in-gr. Convenience
classification → quality-gate audit item ("no decision reduces to convenience").
GR→math leakage → disjointness + vocabulary scans + B1 ruling. Math→physics leakage →
mathematician review + §25.7. Core contamination → theory-name scan on core API.
JSON-as-identity → codec separation + Equal-by-canonical-bytes. Protocol drift →
schema-diff test + equivalence note. Staged-call spoofing → linkage pre-validation.
ParameterID collision → BoundParameterRef + distinct-target enforcement. Workspace
drift → import lint. Single-impl determinism overstated → D14 wording locked in tests.
Review-introduced inventions (chain-rule Derivative node, subset-conformance,
colon-canonical IDs) → R1/R3/R4 rejections above.

## 15. Architectural quality gate

```text
[ ] PhysMath contains no physical concepts (review + scan).
[ ] core contains no theory-specific semantics; every item has a §6 record
    (empty core is a PASS).
[ ] gr owns all GR semantics; GR-local math is disjoint + fallible-projected.
[ ] No reverse/lateral/phys-gr/phys imports (mechanical proof).
[ ] All core/specific decisions justified by record, never convenience.
[ ] No util/common/shared packages.
[ ] phys-gr inspected, preserved, checksummed; zero modifications.
[ ] GR protocol semantics reused; no provisional schema; GAP rule armed.
[ ] Equivariance owned by phys-lib/gr; six steps exact; goldens pinned.
[ ] fixture2 (phys-fixture2, fully conforming minimal) proves independence + isolation.
[ ] Frozen Phys untouched; replace-removal gated.
[ ] Unsupported GR math kept out of PhysMath (R1 enforced).
[ ] Formalization executable through nested protocol packages.
[ ] B1/B2 adopted (D1/D2); zero awaiting-input items.
[ ] Deferred list (§3 F1–F7) untouched; nothing ruled-out smuggled in.
```

## 16. Final coding-agent handoff checklist

```text
- [ ] Read PM-REP-001 v1.2 (operative file), §§4–5 identity, §7–10 representation,
      §17 rules, §§12–14 derivation/theorem/framework, §16 fixture, §§20–21/25 tests.
- [ ] Read phys-gr/solution/{manifest.json,manifest_test.go,derivation_test.go},
      replay/, AGENTS.md §8.1 as protocol authority. Copy semantics, never invent.
- [ ] Execute Phases 0–10 in order; respect exit criteria; never skip gates.
- [ ] Apply D1–D17 and R1–R6 exactly; F1–F7 are out of scope.
- [ ] On any gap: STOP and file an ARCHITECTURAL BLOCKER (format §20 of mandate) —
      do not invent identity, interpretation, serialization, derivation, protocol,
      namespace, condition, or ownership rules.
- [ ] Forbidden in Phase 1: new PhysMath families/nodes; PhysicalContract;
      phys-equiv; deferred theory modules; populated-without-evidence core;
      phys-gr or phys imports; provisional schemas; silent discharge; NodePath selectors.
```

```text
FINAL HANDOFF:
- zero unresolved architectural blockers
- zero implementation-critical ambiguities
- no decision left to the coding agent
- no modification of phys-gr
- no modification of frozen phys
- phys-math remains pure mathematics
- phys-lib/core remains empty unless evidence-based promotion occurs
- phys-lib/gr owns GR semantics
- no phys-gr imports in canonical modules
- Phase-1 GR scope is explicitly bounded
- protocol authority is the existing GR precedent
- equivariance is the canonical Phase-1 vertical slice
```
