## REVIEW: PhysMath Architectural Requirements v0.4

**Context reviewed:** `plan.md` against `background.md`, `TOE_gpt.md`, `AGENTS.md`, `README.md`.

**Verdict:**  
**NOT READY FOR IMPLEMENTATION PLAN.**  
The architecture is directionally correct, but it is still a design-review draft. By its own rule in §26, no implementation plan is authorized while blockers remain `OPEN`. All `B-001` through `B-034` are still `OPEN`. More importantly, several architectural blockers are not yet in the register at all.

```text
REVIEW: PhysMath Architectural Requirements v0.4

Constitutional boundary: PASS as written
Layer ownership intent: PASS
Enforcement mechanisms: FAIL / OPEN
Mathematical-object ownership: OPEN (B-024)
Physical-object extension boundary: OPEN (B-025, B-028)
Interpretation-ambiguity rule: OPEN (B-026)
Paper-notation semantic boundary: PASS
Math IR/type model: FAIL / OPEN (B-001..B-010)
Exactness boundary: PASS as intent; OPEN in operation model (B-009, B-010)
Mathematical-condition model: OPEN (B-006, B-034)
Phys metadata ownership: PASS as intent; OPEN in attachment API (B-032, B-033)
Mathematical execution trace: OPEN (B-011, B-014)
Cross-framework model: OPEN (B-012, B-013)
External Phys libraries: OPEN (B-025, B-028, B-030)
Phys bridge: OPEN (B-015, B-029)
Formalization handoff: OPEN (B-029)
Numerical handoff: OPEN (B-016)
AI interface: OPEN (B-017)
Testing architecture: OPEN (B-020)
Lifecycle gates: PASS as process; G7 not reached

FINAL STATUS: NOT READY
```

---

## Architectural blockers that must close before implementation planning

### 1. The plan’s own blocker register already blocks G7

`plan.md` §26 says:

> No implementation plan is authorized while a blocker marked `OPEN` remains unresolved unless explicitly deferred with containment.

Every blocker `B-001` through `B-034` is `OPEN`. Therefore the implementation-plan handoff checklist in §31 cannot be signed off. This is not a minor documentation issue; it is the primary architectural gate.

The critical unresolved clusters are:

- **Core math IR:** `B-001`, `B-002`, `B-003`, `B-004`, `B-005`
- **Math/physics boundary:** `B-006`, `B-007`, `B-024`, `B-034`
- **Tensor/geometry/Stage:** `B-008`
- **Exactness/asymptotics/RG:** `B-009`, `B-010`
- **Trace/replay/failure:** `B-011`, `B-014`
- **Cross-framework/interfaces:** `B-012`, `B-013`
- **Phys bridge/handoff:** `B-015`, `B-029`, `B-031`, `B-033`
- **External `phys-*` libraries:** `B-025`, `B-028`, `B-030`, `B-032`
- **Numerical handoff:** `B-016`
- **AI interface:** `B-017`
- **Resource/security/testing:** `B-018`, `B-020`, `B-022`, `B-023`

Until these are closed, a coding agent would have to invent architecture.

---

### 2. Additional blocker: PhysMath core vs bridge dependency is unresolved

`plan.md` §1 and §25 say:

- `PhysMath` depends on the public `Phys` API.
- `PhysMath` imports `Phys` only through its public API.
- `PhysMath` must contain mathematical objects only and must not own physical metadata.

This is architecturally ambiguous. If `PhysMath` core imports `Phys`, then physical types, metadata, or trust concepts can leak into mathematical object identity. If it does not, then the dependency statement is wrong.

**Required decision:**  
Either:

- `PhysMath` core is independent of `Phys`, and only a separate bridge package depends on both; or
- `PhysMath` core may import only a narrow metadata-reference interface, with tests proving no physical semantics enter mathematical objects.

**Exit condition:** acyclic package graph + import tests + no `Phys` physical types in `PhysMath` core.

---

### 3. Additional blocker: TOE joint-dynamics acceptance contract is missing

`TOE_gpt.md` identifies the real missing heart of the TOE:

\[
\mathcal D[\Psi,Q,\mathcal C]=0
\]

and the need to separate `t`-flow from `k`-flow.

`plan.md` correctly treats Dancer–Dance–Stage as a workload, not ontology. But it does **not** define an acceptance contract for representing the missing joint dynamics across three independently owned `phys-*` extensions.

`B-013` covers cross-framework relationships, but it does not require a closed joint dynamics object. Without that, PhysMath could pass the TOE “stress test” by representing three disconnected sectors.

**Required decision:**  
Define how PhysMath represents a candidate joint dynamics over \((\Psi,Q,\mathcal C)\) without hardcoding TOE semantics.

**Exit condition:** formal acceptance test that can express the coupling, source/sink terms, and compatibility between physical `t`-flow and RG `k`-flow.

---

### 4. Additional blocker: `t`-flow vs `k`-flow is mentioned but not gated

`plan.md` §12 says PhysMath must support distinct evolution parameters. But there is no blocker for it.

`TOE_gpt.md` explicitly warns:

\[
\partial_t\Psi \neq \partial_k\Psi
\]

and that conflating actual time evolution with coarse-graining scale is a serious error.

