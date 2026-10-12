Phase 2 is where the foundation we just built becomes a **research system rather than primarily a representation/verification system**.

One important caveat: **the documents we've closed so far do not define a detailed Phase-2 specification**. So the following is the architectural implication of Phase 1 and the explicitly deferred items, not a claim that a Phase-2 contract has already been decided.

## What Phase 1 has actually established

We now have:

```text
phys-artifact
    = canonical identity + serialization

phys-math
    = pure mathematical representation + seven deterministic rules + replay

phys-lib
    = physical/framework semantics

core
    = deliberately empty
```

And, importantly, we have demonstrated:

```text
same mathematics
        ↓
different framework interpretations
        ↓
without contaminating phys-math
```

The GR fixture and fixture2 establish that principle experimentally.

Phase 2 therefore shouldn't be "more infrastructure." It should test whether that separation survives **real theories**.

---

# What Phase 2 should mean

The natural Phase-2 progression is:

```text
Phase 1
Can we represent mathematics and framework semantics correctly?

            ↓

Phase 2
Can we represent substantially different physical theories
using the same mathematical substrate?

            ↓

Phase 3+
Can we reason across those theories, compare them,
identify conflicts/gaps, and eventually build research tooling?
```

The important shift is from **architecture validation** to **scientific expressiveness evaluation**.

---

# The most important Phase-2 candidates

The Phase-1 plan explicitly deferred production packages for:

```text
qm
bohm
cst
cdt
```

and also deferred full Schwarzschild/GR computational re-expression. plan9.3

That gives us a very natural Phase-2 experiment:

```text
phys-lib/
    gr/
    qm/
    bohm/
    cst/
    cdt/
```

But the goal should **not** be "implement five theories."

The real experiment is:

> **Can each theory express its distinctive physical semantics while reusing the same `phys-math` mathematical substrate?**

That's a much stronger test.

---

# Bohmian mechanics is especially important

Our earlier equivariance discussion is actually a perfect Phase-2 test.

In Phase 1 we have a minimal mathematical pattern:

\[
f(\rho_X(g,x))=\rho_Y(g,f(x)).
\]

In Bohmian mechanics, "equivariance" has a much more specific physical meaning.

So Phase 2 could expose a very useful comparison:

```text
                phys-math
                    │
            mathematical structure
                    │
       ┌────────────┴────────────┐
       │                         │
      GR                        Bohm
       │                         │
 different physical          different physical
 interpretation              interpretation
 assumptions                  assumptions
 theorem packaging            theorem packaging
```

This will tell us whether our framework boundary is actually doing useful work.

The desired outcome is **not** that GR and Bohm share a universal `Equivariance` object.

It is that they can both reuse the appropriate mathematics without pretending their physical claims are identical.

---

# Phase 2 also implies a much harder test of `phys-math`

Phase 1 used a deliberately tiny mathematical universe.

Phase 2 will expose whether that universe is actually sufficient.

For example, a future framework may require:

- richer function composition
- more sophisticated transformations
- limits/asymptotics
- differential operators
- tensor operations
- probability distributions
- measures
- geometry
- discrete structures

At that point we need to ask:

> Is this genuinely **mathematics that belongs in PhysMath**, or is it mathematical structure specific to one physical theory?

That is where the Architecture of Complexity principle becomes operational again.

The important discipline is:

```text
new requirement
    ↓
classify
    ├── theory-neutral mathematics → phys-math candidate
    ├── physical meaning → phys-lib/<framework>
    └── unclear → keep specific until evidence accumulates
```

So Phase 2 is likely where the **real pressure on the empty `core` and closed PhysMath vocabulary begins**.

---

# Phase 2 implies much more serious theorem semantics

Phase 1's equivariance fixture taught us something important:

```text
representation
≠
derivation
≠
theorem
≠
interpretation
```

Phase 2 will have to deal with actual theoretical claims rather than the minimal six-step fixture.

That means every substantial theory package will need to preserve:

```text
physical claim
      ↓
assumptions
      ↓
mathematical representation
      ↓
derivation
      ↓
conclusion
      ↓
physical interpretation
```

The semantic-liveness protocol we added to `phys-system/AGENTS.md` therefore becomes much more important in Phase 2 than it was in Phase 1.

---

# Phase 2 is also where "same mathematics, different physics" becomes measurable

The most interesting thing we can start doing is finding structures such as:

```text
             Mathematical artifact M
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
          GR          Bohm         CDT
          │            │            │
       meaning A    meaning B    meaning C
```

Then ask:

1. Which mathematics truly is shared?
2. Which assumptions differ?
3. Which conclusions are framework-specific?
4. Which concepts we thought were generic turn out to be theory-specific?
5. Where do theories genuinely conflict rather than merely use different terminology?

That is much closer to the eventual **Theory-of-Everything evaluation framework** you have been aiming toward.

---

# What Phase 2 should *not* imply

It should not mean:

```text
"Now build everything."
```

In particular, the Phase-1 deferred boundaries should not be casually reopened:

- no automatic expansion of PhysMath
- no giant physics ontology
- no universal `PhysicalContract`
- no generic "theory" abstraction just because multiple packages exist
- no framework registry for its own sake
- no early numerical engine
- no premature agent orchestration
- no full GR computational rewrite just because GR exists
- no assumption that every shared concept belongs in `core`

The lesson from Phase 1 is actually the opposite:

> **Make the smallest abstraction survive multiple theories before promoting it.**

---

# The real implication for the architecture

Phase 1 established:

```text
                    PHYS-SYSTEM
                         │
       ┌─────────────────┼──────────────────┐
       ↓                 ↓                  ↓
phys-artifact        phys-math          phys-lib
                                          │
                              ┌───────────┼───────────┐
                              ↓           ↓           ↓
                             GR         Bohm        CST...
```

Phase 2 should start filling the right-hand side **without enlarging the left-hand side merely because a new theory needs something**.

That gives us a very useful empirical loop:

```text
new theory
   ↓
encounter mathematical requirement
   ↓
classify ownership
   ↓
implement minimally
   ↓
test against existing theories
   ↓
promotion only when genuinely theory-neutral
```

That is a much stronger way to grow the system than designing a huge universal ontology up front.

## In one sentence

**Phase 1 proved that the architecture can keep mathematics, physical semantics, and framework identity separate; Phase 2 should prove that real, substantially different physical theories can actually live on that foundation without forcing us to collapse those boundaries.**

And that means the first Phase-2 target should probably be chosen less for completeness and more for **maximum architectural stress**—Bohmian mechanics is a particularly strong candidate because it will pressure the representation of probability, dynamics, quantum state, and equivariance in ways that GR does not.