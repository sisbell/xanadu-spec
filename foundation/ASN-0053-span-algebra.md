> **ASN-0053 · Span Algebra** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md)  
> [Condensed statements →](ASN-0053-span-algebra.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0053: Span Algebra

*2026-03-18, revised 2026-03-19*

A span σ = (s, ℓ) denotes the half-open interval ⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}, and by TA-strict every span is non-empty.

But a single span is merely a building block. This ASN formalizes the algebra over spans: comparing them, merging them, splitting them, normalizing collections to canonical form, and taking their difference.

Nelson provides span-sets as the mechanism for arbitrary content designation: "if you want to designate a separated series of items exactly, including nothing else, you do this by a span-set, which is a series of spans" (LM 4/25). Gregory confirms the implementation: two dedicated merge sites in the backend (the enfilade-level `isanextensionnd` and the output-level `putvspaninlist`) enforce precise adjacency and overlap conditions on spans.


## The reach function

For a span σ = (s, ℓ), we write:

  start(σ) = s,    width(σ) = ℓ,    reach(σ) = s ⊕ ℓ

The reach is the first position beyond σ — the exclusive upper bound. It is well-defined by TA0 and satisfies reach(σ) > start(σ) by TA-strict. Two spans with the same start and reach denote the same set of positions, because a span's content is entirely determined by its endpoints: Nelson states that "there is no choice as to what lies between; this is implicit in the choice of first and last point" (LM 4/25).

We shall need the reverse: given two positions a ≤ b on the tumbler line, can we recover the displacement from a to b — the unique width w such that a ⊕ w = b?

The displacement recovering b from a is w = b ⊖ a (TumblerSub, ASN-0034); WF and WR below discharge the conditions under which it round-trips.

**S6** (*LevelConstraint*). Two tumblers t₁ and t₂ are *level-compatible*, written level_compat(t₁, t₂), when they have the same length:

  level_compat(t₁, t₂)  ≡  #t₁ = #t₂

A span σ = (s, ℓ) is *level-uniform* when level_compat(s, ℓ), i.e., #s = #ℓ. For a level-uniform span, #reach(σ) = #s by the result-length identity (#(s ⊕ ℓ) = #ℓ), so start, width, and reach all share one tumbler length.

Gregory confirms the implementation enforces this: the split operation requires the cut and the width to share a tumbler length and aborts when this invariant is violated (Q14, Q15).

**WF** (*WellFormedSpanFromEndpoints*). For s, r ∈ T with s < r and #s = #r, the pair γ = (s, r ⊖ s) is a well-formed level-uniform span (satisfying T12) with reach(γ) = r.

*Proof.* Since s < r and #s = #r, the divergence k is of type (i) with k ≤ #s — equal length excludes the prefix case. The width r ⊖ s has a positive component at position k (namely rₖ − sₖ > 0), so it is positive with action point k ≤ #s; T12 is satisfied. By D1, reach(γ) = s ⊕ (r ⊖ s) = r. The span is level-uniform: #width(γ) = #(r ⊖ s) = max(#r, #s) = #s = #start(γ).  ∎

**WR** (*WidthRecovery*). For a level-uniform span σ = (s, ℓ): reach(σ) ⊖ start(σ) = width(σ).

*Proof.* The reach has #reach(σ) = #s (since #(s ⊕ ℓ) = #ℓ = #s by the result-length identity). Width recovery follows from displacement uniqueness in the foundation: since s ⊕ ℓ = reach(σ), D2 (DisplacementUnique, ASN-0034) gives reach(σ) ⊖ start(σ) = ℓ = width(σ), provided its preconditions hold for (a, b, w) = (s, reach(σ), ℓ). We discharge them: s < reach(σ) by TA-strict on T12; ℓ > 0 and its action point k ≤ #s by T12; s ⊕ ℓ = reach(σ) by definition of reach (so TA0's preconditions hold, giving #(s ⊕ ℓ) = #ℓ = #s); divergence(s, reach(σ)) = k ≤ #s, the D2 precondition on divergence (established as in WF's proof: #s = #reach(σ) excludes the prefix case, so the divergence is of type (i)); #s ≤ #reach(σ) since both equal #s. Every D2 precondition is met, so reach(σ) ⊖ start(σ) = width(σ).  ∎

A worked instance of the unequal-length failure: σ = ([1, 3, 5], [0, 2]) has reach [1, 5], but [1, 5] ⊖ [1, 3, 5] = [0, 2, 0] ≠ [0, 2] — when #start > #width the recovered displacement does not round-trip.


## Convexity

The first property of spans is that they admit no gaps:

**S0** (*Convexity*). `(A p, q, r : p ∈ ⟦σ⟧ ∧ r ∈ ⟦σ⟧ ∧ p ≤ q ≤ r : q ∈ ⟦σ⟧)`

*Proof.* If start(σ) ≤ p ≤ q ≤ r < reach(σ), then start(σ) ≤ q < reach(σ), so q ∈ ⟦σ⟧.  ∎

This follows solely from T1 being a total order. Every position between two members of a span is itself a member — a span cannot "skip" a position. Sub-addresses like [1, 3, 0, 5] that fall numerically between [1, 3] and [1, 7] are genuinely interior to any span containing both endpoints, because `tumblercmp` compares tumblers lexicographically without treating zero-separators specially (Gregory, Q11).


## How two spans relate

**SC** (*SpanClassification*). Given spans α and β, their relationship is determined by comparing starts and reaches under T1. Since T1 is a total order, five mutually exclusive cases arise:

(i) *Separated.* reach(α) < start(β) or reach(β) < start(α). The spans share no positions and have space between them.

(ii) *Adjacent.* reach(α) = start(β) or reach(β) = start(α). The spans share no positions but touch at a single boundary point.

(iii) *Proper overlap.* The spans share positions but neither contains the other: start(α) < start(β) < reach(α) < reach(β), or symmetrically.

(iv) *Containment.* One span's denotation is a proper subset of the other's: start(α) ≤ start(β) and reach(β) ≤ reach(α) with at least one inequality strict, or symmetrically.

(v) *Equal.* start(α) = start(β) and reach(α) = reach(β).

Cases (i) and (ii) are the *disjoint* cases — ⟦α⟧ ∩ ⟦β⟧ = ∅. Cases (iii), (iv), and (v) are the *overlapping* cases — ⟦α⟧ ∩ ⟦β⟧ ≠ ∅.

*Exhaustiveness.* Assume without loss of generality that start(α) ≤ start(β); configurations with start(α) > start(β) yield the same case with α, β exchanged, since each case clause is either symmetric (cases (i), (ii), (v)) or carries an explicit "or symmetrically" rider (cases (iii), (iv)). Compare reach(α) with start(β): if reach(α) < start(β), case (i); if reach(α) = start(β), case (ii); if reach(α) > start(β), the spans share positions. In the sharing case, compare start(α) with start(β): if start(α) < start(β), compare reach(α) with reach(β) — reach(α) < reach(β) gives case (iii), reach(α) ≥ reach(β) gives case (iv). If start(α) = start(β), compare reaches — reach(α) = reach(β) gives case (v), otherwise case (iv). Every ordering of the four boundary points {start(α), reach(α), start(β), reach(β)}, subject to start < reach for each span, falls into exactly one case.


## Intersection

**S1** (*IntersectionClosure*). For level-uniform spans α and β with level_compat(start(α), start(β)), the intersection is either empty or a single span. No configuration of two such spans produces a fragmented intersection.

Formally: for level-uniform spans α and β with level_compat(start(α), start(β)), either ⟦α⟧ ∩ ⟦β⟧ = ∅, or there exists a span γ such that ⟦γ⟧ = ⟦α⟧ ∩ ⟦β⟧.

