## Adversarial Review — `plan9.2.md`

**Context reviewed:** `plan9.2.md` against `plan9.1.md`, the attachment of `PM-REP-001 v1.2/v1.4`, and the target: no architectural blockers, no ambiguity for the coding agent about architectural decisions.

**Verdict:**  
**NOT CLOSED. THREE DEMONSTRATED BLOCKERS REMAIN, PLUS FOUR RESIDUALS.**

The plan is the strongest document yet. It adopts decisions D1–D17, resolves R1–R6 (rejecting three review-introduced inventions), specifies deferred boundaries F1–F7, pins the topology, and adds a 20-gate acceptance matrix and a quality-gate checklist. Its claim “zero open architectural blockers, zero implementation-critical ambiguities” is *mostly* accurate.

But three items fail the standard the plan itself sets. Each is a decision the plan declares but which conflicts with the spec, or a mechanism the coding agent must invent.

---

## I. Blockers resolved since the previous round

For the record, these are now pinned:

- B1/B2 → adopted (D1, D2).
- GR fixture refactor vs preserve → resolved (D3, D8): fresh `phys-lib/gr/equivariance/` is the canonical-pattern fixture; `phys-gr/solution/` remains archive.
- Phase 4 sequencing → resolved (D6): scaffolding + record template only.
- `phys-lib/gr` Phase-1 scope → pinned (D7, F1): framework + GR-local algebra + equivariance; Schwarzschild re-expression deferred.
- Chain-rule invention (Gemini) → rejected (R1).
- Colon-joined rule IDs → rejected (R3).
- Subset-conformance reading of fixture2 → rejected (R4): full minimal framework.
- `BaseFrameworkRefs` as import authority → rejected (D13): artifact/data-only.
- Vocabulary tests as substring traps → replaced with semantic/mechanical split (D16).

---

## II. Demonstrated blockers

### Blocker 1 — D15 contradicts spec §14.2

- `plan9.2.md` D15: “Category vocabulary for GR `PhysicalAssumption` is the **closed enum** `{convention, regularity, model}`. Any other value is rejected; **extension requires a spec change**.”
- Spec §14.2: “`Category` is a framework-owned semantic label and **intentionally remains an open string** rather than a global ontology key.”
- Spec §14.2 further: “The architecture imposes **no universal physical assumption vocabulary** beyond the package contract.”

D15 turns a framework-owned open string into a Phase-1-architecture-closed enum. The sentence “extension requires a spec change” is exactly what §14.2 refuses: it makes the vocabulary architecture-owned.

**Concrete counterexample:** A future `phys-qm` package defines `PhysicalAssumption { Category: "quantum-postulate" }`. This is spec-compliant per §14.2. Under D15 as written, it is rejected by any validator that implements D15 universally. Two implementations legitimately disagree — one adopts D15 as universal, one treats D15 as a GR-local convention.

**Failure consequence:** If D15 is a GR-local convention, the plan should say so and Phase-1 validation for fixture2 cannot enforce it. If D15 is universal, the plan must amend the spec — which is a spec change, not an implementation decision.

**Exit condition:** Rewrite D15 to read: *“For Phase 1, `phys-lib/gr` uses exactly the Category vocabulary `{convention, regularity, model}`. This is a GR-package decision, not a cross-framework vocabulary. No framework validator may reject another framework's Category values on the basis of this list.”* Or amend spec §14.2 to declare the three values closed across all `phys-*` frameworks.

---

### Blocker 2 — “Ported golden vectors” mechanism unspecified

- `plan9.2.md` §7: “every row is a deliberate re-expression against the locked architecture, **validated by ported golden vectors** (copied data, D4).”
- `plan9.2.md` §8 Phase 5 Exit: “GR regression via **ported vectors**.”
- `plan9.2.md` §9: “Copy (never reinterpret): 15-key manifest schema (values change: ...)”
- `plan9.2.md` D4: “Legacy validation uses **copied golden vectors**, never imports.”
- Spec §20.5: “Every durable family and every executable rule contract **MUST** have at least one positive canonical vector and one negative vector in the reference implementation. … The equivariance fixture additionally pins the exact step sequence and expected `ArtifactRef`s.”

The plan does not say whether a “ported” vector is:

- **(a)** a byte-for-byte copy of the corresponding `phys-gr` test vector, or
- **(b)** a semantically equivalent vector whose bytes and `ArtifactRef` are re-derived from the fresh `phys-lib/gr` implementation.

This distinction is load-bearing: spec §7.2 mandates that a durable `physmath/expression/1` artifact’s canonical payload is `UInt8(root-node-tag) || node-specific canonical payload`. The existing `phys-gr` `symbolic/expr.go` serializes to *canonical JSON*, not to binary tags. So a byte-for-byte copy of a `phys-gr` vector will never match a `phys-lib/gr` vector. If the plan intends byte-for-byte copies, all copied vectors will fail. If it intends semantic copies, the copied vectors do not exist yet and must be authored.

