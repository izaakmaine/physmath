# The Underlying Structural Problems — and What I Would Do Differently

## The headline

The **partition is right; the joints are wrong.** Three-project separation (research / mathematics / physical semantics) with Phys as trust substrate is the correct architecture and should not be reopened. But the architecture was assembled by accretion around a frozen core, and every joint inherited the same pattern: each project re-invents its own miniature version of identity, canonicalization, replay, and trust — then the architecture writes prose telling the mini-models not to conflict. The structural debt is not in any layer; it is in **what crosses between them and what nothing yet owns**. Seven problems, five of which have a common cure.

---

## The seven underlying problems

**P1 — The trust gradient has no upward path.** Phys's trust model admits nothing new: constructors are closed, the only public raw mint is forced-`HYPOTHESIS`, and there is no curation instrument. The consequence is structural, not cosmetic: the ecosystem has **two castes** — the frozen trusted island (two physics domains, forever) and an untrusted archipelago (every future domain, permanently hypothesis-grade). The more the ecosystem succeeds, the larger the untrusted zone grows relative to the trusted one. Trust does not accumulate. plan3's §29 deferral is *honest* but defines no ladder — "until a future Phys specification defines a mechanism" is a placeholder, not a trajectory.

**P2 — Phys is two systems wearing one coat.** Look at what Phys actually contains: a **type system** (`Expr`, `Kind`, `Dimension`, canonicalization, ops) and a **trust registry** (`Provenance`, `CorpusStatus`, MRC, sessions, ledgers). These have completely different reuse relationships — PhysMath constitutionally needs *neither*, phys-* needs the registry and maybe the type system, the Numerical Layer needs formalized artifacts. Forcing all consumers through one facade is why plan3 needs the entire §5 `physref` quarantine apparatus, the import tests, and the may/may-not lists: they exist to defend against a dependency that delivers almost nothing.

**P3 — There is no shared border currency.** Every crossing is pairwise bespoke: PhysMath → Phys (the projection subset — the question the GR chain spent three revision cycles pinning as D22–D24), phys-* → Phys metadata, PhysMath → phys-* math refs. Multiply by 17 PhysMath sorts and N extensions and the bridge becomes the dominant cost of the whole program. The GR chain proved a *single pair* (GRExpr → `core.Expr`) required exact pins of subset, minting path, IDs, and provenance law. The architecture treats this hardest problem as a `mapping` field in an export item.

**P4 — Three parallel meta-models, and the replay count is already false.** Phys has object hashes and ledger replay; PhysMath has (unpinned) content identity and trace replay; extensions have envelope IDs. No section defines one ecosystem story for *what an identity is, what versioning means, what canonical means*. Worse: plan3 §21 says "exactly two formal replay domains" — but the GR chain already built a third (the GRStep chain, D32), and every recording extension (phys-qm guidance dynamics, phys-cdt histories) will need its own. "Two" is false by the architecture's own precedent; carrying it forward forces extensions to choose between unrecorded operations and undocumented replay domains.

**P5 — phys-\* is a family with no shared kit.** The architecture governs each extension's relationship to Phys/PhysMath but provides nothing for the family's uniformity. Every extension that wants to exercise Phys ops needs the same bridge machinery; every extension needs envelope validation, import-graph tests, no-duplicate-metadata scans, interpretation-classification checks. If each of five packages builds its own, you get five divergent bridge implementations with five drift classes — the exact failure mode the whole document chain exists to prevent, reintroduced at family scale.

**P6 — Extension trust is deferred without a named target.** Related to P1 but distinct: even granting that *object-level* trust for new domains is future work, nothing defines what the future instrument would even be, so extensions can't be built curation-ready. "Reusable and formally specified while remaining external" is a status, not a design.

**P7 — Three growth mechanisms, no coordination.** Phys has a Growth Gate; PhysMath has a versioned sort-extension protocol; extensions have admission criteria; Project A is a research program. §38 routes a capability to its *destination* but nothing sequences *coordinated* additions (asymptotics live in PhysMath, but asymptotics-with-regime-semantics touch phys-*, and `APPROXIMATED` activation someday touches Phys — one capability, three gates, no ordering discipline).

---

## What I would do differently

**D1 — Cut the PhysMath → Phys dependency entirely (amend ADR-003).** PhysMath becomes a pure mathematics library with zero dependencies beyond stdlib. The bridge is Project A's handoff tooling — which plan3 §26 already assigns there. This dissolves P2's whole problem class: no physref quarantine, no containment import tests, no "project-level dependency vs. mathematics-only" tension to explain. The constitution's strongest claim — *PhysMath is mathematics only* — becomes mechanically true in the strongest available sense: **it cannot touch physical semantics because it does not know Phys exists.** The one capability §5's physref preserves (schema-compatibility verification) moves to the bridge layer where it belongs.

