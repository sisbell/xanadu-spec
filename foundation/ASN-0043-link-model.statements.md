> **ASN-0043 · Link Model** — condensed claim statements  
> [← Full note](ASN-0043-link-model.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0043 Claim Statements

*Source: ASN-0043-link-model.md (revised 2026-04-09) — Extracted: 2026-05-31*

## Definition — LinkStore

`Σ.L : T ⇀ Link` is the *link store*, a partial function mapping tumbler addresses to link values. The domain `dom(Σ.L)` is the set of addresses at which links have been created. We specify the type `Link` below.

The full system state is now:

`Σ = (Σ.C, Σ.M, Σ.L)`

where `Σ.C` is the content store (ASN-0036), `Σ.M` is the family of arrangements (ASN-0036), and `Σ.L` is the link store (this ASN).

## Definition — StateExtension

A state `Σ'` *extends* `Σ`, written `Σ' ⊒ Σ`, iff all three stores grow monotonically and agree on the shared domain: `dom(Σ.C) ⊆ dom(Σ'.C)` with `Σ'.C(a) = Σ.C(a)` for every `a ∈ dom(Σ.C)`; `dom(Σ.M) ⊆ dom(Σ'.M)` with `Σ'.M(d) = Σ.M(d)` for every `d ∈ dom(Σ.M)`; and `dom(Σ.L) ⊆ dom(Σ'.L)` with `Σ'.L(a) = Σ.L(a)` for every `a ∈ dom(Σ.L)`. Extension permits each store to acquire new entries but forbids any change to or removal of an existing one.

## Definition — SubspaceI

`subspace_I(a) = E(a)₁` — the first component of the *element field* of an element-level I-address, distinct from ASN-0036's `subspace`. We define `subspace_I` uniformly across every tumbler on which T4b's `E` projection is well-defined — i.e., every T4-valid tumbler `a` with `zeros(a) = 3` and `#E(a) ≥ 1`. T4-validity discharges T4b's domain condition (UniqueParse, ASN-0034); `zeros(a) = 3` together with `#E(a) ≥ 1` ensures the projected element field is non-empty so its first component `E(a)₁` exists.

## Definition — StateLocalInvariants

The *state-local L- and S-invariants* are L0, L1, L1a, L1b, L1c, L3, L5, L6, L14, L14a, L-fin, together with ASN-0036's S0–S3, S7a, S7b, S7d, S8-fin, S8a, S8-depth, D-CTG, D-MIN, D-SEQ.

## Definition — StandardTriple

The standard link form has arity 3, with slot 1 as the *from-endset*, slot 2 as the *to-endset*, and slot 3 as the *type-endset*. We write `(F, G, Θ)` for a link following this convention. Nelson's MAKELINK operation takes these three endsets plus a home document, and Gregory's implementation fixes the arity at 3. The standard triple is the dominant case — but it is a convention, not a structural limit.

*Named accessor.* We introduce the abbreviation `Σ.L(a).type ≡ Σ.L(a).e₃` as a synonym for the indexed accessor of slot 3.

---

## Σ.L — LinkStore (DEF, definition)

`Σ.L : T ⇀ Link` — the link store, mapping addresses to link values; introduced above.

## L-fin — LinkStoreFiniteness (INV, invariant)

For each reachable system state, `dom(Σ.L)` is finite:

`|dom(Σ.L)| < ∞`

This parallels S8-fin (FiniteArrangement, ASN-0036) for arrangements.

## L0 — SubspacePartition (INV, invariant)

Every link address has subspace identifier `s_L`:

`(A a ∈ dom(Σ.L) :: subspace_I(a) = s_L)`

## L0b — LinkAddressValidity (THM, theorem)

Every link address is T4-valid:

`(A a ∈ dom(Σ.L) :: T4-valid(a))`

This lifts the per-address T4-validity of L1c's chain terminus to a universal invariant over `dom(Σ.L)`. Consequently the T4b projections — and hence `home` and `subspace_I` — are well-defined at every `a ∈ dom(Σ.L)`.

## L0a — ContentSubspaceScope (DEF, definition)

*Content-side T4-validity.* By ASN-0036's S7b, every `b ∈ dom(Σ.C)` has `zeros(b) = 3` and well-defined T4b projections; since T4b's definitional domain (UniqueParse, ASN-0034) is precisely the T4-valid subset of `T`, every `b ∈ dom(Σ.C)` is T4-valid. Define:

`dom(Σ.C)|_{s_C} = {a ∈ dom(Σ.C) : subspace_I(a) = s_C}`

— the slice of `dom(Σ.C)` whose addresses occupy subspace `s_C` (`subspace_I` is well-defined on these element-level addresses by the Notational convention's uniform definition). Call a state `Σ` *`s_C`-resident* iff `(A b ∈ dom(Σ.C) :: subspace_I(b) = s_C)` — every stored content address occupies subspace `s_C`.

## L1 — LinkElementLevel (INV, invariant)

Every link address is an element-level tumbler:

`(A a ∈ dom(Σ.L) :: zeros(a) = 3)`

This parallels S7b for content (ASN-0036). A link address carries all four tumbler fields (node, user, document, element), enabling the same structural attribution that content addresses enjoy.

## L1a — LinkScopedAllocation (INV, invariant)

Every link address is allocated under the tumbler prefix of the document whose owner created it. By L0b, `home(a)` is well-defined on every `a ∈ dom(Σ.L)`, so we state the invariant in terms of it directly:

`(A a ∈ dom(Σ.L) :: home(a) ∈ dom(Σ.M))`

The membership clause requires that `home(a)` be an allocated, owned document in the current state.

## L1b — LinkElementFieldDepth (INV, invariant)

Every link address has element field depth at least 2:

`(A a ∈ dom(Σ.L) :: #E(a) ≥ 2)`

This mirrors S8a's `#t ≥ 2` for V-positions (ASN-0036). A link address must carry two distinct element-field components: a *subspace identifier* — the first component `E(a)₁ = s_L`, fixed by L0 — and a *within-subspace ordinal* that follows it.

## L1d — SubspaceDisjointnessLocal (LEMMA, lemma)

Two T4-valid element-level tumblers residing in distinct subspaces are distinct, and consequently links and `s_C`-resident content occupy disjoint address sets.

(a) *Pairwise separation.* For T4-valid `x, y` with `zeros(x) = zeros(y) = 3` and `subspace_I(x) ≠ subspace_I(y)`: `x ≠ y`. This is T7 (SubspaceDisjointness, ASN-0034) in the `subspace_I` notation — T7's precondition (T4-validity and `zeros = 3` on each side) is met, `subspace_I(x) = E(x)₁ ≠ E(y)₁ = subspace_I(y)`, so T7's postcondition gives `x ≠ y`.

(b) *Scoped store disjointness.* `dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`. Every `a ∈ dom(Σ.L)` is T4-valid (L0b) with `zeros(a) = 3` (L1); every `b ∈ dom(Σ.C)` is T4-valid (L0a) with `zeros(b) = 3` (S7b, ElementLevelIAddresses, ASN-0036, a fortiori for `b ∈ dom(Σ.C)|_{s_C}`). For every `a ∈ dom(Σ.L)` and every `b ∈ dom(Σ.C)|_{s_C}`, L0 gives `subspace_I(a) = s_L`, the `s_C`-residence restriction gives `subspace_I(b) = s_C`, and `s_L ≠ s_C`; part (a) then yields `a ≠ b`. Universally instantiating over the product `dom(Σ.L) × dom(Σ.C)|_{s_C}` lifts this pairwise distinctness to the scoped set disjointness `dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`.

## L1c — LinkAllocatorConformance (AXIOM, axiom)

Every link address is a T10a-conforming allocator output (AllocatorDiscipline, ASN-0034) — the T4-valid terminus of an allocation chain seeded at its document-level prefix.

*Chain.* There exists a T4-valid document-level seed `s` and a T10a-conforming step sequence terminating at `a`:

`(A a ∈ dom(Σ.L) :: (E s ∈ T, n ≥ 1, t₀, t₁, ..., tₙ, k₁, ..., kₙ :: T4-valid(s) ∧ zeros(s) = 2 ∧ t₀ = s ∧ tₙ = a ∧ (A i : 1 ≤ i ≤ n : tᵢ = inc(tᵢ₋₁, kᵢ) ∧ kᵢ ∈ {0, 1, 2} ∧ (kᵢ = 1 ⟹ zeros(tᵢ₋₁) ≤ 3) ∧ (kᵢ = 2 ⟹ zeros(tᵢ₋₁) ≤ 2)) ∧ k₁ = 2 ∧ (A i : 1 ≤ i ≤ n : #tᵢ > #s)))`

The first step seats the field-separating zero at position `#s + 1`, between the document prefix and the element field.

*Postconditions.* T4-validity of `a` follows from T10a.4 along the chain. The seed satisfies `s = home(a)`: `a` agrees with `s` on positions `1..#s` and has a zero at position `#s + 1`, so the document-level prefix of `a` is exactly `s`.

## CPP — ChainPrefixPreservation (LEMMA, lemma)

Let `t₀, t₁, ..., tₙ` be a T10a-conforming chain of T4-valid tumblers (T4-validity propagated along the chain by T10a.4), let `p` be a fixed length with `p ≤ #t₀`, and assume the *sibling-advance length precondition*: every sibling-advance step (`kᵢ = 0`) acts on an input strictly longer than `p`, i.e. `#tᵢ₋₁ > p`. Under these hypotheses every step leaves positions `1..p` fixed. A child-spawn `inc(·, k')` (`k' ≥ 1`) agrees with its input on positions `1..#tᵢ₋₁` (TA5(b)); chain lengths are non-decreasing (each step preserves or increases length, by TA5(c)/TA5(d)), so `#tᵢ₋₁ ≥ #t₀ ≥ p` and this agreement covers `1..p`. A sibling advance `inc(·, 0)` modifies only the `sig` position (TA5(c)), which for the T4-valid input is the terminal position `#tᵢ₋₁` (TA5-SigValid); the precondition `#tᵢ₋₁ > p` places that position strictly beyond `p`, so positions `1..p` are again untouched. Then by induction on chain length every `tᵢ`, and in particular the terminus `tₙ`, agrees with `t₀` on positions `1..p`.

## FSP — FreshSiblingConformance (LEMMA, lemma)

Let `Σ` satisfy the state-local L- and S-invariants with `s_C`-resident content. Suppose a tumbler `a` satisfies:

- (h1) *Freshness:* `a ∉ dom(Σ.L)`;
- (h2) *Producibility:* `a` is the terminus of a T10a-conforming chain seeded at a T4-valid document-level tumbler `home(a) ∈ dom(Σ.M)`;
- (h3) *Shape:* `subspace_I(a) = s_L`, `zeros(a) = 3`, `#E(a) ≥ 2`, and `a` is T4-valid.

Let `ℓ = (e₁, ..., e_N)` with `N ≥ 3`, each `eᵢ ∈ Endset` (a finite set of T12-well-formed spans), and `e₃ ≠ ∅`. Define `Σ'` by `Σ'.L = Σ.L ∪ {a ↦ ℓ}`, `Σ'.C = Σ.C`, `Σ'.M = Σ.M`. Then `Σ'` satisfies every state-local L- and S-invariant; the `Σ → Σ'` transition satisfies the transition invariants L12 (LinkImmutability) and L12a (LinkStoreMonotonicity); and `Σ' ⊒ Σ` (StateExtension).

## FSE — FreshSiblingExistence (LEMMA, lemma)

Let `Σ` satisfy L-fin, and let `a ∈ dom(Σ.L)` be a conforming link address (T4-valid, `subspace_I(a) = s_L`, `zeros(a) = 3`, `home(a) ∈ dom(Σ.M)`, producible by an L1c chain). Then there exists `i ≥ 1` with `a' = incⁱ(a, 0) ∉ dom(Σ.L)`, and this `a'` satisfies: `home(a') = home(a)`, `subspace_I(a') = s_L`, `zeros(a') = 3`, `#E(a') = #E(a)`, `a'` T4-valid, and `a'` producible by an L1c chain (the chain for `a` extended by `i` sibling advances).

## L2 — OwnershipEndsetIndependence (LEMMA, lemma)

The home document of a link is determined entirely by the link's address and is independent of the link's endsets. This is an immediate consequence of the `home` definition: `home(a) = N(a).0.U(a).0.D(a)` is computed by T4 field extraction from the address `a` alone, with the endset content `Σ.L(a)` never appearing as an argument.

## L3 — NEndsetStructure (INV, invariant)

Every link in the link store is a sequence of at least three endsets, each in `Endset`, with slot 3 a non-empty type endset:

`(A a ∈ dom(Σ.L) :: |Σ.L(a)| ≥ 3 ∧ (A i : 1 ≤ i ≤ |Σ.L(a)| : Σ.L(a).eᵢ ∈ Endset) ∧ Σ.L(a).e₃ ≠ ∅)`

Nelson [LM 4/79] explicitly calls for N-endset support beyond three: "4-sets, 5-sets ... n-sets supported in link storage and search." Gregory's implementation fixes N = 3, while this model admits N ≥ 3. The non-emptiness conjunct `Σ.L(a).e₃ ≠ ∅` requires a conforming link's type slot to provide a classifying address.

## L4 — EndsetGenerality (META, meta-property)

The spans within an endset may reference any addresses in the tumbler space. There is no constraint confining spans to a single document, to content addresses only, or to addresses at which content currently exists.

The formal content follows from definitions: by L3, every link value is a sequence of endsets of type `Endset = 𝒫_fin(Span)`, where `Span` is the set of well-formed pairs satisfying T12. Therefore:

`(A a ∈ dom(Σ.L), i : 1 ≤ i ≤ |Σ.L(a)|, (s, ℓ) ∈ Σ.L(a).eᵢ :: s ∈ T ∧ (s, ℓ) satisfies T12)`

Beyond T12 well-formedness, the model imposes no constraint on endset spans. The following sub-items make explicit what is left unrestricted:

(a) *Cross-document endsets.* A single endset may contain spans whose start addresses fall under different document-level prefixes.

(b) *Intra-document links.* Nothing prevents a link's endsets from referencing content within the link's own home document.

(c) *Cross-subspace endsets.* Endset spans may reference addresses in the link subspace — that is, addresses of other links.

## L5 — EndsetSetSemantics (INV, invariant)

An endset is an *unordered* set; the ordering of spans within an endset carries no semantic meaning. The substantive commitment is structural, about the operators the model provides, not a fact about sets: the model exposes **no span-positional accessor** within an endset. There is no operator `e.spanⱼ` selecting the j-th span of an endset; span access is by membership `(s, ℓ) ∈ e` only.

## L6 — SlotDistinction (INV, invariant)

The endsets within a link are addressable by slot position. The link model provides a positional accessor `Σ.L(a).eᵢ` returning the i-th endset, defined for every `a ∈ dom(Σ.L)` and every `i ∈ {1, ..., |Σ.L(a)|}`; slot index is a primitive of the model, not a derived label over an unordered collection. Link equality is component-wise tuple equality, by the `Link = {(e₁, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}` definition.

Standard-triple consequence: when `F ≠ G`, `(F, G, Θ) ≠ (G, F, Θ)`; more generally, any slot-permutation that swaps differing entries produces a distinct link value by component-wise tuple inequality.

## L7 — DirectionalFlexibility (META, meta-property)

The invariants L0–L14 and L-fin impose no constraint on which of the from/to slots carries directional significance; any directional interpretation is determined by the link type, outside the link structure.

## L8 — TypeByAddress (DEF, definition)

Type matching is by *address identity*, not by content at the address. Whether two links share the same type is determined by whether their type endsets reference the same addresses, not by what is stored at those addresses:

`same_type(a₁, a₂) ⟺ coverage(Σ.L(a₁).type) = coverage(Σ.L(a₂).type)`

where `coverage(·)` is the address-set projection defined above. The relation is on coverage (the address set referenced by the endset), not on span-set identity: two type endsets with different span decompositions but identical address coverage denote the same type.

*Consequences.* Since the defining criterion is set equality on coverage, `same_type` inherits reflexivity, symmetry, and transitivity directly from `=` on sets, hence is an equivalence relation on `dom(Σ.L)`, partitioning the link store into type-equivalence classes.

## L9 — TypeGhostPermission (LEMMA, lemma)

Ghost types are permitted. For any state `Σ` satisfying the state-local L- and S-invariants, with `dom(Σ.M) ≠ ∅`, and `s_C`-resident (L0a), there exists for every arity `N ≥ 3` a conforming state `Σ'` extending `Σ` (`Σ' ⊒ Σ`, StateExtension) with a link of arity `N` whose type endset references an address outside `dom(Σ'.C) ∪ dom(Σ'.L)`:

`(A Σ : Σ satisfies the state-local L- and S-invariants ∧ dom(Σ.M) ≠ ∅ ∧ Σ s_C-resident : (A N ≥ 3 :: (E Σ' extending Σ, a ∈ dom(Σ'.L) :: |Σ'.L(a)| = N ∧ (E (t, len) ∈ Σ'.L(a).type :: t ∉ dom(Σ'.C) ∪ dom(Σ'.L)) ∧ Σ' satisfies the state-local L- and S-invariants)))`

## PrefixSpanCoverage — PrefixSpanCoverage (LEMMA, lemma)

For any tumbler `x` with `#x ≥ 1`, the unit-depth displacement `δ(1, #x)` (OrdinalDisplacement, ASN-0034) is `[0, ..., 0, 1]` of length `m = #x`, with action point `k = m`; the span `(x, δ(1, m))` is well-formed by T12; and:

`coverage({(x, δ(1, #x))}) = {t ∈ T : x ≼ t}`

*Derivation.* By OrdinalShift (ASN-0034), `shift(x, 1) = x ⊕ δ(1, #x) = x ⊕ δ(1, m)` — the tumbler of length `m` agreeing with `x` on positions `1..m−1` and with last component `x_m + 1`. By T12 (SpanWellDefinedness, ASN-0034), with `δ(1, m) > 0` and action point `m ≤ #x`, the span is well-formed and `coverage({(x, δ(1, m))}) = {t ∈ T : x ≤ t < x ⊕ δ(1, m)} = {t ∈ T : x ≤ t < shift(x, 1)}`. This half-open interval equals `subtree(x) = {t : x ≼ t}` by mutual inclusion (established via T1 cases (i)/(ii)).

## L10 — TypeHierarchyByContainment (LEMMA, lemma)

For type addresses `p, c ∈ T` where `p ≼ c` (p is a prefix of c), define `subtypes(p) = {c ∈ T : p ≼ c}`. By T5 (ContiguousSubtrees, ASN-0034), `subtypes(p)` is a contiguous interval under T1. By PrefixSpanCoverage:

`coverage({(p, δ(1, #p))}) = {t ∈ T : p ≼ t} = subtypes(p)`

A single span query rooted at `p` matches all and only subtypes of `p`.

*Hierarchy inclusion.* The map `p ↦ subtypes(p)` reverses prefix order:

`(A p₁, p₂ ∈ T :: p₁ ≼ p₂ ⟹ subtypes(p₂) ⊆ subtypes(p₁))`

Let `c ∈ subtypes(p₂)`, so `p₂ ≼ c`. We derive `p₁ ≼ c` inline from PrefixRelation's definition (ASN-0034): `p₁ ≼ p₂` means `#p₁ ≤ #p₂` and `(A j : 1 ≤ j ≤ #p₁ : p₂_j = p₁_j)`; `p₂ ≼ c` means `#p₂ ≤ #c` and `(A j : 1 ≤ j ≤ #p₂ : c_j = p₂_j)`. By transitivity of `≤` on naturals, `#p₁ ≤ #c`. For positions `1 ≤ j ≤ #p₁`: since `#p₁ ≤ #p₂`, the range `1..#p₁` is contained in `1..#p₂`, so `c_j = p₂_j` (from the second agreement), and `p₂_j = p₁_j` (from the first), giving `c_j = p₁_j`. By PrefixRelation, `p₁ ≼ c`, i.e., `c ∈ subtypes(p₁)`.

## L11a — LinkUniqueness (LEMMA, lemma)

Distinct T10a-conforming allocation events produce distinct link addresses. Formally, for any pair of allocation events producing link addresses `a₁` and `a₂` in the system, if the events are distinct then `a₁ ≠ a₂` as tumblers. This is GlobalUniqueness (ASN-0034) instantiated at link addresses.

By S7d (DocumentAllocationDiscipline, ASN-0036) each home `home(a) ∈ dom(Σ.M)` is a node of the system's single allocator tree 𝒯 (T4-valid by DocVal). By L1c each link's allocation chain is seeded at that document node and proceeds by T10a steps, so it never leaves 𝒯. Hence both link-producing events are distinct allocation events within the single T10a system 𝒯, which is exactly GlobalUniqueness's precondition; GlobalUniqueness then yields `a₁ ≠ a₂` directly.

## L11b — NonInjectivity (LEMMA, lemma)

The link store imposes no injectivity constraint — multiple addresses may store the same endset sequence:

`(A Σ satisfying the state-local L- and S-invariants and s_C-resident (L0a), a ∈ dom(Σ.L) :: (E Σ' extending Σ, a' ∈ dom(Σ'.L) :: a' ≠ a ∧ Σ'.L(a') = Σ.L(a) ∧ Σ' satisfies the state-local L- and S-invariants))`

The invariants *permit* non-injectivity — every state with a link can be extended to a non-injective state — but they do not *require* it.

*Construction of fresh `a'`.* By FSE applied to the conforming link `a ∈ dom(Σ.L)`, there is a fresh `a' = incⁱ(a, 0) ∉ dom(Σ.L)` with `i ≥ 1` (so `a' ≠ a`), `home(a') = home(a) ∈ dom(Σ.M)`, `subspace_I(a') = s_L`, `zeros(a') = 3`, `#E(a') = #E(a) ≥ 2`, `a'` T4-valid, and `a'` producible by `a`'s L1c chain extended with `i` sibling advances. Define `Σ'` by:

`Σ'.L = Σ.L ∪ {a' ↦ Σ.L(a)}`, `Σ'.C = Σ.C`, `Σ'.M = Σ.M`.

## L12 — LinkImmutability (INV, invariant)

Once created, a link's address persists and its value is permanently fixed:

`(A Σ, Σ' : Σ → Σ' : (A a : a ∈ dom(Σ.L) : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)))`

for every state transition `Σ → Σ'`. This parallels S0 (ContentImmutability, ASN-0036) in both halves: the address endures, and the value at that address — the triple of endsets — never changes.

