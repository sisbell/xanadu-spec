> **ASN-0115 · RETRIEVEV — Content Delivery by Spec-Set** — condensed claim statements  
> [← Full note](ASN-0115-retrievev-operation-content-delivery-by-spec-set.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0115 Claim Statements

*Source: ASN-0115-retrievev-operation-content-delivery-by-spec-set.md (revised 2026-06-04) — Extracted: 2026-06-10*

## Definition — VSpec

A *V-spec* is a pair `ρ = (d, σ)` naming an **allocated** document `d` — a tumbler with `zeros(d) = 2` that is present in the arrangement family, `d ∈ dom(Σ.M)` — and a well-formed, level-uniform, **ordinal-level** span `σ = (s, ℓ)` whose start `s` is a *well-formed V-position*: a zero-free tumbler of depth at least 2 with positive components,

> `zeros(s) = 0 ∧ #s ≥ 2 ∧ (A i : 1 ≤ i ≤ #s : sᵢ > 0)`

Ordinal-level means the width acts at the deepest component, `actionPoint(ℓ) = #ℓ`.

Depth compatibility is a *consulting-state* predicate, not a well-formedness condition:

> `depthcompat(ρ, Σ) ≡ V_S(d) = ∅ ∨ #s = m_S(d)`

(well-formed because the disjunction guards `m_S(d)`, which is defined only while `V_S(d) ≠ ∅`), where `S = s₁`.

---

## Definition — SpecSet

A *spec-set* is a finite **ordered** sequence `R = ⟨ρ₁, …, ρₚ⟩` of V-specs, `p ≥ 0`.

---

## Definition — DepthCompat

Call `ρ` *depth-compatible at `Σ`* — `depthcompat(ρ, Σ)` — when the named subspace is empty or the start sits at its current common depth:

> `depthcompat(ρ, Σ) ≡ V_S(d) = ∅ ∨ #s = m_S(d)`

---

## Definition — ActivePositions

`ρ`'s *active positions* at state `Σ`:

> `act(ρ, Σ) = dom(Σ.M(d)) ∩ ⟦σ⟧`  when `depthcompat(ρ, Σ)`
> `act(ρ, Σ) = ∅`  otherwise

`act(ρ, Σ)` is finite and totally ordered, admitting a unique ascending enumeration `v₁ < v₂ < … < v_k` where `k = |act(ρ, Σ)|`.

---

## Definition — DeliveryItem

The *delivery item* for `v` (with `a = Σ.M(d)(v)`):

> `item(v, ρ, Σ) =`
> `  ⟨content, Σ.C(a)⟩`   if `subspace(v) = s_C`   (then `a ∈ dom(Σ.C)` by S3★)
> `  ⟨ref, a⟩`            if `subspace(v) = s_L`   (then `a ∈ dom(Σ.L)` by S3★)

These two cases are exhaustive on active positions (S3★-aux, SubspaceExhaustiveness).

---

## Definition — PerSpecDelivery

The *per-spec delivery* is the ascending-V sequence:

> `deliver₁(ρ, Σ) = ⟨item(v₁, ρ, Σ), …, item(v_k, ρ, Σ)⟩`

where `v₁ < v₂ < … < v_k` is the unique ascending enumeration of `act(ρ, Σ)`.

---

## Definition — Confinement (LEMMA)

**Confinement.** For an ordinal-level, level-uniform span `σ = (s, ℓ)` with `#s = #ℓ = m ≥ 2`, every `t ∈ ⟦σ⟧` extends the length-`(m − 1)` prefix `p = [s₁, …, s_{m−1}]` of `s`:

> `p ≼ t` — hence `#t ≥ m − 1` and `tⱼ = sⱼ` for `1 ≤ j < m`.

In particular `t₁ = s₁`, so `⟦σ⟧` lies wholly in subspace `s₁` and cannot cross the subspace boundary.

*Proof sketch.* Ordinal-level width acts only at position `m` (`actionPoint(ℓ) = m`), so the length-`(m − 1)` prefix `p` satisfies `p ≼ s`, and the reach `reach(σ) = s ⊕ ℓ` copies that prefix unchanged below the action point (TumblerAdd), giving `p ≼ reach(σ)`. For any `t ∈ ⟦σ⟧`, `s ≤ t < reach(σ)`, hence `s ≤ t ≤ reach(σ)`; T5 (ContiguousSubtrees) then yields `p ≼ t`.

---

## Definition — UnitSpec (LEMMA)

**UnitSpec.** Let `d ∈ dom(Σ.M)` and let `v ∈ dom(Σ.M(d))` be a bound position; write `S = subspace(v)` and `a = Σ.M(d)(v)`. Define the *unit spec at `v`*: `unit(d, v) = (d, (v, δ(1, #v)))`, with `δ` the ordinal displacement. Then:

> (a) *Well-formedness.* `unit(d, v)` is a V-spec: its named document satisfies both document conjuncts (`zeros(d) = 2`, `d ∈ dom(Σ.M)`), its start has the required V-position shape, and its span is well-formed (T12), level-uniform, and ordinal-level.
>
> (b) *Depth compatibility.* `depthcompat(unit(d, v), Σ)` holds.
>
> (c) *Singleton active set.* `act(unit(d, v), Σ) = {v}`.
>
> (d) *Unit delivery.* `deliver₁(unit(d, v), Σ)` is the one-item sequence `⟨item(v, unit(d, v), Σ)⟩` — `⟨⟨content, Σ.C(a)⟩⟩` when `S = s_C`, `⟨⟨ref, a⟩⟩` when `S = s_L`; no third case arises.

---

## R0 — Deliver (DEF, function)

`deliver(R, Σ) = deliver₁(ρ₁, Σ) ⌢ deliver₁(ρ₂, Σ) ⌢ … ⌢ deliver₁(ρₚ, Σ)`

where:
- `deliver₁(ρ, Σ)` = items of `act(ρ, Σ)` in ascending T1 order
- `act(ρ, Σ) = dom(Σ.M(d)) ∩ ⟦σ⟧` when `depthcompat(ρ, Σ)` (`V_S(d) = ∅ ∨ #s = m_S(d)`), and `∅` otherwise
- `item` carries `Σ.C(a)` for content positions (`subspace(v) = s_C`), the reference `a` for link positions (`subspace(v) = s_L`)

Boundary: when `p = 0` the concatenation has no factors, so `deliver(⟨⟩, Σ) = ⟨⟩`.

---

## R1 — MaterialDelivery (POST, postcondition)

For every active content position, the delivered item carries the bound content value `Σ.C(Σ.M(d)(v))`, not a description of where that value is stored.

---

## R2 — Faithfulness (POST, postcondition)

Every delivered content item carries exactly the value bound, in the content store, to the address the arrangement assigns its position: for every active `v` with `subspace(v) = s_C`,

> `item(v, ρ, Σ) = ⟨content, Σ.C(Σ.M(d)(v))⟩`

No other value may be substituted.

Frame limit: this governs the denotation of delivery, not any transmission channel.

---

## R3 — SpecSetExactness (POST, postcondition)

The delivery contains an item for *exactly* the active positions `act(ρⱼ, Σ)` of each spec, and no others: every delivered item arises from some `v ∈ act(ρⱼ, Σ)` (nothing extra), and every `v ∈ act(ρⱼ, Σ)` contributes an item (nothing active omitted).

For a spec depth-compatible at `Σ` this reads as span-for-span exactness:

> `act(ρⱼ, Σ) = ⟦σⱼ⟧ ∩ dom(Σ.M(dⱼ))`

— every position the span names and the arrangement binds, and no other.

For a spec depth-incompatible at `Σ`:

> `act(ρⱼ, Σ) = ∅`

so that spec contributes nothing.

---

## R4 — ArrangementRelativity (POST, postcondition)

Each V-spec `(dⱼ, σⱼ)` is resolved through the arrangement `Σ.M(dⱼ)` of the document it names — and through no other. The delivered material reflects exactly what the named arrangement binds those spans to.

---

## R5 — OrderFidelity (POST, postcondition)

Across V-specs, delivery follows spec-set sequence order: the items of `ρᵢ` wholly precede the items of `ρⱼ` whenever `i < j`, irrespective of the relative V-magnitudes of the two specs.

Within a single V-spec, items are delivered in ascending T1 order of their V-positions.

Each item's extent is fixed by its position; the boundary between consecutive items is implicit in the spec-set structure and the span endpoints, with nothing interpolated between them.

---

## R6 — SilentGapFiltering (POST, postcondition)

A named position the consulted arrangement does not make active — one outside `act(ρⱼ, Σ)` — contributes nothing to the delivery and causes no failure; delivery succeeds and returns the items for exactly the active positions `act(ρⱼ, Σ)`, the rest represented by their absence.

When `ρⱼ` is depth-compatible at `Σ`, `act(ρⱼ, Σ) = dom(Σ.M(dⱼ)) ∩ ⟦σⱼ⟧`, so the filtered positions are precisely the geometrically unbound ones:

> `v ∈ ⟦σⱼ⟧ \ dom(Σ.M(dⱼ))`

When `ρⱼ` is depth-incompatible at `Σ`, `act(ρⱼ, Σ) = ∅` and the whole span is filtered, still without failure.

Moreover, for a depth-compatible `ρⱼ`, restricted to the depth-`#s`, subspace-`S` slice of `⟦σⱼ⟧` — the only named positions the arrangement can bind (the slice is pinned at the span's own depth `#s` because `m_S(d)` is undefined in the `V_S(d) = ∅` branch, while `#s` is not) — the unbound portion never falls as an interior hole within the subspace's contiguous active range; and whenever that slice meets the active range, the unbound portion is exactly a *terminal overrun* past the bound frontier.

The no-interior-hole guarantee is a claim about the bindable slice, not about every named tumbler in the interval.

---

## R7 — Repeatability (THEOREM, implication)

Let `Σ`, `Σ'` be two states of one evolving docuverse with one a reachability descendant of the other along the sequential transition order, and let `R` be a spec-set at the *earlier* state of the pair — without loss of generality `Σ →* Σ'`, with `dⱼ ∈ dom(Σ.M)` for every `j`.

Then each `dⱼ ∈ dom(Σ'.M)` as well, so the restrictions `Σ'.M(dⱼ)|⟦σⱼ⟧` and the delivery `deliver(R, Σ')` are well-defined; and if the consulted arrangement restrictions agree:

> `Σ.M(dⱼ)|⟦σⱼ⟧ = Σ'.M(dⱼ)|⟦σⱼ⟧` for every `j`

then:

> `deliver(R, Σ) = deliver(R, Σ')`

The arrangement is the sole mutable input.

---

## R8 — TransclusionCoResolution (THEOREM, implication)

If two **distinct** active positions `v, v'` (within one spec or across specs) — distinct as positions, `(d, v) ≠ (d', v')` — resolve to the same address:

> `Σ.M(d)(v) = Σ.M(d')(v') = a`

then they share one subspace, and the co-delivery guarantee is content-only.

In the **content sub-case** (`a ∈ dom(Σ.C)`) the two distinct positions are co-resolved through the one shared address `a`:

> (i) both items carry the identical value `Σ.C(a)` (R2);
> (ii) both resolve *through* `a` — identity-preserving co-resolution — so `origin(a)` of both is one and the same (S4, S7);
> (iii) the operation performs no deduplication, so the shared content appears once per V-position.

The sharing is a fact of *resolution*, not of the delivered output: each item carries the value `Σ.C(a)`, never the address `a` (R1), so the co-delivery is byte-indistinguishable from the delivery of two coincidentally-equal contents at distinct addresses (S4) and discloses nothing about the shared origin.

The **link sub-case** is *vacuous*: two distinct active link positions can never share a link address (CL-OWN forces `origin(Σ.M(d)(v)) = d`; CL-UNIQ makes `Σ.M(d)` injective on the link subspace), so genuine transclusion is confined to content.

---

## R9 — CoherentMultiOriginAssembly (POST, postcondition)

A spec-set drawing on multiple origins is delivered as one ordered sequence (R5), assembled by resolving each spec against its own document's arrangement independently (R4).

How much origin survives *into the delivered stream* is *kind-asymmetric*, tracking the payload asymmetry the `item` definition fixes:

- A **link** item carries the address `a` itself, so its home `home(a)` is recoverable from the delivered output;
- A **content** item carries only the value `Σ.C(a)` (R1), so its origin `origin(a)` is *not* recoverable from the output — it is determinate only through the resolution mapping `v ↦ a`, an internal artifact of computing `deliver`.

---

## R10 — SubspaceCrossingObservability (POST, postcondition)

When an active position lies in the link subspace (`subspace(v) = s_L`), it resolves (by S3★) to a link address `a ∈ dom(Σ.L)`, and the delivered item is a *reference* to that link entity — an item distinguishable in kind from a content-value item.

A spec-set spanning both subspaces therefore yields a heterogeneous delivery in which the subspace boundary is observable as a change of item kind.

A span confined to the text subspace never exposes link-subspace material.

---

## R11 — PermanentSourcing (INV, invariant)

Delivery sources every content item from the immutable content store by I-address.

Consequently a content address that has ever entered `dom(Σ.C)` remains deliverable for all time: if any arrangement — the document's own, a later version's, or a transcluding document's — binds some V-position to `a`, then a spec over that document resolves to `a` and delivers `Σ.C(a)`, even if the originally-creating document's *current* arrangement no longer references `a`.

The weakest precondition for delivery to include an item *sourced from* `a` is a single live condition:

> (i) the consulted arrangement binds some active content position to `a` — a `v ∈ act(ρ, Σ)` with `subspace(v) = s_C` and `Σ.M(d)(v) = a`.

Stating (i) through `act` rather than bare namedness folds in the depth condition the override makes operative: `v ∈ act` entails the spec is depth-compatible at `Σ` (else `act = ∅`), that `v` is named (`act ⊆ ⟦σ⟧`), and that `v` is bound (`act ⊆ dom(Σ.M(d))`). There is no independent store-membership conjunct: the active position is a content position (`subspace(v) = s_C`), so S3★ discharges store membership directly — `Σ.M(d)(v) = a ⟹ a ∈ dom(Σ.C)` — the instant (i) holds; immutability (S0) then holds `Σ.C(a)` fixed for all time.
