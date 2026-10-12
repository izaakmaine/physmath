## background

give prompt for phase 2, target is to derive equivariance theorem under phys-lib/bohm package, its workflow should be patterned after the phys-lib/gr package with all the insights we gained so far per our root prompt >> # AGENTS.md — workspace entry pointer

For work involving `phys-artifact`, `phys-math`, `phys-lib`, `phys`,
or cross-repository architecture, read `phys-system/AGENTS.md` first.

Package-local `AGENTS.md` files remain authoritative for local
implementation details and constraints. The frozen `phys/AGENTS.md`
governs the legacy system and is never modified by new-architecture work.


## prompt

Yes. Phase 2 should be a **real theory stress test**, not another toy fixture. The target is the Bohmian equivariance theorem, while retaining everything we learned from GR: semantic liveness, persisted-artifact verification, independent evidence, strict framework boundaries, and no convenience-driven expansion of `phys-math`.

The existing `phys-system/AGENTS.md` now explicitly requires agents to map **claim → representation → derivation → conclusion**, test semantic liveness, and ask what incorrect implementation could still pass the tests. AGENTS It also explicitly warns that passing replay does not by itself prove the intended proposition and requires semantic witnesses and adversarial mutation. AGENTS

The Bohmian source material gives us a particularly good target: the equivariance result is conditional on the current/guidance law and continuity equations, and explicitly does **not** by itself derive the initial equality/Born rule. v9.2_math v9.2_math(1)

Use this prompt:

---

# Phase 2 Implementation — `phys-lib/bohm` Bohmian Equivariance

You are beginning **Phase 2** of the Phys system.

Phase 1 is closed and verified. The objective of Phase 2 is to stress-test the architecture against a **substantive physical theory** by implementing a canonical Bohmian-mechanics framework and deriving its **equivariance theorem** using the same disciplined workflow established by `phys-lib/gr`.

This is not a GR fixture copied into another directory.

The purpose is to determine whether:

```text id="d7ve11"
the same canonical mathematical substrate
        +
a different physical framework
```

can represent a genuinely different physical theorem without contaminating `phys-math` or collapsing framework semantics together.

---

# 0. Read the collective protocol first

Before changing code, read:

```text
/home/chaschel/Documents/go/AGENTS.md
/home/chaschel/Documents/go/phys-system/AGENTS.md
```

Then inspect:

```text
phys-artifact/
phys-math/
phys-lib/
phys-lib/gr/
phys-lib/gr/equivariance/
```

Also inspect the current normative representation specification actually present in the workspace.

If multiple PM-REP versions exist, determine which one is authoritative from its explicit version/authority metadata. Do not silently choose a version based on filename alone.

The collective protocol is authoritative for agent behavior; package-local `AGENTS.md` files remain authoritative for local implementation details.

---

# 1. Phase-2 objective

Primary deliverable:

```text id="rj7wqf"
phys-lib/bohm/
```

with a canonical Bohmian equivariance theorem fixture patterned after:

```text
phys-lib/gr/equivariance/
```

but using **Bohmian physical semantics and an actual Bohmian equivariance claim**, not the GR schematic `f(v)=2v` example.

The framework package should use the normal `phys-*` six-family architecture:

```text id="z2cc7x"
<ns>/framework/1
<ns>/assumption/1
<ns>/definition/1
<ns>/equation/1
<ns>/theorem/1
<ns>/interpretation/1
```

Use:

```text
PackageNamespace = "phys-bohm"
FrameworkID       = "bohmian_mechanics"
FrameworkVersion  = "1"
```

unless the current normative specification explicitly requires another value.

Do not create a global Bohmian ontology in `phys-math`.

---

# 2. Do not automatically create `phys-lib/qm`

Bohmian mechanics naturally depends on quantum-mechanical structure, but do **not** create `phys-lib/qm` merely because it seems conceptually tidy.

First determine whether the theorem genuinely requires a separate QM framework dependency.

Use this decision rule:

```text id="0j2ugf"
If Bohmian semantics can be expressed coherently in phys-lib/bohm
using explicit QM premises/assumptions:
    do not create phys-lib/qm.

If a real framework dependency is required:
    classify the dependency explicitly.
    If a production phys-lib/qm package is necessary,
    make it a minimal fully conforming base framework.

Dependency direction:
    phys-bohm → phys-qm

Never:
    phys-qm → phys-bohm
```

Do not create a placeholder/stub QM package just to satisfy the architecture.

---

# 3. Scientific source of the target theorem

Use the current research source containing the Bohmian current/guidance and equivariance sections as the source of **scientific intent**, not as a replacement for PM-REP schema.

The target content is:

```text id="g7b4fm"
Configuration:
    Q(t) ∈ 𝒬[M,Φ]

Wave function:
    Ψ(q,t)

Probability density:
    ρΨ = |Ψ|²

Current:
    j^A = (ħ/m_A) Im(Ψ* ∇^A Ψ)

Guidance:
    v^A = j^A / |Ψ|²
```

where defined.

The theorem assumptions include that:

```text id="j1r93v"
∂t |Ψ|² + ∇_A j^A = 0

∂t ρ + ∇_A(ρ v^A) = 0

ρ(t₀) = |Ψ(t₀)|²
```

and the required regularity/uniqueness conditions for propagation of the density equality.

The intended conclusion is:

```text id="u4r0ls"
ρ(t) = |Ψ(t)|²
```

for the domain/time interval covered by the stated assumptions.

The theorem is **conditional**.

Do not claim that equivariance:

```text id="n5m9ib"
derives the Born rule
```

or:

```text id="g1m15b"
derives the initial equality ρ(t₀)=|Ψ(t₀)|²
```

unless a separate theorem with separate premises is actually proved.

The source explicitly distinguishes equivariance from derivation of the initial equality. v9.2_math

---

# 4. Configuration-space discipline

Do not hard-code:

```text
𝑅^(3N)
```

as if that were the universal Bohmian ontology.

The research source explicitly frames the configuration space as something constructed from recovered physics:

```text
𝒬[M,Φ]
```

rather than assuming `R^(3N)`. v9.2_math

For a minimal executable Phase-2 witness, a concrete nonrelativistic coordinate sector may be chosen where justified, but the framework should record that specialization as an explicit scope/assumption rather than silently turning it into the universal configuration ontology.

---

# 5. First implementation gate: mathematical expressibility inventory

Before extending `phys-math`, inspect whether the existing Phase-1 representation can formally express the mathematical content needed for the theorem.

Inventory at least:

```text id="w7jrk2"
complex-valued quantities
complex conjugation
modulus / |Ψ|²
imaginary part
spatial derivatives
time derivatives
gradient
divergence
vector/scalar distinction
division by |Ψ|²
continuity equations
transport/flow equality
uniqueness / regularity assumptions
```

For every requirement classify it:

```text id="t12od4"
PHYS-MATH EXISTING
PHYS-MATH GAP
BOHM-LOCAL MATHEMATICS
PHYSICAL SEMANTICS
ASSUMPTION / INTERPRETATION
DEFER
```

Do not invent PhysMath constructs merely because Bohm needs them.

The current PhysMath boundary remains theory-neutral.

---

# 6. Mathematical-gap rule

If a required mathematical construct is not representable using current `phys-math`:

**do not immediately add an AST node, durable family, rule, or operator.**

First determine whether the missing construct is genuinely:

```text id="fg5v7p"
pure mathematics
framework-independent
reusable across future theories
```

versus:

```text id="s9v3ns"
Bohmian-specific mathematics/semantics
```

If it is genuinely generic but the current Phase-2 authority does not explicitly authorize modifying the PhysMath ontology, report:

```text
PHYS-MATH GAP

Required mathematical capability:
...

Why current PhysMath cannot represent it:
...

Why the capability appears theory-neutral:
...

Minimal candidate representation:
...

Candidate rule/serialization consequences:
...

Why this cannot safely remain Bohm-local:
...
```

