# Adversarial Review Result

VERDICT:
PASS WITH FINDINGS

ARCHITECTURAL BLOCKERS:
0

HIGH FINDINGS:
2

MEDIUM FINDINGS:
4

LOW / OBSERVATIONS:
4

## 1. Executive Assessment

The Phase-1 implementation of `phys-artifact`, `phys-math`, and `phys-lib` is **architecturally faithful** to the closed specification (PM-REP-001 v1.2 / plan9.3). No ownership boundary, dependency direction, canonical identity, or physical/math separation violations were found that would block future framework adoption. The 16-family registry, seven-rule closure, ParameterID semantics, condition lifecycle, replay integrity, and protocol packaging all match the closed contracts.

Two **HIGH** findings (framework dependency closure is not transitive in `InClosure`; several durable artifact types lack pre-mint validation) and four **MEDIUM** findings (missing ConditionRefs dedup in `Derivation.ValidateStructure`, latent BigInt slice aliasing in `cloneExpr`, `gr.CheckClosure` lacks transitive cycle detection, and `phys-lib/go.mod` pseudo-version breaks `go list -m all`) represent correctness and integrity weaknesses that do not invalidate the delivered architecture but should be resolved before Phase 2.

## 2. Findings

### H-1 HIGH — Framework `InClosure` is not transitive

Location:
`/home/chaschel/Documents/go/phys-lib/gr/framework.go:156-166`

Claim being tested:
Spec §14.9 requires that a specialized framework MAY reference assumptions/interpretations owned by itself or by **any framework in the transitive `BaseFrameworkRefs` closure**.

Observed behavior:
`gr.FrameworkArtifact.InClosure` checks only the owning namespace and **direct** `BaseFrameworkRefs`. It does not recurse into base frameworks. `fixture2.InClosure` has the identical gap (`/home/chaschel/Documents/go/phys-lib/internal/conformance/fixture2/framework.go:151-161`).

Why it matters architecturally:
A future `phys-bohm` declaring `phys-qm` as a base could not reference `phys-qm` assumptions through the shared closure logic. This breaks the base/specialized dependency contract and forces every future framework to re-implement transitive closure correctly from scratch.

Evidence:
```go
// gr/framework.go:156-166
func (f FrameworkArtifact) InClosure(fam physmath.FamilyID, families map[artifact.ArtifactRef]physmath.FamilyID) bool {
    if fam.Namespace == f.PackageNamespace && isGRFamily(fam) {
        return true
    }
    for _, base := range f.BaseFrameworkRefs {
        if bf, ok := families[base]; ok && fam.Namespace == bf.Namespace {
            return true
        }
    }
    return false
}
```

Recommended resolution:
Implement transitive closure traversal in `InClosure` (or extract a shared helper) so that base-of-base namespaces are admitted. Add a test with a two-level base chain.

---

### H-2 HIGH — Durable artifacts lack `Validate`/pre-mint validation

Location:
`phys-math/families.go` (`Theorem`, `Derivation`, `Equation`, `MathCondition`, `FunctionApplication`, `OperatorApplication` partial) and `phys-lib/gr` family types.

Claim being tested:
Durable artifacts should fail closed on invalid payloads before they enter the content-addressed store.

Observed behavior:
Only a subset of durable types implement `Validate()` (`Function`, `Map`, `Tensor`, `Index`, `Collection`, `OperatorApplication` partial, `Transformation` partial, `EquationSet`, `PhysicalAssumption`, `PhysicalDefinition`, `PhysicalEquation`, `PhysicalTheorem`, `PhysicalInterpretation`, `FrameworkArtifact`). `Theorem`, `Derivation`, `Equation`, `MathCondition`, and `FunctionApplication` have **no** `Validate` method. The equivariance fixture mints `Theorem`, `Derivation`, and `Equation` via raw `store.Put` without any validation gate. A `Theorem` whose `ConclusionRef` targets `expression/1` instead of `relation/1` would be minted silently.

