### Adversarial Architectural Audit: `plan7.md`

`plan7.md` successfully establishes a clear governance roadmap and aligns core architectural boundaries, but as a technical artifact, it functions as an action plan for authoring `PHYSMATH_PHYS_EXTERNAL_ARCHITECTURE_v1.3.md` rather than a fully executable specification. If handed directly to a coding agent today, several fatal technical deadlocks, underspecified binary framing rules, unhandled AST edge cases, and state-machine ambiguities would force the agent to make arbitrary architectural decisions.

---

### 1. Binary Framing Paradox & Embedded `ArtifactRef` Hashing

* **The Architectural Deadlock:** Section 6 defines the top-level digest formula:

$$\text{SHA-256}(\text{Len32}(\text{Namespace}) \parallel \text{Namespace} \parallel \text{Len32}(\text{SchemaVersion}) \parallel \text{SchemaVersion} \parallel \text{Len32}(\text{ArtifactType}) \parallel \text{ArtifactType} \parallel \text{CanonicalPayload})$$



while Section 7 defines embedded `ArtifactRef` serialization as:

$$\text{EncodeArtifactRef} = \text{Len32}(\text{Namespace}) \parallel \text{Namespace} \parallel \text{Len32}(\text{SchemaVersion}) \parallel \text{SchemaVersion} \parallel \text{Len32}(\text{ArtifactType}) \parallel \text{ArtifactType} \parallel \text{Digest}_{32}$$



Section 8 specifies that JSON is **not** canonical, but fails to define the normative canonical binary serialization format for `CanonicalPayload` itself.


* **Coding Agent Failure Mode:**
1. **String Length Ambiguity:** `Len32` does not specify whether string length represents standard UTF-8 byte count (`len(s)`) or Unicode rune count (`len([]rune(s))`). Non-ASCII namespaces will produce divergent length headers across agents.


2. **CanonicalPayload Format Void:** When serializing a struct like `MathTrace` containing embedded `ArtifactRef` fields, one agent will serialize the payload using Canonical JSON (RFC 8785), another using CBOR (RFC 8949 deterministic), and a third using raw binary framing. All three will produce completely different top-level digest hashes.




* **Unambiguous Resolution Requirement:**
* Define `Len32(s)` strictly as the 32-bit Big-Endian unsigned integer equal to the raw byte length of string `s` encoded in UTF-8 (`len([]byte(s))`).


* Standardize a single normative canonical binary layout algorithm for struct payloads (e.g., Canonical JSON RFC 8785 where embedded `ArtifactRef` fields are rendered as fixed 4-byte-framed byte sequences or hex strings) before calculating `CanonicalPayload` bytes.





---

### 2. Parameter Identity Construction & Parameter Scope Void

* **The Architectural Deadlock:** Section 11 eliminates physical domain enums (`COORDINATE`, `TIME`, `RG_SCALE`) from `PhysMath` in favor of "parameter identity". Section 12 mandates that parameter identity is immutable under display label renames ($t \to \tau$).


* **Coding Agent Failure Mode:**
* `plan7.md` does not specify the concrete Go type or generation rule for a "parameter identity".


* If two independent systems create a `PhysMath` derivative expression $\partial / \partial t$, agent A will generate a random UUID v4 for parameter identity $t$, while agent B will generate a string slug `"t"`.
* When testing structural equality ($\text{Expr}_A == \text{Expr}_B$), the ASTs will fail to match because their parameter identities diverge, destroying deterministic equivalence checking across decoupled repositories.




* **Unambiguous Resolution Requirement:**
* Define parameter identity as a deterministic `[32]byte` hash derived from a canonical namespace and parameter symbol string:

$$\text{ParamID} = \text{SHA-256}(\text{Len32}(\text{Namespace}) \parallel \text{Namespace} \parallel \text{Len32}(\text{SymbolName}) \parallel \text{SymbolName})$$


* Require all local differential operators targeting the same logical parameter within a given namespace to resolve to identical deterministic parameter identity hashes.





---

### 3. Associative Operator Projection Edge Cases ($n=0$ and $n=1$)

* **The Architectural Deadlock:** Section 17 defines associative operator projection by ordering `Add`/`Mul` child nodes by canonical digest and constructing a left-associative binary tree:

$$\text{Project}(\text{Add}([a, b, c])) \longrightarrow \text{Phys.Add}(\text{Phys.Add}(a, b), c)$$


* **Coding Agent Failure Mode:**
* `plan7.md` omits handling for single-child (`Add([a])`) or empty (`Add([])`) n-ary nodes in `PhysMath`.


* When projecting `Add([a])`, agent A might collapse it directly to `a`, while agent B wraps it in `Phys.Add(a, Phys.Rational(0, 1))`.
* For `Add([])`, agent A might panic, agent B returns `Phys.Rational(0, 1)`, and agent C marks it `UNREPRESENTABLE`.


* **Unambiguous Resolution Requirement:**
* `PhysMath` canonicalization rules MUST specify:
1. $n=0$: Empty associative nodes (`Add([])`, `Mul([])`) are invalid in canonical `PhysMath` ASTs and must be simplified to their algebraic identity (`Rational(0,1)` for `Add`, `Rational(1,1)` for `Mul`) during AST canonicalization prior to projection.


2. $n=1$: Single-child associative nodes (`Add([a])`, `Mul([a])`) automatically collapse to child $a$ during canonicalization.


3. $n \ge 2$: Binary tree construction processes canonical children sequentially from index 0 to $n-1$ into a left-associative binary chain.







---

### 4. Transcendental and Dimensional Physical Constants in Pure Rational `PhysMath`

* **The Architectural Deadlock:** Section 28 strictly forbids floating-point AST values in `PhysMath` exact mathematics and requires `PhysMath.Rational` ($p/q$) or `Float64Bits` in numerical contracts.


* **Coding Agent Failure Mode:**
* Mathematical and physical expressions routinely involve irrational constants ($\pi$, $e$) or dimensional physical constants ($\hbar$, $c$, $G$).
* An agent attempting to represent $e^{i k x}$ or $\sqrt{2\pi}$ in `PhysMath` cannot use `PhysMath.Rational` without truncation.


* Without explicit guidance, an agent will either attempt to invent floating-point node extensions or hardcode fixed-precision rational approximations (e.g., $355/113$), destroying exact symbolic equivalence.




* **Unambiguous Resolution Requirement:**
* Transcendental constants and physical constants MUST be represented in `PhysMath` strictly as reserved symbolic nodes (`Symbol("PI")`, `Symbol("E")`, `Symbol("H_BAR")`).


* Symbolic constants remain un-evaluated exact symbols within `PhysMath`. Lowering to numeric execution (`Numerical`) resolves reserved symbolic constants to high-precision IEEE-754 binary representation via a deterministic lookup table owned by `phys-*`.





---

### 5. `ProposalState` Overload & Rejection Lineage Ambiguity

* **The Architectural Deadlock:** Section 22 defines `ProposalState` (`DRAFT`, `SUBMITTED`, `RESOLVED`) and `ProposalOutcome` (`PROJECTED_AS_HYPOTHESIS`, `PARTIALLY_PROJECTABLE`, `UNREPRESENTABLE`, `REJECTED`, `REQUEST_REVISION`). It states that on `REQUEST_REVISION`, the original proposal transitions to `RESOLVED` and the child revision sets `ParentProposalRef`.


* **Coding Agent Failure Mode:**
* Merging successful completion, outright rejection, and revision requests into a single `RESOLVED` state creates ambiguity in state-machine queries.
* An agent querying for finalized/accepted proposals will retrieve proposals that were actually rejected or revised, unless `ProposalOutcome` is stored and indexed as a primary query predicate alongside `ProposalState`.




* **Unambiguous Resolution Requirement:**
* Explicitly mandate that every persisted proposal record contains a compound state tuple:

$$(\text{State}: \text{ProposalState}, \text{Outcome}: \text{ProposalOutcome})$$


* Define legal terminal state transitions strictly:
* `SUBMITTED` $\to$ `(RESOLVED, PROJECTED_AS_HYPOTHESIS)`
* `SUBMITTED` $\to$ `(RESOLVED, REJECTED)`
* `SUBMITTED` $\to$ `(RESOLVED, REQUEST_REVISION)`


* State transitions to `RESOLVED` are final and immutable. Proposals in `(RESOLVED, REQUEST_REVISION)` reject any further submission or state modification.





