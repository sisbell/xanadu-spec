> **ASN-0119 · REARRANGE Operation** — condensed claim statements  
> [← Full note](ASN-0119-rearrange-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0119 Claim Statements

*Source: ASN-0119-rearrange-operation.md (revised 2026-06-08) — Extracted: 2026-06-11*

## Definition — CutSequence

A *cut sequence* is a strictly ascending list of V-positions `c₀ < c₁ < ... < c_{n-1}` in the text subspace `s_C` at depth 2, with `n ∈ {3, 4}` and every cut landing on a boundary of the current arrangement (ASN-0084, CutSequence — its conditions CS3/CS4 fix exactly this subspace and depth).

For three cuts the affected interval `[c₀, c₂)` splits into two regions:

      α = { v : c₀ ≤ v < c₁ },    β = { v : c₁ ≤ v < c₂ },

with widths `w_α = ord(c₁) − ord(c₀)` and `w_β = ord(c₂) − ord(c₁)`.

For four cuts the interval `[c₀, c₃)` splits into three:

      α = [c₀, c₁),    μ = [c₁, c₂),    β = [c₂, c₃),

with `w_μ = ord(c₂) − ord(c₁)`.

Both moved-block widths are strictly positive — `w_α ≥ 1` and `w_β ≥ 1`, with `w_μ ≥ 1` as well in the four-cut case — a consequence of CS2's strict ascent.

---

## Definition — Coverage

      coverage(a, i) := coverage(Σ.L(a).eᵢ)

a purely combinatorial function of the endset that consults no state component. The suppression of the state argument is harmless across the REARRANGE transition exactly because RA6 freezes the link store: `Σ'.L = Σ.L ⟹ Σ'.L(a).eᵢ = Σ.L(a).eᵢ ⟹ coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`, so `coverage(a, i)` is one fixed address set across the transition.

---

## Definition — Project

The footprint of link `a`, endset `i`, in document `d` at state `Σ`:

      project(a, i, d, Σ) = { v ∈ dom(M(d)) : M(d)(v) ∈ coverage(a, i) }

---

## REARRANGE_K — RearrangeK (DEF, function)

Operation imported from ASN-0084: 3-/4-cut transposition in the text subspace at depth 2, specified by PivotPostcondition (R-EXT, R-P1, R-P2) or SwapPostcondition (R-EXT, R-S1, R-S2, R-S3) with frame R-FRAME-P/R-FRAME-S.

For a 3- or 4-cut sequence `K` satisfying the preconditions R-PRE, `REARRANGE_K(Σ, d)` produces the post-state `M'(d)` fixed by:

**Pivot postcondition** (`n = 3`):

      v < c₀ ∨ v ≥ c₂  ⟹  M'(d)(v) = M(d)(v),                  (R-EXT)
      M'(d)(c₀ + j)       = M(d)(c₁ + j),   0 ≤ j < w_β,        (R-P1)
      M'(d)(c₀ + w_β + j) = M(d)(c₀ + j),   0 ≤ j < w_α.        (R-P2)

**Swap postcondition** (`n = 4`):

      v < c₀ ∨ v ≥ c₃  ⟹  M'(d)(v) = M(d)(v),                  (R-EXT)
      M'(d)(c₀ + j)             = M(d)(c₂ + j),  0 ≤ j < w_β,   (R-S1)
      M'(d)(c₀ + w_β + j)       = M(d)(c₁ + j),  0 ≤ j < w_μ,   (R-S2)
      M'(d)(c₀ + w_β + w_μ + j) = M(d)(c₀ + j),  0 ≤ j < w_α.   (R-S3)

**Preconditions R-PRE**: (i) the document's arrangement exists and its text subspace is non-empty; (ii) the cuts form a CS1–CS5 cut sequence — in particular strictly ascending, by CS2; (iii) the affected interval `[c₀, c_{n-1})` lies wholly within the active text subspace.

---

## RA0 — ContentStoreFrame (LEMMA, lemma)

`Σ'.C = Σ.C` — the content store is a verbatim frame; no I-address is created, destroyed, or rebound.

---

## RA1 — IdentityCorrespondence (LEMMA, lemma)

`M'(d)(π(v)) = M(d)(v)` (ASN-0084 ArrangementRearrangement DEF + R-PPERM / R-SPERM correctness clauses), hence `ran(M'(d)) = ran(M(d))` (ASN-0084 R-RI) — I-addresses carried across the reassignment.

The middle step establishing the range equality:

      ran(M'(d)) = { M'(d)(π(v)) : v ∈ dom(M(d)) }
                 = { M(d)(v)     : v ∈ dom(M(d)) }
                 = ran(M(d)).

---

## RA2 — Permutation (LEMMA, lemma)

The induced `π` (R-PPERM/R-SPERM) is a bijection of `dom(M(d))` onto itself:

      π : dom(M(d)) → dom(M(d)),   satisfying   M'(d)(π(v)) = M(d)(v),

      dom(M'(d)) = dom(M(d)).               **(RA2, ASN-0084 R-PIV / R-SWP)**

The domain identity follows from R-PIV/R-SWP establishing the named destinations constitute a total function on `dom(M(d))`, and that the region destinations tile `[ord(c₀), ord(c_{n-1}))` exactly (disjoint and exhausting, R-EXT covering the complement, so every position is assigned exactly once).

---

## RA2a — TextSubspaceClosure (LEMMA, lemma)

      π(V_{s_C}(d)) = V_{s_C}(d).

`π` fixes every non-text position pointwise (R-PPERM/R-SPERM non-S branch); injectivity (RA2) plus finiteness (S8-fin) close the text subspace onto itself.

*Proof sketch*: Every position with `subspace(v) ≠ s_C` is fixed pointwise by `π`'s non-S branch. Suppose some `v ∈ V_{s_C}(d)` had `π(v) = w` with `subspace(w) ≠ s_C`. Then `w ∈ dom(M(d))`, the non-S branch gives `π(w) = w = π(v)`, and `v ≠ w` (their subspaces differ) — contradicting `π`'s injectivity (RA2). So `π(V_{s_C}(d)) ⊆ V_{s_C}(d)`; `π` is injective and `V_{s_C}(d)` is finite (S8-fin), so the restriction is onto.

---

## S2 — FunctionalityPreserved (INV, predicate)

`M'(d)` is single-valued — the disjoint tiling of destinations (R-PIV/R-SWP) gives each V-position one I-address (ASN-0036 S2).

---

## S3★ — ReferentialIntegrityPreserved (INV, predicate)

Per-subspace referential integrity:

      v ∈ dom(M'(d)) ∧ subspace(v) = s_C  ⟹  M'(d)(v) ∈ dom(C),
      v ∈ dom(M'(d)) ∧ subspace(v) = s_L  ⟹  M'(d)(v) ∈ dom(L).

*Derivation*: Take a text position `v ∈ dom(M'(d))` with `subspace(v) = s_C`. Then `M'(d)(v) = M(d)(π⁻¹(v))`, and `π⁻¹(v)` is again a text position (RA2a); pre-state S3★ applied at `π⁻¹(v)` gives `M(d)(π⁻¹(v)) ∈ dom(C)`, so `M'(d)(v) ∈ dom(C)`. A link position `v` with `subspace(v) = s_L` is fixed pointwise by the non-text-subspace frame (ASN-0084, R-NS / R-FRAME-P/S(a)), so `M'(d)(v) = M(d)(v) ∈ dom(L)` by pre-state S3★.

---

## S8★ — SpanDecompositionPreserved (INV, predicate)

`M'(d)` admits the unique maximal correspondence-run decomposition S8 guarantees — content subspace by ASN-0084 R-BLK + R-CANON, link subspace by the frozen frame (ASN-0047 S8★).

*Content subspace*: ASN-0084 R-BLK (RunDecompositionTransformation) carries the pre-state run partition to a disjoint, covering run partition of `M'(d)`, and R-CANON (CanonicalityOfMergeNormalForm) shows that partition's merge-normal form is the unique maximal-run decomposition S8 guarantees.

*Link subspace*: S8★'s trivial length-1 decomposition and the value-dependent CL-OWN/CL-UNIQ ride untouched on the frozen `s_L` frame.

---

## RA3 — VExtentConservation (LEMMA, lemma)

      | dom(M'(d)) | = | dom(M(d)) |,    min and max V-position fixed.

Since `dom(M'(d)) = dom(M(d))` (RA2), the active text run is literally unchanged as a set, so its cardinality and its endpoints are invariant.

---

## RA4 — EntityProvenanceFrame (LEMMA, lemma)

      Σ'.E = Σ.E ∧ Σ'.R = Σ.R

The entity set and provenance relation are verbatim frames; REARRANGE writes only `M(d)` and touches neither (the `E`/`R` components of the lifted `K.μ~` frame).

---

## RA5 — Discoverability (LEMMA, lemma)

      moved content is discoverable under its new V-position,
      and resolves to its original I-address.

Formally: evaluating `M'(d)(π(v))` equals `M(d)(v)` by RA1 — the same I-address the content always had. A navigation to `π(v)` recovers exactly the content that lived at `v`, together with its origin (RA0 leaves `origin` invariant) and its links (RA7a places their footprints at `π(v)`).

---

## RA6 — LinkStoreFrame (LEMMA, lemma)

      Σ'.L = Σ.L

Links are untouched; a link anchored in a moved region survives and travels with its content because endsets reference unchanged I-addresses.

---

## RA7a — FootprintTransport (LEMMA, lemma)

      project(a, i, d, Σ') = π( project(a, i, d, Σ) ).

*Proof*: For any `v ∈ dom(M(d))`,

      v ∈ project(a, i, d, Σ)
        ⟺ M(d)(v) ∈ coverage(a, i)            (definition of project)
        ⟺ M'(d)(π(v)) ∈ coverage(a, i)        (RA1: M'(d)(π(v)) = M(d)(v))
        ⟺ π(v) ∈ project(a, i, d, Σ'),        (definition of project; π(v) ∈ dom(M'(d)) by RA2)

and since `π` is a bijection of `dom(M(d))` (RA2), this gives the equality.

---

## RA7b — DiscoverabilityPreserved (LEMMA, lemma)

      project(a, i, d, Σ') ≠ ∅   ⟺   project(a, i, d, Σ) ≠ ∅.

Corollary of RA7a (`π` a bijection), with discoverability reduced to `coverage(a, i) ∩ ran(M(d)) ≠ ∅` by ASN-0098 LP12; by RA1 that intersection is invariant.

---

## RA7c — FootprintRunStructure (LEMMA, lemma)

      project(a, i, d, Σ) ⊆ one region
        (the s_C exterior, α, μ, β, or the frozen link subspace s_L)
        ⟹  π preserves the footprint's run structure
            (in particular, a single run stays a single run).

Within-region confinement is sufficient (not necessary) for contiguity-preservation.

*Supporting fact* (ASN-0084 R-COMM, PermutationShiftCommutativity): within each region `π` commutes with ordinal shift, `π(v + k) = π(v) + k`, so it acts there as a *constant displacement* — a rigid translation carrying the whole region by one fixed net offset. The constants are: pivot — every position of `β` by `−w_α` (R-P1), every position of `α` by `+w_β` (R-P2), exterior by `0` (R-EXT); swap — `β` by `−(w_α+w_μ)`, `μ` by `w_β−w_α`, `α` by `w_β+w_μ`, exterior by `0`.

---

## RA8a — FinalStateInvariance (LEMMA, lemma)

      π₂ ∘ π₁ = π   ⟹   M'_comp(d) = M'(d).

Both sides are `u ↦ M(d)(π⁻¹(u))` pointwise.

*Proof*: Writing `π₁, π₂` for the two moves' bijections: `M_mid(d)(π₁(v)) = M(d)(v)` and `M'_comp(d)(π₂(u)) = M_mid(d)(u)` give, at `u = π₁(v)`, `M'_comp(d)((π₂ ∘ π₁)(v)) = M(d)(v)`. A rearrangement's post-arrangement is uniquely determined by its bijection and the pre-state: the bijection equation inverts to `M'(d)(u) = M(d)(π⁻¹(u))` since `π` is a bijection (RA2). Hence `π₂ ∘ π₁ = π ⟹ M'_comp(d) = M'(d)`.

---

## RA8b — IntermediateDivergence (LEMMA, lemma)

A two-move composite passes through an observable intermediate arrangement realized by neither endpoint of the atomic transposition:

      M_mid(d) ≠ M(d)  ∧  M_mid(d) ≠ M'(d).

*Exhibited instance*: For the worked pivot `A B C D E ↦ A C D E B` (atomic cuts `ord 2, 3, 6`), the two-move realization is:

      Move 1 (cuts ord 2,3,5):  A B C D E  ↦  A C D B E   = Σ_mid
      Move 2 (cuts ord 4,5,6):  A C D B E  ↦  A C D E B   = Σ'

The intermediate state satisfies `M_mid([s_C,4]) = a₂`, while `M([s_C,4]) = a₄` and `M'([s_C,4]) = a₅`, so `M_mid(d) ≠ M(d)` and `M_mid(d) ≠ M'(d)`.

---

## RA9 — DocumentIsolation (LEMMA, lemma)

      (∀ d' ≠ d :: M'(d') = M(d'))   ∧   Σ'.C = Σ.C   ∧   Σ'.L = Σ.L.

Every other document, including transcluders of the rearranged I-addresses, is invariant. The cross-document frame clauses are ASN-0084's R-FRAME-P/S(b)/(c); the content clause is RA0; the link clause is RA6.
