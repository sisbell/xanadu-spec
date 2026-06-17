> **ASN-0115 · RETRIEVEV — Content Delivery by Spec-Set** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0115-retrievev-operation-content-delivery-by-spec-set.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0115: RETRIEVEV — Content Delivery by Spec-Set

*2026-06-04*

## The problem

Of all the operations in the protocol, Nelson singles out exactly one as
concerned with the delivery of actual content: "Of the 17 current commands in
XU.87.1, only one command (RETRIEVEV) is concerned with delivery of the actual
content fragments" (4/61), and that command "returns the material (text and
links) determined by `<spec set>`" (4/67). Every other span-taking command
returns *descriptions* of where content lives — document identities, extents,
link identities. RETRIEVEV alone crosses the line from naming to delivering.

We are asked a precise question. The system is handed a *spec-set* — an ordered
series of spans naming positions in one or more documents — and asked to deliver
the material those spans determine. What comes back? What relationship must the
returned material bear to the spec-set that named it, and to the arrangements
that bind those spans to content? What does delivering a whole spec-set *together*
disclose — about content shared by transclusion, and about spans that cross from
the text subspace into the link subspace — that delivering one span in isolation
would conceal? And what invariants govern the material the operation may return?

## The substrate we build on

**Standing precondition (reachability).** Throughout this ASN, every state `Σ`
ranges over states *reachable from the initial state `Σ₀` under the sequential
transition order* (ASN-0047, SequentialTransitionAxiom).

We take the strand model as given. The *content store* `Σ.C : T ⇀ Val`
(ASN-0036) binds content addresses to values; it is append-only and immutable —
once `a ∈ dom(Σ.C)`, `a` persists and `Σ.C(a)` never changes (ASN-0036, S0
ContentImmutability; S1 StoreMonotonicity). The *link store* `Σ.L : T ⇀ Link`
(ASN-0043, ASN-0093) is likewise permanent (ASN-0043, L12 LinkImmutability). The
*arrangement* of a document `d` is a partial function `Σ.M(d) : T ⇀ T`
(ASN-0036) from V-positions to I-addresses; it is a genuine function (S2
ArrangementFunctionality) and, unlike the two stores, it is the one component
that may lose entries through editing (ASN-0047, P3 ArrangementMutabilityOnly).

A V-position carries its subspace in its first component, `subspace(v) = v₁`
(ASN-0036), with the fixed identifiers `s_C = 1` for content and `s_L = 2` for
links (ASN-0047, SubspaceConventionAxiom). Generalized referential integrity
fixes where each kind of position resolves: for `v ∈ dom(Σ.M(d))`,
`subspace(v) = s_C ⟹ Σ.M(d)(v) ∈ dom(Σ.C)` and
`subspace(v) = s_L ⟹ Σ.M(d)(v) ∈ dom(Σ.L)` (ASN-0047, S3★), and every active
V-position lies in one of these two subspaces (S3★-aux). Content and link stores
are disjoint (ASN-0047, L14 StoreDisjointness).

A span `σ = (s, ℓ)` is well-formed when `Pos(ℓ)` and `actionPoint(ℓ) ≤ #s`
(ASN-0034, T12); its denotation is the half-open tumbler interval
`⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` (ASN-0053). We take spans to be level-uniform
(`#s = #ℓ`) so start, width, and reach share a length and any two endpoints are
T1-comparable at equal depth (ASN-0053, S6). Every content address has a
well-defined *origin* — the document-level prefix that allocated it,
`origin(a) = N(a).0.U(a).0.D(a)` — and origin distinguishes content created by
distinct documents while identifying transcluded content as one (ASN-0036, S4
OriginBasedIdentity; S7 StructuralAttribution).

## What a spec-set is, and what delivery is

A *V-spec* is a pair `ρ = (d, σ)` naming an **allocated** document `d` — a
tumbler with `zeros(d) = 2` (ASN-0045) that is present in the arrangement family,
`d ∈ dom(Σ.M)` — and a well-formed, level-uniform, **ordinal-level** span
`σ = (s, ℓ)` whose start `s` is a *well-formed V-position*: a zero-free tumbler of
depth at least 2 with positive components,
`zeros(s) = 0 ∧ #s ≥ 2 ∧ (A i : 1 ≤ i ≤ #s : sᵢ > 0)`. This is the shape
ASN-0036's S8a requires of every *bound* position; we impose it here directly as a
constraint on the span's start, whether or not that start is itself bound. Depth
compatibility is deliberately *not* a well-formedness condition: `m_S(d)` is
mutable — ASN-0047 re-pins a cleared subspace on its next insertion, so
`#s = m_S(d)` need not persist — so it is a *consulting-state* predicate
`depthcompat(ρ, Σ)`.
Ordinal-level means the width acts at the deepest component,
`actionPoint(ℓ) = #ℓ` (ASN-0082, OrdinalLevel). This is the deepest-action-point
discipline that keeps a span within a single subspace:

> **Confinement (lemma).** For an ordinal-level, level-uniform span `σ = (s, ℓ)`
> with `#s = #ℓ = m ≥ 2`, every `t ∈ ⟦σ⟧` extends the length-`(m − 1)` prefix
> `p = [s₁, …, s_{m−1}]` of `s`: `p ≼ t` — hence `#t ≥ m − 1` and `tⱼ = sⱼ` for
> `1 ≤ j < m`. In particular `t₁ = s₁`, so `⟦σ⟧` lies wholly in subspace `s₁` and
> cannot cross the subspace boundary (generalizes ASN-0058's C0a,
> PrefixConfinement).
>
> *Proof.* Ordinal-level width acts only at position `m` (`actionPoint(ℓ) = m`),
> so the length-`(m − 1)` prefix `p = [s₁, …, s_{m−1}]` satisfies `p ≼ s`, and the
> reach `reach(σ) = s ⊕ ℓ` copies that prefix unchanged below the action point
> (TumblerAdd, ASN-0034), giving `p ≼ reach(σ)`. For any `t ∈ ⟦σ⟧`,
> `s ≤ t < reach(σ)`, hence `s ≤ t ≤ reach(σ)`; T5 (ContiguousSubtrees, ASN-0034)
> then yields `p ≼ t`, which carries both the length bound `#t ≥ #p = m − 1`
> (Prefix, ASN-0034) and the agreement `tⱼ = sⱼ` for `1 ≤ j < m`. ∎

Without ordinal-level width, a merely level-uniform well-formed span may have
`actionPoint(ℓ) = 1` and straddle from the content subspace into the link
subspace (e.g. `s = [1,5]`, `ℓ = [2,0]`: `s ⊕ ℓ = [3,0]`, and `[2,3] ∈ ⟦σ⟧` has
`subspace = s_L`). A single boundary-crossing span is therefore outside this
ASN's scope; designating both subspaces together is achieved by *composing*
per-subspace ordinal spans into the spec-set, not by one straddling span. The
allocation precondition `d ∈ dom(Σ.M)` makes the named arrangement `Σ.M(d)`
well-defined. A *spec-set* is a finite **ordered** sequence
`R = ⟨ρ₁, …, ρₚ⟩` of V-specs, `p ≥ 0`. The ordering is part of the request:
Nelson's caution that "if you want to designate a separated series of items
exactly, including nothing else, you do this by a span-set, which is a series of
spans" (4/25) tells us a spec-set is a *sequence*, not a set — its order carries
meaning, and it designates content *exactly*.

For a V-spec `ρ = (d, σ)`, write `S = s₁` for the subspace its start designates.
Call `ρ` *depth-compatible at `Σ`* — `depthcompat(ρ, Σ)` — when the named subspace
is empty or the start sits at its current common depth:

> `depthcompat(ρ, Σ) ≡ V_S(d) = ∅ ∨ #s = m_S(d)`

(well-formed because the disjunction guards `m_S(d)`, which ASN-0047 defines only
while `V_S(d) ≠ ∅`). We define `ρ`'s *active positions* at state `Σ` by a case
split on this predicate:

> `act(ρ, Σ) = dom(Σ.M(d)) ∩ ⟦σ⟧`  when `depthcompat(ρ, Σ)`
> `act(ρ, Σ) = ∅`  otherwise

