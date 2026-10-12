# PhysMath Requirements, Architectural Boundaries, Blocker Register, and Lifecycle Checklist

**Document ID:** PM-ARCH-REQ-001  
**Version:** 0.4  
**Status:** Design-review draft — strict ownership/non-overlap revision  
**Purpose:** Identify and close architectural blockers before implementation planning; provide a normative boundary between `PhysMath`, `Phys`, and the Numerical Layer; define the review and lifecycle gates that a coding agent must satisfy.

---

## 0. Authority and scope

This document governs the **design and implementation boundary of PhysMath**. It does not replace the frozen `Phys` specification. Where PhysMath consumes `Phys`, the existing `Phys` contract remains authoritative.

This document incorporates three locked architectural decisions:

1. **PhysMath is theoretical mathematical physics only.** It is an exact symbolic, pen-and-paper-style laboratory. Numerical computation is outside PhysMath.
2. **PhysMath uses compiler semantics rather than paper-notation semantics.** Mathematical notation is syntax/presentation; the semantic representation is explicit typed mathematical structure such as expressions, functions, operators, tensors, maps, relations, and transformations. Conventional physics meaning must never be guessed from notation alone.
3. **The Dancer–Dance–Stage TOE is the first major workload and stress test, not the definition of PhysMath.** PhysMath must remain useful for theoretical mathematical physics beyond that TOE.

The goal is not to settle physics. The goal is to establish a software architecture in which an AI agent can construct, transform, challenge, and formalize mathematical-physics hypotheses without silently changing the mathematics, importing numerical semantics, collapsing distinct domains, or bypassing the trust boundary established by `Phys`. This document also defines the **boundary and lifecycle for external Phys physics libraries**, so the simultaneous development of PhysMath and TOE-targeted packages such as `phys-qm`, `phys-bohm`, `phys-gr`, and causal/discrete-spacetime extensions follows one architecture rather than ad hoc package growth.

---

# 1. Executive architectural statement

The system is intentionally divided into three jobs:

```text
                         THEORETICAL RESEARCH
                                │
                                ▼
                        ┌────────────────┐
                        │   PhysMath     │
                        │ exact symbolic │
                        │ exploration    │
                        └───────┬────────┘
                                │ explicit formalization/export
                                ▼
                        ┌────────────────┐
                        │      Phys      │
                        │ trusted typed  │
                        │ physics IR     │
                        │ + provenance   │
                        │ + replay       │
                        └───────┬────────┘
                                │ formalized objects/artifacts
                                ▼
                        ┌────────────────┐
                        │ Numerical      │
                        │ Layer          │
                        │ computation    │
                        └────────────────┘
```

The dependency rule is:

```text
                       ┌──────────────┐
                       │    Phys      │
                       │ trusted base │
                       └──────┬───────┘
                              ▲
                              │
                       ┌──────┴───────┐
                       │   PhysMath    │
                       │ math only     │
                       └───────────────┘

Phys extensions (examples):
  phys-qm ─────► Phys + PhysMath
  phys-bohm ──► Phys + PhysMath + selected domain extensions
  phys-gr ────► Phys + PhysMath
  phys-cst ───► Phys + PhysMath

Numerical Layer ─────► Phys + selected Phys extensions
Numerical Layer -X-> PhysMath runtime
Phys ────────────────► nothing above itself
```

`PhysMath` depends on the public `Phys` API. External physics libraries extend the `Phys` ecosystem above the frozen base and MAY depend on both `Phys` and `PhysMath`. `Phys` itself MUST NOT depend on `PhysMath` or any external extension. The Numerical Layer consumes formalized `Phys`/extension artifacts and MUST NOT use PhysMath as a semantic backdoor.

The central design principle is:

> **PhysMath constructs and explores exact mathematical structures. Phys and its external domain libraries define explicit physical meaning and formal physical objects. The Numerical Layer computes consequences of those formal physical objects.**

No layer automatically adjudicates whether a theory is physically true.

---

# 2. Normative language

The keywords have the following meaning:

- **MUST** — mandatory architectural or semantic requirement.
- **MUST NOT / NEVER** — prohibited behavior.
- **SHOULD** — default design unless a documented review decision overrides it.
- **MAY** — permitted but not required.
- **BLOCKER** — a design issue that prevents implementation authorization.
- **NON-BLOCKING** — useful improvement that does not justify reopening architecture.

A coding-agent implementation is non-conforming if it requires the coding agent to invent an answer to an unresolved blocker.

---

# 3. Constitutional requirements

These requirements are already **LOCKED** and are not open design questions.

| ID | Requirement | Status |
|---|---|---|
| REQ-PM-CON-001 | PhysMath MUST be limited to theoretical mathematical physics and exact symbolic reasoning. | LOCKED |
| REQ-PM-CON-002 | PhysMath MUST NOT contain a numerical runtime. | LOCKED |
| REQ-PM-CON-003 | PhysMath MUST NOT perform floating-point evaluation, numerical simulation, Monte Carlo, parameter fitting, numerical PDE/ODE solving, GPU computation, or empirical-data analysis. | LOCKED |
| REQ-PM-CON-004 | Exact rational/integer/symbolic computation is permitted and is not considered numerical computation in the prohibited sense. | LOCKED |
| REQ-PM-CON-005 | Mathematical notation MUST NOT be treated as a semantic ontology merely because it resembles notation used in physics papers. | LOCKED |
| REQ-PM-CON-006 | Semantic meaning MUST arise from explicit declarations, typed structures, functions, operators, relations, or domain contracts. | LOCKED |
| REQ-PM-CON-007 | Any notation frontend, if introduced later, MUST desugar into explicit typed mathematical structures and MUST NOT infer undeclared physics semantics from notation conventions. | LOCKED |
| REQ-PM-CON-008 | PhysMath MUST NOT become a disguised physics-paper notation interpreter. | LOCKED |
| REQ-PM-CON-009 | The Dancer–Dance–Stage TOE is a workload/stress test, not the PhysMath ontology. | LOCKED |
| REQ-PM-CON-010 | PhysMath MUST support domains outside the initial TOE without requiring changes to the architectural constitution. | LOCKED |
| REQ-PM-CON-011 | PhysMath MUST NOT silently promote exploratory results into trusted Phys corpus authority. | LOCKED |
| REQ-PM-CON-012 | PhysMath MAY export selected work to Phys only through an explicit, auditable bridge. | LOCKED |
| REQ-PM-CON-013 | Phys MUST remain below PhysMath in the dependency graph. | LOCKED |
| REQ-PM-CON-014 | Numerical computation MUST consume formalized Phys objects/artifacts rather than raw PhysMath runtime state. | LOCKED |
| REQ-PM-CON-015 | No layer may claim mathematical consistency as proof of empirical truth. | LOCKED |
| REQ-PM-CON-016 | Research uncertainty, contradiction, failed derivation, counterexample, and unrepresentability MUST be representable as explicit outcomes rather than hidden exceptions. | LOCKED |
| REQ-PM-CON-017 | PhysMath MUST contain mathematical objects and mathematical relationships only; it MUST NOT define physical ontology, physical observables, physical regimes, or framework-specific physical meaning. | LOCKED |
| REQ-PM-CON-018 | A mathematical object MUST NOT acquire physical semantics merely because its name resembles a physics term. | LOCKED |
| REQ-PM-CON-019 | Physical objects MUST be defined in the Phys base or in an external Phys domain library, never in PhysMath. | LOCKED |
| REQ-PM-CON-020 | External Phys domain libraries MUST depend on the public Phys API and MAY depend on PhysMath; Phys MUST NOT depend on them. | LOCKED |
| REQ-PM-CON-021 | An external Phys library MUST distinguish interpretation-neutral physical contracts from framework-specific physical semantics; interpretation dependence MUST be explicit, never silent. | LOCKED |
| REQ-PM-CON-022 | A physical object exported from PhysMath MUST be classified as mathematical-only until an external Phys package provides an explicit physical contract for it. | LOCKED |
| REQ-PM-CON-023 | The formalization handoff target is the Phys ecosystem: Phys base for trusted substrate plus the relevant external Phys domain library for domain-specific physical meaning. | LOCKED |
| REQ-PM-CON-024 | The Dancer–Dance–Stage TOE MUST be exercised through reusable external Phys libraries such as `phys-qm`, `phys-bohm`, `phys-gr`, and causal/discrete-spacetime packages, rather than by adding TOE semantics to PhysMath core. | LOCKED |
| REQ-PM-CON-025 | PhysMath MUST NOT own a persistent assumption, convention, provenance, corpus-status, MRC, or other trust-authority metadata layer. | LOCKED |
| REQ-PM-CON-026 | Mathematical conditions such as `x != 0`, `x > 0`, branch restrictions, domains, and boundary conditions MAY exist in PhysMath only as mathematical structures/expressions, not as epistemic or physical assumption metadata. | LOCKED |
| REQ-PM-CON-027 | The `Phys` layer MUST own the canonical assumption, convention, provenance, corpus-status, trust, and related physical metadata model. | LOCKED |
| REQ-PM-CON-028 | External `phys-*` libraries MUST consume and populate the `Phys` metadata model; they MUST NOT duplicate or fork equivalent assumption, provenance, corpus-status, or trust systems. | LOCKED |
| REQ-PM-CON-029 | External `phys-*` libraries remain externally versioned modules above Phys and MUST NOT become Phys base code merely because they define physical objects used by the TOE stress test. | LOCKED |
| REQ-PM-CON-030 | PhysMath research traces MAY record mathematical operation history and failures, but MUST NOT assign physical provenance, corpus status, or trust authority to those traces. | LOCKED |
| REQ-PM-CON-031 | The formalization handoff MUST attach physical assumptions, conventions, provenance, and related trust metadata through Phys and the relevant external Phys library, not by retrofitting metadata into PhysMath mathematical objects. | LOCKED |

