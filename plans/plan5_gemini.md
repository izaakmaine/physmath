### 1. Ownership Vacuum for Ecosystem `ArtifactRef` & Canonical Digest Engine

* **The Architectural Deadlock:** Section 34 defers the `phys-interop` package for MVP, while Section 3 enforces strict isolation ($\text{PhysMath} \bot \text{Phys}$). However, Section 5 mandates an ecosystem-wide `ArtifactRef` scheme and canonical SHA-256 digest calculation across all projects. If `phys-interop` is omitted and `PhysMath` cannot import `Phys` (or vice versa), there is no specified package location for the Go struct definition of `ArtifactRef` and its canonical byte encoder.
* **Coding Agent Failure:** Agents working in `[github.com/PithomLabs/physmath](https://github.com/PithomLabs/physmath)` and `[github.com/PithomLabs/phys](https://github.com/PithomLabs/phys)` will independently invent duplicate `ArtifactRef` Go types and serialization codecs. This creates code duplication, divergent canonical byte output, and digest mismatch across repository boundaries.


* **Unambiguous Decision:**
1. Define a lightweight, zero-dependency baseline type repository/package: `[github.com/PithomLabs/phys-core/artifact](https://github.com/PithomLabs/phys-core/artifact)` (or `phys/core/artifact`).
2. This package contains **only** the `ArtifactRef` struct, canonical byte interface definitions, and standard errors. It contains zero domain physics, zero symbolic math, and zero trust logic.
3. `PhysMath`, `Phys`, `phys-*`, and `TOE-Agent` all import `[github.com/PithomLabs/phys-core/artifact](https://github.com/PithomLabs/phys-core/artifact)` as their single source of identity truth, keeping $\text{PhysMath} \bot \text{Phys}$ intact at the domain logic level.





---

### 2. Paradox Between Extension Protocol, Static Typing, and Zero Trait Runtime

* **The Architectural Deadlock:** Section 9 allows `phys-*` packages to introduce mathematical extension sorts. Section 3 prohibits `PhysMath` from importing `phys-*`. Section 1.2 explicitly forbids dynamic/unrestricted algebraic trait runtimes in `PhysMath`. If `PhysMath` cannot import `phys-*` at compile time and cannot use dynamic runtime trait reflection/lookup tables, `PhysMath`'s AST transformers, evaluation engines, and canonicalizers cannot process or traverse AST nodes containing external extension sorts.


* **Coding Agent Failure:** An agent attempting to implement `PhysMath` AST traversal for external extension sorts will either create illegal static imports to `phys-*` packages or build a dynamic reflection/trait registry that violates Section 1.2.


* **Unambiguous Decision:**
1. `PhysMath`'s mathematical AST sort set is **strictly closed** at compile-time within `PhysMath` itself (Section 8).


2. External `phys-*` packages **do not** inject new AST node sorts into `PhysMath`. Instead, `phys-*` packages wrap core `PhysMath` ASTs inside domain-specific structures (`PhysicalObject`, `PhysicalContract`).


3. If a genuinely new mathematical sort is required (e.g., simplicial complexes or differential forms), it must be implemented directly inside `PhysMath` via a versioned schema update to `PhysMath`, maintaining static Go typing and deterministic AST traversal.





---

### 3. Lowering Boundary & Dependency Deficit for the Numerical Layer

* **The Architectural Deadlock:** Section 0 defines dependency rules as `Numerical -> Phys + selected phys-*`, omitting `PhysMath`. However, Section 39 specifies that `ComputationalContract` contains mathematical equations and input references (`mathematical_input_refs`). If `Numerical` does not import `PhysMath`, it cannot parse or execute the symbolic AST equations stored in `PhysMath` format. Furthermore, Section 39 leaves unassigned whether `phys-*` or `Numerical` performs symbolic-to-numeric lowering.


* **Coding Agent Failure:** Agents implementing `Numerical` solvers will fail to compile because AST type definitions are missing, or agents will inappropriately attempt to build symbolic AST parsers inside `Numerical`.
* **Unambiguous Decision:**
1. `phys-*` packages own the symbolic-to-numeric lowering step.
2. Prior to handing off a `ComputationalContract` to the Numerical Layer, `phys-*` lowers pure `PhysMath` ASTs into flattened numeric representations (e.g., serialized array operations, matrix topologies, or lowered bytecodes).
3. `Numerical` consumes lowered numerical arrays and execution plans; it does not read or parse symbolic `PhysMath` ASTs directly.