Why it matters architecturally:
Invalid durable payloads can propagate into derivations, theorems, and cross-framework references. Errors surface only later in `TargetChecks` or replay, violating the fail-closed principle.

Evidence:
- `phys-lib/gr/equivariance/fixture.go:212-260` — `store.Put` called for `RelationRef`, `EquationRef`, `TheoremRef`, `DerivationRef` with no `Validate` call.
- `phys-math/families.go` — no `Validate` method on `Theorem`, `Derivation`, `Equation`, `MathCondition`, `FunctionApplication`.

Recommended resolution:
Add `Validate()` methods to all durable types and invoke them before every `store.Put` in production code and fixtures.

---

### M-1 MEDIUM — `Derivation.ValidateStructure` misses duplicate `ConditionRefs`

Location:
`phys-math/families.go:724-749`

Claim being tested:
SET-typed fields must reject duplicate canonical elements per spec §4.9.

Observed behavior:
`ValidateStructure` deduplicates `PremiseRefs` and `ResultRefs`, and checks step-ordinal uniqueness, but does **not** check `DerivationStep.ConditionRefs` for duplicates. A step containing the same condition ref twice passes structural validation, though `DerivationStep.Encode` would later fail because `artifact.EncodeSet` rejects duplicates.

Why it matters architecturally:
Inconsistency between structural validation and encoding creates a gap where malformed derivations are accepted by one gate and rejected by another.

Evidence:
```go
// families.go:724-749
func (d Derivation) ValidateStructure() error {
    if err := checkDuplicateRefs("Derivation.PremiseRefs", d.PremiseRefs); err != nil { ... }
    if err := checkDuplicateRefs("Derivation.ResultRefs", d.ResultRefs); err != nil { ... }
    seen := make(map[uint32]bool, len(d.Steps))
    for _, s := range d.Steps { ... }
    ...
}
```
No `checkDuplicateRefs` call for `s.ConditionRefs`.

Recommended resolution:
Add a duplicate check for `DerivationStep.ConditionRefs` inside `ValidateStructure` (or add `DerivationStep.Validate`).

---

### M-2 MEDIUM — `cloneExpr` shallow-copies `BigInt.Mag` slices

Location:
`phys-math/store.go:162-212` (default case)

Claim being tested:
`cloneExpr` must produce a structurally independent copy to preserve canonical immutability.

Observed behavior:
For `TagRational` and `TagSymbol`, `cloneExpr` executes `cp := e` (shallow struct copy) and only deep-copies `ParamID`/`ConstantID` pointers. The `BigInt.Mag` slice is shared between the original and the clone.

Why it matters architecturally:
If any caller mutates `cloneExpr` output's `Numerator.Mag` or `Denominator.Mag`, the source expression's BigInt is also mutated. Current production code never mutates these slices, so no failure occurs today, but the aliasing violates the immutability expectation of canonical artifacts.

Evidence:
```go
// store.go:198-210
default:
    cp := e
    if e.ParamID != nil {
        p := *e.ParamID
        cp.ParamID = &p
    }
    if e.ConstantID != nil {
        c := *e.ConstantID
        cp.ConstantID = &c
    }
    cp.Numerator = e.Numerator   // shares Mag slice
    cp.Denominator = e.Denominator // shares Mag slice
    return cp
}
```

Recommended resolution:
Deep-copy `Mag` in the default case:
```go
cp.Numerator.Mag = append([]byte(nil), e.Numerator.Mag...)
cp.Denominator.Mag = append([]byte(nil), e.Denominator.Mag...)
```

---

### M-3 MEDIUM — `gr.CheckClosure` lacks transitive cycle detection

Location:
`/home/chaschel/Documents/go/phys-lib/gr/framework.go:135-151`

Claim being tested:
`BaseFrameworkRefs` must form an acyclic directed graph per spec §14.9.

Observed behavior:
`gr.CheckClosure` validates only direct bases: self-reference is rejected, and non-`framework/1` targets are rejected. It does **not** recurse into base frameworks. `fixture2.CheckClosure` (`/home/chaschel/Documents/go/phys-lib/internal/conformance/fixture2/framework.go:92-147`) **does** implement transitive cycle detection.

