> **ASN-0043 · Link Model** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md)  
> [Condensed statements →](ASN-0043-link-model.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0043: Link Model

*2026-03-16 (revised 2026-04-09, revision 44)*

The two-space model (ASN-0036) established two state components: the content store `Σ.C` — an immutable, append-only mapping from I-addresses to values — and the arrangements `Σ.M(d)` — mutable mappings from V-positions to I-addresses, one per document. Together these give us content: its existence, its identity, and its presentation.

But the docuverse is not merely a store of content. Nelson:

> "The link mechanism ties together the whole corpus of materials on the Xanadu system. There is essentially nothing in the Xanadu system except documents and their arbitrary links." [LM 4/41]

We are looking for the second primitive. Content is substance; what is the nature of connection? We seek the minimum structure that a connection between arbitrary spans of tumbler addresses must have, and the properties that such connections must satisfy.


## Why Connections Need Identity

We begin with a guarantee: the system must support connections between arbitrary spans of content. What must such a connection be?

First, connections must be *distinguishable*. If Alice asserts that paragraph P is a commentary on paragraph Q, and Bob independently makes the same assertion, these are two assertions, not one. Two connections between identical content must coexist as separate objects. Nelson confirms this forcefully: MAKELINK "always creates and always returns a fresh ID" — there is no find-or-create. Gregory's implementation confirms: each call to `docreatelink` allocates a new sequential address; there is no deduplication, no uniqueness constraint, no identity-by-endset.

Second, connections must be *owned*. Alice’s annotation is hers; Bob’s is his. The system must record who made each connection, independently of what it connects — a link’s home document records ownership, not destination.

Third, connections should be *referenceable*. One connection should be able to point to another, enabling compound relational structures — which requires a link to be addressable content that other links can reach.

These three requirements — distinguishability, ownership, referenceability — force connections to be first-class addressed objects in the tumbler space. We are compelled to give connections their own permanent tumbler addresses.

We call these addressed connections *links*.


## The Link Store

We introduce the third component of the system state:

**Definition — LinkStore.** `Σ.L : T ⇀ Link` is the *link store*, a partial function mapping tumbler addresses to link values. The domain `dom(Σ.L)` is the set of addresses at which links have been created. We specify the type `Link` below.

The full system state is now:

`Σ = (Σ.C, Σ.M, Σ.L)`

where `Σ.C` is the content store (ASN-0036), `Σ.M` is the family of arrangements (ASN-0036), and `Σ.L` is the link store (this ASN).

*Notation from ASN-0036.* ASN-0036 introduces `Σ.M(d) : T ⇀ T` as the arrangement of document `d`. We treat `Σ.M` itself as a partial function over the tumbler space — `Σ.M : T ⇀ (T ⇀ T)` — and write `dom(Σ.M) = {d ∈ T : Σ.M(d) is defined}` for *the set of allocated documents* in state `Σ`.

**Definition — StateExtension.** A state `Σ'` *extends* `Σ`, written `Σ' ⊒ Σ`, iff all three stores grow monotonically and agree on the shared domain: `dom(Σ.C) ⊆ dom(Σ'.C)` with `Σ'.C(a) = Σ.C(a)` for every `a ∈ dom(Σ.C)`; `dom(Σ.M) ⊆ dom(Σ'.M)` with `Σ'.M(d) = Σ.M(d)` for every `d ∈ dom(Σ.M)`; and `dom(Σ.L) ⊆ dom(Σ'.L)` with `Σ'.L(a) = Σ.L(a)` for every `a ∈ dom(Σ.L)`. Extension permits each store to acquire new entries but forbids any change to or removal of an existing one.

**L-fin — LinkStoreFiniteness.** For each reachable system state, `dom(Σ.L)` is finite:

`|dom(Σ.L)| < ∞`

This parallels S8-fin (FiniteArrangement, ASN-0036) for arrangements.


## Subspace Residence

Links share the tumbler space `T` with content, but they must be categorically distinguishable from content. A link is not a piece of text. It is a relational assertion *about* text — what Nelson calls a "meta-virtual structure connecting parts of documents (which are themselves virtual structures)." The address space provides a natural mechanism for this categorical distinction: subspace separation.

Recall from ASN-0034 (T4, HierarchicalParsing) that every element-level tumbler has the form `N.0.U.0.D.0.E`, where `E` is the element field, and the first component `E₁` is the subspace identifier. By T7 (SubspaceDisjointness, ASN-0034), tumblers with different first element-field components are pairwise distinct: `a.E₁ ≠ b.E₁ ⟹ a ≠ b`.

*Notational convention.* This ASN introduces the projection `subspace_I(a) = E(a)₁` — the first component of the *element field* of an element-level I-address, distinct from ASN-0036's `subspace`. We define `subspace_I` uniformly across every tumbler on which T4b's `E` projection is well-defined — i.e., every T4-valid tumbler `a` with `zeros(a) = 3` and `#E(a) ≥ 1`. T4-validity discharges T4b's domain condition (UniqueParse, ASN-0034); `zeros(a) = 3` together with `#E(a) ≥ 1` ensures the projected element field is non-empty so its first component `E(a)₁` exists.

The system designates at least two subspaces within each document's element field: one for content, one for links. Let `s_C` and `s_L` be the subspace identifiers for content and links respectively, with `s_C ≠ s_L`; `s_L` is the link subspace identifier introduced by this ASN.

**L1 — LinkElementLevel.** Every link address is an element-level tumbler:

`(A a ∈ dom(Σ.L) :: zeros(a) = 3)`

This parallels S7b for content (ASN-0036). A link address carries all four tumbler fields (node, user, document, element), enabling the same structural attribution that content addresses enjoy. Gregory confirms: link addresses are allocated by `findisatoinsertmolecule` with the `LINKATOM` hint, producing full element-level tumblers.

**Definition — home.** For any T4-valid element-level tumbler `a` (so `zeros(a) = 3`), the *home document* is the document-level prefix obtained by field projection:

`home(a) = N(a).0.U(a).0.D(a)`

extracted via T4b's projections `N`, `U`, `D` (UniqueParse, ASN-0034). This is the same field-extraction formula ASN-0036 uses to define `origin` on content addresses, applied here to link addresses.

**L1c — LinkAllocatorConformance.** Every link address is a T10a-conforming allocator output (AllocatorDiscipline, ASN-0034) — the T4-valid terminus of an allocation chain seeded at its document-level prefix.

*Chain.* There exists a T4-valid document-level seed `s` and a T10a-conforming step sequence terminating at `a`:

`(A a ∈ dom(Σ.L) :: (E s ∈ T, n ≥ 1, t₀, t₁, ..., tₙ, k₁, ..., kₙ :: T4-valid(s) ∧ zeros(s) = 2 ∧ t₀ = s ∧ tₙ = a ∧ (A i : 1 ≤ i ≤ n : tᵢ = inc(tᵢ₋₁, kᵢ) ∧ kᵢ ∈ {0, 1, 2} ∧ (kᵢ = 1 ⟹ zeros(tᵢ₋₁) ≤ 3) ∧ (kᵢ = 2 ⟹ zeros(tᵢ₋₁) ≤ 2)) ∧ k₁ = 2 ∧ (A i : 1 ≤ i ≤ n : #tᵢ > #s)))`

The first step seats the field-separating zero at position `#s + 1`, between the document prefix and the element field.

**CPP — ChainPrefixPreservation (local lemma).** Let `t₀, t₁, ..., tₙ` be a T10a-conforming chain of T4-valid tumblers (T4-validity propagated along the chain by T10a.4), let `p` be a fixed length with `p ≤ #t₀`, and assume the *sibling-advance length precondition*: every sibling-advance step (`kᵢ = 0`) acts on an input strictly longer than `p`, i.e. `#tᵢ₋₁ > p`. Under these hypotheses every step leaves positions `1..p` fixed. A child-spawn `inc(·, k')` (`k' ≥ 1`) agrees with its input on positions `1..#tᵢ₋₁` (TA5(b)); chain lengths are non-decreasing (each step preserves or increases length, by TA5(c)/TA5(d)), so `#tᵢ₋₁ ≥ #t₀ ≥ p` and this agreement covers `1..p`. A sibling advance `inc(·, 0)` modifies only the `sig` position (TA5(c)), which for the T4-valid input is the terminal position `#tᵢ₋₁` (TA5-SigValid); the precondition `#tᵢ₋₁ > p` places that position strictly beyond `p`, so positions `1..p` are again untouched. Then by induction on chain length every `tᵢ`, and in particular the terminus `tₙ`, agrees with `t₀` on positions `1..p`.

**Proof of L1c postconditions.** L1c claims two things of each link address `a`: that `a` is T4-valid, and that the chain seed `s` equals `home(a)`. We discharge them in turn.

*T4-validity of `a`.* By T10a.4 (T4PreservationUnderDiscipline, ASN-0034), every output of a T10a-conforming allocator step is T4-valid given a T4-valid input. The chain begins at the T4-valid seed `s` and proceeds entirely by T10a steps, so by induction on chain length, `tₙ = a` is T4-valid. With T4-validity of `a` established and L1's `zeros(a) = 3` placing `a` at element level, T4b's projections `N(a)`, `U(a)`, `D(a)`, `E(a)` are well-defined; in particular, the document-level prefix `home(a)` (Definition — home) is well-defined.

*`s = home(a)`.* Apply CPP to this chain with `t₀ = s` and `p = #s`. The sibling-advance length precondition holds: the opening step is the child-spawn `k₁ = 2`, which lifts length to `#s + 2` before any sibling advance, and lengths never decrease, so every sibling advance acts on an input of length `≥ #s + 2 > #s = p`. CPP then yields that `a` agrees with `s` on positions `1..#s`. This first invocation says nothing about position `#s + 1`, since CPP's precondition `p ≤ #t₀ = #s` forbids `p = #s + 1`; agreement on `1..#s` alone would permit `a`'s third zero to fall at `#s + 2` or later. We therefore pin the third zero with a second CPP invocation. The child-spawn `k₁ = 2` seats a zero at position `#s + 1` of `t₁`, so `(t₁)_{#s+1} = 0` with `#t₁ = #s + 2` (TA5(d)). Apply CPP to the post-seed sub-chain `t₁, ..., tₙ` with `t₀' = t₁` and `p = #s + 1`: this is valid since `p = #s + 1 ≤ #s + 2 = #t₁`, and every sibling advance after the seed acts on an input of length `≥ #t₁ = #s + 2 > #s + 1 = p` (lengths never decrease below `#t₁`). CPP then yields `a_{#s+1} = (t₁)_{#s+1} = 0`. Combining the two invocations, `a` agrees with `s` on positions `1..#s` and has a zero at position `#s + 1`. Since `s` ends at position `#s` with a positive component (T4-validity of `s`), the third zero of `a` first appears at exactly position `#s + 1`, and the prefix of `a` ending just before that zero is exactly the length-`#s` prefix `s`, which by definition is `home(a)`. Hence `s = home(a)`.

**DocVal — document T4-validity (consequence of S7d + T10a.4).** Let `𝒯` denote the system's single T10a allocator tree, rooted at the T4-valid root allocator that T10a's discipline posits. Every `d ∈ dom(Σ.M)` is T4-valid, with `zeros(d) = 2`. By S7d (DocumentAllocationDiscipline, ASN-0036) such a `d` is the terminus of a T10a-conforming allocator chain from `𝒯`'s root (T4-valid by T10a's root-of-allocator-tree axiom), and T10a.4 (T4PreservationUnderDiscipline, ASN-0034) propagates T4-validity along each chain step.

**L0b — LinkAddressValidity.** Every link address is T4-valid:

`(A a ∈ dom(Σ.L) :: T4-valid(a))`

This lifts the per-address T4-validity of L1c's chain terminus to a universal invariant over `dom(Σ.L)`. Consequently the T4b projections — and hence `home` and `subspace_I` — are well-defined at every `a ∈ dom(Σ.L)`.

**L0 — SubspacePartition.** Every link address has subspace identifier `s_L`:

`(A a ∈ dom(Σ.L) :: subspace_I(a) = s_L)`

