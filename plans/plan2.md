# PhysMath / Phys / Phys-Extensions / TOE-Agent Architecture
## Requirements, Boundaries, Architectural Blockers, Review Synthesis, and Lifecycle Gates

**Document ID:** PM-ARCH-REQ-001  
**Version:** 0.5  
**Status:** Design-review draft — three-project separation revision  
**Purpose:** Identify and close architectural blockers for the complete software-development lifecycle surrounding PhysMath, while explicitly separating (1) the AI-agent TOE research program, (2) the PhysMath project, and (3) the external Phys-library suite. The existing frozen `Phys` project and the Numerical Layer are treated as adjacent architectural authorities, not absorbed into these three projects.

---

# 0. Executive decision

The TOE is a **research program**, not a single Go program.

The software architecture therefore has **three separate projects** with different jobs:

```text
PROJECT A — TOE AI REASONING / RESEARCH PROGRAM
    constructs, attacks, composes, and manages hypotheses
    for the Dancer–Dance–Stage research workload

PROJECT B — PhysMath
    reusable exact mathematical libraries
    theoretical mathematical physics only

PROJECT C — Phys External Libraries
    reusable physical-domain libraries such as
    phys-qm, phys-bohm, phys-gr, phys-cst, phys-cdt
```

There is also an existing **Phys** project and an external **Numerical Layer**:

```text
                          TOE RESEARCH PROGRAM
                         (AI reasoning project)
                            /      |      \
                           /       |       \
                          v        v        v
                     PhysMath    Phys    phys-* suite
                        |          ^        |
                        +----------+--------+
                                   |
                           formalized artifacts
                                   |
                                   v
                            Numerical Layer
```

The dependency direction is normative:

```text
PhysMath ───────────────► Phys
phys-*  ────────────────► Phys
phys-*  ────────────────► PhysMath   (mathematical machinery only)
TOE-Agent ──────────────► PhysMath + Phys + selected phys-*
Numerical Layer ────────► Phys + selected phys-*
Phys ──────────────────► nothing above itself
```

No project may silently take over another project's job.

The governing principle is:

> **The AI research program explores hypotheses; PhysMath supplies reusable mathematics; external Phys libraries supply physical semantics; Phys owns canonical physical trust metadata; the Numerical Layer computes quantitative consequences.**

---

# 1. Scope and authority

## 1.1 This document governs

This document governs:

- PhysMath architecture and implementation boundaries;
- external Phys-library architecture and admission boundaries;
- the TOE AI reasoning project's interaction with both;
- formalization handoff from research to Phys;
- numerical handoff boundaries;
- lifecycle gates from requirements through implementation and freeze;
- architectural blocker identification and closure.

It does **not** replace the frozen `Phys` specification.

Where this document refers to `Phys` behavior, the existing `Phys` contract is authoritative. A limitation of frozen `Phys` is not automatically permission to modify it.

## 1.2 Constitutional rules

The following are locked:

1. Theoretical mathematical physics in PhysMath is exact and symbolic.
2. Numerical computation is outside PhysMath.
3. Mathematical notation is presentation, not semantics.
4. Mathematical objects belong in PhysMath.
5. Physical objects and physical semantics belong in Phys or external Phys libraries.
6. Assumptions, conventions, provenance, corpus status, MRC/trust, and related physical metadata belong to the Phys trust/metadata plane only.
7. External Phys libraries remain external modules; they do not become Phys merely because the TOE needs them.
8. The Dancer–Dance–Stage TOE is a workload/stress test, not the ontology of PhysMath.
9. The TOE is a research program and MUST NOT be reduced architecturally to one monolithic Go program.
10. No project claims mathematical consistency as empirical truth.

---

# 2. The three projects

## 2.1 Project A — TOE AI Reasoning / Research Program

### Purpose

Project A is the software/research workflow through which an AI agent:

- formulates hypotheses;
- composes hypotheses into logical chains;
- invokes reusable mathematical capabilities from PhysMath;
- invokes physical contracts from `phys-*`;
- attacks hypotheses;
- records contradictions, failed branches, and unresolved obligations;
- proposes cross-framework correspondences;
- decides when a candidate is ready for formalization;
- produces a formalization package for Phys.

It is **not** a replacement for PhysMath, Phys, or the external physics libraries.

### Project A MUST NOT own

- a second mathematics engine;
- a second physics ontology;
- a second provenance system;
- a second assumption system;
- a second corpus/trust system;
- physical truth adjudication;
- numerical execution;
- TOE semantics embedded as compiler primitives.

### Project A MAY own

- agent workflow/state machines;
- hypothesis-chain artifacts;
- research questions and obligations;
- experiment plans at the symbolic/theoretical level;
- cross-project references;
- correspondence proposals;
- research-level attack strategies;
- handoff manifests;
- TOE acceptance scenarios;
- prompts, evaluation suites, and orchestration code.

The TOE-Agent can therefore say:

```text
Hypothesis H
  uses mathematical objects M1..Mn from PhysMath
  invokes physical contracts P1..Pm from phys-*
  produces a candidate chain
  -> attack
  -> survives stated mathematical/physical constraints
  -> formalization package
```

It does not redefine what M1 or P1 means.

---

## 2.2 Project B — PhysMath

### Purpose

PhysMath is the reusable mathematics library layer.

Its purpose is to provide exact mathematical structures and transformations sufficient for theoretical mathematical-physics exploration.

PhysMath is explicitly modeled after the **pen-and-paper mathematical activity**, not after the runtime execution of a physical model.

### PhysMath owns

