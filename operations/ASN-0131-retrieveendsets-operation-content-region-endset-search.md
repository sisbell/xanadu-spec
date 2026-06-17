> **ASN-0131 · RETRIEVEENDSETS — Surfacing Anchoring Over a Content Region** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md), [ASN-0127 · Content-Region Link Query](../foundation/ASN-0127-content-region-link-query.md)  
> [Condensed statements →](ASN-0131-retrieveendsets-operation-content-region-endset-search.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0131: RETRIEVEENDSETS — Surfacing Anchoring Over a Content Region

*2026-06-13*

We have, by the time we reach this note, two stores and an arrangement family in the
system state `Σ = (Σ.C, Σ.L, Σ.E, Σ.M, Σ.R)` (ASN-0047). The content store `Σ.C : T ⇀ Val`
(ASN-0036) maps I-addresses to immutable content. The arrangement family `Σ.M`, with
`Σ.M(d) : T ⇀ T` (ASN-0036), maps each V-position of a document `d` to the I-address it
currently occupies. The link store `Σ.L : T ⇀ Link` (ASN-0043) maps link addresses to
link values, where a link value is a sequence of at least three endsets (L3, ASN-0043)
and an endset is a finite set of well-formed spans (`Endset = 𝒫_fin(Span)`, ASN-0043). A
link does not anchor to a position; it anchors to *content* — its endsets reference
I-addresses, and `coverage(e)` (ASN-0098, ASN-0043) is the set of addresses an endset
denotes. This is why links survive editing: the strap is tied to the content's identity,
not to any document's current ordering of it.

We already possess, in the foundation, a query that asks of a region "what is reachable
here?" and answers with *link identities*: `findlinks_V(W, d, Σ)` (F-V, ASN-0127). The
present note studies its sibling. We ask the very same question of a region — *what
touches here?* — but we demand a different answer. We do not want the names of the links.
We want the *anchoring itself*: the endsets, the spans where links attach. We want to be
told **that** this content is bound, and **how** it is bound — at which spans, on which
side of the link — without being told **which** links bind it. Why one would want such
an answer; what it can and cannot certify; and what it must guarantee as the document
beneath it is edited and as links are withdrawn — these are the questions of this note.

We name the operation `RETRIEVEENDSETS`, after the FEBE operation that realises it, and
write `RE(W, d, Σ)` for its result.

## The region, and what it resolves to

A region is not a set of I-addresses handed to us directly. One asks the question *of a
place in a document* — "of this passage, what anchoring touches it?" — and the system
must first discover what content presently occupies that place. So a region is a pair
`(W, d)` with `d ∈ dom(Σ.M)` a document and `W ⊆ T` a set of V-positions, which we require
to lie in the **content subspace**: `(∀ v ∈ W : subspace(v) = s_C)`, where `subspace(v) =
v₁` is the V-position's subspace identifier and `s_C` the content one (ASN-0047). These are
the text positions of `d` — typically the V-positions of a span in `d`'s text. The
restriction is a caller obligation, not a check the operation performs. Its **content-image** is

> `I = image(W, d, Σ) = {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d))}`     (F-IMG, ASN-0127),

the I-addresses that the region's V-positions currently map to through `d`'s
arrangement. Because every `v ∈ W` carries `subspace(v) = s_C`, generalized referential
integrity places the image in content: `I ⊆ dom(Σ.C)` (S3★, ASN-0047). The region is
resolved to content through the present arrangement, and everything downstream is phrased
in I-addresses, where links live.

## When does an endset touch the region?

Fix the region `(W, d)` and its image `I`. We must say, of a single endset `e`, what it
is for `e` to *touch* the region. The endset denotes `coverage(e) ⊆ T` (ASN-0098). The
region denotes `I ⊆ T`. We are looking for the weakest relation between these two sets
that faithfully captures "this anchoring reaches into the region."

Consider the candidates. *Containment* — `coverage(e) ⊆ I` — would require the entire
endset to lie inside the region. But this is plainly too strong, and we can see why by
asking what it would discard. An endset may cover a whole chapter and intersect our
one-line region in a single place; under containment we would not surface it, yet its
anchoring manifestly reaches our line. Worse, an endset straddling the region boundary —
covering content both inside and outside `W` — would be silently dropped, when it is
exactly such an anchoring we most want to see. Containment answers a different question
("which anchorings live *wholly within* here?"), not ours.

The relation we want is *overlap*: the endset touches the region exactly when it covers
at least one address the region also covers — the disjunction Nelson phrases as matching
"all or any part of" the requested set, where a single shared address is already real
contact and the endset need neither lie inside the region nor contain it. We define, for
the fixed region,

> `touch_W(e)  ≡  coverage(e) ∩ image(W, d, Σ) ≠ ∅`

— the subscript naming the region's V-position set `W`.

First, the relation is **existential within an endset**. An endset is a finite *set* of
spans, possibly discontiguous (ASN-0043). `touch_W(e)` asks that *some* address in
`coverage(e)` lies in `I` — not that every span does. The other spans of a touching
endset legitimately point elsewhere; they do not disqualify it, and they are not
clipped away from it.

Second, the relation is **per-endset, not per-link**. We judge each endset on its own
coverage against the region. A link carries several endsets — by convention a from-endset
`e₁`, a to-endset `e₂`, a type-endset `e₃`, and possibly more (L3). RETRIEVEENDSETS asks
its one region of *each* endset independently. A link's from-endset may touch while its
to-endset points to a destination far outside `W`; then the from-endset is surfaced and
the to-endset is not. There is no four-set request here differentiating slot from slot
(that is the richer FINDLINKSFROMTOTHREE); there is one region, tested against every
endset, and the endsets that touch are the ones surfaced.

> **Cross-subspace unit-span disjointness (RE-NCD).** Let `s` be a T4-valid element-level
> address (`zeros(s) = 3`) whose element-field subspace identifier is non-content,
> `E(s)₁ ≠ s_C`. Then the unit-depth span `(s, δ(1, #s))` covers no content:
> `coverage({(s, δ(1, #s))}) ∩ dom(Σ.C) = ∅`.

By PrefixSpanCoverage (ASN-0043) the unit-depth span covers exactly `{t : s ≼ t}`, so it
suffices that no content address extends `s`. Take any `c ∈ dom(Σ.C)`: by S7b (ASN-0036) it is
T4-valid with `zeros(c) = 3`, and by content allocation `E(c)₁ = s_C` (L0, ASN-0093). Were
`s ≼ c`, then `c` agrees with `s` on positions `1..#s` (Prefix, ASN-0034). The agreement carries
all three of `s`'s separator zeros onto `c`; as `c` has only three zeros in all (`zeros(c) = 3`),
these *are* `c`'s separators, so `s` and `c` share a third-zero position and hence the
subspace-identifier position one past it — forcing `E(c)₁ = E(s)₁ ≠ s_C`. But content allocation
gave `E(c)₁ = s_C` — a contradiction. So no content address extends `s`, giving the disjointness.

## The unit of the answer: anchoring without names

Now we can state what RETRIEVEENDSETS returns. We must first settle which links it ranges
over. A link, once created, is permanent and immutable in the store (L12, ASN-0043) — but
the system admits *retraction*, recorded not by deleting the link but by emitting a
withdrawal link that marks the target nullified (ASN-0086). A withdrawn link's anchoring
should not be reported as live — a design decision that fixes the operation as a report over
the *active* population, not the full permanent store. So we range over the links that are
present and not withdrawn — the **addressable** links:

> `addressable(Σ) = dom(Σ.L) ∖ nullified(Σ)`     (over ASN-0086's `nullified`).

ASN-0086's `nullified(Σ)` — the set of withdrawn addresses — is a function of the link store
`Σ.L` alone, so `addressable` depends on `Σ.L` alone.

The operation surfaces, for each addressable link and each of its endsets that touches
the region, that endset, tagged by the slot it occupies:

> `RE(W, d, Σ)  =  { (i, e) : (∃ a : a ∈ addressable(Σ) : 1 ≤ i ≤ |Σ.L(a)| ∧ Σ.L(a).eᵢ = e ∧ touch_W(e)) }`.

The answer is a set of `(role, endset)` pairs. Each pair names the slot `i` — from, to,
type, or higher — and the endset value `e` that occupies it in some touching link.

The answer just defined rests on two premises. It is **finite unconditionally** (RE-FIN): drawn
from the finite supply of slot-endset pairs the store affords — `dom(Σ.L)` is finite (L-fin,
ASN-0093) and each link carries finitely many endsets (L3, ASN-0043) — so only finitely many
`(i, e)` pairs can ever appear, whatever the region. And it is **computable** under one further
hypothesis: that region membership `v ∈ W` is decidable, which holds whenever `W` is *finitely
presented* — given, say, as finitely many spans, `v ∈ W` reduces to span-membership tests (T12,
T2, ASN-0034). Granted that hypothesis, the remaining selection is effective over the finite
store.

The definition **withholds the link address `a`**. The existential `(∃ a : …)` consumes the link and discards it; what
escapes into the answer is `(i, e)`, the anchoring structure, never the identity. Two distinct links sharing an identical
endset value in the same slot — permitted, since the link store is non-injective (L11b,
ASN-0043) — collapse to a single pair `(i, e)`. The answer therefore does not let one
count the links, recover their identities, or — and this is the deeper limit — *pair* a
surfaced from-endset with the to-endset of the same link. A from-span and a to-span may
both appear in the answer, drawn from one link, yet the answer carries nothing that binds
them as one link's two ends. The anchoring is laid bare; the connection is not made
followable.

`RE` reads the
arrangement `Σ.M(d)` (to form the image) and the link store `Σ.L` (for endsets, and,
through `nullified`, for addressability). It never consults the content *values* `Σ.C`,
the entity set `Σ.E`, or the provenance relation `Σ.R`. And it is a pure query: it reads
state and changes none — `Σ' = Σ`. Whatever anchoring it reports, it reports as a fact
about the state it found, leaving that state untouched.

Three degenerate inputs are worth reading straight off the definition.

- *Empty image.* When `W ∩ dom(Σ.M(d)) = ∅` — the region selects no arranged position, as
  for a freshly registered document whose arrangement is still empty (`dom(Σ.M(d)) = ∅`) —
  the image is `I = ∅`, so `touch_W(e) ≡ coverage(e) ∩ ∅ ≠ ∅` is false for *every* endset,
  and `RE(W, d, Σ) = ∅`. A region resolving to no content touches no anchoring.

- *No addressable links.* When `addressable(Σ) = ∅` — the store holds no links, or every
  link present is nullified — the existential `(∃ a ∈ addressable(Σ) : …)` in RE-DEF has no
  witness, so `RE(W, d, Σ) = ∅` whatever the region. Anchoring can only be surfaced from a
  live link.

- *Empty endset slot.* An endset may itself be empty: ASN-0043 admits `∅` in the from- and
  to-slots, and only the type-slot is required non-empty (L3, ASN-0043). Since
  `coverage(∅) = ∅`, `touch_W(∅)` is false against any region, so an empty slot is *never*
  surfaced. The operation reports anchoring only where some span genuinely covers a region
  address.

## Fresh emissions and the addressable population

Write `Θ` for ASN-0086's designated retraction type, and `L_Θ^Σ` for its *retraction slice* —
the arity-3 type-`Θ` links at state `Σ` (ASN-0086).
We adopt, as a **standing assumption** scoped to the addressability results, ASN-0086's
*relational-layer discipline commitment*: every store transition that grows the retraction slice
`L_Θ` — every `→`-step with `L_Θ^Σ ⊊ L_Θ^{Σ'}` — is a `Nullify`. Under that commitment
ASN-0086's *UnitDepthRetractionDiscipline* holds at every layer-reachable state: *every* `L_Θ`
to-set is a single unit-depth span `{(t, δ(1, #t))}` at a link target `t`
(UnitDepthRetractionDiscipline, ASN-0086). And `nullified(Σ)` is an existential over that slice
`L_Θ^Σ ⊆ Σ.L` alone (ASN-0086).

`dom(Σ.L)` is a tumbler-prefix antichain — distinct stored links never nest
(R0a/FlatLinkDomain, ASN-0086).

A `K.λ` step emits a *fresh* link — allocation gives `ℓ_new ∉ dom(Σ.L)`, so `ℓ_new` enters
`dom(Σ'.L)` — and whether that fresh output is *addressable* in its post-state
(`ℓ_new ∉ nullified(Σ')`) turns on whether some *nullifying* to-set covers it — where
"nullifying" means the to-set of a tuple in the retraction slice `L_Θ^{Σ'}`. The standing commitment's
unit-depth to-set then settles the question: every `L_Θ^{Σ'}` to-set is unit-depth at some link
`t ∈ dom(Σ'.L)`, covering `{u : t ≼ u}`, and `dom(Σ'.L)` is a prefix-antichain (above), so
any `t` distinct from `ℓ_new` is prefix-incomparable to it (`t ⋠ ℓ_new`) and cannot cover it. The only nullifying to-set that
could cover `ℓ_new` is therefore one whose *target* is `ℓ_new`. No *pre-existing* `L_Θ` tuple
bears such a to-set: every nullifying tuple already in `Σ.L` targets an address in `dom(Σ.L)` —
by Nullify's P-tgt (ASN-0086) its target is either an `A_rel = dom(Σ.L)` address or, for a
self-emission, the emitter's own address, again in `dom(Σ.L)` — whereas `ℓ_new ∉ dom(Σ.L)` by
freshness, so none targets `ℓ_new`. The only tuple in `Σ'.L` that can target `ℓ_new` through a
nullifying to-set is thus the freshly-emitted `ℓ_new` itself — present only if `ℓ_new` is itself
a retraction — and it covers `ℓ_new` exactly when `ℓ_new` retracts its own emitter address.
Hence the reusable fact — **fresh-output addressability (RE-ADDR)**: a fresh `K.λ` output that
does not retract its own emitter address is addressable in its post-state. In particular every
non-retraction emission (`K ≁ Θ`) is addressable: not being type `Θ`, it is no retraction, so a
fortiori it does not retract its own emitter.

## Extent: the surfaced endset, whole and unclipped

A returned endset must be the link's *actual* anchoring, not an approximation of it, and
not a fragment of it trimmed to the region. Two invariants of different strength must be
kept apart, because the operation rests squarely on one and merely adopts the other.

The load-bearing invariant is **no clipping (RE-CLIP)**: whatever span the answer
reports, it reports at the full extent recorded in the link, never truncated to the
region boundary. The reasoning is Nelson's, and it is decisive: clipping would
*misrepresent the anchoring*. An endset whose span straddles the region boundary would be
reported as a shorter span than the link actually attaches to — a falsehood about the
link's grip. One searches *from* a region in order to learn the true shape of what
attaches there, including how far it reaches beyond; to clip would be to answer that
falsehood. So no reported span is ever shortened to fit the query, and this holds under
*every* reading of the operation.

The reading we *adopt* makes a stronger, separable commitment — **whole-endset surfacing
(RE-WHOLE)**: we return `e = Σ.L(a).eᵢ` in full, *all* of its spans, not merely the slice
of `e` whose spans fall inside `W`. A discontiguous endset is then surfaced with the spans
pointing outside the region intact, since those are precisely the parts that say where
else this anchoring lives.

But this is a *convention*, not a forced consequence. The alternative *touching-spans*
reading would return, at each selected slot, only those spans of `e` that individually meet
the region image — whole spans, never trimmed, so RE-CLIP holds of it equally — discarding
the spans that point elsewhere. The two readings share the same selection (the touching
slot-link pairs) and differ only in the return value at a selected slot, so RE-OVL,
soundness, and completeness stand under both. But the touching-spans return value is
*region-dependent* — the spans it keeps vary with which region is asked — and a
region-dependent return value cannot distribute over unions: union-distributivity (RE-UDIST)
holds for the adopted whole-endset value and would fail for the touching-spans reading. We
therefore mark RE-DEF's *return-value clause*, and with it RE-WHOLE, **provisional**, leaving
the choice to Open Question 1.

## Soundness and completeness: the answer is exactly the touching anchoring

The definition is a biconditional, and its two directions are the operation's correctness
contract — each an immediate read of RE-DEF, not a theorem requiring argument.

**Soundness** is the forward direction: if `(i, e) ∈ RE(W, d, Σ)`, then `e` is a genuine
slot-`i` endset of an addressable link and `touch_W(e)` holds — the existential of RE-DEF
witnesses a real `a` with `Σ.L(a).eᵢ = e`. The operation fabricates no anchoring: nothing in
the answer fails to reach the region, a reported overlap is a true overlap, and a reader who
receives `(1, e)` may rely that some live link really attaches its from-end at the spans of
`e` and that those spans really reach the region.

**Completeness** is the converse: for every addressable link `a` and every slot `i` with
`touch_W(Σ.L(a).eᵢ)`, the pair `(i, Σ.L(a).eᵢ)` is in `RE(W, d, Σ)`. Every endset that
touches the region — by direct anchoring or through transcluded content
— appears; none is silently omitted.

Together they fix the result as *exactly* the touching set — neither more nor less.

## A worked instance

It is worth grounding these claims in a state small enough to compute by hand, yet
arranged to exercise every distinctive postcondition at once. Let `d ∈ dom(Σ.M)` arrange
four pieces of text content at consecutive V-positions of its text subspace (`s_C = 1`):

> `Σ.M(d) = { [1,1] ↦ a₁,  [1,2] ↦ a₂,  [1,3] ↦ a₃,  [1,4] ↦ a₄ }`,

with `a₁ < a₂ < a₃ < a₄` four content I-addresses in `dom(Σ.C)`, consecutive siblings
under `d`'s content sub-allocator (so `a₂ = shift(a₁, 1)`, `a₃ = shift(a₂, 1)`,
`a₄ = shift(a₃, 1)`). Two links inhabit the store. The first, at address `ℓ₁`, is the
standard triple `L₁ = (e₁, e₂, e₃)`:

- from-endset `e₁ = {(a₂, δ(2, #a₂)),  (a₄, δ(1, #a₄))}` — a *discontiguous* endset of two
  spans, one touching the region we are about to draw and one pointing wholly outside it.
  Its first span is width-2 ordinal, reaching from `a₂` across its successor so that
  `{a₂, a₃} ⊆ coverage(e₁)` (the upper bound is exclusive, so this span stops short of
  `a₄`); it *straddles the region boundary*, covering `a₂`, which the region will hold, and
  `a₃`, which it will not. Its second span is unit-depth at `a₄`, with
  `coverage({(a₄, δ(1, #a₄))}) = {t : a₄ ≼ t}` (PrefixSpanCoverage, ASN-0043), reaching
  only `a₄` and its descendants — content the region does not reach;
- to-endset `e₂ = {(a₁, δ(1, #a₁))}` — with `coverage(e₂) = {t : a₁ ≼ t}`
  (PrefixSpanCoverage, ASN-0043), containing none of `a₂`, `a₃`, `a₄` (each is a sibling of
  `a₁`, not a descendant);
- type-endset `e₃ = {(θ, δ(1, #θ))}` — `θ` a classifying address in a *type* subspace,
  T4-valid and element-level (`zeros(θ) = 3`) with subspace identifier `E(θ)₁ = s_type ≠
  s_C`, non-empty as L3 demands. Its single span is unit-depth at `θ` with `E(θ)₁ ≠ s_C`, so
  RE-NCD applies directly: `coverage(e₃) = {t : θ ≼ t}` (PrefixSpanCoverage, ASN-0043) is
  disjoint from content, `coverage(e₃) ∩ dom(Σ.C) = ∅`.

The second, at a distinct address `ℓ₂ ≠ ℓ₁`, is `L₂ = (e₁, e₂′, e₃′)`: it carries the
*same from-endset value* `e₁` in slot 1, which the non-injective store permits (L11b,
ASN-0043). Its remaining two slots we leave abstract but constrain exactly where the
argument needs it — both *miss the region we draw below*:

> `coverage(e₂′) ∩ {a₂} = coverage(e₃′) ∩ {a₂} = ∅`

(equivalently, neither `e₂′` nor `e₃′` covers `a₂`). Concretely one may take `e₂′ = e₂`
and `e₃′ = e₃`, making `L₂` a value-identical twin of `L₁` — which L11b permits, the
distinct address notwithstanding — but the analysis below uses only the stated
disjointness. Both links are addressable.

We ask of the single middle position, `W = {[1,2]}`. The region resolves to its image

> `I = image(W, d, Σ) = { Σ.M(d)([1,2]) } = {a₂}`.

Run the touch test against each endset in play:

- `touch_W(e₁) = coverage(e₁) ∩ {a₂}`. Since `a₂ ∈ coverage(e₁)` — via the first, width-2
  span — this is non-empty: `e₁` **touches**, through `a₂`. Its other covered addresses lie
  outside the region: `a₃` (also under the first span) and `a₄` together with its
  descendants (under the second span); none helps or hinders the test.
- `touch_W(e₂) = {t : a₁ ≼ t} ∩ {a₂} = ∅` — `e₂` reaches `a₁`, arranged at `[1,1] ∉ W`,
  and does not reach `a₂`; it **misses**.
- `touch_W(e₃) = {t : θ ≼ t} ∩ {a₂} = ∅` — `a₂ ∈ dom(Σ.C)` and `coverage(e₃) ∩ dom(Σ.C) = ∅`,
  so `a₂ ∉ coverage(e₃)`; it **misses**.
- `touch_W(e₂′) = coverage(e₂′) ∩ {a₂} = ∅` and `touch_W(e₃′) = coverage(e₃′) ∩ {a₂} = ∅`
  — directly by the disjointness stipulated on `L₂`'s two non-from slots; both **miss**.

The only endset that touches the region is `e₁`, carried in slot 1 by both `ℓ₁` and `ℓ₂`;
every other slot of either link — `e₂`, `e₃`, `e₂′`, `e₃′` — misses. The answer is
therefore a single role-tagged endset,

> `RE(W, d, Σ) = { (1, e₁) }`,

and each of the operation's distinctive claims can be read off it directly:

- **Overlap, not containment (RE-OVL).** `e₁` is surfaced although `coverage(e₁) ⊋ I` — it
  covers `a₃` and `a₄`, which the region does not. A single shared address, `a₂`, sufficed.
  Under a containment test (`coverage(e₁) ⊆ I`) the from-endset would have been wrongly
  discarded, precisely because it straddles the boundary.
- **Unclipped extent (RE-CLIP).** The touching (first) span of the surfaced `e₁` is
  returned at its full recorded extent — the width-2 span `(a₂, δ(2, #a₂))`, reaching
  across `a₃` — not trimmed to the region. A clipping implementation would have returned
  the width-1 span `(a₂, δ(1, #a₂))`, whose coverage `{t : a₂ ≼ t}` (PrefixSpanCoverage,
  ASN-0043) reaches `a₂` and its descendants — hence, among the four arranged pieces, `a₂`
  alone, since `a₃` is a sibling rather than a descendant — falsely shrinking the link's
  grip from `{a₂, a₃}` to `{a₂}` to fit the query.
- **Whole-endset surfacing (RE-WHOLE).** The surfaced `e₁` is returned *entire* — both
  spans, including the unit span at `a₄`, which touches nothing the region holds. Here the
  reading is exercised in earnest, and its distinctive consequence is concrete: the answer
  volunteers anchoring — `a₄` and its descendants — that points *wholly outside* the
  queried region. The two readings part on this instance: the touching-spans reading would
  keep only the first span — it alone meets `I = {a₂}`, the unit span at `a₄` missing it —
  surfacing `{(1, {(a₂, δ(2, #a₂))})}`, honest about extent yet silent about the `a₄` span,
  whereas the *whole-endset* reading we adopt returns
  `RE(W, d, Σ) = {(1, {(a₂, δ(2, #a₂)),  (a₄, δ(1, #a₄))})}` in full.
- **Per-endset surfacing (RE-OVL).** Only slot 1 appears, and from each link separately. Of
  `L₁`, the to-endset `e₂` and the type-endset `e₃` miss the region and are absent, so the
  link's from-end is reported without its to-end. Of `L₂`, the two non-from slots `e₂′` and
  `e₃′` likewise miss — by the disjointness stipulated above — leaving only its shared
  slot-1 `e₁` to contribute. No slot but the first survives the touch test, from either
  link.
- **Anchoring without names (RE-UNIT).** `ℓ₁` and `ℓ₂` both bear `e₁` in slot 1; they
  contribute the *one* pair `(1, e₁)`, which appears once. The answer holds no `ℓ₁`, no
  `ℓ₂`, no count. From `{(1, e₁)}` alone one cannot tell that two links grip here, cannot
  recover either identity, and cannot learn that `e₁`'s links also anchor a to-end at `a₁`
  — the cross-end pairing RE-UNIT withholds.

## Composing regions: union-distributivity

One naturally asks whether a region query decomposes — whether asking of `W₁ ∪ W₂` is the
same as asking of `W₁` and of `W₂` separately and taking the union. For unions it is, and
the proof is short, because it rests on a property the forward image enjoys
unconditionally.

The image of a union is the union of images:

> `image(W₁ ∪ W₂, d, Σ) = image(W₁, d, Σ) ∪ image(W₂, d, Σ)`,

with no injectivity hypothesis. This is immediate from the definition: writing
`f = Σ.M(d)`, we have `image(W, d, Σ) = {f(v) : v ∈ W ∩ dom(f)}`, and
`(W₁ ∪ W₂) ∩ dom(f) = (W₁ ∩ dom(f)) ∪ (W₂ ∩ dom(f))`, so the comprehension over the union
is the union of the comprehensions. (A forward image always distributes over union; it is
*intersection* it fails to respect, as we note below.)

The touch test then distributes as a disjunction. The subscript on `touch_W` (the
predicate fixed above) names the region it tests against, so it specialises to each
sub-region — `touch_{W₁}`, `touch_{W₂}` — and to their union,

> `touch_{W₁ ∪ W₂}(e) ≡ coverage(e) ∩ (image(W₁, d, Σ) ∪ image(W₂, d, Σ)) ≠ ∅ ≡
> touch_{W₁}(e) ∨ touch_{W₂}(e)`,

since a set meets a union of sets exactly when it meets one of them. Now `RE` selects each
*available* slot-endset by exactly this test — and the pool of available pairs,
`Avail(Σ) = { (i, e) : (∃ a ∈ addressable(Σ) : 1 ≤ i ≤ |Σ.L(a)| ∧ Σ.L(a).eᵢ = e) }`, is a
function of `(Σ.L, nullified(Σ))` and does not depend on the region. So

> `RE(W₁ ∪ W₂, d, Σ) = { (i, e) ∈ Avail(Σ) : touch_{W₁ ∪ W₂}(e) }
>   = { (i, e) ∈ Avail(Σ) : touch_{W₁}(e) ∨ touch_{W₂}(e) }
>   = RE(W₁, d, Σ) ∪ RE(W₂, d, Σ)`.

This is the RETRIEVEENDSETS analogue of the discovery query's union-distributivity
(F-UDIST, F-VDIST, ASN-0127): a region query is composable from any cover of the region by
its parts. Querying a passage is the union of querying its lines.

The *intersection* law does not follow in full — but one half of it does, and
unconditionally. We must first see where the forward image fails to distribute over
intersection: in general
`image(W₁ ∩ W₂, d, Σ) ⊆ image(W₁, d, Σ) ∩ image(W₂, d, Σ)`, but the inclusion can be
strict, because distinct V-positions may map to the *same* I-address — the arrangement is
non-injective (M13, M14, ASN-0058). A position in `W₁ ∖ W₂` and a position in `W₂ ∖ W₁`
can share an I-address that then lies in both images, hence in the right-hand
intersection, while contributing nothing to `image(W₁ ∩ W₂, d, Σ)`.

The image `⊆` law that does hold, however, already settles one direction of the RE-level
intersection law. From `image(W₁ ∩ W₂, d, Σ) ⊆ image(W₁, d, Σ)` and the symmetric inclusion
for `W₂`, an endset meeting the smaller image meets each larger one, so the touch test
implies *both* of its sub-region instances:

> `touch_{W₁ ∩ W₂}(e) ⟹ touch_{W₁}(e) ∧ touch_{W₂}(e)`.

Filtering the region-independent pool `Avail(Σ)` by this implication — exactly as the union
proof filters it by the disjunction — gives

> `RE(W₁ ∩ W₂, d, Σ) = { (i, e) ∈ Avail(Σ) : touch_{W₁ ∩ W₂}(e) }
>   ⊆ { (i, e) ∈ Avail(Σ) : touch_{W₁}(e) ∧ touch_{W₂}(e) }
>   = RE(W₁, d, Σ) ∩ RE(W₂, d, Σ)`,

and, like the image `⊆` law it rests on, this needs *no* injectivity hypothesis.

The reverse inclusion, by contrast, **fails in general**. There are two independent obstructions to `⊇`, and the
constructions below separate them.

The first obstruction is the image-level non-distribution noted just above, and a
non-injective arrangement exhibits it. Let `d` arrange two *distinct* V-positions to one
content I-address,

> `Σ.M(d) = { [1,1] ↦ a,  [1,2] ↦ a }`,   `a ∈ dom(Σ.C)`,

and let `ℓ_e ∈ dom(Σ.L)` be a link emitted by `K.λ` carrying in slot 1 the unit-depth
from-endset `e = {(a, δ(1, #a))}` — so `Σ.L(ℓ_e).e₁ = e` and `coverage(e) = {t : a ≼ t}`
(PrefixSpanCoverage, ASN-0043) — addressable in this post-state as a non-retraction
emission, hence `ℓ_e ∉ nullified(Σ)` (RE-ADDR, taking `Σ` as the emission's post-state); so
`(1, e) ∈ Avail(Σ)`. Take the disjoint regions `W₁ = {[1,1]}` and `W₂ = {[1,2]}`:
the two distinct positions `[1,1] ∈ W₁ ∖ W₂` and `[1,2] ∈ W₂ ∖ W₁` carry the shared address
into both images, `image(W₁, d, Σ) = image(W₂, d, Σ) = {a}`, while `W₁ ∩ W₂ = ∅`. Since
`a ∈ coverage(e)` (reflexivity of `≼`), `touch_{W₁}(e)` and `touch_{W₂}(e)` both hold — here
through the *same* witness `a` — so `(1, e) ∈ RE(W₁, d, Σ) ∩ RE(W₂, d, Σ)`; but the
intersection region is empty, so `image(W₁ ∩ W₂, d, Σ) = ∅`, whence `RE(W₁ ∩ W₂, d, Σ) = ∅`
(RE-BND) and `(1, e) ∉ RE(W₁ ∩ W₂, d, Σ)`, refuting `⊇`.

The second obstruction lives in `touch_W` itself, and *no injectivity-style restriction
escapes it*: it is constructible under any arrangement rich enough to admit two disjoint
regions with distinct nonempty images — injectivity included. The touch test is existential — `coverage(e)` need only
*meet* the image — and "meets `image(W₁)`" together with "meets `image(W₂)`" does not entail
"meets `image(W₁ ∩ W₂)`", because the two meetings may be witnessed by *different* addresses.
This is independent of whether `image` distributes over intersection. Take the typical,
fully reachable *injective* arrangement

> `Σ.M(d) = { [1,1] ↦ a,  [1,2] ↦ b }`,   `a ≠ b` in `dom(Σ.C)` — **injective**,

and let `ℓ_{ab} ∈ dom(Σ.L)` be a `K.λ`-emitted link bearing in slot 1 the *two-span*
from-endset `e = {(a, δ(1, #a)),  (b, δ(1, #b))}`, so
`coverage(e) = {t : a ≼ t} ∪ {t : b ≼ t} ⊇ {a, b}` (PrefixSpanCoverage, ASN-0043);
addressable as a non-retraction emission (RE-ADDR), so `(1, e) ∈ Avail(Σ)`. With the disjoint
content-subspace regions `W₁ = {[1,1]}` and `W₂ = {[1,2]}`: `image(W₁, d, Σ) = {a}`,
`image(W₂, d, Σ) = {b}`, and `W₁ ∩ W₂ = ∅`. Now the image distributes *perfectly* —
`image(W₁) ∩ image(W₂) = {a} ∩ {b} = ∅ = image(W₁ ∩ W₂)` — so the first obstruction is wholly
absent. Yet `touch_{W₁}(e)` holds *via `a`* and `touch_{W₂}(e)` holds *via `b`* — *distinct*
witnesses — so `(1, e) ∈ RE(W₁, d, Σ) ∩ RE(W₂, d, Σ)`; while `image(W₁ ∩ W₂, d, Σ) = ∅` gives
`RE(W₁ ∩ W₂, d, Σ) = ∅` (RE-BND), so `(1, e) ∉ RE(W₁ ∩ W₂, d, Σ)`. The `⊇` direction fails
with `Σ.M(d)` **injective** (RE-UDIST-∩). The same shape works for *overlapping* `W₁, W₂` — an
endset covering one address drawn from each region's exclusive part — so the obstruction is
not even confined to disjoint regions.

What *does* force `⊇` is degeneracy of an
altogether different kind — an arrangement too impoverished to furnish two disjoint regions
with distinct nonempty images at all (a single active V-position, say): then any `W₁, W₂` with
both images nonempty share that position, so `image(W₁ ∩ W₂, d, Σ)` is nonempty and the
split-witness construction cannot be mounted, and `⊇` holds vacuously. Because the `⊆` half is unconditional and
the pool `Avail(Σ)` is region-independent, `⊇` — and hence equality — holds *exactly*
(necessary and sufficient) when

> `(∀ (i, e) ∈ Avail(Σ) : touch_{W₁}(e) ∧ touch_{W₂}(e) ⟹ touch_{W₁ ∩ W₂}(e))`,

every available endset that meets both region images also meeting the image of their
intersection. A structurally-checkable sufficient condition is left open (Open Question 4).

## Existence and discoverability: which side does this answer for?

ASN-0127 separates **existence** queries — a fixed `I ⊆ T`, answering a monotone,
*historical* property of the permanent store — from **discovery** queries — `I` resolved
*through a document's current arrangement*, answering a non-monotone, *present-tense* one
(*Anchoring: existence vs discovery*; E-MONO, D-NONMONO, D-ZERO, ASN-0127). We place
RETRIEVEENDSETS on that line.

RETRIEVEENDSETS takes a region `(W, d)` and resolves it through `image(W, d, Σ)`. **It is
discovery-anchored.** Its selection of which links contribute is exactly the discovery
query, filtered to the addressable:

> `sel(W, d, Σ) = { a ∈ addressable(Σ) : (∃ i : touch_W(Σ.L(a).eᵢ)) } = findlinks_V(W, d, Σ) ∩ addressable(Σ)`,

because `findlinks_V(W, d, Σ) = {a ∈ dom(Σ.L) : (∃ i : coverage(Σ.L(a).eᵢ) ∩ image(W,d,Σ) ≠ ∅)}`
(F-V, F-FIND, F-MATCH, ASN-0127). So the links whose endsets RETRIEVEENDSETS surfaces are
precisely the addressable links discoverable through the region — and the operation
inherits the discovery side's temporal character wholesale: present-tense, non-monotone,
arrangement-mediated. A RETRIEVEENDSETS zero — *no anchoring surfaced* — is therefore a
statement of the present (nothing is reachable through this region as it now stands), not
of history (D-ZERO, ASN-0127). The anchoring may well exist, permanently, in links whose
content the region no longer arranges.

## Anchoring reached through borrowed content

A region need not hold content native to its own document. Through transclusion, `d` may
window content whose home is another document `d_src`; the windowed content is not a copy
but the *same content*, sharing its I-addresses with the home (ASN-0036). The question
arises: if a link resides in `d_src` (or anywhere), and reaches our region only because
`d` windows the link's content, must RETRIEVEENDSETS surface that link's endset — and must
it treat such borrowed-content anchoring the same as anchoring on native content?

It must, and the reason is structural: both the anchoring and the query are keyed to
*content identity*. The endset covers I-addresses naming the content's permanent home, not
any borrowing V-position (ASN-0043); the region's image is I-addresses; `touch_W` intersects
them, consulting nothing about where the link lives, where the content is "native," or which
document windows it. This is precisely the discovery query's document-blindness:
discoverability of a link through `d` "turns solely on `coverage ∩ ran(Σ.M(d))`," independent
of the link's home and of the origin of the covered content (LP16, ASN-0098). So if `d`'s
arrangement maps the region's V-positions to I-addresses an endset covers, that endset
touches the region — whether those I-addresses are native to `d` or borrowed from `d_src`. A
link reaching the region through transcluded content is therefore surfaced **identically** to
one reaching native content, and one endset is the same anchoring through every
co-transcluder: at the level of content identity there is nothing to distinguish.

This connects to a general invariant, independent of transclusion: **each surfaced endset's
coverage is permanent**. Links are immutable (L12, ASN-0043) and no transition alters an
endset's coverage (LP3, ASN-0098) — so across every reachable sequence its coverage holds
fixed (LP3★, ASN-0098); once an endset is surfaced the I-addresses it anchors are a fixed
fact about a permanent link (RE-IDENT). The transclusion case is one
reading of it: the *content-level* answer — which content each surfaced endset anchors to
— is invariant, even though what the region's image maps to, and how that image sits
within `d`'s order, are present-tense. The operation's content-level answer is therefore
arrangement-independent even though its *selection* of which endsets to surface is
arrangement-mediated.

## Stability: the answer as the document is edited

Because the operation is a pure function of the present state, resolved through the
present arrangement, its stability is entirely determined by how state changes move the
two things it reads: the region's image and the addressable population. By RE-IDENT each
surfaced endset's coverage is permanent, so editing can change *which* endsets are surfaced
— the membership of the answer — but never the spans of one that is. Every motion
catalogued below is a motion of membership.

**Under editing of the queried document.** The region resolves through `Σ.M(d)`, so
editing `d`'s arrangement moves the image, and the answer tracks it — present-tense,
non-monotone (D-NONMONO, ASN-0127). We read its response first against the *atomic*
arrangement movers of the transition vocabulary (ASN-0047) — extension, contraction,
reordering — each acting as a faithful tracker would:

- *Arrangement extension* `K.μ⁺` appends new V→I mappings at the contiguous frontier of
  `d`'s content subspace, leaving every existing mapping fixed and the arrangement canonical
  (D-CTG/D-SEQ, ASN-0047). The region image can only grow, and only weakly:
  `image(W, d, Σ) ⊆ image(W, d, Σ')` (F-IMG-MONO, ASN-0127). When `W` reaches the appended
  frontier positions, the new content enters the image, endsets covering it newly touch the
  region, and they are surfaced — anchoring that was always there in the store becomes
  *reachable* here without any link being created. When the fixed region does *not* include
  the frontier, the append adds nothing under `W`: the inclusion is equality, and the
  image — hence `RE` — is unchanged.

- *Arrangement contraction* `K.μ⁻` truncates the tail of `d`'s content subspace, retaining a
  canonical prefix `R` of content V-positions and dropping the rest (ASN-0047). The region
  image can only shrink, and only weakly: `image(W, d, Σ') ⊆ image(W, d, Σ)` (F-IMG-CONTR,
  ASN-0127). When `W` reaches the dropped tail, endsets that touched only through the
  departed content cease to be surfaced — the contracted image no longer meets their
  coverage, so the touch test fails where it formerly held (the contraction direction of
  LP10 and the discoverability characterisation LP12, ASN-0098). The link persists in the
  store (L12, ASN-0043), its endset coverage unchanged; it is merely no longer reachable
  *through this region of `d`*. This is a region-local loss of reach, **not** the global
  *orphaning* of LP17 (ASN-0098), whose premise — that the content is reachable from *no*
  document — a single-region contraction does not establish: the link may still touch other
  regions of `d`, or be reachable from other documents. Should the content be re-arranged
  into `d`, the region image grows again (F-IMG-MONO, LP9, ASN-0098) and the endset is
  surfaced once more. When the fixed region lies
  wholly within the retained prefix `R`, the truncation drops nothing under `W`: the
  inclusion is equality, and `RE` is unchanged.

- *Arrangement reordering* `K.μ~` permutes the region's V-positions over the same content;
  the *region image*'s membership can swing (F-IMG-SWING, ASN-0127). This is the *only* way
  a reordering changes the answer: which `(i, e)` pairs are surfaced may change as the image
  swings and some endset newly meets it or ceases to. (A *rendered* answer — one resolved into
  `d`'s present V-order rather than content identity, where piecewise displacement could
  fragment a contiguous run (ASN-0082) — is the mode deferred to Open Question 3; the
  content-identity answer returned here is unaffected.)

The user-facing *insert* and *delete* that **shift** content are not these atomic movers: they
are ASN-0082's displacement primitives, which model the system over a `(C, M)` state omitting
`Σ.L`, `Σ.E`, `Σ.R`. We **adopt as a modelling assumption** the *conservative lift* — that
shift-based insert/delete touch no store but `Σ.M(d)`, framing `Σ.L`, `Σ.E`, `Σ.R`, and `Σ.C`.
Each is then an **M-only edit**, an arrangement edit confined to `Σ.M(d)`, and `RE` tracks it by
image membership exactly as it tracks every ASN-0047 atomic mover above — at *any* content depth.
The addressable population is unmoved across the shift: `addressable(Σ)` and the
region-independent pool `Avail(Σ)` are functions of `Σ.L` (through `nullified`) alone, so only
the region's image can move. Content is *displaced through* `d`'s V-order, and the image's
response is read off the displacement directly — not one-signed the way `K.μ⁺`/`K.μ⁻` are, but
possibly making the fixed region's image *gain*, *lose*, or *both* (the shift family is
non-monotone as a class). At each reachable post-edit state, then, `RE` tracks the image's
motion by membership, each surfaced endset's spans held fixed (RE-IDENT).

Editing of *other* documents does not perturb the answer: the image reads only `Σ.M(d)`,
and a transition touching `d' ≠ d` leaves `Σ.M(d)` fixed (LP5, ASN-0098). Three further
transition kinds leave the answer fixed for the same root reason — each leaves the queried
fiber `Σ.M(d)` and the link store `Σ.L` fixed, so neither the image nor the available pool
moves. *Content allocation* `K.α` frames both stores the answer reads — `M' = M` and
`L' = L` (ASN-0093, frame-extended in ASN-0047) — so the image and `Avail(Σ)` are fixed
outright. *Provenance recording* `K.ρ` only extends `Σ.R`, framing `M'(d) = M(d)` and
`L' = L` (ASN-0047) — so again neither moves. *Entity creation* `K.δ` — registering a new node, account, or document
`e ≠ d` — leaves `Σ.M(d)` untouched (wholly, for node/account creation, where `M' = M`; and
on every pre-existing arrangement, for document registration, LP8, ASN-0098) and leaves
`Σ.L` fixed (frame); document registration is the lone non-trivial frame among the three,
which LP8 discharges.

Finally, a whole *class* of arrangement edits to `d` *itself* leaves a content-region answer
fixed — by a route particular to the content-subspace restriction rather than to locality:
the **link-subspace-confined** edits, those touching only `d`'s link subspace. The
link-subspace extension `K.μ⁺_L` adds a single V-position `v_ℓ` with `subspace(v_ℓ) = s_L`,
mapped to a link address (ASN-0047). Because `W ⊆ s_C`, we have `v_ℓ ∉ W`, so the selecting
set `W ∩ dom(Σ.M(d))` — and with it the image `image(W, d, Σ)` — is unchanged
(F-IMG-MONO sharpened to equality under `W ⊆ s_C`, ASN-0127); and its frame leaves `Σ.L`
fixed (`L' = L`), so the available pool does not move either. A **link-subspace-only
contraction** `K.μ⁻` is secured by the identical route — one retaining the whole content
subspace (`n'_{s_C} = n_{s_C}`) while strictly contracting the link subspace
(`n'_{s_L} < n_{s_L}`, admissible whenever `V_{s_L}(d) ≠ ∅`, since `K.μ⁻` requires at least
one subspace to strictly contract): for `W ⊆ s_C`, retained-position agreement gives
`W ∩ dom(Σ'.M(d)) = W ∩ V_{s_C}(d) = W ∩ dom(Σ.M(d))`, so `image(W, d, Σ') = image(W, d, Σ)`,
while `K.μ⁻`'s frame leaves `Σ.L` fixed and `Avail(Σ)` with it. Either edit gives
`RE(W, d, Σ') = RE(W, d, Σ)`.

**The weakest precondition for contraction-stability.** The qualitative tracking above can
be made exact for one editing motion — a deletion. Fix a `K.μ⁻[d, R]` step on the queried
document: a contraction retaining exactly the arrangement positions in the retention set
`R` (ASN-0047), with `enabled(K.μ⁻[d, R])` its applicability condition. We ask the natural non-trivial
stability question: under what precondition on `Σ` does this step leave `RE(W, d, ·)`
*unchanged*? Two observations narrow it.

First, `K.μ⁻` touches only `Σ.M(d)`; its frame leaves `Σ.L`, and hence `nullified(Σ)`,
fixed. So the *available* pairs `Avail(Σ)` — defined at union-distributivity above and
region-independent — are identical pre- and post-state; only the `touch_W` filter can move. Second, contraction only shrinks the image. By the bridge of D-CWP
(ASN-0127), `image(W, d, Σ') = I_R` where `I_R = {Σ.M(d)(v) : v ∈ W ∩ R}`; writing
`Δ = image(W, d, Σ) ∖ I_R` for the dropped I-addresses, `I_R ⊆ image(W, d, Σ)`. A touch
against the smaller post-image therefore implies a touch against the pre-image, so
`RE(W, d, Σ') ⊆ RE(W, d, Σ)` unconditionally. "Unchanged" reduces to "nothing dropped."

A pair `(i, e)` is dropped exactly when its endset touched the region *only* through the
departing addresses — `coverage(e) ∩ image(W, d, Σ) ≠ ∅` (surfaced pre) yet
`coverage(e) ∩ I_R = ∅` (gone post), which together say `coverage(e) ∩ Δ ≠ ∅` and
`coverage(e) ∩ I_R = ∅`. Demanding that no available pair be dropped:

> `wp(K.μ⁻[d, R],  RE(W, d, ·) = RE(W, d, Σ))  ≡`
> `enabled(K.μ⁻[d, R]) ∧ (∀ (i, e) ∈ Avail(Σ) : coverage(e) ∩ Δ ≠ ∅ ⟹ coverage(e) ∩ I_R ≠ ∅)`

— every available endset that reaches a dropped address must also reach an address
*retained within the region*. Both `I_R` and `Δ` are functions of the pre-state `Σ` and
the retention set `R` alone, so the condition is checkable before the edit.

This is the per-endset refinement of the discovery query's contraction-stability (D-CWP,
ASN-0127), and it is strictly finer. D-CWP asks that every *link* reaching `Δ` also reach
`I_R` — `findlinks(Δ, Σ) ⊆ findlinks(I_R, Σ)`, an existential over slots on each side, so a
link may satisfy it by reaching `Δ` through its from-endset and `I_R` through its
to-endset. RETRIEVEENDSETS surfaces the *endsets*, not the links, so it demands the *same*
endset reach both: a link whose from-endset touches only `Δ` while its to-endset rescues
`I_R` survives in `findlinks_V` yet still drops the pair `(1, from-endset)` from `RE`. The
boundary is D-CWP's, read through the finer lens: at `R = ∅` (full clearance), `I_R = ∅`
and the condition collapses to `(∀ (i, e) ∈ Avail(Σ) : coverage(e) ∩ Δ ≠ ∅ ⟹ false)` —
i.e. no available endset touches `image(W, d, Σ)` at all, which is `RE(W, d, Σ) = ∅`.
Clearing the region preserves the answer exactly when the answer was already empty.

