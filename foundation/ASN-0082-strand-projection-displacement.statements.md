> **ASN-0082 · Strand Projection Displacement** — condensed claim statements  
> [← Full note](ASN-0082-strand-projection-displacement.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0082 Claim Statements

*Source: ASN-0082-strand-projection-displacement.md (revised 2026-04-09) — Extracted: 2026-05-30*

## Definition — ArrangementFunction

M(d) : T ⇀ T — arrangement function mapping V-positions to I-addresses for document d

## Definition — Subspace

subspace(v) = v₁ — the first component of a V-position, identifying its subspace

## Definition — OrdinalLevel

A span σ = (s, ℓ) is ordinal-level when actionPoint(ℓ) = #ℓ (the width acts at the deepest component of ℓ)

## Definition — OrdinalExtraction

For a V-position v with `#v = m ≥ 2`, `ord(v) = [v₂, ..., vₘ]` — the tumbler of length m − 1 obtained by stripping the subspace identifier (component 1) and reindexing, so `ord(v)ⱼ = vⱼ₊₁` for `1 ≤ j ≤ m − 1`.

When v satisfies S8a (every component positive), every component of ord(v) is positive, since ord(v) drops only position 1.

## Definition — VPositionReconstruction

For subspace identifier `S ≥ 1` and ordinal `o = [o₁, ..., oₖ]` with `#o ≥ 1`, `vpos(S, o) = [S, o₁, ..., oₖ]` — prepend S and reindex, so `vpos(S, o)₁ = S` and `vpos(S, o)ⱼ₊₁ = oⱼ`.

Inverses by construction (component identity, T3): `ord(vpos(S, o)) = o` and `vpos(subspace(v), ord(v)) = v`.

*S8a-closure (local postcondition):* when `S ≥ 1` and o is componentwise positive, `vpos(S, o)` is zero-free with all components positive and depth `#o + 1 ≥ 2`, so it satisfies S8a.

## Definition — OrdinalDisplacementProjection

For a displacement w with `w₁ = 0` and `#w = m ≥ 2`, `w_ord = [w₂, ..., wₘ]` — the tumbler of length m − 1 obtained by stripping the (zero) first component, with `w_ordⱼ = wⱼ₊₁`.

When `Pos(w)` (TA-Pos, ASN-0034), the witness for positivity sits at some position `i ≥ 2` (since `w₁ = 0`), so `Pos(w_ord)`; and the first (leftmost) nonzero of w, at position `actionPoint(w) ≥ 2`, maps to position `actionPoint(w) − 1` of w_ord, giving `actionPoint(w_ord) = actionPoint(w) − 1`.

## Definition — ThreeRegions

```
L = {v ∈ V_1(d) : v < p}            — left of contraction
X = {v ∈ V_1(d) : p ≤ v < r}        — the contracted interval
R = {v ∈ V_1(d) : v ≥ r}            — right of contraction
```

By trichotomy of the total order (T1, ASN-0034), every v ∈ V_1(d) falls in exactly one region.

## Definition — ShiftedRightRegion

Q₃ = {σ(v) : v ∈ R} — the set of shifted right-region positions in the post-state

## Definition — OrdinalDisplacement

δ(n, m) = [0, 0, ..., 0, n] of length m — zero at positions 1 through m − 1, and n at position m, with action point m

## Definition — OrdinalShift

shift(v, n) = v ⊕ δ(n, #v)

By TumblerAdd: shift(v, n)ᵢ = vᵢ for i < m, and shift(v, n)ₘ = vₘ + n.

## Definition — TumblerAdd

a ⊕ w: copy prefix, advance at action point, copy tail from w

## Definition — TumblerSub

a ⊖ w: zero prefix, reverse at divergence, copy tail from a

## Definition — SpanReach

reach(σ) = start(σ) ⊕ width(σ)

## Definition — Contraction

Remove span (p, w) from the text subspace of document d (S = 1)

*Preconditions:* S = 1, p ∈ V_1(d), Pos(w), #w = #p, w₁ = 0, #p = 2, containment (p₂ + w₂ − 1 ≤ N)

*Postconditions:* D-SHIFT, D-DOM

*Frame:* D-L, D-CS, D-CD, D-I

---

## S8-depth — FixedDepthVPositions (invariant, cited)

`(A d, v₁, v₂ : v₁ ∈ dom(M(d)) ∧ v₂ ∈ dom(M(d)) ∧ (v₁)₁ = (v₂)₁ : #v₁ = #v₂)` — uniform V-position depth per subspace

## S8a — VPositionWellFormedness (axiom, cited)

`(A v ∈ dom(M(d)) :: zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))` — V-position well-formedness

## S2 — ArrangementFunctionality (axiom, cited)

`(A d, v : v ∈ dom(M(d)) : M(d)(v) is uniquely determined)` — arrangement functionality

## S3 — ReferentialIntegrity (invariant, cited)

`(A d, v : v ∈ dom(M(d)) : M(d)(v) ∈ dom(C))` — referential integrity

## S8-fin — FiniteArrangement (invariant, cited)

For each document d, dom(M(d)) is finite

## S0 — ContentImmutability (invariant, cited)

`a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a)` — content immutability

## D-CTG — VContiguity (invariant, cited)

`(A d, u, q : u ∈ V_1(d) ∧ q ∈ V_1(d) ∧ u < q : (A v : subspace(v) = 1 ∧ #v = #u ∧ zeros(v) = 0 ∧ u < v < q : v ∈ V_1(d)))` (text subspace only; V_2(d) exempt)

## D-MIN — VMinimumPosition (invariant, cited)

min(V_1(d)) = [1, 1, ..., 1] (text subspace only)

## D-SEQ — SequentialPositions (lemma, cited)

V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n} (text subspace only)

## T1 — LexicographicOrder (axiom, cited)

Lexicographic total order on tumblers

## T4 — AddressTumblerStructure (axiom, cited)

Address tumblers have ≤ 3 zeros as field separators; every field component strictly positive

## T12 — SpanWellFormedness (precondition, cited)

span(s, ℓ) well-formed when ℓ > 0 and actionPoint(ℓ) ≤ #s

## TS1 — ShiftOrderPreservation (lemma, cited)

shift preserves strict order: v₁ < v₂ ⟹ shift(v₁, n) < shift(v₂, n)

## TS2 — ShiftInjectivity (lemma, cited)

shift is injective: shift(v₁, n) = shift(v₂, n) ⟹ v₁ = v₂

## TS3 — ShiftComposition (lemma, cited)

shift(shift(v, n₁), n₂) = shift(v, n₁ + n₂) — shift amounts compose additively

## TS4 — ShiftStrictAdvance (lemma, cited)

shift(v, n) > v for n ≥ 1

## OrdShiftHom — OrdinalShiftPreservation (lemma, cited)

For #v = m ≥ 2, n ≥ 1:

(a) subspace(shift(v, n)) = subspace(v)

(b) v satisfies S8a ⟹ shift(v, n) satisfies S8a

## TA2 — WellDefinedSubtraction (lemma, cited)

Subtraction well-defined when a ≥ w

## TA3-strict — OrderPreservationSubtractionStrict (lemma, cited)

a < b ∧ a ≥ w ∧ b ≥ w ∧ #a = #b ⟹ a ⊖ w < b ⊖ w — strict order preservation under subtraction

## TA4 — PartialInverse (lemma, cited)

(a ⊕ w) ⊖ w = a — partial inverse of addition by subtraction

## TA6 — ZeroTumblerMinimality (lemma, cited)

every zero tumbler is strictly less than every positive tumbler

## TA-assoc — TumblerAddAssociativity (lemma, cited)

(a ⊕ b) ⊕ c = a ⊕ (b ⊕ c) when both sides are well-defined

## ReverseInverse — ReversePartialInverse (lemma, cited)

(a ⊖ w) ⊕ w = a under equal-length, zero-prefix, positivity conditions — reverse partial inverse

## WR — WidthRecovery (lemma, cited)

For level-uniform σ: reach(σ) ⊖ start(σ) = width(σ)

## S6 — ReachDepth (lemma, cited)

For level-uniform σ: #reach(σ) = #s

---

## I3 — PostInsertionShift (postcondition, introduced)

*Preconditions:* d is a document; M(d) : T ⇀ T is its arrangement; p ∈ T with #p ≥ 2 and subspace(p) = S ≥ 1; depth-compatible: if {v ∈ dom(M(d)) : subspace(v) = S} ≠ ∅ then #p = #v for any such v (unique depth by S8-depth, ASN-0036); n ≥ 1; M'(d) is the post-insertion arrangement.

*Postcondition:*

`(A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v ≥ p : shift(v, n) ∈ dom(M'(d)) ∧ M'(d)(shift(v, n)) = M(d)(v))`

## I3-V — PostInsertionVacating (postcondition, introduced)

`(A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v ≥ p ∧ v ∉ {shift(u, n) : u ∈ dom(M(d)) ∧ subspace(u) = S ∧ u ≥ p} : v ∉ dom(M'(d)))`

## I3-L — PostInsertionLeftFrame (frame, introduced)

`(A v : v ∈ dom(M(d)) ∧ subspace(v) = S ∧ v < p : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))`

## I3-X — PostInsertionCrossSubspaceFrame (frame, introduced)

`(A v : v ∈ dom(M(d)) ∧ subspace(v) ≠ S : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))`

## I3-D — PostInsertionCrossDocumentFrame (frame, introduced)

`(A d' ≠ d : M'(d') = M(d'))`

## I3-C — PostInsertionContentFrame (frame, introduced)

`dom(C') = dom(C) ∧ (A a ∈ dom(C) : C'(a) = C(a))` — content store unchanged

## I3-CS — PostInsertionSubspaceClosure (postcondition, introduced)

`(A v : v ∈ dom(M'(d)) ∧ subspace(v) = S : (v < p ∧ v ∈ dom(M(d))) ∨ (E u : u ∈ dom(M(d)) ∧ subspace(u) = S ∧ u ≥ p : v = shift(u, n)))` — domain closure within subspace S

## I3-CX — PostInsertionCrossSubspaceClosure (postcondition, introduced)

`(A v : v ∈ dom(M'(d)) ∧ subspace(v) ≠ S : v ∈ dom(M(d)))` — domain closure across subspaces

## I3-VD — PostInsertionDepthUniformity (lemma, derived)

S8-depth preserved post-insertion across all subspaces.

For subspace S: `(A v₁, v₂ ∈ dom(M'(d)) : subspace(v₁) = subspace(v₂) = S ⟹ #v₁ = #v₂ = m)`.

By I3-CS, every v ∈ dom(M'(d)) with subspace(v) = S falls into exactly one of two regions:
- *Left region* (I3-L): v ∈ dom(M(d)) with subspace(v) = S and v < p; these have depth m by S8-depth on M(d).
- *Shifted region* (I3): shift(v, n) for v ∈ dom(M(d)) with subspace(v) = S and v ≥ p; #shift(v, n) = #δₙ = m by the result-length identity of TumblerAdd, and #v = m by S8-depth on M(d).

For any subspace S' ≠ S: by I3-X and I3-CX, the positions in dom(M'(d)) with subspace S' are exactly the positions in dom(M(d)) with subspace S', on which S8-depth holds by hypothesis.

## I3-VP — PostInsertionWellFormedness (lemma, derived)

`(A v ∈ dom(M'(d)) : zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))` — S8a preserved post-insertion

## I3-S3 — PostInsertionReferentialIntegrity (lemma, derived)

`(A v : v ∈ dom(M'(d)) : M'(d)(v) ∈ dom(C'))` — referential integrity preserved post-insertion

## I3-S2 — PostInsertionFunctionality (lemma, derived)

M'(d) is a function — S2 preserved post-insertion; pairwise disjointness of assignment regions ensures no double-assignment

## I3-fin — PostInsertionFiniteness (lemma, derived)

dom(M'(d)) is finite — S8-fin preserved post-insertion; domain closure (I3-CS, I3-CX) and injectivity (TS2) bound M'(d) by pre-state

## I3-S7 — PostInsertionAllocationInvariants (lemma, derived)

S7a, S7b, S7d preserved post-insertion (and S7 as a corollary) — trivially by I3-C (dom(C') = dom(C), per-address values unchanged) and I3-D (document set unchanged)

## I3-S — SpanShiftPreservation (lemma, introduced)

For a level-uniform span σ = (s, ℓ) with #s = #ℓ = m and actionPoint(ℓ) = m, and a shift amount n ≥ 1, the shifted span σ' = (shift(s, n), ℓ) satisfies:

(a) reach(σ') = shift(reach(σ), n)

(b) width(σ') = ℓ

---

## OrdinalOrderEquivalence — OrdinalOrderEquivalence (lemma, introduced)

For V-positions v₁, v₂ with subspace(v₁) = subspace(v₂) = S and #v₁ = #v₂ = m ≥ 2:

`v₁ < v₂ ⟺ ord(v₁) < ord(v₂)`

## OrdAddHom — OrdinalAdditionHomomorphism (lemma, introduced)

For a V-position p with `#p = m ≥ 2` and a displacement w with `w₁ = 0`, `#w = m`, and `Pos(w)`:

(a) `ord(p ⊕ w) = ord(p) ⊕ w_ord` — whole-tumbler addition commutes with ordinal extraction when the displacement has a zero first component.

(b) `subspace(p ⊕ w) = subspace(p)` — the subspace identifier is preserved under any ordinal-zero-prefixed displacement.

(c) `p ⊕ w = vpos(subspace(p), ord(p) ⊕ w_ord)` — the addition lifts cleanly through ord/vpos.

## OrdinalExceedsDisplacement — OrdinalExceedsDisplacement (lemma, introduced)

Fix the contraction parameters: `#p = 2`, `Pos(w)`, `w₁ = 0`, `p ∈ V_1(d)`, and `r = p ⊕ w`. For any tumbler v with `subspace(v) = 1`, `#v = 2`, and `v ≥ r`:

(i) `ord(r) ≥ w_ord` and `ord(r) > w_ord`

(ii) `ord(v) ≥ w_ord` and `ord(v) > w_ord`

(iii) `ord(v) ⊖ w_ord` is well-defined and `Pos`, equal to `ord(p)` when `v = r` and strictly greater than `ord(p)` (under T1) when `v > r`.

---

## D-SHIFT — RightShift (postcondition, introduced)

*Preconditions:* p ∈ V_1(d), #p = 2, Pos(w), #w = #p, w₁ = 0, containment satisfied. r = p ⊕ w; R = {v ∈ V_1(d) : v ≥ r}; M'(d) is the post-contraction arrangement.

`(A v : v ∈ R : σ(v) ∈ dom(M'(d)) ∧ M'(d)(σ(v)) = M(d)(v))`

where σ(v) = vpos(S, ord(v) ⊖ w_ord)

## D-L — LeftPreservation (frame, introduced)

`(A v : v ∈ L : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))`

## D-DOM — DomainCharacterization (postcondition, introduced)

`{v ∈ dom(M'(d)) : subspace(v) = S} = L ∪ Q₃`

## D-CS — CrossSubspaceFrame (frame, introduced)

`(A S' ≠ S : {v ∈ dom(M'(d)) : subspace(v) = S'} = {v ∈ dom(M(d)) : subspace(v) = S'})`

`∧ (A v : v ∈ dom(M(d)) ∧ subspace(v) ≠ S : M'(d)(v) = M(d)(v))`

## D-CD — CrossDocumentFrame (frame, introduced)

`(A d' ≠ d : M'(d') = M(d'))`

## D-I — ContentStoreFrame (frame, introduced)

`Σ'.C = Σ.C`

That is, `dom(Σ'.C) = dom(Σ.C)` and `(A a ∈ dom(Σ.C) : Σ'.C(a) = Σ.C(a))`.

## D-BJ — ShiftBijectivity (lemma, introduced)

*Preconditions:* #p = 2; v₁, v₂ ∈ R with v₁ ≠ v₂ (for injectivity) or v₁ < v₂ (for order-preservation).

σ : R → Q₃ is an order-preserving injection (hence a bijection onto its image Q₃):

(a) Order-preservation: `v₁ < v₂ ⟹ σ(v₁) < σ(v₂)`

(b) Injectivity: `v₁ ≠ v₂ ⟹ σ(v₁) ≠ σ(v₂)`

## D-SEP — GapClosure (lemma, introduced)

*Preconditions:* #p = 2; r = p ⊕ w.

(a) Algebraic identity: `ord(r) ⊖ w_ord = ord(p)`.

(b) When R ≠ ∅: r ∈ V_1(d), r = min(R), and ord(σ(r)) = ord(p), i.e., min({ord(u) : u ∈ Q₃}) = ord(p).

## D-DP — DensePartition (lemma, introduced)

*Preconditions:* #p = 2; L, X, R as defined by ThreeRegions; D-L, D-DOM, D-SHIFT, D-SEP, and D-CTG hold.

(a) No overlap: `L ∩ Q₃ = ∅`

(b) Boundary adjacency: when R ≠ ∅, `min({ord(u) : u ∈ Q₃}) = ord(p)`, and `(A v ∈ L : ord(v) < ord(p))`

## S8-depth-post — FixedDepthPreservation (lemma, introduced)

Post-state V-positions in subspace S share depth 2:

`(A v₁, v₂ ∈ dom(M'(d)) : subspace(v₁) = subspace(v₂) = S ⟹ #v₁ = #v₂ = 2)`

## S8a-post — WellFormednessPreservation (lemma, introduced)

Post-state V-positions are zero-free, of depth at least 2, and componentwise positive:

`(A v ∈ dom(M'(d)) : zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))`

## D-CTG-post — VContiguityPreservation (lemma, introduced)

At S = 1: post-state V_1(d) is contiguous; non-text subspaces preserved verbatim by D-CS.

V_1(d') = L ∪ Q₃ = {[1, k] : 1 ≤ k ≤ N − c}

Satisfies D-CTG's quantifier: for any u, q ∈ V_1(d') with u < q and any v with subspace(v) = 1, #v = 2, u < v < q: v ∈ V_1(d').

## D-MIN-post — VMinimumPreservation (lemma, introduced)

At S = 1: when the post-state V_1(d) is non-empty, min(V_1(d)) = [1, 1]; when the post-state V_1(d) is empty, D-MIN holds vacuously. Non-text subspaces preserved verbatim by D-CS.

## D-SEQ-post — SequentialPositionsPreservation (lemma, introduced)

At S = 1: when post-state V_1(d) non-empty, V_1(d) = {[1, k] : 1 ≤ k ≤ N − c}; non-text subspaces preserved verbatim by D-CS.

## S8-fin-post — FiniteArrangementPreservation (lemma, introduced)

Post-state dom(M'(d)) is finite.

## S2-post — ArrangementFunctionalityPost (lemma, introduced)

Post-state M'(d) is a function.

## S3-post — ReferentialIntegrityPost (lemma, introduced)

Post-state `ran(M'(d)) ⊆ dom(Σ'.C)`.

## S7-post — AllocationInvariantsPreservation (lemma, introduced)

Post-state satisfies S7a, S7b, S7d (and S7 as a corollary) — trivially by D-I (Σ'.C = Σ.C) and D-CD (other documents unchanged).

## D-S — SpanContractionPreservation (lemma, introduced)

For a level-uniform span σₛ = (s, ℓ) with s ∈ R, subspace(s) = 1, #s = #ℓ = 2, and actionPoint(ℓ) = 2, the contracted span σ'ₛ = (σ(s), ℓ) satisfies:

(a) reach(σ'ₛ) = σ(reach(σₛ))

(b) width(σ'ₛ) = ℓ
