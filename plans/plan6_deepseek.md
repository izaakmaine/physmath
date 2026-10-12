## Adversarial Review — `plan6.md` "v1.3 Final Architecture Closure Plan"

**Context reviewed:** `plan6.md` against `plan5.md`, the prior review cycles, and the stated target of *no architectural blockers and no ambiguity for coding agents*.

**Verdict:**  
**plan6 is the right *method* for reaching closure, but it does not itself close the architecture, and executing it as written would introduce at least three new architectural blockers that would reopen the review cycle.** plan6 is a *closure plan*, not the closed specification. The target — a document a coding agent can implement without inventing architecture — is not met by plan6. It is met by the *v1.3 spec that plan6 instructs us to write*, and only if plan6's own new decisions are first pinned.

Below are the architectural blockers that must be resolved before v1.3 can be frozen, plus the ambiguities that would prevent a coding agent from executing plan6 itself.

---

## I. What plan6 gets right

These are genuine advances and should be preserved:

- §1 correctly freezes the load-bearing architecture before attempting closure.
- §3 pins `ArtifactRef` identity as a four-tuple, not a digest alone, and rules out `artifact_id OR content_digest`.
- §4 separates canonical identity encoding from transport encoding. This is the correct architectural distinction.
- §7–§8 pin the projection subset exactly and add the `Pow` exponent rule as a concrete example of “never silently repair.”
- §9 turns projection classification into a compositional algorithm.
- §11 introduces `TransformResult` with explicit `Conditions` — this closes the previously open “condition vacuum.”
- §12 elevates `t`/`k` from a test to a structural distinction via `DifferentialDomain`.
- §14 removes the “promotion” ambiguity by keeping `ProposedInterpretation` and `PhysicalInterpretation` permanently distinct artifacts.
- §15 removes `CompatibilityRules` rather than defining it — the correct simplification.
- §18 separates `ProposalState` from `ProposalOutcome`; this closes the vocabulary-overloading residue.
- §19 forbids inventing new Phys ingestion APIs — this is the right constraint.
- §22 makes replay authority-local but adds a research-level meta-ledger, which resolves the “cross-module replay illusion.”
- §25–§27 turn named tests into executable acceptance properties, pin the frozen Phys authority, and make supersession machine-checkable.
- §28–§29 introduce a completeness test and a simplification pass — both correct.
- §30–§31 provide a hard stopping rule.

These are the load-bearing moves. Everything below is what plan6 itself still leaves open.

---

## II. New architectural blockers introduced by plan6

### B-13-1. `phys-artifact` is a new architectural authority that has not been reasoned about

§2 creates a new module:

```
github.com/PithomLabs/phys-artifact
```

and asserts it is:

```
not Phys
not PhysMath
not phys-interop
```

and that both PhysMath and Phys will depend on it.

This is a **new project** in the ecosystem. Plan5’s frozen dependency graph had:

```
PhysMath
Phys
phys-*
TOE-Agent
Numerical
```

Adding `phys-artifact` makes six authorities. It is not the same as “PhysMath ⊥ Phys” — both now depend transitively on `phys-artifact`. Whether that is acceptable is a new architectural decision that has not been reviewed.

More concretely:

1. **Does Phys import `phys-artifact`?** §19 says “no new Phys API may be introduced.” If Phys imports `phys-artifact`, that is not an API change, it is a dependency change. If Phys does *not* import it, then Phys’s serialized artifacts do not carry an `ArtifactRef` in the ecosystem format, and Phys cannot participate in the reference DAG that §2 asserts.
2. **Who owns `phys-artifact`?** It is not TOE-Agent, not PhysMath, not phys-*, not Phys, not Numerical. The no-invention matrix in §28 requires every architectural concern to have an owner. `phys-artifact` has none.
3. **What versioning discipline does it follow?** If `phys-artifact` changes, both PhysMath and Phys break. This is exactly the coupling plan5 was avoiding.
4. **Does the frozen Phys spec *permit* Phys to depend on a new module?** Plan6 §26 requires “Phys repository/module, frozen specification identifier, frozen specification version, required repository revision/commit.” If the frozen spec does not permit new dependencies, `phys-artifact` cannot be introduced without a Growth Gate — which §19 explicitly prohibits shortcutting.