**Concrete counterexample:** `phys-gr` has a golden value for the metric tensor's canonical serialization. Under (a), the plan copies bytes and the new test asserts equality — guaranteed failure. Under (b), the plan must define what “semantic port” means for a vector whose value is a byte sequence: does the vector’s *semantic content* (metric components as expressions) get re-serialized through `phys-math`’s codec, or through `phys-lib/gr`’s framework codec?

**Failure consequence:** Phase 5’s exit criterion (“GR regression via ported vectors”) is not checkable. Different implementers will produce different vector files, defeating cross-implementation determinism (spec §25.34).

**Exit condition:** Declare in §9 (a) the exact shape of a “ported vector”, (b) the source’s role (informational reference vs regression oracle), (c) whether byte-equality is required or only semantic equality of the represented expression. State which `phys-gr` test files provide the semantic source of truth for GR regression after the port.

---

### Blocker 3 — Classification record mechanism unspecified

- `plan9.2.md` §6 defines the classification protocol and the promotion record fields.
- `plan9.2.md` §8 Phase 4: “record template committed”.
- `plan9.2.md` §12 gate [18]: “classification records present for every §7 row”.
- `plan9.2.md` §13 G2: “§8 records complete”.
- `plan9.2.md` §15: “All core/specific decisions justified by record, never convenience.”

The plan does not say:

- What the record *is* (a Markdown file per abstraction? a JSON manifest? a Go test that asserts something? a comment on the PR?).
- Where it lives in the repo (which directory, which naming convention).
- How gate [18] mechanically verifies presence.
- Whether the record is required before *any* code lands in `phys-lib/core`, or whether it can be retrofitted after the fact.

**Concrete counterexample:** Phase 5 proposes moving `phys-gr/symbolic/normalize.go`’s commutative normalization into `phys-lib/gr` and — per §7 — *not* into `core`. A coding agent produces a `SPLIT` classification: is this a record? If the record is required for every §7 row, that’s ~13 records. Where do they live? If they’re Markdown, gate [18] cannot mechanically verify them. If they’re a JSON file per row, the schema is undefined.

**Failure consequence:** The plan says “evidence-based promotion, no convenience” but provides no mechanism for evidence to be committed, discovered, or checked. A coding agent either invents a record format (violating the plan’s own “no decision left to the coding agent” claim) or produces nothing (violating gate [18]).

**Exit condition:** Specify the record format, location, and verification. Minimum viable form:

```text
phys-lib/classifications/<slug>.md
    Committed with the PR that moves the abstraction.
    Machine-readable preamble delimited by ---.
    CI check: for every file added under core/, at least one classification exists.
```

Or specify an equivalent mechanism. State explicitly whether a record is a *PR-merge* gate or a *Phase-10 audit* gate.

---

## III. Residual issues

### R-1. Empty `core/` package mechanism unspecified

§4 shows `phys-lib/core/` as a directory. §8 Phase 4 says “package core with zero domain abstractions”. §13 G2 says “core-empty assertion test green”. §15 says “empty core is a PASS”.

But `plan9.2.md` does not say whether `core/` is:

- an empty directory (which will fail `go build ./...`),
- a Go package with only a `doc.go` and no exported symbols,
- a placeholder package with a single no-op exported type (which would then have a public API),
- a directory reserved by convention with no Go file at all.

`go test ./...` on a module that has a directory containing no Go files emits `matched no packages`. A CI gate that runs `go test ./...` across the workspace will succeed because it simply skips the empty directory — but then nothing enforces “core is empty”.

**Suggested clarification:** `phys-lib/core/` contains a single `doc.go` with a package doc comment and no exported identifiers. A test in `phys-lib/internal/conformance` asserts `go doc ./core` produces exactly the documented comment and no exported symbols.

### R-2. SUBSTITUTE “distinct ParameterIDs” phrasing

`plan9.2.md` §8 Phase 3 says: “SUBSTITUTE sorted inputs/entries + **distinct ParameterIDs**”. Spec §17.8 says: “**no two substitution entries may share the same `ParameterID`, regardless of `DeclaringArtifactRef`**”.

The plan’s phrasing does not state “regardless of `DeclaringArtifactRef`”. A coding agent could read the plan as “distinct `(DeclaringArtifactRef, ParameterID)` pairs” and accept two entries with the same `ParameterID` but different declaring artifacts — which spec §17.8 explicitly forbids.

**Suggested clarification:** Mirror spec §17.8’s wording verbatim.

### R-3. `OperationInvocation` immutability

`plan9.2.md` §8 Phase 3 says “`OperationInvocation/TransformResult` (immutable; conditions = supplied set, never discharged)”.

Spec §17.1 says `TransformResult` is “in-memory only and immutable after construction”. Spec §17.1 does not state that `OperationInvocation` is immutable. Adding immutability to `OperationInvocation` is a plan-level choice, not a spec requirement. This is fine but should be stated as a plan choice to avoid confusion when the spec is reread.

### R-4. Version-label hygiene is documented but not resolved

