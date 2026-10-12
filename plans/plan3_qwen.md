# Adversarial Architectural Review of `plan3.md` (PhysMath / Phys / Phys-* Ecosystem)

## Executive Verdict
`plan3.md` is a masterclass in epistemic boundary enforcement. It successfully separates the mathematical substrate (`PhysMath`), the physical semantics (`phys-*`), the trust/metadata authority (`Phys`), and the computational execution (`Numerical`). The philosophical architecture is flawless.

However, when subjected to the **Go implementation reality** and the **frozen `specs_v2_3.md` contract**, there are **five critical architectural blockers**. These are not minor nitpicks; they are structural contradictions that will cause circular dependencies, type-system failures, or serialization black holes if a coding agent attempts to implement the system as currently described. 

Below is the adversarial deconstruction of these blockers and the exact architectural patches required to make the plan truly implementation-authorized.

---

## ❌ BLOCKER 1: The Shared Type / Circular Dependency Trap (The `Expr` Ownership Crisis)
**The Contradiction:** 
*   **Section 4 & 5** mandate a strict one-way dependency: `PhysMath ─────► Phys`. `Phys` MUST NEVER depend on `PhysMath`.
*   **The Frozen Reality (`specs_v2_3.md` §11.2):** The `Phys` trust metadata plane defines `Assumption` with a closed union value: `ExprValue(core.Expr)`. 
*   **The Blocker:** If `core.Expr` is owned by `PhysMath`, then `Phys` *must* import `PhysMath` to define its `Assumption` type. This violates the "Phys never depends on PhysMath" rule. If `Phys` owns `core.Expr`, then `PhysMath` is no longer a pure math library; it is forced to use `Phys`'s expression tree, violating "PhysMath owns mathematical objects."

**Required Architectural Patch:**
You must extract the **pure mathematical IR** (specifically `Expr`, `Dimension`, `Rational`, and `Symbol`) into a **zero-dependency shared base module** (e.g., `math-core` or `phys-math-ir`). 
*   `math-core` has NO physical semantics, NO assumptions, NO provenance. It is pure symbolic math.
*   `PhysMath` depends on `math-core` and builds higher-order math (Tensors, Functionals, Manifolds) on top of it.
*   `Phys` depends on `math-core` to embed `Expr` inside `Assumption.ExprValue`.
*   This perfectly preserves the rule: `Phys` never depends on `PhysMath`, but both share the exact same mathematical IR.

---

## ❌ BLOCKER 2: The "Unrepresentable" Handoff Black Hole
**The Contradiction:**
*   **Section 27** mandates that the formalization handoff must classify every component as `FULLY_REPRESENTABLE`, `PARTIALLY_REPRESENTABLE`, or `UNREPRESENTABLE`.
*   **Section 34** mandates that the AI-agent interface must use "machine-readable schemas" and that "prose is documentation only."
*   **The Blocker:** If a mathematical object is `UNREPRESENTABLE` in `Phys` (e.g., a highly complex, unformalized tensor network from `PhysMath`), how is it serialized in the `HandoffPackage`? If it's serialized as a string or prose, it violates Section 34. If it's serialized as a `PhysMath` trace, `Phys` cannot validate it. The plan provides **zero architectural mechanism** for crossing the boundary with an `UNREPRESENTABLE` object without breaking the machine-readability rule.

**Required Architectural Patch:**
The `HandoffPackage` schema must explicitly define an `OpaqueMathPayload` structure for `UNREPRESENTABLE` items:
```go
type OpaqueMathPayload struct {
    MathTraceDigest string // SHA-256 of the PhysMath trace
    SerializedIR    []byte // Canonical JSON of the PhysMath IR (machine-readable, but physically meaningless to Phys)
    LossReport      []LossRecord // Explicit declaration of what physical semantics are missing
}
```
`Phys` must accept this payload, store it, and verify its hash against the `MathTraceDigest`, but it must be **strictly forbidden** from attempting to interpret, validate, or promote the `SerializedIR`. It remains a "quarantined" mathematical artifact attached to a physical hypothesis.

---

## ❌ BLOCKER 3: Type-System Enforcement of Interpretation Ambiguity
**The Contradiction:**
*   **Section 11** states: "Interpretation-specific physics is permitted. Silent interpretation is prohibited." It gives the example of `WaveFunction` (math) vs `BohmianWaveFunction` (physics).
*   **The Blocker:** The plan relies on *naming conventions* and *AI discipline* ("The exact Go names are implementation choices; the semantic separation is mandatory"). In Go, if a function in `phys-bohm` accepts a `math.WaveFunction` as an argument, the compiler will allow it. The "silent interpretation" will happen at runtime, completely bypassing the architectural boundary. "Mandatory" semantic separation cannot be enforced by documentation; it must be enforced by the Go type system.

