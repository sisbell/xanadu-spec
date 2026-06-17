> **ASN-0093 · Allocation Substrate** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](ASN-0040-tumbler-baptism.md), [ASN-0043 · Link Model](ASN-0043-link-model.md)  
> [Condensed statements →](ASN-0093-allocation-substrate.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0093: Allocation Substrate

A Xanadu-style substrate maintains three address-keyed stores: the content store, the link store, and the document-arrangement function. Each store is grown by an allocation primitive that extends the store's domain at a fresh key with structural invariants on the new entry. ASN-0043 introduced the link store and its structural invariants (L0/L1/L1a/L1b/L1c/L3/L12); ASN-0036 introduced the content store and arrangement function. The substrate state is `Σ = (C, L, M)`, where `dom(M)` is the set of allocated documents. The substrate adds four content-side invariants that the inherited models do not carry — C1b (content element-field depth), the C-clause of L0 (content subspace partition), C1c (content allocator conformance), and C2 (content scoped allocation) — proved within this note.


## Scope

**Provided.** Three primitive operations (`K.σ`, `K.α`, `K.λ`) and the structural invariants, sub-allocator chains, chain disciplines, and transition-indexed lemmas they preserve — enumerated, with sources, in the *Properties Introduced* table.

**Substrate axioms:** SubspaceConventionAxiom pinning `s_C = 1 ∧ s_L = 2`; SequentialTransitionAxiom committing transitions to atomic and sequential.

**Deferred to higher-layer ASNs:**

- **Arrangement mutation.** `K.μ⁺`, `K.μ⁻`, `K.μ~`, `K.μ⁺_L` — operations that modify `M(d)` for an existing `d ∈ dom(M)`. The substrate fixes `M(d)` at `∅` on registration (M2, EmptyArrangement, below); these arrangement-extension primitives are deferred to a higher-layer ASN.
- **Entity allocation.** The substrate's `K.σ` is the document-registration primitive without the entity-hierarchy machinery; that machinery, layered on `K.σ`, is deferred to a higher-layer ASN.
- **Provenance recording.** A provenance-emission primitive and the provenance relation `R`. The substrate has no `R` component.
- **Coupling constraints.** Higher-layer coupling invariants binding K.α to K.μ⁺ etc. are out of scope; the substrate's `K.α` and `K.λ` stand independently.
- **Link withdrawal.** Nelson's tombstone-style withdrawal (LM 4/9) is deferred to a higher-layer retraction mechanism.


## State model

The substrate-level state is

> **Σ = (C, L, M)**

where

- `C : T ⇀ Val` is the content store (per ASN-0036): a partial function from element-level tumblers to content values. `Val` is the content value type defined in ASN-0036.
- `L : T ⇀ Link` is the link store (per ASN-0043): a partial function from element-level tumblers to link values, each a sequence of `N ≥ 3` endsets `(e₁, e₂, …, eₙ) ∈ Endset^N`. `Link` and `Endset` are defined in ASN-0043; the StandardTriple convention (slot 1 = from, slot 2 = to, slot 3 = type, written `(F, G, Θ)` for the arity-3 default; ASN-0043) is preserved.
- `M : T ⇀ (T ⇀ T)` is the arrangement function (per ASN-0036): a partial function whose domain `dom(M)` is the set of allocated document addresses, mapping each to its V-position-to-I-address arrangement

`dom(M)` is the set of tumblers committed by `K.σ` events (defined below). A document is *allocated* iff `d ∈ dom(M)`. The `origin(·)` function is the document-level field projection `origin(a) = N(a).0.U(a).0.D(a)`, well-defined on every T4-valid element-level tumbler (`zeros(a) = 3`) via T4b's `N`, `U`, `D` projections (ASN-0034). On content addresses this is ASN-0036's `origin` (S7); on link addresses, the identical projection ASN-0043 names `home` (L1c). This note adopts `origin` uniformly across both stores.

**Terminology.** "Document" in this substrate means "element of `dom(M)`" — a purely structural notion; M0 (below) carries the well-formedness conditions.

The initial state is `Σ₀ = (∅, ∅, ∅)` — no content, no links, no documents.

**Subspace identifiers.** As in ASN-0043, `s_C` and `s_L` denote the content-subspace and link-subspace first-element-field values. This substrate commits to two axioms governing them:

- **SubspaceConventionAxiom (FixedSubspaceIdentifiers).** `s_C = 1 ∧ s_L = 2`. The distinctness `s_C ≠ s_L` (abbreviated **SC-NEQ**) and the sibling relation `s_L = s_C + 1` are immediate consequences. Pinned by Nelson's design (LM 4/30–4/31) and Gregory's `xanadu.h:144–146` / `granf2.c:162` / `do2.c:94`.

- **SequentialTransitionAxiom (SequentialAtomicTransitions).** Transitions `Σ → Σ'` are atomic, uninterruptible, and totally ordered: each transition evaluates its precondition against `Σ` and commits its effect to `Σ'` in one indivisible step.


## Arrangement-function invariants

**M0 (DocumentTumblerWellFormed).**

  `(A d ∈ dom(M) :: T4-valid(d) ∧ zeros(d) = 2)`

Every allocated document address is a T4-valid tumbler (per T4, HierarchicalParsing, ASN-0034) with exactly two zero components (i.e., a document-level address per S7d of ASN-0036). Discharged from `K.σ`'s precondition (below) and inductively across transitions.

**M1 (ArrangementMonotonicity).**

  `(A Σ → Σ' :: dom(M) ⊆ dom(M'))`

`dom(M)` is non-decreasing across all transitions. The substrate admits no transition that removes a document from `dom(M)`. Discharged from the frame conditions of every transition kind: `K.σ` extends `dom(M)` by one element; `K.α` and `K.λ` hold `M` in frame.

**M2 (EmptyArrangement).**

  `(A d ∈ dom(M) :: M(d) = ∅)`

Every allocated document carries the empty arrangement. The substrate fixes `M(d) = ∅` at registration and admits no arrangement-mutation transition, so no document's arrangement value is ever populated. Base case `Σ₀.M = ∅`; discharged at the new key by `K.σ`'s effect clause `M'(d) = ∅`; preserved by `K.α`/`K.λ`, which hold `M` in frame.


## Content store invariants

**C0 (ContentImmutability).**

  `(A Σ → Σ' :: dom(C) ⊆ dom(C') ∧ (A a : a ∈ dom(C) : C'(a) = C(a)))`

Append-only with immutable values: `dom(C)` is non-decreasing, and no transition alters the value bound to an existing key.

**C1 (ContentElementLevel).**

  `(A a ∈ dom(C) :: zeros(a) = 3)`

Every content address is an element-level tumbler. Discharged from `K.α`'s precondition.

**C1b (ContentElementFieldDepth).**

  `(A a ∈ dom(C) :: #E(a) ≥ 2)`

Every content address has at least two element-field components. Discharged from `K.α`'s precondition.

**C1c (ContentAllocatorConformance).** Every content address `a ∈ dom(C)` has a T10a-conforming step sequence from its home document to `a`: a finite sequence `(t₀, t₁, …, tₙ)` with `n ≥ 1`, `t₀ = origin(a)`, and `tₙ = a`, where each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfies T10a's per-step admissibility constraints; additionally, `k₁ = 2` (the first step is a depth-2 increment off the document seed) and `(A i : 1 ≤ i ≤ n : #tᵢ > #origin(a))` (every intermediate length strictly exceeds the seed's).

**C2 (ContentScopedAllocation).**

  `(A a ∈ dom(C) :: origin(a) ∈ dom(M))`

Every content address has its home document allocated. Discharged from `K.α`'s precondition and M1.

**C-fin (ContentStoreFiniteness).**

  `|dom(C)| < ∞`

The content store is finite at every reachable state. Discharged inductively from `Σ₀.C = ∅` and `K.α`'s singleton extension.


## Link store invariants

All invariants below are stated against the reachable-state quantifier — they hold at every `Σ` reachable from `Σ₀` via the transitions defined later in this note.

**L0 (SubspacePartition).**

  `(A a ∈ dom(L) :: E(a)₁ = s_L)`
  `(A a ∈ dom(C) :: E(a)₁ = s_C)`

Every link address has subspace identifier `s_L`; every content address has subspace identifier `s_C`. The L-clause is inherited from ASN-0043; the C-clause is a derived substrate invariant, discharged at the new content key by the inductive-step matrix (L0 / K.α row).

**L1 (LinkElementLevel).**

  `(A a ∈ dom(L) :: zeros(a) = 3)`

Every link address is an element-level tumbler.

**L1a (LinkScopedAllocation).**

  `(A a ∈ dom(L) :: origin(a) ∈ dom(M))`

Every link address has its home document allocated.

**L1b (LinkElementFieldDepth).**

  `(A a ∈ dom(L) :: #E(a) ≥ 2)`

Every link address has at least two element-field components.

**L1c (LinkAllocatorConformance).** Every link address `ℓ ∈ dom(L)` has a *T10a-conforming step sequence* from its home document to `ℓ`: a finite sequence `(t₀, t₁, …, tₙ)` with `n ≥ 1`, `t₀ = origin(ℓ)`, and `tₙ = ℓ`, where each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfies T10a's per-step admissibility constraints; additionally, `k₁ = 2` (the first step is a depth-2 increment off the document seed) and `(A i : 1 ≤ i ≤ n : #tᵢ > #origin(ℓ))` (every intermediate length strictly exceeds the seed's).

**L3 (NEndsetStructure).**

  `(A a ∈ dom(L) :: |L(a)| ≥ 3 ∧ (A i : 1 ≤ i ≤ |L(a)| : L(a).eᵢ ∈ Endset) ∧ L(a).e₃ ≠ ∅)`

Every link is a sequence of at least three endsets, with the type endset (slot 3) non-empty. The StandardTriple default is retained for worked examples and notational convenience but not enforced structurally — the substrate admits arbitrary arity `N ≥ 3`.

