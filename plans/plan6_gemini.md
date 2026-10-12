### Critical Architectural Blockers & Contract Ambiguities

`plan6.md` establishes a sound architectural direction, but several operational edge cases remain ambiguous. Without explicit closure, a coding agent implementing these rules would still be forced to make arbitrary architectural assumptions that break cross-repository deterministic execution, serialization compatibility, and state machines.

---

### 1. Serialization Boundary for Embedded `ArtifactRef`s in Canonical Payloads

* **The Architectural Deadlock:** Section 3 defines the binary framing format for top-level `ArtifactRef` digest computation. Section 15 (`PhysicalContract`) and Section 21 (`MathTrace`) define structures containing fields like `MathRefs []ArtifactRef` or `Input []ArtifactRef`. `plan6.md` does not specify how an embedded `ArtifactRef` is formatted when calculating the canonical payload bytes of a parent artifact.


* **Coding Agent Failure:** One agent serializing a `MathTrace` will format embedded `ArtifactRef` items as JSON objects, another as raw 32-byte hash digests, and a third as 4-byte-framed binary sequences. This will produce divergent canonical payload bytes and hash mismatch across repositories.


* **Unambiguous Decision:**
1. When an `ArtifactRef` appears as a field inside a parent canonical payload, it MUST be serialized using its exact 4-byte Big-Endian binary framing format:

$$\text{Enc}(\text{ArtifactRef}) = \text{Len32}(\text{Namespace}) \parallel \text{Namespace} \parallel \text{Version32}(\text{SchemaVersion}) \parallel \text{Len32}(\text{ArtifactType}) \parallel \text{ArtifactType} \parallel \text{Digest}_{32}$$


2. Embedded `ArtifactRef` fields are NEVER serialized as JSON text, hex strings, or un-framed 32-byte digests inside canonical payload calculations.





---

### 2. Floating-Point Canonicalization Paradox Across `phys-*` and Lowered Contracts

* **The Architectural Deadlock:** Section 4 strictly bans floating-point values in canonical payload representations (*"no floating-point values"*). However, `phys-*` physical parameters (e.g., metric coupling constants, momentum cutoffs $k$) and numerical execution plans inevitably contain real scalar values.


* **Coding Agent Failure:** An agent implementing `phys-*` domain contracts or lowered execution contracts will either violate Section 4 by using standard Go `float64` fields or get blocked trying to serialize physical real constants.


* **Unambiguous Decision:**
1. Pure mathematical expressions in `PhysMath` use `PhysMath.Rational` ($p/q$) exclusively; raw floating-point types remain strictly forbidden in `PhysMath` AST payloads.


2. If real numeric parameters must be represented in `phys-*` canonical payloads or lowered contract configurations, they MUST be encoded either as:
* An exact `PhysMath.Rational` struct (`Numerator int64`, `Denominator uint64`).
* An explicit 64-bit Big-Endian IEEE-754 uint64 integer representation (`math.Float64bits(val)` mapped to 8 Big-Endian bytes).


3. Locale-dependent text formatting, variable-length floating-point string representation, or native non-deterministic floating-point serializations are strictly prohibited in canonical payloads.





---

### 3. Non-Deterministic `MathCondition` Ordering in `TransformResult`

* **The Architectural Deadlock:** Section 11 specifies that AST transformations produce `TransformResult` containing `Conditions []MathCondition`. Section 11 requires conditions to be deterministically ordered, but does not specify the ordering key or algorithm.


* **Coding Agent Failure:** If AST rewriting accumulates conditions during tree traversal (e.g., left-to-right vs. right-to-left), two functionally identical simplification runs will produce identical condition sets in different slice orders ($[x \ne 0, y > 0]$ vs $[y > 0, x \ne 0]$). This leads to non-deterministic `TransformResult` payload digests.


* **Unambiguous Decision:**
1. AST transformation passes collect generated `MathCondition` records into a intermediate slice.
2. Before finalizing `TransformResult` or generating its canonical byte payload, the `Conditions []MathCondition` slice MUST be sorted in ascending order using the lexicographical byte comparison of each condition's canonical payload digest:

$$\text{bytes.Compare}(\text{Digest}(\text{Cond}_A), \text{Digest}(\text{Cond}_B))$$


3. Tree traversal sequence MUST NOT dictate final condition slice ordering.



---

### 4. N-Ary Operator Associativity Mismatch during `PhysMath` $\rightarrow$ `Phys` Projection

* **The Architectural Deadlock:** Section 7 defines the projection subset (`Add`, `Mul`, `Pow`, etc.). `PhysMath` uses n-ary child slices for associative operators (`Add([]Expr)`), whereas frozen `Phys` expressions utilize binary operator trees (`Add(Left, Right)`).


* **Coding Agent Failure:** When projecting `PhysMath.Add([a, b, c])` into frozen `Phys`, an agent will have to choose between left-associative `Add(Add(a, b), c)`, right-associative `Add(a, Add(b, c))`, or term sorting. Unaligned choices cause formalization verification failures against frozen `Phys`.


* **Unambiguous Decision:**
1. `PhysMath` canonicalization flattens all associative operators (`Add`, `Mul`) into n-ary child slices, sorted deterministically by child canonical payload digest.
2. When projecting n-ary `PhysMath` operators to frozen `Phys` binary nodes, the projection procedure MUST deterministically construct a **left-associative binary tree**:

