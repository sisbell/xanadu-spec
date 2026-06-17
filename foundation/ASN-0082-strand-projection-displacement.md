> **ASN-0082 · Strand Projection Displacement** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md), [ASN-0053 · Span Algebra](ASN-0053-span-algebra.md)  
> [Condensed statements →](ASN-0082-strand-projection-displacement.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0082: Strand Projection Displacement

*2026-04-09*

This ASN extends ASN-0036 (Arrangement and V-positions) with two complementary shift properties governing the arrangement transformations that underlie INSERT and DELETE. The *post-insertion shift* (I3 and its preservation lemmas) guarantees that ordinal shift applied uniformly to arrangement positions at or beyond an insertion point preserves mapping values while relocating V-positions forward by a fixed displacement. The *post-contraction shift* is the dual: it characterizes the inverse displacement that closes the gap left by removing a contiguous range of positions, preserving the I-address mappings of the right region while shifting their V-positions backward and re-establishing the foundation's contiguity invariants. The ordinal shift and its inverse — defined via OrdinalShift, OrdinalDisplacement, and TumblerSub (ASN-0034) — are fundamental operations on the tumbler line whose interaction with arrangement mappings determines how contiguous regions of mapped positions are repositioned without altering the content they reference. From these arrangement-layer properties we derive span-algebra corollaries (I3-S for insertion, D-S for contraction) connecting to ASN-0053 (Span Algebra): the displacement arithmetic underlying span endpoints (reach(σ) = start(σ) ⊕ width(σ)) commutes with uniform ordinal translation of a within-region span, so the span's width is preserved under both shift directions.


## Foundation Invariants

This ASN relies on two foundation invariants from ASN-0036 governing V-position structure:

**S8-depth** — *FixedDepthVPositions* (cited, ASN-0036). `(A d, v₁, v₂ : v₁ ∈ dom(Σ.M(d)) ∧ v₂ ∈ dom(Σ.M(d)) ∧ (v₁)₁ = (v₂)₁ : #v₁ = #v₂)`. All V-positions within a given subspace of a document share the same tumbler depth.

**S8a** — *VPositionWellFormedness* (cited, ASN-0036). `(A v ∈ dom(Σ.M(d)) :: zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))`. V-positions are zero-free, have depth at least 2, and have every component strictly positive. The componentwise positivity conjunct entails the specializations `v₁ ≥ 1` (positive subspace identifier) and `v > 0` (positive as a tumbler under lexicographic order), used pointwise in proofs below.

The contraction operation (below) additionally cites the following ASN-0036 properties:

- **S0** (ContentImmutability): for every state transition, `a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a)`.
- **S2** (ArrangementFunctionality): each V-position in dom(M(d)) has a uniquely determined I-address.
- **S3** (ReferentialIntegrity): `ran(M(d)) ⊆ dom(Σ.C)`.
- **S8-fin** (FiniteArrangement): for each document d, dom(M(d)) is finite.
- **D-CTG** (VContiguity, text subspace only): `(A d, u, q : u ∈ V_1(d) ∧ q ∈ V_1(d) ∧ u < q : (A v : subspace(v) = 1 ∧ #v = #u ∧ zeros(v) = 0 ∧ u < v < q : v ∈ V_1(d)))` (quoted verbatim from ASN-0036, including the `zeros(v) = 0` guard on the interior candidate). The link subspace V_2(d) is exempt — sparse with tombstones is permitted (ASN-0036, D-CTG frame note).
- **D-SEQ** (SequentialPositions, text subspace only): when V_1(d) is non-empty with common depth m ≥ 2, there exists n ≥ 1 such that V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n}.
- **D-MIN** (VMinimumPosition, text subspace only): when V_1(d) is non-empty, min(V_1(d)) = [1, 1, ..., 1] of length m. The link subspace is exempt — link positions need not begin at [2, 1, ..., 1].


## The Ordinal Shift

The *ordinal displacement* δ(n, m) is defined in the foundation: for n ≥ 1 and m ≥ 1, δ(n, m) = [0, 0, ..., 0, n] of length m — zero at positions 1 through m − 1, and n at position m, with action point m (OrdinalDisplacement, ASN-0034).

When the depth is determined by context (typically m = #p for insertion position p), we write δₙ.

The *ordinal shift* is defined in the foundation: for a V-position v of depth m and n ≥ 1, shift(v, n) = v ⊕ δ(n, m) (OrdinalShift, ASN-0034). By TumblerAdd: shift(v, n)ᵢ = vᵢ for i < m, and shift(v, n)ₘ = vₘ + n. The shift advances the ordinal within the V-position's subspace by exactly n, leaving all higher-level components unchanged.

We need two properties of this shift.

shift is order-preserving: for v₁, v₂ with #v₁ = #v₂ = m and v₁ < v₂, shift(v₁, n) < shift(v₂, n) (TS1, ASN-0034).

shift is injective: for v₁, v₂ with #v₁ = #v₂ = m, shift(v₁, n) = shift(v₂, n) implies v₁ = v₂ (TS2, ASN-0034).

Subspace preservation and S8a preservation are exactly OrdShiftHom (OrdinalShiftPreservation, ASN-0036): for a V-position v with #v = m ≥ 2 and n ≥ 1, (a) subspace(shift(v, n)) = subspace(v), and (b) when v satisfies S8a, shift(v, n) satisfies S8a. Both clauses require m ≥ 2 — the m = 1 case shift([S], n) = [S + n] would change the subspace identifier — so we exclude it by requiring #p ≥ 2 as an operation precondition. By S8-depth (ASN-0036), all V-positions in subspace S share a uniform depth d; the depth-compatibility precondition on I3 requires d = #p when such V-positions exist, so m = d = #p ≥ 2 holds for every V-position in the shifted region, discharging OrdShiftHom's m ≥ 2 precondition. This also ensures that the comparison v ≥ p in I3's quantifier is between equal-length tumblers, giving it the clean "at or to the right of p" semantics without prefix-case ambiguity. Furthermore, #shift(v, n) = #δₙ = m = #v by the result-length identity of TumblerAdd (ASN-0034).


## Post-Insertion Shift

Let M(d) : T ⇀ T denote the arrangement function for document d — a partial map from V-positions (element-field tumblers in the Vstream) to I-addresses (element-field tumblers in the Istream).

**Scope.** This ASN characterizes the *shift sub-operation* of INSERT — the arrangement transformation that opens a gap of n positions at p by relocating existing content at or beyond p in subspace S = subspace(p) = p₁ (with S ≥ 1) forward by n ≥ 1 ordinal positions, producing M'(d) from M(d). It modifies M(d) only and leaves C unchanged (frame I3-C): shifting existing content does not by itself add or modify any content-store entries.

**I3** — *PostInsertionShift* (POSTCONDITION, introduced). Content at or beyond p shifts forward by n ordinal positions.

*Preconditions:* d is a document; M(d) : T ⇀ T is its arrangement; p ∈ T with #p ≥ 2 and subspace(p) = S ≥ 1; depth-compatible: if {v ∈ dom(M(d)) : subspace(v) = S} ≠ ∅ then #p = #v for any such v (unique depth by S8-depth, ASN-0036); n ≥ 1; M'(d) is the post-insertion arrangement.

*Postconditions:*

`(A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v ≥ p : shift(v, n) ∈ dom(M'(d)) ∧ M'(d)(shift(v, n)) = M(d)(v))`

- I3-V (vacating): `(A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v ≥ p ∧ v ∉ {shift(u, n) : u ∈ dom(M(d)) ∧ subspace(u) = S ∧ u ≥ p} : v ∉ dom(M'(d)))`

*Frame:*

- I3-L (left region): `(A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v < p : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))`
- I3-X (cross-subspace): `(A v : v ∈ dom(M(d)) ∧ subspace(v) ≠ S : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))`
- I3-D (cross-document): `(A d' ≠ d : M'(d') = M(d'))`
- I3-C (content store): `dom(C') = dom(C) ∧ (A a ∈ dom(C) : C'(a) = C(a))` — S0 (ContentImmutability, ASN-0036) guarantees existing content is preserved (`dom(C) ⊆ dom(C')` with values unchanged); the shift stores no new content, so the reverse inclusion holds and dom(C') = dom(C)

*Domain closure:*

- I3-CS (subspace S): `(A v : v ∈ dom(M'(d)) ∧ subspace(v) = S : (v < p ∧ v ∈ dom(M(d))) ∨ (E u : u ∈ dom(M(d)) ∧ subspace(u) = S ∧ u ≥ p : v = shift(u, n)))`
- I3-CX (cross-subspace): `(A v : v ∈ dom(M'(d)) ∧ subspace(v) ≠ S : v ∈ dom(M(d)))`

**Consistency.** Well-definedness of M'(d) rests on two facts about the shifted region within subspace S. *Injectivity*: TS2 (ASN-0034) guarantees distinct v's produce distinct shift(v, n)'s, so the assignment I3 is single-valued. *Strict advance past p*: for v ≥ p in subspace S, shift(v, n) > v ≥ p by TS4 (ASN-0034), so no shifted output coincides with a left-region position (u < p). The remaining clauses are disjoint by subspace partition (left and shifted regions lie in subspace S; I3-X in subspaces ≠ S) and by document partition (I3-D operates on d' ≠ d), and I3-V's vacated positions are exactly those I3-CS excludes. Hence M'(d) and C' are well-defined.

**Structural preservation.**

**I3-VD** — *PostInsertionDepthUniformity* (LEMMA, derived). S8-depth holds for the post-state M'(d) across all subspaces. For subspace S: `(A v₁, v₂ ∈ dom(M'(d)) : subspace(v₁) = subspace(v₂) = S ⟹ #v₁ = #v₂ = m)`. By I3-CS, every v ∈ dom(M'(d)) with subspace(v) = S falls into exactly one of two regions. *Left region* (I3-L): v ∈ dom(M(d)) with subspace(v) = S and v < p; these have depth m by S8-depth on M(d). *Shifted region* (I3): shift(v, n) for v ∈ dom(M(d)) with subspace(v) = S and v ≥ p; #shift(v, n) = #δₙ = m by the result-length identity of TumblerAdd, and #v = m by S8-depth on M(d). Both regions yield depth m. For any subspace S' ≠ S: by I3-X (every pre-state position with subspace S' is in dom(M'(d))) and I3-CX (every post-state position with subspace S' is in dom(M(d))), the positions in dom(M'(d)) with subspace S' are exactly the positions in dom(M(d)) with subspace S', on which S8-depth holds by hypothesis. ∎

**I3-VP** — *PostInsertionWellFormedness* (LEMMA, derived). `(A v ∈ dom(M'(d)) : zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))`. By I3-CS and I3-CX, every v ∈ dom(M'(d)) falls into exactly one of three regions. *Left region* (I3-L): v ∈ dom(M(d)) with subspace(v) = S and v < p; S8a on M(d) gives v's well-formedness directly. *Shifted region* (I3): shift(v, n) for v ∈ dom(M(d)) with subspace(v) = S and v ≥ p; since v ∈ dom(M(d)) satisfies S8a with #v = m ≥ 2 and n ≥ 1, OrdShiftHom (b) (OrdinalShiftPreservation, ASN-0036) gives directly that shift(v, n) satisfies S8a. *Cross-subspace region* (I3-X): v ∈ dom(M(d)) with subspace(v) ≠ S; S8a on M(d) gives v's well-formedness directly. ∎

**I3-S3** — *PostInsertionReferentialIntegrity* (LEMMA, derived). `(A v : v ∈ dom(M'(d)) : M'(d)(v) ∈ dom(C'))`. By I3-C, dom(C') = dom(C). Every v ∈ dom(M'(d)) has M'(d)(v) equal to some M(d)(u) for u ∈ dom(M(d)): shifted positions have M'(d)(shift(u, n)) = M(d)(u) by I3; left-region and cross-subspace positions have M'(d)(v) = M(d)(v) by I3-L and I3-X. By S3 (ReferentialIntegrity, ASN-0036) on the pre-state, M(d)(u) ∈ dom(C) = dom(C'). ∎