**Under link emission.** The one population-*growing* mover is a `K.λ` step that emits a
fresh link `ℓ_new`. By RE-ADDR, a non-retraction emission (`K ≁ Θ`) is addressable in its
post-state, so `ℓ_new ∈ addressable(Σ')`. The step frames the arrangement (`M' = M`,
ASN-0093), so the image — and every `touch_W` it determines — holds fixed; only the available
pool can move. If some endset `Σ.L(ℓ_new).eᵢ` touches the region, the pair
`(i, Σ.L(ℓ_new).eᵢ)` is *added* to the answer; if none does, the answer is unchanged. Either
way the move is monotone — a non-retraction emission (`K ≁ Θ`) can only add pairs, never
remove one — the population-grow analogue of the discovery query's E-MONO/F-LAMBDA (ASN-0127).

**Under self-retraction.** The self-emit `Nullify` — emitter = target — is inert. It is
ASN-0086's `Nullify(Σ, d_retr, a)` with `a = a_emit(Σ, d_retr)` (P-tgt's self-emit branch): a
fresh emission whose unit-depth to-set targets *its own* emitter address `b`. The emitter
`b` is *born-nullified* — its own to-set covers `b`, so `b ∈ nullified(Σ')`, leaving `b`
non-addressable by RE-ADDR's excluded branch and surfacing nothing — and by R-Scope at
target = emitter (ASN-0086) the fresh nullification reaches only the fresh `b`
(`{t : b ≼ t} ∩ dom(Σ'.L) = {b}`), so no *pre-existing* addressable bearer is touched. Adding
no addressable bearer and removing none — `addressable(Σ') = addressable(Σ)` — with `Σ.M(d)`
framed (`M' = M`) holding the image, it leaves `RE(W, d, Σ') = RE(W, d, Σ)`.