---

### 4. Framing Protocol Ambiguity in Canonical SHA-256 Digest Concatenation

* **The Architectural Deadlock:** Section 5 defines digest calculation as:

$$\text{Digest} = \text{SHA-256}(\text{Namespace} \parallel \text{canonical SchemaVersion} \parallel \text{canonical ArtifactType} \parallel \text{canonical Payload})$$



without specifying header framing or byte delimiters. Raw string concatenation (e.g., `NS + Schema + Type + Payload`) creates digest collisions (e.g., $\text{NS}=\text{"a"}, \text{Type}=\text{"bc"}$ yields the same bytes as $\text{NS}=\text{"ab"}, \text{Type}=\text{"c"}$).


* **Coding Agent Failure:** Agents in different repositories will concatenate strings using different formats (raw string vs. null-byte delimited vs. JSON wrapping), resulting in incompatible digest outputs for identical artifacts.


* **Unambiguous Decision:** Digest calculation MUST execute binary Big-Endian length-prefixed framing over canonical UTF-8 bytes:

$$\text{Header} = \text{Len32}(\text{Namespace}) \parallel \text{Namespace} \parallel \text{Len32}(\text{SchemaVersion}) \parallel \text{SchemaVersion} \parallel \text{Len32}(\text{ArtifactType}) \parallel \text{ArtifactType}$$


$$\text{Digest} = \text{SHA-256}(\text{Header} \parallel \text{CanonicalPayloadBytes})$$



where $\text{Len32}(S)$ is the 4-byte unsigned Big-Endian integer representing byte length of string $S$.

---

### 5. Unbound `MathCondition` Lifecycle during Symbolic AST Transformations

* **The Architectural Deadlock:** Section 10 assigns `MathCondition` to `PhysMath` as pure mathematical predicates ($x \ne 0$, $\det(g) \ne 0$). However, `plan5.md` does not specify how conditions are accumulated during symbolic transformations (e.g., when simplifying $f(x) \cdot \frac{x}{x} \rightarrow f(x)$, generating condition $x \ne 0$).


* **Coding Agent Failure:** Agents will invent inconsistent patterns—such as mutating AST nodes directly, attaching conditions as global side-effects, or discarding conditions entirely—causing non-deterministic replay and loss of mathematical preconditions.


* **Unambiguous Decision:**
1. `PhysMath` AST nodes are **pure and immutable** value types; they do not hold mutable condition fields.
2. All `PhysMath` transformation and rewrite functions MUST use explicit function signatures returning a tuple:

$$\text{Transform}(\text{AST}_{\text{in}}) \longrightarrow (\text{AST}_{\text{out}}, \text{ConditionSet}, \text{TransformationTrace})$$


3. `ConditionSet` is an immutable collection of `MathCondition` records produced during the rewrite step and carried along the transformation pipeline.





---

### 6. Undefined Entry-Point Surface for Frozen `Phys` Ingestion

* **The Architectural Deadlock:** Section 26 states that formalization invokes "existing Phys public APIs" to produce hypothesis-grade results, and Section 27 notes there is no runtime promotion to corpus status. However, `plan5.md` does not specify the exact Go package entry points or method calls in `Phys` used during formalization.


* **Coding Agent Failure:** Agents building `TOE-Agent` formalization tools will attempt to invent non-existent `Phys` APIs (e.g., `phys.AcceptFormalization()`) or construct illegal mock objects.


* **Unambiguous Decision:**
1. `TOE-Agent` interacts with `Phys` exclusively through `phys.NewSession()` and `session.IngestHypothesisExpression(expr)`.
2. If the expression matches `Phys`'s projectable quantity mirror (Section 23), `session.IngestHypothesisExpression` returns `(phys.HypothesisHandle, error)`.


3. The formalization outcome is logged by `TOE-Agent` as `PROJECTED_AS_HYPOTHESIS` alongside the returned `phys.HypothesisHandle`. No corpus mutation occurs.





---

### 7. Parameter Domain Typing for Differential Operators ($\partial_t$ vs $\partial_k$)

