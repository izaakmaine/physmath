## prompt

another requirements:

- PhysMath must be under github.com/PithomLabs/phys-math (pure math objects, no physical objects)
- Phys-* libraries must be under github.com/PithomLabs/phys-lib separating core libraries (theory-agnostic) from theory/framework-specific

for both requirements, we must have clear guidelines and protocol as to which is core/specific split per Architecture of Complexity principle and following the Go library design

does that materially affect the current implementation plan or need to accommodate it?

## answer

Yes. **It materially affects the current implementation plan, and we should accommodate it before the coding agent starts coding.** It does **not** overturn the mathematical architecture, but it changes the **public Go library architecture and the destination of the `phys-gr` refactor**.

The current plan targets:

```text
github.com/PithomLabs/phys-artifact
github.com/PithomLabs/physmath
github.com/PithomLabs/phys-gr
```

with `phys-gr` remaining a standalone module. plan9(2)

The new requirement should instead make the target:

```text
github.com/PithomLabs/phys-artifact
        ↓
github.com/PithomLabs/phys-math
        ↓
github.com/PithomLabs/phys-lib
          ├── core/          theory-agnostic physical libraries
          ├── gr/            general relativity
          ├── qm/            quantum mechanics
          ├── bohm/          Bohmian interpretation
          ├── cst/           causal set theory
          └── cdt/           causal dynamical triangulation
```

The important distinction is that **`phys-gr` becomes the migration/source repository, not the final architectural home of GR**.

## I would change one thing from our earlier module decision

I would **not** make every `phys-*` package a separate Go module.

Use:

```text
github.com/PithomLabs/phys-lib
```

as **one Go module containing multiple focused packages**:

```text
phys-lib/
    go.mod
    core/
    gr/
    qm/
    bohm/
    cst/
    cdt/
```

while keeping `phys-math` and `phys-artifact` as independent modules.

So the development workspace becomes:

```text
workspace/
├── phys-artifact/
├── phys-math/
└── phys-lib/
    ├── core/
    ├── gr/
    ├── qm/
    └── ...
└── go.work
```

This is more idiomatic Go than creating a multi-module repository for every physical framework. Go's own guidance notes that multiple modules in one repository add considerable management/versioning complexity and are generally not the default recommendation; workspaces are specifically intended for developing multiple modules together. [Go](https://go.dev/wiki/Modules?utm_source=chatgpt.com)

That actually aligns beautifully with our **Architecture of Complexity**:

```text
repository/module boundary
    → major ownership boundary

package boundary
    → core/specific boundary

interface boundary
    → cohesion without coupling
```

## The critical architectural rule

We should make this explicit for the coding agent:

> **A capability belongs in `phys-lib/core` only when its mathematical/physical semantics are independent of any particular theory, framework, interpretation, or model. Otherwise it belongs in a theory/framework-specific package. When a capability contains both common structure and theory-specific behavior, split it at the narrowest stable interface.**

That becomes our **core/specific promotion protocol**.

### Core eligibility test

Before moving anything from `phys-gr` into `phys-lib/core`, the agent should ask:

```text
1. Does its meaning require GR?
2. Does its behavior depend on GR assumptions?
3. Would QM use the same abstraction with the same semantics?
4. Would another theory reasonably reuse it?
5. Can its API/documentation be explained without naming a specific theory?
6. Does moving it to core reduce duplication rather than merely relocate code?
```

If the answer exposes theory dependence:

```text
→ specific
```

If only part is theory-dependent:

```text
→ split
```

If genuinely theory-independent:

```text
→ core
```

And critically:

> **“Useful to us” is not enough to make something core.**

That prevents the core from becoming a `common`/`util` dumping ground. Go's package guidance explicitly recommends focused packages with meaningful names and warns against generic `util`, `common`, `misc`, `api`, or `types` packages because they accumulate unrelated dependencies and weaken the abstraction boundary. [Go](https://go.dev/blog/package-names?utm_source=chatgpt.com)

## This changes the `phys-gr` refactoring destination

The current implementation plan says, for example, that `symbolic/expr.go` generic pieces move to PhysMath while GR pieces remain in `phys-gr`, and that `metric`, `connection`, `curvature`, etc. remain in `phys-gr`. plan9(2)

That now becomes:

```text
existing phys-gr
      │
      ├── generic mathematical representation
      │        ↓
      │   github.com/PithomLabs/phys-math
      │
      ├── genuinely theory-agnostic physical infrastructure
      │        ↓
      │   github.com/PithomLabs/phys-lib/core
      │
      └── genuinely GR-specific semantics
               ↓
          github.com/PithomLabs/phys-lib/gr
```

That is actually a **better implementation experiment** because it gives us three classification destinations instead of two:

```text
MATHEMATICAL COMMONALITY
        → phys-math

PHYSICAL COMMONALITY
        → phys-lib/core

THEORY / FRAMEWORK SPECIFICITY
        → phys-lib/<framework>
```

This is the exact recursive application of our complexity principle.

## Dependency rule should become

I would lock this:

```text
phys-artifact
      ↑
phys-math
      ↑
phys-lib/core
      ↑
phys-lib/<theory>
```

More precisely:

```text
phys-lib/core
    → phys-math
    → phys-artifact

phys-lib/gr
    → phys-lib/core
    → phys-math
    → phys-artifact

phys-lib/qm
    → phys-lib/core
    → phys-math
    → phys-artifact
```

but:

```text
phys-lib/core  -X→ phys-lib/gr
phys-lib/core  -X→ phys-lib/qm
phys-lib/gr    -X→ phys-lib/qm
phys-math      -X→ phys-lib/*
phys-artifact  -X→ phys-math
phys-artifact  -X→ phys-lib/*
```

This gives us a very clean partial order.

A package that cannot be explained without importing a specific framework should not be in `core`.

## It also changes how we think about Go package design

`phys-lib/core` should **not** necessarily become one giant package called `core`.

Following Go's package-design guidance, package boundaries should reflect coherent client-facing abstractions; package names are part of the API design, and an inability to find a meaningful package name can indicate that the abstraction boundary itself is wrong. [Go](https://go.dev/blog/package-names?utm_source=chatgpt.com)

So the eventual shape may be:

```text
phys-lib/
    core/
        <focused theory-agnostic packages>

    gr/
        geometry/
        metric/
        curvature/
        ...

    qm/
        ...
```

rather than:

```text
phys-lib/
    core/
        everything-theory-independent.go

    gr/
        everything-gr.go
```

But **the agent should derive the actual `core` subpackages from the existing code and client/API analysis**, not invent a taxonomy upfront.

## Protocol should also reflect the split

Every promoted component should have an explicit classification record in the implementation plan:

```text
Component:
Current location:

Candidate:
    PhysMath
    phys-lib/core
    phys-lib/<specific>
    split
    delete
    defer

Reason:
Theory dependence:
Framework assumptions:
Reusable interface:
Known consumers:
```

This becomes the implementation-time version of our Architecture of Complexity.

It prevents:

```text
"this looks generic"
        ↓
move to core
        ↓
core slowly becomes coupled to everything
```

## One more important consequence

The **many-frameworks test becomes stronger**.

Instead of merely proving:

```text
phys-gr ≠ phys-qm
```

we can test:

```text
phys-lib/core
       ↑
   ┌───┴────┐
   │        │
 phys-gr  phys-qm
   │        │
different framework semantics
```

while both consume the same:

```text
phys-math
```

That gives us an actual executable demonstration of:

> **common denominator in the core, differences in the specific layer.**

This is much stronger than a synthetic second framework.

## Bottom line

**Yes, revise the implementation plan now.**

The target architecture should become:

```text
                 phys-artifact
                       │
                       ▼
                 github.com/
                PithomLabs/
                  phys-math
                       │
                       ▼
                github.com/
               PithomLabs/
                 phys-lib
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
           core              framework-specific
       theory-agnostic       ┌────┬────┬────┐
                             gr   qm  bohm  ...
```

The existing `phys-gr` repository is then the **refactoring laboratory**, not the final library boundary.

This is a **material update to plan9(2)**: its module layout, refactoring destinations, dependency graph, Phase 4/7/8 work, conformance tests, and core/specific classification protocol all need to be amended. Its central PhysMath design and equivariance protocol remain intact. The current plan already gives us the right raw material for making that revision. plan9(2)

And I think this is a **better architecture than the previous plan**, because the distinction now becomes explicit at three levels:

```text
phys-artifact = mechanical substrate
phys-math     = mathematical common denominator
phys-lib/core = physical common denominator
phys-lib/gr   = theory/framework-specific semantics
```

That is exactly our **core / specific split applied recursively**, while remaining consistent with idiomatic Go package design.

## Summary

The canonical target is now:

```text
phys-artifact
      ↓
github.com/PithomLabs/phys-math
      ↓
github.com/PithomLabs/phys-lib
      ├── core
      └── <theory/framework-specific>
           ├── gr
           ├── qm
           ├── bohm
           ├── cst
           └── cdt
```

with the recursive complexity rule:

```text
phys-artifact = mechanical substrate
phys-math     = mathematical common denominator
phys-lib/core = theory-agnostic physical common denominator
phys-lib/<x>  = theory/framework-specific semantics
```

The existing `phys-gr` repository is the **refactoring laboratory**, not the final architectural home of GR. 

Every future core/specific split must be justified through the explicit complexity principles and coherent Go package boundaries, rather than convenience or code reuse alone.