In the depth-compatible branch the active positions are the V-positions the span
names that the document's arrangement actually binds. In the override branch —
any consulting-state depth mismatch, `V_S(d) ≠ ∅ ∧ #s ≠ m_S(d)` — the active set
is forced empty, *overriding* the geometric `dom(Σ.M(d)) ∩ ⟦σ⟧`. Force-empty is
chosen over the geometric intersection because that intersection is discontinuous
in the start's depth: with `m_S(d) = 3` (so `V_S(d) = {[S, 1, k] : 1 ≤ k ≤ n_S}`
by D-SEQ★), a too-shallow depth-2 start `[S, 1]` has `V_S(d) ⊆ ⟦σ⟧` and would
vacuum the *entire* subspace, while the neighbouring depth-2 start `[S, 2]`
captures nothing — all-or-nothing on where the shallow start happened to fall. A
spec stale against a re-pinned subspace should deliver nothing rather than vacuum
its re-pinned content; so in the shallow case (`#s < m_S(d)`) the override forces
empty lest the intersection capture deeper content the citation never named. The
override changes nothing in the deep case `#s > m_S(d)`, where the geometric
intersection is independently empty: for any `v ∈ dom(Σ.M(d)) ∩ ⟦σ⟧`, Confinement
gives `p ≼ v` for the length-`(m − 1)` prefix `p` of `s` — in particular
`v₁ = s₁ = S` (so `v ∈ V_S(d)`) and `#v ≥ m − 1` — while S8-depth pins
`#v = m_S(d) < m = #s`. Two sub-cases close the contradiction. If
`m_S(d) < m − 1`, the depth pin contradicts Confinement's length bound
`#v ≥ m − 1` outright. If `m_S(d) = m − 1`, then `#v = m − 1 = #p`, and `p ≼ v`
at equal length forces `v = p` (Prefix, ASN-0034); but `p ≺ s` (a proper prefix,
`#p = m − 1 < m = #s`), so `v < s` by T1 case (ii), contradicting the lower bound
`v ≥ s` that `v ∈ ⟦σ⟧` supplies. In either branch `act(ρ, Σ)` is finite —
it is `∅`, or a subset of `dom(Σ.M(d))`, which is finite (ASN-0036, S8-fin) — and
totally ordered, being a subset of the totally
ordered carrier `T` (ASN-0034, T1).
Finiteness and total order together give it a unique ascending enumeration
`v₁ < v₂ < … < v_{k}` where `k = |act(ρ, Σ)|`.

A start may name a subspace other than `s_C` or `s_L` and still be well-formed —
the V-position shape constrains `#s ≥ 2` and positivity but leaves `s₁`
unconstrained. Such a start is harmless rather than special-cased: by S3★-aux
every active V-position lies in `s_C` or `s_L`, so `V_S(d) = ∅` for any
`S ∉ {s_C, s_L}` at every reachable state; `depthcompat` then holds by its first
disjunct, `act = dom(Σ.M(d)) ∩ ⟦σ⟧`, and Confinement places `⟦σ⟧` wholly in the
unused subspace `S`, disjoint from `dom(Σ.M(d))` — so `act = ∅` and the spec
delivers nothing.

Each active position is resolved through the arrangement to a single address
`a = Σ.M(d)(v)` (well-defined and single-valued by S2), and the *delivery item*
for `v` is determined by the position's subspace:

> `item(v, ρ, Σ) =`
> `  ⟨content, Σ.C(a)⟩`   if `subspace(v) = s_C`   (then `a ∈ dom(Σ.C)` by S3★)
> `  ⟨ref, a⟩`            if `subspace(v) = s_L`   (then `a ∈ dom(Σ.L)` by S3★)

We may write `item` without case in what follows; the two cases are
distinguished by tag. These two cases are *exhaustive* on active positions, so
`item` is total on `act`: every `v ∈ act(ρ, Σ) ⊆ dom(Σ.M(d))` is an active
V-position, and by S3★-aux (SubspaceExhaustiveness, ASN-0047) every active
V-position has `subspace(v) = s_C` or `subspace(v) = s_L` — no third subspace
arises. Hence `item` is well-defined on its stated domain. The *per-spec delivery* is the ascending-V sequence
`deliver₁(ρ, Σ) = ⟨item(v₁, ρ, Σ), …, item(v_k, ρ, Σ)⟩`, and the **delivery** of
the whole spec-set is the concatenation in spec-set order:

> `deliver(R, Σ) = deliver₁(ρ₁, Σ) ⌢ deliver₁(ρ₂, Σ) ⌢ … ⌢ deliver₁(ρₚ, Σ)`     (R0)

The empty-request boundary is settled by the definition: when `p = 0`
the concatenation in R0 has no factors, so `deliver(⟨⟩, Σ) = ⟨⟩` — the empty
spec-set is a valid request whose delivery succeeds and returns nothing.

RETRIEVEV is a *pure query*: `deliver(R, Σ)` is a function of state that modifies
no component of `Σ` and appears in no transition of the substrate's vocabulary
(cf. ASN-0086, Observe).

"Names `v`" alone does not determine a span: many ordinal spans contain `v`,
and some capture further bound positions besides. The following lemma packages
the canonical construction — the unit-width span rooted at a bound position is
a well-formed spec whose active set is exactly that one position.

> **UnitSpec (lemma).** Let `d ∈ dom(Σ.M)` and let `v ∈ dom(Σ.M(d))` be a bound
> position; write `S = subspace(v)` and `a = Σ.M(d)(v)`. Define the *unit spec
> at `v`*: `unit(d, v) = (d, (v, δ(1, #v)))`, with `δ` the ordinal displacement
> of ASN-0034. Then:
>
> (a) *Well-formedness.* `unit(d, v)` is a V-spec in the sense above: its named
> document satisfies both document conjuncts (`zeros(d) = 2`, `d ∈ dom(Σ.M)`),
> its start has the required V-position shape, and its span is well-formed
> (T12), level-uniform, and ordinal-level.
>
> (b) *Depth compatibility.* `depthcompat(unit(d, v), Σ)` holds.
>
> (c) *Singleton active set.* `act(unit(d, v), Σ) = {v}`.
>
> (d) *Unit delivery.* `deliver₁(unit(d, v), Σ)` is the one-item sequence
> `⟨item(v, unit(d, v), Σ)⟩` — `⟨⟨content, Σ.C(a)⟩⟩` when `S = s_C`,
> `⟨⟨ref, a⟩⟩` when `S = s_L`; no third case arises.
>
> *Proof.* (a) The start `v` inherits its shape from S8a (ASN-0036), which every
> bound position satisfies: `zeros(v) = 0`, `#v ≥ 2`, every component positive.
> The width `δ(1, #v) = [0, …, 0, 1]` of length `#v` (OrdinalDisplacement,
> ASN-0034) satisfies `Pos(δ(1, #v))` and `actionPoint(δ(1, #v)) = #v`, so the
> span meets T12 (`actionPoint(ℓ) = #v ≤ #v = #s`), is level-uniform
> (`#ℓ = #v = #s`), and is ordinal-level (`actionPoint(ℓ) = #ℓ`). The document
> conjuncts close from the hypothesis: `d ∈ dom(Σ.M)` directly, and
> `zeros(d) = 2` because `dom(Σ.M) = E_doc` (ASN-0047, M1) places `d` in
> `E_doc = {e ∈ Σ.E : Document(e)}`, and the Document predicate (ASN-0045)
> unfolds to `T4-valid(d) ∧ zeros(d) = 2`.
>
> (b) `v ∈ dom(Σ.M(d))` with `subspace(v) = S` puts `v ∈ V_S(d)`, so
> `V_S(d) ≠ ∅`, and S8-depth (ASN-0036) pins the common depth `m_S(d) = #v`.
> The start is `v` itself, so `#s = #v = m_S(d)` — `depthcompat` holds by its
> second disjunct.
>
> (c) By (b), `act` takes its geometric branch,
> `act(unit(d, v), Σ) = dom(Σ.M(d)) ∩ ⟦σ⟧` with `σ = (v, δ(1, #v))`, and by
> PrefixSpanCoverage (ASN-0043) the unit-width denotation is the prefix subtree,
> `⟦σ⟧ = {t ∈ T : v ≼ t}`. *Lower bound:* `v ≼ v` (Prefix is reflexive) and
> `v ∈ dom(Σ.M(d))` by hypothesis, so `v ∈ act`. *Upper bound:* let `t ∈ act`.
> Then `v ≼ t`, so `t` agrees with `v` on positions `1 … #v` (Prefix, ASN-0034)
> — in particular `subspace(t) = t₁ = v₁ = S` — and `t ∈ dom(Σ.M(d))`, so
> `t ∈ V_S(d)`; S8-depth then pins `#t = m_S(d) = #v`. A prefix at equal length
> is equality: `v ≼ t` with `#t = #v` is agreement at every position of a shared
> length, so `t = v` (Prefix; T3, ASN-0034). Hence `act(unit(d, v), Σ) = {v}`.
>
> (d) By (c) the ascending enumeration of the active set is `⟨v⟩`, so
> `deliver₁(unit(d, v), Σ) = ⟨item(v, unit(d, v), Σ)⟩`, and the case split on
> `S` is the `item` definition — exhaustive because S3★-aux places every bound
> position's subspace in `{s_C, s_L}`. ∎