* **The Architectural Deadlock:** Section 38 mandates distinct handling for time flow ($\partial/\partial t$) and RG flow ($\partial/\partial k$). Section 8 lists generic `Operator` and `OperatorApplication` sorts without defining how differential operator parameters are distinguished at the `PhysMath` AST level.


* **Coding Agent Failure:** Agents will differentiate $\partial_t$ and $\partial_k$ using raw string matching on variable names (`"t"` vs `"k"`), which breaks under De Bruijn indexing, variable substitution, or local coordinate changes.


* **Unambiguous Decision:**
1. `PhysMath.Operator` nodes representing differential operators MUST carry an explicit `DifferentialDomainTag` enum field:

$$\text{DifferentialDomainTag} \in \{\text{DIFF\_COORDINATE}, \text{DIFF\_TIME}, \text{DIFF\_RG\_SCALE}\}$$


2. `PhysMath` canonicalizers and transformers treat `DIFF_TIME` ($\partial_t$) and `DIFF_RG_SCALE` ($\partial_k$) as distinct operator kinds during AST matching and non-commutative operator algebra.





---

### 8. Exponent Domain Discrepancy in `PhysMath` $\rightarrow$ `Phys` Projection Mirror

* **The Architectural Deadlock:** Section 23 specifies the `PhysMath` $\rightarrow$ `Phys` mirror subset (`Symbol`, `Rational`, `Add`, `Mul`, `Neg`, `Pow`, `Sqrt`). `PhysMath.Pow` allows symbolic expressions in the exponent position ($x^y$ or $e^{i k x}$), whereas frozen `Phys` `Pow` accepts only rational numerical exponents ($p/q$).


* **Coding Agent Failure:** Agents writing the `PhysMath` $\rightarrow$ `Phys` projection transformer will panic or cause unhandled runtime errors when encountering symbolic exponents during formalization attempts.
* **Unambiguous Decision:**
1. During projection evaluation, any `PhysMath.Pow(base, exponent)` where `exponent` is not a `PhysMath.Rational` literal MUST be classified as `UNREPRESENTABLE`.


2. The transformer emits a `ProjectionLossReport` with `classification = UNREPRESENTABLE` and `reason = "NON_RATIONAL_EXPONENT_IN_POW"`, leaving the original `PhysMath` AST unaltered in `PhysMath`.





---

### Summary Table of Closed Architectural Decisions

| ID | Issue Area | Deadlock / Failure Mode | Closed Unambiguous Decision |
| --- | --- | --- | --- |
| **ADR-041** | Identity Codec Ownership | Duplicate `ArtifactRef` definitions across repos.

 | Shared, zero-dependency `[github.com/PithomLabs/phys-core/artifact](https://github.com/PithomLabs/phys-core/artifact)` package.

 |
| **ADR-042** | Mathematical Extension Protocol | Static typing deadlock vs. banned dynamic trait runtimes.

 | `PhysMath` AST sort set is strictly closed; `phys-*` wraps ASTs in physical contracts.

 |
| **ADR-043** | Numerical Lowering Boundary | Missing `PhysMath` dependency in `Numerical`.

 | `phys-*` lowers ASTs to numerical arrays/plans prior to `Numerical` handoff.

 |
| **ADR-044** | Canonical Hash Framing | Collisions from string concatenation.

 | Big-Endian length-prefixed binary framing protocol. |
| **ADR-045** | `MathCondition` Context Lifecycle | Unbound condition state during AST rewrites.

 | Immutable `ConditionSet` returned in `(AST, ConditionSet, Trace)` tuple. |
| **ADR-046** | `Phys` Ingestion API Surface | Non-existent promotion APIs invoked.

 | Restricted to `phys.NewSession()` and `session.IngestHypothesisExpression()`.

 |
| **ADR-047** | Flow Operator Parameter Typing | String-based symbol matching for $\partial_t$ vs $\partial_k$.

 | Explicit `DifferentialDomainTag` enum on `PhysMath.Operator` AST nodes.

 |
| **ADR-048** | Projection Exponent Domain | Runtime crash on symbolic exponents in `Pow`.

 | Non-rational exponents map to `UNREPRESENTABLE` with loss report.

 |