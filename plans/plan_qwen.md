# Adversarial Architectural Review: `plan.md` (PhysMath v0.4)

**Context:** Reviewed against the v8.0 Reality-First Constitution (`background.md`), the Dancer/Dance/Stage TOE Blueprint and its mathematical critique (`TOE_gpt.md`), and the strict separation of exploratory math (`PhysMath`), trusted physics (`Phys`), and numerical execution (`Program B`).

**Verdict:** `plan.md` successfully establishes the epistemic boundaries (no truth adjudication, no numerical execution, no automatic promotion). However, when subjected to the **TOE stress test**, the architecture reveals **five fatal structural voids**. These are not minor nitpicks; they are architectural blockers that will prevent the AI agent from even *formulating* the TOE hypotheses, let alone testing them. 

If implemented as written, the system will collapse under the weight of its own strict separation between "pure math" and "trusted physics," leaving the AI with no formal language to explore the interfaces between them.

Here are the five architectural blockers that must be resolved before implementation.

---

### BLOCKER 1: The "Handoff Void" (No Formal Bridge Between Math and Physics)
**The Flaw:** `plan.md` Section 11 strictly bifurcates the world: mathematical conditions (e.g., $x \neq 0$) belong to `PhysMath`; physical assumptions (e.g., `rest_mass_nonnegative`) belong to `Phys`. Section 15 states that when a candidate is ready, it is "handed to Phys together with the explicit physical mappings." 
**Why it breaks the TOE:** The TOE is entirely about *proposing* physical mappings between mathematical structures (e.g., "The CDT path integral maps to the AS effective action in the continuum limit"). `plan.md` provides **zero architectural representation** for this mapping. 
If `PhysMath` cannot hold physical semantics, and `Phys` only accepts already-formed trusted objects, the "handoff" is a black box. The AI agent has no formal way to construct a "Candidate Physical Object" or an "Interpretation Tag" that says, *"This mathematical structure is being proposed as this physical structure, subject to these physical assumptions."* 
**Resolution Required:** We must define a formal **Interpretation/Bridge Layer** (or a "Candidate Physical Object" type) that lives *between* `PhysMath` and `Phys`. It must allow the AI to attach provisional physical semantics, regime tags, and assumption mappings to pure math objects, strictly quarantined from the trusted `Phys` corpus until human review.

### BLOCKER 2: The TOE Sector Organization Problem (Untyped Mathematical Soup)
**The Flaw:** `plan.md` Section 1 explicitly bans the Dancer/Dance/Stage metaphor as compiler ontology. It demands that `PhysMath` be a purely mathematical space (expressions, functions, tensors, manifolds) without physical domain tagging.
**Why it breaks the TOE:** The TOE stress test requires the AI to manipulate three distinct sectors: Stage (CST/CDT geometry), Dancer (Bohmian QFT states), and Dance (RG flow/guidance). If `PhysMath` is purely untyped mathematical soup, the AI has **no structural guardrails** to prevent silent category errors during exploration. For example, the AI might accidentally contract a Stage tensor index with a Dancer Hilbert space index, or apply a Dance RG flow operator to a Stage metric, because `PhysMath` only sees "tensors" and "operators."
**Resolution Required:** `PhysMath` must implement a **Mathematical Domain/Sector Tagging System**. This is not a physical ontology; it is a *mathematical classification* (e.g., `ManifoldBundle`, `HilbertSpace`, `FlowEquation`) that organizes the TOE workload. The AI must be forced to declare which mathematical domain an object belongs to, preventing cross-sector contamination during the exploratory phase, long before it reaches the trusted `Phys` layer.

### BLOCKER 3: The Functional Calculus / Engine Location Contradiction
**The Flaw:** `plan.md` Section 4.2 states: "Wavefunction as a mathematical function -> PhysMath. Bohmian wavefunction as physically operative object -> phys-bohm (Phys extension)." 
**Why it breaks the TOE:** The TOE requires the AI to perform heavy symbolic functional calculus (e.g., taking the functional derivative of the wavefunctional $\Psi[g, \phi]$ to derive the Bohmian guidance equation $\dot{Q} = \nabla S$). 
If this calculus engine lives in `PhysMath`, it operates on pure math functions and cannot enforce physical typing (it won't know it's taking a derivative with respect to a "Stage" metric vs a "Dancer" field). If the calculus engine lives in `Phys` (to enforce physical typing), `Phys` becomes a full Computer Algebra System (CAS), violating its mandate to be a bounded, trusted, replayable substrate.
**Resolution Required:** We must architect a **Typed Symbolic Engine**. The engine must live in `PhysMath` (to allow exploratory calculus), but it must accept **Domain-Tagged Mathematical Objects** (from Blocker 2). The engine performs the calculus purely mathematically, but the type system ensures that the *inputs and outputs* respect the mathematical domain boundaries (e.g., `FunctionalDerivative(HilbertSpaceFunctional, ManifoldField) -> TangentBundle`), preventing the AI from silently mixing the TOE sectors.

