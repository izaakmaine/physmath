Here is the prompt I would give the coding/review agent. It is designed to be **adversarial against the implementation actually produced**, not a generic Go code review, and to concentrate on architectural/semantic blockers rather than style or minor defects.

---

# Adversarial Code Review — `phys-artifact` + `phys-math` + `phys-lib`

You are performing a **hostile architectural/code review** of the Phase-1 implementation that has now been delivered.

Do **not** assume the implementation report is correct merely because tests are green. Independently inspect the code, tests, module graph, schemas, codecs, fixtures, and repository state, and attempt to falsify the claimed architecture.

## Authoritative sources

Treat these as the authority, in this order:

1. `PHYSMATH_PHYS_PHYSSTAR_REPRESENTATION_SPEC_v1.2.md` (`PM-REP-001`)
2. `/home/chaschel/Documents/go/phys-artifact`
3. `/home/chaschel/Documents/go/phys-math`
4. `/home/chaschel/Documents/go/phys-lib`
5. `plan9.3.md`, whose implementation decisions and Phase-1 boundaries are already closed
6. Historical `/home/chaschel/Documents/go/phys-gr`, which is **reference/oracle only and must not be modified**

The implementation report claims Phases 0–10 completed, 20 gates green, legacy hashes unchanged, and migration completed. **Verify those claims independently.**

Do not silently reinterpret, weaken, or replace the specification with “reasonable Go design.”

---

# Review objective

Determine whether the implementation is **architecturally faithful**, not merely whether it compiles.

The central question is:

> Could a future framework such as Bohmian mechanics, causal set theory, or CDT use this foundation without importing GR assumptions into `phys-math`, without creating reverse dependencies, and without changing mathematical identity semantics?

Try hard to prove that the answer is **no**.

Focus on architectural blockers involving:

- ownership
- dependency direction
- mathematical purity
- physical leakage
- canonical identity
- canonical serialization
- durable-family boundaries
- rule semantics
- condition semantics
- replay semantics
- framework isolation
- protocol integrity
- preservation boundaries

Do **not** spend review budget on formatting, naming preferences, micro-optimizations, idiomatic-vs-nonidiomatic Go, or minor refactoring opportunities unless they conceal an architectural problem.

---

# 1. Establish the actual implementation boundary

First inspect the complete repository topology:

```text
/home/chaschel/Documents/go/
├── go.work
├── phys/
├── phys-gr/
├── phys-artifact/
├── phys-math/
└── phys-lib/
```

Verify:

- `phys/` is untouched.
- historical `phys-gr/` is untouched.
- new code exists only in the three canonical modules.
- `phys-lib` is exactly one Go module.
- `core` is still intentionally empty.
- there are no accidental package/module stubs for future theories.
- `phys-gr` is no longer required by the new implementation.
- there are no hidden local replacements or workspace tricks masking dependencies.

Independently run:

```text
go work edit
go list -deps ./...
go mod graph
go test ./...
go vet ./...
```

as appropriate per module/workspace.

Inspect both source imports and module-level dependency edges.

---

# 2. Attack `phys-artifact`

Review `phys-artifact` as the **lowest-level identity/representation layer**.

Verify that it contains only mechanisms justified by the specification:

- `ArtifactRef`
- digest primitives
- `BigInt`
- primitive encodings
- canonical framing/grammar
- related validation

### Identity and encoding attacks

Attempt to falsify:

1. Is `ArtifactRef` truly content-addressed?
2. Is canonical serialization injective over the intended representation?
3. Are there multiple byte encodings for the same semantic value?
4. Can malformed encodings normalize into valid alternative identities?
5. Are leading zeros rejected where required?
6. Is negative zero impossible?
7. Is BigInt zero encoded exactly as required?
8. Are lengths/framing unambiguous?
9. Are truncated, appended, reordered, or extra bytes rejected?
10. Can decode → encode change identity?
11. Can encode → decode → encode produce different bytes?
12. Are digest and reference construction consistent everywhere?
13. Can user-controlled text accidentally enter identity where it should not?
14. Are there generic helpers that make it easy for upper layers to accidentally bypass canonical encoding?

