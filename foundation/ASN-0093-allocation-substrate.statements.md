> **ASN-0093 · Allocation Substrate** — condensed claim statements  
> [← Full note](ASN-0093-allocation-substrate.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0093 Claim Statements

*Source: ASN-0093-allocation-substrate.md (revised unknown) — Extracted: 2026-05-31*

## M0 — DocumentTumblerWellFormed (INV, predicate)

`(A d ∈ dom(M) :: T4-valid(d) ∧ zeros(d) = 2)`

Every allocated document address is a T4-valid tumbler (per T4, HierarchicalParsing, ASN-0034) with exactly two zero components (i.e., a document-level address per S7d of ASN-0036).

## M1 — ArrangementMonotonicity (INV, predicate)

`(A Σ → Σ' :: dom(M) ⊆ dom(M'))`

`dom(M)` is non-decreasing across all transitions. The substrate admits no transition that removes a document from `dom(M)`.

## M2 — EmptyArrangement (INV, predicate)

`(A d ∈ dom(M) :: M(d) = ∅)`

Every allocated document carries the empty arrangement.

## C0 — ContentImmutability (INV, predicate)

`(A Σ → Σ' :: dom(C) ⊆ dom(C') ∧ (A a : a ∈ dom(C) : C'(a) = C(a)))`

Append-only with immutable values: `dom(C)` is non-decreasing, and no transition alters the value bound to an existing key.

## C1 — ContentElementLevel (INV, predicate)

`(A a ∈ dom(C) :: zeros(a) = 3)`

Every content address is an element-level tumbler.

## C1b — ContentElementFieldDepth (INV, predicate)

`(A a ∈ dom(C) :: #E(a) ≥ 2)`

Every content address has at least two element-field components.

## C1c — ContentAllocatorConformance (INV, predicate)

Every content address `a ∈ dom(C)` has a T10a-conforming step sequence from its home document to `a`: a finite sequence `(t₀, t₁, …, tₙ)` with `n ≥ 1`, `t₀ = origin(a)`, and `tₙ = a`, where each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfies T10a's per-step admissibility constraints; additionally, `k₁ = 2` (the first step is a depth-2 increment off the document seed) and `(A i : 1 ≤ i ≤ n : #tᵢ > #origin(a))` (every intermediate length strictly exceeds the seed's).

## C2 — ContentScopedAllocation (INV, predicate)

`(A a ∈ dom(C) :: origin(a) ∈ dom(M))`

Every content address has its home document allocated.

## L0 — SubspacePartition (INV, predicate)

`(A a ∈ dom(L) :: E(a)₁ = s_L)`
`(A a ∈ dom(C) :: E(a)₁ = s_C)`

Every link address has subspace identifier `s_L`; every content address has subspace identifier `s_C`.

## L1 — LinkElementLevel (INV, predicate)

`(A a ∈ dom(L) :: zeros(a) = 3)`

Every link address is an element-level tumbler.

## L1a — LinkScopedAllocation (INV, predicate)

`(A a ∈ dom(L) :: origin(a) ∈ dom(M))`

Every link address has its home document allocated.

## L1b — LinkElementFieldDepth (INV, predicate)

`(A a ∈ dom(L) :: #E(a) ≥ 2)`

Every link address has at least two element-field components.

## L1c — LinkAllocatorConformance (INV, predicate)

Every link address `ℓ ∈ dom(L)` has a *T10a-conforming step sequence* from its home document to `ℓ`: a finite sequence `(t₀, t₁, …, tₙ)` with `n ≥ 1`, `t₀ = origin(ℓ)`, and `tₙ = ℓ`, where each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfies T10a's per-step admissibility constraints; additionally, `k₁ = 2` (the first step is a depth-2 increment off the document seed) and `(A i : 1 ≤ i ≤ n : #tᵢ > #origin(ℓ))` (every intermediate length strictly exceeds the seed's).

## L3 — NEndsetStructure (INV, predicate)

`(A a ∈ dom(L) :: |L(a)| ≥ 3 ∧ (A i : 1 ≤ i ≤ |L(a)| : L(a).eᵢ ∈ Endset) ∧ L(a).e₃ ≠ ∅)`

Every link is a sequence of at least three endsets, with the type endset (slot 3) non-empty.

## L12 — LinkImmutability (INV, predicate)

`(A Σ → Σ' : (A a : a ∈ dom(L) : a ∈ dom(L') ∧ L'(a) = L(a)))`

Once allocated, a link's address persists in `dom(L)` and its value is permanently fixed across all transitions.

