> **ASN-0084 · Cut-Point Rearrangements** — condensed claim statements  
> [← Full note](ASN-0084-bundle-projection-displacement.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0084 Claim Statements

*Source: ASN-0084-bundle-projection-displacement.md (revised 2026-04-10) — Extracted: 2026-05-30*

## ArrangementRearrangement — ArrangementRearrangement (DEF, definition)

An *arrangement rearrangement* is a state transition Σ → Σ' in which dom(M'(d)) = dom(M(d)), C' = C (S0, ASN-0036), M'(d') = M(d') for all d' ≠ d, and there exists a bijection π : dom(M(d)) → dom(M'(d)) such that M'(d)(π(v)) = M(d)(v) for all v ∈ dom(M(d)).

---

## CutSequence — CutSequence (DEF, definition)

A *cut sequence* for document d in subspace S is a tuple K = (c₀, c₁, ..., c_{n−1}) of tumblers satisfying:

(CS1) n ∈ {3, 4} — exactly three or four cuts.

(CS2) c₀ < c₁ < ... < c_{n−1} under T1 (ASN-0034) — strictly ordered.

(CS3) subspace(cᵢ) = S = 1 for all i — all cuts in the text subspace.

(CS4) #cᵢ = 2 for all i — depth-2 positions.

(CS5) ord(cᵢ) ≥ 1 for all i — the second component is positive (zero-free, matching the S8a form [S, q] with q ∈ ℕ⁺). This makes each cut cᵢ = [1, ord(cᵢ)] a singleton-identifiable positive ordinal, so ord(cᵢ) ∈ ℕ⁺ and the truncated subtraction on cut ordinals is well-defined on every adjacent cut pair.

---

## RegionPartition — RegionPartition (DEF, definition)

Given a cut sequence K for document d in subspace S with V_S(d) ≠ ∅:

For n = 3, the *affected range* A = {v ∈ V_S(d) : c₀ ≤ v < c₂} is partitioned:

```
α = {v ∈ V_S(d) : c₀ ≤ v < c₁}     — first region
β = {v ∈ V_S(d) : c₁ ≤ v < c₂}     — second region
```

For n = 4, the *affected range* A = {v ∈ V_S(d) : c₀ ≤ v < c₃} is partitioned:

```
α = {v ∈ V_S(d) : c₀ ≤ v < c₁}     — first region
μ = {v ∈ V_S(d) : c₁ ≤ v < c₂}     — middle region
β = {v ∈ V_S(d) : c₂ ≤ v < c₃}     — second region
```

We write w_α = |α|, w_β = |β|, w_μ = |μ| for the region widths.

---

## R-PRE — RearrangePrecondition (DEF, definition)