**L12 (LinkImmutability).**

  `(A Σ → Σ' : (A a : a ∈ dom(L) : a ∈ dom(L') ∧ L'(a) = L(a)))`

Once allocated, a link's address persists in `dom(L)` and its value is permanently fixed across all transitions.

**SD (StoreDisjointness).**

  `dom(C) ∩ dom(L) = ∅`

Derived from L0 + SC-NEQ + StoreT4Validity + T7 (SubspaceDisjointness, ASN-0034). T7's preconditions are discharged on each side: `zeros(·) = 3` from C1 (content) and L1 (links), and T4-validity from StoreT4Validity (below). With both premises met, every content address has `E(·)₁ = s_C` and every link address has `E(·)₁ = s_L` (L0), and `s_C ≠ s_L` (SC-NEQ), so T7 gives, for every `a ∈ dom(C)` and every `ℓ ∈ dom(L)`, `a ≠ ℓ` — which is exactly `dom(C) ∩ dom(L) = ∅`.

**L-fin (LinkStoreFiniteness).**

  `|dom(L)| < ∞`

The link store is finite at every reachable state. Discharged inductively from `Σ₀.L = ∅` and `K.λ`'s singleton extension.


## Address sub-allocators under documents

The content and link subspaces are organised as sibling element-field sub-allocators rooted at each document. For each `d ∈ dom(M)`, two element-field anchors sit immediately under `d`:

- `b_C(d) := [d.0.s_C]` — the **content sub-allocator anchor** (one-component element field with `E₁ = s_C`, `zeros = 3`, `#E = 1`)
- `b_L(d) := [d.0.s_L]` — the **link sub-allocator anchor** (one-component element field with `E₁ = s_L`, `zeros = 3`, `#E = 1`)

These anchors are *structurally producible* by T10a `inc` steps from `d`: `b_C(d) = inc(d, 2)` appends `[0, 1]` (TA5(d), `k = 2`), yielding `[d.0.1] = [d.0.s_C]` once `s_C = 1` is supplied by SubspaceConventionAxiom; and `b_L(d) = inc(b_C(d), 0)` increments the `sig` component of `[d.0.s_C]` to `s_C + 1` (TA5(c)), yielding `[d.0.(s_C+1)] = [d.0.s_L]` once `s_L = s_C + 1` is supplied by SubspaceConventionAxiom. The anchors themselves are *not* in `dom(C) ∪ dom(L)` — content and link addresses have `#E ≥ 2` (C1b; L1b above), and the anchors have `#E = 1`.

**Active sub-allocator chains.** Define: a sub-allocator chain `A_C(d)` (resp. `A_L(d)`) is *active at state* `Σ` iff `d ∈ dom(M)` at `Σ`.

**Sub-allocator chains are ASN-0040 sibling streams (ChainDiscipline).** Each sub-allocator chain is an instance of ASN-0040's `SiblingStream`. Writing `S(p, k)` for the stream `c₁ = inc(p, k)`, `cₙ₊₁ = inc(cₙ, 0)` (SiblingStream, ASN-0040):

  `A_C(d) = S(b_C(d), 1)`  and  `A_L(d) = S(b_L(d), 1)`,

since each chain's first emission is `inc(anchor, 1)` and successive elements advance by `inc(·, 0)`, coinciding by construction with the SiblingStream recurrence at `p = b_·(d)`, `k = 1`. The depth parameter is `1` in both cases, so the streams append no interior zero; each chain is thereby closed under `inc(·, 0)`, exactly the SiblingStream recurrence. We refer to this derived identity as **ChainDiscipline**.

Both anchors satisfy ASN-0040's `B6` (ValidDepth) at depth `1`: `b_C(d)` and `b_L(d)` are T4-valid with `zeros = 3` (one separator inserted at position `#d + 1` under M0's T4-valid, `zeros = 2` document `d`), depth `1 ∈ {1, 2}`, and `zeros(b_·(d)) + (1 − 1) = 3 ≤ 3`. Each chain's parent `(b_·(d), 1)` is therefore B6-valid.

**Lemma (FirstEmission).** The first emission of each chain has a determinate structural form:
  - *Content chain first-emit:* `t_1^C(d) = inc(b_C(d), 1) = [d.0.s_C.1]` — `E(·)₁ = s_C`, `origin(·) = d`, `#E(·) = 2`, `zeros(·) = 3`, and T4-valid.
  - *Link chain first-emit:* `t_1^L(d) = inc(b_L(d), 1) = [d.0.s_L.1]` — structurally analogous, with `s_L` in place of `s_C`.

*Proof.* By ChainDiscipline, `A_C(d) = S(b_C(d), 1)` with first element `c₁ = inc(b_C(d), 1)`. ASN-0040's SiblingStream postcondition `cₙ = [p₁, …, p_{#p}, 0…0, n]` (with `d − 1 = 0` interior zeros at depth `1`) gives `c₁ = [b_C(d)₁, …, b_C(d)_{#b_C(d)}, 1]`; since `b_C(d) = [d.0.s_C]`, this is `[d.0.s_C.1]`, whence `E(·)₁ = s_C`, `#E(·) = 2`, `origin(·) = d`, and `zeros(·) = 3`. The link case runs the same SiblingStream argument at `p = b_L(d) = [d.0.s_L]` (anchor construction above): `c₁ = inc(b_L(d), 1) = [d.0.s_L.1]` gives `E(·)₁ = s_L`, `#E(·) = 2`, `origin(·) = d`, `zeros(·) = 3`.

*Anchor-construction admissibility.* The increment steps that build the anchors and first emissions from a T4-valid, `zeros = 2` document `d` are each TA5a-admissible, T4-validity propagating along the chain:

- `b_C(d) = inc(d, 2)`: TA5a's `k = 2` case, side condition `zeros(d) ≤ 2` discharged by M0's `zeros(d) = 2`. Hence `b_C(d)` is T4-valid with `zeros = 3`.
- `b_L(d) = inc(b_C(d), 0)`: TA5a's unconditionally-preserving `k = 0` case, so `b_L(d)` is T4-valid given `b_C(d)` T4-valid.
- `inc(b_·(d), 1)` (the first emission): TA5a's `k = 1` case, side condition `zeros(b_·(d)) ≤ 3` discharged by the anchor's T4-validity (T4 forces `zeros ≤ 3`).

This establishes the T4-validity of `[d.0.s_C.1]` (resp. `[d.0.s_L.1]`) and the per-step admissibility of the first `inc(·, 1)` step. ∎

**Per-chain disciplines (ASN-0040 citations).** Each discipline is the cited ASN-0040 result applied to the sibling stream `A_C(d) = S(b_C(d), 1)` (resp. `A_L(d) = S(b_L(d), 1)`), whose parent `(b_·(d), 1)` is `B6`-valid (verified above).

- **ChainElementT4Validity.** Every element of `A_C(d)` (resp. `A_L(d)`) is T4-valid. *Source: ASN-0040 B6(a) (ValidDepth sufficiency)* — for `B6`-valid `(p, d)`, every `cₙ ∈ S(p, d)` satisfies T4.

- **ChainEnumerationInjectivity.** The enumeration of `A_C(d)` (resp. `A_L(d)`) is strictly increasing under T1, `m < n ⟹ t_m < t_n`; hence `n ↦ t_n` is injective and (by T1 trichotomy) order-preserving in both directions, `m < n ⟺ t_m < t_n`. *Source: ASN-0040 S0 (StreamOrdering)* — `(A i, j : 1 ≤ i < j : cᵢ < cⱼ)`.

- **DisjointSubAllocatorChains.** `A_C(d)` and `A_L(d)` are disjoint as address sets, and addresses produced by `A_C(d)` (resp. `A_L(d)`) carry `E(·)₁ = s_C` (resp. `s_L`). *Source: ASN-0040 B7 (NamespaceDisjointness)* — `S(b_C(d), 1) ∩ S(b_L(d), 1) = ∅`, since `(b_C(d), 1) ≠ (b_L(d), 1)` (the anchors disagree at position `#d + 2`, `s_C` vs `s_L`, by SC-NEQ) and both parents are `B6`-valid. The subspace-identifier reading follows from ChainPrefixExtension: every element of `A_C(d)` agrees with `b_C(d)` at position `#d + 2`, where the value is `s_C` (resp. `s_L` for `A_L(d)`).

- **ChainPrefixExtension.** Every element of an active sub-allocator chain extends its anchor under the prefix relation:

    `(A d ∈ dom(M), t ∈ A_C(d) :: b_C(d) ≼ t)`
    `(A d ∈ dom(M), t ∈ A_L(d) :: b_L(d) ≼ t)`

  *Source: ASN-0040 S1 (StreamPrefix)* — `(A n : n ≥ 1 : p ≼ cₙ)`, applied at `p = b_C(d)` (resp. `b_L(d)`).

**Lemma (ChainMembershipForOrigin).** At every reachable state `Σ`, every entry of `dom(C)` (resp. `dom(L)`) inhabits the content (resp. link) sub-allocator chain of its origin, and forms a *contiguous initial segment* of that chain. Letting `A_C(d) = (t_1, t_2, t_3, …)` denote the enumeration of `d`'s content sub-allocator chain (with `t_1` the first emission and `t_{n + 1} = inc(t_n, 0)`), and `A_L(d) = (s_1, s_2, s_3, …)` the analogous link chain:

- `(A d ∈ dom(M) :: (E m_d ≥ 0 :: dom(C) ∩ {a' ∈ T : origin(a') = d} = {t_1, …, t_{m_d}}))` (content contiguous prefix; `{t_1, …, t_0} = ∅` by convention)
- `(A d ∈ dom(M) :: (E n_d ≥ 0 :: dom(L) ∩ {ℓ' ∈ T : origin(ℓ') = d} = {s_1, …, s_{n_d}}))` (link contiguous prefix)

The weaker subset inclusion `dom(C) ∩ {a' : origin(a') = d} ⊆ A_C(d)` (and its link analogue) is the immediate corollary of the contiguous-prefix form.