Look especially for:

- permissive decoders
- alternate constructors
- convenience paths
- JSON-derived identity
- string-based identity
- implicit normalization
- "equivalent" values with different canonical bytes

### Adversarial tests

Create temporary review-only tests where useful. Do not modify the architecture to make them pass.

Try:

- leading-zero integers
- negative zero
- empty/truncated frames
- unknown tags
- trailing bytes
- malformed lengths
- nested malformed values
- encode/decode round trips
- two semantically equal values with intentionally different construction paths

If a failure reveals a real architectural problem, report it.

---

# 3. Attack `phys-math` purity

`phys-math` is the pure mathematical layer.

Verify that no physical semantics have leaked in.

The closed expression AST is exactly ten nodes:

```text
0x01 Symbol
0x02 Rational
0x03 Add
0x04 Mul
0x05 Neg
0x06 Pow
0x07 Sqrt
0x08 Call
0x09 Relation
0x0A BranchSet
```

Unknown tags must fail closed.

Verify that no code, exported API, durable schema, comments used as contract, helper abstraction, or validation path smuggles in:

- GR concepts
- coordinate systems
- metrics
- curvature
- physical dimensions/units as physical meaning
- GR conventions
- quantum interpretation
- group-action ontology
- `PhysicalContract`
- trust/provenance/corpus semantics
- theory-specific functions
- theory-specific tensor operations

Do not merely grep names. Inspect semantic behavior.

The critical test:

> Could a mathematician explain the exported `phys-math` API without knowing that GR exists?

If no, report the exact leakage.

Also verify the opposite direction:

> Could `phys-lib/gr` express GR semantics without modifying or extending the `phys-math` AST?

---

# 4. Attack PhysMath identity semantics

Audit all identity-bearing mathematical structures.

Verify exactly:

```text
ParameterID =
SHA-256(
    String("physmath")
    || Bytes(ScopePathBytes)
    || UInt32BE(ParameterOrdinal)
)
```

For Function parameters in Phase 1:

- ScopePath = empty
- depth = 0
- ScopePathBytes = `UInt32BE(0)`
- ordinal is zero-based

Verify:

- DisplayName does not participate in PID derivation.
- DisplayName remains identity-bearing where specified in Symbol payload.
- `BoundParameterRef` has exactly the required layout.
- no extra wrapper is introduced around the 32-byte PID.
- alpha-freshening is not silently occurring.
- capture rewriting is not silently occurring.
- ordinal changes alter identity.
- renaming has exactly the identity consequences specified by the contract.
- two distinct declaration contexts cannot accidentally collapse.

Try to construct collisions through:

- different declaration artifacts
- same ordinal
- same displayed names
- reordered parameters
- renamed parameters
- empty/non-empty scope paths
- different Function artifacts with otherwise similar bodies

Verify whether the implementation behaves according to the spec, not according to intuitive variable semantics.

---

# 5. Attack the 16 durable PhysMath families

Verify the exact closed registry:

```text
physmath/expression/1
physmath/function/1
physmath/function_application/1
physmath/map/1
physmath/tensor/1
physmath/index/1
physmath/collection/1
physmath/operator_application/1
physmath/transformation/1
physmath/relation/1
physmath/equation/1
physmath/equation_set/1
physmath/definition/1
physmath/condition/1
physmath/derivation/1
physmath/theorem/1
```

Attack:

- accidental extra durable families
- missing validators
- validators that are dead code
- validators that validate only tests' happy paths
- recursive/cyclic schemas
- self-referential artifacts
- cross-family references that should be forbidden
- incorrect namespace ownership
- schema/version mismatches
- family validators accepting malformed references

Specifically verify:

### EquationSet

It is a **SEQUENCE**, not a SET.

Try:

- duplicate entries
- reordered entries
- semantically identical entries with different artifact refs
- empty sequence
- malformed members

### Relation

Verify:

- expression-node framing includes the outer `0x09`
- durable relation payload omits the outer tag
- embedded sides are byte-identical to standalone expression artifacts
- no alternate relation encoding exists

