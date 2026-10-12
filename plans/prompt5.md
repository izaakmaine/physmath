Yes. The next review should **not re-review correctness from scratch**. It should adversarially test whether the remediation **stayed within the closed Phase-1 scope**, especially because H-3 introduced `claim.go` / `claim_test.go` and the report says no closed contract changed.

The attached remediation report claims all findings are complete, H-3 required no protocol change, all tests pass, and no new families/nodes/rules, pipeline work, `replace` directives, or legacy modifications were introduced. It also explicitly claims the original theorem proposition is now semantically live end-to-end. AGENTS

Use this prompt:

---

# Scope Adversarial Review — Phase-1 Remediation

You are performing a **scope-containment and architectural-drift review** of the completed remediation to Phase-1 adversarial findings in:

```text
/home/chaschel/Documents/go/
├── phys-artifact/
├── phys-math/
├── phys-lib/
├── phys-system/
├── phys/       # FROZEN
└── phys-gr/    # HISTORICAL / FROZEN
```

The remediation report claims:

```text id="8z9r7q"
H-1 fixed
H-2 fixed
M-1 fixed
M-2 fixed
M-3 fixed
H-3 fixed
L-3/L-4 fixed

No protocol gap
No architectural blockers
No closed contract changed
No new families/nodes/rules
No Category globalization
No pipeline work
No replace directives
phys/ and phys-gr/ untouched
```

Do **not** accept these claims merely because tests pass.

Your purpose is narrower than a normal code review:

> **Determine whether the remediation fixed the reported defects while remaining strictly inside the already-closed Phase-1 architecture and scope.**

Do not redesign anything.

---

## 1. Authority hierarchy

Use:

```text
1. PM-REP-001 v1.2
2. plan9.3 closed decisions
3. phys-system/AGENTS.md
4. plan10 remediation plan
5. adversarial findings as defect reports
6. implementation claims/report
```

A remediation finding is **not authority to introduce new architecture**.

If remediation code conflicts with PM-REP or plan9.3, report it as scope/architecture drift.

---

# 2. First establish exactly what changed

Inspect the complete diff from the Phase-1 baseline.

Identify:

```text
new files
modified files
deleted files
new exported identifiers
new packages
new durable families
new AST nodes/tags
new rules
new schemas
new protocol fields
new dependencies
new framework relationships
```

Pay particular attention to:

```text
phys-lib/gr/framework.go
phys-lib/gr/*
phys-lib/gr/equivariance/*
phys-lib/internal/conformance/fixture2/*
phys-math/families.go
phys-math/store.go
phys-math/rules*
phys-math/validation*
phys-system/AGENTS.md
/home/chaschel/Documents/go/AGENTS.md
```

The remediation report specifically claims changes to:

```text
equivariance/{fixture,claim}.go
equivariance/{AGENTS.md, protocol/{derivation,claim}_test.go}
```

and several new validation/test files. Verify that those additions are genuinely test/verification mechanisms and did not introduce a new ontology.

---

# 3. Core scope invariant

The remediation must satisfy:

```text
FIX DEFECT
    +
PRESERVE CONTRACT
    +
PRESERVE SCOPE
```

A fix is a scope violation if it solves the finding by introducing:

- a new abstraction not already authorized
- a new durable family
- a new AST node
- a new transformation rule
- a new protocol schema
- a new cross-framework dependency
- a new physical ontology
- new framework semantics in PhysMath
- generalized infrastructure that belongs elsewhere
- Phase-2 functionality prematurely implemented

---

# 4. Attack H-1 / M-3 scope

The reported fix introduces unified transitive framework traversal.

Verify that it only implements the existing transitive closure requirement.

Check that it does **not** introduce:

- a generic framework inheritance ontology in `phys-math`
- a new durable closure family
- a new global framework registry
- a new `core` abstraction
- cross-framework executable dependencies
- specialized logic for future frameworks

Verify helper ownership remains `phys-lib/gr`.

The implementation report says the helper remains GR-owned and was not promoted to core. Test that claim.

### Adversarial question

Could this fix be removed and replaced with a more generic "framework system" abstraction without violating the Phase-1 specification?

If yes, flag unnecessary scope expansion.

If no, explain why the localized implementation is appropriate.

---

# 5. Attack H-2 scope

The remediation adds durable `Validate()` methods and gates minting:

```text
construct
→ Validate
→ Encode
→ Put
```

Verify that the validators enforce **existing family contracts only**.

Look specifically for validation rules that were invented during remediation.

Do not allow:

```text
new theorem semantics
new ontology
new physical semantics
new cross-family meaning
new proof obligations
```

merely because they seem useful.

Check every new validator:

```text
What existing PM-REP requirement authorizes this check?
```

If no answer exists, flag it.

Also verify that validation was not incorrectly centralized into a new generic mechanism that changes ownership.

---

# 6. Attack M-1 / M-2 scope

For:

```text
duplicate ConditionRefs
BigInt.Mag deep-copy
```

verify that fixes are strictly mechanical.

They must not change:

- canonical identity
- canonical bytes
- family semantics
- rule semantics
- API ownership

