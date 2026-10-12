## Adversarial Review — `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.0.md`

**Context reviewed:** the Phase-1 representation spec against `plan8.md`, v1.5’s frozen decisions, and the stated target — “no architectural blockers and no ambiguity for a coding agent about architectural decisions.”

**Verdict: NOT CLOSED. NINE DEMONSTRATED BLOCKERS.**

The spec achieves the crucial *architectural* move that plan8 asked for: it separates Phase-1 representation (PhysMath + `phys-*`) from Phase-2 research/integration (TOE-Agent, Numerical, provenance, formalization), makes the equivariance theorem the reference slice, and puts paper/prose/analysis/interpretation outside PhysMath. Its constitutional decisions (§0.4, §1.2, §2, §3, §24.2, §27) are correct and should not be reopened.

What it does *not* do is finish the job of pinning every durable artifact’s **family ID, primitive encoding, and target family** so that a coding agent handed only this spec can produce byte-identical artifacts. In the process of slimming down from v1.5 it removed material that a coding agent still needs — most notably the arbitrary-precision integer encoding and several artifact-family registrations.

The pattern is the one this series has now seen five times: the architecture is right, the contract is not yet mechanically complete.

---

## I. What the spec gets right (preserve)

- §0.3–§0.4 scope correctly isolates Phase-1 from Phase-2.
- §1.2 cleanly separates Analysis / Representation / Interpretation.
- §1.1 keeps the physics paper outside PhysMath; only its formal payload becomes representable.
- §2.1–§2.2 cleanly partition mathematical vs physical ownership.
- §2.3 affirms the “many physicists, many answers” principle structurally.
- §2.4 correctly denies that representation is truth.
- §3 pins `PhysMath -> phys-artifact`, `phys-* -> {PhysMath, phys-artifact, Phys public API}`, and freezes Phys.
- §4.2–§4.5 give the ArtifactRef type, digest framing, embedded-ref encoding, and primitive vocabulary, including the field-order-is-wire-order rule.
- §5.3 pins `SchemaVersion = "1"`.
- §6.1 fixes the 16-sort closed inventory; §6.2 marks the equivariance-required subset as the implementation priority.
- §7.1 fixes the ten Expression node tags; §7.3–§7.11 give per-node schemas with validation rules.
- §8 pins `ParameterID`, `BoundParameterRef`, alpha-renaming invariance, and the t/k rule.
- §9 pins the mathematical constants registry and physical-constant neutrality.
- §10.1–§10.8 give concrete higher-sort schemas with target-family hints.
- §11.1–§11.5 pin Equation, EquationSet, Definition, MathCondition, and the closed `TransformationRuleID` set.
- §12 pins `DerivationStep` and `Derivation` with determinism and no-hidden-inference rules.
- §13 pins `Theorem` and its formal meaning.
- §14 pins framework identity, physical assumptions, physical definitions/equations/theorems/interpretations, and the "many frameworks may disagree" rule.
- §15 pins theory-local algebra containment and deterministic mapping into PhysMath.
- §16 pins the equivariance theorem reference slice at the architecture level.
- §17.2 pins the differentiate rules that the reference slice needs.
- §18 pins canonical child ordering and SET/SEQUENCE discipline.
- §19 pins equality classes and denies automatic physical equivalence.
- §20 pins canonical serialization, transport separation, local replay, and golden vectors.
- §21 pins `phys-*` conformance and its negative tests.
- §22 pins the Phase-1 no-go list.
- §23 explicitly defers TOE-Agent, Numerical, and trust/provenance.
- §24 pins the coding-agent’s fixed decisions and permitted decisions with a stop rule.
- §25–§26 give the acceptance suite and a short implementation order.
- §27 restates the permanent invariants.
- §28 gives a concrete handoff condition.

The *shape* is right. The *closure* has gaps.

---

## II. Demonstrated blockers

Each blocker identifies the section, a concrete counterexample, and the failure consequence, per the review standard set out in the plan series.

### Blocker 1 — Durable artifact-family registry is absent

