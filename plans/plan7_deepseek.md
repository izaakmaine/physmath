## Adversarial Review — `plan7.md` "v1.3 Consolidated Closure Plan"

**Context reviewed:** `plan7.md` against `plan5.md`, `plan6.md`, the prior adversarial reviews, and the stated target of *no architectural blockers and no ambiguity for coding agents about architectural decisions*.

**Verdict:**  
**plan7 is the right *method* and does not meet the stated target itself.** It correctly identifies that the next artifact must be the actual v1.3 specification, not another closure plan. But plan7 is not v1.3, and its own instructions to the v1.3 author contain four architectural blockers and six residual ambiguities. If those are not resolved *inside plan7* first, the v1.3 author will have to invent architecture — which is the exact failure mode that has survived plan2 through plan6.

This is the seventh consecutive document in the cycle. The cycle continues because each document leaves a small set of decisions to the next. The breaking move is not another closure plan; it is naming the v1.3 author with authority to make the deferred decisions, and stating that no further plan is permitted after v1.3.

---

## I. What plan7 gets right

Preserve these. They are load-bearing and correct.

- §2 corrects the biggest error in plan6: **Phys remains byte-frozen and imports nothing new.** This closes B-13-1 by inverting the dependency direction.
- §3 redefines `phys-artifact` as a mechanical library, not an authority. Correct conceptual move.
- §4 makes `ArtifactRef` a name, not a resolver. This is the sharpest correction in plan7.
- §5–§8 pin identity, framing, embedded-ref encoding, and canonical-vs-transport separation. All correct and mechanically checkable.
- §11–§12 remove `DifferentialDomain` from PhysMath and introduce parameter identity as the t/k distinguisher. Correct conceptual fix.
- §14 makes `PARTIALLY_REPRESENTABLE` vocabulary-present but MVP-unreachable. Good.
- §17 pins associative projection. Good, closes a real ambiguity.
- §19 permanently separates `ProposedInterpretation` from `PhysicalInterpretation`. Good.
- §22 separates `ProposalState` from `ProposalOutcome` and specifies lineage. Good.
- §24 correctly *inherits* the provenance law from Phys instead of inventing it. Correct.
- §26 gives `ComputationalContract` a single owner. Correct direction.
- §28 separates exact symbolic values from numerical `Float64Bits` canonicalization. Correct.
- §29–§31 pin the MVP boundary, Growth Gate phase, and `phys-interop` deferral. Correct.
- §32–§34 give artifact templates, a No-Invention Matrix, and evidence-based arbitration. Good mechanisms.
- §35–§37 make v1.3 self-contained and give it a fixed structure. Correct.

The *direction* of plan7 is right. The *content* is still incomplete.

---

## II. Architectural blockers in plan7

These are blockers because a coding agent tasked with producing v1.3 from plan7 would have to invent architectural decisions. That violates the target.

### Blocker A — Phys artifact reference construction is not defined

§2 says:

> Consumers calculate references to frozen Phys artifacts from the frozen Phys canonical representation.

But §5 pins the universal `ArtifactRef` as `(Namespace, ArtifactType, SchemaVersion, Digest)`, and §6 pins the digest as

```
SHA-256(
  Len32(Namespace) || Namespace
  || Len32(SchemaVersion) || SchemaVersion
  || Len32(ArtifactType) || ArtifactType
  || CanonicalPayload
)
```

To construct an `ArtifactRef` for a Phys artifact, a consumer must know:

