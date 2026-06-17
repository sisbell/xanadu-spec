> **ASN-0084 · Cut-Point Rearrangements** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md)  
> [Condensed statements →](ASN-0084-bundle-projection-displacement.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0084: Cut-Point Rearrangements

*2026-04-10*

This ASN layers a class of arrangement rearrangements over the Strand Model (ASN-0036). The arrangement function M(d) is mutated by transposing regions of V-positions delimited by cut points: three cuts define two adjacent regions that exchange places (the *pivot*); four cuts define two outer regions exchanging across a fixed middle (the *swap*). REARRANGE is confined to the text subspace (S = 1, depth 2); cross-subspace transposition is outside the scope of this ASN. The induced bijection π : dom(M(d)) → dom(M(d)) has a uniform displacement structure on each region, determined by region widths alone. The correspondence-run decomposition guaranteed by S8 (ASN-0036) transforms by splitting at cuts, classifying each run into a region, and reassembling with the per-region displacement.


## State and Vocabulary

We work with the content store C : T ⇀ Val (Σ.C, ASN-0036) and the arrangement function M(d) : T ⇀ T for each document d (Σ.M(d), ASN-0036). The arrangement M(d) is the mutable layer; C is immutable (S0, ASN-0036).

For a V-position v with subspace(v) = v₁ and #v = m, the *ordinal* is ord(v) = [v₂, ..., vₘ] — the tumbler obtained by stripping the subspace identifier v₁ (the complement of ASN-0036's SubspaceProjection, which extracts v₁).

ASN-0036's S8-depth establishes only the lower bound m_s ≥ 2 on each subspace's depth, and ValidFirstInsertionPosition (ASN-0036) leaves the per-subspace depth m_s operator-chosen at initialization (constrained only by m_s ≥ 2). This ASN imposes the additional scope restriction that the text subspace has been initialized at the *minimum* permitted depth m_1 = 2; documents with m_1 > 2 are outside the scope of this ASN. Under this depth-2 restriction, S8-depth gives that every V-position v ∈ V_1(d) satisfies #v = 2 (ordinal depth 1). For parametric uniformity with ASN-0036's V_S(d), [S, k] notation, we use S = 1 throughout and read every appearance of S in this ASN as the text-subspace identifier 1. By D-SEQ (ASN-0036), which characterizes V_1(d) as a sequential range without gaps, V_S(d) = V_1(d) = {[S, k] : 1 ≤ k ≤ N} for some N ≥ 0, and each ord(v) is a singleton tumbler [k] with k ∈ ℕ⁺.

**Identification of singleton tumblers with natural numbers.** At depth 2, we identify the singleton tumbler [k] with the natural number k throughout the displacement and width arithmetic. The identification is licensed as follows. The set of singleton tumblers {[k] : k ∈ ℕ⁺} is in bijection with ℕ⁺ by the map [k] ↔ k (a singleton tumbler is determined by its single component). Under this bijection: T1's strict ordering on tumblers (ASN-0034) restricted to singletons coincides with the standard `<` on ℕ⁺ (lexicographic order on a single component reduces to comparison of that component); for j ≥ 1, OrdinalShift (ASN-0034) gives `shift([k], j) = [k + j]`, with `k + j ∈ ℕ⁺` by addition closure (NAT-closure, ASN-0034); the case j = 0 is covered by the *identity convention* `shift(t, 0) := t`, which we adopt throughout this ASN — it extends OrdinalShift's domain from ℕ⁺ (the foundation's domain) to ℕ, and in particular gives `shift([k], 0) = [k]`. **Truncated subtraction.** We define `m − n` (partial, m ≥ n) as the unique j ∈ ℕ with [n] ≤ [m] and `shift-or-identity([n], j) = [m]`: OrdinalShift gives j ≥ 1 when m > n, and the identity convention gives j = 0 when m = n; existence and uniqueness of j are OrdinalShift's surjectivity onto {[k] : k ≥ n} and TS5 (ShiftAmountMonotonicity, ASN-0034) injectivity in the shift amount. By construction the right-inverse identity `n + (m − n) = m` holds (it is the defining equation `shift([n], m − n) = [m]`). The width of an interval |[c, c')| = ord(c') − ord(c) (this truncated subtraction is total here because c < c' under T1, hence ord(c) < ord(c'), hence ord(c') ≥ ord(c)) yields a natural number. We use this identification implicitly: expressions like `ord(c₀) + j`, `ord(c₁) = ord(c₀) + w_α`, and `w_β = ord(c₂) − ord(c₁)` are read as natural-number arithmetic over the identified domain.

We recall D-CTG (VContiguity, ASN-0036): within each subspace, V-positions form a contiguous ordinal range with no gaps.

**Definition — ArrangementRearrangement.** An *arrangement rearrangement* is a state transition Σ → Σ' in which dom(M'(d)) = dom(M(d)), C' = C (S0, ASN-0036), M'(d') = M(d') for all d' ≠ d, and there exists a bijection π : dom(M(d)) → dom(M'(d)) such that M'(d)(π(v)) = M(d)(v) for all v ∈ dom(M(d)).

We derive that the I-address range is invariant. Since π is a bijection from dom(M(d)) to dom(M'(d)) = dom(M(d)), every u ∈ dom(M'(d)) has the form u = π(v) for exactly one v ∈ dom(M(d)). Therefore: ran(M'(d)) = {M'(d)(u) : u ∈ dom(M'(d))} = {M'(d)(π(v)) : v ∈ dom(M(d))} = {M(d)(v) : v ∈ dom(M(d))} = ran(M(d)). The second equality uses surjectivity of π; the third uses the defining property M'(d)(π(v)) = M(d)(v).

**R-RI — RearrangementReferentialIntegrity (LEMMA).**

*Preconditions:* M(d) is well-defined; M'(d) results from an arrangement rearrangement of M(d) (dom(M'(d)) = dom(M(d)), C' = C, M'(d')= M(d') for d' ≠ d, and there exists a bijection π with M'(d)(π(v)) = M(d)(v)).

*Depends on:* ASN-0036 S3 (referential integrity of the pre-state), C' = C from the rearrangement definition, and the I-address range invariance ran(M'(d)) = ran(M(d)) derived above.

*Postcondition:* ran(M'(d)) ⊆ dom(C').