## SD — StoreDisjointness (INV, predicate)

`dom(C) ∩ dom(L) = ∅`

Derived from L0 + SC-NEQ + StoreT4Validity + T7 (SubspaceDisjointness, ASN-0034). T7's preconditions are discharged on each side: `zeros(·) = 3` from C1 (content) and L1 (links), and T4-validity from StoreT4Validity. With both premises met, every content address has `E(·)₁ = s_C` and every link address has `E(·)₁ = s_L` (L0), and `s_C ≠ s_L` (SC-NEQ), so T7 gives, for every `a ∈ dom(C)` and every `ℓ ∈ dom(L)`, `a ≠ ℓ` — which is exactly `dom(C) ∩ dom(L) = ∅`.

## L-fin — LinkStoreFiniteness (INV, predicate)

`|dom(L)| < ∞`

The link store is finite at every reachable state.

## C-fin — ContentStoreFiniteness (INV, predicate)

`|dom(C)| < ∞`

The content store is finite at every reachable state.

## ChainDiscipline — ContentLinkSubAllocatorChainDiscipline (LEMMA, lemma)

Each sub-allocator chain is an instance of ASN-0040's `SiblingStream`. Writing `S(p, k)` for the stream `c₁ = inc(p, k)`, `cₙ₊₁ = inc(cₙ, 0)` (SiblingStream, ASN-0040):

`A_C(d) = S(b_C(d), 1)`  and  `A_L(d) = S(b_L(d), 1)`,

since each chain's first emission is `inc(anchor, 1)` and successive elements advance by `inc(·, 0)`, coinciding by construction with the SiblingStream recurrence at `p = b_·(d)`, `k = 1`.

## ChainElementT4Validity — ChainElementT4Validity (LEMMA, lemma)

Every element of `A_C(d)` (resp. `A_L(d)`) is T4-valid.

Source: ASN-0040 B6(a) (ValidDepth sufficiency) — for `B6`-valid `(p, d)`, every `cₙ ∈ S(p, d)` satisfies T4.

## ChainEnumerationInjectivity — ChainEnumerationInjectivity (LEMMA, lemma)

The enumeration of `A_C(d)` (resp. `A_L(d)`) is strictly increasing under T1, `m < n ⟹ t_m < t_n`; hence `n ↦ t_n` is injective and (by T1 trichotomy) order-preserving in both directions, `m < n ⟺ t_m < t_n`.

Source: ASN-0040 S0 (StreamOrdering) — `(A i, j : 1 ≤ i < j : cᵢ < cⱼ)`.

## DisjointSubAllocatorChains — DisjointSubAllocatorChains (LEMMA, lemma)

`A_C(d)` and `A_L(d)` are disjoint as address sets, and addresses produced by `A_C(d)` (resp. `A_L(d)`) carry `E(·)₁ = s_C` (resp. `s_L`).

Source: ASN-0040 B7 (NamespaceDisjointness) — `S(b_C(d), 1) ∩ S(b_L(d), 1) = ∅`, since `(b_C(d), 1) ≠ (b_L(d), 1)` (the anchors disagree at position `#d + 2`, `s_C` vs `s_L`, by SC-NEQ) and both parents are `B6`-valid. The subspace-identifier reading follows from ChainPrefixExtension: every element of `A_C(d)` agrees with `b_C(d)` at position `#d + 2`, where the value is `s_C` (resp. `s_L` for `A_L(d)`).

## ChainPrefixExtension — ChainPrefixExtension (LEMMA, lemma)

Every element of an active sub-allocator chain extends its anchor under the prefix relation:

`(A d ∈ dom(M), t ∈ A_C(d) :: b_C(d) ≼ t)`
`(A d ∈ dom(M), t ∈ A_L(d) :: b_L(d) ≼ t)`

Source: ASN-0040 S1 (StreamPrefix) — `(A n : n ≥ 1 : p ≼ cₙ)`, applied at `p = b_C(d)` (resp. `b_L(d)`).

## FirstEmission — FirstEmission (LEMMA, lemma)

The first emission of each chain has a determinate structural form:

- *Content chain first-emit:* `t_1^C(d) = inc(b_C(d), 1) = [d.0.s_C.1]` — `E(·)₁ = s_C`, `origin(·) = d`, `#E(·) = 2`, `zeros(·) = 3`, and T4-valid.
- *Link chain first-emit:* `t_1^L(d) = inc(b_L(d), 1) = [d.0.s_L.1]` — structurally analogous, with `s_L` in place of `s_C`.

