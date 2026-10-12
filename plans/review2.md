This second review is substantially better. It confirms that the remediation did **not** escape the Phase-1 architecture: no new families, AST nodes, rules, protocol schemas, framework coupling, physical ontology, or frozen-repo changes were introduced. It also confirms that `claim.go` is an in-memory witness rather than a new durable semantic layer. adv_review2

There are, however, **two things I would correct before calling the remediation fully closed**.

### 1. M-4 should be classified as deferred, not "still open"

This is the biggest process inconsistency.

Earlier we explicitly decided that adding:

```go
replace github.com/PithomLabs/phys-math => ../phys-math
```

would violate the closed D17 workspace/publication boundary. So the correct disposition is:

```text
M-4 = DEFERRED / OUTSIDE PHASE-1
```

not:

```text
M-4 = unresolved defect requiring attention before Phase 2
```

The review itself correctly observes that the current state is workspace-correct but fails standalone module resolution. adv_review2

I would amend the final report to say:

> M-4 was intentionally deferred under D17; standalone module publication/version resolution is outside the Phase-1 workspace-delivery contract.

Do **not** add `replace` merely to make the review green.

### 2. H-3 is much better, but there is one subtle question left

The review now establishes the semantic chain:

```text
LHS0 → innerRho → Step0 → LHS1 → Step1 → Step2 → conclusion.LHS
RHS0 → innerF   → Step3 → RHS1 → Step4 → Step5 → conclusion.RHS
```

and the mutations apparently fail when the source proposition is altered. adv_review2

That is a significant improvement and addresses the actual defect we found.

But notice something important: the source-side linkage is provided through **in-memory fixture state**:

```text
fx.LHS0
fx.RHS0
fx.InnerRho
fx.InnerFx
```

while the persisted `Theorem` still has a conclusion whose sides are `Step2` and `Step5`. The review says `VerifyClaim` is sufficient, but the final test we should perform is:

> **Can an independent verifier reconstruct the semantic correspondence using only persisted artifacts, without relying on the `Fixture` object's in-memory fields?**

That is the strongest possible test of whether H-3 is genuinely solved rather than merely made true inside this particular test harness.

If the answer is yes, excellent—the protocol has a durable traceable interpretation.

If the answer is no, then we have discovered a more subtle distinction:

```text
test harness proves semantic correspondence
        ≠
persisted protocol carries semantic correspondence
```

I would not call that an automatic blocker yet, because the existing protocol may intentionally define fixture-level verification as the semantic witness. But it is worth checking before we declare H-3 completely closed.

### 3. S-1 is not really "scope creep"

The duplicated transitive traversal in `fixture2` is a maintainability observation, not architectural scope creep. It is isolated under `internal/conformance`, and the review confirms it does not pollute production architecture. adv_review2

I would relabel:

```text
S-1 → OBSERVATION / TEST-HARNESS DUPLICATION
```

rather than scope creep.

Do **not** extract it into `core` simply to eliminate duplication. That would be a much worse architectural decision.

### 4. The exported test API should be reduced if convenient

The review identifies:

```text
VerifyClaim(*Fixture)
InnerRho
InnerFx
```

as unnecessary public surface. adv_review2

This isn't a Phase-1 problem, but I agree it is worth cleaning up. A good final shape would keep these implementation/test witnesses private by moving the relevant protocol tests into the `equivariance` package rather than `equivariance_test`, provided doing so does not undermine the independence the test is supposed to demonstrate.

That should be treated as **test/API hygiene**, not architecture.

---

## My updated disposition

```text
Architecture:                 PASS
Scope containment:             PASS
H-1:                           FIXED
H-2:                           FIXED
M-1:                           FIXED
M-2:                           FIXED
M-3:                           FIXED
L-3/L-4:                       FIXED
H-3:                           PROVISIONALLY FIXED
M-4:                           DEFERRED under D17
S-1:                           TEST-HARNESS OBSERVATION
```

The review's strongest result is that it independently verified the important H-3 property that **`Step2 == Step5` alone no longer suffices**; source-side mutations now fail. adv_review2

So I would **not reopen the architecture**.

I would do one final targeted verification of H-3's portability—from persisted artifacts alone—and then close Phase 1. That test is more valuable than another broad code review.