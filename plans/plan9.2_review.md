## prompt

wait, just to clarify, does gr under phys-lib has a dependency problem where it depends on phys-math and phys-lib/core first before it can do its thing

## answer

Yes — **there is no dependency problem in the current architecture**, provided we distinguish **dependency direction** from **implementation sequencing**.

The key point in `plan9.2` is that:

```text
phys-lib/gr
    ├──> phys-lib/core       [permitted]
    ├──> phys-math           [permitted]
    └──> phys-artifact       [permitted where required]
```

`core` is **not a prerequisite that `gr` must import before it can function**. The plan explicitly says the dependency arrows are a **permitted partial order, not required imports**, and that `gr` need not import `core` while `core` is empty. plan9.2

That is important because we deliberately locked:

```text
phys-lib/core = empty in Phase 1
```

and actual promotion into core happens only when an abstraction is proven to be theory-agnostic. plan9.2

So the actual situation is:

```text
                 ┌───────────────┐
                 │ phys-artifact │
                 └───────▲───────┘
                         │
                 ┌───────┴───────┐
                 │   phys-math   │
                 └───────▲───────┘
                         │
              ┌──────────┴──────────┐
              │  phys-lib/core      │
              │  (empty initially)  │
              └──────────▲──────────┘
                         │
                    ┌────┴────┐
                    │phys-lib/│
                    │   gr    │
                    └─────────┘
```

But **phys-lib/gr can legally bypass the empty `core` and depend directly on `phys-math`**:

```text
phys-lib/gr ───────────────► phys-math ───► phys-artifact
       │
       └──► phys-lib/core   (only when a real core abstraction exists)
```

That's actually the right architecture for our current Phase 1.

### Why this is desirable

We do **not** want to manufacture a `core` abstraction just because `gr` needs to compile.

That would invert the principle we established:

> **Core is discovered from demonstrated commonality; it is not populated in anticipation.**

So implementation should work like this:

```text
1. Build phys-artifact
2. Build phys-math
3. Create empty phys-lib/core
4. Build phys-lib/gr directly against phys-math
5. During GR refactoring, identify candidates
6. Promote a candidate to core only when the evidence-based classification
   protocol proves it belongs there
```

This is exactly why Phase 4 is scaffolding-only and actual classification happens during the GR extraction phase. plan9.2

### One thing I would change in 9.2 wording

The diagram should **not visually imply**:

```text
gr → core → math
```

as though `core` is a mandatory runtime/build dependency.

Instead use:

```text
Allowed dependency partial order:

phys-lib/gr
    ├──→ phys-lib/core      (only if promoted abstractions are used)
    ├──→ phys-math
    └──→ phys-artifact       (where directly required)

phys-lib/core
    └──→ phys-math
```

That removes a potentially serious misunderstanding for the coding agent.

And this fits the deeper Architecture of Complexity principle: **we don't insert an intermediate layer merely because the diagram looks cleaner; we introduce it only when there is a real common denominator to unify.**

So, in plain terms:

**No, `phys-lib/gr` does not have to wait for `phys-lib/core` to become populated. `core` is optional at the dependency level until evidence gives it something real to own.**


## Consolidate Review

I consolidated the four reviews against `plan9.2.md` and the operative PM-REP specification, rather than treating the four reviews as equally authoritative. There is an important correction first:

**The Qwen review must be excluded from the architectural chain.** It is reviewing a different “Physics Compiler MVP v2.3” with `Challenge/Review`, `Session`, `MRC`, `E=mc²`, a 39-file tree, etc. Those concepts do not belong to this `phys-artifact` / `phys-math` / `phys-lib` architecture. plan9.2_qwen

The useful findings come from **DeepSeek, Gemini, and Z**, reconciled against PM-REP-001 and the actual `plan9.2.md`.

## Consolidated verdict

`plan9.2` is architecturally very strong, but **not yet at the stated target of zero architectural blockers and zero implementation-critical ambiguity**.

DeepSeek identifies three actual blockers: the GR-local `Category` rule conflicts with the specification unless scoped to GR, “ported golden vectors” are undefined, and the core/specific classification record mechanism is undefined. plan9.2_deepseek plan9.2_deepseek plan9.2_deepseek

Z identifies the deeper Phase-5 scope problem: the plan's refactoring table still implies re-expression of the GR computational pipeline into PhysMath even though the Phase-1 scope explicitly defers that pipeline and the PhysMath expressibility boundary makes some of it impossible. plan9.2_z

Gemini adds several mechanical pins that should be incorporated, especially the exact AST tag mapping, BigInt zero encoding, explicit equivariance function bodies, and a real `doc.go` for the empty core package. plan9.2_gemini plan9.2_gemini

There is also one **important Gemini recommendation that should NOT be adopted**: inventing `"phys-gr:func/sin"` / `"phys-gr:func/cos"` projection identifiers. That would cross the expressibility boundary and invent identity semantics not established by PM-REP. The correct rule is that only the **PhysMath-representable subset** projects; GR-local `Sin`/`Cos` remain GR-local and projection fails when a durable PhysMath representation cannot be formed.

---

# The closure decisions

These should become the authoritative architectural pins for the next revision.

### A. `PhysicalAssumption.Category` is GR-local, not global

Keep your locked three-value vocabulary:

```text
convention
regularity
model
```

But explicitly state:

> PM-REP defines `Category` as an open framework-owned string. `phys-lib/gr` voluntarily constrains its own Phase-1 Category vocabulary to `convention`, `regularity`, and `model`. This is a GR package contract, not a global `phys-*` restriction. Other frameworks may define different Category values without violating PM-REP.

This preserves your decision while eliminating the DeepSeek contradiction. plan9.2_deepseek

### B. Close the Phase-5 GR scope completely

This is the most important substantive correction.

Phase 1 `phys-lib/gr` is:

```text
FrameworkArtifact
six GR framework families
GR physical assumptions / interpretations
GR-local algebra
PhysMath projection of representable fragments
equivariance protocol fixture
required conformance tests
```

It is **not**:

```text
Christoffel computation
Ricci computation
Einstein/vacuum evaluation pipeline
weak-field computation pipeline
full Schwarzschild derivation port
SessionCorrespondence with frozen Phys
```

Those remain in the untouched historical `phys-gr` and are future work.

The refactoring map must therefore change wording such as:

```text
formula loops re-expressed on phys-math
```

to:

```text
PhysMath-expressible fragments projected to PhysMath;
GR computational pipeline remains GR-local and Phase-1 deferred.
```

This is exactly the scope correction Z is calling for. plan9.2_z

This also preserves the important distinction:

