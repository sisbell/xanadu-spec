> **ASN-0077 · SHOWORIGIN Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0077-showorigin-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0077: SHOWORIGIN Operation

*2026-05-25*

Suppose a reader confronts a passage — perhaps a single character, perhaps an entire chapter — and asks: *where did this come from?* In what document was it first set down, by what allocator was it first baptised? The answer must not depend on what the reader is doing or where the passage currently appears. A quote in a tenth-generation derivative document still has one true source. A character copied from one paragraph to another still has one true author. This invariance is fundamentally a *pointwise* property: a fixed address names one home document, and that answer is the same in every state of the system. Span-level results are derived from this pointwise guarantee rather than inheriting it unchanged.

Nelson states the requirement plainly: *"You always know where you are, and can at once ascertain the home document of any specific word or character."* [LM 2/40] The phrase *any specific word or character* sets the lower bound on scale; the phrase *at once* rules out any procedure that walks chains of indirection. The operation we are searching for is called SHOWORIGIN. Its input is a span of content. Its output is the identity of the home document — or, when the span draws from multiple sources, the set of home documents present. We must show that this operation can be specified abstractly, that its result is determined by the content alone, and that the specification extends uniformly from one address to spans of any size.

## Where origin already lives

The origin of a single I-address is not a new fact we must compute — it is recorded in the address itself. Foundation ASN-0036 establishes this as S7: for every `a ∈ dom(Σ.C)`, the *origin* is the document-level tumbler obtained by truncating the element field,

> `origin(a) = N(a).0.U(a).0.D(a)`,

a projection that is total on `dom(C)`, single-valued, and document-level (`zeros(origin(a)) = 2`). By S7d (DocumentAllocationDiscipline), distinct documents have distinct tumblers, so `origin(a₁) = origin(a₂)` says exactly that `a₁` and `a₂` were allocated by the same document. By S7's clause (d), `origin(a)` is invariant across every state in which `a ∈ dom(C)`. The structural projection reads only components of `a` itself; no registry, no index, no external context is consulted.

The same structural projection extends uniformly to link addresses.

**Claim O0 (Origin extended to dom(L)).** *Define `origin : dom(C) ∪ dom(L) → E_doc` by uniformly applying S7's structural projection:*

> *`origin(x) = N(x).0.U(x).0.D(x)` for all `x ∈ dom(C) ∪ dom(L)`.*

*This extension satisfies:*

> *(a) Structural well-definedness — for every `x ∈ dom(C) ∪ dom(L)`, T4b's projections `N(x), U(x), D(x)` are defined, and `origin(x)` is a document-level tumbler with `zeros(origin(x)) = 2`.*
>
> *(b) Semantic correspondence — for every `x ∈ dom(C) ∪ dom(L)`, `origin(x)` is the tumbler of the document that allocated `x`.*
>
> *(c) Totality and single-valuedness — `origin` is total on `dom(C) ∪ dom(L)` and single-valued.*

*Derivation.* (a) For `x ∈ dom(C)`, S7b (ASN-0036) gives `zeros(x) = 3`. For `x ∈ dom(L)`, L1 (LinkElementLevel, ASN-0047) gives `zeros(x) = 3` — every link address is an element-level tumbler — while L1b (ASN-0047) supplies the complementary element-field depth `#E(x) ≥ 2`. In both cases T4b (UniqueParse, ASN-0034) is applicable, so the projections `N(x), U(x), D(x)` are well-defined; the constructed tumbler `N(x).0.U(x).0.D(x)` is document-level by direct count of separators (`zeros = 2`).

(b) For `x ∈ dom(C)`, S7 of ASN-0036 supplies the correspondence: `origin(x)` is the document that performed the allocation event placing `x` into `dom(C)`. For `x ∈ dom(L)`, two foundation facts compose directly. First, L1c (LinkAllocatorConformance, ASN-0047) ties every `ℓ ∈ dom(L)` to an allocation chain rooted at the document-level seed `t₀ = origin(ℓ)` (with `zeros(t₀) = 2`), whose descent step `k₁ = 2` enters `origin(ℓ)`'s element-field allocation region; since every `ℓ ∈ dom(L)` carries `subspace_I(ℓ) = s_L` (L0, SubspacePartition, ASN-0047), that chain routes specifically through `origin(ℓ)`'s link sub-allocator `A_L(origin(ℓ))`; by SubAllocatorBundle (ASN-0047), `dom(A_C(d)) ∩ dom(A_L(d)) = ∅` and `dom(A_L(d)) ∩ dom(A_L(d')) = ∅` for `d ≠ d'`, so this attribution is unambiguous. Second, the Allocator hierarchy definition (ASN-0047) states that `A_L(d)` is `d`'s link sub-allocator and that every output `ℓ` of `A_L(d)` satisfies `origin(ℓ) = d`: the allocating document of any `A_L(d)` output is exactly `d`. Composing the two, `ℓ` is an output of `A_L(origin(ℓ))`, whose allocating document is `origin(ℓ)`. Hence `origin(ℓ)` names the document that allocated `ℓ`.

(c) Totality on `dom(C) ∪ dom(L)` requires both well-formedness of the structural projection at every `x` in the domain *and* membership of the result in the stated codomain `E_doc`. (a) discharges the first conjunct — for every `x ∈ dom(C) ∪ dom(L)`, T4b's projections `N(x), U(x), D(x)` are defined and `origin(x) = N(x).0.U(x).0.D(x)` is a syntactically well-formed document-level tumbler. (b) underwrites the second — `origin(x)` is the tumbler of the document that allocated `x`, and that allocating document inhabits `E_doc`. For `x ∈ dom(C)`: P6 (ExistentialCoherence, ASN-0047) states `(A a ∈ dom(C) :: origin(a) ∈ E_doc)`, so `origin(x) ∈ E_doc`. For `x ∈ dom(L)`: L1a (LinkScopedAllocation, ASN-0047) states `(A a ∈ dom(Σ.L) :: origin(a) ∈ E_doc)` as a per-state invariant, so `origin(x) ∈ E_doc` directly. Composing the two cases, `origin(x) ∈ E_doc` for every `x ∈ dom(C) ∪ dom(L)`. Single-valuedness is T4b's functional definition of projections. ∎

What we do not yet have is an operation that takes a *span* — not just one address — and reports the documents present. That is what we now construct.

## Lifting origin to an I-span

Let σ be an I-span (foundation ASN-0053, T12), with start `s` and width `ℓ`, denoting the half-open interval

> `⟦σ⟧ = { t ∈ T : s ≤ t < s ⊕ ℓ }`.

Not every position in `⟦σ⟧` need lie in `dom(C)`; only those that do are content. We define the I-span lift of origin:

> `origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(Σ.C) }`.

The result is a finite set of document-level tumblers — finite because `dom(C)` is finite (C-fin, foundation ASN-0047). The set may be empty (no positions in σ are allocated), a singleton (all allocated addresses come from one document), or larger (σ crosses content subspaces of distinct documents).

By S7a (DocumentScopedAllocation, foundation ASN-0036), every I-address allocated by document `d` carries `d`'s prefix. Two addresses share an origin iff they share the prefix `N(a).0.U(a).0.D(a)`. The structural fact this delivers is what we call the origin partition:

**Claim O1 (Origin partitions allocated content).** *Define the relation `~_o` on `⟦σ⟧ ∩ dom(C)` by `a₁ ~_o a₂ ⟺ origin(a₁) = origin(a₂)`. Then:*

> *(a) `~_o` is an equivalence relation on `⟦σ⟧ ∩ dom(C)`;*
> *(b) the quotient map `[a]_{~_o} ↦ origin(a)` is a bijection from `(⟦σ⟧ ∩ dom(C)) / ~_o` to `origins_I(Σ, σ)`;*
> *(c) each equivalence class consists exactly of those I-addresses in `⟦σ⟧ ∩ dom(C)` allocated by one document — by S7d (DocumentAllocationDiscipline, ASN-0036), one document tumbler; by the Allocator hierarchy definition and SubAllocatorBundle (ASN-0047), the outputs of that document's unique content sub-allocator `A_C(d)`.*

*Derivation.* (a) Reflexivity, symmetry, and transitivity are inherited from equality on the codomain of `origin`. (b) The map is well-defined: if `a₁ ~_o a₂` then `origin(a₁) = origin(a₂)` by definition. It is surjective onto `origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(C) }` by the definition of `origins_I` (every element is hit by the class of some `a`). It is injective: if `[a₁]_{~_o} ≠ [a₂]_{~_o}` then `origin(a₁) ≠ origin(a₂)`, so the images differ. (c) Fix an equivalence class `[a]_{~_o}` and write `d = origin(a)`. By S7a, every `b ∈ ⟦σ⟧ ∩ dom(C)` with `origin(b) = d` was allocated by the document whose tumbler is `d`. By S7d, distinct documents have distinct tumblers, so the tumbler `d` names exactly one document. By the Allocator hierarchy definition (ASN-0047), every output of `d`'s content sub-allocator `A_C(d)` has `subspace_I = s_C` and every output of `d`'s link sub-allocator `A_L(d)` has `subspace_I = s_L`; by SubAllocatorBundle (ASN-0047), `dom(A_C(d)) ∩ dom(A_L(d)) = ∅`. By L0 (SubspacePartition, ASN-0047), every `b ∈ dom(C)` has `subspace_I(b) = s_C`, so `d`'s allocations into `dom(C)` route exclusively through `A_C(d)` (and not `A_L(d)`). Hence the class `[a]_{~_o}` consists exactly of addresses allocated by document `d` — equivalently, the outputs of `A_C(d)`. ∎

Two corollaries follow without further argument.

**Corollary O1.1 (Single-origin sufficiency).** *If every `a ∈ ⟦σ⟧ ∩ dom(C)` satisfies `origin(a) = d` for a fixed `d`, then `|origins_I(Σ, σ)| ≤ 1`* — direct from the singleton image of the bijection in O1(b). The bound is `≤ 1` rather than `= 1` because `⟦σ⟧ ∩ dom(C)` may be empty.

**Corollary O1.2 (Multi-origin diagnostic).** *If `|origins_I(Σ, σ)| > 1`, then `σ` contains I-addresses allocated by at least two distinct documents* — direct from the bijection in O1(b) combined with S7d. This is what justifies treating multi-origin results as informative: such spans necessarily cross document-allocation boundaries in the I-stream. T12 admits these spans (placing no upper limit on width), but they do not arise from any single document's allocation activity.

## Lifting origin to a V-span