Why it matters architecturally:
If `gr` ever declares a base framework, a cycle such as `gr -> phys-qm -> gr` would go undetected. Future framework authors might copy `gr.CheckClosure` as a template and inherit the gap.

Evidence:
```go
// gr/framework.go:135-151
func (f FrameworkArtifact) CheckClosure(families map[artifact.ArtifactRef]physmath.FamilyID) error {
    for _, r := range f.BaseFrameworkRefs {
        fam, ok := families[r]
        if !ok { return error }
        if fam.ArtifactType != "framework/1" || fam.SchemaVersion != "1" { return error }
        if fam.Namespace == f.PackageNamespace { return ErrSelfFramework }
    }
    return nil
}
```

Recommended resolution:
Recurse into resolved base frameworks, matching `fixture2.CheckClosure`'s approach.

---

### M-4 MEDIUM — `phys-lib/go.mod` pseudo-version breaks `go list -m all`

Location:
`/home/chaschel/Documents/go/phys-lib/go.mod`

Claim being tested:
Module metadata should be resolvable with standard Go tooling.

Observed behavior:
`go list -m all` fails in all three modules with:
```
go: github.com/PithomLabs/phys-math@v0.0.0: reading github.com/PithomLabs/phys-math/go.mod at revision v0.0.0: ... exit status 128: remote: Repository not found.
```
`go test` succeeds only because the workspace `go.work` overrides the module graph. Outside the workspace (e.g., clean CI), `phys-lib` is unresolvable.

Evidence:
Command output from `go list -m all` in `phys-artifact`, `phys-math`, and `phys-lib`.

Recommended resolution:
Add a `replace` directive in `phys-lib/go.mod`:
```go
replace github.com/PithomLabs/phys-math => ../phys-math
```

---

### L-1 OBSERVATION — `phys-gr/` is not a git repository

Location:
`/home/chaschel/Documents/go/phys-gr/`

Claim being tested:
Legacy preservation should be independently verifiable via tree-hash oracle.

Observed behavior:
`phys-gr/` contains no `.git` directory. `git status` cannot be run. The directory exists and its files appear untouched, but independent tree-hash verification is impossible.

Evidence:
`ls -la /home/chaschel/Documents/go/phys-gr/.git` → `no .git`

Recommended resolution:
Initialize a git repository in `phys-gr/` or maintain committed tree-hash manifests outside version control.

---

### L-2 OBSERVATION — `ArtifactRef` fields are exported and unvalidated in memory

Location:
`/home/chaschel/Documents/go/phys-artifact/artifact.go:13-18`

Claim being tested:
Identity-bearing structs should be opaque or validated on construction.

Observed behavior:
`ArtifactRef` is a plain struct with exported fields. Any caller can write `ArtifactRef{Namespace: "INVALID!", ...}` and use it as a map key, encode it, or pass it to rules. Only `ComputeRef` and `DecodeArtifactRef` validate grammars.

Evidence:
Code inspection.

Recommended resolution:
Make fields unexported and provide a constructor, or document that in-memory refs are not guaranteed canonical.

---

### L-3 OBSERVATION — Dummy `var _` assignments in test files

Location:
`/home/chaschel/Documents/go/phys-math/rules_test.go:393`, `/home/chaschel/Documents/go/phys-math/identity_test.go:134`

Claim being test files should be clean.

Observed behavior:
`var _ = big.NewInt` and `var _ = bytes.Equal` suppress unused-import errors. Harmless but indicates leftover import hygiene.

Evidence:
Code inspection.

Recommended resolution:
Remove the dummy assignments and any truly unused imports.

---

### L-4 OBSERVATION — `TestSimplifyPowZeroCondition` generates expected condition bytes via the implementation

Location:
`/home/chaschel/Documents/go/phys-math/rules_test.go:160`

Claim being tested:
Golden vectors should be independently derived, not produced by the code under test.

