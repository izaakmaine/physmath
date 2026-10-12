## The underlying structural problems

The recurring blockers in `plan3.md` (B-002, B-011, B-012, B-014, N-1, N-2, N-4, N-7) are not independent defects. They are symptoms of five deeper problems in how the relationship between Phys, PhysMath, and `phys-*` is being modeled.

### P1. The relationship is drawn as a stack, but it is actually a reference DAG

Every version of the plan draws `PhysMath → Phys`, `phys-* → Phys`, `phys-* → PhysMath`, `TOE-Agent → …`. Arrows flow upward. This framing forces the architecture to answer questions like:

- “Which layer owns provenance?”
- “Where is the metadata attached?”
- “How does a lower layer reference an upper layer?”

These questions would not arise if the relationship were modeled as **content-addressed artifacts that reference each other by digest**. Mathematics does not *depend* on physics; a physical interpretation *references* a mathematical structure; a trust assertion *references* a physical artifact. Three orthogonal concerns, not three layers.

The `physref/*` isolation package in §5, the `PhysicalArtifactEnvelope` in §9, and the `phys_metadata_refs[]` field in §26 all exist *because* the stack model creates leakage problems that the DAG model would not create in the first place.

### P2. Phys is being asked to serve two incompatible roles

Phys currently does two very different things:

1. It provides a **formal substrate** — `Expr`, `Kind`, `Dimension`, `MintObject`, canonical JSON, the operation set.
2. It provides a **trust authority** — provenance, corpus status, MRC, session ledger, candidate containment.

These are conceptually independent. Role (1) is one formal IR among possible formal IRs; it happens to be quantity-oriented. Role (2) is a general capability that could annotate *any* formal artifact, regardless of IR.

The plan inherits Phys's frozen design and therefore inherits the conflation. This is why §29–§30 have to accept that “much of the TOE will exceed the frozen Phys object model,” and why the “TOE Phys witness” has to be manufactured as a special acceptance criterion. If Phys's trust model were separated from Phys's formal substrate, the TOE could be fully trust-annotated without requiring its physics to fit Phys's quantity model.

### P3. The envelope pattern conflates identity, reference, and trust

`PhysicalArtifactEnvelope` in §9 bundles four distinct concerns into a single artifact:

- **identity** (`extension_id`, `object_id`, `object_type`),
- **mathematical references** (`math_refs[]`),
- **physical-contract references** (`physical_contract_refs[]`),
- **trust references** (`phys_metadata_refs[]`),
- plus a `payload`.

This is why N-4 (cross-extension payload interpretation) is unavoidable: if the envelope is the only interop shape, the payload must be introspectable, which contradicts the plan's own extension-ownership principle. If the payload is opaque, the envelope is doing no work beyond identifying the object.

The three concerns should be separated:

- An artifact *is* its payload, identified by content digest and namespace.
- A **reference** is a `(registry, digest)` tuple.
- A **trust annotation** is a separate record that references an artifact digest.

Under this model, an envelope is unnecessary; a physical artifact's cross-references are just fields of the artifact itself, and its trust annotation is a sidecar.

### P4. The math-condition / physical-assumption split is type-based, but the underlying distinction is provenance-based

§7 declares that `x > 0` is a mathematical condition in PhysMath, while `rest_mass_nonnegative` is a physical assumption in Phys. But in practice these are often the *same linguistic artifact*: `r > 2M` is both a mathematical inequality and a physical horizon condition.

The real distinction is not the type of the artifact. It is:

- **who asserted it**,
- **for what framework**,
- **in what scope**,
- **with what epistemic standing**.

These are all metadata attached to *an occurrence* of the artifact, not properties of its type. Making them into different types forces a translation step (`requested_phys_metadata[]`, the condition→assumption mapping in N-7) that the underlying semantics do not require.

### P5. The trust model is binary, but the epistemic reality is graded

PhysMath = “untrusted.” `phys-*` = “external unless formalized.” Phys = “trusted.” This binary view of trust is why the external-curation question (B-011, ADR-007) keeps returning: there is no middle ground where an external framework is *conditionally trustworthy for a stated scope*.

Phys's own corpus vocabulary (`ESTABLISHED`, `CONTESTED`, `SUPERSEDED`, `FALSIFIED`) already anticipates this. The problem is that the layer model collapses it: either you are inside the Phys trust boundary or you are outside.

### P6. Handoff is modeled as an artifact, but it is actually a protocol

§26 specifies a `HandoffPackage` as the boundary. But a real handoff involves:

- Project A proposing a fragment,
- Phys refusing due to unrepresentability,
- `phys-*` requiring additional contracts,
- Project A revising,
- Phys accepting.

This is a *negotiation*, not a transfer. Modeling it as a static artifact forces loss accounting (§27) to do work that a protocol would do naturally: the “loss report” is really “the sequence of refusals and revisions.”

---

## What I would do differently

The coherent alternative is to stop drawing the relationship as a stack and start drawing it as **four content-addressed registries that reference each other by digest**.

### 1. Replace layers with orthogonal concerns

There are four orthogonal artifact dimensions:

| Dimension | Artifact kind | Owner |
|---|---|---|
| Mathematical structure | Typed, canonical, exact | Whoever creates it |
| Physical interpretation | “This math structure represents that physical thing” | An interpretation framework |
| Trust assertion | “This artifact has this corpus/provenance standing” | A curation authority |
| Research state | “These artifacts are being investigated together” | A research process |

These are not layers. They are *dimensions along which any artifact may carry metadata*. A single physical artifact might carry:

- a mathematical payload,
- an interpretation scope,
- a trust annotation,
- a research provenance chain.

Each dimension is owned by a different authority, but they compose because they reference a *shared content digest* of the artifact.

