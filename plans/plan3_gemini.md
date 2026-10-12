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