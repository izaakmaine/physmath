I read the four adversarial reviews carefully and consolidated them against `plan9.1.md`. The reviews are remarkably consistent: **the topology is correct and should not be redesigned, but `plan9.1` is not yet safe to hand to an implementation agent because several decisions are still left implicit.** The strongest common finding is that the remaining issues are closure/pinning issues, not a need for another architectural redesign. plan9.1_deepseek plan9.1_gemini plan9.1_z

The four reviews collectively identify the following decisions that need to be **explicitly adopted in 9.2**:

| Area | 9.2 decision |
|---|---|
| **B1: trig / GR-local algebra** | **Adopt A.** Keep `Sin`, `Cos`, unknown-function shape, trig simplification, and related GR-only algebra entirely inside `phys-lib/gr`. `phys-math` remains the closed 10-node representation and is never dynamically extended. |
| **B2: regime conditions** | **Adopt A.** Framework-level GR regularity/non-vanishing conditions are `PhysicalAssumption` artifacts. `MathCondition` is attached to derivations only when required by a `phys-math` rule contract. |
| **`phys-gr` preservation** | The existing `phys-gr` codebase is **read-only historical/reference material**. Never modify, move, rename, delete, or refactor it. |
| **GR protocol refactor** | The **new** canonical protocol fixture in `phys-lib/gr` gets the nested package layout. This satisfies the protocol-refactor requirement; the original `phys-gr/solution/` remains untouched. The reviews explicitly identified the old wording as ambiguous and requiring this pin. plan9.1_deepseek |
| **`phys-gr` imports** | **No new module may import `github.com/PithomLabs/phys-gr`.** Remove the “temporary shim allowed” exception. Legacy validation uses copied golden vectors, not imports. plan9.1_z |
| **`phys-lib/core`** | Starts empty. Phase 4 is **scaffolding/classification setup only**; no speculative promotion. Actual classification/promotion occurs while refactoring `phys-lib/gr`. plan9.1_deepseek |
| **Core durable artifacts** | `phys-lib/core` is **in-memory only in Phase 1**. It publishes no durable artifact families, no `FrameworkArtifact`, and no new `ArtifactRef`-bearing schema. This closes the closed-family-registry issue identified by Z. plan9.1_z |
| **Phase-1 GR scope** | Pin the recommended interpretation: `phys-lib/gr` Phase 1 delivers framework artifacts, the six GR framework families, GR-local algebra, and the equivariance fixture. **The Schwarzschild protocol remains canonical in untouched `phys-gr/solution/` and is not reimplemented in Phase 1.** This avoids silently importing the frozen `phys` module and avoids scope explosion. plan9.1_z |
| **GR-local expression type** | GR-local expression representation is a **disjoint Go type** from `phys-math.Expression`; no shared interface and no tagged-union extension. Only an explicit deterministic projection of the representable subset is allowed. plan9.1_qwen |
| **Equivariance symbols** | No bare `"g"` / `"x"` symbols in durable artifacts. They must derive from the exact `ParameterID`s of the declared `Function` artifacts. plan9.1_qwen |
| **Tensor boundary** | `phys-math` owns generic mathematical tensor structure; GR owns chart, metric-signature, symmetry, connection, and other physical validation. plan9.1_qwen |
| **MathCondition → PhysicalAssumption** | Pin the mechanical bridge; no condition is silently discharged. plan9.1_qwen |
| **Namespace identity** | Go module names and durable artifact namespaces remain distinct. Do **not** rename `physmath` namespace merely because the Go module is `github.com/PithomLabs/phys-math`; likewise GR's durable framework namespace remains its specified value. plan9.1_z |
| **`BaseFrameworkRefs`** | Artifact/data dependency only; never a permission for Go imports between framework packages. plan9.1_z |
| **fixture2** | Pin its exact namespace and explicitly define whether the test fixture is a fully conforming framework or a minimal conformance harness; do not leave “e.g.” values. plan9.1_z |
| **Cross-implementation determinism** | Do not pretend a single implementation proves the two-implementation property. Make Phase 1 determinism a single-implementation acceptance criterion and explicitly defer true independent-implementation equivalence until a second implementation exists. |
| **Frozen `phys`** | With the Phase-1 GR scope above, **new modules have no `phys` dependency at all**. Future imports, if ever required, must use workspace wiring rather than inventing an unavailable published version. plan9.1_z |
| **Vocabulary tests** | Make them semantic/mechanical rather than substring traps. Mathematical `Dimension`, comparison operators, `Function`, etc. are not automatically forbidden; physical meaning is what is prohibited. |
| **Version/tagging** | Do not invent release/version mechanics as part of Phase 1. `go.work` is local development wiring; publication/tag strategy is separate unless required by the spec. |
| **B1/B2 status** | They become **adopted architectural decisions**, not “recommended” or “awaiting input”. |