### 2. Content addressing as the universal interop mechanism

Every artifact has:

```
RegistryNamespace + ContentDigest
```

That pair is the only required common structure. Any two components can interoperate by agreeing on this pair. There is no `Envelope`, no `physref/*`, no `math-core / math-domain` partition required for isolation — because PhysMath's artifacts and Phys's artifacts are all identified the same way.

Cross-references become `(namespace, digest)` tuples. Trust annotations are records of the form:

```
TrustAssertion {
    subject: (namespace, digest),
    provenance: DERIVED,
    corpus_status: CONTESTED,
    ...
}
```

The `subject` field makes no assumption about what *kind* of artifact it annotates. PhysMath can be annotated; a `phys-bohm` object can be annotated; a Project A hypothesis can be annotated. No layer dominance is implied.

### 3. Promote trust to a standalone concern; demote Phys's formal substrate

Phys-as-written conflates trust with a quantity-oriented IR. Split them:

- **PhysTrust** (new or refactored): a small, stable, theory-neutral trust model. It knows only `TrustAssertion` and the ledger that stores them. It has no `Kind`, no `Dimension`, no operation set.
- **Phys-Quantity-Formalism** (existing): Phys's current formal substrate. It becomes *one* framework that produces content-addressed artifacts, exactly like PhysMath.

Under this view, Phys's existing `mechanics/` and `relativity/` packages are not “the trusted base”; they are *one interpretation framework* whose artifacts happen to be trusted by curation. This is a reclassification, not a rewrite.

The benefit: the TOE workload no longer needs to force its rich physics through Phys's quantity model. It can produce its artifacts in PhysMath, annotate them with `phys-*` interpretations, and attach `PhysTrust` assertions by digest.

### 4. Make `phys-*` the interpretation registry, not an extension layer

A `phys-bohm` artifact should look like:

```
BohmianConfiguration {
    // payload is interpretation-specific
    math_refs: [(math, digest1), (math, digest2)],
    // optional trust annotations resolve against PhysTrust
    trust_refs: [(trust, digest3)],
}
```

The `math_refs` are just digests into PhysMath. The `trust_refs` are digests into PhysTrust. Neither is embedded; both are references. No envelope. No metadata attachment API. No duplicate provenance system — because the extension never holds a provenance; it holds a *reference to a trust assertion*.

### 5. Treat conditions as content-addressed; treat interpretation as annotation

A condition `r > 2M` is a PhysMath artifact. Its *interpretation* is a separate artifact:

```
PhysicalInterpretation {
    math_ref: (math, digest_condition),
    framework: "gr",
    meaning: "horizon condition for Schwarzschild mass M",
    scope: "static, exterior region",
}
```

The same condition can have multiple interpretations in different frameworks — GR, alternative gravity, numerics — without any of them claiming to *be* the condition. This dissolves N-7 (condition→assumption mapping) and simplifies §7 considerably.

### 6. Model handoff as a protocol, not an artifact

Replace `HandoffPackage` with a small protocol:

```
Proposal → Review(Phys) → {Accept, Reject(reason), RequestRevision(requirements)}
       → Revision → Review → …
       → AcceptedFragment
```

The durable artifact is `AcceptedFragment` — a tuple of `(math_refs, interpretation_refs, trust_assertions)`. Loss accounting becomes the log of rejected proposals, which is inherently complete because rejections are typed responses.

### 7. Specify the IR by composition rules, not sort names

`plan3.md` §12 lists sort names. What an IR specification actually needs to provide is:

- For each sort, its **domain and codomain**.
- Its **composition rules** — which sorts apply to which.
- Its **canonical form**.
- Its **equality class**.

`Function`, `Functional`, `Operator` are not peers; they are distinguished by *composition behavior*. Specifying them as sort names in a list leaves the semantics to the implementer, which is exactly what §43 forbids.

A minimal IR spec would answer: “Given `f : A → B` and `x : A`, is `f(x) : B` legal?” Without that, the IR is a vocabulary, not a type system.

---

## Why this resolves the recurring blockers

| Blockers | What dissolves them |
|---|---|
| B-002 `PhysMath → Phys` dependency | No project-level dependency. Both produce content-addressed artifacts. |
| B-011 external object contract | Envelope replaced by content-addressed artifact + reference tuple. |
| B-012 metadata attachment | Trust assertions attach by digest, not by API. |
| B-013/B-014 handoff + loss | Handoff is a protocol; loss is the typed refusal log. |
| N-1 math-core/math-domain partition | No partition needed; the isolation is provided by content addressing. |
| N-2 `physref/*` opacity | Opaque reference is just `(namespace, digest)`. |
| N-4 cross-extension payload | Payload is never introspected; composition is via references. |
| N-7 condition→assumption mapping | Interpretation is a separate artifact, not a promoted condition. |

The `UNREPRESENTABLE` target (N-6) also becomes local: an artifact is either resolvable by a trust registry or it is not. Unrepresentability in Phys *specifically* is a fact about one curation authority, not a property of the architecture.

---

## What this preserves

The three-project split is directionally right. The alternative above keeps it:

- **PhysMath** remains the mathematics registry.
- **phys-*** remains the interpretation registry.
- **Phys** is split into a trust authority (PhysTrust) and a formal framework (existing Phys), which resolves the conflation.
- **Project A** remains the research process, referencing artifacts across registries.
- **Numerical Layer** consumes only artifacts that resolve to trust-annotated interpretations.

The change is that these are no longer drawn as a hierarchy. They are drawn as **siblings that reference each other by digest**. The relationship is a DAG of artifacts, not a stack of layers.

That is the underlying architectural problem. Every recurring blocker in `plan2.md` and `plan3.md` is a symptom of trying to enforce a stack where a DAG belongs.