(i) M(d) is well-defined (the document's arrangement exists).

(ii) V_S(d) ≠ ∅ (the subspace is non-empty).

(iii) The cut sequence K = (c₀, ..., c_{n−1}) satisfies CS1–CS5.

(iv) The affected range lies entirely within the current arrangement:

`(A v : subspace(v) = S ∧ #v = 2 ∧ c₀ ≤ v < c_{n−1} : v ∈ V_S(d))`

*Consequences:* Width positivity: w_α ≥ 1 and w_β ≥ 1 in both forms, and additionally w_μ ≥ 1 when n = 4. By R-PRE(iv) and D-SEQ (ASN-0036), the widths are computable from the cut-point ordinals: w_α = ord(c₁) − ord(c₀); w_β = ord(c₂) − ord(c₁) for n = 3 and ord(c₃) − ord(c₂) for n = 4; w_μ = ord(c₂) − ord(c₁) for n = 4.

---

## PivotPostcondition — PivotPostcondition (DEF, definition)

Given a 3-cut sequence K = (c₀, c₁, c₂) satisfying R-PRE, the *pivot* produces arrangement M'(d) defined by:

(R-EXT) For v ∈ V_S(d) with v < c₀ or v ≥ c₂:

`M'(d)(v) = M(d)(v)`

(R-P1) For 0 ≤ j < w_β:

`M'(d)(c₀ + j) = M(d)(c₁ + j)`

(R-P2) For 0 ≤ j < w_α:

`M'(d)(c₀ + w_β + j) = M(d)(c₀ + j)`

The domain is dom(M'(d)) = dom(M(d)).

(R-FRAME-P) Frame conditions:

(a) For v ∈ dom(M(d)) with subspace(v) ≠ S: M'(d)(v) = M(d)(v).

(b) For all d' ≠ d: M'(d') = M(d').

(c) C' = C (S0, ASN-0036).

---

## SwapPostcondition — SwapPostcondition (DEF, definition)

Given a 4-cut sequence K = (c₀, c₁, c₂, c₃) satisfying R-PRE, the *swap* produces M'(d) defined by:

(R-EXT) For v ∈ V_S(d) with v < c₀ or v ≥ c₃:

`M'(d)(v) = M(d)(v)`

(R-S1) For 0 ≤ j < w_β:

`M'(d)(c₀ + j) = M(d)(c₂ + j)`

(R-S2) For 0 ≤ j < w_μ:

`M'(d)(c₀ + w_β + j) = M(d)(c₁ + j)`

(R-S3) For 0 ≤ j < w_α:

`M'(d)(c₀ + w_β + w_μ + j) = M(d)(c₀ + j)`

The domain is dom(M'(d)) = dom(M(d)).

(R-FRAME-S) Frame conditions:

(a) For v ∈ dom(M(d)) with subspace(v) ≠ S: M'(d)(v) = M(d)(v).

(b) For all d' ≠ d: M'(d') = M(d').

(c) C' = C (S0, ASN-0036).

---

## REARRANGE_K — RearrangeK (OPERATION, method)

REARRANGE_K(Σ, d) is the state transition Σ → Σ' that produces Σ' satisfying PivotPostcondition (when n = 3) or SwapPostcondition (when n = 4) together with the corresponding frame conditions R-FRAME-P (n = 3) or R-FRAME-S (n = 4). The two cases of n are mutually exclusive (CS1) and exhaustive over admissible cut sequences, so REARRANGE_K is a single operation whose postcondition specializes by the cut count.

*Partiality.* REARRANGE_K is partial, defined exactly where R-PRE(K) holds against Σ.M(d).

---

## Split — RunSplit (DEF, definition)

Given a run b = (v, a, n) under some arrangement A and an interior offset c with 1 ≤ c < n, the *split* at c produces two runs: (v, a, c) and (v + c, a + c, n − c). Their V-extents (ordinal ranges [ord(v), ord(v) + c) and [ord(v) + c, ord(v) + n)) are disjoint and partition b's V-extent.

Both pieces inherit S8-cons (consistency under A). For the first piece (v, a, c): A(v + k) = a + k for 0 ≤ k < c. For the second piece (v + c, a + c, n − c): A((v + c) + k) = (a + c) + k for 0 ≤ k < n − c. By Extended Associativity, (v + c) + k = v + (c + k), and since c + k < n, S8-cons of the original run yields A(v + (c + k)) = a + (c + k) = (a + c) + k.

---

## Merge — RunMerge (DEF, definition)

Two runs (v₁, a₁, n₁) and (v₂, a₂, n₂) under arrangement A are *mergeable* when v₂ = v₁ + n₁ (V-adjacent) and a₂ = a₁ + n₁ (I-adjacent). The merged run is (v₁, a₁, n₁ + n₂).

It inherits S8-cons: for 0 ≤ k < n₁ it holds by run 1, and for n₁ ≤ k < n₁ + n₂, writing k = n₁ + j with 0 ≤ j < n₂, Extended Associativity gives v₁ + k = (v₁ + n₁) + j = v₂ + j, so A(v₁ + k) = A(v₂ + j) = a₂ + j = (a₁ + n₁) + j = a₁ + k.

---

## CanonicalRunDecomposition — CanonicalRunDecomposition (DEF, definition)

The *canonical run decomposition* of an arrangement is S8's (CorrespondenceRunPartition, ASN-0036) unique maximal-run partition.

*Termination and confluence of merging:* Each application of Merge replaces two runs by one, strictly decreasing the (finite, ≥ 1) run count, so every sequence of merges terminates. The terminal partition, which by construction admits no mergeable pair, satisfies R-CANON's hypotheses and is the canonical decomposition. Because that decomposition is unique (S8), the terminal result is independent of the order in which mergeable pairs are fused: iterated merging is confluent, and the canonical partition is its unique normal form.

---

## R-PIV — PivotWellDefined (LEMMA, supporting)

The pivot postcondition defines a total function on dom(M(d)) (each position is assigned exactly one I-address).

Formally: (a) every v ∈ dom(M(d)) falls under exactly one of R-FRAME-P(a), R-EXT, R-P1, R-P2, and (b) the right-hand sides M(d)(c₁ + j) for j < w_β and M(d)(c₀ + j) for j < w_α are well-defined (all source positions lie in dom(M(d)) by R-PRE(iv)).

The R-P1 ordinal range is [p, p + w_β) and the R-P2 ordinal range is [p + w_β, p + w_β + w_α), where p = ord(c₀). These are disjoint, their union is [p, p + w_α + w_β) = [c₀, c₂) ∩ V_S(d), and together with R-EXT (covering V_S(d) \ [c₀, c₂)), every position is covered exactly once.

---

## R-SWP — SwapWellDefined (LEMMA, supporting)

The swap postcondition defines a total function on dom(M(d)).

Formally: (a) every v ∈ dom(M(d)) falls under exactly one of R-FRAME-S(a), R-EXT, R-S1, R-S2, R-S3, and (b) all right-hand side positions lie in dom(M(d)) by R-PRE(iv).

Let p = ord(c₀). The ordinal ranges are: R-S1: [p, p + w_β); R-S2: [p + w_β, p + w_β + w_μ); R-S3: [p + w_β + w_μ, p + w_β + w_μ + w_α); R-EXT: outside [p, p + w_α + w_μ + w_β). Since w_α ≥ 1, w_β ≥ 1, w_μ ≥ 1, the half-open intervals are non-empty with strictly increasing left endpoints: p < p + w_β < p + w_β + w_μ < p + w_β + w_μ + w_α. Their union covers V_S(d) and p + w_β + w_μ + w_α = ord(c₃).

---

## R-PPERM — PivotPermutation (LEMMA, lemma)

The *cut-point-induced bijection* π : dom(M(d)) → dom(M'(d)) satisfying M'(d)(π(v)) = M(d)(v) is the specific bijection determined by the cut sequence K and the region partition. The formula is:

```
         ⎧ v                   if subspace(v) ≠ S                       (non-S)
         ⎪ v                   if v ∈ V_S(d) and (v < c₀ or v ≥ c₂)   (subspace-S exterior)
π(v) =  ⎨ c₀ + w_β + j        if v = c₀ + j, 0 ≤ j < w_α              (α → end)
         ⎩ c₀ + j              if v = c₁ + j, 0 ≤ j < w_β              (β → start)
```

The subspace-S exterior, α, and β branches partition V_S(d), so the four-case piecewise definition is total on dom(M(d)).

*Correctness:* M'(d)(π(v)) = M(d)(v) in each case — for α: M'(d)(c₀ + w_β + j) = M(d)(c₀ + j) by R-P2; for β: M'(d)(c₀ + j) = M(d)(c₁ + j) by R-P1.

*Bijectivity:* π is an injection from dom(M(d)) into itself; dom(M(d)) is finite (S8-fin of ASN-0036); on a finite set every self-injection is a bijection.

---

## R-SPERM — SwapPermutation (LEMMA, lemma)

The *cut-point-induced bijection* π satisfying M'(d)(π(v)) = M(d)(v) is the specific bijection determined by the 4-cut sequence K and the regions α, μ, β. The formula is:

```
         ⎧ v                        if subspace(v) ≠ S                      (non-S)
         ⎪ v                        if v ∈ V_S(d) and (v < c₀ or v ≥ c₃)   (subspace-S exterior)
         ⎪ c₀ + w_β + w_μ + j       if v = c₀ + j, 0 ≤ j < w_α             (α → end)
π(v) =  ⎨ c₀ + w_β + j             if v = c₁ + j, 0 ≤ j < w_μ             (μ → middle)
         ⎩ c₀ + j                   if v = c₂ + j, 0 ≤ j < w_β             (β → start)
```

The subspace-S exterior, α, μ, and β branches partition V_S(d), so the five-case piecewise definition is total on dom(M(d)).

*Correctness:* M'(d)(π(v)) = M(d)(v) in each case — for α: M'(d)(c₀ + w_β + w_μ + j) = M(d)(c₀ + j) by R-S3; for μ: M'(d)(c₀ + w_β + j) = M(d)(c₁ + j) by R-S2; for β: M'(d)(c₀ + j) = M(d)(c₂ + j) by R-S1.

*Bijectivity:* π is an injection from dom(M(d)) into itself and dom(M(d)) is finite (S8-fin of ASN-0036), hence a bijection.

---

## R-FRAME-P — PivotFrameConditions (FRAME, predicate)

(a) For v ∈ dom(M(d)) with subspace(v) ≠ S: M'(d)(v) = M(d)(v).

(b) For all d' ≠ d: M'(d') = M(d').

(c) C' = C (S0, ASN-0036).

---

## R-FRAME-S — SwapFrameConditions (FRAME, predicate)

(a) For v ∈ dom(M(d)) with subspace(v) ≠ S: M'(d)(v) = M(d)(v).

(b) For all d' ≠ d: M'(d') = M(d').

(c) C' = C (S0, ASN-0036).

---

## R-NS — NonSubspaceInvariance (LEMMA, lemma)

*(NS-M) Pointwise identity on non-S.* For every v ∈ dom(M(d)) with subspace(v) ≠ S: M'(d)(v) = M(d)(v).

*Proof.* The frame condition R-FRAME-P(a) (n = 3) or R-FRAME-S(a) (n = 4) gives M'(d)(v) = M(d)(v) directly for every v ∈ dom(M(d)) with subspace(v) ≠ S. ∎

---

## R-RI — RearrangementReferentialIntegrity (LEMMA, lemma)

*Preconditions:* M(d) is well-defined; M'(d) results from an arrangement rearrangement of M(d) (dom(M'(d)) = dom(M(d)), C' = C, M'(d') = M(d') for d' ≠ d, and there exists a bijection π with M'(d)(π(v)) = M(d)(v)).

*Postcondition:* ran(M'(d)) ⊆ dom(C').

*Key intermediate:* I-address range invariance: ran(M'(d)) = ran(M(d)).

*Derivation:* ran(M'(d)) = {M'(d)(π(v)) : v ∈ dom(M(d))} = {M(d)(v) : v ∈ dom(M(d))} = ran(M(d)).

*Chaining:* ran(M'(d)) = ran(M(d)) ⊆ dom(C) = dom(C').

---

## R-COMM — PermutationShiftCommutativity (LEMMA, lemma)

Let π be a cut-point permutation (R-PPERM or R-SPERM) for a cut sequence K satisfying R-PRE. For any V-position v ∈ dom(M(d)) and offset k ≥ 0 such that v + k ∈ dom(M(d)) and v, v + k lie in the same region — where the regions are the non-S subspace ({v ∈ dom(M(d)) : subspace(v) ≠ S}), the subspace-S exterior, α, μ, or β:

`π(v + k) = π(v) + k`

Per-region verification (using R-PPERM / R-SPERM formulas and Extended Associativity):

- *Non-S subspace (both forms):* π(v) = v and π(v + k) = v + k, so π(v + k) = v + k = π(v) + k.
- *Subspace-S exterior (both forms):* π(v + k) = v + k = π(v) + k.
- *3-cut α:* v = c₀ + j', 0 ≤ j' < w_α; same-region gives 0 ≤ j' + k < w_α; π(v + k) = c₀ + w_β + (j' + k) = (c₀ + w_β + j') + k = π(v) + k.
- *3-cut β:* v = c₁ + j', 0 ≤ j' < w_β; same-region gives 0 ≤ j' + k < w_β; π(v + k) = c₀ + (j' + k) = (c₀ + j') + k = π(v) + k.
- *4-cut α:* v = c₀ + j', 0 ≤ j' < w_α; same-region gives 0 ≤ j' + k < w_α; π(v + k) = c₀ + w_β + w_μ + (j' + k) = π(v) + k.
- *4-cut μ:* v = c₁ + j', 0 ≤ j' < w_μ; same-region gives 0 ≤ j' + k < w_μ; π(v + k) = c₀ + w_β + (j' + k) = π(v) + k.
- *4-cut β:* v = c₂ + j', 0 ≤ j' < w_β; same-region gives 0 ≤ j' + k < w_β; π(v + k) = c₀ + (j' + k) = π(v) + k.