Then stop **only the dependent behavior** and ask the architect.

Do not manufacture a fake representation using strings, synthetic function IDs, opaque theorem names, or prose.

If the current normative specification already authorizes the required mathematical extension, follow that specification exactly.

---

# 7. Bohm-local mathematics boundary

Bohm-local symbolic machinery is permitted when necessary, provided:

```text id="a3noxg"
GR/QM/Bohm physical semantics remain local
local representation is explicitly named
projection into PhysMath is explicit and fallible
nonrepresentable structures are not disguised as PhysMath
no synthetic mathematical identifiers are invented to fake compatibility
```

Do not introduce:

```text
phys-bohm:func/conjugate
phys-bohm:func/gradient
phys-bohm:func/divergence
```

as if they were PhysMath mathematical identities.

If a local Bohm representation cannot be projected into PhysMath, leave it local and make that boundary explicit.

---

# 8. The actual theorem must be formally represented

Do not make `manifest.json` carry the theorem.

Use the same semantic separation established in Phase 1:

```text
manifest
    = metadata

formal artifacts
    = mathematical / physical content

derivation
    = executable proof/protocol trace
```

The theorem must ultimately be represented through the existing durable framework:

```text id="nnbbt7"
PhysMath theorem
    ↓
Bohm PhysicalTheorem
    ↓
Bohm Interpretation
    ↓
Bohm assumptions
```

where supported by the schema.

Manifest prose never substitutes for formal mathematics.

---

# 9. Design the theorem before coding the derivation

Write the semantic claim explicitly before implementing the trace:

```text
CLAIM

Given:
    Ψ
    j
    v = j / |Ψ|²
    continuity equation for |Ψ|²
    continuity equation for ρ
    initial equality
    required regularity/uniqueness conditions

Prove:
    ρ(t) = |Ψ(t)|²
```

Then construct the claim-to-conclusion graph:

```text id="9j2t1z"
CLAIM
  ↓
formal LHS / RHS / proposition structure
  ↓
initial durable artifacts
  ↓
derivation steps
  ↓
designated final artifacts
  ↓
formal conclusion relation
  ↓
PhysMath Theorem
  ↓
Bohm PhysicalTheorem
  ↓
Bohm Interpretation
```

Every artifact defining the claim must be semantically live.

This requirement is now part of the collective operating protocol. AGENTS

---

# 10. No tautological equivariance theorem

Do not reduce the theorem to an accidental tautology such as:

```text
X = X
```

merely because both sides simplify to the same object.

The theorem must preserve the actual proposition:

```text id="lyzp5h"
ρ(t₀)=|Ψ(t₀)|²
        +
same transport law
        +
same velocity/current relation
        +
uniqueness
        ⇒
ρ(t)=|Ψ(t)|²
```

The semantic proof obligation is stronger than final byte equality.

Exactly as the Phase-1 GR correction taught us:

```text
normal_form(LHS) == normal_form(RHS)
```

does not by itself establish the intended theorem.

---

# 11. Fresh persisted-artifact verifier is mandatory from day one

Do not repeat the original GR workflow where this was discovered only during remediation.

The Bohm equivariance fixture must include a fresh verifier using only persisted state.

Pattern:

```text id="s7s0m2"
Build()
    ↓
extract only Store + durable refs
    ↓
discard Fixture semantic state
    ↓
fresh verifier
    ↓
reconstruct original proposition
    ↓
reconstruct derivation
    ↓
verify semantic trace
    ↓
verify conclusion
```

The fresh verifier must not rely on:

```text
Fixture.LHS0
Fixture.RHS0
Fixture.Assumption objects
Fixture-specific semantic caches
manifest prose
```

unless those values themselves are reconstructed from persisted artifacts.

The test must prove that the persisted representation contains enough information to establish the theorem's identity and trace.

---

# 12. Required H-3-style semantic mutation battery

Build these tests into the Bohm fixture from the beginning:

```text id="k0kzyu"
mutate original theorem LHS
mutate original theorem RHS
remove/change a defining assumption
change current definition
change guidance relation
change continuity-equation premise
break derivation linkage
change declaration
change ParameterID
keep final normal forms fixed
```

Every semantically invalid mutation must fail.

The critical test is:

> Can the original proposition be changed while the final normalized result remains unchanged and verification still succeeds?

Expected:

```text
NO
```

---

# 13. Assumption discipline

Every condition required by the derivation must be explicit.

Examples include:

```text id="7p1jcp"
|Ψ|² ≠ 0 where v = j/|Ψ|² is evaluated
regularity of Ψ
regularity of v
existence of the relevant flow
uniqueness of the transport/continuity evolution
initial equality ρ(t₀)=|Ψ(t₀)|²
```

Do not silently introduce any of these.

Do not confuse:

```text mathematical condition
```

with:

```text Bohmian physical assumption
```

Use `physmath/condition/1` only when the condition is genuinely expressible as mathematics.

Non-expressible physical/regularity assumptions remain framework-level.

Follow the established condition lifecycle:

```text
materialize
→ supply ConditionRefs
→ check
→ thread
```

and do not have rules silently generate missing conditions. This boundary is already explicit in the collective protocol. AGENTS

---

# 14. Bohm framework artifacts

Implement the minimum required framework artifacts:

```text
FrameworkArtifact

PhysicalAssumption
PhysicalDefinition
PhysicalEquation
PhysicalTheorem
PhysicalInterpretation
```

Do not add framework-specific durable families.

Potential semantic roles include:

```text
wave-function / dynamical assumptions
configuration-space assumptions
current definition
guidance law
continuity equation
equivariance theorem
Bohmian interpretation of the mathematical result
```

Use the standard framework schemas and target-family constraints already established.

PhysicalDefinition and PhysicalInterpretation remain distinct:

```text
PhysicalDefinition
    = what mathematical object/equation means physically

PhysicalInterpretation
    = interpretive context attaching that meaning
```

Do not merge them.

---

# 15. Framework closure

If a real QM base framework is introduced, test:

```text
phys-bohm → phys-qm
```

including:

```text
direct base
transitive base
valid inherited assumption
invalid unrelated assumption
reverse dependency
cycle
dangling base
wrong family
wrong namespace
```

Use the transitive closure behavior already hardened in GR.

Do not duplicate a production closure abstraction merely because fixture2 has intentional test-side isolation.

---

# 16. Bohmian protocol layout

Use the GR pattern:

```text id="rj3hjc"
phys-lib/bohm/
    AGENTS.md

    framework.go
    assumptions.go
    definitions.go
    equations.go
    theorem.go
    interpretation.go
    local algebra where actually necessary

    equivariance/
        AGENTS.md
        manifest.json
        protocol/
            derivation_test.go
            fresh_test.go
```

Names may be adjusted idiomatically, but ownership and package structure must remain coherent.

The nested external protocol-test boundary is intentional.

---

# 17. Protocol contract

The Bohm fixture's `AGENTS.md` must explicitly document:

```text id="8b4gkp"
the theorem claim
the physical assumptions
the mathematical representation
the derivation strategy
the conclusion
the limitations
the non-goals
the semantic-liveness invariant
the fresh-verifier invariant
```

Manifest must use the established protocol schema.

Do not create a Bohm-specific manifest schema.

Manifest fields must remain metadata unless their normative schema meaning explicitly says otherwise.

---

# 18. Derivation strategy

The preferred mathematical proof structure is:

```text id="fsz7vh"
1. Establish the Bohmian current j from the stated wavefunction/dynamics
   where the representation permits.

2. Establish:
       ∂t |Ψ|² + ∇·j = 0

3. Establish the guidance relation:
       v = j / |Ψ|²

4. Establish the actual-density transport equation:
       ∂t ρ + ∇·(ρv) = 0

5. Substitute:
       |Ψ|² v = j

6. Therefore:
       ∂t |Ψ|² + ∇·(|Ψ|² v) = 0

7. Compare with the actual-density transport equation.

8. Under the stated uniqueness/regularity conditions and
   initial equality:
       ρ(t₀)=|Ψ(t₀)|²

9. Conclude:
       ρ(t)=|Ψ(t)|²
```