- which `Namespace` value Phys artifacts live in (`phys.*`? `phys.object.*`? something else?);
- which `ArtifactType` values Phys exposes (`Object`, `Expr`, `Session`, `Ledger`, `ResearchCandidate`, `ManifestItem`, …);
- which `SchemaVersion` a given Phys artifact carries (the Phys project uses `mrc-v0.4` as a spec version — is that also the artifact schema version?);
- what `CanonicalPayload` means for each Phys artifact type (the README says Phys has deterministic canonical JSON with SHA-256, but that hash is computed over a different byte stream than §6's framing).

None of this is in plan7 or in any prior document. A coding agent producing v1.3 must invent all four. That is a new architectural decision.

**Exit condition:** plan7 must specify — or explicitly delegate with the delegation recorded as a plan7 decision — (a) the Phys namespace, (b) the Phys artifact-type taxonomy, (c) the Phys schema-version convention, and (d) the adapter that maps Phys canonical bytes to §6 framing.

---

### Blocker B — "Parameter identity" is undefined

§11–§12 say:

> A differential operator targets a parameter identity.
> The identity is immutable.
> Renaming a display label does not change parameter identity.

No type, no scheme, no scope rule. A coding agent producing v1.3 cannot implement the `t`/`k` invariance law without knowing:

- Is a parameter identity a generated UUID, a scoped name, a content hash of a declaration, an opaque token minted by PhysMath, or something else?
- How are two textual occurrences of `t` bound to the same parameter identity? Through declaration, scope, or environment?
- How does alpha-renaming interact with parameter identity (given §10 treats `Symbol` as an `Expr` node kind and §24 of plan6 pinned scope-based identity generally)?
- If a parameter identity is content-addressed, what is its canonical payload?

The t/k distinction is a hard requirement across plan5 §38, plan6 §12, and plan7 §11–§12. It cannot be enforced without a concrete parameter-identity type.

**Exit condition:** declare the representation of parameter identity in v1.3, with one of the above schemes chosen.

---

### Blocker C — `phys-*` → Numerical dependency edge is unstated

§26 says:

> Numerical owns the `ComputationalContract` schema because it is the execution boundary contract, while `phys-*` owns production of valid instances from physical semantics.

But if `phys-*` produces instances of a Numerical-owned schema, then `phys-*` depends on Numerical's schema definitions. That is a new dependency edge `phys-* → Numerical` that does not appear in any prior dependency graph (§2 of plan7, §4 of plan5, §6 of plan2).

Three legitimate resolutions exist:

1. Accept `phys-* → Numerical` as a real edge and amend the dependency graph.
2. Lift the `ComputationalContract` schema into a neutral shared module (like `phys-artifact`) that both `phys-*` and Numerical import.
3. Split Numerical into "schema" and "implementation" projects so only the schema is depended on.

Plan7 chooses none of these. The No-Invention Matrix in §33 lists "Numerical contract" as a row, but the row cannot be filled in v1.3 without a decision that plan7 does not make.

**Exit condition:** declare the dependency-graph consequence of Numerical-owned `ComputationalContract`.

---

### Blocker D — The v1.3 author's authority is not established

§9 says v1.3 must contain the normative PhysMath sort table. §11 says v1.3 must remove `DifferentialDomain`. §14 says v1.3 must pin `STRICT_MIRROR_ONLY` for `ProjectionPolicy`. §19 says v1.3 must separate `ProposedInterpretation` from `PhysicalInterpretation`.

But plan7 does not say:

- who authors v1.3;
- whether that author has authority to *decide* the deferred items (the sort table cells, the parameter-identity representation, the Phys adapter, the `phys-*` → Numerical edge);
- what the review process is for v1.3 before freeze;
- who has final sign-off;
- what happens if the four reviewers disagree on v1.3.

Given that seven documents in this series have left decisions to their successor, an unnamed v1.3 author with unspecified authority will almost certainly leave new decisions to a hypothetical v1.4. That is the structural reason the cycle persists.

**Exit condition:** plan7 must name the v1.3 author, define their decision authority, and state that no further closure plan is permitted after v1.3.

---

## III. Residual ambiguities that would leave the v1.3 author with a choice

These are smaller than the blockers above but still leave open decisions.

### A-1. §5 "I recommend `string` for `SchemaVersion`" is not a normative statement

`I recommend` is prose. For v1.3 to be self-contained, plan7 must state `SchemaVersion MUST be a string` (or `uint32`, if that's the decision). The recommendation format leaves the v1.3 author with a choice.

### A-2. §21 "Pin MVP: `ProjectionPolicy = STRICT_MIRROR_ONLY`" does not say where the pin lives

Is this pinned in plan7, in v1.3, or in the frozen Phys spec? If in v1.3, plan7 must say so explicitly. As written it reads like a decision but the location is unspecified.

### A-3. §22 `RESOLVED` after `REQUEST_REVISION` is unusual

`RESOLVED` normally connotes success. Making it also the terminal state for a rejected or revision-requested proposal will confuse readers. If this is intended, plan7 should say so explicitly, with a short rationale; otherwise v1.3 will inherit the confusion.

### A-4. §9 "the existing v1.2 inventory" is not identified

Which document is v1.2? plan5.md is versioned 1.2 in its header, but plan5's §8 does not claim to be a normative inventory; it says "the exact final core sort set is implementation-pinned before coding." The v1.3 author cannot follow §9 without a specific source. plan7 must cite the document and section.

### A-5. §23's "provided this is confirmed against the frozen specification" defers confirmation

`hypothesis.NewCandidateConcept` is asserted as the raw hypothesis-grade mint path, subject to confirmation. If it is not present in the frozen Phys spec, the whole formalization protocol has no mint path. plan7 must either confirm it in place or state what happens if confirmation fails.

### A-6. §24 cites "the frozen Phys section" without naming it

The provenance law is inherited. The citation must be given so the v1.3 author can verify it. plan7 currently says the citation will be added; that is a decision plan7 leaves to v1.3.

---

## IV. A structural observation

The recurring pattern across plan2–plan7 is:

1. A prior document (n) is reviewed.
2. A new document (n+1) is written that consolidates the reviews and defers a small set of decisions to document (n+2).
3. Those deferred decisions become the blockers identified in review of (n+1).

plan7 identifies that pattern and states v1.3 should be the *last* document before implementation. That is correct. But plan7 does not change the underlying mechanism: it still defers decisions. It defers them to v1.3, and v1.3 will in turn defer them if the v1.3 author lacks authority to decide.

The way out is not another closure plan. It is:

- Name the v1.3 author.
- Give that author decision authority for the deferred items.
- State that v1.3 is the last architecture document.
- Freeze or reject v1.3 on the outcome of the four-agent audit, with the arbitration rule from §34.

If plan7 does these four things, the cycle terminates at v1.3. If it does not, the cycle continues.

---

## V. Final verdict

```text
REVIEW: plan7.md v1.3 Closure Plan

Architectural direction:                 PASS
Phys freeze corrected:                   PASS
phys-artifact as mechanical library:     PASS
ArtifactRef as name not resolver:        PASS
Identity + framing + embedded refs:      PASS
DifferentialDomain removed:              PASS
t/k via parameter identity:              PARTIAL (Blocker B)
Projection rules + MVP zero-loss:        PASS
Associative projection:                  PASS
Condition semantics:                     PASS
Interpretation separation:               PASS
PhysicalContract narrow schema:          PASS
Formalization typed messages:            PASS (as a task for v1.3)
Formalization state machine:             PASS
Phys ingestion API constraint:           PASS
Provenance law inheritance:              PARTIAL (A-6)
MathTrace unification:                   PASS
Research meta-ledger:                    PASS
ComputationalContract ownership:         PARTIAL (Blocker C)
In-process vs persisted handoff:         PASS
Floating-point canonicalization:         PASS
MVP boundary:                            PASS
Growth Gate post-MVP:                    PASS
phys-interop deferred:                   PASS
Artifact contract template:              PASS
No-Invention Matrix:                     PASS as a mechanism
Reviewer arbitration:                    PASS
Self-contained v1.3:                     PASS as a requirement

Blockers introduced by plan7 itself:
  Phys artifact ref construction:        OPEN (Blocker A)
  Parameter identity type:               OPEN (Blocker B)
  phys-* -> Numerical edge:              OPEN (Blocker C)
  v1.3 author authority:                 OPEN (Blocker D)

Residual ambiguities:
  §5 "I recommend" vs normative:         OPEN (A-1)
  §21 pin location:                      OPEN (A-2)
  §22 RESOLVED semantic:                 OPEN (A-3)
  §9 v1.2 inventory citation:            OPEN (A-4)
  §23 Phys API confirmation:             OPEN (A-5)
  §24 provenance law citation:           OPEN (A-6)

FINAL STATUS: PLAN7 MUST RESOLVE BLOCKERS A-D AND AMBIGUITIES A-1..A-6
              BEFORE V1.3 CAN BE AUTHORED. PLAN7 DOES NOT MEET THE
              STATED TARGET OF "NO ARCHITECTURAL BLOCKERS" ITSELF.
```

---

## VI. The correct next move

Do not write v1.3 yet. Do not write another closure plan either. The move that actually closes the cycle is to append a short **plan7.1 addendum** containing:

1. A decision on Blocker A: the Phys namespace, artifact-type taxonomy, schema-version convention, and byte-framing adapter.
2. A decision on Blocker B: the parameter-identity representation.
3. A decision on Blocker C: which of the three `phys-* → Numerical` resolutions is chosen.
4. A decision on Blocker D: who authors v1.3, with what authority, subject to what review, and with what freeze rule.
5. Six short clarifications resolving A-1 through A-6.

Once plan7.1 exists, v1.3 can be authored without the v1.3 author inventing architecture. Then one blocker-only audit, arbitration per §34, and freeze.

That is the shortest path from here to an implementation plan. It requires resolving four decisions in plan7 itself, not writing a v1.4.