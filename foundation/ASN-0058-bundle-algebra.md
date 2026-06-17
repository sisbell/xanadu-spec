> **ASN-0058 · Mapping Block Algebra** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md), [ASN-0053 · Span Algebra](ASN-0053-span-algebra.md)  
> [Condensed statements →](ASN-0058-bundle-algebra.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0058: Mapping Block Algebra

*2026-03-20, revised 2026-03-22*

## The Problem

ASN-0036 establishes that a document's arrangement `M(d)` is a partial function from V-positions to I-addresses (S2), and that this function decomposes into correspondence runs (S8). But S8 asserts only that a decomposition *exists*. We now ask: what is the algebra of these runs? How do they compose and decompose? What invariants must any valid representation of the arrangement preserve?

Nelson names the central data structure the *Permutation Of Order Matrix* — the POOM. The Istream records what exists; the Vstream records how it is arranged. The POOM mediates between the two orderings. We seek the abstract properties of this mediation — properties that any implementation must satisfy, regardless of its internal data structures.

## The Mapping Block

The arrangement pairs V-positions with I-addresses. These pairings are not arbitrary — they cluster into contiguous runs where consecutive V-positions map to consecutive I-addresses. Nelson identifies this clustering as the fundamental unit of representation:

> "An I-span (identity span) describes a contiguous set of elements in the document's v-stream which have contiguous identity (I-stream) addresses. A document may be described completely by a sequence of I-spans covering its entire v-stream." [LM 4/36]

We adopt the term *mapping block* to distinguish the abstract object from any particular representation.

**Definition (Mapping Block).** A mapping block `β = (v, a, n)` consists of:

- `v ∈ T` — the V-start (a position in the document's virtual stream)
- `a ∈ T` — the I-start (an address in the permanent content store)
- `n ∈ ℕ` with `n ≥ 1` — the width (count of positions mapped)

It denotes the set of position-address pairs:

`⟦β⟧ = {(v + k, a + k) : 0 ≤ k < n}`

where `v + k` and `a + k` denote ordinal displacement at the tumbler's own depth (made precise by the convention below). The *V-extent* is `V(β) = {v + k : 0 ≤ k < n}`; the *I-extent* is `I(β) = {a + k : 0 ≤ k < n}`.

**Convention (OrdinalShiftBase).** Throughout this ASN, for any tumbler `t` and natural number `k ≥ 0`:

- For `k ≥ 1`, `t + k` denotes `shift(t, k)` — the OrdinalShift of ASN-0034 at the tumbler's own depth.
- For `k = 0`, `t + 0 = t` by definition — the identity of ordinal shift.

The symbol `+` is overloaded: when the left operand is a tumbler and the right is a natural number (e.g., `v + k`, `a + n₁`), `+` denotes the ordinal shift just defined; when both operands are natural numbers (e.g., `n₁ + n₂`, `c + j`), `+` denotes ordinary natural-number addition.

This is the correspondence run of ASN-0036 S8, elevated to a first-class algebraic object. We now establish its properties.

### Width Coupling

Nelson states it directly:

> "Their width is defined by a single difference tumbler (the same in both spaces), since the V-stream and the I-stream widths must be identical." [LM 4/36]

**M0 (WidthCoupling).** For every mapping block `β = (v, a, n)`:

`|V(β)| = |I(β)| = n`

Both projections have equal cardinality, both equal to the block's width. At `n = 1`, `V(β) = {v + 0} = {v}` and `I(β) = {a + 0} = {a}` are singletons by OrdinalShiftBase, so `|V(β)| = |I(β)| = 1 = n` directly; the monotonicity argument below handles `n ≥ 2`. For all `j, k` with `0 ≤ j < k < n`: if `j = 0`, then `v + j = v` and `v + k > v` by TS4 (ShiftStrictIncrease, ASN-0034) — applicable since `k ≥ 1`; if `j ≥ 1`, then `v + j < v + k` by TS5 (ShiftAmountMonotonicity, ASN-0034) — applicable since `1 ≤ j < k`. Either way, `v + j < v + k`, so the `n` values `v + 0, v + 1, ..., v + (n − 1)` are pairwise distinct and `|V(β)| = n`. Likewise for `I(β)`.

The Vstream is an *arrangement* of Istream content — each V-position references exactly one I-byte, and each reference is to exactly one byte. There is no compression, expansion, or transformation between the spaces. The mapping is positional and unit-ratio.

Gregory's implementation confirms the structural enforcement. Each POOM bottom crum stores separate V-width and I-width tumblers — the same integer count encoded at different hierarchical depths. The construction path in `insertpm` derives both from a shared integer `inc`: it extracts the byte count from the I-width via `tumblerintdiff`, then re-encodes that same count as V-width at the V-address depth via `tumblerincrement`. No subsequent operation writes to an existing crum's width fields — the coupling is established at creation and maintained by immutability.

### Order Preservation

**M1 (OrderPreservation).** Within a mapping block `β = (v, a, n)`, the mapping preserves ordinal position. For all `j, k` with `0 ≤ j < k < n`:

`v + j < v + k  ∧  a + j < a + k`

The `j`-th V-position maps to the `j`-th I-address, and both orderings agree.

*Proof.* By the M0 argument (TS4 at `j = 0`, TS5 for `j ≥ 1`), `v + j < v + k` for `0 ≤ j < k < n`; the identical argument with `a` in place of `v` gives `a + j < a + k`. ∎

Nelson's motivation is structural:

> "The first point of a span may designate a server, an account, a document or an element; so may the last point. There is no choice as to what lies between; this is implicit in the choice of first and last point." [LM 4/25]

A span on the tumbler line is defined by its endpoints. The internal ordering follows from the total order T1 (ASN-0034). There is no reversal flag, no permutation within a span, no mechanism for a single mapping unit to represent anything other than ordinal correspondence. To represent content in reverse order requires multiple blocks, each individually monotone, arranged in the desired V-sequence.

M0 and M1 together characterize the mapping block: it is a *width-preserving monotone injection* from a contiguous V-range to a contiguous I-range. The word "injection" is precise — within a single block, distinct V-positions map to distinct I-addresses.

**Remark (Span Algebra Analogy).** A mapping block `β = (v, a, n)` is naturally analogous to a paired V-span and I-span in the sense of ASN-0053 — the obvious candidates are `(v, δ(n, #v))` and `(a, δ(n, #a))`. The analogy is not an identity: the ASN-0053 denotation `⟦(v, δ(n, #v))⟧ = {t : v ≤ t < v + n}` includes tumblers at depths other than `#v`, whereas `V(β) = {v + k : 0 ≤ k < n}` is exactly the depth-`#v` shift orbit; so `V(β)` corresponds to the depth-`#v` projection of the ASN-0053 span's denotation, not the denotation itself. Similarly, the block's split (M4 below) is analogous to applying S4 (SplitPartition, ASN-0053) to both spans at the cut point, and the block's merge (M7 below) is analogous to S3 (MergeEquivalence, ASN-0053) — but only S3's adjacent-only sub-case, since M7 forbids the overlap that S3 admits in general. Width coupling (M0) ensures the cut point in V-space determines the cut point in I-space, so the two-span analogy stays synchronized throughout.

**M-aux (OrdinalIncrementAssociativity).** For any tumbler `v` and natural numbers `c, j ≥ 0`:

`(v + c) + j = v + (c + j)`

*Proof.* For `c, j ≥ 1`, this is TS3 (ShiftComposition, ASN-0034): `shift(shift(v, c), j) = shift(v, c + j)`. The cases `c = 0` or `j = 0` follow from the OrdinalShiftBase convention introduced above: when `c = 0`, both sides equal `v + j`; when `j = 0`, both sides equal `v + c`. ∎

## The Arrangement as a Set of Blocks

A document's full arrangement is a collection of mapping blocks that together describe `M(d)`.

**Definition (Block Decomposition).** A *block decomposition* of the arrangement of document `d` is a finite set `B = {β₁, ..., βₘ}` of mapping blocks satisfying:

(B1) *Coverage.* Every V-position in `dom(M(d))` appears in exactly one block:

`(A v ∈ dom(M(d)) :: (E! j : 1 ≤ j ≤ m : v ∈ V(βⱼ)))`

(B2) *Disjointness.* No two blocks share a V-position:

`(A i, j : 1 ≤ i < j ≤ m : V(βᵢ) ∩ V(βⱼ) = ∅)`

(B3) *Consistency.* Each block correctly describes `M(d)`:

`(A j : 1 ≤ j ≤ m : (A k : 0 ≤ k < nⱼ : M(d)(vⱼ + k) = aⱼ + k))`

B1 and B2 together assert that the V-extents partition `dom(M(d))`. B3 asserts that the mapping within each block agrees with the global arrangement. The empty arrangement `M(d) = ∅` admits `B = ∅` as a decomposition (uniqueness for this case is discharged in M2).

**M-int (TumblerIntervalCharacterization).** Let `x, y ∈ dom(M(d))` and `n ≥ 1`. If `x ≤ y < x + n`, then writing `m = #x`:

- *Subspace agreement* — `subspace(y) = subspace(x)` (equivalently `(y)_1 = (x)_1`);
- *Depth equality* — `#y = m`;
- *Prefix agreement* — `(y)_j = (x)_j` for all `1 ≤ j < m`;
- *Component-`m` reduction* — `y = x + k` where `k = (y)_m − (x)_m` and `0 ≤ k < n`.

*Proof.* By S8a (ASN-0036), `#x ≥ 2`; let `m = #x`. With `n ≥ 1` and `m ≥ 2`, the action point of `δ(n, m)` is at index `m`, so TumblerAdd (ASN-0034) gives `(x + n)_i = (x)_i` for all `i < m` and `(x + n)_m = (x)_m + n`.

*Prefix and subspace agreement.* The premise gives `x ≤ y < x + n`, hence `x ≤ y ≤ x + n`. Let `p = [(x)_1, ..., (x)_{m−1}]`; since `m ≥ 2`, `#p = m − 1 ≥ 1`. Then `p ≼ x` trivially, and `p ≼ x + n` because TumblerAdd's prefix-copy clause (action point `m`) gives `(x + n)_i = (x)_i` for all `i < m`. By T5 (ContiguousSubtrees, ASN-0034) applied to `p ≼ x`, `p ≼ x + n`, and `x ≤ y ≤ x + n`, we obtain `p ≼ y`, i.e. `(y)_j = (x)_j` for all `1 ≤ j < m`. In particular `(y)_1 = (x)_1`, so `subspace(y) = subspace(x)`.

*Depth equality.* By the subspace agreement just established, S8-depth (ASN-0036) applied to that common subspace gives `#y = #x = m`, so `(y)_j` is defined for all `1 ≤ j ≤ m`.

*Component-`m` reduction.* By depth and prefix agreement, `y` and `x` agree on components 1..m−1 and share depth `m`. We derive `(x)_m ≤ (y)_m < (x)_m + n` from the two inequalities of the premise by explicit T1 (ASN-0034) appeals.

*Lower bound from `x ≤ y`:* Case-split on `x = y` vs `x < y`. If `x = y`, then `(y)_m = (x)_m` directly (equal tumblers agree componentwise), so `(x)_m ≤ (y)_m` trivially. Otherwise `x ≤ y` strengthens to `x < y`. T1 (ASN-0034) case (ii) — `x` a proper prefix of `y` — is excluded by depth equality `#x = #y = m` (a proper prefix would require `#x < #y`), so T1 case (i) applies, supplying a least divergence index `j'` with `(y)_{j'} > (x)_{j'}` and `(y)_i = (x)_i` for `i < j'`. Prefix agreement (just established) gives `(y)_i = (x)_i` for all `1 ≤ i < m`, so `j' ≥ m`; depth `m` confines defined components to `[1, m]`, forcing `j' ≤ m`. Hence `j' = m`, and T1(i) yields `(x)_m < (y)_m`. In both subcases, `(x)_m ≤ (y)_m`.

*Upper bound from `y < x + n`:* TumblerAdd (ASN-0034) at action point `m` ≤ `#x = m` produces a tumbler of length `#x = m`, so `#(x + n) = m`; combined with `#y = m`, T1 case (ii) is excluded by equal depth, and T1 case (i) supplies a least divergence index `j''` with `(y)_{j''} < (x + n)_{j''}` and `(y)_i = (x + n)_i` for `i < j''`. We pin `j''` via transitively-established prefix agreement: prefix agreement gives `(y)_i = (x)_i` for `i < m`, and TumblerAdd's prefix-copy clause (action point `m`) gives `(x + n)_i = (x)_i` for `i < m`; chaining the two, `(y)_i = (x + n)_i` for all `i < m`, hence `j'' ≥ m`. Equal depth `m` forces `j'' ≤ m`; hence `j'' = m`. TumblerAdd's action-point clause gives `(x + n)_m = (x)_m + n`, and T1(i) at `j'' = m` yields `(y)_m < (x + n)_m = (x)_m + n`.

Combining the two derivations: `(x)_m ≤ (y)_m < (x)_m + n`. Set `k = (y)_m − (x)_m`; then `0 ≤ k < n`.

*Case `k = 0`.* Then `(y)_m = (x)_m`, so `y` and `x` agree on all components 1..m (prefix agreement gives 1..m−1, the `k = 0` assignment gives component `m`) and share depth `m`. T3 (CanonicalRepresentation, ASN-0034) — componentwise agreement of equal-depth tumblers gives equality — yields `y = x`. By OrdinalShiftBase, `x + 0 = x`, hence `y = x = x + 0 = x + k`.

*Case `k ≥ 1`.* The tumbler `x + k`, computed via TumblerAdd at action point `m` (well-defined since `m ≥ 2` and `k ≥ 1`), agrees with `y` on components 1..m−1 (by prefix agreement and TumblerAdd's prefix-copy clause: `(x + k)_i = (x)_i = (y)_i` for `i < m`) and on component `m` (by `(x + k)_m = (x)_m + k = (y)_m`, the second equality by definition of `k`), and shares depth `m`. T3 yields `y = x + k`. ∎

**M2 (DecompositionExistence).** Under the standing preconditions S8-fin, S2, S3, S8a, and S8-depth (ASN-0036), every arrangement `M(d)` admits a block decomposition.

This is S8 (CorrespondenceRunPartition, ASN-0036) restated in our vocabulary. S8 produces a finite family of maximal runs `{(vⱼ, aⱼ, nⱼ)}` whose V-extents partition `dom(M(d))` as a disjoint union; reading each triple as a mapping block `βⱼ = (vⱼ, aⱼ, nⱼ)`, we must show that S8's lockstep postcondition (a) and its partition claim deliver B1, B2, B3.

*S8(a) ⟺ B3.* S8's lockstep postcondition (a) asserts `M(d)(shift(vⱼ, k)) = shift(aⱼ, k)` for `0 ≤ k < nⱼ`. Reading `shift(vⱼ, k)` as `vⱼ + k` and `shift(aⱼ, k)` as `aⱼ + k` (OrdinalShift convention, ASN-0034) makes this `M(d)(vⱼ + k) = aⱼ + k`, which is B3 verbatim. At `k = 0`, S8(a) also gives `M(d)(vⱼ) = aⱼ`, so `vⱼ ∈ dom(M(d))`.

*Partition claim ⟺ B1 ∧ B2.* S8's partition claim states that `dom(M(d))` is the disjoint union of the run V-extents, each given as `{shift(vⱼ, k) : 0 ≤ k < nⱼ}`. Reading `shift(vⱼ, k)` as `vⱼ + k` (OrdinalShift convention, ASN-0034, with `shift(vⱼ, 0) = vⱼ` by OrdinalShiftBase) identifies this set with our block V-extent `V(βⱼ) = {vⱼ + k : 0 ≤ k < nⱼ}`. Thus S8's disjoint union `⊎ⱼ V(βⱼ) = dom(M(d))` reads `(A v ∈ dom(M(d)) :: (E! j :: v ∈ V(βⱼ)))`: its *existence* half — every V-position lies in *some* block's V-extent — is B1 (Coverage), and its *uniqueness* half — no V-position lies in two blocks' V-extents — is B2 (Disjointness). The empty arrangement `M(d) = ∅` admits `B = ∅` (S8 produces zero runs; B1, B2, B3 are vacuously satisfied), and this is its *only* decomposition: any block `β = (v, a, n)` has `n ≥ 1`, hence a non-empty V-extent `V(β) ∋ v`, which by B3 forces `v ∈ dom(M(d)) = ∅` — a contradiction. So no decomposition of the empty arrangement can contain a block.

Nelson tells us:

> "There may be many representations of a given v-stream. The representation with the fewest I-spans is the most compact." [LM 4/37]

**Definition (Decomposition Equivalence).** Block decompositions `B` and `B'` of `M(d)` are *equivalent*, written `B ≡ B'`, when they denote the same mapping:

`⋃{⟦β⟧ : β ∈ B} = ⋃{⟦β⟧ : β ∈ B'}`

**M3 (RepresentationInvariance).** If `B ≡ B'`, then for every `v ∈ dom(M(d))`, the I-address determined by `B` equals the I-address determined by `B'`.

This is immediate — equivalent decompositions denote the same set of `(V, I)` pairs, which is a function by S2 (ArrangementFunctionality, ASN-0036). The arrangement `M(d)` is the invariant; the decomposition is a choice of representation.

## Splitting a Mapping Block

We now develop the operations that transform one decomposition into another. The first is splitting: given a mapping block and a cut point in its interior, we produce two smaller blocks that together are equivalent to the original.

**Definition (Interior Point).** An integer `c` is *interior* to block `β = (v, a, n)` when `0 < c < n`.

**M4 (SplitDefinition).** For a mapping block `β = (v, a, n)` and interior point `0 < c < n`, the *split at `c`* produces two blocks:

```
β_L = (v, a, c)
β_R = (v + c, a + c, n − c)
```

Both are well-formed mapping blocks: `c ≥ 1` and `n − c ≥ 1` (since `0 < c < n`), and both starts are valid tumblers (by TA0, ASN-0034: the action point of `δ(c, #v)` is `#v`, satisfying the precondition `k ≤ #v`; similarly for `δ(c, #a)` and `#a`).

**M5 (SplitPartition).** The split is exact — nothing lost, nothing duplicated:

(a) `⟦β_L⟧ ∪ ⟦β_R⟧ = ⟦β⟧`

(b) `⟦β_L⟧ ∩ ⟦β_R⟧ = ∅`

*Verification of (a).* `⟦β_L⟧ = {(v + k, a + k) : 0 ≤ k < c}` and `⟦β_R⟧ = {((v + c) + j, (a + c) + j) : 0 ≤ j < n − c}`. Setting `k = c + j` — so that `(v + c) + j = v + (c + j) = v + k` by M-aux — the union covers `{(v + k, a + k) : 0 ≤ k < n} = ⟦β⟧`. ∎

*Verification of (b).* `V(β_L) = {v + k : 0 ≤ k < c}` and `V(β_R) = {v + k : c ≤ k < n}`. The ranges `[0, c)` and `[c, n)` are disjoint as integer ranges. The map `k ↦ v + k` on `[0, n)` is injective: by the M0 argument (TS4 at `k = 0`, TS5 for `k ≥ 1`), `0 ≤ k < k' < n` implies `v + k < v + k'`, so distinct `k` produce distinct tumblers. Therefore the disjoint integer ranges `[0, c)` and `[c, n)` map to disjoint V-extents, giving `V(β_L) ∩ V(β_R) = ∅`. A pair in `⟦β_L⟧ ∩ ⟦β_R⟧` would have its first component in `V(β_L) ∩ V(β_R)`, which is empty. ∎

What does each piece preserve? Nelson states the principle directly: "splitting is a Vstream operation that must be invisible to Istream properties." We verify each aspect.

**M6 (SplitPreservation).** Each piece is itself a mapping block, and so independently preserves every property that derives from I-address identity:

(a) *Width coupling.* `|V(β_L)| = |I(β_L)| = c` and `|V(β_R)| = |I(β_R)| = n − c`. Each piece is a mapping block, so M0 applies.

(b) *Order preservation.* Both `β_L` and `β_R` satisfy M1. Each is a mapping block; M1 holds for every mapping block.

(c) *I-address fidelity.* For every pair `(v + k, a + k)` in `⟦β⟧`, the same pair appears in exactly one of `⟦β_L⟧` or `⟦β_R⟧`. No I-address is altered, dropped, or duplicated. This is M5 restated.

The split changes how the arrangement is *represented*, not what the arrangement *is*.

**M6f (SplitFrame).** If `B` is a decomposition of `M(d)` containing `β`, then `(B \ {β}) ∪ {β_L, β_R}` is also a decomposition of `M(d)`, and the two decompositions are equivalent. All blocks in `B \ {β}` are unchanged.

*Verification.* B1 (coverage) is preserved by M5(a). B2 (disjointness) is preserved because `V(β_L) ∪ V(β_R) = V(β)` (by M5(a)) and `V(β_L) ∩ V(β_R) = ∅` (by M5(b)), so the new blocks occupy exactly the V-extent vacated by `β`, which was disjoint from all other blocks. B3 (consistency) follows from the definition of `β_L` and `β_R` — each maps its V-positions to the same I-addresses as `β` did. ∎

Gregory's implementation confirms the exactness. The `slicecbcpm` function applies the same scalar count — the V-offset of the cut — to both dimensions, using each dimension's own tumbler exponent. The resulting pieces preserve exact I-displacements and I-widths. The developer's own comment `/* I really don't understand this loop */` notwithstanding, the loop is correct precisely because the mantissa invariant (same byte count in both dimensions) is maintained through exact integer arithmetic with no rounding or alignment.

## Merging Adjacent Blocks

The inverse of splitting is merging. Nelson states the necessary and sufficient condition:

> "Two adjacent I-spans in a document may be combined if they describe V-contiguous elements which are also I-contiguous." [LM 4/36]

He restates it concretely:

> "They can be merged if one end of the next I-span can also be described as one past one end of the first." [LM 4/36]

We formalize both conditions.

**Definition (V-Adjacent).** Blocks `β₁ = (v₁, a₁, n₁)` and `β₂ = (v₂, a₂, n₂)` with `v₁ < v₂` are *V-adjacent* when `v₂ = v₁ + n₁` — the V-extent of `β₂` immediately follows that of `β₁`.

**Definition (I-Adjacent).** Blocks `β₁` and `β₂` (with `v₁ < v₂`) are *I-adjacent* when `a₂ = a₁ + n₁` — the I-extent of `β₂` immediately follows that of `β₁`.

**M7 (MergeCondition).** Let `B` be a decomposition of `M(d)`, and let `β₁ = (v₁, a₁, n₁)` and `β₂ = (v₂, a₂, n₂)` be blocks in `B` with `v₁ < v₂`. They may be merged into a single block compatible with `M(d)` if and only if they are both V-adjacent and I-adjacent:

`v₂ = v₁ + n₁  ∧  a₂ = a₁ + n₁`

When both conditions hold, the merged block is:

`β₁ ⊞ β₂ = (v₁, a₁, n₁ + n₂)`

(We write `⊞` for block merge to distinguish it from tumbler addition `⊕` of ASN-0034.)

Both conditions are necessary; V-overlap is treated separately below.

*V-adjacency alone is insufficient.* If the I-extents are not contiguous, the merged block `(v₁, a₁, n₁ + n₂)` would predict `M(d)(v₁ + n₁) = a₁ + n₁`, but the arrangement maps that position to `a₂ ≠ a₁ + n₁`, violating B3.

*I-adjacency alone is insufficient.* If `v₂ > v₁ + n₁` (the V-extents leave a gap), the merged block claims a mapping at `v₁ + n₁ ∉ V(β₁) ∪ V(β₂)`, so either (a) `v₁ + n₁ ∉ dom(M(d))`, violating B3 for the merged block (which asserts the position is in its domain); or (b) some other block `β'' ∈ B` covers `v₁ + n₁`, in which case `V(β₁ ⊞ β₂) ∩ V(β'') ⊇ {v₁ + n₁}`, violating B2 between the merged block and `β''`.

*Overlap is impossible.* The remaining case `v₂ < v₁ + n₁` is ruled out by the following sub-lemma.

**M7-cov (NonOverlap).** Let `B` be a decomposition of `M(d)` and let `β₁ = (v₁, a₁, n₁)` and `β₂ = (v₂, a₂, n₂)` be distinct blocks in `B` with `v₁ < v₂`. Then `v₂ ≥ v₁ + n₁`.

*Proof.* Since `β₁, β₂ ∈ B` and B is a decomposition of `M(d)`, B3 places `v₁, v₂ ∈ dom(M(d))`. We argue by contradiction: suppose `v₂ < v₁ + n₁`. Then `v₁ ≤ v₂ < v₁ + n₁` (the left inequality is the lemma's hypothesis `v₁ < v₂`), and `n₁ ≥ 1` by the mapping block definition. M-int (TumblerIntervalCharacterization) applied with `x = v₁`, `y = v₂`, `n = n₁` yields `v₂ = v₁ + k` for `k = (v₂)_m − (v₁)_m` with `0 ≤ k < n₁`. The case `k = 0` would give `v₂ = v₁ + 0 = v₁` (by OrdinalShiftBase), contradicting `v₁ < v₂`; hence `k ∈ [1, n₁)`, so `v₂ ∈ V(β₁)`. Combined with `v₂ ∈ V(β₂)`, this gives `v₂ ∈ V(β₁) ∩ V(β₂)`, violating B2 of the original decomposition. ∎

*Verification of the merge identity.* `⟦β₁ ⊞ β₂⟧ = {(v₁ + k, a₁ + k) : 0 ≤ k < n₁ + n₂}`. For `k < n₁`, this gives `⟦β₁⟧`. For `k ≥ n₁`, set `j = k − n₁`: then `v₁ + k = (v₁ + n₁) + j = v₂ + j` and similarly `a₁ + k = a₂ + j` (by M-aux), giving `⟦β₂⟧`. So `⟦β₁ ⊞ β₂⟧ = ⟦β₁⟧ ∪ ⟦β₂⟧`. ∎

Gregory's implementation confirms the bidimensional requirement. The `isanextensionnd` function checks `lockeq(reach.dsas, originptr->dsas, dspsize(POOM))` with `dspsize(POOM) = 2`, requiring exact tumbler equality in both I and V dimensions simultaneously. Neither dimension alone suffices.

**M7f (MergeFrame).** If `B` is a decomposition of `M(d)` containing both `β₁` and `β₂`, then `(B \ {β₁, β₂}) ∪ {β₁ ⊞ β₂}` is an equivalent decomposition. All blocks in `B \ {β₁, β₂}` are unchanged.

*Verification.* Write `β₁ ⊞ β₂ = (v₁, a₁, n₁ + n₂)`, with merge condition `v₂ = v₁ + n₁` and `a₂ = a₁ + n₁`. We check B1, B2, B3.

B1 (coverage) and B2 (disjointness) follow from the verification of the merge identity above: `V(β₁ ⊞ β₂) = V(β₁) ∪ V(β₂)`, so the merged block occupies exactly the V-extent vacated by `β₁` and `β₂`, which was disjoint from all other blocks in `B`. No V-position is gained or lost; all blocks in `B \ {β₁, β₂}` are unchanged.

B3 (consistency) for `β₁ ⊞ β₂` requires `M(d)(v₁ + k) = a₁ + k` for every `0 ≤ k < n₁ + n₂`. We case-split on `k`.

*Case `0 ≤ k < n₁`.* B3 for `β₁` gives `M(d)(v₁ + k) = a₁ + k` directly.

*Case `n₁ ≤ k < n₁ + n₂`.* Set `j = k − n₁`, so `0 ≤ j < n₂`. The V-position translation uses M-aux on the merge condition `v₂ = v₁ + n₁`:

`v₁ + k = v₁ + (n₁ + j) = (v₁ + n₁) + j = v₂ + j`

— the inner step is natural-number addition `n₁ + j = k`, and the outer step is M-aux applied to `(v, c, j) = (v₁, n₁, j)`. B3 for `β₂` at index `j` then gives `M(d)(v₂ + j) = a₂ + j`. The I-address translation uses M-aux on the merge condition `a₂ = a₁ + n₁` in the same shape:

`a₂ + j = (a₁ + n₁) + j = a₁ + (n₁ + j) = a₁ + k`

— M-aux applied to `(v, c, j) = (a₁, n₁, j)`. Chaining: `M(d)(v₁ + k) = M(d)(v₂ + j) = a₂ + j = a₁ + k`, as required. ∎

**M8 (MergeInformationLoss).** The merge is information-destroying with respect to the boundary. Given only `β₁ ⊞ β₂ = (v₁, a₁, n₁ + n₂)`, the individual widths `n₁` and `n₂` cannot be recovered. The merged block is indistinguishable from one that was never split.

This follows from the definition — the merged block is a triple `(v, a, n)` with no record of internal boundaries. Gregory confirms: a POOM bottom crum stores only `{displacement, width, homedoc}`, with no operation count, sub-span list, or boundary history. The merge at `insertnd.c:251` reduces to `dspadd` — scalar addition on the width, not annotated, not logged, not reversible. Even the spanfilade coalesces adjacent I-spans from the same source document, erasing the boundary there as well.

## The Split-Merge Duality

Split and merge are inverse operations. This is the algebraic core of the permutation model, and it holds because width coupling (M0) forces both dimensions to split and merge at the same count.

**M9 (SplitMergeInverse).** For any mapping block `β = (v, a, n)` and interior point `0 < c < n`, the two pieces produced by split satisfy the merge condition and merge back to the original:

```
split(β, c) = (β_L, β_R)
  where β_L = (v, a, c) and β_R = (v + c, a + c, n − c)

V-adjacency: v + c = v + c  ✓
I-adjacency: a + c = a + c  ✓

β_L ⊞ β_R = (v, a, c + (n − c)) = (v, a, n) = β  ∎
```

**M10 (MergeSplitInverse).** For any blocks `β₁ = (v₁, a₁, n₁)` and `β₂ = (v₂, a₂, n₂)` satisfying the merge condition (`v₂ = v₁ + n₁`, `a₂ = a₁ + n₁`), splitting the merged block at the original boundary recovers both:

```
split(β₁ ⊞ β₂, n₁)
  = ((v₁, a₁, n₁), (v₁ + n₁, a₁ + n₁, n₂))
  = (β₁, β₂)  ∎
```

M9 and M10 together establish a bijection between `{block with interior cut point}` and `{pair of mergeable blocks}`.

## The Canonical Decomposition

Among all equivalent decompositions of a given arrangement, there is a distinguished one — the one where every possible merge has been performed.

**Definition (Maximally Merged).** A block decomposition `B` is *maximally merged* when no two blocks in `B` satisfy the merge condition (M7). For every pair `βᵢ, βⱼ ∈ B` with `i ≠ j`: they are not V-adjacent, or they are not I-adjacent, or both.

**M11 (CanonicalExistence).** Every arrangement `M(d)` admits a maximally merged block decomposition.

*Construction.* Start with any decomposition `B` (which exists by M2). While there exist `βᵢ, βⱼ ∈ B` satisfying the merge condition: replace them with `βᵢ ⊞ βⱼ` (by M7f, the result is an equivalent decomposition). Each merge strictly decreases `|B|` by exactly 1 and preserves equivalence. The process terminates by well-foundedness of ℕ — `|B| ∈ ℕ` strictly decreases on every merge step, and `ℕ` admits no infinite strictly-descending chain — so after finitely many merges no mergeable pair remains. ∎

We must now establish that the result is independent of merge order.

**M12 (CanonicalUniqueness).** The maximally merged decomposition is unique.

Define a *maximal run* of `f = M(d)` as a triple `(v, a, n)` such that:
1. `(A k : 0 ≤ k < n : f(v + k) = a + k)` — it is a correspondence run
2. `¬(E v' :: v' + 1 = v ∧ v' ∈ dom(f) ∧ f(v') + 1 = a)` — it cannot be extended left
3. `v + n ∉ dom(f)  ∨  f(v + n) ≠ a + n` — it cannot be extended right

The proof factors into two sub-lemmas: M12a (maximal runs partition `dom(f)`) and M12b (every block of a maximally merged decomposition is a maximal run).

**M12a (RunDisjointness).** Maximal runs of `f` pairwise have disjoint V-extents: if `R₁ = (v₁, a₁, n₁)` and `R₂ = (v₂, a₂, n₂)` are maximal runs of `f` with `V(R₁) ∩ V(R₂) ≠ ∅`, then `(v₁, a₁, n₁) = (v₂, a₂, n₂)`.

*Proof.* Suppose `v ∈ V(R₁) ∩ V(R₂)` with `v₁ ≤ v₂` WLOG. Then `v = v₁ + k` for some `0 ≤ k < n₁` and `v = v₂ + k'` for some `0 ≤ k' < n₂`. Since `v₁, v₂ ∈ dom(f) ⊆ dom(M(d))` (both are starts of correspondence runs), S8a (VPositionWellFormedness, ASN-0036) gives `#v₁ ≥ 2` and `#v₂ ≥ 2`.

*Equal starts.* We show `(v₁, a₁) = (v₂, a₂)` by case-splitting on the start positions.

*Case `v₁ = v₂`:* Set `k₂ = 0`. Condition 1 of `R₁` at offset 0 gives `f(v₁) = f(v₁ + 0) = a₁ + 0 = a₁` (using OrdinalShiftBase), and similarly condition 1 of `R₂` gives `f(v₂) = a₂`. Functionality of `f = M(d)` (S2, ArrangementFunctionality, ASN-0036) applied at `v₁ = v₂` yields `a₁ = f(v₁) = f(v₂) = a₂`. So this case delivers both `v₁ = v₂` and `a₁ = a₂` directly, ready for Equal widths below.

*Case `v₁ < v₂`:* Since `v₂ ≤ v = v₁ + k` with `k < n₁`, we have `v₁ < v₂ < v₁ + n₁`. M-int (TumblerIntervalCharacterization) applied with `x = v₁`, `y = v₂`, `n = n₁` (premises: `v₁, v₂ ∈ dom(M(d))` since both are V-starts of correspondence runs in `dom(f) ⊆ dom(M(d))`; `v₁ ≤ v₂ < v₁ + n₁` from above; `n₁ ≥ 1` by M0) gives `v₂ = v₁ + k₂` for `k₂ = (v₂)_m − (v₁)_m` with `0 ≤ k₂ < n₁`, with `v₁` and `v₂` sharing subspace of common depth `m`. The strict `v₁ < v₂` rules out `k₂ = 0` (which would give `v₂ = v₁`), so `1 ≤ k₂ ≤ k < n₁`. Both runs map `v₂` through `f` — condition 1 of `R₁` gives `f(v₂) = f(v₁ + k₂) = a₁ + k₂`, and condition 1 of `R₂` gives `f(v₂) = a₂` — so `a₂ = a₁ + k₂`. Now set `v' = v₁ + (k₂ − 1) ∈ V(R₁) ⊆ dom(f)`. By M-aux, `v' + 1 = v₁ + k₂ = v₂` and `f(v') + 1 = (a₁ + (k₂ − 1)) + 1 = a₁ + k₂ = a₂`. So `R₂` can be extended left, contradicting condition 2 of `R₂`. The contradiction excludes this case, leaving only `v₁ = v₂` (with `a₁ = a₂` from the surviving case).

*Equal widths.* By NAT-order trichotomy on `(n₁, n₂)`, exactly one of `n₁ < n₂`, `n₁ = n₂`, or `n₂ < n₁` holds. Suppose `n₁ < n₂`. Then `v₁ + n₁ ∈ V(R₂)` (at offset `n₁ < n₂` from `v₂ = v₁`), so `v₁ + n₁ ∈ dom(f)` and `f(v₁ + n₁) = a₂ + n₁ = a₁ + n₁` by condition 1 of `R₂`. But condition 3 of `R₁` requires `v₁ + n₁ ∉ dom(f) ∨ f(v₁ + n₁) ≠ a₁ + n₁` — contradiction. The symmetric case `n₂ < n₁` contradicts condition 3 of `R₂` by the same argument (with `R₁` supplying the right-witness). The surviving case is `n₁ = n₂`, giving `(v₁, a₁, n₁) = (v₂, a₂, n₂)`. ∎

*Partition corollary.* Every `v ∈ dom(f)` belongs to at least one maximal run. Start with the trivial run `R₀ = (v, f(v), 1)`, which satisfies condition 1: at `k = 0`, `f(v + 0) = f(v) = a + 0` (using OrdinalShiftBase). Extend in two phases.

*Right-extension phase.* Given a run `R = (v_R, a_R, n)` satisfying condition 1, test condition 3: is `v_R + n ∉ dom(f)` or `f(v_R + n) ≠ a_R + n`? If yes, stop (condition 3 now holds for `R`). Otherwise — `v_R + n ∈ dom(f)` and `f(v_R + n) = a_R + n` — replace `R` by `R' = (v_R, a_R, n + 1)`. Condition 1 is preserved: it held at `0 ≤ k < n` in `R`, and the new index `k = n` is exactly what the test verified. `V(R') = V(R) ∪ {v_R + n} ⊋ V(R)`.

*Left-extension phase.* Given a run `R = (v_R, a_R, n)` satisfying conditions 1 and 3, test condition 2: does some `v' ∈ dom(f)` satisfy `v' + 1 = v_R` and `f(v') + 1 = a_R`? If no such `v'` exists, stop (condition 2 holds for `R`). Otherwise replace `R` by `R' = (v', f(v'), n + 1)` — note this uses TumblerAdd only (we never decrement). Condition 1 is preserved: at `k = 0`, `f(v') = a_{R'}` directly; for `1 ≤ k < n + 1`, M-aux gives `v' + k = (v' + 1) + (k − 1) = v_R + (k − 1)`, and `R`'s condition 1 at index `k − 1` gives `f(v_R + (k − 1)) = a_R + (k − 1) = (f(v') + 1) + (k − 1) = a_{R'} + k`. Condition 3 is preserved: by M-aux, `v' + (n + 1) = (v' + 1) + n = v_R + n`, and `a_{R'} + (n + 1) = f(v') + (n + 1) = (f(v') + 1) + n = a_R + n`, so the boundary test on `R'` matches the (already-verified) boundary test on `R`. `V(R')` adds `v'`, which is distinct from every `v_R + k` (`k ≥ 0`) by T1(i) strict ordering: `v' < v' + 1 = v_R ≤ v_R + k`.

