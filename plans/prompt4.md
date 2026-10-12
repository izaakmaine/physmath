Yes. I would make the remediation prompt **strictly corrective, not exploratory**: four findings are localized implementation fixes; H-3 is a semantic/protocol issue and must not be “fixed” by inventing a new theorem mechanism.

The prompt below also makes `phys-system/AGENTS.md` the new collective operating protocol.

---

# Remediation Prompt — Phase-1 Adversarial Findings

You are now remediating the Phase-1 implementation of:

```text
/home/chaschel/Documents/go/
├── phys-artifact/
├── phys-math/
├── phys-lib/
│   ├── core/
│   └── gr/
├── phys-system/
│   └── AGENTS.md
├── phys/       # FROZEN — DO NOT MODIFY
└── phys-gr/    # HISTORICAL — DO NOT MODIFY
```

This is **not a redesign task**. The Phase-1 architecture has already passed adversarial review with **zero architectural blockers**. The task is to correct the identified implementation/integrity defects while preserving all closed contracts.

The adversarial review found:

```text
H-1 HIGH  framework InClosure is not transitive
H-2 HIGH  durable artifacts lack complete pre-mint validation

M-1 MEDIUM DerivationStep.ConditionRefs duplicates not rejected structurally
M-2 MEDIUM cloneExpr shallow-copies BigInt.Mag
M-3 MEDIUM gr.CheckClosure lacks transitive cycle detection

H-3 HIGH  equivariance theorem conclusion is disconnected from original
            LHS0/RHS0 statement
```

The first five are localized fixes. H-3 requires semantic discipline and may expose a protocol gap.

The review evidence for H-1/H-2 is in the adversarial report. adv_review adv_review

The H-3 issue is evidenced by the actual equivariance implementation: `LHS0`/`RHS0` are constructed, but the persisted derivation/theorem ultimately uses `Step2` and `Step5`. fixture fixture

## 0. First read the operating protocol

Before changing code, read:

```text
phys-system/AGENTS.md
```

Treat it as the collective AI-agent operating protocol for `phys-artifact`, `phys-math`, `phys`, and `phys-lib`.

In particular, follow its requirements concerning:

- semantic claim preservation
- semantic liveness
- formal representation versus interpretation
- durable-artifact validation
- independent evidence
- adversarial falsification
- framework boundaries
- frozen legacy preservation

Do not modify the legacy `phys/AGENTS.md`.

---

# 1. Non-negotiable architectural constraints

Preserve all of the following:

```text
phys-artifact
    ↓
phys-math
    ↓
phys-lib/core
    ↓
phys-lib/<framework>
```

with no reverse dependencies and no framework-to-framework Go imports.

Do not:

- add new PhysMath AST nodes
- add new durable PhysMath families
- add `Equivariance` as a special PhysMath ontology
- add `Group` or `GroupAction`
- add synthetic `phys-gr:func/sin` / `phys-gr:func/cos` identities
- introduce a generic `PhysicalContract`
- add `phys-qm`, `phys-bohm`, `phys-cst`, or `phys-cdt` production packages
- modify `phys/`
- modify `phys-gr/`
- import `phys-gr`
- add `replace` directives to satisfy standalone module resolution
- broaden Phase-1 GR computational scope
- weaken existing validation merely to make tests pass

Do not create another planning document.

---

# 2. H-1 + M-3 — unify framework closure traversal

### Problem

`gr.FrameworkArtifact.InClosure` currently checks only:

- self namespace
- direct base frameworks

and does not traverse base-of-base closure. `gr.CheckClosure` likewise does not detect transitive cycles.

The review identified this as H-1 and M-3. adv_review adv_review

### Required behavior

Framework closure must support:

```text
F
├── base A
│   └── base B
│       └── base C
```

and correctly admit artifacts owned by:

```text
F
A
B
C
```

through the **transitive BaseFrameworkRefs closure**.

It must also reject cycles such as:

```text
F → A → B → F
```

and related cycles.

### Implementation requirements

Implement a deterministic traversal that:

- resolves framework refs from the supplied artifact/family store/map
- walks recursively/transitively
- tracks visited framework refs or equivalent identity
- detects cycles
- rejects dangling framework refs
- rejects refs to non-`framework/1`
- rejects malformed namespace ownership
- preserves current self-reference rejection

Avoid duplicating subtly different traversal algorithms in `InClosure` and `CheckClosure`.

A shared internal helper is acceptable **only if its ownership remains coherent with `phys-lib/gr`** and it does not create a forbidden cross-framework dependency.

Do not promote this into `phys-lib/core` merely because it is reusable in two functions.

### Required tests

Add adversarial tests for:

```text
depth 0 / self
depth 1 / direct base
depth 2 / base-of-base
depth 3 / deeper closure
dangling base
wrong family
self-cycle
two-node cycle
three-node transitive cycle
```

Also verify that a valid two-level framework chain permits `InClosure` to recognize the deepest base namespace.

