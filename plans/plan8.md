## prompt

TOP PRIORITY
- Our top priority is implementing PhysMath and phys-* external libraries so the implementation guidelines must be solid. 
- The unit of research is the typical physics paper where it may describe a theorem or hypotheses but the ultimate output of a physics paper is an equation or set of equations otherwise it is just prose. criteria for implementation guidelines for this top priority
- PhysMath is not specific to any physical theory
- phys-* external libraries are tied to specific assumptions, hypotheses, theorem and the like. Even standard quantum mechanics has its own interpretation so the rule is that "Give a hundred physicits a question and you will get a hundred different answers". That is the essence of phys-* libraries
- To accomplish this, we set the reference target of implementation to representing the derivation of equivariance theorem (instead of the TOE target mentioned earlier)

SECONDARY PRIORITY
- Numerical layer and TOE agent can be implemented later as separate projects
- the other things that complicates our specs definition is the entangled role of provenance and the like. We should 1) focus more on how Phys and PhysMath can concentrate on the representation part of physics (the R in ARI analysis, representation and interpretation) and 2) let AI agents and humans focus on the analysis and interpretation parts of deriving hypotheses and theorems as part of the research process

We do not have time for endless reviews so we need to start the ball rolling to coding implementation as much as possible for the top priority. Do you suggest to separate the specs of top priority mentioned above from secondary priority so that we do not conflate the two?



## answer


Yes. **I strongly recommend separating them now.** In fact, given your clarified priorities, I would go further: **v1.5 should not be the implementation architecture for the first coding phase.** It is trying to close too many boundaries simultaneously.

The current v1.5 still treats PhysMath, phys-*, Phys, TOE-Agent, Numerical, provenance/trust, formalization, research states, and Growth Gate as one architecture. It explicitly places all of those projects in the target ecosystem and gives them production dependency relationships. PHYSMATH_PHYS_EXTERNAL_ARCHITEC… PHYSMATH_PHYS_EXTERNAL_ARCHITEC… It also still spends substantial architectural surface on ResearchHypothesis, formalization, TOE boundaries, and numerical execution. PHYSMATH_PHYS_EXTERNAL_ARCHITEC… PHYSMATH_PHYS_EXTERNAL_ARCHITEC…

That is exactly the conflation you are now trying to eliminate.

## My recommendation: two specifications, with only one being implementation-critical now

### Specification A — **PhysMath + phys-* Representation Architecture**

This becomes the **immediate implementation contract**.

Its question is:

> **How do we represent mathematical structure and alternative physical theories/frameworks precisely enough that a coding agent can implement them without inventing semantics?**

Nothing else needs to be solved yet.

The architectural hierarchy becomes:

```text
                    PHYSICS PAPER
                         │
               analysis / interpretation
                         │
                         ▼
                 equations / theorem
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
         PhysMath                 phys-*
     mathematical form       physical meaning
       / derivation            / assumptions
             │                       │
             └───────────┬───────────┘
                         ▼
                  REPRESENTATION
```

And importantly:

```text
PhysMath = "What mathematical structure is this?"
phys-*   = "What physical system/framework does this structure represent?"
```

PhysMath remains completely theory-neutral.

A `phys-*` package is deliberately **not** merely a plugin around PhysMath. It is a framework-specific semantic layer capable of saying:

```text
this mathematical object
+ these assumptions
+ these equations
+ these definitions
+ these transformation rules
+ this theorem
= this particular physical framework
```

That directly captures your “one hundred physicists, one hundred answers” principle.

The same mathematical object can therefore have:

```text
phys-qm interpretation A
phys-qm interpretation B
phys-bohm interpretation
phys-gr interpretation
...
```

without changing PhysMath.

The existing v1.5 principle that multiple frameworks can interpret the same mathematical artifact is exactly the right direction. PHYSMATH_PHYS_EXTERNAL_ARCHITEC…

---

# The crucial change: make the equivariance theorem the reference vertical slice

I agree strongly with replacing the earlier TOE-oriented target.

