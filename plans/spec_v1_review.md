I consolidated all four adversarial reviews against the Phase-1 `PM-REP-001 v1.0` specification and reconciled overlapping findings rather than simply adding their blocker counts together.

The reviews agree that the **strategic architecture is correct**: Phase-1/Phase-2 separation, paper-as-research-unit, ARI boundary, theory-neutral PhysMath, framework-specific `phys-*`, the “hundred physicists, hundred answers” principle, and the equivariance theorem as the reference slice all pass. spec_v1_deepseek spec_v1_z

The consolidated review identifies **15 distinct mechanical blockers after deduplication**, centered on:

- complete artifact-family registry
- exact `BigInt`, `ConstantID`, and enum encodings
- deterministic `ParameterID` scoping
- exact `BoundParameterRef` encoding
- durable `Relation` payload
- every `ArtifactRef` target family
- exact parameter bytes and semantics for every transformation rule
- explicit definition/function application
- deterministic substitution context
- framework-local RuleID namespace
- one normative equivariance artifact graph
- explicit `phys-*` family IDs and serialization
- minimum-assumption/specialization boundary

Those are the findings repeatedly substantiated by DeepSeek, Gemini, Qwen, and Z. spec_v1_deepseek spec_v1_gemini spec_v1_qwen spec_v1_z

I also explicitly **rejected several proposed fixes that would damage the Phase-1 boundary**, especially Qwen’s request to pull frozen-Phys `Kind/Dimension/AssumptionSet` integration back into this specification. That belongs to the deferred formalization/integration layer, not the representation core.

Most importantly, the central missing capability is now clear: the equivariance slice needs an explicit rule for applying a represented function definition. Z identifies this as the critical derivation gap, while Gemini independently reaches the same conclusion from the missing transformation semantics. spec_v1_z spec_v1_qwen

I captured the complete consolidated result, including the proposed normative resolutions, in:

:chatgpt-content-reference{index="8"}[**PM-REP-001 v1.0 — Consolidated Architectural Review**](sandbox:/mnt/data/PM-REP-001_v1.0_CONSOLIDATED_REVIEW.md)

The right next move is **one mechanical amendment to PM-REP-001, not another architecture redesign**. After those locks are inserted, the four reviewers should perform the final blocker-only pass, and then we move directly to the coding implementation plan.