- §5.2 declares: “Artifact types MUST be explicitly registered in the owning package contract. No implementation may invent an unregistered durable artifact type.”
- §5.1 names namespaces but never enumerates `<Namespace>/<ArtifactType>/<SchemaVersion>` for the durable artifact families the spec itself defines.
- The spec defines schemas for `Equation` (§11.1), `EquationSet` (§11.2), `Definition` (§11.3), `Derivation` (§12.3 — family given), `Theorem` (§13.1 — family given), `PhysicalAssumption` (§14.2), `PhysicalDefinition` (§14.3), `PhysicalEquation` (§14.4), `PhysicalTheorem` (§14.5), `PhysicalInterpretation` (§14.6), `FrameworkArtifact` (§14.1 — family given), `Relation` (referenced at §11.1 as `physmath/relation/1`), `Condition` (§11.4 — family given). No family ID is given for the first four Math artifacts (`Equation`, `EquationSet`, `Definition`) nor for the four `phys-*` artifacts (`PhysicalAssumption`, `PhysicalDefinition`, `PhysicalEquation`, `PhysicalTheorem`, `PhysicalInterpretation`).

**Concrete counterexample:** A coding agent implementing `Equation` must choose between `physmath/equation/1`, `physmath/eq/1`, `physmath/math_equation/1`, or `physmath/relation_equation/1`. Two implementations produce different family IDs for identical semantic content; their `Equation` artifacts are not interchangeable; the `ArtifactRef` computed by each implementation differs.

**Failure consequence:** No cross-implementation portability of any `Equation`, `EquationSet`, `Definition`, or `phys-*` artifact. `Derivation.PremiseRefs`, `Theorem.PremiseRefs`, `PhysicalEquation.EquationRef`, and similar references cannot be validated across implementations. §28’s “no invention” handoff condition cannot be met.

**Exit condition:** Enumerate `<Namespace>/<ArtifactType>/<SchemaVersion>` for every durable family the spec defines.

---

### Blocker 2 — Arbitrary-precision integer primitive encoding is missing

- §7.4 uses `Numerator BigInt` and `Denominator BigInt`.
- §4.5 lists `UInt8, UInt16, UInt32, UInt64, Digest32` but **not** `BigInt`.
- §4.6 forbids `Float64Bits` in exact payloads but does not define its replacement.
- v1.5 defined `BigInt` (sign byte + length-prefixed magnitude); the slimmed spec dropped it.

**Concrete counterexample:** A coding agent must encode `Rational(-3, 7)`. Option A: sign-magnitude with `0x01` sign byte and minimal big-endian magnitude. Option B: two's-complement fixed-width. Option C: decimal string. All satisfy every other rule in the spec. Their canonical bytes differ; their digests differ; their Rational artifacts are non-portable.

**Failure consequence:** Every Rational node, every Symbol that participates in rational arithmetic, and every DerivationStep whose parameters reference a Rational produce implementation-dependent bytes.

**Exit condition:** Define `BigInt` as a canonical primitive with sign byte, length framing, and minimal-magnitude rules, matching v1.5 §5.5 (or an equivalent explicit form).

---

### Blocker 3 — `ComparisonOperator` canonical encoding is not closed

- §7.10 defines a closed vocabulary: `EQ, NEQ, LT, LTE, GT, GTE`.
- §4.5 defines two encodings that could apply: `Enum = canonical String of a closed token` and `TaggedUnion = UInt8 discriminator || variant payload`.
- §7.10 does not select one.

**Concrete counterexample:** A coding agent implementing the Relation node body must choose. Option A encodes `EQ` as `Len32(2) || "EQ"` (Enum). Option B encodes `EQ` as `0x01` (TaggedUnion). The Relation artifacts, their canonical bytes, their digests, and any `Equation` wrapping them diverge across implementations.

**Failure consequence:** Relation artifacts are non-portable; §11.1 Equation.RelationRef cannot be validated cross-implementation; §12.3 Derivation.PremiseRefs that cite a relation cannot be replayed.

**Exit condition:** Declare `ComparisonOperator` as `Enum` or `TaggedUnion`, with the exact byte mapping.

---

### Blocker 4 — `Relation` durable artifact payload is undefined

- §7.10 defines the *Expression node kind* `Relation { Operator, Left, Right }`.
- §11.1 says `Equation.RelationRef MUST target physmath/relation/1`, implying a *durable* artifact family named `physmath/relation/1`.
- The spec never defines the payload schema for `physmath/relation/1`, nor distinguishes it from the Expression node.

**Concrete counterexample:** A coding agent must decide whether the durable `physmath/relation/1` payload is (a) the same `{Operator, Left, Right}` record as the Expression node; (b) a wrapper `{ RelationNode }`; (c) a body without the outer tag; or (d) something else. Two implementations produce different canonical payloads for the same semantic relation.

**Failure consequence:** `Equation` and `MathCondition` both reference `physmath/relation/1`; without a defined payload, their references cannot be validated.

**Exit condition:** Define the `physmath/relation/1` payload schema and state its relationship to the Expression node kind.

