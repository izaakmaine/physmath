# PhysMath Architecture — Consolidated Requirements, Boundaries, Decision Register, and Implementation-Authorization Checklist

**Document ID:** PM-ARCH-REQ-001  
**Version:** 1.0  
**Status:** **ARCHITECTURE CLOSED — IMPLEMENTATION PLAN AUTHORIZED**  
**Purpose:** Eliminate architectural blockers and ambiguity for coding agents across the complete lifecycle of the PhysMath ecosystem: requirements, architecture, implementation, adversarial review, acceptance, and freeze.

---

# 0. Executive decision

The four adversarial reviews were read in full and consolidated. The reviews agree that the three-project split is the correct architecture; the remaining findings concern **mechanical closure of interfaces, schemas, ownership, replay, metadata, and acceptance criteria**, not reopening the architecture itself.

The architecture is now:

```text
PROJECT A — TOE-Agent / AI Research Program
    hypothesis construction
    hypothesis chains
    attacks / counterexamples
    correspondence proposals
    research state
    handoff manifests

PROJECT B — PhysMath
    reusable exact mathematics
    theoretical mathematical physics
    mathematical IR
    symbolic mathematical transformations
    mathematical traces

PROJECT C — Phys External Libraries
    reusable physical-domain semantics
    phys-qm
    phys-bohm
    phys-gr
    phys-cst / phys-cdt
    future physical domains

EXISTING FROZEN PHYS
    trusted physical substrate
    canonical physical metadata
    physical assumptions
    physical conventions
    provenance
    corpus status
    MRC / trust
    trusted derivation / ledger authority

NUMERICAL LAYER
    numerical computation
    simulation
    quantitative evaluation
    empirical comparison
```

The central separation is absolute:

```text
TOE-Agent        = reasons about hypotheses

PhysMath         = provides reusable mathematics

phys-*           = provides reusable physical semantics

Phys             = owns physical trust/metadata and trusted formalization

Numerical Layer  = computes quantitative consequences
```

No project may silently absorb another project's job.

---

# 1. Constitutional requirements

The following are **LOCKED**.

| ID | Requirement | Status |
|---|---|---|
| REQ-PM-CON-001 | PhysMath is limited to theoretical mathematical physics and exact symbolic reasoning. | LOCKED |
| REQ-PM-CON-002 | PhysMath contains no numerical runtime. | LOCKED |
| REQ-PM-CON-003 | PhysMath performs no floating-point computation, simulation, Monte Carlo, optimization, fitting, numerical ODE/PDE solving, GPU computation, or empirical-data analysis. | LOCKED |
| REQ-PM-CON-004 | Exact symbolic integer/rational computation is permitted. | LOCKED |
| REQ-PM-CON-005 | Mathematical notation is presentation, not semantics. | LOCKED |
| REQ-PM-CON-006 | Mathematical semantics arise from explicit typed mathematical structures. | LOCKED |
| REQ-PM-CON-007 | A future notation frontend, if any, must desugar into explicit mathematical structures and may not infer undeclared physics semantics. | LOCKED |
| REQ-PM-CON-008 | PhysMath is not a physics-paper notation interpreter. | LOCKED |
| REQ-PM-CON-009 | Dancer/Dance/Stage is an acceptance workload, not PhysMath ontology. | LOCKED |
| REQ-PM-CON-010 | PhysMath must support theoretical physics beyond the initial TOE. | LOCKED |
| REQ-PM-CON-011 | PhysMath never promotes work into trusted Phys authority. | LOCKED |
| REQ-PM-CON-012 | Formalization from research into Phys occurs only through the explicit handoff contract defined here. | LOCKED |
| REQ-PM-CON-013 | Phys remains below all layers that consume it and never depends on PhysMath or phys-* extensions. | LOCKED |
| REQ-PM-CON-014 | Numerical computation consumes formalized Phys/phys-* computational contracts, not raw PhysMath research state. | LOCKED |
| REQ-PM-CON-015 | No layer equates mathematical consistency with empirical truth. | LOCKED |
| REQ-PM-CON-016 | Contradictions, failures, counterexamples, unresolved obligations, and unrepresentability are first-class research outcomes. | LOCKED |
| REQ-PM-CON-017 | PhysMath contains mathematical objects and mathematical relationships only. | LOCKED |
| REQ-PM-CON-018 | A mathematical object's name never gives it physical semantics. | LOCKED |
| REQ-PM-CON-019 | Physical objects are defined only in Phys or external phys-* libraries. | LOCKED |
| REQ-PM-CON-020 | External phys-* libraries are separately versioned modules above Phys. | LOCKED |
| REQ-PM-CON-021 | Interpretation dependence is allowed only when explicitly declared. | LOCKED |
| REQ-PM-CON-022 | PhysMath cannot own physical observables, physical regimes, physical ontology, or framework-specific semantics. | LOCKED |
| REQ-PM-CON-023 | Physical trust metadata has one canonical owner: Phys. | LOCKED |
| REQ-PM-CON-024 | PhysMath and phys-* cannot implement parallel assumption/provenance/corpus/trust systems. | LOCKED |
| REQ-PM-CON-025 | The TOE is a research program and is never architecturally reduced to one monolithic Go program. | LOCKED |
| REQ-PM-CON-026 | Composability, extensibility, and reusability are first-class architectural goals. | LOCKED |

---

# 2. The five authorities/projects and exact job boundaries

## 2.1 Project A — TOE-Agent

Project A is the **AI reasoning/research program**.

It owns:

- hypotheses;
- hypothesis chains;
- research obligations;
- research status;
- attack strategies;
- counterexample records;
- cross-framework correspondence proposals;
- TOE workload orchestration;
- references to PhysMath traces;
- references to phys-* contracts;
- formalization handoff packages.

It MUST NOT own:

- a second symbolic mathematics engine;
- physical ontology;
- canonical Phys assumptions;
- canonical provenance;
- corpus status;
- MRC/trust authority;
- numerical execution.

Project A is the only project that combines multiple hypothesis-level physical claims into a research program.

---

## 2.2 Project B — PhysMath

PhysMath is a **reusable mathematics library project**.

It owns:

- mathematical structures;
- mathematical types/sorts;
- exact symbolic transformations;
- mathematical functions and functionals;
- mathematical operators;
- tensor/index structures;
- mathematical geometry;
- mathematical conditions;
- exact limits/asymptotics where supported;
- mathematical equivalence machinery;
- mathematical traces.

It MUST NOT own:

- physical ontology;
- physical object identity;
- physical assumptions as epistemic metadata;
- physical conventions;
- physical provenance;
- corpus status;
- MRC/trust;
- empirical evidence;
- numerical execution.

A PhysMath mathematical trace is **not provenance**.

---

## 2.3 Project C — Phys External Libraries

Project C is the family of external physics modules.

Examples:

```text
phys-qm
phys-bohm
phys-gr
phys-cst
phys-cdt
```

These names are representative targets, not a frozen repository tree.

External libraries own:

- physical-domain objects;
- physical relationships;
- physical interpretation;
- framework scope;
- physical contract definitions;
- domain-specific physical mappings to PhysMath mathematics.

They MUST NOT own:

- a duplicate mathematical engine;
- a duplicate assumption system;
- a duplicate provenance system;
- a duplicate corpus-status system;
- a duplicate trust/MRC system;
- physical metadata authority.

External libraries remain external even when they are essential to the TOE.

---

## 2.4 Existing Phys

Phys is the **trusted physical foundation**.

Phys owns:

- canonical physical metadata;
- physical assumptions;
- physical conventions;
- physical provenance;
- physical corpus status;
- MRC/trust authority;
- trusted derivation/session/ledger semantics;
- formalized physical object authority where the frozen model supports it;
- Growth Gate decisions for kernel changes.

Phys MUST NOT become:

- a general mathematics engine;
- a general CAS;
- a research hypothesis runtime;
- a numerical runtime;
- a repository for every new domain-specific object.

---

## 2.5 Numerical Layer

The Numerical Layer owns:

- numerical evaluation;
- numerical solvers;
- simulation;
- numerical optimization;
- parameter fitting;
- Monte Carlo;
- quantitative prediction;
- empirical comparison;
- accelerator/GPU execution.

It MUST consume a declared computational contract produced from formalized Phys/phys-* artifacts.

It MUST NOT:

- infer missing physical semantics;
- repair missing assumptions;
- reinterpret mathematical notation;
- overwrite physical meaning;
- turn an unformalized research artifact into an authoritative physical model.

---

# 3. Absolute non-overlap matrix

| Concern | TOE-Agent | PhysMath | phys-* | Phys | Numerical |
|---|---|---|---|---|---|
| Hypothesis construction | PRIMARY | Supports | Supports contracts | Receives | No |
| Hypothesis chain | PRIMARY | Referenced | Referenced | Formalization target | No |
| Mathematical objects | References | **PRIMARY** | Reuses | Selected subset | Consumes derived contract only |
| Mathematical transformations | Orchestrates | **PRIMARY** | Reuses | Bounded | Numerical analogues only |
| Mathematical conditions | Owns research references | **PRIMARY** | Reuses | Interprets as physical assumptions when needed | Consumes formalized results |
| Physical objects | References | NEVER | **PRIMARY** | Trusted subset | Consumes |
| Physical assumptions | References | NEVER | Declares requirements only | **PRIMARY** | Consumes |
| Physical conventions | References | NEVER | Declares requirements only | **PRIMARY** | Consumes |
| Physical provenance | References | NEVER | NEVER duplicates | **PRIMARY** | Consumes |
| Corpus status | Research status only | NEVER | NEVER duplicates | **PRIMARY** | Consumes |
| MRC/trust | NEVER | NEVER | NEVER | **PRIMARY** | NEVER |
| Mathematical trace | **References** | **PRIMARY** | May reference | May reference hash | Computation log only |
| Physical ledger | References | NEVER owns | References | **PRIMARY** | No |
| Numerical execution | NEVER | NEVER | NEVER | NEVER | **PRIMARY** |
| Empirical adjudication | NEVER | NEVER | NEVER | NEVER | External scientific process |
| TOE ontology | PRIMARY workload labels | NEVER | External contracts | Formal physical subset | Never |

---

# 4. Dependency graph

The graph is:

```text
                           Phys
                            ▲
                ┌───────────┼───────────┐
                │           │           │
             PhysMath      phys-qm     phys-gr
                ▲            │           │
                │            │           │
                └─────┬──────┴─────┬─────┘
                      │            │
                   phys-bohm    phys-cst/cdt
                      │            │
                      └──────┬─────┘
                             ▲
                             │
                         TOE-Agent
```

For clarity, the arrows mean:

```text
PhysMath  ─────► Phys
phys-*    ─────► Phys
phys-*    ─────► PhysMath    only for reusable mathematics
TOE-Agent ─────► PhysMath
TOE-Agent ─────► Phys
TOE-Agent ─────► selected phys-*
Numerical ─────► Phys
Numerical ─────► selected phys-*
```

Additional rules:

1. Phys never imports anything above it.
2. PhysMath core mathematical packages never import phys-*.
3. PhysMath's project-level dependency on Phys is limited to a designated interoperability/reference boundary; the pure mathematical core remains physically agnostic.
4. phys-* may depend on PhysMath for mathematics.
5. phys-* may depend on other phys-* only through public physical contracts, with no cycles.
6. TOE-Agent may compose projects but may not import private/internal implementation packages.
7. Numerical Layer never consumes Project A research state as physical authority.

---

# 5. Resolving the PhysMath → Phys / mathematics-only tension

This was one of the strongest review findings.

The resolution is:

```text
PhysMath project
    │
    ├── math-core/*
    │      NO Phys import
    │
    ├── math-domain/*
    │      NO physical semantics
    │
    └── physref/*
           public Phys API only
           opaque references only
           no physical semantics
```

The **project-level dependency** on Phys is retained, but the mathematical core cannot import physical semantics.

The interoperability/reference boundary may:

- reference a Phys artifact ID;
- reference a Phys schema/version;
- hold an opaque Phys object reference;
- verify compatibility with a Phys public schema.

It may not:

- define physical ontology;
- create physical assumptions;
- create provenance;
- attach corpus status;
- decide interpretation.

This makes the dependency mechanically real without contaminating mathematical semantics.

---

# 6. PhysMath mathematical object ownership

The classifier is semantic.

A mathematical object is defined entirely by mathematical structure.

A physical object asserts a physical role.

Examples:

| Concept | Home | Meaning |
|---|---|---|
| Hilbert space | PhysMath | Mathematical structure |
| Vector space | PhysMath | Mathematical structure |
| Inner product | PhysMath | Mathematical operation/structure |
| Operator | PhysMath | Mathematical operator |
| Function | PhysMath | Mathematical function |
| Wavefunction as a mathematical function | PhysMath | Mathematical function with declared domain/codomain |
| Quantum state as a physical state | phys-qm | Physical contract |
| Bohmian wavefunction as physically operative | phys-bohm | Framework-specific physical contract |
| Bohmian configuration Q | phys-bohm | Framework-specific physical object |
| Physical spacetime | phys-gr / phys-cst / phys-cdt as applicable | Physical contract |
| Mass / energy / physical momentum | Phys or appropriate phys-* | Physical object |

The name `WaveFunction` does not decide the classification.

---

# 7. Mathematical conditions versus physical assumptions

These are different artifact classes.

PhysMath may represent:

```text
x > 0
x != 0
det(g) != 0
r > 2M
domain(f)
boundary_condition
```

as mathematics.

These are not Phys assumptions until a physical contract explicitly interprets them.

Phys owns:

```text
rest_mass_nonnegative
special_relativity_regime
metric.signature = -+++
source = ...
corpus_status = ...
```

The hard rule is:

```text
mathematical condition  -> PhysMath
physical assumption     -> Phys
physical convention     -> Phys
physical regime         -> Phys
physical provenance     -> Phys
corpus/trust status     -> Phys
```

The conversion from a mathematical condition into a physical assumption is a **formalization action**.

---

# 8. Phys metadata monopoly

There is exactly one physical trust/metadata model.

Phys owns:

```text
Assumption
Convention
Provenance
CorpusStatus
MRC
Trust
PhysicalFormalization
TrustedLedger
```

PhysMath MUST NOT implement equivalents.

External phys-* MUST NOT implement equivalents.

An external library may contain an opaque:

```text
PhysMetadataRef
PhysMetadataRequestRef
```

but these are references, not local metadata systems.

This is the only permitted extension pattern.

---

# 9. External phys-* object contract

External physical objects are not forced into the frozen Phys `Kind`/`Dimension` model.

Each phys-* package defines its own domain types.

Every external object participating in a cross-project artifact also exposes a standard envelope:

```text
PhysicalArtifactEnvelope
    schema_version
    extension_id
    object_type
    object_id
    math_refs[]
    physical_contract_refs[]
    phys_metadata_refs[]
    payload
```

Rules:

1. `object_type` is namespaced to the extension.
2. `object_id` is stable within the extension's schema version.
3. `math_refs` point to exact PhysMath objects.
4. `physical_contract_refs` identify the physical meaning defined by the extension.
5. `phys_metadata_refs` are references to canonical Phys records only.
6. `payload` contains extension-owned physical structure.
7. No envelope contains a package-local provenance/assumption/corpus/trust system.

External objects may remain fully external.

---

# 10. External-library admission

A new physical concept is routed as follows:

```text
Pure mathematics?
    YES -> PhysMath

Physical meaning already covered by frozen Phys?
    YES -> Phys

Physical meaning not covered by frozen Phys?
    YES -> external phys-*
```

A concept goes into frozen Phys only when all applicable conditions hold:

1. foundational;
2. theory-neutral within declared scope;
3. reusable across multiple physical domains or independently foundational;
4. representable by the existing Phys substrate without semantic distortion;
5. not merely required by one project;
6. not an interpretation-specific ontology;
7. independently justified under the Phys Growth Gate.

Everything else stays external.

This is the default:

```text
EXTERNAL FIRST
    ↓
independent evidence
    ↓
Growth Gate
    ↓
possible Phys promotion
```

The TOE cannot force Phys kernel growth.

---

# 11. Interpretation ambiguity

Interpretation-specific physics is permitted.

Silent interpretation is prohibited.

A phys-* object MUST be classified as either:

```text
INTERPRETATION_NEUTRAL
```

or

```text
FRAMEWORK_SPECIFIC
```

If multiple incompatible interpretations exist, the library MUST:

- split the concepts;
- or make the framework scope explicit.

It MUST NOT define an ambiguous universal object.

Canonical example:

```text
PhysMath
    HilbertSpace
    Vector
    Function
    Operator
    WaveFunction (mathematical)

phys-qm
    QuantumState

phys-bohm
    BohmianWaveFunction
    BohmianConfiguration
    GuidanceLaw
```

The exact Go names are implementation choices; the semantic separation is mandatory.

---

# 12. PhysMath IR architecture

PhysMath is not a single scalar-expression AST.

Its core mathematical IR is **multi-sorted**.

Required semantic categories:

```text
Expression
Symbol
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
Set / Collection
Transformation
Limit
Asymptotic / Truncation
DerivationObligation
```

The core sort set is closed and versioned for each PhysMath major specification.

Extensions may introduce new mathematical sorts only through a versioned extension protocol.

An extension sort MUST declare:

- stable sort ID;
- schema version;
- canonical representation;
- equality semantics;
- hashing semantics;
- serialization;
- allowed composition;
- transformation support;
- resource bounds.

Dynamic unrestricted sort creation at runtime is prohibited.

This preserves extensibility without sacrificing determinism.

---

# 13. Symbol identity, scopes, and binding

Display spelling never determines identity.

The model MUST distinguish:

- global symbols;
- local symbols;
- bound variables;
- function parameters;
- tensor indices;
- dummy variables;
- generated identifiers.

Canonicalization MUST be based on declared scope/binding identity.

Alpha-equivalent bound structures must canonicalize consistently.

Index names are presentation only.

Thus:

```text
r
```

in one mathematical scope is not automatically the same object as:

```text
r
```

in another scope.

---

# 14. Mathematical equality

PhysMath MUST distinguish:

1. structural identity;
2. definitional equality;
3. declared mathematical equivalence;
4. approximate/asymptotic equivalence;
5. physical identification.

Only structural identity drives canonical hashes.

A physical identification is never created by mathematical equality.

A research correspondence is never silently upgraded to an identity.

---

# 15. Canonicalization and hashing

Every canonicalizable PhysMath artifact is serialized with:

```text
schema_version
sort
canonical_payload
```

Canonicalization rules include:

- deterministic field order;
- deterministic child order where commutativity permits;
- deterministic numeric normalization;
- stable scope/binder representation;
- no dependence on pretty-printing;
- no dependence on map iteration order;
- no hidden timestamps/randomness;
- no floating-point serialization for exact values.

Hashing is over canonical bytes.

Mathematical equivalence that is not structural is never encoded by changing the content hash.

---

# 16. Exactness and symbolic operations

PhysMath permits exact theoretical operations.

Examples:

```text
integer arithmetic
rational arithmetic
symbolic substitution
symbolic differentiation
exact algebraic simplification
exact tensor contractions where supported
formal limits
formal asymptotics
```

PhysMath does not become an unrestricted CAS.

The operation set is bounded by explicit contracts.

A missing operation fails explicitly instead of silently invoking a general-purpose solver.

---

# 17. Higher-order mathematics

PhysMath MUST be able to represent the mathematics required by theoretical physics without pretending every object is a scalar expression.

This includes, where supported:

```text
functions
functionals
operators
operator applications
maps
tensor fields
indexed families
differential operators
variational expressions
geometric structures
```

The implementation MUST type-check domain/codomain compatibility before applying transformations.

A scalar expression tree cannot impersonate a function, operator, tensor, manifold, or transformation merely by naming.

---

# 18. Limits and asymptotics

PhysMath supports theoretical asymptotic reasoning.

An asymptotic artifact records at least:

```text
source expression
expansion/limit parameter
direction/domain information
order
truncation
remainder notation if applicable
```

Example:

```text
F(ε) = a0 + a1 ε + O(ε²)
```

is mathematical structure.

It does not imply a physical regime.

The physical statement that:

```text
ε -> 0
```

corresponds to a specific physical limit belongs to an external physical contract.

No numerical evaluation is implied.

---

# 19. Time flow versus RG scale flow

PhysMath distinguishes mathematical parameter identity.

` t ` and ` k ` are different mathematical parameters unless an explicit mathematical mapping says otherwise.

A derivative carries its parameter reference.

Therefore:

```text
∂/∂t
```

and

```text
∂/∂k
```

cannot become identical because both are derivatives.

Physical roles such as:

```text
t = physical time
k = renormalization scale
```

are supplied by physical contracts in Project C / Phys.

This directly prevents the TOE category error:

```text
physical time evolution != RG/coarse-graining flow
```

---

# 20. Mathematical execution trace

PhysMath MAY persist a mathematical trace.

The trace contains:

```text
MathTrace
    trace_version
    root_objects[]
    steps[]
    intermediate_hashes[]
    failures[]
    counterexamples[]
    unresolved_obligations[]
```

A step records:

```text
operation_id
input_refs[]
output_refs[]
parameters
preconditions
result_hash
```

The trace is:

- deterministic;
- replayable in PhysMath;
- mathematical;
- non-authoritative physically.

It is not:

- provenance;
- corpus status;
- empirical evidence;
- trust authority.

---

# 21. Two replay domains

There are exactly two formal replay domains:

## PhysMath replay

Replays mathematical operations over PhysMath objects.

## Phys replay

Replays trusted Phys operations over Phys physical artifacts.

A Phys session MUST NOT claim to replay an arbitrary PhysMath trace.

A Phys formalization may reference:

```text
MathTraceDigest
```

and the formalized Phys artifact may contain references to the relevant mathematical source.

