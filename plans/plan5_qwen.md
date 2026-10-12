# Adversarial Architectural Review of `plan5.md` (v1.1)

## Executive Verdict
The conceptual separation of Mathematics (PhysMath), Physical Semantics (`phys-*`), and Trust/Authority (Phys) is philosophically sound and strictly necessary. However, when subjected to the **mechanical realities of Go module dependencies, closed-type enums, and cross-boundary data flow**, the document contains **five fatal architectural blockers**. 

If handed to a coding agent as-is, the agent will either violate the "no dependency" rule, attempt to invent duplicate metadata systems, or fail to implement the formalization handoff because the projection mechanism is undefined.

Below is the adversarial deconstruction of these blockers and the exact architectural patches required to make the document unambiguous and implementation-ready.

---

### ❌ BLOCKER 1: The Dependency / Interop Paradox (Section 5.1 vs 5.2)
**The Flaw:** Section 5.1 mandates that the PhysMath mathematical core MUST have no dependency on Phys. Section 5.2 then states that interaction occurs in an `interop / adapter` package that holds `PhysArtifactRef`. 
**The Go Reality:** If the `interop` package lives inside the PhysMath module and references Phys types, the entire PhysMath module now depends on Phys. This violates the core constitutional rule. If it doesn't reference Phys types, it cannot hold `PhysArtifactRef`.
**The Fix:** The `interop` code **cannot** live in the PhysMath module. You must choose one of two mechanical solutions:
1. **The Bridge Module:** The `interop` code lives in a separate, third Go module (e.g., `github.com/PithomLabs/phys-bridge`) that depends on *both* PhysMath and Phys.
2. **The API Split:** Phys must be split into `phys-api` (a zero-dependency module containing only `ArtifactRef` interfaces and schema IDs) and `phys-core` (the trusted substrate). PhysMath can safely depend on `phys-api` without importing any physical semantics. 
*Directive for coding agent:* The `interop` package is explicitly banned from the PhysMath repository.

### ❌ BLOCKER 2: The Metadata Monopoly vs. Closed Enums (Section 11, 15, 17)
**The Flaw:** The plan states `phys-*` MUST NOT create duplicate trust metadata, and Phys owns all assumptions. However, the frozen Phys MVP has a **closed** `AssumptionKind` enum (exactly 7 values) and a closed `Kind` enum (18 values). A `phys-gr` package cannot represent a `Metric` or a `SchwarzschildExterior` regime using Phys's closed enums without modifying Phys. If it uses its own enums, it violates the "no duplicate metadata" rule. The coding agent is trapped.
**The Fix:** The architecture must explicitly define the **External Metadata Contract**:
1. `phys-*` packages **MUST** use Phys's exact `AssumptionKind` enums. Domain-specific regimes must be encoded as string keys/values using the existing kinds (e.g., `Kind: Domain, Key: "regime", Value: "schwarzschild_exterior"`).
2. For physical objects that don't fit the frozen `Kind` enum (like `Tensor` or `Metric`), they **cannot** be minted as `core.Object`s. They remain strictly un-typed in Phys, represented only as opaque `ArtifactRef`s or projected as collections of scalar `Expression`s.
*Directive for coding agent:* `phys-*` packages are forbidden from defining new `AssumptionKind` or `Kind` enums. They must map domain semantics to the frozen Phys string keys.

### ❌ BLOCKER 3: The Formalization Projection Gap (Section 31, 33)
**The Flaw:** The plan describes a "Formalization Protocol" where a `FormalizationProposal` is sent to Phys. But it never defines *how* a rich `phys-*` object (like a `Tensor` or `GRExpr`) is actually projected into the frozen Phys substrate. The coding agent writing `phys-gr` will not know how to implement the "handoff" because Phys only accepts `core.Expr` and `core.Object`.
**The Fix:** The architecture must mandate a **Projection Interface**. Every `phys-*` object that claims to be `FULLY_REPRESENTABLE` or `PARTIALLY_REPRESENTABLE` must implement a deterministic lowering mechanism.
```go
type Projectable interface {
    ProjectToPhys() (phys.Artifact, LossReport)
}
```
The `LossReport` must explicitly state what mathematical/physical structure was discarded (e.g., "Tensor covariance lost; projected as 16 scalar Expressions"). 
*Directive for coding agent:* Formalization is not a conceptual handoff; it requires a concrete, code-level `ProjectToPhys()` method on external artifacts.