**Required decision:**  
A formal parameter-identity model: parameter identity + declared role + domain + transformation law.

**Exit condition:** type system distinguishes physical time, RG scale, and other formal parameters; compatibility condition between `∂_t` and `∂_k` is explicit.

---

### 5. Additional blocker: interface ownership is under-specified

The TOE interfaces are:

- Dancer ↔ Stage
- Dance ↔ Stage
- Dancer ↔ Dance

These are physical relationships. `plan.md` says physical contracts belong in `Phys` or external `phys-*` libraries. But it does not say what kind of object an interface is.

Is a Dancer↔Stage interface:

- a mathematical relation in PhysMath?
- a physical contract in `phys-qm`/`phys-cst`?
- a bridge object?
- all three?

Without a clear answer, the bridge can silently merge frameworks.

**Required decision:** explicit cross-extension interface schema.

**Exit condition:** interface objects have an owning layer, explicit interpretation scope, and composability tests.

---

### 6. Additional blocker: mathematical vs physical dimensions

`B-007` is open, but the TOE context makes it urgent. PhysMath must manipulate \(c\), \(\hbar\), \(G\), metric signature, and regime conditions without owning physical dimensions or units.

If PhysMath cannot represent any dimensional structure, it cannot do theoretical mathematical physics. If it represents physical dimensions, it violates `REQ-PM-CON-017` and related ownership rules.

**Required decision:** distinguish mathematical dimension/index structure from physical dimension/unit semantics.

**Exit condition:** PhysMath can represent symbolic parameters and mathematical constraints; physical dimensions attach only through `Phys`.

---

### 7. Additional blocker: PhysMath constructor/mint authority

`Phys` has `MintObject` and a 7-point validation contract. `PhysMath` is exploratory, but it still needs a construction/validation authority for mathematical objects.

`B-022` covers security containment, but not the core question: who mints a PhysMath mathematical object, and what validation prevents malformed structures from corrupting research state?

**Required decision:** PhysMath constructor contract.

**Exit condition:** malformed mathematical objects are rejected; no physical ontology is allowed in constructors; exploratory objects remain distinguishable from trusted `Phys` objects.

---

### 8. Additional blocker: formalization handoff loss accounting

`B-015` and `B-029` are open. The bridge must classify exports as:

```text
FULLY_REPRESENTABLE
PARTIALLY_REPRESENTABLE
UNREPRESENTABLE
```

and must account for lost mathematical conditions, lost approximations, and physical assumptions that cannot attach.

Without this, a partial export can silently become trusted.

**Required decision:** exact bridge schema and loss-accounting format.

**Exit condition:** every exported component records source identity, target identity, mapping, losses, and required `Phys` metadata attachment.

---

### 9. Additional blocker: external `phys-*` composability contract

`B-028` is open. The TOE stress test requires `phys-qm`, `phys-bohm`, `phys-gr`, `phys-cst`, and possibly `phys-cdt` to compose without merging frameworks.

The plan says external libraries may depend on `Phys` and `PhysMath`. But it does not define the contract for declaring physical meaning over PhysMath mathematical objects.

**Required decision:** external Phys library extension schema/API.

**Exit condition:** interpretation-neutral vs framework-specific contracts are mechanically distinguishable; no package-local provenance or assumption system is allowed.

---

### 10. Additional blocker: AI-agent machine-readable schema

`B-017` is open. PhysMath is intended for AI agents. Without a versioned machine-readable interchange format, agents will fall back to prose, which violates the compiler-semantics principle.

**Required decision:** canonical AI-agent schema for mathematical objects, operations, errors, traces, and handoff.

**Exit condition:** an AI agent can construct, transform, attack, and hand off a hypothesis without relying on paper notation or prose interpretation.

---

## What is already correct

- The three-layer separation is coherent: PhysMath = exact symbolic exploration; Phys = trusted physical formalization; Numerical Layer = computation.
- Numerical computation is correctly excluded from PhysMath.
- Paper notation is correctly rejected as semantics.
- Physical ontology is correctly assigned to `Phys` or external `phys-*` libraries.
- The TOE is correctly treated as a workload, not as PhysMath ontology.
- The one-way promotion boundary and no-automatic-truth-adjudication rule are correct.
- The plan correctly refuses to let `Phys` grow just because PhysMath needs more mathematics.

---

## Required next actions

1. Close `B-001`–`B-010`: core mathematical IR, sorts, equality, scope, conditions, typing, tensor/geometry, calculus, approximations.
2. Close `B-024`–`B-034`: mathematical-vs-physical ownership, base-vs-extension admission, interpretation ambiguity, dependency graph, extension contract, handoff schema, metadata attachment, and condition-to-assumption enforcement.
3. Add and close the additional blockers above: core/bridge dependency, TOE joint dynamics, `t`/`k` flow, interface ownership, mathematical-vs-physical dimensions, PhysMath mint authority, loss accounting, external library composability, and AI schema.
4. Do **not** authorize implementation planning until the blocker register is closed or explicitly deferred with containment.
5. Re-run this adversarial review after the decision register is populated.

**FINAL STATUS: NOT READY.**  
The architecture is promising, but it is not yet mechanically enforceable. The boundary is the architecture; right now the boundary is still partly prose.