*Proof.* Define s' = max(start(α), start(β)) and r' = min(reach(α), reach(β)). The forward inclusion holds unconditionally, before any case split, by the total order alone: take any t ∈ ⟦α⟧ ∩ ⟦β⟧, then start(α) ≤ t < reach(α) and start(β) ≤ t < reach(β), so t ≥ max(start(α), start(β)) = s' and t < min(reach(α), reach(β)) = r'; hence s' ≤ t < r'. Thus ⟦α⟧ ∩ ⟦β⟧ ⊆ {t : s' ≤ t < r'}.

Now split on r' against s'. If r' ≤ s', then {t : s' ≤ t < r'} = ∅, so the forward inclusion forces ⟦α⟧ ∩ ⟦β⟧ = ∅ — emptiness by derivation, not assertion; this covers the separated and adjacent cases. Otherwise r' > s', and:

  ⟦α⟧ ∩ ⟦β⟧ = {t : s' ≤ t < r'}

The forward inclusion is already in hand; for the reverse, take any t with s' ≤ t < r'. Then t ≥ s' = max(start(α), start(β)) ≥ start(α) and t < r' = min(reach(α), reach(β)) ≤ reach(α), so t ∈ ⟦α⟧; symmetrically t ≥ start(β) and t < reach(β), so t ∈ ⟦β⟧; hence t ∈ ⟦α⟧ ∩ ⟦β⟧. The intersection is therefore the half-open interval [s', r'). The set is non-empty (s' is a member since s' < r'). By level-uniformity and S6, all boundary tumblers — start(α), reach(α), start(β), reach(β) — share the same length. So #s' = #r', and with s' < r', WF gives that the pair γ = (s', r' ⊖ s') is a well-formed level-uniform span with reach(γ) = r'.  ∎

Gregory confirms this from the implementation: intersecting two spans yields at most one output span (Q10).

A concrete instance: let α = ([1, 3], [0, 4]) and β = ([1, 5], [0, 6]). Then reach(α) = [1, 7], reach(β) = [1, 11], s' = [1, 5], r' = [1, 7]. The intersection is ([1, 5], [0, 2]) — a single span covering positions [1, 5] through [1, 7) exclusive.


## The empty set is not a span

**S2** (*EmptyDistinction*). The empty set of positions is not the denotation of any span. Every well-formed span denotes a non-empty set.

This follows from T12 and TA-strict: ℓ > 0 and k ≤ #s imply s ⊕ ℓ > s, so the half-open interval [s, s ⊕ ℓ) contains at least s itself.

So the intersection of two disjoint spans is "no span" — the empty set — not a zero-width span.


## Merge

Two spans α and β are *adjacent* when the reach of one equals the start of the other:

  adjacent(α, β)  ≡  reach(α) = start(β)  ∨  reach(β) = start(α)

Adjacent spans share no positions (reach is an exclusive upper bound) but their denotations abut — there is no gap between them.

**S3** (*MergeEquivalence*). For level-uniform spans α and β with level_compat(start(α), start(β)), when they overlap or are adjacent, the union ⟦α⟧ ∪ ⟦β⟧ is the denotation of a single span. Moreover, this merged span is identical to one specified directly with the same endpoints.

*Proof.* Without loss of generality, assume start(α) ≤ start(β). The adjacency predicate has two disjuncts; under this assumption the disjunct reach(β) = start(α) is vacuous, since it would give reach(β) = start(α) ≤ start(β) < reach(β), i.e. reach(β) < reach(β). Overlap-or-adjacency therefore reduces to reach(α) = start(β) (adjacency) or reach(α) > start(β) (overlap), whence reach(α) ≥ start(β). Define:

  s = start(α) = min(start(α), start(β))
  r = max(reach(α), reach(β))

Then ⟦α⟧ ∪ ⟦β⟧ = {t : s ≤ t < r}. To verify the union: every position in ⟦α⟧ satisfies s ≤ t (since s = start(α)) and t < r (since reach(α) ≤ r). Every position in ⟦β⟧ satisfies s ≤ t (since start(β) ≥ start(α) = s) and t < r (since reach(β) ≤ r). Conversely, take any t with s ≤ t < r. Two cases arise. *Case 1:* t < reach(α). Then start(α) = s ≤ t < reach(α), so t ∈ ⟦α⟧. *Case 2:* t ≥ reach(α). Since t < r = max(reach(α), reach(β)) and t ≥ reach(α), we have r > reach(α), which forces r = reach(β). Then t < reach(β). The overlap/adjacency condition gives t ≥ reach(α) ≥ start(β), so start(β) ≤ t < reach(β), giving t ∈ ⟦β⟧. Every t ∈ [s, r) falls in ⟦α⟧ ∪ ⟦β⟧.

The merged span γ = (s, r ⊖ s) denotes {t : s ≤ t < r}. Level-uniformity and S6 ensure #s = #r (both are starts or reaches of level-uniform spans at the same length), and s < r since the union is non-empty, so by WF the pair γ = (s, r ⊖ s) is a well-formed level-uniform span with reach(γ) = r. The denotation depends only on the endpoints s and r, not on the history of how they were obtained.  ∎

A concrete instance (reusing S1's spans): let α = ([1, 3], [0, 4]) and β = ([1, 5], [0, 6]). Then reach(α) = [1, 7] and reach(β) = [1, 11]. Since start(α) = [1, 3] ≤ start(β) = [1, 5] and reach(α) = [1, 7] > start(β) = [1, 5], the spans overlap. We have s = [1, 3] and r = max([1, 7], [1, 11]) = [1, 11]. The merged span is γ = ([1, 3], [1, 11] ⊖ [1, 3]) = ([1, 3], [0, 8]) — divergence at position 2 gives 11 − 3 = 8. Verify: reach(γ) = [1, 3] ⊕ [0, 8] = [1, 11]. And ⟦α⟧ ∪ ⟦β⟧ = {t : [1, 3] ≤ t < [1, 7]} ∪ {t : [1, 5] ≤ t < [1, 11]} = {t : [1, 3] ≤ t < [1, 11]} = ⟦γ⟧ — the overlap region [1, 5]..[1, 7) is covered by both spans, and the union fills the interval without gaps.

Nelson grounds this in the normalization guarantee: "A spanset may be presented to the back end with any degree of overlap among the spans. This is because the system in effect performs a boolean OR to create a normalized specset, i.e. a non-overlapping coverage of the same portion of tumbler-space" (LM 4/37, Q3). The system treats {[a, b], [b, c]} and {[a, c]} as equivalent representations of the same address range.

**S3a** (*MergeCommutativity*). The merge of α and β yields the same span as the merge of β and α: ⟦α⟧ ∪ ⟦β⟧ = ⟦β⟧ ∪ ⟦α⟧. This follows from set union being commutative.


## Split

Splitting is the reverse of merging: given a span σ and a point interior to it, decompose σ into two adjacent parts that together reconstitute the original.

**Definition** (*Interior point*). A position p is *interior* to span σ when start(σ) < p < reach(σ). By the definition of ⟦σ⟧ = {t : start(σ) ≤ t < reach(σ)}, every interior point is in ⟦σ⟧.

**S4** (*SplitPartition*). For a level-uniform span σ = (s, ℓ) and an interior point p with level_compat(s, p), the displacements d = p ⊖ s and d' = reach(σ) ⊖ p are well-defined with #d = #s = #d' (all tumblers at the same length). The left span λ = (s, d) and right span ρ = (p, d') satisfy:

  (a) ⟦λ⟧ ∪ ⟦ρ⟧ = ⟦σ⟧                  (nothing lost)
  (b) ⟦λ⟧ ∩ ⟦ρ⟧ = ∅                      (nothing duplicated)
  (c) reach(λ) = start(ρ) = p             (the parts are adjacent)

*Proof.* First we verify T12 for both constructed spans. For λ = (s, d) where d = p ⊖ s: since s < p and #s = #p, WF gives a well-formed level-uniform span with reach(λ) = p. For ρ = (p, d') where d' = reach(σ) ⊖ p: since p < reach(σ) and #p = #reach(σ) (level-uniformity gives #reach = #s = #p), WF gives a well-formed level-uniform span with reach(ρ) = reach(σ).

(a): ⟦λ⟧ ∪ ⟦ρ⟧ = {t : s ≤ t < p} ∪ {t : p ≤ t < reach(σ)} = {t : s ≤ t < reach(σ)} = ⟦σ⟧.

(b): ⟦λ⟧ ∩ ⟦ρ⟧ = {t : s ≤ t < p ∧ p ≤ t} = ∅, since t < p and t ≥ p cannot both hold.

(c): Since #s = #p (level compatibility) and s < p, the divergence is of type (i) with divergence(s, p) ≤ #s — equal length excludes the prefix case — so D1's preconditions (s < p, divergence(s, p) ≤ #s, #s ≤ #p) are met and D1 gives s ⊕ (p ⊖ s) = p. So reach(λ) = s ⊕ d = p = start(ρ).  ∎

A concrete instance: let σ = ([1, 0, 1, 0, 1, 0, 5], [0, 0, 0, 0, 0, 0, 8]), a level-uniform span with #s = #ℓ = 7. The action point is k = 7, giving reach = [1, 0, 1, 0, 1, 0, 13]. Split at p = [1, 0, 1, 0, 1, 0, 9], which is interior (s < p < reach at position 7) and level-compatible (#p = 7 = #s).

We compute d = p ⊖ s: divergence at position 7 (9 vs 5), so d = [0, 0, 0, 0, 0, 0, 4]. And d' = reach ⊖ p: divergence at position 7 (13 vs 9), so d' = [0, 0, 0, 0, 0, 0, 4]. Both have length 7 = #s.

The split parts: λ = ([1, 0, 1, 0, 1, 0, 5], [0, 0, 0, 0, 0, 0, 4]) and ρ = ([1, 0, 1, 0, 1, 0, 9], [0, 0, 0, 0, 0, 0, 4]).

Verify S4: (a) ⟦λ⟧ ∪ ⟦ρ⟧ = {t : [.., 5] ≤ t < [.., 9]} ∪ {t : [.., 9] ≤ t < [.., 13]} = {t : [.., 5] ≤ t < [.., 13]} = ⟦σ⟧. (b) ⟦λ⟧ ∩ ⟦ρ⟧ = ∅ (t < [.., 9] vs t ≥ [.., 9]). (c) reach(λ) = [.., 5] ⊕ [.., 4] = [.., 9] = p = start(ρ).

Each element of ⟦σ⟧ appears in exactly one of ⟦λ⟧ or ⟦ρ⟧ — those before p go left, those from p onward go right. The partition is forced by the total order; there is no ambiguity. Nelson confirms the structural basis: "each element occupies exactly one position on the tumbler line" and spans include "everything between their endpoints with no discretion" (Q2).

**S5** (*SplitWidthComposition*). Under the same conditions as S4, the widths of the two parts compose to the original width:

  d ⊕ d' = ℓ

*Proof.* By D1, s ⊕ d = p (since s < p, #s = #d = #p, and equal length forces divergence(s, p) ≤ #s by excluding the prefix case). By D1 again, p ⊕ d' = reach(σ) (since p < reach(σ), #p = #d' = #reach, and equal length likewise forces divergence(p, reach(σ)) ≤ #p). Chaining:

  (s ⊕ d) ⊕ d' = reach(σ) = s ⊕ ℓ

We discharge TA-assoc's preconditions (Pos(d), Pos(d'), k_d ≤ #s, k_{d'} ≤ #d, ASN-0034):

- *Pos(d)*: T12 on λ gives d > 0.
- *Pos(d')*: T12 on ρ gives d' > 0.
- *k_d ≤ #s*: T12 on λ bounds the action point of d by #s.
- *k_{d'} ≤ #d*: T12 on ρ gives k_{d'} ≤ #p = #s, and level-uniformity of λ gives #d = #s, so k_{d'} ≤ #d.

TA-assoc applies, yielding (i) associativity (s ⊕ d) ⊕ d' = s ⊕ (d ⊕ d') with both sides well-defined; (ii) Pos(d ⊕ d'); (iii) actionPoint(d ⊕ d') = min(k_d, k_{d'}). From the chain,

  s ⊕ (d ⊕ d') = s ⊕ ℓ

We discharge TA-LC's preconditions (TA-LC, ASN-0034) with a := s, x := d ⊕ d', y := ℓ:

- *Pos(d ⊕ d')*: consequence (ii) of TA-assoc.
- *Pos(ℓ)*: T12 on σ.
- *actionPoint(d ⊕ d') ≤ #s*: consequence (iii) gives actionPoint(d ⊕ d') = min(k_d, k_{d'}) ≤ k_d ≤ #s, the last bound being T12 on λ discharged above.
- *actionPoint(ℓ) ≤ #s*: T12 on σ.
- *s ⊕ (d ⊕ d') = s ⊕ ℓ*: the chain above.

TA-LC gives:

  d ⊕ d' = ℓ.  ∎

Continuing the S4 worked instance (σ = ([1, 0, 1, 0, 1, 0, 5], [0, 0, 0, 0, 0, 0, 8]) split at p = [1, 0, 1, 0, 1, 0, 9], giving d = d' = [0, 0, 0, 0, 0, 0, 4]): d ⊕ d' = [0, 0, 0, 0, 0, 0, 4] ⊕ [0, 0, 0, 0, 0, 0, 4]. Action point k = 7: 4 + 4 = 8. Result = [0, 0, 0, 0, 0, 0, 8] = ℓ.

**S4a** (*SplitMergeInverse*). For a level-uniform span σ = (s, ℓ) and an interior point p with level_compat(s, p), splitting σ at p (S4) and merging the two parts (S3) recovers σ exactly.

*Proof.* The split produces λ = (s, d) with reach(λ) = p, and ρ = (p, d') with reach(ρ) = reach(σ). Since reach(λ) = start(ρ), the two parts are adjacent, and S3 applies. The merge constructs γ = (s_m, r_m ⊖ s_m) where s_m = min(s, p) = s (since s < p) and r_m = max(p, reach(σ)) = reach(σ) (since p < reach(σ)). The merged width is reach(σ) ⊖ s = reach(σ) ⊖ start(σ) = ℓ, by WR (σ is level-uniform). So γ = (s, ℓ) = σ — the original span is recovered exactly.  ∎

**S3b** (*MergeSplitInverse*). For adjacent level-uniform spans α and β with level_compat(start(α), start(β)), merging α and β (S3) and splitting the result at the shared boundary (S4) recovers the unordered pair {α, β} exactly: the split yields a left part λ and a right part ρ with {λ, ρ} = {α, β}. The assignment of α and β to the left/right positions is determined by the adjacency direction: in Case A (reach(α) = start(β)), λ = α and ρ = β; in Case B (reach(β) = start(α)), λ = β and ρ = α.

*Proof.* Adjacency means reach(α) = start(β) or reach(β) = start(α). We handle each disjunct.

*Case A: reach(α) = start(β).* The merge produces γ = (start(α), r ⊖ start(α)) where r = max(reach(α), reach(β)) = reach(β), since reach(α) = start(β) < reach(β) (β is non-empty). So γ = (start(α), reach(β) ⊖ start(α)) with reach(γ) = reach(β). The shared boundary p = start(β) is interior to γ: start(α) < start(β) (since α is non-empty, start(α) < reach(α) = start(β)) and start(β) < reach(β) = reach(γ) (since β is non-empty). Level compatibility holds by assumption.

Splitting γ at p yields λ = (start(α), p ⊖ start(α)) and ρ = (p, reach(γ) ⊖ p). For λ: p ⊖ start(α) = reach(α) ⊖ start(α) = width(α) by WR (α is level-uniform). So λ = (start(α), width(α)) = α. For ρ: reach(γ) ⊖ p = reach(β) ⊖ start(β) = width(β) by WR (β is level-uniform). So ρ = (start(β), width(β)) = β.

*Case B: reach(β) = start(α).* By S3a (merge commutativity) the merge of α and β equals the merge of β and α, which is the Case A configuration with the roles of α and β exchanged. Applying Case A to the pair ⟨β, α⟩, splitting the merged span at the shared boundary start(α) yields left part λ = β and right part ρ = α. The unordered pair {λ, ρ} = {β, α} = {α, β} is recovered exactly; the left-right assignment is reversed relative to Case A.  ∎


## Span-sets

A *span-set* is a finite sequence of spans Σ = ⟨σ₁, σ₂, ..., σₙ⟩. Its denotation is the union:

  ⟦Σ⟧ = ⟦σ₁⟧ ∪ ⟦σ₂⟧ ∪ ... ∪ ⟦σₙ⟧

Two span-sets are *equivalent* when they denote the same set of positions: Σ₁ ≡ Σ₂ ⟺ ⟦Σ₁⟧ = ⟦Σ₂⟧. The empty span-set ⟨⟩ denotes ∅. The singleton span-set ⟨σ⟩ denotes ⟦σ⟧. For span-sets Σ₁ = ⟨α₁, ..., αₘ⟩ and Σ₂ = ⟨β₁, ..., βₙ⟩, the *union* Σ₁ ∪ Σ₂ is the concatenated sequence ⟨α₁, ..., αₘ, β₁, ..., βₙ⟩; by the denotation definition, ⟦Σ₁ ∪ Σ₂⟧ = ⟦Σ₁⟧ ∪ ⟦Σ₂⟧.

**S7** (*CoveringExistence*). Every finite set of positions P ⊂ T admits a covering span-set Σ with |Σ| = |P| and ⟦Σ⟧ ⊇ P. This is a *covering* claim, not an exact representation: in general no span-set Σ satisfies ⟦Σ⟧ = P for an arbitrary finite P.

*Proof.* For any tumbler t, define ℓ = [0, ..., 0, 1] with #ℓ = #t (all components zero except the last, which is 1). Then ℓ > 0 (the last component is nonzero) and the action point k = #t ≤ #t, so (t, ℓ) satisfies T12. By TA-strict, t ⊕ ℓ > t, so t ∈ [t, t ⊕ ℓ) = ⟦(t, ℓ)⟧ — the span covers t. Taking one such span per position in P gives Σ with |Σ| = |P| and ⟦Σ⟧ ⊇ P.

*Why exact representation fails in general.* Every span denotes an *infinite* set, so no non-empty finite P can be denoted exactly. Fix a span (s, ℓ) and let k = actionPoint(ℓ); by T12, k ≤ #s and ℓₖ ≠ 0, and by TumblerAdd reach(s, ℓ)ₖ = sₖ + ℓₖ > sₖ while reach agrees with s on positions 1..k−1. Consider the proper deeper extensions s.0, s.0.0, s.0.0.0, … — each appends one or more trailing zeros to s. Every such extension e agrees with s on all positions 1..#s, so e > s by the prefix convention (T1 case (ii)), and at the divergence position k we have eₖ = sₖ < sₖ + ℓₖ = reach(s, ℓ)ₖ with agreement on positions 1..k−1, so e < reach(s, ℓ) by T1 case (i). Thus every extension lies in [s, reach(s, ℓ)) = ⟦(s, ℓ)⟧, and by T0(b) there are infinitely many of them. Hence ⟦σ⟧ is infinite for every span σ, and ⟦Σ⟧ is infinite for every non-empty span-set Σ. No non-empty finite P can satisfy ⟦Σ⟧ = P — the obstruction is the finite-vs-infinite mismatch, independent of P's internal shape. The inclusion ⟦Σ⟧ ⊇ P is therefore the strongest finite guarantee available.

Nelson confirms the covering reach: "a tumbler-span may range in possible size from one byte to the whole docuverse" (LM 4/24, Q4).


## Normalization

A span-set is *normalized* when its components are sorted, non-overlapping, and non-adjacent:

**Definition** (*Normalized span-set*). A span-set Σ = ⟨σ₁, ..., σₙ⟩ is normalized iff:

  (N1) *Sorted.* `(A i : 1 ≤ i < n : start(σᵢ) < start(σᵢ₊₁))`
  (N2) *Separated.* `(A i : 1 ≤ i < n : reach(σᵢ) < start(σᵢ₊₁))`

Condition N2 uses strict inequality. If reach(σᵢ) = start(σᵢ₊₁), the spans are adjacent and could be merged — so the form is not yet minimal. If reach(σᵢ) > start(σᵢ₊₁), the spans overlap and must be merged. The normalized form is the irreducible representation: every span is as large as it can be, and no two spans can be combined.

**Definition** (*Mutually level-compatible*). A span-set Σ = ⟨σ₁, ..., σₙ⟩ is *mutually level-compatible* when level_compat(start(σᵢ), start(σⱼ)) holds for all 1 ≤ i, j ≤ n. By S6, this is equivalent to: there exists a single length L with #start(σᵢ) = L for every i. When each component σᵢ is also level-uniform, all boundary tumblers of every span — start(σᵢ), width(σᵢ), reach(σᵢ) — share the common length L, so any pair of distinct endpoints a < b drawn from any pair of spans has #a = #b.

**S8** (*NormalizationExistence*). Every span-set Σ whose component spans are level-uniform and mutually level-compatible has a normalized equivalent Σ̂ with Σ̂ ≡ Σ.

*Construction.* If n = 0, the result is the empty span-set ⟨⟩, which vacuously satisfies N1 and N2. For n ≥ 1, proceed as follows. Sort the component spans into non-decreasing order of start position; T1 totally orders tumblers, but distinct spans may share a start (SC cases (iv) and (v)), so any ties are broken arbitrarily; any resulting normalized equivalent satisfies N1/N2 with ⟦Σ̂⟧ = ⟦Σ⟧, which is all S8 requires. Seed the current interval [s, r) = [start(σ₁), reach(σ₁)) from the first span in sorted order, with the emitted set E = ∅. Then scan σ₂, ..., σₙ left to right. For each span σᵢ with i ≥ 2:

  — If start(σᵢ) ≤ r (overlap or adjacency): extend r to max(r, reach(σᵢ)).
  — If start(σᵢ) > r (separated): emit the current interval as a span (s, r ⊖ s). Level-uniformity and S6 ensure #s = #r, and s < r (the current interval was initialized from a non-empty span and is only extended by the merge step), so by WF the emitted pair (s, r ⊖ s) is a well-formed level-uniform span with reach r. Then start a new current interval at [start(σᵢ), reach(σᵢ)).

After processing all spans, emit the final interval — WF applies identically (the final interval is non-empty because it was initialized from a non-empty span).

*Loop invariant.* Let E be the set of emitted spans after processing σ₁..σᵢ, and [s, r) the current interval. The invariant J is:

  J: ⟦E⟧ ∪ [s, r) = ⟦σ₁⟧ ∪ ... ∪ ⟦σᵢ⟧

*Initialization.* After the first span σ₁, E = ∅ and [s, r) = [start(σ₁), reach(σ₁)) = ⟦σ₁⟧. J holds.

*Merge step.* When start(σᵢ) ≤ r, the new interval [s, max(r, reach(σᵢ))) covers [s, r) ∪ [start(σᵢ), reach(σᵢ)). The first term is the old current interval; the second is ⟦σᵢ⟧. Since start(σᵢ) ≤ r ensures no gap, the union is [s, max(r, reach(σᵢ))). E is unchanged, so ⟦E⟧ ∪ [s, max(r, reach(σᵢ))) = ⟦E⟧ ∪ [s, r) ∪ ⟦σᵢ⟧. By the inductive hypothesis, this equals ⟦σ₁⟧ ∪ ... ∪ ⟦σᵢ⟧. J is preserved.

*Emit step.* When start(σᵢ) > r, the current interval [s, r) is emitted and a new interval [start(σᵢ), reach(σᵢ)) begins. The emitted span covers exactly [s, r), so ⟦E'⟧ = ⟦E⟧ ∪ [s, r). The new current interval is ⟦σᵢ⟧. Then ⟦E'⟧ ∪ ⟦σᵢ⟧ = ⟦E⟧ ∪ [s, r) ∪ ⟦σᵢ⟧ = ⟦σ₁⟧ ∪ ... ∪ ⟦σᵢ⟧. J is preserved.

*Finalization.* After all n spans, emit the final [s, r). The total output satisfies ⟦Σ̂⟧ = ⟦σ₁⟧ ∪ ... ∪ ⟦σₙ⟧ = ⟦Σ⟧.

The result is a sequence of spans satisfying N1 and N2. *N2 (separated reaches):* each emit occurs precisely when start(σᵢ) > r, so the emitted interval's reach r lies strictly below the start of the next interval; consecutive emitted spans are therefore separated. *N1 (sorted starts):* sortedness of the input alone yields only ≤ on starts — equal-start inputs are admitted (SC cases (iv), (v); the sort breaks ties arbitrarily) — so it does not establish N1's strict inequality. The strictness comes instead from the emit condition. A new interval opens only at a span with start(σᵢ) > r, where r is the reach of the interval just emitted, and that emitted interval's start s satisfies s < r (the interval is non-empty). Hence the next emitted start, which is start(σᵢ), satisfies start(σᵢ) > r > s — each emitted start strictly exceeds the previous emitted reach, and a fortiori the previous emitted start. So consecutive emitted starts are strictly increasing.

*Termination.* The scan visits each of the n input spans exactly once — bound function t = n − i.  ∎

A concrete instance exercises both branches. Take the span-set Σ = ⟨σ₁, σ₂, σ₃⟩ with σ₁ = ([1, 7], [0, 2]), σ₂ = ([1, 3], [0, 5]), σ₃ = ([1, 10], [0, 3]). The reaches are reach(σ₁) = [1, 9], reach(σ₂) = [1, 8], reach(σ₃) = [1, 13]. All spans are level-uniform with #start = #width = 2.

*Sort by start.* The sorted order is ⟨σ₂, σ₁, σ₃⟩ = ⟨([1, 3], [0, 5]), ([1, 7], [0, 2]), ([1, 10], [0, 3])⟩.

*Initialize.* Set [s, r) = [[1, 3], [1, 8]), E = ∅. J holds: ∅ ∪ {t : [1, 3] ≤ t < [1, 8]} = ⟦σ₂⟧.

*Process σ₁.* start(σ₁) = [1, 7] ≤ r = [1, 8] — merge branch. Update r = max([1, 8], [1, 9]) = [1, 9]. Current interval becomes [[1, 3], [1, 9]). J: ∅ ∪ {t : [1, 3] ≤ t < [1, 9]} = ⟦σ₂⟧ ∪ ⟦σ₁⟧ — the overlap region {t : [1, 7] ≤ t < [1, 8]} is covered by both spans, and [1, 9] extends one position beyond reach(σ₂).

*Process σ₃.* start(σ₃) = [1, 10] > r = [1, 9] — emit branch. Emit (s, r ⊖ s) = ([1, 3], [1, 9] ⊖ [1, 3]) = ([1, 3], [0, 6]). Verify T12: divergence at position 2 gives width component 9 − 3 = 6 > 0, action point k = 2 ≤ #s = 2. Verify reach: [1, 3] ⊕ [0, 6] = [1, 9]. Start new interval [[1, 10], [1, 13]). J: {t : [1, 3] ≤ t < [1, 9]} ∪ {t : [1, 10] ≤ t < [1, 13]} = ⟦σ₂⟧ ∪ ⟦σ₁⟧ ∪ ⟦σ₃⟧.

*Finalize.* Emit ([1, 10], [1, 13] ⊖ [1, 10]) = ([1, 10], [0, 3]). Verify reach: [1, 10] ⊕ [0, 3] = [1, 13].

Result: Σ̂ = ⟨([1, 3], [0, 6]), ([1, 10], [0, 3])⟩. N1: [1, 3] < [1, 10]. N2: reach of first = [1, 9] < [1, 10] = start of second. The three original spans — two overlapping, one separated — reduce to two disjoint spans.

**S9** (*NormalizationUniqueness*). The normalized form is unique: if Σ̂₁ and Σ̂₂ are both normalized and Σ̂₁ ≡ Σ̂₂, then Σ̂₁ = Σ̂₂.

*Proof.* Let Σ̂₁ = ⟨α₁, ..., αₘ⟩ and Σ̂₂ = ⟨β₁, ..., βₙ⟩, both normalized, with ⟦Σ̂₁⟧ = ⟦Σ̂₂⟧ = S. Suppose Σ̂₁ ≠ Σ̂₂. Let i be the smallest index where αᵢ ≠ βᵢ (if one sequence is shorter, take i past the shorter one's end). For j < i, αⱼ = βⱼ.

The configuration start(αᵢ) = start(βᵢ) ∧ reach(αᵢ) = reach(βᵢ) cannot occur at a divergence index: two spans sharing start and reach share width, since start ⊕ w₁ = reach = start ⊕ w₂ forces w₁ = w₂ by left cancellation (TA-LC, ASN-0034), whence αᵢ = βᵢ — contradicting that i is a divergence index. The case split below is therefore exhaustive: any genuine divergence at i differs in start or in reach.

*Case 1a:* Both αᵢ and βᵢ exist, with start(αᵢ) < start(βᵢ). Then start(αᵢ) ∈ S since start(αᵢ) ∈ ⟦αᵢ⟧. But start(αᵢ) ∉ ⟦βⱼ⟧ for any j. For j < i: αⱼ = βⱼ by minimality of i, so reach(βⱼ) = reach(αⱼ). N2 on Σ̂₁ gives reach(αⱼ) < start(αⱼ₊₁), and repeated N1 on Σ̂₁ gives start(αⱼ₊₁) ≤ start(αᵢ) (with equality when j+1 = i, strict otherwise); chaining, reach(βⱼ) < start(αᵢ), so start(αᵢ) ∉ ⟦βⱼ⟧. For j ≥ i: N1 on Σ̂₂ gives start(βⱼ) ≥ start(βᵢ), and the case hypothesis gives start(βᵢ) > start(αᵢ); so start(βⱼ) > start(αᵢ), and start(αᵢ) ∉ ⟦βⱼ⟧. So start(αᵢ) ∉ ⟦Σ̂₂⟧ = S. Contradiction.

*Case 1b:* αᵢ exists, βᵢ does not exist (i.e., n < i ≤ m, so Σ̂₂ is the shorter sequence with n = i − 1). Then start(αᵢ) ∈ S since start(αᵢ) ∈ ⟦αᵢ⟧. The range j ≥ i is vacuous (no such βⱼ exists). For j < i: αⱼ = βⱼ by minimality of i, so reach(βⱼ) = reach(αⱼ); N2 on Σ̂₁ gives reach(αⱼ) < start(αⱼ₊₁), and repeated N1 on Σ̂₁ gives start(αⱼ₊₁) ≤ start(αᵢ); chaining, reach(βⱼ) < start(αᵢ), so start(αᵢ) ∉ ⟦βⱼ⟧. Since all βⱼ are exhausted by j < i, start(αᵢ) ∉ ⟦Σ̂₂⟧ = S. Contradiction.

*Case 2:* start(αᵢ) = start(βᵢ) but reach(αᵢ) ≠ reach(βᵢ). Since the inequality is strict in exactly one direction, two sub-cases arise.

*Case 2a:* reach(αᵢ) < reach(βᵢ). Set p = reach(αᵢ). Then p ∈ ⟦βᵢ⟧ since start(βᵢ) = start(αᵢ) < reach(αᵢ) = p < reach(βᵢ), so p ∈ S. But p ∉ ⟦αᵢ⟧ since p = reach(αᵢ) is the exclusive upper bound. For j < i, p ∉ ⟦αⱼ⟧ since p = reach(αᵢ) > reach(αⱼ) by N2 (reach(αⱼ) < start(αⱼ₊₁)), repeated application of N1 (start(αⱼ₊₁) < ... < start(αᵢ)), and non-emptiness (start(αᵢ) < reach(αᵢ)). For j > i, p ∉ ⟦αⱼ⟧ since p = reach(αᵢ) < start(αᵢ₊₁) ≤ start(αⱼ) by N2 and N1. So p ∉ ⟦Σ̂₁⟧, but p ∈ S. Contradiction.

*Case 2b:* reach(αᵢ) > reach(βᵢ). Symmetric to Case 2a, exchanging the roles of Σ̂₁ and Σ̂₂. Set p = reach(βᵢ). Then p ∈ ⟦αᵢ⟧ since start(αᵢ) = start(βᵢ) < reach(βᵢ) = p < reach(αᵢ), so p ∈ S. But p ∉ ⟦βᵢ⟧ since p = reach(βᵢ) is the exclusive upper bound. For j < i, p ∉ ⟦βⱼ⟧ since p = reach(βᵢ) > reach(βⱼ) by N2 on Σ̂₂, repeated N1, and non-emptiness. For j > i, p ∉ ⟦βⱼ⟧ since p = reach(βᵢ) < start(βᵢ₊₁) ≤ start(βⱼ) by N2 and N1 on Σ̂₂. So p ∉ ⟦Σ̂₂⟧, but p ∈ S. Contradiction.

*Case 3a:* Both αᵢ and βᵢ exist, with start(αᵢ) > start(βᵢ). Symmetric to Case 1a, exchanging the roles of Σ̂₁ and Σ̂₂: start(βᵢ) ∈ S since start(βᵢ) ∈ ⟦βᵢ⟧. For j < i: βⱼ = αⱼ by minimality of i, so reach(αⱼ) = reach(βⱼ); N2 on Σ̂₂ and repeated N1 give reach(αⱼ) = reach(βⱼ) < start(βᵢ), so start(βᵢ) ∉ ⟦αⱼ⟧. For j ≥ i: N1 on Σ̂₁ gives start(αⱼ) ≥ start(αᵢ), and the case hypothesis gives start(αᵢ) > start(βᵢ); so start(βᵢ) ∉ ⟦αⱼ⟧. So start(βᵢ) ∉ ⟦Σ̂₁⟧ = S. Contradiction.

*Case 3b:* βᵢ exists, αᵢ does not exist (i.e., m < i ≤ n, so Σ̂₁ is the shorter sequence with m = i − 1). Symmetric to Case 1b: start(βᵢ) ∈ S since start(βᵢ) ∈ ⟦βᵢ⟧. The range j ≥ i is vacuous (no such αⱼ exists). For j < i: βⱼ = αⱼ, so reach(αⱼ) = reach(βⱼ); N2 on Σ̂₂ and repeated N1 give reach(αⱼ) = reach(βⱼ) < start(βᵢ), so start(βᵢ) ∉ ⟦αⱼ⟧. Since all αⱼ are exhausted by j < i, start(βᵢ) ∉ ⟦Σ̂₁⟧ = S. Contradiction.

All cases yield contradiction, so Σ̂₁ = Σ̂₂.  ∎

Nelson confirms the uniqueness: "Each run yields exactly one span... The minimal span-set is unique" (Q4).


## Union is order-independent

**S10** (*UnionOrderIndependence*). For span-sets Σ₁, Σ₂ whose component spans are level-uniform and mutually level-compatible across both sets, the normalized form of their union is independent of the order in which spans are combined:

  normalize(Σ₁ ∪ Σ₂) = normalize(Σ₂ ∪ Σ₁)                  (commutativity)

For span-sets Σ₁, Σ₂, Σ₃ whose component spans are level-uniform and mutually level-compatible across all three sets:

  normalize((Σ₁ ∪ Σ₂) ∪ Σ₃) = normalize(Σ₁ ∪ (Σ₂ ∪ Σ₃))    (associativity)

*Proof.* ⟦Σ₁ ∪ Σ₂⟧ = ⟦Σ₁⟧ ∪ ⟦Σ₂⟧ = ⟦Σ₂⟧ ∪ ⟦Σ₁⟧ = ⟦Σ₂ ∪ Σ₁⟧. Let A = normalize(Σ₁ ∪ Σ₂) and B = normalize(Σ₂ ∪ Σ₁). The hypothesis that the component spans are level-uniform and mutually level-compatible across both sets is preserved under union (concatenation introduces no new spans), so S8 applies to each union span-set: A is a normalized equivalent of Σ₁ ∪ Σ₂ and B a normalized equivalent of Σ₂ ∪ Σ₁, giving ⟦A⟧ = ⟦Σ₁ ∪ Σ₂⟧ = ⟦Σ₂ ∪ Σ₁⟧ = ⟦B⟧. Both A and B are normalized with equal denotation, so by S9 (uniqueness) A = B. Associativity follows identically — S8 supplies a normalized equivalent for each grouping (the level hypotheses hold across all three sets), S9 forces the two to coincide, and the underlying denotations agree by associativity of set union.  ∎

The set-theoretic semantics — span-sets denote byte collections, not ordered series — is Nelson's intent (Q8): "what matters is which bytes are designated, not the order of the series."


## Difference

When one span contains another, the remainder is always bounded:

**S11** (*DifferenceBound*). For level-uniform spans α and β with level_compat(start(α), start(β)) and ⟦β⟧ ⊆ ⟦α⟧, the set difference ⟦α⟧ \ ⟦β⟧ is expressible as a span-set of at most two spans.

*Proof.* We first derive the boundary characterization of containment: ⟦β⟧ ⊆ ⟦α⟧ implies start(α) ≤ start(β) and reach(β) ≤ reach(α). For the start: start(β) ∈ ⟦β⟧ ⊆ ⟦α⟧ gives start(α) ≤ start(β) (and, since start(β) ∈ ⟦α⟧, also start(β) < reach(α)). For the reach: suppose for contradiction reach(β) > reach(α). Then start(β) < reach(α) < reach(β), so reach(α) ∈ ⟦β⟧ ⊆ ⟦α⟧, whence reach(α) < reach(α) — contradiction. Hence reach(β) ≤ reach(α). We now derive the decomposition by element-chasing. Let t ∈ ⟦α⟧, i.e., start(α) ≤ t < reach(α). The total order T1 splits this range into three sub-ranges relative to β's endpoints:

  (L) start(α) ≤ t < start(β)
  (M) start(β) ≤ t < reach(β)
  (R) reach(β) ≤ t < reach(α)

These three sub-ranges are exhaustive and pairwise disjoint by T1's totality: compare t with start(β) — if t < start(β) then (L); else t ≥ start(β), and compare t with reach(β) — if t < reach(β) then (M), else t ≥ reach(β) combined with t < reach(α) gives (R). In sub-range (M), t ∈ ⟦β⟧ by definition of ⟦β⟧. In sub-ranges (L) and (R), t ∉ ⟦β⟧: for (L), t < start(β) is below β's start; for (R), t ≥ reach(β) is at or above β's exclusive upper bound. Therefore ⟦α⟧ \ ⟦β⟧ = (L) ∪ (R):

  Left:   {t : start(α) ≤ t < start(β)}      (empty when start(α) = start(β))
  Right:  {t : reach(β) ≤ t < reach(α)}       (empty when reach(β) = reach(α))

We construct the spans explicitly. For the left interval, when start(α) < start(β), define λ = (start(α), start(β) ⊖ start(α)). Since start(α) < start(β) and #start(α) = #start(β) (level-compatibility), WF gives a well-formed level-uniform span with reach(λ) = start(β).

For the right interval, when reach(β) < reach(α), define ρ = (reach(β), reach(α) ⊖ reach(β)). Since reach(β) < reach(α) and #reach(β) = #reach(α) (level-uniformity and level-compatibility ensure all boundary tumblers share the same length), WF gives a well-formed level-uniform span with reach(ρ) = reach(α).

The result is a span-set of 0, 1, or 2 components:

  (a) Both boundaries coincide (α = β): difference is empty — 0 spans.
  (b) One boundary coincides: difference is one span.
  (c) Neither coincides: difference is two spans.

The bound of two is tight in case (c). When neither boundary coincides, start(α) < start(β) and reach(β) < reach(α), so reach(λ) = start(β) and start(ρ) = reach(β), giving reach(λ) < start(ρ) (since β is non-empty by S2, start(β) < reach(β)). Suppose for contradiction that a single span γ satisfies ⟦γ⟧ = ⟦λ⟧ ∪ ⟦ρ⟧. Pick any t ∈ ⟦β⟧, non-empty by S2. Then start(α) ∈ ⟦λ⟧ ⊆ ⟦γ⟧ and reach(β) ∈ ⟦ρ⟧ ⊆ ⟦γ⟧, and start(α) < start(β) ≤ t < reach(β) places t between two members of ⟦γ⟧. By S0 (convexity), t ∈ ⟦γ⟧ = ⟦λ⟧ ∪ ⟦ρ⟧. But t ∉ ⟦λ⟧ (since t ≥ start(β) = reach(λ)) and t ∉ ⟦ρ⟧ (since t < reach(β) = start(ρ)) — contradiction. Therefore no single span can represent the result, and two is the minimum.  ∎

A concrete instance: let α = ([1, 3], [0, 8]) and β = ([1, 5], [0, 4]). Then reach(α) = [1, 11] and reach(β) = [1, 9]. Containment holds: [1, 3] ≤ [1, 5] and [1, 9] ≤ [1, 11]. The left difference span is λ = ([1, 3], [1, 5] ⊖ [1, 3]) = ([1, 3], [0, 2]); reach(λ) = [1, 3] ⊕ [0, 2] = [1, 5] = start(β). The right difference span is ρ = ([1, 9], [1, 11] ⊖ [1, 9]) = ([1, 9], [0, 2]); reach(ρ) = [1, 9] ⊕ [0, 2] = [1, 11] = reach(α). Verify: ⟦α⟧ = {t : [1, 3] ≤ t < [1, 11]} = ⟦λ⟧ ∪ ⟦β⟧ ∪ ⟦ρ⟧, so ⟦α⟧ \ ⟦β⟧ = ⟦λ⟧ ∪ ⟦ρ⟧ = {t : [1, 3] ≤ t < [1, 5]} ∪ {t : [1, 9] ≤ t < [1, 11]}.

Nelson confirms the bound and the mechanism: "Removing a contained span from a containing span always produces at most two contiguous spans, expressible as a span-set" (Q5). The system provides span-sets precisely for representing such non-contiguous remainders. The result is "a structural consequence of the tumbler line being linearly ordered, combined with the span-set mechanism for non-contiguous selections."


## Difference for separated and adjacent spans

**S11a** — *DifferenceSeparated* (LEMMA, lemma). For level-uniform spans α and β with level_compat(start(α), start(β)) in SC case (i) (separated) or (ii) (adjacent): ⟦α⟧ \ ⟦β⟧ = ⟦α⟧.

*Proof.* In both cases ⟦α⟧ ∩ ⟦β⟧ = ∅ (SC classifies (i) and (ii) as the disjoint cases). When the intersection is empty, removing β's positions from α removes nothing: ⟦α⟧ \ ⟦β⟧ = ⟦α⟧. The result is a span-set of exactly 1 span.  ∎

*Worked example.* Let α = ([1, 3], [0, 4]) and β = ([1, 10], [0, 2]). Then reach(α) = [1, 7] and start(β) = [1, 10]. Since [1, 7] < [1, 10], the spans are separated (SC case (i)). No position belongs to both spans, so ⟦α⟧ \ ⟦β⟧ = ⟦α⟧ = {t : [1, 3] ≤ t < [1, 7]}, a single span.


## Difference for equal spans

**S11b** — *DifferenceEqual* (LEMMA, lemma). For level-uniform spans α and β with level_compat(start(α), start(β)) in SC case (v) (equal): ⟦α⟧ \ ⟦β⟧ = ∅.

*Proof.* Equal spans have start(α) = start(β) and reach(α) = reach(β), so ⟦α⟧ = ⟦β⟧. The set difference of a set with itself is empty: ⟦α⟧ \ ⟦β⟧ = ∅. The result is a span-set of 0 spans.  ∎

*Worked example.* Let α = β = ([1, 3], [0, 4]). Then ⟦α⟧ \ ⟦β⟧ = ∅.


## Difference for proper overlap

**S11c** — *DifferenceOverlap* (LEMMA, lemma). For level-uniform spans α and β with level_compat(start(α), start(β)) in SC case (iii) (proper overlap): ⟦α⟧ \ ⟦β⟧ is expressible as a span-set of exactly 1 span.

*Proof.* SC case (iii) has two sub-cases. We first handle start(α) < start(β) < reach(α) < reach(β), then start(β) < start(α) < reach(β) < reach(α).

**Case 1:** start(α) < start(β) < reach(α) < reach(β). We derive the difference by element-chasing.

(⊆) For t ∈ ⟦α⟧ (i.e., start(α) ≤ t < reach(α)) with t ∉ ⟦β⟧: if t ≥ start(β), then start(β) ≤ t < reach(α) < reach(β), so t ∈ ⟦β⟧ — contradiction. Hence t < start(β), so t ∈ {t : start(α) ≤ t < start(β)}.

(⊇) For t with start(α) ≤ t < start(β): t ∈ ⟦α⟧ since start(α) ≤ t < start(β) < reach(α). And t ∉ ⟦β⟧ since t < start(β). So t ∈ ⟦α⟧ \ ⟦β⟧.

Therefore ⟦α⟧ \ ⟦β⟧ = {t : start(α) ≤ t < start(β)}. This is non-empty (start(α) < start(β) and start(α) ∈ ⟦α⟧ \ ⟦β⟧) and forms a single contiguous interval. We construct the span explicitly.

Define γ = (start(α), start(β) ⊖ start(α)). Since start(α) < start(β) and #start(α) = #start(β) (level-compatibility), WF gives a well-formed level-uniform span with reach(γ) = start(β).

The denotation ⟦γ⟧ = {t : start(α) ≤ t < start(β)} = ⟦α⟧ \ ⟦β⟧. The result is exactly 1 span.

*Worked example.* Let α = ([1, 3], [0, 7]) and β = ([1, 6], [0, 8]). Then reach(α) = [1, 10] and reach(β) = [1, 14]. Verify proper overlap: start(α) = [1, 3] < start(β) = [1, 6] < reach(α) = [1, 10] < reach(β) = [1, 14]. The difference ⟦α⟧ \ ⟦β⟧ = {t : [1, 3] ≤ t < [1, 6]}. Construct γ = ([1, 3], [1, 6] ⊖ [1, 3]) = ([1, 3], [0, 3]). Verify: reach(γ) = [1, 3] ⊕ [0, 3] = [1, 6] = start(β). Verify denotation: ⟦γ⟧ = {t : [1, 3] ≤ t < [1, 6]} = ⟦α⟧ \ ⟦β⟧.

**Case 2:** start(β) < start(α) < reach(β) < reach(α). We derive the difference by element-chasing. For t ∈ ⟦α⟧ (i.e., start(α) ≤ t < reach(α)): if t < reach(β), then start(β) < start(α) ≤ t and t < reach(β), so t ∈ ⟦β⟧; if t ≥ reach(β), then t ∉ ⟦β⟧ (since reach(β) is the exclusive upper bound of β). Therefore ⟦α⟧ \ ⟦β⟧ = {t : reach(β) ≤ t < reach(α)}.

Define γ' = (reach(β), reach(α) ⊖ reach(β)). We establish #reach(β) = #reach(α): level-uniformity of α gives #reach(α) = #start(α), level-uniformity of β gives #reach(β) = #start(β), and level_compat(start(α), start(β)) gives #start(α) = #start(β). With reach(β) < reach(α) given, WF gives a well-formed level-uniform span with reach(γ') = reach(α).

The denotation ⟦γ'⟧ = {t : reach(β) ≤ t < reach(α)} = ⟦α⟧ \ ⟦β⟧. The result is exactly 1 span.  ∎

*Worked example for Case 2.* Let β = ([1, 3], [0, 7]) and α = ([1, 6], [0, 8]). Then reach(β) = [1, 3] ⊕ [0, 7] = [1, 10] and reach(α) = [1, 6] ⊕ [0, 8] = [1, 14]. Verify Case 2 ordering: start(β) = [1, 3] < start(α) = [1, 6] < reach(β) = [1, 10] < reach(α) = [1, 14]. Element-chase: for t ∈ ⟦α⟧, i.e., [1, 6] ≤ t < [1, 14], split on whether t < reach(β) = [1, 10]. If t < [1, 10], then [1, 3] = start(β) < [1, 6] ≤ t < [1, 10] = reach(β), so t ∈ ⟦β⟧. If t ≥ [1, 10], then t ≥ reach(β), so t ∉ ⟦β⟧. Hence ⟦α⟧ \ ⟦β⟧ = {t : [1, 10] ≤ t < [1, 14]}. Construct γ' = (reach(β), reach(α) ⊖ reach(β)) = ([1, 10], [1, 14] ⊖ [1, 10]) = ([1, 10], [0, 4]) — divergence at position 2, width component 14 − 10 = 4. Verify reach(γ') = [1, 10] ⊕ [0, 4] = [1, 14] = reach(α). Verify denotation: ⟦γ'⟧ = {t : [1, 10] ≤ t < [1, 14]} = ⟦α⟧ \ ⟦β⟧. The result is exactly 1 span, as the lemma asserts.


## Unified difference bound

The four results combine into a single statement covering all SC cases:

**S11d** — *GeneralDifferenceBound* (LEMMA, lemma). For level-uniform spans α and β with level_compat(start(α), start(β)), the set difference ⟦α⟧ \ ⟦β⟧ is expressible as a span-set of at most 2 spans.

*Proof.* By SC, exactly one of five cases holds. For the reverse containment sub-case of SC(iv) — start(β) ≤ start(α) and reach(α) ≤ reach(β) with at least one strict — we derive ⟦α⟧ ⊆ ⟦β⟧: for t ∈ ⟦α⟧, start(β) ≤ start(α) ≤ t and t < reach(α) ≤ reach(β), so t ∈ ⟦β⟧. Hence the difference is empty.

| SC case | Difference | Bound | By |
|---------|-----------|-------|----|
| (i) Separated | ⟦α⟧ | 1 span | S11a |
| (ii) Adjacent | ⟦α⟧ | 1 span | S11a |
| (iii) Proper overlap | 1 span | 1 span | S11c |
| (iv) Containment (⟦β⟧ ⊂ ⟦α⟧) | at most 2 spans | 2 spans | S11 |
| (iv) Containment (⟦α⟧ ⊂ ⟦β⟧) | ∅ | 0 spans | ⟦α⟧ ⊆ ⟦β⟧ (derived above) |
| (v) Equal | ∅ | 0 spans | S11b |

The maximum across all cases is 2, achieved only in the containment case.  ∎

The bound of 2 is tight: S11 shows containment achieves it. No SC case exceeds it. This confirms Nelson's span-set mechanism is sufficient for representing any two-span difference.


## Properties Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| D0 | Displacement well-definedness: a < b and divergence(a, b) ≤ #a (DisplacementWellDefined, ASN-0034) | cited |
| D1 | Displacement round-trip: for a < b with divergence(a, b) ≤ #a and #a ≤ #b, a ⊕ (b ⊖ a) = b (DisplacementRoundTrip, ASN-0034) | cited |
| D2 | Displacement uniqueness: any w with a ⊕ w = b equals b ⊖ a (DisplacementUnique, ASN-0034) | cited |
| WR | Width recovery: for level-uniform σ, reach(σ) ⊖ start(σ) = width(σ) | introduced |
| WF | For s, r ∈ T with s < r and #s = #r, the pair (s, r ⊖ s) is a well-formed level-uniform span with reach r | introduced |
| S0 | Spans are convex: every position between two members is also a member | introduced |
| SC | Span classification: five exhaustive cases (separated, adjacent, proper overlap, containment, equal) | introduced |
| S6 | Level constraint: level_compat(t₁, t₂) ≡ #t₁ = #t₂; a span is level-uniform when #start = #width | introduced |
| S1 | Intersection of two level-uniform, level-compatible spans is either empty or a single span | introduced |
| S2 | The empty set is not the denotation of any span — every span is non-empty | introduced |
| S3 | Adjacent or overlapping level-uniform, level-compatible spans merge to a single span | introduced |
| S3a | Span merge is commutative | introduced |
| S4 | Split at a level-compatible interior point produces an exact partition: nothing lost, nothing duplicated, the two parts adjacent | introduced |
| TA-LC | a ⊕ x = a ⊕ y ⟹ x = y (LeftCancellation, ASN-0034) | cited |
| TA-assoc | (a ⊕ b) ⊕ c = a ⊕ (b ⊕ c) under ordered action points (AdditionAssociative, ASN-0034) | cited |
| S5 | The widths of two split parts compose under ⊕ to the original width | introduced |
| S4a | Split-merge inverse: splitting σ at a level-compatible interior point and merging recovers σ exactly | introduced |
| S3b | Merge-split inverse: merging adjacent level-uniform spans and splitting at the original boundary recovers the unordered pair {α, β} exactly | introduced |
| S7 | Every finite set of positions P admits a covering span-set Σ with |Σ| = |P| and ⟦Σ⟧ ⊇ P; exact representation of any non-empty finite P is impossible (every span denotes an infinite set) | introduced |
| S8 | Every level-compatible span-set has a normalized equivalent: sorted, non-overlapping, non-adjacent | introduced |
| S9 | The normalized form of a span-set is unique | introduced |
| S10 | For level-uniform, mutually level-compatible span-sets, union (as normalization) is commutative and associative | introduced |
| S11 | For level-uniform, level-compatible spans with containment, the difference is at most 2 spans | introduced |
| S11a | Separated or adjacent spans: ⟦α⟧ \ ⟦β⟧ = ⟦α⟧ (1 span) | introduced |
| S11b | Equal spans: ⟦α⟧ \ ⟦β⟧ = ∅ (0 spans) | introduced |
| S11c | Proper overlap: ⟦α⟧ \ ⟦β⟧ is exactly 1 span | introduced |
| S11d | General difference bound: ⟦α⟧ \ ⟦β⟧ is at most 2 spans for any SC case | introduced |
| σ.reach | reach(σ) = start(σ) ⊕ width(σ) — the exclusive upper bound | introduced |
| σ.denotation | ⟦σ⟧ = {t ∈ T : start(σ) ≤ t < reach(σ)} | introduced |
| Σ.setdenotation | ⟦Σ⟧ = union of component span denotations | introduced |
| N1, N2 | Normalized form conditions: sorted starts, separated reaches | introduced |


## Open Questions

- What abstract property must a span-set satisfy to guarantee that its normalized form remains valid as new addresses are allocated in the tumbler space?
- Under what conditions does the intersection of two spans at different hierarchical levels admit a well-formed span representation?
- What invariant must a span algebra maintain to ensure that split followed by merge always recovers the original span, even when the split point is at a finer hierarchical level than the original?
- Must the system distinguish between a span over unpopulated space and a span over populated space at the algebraic level, or is this distinction purely a content-layer concern?
- What guarantees must span operations provide at subspace boundaries, where hierarchical level transitions are structurally inherent?
- When the minimal span-set covering a target population changes due to address allocation, what is the minimal update to the old normalized form that produces the new one?
- Does the general difference bound extend to span-set difference? Given normalized span-sets Σ₁ and Σ₂, what is the tight bound on |normalize(⟦Σ₁⟧ \ ⟦Σ₂⟧)|?
- All properties here quantify over the denotation ⟦σ⟧ rather than the tumbler-length of a width; what must hold for width comparison by tumbler representation (governed by T3's canonical form) to agree with comparison by denotation?
