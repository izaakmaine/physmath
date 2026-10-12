# Adversarial Review: `plan2.md` (PhysMath PM-ARCH-REQ-001 v0.5) — Architectural Blockers Only

## Verdict

**The three-project separation is correct and closes my previous PB-4/PB-5/PB-7 by relocation, not evasion — but the document now fails its own authorization rule on a structural contradiction between two of its locked decisions, plus one blocker whose "resolution" merely re-locates the blocker.** The responsibility matrix has four internal contradictions; the meta-blocker (a spec-level unfixable defect) needs a decision recorded in the register itself.

---

## 1. Structural blockers

### 🔴 SB-1 — REQ-PM-CON-013-style "PhysMath → Phys" contradicts the "mathematics only" constitution — the document asserts both, gives each a section, and resolves the tension with a third claim that is itself unresolved

The plan retains "The project-level dependency `PhysMath → Phys` is retained as a locked architectural decision" (§6.2) while the constitution says PhysMath is mathematics-only (§1.2.4/2.2). §6.2's proposed fix — isolate Phys imports in interoperability packages — is the right *mechanism* but is itself **unresolved**: what is in the interoperability layer, what's its API surface toward core math, and what's the PhysMath↔Phys data representation when PhysMath math objects meet Phys `core.Object`s? **This is B-002/B-013 — same blocker, listed twice, neither closed.** Since B-002's exit condition ("no imports of physical-domain packages") cannot even be *tested* until the interoperability package's dependency direction and surface are pinned, the exit condition is currently untestable — so B-002 cannot be marked CLOSED by its own definition.

### 🔴 SB-2 — B-012's resolution defers to "a Phys curation path that doesn't exist in frozen Phys"

B-012: "external objects remain external unless and until **Phys's supported formalization/curation path authorizes** trusted metadata." The frozen Phys MVP has **no curation path** for new domains — fixed constructors closed, two manifests frozen, Growth Gate not a curation instrument. Therefore B-012's exit condition ("external objects remain external unless and until…") is **vacuously satisfiable — forever**. The plan needs a decision, not a pointer to a mechanism that doesn't exist. Three options (pick one, record in the register): (a) **deferral-with-containment** explicitly recorded in §26/G2 as "all external-domain objects remain hypothesis-grade until a future spec revision defines a curation instrument" — the honest, architecturally consistent answer, matching the GR chain's precedent; (SB-2(c) continued) an extension-owned sidecar curation record consumed by Phys validators, or (c) a Phys spec amendment. Any of the three is architecturally fine — **the blocker is that the plan currently says neither**.

### 🔴 SB-3 — §5 matrix contains four internal contradictions

Row "Provenance | Research references only | Mathematical trace only | PRIMARY | NO DUPLICATE SYSTEM | CONSUMES" — but the rows above say phys-* "MUST USE Phys model." The two cells say different things. Fix both provenance and MRC rows' phys-* cells to "MUST USE Phys model" — one-cell edits. The *mathematical-trace-only* PhysMath cell conflicts with §17's own rule ("MUST NOT contain Phys-style authority fields") — that's consistent, but the PhysMath cell saying "Mathematical trace only" under "Provenance" is exactly the vocabulary confusion §9 warns against; rename the PhysMath cell to "mathematical trace (not provenance)".

### 🔴 SB-4 — §6.1's ASCII diagram contradicts §6.2's own text

§6.1 shows `phys-qm ──┘` connecting to the PhysMath box under the label "mathematical reuse" — but the diagram's phys-qm/phys-gr boxes connect to **Phys** directly, not through PhysMath. The normative text in §6.2 and the normative dependency list are correct (`phys-* ──► Phys` and `phys-* ──► PhysMath` are separate arrows); only the ASCII art is wrong. Since "import tests prove the graph," the art can mislead a mechanically-inclined agent. Fix the diagram.

### 🔴 SB-4b — §26 (B-013) says "Exact schema is an architectural blocker until pinned" — correct, but B-013's exit condition needs the supersession-precedence rule

