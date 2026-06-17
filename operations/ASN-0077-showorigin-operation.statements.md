> **ASN-0077 · SHOWORIGIN Operation** — condensed claim statements  
> [← Full note](ASN-0077-showorigin-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0077 Claim Statements

*Source: ASN-0077-showorigin-operation.md (revised 2026-05-25) — Extracted: 2026-06-04*

## Definition — OriginProjection

`origin(a) = N(a).0.U(a).0.D(a)`

a projection that is total on `dom(C)`, single-valued, and document-level (`zeros(origin(a)) = 2`). Defined for every `a ∈ dom(Σ.C)` by S7 of ASN-0036.

---

## O0 — OriginExtendedToLinks (CLAIM, DEFINITION)

*Define `origin : dom(C) ∪ dom(L) → E_doc` by uniformly applying S7's structural projection:*

> *`origin(x) = N(x).0.U(x).0.D(x)` for all `x ∈ dom(C) ∪ dom(L)`.*

*This extension satisfies:*

> *(a) Structural well-definedness — for every `x ∈ dom(C) ∪ dom(L)`, T4b's projections `N(x), U(x), D(x)` are defined, and `origin(x)` is a document-level tumbler with `zeros(origin(x)) = 2`.*
>
> *(b) Semantic correspondence — for every `x ∈ dom(C) ∪ dom(L)`, `origin(x)` is the tumbler of the document that allocated `x`.*
>
> *(c) Totality and single-valuedness — `origin` is total on `dom(C) ∪ dom(L)` and single-valued.*

---

## Definition — OriginsI

`origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(Σ.C) }`

I-span lift of origin. Let σ be an I-span with start `s` and width `ℓ`, denoting the half-open interval `⟦σ⟧ = { t ∈ T : s ≤ t < s ⊕ ℓ }`.

---

## Definition — OriginsV

`origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }` — (F1)

V-span lift via arrangement. `M(d)` is the arrangement of document `d`.

---

## O1 — OriginPartitionsAllocatedContent (CLAIM, LEMMA)

*Define the relation `~_o` on `⟦σ⟧ ∩ dom(C)` by `a₁ ~_o a₂ ⟺ origin(a₁) = origin(a₂)`. Then:*

> *(a) `~_o` is an equivalence relation on `⟦σ⟧ ∩ dom(C)`;*
> *(b) the quotient map `[a]_{~_o} ↦ origin(a)` is a bijection from `(⟦σ⟧ ∩ dom(C)) / ~_o` to `origins_I(Σ, σ)`;*
> *(c) each equivalence class consists exactly of those I-addresses in `⟦σ⟧ ∩ dom(C)` allocated by one document — by S7d (DocumentAllocationDiscipline, ASN-0036), one document tumbler; by the Allocator hierarchy definition and SubAllocatorBundle (ASN-0047), the outputs of that document's unique content sub-allocator `A_C(d)`.*

---

## O1.1 — SingleOriginSufficiency (COROLLARY, LEMMA)

*If every `a ∈ ⟦σ⟧ ∩ dom(C)` satisfies `origin(a) = d` for a fixed `d`, then `|origins_I(Σ, σ)| ≤ 1`* — direct from the singleton image of the bijection in O1(b). The bound is `≤ 1` rather than `= 1` because `⟦σ⟧ ∩ dom(C)` may be empty.

---

## O1.2 — MultiOriginDiagnostic (COROLLARY, LEMMA)

*If `|origins_I(Σ, σ)| > 1`, then `σ` contains I-addresses allocated by at least two distinct documents* — direct from the bijection in O1(b) combined with S7d.

---

## O2 — BlockUniformity (CLAIM, LEMMA)

*For each mapping block `(vⱼ, aⱼ, nⱼ)` arising in a decomposition of `f = M(d) ↾ ⟦σ⟧`, every I-address in `I(βⱼ)` shares `origin(aⱼ)`.*

Context: Each block `βⱼ` denotes the V→I correspondence `vⱼ + i ↦ aⱼ + i` for `0 ≤ i < nⱼ` (B3, ASN-0058).

---

## O3 — StructuralDerivation (CLAIM, LEMMA)

*`origin(a)` is computable from `a` alone, consulting no further state. `origins_I(Σ, σ)` is computable from `⟦σ⟧ ∩ dom(C)` alone; `origins_V(Σ, d, σ)` is computable from `M(d) ↾ ⟦σ⟧` alone.*

---

## O4 — ParallelWitnessesToSingleOrigin (CLAIM, LEMMA)

*Suppose `a ∈ dom(Σ.C)` with `origin(a) = d₁`, and suppose `d₂, d₃, ..., dₙ` are distinct documents each holding a V-position `vᵢ ∈ dom(M(dᵢ))` with `M(dᵢ)(vᵢ) = a` (for `2 ≤ i ≤ n`). Then for every `i ∈ {2, ..., n}`:*

> *`origin(M(dᵢ)(vᵢ)) = origin(a) = d₁`.*

*The right-hand side does not depend on `i`. Each `dᵢ` for `i ≥ 2` is an independent witness to the same fact.*

---

## O5 — OriginPermanence (CLAIM, LEMMA)

*For any `a ∈ dom(Σ.C) ∪ dom(Σ.L)` and any reachable transition `Σ → Σ'`: `origin'(a) = origin(a)`.*

---

## O5★ — MultiStepOriginPermanence (CLAIM, LEMMA)

*For any `a ∈ dom(Σ.C) ∪ dom(Σ.L)` and any reachable state sequence `Σ →* Σ'`: `a ∈ dom(Σ'.C) ∪ dom(Σ'.L)` and `origin'(a) = origin(a)`.*

---

## O6 — MonotonicGrowthUnderState (CLAIM, LEMMA)

*For any reachable `Σ → Σ'` and any I-span `σ`: `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)`.*

---

## O6★ — MultiStepMonotonicGrowth (CLAIM, LEMMA)

*For any reachable state sequence `Σ →* Σ'` and any I-span `σ`: `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)`.*

---

## O7 — VSpanStabilityUnderFixedArrangement (CLAIM, LEMMA)

*For any reachable `Σ → Σ'` such that `M'(d) ↾ ⟦σ⟧ = M(d) ↾ ⟦σ⟧`, we have `origins_V(Σ', d, σ) = origins_V(Σ, d, σ)`.*

---

## Definition — WFV (DEFINITION, PREDICATE)

*For a state Σ, document `d`, and V-span `σ = (u, ℓ)`, the predicate `WF_V(Σ, d, σ)` is the conjunction:*

> *(i) `d ∈ Σ.E_doc` — the source document is allocated (ASN-0047);*
> *(ii) σ is level-uniform: `#u = #ℓ` (S6, ASN-0053);*
> *(iii) `V_{u₁}(d) ≠ ∅` — the subspace identified by `u₁` is non-empty in `d`'s arrangement;*
> *(iv) T12 holds for `(u, ℓ)`: `Pos(ℓ)` and `actionPoint(ℓ) ≤ #u` (ASN-0034);*
> *(v) `#ℓ = #u = m`, where `m` is the common V-position depth in subspace `u₁` of `d` (S8-depth, ASN-0036);*
> *(vi) the range condition `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))`.*

---

## SDP — SubspaceDepthPreservation (LEMMA, LEMMA)

*Let `Σ → Σ'` be a reachable arrangement-extension transition on `d` — K.μ⁺ on `d` or K.μ⁺_L on `d` — and let `S ∈ {s_C, s_L}` be a subspace with `V_S(d)|_Σ ≠ ∅`. Then `V_S(d)|_Σ ⊆ V_S(d)|_{Σ'}`, and the common depth that S8-depth (ASN-0036) fixes on `V_S(d)` is the same at both states: writing `m` for the depth at Σ and `m'` for the depth at Σ', `m' = m`.*

---

## O8 — ISpanContainmentMonotonicity (CLAIM, LEMMA)

*For I-spans `σ₁, σ₂` with `⟦σ₁⟧ ⊆ ⟦σ₂⟧`: `origins_I(Σ, σ₁) ⊆ origins_I(Σ, σ₂)`.*

---

## O9 — OriginTracksCreationNotContent (CLAIM, LEMMA)

*Let `a₁, a₂ ∈ dom(C)` with `C(a₁) = C(a₂)` (identical content values). If `a₁` and `a₂` were produced by allocation events under distinct documents `d₁` and `d₂` (with `d₁ ≠ d₂`), then `origin(a₁) ≠ origin(a₂)`.*

---

## O10 — ReadOnlyFrameIdempotence (CLAIM, LEMMA)

*Let `op` be either SHOWORIGIN_I or SHOWORIGIN_V. Then for any Σ in which the precondition holds: (a) `op(Σ) = (Σ', result)` with `Σ' = Σ`; (b) two consecutive applications at the same state yield identical results.*

---

## O11 — VSpanPreservationUnderKMuPlus (CLAIM, LEMMA)

*For any reachable K.μ⁺ transition `Σ → Σ'` extending `M(d)` and any V-span `σ` over `d` with `WF_V(Σ, d, σ)` — in particular conjunct (vi), `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))`: `origins_V(Σ, d, σ) = origins_V(Σ', d, σ)`.*

---

## O11' — VSpanPreservationUnderKMuPlusL (CLAIM, LEMMA)

*For any reachable K.μ⁺_L transition `Σ → Σ'` extending `M(d)` and any V-span `σ` over `d` with `WF_V(Σ, d, σ)`: `origins_V(Σ, d, σ) = origins_V(Σ', d, σ)`.*

---

## O11.1 — WellFormednesPreservationUnderArrangementExtension (COROLLARY, LEMMA)

*Let σ be a V-span over `d` with `WF_V(Σ, d, σ)`. For any reachable arrangement-extension transition `Σ → Σ'` — K.μ⁺ on `d` or K.μ⁺_L on `d` — `WF_V(Σ', d, σ)` holds.*

---

## O11★★ — MultiStepVSpanPreservationUnderMixedChain (CLAIM, LEMMA)

*For any reachable state sequence `Σ →* Σ'` in which every `M(d)`-modifying step is either K.μ⁺ on `d` or K.μ⁺_L on `d` (i.e., no K.μ⁻ on `d` and no K.μ~ on `d` along the chain), and any V-span `σ` over `d` with `WF_V(Σ, d, σ)`: `origins_V(Σ, d, σ) = origins_V(Σ', d, σ)`.*

---

## O12 — VSpanContainmentMonotonicity (CLAIM, LEMMA)

*For V-spans `σ₁, σ₂` over the same document `d` with `⟦σ₁⟧ ⊆ ⟦σ₂⟧`: `origins_V(Σ, d, σ₁) ⊆ origins_V(Σ, d, σ₂)`.*

---

## O13 — KMuMinusAdmissibilityLoss (CLAIM, NEGATIVE)

*There exist Σ, a V-span σ over `d` with `WF_V(Σ, d, σ)`, and a reachable K.μ⁻ transition `Σ → Σ'` on `d` such that `WF_V(Σ', d, σ)` fails at conjunct (vi) — equivalently, `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊄ dom(M'(d))`. Consequently, no K.μ⁻ analogue of O11 / O11' / O11★★ holds — the V-span operation is no longer admissible at the post-state on the original input, so preservation of `origins_V` is not even formulable.*

Failure condition: conjunct (vi) ceases to hold whenever the K.μ⁻ retention parameters drop V-positions strictly inside `⟦σ⟧` from `dom(M(d))`. By K.μ⁻'s constructive retention `R = ⋃_{S ∈ {s_C, s_L}} {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}`, this happens precisely when some position in `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))` carries a sequential index `k` greater than `n'_S` in its subspace `S`.

---

## O14 — KMuTildeNonPreservation (CLAIM, NEGATIVE)

*There exist Σ, a reachable K.μ~ transition `Σ → Σ'` on `d`, and a V-span `σ` over `d` such that σ is well-formed at both Σ and Σ', yet:*

> *(i) `origins_V(Σ, d, σ) ⊄ origins_V(Σ', d, σ)`, and*
> *(ii) `origins_V(Σ', d, σ) ⊄ origins_V(Σ, d, σ)`.*

*That is, neither set is a subset of the other; no monotonicity claim parallel to O11 / O11' / O11★★ holds for K.μ~.*

---

## Definition — SHOWORIGINISpec (OPERATION, DEFINITION)

**SHOWORIGIN over an I-span.**
- *Preconditions*: `σ = (s, ℓ)` is a well-formed I-span — explicitly, the conjuncts of T12 (SpanWellDefinedness, ASN-0034): (i) `s ∈ T`; (ii) `ℓ ∈ T`; (iii) `Pos(ℓ)` (TA-Pos, ASN-0034); (iv) `actionPoint(ℓ) ≤ #s` (ActionPoint, ASN-0034).
- *Postcondition*: the result is `origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(Σ.C) }`.
- *Frame*: `Σ' = Σ`. The operation does not modify `C`, `L`, `E`, `M`, or `R`.

---

## Definition — SHOWORIGINVSpec (OPERATION, DEFINITION)

**SHOWORIGIN over a content reference.**
- *Preconditions*: `WF_V(Σ, d, σ)` (the V-span well-formedness predicate defined above, conjuncts (i)–(vi)). The subspace identifier `u₁` may be either `s_C` (content) or `s_L` (link); `origin` is total on `dom(C) ∪ dom(L)`. The postcondition is well-formed in either case because each indexed value lands in that domain: for every `v ∈ ⟦σ⟧ ∩ dom(M(d))`, S3★-aux (SubspaceExhaustiveness, ASN-0047) gives `subspace(v) ∈ {s_C, s_L}`, and with this antecedent discharged S3★ (GeneralizedReferentialIntegrity, ASN-0047) places `M(d)(v) ∈ dom(C) ∪ dom(L)`, so `origin(M(d)(v))` is defined (with the link case trivializing to `{d}` by CL-OWN).
- *Postcondition*: the result is `origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }` (form (F1)).
- *Frame*: `Σ' = Σ`.

---

## wp(SHOWORIGIN_I, |result| = 1) — WpShoworiginISingleOrigin (CLAIM, CHARACTERIZATION)

> `wp(SHOWORIGIN_I(σ), |result| = 1) = (⟦σ⟧ ∩ dom(C) ≠ ∅) ∧ (A a, b : a, b ∈ ⟦σ⟧ ∩ dom(C) : origin(a) = origin(b))`.

The postcondition `|result| = 1` says the result set is a singleton. `|origins_I(Σ, σ)| = 1` iff (a) `origins_I(Σ, σ) ≠ ∅` and (b) all elements of `origins_I(Σ, σ)` are equal.

---

## wp(SHOWORIGIN_V, d_q ∈ result) — WpShoworiginVDocumentPresence (CLAIM, CHARACTERIZATION)

> `wp(SHOWORIGIN_V(d, σ), d_q ∈ result) = (E v : v ∈ ⟦σ⟧ ∩ dom(M(d)) : origin(M(d)(v)) = d_q)`.

By (F1), `d_q ∈ origins_V(Σ, d, σ)` iff `(E v : v ∈ ⟦σ⟧ ∩ dom(M(d)) : origin(M(d)(v)) = d_q)`; since SHOWORIGIN_V's frame is `Σ' = Σ`, the post-state predicate equals the pre-state predicate, yielding the wp.