**Required Architectural Patch:**
The plan must mandate **Strict Encapsulation via Go Types**. 
*   Physical packages (`phys-*`) MUST define their own nominal physical types (e.g., `phys-bohm.State`).
*   These physical types MUST wrap the `PhysMath` types in unexported fields.
*   **Crucial Rule:** No function in a `phys-*` package may accept a raw `PhysMath` type as an input parameter. It must accept the physical wrapper. 
*   To extract the math, the physical type must expose an explicit, auditable method: `func (s State) MathematicalView() math.WaveFunction`. 
This makes "silent interpretation" a compile-time error, perfectly aligning the code with the constitution.

---

## ❌ BLOCKER 4: The Replay Domain Delegation Mechanism
**The Contradiction:**
*   **Section 21** establishes two distinct replay domains: `PhysMath` replays math; `Phys` replays physical trust/ledger. "A Phys session MUST NOT claim to replay an arbitrary PhysMath trace."
*   **The Blocker:** When a `HandoffPackage` is formalized into `Phys`, the `Phys` ledger records the derivation. But the actual mathematical transformations (e.g., functional derivatives, tensor contractions) happened in `PhysMath`. If `Phys` cannot replay the math, how does it verify that the physical metadata (assumptions, provenance) correctly attaches to the mathematical result? The plan lacks the exact mechanism for `Phys` to validate the handoff without violating the "two replay domains" rule.

**Required Architectural Patch:**
The `Phys` ledger must introduce a `MathStepReference` record type. 
When `Phys` ingests a formalization, it does *not* re-execute the `PhysMath` operations. Instead, it records:
1.  The `MathTraceDigest` (proving the math was executed and is immutable).
2.  The exact `PhysMath` Object IDs that serve as inputs/outputs to the physical interpretation.
3.  The physical metadata (Assumptions, Conventions) attached at that specific node.
`Phys.Validate()` will verify the hash chain of the *physical metadata attachments* and verify that the `MathTraceDigest` matches the one stored in the `HandoffPackage`. It explicitly outsources the mathematical validity to the `PhysMath` replay domain.

---

## ❌ BLOCKER 5: The Frozen `Kind` Enum vs. External Physical Objects
**The Contradiction:**
*   **Section 9** states: "External physical objects are not forced into the frozen Phys `Kind`/`Dimension` model."
*   **The Frozen Reality (`specs_v2_3.md` §6):** The `Phys` MVP has a hardcoded, closed `Kind` enum (18 values: `Mass`, `Energy`, `Expression`, etc.) and a hardcoded `Expr` node set. 
*   **The Blocker:** If `phys-gr` defines a `MetricTensor`, it cannot be represented as a `core.Object` because `KindMetricTensor` does not exist in the frozen kernel, and `Expr` has no `Tensor` node. If `phys-gr` tries to pass a `MetricTensor` to `Phys` to attach an assumption (e.g., `metric.signature = -+++`), `Phys` will reject it because it doesn't match the frozen `Kind` enum. The plan says "External libraries remain external," but provides no schema for how they interact with `Phys`'s metadata monopoly.

**Required Architectural Patch:**
The plan must explicitly mandate the **`PhysicalArtifactEnvelope`** (briefly mentioned in Section 9) as the *only* bridge between `phys-*` and `Phys`.
*   `Phys` must be extended (via a predefined v0.5 Growth Gate or an explicit MVP envelope mechanism) to accept an `Envelope` object.
*   The `Envelope` contains: `ExtensionID`, `ObjectType`, `ObjectID`, `MathRefs`, and `PhysMetadataRefs`.
*   `Phys` tracks the `PhysMetadataRefs` (Assumptions, Provenance, Corpus Status) on the `Envelope`, but treats the actual payload (`ObjectType/ObjectID`) as an **opaque identifier**. 
*   This allows `phys-gr` to attach physical trust metadata to a `MetricTensor` without forcing the frozen `Phys` kernel to understand what a `MetricTensor` is. The frozen 18 `Kind`s remain strictly for the base MVP corpus; everything else flows through the `Envelope`.

---

## Final Handoff Status

`plan3.md` is 95% complete and conceptually brilliant. However, it cannot be handed to a coding agent until these 5 blockers are resolved in the text. 

**Required Addendum to `plan3.md` before implementation authorization:**
1.  **Resolve `Expr` ownership:** Extract pure math IR to a shared `math-core` base to prevent `Phys` from importing `PhysMath`.
2.  **Define `OpaqueMathPayload`:** Explicitly schema the `UNREPRESENTABLE` handoff to preserve machine-readability without forcing physical interpretation.
3.  **Mandate Go Type Encapsulation:** Forbid raw `PhysMath` types in `phys-*` function signatures to make "silent interpretation" a compile-time error.
4.  **Define `MathStepReference`:** Explicitly codify how `Phys` validates physical metadata attachments by hashing `PhysMath` digests, rather than replaying the math.
5.  **Mandate the `PhysicalArtifactEnvelope`:** Explicitly define the opaque wrapper that allows external `phys-*` objects to receive `Phys` trust metadata without violating the frozen `Kind` enum.

Once these 5 structural patches are integrated, the architectural blockers drop to zero, and the plan is fully authorized for implementation.