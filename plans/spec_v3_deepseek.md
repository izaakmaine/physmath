## Adversarial Review — `PM-REP-001 v1.1`

**Context reviewed:** v1.1 against its own handoff rule (§28) and the target: a coding agent, given only v1.1, the frozen Phys authority, and the GR example, must not have to invent any architectural decision.

**Verdict:**  
**NOT CLOSED. SIX DEMONSTRATED BLOCKERS REMAIN.**

v1.1 resolves all seven blockers from the consolidated review of v1.0 and adds the GR-style protocol packaging (§1.4), the closed rule registry with exact parameters and semantics (§17), the closed ArtifactRef target-family map (§5.7), the base/specialized framework dependency closure (§14.8, §14.9), and the exact equivariance fixture (§16). The strategic closure achieved by `spec_v2` is preserved and extended.

But v1.1 does not yet satisfy its own §28 handoff rule. Six issues remain where two legitimate implementations are possible, or a required schema/rule is absent. Each meets the blocker test used in this review chain: contradiction between normative rules, two legitimate implementations remaining, missing schema/type/owner/validation, or the coding agent must invent architecture.

---

## I. Blockers resolved from the v1.0 review

| v1.0 Blocker | v1.1 Status |
|---|---|
| Artifact-family registry | **Resolved.** §5.6 enumerates all PhysMath and `phys-*` families. |
| BigInt encoding | **Resolved.** §4.6 gives sign + length + minimal magnitude. |
| ConstantID encoding | **Resolved.** §9.1 pins `ConstantID string` with `String()`. |
| Closed-enum encoding | **Resolved.** §4.5 `Enum = String(canonical-token)`. |
| ParameterID scope rules | **Resolved.** §8.2 pins Function-only binding, depth 0. |
| BoundParameterRef encoding | **Resolved.** §4.10 pins `EncodeArtifactRef || Digest32(ParameterID)`. |
| Relation node vs durable payload | **Resolved.** §7.11 + §19.2 + §16.10 pin the relation body. |
| ComparisonOperator encoding | **Resolved.** §7.11 uses `Enum(Operator)`. |
| Rule parameter schemas | **Resolved.** §17.2–§17.9 pin parameters for all seven rules. |
| ArtifactRef target families | **Resolved.** §5.7 is a closed target-family map. |
| `EXPR-APPLY-001` existence | **Resolved.** §17.7. |
| Substitution declaring-artifact rule | **Resolved.** §17.8 requires the declaring Function in the inputs. |
| `ProducerNamespace` in rule records | **Resolved.** §10.8, §12.2 add `ProducerNamespace`. |
| FrameworkVersion canonical form | **Resolved.** §5.5 pins opaque string with `String()` encoding. |
| `OperationResult` schema | **Resolved.** §17.1 pins `TransformResult { OutputRef, ConditionRefs }`. |
| phys-* cross-namespace assumptions | **Resolved.** §14.9 dependency closure. |

---

## II. Demonstrated blockers

### Blocker 1 — Condition provision to a rule is not defined

- §17.4 states the `Pow(x, 0) -> Rational(1/1)` rewrite is permitted "only when an explicit `MathCondition x != 0` is present in the derivation context."
- §17.1 pins the operation shape as `Apply(...) (TransformResult, error)` — arguments unspecified.
- §17.2's rule contract table says `EXPR-SIMPLIFY-001` has **arity 1** and **empty parameters**.
- §12.2's `DerivationStep.ConditionRefs` is a SET of `ArtifactRef`s, but the rule has no channel to read them.
- §17.10 says "conditions MUST be explicit" but does not say how a rule receives them.

**Concrete counterexample:** A coding agent implementing `EXPR-SIMPLIFY-001` must decide how the `x != 0` condition reaches the rule. Option A: extend `Apply` to take an ambient condition set. Option B: add the condition to the rule's `Inputs`. Option C: add a fifth parameter to `TransformResult`-consuming APIs. Option D: require the caller to pre-populate `DerivationStep.ConditionRefs` and have the rule read a step-scoped context. Each is consistent with §17.1–§17.10 as written.