---

# 6. Attack the seven-rule closure

The allowed rule pairs are exactly:

```text
(physmath, EXPR-CANON-001)
(physmath, EXPR-SIMPLIFY-001)
(physmath, EXPR-SUBSTITUTE-001)
(physmath, RAT-NORMALIZE-001)
(physmath, DIFFERENTIATE-001)
(physmath, SELECT-BRANCH-001)
(physmath, EXPR-APPLY-001)
```

Verify:

- canonical identity is the pair `(ProducerNamespace, RuleID)`
- colon-joined strings are never used as canonical identity
- unknown pairs fail closed

Then attack every rule individually.

## CANON

Verify idempotence and canonical stability.

## SIMPLIFY

Verify:

- rational folding happens only when **all current children are Rational**
- `Pow(x,0)` succeeds only with the exact supplied `NEQ(x,0)` condition
- rule never invents conditions
- no hidden theorem proving is occurring
- no algebraic transformations have slipped into the rule beyond the contract

## SUBSTITUTE

Verify:

- exact `1 + D` arity
- declaring Functions sorted by ArtifactRef bytes
- entries sorted by canonical entry bytes
- all target Parameters declared
- **no two entries share a ParameterID regardless of DeclaringArtifactRef**
- no hidden alpha-freshening
- no renaming allocator
- no capture-rewrite machinery

Attack with:

- same PID under different declaration refs
- duplicate substitutions
- reordered declarations
- reordered entries
- undeclared parameters
- mismatched declaration artifacts

## APPLY

Verify:

- exactly two inputs
- first input is a root Call
- referenced Function validation is exact
- function body is applied capture-free by construction
- nested Call cases obey the root-Call-only rule
- no accidental support for forbidden recursive/nested application behavior
- no alpha-freshening

## DIFFERENTIATE

Verify:

- exactly two inputs
- expression + declaring Function
- scalar scope only
- order >= 1
- nonnegative integer exponent behavior
- Call/Relation/BranchSet differentiation remains out of scope
- no invented derivative AST node
- no implicit expansion beyond contract

## SELECT

Attack branch/range validation and failure behavior.

## RAT-NORMALIZE

Attack canonical rational reduction, signs, zero, denominator behavior, and malformed values.

---

# 7. Attack condition semantics

This is a particularly important boundary.

The implementation must obey:

```text
condition/1 is durable
        ↓
fixture/framework materializes condition
        ↓
OperationInvocation.ConditionRefs supplies it
        ↓
rule checks exact supplied condition
        ↓
success/fail closed
        ↓
TransformResult.ConditionRefs returns normalized supplied set
```

Rules generate **zero durable conditions**.

Verify that nowhere in the executor or rule implementation does a rule:

- mint a new durable condition
- silently discharge a condition
- synthesize a missing predicate
- broaden condition semantics
- mutate supplied conditions

Attack `Pow(x,0)` and any other guarded rule with:

- absent condition
- wrong condition
- equivalent-but-different condition artifact
- extra unrelated condition
- reordered conditions
- duplicate conditions
- foreign-family condition refs

Determine whether matching is truly based on the required canonical identity/bytes.

---

# 8. Attack `TransformResult` and execution integrity

Inspect the actual executor/store implementation.

Verify:

- content-addressed execution
- immutable `TransformResult`
- normalized supplied condition threading
- no durable artifacts created as an implicit side effect
- no hidden global state
- no rule-specific backchannels
- no mutable shared result state
- no executor path bypassing rule validation

Try to break the executor with:

- foreign input bytes
- foreign family refs
- future/unknown rule IDs
- malformed artifacts
- dangling refs
- incorrect arity
- mismatched producer namespace
- tampered outputs
- replay against altered store contents

Look for circular validation such as:

> "the validator accepts this because the executor produced it, and the executor accepts it because the validator accepts it."

Any self-trust loop is a finding.

---

# 9. Attack derivation replay

Replay must verify semantics, not merely replay stored outputs.

Verify:

- ordered rule sequence
- rule-family validation
- exact output-reference equality
- condition-context equality
- result carry-through
- correct final designated output
- no skipped transformation
- no hidden step insertion
- no omitted step
- no replacement with semantically similar artifact

Mutate:

- one rule ID
- one input ref
- one output ref
- one condition ref
- step ordering
- step count
- family
- namespace
- intermediate artifact
- final artifact

The replay verifier must reject each invalid mutation for the correct reason.

Also check whether the "golden" expected outputs are genuinely independent of the implementation under test. A test generated from the same implementation cannot establish correctness.

---

# 10. Attack `phys-lib/core`

`core` must remain empty in Phase 1.

Verify:

```text
core/doc.go
```

is genuinely just the package declaration and documentation, with zero exported domain identifiers.

Attack for accidental abstraction leakage:

- helper types
- interfaces
- "temporary" shared structs
- framework-neutral aliases that are actually physical
- generic utilities that create a dumping ground
- GR abstractions promoted for convenience
- test-only exported contracts

Also verify classification governance exists independently of the empty core.

---

# 11. Attack the classification mechanism

Verify every significant legacy abstraction is classified according to the required mechanism.

Classification records must:

- exist at `phys-lib/classifications/<slug>.md`
- contain required front matter
- contain required evidence
- explicitly distinguish `CORE`, `SPLIT`, `GR`, `REPLACE`, `DELETE`, `DEFER`, etc.
- not merely restate conclusions without evidence

Attack the CI/governance implementation for loopholes:

- Can a missing record pass?
- Can a CORE record have empty evidence?
- Can a core export slip through despite the scanner?
- Can a record point to an unrecognized source row?
- Can classification be bypassed through a new file/path?

---

# 12. Attack `phys-lib/gr` boundary

Verify `phys-lib/gr` owns GR-specific meaning.

GR-local algebra must remain disjoint from `phys-math.Expression`.

Verify:

- `Sin`
- `Cos`
- unknown-function shape
- trig simplification
- regime-sensitive zero testing
- GR derivative caps/local derivative behavior

remain GR-local where required.

Critically verify there are **no synthetic PhysMath identities** such as:

```text
phys-gr:func/sin
phys-gr:func/cos
```

and no shared interface that makes GR algebra masquerade as PhysMath algebra.

`ToPhysMath()` must be:

- explicit
- deterministic
- fallible
- limited to the representable subset

Attack it with nonrepresentable GR expressions and ensure it returns the dedicated failure rather than approximating, inventing identities, or silently changing meaning.

---

# 13. Attack GR physical-family semantics

Verify the six framework-owned families and `FrameworkArtifact` have exactly the intended ownership.

Pay particular attention to:

- framework identity
- package namespace
- opaque framework version
- base framework refs as artifact/data dependencies only
- no Go framework-to-framework coupling

Attack:

- invalid namespace
- wrong framework identity
- unrelated namespace references
- self references
- cycles
- malformed bases
- hidden cross-framework imports

### Category

Verify that:

```text
convention
regularity
model
```

are constrained **only by GR validators**.

There must be no global validator in `phys-artifact` or `phys-math` that assumes those are the universal categories of physical frameworks.

A hypothetical fixture2 category such as:

```text
test-postulate
```

must remain legal outside GR.

---

# 14. Attack the D2 assumption bridge

Verify the exact boundary:

```text
PhysicalAssumption
       ↕
existing physmath/condition/1
       ↕
OperationInvocation.ConditionRefs
```

Check that:

- framework assumptions can reference existing mathematical conditions
- non-PhysMath-expressible assumptions do not pretend to be mathematical conditions
- convention assumptions can correctly carry empty `ConditionRefs`
- descriptions are identity-bearing where required
- rules do not create the conditions

Look specifically for semantic laundering where a physical assumption is smuggled into `phys-math` through generic-looking structures.

---

# 15. Attack the equivariance fixture

Do not confuse this fixture with a universal PhysMath `Equivariance` ontology.

Verify that the fixture remains a framework-level theorem/protocol instance built out of generic PhysMath structures.