```text
phys-gr = historical implementation evidence / executable legacy baseline
phys-lib/gr = canonical new GR library
```

The original `phys-gr` remains untouched.

### C. Define “ported golden vector” precisely

Do **not** copy legacy canonical bytes.

The old `phys-gr` symbolic representation is not the new canonical PhysMath representation, so byte-for-byte legacy copying is impossible as a general rule. DeepSeek correctly flags this as a real ambiguity. plan9.2_deepseek

Lock the definition as:

> A Phase-1 ported vector is a **semantic regression fixture derived from the immutable `phys-gr` implementation**, whose expected canonical representation is independently encoded using the PM-REP canonical representation of the new architecture.

Three cases:

```text
PhysMath-expressible GR content
    → expected PhysMath canonical bytes / ArtifactRef pinned

GR-local algebra
    → deterministic package-local semantic test vector
    → no invented durable PhysMath family

non-representable GR content
    → explicit projection-failure vector
```

The old `phys-gr` bytes are **source evidence**, not canonical-byte equality targets.

Source-of-truth authority:

```text
phys-gr symbolic / metric / connection / curvature /
vacuum / limit tests and behavior
```

but the new expected bytes are derived under PM-REP, reviewed once, and then become the new regression oracle.

No import from `phys-gr`.

### D. Define the core/specific classification record protocol

This was the clearest remaining governance gap.

Use:

```text
phys-lib/
    classifications/
        001-symbolic-expr.md
        002-symbolic-normalize.md
        ...
```

One record per row in the refactoring map.

Each file has a fixed machine-readable preamble:

```text
---
id: symbolic-expr
source: phys-gr/symbolic/expr.go
class: SPLIT
target: phys-math, phys-lib/gr
status: adopted
---
```

followed by the required Architecture-of-Complexity analysis:

```text
Why it is theory-agnostic:
Known/current consumers:
Potential future consumers:
Theory-specific assumptions:
Stable interface:
Dependencies:
Why it does not belong in phys-math:
Why it does not belong in phys-lib/gr:
```

Rules:

```text
PHYS-LIB/CORE
    full evidence record required

SPLIT
    classification + exact interface/ownership boundary required

PHYS-MATH
PHYS-LIB/GR
REPLACE
DELETE
DEFER
    classification record required, but no core-promotion evidence section needed
```

The record is committed **with the change that establishes the classification**, not reconstructed later.

CI must verify that every refactoring-map row has exactly one classification record and that every `PHYS-LIB/CORE` classification contains the full evidence fields.

This directly closes the DeepSeek blocker. plan9.2_deepseek

### E. `phys-lib/core` gets a real Go package stub

Use:

```text
phys-lib/core/doc.go
```

containing package documentation and:

```go
package core
```

with no exported identifiers.

That preserves the “empty core” architectural meaning while making the package explicit to Go tooling. Gemini's observation is useful here. plan9.2_gemini

The classification-record directory is **documentation**, not a package under `core`, so it does not contaminate the empty-core test.

### F. Conditions must never be generated by Phase-1 rules

This is an important correction to the current wording in `plan9.2`.

The specification says Phase-1 rules do **not** generate new mathematical conditions; `Pow(x,0)` succeeds only when the exact `NEQ(x,0)` condition is already supplied. PHYSMATH_PHYS_PHYSSTAR_REPRESEN… PHYSMATH_PHYS_PHYSSTAR_REPRESEN…

Therefore the bridge is:

```text
artifact author constructs durable physmath/condition/1
        ↓
OperationInvocation.ConditionRefs supplies its ArtifactRef
        ↓
rule checks exact predicate
        ↓
TransformResult carries supplied ConditionRefs
        ↓
phys-lib/gr may reference that existing condition from PhysicalAssumption
```

Not:

```text
rule invents MathCondition
        ↓
TransformResult creates durable condition
```

That latter interpretation must be explicitly ruled out.

### G. Tensor operations remain out of Phase-1 PhysMath

Keep Tensor in PhysMath as representation/validation:

```text
rank
dimension
variance
dense shape
canonical representation
```

but **not** generic tensor arithmetic such as contraction, raising/lowering, etc., because the Phase-1 executable transformation registry is the seven-rule registry. Z explicitly called out this possible over-reading. plan9.2_z

GR tensor computation remains GR-side.

### H. Exact AST tag mapping must be written into the plan

Even though PM-REP already fixes it, include the explicit table:

```text
0x01 Symbol
0x02 Rational
0x03 Add
0x04 Mul
0x05 Neg
0x06 Pow
0x07 Sqrt
0x08 Call
0x09 Relation
0x0A BranchSet
```

Gemini's finding is valid as a zero-ambiguity implementation pin. plan9.2_gemini

### I. Equivariance function bodies are normative fixture data

Pin exactly:

```text
f(v)        = 2 * v
rho_X(g,v)  = g * v
rho_Y(g,v)  = g * v

LHS_0 = Call(f, [Call(rho_X, [g,x])])
RHS_0 = Call(rho_Y, [g,Call(f,[x])])
```

with the already-locked Function-derived ParameterIDs.

Do not let the implementation invent equivalent functions. Gemini correctly identifies this as necessary for deterministic execution. plan9.2_gemini

### J. Do NOT adopt Gemini's invented trig projection keys

Do **not** add:

```text
phys-gr:func/sin
phys-gr:func/cos
```

Those strings were proposed by Gemini but are not established by the architecture. plan9.2_gemini

The correct architectural rule is:

```text
GR-local Sin/Cos
    ↓
remain GR-local

ToPhysMath()
    ↓
only representable subset succeeds

non-representable GR expression
    ↓
ErrUnrepresentableProjection
```

No synthetic PhysMath function artifact is invented merely to make trig projectable.

This is consistent with the plan's own disjoint GR-local type rule and the expressibility boundary. plan9.2

### K. Exact substitution wording

Use the specification's stronger formulation:

> No two substitution entries may share the same `ParameterID`, **regardless of `DeclaringArtifactRef`**.

DeepSeek correctly identifies that the current shorthand “distinct ParameterIDs” is slightly too easy to misread. plan9.2_deepseek

### L. Namespace and fixture values

Keep:

```text
physmath       durable PhysMath namespace
phys-gr        GR durable namespace
phys-fixture2  second-framework durable namespace
```

Go module paths remain:

```text
github.com/PithomLabs/phys-math
github.com/PithomLabs/phys-lib
github.com/PithomLabs/phys-artifact
```

Do not harmonize these namespaces.

Also pin the Phase-1 fixture `FrameworkVersion` values rather than leaving them implicit. The clean deterministic choice is:

```text
GR fixture:       "1"
fixture2:         "1"
```

### M. Core remains genuinely empty

`phys-lib/core` containing only `doc.go` is a PASS.

No speculative:

```text
chart
replay helper
tensor helper
common
util
shared
generic physical interface
```

gets promoted merely because it appears reusable.

The Architecture-of-Complexity rule remains:

```text
coupling
    separate conflated concerns

cohesion
    unify only through smallest stable interface

core/specific
    move only the demonstrated true denominator downward
    keep variation outward
```

---

# Final consolidated status

After reconciliation:

```text
Sibling workspace topology                  PASS
phys-gr preservation                        PASS
No phys-gr imports                          PASS
phys-math purity                            PASS
phys-artifact boundary                      PASS
core empty/in-memory-only                   PASS
core/specific governance                    PASS after record pin
B1 GR-local algebra                         PASS
B2 condition architecture                   PASS after condition-materialization pin
Namespace separation                        PASS
BaseFrameworkRefs semantics                 PASS
Equivariance structure                      PASS
fixture2 namespace/conformance              PASS
Protocol authority                          PASS
AST tags                                    PASS after explicit listing
Parameter binding                           PASS
Rule closure                                PASS
Tensor boundary                             PASS after explicit no-ops pin

Category scope                              MUST PIN GR-LOCAL
Ported-vector definition                    MUST PIN
Classification-record mechanism             MUST PIN
Phase-5 GR computational scope               MUST PIN
Condition materialization                   MUST PIN
```

So the reviews converge on **four substantive closure pins**, not another architectural redesign:

```text
1. GR Category enum is package-local, not global.
2. Phase-5 GR scope excludes the computational pipeline.
3. Ported vectors have an explicit semantic-regression format.
4. Core/specific classifications have an explicit committed-record protocol.
```

Then there are several mechanical clarifications, especially condition materialization, exact AST tags, empty `core/doc.go`, substitution wording, and fixture data.

The existing `plan9.2` already has the right foundation: its D1–D17 decisions, D3/D4 preservation rules, D7 scope, D8 protocol structure, D9 disjoint algebra, and D10 parameter binding are all sound. plan9.2

**Conclusion: `plan9.2` is not yet closed, but it is now one focused amendment away from closure. No macro-architecture should be reopened.**


## Qwen Review

Yes. I reviewed the attached `plan9.1_qwen(1).md` against the **current architecture we have subsequently locked**, not just against its own conclusions. The result is that it contains some genuinely useful findings, but several of its “required amendments” are now stale or technically inconsistent with the later PM-REP decisions.

The review itself explicitly says it is auditing `plan9.1` against PM-REP-001 v1.2 and identifies five traps. plan9.1_qwen

## Net-validity assessment

| Finding | Net status | Why |
|---|---|---|
| `phys-gr` / `phys-lib/gr` lifecycle | **PARTIALLY VALID** | Correct that the new canonical GR implementation belongs in `phys-lib/gr` and old `phys-gr` is preserved. But its claim that *ported Schwarzschild* should be a new Phase-1 fixture is now superseded by our explicit Phase-1 scope decision. |
| GR-local expression type must be disjoint | **VALID** | This remains an important boundary and is already locked in D9. |
| Equivariance `g,x` must use Function-derived ParameterIDs | **VALID** | Still fully valid and already locked in D10. |
| Tensor math vs GR semantics boundary | **VALID, WITH A TIGHTER PHASE-1 SCOPE** | The conceptual boundary is right, but Phase-1 PhysMath tensor behavior should be representation/validation only; no generic tensor operations should be silently introduced. |
| MathCondition → PhysicalAssumption bridge | **INVALID AS WRITTEN** | The review assumes a Phase-1 rule *generates* a new MathCondition. PM-REP explicitly says Phase-1 rules generate no new mathematical conditions; `Pow(x,0)` succeeds only when an exact condition is already supplied.  |

### 1. `phys-gr` Day-2 state: only half of the review survives

The review says all new protocol fixtures, including a ported Schwarzschild fixture, should move to `phys-lib/gr`. plan9.1_qwen

The first half is correct:

```text
phys-gr
    = untouched historical/reference artifact

phys-lib/gr
    = canonical new GR implementation
```

But we subsequently **explicitly bounded Phase 1**:

```text
phys-lib/gr:
    FrameworkArtifact
    six framework families
    GR-local algebra
    equivariance fixture
    required tests

Schwarzschild:
    remains in untouched phys-gr/solution/
    deferred
```

So the Qwen review's “ported Schwarzschild” requirement is **stale**. The preserved `phys-gr` solution is a semantic/reference anchor, not a Phase-1 migration target.

The useful surviving point is simply:

> The lifecycle of `phys-gr` must be explicit, and the new canonical GR work belongs in `phys-lib/gr`.

That is already captured in the current plan.

### 2. GRExpr disjointness: definitely valid

This is a strong finding.

The review warns against creating a common interface or dynamically extending PhysMath to absorb GR-local `Sin`, `Cos`, and `UnknownFunction`. plan9.1_qwen

Keep:

```text
phys-lib/gr GRExpr
        ≠
phys-math Expression
```

with an explicit fallible projection of only the representable subset.

This is completely consistent with our B1 decision and current D9. It is not merely a coding preference; it protects the core/specific boundary.

### 3. Equivariance ParameterIDs: definitely valid

The review's warning about:

```go
NewSymbol("g")
NewSymbol("x")
```

is correct. plan9.1_qwen

The fixture must use:

```text
rho_X
rho_Y
f
```

as Function artifacts and derive the relevant symbols from their declared parameter slots.

This remains a hard determinism requirement.

### 4. Tensor boundary: valid, but we should make it stricter

The review correctly says GR concepts such as:

```text
ChartID
metric signature
connection semantics
GR-specific tensor meaning
```

must not leak into PhysMath. plan9.1_qwen

However, I would **not** carry forward its suggestion that PhysMath should necessarily have generic tensor contraction operations merely because they are mathematically generic.

Our later closure is tighter:

```text
Phase-1 phys-math Tensor:
    structure
    rank
    dimension
    index variance
    dense representation
    validation
    canonical identity

Phase-1 GR:
    actual GR tensor computation
    contractions
    connection semantics
    metric semantics
    chart compatibility
```

This is supported by the Phase-1 rule lock: the only executable PhysMath transformations are the seven registered rule pairs. 

So the **boundary finding is valid; the operational scope must remain constrained.**

### 5. The MathCondition bridge in this review is the biggest technical error

The review states:

> `phys-math` encounters `Pow(r,0)`, fails internally, and returns a `TransformResult` containing a new `MathCondition`. plan9.1_qwen

That is **not the final architecture**.

The normative rule is:

```text
Pow(x,0) -> 1
```

only when an explicit condition:

```text
NEQ(x,0)
```

is already supplied. Without it, the transformation **fails**. Phase-1 rules do not synthesize a new mathematical condition. 

The correct flow is:

```text
1. Construct durable physmath/condition/1
       │
       ▼
2. OperationInvocation.ConditionRefs
       │
       ▼
3. EXPR-SIMPLIFY-001 checks exact predicate
       │
       ├── present → transformation succeeds
       └── absent  → fail closed
       │
       ▼
4. phys-lib/gr may reference that existing condition
   from a PhysicalAssumption
```

So **do not adopt Qwen's “rule generates MathCondition” design.**

This distinction matters because it preserves the clean separation between:

```text
rule execution
    ≠
condition creation
```

and prevents PhysMath from becoming an implicit theorem/assumption generator.

---

## One more useful point from the review

Its Phase-4 warning that an actually empty Go directory isn't a real package is valid. plan9.1_qwen

We already resolved this more cleanly:

```text
phys-lib/core/doc.go
```

with:

```go
package core
```

and package documentation, but no exported domain API.

That is the correct implementation of “empty core”: **architecturally empty, but syntactically a real Go package.**

---

# Bottom line

I would classify the attached review this way:

```text
VALID AND SHOULD REMAIN
────────────────────────────────────
GR / phys-lib/gr lifecycle boundary
GRExpr disjointness
Function-derived equivariance ParameterIDs
GR vs mathematical tensor semantic boundary
real Go package stub for empty core

VALID, BUT TIGHTENED
────────────────────────────────────
Phase-1 tensor scope = representation/validation,
not generic tensor computation

STALE / SUPERSEDED
────────────────────────────────────
"ported Schwarzschild" as a Phase-1 deliverable

REJECT
────────────────────────────────────
Phase-1 rule generates a new MathCondition
```

So **there is no new architectural blocker from this Qwen review** beyond points that we have already resolved. Its most valuable contribution is the GRExpr disjointness and ParameterID warnings. Its MathCondition mechanism should explicitly **not** be incorporated. plan9.1_qwen

The next plan revision should therefore preserve the valid findings above but should **not reopen any of the decisions we already locked**.


## Review

Yes. At this point I would make **9.3 the last planning revision**. The goal should not be to reach an impossible state of “nothing whatsoever can ever be questioned.” The line should be:

> **All known architectural decisions are closed in 9.3. The coding agent must now implement. If it encounters a genuinely new architectural ambiguity, it must stop only for that affected decision, ask focused clarification questions first, and then continue.**

That is consistent with the existing stop-rule discipline in the specification and with the reviews: the remaining problems in 9.2 are closure/pinning issues, not a need for another redesign. PHYSMATH_PHYS_PHYSSTAR_REPRESEN… plan9.2_deepseek plan9.2_z

Use this prompt:

```text
TASK: Produce Implementation Plan 9.3 and then proceed to implementation.

You are the senior coding/implementation agent.

This is the FINAL planning revision.
We do NOT want another endless review cycle.

Your first deliverable is:
    plan9.3.md

After plan9.3 is complete, you are EXPECTED TO BEGIN IMPLEMENTATION.

The purpose of 9.3 is NOT to redesign the architecture or discover more theoretical perfection.
Its purpose is to absorb the remaining validated findings from the consolidated adversarial reviews,
remove the last implementation-critical ambiguities, and establish a clean handoff to execution.

============================================================
1. AUTHORITATIVE SOURCES
============================================================

Use, in this order:

1. PM-REP-001 v1.2 — authoritative specification.
2. The actual existing GR protocol implementation in:
       /home/chaschel/Documents/go/phys-gr
3. Current plan:
       plan9.2.md
4. Consolidated/adversarial reviews of plan9.2:
       plan9.2_deepseek.md
       plan9.2_gemini.md
       plan9.2_qwen.md
       plan9.2_z.md

IMPORTANT:
The attached `plan9.2_qwen.md` contains useful findings only where they concern
the actual phys-math / phys-lib architecture.
Ignore any unrelated material referring to another architecture or project,
including Session/MRC/E=mc²/39-file/compiler architecture/etc.
Do not import unrelated architectural concepts from that review.

Do not treat a review as more authoritative than PM-REP-001.
When a review conflicts with the specification, resolve against the specification
and record the resolution explicitly.

============================================================
2. PRIMARY OBJECTIVE
============================================================

Produce:

    plan9.3.md

with:

    ZERO KNOWN ARCHITECTURAL BLOCKERS
    ZERO KNOWN IMPLEMENTATION-CRITICAL AMBIGUITIES

Then START IMPLEMENTATION.

Do not create another planning revision unless the architect explicitly requests one.

9.3 must be a closure-and-execution document, not another exploratory review.

The plan must distinguish:

    ALREADY DECIDED
        implementation must follow

    DEFERRED
        explicitly outside current scope

    PERMITTED IMPLEMENTATION DETAIL
        coding agent may decide without asking

    NEW ARCHITECTURAL BLOCKER
        coding agent must ask the architect before implementing the affected behavior

The coding agent MUST NOT invent architecture.

============================================================
3. HARD STOP / CLARIFICATION POLICY
============================================================

This is the most important process rule.

After plan9.3 is written, DO NOT stop implementation merely because
a minor detail is not explicitly named.

Implementation may proceed on all decisions covered by the architecture,
specification, and plan.

ASK THE ARCHITECT ONLY when an encountered issue changes one or more of:

    ownership
    dependency direction
    canonical identity
    canonical serialization
    durable schema
    artifact family
    physical semantics
    PhysMath vocabulary
    rule semantics
    protocol semantics
    framework boundary
    Phase-1 scope
    preservation boundary
    core/specific classification

A mere Go implementation choice is NOT an architectural blocker.

Examples that must NOT trigger a clarification request:

    file naming
    unexported helper naming
    constructor naming
    internal data structures
    test helper organization
    error variable naming where semantics are unchanged
    CI shell mechanics
    local fixture organization
    caching
    allocation strategy
    benchmark structure

When a genuine architectural blocker appears:

1. STOP implementation of the affected behavior.
2. Provide a concise clarification request containing:
       - problem
       - exact PM-REP / plan section
       - concrete repository evidence
       - competing interpretations
       - consequence of each
       - recommended resolution when defensible
3. Do NOT silently choose.
4. Do NOT broaden the blocker into unrelated work.
5. Continue implementing unaffected portions where safe.
6. Once the clarification is answered, resume implementation.

Do not ask repeated questions for decisions already fixed below.

============================================================
4. LOCKED REPOSITORY TOPOLOGY
============================================================

Workspace:

    /home/chaschel/Documents/go/

Repositories:

    phys/
    phys-gr/
    phys-artifact/
    phys-math/
    phys-lib/

Modules:

    github.com/PithomLabs/phys-artifact
    github.com/PithomLabs/phys-math
    github.com/PithomLabs/phys-lib

`phys-lib` is ONE Go module.

Packages include:

    phys-lib/core
    phys-lib/gr
    phys-lib/internal/conformance/fixture2

Do NOT create separate modules for:
    gr
    qm
    bohm
    cst
    cdt

============================================================
5. ABSOLUTE PRESERVATION RULE
============================================================

Existing:

    /home/chaschel/Documents/go/phys-gr

is a HISTORICAL ARTIFACT AND READ-ONLY SOURCE.

NEVER:

    modify it
    move it
    rename it
    delete it
    truncate it
    refactor it in place
    add compatibility code to it
    change its architecture

Refactoring means:

    phys-gr (read-only source)
        ->
    new canonical implementation in phys-lib/gr

The legacy repository remains physically present after migration.

No new canonical module may import:

    github.com/PithomLabs/phys-gr

There is NO shim exception.

Legacy validation is through copied/re-authored regression vectors
and behavioral comparison, never through imports.

============================================================
6. FROZEN PHYS
============================================================

`phys/` is frozen.

Phase 1 canonical modules do NOT import it.

Do not modify its source, API, or dependency graph.

Do not copy its namespace/artifact references into new Phase-1 artifacts.

`phys-gr` may retain its historical internal:

    replace => ../phys

because `phys-gr` is preserved untouched.

This does NOT authorize that replace directive in any new module.

New modules contain no `replace` directives.

============================================================
7. FINAL DEPENDENCY MODEL
============================================================

Permitted partial order:

    phys-lib/<framework>
        ->
    phys-lib/core
        ->
    phys-math
        ->
    phys-artifact

BUT IMPORTANT:

These arrows describe the allowed dependency order, not mandatory imports.

In particular:

    phys-lib/gr
        MAY use phys-math directly

and:

    phys-lib/gr
        MUST NOT be forced to import phys-lib/core

while core is empty.

`phys-lib/core` is an optional lower layer that only becomes a dependency
when a real abstraction has been promoted there.

Do NOT create a fake core abstraction merely to make the dependency graph
look vertically complete.

Prohibited:

    phys-artifact -> anything
    phys-math -> phys-lib/*
    phys-lib/core -> framework packages
    framework package -> another framework package by Go import
    new module -> phys-gr
    Phase-1 new module -> frozen phys

`BaseFrameworkRefs` is ARTIFACT/DATA dependency only.
It never authorizes a Go import between framework packages.

============================================================
8. PHYS-LIB/CORE
============================================================

`phys-lib/core` starts EMPTY.

In Phase 1 it is IN-MEMORY ONLY.

It publishes:

    no FrameworkArtifact
    no durable framework family
    no new ArtifactRef-bearing schema
    no semantic artifact namespace

The directory MUST be a real Go package:

    phys-lib/core/doc.go

with:

    package core

and package documentation only.

No exported placeholder type.
No fake `Common`.
No `Util`.
No `Shared`.
No generic physical interface.

Phase 4 is:

    module scaffolding
    core/doc.go
    classification mechanism

NOT:

    speculative abstraction extraction

Core promotion is evidence-based.

============================================================
9. CORE / SPECIFIC CLASSIFICATION PROTOCOL
============================================================

Every material `phys-gr` abstraction must receive exactly one classification:

    PHYS-MATH
    PHYS-LIB/CORE
    PHYS-LIB/GR
    SPLIT
    REPLACE
    DELETE
    DEFER

A classification record is required for every refactoring-map row.

Record location:

    phys-lib/classifications/

Naming:

    stable slug derived from the source component

Example:

    phys-lib/classifications/symbolic-expr.md

Each record uses this fixed structure:

    ---
    id: ...
    source: ...
    class: ...
    target: ...
    status: adopted
    ---

    Why it is theory-agnostic:
    Known/current consumers:
    Potential future consumers:
    Theory-specific assumptions:
    Stable interface:
    Dependencies:
    Why it does not belong in phys-math:
    Why it does not belong in phys-lib/gr:

Rules:

    PHYS-LIB/CORE
        requires full evidence record

    SPLIT
        requires exact ownership/interface boundary

    PHYS-MATH
    PHYS-LIB/GR
    REPLACE
    DELETE
    DEFER
        require classification and destination rationale

CI must verify:

    every refactoring-map row has a classification record
    every PHYS-LIB/CORE record contains the required evidence fields

The records are documentation/governance artifacts.
They are NOT exported API and are NOT part of the core package.

Ambiguous candidates default to:

    phys-lib/gr

The principle is:

    coupling
        separate conflated concerns

    cohesion
        unify only through the smallest stable interface

    core/specific
        move only the true common denominator downward
        keep the differences outward

============================================================
10. PHYS-ARTIFACT
============================================================

Strictly mechanical.

Own:

    ArtifactRef
    digest framing
    primitive encoding
    EmbeddedArtifactRef
    String
    Bytes
    UInt8/16/32/64
    Bool
    Digest32
    Optional
    Sequence
    Set
    Enum
    BigInt
    grammar primitives

Do NOT put math or physical semantics here.

No:
    Expression
    Function
    Tensor
    Theorem
    Derivation
    Framework
    Assumption
    physical concepts
    generic utility dumping ground

Boundary testing is semantic/mechanical.
Do not use naive substring traps.

============================================================
11. PHYS-MATH PURITY
============================================================

`phys-math` contains pure mathematical representation and execution only.

No:
    GR
    QM
    physical assumptions
    physical interpretations
    theory names
    physical dimensions as physical meaning
    metric conventions
    physical ontology
    physical semantics

The Phase-1 durable registry is closed.

Do NOT invent families.

The 16 registered mathematical families remain the governing registry.

Deferred sort concepts do not gain invented durable families.

============================================================
12. EXACT AST TAG MAPPING
============================================================

Pin this table explicitly in plan9.3:

    0x01 Symbol
    0x02 Rational
    0x03 Add
    0x04 Mul
    0x05 Neg
    0x06 Pow
    0x07 Sqrt
    0x08 Call
    0x09 Relation
    0x0A BranchSet

Unknown tags MUST be rejected.

No framework may inject additional AST nodes.

============================================================
13. GR-LOCAL ALGEBRA
============================================================

Adopt B1:

    Sin
    Cos
    UnknownFunction shape
    GR-specific trig simplification
    GR regime-aware zero testing
    GR derivative behavior
    other GR-local symbolic rules

remain in:

    phys-lib/gr

`phys-math` remains a closed 10-node AST.

`GRExpr` is a DISJOINT Go type from `phys-math.Expression`.

Do NOT create:
    shared interface
    dynamic tagged union
    common expression interface

Only an explicit deterministic projection of the representable subset is permitted.

Non-representable GR-local expressions MUST fail explicitly.

DO NOT invent synthetic function identifiers such as:

    phys-gr:func/sin
    phys-gr:func/cos

merely to force GR-local trig into PhysMath.

If a GR-local expression cannot be represented by an authorized PhysMath family,
keep it GR-local and fail projection when projection is requested.

============================================================
14. PHASE-1 GR SCOPE — FINAL PIN
============================================================

This is a critical closure rule.

Phase-1 `phys-lib/gr` consists of:

    FrameworkArtifact
    six framework-owned physical families
    GR assumptions/interpretations required by those families
    GR-local algebra
    representable PhysMath fragments
    equivariance fixture
    required framework/conformance tests

Phase 1 DOES NOT re-implement:

    Christoffel computation pipeline
    Ricci computation pipeline
    Einstein/vacuum computation pipeline
    weak-field computation pipeline
    complete Schwarzschild protocol
    SessionCorrespondence
    frozen-Phys integration

The full Schwarzschild executable protocol remains in:

    phys-gr/solution/

unchanged and untouched.

Its future re-expression is deferred.

Correct the refactoring map wording:

WRONG:
    "formula loops re-expressed on phys-math"

RIGHT:
    "PhysMath-expressible fragments project to PhysMath;
     GR computational pipeline remains GR-local and Phase-1 deferred."

The phrase "ported GR vectors" MUST NOT imply that the full GR computation
pipeline has been reimplemented.

============================================================
15. PORTED GOLDEN VECTOR DEFINITION
============================================================

Define “ported golden vector” precisely.

A ported vector is a semantic regression fixture derived from the immutable
legacy `phys-gr` behavior, but expressed under the NEW canonical architecture.

Legacy bytes are NOT assumed to equal new canonical bytes.

Legacy `phys-gr` serialization is source evidence.
PM-REP canonical serialization is the new oracle.

Three categories:

    A. PhysMath-expressible content
       -> re-encode under phys-math canonical representation
       -> pin new canonical bytes / ArtifactRef

    B. GR-local content
       -> test package-local GR semantics directly
       -> no invented PhysMath family

    C. Non-representable projection
       -> explicit projection-success/failure test

Source evidence for GR regression includes the legacy:

    symbolic/
    coordinates/
    tensor/
    metric/
    connection/
    curvature/
    vacuum/
    limit/

The new regression fixtures MUST NOT import phys-gr.

Phase-5 exit criteria MUST specify exactly which vectors are required.

At minimum include:

    framework artifact serialization
    GR physical-family validation
    GR-local algebra behavior
    representable projection success
    non-representable projection failure

Do NOT describe these as byte-for-byte copies of legacy `phys-gr` encodings.

============================================================
16. B2 CONDITION MODEL
============================================================

Adopt B2:

framework-level physical regularity/non-vanishing assumptions belong in:

    PhysicalAssumption

Examples:

    r != 0
    A(r) != 0
    sin(theta) != 0

But PhysMath rule execution does NOT invent durable conditions.

For:

    Pow(x,0) -> 1

the exact condition:

    NEQ(x, 0)

must already exist as the supplied mathematical condition context.

Flow:

    durable physmath/condition/1 exists
        ->
    OperationInvocation.ConditionRefs supplies its ArtifactRef
        ->
    rule checks exact predicate
        ->
    success OR fail closed
        ->
    framework may reference the existing condition
       from PhysicalAssumption

No silent discharge.

No Phase-1 rule creates a new durable MathCondition.

============================================================
17. OPERATION INVOCATION
============================================================

Use the exact PM-REP operation contract.

Important:

    ConditionRefs do NOT count toward rule input arity.

`TransformResult` is in-memory only and immutable after construction.

Phase-1 rules do not generate new mathematical conditions.

Therefore:

    TransformResult.ConditionRefs
        = normalized supplied condition set

unless PM-REP is later changed by an explicit specification amendment.

No `OperationResult`.

No `TraceRef` in TransformResult.

============================================================
18. RULE REGISTRY
============================================================

Only these seven Phase-1 rule pairs exist:

    (physmath, EXPR-CANON-001)
    (physmath, EXPR-SIMPLIFY-001)
    (physmath, EXPR-SUBSTITUTE-001)
    (physmath, RAT-NORMALIZE-001)
    (physmath, DIFFERENTIATE-001)
    (physmath, SELECT-BRANCH-001)
    (physmath, EXPR-APPLY-001)

Canonical rule identity is:

    (ProducerNamespace, RuleID)

A colon-joined form may exist only for:
    transport
    diagnostics
    error keys

It is NOT canonical identity.

Unknown pairs fail closed.

OperatorApplication:
    OperatorRef absent in Phase 1
    OperatorID exactly as specified by PM-REP
    no invented operator family

============================================================
19. DIFFERENTIATE
============================================================

Do NOT adopt the reviewer-invented chain-rule/Derivative-node design.

Call differentiation remains out of scope for the Phase-1 core runtime.

Do NOT introduce:
    Derivative AST node
    Derivative artifact
    hidden derivative operator
    automatic theory-specific derivative knowledge

Implement exactly the closed Phase-1 derivative rules from PM-REP.

============================================================
20. SUBSTITUTE
============================================================

Use the exact PM-REP rule.

For `EXPR-SUBSTITUTE-001`:

    Input[0] = target expression
    Inputs[1..D] = declaring Function artifacts

Declaring Function refs:
    sorted by canonical ArtifactRef bytes

Parameter entries:
    sorted lexicographically by exact canonical entry bytes

Targets:
    must resolve to their declaring Functions

CRITICAL:

    No two substitution entries may share the same ParameterID,
    REGARDLESS OF DeclaringArtifactRef.

Matching is by ParameterID.

Do not infer declaration context from:
    DisplayName
    repository lookup
    replacement expression

============================================================
21. PHASE-1 MATH FAMILY IMPLEMENTATION
============================================================

Phase 2 must explicitly account for every Phase-1 registered PhysMath durable family,
including at minimum:

    expression
    function
    function_application
    map
    tensor
    index
    collection
    operator_application
    transformation
    relation
    equation
    equation_set
    definition
    condition
    derivation
    theorem

Clearly distinguish:

    durable family implemented
    in-memory-only/deferred sort
    operational rule support

Do not accidentally omit FunctionApplication or Transformation
from the implementation inventory.

============================================================
22. TENSOR BOUNDARY
============================================================

PhysMath Tensor owns mathematical structure only:

    Rank
    Dimension
    IndexVariance
    dense representation
    shape validation
    canonical representation

Phase-1 PhysMath MUST NOT invent generic tensor transformation rules
outside the seven-rule execution lock.

GR owns:

    ChartID
    metric signature
    GR symmetry meaning
    connection semantics
    GR contraction semantics
    physical validation

Do not move GR tensor semantics into PhysMath.

============================================================
23. NAMESPACE SEPARATION
============================================================

Do NOT conflate:

    Go module identity
    durable artifact namespace

Keep:

    Go module:
        github.com/PithomLabs/phys-math

    Durable PhysMath namespace:
        physmath

Keep:

    Go package:
        github.com/PithomLabs/phys-lib/gr

    Durable GR namespace:
        phys-gr

Fixture2 durable namespace:

    phys-fixture2

These are pinned and deterministic.

No global namespace registry is required.

============================================================
24. GR PHYSICAL ASSUMPTION CATEGORY
============================================================

PM-REP leaves Category as a framework-owned string.

For Phase 1, `phys-lib/gr` chooses to constrain its own Category vocabulary to:

    convention
    regularity
    model

This is a GR package-contract decision only.

It is NOT a global restriction on all future phys-* frameworks.

Another framework may define different Category values without violating PM-REP.

No cross-framework validator may reject another framework merely because
its Category differs from the GR vocabulary.

Do not amend PM-REP merely to support this GR-local choice.

============================================================
25. FRAMEWORK ARTIFACT
============================================================

GR FrameworkArtifact:

    PackageNamespace = "phys-gr"
    BaseFrameworkRefs = empty in Phase 1

fixture2:

    namespace = "phys-fixture2"
    fully conforming minimal framework
    valid FrameworkArtifact
    all six framework-owned families implemented minimally

BaseFrameworkRefs are artifact dependencies only.

Framework packages do not import one another by Go in Phase 1.

============================================================
26. FRAMEWORK VERSION
============================================================

FrameworkVersion is a non-empty opaque package-owned string.

The exact reference-fixture value MUST be pinned.

Use:

    GR fixture version = "1"
    fixture2 version = "1"

unless the authoritative specification explicitly dictates a different value.

Do not invent semantic version ontology inside PhysMath.

============================================================
27. EQUIVARIANCE FIXTURE
============================================================

Canonical location:

    phys-lib/gr/equivariance/

with:

    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go

Use actual GR protocol semantics and loading conventions as precedent.

Do NOT invent a second protocol format.

Canonical mathematical realization:

    f(v)       = 2 * v
    rho_X(g,v) = g * v
    rho_Y(g,v) = g * v

Canonical initial structure:

    LHS_0 = Call(f, [Call(rho_X, [g, x])])
    RHS_0 = Call(rho_Y, [g, Call(f, [x])])

Staged Calls are mandatory because EXPR-APPLY is root-Call-only.

Exact six transformations:

    Step 0 APPLY rho_X
    Step 1 APPLY f
    Step 2 CANON
    Step 3 APPLY f
    Step 4 APPLY rho_Y
    Step 5 CANON

Final canonical expression bytes MUST match.

Relation construction is deterministic packaging from those final expression bytes.
It is NOT a new theorem-proving rule.

`rho_X` and `rho_Y` may be canonically identical in the minimal fixture.
That is intentional and MUST be documented.

============================================================
28. EQUIVARIANCE PARAMETER BINDING
============================================================

Function artifacts are mandatory for:

    f
    rho_X
    rho_Y

The schematic variables must use the exact Function-derived ParameterIDs.

No:

    NewSymbol("g")
    NewSymbol("x")

for durable identities.

Pin display names:

    f slot      = "v"
    rho_X slots = "g", "v"
    rho_Y slots = "g", "v"
    schematic   = "g", "x"

DisplayName participates in canonical Symbol identity even though it is not
mathematical meaning.

Therefore fixture display names are golden-vector data.

============================================================
29. FIXTURE2
============================================================

Location:

    phys-lib/internal/conformance/fixture2/

Durable namespace:

    phys-fixture2

It is:

    test-only
    not a production framework package
    not imported by production code
    not imported by core
    not imported by gr

It is nevertheless a FULLY CONFORMING MINIMAL framework:

    FrameworkArtifact
    assumption family
    definition family
    equation family
    theorem family
    interpretation family
    minimal content for each

Use exact namespace isolation tests.

Prove:

    PhysMath identity unchanged
    framework identities distinct
    assumptions distinct
    interpretations distinct
    no global winner
    invalid cross-framework references rejected
    self/cyclic dependencies rejected

============================================================
30. PROTOCOL AUTHORITY
============================================================

The actual existing GR protocol is authoritative precedent.

Inspect and reuse the actual:

    solution/manifest.json
    solution/manifest_test.go
    solution/derivation_test.go
    AGENTS.md
    replay/

Do NOT reconstruct the protocol from memory.

Reuse the actual schema/loading/execution semantics.

The new nested fixture is a refactoring/re-expression of the protocol pattern,
NOT a replacement of the historical `phys-gr/solution/`.

If a required equivariance structure cannot be expressed under the authorized protocol:

    STOP
    classify as PROTOCOL GAP
    ask for clarification

Do NOT invent:
    new manifest fields
    alternate schema
    alternate protocol format

============================================================
31. GOLDEN / CANONICAL IDENTITY
============================================================

Canonical binary encoding, not JSON, defines identity.

Keep the PM-REP primitive framing exactly.

BigInt zero:

    sign = 0x00
    magnitude length = UInt32BE(0)
    no magnitude bytes

AST tags are the fixed mapping from §12.

ParameterID:

    SHA-256(
        String("physmath")
        ||
        Bytes(ScopePathBytes)
        ||
        UInt32BE(ParameterOrdinal)
    )

Function parameters use:

    depth = 0
    ScopePathBytes = UInt32BE(0)

BoundParameterRef:

    EncodeArtifactRef(DeclaringArtifactRef)
    ||
    Digest32(ParameterID)

No extra Bytes() wrapper around ParameterID.

============================================================
32. RELATION
============================================================

Keep both:

    Expression Relation node
    durable physmath/relation/1 artifact

Expression node:

    0x09
    || Enum(operator)
    || CanonicalExpression(left)
    || CanonicalExpression(right)

Durable relation payload:

    Enum(operator)
    || CanonicalExpression(left)
    || CanonicalExpression(right)

No reference-indirection alternative.

Theorem conclusion is a durable relation whose embedded expression bytes
equal the final derivation-result bytes.

No hidden equality inference.

============================================================
33. EQUATIONSET
============================================================

Phase-1 EquationSet ordering is:

    SEQUENCE

with uniqueness validation.

Do not reinterpret it as a SET merely because its elements are equations.

============================================================
34. EXACT PROTOCOL / MANIFEST HANDLING
============================================================

The canonical fixture package must use:

    AGENTS.md
    manifest.json
    protocol/derivation_test.go

The manifest is metadata/protocol configuration.

It must not become a hidden mathematical premise.

Use:
    strict decoding
    unknown-field rejection
    artifact presence checks
    ordered execution
    expected-value assertions
    fixture firewalls
    mutation tests
    no-skip checks

Schema semantics must remain consistent with the actual GR precedent.

============================================================
35. PHASE ORDER
============================================================

9.3 must retain a concrete execution sequence:

Phase 0
    authority
    repository inspection
    immutable tree hashes
    workspace verification

Phase 1
    phys-artifact

Phase 2
    phys-math representation + identity

Phase 3
    phys-math rules + derivation/replay

Phase 4
    phys-lib module
    core/doc.go
    classification-record mechanism
    NO speculative core extraction

Phase 5
    phys-lib/gr
    GR framework families
    GR-local algebra
    PhysMath projection
    assumption bridge
    classification execution
    semantic ported vectors

Phase 6
    canonical GR/equivariance protocol package

Phase 7
    equivariance six-step slice

Phase 8
    fixture2

Phase 9
    full vectors
    negatives
    contamination tests
    deterministic replay
    classification audit
    protocol audit

Phase 10
    migration/conformance gate
    remove phys-gr from go.work ONLY when gate passes
    verify legacy phys-gr tree unchanged
    verify frozen phys unchanged

Do not reorder phases merely to make the plan shorter.

============================================================
36. IMPLEMENTATION VS LEGACY VALIDATION
============================================================

Legacy `phys-gr` behavior is REFERENCE EVIDENCE.

The new architecture is the IMPLEMENTATION TARGET.

Do not preserve legacy coupling simply because it already exists.

Do not reproduce legacy serialization merely for compatibility if it conflicts
with canonical PM-REP representation.

When old and new representations differ:

    compare semantic behavior
    compare intended represented values
    re-encode using new canonical rules

Do not compare incompatible serialization formats byte-for-byte.

============================================================
37. QUALITY GATES
============================================================

Plan9.3 must include mechanical and review gates.

Mechanical examples:

    go list
    go mod graph
    import scanning
    tree hash verification
    vocabulary checks
    canonical-byte tests
    golden ArtifactRefs
    rule closure
    target-family validation
    deterministic replay
    protocol schema checks

Architectural review criteria:

    mathematician test
    core/specific reasoning
    physical-semantics ownership
    cohesion/coupling assessment
    no convenience-driven promotion

Do not falsely claim a mechanical test proves what only architectural review can establish.

============================================================
38. FINAL HANDOFF LINE
============================================================

At the end of plan9.3 write:

    STATUS: CLOSED FOR IMPLEMENTATION

and:

    No known architectural blockers remain.
    No known implementation-critical architectural ambiguities remain.
    Implementation begins now.

Then the coding agent MUST move from planning to implementation.

============================================================
39. FINAL CLARIFICATION BOUNDARY
============================================================

After plan9.3, no further general review loop is required.

The agent should NOT return with:

    "I found another small ambiguity, therefore let's write plan9.4."

Instead:

    If it is an ordinary implementation detail:
        choose an idiomatic implementation and proceed.

    If it changes no architecture:
        proceed.

    If it reveals a genuine contradiction / missing semantic rule /
    ownership conflict / canonical-identity conflict / protocol gap:
        stop ONLY the affected behavior,
        ask a focused architectural clarification question,
        resume after the answer.

This is the final decision boundary.

============================================================
40. REQUIRED DELIVERABLE
============================================================

Produce `plan9.3.md`.

Do not produce source code as the planning deliverable.

Do not create plan9.4.

Do not reopen:
    topology
    phys-math purity
    phys-lib ownership
    phys-gr preservation
    empty-core principle
    B1
    B2
    equivariance representation
    fixture2 namespace
    protocol precedent
    seven-rule registry
    frozen Phys boundary

The plan must include:

    executive status
    adopted decisions
    deferred decisions
    exact repository topology
    exact dependency rules
    core/specific classification protocol
    complete refactoring map
    exact Phase-5 GR scope
    exact ported-vector definition
    exact condition lifecycle
    exact equivariance fixture
    exact fixture2 conformance model
    exact protocol migration
    acceptance/test matrix
    migration gates
    risks/failure modes
    implementation sequence
    clarification/stop policy
    final handoff checklist

FINAL PRINCIPLE:

    Plan enough to remove architectural ambiguity.
    Do not plan forever.

    plan9.3 is the closure point.
    Then implement.

```

