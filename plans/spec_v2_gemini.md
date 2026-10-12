An adversarial audit of `spec_v2.md` (`PM-REP-001` Consolidated Decisions) against the target of **zero architectural blockers and zero ambiguity for a coding agent** confirms that while strategic alignment is complete, **6 critical mechanical blockers** still prevent a coding agent from writing deterministic, hash-interoperable code.

While `spec_v2.md` successfully locks high-level decisions (such as `BigInt` sign/magnitude framing, `ConstantID` representation, and `EXPR-APPLY-001` existence), a developer or AI coding agent attempting to implement `PhysMath`, `phys-*`, and the equivariance reference slice will immediately hit binary serialization ambiguities, missing schema layouts, and parameter scope gaps.

---

### Blocker 1: Binary Layout Ambiguity for `Bytes(T)` Wrappers & `ArtifactRef` Wire Format

* **Sections:** §2.C, §2.D, §4.I, §8.R
* **Issue:**
1. §2.C defines `ParameterID = SHA-256(String("physmath") || Bytes(ScopePathBytes) || UInt32BE(ParameterOrdinal))`, while §2.D states: "`Encode(BoundParameterRef) = EncodeArtifactRef(DeclaringArtifactRef) || Digest32(ParameterID)`. No `Bytes()` wrapper surrounds the 32-byte ParameterID." However, the spec never defines the canonical binary serialization of `Bytes(X)` itself.
2. The specification lacks an explicit binary wire framing definition for `EncodeArtifactRef(ref)`.


* **Ambiguity Counterexample:**
* Developer A interprets `Bytes(ScopePathBytes)` as length-prefixed raw bytes: `UInt32BE(4) || 0x00000000` (8 bytes total).
* Developer B interprets `Bytes(ScopePathBytes)` as inline raw bytes: `0x00000000` (4 bytes total).
* Developer A serializes `ArtifactRef` as `String(ArtifactType) || Digest32` (`UInt32BE(len) || UTF8(Type) || SHA256_32bytes`).
* Developer B serializes `ArtifactRef` as `Digest32` only, relying on out-of-band context.


* **Failure Consequence:** `ParameterID` digests and `BoundParameterRef` binary outputs diverge across engines, causing immediate verification failure in cross-implementation DAG checks.
* **Required Fix:**
* Define generic `Bytes(data)` explicitly as `UInt32BE(len(data)) || data`.
* Define `EncodeArtifactRef(ref)` normatively as `String(ref.ArtifactType) || ref.Digest` (where `String(s)` is `UInt32BE(len(s)) || UTF8(s)` and `ref.Digest` is raw `[32]byte`).



---

### Blocker 2: Missing Canonical Struct Schemas & Binary Layouts for `phys-*` Artifact Families

* **Sections:** §2.A, §4.I, §8.P, §8.R
* **Issue:** §2.A and §8.P register six canonical `phys-*` artifact families:
* `<namespace>/framework/1`
* `<namespace>/assumption/1`
* `<namespace>/definition/1`
* `<namespace>/equation/1`
* `<namespace>/theorem/1`
* `<namespace>/interpretation/1`


However, `spec_v2.md` provides **zero field layouts or binary struct definitions** for these six families. §8.R demands "canonical serialization vector" and "exact collection semantics," but a coding agent has no schema to serialize.
* **Failure Consequence:** An agent generating `phys-qm/framework/1` or `phys-bohm/assumption/1` must invent struct fields (e.g., guessing whether `PhysicalDefinition` uses `Name String` or `Symbol String`, or whether fields are `ArtifactRef` vs `[]ArtifactRef`), breaking canonical SHA-256 calculation.
* **Required Fix:** Provide the exact field-by-field Go struct / binary wire layout for all 6 `phys-*` families, specifying field sequence, field types, and optionality.

---

### Blocker 3: Free Variable Binding Mechanism for Standalone Equations & Equivariance Fixture

* **Sections:** §2.C, §2.D, §5.L
* **Issue:** §2.C states: *"Phase-1 supports Function parameter binding as the canonical binding mechanism... Nested lexical binders are not part of the Phase-1 binder vocabulary."*
In the reference slice (§5.L), the equivariance relation is $f(\rho_X(g, x)) = \rho_Y(g, f(x))$. Inside the expressions, $g$ and $x$ appear as `Symbol` AST nodes requiring `ParameterID`s.
If `Function` is the *only* parameter binding mechanism in Phase 1, how are free variables $g \in G$ and $x \in X$ declared and assigned `ParameterID`s in a standalone `physmath/equation/1` or `physmath/relation/1` that is outside any `Function` body?
* **Failure Consequence:** A coding agent cannot construct the `Symbol` nodes for $g$ and $x$ in the equivariance relation because there is no declaring `Function` artifact to supply `DeclaringArtifactRef` or calculate `ParameterID = SHA-256(...)`.
* **Required Fix:** Specify either:
1. The equivariance relation parameters $g$ and $x$ MUST be bound by dummy/wrapper `Function` artifacts (or universal quantifier definitions) that declare $g$ and $x$ as `ParameterIDs`, OR
2. Define an explicit `Equation.ParameterIDs []ParameterID` binding field on `physmath/equation/1`.



---

### Blocker 4: Inlined Expression AST vs Durable `physmath/expression/1` Boundary Mismatch