*Termination.* By S8-fin (ASN-0036), `dom(M(d))` is finite, hence `dom(f) ⊆ dom(M(d))` is finite. Each extension step strictly enlarges `V(R) ⊆ dom(f)` by one element, so each phase terminates after at most `|dom(f)|` steps. The resulting run `R*` satisfies conditions 1, 2, and 3 — a maximal run with `v ∈ V(R₀) ⊆ V(R*)`.

By M12a, `v` belongs to at most one maximal run. So the set of maximal runs partitions `dom(f)` and is uniquely determined by `f`.

**M12b (NoExtensionInMaximallyMerged).** Let `B` be a maximally merged decomposition of `M(d)`. Every block `β = (v, a, n) ∈ B` satisfies conditions 2 and 3 of being a maximal run of `f = M(d)`: it cannot be left-extended or right-extended in `f`.

*Proof.* We prove the contrapositives.

*No right-extension.* Suppose condition 3 fails: `v + n ∈ dom(f)` and `f(v + n) = a + n`. Some block `β' = (v', a', n') ∈ B` covers `v + n`, so `v + n = v' + j` for some `0 ≤ j < n'`. We first note `β' ≠ β`: if `β' = β`, then `v' = v` and `n' = n`, so `v + n = v + j` with `0 ≤ j < n`; but the map `i ↦ v + i` is strictly monotone on `[0, n]` (by the M0 argument: TS4 at `i = 0`, TS5 for `i ≥ 1`), so `v + j = v + n` is impossible. Hence `β' ≠ β`. We show `j = 0`. Suppose for contradiction `j ≥ 1`. Since `v, v' ∈ dom(M(d))` (both are V-starts of blocks in `B`), S8a (ASN-0036) gives `#v ≥ 2` and `#v' ≥ 2`, discharging the depth preconditions of OrdShiftHom (ASN-0036). With `j ≥ 1` and `n ≥ 1` supplying the positive-shift preconditions, OrdShiftHom gives `subspace(v' + j) = subspace(v')` and `subspace(v + n) = subspace(v)`; since `v' + j = v + n`, `subspace(v') = subspace(v)`. By S8-depth (ASN-0036) applied to that subspace, `#v' = #v = m ≥ 2`. For depth of `v + (n − 1)`: if `n = 1`, then `v + (n − 1) = v + 0 = v` by OrdinalShiftBase, which has depth `m`; if `n ≥ 2`, then `n − 1 ≥ 1` and OrdinalShift's postcondition `#shift(v, n − 1) = #v` (ASN-0034) gives `v + (n − 1)` depth `m`. The same case split for `v' + (j − 1)` — using OrdinalShift's `#shift(v', j − 1) = #v'` in the `j ≥ 2` subcase — gives depth `m`. By M-aux, `v + n = (v + (n − 1)) + 1` and `v + n = v' + j = (v' + (j − 1)) + 1`. The unit shift `δ(1, m)` has action point `m`: applied to a depth-`m` tumbler `x`, TumblerAdd (ASN-0034) produces `[x₁, ..., x_{m−1}, x_m + 1]`. Equal outputs force equal inputs componentwise — agreement below `m`, plus `x_m + 1 = y_m + 1` forcing `x_m = y_m`. So `v + (n − 1) = v' + (j − 1)`. Since `v + (n − 1) ∈ V(β)` and `v' + (j − 1) ∈ V(β')`, `V(β) ∩ V(β') ≠ ∅`, contradicting B2. Hence `j = 0` and `v' = v + n`. Then `a' = f(v + n) = a + n`, so `β` and `β'` are V-adjacent (`v' = v + n`) and I-adjacent (`a' = a + n`) — contradicting `B` being maximally merged.

