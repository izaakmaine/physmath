# Implementation Plan 9.3 — `phys-artifact` + `phys-math` + `phys-lib` (closure and execution)

```text
STATUS: CLOSED FOR IMPLEMENTATION
```

> **File note.** This is the implementation-plan artifact under `physmath/plans/`. Canonical
> Go code lives only in the sibling workspace `/home/chaschel/Documents/go/` as
> `phys-artifact/`, `phys-math/`, `phys-lib/` (frozen `phys/`, historical `phys-gr/`).

## 1. Executive status

Architecture closed. Topology, boundaries, rule registry, fixture contracts, and scope are
all decided below. This document distinguishes four decision states, and only four:

```text
ALREADY DECIDED ......... implementation must follow (§2)
DEFERRED ................ explicitly outside Phase-1 scope, with boundary (§3)
PERMITTED DETAIL ........ coding agent decides idiomatically, no asking (§17)
NEW BLOCKER ............. stop affected behavior, ask architect (§17)
```

Operative spec: `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md` (Document ID
PM-REP-001; header `Version: 1.2`, body references to "v1.4" are label hygiene — file
content governs as written). Reviews absorbed: `plan9.2_deepseek/gemini/qwen/z.md`
(deepseek's 3 blockers, gemini's 5 hazards, qwen's 6 locks, z's scope pin + 6 one-liners).
Where a review conflicts with the spec or the §40 mandate, this plan resolves against the
higher authority and records it (§2, R-items). In particular: gemini Hazard-5's
`phys-gr:func/sin` keys are **rejected** (§13 mandate — no synthetic identifiers);
Deepseek Blocker-1 is resolved by scoping Category to GR-local (§24 mandate).

## 2. Adopted decisions (ALREADY DECIDED — follow exactly)

