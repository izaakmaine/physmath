## question

why is equivariance under phys-lib/gr, is that the equivariance theorem

## answer

Yes—but there is an important distinction.

The thing under:

```text
phys-lib/gr/equivariance/
```

is **the executable canonical protocol fixture for an equivariance theorem instance**, not a new mathematical “Equivariance” type in `phys-math`.

The plan deliberately separates three layers:

### 1. The mathematical content belongs to `phys-math`

The actual mathematical objects are:

```text
f(v) = 2v
ρ_X(g,v) = g·v
ρ_Y(g,v) = g·v

ρ_Y(g, f(x)) = f(ρ_X(g,x))
```

and their representation as `Expression`, `Equation`, `Derivation`, `Theorem`, etc.

So **equivariance as a mathematical relationship is represented using ordinary PhysMath constructs**. There is no `Equivariance` AST node, no `GroupAction` sort, and no special equivariance family. The plan explicitly rejects those additions. plan9.3

### 2. The GR interpretation belongs to `phys-lib/gr`

The fixture culminates in:

```text
relation → Equation → Theorem + PhysicalTheorem
```

where the `PhysicalTheorem` carries the framework-side interpretation: assumptions, definitions/interpretations, and the GR `FrameworkArtifact` references. plan9.3

That's why the canonical fixture lives in `phys-lib/gr`.

It is not saying:

> “equivariance is inherently a GR mathematical concept.”

It is saying:

> “This particular executable theorem/protocol is a GR framework fixture demonstrating how PhysMath mathematics is given physical/framework meaning.”

### 3. The `equivariance/` directory is primarily a protocol/test fixture

Phase 6 is about migrating the **protocol package**:

```text
phys-lib/gr/equivariance/
    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go
```

The purpose is to prove that the canonical framework implementation can execute and replay this theorem instance according to the existing GR protocol semantics. The plan explicitly calls this the “canonical GR/equivariance protocol package.” plan9.3

So I would phrase it this way:

> **The equivariance theorem's mathematical representation is PhysMath; the GR-specific theorem instance and its executable protocol fixture live in `phys-lib/gr`.**

That distinction is actually important for the architecture. Otherwise we'd accidentally make `phys-math` aware that “equivariance” is a special mathematical ontology, which the plan deliberately avoids.


## question

so the equivariance theorem under gr different from equivariance theorem in bohmian mechanics?

## answer

Yes. **They can share the same abstract mathematical shape, while being different theorems once the physical meaning is attached.**

The crucial distinction is:

\[
f\circ \rho_X(g,\cdot)=\rho_Y(g,\cdot)\circ f
\]

is the **generic equivariance pattern**. PhysMath can represent that pattern without knowing whether it comes from GR, Bohmian mechanics, or something else.

But the actual theorem instance depends on what \(f\), \(X\), \(Y\), \(g\), \(\rho_X\), and \(\rho_Y\) mean.

### GR

An equivariance statement might concern how some mathematical/physical construction behaves under a transformation such as a change of coordinates or another symmetry transformation. Its assumptions and interpretation are relativistic/geometric.

### Bohmian mechanics

“Equivariance” has a much more specific physical meaning: the \(|\psi|^2\) distribution is preserved by the Bohmian dynamics under appropriate conditions. That is a substantially different physical theorem, even though it can be expressed using the same broad idea of a map commuting with an action/flow.

So:

```text
                 SAME MATHEMATICAL PATTERN
                           │
                           ▼
              f ∘ ρX = ρY ∘ f
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        GR interpretation        Bohmian interpretation
        different objects       different objects
        different assumptions   different assumptions
        different physics       different physics
```

And this exposes an important point about our current fixture:

**The `phys-lib/gr/equivariance` fixture is not a full physical GR equivariance theorem.** It is a deliberately minimal executable realization:

```text
f(v) = 2v
ρX(g,v) = g·v
ρY(g,v) = g·v
```

whose purpose is to prove the representation/protocol machinery. The plan explicitly calls it the canonical GR/equivariance protocol fixture and uses GR-side interpretation around the PhysMath theorem chain. plan9.3

So I would make the architecture mentally read as:

```text
phys-math
    └── generic mathematical equivariance structure

phys-lib/gr
    └── GR-specific equivariance theorem instances/protocols

phys-lib/bohm
    └── Bohmian-specific equivariance theorem instances/protocols
```

They **must not become one shared `Equivariance` physical ontology** in `phys-math`.

This is actually one of the strongest reasons the framework separation matters: **same mathematical structure, different physical semantics.** The architecture should let GR and Bohmian mechanics reuse the mathematics without pretending that their equivariance theorems are the same physical theorem.