This closes the original four blockers plus the additional topology-specific blockers identified by the other reviews. The four-review consensus is that the architecture itself is sound; the missing work is to turn these into explicit pins and remove contradictory wording. plan9.1_deepseek plan9.1_z

One subtle point is especially important: **do not let 9.2 “solve” ambiguity by inventing new PhysMath families, new framework semantics, or a new generic core ontology.** The reviews repeatedly reinforce the closed representation and the evidence-based core boundary. plan9.1_deepseek

## Prompt for the coding agent — write `plan9.2.md`

```text
You are the senior implementation architect responsible for producing the next implementation plan.

TASK
Write `plan9.2.md`.

Do NOT implement code.
Do NOT modify repositories.
Do NOT generate starter code.
Do NOT merely summarize plan9.1 or the reviews.

Your job is to produce a single, implementation-authoritative plan that closes every remaining architectural ambiguity so that a coding agent can execute Phase 0 onward without inventing any architectural decision.

AUTHORITATIVE INPUTS

1. PM-REP-001 v1.2 — authoritative specification.
2. `plan9.1.md` — current implementation plan.
3. Four adversarial reviews of `plan9.1`:
   - `plan9.1_deepseek.md`
   - `plan9.1_gemini.md`
   - `plan9.1_qwen.md`
   - `plan9.1_z.md`
4. Actual existing repository:
   `/home/chaschel/Documents/go/phys-gr`
5. Existing workspace:
   `/home/chaschel/Documents/go/`

IMPORTANT VERSION RULE
The authoritative specification for this task is PM-REP-001 v1.2.
Some review text refers to v1.4 because of an authority-label mismatch in the surrounding materials. Do not redesign semantics because of that naming mismatch. Treat PM-REP-001 v1.2 as the operative specification and record the version-label discrepancy only as documentation hygiene.

PRIMARY GOAL

The acceptance criterion for plan9.2 is:

NO ARCHITECTURAL BLOCKERS.
NO AMBIGUITY FOR THE CODING AGENT.
NO UNAUTHORIZED ARCHITECTURAL INVENTION REQUIRED DURING IMPLEMENTATION.

Every issue identified by the four reviews must be either:
- explicitly decided and adopted,
- explicitly deferred with a defined future boundary,
- or explicitly ruled out.

Do not leave any “recommended”, “probably”, “e.g.”, “TBD”, “awaiting architectural input”, or equivalent wording for an implementation-critical decision.

==================================================
I. LOCKED ARCHITECTURAL TOPOLOGY
==================================================

Preserve this exact repository model:

/home/chaschel/Documents/go/
├── go.work
├── phys/                  # frozen existing repository; DO NOT MODIFY
├── phys-gr/               # historical artifact and migration reference; DO NOT MODIFY
├── phys-artifact/         # canonical module
├── phys-math/             # canonical module
└── phys-lib/              # canonical module

Modules:

github.com/PithomLabs/phys-artifact
github.com/PithomLabs/phys-math
github.com/PithomLabs/phys-lib

`phys-lib` is ONE Go module with packages:

phys-lib/core
phys-lib/gr
phys-lib/internal/conformance/fixture2

Do not create separate Go modules for `gr`, `qm`, `bohm`, `cst`, or `cdt`.

`phys-gr` is NOT the final architecture.
It is the preserved historical/source artifact.

CRITICAL PRESERVATION RULE:

The existing `phys-gr` repository remains byte-untouched.
Do not:
- edit files,
- move files,
- rename files,
- delete files,
- truncate files,
- migrate files in place,
- add compatibility code to it,
- or change its architecture.

Refactoring means:

existing `phys-gr`
        ↓
fresh canonical implementation
        ↓
`phys-lib/gr`

The original `phys-gr` remains present as historical reference.

==================================================
II. FINAL DEPENDENCY DIRECTION
==================================================

Canonical dependency order:

phys-lib/<framework>
        ↓
phys-lib/core
        ↓
phys-math
        ↓
phys-artifact

Direct use of `phys-math` and `phys-artifact` from `phys-lib/gr` is allowed where required by the specification.

The direction is a PERMITTED partial order, not a requirement that `phys-lib/gr` must import `core`.

Prohibit:
- phys-artifact → anything
- phys-math → phys-lib/*
- phys-lib/core → framework packages
- framework-package → another framework-package via Go import
- any new canonical module → github.com/PithomLabs/phys-gr

`BaseFrameworkRefs` is a DATA/ARTIFACT dependency only.
It never authorizes a Go import between framework packages.

The Phase-1 implementation must contain NO active dependency on `phys-gr`.

==================================================
III. LEGACY `phys-gr` IMPORT / SHIM DECISION
==================================================

Delete the conceptual “temporary shim allowed” exception from plan9.1.

New modules MUST NOT import:

github.com/PithomLabs/phys-gr

under any circumstance.

There is no compatibility-shim architecture.

Legacy validation is done using:
- ported golden vectors,
- copied expected values,
- protocol-semantic comparison,
- deterministic replay checks,

NOT by importing `phys-gr`.

`phys-gr` may be inspected as historical/source material only.

==================================================
IV. PHYS-ARTIFACT
==================================================

`phys-artifact` is strictly mechanical.

It contains:
- ArtifactRef
- digest framing
- primitive encoding
- embedded encoding
- Set
- Optional
- Sequence
- BigInt
- grammar primitives
- other purely mechanical artifact/identity machinery explicitly required by PM-REP-001

It contains NO:
- mathematical expression concepts
- physical semantics
- theory/framework concepts
- derivation semantics
- theorem semantics
- protocol semantics
- generic utility dumping-ground abstractions

Its vocabulary test must be SEMANTIC rather than naive substring matching.

Do not reject an identifier merely because a substring happens to resemble a forbidden concept.
Use whole identifiers / exported API semantics / imports / declared schema meaning.

==================================================
V. PHYS-MATH ABSOLUTE BOUNDARY
==================================================

`phys-math` contains ONLY pure mathematical representation and execution.

It contains:
- closed PM-REP expression representation
- Function / Map / Tensor / Index / Collection / OperatorApplication
- ParameterID / ConstantID / BoundParameterRef
- Equation / EquationSet / Definition / MathCondition
- Transformation
- DerivationStep
- Derivation / Theorem
- canonical identity/serialization
- the seven closed rule families
- deterministic replay

It MUST NOT contain:
- GR
- QM
- physical dimensions such as meter/second/etc.
- physical assumptions
- physical interpretations
- metric conventions
- theory names
- theory-specific transforms
- physical-domain ontology
- trust/provenance/corpus semantics

Clarification:
Mathematical concepts such as:
- Tensor.Dimension
- rank
- index variance
- EQ / NEQ / LT / LTE / GT / GTE
- mathematical Function
are allowed.

The ban on “conventions” refers to physical/theory conventions, not ordinary mathematical operators.

==================================================
VI. B1 — CLOSED ARCHITECTURAL DECISION
==================================================

ADOPT OPTION A.

`phys-math` remains the closed 10-node representation.
Do NOT add:
- Sin node
- Cos node
- arbitrary GR-specific nodes
- dynamically extensible node types

GR expressions requiring:
- Sin
- Cos
- UnknownFunction shape
- GR-specific simplification
- regime-aware zero testing
- GR derivative caps
- other GR-local symbolic behavior

remain inside `phys-lib/gr` as theory-local algebra.

Equivariance does not use trig and must not depend on GR-local algebra.

If GR-local algebra needs to project a representable subset into PhysMath,
use an explicit deterministic conversion.

Do NOT create a shared interface implemented by both the GR-local expression type and phys-math.Expression.

Do NOT extend the PhysMath tagged union dynamically.

If a local GR expression cannot be represented by PhysMath, the conversion must fail explicitly with a dedicated error.

==================================================
VII. B2 — CLOSED ARCHITECTURAL DECISION
==================================================

ADOPT OPTION A.

Framework-level GR regime/non-vanishing requirements belong in framework-owned `PhysicalAssumption` artifacts.

Examples:
- r != 0
- A(r) != 0
- sin(theta) != 0
- regular-coordinate-domain requirements

Do NOT attach these to every derivation step merely because the GR implementation happens to depend on them.

`phys-math.DerivationStep` carries `MathCondition` only when required by a closed PhysMath transformation-rule contract.

Example:
`EXPR-SIMPLIFY-001` requiring a proof obligation for `Pow(x,0)`.

Mechanical bridge:

1. phys-math rule execution returns `TransformResult` with the MathCondition when it cannot discharge it.
2. phys-lib/gr captures that condition reference.
3. phys-lib/gr creates a PhysicalAssumption referencing that MathCondition.
4. The relevant PhysicalTheorem / PhysicalEquation references the PhysicalAssumption.
5. No condition is silently discharged.

==================================================
VIII. CORE / SPECIFIC ARCHITECTURE
==================================================

`phys-lib/core` starts EMPTY.

This is deliberate.

Core promotion requires the §8 classification record from plan9.1.

For every significant abstraction considered during refactoring, classify exactly one:

PHYS-MATH
PHYS-LIB/CORE
PHYS-LIB/GR
SPLIT
REPLACE
DELETE
DEFER

For SPLIT:
- identify exact type/function/interface boundary
- identify ownership of both sides
- identify dependency direction

Eligibility for `phys-lib/core` requires:
- theory-independent physical meaning
- stable semantics across frameworks
- no theory-specific assumptions in API
- meaningful reusable interface
- demonstrable multi-framework consumer evidence
- documentation possible without naming a specific theory

“Useful to us” is insufficient.

Default for ambiguity:
KEEP IN `phys-lib/gr`.

==================================================
IX. CORE DURABLE-ARTIFACT RULE
==================================================

NEW ARCHITECTURAL DECISION:

`phys-lib/core` is IN-MEMORY ONLY in Phase 1.

It does NOT publish:
- FrameworkArtifact
- durable artifact families
- new durable schemas
- ArtifactRef-bearing domain families

The durable-family registry remains closed by PM-REP.

If a future core abstraction requires durable representation:
- either it must be expressed through an already-authorized representation,
- or a future versioned specification change must authorize a new durable family.

Do not invent a core artifact namespace.

==================================================
X. PHASE 4 REDEFINED
==================================================

Phase 4 is NOT actual core extraction.

Phase 4 consists only of:
- create phys-lib module if needed
- create empty phys-lib/core package
- establish package/build scaffolding
- establish classification-record mechanism
- verify the core package is empty of promoted domain abstractions

NO core promotion occurs in Phase 4.

The actual candidate inventory/classification is performed while refactoring `phys-lib/gr` in Phase 5, when concrete extracted code exists and consumer evidence can be assessed.

Do not populate core merely to make it look useful.

==================================================
XI. `phys-lib/gr` SCOPE — PHASE 1
==================================================

PIN THIS SCOPE.

Phase-1 canonical `phys-lib/gr` delivers:

1. GR framework identity / FrameworkArtifact
2. six authorized framework-owned families required by the specification
3. GR physical assumptions and interpretations required by those families
4. GR-local symbolic algebra needed by the port
5. the Phase-1 equivariance protocol fixture
6. conformance/integrity tests required by the specification

The following remain ONLY in untouched historical `phys-gr` during Phase 1:

- original `solution/`
- original Schwarzschild protocol implementation
- original replay implementation
- original session-coupling to frozen Phys

The Schwarzschild solution remains the semantic/reference anchor.

Its full re-expression in `phys-lib/gr` is DEFERRED to a separately scoped future phase.

Do NOT import frozen `phys` merely to reproduce the old SessionCorrespondence in Phase 1.

This is an explicit Phase-1 scope boundary.

==================================================
XII. `phys-gr` PROTOCOL REFACTOR
==================================================

The protocol-pattern requirement is satisfied by creating the NEW canonical GR fixture inside `phys-lib/gr`.

Use the canonical nested structure:

<fixture>/
    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go

Reuse the ACTUAL schema/loading/execution semantics from the existing GR example.

Do NOT invent a provisional manifest schema.

Do NOT alter the original:
`phys-gr/solution/`

Explicitly state:

The new `phys-lib/gr` fixture is the canonical refactored protocol fixture.
The original `phys-gr/solution/` is preserved unchanged as historical/reference material.

==================================================
XIII. EQUIVARIANCE FIXTURE
==================================================

The equivariance fixture is owned by `phys-lib/gr`.

Do NOT create `phys-equiv`.

Use the exact six-step derivation already fixed by PM-REP-001.

Use staged Calls for the required intermediate representations.

Do not:
- use durable Map-for-f substitution
- use durable FunctionApplication for the staged LHS/RHS
- introduce distributivity as an extra rule
- invent a GroupAction canonical sort
- introduce trig
- invent a new rule family

Parameter binding is mandatory.

The fixture MUST define Function artifacts for:
- f
- rho_X
- rho_Y

Every durable free symbol MUST bind through the Function declaration's ParameterID.

Do NOT use a bare:
NewSymbol("g")
NewSymbol("x")

for the durable fixture.

Use the exact generated ParameterIDs from the Function declarations.

Pin all golden ArtifactRefs deterministically.

If rho_X and rho_Y resolve to the same canonical PhysMath artifact in the minimal fixture, document that this is intentional:
their physical distinction lives in the framework interpretation, not in duplicate mathematical identity.

==================================================
XIV. FIXTURE2
==================================================

Second-framework conformance fixture lives at:

phys-lib/internal/conformance/fixture2/

It is:
- test-only
- never imported by production packages
- never a production theory package
- never a dependency of core
- never a dependency of gr

Pin one exact namespace string.
Do not say “e.g.”.

Use a valid `phys-*` namespace and keep it stable across golden vectors.

Define clearly whether the fixture is:
- a fully conforming minimal framework,
or
- a conformance harness that exercises only the framework capabilities required by the test.

Align this explicitly with the literal PM-REP definition of “conforming framework namespace”.
Do not rely on an implicit interpretation.

The fixture must prove:
- distinct framework identity
- PhysMath identity unchanged across frameworks
- no global-framework winner
- unrelated-assumption references rejected
- cyclic/self-framework dependency rejection
- cross-framework disagreement is representable without changing PhysMath identity

==================================================
XV. TENSOR BOUNDARY
==================================================

`phys-math` owns mathematical tensor structure only:

- rank
- mathematical dimension/size
- index variance
- dense representation
- generic shape validation
- generic tensor operations authorized by PM-REP

`phys-lib/gr` owns:
- ChartID compatibility
- metric signature
- GR symmetry semantics
- connection/Christoffel semantics
- GR-specific contraction semantics
- distinction between connection coefficients and ordinary mathematical tensors
- GR semantic validation

Do NOT place GR chart/signature/connection meaning into phys-math.

==================================================
XVI. NAMESPACE IDENTITY
==================================================

Do not conflate Go module identity with durable namespace identity.

Keep the specification-defined namespace constants exactly where they are required.

For example:
Go module:
github.com/PithomLabs/phys-math

does NOT imply changing the durable namespace:
physmath

Likewise, do not “harmonize” the GR durable namespace merely because its Go package now lives at:
github.com/PithomLabs/phys-lib/gr

State all namespace constants explicitly in plan9.2.

Pin fixture2 namespace explicitly.

==================================================
XVII. PARAMETERID / BINARY FRAMING
==================================================

Pin the exact framing rules from PM-REP-001.

Do not invent alternate encoding.

ParameterID derivation must use the exact artifact framing primitives required by the specification.

For top-level parameters, explicitly state the empty ScopePath encoding.

Function parameter slot ordinals must deterministically derive the corresponding ParameterID.

BoundParameterRef must use the exact durable reference/framing construction from the spec.

==================================================
XVIII. DIFFERENTIATE / CALL NODES
==================================================

The plan must explicitly resolve how `DIFFERENTIATE-001` handles `Call` nodes WITHOUT importing theory-specific derivative knowledge into PhysMath.

The implementation plan must distinguish:
- generic structural differentiation available to PhysMath
- derivative information associated with mathematical Function artifacts
- theory-specific derivative/rewrite knowledge, which belongs outside PhysMath

Do not invent a GR-specific derivative operator or a new AST node.

If the current PM-REP contract already specifies the representation of derivative information, use that exact mechanism.

If it does not, the plan must identify the smallest compatible implementation consistent with the existing closed families and state the limitation rather than silently extending the schema.

==================================================
XIX. RULE EXECUTION LOCK
==================================================

Only these seven rule identities are executable in Phase 1:

physmath:EXPR-CANON-001
physmath:EXPR-SIMPLIFY-001
physmath:EXPR-SUBSTITUTE-001
physmath:RAT-NORMALIZE-001
physmath:DIFFERENTIATE-001
physmath:SELECT-BRANCH-001
physmath:EXPR-APPLY-001

Unknown rule IDs fail closed.

OperatorApplication.OperatorRef remains absent/invalid in Phase 1 except where the specification explicitly authorizes it.

`OperatorID` semantics must remain exactly those pinned by PM-REP.

==================================================
XX. GO.WORK / FROZEN PHYS
==================================================

Extend the workspace to include:

./phys
./phys-artifact
./phys-math
./phys-lib

`phys` remains untouched.

New modules contain NO `replace` directives.

Phase 1 has ZERO imports of `phys`.

Therefore:
- no local-path dependency on `phys`
- no fabricated version dependency
- no frozen namespace references in new Phase-1 artifacts

Future Phys integration, if ever needed, is a separate decision.

`phys-gr` is removed from active workspace membership only after the explicitly defined migration gate is satisfied.

The exact verification must be mechanical:
- grep/import scan
- `go list`
- `go mod graph`
- `go test ./...`

==================================================
XXI. VERSION / PUBLICATION POLICY
==================================================

Do not invent release/tag semantics as part of the implementation architecture.

For Phase 1:
- module identity is declared in each go.mod
- go.work is local workspace wiring
- publication/tag strategy is explicitly outside Phase-1 architecture unless required by PM-REP

==================================================
XXII. PHYSICAL ASSUMPTION CATEGORIES
==================================================

Pin a small, explicit vocabulary for GR framework assumptions rather than leaving free-form implementation guesses.

At minimum distinguish:
- convention
- regularity/domain condition
- model/interpretation assumption

Do not encode GR convention meaning in comments only.

Examples such as:
- metric signature
- curvature convention
- cosmological constant choice
- coordinate regularity/non-vanishing
must become explicit framework-owned assumption artifacts where the specification requires them.

Only predicates expressible by PhysMath may be carried through MathCondition/ConditionRefs.
Non-expressible descriptive conventions use package-owned descriptions and no fabricated PhysMath condition.

==================================================
XXIII. CROSS-IMPLEMENTATION DETERMINISM
==================================================

Do not claim that a single implementation has demonstrated independent cross-implementation equivalence.

Phase-1 acceptance:
- deterministic serialization
- deterministic identity
- deterministic replay
- repeated replay yields identical golden ArtifactRefs

True cross-implementation equivalence is a future acceptance requirement that becomes active when an independent implementation exists.

==================================================
XXIV. FORMALIZATION / PROTOCOL AUTHORITY
==================================================

The protocol is executable, not prose-only.

Required fixture structure:

<pkg>/
    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go

The actual existing GR example remains the semantic/protocol precedent.

Reuse:
- manifest schema
- strict typed decoding
- unknown-field rejection
- artifact-presence checks
- ordered operation execution
- firewalls
- mutation tests
- independence checks
- replay semantics

Do NOT invent a new manifest schema.

If an equivariance requirement cannot be represented using the authorized schema:
STOP and report PROTOCOL GAP.
Do not add a convenient new manifest field.

==================================================
XXV. TEST / QUALITY GATES
==================================================

The plan must define concrete acceptance gates for:

1. dependency graph
2. namespace separation
3. phys-artifact purity
4. phys-math purity
5. core emptiness/promotion records
6. GR-locality of theory-specific algebra
7. legacy phys-gr immutability
8. zero new imports from phys-gr
9. zero Phase-1 imports of frozen Phys
10. ParameterID determinism
11. canonical encoding
12. rule-ID closure
13. derivation-step target-family validation
14. condition threading
15. staged-call linkage
16. equivariance six-step golden refs
17. fixture2 isolation
18. architecture-of-complexity classification records
19. protocol-manifest compliance
20. deterministic replay

Mechanical CI tests must be distinguished from architectural review criteria.

The “mathematician test” is a REVIEW CRITERION.
Vocabulary/import/API scanning is the MECHANICAL gate.

==================================================
XXVI. REFACTORING MAP
==================================================

Retain and improve plan9.1's refactoring table.

For every major existing `phys-gr` component provide:

CURRENT
TARGET
CLASS
COMMON DENOMINATOR
SPECIFIC DIFFERENCE
TARGET INTERFACE
DEPENDENCIES
TEST IMPACT
SEQUENCE

Do this before implementation.

Especially resolve:

symbolic/expr.go
symbolic/normalize.go
symbolic/differentiate.go
symbolic/subst.go
symbolic/truncate.go
tensor/
coordinates/
metric/
connection/
curvature/
vacuum/
limit/
replay/
solution/

Do not treat “move” as sufficient.
The new implementation is a deliberate refactor/re-expression against the locked architecture.

==================================================
XXVII. PHASE ORDER
==================================================

Use an execution order consistent with the final decisions:

Phase 0
Authority + repository inventory + workspace verification.
NO source modifications to phys or phys-gr.

Phase 1
phys-artifact.

Phase 2
phys-math representation + identity.

Phase 3
phys-math seven rules + derivation/replay.

Phase 4
phys-lib module + EMPTY core scaffolding + classification-record mechanism.

Phase 5
phys-lib/gr framework implementation and refactoring from phys-gr.
Perform actual core/specific classification here.
Default ambiguous items to gr.
No speculative core promotion.

Phase 6
GR protocol fixture in canonical nested layout using actual GR schema/loading semantics.

Phase 7
Equivariance six-step vertical slice.

Phase 8
Second-framework conformance fixture.

Phase 9
Full golden vectors, negatives, contamination tests, deterministic replay, architecture conformance.

Phase 10
Final migration gate:
- no active phys-gr dependency
- no new Phys dependency
- remove phys-gr from go.work
- legacy phys-gr remains physically present and untouched
- final architecture audit

==================================================
XXVIII. REQUIRED DELIVERABLE FROM YOU
==================================================

Produce `plan9.2.md` as a complete implementation plan.

It must contain:

1. Executive architectural status
2. Explicit adopted decisions
3. Explicit deferred decisions
4. Final repository/module/package topology
5. Dependency rules
6. Core/specific classification protocol
7. Full refactoring map
8. Phase-by-phase implementation plan
9. Exact protocol migration plan
10. Exact equivariance fixture plan
11. Exact fixture2 conformance plan
12. Acceptance/test matrix
13. Migration gates
14. Risks and failure modes
15. Architectural quality gate
16. Final “coding-agent handoff” checklist

At the beginning state:

STATUS: ARCHITECTURALLY CLOSED FOR IMPLEMENTATION PLANNING

At the end state:

FINAL HANDOFF:
- zero unresolved architectural blockers
- zero implementation-critical ambiguities
- no decision left to the coding agent
- no modification of phys-gr
- no modification of frozen phys
- phys-math remains pure mathematics
- phys-lib/core remains empty unless evidence-based promotion occurs
- phys-lib/gr owns GR semantics
- no phys-gr imports in canonical modules
- Phase-1 GR scope is explicitly bounded
- protocol authority is the existing GR precedent
- equivariance is the canonical Phase-1 vertical slice

IMPORTANT
Do not write code.
Do not ask the architect to make the decisions already pinned above.
Do not reopen the settled module topology.
Do not invent new PhysMath families.
Do not invent a generic PhysicalContract.
Do not create phys-equiv.
Do not create theory-specific modules that are deferred.
Do not populate core for convenience.
Do not modify phys-gr.
Do not modify frozen phys.

If any statement in the four reviews appears to conflict with the actual PM-REP-001 v1.2 text or the actual inspected `phys-gr` repository, resolve it against the authoritative source and record the resolution explicitly in plan9.2 rather than silently guessing.

The output must be a plan, not implementation.
```

The key improvement over 9.1 is that **9.2 should not merely “add six pins.” It should rewrite the decision state of the plan so that every formerly ambiguous choice has an adopted architectural status and every deferred area has an explicit boundary.** That is what the reviews collectively say is needed to make the handoff genuinely executable. plan9.1_deepseek plan9.1_z