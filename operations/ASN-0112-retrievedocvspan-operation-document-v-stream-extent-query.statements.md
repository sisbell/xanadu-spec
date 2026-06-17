> **ASN-0112 · RETRIEVEDOCVSPAN Operation — Document V-Stream Extent Query** — condensed claim statements  
> [← Full note](ASN-0112-retrievedocvspan-operation-document-v-stream-extent-query.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0112 Claim Statements

*Source: ASN-0112-retrievedocvspan-operation-document-v-stream-extent-query.md (revised 2026-06-04) — Extracted: 2026-06-11*

## Definition — OccupiedVPositions

> `O(d) = dom(M(d))`
>
> The set of *occupied V-positions* of `d`: the positions that currently carry content in the arrangement.

## Definition — SpanEndpoints

> `origin_d = min O(d)`,  `reach_d = shift(max O(d), 1)`,  `extent_d = reach_d ⊖ origin_d`,  `σ_d = (origin_d, extent_d)`
>
> Defined when `O(d) ≠ ∅`. The reach advances one ordinal step past the maximum occupied position, realizing the half-open convention.

## Definition — SpanDenotation

> `⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` (T12), where `σ = (s, ℓ)`.
>
> `reach(σ) = s ⊕ ℓ` (ASN-0053).

## Definition — OrdinalShift

> `shift(t, n) = t ⊕ δ(n, #t)`
>
> Advances `t`'s last component by `n` (ASN-0034).

## Definition — SubspaceConvention

> `s_C = 1`, `s_L = 2` (SubspaceConventionAxiom). Because `s_C < s_L` at the first component, T1 places every content position before every link position.

## Definition — PerSubspacePositions

> `V_S(d) = {v ∈ O(d) : subspace(v) = S}`
>
> For `S ∈ {s_C, s_L}`. The content instance is `V_{s_C}(d)`, the link instance is `V_{s_L}(d)`.

## Definition — OccupiedDepth

> A tumbler `t` is *occupied-depth* at `(Σ, d)` iff `#t = m_S(d)` for some subspace `S ∈ {s_C, s_L}` with `V_S(d) ≠ ∅` — its depth is the S8-depth common depth of some non-empty subspace of `d`'s arrangement; in the cross-subspace case with `m_C ≠ m_L` there are exactly two occupied depths.

---

## V-frame — VFrame (INV, predicate)

`Σ' = Σ` — the query mutates no state component (`C, L, E, M, R` all unchanged)

## V0 — RetrieveDocVSpanSignature (AXIOM, function)

`RETRIEVEDOCVSPAN : dom(M) → SpanSet` (uniform ASN-0053 span-set codomain): the singleton span-set `⟨σ_d⟩` carrying one well-formed span `σ_d = (origin_d, extent_d)` for a non-empty document, or the empty span-set `⟨⟩` (denoting `∅`) when `O(d) = ∅` — never a content sequence, never a count

## V1 — OriginIsOccupied (LEMMA, lemma)

When `O(d) ≠ ∅`, `origin_d = min O(d)` under T1 and `origin_d ∈ O(d)` (the origin is an occupied position)

## V2 — OccupiedCoverage (LEMMA, lemma)

`O(d) ⊆ ⟦σ_d⟧` (coverage); the actual reach `r⋆ = origin_d ⊕ extent_d ≥ reach_d = shift(max O(d), 1) > max O(d)`; the span `(origin_d, extent_d)` is always a well-formed T12 span

Sub-claims:
- (a) *`#origin_d ≤ #reach_d`*: D1 (DisplacementRoundTrip) closes exactly — `r⋆ = origin_d ⊕ (reach_d ⊖ origin_d) = reach_d`. For any `v ∈ O(d)`, `origin_d ≤ v ≤ max O(d) < reach_d = r⋆`.
- (b) *`#origin_d > #reach_d`* (cross-subspace, `zpd = 1`): `r⋆` agrees with `reach_d` on every position `1 ≤ i ≤ q`, and `reach_d < r⋆` (T1 case (ii)). Hence `max O(d) < reach_d < r⋆`, so every `v ∈ O(d)` lies in `⟦σ_d⟧`.

## V3 — BoundingEndpoints (LEMMA, lemma)

`origin_d` is the greatest lower bound of `O(d)`; the *constructed endpoint* `reach_d` is the least strict upper bound of `max O(d)` among tumblers at the depth of `max O(d)`

- Lower bound: any span `σ'` with `O(d) ⊆ ⟦σ'⟧` satisfies `start(σ') ≤ min O(d) = origin_d`.
- Upper bound: `reach_d = shift(w, 1) = inc(w, 0)` (TA5-SIG, since `sig(w) = #w` by S8a). TA5 (HierarchicalIncrement, ASN-0034): `inc(w, 0)` is the smallest same-length tumbler strictly greater than `w`; the T1-immediate successor is the deeper `w.0` satisfying `w < w.0 < inc(w, 0) = reach_d`.

## V-ReachTight — VReachTight (LEMMA, lemma)

`reach(σ_d) = reach_d ⟺ #origin_d ≤ #reach_d` — the denotational reach attains the constructed endpoint exactly when origin depth does not exceed reach depth; automatic in the single-subspace regime

## V-LevelUniform — VLevelUniform (LEMMA, lemma)

`σ_d` is level-uniform (S6: `#origin_d = #extent_d`) `⟺ #origin_d ≥ #reach_d`, since `#extent_d = max(#origin_d, #reach_d)` (TA2); always level-uniform in the single-subspace regime

## V4 — VstreamBounded (INV, predicate)

`extent_d` is computed from `O(d) = dom(M(d))` alone; content in `dom(C)` but absent from the arrangement (deleted, or native elsewhere) contributes nothing (Vstream-bounded, not Istream)

## V5 — ExactCover (LEMMA, lemma)

When all occupied positions share one subspace, `⟦σ_d⟧` contains no occupied-depth position outside `O(d)`, where `t` is *occupied-depth* at `(Σ, d)` iff `#t = m_S(d)` for some `S ∈ {s_C, s_L}` with `V_S(d) ≠ ∅` (exact cover of a contiguous run)

Sub-claims (from body):
- (i) *Prefix-pinning*: Let `t` be any depth-`m_s` tumbler with `origin_d ≤ t < reach_d`. The endpoints agree on positions `1..m_s−1`, so `t` carries the prefix `[s,1,…,1]` exactly.
- (ii) *Discreteness at the boundary cell*: With the prefix pinned, `t ≥ origin_d` gives `t_{m_s} ≥ 1`, and `t < reach_d` gives `t_{m_s} < n_s + 1`, whence `t_{m_s} ≤ n_s` (TA5 tightness). So `t = [s,1,…,1,k]` with `1 ≤ k ≤ n_s`.

## V6 — BoundingBox (LEMMA, lemma)

When occupied positions span more than one subspace, `⟦σ_d⟧` contains an occupied-depth position outside `O(d)` (witness `w⋆ = [s_C,1,…,1,n_C+1]` at depth `m_C`) — the negation of V5 (bounding box, not exact cover); corollary: `O(d) ⊊ ⟦σ_d⟧`; forced because a span denotes one convex region (ASN-0053 S0) and cannot trace a separated series

## V8 — OriginPermanence (INV, predicate)

While the content subspace is non-empty, `origin_d = [s_C,1,…,1]`, invariant under all editing that leaves content present (origin permanence)

## V9 — SpanFunctionOfExtremes (LEMMA, lemma)

`origin_d` and `extent_d` are functions of the extremes `(min O(d), max O(d))` alone — never of the values `M(d)(v)` or the interior of `O(d)`; a pure rearrangement (preserving `O(d)`) returns the identical span, and a composition change moves the span iff it moves an extreme (forward direction by functionality of the extremes-to-span map, converse by V9a) — every composition change moves an extreme in the single-subspace regime (final component of `extent_d` equals `n_s`), but not in general in the cross-subspace regime, where content-side changes keeping `n'_{s_C} ≥ 1` leave the extremes, hence the span, fixed

## V9a — ExtremesRecoverable (LEMMA, lemma)

The extremes-to-span map `(min O(d), max O(d)) ↦ (origin_d, extent_d)` is injective, by explicit inverse: `min O(d) = origin_d`; `reach_d = origin_d ⊕ extent_d` when the returned width's final component is positive (`⟺ #origin_d ≤ #reach_d`, D1 round-trip), else `#reach_d = sig(extent_d)` and `reach_d` is read componentwise off TumblerSub's formula at `zpd = 1` (zero-freeness of `reach_d`, S8a); `max O(d)` is `reach_d` with final component decremented (OrdinalShift, TS2) — so a moved extreme always moves the span (extremes recoverable)

Case split on returned width's final component:
- *Final component positive* (`#o ≤ #r`): D1 applies — `o ⊕ e = o ⊕ (r ⊖ o) = r`. The reach is recovered as `r = o ⊕ e`.
- *Final component zero* (`#o > #r`, so `#e = #o`): `sig(e) = #r` (TA5-SIG; zero-freeness of `r` is load-bearing). Componentwise: `r = [o₁ + e₁, e₂, …, e_{sig(e)}]`.
- In both cases: `max O(d)` agrees with `r` on positions `1..#r − 1` and has final component `r_{#r} − 1`.

## V9b — ExactnessDiscriminator (LEMMA, lemma)

For any non-empty result, `extent_d₁ = 0 ⟺ Exact` — single-subspace endpoints first diverge at `zpd(reach_d, origin_d) = m_s ≥ 2` (S8a), so TumblerSub zeroes position 1 of the width; cross-subspace endpoints diverge at position 1, so `extent_d₁ = s_L − s_C ≥ 1` (TumblerSub at `zpd = 1`); the V5/V6 dichotomy identifies the zero case with `Exact` — the slot-1 partner of V9a's final-component tightness discriminator (exactness discriminator)

## V11 — TotalAnswerability (LEMMA, lemma)

The operation is total over allocated documents; `O(d) = ∅` yields the distinguished empty span-set `⟨⟩` (V0), with `origin_d` undefined and no extent — the implementation's zeros are a sentinel, not a legal address (TA6)

## V12 — InformationGain (LEMMA, lemma)

The result of `RETRIEVEDOCVSPAN(d)` determines time-varying arrangement facts that the permanent identity `d` cannot: the returned span-set decides emptiness (`RETRIEVEDOCVSPAN(d) = ⟨⟩ ⟺ O(d) = ∅`, V11) and, when non-empty in the single-subspace regime, its span `σ_d` fixes the exact occupied count `|O(d)| = n_s` — the final component of the returned width `extent_d = [0,…,0,n_s]` (TumblerSub at `zpd = m_s`), equivalently the final component of `reach(σ_d) = reach_d` less one (V-ReachTight); `d` is invariant under every edit (P1, EntityPermanence, ASN-0047) and reports none of these (information gain)

## V13 — Independence (LEMMA, lemma)

`σ_d` depends only on `O(d)`; two documents sharing content report independent spans; transcluded positions count toward the borrowing document's extent (independence)

## V14 — Permanence (INV, predicate)

Every *occupied* position in `O(d)` maps through `M(d)` to a permanent, immutable image, by subspace (S3★): content positions to `dom(C)` (S0, P0), link positions to `dom(L)` (L12); covered-but-unoccupied positions in the cross-subspace case (V6) carry no `M(d)` image; sharing preserves what the span denotes (permanence)

## V16 — Determinism (LEMMA, lemma)

`σ_d` is a pure function of `O(d)`; equal arrangements return identical spans, independent of how the arrangement was built; the returned span is a snapshot, not a live view (determinism)

## V18 — OriginMigration (LEMMA, lemma)

Within the non-empty-preserving editing vocabulary `{K.μ⁺, K.μ⁺_L, K.μ⁻, K.μ~}` (ASN-0047), V8's origin moves only at the two content-occupancy-toggling transitions: a `K.μ⁻` content-clearing migrates `origin_d` up to the link minimum `[s_L,1,…,1]`, a `K.μ⁺` first-content insertion into a link-only document migrates it down to the content anchor `[s_C,1,…,1]`; `K.μ⁺_L`, `K.μ~`, and occupancy-preserving `K.μ⁺`/`K.μ⁻` fix the origin (origin migration bounds V8)