**D2 — Define PhysMath's `Expression` sort as the deliberate semantic mirror of `core.Expr`.** Same node set (quantity-expression subset: Symbol/Rational/Add/Mul/Neg/Pow/Sqrt), same canonicalizer semantics, same exponent domain — documented as an obligation, tested by canonical round-trip. Relations and branch structures stay in PhysMath's separate `Relation`/branch sorts and cross via their own mappings (as GR's Compare artifacts stayed `core.Object`s). This converts P3's hardest crossing from translation to **identity**: the interchange subset stops being a bridge feature and becomes a sort definition. The richer sorts (Function, Operator, Tensor, manifolds) are non-bridgeable under *any* scheme — frozen Phys cannot hold them — so nothing is lost, and the honest UNREPRESENTABLE boundary becomes crisp rather than negotiated per export.

**D3 — Name the fourth component: `phys-interop` — shared infrastructure, not an authority.** One versioned module containing: the bridge triple generalized from GR D22–D24 (`ToPhysExpr` / `ToPhysObject` via `hypothesis.NewCandidateConcept`, forced `HYPOTHESIS`/`NONE`, content-derived IDs / `FromPhysExpr`), the envelope codec, the ecosystem identity scheme, and the extension test-kit (import-graph, no-duplicate-metadata, envelope validation, interpretation-classification checks). It owns no semantics; every rule it enforces is defined in the architecture. Extensions depend on Phys + PhysMath + phys-interop; Project A depends on everything public. This is the direct generalization of what the GR chain already built ad hoc — built once, reviewed once, instead of quintuplicated (resolves P5, and gives P3's crossing its only sensible home).

```text
Phys          ← stdlib only (frozen)
PhysMath      ← stdlib only (pure mathematics; Expression sort = Phys mirror)
phys-interop  ← Phys + PhysMath (bridge, envelope, identity, test kit)
phys-*        ← Phys + PhysMath + phys-interop
Project A     ← public surfaces of all of the above
Numerical     ← Phys + selected phys-*
```

Acyclic, two zero-dependency foundations, one shared infrastructure layer. The mirror design makes phys-interop *small* — the bridge is near-trivial when both sides of the scalar border are the same semantics.

**D4 — Replay domains are per-authority, not "two."** Phys ledger, PhysMath trace, and one operational trace per recording extension (GR already is the third). Each domain self-contained; cross-references by digest only. Corrects the false count before five extensions inherit it.

**D5 — Replace the open deferral with a named trust ladder.** (a) Accept the two castes explicitly — hypothesis-grade is the *correct* epistemic status for new science, not a demotion; the system's honesty is the product. (b) Name the future instrument as a design target: a **CurationRecord** — *library-level* admission (a human-signed attestation to an extension's contract and corpus claim, validated by a Phys validator hook), which is governance, not physics — a small, generic, theory-neutral Growth Gate candidate. This is how real package ecosystems work: the stdlib doesn't absorb packages; maintainers vouch for versions. (c) Interim: ecosystem-level attestation records live in phys-interop, explicitly labeled non-authoritative. Same honesty as §29, but "here is the design we will build when authorized" instead of "someone should eventually."

**D6 — One ecosystem meta-model chapter.** Identity (content-addressed everywhere, one scheme), versioning (schema_version per artifact family, one compatibility rule), canonical (one definition, per-sort/per-artifact obligations), naming (the D31 no-shadowing discipline extended across all five authorities), replay (D4's enumeration). This kills the vocabulary-collision class *permanently* rather than patching pairs — the structural version of the SB-5 fix.

**D7 — A capability ledger.** One register where cross-project capability needs are logged, routed through §38's tree, and *sequenced* when multiple projects must move together. Lightweight — a table and a gate step — but it is what prevents the next integration from rediscovering today's boundary problems.

---

## Why this is more cohesive, and what it costs

The cohesion gain is concentrated: **D1–D3 alone eliminate the majority of the structural debt** — the PhysMath/Phys tension class (SB-1's entire lineage), the projection/minting/identity residuals (R-1, R-2), and the family-uniformity problem (P5) — without touching a frozen byte. The architecture stops being five authorities connected by prose and becomes two pure foundations, one infrastructure layer, and a family that inherits enforcement rather than re-implementing it.

The costs, honestly: the mirror obligation constrains PhysMath's scalar layer to Phys's semantics (a constraint worth paying — that layer was never going to bridge anything richer); D1 amends a CLOSED ADR (justified — the dependency delivers near-zero value and generates the entire containment apparatus); phys-interop is a fifth artifact to govern (but absorbs work otherwise done five times); and the CurationRecord requires a future, small Phys amendment (a named target is a better trajectory than an open deferral at identical honesty).

**Bottom line:** nothing in the three-project split needs reopening. The relationship problems are all *connective-tissue* problems — dependency direction, border currency, identity, replay, trust ladder, family kit — and each has a proven precedent somewhere in the GR chain that was solved locally but never generalized. Generalizing those five precedents, cutting one vestigial dependency, and writing the meta-model down once is the difference between an architecture that is *correct* and one that is *coherent*.