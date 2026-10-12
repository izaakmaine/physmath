I would use `plan10` as the remediation execution plan. It is appropriately corrective, bounded, and incorporates the adversarial findings without reopening the architecture. It explicitly makes `phys-system/AGENTS.md` the operating protocol, adds the root pointer, separates the mechanical fixes from the semantic H-3 investigation, and preserves the frozen repositories. plan10

There is, however, **one refinement I strongly recommend before handing it to the coding agent**.

## H-3 needs one more explicit invariant

The current wording correctly says the intended claim is:

```text
f(rho_X(g,x)) = rho_Y(g,f(x))
```

and that `LHS0/RHS0` are currently disconnected from the persisted `Step2 = Step5` theorem. plan10

But add this explicit invariant:

```text
H-3 semantic invariant:

The persisted theorem must have a traceable correspondence from its
conclusion sides to the two sides of the claimed proposition.

It is insufficient that:
    normalize(LHS0) == normalize(RHS0)

The verifier must establish:
    LHS0  → declared left derivation/result  → conclusion.LHS
    RHS0  → declared right derivation/result → conclusion.RHS

No theorem-side artifact may be semantically dead.
```

That makes the target unambiguous without prescribing *how* the existing protocol must encode it.

I would also add one very specific adversarial test:

```text
Construct a mutant in which LHS0 and/or RHS0 is replaced by a
different valid expression while Step2 and Step5 remain unchanged.

The verification MUST fail.

If the current protocol has no mechanism by which this can fail,
report PROTOCOL GAP / SEMANTIC GAP rather than inventing one.
```

That is the test that directly catches the defect we found.

## Everything else in plan10 looks right

The closure remediation is correctly framed as a **single transitive traversal concern**, including depth-2+ and cycle cases. plan10

The durable-minting invariant is also correctly strengthened to:

```text
construct → Validate → Encode → Put
```

including fixture code, rather than treating tests as an exemption. plan10

The `ConditionRefs` and `BigInt.Mag` fixes are appropriately localized. plan10

And I agree with the treatment of L-3/L-4 as hardening rather than architecture: L-4 specifically strengthens test independence without changing semantics. plan10

The final regression requirements also preserve the correct boundary: no `replace`, legacy hashes rechecked, and H-3 cannot be declared fixed merely because normal-form equality still passes. plan10

### One governance improvement

At the top, I would change:

```text
Authority: PM-REP-001 v1.2, plan9.3, phys-system/AGENTS.md,
adversarial findings...
```

to explicitly state:

```text
Authority hierarchy:
1. PM-REP-001 v1.2
2. plan9.3 closed decisions
3. phys-system/AGENTS.md for agent operating discipline
4. plan10 remediation instructions
5. adversarial findings as defect reports
```

That prevents a future implementation agent from treating an adversarial recommendation as higher authority than the representation specification.

Other than those small clarifications, **I would send plan10 to the coding agent now**. The plan has reached the right level: fixed remediation for H-1/H-2/M-1/M-2/M-3/L-3/L-4, and a deliberately constrained stop-and-ask path for H-3.