---

# 4. Job scope: no overlap between layers

## 4.1 Responsibility matrix

| Capability | PhysMath | Phys | Numerical Layer |
|---|---:|---:|---:|
| Construct exploratory mathematical objects | **PRIMARY** | LIMITED | NO |
| Construct theoretical hypotheses | **PRIMARY** | LIMITED / RECEIVES | NO |
| Exact symbolic algebra | **PRIMARY** | BOUNDED | NO |
| Symbolic differentiation/transformation | **PRIMARY** | BOUNDED, workload-specific | NO |
| General mathematical-physics representation | **PRIMARY** | SELECTED trusted subset | NO |
| Mathematical typing | **PRIMARY** | CONSUMES / VALIDATES selected mappings | CONSUMES |
| Physical ontology / physical typing | **NEVER** | **PRIMARY** in base and extensions | CONSUMES |
| Dimensions/units as physical semantics | **NEVER** | **PRIMARY** | Numeric propagation only |
| Mathematical conditions as expressions | **PRIMARY** | Consumes as needed | Consumes formalized state |
| Physical assumptions / conventions / regimes | **NEVER** | **PRIMARY** | Consumes formalized state |
| Mathematical execution trace | **PRIMARY, non-authoritative** | Retains trusted derivation ledger | Computation logs |
| Provenance / corpus status / trust metadata | **NEVER** | **PRIMARY** | Consumes; never adjudicates |
| Replayable derivation history | **PRIMARY research history** | **PRIMARY trusted ledger** | Computation logs |
| Corpus authority | NO | **YES, human-curated** | NO |
| Automatic truth adjudication | **NEVER** | **NEVER** | **NEVER** |
| Counterexample search, symbolic | **PRIMARY** | BOUNDED / formal checks | NUMERICAL counterexamples only after Phys formalization |
| Numerical evaluation | **NEVER** | **NEVER as a runtime job** | **PRIMARY** |
| Simulation | **NEVER** | **NEVER** | **PRIMARY** |
| Optimization/fitting | **NEVER** | **NEVER** | **PRIMARY** |
| Empirical-data comparison | **NEVER** | **NEVER** | **PRIMARY** |
| Export to Phys | **PRIMARY** | RECEIVES / VALIDATES | NO |
| Consume Phys formalizations | YES | PRIMARY | **PRIMARY** |
| Define universal mathematical notation semantics | NO | NO | NO |
| Decide physical truth | NO | NO | NO |

The following distinction is absolute:

> **PhysMath is where a hypothesis is mathematically constructed and attacked. Phys is where a selected formalization becomes a trusted, replayable physical artifact. The Numerical Layer is where that formalization is evaluated computationally.**

## 4.2 Mathematical object versus physical object

This classification is constitutional and MUST be decided before implementation.

A **mathematical object** is defined only by mathematical structure and rules of composition. A **physical object** asserts a role in a physical theory, model, regime, ontology, observable set, or empirical interpretation.

Examples:

| Object / concept | Classification | Required home | Rule |
|---|---|---|---|
| Hilbert space | Mathematical | **PhysMath** | Pure mathematical structure |
| Vector / state vector | Mathematical | **PhysMath** | Mathematical vector in a declared space |
| Wavefunction as a mathematical function | Mathematical | **PhysMath** | A function with explicit domain/codomain; no physical ontology implied |
| Generic `QuantumState` as physical state | Physical | **Phys extension** | Must carry an explicit quantum-theory contract |
| Bohmian wavefunction as physically operative object | Framework-specific physical | **`phys-bohm`** | Bohmian semantics must be explicit |
| Actual Bohmian configuration `Q` | Framework-specific physical | **`phys-bohm`** | Physical ontology belongs to Bohmian extension |
| Mass / energy / four-momentum as physical quantities | Physical | **Phys or relevant extension** | Physical dimensions and semantics are explicit |

The exact names are illustrative. The rule is semantic, not lexical.

**Critical consequence:** “wavefunction” is not one indivisible category. A wavefunction viewed strictly as a mathematical function belongs to PhysMath. A wavefunction asserted to be a physical state, beable-generating object, ontic object, or interpretation-specific entity belongs to an appropriate Phys extension. PhysMath MUST NOT silently choose that interpretation.

## 4.3 Physical semantics require an extension boundary

When an AI agent needs a physical object that Phys does not already define, the agent MUST NOT introduce that object into PhysMath merely because the mathematical representation already exists there. The object requires a Phys extension.

The extension is responsible for:

1. declaring the physical object;
2. defining its physical meaning;
3. tying it to explicit PhysMath mathematical objects;
4. declaring the physical meaning and framework scope;
5. attaching assumptions, regime, conventions, and provenance through the canonical Phys metadata model;
6. declaring whether the contract is interpretation-neutral or framework-specific;
6. exposing composable relationships to other physical objects;
7. providing the formalization and verification surface used by later numerical layers.

This makes the division precise:

```text
PhysMath:     What is this mathematical structure?
Phys extension: What physical thing, if any, is this structure used to represent?
Numerical:     What numerical consequences follow from that formalized physical model?
```

---

# 5. Explicit non-overlap rules

## 5.1 PhysMath MUST NOT absorb numerical responsibilities

PhysMath MUST NOT acquire numerical evaluation merely because symbolic expressions can eventually be assigned values.

The following are outside PhysMath:

- floating-point arithmetic;
- numerical root finding;
- numerical integration;
- numerical differentiation;
- finite-difference/finite-element solvers;
- ODE/PDE numerical solvers;
- Monte Carlo;
- stochastic simulation;
- numerical optimization;
- parameter estimation;
- statistical fitting;
- GPU/accelerator execution;
- benchmarking physical predictions against empirical datasets.

Exact symbolic evaluation such as `2 * 3 = 6`, rational normalization, exact polynomial manipulation, or exact finite combinatorics is allowed.

## 5.2 Phys MUST NOT absorb exploratory responsibilities

`Phys` MUST NOT become an unrestricted symbolic research environment merely because PhysMath can produce richer mathematics.

The trusted Phys substrate remains constrained by its own frozen architecture, provenance model, MRC rules, canonicalization, and replay/audit model.

A Phys limitation is not automatically a defect. PhysMath exists partly so that theoretical work can continue outside the trusted core without forcing kernel growth.

## 5.3 Numerical Layer MUST NOT become a second formal semantics engine

The Numerical Layer MUST consume formalized meaning from Phys. It MUST NOT silently reinterpret symbols, redefine dimensions, repair malformed equations, infer missing physical assumptions, or change ontology to make computation possible.

If the Phys artifact is not sufficiently formalized for the numerical computation, computation stops and the gap is reported.

---

# 6. Compiler architecture principle

PhysMath MUST follow the same fundamental lesson as Phys:

> **The visible mathematical notation is not the semantic substrate. The semantic substrate is a typed program representation.**

The system MUST therefore reason in terms of explicit structures such as:

- expressions;
- functions and function applications;
- operators and operator applications;
- relations/equations/inequalities;
- sets, maps, sequences, and indexed families;
- vectors and matrices;
- tensors and explicit index structures;
- manifolds, charts, metrics, connections, and curvature objects where supported;
- differential operators;
- functionals and variational operators;
- discrete structures such as graphs, posets, complexes, and combinatorial objects;
- transformations and derivation steps;
- assumptions and constraints;
- exact or explicitly approximate symbolic forms.

The representation must be compositional. A symbol is not allowed to acquire domain meaning simply because its spelling resembles a conventional physics symbol.