Do **not** make:

```text
TOE formalization
Dancer / Dance / Stage
Numerical Contract
hypothesis lifecycle
```

the architectural acceptance target.

Instead make:

> **Representation of the derivation of an equivariance theorem**

the canonical implementation specimen.

That is much better because it forces the architecture to demonstrate nearly everything that actually matters for the top priority:

```text
definitions
objects
spaces
maps
transformations
parameters
operators
equations
conditions
composition
derivation steps
assumptions
theorem statement
proof/derivation structure
```

without requiring us to solve the whole research platform.

The target should therefore be something like:

```text
Paper
  └── Equation/Theorem family
        ├── Definitions
        ├── Assumptions
        ├── Mathematical objects
        ├── Transformations
        ├── Intermediate equations
        ├── Conditions
        ├── Derivation
        └── Final equivariance theorem
```

But there is an important boundary:

**The paper itself does not belong in PhysMath.**

A paper is a **research artifact**.

PhysMath only represents the mathematical content extracted from it.

That distinction is critical.

---

# The unit of research should indeed be the physics paper

I agree with your premise, with one refinement.

The research unit is:

```text
PhysicsPaper
```

but the **computational unit of representation** is:

```text
EquationSet / Derivation / Theorem
```

A paper can contain prose, motivation, historical discussion, interpretation, experimental narrative, etc.

But the part that PhysMath/phys-* need to operationalize is the formal mathematical payload:

```text
definitions
equations
relations
transformations
assumptions
derivations
theorems
```

So conceptually:

```text
Paper
│
├── prose                     ← humans / AI agents
├── analysis                  ← humans / AI agents
├── interpretation            ← humans / AI agents
│
└── formal mathematical core
      │
      ├── equations
      ├── definitions
      ├── assumptions
      ├── transformations
      └── derivations
             │
             ▼
          PhysMath
             │
             ▼
          phys-*
```

That is a much cleaner architecture.

---

# This also gives us a much cleaner meaning of ARI

Your ARI framing becomes extremely useful here:

```text
A = Analysis
R = Representation
I = Interpretation
```

The first implementation phase should deliberately concentrate on:

```text
                     ┌───────────────────────┐
                     │       RESEARCH        │
                     └───────────────────────┘
                                │
                 ┌──────────────┴──────────────┐
                 ▼                             ▼
            ANALYSIS                      INTERPRETATION
          humans + AI                    humans + AI
                 │                             │
                 └──────────────┬──────────────┘
                                ▼
                        REPRESENTATION
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
                PhysMath                 phys-*
```

And that means **we should stop trying to encode Analysis and Interpretation into the foundation architecture.**

The architecture should provide the representation substrate that those processes consume and produce.

---

# What PhysMath should actually be responsible for

I would make its responsibility much narrower and stronger:

### PhysMath owns

```text
mathematical objects
mathematical types
expressions
equations / relations
functions
maps
operators
tensors
indices
transformations
conditions
derivations
mathematical identities
canonicalization
mathematical equality
deterministic serialization
mathematical derivation traces
```

### PhysMath does NOT own

```text
physics
physical assumptions
physical interpretation
experimental meaning
paper metadata
authors
citations
provenance
scientific truth
theory preference
research workflow
hypothesis status
numerical execution
```

This is more important than adding more machinery.

---

# What phys-* should actually be responsible for

This is where the “hundred physicists” idea belongs.

A `phys-*` package owns:

```text
physical ontology
physical assumptions
physical vocabulary
framework-specific equations
framework-specific constraints
framework-specific transformation rules
framework-specific theorem statements
framework-specific derivation validity
framework-specific interpretation of mathematical structures
framework-specific mappings into PhysMath
```

For example, `phys-gr` might say:

```text
Metric
    -> Lorentzian metric under GR assumptions

Connection
    -> Levi-Civita connection under GR assumptions

Curvature
    -> Riemann curvature tensor

Einstein equation
    -> specific relation involving geometry and stress-energy

Equivariance
    -> equivariance under this particular symmetry/action structure
```