### ❌ BLOCKER 4: The Cross-Module Replay Illusion (Section 28, 29, 48)
**The Flaw:** The plan states "Replay is authority-local" and "There is no fixed number of replay domains." But the TOE-Agent's entire purpose is to compose multi-module hypothesis chains (e.g., using PhysMath for a limit, then `phys-gr` for a metric, then Phys for a dimension check). If replay is strictly local, the TOE-Agent cannot produce a single, coherent, replayable derivation ledger for a cross-domain hypothesis. 
**The Fix:** The plan must explicitly define the TOE-Agent's **Research Ledger** as a *meta-ledger*. It does not replay the internal math/physics steps; it replays the *sequence of artifact references and formalization proposals*.
```go
type ResearchStep struct {
    Operation    string
    InputRefs    []ArtifactRef
    OutputRef    ArtifactRef
    ModuleOrigin string // "PhysMath", "phys-gr", "Phys"
}
```
*Directive for coding agent:* The TOE-Agent builds a sequential list of `ArtifactRef` transitions. It is explicitly forbidden from attempting to replay the internal operations of PhysMath or `phys-*`.

### ❌ BLOCKER 5: The "Mathematical Condition" vs "Physical Assumption" Enforcement (Section 14, 26)
**The Flaw:** The plan distinguishes between a mathematical condition (`r > 2M` in PhysMath) and a physical assumption (exterior region in `phys-gr`). But it doesn't specify how the boundary is enforced in code. If `phys-gr` imports a `MathCondition` from PhysMath, how does it "attach" a physical interpretation without creating a duplicate assumption system?
**The Fix:** The plan must define the **PhysicalInterpretation** struct (hinted at in Section 12) as the *only* bridge. `phys-*` packages never mutate or duplicate the `MathCondition`; they only create a read-only `PhysicalInterpretation` artifact that links the math reference to a Phys-trusted assumption key.
```go
type PhysicalInterpretation struct {
    MathConditionRef physmath.ArtifactID
    PhysAssumptionID phys.AssumptionID
    Scope            string
}
```
*Directive for coding agent:* `phys-*` packages must never create a new assumption. They must create a `PhysicalInterpretation` that maps an existing `MathCondition` to an existing `PhysAssumptionID`.

---

### Final Handoff Status
The conceptual architecture is correct, but the **mechanical boundaries are porous**. 

To make this document unambiguous for a coding agent, you must inject the following **Implementation Locks** into Section 48 (Coding-agent completion criterion):

1. `[LOCK]` The `interop` package is banned from the PhysMath module; it must live in a separate bridge module or rely on a zero-physics `phys-api` module.
2. `[LOCK]` `phys-*` packages are forbidden from defining new `AssumptionKind` or `Kind` enums; they must map domain semantics to frozen Phys string keys.
3. `[LOCK]` Formalization requires a concrete `ProjectToPhys() (phys.Artifact, LossReport)` interface; conceptual handoffs are rejected.
4. `[LOCK]` The TOE-Agent's Research Ledger is a meta-ledger of `ArtifactRef` transitions; it must not attempt to replay internal module operations.
5. `[LOCK]` Physical interpretations of mathematical conditions must be implemented via a read-only `PhysicalInterpretation` struct, never by duplicating assumption metadata.

With these 5 locks applied, the architecture transitions from a philosophical blueprint to a **mechanically enforceable Go contract**.