### Example

The semantic representation of

```text
m c²
```

must be constructed from explicit objects equivalent to:

```text
Mul(
    Symbol("m"),
    Pow(Symbol("c"), Rational(2,1)),
)
```

A renderer MAY later print `mc²`, but rendering is not semantics.

Likewise, the semantic representation of a derivative such as

```text
∂f/∂x
```

must be an explicit differentiation structure or function application, not an opaque “physics derivative symbol.”

---

# 7. No paper-notation interpreter

This is a hard design boundary.

PhysMath MUST NOT contain a general rule such as:

```text
"G_{μν}" + context => Einstein tensor
"A_μ" + context => electromagnetic potential
"H" + context => Hamiltonian
"Ψ" + context => quantum state
```

Such inference is semantically unsafe.

The compiler may receive a declaration such as:

```text
G : Tensor(rank=2, manifold=M)
```

or:

```text
Psi : Function(ConfigurationSpace -> Complex)
```

and then use those declarations to type-check subsequent expressions.

The important distinction is:

```text
paper notation -> syntax
explicit declaration -> semantics
```

not:

```text
paper notation -> guessed physics meaning
```

A future text/LaTeX frontend therefore cannot be used as an excuse to violate this boundary.

---

# 8. PhysMath mathematical scope

PhysMath is intended to be broad enough for theoretical mathematical physics, but it is not defined by a closed encyclopedia of mathematics.

## 8.1 Required capability families

The architecture MUST be capable of supporting, through core structures and extensible domain packages:

- algebraic structures;
- symbolic arithmetic;
- exact calculus and differential operators;
- linear and multilinear algebra;
- tensor/index mathematics;
- geometry and differential geometry;
- functions, function spaces, and functionals;
- operator mathematics;
- variational constructions;
- differential equations at the symbolic/theoretical level;
- asymptotic and perturbative symbolic constructions;
- probability/statistical structures as mathematical objects;
- discrete mathematics and combinatorics;
- mathematical structures used in quantum theory, field theory, gravitation, statistical mechanics, and related disciplines;
- alternative formulations of the same theoretical structure;
- cross-framework composition where compatibility is explicit.

This is a **capability requirement**, not a demand that v0 implement every branch of mathematics.

## 8.2 No monolithic “universal mathematics” type

PhysMath SHOULD NOT attempt to create one universal `MathObject` that erases distinctions between:

```text
number
expression
function
operator
set
map
vector
covector
tensor
manifold
field
functional
relation
proof obligation
```

These categories have different composition rules and failure modes.

The architecture should therefore use multiple related sorts with explicit interfaces and coercions only where mathematically valid.

## 8.3 Extensibility over enumeration

A future mathematical structure MUST NOT require editing a global list of every possible mathematical concept.

Domain packages should add semantics where appropriate without forcing all existing packages to know about every future domain.

---

# 9. Exactness model

PhysMath is symbolic-first and exact by default.

## 9.1 Allowed

- exact integers;
- exact rationals;
- symbolic constants;
- symbolic parameters;
- exact finite sums/products;
- exact algebraic transformations;
- symbolic limits where a defined operation supports them;
- symbolic differentiation/integration where supported;
- symbolic asymptotic forms;
- bounded symbolic series/truncation where supported;
- exact discrete structures;
- explicit approximation objects that retain their approximation semantics.

## 9.2 Forbidden

- machine floating point as hidden semantics;
- implicit decimal approximation;
- numerical evaluation hidden inside a symbolic operation;
- approximate equality masquerading as exact equality;
- numerical fallback when exact reasoning fails;
- silent loss of exact assumptions to permit a computation.

## 9.3 Approximation is mathematical, not numerical

An approximation such as

```text
A(ε) = 1 + ε a₁ + O(ε²)
```

may be legitimate theoretical mathematics if represented explicitly and exactly as an approximation/truncation structure.

It must not silently turn into a floating-point calculation.

---

# 10. Equality, identity, and equivalence

This is an architectural issue, not an implementation detail.

PhysMath MUST distinguish at least the following concepts where applicable:

1. **structural identity** — same canonical representation;
2. **definitional equality** — same by an explicit definition;
3. **mathematical equivalence** — same under a declared equivalence rule;
4. **physical identification** — an explicit physical correspondence, never inferred from notation;
5. **approximate equivalence** — a relation with declared approximation semantics.

The system MUST NOT collapse these into a single `Equal()` operation.

Two mathematically equivalent expressions may have different structural forms. Conversely, two structurally similar objects may be physically different because their types, assumptions, regimes, or domains differ.

---

# 11. Mathematical conditions versus physical assumptions

This section resolves a critical ownership boundary.

PhysMath MUST represent mathematical conditions only as mathematical structures. It MUST NOT own a first-class epistemic `Assumption` concept whose purpose is to describe physical knowledge, regime, interpretation, or trust.

Examples that belong in the mathematical representation include:

```text
x != 0
x > 0
m >= 0
branch_condition(x)
boundary_condition
domain(f)
```

These are expressions, relations, functions, sets, or other mathematical structures. They can participate in symbolic transformations, branch selection, domain checks, and exact derivations.

By contrast, statements such as:

```text
rest_mass_nonnegative
special_relativistic_regime
metric.signature = -+++
source = published theory
corpus_status = ESTABLISHED
```

are physical/trust metadata. They belong to Phys and its external physical extensions.

### Hard rule

```text
Mathematical condition  -> PhysMath mathematical object
Physical assumption     -> Phys metadata
Physical convention     -> Phys metadata
Physical regime         -> Phys metadata
Physical provenance     -> Phys metadata
Corpus/trust status     -> Phys metadata
```

PhysMath MAY receive references to external Phys metadata when operating as part of a larger formalization workflow, but MUST NOT recreate, own, or reinterpret that metadata.

An external Phys library MUST use the canonical Phys metadata model instead of defining package-local equivalents.

# 12. Time, scale, and other dimensional flows

PhysMath MUST support the distinction between conceptually different evolution parameters.

The Dancer–Dance–Stage workload is an explicit stress test:

- physical/temporal flow `t`;
- renormalization/coarse-graining scale `k`;
- other formal parameters that may be neither.

The compiler MUST NOT equate two flow parameters because both appear as derivatives or both are called “scale” in informal discussion.

The correct semantic model is:

```text
parameter identity + declared role + domain + transformation law
```

not spelling-based equivalence.

---

# 13. Functions, operators, and higher mathematical structures

A major architectural blocker is preventing everything from collapsing into a scalar expression tree.

PhysMath MUST be able to represent concepts that are naturally:

- functions;
- functionals;
- operators;
- maps between structured spaces;
- transformations;
- indexed families;
- distributions or generalized functions where explicitly supported;
- tensor fields and geometric objects;
- differential operators.

The design MUST define clear boundaries among:

```text
Expression
Function
Function application
Operator
Operator application
Transformation
Relation
Proof/derivation obligation
```

A `Call` node alone is not enough to solve this if the type system cannot distinguish these categories.

The implementation plan MUST therefore resolve how higher-order mathematical objects are typed and composed before coding begins.

---

# 14. Symbol identity and namespaces

Raw symbol spelling MUST NOT determine semantic identity.

At minimum, the architecture needs explicit handling for:

- namespace/module ownership;
- local bindings;
- parameter scopes;
- bound variables;
- dummy/index variables;
- function argument scopes;
- shadowing;
- generated names;
- stable identifiers independent of pretty-printing.

For example, two symbols named `r` in different derivations MUST NOT be assumed to denote the same object.

The canonical identity should be based on declared scope/identity, not display text.

---

# 15. Mathematical derivation and research trace

PhysMath is an exploratory mathematical laboratory. Its durable internal record, if any, is a **mathematical execution trace**, not a physical provenance or trust ledger.

A trace SHOULD capture:

- mathematical object identifiers;
- exact premises as mathematical expressions;
- operation sequence;
- intermediate mathematical states;
- transformations used;
- failed attempts;
- counterexamples;
- unresolved mathematical obligations;
- alternative mathematical formulations.

A PhysMath trace MUST NOT assign:

- physical provenance;
- physical corpus status;
- empirical confidence;
- interpretation authority;
- trusted physical-object status.

When an AI agent decides that a candidate is ready for physical formalization, the trace is handed to Phys together with the explicit physical mappings required by the relevant external Phys libraries. Phys then creates the canonical physical assumptions, conventions, provenance, and trust metadata.

# 16. Counterexamples and contradiction-first reasoning

PhysMath SHOULD support an AI agent that attacks its own hypotheses rather than merely completing derivations.

Examples:

```text
Hypothesis H
   ↓
instantiate explicit domain/assumptions
   ↓
derive consequences
   ↓
search for contradiction / invariant violation / counterexample
   ↓
record failure OR retain candidate
```

The architecture MUST distinguish:

- “not proven”;
- “derivation failed”;
- “contradiction found”;
- “counterexample found”;
- “mathematically inconsistent under stated assumptions”;
- “unrepresentable in current PhysMath”;
- “representable but outside current operation coverage.”

These outcomes are materially different and MUST NOT collapse into a generic error.

---

# 17. Multi-framework reasoning

PhysMath MUST support mathematical work that is reusable across multiple physical frameworks without silently asserting that the frameworks are compatible.

A framework-specific physical meaning is supplied by external Phys libraries. PhysMath itself manipulates the mathematics underlying those frameworks.

The Dancer–Dance–Stage workload specifically tests this requirement through external contracts such as:

- **Dancer:** Bohmian/QFT mathematical structures plus physical contracts supplied by `phys-qm` / `phys-bohm`;
- **Dance:** mathematical flow/evolution structures plus physical contracts for time evolution and scale evolution;
- **Stage:** mathematical causal/discrete structures plus physical contracts supplied by `phys-gr`, `phys-cst`, `phys-cdt`, or equivalent extensions.

A cross-framework relationship MUST carry explicit information about:

- source physical framework or extension;
- target physical framework or extension;
- mathematical mapping or correspondence;
- physical contract, if any;
- assumptions;
- regime;
- directionality;
- exactness/approximation;
- whether the relationship is established, proposed, or unresolved.

PhysMath MUST NOT infer compatibility from:

```text
same notation
same mathematical shape
same dimension
same symbol spelling
same theorem name
```

# 18. Dancer–Dance–Stage as the initial acceptance workload

The current TOE blueprint is a demanding architectural stress test because it requires reusable mathematical structures and separately defined physical semantics to interact without collapsing them into one language.

The TOE MUST therefore be implemented as a **composition of reusable PhysMath mathematics plus external Phys physics libraries**. It MUST NOT be encoded as Dancer/Dance/Stage compiler ontology.

A representative target decomposition is:

```text
PhysMath
  ├── Hilbert/function spaces and operators
  ├── configuration-space mathematics
  ├── calculus / differential equations
  ├── tensors / geometry / discrete mathematics
  └── exact transformations and derivations

Phys extensions
  ├── phys-qm
  ├── phys-bohm
  ├── phys-gr
  ├── phys-cst / phys-cdt
  └── future physics packages
```

These package names are target examples, not an implementation commitment. The architecture must remain generic.

### Dancer

Mathematical side: quantum-state spaces, Hilbert spaces, function spaces, operators, algebras, configuration spaces, wavefunctions as mathematical functions.

Physical side: quantum states, observables, particles/beables, Bohmian configuration, and interpretation-specific ontology. These belong in the relevant Phys extension.

### Dance

Mathematical side: differential equations, flows, functional derivatives, variational structures, RG mathematics, transformations.

Physical side: temporal dynamics, guidance laws, field evolution, coarse-graining/scale dynamics, and physical interpretations of those equations. These belong in the relevant Phys extension.

### Stage

Mathematical side: posets, causal orders, graphs, simplicial complexes, triangulations, tensors, metrics, connections, curvature, combinatorial histories.

Physical side: spacetime, causal structure, dynamical geometry, physical regimes and interpretations. These belong in the relevant Phys extension.

### Interfaces

The workload must be able to represent candidate mathematics for:

```text
Dancer ↔ Stage
Dance  ↔ Stage
Dancer ↔ Dance
```

The physical claim represented at an interface MUST come from explicit external Phys contracts. PhysMath supplies the reusable mathematical machinery needed to formulate and test that claim.

The TOE is therefore a stress test of:

1. mathematical composability;
2. physical-package composability;
3. interpretation-boundary discipline;
4. formalization handoff;
5. extension reuse;
6. cross-framework correspondence;
7. absence of hard-coded TOE semantics.

The TOE tests the architecture; it does not define the architecture.

# 19. Phys boundary, external physics libraries, and formalization bridge

This section defines **what, when, where, why, and how** physical extensions are created and how they interact with PhysMath.

## 19.1 One-way dependency

The dependency graph MUST remain acyclic:

```text
                 PhysMath
                    │
                    ▼
                   Phys
                    ▲
                    │
        ┌───────────┼───────────┐
        │           │           │
     phys-qm    phys-bohm    phys-gr
        │           │           │
        └──────► Numerical Layer ◄──────┘
```

More precisely:

```text
PhysMath  ─────► Phys
phys-*    ─────► Phys
phys-*    ─────► PhysMath   (only when mathematical machinery is needed)
Numerical ─────► Phys + selected phys-* packages
Phys      ─────► nothing above itself
Numerical -X->  PhysMath runtime
PhysMath -X->   phys-* physical semantics
```

A physical extension MAY use mathematical structures from PhysMath, but that never creates a reverse dependency from Phys to PhysMath.

## 19.2 What is an external Phys library?

An external Phys library is a separately versioned package that adds **physical objects and physical relationships** above the trusted Phys base. It is not merely a convenience wrapper around PhysMath.

Examples include:

```text
phys-qm       quantum-theory physical contracts
phys-bohm     Bohmian physical ontology/dynamics
phys-gr       general-relativistic physical contracts
phys-cst      causal-set physical contracts
phys-cdt      causal-dynamical-triangulation physical contracts
```

These are illustrative target packages, not frozen repository names.

## 19.3 When is a Phys extension required?

An external Phys library is required when the AI agent needs a concept whose meaning is **physical rather than purely mathematical**, and that physical contract is not already supplied by the Phys base.

The agent MUST NOT solve this by adding the physical concept to PhysMath.

Decision rule:

```text
Is the object defined solely by mathematics?
    YES -> PhysMath
    NO
      ↓
Does it assert physical meaning?
    YES -> Phys base or external Phys library
```

## 19.4 Where does the extension live?

External Phys libraries MUST live above the frozen Phys base as independently versioned modules/packages. They MUST depend on public APIs only and MUST NOT import `internal/kernel`.

A domain extension MUST NOT modify the frozen Phys repository merely to add domain-specific ontology unless the existing Phys Growth Gate independently establishes that a foundational kernel primitive is missing.

## 19.5 Why require explicit physical extensions?

The separation provides:

- reusable mathematical infrastructure in PhysMath;
- stable trusted physical infrastructure in Phys;
- independent domain semantics;
- explicit interpretation boundaries;
- smaller packages and clearer review surfaces;
- reuse across unrelated theories;
- a clean path for AI-generated hypotheses to request new physical concepts without corrupting the mathematical layer.

## 19.6 Physical-object admission criteria

A physical object defined by a Phys extension MUST satisfy all applicable criteria before it is treated as a reusable library concept:

1. **Explicit physical contract** — the package states what physical role the object has.
2. **Explicit mathematical representation** — the object is tied to concrete PhysMath structures; paper notation alone is insufficient.
3. **Semantic uniqueness within scope** — the object name MUST NOT hide multiple incompatible interpretations.
4. **Interpretation classification** — the package MUST state whether the object is interpretation-neutral, interpretation-compatible under declared assumptions, or framework-specific.
5. **Explicit regime and conventions** — relevant assumptions and conventions are declared rather than inferred.
6. **Canonical Phys metadata use** — source/reference, assumptions, conventions, and status are represented through Phys rules rather than package-local metadata.
7. **Composability** — relationships to other objects are typed and explicit.
8. **Reusability** — the object provides value beyond one single derivation unless intentionally marked as hypothesis-local.
9. **No notation semantics** — semantic meaning does not depend on how the object is printed.
10. **No silent truth claim** — existence in a package does not imply empirical truth or universal validity.

## 19.7 Metadata ownership: assumptions, provenance, conventions, and trust

This boundary is absolute. There is **one Phys trust/metadata model**. PhysMath and external `phys-*` libraries do not create competing versions of it.

### PhysMath owns only mathematical structure

PhysMath MAY represent a condition mathematically:

```text
x > 0
m >= 0
Det(g) != 0
r > 2M
```

but these are mathematical expressions or relations. PhysMath MUST NOT label them as physical assumptions, attach corpus status to them, declare them established facts, or assign provenance to their physical interpretation.

PhysMath MAY record:

- operation inputs/outputs;
- exact intermediate mathematical states;
- transformation identifiers;
- deterministic execution traces;
- failure and counterexample results;
- mathematical object identities.

Those records are **computational/research traces**, not Phys trust metadata.

### Phys owns the trust metadata plane

The canonical Phys layer owns:

- physical assumptions;
- physical regimes;
- conventions;
- physical provenance;
- corpus status;
- MRC/trust decisions;
- trusted physical-object authority;
- trusted physical derivation/session metadata.

External `phys-*` libraries MUST express their domain-specific metadata through these Phys facilities. An extension may define a new physical object or relationship, but it MUST NOT define a parallel `Provenance`, `CorpusStatus`, `AssumptionSet`, `ConventionSet`, MRC, or trust authority.

This rule prevents metadata fragmentation such as:

```text
phys-qm.AssumptionSet       X
phys-bohm.Provenance        X
phys-gr.CorpusStatus        X
Phys.AssumptionSet          ✓
Phys.Provenance             ✓
Phys.CorpusStatus           ✓
```

The extension owns **physical semantics**; Phys owns the **canonical trust metadata machinery**.

## 19.7 Interpretation ambiguity rule

Interpretation dependence is permitted; **silent interpretation dependence is not**.

A library MUST use one of two approaches:

```text
A. Interpretation-neutral contract
   The object has a clearly stated physical meaning shared by the
   declared scope of theories.

B. Framework-specific contract
   The object is explicitly scoped to a named framework/package.
```

A framework-specific object MUST NOT masquerade as an interpretation-neutral object.

### Canonical example

```text
PhysMath
  HilbertSpace
  Vector
  ComplexFunction
  Operator
  InnerProduct
  WaveFunction-as-mathematical-function

phys-qm / phys-bohm
  QuantumState / framework-specific state contract
  BohmianWaveFunction
  BohmianConfiguration
  GuidanceLaw
```

The exact API names are implementation questions. The ownership rule is not.

## 19.8 Formalization handoff

When a PhysMath hypothesis chain reaches the handoff criterion—mathematically coherent, internally consistent under stated mathematical assumptions, and compatible with the required physical contracts—the AI agent MUST produce an explicit formalization package for the **Phys ecosystem**.

The handoff MUST identify:

```text
mathematical objects        -> PhysMath source identities
physical objects            -> relevant Phys / phys-* identities
assumptions                 -> mathematical + physical sources
conventions                 -> explicit source contracts
relationships               -> typed mappings
derivation                   -> exact operation sequence
regime                       -> physical extension contract
unknowns / unresolved parts  -> explicit obligations
```

The receiving target is not necessarily the frozen `Phys` package alone. A hypothesis involving quantum mechanics, Bohmian mechanics, GR, CST/CDT, or another domain is formalized against the relevant external Phys library while using the trusted Phys substrate underneath.

## 19.9 Export classification

The bridge MUST classify each exported mathematical component as:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

The bridge MUST NOT silently coerce or discard information. It SHOULD retain:

- source PhysMath identity;
- target Phys/extension identity, if created;
- mathematical mapping;
- physical contract mapping;
- mathematical conditions preserved/unrepresented;
- Phys metadata attachment requirements;
- physical provenance mapping performed by Phys;
- approximation status;
- semantic information lost, if any;
- reason for partial/unrepresentable status.

## 19.10 No automatic corpus promotion

Successful formalization does not automatically make a candidate a trusted corpus fact. Human-controlled corpus governance remains authoritative.

## 19.11 Phys kernel growth remains exceptional

If an external Phys library cannot express something needed by PhysMath, the default route is package-level extension, not kernel modification:

```text
PhysMath requirement
        ↓
reusable mathematics? ──YES──► PhysMath
        ↓ NO
physical domain object? ──YES──► phys-* extension
        ↓ NO
generic foundational Phys primitive?
        ↓
Growth Gate evidence required
```

Neither “the TOE needs it” nor “PhysMath is more convenient if Phys supports it” is sufficient justification for kernel growth.

# 19.12 External Phys library lifecycle

Every external Phys library intended for reuse MUST pass the following lifecycle before it becomes a supported dependency of a derivation:

```text
Need identified
    ↓
Object classified as physical
    ↓
Base Phys vs extension decision
    ↓
Physical contract drafted
    ↓
Interpretation scope declared
    ↓
Math ↔ physics mapping specified
    ↓
Assumptions / regime / provenance specified
    ↓
Adversarial ambiguity review
    ↓
Composability tests
    ↓
Reusable package release
```

A package may be used experimentally before release, but experimental status MUST remain explicit and MUST NOT be mistaken for trusted corpus authority.

# 20. Numerical Layer boundary

The Numerical Layer exists above the formalization boundary and consumes formalized Phys objects together with any selected external Phys domain contracts.

## 20.1 Numerical inputs

A numerical computation MUST begin from formalized Phys objects/artifacts and, where domain-specific semantics are needed, the corresponding external Phys library artifacts. The numerical layer consumes the canonical Phys assumptions, conventions, provenance, and other trust metadata; it does not recreate them.

The Numerical Layer MAY perform:

- numerical evaluation;
- simulation;
- parameter sweeps;
- optimization;
- fitting;
- numerical differential-equation solving;
- Monte Carlo/statistical computation;
- empirical-data comparison;
- HPC/GPU execution.

These capabilities are explicitly outside PhysMath.

## 20.2 No semantic repair

The Numerical Layer MUST NOT:

- infer undeclared physical meaning from symbols;
- fill missing assumptions silently;
- reinterpret dimensions;
- choose among physically distinct formulations without an explicit instruction;
- write numerical results directly into trusted Phys corpus authority;
- mutate Phys semantics to make a computation easier.

## 20.3 Result direction

Numerical evidence may feed a **research artifact or human review process**, but it MUST NOT automatically change:

- corpus status;
- provenance;
- theory truth status;
- hypothesis promotion state.

---

# 21. AI-agent interface requirements

PhysMath is intended to be used by AI agents as a machine-operable theoretical laboratory.

The AI-facing interface MUST expose explicit structure rather than encourage prose-only reasoning.

An agent should be able to perform a loop equivalent to:

```text
1. Declare objects and types.
2. Construct mathematical condition expressions and other mathematical structures.
3. Construct equations/functions/operators.
4. Compose transformations.
5. Inspect exact intermediate states.
6. Attempt derivations.
7. Attack the result with contradiction/counterexample checks.
8. Record mathematical failures and unresolved obligations.
9. Compare alternative mathematical formulations.
10. Explicitly hand off selected work to Phys for physical formalization and trust metadata.
```

The interface MUST make it difficult to:

- smuggle in assumptions;
- confuse symbols by spelling;
- confuse parameters by name;
- replace exact objects with approximate ones silently;
- assign or lose physical provenance;
- skip a required transformation step;
- silently reinterpret a theory boundary.

---

# 22. Canonicalization and deterministic research state

PhysMath SHOULD use deterministic canonical representations wherever objects are serialized, hashed, compared, or replayed.

The design MUST answer, before implementation:

- how structurally equivalent expressions canonicalize;
- how commutative child order is normalized;
- how bound variables are normalized;
- how tensors and indices are ordered;
- how mathematical condition expressions are ordered;
- how domain identities are serialized;
- how alternative representations are compared;
- how exact parameters are encoded;
- which data is semantic versus presentation-only.

Research replay MUST replay recorded transformations rather than relying solely on recomputation from current implementation behavior.

A hash is an integrity check; it is not a substitute for the replayable semantic record.

---

# 23. Error and failure model

PhysMath must have typed failures at architectural boundaries.

At minimum, the design needs distinguishable classes for:

```text
MalformedStructure
TypeMismatch
DomainMismatch
DimensionMismatch
AssumptionConflict
ConventionConflict
UnsupportedTransformation
UnresolvedObligation
DerivationFailure
CounterexampleFound
ContradictionFound
NonTerminationOrResourceLimit
Unrepresentable
PartiallyRepresentable
ExportConflict
```

Error values MUST carry enough structured context for an AI agent to continue reasoning rather than receiving an opaque string.

---

# 24. Resource and termination boundaries

Theoretical symbolic computation can diverge, recurse without bound, or experience combinatorial explosion.

PhysMath MUST define resource controls before production implementation.

At minimum:

- operation budget;
- recursion/depth limits where relevant;
- symbolic-size limits;
- rewrite-step limits;
- expression-growth detection;
- deterministic cancellation/timeout behavior;
- preservation of partial research state on failure.

A resource limit is a computational limitation, not a mathematical contradiction.

The system MUST NOT label a computation “false” merely because it hit a resource bound.

---

# 25. Go architecture constraints

The implementation MUST use compiler-oriented package boundaries rather than one monolithic package.

The eventual package graph SHOULD separate concerns similar to:

```text
PhysMath
├── mathematical IR / types
├── canonicalization
├── algebra / calculus
├── tensor / geometry / discrete mathematics
├── assumptions / relations
├── derivation / research ledger
├── counterexample / verification helpers
└── Phys bridge

Phys base
└── trusted physical substrate

External Phys libraries
├── phys-qm
├── phys-bohm
├── phys-gr
├── phys-cst / phys-cdt
└── future domain packages

Numerical Layer
└── consumes Phys + selected Phys extensions
```

These names are illustrative, not frozen package names. The implementation plan MUST settle the actual graph after the blockers below are resolved.

The package graph MUST preserve:

- PhysMath imports Phys only through its public API;
- no import cycle with Phys;
- external Phys libraries may import Phys and PhysMath but never Phys internal/kernel;
- no Numerical Layer dependency into PhysMath runtime;
- physical domain extensions not required by mathematical core;
- research history separate from trusted Phys ledger;
- explicit interfaces rather than hidden global registries where possible.

---

# 26. Architectural blocker register

**Rule:** No implementation plan is authorized while a blocker marked `OPEN` remains unresolved unless the review explicitly records it as intentionally deferred with a concrete containment strategy.

| Blocker | Question that must be answered | Initial status | Exit condition |
|---|---|---|---|
| B-001 | What are the exact mathematical sorts in the core IR? | OPEN | Typed sort model approved |
| B-002 | How are expressions, functions, operators, maps, relations, and transformations distinguished? | OPEN | Distinct semantics + composition rules |
| B-003 | What coercions/conversions are permitted, and which are forbidden? | OPEN | Explicit coercion matrix |
| B-004 | What constitutes structural equality vs mathematical equivalence? | OPEN | Equality model + tests |
| B-005 | How are names scoped and bound across functions, indices, local derivations, and modules? | OPEN | Symbol identity model |
| B-006 | What is the exact representation of mathematical condition expressions and bounded transformation preconditions in PhysMath, without creating a Phys-style assumption metadata layer? | OPEN | Mathematical condition/operation-precondition contract |
| B-007 | How are dimensions and physical types represented without forcing every mathematical object to be a physical quantity? | OPEN | Math/physics typing boundary |
| B-008 | How are tensors, indices, manifolds, maps, and geometric structures represented without collapsing them into generic expressions? | OPEN | Tensor/geometry IR contract |
| B-009 | What symbolic calculus is in v0 and what is explicitly deferred? | OPEN | Capability matrix with bounded semantics |
| B-010 | How are approximations, asymptotics, truncations, and exact symbolic limits represented? | OPEN | Exact approximation model |
| B-011 | How are mathematical execution traces, failed derivations, and counterexamples recorded/replayed without becoming a physical provenance or trust ledger? | OPEN | Trace contract + failure model |
| B-012 | How are alternative formulations represented and compared? | OPEN | Representation/correspondence model |
| B-013 | How are cross-framework relationships declared without silently asserting compatibility? | OPEN | Correspondence contract |
| B-014 | How does PhysMath prove to itself that a transformation executed is the transformation recorded? | OPEN | Deterministic replay design |
| B-015 | What exact information must survive Phys export, and how is partial/unrepresentable content handled? | OPEN | Bridge schema + loss accounting |
| B-016 | What exact artifact does the Numerical Layer receive from Phys? | OPEN | Numerical handoff schema |
| B-017 | What serialization format is the canonical AI-agent interchange format? | OPEN | Versioned machine-readable schema |
| B-018 | What resource/termination controls are part of semantics vs runtime policy? | OPEN | Resource contract |
| B-019 | What is the extension mechanism for new mathematical domains? | OPEN | Domain-extension protocol |
| B-020 | What test-oracle strategy verifies symbolic transformations without requiring an unrestricted theorem prover? | OPEN | Layered test strategy |
| B-021 | What implementation dependencies, if any, are allowed beyond Go's standard facilities? | OPEN | Dependency policy |
| B-022 | What security/failure containment prevents malformed AI-generated structures from corrupting trusted state? | OPEN | Trust-boundary tests |
| B-023 | What evidence is required before an implementation stage may be considered complete? | OPEN | Lifecycle evidence matrix |
| B-024 | What exact rule distinguishes a mathematical object from a physical object? | OPEN | Object-ownership classifier + negative tests |
| B-025 | Which physical objects belong in frozen Phys versus external Phys libraries? | OPEN | Base-vs-extension admission matrix |
| B-026 | How is interpretation ambiguity detected and prevented from becoming silent physical semantics? | OPEN | Interpretation-neutral/framework-specific contract |
| B-027 | What exact dependency graph governs Phys extensions that also consume PhysMath? | OPEN | Acyclic module graph + import tests |
| B-028 | What is the external Phys library contract for declaring physical meaning over PhysMath objects? | OPEN | Extension schema/API |
| B-029 | What exact artifact is handed from PhysMath to the Phys ecosystem when a hypothesis is ready for formalization? | OPEN | Formalization handoff schema |
| B-030 | What naming/scoping rules prevent a mathematical name such as `WaveFunction` from silently becoming an interpretation-specific physical ontology? | OPEN | Naming and semantic-scope policy |
| B-031 | How are PhysMath, Phys base, and external Phys package versions represented so a derivation remains replayable while physical provenance remains owned by Phys? | OPEN | Version identity + Phys metadata contract |
| B-032 | Can an external `phys-*` package introduce any parallel assumption, convention, provenance, corpus-status, MRC, or trust type? | OPEN | Static dependency/API audit + forbidden-type tests |
| B-033 | What exact Phys API is the single canonical attachment point for physical assumptions, conventions, provenance, and corpus status defined by external extensions? | OPEN | Phys metadata attachment contract |
| B-034 | How is the boundary enforced so a mathematical condition in PhysMath cannot silently become a physical assumption without an explicit Phys formalization step? | OPEN | Bridge tests + semantic-scope tests |

### Blockers already resolved by constitutional lock

The following are not open architectural questions unless implementation violates them:

```text
layer ownership
mathematical-only ownership of PhysMath
numerical exclusion from PhysMath
compiler-semantics principle
no paper-notation interpreter
one-way Phys dependency
external Phys extension boundary
interpretation-neutral vs framework-specific physical contracts
explicit Phys bridge
no automatic truth adjudication
TOE as workload rather than ontology
```

---

# 27. Lifecycle gates

The PhysMath software lifecycle is gated. The implementation plan is a deliverable of the design process, not the starting point.

## Gate G0 — Charter and boundary lock

Must establish:

- purpose;
- layer boundaries;
- constitutional requirements;
- dependency direction;
- non-goals;
- initial target workload.

**Exit:** zero ambiguity about what PhysMath is and is not.

## Gate G1 — Mathematical semantic model

Must establish:

- core mathematical sorts;
- expression/function/operator distinctions;
- symbol identity;
- equality classes;
- coercion policy;
- assumptions;
- exactness.

**Exit:** no core type can be implemented with multiple incompatible interpretations.

## Gate G2 — Mathematical operation contract

Must establish:

- symbolic algebra;
- calculus;
- tensor/geometry operations;
- higher-order functions/operators;
- approximation/asymptotic support;
- failure semantics;
- bounded automation.

**Exit:** the coding agent knows exactly which transformations exist and what each guarantees.

## Gate G3 — Mathematical execution trace and failure model

Must establish:

- mathematical derivation trace;
- failures;
- counterexamples;
- alternative formulations;
- replay/canonicalization;
- explicit prohibition on physical provenance/trust metadata in PhysMath.

**Exit:** mathematical research state is reproducible without creating a second physical trust system.

## Gate G4 — Phys boundary, external libraries, and numerical handoff

Must establish:

- mathematical-vs-physical object ownership;
- frozen Phys base vs external Phys extension boundary;
- external library admission criteria;
- interpretation-neutral vs framework-specific contract;
- dependency graph for `phys-*` packages;
- export classification;
- information-loss accounting;
- canonical Phys metadata ownership and attachment;
- formalization handoff artifact;
- numerical input contract;
- prohibition on reverse dependencies.

**Exit:** no mathematical/physical ownership ambiguity and no layer overlap remains.

## Gate G5 — AI-agent interface

Must establish:

- machine-readable schemas;
- operation invocation model;
- deterministic identifiers;
- error model;
- context retrieval model;
- mathematical object schemas;
- physical-extension references;
- formalization handoff artifact format.

**Exit:** an AI agent can operate the system without relying on prose interpretation.

## Gate G6 — Verification architecture

Must establish:

- unit/property tests;
- golden derivations;
- adversarial tests;
- negative tests;
- differential/cross-check strategy;
- bridge tests;
- repository/package graph tests.

**Exit:** every semantic guarantee has a corresponding oracle or invariant test.

## Gate G7 — Implementation plan authorization

Only now may the coding agent produce the implementation plan.