* **Sections:** §2.E, §3.G, §4.I
* **Issue:** §2.A registers `physmath/expression/1` as a durable artifact family. §4.I states `Function.BodyRef -> physmath/expression/1`. However:
1. §2.E defines `physmath/relation/1` as containing inlined `CanonicalExpression(Left)` and `CanonicalExpression(Right)` AST nodes (not `ArtifactRef`s).
2. §3.G specifies `EXPR-SUBSTITUTE-001` parameters as `EncodeArtifactRef(ReplacementExpression)`, requiring replacement targets to be durable `physmath/expression/1` artifacts.
3. §3.G specifies `EXPR-APPLY-001` input `Input[0]` as "Expression containing a root Call". If `Input[0]` is a `physmath/expression/1` durable `ArtifactRef`, the rule does not specify how to apply function expansion to a `Call` node nested *inside* a sub-expression (e.g., $1 + f(x)$).


* **Failure Consequence:**
* Agents will mix inlined expression byte streams with `ArtifactRef` digests unpredictably.
* Engine evaluations will fail on nested function applications because `EXPR-APPLY-001` is restricted to root `Call` nodes without a path selector mechanism.


* **Required Fix:**
* Explicitly state that an `Expression` node tree is strictly inlined inside payload containers (`Function.Body`, `Relation.Left/Right`), and `physmath/expression/1` durable artifacts exist ONLY as top-level canonical roots for transformation inputs/outputs.
* Define an optional `NodePath []UInt32` field in `EXPR-APPLY-001` parameter payload to designate targeted nested `Call` nodes.



---

### Blocker 5: Missing Binary Layouts for `physmath/derivation/1`, `DerivationStep`, and `physmath/transformation/1`

* **Sections:** §2.A, §3.G, §7.O, §10
* **Issue:** §2.A registers `physmath/derivation/1` and `physmath/transformation/1`. §7.O specifies that transformation steps must carry `(ProducerNamespace, RuleID)`. However, the spec omits the precise byte-level payload schema for these structs.
* **Missing Details:**
* Is `DerivationStep` an embedded struct within `physmath/derivation/1` or its own durable artifact family? §2.A does not list `physmath/derivation_step/1`.
* What is the exact sequence of fields in `DerivationStep`?


* **Required Fix:** Normatively define the byte schema for `physmath/derivation/1` and `physmath/transformation/1`:

```text
DerivationPayload =
    UInt32BE(StepCount)
    || Step_0 || ... || Step_(N-1)

DerivationStep =
    UInt32BE(StepOrdinal)
    || String(ProducerNamespace)
    || String(RuleID)
    || Bytes(Parameters)
    || UInt32BE(InputCount) || EncodeArtifactRef(Input_0) || ...
    || EncodeArtifactRef(OutputRef)

```

---

### Blocker 6: Missing Global Generic Serialization Rules for `Optional<T>`, `SET<T>`, and `SEQUENCE<T>`

* **Sections:** §2.B, §8.R
* **Issue:** §8.R references `Optional`, `SET`, and `SEQUENCE` serialization primitives, but no central normative section defines their wire tags and collection ordering.
* **Failure Consequence:** Developers will vary between 1-byte vs 4-byte presence flags for `Optional<T>`, and will disagree on canonical byte-sorting algorithms for `SET<T>`.
* **Required Fix:** Add a normative Primitive Collection Encoding section:
* `Optional<T>`: `0x00` if absent; `0x01 || CanonicalEncoding(T)` if present (1-byte discriminator).
* `SEQUENCE<T>`: `UInt32BE(count) || CanonicalEncoding(T_0) || ... || CanonicalEncoding(T_(N-1))`.
* `SET<T>`: `UInt32BE(count) || CanonicalEncoding(T_sorted_0) || ... || CanonicalEncoding(T_sorted_(N-1))` where items are sorted in ascending lexicographical byte order (`memcmp`).



---

### Summary of Architectural & Binary Gaps

| Blocker | Specification Section | Failure Mode | Required Action |
| --- | --- | --- | --- |
| **1. `Bytes(T)` & `ArtifactRef` Framing** | §2.C, §2.D, §4.I | Incompatible SHA-256 hashes for `ParameterID` & `BoundParameterRef` | Define `Bytes(d) = UInt32BE(len) || d` and `EncodeArtifactRef = String(Type) || Digest32` |
| **2. `phys-*` Struct Schemas** | §2.A, §8.P, §8.R | Inability to serialize physical framework/assumption/theorem artifacts | Define explicit field-by-field structs for all 6 `phys-*` families |
| **3. Equivariance Variable Scope** | §2.C, §5.L | Inability to compute valid `ParameterID`s for $g$ and $x$ in equations | Specify parameter binding container for equation/relation free variables |
| **4. Inlined AST vs Durable `Expression**` | §2.E, §3.G, §4.I | Expression AST parsing failures & failure to transform nested `Call` nodes | Clarify AST inlining rules & add `NodePath` parameter to `EXPR-APPLY-001` |
| **5. Derivation & Transformation Layout** | §2.A, §3.G, §7.O | Inability to serialize derivation steps deterministically | Define normative field order and byte layouts for `Derivation` and `DerivationStep` |
| **6. Generic Collection Wire Rules** | §2.B, §8.R | Divergent serialization for optional fields and sets | Define normative wire tags for `Optional<T>`, `SEQUENCE<T>`, and byte-sorted `SET<T>` |