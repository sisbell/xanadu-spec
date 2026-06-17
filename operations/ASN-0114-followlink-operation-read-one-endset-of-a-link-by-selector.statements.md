> **ASN-0114 · FOLLOWLINK — Reading One Endset of a Link by Selector** — condensed claim statements  
> [← Full note](ASN-0114-followlink-operation-read-one-endset-of-a-link-by-selector.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0114 Claim Statements

*Source: ASN-0114-followlink-operation-read-one-endset-of-a-link-by-selector.md (revised 2026-06-04) — Extracted: 2026-06-10*

## Definition — Coverage

> `coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})`

For a span-set `R`, `coverage(R) := ⟦R⟧`. An equality `coverage(R) = coverage(eᵢ)` unfolds to `⟦R⟧ = coverage(eᵢ)`.

## Definition — FirstCollapse

> `coverage(R) = ∅ ⟺ R = ⟨⟩`

Each `⟸` is immediate — the empty object covers nothing — and each `⟹` is the contrapositive of S2. No object assembled from one or more spans can have empty coverage; coverage vanishes exactly on the empty object.

## Definition — SecondCollapse

> `coverage(e) = ∅ ⟺ e = ∅`

Symmetric to FirstCollapse, for endsets: `e ∈ Endset = 𝒫_fin(Span)`.

## Definition — SelectorValidityWP

> `wp(followlink(a, i), R is a span-set ∧ coverage(R) = coverage(Σ.L(a).eᵢ))`
> `≡ a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)|`

---

## F0 — FollowLinkDefined (DEF, function)

> **F0 (FollowLink).** `followlink(Σ, a, i)` is *defined* (returns a span-set) exactly when `a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)|`; otherwise it returns the distinguished error value `⊥`.

---

## F1 — CoverageExactness (POST, ensures)

> **F1 (CoverageExactness).** For `a ∈ dom(Σ.L)` and `1 ≤ i ≤ |Σ.L(a)|`, with `R = followlink(Σ, a, i)`:
> `coverage(R) = coverage(Σ.L(a).eᵢ)`.

---

## F2 — DiscontiguityFaithfulness (LEMMA, lemma)

> **F2 (DiscontiguityFaithfulness).** If `coverage(Σ.L(a).eᵢ)` is disconnected, then any `R` satisfying F1 has `|R| ≥ 2`. The discontiguous structure of the recorded end survives into the result; coverage exactness alone enforces it.

Sub-argument (stated in body): Suppose `coverage(Σ.L(a).eᵢ)` is *disconnected*: there exist `p < q < r` in `T` with `p, r ∈ coverage(eᵢ)` but `q ∉ coverage(eᵢ)`. A single span `σ` is order-convex — `⟦σ⟧` contains every position between any two of its members (ASN-0053, S0).
- `R ≠ ⟨⟩`: disconnectedness supplies `p, r ∈ coverage(eᵢ)`, so `coverage(eᵢ) ≠ ∅`, and by F1 `coverage(R) ≠ ∅`, which forces `R ≠ ⟨⟩` by the first collapse; hence `|R| ≥ 1`.
- `|R| ≠ 1`: if `R` were the singleton `⟨σ⟩` with `⟦σ⟧ ⊇ {p, r}`, then `q ∈ ⟦σ⟧ = coverage(R)`, yet `q ∉ coverage(eᵢ)` — contradicting F1.
- Hence `|R| ≥ 2`.

---

## F3 — RepresentationInvariance (LEMMA, lemma)

> **F3 (RepresentationInvariance).** Any two span-sets `R, R'` each satisfying F1 for the same `(Σ, a, i)` are denotationally equal: `coverage(R) = coverage(R')`. The operation's guarantee is a property of the position set, not of the span decomposition or the ordering of spans within the result.

---

## F4 — PureRead (INV, predicate)

> **F4 (PureRead).** `followlink` induces no state transition. For the state `Σ` against which it is evaluated, the post-state equals `Σ` in every component — the content store `Σ.C`, the link store `Σ.L`, the entity set `Σ.E`, every arrangement `Σ.M(d)`, and the provenance relation `Σ.R` are identical before and after.

---

## F5 — TemporalDeterminism (LEMMA, lemma)

> **F5 (TemporalDeterminism).** Let `Σ →* Σ'` be any reachable transition sequence with `a ∈ dom(Σ.L)`. Then `a ∈ dom(Σ'.L)` and `coverage(followlink(Σ', a, i)) = coverage(followlink(Σ, a, i))` for every valid selector `i`.

*Derivation.* LP13 (UnconditionalLinkPersistence, ASN-0098) supplies: "for every reachable state sequence `Σ →* Σ'` and every `a ∈ dom(Σ.L)`: `a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)`." Hence `Σ'.L(a).eᵢ = Σ.L(a).eᵢ`, and F1 applied at each state equates the coverages of the two results.

---

## F6 — SlotConfinement (LEMMA, lemma)

> **F6 (SlotConfinement).** `followlink(Σ, a, i)` is a function of the single endset `Σ.L(a).eᵢ` (up to coverage). Formally, for links `a, a'` with `coverage(Σ.L(a).eᵢ) = coverage(Σ.L(a').eᵢ)` and arbitrary contents at all slots `j ≠ i`, the results satisfy `coverage(followlink(Σ, a, i)) = coverage(followlink(Σ, a', i))`. The *coverage* of the result thus turns on no `eⱼ` with `j ≠ i`.

---

## F7 — EmptyVersusInvalid (INV, predicate)

> **F7 (EmptyVersusInvalid).** The empty span-set `⟨⟩` (a success, denoting `∅`) and the error value `⊥` (a domain violation) are distinct return categories, `⟨⟩ ≠ ⊥`. For a valid selector `1 ≤ i ≤ |Σ.L(a)|` over a link `a ∈ dom(Σ.L)` whose end `eᵢ` is empty, `followlink(Σ, a, i) = ⟨⟩`. For an invalid selector — `i < 1`, or `i > |Σ.L(a)|`, or `a ∉ dom(Σ.L)` — `followlink(Σ, a, i) = ⊥`. An implementation that collapses these two cases is incorrect.

Companion wp forms (stated in body):

> `wp(followlink(a, i), result ≠ ⊥) ≡ a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)|`

> `wp(followlink(a, i), R = ⟨⟩) ≡ a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)| ∧ Σ.L(a).eᵢ = ∅`

---

## F8 — ContentIndependence (LEMMA, lemma)

> **F8 (ContentIndependence).** `followlink(Σ, a, i)` is defined and satisfies F1 whenever `a ∈ dom(Σ.L)` and `1 ≤ i ≤ |Σ.L(a)|`, irrespective of whether any address in `coverage(Σ.L(a).eᵢ)` currently holds content or a link in `Σ`. The result reports the recorded region; the existence of material at that region is a separate question the operation does not ask.