Where the anchors are defined as:

- `b_C(d) := [d.0.s_C]` — one-component element field with `E₁ = s_C`, `zeros = 3`, `#E = 1`
- `b_L(d) := [d.0.s_L]` — one-component element field with `E₁ = s_L`, `zeros = 3`, `#E = 1`

With anchor construction: `b_C(d) = inc(d, 2)` and `b_L(d) = inc(b_C(d), 0)`.

## ChainMembershipForOrigin — ChainMembershipForOrigin (LEMMA, lemma)

At every reachable state `Σ`, every entry of `dom(C)` (resp. `dom(L)`) inhabits the content (resp. link) sub-allocator chain of its origin, and forms a *contiguous initial segment* of that chain. Letting `A_C(d) = (t_1, t_2, t_3, …)` denote the enumeration of `d`'s content sub-allocator chain (with `t_1` the first emission and `t_{n + 1} = inc(t_n, 0)`), and `A_L(d) = (s_1, s_2, s_3, …)` the analogous link chain:

- `(A d ∈ dom(M) :: (E m_d ≥ 0 :: dom(C) ∩ {a' ∈ T : origin(a') = d} = {t_1, …, t_{m_d}}))` (content contiguous prefix; `{t_1, …, t_0} = ∅` by convention)
- `(A d ∈ dom(M) :: (E n_d ≥ 0 :: dom(L) ∩ {ℓ' ∈ T : origin(ℓ') = d} = {s_1, …, s_{n_d}}))` (link contiguous prefix)

The weaker subset inclusion `dom(C) ∩ {a' : origin(a') = d} ⊆ A_C(d)` (and its link analogue) is the immediate corollary of the contiguous-prefix form.

## StoreT4Validity — StoreT4Validity (LEMMA, lemma)

At every reachable state `Σ`, every entry of `dom(C) ∪ dom(L)` is a T4-valid tumbler:

`(A a ∈ dom(C) :: T4-valid(a))`
`(A ℓ ∈ dom(L) :: T4-valid(ℓ))`

## FirstEmissionFreshness — FirstEmissionFreshness (LEMMA, lemma)

At every reachable state `Σ`, the first emission of an active sub-allocator chain — the address that K.α (resp. K.λ) commits when the corresponding first-emit predicate fires — is fresh against `dom(C) ∪ dom(L)`:

- *Content first-emit:* Under the K.α first-emit predicate `{a' ∈ dom(C) : origin(a') = d} = ∅`, the first emission `a = [d.0.s_C.1]` of `A_C(d)` satisfies `a ∉ dom(C) ∪ dom(L)` at the K.α event that commits `a`.
- *Link first-emit:* Under the K.λ first-emit predicate `{ℓ' ∈ dom(L) : origin(ℓ') = d} = ∅`, the first emission `ℓ = [d.0.s_L.1]` of `A_L(d)` satisfies `ℓ ∉ dom(L) ∪ dom(C)` at the K.λ event that commits `ℓ`.

## SubsequentEmissionFreshness — SubsequentEmissionFreshness (LEMMA, lemma)

At every reachable state `Σ`, the subsequent emission of an active sub-allocator chain — the address `a = inc(a_prev, 0)` that K.α commits when its subsequent-emit predicate fires, with `a_prev = max{a' ∈ dom(C) : origin(a') = d}` (resp. `ℓ = inc(ℓ_prev, 0)` for K.λ, with `ℓ_prev = max{ℓ' ∈ dom(L) : origin(ℓ') = d}`) — is fresh against `dom(C) ∪ dom(L)`. Freshness splits three ways:

- *Within-document* (against `{a' ∈ dom(C) : origin(a') = d}`): `a = inc(a_prev, 0) ∈ A_C(d)` by ChainDiscipline's closure under `inc(·, 0)`, with `a_prev ∈ A_C(d)` by ChainMembershipForOrigin; ChainEnumerationInjectivity applied to `(a_prev, a)` gives strict advance past every prior same-chain sibling, so `a ∉ dom(C)` at `d`.
- *Cross-document* (against `{a' ∈ dom(C) : origin(a') ≠ d}`): Cross-document disjointness applied to `(d, origin(a'))` plus T10 (PartitionIndependence, ASN-0034) gives `a ≠ a'`, exactly as in the FirstEmissionFreshness content-against-`dom(C)` case.
- *Cross-subspace* (against `dom(L)`): `E(a)₁ = s_C` (read along `A_C(d)` via DisjointSubAllocatorChains — structural, since `a ∈ A_C(d)`) while `E(ℓ)₁ = s_L` for every pre-existing peer `ℓ ∈ dom(L)` (L0); SC-NEQ and T7 (SubspaceDisjointness, ASN-0034) close `a ≠ ℓ`, exactly as in the FirstEmissionFreshness content-against-`dom(L)` case.