**L0a — ContentSubspaceScope.** *Content-side T4-validity.* By ASN-0036's S7b, every `b ∈ dom(Σ.C)` has `zeros(b) = 3` and well-defined T4b projections; since T4b's definitional domain (UniqueParse, ASN-0034) is precisely the T4-valid subset of `T`, every `b ∈ dom(Σ.C)` is T4-valid. Define:

`dom(Σ.C)|_{s_C} = {a ∈ dom(Σ.C) : subspace_I(a) = s_C}`

— the slice of `dom(Σ.C)` whose addresses occupy subspace `s_C` (`subspace_I` is well-defined on these element-level addresses by the Notational convention's uniform definition). Call a state `Σ` *`s_C`-resident* iff `(A b ∈ dom(Σ.C) :: subspace_I(b) = s_C)` — every stored content address occupies subspace `s_C`.

**L1a — LinkScopedAllocation.** Every link address is allocated under the tumbler prefix of the document whose owner created it. By L0b, `home(a)` is well-defined on every `a ∈ dom(Σ.L)`, so we state the invariant in terms of it directly:

`(A a ∈ dom(Σ.L) :: home(a) ∈ dom(Σ.M))`

The membership clause requires that `home(a)` be an allocated, owned document in the current state. Nelson is explicit on this point — a link's home document is "the document under which the link is filed," presupposing an actual document with an owner. This parallels S7a (DocumentScopedAllocation, ASN-0036) for content. Gregory confirms: `docreatelink` allocates the link address within the creating document's address space via `findisatoinsertmolecule`, which extends the document's I-stream. The allocation prefix is determined by the document parameter — a document that must already exist for `docreatelink` to be called.

**L1b — LinkElementFieldDepth.** Every link address has element field depth at least 2:

`(A a ∈ dom(Σ.L) :: #E(a) ≥ 2)`

This mirrors S8a's `#t ≥ 2` for V-positions (ASN-0036). A link address must carry two distinct element-field components: a *subspace identifier* — the first component `E(a)₁ = s_L`, fixed by L0 — and a *within-subspace ordinal* that follows it. The subspace identifier alone cannot address a link, since L0 assigns the same `s_L` to every link in a document; distinguishing siblings (L11a, LinkUniqueness) requires a further ordinal component, so the element field needs at least these two. Gregory confirms: under the `LINKATOM` hint, `findisatoinsertmolecule` allocates the element portion as a fixed subspace digit (`2`, the link subspace) followed by a separately-incremented ordinal digit — the link element field carries exactly these two components, never one.

**L1d — SubspaceDisjointness (local lemma).** Two T4-valid element-level tumblers residing in distinct subspaces are distinct, and consequently links and `s_C`-resident content occupy disjoint address sets.

(a) *Pairwise separation.* For T4-valid `x, y` with `zeros(x) = zeros(y) = 3` and `subspace_I(x) ≠ subspace_I(y)`: `x ≠ y`. This is T7 (SubspaceDisjointness, ASN-0034) in the `subspace_I` notation — T7's precondition (T4-validity and `zeros = 3` on each side) is met, `subspace_I(x) = E(x)₁ ≠ E(y)₁ = subspace_I(y)`, so T7's postcondition gives `x ≠ y`.

(b) *Scoped store disjointness.* `dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`. Every `a ∈ dom(Σ.L)` is T4-valid (L0b) with `zeros(a) = 3` (L1); every `b ∈ dom(Σ.C)` is T4-valid (L0a) with `zeros(b) = 3` (S7b, ElementLevelIAddresses, ASN-0036, a fortiori for `b ∈ dom(Σ.C)|_{s_C}`). For every `a ∈ dom(Σ.L)` and every `b ∈ dom(Σ.C)|_{s_C}`, L0 gives `subspace_I(a) = s_L`, the `s_C`-residence restriction gives `subspace_I(b) = s_C`, and `s_L ≠ s_C`; part (a) then yields `a ≠ b`. Universally instantiating over the product `dom(Σ.L) × dom(Σ.C)|_{s_C}` lifts this pairwise distinctness to the scoped set disjointness `dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`.

Links and `s_C`-resident content cannot share an address. They are peers in the tumbler space — both first-class, both permanent, both addressable — but they are different kinds of entity occupying different regions. Gregory confirms this at the implementation level: the granfilade has exactly two leaf types (`GRANTEXT = 1` and `GRANORGL = 2`), distinguished by an `infotype` discriminator in the bottom crum. Content stores byte sequences; links store pointers to nested enfilades encoding the endset structure. Runtime predicates (`istextcrum`, `islinkcrum`) explicitly test for and separate these two categories.


## Home and Ownership

The home document `home(a)` (Definition — home) determines the link's owner. For the creating document `d`, L1c's `s = home(a)` postcondition gives `home(a) = s = d` directly.

The critical property — the one that distinguishes this design from systems where annotations are embedded in the annotated content:

**L2 — OwnershipEndsetIndependence.** The home document of a link is determined entirely by the link's address and is independent of the link's endsets. This is an immediate consequence of the `home` definition: `home(a) = N(a).0.U(a).0.D(a)` is computed by T4 field extraction from the address `a` alone, with the endset content `Σ.L(a)` never appearing as an argument.

Nelson makes this a first principle: "A link need not point anywhere in its home document. Its home document indicates who owns it, and not what it points to. Conversely, links connecting parts of a document need not reside in that document." This separation of residence from reference is what permits annotation without modification. Your link lives in your document, under your authority, even though its endsets reach into someone else's content. The annotated document is untouched — no byte added, no structure modified, no permission required.


## The Endset Structure

What internal structure must a link have? We seek the minimal structure sufficient for typed, directional connections between arbitrary spans.

A connection has at least two sides — a *source* and a *target*. Without two sides there is no connection. But two sides alone do not suffice: we cannot distinguish a citation from a comment from a refutation by structure alone. If all links are structurally uniform two-endset connections, one cannot ask "find all citations" without also retrieving every comment and footnote. Classification is required.

Nelson's design resolves this not by adding a metadata field — a type tag bolted onto a binary link — but by adding a *third endset*, structurally identical to the first two, pointing into the address space. The type endset is part of every link's identity: Nelson treats it as symmetrical with from and to ["A link's type is specified by yet another end-set, pointing anywhere in the docuverse. This is symmetrical with the other endsets." LM 4/44].

Per Nelson, every link carries a *non-empty* type endset — the type is the link's classifying address and must reference at least one tumbler.

Adding the third endset achieves three things simultaneously:

1. **Extensibility.** Any user can define new types by choosing new addresses, without schema changes. Nelson: "The set of link types is open-ended, and indeed any user may define his or her link types for a particular purpose."

2. **Uniformity.** All endsets have the same representation — a set of spans in the tumbler space. The link is a homogeneous sequence, not a pair-plus-metadata.

3. **Hierarchical classification.** Because tumbler prefix containment is decidable — `p ≼ t` requires only finite component-wise equality (PrefixRelation, ASN-0034), computable from the tumblers alone (T2, IntrinsicComparison) — type addresses support hierarchical relationships: a type at address `p` and a subtype at an address extending `p` are related by prefix ordering. A query matching `p` matches both (by T5, ContiguousSubtrees).

But Nelson's design does not stop at three. We now define the components, admitting arity beyond three.

**Definition — Endset.** An *endset* is a finite set of well-formed spans:

`Endset = 𝒫_fin(Span)`

where `Span` is the set of well-formed span pairs `(s, ℓ)` satisfying T12 (SpanWellDefinedness, ASN-0034): `ℓ > 0` and the action point `k` of `ℓ` satisfies `k ≤ #s`. The empty set `∅` is a valid endset — a link may have an endset that references nothing.

**Definition — Link.** A *link value* is a finite sequence of N ≥ 3 endsets:

`Link = {(e₁, e₂, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`

We write `|L|` for the *arity* of a link — the number of endsets in the sequence.

**Convention — StandardTriple.** The standard link form has arity 3, with slot 1 as the *from-endset*, slot 2 as the *to-endset*, and slot 3 as the *type-endset*. We write `(F, G, Θ)` for a link following this convention. Nelson's MAKELINK operation takes these three endsets plus a home document, and Gregory's implementation fixes the arity at 3. The standard triple is the dominant case — but it is a convention, not a structural limit.

*Named accessor.* We introduce the abbreviation `Σ.L(a).type ≡ Σ.L(a).e₃` as a synonym for the indexed accessor of slot 3.

**L3 — NEndsetStructure.** Every link in the link store is a sequence of at least three endsets, each in `Endset`, with slot 3 a non-empty type endset:

`(A a ∈ dom(Σ.L) :: |Σ.L(a)| ≥ 3 ∧ (A i : 1 ≤ i ≤ |Σ.L(a)| : Σ.L(a).eᵢ ∈ Endset) ∧ Σ.L(a).e₃ ≠ ∅)`

Nelson [LM 4/79] explicitly calls for N-endset support beyond three: "4-sets, 5-sets ... n-sets supported in link storage and search." Gregory's implementation fixes N = 3, while this model admits N ≥ 3. The implementation can store sub-arity links (arity-2, or arity-3 with an empty type slot); such states lie outside this ASN's conforming link store. The non-emptiness conjunct `Σ.L(a).e₃ ≠ ∅` requires a conforming link's type slot to provide a classifying address.


## Endset Properties

Each endset is a set of spans — potentially multiple, potentially discontiguous, potentially spanning multiple documents. Nelson:

> "We see from above that one end of a link may be on a broken, discontiguous set of bytes. This illustrates the endset: a link may be to or from an arbitrary set of bytes. These may be anywhere in the docuverse."

We now state the properties that endsets must satisfy.

**L4 — EndsetGenerality.** The spans within an endset may reference any addresses in the tumbler space. There is no constraint confining spans to a single document, to content addresses only, or to addresses at which content currently exists.

The formal content follows from definitions: by L3, every link value is a sequence of endsets of type `Endset = 𝒫_fin(Span)`, where `Span` is the set of well-formed pairs satisfying T12. Therefore:

`(A a ∈ dom(Σ.L), i : 1 ≤ i ≤ |Σ.L(a)|, (s, ℓ) ∈ Σ.L(a).eᵢ :: s ∈ T ∧ (s, ℓ) satisfies T12)`

Beyond T12 well-formedness, the model imposes no constraint on endset spans. The following sub-items make explicit what is left unrestricted:

(a) *Cross-document endsets.* A single endset may contain spans whose start addresses fall under different document-level prefixes. Gregory confirms: the sporglset data structure stores one `sporgladdress` per span entry, and the conversion function `specset2sporglset` iterates over specset elements with different `docisa` values without rejection. A link whose from-endset touches passages in three different documents is a single link with a single multi-span endset, not three separate links.

(b) *Intra-document links.* Nothing prevents a link's endsets from referencing content within the link's own home document. Nelson: "links connecting parts of a document need not reside in that document" — the converse, that they *may* reside in the document they connect, is equally valid. Heading links, paragraph markers, and footnote links are standard examples of intra-document connections.

(c) *Cross-subspace endsets.* Endset spans may reference addresses in the link subspace — that is, addresses of other links.

**L5 — EndsetSetSemantics.** An endset is an *unordered* set; the ordering of spans within an endset carries no semantic meaning. The substantive commitment is structural, about the operators the model provides, not a fact about sets: the model exposes **no span-positional accessor** within an endset. There is no operator `e.spanⱼ` selecting the j-th span of an endset; span access is by membership `(s, ℓ) ∈ e` only.

Gregory confirms exhaustively. During storage, spans receive sequential V-addresses within the link's own permutation matrix (an artifact of linked-list traversal order). Upon retrieval, spans come back ordered by I-address value, not by insertion sequence — the original ordering is not preserved or recoverable. No code path in the implementation treats any span as "primary" or consults positional index within an endset. All link-finding (`sporglset2linksetinrange`) and intersection (`intersectlinksets`) operations iterate uniformly, comparing addresses by value without regard to position.

**Definition — Coverage.** For an endset `e`, define the *coverage* as the union of the sets denoted by its spans:

`coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})`

This is the set of all tumbler addresses referenced by the endset. Note that coverage is a lossy projection: two endsets with different span decompositions may have identical coverage.


## Slot Distinction and Directionality

Although all endsets within a link are structurally identical (all are elements of `Endset`), they are not interchangeable. Each endset occupies a distinguished position — its slot index — and search can constrain on each slot independently.

**L6 — SlotDistinction.** The endsets within a link are addressable by slot position. The link model provides a positional accessor `Σ.L(a).eᵢ` returning the i-th endset, defined for every `a ∈ dom(Σ.L)` and every `i ∈ {1, ..., |Σ.L(a)|}`; slot index is a primitive of the model, not a derived label over an unordered collection. Link equality is component-wise tuple equality, by the `Link = {(e₁, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}` definition.

Standard-triple consequence: when `F ≠ G`, `(F, G, Θ) ≠ (G, F, Θ)`; more generally, any slot-permutation that swaps differing entries produces a distinct link value by component-wise tuple inequality.

Gregory's implementation encodes this distinction at two independent levels: in the link's own permutation matrix (V-addresses 1.1, 2.1, 3.1 for from, to, and type) and in the spanfilade index (ORGL-range prefixes `LINKFROMSPAN = 1`, `LINKTOSPAN = 2`, `LINKTHREESPAN = 3`). Each slot is indexed under a distinct prefix — the structural witness that slot position is a primitive distinction in the stored form.

But the slot distinction is *structural*, not *semantic*. Whether "from" means "source" and "to" means "destination" is not determined by any invariant of the link structure:

**L7 — DirectionalFlexibility.** The invariants L0–L14 and L-fin impose no constraint on which of the from/to slots carries directional significance; any directional interpretation is determined by the link type, outside the link structure.

Nelson: "A link is typically directional. Thus it has a from-set, the bytes the link is 'from,' and a to-set, the bytes the link is 'to.' (What 'from' and 'to' mean depend on the specific case.)" The word "typically" is deliberate. A citation link is directional — it goes *from* citing text *to* cited source. A counterpart link marking equivalence has no meaningful direction. A heading link populates only one content endset — Nelson calls it "inane" to label that one endset "from." The structure provides two slots; the type defines whether the distinction carries directional weight.


## A Shared Conformance Lemma

The *state-local L- and S-invariants* are L0, L1, L1a, L1b, L1c, L3, L5, L6, L14, L14a, L-fin, together with ASN-0036's S0–S3, S7a, S7b, S7d, S8-fin, S8a, S8-depth, D-CTG, D-MIN, D-SEQ.

**FSP — FreshSiblingConformance (local lemma).** Let `Σ` satisfy the state-local L- and S-invariants with `s_C`-resident content. Suppose a tumbler `a` satisfies:

- (h1) *Freshness:* `a ∉ dom(Σ.L)`;
- (h2) *Producibility:* `a` is the terminus of a T10a-conforming chain seeded at a T4-valid document-level tumbler `home(a) ∈ dom(Σ.M)`;
- (h3) *Shape:* `subspace_I(a) = s_L`, `zeros(a) = 3`, `#E(a) ≥ 2`, and `a` is T4-valid.

Let `ℓ = (e₁, ..., e_N)` with `N ≥ 3`, each `eᵢ ∈ Endset` (a finite set of T12-well-formed spans), and `e₃ ≠ ∅`. Define `Σ'` by `Σ'.L = Σ.L ∪ {a ↦ ℓ}`, `Σ'.C = Σ.C`, `Σ'.M = Σ.M`. Then `Σ'` satisfies every state-local L- and S-invariant; the `Σ → Σ'` transition satisfies the transition invariants L12 (LinkImmutability) and L12a (LinkStoreMonotonicity); and `Σ' ⊒ Σ` (StateExtension).

*Proof.* The construction adds one link-store entry at `a`; `Σ'.C = Σ.C` and `Σ'.M = Σ.M`. We treat the new entry and the carry-over of existing entries.

- *L0.* By L1d(a) (pairwise separation, with `zeros = 3` per side and `s_L ≠ s_C`): `subspace_I(a) = s_L` (h3) and `dom(Σ'.C) = dom(Σ.C)` is `s_C`-resident, so `a` is distinct from every `b ∈ dom(Σ'.C)`, giving `a ∉ dom(Σ'.C)` and preserving `dom(Σ'.L) ∩ dom(Σ'.C)|_{s_C} = ∅`. Existing addresses unchanged.
- *L1.* `zeros(a) = 3` (h3); existing entries by L1 on `Σ`.
- *L1a.* `home(a) ∈ dom(Σ.M) = dom(Σ'.M)` (h2). For existing `b`, `home(b)` depends only on `b`'s fields (unchanged) and `home(b) ∈ dom(Σ.M)` by L1a on `Σ`.
- *L1b.* `#E(a) ≥ 2` (h3); existing entries by L1b on `Σ`.
- *L1c.* By h2, `a` is the terminus of a T10a-conforming chain seeded at `s = home(a)` with `zeros(s) = 2`. L1c's strong conjuncts follow from this seed-equals-home constraint together with `zeros(a) = 3` (h3): only a `k' = 2` step changes the zero count (TA5a; `inc(·, 0)` and `inc(·, 1)` preserve `zeros`, while `inc(·, 2)` seats one separator), so the jump `2 → 3` needs exactly one such step, and it must be first (`k₁ = 2`) — any earlier `k' ∈ {0, 1}` step would advance `s` or occupy position `#s + 1` with a nonzero, pushing the third zero past `#s + 1` and forcing `home(a) ≠ s` — after which every address has length `≥ #s + 2`, so `#tᵢ > #s` for all `i`. Existing entries by L1c on `Σ`.
- *L3.* `|ℓ| = N ≥ 3`, each `eᵢ ∈ Endset`, `e₃ ≠ ∅` by the payload hypothesis (the non-emptiness conjunct constrains slot 3 alone, so empty slots `4..N` are admissible). Existing entries by L3 on `Σ`.
- *L5.* Each endset of `ℓ` is a set under extensional membership; existing entries unchanged.
- *L6.* `ℓ` is an `N`-tuple of endsets with well-defined positional accessors, conforming to the `Link` definition; existing entries unchanged.
- *L12 / L12a (transition).* For every `b ∈ dom(Σ.L)`: `b ∈ dom(Σ'.L)` and `Σ'.L(b) = Σ.L(b)`, since only the entry at `a` is added — this discharges L12 across `Σ → Σ'`, and `dom(Σ.L) ⊆ dom(Σ'.L)` discharges its corollary L12a.
- *`Σ' ⊒ Σ` (StateExtension).* All three conjuncts hold: `Σ'.C = Σ.C` and `Σ'.M = Σ.M` give equality — hence trivially monotone growth with agreement — on the shared domains of `C` and `M`; and `Σ'.L = Σ.L ∪ {a ↦ ℓ}` with `a ∉ dom(Σ.L)` (h1) grows `L` only at the fresh address, so `dom(Σ.L) ⊆ dom(Σ'.L)` with `Σ'.L(b) = Σ.L(b)` for every `b ∈ dom(Σ.L)` (the L12 conjunct just established). Hence `Σ' ⊒ Σ`.
- *L14.* `dom(Σ'.C) ∪ dom(Σ'.L) = dom(Σ.C) ∪ (dom(Σ.L) ∪ {a})`; disjointness over the `s_C`-slice holds since `a` is in `s_L` and `Σ'.C = Σ.C`.
- *L14a.* For every `(d, v)` with `v ∈ dom(Σ'.M(d)) = dom(Σ.M(d))`: `Σ'.M(d)(v) ∈ dom(Σ.C)|_{s_C}` by S3 on `Σ` together with the `s_C`-residence of content; since `dom(Σ'.L) ∩ dom(Σ.C)|_{s_C} = ∅` by L0 (above), `Σ'.M(d)(v) ∉ dom(Σ'.L)`.
- *L-fin.* `dom(Σ'.L) = dom(Σ.L) ∪ {a}` is finite, since `dom(Σ.L)` is finite.
- *ASN-0036 invariants.* `Σ'.C = Σ.C` discharges S0, S1, S7a, S7b verbatim; `Σ'.M = Σ.M` discharges S2, S3, S7d, S8-fin, S8a, S8-depth, D-CTG, D-MIN, D-SEQ verbatim. ∎

We also need the existence of such a fresh sibling:

**FSE — FreshSiblingExistence (local lemma).** Let `Σ` satisfy L-fin, and let `a ∈ dom(Σ.L)` be a conforming link address (T4-valid, `subspace_I(a) = s_L`, `zeros(a) = 3`, `home(a) ∈ dom(Σ.M)`, producible by an L1c chain). Then there exists `i ≥ 1` with `a' = incⁱ(a, 0) ∉ dom(Σ.L)`, and this `a'` satisfies: `home(a') = home(a)`, `subspace_I(a') = s_L`, `zeros(a') = 3`, `#E(a') = #E(a)`, `a'` T4-valid, and `a'` producible by an L1c chain (the chain for `a` extended by `i` sibling advances).

*Proof.* By T10a.7 the `inc(·, 0)` sibling enumeration `a, inc(a, 0), inc²(a, 0), …` is injective and hence infinite; by L-fin, `dom(Σ.L)` is finite, so the least `i ≥ 1` with `incⁱ(a, 0) ∉ dom(Σ.L)` exists — set `a' = incⁱ(a, 0)`, immediately fresh. Each `inc(·, 0)` step modifies only the `sig` position (TA5(c)), which for the T4-valid `a` and its T4-valid outputs (T10a.4) is the terminal position `#·` (TA5-SigValid), incrementing a nonzero component — no separator zero added or removed — so `zeros(a') = zeros(a) = 3` and each step is unconditionally T4-preserving (TA5a, `k = 0`). By T10a.1 (UniformSiblingLength) `#a' = #a`. Since each `inc(·, 0)` step modifies only the terminal position `#a` (established above), `a'` agrees with `a` on every non-terminal position. The home document is fixed by the prefix of `a` up to and including its third (element-separator) zero, which sits at position `#home(a) + 1`; this position is non-terminal — `#home(a) + 1 < #a`, since `#E(a) ≥ 2` (L1b) — so it is left untouched, as are all of positions `1..#home(a)`. Hence `a'` inherits both `a`'s document prefix and its separator zero, giving `home(a') = home(a)`, whence `#E(a') = #E(a)`. The subspace identifier is fixed since `inc(·, 0)` advances the rightmost (ordinal) component (L1b), leaving `subspace_I(a') = s_L`. Producibility follows by extending `a`'s L1c chain with `i` sibling advances, each unconditionally T4-preserving. ∎