Another `phys-*` framework could use the same underlying mathematical structures differently.

That is precisely what we want.

---

# And I would deliberately remove provenance from the first architecture

This is one of the biggest changes I recommend.

Your instinct is correct:

> **Do not allow provenance/trust/corpus machinery to become the architecture's center of gravity.**

The current v1.5 still makes the trust system a constitutional concern and carries provenance/assumptions/conventions throughout the architecture. PHYSMATH_PHYS_EXTERNAL_ARCHITEC…

For Phase 1, we only need:

```text
representation provenance
```

in the very modest sense of:

```text
Where did this equation/derivation come from?
```

That can simply be an external reference.

We do **not** need to settle:

```text
trusted
established
corpus
MRC
promotion
human admission
research state
formalization acceptance
```

Those are second-order concerns.

They should move into Specification B.

---

# Specification B — Research / Integration Architecture

This should be explicitly declared **deferred**.

It contains:

```text
PhysicsPaper
ResearchHypothesis
CorrespondenceProposal
ProposedInterpretation
formalization workflow
TOE-Agent
Numerical
ComputationalContract
provenance
trust
corpus status
research state machines
paper lineage
human/AI research workflow
```

That material can evolve independently.

And importantly:

**Specification B must consume Specification A.**

Not the reverse.

```text
             Research / AI / Humans
                       │
             Specification B
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          PhysMath             phys-*
       Specification A      Specification A
```

This gives us a very clean dependency direction.

---

# I would also change the definition of MVP

Right now the architecture implicitly defines MVP as a cross-project ecosystem.

I would redefine MVP much more aggressively:

## MVP = represent one nontrivial derivation end-to-end

Specifically:

```text
equivariance theorem
       ↓
definitions
       ↓
mathematical objects
       ↓
assumptions
       ↓
transformations
       ↓
equations
       ↓
derivation
       ↓
theorem
```

The acceptance question becomes:

> **Can PhysMath + one phys-* library represent the derivation of an equivariance theorem completely, canonically, and reproducibly without either package having to invent the other's semantics?**

That is a spectacularly better test.

Because if we cannot do that cleanly, adding TOE-Agent, Numerical, provenance, trust, or paper workflows will only hide the architectural weaknesses.

---

# This also changes what should be in the coding guidelines

The coding-agent instruction should become something very close to:

```text
Implement the representation architecture only.

PhysMath must remain theory-neutral.

phys-* packages must contain physical meaning and framework-specific
assumptions, definitions, equations, and theorem/derivation rules.

Do not encode physical meaning in PhysMath.

Do not encode research workflow in PhysMath or phys-*.

Do not implement TOE-Agent.

Do not implement Numerical.

Do not implement provenance/trust/corpus machinery beyond the explicitly
required representation references.

Do not modify Phys.

The reference implementation target is the complete representation of
the derivation of an equivariance theorem.

When a required mathematical capability is missing:
    determine whether it is mathematical -> PhysMath
    or physical/framework-specific -> phys-*

When the distinction is unclear:
    stop; do not invent a cross-layer semantic rule.
```

That is dramatically easier for a coding agent to follow.

---

# One further architectural adjustment I would make

I would **not make “PhysicsPaper” a PhysMath artifact**.

Instead:

```text
Research layer
    PhysicsPaper
         │
         │ extracts formal content
         ▼
    PhysMath artifacts
         │
         ▼
    phys-* interpretation
```

That preserves the distinction between:

```text
what the paper says
```

and

```text
what mathematical structure the paper contains
```

and

```text
what physical meaning a particular framework assigns to that structure
```

That distinction will become enormously valuable later when AI agents are doing analysis.

---

# The resulting architecture is much simpler

I would now aim for this:

```text
                 RESEARCH WORLD
          Papers / Humans / AI Agents
                  A + I
                     │
                     │
                     ▼
             ┌───────────────┐
             │   REPRESENT   │
             └───────────────┘
                     │
             ┌───────┴───────┐
             ▼               ▼
        ┌─────────┐      ┌─────────┐
        │ PhysMath│      │ phys-*  │
        │    R    │◄────►│    R+I  │
        └─────────┘      └─────────┘
             │               │
             └───────┬───────┘
                     ▼
                  Phys
             frozen authority
```

And later:

```text
                   REPRESENTATION
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
          Numerical              Research
             │                       │
             │                       │
             └───────────┬───────────┘
                         ▼
                    TOE-Agent
```

But **none of that needs to block today's implementation**.

## Bottom line

**Yes: separate them, and make the separation architectural rather than merely organizational.**

I would make **PhysMath + phys-* + the frozen Phys boundary = Phase-1 architecture**, with **equivariance-theorem derivation as the reference vertical slice**.

Then make **TOE-Agent + Numerical + provenance/trust + research state + paper workflow = Phase-2 integration architecture**, explicitly non-blocking for Phase 1.

The current v1.5 already contains enough of the right representation machinery that we should **not throw it away**. Instead, strip its Phase-2 material out of the critical path and make the equivariance derivation the benchmark that determines whether the Phase-1 API/schema is actually good. The current document's strong separation of pure mathematics from physical interpretation is exactly the foundation to retain. PHYSMATH_PHYS_EXTERNAL_ARCHITEC…

**I would make this split now rather than doing another giant cross-project blocker review.** The next useful artifact should be a **Phase-1 PhysMath + phys-* Representation Specification**, not another all-encompassing architecture revision.


## prompt

just to clarify, to implement the equivariance theorem representation, does it mean you will generate the phys-* libraries for quantum mechanics etc

## answer

Yes — **but not all of them at once, and the distinction matters.**

The implementation target has two layers:

```text
PhysMath
    ↓
generic mathematical representation of equivariance
    ↓
phys-* library
    ↓
framework-specific interpretation and assumptions
```

So the **equivariance theorem itself should not be hard-coded into PhysMath as a quantum-mechanics theorem**.

### What PhysMath implements

PhysMath should be able to represent the mathematical structure needed for an equivariance derivation:

```text
state space
group / transformation
action
measure / density
mapping
operator
equation
condition
derivation step
theorem statement
```

and the generic mathematical relationship:

```text
ρ
  ── transformation/action ──► ρ'

equivariance:
        ρ' = transformed(ρ)
```

The exact mathematical structure depends on the theorem, but **PhysMath knows only that it is mathematics**.

### What `phys-*` implements

A `phys-*` library gives that mathematics physical meaning.

For example, an initial implementation could be:

```text
phys-qm
```

which supplies the quantum-mechanical concepts, assumptions, equations, operators, and interpretation needed to instantiate a particular equivariance theorem.

Then potentially:

```text
phys-bohm
phys-gr
phys-cst
phys-cdt
...
```

could provide **different physical interpretations of related mathematical structures**.

That is actually an excellent test of the architecture because the same underlying mathematical substrate can be reused while the physical semantics differ.

For example:

```text
                    PhysMath
                       │
             generic mathematical
             equivariance structure
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      phys-qm      phys-bohm      phys-gr
          │            │            │
       QM meaning   Bohm meaning   GR meaning
```

The crucial rule remains:

> **PhysMath never needs to know that the structure came from quantum mechanics, Bohmian mechanics, GR, or any other theory.**

---

## But I would not implement five `phys-*` libraries immediately

For the first coding wave, I recommend:

```text
Phase 1A
    PhysMath core

Phase 1B
    one reference phys-* implementation

Phase 1C
    second phys-* implementation specifically to prove
    that framework disagreement works
```

The first `phys-*` should be chosen based on the **equivariance theorem we are actually going to encode**.

That is important because "equivariance theorem" is a mathematical pattern, while a particular physical equivariance theorem belongs to a particular framework.

So the real acceptance test becomes:

```text
             One physics-paper theorem
                       │
                       ▼
              PhysMath representation
                       │
                       ▼
             phys-* interpretation
                       │
                       ▼
          complete derivation reproduced
```

Then the **second framework** is the architectural stress test:

```text
Same / related mathematical structures
              │
       ┌──────┴──────┐
       ▼             ▼
    phys-qm       phys-bohm
       │             │
   interpretation A  interpretation B
```

If both can coexist without contaminating PhysMath, we have strong evidence that the architecture is doing what we intended.

### So the answer is

**Yes, we should generate `phys-*` libraries as part of implementing the equivariance-theorem vertical slice — but the immediate goal is not "build all physics theories."**

The goal is:

> **Build PhysMath + one concrete `phys-*` framework deeply enough to represent an actual equivariance-theorem derivation, then add a second framework to prove that the separation is real.**

That is much more tractable than trying to finish `phys-qm`, `phys-bohm`, `phys-gr`, `phys-cst`, and `phys-cdt` before we have validated the core representation model.


## prompt

and that phys-qm will be used by other interpretations aside from bohmian mechanics, right? how do we ensure that we maintain the minimum assumptions for standard QM like phys-qm without getting it tied to different interpretations of quantum mechanics QM? same goes with phys-cdt, it circles back to "hundred physicists, hundred answers" principle

## answer

Yes. **`phys-qm` should be reusable by multiple quantum interpretations.** In fact, that is one of the strongest tests of the architecture you are building.

The key is that we should **not equate “phys-qm” with “one interpretation of quantum mechanics.”** We should define it as the **minimum physical formalism that a family of QM interpretations can share**.

The same principle applies to `phys-cdt`, `phys-gr`, `phys-cst`, etc.

## The architectural rule I would lock

Use a two-level structure:

```text
PhysMath
   │
   │ pure mathematical representation
   ▼
phys-qm
   │
   │ minimal QM physical commitments
   │
   ├───────────────┬────────────────┬────────────────┐
   ▼               ▼                ▼                ▼
phys-bohm    phys-qm-copenhagen   phys-qm-MWI   phys-qm-collapse
```

The important point is that:

```text
phys-qm
    ≠
Bohmian mechanics
```

and:

```text
phys-qm
    ≠
Copenhagen interpretation
```

Rather:

```text
phys-qm
    = common physical formalism on which multiple
      interpretations can operate
```

That directly implements your **“hundred physicists, hundred answers”** principle.

---

# What belongs in `phys-qm`?

The minimum assumptions should be defined by a very strict test:

> **Would removing this assumption change what we mean by the common QM formalism, rather than merely choosing one interpretation of that formalism?**

That gives us a useful separation.

For example, `phys-qm` can contain things such as:

```text
state representation
observables
operators
unitary transformations
probability structure
Hilbert-space structure
composition rules
symmetry structure
standard mathematical/physical relationships
```

But it should **not** prematurely encode:

```text
what the wavefunction physically is
whether collapse is real
whether particles have definite positions
whether branches are physically real
whether measurement is fundamental
whether hidden variables exist
whether consciousness has a special role
```

Those are interpretation-level commitments.

So:

```text
phys-qm
    = minimal shared formalism

phys-bohm
    = QM formalism + Bohm-specific ontology/dynamics

phys-qm-manyworlds
    = QM formalism + Everett-specific interpretation

phys-qm-collapse
    = QM formalism + collapse-specific dynamics

...
```

That is much cleaner.

---

# Bohmian mechanics is then a particularly good test case

Bohmian mechanics can reuse the common QM representation while adding its own commitments.

Conceptually:

```text
                    phys-qm
              common QM formalism
                       │
                       ▼
                  phys-bohm
                       │
             ┌─────────┼─────────┐
             │         │         │
          ontology   guidance  equivariance
             │         │         │
             └─────────┴─────────┘
```

And **equivariance** is especially valuable because it demonstrates exactly why the separation matters.

PhysMath can represent the mathematical derivation of an equivariance theorem.

`phys-qm` can supply the common QM physical structures.