**Exit condition:** Either (a) declare that `phys-artifact` is not a new module but a *schema family* defined per-project with a cross-project equivalence rule, or (b) explicitly amend the Phys frozen dependency graph to allow a Phys → `phys-artifact` import, with the Frozen Spec updated by a Growth Gate decision, or (c) treat Phys as an artifact type whose `ArtifactRef`-equivalent is defined inside Phys itself and cross-referenced via a translation layer. Plan6 chooses none of these.

---

### B-13-2. §2 and §5 contradict each other on what `ArtifactRef` replaces

§2 says:

> Every ecosystem-wide serialized reference uses the same `ArtifactRef` definition.

§5 says:

> There is NO mandatory global registry … NO implicit network lookup.

If there is no registry and no lookup, then an `ArtifactRef` is only resolvable by whoever already knows how to resolve it. That means `ArtifactRef` is a *naming* convention, not a *reference* mechanism. But §2 asserts it is the mechanism by which “ecosystem-wide serialized references” work.

The tension: an `ArtifactRef` that cannot be resolved without an out-of-band registry is a pointer with no address space. Cross-project serialized references require a *resolution contract*, which plan6 explicitly forbids.

**Exit condition:** Declare that `ArtifactRef` is a *name* (like a URN), not a *resolver* (like a URL). A resolver is a storage-environment concern, not part of the type. This is fine, but it must be stated, and §2 must not imply `ArtifactRef` alone enables cross-project reference resolution.

---

### B-13-3. §12’s `DifferentialDomain` enum introduces physical semantics into PhysMath

§12 proposes:

```
DifferentialDomain =
    COORDINATE
    TIME
    RG_SCALE
```

`TIME` is a physical concept, and `RG_SCALE` is a physics-derived concept (renormalization). Encoding them in PhysMath’s IR puts physical role names in the mathematical substrate, which plan5 ADR-006 and plan6 §1 freeze against.

This is exactly the kind of “notation becomes semantics” failure that earlier reviews flagged. The distinction between “a symbol named `t`” and “a symbol tagged `TIME`” is a naming discipline, not a semantic one.

A cleaner model: `DifferentialDomain` is an *opaque parameter identity*, and *physical frameworks* (or Project A) supply the mapping from a parameter to a physical role. This keeps PhysMath pure and lets physics name the roles.

**Exit condition:** Either remove the enum and make the tag an opaque identity, or explicitly amend the constitutional rule (“PhysMath is pure mathematics”) to allow *physical-role-typed* mathematical parameters. Plan6 does neither.

---

### B-13-4. §6 hands off IR pinning to v1.3 without verifying it can be done

§6 says v1.3 “must contain a normative table” for the PhysMath IR, then says the “existing v1.2 sort inventory should be preserved rather than redesigned.”

Plan6 does not verify that the v1.2 inventory *can* be pinned consistently with:

- §8 (closed AST),
- §9 (extension protocol),
- §6 (projection subset = 7 sorts),
- §12 (differential domain tag),
- §11 (conditions as separate artifacts).

Three concrete tensions:

1. If the core sort set is closed (plan5 §9), then how do `phys-*` “theory-local algebras” (§13 of plan6) *not* become new sorts? §13 says they don’t, but the mechanism is not stated.
2. If `Pow` is projectable only with rational exponent (§8), is the exponent itself a `PhysMath` sort? Or a raw rational? Or a PhysMath sort that is separately projectable?
3. If `TransformResult` (§11) is a new PhysMath type, is it a sort in the same IR, or a meta-level type? Plan6 does not say.

**Exit condition:** plan6 must either (a) declare the v1.2 inventory provisionally closed and require v1.3 to demonstrate consistency, or (b) leave explicit room for v1.3 to revise the inventory. It does neither.

---

### B-13-5. §17’s formalization protocol schemas are still field lists

§17 gives:

```
FormalizationProposal:
    ProposalRef
    ProducerNamespace
    ProposalVersion
    SourceArtifactRefs
    RequestedTarget
    ProjectionPolicy
    RequiredFrameworkRefs
```

None of these fields has a type. `ProjectionPolicy` in particular is undefined. `RequestedTarget` could be a namespace, a module, a `phys-*` package, an API name, or a formalization contract — the document does not say. `RequiredFrameworkRefs` could be opaque IDs, `phys-*` package names, or `ArtifactRef`s.