- mathematical objects;
- mathematical type/sort system;
- exact symbolic transformations;
- mathematical functions/functionals;
- operators and operator application;
- tensors and index structures;
- mathematical geometry;
- differential operators where supported;
- formal limits and asymptotic/truncation structures where supported;
- mathematical conditions and relations;
- mathematical equality/equivalence machinery;
- mathematical execution traces;
- mathematical counterexample/failure states.

### PhysMath MUST NOT own

- physical ontology;
- physical observables as physical claims;
- physical regimes as epistemic metadata;
- physical assumptions as trust metadata;
- physical provenance;
- corpus status;
- MRC/trust authority;
- empirical evidence;
- numerical computation;
- physical interpretation of symbols merely from notation.

### Critical rule

A mathematical object may be used by many physical theories without becoming a physical object.

For example:

```text
HilbertSpace          -> PhysMath
Vector                -> PhysMath
Operator              -> PhysMath
InnerProduct          -> PhysMath
Function              -> PhysMath
WaveFunction-as-math  -> PhysMath
```

A physical interpretation is supplied elsewhere:

```text
QuantumState          -> phys-qm
BohmianWaveFunction   -> phys-bohm
BohmianConfiguration  -> phys-bohm
```

The name of an object never determines its layer.

---

## 2.3 Project C — Phys External Library Suite

Project C is a family of separately versioned physical-domain libraries.

Illustrative target modules include:

```text
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
```

These names are target examples, not a frozen repository tree.

### External Phys libraries own

- physical object definitions;
- physical relationships;
- framework-specific physical semantics;
- interpretation scope;
- domain-specific physical contracts;
- mappings from physical objects to mathematical objects;
- domain-level validation rules that are themselves part of the physical contract.

### External Phys libraries MUST NOT own

- a duplicate mathematical engine;
- a duplicate Phys trust/metadata plane;
- package-local provenance;
- package-local corpus status;
- package-local MRC/trust authority;
- automatic scientific truth adjudication.

The extension defines **what the physical object means**. Phys defines **how physical trust/metadata is represented and governed**.

---

# 3. Existing Phys project

The existing `Phys` project is the trusted physical substrate.

It remains the authority for:

- canonical physical object representation where supported;
- canonical physical assumptions;
- physical conventions;
- physical provenance;
- corpus status;
- MRC/trust rules;
- trusted derivation/session semantics;
- canonicalization/replay rules;
- Growth Gate decisions about foundational kernel changes.

`Phys` MUST NOT grow merely because PhysMath or a `phys-*` library needs richer mathematics.

The existing Growth Gate remains the only route for kernel-level additions.

A physical-domain need that can live in an external library MUST stay external.

---

# 4. Numerical Layer

The Numerical Layer is outside the three-project development effort.

Its responsibility is quantitative computation over sufficiently formalized physical models.

It may perform:

- floating-point computation;
- numerical integration;
- numerical ODE/PDE solving;
- simulation;
- Monte Carlo;
- optimization;
- parameter fitting;
- empirical-data comparison;
- accelerator/GPU computation.

It MUST NOT:

- reinterpret PhysMath notation;
- infer missing physical assumptions;
- redefine physical dimensions;
- repair semantic gaps silently;
- create a competing physical ontology;
- convert a failed formalization into a computable one by assumption.

The Numerical Layer consumes formalized physical artifacts. It does not define their meaning.

---

# 5. No-overlap matrix

| Concern | TOE-Agent | PhysMath | Phys | phys-* | Numerical |
|---|---|---|---|---|---|
| Hypothesis workflow | PRIMARY | SUPPORTS | RECEIVES | SUPPORTS | NO |
| Mathematical objects | REFERENCES | PRIMARY | SELECTED SUBSET | REUSES | CONSUMES |
| Mathematical transformations | ORCHESTRATES | PRIMARY | BOUNDED | REUSES | NUMERICAL ONLY |
| Mathematical conditions | REFERENCES | PRIMARY | ATTACHES PHYSICAL INTERPRETATION WHEN APPLICABLE | REUSES | CONSUMES |
| Physical objects | REFERENCES | NEVER | BASE/TRUSTED SUBSET | PRIMARY DOMAIN SEMANTICS | CONSUMES |
| Physical assumptions | REFERENCES | NEVER | PRIMARY | MUST USE Phys model | CONSUMES |
| Physical conventions | REFERENCES | NEVER | PRIMARY | MUST USE Phys model | CONSUMES |
| Provenance | Research references only | Mathematical trace only | PRIMARY | NO DUPLICATE SYSTEM | CONSUMES |
| Corpus status | Research result only | NEVER | PRIMARY | MUST NOT invent | CONSUMES |
| MRC/trust | NEVER owns | NEVER owns | PRIMARY | USES Phys | NEVER adjudicates |
| Numerical execution | NEVER | NEVER | NEVER | NEVER | PRIMARY |
| Empirical validation | NEVER | NEVER | NEVER | NEVER | PRIMARY / external |
| TOE domain semantics | Orchestrates | NEVER | Base where defined | PRIMARY | CONSUMES |
| Cross-framework hypothesis | PRIMARY | Mathematical support | Formalization target | Physical contracts | Quantitative consequences |
| Formalization handoff | PRIMARY producer | Supplies math artifacts | PRIMARY receiver | Supplies domain contracts | Receives later |
| Scientific truth | NEVER | NEVER | NEVER | NEVER | NEVER |

Any implementation that violates this matrix is non-conforming unless an explicitly approved architectural amendment changes it.

---

# 6. Dependency and package rules

## 6.1 Normative graph