Re-run golden vectors and compare with the Phase-1 baseline.

A correct hardening fix should not silently create identity drift.

---

# 7. Attack L-3 / L-4 scope

Verify:

### L-3

Unused-import cleanup has no semantic effect.

### L-4

The hardcoded `NEQ(x,0)` bytes are genuinely:

```text
SPEC-DERIVED
```

rather than secretly generated at test runtime.

Check that the production encoder is no longer the source of truth for the expected value.

The test should fail if the canonical encoding changes.

This is test-independence hardening, not a new semantic rule.

---

# 8. H-3 is the primary scope attack

This is the most important part of the review.

The implementation claims H-3 was resolved **without a new protocol mechanism**, using existing staged-linkage and verifier-trace mechanisms.

Verify that claim independently.

The intended proposition is:

\[
f(\rho_X(g,x))=\rho_Y(g,f(x)).
\]

The original fixture contains:

```text
LHS0 = f(rho_X(g,x))
RHS0 = rho_Y(g,f(x))
```

and the remediation claims these are now semantically live end-to-end.

Do not merely verify that:

```text
Step2 == Step5
```

continues to hold.

Instead establish the complete chain:

```text
LHS0
  ↓
declared left derivation/result
  ↓
conclusion.LHS

RHS0
  ↓
declared right derivation/result
  ↓
conclusion.RHS
```

---

# 9. H-3 "no invention" attack

Inspect the newly added:

```text
equivariance/claim.go
equivariance/protocol/claim_test.go
```

very carefully.

Determine whether they introduce any implicit new protocol concept under a different name.

Search for things resembling:

```text
Claim
Proposition
Statement
ProofObligation
SideWitness
Correspondence
NormalizationWitness
TraceBinding
SourceExpression
```

and determine whether any such object is:

- durable
- serialized
- a new family
- an implicit protocol schema
- a new theorem ontology

The remediation is only within scope if these are **in-memory verification structures or tests using existing durable structures**, unless the existing specification already authorizes otherwise.

If `claim.go` creates a new persistent semantic layer, flag it.

---

# 10. H-3 falsification

Perform these exact attacks:

### A. Mutate LHS0

Change it to a different valid expression.

Keep:

```text
Step2
Step5
```

unchanged.

Verification must fail.

### B. Mutate RHS0

Same requirement.

### C. Disconnect LHS0

Make staged `LHS1` no longer correspond to the declared original left side.

Verification must fail.

### D. Disconnect RHS0

Same.

### E. Replace declaration

Replace:

```text
f
rho_X
rho_Y
```

with different valid Function artifacts while keeping final normal forms fixed.

Verification must fail where that declaration is part of the claimed proposition.

### F. Identity-preserving-looking mutation

Change a display name / ParameterID / declaration artifact in a way that appears superficially equivalent.

Verify that the correct identity rules apply.

The key objective is:

> **A theorem cannot remain valid merely because its independently computed final normal forms still happen to be equal.**

---

# 11. Attack semantic laundering

Determine whether the remediation accidentally turns:

```text
manifest metadata
```

into:

```text
formal theorem input
```

or turns test-only metadata into formal proof state.

The manifest remains metadata.

No prose field should become a mathematical premise.

The protocol fixture must still derive formal content from actual artifact references.

---

# 12. Attack `phys-math` contamination

Search all remediation changes for GR terminology or framework-specific semantics.

Verify no new exported PhysMath API now mentions:

```text
GR
GeneralRelativity
equivariance
rho_X
rho_Y
metric
curvature
coordinate
framework
physical
interpretation
```

unless explicitly permitted as non-semantic test identifiers/comments.

Particularly verify that H-3 did **not** cause an `Equivariance` abstraction to leak into `phys-math`.

The fixture may be called `equivariance`; the mathematical core must remain generic.

---

# 13. Attack core contamination

The remediation report claims:

```text
core still empty
```

Verify:

- zero exported domain symbols
- no closure helper
- no generic theorem-claim abstraction
- no validation registry
- no framework registry
- no "temporary" shared abstraction

If any remediation finding caused new code in `phys-lib/core`, flag it.

---

# 14. Attack Phase-1 scope creep

Specifically search for accidental implementation of deferred work:

```text
Schwarzschild
Christoffel pipeline
Ricci pipeline
Einstein/vacuum pipeline
weak-field pipeline
SessionCorrespondence
numerical execution
sparse tensors
future QM/Bohm/CST/CDT packages
two-implementation equivalence
publication/version machinery
frozen-phys integration
```

Any new implementation beyond the closed scope must be reported even if useful.

Distinguish:

```text
test fixture/reference vocabulary
```

from:

```text
actual implementation of deferred capability
```

Do not flag mere mentions unless executable capability was introduced.

---

# 15. Attack dependency drift

Verify:

```text
phys-artifact
    ↓
phys-math
    ↓
phys-lib
```

remains intact.

Specifically verify the remediation did not introduce:

- `phys-gr` imports
- frozen `phys` imports
- framework-to-framework imports
- new `replace` directives
- indirect dependency loopholes
- test-only dependency violations