$$\text{Project}(\text{Add}([c_0, c_1, c_2])) \longrightarrow \text{Phys.Add}(\text{Phys.Add}(c_0, c_1), c_2)$$





---

### 5. Parent Lineage and Terminal State Rules for `REQUEST_REVISION` Proposals

* **The Architectural Deadlock:** Section 18 states that `ProposalOutcome = REQUEST_REVISION` creates a new proposal revision rather than mutating an old proposal. However, Section 18 does not define the resulting `ProposalState` of the original proposal, nor how the new revision references its predecessor.


* **Coding Agent Failure:** The agent will leave original proposals hanging in `ProposalState = SUBMITTED`, or fail to record parent-child revision lineage in `FormalizationProposal` payloads.


* **Unambiguous Decision:**
1. When formalization evaluation returns `ProposalOutcome = REQUEST_REVISION`, the original proposal transitions to `ProposalState = RESOLVED` (a terminal state).
2. The schema for `FormalizationProposal` MUST include an optional field:

$$\text{ParentProposalRef} \quad *\text{ArtifactRef}$$


3. The revised proposal sets `ParentProposalRef` to the `ArtifactRef` of the resolved parent proposal. Re-submitting or mutating a proposal in `ProposalState = RESOLVED` is strictly prohibited.





---

### 6. Data-Flow Execution Contract for `ComputationalContract` in `Numerical`

* **The Architectural Deadlock:** Section 23 specifies the data flow:

$$\text{PhysMath} \longrightarrow \text{phys-*} \text{ domain lowering} \longrightarrow \text{ComputationalContract} \longrightarrow \text{Numerical}$$



Section 5 states that in-memory values use Go structs, while cross-process boundaries use `ArtifactRef`. `plan6.md` does not specify whether `ComputationalContract` passed to `Numerical` contains in-memory numeric slice data or serialized `ArtifactRef` pointers requiring external resolution.


* **Coding Agent Failure:** An agent implementing `Numerical` solvers will attempt to build external `ArtifactRef` data fetching clients or panic when encountering unresolved reference fields during array operations.


* **Unambiguous Decision:**
1. In-memory handoff of `ComputationalContract` from `phys-*` to `Numerical` within the same Go process MUST provide fully-populated, in-memory Go arrays and numeric execution plans.
2. `Numerical` DOES NOT perform `ArtifactRef` lookups or network resolution to execute numeric computation.


3. `ArtifactRef` fields on `ComputationalContract` exist purely as immutable metadata tags for post-execution provenance logging.





---

### 7. Symbol Invariance Law for `DifferentialDomainTag`

* **The Architectural Deadlock:** Section 12 introduces `DifferentialDomainTag` (`DIFF_COORDINATE`, `DIFF_TIME`, `DIFF_RG_SCALE`) on `PhysMath.Operator` nodes. It does not specify how canonicalizers and transformation engines treat domain tags during variable substitution or symbol renaming (e.g., renaming variable $t \to \tau$).


* **Coding Agent Failure:** An agent performing variable substitution might strip or reset `DifferentialDomainTag` to `DIFF_COORDINATE` when a target variable symbol changes, breaking domain separation downstream.


* **Unambiguous Decision:**
1. `DifferentialDomainTag` is an immutable intrinsic property of the differential operator node; it is invariant under symbol renaming and variable substitution.
2. Renaming target variable $t \to \tau$ updates the target variable identifier but leaves `DifferentialDomainTag = DIFF_TIME` untouched.
3. Canonical AST transformations forbid commuting operators with differing `DifferentialDomainTag` values unless an explicit non-commutative domain commutation rule applies.





---

### Consolidated Resolution Matrix for v1.3

| Issue ID | Concern Area | Deadlock / Failure Mode | Unambiguous Architectural Rule |
| --- | --- | --- | --- |
| **ADR-049** | Embedded `ArtifactRef` Framing | Mismatched hash output when nesting references.

 | Embedded references inside payloads MUST use the binary 4-byte Big-Endian framing standard.

 |
| **ADR-050** | Real Scalar Serialization | Banned floats blocking physical constants.

 | Real values in `phys-*` use `PhysMath.Rational` or double-precision IEEE-754 Big-Endian uint64 bits. |
| **ADR-051** | Condition Slice Ordering | Non-deterministic payload digests.

 | `Conditions []MathCondition` MUST be sorted by canonical digest byte order before digest calculation.

 |
| **ADR-052** | Operator Projection Associativity | Tree structure mismatch during projection.

 | Projections of n-ary `PhysMath` operators construct deterministic left-associative binary trees in `Phys`.

 |
| **ADR-053** | Proposal State Machine Terminality | Hanging proposals on revision request.

 | `REQUEST_REVISION` sets parent state to `RESOLVED`; child links via `ParentProposalRef`.

 |
| **ADR-054** | Lowered Handoff Execution Data | Solver attempting external reference resolution.

 | `ComputationalContract` passed to `Numerical` carries fully-populated in-memory data arrays.

 |
| **ADR-055** | Differential Domain Invariance | Domain tags lost during variable substitution.

 | `DifferentialDomainTag` is immutable under symbol renames and transformation rewrites.

 |