Run equivalent conformance coverage against `fixture2` where appropriate.

---

# 3. H-2 — enforce durable pre-mint validation

### Problem

Several durable PhysMath artifacts do not have a complete `Validate()` path, including:

```text
Theorem
Derivation
Equation
MathCondition
FunctionApplication
```

and related durable paths can `Store.Put` encoded bytes without validating the semantic payload first. The review found that the equivariance fixture can mint `Theorem`, `Derivation`, and `Equation` this way. adv_review

### Required invariant

The following must become true:

```text
construct
    ↓
Validate
    ↓
Encode
    ↓
Store.Put
```

Invalid durable artifacts must **not enter the content-addressed store**.

Do not solve this by making `Store.Put` silently infer arbitrary semantics from raw bytes unless the existing architecture explicitly supports family-dispatched validation there.

Prefer the existing repository pattern for durable-family validation.

### Add/complete validators

Implement `Validate()` or the existing equivalent contract for all currently durable types lacking complete validation:

```text
Theorem
Derivation
Equation
MathCondition
FunctionApplication
```

Audit `OperatorApplication` and `Transformation` for partial validation and complete them where their existing family contract requires it.

Do not invent validation rules that aren't supported by PM-REP or existing family semantics.

### Critical theorem checks

A `Theorem` must reject invalid conclusion families, e.g.:

```text
ConclusionRef → expression/1
```

when its schema requires the theorem conclusion to be the appropriate relation family.

Similarly check:

- required refs present
- correct family
- correct namespace
- structural uniqueness
- valid referenced artifacts
- schema correctness
- no malformed/dangling references

### Store/minting audit

Search all production code and fixtures for:

```text
store.Put(...)
```

and classify every durable-artifact mint path.

Every durable family must pass through validation before minting.

Do not leave fixture code as an exception merely because it is test code.

---

# 4. M-1 — duplicate `ConditionRefs`

`Derivation.ValidateStructure` already checks duplicate `PremiseRefs` and `ResultRefs`, but does not structurally reject duplicate `DerivationStep.ConditionRefs`. The encoder rejects them later through SET encoding, creating inconsistent validation layers. adv_review

### Required fix

Make structural validation reject:

```text
ConditionRefs = [A, A]
```

before encoding.

Use the same duplicate semantics already used for SET-valued refs.

Do not change the SET/SEQUENCE semantics themselves.

Add:

```text
valid unique conditions → PASS
duplicate condition → FAIL
reordered conditions according to SET semantics → behavior unchanged
```

---

# 5. M-2 — deep-copy `BigInt.Mag`

The review found that `cloneExpr` shallow-copies `BigInt.Mag`, so cloned expressions can share mutable backing slices. adv_review

### Required fix

Make the clone structurally independent.

For every cloned BigInt magnitude:

```go
cp.Numerator.Mag = append([]byte(nil), e.Numerator.Mag...)
cp.Denominator.Mag = append([]byte(nil), e.Denominator.Mag...)
```

or an equivalent correct deep-copy mechanism.

Do not alter canonical encoding or identity semantics.

### Required test

Construct an expression containing mutable magnitude data, clone it, mutate the clone's magnitude through a review/test path, and prove the original remains unchanged.

Also verify existing golden bytes remain identical.

---

# 6. H-3 — equivariance theorem semantic correctness

This is the most important remediation.

## The intended claim

The fixture is supposed to represent:

```text
f(rho_X(g,x)) = rho_Y(g,f(x))
```

with:

```text
f(v)       = 2*v
rho_X(g,v) = g*v
rho_Y(g,v) = g*v
```

Those original sides are explicitly constructed in `fixture.go` as `LHS0` and `RHS0`. fixture

## Current defect

The six-step derivation instead computes two branches whose final results are:

```text
Step2 = 2*g*x
Step5 = 2*g*x
```

and the theorem relation is constructed from those final bytes. fixture fixture

`LHS0` and `RHS0` are therefore effectively dead relative to the persisted theorem conclusion.

The protocol test checks staged linkage and final equality, but does not establish that the theorem conclusion is semantically the original `LHS0 = RHS0` proposition. derivation_test

## Do NOT make this incorrect "fix"

Do **not** simply change:

```text
Relation(Step2, Step5)
```

to:

```text
Relation(LHS0, RHS0)
```

and declare success.

That would preserve the original statement but may break the existing theorem-conclusion/replay contract, because the derivation currently produces normalized final results rather than directly outputting `LHS0` and `RHS0`.

Likewise, do not weaken `CheckTheoremConclusion`, remove replay checks, or redefine theorem equality merely to make the fixture pass.

## Required investigation

First determine whether PM-REP, plan9.3, or an existing protocol mechanism already provides a way to express:

```text
original proposition
    ↓
derivation of its left side
    ↓
derivation of its right side
    ↓
normal-form/result correspondence
    ↓
theorem conclusion
```

using existing durable structures.

