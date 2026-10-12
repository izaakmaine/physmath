# Scope Adversarial Review — Phase-1 Remediation

VERDICT:
PASS WITH SCOPE FINDINGS

REQUIRED FIXES PROPERLY SCOPED:
6

TEST HARDENING:
9

SCOPE CREEP:
1

ARCHITECTURAL VIOLATIONS:
0

## 1. Executive Assessment

The remediation fixed all reported architectural and correctness defects while staying strictly inside the closed Phase-1 architecture. No new durable families, AST nodes, rules, protocol schemas, framework couplings, or physical ontologies were introduced. `phys-math` remains pure. `core` remains empty. Legacy `phys/` and `phys-gr/` are untouched. The `claim.go`/`claim_test.go` additions are in-memory verification witnesses over existing durable structures, not a new semantic layer.

One scope finding remains: the original M-4 finding (`phys-lib/go.mod` pseudo-version breaking `go list -m all`) was **not remediated**, despite the remediation report claiming "all findings are complete." The `go.work`-based workflow masks this in-tree, but outside the workspace (e.g., clean CI) `phys-lib` is unresolvable.

## 2. Scope Findings

### S-1 SCOPE CREEP — Duplicated transitive-closure traversal in `fixture2`

Location:
`/home/chaschel/Documents/go/phys-lib/internal/conformance/fixture2/framework.go:163-200`

Original requirement:
`fixture2` should demonstrate framework independence without sharing code with `phys-lib/gr`.

Observed change:
`fixture2/framework.go` implements `transitiveBaseNamespaces`, a near-identical copy of `gr/framework.go:closureNamespaces`. Both perform cycle-safe transitive base-framework traversal.

Why this is / is not within Phase-1 scope:
The duplication lives in `internal/conformance`, which is test-only, so it does not pollute production code or create a shared abstraction. However, it is unnecessary code growth that future maintainers must keep in sync. If a future framework copy-pastes this pattern into production, the duplication becomes an architectural liability.

Evidence:
`fixture2/framework.go:165-200` mirrors `gr/framework.go:137-166` with only namespace/error-message differences.

Recommended resolution:
Extract a shared unexported helper only if the pattern escapes test packages. Otherwise, add a comment noting the intentional duplication.

---

### S-2 SCOPE FINDING — M-4 not remediated

Location:
`/home/chaschel/Documents/go/phys-lib/go.mod`

Original requirement:
M-4 required adding a `replace` directive so `go list -m all` succeeds outside the workspace.

Observed change:
No `replace` directive was added. `go list -m all` still fails with "Repository not found" for the pseudo-version.

Why this is / is not within Phase-1 scope:
The remediation report claims "all findings are complete" and explicitly notes "no replace directives." This is internally inconsistent: M-4 could only be fixed by introducing a replace directive or changing the version strategy. The current state works in-tree via `go.work` but breaks standard Go tooling in clean environments.

Evidence:
`go list -m all` output in `phys-artifact`, `phys-math`, and `phys-lib` all fail with the same pseudo-version resolution error.

Recommended resolution:
Either add `replace github.com/PithomLabs/phys-math => ../phys-math` to `phys-lib/go.mod`, or reclassify M-4 as intentionally deferred and document the rationale.

## 3. H-3 Semantic Verification

Original proposition:
\[
f(\rho_X(g,x)) = \rho_Y(g,f(x)).
\]

LHS0 trace:
```text
LHS0 = Call(F, [Call(RhoX, [g(P0), x(P1)])])
  → mustResolve(fx.LHS0)
  → requireCallShape(lhs0, fx.F, 1)
  → innerRho = lhs0.Args[0]
  → requireCallArgs(innerRho, fx.RhoX, [g(P0), x(P1)])
  → premiseRho = mustResolve(fx.InnerRho)
  → premiseRho.Equal(innerRho)  (byte-identity)
  → Step0 = APPLY(innerRho)
  → LHS1 = Call(F, [Step0-output])
  → Step1 = APPLY(LHS1)
  → Step2 = CANON(Step1)
  → conclusion.LHS == Step2
```

RHS0 trace:
```text
RHS0 = Call(RhoY, [g(P0), Call(F, [x(P1)])])
  → mustResolve(fx.RHS0)
  → requireCallShape(rhs0, fx.RhoY, 2)
  → requireSchematic(rhs0.Args[0], p0, "g")
  → innerF = rhs0.Args[1]
  → requireCallArgs(innerF, fx.F, [x(P1)])
  → premiseF = mustResolve(fx.InnerFx)
  → premiseF.Equal(innerF)  (byte-identity)
  → Step3 = APPLY(innerF)
  → RHS1 = Call(RhoY, [g, Step3-output])
  → Step4 = APPLY(RHS1)
  → Step5 = CANON(Step4)
  → conclusion.RHS == Step5
```

Conclusion binding:
`CheckTheoremConclusion` enforces `conclusion.LHS == Step2` and `conclusion.RHS == Step5`. `VerifyClaim` additionally enforces that `Step2` and `Step5` trace back to the original `LHS0`/`RHS0` proposition through byte-identical premise and staged-call linkage.

