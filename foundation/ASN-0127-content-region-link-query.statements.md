> **ASN-0127 · Content-Region Link Query** — condensed claim statements  
> [← Full note](ASN-0127-content-region-link-query.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0127 Claim Statements

*Source: ASN-0127-content-region-link-query.md (revised unknown) — Extracted: 2026-06-10*

## Definition — Image

**F-IMG (ImageDefinition).** *For `d ∈ dom(Σ.M)` and `W ⊆ T`:*

> `image(W, d, Σ) ≡ {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d))}`

*For `d ∉ dom(Σ.M)`, `image(W, d, Σ)` is undefined.*

Degenerate cases: `image(∅, d, Σ) = ∅`; `image(W, d, Σ) = ∅` whenever `W ∩ dom(Σ.M(d)) = ∅` — in particular for a freshly registered document whose arrangement is empty (`dom(Σ.M(d)) = ∅`).

---

## F-IMG-MONO — ImageMonotonicityUnderArrangementExtension (LEMMA, lemma)

*If `Σ → Σ'` extends `Σ.M(d)` (a K.μ⁺ or K.μ⁺_L step that adds positions to `d`'s arrangement while agreeing on prior positions), then for every `W ⊆ T`:*

> `image(W, d, Σ) ⊆ image(W, d, Σ')`

Extension frame gives: `dom(Σ.M(d)) ⊆ dom(Σ'.M(d))` with `Σ'.M(d)(v) = Σ.M(d)(v)` for every `v ∈ dom(Σ.M(d))`.

---

## F-IMG-CONTR — ImageContractionUnderArrangementContraction (LEMMA, lemma)

*If `Σ → Σ'` contracts `Σ.M(d)` (a K.μ⁻ step), then:*

> `image(W, d, Σ') ⊆ image(W, d, Σ)`

Contraction frame gives: `dom(Σ'.M(d)) ⊆ dom(Σ.M(d))` with `Σ'.M(d)(v) = Σ.M(d)(v)` for every `v ∈ dom(Σ'.M(d))` (retained-domain agreement).

---

## F-IMG-SWING — ImageSwingUnderReorder (LEMMA, lemma)

*If `Σ → Σ'` is a K.μ~ reorder of `d`'s arrangement with witnessing bijection `π`, then:*

> `image(W, d, Σ') = {Σ.M(d)(u) : u ∈ π⁻¹(W) ∩ dom(Σ.M(d))}`

The total range is preserved (LP11, ASN-0098: `ran(Σ'.M(d)) = ran(Σ.M(d))`) but the forward image of a fixed sub-region `W` may move; the index-set cardinality is pinned:

> `|π⁻¹(W) ∩ dom(Σ.M(d))| = |W ∩ dom(Σ.M(d))|`

so under injective `Σ.M(d)` the image cardinality is pinned as well: only membership change is realizable.

---

## F-IMG-TAX — MovedImageShapeTaxonomy (LEMMA, lemma)

*A moved image — `image(W, d, Σ') ≠ image(W, d, Σ)` under a K.μ~ reorder — stands to its predecessor in exactly one of two shapes: containment motion (`⊆`-comparable — strict, since equality is no move — so the cardinality changes) or incomparable motion (neither `⊆` nor `⊇`). Injectivity decides which shapes are available: under injective `Σ.M(d)` only incomparable motion is realizable; non-injectivity — content sharing (M13/M14, ASN-0058) — makes containment motion available but does not force it: incomparable motion remains realizable as well.*

---

## Definition — MatchPredicate

**F-MATCH (MatchPredicate).** *For `a ∈ dom(Σ.L)` and `I ⊆ T`:*

> `matches(a, I, Σ) ≡ (E i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ I ≠ ∅)`

A link matches the I-address set when *some* slot's coverage meets it.

---

## Definition — FindLinks

**F-FIND (FindPrimitive).** *The bare comprehension:*

> `findlinks(I, Σ) ≡ {a ∈ dom(Σ.L) : matches(a, I, Σ)}`

Degenerate case: `findlinks(∅, Σ) = ∅`: for every `a ∈ dom(Σ.L)` and every slot `i`, `coverage(Σ.L(a).eᵢ) ∩ ∅ = ∅`, so `matches(a, ∅, Σ)` is false; the comprehension collects no link.

---

## F-UDIST — UnionDistributivity (LEMMA, lemma)

*For all I-address sets `I₁, I₂ ⊆ T` — no disjointness required:*

> `findlinks(I₁ ∪ I₂, Σ) = findlinks(I₁, Σ) ∪ findlinks(I₂, Σ)`

---

## F-IMONO — FindMonotonicityInI (LEMMA, lemma)

*Corollary of F-UDIST. For all I-address sets `I' ⊆ I ⊆ T`:*

> `findlinks(I', Σ) ⊆ findlinks(I, Σ)`

---

## Definition — FindLinksV

**F-V (TwoPhaseFactoring).** *For `d ∈ dom(Σ.M)`, `W ⊆ T`:*

> `findlinks_V(W, d, Σ) ≡ findlinks(image(W, d, Σ), Σ)`

*Undefined when `d ∉ dom(Σ.M)`. Degenerate case: `findlinks_V(W, d, Σ) = ∅` whenever `image(W, d, Σ) = ∅` — in particular for `W = ∅`, for any `W` with `W ∩ dom(Σ.M(d)) = ∅`, and for a freshly registered `d` with empty arrangement — since `findlinks(∅, Σ) = ∅` (F-FIND).*

---

## F-FULL — FullRegionReduction (LEMMA, lemma)

*For `d ∈ dom(Σ.M)` and any region `W ⊇ dom(Σ.M(d))` — in particular `W = dom(Σ.M(d))` or `W = T`:*

> `findlinks_V(W, d, Σ) = {a ∈ dom(Σ.L) : discoverable_from(a, d, Σ)}`

*where `discoverable_from` is ASN-0098's discovery predicate.*

---

## F-VDIST — RegionUnionDistributivity (LEMMA, lemma)

*For `d ∈ dom(Σ.M)` and any V-regions `W₁, W₂ ⊆ T` — no disjointness required:*

> `findlinks_V(W₁ ∪ W₂, d, Σ) = findlinks_V(W₁, d, Σ) ∪ findlinks_V(W₂, d, Σ)`

---

## F-CIL — ComprehensionInvariantUnderΣL (LEMMA, lemma)

*Meta-lemma. If `Σ.L = Σ'.L` as partial functions, then for every comprehension*

> `{a ∈ dom(Σ.L) : P(a, Σ)}`

*whose membership predicate `P` consults only `Σ.L` and query-data (never `Σ.M`, `Σ.C`, `Σ.E`, `Σ.R`):*

> `{a ∈ dom(Σ.L) : P(a, Σ)} = {a ∈ dom(Σ'.L) : P(a, Σ')}`

Derivation chain: `Σ.L = Σ'.L` gives `dom(Σ.L) = dom(Σ'.L)` and per-link value equality `Σ.L(a) = Σ'.L(a)`; L6 gives `|Σ.L(a)| = |Σ'.L(a)|` and per-slot endset equality; coverage is deterministic; set extensionality closes.

---

## F-CIL-perlink — PerLinkInvarianceUnderValuePreservation (LEMMA, lemma)

*Sub-lemma. For any `a` with `a ∈ dom(Σ.L) ∩ dom(Σ'.L)` and `Σ'.L(a) = Σ.L(a)`:*

> `matches(a, I, Σ) ⟺ matches(a, I, Σ')` for every `I ⊆ T`

---

## F-PRES — PublishedFramePreservation (LEMMA, lemma)

*Every transition in `V_atomic ∖ {K.λ} = {K.α, K.δ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` and the composite `K.μ~` preserves the link store:*

> `dom(Σ'.L) = dom(Σ.L) ∧ (A a ∈ dom(Σ.L) :: Σ'.L(a) = Σ.L(a))`

---

## F-INERT — LinkStoreInertPreservation (LEMMA, lemma)

*For every transition in `(V_atomic ∪ {K.μ~}) ∖ {K.λ}` and every `I ⊆ T`:*

> `findlinks(I, Σ) = findlinks(I, Σ')`

The lift to any path `Σ →* Σ'` whose every atomic step is in `V_atomic ∖ {K.λ}` is by induction on path length.

---

## F-LAMBDA — KλInducedIncrement (LEMMA, lemma)

*For a single-step transition `Σ → Σ'` produced by `K.λ` allocating a fresh link `ℓ_new` with endsets `(e₁, …, e_N)`, and any `I ⊆ T`:*

> `findlinks(I, Σ') = findlinks(I, Σ) ⊎ ({ℓ_new} if matches(ℓ_new, I, Σ') else ∅)`

The two parts are disjoint: K.λ's freshness precondition gives `ℓ_new ∉ dom(Σ.L) ∪ dom(Σ.C)`, hence `ℓ_new ∉ findlinks(I, Σ)`.

---

## E-INV — CoveragePermanence (LEMMA, lemma)

*For fixed `I` and any `Σ →* Σ'`, every `a ∈ dom(Σ.L)` satisfies:*

> `a ∈ dom(Σ'.L)` and `matches(a, I, Σ') ⟺ matches(a, I, Σ)`

Derivation: LP13 (UnconditionalLinkPersistence, ASN-0098) gives `a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)` across `Σ →* Σ'`; F-CIL-perlink then delivers the match equivalence.

---

## E-MONO — ExistenceMonotonicity (LEMMA, lemma)

*For fixed `I`:*

> `Σ →* Σ' ⟹ findlinks(I, Σ) ⊆ findlinks(I, Σ')`

The store grows across the transitive closure (Store Monotonicity★, ASN-0098), coverage is invariant (E-INV), so the matching set only gains members.

---

## E-CONS — CreationConservation (LEMMA, lemma)

*For fixed `I`, the set difference `findlinks(I, Σ') ∖ findlinks(I, Σ)` over `Σ →* Σ'` consists of exactly those links created on that path whose stored value matches `I`.*

The statement is a two-direction set equality. "Created on the path" means some atomic step `Σₖ → Σₖ₊₁` on the path is a K.λ allocating `a`; this event reading coincides with `a ∈ dom(Σ'.L) ∖ dom(Σ.L)` in both directions. For created `a`, match-at-creation and match-at-`Σ'` are the same predicate by E-INV on the suffix.

---

## D-PRES — PresentTenseResolution (OBS, observation)

*Editing `d_q` moves content into or out of the queried V-region without any link being created or retracted, so the resolved request — and hence `findlinks_V` — can change while `dom(Σ.L)` is fixed.*

---

## D-ABSORB — ImageMotionAbsorption (LEMMA, lemma)

*Across any `Σ.L`-preserving transition `Σ → Σ'` (F-PRES), image-motion is necessary but not sufficient for the discovery set to move:*

> `findlinks_V(W, d_q, Σ') ≠ findlinks_V(W, d_q, Σ)` requires `image(W, d_q, Σ') ≠ image(W, d_q, Σ)`, but a moved image can leave the discovery set fixed.

Necessity: if the image is unchanged, F-INERT at the common I-argument gives equality. Insufficiency: a multi-span slot can absorb the motion when the swapped-in address re-witnesses the same links.

---

## D-NONMONO — DiscoveryNonMonotonicity (LEMMA, lemma)

*`findlinks_V` is not monotone across `Σ →* Σ'`. By case analysis on the K-transition:*

- *K.μ⁺ or K.μ⁺_L on `d_q`*: `image(W, d_q, Σ) ⊆ image(W, d_q, Σ')` (F-IMG-MONO); F-INERT bridges the evaluation state; hence:
  > `findlinks_V(W, d_q, Σ) ⊆ findlinks_V(W, d_q, Σ')`

- *K.μ⁻ on `d_q`*: `image(W, d_q, Σ') ⊆ image(W, d_q, Σ)` (F-IMG-CONTR); F-INERT bridges; hence:
  > `findlinks_V(W, d_q, Σ') ⊆ findlinks_V(W, d_q, Σ)`

- *K.μ~ on `d_q`*: `Σ.L` fixed (F-PRES/F-INERT); every motion comes through the image (D-ABSORB); F-IMG-SWING moves the image only when `W` is not fixed setwise by `π` — when `π⁻¹(W) ∩ dom(Σ.M(d_q)) = W ∩ dom(Σ.M(d_q))`, both image and discovery set are invariant. When the image does move: *containment image-motion* (`image(W, d_q, Σ) ⊆ image(W, d_q, Σ')` or reverse) yields monotone transfer via F-IMONO bridged through F-INERT; *incomparable image-motion* refutes monotonicity.

- *Transitions not on `d_q`*: `Σ'.M(d_q) = Σ.M(d_q)`, so the image is unchanged; the result changes only if `K.λ` adds a matching link (F-LAMBDA).

---

## D-CWP — ContractionStabilityWP (LEMMA, lemma)

*Fix a K.μ⁻ contraction `Σ → Σ'` on the query document `d_q` with retention set `R = ⋃ {[S, 1, …, 1, k] : S ∈ {s_C, s_L} ∧ 1 ≤ k ≤ n'_S}` (ASN-0047), so that `enabled(K.μ⁻[d_q, R])` holds — the retention counts `n'_S ∈ {0, …, n_S}` are admissible and at least one subspace strictly contracts — and `Σ'.M(d_q) = Σ.M(d_q) ↾ R`. The post-state image reduces to a pre-state quantity (the **bridge**):*

> `image(W, d_q, Σ') = {Σ.M(d_q)(v) : v ∈ W ∩ R} ≡ I_R`

*Write `Δ ≡ image(W, d_q, Σ) ∖ I_R` for the I-addresses the contraction drops from the queried region (with `image(W, d_q, Σ) = I_R ∪ Δ`, by F-IMG-CONTR). The contraction leaves the discovery set fixed*

> `findlinks_V(W, d_q, Σ') = findlinks_V(W, d_q, Σ)` iff `findlinks(Δ, Σ) ⊆ findlinks(I_R, Σ)`

*— i.e. iff every link reaching a dropped I-address also reaches an I-address retained within the queried region (`I_R`). Both `I_R = {Σ.M(d_q)(v) : v ∈ W ∩ R}` and `Δ = image(W, d_q, Σ) ∖ {Σ.M(d_q)(v) : v ∈ W ∩ R}` are functions of the pre-state `Σ` and the retention set `R` alone.*

Boundary case `R = ∅`: stability condition collapses to `findlinks_V(W, d_q, Σ) = ∅`; full clearance preserves the discovery set exactly when it was already empty.

---

## D-ZERO — PresentNotHistorical (LEMMA, lemma)

*A discovery zero `findlinks_V(W, d_q, Σ) = ∅` asserts that no link in `dom(Σ.L)` is presently reachable through the queried region — no link's coverage meets `image(W, d_q, Σ)`. It does not assert historical absence.*

*By contrast, an existence zero against fixed `I` certifies historical absence. Take any path `Σ₀ →* Σ` from the initial state (`L₀ = ∅`):*

> `findlinks(I, Σ) = ∅` implies no link satisfying `I` was ever created on any path from the initial state.

By E-CONS, `findlinks(I, Σ) ∖ findlinks(I, Σ₀) = ∅` makes the difference empty; `findlinks(I, Σ₀) = ∅` (immediate from `L₀ = ∅`) rules out any pre-existing match.