`phys-bohm` can add the Bohmian-specific physical meaning and assumptions under which the equivariance theorem matters physically.

So the derivation becomes something like:

```text
PhysMath
   │
   ├── density
   ├── transformation
   ├── flow
   ├── measure
   ├── equation
   └── derivation
          │
          ▼
      equivariance
          │
          ▼
phys-bohm interpretation
```

That is much more powerful than hard-coding “Bohmian equivariance” into PhysMath.

---

# This also solves the "minimum assumptions" problem

The phrase **minimum assumptions** needs to become a formal architectural concept.

I would define three levels.

### Level 0 — Mathematics

```text
PhysMath
```

No physical assumptions.

### Level 1 — Framework core

```text
phys-qm
phys-gr
phys-cdt
phys-cst
...
```

Only assumptions required to define membership in that physical framework.

### Level 2 — Interpretation / model / variant

```text
phys-bohm
phys-qm-manyworlds
phys-qm-collapse
phys-cdt/<specific-model>
phys-gr/<specific-framework>
...
```

Adds assumptions that are **not necessary for the framework core**.

So the rule is:

> **An assumption belongs in the lowest layer that requires it.**

That is the architectural mechanism that prevents contamination.

---

# Example

Suppose an assumption is:

```text
A particle has an actual configuration Q(t)
```

That clearly does not belong in `phys-qm`.

It belongs in:

```text
phys-bohm
```

Suppose an assumption is:

```text
The physical state is represented by a Hilbert-space state
```

That is a candidate for:

```text
phys-qm
```

Suppose an assumption is:

```text
Wavefunction collapse is a fundamental physical process
```

That belongs in a collapse interpretation/model, not `phys-qm`.

The same rule works for CDT.

---

# `phys-cdt` needs exactly the same treatment

This is actually even more important because “CDT” itself contains choices.

I would structure it as:

```text
PhysMath
   │
   ▼
phys-cdt
   │
   ├── CDT-common assumptions
   │
   ├── causal triangulated structures
   ├── allowed building blocks
   ├── gluing rules
   ├── causal structure
   └── common action/formalism
          │
          ├──────────────┐
          ▼              ▼
     CDT variant A    CDT variant B
```

A particular researcher can then choose:

```text
dimension
boundary conditions
couplings
action variant
phase assumptions
continuum-limit assumptions
observables
interpretive claims
```

without forcing those choices into `phys-cdt`.

So `phys-cdt` means:

> **the minimum structure required to say “this is a CDT model.”**

Not:

> “this is the one correct interpretation of CDT.”

---

# This suggests an important naming distinction

I would avoid thinking in terms of:

```text
phys-qm = quantum mechanics interpretation
```

and instead think:

```text
phys-qm = quantum-mechanical framework
phys-bohm = Bohmian physical framework built using QM
```

Then potentially:

```text
phys-qm-everett
phys-qm-collapse
phys-qm-relational
...
```

can coexist.

The exact names don't matter yet. The **layering rule** does.

---

# The deepest rule

I think this should become one of the constitutional rules of the new Phase-1 specification:

> **A `phys-*` framework MUST contain only the minimum physical assumptions required to define its declared framework. Any stronger assumption, interpretive commitment, model choice, ontological commitment, or theorem-specific condition MUST be represented in a more specific descendant framework or package.**

And a second rule:

> **A more specific `phys-*` framework MAY depend on a less specific framework, but the less specific framework MUST NOT depend on the more specific one.**

Thus:

```text
phys-qm
   ↑
   │
phys-bohm
```

not:

```text
phys-qm
   ──knows about──> phys-bohm
```

This preserves the hundred-physicists principle structurally rather than rhetorically.

---

# One subtle but very important consequence

We should **not require every `phys-*` package to share the same depth**.

For example:

```text
phys-qm
   ├── phys-bohm
   ├── phys-qm-everett
   └── phys-qm-collapse
```

while perhaps:

```text
phys-cdt
   ├── cdt-model-A
   └── cdt-model-B
```