The replay boundary is explicit.

---

# 22. Project A research state

Project A owns research workflow state.

A hypothesis record contains:

```text
ResearchHypothesis
    schema_version
    hypothesis_id
    parent_ids[]
    mathematical_refs[]
    physical_contract_refs[]
    correspondence_refs[]
    trace_refs[]
    obligations[]
    status
```

Project A can combine multiple PhysMath traces and multiple physical contracts.

PhysMath does not own the chain.

External phys-* does not own the chain.

Phys does not become the research workflow engine.

---

# 23. Research statuses versus failure classes

These vocabularies are deliberately separate.

## Research status

The following status values are reserved for Project A:

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

`FORMALIZED` means a formalization artifact exists.

It does not mean:

```text
TRUTH_ESTABLISHED
EMPIRICALLY_CONFIRMED
CORPUS_ESTABLISHED
```

## Failure/obstacle class

Independent obstacle labels are:

```text
SUBSTRATE_BOUND
PACKAGE_SOLVABLE
REPRESENTATION_UNFAITHFUL
ENCODING_UNREPRESENTABLE
SILENT_SEMANTIC_ERROR
```

These labels are never reused as Phys corpus status.

This removes the prior `UNREPRESENTABLE` vocabulary collision.

---

# 24. Cross-framework correspondence artifact

Project A owns research-level correspondences.

The canonical schema is:

```text
CorrespondenceProposal
    schema_version
    correspondence_id
    source_framework_refs[]
    target_framework_refs[]
    source_math_refs[]
    target_math_refs[]
    mapping_refs[]
    assumptions_math_refs[]
    physical_contract_refs[]
    regime_refs[]
    direction
    exactness
    status
    obligations[]
    evidence_refs[]
```

The artifact states:

> “This relationship is proposed.”

It never silently asserts:

```text
isomorphism
identity
duality
physical equivalence
empirical truth
```

unless those are separately formalized and justified.

---

# 25. External phys-* composability

External physical packages may compose.

Composition requires:

1. public contract references;
2. explicit dependency direction;
3. namespace-qualified object identities;
4. declared compatibility conditions;
5. no local trust systems;
6. no cyclic package dependencies;
7. no hidden interpretation conversion.

For example:

```text
phys-bohm
    depends on public phys-qm contract

phys-bohm
    depends on PhysMath mathematics

phys-qm
    depends on Phys + PhysMath

phys-gr
    depends on Phys + PhysMath
```

A cross-package relationship is an explicit contract, not an automatic consequence of similar types.

---

# 26. Formalization handoff

The formalization handoff is a **Project A artifact**.

The receiving authority is the Phys ecosystem:

```text
Phys
+
relevant phys-* library
```

The handoff contains:

```text
HandoffPackage
    schema_version
    research_id
    hypothesis_chain_id
    source_math_trace_digest
    math_object_refs[]
    physical_object_refs[]
    physical_contract_refs[]
    correspondence_refs[]
    mathematical_conditions[]
    requested_phys_metadata[]
    derivation_refs[]
    unresolved_obligations[]
    export_items[]
```

The handoff is the only formal crossing from research into trusted formalization.

---

# 27. Handoff classification and loss accounting

Every exported component is classified:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

Each export item contains:

```text
source_ref
target_ref
classification
mapping
losses[]
approximations[]
conditions[]
unresolved_obligations[]
```

No information may be silently discarded.

A partial export cannot become trusted by implication.

An unrepresentable component remains explicitly external.

---

# 28. Handoff precedence

When a research artifact and a Phys canonical artifact refer to the same physical identity:

```text
Phys canonical physical artifact
    has precedence for trusted physical semantics.
```

Research artifacts are immutable source evidence.

They cannot overwrite:

- Phys assumptions;
- Phys conventions;
- Phys provenance;
- Phys corpus status;
- trusted Phys object state.

The handoff records both:

```text
source hash
target identity
```

so the two artifacts remain auditable.

---

# 29. External-library trust boundary

The current frozen Phys MVP may not support trusted corpus admission for every future external domain.

This is explicitly accepted.

For external-domain objects not supported by the frozen Phys corpus model:

```text
external physical definition
        ↓
reusable external library
        ↓
research use permitted
        ↓
formalization package
        ↓
UNREPRESENTABLE / PARTIALLY_REPRESENTABLE / FULLY_REPRESENTABLE
```

No external library is allowed to invent a hidden curation mechanism.

Until a future Phys specification defines a curation/admission mechanism, new-domain objects remain external and are not automatically trusted Phys corpus objects.

The Phys Growth Gate is not a curation shortcut.

---

# 30. Minimal TOE Phys handoff witness

The TOE stress test must exercise the Phys boundary even though much of the TOE will exceed the frozen Phys object model.

Therefore TOE acceptance requires at least:

```text
ONE end-to-end FULLY_REPRESENTABLE Phys fragment
+
ONE explicitly UNREPRESENTABLE or PARTIALLY_REPRESENTABLE fragment
```

within the overall TOE research program.

The purpose is to prove both:

```text
formalization works where it can
```

and:

```text
formalization fails honestly where it cannot
```

No requirement exists that all Dancer/Dance/Stage objects become Phys corpus objects.

This is deliberate.

---

# 31. TOE joint dynamics acceptance

The TOE blueprint identifies a research obligation of the form:

\[
\mathcal D[\Psi,Q,\mathcal C]=0
\]

The architecture does not define the equation.

Project A may formulate candidate versions.

PhysMath provides the mathematics needed to represent and manipulate them.

phys-* packages provide the physical meanings of:

```text
Psi
Q
C
```

and related objects.

Phys formalizes the projectable physical subset.

Therefore the TOE test asks:

```text
Can the architecture formulate and audit a candidate joint law?
```

It does not ask:

```text
Does the architecture already know the joint law?
```

---

# 32. Dancer / Dance / Stage workload mapping

## Dancer

Mathematical workload:

```text
Hilbert spaces
function spaces
wavefunctions as mathematical functions
operators
operator algebra
configuration-space mathematics
```

Physical workload:

```text
quantum state
Bohmian wavefunction
Bohmian configuration
beables
```

Physical side belongs to phys-qm / phys-bohm.

## Dance

Mathematical workload:

```text
differential equations
functional derivatives
flows
variational structures
formal RG mathematics
exact transformations
```

Physical workload:

```text
time evolution
guidance dynamics
field dynamics
coarse-graining
RG interpretation
```

Physical side belongs to relevant phys-* contracts.

## Stage

Mathematical workload:

```text
posets
causal order
graphs
simplicial complexes
triangulations
tensor geometry
metrics
connections
curvature
combinatorial histories
```

Physical workload:

```text
spacetime
causal structure
dynamical geometry
physical regime
```

Physical side belongs to phys-gr / phys-cst / phys-cdt or other declared extensions.

---

# 33. TOE interfaces

The research program must be capable of formulating:

```text
Dancer ↔ Stage
Dance  ↔ Stage
Dancer ↔ Dance
```

The architecture never infers these interfaces from notation.

Each interface consists of:

```text
Project A correspondence proposal
+
PhysMath mathematical mappings
+
phys-* physical contracts
+
Phys metadata at formalization
```

No additional permanent “Interpretation Layer” exists.

---

# 34. AI-agent protocol

Project A communicates with PhysMath and phys-* through public machine-readable structures.

The protocol is versioned.

Core artifact families:

```text
MathObject
MathTrace
ResearchHypothesis
CorrespondenceProposal
PhysicalContractRef
HandoffPackage
```

The AI agent MUST NOT use prose as the authoritative representation of mathematical state.

Prose is documentation only.

A future transport may be:

- direct Go API;
- file artifact;
- process boundary;
- service boundary.

The canonical semantics are the versioned schemas, not the transport.

This deliberately avoids coupling architecture to one deployment style.

---

# 35. Numerical handoff contract

The Numerical Layer may consume:

```text
formalized Phys object
+
phys-* computational contract
+
explicit numerical input
+
declared numerical approximation policy
```

It may not consume:

```text
raw ResearchHypothesis
raw MathTrace
raw CorrespondenceProposal
```

as authoritative physical semantics.

A numerical implementation must stop when required physical semantics are missing.

It cannot fill gaps with guessed assumptions.

---

# 36. Resource and adversarial containment

All externally supplied mathematical/research artifacts are untrusted input.

Each implementation MUST define bounded resources for:

- expression depth;
- object graph size;
- tensor rank/size;
- recursion depth;
- transformation count;
- memory;
- serialization size;
- trace length.

Resource exhaustion is represented explicitly as:

```text
RESOURCE_LIMIT
```

It is never treated as proof of mathematical inconsistency.

Malformed input must not corrupt:

- canonical state;
- traces;
- handoff artifacts;
- Phys metadata;
- trusted Phys ledgers.

---

# 37. No-Go architectures

The following are explicitly rejected:

1. PhysMath as a general CAS.
2. PhysMath as a numerical engine.
3. PhysMath containing physical ontology.
4. PhysMath containing its own trust metadata.
5. phys-* packages containing duplicate provenance/assumption systems.
6. Dancer/Dance/Stage compiler ontology.
7. A permanent Interpretation Layer.
8. A Candidate Physical Object layer inside PhysMath.
9. Automatic conversion from mathematical equivalence to physical identity.
10. Automatic hypothesis promotion.
11. A single monolithic TOE Go application.
12. Dynamic unrestricted runtime sort invention.
13. Numerical Layer semantic repair.
14. Phys kernel growth merely because a TOE workload is difficult.

---

# 38. Growth Gate routing

When a capability is missing:

```text
Can it live in PhysMath as mathematics?
    YES -> PhysMath

Can it live in an external phys-* domain package?
    YES -> phys-*

Is it a foundational, theory-neutral Phys primitive?
    YES -> Growth Gate

Otherwise
    -> remain external
```

The burden of kernel growth remains on evidence.

The TOE is a stress test, not a reason to weaken the Growth Gate.

---

# 39. Architectural decision register

All prior blocker findings are resolved by explicit decisions.