This is exactly the pattern this review cycle keeps flagging: field names without types.

**Exit condition:** concrete type for each field, canonical form, and version identifier for the schema.

---

### B-13-6. §20 states a provenance law without deriving it from the frozen Phys spec

§20 says:

> Any session-recorded correspondence operating on hypothesis-grade operands remains hypothesis-grade under the frozen Phys contamination law.

This is a strong claim. It is stated as fact. But it requires the frozen Phys spec to actually have this law. Plan6 does not cite the frozen spec.

If the frozen Phys spec does not have this law, plan6 is inventing Phys behavior — which §19 explicitly forbids.

**Exit condition:** cite the frozen Phys spec section for this law, or amend the frozen spec via Growth Gate. Do not assert it.

---

### B-13-7. §23 assigns the Numerical boundary to `phys-*` without saying who owns the contract

§23 says:

```
PhysMath
   ↓
phys-* domain-specific lowering
   ↓
ComputationalContract / execution representation
   ↓
Numerical
```

Who owns the `ComputationalContract` schema? `phys-*`? A new module? Phys? Numerical itself?

If `phys-*` owns it, then different `phys-*` packages produce different contract schemas, and the Numerical Layer needs to understand all of them. That violates the “Physical semantics are owned by `phys-*`” principle in a way that makes composition impossible.

If Numerical owns it, then Numerical defines what `phys-*` must produce, which violates “Numerical does not infer missing physical semantics.”

**Exit condition:** declare the owner and the schema.

---

### B-13-8. §30–§31 stopping rule is not arbitrated

§31 says:

> DeepSeek: 0 BLOCKER, Gemini: 0 BLOCKER, Qwen: 0 BLOCKER, Z: 0 BLOCKER

But there is no arbitration mechanism when 3 reviewers say “0 BLOCKER” and 1 says “1 BLOCKER.” The stopping rule depends on the reviewers being exhaustive, which they are not guaranteed to be.

Worse, it allows a single reviewer to indefinitely block closure by claiming a blocker that the other three do not recognize. Without an arbitration rule, the review process has no exit.

**Exit condition:** define the arbitration rule: e.g., majority vote, a designated tiebreaker, or a rule that requires the blocking reviewer to provide a concrete counterexample (“given X, When Y, Then Z fails”) as §30 already requires.

---

## III. Ambiguities that would prevent a coding agent from executing plan6

### A-1. plan6 is not the artifact it says to produce

Plan6 says: “the next artifact should be a final architecture-closure amendment.” It also says: “v1.3 architecture must contain a normative table.” It is therefore unclear whether plan6 *is* v1.3 or is a plan to produce v1.3.

If plan6 is a plan, a coding agent reading it cannot proceed without the v1.3 spec. If plan6 is v1.3, it is incomplete. Either way, plan6 is not what the coding agent needs.

**Exit condition:** clarify that plan6 is a plan and that v1.3 is the deliverable.

### A-2. §29 simplification pass is not bounded

§29 says: “Do not accept complexity merely to close a reviewer comment.” It then lists ten forbidden abstractions. But it does not say who decides whether a proposed simplification is acceptable, or how it interacts with the frozen decisions in §1.

**Exit condition:** bound the simplification pass: which decisions in §1 are frozen and cannot be simplified away, and which are open.

### A-3. §28 no-invention matrix has no completeness criterion

§28 says the matrix is the “central handoff mechanism” and requires it to have a row for each architectural concern. But there is no rule for *which concerns* must be in the matrix, nor for how to detect a missing row.

**Exit condition:** list the exact concerns the matrix must cover, and add a meta-check: “For every artifact class, IR sort, authority, and dependency in §1, the matrix has a row.”

### A-4. §2 and §19 conflict on Phys modification

§2 requires Phys to depend on `phys-artifact`. §19 says “no new Phys API may be introduced to support the architecture.”

Adding a dependency is not an API change, but it is a change to Phys’s go.mod. If the frozen Phys module is truly frozen, then §2 cannot be executed without breaking the frozen state. If §19 permits dependency changes, then Phys is not actually frozen.

**Exit condition:** clarify whether “frozen Phys” includes its dependency graph.

---

## IV. What plan6 should add before v1.3 is written

### Add to plan6 §2:

- An owner for `phys-artifact`.
- A statement of whether Phys imports it.
- If not, how Phys artifacts carry ecosystem references.
- Whether adding this module requires a Growth Gate decision.

### Add to plan6 §3:

- A statement that `ArtifactRef` is a *name*, not a resolver.
- A statement of how names get bound to storage in practice (e.g., per-environment resolvers).

### Add to plan6 §6:

- A commitment that v1.3 will contain a *closed* IR sort table and that v1.3’s acceptance includes consistency with §8–§13 of plan6.

### Add to plan6 §12:

- Either an opaque `DifferentialDomain` identifier or an explicit amendment to the “PhysMath is mathematics only” rule.

### Add to plan6 §17:

- Concrete field types for `FormalizationProposal` and `FormalizationResponse`.
- An owner for the schema.

### Add to plan6 §20:

- A citation to the frozen Phys spec for the contamination law, or an explicit Growth Gate request.

### Add to plan6 §23:

- An owner for `ComputationalContract` and a statement of its schema.

### Add to plan6 §26:

- A statement of what “frozen” means: module + dependency graph + spec, or just spec?

### Add to plan6 §31:

- An arbitration rule for disagreements among the four reviewers.

### Add to plan6 §28:

- A completeness criterion for the matrix: what set of concerns must all have rows.

---

## V. Final verdict

```text
REVIEW: plan6.md — v1.3 Closure Plan

Architectural model:                    PASS
Load-bearing freeze list:               PASS
ArtifactRef identity pinned:            PASS (identity) / OPEN (ownership)
Canonical vs transport encoding split:  PASS
Projection subset + Pow rule:           PASS
Compositional projection algorithm:    PASS
TransformResult with Conditions:        PASS
DifferentialDomain:                     OPEN (B-13-3)
Theory-local algebras:                  PASS
Promotion ambiguity removed:            PASS
PhysicalContract simplification:        PASS
Formalization protocol schemas:         OPEN (B-13-5)
Proposal state vs outcome:              PASS
Phys ingestion API constraint:          PASS
Provenance law:                         OPEN (B-13-6)
MathTrace unification:                  PASS
Research meta-ledger:                   PASS
Numerical boundary:                     OPEN (B-13-7)
Status state machines:                  OPEN (plan step)
Executable acceptance properties:       PASS (as a requirement)
Frozen Phys authority pinning:          PASS (as a requirement)
Supersession machine-check:             PASS
No-invention matrix:                    OPEN (completeness criterion)
Simplification pass:                    OPEN (bounds)

New blockers introduced by plan6:
  phys-artifact ownership:              OPEN (B-13-1)
  ArtifactRef is a name not a resolver: OPEN (B-13-2)
  DifferentialDomain purity:            OPEN (B-13-3)
  IR pinning handoff:                   OPEN (B-13-4)
  Formalization schemas:                OPEN (B-13-5)
  Provenance law derivation:            OPEN (B-13-6)
  Numerical boundary ownership:         OPEN (B-13-7)
  Reviewer arbitration:                 OPEN (B-13-8)

FINAL STATUS: PLAN VIABLE, BUT PLAN6 MUST FIRST RESOLVE B-13-1..B-13-8
              BEFORE THE V1.3 SPEC CAN BE WRITTEN AND FROZEN
```

---

## VI. What plan6 is, accurately stated

Plan6 is not the closed architecture. It is a good plan for producing the closed architecture. It correctly identifies the deep structural issues — identity, projection, conditions, `t`/`k`, promotion, schemas, protocol, replay, numerical boundary, and reviewer discipline — and it correctly proposes the shape of each resolution.

But by introducing `phys-artifact`, `DifferentialDomain`, and a stopping rule without arbitration, plan6 adds three new architectural questions that will reopen the cycle if not resolved first.

The path to closure is:

1. **Resolve B-13-1 through B-13-8 inside plan6 itself.** These are plan-level decisions, not spec-level decisions. They must be pinned before plan6 is used to generate v1.3.
2. **Then write v1.3** using plan6’s §28 matrix as the completeness test.
3. **Then run the blocker-only audit** with the arbitration rule from B-13-8.
4. **Then freeze.**

Plan6 is close. It is not closed. The difference is exactly the set of findings above: the new module, the enum, the arbitration rule, and the schemas that plan6 leaves for v1.3 to define without pinning their owners.