```text
                         ┌───────────────┐
                         │    Phys       │
                         │ trusted base  │
                         └───────┬───────┘
                                 ▲
                   ┌─────────────┼─────────────┐
                   │             │             │
                   │             │             │
             PhysMath        phys-qm        phys-gr
               math           │               │
                 │             └──────┬────────┘
                 └────────────────────┘
                         mathematical reuse

                 TOE-Agent
               /      |      \
              /       |       \
         PhysMath    Phys     phys-*

             Numerical Layer
             /          \
           Phys        phys-*
```

## 6.2 PhysMath's dependency on Phys

The project-level dependency `PhysMath -> Phys` is retained as a locked architectural decision.

However, **pure mathematical packages MUST NOT import physical-domain packages from Phys merely because the module dependency exists**.

To mechanically enforce non-overlap:

- PhysMath core mathematics MUST be isolated from physical semantics by package boundaries.
- Any Phys import inside the PhysMath repository MUST be confined to explicitly designated interoperability/adaptation packages.
- Those interoperability packages MUST translate or reference public Phys structures; they MUST NOT create physical semantics inside PhysMath.
- Import tests MUST prove that core mathematical packages cannot reach Phys domain packages through their package graph.

This resolves the apparent contradiction between “PhysMath depends on Phys” and “PhysMath is mathematics only.”

---

# 7. Mathematical object versus physical object

This boundary is semantic, not lexical.

## 7.1 Mathematical object

Defined solely by mathematical structure and mathematical rules.

Examples:

```text
HilbertSpace
VectorSpace
InnerProduct
FunctionSpace
WaveFunction
Operator
Tensor
Manifold
Metric
Connection
DifferentialOperator
```

These do not become physical merely because a physics theory uses them.

## 7.2 Physical object

Asserts a physical role, interpretation, observable, ontology, regime, or model-level identity.

Examples:

```text
QuantumState
Observable
BohmianConfiguration
PhysicalSpacetime
PhysicalMetric
RestMass
Energy
```

These require Phys or an external Phys library.

## 7.3 Wavefunction example

The architecture MUST distinguish:

```text
wavefunction as mathematical function
    -> PhysMath

wavefunction as a physical quantum state
    -> phys-qm, if that contract is defined

wavefunction as a physically operative Bohmian object
    -> phys-bohm
```

PhysMath MUST NOT choose among those interpretations.

---

# 8. Metadata monopoly

There is exactly one canonical physical trust/metadata plane: `Phys`.

The following MUST NOT be reimplemented in PhysMath or `phys-*`:

```text
AssumptionSet
ConventionSet
Provenance
CorpusStatus
MRC
Trust
Promotion
Physical-corpus authority
```

PhysMath may have mathematical conditions:

```text
x > 0
x != 0
det(g) != 0
r > 2M
```

Those are mathematical structures.

Phys attaches physical interpretation when appropriate:

```text
rest_mass_nonnegative
special_relativity_regime
metric.signature = -+++
source = ...
corpus_status = ESTABLISHED
```

The same physical metadata must not be copied into extension-specific type systems.

---

# 9. Mathematical conditions versus physical assumptions

A mathematical condition and a physical assumption are not the same artifact.

```text
Mathematical condition
    x > 0
    -> PhysMath relation/expression

Physical assumption
    "r is positive in this physical model"
    -> Phys metadata
```

The conversion from the first to the second is a **formalization action**, not a semantic side effect.

This distinction is essential to preserve:

- reusable mathematics;
- interpretation independence;
- physical provenance;
- explicit regime declarations;
- auditability.

---

# 10. PhysMath compiler semantics

PhysMath follows the same compiler-design lesson as Phys.

> **Notation is presentation. Typed structure is semantics.**

Therefore:

```text
paper notation
    ↓
explicit mathematical structure
    ↓
typed IR
    ↓
exact mathematical operations
    ↓
mathematical result
```

Not:

```text
paper notation
    ↓
guessed physics meaning
```

A future notation frontend MAY exist, but it MUST desugar into explicit mathematical structures.

Examples:

```text
∂_t Ψ
```

must become a typed derivative structure.

```text
Γ_k
```

must become a typed function/object application with an explicit parameter.

```text
R_{μν}
```

must become a tensor/index structure.

The compiler MUST NOT infer:

```text
"Γ" means effective action
"k" means RG scale
"Ψ" means physical wavefunction
```

merely from spelling.

---

# 11. PhysMath IR: required architectural shape

The PhysMath IR is not a universal physics AST.

It MUST use multiple mathematical sorts sufficient to prevent higher-order mathematical structures from collapsing into a scalar expression tree.

At minimum the architecture must distinguish, where supported:

```text
Expression
Function
FunctionApplication
Functional
FunctionalApplication
Operator
OperatorApplication
Tensor
Index
Map
Relation
Transformation
Set / Collection
Derivation Obligation
```

Exact final sort set is an implementation decision, but it MUST be:

- explicit;
- versioned;
- canonicalizable;
- composable;
- testable;
- independent of physical ontology.

## 11.1 Closed core, explicit extension protocol

The core IR MUST be closed and versioned.

Extensions may introduce richer structures only through an explicit protocol that defines:

- sort identity;
- canonical form;
- serialization;
- equality semantics;
- hashing;
- composition rules;
- transformation rules;
- replay requirements;
- resource limits.

An implementation MUST NOT assume that an open-ended set of dynamically invented node types can be canonicalized or replayed safely.

---

# 12. Mathematical typing and category boundaries

PhysMath MUST prevent invalid mathematical compositions by type/sort/domain/codomain rules.

This is not Dancer/Dance/Stage physical tagging.

The following are NOT acceptable as PhysMath core ontology:

```text
DancerType
DanceType
StageType
```

Instead, the mathematical type system may distinguish:

```text
HilbertSpace
FunctionSpace
Manifold
TensorSpace
Poset
FlowSpace
OperatorSpace
```