`plan9.2.md` §1 notes the spec’s header (`Version: 1.2`) contradicts its body (references to `v1.4`). The plan’s operative rule — “the file’s normative content governs as written; **PM-REP-001 v1.2** names the specification throughout this plan” — is reasonable, but §16’s handoff checklist tells the coding agent to “Read **PM-REP-001 v1.2** (operative file)”. A coding agent searching for a file named `..._v1.2.md` will find one; a coding agent reading the file’s body will find `v1.4`. This mismatch is not a blocker, but it will produce two traceability chains: one using “v1.2” (the filename) and one using “v1.4” (the body). The plan should either cite a canonical identifier (e.g. `Document ID: PM-REP-001, file PHYSMATH_..._v1.2.md, body version 1.4`) or amend the file’s header.

---

## IV. Path to closure

To satisfy the plan’s own “ARCHITECTURALLY CLOSED FOR IMPLEMENTATION PLANNING” status:

1. **Blocker 1.** Rewrite D15 to scope the Category enum to `phys-lib/gr` only, or amend spec §14.2 to close the vocabulary across all frameworks.
2. **Blocker 2.** Define “ported golden vector”: shape, role (oracle vs reference), byte-equality vs semantic-equality, and the source-of-truth test files.
3. **Blocker 3.** Specify the classification record: format, location, and the CI/gate mechanism that verifies presence for every §7 row.
4. **R-1 through R-4.** Apply as one-line clarifications.

None of these reopens the four-tier topology, the core/specific promotion principle, the equivariance slice, or the GR-local algebra rule. All four are contract-pinning in the same sense the previous rounds of review used.

---

## V. Final verdict

```text
REVIEW: plan9.2.md

Sibling workspace topology:                       PASS
phys-gr byte-untouched (D3):                      PASS
No phys-gr imports (D4):                          PASS
Empty core Phase-1 boundary (D5):                 PASS (residual R-1)
Phase 4 scaffolding only (D6):                    PASS
phys-lib/gr Phase-1 scope (D7, F1):               PASS
Nested protocol fixture (D8):                     PASS
Disjoint GR-local algebra (D9):                   PASS
Fixture symbols (D10):                            PASS
Tensor split (D11):                               PASS
Namespace separation (D12):                       PASS
BaseFrameworkRefs artifact-only (D13):            PASS
Phase-1 determinism (D14):                        PASS
GR Category vocabulary (D15):                     FAIL (Blocker 1)
Vocabulary tests semantic (D16):                  PASS
go.work local-only (D17):                         PASS
R1 chain-rule rejection:                          PASS
R2 fixture directory pin:                         PASS
R3 rule identity pair:                            PASS
R4 fixture2 fully conforming minimal:             PASS
R5 no Phase-1 Phys imports:                       PASS
R6 arrows permitted not required:                 PASS
Deferred F1–F7 boundaries:                        PASS
Refactoring map (§7):                             PASS
Phase plan (§8):                                  PASS
Protocol migration (§9):                          PARTIAL (Blocker 2)
Equivariance fixture plan (§10):                  PASS
fixture2 conformance plan (§11):                  PASS
Acceptance matrix (§12):                          PASS as a set
Migration gates (§13):                            PASS
Risks (§14):                                      PASS
Quality gate (§15):                               PASS as a checklist

Blocker 1  D15 vs spec §14.2:                     OPEN
Blocker 2  Ported-vector mechanism:               OPEN
Blocker 3  Classification record mechanism:       OPEN

R-1  Empty core package shape:                    residual
R-2  SUBSTITUTE phrasing:                         residual
R-3  OperationInvocation immutability:            residual
R-4  Version-label hygiene:                       residual

FINAL STATUS: NOT CLOSED. THREE DEMONSTRATED BLOCKERS.
```

---

## VI. Assessment

`plan9.2.md` is the closest document in the entire chain to a genuinely closed implementation plan. Its topology, decisions, refactoring map, phase plan, and quality gate are all substantially correct, and its rejection of three review-introduced inventions (R1, R3, R4) is exactly the kind of discipline the earlier rounds were missing.

The three remaining blockers are not architecture. They are:

1. **One spec contradiction** (D15 closing a spec-open vocabulary).
2. **One undefined mechanism** (ported vectors — byte vs semantic, oracle vs reference).
3. **One undefined verification mechanism** (classification record — format, location, gate).

Each is a contract-pinning task. None requires revisiting the four-tier split, the `phys-math` purity rule, the `core` promotion principle, the `phys-gr` preservation rule, the equivariance fixture, or the GR-local algebra rule.

Two structural observations worth recording:

- The plan’s distinction between **review criteria** (mathematician test, quality gate items) and **mechanical CI gates** (imports, vocabularies, `go mod graph`, determinism) is the right pattern. It acknowledges that not everything is checkable mechanically without inventing a theorem prover. This is a maturity milestone for the chain.
- The plan’s explicit rejection of R1 (chain-rule Derivative node), R3 (colon-joined rule IDs), and R4 (subset conformance) is exactly what the earlier “stop rule” clauses were designed to enable. The plan is applying the discipline it inherits.

The remaining work is one focused amendment: fix D15, pin the ported-vector mechanism, pin the classification record mechanism. That is one document, not another review round.