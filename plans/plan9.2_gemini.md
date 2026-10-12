`plan9.2.md` reaches architectural closure on topology, module boundaries, rule-set locking, and blocker resolutions (B1/B2, D1–D17, R1–R6). However, an adversarial code-level audit reveals **five residual implementation hazards** where ambiguity in binary framing, AST enum tagging, rule evaluation payloads, and Go package compilation will cause an automated coding agent to stall, make divergent choices, or fail mechanical CI gates.

---

## 1. Residual Implementation Hazards & Directives

### Hazard 1: Unpinned AST Node Tag Bytes (Phase 2)

* **Issue**: Section 8 (Phase 2) defines a "10-node AST (tags `0x01`–`0x0A`)", but `plan9.2.md` does not explicitly map tag bytes to specific node types in the text. An agent assigning `0x01` to `Symbol` versus `Rational` will break binary `ArtifactRef` reproducibility across runs.


* **Directive**: The agent must bind AST tags to big-endian byte values:
* `0x01`: `Symbol`
* `0x02`: `Rational`
* `0x03`: `Add`
* `0x04`: `Mul`
* `0x05`: `Neg`
* `0x06`: `Pow`
* `0x07`: `Sqrt`
* `0x08`: `Call`
* `0x09`: `Relation`
* `0x0A`: `BranchSet`



### Hazard 2: BigInt Zero Magnitude Byte Length (Phase 1, Spec §4.6)

* **Issue**: Section 8 (Phase 1) specifies `§4.6 BigInt` primitive framing. When representing zero ($0$), integer sign is non-negative (`0x00`), but magnitude length can be ambiguous ($0$ bytes vs. $1$ byte of `0x00`).


* **Directive**: Zero ($0$) MUST be encoded strictly as `SignByte = 0x00` followed by `UInt32BE(0)` (`0x00000000`) with zero magnitude payload bytes. No leading `0x00` bytes are permitted in any non-zero magnitude payload.

### Hazard 3: Missing Lambda Bodies for Equivariance Steps (Phase 7 & §10)

* **Issue**: Phase 7 instructs the agent to run six steps (`APPLY, APPLY, CANON, APPLY, APPLY, CANON`) to reach $2 \cdot g \cdot x$ on both sides. However, without explicit expression definitions for $f$, $\rho_X$, and $\rho_Y$, `EXPR-APPLY-001` cannot execute deterministically.


* **Directive**: The agent must instantiate the standard test definitions:
* $f(v) \to 2 \cdot v$
* $\rho_X(g, x) \to g \cdot x$
* $\rho_Y(g, v) \to g \cdot v$
* `LHS_0` = $\text{Call}(f, [\text{Call}(\rho_X, [g, x])])$
* `RHS_0` = $\text{Call}(\rho_Y, [g, \text{Call}(f, [x])])$



### Hazard 4: Empty `phys-lib/core` Package Compilation Failure (Phase 4)

* **Issue**: Phase 4 initializes `phys-lib/core` as an empty package. Running `go test ./...` or `go build ./...` on a Go package directory containing no `.go` files causes the Go toolchain to abort with `build constraints exclude all Go files` or `no Go files in...`.


* **Directive**: The agent must commit a minimal `doc.go` file inside `phys-lib/core/` containing only `package core` and package documentation to ensure workspace-wide `go test ./...` commands pass during Phase 4.



### Hazard 5: Built-in `GRExpr` Function Keys for Projection (D9 & Phase 5)

* **Issue**: Decision D9 mandates explicit fallible projection between disjoint `GRExpr` (which contains `Sin`/`Cos`) and `phys-math.Expression` (where trig functions are represented as `Call` nodes). The exact target string or identifier for built-in trig functions during `ToPhysMath()` projection is unstated.


* **Directive**: During projection, `GRExpr` nodes map to `phys-math.Call` nodes using package-qualified function identity keys `"phys-gr:func/sin"` and `"phys-gr:func/cos"`. Reverse projection `FromPhysMath()` must fail with `ErrUnrepresentableProjection` if a `Call` node uses an unrecognized target key.



---

## 2. Agent Execution Matrix

| Phase | Core Objective | Primary Deliverable | Required Validation Gate |
| --- | --- | --- | --- |
| **Phase 0** | Workspace & Hash Locks | `go.work` extended; `phys-gr` & `phys` checksummed

 | Immutability oracle verified

 |
| **Phase 1** | `phys-artifact` Codec | Primitive encoders (BigInt zero byte rule applied)

 | `go test ./...` green; zero imports

 |
| **Phase 2** | `phys-math` Core IR | 10-node AST (explicit `0x01`–`0x0A` tag mapping)

 | Family golden vectors pass

 |
| **Phase 3** | `phys-math` Rule Engine | 7 closed rules; pair-based `RuleRef` locking

 | Rule-ID closure + condition tests

 |
| **Phase 4** | `phys-lib/core` Scaffold | `phys-lib/core/doc.go` scaffolding

 | Empty-core assertion green

 |
| **Phase 5** | `phys-lib/gr` Framework | Six framework families; trig projection bridge

 | Framework negatives & ported vectors pass

 |
| **Phase 6** | Protocol Fixture | Embedded nested GR protocol fixture

 | Strict schema decode pass

 |
| **Phase 7** | Equivariance Slice | 6-step derivation replay yielding $2 \cdot g \cdot x$<br> | Golden `ArtifactRef` equality verified

 |
| **Phase 8** | `fixture2` Conformance | `phys-fixture2` isolated minimal framework

 | Cross-namespace non-interference pass

 |
| **Phase 9–10** | Suite & Migration | Clean `go.work`; zero-import verification

 | Gate 10 check; `phys-gr` removed from `go.work`<br> |

---

## 3. Execution Clearance

`plan9.2.md` is architecturally closed. Armed with the five concrete directives above, the coding agent can execute Phase 0 through Phase 10 without making design assumptions or encountering toolchain stalls.