## 

what are the underlying architectural and structural problems about the relationship of Phys, PhysMath and Phys-* external libraries? what would you do differently to resolve them in a more cohesive coherent way?

## qwen

Based on a deep adversarial review of the provided architectural documents (Plan 3, plan2, background, and the various compiler specs), the current proposed relationship between **PhysMath**, **Phys**, and the **Phys-\*** external libraries suffers from **four fundamental architectural pathologies**. 

While the *intent* of the separation is correct (preventing mathematical seduction and enforcing the Reality-First constitution), the *mechanisms* proposed (like the `HandoffPackage`, `PhysicalArtifactEnvelope`, and the `PhysMath -> Phys` dependency) create severe friction, circular risks, and over-engineered "交接" (handoff) illusions.

Here is the diagnosis of the underlying structural problems, followed by a radically simplified, more cohesive architectural resolution.

---

### Part 1: The 4 Architectural Pathologies

#### 1. The Dependency Inversion Paradox (PhysMath $\to$ Phys)
**The Problem:** Plan 3 explicitly states that `PhysMath` depends on `Phys` at the project level, but also mandates that `PhysMath` must remain "mathematics only" and never own physical assumptions. 
**The Reality:** If `PhysMath` imports `Phys`, the semantic boundary is already breached. A coding agent will inevitably use `Phys.Assumption` or `Phys.Dimension` inside `PhysMath` "just for convenience," silently turning mathematical conditions into physical metadata. Furthermore, `Phys` (the trust kernel) *needs* `PhysMath` (for expression trees, exact rationals, symbolic calculus). A bidirectional or upward dependency violates Go's strict acyclic package rules and creates a monolithic trap.

#### 2. The "Handoff" Illusion
**The Problem:** The architecture spends immense effort designing a `HandoffPackage` and classifying exports as `FULLY_REPRESENTABLE` vs `UNREPRESENTABLE` to bridge `PhysMath` and `Phys`.
**The Reality:** **AI agents do not work this way.** An AI theorist does not derive a pure mathematical theorem in a vacuum and then "hand it off" to a physics layer. The AI reasons *within a physical context*. When the AI is doing General Relativity, it is using `phys-gr`. The idea that `PhysMath` exists as a separate, un-physical sandbox that must later be "formalized" into `Phys` is an artificial bureaucratic hurdle that doesn't match the cognitive workflow of theoretical physics.

#### 3. Metadata Monopoly vs. Domain Flexibility
**The Problem:** `Phys` is declared the sole monopoly owner of `Assumption`, `Convention`, and `Provenance`. 
**The Reality:** How does `phys-gr` express the convention `metric.signature = -+++`? How does `phys-qm` express the `CommutationRelation`? If `Phys` must pre-define every possible physical convention and assumption type, `Phys` becomes a bloated encyclopedia of physics, violating the "frozen kernel" rule. If `phys-gr` creates its own `GRConvention` type, it breaks the Metadata Monopoly, leading to fragmented trust ledgers.

#### 4. Ontological Homelessness (The Wavefunction Problem)
**The Problem:** Where does `WaveFunction` live? 
**The Reality:** 
* In `PhysMath`, it is just a mathematical function $f: \mathbb{R}^3 \to \mathbb{C}$.
* In `phys-qm`, it is a physical state vector in a Hilbert space.
* In `phys-bohm`, it is the ontological pilot-wave generating the guidance equation.
The current architecture struggles to prevent `PhysMath` from accidentally treating a `WaveFunction` as a physical object, or `phys-bohm` from silently redefining the mathematical rules of calculus.

---

### Part 2: The Cohesive Resolution (The "Contract & Wrapper" Architecture)

To resolve these issues, we must abandon the "Handoff" paradigm and the "PhysMath $\to$ Phys" dependency. Instead, we adopt a strict **Bottom-Up Dependency Graph** combined with a **Semantic Wrapper Pattern**.