---

### 6. `MathCondition` Deduplication Rules During Rewriting

* **The Architectural Deadlock:** Section 18 states that `Conditions []MathCondition` are accumulated during transformation and canonically sorted by condition digest bytes before result finalization.


* **Coding Agent Failure Mode:**
* Multi-step AST rewrites frequently generate identical conditions (e.g., $x \ne 0$ generated at step 1 and step 3).
* `plan7.md` does not specify whether sorting deduplicates the slice. Agent A will retain duplicate condition entries (`[x != 0, x != 0]`), whereas agent B will deduplicate them (`[x != 0]`).


* As a result, identical algebraic transformations will produce differing `TransformResult` canonical bytes and digest mismatches.




* **Unambiguous Resolution Requirement:**
* Condition slice finalization MUST execute a strict canonical sort-and-deduplicate pass:
1. Compute canonical digest for each `MathCondition` in `Conditions`.
2. Sort `Conditions` in ascending order using lexicographical byte comparison (`bytes.Compare`) of condition digests.


3. Remove adjacent duplicate elements having identical condition digests.







---

### 7. Go Module Import Topology and `ComputationalContract` Ownership

* **The Architectural Deadlock:** Section 26 assigns ownership of the `ComputationalContract` schema to `Numerical` because it represents the execution boundary, while `phys-*` produces valid instances.


* **Coding Agent Failure Mode:**
* If `phys-*` imports `Numerical` to instantiate `ComputationalContract`, and `Numerical` imports `phys-artifact` for provenance metadata, an agent building these repositories as separate Go modules must know the exact Go module paths and interfaces.
* If `Numerical` needs to inspect domain model metadata or framework references, a circular dependency risk arises between `phys-*` and `Numerical`.


* **Unambiguous Resolution Requirement:**
* Dependency hierarchy MUST follow a strict one-way DAG:

$$\text{phys-artifact} \longleftarrow \text{Numerical} \longleftarrow \text{phys-*} \longleftarrow \text{TOE-Agent}$$


* `Numerical` MUST NOT import `phys-*` or `PhysMath`. `Numerical` defines `ComputationalContract` using primitive Go types, raw binary/numeric slices, and generic `ArtifactRef` metadata structs imported from `phys-artifact`.





---

### Consolidated Blocker Matrix for `v1.3.md` Authoring

| Section | Issue / Deadlock Area | Agent Failure Mode | Required Resolution Rule in `v1.3.md` |
| --- | --- | --- | --- |
| **§6 & §7** | String Length & Canonical Payload Framing | Divergent length headers for non-ASCII strings; unpinned binary serializer for `CanonicalPayload`.

 | Define `Len32` as UTF-8 byte count (`len([]byte)`). Standardize exact binary/Canonical-JSON serializer for struct payloads.

 |
| **§11 & §12** | Parameter Identity Construction | Equivalent derivative operators generating distinct random IDs and failing AST equality.

 | Define parameter identity strictly as deterministic 32-byte hash over `(Namespace, SymbolName)`.

 |
| **§17** | Associative Operator Edge Cases | Panics or non-deterministic wrapping for single-child (`n=1`) or empty (`n=0`) nodes.

 | Mandate AST canonicalization: $n=0$ collapses to algebraic identity (`0` or `1`), $n=1$ collapses to child node.

 |
| **§28** | Irrationals & Physical Constants | Inability to represent $\pi, e, \hbar$ in exact rational AST without truncation.

 | Require exact constants to be represented as reserved `Symbol` nodes; lower to `Float64Bits` in `Numerical` via lookup table.

 |
| **§22** | Proposal State Machine Overload | Inability to distinguish accepted vs rejected/revised proposals in state queries.

 | Mandate compound state tuple `(ProposalState, ProposalOutcome)` for all persisted proposals.

 |
| **§18** | Condition Deduplication | Duplicate condition entries generating mismatched canonical `TransformResult` digests.

 | Require lexicographical sort AND adjacent deduplication on condition digests during finalization.

 |
| **§26** | Go Module Import Loops | Import cycle between `phys-*` and `Numerical`.

 | Enforce unidirectional import DAG: `phys-artifact` $\leftarrow$ `Numerical` $\leftarrow$ `phys-*`.

 |