*No left-extension.* Suppose condition 2 fails: there exists `v'` with `v' + 1 = v`, `v' ∈ dom(f)`, and `f(v') + 1 = a`. Some block `β'' = (v'', a'', n'') ∈ B` covers `v'`, so `v' = v'' + k` for some `0 ≤ k < n''`. Note `β'' ≠ β`: from `v' + 1 = v`, TS4 (ASN-0034) gives `v' < v' + 1 = v`; `V(β) = {v + i : 0 ≤ i < n}` has every element `≥ v` (OrdinalShiftBase at `i = 0`, TS4 for `i ≥ 1`), so `v' < v` places `v' ∉ V(β)`. Since `β''` covers `v' ∉ V(β)`, `β'' ≠ β`. By M-aux, `v = v' + 1 = (v'' + k) + 1 = v'' + (k + 1)`. If `k + 1 < n''`, then `v'' + (k + 1) ∈ V(β'')`, so `v ∈ V(β'') ∩ V(β)`, contradicting B2 (which applies since `β'' ≠ β`). Hence `k + 1 = n''` (combined with `k < n''`), so `v' = v'' + (n'' − 1)`. By M-aux again, `v'' + n'' = (v'' + (n'' − 1)) + 1 = v' + 1 = v` (V-adjacent). And `a'' + n'' = (a'' + (n'' − 1)) + 1 = f(v') + 1 = a` (I-adjacent, since `f(v') = a'' + (n'' − 1)` by B3). So `β''` and `β` satisfy the merge condition — contradicting `B` being maximally merged. ∎

