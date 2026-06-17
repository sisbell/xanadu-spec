> **ASN-0118 · The COPY Operation — Transclusion as Shared Reference** — condensed claim statements  
> [← Full note](ASN-0118-copy-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0118 Claim Statements

*Source: ASN-0118-copy-operation.md (revised 2026-06-11) — Extracted: 2026-06-11*

## Definition — VSpec

A *V-spec* is a pair `ρ = (d_s, σ)`: an allocated source document `d_s ∈ dom(Σ.M)` together with a span `σ = (s, ℓ)` admissible under two conditions:
1. *Well-formed* in the sense of T12 (ASN-0034): `Pos(ℓ)` and `actionPoint(ℓ) ≤ #s`, so its denotation `⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` is a well-defined order-convex set of tumblers.
2. Draws from a non-empty source subspace: `V_{subspace(s)}(d_s) ≠ ∅`.

## Definition — ActivePositions

The *active positions* of a V-spec are those tumblers the span denotes that the source arrangement actually binds:

> `act(ρ, Σ) = dom(Σ.M(d_s)) ∩ ⟦σ⟧`

The set is finite (subset of the finite `dom(Σ.M(d_s))`, S8-fin) and totally ordered (subset of the totally ordered carrier `T`, T1), hence has a unique ascending enumeration `v₁ < … < v_k`.

## Definition — SpecSet

A *spec-set* `R = ⟨ρ₁, …, ρ_q⟩` is a finite ordered sequence of V-specs with `q ≥ 1`.

## Definition — Resolution

> `resolve(R, Σ) = expand(resolve(R))`,  where
> `expand(⟨(aⱼ, nⱼ)⟩ⱼ) = ⟨a₁, a₁+1, …, a₁+(n₁−1), …, aₖ, …, aₖ+(nₖ−1)⟩`

`resolve(R, Σ) = ⟨c₀, c₁, …, c_{W−1}⟩`, with `W = |resolve(R,Σ)|` the total count of resolved addresses (the sum of the run widths `nⱼ`). A content-reference sequence resolves by concatenation: `resolve(R) = resolve(ρ₁) ⌢ … ⌢ resolve(ρ_q)`.

The per-run lockstep fixes each run's images in step with its bound positions: `Σ.M(d_s)(vⱼ + k) = aⱼ + k` for every `0 ≤ k < nⱼ`.

## Definition — ContentSubspaceRange

> `ran_C(Σ, d) = {a : (∃ v ∈ dom(Σ.M(d)) : subspace(v) = s_C ∧ Σ.M(d)(v) = a)}`

## Definition — CopyPrecondition

`enabled(COPY(Σ, d, p, R))` holds when:
- `d ∈ dom(Σ.M)` (destination is an allocated document)
- `p` is a valid insertion position: `p = min(V_{s_C}(d))` or a shift thereof when `V_{s_C}(d) ≠ ∅`, or the canonical first position `[s_C, 1, …, 1]` when `V_{s_C}(d) = ∅`
- Every member of `R` is a V-spec admissible at `Σ`: source allocated (`d_s ∈ dom(Σ.M)`), span T12-well-formed, and source subspace non-empty (`V_{subspace(s)}(d_s) ≠ ∅`)
- Content residence: `(∀ ρ ∈ R, v ∈ act(ρ, Σ) : subspace(v) = s_C)`
- `W ≥ 1`

---

## CP0 — ResolutionIntegrity (LEMMA, postcondition)

`resolve(R, Σ)` reads each active source position through its arrangement, in spec-set order, yielding `⟨c₀,…,c_{W−1}⟩` with:

**(a) Every resolved address already exists.**
> `(∀ i : 0 ≤ i < W : cᵢ ∈ dom(Σ.C))`

Each `cᵢ` is `Σ.M(d_s)(v)` for some active position `v ∈ act(ρ, Σ) ⊆ dom(Σ.M(d_s))` with `subspace(v) = s_C`; referential integrity S3★ gives `Σ.M(d_s)(v) ∈ dom(Σ.C)`.

**(b) Resolution is a pure read.**
`resolve` is a function of `Σ`; it modifies no component — not `Σ.C`, not any `Σ.M(d)`, not `Σ.L`, not `Σ.R`. The source document is consulted, never altered, by the act of resolving a spec-set against it.

**(c) Non-contiguity survives resolution.**
ASN-0058's C1a (RestrictionDecomposition) supplies the unique maximal-run decomposition of any restriction `M(d_s)|⟦σ⟧` whose domain lies in a single subspace. When a single V-span covers content the source itself assembled from several disjoint I-regions, that decomposition returns several run-pairs in V-start order (C1b, ResolutionSequenceOrder) — so the expanded sequence is *not* one contiguous run and records as many distinct origins as the source content had homes.

---

## CP1 — TransclusionFrame (POST, ensures)

> `dom(Σ'.C) = dom(Σ.C) ∧ (∀ a : a ∈ dom(Σ.C) : Σ'.C(a) = Σ.C(a))`

COPY allocates no content; the placed material refers to existing I-addresses.

---

## CP2 — Placement (POST, ensures)

> `(∀ i : 0 ≤ i < W : Σ'.M(d)(p + i) = cᵢ)`

`W` destination V-positions, freshly bound, carry the resolved (pre-existing) I-addresses; the placed material shares the source's content identity.

---

## CP3 — PriorArrangementPreservation (POST, ensures)

Three sub-clauses:

**CP3a** — *Displacement*:
> `(∀ v : v ∈ V_{s_C}(d) ∧ v ≥ p : Σ'.M(d)(v + W) = Σ.M(d)(v))`

**CP3b** — *Left frame*:
> `(∀ v : v ∈ V_{s_C}(d) ∧ v < p : Σ'.M(d)(v) = Σ.M(d)(v))`

**CP3c** — *Domain closure (text subspace)*:
> `{v ∈ dom(Σ'.M(d)) : subspace(v) = s_C} =`
> `  {v ∈ V_{s_C}(d) : v < p} ∪ {p + i : 0 ≤ i < W} ∪ {v + W : v ∈ V_{s_C}(d) ∧ v ≥ p}`

Order-preserving, injective, non-destructive; S2 functionality is dischargeable from the postconditions alone.

---

## CP4 — MultiplicityIncrease (LEMMA, lemma)

Total references into the placed set increase by exactly `W`; each placed `cᵢ`'s own reference count increases by its occurrence count in `resolve(R, Σ)` (≥ 1); distinct V-positions binding one address are permanently independent occurrences (S5, M14).

Formally: COPY adds `W` new `(document, V-position)` references (one per placement, CP2). The shifted bindings *replace* their pre-state reference pairs rather than adding to them (each `(v, a)` becomes `(v + W, a)`, the pre-shift position vacated by CP3c). CP3c with CP6 excludes any other reference from being created or destroyed.

For a fixed placed address `cᵢ`, its own reference count increases by the number of times `cᵢ` *occurs in* `resolve(R, Σ)`. The aggregate increase `W` and the per-address increase (the occurrence count) are distinct quantities; they coincide only when every resolved address is distinct.

---

## CP5 — OriginInvariance (LEMMA, lemma)

`origin(cᵢ)` is unchanged by COPY (S7(d)) and equals the document that *originally allocated* `cᵢ` — the spec-set source, a third document the source transcluded from, or `d` itself (copy-back / self-transclusion); attribution and ownership stay with that allocator.

Formally: CP1 keeps `cᵢ` in the store, and S7(d) makes `origin` constant while it is stored:
> `(∀ i : 0 ≤ i < W : origin(cᵢ) = origin_pre(cᵢ))`

where `origin(a) = N(a).0.U(a).0.D(a)` is the document-level prefix recovered from the address by field projection (ASN-0036, S7).

---

## CP6 — SourceIsolation (POST, ensures)

> `(∀ d' : d' ≠ d : Σ'.M(d') = Σ.M(d'))`

and cross-subspace frame — closing `d`'s non-`s_C` domain to its pre-state value with bindings preserved:

> `{v ∈ dom(Σ'.M(d)) : subspace(v) ≠ s_C} = {v ∈ dom(Σ.M(d)) : subspace(v) ≠ s_C}`
> `  ∧ (∀ v : v ∈ dom(Σ.M(d)) ∧ subspace(v) ≠ s_C : Σ'.M(d)(v) = Σ.M(d)(v))`

Every source and every other document is unmodified; the source's connectedness nonetheless grows (shared identity + provenance).

---

## CP7 — Links (LEMMA, lemma)

**(a)** Link store frame:
> `Σ'.L = Σ.L`

**(b)** *LinkSurvivalUnderReuse*: Let `a` be a link with `coverage(Σ.L(a).eⱼ) ∩ {c₀, …, c_{W−1}} ≠ ∅` for some endset `j`. After COPY:
> `coverage(Σ.L(a).eⱼ) ∩ ran(Σ'.M(d)) ≠ ∅`

and the discoverability characterisation at the post-state yields:
> `discoverable_from(a, d, Σ') ⟺ (∃ i : coverage(Σ'.L(a).eᵢ) ∩ ran(Σ'.M(d)) ≠ ∅)`

(ASN-0098, LP12), with `Σ'.L = Σ.L` by (a) so coverage is unchanged.

Links to the destination's prior content remain discoverable: prior images are retained in range via CP3a/CP3b, hence LP12 at the post-state — with coverage unchanged — keeps them discoverable from `d`.

The weakest precondition for link discoverability after COPY:
> `wp(COPY, "a discoverable from d") ≡ enabled(COPY(Σ, d, p, R)) ∧ (∃ j : coverage(Σ.L(a).eⱼ) ∩ {c₀, …, c_{W−1}} ≠ ∅)`

The full post-state range equation used above:
> `ran(Σ'.M(d)) = ran(Σ.M(d)) ∪ {c₀, …, c_{W−1}}`

---

## CP8 — ProvenanceClosure (POST, ensures)

> `Σ'.R = Σ.R ∪ {(cᵢ, d) : 0 ≤ i < W}`

Membership (`⊇`) holds by:
- A fresh K.ρ step for range-new addresses not already in `Σ.R` (J1'★-admissible; the canonical composite's only K.ρ steps)
- Permanence P2 for range-new addresses already in `Σ.R` (re-COPY of deleted content; a redundant K.ρ is an admissible no-op variant)
- P4★ + P2 for addresses already in `d`'s current range

The `⊆` direction pins `Σ'.R ∖ Σ.R` to the placed pairs:
> `Σ'.R ∖ Σ.R ⊆ {(cᵢ, d) : 0 ≤ i < W}`

---

## CP9 — SelfTransclusionAdmissibility (LEMMA, lemma)

When `d_s = d`, resolution reads the pre-state arrangement `Σ.M(d)`, so placement adds independent V-positions of `d` referring to addresses `d` already bound; no content is duplicated.

The result is a document with two (or more) V-positions mapping to one address — admitted by S5 and permanently independent by M14:
> `(∃ v₁, v₂ ∈ dom(Σ'.M(d)) : v₁ ≠ v₂ ∧ Σ'.M(d)(v₁) = Σ'.M(d)(v₂))`

is consistent with all invariants when `d_s = d`.

---

## CP10 — ImmutabilityPreservation (LEMMA, lemma)

S0 preserved across COPY (corollary of CP1); reused content carries identical bytes into the destination because they are the same bytes:

> `(∀ a : a ∈ dom(Σ.C) : Σ'.C(a) = Σ.C(a))`

Because `Σ.C` is untouched (CP1), content immutability S0 is preserved trivially across the COPY composite `Σ →* Σ'` — every atomic step frames `Σ.C`, so the preservation holds step by step, hence initial-to-final.

---

## CP11 — OriginMultisetPreservation (LEMMA, lemma)

> `⦃ origin(cᵢ) : 0 ≤ i < W ⦄`

is preserved verbatim into the destination's arrangement; cross-origin blocks cannot merge (M16 CrossOriginMergeImpossibility).

Within a block, all addresses share an origin (`origin(cᵢ + 1) = origin(cᵢ)` for contiguous addresses, ASN-0058 M16a); across a block boundary where the origins differ, the blocks *cannot be merged* (ASN-0058, M16). Therefore each fragment retains its distinct home, and each home remains queryable from the destination address that binds it.

---

## CP12 — EntityFrame (POST, ensures)

> `Σ'.E = Σ.E`

COPY mints no node, account, or document (the composite runs no K.δ step); with CP8's `⊆` direction, every state component is bounded above by the operation's own clauses.