## L12a — LinkStoreMonotonicity (LEMMA, lemma)

The domain of the link store is monotonically non-decreasing:

`[dom(Σ.L) ⊆ dom(Σ'.L)]`

for every state transition `Σ → Σ'`. This is the direct corollary of L12, paralleling S1 (StoreMonotonicity) for the content store.

## L12b — HomeDocumentPersistence (LEMMA, lemma)

The home documents of all existing links remain allocated across every state transition:

`(A Σ, Σ' : Σ → Σ' :: {home(a) : a ∈ dom(Σ.L)} ⊆ dom(Σ'.M))`

*Derivation.* Let `a ∈ dom(Σ.L)`. By L12a, `a ∈ dom(Σ'.L)`. Applying L1a (LinkScopedAllocation) to `Σ'`: `home(a) ∈ dom(Σ'.M)`. The inclusion `{home(a) : a ∈ dom(Σ.L)} ⊆ dom(Σ'.M)` follows by set-builder closure over `dom(Σ.L) ⊆ dom(Σ'.L)`.

## L13 — ReflexiveAddressing (LEMMA, lemma)

The *canonical span* for an endset reference to a link entity is the unit-depth span at the link's own address (link addresses are admissible endset-span targets by L4(c), cross-subspace endsets). For any link at address `b ∈ dom(Σ.L)`, `b` is an element-level tumbler by L1, so `#b ≥ 1` and PrefixSpanCoverage applies. The unit-depth span `(b, δ(1, #b))` is well-formed, and:

`coverage({(b, δ(1, #b))}) = {t ∈ T : b ≼ t}`

The canonical span contains exactly the target entity and its extensions, with no extraneous tumblers. More generally, an endset *references* an entity at address `a` when `a ∈ coverage(e)`, and `(b, δ(1, #b))` is the canonical span for referencing the entity at `b`.

## L14 — DualPrimitive (INV, invariant)

The set of addresses at which entity values reside is `dom(Σ.C) ∪ dom(Σ.L)`. No state component maps an address outside this union to an entity value. Arrangements `Σ.M(d)` are mappings *between* addresses — they relate V-positions to I-addresses — but V-positions are not entities in their own right. The two domains are disjoint over the `s_C`-resident slice of content, discharged by L1d(b):

`dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`

## L14a — NonTranscludability (INV, invariant)

In any `s_C`-resident system (L0a):

`(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∉ dom(Σ.L))`

This is discharged by S3 together with L1d(b): S3 (ReferentialIntegrity, ASN-0036) requires `(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∈ dom(Σ.C))`; the `s_C`-residence hypothesis places `Σ.M(d)(v)` in `dom(Σ.C)|_{s_C}`; and L1d(b) establishes `dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`, so no V-position image can be a link address.

## coverage(e) — Coverage (DEF, definition)