## The Type Endset

The type endset deserves extended treatment. It is structurally an endset — a finite set of spans — but its role is semantic classification, and it has distinctive properties that follow from that role.

**L8 — TypeByAddress.** Type matching is by *address identity*, not by content at the address. Whether two links share the same type is determined by whether their type endsets reference the same addresses, not by what is stored at those addresses:

`same_type(a₁, a₂) ⟺ coverage(Σ.L(a₁).type) = coverage(Σ.L(a₂).type)`

where `coverage(·)` is the address-set projection defined above. The relation is on coverage (the address set referenced by the endset), not on span-set identity: two type endsets with different span decompositions but identical address coverage denote the same type.

*Consequences.* Since the defining criterion is set equality on coverage, `same_type` inherits reflexivity, symmetry, and transitivity directly from `=` on sets, hence is an equivalence relation on `dom(Σ.L)`, partitioning the link store into type-equivalence classes.

Nelson: "What the 'type' designation points to is completely arbitrary. This is because of the way we will be searching for links. The search mechanism does not actually look at what is stored under the 'type' it is searching for; it merely considers the type's address."

Gregory confirms at the implementation level: `sporglset2linksetinrange` performs range-overlap matching via `crumqualifies2d` (a half-open interval intersection test), and `intersectlinksets` compares only link ISAs across the three endset queries — there is no span-equality test at any stage of retrieval. The implementation thus indexes by I-address coverage, not by span-set comparison.

Type matching decouples classification from content retrieval: a search for type X never fetches the bytes at address X — it only matches the address. This means:

**L9 — TypeGhostPermission.** Ghost types are permitted. For any state `Σ` satisfying the state-local L- and S-invariants, with `dom(Σ.M) ≠ ∅`, and `s_C`-resident (L0a), there exists for every arity `N ≥ 3` a conforming state `Σ'` extending `Σ` (`Σ' ⊒ Σ`, StateExtension) with a link of arity `N` whose type endset references an address outside `dom(Σ'.C) ∪ dom(Σ'.L)`:

`(A Σ : Σ satisfies the state-local L- and S-invariants ∧ dom(Σ.M) ≠ ∅ ∧ Σ s_C-resident : (A N ≥ 3 :: (E Σ' extending Σ, a ∈ dom(Σ'.L) :: |Σ'.L(a)| = N ∧ (E (t, len) ∈ Σ'.L(a).type :: t ∉ dom(Σ'.C) ∪ dom(Σ'.L)) ∧ Σ' satisfies the state-local L- and S-invariants)))`

*Witness.* Take any conforming `Σ`. Choose a subspace identifier `s_X ∈ ℕ` with `s_X ≥ 1`, `s_X ≠ s_C`, and `s_X ≠ s_L` (such `s_X` exists since ℕ is unbounded and `s_C`, `s_L` are only two values).

*Selection of `d`.* Pick any `d ∈ dom(Σ.M)` (nonempty by the L9 precondition); `d` is T4-valid with `zeros(d) = 2` by DocVal, and `d ∈ dom(Σ'.M)` since `Σ'.M = Σ.M`.

*Construction of `g` (T4-valid ghost address in subspace `s_X`).* Build `g = d.0.s_X.1` — concatenate `[0, s_X, 1]` to `d`. T4-validity: `d` is T4-valid with `zeros(d) = 2`; appending `0` introduces one new zero (so `zeros(g) = 3`); the last component of `d` is strictly positive by T4-validity of `d`, so the new `0` does not create adjacent zeros, and `s_X ≥ 1` separates the new `0` from the trailing `1`; the first component of `g` (inherited from `d`) and the last (`1`) are strictly positive; every non-separator component is strictly positive (inherited components by T4-validity of `d`; `s_X ≥ 1` by construction; `1 > 0` at the tail). T4b's projections therefore apply: `E(g) = [s_X, 1]`, giving `subspace_I(g) = s_X` and `#E(g) = 2`. By L0 applied to `Σ`, `dom(Σ.L) ⊆ {t : subspace_I(t) = s_L}`; by the L9 precondition (`s_C`-residence of content), `dom(Σ.C) ⊆ {t : subspace_I(t) = s_C}`. By L1d(a) (pairwise separation, with `zeros = 3` per side — `g` by construction, content by S7b, links by L1 — and T4-validity per side — `g` by construction, content by L0a, links by L0b — and `s_X` distinct from both `s_C` and `s_L`), `g` is distinct from every content address (subspace `s_C`) and every link address (subspace `s_L`), so `g ∉ dom(Σ.C) ∪ dom(Σ.L)`.

*Choice of `a` (fresh link address under `d`).* By L-fin, `dom(Σ.L)` is finite. We construct `a` explicitly by case analysis on `d`'s prior link-allocation state; the construction yields the freshness `a ∉ dom(Σ.L)` and the producibility chain.

*Case A — `d` has no prior link allocations under `Σ` (`{b ∈ dom(Σ.L) : home(b) = d} = ∅`).* Set `a = d.0.s_L.1`. The producer chain from `d` to `a`: (i) `inc(d, 2)` → `d.0.1` — element field depth 1, subspace 1 (`k' = 2`, requiring `zeros(d) ≤ 2` by TA5a; satisfied since `zeros(d) = 2`); (ii) sibling sweep `inc(·, 0)` from subspace 1 across to subspace `s_L` at element field depth 1, applied `s_L − 1` times — each step a `k = 0` sibling advance, unconditionally T4-preserving (each intermediate `d.0.j` for `j ∈ [2, s_L]` is T4-valid: `zeros = 3`, every non-separator component positive since `j ≥ 1`, no adjacent zeros); (iii) `inc(d.0.s_L, 1)` → `d.0.s_L.1` = `a` — child-spawn to element field depth 2 (`k' = 1`, requiring `zeros(d.0.s_L) ≤ 3` by TA5a; satisfied since `zeros(d.0.s_L) = 3`; the output has `zeros(a) = 3`). Each step conforms to T10a. *Home and freshness:* the chain just exhibited is an L1c-form chain seeded at `d` — opening child-spawn `k₁ = 2`, every subsequent step operating at length `> #d` (steps (ii) and (iii) above), terminating at the T4-valid `a` — so L1c's `s = home(a)` postcondition, instantiated at `s = d`, gives `home(a) = d` directly. The case hypothesis then yields `a ∉ dom(Σ.L)`: any `b ∈ dom(Σ.L)` with `b = a` would have `home(b) = home(a) = d`, contradicting the empty-set hypothesis.

*Case B — `d` has prior link allocations under `Σ` (`{b ∈ dom(Σ.L) : home(b) = d} ≠ ∅`).* Pick any existing link `b ∈ dom(Σ.L)` with `home(b) = d`. By FSE applied to `b`, there is a fresh `a = incⁱ(b, 0) ∉ dom(Σ.L)` with `home(a) = home(b) = d`, `subspace_I(a) = s_L`, `zeros(a) = 3`, `#E(a) = #E(b) ≥ 2`, `a` T4-valid and producible by an L1c chain.

Define `Σ'` as `Σ` extended with the padded payload `Σ'.L(a) = (∅, ∅, {(g, δ(1, #g))}, ∅, ..., ∅)` (`N − 3` empty endsets at slots `4..N`, vacuous when `N = 3`), `Σ'.C = Σ.C`, and `Σ'.M = Σ.M` (the arrangement store is unchanged, since `d ∈ dom(Σ.M)` is reused).

*Application to L9.* FSP's address hypotheses h1–h3 (freshness, producibility, shape) for `a` are established by the Case A / Case B construction above. For the payload `ℓ = (∅, ∅, {(g, δ(1, #g))}, ∅, ..., ∅)`, each `∅ ∈ Endset` and the single span is T12-well-formed since `#g = #d + 3 ≥ 1` gives `δ(1, #g) > 0` with action point `#g`; slot 3 is non-empty, so `ℓ` satisfies FSP's payload hypothesis at every arity `N ≥ 3`. Apply FSP.

By FSP, `Σ'` satisfies every state-local L- and S-invariant, and `Σ' ⊒ Σ` (StateExtension). It remains to establish the L9-specific conclusion that FSP leaves open: the ghost-type disjointness `g ∉ dom(Σ'.C) ∪ dom(Σ'.L)`. The disjointness `g ∉ dom(Σ.C) ∪ dom(Σ.L)` established above transfers to `Σ'` unchanged: `Σ'.C = Σ.C`, and the sole new link address `a ≠ g` lies in subspace `s_L ≠ s_X`, so `g ∉ dom(Σ'.C) ∪ dom(Σ'.L)`. ∎

No property of L0–L14 or L-fin constrains type endset targets to content addresses. Nelson: "Indeed, there is no need for the presence of elements at the addresses specified. Link types may be ghost elements." The type address is a pure name — a position chosen by convention, not a pointer to content that must be dereferenced.

The load-bearing consequence of L8 and L9 together: a type address need contain no stored content at the moment a link names it. The act of choosing the address *is* the act of defining the type, and that act stores nothing at the address it names — the type exists as soon as someone uses it.

**PrefixSpanCoverage (local lemma, span/tumbler algebra).** For any tumbler `x` with `#x ≥ 1`, the unit-depth displacement `δ(1, #x)` (OrdinalDisplacement, ASN-0034) is `[0, ..., 0, 1]` of length `m = #x`, with action point `k = m`; the span `(x, δ(1, m))` is well-formed by T12; and:

`coverage({(x, δ(1, #x))}) = {t ∈ T : x ≼ t}`

*Derivation.* By OrdinalShift (ASN-0034), `shift(x, 1) = x ⊕ δ(1, #x) = x ⊕ δ(1, m)` — the tumbler of length `m` agreeing with `x` on positions `1..m−1` and with last component `x_m + 1`. By T12 (SpanWellDefinedness, ASN-0034), with `δ(1, m) > 0` and action point `m ≤ #x`, the span is well-formed and `coverage({(x, δ(1, m))}) = {t ∈ T : x ≤ t < x ⊕ δ(1, m)} = {t ∈ T : x ≤ t < shift(x, 1)}`. We show this half-open interval equals `subtree(x) = {t : x ≼ t}` by mutual inclusion.

(⊇) Suppose `x ≼ t`. Then `t` agrees with `x` on positions `1..m`. If `#t = m` then `t = x` (T3, CanonicalRepresentation), so `x ≤ t`; if `#t > m` then `x` is a proper prefix of `t` and T1 case (ii) gives `x < t` — either way `x ≤ t`. For the upper bound, `t` and `shift(x, 1)` agree on positions `1..m−1` (both equal `x` there) and at position `m` satisfy `t_m = x_m < x_m + 1 = shift(x, 1)_m`; T1 case (i) at the divergence position `m` gives `t < shift(x, 1)`. Hence `t ∈ {t : x ≤ t < shift(x, 1)}`.

(⊆) Suppose `x ≤ t < shift(x, 1)`; we show `x ≼ t`. If `x = t`, done. Otherwise `x < t`, so by T1 either (ii) `x` is a proper prefix of `t` — giving `x ≼ t` directly — or (i) `x` and `t` first diverge at some `k ≤ m` with `x_k < t_k`. In case (i) we derive a contradiction with `t < shift(x, 1)`. Since `shift(x, 1)` agrees with `x` on positions `1..m−1`, and `t` agrees with `x` on positions `1..k−1`, `t` and `shift(x, 1)` agree on positions `1..k−1`. If `k < m`: at position `k`, `t_k > x_k = shift(x, 1)_k`, so T1 case (i) gives `shift(x, 1) < t`, contradicting `t < shift(x, 1)`. If `k = m`: `t_m > x_m` forces `t_m ≥ x_m + 1 = shift(x, 1)_m`; if `t_m > shift(x, 1)_m`, T1 case (i) gives `shift(x, 1) < t` (contradiction); if `t_m = shift(x, 1)_m`, then `t` agrees with `shift(x, 1)` on positions `1..m`, so either `t = shift(x, 1)` (when `#t = m`) or `shift(x, 1)` is a proper prefix of `t` (T1 case (ii), `shift(x, 1) < t`) — both contradict `t < shift(x, 1)`. Thus case (i) is impossible, and `x ≼ t`.

By T5 (ContiguousSubtrees, ASN-0034), `subtree(x)` is order-convex, confirming that the matched set is exactly the contiguous prefix interval and contains no extraneous tumblers. ∎

**L10 — TypeHierarchyByContainment.** For type addresses `p, c ∈ T` where `p ≼ c` (p is a prefix of c), define `subtypes(p) = {c ∈ T : p ≼ c}`. By T5 (ContiguousSubtrees, ASN-0034), `subtypes(p)` is a contiguous interval under T1. By PrefixSpanCoverage:

`coverage({(p, δ(1, #p))}) = {t ∈ T : p ≼ t} = subtypes(p)`

A single span query rooted at `p` matches all and only subtypes of `p`.

*Hierarchy inclusion.* The map `p ↦ subtypes(p)` reverses prefix order:

`(A p₁, p₂ ∈ T :: p₁ ≼ p₂ ⟹ subtypes(p₂) ⊆ subtypes(p₁))`

Let `c ∈ subtypes(p₂)`, so `p₂ ≼ c`. We derive `p₁ ≼ c` inline from PrefixRelation's definition (ASN-0034): `p₁ ≼ p₂` means `#p₁ ≤ #p₂` and `(A j : 1 ≤ j ≤ #p₁ : p₂_j = p₁_j)`; `p₂ ≼ c` means `#p₂ ≤ #c` and `(A j : 1 ≤ j ≤ #p₂ : c_j = p₂_j)`. By transitivity of `≤` on naturals (NAT-order, ASN-0034), `#p₁ ≤ #c`. For positions `1 ≤ j ≤ #p₁`: since `#p₁ ≤ #p₂`, the range `1..#p₁` is contained in `1..#p₂`, so `c_j = p₂_j` (from the second agreement), and `p₂_j = p₁_j` (from the first), giving `c_j = p₁_j`. By PrefixRelation, `p₁ ≼ c`, i.e., `c ∈ subtypes(p₁)`. A query rooted at a shallower type address therefore subsumes the matches of any query rooted at one of its descendants — the subtype intervals nest in the same direction as the prefix order they encode.