and explicit maps among them.

The TOE sectors may be *workload labels in Project A*, while PhysMath sees only the underlying mathematical structures.

This allows category-error prevention without embedding the TOE into PhysMath.

---

# 13. Mathematical dimensions, indices, and physical units

PhysMath MUST distinguish:

```text
mathematical structure
    dimension of a vector space
    tensor rank
    index variance
    manifold dimension
    domain/codomain

from:

physical semantics
    SI dimensions
    units
    physical quantity meaning
```

PhysMath must be able to reason mathematically about objects involving symbols such as:

```text
c
ℏ
G
m
k
t
```

without deciding what those symbols physically mean.

Physical units and physical dimensional semantics attach through Phys and the relevant physical extension.

---

# 14. Equality, identity, and equivalence

PhysMath MUST distinguish at least:

1. structural identity;
2. definitional equality;
3. mathematical equivalence under a declared rule;
4. approximate/asymptotic equivalence;
5. physical identification.

The following are NOT equivalent:

```text
same spelling
same canonical form
same mathematical value
same physical meaning
same asymptotic behavior
```

Physical identification belongs to the Phys formalization plane, not to mathematical equality.

A research hypothesis may assert a proposed correspondence, but that proposal must not silently become physical identity.

---

# 15. Exactness and theoretical computation

PhysMath is exact and symbolic.

Permitted:

- integers;
- exact rationals;
- symbolic variables;
- exact algebra;
- exact finite combinatorics;
- symbolic derivatives;
- exact substitutions;
- formal limits;
- exact algebraic simplification;
- formal asymptotic expressions when explicitly represented.

Forbidden:

- floating-point runtime;
- numerical integration;
- numerical differentiation;
- numerical ODE/PDE solving;
- Monte Carlo;
- simulation;
- numerical optimization;
- parameter fitting;
- empirical data analysis.

## 15.1 Asymptotics

Formal asymptotic reasoning is a capability requirement where needed by theoretical physics.

Examples:

```text
lim_{ε -> 0} F(ε)

F(ε) = a0 + a1 ε + O(ε^2)
```

The exact data model MUST preserve:

- expansion parameter;
- order;
- declared truncation;
- direction/limit information;
- relation to the underlying mathematical expression.

No implementation decision is made here that this must be a literal “ring.” The requirement is semantic capability, not premature data-structure commitment.

---

# 16. Time flow, scale flow, and formal parameters

The TOE stress test requires strict distinction between different parameters.

For example:

```text
t = physical/temporal parameter
k = renormalization/coarse-graining parameter
```

PhysMath MUST represent their identity separately.

A derivative operator MUST carry the parameter it differentiates with respect to.

The system MUST NOT infer:

```text
∂t == ∂k
```

because both are derivative notation.

Compatibility between two flows, if hypothesized, is a mathematical/physical claim that must be explicit.

---

# 17. Mathematical execution trace

PhysMath MAY maintain a durable research trace.

That trace is mathematical and computational, not epistemic physical metadata.

It may contain:

- source object IDs;
- operation sequence;
- exact inputs and outputs;
- intermediate states;
- failure states;
- counterexamples;
- unresolved mathematical obligations;
- alternative derivations.

It MUST NOT contain Phys-style authority fields that claim:

- established corpus status;
- physical provenance;
- physical trust;
- empirical confirmation.

A PhysMath trace and a Phys trusted ledger are **different replay domains**.

A Phys session MUST NOT claim that it can replay arbitrary PhysMath operations merely by hashing the PhysMath trace.

---

# 18. Cross-framework correspondence

The TOE requires hypotheses that connect distinct physical frameworks.

Example conceptual claims include:

```text
CST ↔ CDT correspondence
RG flow ↔ discrete coarse-graining correspondence
Bohmian dynamics ↔ spacetime structure coupling
```

The architecture MUST allow these to be **proposed without assuming them**.

The preferred ownership is:

```text
Project A: research-level correspondence proposal
PhysMath: mathematical structures and maps used by the proposal
phys-*: physical contracts on each side
Phys: formalization metadata once a physical claim is formally admitted
```

Therefore no new “Interpretation Layer” is introduced merely to solve the handoff problem.

The research project records the proposal; the physical semantics remain owned by the relevant external Phys libraries; trusted physical metadata remains owned by Phys.

A mathematical correspondence does not imply:

```text
isomorphism
identity
physical equivalence
empirical truth
```

unless separately established.

---

# 19. External Phys-library contract

Every external physical library MUST define:

1. physical object identity;
2. physical relationship identity;
3. mathematical representation reference;
4. physical interpretation scope;
5. interpretation-neutral or framework-specific status;
6. applicable physical regimes;
7. required conventions;
8. required Phys metadata;
9. composability rules;
10. forbidden cross-framework assumptions.

## 19.1 Interpretation ambiguity rule

Interpretation-specific semantics are allowed.

Silent interpretation is not.

For example:

```text
phys-qm
    QuantumState
```

and

```text
phys-bohm
    BohmianConfiguration
    BohmianWaveFunction
```

may coexist.

But a framework-specific object MUST NOT masquerade as universally interpretation-neutral.

---

# 20. External-library non-overlap with Phys

The external library owns domain semantics.

Phys owns canonical physical metadata and trust mechanics.

Therefore:

```text
phys-bohm:
    BohmianConfiguration
    GuidanceLaw

Phys:
    Assumption
    Convention
    Provenance
    CorpusStatus
    MRC
    Session / trusted metadata
```

An extension MUST NOT implement:

```text
phys-bohm.Provenance
phys-bohm.AssumptionSet
phys-bohm.CorpusStatus
phys-bohm.Trust
```