#### The New Golden Rule of Dependencies
```text
PhysMath (Pure Math)  <-- Phys (Trust & Metadata Interfaces) <-- Phys-* (Domain Physics)
```
*   **PhysMath** has **ZERO** knowledge of `Phys` or `Phys-*`. It only knows math (Expressions, Tensors, Manifolds, Exact Rationals).
*   **Phys** depends on **PhysMath** (to use its expression trees for dimensional analysis and symbolic logic). `Phys` defines the *Interfaces* for trust (Metadata, MRC rules, Dimensional bases) but contains almost no specific physics equations.
*   **Phys-\*** depends on **BOTH**. It uses `PhysMath` for the math, and `Phys` for the trust/metadata.

#### Resolution 1: Eradicate the "Handoff" via Semantic Wrappers
Instead of a `HandoffPackage`, we use **Semantic Wrappers with Physical Contracts**.
A mathematical object only becomes a physical object when a `Phys-*` library wraps it and attaches a `Phys` metadata contract.

```go
// IN PhysMath (Pure Math, No Physics)
type MathTensor struct { Rank int; Components []Expr; /* ... */ }

// IN Phys (The Trust Interface)
type Convention struct { Key string; Value string }
type PhysicalAssumption struct { Kind string; Expr Expr }

// IN phys-gr (The Domain Physics)
type MetricTensor struct {
    math    PhysMath.MathTensor      // The mathematical engine
    contract Phys.Contract           // The physical trust metadata
}

// The Contract explicitly binds the math to physical reality
func NewMinkowskiMetric() MetricTensor {
    return MetricTensor{
        math: buildDiagonalTensor(-1, 1, 1, 1),
        contract: Phys.NewContract(
            Phys.Kind("SpacetimeMetric"),
            Phys.Convention("signature", "-+++"),
            Phys.Assumption("LorentzSymmetry", true),
        ),
    }
}
```
**Why this works:** There is no "handoff." The AI uses `MetricTensor` directly. The compiler enforces that `MetricTensor` cannot be passed to a pure `PhysMath` function without stripping its contract, and a pure `MathTensor` cannot be used in a physical equation without a `phys-*` wrapper providing the contract.

#### Resolution 2: Mathematical Conditions vs. Physical Assumptions
We resolve the Metadata Monopoly conflict by strictly separating **Mathematical Preconditions** (owned by `PhysMath` / generic logic) from **Physical Assumptions** (owned by `Phys` / `Phys-*`).

*   **MathPrecondition (Logical/Mathematical):** "The denominator $r \neq 0$". "The matrix is invertible ($\det \neq 0$)". This is handled by `PhysMath`'s symbolic engine. It does not require `Phys` metadata.
*   **PhysicalAssumption (Ontological/Empirical):** "Rest mass $m \ge 0$". "The speed of light $c > 0$". "We are in the non-relativistic regime ($v \ll c$)". This is handled by `Phys` and attached via the `Phys-*` wrapper.

When `phys-sr` derives $E=mc^2$, the symbolic engine uses the *MathPrecondition* ($c \neq 0$) to simplify fractions, but the *PhysicalAssumption* ($m \ge 0$) is used by the `SelectBranch` operation to choose the positive energy root. The MRC (Math Reality Check) verifies that physical assumptions are never silently dropped.

#### Resolution 3: The "Dancer/Dance/Stage" as Workload Labels, Not Compiler Types
To prevent the TOE metaphor from corrupting the compiler, we enforce that **Dancer, Dance, and Stage are strictly Project A (TOE-Agent) concepts**, not `Phys` or `PhysMath` types.

*   `PhysMath` sees: `HilbertSpace`, `Functional`, `Manifold`, `Poset`.
*   `Phys-*` sees: `QuantumState`, `GuidanceLaw`, `LorentzianMetric`, `CausalOrder`.
*   `TOE-Agent` sees: `Dancer`, `Dance`, `Stage`.

If the AI tries to write `PhysMath.Dancer`, the compiler throws a `CategoryError`. The AI must map its TOE intent to the correct physical library (`phys-bohm` or `phys-qm`).

