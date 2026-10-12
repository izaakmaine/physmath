The revised implementation plan (`plan9.1.md`) successfully establishes a clean Go topology, enforcing a sibling workspace layout (`phys-artifact`, `phys-math`, `phys-lib`), preserving `phys-gr` byte-untouched as a reference baseline, and defining a strict core/specific promotion protocol.

However, an adversarial review reveals that **`plan9.1.md` cannot be handed to a coding agent as-is**. It explicitly leaves two architectural blockers (B1 and B2) open for external input and contains five implementation ambiguities regarding AST differentiation, binary framing, package paths, and protocol layout.

Below is the adversarial review, followed by the definitive rulings required to close all remaining blockers and ambiguities.

---

## 1. Resolution of Carried-Over Architectural Blockers

Section 16 of `plan9.1.md` lists two open blockers awaiting architectural input. Leaving these open guarantees that a coding agent will either halt execution or make non-conforming design choices during Phases 3 and 5.

### RULING: Architectural Blocker B1 — Expression Node-Set & Local Math

* **Issue**: `phys-gr` relies on trigonometric nodes (`Sin`, `Cos`) and radial functions ($A(r), B(r)$), but `phys-math` defines a closed 10-node AST (`Symbol`, `Rational`, `Add`, `Mul`, `Neg`, `Pow`, `Sqrt`, `Call`, `Relation`, `BranchSet`).


* **Definitive Decision (Option A Adopted)**:


1. `phys-math` AST remains strictly closed at 10 nodes. No trigonometric or domain-specific nodes will be added to `phys-math`.


2. Functions like $\sin(x)$, $\cos(x)$, $A(r)$, or $B(r)$ are represented in `phys-math` purely as `Call` nodes referencing standard or framework-declared `Function` artifacts (e.g., `physmath/function/1`).


3. Local trigonometric simplifications and rewrite rules live entirely within `phys-lib/gr` as framework-local algebra rules, consuming standard `phys-math` ASTs.





### RULING: Architectural Blocker B2 — Regime Non-Vanishing Placement

* **Issue**: Disagreement on whether non-vanishing conditions ($r \neq 0, A(r) \neq 0, \sin\theta \neq 0$) belong in top-level physical assumptions or per-step mathematical conditions.


* **Definitive Decision (Option A Adopted)**:


1. Non-vanishing regime conditions are declared as framework-level `PhysicalAssumption` artifacts in `phys-lib/gr`.


2. `DerivationStep` objects in `phys-math` carry explicit `MathCondition` instances **only** when mandated by a specific closed transformation rule contract (e.g., `EXPR-SIMPLIFY-001` requiring `NEQ(x,0)` for `Pow(x,0) -> 1`).





---

## 2. Adversarial Audit: Gaps & Implementation Hazards