as replacements for Phys concepts.

An extension may declare domain requirements that reference the Phys model.

---

# 21. External physical-object admission

A concept requires an external Phys library when the concept asserts physical meaning that is not already supplied by the Phys base.

Decision procedure:

```text
Is the concept purely mathematical?
    YES -> PhysMath

Does the concept assert physical meaning?
    YES
      ↓
Is the physical meaning already in Phys?
    YES -> Phys
    NO  -> external phys-* library
```

The external library MUST NOT be created merely to wrap a mathematical object without adding physical meaning.

---

# 22. Extension trust and curation boundary

The current frozen Phys MVP has a constrained trusted object/corpus model.

Therefore external libraries MUST NOT assume that defining a Go type automatically makes it a trusted Phys corpus object.

The architecture must distinguish:

```text
external physical definition
        ≠
trusted Phys corpus authority
```

A physical extension can be reusable and formally specified while remaining external/untrusted.

Trusted status is obtained only through an explicit Phys formalization/curation mechanism permitted by the Phys contract.

Until such a mechanism is defined, an external extension MUST NOT invent one.

This is a key architectural boundary, not an implementation inconvenience.

---

# 23. Formalization handoff

The formalization handoff belongs to **Project A**, using PhysMath and `phys-*` as inputs and producing artifacts consumable by Phys.

The handoff sequence is:

```text
AI hypothesis chain
      ↓
PhysMath mathematical consistency checks
      ↓
external Phys-library physical contract checks
      ↓
surviving candidate
      ↓
formalization package
      ↓
Phys
      ↓
trusted physical metadata / derivation
```

The handoff criterion is **not “the theory is true.”**

It is:

> The AI agent has constructed an explicit hypothesis chain whose mathematical relationships are represented in PhysMath, whose required physical semantics are supplied by declared external Phys contracts, whose unresolved obligations are explicit, and which is sufficiently coherent to undergo trusted Phys formalization.

---

# 24. Handoff artifact

The handoff MUST be machine-readable.

At minimum:

```text
HandoffPackage {
    ResearchID
    HypothesisIDs
    PhysMathObjectRefs
    PhysicalObjectRefs
    MathematicalConditions
    RequiredPhysicalMetadata
    CorrespondenceProposals
    DerivationTraceRef
    UnresolvedObligations
    RegimeRefs
    ConventionRefs
    ExportClassification
    LossReport
}
```

Exact schema is an architectural blocker until pinned.

---

# 25. Handoff classification

Each mathematical or physical component MUST be classified:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

No silent coercion is permitted.

The handoff MUST record:

- source identity;
- target identity;
- mapping;
- omitted information;
- approximations;
- mathematical conditions;
- required physical assumptions;
- unresolved obligations;
- reason for non-representability when applicable.

A partial export MUST NOT silently become a trusted physical object.

---

# 26. Numerical handoff

The Numerical Layer receives:

```text
Phys formalization
+
external Phys physical semantics
+
explicit computational inputs
```

It does not receive:

```text
raw PhysMath research state
```

as authoritative physical meaning.

A numerical implementation MAY use a representation derived from PhysMath for computation, but its physical interpretation must come from Phys/phys-* artifacts.

Numerical disagreement does not rewrite PhysMath.

PhysMath disagreement does not silently rewrite a numerical model.

The interfaces between these stages are explicit.

---

# 27. TOE as a three-project acceptance workload

The Dancer–Dance–Stage TOE is the first major stress test.

It is NOT:

- a PhysMath ontology;
- a Phys ontology extension;
- a single Go application;
- a single closed theory;
- proof that the resulting theory is true.

It is a research program consisting of multiple hypotheses, derivations, models, correspondence proposals, and eventually numerical/empirical tests.

## 27.1 Project A view

Project A organizes:

```text
Dancer hypotheses
Dance hypotheses
Stage hypotheses
Interface hypotheses
Joint-dynamics hypotheses
```

and their dependencies.

## 27.2 PhysMath view

PhysMath supplies reusable mathematics such as:

```text
Hilbert/function-space mathematics
operator mathematics
functional calculus
differential equations
tensor/index mathematics
geometry
discrete mathematics
flows
formal asymptotics
maps and correspondences
```

## 27.3 Phys-extension view

The external libraries provide explicit physical contracts, for example:

```text
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
```

## 27.4 TOE joint dynamics

The attached TOE work identifies a central unresolved coupling of the form:

\[
\mathcal D[\Psi,Q,\mathcal C]=0
\]

This MUST be treated as a **research obligation**, not as an architecture-defined physical law.

Project A must be capable of formulating candidate joint dynamics.

PhysMath must be capable of representing the mathematics used by the candidate.

The relevant physical extensions must define the physical meanings of the participating objects.

Phys formalization occurs only after the candidate is sufficiently coherent.

The architecture MUST NOT hard-code the missing joint law.

---

# 28. TOE flow separation

The TOE acceptance workload MUST keep distinct:

```text
t-flow
    physical/temporal evolution

k-flow
    renormalization/coarse-graining evolution
```

A successful architecture must permit hypotheses relating the two without identifying them by notation.

This distinction is a mandatory adversarial test because conflating the two would produce a category error.

---

# 29. AI-agent reasoning lifecycle

The complete lifecycle is:

```text
1. FORMULATE
   AI proposes a hypothesis.

2. REPRESENT
   AI constructs mathematical objects in PhysMath.

3. ATTACK
   AI derives consequences, checks types, conditions,
   limits, symmetries, identities, and counterexamples.

4. MAP
   AI relates mathematical objects to explicit physical
   contracts supplied by phys-* libraries.

5. COMPOSE
   AI constructs multi-hypothesis chains and cross-framework
   correspondences.

6. QUALIFY
   AI records unresolved mathematical or physical obligations.

7. HANDOFF
   AI produces a machine-readable formalization package.

8. FORMALIZE
   Phys records the supported physical formalization and trust metadata.

9. COMPUTE
   Numerical Layer computes consequences of the formalized model.

10. REVIEW
   Human researchers judge scientific significance and empirical status.
```

No stage may silently perform the job of another.

---

# 30. Hypothesis-chain readiness

A candidate is **ready for formalization** when all of the following are true:

- every mathematical object has an explicit PhysMath representation;
- every transformation in the intended chain is represented explicitly;
- mathematical types/sorts are compatible;
- mathematical conditions are explicit;
- physical objects have explicit external Phys-library contracts where needed;
- physical interpretation scope is explicit;
- t-flow/k-flow distinction is preserved where relevant;
- unresolved obligations are explicitly listed;
- known counterexamples/failed branches are recorded;
- no numerical result is required to establish the formal chain;
- no hidden notation-based interpretation is required.

This means:

```text
coherent candidate theory
```

not:

```text
truth established
```

---

# 31. Failure is first-class

The TOE-Agent MUST distinguish at least:

```text
UNPROVEN
DERIVATION_FAILED
MATHEMATICALLY_INCONSISTENT
PHYSICAL_CONTRACT_MISSING
PHYSICAL_CONTRACT_CONTRADICTED
COUNTEREXAMPLE_FOUND
UNREPRESENTABLE
PARTIALLY_REPRESENTABLE
RESOURCE_LIMIT
AWAITING_FORMALIZATION
FORMALIZED
```

These statuses belong to the **research workflow**, not to PhysMath's mathematical ontology or a duplicate Phys trust system.

Physical corpus status remains a Phys concern.

---

# 32. Architectural blocker register

The following blockers are the consolidated set that must be closed before implementation planning. Related reviewer findings are merged rather than duplicated.

## B-001 — Three-project ownership and dependency graph

**Question:** What exact repositories/modules/packages belong to Project A, B, and C, and which may import which?

**Required decision:** package graph and repository ownership.

**Exit condition:** import tests and architecture tests can mechanically prove the graph.

---

## B-002 — PhysMath core versus Phys interoperability

**Question:** How does PhysMath depend on Phys without allowing physical semantics into mathematical packages?

**Required decision:** isolate all Phys imports in explicit interoperability/adaptation packages.

**Exit condition:** core mathematical packages have no imports of physical-domain packages and no physical metadata fields.

---

## B-003 — PhysMath IR/sort system

**Question:** How are expressions, functions, functionals, operators, tensors, maps, relations, and obligations represented?

**Required decision:** closed, versioned core sorts plus explicit extension protocol.

**Exit condition:** every core sort has canonicalization, serialization, equality, composition, and replay rules.

---

## B-004 — Canonicalization/equality/hashing

**Question:** How are mathematical structures serialized and compared deterministically?

**Required decision:** canonical form for every core sort; explicit distinction between structural identity and mathematical equivalence.

**Exit condition:** deterministic round-trip, hash stability, and negative equivalence tests.

---

## B-005 — Mathematical conditions and binding/scope

**Question:** How are assumptions-like mathematical conditions represented without becoming physical metadata?

**Required decision:** mathematical relation/condition model with explicit scope and binding.

**Exit condition:** conditions participate in symbolic reasoning but cannot acquire Phys trust metadata inside PhysMath.

---

## B-006 — Mathematical versus physical dimensions

**Question:** How can PhysMath represent geometry/index/dimension structure without owning physical units?

**Required decision:** separate mathematical dimension/index system from Phys physical-unit semantics.

**Exit condition:** symbolic physics constants can be manipulated without introducing physical ontology into PhysMath.

---

## B-007 — Higher-order mathematical operations

**Question:** Can PhysMath express the mathematics required by the TOE without becoming a CAS or collapsing into scalar expressions?

**Required decision:** bounded exact operations over declared mathematical sorts.

**Exit condition:** functional/operator/tensor examples type-check and compose; unsupported cases fail explicitly.

---

## B-008 — Formal limits/asymptotics

**Question:** Can the TOE express theoretical limits and truncations symbolically?

**Required decision:** exact formal representation of limits/asymptotics where in scope.

**Exit condition:** representative TOE scaling claims are expressible without numerical execution.

---

## B-009 — Mathematical trace versus Phys ledger

**Question:** What exactly is replayable in PhysMath, and what is replayable in Phys?

**Required decision:** two explicit replay domains with a formal handoff reference.

**Exit condition:** a Phys ledger cannot claim to replay a non-projectable PhysMath trace.

---

## B-010 — Cross-framework correspondence artifacts

**Question:** Where is a conjectural correspondence recorded without being promoted to identity?

**Required decision:** Project A owns research-level correspondence artifacts referencing PhysMath and external Phys contracts.

**Exit condition:** a correspondence can be proposed, attacked, rejected, or handed off without creating physical identity automatically.

---

## B-011 — External Phys-library object contract

**Question:** What exact artifact shape defines a physical object outside the frozen Phys base?

**Required decision:** extension-owned physical types plus explicit mathematical references and Phys metadata references where supported.

**Exit condition:** `phys-*` can define non-quantity ontology without forcing it into the frozen Phys `Kind`/`Dimension` model.

---

## B-012 — Phys metadata attachment to external objects

**Question:** How can assumptions/provenance/conventions/trust apply to external physical objects without creating duplicate metadata systems?

**Required decision:** canonical attachment mechanism through Phys; extension-local trust systems prohibited.