```text
D1  B1: Sin/Cos/UF-shape/trig-simplification/regime-zero-test/GR-derivative-caps live in
    phys-lib/gr (§15 theory-local). phys-math closed 10 nodes, never extended.
D2  B2: regime/non-vanishing requirements are framework-level PhysicalAssumptions.
    OperationInvocation carries MathCondition/ConditionRefs ONLY when a closed rule
    contract demands a supplied condition. Phase-1 rules generate no conditions.
D3  phys-gr byte-untouched, forever. No edit/move/rename/delete/refactor-in-place/compat code.
D4  No new module imports github.com/PithomLabs/phys-gr, no exceptions, no shims.
    Validation via copied vectors only.
D5  phys-lib/core starts EMPTY and is IN-MEMORY ONLY in Phase 1: no FrameworkArtifact,
    no durable families, no ArtifactRef schemas, no namespace. Promotion needs a §6 record.
D6  Phase 4 = module scaffolding + core/doc.go + classification mechanism. No promotions.
D7  Phase-1 gr scope: FrameworkArtifact + six families + GR-local algebra + equivariance
    fixture + required tests. Schwarzschild stays executable ONLY in phys-gr/solution/.
    Zero Phase-1 frozen-phys imports; zero frozen-namespace references.
D8  New fixture at phys-lib/gr/equivariance/{AGENTS.md,manifest.json,
    protocol/derivation_test.go} with GR-identical schema/loading/execution semantics.
D9  GRExpr disjoint from phys-math.Expression. No shared interface. Sole bridge: explicit
    deterministic ToPhysMath() of the representable subset; failure yields
    ErrUnrepresentableProjection. REJECTED (R7): no "phys-gr:func/sin" style keys —
    Sin/Cos are never projected, never given PhysMath function identities.
D10 Fixture Functions f/rho_X/rho_Y mandatory; durable symbols carry exact declaring-slot
    ParameterIDs. No bare NewSymbol("g")/("x"). Displays pinned: f:"v"; rho_X/rho_Y:"g","v";
    schematic "g","x" (identity-bearing golden data).
D11 Tensor split: phys-math = rank, math dimension/size, variance, dense layout, shape
    validation, canonical serialization, spec-authorized representation ONLY — Phase-1
    phys-math Tensor = construction + validation + serialization, NO tensor operations
    (seven-rule lock admits none). gr = ChartID, signature, symmetry, connection
    distinction, contraction/validation semantics.
D12 Namespaces pinned: durable physmath (unchanged) vs module github.com/PithomLabs/phys-math;
    durable phys-gr (PackageNamespace) vs package phys-lib/gr; fixture2 = phys-fixture2.
D13 BaseFrameworkRefs = artifact/data dependency only; never a Go import.
D14 Phase-1 determinism = single-implementation serialization/identity/replay determinism +
    repeated-replay golden equality. Two-implementation equivalence deferred (F4).
D15 GR Category = closed GR-package-contract enum {convention, regularity, model}; other
    values rejected BY GR VALIDATORS ONLY. Not global: other frameworks define their own
    values; no cross-framework rejection on Category (spec §14.2 preserved). "Extension
    requires a spec change" is DROPPED — extension is a gr package-contract decision.
    Convention-type/non-expressible assumptions publish EMPTY ConditionRefs + identity-
    bearing Description; only PhysMath-expressible predicates travel via conditions.
D16 Vocabulary tests semantic/mechanical (identifiers, API semantics, imports, schema
    meaning). Math Dimension/operators/Function permitted; physical meaning banned.
    Mathematician test = review criterion; scans = CI gates.
D17 go.work local wiring only; no releases/tags; no replace in new modules.
D18 Ported vectors are category-typed (§9): (A) re-encoded PhysMath-expressible content
    with NEW canonical bytes; (B) gr-local behavior tests; (C) projection success/failure
    tests. Legacy bytes are NEVER byte-compared to canonical bytes.
D19 Classification records live at phys-lib/classifications/<slug>.md with --- front-matter
    (id/source/class/target/status) + the §6 evidence fields; committed with the moving PR;
    CI enforces per-row presence + evidence completeness for CORE. Full records required
    for PHYS-LIB/CORE; classification + named boundary for SPLIT; classification for rest.
D20 core/doc.go = `package core` + doc comment ONLY, zero exported identifiers; record
    template is documentation, exempt from the empty-core assertion.
D21 SUBSTITUTE wording mirrors spec §17.8 verbatim: no two entries share a ParameterID,
    REGARDLESS OF DeclaringArtifactRef. OperationInvocation immutability is NOT required
    (only TransformResult immutability is normative); do not assert it.
D22 Executor creates NO durable artifacts: conditions are materialized by fixture/framework
    code BEFORE referencing; TransformResult.ConditionRefs reference pre-materialized
    conditions (= normalized supplied set; Phase-1 rules generate zero conditions).
D23 Deferred sorts (Functional, FunctionalApplication, Operator, Limit,
    AsymptoticTruncation, DerivationObligation) are in-memory-only; inventing families
    (e.g. physmath/limit/1) is forbidden. APPLY is capture-free by construction: no
    alpha-freshening, renaming, allocators. Call differentiation out of scope (R1 stands).
D24 FrameworkVersion pinned: gr fixture "1", fixture2 "1" (opaque strings, no semver ontology).
D25 EquationSet = SEQUENCE with uniqueness validation, never a SET.
D26 Relation: Expression node = 0x09||Enum||Left||Right; durable payload = Enum||Left||Right
    (NO outer tag); embedded sides byte-identical to standalone expression artifacts.
D27 AST tags: 0x01 Symbol, 0x02 Rational, 0x03 Add, 0x04 Mul, 0x05 Neg, 0x06 Pow, 0x07 Sqrt,
    0x08 Call, 0x09 Relation, 0x0A BranchSet. Unknown tags rejected.
D28 BigInt zero = 0x00 || UInt32BE(0), no magnitude bytes; no leading-zero magnitudes; no
    negative zero. ParameterID = SHA-256(String("physmath")||Bytes(ScopePathBytes)||
    UInt32BE(ordinal)); Function params depth 0, ScopePathBytes = UInt32BE(0) =
    EncodeBytes([]). BoundParameterRef = EncodeArtifactRef(decl)||Digest32(pid), no wrapper.
D29 Rule arity locked: DIFFERENTIATE-001 = exactly 2 inputs (expr + declaring Function);
    SUBSTITUTE-001 = 1+D (target + declaring Functions, sorted by ArtifactRef bytes; entries
    sorted by canonical entry bytes); APPLY-001 = exactly 2 (root-Call expr + referenced
    Function). Canonical identity = (ProducerNamespace,RuleID) pair; colon form is
    transport/diagnostic only. Unknown pairs → ErrUnknownRuleID, fail closed.
D30 fixture2 = FULLY CONFORMING MINIMAL framework (FrameworkArtifact + all six families,
    minimal content), test-only, isolated under internal/, never imported by production.
```

## 3. Deferred decisions (explicit boundaries)

```text
F1 Schwarzschild re-expression (sole executable home: phys-gr/solution/).
F2 Production qm/bohm/cst/cdt (no dirs, no stubs).
F3 Unevidenced core promotions (default: stay in gr; empty core PASSES).
F4 Two-implementation equivalence (Phase 1 proves single-impl determinism).
F5 Any new-module frozen-Phys integration (zero imports/references in Phase 1).
F6 Publication/tag/version strategy. F7 Sparse tensors + deferred-sort execution.
```