For an endset `e`, define the *coverage* as the union of the sets denoted by its spans:

`coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})`

This is the set of all tumbler addresses referenced by the endset. Note that coverage is a lossy projection: two endsets with different span decompositions may have identical coverage.

## home(a) — Home (DEF, definition)

For any T4-valid element-level tumbler `a` (so `zeros(a) = 3`), the *home document* is the document-level prefix obtained by field projection:

`home(a) = N(a).0.U(a).0.D(a)`

extracted via T4b's projections `N`, `U`, `D` (UniqueParse, ASN-0034). This is the same field-extraction formula ASN-0036 uses to define `origin` on content addresses, applied here to link addresses.

## Endset — Endset (DEF, definition)

An *endset* is a finite set of well-formed spans:

`Endset = 𝒫_fin(Span)`

where `Span` is the set of well-formed span pairs `(s, ℓ)` satisfying T12 (SpanWellDefinedness, ASN-0034): `ℓ > 0` and the action point `k` of `ℓ` satisfies `k ≤ #s`. The empty set `∅` is a valid endset — a link may have an endset that references nothing.

## Link — Link (DEF, definition)

A *link value* is a finite sequence of N ≥ 3 endsets:

`Link = {(e₁, e₂, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`

We write `|L|` for the *arity* of a link — the number of endsets in the sequence.