**Under retraction.** A link is never deleted (L12, ASN-0043), but it can be *withdrawn*:
a retraction marks it nullified (ASN-0086), and we range only over the addressable links.

A retraction is itself a link emission, and this matters for what a retraction step does to
the population. Withdrawing `ℓ` is realised as `Nullify(Σ, d_retr, ℓ) ≡ Emit_Θ(Σ, d_retr,
∅, {(ℓ, δ(1, #ℓ))})` (ASN-0086), and `Emit_Θ` *is* a `K.λ` step (Emit_K, ASN-0086): it
emits a fresh **retraction link** `b`, with `Σ'.L(b) = (∅, {(ℓ, δ(1, #ℓ))}, Θ)`. Its to-set
covers `ℓ`, not `b` (`ℓ ≠ b`, both in the flat antichain), so `b` does not retract its own
emitter address; `b` is therefore addressable in `Σ'` (`b ∉ nullified(Σ')`) by RE-ADDR. So a
single retraction does two things at once — it removes `ℓ` from `addressable` (through the
nullified marking) *and* adds the emitter `b` to it. We must ask what the emitter `b` can contribute. Its three
endsets are the from-set `∅` — empty by `Nullify`'s definition (`Emit_Θ(…, ∅, …)`, ASN-0086) —
a to-set `{(ℓ, δ(1, #ℓ))}` whose single span covers `ℓ` and `ℓ`'s extensions, and the retraction
type-set `Θ`. The first two are content-disjoint *unconditionally*. The from-set is empty, and
`coverage(∅) = ∅` touches nothing. The to-set is a unit-depth span — ASN-0086's `Nullify` fixes
its width at `δ(1, #ℓ)` — whose start `ℓ` is a link address `ℓ ∈ dom(Σ.L)` that `Nullify`
targets, hence T4-valid (StoreT4Validity, ASN-0093) and genuinely element-level
(`zeros(ℓ) = 3`, L1, ASN-0093) with non-content subspace identifier `E(ℓ)₁ = s_L ≠ s_C`
(L0, ASN-0093; SC-NEQ), so RE-NCD applies directly:
`coverage({(ℓ, δ(1, #ℓ))}) ∩ dom(Σ.C) = ∅`. So against a content image
`I ⊆ dom(Σ.C)`, neither the from-set nor the to-set can touch. (This content-disjointness is
exactly what the standing `W ⊆ s_C` obligation buys.)