**Failure consequence:** The `Apply` API, the `TransformResult` payload, and the `DerivationStep` validation are non-portable across implementations. A derivation using `EXPR-SIMPLIFY-001` on a `Pow(x, 0)` cannot be replayed cross-implementation.

**Exit condition:** Declare the exact mechanism by which a rule that requires a condition receives it (input arity change, ambient context, explicit `ConditionRefs` argument, etc.) and reconcile with §17.2's arity table.

---

### Blocker 2 — Fixture `Symbol` DisplayNames are not pinned

- §7.4 includes `DisplayName` in the `Symbol` schema. §4.8 says record fields are encoded in declaration order, so `DisplayName` **is** part of canonical bytes.
- §8.4 says alpha-renaming does not affect `ParameterID`. That does not affect the `Symbol`'s canonical bytes.
- §16.5 says `g = Symbol(ParameterID = PID_0)` and `x = Symbol(ParameterID = PID_1)`. It does not pin `DisplayName`.
- §16.9 and §25.20 rely on golden `ArtifactRef`s for the Step 2 and Step 5 outputs.
- §16.5 also does not pin the `DisplayName` used inside `rho_X`, `rho_Y`, or `f`'s bodies — but the LHS/RHS inherit their final `DisplayName`s from the caller's arguments, so the fixture's golden hashes depend on the caller's `DisplayName` choices.

**Concrete counterexample:** Implementation A chooses `g = Symbol(PID_0, "g")` and `x = Symbol(PID_1, "x")`. Implementation B chooses `g = Symbol(PID_0, "u")` and `x = Symbol(PID_1, "v")`. Both pass every validation rule in v1.1. The Step 2 output canonical bytes differ; the golden `ArtifactRef`s differ; §25.25's cross-implementation identity assertion fails.

**Failure consequence:** The reference fixture's golden vectors are implementation-dependent. §16.9's "expected output `ArtifactRef`s for every step" cannot be pinned across implementations.

**Exit condition:** Pin the exact `DisplayName` values for the fixture's symbols (including caller-side `g`, `x` and function-body-side symbols), or exclude `DisplayName` from canonical bytes, or state that the fixture does not require matching golden hashes across implementations.

---

### Blocker 3 — `phys-*` namespace registration is not defined

- §5.1 lists `phys-<domain>` as a namespace family with examples `phys-qm`, `phys-bohm`, `phys-gr`, `phys-cst`, `phys-cdt`.
- §5.6 declares the six families `<ns>/framework/1`, `<ns>/assumption/1`, `<ns>/definition/1`, `<ns>/equation/1`, `<ns>/theorem/1`, `<ns>/interpretation/1` "for every registered `phys-*` namespace `<ns>`".
- No section defines a registration mechanism. No section defines whether `phys-foo` is valid for arbitrary `foo` matching the grammar.
- §14.1 says `PackageNamespace = artifact namespace of the owning package` — self-declared, not registered.
- §5.7 refers to `<registered phys-namespace>` and "the owning framework's dependency closure" without specifying what makes a namespace "registered".

**Concrete counterexample:** A coding agent building the second fixture (§16.13) must decide whether to use `phys-qm-alt`, `phys-qm2`, `phys-alt`, or a fresh namespace. Implementation A accepts any `phys-<domain>` matching the grammar. Implementation B maintains a fixed allow-list that does not include the second fixture's choice. Both are consistent with v1.1.

**Failure consequence:** Cross-implementation framework identity is not portable; a framework artifact produced by Implementation B may be rejected by Implementation A's validation.

**Exit condition:** Either declare that any `phys-<domain>` matching the identifier grammar is a valid namespace, or publish a fixed registration list, or specify an explicit registration process.

---

### Blocker 4 — `EXPR-SIMPLIFY-001` "Rational arithmetic" rule is underspecified

- §17.4 lists:
  ```
  Rational arithmetic on exact Rational children
      -> exact reduced Rational result
  ```
  as a rewrite rule without specifying what "on exact Rational children" means structurally.
- §17.4 says "at each node, repeatedly apply the closed rewrite list until that node is locally stable."
- §17.4 does not state whether `EXPR-SIMPLIFY-001` requires canonical input, nor whether all children must be `Rational` for the rule to fire.