*Proof of M12.* (⟹) Let `B` be maximally merged, and let `R` denote the set of maximal runs of `f`. By M12b, every `β ∈ B` satisfies conditions 2 and 3 of being a maximal run; condition 1 is B3 applied to `β`. So every block of `B` is a maximal run — this is the forward inclusion `B ⊆ R`. For the reverse inclusion `R ⊆ B`: fix any maximal run `R* = (v_R, a_R, n_R) ∈ R`. Condition 1 of being a maximal run, instantiated at `k = 0`, gives `f(v_R + 0) = a_R + 0`, i.e. `f(v_R) = a_R`, hence `v_R ∈ dom(f)`. By B1 (Coverage) applied to `B`, some block `β ∈ B` satisfies `v_R ∈ V(β)`; by the forward inclusion just established, this `β` is itself a maximal run. So `R*` and `β` are both maximal runs containing `v_R` in their V-extents, hence with `V(R*) ∩ V(β) ≠ ∅`. M12a (RunDisjointness) then forces `R* = β`, so `R* ∈ B`. The two inclusions give `B = R` — `B` is exactly the set of maximal runs.

(⟸) The set of maximal runs `R` is maximally merged. Fix any two distinct maximal runs `β_1 = (v_1, a_1, n_1)` and `β_2 = (v_2, a_2, n_2)` in `R`, and suppose for the M7 question they satisfy V-adjacency `v_2 = v_1 + n_1`. We show M7's I-adjacency premise `a_2 = a_1 + n_1` necessarily fails. Condition 3 of `β_1` (no right-extension) is the disjunction `v_1 + n_1 ∉ dom(f)  ∨  f(v_1 + n_1) ≠ a_1 + n_1`. Condition 1 of `β_2` at `k = 0` gives `f(v_2 + 0) = a_2 + 0`, i.e. `f(v_2) = a_2`, hence `v_2 ∈ dom(f)`; substituting `v_1 + n_1 = v_2` (V-adjacency) yields `v_1 + n_1 ∈ dom(f)`, which eliminates the first disjunct of condition 3. The remaining disjunct must hold: `f(v_1 + n_1) ≠ a_1 + n_1`. Substituting `v_1 + n_1 = v_2` on the left side and `f(v_2) = a_2` (the just-derived condition-1 instance) reduces this to `a_2 ≠ a_1 + n_1` — exactly the negation of M7's I-adjacency premise. So `β_1` and `β_2` cannot satisfy M7. Since this applies to every V-adjacent pair in `R`, no two distinct maximal runs are mergeable, and `R` is maximally merged.