---

### Blocker 5 — `TransformationRuleID` parameter schemas are incomplete

- §11.5 pins the closed registry: `EXPR-CANON-001, EXPR-SIMPLIFY-001, EXPR-SUBSTITUTE-001, RAT-NORMALIZE-001, DIFFERENTIATE-001, SELECT-BRANCH-001`.
- §17.2 pins DIFFERENTIATE-001’s derivative rules.
- §17.4 pins EXPR-SUBSTITUTE-001’s parameter schema.
- No section defines the canonical `Parameters` bytes for `EXPR-CANON-001`, `EXPR-SIMPLIFY-001`, `RAT-NORMALIZE-001`, or `SELECT-BRANCH-001`.
- §24.2 says the coding agent must not decide the rule semantics, but the coding agent is not told what the rule parameters are.

**Concrete counterexample:** A coding agent must implement `DerivationStep.Parameters` for `SELECT-BRANCH-001`. Option A: `UInt32BE(BranchOrdinal)`. Option B: `Bytes(ordinal_string)`. Option C: empty (branch selected by context). None is illegal under the spec as written. Two implementations produce different DerivationStep bytes for the same branch selection.

**Failure consequence:** Derivation and Theorem artifacts are non-portable; §20.4 golden vectors cannot be canonicalized because the rule parameters are ambiguous.

**Exit condition:** For each of the six RuleIDs, declare the exact `Parameters` byte schema.

---

### Blocker 6 — `Encode(BoundParameterRef)` is not defined

- §10.7 says, for `differentiate`:
  ```
  Parameters = Encode(BoundParameterRef) || UInt32BE(Order)
  ```
- §8.5 defines `BoundParameterRef { DeclaringArtifactRef, ParameterID }`.
- No section defines the canonical encoding of a `BoundParameterRef`.
- `ParameterID` is `[32]byte`, but the spec does not say whether it is embedded as `Digest32` or as a `Bytes` frame wrapping the 32 bytes.
- The spec does not say whether `DeclaringArtifactRef` uses the embedded-ref encoding of §4.4.

**Concrete counterexample:** A coding agent must encode `differentiate` parameters. Option A: `EmbeddedRef(DeclaringArtifactRef) || ParameterID[32]`. Option B: `EmbeddedRef(DeclaringArtifactRef) || Bytes(ParameterID)`. Option C: `EmbeddedRef(DeclaringArtifactRef) || Digest32(ParameterID)`. All are legal under the spec as written. Two implementations produce different canonical bytes for the same differentiation request.

**Failure consequence:** DIFFERENTIATE-001 traces are non-portable; the equivariance reference slice’s derivation cannot be replayed across implementations.

**Exit condition:** Define `Encode(BoundParameterRef)` explicitly.

---

### Blocker 7 — `ConstantID` type is not declared

- §7.3 uses `ConstantID Optional<ConstantID>`.
- §9.1 gives string values (`math.const.pi`, `math.const.e`, `math.const.i`).
- No section declares `type ConstantID` or its canonical encoding.
- The spec does not say whether `ConstantID` is a `string`, a `[N]byte`, or an enum-like token.

**Concrete counterexample:** A coding agent must encode `Symbol { DisplayName: "π", ConstantID: "math.const.pi" }`. Option A: `String("math.const.pi")` per §4.5. Option B: `Digest32(SHA-256("math.const.pi"))` for compactness. Both are compatible with the text. Two implementations produce different Symbol bytes.

**Failure consequence:** Symbol identity differs across implementations; any expression containing a mathematical constant is non-portable.

**Exit condition:** Declare `ConstantID` as a concrete type and its canonical encoding.

---

### Blocker 8 — Target families for embedded `ArtifactRef` fields are largely unspecified

- §10.1 states `Function.BodyRef` targets `physmath/expression/1`.
- §10.2 states `FunctionApplication.FunctionRef` targets `physmath/function/1`.
- §10.4 `Tensor.IndexRefs` and `Tensor.ComponentRefs` — no target family.
- §10.5 `Index.SpaceRef` — no target family.
- §10.3 `Map.DefinitionRef` — described as “an explicit mathematical definition artifact,” but no family ID for a `Definition` artifact is provided (Blocker 1).
- §10.6 `Collection.ElementRefs` — no target family.
- §11.1 `Equation.RelationRef` — target family is stated.
- §11.3 `Definition.DefinedRef` and `Definition.DefiningRef` — no target family.
- §14.2–§14.6 `FrameworkRef`, `MathematicalRefs`, `AssumptionRefs`, `DefinitionEquation`, `ScopeRefs`, `InterpretationRefs` — no target families.
- §13.1 `Theorem.PremiseRefs`, `ConclusionRef`, `AssumptionRefs`, `DerivationRef` — targets not stated.