#### Resolution 4: A Minimal, Interface-Driven `Phys` Core
`Phys` does not need to contain the Standard Model or General Relativity. `Phys` only contains:
1.  **The MRC Engine:** Rules for dimensional consistency, kind compatibility, and provenance contamination.
2.  **The Metadata Schema:** Definitions of `Assumption`, `Convention`, `Provenance`, `CorpusStatus`.
3.  **The Base SI Dimensions:** $M, L, T, I, \Theta, N, J$.
4.  **The Ledger/Session:** The hash-chained replay mechanism.

Everything else (Newton's laws, Lorentz factors, Einstein tensors) lives in `phys-mechanics`, `phys-sr`, `phys-gr`. `Phys` is just the constitution; `Phys-*` are the laws.

---

### Part 3: How the AI Workflow Actually Looks Now

Let's trace the AI deriving the Newtonian limit of a metric, using the new cohesive architecture.

1.  **AI Intent (Project A / TOE-Agent):** "I want to test if this causal set reduces to a classical metric."
2.  **Context Selection:** The AI loads `phys-cst` and `phys-gr`.
3.  **Mathematical Operation (PhysMath):** The AI uses `PhysMath.Calculus` to perform a Taylor expansion of a symbolic expression. *No physical metadata is involved yet.*
4.  **Physical Binding (Phys-gr):** The AI wraps the result in a `phys-gr.WeakFieldMetric` struct. This struct automatically attaches the `PhysicalAssumption("WeakField", eps << 1)` via the `Phys` contract interface.
5.  **MRC Enforcement (Phys):** The AI attempts to add this `WeakFieldMetric` to a `QuantumState` from `phys-qm`. 
    *   *Compiler Check:* `Phys.MRC` checks the `Kind` and `Dimension`.
    *   *Result:* `CategoryMismatchError`. A metric (Stage) cannot be added to a state (Dancer).
6.  **Ledger Recording (Phys):** The valid derivation steps are committed to the `Session` ledger, recording the exact `PhysMath` operations and the `Phys-gr` contracts used.
7.  **Handoff to Reality:** The AI outputs a `ResearchCandidate` artifact containing the derivation hash, the physical contracts invoked, and the mathematical preconditions required.

### Summary of the Architectural Shift

| Feature | Old Plan (Plan 3 / v0.5) | New Cohesive Resolution |
| :--- | :--- | :--- |
| **Dependency** | `PhysMath` $\to$ `Phys` (Risk of leakage) | `PhysMath` $\leftarrow$ `Phys` $\leftarrow$ `Phys-*` (Strict acyclic) |
| **Bridging** | Complex `HandoffPackage` & `Envelope` | **Semantic Wrappers** with `Phys.Contract` |
| **Metadata** | Monopolized by `Phys`, causing bloat | `Phys` owns the *Schema*; `Phys-*` owns the *Instances* |
| **Conditions** | Blurred between Math and Physics | Strict split: `MathPrecondition` vs `PhysicalAssumption` |
| **AI Workflow** | Derive in Math $\to$ Handoff to Physics | Derive *within* a Physics Context using Math engines |
| **TOE Metaphor** | Risk of becoming compiler types | Strictly confined to Project A (Agent vocabulary) |

**The Ultimate Takeaway:** 
You do not need a "translation layer" or a "handoff protocol" if you design the system so that **mathematics is the engine, physics is the steering wheel, and trust is the chassis.** By using the Wrapper Pattern, the AI never has to "translate" math into physics; it simply drives the math engine using the physics steering wheel from the very first line of code. The compiler enforces that the chassis (Trust/MRC) is never bypassed.



## deepseek

## The underlying structural problems

The recurring blockers in `plan3.md` (B-002, B-011, B-012, B-014, N-1, N-2, N-4, N-7) are not independent defects. They are symptoms of five deeper problems in how the relationship between Phys, PhysMath, and `phys-*` is being modeled.

### P1. The relationship is drawn as a stack, but it is actually a reference DAG

Every version of the plan draws `PhysMath → Phys`, `phys-* → Phys`, `phys-* → PhysMath`, `TOE-Agent → …`. Arrows flow upward. This framing forces the architecture to answer questions like:

- “Which layer owns provenance?”
- “Where is the metadata attached?”
- “How does a lower layer reference an upper layer?”

These questions would not arise if the relationship were modeled as **content-addressed artifacts that reference each other by digest**. Mathematics does not *depend* on physics; a physical interpretation *references* a mathematical structure; a trust assertion *references* a physical artifact. Three orthogonal concerns, not three layers.

The `physref/*` isolation package in §5, the `PhysicalArtifactEnvelope` in §9, and the `phys_metadata_refs[]` field in §26 all exist *because* the stack model creates leakage problems that the DAG model would not create in the first place.

### P2. Phys is being asked to serve two incompatible roles

Phys currently does two very different things:

1. It provides a **formal substrate** — `Expr`, `Kind`, `Dimension`, `MintObject`, canonical JSON, the operation set.
2. It provides a **trust authority** — provenance, corpus status, MRC, session ledger, candidate containment.

These are conceptually independent. Role (1) is one formal IR among possible formal IRs; it happens to be quantity-oriented. Role (2) is a general capability that could annotate *any* formal artifact, regardless of IR.

The plan inherits Phys's frozen design and therefore inherits the conflation. This is why §29–§30 have to accept that “much of the TOE will exceed the frozen Phys object model,” and why the “TOE Phys witness” has to be manufactured as a special acceptance criterion. If Phys's trust model were separated from Phys's formal substrate, the TOE could be fully trust-annotated without requiring its physics to fit Phys's quantity model.

### P3. The envelope pattern conflates identity, reference, and trust

`PhysicalArtifactEnvelope` in §9 bundles four distinct concerns into a single artifact:

- **identity** (`extension_id`, `object_id`, `object_type`),
- **mathematical references** (`math_refs[]`),
- **physical-contract references** (`physical_contract_refs[]`),
- **trust references** (`phys_metadata_refs[]`),
- plus a `payload`.

This is why N-4 (cross-extension payload interpretation) is unavoidable: if the envelope is the only interop shape, the payload must be introspectable, which contradicts the plan's own extension-ownership principle. If the payload is opaque, the envelope is doing no work beyond identifying the object.

The three concerns should be separated:

- An artifact *is* its payload, identified by content digest and namespace.
- A **reference** is a `(registry, digest)` tuple.
- A **trust annotation** is a separate record that references an artifact digest.

Under this model, an envelope is unnecessary; a physical artifact's cross-references are just fields of the artifact itself, and its trust annotation is a sidecar.

### P4. The math-condition / physical-assumption split is type-based, but the underlying distinction is provenance-based

§7 declares that `x > 0` is a mathematical condition in PhysMath, while `rest_mass_nonnegative` is a physical assumption in Phys. But in practice these are often the *same linguistic artifact*: `r > 2M` is both a mathematical inequality and a physical horizon condition.

The real distinction is not the type of the artifact. It is:

- **who asserted it**,
- **for what framework**,
- **in what scope**,
- **with what epistemic standing**.

These are all metadata attached to *an occurrence* of the artifact, not properties of its type. Making them into different types forces a translation step (`requested_phys_metadata[]`, the condition→assumption mapping in N-7) that the underlying semantics do not require.

### P5. The trust model is binary, but the epistemic reality is graded

PhysMath = “untrusted.” `phys-*` = “external unless formalized.” Phys = “trusted.” This binary view of trust is why the external-curation question (B-011, ADR-007) keeps returning: there is no middle ground where an external framework is *conditionally trustworthy for a stated scope*.

Phys's own corpus vocabulary (`ESTABLISHED`, `CONTESTED`, `SUPERSEDED`, `FALSIFIED`) already anticipates this. The problem is that the layer model collapses it: either you are inside the Phys trust boundary or you are outside.

### P6. Handoff is modeled as an artifact, but it is actually a protocol

§26 specifies a `HandoffPackage` as the boundary. But a real handoff involves:

- Project A proposing a fragment,
- Phys refusing due to unrepresentability,
- `phys-*` requiring additional contracts,
- Project A revising,
- Phys accepting.

This is a *negotiation*, not a transfer. Modeling it as a static artifact forces loss accounting (§27) to do work that a protocol would do naturally: the “loss report” is really “the sequence of refusals and revisions.”

---

## What I would do differently

The coherent alternative is to stop drawing the relationship as a stack and start drawing it as **four content-addressed registries that reference each other by digest**.

### 1. Replace layers with orthogonal concerns

There are four orthogonal artifact dimensions:

| Dimension | Artifact kind | Owner |
|---|---|---|
| Mathematical structure | Typed, canonical, exact | Whoever creates it |
| Physical interpretation | “This math structure represents that physical thing” | An interpretation framework |
| Trust assertion | “This artifact has this corpus/provenance standing” | A curation authority |
| Research state | “These artifacts are being investigated together” | A research process |

These are not layers. They are *dimensions along which any artifact may carry metadata*. A single physical artifact might carry:

- a mathematical payload,
- an interpretation scope,
- a trust annotation,
- a research provenance chain.

Each dimension is owned by a different authority, but they compose because they reference a *shared content digest* of the artifact.

### 2. Content addressing as the universal interop mechanism

Every artifact has:

```
RegistryNamespace + ContentDigest
```

That pair is the only required common structure. Any two components can interoperate by agreeing on this pair. There is no `Envelope`, no `physref/*`, no `math-core / math-domain` partition required for isolation — because PhysMath's artifacts and Phys's artifacts are all identified the same way.

Cross-references become `(namespace, digest)` tuples. Trust annotations are records of the form:

```
TrustAssertion {
    subject: (namespace, digest),
    provenance: DERIVED,
    corpus_status: CONTESTED,
    ...
}
```

The `subject` field makes no assumption about what *kind* of artifact it annotates. PhysMath can be annotated; a `phys-bohm` object can be annotated; a Project A hypothesis can be annotated. No layer dominance is implied.

### 3. Promote trust to a standalone concern; demote Phys's formal substrate

Phys-as-written conflates trust with a quantity-oriented IR. Split them:

- **PhysTrust** (new or refactored): a small, stable, theory-neutral trust model. It knows only `TrustAssertion` and the ledger that stores them. It has no `Kind`, no `Dimension`, no operation set.
- **Phys-Quantity-Formalism** (existing): Phys's current formal substrate. It becomes *one* framework that produces content-addressed artifacts, exactly like PhysMath.

Under this view, Phys's existing `mechanics/` and `relativity/` packages are not “the trusted base”; they are *one interpretation framework* whose artifacts happen to be trusted by curation. This is a reclassification, not a rewrite.

The benefit: the TOE workload no longer needs to force its rich physics through Phys's quantity model. It can produce its artifacts in PhysMath, annotate them with `phys-*` interpretations, and attach `PhysTrust` assertions by digest.

### 4. Make `phys-*` the interpretation registry, not an extension layer

A `phys-bohm` artifact should look like:

```
BohmianConfiguration {
    // payload is interpretation-specific
    math_refs: [(math, digest1), (math, digest2)],
    // optional trust annotations resolve against PhysTrust
    trust_refs: [(trust, digest3)],
}
```

The `math_refs` are just digests into PhysMath. The `trust_refs` are digests into PhysTrust. Neither is embedded; both are references. No envelope. No metadata attachment API. No duplicate provenance system — because the extension never holds a provenance; it holds a *reference to a trust assertion*.

### 5. Treat conditions as content-addressed; treat interpretation as annotation

A condition `r > 2M` is a PhysMath artifact. Its *interpretation* is a separate artifact:

```
PhysicalInterpretation {
    math_ref: (math, digest_condition),
    framework: "gr",
    meaning: "horizon condition for Schwarzschild mass M",
    scope: "static, exterior region",
}
```

The same condition can have multiple interpretations in different frameworks — GR, alternative gravity, numerics — without any of them claiming to *be* the condition. This dissolves N-7 (condition→assumption mapping) and simplifies §7 considerably.

### 6. Model handoff as a protocol, not an artifact

Replace `HandoffPackage` with a small protocol:

```
Proposal → Review(Phys) → {Accept, Reject(reason), RequestRevision(requirements)}
       → Revision → Review → …
       → AcceptedFragment
```

The durable artifact is `AcceptedFragment` — a tuple of `(math_refs, interpretation_refs, trust_assertions)`. Loss accounting becomes the log of rejected proposals, which is inherently complete because rejections are typed responses.

### 7. Specify the IR by composition rules, not sort names

`plan3.md` §12 lists sort names. What an IR specification actually needs to provide is:

- For each sort, its **domain and codomain**.
- Its **composition rules** — which sorts apply to which.
- Its **canonical form**.
- Its **equality class**.

`Function`, `Functional`, `Operator` are not peers; they are distinguished by *composition behavior*. Specifying them as sort names in a list leaves the semantics to the implementer, which is exactly what §43 forbids.

A minimal IR spec would answer: “Given `f : A → B` and `x : A`, is `f(x) : B` legal?” Without that, the IR is a vocabulary, not a type system.

---

## Why this resolves the recurring blockers

| Blockers | What dissolves them |
|---|---|
| B-002 `PhysMath → Phys` dependency | No project-level dependency. Both produce content-addressed artifacts. |
| B-011 external object contract | Envelope replaced by content-addressed artifact + reference tuple. |
| B-012 metadata attachment | Trust assertions attach by digest, not by API. |
| B-013/B-014 handoff + loss | Handoff is a protocol; loss is the typed refusal log. |
| N-1 math-core/math-domain partition | No partition needed; the isolation is provided by content addressing. |
| N-2 `physref/*` opacity | Opaque reference is just `(namespace, digest)`. |
| N-4 cross-extension payload | Payload is never introspected; composition is via references. |
| N-7 condition→assumption mapping | Interpretation is a separate artifact, not a promoted condition. |

The `UNREPRESENTABLE` target (N-6) also becomes local: an artifact is either resolvable by a trust registry or it is not. Unrepresentability in Phys *specifically* is a fact about one curation authority, not a property of the architecture.

---

## What this preserves

The three-project split is directionally right. The alternative above keeps it:

- **PhysMath** remains the mathematics registry.
- **phys-*** remains the interpretation registry.
- **Phys** is split into a trust authority (PhysTrust) and a formal framework (existing Phys), which resolves the conflation.
- **Project A** remains the research process, referencing artifacts across registries.
- **Numerical Layer** consumes only artifacts that resolve to trust-annotated interpretations.

The change is that these are no longer drawn as a hierarchy. They are drawn as **siblings that reference each other by digest**. The relationship is a DAG of artifacts, not a stack of layers.

That is the underlying architectural problem. Every recurring blocker in `plan2.md` and `plan3.md` is a symptom of trying to enforce a stack where a DAG belongs.


## gemini

The core architectural defect in the relationship between `Phys`, `PhysMath`, and `Phys-*` is that it attempts to solve **lifecycle and operational concerns using static package dependency rules**. By forcing hard domain isolations (e.g., `PhysMath` cannot import `phys-*`, `Phys` is completely frozen, and `phys-*` cannot hold metadata), the design creates severe structural deadlocks where real-world research data, mathematical transforms, and numerical execution have nowhere to intersect.

---

### Part 1: The Underlying Structural Problems

#### 1. The Type/Sort Extension Trap (Strict Isolation vs. Expressivity)

`PhysMath` treats mathematical sorts as a closed, core-controlled taxonomy, forcing external domain libraries (`phys-*`) to plug in via opaque "extension sorts."

* **The Structural Problem:** True theoretical physics requires domain-specific symbolic manipulation (e.g., non-commutative Clifford algebras, path integral measures, p-adic differential operators, dummy tensor index contractions).
* **The Consequence:** If `PhysMath` cannot evaluate or inspect these nodes without calling external code, its core AST transformation engine (canonicalization, symbolic differentiation, substitution, De Bruijn index binding) collapses into either dynamic-dispatch hell or opaque black-box blobs that strip `PhysMath` of its symbolic power.

#### 2. The Metadata Vacuum & Rigid Trust Binary

The architecture assumes physical metadata exists in only two states: non-existent or immutably formal in `Phys`. `Phys` is frozen, while `phys-*` is explicitly forbidden from implementing metadata systems.

* **The Structural Problem:** Active AI research (`TOE-Agent`) operates perpetually in a third state: **hypothetical, un-curated, and candidate metadata** (e.g., tentative gauge fixings, provisional lattice cutoffs, trial physical parameters).
* **The Consequence:** Candidate assumptions are left completely orphaned with no schema container prior to formal handoff, forcing agents to hack metadata into ad-hoc string annotations or external untracked state.

#### 3. The Lowering Void (Boundary Paradox)

`PhysMath` manages abstract syntax trees (ASTs). `Phys` manages physical metadata and scalar ledgers. `Numerical` executes performance code (differential equation solvers, tensor networks).

* **The Structural Problem:** There is no explicit lowering specification to translate symbolic ASTs bound to physical metadata into machine-executable numerical code.
* **The Consequence:** The system expects `Numerical` to consume formalized computational contracts, but nobody owns the compilation step that converts symbolic math ($\frac{dQ}{dt} = v(Q,t)$) into vectorized JIT function pointers or memory-mapped arrays.

#### 4. Split Authority Over Canonicalization & Physical Identity

In `PhysMath`, equality and content hashing are purely syntactic (alpha-equivalence, De Bruijn indexing). In `Phys` / `phys-*`, equality is semantic and physical (gauge invariance, symmetry group equivalence, dimensional analysis).

* **The Structural Problem:** Because `PhysMath` cannot know about physical rules, two expressions that are physically identical under a gauge transformation or parameter substitution will produce completely different `PhysMath` canonical content hashes.
* **The Consequence:** Deduplication and cache lookups in Project A fail because syntactic uniqueness does not equal physical uniqueness.

#### 5. The One-Shot Binary Handoff Fallacy

Crossing from dynamic exploration (`TOE-Agent`) into formal substrate (`Phys`) via `HandoffPackage` is treated as a single binary event.

* **The Structural Problem:** Real formalization is an iterative negotiation loop. A research hypothesis is rarely 100% representable on the first attempt.
* **The Consequence:** Without a transactional staging environment, handoffs either crash atomically on minor unrepresentable terms or pollute `Phys` with incomplete formalizations.

---

### Part 2: The Cohesive Architectural Redesign

To resolve these problems cohesively, the system should be re-architected around four foundational patterns: an **Open Algebraic Trait System**, a **Stratified IR Compiler Pipeline**, a **Scoped Context Hierarchy**, and a **Staged Formalization Protocol**.

```text
 ┌────────────────────────────────────────────────────────────────────────┐
 │                      TOE-Agent (Research Layer)                        │
 │           Drafts hypotheses, manages candidate context, runs search    │
 └──────────────────────────────────┬─────────────────────────────────────┘
                                    │ Emits Staged Hypotheses
                                    ▼
 ┌────────────────────────────────────────────────────────────────────────┐
 │                  Phys-Compiler & Intermediate Rep (IR)                 │
 │                                                                        │
 │  1. Symbolic IR (PhysMath): Pure AST + Algebraic Traits (De Bruijn)    │
 │  2. Semantic IR (phys-*):   Binds Units, Gauges, & Domain Ops         │
 │  3. Executable IR (Lowering): Vectorized Kernels for Numerical Engine  │
 └──────────────────────────────────┬─────────────────────────────────────┘
                                    │
            ┌───────────────────────┴───────────────────────┐
            ▼                                               ▼
 ┌─────────────────────────────┐                 ┌─────────────────────────────┐
 │    Phys (Scoped Ledger)     │                 │   Numerical Execution Engine│
 │ Staged Contexts -> Frozen   │                 │   Consumes Compiled Exec-IR   │
 └─────────────────────────────┘                 └─────────────────────────────┘

```

#### 1. Replace Enums with an Open Algebraic Trait System

Instead of hardcoding closed core sorts in `PhysMath` and treating external types as opaque extensions, structure `PhysMath` around **Algebraic Traits** (similar to Typeclasses in Haskell or Traits in Rust/Go interfaces).

* `PhysMath` defines the structural AST core and fundamental algebraic traits:
* `Trait: CommutativeGroup`, `Trait: VectorSpace`, `Trait: TensorAlgebra`, `Trait: Differentiable`


* `phys-*` packages implement these traits for their specific mathematical domain types (e.g., `phys-cst` implements `TensorAlgebra` and `Differentiable` on Clifford basis elements).
* **Result:** `PhysMath` canonicalization, substitution, and differentiation engines operate over generic trait behaviors rather than concrete type enums. External math retains full symbolic capabilities without requiring `PhysMath` to import domain packages.

#### 2. Reframe the Stack as a Stratified IR Compiler Pipeline

Treat the transition from pure math to physical theory to numerical code as a multi-stage compilation pipeline:

1. **Symbolic IR (`PhysMath`):** Math-pure AST nodes using De Bruijn indexing for variable binding. Unaware of physical units or hardware bounds.
2. **Semantic Physical IR (`phys-*`):** Attaches domain context to Symbolic IR. Performs dimensional analysis, gauge checking, and symmetry reductions.
3. **Executable IR (Lowering Bridge):** Lowers Semantic IR into typed, memory-contiguous computational graphs (e.g., C/LLVM function pointers, bytecodes, or CPU/GPU kernel wrappers).

* **Result:** Eliminates the boundary paradox. `phys-*` explicitly acts as the intermediate compiler that transforms `PhysMath` symbolic equations into high-performance `Numerical` executables.

#### 3. Replace the Frozen Substrate with a Scoped Context Hierarchy

Eliminate the rigid binary where `Phys` is frozen and `phys-*` has no metadata. Introduce a **Hierarchical Context System**:

```go
type ScopedContext struct {
    Parent    *ScopedContext        // Pointer to parent context (Root is Immutable Phys)
    Trust     TrustLevel            // PROVISIONAL, VERIFIED, CANONICAL
    Conventions map[Symbol]Convention
    Assumptions map[ID]Assumption
}

```

* **Root Context (`Phys` Core):** The immutable, curated ground truth.
* **Child Context (`TOE-Agent` Session):** Inherits all root rules from `Phys`, but allows local mutations, provisional assumptions, and un-curated parameters during active research.
* **Formalization:** Promoting a hypothesis to `Phys` simply collapses and merges a valid `Child Context` into the `Root Context` after automated validation passes.

#### 4. Define an Explicit Staged Formalization Protocol

Replace the binary `HandoffPackage` dump with a stateful formalization lifecycle:

* **Stage 1 (Provisional Intake):** Handoff creates a `StagingArea` session in `Phys`.
* **Stage 2 (Loss Analysis):** The ingestion engine generates a precise `RepresentationReport`:
* `FULLY_REPRESENTABLE`: Automatically mapped to canonical schema.
* `PARTIALLY_REPRESENTABLE`: Mapped with explicit loss tags.
* `UNREPRESENTABLE`: Isolated in a quarantined research log.


* **Stage 3 (Refinement Loop):** The AI research agent receives the `RepresentationReport`, iterates on the lossy components, and re-submits until zero structural loss is achieved.
* **Stage 4 (Commit):** Atomic write to the immutable `Phys` ledger.

#### 5. Unified `AuditManifest` as First-Class Artifact

Every research output exports a single, self-contained `AuditManifest` that deterministically links all layers together:

```json
{
  "audit_version": "2.0",
  "hypothesis_id": "hyp_bohm_invariant_082",
  "symbolic_hash": "sha256:a4f8...",
  "physical_context_hash": "sha256:c9e1...",
  "executable_kernel_id": "kernel_cuda_v12",
  "trace_digest": "math_trace_9942",
  "phys_session_id": "sess_canonical_2026_09"
}

```

This ensures any external verifier or agent can replay the symbolic derivation in `PhysMath`, inspect the context in `Phys`, re-compile via `phys-*`, and execute the solver in `Numerical` with absolute deterministic reproducibility.