The desired semantic invariant is:

```text
LHS0 = f(rho_X(g,x))
RHS0 = rho_Y(g,f(x))

      │                 │
      │ independently   │
      │ derived         │
      ▼                 ▼

    Step2             Step5

      └─────── same canonical bytes ───────┘
```

The persisted theorem must preserve the fact that **Step2 and Step5 are the results of deriving the two sides of the original proposition**, not merely two arbitrary expressions that happen to be equal.

### Mandatory semantic-liveness test

Add a test that would fail if:

```text
LHS0
RHS0
```

were altered, disconnected, or replaced while the independently computed `Step2`/`Step5` remained unchanged.

This test is required specifically to prevent recurrence of the defect.

### Critical stop condition

If the current durable/protocol model has **no existing way** to encode and verify that correspondence, do not invent:

- a new family
- a new AST node
- a new rule
- a new theorem schema
- an ad-hoc field
- a hidden side channel
- a new protocol schema

Instead stop **only the H-3 implementation path** and report:

```text
PROTOCOL GAP / SEMANTIC GAP

Claim:
    f(rho_X(g,x)) = rho_Y(g,f(x))

Current representation:
    ...

Existing mechanisms considered:
    ...

Why they are insufficient:
    ...

Smallest candidate resolutions:
    A. ...
    B. ...
    C. ...

Recommended resolution:
    ...
```

Then ask the architect for the decision.

You may continue implementing and testing H-1/H-2/M-1/M-2/M-3 while H-3 is awaiting resolution.

---

# 7. Strengthen equivariance adversarial tests

The existing test suite already checks:

- six-step rule order
- staged LHS/RHS linkage
- replay
- deterministic refs
- foreign conclusion rejection. derivation_test derivation_test

Extend it to test the semantic claim itself.

Required mutation categories:

```text
mutate LHS0 only
mutate RHS0 only
disconnect LHS0 from staged LHS1
disconnect RHS0 from staged RHS1
replace original function declaration
replace original rho_X declaration
replace original rho_Y declaration
keep Step2/Step5 unchanged while corrupting source-side statement
```

A semantically invalid theorem must not remain verifiable merely because the final normal forms remain equal.

---

# 8. Independent evidence requirement

For any new semantic golden introduced by H-3, do not generate the expected value from the same production function under test.

The adversarial review already found this weakness in the `Pow(x,0)` condition test. adv_review

For each newly added semantic invariant, state whether the evidence is:

```text
SPEC-DERIVED
HAND-DERIVED
LEGACY-DERIVED
INDEPENDENTLY-COMPUTED
IMPLEMENTATION-DERIVED
```

An implementation-derived expected value cannot be the sole correctness witness.

---

# 9. Regression requirements

After remediation:

```text
go test ./...
go vet ./...
go list -deps ./...
go mod graph
```

and all existing Phase-1 acceptance tests must remain green.

Re-run:

- import scans
- namespace scans
- math-purity checks
- core-empty checks
- legacy-preservation checks
- deterministic golden checks
- fixture2 isolation
- repeated replay

Confirm specifically:

```text
phys-artifact: unchanged semantics
phys-math: unchanged canonical identities
phys-lib/gr: unchanged framework boundary
phys-lib/core: still empty
phys-gr: untouched
phys: untouched
```

Do not introduce a `go.mod replace` as part of this remediation; the current workspace-only development model remains authoritative.

---

# 10. Required implementation report

At completion, report:

```text
# Remediation Report

STATUS:
[COMPLETE / PARTIALLY COMPLETE — H-3 BLOCKED ON ARCHITECTURAL QUESTION]

H-1:
[fixed + evidence]

H-2:
[fixed + evidence]

M-1:
[fixed + evidence]

M-2:
[fixed + evidence]

M-3:
[fixed + evidence]

H-3:
[fixed / protocol gap identified]

## Semantic Invariants

- durable artifacts cannot enter Store without validation
- framework closure is transitive
- framework cycles are rejected
- ConditionRefs duplicates are rejected structurally
- cloned BigInts are structurally independent
- original theorem proposition remains semantically live
- final equality does not substitute for proposition preservation

## Tests

[exact commands]
[results]

## Files Changed

[list]

## Architectural Impact

[state explicitly whether any closed contract changed]

## Remaining Questions

[only genuine architectural/protocol questions]
```

Do **not** declare H-3 fixed merely because `Verify()` passes.

The completion criterion for H-3 is:

> **The executable fixture must establish the intended equivariance proposition, not merely establish equality between two independently computed final normal forms.**

---

### Final instruction

Implement H-1, H-2, M-1, M-2, and M-3 now.

Investigate H-3 against the existing formal/protocol contracts. **Do not invent a solution if the current schema cannot represent the required semantic correspondence.** If a genuine protocol gap exists, stop only H-3, provide the focused question and evidence, and continue all unaffected remediation work.

No plan revision. No broad redesign. No silent semantic invention.