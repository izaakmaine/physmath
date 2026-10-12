# Adversarial Review of `plan4.md` (Architecture v1.1)

## Executive Verdict
**v1.1 is a massive architectural leap forward.** The decision to enforce `PhysMath ⟂ Phys` (orthogonality) and replace the "stack" model with an "artifact reference" model is exactly what was needed to prevent semantic bleed. Treating the TOE as a consumer/workload rather than an ontology is the correct Reality-First application.

However, when subjected to the strict reality of **Go implementation mechanics** and the **frozen Phys MVP constraints**, v1.1 leaves **four critical architectural blockers** unresolved. If handed to a coding agent as-is, the agent will be forced to invent unauthorized structures to bridge these gaps. 

Below is the adversarial deconstruction of these 4 blockers and the exact architectural locks required to make the document 100% implementation-ready.

---

## ❌ BLOCKER 1: The In-Memory vs. Serialized Reference Ambiguity
**The Flaw:** v1.1 states that artifacts are connected by "explicit references" and that the exact identity mechanism "may be content-addressed". It blurs the line between *in-memory Go composition* and *serialized artifact IDs*.
**The Implementation Trap:** If `PhysMath` and `Phys` are orthogonal, they cannot share a base `Artifact` interface. If the coding agent tries to pass "Artifact IDs" around in memory to compose a `phys-gr.Metric`, `Phys` will have to act like a database client to resolve those IDs, violating its nature as a pure in-memory trusted substrate.
**The Architectural Lock:**
We must explicitly separate **in-memory composition** from **serialized identity**.
*   **In-Memory:** `phys-*` packages own the composition. A `phys-gr` package imports both `PhysMath` and `Phys` and defines concrete Go structs (e.g., `type PhysicalMetric struct { MathExpr physmath.Expr; Trust phys.Assumption }`). 
*   **Serialized/Graph:** Artifact IDs (content hashes or UUIDs) are *only* used when serializing to JSON for the TOE-Agent's research graph or for cross-process handoff. 
*   **Rule:** No package passes opaque IDs in memory to avoid importing another package's types. Go interfaces and structs handle in-memory composition; IDs handle serialization.

## ❌ BLOCKER 2: The Formalization Protocol's "ACCEPTED" State
**The Flaw:** v1.1 defines a formalization protocol where a `FormalizationProposal` goes to `Phys` and can result in `ACCEPTED`, `REJECTED`, etc.
**The Implementation Trap:** The frozen `Phys` MVP (specs v2.3) **explicitly forbids runtime corpus promotion** (REQ-002-12). `Phys` cannot "ACCEPT" a new formalization into its trusted corpus at runtime. Corpus status is human-curated via `manifest.json`. If the coding agent builds an `ACCEPTED` state machine inside `Phys`, it violates the frozen MVP.
**The Architectural Lock:**
The formalization protocol's terminal state in the MVP must be explicitly pinned to the **Provisional/Research boundary**.
*   **MVP Termination:** The protocol terminates when `Phys` successfully validates the proposal and seals it as a `ResearchCandidate` (Provisional, `HYPOTHESIS` status). 
*   **Offline Promotion:** The `ACCEPTED` state is **not a runtime code state**. It is an offline, human-driven action that results in updating the `manifest.json` files and rebuilding the binary. 
*   **Rule:** `Phys` runtime APIs only produce `UnverifiedResearchCandidate` or `ResearchCandidate`. There is no `AcceptFormalization()` API in the MVP.