Ruled out: new PhysMath families/nodes; PhysicalContract; phys-equiv; second schemas;
dynamic AST extension; silent discharge; NodePath selectors; convenience promotion.

## 4. Exact repository topology

```text
/home/chaschel/Documents/go/
  go.work            (add use ./phys-artifact ./phys-math ./phys-lib; drop ./phys-gr at §16-G3)
  phys/              FROZEN — untouched (github.com/PithomLabs/phys)
  phys-gr/           HISTORICAL — untouched (github.com/PithomLabs/phys-gr)
  phys-artifact/     CREATE — module github.com/PithomLabs/phys-artifact
  phys-math/         exists empty — module github.com/PithomLabs/phys-math
  phys-lib/          exists empty — module github.com/PithomLabs/phys-lib (ONE module)
    core/doc.go                    (package core + docs, zero exports)
    classifications/<slug>.md      (D19 governance records; not API)
    gr/                            (canonical GR library)
      equivariance/{AGENTS.md,manifest.json,protocol/derivation_test.go}
    internal/conformance/fixture2/ (test-only; phys-fixture2; same nested protocol shape)
```

## 5. Dependency rules

Permitted partial order (allowed, not mandatory — gr MUST NOT be forced to import empty
core; no fake core abstraction to "complete" the graph): `phys-lib/<fw> → core → phys-math
→ phys-artifact`, plus direct `gr → phys-math | phys-artifact`. Prohibited: artifact→anything;
math→lib; core→frameworks; framework→framework Go imports; new→phys-gr; Phase-1 new→phys.
Enforced by import scanner + `go list` + `go mod graph` in CI.

## 6. Classification protocol

Seven classes; §2-D19 record format/location/CI; §2-D5 evidence bar; ambiguity → gr.
Coupling/cohesion/core-specific applied per decision and auditable in Phase 9.

## 7. Refactoring map (corrected)

Same rows as plan9.2 §7 with two corrected wordings: metric/connection/curvature/vacuum row
TARGET INTERFACE reads "PhysMath-expressible fragments project to PhysMath; the GR
computation pipeline (Christoffel/Ricci/vacuum/weak-field evaluation) is NOT
re-implemented — F1-deferred; disjoint gr-local algebra retained"; solution/ row CLASS =
DEFER (F1), TARGET = "preserved reference only; no Phase-1 re-expression". All other rows
(SPLIT mechanics, disjoint GRExpr + fallible projection, tensor validation split, replay
shape split, truncate/limit/coords → gr) stand as in plan9.2 §7.

## 8. Phase-5 GR scope (exact deliverable)

FrameworkArtifact (PackageNamespace "phys-gr", empty BaseFrameworkRefs); six families
populated ONLY with PhysMath-expressible content + D15 convention assumptions;
disjoint gr-local algebra covering fixture + validation needs; D2 bridge; §6
classification execution; category-A/B/C vectors (§9). Exit vectors minimum: framework
artifact serialization; six-family validation (+12 §21.1 negatives); gr-local algebra
behavior on copied vectors; projection success; projection failure. Pipeline results are
NOT exit vectors.

## 9. Ported-vector definition

(A) PhysMath-expressible: semantic content re-encoded under new canonical rules; NEW
bytes/refs pinned; legacy bytes are reference evidence only, never byte-compared.
(B) GR-local: package-local behavior tests, no invented families. (C) Projection:
explicit success/failure tests incl. Sin/Cos → ErrUnrepresentableProjection. Sources:
legacy symbolic/, coordinates/, tensor/, metric/, connection/, curvature/, vacuum/,
limit/ testdata (read-only).

## 10. Condition lifecycle

Durable `condition/1` materialized by fixture/framework code → ArtifactRef supplied in
`OperationInvocation.ConditionRefs` (arity-exempt) → rule checks exact predicate
(e.g. NEQ(x,0) byte-identical for Pow(x,0)) → success or fail-closed → gr may reference
the existing condition from a PhysicalAssumption (D15 Category as applicable). Executor
creates nothing durable; TransformResult.ConditionRefs = normalized supplied set.

## 11. Equivariance fixture (exact)