---

## R-BLK — RunDecompositionTransformation (LEMMA, lemma)

Let B = {b₁, ..., bₘ} be a run partition of M(d) (per S8). Let the cut sequence K have cut positions c₀, ..., c_{n−1}. The rearranged arrangement M'(d) admits a run partition B' — disjoint and covering, but not necessarily the maximal (canonical) decomposition — obtained by:

*Phase 1: Split.* Process cut positions in index order (c₀, c₁, ..., c_{n−1}). For each cut position cᵢ:

- *Interior of a run:* if cᵢ ∈ V(bₖ) for some bₖ = (vₖ, aₖ, nₖ) with cᵢ ≠ vₖ, split bₖ at offset c = ord(cᵢ) − ord(vₖ), producing (vₖ, aₖ, c) and (vₖ + c, aₖ + c, nₖ − c).
- *Boundary of a run:* if cᵢ ∈ V(bₖ) and cᵢ = vₖ, no split needed.
- *Outside ⋃_k V(bₖ):* no split performed. By CS2–CS4 and R-PRE(iv), c₀, ..., c_{n−2} ∈ V_S(d) ⊆ ⋃_k V(bₖ); only c_{n−1} may fall outside, and EXT-VAC gives c_{n−1} ∉ dom(M(d)) with empty right exterior.