Gregory documents this in the bootstrap document's type registry: `MARGIN` at address `1.0.2.6.2` is hierarchically nested under `FOOTNOTE` at `1.0.2.6`. A query for all footnote-family links, expressed as a span query rooted at `1.0.2.6`, matches both types because `1.0.2.6.2` lies within `[1.0.2.6, 1.0.2.7)`. The subtyping mechanism is the tumbler ordering itself — no separate hierarchy data structure is needed.


## Link Distinctness and Permanence

We now establish the identity semantics of links. The three requirements we began with — distinguishability, ownership, referenceability — crystallize into two derived properties.

**L11a — LinkUniqueness.** Distinct T10a-conforming allocation events produce distinct link addresses. Formally, for any pair of allocation events producing link addresses `a₁` and `a₂` in the system, if the events are distinct then `a₁ ≠ a₂` as tumblers. This is GlobalUniqueness (ASN-0034) instantiated at link addresses.

By S7d (DocumentAllocationDiscipline, ASN-0036) each home `home(a) ∈ dom(Σ.M)` is a node of the system's single allocator tree 𝒯 (T4-valid by DocVal). By L1c each link's allocation chain is seeded at that document node and proceeds by T10a steps, so it never leaves 𝒯. Hence both link-producing events are distinct allocation events within the single T10a system 𝒯, which is exactly GlobalUniqueness's precondition; GlobalUniqueness then yields `a₁ ≠ a₂` directly. Gregory confirms the coordination concretely: all links homed in a document are placed by a single stateless query-and-increment over the `home.0.2.*` subtree of the one global granfilade, guarded by a `homedoc` equality check against cross-document bleed.

**L11b — NonInjectivity.** The link store imposes no injectivity constraint — multiple addresses may store the same endset sequence:

`(A Σ satisfying the state-local L- and S-invariants and s_C-resident (L0a), a ∈ dom(Σ.L) :: (E Σ' extending Σ, a' ∈ dom(Σ'.L) :: a' ≠ a ∧ Σ'.L(a') = Σ.L(a) ∧ Σ' satisfies the state-local L- and S-invariants))`

The invariants *permit* non-injectivity — every state with a link can be extended to a non-injective state — but they do not *require* it.