| ADR | Decision | Resolution | Status |
|---|---|---|---|
| ADR-001 | Project ownership | TOE-Agent, PhysMath, phys-* suite explicitly separate; Phys and Numerical remain separate authorities. | CLOSED |
| ADR-002 | Dependency graph | Acyclic graph defined in §4. | CLOSED |
| ADR-003 | PhysMath/Phys dependency | Project-level dependency retained; pure math core isolated; Phys references confined to opaque interop package. | CLOSED |
| ADR-004 | Math/physics ownership | Mathematics in PhysMath; physical meaning in Phys/phys-*. | CLOSED |
| ADR-005 | Phys metadata monopoly | Assumptions, conventions, provenance, corpus status, MRC/trust belong only to Phys. | CLOSED |
| ADR-006 | External physical objects | Extension-owned types plus standard envelope; not forced into frozen Phys quantity model. | CLOSED |
| ADR-007 | Extension curation | No package-local curation; unsupported new-domain objects remain external until future Phys curation support. | CLOSED |
| ADR-008 | Interpretation ambiguity | Every physical object declares interpretation-neutral or framework-specific scope. | CLOSED |
| ADR-009 | PhysMath IR | Multi-sorted typed mathematical IR with closed versioned core sorts. | CLOSED |
| ADR-010 | IR extension | Versioned extension-sort protocol with canonicalization and replay contracts. | CLOSED |
| ADR-011 | Canonicalization | Canonical bytes are deterministic and schema-versioned. | CLOSED |
| ADR-012 | Equality | Structural, definitional, mathematical, approximate, and physical identity are distinct. | CLOSED |
| ADR-013 | Scope/binding | Scope/binding identity, not spelling, determines symbol identity. | CLOSED |
| ADR-014 | Mathematical conditions | Conditions remain mathematical structures in PhysMath. | CLOSED |
| ADR-015 | Physical assumptions | Conditions acquire physical epistemic status only in Phys formalization. | CLOSED |
| ADR-016 | Higher-order math | Functions, functionals, operators, tensors, maps, relations, etc. are separate mathematical sorts. | CLOSED |
| ADR-017 | Limits/asymptotics | Formal symbolic capability is required; implementation is bounded, not a general CAS. | CLOSED |
| ADR-018 | Replay | PhysMath trace and Phys ledger are separate replay domains. | CLOSED |
| ADR-019 | Handoff precedence | Trusted Phys state supersedes research assertions without mutating source evidence. | CLOSED |
| ADR-020 | Handoff schema | Versioned machine-readable HandoffPackage is the formalization boundary. | CLOSED |
| ADR-021 | Handoff loss | FULLY/PARTIALLY/UNREPRESENTABLE with mandatory loss accounting. | CLOSED |
| ADR-022 | Research correspondence | Project A owns correspondence proposals; no semantic promotion. | CLOSED |
| ADR-023 | Research status | Dedicated Project A status vocabulary. | CLOSED |
| ADR-024 | Failure taxonomy | Dedicated obstacle vocabulary distinct from status and corpus vocabularies. | CLOSED |
| ADR-025 | t/k separation | Mathematical parameters are distinct by identity; physical role mapping is external. | CLOSED |
| ADR-026 | TOE joint law | D[Psi,Q,C]=0 is a research obligation, never a compiler primitive. | CLOSED |
| ADR-027 | External composability | Cross-extension composition uses public contract references; cycles prohibited. | CLOSED |
| ADR-028 | AI protocol | Machine-readable schemas are authoritative over prose. | CLOSED |
| ADR-029 | Numerical boundary | Numerical Layer consumes formalized Phys/phys-* contracts only. | CLOSED |
| ADR-030 | TOE Phys witness | Acceptance requires one FULLY and one PARTIALLY/UNREPRESENTABLE path. | CLOSED |
| ADR-031 | Growth Gate | External-first routing; kernel growth requires independent foundational evidence. | CLOSED |
| ADR-032 | Resource safety | Bounded resource model and explicit RESOURCE_LIMIT status. | CLOSED |
| ADR-033 | Project A / trace linkage | Research hypotheses reference immutable MathTrace IDs/digests; no trust semantics imported. | CLOSED |
| ADR-034 | Coding-agent authority | No implementation may invent architecture; all architectural decisions are closed here. | CLOSED |

---

# 40. Consolidated adversarial-review disposition

## Review 1 — DeepSeek

Accepted and closed:

- exact project ownership;
- package graph;
- PhysMath/Phys interoperability;
- IR/sort system;
- canonicalization;
- condition/scope model;
- mathematical versus physical dimensions;
- higher-order mathematics;
- asymptotics;
- trace versus Phys ledger;
- correspondence artifacts;
- external physical object shape;
- metadata attachment;
- handoff schema;
- loss accounting;
- t/k typing;
- TOE joint-dynamics acceptance;
- AI protocol;
- numerical handoff;
- Growth Gate routing;
- resource/adversarial controls;
- Project A interface;
- minimal TOE Phys witness;
- cross-extension composability;
- research-state/trace linkage.

All are now ADR-001 through ADR-034.

## Review 2 — Gemini

Accepted:

- three-project architecture;
- rejection of Candidate Physical Object / Interpretation Layer;
- reusable mathematics in PhysMath;
- physical semantics in phys-*;
- trust monopoly in Phys;
- explicit handoff/loss accounting;
- separation of t-flow and k-flow;
- generic mathematical typing rather than Dancer/Dance/Stage ontology.

Corrected:

- PhysMath is not a monolithic dynamic runtime;
- extensibility is achieved through versioned mathematical-sort contracts rather than unrestricted runtime invention;
- physical correspondence proposals remain Project A artifacts.

## Review 3 — Qwen

Accepted:

- mathematics-only PhysMath;
- external physical semantics;
- single Phys metadata model;
- Project A owns research correspondences;
- five-cluster blocker structure.

Resolved:

- every cluster now has explicit ADR decisions and closed exit criteria;
- implementation authorization no longer depends on an unresolved blocker register.

## Review 4 — Z

Accepted and fixed:

- PhysMath/Phys structural tension;
- non-existent Phys curation path;
- matrix contradiction;
- dependency-diagram ambiguity;
- handoff precedence;
- replay-domain dependency;
- status-vocabulary collision.

Specific corrections:

```text
"mathematical trace" != "provenance"
```

and:

```text
ResearchStatus.UNREPRESENTABLE
```

is distinct from obstacle-class:

```text
ENCODING_UNREPRESENTABLE
```

No vocabulary is overloaded with two meanings.

---