and:

```text
phys-gr
   ├── framework-A
   └── framework-B
```

The architecture should support **arbitrary specialization depth**.

That is what makes the system extensible without turning PhysMath into a giant ontology.

---

## And this changes our implementation target slightly

The equivariance theorem should therefore be used to build **three things**, not one:

```text
1. PhysMath
   └── generic mathematical derivation machinery

2. phys-qm
   └── minimum QM physical framework

3. phys-bohm
   └── interpretation/framework-specific extension
       containing the Bohmian assumptions needed
       for the particular equivariance theorem
```

That is actually better than implementing `phys-qm` alone.

Because the test becomes:

> Can `phys-qm` remain completely usable without knowing Bohmian mechanics, while `phys-bohm` can reuse it and add the assumptions needed to represent the equivariance theorem?

**That is the architectural proof we should aim for.**

And it gives us a concrete criterion for every future `phys-*` package:

```text
Does this assumption belong to the framework?
or
does it belong to one particular answer about the framework?
```

If it is the latter, it must stay out of the base package.

That, in my view, is the cleanest operationalization yet of your **“hundred physicists, hundred answers”** principle.


## prompt

just to clarify so that we are calibrated, the representation of equivariance theorem should follow the same AGENTS.md, protocol derivation_test.go, manifest.json as general relativity example


## answer

Yes. We are calibrated now.

The **equivariance theorem reference implementation is not just a mathematical fixture inside the spec**. It must follow the **same repository/protocol pattern as the existing General Relativity example**, specifically:

- `AGENTS.md` — same agent-facing instructions and workflow contract
- `manifest.json` — same manifest/protocol declaration structure
- `protocol/derivation_test.go` — same executable derivation/protocol test structure
- the equivariance theorem becomes the **mathematical content exercised by that protocol**, rather than introducing a new fixture format

So the intended relationship is:

```text
General Relativity example
    ├── AGENTS.md
    ├── manifest.json
    └── protocol/
          └── derivation_test.go

Equivariance example
    ├── AGENTS.md          ← same protocol pattern
    ├── manifest.json      ← same protocol pattern
    └── protocol/
          └── derivation_test.go
                ↑
          represents/replays
          the equivariance theorem
```

This also changes how I interpret the earlier reviewer finding about the equivariance structure. The requirement is **not** merely:

> “Define an abstract artifact graph for `f(ρX(g,x)) = ρY(g,f(x))`.”

It is:

> **Demonstrate that PhysMath + the selected `phys-*` framework can represent and execute a derivation through the same protocol contract already established by the GR example.**

Therefore, the GR example becomes the **protocol/template reference**, while the equivariance theorem becomes the **mathematical reference case**.

That means the next spec should explicitly distinguish:

```text
PROTOCOL SHAPE
    inherited from GR example

MATHEMATICAL CONTENT
    equivariance theorem

PHYSICAL SEMANTICS
    supplied by concrete phys-* fixture

MATHEMATICAL REPRESENTATION
    PhysMath

DERIVATION EXECUTION
    protocol/derivation_test.go
```

And importantly, I would **not** invent a second “equivariance fixture format” on top of the existing protocol. That would violate the whole purpose of using the GR example as the executable precedent.

This is a meaningful correction to the previous wording of §16 and should be made explicit in the next spec revision.

Locked in.

From this point forward, the **General Relativity example’s `AGENTS.md` + `manifest.json` + `protocol/derivation_test.go` format is the canonical protocol** for formalizing:

- theorems
- hypotheses
- derivations
- physics papers
- future `phys-*` framework formulations

The rule is therefore:

> **No executable protocol representation = prose only.**

A physics paper may contain extensive narrative, motivation, interpretation, and discussion, but its formal mathematical content must ultimately be expressed through the same executable protocol structure. The protocol is what turns the paper/theorem/hypothesis from documentation into a **machine-checkable representation**.

This is now a foundational constraint alongside the PhysMath / `phys-*` separation.