Since the maximal runs are uniquely determined by `f` and every maximally merged decomposition equals the set of maximal runs, the maximally merged decomposition is unique. ∎

Nelson observes: "The representation with the fewest I-spans is the most compact." [LM 4/37] The maximally merged decomposition is this most compact representation — uniquely determined by the arrangement `M(d)`, independent of any choice of representation or any history of how the arrangement was constructed.

### A Worked Example (Canonical Decomposition)

Consider a document `d` with eight text-subspace V-positions, `[1, k]` for `k = 1, ..., 8`, arranged as follows (tumblers shown are element-field ordinals; the full document prefix `N.0.U.0.D.0.` is elided):

| V-position | I-address |
|------------|-----------|
| `[1, 1]` | `[1, 10]` |
| `[1, 2]` | `[1, 11]` |
| `[1, 3]` | `[1, 12]` |
| `[1, 4]` | `[1, 13]` |
| `[1, 5]` | `[1, 14]` |
| `[1, 6]` | `[1, 40]` |
| `[1, 7]` | `[1, 41]` |
| `[1, 8]` | `[1, 42]` |

We start with a three-block decomposition `B = {β₁, β₂, β₃}`:

- `β₁ = ([1, 1], [1, 10], 3)` — V: `[1, 1]..[1, 3]`, I: `[1, 10]..[1, 12]`
- `β₂ = ([1, 4], [1, 13], 2)` — V: `[1, 4]..[1, 5]`, I: `[1, 13]..[1, 14]`
- `β₃ = ([1, 6], [1, 40], 3)` — V: `[1, 6]..[1, 8]`, I: `[1, 40]..[1, 42]`

Assume all eight I-addresses originate from a single source document, so the elided prefix `N.0.U.0.D.0.` is the same for every address shown; the I-adjacency comparisons below operate within that common prefix.

The V-extents partition `{[1, k] : 1 ≤ k ≤ 8}`, and each block correctly describes `M(d)` — B1–B3 are satisfied.

**Merge check.** We test the merge condition (M7) on each V-adjacent pair:

- `β₁` and `β₂`: V-adjacent? `v₂ = [1, 4] = [1, 1] + 3` ✓. I-adjacent? `a₂ = [1, 13] = [1, 10] + 3` ✓. Both conditions hold — the blocks merge to `β₁ ⊞ β₂ = ([1, 1], [1, 10], 5)`.

- `β₂` and `β₃`: V-adjacent? `v₃ = [1, 6] = [1, 4] + 2` ✓. I-adjacent? `a₃ = [1, 40] ≠ [1, 13] + 2 = [1, 15]` ✗. The I-extents are not contiguous — cannot merge.

After merging, the decomposition is `B' = {([1, 1], [1, 10], 5),\; ([1, 6], [1, 40], 3)}`.

**Canonicality check.** The surviving pair: V-adjacent? `[1, 6] = [1, 1] + 5` ✓. I-adjacent? `[1, 40] ≠ [1, 10] + 5 = [1, 15]` ✗. No mergeable pair remains, so `B'` is maximally merged. By M12, this is the unique canonical decomposition.

The boundary at V-position `[1, 6]` persists because V-adjacency holds but I-adjacency does not — confirming M7's necessity. The I-addresses jump from `[1, 14]` to `[1, 40]`, indicating that content at `[1, 6]..[1, 8]` was allocated at a different point in the Istream than content at `[1, 1]..[1, 5]`.

## Shared Content

The function `M(d)` is not necessarily injective — the same I-address can appear at multiple V-positions.

**M13 (SharedContent).** The arrangement `M(d)` permits multiple V-positions to share the same I-address:

`(E Σ : Σ satisfies S0–S3 : (E d, a :: |{v : M(d)(v) = a}| > 1))`

*Derivation.* S5 (UnrestrictedSharing, ASN-0036) establishes that for any `N ∈ ℕ`, the invariants S0–S3 are consistent with sharing multiplicity exceeding `N` — both across documents and *within a single document*. S5's within-document construction at parameter `N` yields a state `Σ_N` satisfying S0–S3 with a document `d` and I-address `a` such that `|{v : v ∈ dom(M(d)) ∧ M(d)(v) = a}| = N + 1`. Instantiating at `N = 1` gives `Σ_1` with `|{v : M(d)(v) = a}| = 2 > 1`, witnessing the existential claim. ∎

This is transclusion within a single document — the same content appearing at multiple points in the same arrangement. Nelson confirms the mechanism is unrestricted:

> "Bytes native elsewhere have an ordinal position in the byte stream just as if they were native to the document." [LM 4/11]

The same content can be included at multiple positions. Each occurrence is a separate mapping block — the blocks share I-extents but have disjoint V-extents (by B2). S5 establishes that no bound limits the number of V-positions referencing a given I-address; M13 records the consequence at the smallest non-trivial multiplicity.

**M14a (SharedIExtentUnmergeable).** Any two distinct blocks `β₁ = (v₁, a₁, n₁)`, `β₂ = (v₂, a₂, n₂)` whose I-extents share at least one position cannot satisfy I-adjacency.

*Verification.* Suppose `a₁ + i = a₂ + j` for some `0 ≤ i < n₁` and `0 ≤ j < n₂`. The merge premise `a₂ = a₁ + n₁` would give, via M-aux, `a₁ + i = (a₁ + n₁) + j = a₁ + (n₁ + j)`. Since `n₁ ≥ 1`, `n₁ + j ≥ 1`; by TS4 (ASN-0034) `a₁ + 0 = a₁ < a₁ + (n₁ + j)`, so `i ≠ 0`, and by TS5 (ASN-0034) ordinal shift is strictly monotone on positive displacements, so `a₁ + i = a₁ + (n₁ + j)` with `i, n₁ + j ≥ 1` forces `i = n₁ + j ≥ n₁` — contradicting `i < n₁`. So I-adjacency is unsatisfiable whenever the I-extents overlap. ∎

**M14 (IndependentOccurrences, corollary of M14a).** In particular, two mapping blocks `β₁ = (v₁, a, n)` and `β₂ = (v₂, a, n)` sharing I-start `a` and width `n` (with `v₁ ≠ v₂`) have identical — hence overlapping — I-extents, so by M14a they cannot merge: distinct V-positions referencing the same content are permanently independent entries.

The mapping block algebra does not conflate shared content — each occurrence is a separate representational entity.

## Document Independence

Each document's arrangement is independently represented. This is a direct consequence of ASN-0036's framework — `M(d)` is per-document — but it has concrete consequences for the mapping block algebra.

**M15 (MappingIndependence).** For any two documents `d₁ ≠ d₂`:

(a) Block decompositions are per-document objects; membership of a triple `(v, a, n)` in a decomposition of `M(d₁)` entails no relationship to any decomposition of `M(d₂)`.

(b) *Frame condition.* The split frame M6f and the merge frame M7f, applied to a decomposition `B` of `M(d₁)`, name and modify only `B` itself; no block of any decomposition of `M(d₂)` is named, read, or modified by either operation. In particular, every block in every decomposition of `M(d₂)` is unchanged by any split or merge on `B`.