The plan MUST contain:

- exact package graph;
- file list or bounded file strategy;
- data structures;
- interfaces;
- operation list;
- test matrix;
- milestone order;
- blocker dependencies;
- explicit non-goals.

The plan MUST NOT reopen G0–G6 decisions without a documented change request.

## Gate G8 — Implementation

Implementation proceeds only against the approved plan.

Any discovered requirement that changes semantic architecture returns to the appropriate gate rather than being patched ad hoc.

## Gate G9 — Adversarial review

Reviewers must actively attempt to break:

- layer boundaries;
- semantic typing;
- symbol identity;
- exactness;
- assumption handling;
- provenance;
- replay;
- cross-framework isolation;
- Phys bridge fidelity;
- numerical handoff;
- AI-generated malformed structures.

## Gate G10 — Freeze

Freeze requires:

- all constitutional requirements passing;
- all blockers closed or explicitly deferred with containment;
- package graph verified;
- adversarial review passed;
- reproducibility checks passed;
- documentation synchronized with implementation;
- implementation tree locked.

---

# 28. Required test families

A credible PhysMath implementation is not established by happy-path derivations alone.

## 28.1 Representation tests

- every core mathematical sort can be constructed;
- malformed objects are rejected;
- no semantic decay occurs when structures are nested;
- functions/operators remain distinguishable from scalar expressions;
- scopes are preserved.

## 28.2 Compiler-semantics tests

- notation/rendering does not create semantics;
- undeclared symbols remain undeclared;
- explicit declarations determine meaning;
- pretty-print changes do not change identity;
- identical spelling in different scopes remains distinct.

## 28.3 Exactness tests

- exact values remain exact;
- no floating-point path is reached from symbolic operations;
- approximation objects retain declared order/conditions;
- approximate equality cannot satisfy exact equality checks.

## 28.4 Mathematical-condition tests

- mathematical condition expressions remain explicit;
- transformation preconditions are not silently fabricated;
- branch conditions remain explicit;
- mathematical domain restrictions are not lost;
- no Phys-style assumption metadata type exists in PhysMath.

## 28.4A Phys metadata ownership tests

- physical assumptions are created only through Phys;
- external `phys-*` packages reuse Phys metadata types;
- no external package defines duplicate provenance/corpus/trust types;
- physical conventions and regimes attach through Phys;
- PhysMath cannot assign corpus status or provenance to a mathematical object.

## 28.5 Derivation tests

Each durable derivation package SHOULD have explicit artifacts equivalent to:

```text
README.md
manifest.json
manifest_test.go
derivation_test.go
```

The derivation test MUST verify real transformation paths rather than merely checking a final constant.

Anti-hardcoding tests SHOULD mutate intermediate states and confirm the final verification fails.

## 28.6 Cross-framework tests

- incompatible mappings are rejected;
- proposed correspondences remain marked proposed;
- established mathematical identities are not upgraded into physical couplings without an explicit relationship object;
- regime mismatch is detected.

## 28.7 Phys bridge tests

- FULLY_REPRESENTABLE exports preserve semantics;
- PARTIALLY_REPRESENTABLE exports record every known loss;
- UNREPRESENTABLE exports do not create trusted objects;
- mathematical conditions survive the bridge where representable;
- physical assumptions, conventions, and provenance are attached through Phys rather than PhysMath;
- Phys internal kernel is never bypassed.

## 28.8 Numerical boundary tests

- numerical packages cannot import PhysMath runtime packages;
- numerical inputs originate from Phys artifacts;
- malformed Phys artifacts stop computation;
- numerical results cannot automatically mutate trusted Phys corpus state.

## 28.9 Adversarial silent-wrongness tests

The primary adversarial category is not a panic. It is a plausible but wrong result.

Tests MUST target:

- type decay;
- namespace collisions;
- incorrect coercion;
- lost assumptions;
- wrong tensor/index contraction;
- wrong dimensional compatibility;
- wrong branch selection;
- silently different representations treated as identical;
- notation-based semantic guessing;
- numerical leakage;
- replay mismatch masked by hashes;
- lossy Phys export.

---

# 29. Review protocol for other AI agents

An external AI review MUST answer the following questions explicitly.

### Boundary review

1. Can any numerical computation occur inside PhysMath?
2. Can PhysMath runtime code leak into Phys?
3. Can Numerical Layer code depend on PhysMath?
4. Can any layer silently adjudicate physical truth?

### Semantic review

5. Is mathematical meaning determined by typed structures rather than notation conventions?
6. Can a symbol change meaning merely because it appears in a familiar physics equation?
7. Are functions, operators, tensors, and relations preserved as distinct semantic categories?
8. Are equality and identity being conflated?

### Mathematical review

9. Can assumptions be lost or fabricated?
10. Can approximation silently become exact or numerical?
11. Can different frameworks be combined without an explicit correspondence?
12. Can a resource failure be mistaken for mathematical contradiction?

### Trust review

13. Can PhysMath bypass Phys trust boundaries?
14. Can a lossy export silently become trusted?
15. Can a numerical result automatically change a Phys corpus status?
16. Can an AI-generated malformed object corrupt trusted state?

### Lifecycle review

17. Are any implementation choices still implicit?
18. Does the coding agent still have to choose among multiple architectures?
19. Does every semantic guarantee have a test oracle?
20. Are unresolved blockers explicitly contained rather than hidden?
21. Can every object be unambiguously classified as mathematical or physical?
22. If physical, is it in Phys base or an explicit external Phys library?
23. If quantum-related, is interpretation dependence explicit?
24. Can an external Phys library reuse PhysMath without creating a cycle?
25. Is the formalization handoff target the Phys ecosystem rather than PhysMath itself?

A review finding that changes the answers to questions 1–4, 5–8, or 13–16 is automatically a **BLOCKER**.

---

# 30. “No overlap” acceptance matrix

The architecture passes the boundary review only if the following statements are all true.

| Statement | MUST be true |
|---|:---:|
| PhysMath can construct a new theoretical mathematical hypothesis without modifying Phys. | ✓ |
| Phys can remain frozen while PhysMath explores mathematics Phys cannot presently express. | ✓ |
| PhysMath never needs a numeric runtime to complete its own job. | ✓ |
| Numerical computation has a clear formal input boundary at Phys and selected Phys extensions. | ✓ |
| No numerical result can silently alter Phys authority. | ✓ |
| No renderer/notation convention can create semantic meaning. | ✓ |
| PhysMath contains mathematical objects, not physical ontology. | ✓ |
| A physical object can be added through an external Phys library without modifying PhysMath. | ✓ |
| Interpretation-specific quantum objects cannot masquerade as universal physical objects. | ✓ |
| A future domain can be added without changing the constitutional layer boundaries. | ✓ |
| The TOE can be implemented as reusable PhysMath + external Phys libraries rather than compiler ontology. | ✓ |
| A failed hypothesis can be retained as useful research data. | ✓ |
| A partially representable hypothesis cannot silently become fully formalized. | ✓ |

---

# 31. Implementation-plan handoff checklist

The coding agent MUST NOT draft implementation details until all required items below are signed off.

## Architecture

- [ ] Layer responsibilities are explicit.
- [ ] Dependency direction is explicit.
- [ ] Numerical exclusion from PhysMath is enforced structurally.
- [ ] Phys trust boundary is explicit.
- [ ] TOE is treated as workload, not ontology.

## Mathematical semantics

- [ ] Core mathematical sorts are defined.
- [ ] Mathematical objects are explicitly separated from physical ontology.
- [ ] No physical interpretation is encoded in mathematical object constructors.
- [ ] Expression/function/operator semantics are defined.
- [ ] Symbol identity and scope are defined.
- [ ] Equality classes are defined.
- [ ] Coercion policy is defined.
- [ ] Mathematical condition/precondition model is defined.
- [ ] Phys metadata is explicitly excluded from PhysMath.
- [ ] Exactness/approximation model is defined.
- [ ] Tensor/index and geometric semantics are defined to the planned scope.

## Operations

- [ ] Algebraic transformations are defined.
- [ ] Calculus operations are defined.
- [ ] Higher-order function/operator operations are defined.
- [ ] Approximation/asymptotic operations are defined.
- [ ] Failure semantics are defined.
- [ ] Resource/termination policy is defined.

## Research state

- [ ] Derivation ledger is defined.
- [ ] Hypothesis states are defined.
- [ ] Counterexample/contradiction states are defined.
- [ ] Replay model is defined.
- [ ] Canonicalization is defined.
- [ ] Alternative representations/correspondences are defined.

## Integration