Also verify that `phys-system` is documentation-only and is not imported by production code.

---

# 16. Attack API expansion

List every newly exported identifier added by remediation.

For each one ask:

```text
Was this already required by the closed architecture?
Is it required solely to correct the finding?
Could it have been private?
Does it create a new extension point?
Does it create a new ontology?
```

Prefer private/in-package mechanisms where the specification does not require a public API.

Flag unnecessary public surface as scope risk.

---

# 17. Attack canonical identity drift

Recompute the Phase-1 goldens after remediation.

Expected:

```text
Existing canonical bytes remain unchanged
```

except where the remediation explicitly targets a bug in an invalid/unpinned artifact.

In particular verify:

- ParameterIDs unchanged
- Function refs unchanged
- relation encoding unchanged
- theorem encoding unchanged
- derivation encoding unchanged
- equivariance 22-golden set unchanged unless a previously incorrect golden is intentionally corrected

The remediation report claims all 22 equivariance goldens remain unchanged. Verify independently.

---

# 18. Attack test independence

The review previously found the L-4 circularity in `Pow(x,0)`.

Revisit the entire remediation test suite and ask:

> Did the remediation introduce any new test whose expected result is generated by the same production implementation it is supposed to verify?

Search for patterns such as:

```text
expected := productionEncoder(...)
want := productionCanonicalizer(...)
golden := Build(...)
```

where that production result becomes the correctness oracle.

Distinguish legitimate structural use of production constructors from circular correctness assertions.

---

# 19. Attack "same code, stronger test" illusions

For every remediation test, determine whether it can actually fail against the pre-fix implementation.

For each new test:

```text
Would this test have failed before the fix?
```

If no, classify it as weak/non-diagnostic.

For H-3 this is particularly important.

The new semantic-liveness test must demonstrably fail against the old implementation.

---

# 20. Documentation/protocol boundary

Verify:

```text
phys-system/AGENTS.md
```

is documentation-only.

Verify the new root:

```text
/home/chaschel/Documents/go/AGENTS.md
```

is only a pointer and does not become an executable dependency or semantic authority encoded inside the software.

Do not modify the frozen:

```text
phys/AGENTS.md
```

---

# 21. Legacy preservation

Independently verify:

```text
phys/
phys-gr/
```

remain unchanged.

The remediation report claims their hashes remain:

```text
phys-gr: acf0259f…
phys:    9885a03f…
```

Verify against the established Phase-0 oracles.

Removing `phys-gr` from `go.work` remains permitted; modifying/deleting its contents is not.

---

# 22. Final scope classification

Classify every remediation change as one of:

```text
REQUIRED FIX
TEST HARDENING
DOCUMENTATION
UNNECESSARY BUT HARMLESS
SCOPE CREEP
ARCHITECTURAL VIOLATION
```

Do not use severity merely because something is imperfect.

A scope review is answering:

> Did we fix only what needed fixing?

---

# 23. Required final report

Return:

```text
# Scope Adversarial Review

VERDICT:
[PASS / PASS WITH SCOPE FINDINGS / BLOCKED]

REQUIRED FIXES PROPERLY SCOPED:
[number]

TEST HARDENING:
[number]

SCOPE CREEP:
[number]

ARCHITECTURAL VIOLATIONS:
[number]

## 1. Executive Assessment

[brief conclusion]

## 2. Scope Findings

### [ID] [CLASS] — [title]

Location:
...

Original requirement:
...

Observed change:
...

Why this is / is not within Phase-1 scope:
...

Evidence:
...

Recommendation:
...

## 3. H-3 Semantic Verification

Original proposition:
...

LHS0 trace:
...

RHS0 trace:
...

Conclusion binding:
...

Mutation evidence:
...

Does Step2 = Step5 still substitute for proposition preservation?
[YES/NO]

Does the implementation introduce a new protocol ontology?
[YES/NO]

## 4. Scope Invariants

- no new PhysMath families
- no new AST nodes
- no new rules
- no physical leakage
- core remains empty
- no new framework coupling
- no frozen-repo modification
- no Phase-2 functionality
- canonical identities preserved
- protocol schema not expanded without authorization
- documentation remains documentation-only

[VERIFIED / FAILED for each]

## 5. Newly Added Public API

[list and rationale]

## 6. Deferred Work Accidentally Implemented

[list or NONE]

## 7. Final Recommendation
```

---

## Critical review rule

**Do not recommend redesign merely because a more general abstraction would be elegant.**

The goal is not to make the remediation "better."

The goal is:

> **Fix the defects, preserve the closed architecture, preserve Phase-1 scope, and make the new semantic invariants enforceable.**

For H-3, pay special attention to whether the new `claim.go` mechanism is genuinely an **in-memory verification witness over existing structures** or whether it has quietly become a new semantic/protocol layer. That is the most likely place for scope creep in this remediation.

Also test the strongest adversarial question:

> **Could the implementation have made H-3 pass by proving a stronger-looking but actually different statement than the original equivariance proposition?**

If yes, report it even if every existing test passes.