**Concrete counterexample:** A coding agent implementing `Tensor.ComponentRefs` must choose between `physmath/expression/1`, `physmath/collection/1`, `physmath/tensor/1`, or a new family. Each choice yields a different, self-consistent implementation, and the tensors are non-portable.

**Failure consequence:** Cross-artifact validation cannot be performed; the reference slice's tensors and indices are non-portable.

**Exit condition:** For every `ArtifactRef`-typed field in every durable schema, declare its target family (or explicitly declare it “open” and specify what validation is possible).

---

### Blocker 9 — `phys-*` artifact families are not registered

- §14.2–§14.6 define `PhysicalAssumption`, `PhysicalDefinition`, `PhysicalEquation`, `PhysicalTheorem`, `PhysicalInterpretation`.
- None of these is assigned a family ID.
- §14.1 assigns `FrameworkArtifact` the family `<phys-namespace>/framework/1`, but no analogous assignment is made for the others.
- The natural families would be `<phys-namespace>/assumption/1`, `<phys-namespace>/definition/1`, `<phys-namespace>/equation/1`, `<phys-namespace>/theorem/1`, `<phys-namespace>/interpretation/1` — but the spec does not state this.

**Concrete counterexample:** A coding agent implementing `phys-gr` must decide whether `PhysicalEquation` uses `phys-gr/equation/1`, `phys-gr/phys_equation/1`, `phys-gr/physical_equation/1`, or another string. Two packages (or two implementations of the same package) diverge.

**Failure consequence:** The reference slice’s `phys-*` framework artifacts cannot be cross-validated; §16.6’s “many frameworks, many answers” acceptance test cannot be enforced across implementations.

**Exit condition:** Register the exact family ID for every `phys-*` durable artifact type.

---

## III. Residual issues

These are real gaps but not demonstrated blockers under the review standard, in the sense that they do not force a choice among architecturally legitimate implementations — they force a choice among *implementation conveniences*.

- §14.2 `Category` is a free-form `string`; whether it is closed or open is unstated.
- §14.3 `DefinitionEquation Optional<ArtifactRef>` — target family not stated.
- §14.5 `InterpretationRefs` — target family not stated.
- §16.3 lists required represented objects without stating which artifact family each must be.
- §20.4 lists golden-vector families by informal name (`ArtifactRef encoding`, `DIFFERENTIATE-001 parameters`) rather than by family ID.
- §12.4 “all same-run inputs are available” is not formalized.
- §15.3 requires mapping documentation but does not require it to be a durable artifact.
- §17.4’s substitution parameter set is described but not given a canonical byte schema (SET ordering is implied but not pinned).
- §25.6 add/mul normalization test depends on §18.1’s “canonical encoded child bytes” — which itself depends on §4.5 primitives and Blocker 2’s missing `BigInt`.
- §11.4 MathCondition is described as family `physmath/condition/1` but the surrounding §11 does not assign families to the other Math support artifacts.
- §12.2 “DerivationStep.Ordinal = UInt32 unique sequential position starting at 0” — the phrase “unique sequential” is redundant if `Ordinal` is the sequence index; the spec should say whether the list order or the field value governs.
- §7.3 validation distinguishes “durable symbol” and “ephemeral unbound symbol” but no schema distinguishes them; a coding agent must infer from context.

---

## IV. Path to closure

To satisfy §28’s handoff condition, the spec must resolve the nine blockers above. Concretely:

1. **Blocker 1 + Blocker 9.** Add a `§5.x Artifact Family Registry` section enumerating `<Namespace>/<ArtifactType>/<SchemaVersion>` for every durable family the spec defines, including `physmath/equation/1`, `physmath/equation_set/1`, `physmath/definition/1`, `physmath/relation/1`, `physmath/derivation/1`, `physmath/theorem/1`, and, for each `phys-*` namespace, `<ns>/framework/1`, `<ns>/assumption/1`, `<ns>/definition/1`, `<ns>/equation/1`, `<ns>/theorem/1`, `<ns>/interpretation/1`.
2. **Blocker 2.** Restore the canonical `BigInt` primitive: sign byte, length prefix, minimal big-endian magnitude (matching v1.5 §5.5).
3. **Blocker 3.** Declare `ComparisonOperator` as `Enum` (recommended) or `TaggedUnion`, with exact bytes.
4. **Blocker 4.** Define the `physmath/relation/1` payload and its relation to the Expression node kind.
5. **Blocker 5.** For each of the six `TransformationRuleID`s, declare the exact `Parameters` byte schema.
6. **Blocker 6.** Define `Encode(BoundParameterRef)`: `EmbeddedRef(DeclaringArtifactRef) || ParameterID[32]`.
7. **Blocker 7.** Declare `type ConstantID string` and its canonical `String()` encoding.
8. **Blocker 8.** For every `ArtifactRef` field in every schema, declare its target family — or explicitly declare it open and specify the validation rule.