- [ ] Mathematical-vs-physical ownership rule is defined.
- [ ] Phys base vs external Phys library admission criteria are defined.
- [ ] Interpretation-neutral vs framework-specific physical contracts are defined.
- [ ] External Phys package dependency rules are mechanically enforced.
- [ ] Phys export schema is defined.
- [ ] Loss accounting is defined.
- [ ] Phys metadata attachment/mapping is defined without duplicating metadata in external packages.
- [ ] Formalization handoff schema is defined.
- [ ] Numerical handoff schema is defined.
- [ ] Reverse dependency is mechanically prevented.

## AI interface

- [ ] Machine-readable schemas are versioned.
- [ ] Operation inputs/outputs are typed.
- [ ] Errors are structured.
- [ ] Identifiers are deterministic and scoped.
- [ ] No prose-only semantic dependency remains.

## Testing

- [ ] Golden derivations exist.
- [ ] Negative tests exist.
- [ ] Anti-hardcoding tests exist.
- [ ] Silent-wrongness tests exist.
- [ ] Bridge tests exist.
- [ ] Physical object admission tests exist.
- [ ] Interpretation-ambiguity tests exist.
- [ ] External Phys package dependency tests exist.
- [ ] Numerical boundary tests exist.
- [ ] Package graph tests exist.
- [ ] Adversarial review plan exists.

---

# 32. Completion criteria

PhysMath design is considered **architecturally complete** only when all of the following hold:

1. The three-layer job boundary is mechanically enforceable, not merely documented.
2. The semantic model is explicit enough that a coding agent does not need to guess what an object means.
3. Paper notation is strictly presentation/frontend syntax, never the ontology.
4. The mathematical type system prevents silent category collapse.
5. Exact symbolic reasoning is separated from numerical computation.
6. Mathematical condition expressions remain exact through transformations; physical assumptions, regimes, conventions, and provenance are attached only through Phys.
7. Research failures and counterexamples are first-class data.
8. Cross-framework correspondences are explicit and conditional.
9. Phys export is explicit, loss-aware, and non-authoritative by itself.
10. Every physical object has an explicit home in Phys base or an external Phys library.
11. Interpretation-specific physical semantics are explicit and never silently embedded in mathematical objects.
12. External Phys libraries can depend on PhysMath without creating a reverse dependency.
13. Numerical computation has a clean Phys/extension boundary.
14. Every remaining architectural choice is recorded in a design decision register.
15. The implementation plan can be followed without reopening semantic architecture.

---

# 33. Final architectural invariants

The final system MUST preserve these invariants:

```text
INVARIANT-1
Mathematical notation is not mathematical meaning.

INVARIANT-2
Explicit typed structure is the semantic substrate.

INVARIANT-3
PhysMath is theoretical and exact; numerical computation is elsewhere.

INVARIANT-4
PhysMath may explore without requiring Phys to expand.

INVARIANT-5
Phys remains the trusted formal physics boundary.

INVARIANT-6
Numerical computation begins from Phys formalization.

INVARIANT-7
No layer automatically decides physical truth.

INVARIANT-8
Cross-framework relationships are explicit, not inferred from similarity.

INVARIANT-9
Failed reasoning is data, not merely an exception.

INVARIANT-10
The TOE is a stress test of the architecture, not the architecture itself.

INVARIANT-11
No silent information loss across trust boundaries.

INVARIANT-12
No coding agent may choose among unresolved architectures.

INVARIANT-13
PhysMath owns mathematical objects; Phys owns physical meaning.

INVARIANT-14
A physical object not already in Phys is added as an external Phys library object, not as PhysMath ontology.

INVARIANT-15
Interpretation dependence must be explicit; ambiguous physical concepts cannot be represented as universal contracts by implication.

INVARIANT-16
The formalization handoff is to the Phys ecosystem, not to PhysMath itself.

INVARIANT-17
PhysMath has no competing trust metadata plane; physical assumptions, conventions, provenance, corpus status, and MRC authority belong to Phys.

INVARIANT-18
External phys-* libraries define physical semantics but reuse the canonical Phys metadata model and remain external modules.

INVARIANT-19
A mathematical condition is not a physical assumption until an explicit Phys formalization step gives it that role.
```

---

# 34. Design decision register requirement

Every blocker closed during review MUST produce a stable decision entry containing:

```text
Decision ID
Question
Decision
Rationale
Alternatives rejected
Affected requirements
Affected packages
Migration impact
Tests required
Review status
```

Decisions that affect semantic behavior MUST be frozen before implementation of dependent packages.

The implementation repository SHOULD contain a machine-readable or otherwise mechanically checkable representation of the finalized decision set.

---

# 35. Change-control rule

A later request that changes any constitutional requirement MUST be treated as an architecture change, not a feature request.

Examples:

- “Add numerical experimentation to PhysMath.”
- “Let Phys depend on PhysMath.”
- “Infer what `G_{μν}` means from conventional physics notation.”
- “Let a successful export automatically become trusted corpus data.”
- “Make the Numerical Layer call PhysMath to discover missing semantics.”
- “Put `QuantumState` or `BohmianConfiguration` into PhysMath because their mathematics already exists there.”
- “Use `WaveFunction` as a universal physical ontology without declaring an interpretation/framework.”
- “Create a package-local provenance or assumption system inside `phys-qm`, `phys-bohm`, or another external Phys library.”
- “Store physical corpus status or MRC authority inside PhysMath.”
- “Add a new physical domain by modifying PhysMath instead of creating a Phys extension.”

Each requires reopening this document and passing the appropriate lifecycle gate.

---

# 36. Final handoff condition

**Implementation planning is authorized only when the architectural blockers are closed or explicitly deferred with containment, and an independent AI review confirms that the three jobs do not overlap.**

The intended outcome is not a larger Phys, not a numerical CAS disguised as PhysMath, and not a physics ontology disguised as reusable mathematics.

The intended outcome is:

```text
                         AI THEORETICAL REASONING
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │        PhysMath         │
                    │                         │
                    │ mathematical objects   │
                    │ exact symbolic work    │
                    │ hypothesis construction│
                    │ derivation / challenge │
                    │ reusable math libraries│
                    └────────────┬────────────┘
                                 │ math + explicit handoff
                                 ▼
                    ┌─────────────────────────┐
                    │          Phys           │
                    │   trusted base layer    │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
        ┌──────────┐       ┌───────────┐      ┌──────────┐
        │ phys-qm  │       │ phys-bohm │      │ phys-gr  │
        │ physical │       │ physical  │      │ physical │
        │ contracts│       │ ontology  │      │ contracts│
        └────┬─────┘       └─────┬─────┘      └────┬─────┘
             └───────────────────┼──────────────────┘
                                 │ formalized model
                                 ▼
                    ┌─────────────────────────┐
                    │    Numerical Layer      │
                    │                         │
                    │ evaluation / simulation │
                    │ fitting / optimization  │
                    │ empirical comparison    │
                    └─────────────────────────┘
```

**The boundary is the architecture.**

---

## Appendix A — Source workload note

The current Dancer–Dance–Stage TOE blueprint is treated only as the initial acceptance workload. It explicitly frames the three sectors and their interfaces and identifies the still-open joint dynamics coupling the causal structure, wavefunctional, and actual configuration as the place where a genuinely new physical claim would reside. That is useful for stress-testing PhysMath's ability to represent interfaces without turning the TOE into hard-coded ontology.

The source workload does not define the general PhysMath mathematical ontology or the admission criteria for external Phys libraries.

---

## Appendix B — Minimal review verdict template

```text
REVIEW: PhysMath Architectural Requirements v0.4

Constitutional boundary: PASS / FAIL
Mathematical-object ownership: PASS / FAIL
Physical-object extension boundary: PASS / FAIL / OPEN
Interpretation-ambiguity rule: PASS / FAIL / OPEN
Paper-notation semantic boundary: PASS / FAIL
Math IR/type model: PASS / FAIL / OPEN
Exactness boundary: PASS / FAIL
Mathematical-condition model: PASS / FAIL / OPEN
Phys metadata ownership: PASS / FAIL / OPEN
Mathematical execution trace: PASS / FAIL / OPEN
Cross-framework model: PASS / FAIL / OPEN
External Phys libraries: PASS / FAIL / OPEN
Phys bridge: PASS / FAIL / OPEN
Formalization handoff: PASS / FAIL / OPEN
Numerical handoff: PASS / FAIL / OPEN
AI interface: PASS / FAIL / OPEN
Testing architecture: PASS / FAIL / OPEN
Lifecycle gates: PASS / FAIL

BLOCKERS:
- ...

MAJOR:
- ...

MINOR:
- ...

FINAL STATUS:
NOT READY / CONDITIONAL / READY FOR IMPLEMENTATION PLAN
```