The type-set `Θ` is the slot RE-NCD does *not* reach. A type endset may, by
design, point *anywhere* in the address space, content included (L4 EndsetGenerality, L9
TypeGhostPermission, ASN-0043), and ASN-0086 fixes the designated retraction type only as a
type endset whose coverage selects the conventional retraction address set — carrying no
structural disjointness from content, and not confined to unit depth. RE-NCD is confined to
unit-depth spans, where coverage reduces to the prefix relation `s ≼ c`; across the interior of
a *wide* span `(s, ℓ_s)` that argument fails, and the interval
`{t : s ≤ t < s ⊕ ℓ_s}` may include content even when `s` lies outside it. So
`coverage(Θ) ∩ dom(Σ.C) = ∅` is a construction hypothesis, not a theorem; this note carries
it as such, its exception — a type-slot match against content — taken up by Open Question 6.

*Under* this hypothesis `coverage(Θ) ∩ dom(Σ.C) = ∅` the emitter `b` surfaces nothing against
a content image `I ⊆ dom(Σ.C)`, so a retraction's *net* effect on `RE` is removal only.
*Absent* it, `b`'s type-slot `Θ` could meet the content image and surface the fresh pair
`(3, Θ)`, making the retraction *add* anchoring as well as remove it; the forward direction of
the stability result below therefore rests on the hypothesis.

