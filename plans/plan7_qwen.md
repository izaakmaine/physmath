# Adversarial Review of `plan7.md` (Gate H: Theory/Epistemic Package Isolation)

## Executive Verdict
`plan7.md` is a critical and conceptually sound hardening pass. It correctly translates the epistemic isolation rules (from `specs_falsifiabilty.md` and `adv_review5.md`) into mechanical, testable gates. 

However, when subjected to the strict "zero ambiguity for the coding agent" standard, **four specific blockers** emerge. These are not architectural flaws, but rather **specification traps** where the plan's wording will cause a coding agent to write incorrect tests, fail to compile, or misinterpret the allowlists. 

Below is the exact deconstruction of these traps and the required patches to make `plan7.md` 100% bulletproof.

---

### ❌ BLOCKER 1: The "mechanics-only keys" trap in H4
**The Flaw:** The plan states: *"mechanics-only keys enumerated from its manifest in-implementation."*
**The Reality:** The mechanics manifest contains *framework-level* scope metadata (e.g., `classical_nonrelativistic`), not *object-level* assumption keys. Relativity, by contrast, has object-level keys (`rest_mass_nonnegative`, `speed_of_light_positive`, `rest_frame`). If the coding agent tries to extract object-level assumption keys from the mechanics manifest, they will find none, leading to confusion or a test that incorrectly checks framework metadata on individual objects.
**The Fix:** Explicitly state that the set of mechanics-specific *object-level* assumption keys is **empty**. 
*Patch text for H4:* "The mechanics package has zero object-level assumption keys. The test must assert that no mechanics object contains any relativity-specific keys (`rest_frame`, `rest_mass_nonnegative`, `speed_of_light_positive`). Conversely, relativity objects must not contain non-relativity keys."

### ❌ BLOCKER 2: The "hypothesis concept" misclassification in the Reverse Allowlist
**The Flaw:** The reverse allowlist includes `hypothesis concept` as an *"ESTABLISHED-yielding constructor"*.
**The Reality:** `hypothesis.NewCandidateConcept` explicitly yields `HYPOTHESIS` provenance and `NONE` corpus status. It **does not** yield `ESTABLISHED`. Including it in an ESTABLISHED allowlist is a category error that will break the coding agent's AST/corpus-status validation logic.
**The Fix:** Remove `hypothesis concept` from the ESTABLISHED allowlist. Provide the exact, exhaustive list of non-manifest, ESTABLISHED-yielding object constructors.
*Patch text for Reverse Allowlist:* "Pinned explicit set of non-manifest, ESTABLISHED-yielding object constructors: `{mechanics.NewKineticEnergy, relativity.NewVelocity, relativity.ZeroThreeMomentum, relativity.ZeroEnergy, relativity.ZeroVelocity}`. Every other ESTABLISHED-yielding zero-argument constructor must map 1:1 to a manifest item."

### ❌ BLOCKER 3: H3 "explicitly required/generated keys" is open-ended
**The Flaw:** H3 requires output assumption keys to be a subset of inputs + *"explicitly required/generated keys — e.g. denominator preconditions, `selected_branch/*`"*.
**The Reality:** A coding agent needs a closed, exhaustive list to write a deterministic test. If the list is open-ended ("e.g."), the test will either be too loose or fail on unexpected but valid keys.
**The Fix:** Pin the exhaustive list of operation-generated assumption keys for the MVP.
*Patch text for H3:* "The exhaustive, closed set of operation-generated assumption keys in MVP is exactly: 
1. `denominator/<hash>` (generated exclusively by `Divide`)
2. `selected_branch/<hash>` (generated exclusively by `SelectBranch`). 
Any `ops` function introducing a new assumption key outside this exact set is a violation."

### ❌ BLOCKER 4: H2 AST enforcement mechanism is unspecified
**The Flaw:** H2 says: *"reject `symbol == "c"/"m"`-style branches"* in `ops`.
**The Reality:** How does the coding agent enforce this? Without a specified mechanism, they might try to write a runtime check (which is impossible since symbols are just data) or guess at the test structure.
**The Fix:** Mandate an **AST source scan** to enforce this statically.
*Patch text for H2:* "Enforced via AST source scan in `ops` tests. The test must parse `ops/*.go` and fail if it finds any string literal equality checks against physical symbol names (e.g., `"c"`, `"m"`, `"v"`, `"E"`, `"p"`). The kernel physics allowlist (`KindMinkowski`, `lorentz_factor`) is verified similarly via AST scan of `internal/kernel`."

---

### 🛠️ Minor Refinements for Zero Ambiguity

1. **Test File Locations:** Explicitly map the AST scans to avoid file-creation violations. 
   * *Patch:* "H1 (import boundaries) and H2 (kernel/ops AST scans) tests reside in `core/object_test.go` (which runs as external `core_test` and can use `go/ast` to scan the repo). No new files are created."
2. **Mutation Testing Clarity:** Clarify the exact mutant for H4.
   * *Patch:* "Mutation for H4: Alter a mechanics constructor (e.g., `NewMass`) to secretly attach `rest_mass_nonnegative`. The H4 test must catch this ambient cross-package assumption injection."

---

### Final Handoff Status
With these 4 patches applied, `plan7.md` transitions from a "conceptually correct governance document" to a **mechanically executable contract**. 

The coding agent will have:
- Exact allowlists (no guessing).
- Exhaustive key sets (no open-ended logic).
- Clear AST enforcement mechanisms (no runtime impossibilities).
- Correct corpus-status categorizations (no HYPOTHESIS/ESTABLISHED confusion).

**Verdict: CLEARED FOR HANDOFF** once the 4 blockers are patched into the text.