*Construction of fresh `a'`.* By FSE applied to the conforming link `a ∈ dom(Σ.L)`, there is a fresh `a' = incⁱ(a, 0) ∉ dom(Σ.L)` with `i ≥ 1` (so `a' ≠ a`), `home(a') = home(a) ∈ dom(Σ.M)`, `subspace_I(a') = s_L`, `zeros(a') = 3`, `#E(a') = #E(a) ≥ 2`, `a'` T4-valid, and `a'` producible by `a`'s L1c chain extended with `i` sibling advances. Define `Σ'` by:

`Σ'.L = Σ.L ∪ {a' ↦ Σ.L(a)}`, `Σ'.C = Σ.C`, `Σ'.M = Σ.M`.

*Conformance of `Σ'`.* This is a fresh-sibling extension, so we appeal to FSP (FreshSiblingConformance, *A Shared Conformance Lemma* above). FSP's content-residence hypothesis is discharged by the L11b precondition (`Σ` is `s_C`-resident), which `Σ` satisfies by assumption. FSP's address hypotheses h1 (freshness), h2 (producibility), h3 (shape) for `a'` are exactly the outputs of FreshSiblingExistence above. The payload `ℓ = Σ.L(a)`, an `N`-tuple with `N ≥ 3`, has T12-well-formed spans (inherited from the conforming link `a` by L4 on `Σ`) and satisfies L3 (arity ≥ 3, slot 3 the non-empty type endset, by L3 on `Σ`). By FSP, `Σ'` satisfies every state-local L- and S-invariant, the `Σ → Σ'` transition satisfies L12 and L12a, and `Σ' ⊒ Σ` (StateExtension). The L11b-specific delta is the *endset equality* `Σ'.L(a') = Σ.L(a)`, which holds by construction — this is the non-injectivity witness, not an invariant.

Two links with identical endsets — same from, same to, same type — but different addresses are separate objects, independently owned, independently removable, independently targetable by other links.

**L12 — LinkImmutability.** Once created, a link's address persists and its value is permanently fixed:

`(A Σ, Σ' : Σ → Σ' : (A a : a ∈ dom(Σ.L) : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)))`

for every state transition `Σ → Σ'`. This parallels S0 (ContentImmutability, ASN-0036) in both halves: the address endures, and the value at that address — the triple of endsets — never changes.

The evidence is unambiguous. Nelson's FEBE protocol defines exactly five link operations: MAKELINK (create), FINDLINKSFROMTOTHREE (search), FINDNUMOFLINKSFROMTOTHREE (count), FINDNEXTNLINKSFROMTOTHREE (paginate), and RETRIEVEENDSETS (read). There is no MODIFYLINK, UPDATELINK, or EDITENDSETS. The only write operation is creation; the rest are queries. Gregory confirms at the implementation level: `insertendsetsinorgl` and `insertendsetsinspanf` are called exclusively from `docreatelink`; no other code path writes to the link's orgl or spanfilade entries. The link orgl is written once by `createorglingranf` and never touched again.

**L12a — LinkStoreMonotonicity.** The domain of the link store is monotonically non-decreasing:

`[dom(Σ.L) ⊆ dom(Σ'.L)]`

for every state transition `Σ → Σ'`. This is the direct corollary of L12, paralleling S1 (StoreMonotonicity) for the content store.

**L12b — HomeDocumentPersistence.** The home documents of all existing links remain allocated across every state transition:

`(A Σ, Σ' : Σ → Σ' :: {home(a) : a ∈ dom(Σ.L)} ⊆ dom(Σ'.M))`

*Derivation.* Let `a ∈ dom(Σ.L)`. By L12a, `a ∈ dom(Σ'.L)`. Applying L1a (LinkScopedAllocation) to `Σ'`: `home(a) ∈ dom(Σ'.M)`. The inclusion `{home(a) : a ∈ dom(Σ.L)} ⊆ dom(Σ'.M)` follows by set-builder closure over `dom(Σ.L) ⊆ dom(Σ'.L)`.

This is the link-side dual of S7a's persistence guarantee for content-bearing documents.


## Reflexive Addressing

Because links have tumbler addresses (L0, L1), and endsets can reference any tumbler address (L4), endsets can reference link addresses. This enables *link-to-link* connections — a link whose endset points at another link's address.

**L13 — ReflexiveAddressing.** The *canonical span* for an endset reference to a link entity is the unit-depth span at the link's own address (link addresses are admissible endset-span targets by L4(c), cross-subspace endsets). For any link at address `b ∈ dom(Σ.L)`, `b` is an element-level tumbler by L1, so `#b ≥ 1` and PrefixSpanCoverage applies. The unit-depth span `(b, δ(1, #b))` is well-formed, and:

`coverage({(b, δ(1, #b))}) = {t ∈ T : b ≼ t}`

The canonical span contains exactly the target entity and its extensions, with no extraneous tumblers. More generally, an endset *references* an entity at address `a` when `a ∈ coverage(e)`, and `(b, δ(1, #b))` is the canonical span for referencing the entity at `b`.

Nelson: "Because of the universality of tumbler-space, and the fact that links are located there as well as data, it becomes easy for a link to point at another link (or, indeed, to point at several). The to-set of the link need simply point to the actual link address in the tumbler line, with a span of 1 to designate that unit only."

Nelson's "span of 1" is the informal rendering of `δ(1, #b)`: advance by 1 at the depth of the target address.

Gregory confirms at the implementation level that this is not merely theoretically possible but architecturally unavoidable. The type `typeisa` is `typedef tumbler typeisa` — a bare tumbler with no type discriminant. The endset conversion functions (`specset2sporglset`, `vspanset2sporglset`) accept any tumbler address without checking whether it refers to content or to a link. The insertion functions (`insertspanf`, `insertpm`) store whatever `sporgladdress` they receive, with no type validation. The retrieval function (`findorgl`) resolves any address that maps to a valid granfilade entry, regardless of its atom type. There is no code, at any layer, that draws a boundary between "addressable objects" and "non-addressable objects."

From L13, arbitrary relational structures can be composed:

> "Complex relational structures, such as the faceted link, may be constructed with links to links. These use the two-sided link structure much like the CONS cell in LISP, and may be built into arbitrary compound links."

The link plays the same role for structured connections that the cons cell plays for structured data: a universal building block from which compound relational structures of arbitrary complexity are assembled by chaining. This model admits both realizations: a faceted link may be built by chaining links via link-to-link references (L13), or realized directly as a single link of arity `N` (L3).


## The Dual-Primitive Architecture

We can now state the architectural consequence that unifies the preceding properties. The docuverse is built from exactly two kinds of stored entity:

**L14 — DualPrimitive.** The set of addresses at which entity values reside is `dom(Σ.C) ∪ dom(Σ.L)`. No state component maps an address outside this union to an entity value. Arrangements `Σ.M(d)` are mappings *between* addresses — they relate V-positions to I-addresses — but V-positions are not entities in their own right. The two domains are disjoint over the `s_C`-resident slice of content, discharged by L1d(b):

`dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`

Nelson: "In the present implementation, the only entities actually stored in tumbler-space are content bytes and links. While a number on the line may represent a document or an account, that doesn't mean there's an object stored for it. What's stored is the contents — bytes and links." Documents, accounts, servers, and nodes are organizational concepts — positions in the tumbler hierarchy that structure the address space — but they have no stored representation. Only content and links occupy storage.

Gregory confirms with emphasis: the granfilade's union type has exactly two variants (`GRANTEXT` and `GRANORGL`), the hint mechanism accepts exactly two atom types (`TEXTATOM = 1` and `LINKATOM = 2`), and the Vstream within each document is partitioned into exactly two regions (text at `1.x`, links at `2.x`). No third category exists.

The two primitives are peers. Both have permanent tumbler addresses. Both are stored in the same master index (the granfilade). Both support the same addressing and containment mechanisms. But they are categorically different:

| | Content | Links |
|---|---|---|
| State component | `Σ.C : T ⇀ Val` | `Σ.L : T ⇀ Link` |
| Subspace | `s_C` | `s_L` |
| Payload | Opaque values (bytes) | Structured endset sequences (N ≥ 3; standard triple by convention, slot 3 the type) |
| Sharing | Transcludable — same I-address in multiple arrangements (S5) | Non-transcludable — arrangements cannot map to link addresses (L14a) |
| Address determines | Content origin (S7) | Link home and owner (L2) |

Content identity is *shareable*: the same I-address can appear in the arrangements of multiple documents via transclusion, and this sharing is the mechanism for content reuse (S5, ASN-0036). Link identity is *non-transcludable*: no arrangement may map a V-position to a link address. We state this as an explicit invariant:

**L14a — NonTranscludability.** In any `s_C`-resident system (L0a):

`(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∉ dom(Σ.L))`

A connection is an assertion by a specific principal about specific content, and assertions are not transferable by reference. A link at address `a` is homed in `home(a)` and owned by the principal of `home(a)` — period. It cannot be transcluded into another owner's authority.

This is discharged by S3 together with L1d(b): S3 (ReferentialIntegrity, ASN-0036) requires `(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∈ dom(Σ.C))`; the `s_C`-residence hypothesis places `Σ.M(d)(v)` in `dom(Σ.C)|_{s_C}`; and L1d(b) establishes `dom(Σ.L) ∩ dom(Σ.C)|_{s_C} = ∅`, so no V-position image can be a link address.


## Worked Example

We construct a minimal conforming state to verify that L0–L14 hold simultaneously.

**Setup.** Node 1, user 1, document 1. The content subspace identifier is `s_C = 1`, the link subspace identifier is `s_L = 2`, and the ghost-type subspace identifier is `s_X = 3` — a fresh subspace disjoint from both `s_C` and `s_L`, used only by the type endset of the link below.

Content addresses have element field starting with 1; link addresses have element field starting with 2. The document prefix is `1.0.1.0.1`.

**Content store.** Two content characters at addresses:

- `c₁ = 1.0.1.0.1.0.1.1` — first character, element field `1.1`
- `c₂ = 1.0.1.0.1.0.1.2` — second character, element field `1.2`

So `Σ.C = {c₁ ↦ v₁, c₂ ↦ v₂}` for some values `v₁, v₂ ∈ Val`.

**Arrangement.** One document `d = 1.0.1.0.1` with `Σ.M(d) = {[1.1] ↦ c₁, [1.2] ↦ c₂}` (V-positions are element-field tumblers within the document).

**Link store.** One link — a citation from `c₁` to `c₂` with a ghost type — at address:

- `a = 1.0.1.0.1.0.2.1` — element field `2.1` (subspace 2, ordinal 1)

Choose a ghost type address `g = 1.0.1.0.1.0.3.1` — an element-level tumbler in the same document `d` as the link, with element field `[s_X, 1] = [3, 1]`, so `subspace_I(g) = 3 = s_X`. Define:

All addresses here have depth 8, so the unit-depth displacement is `δ(1, 8) = [0, 0, 0, 0, 0, 0, 0, 1]`.

- From-endset: `F = {(c₁, δ(1, 8))}` (action point `k = 8 = #c₁`, unit width)
- To-endset: `G = {(c₂, δ(1, 8))}`
- Type-endset: `Θ = {(g, δ(1, 8))}`

So `Σ.L = {a ↦ (F, G, Θ)}`.

**Verification.** (L7, a META property imposing no constraint on any state, carries no per-state check and is omitted.)

*L0 (SubspacePartition).* `subspace_I(a) = 2 = s_L`, `subspace_I(c₁) = subspace_I(c₂) = 1 = s_C`; by L1d(a) (pairwise separation, with `zeros = 3` per side and distinct subspaces), `a ≠ c₁` and `a ≠ c₂`, so `dom(Σ.L) ∩ dom(Σ.C) = ∅`. ✓

*L1 (LinkElementLevel).* `zeros(a) = zeros(1.0.1.0.1.0.2.1) = 3`. ✓

*L1a (LinkScopedAllocation).* `home(a) = 1.0.1.0.1 = d`, the creating document; `d ∈ dom(Σ.M)` since `Σ.M(d) = {[1.1] ↦ c₁, [1.2] ↦ c₂}` is defined above — `d` is an allocated, owned document. ✓

*L1b (LinkElementFieldDepth).* `E(a) = [2, 1]`, so `#E(a) = 2 ≥ 2`. ✓

*L1c (LinkAllocatorConformance).* The link address `a = 1.0.1.0.1.0.2.1` is producible by a T10a-conforming allocator from the document prefix `d = 1.0.1.0.1` (taken as the L1c seed `s = d`; T4-valid with `zeros(s) = 2`): (i) `inc(s, 2)` → `1.0.1.0.1.0.1` — element depth 1, subspace 1 (`k₁ = 2` with `zeros(s) = 2`, satisfying TA5a: `k = 2` requires `zeros ≤ 2`); (ii) `inc(1.0.1.0.1.0.1, 0)` → `1.0.1.0.1.0.2` — sibling advance to subspace 2 (`k₂ = 0`, unconditionally T4-preserving, zero count preserved at 3); (iii) `inc(1.0.1.0.1.0.2, 1)` → `1.0.1.0.1.0.2.1` = `a` — child at depth 2 (`k₃ = 1`, requiring `zeros(1.0.1.0.1.0.2) ≤ 3` by TA5a; satisfied since `zeros(1.0.1.0.1.0.2) = 3`; the output has `zeros(a) = 3`). Each step conforms to T10a; only the first step is `kᵢ = 2` (TA5a's `zeros ≤ 2` precondition fires once at `k₁ = 2` and cannot fire again, since `zeros(t₁) = 3` thereafter). Chain shape: `t₀ = s = d` (the T4-valid document-level seed); `k₁ = 2`; intermediate lengths are `#t₁ = #t₂ = 7 > 5 = #s` and `#t₃ = #a = 8 > 5` — every step after the seed operates above `#s`. Postconditions: T4-validity of `a` follows from T10a.4 along the chain; `s = home(a) = 1.0.1.0.1` follows from CPP (the third zero of `a` first appears at position `#s + 1 = 6`, the one seated by `k₁ = 2`). ✓

*L2 (OwnershipEndsetIndependence).* `home(a) = 1.0.1.0.1`, computed from the field structure of `a` alone. The endsets `(F, G, Θ)` are not consulted. ✓

*L3 (NEndsetStructure).* `|Σ.L(a)| = 3 ≥ 3`, slot 3 is the type endset `Θ = {(g, δ(1, 8))} ≠ ∅`, and each endset is in `𝒫_fin(Span)`. ✓

*L4 (EndsetGenerality).* Each span is well-formed by T12: for `(c₁, δ(1, 8))`, `δ(1, 8) > 0` and the action point `k = 8 ≤ #c₁ = 8`. Similarly for the other spans. Start addresses are in `T`. ✓

*L5 (EndsetSetSemantics) at `Σ` — singleton case.* Each endset at `Σ` is a singleton set, so set semantics hold trivially here. L5's substantive content — order-irrelevance and extensional equality across a `≥ 2`-span endset — has no singleton witness. ✓

*L6 (SlotDistinction).* `Σ.L(a) = (F, G, Θ)` is a 3-tuple of endsets, with positional accessors `Σ.L(a).e₁ = F`, `Σ.L(a).e₂ = G`, `Σ.L(a).e₃ = Θ` well-defined. Standard-triple consequence: since `F ≠ G`, `(F, G, Θ) ≠ (G, F, Θ)` by component-wise tuple inequality at slot 1. ✓

*L8 (TypeByAddress) at `Σ` — reflexivity.* The single-link state admits a non-vacuous reflexivity check: `same_type(a, a) ⟺ coverage(Σ.L(a).type) = coverage(Σ.L(a).type)`. The right-hand side is a set-equality of identical sets, true by reflexivity. To exhibit the actual coverage concretely, `Σ.L(a).type = Θ = {(g, δ(1, 8))}`; since `#g = 8`, PrefixSpanCoverage applies, giving `coverage({(g, δ(1, 8))}) = {t ∈ T : g ≼ t}` — the set of all tumblers extending `g`. This is the address set against which any other link's type would be compared under L8's coverage-equality criterion. ✓

*L9 (TypeGhostPermission) at `Σ` — ghost-type disjointness.* The ghost type address `g = 1.0.1.0.1.0.3.1` is disjoint from every stored address, verified by direct enumeration. The stores are `dom(Σ.C) = {c₁, c₂}` and `dom(Σ.L) = {a}`, so `dom(Σ.C) ∪ dom(Σ.L) = {c₁, c₂, a}` — three entries to check. Each address here is element-level (`zeros = 3`), so T7 (SubspaceDisjointness, ASN-0034) applies: two element-level tumblers differing in the first element-field component are distinct. `subspace_I(g) = 3`, while `subspace_I(c₁) = subspace_I(c₂) = 1` and `subspace_I(a) = 2`; since `3 ∉ {1, 2}`, T7 gives `g ≠ c₁`, `g ≠ c₂`, and `g ≠ a`. Hence `g ∉ dom(Σ.C) ∪ dom(Σ.L)`. The link `a` itself is the arity-3 witness for L9's existential: its type endset `Θ = {(g, δ(1, 8))}` references `g`, an address outside `dom(Σ.C) ∪ dom(Σ.L)`. ✓

*L10 (TypeHierarchyByContainment).* For the ghost type at `g = 1.0.1.0.1.0.3.1`, define a parent type `p = 1.0.1.0.1.0.3` with displacement `δ(1, 7) = [0, 0, 0, 0, 0, 0, 1]` (action point `k = 7 = #p`). The coverage of `(p, δ(1, #p))` is `{t : p ≤ t < shift(p, 1)} = {t : 1.0.1.0.1.0.3 ≤ t < 1.0.1.0.1.0.4}`. Since `g = 1.0.1.0.1.0.3.1` and `p ≼ g`, by T1(ii) `g ≥ p`, and `g < 1.0.1.0.1.0.4` because `g` agrees with `p` at position 7 (both have value 3) while `inc(p, 0)` has value 4 there. So `g ∈ coverage({(p, δ(1, #p))})` — a single span query at `p` matches the subtype at `g`. ✓

*L11a (LinkUniqueness).* `a` was produced by forward allocation. With `|dom(Σ.L)| = 1`, no collision is possible. ✓

*L11b (NonInjectivity).* `a ∈ dom(Σ.L)` satisfies the universal quantifier's precondition; the single-link state `Σ` trivially admits the permission, with no distinct addresses to collide. ✓

*L12, L12a (transition invariants).* These constrain state transitions, not individual states, so they impose no condition on `Σ` in isolation. ✓

*L14 (DualPrimitive).* `dom(Σ.C) ∪ dom(Σ.L) = {c₁, c₂, a}`. All stored entities. `dom(Σ.C) ∩ dom(Σ.L) = ∅`. ✓

*L14a (NonTranscludability).* `ran(Σ.M(d)) = {c₁, c₂}`. For each `v ∈ dom(Σ.M(d))`: `Σ.M(d)(v) ∈ {c₁, c₂} ⊆ dom(Σ.C)`, and `dom(Σ.L) = {a}` with `a ∉ {c₁, c₂}` (by L0). So `Σ.M(d)(v) ∉ dom(Σ.L)`. ✓

*L-fin (LinkStoreFiniteness) at `Σ` (state-local).* `|dom(Σ.L)| = 1`, which is finite. ✓

*S2 (ArrangementFunctionality, ASN-0036).* `Σ.M(d) = {[1.1] ↦ c₁, [1.2] ↦ c₂}` assigns each V-position a single image: `[1.1] ↦ c₁` and `[1.2] ↦ c₂`, with no V-position mapped to two distinct I-addresses. ✓

*S3 (ReferentialIntegrity, ASN-0036).* `ran(Σ.M(d)) = {c₁, c₂} ⊆ dom(Σ.C)`. ✓

*S7a (DocumentScopedAllocation, ASN-0036).* For each content address: `N(c₁).0.U(c₁).0.D(c₁) = 1.0.1.0.1 = d ∈ dom(Σ.M)`, and identically for `c₂`. Both content addresses sit under their allocated home document. ✓

*S7b (ElementLevelIAddresses, ASN-0036).* `zeros(c₁) = zeros(1.0.1.0.1.0.1.1) = 3` and `zeros(c₂) = zeros(1.0.1.0.1.0.1.2) = 3`; both T4-valid (no adjacent zeros, every non-separator component positive). T4b's projections `N, U, D, E` are therefore well-defined on each. ✓

*S7d (DocumentAllocationDiscipline, ASN-0036).* `dom(Σ.M) = {d}` with `d = 1.0.1.0.1`; `zeros(d) = 2`, T4-valid by DocVal; `d` is producible by a single `inc(r, 2)` allocation event from a user-level root `r = 1.0.1` (`zeros(r) = 1 ≤ 2`, satisfying TA5a's side condition for `k' = 2`): `inc(1.0.1, 2)` appends one separator zero and a final `1`, yielding `1.0.1.0.1 = d`, so `d` is a T10a-allocated node in 𝒯. ✓

*S8a (VPositionWellFormedness, ASN-0036).* S8a requires `dom(Σ.M(d)) ⊆ {t ∈ T : zeros(t) = 0 ∧ #t ≥ 2}` with every component positive. The V-positions of `Σ.M(d)` are `{[1, 1], [1, 2]}`. For each: `zeros([1, 1]) = zeros([1, 2]) = 0` (no component is zero); `#[1, 1] = #[1, 2] = 2 ≥ 2`; and every component of each is positive (`1 > 0` throughout). All three conjuncts hold for both V-positions. ✓

*S8-depth (FixedDepthVPositions, ASN-0036).* Each V-position of `Σ.M(d)` has element-field depth `2`: `#[1.1] = #[1.2] = 2 ≥ 2`. ✓

*S8-fin (FiniteArrangement, ASN-0036).* `dom(Σ.M(d)) = {[1.1], [1.2]}`, so `|dom(Σ.M(d))| = 2 < ∞`; consequently `ran(Σ.M(d)) = {c₁, c₂}` is finite. ✓

*D-CTG (VContiguity, ASN-0036).* The V-position set `V_1(d) = dom(Σ.M(d)) = {[1, 1], [1, 2]}` is contiguous along its second component at fixed subspace `1`: positions `[1, 1]` and `[1, 2]` cover `{[1, k] : 1 ≤ k ≤ 2}` with no gaps. ✓

*D-MIN (VMinimumPosition, ASN-0036).* `V_1(d) = {[1, 1], [1, 2]} ≠ ∅`, and its T1-least element is `min(V_1(d)) = [1, 1] = [1, 1, ..., 1]` of length `m_1 = 2` — the all-ones tumbler at the common text-subspace depth. This is the witness the D-SEQ check below invokes. ✓

*D-SEQ (SequentialPositions, ASN-0036).* `V_1(d) = {[1, k] : 1 ≤ k ≤ 2}` is a contiguous arithmetic sequence of element-field tumblers at depth 2, starting at the D-MIN witness `[1, 1]` and advancing by `inc(·, 0)` to `[1, 2]`. ✓

**Extension: L11b non-injectivity, L13, and transition verification.**

We extend the state in six steps, naming each intermediate state, to verify L11b, L12, L13, the higher-arity/discrimination behavior of L3/L6/L8, and the multi-span content of L5/L8 non-vacuously.

*Each added link is a fresh sibling.* Each of `a'`, `a₂`, `a₃`, `a₄`, `a₅`, `a₆` is the next `inc(·, 0)` sibling of the previous link, so each step is a fresh-sibling FSP extension, discharging L12 (LinkImmutability) and L12a (LinkStoreMonotonicity).
*Step 1: adding `a'`.* Define `a' = 1.0.1.0.1.0.2.2` with `Σ_1.L(a') = (F, G, Θ)` — same endsets as `a`. The intermediate state is `Σ_1` with `Σ_1.L = {a ↦ (F, G, Θ),\; a' ↦ (F, G, Θ)}`, `Σ_1.C = Σ.C`, `Σ_1.M = Σ.M`.

*L11b non-injectivity in `Σ_1`.* `|dom(Σ_1.L)| = 2`, `a ≠ a'`, and `Σ_1.L(a) = Σ_1.L(a') = (F, G, Θ)`. The link store is non-injective — two distinct addresses map to the same triple. This is the witness for L11b applied to `Σ` with `a`. ✓

*Step 2: adding the meta-link `a₂`.* Define the span targeting `a`: `δ(1, 8) = [0, 0, 0, 0, 0, 0, 0, 1]` has action point `k = 8 = #a`, and `k ≤ #a` holds, so `(a, δ(1, 8))` is well-formed by T12. ✓

Define the meta-link:

- From-endset: `F₂ = {(a, δ(1, 8))}` — pointing at the first link
- To-endset: `G₂ = {(c₂, δ(1, 8))}` — pointing at content
- Type-endset: `Θ₂ = {(g, δ(1, 8))}` — same ghost type
Allocate `a₂ = 1.0.1.0.1.0.2.3` — the next link-subspace sibling after `a'` via `inc(·, 0)`. The final state is `Σ_2` with `Σ_2.L = {a ↦ (F, G, Θ),\; a' ↦ (F, G, Θ),\; a₂ ↦ (F₂, G₂, Θ₂)}`, `Σ_2.C = Σ_1.C`, `Σ_2.M = Σ_1.M`.

*L13 (ReflexiveAddressing).* The from-endset of `a₂` contains the span `(a, δ(1, 8))` where `a ∈ dom(Σ_2.L)`. This is a concrete link-to-link reference — `a₂`'s from-endset targets the link entity at `a`. ✓

*L0 for `a₂`.* `subspace_I(a₂) = 2 = s_L`. The from-endset span `(a, δ(1, 8))` references `a` with `subspace_I(a) = 2 = s_L` — a same-subspace reference from `s_L` to `s_L`, permitted by L4. ✓

*L4 for `a₂`.* The span `(a, δ(1, 8))` has `a ∈ T` and satisfies T12 (verified above). No constraint prevents the span from referencing a link-subspace address. ✓

*Step 3: adding the arity-4 faceted link `a₃`.* Construct an arity-4 link `a₃` to exercise L3, L6, and L8 in the higher-arity regime. Define `a₃ = 1.0.1.0.1.0.2.4` — the next sibling in the link subspace after `a₂` (sibling advance from `a₂ = 1.0.1.0.1.0.2.3` via `inc(·, 0)` to `1.0.1.0.1.0.2.4`, unconditionally T4-preserving).

Suppose `a₃` connects an annotated passage (`c₁`) to a discussion (`c₂`), under a ghost type (`g`), with a fourth endset recording a supporting reference back to the original meta-link (`a₂`). Define the four endsets:

- Slot 1 (from): `F₃ = {(c₁, δ(1, 8))}` — the annotated content
- Slot 2 (to): `G₃ = {(c₂, δ(1, 8))}` — the discussion content
- Slot 3 (type): `Θ₃ = {(g, δ(1, 8))}` — the ghost type, by L3's slot-3 convention
- Slot 4 (supporting reference): `R₃ = {(a₂, δ(1, 8))}` — a fourth endset whose semantic role is determined by the type at `g`, outside this ASN's scope

The final state is `Σ_3` with `Σ_3.L = Σ_2.L ∪ {a₃ ↦ (F₃, G₃, Θ₃, R₃)}`, `Σ_3.C = Σ_2.C`, `Σ_3.M = Σ_2.M`.

*L3 (NEndsetStructure) at arity 4.* `|Σ_3.L(a₃)| = 4 ≥ 3`, and each `eᵢ ∈ Endset` since each is a singleton set of T12-well-formed spans. Slot 3 is the type endset (Θ₃ = `{(g, δ(1, 8))}` ≠ ∅) by L3's slot-3 convention, which fixes the role of position 3 uniformly for every arity `N ≥ 3` and requires non-emptiness there. ✓

*L6 (SlotDistinction) at arity 4.* `Σ_3.L(a₃) = (F₃, G₃, Θ₃, R₃)` is a 4-tuple of endsets with positional accessors `Σ_3.L(a₃).eᵢ` well-defined for `i ∈ {1, 2, 3, 4}`. The four entries have pairwise-distinct start addresses across slots (`c₁`, `c₂`, `g`, `a₂` are four distinct tumblers), so the transposition `π = (1 2)` yields `(G₃, F₃, Θ₃, R₃) ≠ (F₃, G₃, Θ₃, R₃)` (slot 1 differs) and `π = (1 4)` yields `(R₃, G₃, Θ₃, F₃) ≠ (F₃, G₃, Θ₃, R₃)` (slot 1 differs) — slot positions are addressable distinctly at arity 4. ✓

*L8 (TypeByAddress) at arity 4.* `Σ_3.L(a₃).type = Σ_3.L(a₃).e₃ = Θ₃ = {(g, δ(1, 8))}`. For the existing arity-3 link `a` with `Σ_3.L(a).type = Σ_3.L(a).e₃ = Θ = {(g, δ(1, 8))}`, coverage-based matching gives `same_type(a, a₃) ⟺ coverage(Σ_3.L(a).e₃) = coverage(Σ_3.L(a₃).e₃)`. Both endsets are `{(g, δ(1, 8))}` (a unit-depth span at `g`), so by PrefixSpanCoverage each has coverage `{t ∈ T : g ≼ t}` — the two coverage sets are identical. The arity-3 and arity-4 links share a type without any need to inspect content at `g`. ✓

*Step 4: adding `a₄` with a distinct ghost type — exercising L8 discrimination.* We add a link `a₄` whose type endset targets a *different* ghost type `g'`, and verify `same_type(a, a₄) = false`.

Define `g' = 1.0.1.0.1.0.3.2` — a sibling of `g = 1.0.1.0.1.0.3.1` in subspace `s_X`. By construction: `subspace_I(g') = 3 = s_X` (so `g' ∉ dom(Σ_3.C) ∪ dom(Σ_3.L)` by L0, since `s_X ∉ {s_C, s_L}` — the ghost is structurally outside the stored entities, by the same subspace-separation argument that placed `g` outside `Σ`'s stores); T4-validity holds (`zeros(g') = 3`, no adjacent zeros at the appended `[0, 3, 2]` tail since the inherited last component of `d = 1.0.1.0.1` is `1 > 0`, and the trailing component `2 > 0`); `#g' = 8`. The sibling step `g → g'` is `inc(g, 0)` (`k = 0`, unconditionally T4-preserving by TA5a), so `g'` is a structurally well-formed ghost address in `s_X`.

Define `a₄ = 1.0.1.0.1.0.2.5` — the next sibling in the link subspace after `a₃`, by `inc(a₃, 0)`. Define the endsets:

- From-endset: `F₄ = {(c₁, δ(1, 8))}`
- To-endset: `G₄ = {(c₂, δ(1, 8))}`
- Type-endset: `Θ₄ = {(g', δ(1, 8))}` — targeting `g'`, *not* `g`

The final state is `Σ_4` with `Σ_4.L = Σ_3.L ∪ {a₄ ↦ (F₄, G₄, Θ₄)}`, `Σ_4.C = Σ_3.C`, `Σ_4.M = Σ_3.M`.

*L8 (TypeByAddress) at `Σ_4` — discrimination.* `Σ_4.L(a).type = Θ = {(g, δ(1, 8))}` and `Σ_4.L(a₄).type = Θ₄ = {(g', δ(1, 8))}`. By PrefixSpanCoverage applied to each (using `#g = #g' = 8`, so `δ(1, 8)` is the unit-depth displacement at the action point at the tail):

- `coverage({(g, δ(1, 8))}) = {t ∈ T : g ≼ t}`
- `coverage({(g', δ(1, 8))}) = {t ∈ T : g' ≼ t}`

These two prefix sets are disjoint. Suppose `t` extends both: `g ≼ t` and `g' ≼ t`. Then `t` agrees with `g` and with `g'` at all positions `1..8`, so `t_8 = g_8 = 1` and simultaneously `t_8 = g'_8 = 2` — contradiction. Hence `coverage(Θ) ∩ coverage(Θ₄) = ∅`, and a fortiori `coverage(Θ) ≠ coverage(Θ₄)`. By L8's defining biconditional, `same_type(a, a₄) ⟺ coverage(Σ_4.L(a).type) = coverage(Σ_4.L(a₄).type)`, and the right-hand side is false; therefore `same_type(a, a₄) = false`. ✓

*Step 5: adding `a₅` with a multi-span type endset — exercising L5 order-irrelevance and extensional equality.* We add a link `a₅` whose type endset contains two spans, and verify L5's substantive content: span order within an endset is irrelevant, and endset equality is extensional set equality over `Span`.

Define `a₅ = 1.0.1.0.1.0.2.6` — the next sibling in the link subspace after `a₄`, by `inc(a₄, 0)`. Recall `g = 1.0.1.0.1.0.3.1` and its sibling `g' = 1.0.1.0.1.0.3.2` (Step 4), both ghosts in subspace `s_X` and outside `dom(Σ_4.C) ∪ dom(Σ_4.L)`; note `g ⊕ δ(1, 8) = g'`, so `g' = shift(g, 1)`. Define the two-span type endset

`Θ_split = {(g, δ(1, 8)), (g', δ(1, 8))}`

together with `F₅ = {(c₁, δ(1, 8))}`, `G₅ = {(c₂, δ(1, 8))}`, `Θ₅ = Θ_split`. Both type spans are T12-well-formed: `δ(1, 8) > 0` with action point `8 = #g = #g'`. The final state is `Σ_5` with `Σ_5.L = Σ_4.L ∪ {a₅ ↦ (F₅, G₅, Θ_split)}`, `Σ_5.C = Σ_4.C`, `Σ_5.M = Σ_4.M`.

*L5 (EndsetSetSemantics) at `Σ_5` — order-irrelevance across a 2-span endset.* `Θ_split` has two distinct members, `(g, δ(1, 8))` and `(g', δ(1, 8))` (distinct since `g ≠ g'`). By L5's biconditional, `Θ_split = {(g', δ(1, 8)), (g, δ(1, 8))}` — the reordered presentation has the same membership set, so the two are equal as endsets; the textual order in which the spans are written carries no semantic weight, and the model exposes no accessor that could distinguish the two presentations. Extensional equality: an endset `e` satisfies `e = Θ_split` iff `e` has exactly the members `{(g, δ(1, 8)), (g', δ(1, 8))}` — for instance the three-term presentation `{(g, δ(1, 8)), (g', δ(1, 8)), (g, δ(1, 8))}` denotes the same set `Θ_split` (membership is idempotent), while any endset that omits a member or adds a distinct span is unequal. This is the L5 content a singleton endset cannot exhibit. ✓

*L3 (NEndsetStructure) at `Σ_5`.* `|Σ_5.L(a₅)| = 3 ≥ 3` and slot 3 `Θ_split ≠ ∅` (two members); each endset is a finite set of T12-well-formed spans. The non-emptiness requirement on slot 3 is met by a multi-span endset exactly as by a singleton. ✓

*Step 6: adding `a₆` with a single-span type endset of identical coverage — exercising L8 coverage-vs-decomposition.* We add `a₆` whose type endset is a single span covering exactly the addresses that `Θ_split` covers as two spans, and verify `same_type(a₅, a₆) = true` without any `Θ` span-set equality.

Define `a₆ = 1.0.1.0.1.0.2.7` — the next sibling after `a₅`. Define the single-span type endset

`Θ_single = {(g, δ(2, 8))}`

where `δ(2, 8) = [0, 0, 0, 0, 0, 0, 0, 2]` is the width-2 displacement at the tail (`Pos`, action point `8 ≤ #g = 8`), so `(g, δ(2, 8))` is T12-well-formed. With `F₆ = {(c₁, δ(1, 8))}`, `G₆ = {(c₂, δ(1, 8))}`, `Θ₆ = Θ_single`, the final state is `Σ_6` with `Σ_6.L = Σ_5.L ∪ {a₆ ↦ (F₆, G₆, Θ_single)}`, `Σ_6.C = Σ_5.C`, `Σ_6.M = Σ_5.M`.

*Coverage equality of `Θ_split` and `Θ_single`.* By TumblerAdd at action point `8 = #g`: `g ⊕ δ(1, 8) = g'` (last component `1 → 2`) and `g ⊕ δ(2, 8) = 1.0.1.0.1.0.3.3` (last component `1 → 3`); write `h = 1.0.1.0.1.0.3.3`, and note `g' ⊕ δ(1, 8) = h` likewise (last component `2 → 3`). Then, directly from the Coverage definition:

- `coverage(Θ_single) = {t : g ≤ t < g ⊕ δ(2, 8)} = {t : g ≤ t < h}` (one span).
- `coverage(Θ_split) = {t : g ≤ t < g'} ∪ {t : g' ≤ t < h}` (two spans).

The two half-open intervals `[g, g')` and `[g', h)` are adjacent at the shared boundary `g'` (and `g < g' < h` under T1), so their union is exactly `[g, h) = {t : g ≤ t < h}`: any `t` with `g ≤ t < h` satisfies either `t < g'` (first interval) or `g' ≤ t` (second), and no `t` outside `[g, h)` is captured. Hence `coverage(Θ_split) = coverage(Θ_single)`, even though `Θ_split ≠ Θ_single` as endsets — the single span `(g, δ(2, 8))` is not a member of `Θ_split`, and neither member of `Θ_split` equals `(g, δ(2, 8))` (different widths), so by L5 the two are distinct span collections.

*L8 (TypeByAddress) at `Σ_6` — coverage match across distinct decompositions.* `Σ_6.L(a₅).type = Θ_split` and `Σ_6.L(a₆).type = Θ_single`. By the defining biconditional, `same_type(a₅, a₆) ⟺ coverage(Σ_6.L(a₅).type) = coverage(Σ_6.L(a₆).type)`, and the right-hand side holds by the coverage equality just established. Therefore `same_type(a₅, a₆) = true` — the two links share a type despite holding *different span decompositions* in their type endsets, where a span-set-identity criterion would over-discriminate (`Θ_split ≠ Θ_single`). ✓


## Properties Introduced

| Label | Type | Statement | Status |
|-------|------|-----------|--------|
| Σ.L | DEF | `Σ.L : T ⇀ Link` — the link store, mapping addresses to link values | introduced |
| L-fin | INV | LinkStoreFiniteness — `|dom(Σ.L)| < ∞` for each reachable state; parallels S8-fin (ASN-0036) | introduced |
| L0 | INV | SubspacePartition — link addresses occupy subspace `s_L`: `(A a ∈ dom(Σ.L) :: subspace_I(a) = s_L)`; see L0a, L0b | introduced |
| L0b | THM | LinkAddressValidity — every link address is T4-valid: `(A a ∈ dom(Σ.L) :: T4-valid(a))`; see L1c | introduced |
| L0a | DEF | ContentSubspaceScope — `dom(Σ.C)\|_{s_C} = {a ∈ dom(Σ.C) : subspace_I(a) = s_C}` is the `s_C`-resident slice of content; a state is *`s_C`-resident* iff `(A b ∈ dom(Σ.C) :: subspace_I(b) = s_C)` | introduced |
| L1 | INV | LinkElementLevel — every link address is an element-level tumbler: `(A a ∈ dom(Σ.L) :: zeros(a) = 3)` | introduced |
| L1a | INV | LinkScopedAllocation — every link address is allocated under the creating document's tumbler prefix | introduced |
| L1b | INV | LinkElementFieldDepth — every link address has element field depth ≥ 2: `(A a ∈ dom(Σ.L) :: #E(a) ≥ 2)` | introduced |
| L1d | LEMMA | SubspaceDisjointness (local) — (a) T4-valid element-level tumblers in distinct subspaces are distinct (T7 in `subspace_I` notation); (b) `dom(Σ.L) ∩ dom(Σ.C)\|_{s_C} = ∅` | introduced |
| L1c | AXIOM | LinkAllocatorConformance — link allocation conforms to T10a (AllocatorDiscipline, ASN-0034); every link address is the T4-valid terminus of a T10a-conforming chain seeded at its document-level prefix (full statement and postconditions in body) | introduced |
| CPP | LEMMA | ChainPrefixPreservation (local) — along a T10a-conforming chain of T4-valid tumblers with `p ≤ #t₀`, if every sibling-advance step acts on an input of length `> p`, the terminus agrees with `t₀` on positions `1..p` | introduced |
| FSP | LEMMA | FreshSiblingConformance (local) — appending one fresh sibling link `a` (h1 freshness, h2 producibility, h3 shape) carrying any L3-conforming payload, with `Σ'.C = Σ.C` and `Σ'.M = Σ.M`, preserves every state-local L- and S-invariant, satisfies the `Σ → Σ'` transition invariants L12/L12a, and yields `Σ' ⊒ Σ` (StateExtension) | introduced |
| FSE | LEMMA | FreshSiblingExistence (local) — for a conforming link `a ∈ dom(Σ.L)` under L-fin, there exists `i ≥ 1` with `a' = incⁱ(a, 0) ∉ dom(Σ.L)` satisfying `home(a') = home(a)`, `subspace_I(a') = s_L`, `zeros(a') = 3`, `#E(a') = #E(a)`, T4-valid and L1c-producible | introduced |
| L2 | LEMMA | OwnershipEndsetIndependence — `home(a)` depends only on `a`, not on the link's endsets | introduced |
| L3 | INV | NEndsetStructure — every link has at least three endsets, with slot 3 a non-empty type endset: `\|Σ.L(a)\| ≥ 3 ∧ Σ.L(a).e₃ ≠ ∅`; arity 3 `(F, G, Θ)` is the standard triple, higher arity admitted | introduced |
| L4 | META | EndsetGenerality — the model imposes no constraint on endset spans beyond T12 well-formedness (definitional from L3): no single-document, content-only, or existence restriction | introduced |
| L5 | INV | EndsetSetSemantics — an endset is an unordered set; only span membership matters | introduced |
| L6 | INV | SlotDistinction — endsets within a link are addressable by positional accessor `Σ.L(a).eᵢ`; link equality is component-wise tuple equality | introduced |
| L7 | META | DirectionalFlexibility — L0–L14 and L-fin impose no constraint on directional significance of from/to slots | introduced |
| L8 | DEF | TypeByAddress — type matching is by address coverage: `same_type(a₁, a₂) ⟺ coverage(Σ.L(a₁).type) = coverage(Σ.L(a₂).type)`; `.type` is slot 3, well-defined by L3 | introduced |
| L9 | LEMMA | TypeGhostPermission — any conforming, `s_C`-resident (L0a) state with `dom(Σ.M) ≠ ∅` can be extended, for every arity `N ≥ 3`, with a link of arity `N` whose type endset references addresses outside `dom(Σ.C) ∪ dom(Σ.L)`; arity-3 witness `(∅, ∅, {(g, δ(1, #g))})` extends to higher arities by padding empty endsets at slots `4..N` | introduced |
| PrefixSpanCoverage | LEMMA | For any tumbler `x` with `#x ≥ 1`, the unit-depth span has `coverage({(x, δ(1, #x))}) = {t ∈ T : x ≼ t}`; derived from OrdinalShift (`x ⊕ δ(1, #x) = shift(x, 1)`), T12, T1 cases (i)/(ii), and T5 (ASN-0034) | introduced |
| L10 | LEMMA | TypeHierarchyByContainment — `coverage({(p, δ(1, #p))}) = subtypes(p)` by PrefixSpanCoverage | introduced |
| L11a | LEMMA | LinkUniqueness — distinct T10a allocation events yield distinct link addresses; see GlobalUniqueness (ASN-0034) | introduced |
| L11b | LEMMA | NonInjectivity — every conforming, `s_C`-resident (L0a) state with a link can be extended to a non-injective conforming state | introduced |
| L12 | INV | LinkImmutability — `(A Σ, Σ' : a ∈ dom(Σ.L) : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a))` for every state transition | introduced |
| L12a | LEMMA | LinkStoreMonotonicity — `dom(Σ.L) ⊆ dom(Σ'.L)` for every state transition | introduced |
| L12b | LEMMA | HomeDocumentPersistence — `{home(a) : a ∈ dom(Σ.L)} ⊆ dom(Σ'.M)` for every state transition; joint consequence of L1a (at `Σ'`) and L12a | introduced |
| L13 | LEMMA | ReflexiveAddressing — link addresses are valid endset span targets; canonical span coverage by PrefixSpanCoverage | introduced |
| L14 | INV | DualPrimitive — stored entities partition into content (`dom(Σ.C)`) and links (`dom(Σ.L)`) with no third category | introduced |
| L14a | INV | NonTranscludability (under `s_C`-resident content) — `(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∉ dom(Σ.L))` | introduced |
| coverage(e) | DEF | the union of address sets denoted by the spans in endset e | introduced |
| home(a) | DEF | document-level prefix extracted from a link address via T4 field parsing — the document under whose prefix the link resides | introduced |
| Endset | DEF | `𝒫_fin(Span)` — a finite set of well-formed spans | introduced |
| Link | DEF | `{(e₁, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`; standard triple `(F, G, Θ)` by convention, slot 3 is the type endset | introduced |


## Open Questions

- Should a content-side invariant fix a global content-subspace constant, so that content-side disjointness extends from the `s_C`-resident slice to all of `dom(Σ.C)`?
- What invariants must hold between the link store and the content store when the same I-address appears in multiple arrangements via transclusion?
- What well-formedness constraints, if any, govern compound link structures where links reference other links through endsets?
- Under what conditions should two endsets with different span decompositions but identical coverage be treated as equivalent for query purposes?
- What constraints govern the allocation ordering of link addresses relative to content addresses within the same document?
- What must a conforming type address hierarchy satisfy beyond tumbler prefix containment?
- Must the link store maintain consistency with the arrangements `Σ.M`, or are the two components independently mutable?
- If links are permitted V-positions in the arrangement layer, what must S3 (ReferentialIntegrity) guarantee so that non-transcludability (L14a) is preserved?