But the answer deduplicates, and we must read its stability at the granularity it actually
has. Its elements are `(role, endset)` pairs with link identity discarded (RE-UNIT): a
pair `(i, e)` is present exactly when *some* addressable link bears `e` in slot `i` and `e`
touches the region. So withdrawing one link `ℓ` does not, by itself, remove the pairs it
bore. Retracting `ℓ` removes `ℓ` from `addressable(Σ)`, and *permanently* so: by R6a (ASN-0086)
and induction, `ℓ ∈ nullified` at every state reachable from the retraction's post-state. The same
step adds the emitter `b`, whose only possible content-region contribution is the fresh pair
`(3, Θ)` (just shown) — inert under the net-removal-only hypothesis (above), so `b` re-witnesses
no pair the answer carries. A pair
`(i, e)` that `ℓ` contributed therefore leaves the answer **iff `ℓ` was its sole addressable
bearer in `Σ`**. The forward half — sole bearer ⟹ the pair drops — is the conjunction just
assembled: `ℓ` leaves `addressable` permanently (just shown) and the emitter `b` is inert (above),
so neither keeps `(i, e)` alive. The backward half — some *other* live bearer ⟹
the pair survives — is not free: it asserts that the other bearer outlives the very step
that withdraws `ℓ`, and a retraction, being a state transition, could a priori nullify
more than its named target. We discharge it by bounding the retraction's reach. Take
`ℓ' ∈ addressable(Σ)` with `ℓ' ≠ ℓ`, bearing `e` in slot `i`. A single Nullify contributes
*exactly* its target to the nullified set — `{t : ℓ ≼ t} ∩ dom(Σ'.L) = {ℓ}`: the unit-depth
to-set `{(ℓ, δ(1, #ℓ))}` covers `{t : ℓ ≼ t}` (PrefixSpanCoverage, ASN-0043), which meets the
prefix-antichain `dom(Σ'.L)` (above) in `ℓ` alone (R-Scope SingleTupleScope, ASN-0086) —
so the fresh retraction tuple `b` nullifies no link
address but `ℓ`, leaving every other store element `ℓ' ≠ ℓ` outside its reach.
Hence `ℓ'`, already `∉ nullified(Σ)` and distinct from `ℓ`, is `∉ nullified(Σ')`:
`ℓ' ∈ addressable(Σ')`. Its value is unchanged (L12, ASN-0043), so it still bears `e` in
slot `i`; the `K.λ` step frames `Σ.M(d)` (`M' = M`), leaving the image — and with it
`touch_W(e)` — fixed. Thus `ℓ'` still witnesses `(i, e)` in `Σ'`, and a pair still borne
by some other addressable link survives the retraction untouched. Our worked instance
makes the distinction concrete: `ℓ₁` and `ℓ₂` both carry `e₁` in slot 1, collapsing to the
single pair `(1, e₁)`; retracting `ℓ₁` alone leaves `(1, e₁)` in the answer, because `ℓ₂`
survives the step — R-Scope confines the fresh nullification to `ℓ₁`, so `ℓ₂` (distinct
from `ℓ₁`) remains in `addressable(Σ')` and still bears `e₁` (value fixed by L12). Only
when *both* are withdrawn does `(1, e₁)` depart — and even then not permanently: the
retracted link's membership in `addressable` is gone forever (R6a, persisting across the
whole ASN-0047 vocabulary), so anchoring can never again surface *because `ℓ₁` bears it*,
yet the *pair value* `(1, e₁)` re-enters the answer the moment any live link bears `e₁` in
slot 1 and touches the region — a still-present bearer, or a freshly emitted link with a new
identity (R6c). The answer tracks the anchoring values of the active *population*, not the
fate of any one link — the slip RE-UNIT's deduplication guards against.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| RE-DEF | `RE(W, d, Σ) = { (i, e) : (∃ a ∈ addressable(Σ) : 1 ≤ i ≤ \|Σ.L(a)\| ∧ Σ.L(a).eᵢ = e ∧ touch_W(e)) }`, where `(W, d)` has `d ∈ dom(Σ.M)` and `W ⊆ T` a content-subspace V-position set resolving to `I = image(W, d, Σ)` (F-IMG, ASN-0127); `touch_W(e) ≡ coverage(e) ∩ I ≠ ∅`; `addressable(Σ) = dom(Σ.L) ∖ nullified(Σ)` (ASN-0086); frame `Σ' = Σ`. Its return value at a selected slot follows the adopted convention RE-WHOLE (§Extent) | introduced |
| RE-LOC | Locality — for fixed `(W, d)`, `RE` reads `Σ.M(d)` (image) and `Σ.L` (endsets, and via `nullified` addressability) alone; `Σ.C`, `Σ.E`, `Σ.R` are never consulted. Hence `RE` is a deterministic function of `(W, d, Σ)` | introduced |
| RE-UNIT | Anchoring without names — the answer's elements are `(role, endset)` pairs, never link identities; the address is withheld, distinct links sharing an endset value collapse to one pair, multiplicity is not recoverable, and a surfaced from-endset cannot be paired with its link's to-endset | introduced |
| RE-OVL | Overlap matching — an endset is surfaced iff at least one address it covers lies in the region's image (overlap, not containment); single-address overlap suffices; the test is existential *within* an endset and applied *per-endset*, with no per-slot request differentiation | introduced |
| RE-CLIP | No clipping — every surfaced span is reported at the full extent recorded in the link, never truncated to the region boundary; holds under both readings (§Extent) | introduced |
| RE-WHOLE | Whole-endset surfacing (adopted convention) — a surfaced endset is returned in full, *all* its spans (not only those intersecting `W`), a commitment separable from RE-CLIP; the alternative touching-spans reading is left open by Open Question 1 (§Extent) | introduced (provisional) |
| RE-BND | Boundary cases — `RE(W, d, Σ) = ∅` whenever the image is empty (`W ∩ dom(Σ.M(d)) = ∅`) or `addressable(Σ) = ∅`; an empty endset slot has `coverage(∅) = ∅`, so `touch_W(∅)` is false and it is never surfaced | introduced |
| RE-NCD | Cross-subspace unit-span disjointness — a unit-depth span `(s, δ(1, #s))` whose T4-valid element-level start has a non-content subspace identifier (`zeros(s) = 3`, `E(s)₁ ≠ s_C`) covers no content: `coverage({(s, δ(1, #s))}) ∩ dom(Σ.C) = ∅` (PrefixSpanCoverage, ASN-0043; S7b, ASN-0036; L0, ASN-0093; field-agreement on separator zeros) | introduced |
| RE-FIN | Finiteness and computability — `RE(W, d, Σ)` is finite *unconditionally* (drawn from the finite supply of slot-endset pairs: `dom(Σ.L)` finite by L-fin, ASN-0093, and each link bears finitely many endsets by L3, ASN-0043); and given a *finitely presented* `W` (region membership `v ∈ W` decidable, e.g. `W` given as finitely many spans), it is computed by finitely many decidable tests over the finite store — image construction over the finite `dom(Σ.M(d))` (S8-fin, ASN-0036), `coverage`-membership by intrinsic comparison on half-open T1-intervals (T12, T2, ASN-0034), and addressability via the computable `nullified` (ASN-0086) | introduced |
| RE-ADDR | Fresh-output addressability — a fresh `K.λ` output that does not retract its own emitter address is addressable in its post-state (`ℓ_new ∉ nullified(Σ')`); in particular every non-retraction emission (`K ≁ Θ`) is addressable. Conditions: the standing discipline's unit-depth to-set and the prefix-antichain of `dom(Σ.L)` (R0a, ASN-0086) | introduced |
| RE-SND | Soundness — `(i, e) ∈ RE(W, d, Σ) ⟹ e` is a genuine slot-`i` endset of an addressable link ∧ `touch_W(e)`; no false positives | introduced |
| RE-CMP | Completeness — every addressable link `a` and slot `i` with `touch_W(Σ.L(a).eᵢ)` has `(i, Σ.L(a).eᵢ) ∈ RE(W, d, Σ)`; the answer is *exactly* the touching set, native or transcluded content alike | introduced |
| RE-UDIST | Union-distributivity — `RE(W₁ ∪ W₂, d, Σ) = RE(W₁, d, Σ) ∪ RE(W₂, d, Σ)`, the RE-level analogue of F-UDIST/F-VDIST (ASN-0127) | introduced |
| RE-UDIST-∩ | Intersection (one-sided) — `RE(W₁ ∩ W₂, d, Σ) ⊆ RE(W₁, d, Σ) ∩ RE(W₂, d, Σ)` holds unconditionally; the reverse `⊇` fails in general, and equality holds exactly under the touch-implication condition `(∀ (i,e) ∈ Avail(Σ) : touch_{W₁}(e) ∧ touch_{W₂}(e) ⟹ touch_{W₁ ∩ W₂}(e))` (§Composing regions) | introduced |
| RE-SEL | Discovery-side selection — `sel(W, d, Σ) = findlinks_V(W, d, Σ) ∩ addressable(Σ)` (F-V, ASN-0127): the contributing links are the addressable links discoverable through the region, so `RE` is discovery-anchored — present-tense, non-monotone (D-NONMONO, D-ZERO, ASN-0127), not existence-anchored | introduced |
| RE-TRANS | Transclusion blindness — surfacing is by content identity, independent of the link's home and the covered content's origin (LP16, ASN-0098): a link reaching the region through transcluded content is surfaced identically to one on native content, each span describing content identity, not the borrowing V-position | introduced |
| RE-IDENT | Content-identity invariance — each surfaced endset's coverage is permanent (L12, ASN-0043; LP3★, ASN-0098), so the content-level answer (which I-addresses each surfaced endset anchors to) is arrangement-independent, even though the *selection* of surfaced endsets is arrangement-mediated | introduced |
| RE-EDIT | Present-tense stability under editing — `RE` tracks `d`'s content-subspace arrangement, so the answer is non-monotone (D-NONMONO, ASN-0127) while each surfaced endset's spans stay invariant (RE-IDENT); it moves only under content-subspace arrangement movers on `d` and `K.λ` emission/retraction (RE-RET), and is left fixed by every other transition (§Stability) | introduced |
| RE-RET | Retraction stability — withdrawing a link `ℓ` (Nullify, ASN-0086) marks it nullified, removing it from `addressable(Σ)` permanently; under the net-removal-only hypothesis `coverage(Θ) ∩ dom(Σ.C) = ∅`, a pair `(i, e)` that `ℓ` bore drops iff `ℓ` was its sole addressable bearer in `Σ` (RE-UNIT; §Stability) | introduced |
| RE-CWP | Contraction-stability weakest precondition — for a `K.μ⁻[d, R]` step, `RE(W, d, ·) = RE(W, d, Σ)` iff `enabled(K.μ⁻[d, R]) ∧ (∀ (i, e) ∈ Avail(Σ) : coverage(e) ∩ Δ ≠ ∅ ⟹ coverage(e) ∩ I_R ≠ ∅)`, where `I_R = {Σ.M(d)(v) : v ∈ W ∩ R}` (D-CWP bridge, ASN-0127) and `Δ = image(W, d, Σ) ∖ I_R`. The boundary `R = ∅` collapses to `RE(W, d, Σ) = ∅` | introduced |

## Open Questions

1. Must a surfaced endset be reported in its entirety, or only those of its spans that intersect the region (the whole-endset return value vs. the touching-spans restriction, §Extent) — and which choice is the faithful rendering of the link's anchoring, weighing the union-distributivity trade-off set out there (RE-UDIST)?

2. When distinct addressable links carry an identical endset value in the same slot, must the operation's answer preserve their multiplicity, or is collapsing them to a single surfaced endset a faithful answer?

3. When a surfaced endset is rendered into the querying document's V-positions rather than content identity, what must the answer guarantee for endset content the document does not currently arrange?

4. Given the exact, necessary-and-sufficient touch-implication characterisation of intersection-equality and the proof that no *injectivity-style* restriction alone discharges it (both settled in RE-UDIST-∩), what is the weakest *structurally-restricted sufficient* condition — phrased directly on the available endsets' coverages and the three region images `image(W₁)`, `image(W₂)`, `image(W₁ ∩ W₂)`, with the per-endset `touch` quantifier eliminated?

5. What completeness guarantee must hold when anchoring that touches a region resides in a link store not co-resident with the queried document?

6. What must hold of a type-slot match against a content region for it to be meaningful, given that type endsets are matched by address and ordinarily reference classifying addresses disjoint from content?

7. What must a region query guarantee when its V-positions are drawn from the link subspace (`subspace(v) = s_L`) rather than the content subspace?