Do not treat step 8 as a consequence unless its required mathematical conditions are actually represented.

If a step is being treated as a supplied premise rather than derived, mark it explicitly as a premise.

Do not silently turn:

```text
ASSUMED
```

into:

```text
DERIVED
```

---

# 19. Scope boundaries

Phase 2 target is **Bohmian equivariance**.

Do not implement unrelated Bohmian/QM work merely because the concepts are nearby.

Explicitly defer unless independently required:

```text
Born-rule derivation
typicality theory
quantum equilibrium relaxation / subquantum H theorem
measurement theory
effective wavefunctions
conditional wavefunctions
spin
identical particles
relativistic Bohmian mechanics
quantum field theory
configuration-space ontology beyond the target fixture
numerical trajectory integration
empirical simulation
```

The current research source discusses some of these topics, but they are not automatically Phase-2 deliverables.

The target should be the smallest rigorous Bohmian theorem that genuinely stresses the architecture.

---

# 20. No Phase-2 scope inflation in PhysMath

Do not respond to Bohmian requirements with:

```text
+10 AST nodes
+20 durable families
+universal operator ontology
+generic PhysicsContract
+global wavefunction ontology
```

The Phase-1 lesson is:

```text
new theory requirement
    ↓
classify ownership
    ↓
use existing mathematics where possible
    ↓
keep theory-specific structure local
    ↓
promote only with multi-framework evidence
```

`phys-lib/core` remains empty unless a real multi-framework abstraction now has evidence.

Any proposed promotion requires a classification record.

---

# 21. Identity and canonicalization

For all new mathematical artifacts:

- use exact existing canonical identity rules;
- use declaration-bound ParameterIDs;
- do not use bare symbolic names as semantic identity;
- pin canonical bytes for important goldens;
- reject malformed and foreign refs;
- preserve SET/SEQUENCE semantics;
- verify encode/decode round-trip;
- verify repeated construction determinism.

Do not use implementation-generated values as the sole correctness oracle.

Evidence classes remain:

```text
SPEC-DERIVED
HAND-DERIVED
LEGACY-PORTED
INDEPENDENTLY-COMPUTED
IMPLEMENTATION-DERIVED
```

Implementation-derived goldens may detect drift but cannot be the only semantic witness.

---

# 22. Adversarial testing

For every important theorem component ask:

> What incorrect implementation could still make the current tests pass?

Attack:

```text id="l0pzkl"
wrong assumption
missing initial equality
wrong current
wrong velocity denominator
wrong continuity equation
wrong configuration domain
wrong declaration
wrong parameter binding
broken derivation linkage
dropped step
reordered step
metadata substituted for formal artifact
same final normal form from different proposition
```

Tests should include:

```text
positive
negative
mutation
semantic witness
independent golden
```

This is now a mandatory collective testing principle. AGENTS

---

# 23. Cross-framework comparison

After the Bohm fixture works, verify the architectural distinction:

```text id="svyo3q"
GR equivariance fixture
        ≠
Bohmian equivariance theorem
```

even though the word "equivariance" appears in both.

Do not share a physical `Equivariance` type between frameworks.

The commonality should exist only where the mathematics is genuinely shared.

The Bohmian theorem should have its own:

```text
assumptions
definitions
physical equations
interpretation
physical theorem
```

while using PhysMath mathematics where appropriate.

---

# 24. Required Bohm fixture deliverables

At minimum:

```text id="rrp7v6"
[ ] phys-lib/bohm framework artifact
[ ] all six framework families where required
[ ] Bohmian assumptions
[ ] Bohmian definitions
[ ] Bohmian physical equations
[ ] Bohmian physical theorem
[ ] Bohmian interpretation
[ ] mathematical theorem artifact
[ ] explicit derivation
[ ] canonical nested equivariance fixture
[ ] manifest using existing protocol schema
[ ] external protocol/derivation_test.go
[ ] fresh persisted-artifact verifier
[ ] semantic-liveness tests
[ ] persisted-artifact mutation tests
[ ] independent goldens
[ ] framework isolation tests
[ ] classification records for significant new abstractions
```

---

# 25. Completion criteria

Phase 2 is **not complete merely because Go tests pass**.

Report independently:

```text
STRUCTURAL
    package/family/schema/module requirements satisfied

BEHAVIORAL
    positive and negative execution tests pass

SEMANTIC
    actual Bohmian equivariance proposition is preserved
    from claim through derivation to conclusion

INDEPENDENCE
    fresh verification works from persisted artifacts
    important goldens are independently derived

BOUNDARY
    Bohm semantics remain in phys-lib/bohm
    PhysMath remains theory-neutral
    no forbidden framework dependencies
```

---

# 26. Required final adversarial closure

Before declaring Phase 2 complete, explicitly answer:

```text
1. What is the exact Bohmian theorem being proved?

2. Which parts are mathematical propositions?

3. Which parts are Bohmian physical assumptions?

4. Which parts are framework interpretations?

5. Which mathematical components were representable in existing PhysMath?

6. Which were not?

7. Did any PhysMath extension become necessary?
   If yes, was it explicitly authorized?

8. Can a fresh verifier reconstruct the theorem from persisted artifacts?

9. Can the original proposition be corrupted while the final normal forms
   remain fixed and still pass?
   MUST be NO.

10. Does the theorem derive initial Born/equilibrium equality?
    MUST be NO unless separately and explicitly proved.

11. What incorrect implementation could still pass the tests?

12. What remaining limitations prevent this fixture from being a complete
    Bohmian formalization?
```

---

# 27. Stop policy

Proceed without asking about ordinary Go implementation details.

Do **not** stop for:

```text
helper names
constructor names
file names
private structs
test organization
allocation
CI mechanics
```

Stop only the affected path when encountering:

```text
ownership ambiguity
dependency ambiguity
canonical identity ambiguity
serialization ambiguity
durable-family ambiguity
new PhysMath vocabulary
new rule semantics
new protocol semantics
physical-semantic ambiguity
framework-boundary ambiguity
Phase-2 scope ambiguity
genuine mathematical-representation gap
```

When stopping, report:

```text
CLAIM
CURRENT REPRESENTATION
SPECIFICATION REQUIREMENT
IMPLEMENTATION EVIDENCE
COMPETING INTERPRETATIONS
CONSEQUENCES
RECOMMENDED RESOLUTION
```

Do not invent a solution merely to avoid asking.

---

# 28. Final report

Return:

```text
# Phase-2 Bohmian Equivariance Report

STATUS:
[COMPLETE / BLOCKED ON SPECIFIC GAP]

## Framework
...

## Theorem
...

## Mathematical representation
...

## Physical assumptions
...

## Derivation
...

## Semantic claim-to-conclusion trace
...

## Fresh persisted-artifact verification
...

## Mutation battery
...

## PhysMath gaps
...

## Framework-local mathematics
...

## Core promotion decisions
...

## Protocol
...

## Test independence
...

## Regression
...

## Scope audit
...

## Remaining limitations
...

## Architectural impact
...

## Phase-3 recommendations
...
```

### Final rule

The goal is **not** to prove that Bohmian mechanics is true.

The goal is to determine whether the Phys architecture can faithfully represent and derive a real Bohmian theorem while preserving:

```text
scientific intent
      ↓
physical semantics
      ↓
pure mathematics
      ↓
canonical identity
      ↓
formal derivation
      ↓
theorem conclusion
      ↓
framework interpretation
```

without collapsing those layers.

Do not call the theorem complete merely because:

```text
go test ./...
```

is green.

The theorem is complete only when the **semantic claim survives all the way into the persisted formal conclusion and can be independently reconstructed and falsified from the persisted representation**.