## ❌ BLOCKER 3: The Mathematical Precondition Vacuum
**The Flaw:** v1.1 states: *"PhysMath MUST NOT own a persistent assumption, convention, provenance, corpus-status, MRC, or other trust-authority metadata layer."*
**The Implementation Trap:** `PhysMath` is a symbolic engine. It *must* track mathematical preconditions (e.g., `x != 0` for division, `det(g) != 0` for matrix inversion). If `PhysMath` is forbidden from having an "assumption layer", how does it track `x != 0` without either (a) silently failing, or (b) secretly importing `Phys.Assumption` (violating orthogonality)?
**The Architectural Lock:**
We must explicitly define a **zero-overhead mathematical condition type** that is strictly distinct from epistemic trust metadata.
*   **`PhysMath.Condition`:** A purely structural, lightweight mathematical predicate (e.g., an `Expr` relation or a simple boolean flag). It has **no** provenance, **no** corpus status, **no** trust semantics, and **no** human curation. It exists solely to make symbolic transformations mathematically valid.
*   **`Phys.Assumption`:** The epistemic, trust-bearing metadata that wraps or interprets the condition for physical formalization.
*   **Rule:** `PhysMath` owns `Condition` (mathematical necessity). `Phys` owns `Assumption` (physical trust). They are structurally distinct types, preventing semantic bleed.

## ❌ BLOCKER 4: The `phys-interop` "MAY" Trap
**The Flaw:** v1.1 introduces `phys-interop` as a "MAY" for shared infrastructure (serialization, ID generation).
**The Implementation Trap:** "MAY" in an architecture document is a magnet for scope creep. A coding agent will inevitably use `phys-interop` to dump shared types, interfaces, and composition logic, accidentally recreating the exact semantic bleed v1.1 was designed to prevent.
**The Architectural Lock:**
For the MVP, `phys-interop` must be **explicitly deferred or strictly bounded**.
*   **MVP Decision:** Defer `phys-interop` to v0.5. 
*   **MVP Execution:** `phys-*` packages handle their own in-memory composition (via direct imports). Serialization is handled by standard `encoding/json` with explicit struct tags within each package. 
*   **Rule:** No shared interop package exists in the MVP. If a shared utility is absolutely required later, it is strictly limited to stateless serialization codecs and ID generation, with a hard ban on owning any semantic types.

---

## 🛠️ The Final Addendum for `plan4.md`

To make this document 100% bulletproof for the coding agent, append this exact section to the end of `plan4.md`:

```markdown
49.1 Implementation Locks (v1.1 Hardening)
The following architectural decisions are locked to prevent coding-agent hallucination:

LOCK 1: In-Memory vs. Serialized Composition
- In-memory composition is owned by `phys-*` packages via standard Go structs 
  (e.g., `phys-gr` imports `PhysMath` and `Phys` to build `PhysicalMetric`).
- Artifact IDs (content hashes) are strictly for serialization, JSON handoff, 
  and TOE-Agent graph storage. 
- No package passes opaque IDs in memory to avoid type imports.

LOCK 2: Formalization Protocol Terminal State
- The `Phys` runtime API terminates at `ResearchCandidate` (Provisional/HYPOTHESIS).
- There is NO runtime `AcceptFormalization()` API. 
- The `ACCEPTED` corpus status is an offline, human-driven action that updates 
  `manifest.json` and requires a binary rebuild.

LOCK 3: Mathematical Precondition vs. Physical Assumption
- `PhysMath` owns `MathCondition`: a zero-overhead, purely structural predicate 
  (e.g., `x != 0`) with NO provenance, NO trust, and NO corpus status.
- `Phys` owns `Assumption`: the epistemic, trust-bearing metadata.
- These are strictly distinct types. `PhysMath` never imports `Phys.Assumption`.

LOCK 4: No `phys-interop` in MVP
- The `phys-interop` package is explicitly DEFERRED to v0.5.
- MVP composition is handled by direct imports in `phys-*` packages.
- MVP serialization is handled by standard `encoding/json` within each package.
```

### Final Verdict
With these 4 locks applied, **`plan4.md` (v1.1) is architecturally complete.** 
It successfully enforces the Reality-First constitution, maintains the strict orthogonality of `PhysMath` and `Phys`, provides a clear path for `phys-*` extensions, and correctly positions the TOE-Agent as a consumer rather than an ontology. 

**You are cleared to hand this off to the implementation planning agent.**