*Proof.* By the I-address range invariance shown above, ran(M'(d)) = ran(M(d)). By S3 of the pre-state, ran(M(d)) ⊆ dom(C). By C' = C, dom(C) = dom(C'). Chaining: ran(M'(d)) = ran(M(d)) ⊆ dom(C) = dom(C'). ∎

**Invariant preservation.** The following ASN-0036 invariants depend only on `dom(M(d))` and are preserved because `dom(M'(d)) = dom(M(d))`: D-CTG, D-CTG-depth (vacuous under the depth-2 scope of this ASN), D-MIN, D-SEQ, S8-fin, S8a, S8-depth. S2 (arrangement functionality) holds because each u ∈ dom(M'(d)) has u = π(v) for exactly one v (bijectivity), so M'(d)(u) = M(d)(v) is uniquely determined. S3 (referential integrity) is precisely the postcondition of R-RI above — ran(M'(d)) ⊆ dom(C'). **C-transport.** C' = C by the rearrangement definition, so every invariant stated on Σ.C alone transports by identity: S0, S1, S4, S7, S7a, S7b, S7d. S5 (unrestricted sharing) is a permission rather than an obligation; bijectivity of π preserves any pre-existing sharing pattern. Every ASN-0036 invariant except S8 (CorrespondenceRunPartition) is therefore maintained directly by an arrangement rearrangement. *Post-state S8 discharge.* Since `dom(M'(d)) = dom(M(d))` and all of foundation S8's preconditions — S8-fin, S2, S3, S8a, S8-depth — are established in this audit (R-RI for S3), foundation S8 (ASN-0036) applies to M'(d) directly: it supplies the post-state maximal correspondence-run partition and its uniqueness.

Notation: at depth 2, V-positions have the form [S, p]. We write `c₀ + j` for the V-position [S, ord(c₀) + j] — that is, ordinal shift via OrdinalShift (ASN-0034): `c₀ + j = shift(c₀, j)`, consistent with the correspondence-run convention of ASN-0036; the j = 0 case is the identity convention `shift(t, 0) := t` adopted above, so `c₀ + 0 = c₀`.

**Extended Associativity.** For all j, k ∈ ℕ, `(c + j) + k = c + (j + k)`: the j, k ≥ 1 case is TS3 (ShiftComposition, ASN-0034), `shift(shift(v, n₁), n₂) = shift(v, n₁ + n₂)`, and the cases with j = 0 or k = 0 hold by the identity convention. We name this identity *Extended Associativity*. The same identity convention extends OrdShiftHom (a) — `subspace(shift(v, n)) = subspace(v)` (ASN-0036) — to n = 0, since shift(v, 0) = v.

**SUBCONF — Subspace confinement.** For any V-position v with #v = m ≥ 2 and any n ∈ ℕ, `subspace(v + n) = subspace(v)`: an in-region ordinal shift preserves the subspace, by OrdShiftHom (a) of ASN-0036 (extended to n = 0 above). Consequently every correspondence run lies within a single subspace — each of its positions is a shift v + k (0 ≤ k < n) of its V-start v, so all share subspace(v).


## Cut Points and the Region Partition

A *cut sequence* specifies the boundaries of regions to transpose: a cut-point rearrangement is the arrangement rearrangement (in the sense above) whose bijection π is induced by such a tuple. We formalize this as a tuple of tumblers within a single subspace. The cut positions are tumblers satisfying CS1–CS5 below; the last cut c_{n−1} serves as an exclusive upper bound and need not belong to V_S(d).

**Definition — CutSequence.** A *cut sequence* for document d in subspace S is a tuple K = (c₀, c₁, ..., c_{n−1}) of tumblers satisfying:

(CS1) n ∈ {3, 4} — exactly three or four cuts.

(CS2) c₀ < c₁ < ... < c_{n−1} under T1 (ASN-0034) — strictly ordered.

(CS3) subspace(cᵢ) = S = 1 for all i — all cuts in the text subspace.

(CS4) #cᵢ = 2 for all i — depth-2 positions.

(CS5) ord(cᵢ) ≥ 1 for all i — the second component is positive (zero-free, matching the S8a form [S, q] with q ∈ ℕ⁺). This makes each cut cᵢ = [1, ord(cᵢ)] a singleton-identifiable positive ordinal, so ord(cᵢ) ∈ ℕ⁺ and the truncated subtraction on cut ordinals (introduced above) is well-defined on every adjacent cut pair.

The cut positions partition the V-positions of the affected range into regions. For n = 3 (the *pivot*), the cuts define two adjacent regions. For n = 4 (the *swap*), the cuts define two outer regions separated by a middle region.

**Definition — RegionPartition.** Given a cut sequence K for document d in subspace S with V_S(d) ≠ ∅:

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

Pairwise disjointness follows from the strict ordering of cut points and the trichotomy of T1: for any two distinct inter-cut intervals [c_i, c_{i+1}) and [c_j, c_{j+1}) with i < j, every v ∈ [c_i, c_{i+1}) satisfies v < c_{i+1} ≤ c_j (by CS2), so v ∉ [c_j, c_{j+1}) — the intervals are disjoint.

*Exhaustiveness.* For any v ∈ A, we show v lies in exactly one inter-cut interval by T1 trichotomy. For n = 3, A = {v ∈ V_S(d) : c₀ ≤ v < c₂}. T1 trichotomy on (v, c₁) yields three sub-cases: (i) v < c₁ — combined with c₀ ≤ v gives v ∈ [c₀, c₁) = α; (ii) v = c₁ — then v = c₁ < c₂ gives v ∈ [c₁, c₂) = β; (iii) v > c₁ — combined with v < c₂ gives v ∈ [c₁, c₂) = β. Each sub-case places v in exactly one region; disjointness above rules out double-counting. For n = 4, A = {v ∈ V_S(d) : c₀ ≤ v < c₃}. T1 trichotomy on (v, c₁) and (v, c₂) yields five admissible sub-cases (the c_0 ≤ v < c_3 hypothesis rules out v < c_0 and v ≥ c_3): (i) v < c₁ — then c₀ ≤ v < c₁ gives v ∈ α; (ii) v = c₁ — then v = c₁ < c₂ gives v ∈ [c₁, c₂) = μ; (iii) c₁ < v < c₂ — gives v ∈ μ; (iv) v = c₂ — then v = c₂ < c₃ gives v ∈ [c₂, c₃) = β; (v) c₂ < v < c₃ — gives v ∈ β. Each sub-case places v in exactly one region. Each region is a set of consecutive V-positions (by D-CTG, ASN-0036, restricted to the interval between its bounding cuts).

We write w_α = |α|, w_β = |β|, w_μ = |μ| for the region widths.


## Rearrangement Postconditions

The following precondition and postcondition clauses define the rearrangement operation. They are the assumed operational context for the properties introduced in this ASN.

**R-PRE — RearrangePrecondition.**

(i) M(d) is well-defined (the document's arrangement exists).

(ii) V_S(d) ≠ ∅ (the subspace is non-empty — one cannot rearrange nothing).

(iii) The cut sequence K = (c₀, ..., c_{n−1}) satisfies CS1–CS5.

(iv) The affected range lies entirely within the current arrangement:

`(A v : subspace(v) = S ∧ #v = 2 ∧ c₀ ≤ v < c_{n−1} : v ∈ V_S(d))`

Clause (iv) ensures that the affected range is covered: no gap exists within [c₀, c_{n−1}). Combined with D-CTG, this says the entire inter-cut range consists of valid V-positions in V_S(d).

**Consequences of R-PRE.** *Width positivity.* Under R-PRE(iii) and R-PRE(iv), each region width equals a cut-ordinal difference and is a positive natural number: w_α ≥ 1 and w_β ≥ 1 in both forms, and additionally w_μ ≥ 1 when n = 4. By R-PRE(iv) and D-SEQ (ASN-0036), the widths are computable from the cut-point ordinals: w_α = ord(c₁) − ord(c₀); w_β = ord(c₂) − ord(c₁) for n = 3 and ord(c₃) − ord(c₂) for n = 4; w_μ = ord(c₂) − ord(c₁) for n = 4. For each adjacent cut pair (c_i, c_{i+1}): CS5 gives ord(c_i), ord(c_{i+1}) ∈ ℕ⁺ and CS2 gives c_i < c_{i+1}, so by the singleton-tumbler identification (State section) ord(c_i) < ord(c_{i+1}), whence the difference ord(c_{i+1}) − ord(c_i) ≥ 1 is a well-defined positive natural (discharging the m ≥ n precondition of the truncated subtraction). The same identification reduces the membership condition to `c_i ≤ v < c_{i+1} ⟺ ord(c_i) ≤ ord(v) < ord(c_{i+1})`, so R-PRE(iv) places every depth-2 subspace-S position with ordinal in [ord(c_i), ord(c_{i+1})) into V_S(d), and the count of V-positions in [c_i, c_{i+1}) equals ord(c_{i+1}) − ord(c_i) ≥ 1. Instantiating at i = 0 yields w_α ≥ 1; at i = 1 (n = 4) yields w_μ ≥ 1; at i = n − 2 yields w_β ≥ 1.

*Empty right-exterior boundary case (EXT-VAC).* When c_{n−1} ∉ V_S(d), c_{n−1} ∉ dom(M(d)): by D-SEQ (ASN-0036), V_S(d) = {[S, 1], ..., [S, N]}, and any depth-2 subspace-S cut [S, q] with 1 ≤ q ≤ N would lie in V_S(d), so c_{n−1} ∉ V_S(d) lies strictly above the maximum [S, N]; the right-exterior set {v ∈ V_S(d) : v ≥ c_{n−1}} is therefore empty, and since every subspace-S depth-2 element of dom(M(d)) belongs to V_S(d), c_{n−1} ∉ dom(M(d)).


### 3-Cut Pivot Postcondition

Three cuts produce two adjacent regions that exchange places. The operation is: place β's content where α was, then place α's content immediately after.

**Definition — PivotPostcondition.** Given a 3-cut sequence K = (c₀, c₁, c₂) satisfying R-PRE, the *pivot* produces arrangement M'(d) defined by:

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

In words: the first w_β positions of the affected range receive the content that was in β (clause R-P1). The next w_α positions receive the content that was in α (clause R-P2). Everything outside the affected range is unchanged (clause R-EXT). Positions in other subspaces, other documents, and the content store are all preserved.


### 4-Cut Swap Postcondition

Four cuts produce two outer regions separated by a middle region. The semantics is a direct extension of the pivot: place β's content where α was, place μ's content immediately after, place α's content last.

**Definition — SwapPostcondition.** Given a 4-cut sequence K = (c₀, c₁, c₂, c₃) satisfying R-PRE, the *swap* produces M'(d) defined by:

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

The arrangement is: region β content starting at c₀ (clause R-S1), then middle content (clause R-S2), then region α content (clause R-S3). Everything outside [c₀, c₃) is unchanged (clause R-EXT). Positions in other subspaces, other documents, and the content store are all preserved.

**Operation — REARRANGE_K.** REARRANGE_K(Σ, d) is the state transition Σ → Σ' that produces Σ' satisfying PivotPostcondition (when n = 3) or SwapPostcondition (when n = 4) together with the corresponding frame conditions R-FRAME-P (n = 3) or R-FRAME-S (n = 4). The two cases of n are mutually exclusive (CS1) and exhaustive over admissible cut sequences, so REARRANGE_K is a single operation whose postcondition specializes by the cut count. *Partiality.* REARRANGE_K is partial, defined exactly where R-PRE(K) holds against Σ.M(d).


## Non-S Subspace Invariance

REARRANGE_K affects only the subspace-S portion of M(d); positions in any other subspace pass through unchanged.

**R-NS — NonSubspaceInvariance (LEMMA).** *(NS-M) Pointwise identity on non-S.* For every v ∈ dom(M(d)) with subspace(v) ≠ S: M'(d)(v) = M(d)(v).

*Proof.* The frame condition R-FRAME-P(a) (n = 3) or R-FRAME-S(a) (n = 4) gives M'(d)(v) = M(d)(v) directly for every v ∈ dom(M(d)) with subspace(v) ≠ S. ∎


## Postcondition Well-Definedness

**R-PIV — PivotWellDefined (LEMMA, supporting).** The pivot postcondition defines a total function on dom(M(d)) (each position is assigned exactly one I-address).

*Proof.* We must show: (a) every v ∈ dom(M(d)) falls under exactly one clause, and (b) the right-hand sides are well-defined.

For v ∈ dom(M(d)) with subspace(v) ≠ S: R-FRAME-P(a) assigns M'(d)(v) = M(d)(v), and no other clause applies (R-EXT, R-P1, R-P2 operate only on subspace S positions).

It remains to show that every v ∈ V_S(d) falls under exactly one of R-EXT, R-P1, R-P2.

For (a): the positions addressed by R-EXT are those outside [c₀, c₂). The positions addressed by R-P1 are {c₀ + j : 0 ≤ j < w_β}. At depth 2, c₀ = [S, p] and c₀ + j = [S, p + j], so these positions have ordinals p, p + 1, ..., p + w_β − 1. By R-PRE(iv), all V-positions with subspace S, depth 2, and ordinal in [p, p + w_α + w_β) lie in V_S(d), so the R-P1 positions are distinct elements of V_S(d). The positions addressed by R-P2 are {c₀ + w_β + j : 0 ≤ j < w_α} = {[S, p + w_β + j] : 0 ≤ j < w_α}, with ordinals p + w_β, ..., p + w_β + w_α − 1; the compound destination c₀ + w_β + j is read left-associatively as (c₀ + w_β) + j (Extended Associativity).

The R-P1 ordinal range is [p, p + w_β). The R-P2 ordinal range is [p + w_β, p + w_β + w_α). Both ranges are non-empty (since w_β ≥ 1 and w_α ≥ 1 by Width positivity), and they are disjoint because [p, p + w_β) ∩ [p + w_β, p + w_β + w_α) = ∅ — the half-open intervals meet at the shared endpoint p + w_β, which is included only in the second range. Their union is [p, p + w_β + w_α) = [p, p + w_α + w_β). And p + w_α + w_β is the ordinal of c₂ (since |[c₀, c₂)| = w_α + w_β, and by R-PRE(iv) the ordinals in [c₀, c₂) lie consecutively in V_S(d)). So the union of R-P1 and R-P2 covers exactly [c₀, c₂) ∩ V_S(d). Together with R-EXT (covering V_S(d) \ [c₀, c₂)), every position is covered exactly once.

For (b): the right-hand sides reference M(d)(c₁ + j) for j < w_β and M(d)(c₀ + j) for j < w_α. By R-PRE(iv), all positions in [c₀, c₂) are in V_S(d) ⊆ dom(M(d)). The positions c₁ + j for j < w_β have ordinals in [ord(c₁), ord(c₂)) = the ordinals of β. The positions c₀ + j for j < w_α have ordinals in [ord(c₀), ord(c₁)) = the ordinals of α. Both sets are subsets of [c₀, c₂) ∩ V_S(d) ⊆ dom(M(d)). ∎


**R-SWP — SwapWellDefined (LEMMA, supporting).** The swap postcondition defines a total function on dom(M(d)).

*Proof.* We must show: (a) every v ∈ dom(M(d)) falls under exactly one clause, and (b) the right-hand sides are well-defined.

For v ∈ dom(M(d)) with subspace(v) ≠ S: R-FRAME-S(a) assigns M'(d)(v) = M(d)(v), and no other clause applies.

It remains to show that every v ∈ V_S(d) falls under exactly one of R-EXT, R-S1, R-S2, R-S3.

For (a): let p = ord(c₀). The positions addressed by each clause have the following ordinal ranges:

- R-EXT: ordinals outside [p, p + w_α + w_μ + w_β), i.e., ord(v) < p or ord(v) ≥ p + w_α + w_μ + w_β.
- R-S1: {c₀ + j : 0 ≤ j < w_β}, ordinals [p, p + w_β).
- R-S2: {c₀ + w_β + j : 0 ≤ j < w_μ}, ordinals [p + w_β, p + w_β + w_μ); the compound destination c₀ + w_β + j is read left-associatively as (c₀ + w_β) + j (Extended Associativity).
- R-S3: {c₀ + w_β + w_μ + j : 0 ≤ j < w_α}, ordinals [p + w_β + w_μ, p + w_β + w_μ + w_α); the compound destination c₀ + w_β + w_μ + j is read left-associatively as ((c₀ + w_β) + w_μ) + j (Extended Associativity).

Pairwise disjointness: the four ordinal ranges are [p, p + w_β), [p + w_β, p + w_β + w_μ), [p + w_β + w_μ, p + w_β + w_μ + w_α), and the exterior. Since w_α ≥ 1, w_β ≥ 1, and w_μ ≥ 1 by Width positivity (n = 4 case), the half-open intervals are non-empty and their left endpoints are strictly increasing: p < p + w_β < p + w_β + w_μ < p + w_β + w_μ + w_α. Hence no two intervals overlap, and none overlaps with the exterior.

Exhaustiveness: the union of R-S1, R-S2, R-S3 covers ordinals [p, p + w_β + w_μ + w_α). And p + w_β + w_μ + w_α = p + w_α + w_μ + w_β = ord(c₃) (since |[c₀, c₃)| = w_α + w_μ + w_β and by R-PRE(iv) the ordinals in [c₀, c₃) lie consecutively in V_S(d)). So the union of all four clauses covers V_S(d).

For (b): the right-hand sides reference M(d)(c₂ + j) for j < w_β (ordinals of β), M(d)(c₁ + j) for j < w_μ (ordinals of μ), and M(d)(c₀ + j) for j < w_α (ordinals of α). All three sets are subsets of [c₀, c₃) ∩ V_S(d) ⊆ dom(M(d)) by R-PRE(iv). ∎


## The 3-Cut Pivot Permutation

**R-PPERM — PivotPermutation (LEMMA).** The *cut-point-induced bijection* π : dom(M(d)) → dom(M'(d)) satisfying M'(d)(π(v)) = M(d)(v) is the specific bijection determined by the cut sequence K and the region partition. The formula is:

```
         ⎧ v                   if subspace(v) ≠ S                  (non-S)
         ⎪ v                   if v ∈ V_S(d) and (v < c₀ or v ≥ c₂)  (subspace-S exterior)
π(v) =  ⎨ c₀ + w_β + j        if v = c₀ + j, 0 ≤ j < w_α              (α → end)
         ⎩ c₀ + j              if v = c₁ + j, 0 ≤ j < w_β              (β → start)
```

The subspace-S exterior, α, and β branches partition V_S(d), so the four-case piecewise definition is total on dom(M(d)).

*Proof.* We verify M'(d)(π(v)) = M(d)(v) in each case. For v ∈ dom(M(d)) with subspace(v) ≠ S: π(v) = v by the non-S clause of the definition above, and M'(d)(v) = M(d)(v) by R-NS(NS-M). For v ∈ V_S(d) with v < c₀ or v ≥ c₂: π(v) = v, and M'(d)(v) = M(d)(v) by R-EXT. For v = c₀ + j in α: π(v) = c₀ + w_β + j, and M'(d)(c₀ + w_β + j) = M(d)(c₀ + j) = M(d)(v) by R-P2. For v = c₁ + j in β: π(v) = c₀ + j, and M'(d)(c₀ + j) = M(d)(c₁ + j) = M(d)(v) by R-P1.

Injectivity: within each case, the mapping is injective (the exterior is the identity; the α case maps distinct j to distinct c₀ + w_β + j; the β case maps distinct j to distinct c₀ + j). Across cases: the four image sets — {v ∈ dom(M(d)) : subspace(v) ≠ S}, V_S(d) \ [c₀, c₂), {c₀ + w_β + j : 0 ≤ j < w_α}, {c₀ + j : 0 ≤ j < w_β} — are pairwise disjoint (the first is disjoint from the rest by subspace separation; the remaining three are pairwise disjoint as shown in R-PIV). Surjectivity: π is an injection from dom(M(d)) into itself, and dom(M(d)) is finite (S8-fin of ASN-0036); on a finite set, every self-injection is a bijection, so π is surjective. ∎

The pivot postcondition preserves dom(M(d)) (R-PIV), preserves C (R-FRAME-P(c)), and admits the bijection π satisfying M'(d)(π(v)) = M(d)(v) (R-PPERM); it therefore constitutes an arrangement rearrangement, and the invariant preservation established above applies.


## The 4-Cut Swap Permutation

**R-SPERM — SwapPermutation (LEMMA).** The *cut-point-induced bijection* π satisfying M'(d)(π(v)) = M(d)(v) is the specific bijection determined by the 4-cut sequence K and the regions α, μ, β. The formula is:

```
         ⎧ v                        if subspace(v) ≠ S                     (non-S)
         ⎪ v                        if v ∈ V_S(d) and (v < c₀ or v ≥ c₃)     (subspace-S exterior)
         ⎪ c₀ + w_β + w_μ + j       if v = c₀ + j, 0 ≤ j < w_α                (α → end)
π(v) =  ⎨ c₀ + w_β + j             if v = c₁ + j, 0 ≤ j < w_μ                (μ → middle)
         ⎩ c₀ + j                   if v = c₂ + j, 0 ≤ j < w_β                (β → start)
```

The subspace-S exterior, α, μ, and β branches partition V_S(d), so the five-case piecewise definition is total on dom(M(d)).

*Proof.* We verify M'(d)(π(v)) = M(d)(v) in each case.

For v ∈ dom(M(d)) with subspace(v) ≠ S: π(v) = v by the non-S clause of the definition above, and M'(d)(v) = M(d)(v) by R-NS(NS-M).

For v ∈ V_S(d) with v < c₀ or v ≥ c₃: π(v) = v, and M'(d)(v) = M(d)(v) by R-EXT.

For v = c₀ + j in α (0 ≤ j < w_α): π(v) = c₀ + w_β + w_μ + j, and M'(d)(c₀ + w_β + w_μ + j) = M(d)(c₀ + j) = M(d)(v) by R-S3.

For v = c₁ + j in μ (0 ≤ j < w_μ): π(v) = c₀ + w_β + j, and M'(d)(c₀ + w_β + j) = M(d)(c₁ + j) = M(d)(v) by R-S2.

For v = c₂ + j in β (0 ≤ j < w_β): π(v) = c₀ + j, and M'(d)(c₀ + j) = M(d)(c₂ + j) = M(d)(v) by R-S1.

Injectivity: within each case, the mapping is injective (the exterior is the identity; the α case maps distinct j to distinct c₀ + w_β + w_μ + j; the μ case maps distinct j to distinct c₀ + w_β + j; the β case maps distinct j to distinct c₀ + j). Across cases: the five image sets — {v ∈ dom(M(d)) : subspace(v) ≠ S}, V_S(d) \ [c₀, c₃), {c₀ + w_β + w_μ + j : 0 ≤ j < w_α}, {c₀ + w_β + j : 0 ≤ j < w_μ}, {c₀ + j : 0 ≤ j < w_β} — are pairwise disjoint (the first is disjoint from the rest by subspace separation; the remaining four are pairwise disjoint as shown in R-SWP). Surjectivity: π is an injection from dom(M(d)) into itself, and dom(M(d)) is finite (S8-fin of ASN-0036); on a finite set, every self-injection is a bijection, so π is surjective. ∎

The swap postcondition preserves dom(M(d)) (R-SWP), preserves C (R-FRAME-S(c)), and admits the bijection π satisfying M'(d)(π(v)) = M(d)(v) (R-SPERM); it therefore constitutes an arrangement rearrangement, and the invariant preservation established above applies.

The two forms are distinct primitives: the 3-cut pivot transposes two *adjacent* regions, while the 4-cut swap transposes two regions separated by at least one middle position (R-PRE(iv) together with CS2–CS4 forces w_μ ≥ 1, by the Width positivity consequence).


## Displacement Analysis

The permutations R-PPERM and R-SPERM can be characterized by ordinal displacements — how far each position moves within its subspace. Each magnitude is reported as a *direction* (forward, backward, or fixed) together with a natural-number ordinal distance, read off the explicit R-PPERM/R-SPERM formulas. The truncated subtraction (defined above) supplies the distances, all on its defined domain of single-component depth-2 ordinals.

*Remark (per-region displacement uniformity).* Read off the explicit R-PPERM and R-SPERM formulas, the offset j within a region cancels, so every position in a region moves by the same direction and distance — the displacement depends only on the region widths, not on the position's location within the region. Writing the displacement as (direction, distance):

- *Non-S domain and subspace-S exterior:* π(v) = v (the non-S clause of R-PPERM/R-SPERM on the non-S domain, the exterior clause on the subspace-S exterior), so the displacement is *fixed* (distance 0).
- *3-cut.* On α, v = c₀ + j (0 ≤ j < w_α) maps to c₀ + w_β + j, so ord(π(v)) − ord(v) = w_β: *forward by w_β*. On β, v = c₁ + j (0 ≤ j < w_β) maps to c₀ + j, with ord(v) − ord(π(v)) = w_α: *backward by w_α*.
- *4-cut.* On α, v = c₀ + j maps to c₀ + w_β + w_μ + j: *forward by w_β + w_μ*. On β, v = c₂ + j maps to c₀ + j, with ord(v) − ord(π(v)) = w_α + w_μ: *backward by w_α + w_μ*. On μ, v = c₁ + j maps to c₀ + w_β + j; comparing ord(π(v)) = ord(c₀) + w_β + j against ord(v) = ord(c₀) + w_α + j reduces to comparing w_β and w_α: *forward by w_β − w_α* when w_β > w_α, *backward by w_α − w_β* when w_β < w_α, and *fixed* when w_β = w_α.


## Correspondence-Run Decomposition Transformation

We recall from S8 (CorrespondenceRunPartition, ASN-0036) that for every v ∈ dom(M(d)) there exists a unique correspondence run (v_s, a_s, n) with v ∈ {v_s + k : 0 ≤ k < n} and M(d)(v_s + k) = a_s + k for all 0 ≤ k < n. Here the second operand a_s of `a_s + k` is an I-address (element-level, zeros = 3 by S7b), so `+` on I-addresses denotes `shift(a_s, k)` per ASN-0036's S8 run convention — the last-component ordinal increment, valid at the I-address's depth per OrdinalShift (ASN-0034). Equivalently, S8 yields a finite partition of dom(M(d)) into correspondence runs. We layer three new operations (Split, Merge, and a canonical decomposition) over the foundation's runs. Throughout this section, when we say *run* we mean a correspondence run (v, a, n) with n ≥ 1, and write V(v, a, n) = {v + k : 0 ≤ k < n} for its V-extent. S8's consistency clause, applied per-position, gives *S8-cons*: M(d)(v + k) = a + k within a run. S8's partition property gives *run-partition*: every v ∈ dom(M(d)) lies in exactly one run (disjointness and coverage).

**Split.** Given a run b = (v, a, n) under some arrangement A and an interior offset c with 1 ≤ c < n, the *split* at c produces two runs: (v, a, c) and (v + c, a + c, n − c). Their V-extents (ordinal ranges [ord(v), ord(v) + c) and [ord(v) + c, ord(v) + n)) are disjoint and partition b's V-extent.

Both pieces inherit S8-cons (consistency under A). For the first piece (v, a, c), we need A(v + k) = a + k for 0 ≤ k < c; this holds by restricting the original S8-cons to the subrange k < c < n. For the second piece (v + c, a + c, n − c), we need A((v + c) + k) = (a + c) + k for 0 ≤ k < n − c. By Extended Associativity (above), (v + c) + k = v + (c + k) for all k ∈ ℕ. Since c + k < n, the original S8-cons yields A(v + (c + k)) = a + (c + k). Extended Associativity likewise gives (a + c) + k = a + (c + k), applied to the I-address a per the preamble. This completes the derivation: A((v + c) + k) = a + (c + k) = (a + c) + k.

**Merge.** Two runs (v₁, a₁, n₁) and (v₂, a₂, n₂) under arrangement A are *mergeable* when v₂ = v₁ + n₁ (V-adjacent) and a₂ = a₁ + n₁ (I-adjacent). The merged run is (v₁, a₁, n₁ + n₂). It inherits S8-cons: for 0 ≤ k < n₁ it holds by run 1, and for n₁ ≤ k < n₁ + n₂, writing k = n₁ + j with 0 ≤ j < n₂, Extended Associativity gives v₁ + k = (v₁ + n₁) + j = v₂ + j, so A(v₁ + k) = A(v₂ + j) = a₂ + j = (a₁ + n₁) + j = a₁ + k by run 2 and the I-adjacency a₂ = a₁ + n₁ (at the junction k = n₁ this is exactly A(v₂) = a₂ = a₁ + n₁).

**Canonical decomposition.** The *canonical run decomposition* of an arrangement is S8's (CorrespondenceRunPartition, ASN-0036) unique maximal-run partition.

**R-COMM — PermutationShiftCommutativity (LEMMA).** Let π be a cut-point permutation (R-PPERM or R-SPERM) for a cut sequence K satisfying R-PRE. For any V-position v ∈ dom(M(d)) and offset k ≥ 0 such that v + k ∈ dom(M(d)) and v, v + k lie in the same region — where the regions are the non-S subspace ({v ∈ dom(M(d)) : subspace(v) ≠ S}), the subspace-S exterior, α, μ, or β:

`π(v + k) = π(v) + k`

In words: the cut-point permutation commutes with ordinal shift within each region. Every position in a region receives the same ordinal displacement, so shifting within the region before or after applying π yields the same result.

*Proof.* We verify each region case using the explicit R-PPERM and R-SPERM formulas, with associativity of natural-number addition at the ordinal level as the sole algebraic tool. In each subspace-S case, the same-region hypothesis bounds the shifted offset j' + k inside the region's width, justifying application of the corresponding R-PPERM or R-SPERM branch.

*Non-S subspace (both forms):* By the hypothesis, v and v + k are both non-S; the non-S clause of R-PPERM/R-SPERM gives π(v) = v and π(v + k) = v + k, hence π(v + k) = v + k = π(v) + k.

*Subspace-S exterior (both forms):* π(v + k) = v + k = π(v) + k, since π is the identity on the exterior.

*3-cut α:* v = c₀ + j' for some 0 ≤ j' < w_α. The same-region hypothesis "v + k ∈ α" places v + k = c₀ + (j' + k) with 0 ≤ j' + k < w_α (because α is defined as {c₀ + i : 0 ≤ i < w_α}, and the bijection between α's positions and offsets in [0, w_α) is supplied by the singleton-tumbler identification of V-positions with their ordinals). This bound discharges R-PPERM's α-branch precondition, yielding π(v + k) = c₀ + w_β + (j' + k). Also π(v) + k = (c₀ + w_β + j') + k = c₀ + w_β + (j' + k) by associativity (Extended Associativity).

*3-cut β:* v = c₁ + j' for some 0 ≤ j' < w_β. The same-region hypothesis "v + k ∈ β" gives 0 ≤ j' + k < w_β, discharging R-PPERM's β-branch precondition. Then v + k = c₁ + (j' + k), and by R-PPERM: π(v + k) = c₀ + (j' + k). Also π(v) + k = (c₀ + j') + k = c₀ + (j' + k) by associativity.

*4-cut α:* v = c₀ + j' for some 0 ≤ j' < w_α. The same-region hypothesis "v + k ∈ α" gives 0 ≤ j' + k < w_α, discharging R-SPERM's α-branch precondition. Then v + k = c₀ + (j' + k), and by R-SPERM: π(v + k) = c₀ + w_β + w_μ + (j' + k). Also π(v) + k = (c₀ + w_β + w_μ + j') + k = c₀ + w_β + w_μ + (j' + k) by associativity.

*4-cut μ:* v = c₁ + j' for some 0 ≤ j' < w_μ. The same-region hypothesis "v + k ∈ μ" gives 0 ≤ j' + k < w_μ, discharging R-SPERM's μ-branch precondition. Then v + k = c₁ + (j' + k), and by R-SPERM: π(v + k) = c₀ + w_β + (j' + k). Also π(v) + k = (c₀ + w_β + j') + k = c₀ + w_β + (j' + k) by associativity.

*4-cut β:* v = c₂ + j' for some 0 ≤ j' < w_β. The same-region hypothesis "v + k ∈ β" gives 0 ≤ j' + k < w_β, discharging R-SPERM's β-branch precondition. Then v + k = c₂ + (j' + k), and by R-SPERM: π(v + k) = c₀ + (j' + k). Also π(v) + k = (c₀ + j') + k = c₀ + (j' + k) by associativity. ∎

**R-BLK — RunDecompositionTransformation (LEMMA).** Let B = {b₁, ..., bₘ} be a run partition of M(d) (per S8) — including runs whose V-extents lie in V_S(d) and runs whose V-extents lie in subspaces other than S. Let the cut sequence K have cut positions c₀, ..., c_{n−1}. The rearranged arrangement M'(d) admits a run partition B' — disjoint and covering, but not necessarily the maximal (canonical) decomposition — obtained by:

*Phase 1: Split.* Process cut positions in index order (c₀, c₁, ..., c_{n−1}), maintaining the partition as it is progressively refined. For each cut position cᵢ, classify by whether cᵢ falls within some run's V-extent:

- *Interior of a run:* if cᵢ ∈ V(bₖ) for some bₖ = (vₖ, aₖ, nₖ) with cᵢ ≠ vₖ, split bₖ at the offset c = ord(cᵢ) − ord(vₖ), producing (vₖ, aₖ, c) and (vₖ + c, aₖ + c, nₖ − c). The two new runs partition the V-extent of the original.
- *Boundary of a run:* if cᵢ ∈ V(bₖ) and cᵢ = vₖ, no split is needed — the cut already coincides with a run boundary.
- *Outside ⋃_k V(bₖ):* no split is performed. By CS2–CS4 and R-PRE(iv), c₀, …, c_{n−2} ∈ V_S(d) ⊆ ⋃_k V(bₖ); only c_{n−1} may fall outside, and EXT-VAC then gives c_{n−1} ∉ dom(M(d)) with empty right exterior, so no run straddles it.

*Non-S runs are not split.* Let b = (v_b, a_b, n_b) ∈ B with subspace(v_b) = S' ≠ S. By SUBCONF, every V-position v_b + k satisfies subspace(v_b + k) = subspace(v_b) = S' ≠ S, so V(b) ⊆ dom(M(d)) \ V_S(d). By CS3 every cut position lies in subspace S, so no cut falls in V(b) and Phase 1 never splits b.

*Phase 2: Classify.* Each run in the post-split partition lies entirely within one of R-COMM's regions — the non-S subspace (V-extent in some subspace S' ≠ S), the subspace-S exterior (`v < c₀` or `v ≥ c_{n−1}`), α, μ if 4-cut, or β — because no run crosses a cut boundary (subspace-S runs are split at S-subspace cuts, and non-S runs are entirely contained in their subspace, shown above).

*Phase 3: Reassemble.* Apply the permutation π to each run's V-start. Each run (vₖ, aₖ, nₖ) in the post-split, post-classify partition becomes (π(vₖ), aₖ, nₖ): the V-start is replaced by π(vₖ); the I-start aₖ and width nₖ are preserved verbatim. Per region:

- Non-S runs: carried through unchanged. By the non-S clause of R-PPERM/R-SPERM, π is the identity on V(b), so each non-S run b = (v_b, a_b, n_b) maps to itself (v_b, a_b, n_b).
- Exterior runs: π(vₖ) = vₖ by the subspace-S exterior clause of R-PPERM/R-SPERM; the triple carries through unchanged.
- α runs: π(vₖ) is computed by the α-branch of R-PPERM (3-cut) or R-SPERM (4-cut).
- β runs: π(vₖ) is computed by the β-branch of R-PPERM (3-cut) or R-SPERM (4-cut).
- μ runs (4-cut only): π(vₖ) is computed by the μ-branch of R-SPERM.

*Same-region discharge of the commutation identity.* By Phase 2, each post-split run (vⱼ, aⱼ, nⱼ) lies entirely within one region, so it satisfies the same-region precondition of R-COMM, which gives π(vⱼ + k) = π(vⱼ) + k for every 0 ≤ k < nⱼ uniformly across all regions.

*I-start, width, and contiguity of reassembled runs.* The commutation identity carries the consecutive V-positions vⱼ, vⱼ + 1, ..., vⱼ + (nⱼ − 1) to the consecutive V-positions π(vⱼ), π(vⱼ) + 1, ..., π(vⱼ) + (nⱼ − 1): each reassembled run (π(vⱼ), aⱼ, nⱼ) therefore occupies a contiguous V-position range with its width intact. Its I-start is aⱼ by the permutation defining property M'(d)(π(vⱼ)) = M(d)(vⱼ) = aⱼ. Each reassembled run is thus a valid run.

The resulting runs satisfy S8-cons (consistency under M'(d)). *Subspace-S runs:* for each reassembled run (π(vⱼ), aⱼ, nⱼ) and 0 ≤ k < nⱼ: M'(d)(π(vⱼ) + k) = M'(d)(π(vⱼ + k)) = M(d)(vⱼ + k) = aⱼ + k, where the first equality is the commutation identity discharged above and the second is the permutation defining property M'(d)(π(v)) = M(d)(v). *Non-S runs:* since π is the identity on V(b), each non-S run inherits S8-cons under M'(d): for 0 ≤ k < n_b, M'(d)(v_b + k) = M(d)(v_b + k) = a_b + k.

Run-partition (disjointness and coverage) for M'(d): π is a bijection on dom(M(d)) = dom(M'(d)), and π restricts to the identity on the non-S part of dom(M(d)) (the non-S clause of R-PPERM/R-SPERM) and to a bijection on V_S(d) (R-PPERM/R-SPERM). The V-extents of the reassembled subspace-S runs are pairwise disjoint and cover V_S(d) (from the partition property of the pre-reassembly subspace-S partition and bijectivity of π|_{V_S(d)}); the V-extents of the carried-over non-S runs are pairwise disjoint and cover dom(M(d)) \ V_S(d) (inherited from the pre-state partition, since non-S runs carry through unchanged). Pairwise disjointness across the two groups holds because subspace-S and non-S V-extents lie in distinct subspaces (T10 of ASN-0034: non-nesting prefixes generate disjoint subtrees). Together these establish that B′ is a run partition of dom(M'(d)) — its runs are pairwise disjoint and their union covers dom(M'(d)).

This completes R-BLK: B' is a valid run partition of M'(d). B′ may contain mergeable adjacent pairs even when the pre-state partition B had none, since the rearrangement can bring runs of common origin into V- and I-adjacency; B′ is therefore covering and disjoint but not necessarily maximal.

**R-CANON — CanonicalityOfMergeNormalForm (LEMMA).** *Let B′ be a pairwise-disjoint, covering partition of dom(M'(d)) into valid runs — each (v, a, n) ∈ B′ has n ≥ 1 and satisfies S8-cons under M'(d). Suppose B′ contains no mergeable pair: there is no ordered pair of runs (v, a, n), (v′, a′, n′) ∈ B′ with v′ = v + n (V-adjacent) and a′ = a + n (I-adjacent). Then B′ is the canonical run decomposition of M'(d) — the unique maximal-run partition guaranteed by S8 (ASN-0036).*

We first record facts used in both directions. Every run lies within a single subspace (SUBCONF). Within that subspace all V-positions share one depth m (S8-depth, ASN-0036) — for a non-text subspace m may exceed 2 — and shift alters only the m-th (last) component (OrdinalShift, ASN-0034). Writing ord(p) = [p₂, ..., pₘ] for the subspace-stripped ordinal (the State section's ord, now read at the run's depth m), incrementing or decrementing ord by a natural number means shifting that last component, and two positions of one subspace are equal iff their ords agree (T3, ASN-0034: they already share the subspace component p₁ and the depth m, so component-wise agreement on positions 2..m forces equality). Hence each position along a run's shift-line has a unique immediate ordinal predecessor and successor, and shift is strictly increasing (TS4) and injective in its amount (TS5, ASN-0034). Consequently any two runs of B′ that share a V-position coincide, by the disjointness of B′.

*No run admits a forward extension.* Take r = (v, a, n) ∈ B′ and suppose, for contradiction, it admits a forward extension: w = v + n satisfies w ∈ dom(M'(d)) and M'(d)(w) = a + n. Since B′ covers dom(M'(d)), w lies in some run r′ = (v′, a′, n′) ∈ B′ at offset i, w = v′ + i with 0 ≤ i < n′. Suppose i ≥ 1. Then v′ + (i − 1) ∈ V(r′) has ordinal ord(w) − 1 = ord(v) + n − 1 = ord(v + (n − 1)); by uniqueness of the immediate predecessor, v′ + (i − 1) = v + (n − 1), the last V-position of r (offset n − 1, valid as n ≥ 1). This position lies in both r and r′, so r = r′; but then w = v + i with i < n′ = n contradicts w = v + n by injectivity of shift in its amount. Hence i = 0: w = v′ is the *start* of r′, and a′ = M'(d)(v′) = M'(d)(w) = a + n. Thus v′ = v + n and a′ = a + n — the pair (r, r′) is mergeable (and r ≠ r′ since v′ = v + n > v by TS4), contradicting the hypothesis. So r admits no forward extension.

*No run admits a backward extension.* Symmetrically, suppose r = (v, a, n) ∈ B′ admits a backward extension: some u ∈ dom(M'(d)) satisfies u + 1 = v (v is the immediate ordinal successor of u, ord(u) = ord(v) − 1) and M'(d)(u) + 1 = a. The covering partition places u in some run r″ = (v″, a″, n″) at offset j, u = v″ + j. Suppose j < n″ − 1. Then v″ + (j + 1) ∈ V(r″) has ordinal ord(u) + 1 = ord(v), so v″ + (j + 1) = v ∈ r ∩ r″ and r = r″; but then u, of ordinal ord(v) − 1, would lie below r's least ordinal ord(v) — impossible inside r. Hence j = n″ − 1: u is the *last* position of r″, so by S8-cons M'(d)(u) = a″ + (n″ − 1). The predecessor conditions then give v = u + 1 = (v″ + (n″ − 1)) + 1 = v″ + n″ and a = M'(d)(u) + 1 = (a″ + (n″ − 1)) + 1 = a″ + n″. Thus the pair (r″, r) is mergeable, contradicting the hypothesis. So r admits no backward extension.

*Conclusion.* Every run of B′ is maximal in S8's sense, so B′ is a partition of dom(M'(d)) into maximal runs. S8 (ASN-0036) guarantees that the partition into maximal runs is unique — it is, by definition, the canonical decomposition. Hence B′ equals the canonical decomposition. ∎

*Termination and confluence of merging.* Each application of Merge replaces two runs by one, strictly decreasing the (finite, ≥ 1) run count, so every sequence of merges terminates. Merge preserves the partition hypotheses — coverage and disjointness survive because the fused V-extents union exactly, and the merged run satisfies S8-cons (Merge, above) — so the terminal partition, which by construction admits no mergeable pair, satisfies R-CANON's hypotheses and is therefore the canonical decomposition. Because that decomposition is unique (S8), the terminal result is independent of the order in which mergeable pairs are fused: iterated merging is confluent, and the canonical partition is its unique normal form.


## Worked Example: 3-Cut Pivot on a 5-Position Document

We trace a concrete 3-cut pivot to verify the postconditions against explicit values. Let document d have subspace S = 1 with V_S(d) = {[1,1], [1,2], [1,3], [1,4], [1,5]}, and let the arrangement be:

```
M(d)([1,1]) = 3.0.1.0.1.0.1.1    (I-address A)
M(d)([1,2]) = 3.0.1.0.1.0.1.2    (I-address B)
M(d)([1,3]) = 3.0.1.0.1.0.1.3    (I-address C)
M(d)([1,4]) = 5.0.2.0.1.0.1.1    (I-address D)
M(d)([1,5]) = 5.0.2.0.1.0.1.2    (I-address E)
```

Content A–C originates from document 3.0.1.0.1 (origin 3.0.1.0.1); D–E from document 5.0.2.0.1 (origin 5.0.2.0.1). The canonical run partition has two runs: b₁ = ([1,1], 3.0.1.0.1.0.1.1, 3) and b₂ = ([1,4], 5.0.2.0.1.0.1.1, 2).

We apply a 3-cut pivot with K = ([1,2], [1,4], [1,5]): c₀ = [1,2], c₁ = [1,4], c₂ = [1,5]. The affected range is [c₀, c₂) = {[1,2], [1,3], [1,4]}. Region α = {[1,2], [1,3]} (w_α = 2), region β = {[1,4]} (w_β = 1).

**R-PRE verification.** (i) M(d) well-defined. (ii) V_S(d) ≠ ∅. (iii) CS1: n = 3; CS2: [1,2] < [1,4] < [1,5]; CS3: all subspace 1; CS4: all depth 2; CS5: ordinals 2, 4, 5 ≥ 1. (iv) All positions in [[1,2], [1,5)) are in V_S(d). Width positivity: w_α = 2 ≥ 1, w_β = 1 ≥ 1 (consequence). ✓

**Applying the postconditions.** We compute M'(d) position by position:

R-EXT: M'(d)([1,1]) = M(d)([1,1]) = A. M'(d)([1,5]) = M(d)([1,5]) = E.

R-P1 (j = 0): M'(d)(c₀ + 0) = M'(d)([1,2]) = M(d)(c₁ + 0) = M(d)([1,4]) = D.

R-P2 (j = 0): M'(d)(c₀ + 1 + 0) = M'(d)([1,3]) = M(d)(c₀ + 0) = M(d)([1,2]) = B.

R-P2 (j = 1): M'(d)(c₀ + 1 + 1) = M'(d)([1,4]) = M(d)(c₀ + 1) = M(d)([1,3]) = C.

**Result:**

```
M'(d)([1,1]) = A     (exterior, unchanged)
M'(d)([1,2]) = D     (was β, now at start of affected range)
M'(d)([1,3]) = B     (was α position 1, shifted forward by w_β = 1)
M'(d)([1,4]) = C     (was α position 2, shifted forward by w_β = 1)
M'(d)([1,5]) = E     (exterior, unchanged)
```

**R-PPERM verification.** The permutation π: π([1,1]) = [1,1] (exterior), π([1,2]) = [1,3] (α: c₀ + 0 → c₀ + w_β + 0 = [1,3]), π([1,3]) = [1,4] (α: c₀ + 1 → c₀ + w_β + 1 = [1,4]), π([1,4]) = [1,2] (β: c₁ + 0 → c₀ + 0 = [1,2]), π([1,5]) = [1,5] (exterior). We check: M'(d)(π([1,2])) = M'(d)([1,3]) = B = M(d)([1,2]) ✓. M'(d)(π([1,4])) = M'(d)([1,2]) = D = M(d)([1,4]) ✓.

**R-RI verification.** ran(M'(d)) = {A, D, B, C, E} = ran(M(d)) (the same five I-addresses, only their V-position assignments rearranged). Since ran(M(d)) ⊆ dom(C) by S3 of the pre-state and C' = C, ran(M'(d)) ⊆ dom(C'). ✓

**Displacement verification.** Reading each position's displacement as (direction, ordinal distance), computed from ord(π(v)) − ord(v): [1,1] fixed (exterior left); [1,2] forward 3 − 2 = 1 = w_β (α, j = 0); [1,3] forward 4 − 3 = 1 = w_β (α, j = 1); [1,4] backward 4 − 2 = 2 = w_α (β, j = 0); [1,5] fixed (exterior right). The α-region displacement is uniformly forward by 1, the β-region displacement uniformly backward by 2, and the two exterior positions are fixed — confirming per-region displacement uniformity for this example.

**Run decomposition via R-BLK.** *Phase 1 (Split):* c₀ = [1,2] is interior to b₁ = ([1,1], A, 3) at offset ord(c₀) − ord(b₁.v) = 2 − 1 = 1; split b₁ into ([1,1], A, 1) and ([1,2], B, 2). c₁ = [1,4] coincides with b₂'s V-start ([1,4]), so no split is performed at c₁ (boundary case). c₂ = [1,5] is interior to b₂ = ([1,4], D, 2) at offset ord(c₂) − ord(b₂.v) = 5 − 4 = 1; split b₂ into ([1,4], D, 1) and ([1,5], E, 1), separating the β-content D from the exterior-right content E. Post-split partition: {([1,1], A, 1), ([1,2], B, 2), ([1,4], D, 1), ([1,5], E, 1)}.

*Phase 2 (Classify):* Each post-split run lies entirely within one region. ([1,1], A, 1) has V-extent {[1,1]} with ord = 1 < ord(c₀) = 2, so it lies in the *exterior left* region. ([1,2], B, 2) has V-extent {[1,2], [1,3]} with ordinals in [ord(c₀), ord(c₁)) = [2, 4), so it lies in *α*. ([1,4], D, 1) has V-extent {[1,4]} with ordinal in [ord(c₁), ord(c₂)) = [4, 5), so it lies in *β*. ([1,5], E, 1) has V-extent {[1,5]} with ord = 5 ≥ ord(c₂) = 5, so it lies in the *exterior right* region. No run is classified into the non-S region because every V-position in this example has subspace 1 = S; the non-S region is empty here.

*Phase 3 (Reassemble):* Apply each run's region displacement to its V-start. The per-region displacements (3-cut pivot) are: exterior-left fixed, α forward by w_β = 1, β backward by w_α = 2, exterior-right fixed.

- ([1,1], A, 1) → ([1,1], A, 1) (exterior left, fixed, V-start unchanged).
- ([1,2], B, 2) → ([1,3], B, 2) (α, forward 1; V-start shifted from [1,2] to [1,3], width and I-start preserved).
- ([1,4], D, 1) → ([1,2], D, 1) (β, backward 2; V-start shifted from [1,4] to [1,2], width and I-start preserved).
- ([1,5], E, 1) → ([1,5], E, 1) (exterior right, fixed, V-start unchanged).

Sorted by V-start: {([1,1], A, 1), ([1,2], D, 1), ([1,3], B, 2), ([1,5], E, 1)}. *S8-cons verification on reassembled runs:* ([1,3], B, 2): M'(d)([1,3]) = B, M'(d)([1,4]) = C = B + 1 ✓. The width-1 runs ([1,1], A, 1), ([1,2], D, 1), ([1,5], E, 1) satisfy S8-cons trivially at their lone offset k = 0.

*Merge check:* No V-adjacent, I-adjacent pair. ([1,1], A, 1) and ([1,2], D, 1) are V-adjacent (1 + 1 = 2) but not I-adjacent (origin(A) = 3.0.1.0.1 ≠ origin(D) = 5.0.2.0.1, so A + 1 ≠ D). ([1,2], D, 1) and ([1,3], B, 2) are V-adjacent (2 + 1 = 3) but not I-adjacent (origin(D) ≠ origin(B), so D + 1 ≠ B). ([1,3], B, 2) and ([1,5], E, 1) are V-adjacent (3 + 2 = 5) but not I-adjacent (B + 2 = 3.0.1.0.1.0.1.4 ≠ E = 5.0.2.0.1.0.1.2, different origins).

**Canonical partition** (canonical by R-CANON, since the merge check above found no mergeable pair)**:** {([1,1], A, 1), ([1,2], D, 1), ([1,3], B, 2), ([1,5], E, 1)}. The rearrangement preserved one run's interior structure ([1,3], B, 2) — B and C remained adjacent — while isolating A from B/C and pulling D into the position between A and B. No new merges arose because the pre-state I-address origins differ across the boundaries created by the pivot.


## Worked Example: 4-Cut Swap on an 8-Position Document

We trace a 4-cut swap with unequal region widths. Let document d have subspace S = 1 with V_S(d) = {[1,1], ..., [1,8]}, and let the arrangement be:

```
M(d)([1,1]) = 3.0.1.0.1.0.1.1    (I-address A)
M(d)([1,2]) = 3.0.1.0.1.0.1.2    (I-address B)
M(d)([1,3]) = 3.0.1.0.1.0.1.3    (I-address C)
M(d)([1,4]) = 7.0.1.0.1.0.1.1    (I-address D)
M(d)([1,5]) = 5.0.2.0.1.0.1.1    (I-address E)
M(d)([1,6]) = 5.0.2.0.1.0.1.2    (I-address F)
M(d)([1,7]) = 5.0.2.0.1.0.1.3    (I-address G)
M(d)([1,8]) = 3.0.1.0.1.0.1.4    (I-address H)
```

Content A–C originates from document 3.0.1.0.1; D from document 7.0.1.0.1; E–G from document 5.0.2.0.1; H from document 3.0.1.0.1. The canonical run partition has four runs: b₁ = ([1,1], A, 3), b₂ = ([1,4], D, 1), b₃ = ([1,5], E, 3), b₄ = ([1,8], H, 1).

We apply a 4-cut swap with K = ([1,2], [1,4], [1,5], [1,8]): c₀ = [1,2], c₁ = [1,4], c₂ = [1,5], c₃ = [1,8]. The affected range is [c₀, c₃) = {[1,2], ..., [1,7]}. Region α = {[1,2], [1,3]} (w_α = 2), middle μ = {[1,4]} (w_μ = 1), region β = {[1,5], [1,6], [1,7]} (w_β = 3). Since w_α = 2 ≠ w_β = 3, the middle displacement w_β − w_α = 1 is nonzero.

**R-PRE verification.** As in the first example (only the values differ). ✓

**Applying the postconditions.** We compute M'(d) position by position:

R-EXT: M'(d)([1,1]) = M(d)([1,1]) = A. M'(d)([1,8]) = M(d)([1,8]) = H.

R-S1 (j = 0): M'(d)(c₀ + 0) = M'(d)([1,2]) = M(d)(c₂ + 0) = M(d)([1,5]) = E.

R-S1 (j = 1): M'(d)(c₀ + 1) = M'(d)([1,3]) = M(d)(c₂ + 1) = M(d)([1,6]) = F.

R-S1 (j = 2): M'(d)(c₀ + 2) = M'(d)([1,4]) = M(d)(c₂ + 2) = M(d)([1,7]) = G.

R-S2 (j = 0): M'(d)(c₀ + 3 + 0) = M'(d)([1,5]) = M(d)(c₁ + 0) = M(d)([1,4]) = D.

R-S3 (j = 0): M'(d)(c₀ + 3 + 1 + 0) = M'(d)([1,6]) = M(d)(c₀ + 0) = M(d)([1,2]) = B.

R-S3 (j = 1): M'(d)(c₀ + 3 + 1 + 1) = M'(d)([1,7]) = M(d)(c₀ + 1) = M(d)([1,3]) = C.

**Result:**

```
M'(d)([1,1]) = A     (exterior, unchanged)
M'(d)([1,2]) = E     (from β via R-S1)
M'(d)([1,3]) = F     (from β via R-S1)
M'(d)([1,4]) = G     (from β via R-S1)
M'(d)([1,5]) = D     (from μ via R-S2)
M'(d)([1,6]) = B     (from α via R-S3)
M'(d)([1,7]) = C     (from α via R-S3)
M'(d)([1,8]) = H     (exterior, unchanged)
```

The three swap clauses tile [c₀, c₃) = [[1,2], [1,8]) exactly: R-S1 covers ordinals 2–4 (w_β = 3 positions), R-S2 covers ordinal 5 (w_μ = 1 position), R-S3 covers ordinals 6–7 (w_α = 2 positions). Total: 3 + 1 + 2 = 6 = |[c₀, c₃)|. ✓

**R-SPERM verification.** The permutation π:

- π([1,1]) = [1,1] (exterior).
- π([1,2]) = c₀ + w_β + w_μ + 0 = [1,6] (α: j = 0). Check: M'(d)([1,6]) = B = M(d)([1,2]) ✓.
- π([1,3]) = c₀ + w_β + w_μ + 1 = [1,7] (α: j = 1). Check: M'(d)([1,7]) = C = M(d)([1,3]) ✓.
- π([1,4]) = c₀ + w_β + 0 = [1,5] (μ: j = 0). Check: M'(d)([1,5]) = D = M(d)([1,4]) ✓.
- π([1,5]) = c₀ + 0 = [1,2] (β: j = 0). Check: M'(d)([1,2]) = E = M(d)([1,5]) ✓.
- π([1,6]) = c₀ + 1 = [1,3] (β: j = 1). Check: M'(d)([1,3]) = F = M(d)([1,6]) ✓.
- π([1,7]) = c₀ + 2 = [1,4] (β: j = 2). Check: M'(d)([1,4]) = G = M(d)([1,7]) ✓.
- π([1,8]) = [1,8] (exterior).

**R-RI verification.** As in the first example (only the values differ). ✓

**Displacement verification.** Reading each position's displacement as (direction, ordinal distance) from ord(π(v)) − ord(v): [1,2] forward 6 − 2 = 4 = w_β + w_μ ✓; [1,3] forward 7 − 3 = 4 ✓; [1,4] forward 5 − 4 = 1 = w_β − w_α, the μ sub-case with w_β > w_α ✓; [1,5] backward 5 − 2 = 3 = w_α + w_μ ✓; [1,6] backward 6 − 3 = 3 ✓; [1,7] backward 7 − 4 = 3 ✓. The middle-region displacement is forward by 1, confirming the asymmetric structure when w_α ≠ w_β.

**Run decomposition via R-BLK.** *Phase 1 (Split):* c₀ = [1,2] is interior to b₁ = ([1,1], A, 3) at offset 1. Split: ([1,1], A, 1) and ([1,2], B, 2). The remaining cuts c₁ = [1,4], c₂ = [1,5], c₃ = [1,8] coincide with run boundaries (c₁ = b₂'s start, c₂ = b₃'s start, c₃ = b₄'s start), so no further splits. Post-split partition: {([1,1], A, 1), ([1,2], B, 2), ([1,4], D, 1), ([1,5], E, 3), ([1,8], H, 1)}.

*Phase 2 (Classify):* ([1,1], A, 1) → exterior left. ([1,2], B, 2) → α. ([1,4], D, 1) → μ. ([1,5], E, 3) → β. ([1,8], H, 1) → exterior right.

*Phase 3 (Reassemble):* Apply region displacements:

- ([1,1], A, 1) → ([1,1], A, 1) (exterior, fixed)
- ([1,2], B, 2) → ([1,6], B, 2) (α, forward 4)
- ([1,4], D, 1) → ([1,5], D, 1) (μ, forward 1)
- ([1,5], E, 3) → ([1,2], E, 3) (β, backward 3)
- ([1,8], H, 1) → ([1,8], H, 1) (exterior, fixed)

Sorted by V-start: {([1,1], A, 1), ([1,2], E, 3), ([1,5], D, 1), ([1,6], B, 2), ([1,8], H, 1)}. Checking S8-cons: for run ([1,2], E, 3), M'(d)([1,2]) = E, M'(d)([1,3]) = F = E + 1, M'(d)([1,4]) = G = E + 2 ✓.

*Merge check:* ([1,6], B, 2) and ([1,8], H, 1) are V-adjacent (6 + 2 = 8) and I-adjacent (B + 2 = 3.0.1.0.1.0.1.4 = H). Merge: ([1,6], B, 3). No other pair satisfies both conditions — ([1,1], A, 1) and ([1,2], E, 3) differ in origin; ([1,2], E, 3) and ([1,5], D, 1) differ in origin; ([1,5], D, 1) and ([1,6], B, 2) differ in origin.

**Canonical partition** (canonical by R-CANON: after the merge of B and H, the merge check leaves no mergeable pair)**:** {([1,1], A, 1), ([1,2], E, 3), ([1,5], D, 1), ([1,6], B, 3)}. The rearrangement brought B, C (formerly at [1,2]–[1,3]) adjacent to H (at [1,8]), and since B + 2 = H, they merge into a single run of width 3. Meanwhile A, formerly part of a width-3 run with B and C, is now isolated.


## Worked Example: 4-Cut Swap with Equal Region Widths (w_α = w_β)

The two preceding examples leave the μ-displacement sub-case w_α = w_β untraced. We trace a 4-cut swap with w_α = w_β to verify the fixed-μ branch — μ is fixed pointwise by π, even though the surrounding α and β regions exchange places. Let document d have subspace S = 1 with V_S(d) = {[1,1], ..., [1,7]}, and let the arrangement be:

```
M(d)([1,1]) = 3.0.1.0.1.0.1.1    (I-address A)
M(d)([1,2]) = 3.0.1.0.1.0.1.2    (I-address B)
M(d)([1,3]) = 3.0.1.0.1.0.1.3    (I-address C)
M(d)([1,4]) = 7.0.1.0.1.0.1.1    (I-address D)
M(d)([1,5]) = 5.0.2.0.1.0.1.1    (I-address E)
M(d)([1,6]) = 5.0.2.0.1.0.1.2    (I-address F)
M(d)([1,7]) = 9.0.1.0.1.0.1.1    (I-address G)
```

Content A–C originates from document 3.0.1.0.1; D from 7.0.1.0.1; E–F from 5.0.2.0.1; G from 9.0.1.0.1. The canonical run partition has four runs: b₁ = ([1,1], A, 3), b₂ = ([1,4], D, 1), b₃ = ([1,5], E, 2), b₄ = ([1,7], G, 1).

We apply a 4-cut swap with K = ([1,2], [1,4], [1,5], [1,7]): c₀ = [1,2], c₁ = [1,4], c₂ = [1,5], c₃ = [1,7]. The affected range is [c₀, c₃) = {[1,2], ..., [1,6]}. Region α = {[1,2], [1,3]} (w_α = 2), middle μ = {[1,4]} (w_μ = 1), region β = {[1,5], [1,6]} (w_β = 2). Since w_α = w_β = 2, the μ-branch displacement w_β − w_α vanishes and the fixed-μ sub-case applies.

**R-PRE verification.** As in the first example (only the values differ). ✓

**Applying the postconditions.** We compute M'(d) position by position:

R-EXT: M'(d)([1,1]) = M(d)([1,1]) = A. M'(d)([1,7]) = M(d)([1,7]) = G.

R-S1 (j = 0): M'(d)(c₀ + 0) = M'(d)([1,2]) = M(d)(c₂ + 0) = M(d)([1,5]) = E.

R-S1 (j = 1): M'(d)(c₀ + 1) = M'(d)([1,3]) = M(d)(c₂ + 1) = M(d)([1,6]) = F.

R-S2 (j = 0): M'(d)(c₀ + w_β + 0) = M'(d)([1,4]) = M(d)(c₁ + 0) = M(d)([1,4]) = D.

R-S3 (j = 0): M'(d)(c₀ + w_β + w_μ + 0) = M'(d)([1,5]) = M(d)(c₀ + 0) = M(d)([1,2]) = B.

R-S3 (j = 1): M'(d)(c₀ + w_β + w_μ + 1) = M'(d)([1,6]) = M(d)(c₀ + 1) = M(d)([1,3]) = C.

**Result:**

```
M'(d)([1,1]) = A     (exterior, unchanged)
M'(d)([1,2]) = E     (from β via R-S1)
M'(d)([1,3]) = F     (from β via R-S1)
M'(d)([1,4]) = D     (from μ via R-S2 — *fixed in place*, μ displacement zero when w_β = w_α)
M'(d)([1,5]) = B     (from α via R-S3)
M'(d)([1,6]) = C     (from α via R-S3)
M'(d)([1,7]) = G     (exterior, unchanged)
```

The R-S2 clause exhibits the structural property of the w_α = w_β branch: M'(d)([1,4]) = M(d)([1,4]) = D, because the destination ord c₀ + w_β = 4 coincides with the source ord c₁ = 4 when w_β = w_α. R-S2 is *not* vacuous — it still asserts the equation M'(d)([1,4]) = M(d)([1,4]) — but it discharges to a fixed-point identity at every offset j < w_μ. The three swap clauses tile [c₀, c₃) = [[1,2], [1,7)) exactly: R-S1 covers ordinals 2–3 (w_β = 2 positions), R-S2 covers ordinal 4 (w_μ = 1 position), R-S3 covers ordinals 5–6 (w_α = 2 positions). Total: 2 + 1 + 2 = 5 = |[c₀, c₃)|. ✓

**R-SPERM verification.** The permutation π:

- π([1,1]) = [1,1] (exterior).
- π([1,2]) = c₀ + w_β + w_μ + 0 = [1,5] (α: j = 0). Check: M'(d)([1,5]) = B = M(d)([1,2]) ✓.
- π([1,3]) = c₀ + w_β + w_μ + 1 = [1,6] (α: j = 1). Check: M'(d)([1,6]) = C = M(d)([1,3]) ✓.
- π([1,4]) = c₀ + w_β + 0 = [1,4] (μ: j = 0). Check: M'(d)([1,4]) = D = M(d)([1,4]) ✓.
- π([1,5]) = c₀ + 0 = [1,2] (β: j = 0). Check: M'(d)([1,2]) = E = M(d)([1,5]) ✓.
- π([1,6]) = c₀ + 1 = [1,3] (β: j = 1). Check: M'(d)([1,3]) = F = M(d)([1,6]) ✓.
- π([1,7]) = [1,7] (exterior).

Note π([1,4]) = [1,4]: μ is the single position fixed by π via the μ-branch (as distinct from the exterior, which is fixed via R-FRAME-S(a)). The rearrangement is still a genuine swap — α and β positions move — but the middle region holds in place pointwise.

**R-RI verification.** As in the first example (only the values differ). ✓

**Displacement verification.** Reading each position's displacement as (direction, ordinal distance) from ord(π(v)) − ord(v): [1,2] forward 5 − 2 = 3 = w_β + w_μ ✓; [1,3] forward 6 − 3 = 3 ✓; [1,4] fixed — the μ sub-case with w_β = w_α ✓; [1,5] backward 5 − 2 = 3 = w_α + w_μ ✓; [1,6] backward 6 − 3 = 3 ✓. The middle-region displacement vanishes, confirming the structural symmetry of the w_β = w_α sub-case.

**Run decomposition via R-BLK.** *Phase 1 (Split):* c₀ = [1,2] is interior to b₁ = ([1,1], A, 3) at offset 1. Split: ([1,1], A, 1) and ([1,2], B, 2). The remaining cuts c₁ = [1,4], c₂ = [1,5], c₃ = [1,7] coincide with run boundaries (c₁ = b₂'s start, c₂ = b₃'s start, c₃ = b₄'s start), so no further splits. Post-split partition: {([1,1], A, 1), ([1,2], B, 2), ([1,4], D, 1), ([1,5], E, 2), ([1,7], G, 1)}.

*Phase 2 (Classify):* ([1,1], A, 1) → exterior left. ([1,2], B, 2) → α. ([1,4], D, 1) → μ. ([1,5], E, 2) → β. ([1,7], G, 1) → exterior right.

*Phase 3 (Reassemble):* Apply region displacements (α: forward 3, μ: fixed, β: backward 3, exteriors: fixed):

- ([1,1], A, 1) → ([1,1], A, 1) (exterior)
- ([1,2], B, 2) → ([1,5], B, 2) (α, V-start shifted forward 3)
- ([1,4], D, 1) → ([1,4], D, 1) (μ, V-start unchanged — fixed)
- ([1,5], E, 2) → ([1,2], E, 2) (β, V-start shifted backward 3)
- ([1,7], G, 1) → ([1,7], G, 1) (exterior)

Sorted by V-start: {([1,1], A, 1), ([1,2], E, 2), ([1,4], D, 1), ([1,5], B, 2), ([1,7], G, 1)}. The μ-run ([1,4], D, 1) carries through Phase 3 untouched because its assigned displacement is zero; the α- and β-runs exchange positions across this fixed centre.

*S8-cons verification on reassembled runs:* ([1,2], E, 2): M'(d)([1,2]) = E, M'(d)([1,3]) = F = E + 1 ✓. ([1,5], B, 2): M'(d)([1,5]) = B, M'(d)([1,6]) = C = B + 1 ✓. The width-1 runs ([1,1], A, 1), ([1,4], D, 1), ([1,7], G, 1) satisfy S8-cons trivially at the lone offset k = 0.

*Merge check:* No V-adjacent, I-adjacent pair: ([1,1], A, 1) and ([1,2], E, 2) differ in origin (3.0.1.0.1 vs 5.0.2.0.1); ([1,2], E, 2) and ([1,4], D, 1) differ in origin (5.0.2.0.1 vs 7.0.1.0.1); ([1,4], D, 1) and ([1,5], B, 2) differ in origin (7.0.1.0.1 vs 3.0.1.0.1); ([1,5], B, 2) and ([1,7], G, 1) differ in origin (3.0.1.0.1 vs 9.0.1.0.1).

**Canonical partition** (canonical by R-CANON, since the merge check above found no mergeable pair)**:** {([1,1], A, 1), ([1,2], E, 2), ([1,4], D, 1), ([1,5], B, 2), ([1,7], G, 1)}. The rearrangement exchanges the α- and β-runs across a fixed μ-run; the canonical partition is reached without further merges because each region's I-address origin differs from its neighbours'. The example confirms the fixed-μ sub-case: the μ-run is structurally invariant under R-BLK's Phase 3 reassembly when w_α = w_β.


## Worked Example: 4-Cut Swap with w_β < w_α (Backward μ Sub-Case)

We exhibit a 4-cut swap with w_β < w_α (w_α = 3, w_β = 1, w_μ = 2), the μ-branch sub-case in which the μ-region shifts *backward* by w_α − w_β. The asymmetry w_α > w_β reverses the direction of the μ-displacement: the middle region moves earlier in the V-position ordering, into the slot vacated by the (now-narrower) β-region, while α stretches across the right end of the affected range.

Let document d have subspace S = 1 with V_S(d) = {[1,1], ..., [1,8]}, and let the arrangement be:

```
M(d)([1,1]) = 3.0.1.0.1.0.1.1    (I-address A)
M(d)([1,2]) = 3.0.1.0.1.0.1.2    (I-address B)
M(d)([1,3]) = 3.0.1.0.1.0.1.3    (I-address C)
M(d)([1,4]) = 3.0.1.0.1.0.1.4    (I-address D)
M(d)([1,5]) = 5.0.2.0.1.0.1.1    (I-address E)
M(d)([1,6]) = 5.0.2.0.1.0.1.2    (I-address F)
M(d)([1,7]) = 7.0.1.0.1.0.1.1    (I-address G)
M(d)([1,8]) = 9.0.1.0.1.0.1.1    (I-address H)
```

Content A–D originates from document 3.0.1.0.1; E–F from 5.0.2.0.1; G from 7.0.1.0.1; H from 9.0.1.0.1. The canonical run partition has four runs: b₁ = ([1,1], A, 4), b₂ = ([1,5], E, 2), b₃ = ([1,7], G, 1), b₄ = ([1,8], H, 1).

We apply a 4-cut swap with K = ([1,2], [1,5], [1,7], [1,8]): c₀ = [1,2], c₁ = [1,5], c₂ = [1,7], c₃ = [1,8]. The affected range is [c₀, c₃) = {[1,2], ..., [1,7]}. Region α = {[1,2], [1,3], [1,4]} (w_α = 3), middle μ = {[1,5], [1,6]} (w_μ = 2), region β = {[1,7]} (w_β = 1). Since w_β = 1 < w_α = 3, the μ-region shifts backward by w_α − w_β = 2, the backward sub-case.

**R-PRE verification.** As in the first example (only the values differ). ✓

**Applying the postconditions.** We compute M'(d) position by position:

R-EXT: M'(d)([1,1]) = M(d)([1,1]) = A. M'(d)([1,8]) = M(d)([1,8]) = H.

R-S1 (j = 0): M'(d)(c₀ + 0) = M'(d)([1,2]) = M(d)(c₂ + 0) = M(d)([1,7]) = G.

R-S2 (j = 0): M'(d)(c₀ + w_β + 0) = M'(d)([1,3]) = M(d)(c₁ + 0) = M(d)([1,5]) = E.

R-S2 (j = 1): M'(d)(c₀ + w_β + 1) = M'(d)([1,4]) = M(d)(c₁ + 1) = M(d)([1,6]) = F.

R-S3 (j = 0): M'(d)(c₀ + w_β + w_μ + 0) = M'(d)([1,5]) = M(d)(c₀ + 0) = M(d)([1,2]) = B.

R-S3 (j = 1): M'(d)(c₀ + w_β + w_μ + 1) = M'(d)([1,6]) = M(d)(c₀ + 1) = M(d)([1,3]) = C.

R-S3 (j = 2): M'(d)(c₀ + w_β + w_μ + 2) = M'(d)([1,7]) = M(d)(c₀ + 2) = M(d)([1,4]) = D.

**Result:**

```
M'(d)([1,1]) = A     (exterior, unchanged)
M'(d)([1,2]) = G     (from β via R-S1)
M'(d)([1,3]) = E     (from μ via R-S2)
M'(d)([1,4]) = F     (from μ via R-S2)
M'(d)([1,5]) = B     (from α via R-S3)
M'(d)([1,6]) = C     (from α via R-S3)
M'(d)([1,7]) = D     (from α via R-S3)
M'(d)([1,8]) = H     (exterior, unchanged)
```

The three swap clauses tile [c₀, c₃) = [[1,2], [1,8)) exactly: R-S1 covers ordinal 2 (w_β = 1 position), R-S2 covers ordinals 3–4 (w_μ = 2 positions), R-S3 covers ordinals 5–7 (w_α = 3 positions). Total: 1 + 2 + 3 = 6 = |[c₀, c₃)|. ✓

**R-SPERM verification.** The permutation π:

- π([1,1]) = [1,1] (exterior).
- π([1,2]) = c₀ + w_β + w_μ + 0 = [1,5] (α: j = 0). Check: M'(d)([1,5]) = B = M(d)([1,2]) ✓.
- π([1,3]) = c₀ + w_β + w_μ + 1 = [1,6] (α: j = 1). Check: M'(d)([1,6]) = C = M(d)([1,3]) ✓.
- π([1,4]) = c₀ + w_β + w_μ + 2 = [1,7] (α: j = 2). Check: M'(d)([1,7]) = D = M(d)([1,4]) ✓.
- π([1,5]) = c₀ + w_β + 0 = [1,3] (μ: j = 0). Check: M'(d)([1,3]) = E = M(d)([1,5]) ✓.
- π([1,6]) = c₀ + w_β + 1 = [1,4] (μ: j = 1). Check: M'(d)([1,4]) = F = M(d)([1,6]) ✓.
- π([1,7]) = c₀ + 0 = [1,2] (β: j = 0). Check: M'(d)([1,2]) = G = M(d)([1,7]) ✓.
- π([1,8]) = [1,8] (exterior).

The μ-region positions [1,5] and [1,6] map *backward* to [1,3] and [1,4] respectively — a uniform backward shift of w_α − w_β = 3 − 1 = 2, the μ sub-case with w_β < w_α recorded in the Displacement Analysis remark.

**R-RI verification.** As in the first example (only the values differ). ✓

**Displacement verification.** Reading each position's displacement as (direction, ordinal distance) from ord(π(v)) − ord(v): [1,2] forward 5 − 2 = 3 = w_β + w_μ ✓; [1,3] forward 6 − 3 = 3 ✓; [1,4] forward 7 − 4 = 3 ✓; [1,5] backward 5 − 3 = 2 = w_α − w_β, the μ sub-case with w_β < w_α ✓; [1,6] backward 6 − 4 = 2 ✓; [1,7] backward 7 − 2 = 5 = w_α + w_μ ✓. The middle-region displacement is uniformly backward by 2, confirming the backward μ sub-case at every offset.

**Run decomposition via R-BLK.** *Phase 1 (Split):* c₀ = [1,2] is interior to b₁ = ([1,1], A, 4) at offset 1. Split: ([1,1], A, 1) and ([1,2], B, 3). The remaining cuts c₁ = [1,5], c₂ = [1,7], c₃ = [1,8] coincide with run boundaries (c₁ = b₂'s start, c₂ = b₃'s start, c₃ = b₄'s start), so no further splits. Post-split partition: {([1,1], A, 1), ([1,2], B, 3), ([1,5], E, 2), ([1,7], G, 1), ([1,8], H, 1)}.

*Phase 2 (Classify):* ([1,1], A, 1) → exterior left. ([1,2], B, 3) → α (ordinals 2, 3, 4 ∈ [ord(c₀), ord(c₁)) = [2, 5)). ([1,5], E, 2) → μ (ordinals 5, 6 ∈ [ord(c₁), ord(c₂)) = [5, 7)). ([1,7], G, 1) → β (ordinal 7 ∈ [ord(c₂), ord(c₃)) = [7, 8)). ([1,8], H, 1) → exterior right.

*Phase 3 (Reassemble):* Apply region displacements (α: forward 3, μ: backward 2, β: backward 5, exteriors: fixed):

- ([1,1], A, 1) → ([1,1], A, 1) (exterior)
- ([1,2], B, 3) → ([1,5], B, 3) (α, V-start shifted forward 3)
- ([1,5], E, 2) → ([1,3], E, 2) (μ, V-start shifted backward 2 — the backward μ sub-case in action)
- ([1,7], G, 1) → ([1,2], G, 1) (β, V-start shifted backward 5)
- ([1,8], H, 1) → ([1,8], H, 1) (exterior)

Sorted by V-start: {([1,1], A, 1), ([1,2], G, 1), ([1,3], E, 2), ([1,5], B, 3), ([1,8], H, 1)}.

*S8-cons verification on reassembled runs:* ([1,3], E, 2): M'(d)([1,3]) = E, M'(d)([1,4]) = F = E + 1 ✓. ([1,5], B, 3): M'(d)([1,5]) = B, M'(d)([1,6]) = C = B + 1, M'(d)([1,7]) = D = B + 2 ✓. The width-1 runs satisfy S8-cons trivially.

*Merge check:* ([1,1], A, 1) and ([1,2], G, 1) are V-adjacent (1 + 1 = 2) but not I-adjacent (origin(A) = 3.0.1.0.1 ≠ origin(G) = 7.0.1.0.1, so A + 1 ≠ G). ([1,2], G, 1) and ([1,3], E, 2) are V-adjacent (2 + 1 = 3) but differ in origin (G vs E). ([1,3], E, 2) and ([1,5], B, 3) are V-adjacent (3 + 2 = 5) but differ in origin (E vs B). ([1,5], B, 3) and ([1,8], H, 1) are V-adjacent (5 + 3 = 8); checking I-adjacency: B + 3 = 3.0.1.0.1.0.1.5 ≠ H = 9.0.1.0.1.0.1.1. No mergeable pair.

**Canonical partition** (canonical by R-CANON, since the merge check above found no mergeable pair)**:** {([1,1], A, 1), ([1,2], G, 1), ([1,3], E, 2), ([1,5], B, 3), ([1,8], H, 1)}. The rearrangement extracts G into the slot vacated by α's leftward content (originally B at [1,2]), places the μ-content (E, F) one step earlier in the V-ordering (backward by 2), and pushes B, C, D to the right end of the affected range. The example confirms the backward μ sub-case: the μ-region moves earlier when the narrower β cannot accommodate α's full width, with the displacement magnitude w_α − w_β exposed cleanly in both the explicit π formula and the post-reassembly V-start of the μ-run.


## Worked Example: 3-Cut Pivot at the Boundary (Minimum V_S(d), Empty Right Exterior)

We exhibit the *boundary* configuration: a 3-cut pivot on the minimum-size V_S(d) admitting a 3-cut sequence, with the rightmost cut placed strictly above max(V_S(d)) so the right exterior is empty. This case exercises three structural edges simultaneously — the minimum w_α = w_β = 1 (Phase 2 classifies a single position into each region), V_S(d) size 2 (the smallest V_S(d) for which a 3-cut sequence with non-degenerate regions exists), and the "Outside ⋃_k V(b_k)" sub-case of Phase 1 (where the last cut falls outside dom(M(d)) and triggers the empty-right-exterior trace recorded in R-BLK).

Let document d have subspace S = 1 with V_S(d) = {[1,1], [1,2]} (so N = max{ord(v) : v ∈ V_S(d)} = 2), and let the arrangement be:

```
M(d)([1,1]) = 3.0.1.0.1.0.1.1    (I-address A)
M(d)([1,2]) = 5.0.2.0.1.0.1.1    (I-address B)
```

Content A originates from document 3.0.1.0.1; B from document 5.0.2.0.1. The canonical run partition has two width-1 runs: b₁ = ([1,1], A, 1) and b₂ = ([1,2], B, 1) — distinct origins prevent any merge in the pre-state.

We apply a 3-cut pivot with K = ([1,1], [1,2], [1,3]): c₀ = [1,1], c₁ = [1,2], c₂ = [1,3] = [S, N + 1]. The affected range [c₀, c₂) at depth 2 in subspace 1 covers ordinals {1, 2}; under D-SEQ (V_S(d) = {[1,1], [1,2]}), this is exactly V_S(d). Region α = {[1,1]} (w_α = 1), region β = {[1,2]} (w_β = 1). Both exteriors are *empty*: the left exterior {v ∈ V_S(d) : v < c₀} is empty because ord(c₀) = 1 = min{ord(v) : v ∈ V_S(d)}; the right exterior {v ∈ V_S(d) : v ≥ c₂} is empty because ord(c₂) = 3 > N = 2.

**R-PRE verification.** As in the first example (only the values differ), with one boundary-specific point: R-PRE(iv)'s bound v < c₂ is exclusive, so c₂ = [1,3] need not lie in V_S(d) — and here it does not (ord(c₂) = 3 > N = 2). ✓

**Applying the postconditions.** Both exteriors are empty, so R-EXT contributes no equations; the entire V_S(d) is covered by R-P1 and R-P2.

R-P1 (j = 0): M'(d)(c₀ + 0) = M'(d)([1,1]) = M(d)(c₁ + 0) = M(d)([1,2]) = B.

R-P2 (j = 0): M'(d)(c₀ + 1 + 0) = M'(d)([1,2]) = M(d)(c₀ + 0) = M(d)([1,1]) = A.

**Result:**

```
M'(d)([1,1]) = B     (was β at c₁, pivoted to c₀)
M'(d)([1,2]) = A     (was α at c₀, pivoted past β to c₀ + w_β)
```

The pivot reduces to a transposition of the two V-positions. No position is fixed by π (a *pure* swap with no exterior anchor).

**R-PPERM verification.** The permutation π: π([1,1]) = c₀ + w_β + 0 = [1,2] (α: j = 0). π([1,2]) = c₀ + 0 = [1,1] (β: j = 0). Bijectivity: π is an involution ((π ∘ π)([1,1]) = π([1,2]) = [1,1], symmetrically for [1,2]). Check: M'(d)(π([1,1])) = M'(d)([1,2]) = A = M(d)([1,1]) ✓. M'(d)(π([1,2])) = M'(d)([1,1]) = B = M(d)([1,2]) ✓.

**R-RI verification.** As in the first example (only the values differ). ✓

**Displacement verification.** Reading each position's displacement as (direction, ordinal distance): [1,1] forward 2 − 1 = 1 = w_β (α, j = 0); [1,2] backward 2 − 1 = 1 = w_α (β, j = 0). The displacement is uniformly forward by 1 on α and uniformly backward by 1 on β, confirming per-region displacement uniformity at minimum width. No position is fixed — both exterior regions are empty, so the fixed (distance-0) case arises nowhere on V_S(d).

**Run decomposition via R-BLK.** *Phase 1 (Split):* c₀ = [1,1] coincides with b₁'s V-start ([1,1]) — boundary case, no split. c₁ = [1,2] coincides with b₂'s V-start ([1,2]) — boundary case, no split. c₂ = [1,3] = [S, N + 1] falls *outside* ⋃_k V(b_k): every run b_k in the partition of V_S(d) has V-extent V(b_k) ⊆ V_S(d) = {[1,1], [1,2]}, so max{ord(v) : v ∈ V(b_k)} ≤ 2 < 3 = ord(c₂), hence c₂ ∉ V(b_k) for every k. This is the *empty right exterior* sub-case described in Phase 1 ("Outside ⋃_k V(b_k)"); no split occurs at c₂, no run is bisected by c₂, and the right-exterior region {v ∈ V_S(d) : v ≥ c₂} contains zero V-positions. Post-split partition: {([1,1], A, 1), ([1,2], B, 1)} — identical to the pre-state partition, since every cut fell at a run boundary or outside V_S(d).

*Phase 2 (Classify):* ([1,1], A, 1) has V-extent {[1,1]} with ord = 1 in [ord(c₀), ord(c₁)) = [1, 2), so it lies in *α*. ([1,2], B, 1) has V-extent {[1,2]} with ord = 2 in [ord(c₁), ord(c₂)) = [2, 3), so it lies in *β*. No run is classified into the left exterior (empty) or the right exterior (empty). The non-S region is also empty (every V-position has subspace 1 = S). Every run is classified into α or β.

*Phase 3 (Reassemble):* Apply π to each run's V-start.

- ([1,1], A, 1) → (π([1,1]), A, 1) = ([1,2], A, 1) (α-branch of R-PPERM with j = 0 gives π([1,1]) = c₀ + w_β + 0 = [1,2]).
- ([1,2], B, 1) → (π([1,2]), B, 1) = ([1,1], B, 1) (β-branch of R-PPERM with j = 0 gives π([1,2]) = c₀ + 0 = [1,1]).

Sorted by V-start: {([1,1], B, 1), ([1,2], A, 1)}. *S8-cons verification:* both runs are width 1, so S8-cons holds trivially at the lone offset k = 0 (M'(d)([1,1]) = B = B + 0; M'(d)([1,2]) = A = A + 0).

*Merge check:* ([1,1], B, 1) and ([1,2], A, 1) are V-adjacent (1 + 1 = 2) but not I-adjacent (origin(B) = 5.0.2.0.1 ≠ origin(A) = 3.0.1.0.1, so B + 1 ≠ A). No mergeable pair.

**Canonical partition** (canonical by R-CANON, since the merge check above found no mergeable pair)**:** {([1,1], B, 1), ([1,2], A, 1)}. The rearrangement exchanges the two positions across the cut sequence; both runs remain width-1, no merges arise, and the canonical decomposition of M'(d) coincides with the post-Phase-3 partition. The example confirms three structural edges of R-BLK simultaneously: (a) the minimum w_α = w_β = 1 still admits valid Phase-2 classification (α and β each receive exactly one run); (b) the empty-right-exterior dispatch in Phase 1 fires correctly at c₂ = [S, N + 1] (the "Outside ⋃_k V(b_k)" sub-case), with no run bisected and no right-exterior classification; (c) Phase 3 reassembles via π alone at minimum size, with the I-start and width of each run preserved verbatim and the V-starts transposed by the explicit R-PPERM formulas.


## Worked Example: 3-Cut Pivot with a Non-S (Link-Subspace) Position

We trace a 3-cut pivot on a document whose arrangement also references the *link* subspace (subspace 2), to pin down that a non-S position is fixed by π and its run is carried through untouched while the text subspace is rearranged.

Let document d have text positions V_1(d) = {[1,1], [1,2], [1,3]} and a single link position [2,1] (subspace 2, depth 2), so dom(M(d)) = {[1,1], [1,2], [1,3], [2,1]}. The arrangement is:

```
M(d)([1,1]) = 3.0.1.0.1.0.1.1    (I-address A, text content)
M(d)([1,2]) = 3.0.1.0.1.0.1.2    (I-address B, text content)
M(d)([1,3]) = 3.0.1.0.1.0.1.3    (I-address C, text content)
M(d)([2,1]) = 3.0.1.0.1.0.2.1    (I-address L, link content)
```

The text I-addresses A–C lie in element subspace 1 (the `...0.1.k` tail); the link I-address L lies in element subspace 2 (the `...0.2.1` tail), satisfying zeros(L) = 3 (element-level, S7b of ASN-0036). The canonical run partition has two runs in distinct V-subspaces: the text run b₁ = ([1,1], 3.0.1.0.1.0.1.1, 3) and the link run b₂ = ([2,1], 3.0.1.0.1.0.2.1, 1). No run spans both subspaces — every run lies within a single subspace (SUBCONF).

We apply a 3-cut pivot with K = ([1,1], [1,2], [1,3]): c₀ = [1,1], c₁ = [1,2], c₂ = [1,3]. All cuts lie in subspace 1 (CS3) at depth 2 (CS4). The affected range is [c₀, c₂) = {[1,1], [1,2]}. Region α = {[1,1]} (w_α = 1), region β = {[1,2]} (w_β = 1). The text position [1,3] is the right exterior (ord ≥ ord(c₂) = 3), and the link position [2,1] lies entirely outside subspace S.

**R-PRE verification.** (i) M(d) well-defined. (ii) V_S(d) ≠ ∅. (iii) CS1: n = 3; CS2: [1,1] < [1,2] < [1,3]; CS3: all cuts subspace 1; CS4: all depth 2; CS5: ordinals 1, 2, 3 ≥ 1. (iv) Every subspace-1 depth-2 position in [[1,1], [1,3)) — namely [1,1], [1,2] — lies in V_1(d); the link position [2,1] is *not* quantified over (its subspace is 2 ≠ S). Width positivity: w_α = 1 ≥ 1, w_β = 1 ≥ 1. ✓

**Applying the postconditions.** We compute M'(d) position by position:

R-P1 (j = 0): M'(d)(c₀ + 0) = M'(d)([1,1]) = M(d)(c₁ + 0) = M(d)([1,2]) = B.

R-P2 (j = 0): M'(d)(c₀ + 1 + 0) = M'(d)([1,2]) = M(d)(c₀ + 0) = M(d)([1,1]) = A.

R-EXT: M'(d)([1,3]) = M(d)([1,3]) = C (text right exterior).

R-FRAME-P(a): M'(d)([2,1]) = M(d)([2,1]) = L (subspace 2 ≠ S, so the non-S frame condition applies — the link position is untouched).

**Result:**

```
M'(d)([1,1]) = B     (was β, pivoted to start)
M'(d)([1,2]) = A     (was α, pivoted past β)
M'(d)([1,3]) = C     (text exterior, unchanged)
M'(d)([2,1]) = L     (link subspace, unchanged — non-S pass-through)
```

**R-PPERM verification.** The permutation π: π([1,1]) = c₀ + w_β + 0 = [1,2] (α); π([1,2]) = c₀ + 0 = [1,1] (β); π([1,3]) = [1,3] (subspace-S exterior); π([2,1]) = [2,1] (non-S branch — *fixed pointwise by the non-S clause of R-PPERM*). Check: M'(d)(π([1,1])) = M'(d)([1,2]) = A = M(d)([1,1]) ✓; M'(d)(π([2,1])) = M'(d)([2,1]) = L = M(d)([2,1]) ✓ (the non-S defining equation holds with π the identity).

**R-RI verification.** ran(M'(d)) = {B, A, C, L} = ran(M(d)); since ran(M(d)) ⊆ dom(C) and C' = C, ran(M'(d)) ⊆ dom(C'). ✓

**Run decomposition via R-BLK.** *Phase 1 (Split):* the cuts c₀ = [1,1], c₁ = [1,2], c₂ = [1,3] all lie in subspace 1. c₀ coincides with b₁'s V-start (boundary, no split). c₁ = [1,2] is interior to b₁ = ([1,1], A, 3) at offset 1; split into ([1,1], A, 1) and ([1,2], B, 2). c₂ = [1,3] is interior to ([1,2], B, 2) at offset 1; split into ([1,2], B, 1) and ([1,3], C, 1). The link run b₂ = ([2,1], L, 1) is never touched: by CS3 every cut is in subspace 1, so no cut falls in V(b₂) ⊆ subspace 2. Post-split partition: {([1,1], A, 1), ([1,2], B, 1), ([1,3], C, 1), ([2,1], L, 1)}.

*Phase 2 (Classify):* ([1,1], A, 1) → α (ord 1 ∈ [1, 2)). ([1,2], B, 1) → β (ord 2 ∈ [2, 3)). ([1,3], C, 1) → exterior right (ord 3 ≥ 3). ([2,1], L, 1) → *non-S region* (subspace 2 ≠ S).

*Phase 3 (Reassemble):* Apply each run's region displacement (α forward by w_β = 1, β backward by w_α = 1, exterior fixed, non-S fixed):

- ([1,1], A, 1) → ([1,2], A, 1) (α, forward 1)
- ([1,2], B, 1) → ([1,1], B, 1) (β, backward 1)
- ([1,3], C, 1) → ([1,3], C, 1) (exterior, fixed)
- ([2,1], L, 1) → ([2,1], L, 1) (non-S, carried verbatim — π is the identity by the non-S clause of R-PPERM)

Sorted by V-start: {([1,1], B, 1), ([1,2], A, 1), ([1,3], C, 1), ([2,1], L, 1)}. All width 1, so S8-cons holds trivially.

*Run-partition (disjointness and coverage).* The three reassembled subspace-1 runs cover V_1(d) = {[1,1], [1,2], [1,3]}; the carried-over link run covers dom(M(d)) \ V_1(d) = {[2,1]}. The two groups are disjoint because subspace-1 and subspace-2 V-extents lie under non-nesting prefixes ([1,…] and [2,…]), which generate disjoint subtrees by T10 (ASN-0034). Their union is dom(M'(d)), so B′ is a run partition of M'(d) — disjoint and covering; maximality is not claimed.

*Merge check:* No mergeable pair. The text runs ([1,1], B, 1), ([1,2], A, 1), ([1,3], C, 1) are pairwise V-adjacent but not I-adjacent (B + 1 = 3.0.1.0.1.0.1.3 = C ≠ A; A + 1 = B ≠ C). The link run ([2,1], L, 1) is not V-adjacent to any text run — its subspace differs — so no cross-subspace merge can arise.

**Canonical partition** (canonical by R-CANON, since the merge check above found no mergeable pair)**:** {([1,1], B, 1), ([1,2], A, 1), ([1,3], C, 1), ([2,1], L, 1)}. The text subspace is rearranged (A and B transpose, C anchored at the exterior), while the link position [2,1] passes through entirely untouched — fixed by π, carried verbatim by R-BLK, and kept disjoint from the rearranged text runs by subspace separation. This example exercises the non-S machinery (R-NS, R-FRAME-P(a), R-BLK's verbatim carry, and the T10 cross-group disjointness) that the text-only examples leave latent.


## Properties Introduced

| Label | Type | Statement | Status |
|-------|------|-----------|--------|
| CutSequence | DEF | Tuple (c₀, ..., c_{n−1}) with n ∈ {3,4}, strictly ordered, same subspace, depth 2, positive ordinal (CS1–CS5) | introduced |
| RegionPartition | DEF | Partition of affected range into regions α, β (3-cut) or α, μ, β (4-cut) by cut positions | introduced |
| R-PRE | DEF | Precondition clauses (i)–(iv): M(d) exists, V_S(d) non-empty, cuts satisfy CS1–CS5, affected range covered | introduced |
| PivotPostcondition | DEF | 3-cut rearrangement: β content placed at c₀, then α content, exterior unchanged (R-EXT, R-P1, R-P2) | introduced |
| SwapPostcondition | DEF | 4-cut rearrangement: β at c₀, then μ, then α, exterior unchanged (R-EXT, R-S1, R-S2, R-S3) | introduced |
| REARRANGE_K | OPERATION | State transition Σ → Σ' parameterized by cut sequence K and document d; precondition R-PRE(K); postcondition PivotPostcondition (n=3) or SwapPostcondition (n=4) plus frame conditions R-FRAME-P or R-FRAME-S | introduced |
| ArrangementRearrangement | DEF | State transition with dom(M'(d)) = dom(M(d)), C' = C, M'(d') = M(d') for d' ≠ d, and bijection π with M'(d)(π(v)) = M(d)(v) | introduced |
| Split | DEF | Correspondence run (v, a, n) at interior offset c yields (v, a, c) and (v + c, a + c, n − c) | introduced |
| Merge | DEF | V-adjacent and I-adjacent correspondence runs (v₁, a₁, n₁), (v₂, a₂, n₂) combine to (v₁, a₁, n₁ + n₂) | introduced |
| CanonicalRunDecomposition | DEF | Names the S8-unique (ASN-0036) maximal-run partition; Split/Merge relate a valid partition to it | introduced |
| R-PIV | LEMMA | Pivot postcondition is a total function on dom(M(d)) | supporting |
| R-SWP | LEMMA | Swap postcondition is a total function on dom(M(d)) | supporting |
| R-PPERM | LEMMA | Bijection π for 3-cut pivot: α shifts forward by w_β, β shifts backward by w_α | introduced |
| R-SPERM | LEMMA | Bijection π for 4-cut swap: α shifts forward by w_β + w_μ, μ shifts by w_β − w_α, β shifts backward by w_α + w_μ | introduced |
| R-FRAME-P | FRAME | Pivot: other subspaces, other documents, and content store are preserved | introduced |
| R-FRAME-S | FRAME | Swap: other subspaces, other documents, and content store are preserved | introduced |
| R-NS | LEMMA | M'(d) = M(d) on non-S positions (NS-M) | introduced |
| R-RI | LEMMA | Rearrangement preserves S3 (referential integrity): ran(M'(d)) = ran(M(d)) ⊆ dom(C) = dom(C') | introduced |
| R-COMM | LEMMA | π(v + k) = π(v) + k when v and v + k lie in the same region: cut-point permutation commutes with ordinal shift | introduced |
| R-BLK | LEMMA | Run partition transforms by split-at-cuts then displace-per-region, yielding a run partition B′ (run-partition disjointness/coverage + S8-cons) under M'(d); maximality not claimed | introduced |
| R-CANON | LEMMA | A covering, disjoint partition into valid runs with no mergeable pair is the S8-unique maximal (canonical) decomposition; hence iterated merging terminates at it and is confluent | introduced |


## Open Questions

Does the 4-cut swap definition generalize to k-cut rearrangements for k > 4, and if so, what is the natural class of permutations that "rearrangement by cut points" can express?

What must a well-formed editing sequence guarantee about the composition of multiple rearrangements — is the composition of two rearrangements always expressible as a single rearrangement, or can sequences of rearrangements produce arrangements unreachable by any single operation?

Under what conditions can a rearrangement cause the number of correspondence runs in the canonical partition to increase, and is there an upper bound on the increase relative to the number of cut points?

What constraints, if any, must cut points satisfy relative to the run boundaries of the canonical partition, or are arbitrary cut positions within the V-span always valid?

What is the weakest precondition for REARRANGE_K to establish the post-state invariant suite Q?
