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


## prompt

there is a lot of reference from arxiv per below, how do we resolve this?&#x20;

The equivariance theorem in [Bohmian Mechanics]\([https://arxiv.org/html/1704.08017v3](https://arxiv.org/html/1704.08017v3)) proves that if a particle's initial position distribution matches the quantum equilibrium value $\vert{}\psi_t\vert{}^2$ at a starting time, it will preserve this statistical distribution at all future times. [1]&#x20;
**##** **Core** **Concepts** **of** **Equivariance**

\* Definition: Equivariance means that probability distributions of particle configurations remain synchronized with the evolving quantum wave function over time. [2, 3]&#x20;
\* Continuity Equation: The theorem mathematically relies on the conservation of probability and the quantum continuity equation $\frac{\partial \rho}{\partial t} = -\nabla \cdot (\rho v)$. [3]&#x20;
\* Quantum Equilibrium: Because |ψ|² distribution is preserved (equivariant), empirical measurements in Bohmian mechanics statistically match standard quantum mechanics predictions (the Born rule). [1, 4]&#x20;
\* Pilot-Wave Dynamics: Particle positions evolve deterministically via guiding equations, yet aggregate outcomes mimic standard quantum randomness due to this foundational equivariance. [5, 6]&#x20;

If you'd like, I can:

\* Provide the mathematical derivation of the continuity equation in pilot-wave theory.
\* Discuss how equivariance relates to quantum non-equilibrium hypotheses.

Let me know how you'd like to proceed.

[1] [https\://arxiv.org]\(https\://arxiv.org/html/1804.08853v3)
[2] [https\://arxiv.org]\(https\://arxiv.org/html/0704.3070v1)
[3] [https\://arxiv.org]\(https\://arxiv.org/html/0903.2601v1)
[4] [https\://arxiv.org]\(https\://arxiv.org/pdf/2401.04091)
[5] [https\://arxiv.org]\(https\://arxiv.org/html/1704.08017v3)
[6] [https\://arxiv.org]\(https\://arxiv.org/pdf/0903.2601)


## answer

We should **not put all of those arXiv papers directly into the Phase-2 implementation prompt as if they were equal authorities**. That would recreate exactly the ambiguity we just worked to eliminate.

The right solution is a **scientific-source hierarchy**.

### 1. Pick one primary source for the theorem

For the Phase-2 target, make **Tumulka, *Bohmian Mechanics*, arXiv:1704.08017v3** the primary scientific source.

It directly states the nonrelativistic Bohmian equation of motion, Schrödinger evolution, the probability-current form of the velocity, and the position equivariance theorem: if the initial configuration is distributed as \(|\psi_0|^2\), then the configuration remains \(|\psi_t|^2\)-distributed. [arXiv](https://arxiv.org/html/1704.08017v3)

That should be the source from which the **canonical Phase-2 theorem is formally extracted**.

### 2. Use the other papers as supporting sources, not competing specifications

I would classify the references like this:

| Tier | Source | Role |
|---|---|---|
| **Primary** | Tumulka 2019/2021, `1704.08017v3` | Defines the concrete theorem target and standard nonrelativistic Bohmian setting |
| **Formal foundation** | Dürr et al., `0704.3070v1` | Gives the more abstract definition of equivariance and explains the continuity-equation/uniqueness structure |
| **Independent cross-check** | `0903.2601v1` | Concise independent presentation of position equivariance from the continuity equation |
| **Deferred extension** | `1804.08853v3` | Particle creation / variable-particle-number extension; not required for the base theorem |
| **Deferred extension** | `2401.04091` | Momentum-equivariance/generalized dynamics; not required for the base position-equivariance target |

The 2007 paper is particularly useful because it defines equivariance as preservation of the functional \(P^\psi\) under Bohmian flow and explicitly relates this to continuity equations and uniqueness. [arXiv](https://arxiv.org/html/0704.3070v1)

The 2009 review independently states the familiar result that an initial \(|\Psi|^2\) distribution remains \(|\Psi_t|^2\), deriving this from the continuity equation. [arXiv](https://arxiv.org/html/0903.2601v1)

The 2018 paper is an **extension** into particle creation, where configuration space becomes a variable-particle-number space; that is useful later but should not silently expand the Phase-2 target. [arXiv](https://arxiv.org/html/1804.08853v3)

The 2024 source is also broader—for example, it discusses momentum equivariance—so it should not accidentally turn our position-equivariance fixture into a momentum-equivariance project. [arXiv](https://arxiv.org/pdf/2401.04091)

---

# 3. There is an important correction to our current Phase-2 prompt

The prompt currently contains a synthesized theorem:

\[
v^A=\frac{j^A}{|\Psi|^2},
\]

\[
\partial_t|\Psi|^2+\nabla_Aj^A=0,
\]

\[
\partial_t\rho+\nabla_A(\rho v^A)=0,
\]

therefore

\[
\rho(t)=|\Psi(t)|^2.
\]

That is a reasonable abstraction, and it is consistent with the supporting literature, but **we should not pretend that this synthesized form is itself the primary source**.

The source hierarchy should instead be:

```text
Primary source theorem
        ↓
extract exact scientific claim
        ↓
identify assumptions
        ↓
translate into canonical PhysMath representation
        ↓
derive executable proof
```

This keeps scientific interpretation separate from our implementation design.

---

# 4. More importantly, separate three propositions

The references currently mix several related claims that we should **not collapse into one theorem**.

### Proposition A — Position equivariance

This is our Phase-2 target:

\[
\rho(t_0)=|\psi_{t_0}|^2
\Longrightarrow
\rho(t)=|\psi_t|^2.
\]

This is the theorem we should formalize.

### Proposition B — Quantum equilibrium hypothesis

This says, roughly, that the actual configuration is distributed according to \(|\psi|^2\).

That is a **physical/theoretical assumption or additional principle**, not something we should pretend is derived by equivariance.

Tumulka explicitly distinguishes the equivariance theorem from the theory's rule that the actual configuration is in quantum equilibrium. [arXiv](https://arxiv.org/html/1704.08017v3)

### Proposition C — Born-rule empirical predictions

The claim that Bohmian mechanics reproduces standard quantum statistics is downstream of the quantum-equilibrium framework and measurement analysis. It should **not** be the conclusion of our equivariance theorem fixture.

This distinction is very important because the user's pasted text says:

> “Because \(|\psi|^2\) distribution is preserved (equivariant), empirical measurements ... match ... the Born rule.”

That is too compressed for our formal system. Equivariance establishes preservation of the \(|\psi|^2\) distribution; connecting that to empirical outcomes involves additional theoretical structure. Tumulka's discussion similarly treats the equilibrium rule separately from the mathematical equivariance theorem. [arXiv](https://arxiv.org/html/1704.08017v3)

So our Phase-2 theorem should **stop at equivariance**.

---

# 5. Use a source-to-claim matrix

This is the process improvement I would add to the Phase-2 prompt.

Before coding, the agent must create an internal table like:

```text
CLAIM: Position equivariance

Scientific sources:
    BH-T1 = arXiv:1704.08017v3
    BH-F1 = arXiv:0704.3070v1
    BH-R1 = arXiv:0903.2601v1

Primary authority:
    BH-T1

Formal cross-check:
    BH-F1

Independent confirmation:
    BH-R1

Deferred extensions:
    BH-X1 = arXiv:1804.08853v3
    BH-X2 = arXiv:2401.04091

Phase-2 theorem boundary:
    position equivariance only

Not proved by this fixture:
    initial quantum equilibrium
    Born rule
    typicality
    measurement statistics
    momentum equivariance
    particle creation
```

This means the coding agent doesn't need to reason through five papers simultaneously.

---

# 6. Put the source dossier in `phys-system`

Given the system we have now built, I would go one step further.

Create something like:

```text
phys-system/
    research/
        bohmian/
            equivariance.md
            sources.md
```

This remains **documentation only**.

`equivariance.md` says what scientific proposition we are targeting.

`sources.md` records:

```text
source ID
author
title
arXiv ID/version
role
claims supported
claims explicitly NOT adopted
```

Then the Phase-2 implementation prompt can simply say:

> Read `phys-system/research/bohmian/equivariance.md` and `sources.md`. The source dossier is the scientific-input boundary for this fixture.

That is much cleaner than embedding a bibliography into every coding prompt.

---

# 7. This also protects us from source drift

Notice that the references are versions:

```text
1704.08017v3
0704.3070v1
0903.2601v1
1804.08853v3
```

That's good.

We should **pin the exact arXiv version** used as evidence.

The agent should never silently replace:

```text
1704.08017v3
```

with whatever the latest revision happens to be.

That gives us reproducibility:

```text
scientific source
     ↓
exact version
     ↓
formal claim
     ↓
canonical artifact
```

This is analogous to the canonical identity discipline in the software layer.

---

# 8. The Phase-2 prompt should therefore change

I would replace the current large "Scientific source of the target theorem" section with something like:

> **Scientific-source protocol**
>
> The Phase-2 Bohmian equivariance fixture must be grounded in a pinned scientific-source dossier under `phys-system/research/bohmian/`.
>
> Primary source: Tumulka, *Bohmian Mechanics*, arXiv:1704.08017v3.
>
> Supporting formal source: Dürr et al., *On the Uniqueness of Quantum Equilibrium in Bohmian Mechanics*, arXiv:0704.3070v1.
>
> Independent cross-check: arXiv:0903.2601v1.
>
> Deferred extensions: arXiv:1804.08853v3 and arXiv:2401.04091.
>
> The primary source determines the Phase-2 scientific target. Supporting sources may clarify definitions, assumptions, or mathematical justification but may not silently enlarge or alter the target. Deferred sources are literature evidence only and must not introduce particle-creation, momentum-equivariance, relativistic, or other extensions into the Phase-2 fixture.
>
> Before implementation, produce a source-to-claim trace identifying the exact proposition, assumptions, equations, and conclusion being formalized. Any disagreement among sources must be surfaced; the agent must not reconcile it silently.

That is the right solution.

---

## And there is a deeper benefit

This turns the whole architecture into:

```text
                    SCIENTIFIC LITERATURE
                           │
                    source hierarchy
                           │
                           ▼
                  SEMANTIC CLAIM
                           │
                           ▼
                  PHYSICAL FRAMEWORK
                    phys-lib/bohm
                           │
                           ▼
                 PURE MATHEMATICS
                     phys-math
                           │
                           ▼
               CANONICAL REPRESENTATION
                   phys-artifact
```

The arXiv papers are therefore **scientific evidence feeding the framework**, not things that get mixed directly into the mathematical/compiler layer.

That's exactly the separation we want Phase 2 to test.

And in this particular case, the source hierarchy also prevents a very important mistake: **we formalize Bohmian position equivariance, not "equivariance + Born rule + typicality + measurement + particle creation" as one enormous theorem.** The literature itself distinguishes those layers. [arXiv](https://arxiv.org/html/1704.08017v3)



## prompt