Non-S runs are not split: every cut position lies in subspace S (CS3), and by SUBCONF every position in a non-S run has subspace ≠ S, so no cut falls in any non-S run.

*Phase 2: Classify.* Each run in the post-split partition lies entirely within one region (non-S subspace, subspace-S exterior, α, μ if 4-cut, or β), because no run crosses a cut boundary.

*Phase 3: Reassemble.* Each run (vₖ, aₖ, nₖ) becomes (π(vₖ), aₖ, nₖ): the V-start is replaced by π(vₖ); the I-start aₖ and width nₖ are preserved verbatim.

*Validity of reassembled runs:* By R-COMM (same-region precondition discharged by Phase 2), π(vⱼ + k) = π(vⱼ) + k for every 0 ≤ k < nⱼ. Hence S8-cons under M'(d): M'(d)(π(vⱼ) + k) = M'(d)(π(vⱼ + k)) = M(d)(vⱼ + k) = aⱼ + k.

*Run-partition property:* B' is pairwise disjoint and covers dom(M'(d)) = dom(M(d)). Subspace-S runs: bijectivity of π|_{V_S(d)} maps the pre-reassembly disjoint covering of V_S(d) to a disjoint covering of V_S(d). Non-S runs: carried verbatim, inheriting disjointness and coverage of dom(M(d)) \ V_S(d). Cross-group disjointness by T10 (ASN-0034).