Apply the residual clarifications. Then add golden vectors for each resolved family and each rule’s parameters.

---

## V. Final verdict

```text
REVIEW: PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.0

Phase-1 / Phase-2 separation:               PASS
Paper-as-research-unit rule:                PASS
ARI boundary:                               PASS
PhysMath purity:                            PASS
phys-* ownership of physical meaning:       PASS
Many-frameworks principle:                  PASS
Representation != truth:                    PASS
Dependency graph:                           PASS
ArtifactRef type and identity:              PASS
Canonical digest framing:                   PASS
Embedded ArtifactRef encoding:              PASS
Primitive encoding (UInt*/String/Bytes):    PARTIAL — BigInt missing (Blocker 2)
Float64Bits exclusion:                      PASS
Namespace grammar:                          PASS
Artifact-family registry:                   FAIL (Blocker 1, Blocker 9)
Expression node tags:                       PASS
Symbol/Rational/Add/Mul/Neg/Pow/Sqrt:       PASS (given BigInt)
Call:                                       PASS
Relation node:                              PARTIAL (Blocker 3, Blocker 4)
BranchSet:                                  PASS
ParameterID formula:                        PASS
BoundParameterRef:                          PASS (but Blocker 6 on its encoding)
ConstantID:                                 FAIL (Blocker 7)
Function/FunctionApplication/Map:           PARTIAL (Blocker 8)
Tensor/Index/Collection:                    PARTIAL (Blocker 8)
OperatorApplication differentiate:          PASS
Transformation registry:                    PARTIAL (Blocker 5)
Equation/EquationSet/Definition:            PARTIAL (Blocker 1, Blocker 4)
MathCondition:                              PASS
Derivation / DerivationStep:                PASS (subject to rule parameters)
Theorem:                                    PASS (subject to target families)
FrameworkArtifact:                          PASS
PhysicalAssumption:                         FAIL (Blocker 9)
PhysicalDefinition:                         FAIL (Blocker 9)
PhysicalEquation:                           FAIL (Blocker 9)
PhysicalTheorem:                            FAIL (Blocker 9)
PhysicalInterpretation:                     FAIL (Blocker 9)
Theory-local algebra rule:                  PASS
Equivariance reference slice:               PASS at architectural level
Operation semantics for differentiate:      PASS
Canonicalization / SET / SEQUENCE:          PASS
Equality rules:                             PASS
Serialization / replay / golden vectors:    PASS
phys-* conformance:                         PASS
Phase-1 no-go list:                         PASS
Secondary-system boundary:                  PASS
Implementation contract / stop rule:        PASS
Acceptance suite:                           PASS as a set

FINAL STATUS: NOT CLOSED. NINE DEMONSTRATED BLOCKERS.
```

---

## VI. An observation about the review cycle itself

`plan8.md` correctly recommended *splitting* the architecture so the Phase-1 representation spec could be frozen without dragging Phase-2 concerns behind it. The spec does that at the conceptual level.

But in the process of slimming v1.5, the spec removed two things a coding agent cannot reconstruct:

- the **artifact-family registry** that gives every durable object an ecosystem-stable identity, and
- the **primitive encoding of `BigInt`** that every `Rational` depends on.

Both were present in v1.5 and were the correct calls to make. Restoring them in a v1.1 amendment will not reopen the Phase-1/Phase-2 separation and will not re-entangle the architecture with provenance, trust, or research state. The remaining blockers — `ComparisonOperator` encoding, the `Relation` durable payload, `Encode(BoundParameterRef)`, `ConstantID` typing, `phys-*` family IDs, and the four pending rule-parameter schemas — are equally bounded.

None of them requires re-litigating the constitutional architecture. They require the one thing the spec is closest to: a single pass that turns every named artifact type into a registered, typed, canonically-encoded family, and every rule into a rule with exact parameter bytes. Once that is done, §28’s handoff condition can be honestly claimed.