A reader more naturally has access to a V-span — a contiguous region of positions in the document they are reading. The content at those positions may be native (allocated by the reader's document) or transcluded (allocated elsewhere, included by reference). SHOWORIGIN must resolve this question through the document's arrangement.

Foundation ASN-0058 supplies the machinery in subspace-agnostic form. Let `f = M(d) ↾ ⟦σ⟧` — the restriction of `d`'s arrangement to the positions of σ. By C1a (RestrictionDecomposition, ASN-0058), `f` admits a unique maximally merged block decomposition

> `{β₁, ..., βₖ} = {(v₁, a₁, n₁), ..., (vₖ, aₖ, nₖ)}`,

where each block `βⱼ` denotes the V→I correspondence `vⱼ + i ↦ aⱼ + i` for `0 ≤ i < nⱼ` (B3, ASN-0058), and the blocks partition `dom(f)` (B1, ASN-0058). C1a's preconditions — functionality (S2), finite domain (S8-fin), and common depth `m ≥ 2` (S8-depth combined with S8a, ASN-0036) — are subspace-agnostic; the decomposition is well-defined whether the V-positions of `dom(f)` lie in the content subspace (so I-addresses lie in `dom(C)` by S3★) or the link subspace (so I-addresses lie in `dom(L)` by S3★).

The V-span origin set is defined directly as the image of the arrangement under `origin`:

> *(F1)* `origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }`.

Within a single mapping block the origin is constant: a block maps a contiguous run of V-positions to a contiguous run of I-addresses, and those I-addresses all share one origin. One origin per block therefore suffices to account for the block's entire content.

**Claim O2 (Block uniformity).** *For each mapping block `(vⱼ, aⱼ, nⱼ)` arising in a decomposition of `f = M(d) ↾ ⟦σ⟧`, every I-address in `I(βⱼ)` shares `origin(aⱼ)`.*

*Derivation.* Fix `0 ≤ i < nⱼ`. B3 (Consistency, ASN-0058) gives `f(vⱼ + i) = aⱼ + i`; since `f` is a restriction of `M(d)`, also `M(d)(vⱼ + i) = aⱼ + i`. B1 (Coverage, ASN-0058) gives `vⱼ + i ∈ V(βⱼ) ⊆ dom(f) ⊆ dom(M(d))`. The bridge is uniform: M-int (TumblerIntervalCharacterization, ASN-0058) applies at `x = vⱼ`, `y = vⱼ + i`, `n = nⱼ` — its precondition `vⱼ, vⱼ + i ∈ dom(M(d))` (by B1) with `vⱼ ≤ vⱼ + i < vⱼ + nⱼ` is met — and yields `subspace(vⱼ + i) = subspace(vⱼ)` for every `0 ≤ i < nⱼ`. Two cases by subspace of `vⱼ`, exhaustive by S3★-aux (SubspaceExhaustiveness, ASN-0047) applied to `vⱼ ∈ dom(M(d))`: `subspace(vⱼ) ∈ {s_C, s_L}`. *Content block* (`subspace(vⱼ) = s_C`): M-int gives `subspace(vⱼ + i) = s_C`; with this antecedent discharged, S3★ (ASN-0047) at `vⱼ + i ∈ dom(M(d))` gives `aⱼ + i ∈ dom(C)`. M16a (OriginInvarianceUnderShift, ASN-0058) requires *both* conjuncts `aⱼ ∈ dom(C)` and `aⱼ + i ∈ dom(C)`: the latter is the instance just derived, and the former is the same derivation at `i = 0` (which gives `aⱼ + 0 = aⱼ ∈ dom(C)`). With both conjuncts of M16a's precondition discharged at `(aⱼ, i)`, M16a delivers `origin(aⱼ + i) = origin(aⱼ)`. *Link block* (`subspace(vⱼ) = s_L`): M-int gives `subspace(vⱼ + i) = s_L`; with this antecedent discharged, S3★ at `vⱼ + i ∈ dom(M(d))` gives `aⱼ + i ∈ dom(L)`. With both CL-OWN preconditions — `vⱼ + i ∈ dom(M(d))` and `subspace(vⱼ + i) = s_L` — discharged at `vⱼ + i` and (by `i = 0`) at `vⱼ`, CL-OWN (ASN-0047) gives `origin(M(d)(vⱼ)) = d` (so `origin(aⱼ) = d`) and `origin(M(d)(vⱼ + i)) = d` (so `origin(aⱼ + i) = d`). Hence `origin(aⱼ + i) = d = origin(aⱼ)`. In both cases `origin(aⱼ + i) = origin(aⱼ)`. ∎

Like its I-span counterpart, `origins_V(Σ, d, σ)` is a finite set of document-level tumblers — finite because `⟦σ⟧ ∩ dom(M(d)) ⊆ dom(M(d))` is finite by S8-fin (FiniteArrangement, ASN-0036), and the image of a finite set under `origin` (form (F1)) is finite. The set may be smaller than `k` if multiple blocks share an origin — for instance, two separately-transcluded passages drawn from the same source document, or transcluded content interleaved with native content of `d` where the native portions and `d` itself share an origin (`d` itself, for native).

A level-uniform V-span lies in a single subspace. Mixed V-spans (crossing both subspaces) are excluded by the conjunction of C0 (OrdinalDisplacementNecessity, ASN-0058) and C0a (PrefixConfinement, ASN-0058). C0 forces the displacement's action point to coincide with the common depth `m ≥ 2`, so `ℓ₁ = 0`; TumblerAdd's prefix-copy rule then gives `reach(σ)_1 = u_1`. C0a delivers `t_j = u_j` for every `1 ≤ j < m` and every `t ∈ ⟦σ⟧`; in particular `t_1 = u_1 = subspace(u)`. Every position in `⟦σ⟧` therefore shares `u`'s subspace identifier, so a level-uniform V-span lies in a single subspace.

## Structural derivation

The most important claim about SHOWORIGIN is not what it computes but what it does *not* need.

**Claim O3 (Structural derivation).** *`origin(a)` is computable from `a` alone, consulting no further state. `origins_I(Σ, σ)` is computable from `⟦σ⟧ ∩ dom(C)` alone; `origins_V(Σ, d, σ)` is computable from `M(d) ↾ ⟦σ⟧` alone.*

*Derivation.* For the pointwise claim: (i) S7 of ASN-0036 defines `origin(a) = N(a).0.U(a).0.D(a)` on `dom(C)`; O0 (above) extends the same structural projection uniformly to `dom(L)`, so `origin` is total on `dom(C) ∪ dom(L)`. (ii) T4b (UniqueParse, ASN-0034) defines `N(a), U(a), D(a)` as projections that read only the component sequence of `a` — they require the structural facts `zeros(a) ≥ 2` (here `= 3` by S7b of ASN-0036 for `dom(C)` and by L1 (LinkElementLevel) of ASN-0047 for `dom(L)`) and the field-separator positions, both determinable by scanning `a`. (iii) Composition of two functions that read only `a` is a function that reads only `a`. Hence `origin(a)` consults no state beyond `a`.

For the I-span lift: `origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(C) }` evaluates `origin` pointwise. The set `⟦σ⟧ ∩ dom(C)` is determined by σ (whose denotation is a function of `start(σ)` and `width(σ)` alone, by ASN-0053) and by `dom(C)` (the set of allocated content addresses in Σ). No other component of Σ is consulted.

For the V-span lift, by (F1): `origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }`. The arrangement `M(d) ↾ ⟦σ⟧` determines `dom(M(d) ↾ ⟦σ⟧) = ⟦σ⟧ ∩ dom(M(d))` and the function values `M(d)(v)` for `v` in that domain. Well-definedness of `origin(M(d)(v))` at each `v` in the indexing set is *structural*, requiring no state read: the projection `N(x).0.U(x).0.D(x)` is total on any tumbler `x` with `zeros(x) ≥ 2` (T4b, UniqueParse, ASN-0034 — the parse reads only `x`'s component sequence), so given the value `M(d)(v)` that the restriction itself supplies, `origin(M(d)(v))` is computed from that address value alone. ∎

The consequence for transclusion is decisive. A document whose server is unreachable still has its tumbler recorded in every transcluded I-address that originated from it; SHOWORIGIN reports this tumbler from the address structure alone. The unreachability of the source bears on whether the bytes can be *fetched*, not on whether the origin can be *named*.

This is what Nelson means when he insists attribution is *unstrippable within the docuverse* [Q1]: there is no metadata to strip. The origin claim is part of the address, and the address is the means by which the bytes are retrieved.

## Direct resolution through transclusion

Suppose content was allocated in document `d₁`. Document `d₂` transcludes it; `d₃` transcludes from `d₂`; this continues to `dₙ`. A reader of `dₙ` asks SHOWORIGIN. What does it return?

Because each transclusion is by reference rather than copy, every intermediate document's arrangement records the *same* I-address `a` — the bytes baptised by `d₁` — rather than a copy (K.μ⁺ and J4 of ASN-0047 propagate the original I-address unchanged). So any of these arrangements can be queried with the same result:

**Claim O4 (Parallel witnesses to a single origin).** *Suppose `a ∈ dom(Σ.C)` with `origin(a) = d₁`, and suppose `d₂, d₃, ..., dₙ` are distinct documents each holding a V-position `vᵢ ∈ dom(M(dᵢ))` with `M(dᵢ)(vᵢ) = a` (for `2 ≤ i ≤ n`). Then for every `i ∈ {2, ..., n}`:*

> *`origin(M(dᵢ)(vᵢ)) = origin(a) = d₁`.*

*The right-hand side does not depend on `i`. Each `dᵢ` for `i ≥ 2` is an independent witness to the same fact.*

*Derivation.* Fix `i ∈ {2, ..., n}`. By hypothesis, `M(dᵢ)(vᵢ) = a`. The pure projection `origin` (defined on `dom(C)`, by S7 of ASN-0036) takes `M(dᵢ)(vᵢ)` to `origin(M(dᵢ)(vᵢ)) = origin(a)`. By hypothesis `origin(a) = d₁`. This argument uses only `dᵢ`'s entry at `vᵢ` and the projection; it never names or reads `dⱼ` for any `j ≠ i`. ∎

This is what Nelson means by *at once* [Q10]: O4 makes each intermediate document an independent witness to `d₁`, so the origin is reported from any one of them directly.

## Permanence

We turn to the question raised at the start: does the answer change?

**Claim O5 (Origin permanence).** *For any `a ∈ dom(Σ.C) ∪ dom(Σ.L)` and any reachable transition `Σ → Σ'`: `origin'(a) = origin(a)`.*

*Derivation.* P3 (ArrangementMutabilityOnly, ASN-0047) — which includes `dom(C) ⊆ dom(C')` and `dom(L) ⊆ dom(L')` — discharges membership preservation: `a ∈ dom(Σ.C) ∪ dom(Σ.L)` entails `a ∈ dom(Σ'.C) ∪ dom(Σ'.L)`, so the projection `origin'` is defined at `a`. By O3 (sub-claim 1), `origin` is a pure projection of the component sequence of its argument — it consults no state beyond the address itself. Evaluating the same pure function on the same value `a` yields the same result in any state; hence `origin'(a) = origin(a)`. ∎

**Claim O5★ (Multi-step origin permanence).** *For any `a ∈ dom(Σ.C) ∪ dom(Σ.L)` and any reachable state sequence `Σ →* Σ'`: `a ∈ dom(Σ'.C) ∪ dom(Σ'.L)` and `origin'(a) = origin(a)`.*

*Derivation.* `origin` is well-defined on each store separately (`dom(C)` and `dom(L)`), so we pair each value-preservation clause with the membership clause that makes its accessor well-defined and take as the single-step guarantee the finite conjunction of four per-store clauses: `c₁ ≡ [a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C)]`, `c₂ ≡ [a ∈ dom(Σ.L) ⟹ a ∈ dom(Σ'.L)]`, `c₃_C ≡ [a ∈ dom(Σ.C) ⟹ origin'(a) = origin(a)]`, and `c₃_L ≡ [a ∈ dom(Σ.L) ⟹ origin'(a) = origin(a)]`. O5 establishes each single-step: under `a ∈ dom(Σ.C)`, `c₁` gives `a ∈ dom(Σ'.C)` (so `origin'` is well-defined at `a` and `c₃_C` follows), and symmetrically `c₂`, `c₃_L` under `a ∈ dom(Σ.L)`. The Closure schema (★) (ClosureSchema, ASN-0098) lifts the conjunction `c₁ ∧ c₂ ∧ c₃_C ∧ c₃_L` to its transitive closure `Σ →* Σ'`. The disjunctive conclusion follows by case split on the disjunctive hypothesis: if `a ∈ dom(Σ.C)`, the closed `c₁` gives `a ∈ dom(Σ'.C)` and the closed `c₃_C` gives `origin'(a) = origin(a)`; if `a ∈ dom(Σ.L)`, the closed `c₂` and `c₃_L` give the same pair symmetrically. Either way both conjuncts of the claim hold. ∎

For the I-span lift, permanence has a directional character.

**Claim O6 (Monotonic growth under state).** *For any reachable `Σ → Σ'` and any I-span `σ`: `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)`.*

*Derivation.* Fix any `o ∈ origins_I(Σ, σ)`. By definition, there exists `a ∈ ⟦σ⟧ ∩ dom(Σ.C)` with `origin(a) = o`. (1) By P0 (ContentPermanence, ASN-0047), `dom(Σ.C) ⊆ dom(Σ'.C)`; hence `a ∈ dom(Σ'.C)`. (2) Since `⟦σ⟧` is a state-independent function of σ alone (ASN-0053), the membership `a ∈ ⟦σ⟧` is preserved. (3) Therefore `a ∈ ⟦σ⟧ ∩ dom(Σ'.C)`. (4) By O5, `origin'(a) = origin(a) = o`. (5) Hence `o ∈ origins_I(Σ', σ)`. Since `o` was arbitrary, `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)`. ∎

**Claim O6★ (Multi-step monotonic growth).** *For any reachable state sequence `Σ →* Σ'` and any I-span `σ`: `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)`.*

*Derivation.* Fix any `o ∈ origins_I(Σ, σ)`; there is `a ∈ ⟦σ⟧ ∩ dom(Σ.C)` with `origin(a) = o`. (1) Store Monotonicity★ (ASN-0098) gives `dom(Σ.C) ⊆ dom(Σ'.C)` across `Σ →* Σ'` (the multi-step companion of the single-step P0 invoked in O6), so `a ∈ dom(Σ'.C)`. (2) Since `⟦σ⟧` is a state-independent function of σ alone (ASN-0053), `a ∈ ⟦σ⟧ ∩ dom(Σ'.C)`. (3) By O5★, `origin'(a) = origin(a) = o`. (4) Hence `o ∈ origins_I(Σ', σ)`. Since `o` was arbitrary, `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)`. ∎

New allocations within σ may introduce new origins, but existing origins cannot be reassigned or removed.

The V-span lift is more nuanced. `origins_V(Σ, d, σ)` depends on the arrangement `M(d)`, which is mutable under editing operations. A passage transcluded today may be removed tomorrow, in which case the corresponding source document is no longer represented at those V-positions. The strongest claim we can make about V-span permanence requires fixing the arrangement:

**Claim O7 (V-span stability under fixed arrangement).** *For any reachable `Σ → Σ'` such that `M'(d) ↾ ⟦σ⟧ = M(d) ↾ ⟦σ⟧`, we have `origins_V(Σ', d, σ) = origins_V(Σ, d, σ)`.*

*Derivation.* By (F1), `origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }`. (1) The frame condition `M'(d) ↾ ⟦σ⟧ = M(d) ↾ ⟦σ⟧` gives `dom(M'(d) ↾ ⟦σ⟧) = dom(M(d) ↾ ⟦σ⟧)`; that is, `⟦σ⟧ ∩ dom(M'(d)) = ⟦σ⟧ ∩ dom(M(d))`. The two indexing sets are identical. (2) For each `v` in the common indexing set, the function values agree: `M'(d)(v) = M(d)(v)`. (3) Let `a = M(d)(v) = M'(d)(v)`. Since `v ∈ dom(M(d))`, S3★-aux (SubspaceExhaustiveness, ASN-0047) gives `subspace(v) ∈ {s_C, s_L}`; with this antecedent discharged, S3★ (GeneralizedReferentialIntegrity, ASN-0047) gives `a ∈ dom(Σ.C) ∪ dom(Σ.L)` (the `s_C` clause placing `a ∈ dom(C)`, the `s_L` clause placing `a ∈ dom(L)`). (4) P3 (ArrangementMutabilityOnly, ASN-0047) — `dom(C) ⊆ dom(C')` and `dom(L) ⊆ dom(L')` — gives `a ∈ dom(Σ'.C) ∪ dom(Σ'.L)`. (5) By O5, `origin'(a) = origin(a)`. (6) Hence `origin'(M'(d)(v)) = origin(M(d)(v))` for every `v` in the common indexing set. (7) The two sets `origins_V(Σ', d, σ)` and `origins_V(Σ, d, σ)` are constructed by applying the same operation to the same data, and therefore coincide. ∎

**Definition (WF_V — V-span well-formedness).** *For a state Σ, document `d`, and V-span `σ = (u, ℓ)`, the predicate `WF_V(Σ, d, σ)` is the conjunction:*

> *(i) `d ∈ Σ.E_doc` — the source document is allocated (ASN-0047);*
> *(ii) σ is level-uniform: `#u = #ℓ` (S6, ASN-0053);*
> *(iii) `V_{u₁}(d) ≠ ∅` — the subspace identified by `u₁` is non-empty in `d`'s arrangement;*
> *(iv) T12 holds for `(u, ℓ)`: `Pos(ℓ)` and `actionPoint(ℓ) ≤ #u` (ASN-0034);*
> *(v) `#ℓ = #u = m`, where `m` is the common V-position depth in subspace `u₁` of `d` (S8-depth, ASN-0036);*
> *(vi) the range condition `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))`.*

`WF_V(Σ, d, σ)` collects the conditions under which the V-span origin set (F1) is well-defined: a level-uniform span confined to a single subspace whose denoted positions are all present in `d`'s arrangement.

An arrangement *extension* on `d` preserves the per-subspace common depth that S8-depth fixes: a subspace non-empty before the extension retains the same depth after it.

**Lemma SDP (Subspace-depth preservation under arrangement extension).** *Let `Σ → Σ'` be a reachable arrangement-extension transition on `d` — K.μ⁺ on `d` or K.μ⁺_L on `d` — and let `S ∈ {s_C, s_L}` be a subspace with `V_S(d)|_Σ ≠ ∅`. Then `V_S(d)|_Σ ⊆ V_S(d)|_{Σ'}`, and the common depth that S8-depth (ASN-0036) fixes on `V_S(d)` is the same at both states: writing `m` for the depth at Σ and `m'` for the depth at Σ', `m' = m`.*

*Derivation.* Both K.μ⁺ and K.μ⁺_L extend `dom(M(d)) ⊆ dom(M'(d))` while preserving prior mappings — K.μ⁺ via ASN-0047's extension clause, K.μ⁺_L via its `M'(d) = M(d) ∪ {v_ℓ ↦ ℓ}` effect. The projections `subspace(v) = v_1` (Subspace, ASN-0036) and `#v` read only the component sequence of the tumbler `v` itself, independent of state: the same tumbler has the same first component and the same length at Σ and Σ'. Hence every `v ∈ V_S(d)|_Σ` — already in `dom(M(d))`, still in `dom(M'(d))` by the extension clause, with subspace identifier `S` and depth unchanged — inhabits `V_S(d)|_{Σ'}`, giving `V_S(d)|_Σ ⊆ V_S(d)|_{Σ'}`. Since `V_S(d)|_Σ ≠ ∅`, the superset `V_S(d)|_{Σ'}` is non-empty, so S8-depth at Σ' fixes a single common depth `m'` across it. The pre-state positions all have depth `m` (S8-depth at Σ) and lie in `V_S(d)|_{Σ'}`; S8-depth forcing a single value across the set gives `m' = m`. ∎

The complementary case yields a parallel *preservation* result for arrangement *extensions*: when `M(d)` grows by K.μ⁺ and the V-span σ is well-formed at the pre-state, the reported origins are exactly preserved — not merely non-decreasing.

**Claim O11 (V-span preservation under K.μ⁺).** *For any reachable K.μ⁺ transition `Σ → Σ'` extending `M(d)` and any V-span `σ` over `d` with `WF_V(Σ, d, σ)` — in particular conjunct (vi), `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))`: `origins_V(Σ, d, σ) = origins_V(Σ', d, σ)`.*

*Derivation.* We show inclusion in both directions.

(⊆): Fix `o ∈ origins_V(Σ, d, σ)`. By (F1), there exists `v ∈ ⟦σ⟧ ∩ dom(M(d))` with `origin(M(d)(v)) = o`. (1) K.μ⁺ (ArrangementExtension, ASN-0047) extends `dom(M(d)) ⊆ dom(M'(d))` while preserving existing mappings: `(A v : v ∈ dom(M(d)) : M'(d)(v) = M(d)(v))`. Hence `v ∈ dom(M'(d))` and `M'(d)(v) = M(d)(v)`. (2) Since `⟦σ⟧` is state-independent (ASN-0053), `v ∈ ⟦σ⟧ ∩ dom(M'(d))`. (3) Let `a = M(d)(v) = M'(d)(v)`. Since `v ∈ dom(M(d))`, S3★-aux (SubspaceExhaustiveness, ASN-0047) gives `subspace(v) ∈ {s_C, s_L}`; with this antecedent discharged, S3★ (GeneralizedReferentialIntegrity, ASN-0047) gives `a ∈ dom(C) ∪ dom(L)`. (4) By O5, `origin'(a) = origin(a) = o`. (5) Hence `o ∈ origins_V(Σ', d, σ)`.

(⊇): Fix `o ∈ origins_V(Σ', d, σ)`. By (F1), there exists `v ∈ ⟦σ⟧ ∩ dom(M'(d))` with `origin(M'(d)(v)) = o`. Two cases.

*Case (i): `v ∈ dom(M(d))`.* Then by K.μ⁺'s mapping preservation, `M'(d)(v) = M(d)(v)`. Hence `v ∈ ⟦σ⟧ ∩ dom(M(d))` with `origin(M(d)(v)) = o`, so `o ∈ origins_V(Σ, d, σ)`.

*Case (ii): `v ∈ dom(M'(d)) ∖ dom(M(d))`.* We show this case is impossible. By ContentSubspaceRestriction (K.μ⁺ amendment, ASN-0047), every newly added V-position has `subspace(v) = s_C`. By precondition (v) of SHOWORIGIN_V, σ is level-uniform with common depth `m ≥ 2` (the bound inherited from S8a, ASN-0036). Two sub-cases by `subspace(u)`:

*Sub-case (a): `subspace(u) = s_C`.* By SDP applied with `S = s_C` (non-empty at Σ by precondition (iii)) to the K.μ⁺ step, the common content-subspace depth is preserved: `m' = m`. The newly added position `v` has `subspace(v) = s_C` and `v ∈ dom(M'(d))`, so `v ∈ V_{s_C}(d)|_{Σ'}`, whence `#v = m' = m` by S8-depth at Σ'. Combined with `v ∈ ⟦σ⟧` (which unfolds to `u ≤ v < reach(σ)`), precondition (vi) at Σ gives `v ∈ dom(M(d))`, contradicting `v ∉ dom(M(d))`.

*Sub-case (b): `subspace(u) = s_L`.* By C0a (PrefixConfinement, ASN-0058) applied to level-uniform σ with `m ≥ 2`, every `t ∈ ⟦σ⟧` satisfies `t_1 = u_1 = s_L`. The newly added position has `subspace(v) = s_C` (established at the start of Case (ii) via ContentSubspaceRestriction, K.μ⁺ amendment, ASN-0047). Since `s_C ≠ s_L` (by SC-NEQ, ASN-0047), `subspace(v) ≠ s_L`. So `v ∉ ⟦σ⟧`, contradicting `v ∈ ⟦σ⟧ ∩ dom(M'(d))`.

Both sub-cases yield contradictions, so case (ii) is impossible. Hence every `o ∈ origins_V(Σ', d, σ)` is in `origins_V(Σ, d, σ)`. Combined with (⊆), the two sets are equal. ∎

The link-subspace extension K.μ⁺_L is a formally distinct transition with its own precondition (`ℓ ∈ dom(L)`, `origin(ℓ) = d`, `ℓ ∉ ran(M(d))`) and effect (adding a single fresh V-position `v_ℓ`). Its parallel claim:

**Claim O11' (V-span preservation under K.μ⁺_L).** *For any reachable K.μ⁺_L transition `Σ → Σ'` extending `M(d)` and any V-span `σ` over `d` with `WF_V(Σ, d, σ)`: `origins_V(Σ, d, σ) = origins_V(Σ', d, σ)`.*

*Derivation.* We show inclusion in both directions.

(⊆): K.μ⁺_L (LinkSubspaceExtension, ASN-0047) has effect `M'(d) = M(d) ∪ {v_ℓ ↦ ℓ}` with `dom(M'(d)) = dom(M(d)) ∪ {v_ℓ} ⊃ dom(M(d))`. The strict containment forces `v_ℓ ∉ dom(M(d))` directly — equality of `dom(M(d)) ∪ {v_ℓ}` with `dom(M(d))` would collapse the union and contradict strictness — so the effect preserves `M(d)(v)` at every prior `v ∈ dom(M(d))`. Fix `o ∈ origins_V(Σ, d, σ)`. By (F1), there exists `v ∈ ⟦σ⟧ ∩ dom(M(d))` with `origin(M(d)(v)) = o`. Then `v ∈ dom(M'(d))` and `M'(d)(v) = M(d)(v)`; with `⟦σ⟧` state-independent (ASN-0053), `v ∈ ⟦σ⟧ ∩ dom(M'(d))`. Let `a = M(d)(v) = M'(d)(v)`; since `v ∈ dom(M(d))`, S3★-aux (SubspaceExhaustiveness, ASN-0047) gives `subspace(v) ∈ {s_C, s_L}`, and with this antecedent discharged S3★ (GeneralizedReferentialIntegrity, ASN-0047) gives `a ∈ dom(C) ∪ dom(L)`; by O5, `origin'(a) = origin(a) = o`. Hence `o ∈ origins_V(Σ', d, σ)`.

(⊇): Fix `o ∈ origins_V(Σ', d, σ)`. By (F1), there exists `v ∈ ⟦σ⟧ ∩ dom(M'(d))` with `origin(M'(d)(v)) = o`.

*Case (i): `v ∈ dom(M(d))`.* As above, `M'(d)(v) = M(d)(v)`, so `o ∈ origins_V(Σ, d, σ)`.

*Case (ii): `v ∈ dom(M'(d)) ∖ dom(M(d)) = {v_ℓ}`.* By K.μ⁺_L's V-position precondition, `subspace(v_ℓ) = s_L` and `#v_ℓ ≥ 2` (either `#v_ℓ = m_L(d)` when `V_{s_L}(d) ≠ ∅`, or `#v_ℓ` is the freely chosen first-link depth `≥ 2` of `ValidFirstLinkPosition` when `V_{s_L}(d) = ∅`; LinkSubspaceDepth, ASN-0047). By precondition (v) of SHOWORIGIN_V on σ, σ is level-uniform with common depth `m ≥ 2`. Two sub-cases:

*Sub-case (a): `subspace(u) = s_C`.* By C0a (PrefixConfinement, ASN-0058), every `t ∈ ⟦σ⟧` satisfies `t_1 = u_1 = s_C`. But `subspace(v_ℓ) = s_L ≠ s_C` (SC-NEQ, ASN-0047). So `v_ℓ ∉ ⟦σ⟧`, contradicting `v = v_ℓ ∈ ⟦σ⟧`.

*Sub-case (b): `subspace(u) = s_L`.* By SDP applied with `S = s_L` (non-empty at Σ by precondition (iii), since `subspace(u) = s_L`) to the K.μ⁺_L step, the common link-subspace depth is preserved: `m' = m`. The newly added position `v_ℓ` has `subspace(v_ℓ) = s_L` and `v_ℓ ∈ dom(M'(d))`, so `v_ℓ ∈ V_{s_L}(d)|_{Σ'}`, whence `#v_ℓ = m' = m` by S8-depth at Σ'. With `v_ℓ ∈ ⟦σ⟧`, `#v_ℓ = m`, and `u ≤ v_ℓ < reach(σ)`, precondition (vi) at Σ gives `v_ℓ ∈ dom(M(d))`, contradicting `v_ℓ ∉ dom(M(d))`.

Both sub-cases contradict, so case (ii) is impossible. Combined with case (i), `origins_V(Σ', d, σ) ⊆ origins_V(Σ, d, σ)`; with (⊆) we have equality. ∎

Well-formedness of σ is itself preserved across an arrangement extension: if `WF_V` holds at the pre-state, it holds again at the post-state.

**Corollary O11.1 (Well-formedness preservation under arrangement extension).** *Let σ be a V-span over `d` with `WF_V(Σ, d, σ)`. For any reachable arrangement-extension transition `Σ → Σ'` — K.μ⁺ on `d` or K.μ⁺_L on `d` — `WF_V(Σ', d, σ)` holds.*

*Derivation.* Conjuncts (ii) (level-uniformity `#u = #ℓ`) and (iv) (T12 conjuncts on `(u, ℓ)`) are structural properties of `(u, ℓ)` alone and state-independent — they read only the tumblers `u` and `ℓ`, not any state component. Conjunct (v), `#ℓ = #u = m`, is *not* purely structural: it splits into the structural length identity `#ℓ = #u` (a comparison on `u` and `ℓ` alone, state-independent) and the coupling `#u = m`, where `m` is the common V-position depth that S8-depth fixes on `V_{u₁}(d)` — a property of the arrangement `M(d)`, hence state-dependent. The structural identity carries to Σ' unchanged; the coupling requires proof and is discharged below. Conjunct (i) `d ∈ Σ.E_doc` lifts to `d ∈ Σ'.E_doc` by P1 (EntityPermanence, ASN-0047). The remaining state-dependent conjuncts (iii), (v), and (vi) we discharge in turn:

- *Conjunct (iii) (`V_{u₁}(d) ≠ ∅`):* K.μ⁺ / K.μ⁺_L gives `dom(M(d)) ⊆ dom(M'(d))` with prior mappings preserved (K.μ⁺ via ASN-0047's extension clause; K.μ⁺_L via its `M'(d) = M(d) ∪ {v_ℓ ↦ ℓ}` effect). Since `subspace(v) = v_1` is a structural projection independent of state, every `v ∈ V_{u₁}(d)|_Σ` retains its subspace identifier at Σ' and lies in `V_{u₁}(d)|_{Σ'}`. Non-emptiness is preserved.

- *Conjunct (v) (common-depth coincidence):* the structural identity `#ℓ = #u` is state-independent and carries unchanged; what requires proof is the coupling `#u = m`. Conjunct (v) holds at Σ' iff the common depth `m'` that S8-depth fixes on `V_{u₁}(d)|_{Σ'}` again equals `#u`. By SDP applied with `S = u₁` (non-empty at Σ by conjunct (iii)) to the extension step — K.μ⁺ or K.μ⁺_L on `d`, either kind discharging SDP's hypothesis — the common depth in subspace `u₁` is preserved: `m' = m = #u` (with `m = #u` by conjunct (v) at Σ). Hence conjunct (v) holds at Σ'.

- *Conjunct (vi) (range condition):* with `m = #u` fixed (conjunct (v), just established at Σ'), the set `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m}` is determined by σ and the integer `m` alone — state-independent — and is the same set at Σ and Σ'. At Σ it is contained in `dom(M(d))`; by `dom(M(d)) ⊆ dom(M'(d))` (the K.μ⁺ / K.μ⁺_L extension clause), the same set is contained in `dom(M'(d))` at Σ'. This monotonicity step is all conjunct (vi) requires. Hence conjunct (vi) holds at Σ'. ∎

**Claim O11★★ (Multi-step V-span preservation under mixed K.μ⁺/K.μ⁺_L chain).** *For any reachable state sequence `Σ →* Σ'` in which every `M(d)`-modifying step is either K.μ⁺ on `d` or K.μ⁺_L on `d` (i.e., no K.μ⁻ on `d` and no K.μ~ on `d` along the chain), and any V-span `σ` over `d` with `WF_V(Σ, d, σ)`: `origins_V(Σ, d, σ) = origins_V(Σ', d, σ)`.*

*Derivation.* By induction on the length `n ≥ 0` of the chain `Σ = Σ_0 → Σ_1 → ⋯ → Σ_n = Σ'`. *Base* (`n = 0`): `Σ' = Σ`, so both sides coincide. *Step* (`n ≥ 1`): the inductive hypothesis applied to `Σ →* Σ_{n-1}` gives `origins_V(Σ, d, σ) = origins_V(Σ_{n-1}, d, σ)`. Well-formedness of σ at `Σ_{n-1}` is preserved by induction along the chain using Corollary O11.1 at each `M(d)`-modifying step (K.μ⁺ or K.μ⁺_L on `d`) and the state-independence of σ's structural conjuncts at each non-`M(d)`-modifying step (where the range condition and common depth are inherited unchanged because `M(d)` is unchanged). The one conjunct that is neither structural nor a function of `M(d)` — conjunct (i), `d ∈ Σ.E_doc` — is preserved at every step by P1 (EntityPermanence, ASN-0047), which Corollary O11.1 already invokes for the extension steps; so well-formedness carries through the non-`M(d)` steps in full. Three sub-cases for the final step `Σ_{n-1} → Σ_n`:

(i) The step is K.μ⁺ on `d`. By O11 applied at `Σ_{n-1} → Σ_n` (with σ well-formed at `Σ_{n-1}`), `origins_V(Σ_{n-1}, d, σ) = origins_V(Σ_n, d, σ)`.

(ii) The step is K.μ⁺_L on `d`. By O11' applied at `Σ_{n-1} → Σ_n` (with σ well-formed at `Σ_{n-1}`), `origins_V(Σ_{n-1}, d, σ) = origins_V(Σ_n, d, σ)`.

(iii) The step does not modify `M(d)`. This is the complement of sub-cases (i) and (ii) — the sub-cases partition every transition by whether it modifies `M(d)`. In every instance of this class `M(d)|_{Σ_n} = M(d)|_{Σ_{n-1}}`, so the restriction `M(d) ↾ ⟦σ⟧` is unchanged; O7 (V-span stability under fixed arrangement) at `Σ_{n-1} → Σ_n` gives `origins_V(Σ_{n-1}, d, σ) = origins_V(Σ_n, d, σ)`.

Composing with the inductive hypothesis by transitivity of equality, `origins_V(Σ, d, σ) = origins_V(Σ_n, d, σ)`. ∎

O11, O11', and O11★★ together cover every arrangement-extending transition. The non-extension transitions K.μ⁻ and K.μ~ behave differently. We record the failure modes as labeled negative claims.

**Claim O13 (K.μ⁻ admissibility loss).** *There exist Σ, a V-span σ over `d` with `WF_V(Σ, d, σ)`, and a reachable K.μ⁻ transition `Σ → Σ'` on `d` such that `WF_V(Σ', d, σ)` fails at conjunct (vi) — equivalently, `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊄ dom(M'(d))`. Consequently, no K.μ⁻ analogue of O11 / O11' / O11★★ holds — the V-span operation is no longer admissible at the post-state on the original input, so preservation of `origins_V` is not even formulable.*

*Failure condition.* Conjunct (vi) — `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))` — ceases to hold whenever the K.μ⁻ retention parameters drop V-positions strictly inside `⟦σ⟧` from `dom(M(d))`. By K.μ⁻'s constructive retention `R = ⋃_{S ∈ {s_C, s_L}} {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` (foundation ASN-0047), this happens precisely when some position in `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))` carries a sequential index `k` greater than `n'_S` in its subspace `S`. K.μ⁻'s strict-contraction precondition `(E S ∈ {s_C, s_L} : V_S(d) ≠ ∅ : n'_S < n_S)` guarantees at least one subspace shrinks strictly, and any contraction whose strict shrinkage falls inside `⟦σ⟧` witnesses admissibility loss.

*Consequence.* SHOWORIGIN_V's domain of admissibility shrinks as `dom(M(d))` shrinks. A reader at Σ' must pose a smaller still-admissible query.

**Claim O14 (K.μ~ non-preservation).** *There exist Σ, a reachable K.μ~ transition `Σ → Σ'` on `d`, and a V-span `σ` over `d` such that σ is well-formed at both Σ and Σ', yet:*

> *(i) `origins_V(Σ, d, σ) ⊄ origins_V(Σ', d, σ)`, and*
> *(ii) `origins_V(Σ', d, σ) ⊄ origins_V(Σ, d, σ)`.*

*That is, neither set is a subset of the other; no monotonicity claim parallel to O11 / O11' / O11★★ holds for K.μ~.*

*Failure mechanism.* K.μ~ realises the bijection equation `M'(d)(π(v)) = M(d)(v)` with `π ≠ id`; equivalently `M'(d)(v) = M(d)(π⁻¹(v))`, which reassigns function values at individual V-positions. The argument used in O11's derivation — invoking K.μ⁺'s mapping-preservation clause `M'(d)(v) = M(d)(v)` for `v ∈ dom(M(d))` — has no K.μ~ analogue, because K.μ~ permits exactly the opposite relation.

## Span containment monotonicity

Nelson is explicit that the system must distinguish no scale below *any specific word or character*: the mechanism that names the home of a million-character chapter must name the home of a single character [Q8]. Because the pointwise projection O3 performs the work uniformly, with no procedural case distinction on size, attribution at the paragraph level reduces to attribution at the character level — and O8 records the elementary set-inclusion consequence: enlarging the span never loses an origin.

**Claim O8 (I-span containment monotonicity).** *For I-spans `σ₁, σ₂` with `⟦σ₁⟧ ⊆ ⟦σ₂⟧`: `origins_I(Σ, σ₁) ⊆ origins_I(Σ, σ₂)`.*

*Derivation.* Fix `o ∈ origins_I(Σ, σ₁)`. By definition, there exists `a ∈ ⟦σ₁⟧ ∩ dom(Σ.C)` with `origin(a) = o`. By hypothesis `⟦σ₁⟧ ⊆ ⟦σ₂⟧`, so `a ∈ ⟦σ₂⟧`. Since `a ∈ dom(Σ.C)` is unchanged, `a ∈ ⟦σ₂⟧ ∩ dom(Σ.C)`, and `origin(a) = o ∈ origins_I(Σ, σ₂)`. ∎

The smallest case is the singleton: for any `a ∈ dom(C)`, the singleton span (containing only `a`) yields `origins_I = {origin(a)}`. The largest case is unbounded — by T0(b) of ASN-0034, there is no maximum tumbler length, so spans can be arbitrarily wide.

The V-span counterpart follows by the same set-inclusion argument, routed through (F1) instead of the definition of `origins_I`. The hypothesis is denotational containment of the spans; the arrangement `M(d)` is held fixed.

**Claim O12 (V-span containment monotonicity).** *For V-spans `σ₁, σ₂` over the same document `d` with `⟦σ₁⟧ ⊆ ⟦σ₂⟧`: `origins_V(Σ, d, σ₁) ⊆ origins_V(Σ, d, σ₂)`.*

*Derivation.* Fix `o ∈ origins_V(Σ, d, σ₁)`. By (F1), there exists `v ∈ ⟦σ₁⟧ ∩ dom(M(d))` with `origin(M(d)(v)) = o`. By hypothesis `⟦σ₁⟧ ⊆ ⟦σ₂⟧`, so `v ∈ ⟦σ₂⟧`. Since `v ∈ dom(M(d))` is unchanged, `v ∈ ⟦σ₂⟧ ∩ dom(M(d))`, and `origin(M(d)(v)) = o ∈ origins_V(Σ, d, σ₂)`. ∎

## Identity, not equivalence

The system distinguishes *wrote the same words* from *quoted from the original* [Q1, Q9]. Two documents that independently produce identical text have distinct I-addresses; transcluded content shares an I-address with its source. SHOWORIGIN tracks the I-address, so it reports identity-of-origin, not equivalence-of-text.

**Claim O9 (Origin tracks creation, not content).** *Let `a₁, a₂ ∈ dom(C)` with `C(a₁) = C(a₂)` (identical content values). If `a₁` and `a₂` were produced by allocation events under distinct documents `d₁` and `d₂` (with `d₁ ≠ d₂`), then `origin(a₁) ≠ origin(a₂)`.*

*Derivation.* (1) By hypothesis, the allocation event producing `a₁` was performed by document `d₁`; by S7a (DocumentScopedAllocation, ASN-0036), `origin(a₁) = d₁`. (2) Similarly, `origin(a₂) = d₂`. (3) By hypothesis, `d₁ ≠ d₂`. (4) By S7d (DocumentAllocationDiscipline, ASN-0036), distinct documents have distinct document-level tumblers; hence `d₁ ≠ d₂` at the tumbler level. (5) Therefore `origin(a₁) = d₁ ≠ d₂ = origin(a₂)`. The hypothesis `C(a₁) = C(a₂)` does not enter the derivation: the conclusion holds regardless of whether content values agree. ∎

The stronger fact, that the addresses themselves differ (`a₁ ≠ a₂`), is supplied independently by S4 (OriginBasedIdentity, ASN-0036) — distinct allocation events produce distinct addresses, with or without distinct documents. But the relevant point for SHOWORIGIN is the document-level distinction at the projection: reading the same value back from two addresses tells you the bytes match; it does not tell you they came from the same place.

## The operation

We can now specify SHOWORIGIN as a non-state-modifying operation in two arities.

**SHOWORIGIN over an I-span.**
- *Preconditions*: `σ = (s, ℓ)` is a well-formed I-span — explicitly, the conjuncts of T12 (SpanWellDefinedness, ASN-0034): (i) `s ∈ T`; (ii) `ℓ ∈ T`; (iii) `Pos(ℓ)` (TA-Pos, ASN-0034); (iv) `actionPoint(ℓ) ≤ #s` (ActionPoint, ASN-0034).
- *Postcondition*: the result is `origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(Σ.C) }`.
- *Frame*: `Σ' = Σ`. The operation does not modify `C`, `L`, `E`, `M`, or `R`.

**SHOWORIGIN over a content reference.**
- *Preconditions*: `WF_V(Σ, d, σ)` (the V-span well-formedness predicate defined above, conjuncts (i)–(vi)). The subspace identifier `u₁` may be either `s_C` (content) or `s_L` (link); `origin` is total on `dom(C) ∪ dom(L)`. The postcondition is well-formed in either case because each indexed value lands in that domain: for every `v ∈ ⟦σ⟧ ∩ dom(M(d))`, S3★-aux (SubspaceExhaustiveness, ASN-0047) gives `subspace(v) ∈ {s_C, s_L}`, and with this antecedent discharged S3★ (GeneralizedReferentialIntegrity, ASN-0047) places `M(d)(v) ∈ dom(C) ∪ dom(L)`, so `origin(M(d)(v))` is defined (with the link case trivializing to `{d}` by CL-OWN).
- *Postcondition*: the result is `origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }` (form (F1)).
- *Frame*: `Σ' = Σ`.

**Claim O10 (Read-only frame; idempotence).** *Let `op` be either SHOWORIGIN_I or SHOWORIGIN_V. Then for any Σ in which the precondition holds: (a) `op(Σ) = (Σ', result)` with `Σ' = Σ`; (b) two consecutive applications at the same state yield identical results.*

*Derivation.* (a) The frame clause of the operation specification declares `Σ' = Σ` explicitly — every component (`C`, `L`, `E`, `M`, `R`) is unchanged. This is the definition of the operation. (b) Let `op(Σ) = (Σ, r₁)` be the first application. By (a), the post-state is `Σ`. The second application is `op(Σ) = (Σ, r₂)`. The operation's result is a pure function of `Σ`, σ, and (for SHOWORIGIN_V) `d` (because the result is defined by an expression in (F1) or its I-span analogue, which mentions only state-derivable sets and the projection `origin`). Applying the same function to the same arguments yields the same value: `r₁ = r₂`. ∎

Idempotence is essential: SHOWORIGIN must be a passive observation. Without the read-only frame, the act of asking would alter the answer, which would defeat the purpose of having an answer at all.

### Edge cases

Each of the following configurations satisfies the operation precondition; we record what the postcondition delivers in each.

*Empty intersection (I-span).* When `⟦σ⟧ ∩ dom(Σ.C) = ∅` — the well-formed span happens to contain no allocated content addresses — the postcondition expression evaluates to `∅`. The operation succeeds and returns the empty set as a legitimate output. This case is not exceptional: by O6, an empty result at Σ may become non-empty at some `Σ'` if new content is allocated within σ.

*Singleton I-span.* For any `a ∈ dom(Σ.C)`, the span `σ_a = (a, [0, ..., 0, 1])` of length `#a` with all-zero prefix and final component 1 satisfies T12: `Pos(ℓ)` holds, and `actionPoint(ℓ) = #a ≤ #a`. By TA-strict (ASN-0034), `a ⊕ ℓ > a`, so `a ∈ ⟦σ_a⟧`. The operationally relevant fact is the *single-origin* result `origins_I(Σ, σ_a) = {origin(a)}` — not the strict-singleton intersection `⟦σ_a⟧ ∩ dom(C) = {a}`. We establish single-origin directly, since it is robust to whatever content addresses may inhabit the span: we show that every `b ∈ ⟦σ_a⟧ ∩ dom(C)` satisfies `origin(b) = origin(a)`. Since `a` itself is such a `b`, the set `origins_I(Σ, σ_a) = {origin(b) : b ∈ ⟦σ_a⟧ ∩ dom(C)}` then collapses to the singleton `{origin(a)}`. Fix `b ∈ ⟦σ_a⟧ ∩ dom(C)`. We dispose of the three length cases in turn.

*Case `#b < #a` is excluded by T1.* Suppose `#b < #a`. Since `b ∈ ⟦σ_a⟧`, T12 (SpanWellDefinedness, ASN-0034) — whose denotation is `{t ∈ T : a ≤ t < a ⊕ ℓ}` — gives `a ≤ b`, i.e. `a < b ∨ a = b` (T1 (d), ASN-0034). Equality `a = b` is ruled out by T3 of ASN-0034 (which requires `#a = #b`, contradicting `#b < #a`), leaving `a < b`. T1 case (ii) requires `a` to be a proper prefix of `b`, i.e. `#a < #b` — contradicting `#b < #a`. T1 case (i) requires some `k ≤ min(#a, #b) = #b` with `a_k < b_k` and agreement on positions `1, ..., k − 1`; since `k ≤ #b < #a`, position `k` falls in TumblerAdd's prefix-copy region for `a ⊕ ℓ`, giving `(a ⊕ ℓ)_k = a_k < b_k`. By T1 case (i) at the same `k`, `a ⊕ ℓ < b` — contradicting `b < a ⊕ ℓ`. Hence `#b ≥ #a`.

With `#b ≥ #a` in hand, we derive `b`'s component-wise agreement with `a` on positions 1 through `#a` in three steps: prefix agreement at positions `1 ≤ i < #a`, then the lower and upper bounds on position `#a`, then closure via T0 discreteness.

*Prefix agreement at positions `1 ≤ i < #a`.* Suppose for contradiction that some `i ∈ {1, ..., #a − 1}` satisfies `a_i ≠ b_i`, and let `j` be the smallest such index — well-defined as `min S` where `S = {i ∈ ℕ : 1 ≤ i ≤ #a − 1 ∧ a_i ≠ b_i}` is a nonempty subset of ℕ, with minimum supplied by NAT-wellorder (ASN-0034). By minimality of `j`, `a_i = b_i` for `1 ≤ i < j`. Since `j < #a`, position `j` lies in the prefix-copy region of `a ⊕ ℓ`: by TumblerAdd (ASN-0034), `(a ⊕ ℓ)_j = a_j`. Two sub-cases by T0 trichotomy on `a_j` versus `b_j` (NAT-order, ASN-0034; the case `a_j = b_j` is excluded by the construction of `j`):

- *Sub-case `a_j < b_j`:* set `k = j` as the divergence position between `b` and `a ⊕ ℓ`. Agreement on `1, ..., j − 1` holds because `b_i = a_i = (a ⊕ ℓ)_i` for `i < j` (the first equality by minimality of `j`; the second by TumblerAdd's prefix-copy region). At position `j`, `b_j > a_j = (a ⊕ ℓ)_j`, and `j ≤ min(#b, #(a ⊕ ℓ)) = min(#b, #a) = #a` (using `#b ≥ #a`). By T1 case (i) at `k = j`, `(a ⊕ ℓ) < b` — contradicting `b < a ⊕ ℓ`.
- *Sub-case `a_j > b_j`:* set `k = j` as the divergence position between `b` and `a`. Agreement on `1, ..., j − 1` holds by minimality of `j`. At position `j`, `b_j < a_j`, with `j ≤ min(#a, #b) = #a`. By T1 case (i) at `k = j`, `b < a` — contradicting `a ≤ b`.

Both sub-cases yield contradictions, so no such `j` exists; `a_i = b_i` for all `1 ≤ i < #a`.

*Lower bound `a_{#a} ≤ b_{#a}` from `a ≤ b`.* If `a = b`, then `a_{#a} = b_{#a}` directly (with `#a = #b` by T3, ASN-0034), so `a_{#a} ≤ b_{#a}`. Otherwise `a < b`. With prefix agreement on `1, ..., #a − 1` just established, T1's first-divergence position between `a` and `b` cannot fall at any `i < #a`. Two cases remain by T1's case structure:
- *T1 case (i) at `k = #a`:* gives `a_{#a} < b_{#a}`. By NAT-order's `≤`-definition, `a_{#a} ≤ b_{#a}`.
- *T1 case (ii) (proper-prefix):* requires `#a + 1 ≤ #b` and agreement on positions `1, ..., #a`. The agreement clause includes position `#a`, so `a_{#a} = b_{#a}`, hence `a_{#a} ≤ b_{#a}`.

Either way, `a_{#a} ≤ b_{#a}`.

*Upper bound `b_{#a} < a_{#a} + 1` from `b < a ⊕ ℓ`.* The result-length identity `#(a ⊕ ℓ) = #ℓ = #a` (TumblerAdd / TA0, ASN-0034) combined with `#b ≥ #a` excludes T1 case (ii) as the source of `b < a ⊕ ℓ`: T1 case (ii) would require `#b + 1 ≤ #(a ⊕ ℓ) = #a`, i.e., `#b < #a`, contradicting `#b ≥ #a`. Hence `b < a ⊕ ℓ` holds by T1 case (i) at some `k ≤ min(#b, #(a ⊕ ℓ)) = #a`. With prefix agreement of `b` with `a` on `1, ..., #a − 1` (just established) and TumblerAdd's prefix-copy region giving `(a ⊕ ℓ)_i = a_i` for `1 ≤ i < #a`, we have `b_i = a_i = (a ⊕ ℓ)_i` for `1 ≤ i < #a`. The first divergence between `b` and `a ⊕ ℓ` therefore cannot fall at any `i < #a`. Hence `k = #a`, and T1 case (i) at `k = #a` gives `b_{#a} < (a ⊕ ℓ)_{#a} = a_{#a} + 1` (the last equality by TumblerAdd's action-point component, since `ℓ_{#a} = 1`).

*Squeeze closure via T0 discreteness.* Combining `a_{#a} ≤ b_{#a}` and `b_{#a} < a_{#a} + 1` — both inequalities on ℕ since components inhabit ℕ by T0 — NAT-discrete (ASN-0034) gives `b_{#a} = a_{#a}`. NAT-discrete's statement `(A m, n ∈ ℕ :: m ≤ n < m + 1 ⟹ n = m)`, instantiated at `m = a_{#a}` and `n = b_{#a}`, directly yields the conclusion: no natural number lies strictly between `a_{#a}` and `a_{#a} + 1`.

Combining the three steps, `b` agrees with `a` at every position `1 ≤ i ≤ #a`.

*Case `#b = #a` gives `b = a` directly* by T3 (component-wise equality with equal length); hence `origin(b) = origin(a)` trivially.

*Case `#b > #a` yields `origin(b) = origin(a)` by structural prefix coincidence.* The T1 analysis above forces `b` to be a proper extension of `a` — `a` agrees with `b` on all positions `1, ..., #a`. By S7b (ASN-0036), `a ∈ dom(C)` requires `zeros(a) = 3`, and likewise `b ∈ dom(C)` requires `zeros(b) = 3`. From this we derive the document-level prefix coincidence in two steps. First, a zero-count balance argument places all of `b`'s zeros within positions `1, ..., #a`: `a`'s three zeros all lie within positions `1, ..., #a` (trivially, since `#a` is `a`'s length); `b` agrees with `a` on those positions, so `b` carries the same three zeros at the same positions within `1, ..., #a`; since `zeros(b) = 3` is the total zero count of `b`, no zero of `b` lies in positions `#a + 1, ..., #b`. Second, T4b's field-separator parse of `b` is therefore controlled entirely by `b`'s first three zeros — at the same positions as `a`'s — so `b`'s document-element separator (the third zero) coincides positionally with `a`'s. The document-level prefix `N(b).0.U(b).0.D(b)`, truncated at `b`'s third zero, is computed from positions of `b` that already lie within `a`, and it coincides with `N(a).0.U(a).0.D(a)`. Hence `origin(b) = origin(a)` by S7's structural projection (ASN-0036).

The result established is single-origin, not the strict-singleton intersection `⟦σ_a⟧ ∩ dom(C) = {a}`: whether or not a longer extension `b` inhabits the span, it contributes no new origin.

Combining the three cases — `#b < #a` excluded, `#b = #a` giving `origin(b) = origin(a)`, and `#b > #a` giving `origin(b) = origin(a)` — every `b ∈ ⟦σ_a⟧ ∩ dom(C)` satisfies `origin(b) = origin(a)`. Hence `origins_I(Σ, σ_a) = {origin(a)}`, a single document.

*Cross-subspace I-span.* If `⟦σ⟧` spans positions in both the content subspace (`subspace_I = s_C`) and the link subspace (`subspace_I = s_L`) — say, `s` has element field beginning with `s_C` and `reach(σ)` has element field beginning beyond `s_L` — then `⟦σ⟧ ∩ dom(Σ.C)` automatically excludes the link addresses (by L0 of ASN-0047, `dom(L) ⊆ {a : subspace_I(a) = s_L}` and `dom(C) ⊆ {a : subspace_I(a) = s_C}`, and L14 gives `dom(C) ∩ dom(L) = ∅`). The lift's intersection with `dom(C)` therefore silently drops link addresses; no link origins appear in `origins_I`. This is a deliberate choice of the I-span lift's definition: SHOWORIGIN over an I-span reports origins of content, not of links.

*V-span over link subspace.* When `u₁ = s_L`, the V-span lies in `d`'s link subspace. By S3★ (ASN-0047), every `v ∈ ⟦σ⟧ ∩ dom(M(d))` maps to a link `M(d)(v) ∈ dom(L)`; by CL-OWN (ASN-0047), `origin(M(d)(v)) = d`. So `origins_V(Σ, d, σ) = {d}`; on admissible inputs precondition (vi) forces the intersection to be non-empty (see the "Empty-restriction within a non-empty document" edge case below), so the empty-result branch does not arise. The V-span operation is uniformly defined across subspaces — `origin` is total on `dom(C) ∪ dom(L)` (per the extension introduced earlier) — but for the link case the answer is trivially the home document. This is the formal counterpart of Nelson's design principle that links are first-class transcludable material with home documents.

*Empty document arrangement (V-span).* If `M(d) = ∅`, then every subspace projection `V_S(d)` is empty for every `S`. The precondition (iii) of the V-span operation — `V_{u₁}(d) ≠ ∅` — fails, so the operation is *not admissible* on empty documents. There is no V-span over which to query origin because no V-positions exist. (Compare with the I-span case, where empty intersection produces a well-formed empty result; in the V-span case, the precondition itself is unsatisfiable.) The asymmetry reflects that V-span queries are document-relative — there must be at least one V-position to fix a depth `m`, by S8-depth's vacuity on empty subspaces.

*Empty-restriction within a non-empty document (V-span).* Can a well-formed V-span have empty intersection with `dom(M(d))`? No, and the structural reason is direct. By TA-strict (ASN-0034), `u = start(σ) ∈ ⟦σ⟧`. By precondition (v), `#u = m`, so `u` is a depth-`m` position in `⟦σ⟧`. Precondition (vi) — the range condition `{v ∈ T : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d))` — then gives `u ∈ dom(M(d))`. Hence `u ∈ ⟦σ⟧ ∩ dom(M(d))`, so the intersection is non-empty and the result has at least one origin. The empty-result case does not arise for well-formed V-spans.

### Weakest precondition for single-origin output

We compute two wp characterisations of what SHOWORIGIN reveals about state. The first concerns when SHOWORIGIN_I returns a single origin; for the I-span operation:

> `wp(SHOWORIGIN_I(σ), |result| = 1) = (⟦σ⟧ ∩ dom(C) ≠ ∅) ∧ (A a, b : a, b ∈ ⟦σ⟧ ∩ dom(C) : origin(a) = origin(b))`.

*Derivation.* The postcondition `|result| = 1` says the result set is a singleton. (1) `|origins_I(Σ, σ)| = 1` iff (a) `origins_I(Σ, σ) ≠ ∅` and (b) all elements of `origins_I(Σ, σ)` are equal. (2) Non-emptiness `origins_I(Σ, σ) ≠ ∅` iff `⟦σ⟧ ∩ dom(C) ≠ ∅`: the result is the image of the intersection under `origin`, so the result is empty iff the intersection is empty (the image of the empty set is empty; the image of a non-empty set under a total function is non-empty). (3) All elements equal iff every pair shares a common value: `(A a, b : a, b ∈ ⟦σ⟧ ∩ dom(C) : origin(a) = origin(b))`. (4) Conjoining: the wp is exactly the precondition that the intersection is non-empty and consists of addresses sharing a single origin. ∎

This wp is the exact characterisation of *single-origin spans*: any I-span whose allocated content lies wholly under one document's allocation prefix. Equivalently (by the partition O1), it is the precondition for `(⟦σ⟧ ∩ dom(C)) / ~_o` to be a one-element quotient.

A second wp characterises when a specific document is reported by SHOWORIGIN_V:

> `wp(SHOWORIGIN_V(d, σ), d_q ∈ result) = (E v : v ∈ ⟦σ⟧ ∩ dom(M(d)) : origin(M(d)(v)) = d_q)`.

*Derivation.* By (F1), `d_q ∈ origins_V(Σ, d, σ)` iff `(E v : v ∈ ⟦σ⟧ ∩ dom(M(d)) : origin(M(d)(v)) = d_q)`; since SHOWORIGIN_V's frame is `Σ' = Σ`, the post-state predicate equals the pre-state predicate, yielding the wp. ∎

That is, the precondition that some block of the C1a decomposition of `(d, σ)` is sourced from `d_q`. This delivers the operational use of SHOWORIGIN as a discovery probe: a reader who suspects that material from `d_q` is present in some region of `d`'s arrangement can confirm or refute by SHOWORIGIN's output.

## What SHOWORIGIN does not promise

The claims above bound what SHOWORIGIN guarantees. Three exclusions deserve explicit statement.

*Not historical containment.* SHOWORIGIN reports origin, not the set of documents that *have ever contained* the queried content. A document that once transcluded the content and then contracted its arrangement (`K.μ⁻`) is no longer represented by `M(d)` and does not appear in `origins_V`. Gregory's investigation of the spanfilade [Q17] confirms that the implementation's `find_documents_containing` mixes these two notions and returns a superset of currently-containing documents — a behaviour distinct from SHOWORIGIN.

*Not human authorship.* As Nelson notes [Q2], the User field of the tumbler identifies an owning *account*, not necessarily a known human. *John Doe publication* is permitted: anonymous and pseudonymous content has well-defined origin without revealing identity. SHOWORIGIN reports what the address structure encodes, and no more.

*Not transitive provenance.* The result names `d₁` (the original allocator), not the transclusion chain `d₁ → d₂ → ... → dₙ`.

## A worked example

We exhibit one concrete scenario that demonstrates the load-bearing facts: the multi-origin result, the negative witnesses for arrangement contraction (O13) and reordering (O14), and one weakest-precondition evaluation. The remaining claims are corollaries proved in the body; we do not replay them against the scenario.

*Initial state Σ₀.* Document `d₁` allocates content at I-addresses `[d₁.0.1.1]` through `[d₁.0.1.5]` containing the five characters of *Hello*. Document `d₂` arranges these five I-addresses at V-positions `[1,1,1]` through `[1,1,5]` in its own arrangement — a transclusion of the entire word, by reference. Document `d₃` similarly transcludes `d₂`'s arrangement of these positions, recording I-addresses `[d₁.0.1.1]` through `[d₁.0.1.5]` at its own V-positions — note that `d₃`'s arrangement records the original I-addresses directly, not pointers to `d₂`'s arrangement.

A reader at `d₃` asks SHOWORIGIN over the V-span containing all five positions. The block decomposition (C1a, ASN-0058) of `M(d₃) ↾ ⟦σ⟧` yields one mapping block `(v_start, [d₁.0.1.1], 5)`. By O2, this block contributes one origin: `origin([d₁.0.1.1]) = d₁`. The answer is `{d₁}`. The intermediate document `d₂` does not appear — illustrating O4 (parallel witnesses): `d₂` and `d₃` each independently hold an arrangement entry mapping their own V-positions to the same I-address `[d₁.0.1.1]`, and either document could be queried with identical result.

*Transition chain Σ₀ → Σ₀' → Σ₀'' → Σ₁ (allocation of native content in `d₃`).* The composite of "allocation of native content" decomposes into three atomic transitions: `Σ₀ → Σ₀'` is a K.α (ContentAllocation, ASN-0047) emitting the first content I-address `[d₃.0.1.1]` into `dom(C)`; `Σ₀' → Σ₀''` is a second K.α emitting `[d₃.0.1.2]`; `Σ₀'' → Σ₁` is a single K.μ⁺ (ArrangementExtension, ASN-0047) extending `M(d₃)` with the two new mappings `[1,1,6] ↦ [d₃.0.1.1]` and `[1,1,7] ↦ [d₃.0.1.2]`. Neither K.α step modifies any `M(d)` (their effect names only `C`); the K.μ⁺ step is the sole `M(d₃)`-modifying step in the chain. A SHOWORIGIN over the full seven-position V-span at Σ₁ returns two origins:

> `origins_V(Σ₁, d₃, σ_{1..7}) = { origin([d₁.0.1.1]), origin([d₃.0.1.1]) } = { d₁, d₃ }`.

Two mapping blocks, two origins. The block for the first five positions traces to `d₁`; the block for the last two traces to `d₃`. The multi-origin case is not a degenerate case — it is the expected case for any document of mixed authorship.

From Σ₁ we now evaluate two independent probes — a K.μ~ reordering (O14) and a K.μ⁻ contraction (O13) — each applied directly to Σ₁.

*Probe Σ₁ → Σ₁' (arrangement reordering in `d₃`, exhibiting K.μ~).* A K.μ~ transition (ASN-0047) realises the bijection equation via `π : dom(M(d₃)) → dom(M(d₃))` that swaps `[1,1,3]` and `[1,1,7]` and fixes every other V-position. The post-state arrangement is `M'(d₃)([1,1,3]) = [d₃.0.1.2]` (formerly `[d₁.0.1.3]`) and `M'(d₃)([1,1,7]) = [d₁.0.1.3]` (formerly `[d₃.0.1.2]`), with all other entries unchanged. This swap is an admissible K.μ~: its precondition holds (`M(d₃)|_{dom_C}` takes seven distinct values, above two), both swapped positions lie in the content subspace and carry distinct I-values so the net effect is genuine (`M'(d₃) ≠ M(d₃)`), and the admissibility clauses together with the invariant package they preserve (S8a, S8-depth, D-CTG★, D-MIN★, S3★, S3★-aux) are guaranteed by K.μ~'s definition in ASN-0047 — we do not re-derive foundation invariant-preservation here. What O14 turns on is the effect on origins, which we now exhibit.

Query SHOWORIGIN over the singleton V-span `σ_{3} = ([1,1,3], [0,0,1])` (T12-well-formed by inspection):

> At Σ₁: `origins_V(Σ₁, d₃, σ_{3}) = { origin([d₁.0.1.3]) } = { d₁ }`.
>
> At Σ₁': `origins_V(Σ₁', d₃, σ_{3}) = { origin([d₃.0.1.2]) } = { d₃ }`.

The inclusion `origins_V(Σ₁, d₃, σ_{3}) ⊆ origins_V(Σ₁', d₃, σ_{3})` *fails*: `{d₁} ⊄ {d₃}`, and neither set is a subset of the other. This is the canonical witness of O14 (K.μ~ non-preservation): the *mapping reassignment* failure mode that disqualifies K.μ~ from a monotonic-growth claim parallel to O11 / O11' / O11★★. Even though `|dom(M(d₃))|` is unchanged and every I-address remains allocated (by P0), the function values at individual V-positions are reassigned by the bijection, and origins can shift in and out of any sub-region of the arrangement — the failure mechanism recorded in O14.

*Probe Σ₁ → Σ₂ (arrangement contraction in `d₃`, exhibiting K.μ⁻).* Independently from Σ₁, `d₃` contracts its arrangement via K.μ⁻ to retain only V-positions `[1,1,1]` through `[1,1,5]` (the transcluded `Hello`). The native suffix is removed from the arrangement; `dom(M(d₃))` shrinks. By P0 (ContentPermanence, ASN-0047), `[d₃.0.1.1]` and `[d₃.0.1.2]` remain in `dom(C)`, but they are no longer in `ran(M(d₃))`.

At Σ₁, `origins_V(Σ₁, d₃, σ_{1..7}) = {d₁, d₃}` is well-formed — the seven depth-`m` positions all lie in `dom(M(d₃))`, satisfying precondition (vi). After the K.μ⁻ contraction, positions `[1,1,6]` and `[1,1,7]` no longer lie in `dom(M(d₃))`; precondition (vi) fails for σ_{1..7} at Σ₂. This is the canonical witness of O13 (K.μ⁻ admissibility loss): SHOWORIGIN_V is no longer admissible at this input on the post-state. A reader at Σ₂ who wants origins over `d₃`'s contracted arrangement must pose a smaller, still-admissible query — for instance, σ_{1..5}, which remains well-formed and yields `{d₁}`. The gap between O6 (I-span monotonicity, which remains well-formed because `dom(C)` only grows by P0) and the V-span case is therefore not a *non-monotonicity* of the V-span lift on a fixed input; it is *loss of admissibility*, as O13 records: arrangement contractions can render previously well-formed V-span queries inposable, since the operation requires the queried V-positions to be present in the arrangement. SHOWORIGIN_V's domain of admissibility shrinks as `dom(M(d))` shrinks.

*One wp evaluation (V-span discovery probe).* The weakest-precondition formula `wp(SHOWORIGIN_V(d, σ), d_q ∈ result) = (E v : v ∈ ⟦σ⟧ ∩ dom(M(d)) : origin(M(d)(v)) = d_q)` is the operational instrument by which a reader treats SHOWORIGIN as a discovery probe. We evaluate it at Σ₁ with d = `d₃` and the V-span σ_{1..7}:

- *Satisfying configuration for d_q = d₁.* V-position `[1,1,1]` lies in `⟦σ_{1..7}⟧ ∩ dom(M(d₃))` and satisfies `origin(M(d₃)([1,1,1])) = origin([d₁.0.1.1]) = d₁` — the existential witness exists. Hence the wp evaluates to *true*, and indeed `d₁ ∈ origins_V(Σ₁, d₃, σ_{1..7}) = {d₁, d₃}`.
- *Satisfying configuration for d_q = d₃.* V-position `[1,1,6]` lies in `⟦σ_{1..7}⟧ ∩ dom(M(d₃))` and satisfies `origin(M(d₃)([1,1,6])) = origin([d₃.0.1.1]) = d₃` — the existential witness exists. Hence the wp evaluates to *true*, and indeed `d₃ ∈ origins_V(Σ₁, d₃, σ_{1..7})`.
- *Falsifying configuration for d_q = d₂.* Although `d₂` is an intermediate transcluding document in our chain (`d₃` transcludes `d₂`, which transcludes `d₁`), no V-position `v ∈ ⟦σ_{1..7}⟧ ∩ dom(M(d₃))` maps to an I-address with origin `d₂`. The seven V-positions of `d₃`'s arrangement in this range map either to `[d₁.0.1.k]` (transcluded content, origin `d₁`) or to `[d₃.0.1.k]` (native content, origin `d₃`). None map to any I-address of the form `[d₂.0.1.k]`, because `d₂` itself never allocated content for this passage — it only transcluded `d₁`'s allocation. The existential witness does not exist, so the wp evaluates to *false*, and indeed `d₂ ∉ origins_V(Σ₁, d₃, σ_{1..7})`.

The `d_q = d₂` falsifying evaluation is the operational confirmation of O4 (parallel witnesses): an intermediate transcluding document does *not* appear in the direct origin set of a downstream reader, even though that intermediate document independently transcluded the same content.

## Summary

The abstract specification of SHOWORIGIN reduces to three primitives:

(1) The pointwise projection `origin : dom(C) ∪ dom(L) → E_doc`, which is structural, total, and permanent.

(2) The lift to I-spans, `origins_I(Σ, σ) = origin(⟦σ⟧ ∩ dom(C))`, computable from the span and `dom(C)` alone.

(3) The lift to V-spans, `origins_V(Σ, d, σ) = origin(ran(M(d) ↾ ⟦σ⟧))`, computable from the span, the arrangement, and `dom(C) ∪ dom(L)` alone. Uniform across subspaces: content-subspace V-spans report origins of their resolved content; link-subspace V-spans report `{d}` (the home document) trivially via CL-OWN.

Every other property in this note derives from these three. The operation derives no new knowledge; it presents existing structural facts about the address space.

Any implementation of Xanadu that claims to support SHOWORIGIN must satisfy O0–O14 (with the O1 corollaries O1.1, O1.2, the well-formedness corollary O11.1, and the multi-step companions O5★, O6★, O11★★). The operation may be realised through different mechanisms — direct tumbler-prefix decomposition, spanfilade lookup, granfilade traversal, or per-block `homedoc` records [Q12, Q13] — and these mechanisms may have different operational characteristics. But the abstract guarantees they deliver must coincide: every byte names its home, and every span reveals its sources.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| O0 | Origin extended to a total, single-valued, document-level projection on `dom(C) ∪ dom(L)`: `origin : dom(C) ∪ dom(L) → E_doc` | introduced |
| `origins_I(Σ, σ)` | `origins_I(Σ, σ) = { origin(a) : a ∈ ⟦σ⟧ ∩ dom(Σ.C) }` — I-span lift of origin | introduced |
| `origins_V(Σ, d, σ)` | `origins_V(Σ, d, σ) = { origin(M(d)(v)) : v ∈ ⟦σ⟧ ∩ dom(M(d)) }` — V-span lift via arrangement | introduced |
| O1 | Origin partitions allocated content: `~_o` is an equivalence on `⟦σ⟧ ∩ dom(C)` whose quotient is in bijection with `origins_I(Σ, σ)`; each class corresponds to one document's allocations | introduced |
| O1.1 | Single-origin sufficiency: confinement to one document's content yields `|origins_I| ≤ 1` (corollary of O1) | introduced |
| O1.2 | Multi-origin diagnostic: `|origins_I| > 1` ⇒ σ crosses ≥ 2 document allocation boundaries (corollary of O1) | introduced |
| O2 | Block uniformity: every I-address within a single mapping block shares one origin | introduced |
| O3 | Structural derivation: `origin(a)` and both lifts consult only the address (and, for V-span, the arrangement restricted to the span) | introduced |
| O4 | Parallel witnesses to a single origin: each intermediate document `d_i` (`2 ≤ i ≤ n`) independently records the same I-address `a`, and any can be queried with identical result `origin(a)` | introduced |
| O5 | Origin permanence: `origin'(a) = origin(a)` under every reachable transition `Σ → Σ'`, for any `a ∈ dom(Σ.C) ∪ dom(Σ.L)` | introduced |
| O5★ | Multi-step origin permanence: `origin'(a) = origin(a)` and `a ∈ dom(Σ'.C) ∪ dom(Σ'.L)` for every reachable `Σ →* Σ'` | introduced |
| O6 | Monotonic growth under state: `origins_I` is non-decreasing as content is added | introduced |
| O6★ | Multi-step monotonic growth: `origins_I(Σ, σ) ⊆ origins_I(Σ', σ)` for every reachable `Σ →* Σ'` | introduced |
| O7 | V-span stability under fixed arrangement: `origins_V` is unchanged when the arrangement restricted to the span is unchanged | introduced |
| O8 | I-span containment monotonicity: `⟦σ₁⟧ ⊆ ⟦σ₂⟧` ⇒ `origins_I(Σ, σ₁) ⊆ origins_I(Σ, σ₂)` | introduced |
| O9 | Origin tracks creation, not content: two addresses allocated under distinct documents have distinct origins, regardless of content values | introduced |
| O10 | Read-only frame; idempotence: SHOWORIGIN preserves the state; consecutive applications at the same state yield identical results | introduced |
| WF_V | V-span well-formedness predicate `WF_V(Σ, d, σ)`: the conjunction of conjuncts (i)–(vi) under which the V-span origin set is well-defined | introduced |
| SDP | Subspace-depth preservation under arrangement extension: for K.μ⁺ / K.μ⁺_L on `d` and a subspace `S` non-empty at Σ, `V_S(d)|_Σ ⊆ V_S(d)|_{Σ'}` and the S8-depth common depth is unchanged (`m' = m`) | introduced |
| O11 | V-span preservation under K.μ⁺: for `WF_V(Σ, d, σ)`, content-subspace arrangement extensions exactly preserve `origins_V` — equality, not merely inclusion | introduced |
| O11' | V-span preservation under K.μ⁺_L: for `WF_V(Σ, d, σ)`, link-subspace arrangement extensions exactly preserve `origins_V` | introduced |
| O11.1 | Well-formedness preservation under arrangement extension: `WF_V(Σ, d, σ)` + K.μ⁺ / K.μ⁺_L on `d` ⇒ `WF_V(Σ', d, σ)` | introduced |
| O11★★ | Multi-step V-span preservation under mixed K.μ⁺/K.μ⁺_L chain: for `WF_V(Σ, d, σ)`, any reachable chain whose `M(d)`-modifying steps are each K.μ⁺ or K.μ⁺_L on `d` preserves `origins_V` exactly | introduced |
| O12 | V-span containment monotonicity: `⟦σ₁⟧ ⊆ ⟦σ₂⟧` ⇒ `origins_V(Σ, d, σ₁) ⊆ origins_V(Σ, d, σ₂)` | introduced |
| O13 | K.μ⁻ admissibility loss (negative claim): there exist Σ, σ with `WF_V(Σ, d, σ)`, and a K.μ⁻ transition `Σ → Σ'` on `d` such that `WF_V(Σ', d, σ)` fails at conjunct (vi); no K.μ⁻ analogue of O11/O11'/O11★★ holds because preservation is not even formulable once admissibility is lost | introduced |
| O14 | K.μ~ non-preservation (negative claim): there exist Σ, a K.μ~ transition `Σ → Σ'` on `d`, and σ well-formed at both Σ and Σ' such that `origins_V(Σ, d, σ)` and `origins_V(Σ', d, σ)` are incomparable under set inclusion; no monotonicity claim parallel to O11/O11'/O11★★ holds for K.μ~ | introduced |
| wp(SHOWORIGIN_I, \|result\| = 1) | `(⟦σ⟧ ∩ dom(C) ≠ ∅) ∧ (A a, b ∈ ⟦σ⟧ ∩ dom(C) : origin(a) = origin(b))` — characterisation of single-origin I-spans | introduced |
| wp(SHOWORIGIN_V, d_q ∈ result) | `(E v : v ∈ ⟦σ⟧ ∩ dom(M(d)) : origin(M(d)(v)) = d_q)` — characterisation of when a queried document appears in the V-span result | introduced |
| SHOWORIGIN (I-span) | Operation over a well-formed I-span (T12 conjuncts (i)–(iv)) returning `origins_I(Σ, σ)` with `Σ' = Σ` | introduced |
| SHOWORIGIN (V-span) | Operation over a well-formed V-span (`WF_V(Σ, d, σ)`, conjuncts (i)–(vi)) returning `origins_V(Σ, d, σ)` with `Σ' = Σ` | introduced |

## Open Questions

The I-span lift deliberately reports only content origins, silently dropping link addresses present in the range (settled in the cross-subspace edge case); must a *unified* operation also be provided that reports the origins of both content and link addresses present in an I-stream range, and what must it guarantee where the two subspaces meet?

When a span's content has been transcluded through several intermediate documents, must any abstract operation be provided that surfaces the intermediate chain, or is the direct origin answer sufficient?

Must SHOWORIGIN distinguish content that was natively allocated in a queried document from content transcluded into it, or is this distinction the responsibility of a separate operation?

Does the system require a complementary operation reporting historical containment (from `Σ.R`) distinct from current arrangement origins, and what invariants must couple the two?
