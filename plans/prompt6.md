Use this as the next implementation-agent prompt. The key change is that this is now **final hardening/closure work**, not another review or redesign cycle. The second adversarial review found no architectural violations; H-1/H-2/M-1/M-2/M-3/H-3 are reported fixed, with M-4 and a small test-harness/API cleanup remaining. adv_review2 adv_review2

---

# Final Phase-1 Hardening and Closure Prompt

You are implementing the **final targeted hardening and closure work** after the Phase-1 scope adversarial review.

This is **not a redesign** and **not another planning cycle**.

The Phase-1 architecture is already closed and the remediation has passed scope review with **zero architectural violations**. Your job is to resolve only the remaining items below, verify H-3 at its strongest level, and then close Phase 1.

## Authority

Use this authority hierarchy:

```text
1. PM-REP-001 v1.2
2. plan9.3 closed decisions
3. phys-system/AGENTS.md
4. plan10 remediation contract
5. adversarial review findings
6. current implementation
```

Do not let a review recommendation override a closed architectural decision.

---

# 1. Hard constraints

Preserve all existing Phase-1 invariants.

Do not:

- modify `/home/chaschel/Documents/go/phys/`
- modify `/home/chaschel/Documents/go/phys-gr/`
- import `phys-gr`
- add `replace` directives
- add PhysMath AST nodes
- add PhysMath durable families
- add new transformation rules
- add `Equivariance`, `Group`, or `GroupAction` ontology to `phys-math`
- add physical semantics to `phys-math`
- promote anything into `phys-lib/core`
- introduce a second protocol schema
- implement deferred GR pipelines
- add future production `qm`, `bohm`, `cst`, `cdt`
- create another planning document

The existing architecture remains authoritative.

---

# 2. M-4 — formally defer, do not "fix" with replace

The scope review found that:

```text
phys-lib/go.mod
```

still contains the pseudo-version of `phys-math`, causing standalone:

```text
go list -m all
```

to fail outside the workspace.

The review recommended either `replace` or formal deferral. adv_review2

We already decided that `replace` is **not permitted in Phase 1** under D17.

Therefore:

### Do NOT add:

```go
replace github.com/PithomLabs/phys-math => ../phys-math
```

Instead, formally classify M-4 as:

```text
DEFERRED — OUTSIDE PHASE-1 WORKSPACE DELIVERY
```

Document the rationale in the appropriate Phase-1 documentation/remediation record:

- Phase 1 uses `go.work` for local multi-module development.
- Publication/version strategy is deferred.
- Standalone module resolution is a publication concern, not a Phase-1 architecture failure.
- No `replace` directive should be introduced solely to satisfy this review.

Do not pretend M-4 was fixed.

The final report must explicitly say:

```text
M-4: DEFERRED under D17; not implemented.
```

---

# 3. S-1 — fixture2 duplicated closure traversal

The scope review identified duplicated transitive closure traversal between:

```text
phys-lib/gr/framework.go
phys-lib/internal/conformance/fixture2/framework.go
```

but also confirmed that fixture2 is test-only and this is not an architectural violation. adv_review2

Do **not** extract this into `phys-lib/core` merely to remove duplication.

Preferred action:

- keep the fixture2 implementation isolated;
- add a concise code comment explaining that the duplicated traversal is intentional test-side isolation and must remain independent from `gr`.

Do not introduce a shared production abstraction.

Reclassify this internally as:

```text
OBSERVATION — TEST-HARNESS DUPLICATION
```

not architectural scope creep.

---

# 4. Optional test/API cleanup — reduce unnecessary exported test surface

The review identified:

```text
VerifyClaim(*Fixture)
InnerRho
InnerFx
```

as additional exported API that is not required by the closed architecture. adv_review2

Inspect whether these can safely become package-private by moving the relevant protocol tests into the `equivariance` package instead of `equivariance_test`.

Preferred result:

```text
verify claim machinery → private
fixture linkage fields → private
```

while retaining genuine test independence.

Do this only if it does not weaken the independent protocol-test boundary.

Do not expose new API merely for convenience.

If reducing the API would require an architectural change, leave it alone and report it as harmless test-surface debt.

---

# 5. Final H-3 verification — strongest remaining check

H-3 was fixed using existing staged-linkage plus verifier-trace mechanisms. The scope review found that:

```text
LHS0 → left trace → Step2 → conclusion.LHS
RHS0 → right trace → Step5 → conclusion.RHS
```

is now checked, and source-side mutations fail. adv_review2

Do not redesign this.

However, perform one **final portability/information-source test**:

> Can the semantic correspondence be verified from the persisted artifacts and the existing verifier machinery, without relying on the mutable in-memory `Fixture` object as the sole source of the original theorem proposition?

The desired property is:

```text
persisted artifacts
    ↓
recover original proposition
    ↓
recover derivation
    ↓
recover left/right trace
    ↓
verify conclusion correspondence
```

The point is to distinguish:

```text
fixture-memory witness
```

from:

```text
persisted protocol semantics
```

### Required investigation

Inspect whether the existing durable artifacts already contain enough information to reconstruct:

```text
LHS0 = f(rho_X(g,x))
RHS0 = rho_Y(g,f(x))
```

and their correspondence to the two final results.

Do not assume the answer is no.

Do not invent a new schema if the existing artifacts already suffice.

Do not add:

- new durable fields
- new durable family
- new theorem type
- new rule
- new protocol schema
- hidden side channel

unless the specification explicitly authorizes it.

### If existing artifacts are sufficient

Implement the strongest possible verifier using the existing structures.

Add a test proving that a fresh verifier can reconstruct the claim without depending on `Fixture`-only fields.

Then test:

```text
mutated LHS0
mutated RHS0
broken LHS staged linkage
broken RHS staged linkage
changed declaration
unchanged final Step2/Step5
```

All must fail.

### If existing artifacts are not sufficient

Do **not** invent a protocol extension.

Report:

```text
PROTOCOL GAP / SEMANTIC GAP

Persisted information available:
...

Missing correspondence:
...

Why existing verifier cannot reconstruct it:
...

Existing mechanisms considered:
...

Recommended resolution:
...
```

But before declaring a gap, exhaust the existing:

- theorem
- derivation
- premise refs
- step inputs/outputs
- staged-call artifacts
- Function artifacts
- Symbol/ParameterID identity
- existing protocol linkage

The current review believes H-3 is already solved; the purpose of this task is simply to determine whether that solution is also independently reconstructible from persisted information. adv_review2

---

# 6. Verify that H-3 did not introduce an accidental new ontology

Inspect:

```text
phys-lib/gr/equivariance/claim.go
phys-lib/gr/equivariance/protocol/claim_test.go
```

Confirm that `VerifyClaim` remains:

```text
in-memory verification over existing artifacts
```

and is not secretly:

```text
new semantic model
new protocol layer
new durable representation
```

The scope review explicitly found no new protocol ontology. Preserve that result. adv_review2

---

# 7. Regression verification

After the above work, run the complete Phase-1 verification set.

At minimum:

```text
go test -count=1 ./...
go vet ./...
go list -deps ./...
go mod graph
```

Then run:

- import isolation scans
- namespace checks
- PhysMath purity checks
- empty-core check
- classification governance
- all 22 equivariance goldens
- fixture2 isolation/conformance
- replay mutation negatives
- H-3 semantic-liveness mutations
- deterministic repeated builds
- legacy preservation/oracle checks

Verify:

```text
phys-artifact     PASS
phys-math         PASS
phys-lib          PASS
legacy phys       unchanged
legacy phys-gr    unchanged
```

The previous review reported all 22 equivariance goldens unchanged; preserve that invariant unless a test is demonstrably correcting an invalid artifact rather than causing identity drift. adv_review2

---

# 8. Final semantic invariants

Before declaring closure, verify all of these explicitly:

```text
[ ] invalid durable objects cannot enter Store
[ ] framework closure is transitive
[ ] framework cycles rejected at any depth
[ ] ConditionRefs duplicates rejected structurally
[ ] cloned BigInts are structurally independent
[ ] original theorem proposition is semantically live
[ ] final equality never substitutes for proposition preservation
[ ] LHS0/RHS0 cannot become irrelevant while final bytes stay fixed
[ ] no new PhysMath ontology introduced
[ ] no new protocol schema introduced
[ ] core remains empty
[ ] no framework coupling introduced
[ ] no Phase-2 capability implemented
[ ] canonical identities preserved
[ ] frozen repositories untouched
```

The scope review already verified these architectural invariants broadly; this step is to ensure the final hardening has not disturbed them. adv_review2

---

# 9. Final report

Do not create another plan.

Return exactly:

```text
# Phase-1 Final Hardening Report

STATUS:
[COMPLETE / H-3 GAP FOUND]

## M-4
Status:
DEFERRED

Reason:
...

## S-1
Status:
OBSERVATION ONLY

Action:
...

## Test/API Surface
Status:
[reduced / retained]
Reason:
...

## H-3
Status:
[VERIFIED / GAP]

Persisted-artifact reconstruction:
...

Semantic chain:
LHS0 → ...
RHS0 → ...

Conclusion binding:
...

Mutations:
...

Does final equality substitute for proposition preservation?
NO

Does verification depend solely on Fixture-memory state?
[YES / NO]

Does claim.go introduce a new ontology?
NO

## Regression
[exact commands and results]

## Scope Invariants
[all 14 checks]

## Architectural Impact
No closed contract changed.

## Remaining Questions
[only genuine unresolved protocol questions]
```

## Final completion rule

The Phase-1 effort is complete when:

```text
M-4 = explicitly deferred under D17
S-1 = documented test-harness duplication
H-3 = independently verified against persisted artifacts,
       OR a genuine protocol gap is reported
all regression gates = green
no architecture changed
```

Do not reopen the design.

Do not produce `plan11`.

Do not turn a documentation/test-harness concern into a new shared abstraction.

The objective is to **finish Phase 1 cleanly and leave a trustworthy foundation for Phase 2**, not to optimize the codebase indefinitely.