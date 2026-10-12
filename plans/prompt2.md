Use this as the revised coding-agent prompt. It assumes the agent already has **PM-REP-001 v1.2** and the existing `phys-gr` repository, so it does not repeat the specification.

---

# Coding Agent Prompt — Revise Implementation Plan for `phys-math` + `phys-lib`

You are the senior implementation architect and coding agent for the PhysMath / `phys-lib` work.

You have already produced an implementation plan for the existing `phys-gr` repository. **Do not start coding yet.** Revise that implementation plan to incorporate the architectural requirements below, then stop for architectural input only if a genuine blocker remains.

## 1. Authoritative architecture

Treat **PM-REP-001 v1.2** as the authoritative specification.

Do not restate the specification.

Do not redesign settled decisions.

The implementation plan must now target these canonical Go library boundaries:

```text
github.com/PithomLabs/phys-artifact
        ↓
github.com/PithomLabs/phys-math
        ↓
github.com/PithomLabs/phys-lib
```

`phys-lib` contains the physical-domain libraries:

```text
github.com/PithomLabs/phys-lib/core
github.com/PithomLabs/phys-lib/gr
github.com/PithomLabs/phys-lib/qm
github.com/PithomLabs/phys-lib/bohm
github.com/PithomLabs/phys-lib/cst
github.com/PithomLabs/phys-lib/cdt
...
```

The exact package decomposition inside `phys-lib/core` and the specific framework packages must be derived from the existing implementation and the architecture below; do not invent a taxonomy merely for symmetry.

## 2. Canonical dependency direction

The target dependency direction is:

```text
phys-lib/<specific>
        ↓
phys-lib/core
        ↓
phys-math
        ↓
phys-artifact
```

Permitted framework-to-framework relationships must follow the specification's explicit framework dependency mechanism.

The following are prohibited:

```text
phys-artifact → phys-math
phys-artifact → phys-lib/*

phys-math → phys-lib/*
phys-math → phys-lib/core

phys-lib/core → phys-lib/gr
phys-lib/core → phys-lib/qm
phys-lib/core → any specific framework

phys-lib/gr → phys-lib/qm
phys-lib/qm → phys-lib/gr
```

unless an explicit framework dependency mechanism in the specification requires such a relationship.

Do not use package structure to hide prohibited dependencies.

## 3. Module structure

Use a multi-module repository/workspace:

```text
workspace/
    phys-artifact/
    phys-math/
    phys-lib/
    go.work
```

`phys-lib` should normally be **one Go module** containing focused packages rather than one Go module per theory/framework.

Use:

```text
github.com/PithomLabs/phys-artifact
github.com/PithomLabs/phys-math
github.com/PithomLabs/phys-lib
```

Do not create a separate module for `phys-lib/gr`, `phys-lib/qm`, etc. unless repository evidence demonstrates a compelling architectural requirement.

The existing local `replace => ../phys` in `phys-gr` is transitional only. Establish the workspace wiring first, verify the frozen Phys dependency resolves correctly, then remove the committed local replacement as part of migration. Do not modify frozen Phys.

## 4. PhysMath boundary — absolute

`github.com/PithomLabs/phys-math` must contain **pure mathematical objects and mathematical representation**.

It MUST NOT contain:

- physical objects
- physical assumptions
- physical interpretations
- theory/framework names
- GR semantics
- QM semantics
- physical dimensions as physical meaning
- physical conventions
- trust semantics
- corpus/provenance semantics
- theory-specific transformations

A PhysMath API must be explainable without naming a physical theory.

Apply this test:

> Could the abstraction be used unchanged by a mathematician who knows nothing about physics?

If not, it does not belong in PhysMath.

Do not leak physical semantics into:

- AST nodes
- canonical encoding
- identities
- transformation rules
- derivation representation
- mathematical equality
- mathematical constructors

## 5. `phys-lib/core` boundary — new mandatory layer

Introduce an explicit **theory-agnostic physical core** under:

```text
github.com/PithomLabs/phys-lib/core
```

This layer is for physical abstractions that are genuinely shared across multiple physical frameworks but are not pure mathematics.

Examples may eventually include reusable physical concepts or interfaces, but **do not assume an abstraction is core simply because `phys-gr` currently uses it**.

A capability belongs in `phys-lib/core` only when:

1. its physical meaning is independent of a particular theory/framework;
2. its semantics are stable across multiple reasonable physical frameworks;
3. the API does not encode theory-specific assumptions;
4. moving it into core reduces duplication or establishes a meaningful reusable interface;
5. it can be documented without making a specific theory normative.

If the abstraction depends on GR, QM, Bohmian mechanics, CDT, CST, etc., it belongs in the corresponding specific package.

## 6. Core/specific classification protocol

For **every significant existing `phys-gr` abstraction**, perform this classification:

```text
PHYS-MATH
PHYS-LIB/CORE
PHYS-LIB/GR
SPLIT
REPLACE
DELETE
DEFER
```

For every `SPLIT`, identify the exact interface between the common and specific halves.

For every promotion to `phys-lib/core`, record:

```text
Component:
Current location:
Candidate destination:
Why it is theory-agnostic:
Known/current consumers:
Potential future consumers:
Theory-specific assumptions:
Stable interface:
Dependencies:
Why it does not belong in phys-math:
Why it does not belong in phys-gr:
```

Do not promote code merely because it is technically reusable.

Do not create `common`, `util`, `shared`, or catch-all packages to avoid making this decision.

## 7. Architecture of Complexity protocol

Apply these three principles continuously:

### Coupling

If different concerns are conflated:

```text
separate them
```

Examples:

```text
mathematics vs physical semantics
physical core vs theory-specific semantics
canonical identity vs protocol packaging
formal representation vs execution
```

### Cohesion

If things that must be unified are separated:

```text
identify the smallest stable interface that should unify them
```

Do not create abstraction boundaries merely because two things happen to be in different directories.

### Core / specific split

Repeatedly ask:

```text
What is the smallest true common denominator?
```

Put that denominator in the lowest appropriate layer.

Push differences outward.

The implementation plan should explicitly use these principles when explaining significant refactoring decisions.

## 8. Go library design

Follow idiomatic Go library design.

Prefer:

```text
focused packages
small public APIs
clear ownership
minimal dependencies
stable interfaces
```

Avoid:

```text
generic util/common packages
god packages
unnecessary interfaces
premature abstraction
cyclic dependencies
packages organized only by implementation convenience
```

Package boundaries must correspond to meaningful concepts and ownership, not merely file movement.

A package should have a clear answer to:

> “What does this package own that its callers should not implement themselves?”

## 9. Existing `phys-gr` is the refactoring laboratory

The repository:

```text
/home/chaschel/Documents/go/phys-gr
```

is the real implementation substrate.

Do not treat `phys-gr` as the final architecture.

Use it to discover the true layering:

```text
existing phys-gr
       ↓
identify mathematical commonality
       ↓
phys-math

identify theory-agnostic physical commonality
       ↓
phys-lib/core

identify GR-specific semantics
       ↓
phys-lib/gr
```

The purpose of the refactor is not merely to move files.

It is to discover **what complexity belongs where**.

## 10. Existing GR protocol is authoritative precedent

The coding agent has already inspected the real repository.

Use the actual existing GR files as the protocol authority:

```text
solution/manifest.json
solution/manifest_test.go
solution/derivation_test.go
AGENTS.md
replay/
```

Do not invent a provisional protocol format.

Do not create a second semantic manifest design.

Reuse the actual GR schema/loading/execution conventions.

The new formalization package layout must follow the canonical v1.2 pattern:

```text
<pkg>/
    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go
```

while preserving the **actual GR precedent's semantic/loading behavior**.

Treat this as a packaging refactor, not as a new protocol.

If a required equivariance feature cannot be expressed using the real GR protocol conventions without inventing a new semantic field, stop and report:

```text
PROTOCOL GAP
```

rather than improvising.

## 11. Formalization protocol is constitutional

For all future theorem, hypothesis, derivation, or physics-paper formalization:

```text
AGENTS.md
manifest.json
protocol/derivation_test.go
```

is the executable formalization package.

Do not treat protocol packaging as documentation.

Do not equate:

```text
Markdown
LaTeX
README
JSON metadata
```

with formalization.

Formalization exists when the representation can be executed/replayed through the established protocol.

The implementation plan must preserve this property for the equivariance fixture and future physics work.

## 12. Existing GR versus target `phys-gr`

Do not describe the current GR implementation and the target architecture as if they are the same thing.

Maintain two explicit conceptual states:

```text
LEGACY GR IMPLEMENTATION
    evidence
    regression baseline
    migration source

TARGET GR LIBRARY
    github.com/PithomLabs/phys-lib/gr
    conforming phys-* implementation
```

Preserve meaningful GR behavior and regression coverage.

Do not preserve legacy directory structure or accidental architecture merely for compatibility.

## 13. GR-local mathematics

Existing GR-specific mathematical machinery such as:

```text
Sin / Cos
UnknownFunction
GR-specific tensor algebra
weak-field reduction
GR-specific differentiation restrictions
metric-specific algebra
```

must remain framework-local unless it is already representable using the closed PhysMath specification.

Do not expand PhysMath just because GR needs a capability.

Do not invent new PhysMath AST nodes.

If a GR capability is needed but not supported by PhysMath:

```text
keep it in phys-lib/gr
```

using the theory-local mechanism allowed by the specification.

## 14. Primary Phase-1 physical library

The primary physical owner remains:

```text
github.com/PithomLabs/phys-lib/gr
```

Do not invent `phys-equiv`.

The equivariance reference fixture should be owned by GR because the existing real-world implementation gives us the best substrate for validating the architecture.

The target should therefore be:

```text
phys-math
    +
phys-lib/core
    +
phys-lib/gr
    +
equivariance protocol fixture
```

## 15. Many-frameworks test

The architecture must still demonstrate the many-frameworks principle.

Do not create a synthetic production framework solely for this.

Instead, add a minimal **test-only second `phys-*` framework fixture** during conformance testing:

```text
phys-lib/<test-framework>
```

or an equivalent test fixture namespace/package permitted by the specification.