# 41. Architecture-level acceptance tests

Before coding begins, architecture is accepted when the design can support these tests.

## Boundary tests

```text
PhysMath core imports no phys-* package.
phys-* imports no internal Phys implementation.
Phys imports nothing above itself.
Numerical imports no Project A internals.
```

## Ownership tests

```text
HilbertSpace -> PhysMath
WaveFunction-as-math -> PhysMath
BohmianConfiguration -> phys-bohm
Physical assumption -> Phys
Provenance -> Phys
Corpus status -> Phys
```

## Semantic tests

```text
same spelling != same identity
mathematical equality != physical identity
t != k
mathematical condition != physical assumption
wavefunction math != Bohmian ontology
```

## Handoff tests

```text
FULLY -> formalizable
PARTIALLY -> loss report required
UNREPRESENTABLE -> no silent coercion
```

## Replay tests

```text
PhysMath trace replays in PhysMath
Phys ledger replays in Phys
Phys ledger does not claim to replay arbitrary PhysMath operations
```

## TOE tests

```text
Dancer mathematics representable
Dance mathematics representable
Stage mathematics representable
cross-interface hypotheses representable
joint-dynamics candidate representable
at least one fully representable Phys fragment
at least one partial/unrepresentable fragment
```

---

# 42. Full lifecycle

The complete software/research lifecycle is:

```text
G0  Charter
        ↓
G1  Requirements
        ↓
G2  Architecture / blocker closure
        ↓
G3  Project-specific implementation plans
        ↓
G4  Implementation
        ↓
G5  Unit / contract tests
        ↓
G6  Adversarial review
        ↓
G7  TOE stress acceptance
        ↓
G8  Freeze / release
```

The three projects proceed independently within the shared architecture.

A TOE research milestone is not a release of PhysMath.

A PhysMath release is not a TOE theory result.

A phys-* release is not Phys corpus authority.

A Numerical result is not a Phys trust decision.

---

# 43. Coding-agent no-invention rule

A coding agent receiving this specification MUST NOT invent decisions about:

- layer ownership;
- dependency direction;
- physical versus mathematical ownership;
- metadata ownership;
- external-library trust;
- interpretation scope;
- IR sort categories;
- canonicalization authority;
- replay domains;
- handoff precedence;
- research status;
- numerical boundaries;
- TOE joint-dynamics semantics.

Implementation choices may be made only inside these closed architectural constraints.

Examples of implementation choices that remain open:

```text
exact Go file layout
private helper names
internal algorithm choice
serialization library
transport mechanism
test organization
performance optimization
```

provided they do not alter architectural semantics.

---

# 44. Final implementation-authorization checklist

```text
[✓] Three projects have explicit ownership.
[✓] Phys remains the frozen trusted substrate.
[✓] PhysMath is mathematics-only.
[✓] phys-* owns physical semantics.
[✓] Phys owns all canonical physical trust metadata.
[✓] Numerical Layer owns numerical computation.
[✓] No overlapping assumption/provenance/trust systems exist.
[✓] Dependency graph is acyclic.
[✓] PhysMath core is insulated from physical-domain packages.
[✓] External phys-* objects have a defined contract.
[✓] Interpretation ambiguity is explicitly represented.
[✓] PhysMath IR is multi-sorted and versioned.
[✓] Extension sorts have a canonicalization protocol.
[✓] Equality layers are separated.
[✓] Scope/binding identity is explicit.
[✓] Mathematical conditions are separated from physical assumptions.
[✓] Physical dimensions/units remain outside PhysMath.
[✓] Higher-order mathematics has explicit type boundaries.
[✓] Limits/asymptotics are formal mathematical structures.
[✓] t-flow and k-flow cannot be conflated by notation.
[✓] PhysMath traces and Phys ledgers are separate.
[✓] Handoff is machine-readable.
[✓] Handoff losses are explicit.
[✓] Phys canonical artifacts supersede research assertions for trusted meaning.
[✓] Research status is separated from failure taxonomy and corpus status.
[✓] Cross-framework correspondence is a proposal, not implicit identity.
[✓] External phys-* packages compose through explicit public contracts.
[✓] TOE joint dynamics is a research obligation, not hard-coded ontology.
[✓] AI-agent interaction is machine-readable.
[✓] Numerical computation cannot repair semantic gaps.
[✓] Growth Gate remains external-first.
[✓] Resource exhaustion is explicit.
[✓] TOE acceptance includes both successful and honest failed handoff paths.
[✓] No unresolved architectural blocker remains.
```

---

# 45. Final architectural invariant

The complete ecosystem is governed by this chain:

\[
\boxed{
\text{AI research}
\rightarrow
\text{PhysMath mathematics}
\rightarrow
\text{phys-* physical semantics}
\rightarrow
\text{Phys formalization/trust}
\rightarrow
\text{Numerical consequences}
}
\]

with the strict ownership rule:

\[
\boxed{
\begin{aligned}
\text{TOE-Agent} &\rightarrow \text{hypotheses} \\
\text{PhysMath} &\rightarrow \text{mathematics} \\
\text{phys-*} &\rightarrow \text{physics} \\
\text{Phys} &\rightarrow \text{trust metadata} \\
\text{Numerical} &\rightarrow \text{computation}
\end{aligned}}
\]

The Dancer–Dance–Stage TOE is deliberately the first severe stress test of this architecture, not its definition.

The architecture is **closed**.

**No architectural blocker remains open.**

**Implementation planning is authorized.**

The next document supplied to a coding agent should therefore be a project-specific implementation plan derived from these closed decisions, not another architecture-discovery exercise.