## Delivery returns material, not location

The first thing to settle is *what kind of thing* comes back. The contrast
Nelson draws is sharp: FINDDOCSCONTAINING "returns a list of all documents
containing any of the material … regardless of where the native copies are
located" (4/63) — locations; RETRIEVEDOCVSPAN "returns a span determining the
origin and extent" (4/68) — an address; only RETRIEVEV "returns the material"
(4/67). The span-set is the *name* of what is wanted; RETRIEVEV is what turns
that name into delivered material.

In our model this is exactly the form of `item`: for a content position the
delivered item carries `Σ.C(a)` — the value, not the address `a`; for a link
position it carries a reference to the link entity. The operation resolves the
arrangement and dereferences the content store; it does not return the V-to-I
mapping, nor the I-addresses, as its payload.

> **R1 (MaterialDelivery).** For every active content position, the delivered
> item carries the bound content value `Σ.C(Σ.M(d)(v))`, not a description of
> where that value is stored.

This is the property that distinguishes RETRIEVEV from every other span-taking
command, and any alternative implementation of "deliver the content of a
spec-set" must satisfy it. Gregory's two-phase realization — `specset2ispanset`
(resolve each V-spec through the document's arrangement to I-addresses) followed
by `ispanset2vstuffset` (fetch the bytes from the granfilade by I-address) —
is one mechanization of R0/R1; an alternative back end with a different index
structure would still owe the same two-phase semantics. We note that the link
path (`vspanset2sporglset`, which annotates I-addresses with source-document
provenance for link operations) is *not* on the content-delivery path: content
delivery needs no such annotation, because it delivers the value itself.

## Faithfulness, and where the invariant stops

What relationship must the delivered value bear to the content the span names? It
must be the content itself — no character altered, fabricated, or dropped.

> **R2 (Faithfulness).** Every delivered content item carries exactly the value
> bound, in the content store, to the address the arrangement assigns its
> position: for every active `v` with `subspace(v) = s_C`,
> `item(v, ρ, Σ) = ⟨content, Σ.C(Σ.M(d)(v))⟩`. No other value may be substituted.

The justification is structural. Resolution is single-valued because `Σ.M(d)` is a
function (S2), so the resolved address `a = Σ.M(d)(v)` is determinate; S3★ places
`a` in the content store, so `Σ.C(a)` is defined; and the `item` definition *sets*
the delivered content value to exactly `Σ.C(a)`. No other value may be
substituted, because the delivered value simply *is* the store's value at the
resolved address — that is the whole of R2. Permanence *across* states — that the
byte at an I-address never changes after creation, so the same resolution yields
the same value at a later state — is a distinct guarantee that R2 does not invoke;
it is carried by content immutability (S0).

We must be equally precise about where this invariant *stops*. Faithfulness is a
property of *what the operation is defined to return* — it is a semantic
guarantee about resolution and dereference, not a guarantee about a transmission
channel. Nelson disclaims the latter explicitly: storage and "attempts to
deliver such material, are at User's risk" (5/18), with "no guarantee as to the
correctness or authenticity" (5/18) of the channel. So R2 governs the
denotation of `deliver` and asserts nothing about the medium — a *frame limit*,
not a further claim.

## Exactness and arrangement-relativity

Must the delivered material correspond span-for-span to the spec-set — nothing
extra, nothing requested-and-present silently omitted? Yes, and the two halves
are visible directly in R0.

> **R3 (SpecSetExactness).** The delivery contains an item for *exactly* the
> active positions `act(ρⱼ, Σ)` of each spec, and no others: every delivered item
> arises from some `v ∈ act(ρⱼ, Σ)` (nothing extra), and every `v ∈ act(ρⱼ, Σ)`
> contributes an item (nothing active omitted). For a spec depth-compatible at `Σ`
> this reads as span-for-span exactness, `act(ρⱼ, Σ) = ⟦σⱼ⟧ ∩ dom(Σ.M(dⱼ))` —
> every position the span names and the arrangement binds, and no other; for a spec
> depth-incompatible at `Σ`, `act(ρⱼ, Σ) = ∅`, so that spec contributes nothing.

The upper bound holds because `act(ρ, Σ) ⊆ ⟦σ⟧` in either branch of the `act`
definition: the half-open interval is the exact extent the span designates, and
"there is no choice as to what lies between; this is implicit in the choice of
first and last point" (4/25). The lower bound holds because the delivery realizes
*all* of `act` (R0), and `act` is by definition `⟦σ⟧ ∩ dom(Σ.M(d))` for a
depth-compatible spec — so every named-and-bound position contributes — and `∅`
for a depth-incompatible one, where nothing is active to omit. We stress
that the span width `ℓ` is a tumbler boundary, not a count — "a tumbler-span …
does not designate the number of bytes contained" (Nelson, 4/25; ASN-0034) — so
the delivered quantity for a spec equals `|act(ρ, Σ)|`, the number of *active*
positions; when that count attains the width's deepest component is
characterised by the nominal-extent corollary of §"Partial delivery". Where a
requested boundary falls between stored positions,
an implementation must clip to the interval exactly (Gregory's `whereoncrum`
classification and `context2vtext` boundary clip realize this); the clip changes
no abstract content, it merely realizes the half-open interval precisely.

Against *which* arrangement is the resolution performed? Against the one named.

> **R4 (ArrangementRelativity).** Each V-spec `(dⱼ, σⱼ)` is resolved through the
> arrangement `Σ.M(dⱼ)` of the document it names — and through no other. The
> delivered material reflects exactly what the named arrangement binds those
> spans to.

This addresses the apparent dilemma between "the content each span *currently*
designates" and "content as it stood at some version." RETRIEVEV resolves it in
favor of *current*: it consults the named document's arrangement as it stands at
`Σ`. What naming `dⱼ` settles is *which* arrangement to consult — the version is
encoded in the document tumbler ("the Document field of the tumbler may be
continually subdivided, with new subfields … indicating daughter documents and
versions," 4/29), distinct versions are distinct document tumblers with distinct
arrangements (ASN-0036, S7d), and there is no privileged "basic" version the
system could silently substitute ("there is thus no 'basic' version of a document
set apart from other versions," 2/19). Naming `dⱼ` does *not* freeze that
arrangement, however: resolution is against the *current* `Σ.M(dⱼ)`, which is
mutable (K.μ⁻/K.μ⁺/K.μ~, ASN-0047; P3).

## Order and boundaries

In what order does the material arrive, and what governs the boundary between one
position's contribution and the next?

> **R5 (OrderFidelity).** Across V-specs, delivery follows spec-set sequence
> order: the items of `ρᵢ` wholly precede the items of `ρⱼ` whenever `i < j`,
> irrespective of the relative V-magnitudes of the two specs. Within a single
> V-spec, items are delivered in ascending T1 order of their V-positions. Each
> item's extent is fixed by its position; the boundary between consecutive items
> is implicit in the spec-set structure and the span endpoints, with nothing
> interpolated between them.

The cross-spec half is the substantive commitment: a later spec naming
numerically smaller V-positions is *not* re-sorted ahead of an earlier spec
naming larger ones. Concatenation in R0 is by sequence index `j`, not by
V-magnitude. Gregory's implementation confirms exactly this stratification —
within one V-spec the contexts are V-sorted (`incontextlistnd`), but across
V-specs the per-spec results are appended in spec-set order with no global
re-sort. An alternative implementation that globally V-sorted a multi-spec
request would deliver a *different* object and violate R5.

## Partial delivery: the gap is legal, not an error

What if part of the spec-set names content that no longer exists in the named
arrangement, or was never established there? The architecture answers: deliver
what can be delivered, signal the gap by absence, and never fail the whole.

> **R6 (SilentGapFiltering).** A named position the consulted arrangement does not
> make active — one outside `act(ρⱼ, Σ)` — contributes nothing to the delivery and
> causes no failure; delivery succeeds and returns the items for exactly the active
> positions `act(ρⱼ, Σ)`, the rest represented by their absence. When `ρⱼ` is
> depth-compatible at `Σ`, `act(ρⱼ, Σ) = dom(Σ.M(dⱼ)) ∩ ⟦σⱼ⟧`, so the filtered
> positions are precisely the geometrically unbound ones,
> `v ∈ ⟦σⱼ⟧ \ dom(Σ.M(dⱼ))`; when `ρⱼ` is depth-incompatible at `Σ`,
> `act(ρⱼ, Σ) = ∅` and the whole span is filtered, still without failure. Moreover,
> for a depth-compatible `ρⱼ`, restricted to the depth-`#s`, subspace-`S` slice of
> `⟦σⱼ⟧` — the only named positions the arrangement can bind (the slice is pinned
> at the span's own depth `#s` because `m_S(d)` is undefined in the `V_S(d) = ∅`
> branch, ASN-0047, while `#s` is not) — the unbound portion
> never falls as an interior hole within the subspace's contiguous active range;
> and whenever that slice meets the active range, the unbound portion is exactly a
> *terminal overrun* past the bound frontier. The no-interior-hole guarantee is a
> claim about the bindable slice, not about every named tumbler in the interval.

This is forced by the model. For a depth-compatible `ρ` the active set is the
intersection `dom(Σ.M(d)) ∩ ⟦σ⟧`, so a named position the arrangement does not
bind is simply not enumerated; for a depth-incompatible `ρ` the active set is `∅`
outright (the `act` override), so every named position — bound or not — is dropped.
Either way a non-active position contributes nothing and no failure arises.

The substrate sharpens *where* such a gap can fall, and we take the cases of the
`act` definition in turn. Fix a V-spec `(d, σ)` rooted in subspace `S = s₁`.
*Depth-incompatible at `Σ`* (`V_S(d) ≠ ∅ ∧ #s ≠ m_S(d)`): the override gives
`act = ∅`, so the active range is empty — there is no interior range for a hole to
fall in, and every named position is, vacuously, a terminal overrun of the empty
active range. *Depth-compatible at `Σ`*: either `V_S(d) = ∅` or `#s = m_S(d)`. If
`V_S(d) = ∅`, then `⟦σ⟧` lies wholly in subspace `S` (Confinement) while `d` binds
no subspace-`S` position, so `act = dom(Σ.M(d)) ∩ ⟦σ⟧ = ∅`; again every named
position is an unbound terminal overrun of the empty active range, with no interior
range for a hole. Otherwise `V_S(d) ≠ ∅` and `#s = m_S(d)` — the span is rooted at
exactly the subspace's common depth `m_S`, which is the case the remainder of this
argument analyses. By
D-SEQ★ (ASN-0047) — the content-subspace instance being D-SEQ (ASN-0036) — the
active positions of `d` in subspace `S` are the contiguous prefix
`V_S(d) = {[S, 1, …, 1, k] : 1 ≤ k ≤ n_S}`, varying only in the last component.

We confine the gap analysis to the *bindable slice* of `⟦σ⟧`: its depth-`m_S`,
subspace-`S` members. These are the only named positions `dom(Σ.M(d))` can
contain, since every active position has depth `m_S` (S8-depth) and subspace `S`.
Named positions of `⟦σ⟧` deeper than `m_S` are necessarily unbound: S8-depth
fixes the depth of every active subspace-`S` position at exactly `m_S`, so a
named position of depth `> m_S` is absent from `dom(Σ.M(d))` and dropped from
`act`.

The span is ordinal-level of depth `m_S ≥ 2`, so by Confinement every
`t ∈ ⟦σ⟧` agrees with `s` on positions `1 … m_S − 1`; hence every depth-`m_S`
member of `⟦σ⟧` shares `s`'s first `m_S − 1` components and varies only in the
last coordinate `k`.
To name those components we appeal to `act ≠ ∅`, the substantive case: pick any
`v ∈ act ⊆ V_S(d)`. By D-SEQ★ `v = [S, 1, …, 1, k_v]`, and `v ∈ ⟦σ⟧` at depth
`m_S` forces `v` to agree with `s` on positions `1 … m_S − 1`, so `s`'s first
`m_S − 1` components are exactly `[S, 1, …, 1]` — `act ≠ ∅` forces a canonical
start `s = [S, 1, …, 1, s_{m_S}]`. Hence the depth-`m_S` slice of `⟦σ⟧` is exactly
`{[S, 1, …, 1, k] : s_{m_S} ≤ k < s_{m_S} + ℓ_{m_S}}` — with `ℓ_{m_S}` the width's
deepest component — the only free coordinate being `k`. (If instead `act = ∅`
while `V_S(d) ≠ ∅`, the depth-`m_S` slice of `⟦σ⟧` is disjoint from `V_S(d)`, so
it meets no bound position and punches no interior hole within the active range;
the terminal-overrun half of R6 is then vacuously satisfied.)

A depth-`m_S` named position `[S, 1, …, 1, k]` is bound iff `k ≤ n_S` — exactly
the D-SEQ★ frontier. Therefore the unbound members of the bindable slice are
precisely those with `k > n_S`: a contiguous tail beyond the active frontier. An
*interior* gap — a depth-`m_S` named position `[S, 1, …, 1, k]` with `k ≤ n_S`
yet absent from the arrangement — is impossible, because D-SEQ★ makes every such
`k` bound. Within the bindable slice the gap is always an overrun past the
frontier, never a hole inside it; this is the precise sense in which R6's
"represented by its absence" lands, and it is what the §"Exactness" boundary-clip
remark realizes operationally. It is also forced by Nelson's design intent. A
span addresses a *range*, and "a span that contains nothing today may at a later
time contain a million documents" (4/25) — an empty or partly-empty range is an
anticipated, legal state, not a fault. The same architecture governs the only
place Nelson states an explicit rule about unsatisfiable request parts: "THE
QUANTITY OF LINKS NOT SATISFYING A REQUEST DOES NOT IN PRINCIPLE IMPEDE SEARCH ON
OTHERS" (4/60) — serve the satisfiable part, drop the rest, never fail the whole.
Gregory's resolution chain bears this out: an unresolvable V-range produces an
empty I-span set and a successful, empty contribution — indistinguishable from a
legitimately empty region. The gap is signalled *structurally* (the caller
compares what it asked for against what arrived), not by an error code.

*Corollary (nominal-extent attainment).* Call the width's deepest component
`ℓ_{#ℓ}` the span's *nominal extent*. It is the cardinality of the bindable
slice — the depth-`#s`, subspace-`S` members of `⟦σ⟧` (R6): by Confinement every
depth-`#s` member of `⟦σ⟧` shares `s`'s first `#s − 1` components, leaving only
the last to range over `s_{#s} ≤ k < s_{#s} + ℓ_{#ℓ}`. And `ℓ_{#ℓ} ≥ 1`: the
span is ordinal-level, so its action point is `#ℓ`, and ActionPoint's
postcondition `w_{actionPoint(w)} ≥ 1` (ASN-0034) applies at that position. The
delivered quantity attains the nominal extent — `|act(ρ, Σ)| = ℓ_{#ℓ}` — iff the
spec is depth-compatible at `Σ` and every member of the bindable slice is bound.
The three branches of the `act` analysis above cover both directions.
*Depth-incompatible:* the override gives `act = ∅`, so
`|act| = 0 < 1 ≤ ℓ_{#ℓ}` — both sides of the biconditional fail.
*Depth-compatible with `V_S(d) = ∅`:* the slice is non-empty (`ℓ_{#ℓ} ≥ 1`
members) and wholly unbound — a bound slice member would lie in `V_S(d)` — so
the right side fails, and `act = dom(Σ.M(d)) ∩ ⟦σ⟧ = ∅` fails the left.
*Depth-compatible with `V_S(d) ≠ ∅`* (so `#s = m_S(d)`): `act` lies inside the
bindable slice — every `v ∈ act` is bound and, by Confinement, in subspace `S`,
so `v ∈ V_S(d)` and S8-depth (ASN-0036) pins `#v = m_S(d) = #s` — and every
bound slice member lies in `dom(Σ.M(d)) ∩ ⟦σ⟧ = act`, so `act` is exactly the
bound members of the slice. The two finite cardinalities therefore agree iff the
slice is bound throughout; by the frontier analysis, a slice member
`[S, 1, …, 1, k]` is bound iff `k ≤ n_S`, so — under the canonical start that a
non-empty `act` forces — attainment is equivalent to
`s_{#s} + ℓ_{#ℓ} − 1 ≤ n_S`.

Note the boundary R6 does *not* cover: R6 concerns the absence of *binding* for a
named position within an allocated document (`d ∈ dom(Σ.M)`), not the allocation
of the document itself.

*Worked instance.* Let document `d` have a content arrangement bound at exactly
four positions, `V_1(d) = {[1, k] : 1 ≤ k ≤ 4}` (so `n_1 = 4`), each resolving to
its own content address `Σ.M(d)([1, k]) ∈ dom(Σ.C)`. Build the single-spec request
`R = ⟨(d, σ)⟩` whose span starts at `s = [1, 2]` with ordinal width
`ℓ = δ(5, 2) = [0, 5]`, so `reach(σ) = s ⊕ ℓ = [1, 7]` — the span names `[1,2]` up
to but not including `[1,7]`. Its depth-2 slice is
`⟦σ⟧ ∩ {t : #t = 2} = {[1, 2], [1, 3], [1, 4], [1, 5], [1, 6]}`; the full
denotation `⟦σ⟧ = {t ∈ T : [1,2] ≤ t < [1,7]}` also contains deeper tumblers such
as `[1,2,1]` (a proper extension of `[1,2]`, hence `> [1,2]` by T1 case (ii), and
`< [1,7]` by T1 case (i) at position 2), but the arrangement here binds only
depth-2 positions, so only the slice meets `dom(Σ.M(d))`. The spec is
depth-compatible (`#s = 2 = m_1(d)`), so `act` takes its geometric branch;
intersecting with the arrangement,
`act((d, σ), Σ) = dom(Σ.M(d)) ∩ ⟦σ⟧ = {[1, 2], [1, 3], [1, 4]}`, so the delivery is
`deliver(R, Σ) = ⟨⟨content, Σ.C(Σ.M(d)([1,2]))⟩, ⟨content, Σ.C(Σ.M(d)([1,3]))⟩,
⟨content, Σ.C(Σ.M(d)([1,4]))⟩⟩`. Check the four claims against this result. R1:
each item carries the *value* `Σ.C(Σ.M(d)([1,k]))`, not the address. R3 upper
bound: every delivered item is named by `σ` (all three lie in `⟦σ⟧`), nothing
extra. R3 lower bound: every named-and-bound position contributes — `[1,2]`,
`[1,3]`, `[1,4]` are exactly `⟦σ⟧ ∩ dom(Σ.M(d))`, none omitted. R5: the three
items are in ascending T1 order `[1,2] < [1,3] < [1,4]`. R6: the named positions
`[1,5]` and `[1,6]` have no binding (`k = 5, 6 > n_1 = 4`), so they are filtered
silently — the request succeeds and returns the three bound items, with the two
unbound positions represented by their absence. The gap is a terminal overrun: it
is exactly the named tail past the frontier `n_1 = 4`, not a hole inside the bound
range `{[1,1], …, [1,4]}` (which the span happens not to name below `[1,2]` and
fully names from `[1,2]` to `[1,4]`). This is the partial-delivery boundary in
full: a multi-position span that reaches past the bound range, delivered up to
the frontier and clipped to the interval exactly.

## Repeatability

If the same spec-set is asked again, against unchanged arrangements, must the
delivered material be identical?

> **R7 (Repeatability).** Let `Σ`, `Σ'` be two states of one evolving docuverse
> with one a reachability descendant of the other along the sequential transition
> order, and let `R` be a spec-set at the *earlier* state of the pair — without
> loss of generality `Σ →* Σ'` (ASN-0047, SequentialTransitionAxiom), with
> `dⱼ ∈ dom(Σ.M)` for every `j`. Then each `dⱼ ∈ dom(Σ'.M)` as well, so the
> restrictions `Σ'.M(dⱼ)|⟦σⱼ⟧` and the delivery `deliver(R, Σ')` are
> well-defined; and if the consulted arrangement restrictions agree,
> `Σ.M(dⱼ)|⟦σⱼ⟧ = Σ'.M(dⱼ)|⟦σⱼ⟧` for every `j`, then
> `deliver(R, Σ) = deliver(R, Σ')`.

The well-definedness clause is discharged by M1 (ArrangementMonotonicity,
ASN-0047): at each atomic step of `Σ →* Σ'` it gives `dom(Σ.M) ⊆ dom(Σ'.M)`,
so `dⱼ ∈ dom(Σ'.M)` for every `j`.
`deliver` is a function of two things: the consulted arrangement restrictions,
and the stores the resolved values are drawn from. We first show the active sets
agree, `act(ρⱼ, Σ) = act(ρⱼ, Σ')`. If `Σ.M(dⱼ)|⟦σⱼ⟧` is non-empty, a shared bound
position `v ∈ ⟦σⱼ⟧ ∩ dom(M(dⱼ))` lies in subspace `S = s₁` (Confinement), so
`v ∈ V_S(dⱼ)` at both states and S8-depth pins `m_S(dⱼ) = #v` equally at each;
depth-compatibility then holds-or-fails identically, and where it holds `act` is
the equal restriction's (equal, non-empty) domain at both. Where it fails
identically at the two states, both take the override and
`act(ρⱼ, Σ) = ∅ = act(ρⱼ, Σ')`, so the active sets still agree despite the
non-empty restriction — the override discards it at both. If `Σ.M(dⱼ)|⟦σⱼ⟧` is
empty, then `⟦σⱼ⟧ ∩ dom(M(dⱼ)) = ∅` at both states, so `act(ρⱼ, Σ) = ∅ = act(ρⱼ, Σ')`
whichever branch each state takes (the depth-compatible branch yields the empty
intersection, the override branch yields `∅` directly). Either way the active sets
and the resolved addresses agree position-for-position. Fix any resolved
address `a`, the same at both states by that agreement. For a **link position**
the delivered item is `⟨ref, a⟩` — it carries the resolved *address*,
never the link value `Σ.L(a)` — so its stability is already settled: equal
resolved addresses give the identical reference item `⟨ref, a⟩` at both states,
with no appeal to any store invariant. For a **content position** the item carries
the value `Σ.C(a)`, and here value-persistence is the load-bearing fact. The
hypothesis gives `Σ →* Σ'` directly. Because the consulted restriction binds
`a` at both states, S3★ places it in the content store at each,
`a ∈ dom(Σ.C) ∩ dom(Σ'.C)`; over the intervening transitions `Σ →* Σ'`, content
immutability (S0) holds the stored entry fixed, giving `Σ.C(a) = Σ'.C(a)`. Hence
for every resolved address the delivered value or reference is the same at both
states, and the two deliveries are identical. The arrangement is the sole mutable
input (R4; P3) — `deliver` is a function of the consulted restriction and the
immutable stores — so keeping that restriction unchanged (R7's hypothesis)
*secures* repeatability, which a caller obtains by citing a version whose
arrangement it does not subsequently edit. This is a sufficiency, not a
biconditional: an unchanged restriction guarantees an identical delivery, but the
converse fails. A rebinding that lands on an equal-valued content address —
permitted by S4 (OriginBasedIdentity), which allows `a₁ ≠ a₂` with
`Σ.C(a₁) = Σ.C(a₂)` — leaves the delivery byte-identical while the consulted
restriction changed, so identical delivery does not entail an unchanged
restriction. The caller relies on the forward direction and never on that
coincidence.

## What co-delivery does with transclusion

Suppose two distinct positions in the request — in the same spec or different specs — resolve to the same content
address `a`. This is
transclusion: the same content, included by reference in two places, carrying one
permanent I-address wherever it appears (ASN-0036, S5 UnrestrictedSharing).

> **R8 (TransclusionCoResolution).** If two **distinct** active positions `v, v'`
> (within one spec or across specs) — distinct as positions, `(d, v) ≠ (d', v')` —
> resolve to the same address, `Σ.M(d)(v) = Σ.M(d')(v') = a`, then they share one
> subspace, and the co-delivery guarantee is content-only. In the **content
> sub-case** (`a ∈ dom(Σ.C)`) the two distinct positions are co-resolved through
> the one shared address `a`: (i) both items
> carry the identical value `Σ.C(a)` (R2); (ii) both resolve *through* `a` —
> identity-preserving co-resolution — so `origin(a)` of both is one and the same
> (S4, S7); and (iii) the operation performs no deduplication, so the shared
> content appears once per V-position. The sharing is a fact of *resolution*, not
> of the delivered output: each item carries the value `Σ.C(a)`, never the address
> `a` (R1), so the co-delivery is byte-indistinguishable from the delivery of two
> coincidentally-equal contents at distinct addresses (S4) and discloses nothing
> about the shared origin. The **link sub-case** is *vacuous*: two
> distinct active link positions can never share a link address. Genuine
> transclusion is therefore confined to content.

The box's two structural claims — that the positions share one subspace, and that
the link sub-case is vacuous — are established as follows.

*Why the two positions share a subspace.* By S3★ the shared address `a` lies in
`dom(Σ.C)` or in `dom(Σ.L)` but, by store disjointness (L14, ASN-0047), not both. To run
store membership *back* to subspace we need that each of `subspace(v)`,
`subspace(v')` is one of `s_C`, `s_L` to begin with — supplied by S3★-aux
(SubspaceExhaustiveness, ASN-0047) for the active positions `v, v'` — whereupon
the contrapositive of the off-store S3★ branch closes the step: were
`subspace(v) = s_L` while `a ∈ dom(Σ.C)`, S3★ would force `a ∈ dom(Σ.L)`,
contradicting L14; so `a ∈ dom(Σ.C)` fixes `subspace(v) = s_C`, and symmetrically
`a ∈ dom(Σ.L)` fixes `subspace(v) = s_L`. The same dispatch applied to `v'` yields
`subspace(v) = subspace(v')`. The content sub-case is the realizable one: S5
(UnrestrictedSharing) permits a content address to be bound at arbitrarily many
V-positions, within one document and across documents.

*Why the link sub-case is vacuous.* Two **distinct** active link positions can
never share a link address. CL-OWN (ASN-0047) forces `origin(Σ.M(d)(v)) = d` for
every link-subspace position, so a link address `a` can be bound only in the
arrangement of `origin(a) = home(a)`; two documents both binding `a` in their link
subspaces are forced equal, `d = d'`. Within that one document, CL-UNIQ (ASN-0047)
makes `Σ.M(d)` injective on the link subspace, so two positions both mapping to `a`
are forced equal, `v = v'`. Both are per-state invariants of every reachable state
(ASN-0047, ExtendedReachableStateInvariants). Genuine link transclusion therefore
does not occur, and the only multiplicity available for a link is a single bound
V-position named by two overlapping specs — which delivers the identical reference
`⟨ref, a⟩` (the `item` definition) twice with common provenance `home(a)`
(ASN-0043, L1a), and is
*not* transclusion (it is one position, not two). The substantive co-delivery
guarantee of R8 is thus confined to content.

Within content, identity is structural, not incidental. Content identity in
Xanadu is by creation, not by value: two independently created identical strings
get distinct addresses, while transcluded content shares one address (S4). So
delivering both positions by way of the same `a` is identity-preserving by
construction — in computing the delivery the operation never copies, it
dereferences the one address `a` twice, so both items are the one content
delivered twice, not two independent reproductions of it.

Nor does the operation merge the two items into one. This is forced abstractly —
two distinct V-positions are two distinct entries, and a delivery that dropped
one would violate R3 (it would silently omit a named, bound position). It is also
exactly Gregory's behavior: the consolidation step that would merge co-referent
spans is absent (the `consolidatespans` call is commented out), so identical
bytes are delivered once per V-position.

*Worked instance.* Let document `d` transclude one stretch of content twice: V-positions `u`
and `w` (with `u < w`, both in subspace `s_C`) both map to the same content
address `a`, i.e. `Σ.M(d)(u) = Σ.M(d)(w) = a`. Take the spec-set of the two unit
specs in reverse V-order, `R = ⟨unit(d, w), unit(d, u)⟩` — explicitly
`⟨(d, (w, δ(1, #w))), (d, (u, δ(1, #u)))⟩`. Both positions are bound, so UnitSpec
applies to each spec: `act(unit(d, w), Σ) = {w}` and `act(unit(d, u), Σ) = {u}`
(UnitSpec (c)), and each per-spec delivery is the single item
`⟨⟨content, Σ.C(a)⟩⟩` (UnitSpec (d), at `S = s_C`). Concatenating in spec-set
order (R0),
`deliver(R, Σ) = ⟨⟨content, Σ.C(a)⟩, ⟨content, Σ.C(a)⟩⟩`: two items, the
*same* value both times (R8.i), in the order the specs were given — `w` before
`u`, against V-magnitude (R5) — with neither dropped (R8.iii). The two appearances resolve
through the single address `a` — a fact of the resolution, not of the delivered
stream, which by R8 carries two byte-identical values that disclose nothing about
the sharing. Co-delivery adds nothing here that two separate single-span
deliveries would not.

## What co-delivery reveals: coherent multi-origin assembly

A single spec-set may gather spans whose content was created in different
documents. What must delivery guarantee about presenting that material?

> **R9 (CoherentMultiOriginAssembly).** A spec-set drawing on multiple origins is
> delivered as one ordered sequence (R5), assembled by resolving each spec
> against its own document's arrangement independently (R4). How much origin
> survives *into the delivered stream* is *kind-asymmetric*, tracking the payload
> asymmetry the `item` definition fixes: a **link** item carries the address `a`
> itself (the `item` definition), so its home `home(a)` is recoverable from the
> delivered output; a **content** item carries only the value `Σ.C(a)` (R1), so
> its origin `origin(a)` is *not*
> recoverable from the output — it is determinate only through the resolution
> mapping `v ↦ a`, an internal artifact of computing `deliver`.

Two obligations sit in this co-assembly, and they differ in how much they
constrain the output. The material must be *coherent* — one ordered stream the
caller reads as a single delivery, with fragments slotted in spec-set order
regardless of where they physically originate ("the virtual byte stream of a
document may include bytes from any other document," 4/10; non-native bytes have
"an ordinal position … just as if they were native," 4/11). And each fragment's
origin must stay *determinate* — co-assembly must not fuse distinct origins into
an anonymous blob. This second obligation is met automatically: `origin` and
`home` are functions of the resolved address, so no faithful resolution could
lose a fragment's home document. Because each spec is resolved against its own
arrangement (R4), cross-document spec-sets are resolved per document and then
concatenated — Gregory's
`specset2ispanset` loop calls the per-document lookup once per spec, reading each
document's arrangement in isolation, exactly the independence R9 requires.

*Worked instance.* Let two **distinct** documents `d₁ ≠ d₂` (distinct
document-level tumblers, `zeros(dⱼ) = 2`, with distinct allocation events,
ASN-0036 S7d) each create and bind their own content. Document `d₁` binds a
content position `v₁` (`subspace(v₁) = s_C`) to a content address `a₁ ∈ dom(Σ.C)`
that `d₁` itself allocated, so `origin(a₁) = d₁` (S7(b)); document `d₂` binds a
content position `v₂` (`subspace(v₂) = s_C`) to a content address `a₂ ∈ dom(Σ.C)`
that `d₂` allocated, so `origin(a₂) = d₂`. Because `d₁ ≠ d₂` were distinct
allocating documents, S7(c) (StructuralAttribution) gives `origin(a₁) ≠
origin(a₂)` — the two fragments carry distinct, determinate home documents. Build
the cross-document spec-set of unit specs `R = ⟨unit(d₁, v₁), unit(d₂, v₂)⟩` —
explicitly `⟨(d₁, (v₁, δ(1, #v₁))), (d₂, (v₂, δ(1, #v₂)))⟩`. Each `vⱼ` is bound
in its own document, so UnitSpec applies per spec. By R4 each spec is resolved
against its own arrangement in isolation: `unit(d₁, v₁)` through `Σ.M(d₁)` yields
`act(unit(d₁, v₁), Σ) = {v₁}` (UnitSpec (c)) and the single item
`⟨content, Σ.C(a₁)⟩` (UnitSpec (d)); `unit(d₂, v₂)` through `Σ.M(d₂)` yields
`act(unit(d₂, v₂), Σ) = {v₂}` and item `⟨content, Σ.C(a₂)⟩`. (a)
Coherent ordered assembly: by R0's concatenation in spec-set order,
`deliver(R, Σ) = ⟨⟨content, Σ.C(a₁)⟩, ⟨content, Σ.C(a₂)⟩⟩` — `d₁`'s item precedes
`d₂`'s item because spec 1 precedes spec 2 (R5), irrespective of the T1-magnitudes
of `a₁`, `a₂` or of `d₁`, `d₂`. (b) Determinate, resolution-recoverable
provenance: each delivered item was resolved through a definite address whose
home document is fixed by the resolution mapping — `origin(a₁) = d₁` and
`origin(a₂) = d₂` (S7) — and these are distinct, so the resolution attributes
each fragment to its own creating document. Both items here are content, so
neither origin travels in the delivered output — each item carries `Σ.C(aⱼ)`, not
`aⱼ` (R1) — and the attribution is recoverable through the resolution mapping,
not inline: the content side of R9's kind-asymmetry. The delivered stream is one
coherent sequence (a) whose fragments remain individually attributable, through
resolution, to their distinct creating documents (b) — exactly the dual
obligation R9 names.

## What co-delivery reveals: subspace crossing

A document's arrangement maps positions in two subspaces:
content (`s_C`) and links (`s_L`). A spec-set with specs in both subspaces
gathers positions of both kinds.

> **R10 (SubspaceCrossingObservability).** When an active position lies in the
> link subspace (`subspace(v) = s_L`), it resolves (by S3★) to a link address
> `a ∈ dom(Σ.L)`, and the delivered item is a *reference* to that link entity —
> an item distinguishable in kind from a content-value item. A spec-set spanning
> both subspaces therefore yields a heterogeneous delivery in which the subspace
> boundary is observable as a change of item kind. A span confined to the text
> subspace never exposes link-subspace material.

This is the abstract content of "delivery is not restricted to the text
subspace." The arrangement is a single map over both subspaces; resolution
follows it wherever the named positions lead. For a content position the item
carries a value; for a link position the item carries the link's address — *not*
the link's endset structure, which is the concern of operations that read a link
by address (out of scope here). The crossing is *observable* precisely because
the two item kinds differ: co-delivery of a both-subspace spec-set exhibits the
boundary that a single text-subspace span could never reveal. Gregory's
realization discriminates exactly this way — a resolved position whose stored
crum is content yields a text item, one whose crum is a link-organizer yields an
address item — so the wire stream carries content fragments and link-address
references intermixed, with the boundary visible in the tagging.

*Worked instance.* Let document `d` bind a content position
`v_C` (`subspace(v_C) = s_C`) to a content address `a_C ∈ dom(Σ.C)`, and a link
position `v_L` (`subspace(v_L) = s_L`) to a link address `a_L ∈ dom(Σ.L)`; the
two are disjoint stores (L14). Build a two-spec spec-set of unit specs, one per
subspace: `R = ⟨unit(d, v_C), unit(d, v_L)⟩` — explicitly
`⟨(d, (v_C, δ(1, #v_C))), (d, (v_L, δ(1, #v_L)))⟩`. Each unit span is
ordinal-level (UnitSpec (a)), so it stays within its own subspace (Confinement
lemma) and neither straddles; both positions are bound, so UnitSpec (c) gives
`act(unit(d, v_C), Σ) = {v_C}` and `act(unit(d, v_L), Σ) = {v_L}`. Resolving the
first spec gives `subspace(v_C) = s_C`, hence by S3★ `a_C ∈ dom(Σ.C)` and the
single item `⟨content, Σ.C(a_C)⟩` (UnitSpec (d)); resolving the second gives
`subspace(v_L) = s_L`, hence by S3★ `a_L ∈ dom(Σ.L)` and the single item
`⟨ref, a_L⟩` (UnitSpec (d)). Therefore, concatenating in spec-set order (R0),
`deliver(R, Σ) = ⟨⟨content, Σ.C(a_C)⟩, ⟨ref, a_L⟩⟩` — a heterogeneous stream
whose two items differ in tag (`content` vs `ref`), and the subspace boundary
between the two specs is observable precisely as that change of item kind. A
single text-subspace span could yield only `content`-tagged items and could never
expose the `ref` item; the crossing is visible only because the spec-set
designates both subspaces together.

## What governs the material: permanence of the source

Finally, what invariant governs the addresses the bytes are drawn from?

> **R11 (PermanentSourcing).** Delivery sources every content item from the
> immutable content store by I-address. Consequently a content address that has
> ever entered `dom(Σ.C)` remains deliverable for all time: if any arrangement —
> the document's own, a later version's, or a transcluding document's — binds
> some V-position to `a`, then a spec over that document resolves to `a` and
> delivers `Σ.C(a)`, even if the originally-creating document's *current*
> arrangement no longer references `a`.

Here is the decisive distinction between *deletion* and *removal of content*.
"Deletion" in Xanadu is contraction of an arrangement — the V-to-I binding leaves
`Σ.M(d)`; the bytes do not leave `Σ.C`, which is append-only and immutable
(S0, S1). The content becomes *orphaned* relative to the contracting document
(unreachable through *that* document's current arrangement) but is not absent
from the store. The weakest precondition for delivery to include an item
*sourced from* `a` — one whose resolving address, in the resolution mapping
`v ↦ Σ.M(d)(v)` that computes `deliver` (R9), is `a` — is therefore a *single*
live condition: (i) the consulted arrangement binds some *active* content
position to `a` — a `v ∈ act(ρ, Σ)` with `subspace(v) = s_C` and
`Σ.M(d)(v) = a`. The postcondition is pinned to the source address, not to
value-appearance: for the bare appearance of `Σ.C(a)` in the output, (i) is
sufficient but not necessary, since S4 permits a distinct `a' ≠ a` with
`Σ.C(a') = Σ.C(a)`, and an active position resolving to `a'` puts the same value
in the output while (i) is false — the delivered value does not identify its
source address (R1, R8). Stating (i) through `act` rather than bare namedness folds in
the depth condition the override makes operative: `v ∈ act` entails the spec is
depth-compatible at `Σ` (else `act = ∅`), that `v` is named (`act ⊆ ⟦σ⟧`), and
that `v` is bound (`act ⊆ dom(Σ.M(d))`). There is no independent store-membership
conjunct to add. The active position is a content position
(`subspace(v) = s_C`), so generalized referential integrity discharges store
membership directly — `Σ.M(d)(v) = a ⟹ a ∈ dom(Σ.C)` (S3★) — the instant (i)
holds; immutability (S0) then holds `Σ.C(a)` fixed for all time. A version
created before a deletion still binds the address, and so still delivers the
content — which is what makes identity-preserving restoration possible at all,
and what makes "any portion of any version (historical or alternative)" (2/19)
retrievable. Gregory's content fetch confirms the asymmetry: the granfilade
lookup is by I-address with no liveness check; whatever was committed at an
address is returned whenever an arrangement resolves to it.

*Worked instance.* Let content address `a` be created under document `d` and
bound there at V-position `v_d` (`subspace(v_d) = s_C`), so `Σ.M(d)(v_d) = a` and
`a ∈ dom(Σ.C)`. Fork a later version `d'` (a distinct document tumbler, ASN-0036
S7d) that still binds `a` at some position `v'` (`subspace(v') = s_C`,
`Σ.M(d')(v') = a`) — versions share the one Istream content pool, so this is the
same address. Now contract `d`'s arrangement by K.μ⁻ (ASN-0047), removing the
binding of `v_d`: the post-state `Σ'` has `v_d ∉ dom(Σ'.M(d))`, so `a` is
orphaned relative to `d`. Yet `a` never leaves the store — `dom(Σ.C) ⊆ dom(Σ'.C)`
and `Σ'.C(a) = Σ.C(a)` by S0/S1 — and the contraction touches only `Σ.M(d)`, so
`Σ'.M(d')(v') = a` still holds (ASN-0047, K.μ⁻ frame: `(A d'' : d'' ≠ d :
M'(d'') = M(d''))`). Take the single-unit-spec request `R = ⟨unit(d', v')⟩`,
whose span is the unit-width span rooted at `v'` — `σ' = (v', δ(1, #v'))`. Since
`v'` is bound in `Σ'.M(d')` with `subspace(v') = s_C`, UnitSpec applies at `Σ'`:
the spec is depth-compatible (UnitSpec (b), `#v' = m_{s_C}(d')` by S8-depth since
`v' ∈ V_{s_C}(d')`), its active set is exactly the singleton
`act(unit(d', v'), Σ') = {v'}` (UnitSpec (c)), the resolution is
`Σ'.M(d')(v') = a`, and
`deliver(R, Σ') = ⟨⟨content, Σ'.C(a)⟩⟩` (UnitSpec (d)) — the spec over the
*surviving* version
delivers `Σ.C(a)` even though `d`'s current arrangement no longer references it.
The wp's single live condition (i) holds at `d'` though it has been falsified at
`d`; deletion-as-contraction is local to the arrangement it edits, never to the
store.

## Synthesis

RETRIEVEV is, abstractly, *resolution followed by faithful dereference, in
order*. Given a spec-set, it delivers the material the spans determine (R1):
each named, bound V-position is resolved through its own document's arrangement
(R4) to an address, and the immutable value at that address is delivered
faithfully (R2), confined exactly to the named positions (R3), in spec-set order
across specs and ascending V-order within (R5). Named positions with no binding
are filtered silently and never fail the request (R6); against unchanged
arrangements the result is bit-for-bit repeatable (R7), because the arrangement
is the only mutable input and the stores never alter what they hold.

Delivering a whole spec-set together exceeds delivering its spans separately in
some respects but not all. For transclusion it does *not*: distinct content positions
sharing a resolved address deliver identical material with no deduplication, yet
co-delivery discloses nothing about the sharing (R8). Genuine transclusion is a
content phenomenon, since CL-OWN and
CL-UNIQ make distinct link positions sharing an address unreachable, so the link
sub-case is vacuous (R8). Where co-delivery does
exceed the sum of its parts is assembly and crossing: it assembles multi-origin
material into one coherent stream whose fragments stay attributable to their
origins through resolution (R9); and it makes the text/link subspace boundary
observable as a change in item kind (R10). Underneath all of it, the store is
permanent: orphaned-but-referenced content remains deliverable for all time
(R11). Each of R1–R11 is an obligation
any faithful realization must meet; the one implementation we have evidence for
meets them, with its two-phase resolve-then-fetch structure realizing R0 and its
absent consolidation step realizing the no-deduplication corollary of R3 and R8.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| R0 | `deliver(R, Σ)` = per-spec deliveries concatenated in spec-set order; `deliver₁(ρ,Σ)` = items of `act(ρ,Σ)` in ascending T1 order, where `act(ρ,Σ) = dom(Σ.M(d)) ∩ ⟦σ⟧` when `ρ` is depth-compatible at `Σ` (`V_S(d) = ∅ ∨ #s = m_S(d)`) and `∅` otherwise; `item` carries `Σ.C(a)` for content positions, the reference `a` for link positions | introduced |
| R1 | MaterialDelivery: a content item carries the bound value `Σ.C(Σ.M(d)(v))`, not a description of its location | introduced |
| R2 | Faithfulness: every content item equals `Σ.C(Σ.M(d)(v))` (from S2 + S3★ and the `item` definition); no value may be substituted. Frame limit: this governs the denotation of delivery, not any transmission channel | introduced |
| R3 | SpecSetExactness: items arise for exactly `act(ρⱼ, Σ)` — for a depth-compatible spec this is `⟦σⱼ⟧ ∩ dom(Σ.M(dⱼ))` (nothing outside the spans, nothing named-and-bound omitted), for a depth-incompatible spec it is `∅` | introduced |
| R4 | ArrangementRelativity: each V-spec is resolved through the current `Σ.M(dⱼ)` alone; naming `dⱼ` selects which version's arrangement to consult, and delivery reflects what that arrangement currently binds | introduced |
| R5 | OrderFidelity: spec-set sequence order across specs (no global V re-sort); ascending V-order within a spec; boundaries implicit in spans | introduced |
| R6 | SilentGapFiltering: a position outside `act(ρⱼ, Σ)` contributes nothing and causes no failure (gap signalled by absence); for a depth-compatible spec the unbound portion of the bindable slice — the depth-`#s`, subspace-`S` members of `⟦σⱼ⟧` — is never an interior hole in the active range, and is a terminal overrun past the bound frontier whenever the slice meets that range; a depth-incompatible spec has `act = ∅` | introduced |
| R7 | Repeatability: for `R` a spec-set at the earlier state of the pair (each `dⱼ ∈ dom(M)` lifted to the later state by M1, ASN-0047), equal consulted arrangement restrictions ⟹ identical delivery; the arrangement is the sole mutable input | introduced |
| R8 | TransclusionCoResolution: two distinct content positions sharing a resolved address deliver identical material via identity-preserving co-resolution through the one shared address, with no deduplication (one item per V-position); the sharing is internal to resolution and not disclosed by the output, which carries values not addresses (R1) and is byte-indistinguishable from coincidental value-equality (S4, R9); the link sub-case is vacuous (CL-OWN + CL-UNIQ forbid distinct link positions sharing an address), so genuine transclusion is confined to content | introduced |
| R9 | CoherentMultiOriginAssembly: multi-origin spec-sets deliver as one ordered stream (R5), resolved per document (R4); output-recoverable provenance is kind-asymmetric — a link item carries `a`, so `home(a)` is recoverable from the delivered output (the `item` definition; L1a, HomeOriginCoincidence), while a content item carries only `Σ.C(a)`, so `origin(a)` (S7) is determinate only through the resolution mapping, not the output | introduced |
| R10 | SubspaceCrossingObservability: link-subspace positions resolve (S3★) to link addresses and deliver as references — kind-distinct from content items — making the subspace crossing observable | introduced |
| R11 | PermanentSourcing: content is sourced from the immutable store by I-address; an address ever in `dom(Σ.C)` remains deliverable whenever any arrangement binds a position to it, including orphaned-but-referenced content | introduced |

## Open Questions

What must content delivery guarantee about inline provenance — must a delivered fragment carry, within the delivered material itself, enough to ascertain its origin, or may origin be recoverable only by a separate query?

Under what conditions, if any, may a content-delivery operation be permitted to fail outright rather than deliver partially?

Were generalized referential integrity (S3★) relaxed — so that an arrangement could bind a position to an address present in neither store — what must delivery guarantee for the resulting dangling reference?

What faithfulness, if any, may be required of the delivery channel itself, given that the storage-layer faithfulness invariant does not extend to transmission?

What must delivery guarantee when a single span's denotation straddles the subspace boundary, so that one contiguous named range yields both content items and link-reference items?