*Derivation of (a).* Definitional. A block decomposition `B` of `M(d_1)` is a finite set of triples whose B1, B2, B3 conditions quantify over `M(d_1)` exclusively — B1 over `dom(M(d_1))`, B3 over `M(d_1)`'s function values. No B-condition references `M(d_2)` or any of its decompositions, so the well-formedness of `B` is independent of every decomposition of `M(d_2)`, and vice versa.

*Derivation of (b).* The two transformations involved are syntactic set operations on a single decomposition:
- M6f rewrites `B` to `(B \ {β}) ∪ {β_L, β_R}` — the only inputs are `B`, `β`, and the cut point;
- M7f rewrites `B` to `(B \ {β_1, β_2}) ∪ {β_1 ⊞ β_2}` — the only inputs are `B`, `β_1`, and `β_2`.

Neither transformation reads, names, or writes to any other decomposition. Since `M(d_2)`'s decompositions are not in the operation's input or output, they are unchanged. ∎

Nelson states the principle at the arrangement level:

> "Note that the owner of a document may delete bytes from the owner's current version, but those bytes remain in all other documents where they have been included." [LM 4/11]

## Cross-Origin Merge Impossibility

The merge condition (M7) interacts naturally with the tumbler address structure. We first establish a lemma about origin invariance under ordinal shift; the cross-origin merge impossibility then follows as a corollary.

**M16a (OriginInvarianceUnderShift).** For any `a ∈ dom(C)` and any `k ≥ 0` with `a + k ∈ dom(C)`:

`origin(a + k) = origin(a)`

*Proof.* At `k = 0`, `a + 0 = a` by OrdinalShiftBase, so the equality is immediate.

For `k ≥ 1`: under the standing precondition that the content store `C` is populated by a system conforming to T10a (ASN-0034), T10a.4 (T4PreservationUnderDiscipline, ASN-0034) gives T4-validity of every `a ∈ dom(C)`, and S7b (ElementLevelIAddresses, ASN-0036) applied to `a ∈ dom(C)` gives `zeros(a) = 3`. Together — T4-validity ensures the three separator zeros occupy structurally well-defined positions, and `zeros(a) = 3` fixes their count — they structurally decompose `a` into a document prefix `N(a).0.U(a).0.D(a)` followed by the separator zero and the element field `E(a)`, with `#a = #(N(a).0.U(a).0.D(a)) + 1 + #E(a)` (the `+1` accounts for the separator zero between `D` and `E`). T4a (SyntacticEquivalence, ASN-0034) makes every field segment of a T4-valid address non-empty, so `#E(a) ≥ 1`, whence `#(N(a).0.U(a).0.D(a)) = #a − #E(a) − 1 ≤ #a − 2` — every index of the document prefix lies strictly below the action point `#a`.

The shift `a + k = a ⊕ δ(k, #a)` has action point `#a`. Let `z₃ = #a − #E(a)` be the position of `a`'s third separator zero (between `D(a)` and `E(a)`); since `#E(a) ≥ 1`, `z₃ ≤ #a − 1`, so the document prefix and all three separator zeros lie at indices `< #a`. By TumblerAdd (ASN-0034), every component at indices `i < #a` is copied unchanged from `a` to `a + k`; in particular `a + k` agrees with `a` on positions `[1, z₃]`, including all three separator zeros. T10a.4 and S7b applied to `a + k ∈ dom(C)` give T4-validity and `zeros(a + k) = 3`. Since `a + k` already carries three zeros at `a`'s separator positions (all `< #a`) and has exactly three zeros total, its third separator also sits at `z₃`, so its document prefix is `(a + k)[1, z₃ − 1] = a[1, z₃ − 1]`. By T3 (CanonicalRepresentation, ASN-0034), componentwise agreement of equal-depth tumblers gives `N(a+k).0.U(a+k).0.D(a+k) = N(a).0.U(a).0.D(a)`, hence `origin(a + k) = origin(a)`. ∎

Ordinal increment never crosses an origin boundary, because the document prefix lies strictly below the action point.

**M16b (SplitOriginTraceability, Corollary of M16a).** When mapping block `β = (v, a, n)` belongs to a decomposition `B` of `M(d)`, every I-address in `I(β)` shares `β`'s origin:

`(A k : 0 ≤ k < n : origin(a + k) = origin(a))`

Consequently, after splitting `β` at any interior point `c` (M4) into `β_L = (v, a, c)` and `β_R = (v + c, a + c, n − c)`, each piece independently identifies the home document of its content — `β_L`'s I-addresses and `β_R`'s I-addresses all share `origin(a)`.

*Derivation.* Fix `0 ≤ k < n`. B3 (Consistency) applied to `β ∈ B` gives `M(d)(v + k) = a + k`, so `v + k ∈ dom(M(d))`. S3 (ReferentialIntegrity, ASN-0036) applied to `v + k` yields `M(d)(v + k) ∈ dom(C)`, hence `a + k ∈ dom(C)`. This discharges M16a's precondition at `(a, k)`, and M16a delivers `origin(a + k) = origin(a)`. The conclusion for `β_L` and `β_R` follows: `I(β_L) = {a + k : 0 ≤ k < c} ⊆ I(β)` and `I(β_R) = {a + c + j : 0 ≤ j < n − c} = {a + k : c ≤ k < n} ⊆ I(β)` (the second equality is M-aux), so each piece's I-addresses inherit the common origin. ∎

**M16 (CrossOriginMergeImpossibility).** Let `β₁ = (v₁, a₁, n₁)` and `β₂ = (v₂, a₂, n₂)` be blocks with `a₁, a₂ ∈ dom(C)`. If `origin(a₁) ≠ origin(a₂)` — the I-addresses were allocated by different documents — then the blocks cannot satisfy I-adjacency:

`(A β₁, β₂ : a₁, a₂ ∈ dom(C) ∧ origin(a₁) ≠ origin(a₂) : ¬(a₂ = a₁ + n₁))`

*Proof.* The mapping block definition requires `n ≥ 1` as part of well-formedness, so `n₁ ≥ 1`. Suppose for contradiction `a₂ = a₁ + n₁`. Then `a₁ + n₁ = a₂ ∈ dom(C)` (by hypothesis on `β₂`), discharging M16a's precondition. M16a applied to `a₁` with `k = n₁` gives `origin(a₁ + n₁) = origin(a₁)`, hence `origin(a₂) = origin(a₁)` — contradicting the hypothesis `origin(a₁) ≠ origin(a₂)`. So `a₂ ≠ a₁ + n₁`. ∎

Gregory's implementation includes an explicit `homedoc` guard as the first check in `isanextensionnd`, short-circuiting full I-address comparison when the two blocks have distinct document origins.

The consequence is that the canonical decomposition naturally preserves origin boundaries. In a maximally merged decomposition, every block maps to a contiguous I-range under a single document prefix. Blocks spanning multiple origins cannot arise, because the I-addresses of distinct origins are never adjacent on the tumbler line.

## Content References

The block algebra characterizes how arrangements decompose into contiguous runs. We now define content references — a mechanism for identifying a span of positions within a document's arrangement — and resolution, which extracts the I-address runs from the block decomposition restricted to that span. We work with the content store C : T ⇀ Val and per-document arrangement M(d) : T ⇀ T from ASN-0036, both state-indexed by the ambient state Σ. Define the state-indexed *document set* `D(Σ) = {d : M(Σ, d) is defined}` — the documents for which an arrangement is defined in state Σ. As with the abbreviation `M(d)` for `M(Σ, d)`, we write `D` for `D(Σ)` when the ambient state is clear from context. For a subspace identifier S and a document d ∈ D, write `V_S(d) := {v ∈ dom(M(d)) : subspace(v) = S}` — the V-positions of M(d) in subspace S. This generalizes ASN-0036's `V_1(d)` (the text-subspace projection) to arbitrary subspace identifiers; in particular, `V_{u₁}(d)` denotes the projection in the subspace indexed by the first component of a span-start `u`.