It must demonstrate:

```text
same or overlapping mathematical representation
+
different physical assumptions/interpretation
```

The second fixture is evidence for framework independence, not a new production product.

## 16. Refactoring map requirements

For the current `phys-gr` directories:

```text
symbolic/
tensor/
metric/
connection/
curvature/
vacuum/
limit/
coordinates/
replay/
solution/
```

produce a concrete migration table:

```text
CURRENT
TARGET
CLASSIFICATION
COMMON DENOMINATOR
SPECIFIC DIFFERENCE
TARGET INTERFACE
DEPENDENCIES
TEST IMPACT
MIGRATION SEQUENCE
```

Pay particular attention to:

```text
symbolic
tensor
replay
```

because these are likely to contain both common and GR-specific responsibilities.

## 17. `phys-artifact` boundary

Keep `phys-artifact` strictly mechanical:

```text
ArtifactRef
digest
canonical framing
primitive codecs
grammar validation
```

No mathematical vocabulary.

No physical vocabulary.

No protocol semantics.

No generic application utilities.

The implementation plan should include a boundary test that rejects accidental promotion of math/physics concepts into `phys-artifact`.

## 18. Required implementation-plan phases

Revise the plan around this dependency ordering:

```text
Phase 0
Repository / authority / protocol inventory
        ↓
Phase 1
phys-artifact
        ↓
Phase 2
phys-math representation + identity
        ↓
Phase 3
phys-math rules + derivation
        ↓
Phase 4
phys-lib/core candidate extraction
        ↓
Phase 5
phys-lib/gr extraction and refactoring
        ↓
Phase 6
GR protocol migration
        ↓
Phase 7
Equivariance vertical slice
        ↓
Phase 8
Second-framework conformance fixture
        ↓
Phase 9
Golden vectors / negatives / determinism
        ↓
Phase 10
Adapter cleanup + architecture conformance
```

You may combine phases where repository reality makes that cleaner, but preserve the dependency ordering.

## 19. Architecture decision matrix

Before coding, provide a concise decision table covering:

```text
Concern
Owner
Reason
Dependency direction
Evidence
```

At minimum cover:

```text
Artifact identity
Canonical serialization
Mathematical AST
Parameter identity
Mathematical transformations
Derivation
Theorem
Theory-agnostic physical concepts
GR semantics
GR-local mathematics
Protocol
Frozen Phys integration
```

## 20. Architectural blocker rule

During plan revision and implementation, do not silently invent architecture.

Stop only if a decision affects:

```text
ownership
core/specific boundary
module dependency direction
canonical identity
serialization
PhysMath vocabulary
phys-lib/core semantics
framework semantics
protocol semantics
Phase-1 scope
frozen Phys boundary
```

When blocked, report:

```text
ARCHITECTURAL BLOCKER

Decision needed:
...

Evidence:
<exact repository files/code>

Conflict with PM-REP-001 v1.2:
...

Why current architecture is insufficient:
...

Possible resolutions:
A. ...
B. ...

Recommended resolution:
...

Impact:
...
```

Ordinary Go implementation choices are not blockers.

## 21. Required final output

Return a revised implementation plan, not code.

The plan must include:

```text
1. Authority and scope
2. Repository/current-state inventory
3. Target module graph
4. phys-artifact boundary
5. phys-math boundary
6. phys-lib/core boundary
7. phys-lib/<specific> boundary
8. Core/specific classification protocol
9. Full phys-gr refactoring map
10. GR protocol migration strategy
11. Equivariance implementation plan
12. Second-framework conformance plan
13. Test and acceptance matrix
14. Compatibility/migration strategy
15. Complexity/risk register
16. Architectural blocker log
17. Deferred work
18. Final dependency-ordered execution sequence
```

## 22. Final quality gate

Before declaring the plan ready for coding, verify all of these:

```text
[ ] PhysMath contains no physical concepts.
[ ] phys-lib/core contains no theory-specific semantics.
[ ] phys-lib/gr owns GR-specific semantics.
[ ] No reverse dependency exists.
[ ] Core/specific decisions are explicitly justified.
[ ] No generic util/common dumping ground was introduced.
[ ] Existing GR repository was actually inspected.
[ ] Actual GR protocol files are reused as authority.
[ ] No provisional manifest schema was invented.
[ ] Equivariance remains owned by phys-lib/gr.
[ ] Second-framework conformance is included.
[ ] Frozen Phys is untouched.
[ ] Legacy replaces have an explicit migration/removal gate.
[ ] Unsupported GR mathematics is not forced into PhysMath.
[ ] Protocol formalization remains executable.
[ ] Genuine architectural blockers are surfaced rather than guessed.
```

The goal of this revision is **not to make the implementation plan longer**.

The goal is to make the architecture impossible to misunderstand:

```text
phys-artifact
    = mechanical foundation

phys-math
    = pure mathematical common denominator

phys-lib/core
    = theory-agnostic physical common denominator

phys-lib/gr
    = GR-specific physical semantics

protocol
    = executable formalization boundary
```

Use the existing `phys-gr` implementation to discover and validate these boundaries before writing code.