### Why these additions matter

The biggest correction is the **Phase-5 boundary**. Z correctly spotted that the current plan still has one foot in “reimplement the GR computation pipeline” while D7/F1 say that pipeline is deferred. That could otherwise cause the agent to hit an unavoidable representational wall halfway through implementation. plan9.2_z

The second major closure is the **ported-vector definition**. DeepSeek is right that “ported golden vector” is not sufficiently defined when the legacy representation and the new canonical PhysMath representation are structurally different. plan9.2_deepseek

The third is the **classification-record mechanism**. Without a concrete location and verification rule, “evidence-based core promotion” is an architectural principle but not an executable process. plan9.2_deepseek

And I intentionally corrected the Qwen review's condition mechanism: PM-REP's rule contract says conditions are **supplied context**, and `Pow(x,0)` fails without the exact supplied `x != 0` condition; Phase-1 rules do not silently manufacture a new durable condition. PHYSMATH_PHYS_PHYSSTAR_REPRESEN… PHYSMATH_PHYS_PHYSSTAR_REPRESEN…

That gives us a sensible stopping line: **9.3 closes the known architecture; implementation starts immediately afterward; only a genuinely new architectural contradiction triggers a focused clarification question.** This preserves the no-invention discipline without turning the project into an endless planning exercise.