B' may contain mergeable adjacent pairs; it is covering and disjoint but not necessarily maximal.

---

## R-CANON — CanonicalityOfMergeNormalForm (LEMMA, lemma)

*Hypotheses:* Let B′ be a pairwise-disjoint, covering partition of dom(M'(d)) into valid runs — each (v, a, n) ∈ B′ has n ≥ 1 and satisfies S8-cons under M'(d). Suppose B′ contains no mergeable pair: there is no ordered pair of runs (v, a, n), (v′, a′, n′) ∈ B′ with v′ = v + n (V-adjacent) and a′ = a + n (I-adjacent).

*Conclusion:* B′ is the canonical run decomposition of M'(d) — the unique maximal-run partition guaranteed by S8 (ASN-0036).

*No forward extension:* Take r = (v, a, n) ∈ B′. Suppose w = v + n ∈ dom(M'(d)) and M'(d)(w) = a + n. Then w lies in some r′ = (v′, a′, n′) ∈ B′ at offset i. If i ≥ 1, then v′ + (i − 1) = v + (n − 1) places r ∩ r′ ≠ ∅, contradicting disjointness. So i = 0: v′ = w = v + n and a′ = a + n, making (r, r′) a mergeable pair — contradiction. Hence no forward extension exists.

*No backward extension:* Symmetrically, if u + 1 = v and M'(d)(u) + 1 = a then u lies in some r″ = (v″, a″, n″) at last offset (j = n″ − 1), giving v″ + n″ = v and a″ + n″ = a, making (r″, r) mergeable — contradiction.

*Conclusion:* Every run of B′ is maximal in S8's sense. S8 (ASN-0036) guarantees the partition into maximal runs is unique, so B′ equals the canonical decomposition. ∎

---

## Definition — OrdinalStripping

For a V-position v with subspace(v) = v₁ and #v = m, the *ordinal* is ord(v) = [v₂, ..., vₘ] — the tumbler obtained by stripping the subspace identifier v₁.

Under the depth-2 scope of this ASN (m = 2), ord(v) is a singleton tumbler [k] with k ∈ ℕ⁺, identified with the natural number k throughout displacement and width arithmetic.

---

## Definition — TruncatedSubtraction

`m − n` (partial, m ≥ n) is the unique j ∈ ℕ with [n] ≤ [m] and `shift-or-identity([n], j) = [m]`: OrdinalShift gives j ≥ 1 when m > n, and the identity convention gives j = 0 when m = n.

The right-inverse identity `n + (m − n) = m` holds by the defining equation. The width of an interval |[c, c')| = ord(c') − ord(c) yields a natural number (total here because c < c' under T1 implies ord(c') ≥ ord(c)).

---

## Definition — VExtent

For a correspondence run (v, a, n), the V-extent is:

`V(v, a, n) = {v + k : 0 ≤ k < n}`

S8's consistency clause (*S8-cons*): M(d)(v + k) = a + k within a run. S8's partition property (*run-partition*): every v ∈ dom(M(d)) lies in exactly one run (disjointness and coverage).

---

## Definition — ExtendedAssociativity

For all j, k ∈ ℕ, `(c + j) + k = c + (j + k)`: the j, k ≥ 1 case is TS3 (ShiftComposition, ASN-0034), `shift(shift(v, n₁), n₂) = shift(v, n₁ + n₂)`, and the cases with j = 0 or k = 0 hold by the identity convention `shift(t, 0) := t`.

---

## Definition — SubspaceConfinement (SUBCONF)

For any V-position v with #v = m ≥ 2 and any n ∈ ℕ, `subspace(v + n) = subspace(v)`: an in-region ordinal shift preserves the subspace, by OrdShiftHom (a) of ASN-0036 extended to n = 0. Consequently every correspondence run lies within a single subspace.
