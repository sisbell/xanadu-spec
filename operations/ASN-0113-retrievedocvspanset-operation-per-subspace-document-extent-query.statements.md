> **ASN-0113 · RETRIEVEDOCVSPANSET Operation — Per-Subspace Document Extent Query** — condensed claim statements  
> [← Full note](ASN-0113-retrievedocvspanset-operation-per-subspace-document-extent-query.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0113 Claim Statements

*Source: ASN-0113-retrievedocvspanset-operation-per-subspace-document-extent-query.md (revised 2026-06-04) — Extracted: 2026-06-08*

## Definition — OccupiedVPositions

`O(d) = dom(M(d))`

where `d` is an allocated document (`d ∈ dom(M)`), `M(d) : T ⇀ T` is the arrangement, and `O(d)` is the set of *occupied V-positions* of `d`.

---

## Definition — ActiveVPositions

`V_S(d) = {v ∈ O(d) : subspace(v) = S}`

where `subspace(v) = v₁` (the first component of `v`), and the docuverse fixes `s_C = 1` (content), `s_L = 2` (links).

---

## Definition — VSlice

`VSlice(S, m) = {t ∈ T : t₁ = S ∧ #t = m ∧ zeros(t) = 0}`

— the depth-`m`, zero-free tumblers of subspace `S`, the population from which active V-positions are drawn (S8a).

---

## Definition — ExtentSpan

For `S ∈ occupied(d)` (i.e., `V_S(d) ≠ ∅`):

`ext(d, S) = (start_S, δ(n_S, m_S))`,  where `start_S = [S,1,…,1]` of depth `m_S`

where `n_S = |V_S(d)|` and `m_S` is the common depth of V-positions in subspace `S` (S8-depth).

---

## Definition — OccupiedSubspaces

`occupied(d) = {S ∈ {s_C, s_L} : V_S(d) ≠ ∅}`

— see W6.

---

## W-pre — Precondition (PRE, requires)

`RETRIEVEDOCVSPANSET(d)` requires `d ∈ dom(M)` (equivalently, by M0 of ASN-0093, `Document(d) ∧ d ∈ dom(M)`: a T4-valid document-level tumbler that some K.σ registration event has placed into `dom(M)`).

An *allocated empty* document (`d ∈ dom(M)`, `M(d) = ∅`) legitimately yields the defined empty span-set `⟨⟩`, whereas an *unallocated* identity (`d ∉ dom(M)`) lies outside the operation's domain and signals failure rather than fabricating `⟨⟩`.

---

## W0 — ResultType (POST, postcondition)

`RETRIEVEDOCVSPANSET(d)` returns a normalized span-set (≤ 2 members), or `⟨⟩` when both counted subspaces are empty; never a content sequence or a cardinality.

Formal body: for an *allocated* document `d` (W-pre), `RETRIEVEDOCVSPANSET(d)` returns a normalized span-set; for an allocated document that is *empty in both counted subspaces* (`d ∈ dom(M)` with `V_{s_C}(d) = V_{s_L}(d) = ∅`) it returns `⟨⟩`, the distinguished value denoting `∅` (which is not a T12 span, since every well-formed span is non-empty).

---

## W1 — SubspaceExtent (DEF, definition)

`n_S(d) = |V_S(d)|` is the extent of subspace `S` in `d`.

---

## W2 — ExtentSpanEncoding (DEF, definition)

*for `S ∈ occupied(d)` (`V_S(d) ≠ ∅`)*, `ext(d, S) = ([S,1,…,1], δ(n_S, m_S))` is the extent span encoding `n_S`

where `start_S = [S,1,…,1]` of depth `m_S` and `δ(n_S, m_S) = [0,…,0,n_S]` (the canonical pure depth-`m_S` shift, ASN-0034).

---

## W3 — ExtentSpanWellFormed (LEMMA, lemma)

*for `S ∈ occupied(d)`*, `ext(d, S)` is a well-formed, level-uniform T12 span with `reach = [S,1,…,1,1+n_S]`.

Precondition: `S ∈ occupied(d)` (i.e., `V_S(d) ≠ ∅`, hence `n_S ≥ 1`).

Subclaims:
- (a) `Pos(δ(n_S, m_S))` holds (`n_S ≥ 1`, `m_S ≥ 1`)
- (b) `actionPoint(δ(n_S, m_S)) = m_S ≤ #start_S = m_S`
- (c) T12's two preconditions hold → the span is well-formed
- (d) level-uniform: `#δ(n_S, m_S) = m_S = #start_S`
- (e) `reach(ext(d, S)) = start_S ⊕ δ(n_S, m_S) = shift(start_S, n_S) = [S,1,…,1,1+n_S]`

---

## W4 — ExactCoverage (LEMMA, lemma)

*for `S ∈ occupied(d)`*:

`⟦ext(d, S)⟧ ∩ VSlice(S, m_S) = V_S(d)`

(complete and exclusive)

where `⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` for span `σ = (s, ℓ)` (T12) and `VSlice(S, m_S) = {t ∈ T : t₁ = S ∧ #t = m_S ∧ zeros(t) = 0}`.

---

## W6 — OccupiedSubspacesDef (DEF, definition)

`occupied(d) = {S ∈ {s_C, s_L} : V_S(d) ≠ ∅}`

---

## W7 — OneSpanPerOccupiedSubspace (INV, invariant)

OneSpanPerOccupiedSubspace — result has exactly `|occupied(d)|` members, one per kind, not per fragment or item.

Formal body:

`RETRIEVEDOCVSPANSET(d) = ⟨ ext(d, S) : S ∈ occupied(d), in increasing S ⟩`

the empty span-set `⟨⟩` when `occupied(d) = ∅`.

---

## W8 — PureQuery (INV, invariant)

PureQuery — `Σ' = Σ`; the operation writes nothing and its result is a function of `dom(M(d))` alone (`C`, `L`, and the I-address values `M(d)(v)` are not read), so it changes only when `M(d)` changes.

Formal read-set constraint: each member is computed from `V_{s_C}(d)` and `V_{s_L}(d)`, and these sets `V_S(d) = {v ∈ dom(M(d)) : v₁ = S}` are determined by `dom(M(d))` and the subspace projection — the I-address *values* `M(d)(v)` are never consulted, and neither `C` nor `L` is read to produce the span-set.

---

## W9 — TwoKindsOnly (LEMMA, lemma)

TwoKindsOnly — `O(d) = V_{s_C}(d) ⊔ V_{s_L}(d)` (derived from S3★-aux); no third subspace holds content, so no third member can arise.

Formal derivation basis: S3★-aux (SubspaceExhaustiveness, ASN-0047):

`(A d, v : v ∈ dom(M(d)) : subspace(v) = s_C ∨ subspace(v) = s_L)`

with disjointness from `s_C ≠ s_L` (SC-NEQ).

---

## W10 — SubspaceConfinement (LEMMA, lemma)

SubspaceConfinement:

`(A t : t ∈ ⟦ext(d, S)⟧ : t₁ = S)`

for `t` of any depth.

---

## W11 — Disjointness (LEMMA, lemma)

Disjointness:

`⟦ext(d, s_C)⟧ ∩ ⟦ext(d, s_L)⟧ = ∅`

Derivation: for any `t` in the intersection we would need `t₁ = s_C` and `t₁ = s_L` at once (W10), impossible since `s_C ≠ s_L` (SC-NEQ, `1 ≠ 2`).

---

## W13 — UniformShape (INV, invariant)

UniformShape — result is normalized, members drawn from the fixed ordered kind-list `(s_C, s_L)`.

Normalization follows from W11: the two members are disjoint and ordered `s_C < s_L`, with `reach(ext(d, s_C)) < start_{s_L}` by T1, so no merging is possible and the sequence is in normal form.

---

## W14 — Comparability (LEMMA, lemma)

Comparability — per-kind comparison `n_S(d₁)` vs `n_S(d₂)` is well-defined across documents sharing a kind-list; an absent member reads as `n_S = 0`.

Formal basis: by W6/W7, the operation omits kind `S` exactly when `V_S(d) = ∅`, which is exactly when `n_S(d) = |V_S(d)| = 0` (W1). Hence for any two allocated documents `d₁, d₂` the per-kind comparison `n_S(d₁)` versus `n_S(d₂)` is well-defined over the *entire* fixed kind-list — text-extent to text-extent and link-extent to link-extent — *provided both reports range over the same kind-list*.

---

## W15 — Independence (LEMMA, lemma)

Independence — `n_{s_C}` depends only on `V_{s_C}(d)`, `n_{s_L}` only on `V_{s_L}(d)`; subspace edits do not cross.

Formal statement: `n_{s_C}(d)` is a function of `V_{s_C}(d)` alone, and `n_{s_L}(d)` of `V_{s_L}(d)` alone; consequently an edit confined to one subspace leaves the other subspace's reported extent unchanged.

Basis: `V_S(d) = {v ∈ O(d) : v₁ = S}` is selected by predicate `v₁ = S`; `s_C ≠ s_L` (SC-NEQ) makes `V_{s_C}(d)` and `V_{s_L}(d)` disjoint, so `n_{s_C} = |V_{s_C}(d)|` and `n_{s_L} = |V_{s_L}(d)|` are computed from non-overlapping data (W1).

---

## W16 — Partition (LEMMA, lemma)

Partition — the members disjointly cover exactly the counted active V-positions; no orphan, no phantom.

Formal statement:

`(⊔ S : S ∈ occupied(d) : ⟦ext(d, S)⟧ ∩ VSlice(S, m_S)) = {v ∈ O(d) : v₁ ∈ {s_C, s_L}}`

a *disjoint* union (W11 gives disjointness; W4 gives that each part is exactly `V_S(d)`; and `O(d)` restricted to the counted subspaces is `V_{s_C}(d) ⊔ V_{s_L}(d)` by definition).

---

## W17 — ExtentDeterminesPopulation (LEMMA, lemma)

ExtentDeterminesPopulation — each V-slice position within `ext(d, S)` carries content (`M(d)(v) ∈ dom(C)`/`dom(L)`, S3★); one step beyond W4's coverage equality.

Formal statement: for each occupied `S` and each `v ∈ ⟦ext(d, S)⟧ ∩ VSlice(S, m_S)`:

- `M(d)(v) ∈ dom(C)` for `S = s_C`
- `M(d)(v) ∈ dom(L)` for `S = s_L`

(S3★).

---

## W19 — ResultCardinalityWP (LEMMA, lemma)

ResultCardinalityWP:

Empty result:

`wp(RETRIEVEDOCVSPANSET(d), "result = ⟨⟩") ≡ d ∈ dom(M) ∧ V_{s_C}(d) = ∅ ∧ V_{s_L}(d) = ∅`

Two-member result:

`wp(RETRIEVEDOCVSPANSET(d), "|result| = 2") ≡ d ∈ dom(M) ∧ V_{s_C}(d) ≠ ∅ ∧ V_{s_L}(d) ≠ ∅`

One-member result:

`wp(RETRIEVEDOCVSPANSET(d), "|result| = 1") ≡ d ∈ dom(M) ∧ (V_{s_C}(d) = ∅ ⊻ V_{s_L}(d) = ∅)`

The three preconditions partition the allocated states (`d ∈ dom(M)`) by the pair of emptiness bits — `(∅, ∅)`, exactly one empty, neither empty — exhausting the result's three possible cardinalities.

---

## W20 — FaithfulCount (LEMMA, lemma)

FaithfulCount:

- `n_{s_L}(d) = |V_{s_L}(d)|` counts the links *arranged* in `d` exactly: CL-OWN restricts `V_{s_L}(d)` to links homed at `d`; CL-UNIQ makes `M(d)` restricted to `V_{s_L}(d)` injective — each arranged link occupies exactly one link-subspace V-position. Together they make `M(d)|_{V_{s_L}(d)}` a bijection onto `ran(M(d)|_{s_L})`, so `|V_{s_L}(d)|` counts exactly those links (a subset of all links homed at `d`).

- `n_{s_C}(d) = |V_{s_C}(d)|` counts arranged content positions: each content V-position carries exactly one I-address (S2) drawn from `dom(C)` (S3★), so `n_{s_C}(d)` is the number of arranged content positions — faithful by functionality and referential integrity.