**I3-S2** — *PostInsertionFunctionality* (LEMMA, derived). `M'(d)` is a function — S2 (ArrangementFunctionality, ASN-0036) holds for the post-state. The consistency check above establishes pairwise disjointness of the three assignment regions (shifted, left, cross-subspace); since each region assigns exactly one value per position, no position in dom(M'(d)) receives two values. ∎

**I3-fin** — *PostInsertionFiniteness* (LEMMA, derived). `dom(M'(d))` is finite — S8-fin (FiniteArrangement, ASN-0036) holds for the post-state. By I3-CS and I3-CX, every position in dom(M'(d)) either belongs to dom(M(d)) directly (left-region or cross-subspace) or is shift(v, n) for some v ∈ dom(M(d)) with subspace(v) = S and v ≥ p. The shifted-image set is at most as large as the source set by injectivity (TS2, ASN-0034). Both contributing sets are subsets or injective images of dom(M(d)), which is finite by S8-fin on the pre-state; their union is therefore finite. ∎

**I3-S7** — *PostInsertionAllocationInvariants* (LEMMA, derived). The dom(C)- and document-set-scoped invariants S7a (DocumentScopedAllocation), S7b (ElementLevelIAddresses), S7d (DocumentAllocationDiscipline), and the derived theorem S7 (StructuralAttribution) carry trivially: I3-C fixes dom(C) and its per-address values, I3-D fixes the document set, and these invariants are functions solely of those unchanged sets. ∎

**Arrangement invariants not preserved.** The shift preserves typing invariants (S8-depth, S8a, S3) but interacts with the contiguity invariants of ASN-0036 in a way that depends on the target subspace. The foundation scopes D-CTG (VContiguity), D-MIN (VMinimumPosition), and D-SEQ (SequentialPositions) to the text subspace V_1(d); the link subspace V_2(d) is explicitly exempt (ASN-0036, D-CTG frame note, D-MIN, D-SEQ).

*Case S = 1 (text subspace).* The gap created by the shift — n vacated positions between the left region and the shifted region — violates D-CTG: the post-state V_1(d) is not contiguous, as the worked example confirms ({[1,1], [1,2], [1,5], [1,6], [1,7]} has a gap between [1,2] and [1,5]). D-SEQ is likewise violated, since V_1(d) is no longer {[1, k] : 1 ≤ k ≤ n} for any n. When p = min(V_1(d)), the shift vacates the minimum position, additionally violating D-MIN.

*Case S ≠ 1 (non-text subspace; in particular S = 2, link).* The foundation does not impose D-CTG, D-MIN, or D-SEQ on V_S(d), so the shift creates no foundation-level violation: a post-state V_2(d) with a tombstone gap at the vacated positions is well-formed under ASN-0036's frame notes.

**Gap region.** I3-CS excludes the positions in the gap [p, shift(p, n)) from dom(M'(d)), since they are neither left-region positions nor shifted images: p is not < p (so I3-L excludes it), and no shifted image lands in the gap — two cases establish this: (1) when v = p, shift(p, n) equals the exclusive upper bound of [p, shift(p, n)) and so is not in the gap; (2) when v > p with #v = #p = m, TS1 (ShiftOrderPreservation, ASN-0034) gives shift(v, n) > shift(p, n), placing the image strictly past the gap's upper bound. These n gap positions are exactly the region I3-CS excludes from dom(M'(d)).


### Worked Example

Consider document d with five characters at V-positions [1, 1] through [1, 5], mapped to contiguous I-addresses b, b + 1, ..., b + 4.

Insert two characters at p = [1, 3]. Parameters: n = 2, S = 1, m = 2, δ₂ = [0, 2].

The left-region frame (I3-L) preserves [1, 1] and [1, 2] with unchanged I-addresses. I3 shifts: shift([1, 3], 2) = [1, 3] ⊕ [0, 2] = [1, 5], shift([1, 4], 2) = [1, 6], shift([1, 5], 2) = [1, 7]. Each shifted position preserves its I-address:

| V (before) | I (before) | V (after) | I (after) | Region |
|---|---|---|---|---|
| [1, 1] | b | [1, 1] | b | left (I3-L) |
| [1, 2] | b + 1 | [1, 2] | b + 1 | left (I3-L) |
| [1, 3] | b + 2 | [1, 5] | b + 2 | shifted (I3) |
| [1, 4] | b + 3 | [1, 6] | b + 3 | shifted (I3) |
| [1, 5] | b + 4 | [1, 7] | b + 4 | shifted (I3) |

Positions [1, 1] and [1, 2] are below p = [1, 3] and remain unchanged (I3-L). The three V-positions at or beyond p are each advanced by δ₂ = [0, 2]; their I-addresses are unchanged (I3).

**I3-V trace.** The shifted-image set is {shift(v, 2) : v ∈ dom(M(d)), subspace(v) = 1, v ≥ [1, 3]} = {[1, 5], [1, 6], [1, 7]}. I3-V applies to each original position at or beyond p that is *not* in this set:

- [1, 3]: not in {[1, 5], [1, 6], [1, 7]} → I3-V vacates: [1, 3] ∉ dom(M'(d)).
- [1, 4]: not in {[1, 5], [1, 6], [1, 7]} → I3-V vacates: [1, 4] ∉ dom(M'(d)).
- [1, 5]: *is* in the shifted-image set — [1, 5] = shift([1, 3], 2). I3-V's exclusion condition prevents vacating. I3 reassigns: M'(d)([1, 5]) = M(d)([1, 3]) = b + 2. The original value M(d)([1, 5]) = b + 4 is superseded — [1, 5] is retained at its shifted value, not its original one.

Positions [1, 3] and [1, 4] are the gap positions in [p, shift(p, n)) = [[1, 3], [1, 5]). Position [1, 5] demonstrates the overlap case: it is both an original position at or beyond p and a shifted destination, so I3 governs its post-state value while I3-V does not apply. ∎

**Boundary: insert at start.** Set p = [1, 1]. No V-position v satisfies v < p (since [1, 1] is the smallest in subspace 1), so I3-L's quantifier ranges over ∅ and holds vacuously. I3 shifts all five positions: shift([1, 1], 2) = [1, 3], ..., shift([1, 5], 2) = [1, 7], each preserving its I-address. ∎

**Boundary: insert past end.** Set p = [1, 6]. No V-position v satisfies v ≥ p, so I3's quantifier ranges over ∅ and holds vacuously. I3-L preserves all five positions [1, 1] through [1, 5] with unchanged I-addresses. ∎

**Boundary: empty document.** When dom(M(d)) = ∅, both I3 and I3-L quantify over ∅ and hold vacuously. The postcondition is consistent: insertion into an empty document creates no conflicts. ∎

**Cross-subspace preservation: text insertion leaves link subspace untouched.** Consider document d with both text and link subspaces populated. The text subspace S = 1 has three contiguous positions; the link subspace S = 2 has two sparse positions (allowed by the foundation's frame note on D-CTG for V_2). All positions have depth 2.

M(d) = {[1, 1] → b, [1, 2] → b + 1, [1, 3] → b + 2,  [2, 5] → ℓ₁, [2, 9] → ℓ₂}

Insert two text positions at p = [1, 2]. Parameters: n = 2, S = subspace(p) = 1, m = #p = 2, δ₂ = [0, 2].

By the depth-compatibility precondition on I3 (`#p = #v` for any v in subspace S of dom(M(d))), the comparison `v ≥ p` is between equal-length tumblers within subspace S = 1. Since v < p for v = [1, 1] and v ≥ p for v ∈ {[1, 2], [1, 3]}, I3-L preserves [1, 1] and I3 shifts the other two text positions:

- shift([1, 2], 2) = [1, 4]
- shift([1, 3], 2) = [1, 5]

The link-subspace positions [2, 5] and [2, 9] have subspace 2 ≠ S, so they fall under I3-X, which preserves both their positions and their I-address mappings:

| V (before) | I (before) | V (after) | I (after) | Region |
|---|---|---|---|---|
| [1, 1] | b | [1, 1] | b | left (I3-L) |
| [1, 2] | b + 1 | [1, 4] | b + 1 | shifted (I3) |
| [1, 3] | b + 2 | [1, 5] | b + 2 | shifted (I3) |
| [2, 5] | ℓ₁ | [2, 5] | ℓ₁ | cross-subspace (I3-X) |
| [2, 9] | ℓ₂ | [2, 9] | ℓ₂ | cross-subspace (I3-X) |

**Verification:**

- *I3-L:* [1, 1] < p = [1, 2] (T1, divergence at component 2 with 1 < 2); M'(d)([1, 1]) = b = M(d)([1, 1]). ✓
- *I3:* M'(d)([1, 4]) = b + 1 = M(d)([1, 2]); M'(d)([1, 5]) = b + 2 = M(d)([1, 3]). ✓
- *I3-X:* For v ∈ {[2, 5], [2, 9]}, subspace(v) = 2 ≠ 1 = S; v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v) by I3-X. ✓
- *I3-V:* Positions [1, 2] and [1, 3] are in dom(M(d)), have subspace 1, and ≥ p; the shifted-image set is {[1, 4], [1, 5]}. Neither [1, 2] nor [1, 3] is in this set, so I3-V vacates both: [1, 2] ∉ dom(M'(d)) and [1, 3] ∉ dom(M'(d)). ✓
- *I3-CS:* dom(M'(d)) ∩ subspace 1 = {[1, 1], [1, 4], [1, 5]} — exactly the left-region position and the shifted images. No gap positions [1, 2] or [1, 3] are in dom(M'(d)). ✓
- *I3-CX:* dom(M'(d)) ∩ subspace 2 = {[2, 5], [2, 9]} = dom(M(d)) ∩ subspace 2. The sparse link subspace is preserved verbatim — the tombstone gap at [2, 6], [2, 7], [2, 8] remains. ✓
- *I3-C:* dom(C') = dom(C) and values unchanged. The shift sub-operation modifies only M(d); no I-addresses are allocated. ✓

The link-subspace positions, having subspace identifier 2 ≠ 1, lie outside the quantifier ranges of I3 and I3-V, so the sparse V_2(d) with its tombstone gap is unaffected by the text-subspace insertion. ∎

**Cross-subspace insertion into the link subspace: a shifted image lands in a former tombstone slot.** When S = 2 is the shifted-into region, a shifted image may land in a slot that was a tombstone (an absent V-position) in the pre-state; we verify S2 and S3 still hold. Consider document d with a sparse link subspace containing a tombstone gap at [2, 3]:

M(d) = {[2, 1] → ℓ₁, [2, 2] → ℓ₂, [2, 4] → ℓ₃}

Insert one link position at p = [2, 1]. Parameters: n = 1, S = subspace(p) = 2, m = #p = 2, δ₁ = [0, 1]. All subspace-2 positions satisfy v ≥ p (since [2, 1] is the smallest), so all three shift; there is no left region.

- shift([2, 1], 1) = [2, 2]
- shift([2, 2], 1) = [2, 3]  — lands in the former tombstone slot
- shift([2, 4], 1) = [2, 5]

| V (before) | I (before) | V (after) | I (after) | Region |
|---|---|---|---|---|
| [2, 1] | ℓ₁ | [2, 2] | ℓ₁ | shifted (I3) |
| [2, 2] | ℓ₂ | [2, 3] | ℓ₂ | shifted (I3) |
| [2, 4] | ℓ₃ | [2, 5] | ℓ₃ | shifted (I3) |

**Verification:**

- *I3:* M'(d)([2, 2]) = ℓ₁ = M(d)([2, 1]); M'(d)([2, 3]) = ℓ₂ = M(d)([2, 2]); M'(d)([2, 5]) = ℓ₃ = M(d)([2, 4]). The image [2, 3] occupies a slot absent from dom(M(d)). ✓
- *I3-V:* The shifted-image set is {[2, 2], [2, 3], [2, 5]}. Among the original positions {[2, 1], [2, 2], [2, 4]} at or beyond p: [2, 1] and [2, 4] are not in the image set, so I3-V vacates both; [2, 2] *is* in the image set ([2, 2] = shift([2, 1], 1)), so I3-V does not vacate it and I3 reassigns M'(d)([2, 2]) = ℓ₁. ✓
- *I3-CS:* dom(M'(d)) ∩ subspace 2 = {[2, 2], [2, 3], [2, 5]} — exactly the shifted images (no left region since p is the minimum). The former tombstone slot [2, 3] is now occupied; this is permitted because V_2(d) carries no D-CTG/D-MIN/D-SEQ obligation. ✓
- *S2 (functionality):* The three images are distinct (injectivity, TS2), so each post-state position receives one value. The slot [2, 3] was a tombstone — absent from dom(M(d)) — so filling it creates no double-assignment with any surviving mapping. ✓
- *S3 (referential integrity):* {ℓ₁, ℓ₂, ℓ₃} = ran(M'(d)) ⊆ ran(M(d)) ⊆ dom(C) = dom(C') (I3-C); the shift relocates V-positions only and stores no new content. ✓

A shifted image landing in a former tombstone slot is therefore well-formed: link sparsity, not gap structure, is the invariant, and the strict-advance and injectivity facts that underwrite consistency hold identically for S = 2. ∎


## Span Width Preservation

The point-level shift I3 lifts to a span-level property connecting this ASN to the span algebra framework of ASN-0053.

Consider a level-uniform span σ = (s, ℓ) with #s = #ℓ = m and actionPoint(ℓ) = m. We call a span *ordinal-level* when its width acts purely at the deepest component: actionPoint(ℓ) = m. Define the shifted span σ' = (shift(s, n), ℓ). We verify that σ' is a well-formed span (T12, ASN-0034): ℓ > 0 is inherited from σ, and actionPoint(ℓ) = m ≤ #shift(s, n) = m (by TumblerAdd's result-length identity: #shift(s, n) = #δₙ = m).

**I3-S** — *SpanShiftPreservation* (LEMMA, introduced). For a level-uniform span σ = (s, ℓ) with #s = #ℓ = m and actionPoint(ℓ) = m, and a shift amount n ≥ 1, the shifted span σ' = (shift(s, n), ℓ) satisfies:

(a) reach(σ') = shift(reach(σ), n)

(b) width(σ') = ℓ

*Derivation of (a).* Since actionPoint(ℓ) = m and Pos(ℓ), ℓ has all zeros before position m, so ℓ = [0, …, 0, ℓₘ] = δ(ℓₘ, m) (OrdinalDisplacement, ASN-0034). Hence for any tumbler t of length m, `t ⊕ ℓ = t ⊕ δ(ℓₘ, m) = shift(t, ℓₘ)` (OrdinalShift, ASN-0034); likewise shift(s, n) = s ⊕ δₙ. The two reach expressions therefore reduce, via TS3 (ShiftComposition, ASN-0034), to shifts of s:

- reach(σ') = shift(s, n) ⊕ ℓ = shift(shift(s, n), ℓₘ) = shift(s, n + ℓₘ);
- shift(reach(σ), n) = shift(s ⊕ ℓ, n) = shift(shift(s, ℓₘ), n) = shift(s, ℓₘ + n).

Both are shifts of s, differing only in the scalar shift amount: n + ℓₘ versus ℓₘ + n. These denote the same natural number by ℕ-addition commutativity (`n + ℓₘ = ℓₘ + n`, a standard property of the carrier set ℕ). With this identity, the two TS3 compositions coincide: reach(σ') = shift(s, n + ℓₘ) = shift(s, ℓₘ + n) = shift(reach(σ), n). ∎

*Derivation of (b).* The span σ' = (shift(s, n), ℓ) is level-uniform: #shift(s, n) = m = #ℓ by the result-length identity of TumblerAdd. Its width is by definition its second component ℓ; consistently, by WR (WidthRecovery, ASN-0053), width(σ') = reach(σ') ⊖ start(σ') = (shift(s, n) ⊕ ℓ) ⊖ shift(s, n) = ℓ. ∎

*Verification against worked example.* From the insertion example above (p = [1, 3], n = 2, m = 2), take the span σ = ([1, 3], [0, 3]) covering the three pre-insertion positions [1, 3] through [1, 5]. Then reach(σ) = [1, 3] ⊕ [0, 3] = [1, 6], and the shifted span is σ' = (shift([1, 3], 2), [0, 3]) = ([1, 5], [0, 3]). For (a): reach(σ') = [1, 5] ⊕ [0, 3] = [1, 8], and shift(reach(σ), 2) = shift([1, 6], 2) = [1, 6] ⊕ [0, 2] = [1, 8]. ✓ For (b): width(σ') = [0, 3] = ℓ. ✓

Both endpoints of a within-subspace span shift by the same displacement; the width — the displacement between them — is invariant.


## Ordinal Extraction

We frequently need to separate a V-position into its subspace identifier and its ordinal within that subspace.

**OrdinalExtraction** — *ord(v)* (definition, local). For a V-position v with `#v = m ≥ 2`, `ord(v) = [v₂, ..., vₘ]` — the tumbler of length m − 1 obtained by stripping the subspace identifier (component 1) and reindexing, so `ord(v)ⱼ = vⱼ₊₁` for `1 ≤ j ≤ m − 1`. Both length and componentwise values come from T0's projection. When v satisfies S8a (every component positive), every component of ord(v) is positive, since ord(v) drops only position 1.

**VPositionReconstruction** — *vpos(S, o)* (definition, local). For subspace identifier `S ≥ 1` and ordinal `o = [o₁, ..., oₖ]` with `#o ≥ 1`, `vpos(S, o) = [S, o₁, ..., oₖ]` — prepend S and reindex, so `vpos(S, o)₁ = S` and `vpos(S, o)ⱼ₊₁ = oⱼ`. These are inverses by construction (component identity, T3): `ord(vpos(S, o)) = o` and `vpos(subspace(v), ord(v)) = v`. *S8a-closure (local postcondition):* when `S ≥ 1` and o is componentwise positive, `vpos(S, o)` is zero-free with all components positive and depth `#o + 1 ≥ 2`, so it satisfies S8a.

**OrdinalDisplacementProjection** — *w_ord* (definition, local). For a displacement w with `w₁ = 0` and `#w = m ≥ 2`, `w_ord = [w₂, ..., wₘ]` — the tumbler of length m − 1 obtained by stripping the (zero) first component, with `w_ordⱼ = wⱼ₊₁`. When `Pos(w)` (TA-Pos, ASN-0034), the witness for positivity sits at some position `i ≥ 2` (since `w₁ = 0`), so `Pos(w_ord)`; and the first (leftmost) nonzero of w, at position `actionPoint(w) ≥ 2`, maps to position `actionPoint(w) − 1` of w_ord, giving `actionPoint(w_ord) = actionPoint(w) − 1`.

**Lemma — OrdinalOrderEquivalence** (LEMMA, derived). For V-positions v₁, v₂ with subspace(v₁) = subspace(v₂) = S and #v₁ = #v₂ = m ≥ 2:

`v₁ < v₂ ⟺ ord(v₁) < ord(v₂)`

*Derivation from T1.* The structure shared by v and ord is: (v₁)₁ = (v₂)₁ = S agrees at v's position 1, and ord(vᵢ)_j = (vᵢ)_{j+1} for 1 ≤ j ≤ m − 1 by the definition of ord — an index shift of +1 from ord-coordinates to v-coordinates. Both ordinals have length m − 1 since #v₁ = #v₂ = m.

For (⟹): if v₁ < v₂, T1 (ASN-0034) places the leftmost divergence at some v-position k. Position 1 agrees by hypothesis, so k ≥ 2, with (v₁)ₖ < (v₂)ₖ and (v₁)_j = (v₂)_j for 2 ≤ j < k. Translating through ord: ord(v₁) and ord(v₂) agree at ord-positions 1..k − 2 (carrying values (v₁)₂..(v₁)_{k − 1} = (v₂)₂..(v₂)_{k − 1}) and diverge at ord-position k − 1, where ord(v₁)_{k − 1} = (v₁)ₖ < (v₂)ₖ = ord(v₂)_{k − 1}. T1 on the length-(m − 1) ordinals delivers ord(v₁) < ord(v₂).

For (⟸): the argument is symmetric. If ord(v₁) < ord(v₂), T1 places the divergence at some ord-position j ≥ 1 with ord(v₁)_j < ord(v₂)_j, which corresponds to v-position j + 1 ≥ 2 with (v₁)_{j + 1} < (v₂)_{j + 1}. Position 1 of v already agrees, so this is the leftmost divergence in v, and T1 on v gives v₁ < v₂. ∎

**OrdAddHom** — *OrdinalAdditionHomomorphism* (LEMMA, introduced). For a V-position p with `#p = m ≥ 2` and a displacement w with `w₁ = 0`, `#w = m`, and `Pos(w)`:

- (a) `ord(p ⊕ w) = ord(p) ⊕ w_ord` — whole-tumbler addition commutes with ordinal extraction when the displacement has a zero first component.
- (b) `subspace(p ⊕ w) = subspace(p)` — the subspace identifier is preserved under any ordinal-zero-prefixed displacement.
- (c) `p ⊕ w = vpos(subspace(p), ord(p) ⊕ w_ord)` — the addition lifts cleanly through ord/vpos.

*Derivation from TumblerAdd.* Let `k = actionPoint(w)`. Since `w₁ = 0` and `Pos(w)`, the first (leftmost) nonzero of w sits at `k ≥ 2`, and `k ≤ #w = m = #p`, so `p ⊕ w` is well-defined (TA0, ASN-0034). By TumblerAdd's piecewise construction, `(p ⊕ w)ᵢ = pᵢ` for `i < k` (prefix copy), `(p ⊕ w)_k = p_k + w_k`, and `(p ⊕ w)ᵢ = wᵢ` for `i > k`; the result has length m (result-length identity). For (b): since `k ≥ 2 > 1`, position 1 lies in the prefix-copy region, so `(p ⊕ w)₁ = p₁`, i.e. `subspace(p ⊕ w) = subspace(p)`. For (a): stripping position 1 from both sides and reindexing by −1, ord(p ⊕ w) has at ord-position `j = i − 1` (for `2 ≤ i ≤ m`) the value `(p ⊕ w)ᵢ`. The pair (ord(p), w_ord) has lengths m − 1, with `ord(p)ⱼ = pⱼ₊₁`, `w_ordⱼ = wⱼ₊₁`, and `actionPoint(w_ord) = k − 1` (OrdinalDisplacementProjection). TumblerAdd applied to ord(p) ⊕ w_ord at action point `k − 1` gives prefix copy `ord(p)ⱼ` for `j < k − 1`, sum `ord(p)_{k−1} + w_ord_{k−1}` at `j = k − 1`, and tail `w_ordⱼ` for `j > k − 1` — exactly the index-shifted images of TumblerAdd's three regions on p ⊕ w. Componentwise agreement at every ord-position (T3) gives `ord(p ⊕ w) = ord(p) ⊕ w_ord`. For (c): by the ord/vpos inverse `vpos(subspace(p ⊕ w), ord(p ⊕ w)) = p ⊕ w`; substituting (b) and (a) yields `p ⊕ w = vpos(subspace(p), ord(p) ⊕ w_ord)`. ∎

**Lemma — OrdinalExceedsDisplacement** (LEMMA, introduced). Fix the contraction parameters: `#p = 2`, `Pos(w)`, `w₁ = 0`, `p ∈ V_1(d)`, and `r = p ⊕ w`. For any tumbler v with `subspace(v) = 1`, `#v = 2`, and `v ≥ r`:

- (i) `ord(r) ≥ w_ord` and `ord(r) > w_ord`;
- (ii) `ord(v) ≥ w_ord` and `ord(v) > w_ord`;
- (iii) `ord(v) ⊖ w_ord` is well-defined and `Pos`, equal to `ord(p)` when `v = r` and strictly greater than `ord(p)` (under T1) when `v > r`.

*Derivation.* By OrdAddHom (a), `ord(r) = ord(p) ⊕ w_ord`. TumblerAdd's postcondition `a ⊕ w ≥ w` (ASN-0034) gives directly `ord(r) = ord(p) ⊕ w_ord ≥ w_ord` — the weak half of (i). For the strict half: TA4 (PartialInverse, ASN-0034) gives `(ord(p) ⊕ w_ord) ⊖ w_ord = ord(p)`, its preconditions discharged at depth 1 — `Pos(w_ord)` (OrdinalDisplacementProjection), action point `k = actionPoint(w_ord) = 1 = #ord(p)`, `#w_ord = 1 = k`, and the zero-prefix quantifier `1 ≤ i < 1` vacuous. So `ord(r) ⊖ w_ord = ord(p)`. Since `p ∈ V_1(d)`, S8a gives `p₂ ≥ 1`, so `ord(p) = [p₂]` is `Pos` — a non-zero tumbler — whence `ord(r) ≠ w_ord` (else the difference would be the zero tumbler). With `ord(r) ≥ w_ord` and T1 trichotomy, `ord(r) > w_ord`, completing (i). For (ii): the hypothesis `#v = 2` together with `#r = #w = #p = 2` (result-length identity, TumblerAdd) gives `#v = #r`, so OrdinalOrderEquivalence applies and `v ≥ r` yields `ord(v) ≥ ord(r)`, and T1 transitivity with (i) gives `ord(v) ≥ w_ord` and `ord(v) > w_ord`. For (iii): TA2 (WellDefinedSubtraction, ASN-0034) applies since `ord(v) ≥ w_ord`, giving `ord(v) ⊖ w_ord ∈ T`. Positivity: when `v = r`, `ord(v) ⊖ w_ord = ord(p)`, which is `Pos`; when `v > r`, OrdinalOrderEquivalence (again licensed by `#v = #r = 2`) gives `ord(v) > ord(r)` with `#ord(v) = #ord(r) = 1`, so TA3-strict (ASN-0034) gives `ord(v) ⊖ w_ord > ord(r) ⊖ w_ord = ord(p)`, and `ord(p)` exceeds the zero tumbler (TA6, ASN-0034), so `ord(v) ⊖ w_ord` is `Pos`. ∎


## Post-Contraction Shift

**Scope.** Contraction is the V-arrangement transformation of DELETE: it removes the deleted range, slides the right region back, and re-establishes the foundation's contiguity invariants. It modifies M(d) only; the content store is exactly unchanged (recorded at D-I).

Write V_1(d) = {v ∈ dom(M(d)) : subspace(v) = 1} for the text-subspace V-positions of document d; all V-positions in a given subspace share the same tumbler depth (S8-depth).

A contraction takes a document d and a contraction span (p, w) within the text subspace (S = 1) specifying the contiguous range of V-positions to remove. Let r = p ⊕ w denote the right cut point — the exclusive upper bound of the contraction.

**Contraction formal contract.**

*Preconditions:*

- `S = 1` — contraction is defined only on the text subspace; the foundation's D-CTG, D-MIN, D-SEQ supply the contiguity preconditions only for V_1(d).
- `p ∈ V_1(d)` — p is a current V-position in the text subspace of document d.
- `Pos(w)` (TA-Pos, ASN-0034) — the contraction width is positive.
- `#w = #p` — the displacement has the same depth as p.
- `w₁ = 0` — the displacement preserves the subspace identifier under addition.
- `#p = 2` — V-positions have depth 2, restricting to single-component ordinals.
- Containment: with D-SEQ giving `V_1(d) = {[1, k] : 1 ≤ k ≤ N}` (ASN-0036, text subspace), the condition `p₂ + w₂ − 1 ≤ N` — the contraction span lies entirely within the current arrangement.

The contraction span (p, w) partitions V_1(d) into three disjoint, exhaustive regions.

**Definition — ThreeRegions.**

```
L = {v ∈ V_1(d) : v < p}            — left of contraction
X = {v ∈ V_1(d) : p ≤ v < r}        — the contracted interval
R = {v ∈ V_1(d) : v ≥ r}            — right of contraction
```

By trichotomy of the total order (T1, ASN-0034), every v ∈ V_1(d) falls in exactly one region. The post-state arrangement M'(d) is the arrangement after the contraction has been applied.

**D-SHIFT** — *RightShift* (POST, postcondition). Every position in the right region survives with its I-address mapping intact, but its V-position shifts left by w_ord. Define the shift function: for v ∈ R, let σ(v) = vpos(S, ord(v) ⊖ w_ord) — TumblerSub applied to the ordinal component, then reconstructed as a V-position. The set of shifted right-region positions is Q₃ = {σ(v) : v ∈ R}.

*Preconditions:* As stated in the contraction formal contract: p ∈ V_1(d), #p = 2, Pos(w), #w = #p, w₁ = 0, containment satisfied. r = p ⊕ w; R = {v ∈ V_1(d) : v ≥ r}; M'(d) is the post-contraction arrangement.

*Postconditions:*

`(A v : v ∈ R : σ(v) ∈ dom(M'(d)) ∧ M'(d)(σ(v)) = M(d)(v))`

The shift is well-defined and S8a-preserving. For any v ∈ R we have v ≥ r, so OrdinalExceedsDisplacement (ii) gives `ord(v) ≥ w_ord` — the precondition of TA2 (WellDefinedSubtraction, ASN-0034) — whence `ord(v) ⊖ w_ord ∈ T` and σ(v) = vpos(1, ord(v) ⊖ w_ord) = [1, vₘ − c] exists as a tumbler. By OrdinalExceedsDisplacement (iii) the ordinal `ord(v) ⊖ w_ord = [vₘ − c]` is `Pos` with `vₘ − c ≥ p₂ ≥ 1`, and the subspace identifier 1 is positive, so by vpos's S8a-closure postcondition σ(v) is zero-free, of depth 2, and componentwise positive — full S8a.

The contraction's effect on regions L and X, and on state outside subspace S and document d, must be stated explicitly.

**D-L** — *LeftPreservation* (FRAME, introduced). Positions in the left region are preserved unchanged:

`(A v : v ∈ L : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))`

**D-DOM** — *DomainCharacterization* (POST, introduced). The post-state arrangement within subspace S consists of exactly the preserved left region and the shifted right region:

`{v ∈ dom(M'(d)) : subspace(v) = S} = L ∪ Q₃`

The original X mappings are not preserved — any X address that reappears in Q₃ carries the shifted I-address from the corresponding R position, not its pre-contraction content.

**D-CS** — *CrossSubspaceFrame* (FRAME, introduced). Other subspaces are unchanged — their position sets are exactly the pre-state sets with the same mappings:

`(A S' ≠ S : {v ∈ dom(M'(d)) : subspace(v) = S'} = {v ∈ dom(M(d)) : subspace(v) = S'})`

`∧ (A v : v ∈ dom(M(d)) ∧ subspace(v) ≠ S : M'(d)(v) = M(d)(v))`

The first conjunct establishes domain equality per non-S subspace (no positions added or removed); the second establishes mapping equality (no values changed).

**D-CD** — *CrossDocumentFrame* (FRAME, introduced). Other documents are unchanged:

`(A d' ≠ d : M'(d') = M(d'))`

**D-I** — *ContentStoreFrame* (FRAME, introduced). The content store is unchanged:

`Σ'.C = Σ.C`

That is, `dom(Σ'.C) = dom(Σ.C)` and `(A a ∈ dom(Σ.C) : Σ'.C(a) = Σ.C(a))`. Contraction modifies only the arrangement M(d); no I-addresses are allocated or deallocated, and no content values change.

**Shift correctness.** We verify that the shift σ defined by D-SHIFT is well-behaved: order-preserving, injective, and gap-closing.

**D-BJ** — *ShiftBijectivity* (LEMMA, lemma). The map σ : R → Q₃ is an order-preserving injection. Since Q₃ is *defined* as the image {σ(v) : v ∈ R}, σ is surjective onto Q₃ by definition; the proof obligations are order-preservation and injectivity.

*Preconditions:* #p = 2; v₁, v₂ ∈ R with v₁ ≠ v₂ (for injectivity) or v₁ < v₂ (for order-preservation).

*Postconditions:*

- (a) Order-preservation: `v₁ < v₂ ⟹ σ(v₁) < σ(v₂)`
- (b) Injectivity: `v₁ ≠ v₂ ⟹ σ(v₁) ≠ σ(v₂)`

*Proof of (a).* All ordinals in R share the same depth (S8-depth), giving #ord(v₁) = #ord(v₂). For any v₁ < v₂ in R, we have ord(v₁) < ord(v₂) (by OrdinalOrderEquivalence — both share subspace S = 1 and depth m = 2).

For every v ∈ R we have v ≥ r, so OrdinalExceedsDisplacement (ii) gives `ord(v) ≥ w_ord`.

By TA3-strict (OrderPreservationSubtractionStrict, ASN-0034) — a < b ∧ a ≥ w ∧ b ≥ w ∧ #a = #b ⟹ a ⊖ w < b ⊖ w — with a = ord(v₁), b = ord(v₂), w = w_ord, and both `ord(v₁) ≥ w_ord` and `ord(v₂) ≥ w_ord` from OrdinalExceedsDisplacement (ii), we conclude ord(v₁) ⊖ w_ord < ord(v₂) ⊖ w_ord. Now σ(v₁) and σ(v₂) share subspace S = 1 and depth m = 2, and ord(σ(v₁)) = ord(v₁) ⊖ w_ord < ord(v₂) ⊖ w_ord = ord(σ(v₂)); by the reverse direction of OrdinalOrderEquivalence, σ(v₁) < σ(v₂). ∎

*Proof of (b).* For v₁ ≠ v₂ in R, trichotomy (T1) gives v₁ < v₂ or v₂ < v₁. In either case, part (a) yields σ(v₁) < σ(v₂) or σ(v₂) < σ(v₁), so σ(v₁) ≠ σ(v₂). ∎

**D-SEP** — *GapClosure* (LEMMA, lemma). The contraction width exactly bridges the ordinal distance between p and r, so shifting the right cut point back by the width recovers the ordinal of the left cut point. When R ≠ ∅, D-CTG ensures this algebraic identity has the semantic consequence that the shifted right region begins exactly where the left region ends.

*Preconditions:* #p = 2; r = p ⊕ w.

*Postconditions:*

- (a) Algebraic identity: `ord(r) ⊖ w_ord = ord(p)`.
- (b) When R ≠ ∅: r ∈ V_1(d), r = min(R), and ord(σ(r)) = ord(p), i.e., min({ord(u) : u ∈ Q₃}) = ord(p).

*Proof of (a).* By OrdinalExceedsDisplacement (iii) at `v = r`, `ord(r) ⊖ w_ord = ord(p)`. ✓

*Proof of (b).* Suppose R ≠ ∅, so there exists v ∈ V_1(d) with v ≥ r. The contraction operates under the precondition that D-SEQ holds on the pre-state, giving V_1(d) = {[1, k] : 1 ≤ k ≤ N}; hence v = [1, k_v] with 1 ≤ k_v ≤ N. Since r = p ⊕ w with p = [1, p₂], w₁ = 0, and w₂ = c ≥ 1, TumblerAdd gives r = [1, p₂ + c]. From v ≥ r, T1 at position 2 yields k_v ≥ p₂ + c, so p₂ + c ≤ k_v ≤ N. With the lower bound p₂ + c ≥ 2 ≥ 1 (from p₂ ≥ 1 by S8a on p and c ≥ 1), the ordinal p₂ + c lies in [1, N], so by D-SEQ the position r = [1, p₂ + c] ∈ V_1(d), hence r ∈ R.

Moreover r = min(R): r ≤ v for every v ∈ R by the defining condition v ≥ r of R. By D-BJ, σ is order-preserving, so σ(r) = min(Q₃). By part (a), ord(σ(r)) = ord(p). ∎

**D-DP** — *DensePartition* (LEMMA, lemma). The post-state arrangement in subspace S is exactly the union of the preserved left region and the shifted right region, with no overlap and no gap at the contraction boundary.

*Preconditions:* #p = 2; L, X, R as defined by ThreeRegions; D-L, D-DOM, D-SHIFT, D-SEP, and D-CTG hold.

*Postconditions:*

- (a) No overlap: `L ∩ Q₃ = ∅`
- (b) Boundary adjacency: when R ≠ ∅, `min({ord(u) : u ∈ Q₃}) = ord(p)`, and `(A v ∈ L : ord(v) < ord(p))`

*Proof.* *Case R = ∅:* Q₃ = ∅ by definition, so L ∩ Q₃ = ∅ trivially. *Case R ≠ ∅:* Every v ∈ L satisfies v < p, hence ord(v) < ord(p) (by OrdinalOrderEquivalence — both share subspace S and depth m). By D-SEP(b), when R ≠ ∅ the minimum ordinal in Q₃ is ord(p), and by D-BJ every other element of Q₃ has ordinal strictly greater than ord(p). So every element of L has ordinal strictly less than ord(p) and every element of Q₃ has ordinal ≥ ord(p), giving L ∩ Q₃ = ∅.

The boundary is tight. At depth 2 with contiguous allocation (D-CTG), L contains exactly the positions with ordinals below ord(p), and Q₃ begins at ordinal ord(p) (D-SEP). The ordinals ord(p) − 1 and ord(p) are consecutive natural numbers; no ordinal falls between them. D-DOM confirms that the post-state domain in subspace S is exactly L ∪ Q₃. ∎

**Invariant preservation.** The postconditions and frame conditions above characterize the post-state arrangement. We now verify that the post-state satisfies each system invariant established in ASN-0036.

*Off-subspace and off-document dispatch (convention for all post-lemmas below).* Each post-lemma below establishes its claim for the active text subspace S = 1; we omit the off-subspace and off-document discharge from each proof. The off-subspace obligation (subspaces S' ≠ 1) is discharged uniformly by D-CS, which fixes each non-text subspace's position set and mappings to the pre-state and so carries any pre-state invariant verbatim; the foundation imposes no D-CTG/D-MIN/D-SEQ obligation on non-text subspaces in any case. The off-document obligation (documents d' ≠ d) is discharged uniformly by D-CD. Each lemma below carries only its subspace-1 argument.

**S8-depth-post** — *FixedDepthPreservation* (LEMMA, introduced). The post-state satisfies S8-depth: all V-positions within subspace S share the same depth.

*Proof.* Positions in L retain depth 2 (unchanged by D-L). Positions in Q₃ have depth 2: for v ∈ R, σ(v) = vpos(S, [vₘ − c]) = [S, vₘ − c], which has depth 2. ∎

**S8a-post** — *WellFormednessPreservation* (LEMMA, introduced). The post-state satisfies S8a: all V-positions are zero-free, of depth at least 2, and componentwise positive.

*Proof.* Positions in L satisfy S8a by the pre-state invariant and D-L (unchanged). Positions in Q₃: for each v ∈ R, σ(v) = vpos(1, ord(v) ⊖ w_ord) is well-defined and satisfies S8a, as established at D-SHIFT. ∎

**D-CTG-post** — *VContiguityPreservation* (LEMMA, introduced). At S = 1: the post-state V_1(d) is contiguous.

*Proof.* By D-SEQ (ASN-0036, text subspace), the pre-state V_1(d) = {[1, k] : 1 ≤ k ≤ N}. From the definition of L and D-SEQ on the pre-state,

`L = {[1, k] : 1 ≤ k < p₂}`.

By D-BJ, Q₃ is the order-preserving image of R under σ; applying σ([1, k]) = [1, k − c] to D-SEQ's R = {[1, k] : p₂ + c ≤ k ≤ N} gives

`Q₃ = {[1, k − c] : p₂ + c ≤ k ≤ N} = {[1, k] : p₂ ≤ k ≤ N − c}`.

The two index ranges are disjoint (k < p₂ in L, k ≥ p₂ in Q₃), and the natural numbers p₂ − 1 (the maximum of L's index range) and p₂ (the minimum of Q₃'s) are consecutive — no integer lies strictly between them — so

`L ∪ Q₃ = {[1, k] : 1 ≤ k < p₂} ∪ {[1, k] : p₂ ≤ k ≤ N − c} = {[1, k] : 1 ≤ k ≤ N − c}`.

The closed form covers all boundary configurations. When L = ∅: D-MIN (ASN-0036) gives min V_1(d) = [1, 1], so L = ∅ forces p = min V_1(d) = [1, 1] and p₂ = 1, vacating the L range and reducing the union to Q₃ = {[1, k] : 1 ≤ k ≤ N − c}. When R = ∅: no k ∈ [1, N] satisfies k ≥ p₂ + c, i.e., N < p₂ + c; combined with the containment precondition p₂ + c − 1 ≤ N this forces N = p₂ + c − 1, so N − c = p₂ − 1 and the union reduces to L = {[1, k] : 1 ≤ k ≤ p₂ − 1} = {[1, k] : 1 ≤ k ≤ N − c}. When both are empty, N − c = 0 and the set is empty.

We verify D-CTG's quantifier directly against V_1(d') = L ∪ Q₃ = {[1, k] : 1 ≤ k ≤ N − c}. Take u, q ∈ V_1(d') with u < q (both of depth 2 by S8-depth-post and subspace identifier 1 by S8a-post applied to V_1(d')), and any V-position v with subspace(v) = 1, #v = 2, and u < v < q. Write u = [1, kᵤ], q = [1, k_q], v = [1, k_v]. From u < v < q at depth 2 with shared subspace identifier 1, T1 reduces to the natural-number chain kᵤ < k_v < k_q. Membership of u and q in {[1, k] : 1 ≤ k ≤ N − c} gives 1 ≤ kᵤ and k_q ≤ N − c, so transitivity yields 1 ≤ k_v ≤ N − c, hence v = [1, k_v] ∈ V_1(d'). The interior point lies in V_1(d'), satisfying D-CTG. ∎

**D-MIN-post** — *VMinimumPreservation* (LEMMA, introduced). At S = 1: when the post-state V_1(d) is non-empty, min(V_1(d)) = [1, 1]. When the post-state V_1(d) is empty, D-MIN holds vacuously.

*Proof.* Three cases for S = 1. When L ≠ ∅: the pre-state minimum is min(V_1(d)) = [1, 1] (D-MIN, ASN-0036, text subspace). L ≠ ∅ supplies some v ∈ V_1(d) with v < p, so min(V_1(d)) ≤ v < p by min's lower-bound property and T1's transitivity; hence min(V_1(d)) ∈ L by L's definition L = {v ∈ V_1(d) : v < p}. D-L preserves min(V_1(d)) verbatim into V_1(d'), and since [1, 1] is the T1-minimum of V_1(d) ⊇ L it remains the T1-minimum of L: min(L) = [1, 1]. The closure step min(L ∪ Q₃) = min(L) is supplied by D-DP(b): when R ≠ ∅, D-DP(b) gives `(A v ∈ L : ord(v) < ord(p))` together with `min({ord(u) : u ∈ Q₃}) = ord(p)` (hence `ord(u) ≥ ord(p)` for every u ∈ Q₃), so for every v ∈ L and u ∈ Q₃ we have ord(v) < ord(p) ≤ ord(u), i.e., ord(v) < ord(u); by OrdinalOrderEquivalence (subspace 1 shared throughout V_1(d') by D-DOM, depth 2 shared by S8-depth-post) v < u, making every L element a strict T1-lower-bound for every Q₃ element and forcing min(L ∪ Q₃) = min(L). When R = ∅, Q₃ = ∅ and min(L ∪ Q₃) = min(L) trivially. In both subcases min(L ∪ Q₃) = min(L) = [1, 1]. When L = ∅ and R ≠ ∅: p = min(V_1(d)) = [1, 1] by D-MIN, so ord(p) = [1]. By D-SEP(b), min Q₃ has ordinal ord(p) = [1], giving min Q₃ = [1, 1]. When L = ∅ and R = ∅: V_1(d') = L ∪ Q₃ = ∅, so D-MIN holds vacuously. ∎

**D-SEQ-post** — *SequentialPositionsPreservation* (LEMMA, introduced). At S = 1: when the post-state V_1(d) is non-empty, V_1(d) = {[1, k] : 1 ≤ k ≤ N − c}.

*Proof.* The foundation's D-SEQ derivation (ASN-0036) takes four preconditions on V_1(d): contiguity (D-CTG), minimum at [1, 1] (D-MIN), uniform depth (S8-depth), and componentwise positivity (S8a). We verify each for the post-state, then derive n locally rather than re-invoking the foundation's text-only proof on what is now the post-state.

1. *Contiguity.* By D-CTG-post, V_1(d') = L ∪ Q₃ is contiguous.
2. *Minimum.* By D-MIN-post, when non-empty, min(V_1(d')) = [1, 1].
3. *Uniform depth.* By S8-depth-post, all V-positions in V_1(d') have depth 2.
4. *Componentwise positivity (S8a).* By S8a-post, every position in V_1(d') is zero-free, of depth ≥ 2, and componentwise positive — in particular, the position-2 component of every element is ≥ 1.

Replaying the derivation locally at depth m = 2: L ∪ Q₃ is a contiguous set of depth-2 positions in subspace 1 with minimum [1, 1] and all components positive. Every position has the form [1, k] for some k ≥ 1 (at m = 2 the shared-prefix component range is empty, so componentwise positivity reduces to k ≥ 1). The k-values include 1 (from D-MIN-post). The k-values form a contiguous range (from D-CTG-post on L ∪ Q₃). The set is finite: L ⊆ V_1(d) and Q₃ = σ(R) with R ⊆ V_1(d), so |L ∪ Q₃| ≤ |L| + |Q₃| ≤ |V_1(d)| + |V_1(d)|, which is finite by S8-fin (ASN-0036) on the pre-state. Setting n = max(k-values), we get V_1(d') = {[1, k] : 1 ≤ k ≤ n}. It remains to identify n. The cardinality |L ∪ Q₃| chains through four cited facts:

- |L ∪ Q₃| = |L| + |Q₃| (D-DP(a) disjointness L ∩ Q₃ = ∅)
- = |L| + |R| (D-BJ's bijection σ : R → Q₃)
- = N − |X| (trichotomy partition |V_1(d)| = |L| + |X| + |R| = N on the pre-state's contiguous range)
- = N − c (|X| = c directly: by pre-state D-SEQ, V_1(d) = {[1, k] : 1 ≤ k ≤ N}, and the defining condition p ≤ v < r at depth 2 reduces — via OrdinalOrderEquivalence and ord(r) = [p₂ + c] — to p₂ ≤ k < p₂ + c; the containment precondition p₂ + w₂ − 1 ≤ N places all c indices p₂, …, p₂ + c − 1 within [1, N], so X = {[1, k] : p₂ ≤ k < p₂ + c} has exactly c elements)

Hence n = N − c, and V_1(d') = {[1, k] : 1 ≤ k ≤ N − c}. When V_1(d') is empty (N − c = 0, i.e., the entire text subspace was contracted), D-SEQ holds vacuously. ∎

**S8-fin-post** — *FiniteArrangementPreservation* (LEMMA, introduced). The post-state satisfies S8-fin: `dom(M'(d))` is finite.

*Proof.* By D-DOM, the subspace-1 positions in dom(M'(d)) are L ∪ Q₃. L ⊆ V_1(d) and Q₃ = σ(R) with R ⊆ V_1(d), so |L ∪ Q₃| ≤ |V_1(d)|, which is finite by S8-fin on the pre-state. The off-subspace domains of d are finite by S8-fin via D-CS (per the dispatch convention). ∎

**S2-post** — *ArrangementFunctionality* (LEMMA, introduced). The post-state M'(d) is a function.

*Proof.* By D-DOM, dom(M'(d)) within subspace S is L ∪ Q₃. By D-DP(a), L ∩ Q₃ = ∅. For v ∈ L, M'(d)(v) is uniquely determined by D-L. For v ∈ Q₃, v = σ(u) for a unique u ∈ R (D-BJ, injectivity), and M'(d)(v) = M(d)(u) is uniquely determined by D-SHIFT and S2 on the pre-state. Since the two regions are disjoint and each assigns a unique value, M'(d) is a function within subspace S. ∎

**S3-post** — *ReferentialIntegrity* (LEMMA, introduced). The post-state satisfies `ran(M'(d)) ⊆ dom(Σ'.C)`.

*Proof.* Every I-address in ran(M'(d)) was an I-address in ran(M(d)): positions in L map to the same I-addresses as before (D-L), and positions in Q₃ map to I-addresses from R (D-SHIFT). By S3 on the pre-state, ran(M(d)) ⊆ dom(Σ.C). By D-I (content store frame), dom(Σ.C) ⊆ dom(Σ'.C). Hence the subspace-S contribution to ran(M'(d)) is contained in dom(Σ'.C); the off-subspace contributions are contained in dom(Σ'.C) by S3 on the pre-state via D-CS (per the dispatch convention). ∎

**S7-post** — *AllocationInvariantsPreservation* (LEMMA, introduced). As with I3-S7, the dom(C)- and document-set-scoped invariants S7a, S7b, S7d, and the derived theorem S7 carry trivially: D-I fixes Σ.C (so dom(Σ.C) and its values are unchanged), D-CD fixes the document set, and these invariants are functions solely of those unchanged sets. ∎

### Worked Example

We verify the postconditions against a concrete scenario. Consider document d with subspace S = 1 and five contiguous V-positions:

M(d) = {[1,1] → i₁,  [1,2] → i₂,  [1,3] → i₃,  [1,4] → i₄,  [1,5] → i₅}

Contract at p = [1,2] with w = [0,2], so c = 2 and r = p ⊕ w = [1,4].

**Three-region partition.** L = {[1,1]}, X = {[1,2], [1,3]}, R = {[1,4], [1,5]}.

**Shift computation.** w_ord = [2]. For each v ∈ R:

- σ([1,4]) = vpos(1, [4] ⊖ [2]) = vpos(1, [2]) = [1,2]
- σ([1,5]) = vpos(1, [5] ⊖ [2]) = vpos(1, [3]) = [1,3]

Q₃ = {[1,2], [1,3]}.

**Post-state.** M'(d) = {[1,1] → i₁,  [1,2] → i₄,  [1,3] → i₅}

**Verification:**

- *D-L:* M'(d)([1,1]) = i₁ = M(d)([1,1]). ✓
- *D-SHIFT:* M'(d)([1,2]) = i₄ = M(d)([1,4]); M'(d)([1,3]) = i₅ = M(d)([1,5]). ✓
- *D-DOM:* {v ∈ dom(M'(d)) : subspace(v) = 1} = {[1,1], [1,2], [1,3]} = L ∪ Q₃. ✓
- *D-BJ:* [1,4] < [1,5] and σ([1,4]) = [1,2] < [1,3] = σ([1,5]). ✓
- *D-SEP:* ord(r) ⊖ w_ord = [4] ⊖ [2] = [2] = ord(p). ✓
- *D-DP:* L ∩ Q₃ = ∅; min Q₃ ordinal = [2] = ord(p); all L ordinals < ord(p). ✓

We observe that addresses [1,2] and [1,3] appear in both X and Q₃ but with different I-address mappings: M(d)([1,2]) = i₂ whereas M'(d)([1,2]) = i₄. The addresses are reused by the shift — D-DOM characterizes this correctly.

**Boundary case: L = ∅.** Consider the same five-position arrangement but with contraction at the beginning: p = [1,1], w = [0,2], so c = 2 and r = p ⊕ w = [1,3].

**Three-region partition.** L = ∅, X = {[1,1], [1,2]}, R = {[1,3], [1,4], [1,5]}.

**Shift computation.** w_ord = [2]. For each v ∈ R:

- σ([1,3]) = vpos(1, [3] ⊖ [2]) = vpos(1, [1]) = [1,1]
- σ([1,4]) = vpos(1, [4] ⊖ [2]) = vpos(1, [2]) = [1,2]
- σ([1,5]) = vpos(1, [5] ⊖ [2]) = vpos(1, [3]) = [1,3]

Q₃ = {[1,1], [1,2], [1,3]}.

**Post-state.** M'(d) = {[1,1] → i₃,  [1,2] → i₄,  [1,3] → i₅}

**Verification:**

- *D-L:* L = ∅, vacuously satisfied. ✓
- *D-SHIFT:* M'(d)([1,1]) = i₃ = M(d)([1,3]); M'(d)([1,2]) = i₄ = M(d)([1,4]); M'(d)([1,3]) = i₅ = M(d)([1,5]). ✓
- *D-DOM:* {v ∈ dom(M'(d)) : subspace(v) = 1} = {[1,1], [1,2], [1,3]} = ∅ ∪ Q₃ = Q₃. ✓
- *D-BJ:* [1,3] < [1,4] < [1,5] and σ([1,3]) = [1,1] < [1,2] = σ([1,4]) < [1,3] = σ([1,5]). ✓
- *D-SEP(a):* ord([1,3]) ⊖ [2] = [3] ⊖ [2] = [1] = ord([1,1]) = ord(p). ✓
- *D-SEP(b):* min Q₃ = [1,1], ord([1,1]) = [1] = ord(p). ✓
- *D-DP:* L ∩ Q₃ = ∅ (L = ∅). ✓
- *D-MIN-post:* min Q₃ = [1,1] = [S, 1]. ✓
- *S2-post:* Three distinct V-positions, each assigned a unique I-address. ✓
- *S3-post:* {i₃, i₄, i₅} ⊆ ran(M(d)) ⊆ dom(Σ.C) (S3) ⊆ dom(Σ'.C) (D-I). ✓
- *D-CTG-post:* {[1,1], [1,2], [1,3]} = {[1,k] : 1 ≤ k ≤ 3}, contiguous. ✓

**Boundary case: R = ∅.** Same five-position arrangement. Contract at p = [1,4] with w = [0,2], so c = 2 and r = p ⊕ w = [1,6].

**Three-region partition.** L = {[1,1], [1,2], [1,3]}, X = {[1,4], [1,5]}, R = ∅.

**Shift computation.** R = ∅, so Q₃ = ∅.

**Post-state.** M'(d) = {[1,1] → i₁,  [1,2] → i₂,  [1,3] → i₃}

**Verification:**

- *D-L:* M'(d)([1,k]) = iₖ = M(d)([1,k]) for k ∈ {1,2,3}. ✓
- *D-SHIFT:* R = ∅, vacuously satisfied. ✓
- *D-DOM:* {v ∈ dom(M'(d)) : subspace(v) = 1} = {[1,1], [1,2], [1,3]} = L ∪ ∅ = L. ✓
- *D-DP:* L ∩ Q₃ = ∅ (Q₃ = ∅ since R = ∅). ✓
- *D-CTG-post:* {[1,1], [1,2], [1,3]} = {[1,k] : 1 ≤ k ≤ 3}, contiguous. ✓
- *D-MIN-post:* min L = [1,1] = [S, 1]. ✓
- *S8-depth-post:* All positions have depth 2 (unchanged from pre-state). ✓
- *S8a-post:* All positions in L satisfy S8a by pre-state invariant. ✓
- *S2-post:* Three distinct V-positions, each assigned a unique I-address. ✓
- *S3-post:* {i₁, i₂, i₃} ⊆ ran(M(d)) ⊆ dom(Σ.C) ⊆ dom(Σ'.C). ✓

**Boundary case: L = ∅ and R = ∅ (full deletion).** Same five-position arrangement. Contract at p = [1,1] with w = [0,5], so c = 5 and r = p ⊕ w = [1,6].

**Three-region partition.** L = ∅, X = {[1,1], [1,2], [1,3], [1,4], [1,5]}, R = ∅.

**Shift computation.** R = ∅, so Q₃ = ∅.

**Post-state.** M'(d) restricted to subspace 1 is empty: dom(M'(d)) ∩ {v : subspace(v) = 1} = ∅.

**Verification:**

- *D-L:* L = ∅, vacuously satisfied. ✓
- *D-SHIFT:* R = ∅, vacuously satisfied. ✓
- *D-DOM:* {v ∈ dom(M'(d)) : subspace(v) = 1} = ∅ = ∅ ∪ ∅ = L ∪ Q₃. ✓
- *D-DP:* L ∩ Q₃ = ∅ (Q₃ = ∅ since R = ∅). ✓
- *D-CTG-post:* V_S(d') = ∅, vacuously contiguous. ✓
- *D-MIN-post:* V_S(d') = ∅, D-MIN holds vacuously. ✓
- *S8-depth-post:* V_S(d') = ∅, S8-depth holds vacuously. ✓
- *S8a-post:* V_S(d') = ∅, S8a holds vacuously. ✓
- *S2-post:* No subspace-1 positions exist. ✓
- *S3-post:* No subspace-1 I-addresses to check. ✓

**Cross-subspace preservation: text contraction leaves link subspace untouched.** The setup mirrors the insertion cross-subspace example, now with five contiguous text positions alongside the sparse link subspace.

M(d) = {[1,1] → i₁, [1,2] → i₂, [1,3] → i₃, [1,4] → i₄, [1,5] → i₅,  [2,5] → ℓ₁, [2,9] → ℓ₂}

Contract at p = [1,2] with w = [0,2]. Parameters: S = 1, c = w₂ = 2, r = p ⊕ w = [1,4], #p = 2, Pos(w), w₁ = 0, containment p₂ + w₂ − 1 = 3 ≤ 5 = N. ✓

The contraction is defined only on the text subspace (S = 1). The link subspace V_2(d) = {[2,5], [2,9]} is exempt from D-CTG, D-MIN, D-SEQ — it lies outside the contraction's quantifier ranges (D-SHIFT's R, D-L's L) since those are subsets of V_1(d). D-CS asserts both per-subspace domain equality and mapping equality across non-text subspaces.

**Three-region partition (text subspace only).** L = {[1,1]}, X = {[1,2], [1,3]}, R = {[1,4], [1,5]}.

**Shift computation.** w_ord = [2]. For each v ∈ R:

- σ([1,4]) = vpos(1, [4] ⊖ [2]) = vpos(1, [2]) = [1,2]
- σ([1,5]) = vpos(1, [5] ⊖ [2]) = vpos(1, [3]) = [1,3]

Q₃ = {[1,2], [1,3]}.

**Post-state.** M'(d) = {[1,1] → i₁, [1,2] → i₄, [1,3] → i₅,  [2,5] → ℓ₁, [2,9] → ℓ₂}

| V (before) | I (before) | V (after) | I (after) | Region |
|---|---|---|---|---|
| [1,1] | i₁ | [1,1] | i₁ | left (D-L) |
| [1,2] | i₂ | — (vacated) | — | contracted (X) |
| [1,3] | i₃ | — (vacated) | — | contracted (X) |
| [1,4] | i₄ | [1,2] | i₄ | shifted (D-SHIFT) |
| [1,5] | i₅ | [1,3] | i₅ | shifted (D-SHIFT) |
| [2,5] | ℓ₁ | [2,5] | ℓ₁ | cross-subspace (D-CS) |
| [2,9] | ℓ₂ | [2,9] | ℓ₂ | cross-subspace (D-CS) |

**Verification:**

- *D-L:* M'(d)([1,1]) = i₁ = M(d)([1,1]). ✓
- *D-SHIFT:* M'(d)([1,2]) = i₄ = M(d)([1,4]); M'(d)([1,3]) = i₅ = M(d)([1,5]). ✓
- *D-DOM:* {v ∈ dom(M'(d)) : subspace(v) = 1} = {[1,1], [1,2], [1,3]} = L ∪ Q₃. ✓
- *D-CS:* {v ∈ dom(M'(d)) : subspace(v) = 2} = {[2,5], [2,9]} = {v ∈ dom(M(d)) : subspace(v) = 2}; per-position mapping equality M'(d)([2,5]) = ℓ₁ = M(d)([2,5]) and M'(d)([2,9]) = ℓ₂ = M(d)([2,9]). The sparse link subspace is preserved verbatim — the tombstone gap at [2,6], [2,7], [2,8] remains. ✓
- *D-I:* dom(Σ'.C) = dom(Σ.C) and per-address values unchanged. The contraction modifies only M(d); no I-addresses are allocated or deallocated, and no link payloads are touched. ✓
- *D-BJ:* [1,4] < [1,5] and σ([1,4]) = [1,2] < [1,3] = σ([1,5]). ✓
- *D-SEP(a):* ord(r) ⊖ w_ord = [4] ⊖ [2] = [2] = ord(p). ✓
- *D-SEP(b):* min Q₃ = [1,2], ord([1,2]) = [2] = ord(p). ✓
- *D-DP:* L ∩ Q₃ = ∅; min Q₃ ordinal = [2] = ord(p); all L ordinals < ord(p). ✓
- *D-CTG-post:* V_1(d') = {[1,1], [1,2], [1,3]} = {[1,k] : 1 ≤ k ≤ 3}, contiguous. ✓
- *D-MIN-post:* min V_1(d') = [1,1] = [S, 1]. ✓
- *D-SEQ-post:* V_1(d') = {[1,k] : 1 ≤ k ≤ N − c} = {[1,k] : 1 ≤ k ≤ 3}. ✓
- *Non-text V_2(d'):* the sparse {[2,5], [2,9]} is carried verbatim by the D-CS line above; the foundation imposes no D-CTG/D-MIN/D-SEQ obligation on V_2, so these three V_1 checks discharge the full post-state. ✓
- *S2-post:* Five distinct V-positions in dom(M'(d)), each assigned a unique I-address. ✓
- *S3-post:* {i₁, i₄, i₅, ℓ₁, ℓ₂} ⊆ ran(M(d)) ⊆ dom(Σ.C) (S3) = dom(Σ'.C) (D-I). ✓
- *S8-depth-post:* All seven post-state V-positions have depth 2 — text positions by D-L and shift's depth preservation, link positions by D-CS retaining pre-state depths. ✓
- *S8a-post:* All post-state V-positions are zero-free, depth 2, componentwise positive. ✓

∎


## Span Width Preservation Under Contraction

The point-level shift σ (D-SHIFT) lifts to a span-level property dual to I3-S, connecting the contraction to the span algebra framework of ASN-0053. Consider a level-uniform span σₛ = (s, ℓ) with start in the right region — that is, s ∈ R, subspace(s) = 1, #s = #ℓ = 2, and actionPoint(ℓ) = 2 (ordinal-level in the same sense established for I3-S, restricted to the contraction's depth precondition #p = 2). Extend σ from R to any V-position v with ord(v) ≥ w_ord by defining σ(v) = vpos(1, ord(v) ⊖ w_ord); this is well-defined by TA2 (ASN-0034) and matches σ's definition on R verbatim. Define the contracted span σ'ₛ = (σ(s), ℓ). We verify that σ'ₛ is a well-formed span (T12, ASN-0034): ℓ > 0 is inherited from σₛ, and actionPoint(ℓ) = 2 ≤ #σ(s) = 2 by vpos's result-length identity at depth 1.

**D-S** — *SpanContractionPreservation* (LEMMA, introduced). For a level-uniform span σₛ = (s, ℓ) with s ∈ R, subspace(s) = 1, #s = #ℓ = 2, and actionPoint(ℓ) = 2, the contracted span σ'ₛ = (σ(s), ℓ) satisfies:

(a) reach(σ'ₛ) = σ(reach(σₛ))

(b) width(σ'ₛ) = ℓ

*Derivation of (a).* Both endpoints lie in subspace 1 at depth 2, so we work through the ordinal. Since actionPoint(ℓ) = 2 and Pos(ℓ), ℓ = [0, c'] with c' ≥ 1, and ℓ_ord = [c']. From s ∈ R, OrdinalExceedsDisplacement gives ord(s) ≥ w_ord (so σ(s) is well-defined and Pos); at depth 1, ord(s) = [s₂], w_ord = [c], and σ(s) = vpos(1, [s₂] ⊖ [c]) = [1, s₂ − c] (TumblerSub at depth 1). The far endpoint's ordinal is ord(reach(σₛ)) = ord(s) ⊕ ℓ_ord = [s₂] ⊕ [c'] = [s₂ + c'] (OrdAddHom (a), then TumblerAdd); it dominates w_ord, since TumblerAdd's `a ⊕ w > a` gives [s₂ + c'] > [s₂] = ord(s) ≥ w_ord (clause (i)–(ii) of OrdinalExceedsDisplacement), so by TA2 (ASN-0034) σ(reach(σₛ)) = vpos(1, [s₂ + c'] ⊖ [c]) = [1, (s₂ + c') − c] is well-defined.

Now reach(σ'ₛ) = σ(s) ⊕ ℓ = [1, s₂ − c] ⊕ [0, c'] = [1, (s₂ − c) + c'] (TumblerAdd at action point 2). Both σ(reach(σₛ)) = [1, (s₂ + c') − c] and reach(σ'ₛ) = [1, (s₂ − c) + c'] agree at position 1 (both 1), so the whole claim collapses to a single depth-1 (position-2) natural-number identity:

`(s₂ + c') − c = (s₂ − c) + c'`   for `s₂ ≥ c`,

where each `+` is ℕ addition and each `−` is the ℕ subtraction *induced* by the depth-1 tumbler difference (`[a] ⊖ [c] = [a − c]` for `a ≥ c`). Write `x = s₂ − c`, i.e. `[x] = [s₂] ⊖ [c]`. ReverseInverse at depth 1 (ASN-0034) gives `[s₂ − c] ⊕ [c] = [s₂]`, which is the ℕ fact `x + c = s₂`. Substituting and regrouping: `(s₂ + c') − c = ((x + c) + c') − c = ((x + c') + c) − c`, where the inner regrouping `(x + c) + c' = x + (c + c') = x + (c' + c) = (x + c') + c` uses associativity twice via TA-assoc (ASN-0034, at depth 1 where `[a] ⊕ [b] = [a + b]`, with the positive operands `c, c' ≥ 1`) and the swap `c + c' = c' + c` by ℕ-addition commutativity (a standard property of the carrier set ℕ). The depth-1 partial inverse TA4 (`(y + c) − c = y`, with `y = x + c'`) then cancels `c` to leave `x + c' = (s₂ − c) + c'`. The two tumblers therefore agree componentwise, so σ(reach(σₛ)) = reach(σ'ₛ). ✓ ∎

*Derivation of (b).* The span σ'ₛ = (σ(s), ℓ) is level-uniform: #σ(s) = 2 = #ℓ by vpos's result-length identity. Its width is by definition its second component ℓ; consistently, by WR (WidthRecovery, ASN-0053), width(σ'ₛ) = reach(σ'ₛ) ⊖ start(σ'ₛ) = (σ(s) ⊕ ℓ) ⊖ σ(s) = ℓ. ✓ ∎

*Verification against worked example.* From the contraction example above (p = [1,2], w = [0,2], c = 2), take the span σₛ = ([1, 4], [0, 1]) covering the single pre-contraction position [1, 4]. Then reach(σₛ) = [1, 4] ⊕ [0, 1] = [1, 5], and σ'ₛ = (σ([1, 4]), [0, 1]) = ([1, 2], [0, 1]). For (a): reach(σ'ₛ) = [1, 2] ⊕ [0, 1] = [1, 3], and σ(reach(σₛ)) = σ([1, 5]) = [1, 3]. ✓ For (b): width(σ'ₛ) = [0, 1] = ℓ. ✓


## Statement Registry

| Label | Type | Statement | Status |
|-------|------|-----------|--------|
| M(d) | definition | M(d) : T ⇀ T — arrangement function mapping V-positions to I-addresses for document d | cited (ASN-0036) |
| subspace(v) | definition | subspace(v) = v₁ — the first component of a V-position, identifying its subspace | cited (ASN-0036) |
| ordinal-level | definition | A span σ = (s, ℓ) is ordinal-level when actionPoint(ℓ) = #ℓ (the width acts at the deepest component of ℓ) | introduced (local) |
| S8-depth | invariant | (A d, v₁, v₂ : v₁ ∈ dom(M(d)) ∧ v₂ ∈ dom(M(d)) ∧ (v₁)₁ = (v₂)₁ : #v₁ = #v₂) — uniform V-position depth per subspace | cited (ASN-0036) |
| S8a | axiom | (A v ∈ dom(M(d)) :: zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0)) — V-position well-formedness | cited (ASN-0036) |
| I3 | postcondition | (A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v ≥ p : shift(v, n) ∈ dom(M'(d)) ∧ M'(d)(shift(v, n)) = M(d)(v)) | introduced |
| I3-L | frame | (A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v < p : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v)) | introduced |
| I3-X | frame | (A v : v ∈ dom(M(d)) ∧ subspace(v) ≠ S : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v)) | introduced |
| I3-D | frame | (A d' ≠ d : M'(d') = M(d')) | introduced |
| I3-V | postcondition | (A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v ≥ p ∧ v ∉ {shift(u, n) : u ∈ dom(M(d)) ∧ subspace(u) = S ∧ u ≥ p} : v ∉ dom(M'(d))) | introduced |
| I3-C | frame | dom(C') = dom(C) ∧ (A a ∈ dom(C) : C'(a) = C(a)) — content store unchanged | introduced |
| I3-CS | postcondition | (A v : v ∈ dom(M'(d)) ∧ subspace(v) = S : left-region ∨ shifted-image) — domain closure within subspace S | introduced |
| I3-CX | postcondition | (A v : v ∈ dom(M'(d)) ∧ subspace(v) ≠ S : v ∈ dom(M(d))) — domain closure across subspaces | introduced |
| I3-VD | lemma | S8-depth preserved post-insertion across all subspaces: subspace S by left/shifted region analysis, other subspaces by I3-X and I3-CX | derived |
| I3-VP | lemma | (A v ∈ dom(M'(d)) : zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0)) — S8a preserved post-insertion | derived |
| I3-S3 | lemma | (A v : v ∈ dom(M'(d)) : M'(d)(v) ∈ dom(C')) — referential integrity preserved post-insertion | derived |
| I3-S2 | lemma | M'(d) is a function — S2 preserved post-insertion; pairwise disjointness of assignment regions ensures no double-assignment | derived |
| I3-fin | lemma | dom(M'(d)) is finite — S8-fin preserved post-insertion; domain closure (I3-CS, I3-CX) and injectivity (TS2) bound M'(d) by pre-state | derived |
| I3-S7 | lemma | S7a, S7b, S7d preserved post-insertion (and S7 as a corollary) — trivially by I3-C (dom(C') = dom(C), per-address values unchanged) and I3-D (document set unchanged) | derived |
| I3-S | lemma | For level-uniform σ = (s, ℓ) with actionPoint(ℓ) = m and n ≥ 1: reach((shift(s, n), ℓ)) = shift(reach(σ), n) and width preserved | introduced |
| OrdinalDisplacement | definition | δ(n, m) = [0, ..., 0, n] of length m, action point m | cited (ASN-0034) |
| OrdinalShift | definition | shift(v, n) = v ⊕ δ(n, #v) | cited (ASN-0034) |
| TS1 | lemma | shift preserves strict order: v₁ < v₂ ⟹ shift(v₁, n) < shift(v₂, n) | cited (ASN-0034) |
| TS2 | lemma | shift is injective: shift(v₁, n) = shift(v₂, n) ⟹ v₁ = v₂ | cited (ASN-0034) |
| TS3 | lemma | shift(shift(v, n₁), n₂) = shift(v, n₁ + n₂) — shift amounts compose additively | cited (ASN-0034) |
| OrdShiftHom | lemma | For #v = m ≥ 2, n ≥ 1: (a) subspace(shift(v, n)) = subspace(v); (b) v satisfies S8a ⟹ shift(v, n) satisfies S8a | cited (ASN-0036) |
| SpanReach | definition | reach(σ) = start(σ) ⊕ width(σ) | cited (ASN-0053) |
| TS4 | lemma | shift(v, n) > v for n ≥ 1 | cited (ASN-0034) |
| TA-assoc | lemma | (a ⊕ b) ⊕ c = a ⊕ (b ⊕ c) when both sides are well-defined | cited (ASN-0034) |
| TumblerAdd | definition | a ⊕ w: copy prefix, advance at action point, copy tail from w | cited (ASN-0034) |
| TumblerSub | definition | a ⊖ w: zero prefix, reverse at divergence, copy tail from a | cited (ASN-0034) |
| WR | lemma | For level-uniform σ: reach(σ) ⊖ start(σ) = width(σ) | cited (ASN-0053) |
| S6 | lemma | For level-uniform σ: #reach(σ) = #s | cited (ASN-0053) |
| T12 | precondition | span(s, ℓ) well-formed when ℓ > 0 and actionPoint(ℓ) ≤ #s | cited (ASN-0034) |
| S2 | axiom | (A d, v : v ∈ dom(M(d)) : M(d)(v) is uniquely determined) — arrangement functionality | cited (ASN-0036) |
| S3 | invariant | (A d, v : v ∈ dom(M(d)) : M(d)(v) ∈ dom(C)) — referential integrity | cited (ASN-0036) |
| S8-fin | invariant | For each document d, dom(M(d)) is finite | cited (ASN-0036) |
| D-CTG | invariant | V_1(d) contiguity (text subspace only; V_2(d) exempt) | cited (ASN-0036) |
| D-MIN | invariant | min(V_1(d)) = [1, 1, ..., 1] (text subspace only) | cited (ASN-0036) |
| D-SEQ | lemma | V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n} (text subspace only) | cited (ASN-0036) |
| T4 | axiom | Address tumblers have ≤ 3 zeros as field separators; every field component strictly positive | cited (ASN-0034) |
| S0 | invariant | a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a) — content immutability | cited (ASN-0036) |
| T1 | axiom | Lexicographic total order on tumblers | cited (ASN-0034) |
| TA2 | lemma | Subtraction well-defined when a ≥ w | cited (ASN-0034) |
| TA3-strict | lemma | a < b ∧ a ≥ w ∧ b ≥ w ∧ #a = #b ⟹ a ⊖ w < b ⊖ w — strict order preservation under subtraction | cited (ASN-0034) |
| TA4 | lemma | (a ⊕ w) ⊖ w = a — partial inverse of addition by subtraction | cited (ASN-0034) |
| ReverseInverse | lemma | (a ⊖ w) ⊕ w = a under equal-length, zero-prefix, positivity conditions — reverse partial inverse | cited (ASN-0034) |
| TA6 | lemma | every zero tumbler is strictly less than every positive tumbler | cited (ASN-0034) |
| ord(v) | definition | Ordinal extraction: ord(v) = [v₂, ..., vₘ] strips the subspace identifier; precondition #v ≥ 2 | introduced (local) |
| vpos(S, o) | definition | V-position reconstruction: vpos(S, o) = [S, o₁, ..., oₖ]; preconditions #o ≥ 1, S ≥ 1; inverse of ord; S8a-closure when o componentwise positive | introduced (local) |
| w_ord | definition | Ordinal displacement projection: w_ord = [w₂, ..., wₘ] for V-depth w with w₁ = 0; preconditions #w ≥ 2, w₁ = 0 | introduced (local) |
| OrdinalOrderEquivalence | lemma | v₁ < v₂ ⟺ ord(v₁) < ord(v₂) when subspace(v₁) = subspace(v₂) ∧ #v₁ = #v₂ | introduced (derived from T1) |
| OrdAddHom | lemma | (a) ord(p ⊕ w) = ord(p) ⊕ w_ord; (b) subspace(p ⊕ w) = subspace(p); (c) p ⊕ w = vpos(subspace(p), ord(p) ⊕ w_ord). Preconditions: #p = m ≥ 2, w₁ = 0, #w = m, Pos(w) | introduced (derived from TumblerAdd) |
| OrdinalExceedsDisplacement | lemma | For contraction (r = p ⊕ w, #p = 2, p ∈ V_1(d)) and any v with subspace(v) = 1, #v = 2, v ≥ r: ord(v) > w_ord, ord(v) ⊖ w_ord well-defined and Pos — right-region ordinal dominates the displacement | introduced (derived from TumblerAdd a⊕w≥w, TA4, TA2, TA3-strict, T1, S8a) |
| Contraction | operation | Remove span (p, w) from the text subspace of document d (S = 1); preconditions: S = 1, p ∈ V_1(d), Pos(w), #w = #p, w₁ = 0, #p = 2, containment (p₂ + w₂ − 1 ≤ N); postconditions: D-SHIFT, D-DOM; frame: D-L, D-CS, D-CD, D-I | introduced |
| ThreeRegions | definition | L = {v ∈ V_1(d) : v < p}, X = {v ∈ V_1(d) : p ≤ v < r}, R = {v ∈ V_1(d) : v ≥ r}; partition of V_1(d) | introduced |
| Q₃ | definition | Q₃ = {σ(v) : v ∈ R} — the set of shifted right-region positions in the post-state | introduced |
| D-SHIFT | postcondition | (A v ∈ R : M'(d)(σ(v)) = M(d)(v)) where σ(v) = vpos(S, ord(v) ⊖ w_ord) | introduced |
| D-L | frame | (A v ∈ L : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v)) | introduced |
| D-DOM | postcondition | {v ∈ dom(M'(d)) : subspace(v) = S} = L ∪ Q₃ | introduced |
| D-CS | frame | (A S' ≠ S : {v ∈ dom(M'(d)) : subspace(v) = S'} = {v ∈ dom(M(d)) : subspace(v) = S'}) ∧ (A v : v ∈ dom(M(d)) ∧ subspace(v) ≠ S : M'(d)(v) = M(d)(v)) | introduced |
| D-CD | frame | Cross-document arrangements unchanged | introduced |
| D-I | frame | Σ'.C = Σ.C — content store unchanged (exact equality, strictly stronger than S0) | introduced |
| D-BJ | lemma | σ : R → Q₃ is an order-preserving injection (hence a bijection onto its image Q₃): (a) v₁ < v₂ ⟹ σ(v₁) < σ(v₂), (b) v₁ ≠ v₂ ⟹ σ(v₁) ≠ σ(v₂) | introduced |
| D-SEP | lemma | ord(r) ⊖ w_ord = ord(p); when R ≠ ∅, min Q₃ ordinal = ord(p) | introduced |
| D-DP | lemma | L ∩ Q₃ = ∅ and no residual gap at contraction boundary | introduced |
| S8-depth-post | lemma | Post-state V-positions in subspace S share depth 2 | introduced |
| S8a-post | lemma | Post-state V-positions are zero-free, of depth at least 2, and componentwise positive | introduced |
| D-CTG-post | lemma | At S = 1: post-state V_1(d) is contiguous; non-text subspaces preserved verbatim by D-CS | introduced |
| D-MIN-post | lemma | At S = 1: post-state min V_1(d) = [1, 1] when non-empty; vacuous when empty; non-text subspaces preserved verbatim by D-CS | introduced |
| D-SEQ-post | lemma | At S = 1: when post-state V_1(d) non-empty, V_1(d) = {[1, k] : 1 ≤ k ≤ N − c}; non-text subspaces preserved verbatim by D-CS | introduced |
| S8-fin-post | lemma | Post-state dom(M'(d)) is finite | introduced |
| S2-post | lemma | Post-state M'(d) is a function | introduced |
| S3-post | lemma | Post-state ran(M'(d)) ⊆ dom(Σ'.C) | introduced |
| S7-post | lemma | Post-state satisfies S7a, S7b, S7d (and S7 as a corollary) — trivially by D-I (Σ'.C = Σ.C) and D-CD (other documents unchanged) | introduced |
| D-S | lemma | For level-uniform σₛ = (s, ℓ) with s ∈ R and actionPoint(ℓ) = 2: reach((σ(s), ℓ)) = σ(reach(σₛ)) and width preserved — span-level dual of I3-S for contraction | introduced |


## Open Questions

- When external state records a V-position, what must the system provide to allow that reference to be updated after a shift repositions it?
- Can the gap-closure formula (D-SEP) and dense partition (D-DP) be generalized to ordinals of depth greater than one while preserving the round-trip property (ord(p) ⊕ w_ord) ⊖ w_ord = ord(p) and the commutativity of shift with ordinal increment?
- At depth greater than one, TA4's zero-prefix precondition collides with S8a's componentwise positivity at the intermediate components 2..m − 1: what weaker inverse law could replace TA4 so that the projection round-trip survives at deeper ordinals?
