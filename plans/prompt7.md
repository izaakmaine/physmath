Yes. This should be the **last Phase-1 gate** before Phase 2. Unlike the earlier adversarial reviews, this one should be deliberately narrow: **verify that the completed remediation is actually clean and that nothing remains that would undermine the foundation for Phase 2**. It should not reopen decisions already settled.

Use this prompt:

---

# Final Scoped Adversarial Code Review — Phase-1 Closure Gate

You are performing the **final scoped adversarial code review before Phase 2 begins** for the completed Phase-1 implementation and remediation.

This is a **closure gate, not a redesign review**.

The implementation reports:

```text
PHASE 1: CLOSED
ARCHITECTURE: CLOSED
REMEDIATION: CLOSED

H-1  FIXED
H-2  FIXED
M-1  FIXED
M-2  FIXED
M-3  FIXED
H-3  VERIFIED

L-3  DONE
L-4  DONE

M-4  DEFERRED UNDER D17
S-1  OBSERVATION ONLY
```

The final remediation also reports:

- no protocol gap for H-3
- no new PhysMath ontology
- no new durable families
- no new AST nodes
- no new rules
- no physical leakage
- core remains empty
- no framework coupling
- no Phase-2 functionality
- frozen `phys/` and `phys-gr/` remain untouched
- 22 equivariance goldens unchanged
- all current tests/vet/import scans pass
- H-3 is verified through a fresh persisted-artifact verifier rather than Fixture-memory state

Your task is to **try to falsify those closure claims without reopening the architecture**.

---

# 1. Authority

Use this order:

```text
1. PM-REP-001 v1.2
2. plan9.3 closed decisions
3. phys-system/AGENTS.md
4. plan10 remediation contract
5. prior adversarial findings
6. current implementation
```

Do not reinterpret deferred items as bugs.

Do not turn an observation into architectural scope.

Do not invent new requirements.

---

# 2. Review objective

Answer exactly this:

> **Is Phase 1 actually safe to close and hand over to Phase 2, with no remaining correctness defect, semantic-integrity hole, scope violation, or hidden architectural drift that would materially compromise future work?**

This is the final confidence gate.

You are specifically looking for:

```text
regression introduced by remediation
false claim of closure
semantic hole still surviving
test that passes for the wrong reason
new accidental dependency
new accidental API/ontology
scope leakage
inconsistency between documentation and implementation
```

Do not spend time on style, naming, micro-optimization, or cosmetic cleanup.

---

# 3. Verify the remediation changed only what it claims

Inspect the final diff from the Phase-1 pre-remediation baseline.

Classify every changed file as one of:

```text
REQUIRED FIX
TEST HARDENING
DOCUMENTATION
AUTHORIZED TEST-SUPPORT
UNNECESSARY BUT HARMLESS
SCOPE CREEP
ARCHITECTURAL VIOLATION
```

The expected remediation surface is approximately:

```text
phys-lib/gr/
phys-lib/internal/conformance/fixture2/
phys-lib/gr/equivariance/
phys-math/
phys-system/AGENTS.md
/home/chaschel/Documents/go/AGENTS.md
phys-lib/go.mod       # documentation/comment only for M-4
```

Anything outside the expected remediation area must be explained.

---

# 4. H-1 / M-3 final closure

Verify framework closure is genuinely transitive.

Test:

```text
F
├── A
│   └── B
│       └── C
```

and verify all of:

```text
F
A
B
C
```

are recognized by closure where appropriate.

Then verify rejection of:

```text
self-reference
F → A → F
F → A → B → F
dangling base
wrong family
wrong namespace
```

Confirm `CheckClosure` and `InClosure` are based on coherent traversal semantics.

Confirm no accidental shared framework abstraction was promoted into `core`.

Confirm fixture2 remains intentionally independent rather than sharing the GR implementation.

---

# 5. H-2 final minting-integrity audit

Search every durable `Store.Put` path again.

Verify the invariant:

```text
construct
→ Validate
→ Encode
→ Put
```

for every durable family that requires validation.

Specifically inspect:

```text
Theorem
Derivation
Equation
MathCondition
FunctionApplication
OperatorApplication
Transformation
```

Verify that validation is not:

- partial where the family requires more
- bypassable through another constructor
- bypassable through a fixture helper
- bypassable through an alternate `Put` path
- circular/self-validating

Try malformed durable artifacts and verify they cannot enter the store.

Especially attempt:

```text
Theorem.ConclusionRef → expression/1
dangling references
wrong family
wrong namespace
duplicate SET members
malformed embedded references
```

---

# 6. M-1 / M-2 final checks

Verify:

```text
duplicate DerivationStep.ConditionRefs → rejected structurally
```

and ensure encoding and validation agree.

For `cloneExpr`, verify cloned `BigInt.Mag` values are independent.

Perform a mutation test:

```text
clone
→ mutate clone magnitude
→ original remains unchanged
→ original canonical bytes unchanged
```

Also verify no golden or canonical identity drift was introduced.