The link subsequent emission is symmetric under the content↔link substitution.

## Cross-doc disjointness — CrossDocumentDisjointness (LEMMA, lemma)

For any two distinct documents `d₁, d₂ ∈ dom(M)` with `d₁ ≠ d₂`, the anchors `p_i := b_·(d_i)` (for `· ∈ {L, C}`) are prefix-incomparable, `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`, so by T10 (PartitionIndependence, ASN-0034) every address extending one anchor differs from every address extending the other:

`a ≠ b`  for every `a` with `p₁ ≼ a` and every `b` with `p₂ ≼ b`.

## SubspaceConventionAxiom — FixedSubspaceIdentifiers (AXIOM, axiom)

`s_C = 1 ∧ s_L = 2`

The distinctness `s_C ≠ s_L` (abbreviated **SC-NEQ**) and the sibling relation `s_L = s_C + 1` are immediate consequences.

## SequentialTransitionAxiom — SequentialAtomicTransitions (AXIOM, axiom)

Transitions `Σ → Σ'` are atomic, uninterruptible, and totally ordered: each transition evaluates its precondition against `Σ` and commits its effect to `Σ'` in one indivisible step.

## K.σ — DocumentRegistration (OP, operation)

Extends `dom(M)` by registering a new document address with an empty arrangement.

*Precondition:*
- `d ∉ dom(M)` (fresh document address)
- `T4-valid(d) ∧ zeros(d) = 2` (document-level — discharges M0 at the new key)

*Effect:* `dom(M') = dom(M) ∪ {d}`, with `M'(d) = ∅` and `M'(d') = M(d')` for every `d' ∈ dom(M)`.

*Frame:* `C' = C; L' = L`

## K.α — ContentAllocation (OP, operation)

Extends `dom(C)` with a fresh content address scoped to an allocated document.

*Binding precondition:*
- `d ∈ dom(M)` (home document exists)
- `a` is produced by `d`'s content sub-allocator `A_C(d)`:
  - *First emission* (predicate: `{a' ∈ dom(C) : origin(a') = d} = ∅`): `a = [d.0.s_C.1]`.
  - *Subsequent emission* (predicate: `{a' ∈ dom(C) : origin(a') = d} ≠ ∅`): `a = inc(a_prev, 0)` (TA5(c)) where `a_prev := max{a' ∈ dom(C) : origin(a') = d}`, the next sibling on `A_C(d)`'s `inc(·, 0)` chain. The `max` is well-defined because the set is finite (C-fin restricted by `origin(·) = d`).
- `v ∈ Val` (well-formed content value)

*Effect:* `C' = C ∪ {a ↦ v}`

*Frame:* `L' = L; M' = M`.

## K.λ — LinkAllocation (OP, operation)

Extends `dom(L)` with a fresh link address scoped to an allocated document.

Signature: `K.λ(d, ℓ, (e₁, …, eₙ))` where the link value is a finite sequence of `N` endsets.

*Binding precondition:*
- `d ∈ dom(M)` (home document exists)
- `ℓ` is produced by `d`'s link sub-allocator `A_L(d)`:
  - *First emission* (predicate: `{ℓ' ∈ dom(L) : origin(ℓ') = d} = ∅`): `ℓ = [d.0.s_L.1]`, the determinate first emission of `A_L(d)`.
  - *Subsequent emission* (predicate: `{ℓ' ∈ dom(L) : origin(ℓ') = d} ≠ ∅`): `ℓ = inc(ℓ_prev, 0)` (TA5(c)) where `ℓ_prev := max{ℓ' ∈ dom(L) : origin(ℓ') = d}`, the next sibling on `A_L(d)`'s `inc(·, 0)` chain. The `max` is well-defined because the set is finite (L-fin restricted by `origin(·) = d`).
- `N ≥ 3 ∧ (A i : 1 ≤ i ≤ N : eᵢ ∈ Endset) ∧ e₃ ≠ ∅` (well-formed link value with mandatory non-empty type endset at slot 3 — L3).

*Effect:* `L' = L ∪ {ℓ ↦ (e₁, …, eₙ)}`

*Frame:* `C' = C; M' = M`.
