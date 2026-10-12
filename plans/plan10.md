# Remediation Execution Plan — Phase-1 Adversarial Findings (plan10)

Corrective only. No redesign, no new planning revision, no architectural invention.
Authority hierarchy:
1. PM-REP-001 v1.2
2. plan9.3 closed decisions
3. phys-system/AGENTS.md for agent operating discipline
4. plan10 remediation instructions
5. adversarial findings as defect reports

`phys/` and `phys-gr/` are frozen — never modified, never imported.

## 0. Operating protocol

Before changing code, read `phys-system/AGENTS.md` (verified present, 911 lines).
Follow its semantic-liveness, minting-invariant, falsification, and boundary
rules. Do not modify the legacy `phys/AGENTS.md`.

## 1. Root pointer (new file)

Create `/home/chaschel/Documents/go/AGENTS.md`, minimal pointer only: work on
`phys-artifact`/`phys-math`/`phys-lib`/cross-repo architecture → read
`phys-system/AGENTS.md` first; package-local `AGENTS.md` files govern local
details; frozen `phys/AGENTS.md` governs the legacy system.

## 2. H-1 + M-3 — unified transitive framework closure (`phys-lib/gr/`)

Replace the one-level `InClosure`/`CheckClosure` with a single coherent
transitive implementation: resolve base refs through the supplied families
map, recurse with visited-set cycle detection, reject dangling refs,
non-`framework/1` targets, malformed ownership, and self-reference (same
identity-based rule as `fixture2`; helper stays `gr`-owned, never promoted
to `core`). Mirror equivalent depth/cycle coverage in `fixture2`.

Tests: depth 0 / self, depth 1 / direct base, depth 2 / base-of-base, depth 3,
dangling base, wrong family, wrong namespace, self-cycle, two-node cycle,
three-node transitive cycle, valid two-level chain admitted through the
deepest base namespace.

## 3. H-2 — durable pre-mint validation

Add `Validate()` to `Theorem`, `Derivation`, `Equation`, `MathCondition`,
`FunctionApplication` in `phys-math/families.go` (structural layer: required
refs present, correct families/namespaces per §5.7, uniqueness, schema
correctness); complete partial `OperatorApplication`/`Transformation`
validators where the family contract requires. Gate every `store.Put` mint
path (fixture code included — audit all 10 `Put` sites in `fixture.go`) as
construct → Validate → Encode → Put. Theorem rejects `ConclusionRef →
expression/1` and dangling refs. No validation rules beyond PM-REP/family
semantics.

## 4. M-1 — duplicate `ConditionRefs`

`Derivation.ValidateStructure` rejects duplicate step `ConditionRefs` via the
existing `checkDuplicateRefs` SET semantics. Tests: unique → PASS, duplicate →
FAIL, reordered → unchanged. Encoding semantics untouched.

## 5. M-2 — deep-copy `BigInt.Mag`

Deep-copy magnitudes in `cloneExpr`'s default case:
`cp.Numerator.Mag = append([]byte(nil), ...)` (same for Denominator). Test
mutates the clone's magnitude and proves the original unchanged; golden bytes
re-verified identical. Canonical encoding/identity untouched.

## 6. L-3 / L-4 — test hygiene and independence

L-3: remove the dummy `var _` assignments (`rules_test.go`, `identity_test.go`).
L-4: replace the implementation-derived `NEQ(x,0)` expectation with
independently hardcoded canonical bytes hand-derived from the spec framing,
labeled SPEC-DERIVED; keep the wrong-variable negative. Condition semantics
unchanged.

## 7. H-3 — equivariance semantic correctness (investigate, do not invent)

Intended claim `f(rho_X(g,x)) = rho_Y(g,f(x))` (LHS0/RHS0) is currently
disconnected from the persisted `Step2 = Step5` theorem. Trace whether
PM-REP/plan9.3 provide an existing durable structure binding the original
sides to the derivation and conclusion. Add the semantic-liveness test
(mutate/disconnect LHS0/RHS0/declarations while holding Step2/Step5 fixed →
must fail) plus mutation categories: original-side mutation, staged
disconnects, declaration replacement. Never weaken `CheckTheoremConclusion`,
replay, or theorem equality to make it pass.

H-3 semantic invariant:

The persisted theorem must have a traceable correspondence from its
conclusion sides to the two sides of the claimed proposition.

It is insufficient that:
    normalize(LHS0) == normalize(RHS0)

The verifier must establish:
    LHS0  → declared left derivation/result  → conclusion.LHS
    RHS0  → declared right derivation/result → conclusion.RHS

No theorem-side artifact may be semantically dead.

Adversarial test: construct a mutant in which LHS0 and/or RHS0 is replaced
by a different valid expression while Step2 and Step5 remain unchanged. The
verification MUST fail. If the current protocol has no mechanism by which
this can fail, report PROTOCOL GAP / SEMANTIC GAP rather than inventing one.

Stop condition: if no existing mechanism encodes the correspondence, stop
H-3 only and file the `PROTOCOL GAP / SEMANTIC GAP` report (claim, current
representation, mechanisms considered, why insufficient, resolutions A/B/C +
recommendation), then ask the architect. Continue H-1/H-2/M-1/M-2/M-3
unaffected. Do not invent families, nodes, rules, schemas, fields, or side
channels.

## 8. Regression and report

Full suites + vet + dep/import/namespace scans + golden/determinism
re-verification + legacy hash re-check (`acf0259f…`, `9885a03f…`); no `replace`
directives. Final report per the remediation prompt §10: per-finding evidence,
semantic invariants, exact test commands/results, files changed, explicit
architectural-impact statement, genuine remaining questions only. H-3 is fixed
only when the fixture establishes the intended proposition, not merely final
normal-form equality.
