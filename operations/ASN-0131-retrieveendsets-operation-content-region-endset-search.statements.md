> **ASN-0131 · RETRIEVEENDSETS — Surfacing Anchoring Over a Content Region** — condensed claim statements  
> [← Full note](ASN-0131-retrieveendsets-operation-content-region-endset-search.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0131 Claim Statements

*Source: ASN-0131-retrieveendsets-operation-content-region-endset-search.md (revised 2026-06-13) — Extracted: 2026-06-14*

## Definition — TouchW

`touch_W(e) ≡ coverage(e) ∩ image(W, d, Σ) ≠ ∅`

where the subscript names the region's V-position set `W`, `image(W, d, Σ)` is the content-image (F-IMG, ASN-0127), and `coverage(e)` is the set of addresses an endset denotes (ASN-0098, ASN-0043).

## Definition — Addressable

`addressable(Σ) = dom(Σ.L) ∖ nullified(Σ)`

where `nullified(Σ)` is the set of withdrawn addresses (ASN-0086), a function of `Σ.L` alone.

## Definition — Avail

`Avail(Σ) = { (i, e) : (∃ a ∈ addressable(Σ) : 1 ≤ i ≤ |Σ.L(a)| ∧ Σ.L(a).eᵢ = e) }`

the region-independent pool of available slot-endset pairs.

## Definition — Sel

`sel(W, d, Σ) = { a ∈ addressable(Σ) : (∃ i : touch_W(Σ.L(a).eᵢ)) } = findlinks_V(W, d, Σ) ∩ addressable(Σ)`

---

## RE-DEF — RetrieveEndsetsDef (DEF, function)

`RE(W, d, Σ) = { (i, e) : (∃ a ∈ addressable(Σ) : 1 ≤ i ≤ |Σ.L(a)| ∧ Σ.L(a).eᵢ = e ∧ touch_W(e)) }`, where `(W, d)` has `d ∈ dom(Σ.M)` and `W ⊆ T` a content-subspace V-position set resolving to `I = image(W, d, Σ)` (F-IMG, ASN-0127); `touch_W(e) ≡ coverage(e) ∩ I ≠ ∅`; `addressable(Σ) = dom(Σ.L) ∖ nullified(Σ)` (ASN-0086); frame `Σ' = Σ`. Its return value at a selected slot follows the adopted convention RE-WHOLE (§Extent).

## RE-LOC — RetrieveEndsetLocality (INV, predicate)

Locality — for fixed `(W, d)`, `RE` reads `Σ.M(d)` (image) and `Σ.L` (endsets, and via `nullified` addressability) alone; `Σ.C`, `Σ.E`, `Σ.R` are never consulted. Hence `RE` is a deterministic function of `(W, d, Σ)`.

## RE-UNIT — RetrieveEndsetUnit (INV, predicate)

Anchoring without names — the answer's elements are `(role, endset)` pairs, never link identities; the address is withheld, distinct links sharing an endset value collapse to one pair, multiplicity is not recoverable, and a surfaced from-endset cannot be paired with its link's to-endset.

## RE-OVL — RetrieveEndsetOverlap (INV, predicate)

Overlap matching — an endset is surfaced iff at least one address it covers lies in the region's image (overlap, not containment); single-address overlap suffices; the test is existential *within* an endset and applied *per-endset*, with no per-slot request differentiation.

## RE-CLIP — RetrieveEndsetNoClip (INV, predicate)

No clipping — every surfaced span is reported at the full extent recorded in the link, never truncated to the region boundary; holds under both readings (§Extent).

## RE-WHOLE — RetrieveEndsetWhole (INV, predicate) [provisional]

Whole-endset surfacing (adopted convention) — a surfaced endset is returned in full, *all* its spans (not only those intersecting `W`), a commitment separable from RE-CLIP; the alternative touching-spans reading is left open by Open Question 1 (§Extent).

## RE-BND — RetrieveEndsetBoundary (LEMMA, lemma)

Boundary cases — `RE(W, d, Σ) = ∅` whenever the image is empty (`W ∩ dom(Σ.M(d)) = ∅`) or `addressable(Σ) = ∅`; an empty endset slot has `coverage(∅) = ∅`, so `touch_W(∅)` is false and it is never surfaced.

## RE-NCD — CrossSubspaceUnitSpanDisjoint (LEMMA, lemma)

Cross-subspace unit-span disjointness — a unit-depth span `(s, δ(1, #s))` whose T4-valid element-level start has a non-content subspace identifier (`zeros(s) = 3`, `E(s)₁ ≠ s_C`) covers no content:

`coverage({(s, δ(1, #s))}) ∩ dom(Σ.C) = ∅`

(PrefixSpanCoverage, ASN-0043; S7b, ASN-0036; L0, ASN-0093; field-agreement on separator zeros).

## RE-FIN — RetrieveEndsetFinite (LEMMA, lemma)

Finiteness and computability — `RE(W, d, Σ)` is finite *unconditionally* (drawn from the finite supply of slot-endset pairs: `dom(Σ.L)` finite by L-fin, ASN-0093, and each link bears finitely many endsets by L3, ASN-0043); and given a *finitely presented* `W` (region membership `v ∈ W` decidable, e.g. `W` given as finitely many spans), it is computed by finitely many decidable tests over the finite store — image construction over the finite `dom(Σ.M(d))` (S8-fin, ASN-0036), `coverage`-membership by intrinsic comparison on half-open T1-intervals (T12, T2, ASN-0034), and addressability via the computable `nullified` (ASN-0086).

## RE-ADDR — FreshOutputAddressable (LEMMA, lemma)

Fresh-output addressability — a fresh `K.λ` output that does not retract its own emitter address is addressable in its post-state (`ℓ_new ∉ nullified(Σ')`); in particular every non-retraction emission (`K ≁ Θ`) is addressable. Conditions: the standing discipline's unit-depth to-set and the prefix-antichain of `dom(Σ.L)` (R0a, ASN-0086).

## RE-SND — RetrieveEndsetSoundness (LEMMA, lemma)

Soundness — `(i, e) ∈ RE(W, d, Σ) ⟹ e` is a genuine slot-`i` endset of an addressable link ∧ `touch_W(e)`; no false positives.

## RE-CMP — RetrieveEndsetCompleteness (LEMMA, lemma)

Completeness — every addressable link `a` and slot `i` with `touch_W(Σ.L(a).eᵢ)` has `(i, Σ.L(a).eᵢ) ∈ RE(W, d, Σ)`; the answer is *exactly* the touching set, native or transcluded content alike.

## RE-UDIST — RetrieveEndsetUnionDist (LEMMA, lemma)

Union-distributivity — `RE(W₁ ∪ W₂, d, Σ) = RE(W₁, d, Σ) ∪ RE(W₂, d, Σ)`, the RE-level analogue of F-UDIST/F-VDIST (ASN-0127).

## RE-UDIST-∩ — RetrieveEndsetIntersectDist (LEMMA, lemma)

Intersection (one-sided) — `RE(W₁ ∩ W₂, d, Σ) ⊆ RE(W₁, d, Σ) ∩ RE(W₂, d, Σ)` holds unconditionally; the reverse `⊇` fails in general, and equality holds exactly under the touch-implication condition `(∀ (i,e) ∈ Avail(Σ) : touch_{W₁}(e) ∧ touch_{W₂}(e) ⟹ touch_{W₁ ∩ W₂}(e))` (§Composing regions).

## RE-SEL — RetrieveEndsetSelection (INV, predicate)

Discovery-side selection — `sel(W, d, Σ) = findlinks_V(W, d, Σ) ∩ addressable(Σ)` (F-V, ASN-0127): the contributing links are the addressable links discoverable through the region, so `RE` is discovery-anchored — present-tense, non-monotone (D-NONMONO, D-ZERO, ASN-0127), not existence-anchored.

## RE-TRANS — RetrieveEndsetTransclusion (INV, predicate)

Transclusion blindness — surfacing is by content identity, independent of the link's home and the covered content's origin (LP16, ASN-0098): a link reaching the region through transcluded content is surfaced identically to one on native content, each span describing content identity, not the borrowing V-position.

## RE-IDENT — RetrieveEndsetContentIdentity (INV, predicate)

Content-identity invariance — each surfaced endset's coverage is permanent (L12, ASN-0043; LP3★, ASN-0098), so the content-level answer (which I-addresses each surfaced endset anchors to) is arrangement-independent, even though the *selection* of surfaced endsets is arrangement-mediated.

## RE-EDIT — RetrieveEndsetEditStability (INV, predicate)

Present-tense stability under editing — `RE` tracks `d`'s content-subspace arrangement, so the answer is non-monotone (D-NONMONO, ASN-0127) while each surfaced endset's spans stay invariant (RE-IDENT); it moves only under content-subspace arrangement movers on `d` and `K.λ` emission/retraction (RE-RET), and is left fixed by every other transition (§Stability).

## RE-RET — RetrieveEndsetRetraction (LEMMA, lemma)

Retraction stability — withdrawing a link `ℓ` (Nullify, ASN-0086) marks it nullified, removing it from `addressable(Σ)` permanently; under the net-removal-only hypothesis `coverage(Θ) ∩ dom(Σ.C) = ∅`, a pair `(i, e)` that `ℓ` bore drops iff `ℓ` was its sole addressable bearer in `Σ` (RE-UNIT; §Stability).

## RE-CWP — RetrieveEndsetContractionWP (LEMMA, lemma)

Contraction-stability weakest precondition — for a `K.μ⁻[d, R]` step, `RE(W, d, ·) = RE(W, d, Σ)` iff

`enabled(K.μ⁻[d, R]) ∧ (∀ (i, e) ∈ Avail(Σ) : coverage(e) ∩ Δ ≠ ∅ ⟹ coverage(e) ∩ I_R ≠ ∅)`

where `I_R = {Σ.M(d)(v) : v ∈ W ∩ R}` (D-CWP bridge, ASN-0127) and `Δ = image(W, d, Σ) ∖ I_R`. The boundary `R = ∅` collapses to `RE(W, d, Σ) = ∅`.