Exact realization:

```text
f(v)        = 2v
rho_X(g,v)  = g·v
rho_Y(g,v)  = g·v
```

Verify exact staged structure and six steps:

```text
0 APPLY inner-rho_X
1 APPLY staged LHS
2 CANON
3 APPLY inner-f
4 APPLY staged RHS
5 CANON
```

Verify byte-linkage between staged calls and previous outputs.

Verify:

- Function-declared ParameterIDs are used
- no bare `NewSymbol("g")` / `NewSymbol("x")`
- no Map-for-f
- no durable `FunctionApplication` substitution where not required
- no GroupAction ontology
- no distributivity theorem smuggled into the fixture
- no trig
- no semantic confusion caused by the intentional `rho_X` / `rho_Y` canonical coincidence

Most importantly:

> Verify that the fixture demonstrates protocol/representation capability rather than accidentally establishing an "equivariance" ontology inside `phys-math`.

---

# 16. Attack fixture2 framework independence

`fixture2` must be a **fully conforming independent framework**, not a fake test double.

Verify:

```text
phys-fixture2
```

has:

- FrameworkArtifact
- all six families
- independent namespace
- own framework identity
- different framework-owned meanings
- no import of `gr`
- no source-code dependency on GR implementation
- reuse of PhysMath only

Rebuild its final mathematical artifact independently.

Then verify the claimed property:

> same mathematical artifact bytes, different framework identity/interpretation, no shared framework code.

Attack this by searching for:

- GR helper calls
- GR package imports
- copied GR semantic types
- GR-specific assumptions hidden under generic names
- test-only imports that violate isolation
- framework-specific validators accidentally placed in shared packages

---

# 17. Attack the protocol package

Inspect:

```text
phys-lib/gr/equivariance/
    AGENTS.md
    manifest.json
    protocol/
        derivation_test.go
```

Verify that the actual GR protocol precedent was followed rather than an invented second protocol.

Check:

- strict decoder
- unknown-field rejection
- required-field/presence validation
- manifest schema semantics
- metadata-vs-premise firewall
- ordered derivation expectations
- firewalls
- mutation coverage
- independence
- final conclusion gate

The canonical fixture is allowed to use the existing GR protocol semantics, but it must not import the historical `phys-gr` implementation.

Try to find protocol checks that merely inspect hard-coded expected values instead of validating actual semantic relationships.

---

# 18. Attack module dependency architecture

The permitted direction is:

```text
phys-lib/<specific>
        ↓
      core
        ↓
    phys-math
        ↓
  phys-artifact
```

with direct:

```text
gr → phys-math
gr → phys-artifact
```

permitted.

Verify there is:

- no `phys-artifact → anything`
- no `phys-math → phys-lib`
- no `core → framework`
- no framework → framework Go imports
- no new module → `phys-gr`
- no new module → frozen `phys`

Also look for indirect coupling through:

- test imports
- generated code
- build tags
- internal packages
- code generation
- shared test helpers
- `go:linkname`
- reflection-based registration
- filesystem/runtime discovery

---

# 19. Attack determinism claims

The implementation claims Phase-1 determinism.

Verify repeated execution from clean state.

At minimum test:

```text
same input
same rule
same invocation
same conditions
multiple executions
```

must produce identical canonical outputs.

Repeat enough times to catch:

- map iteration dependence
- unstable ordering
- randomized IDs
- pointer/address dependence
- timestamp dependence
- filesystem-order dependence
- process-global state

Also inspect whether tests themselves accidentally normalize away nondeterminism.

Do not demand two independent implementations; that remains deferred.

---

# 20. Attack test independence

This is critical.

For every major golden vector, determine:

> Was this expected value independently derived, or was it generated by the implementation under test?

Look for:

- golden generation utilities reused by production code
- tests calling the same canonicalizer to construct their expected values
- tests using production constructors to generate both input and expected output
- self-validating fixtures
- circular helper dependencies

A test that merely reproduces the implementation's behavior is weak evidence.

Flag such cases even if the behavior is currently correct.

---