**Exit condition:** external objects remain external unless and until Phys's supported formalization/curation path authorizes trusted metadata.

---

## B-013 — Formalization bridge

**Question:** What exact API/schema crosses from Project A + PhysMath + phys-* into Phys?

**Required decision:** explicit bridge contract, projection subset, IDs, failure modes, and no silent coercion.

**Exit condition:** machine-readable handoff exists and is testable.

---

## B-014 — Handoff loss accounting

**Question:** What happens when Phys cannot represent most of a rich mathematical artifact?

**Required decision:** mandatory `FULLY/PARTIALLY/UNREPRESENTABLE` classification plus loss report.

**Exit condition:** partial export cannot silently become trusted.

---

## B-015 — t-flow / k-flow typing

**Question:** How are distinct formal evolution parameters represented?

**Required decision:** parameter identity, role, scope, and derivative relation.

**Exit condition:** direct negative test proving `∂t` and `∂k` cannot be conflated.

---

## B-016 — TOE joint-dynamics acceptance contract

**Question:** What does it mean for the three-project architecture to support a candidate joint law over Dancer/Dance/Stage?

**Required decision:** Project A acceptance artifacts plus mathematical and physical package interfaces.

**Exit condition:** the workload can formulate a candidate for

\[
\mathcal D[\Psi,Q,\mathcal C]=0
\]

without hardcoding a solution or pretending the candidate is established.

---

## B-017 — AI-agent machine-readable research protocol

**Question:** How does an AI agent manipulate hypotheses without reverting to prose as semantic state?

**Required decision:** versioned research/hypothesis/trace/handoff schemas.

**Exit condition:** construct → attack → revise → handoff can occur without paper notation being authoritative.

---

## B-018 — Numerical handoff boundary

**Question:** What exact artifacts may the Numerical Layer consume?

**Required decision:** formalized Phys/extension artifact surface; no raw research-state interpretation.

**Exit condition:** numerical tests reject incomplete physical formalizations rather than filling semantic gaps.

---

## B-019 — Phys Growth Gate and extension boundary

**Question:** When does a missing capability stay external, and when may it justify Phys kernel growth?

**Required decision:** external-first routing plus evidence-based Growth Gate.

**Exit condition:** TOE workloads can proceed without changing frozen Phys unless a generic foundational primitive is independently demonstrated missing.

---

## B-020 — Resource and adversarial testing

**Question:** How are pathological mathematical objects, deep compositions, huge tensors, recursion, and malicious research inputs contained?

**Required decision:** deterministic resource limits, failure classes, and adversarial tests.

**Exit condition:** resource exhaustion cannot corrupt traces, handoff artifacts, or Phys trust metadata.

---

# 33. Review synthesis — net valid findings

This section consolidates the attached AI reviews in light of the three-project architecture.

## 33.1 DeepSeek review — accepted findings

The following findings remain valid:

- the document cannot authorize implementation while unresolved blockers remain;
- the boundary must be mechanically enforceable, not merely stated in prose;
- the PhysMath/Phys dependency needs package-level containment;
- the TOE joint-dynamics workload needs an explicit acceptance contract;
- `t` and `k` flows require a formal parameter model;
- interface ownership must be explicit;
- mathematical and physical dimensions must be separated;
- PhysMath needs its own constructor/validation contract;
- handoff requires loss accounting;
- external Phys libraries need a common composability contract;
- AI-agent interoperability requires a machine-readable schema.

These are merged into B-001 through B-020.

## 33.2 Z review — accepted findings

The strongest valid points are:

- the bridge needs an actual mechanism, not only a package description;
- external physical objects need a defined shape outside the frozen quantity-oriented Phys core;
- provenance/metadata attachment must not be duplicated in extensions;
- PhysMath's higher-order structure model must be resolved before implementation;
- canonicalization requires a closed/versioned core IR;
- TOE acceptance must acknowledge that much richer mathematics may remain unrepresentable by frozen Phys.

These are merged into B-002, B-003, B-011, B-012, B-013, B-014, and B-016.

The numerical-layer criticism is refined: the issue is not simply “never transitively depend on PhysMath,” because external physical artifacts may legitimately reference mathematical structures. The enforceable rule is that numerical meaning must come from the formalized Phys/phys-* surface, not from raw or semantically reinterpreted research state.

## 33.3 Qwen review — accepted with architectural correction

The five themes were useful, but their proposed ownership is corrected:

- **Bridge:** accepted as a requirement, but no new permanent “Interpretation Layer” is introduced. Project A owns research-level handoff artifacts; Phys remains the trust authority.
- **Domain tagging:** accepted only as generic mathematical typing/sorts. Dancer/Dance/Stage MUST NOT become PhysMath ontology.
- **Typed symbolic engine:** accepted as the PhysMath mathematical operation layer, but it MUST remain mathematics-only.
- **Asymptotics:** accepted as a capability requirement, but “Formal Asymptotic Ring” is not locked as an implementation structure.
- **Correspondence artifact:** accepted, but owned by Project A as a research artifact referencing mathematical and physical contracts.

## 33.4 Gemini review — accepted baseline observations

The useful baseline observations are:

- numerical execution is outside the symbolic substrate;
- AI agents should manipulate machine-readable structures rather than generate Go code dynamically;
- explicit assumptions and branches are necessary;
- provenance and corpus status are distinct concerns in Phys;
- automatic scientific promotion is forbidden.

These are already incorporated into the three-project architecture.

Claims about the frozen Phys MVP are treated as background constraints, not as reasons to import Phys-specific semantics into PhysMath.

---

# 34. Findings explicitly rejected or reframed