Observed behavior:
The test constructs the exact `NEQ(x,0)` condition using `physmath.EncodeRelationBody` — the same function the rule uses for matching. The success path is therefore circular, though the test correctly verifies that a wrong-variable condition fails.

Evidence:
Code inspection.

Recommended resolution:
Hardcode the expected condition payload bytes as a constant derived from the spec's canonical form.

## 3. Claims Independently Verified

- "20 gates green" — PARTIALLY VERIFIED (all existing tests pass: phys-artifact 9, phys-math 35, phys-lib 14+; exact gate count not enumerated in code)
- "legacy hashes unchanged" — PARTIALLY VERIFIED (`phys/` git status shows no tracked-file modifications; `phys-gr/` lacks `.git` so tree-hash oracle cannot be run)
- "migration completed" — VERIFIED (`go.work` includes `./phys-artifact`, `./phys-math`, `./phys-lib`; excludes `./phys-gr`; `phys/` untouched)
- "core is intentionally empty" — VERIFIED (`core/doc.go` is the only file; `TestCoreEmpty` asserts zero exported identifiers)
- "no accidental stubs for future theories" — VERIFIED (no `bohm`, `qm`, `cst`, `cdt`, or other theory directories exist)
- "phys-gr is no longer required by the new implementation" — VERIFIED (`go list -deps` shows zero `phys-gr` imports in phys-artifact, phys-math, or phys-lib production code)
- "no hidden local replacements masking dependencies" — VERIFIED (no `replace` directives in any `go.mod`; `go.work` is plain)
- "dependency direction is acyclic" — VERIFIED (phys-artifact → stdlib; phys-math → phys-artifact; phys-lib/gr → phys-math + phys-artifact; no reverse edges)
- "determinism" — VERIFIED (5 consecutive `go test -count=1` runs produce identical output; golden digests pinned in `golden_test.go`)

## 4. Architectural Invariants Checked

- PhysMath purity — VERIFIED
- Artifact identity — VERIFIED
- Canonical serialization — VERIFIED
- Seven-rule closure — VERIFIED
- Condition lifecycle — VERIFIED
- Replay integrity — VERIFIED
- Core emptiness — VERIFIED
- Classification governance — VERIFIED
- GR boundary — VERIFIED
- Projection boundary — VERIFIED
- Framework independence — VERIFIED
- Protocol integrity — VERIFIED
- Dependency isolation — VERIFIED
- Determinism — VERIFIED
- Legacy preservation — PARTIALLY VERIFIED

## 5. Adversarial Tests Added/Executed

No production code was modified. The following existing tests were executed repeatedly and inspected for self-validation weaknesses:

```text
go test -count=1 -v ./...
go test -count=1 -v ./...   (×5 consecutive runs for determinism)
go vet ./...
go list -deps ./...
```

Code-level adversarial attacks performed via inspection:
- `ArtifactRef` construction bypass (fields exported)
- `BigInt` leading-zero / negative-zero vectors (`TestBigIntVectors`)
- Unknown expression tag rejection (`TestUnknownTagRejected`)
- Relation payload vs node-tag confusion (`TestRelationSerialization`)
- EquationSet sequence preservation (`TestEquationSetSequenceUniqueness`)
- Duplicate ParameterID substitution rejection (`TestSubstitute`)
- `Pow(x,0)` fail-closed condition semantics (`TestSimplifyPowZeroCondition`)
- Replay mutation negatives (`TestMutationNegatives`)
- Framework isolation (`TestIsolation`, `TestNoLegacyOrFrozenImports`)
- Category ownership (`TestCategoryClosed`)

## 6. Final Recommendation

**PASS WITH FINDINGS.** The implementation realizes the closed Phase-1 architecture without architectural blockers. The two HIGH findings (non-transitive `InClosure`, missing durable-artifact validation) and four MEDIUM findings should be addressed before Phase 2 to harden the foundation for multi-framework adoption. No redesign is required; the specified fixes are localized and do not alter the mathematical identity or dependency contracts.