**Concrete counterexample:** A coding agent processes `Add([Rational(1/2), Symbol(x), Rational(1/3)])`. Option A applies the rule only if all children are Rational — the rule does not fire; the expression is preserved. Option B applies the rule pairwise to adjacent Rationals — but the children are not necessarily adjacent unless canonicalized first. Option C canonicalizes first (`Add([Symbol(x), Rational(1/2), Rational(1/3)])` sorted) and then the two Rationals are adjacent, so `Add([Rational(5/6), Symbol(x)])` sorted again. Option C changes the expression; A and B do not.

**Failure consequence:** `EXPR-SIMPLIFY-001` output is implementation-dependent for non-Rational-only Add/Mul nodes. Derivation replay diverges.

**Exit condition:** Declare exactly when the rational-arithmetic rule fires: (a) whether `EXPR-SIMPLIFY-001` requires canonical input, (b) whether the rule fires only when all children are Rational, (c) whether a partial reduction is permitted.

---

### Blocker 5 — `EXPR-SUBSTITUTE-001` duplicate-ParameterID handling is undefined

- §17.8: "duplicate targets are invalid."
- §17.8: "matching inside `Input[0]` is by `ParameterID`."
- A target is a `BoundParameterRef { DeclaringArtifactRef, ParameterID }`.

Two legitimate readings of "duplicate targets":

- **Reading A:** Two entries with the same `BoundParameterRef` (both `DeclaringArtifactRef` and `ParameterID` equal).
- **Reading B:** Two entries with the same `ParameterID`, regardless of `DeclaringArtifactRef`.

Under Reading B, two entries with the same `ParameterID` but different `DeclaringArtifactRef` would match the same Symbols in `Input[0]` (since matching is by `ParameterID`), producing an ambiguous replacement. Under Reading A, they are distinct targets and the resulting ambiguity is undefined.

**Concrete counterexample:** A coding agent processes a substitution with targets `(F1, PID_0)` and `(F2, PID_0)` where F1 ≠ F2. Under Reading A, both entries are accepted and the matching rule substitutes every `Symbol(PID_0)` twice — the second replacement overrides the first, or the operation fails, neither of which is specified. Under Reading B, the operation fails before matching.

**Failure consequence:** Substitution semantics are implementation-dependent. A derivation using `EXPR-SUBSTITUTE-001` with overlapping `ParameterID`s cannot be replayed.

**Exit condition:** State which reading is normative, or reject any `ParameterID` collision regardless of `DeclaringArtifactRef`.

---

### Blocker 6 — `DerivationStep` input/output target families are not validated

- §12.2 defines `DerivationStep { Ordinal, ProducerNamespace, RuleID, InputRefs, OutputRef, ConditionRefs, Parameters }`.
- §12.2 says "Each step may refer only to: an imported prerequisite artifact, or an output from an earlier step."
- §17.2's rule contract table gives exact arity and positional input families for each rule.
- §12.4 lists "all step rule IDs are registered" and "all step parameters satisfy their rule schema" as validity checks. It does **not** require that a step's `InputRefs[*]` and `OutputRef` conform to the corresponding rule contract's families.
- §5.7 does not include `DerivationStep.InputRefs`, `DerivationStep.OutputRef`, or `DerivationStep.ConditionRefs` in the closed target-family map.

**Concrete counterexample:** A coding agent constructs a derivation whose Step 0 is `(physmath, EXPR-CANON-001)` with `InputRefs = [artifact of family physmath/function/1]`. This violates §17.2's contract (Input 0 must be `physmath/expression/1`) but passes §12.4's checks. Implementation A rejects it (validating against §17.2). Implementation B accepts it (validating only §12.4). Both are consistent with v1.1 as written.

**Failure consequence:** Derivation validation is not deterministic; a derivation accepted by one implementation may be rejected by another, and the acceptance conditions are not identical across implementations.

**Exit condition:** Add `DerivationStep.InputRefs[*]`, `DerivationStep.OutputRef`, and `DerivationStep.ConditionRefs` to the closed target-family map; specify that each step's input/output families are determined by the selected rule contract; add the family-conformance check to §12.4.

---