---

# 7. H-3 — final semantic integrity gate

This is the **highest-priority review section**.

The intended proposition is:

\[
f(\rho_X(g,x)) = \rho_Y(g,f(x)).
\]

Verify that the final implementation proves the proposition itself, not merely:

\[
Step2 = Step5.
\]

The final claimed semantic chain is:

```text
LHS0
 → innerRho
 → Step0
 → LHS1
 → Step1
 → Step2
 → conclusion.LHS

RHS0
 → innerF
 → Step3
 → RHS1
 → Step4
 → Step5
 → conclusion.RHS
```

Verify this independently.

---

## 7.1 Fresh-verifier requirement

The fresh H-3 verifier must use only persisted information necessary to reconstruct the claim.

It must not depend on:

```text
Fixture.LHS0
Fixture.RHS0
Fixture.InnerRho
Fixture.InnerFx
```

or any other Fixture-only semantic state.

Construct the strongest possible test:

```text
Build fixture
    ↓
retain only store + durable refs
    ↓
discard Fixture semantic fields
    ↓
fresh verifier
    ↓
reconstruct LHS0/RHS0
    ↓
reconstruct trace
    ↓
replay
    ↓
verify conclusion
```

Verify the manifest remains metadata only.

It must not become a hidden mathematical premise.

---

## 7.2 H-3 mutation battery

Run all of these against **persisted representation**, not just in-memory Fixture fields:

```text
mutate LHS0
mutate RHS0
break LHS staged linkage
break RHS staged linkage
replace f declaration
replace rho_X declaration
replace rho_Y declaration
alter ParameterID
alter display identity where identity-bearing
keep Step2 unchanged
keep Step5 unchanged
```

Every semantically invalid mutation must fail.

The crucial attack is:

> **Can the verifier still succeed when the original proposition is changed but the final normal forms remain identical?**

The answer must be **NO**.

---

## 7.3 Verify both directions independently

Confirm that the verifier doesn't accidentally prove only:

```text
LHS0 → Step2
```

while trusting the RHS, or vice versa.

Both sides must be reconstructed and independently linked.

---

## 7.4 Check for circular H-3 validation

Determine whether:

```text
production fixture
    ↓
claim verifier
```

is simply validating facts that the fixture itself wrote into memory.

The persisted-artifact verifier must establish semantic correspondence independently enough that a broken fixture cannot make itself look correct merely by supplying its own metadata.

---

# 8. Verify no H-3 ontology leaked into PhysMath

Search remediation changes for:

```text
Equivariance
Group
GroupAction
PhysicalClaim
Proposition
ProofObligation
TraceWitness
```

Determine whether any such concept became:

- exported PhysMath API
- durable family
- AST node
- new serialization
- generic core abstraction
- hidden protocol schema

Expected result:

```text
NO
```

`equivariance` remains a framework/test fixture concept assembled from generic PhysMath artifacts.

---

# 9. Verify PhysMath purity after remediation

Inspect the final exported `phys-math` API and implementation.

Verify no remediation change introduced:

```text
GR semantics
physical assumptions
physical interpretations
metric semantics
curvature semantics
framework identity
physical ontology
theory-specific function IDs
```

The question is:

> Could a new future framework use PhysMath without knowing GR exists?

Expected result:

```text
YES
```

---

# 10. Verify core remains empty

Confirm:

```text
phys-lib/core/doc.go
```

still has zero exported domain symbols.

Search for remediation code accidentally moved into core:

```text
closure
claim
validation registry
framework registry
shared theory abstraction
```

Expected result:

```text
NONE
```

---

# 11. Verify framework independence

Re-run the conceptual two-framework test:

```text
phys-lib/gr
        ↕
    phys-math
        ↕
fixture2
```

Confirm fixture2:

- has its own namespace
- has its own framework identity
- does not import GR
- does not reuse GR production helpers
- can reuse identical mathematical artifacts
- does not alter PhysMath identity
- does not require global Category vocabulary

Verify the same mathematics can coexist with distinct framework meaning.

---

# 12. Verify protocol integrity

Inspect:

```text
phys-lib/gr/equivariance/
```

including:

```text
AGENTS.md
manifest.json
protocol/derivation_test.go
protocol/claim_test.go
```

Verify:

- manifest is metadata
- strict decode remains in place
- unknown fields are rejected
- formal derivation comes from artifact data
- nested protocol layout remains D8-compliant
- no second schema was introduced
- H-3 claim verification uses existing durable structures
- no test shortcut bypasses replay

---

# 13. Verify documentation/protocol hierarchy

Confirm:

```text
/home/chaschel/Documents/go/AGENTS.md
        ↓
phys-system/AGENTS.md
        ↓
package-local AGENTS.md
```

and verify:

- root `AGENTS.md` is only a pointer
- `phys-system/AGENTS.md` is documentation-only
- no production code imports documentation
- frozen `/phys/AGENTS.md` remains untouched
- no contradictions were introduced between collective and local protocols