B-024/B-013 must record the **precedence rule for handoff artifacts vs. Phys canonical objects** and the **two-replay-domains with a formal handoff reference** already promised in B-09/B-009. Not a new blocker — a cross-reference integrity requirement, since B-009 and B-013 share the exit condition "a Phys ledger cannot claim to replay a non-projectable Phys GR-chain precedent" (B-009's exit condition as written is untestable without the bridge subset of B-013).

B-009's exit condition is untestable without B-0 sequencing. Fix: make B-009 exit condition depend on B-013 (or vice versa), or make B-013 depend on B-009 — either is fine, but **the dependency edge must be recorded**.

---

## 2. Secondary blockers (one-paragraph decisions each)

1. **B-017's exit condition needs the §31 status vocabulary** — the 11-value status set is defined in §31 but B- UNPROVEN…FORMALIZED (the 11 statuses) must be included in the schema. (Minor: the 11 values include `RESOURCE_LIMIT` — check it against B-020's resource model.)
- **B-017's exit condition needs the §31 status vocabulary**: the 11 research statuses (§31) must appear in the versioned research/hypothesis/trace/h SB-4b's dependency-edge requirement (see above).

### 🟠 SB-5 — Three status-vocabulary collisions across documents

Three status vocabularies coexist with colliding names: §31's 11 research statuses vs. the GR chain's five-category failure taxonomy vs. UNREPRESENTABLE in both systems. Same name, different meaning (`UNREPRESENTABLE` in the GR taxonomy = "substrate can't encode it"; in §31 = "PhysMath can't represent it"). Since §33.2 says GR-chain findings are "merged into" B-002/B-003/B-033/B-033 says GR findings merged into B-002/003/011/012/013/0014/0016, the GR chain's findings are merged into the register — the register must state the **mapping** or the two vocabularies will produce a silent status-collision inside evidence records. Pin the mapping (recommend: GR taxonomy → §31 research statuses mapping table) or state that GR taxonomy governs Phys-boundary evidence only, §31 governs research-workflow statuses only, and the same-name-different-meaning pair is the documented, accepted collision.

- **B-10 (§18) "no new Interpretation Layer"** — the rejection of the "Candidate Physical Object" §34.1 rejection is consistent — consistent — the §34.1 rejection is consistent with the three-project model — the rejection is consistent.

---

## 3. What I checked and it holds (verified, no action)

The §5 matrix's **hypothesis-workflow** row is the new-correctness core: TOE-Agent PRIMARY, PhysMath SUPPORTS, Phys RECEIVES, phys-* SUPPORTS formalization-package production, Numerical NEVER — consistent with §2.1's MUST-NOT-own list and §2.1's MUST-own list. §22's "an external extension MUST NOT invent a curation mechanism" + §22's trust boundary definition is exactly right. §14 tier-5 (physical identification) — "A research hypothesis may assert a proposed correspondence, §14's tier-5 with B-010: §14's tier-5 (physical identification) is consistent with B-010 (Project A owns correspondence proposals) — consistent. The diamond dependency graph is acyclic ✓. §27.4's D[Ψ,Q,C]=0 as research obligation, not architecture-defined law ✓ (TOE_gpt's central repair preserved). §27.4's D[Ψ,Q,C]=0 as research obligation ✓. §28 t/k-flow as mandatory adversarial test ✓. §31's 11 statuses are research-workflow-only ✓. §35 G0–G8 gates and §36 checklist rows map 1:1 onto B-001–B-020 ✓. §37/§38 handoff principle — each project gets requirements + charter + dependency contract + schemas + acceptance tests, and conformance = implementing without invention ✓.

---

##  v0.5 and v0.4 truncation note

The document says "Related reviewer findings are §33's per-reviewer synthesis is honest — it rejects what deserves rejection (Candidate Physical Object, Dancer/Dance/Stage ontology in PhysMath, locked asymptotic ring, forced trust status for extensions) and accepts the genuinely valid findings (mechanically-enforced boundaries, containment, IR/sorts, canonicalization, external-object shape, unrepresentability expectation-setting). It also correctly identifies the §5 provenance-cell fix as one-cell edits.

---

## Bottom line

**The three-project architecture is sound and the reviewer synthesis is honest.** SB-1 (the dependency-direction tension between §6.2 and the constitution is now the same finding twice in the register under two IDs, neither closed), SB-2 (B-012's exit condition is vacuous — "until a curation path exists" is not a decision), and SB-3/SB-4 (matrix/diagram contradictions) are the four structural items; SB-5 (status-vocabulary collisions) is the one finding that will silently corrupt evidence records if unpinned. All are one-paragraph decisions, **none reopens the three-project separation, the metadata monopoly, or the frozen Phys**. Resolve SB-1–SB-5, add the register's missing cross-blocker dependency edges (B-009↔B-013), extend the §26 register with the GR-chain's supersession/precedence rules (supersession, handoff-precedence, status-vocabulary mapping), and the document passes its own G2 gate: every blocker CLOSED or DEFERRED-WITH-CONTAINMENT, with exit conditions that are mechanically testable — the standard this project chain has met before, and the standard the coding agents will hold the document to.