## III. Residual issues

Not blockers under the strict test, but a coding agent will encounter them.

- **§6.1 vs §6.2 deferred sorts.** §6.1 lists `Functional`, `FunctionalApplication`, `Operator`, `Limit`, `AsymptoticTruncation`, `DerivationObligation` as part of the closed 16-sort vocabulary. §6.2 declares them in-memory-only. No schema is defined for any of them — not even an in-memory Go type. A coding agent implementing the full PhysMath package must decide whether these are stubs, opaque types, or unimplemented.
- **§5.4 `FunctionID` unused.** Listed in the free-form identifier grammar but no schema uses it.
- **§8.6 vs §8.3 ParameterID consistency.** §8.2 says ordinals are positions in `Function.ParameterIDs`; §8.3 derives `ParameterID` from ordinal. If a coding agent allows `Function([PID_5, PID_3], body)`, the PIDs are inconsistent with ordinal positions. The spec does not require that `Function.ParameterIDs[i] == derive_parameter_id(i)`.
- **§1.4 GR-style protocol packaging.** `manifest.json`'s schema is not specified in v1.1; the spec refers to the GR example. If the GR example is unavailable, the coding agent cannot produce `manifest.json`.
- **§16.6 X and Y.** Stated to be "framework-owned definitions" but not listed as required artifacts (§16.12 lists only `FrameworkArtifact`, `PhysicalAssumption`, `PhysicalDefinition` or `PhysicalInterpretation`, `PhysicalTheorem`). It is unclear whether the reference fixture must publish a `PhysicalDefinition` for `X` or `Y`.
- **§10.9 vs §12.2 schema overlap.** `Transformation` and `DerivationStep` share fields but `DerivationStep` includes `Ordinal`. §10.8's `Transformation` has no `Ordinal`. A coding agent may try to unify these into one type; §10.9 says they are distinct but the reasoning is documentation, not a structural rule.
- **§5.7 `Collection.ElementRefs[*]` in Phase 1 limited to `physmath/expression/1`.** This is consistent with §10.6 but means `Collection` cannot hold functions or relations in Phase 1, which is unusual for a general-purpose mathematical collection.
- **§14.9 rule 3 dependency closure validation.** Checking the transitive closure requires loading all referenced framework artifacts. §14.9 does not state whether this is per-artifact validation or a whole-ecosystem conformance check.
- **§17.4 `Pow(x, 0)` failure outcome class.** §17.4 says the transformation "fails" without stating which `OperationOutcome` (`UNDEFINED`, `TRANSFORMATION_FAILED`, or `OUT_OF_SCOPE`) is returned.
- **§25.25 cross-implementation golden-vector check.** Depends on Blockers 2 and 4 being resolved. Even with those resolved, the check requires a formally specified mapping between the fixture's logical inputs and its canonical bytes; §16.9 lists transformation steps but not a full byte-level construction procedure.

---

## IV. Path to closure

To satisfy §28, the following must be resolved:

1. **Blocker 1.** Declare the exact mechanism for supplying conditions to a rule (`Apply` signature, `ConditionRefs` argument, derivation context, etc.).
2. **Blocker 2.** Pin every `DisplayName` used in the reference fixture, or remove `DisplayName` from canonical `Symbol` bytes.
3. **Blocker 3.** Declare the `phys-*` namespace registration mechanism or an exhaustive list.
4. **Blocker 4.** Specify the exact triggering condition for the rational-arithmetic rewrite rule.
5. **Blocker 5.** Resolve duplicate-target semantics for `EXPR-SUBSTITUTE-001`.
6. **Blocker 6.** Extend §5.7 with `DerivationStep` field target families and add the family-conformance check to §12.4.

Then apply the residuals as one-line clarifications. None of them reopens the macro-architecture or the Phase-1/Phase-2 separation.

---

## V. Final verdict