The collective protocol should now encode the lessons of Phase 1:

```text
semantic liveness
claim preservation
durable minting invariant
independent golden evidence
falsification
semantic contradiction stop condition
structural + semantic verification
```

---

# 14. Verify deferred M-4 is correctly represented

Do **not** attempt to fix M-4 with `replace`.

Verify:

```text
M-4 = DEFERRED UNDER D17
```

and that documentation does not falsely say M-4 was implemented.

The current workspace model should remain:

```text
go.work
    → local multi-module development
```

while standalone publication/version resolution remains deferred.

This is not a Phase-1 failure.

---

# 15. Verify S-1 remains harmless

Confirm fixture2's duplicated closure traversal is:

```text
test-side
isolated
intentional
documented
not promoted to core
```

Treat it as an observation, not an architectural defect.

---

# 16. Verify no deferred Phase-2 capability leaked in

Search the final implementation for accidental implementation of:

```text
Schwarzschild computation
Christoffel pipeline
Ricci pipeline
Einstein/vacuum pipeline
weak-field pipeline
SessionCorrespondence
numerical execution
sparse tensor execution
Bohmian package
QM package
CST package
CDT package
two-implementation equivalence
publication/version architecture
```

Distinguish textual references from actual executable capability.

Expected result:

```text
NONE
```

---

# 17. Re-run the Phase-1 integrity battery

Run:

```text
go test -count=1 ./...
go vet ./...
go list -deps ./...
go mod graph
```

Then:

- namespace scans
- import scans
- purity scans
- empty-core test
- classification governance
- all 22 equivariance goldens
- fixture2 isolation
- replay mutation suite
- H-3 persisted-artifact mutation suite
- deterministic repeated builds
- legacy preservation verification

Repeat the important suite multiple times where nondeterminism could matter.

---

# 18. Golden integrity

Verify:

```text
22 equivariance goldens
```

remain unchanged.

For any golden that changed, determine whether it is:

```text
intentional correction of invalid evidence
```

or:

```text
unexpected canonical drift
```

Do not accept silent drift.

Also verify the hardcoded L-4 condition expectation remains independent of production encoding.

---

# 19. Test-quality adversarial attack

For every important new remediation test ask:

> Would this test have failed against the pre-remediation implementation?

Especially verify:

```text
H-1 tests
H-2 tests
H-3 semantic tests
L-4 independent golden
```

Reject tests that simply reproduce production behavior.

---

# 20. Final closure criteria

Phase 1 may be declared finally closed only if all are true:

```text
[ ] H-1 genuinely fixed
[ ] H-2 genuinely fixed
[ ] M-1 genuinely fixed
[ ] M-2 genuinely fixed
[ ] M-3 genuinely fixed
[ ] H-3 genuinely verified
[ ] M-4 explicitly deferred under D17
[ ] S-1 remains test-only observation
[ ] no new architecture
[ ] no new PhysMath ontology
[ ] no new durable families
[ ] no new rules
[ ] no physical leakage
[ ] core remains empty
[ ] no framework coupling
[ ] no Phase-2 functionality
[ ] canonical identities preserved
[ ] H-3 persisted-artifact verification works
[ ] semantic mutations fail
[ ] test evidence is not circular
[ ] frozen repositories remain untouched
[ ] documentation hierarchy is coherent
```

---

# 21. Required final report

Return exactly:

```text
# Final Phase-1 Closure Review

VERDICT:
[PASS / PASS WITH FINDINGS / BLOCKED]

ARCHITECTURAL BLOCKERS:
[number]

CORRECTNESS FINDINGS:
[number]

SEMANTIC FINDINGS:
[number]

SCOPE FINDINGS:
[number]

## 1. Closure Assessment
...

## 2. H-1/H-2/M-1/M-2/M-3
[final disposition for each]

## 3. H-3 Semantic Verification

Original proposition:
...

Fresh persisted-artifact verifier:
...

LHS trace:
...

RHS trace:
...

Conclusion binding:
...

Persisted-artifact mutation results:
...

Does final equality substitute for proposition preservation?
NO

Does the verifier depend on Fixture-memory semantic state?
NO

## 4. Scope Integrity
...

## 5. PhysMath Purity
...

## 6. Framework Independence
...

## 7. Protocol Integrity
...

## 8. Test Independence
...

## 9. Deferred Items
M-4: DEFERRED UNDER D17
S-1: OBSERVATION ONLY

## 10. Regression Evidence
[exact commands + results]

## 11. Phase-2 Readiness
[READY / NOT READY]

## 12. Final Recommendation
...
```

## Critical rule

**Do not fix anything during this review.**

This is a verification gate.

If you find a defect, report it with:

```text
exact location
reproduction
architectural/semantic significance
smallest corrective action
```

Do not create another plan.

Do not reopen closed architectural decisions.

Do not convert observations into redesigns.

The desired outcome is:

```text
PHASE 1 VERIFIED CLOSED
        ↓
PHASE 2 READY
```

not another cycle of indefinite hardening.