Location §2-D8. Realization f(v)=2v; rho_X(g,v)=g·v; rho_Y(g,v)=g·v. Structures
LHS_0=Call(f,[Call(rho_X,[g,x])]); staged LHS_1=Call(f,[g·x]); RHS_0=Call(rho_Y,[g,Call(f,[x])]);
staged RHS_1=Call(rho_Y,[g,2·x]). Six steps exactly: 0 APPLY inner-rho_X→g·x;
1 APPLY staged-LHS_1→2·(g·x) [linkage: arg bytes == Step-0 bytes]; 2 CANON→2·g·x;
3 APPLY inner-f→2·x; 4 APPLY staged-RHS_1→g·(2·x) [linkage: arg2 == Step-3 bytes];
5 CANON→2·g·x. Final-bytes equality gate → relation (§2-D26) → Equation → Theorem +
PhysicalTheorem (assumption + definition/interpretation + FrameworkArtifact refs).
No SIMPLIFY-for-CANON substitution, no reordering, no Map-for-f, no durable
FunctionApplication LHS/RHS, no distributivity, no GroupAction, no trig. rho_X/rho_Y
canonical coincidence documented intentional (distinction in interpretation). Golden
refs: every step input/output, relation, equation, theorem, all gr-side artifacts.

## 12. fixture2 conformance model

`phys-fixture2`, fully conforming minimal (D30): FrameworkArtifact (FrameworkID
"fixture2", version "1", empty bases) + minimal assumption/definition/equation/theorem/
interpretation over same-or-overlapping PhysMath refs with different meaning. Proves:
PhysMath identity unchanged; framework/assumption/interpretation distinct; no winner;
unrelated-assumption refs rejected; self/cyclic deps rejected. Isolation: no production
import path; compiler-enforced via internal/.

## 13. Protocol migration

Reuse actual `solution/manifest.json` (15 keys), `manifest_test.go` (embed/strict/
unknown-reject/presence/prose-check), `derivation_test.go` (ordered expectations,
firewalls, M1–M8-style mutation, independence), `replay/` chain semantics, AGENTS.md §8.1
contract. New nested packages mirror semantics; flat→nested equivalence noted in fixture
AGENTS.md. Manifest = metadata, never premise (reverse firewall test). Inexpressible
need → STOP, PROTOCOL GAP report, no invention.

## 14. Acceptance/test matrix

20 gates (mechanical CI unless marked REVIEW): [1] import scan [2] namespace separation
[3] artifact purity [4] math purity [5] core-empty/records [6] disjointness+projection-fail
[7] phys-gr tree-hash oracle [8] zero phys-gr imports [9] zero phys imports [10] ParameterID
vectors+alpha-rename+ordinal [11] framing/BigInt edges [12] rule closure+arity/family matrix
[13] step family validation [14] condition threading+bridge [15] staged linkage [16] six-step
goldens [17] fixture2 isolation [18] per-row records [19] manifest schema-diff vs GR original
[20] N×N repeated-replay determinism. REVIEW: mathematician test, convenience-audit, ownership.

## 15. Migration gates

G1 (Ph0): go.work resolves; tree hashes of phys/ + phys-gr/ recorded as oracles.
G2 (Ph5): §8 vectors pass; §6 records complete. G3 (Ph10): grep+go list+mod graph+tests
prove zero phys-gr/phys imports; drop ./phys-gr from go.work; both legacy tree hashes
unchanged; legacy phys-gr suite still green standalone.

## 16. Risks/failure modes

Dumping-ground → D19 gate + empty-start + default-gr. Convenience classification → audit.
GR→math leakage → disjointness + scans + D1. Math→physics → review + §25.7. Core
contamination → theory-name scan. JSON-as-identity → codec split + canonical equality.
Drift → schema-diff + equivalence note. Linkage spoof → pre-validation. PID collision →
D21 enforcement. Workspace drift → lint. Overstated determinism → D14 wording. Review
inventions → R1/R7/D-series rejections recorded above.

## 17. Implementation sequence + stop policy

Phases 0–10 per mandate §35 (Phase 4 scaffolding-only; Phase 5 bounded by §8; Phase 6
nested fixture; Phase 7 exact slice; Phase 8 fixture2; Phase 9 full suite + audits;
Phase 10 G3). PERMITTED DETAIL (no asking): file/helper/constructor names, internal
structures, test organization, error var names (semantics unchanged), CI shell mechanics,
caching, allocation, benchmarks. ASK ONLY on: ownership, dependency direction, canonical
identity/serialization, durable schema/family, physical semantics, PhysMath vocabulary,
rule/protocol semantics, framework boundary, Phase-1 scope, preservation boundary,
core/specific classification. Procedure: stop affected behavior only; file problem +
spec/plan section + repo evidence + competing readings + consequences + recommendation;
continue unaffected work; resume on answer. Never open a new planning revision for detail.

## 18. Handoff checklist

Provenance (spec + GR files + replay) read; phases executed in order with exits; D1–D30
applied; F1–F7 untouched; ruled-out list clean; gaps use the §17 blocker format.

```text
STATUS: CLOSED FOR IMPLEMENTATION
No known architectural blockers remain.
No known implementation-critical architectural ambiguities remain.
Implementation begins now.
```