```text
REVIEW: PM-REP-001 v1.1

Strategic architecture:                          PASS
Phase-1 / Phase-2 separation:                    PASS
Paper-as-research-unit:                          PASS
ARI boundary:                                    PASS
PhysMath purity:                                 PASS
phys-* physical-semantics ownership:             PASS
Many-frameworks principle:                       PASS
Dependency graph:                                PASS
Artifact identity model:                         PASS
Canonical digest framing:                        PASS
Embedded ref encoding:                           PASS
Primitive encoding (String/Bytes/UInt*):         PASS
BigInt encoding:                                 PASS
ConstantID encoding:                             PASS
Closed-enum encoding:                            PASS
Optional / Sequence / Set:                       PASS
Canonical record rule:                           PASS
Artifact-family registry:                        PASS
ArtifactRef target-family map:                   PASS (Blocker 6 partial)
Expression node set and tags:                    PASS
Symbol / Rational / Add / Mul / Neg / Pow / Sqrt: PASS
Call / Relation / BranchSet:                     PASS
ParameterID formula and scope:                   PASS
BoundParameterRef encoding:                      PASS
Function body binding:                           PASS (residual)
Function / FunctionApplication / Map / Tensor / Index / Collection: PASS
OperatorApplication differentiate:               PASS
Transformation schema:                           PASS
Rule registry and arity:                         PASS (Blocker 4)
Rule semantics for canonicalize:                 PASS
Rule semantics for simplify:                     PARTIAL (Blocker 1, Blocker 4)
Rule semantics for substitute:                   PARTIAL (Blocker 5)
Rule semantics for differentiate:                PASS
Rule semantics for select-branch:                PASS
Rule semantics for apply:                        PASS
MathCondition:                                   PASS
Equation / EquationSet / Definition:             PASS
Derivation / DerivationStep:                     PARTIAL (Blocker 6)
Theorem:                                         PASS
FrameworkArtifact / BaseFrameworkRefs:           PASS
Physical assumption / definition / equation:     PASS
Physical theorem / interpretation:               PASS
Framework dependency closure:                    PASS
Theory-local algebra:                            PASS
Equivariance fixture composition:                PASS (Blocker 2)
Equivariance derivation sequence:                PASS
GR-style protocol packaging:                     PASS (residual)
Canonicalization policy:                         PASS
Equality rules:                                  PASS
Serialization / replay / golden vectors:         PARTIAL (Blocker 2)
phys-* conformance:                              PASS
Phase-1 no-go list:                              PASS
Secondary-system boundary:                       PASS
Implementation contract / stop rule:             PASS
Acceptance suite:                                PASS as a set

Blocker 1  Condition provision to rules:         OPEN
Blocker 2  Fixture DisplayName pinning:          OPEN
Blocker 3  phys-* namespace registration:        OPEN
Blocker 4  EXPR-SIMPLIFY rational arithmetic:    OPEN
Blocker 5  EXPR-SUBSTITUTE duplicate targets:    OPEN
Blocker 6  DerivationStep target families:       OPEN

FINAL STATUS: NOT CLOSED. SIX DEMONSTRATED BLOCKERS.
```

---

## VI. Assessment

v1.1 is the most complete document in the entire series. The macro-architecture is closed. Phase-1 and Phase-2 are separated. The rule registry, canonicalization policy, artifact-family registry, and ArtifactRef target-family map are all present. The equivariance fixture is pinned to a single artifact graph and a single derivation sequence, and the GR-style protocol packaging is required.

The six remaining blockers are contract-pinning issues, not architecture redesigns. None reopens the Phase-1/Phase-2 separation, the PhysMath/`phys-*` split, the many-frameworks principle, or the paper-as-research-unit rule. All can be resolved in a single `v1.2` amendment.

Two of them (Blockers 1 and 5) are semantic: they concern what a rule can read and what a substitution can target. The other four are pinning: fixture bytes (Blocker 2), namespace registration (Blocker 3), rewrite-rule triggering (Blocker 4), and derivation-step validation (Blocker 6).

The path to closure is short and mechanical:

1. Resolve the six blockers.
2. Apply the residuals.
3. Extend §28's checklist with the new pins.
4. Freeze v1.1 as amended.

Until then, the coding agent cannot produce byte-identical artifacts across implementations for the reference fixture, and cannot implement `EXPR-SIMPLIFY-001` or `EXPR-SUBSTITUTE-001` deterministically. Both conditions are exactly what §28's handoff rule was written to prevent.