**Definition (ContentReference).** A *content reference* is a pair (d_s, σ) where d_s ∈ D and σ = (u, ℓ) is a level-uniform V-span satisfying: (i) V_{u₁}(d_s) ≠ ∅ — the subspace contains at least one V-position; (ii) T12 (ASN-0034) holds (equivalently: Pos(ℓ) and actionPoint(ℓ) ≤ #u, the preconditions T12 names); and (iii) `#ℓ = #u = m`, where m is the common V-position depth in subspace u₁ of d_s (S8-depth, ASN-0036). The level-uniformity requirement ensures reach(σ) has depth m (S6, ASN-0053), so the position range is well-bounded. The content reference is well-formed when every depth-m position in the span's range belongs to d_s's arrangement:

`{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d_s))`

The common depth satisfies `m ≥ 2`. Given (i), some `v ∈ V_{u₁}(d_s)` exists, so S8a (VPositionWellFormedness, ASN-0036) gives `#v ≥ 2` and S8-depth gives `m = #v ≥ 2`.

**C0 (OrdinalDisplacementNecessity).** For a well-formed content reference (d_s, σ) with σ = (u, ℓ), common depth m, and action point k of ℓ: k = m. Equivalently, ℓ = δ(ℓₘ, m) — an ordinal displacement.

*Derivation.* ActionPoint's postcondition (ASN-0034) gives `1 ≤ actionPoint(ℓ) ≤ #ℓ`; instantiating at `#ℓ = m` yields `1 ≤ k ≤ m`. It remains to rule out `k < m`. Suppose for contradiction that k < m. Consider the family of depth-m tumblers wⱼ = [u₁, ..., uₖ, uₖ₊₁, ..., u_{m−1}, j] for j > uₘ. Each wⱼ is a member of `T`: by T0's comprehension clause (CarrierSetDefinition, ASN-0034), `T` is the set of nonempty finite sequences over `ℕ` of length ≥ 1, and `wⱼ` is a length-`m` sequence whose first `m − 1` components are `u`'s components (each in `ℕ` since `u ∈ T`) and whose final component is `j ∈ ℕ` with `j > uₘ`. Each wⱼ satisfies u < wⱼ: the two agree on components 1 through m − 1 and j > uₘ at component m, so wⱼ > u by T1(i) (ASN-0034). Each wⱼ satisfies wⱼ < reach(σ): at component k, uₖ < uₖ + ℓₖ (since ℓₖ ≥ 1, k being the action point), so wⱼ < reach(σ) by T1(i). Thus wⱼ ∈ ⟦σ⟧ for every j > uₘ. Since `ℕ` is unbounded (NAT-closure's successor closure `n + 1 ∈ ℕ` combined with NAT-addcompat's strict successor inequality `n < n + 1`, ASN-0034), `{j ∈ ℕ : j > uₘ}` is infinite, and the map `j ↦ wⱼ` is injective (distinct `j` differ at component `m`), so ⟦σ⟧ contains infinitely many distinct depth-m tumblers. Well-formedness requires each to be in dom(M(d_s)), contradicting S8-fin (ASN-0036). Hence k < m is impossible, leaving k = m, and ℓ = [0, ..., 0, ℓₘ] = δ(ℓₘ, m). ∎

**C0a (PrefixConfinement).** For a well-formed content reference (d_s, σ) with σ = (u, ℓ) and m ≥ 2: every t ∈ ⟦σ⟧ satisfies tⱼ = uⱼ for all 1 ≤ j < m.

*Derivation.* By C0, the action point of ℓ is m. Since m ≥ 2, TumblerAdd gives reach(σ)ⱼ = uⱼ for all j < m. Let `p = [u₁, ..., u_{m−1}]`; since m ≥ 2, `#p = m − 1 ≥ 1`. Then `p ≼ u` trivially, and `p ≼ reach(σ)` since reach(σ)ⱼ = uⱼ for all j < m and `#reach(σ) = m` (S6, ASN-0053). Fix any t ∈ ⟦σ⟧, so u ≤ t < reach(σ), hence u ≤ t ≤ reach(σ). By T5 (ContiguousSubtrees, ASN-0034) applied to `p ≼ u`, `p ≼ reach(σ)`, and `u ≤ t ≤ reach(σ)`, we obtain `p ≼ t`, i.e. tⱼ = uⱼ for all 1 ≤ j < m. In particular, t₁ = u₁ (subspace confinement). ∎

**Definition (ContentReferenceSequence).** A *content reference sequence* is an ordered list R = ⟨r₁, ..., rₚ⟩ of content references with p ≥ 1. Different references may name different source documents.


## Resolution

To resolve a content reference, we extract the I-address runs corresponding to the named V-span. The source document's mapping may not be ordinal-contiguous across the full span — prior editing may have interleaved content from multiple allocations, fragmenting the V→I mapping into several contiguous I-address runs.

**Definition (Resolution).** Given content reference (d_s, σ) with σ = (u, ℓ), let f = M(d_s)|⟦σ⟧ be the restriction of M(d_s) to positions in ⟦σ⟧.

**C1a (RestrictionDecomposition).** M11 and M12 hold for any restriction `f = M(d_s)|X` of an arrangement `M(d_s)` to a subset `X ⊆ T` whose induced domain `dom(f)` lies within a single subspace. In particular, the restriction f = M(d_s)|⟦σ⟧ is of this form and admits a unique maximally merged block decomposition.

*Verification that f is a well-formed restriction.* Because `f = M(d_s)|⟦σ⟧` is a restriction, `dom(f) = dom(M(d_s)) ∩ ⟦σ⟧ ⊆ dom(M(d_s))` — every position in dom(f) is a genuine V-position of d_s and so inherits the structural axioms S2, S8-fin, S8a, and S8-depth (ASN-0036) that M11 and M12 invoke. Concretely: f is functional, as a restriction of the function M(d_s) (S2); dom(f) is finite, as a subset of the finite set dom(M(d_s)) (S8-fin); and by C0a every position in dom(f) has first component u₁, so dom(f) ⊆ V_{u₁}(d_s), whence S8-depth gives all of dom(f) a single common depth m, with m ≥ 2 as established at the ContentReference definition above (precondition (i) plus S8a and S8-depth).

*Extension of M11/M12.* M11 (CanonicalExistence) constructs a maximally merged decomposition by iterating: while any two blocks satisfy the merge condition (M7), merge them. The initial singleton-block decomposition — one block (v, f(v), 1) per v ∈ dom(f) — satisfies B1, B2, and B3: B1 (coverage) holds because every v ∈ dom(f) has its own singleton block; B2 (disjointness) holds because singleton V-extents are pairwise disjoint; B3 (consistency) holds directly from S2 (f is a function, so each singleton block's I-address is uniquely determined). Termination follows from S8-fin since the block count is at most |dom(f)|. Each merge step preserves all three conditions by M7f (MergeFrame). Although M7f is stated for decompositions of `M(d)`, its verification depends only on B1, B2, B3 and the definitions of `V(β)` and `⊞` — never on `M(d)` being the arrangement of a specific document. The proof carries over verbatim to any finite partial function `f : T ⇀ T` for which B1–B3 are interpreted with `f` in place of `M(d)`. M12 (CanonicalUniqueness) identifies the maximally merged decomposition with the set of maximal runs of f. Because dom(f) ⊆ dom(M(d_s)), every position M12 reasons about is a genuine V-position of d_s, so the structural appeals in M12 and its M-int lemma — S8a (`#x ≥ 2`, zero-free positive components), S8-depth (common subspace depth), and the OrdShiftHom depth preconditions invoked in M12b — hold directly on dom(f), exactly as they do on a full arrangement. Both proofs therefore run as written with f in place of M(d). ∎

The decomposition yields ⟨β₁, ..., βₖ⟩ ordered by V-start. The *I-address sequence* is:

`resolve(d_s, σ) = ⟨(a₁, n₁), ..., (aₖ, nₖ)⟩`

where βⱼ = (vⱼ, aⱼ, nⱼ). The V-coordinates are discarded; only I-starts and widths are carried forward.

**C1b (ResolutionSequenceOrder).** The runs in `resolve(d_s, σ) = ⟨(a₁, n₁), ..., (aₖ, nₖ)⟩` are listed in strictly increasing order of the V-start of their underlying blocks:

`(A i, j : 1 ≤ i < j ≤ k : vᵢ < vⱼ)`

This is an ordering of the *list positions* in the resolved sequence by the V-starts of the blocks they came from. It is not a claim that the I-address values `a₁, ..., aₖ` themselves are increasing — they need not be (e.g., the worked example below resolves to `⟨(a+1, 2), (b, 2)⟩` where the relation between `a+1` and `b` is unconstrained).

*Derivation.* By C1a, `M(d_s)|⟦σ⟧` admits a unique maximally merged decomposition (call it `{β₁, ..., βₖ}`). By B2 (Disjointness), the V-extents of the blocks are pairwise disjoint, so the V-starts `v₁, ..., vₖ` are pairwise distinct. T1 (LexicographicOrder, ASN-0034) is a total order on tumblers, so the V-starts admit a unique linear arrangement; the notation `⟨β₁, ..., βₖ⟩ ordered by V-start` fixes this linear arrangement, and `resolve(d_s, σ)` inherits it. The list-position index `i` therefore tracks the V-start order: `i < j` iff `vᵢ < vⱼ`. ∎

For a content reference sequence R = ⟨r₁, ..., rₚ⟩, the *composite resolution* concatenates:

`resolve(R) = resolve(r₁) ⌢ ... ⌢ resolve(rₚ)`

Each reference is resolved independently against its own source document's POOM. The *total width* of an I-address sequence ⟨(a₁, n₁), ..., (aₖ, nₖ)⟩ is:

`w(⟨(a₁, n₁), ..., (aₖ, nₖ)⟩) = (+ j : 1 ≤ j ≤ k : nⱼ)`

For a content reference sequence R, the total width is w(resolve(R)).

**C1 (ResolutionIntegrity).** Every resolved I-address is in dom(C):

`(A j : 1 ≤ j ≤ k : (A i : 0 ≤ i < nⱼ : aⱼ + i ∈ dom(C)))`

*Derivation.* Fix any run (aⱼ, nⱼ) in the resolution and any i with 0 ≤ i < nⱼ. The corresponding block βⱼ = (vⱼ, aⱼ, nⱼ) satisfies B3: M(d_s)(vⱼ + i) = aⱼ + i. Since vⱼ + i ∈ dom(M(d_s)), S3 (ReferentialIntegrity, ASN-0036) gives M(d_s)(vⱼ + i) ∈ dom(C), hence aⱼ + i ∈ dom(C). ∎

**C2 (ResolutionWidthPreservation).** For a well-formed content reference (d_s, σ) with σ = (u, δ(ℓₘ, m)), the total resolved width equals ℓₘ:

`w(resolve(d_s, σ)) = (+ j : 1 ≤ j ≤ k : nⱼ) = ℓₘ`

*Derivation.* By C0, ℓ = δ(ℓₘ, m), so reach(σ) = u ⊕ δ(ℓₘ, m) = [u₁, ..., u_{m−1}, uₘ + ℓₘ]. We compute |dom(f)| via the inclusion chain `dom(f) = D_m = E`, where `D_m := {v : u ≤ v < reach(σ) ∧ #v = m}` (the depth-m tumblers in ⟦σ⟧) and `E := {[u₁, ..., u_{m−1}, j] : uₘ ≤ j < uₘ + ℓₘ}`.

*Step 1 — D_m = E.* (⊆) For v ∈ D_m, C0a (PrefixConfinement) gives vⱼ = uⱼ for 1 ≤ j < m; combined with #v = m, this determines v as [u₁, ..., u_{m−1}, vₘ]. TumblerAdd applied to reach(σ) = u ⊕ δ(ℓₘ, m) gives reach(σ)_j = u_j for j < m (prefix-copy) and reach(σ)_m = u_m + ℓ_m (action-point clause). Comparing v against u and against reach(σ) at the divergence point m — using T1(i) with the prefix agreement just established — yields uₘ ≤ vₘ < uₘ + ℓₘ, so v ∈ E. (⊇) For t = [u₁, ..., u_{m−1}, j] ∈ E with uₘ ≤ j < uₘ + ℓₘ, prefix agreement at indices 1..m−1 with both u and reach(σ), combined with T1 at index m (case (i) when j ≠ u_m, or T3-equality when j = u_m), gives u ≤ t < reach(σ); with #t = m, t ∈ D_m.

*Step 2 — dom(f) = D_m.* (⊆) For v ∈ dom(f), v ∈ ⟦σ⟧ and v ∈ dom(M(d_s)). C0a applied to v gives v₁ = u₁, so v ∈ V_{u_1}(d_s); S8-depth (ASN-0036) applied to subspace u_1 of d_s gives #v = m. Hence v ∈ D_m. (⊇) Well-formedness of (d_s, σ) supplies the inclusion `{v : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d_s))` — which is precisely D_m ⊆ dom(M(d_s)). Each v ∈ D_m is in ⟦σ⟧ by definition of D_m, so v ∈ dom(M(d_s)) ∩ ⟦σ⟧ = dom(f).

*Step 3 — |E| = ℓₘ.* The map `j ↦ [u₁, ..., u_{m−1}, j]` from `{j ∈ ℕ : uₘ ≤ j < uₘ + ℓₘ}` (cardinality ℓₘ) onto E is a bijection: distinct j produce distinct m-th components, hence distinct tumblers by T3 (CanonicalRepresentation, ASN-0034); surjectivity onto E is immediate from E's definition.

Combining: |dom(f)| = |D_m| = |E| = ℓₘ via set equality at each step. By B1 (coverage) and B2 (disjointness), the V-extents of the blocks partition dom(f). By M0 (width coupling), |V(βⱼ)| = nⱼ for each block. Therefore (+ j : 1 ≤ j ≤ k : nⱼ) = |dom(f)| = ℓₘ. ∎

### A Worked Example (Content Reference Resolution)

We verify the definitions against a concrete scenario. Let document d have depth-2 V-positions in subspace 1 (m = 2) with canonical decomposition:

`B = {β₁ = ([1,1], a, 3),  β₂ = ([1,4], b, 2),  β₃ = ([1,6], c, 1)}`

where a, b, c are distinct I-addresses with `origin(a) ≠ origin(b) ≠ origin(c)` — three runs of content transcluded from three distinct source documents. The arrangement maps six V-positions: M(d)([1,1]) = a, M(d)([1,2]) = a+1, M(d)([1,3]) = a+2, M(d)([1,4]) = b, M(d)([1,5]) = b+1, M(d)([1,6]) = c.

**Content reference.** Take σ = ([1,2], δ(4, 2)) — start at V-position [1,2] with ordinal displacement [0,4]. Then reach(σ) = [1,2] ⊕ [0,4] = [1,6]. The span range is {v : [1,2] ≤ v < [1,6] ∧ #v = 2} = {[1,2], [1,3], [1,4], [1,5]}. Each is in dom(M(d)), so the reference is well-formed. The displacement is ordinal (action point 2 = m), consistent with C0.

**Restriction.** f = M(d)|⟦σ⟧ has domain {[1,2], [1,3], [1,4], [1,5]} with f([1,2]) = a+1, f([1,3]) = a+2, f([1,4]) = b, f([1,5]) = b+1.

**Decomposition (C1a).** Here f is a restriction of the arrangement M(d_s): it is functional (restriction of a function), dom(f) has 4 elements (finite), and all its V-positions are genuine V-positions of d_s at depth 2. Starting from singleton blocks {([1,2], a+1, 1), ([1,3], a+2, 1), ([1,4], b, 1), ([1,5], b+1, 1)}, we merge:

- [1,2] and [1,3]: V-adjacent ([1,3] = [1,2]+1) and I-adjacent (a+2 = (a+1)+1). Merge → ([1,2], a+1, 2).
- [1,4] and [1,5]: V-adjacent ([1,5] = [1,4]+1) and I-adjacent (b+1 = b+1). Merge → ([1,4], b, 2).

No further merges: ([1,2], a+1, 2) and ([1,4], b, 2) are V-adjacent ([1,4] = [1,2]+2) but not I-adjacent. M16 gives b ≠ (a+1)+2: ordinal increment preserves the document prefix, so origin((a+1)+2) = origin(a), while origin(b) ≠ origin(a) by construction. The decomposition is maximally merged.

**Resolution.** resolve(d, σ) = ⟨(a+1, 2), (b, 2)⟩, ordered by V-start.

**C1 verification.** For run (a+1, 2): B3 gives M(d)([1,2]) = a+1 and M(d)([1,3]) = a+2; S3 gives a+1 ∈ dom(C) and a+2 ∈ dom(C). For run (b, 2): B3 gives M(d)([1,4]) = b and M(d)([1,5]) = b+1; S3 gives b ∈ dom(C) and b+1 ∈ dom(C). ✓

Total width: 2 + 2 = 4 = ℓₘ, confirming C2.


## Properties Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| M0 | WidthCoupling: `\|V(β)\| = \|I(β)\| = n` for mapping block `β = (v, a, n)` | introduced |
| M1 | OrderPreservation: within a block, the `k`-th V-position maps to the `k`-th I-address; both orderings agree | introduced |
| OrdinalShiftBase | `t + k` denotes `shift(t, k)` for `k ≥ 1`, extended by `t + 0 = t` (identity) — convention in force throughout | introduced |
| M-aux | OrdinalIncrementAssociativity: `(v + c) + j = v + (c + j)` for `c, j ≥ 0` — from TS3 (ShiftComposition, ASN-0034) plus OrdinalShiftBase | introduced |
| M-int | TumblerIntervalCharacterization: for `x, y ∈ dom(M(d))` and `n ≥ 1`, `x ≤ y < x + n` entails `subspace(y) = subspace(x)`, `#y = #x = m`, prefix agreement to depth `m − 1`, and `y = x + k` for `k = (y)_m − (x)_m` with `0 ≤ k < n` | introduced |
| M2 | DecompositionExistence: under S8's preconditions (S8-fin, S2, S3, S8a, S8-depth — ASN-0036), every arrangement `M(d)` admits a block decomposition | introduced |
| M3 | RepresentationInvariance: equivalent decompositions determine the same arrangement function | introduced |
| M4 | SplitDefinition: split at interior `c` produces `β_L = (v, a, c)` and `β_R = (v+c, a+c, n−c)` | introduced |
| M5 | SplitPartition: `⟦β_L⟧ ∪ ⟦β_R⟧ = ⟦β⟧` and `⟦β_L⟧ ∩ ⟦β_R⟧ = ∅` | introduced |
| M6 | SplitPreservation: each piece is itself a mapping block and independently preserves width coupling, order, and I-fidelity (origin traceability is deferred to M16b, which depends on M16a) | introduced |
| M6f | SplitFrame: the arrangement `M(d)` is unchanged; only the decomposition changes | introduced |
| M7 | MergeCondition: merge requires V-adjacency (`v₂ = v₁ + n₁`) AND I-adjacency (`a₂ = a₁ + n₁`); result is `(v₁, a₁, n₁ + n₂)` | introduced |
| M7f | MergeFrame: the arrangement `M(d)` is unchanged; only the decomposition changes | introduced |
| M7-cov | NonOverlap: distinct blocks in any decomposition of `M(d)` cannot V-overlap — for `β₁, β₂ ∈ B` with `v₁ < v₂`, `v₂ ≥ v₁ + n₁` | introduced |
| M8 | MergeInformationLoss: the internal boundary is irrecoverably lost; merged block is indistinguishable from one never split | introduced |
| M9 | SplitMergeInverse: splitting then merging recovers the original block | introduced |
| M10 | MergeSplitInverse: merging then splitting at the boundary recovers both original blocks | introduced |
| M11 | CanonicalExistence: every arrangement admits a maximally merged decomposition | introduced |
| M12 | CanonicalUniqueness: the maximally merged decomposition is unique (equals the set of maximal runs of `M(d)`) | introduced |
| M12a | RunDisjointness: distinct maximal runs of `f = M(d)` have disjoint V-extents; via the partition corollary, the set of maximal runs partitions `dom(f)` and is uniquely determined by `f` | introduced |
| M12b | NoExtensionInMaximallyMerged: every block in a maximally merged decomposition satisfies conditions 2 and 3 of being a maximal run — it cannot be left-extended or right-extended in `f` | introduced |
| M13 | SharedContent: multiple V-positions may map to the same I-address within a single arrangement | introduced |
| M14 | IndependentOccurrences (corollary of M14a): blocks sharing I-start and width at distinct V-positions are independent and unmergeable | introduced |
| M14a | SharedIExtentUnmergeable: any two distinct blocks whose I-extents share at least one position cannot satisfy I-adjacency | introduced |
| M15 | MappingIndependence: (a) decompositions of `M(d_1)` and `M(d_2)` are definitionally independent (B-conditions quantify over only one arrangement); (b) frame condition — M6f and M7f, applied to a decomposition of `M(d_1)`, modify only that decomposition and leave every decomposition of `M(d_2)` unchanged | introduced |
| M16 | CrossOriginMergeImpossibility: blocks whose I-addresses originate from different documents cannot satisfy I-adjacency | introduced |
| M16a | OriginInvarianceUnderShift: for `a ∈ dom(C)` and `k ≥ 0` with `a + k ∈ dom(C)`, `origin(a + k) = origin(a)` — ordinal increment never crosses the document prefix | introduced |
| M16b | SplitOriginTraceability: when `β = (v, a, n)` belongs to a decomposition of `M(d)`, `origin(a + k) = origin(a)` for `0 ≤ k < n` (corollary of M16a discharged by B3 + S3); consequently `β_L` and `β_R` independently identify the home document of their content | introduced |
| B1 | Coverage: blocks in a decomposition partition `dom(M(d))` | introduced |
| B2 | Disjointness: no two blocks share a V-position | introduced |
| B3 | Consistency: each block correctly describes `M(d)` | introduced |
| ContentReference | (d_s, σ) with d_s ∈ D, V_{u₁}(d_s) ≠ ∅; σ level-uniform with #u = #ℓ = m; depth-m V-positions in span range ⊆ dom(M(d_s)) | introduced |
| C0 | OrdinalDisplacementNecessity: well-formed content references have ordinal displacements — action point of ℓ equals m | introduced |
| C0a | PrefixConfinement: every t ∈ ⟦σ⟧ satisfies tⱼ = uⱼ for all 1 ≤ j < m when m ≥ 2 (subspace confinement t₁ = u₁ is the j = 1 case) | introduced |
| ContentReferenceSequence | ordered list ⟨r₁, ..., rₚ⟩ with p ≥ 1 | introduced |
| resolve(d_s, σ) | Resolution: maximally merged I-address runs from `M(d_s)\|⟦σ⟧`, V-ordered | introduced |
| C1a | RestrictionDecomposition: M11/M12 hold for any restriction `M(d_s)\|X` of an arrangement whose induced domain lies in a single subspace; in particular `M(d_s)\|⟦σ⟧` admits a unique maximally merged decomposition | introduced |
| C1b | ResolutionSequenceOrder: list-position index i < j in resolve(d_s, σ) iff the underlying blocks satisfy vᵢ < vⱼ | introduced |
| C1 | ResolutionIntegrity: every resolved I-address is in dom(C) | introduced |
| C2 | ResolutionWidthPreservation: total resolved width equals ordinal displacement — w(resolve(d_s, σ)) = ℓₘ | introduced |

## Open Questions

When two V-adjacent blocks in the canonical decomposition fail the merge condition, what is the precise structure of the I-space discontinuity at their boundary — must it be a forward gap, or can it be an arbitrary jump to an unrelated I-region?

Is the set of equivalent decompositions of a given arrangement a lattice under the refinement ordering, with the canonical decomposition as the coarsest element?

What constraints govern the relationship between the total V-extent of an arrangement and the number of blocks in its canonical decomposition?

Does width coupling (M0) entail constraints on the tumbler depth relationship between V-starts and I-starts within a single block?

Must the resolution ordering across a multi-source content reference sequence preserve the sequence order, or may an implementation reorder source references provided the placed content lands at the correct V-positions?