The following are NOT adopted as written:

## 34.1 “PhysMath needs a Candidate Physical Object”

Not as a permanent PhysMath type.

Reason:

A candidate physical object would blur the exact boundary this architecture is designed to protect.

Reframing:

```text
Project A:
    research hypothesis / proposed physical correspondence

PhysMath:
    mathematical objects

phys-*:
    physical-domain contracts

Phys:
    physical metadata / trusted formalization
```

## 34.2 “Dancer/Dance/Stage domain tags belong in PhysMath”

Rejected.

Reason:

They would turn the initial TOE workload into PhysMath ontology.

Generic mathematical sorts are sufficient.

## 34.3 “Formal Asymptotic Ring” as a locked implementation

Rejected as premature.

The capability is required; the concrete implementation is not yet.

## 34.4 “Every external physical object must immediately become a trusted Phys object”

Rejected.

External physical libraries are allowed to remain external and reusable.

Trusted Phys status requires an explicit supported formalization/curation path.

---

# 35. Lifecycle gates

Implementation of the three projects follows these gates.

## G0 — Charter

Must identify:

- Project A;
- PhysMath;
- external Phys-library suite;
- frozen Phys;
- Numerical Layer;
- explicit non-overlap.

## G1 — Requirements

All constitutional requirements approved.

No implementation architecture may be invented later.

## G2 — Blocker closure

Every blocker B-001 through B-020 is:

```text
CLOSED
or
EXPLICITLY DEFERRED WITH CONTAINMENT
```

An unresolved blocker cannot be silently passed to implementation.

## G3 — Architecture

Produce:

- package graph;
- public APIs;
- artifact schemas;
- IR schema;
- extension protocol;
- bridge schema;
- lifecycle transitions;
- test strategy.

## G4 — Adversarial review

Independent AI agents attack:

- boundaries;
- dependency graph;
- canonicalization;
- typing;
- physical-object extensions;
- metadata monopoly;
- bridge;
- TOE stress workload;
- resource limits.

## G5 — Implementation plan

Only after G0-G4 may a coding agent receive an implementation plan.

The coding agent MUST NOT choose unresolved architecture.

## G6 — Implementation

Implement in separate project workstreams.

No cross-project convenience implementation may collapse boundaries.

## G7 — Acceptance

Run the TOE stress workload as a research program, not as one monolithic executable.

Acceptance must include:

- ordinary mathematical workloads;
- TOE mathematical workloads;
- external physical-library workloads;
- cross-framework correspondence workloads;
- handoff workloads;
- negative boundary tests;
- numerical handoff tests.

## G8 — Freeze

Freeze:

- APIs;
- IR;
- schemas;
- package graph;
- extension admission rules;
- handoff rules;
- test evidence.

Future changes follow explicit amendment/Growth Gate processes.

---

# 36. Implementation-plan authorization checklist

Implementation planning is authorized only when:

```text
[ ] Three projects have explicit ownership.
[ ] Phys remains frozen unless independently justified by Growth Gate.
[ ] PhysMath mathematics are isolated from physical semantics.
[ ] External phys-* libraries are isolated from Phys trust metadata duplication.
[ ] Numerical responsibilities are isolated.
[ ] PhysMath IR/sorts are pinned.
[ ] Canonicalization/equality/hashing are pinned.
[ ] Conditions/scopes are pinned.
[ ] Higher-order operations are pinned.
[ ] Asymptotic capability scope is pinned.
[ ] Research trace and Phys replay domains are pinned.
[ ] Correspondence artifact ownership is pinned.
[ ] External physical object contract is pinned.
[ ] Phys metadata attachment path is pinned.
[ ] Bridge/export/loss accounting is pinned.
[ ] t-flow/k-flow semantics are pinned.
[ ] TOE joint-dynamics acceptance contract is pinned.
[ ] AI-agent schema is pinned.
[ ] Numerical handoff is pinned.
[ ] Growth Gate routing is pinned.
[ ] Adversarial/resource tests are pinned.
```

---

# 37. Final architectural invariant

The entire ecosystem is successful only when this distinction remains true throughout the lifecycle:

```text
TOE-Agent
    = reasons about hypotheses

PhysMath
    = provides reusable mathematics

phys-*
    = provides reusable physical semantics

Phys
    = provides trusted physical metadata and formalization

Numerical
    = computes quantitative consequences
```

And:

```text
PhysMath does not become Phys.
phys-* does not become Phys.
TOE-Agent does not become PhysMath.
Numerical does not become a semantics engine.
Phys does not become an unrestricted mathematics engine.
```

The architecture is therefore intentionally asymmetric:

\[
\boxed{
\text{Research}
\rightarrow
\text{Mathematics}
\rightarrow
\text{Physical Formalization}
\rightarrow
\text{Numerical Consequences}
}
\]

but the mathematical and physical libraries remain reusable independently of the TOE.

The Dancer–Dance–Stage program is the first severe test of that architecture, not its definition.

---

# 38. Handoff principle for the coding agents

A coding agent working on any one of the three projects MUST NOT infer architecture from the other project's implementation.

Each project receives:

- this requirements document;
- its own project charter;
- its explicit dependency contract;
- its public artifact schemas;
- its acceptance tests.

A coding agent is conforming only when it can implement the assigned project without inventing:

- new semantic layers;
- new trust systems;
- new physics ontologies;
- hidden mathematical interpretation;
- hidden numerical execution;
- hidden cross-project dependencies.

**Final status: DESIGN-REVIEW / NOT YET IMPLEMENTATION-AUTHORIZED.**

The architecture itself is now separated into three projects. The remaining work is to close the decision-level blockers B-001 through B-020 before implementation planning.