### BLOCKER 4: Asymptotic and Limit Representation (The Scaling Void)
**The Flaw:** `plan.md` Section 8.1 lists "approximation/asymptotic operations" as a required capability, but Section 5.1 strictly forbids numerical evaluation, and Section 11 forbids physical assumptions. 
**Why it breaks the TOE:** The entire TOE hinges on scaling limits and universality: "In the limit $a \to 0$, the CDT effective action flows to the AS fixed point," or "The spectral dimension $D_S(\sigma)$ drops from 4 to 2." `plan.md` provides **zero architectural detail** on how formal limits, Big-O notation, or scaling regimes are represented in `PhysMath`. If the AI cannot formally express asymptotic regimes without smuggling in physical assumptions or resorting to numerical evaluation, it cannot even *write down* the core claims of the TOE.
**Resolution Required:** `PhysMath` must implement a **Formal Asymptotic Ring**. This allows the AI to represent objects like $O(\epsilon^2)$, formal power series, and limit operations ($\lim_{x \to 0}$) as exact, symbolic mathematical structures. The system must explicitly track the "expansion parameter" and "order of truncation" as pure mathematical metadata, entirely divorced from physical regime assumptions.

### BLOCKER 5: Cross-Framework Duality and Conjectural Correspondence
**The Flaw:** `plan.md` Section 17 states: "PhysMath MUST support mathematical work that is reusable across multiple physical frameworks... PhysMath MUST NOT infer compatibility from same notation, same mathematical shape..."
**Why it breaks the TOE:** The `TOE_gpt.md` critique explicitly warned against using mathematical coincidences (like the Index Theorem or spectral dimension matching) to *force* physical equivalences between CST and CDT. The AI needs a way to propose a **Conjectural Duality** or **Correspondence Map** between two mathematically distinct structures (e.g., a causal poset and a simplicial complex). `plan.md`'s strict rule against "inferring compatibility" leaves the AI with no formal vocabulary to even *propose* a duality. It can only state mathematical isomorphisms (which are false here) or physical identities (which are untrusted here).
**Resolution Required:** We must introduce a **Correspondence/Duality Artifact** in `PhysMath`. This artifact explicitly links two mathematically distinct structures via a proposed mapping function, tagged with a "Conjectural Status." It allows the AI to say, *"I propose this mathematical map between Structure A and Structure B,"* without the system silently assuming they are isomorphic or physically identical. This artifact then becomes the exact target for the adversarial null tests (e.g., "Does this map preserve the causal structure?").

---

### Summary of Required Architectural Patches

To survive the TOE stress test, `plan.md` must be amended to include:
1.  **The Bridge Layer:** A formal "Candidate Physical Object" type that attaches provisional physical semantics to `PhysMath` objects for handoff to `Phys`.
2.  **Domain Tagging:** A mathematical sector classification system in `PhysMath` to prevent cross-contamination of Stage/Dancer/Dance mathematics during exploration.
3.  **Typed Symbolic Engine:** A calculus engine in `PhysMath` that respects the Domain Tags to enforce structural boundaries without becoming a CAS.
4.  **Formal Asymptotic Ring:** Exact symbolic representation of limits, scaling parameters, and truncation orders.
5.  **Correspondence Artifacts:** A formal vocabulary for proposing conjectural dualities between mathematically distinct structures.

**Final Adversarial Directive:** 
`plan.md` has successfully built the walls between Math, Physics, and Numerics. But it has forgotten to build the doors. The AI agent cannot explore the TOE interfaces if it is not allowed to formally construct the bridges, the limits, and the conjectural mappings. Resolve these five blockers, and the architecture will be ready for implementation.