| Audit Focus | Identified Ambiguity / Hazard in `plan9.1.md` | Required Directive for Coding Agent |
| --- | --- | --- |
| **AST Differentiation (`DIFFERENTIATE-001`)** | `DIFFERENTIATE-001` must handle `Call` nodes (e.g., $f(g(x))$) without knowing domain-specific derivatives like $\frac{d}{dx}\sin(x) = \cos(x)$.

 | Implement symbolic chain rule in `phys-math`: $\frac{\partial}{\partial x} \text{Call}(f, [u(x)]) = \text{Call}(\text{Derivative}(f, i), [u(x)]) \cdot \frac{\partial u}{\partial x}$. Substitution of analytical derivatives (e.g., $\sin' \to \cos$) is performed by expanding target `Function` bodies via `EXPR-APPLY-001` or `phys-lib/gr` rewrite rules.

 |
| **`ParameterID` Binary Framing** | Section 3.2 gives formula $\text{SHA-256}(\text{"physmath"} \parallel \text{ScopePath} \parallel \text{ParameterOrdinal})$, but does not specify primitive serialization wrappers.

 | Use `phys-artifact` primitive primitives: $\text{SHA-256}(\text{EncodeString}("physmath") \parallel \text{EncodeBytes}(ScopePath) \parallel \text{UInt32BE}(ParameterOrdinal))$. For top-level parameters, $ScopePath$ is an empty byte slice (`EncodeBytes([])` $\to$ `0x00000000`).

 |
| **Protocol Fixture Paths** | Section 1, 10, and 11 place Equivariance in `phys-lib/gr` and second-framework in `fixture2`, but exact directory trees are omitted.

 | Enforce standard nested layouts:

<br>

<br>• GR / Equivariance: `phys-lib/gr/{AGENTS.md, manifest.json, protocol/derivation_test.go}`<br>

<br>• Conformance: `phys-lib/internal/conformance/fixture2/{AGENTS.md, manifest.json, protocol/derivation_test.go}` |
| **`phys-lib/core` Package Structure** | `core/` starts empty, but Go package declarations for future promotions are unassigned.

 | `phys-lib/core` uses Go package `package core` (import path `[github.com/PithomLabs/phys-lib/core](https://github.com/PithomLabs/phys-lib/core)`). Any promoted sub-component must use a named package (e.g., `package space`, path `[github.com/PithomLabs/phys-lib/core/space](https://github.com/PithomLabs/phys-lib/core/space)`). No generic `util` or `common` packages are permitted.

 |
| **`go.work` & Legacy Isolation** | `phys-gr` keeps `replace phys => ../phys [transitional]`, creating potential module lookup conflicts during build.

 | The new modules (`phys-artifact`, `phys-math`, `phys-lib`) **must not** contain `replace` directives. `go.work` coordinates workspace resolution. `phys-gr` remains in `go.work` strictly for regression running until Phase 10.

 |

---

## 3. Concrete Architectural Directives for the Coding Agent

### 3.1 Module & Package Import Topology

The coding agent must construct the following dependency graph, enforced strictly by Go module boundaries:

```text
github.com/PithomLabs/phys-lib/gr
   │
   ├─► github.com/PithomLabs/phys-lib/core
   │
   └─► github.com/PithomLabs/phys-math
          │
          └─► github.com/PithomLabs/phys-artifact

```

* **Prohibited Imports**: `phys-artifact` importing anything; `phys-math` importing `phys-lib`; lateral imports between `phys-lib/gr` and other theory packages except via `BaseFrameworkRefs`.



### 3.2 Canonical Binary Framing (`phys-artifact`)

All identity digests must follow exact big-endian framing:

* **`String`**: `UInt32BE(len)` $\parallel$ `UTF8Bytes`
* **`Bytes`**: `UInt32BE(len)` $\parallel$ `RawBytes`
* **`Optional`**: `0x00` if None; `0x01` $\parallel$ `Payload` if Some
* **`Sequence`**: `UInt32BE(count)` $\parallel$ `Item_0` $\parallel \dots \parallel$ `Item_N`
* **`Set`**: `UInt32BE(count)` $\parallel$ deduplicated items sorted bytewise lexicographically
* **`BigInt`**: `SignByte` (`0x00` for $\ge 0$, `0x01` for $<0$) $\parallel$ `UInt32BE(MagLen)` $\parallel$ Minimal unsigned big-endian bytes (no leading zeros; $0$ is `0x00 0x00000000`).



### 3.3 Rule Execution Lock (`phys-math`)

The rule engine MUST restrict execution to the 7 closed rules using exact string keys:

1. `"physmath:EXPR-CANON-001"`
2. `"physmath:EXPR-SIMPLIFY-001"`
3. `"physmath:EXPR-SUBSTITUTE-001"`
4. `"physmath:RAT-NORMALIZE-001"`
5. `"physmath:DIFFERENTIATE-001"`
6. `"physmath:SELECT-BRANCH-001"`
7. `"physmath:EXPR-APPLY-001"`

Invoking any rule ID outside this set must fail closed with an `ErrUnknownRuleID` error.

---

## 4. Phase Execution Sequence with Closed Decisions

With B1 and B2 closed, the coding agent can execute the phase pipeline without ambiguity:

1. **Phase 0 (Workspace Wiring)**: Extend `go.work` with `./phys-artifact`, `./phys-math`, `./phys-lib`. Verify legacy `phys-gr` passes existing tests.


2. **Phase 1 (`phys-artifact`)**: Create module `[github.com/PithomLabs/phys-artifact](https://github.com/PithomLabs/phys-artifact)`. Implement primitive framing, BigInt encoding, and `ArtifactRef` digest engine. Add public surface vocabulary lint test.


3. **Phase 2 (`phys-math` Core IR)**: Implement 10-node AST, `ParameterID` generator, `BoundParameterRef`, and core durable families.


4. **Phase 3 (`phys-math` Rules & Derivation)**: Implement 7-rule executors, symbolic chain rule for `Call` nodes, `DerivationStep` validator, and hash-chain replay harness.


5. **Phase 4 (`phys-lib/core`)**: Create `[github.com/PithomLabs/phys-lib](https://github.com/PithomLabs/phys-lib)`. Leave `core/` empty.


6. **Phase 5 (`phys-lib/gr`)**: Implement `phys-lib/gr` owning the 6 physical families, GR metric assumptions, and local trig algebra.


7. **Phase 6 & 7 (Equivariance Slice & Protocol)**: Implement the 6-step Equivariance derivation replay in `phys-lib/gr/protocol/derivation_test.go` and validate against `phys-lib/gr/manifest.json`.


8. **Phase 8 (Second-Framework Conformance)**: Implement `phys-lib/internal/conformance/fixture2` asserting namespace isolation and cross-framework non-interference.


9. **Phase 9 & 10 (Vectors & Cleanup)**: Run full test matrix, clean shims, remove `phys-gr` from `go.work`, and verify zero physics vocabulary in `phys-math`.



---

Would you like to generate the complete starter code and test harness for **Phase 1 (`phys-artifact`)** to begin execution?