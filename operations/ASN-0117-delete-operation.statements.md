> **ASN-0117 · DELETE Operation** — condensed claim statements  
> [← Full note](ASN-0117-delete-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0117 Claim Statements

*Source: ASN-0117-delete-operation.md (revised 2026-06-08) — Extracted: 2026-06-11*

## Definition — DeleteRegions

Three regions partition `V_S(d)` by trichotomy of T1:

- `L = {v ∈ V_S(d) : v < p}` — the prefix, untouched
- `X = {v ∈ V_S(d) : p ≤ v < r}` — the deleted block, `|X| = c`
- `R = {v ∈ V_S(d) : v ≥ r}` — the suffix, shifted left

where `p = q_J` is the first deleted position, `r = p ⊕ w = q_{J+c}` is the first surviving position past the gap, `c = w₂` is the deletion width, and `w = [0, c]`.

## Definition — SuffixShift

`σ(q_k) = q_{k−c}` for `k ≥ J + c`, where `q_J` is the first deleted slot and `c` the deletion width — left-shifting the last component by `c` carries the `k`-th slot to the `(k−c)`-th, leaving the shared prefix `[S, 1, …, 1]` untouched. This is the ordinal subtraction `ord(q_k) ⊖ w_ord` of the foundation contraction (ASN-0082), well-defined and order-preserving on the surviving suffix.

## Definition — DeletedAddresses

`A_del = {M(d)(q_k) : J ≤ k < J + c}` — the set of I-addresses mapped by the deleted V-positions `{q_J, …, q_{J+c−1}}` in document `d`.

## Definition — ExclusivelyDeletedAddresses

`A_del^{excl} = A_del \ M(d)(L ∪ R)` — the set of deleted I-addresses that no surviving position of `d` also maps (the addresses `d` loses from its range entirely).

Post-state range identity:
`ran(M'(d)) = ran(M(d)) \ A_del^{excl}`

expanded as:
`ran(M'(d)) = M(d)(L) ∪ M(d)(R) ∪ ran(M(d)|_{V_{s_L}(d)})`

## Definition — DiscoverableFrom

`discoverable_from(a, d, Σ) ⟺ (E i : coverage(Σ.L(a).eᵢ) ∩ ran(M(d)) ≠ ∅)`

(foundation **LP12 (DiscoverabilityCharacterisation)**, ASN-0098)

## Definition — DiscoverabilitySet

`D(d, Σ) = {a ∈ dom(Σ.L) : discoverable_from(a, d, Σ)}`

## Definition — DeleteOperation

**DELETE(`d`, `p`, `w`).**

*Precondition.* `Σ` is a *composite boundary of a valid transition trace* from the initial state `Σ₀` (P4a's sense, ASN-0047); `d ∈ dom(M)`; `S = subspace(p) = s_C`; `m = #p = 2`, equal to the common depth S8-depth fixes on `V_S(d)`; `p ∈ V_S(d)` is S8a-well-formed; `w₁ = 0`, `#w = #p`, `Pos(w)` — so `w = [0, c]` with `c = w₂ ≥ 1`; and *containment* — `p = q_J` and `r = p ⊕ w = q_{J+c}` with `1 ≤ J` and `J + c ≤ N + 1` (the case `J + c = N + 1` deletes a suffix, leaving `R = ∅`).

*Effect.* DELETE is one arrangement contraction realising ASN-0082's displacement family, with the content store held in frame:

- *Case `R ≠ ∅` (`J + c ≤ N`):* DELETE is the K.μ⁻ + K.μ⁺ composite — (1) a K.μ⁻ step that contracts the text subspace to its surviving prefix `L = {q_1, …, q_{J−1}}` (retention count `n'_{s_C} = J − 1`), while holding the link subspace at full retention (`n'_{s_L} = n_{s_L}`); (2) a K.μ⁺ step that re-places the `N − c − (J − 1)` survivors at the closed-up text positions `{q_J, …, q_{N−c}}`, each carrying the I-address it held before — the former images of `q_{J+c}, …, q_N`.

- *Case `R = ∅` (`J + c = N + 1`):* DELETE is a single K.μ⁻ step — a prefix-retention truncation of the text subspace to count `n'_{s_C} = J − 1 = N − c`, the link subspace held at full retention. The delete-everything sub-case `J = 1, c = N` is this with `n'_{s_C} = 0`.

---

## DELETE — DeleteOperation (OP, operation)

Remove the span `(p, w)` of width `c` from document `d`'s arrangement; shift the suffix left to close the gap; touch the content store not at all.

---

## P0 — NonDestruction (LEMMA, postcondition)

*DELETE does not touch the content store:*
`dom(C') = dom(C)` and `(A b : b ∈ dom(C) : C'(b) = C(b))`. In particular every deleted I-address survives: `A_del ⊆ dom(C')` with content preserved.

---

## P2 — GapClosure (LEMMA, postcondition)

The surviving content closes into the dense run `V_S(d') = {q_1, …, q_{N−c}}` of length `N − c`. The prefix `L` is fixed; the suffix `R` shifts left uniformly by `c` via the order-preserving injection `σ`, carrying each survivor's I-address unchanged (`M'(d)(σ(v)) = M(d)(v)`). The underlying arithmetic identity `ord(r) ⊖ w_ord = ord(p)` holds unconditionally (ASN-0082 D-SEP(a)); when `R ≠ ∅` it reads positionally as the gap closing exactly — `σ(q_{J+c}) = q_J`, the first survivor landing where the deletion began (ASN-0082 D-SEP(b)). In the suffix-delete case `J + c = N + 1`, `R = ∅` and `q_{J+c} = q_{N+1}` is not an arranged position: there is no gap to close, and the positional reading is vacuous. Relative order and density are preserved; no hole, no overlap, no degenerate position.

---

## P4 — LinkSurvival (LEMMA, postcondition)

For every link `a ∈ dom(Σ.L)` and slot `i`, the stored endset has unchanged coverage: `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)` (LP3★, MultiStepCoverageInvariance, ASN-0098), so no link's designated content changes, and the link store is untouched (`Σ'.L = Σ.L`). A link discoverable from `d` before the deletion remains discoverable from `d` iff some surviving V-position of `d` still maps into its coverage; otherwise it is undiscoverable from `d` (LP12 read at `Σ'`) yet persists (L12), remains discoverable from every other document that still arranges its coverage (LP12), and is re-discoverable from `d` should the content be re-arranged (Store Monotonicity★ + LP3★ + LP12 at the later state).

---

## P5 — DocumentIsolation (LEMMA, postcondition)

For every `d' ≠ d`: `M'(d') = M(d')`, and every V-position of `d'` resolves to identical content across the transition, in whichever store its subspace designates — the two cases below exhausting `d'`'s positions by S3★-aux (SubspaceExhaustiveness, ASN-0047), which admits no subspace beyond `s_C` and `s_L`.

- For content-subspace positions (`subspace(v') = s_C`): `M'(d')(v') ∈ dom(Σ'.C)` with `C'(M'(d')(v')) = C(M(d')(v'))` (P0).
- For link-subspace positions (`subspace(v') = s_L`): `M'(d')(v') ∈ dom(Σ'.L)` with `Σ'.L(M'(d')(v')) = Σ.L(M(d')(v'))` (DEL-CFRAME).

The arrangement and resolved content of every other document — including any that transcludes the deleted I-addresses — are invariant under DELETE on `d`.

---

## DEL-REMOVE — DomainContraction (POST, postcondition)

The arrangement loses exactly `c` V→I correspondences in subspace `S`: the surviving domain contracts by precisely the deletion width, `|{v ∈ dom(M'(d)) : subspace(v) = S}| = N − c`, and the top `c` position labels leave the domain, `(A k : N − c < k ≤ N : q_k ∉ dom(M'(d)))`. The deleted I-addresses `A_del` persist in `C` (P0); they are not removed from anything else, and may be mapped by other positions of `d` or by other documents.

---

## DEL-SHIFT — SuffixShiftPost (POST, postcondition)

`(A v : v ∈ R : σ(v) ∈ dom(M'(d)) ∧ M'(d)(σ(v)) = M(d)(v))` — verbatim ASN-0082 **D-SHIFT**, with `σ(q_k) = q_{k−c}`.

---

## DEL-LEFT — PrefixUnchanged (POST, postcondition)

`(A v : v ∈ L : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))` — ASN-0082 **D-L**.

---

## DEL-DOM — PostDomain (POST, postcondition)

`{v ∈ dom(M'(d)) : subspace(v) = S} = L ∪ {σ(v) : v ∈ R}` — ASN-0082 **D-DOM**.

---

## DEL-CFRAME — CompositeFrame (FRAME, frame)

`Σ'.L = Σ.L ∧ Σ'.E = Σ.E ∧ Σ'.R = Σ.R` — discharged for both realisations. The link store is fixed in both domain and per-address value (`dom(Σ'.L) = dom(Σ.L)` and `(A a : a ∈ dom(Σ.L) : Σ'.L(a) = Σ.L(a))`); on the fixed entity set, P1 (EntityPermanence) and P8 (EntityHierarchy) survive DELETE trivially.

---

## DEL-FSUB — SubspaceFrame (FRAME, frame)

`(A S' : S' ≠ S : {v ∈ dom(M'(d)) : subspace(v) = S'} = {v ∈ dom(M(d)) : subspace(v) = S'}` and `M'(d)` agrees there`)` — ASN-0082 **D-CS**. In particular the document's links (subspace `s_L`) are not moved by a text deletion.

---

## DEL-FDOC — DocumentFrame (FRAME, frame)

`(A d' : d' ≠ d : M'(d') = M(d'))` — ASN-0082 **D-CD**.

---

## Definition — DiscoverabilityPreservationWP

`wp(DELETE, D(d, Σ') = D(d, Σ))` ≡

```
DELETE-pre
∧ (A a ∈ dom(Σ.L) :
    (E i : coverage(Σ.L(a).eᵢ) ∩ ran(M(d)) ≠ ∅)
    ⟹
    (E i : coverage(Σ.L(a).eᵢ) ∩ (ran(M(d)) \ A_del^{excl}) ≠ ∅))
```

Equivalently, substituting into LP12 at `Σ'`:

```
discoverable_from(a, d, Σ')
  ⟺ (E i : coverage(eᵢ) ∩ (ran(M(d)) \ A_del^{excl}) ≠ ∅)
```

A link drops from `D(d, ·)` precisely when all of its witnesses in `d` lay in `A_del^{excl}` — when the deleted span carried the link's last anchor in `d`. Discoverability is preserved exactly when the deleted span removed no link's last witness in `d`.