*Proof.* Induction on transition sequences from `Σ₀`, taken in the atomic total order of SequentialTransitionAxiom (SequentialAtomicTransitions, above).

*Base.* At `Σ₀`, both `dom(C)` and `dom(L)` are empty, so both inclusions hold vacuously for every `d`.

*Step.* Assume both inclusions hold at `Σ`. The substrate admits three transition kinds:

- *K.σ(d_new):* `C` and `L` are in frame, so for every `d` already in `dom(M)` the intersection set is unchanged and the contiguous-prefix postcondition transfers at the same `m_d` (resp. `n_d`). For the freshly registered `d_new`, the intersection sets are empty in `Σ'`. *Content clause derivation:* By the inductive hypothesis on C2 at `Σ`, every `a ∈ dom(C(Σ))` satisfies `origin(a) ∈ dom(M(Σ))`. By K.σ's precondition, `d_new ∉ dom(M(Σ))`. Therefore `origin(a) ≠ d_new` for every `a ∈ dom(C(Σ))`. Since `C` is in frame (`C(Σ') = C(Σ)`), `dom(C(Σ')) ∩ {a' : origin(a') = d_new} = dom(C(Σ)) ∩ {a' : origin(a') = d_new} = ∅ = {t_1, …, t_0}`, witnessing `m_{d_new} = 0` at `Σ'`. *Link clause derivation:* Symmetric, using the inductive hypothesis on L1a at `Σ` together with K.σ's precondition `d_new ∉ dom(M(Σ))` and frame on `L`, yielding `n_{d_new} = 0`.

- *K.α(d, a, v):* Only `dom(C)` grows, by one element `a` with `origin(a) = d`. For `d' ∈ dom(M)` with `d' ≠ d`, the intersection set `dom(C') ∩ {a' : origin(a') = d'} = dom(C) ∩ {a' : origin(a') = d'}` is unchanged (the new `a` has `origin(a) = d ≠ d'`), so the contiguous-prefix postcondition transfers at the same `m_{d'}`. For `d` itself, two sub-cases via the K.α emission rule:
  - *First emission* (`{a' ∈ dom(C) : origin(a') = d} = ∅`; equivalently `m_d = 0` at `Σ` by IH): by the FirstEmission lemma, `a = [d.0.s_C.1] = t_1` is the first emission of `A_C(d)`'s chain. The intersection set at `Σ'` is `{a} = {t_1}`, witnessing `m_d = 1` at `Σ'`.
  - *Subsequent emission* (`{a' ∈ dom(C) : origin(a') = d} ≠ ∅`; equivalently `m_d ≥ 1` at `Σ` by IH): by IH, the prior intersection is `{t_1, …, t_{m_d}}`. By ChainEnumerationInjectivity, `n ↦ t_n` is strictly increasing under T1, so `t_1 < t_2 < … < t_{m_d}` and the lex-order maximum of `{t_1, …, t_{m_d}}` is `t_{m_d}`. Hence `a_prev := max{a' ∈ dom(C) : origin(a') = d} = t_{m_d}`. By ChainDiscipline, `A_C(d)` is closed under `inc(·, 0)`, so `a = inc(t_{m_d}, 0) = t_{m_d + 1}`. The new intersection set at `Σ'` is `{t_1, …, t_{m_d}, t_{m_d + 1}} = {t_1, …, t_{m_d + 1}}`, witnessing the chain index `m_d + 1` at `Σ'`.

  The link contiguous-prefix postcondition is unchanged by frame on `dom(L)`.

- *K.λ(d, ℓ, (e₁, …, eₙ)):* Symmetric to K.α with content↔link, using the FirstEmission lemma for the first-emit branch (placing `ℓ = s_1`, witnessing `n_d = 1`) and ChainDiscipline for the subsequent-emit branch (placing `ℓ = s_{n_d + 1}` from `ℓ_prev = s_{n_d}` by ChainEnumerationInjectivity, witnessing `n_d + 1` at `Σ'`). The content contiguous-prefix postcondition is unchanged by frame on `dom(C)`. ∎

**Corollary (StoreT4Validity).** At every reachable state `Σ`, every entry of `dom(C) ∪ dom(L)` is a T4-valid tumbler:

  `(A a ∈ dom(C) :: T4-valid(a))`
  `(A ℓ ∈ dom(L) :: T4-valid(ℓ))`

*Proof.* For any `a ∈ dom(C)`, C1c gives a T10a-conforming step sequence from the document seed `origin(a)` to `a`. The seed is T4-valid: C2 gives `origin(a) ∈ dom(M)`, and M0 gives that every `dom(M)` member is T4-valid with `zeros = 2`, so `origin(a)` satisfies T10a.4's initialization premise. By T10a.4 (T4PreservationUnderDiscipline, ASN-0034) every output of a conforming allocator seeded at a T4-valid address satisfies T4, so the terminus `a` is T4-valid. The link case is symmetric, using L1c for the chain and L1a (in place of C2) to land the seed `origin(ℓ) ∈ dom(M)`, whence M0 gives the seed's T4-validity. ∎

**Lemma (FirstEmissionFreshness).** At every reachable state `Σ`, the first emission of an active sub-allocator chain — the address that K.α (resp. K.λ) commits when the corresponding first-emit predicate fires — is fresh against `dom(C) ∪ dom(L)`:

  - *Content first-emit:* Under the K.α first-emit predicate `{a' ∈ dom(C) : origin(a') = d} = ∅`, the first emission `a = [d.0.s_C.1]` of `A_C(d)` satisfies `a ∉ dom(C) ∪ dom(L)` at the K.α event that commits `a`.
  - *Link first-emit:* Under the K.λ first-emit predicate `{ℓ' ∈ dom(L) : origin(ℓ') = d} = ∅`, the first emission `ℓ = [d.0.s_L.1]` of `A_L(d)` satisfies `ℓ ∉ dom(L) ∪ dom(C)` at the K.λ event that commits `ℓ`.

*Proof.*

*Content case, against `dom(C)`.* Under the first-emit predicate at the pre-state `Σ`, every `a' ∈ dom(C)` has `origin(a') ≠ d`. (i) `a = [d.0.s_C.1]` is the first emission of `A_C(d)`, so by ChainPrefixExtension (base case), `b_C(d) ≼ a`. (ii) For every `a' ∈ dom(C)` with `origin(a') ≠ d`: ChainMembershipForOrigin at `Σ` places `a' ∈ A_C(origin(a'))` (well-defined since `origin(a') ∈ dom(M)` by C2), and ChainPrefixExtension gives `b_C(origin(a')) ≼ a'`. (iii) Cross-document disjointness applied to `(d, origin(a'))` gives `b_C(d) ⋠ b_C(origin(a')) ∧ b_C(origin(a')) ⋠ b_C(d)`. (iv) T10 (PartitionIndependence, ASN-0034) closes: `a ≠ a'`.

*Content case, against `dom(L)`.* StoreT4Validity at `Σ` gives T4-validity of every `ℓ ∈ dom(L)`; `a` is T4-valid by ChainElementT4Validity applied to `A_C(d)` (whose first emission `[d.0.s_C.1]` is T4-valid by the FirstEmission lemma). The subspace identifiers split by source: by L0 at `Σ`, `E(ℓ)₁ = s_L` for the pre-existing peer `ℓ ∈ dom(L)`; for the new key `a` we read `E(a)₁ = s_C` from the FirstEmission lemma's structural form `a = [d.0.s_C.1]`. By SC-NEQ, `s_C ≠ s_L`; `zeros(a) = zeros(ℓ) = 3` by FirstEmission's structural form and L1. T7 (SubspaceDisjointness, ASN-0034) closes: `a ≠ ℓ`.

*Link case.* By symmetry under content↔link (swap `A_C(d)↔A_L(d)`, `s_C↔s_L`, C2↔L1a), the link first emission is fresh against `dom(L)` by the cross-document/T10 argument and against `dom(C)` by the T7 argument. ∎

**Lemma (SubsequentEmissionFreshness).** At every reachable state `Σ`, the subsequent emission of an active sub-allocator chain — the address `a = inc(a_prev, 0)` that K.α commits when its subsequent-emit predicate fires, with `a_prev = max{a' ∈ dom(C) : origin(a') = d}` (resp. `ℓ = inc(ℓ_prev, 0)` for K.λ, with `ℓ_prev = max{ℓ' ∈ dom(L) : origin(ℓ') = d}`) — is fresh against `dom(C) ∪ dom(L)`. Freshness splits three ways:

  - *Within-document* (against `{a' ∈ dom(C) : origin(a') = d}`): `a = inc(a_prev, 0) ∈ A_C(d)` by ChainDiscipline's closure under `inc(·, 0)`, with `a_prev ∈ A_C(d)` by ChainMembershipForOrigin; ChainEnumerationInjectivity applied to `(a_prev, a)` gives strict advance past every prior same-chain sibling, so `a ∉ dom(C)` at `d`.
  - *Cross-document* (against `{a' ∈ dom(C) : origin(a') ≠ d}`): Cross-document disjointness applied to `(d, origin(a'))` plus T10 (PartitionIndependence, ASN-0034) gives `a ≠ a'`, exactly as in the FirstEmissionFreshness content-against-`dom(C)` case.
  - *Cross-subspace* (against `dom(L)`): `E(a)₁ = s_C` (read along `A_C(d)` via DisjointSubAllocatorChains — structural, since `a ∈ A_C(d)`) while `E(ℓ)₁ = s_L` for every pre-existing peer `ℓ ∈ dom(L)` (L0); SC-NEQ and T7 (SubspaceDisjointness, ASN-0034) close `a ≠ ℓ`, exactly as in the FirstEmissionFreshness content-against-`dom(L)` case.

The link subsequent emission is symmetric under the content↔link substitution. ∎



## Cross-document disjointness chain

**Lemma (Cross-document disjointness).** For any two distinct documents `d₁, d₂ ∈ dom(M)` with `d₁ ≠ d₂`, the anchors `p_i := b_·(d_i)` (for `· ∈ {L, C}`) are prefix-incomparable, `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`, so by T10 (PartitionIndependence, ASN-0034) every address extending one anchor differs from every address extending the other:

  `a ≠ b`  for every `a` with `p₁ ≼ a` and every `b` with `p₂ ≼ b`.

*Proof.* By M0, both `d₁, d₂` are T4-valid with `zeros = 2`, so (as established under *Sub-allocator chains are ASN-0040 sibling streams*) each anchor `p_i = b_·(d_i)` is T4-valid with `zeros = 3` and is a length-`+2` extension of `d_i` (positions `1..#d_i` reproduce `d_i`, position `#d_i + 1` is the separator `0`, position `#d_i + 2` is `s_·`). They are prefix-incomparable: when `d₁`, `d₂` are themselves prefix-incomparable, a document-level divergence position `k ≤ min(#d₁, #d₂)` lifts unchanged to the anchors; when one properly prefixes the other (WLOG `d₁ ≺ d₂`), the anchors diverge at the separator position `k = #d₁ + 1`, where `p₁[k] = 0` while `p₂[k] = d₂[k] ≠ 0` (by M0, `d₂` carries `d₁`'s two zeros at the shared positions and, having `zeros(d₂) = 2`, no further zero, so position `#d₁ + 1 ≤ #d₂` is nonzero). Either way `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁` (Prefix, ASN-0034), and T10 gives `a ≠ b` for any `a` extending `p₁`, `b` extending `p₂`. ∎


## Substrate primitive operations

The substrate admits three primitive transitions, one per state component. Each is atomic and sequential by SequentialTransitionAxiom (SequentialAtomicTransitions, above).

### K.σ (DocumentRegistration)

Extends `dom(M)` by registering a new document address with an empty arrangement.

*Precondition:*
- `d ∉ dom(M)` (fresh document address)
- `T4-valid(d) ∧ zeros(d) = 2` (document-level — discharges M0 at the new key)

*Effect:* `dom(M') = dom(M) ∪ {d}`, with `M'(d) = ∅` and `M'(d') = M(d')` for every `d' ∈ dom(M)`.

*Frame:* `C' = C; L' = L`

Chains `A_C(d)`, `A_L(d)` are active once `d ∈ dom(M)` (per Address sub-allocators).

### K.α (ContentAllocation)

Extends `dom(C)` with a fresh content address scoped to an allocated document.

*Binding precondition:*
- `d ∈ dom(M)` (home document exists)
- `a` is produced by `d`'s content sub-allocator `A_C(d)`:
  - *First emission* (predicate: `{a' ∈ dom(C) : origin(a') = d} = ∅`): `a = [d.0.s_C.1]`.
  - *Subsequent emission* (predicate: `{a' ∈ dom(C) : origin(a') = d} ≠ ∅`): `a = inc(a_prev, 0)` (TA5(c)) where `a_prev := max{a' ∈ dom(C) : origin(a') = d}`, the next sibling on `A_C(d)`'s `inc(·, 0)` chain. The `max` is well-defined because the set is finite (C-fin restricted by `origin(·) = d`).
- `v ∈ Val` (well-formed content value)

*Effect:* `C' = C ∪ {a ↦ v}`

*Frame:* `L' = L; M' = M`.

### K.λ (LinkAllocation)

Extends `dom(L)` with a fresh link address scoped to an allocated document.

Signature: `K.λ(d, ℓ, (e₁, …, eₙ))` where the link value is a finite sequence of `N` endsets.

*Binding precondition:*
- `d ∈ dom(M)` (home document exists)
- `ℓ` is produced by `d`'s link sub-allocator `A_L(d)`:
  - *First emission* (predicate: `{ℓ' ∈ dom(L) : origin(ℓ') = d} = ∅`): `ℓ = [d.0.s_L.1]`, the determinate first emission of `A_L(d)`.
  - *Subsequent emission* (predicate: `{ℓ' ∈ dom(L) : origin(ℓ') = d} ≠ ∅`): `ℓ = inc(ℓ_prev, 0)` (TA5(c)) where `ℓ_prev := max{ℓ' ∈ dom(L) : origin(ℓ') = d}`, the next sibling on `A_L(d)`'s `inc(·, 0)` chain. The `max` is well-defined because the set is finite (L-fin restricted by `origin(·) = d`).
- `N ≥ 3 ∧ (A i : 1 ≤ i ≤ N : eᵢ ∈ Endset) ∧ e₃ ≠ ∅` (well-formed link value with mandatory non-empty type endset at slot 3 — L3).

*Effect:* `L' = L ∪ {ℓ ↦ (e₁, …, eₙ)}`

*Frame:* `C' = C; M' = M`.


## Worked example

To make the substrate's operation concrete, we trace a small scenario step-by-step starting from `Σ₀ = (∅, ∅, ∅)`.

*Arity convention.* The K.λ invocations below use StandardTriple `N = 3` instances; L3 admits arbitrary `N ≥ 3`.

*Fix a document address.* Let `d = [1, 0, 2, 0, 5]` — `#d = 5`, with zeros at positions 2 and 4 so `zeros(d) = 2`, with positive first and last components (1 and 5) and no adjacent zeros, hence T4-valid. By T4b, its projections are `N(d) = [1]`, `U(d) = [2]`, `D(d) = [5]`. By SubspaceConventionAxiom, `s_C = 1` and `s_L = 2`.

*Step 1 — `K.σ(d)` (document registration).* Precondition: `d ∉ dom(M₀) = ∅` ✓; `T4-valid(d) ∧ zeros(d) = 2` ✓. Effect commits `dom(M₁) = {d}` with `M₁(d) = ∅`; `C₁ = ∅`, `L₁ = ∅`. Both `A_C(d)` and `A_L(d)` are active under `d` (since `d ∈ dom(M₁)`). Verifying invariants at `Σ₁ = (∅, ∅, {d ↦ ∅})`: M0 holds (the single key `d` satisfies `zeros = 2`); M1 holds (`∅ ⊆ {d}`); all C-/L-invariants and SD, L-fin, C-fin are vacuous or trivial on empty stores.

*Step 2 — `K.α(d, a, v)` (first content emission).* Pinning the address from `Σ₁`: the predicate `{a' ∈ dom(C₁) : origin(a') = d} = ∅` selects the first-emit case, so `a = [d.0.s_C.1] = [1, 0, 2, 0, 5, 0, 1, 1]`. Witness it via the C1c chain `(t₀, t₁, t₂)`:
- `t₀ = d = [1, 0, 2, 0, 5]`
- `t₁ = inc(d, 2)`: TA5(d) at `k = 2` gives the structural form, appending `[0, 1]` to yield `[1, 0, 2, 0, 5, 0, 1] = b_C(d)`. Admissibility: TA5a at `k = 2` requires `zeros(d) ≤ 2`; M0 gives `zeros(d) = 2 ≤ 2`, satisfied — hence `t₁` is T4-valid; TA5(d) gives `zeros(t₁) = 3`.
- `t₂ = inc(b_C(d), 1)`: TA5(d) at `k = 1` gives the structural form, appending `1` to yield `[1, 0, 2, 0, 5, 0, 1, 1] = a`. Admissibility: TA5a at `k = 1` requires `zeros(t₁) ≤ 3`, discharged by `t₁`'s T4-validity (T4 forces `zeros ≤ 3`), so `t₂` is T4-valid given `t₁` T4-valid; TA5(d) gives `zeros(t₂) = 3`, `#E(t₂) = 2`.

Verifying preconditions: `a ∉ dom(C₁) ∪ dom(L₁) = ∅` ✓; `zeros(a) = 3` ✓; `E(a) = [1, 1]` so `E(a)₁ = 1 = s_C` ✓; `#E(a) = 2 ≥ 2` ✓; `origin(a) = N(a).0.U(a).0.D(a) = [1].0.[2].0.[5] = d` ✓. Freshness of `a` against `dom(C₁) ∪ dom(L₁)` is supplied by FirstEmissionFreshness, here vacuously since the predecessor stores are empty.

Effect: `C₂ = {a ↦ v}`; `L₂ = ∅`; `M₂ = M₁`. Verifying invariants at `Σ₂`: C0 (extended at fresh `a`), C1 (`zeros(a) = 3`), C1b (`#E(a) = 2`), C1c (chain exhibited above), C2 (`origin(a) = d ∈ dom(M₂)`), C-fin (`|dom(C₂)| = 1 < ∞`) all hold at the new key.

*Step 3 — `K.λ(d, ℓ, F, G, Θ)` (first link emission).* Pinning from `Σ₂`: the predicate `{ℓ' ∈ dom(L₂) : origin(ℓ') = d} = ∅` selects the first-emit case, so `ℓ = [d.0.s_L.1] = [1, 0, 2, 0, 5, 0, 2, 1]`. Witness via the L1c chain `(t₀, t₁, t₂, t₃)`:
- `t₀ = d = [1, 0, 2, 0, 5]`
- `t₁ = inc(d, 2) = [1, 0, 2, 0, 5, 0, 1] = b_C(d)` (admissibility as in Step 2)
- `t₂ = inc(b_C(d), 0)`: TA5(c) at `k = 0` gives the structural form, incrementing `b_C(d)`'s rightmost nonzero component (position 7, from `1` to `2`) to yield `[1, 0, 2, 0, 5, 0, 2] = b_L(d)`. By SubspaceConventionAxiom, `s_L = 2 = s_C + 1`, matching position 7. Admissibility: TA5a at `k = 0` is unconditionally T4-preserving, so `t₂` is T4-valid given `b_C(d)` T4-valid (from Step 2).
- `t₃ = inc(b_L(d), 1)`: TA5(d) at `k = 1` gives the structural form, appending `1` to yield `[1, 0, 2, 0, 5, 0, 2, 1] = ℓ`. Admissibility: TA5a at `k = 1` requires `zeros(b_L(d)) ≤ 3`, discharged by `b_L(d)`'s T4-validity (T4 forces `zeros ≤ 3`), so `t₃` is T4-valid given `b_L(d)` T4-valid. `zeros(ℓ) = 3`, `#E(ℓ) = 2`.

Verifying preconditions: `ℓ ∉ dom(L₂) ∪ dom(C₂) = {a}`. Disagreement at position 7 (`a₇ = 1` vs `ℓ₇ = 2`) gives `ℓ ≠ a`, confirming the L0 + SC-NEQ + T7 derivation: the two addresses sit in disjoint subspaces. `zeros(ℓ) = 3` ✓; `E(ℓ) = [2, 1]` so `E(ℓ)₁ = 2 = s_L` ✓; `#E(ℓ) = 2 ≥ 2` ✓; `origin(ℓ) = d` ✓. Freshness supplied by FirstEmissionFreshness.

Effect: `L₃ = {ℓ ↦ (F, G, Θ)}`; `C₃ = C₂`; `M₃ = M₂`. Verifying invariants at `Σ₃`: L0/L1/L1a/L1b/L1c/L3/L12 all hold at the new key per the matrix; SD holds non-trivially: `dom(C₃) ∩ dom(L₃) = {a} ∩ {ℓ} = ∅` (verified by E(·)₁ disagreement); L-fin holds (`|dom(L₃)| = 1 < ∞`).

*Step 4 — `K.α(d, a', v')` (second content emission, subsequent-emit branch).* Pinning from `Σ₃`: `{a'' ∈ dom(C₃) : origin(a'') = d} = {a}` is non-empty, so the subsequent-emit branch fires with `a' = inc(max{a}, 0) = inc(a, 0)`. Since `sig(a) = 8` with value `1`, TA5(c) gives `a' = [1, 0, 2, 0, 5, 0, 1, 2]`. The C1c chain extends `a`'s chain by one step: `(t₀, t₁, t₂, a')` with `a' = inc(t₂, 0) = inc(a, 0)`. Admissibility of the new step: TA5a at `k = 0` is unconditionally T4-preserving (no side condition), so `a'` is T4-valid given `a` T4-valid (the latter from Step 2's chain exhibition); TA5(c) gives the structural form. Freshness against `dom(C₃) = {a}` discharged by ChainEnumerationInjectivity (within-chain injectivity) applied to `A_C(d)`'s chain (per ChainDiscipline); freshness against `dom(L₃) = {ℓ}` discharged by L0 + SC-NEQ + T7.

Verifying preconditions: `a' ∉ dom(C₃) ∪ dom(L₃) = {a, ℓ}` ✓ (since `a' > a` strictly by TA5(a), and `E(a')₁ = 1 ≠ 2 = E(ℓ)₁`); structural preconditions inherit from `a` via the inc rule (TA5(b) preserves `zeros`, `E(·)₁`, and `origin(·)`).

Effect: `C₄ = {a ↦ v, a' ↦ v'}`; `L₄ = L₃`; `M₄ = M₃`. All invariants continue to hold at `Σ₄`.

*Step 5 — `K.σ(d')` (second document registration).* Fix a second document address `d' = [1, 0, 2, 0, 5, 3]`. Verifying T4-validity: `#d' = 6`, zeros at positions 2 and 4 only (`zeros(d') = 2`), no adjacent zeros (positions (2,3) = (0,2) and (4,5) = (0,5)), first component `d'[1] = 1 ≠ 0`, last component `d'[6] = 3 ≠ 0`. By T4b, `N(d') = [1]`, `U(d') = [2]`, `D(d') = [5, 3]`. Precondition: `d' ∉ dom(M₄) = {d}` ✓ (distinct since `#d = 5 ≠ 6 = #d'`); `T4-valid(d') ∧ zeros(d') = 2` ✓. Effect: `dom(M₅) = {d, d'}`, with `M₅(d') = ∅` and `M₅(d) = M₄(d) = ∅`. `A_C(d')` and `A_L(d')` become active at `Σ₅` (since `d' ∈ dom(M₅)`), alongside the already-active `A_C(d)` and `A_L(d)`.

*Verifying the Cross-document disjointness lemma at Σ₅.* Apply with `d₁ = d`, `d₂ = d'`. Component-by-component, `d'[1..5] = [1, 0, 2, 0, 5] = d` with `#d < #d'`, so `d ≺ d'` (the properly-prefixing case). The anchors are `p₁ = b_L(d) = [1, 0, 2, 0, 5, 0, 2]` (length 7) and `p₂ = b_L(d') = [1, 0, 2, 0, 5, 3, 0, 2]` (length 8). At the separator index `k = #d + 1 = 6`:
- `p₁[6] = 0` (the zero separator inserted by the `b_L` construction at position `#d + 1`)
- `p₂[6] = d'[6] = 3 ≠ 0` (`d'` carries its two zeros at positions 2 and 4 by `zeros(d') = 2`, so position 6 must be nonzero per the T4 zero-count argument)
- `k = 6 ≤ min(#p₁, #p₂) = 7` ✓

Thus `p₁[6] = 0 ≠ 3 = p₂[6]`, witnessing `b_L(d) ⋠ b_L(d') ∧ b_L(d') ⋠ b_L(d)`. The same divergence holds at position 6 for the content anchors `b_C(d) = [1, 0, 2, 0, 5, 0, 1]` and `b_C(d') = [1, 0, 2, 0, 5, 3, 0, 1]`. By T10, every link allocated under `d` (extending `b_L(d)`) differs from every link allocated under `d'` (extending `b_L(d')`); same for content.

*Verifying invariants at Σ₅.* M0 holds: `d` and `d'` both satisfy `T4-valid ∧ zeros = 2`. M1 holds: `{d} ⊆ {d, d'}`. C0/C1/C1b/C1c/C2/C-fin hold by frame on `C` (unchanged from `Σ₄`); in particular, C2 carries the prior content keys `a, a'` whose `origin = d ∈ dom(M₅)`, preserved by M1's extension. L0/L1/L1a/L1b/L1c/L3/L12/L-fin hold by frame on `L` (unchanged from `Σ₄`); L1a holds for `ℓ` since `origin(ℓ) = d ∈ dom(M₅)`. SD: `dom(C₅) ∩ dom(L₅) = {a, a'} ∩ {ℓ} = ∅` (verified by `E(a)₁ = E(a')₁ = s_C ≠ s_L = E(ℓ)₁`). ChainMembershipForOrigin transfers: `dom(C₅) ∩ {a'' : origin(a'') = d} = {a, a'} ⊆ A_C(d)` (per Steps 2 and 4); `dom(C₅) ∩ {a'' : origin(a'') = d'} = ∅ ⊆ A_C(d')` (vacuous, first emission still pending); similarly for `L`.

*Step 6 — `K.α(d', a'', v'')` (first content emission under `d'`).* Pinning the address from `Σ₅`: `{a''' ∈ dom(C₅) : origin(a''') = d'} = ∅` (the content keys `a, a'` have `origin = d ≠ d'`), so the first-emit branch fires with `a'' = [d'.0.s_C.1] = [1, 0, 2, 0, 5, 3, 0, 1, 1]` (length 9). The C1c chain `(t₀, t₁, t₂) = (d', b_C(d'), a'')` and its per-step TA5a admissibility are identical to Step 2 under `d → d'` (`k = 2` discharged by `zeros(d') = 2 ≤ 2` from M0, `k = 1` by `b_C(d')`'s T4-validity), yielding `b_C(d') = [1, 0, 2, 0, 5, 3, 0, 1]` and `a'' = [1, 0, 2, 0, 5, 3, 0, 1, 1]` with `zeros(a'') = 3`, `#E(a'') = 2`; we do not re-exhibit it. The genuinely new material is the multi-component document field: `D(d') = [5, 3]` (against `D(d) = [5]`), so `a''` carries `d'` over positions `1..6` and diverges from the `d`-rooted content key `a = [1, 0, 2, 0, 5, 0, 1, 1]` already at position 6 (`a''₆ = 3 ≠ 0 = a₆`) — the origin extraction `N(a'').0.U(a'').0.D(a'') = [1].0.[2].0.[5, 3] = d'` thus recovers a two-component document field.

Verifying preconditions: freshness `a'' ∉ dom(C₅) ∪ dom(L₅) = {a, a', ℓ}` is supplied by FirstEmissionFreshness applied to `A_C(d')` (first-emit). Other preconditions: `zeros(a'') = 3` ✓; `E(a'') = [1, 1]`, `E(a'')₁ = s_C` ✓; `#E(a'') = 2` ✓; `origin(a'') = N(a'').0.U(a'').0.D(a'') = [1].0.[2].0.[5, 3] = d'` ✓.

Effect: `C₆ = C₅ ∪ {a'' ↦ v''} = {a ↦ v, a' ↦ v', a'' ↦ v''}`; `L₆ = L₅`; `M₆ = M₅`. Invariants at `Σ₆`: C0 (existing values unchanged), C1 (`zeros(a'') = 3`), C1b (`#E(a'') = 2`), C1c (chain exhibited above), C2 (`origin(a'') = d' ∈ dom(M₆)`), C-fin (`|C₆| = 3 < ∞`); ChainMembershipForOrigin extends: `{a''} ⊆ A_C(d')` by FirstEmission.

*Step 7 — `K.λ(d', ℓ'', F'', G'', Θ'')` (first link emission under `d'`).* Pinning from `Σ₆`: `{ℓ''' ∈ dom(L₆) : origin(ℓ''') = d'} = ∅` (`origin(ℓ) = d ≠ d'`), so the first-emit branch fires with `ℓ'' = [d'.0.s_L.1] = [1, 0, 2, 0, 5, 3, 0, 2, 1]` (length 9). The L1c chain `(t₀, t₁, t₂, t₃) = (d', b_C(d'), b_L(d'), ℓ'')` and its per-step TA5a admissibility are identical to Step 3 under `d → d'` (`k = 2` by `zeros(d') = 2 ≤ 2` from M0, `k = 0` unconditional, `k = 1` by `b_L(d')`'s T4-validity), yielding `b_L(d') = [1, 0, 2, 0, 5, 3, 0, 2]` and `ℓ'' = [1, 0, 2, 0, 5, 3, 0, 2, 1]` with `zeros(ℓ'') = 3`, `#E(ℓ'') = 2`; we do not re-exhibit it. As in Step 6, the new material is the multi-component document field `D(d') = [5, 3]`, so `ℓ''` carries `d'` over positions `1..6` and `origin(ℓ'') = N(ℓ'').0.U(ℓ'').0.D(ℓ'') = [1].0.[2].0.[5, 3] = d'` recovers the two-component document field. The new cross-document branch (prefix-incomparable freshness against the `d`-rooted link `ℓ`) is delivered in the precondition verification below.

Verifying preconditions: `ℓ'' ∉ dom(L₆) ∪ dom(C₆) = {ℓ, a, a', a''}`. *Cross-document freshness* against `{ℓ}` (origin = d ≠ d'): by Cross-document disjointness at Step 5, `b_L(d) ⋠ b_L(d') ∧ b_L(d') ⋠ b_L(d)`; `ℓ` extends `b_L(d)` (Step 3) while `ℓ''` extends `b_L(d')`, so by T10, `ℓ'' ≠ ℓ`. *Sub-space freshness* against `{a, a', a''}`: each content address has `E(·)₁ = s_C = 1 ≠ 2 = s_L = E(ℓ'')₁`, so `ℓ'' ≠ a, a', a''`. Other preconditions: `zeros(ℓ'') = 3` ✓; `E(ℓ'') = [2, 1]`, `E(ℓ'')₁ = s_L` ✓; `#E(ℓ'') = 2` ✓; `origin(ℓ'') = d'` ✓.

Effect: `L₇ = L₆ ∪ {ℓ'' ↦ (F'', G'', Θ'')}`; `C₇ = C₆`; `M₇ = M₆`. Invariants at `Σ₇`: L0 (`E(ℓ'')₁ = s_L`), L1 (`zeros(ℓ'') = 3`), L1a (`origin(ℓ'') = d' ∈ dom(M₇)`), L1b (`#E(ℓ'') = 2`), L1c (chain exhibited above), L3 (triple endset with non-empty `Θ''`), L12 (existing link `ℓ ↦ (F, G, Θ)` unchanged), SD (`dom(C₇) ∩ dom(L₇) = {a, a', a''} ∩ {ℓ, ℓ''} = ∅` by SC-NEQ), L-fin (`|L₇| = 2 < ∞`); ChainMembershipForOrigin extends: `{ℓ''} ⊆ A_L(d')` by FirstEmission, witnessing `n_{d'} = 1`.

*Step 8 — `K.λ(d, ℓ_new, F_new, G_new, Θ_new)` (second link emission under `d`, subsequent-emit branch).* Pinning the address from `Σ₇`: `{ℓ''' ∈ dom(L₇) : origin(ℓ''') = d} = {ℓ}` (note `origin(ℓ'') = d' ≠ d`), so the subsequent-emit branch fires with `ℓ_new = inc(max{ℓ}, 0) = inc(ℓ, 0)`. By ChainMembershipForOrigin's contiguous-prefix form at `Σ₇`, `dom(L₇) ∩ {ℓ''' : origin(ℓ''') = d} = {s₁}` with `ℓ = s₁`, so the lex-order max is `s₁` and `ℓ_new = s₂`. Since `sig(ℓ) = 8` with value `1`, TA5(c) gives `ℓ_new = [1, 0, 2, 0, 5, 0, 2, 2]`. The L1c chain extends `ℓ`'s chain by one step: `(t₀, t₁, t₂, t₃, t₄)` with `t₀ = d`, `t₁ = b_C(d)`, `t₂ = b_L(d)`, `t₃ = ℓ`, `t₄ = inc(ℓ, 0) = ℓ_new`. Admissibility of the new step: TA5a at `k = 0` is unconditionally T4-preserving (no side condition), so `ℓ_new` is T4-valid given `ℓ` T4-valid (the latter from Step 3's chain exhibition); TA5(c) gives the structural form.

Verifying preconditions: freshness `ℓ_new = [1, 0, 2, 0, 5, 0, 2, 2] ∉ dom(L₇) ∪ dom(C₇) = {ℓ, ℓ'', a, a', a''}` by position-wise distinctness. Against `ℓ = [1, 0, 2, 0, 5, 0, 2, 1]`: disagreement at position 8 (`ℓ_new₈ = 2 ≠ 1 = ℓ₈`) gives `ℓ_new ≠ ℓ`. Against `ℓ'' = [1, 0, 2, 0, 5, 3, 0, 2, 1]`: disagreement at position 6 (`ℓ_new₆ = 0 ≠ 3 = ℓ''₆`) gives `ℓ_new ≠ ℓ''`. Against each content address (`a₇ = a'₇ = 1`, `a''₇ = 0`): disagreement at position 7 (`ℓ_new₇ = 2`) gives `ℓ_new ≠ a, a', a''`.

Other preconditions: `zeros(ℓ_new) = zeros(ℓ) = 3` (B5a, SiblingZerosPreservation — the per-step `zeros(inc(ℓ, 0)) = zeros(ℓ)`, with `zeros(ℓ) = 3` from Step 3's chain exhibition) ✓; `E(ℓ_new) = [2, 2]`, `E(ℓ_new)₁ = 2 = s_L` ✓; `#E(ℓ_new) = 2` ✓; `origin(ℓ_new) = d` (`inc(ℓ, 0)` modifies only position `sig(ℓ) = #ℓ = 8` by TA5(c) and TA5-SigValid placing `sig` at the T4-valid `ℓ`'s terminal position, so positions `1..7` are fixed, including the document-level prefix and the field-separator structure that origin's truncation depends on; hence `origin(ℓ_new) = origin(ℓ) = d`) ✓.

Effect: `L₈ = L₇ ∪ {ℓ_new ↦ (F_new, G_new, Θ_new)} = {ℓ ↦ (F, G, Θ), ℓ'' ↦ (F'', G'', Θ''), ℓ_new ↦ (F_new, G_new, Θ_new)}`; `C₈ = C₇`; `M₈ = M₇`. Invariants at `Σ₈`: L0–L1c hold at the new key as verified above; L3 (triple endset, non-empty `Θ_new`) ✓; L12 (existing values `ℓ ↦ ·` and `ℓ'' ↦ ·` unchanged) ✓; SD ✓; L-fin (`|L₈| = 3`) ✓. ChainMembershipForOrigin extends: `dom(L₈) ∩ {ℓ''' : origin(ℓ''') = d} = {ℓ, ℓ_new} = {s₁, s₂}` (contiguous prefix of `A_L(d)`, witnessing `n_d = 2`).

*Step 9 — `K.σ(d_alt)` (third document registration, prefix-incomparable with prior documents).* Fix `d_alt = [1, 0, 3, 0, 7]` — `#d_alt = 5`, with zeros at positions 2 and 4 (`zeros(d_alt) = 2`), no adjacent zeros (positions (2,3) = (0,3) and (4,5) = (0,7)), `d_alt[1] = 1 ≠ 0` and `d_alt[5] = 7 ≠ 0`, hence T4-valid. By T4b, `N(d_alt) = [1]`, `U(d_alt) = [3]`, `D(d_alt) = [7]`.

Verify `d_alt ∉ dom(M₈) = {d, d'}`. Compare with `d = [1, 0, 2, 0, 5]`: position 3 disagrees (`d[3] = 2 ≠ 3 = d_alt[3]`), so `d_alt ≠ d`. Compare with `d' = [1, 0, 2, 0, 5, 3]`: position 3 disagrees similarly, so `d_alt ≠ d'`. The other K.σ precondition `T4-valid(d_alt) ∧ zeros(d_alt) = 2` ✓.

Effect: `dom(M₉) = {d, d', d_alt}`, with `M₉(d_alt) = ∅` and `M₉(d) = M₉(d') = ∅` unchanged. `C₉ = C₈`, `L₉ = L₈`. Once `d_alt ∈ dom(M₉)`, the chains `A_C(d_alt)` and `A_L(d_alt)` are available, alongside those already available for `d` and `d'`.

*Verifying the Cross-document disjointness lemma at `Σ₉` for the prefix-incomparable pair `(d, d_alt)`.* `d` and `d_alt` are prefix-incomparable — position 3 of `d = [1, 0, 2, 0, 5]` is `2`, of `d_alt = [1, 0, 3, 0, 7]` is `3`, both within native domains, so neither prefixes the other. This document-level divergence at `k = 3` lifts to the anchors `p₁ = b_L(d) = [1, 0, 2, 0, 5, 0, 2]` and `p₂ = b_L(d_alt) = [1, 0, 3, 0, 7, 0, 2]`: `p₁[3] = 2 ≠ 3 = p₂[3]` at `k = 3 ≤ min(#p₁, #p₂) = 7`, witnessing `b_L(d) ⋠ b_L(d_alt) ∧ b_L(d_alt) ⋠ b_L(d)` (Prefix, ASN-0034); the same divergence holds for the content anchors `b_C(d)`, `b_C(d_alt)`. By T10, every link (resp. content) allocated under `d_alt` differs from every one allocated under `d`. The pair `(d', d_alt)` is analogous: `d'[3] = 2 ≠ 3 = d_alt[3]` gives the same position-3 divergence.

*Verifying invariants at `Σ₉`.* M0 holds at `d_alt`: precondition pins `T4-valid(d_alt) ∧ zeros(d_alt) = 2`; M0 at the prior keys `d, d'` transfers by frame on those entries. M1: `{d, d'} ⊆ {d, d', d_alt}`. C0/C1/C1b/C1c/C2/C-fin hold by frame on `C`; C2 carries the prior content keys' origins `d` (for `a, a'`) and `d'` (for `a''`), all preserved by M1's extension. L0/L1/L1a/L1b/L1c/L3/L12/L-fin hold by frame on `L`; L1a carries `origin(ℓ) = origin(ℓ_new) = d` and `origin(ℓ'') = d'`, preserved by M1. SD: `dom(C₉) ∩ dom(L₉) = {a, a', a''} ∩ {ℓ, ℓ'', ℓ_new} = ∅` (verified by L0's `E(·)₁` partition and StoreT4Validity + T7). ChainMembershipForOrigin transfers at `Σ₉`: under `d`, content gives `{a, a'} = {t₁, t₂}` with `m_d = 2` and link gives `{ℓ, ℓ_new} = {s₁, s₂}` with `n_d = 2`; under `d'`, content gives `{a''} = {t₁}` with `m_{d'} = 1` and link gives `{ℓ''} = {s₁}` with `n_{d'} = 1`; under `d_alt`, both intersections are `∅` with `m_{d_alt} = n_{d_alt} = 0` (vacuous, first emissions under `d_alt` still pending). StoreT4Validity transfers by frame on `C` and `L` together with M1's monotonicity preserving the chain-membership witnesses.


## Discharge of stated invariants

**Simultaneous-induction framing.** The stated invariants, together with the ChainMembershipForOrigin lemma and the StoreT4Validity corollary, are proved by *simultaneous induction* over transition sequences from `Σ₀`: the inductive hypothesis at each step is the *conjunction* of every such property at the current state `Σ`, and the inductive step exhibits each holding at `Σ'` using the conjoined IH. The inductive step for the stated invariants is recorded as a per-(invariant, transition) matrix. The inductive step for the two derived results — ChainMembershipForOrigin and StoreT4Validity — is carried by their standalone proofs (Lemma and Corollary, above).

**Base case verification (at `Σ₀ = (∅, ∅, ∅)`).** Most invariants are vacuously satisfied: M0/M2/C1/C1b/C1c/C2/L0/L1/L1a/L1b/L1c/L3 quantify over `dom(C)`, `dom(L)`, or `dom(M)`, all empty at `Σ₀`. C0, M1, and L12 quantify over transitions `Σ → Σ'`, vacuous at `Σ₀` until the first transition fires. Three invariants are non-vacuous but trivially satisfied at `Σ₀`:

- **SD** (`dom(C) ∩ dom(L) = ∅`): at `Σ₀`, both stores empty, so `∅ ∩ ∅ = ∅` — trivially true.
- **L-fin** (`|dom(L)| < ∞`): `|∅| = 0 < ∞` — trivially true.
- **C-fin** (`|dom(C)| < ∞`): `|∅| = 0 < ∞` — trivially true.

*Derived lemmas at Σ₀.* All derived lemmas hold vacuously at `Σ₀` because `dom(C₀)`, `dom(L₀)`, `dom(M₀)` are empty.

The base case holds.

**Inductive step.** Per (invariant, transition):

| Invariant | K.σ | K.α | K.λ |
|---|---|---|---|
| **M0** (DocumentTumblerWellFormed) | Discharged at new key: precondition pins `T4-valid(d) ∧ zeros(d) = 2` | Preserved: `M` in frame | Preserved: `M` in frame |
| **M1** (ArrangementMonotonicity) | Discharged: effect extends `dom(M)` by union | Preserved: `M` in frame | Preserved: `M` in frame |
| **M2** (EmptyArrangement) | Discharged at new key: effect clause pins `M'(d) = ∅`; prior keys preserved by frame on existing `M` entries | Preserved: `M` in frame | Preserved: `M` in frame |
| **C0** (ContentImmutability) | Preserved: `C` in frame | Discharged: effect extends `dom(C)` at fresh `a` with value `v`; value at existing keys unaltered (definitional in effect clause). The freshness `a ∉ dom(C) ∪ dom(L)` that the "fresh `a`" extension consumes is supplied by FirstEmissionFreshness (first-emit branch) / SubsequentEmissionFreshness (subsequent-emit branch) | Preserved: `C` in frame |
| **C1** (ContentElementLevel) | Preserved: `C` in frame | Discharged at new key: first-emit branch pins `a = [d.0.s_C.1]`, whose form gives `zeros(a) = 3`; subsequent-emit branch has `a = inc(a_prev, 0)`, where `zeros(a) = zeros(a_prev) = 3` by B5a (SiblingZerosPreservation, the per-step `zeros(inc(·, 0)) = zeros(·)`) and the IH on `a_prev` — B5a's precondition `a_prev_{sig(a_prev)} > 0` is discharged from the T4-validity of `a_prev` (IH via ChainElementT4Validity), which by TA5-SigValid gives `sig(a_prev) = #a_prev` with non-zero terminal component (T4's `t_{#t} ≠ 0`) | Preserved: `C` in frame |
| **C1b** (ContentElementFieldDepth) | Preserved: `C` in frame — `dom(C)` unchanged, IH transfers | Discharged at new key: first-emit branch pins `a = [d.0.s_C.1]`, whose form gives `#E(a) = 2`; subsequent-emit branch has `a = inc(a_prev, 0)`: by TA5(b) the step preserves every position except `sig(a_prev)`, and for the T4-valid `a_prev`, TA5-SigValid places `sig(a_prev) = #a_prev` in the element field, so the third separator's position is untouched and `zeros(a) = zeros(a_prev)` by B5a (SiblingZerosPreservation); the element-field boundary is therefore invariant, giving `#E(a) = #E(a_prev) ≥ 2` by the IH on `a_prev` | Preserved: `C` in frame |
| **C1c** (ContentAllocatorConformance) | Preserved: `C` in frame | Discharged at new key via the T10a-conforming step sequence (see *C1c chain exhibition* below — first-emit and subsequent-emit cases) | Preserved: `C` in frame |
| **C2** (ContentScopedAllocation) | Preserved: vacuously (no new content); for prior keys `a ∈ dom(C)`, `origin(a) ∈ dom(M) ⊆ dom(M')` (`C` in frame, M1 extends `dom(M)`) | Discharged at new key; `d ∈ dom(M)` from K.α's precondition. *First-emit:* the binding pins `a = [d.0.s_C.1]`, whose document-level prefix is `origin(a) = d` directly. *Subsequent-emit:* the binding pins `a = inc(a_prev, 0)`, so `origin(a) = d` is *derived*, not pinned — `inc(·, 0)` modifies only position `sig(a_prev) = #a_prev` (TA5-SigValid, the element-field terminal position of the T4-valid `a_prev`), leaving the document-level prefix fixed, hence `origin(inc(a_prev, 0)) = origin(a_prev) = d` by the IH on `a_prev`. Preserved at prior keys (`origin(·)` state-independent per State model, M1 extends `dom(M)`) | Preserved: `C` in frame; prior keys preserved by M1 |
| **L0** (SubspacePartition) | Preserved: `L`, `C` in frame | Preserved on L-clause (`L` in frame); discharged at new key on C-clause: `E(a)₁ = s_C` read from the pinned emission — FirstEmission (first-emit) / DisjointSubAllocatorChains (subsequent-emit, `a = inc(a_prev, 0) ∈ A_C(d)`) | Discharged at new key on L-clause: `E(ℓ)₁ = s_L` read from the pinned emission — FirstEmission (first-emit) / DisjointSubAllocatorChains (subsequent-emit); preserved on C-clause (`C` in frame) |
| **L1** (LinkElementLevel) | Preserved: `L` in frame | Preserved: `L` in frame | Discharged at new key: identical to the C1 K.α discharge above under the content↔link substitution (`ℓ`, `ℓ_prev`, `A_L(d)` for `a`, `a_prev`, `A_C(d)`) |
| **L1a** (LinkScopedAllocation) | Preserved: vacuously (no new link); for prior keys `ℓ ∈ dom(L)`, `origin(ℓ) ∈ dom(M) ⊆ dom(M')` (M1 extends `dom(M)`) | Preserved: `L` in frame; prior keys preserved by M1 | Discharged at new key; `d ∈ dom(M)` from K.λ's precondition. *First-emit:* the binding pins `ℓ = [d.0.s_L.1]`, whose document-level prefix is `origin(ℓ) = d` directly. *Subsequent-emit:* the binding pins `ℓ = inc(ℓ_prev, 0)`, so `origin(ℓ) = d` is *derived* — `inc(·, 0)` modifies only position `sig(ℓ_prev) = #ℓ_prev` (TA5-SigValid, the element-field terminal position of the T4-valid `ℓ_prev`), leaving the document-level prefix fixed, hence `origin(inc(ℓ_prev, 0)) = origin(ℓ_prev) = d` by the IH on `ℓ_prev`. Prior keys preserved by M1 |
| **L1b** (LinkElementFieldDepth) | Preserved: `L` in frame — `dom(L)` unchanged, IH transfers | Preserved: `L` in frame — `dom(L)` unchanged, IH transfers | Discharged at new key: identical to the C1b K.α discharge above under the content↔link substitution (`ℓ`, `ℓ_prev`, `A_L(d)` for `a`, `a_prev`, `A_C(d)`) |
| **L1c** (LinkAllocatorConformance) | Preserved: `L` in frame | Preserved: `L` in frame | Discharged at new key via the T10a-conforming step sequence (see *L1c chain exhibition* below — first-emit and subsequent-emit cases) |
| **L3** (NEndsetStructure) | Preserved: `L` in frame | Preserved: `L` in frame | Discharged at new key: precondition pins `|L(ℓ)| ≥ 3 ∧ (A i : 1 ≤ i ≤ N : eᵢ ∈ Endset) ∧ e₃ ≠ ∅` |
| **L12** (LinkImmutability) | Preserved: `L` in frame | Preserved: `L` in frame | Discharged: effect extends `dom(L)` at fresh `ℓ`; value at existing keys unaltered (definitional). The freshness `ℓ ∉ dom(L) ∪ dom(C)` that the "fresh `ℓ`" extension consumes is supplied by FirstEmissionFreshness (first-emit branch) / SubsequentEmissionFreshness (subsequent-emit branch) |
| **SD** (StoreDisjointness) | Preserved: `C`, `L` both in frame, so `dom(C') = dom(C)`, `dom(L') = dom(L)`, IH transfers | Standing consequence of the L0/C1/L1/StoreT4Validity rows at `Σ'`: those premises hold at `Σ'` by their own rows this step, and the SD invariant statement carries the one T7 derivation. This single argument covers both allocation transitions (K.α and K.λ) | As the K.α cell |
| **L-fin** (LinkStoreFiniteness) | Preserved: `L` in frame | Preserved: `L` in frame | Discharged: `|dom(L')| = |dom(L)| + 1`; finiteness closed under +1 |
| **C-fin** (ContentStoreFiniteness) | Preserved: `C` in frame | Discharged: `|dom(C')| = |dom(C)| + 1`; finiteness closed under +1 | Preserved: `C` in frame |


*C1c chain exhibition.* The substrate's C1c is "every content address has a T10a-conforming step sequence from its home document." For `K.α`'s discharge, two sub-cases:

**First-emit case** (`a = [d.0.s_C.1]`, predicate `{a' ∈ dom(C) : origin(a') = d} = ∅`). The T10a-conforming step sequence witnessing C1c is two inc steps from `d`:

  `(t₀, t₁, t₂)` where `t₀ = d`, `t₁ = inc(d, 2) = b_C(d)`, `t₂ = inc(b_C(d), 1) = [d.0.s_C.1] = a`

Per-step admissibility of both steps is the *anchor-construction admissibility* established in the FirstEmission lemma. New here are C1c's chain-bookkeeping clauses: `k₁ = 2` by construction (step 1 above); `n = 2 ≥ 1` ✓; `#t₁ = #d + 2 > #d` and `#t₂ = #d + 3 > #d`, so `(A i : 1 ≤ i ≤ 2 : #tᵢ > #origin(a))` holds.

**Subsequent-emit case** (`a = inc(max{a' ∈ dom(C) : origin(a') = d}, 0)`, predicate `{a' ∈ dom(C) : origin(a') = d} ≠ ∅`). Let `a_prev = max{a' ∈ dom(C) : origin(a') = d}`. By the inductive hypothesis on C1c, `a_prev` has a T10a-conforming step sequence `(t₀, …, t_n)` with `t₀ = d`, `t_n = a_prev`, `k₁ = 2`, and `(A i : 1 ≤ i ≤ n : #tᵢ > #d)`. The chain for `a` extends this by one step: `(t₀, …, t_n, t_{n+1})` with `t_{n+1} = inc(t_n, 0) = inc(a_prev, 0) = a`. Per-step admissibility of the new step `t_{n+1} = inc(a_prev, 0)`: TA5a at `k = 0` is unconditionally T4-preserving (no side condition), so `t_{n+1}` is T4-valid given `a_prev` T4-valid (the latter supplied by ChainElementT4Validity applied to `A_C(d)`'s chain at `a_prev`); TA5(c) at `k = 0` gives the structural form (length preservation, single-position modification at `sig(a_prev)`). C1c's strengthened clauses on the extended chain: `k₁ = 2` is inherited unchanged from the IH chain; `n + 1 ≥ 1` ✓; for the new step, TA5(c) gives `#t_{n+1} = #t_n > #d` (so the universal `#tᵢ > #d` extends to `i = n + 1`).

*L1c chain exhibition.* The substrate's L1c is "every link address has a T10a-conforming step sequence from its home document." For `K.λ`'s discharge, two sub-cases:

**First-emit case** (`ℓ = [d.0.s_L.1]`, predicate `{ℓ' ∈ dom(L) : origin(ℓ') = d} = ∅`). The T10a-conforming step sequence witnessing L1c is three inc steps from `d`:

  `(t₀, t₁, t₂, t₃)` where `t₀ = d`, `t₁ = inc(d, 2) = b_C(d)`, `t₂ = inc(b_C(d), 0) = b_L(d)`, `t₃ = inc(b_L(d), 1) = [d.0.s_L.1] = ℓ`

Per-step admissibility of all three steps is the *anchor-construction admissibility* established in the FirstEmission lemma (the link chain instantiates the full anchor construction `d → b_C(d) → b_L(d)` plus the `k = 1` first emission). New here are L1c's chain-bookkeeping clauses: `k₁ = 2` by construction (step 1 above); `n = 3 ≥ 1` ✓; `#t₁ = #d + 2 > #d`, `#t₂ = #d + 2 > #d`, `#t₃ = #d + 3 > #d`, so `(A i : 1 ≤ i ≤ 3 : #tᵢ > #origin(ℓ))` holds.

**Subsequent-emit case** (`ℓ = inc(max{ℓ' ∈ dom(L) : origin(ℓ') = d}, 0)`, predicate `{ℓ' ∈ dom(L) : origin(ℓ') = d} ≠ ∅`). Identical to the C1c subsequent-emit case above under the content↔link substitution (`ℓ`, `ℓ_prev`, `A_L(d)` for `a`, `a_prev`, `A_C(d)`): the prior link terminus `ℓ_prev = max{ℓ' ∈ dom(L) : origin(ℓ') = d}` extends its IH chain by one `inc(·, 0)` step, and the same TA5a/TA5(c) citations discharge per-step admissibility and the strengthened clauses.


## Properties Introduced

| ID | Name | Status | Source |
|---|---|---|---|
| M0 | DocumentTumblerWellFormed | INV | Substrate |
| M1 | ArrangementMonotonicity | INV | Substrate |
| M2 | EmptyArrangement | INV | Substrate |
| C0 | ContentImmutability | INV | Substrate; restated from ASN-0036 S0/S1 |
| C1 | ContentElementLevel | INV | Substrate; restated from ASN-0036 S7b |
| C1b | ContentElementFieldDepth | INV | Substrate; content-side analog of L1b |
| C1c | ContentAllocatorConformance | INV | Substrate; content-side analog of L1c |
| C2 | ContentScopedAllocation | INV | Substrate; content-side analog of L1a |
| L0 | SubspacePartition | INV | ASN-0043 (L-clause); C-clause added here |
| L1 | LinkElementLevel | INV | ASN-0043 |
| L1a | LinkScopedAllocation | INV | ASN-0043 |
| L1b | LinkElementFieldDepth | INV | ASN-0043 |
| L1c | LinkAllocatorConformance | INV | ASN-0043 |
| L3 | NEndsetStructure | INV | ASN-0043 |
| L12 | LinkImmutability | INV | ASN-0043 |
| SD | StoreDisjointness | INV (derived) | L0 + SC-NEQ + StoreT4Validity + T7 |
| L-fin | LinkStoreFiniteness | INV (derived) | Inductively from `Σ₀.L = ∅` + K.λ |
| C-fin | ContentStoreFiniteness | INV (derived) | Inductively from `Σ₀.C = ∅` + K.α |
| ChainDiscipline | ContentLinkSubAllocatorChainDiscipline | LEMMA (derived) | ASN-0040 SiblingStream |
| ChainElementT4Validity | ChainElementT4Validity | LEMMA (derived) | Source: ASN-0040 B6(a) (ValidDepth sufficiency), applied to `A_C(d) = S(b_C(d), 1)` / `A_L(d) = S(b_L(d), 1)`. |
| ChainEnumerationInjectivity | ChainEnumerationInjectivity | LEMMA (derived) | Source: ASN-0040 S0 (StreamOrdering), applied to `A_C(d)` / `A_L(d)`. |
| DisjointSubAllocatorChains | DisjointSubAllocatorChains | LEMMA (derived) | Source: ASN-0040 B7 (NamespaceDisjointness) + ChainPrefixExtension; `(b_C(d), 1) ≠ (b_L(d), 1)` by SC-NEQ. |
| ChainPrefixExtension | ChainPrefixExtension | LEMMA (derived) | Source: ASN-0040 S1 (StreamPrefix), applied at `p = b_C(d)` / `b_L(d)`. |
| FirstEmission | FirstEmission | LEMMA (derived) | Substrate |
| ChainMembershipForOrigin | ChainMembershipForOrigin | LEMMA | Substrate |
| StoreT4Validity | StoreT4Validity | LEMMA (derived) | Derived from C1c/L1c + T10a.4: each store entry is the terminus of a T10a-conforming chain from its T4-valid document seed, so T4-validity propagates to the terminus. |
| FirstEmissionFreshness | FirstEmissionFreshness | LEMMA (derived) | Substrate; cross-document (T10) + cross-subspace (T7) freshness — premises inline at the lemma above. |
| SubsequentEmissionFreshness | SubsequentEmissionFreshness | LEMMA (derived) | Substrate; within-/cross-document + cross-subspace freshness — premises inline at the lemma above. |
| Cross-doc disjointness | Cross-document disjointness lemma | LEMMA | T10 (PartitionIndependence, ASN-0034) |
| SubspaceConventionAxiom | FixedSubspaceIdentifiers | AXIOM | Substrate commitment; see State model. |
| SequentialTransitionAxiom | SequentialAtomicTransitions | AXIOM | Substrate commitment: `Σ → Σ'` is atomic, uninterruptible, totally ordered. |
| K.σ | DocumentRegistration | OP | Substrate-level document introduction into `dom(M)` |
| K.α | ContentAllocation | OP | Substrate-level content emission |
| K.λ | LinkAllocation | OP | Substrate-level link emission |


## Open Questions

- *Link withdrawal — which invariant must a withdrawal mechanism revisit?* The load-bearing constraint is L12's value-equality clause `L'(a) = L(a)`; any withdrawal mechanism that mutates a link's value must revisit it.

- *Higher-arity link discipline.* L3 admits arbitrary `N ≥ 3`. Should a higher layer impose an upper bound on arity, or constrain slot interpretation or relations between slots — for example, fixing the StandardTriple convention as a structural commitment rather than a notational default?

- *Concurrency.* `K.σ`, `K.α`, and `K.λ` are stated as atomic transitions; what discipline governs concurrent emission across multiple allocators?

- *Sub-allocator stratification beyond `A_C(d)` and `A_L(d)`.* Future subspace identifiers `s ≥ 3` would require parallel sub-allocators; the present axiom commits to exactly two (content and link). What discipline coordinates a third subspace?