# 21. Attack the preservation boundary

Independently verify the claimed legacy preservation.

Check:

```text
phys/
phys-gr/
```

against the Phase-0 tree-hash oracles.

Verify:

- no source modifications
- no generated files added
- no metadata modifications
- no permissions/renames that alter the tree
- no compatibility shims added inside legacy repositories

Also verify that removing `./phys-gr` from `go.work` did not accidentally turn preservation into deletion.

---

# 22. Attempt architectural exploitation

Try deliberately to break the architecture by proposing hypothetical future frameworks:

### Hypothetical Bohmian package

Ask:

> Can a future `phys-lib/bohm` represent Bohmian equivariance without adding Bohmian concepts to `phys-math`?

### Hypothetical causal-set package

Ask:

> Can it reuse PhysMath mathematical structures without depending on GR?

### Hypothetical CDT package

Ask:

> Can it introduce its physical semantics without requiring a shared "physics core" abstraction?

### Hypothetical second math realization

Ask:

> Can another framework interpret the same PhysMath artifact differently without changing the artifact's identity?

If the answer requires modifying `phys-math` merely to support a physical theory, identify the exact architectural leak.

---

# 23. Review findings classification

Every finding must be classified:

```text
BLOCKER
HIGH
MEDIUM
LOW
OBSERVATION
```

Use this meaning:

### BLOCKER

A violation of:

- ownership boundary
- dependency direction
- canonical identity
- canonical serialization
- durable schema
- rule semantics
- physical/math separation
- framework isolation
- preservation boundary
- Phase-1 scope

or a correctness flaw that would make future framework implementations architecturally unstable.

### HIGH

A serious semantic bug or integrity weakness that does not yet invalidate the architecture but could produce incorrect artifacts or false conformance.

### MEDIUM

Meaningful correctness/test-coverage weakness with a localized fix.

### LOW / OBSERVATION

Minor issues, readability, maintainability, or optional improvements.

Do **not** inflate stylistic issues into blockers.

---

# 24. Required final report

Return the report in exactly this structure:

```text
# Adversarial Review Result

VERDICT:
[PASS / PASS WITH FINDINGS / BLOCKED]

ARCHITECTURAL BLOCKERS:
[number]

HIGH FINDINGS:
[number]

MEDIUM FINDINGS:
[number]

LOW / OBSERVATIONS:
[number]

## 1. Executive Assessment
[brief conclusion]

## 2. Findings

### [ID] [SEVERITY] — [short title]

Location:
[exact file + symbol/function/line if possible]

Claim being tested:
[...]

Observed behavior:
[...]

Why it matters architecturally:
[...]

Evidence:
[tests, code path, command output, or reproduction]

Recommended resolution:
[...]

### ...
```

Then include:

```text
## 3. Claims Independently Verified

- [claim] — VERIFIED / NOT VERIFIED
- [claim] — VERIFIED / NOT VERIFIED
...

## 4. Architectural Invariants Checked

- PhysMath purity
- Artifact identity
- Canonical serialization
- Seven-rule closure
- Condition lifecycle
- Replay integrity
- Core emptiness
- Classification governance
- GR boundary
- Projection boundary
- Framework independence
- Protocol integrity
- Dependency isolation
- Determinism
- Legacy preservation

## 5. Adversarial Tests Added/Executed

[commands and temporary or permanent review tests]

## 6. Final Recommendation
```

---

# Important review discipline

**Do not modify production code as part of this review.**

You may create temporary review tests or scripts needed to falsify claims, but clearly identify them and clean them up unless they expose a genuine permanent regression test that should be retained.

Do not start another planning document.

Do not redesign the architecture merely because you prefer a different implementation.

Do not treat the existence of green tests as proof of architectural correctness.

Do not report generic "could improve" advice.

The goal is to answer one question:

> **Does the implementation actually realize the closed Phase-1 architecture, or has the code introduced a hidden semantic/ownership/identity/dependency loophole that the existing tests fail to expose?**

If there is a genuine architectural blocker, provide the smallest concrete reproduction and stop making speculative changes.