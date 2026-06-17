> **ASN-0036 · Strand Model** — condensed claim statements  
> [← Full note](ASN-0036-strand-model.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0036 Claim Statements

*Source: ASN-0036-strand-model.md (revised 2026-05-29) — Extracted: 2026-05-30*

## Σ.C — ContentStore (DEF, definition)

`Σ.C : T ⇀ Val` — the content store is a partial function from tumblers to content values.

`dom(Σ.C) = {a ∈ T : Σ.C(a) is defined}` — the set of I-addresses at which content has been stored.

## Σ.M(d) — Arrangement (DEF, definition)

`Σ.M(d) : T ⇀ T` — the arrangement of document `d` is a partial function from V-position tumblers to I-address tumblers.

- `dom(Σ.M(d)) ⊆ {t ∈ T : zeros(t) = 0 ∧ #t ≥ 2}` — arrangements map only V-positions; every active key is a zero-free tumbler of depth at least 2 (a subspace identifier followed by a within-subspace ordinal).
- `dom(Σ.M(d)) = {v ∈ T : Σ.M(d)(v) is defined}` — the set of V-positions currently active in `d`.
- `ran(Σ.M(d)) = {Σ.M(d)(v) : v ∈ dom(Σ.M(d))}` — the set of I-addresses that `d` currently references.

## Definition — SubspaceProjection

For any tumbler `v` of depth `#v ≥ 1`, define:

`subspace(v) = v₁`

extracting the subspace identifier as the first component of a V-position.

- Signature: `subspace : T → ℕ` — projects the first component of a tumbler.
- Preconditions: `v ∈ T`, `#v ≥ 1`.

## S0 — ContentImmutability (INV, axiom)

For every state transition `Σ → Σ'`:

`[a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a)]`

Formal: `(A a : a ∈ dom(Σ.C) : a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a))`.

Postconditions:
- (a) Domain persistence — `a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C)`.
- (b) Value preservation — `a ∈ dom(Σ.C) ⟹ Σ'.C(a) = Σ.C(a)`.

## S1 — StoreMonotonicity (THM, theorem)

`[dom(Σ.C) ⊆ dom(Σ'.C)]`

Preconditions: State transition `Σ → Σ'` in a system satisfying S0 (content immutability).

Postconditions: `dom(Σ.C) ⊆ dom(Σ'.C)`.

## S2 — ArrangementFunctionality (INV, axiom)

`(A d, v, a₁, a₂ : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a₁ ∧ Σ.M(d)(v) = a₂ : a₁ = a₂)`

Postconditions: (single image) Each V-position has at most one I-address image.

## S3 — ReferentialIntegrity (INV, axiom)

`(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∈ dom(Σ.C))`

Equivalently: `ran(Σ.M(d)) ⊆ dom(Σ.C)`.

Preservation across transitions: For an operation that adds a V-mapping `M(d)(v) = a`, the post-state must satisfy `a ∈ dom(Σ'.C)`.

## S4 — OriginBasedIdentity (THM, theorem)

For I-addresses `a₁`, `a₂` produced by distinct allocation events:

`a₁ ≠ a₂`

regardless of whether `Σ.C(a₁) = Σ.C(a₂)`.

Preconditions: `a₁, a₂ ∈ dom(Σ.C)` produced by distinct allocation events within a system conforming to T10a (allocator discipline, ASN-0034).

Postconditions: `a₁ ≠ a₂`, regardless of whether `Σ.C(a₁) = Σ.C(a₂)`.

## S5 — UnrestrictedSharing (THM, theorem)

`(A N ∈ ℕ :: (E Σ :: Σ is the initial state of a model of S0–S3 ∧ (E a ∈ dom(Σ.C) :: |{(d, v) : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a}| > N)))`

Preconditions: `N ∈ ℕ` arbitrary.

Postconditions: There exists a state `Σ` — the initial state of a model of S0–S3 — such that for some `a ∈ dom(Σ.C)`, `|{(d, v) : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a}| > N`. The construction works both across documents (multiplicity `N + 1` over `N + 1` documents) and within a single document (multiplicity `N + 1` at `N + 1` distinct V-positions).

## S7b — ElementLevelIAddresses (INV, axiom)

`(A a ∈ dom(Σ.C) :: zeros(a) = 3)`

Postconditions: By T4's field correspondence, all four identifying fields — node, user, document, element — are present and the element field exists. The projections `N(a)`, `U(a)`, `D(a)`, `E(a)` supplied by T4b are all well-defined.

## S7a — DocumentScopedAllocation (INV, axiom)

`(A a : a ∈ dom(Σ.C) :: the document-level prefix N(a).0.U(a).0.D(a) is the tumbler of the document whose owner performed the allocation that placed a into dom(C))`

## S7d — DocumentAllocationDiscipline (INV, axiom)

Every document tumbler `d` satisfies `zeros(d) = 2` and is the result of an allocation event under T10a; distinct documents arise from distinct allocation events.

Postconditions: By GlobalUniqueness (ASN-0034), distinct documents have distinct document-level tumblers.

## S7 — StructuralAttribution (THM, theorem)

For every `a ∈ dom(Σ.C)`, define the *origin* as the document-level prefix obtained by truncating the element field:

`origin(a) = N(a).0.U(a).0.D(a)`

Preconditions: `a ∈ dom(Σ.C)` in a system conforming to S7a, S7b, S7d, T4, T4a, T4b, T0, T10a, T10a.4 (ASN-0034).

Postconditions:
- (a) `origin(a)` is well-defined and is a document-level tumbler with `zeros(origin(a)) = 2`.
- (b) `origin(a)` is the tumbler of the document that allocated `a`.
- (c) For `a₁, a₂` allocated under distinct documents, `origin(a₁) ≠ origin(a₂)`.
- (d) `origin(a)` is invariant across all states in which `a ∈ dom(Σ.C)`.

## S8-fin — FiniteArrangement (INV, axiom)

For every state `Σ` and document `d`, `dom(Σ.M(d))` is a finite set.

Postconditions: `|dom(Σ.M(d))| < ∞`. Consequently `ran(Σ.M(d))` is finite.

## S8a — VPositionWellFormedness (INV, axiom)

`dom(Σ.M(d)) ⊆ {t ∈ T : zeros(t) = 0 ∧ #t ≥ 2}`

Per-component form: every active V-position is a zero-free tumbler of depth at least 2 with all components positive.

## S8-depth — FixedDepthVPositions (INV, axiom)

`(A d, u, w : u ∈ dom(Σ.M(d)) ∧ w ∈ dom(Σ.M(d)) ∧ subspace(u) = subspace(w) : #u = #w)`

Postconditions: Within a subspace `s` of document `d`, if `V_s(d) ≠ ∅` then there exists a common depth `m_s ≥ 2` (by S8a) such that every V-position with `v₁ = s` has length `m_s`. Distinct subspaces may have distinct depths.

## OrdShiftHom — OrdinalShiftPreservation (LEMMA, lemma)

For a V-position `v` with `#v = m ≥ 2` and `n ≥ 1`:

- (a) `subspace(shift(v, n)) = subspace(v)`.
- (b) When `v` satisfies S8a, `shift(v, n)` satisfies S8a.

Preconditions: `v ∈ T`, `#v = m ≥ 2`, `n ≥ 1`.

## S8 — CorrespondenceRunPartition (THM, theorem)

For each document `d`, the active V-positions `dom(Σ.M(d))` decompose into finitely many *correspondence runs*. Under the convention `shift(t, 0) := t`, a correspondence run is a triple `(v, a, n)` with `v ∈ dom(M(d))`, `a = M(d)(v)`, and `n ≥ 1`, such that for every `k` with `0 ≤ k < n`:

- (a) **Lockstep displacement** — `shift(v, k) ∈ dom(M(d))` and `M(d)(shift(v, k)) = shift(a, k)`.
- (b) **Well-defined label** — `a = M(d)(v)` exists and is unique because `M(d)` is a function (S2), and `a ∈ dom(Σ.C)` by referential integrity (S3). Each lockstep image `shift(a, k)` (for `0 ≤ k < n`, with the convention `shift(a, 0) := a`) likewise lies in `dom(Σ.C)`.

A run is *maximal* when it admits neither forward extension (no run `(v, a, n+1)`) nor backward extension (no lockstep predecessor `u` with `shift(u, 1) = v`, `u ∈ dom(M(d))`, `shift(M(d)(u), 1) = a`). The maximal runs partition `dom(Σ.M(d))`, and the maximal-run decomposition is unique.

Preconditions: `dom(M(d))` finite (S8-fin); `M(d)` a function (S2); referential integrity (S3); `(A v ∈ dom(M(d)) :: zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))` (S8a); within each subspace, all V-positions share a common depth (S8-depth). Convention: `shift(t, 0) := t`.

Postconditions: `dom(M(d))` is the disjoint union of finitely many maximal correspondence runs `(vⱼ, aⱼ, nⱼ)`: (a) within each run, `shift(vⱼ, k) ∈ dom(M(d))` and `M(d)(shift(vⱼ, k)) = shift(aⱼ, k)` for `0 ≤ k < nⱼ`, with `shift(vⱼ, k)` a well-formed V-position and `shift(aⱼ, k) ∈ dom(Σ.C)`; (b) the label `aⱼ = M(d)(vⱼ)` is well-defined by S2 and lies in `dom(Σ.C)` by S3; (c) the maximal-run decomposition is unique.

## D-CTG — VContiguity (INV, axiom)

`(A d, u, q : u ∈ V_1(d) ∧ q ∈ V_1(d) ∧ u < q : (A v : subspace(v) = 1 ∧ #v = #u ∧ zeros(v) = 0 ∧ u < v < q : v ∈ V_1(d)))`

where `V_1(d) = {v ∈ dom(M(d)) : subspace(v) = 1}`.

Postconditions: V_1(d) is either empty or occupies every position strictly between its extremes (at the fixed depth).

## D-MIN — VMinimumPosition (INV, axiom)

For each document d with V_1(d) non-empty:

`min(V_1(d)) = [1, 1, ..., 1]`

where the tuple has length m (the common depth of V-positions in the text subspace per S8-depth), and every component is 1.

Formal: `V_1(d) ≠ ∅ ⟹ min(V_1(d)) = [1, 1, ..., 1]` of length `m_1`.

## D-CTG-depth — SharedPrefixReduction (LEMMA, lemma)

For depth m ≥ 3, all positions in a non-empty V_1(d) share components 2 through m − 1. Contiguity reduces to contiguity of the last component alone.

Preconditions: V_1(d) non-empty; common depth `m` (S8-depth); `m ≥ 3`.

Postconditions: `(A u, x ∈ V_1(d), j : 2 ≤ j ≤ m − 1 : uⱼ = xⱼ)`. Contiguity of V_1(d) reduces to contiguity of the m-th (last) component.

## D-SEQ — SequentialPositions (THM, theorem)

For each document d, if V_1(d) is non-empty, then there exists n ≥ 1 such that:

`V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n}`

where the tuple has length m, the common V-position depth in the text subspace (S8-depth). By S8a, `m ≥ 2`.

Formal: `(E n : n ≥ 1 : V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n})` where each tuple has length m.

Preconditions: V_1(d) non-empty; common V-position depth m (S8-depth), with `m ≥ 2` inherited from S8a.

## ValidInsertionPosition — ValidInsertionPosition (DEF, predicate)

Binary predicate for document `d` with `V_1(d) ≠ ∅`.

`ValidInsertionPosition(d, v)` holds iff, writing `N = |V_1(d)|`:

`v = min(V_1(d))` or `v = shift(min(V_1(d)), j)` for some `j ∈ {1, ..., N}`.

Preconditions: Document `d` with `V_1(d) ⊆ dom(M(d))` non-empty; D-CTG holds on V_1(d); D-MIN gives `min(V_1(d)) = [1, ..., 1]` and D-SEQ gives `V_1(d) = {[1, ..., 1, k] : 1 ≤ k ≤ N}`; `m ≥ 2` is the common depth of V_1(d) by S8-depth and S8a.

Postconditions:
- (a) `subspace(v) = 1` and `#v = m`.
- (b) `v` satisfies S8a: `zeros(v) = 0` and all components positive.
- (c) For fixed `d`, exactly `N + 1` values of `v` satisfy the predicate.
- (d) The explicit form is `v = [1, 1, ..., 1, 1 + j]` of depth `m`, with last component `1 + j` and all `m − 1` preceding components equal to 1.

## ValidFirstInsertionPosition — ValidFirstInsertionPosition (DEF, predicate)

Ternary predicate for document `d` with `V_1(d) = ∅`.

`ValidFirstInsertionPosition(d, v, m)` holds iff `m ∈ ℕ` with `m ≥ 2` and `v = [1, 1, ..., 1]` of depth `m`.

Preconditions: Document `d` with `V_1(d) = ∅`; `m ∈ ℕ` with `m ≥ 2`.

Postconditions:
- (a) `subspace(v) = 1` and `#v = m`.
- (b) `v` satisfies S8a: `zeros(v) = 0` and all components positive.
- (c) For fixed `d` and `m`, exactly one value of `v` satisfies the predicate.