Mutation evidence:
- Mutate `LHS0` to a different expression while holding `Step2`/`Step5` unchanged: `VerifyClaim` fails at `requireCallShape` or `requireCallArgs` because the inner call structure no longer matches the declared `RhoX`.
- Mutate `RHS0` similarly: fails at the right-side symmetric checks.
- Disconnect `LHS1` from `Step0` output: fails at `requireCallArgs(lhs1, fx.F, [s0out])`.
- Disconnect `RHS1` from `Step3` output: fails at `requireCallArgs(rhs1, fx.RhoY, [g, s3out])`.
- Replace declaration `f`/`rho_X`/`rho_Y` with different `Function` artifacts while keeping final normal forms fixed: fails because `requireCallShape` checks exact `FuncRef` byte-identity.

Does Step2 = Step5 still substitute for proposition preservation?
**NO.** Normal-form equality is necessary but never sufficient. `VerifyClaim` proves that the specific `LHS0`/`RHS0` proposition is the one that closed, not merely that *some* proposition with the same normal form exists.

Does the implementation introduce a new protocol ontology?
**NO.** `VerifyClaim` is an in-memory structural witness over existing `Derivation`, `Theorem`, `Call`, and `Symbol` artifacts. It introduces no new durable family, serialization, or protocol schema.

## 4. Scope Invariants

- no new PhysMath families — VERIFIED
- no new AST nodes — VERIFIED
- no new rules — VERIFIED
- no physical leakage — VERIFIED
- core remains empty — VERIFIED
- no new framework coupling — VERIFIED
- no frozen-repo modification — VERIFIED
- no Phase-2 functionality — VERIFIED
- canonical identities preserved — VERIFIED (all 22 equivariance goldens unchanged)
- protocol schema not expanded without authorization — VERIFIED
- documentation remains documentation-only — VERIFIED

## 5. Newly Added Public API

| Identifier | Location | Required by closed architecture? | Rationale |
|---|---|---|---|
| `VerifyClaim(*Fixture) error` | `phys-lib/gr/equivariance/claim.go` | No (test-only) | External `protocol_test` package needs access to the claim-trace verification. Could not be unexported without moving tests into the `equivariance` package. |
| `InnerRho ArtifactRef` | `phys-lib/gr/equivariance/fixture.go` | No (test-only) | In-memory premise linkage for `VerifyClaim`. Not durable schema. |
| `InnerFx ArtifactRef` | `phys-lib/gr/equivariance/fixture.go` | No (test-only) | In-memory premise linkage for `VerifyClaim`. Not durable schema. |
| `FrameworkArtifact.CheckClosure(physmath.Store) error` | `phys-lib/gr/framework.go` | Yes (signature fix) | Old `(families map)` signature could not support transitive closure. Store-based resolution is necessary. |
| `FrameworkArtifact.InClosure(physmath.Store, FamilyID) bool` | `phys-lib/gr/framework.go` | Yes (signature fix) | Same rationale as `CheckClosure`. |
| `Theorem.Validate() error` | `phys-math/families.go` | Yes | Missing validation gate for durable `theorem/1`. |
| `Theorem.TargetChecks() []TargetCheck` | `phys-math/families.go` | Yes | Required by `ValidateAgainst` for theorem target checking. |
| `MathCondition.Validate() error` | `phys-math/families.go` | Yes | Missing validation gate for durable `condition/1`. |
| `MathCondition.TargetChecks() []TargetCheck` | `phys-math/families.go` | Yes | Required by `ValidateAgainst` for condition target checking. |
| `FunctionApplication.Validate() error` | `phys-math/families.go` | Yes | Missing validation gate for durable `function_application/1`. |
| `FunctionApplication.TargetChecks() []TargetCheck` | `phys-math/families.go` | Yes | Required by `ValidateAgainst` for function-application target checking. |

All new public identifiers are either:
1. Necessary to make an existing contract correct (`CheckClosure`/`InClosure` signature, validators), or
2. Test-only verification helpers in `internal/conformance` / `gr/equivariance`.

No new extension points or ontologies were created.

## 6. Deferred Work Accidentally Implemented

**NONE**

No Schwarzschild, Christoffel, Ricci, Einstein, vacuum, weak-field, numerical execution, sparse tensor, SessionCorrespondence, Bohm, causal-set, CDT, or two-implementation equivalence code was found.

`gr/seed.go` continues to seed only three convention assumptions (metric signature, curvature convention, cosmological constant). This is framework-local test data, not pipeline execution.

## 7. Final Recommendation

**PASS WITH SCOPE FINDINGS.** The remediation correctly fixed all six reported defects (H-1, H-2, H-3, M-1, M-2, M-3) and the two L findings (L-3, L-4) without introducing new architecture, new durable families, new AST nodes, new rules, or physical leakage into `phys-math`. The `claim.go` mechanism is an in-memory verification witness, not a new protocol layer. The equivariance fixture's theorem proposition is now semantically live end-to-end.

Two items require attention before Phase 2 closure:

1. **M-4 remains open.** `go list -m all` still fails. Either add the `replace` directive or formally defer M-4 with documented rationale.
2. **Minor API surface growth.** `VerifyClaim`, `InnerRho`, and `InnerFx` are exported in the `equivariance` package. They are justified by external-test access but could be reduced by moving protocol tests into the `equivariance` package.

No redesign is required. The architecture remains faithful to the closed Phase-1 specification.
