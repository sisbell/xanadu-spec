> **ASN-0113 · RETRIEVEDOCVSPANSET Operation — Per-Subspace Document Extent Query** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md)  
> [Condensed statements →](ASN-0113-retrievedocvspanset-operation-per-subspace-document-extent-query.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0113: RETRIEVEDOCVSPANSET Operation — Per-Subspace Document Extent Query

*2026-06-04*

We are trying to understand the per-subspace extent query: a document is handed over by
identity alone — no range, no position, no selection — and is expected to report the extent
of *each* of its content kinds separately, how much text and how many links.

Nelson fixes the shape exactly. RETRIEVEDOCVSPANSET "returns a span-set indicating both
the number of characters of text and the number of links in document `<doc id>`" (4/68).
Two facts are packed into that sentence. First, the result is a *span-set* — a series of
spans (4/25) — not a single span. Second, the spans report *two distinct kinds*: text
characters and links, because "there is essentially nothing in the Xanadu system except
documents and their arbitrary links" (4/41), and these two kinds occupy *separate
subspaces* of the document's address tree. Our task is to say, formally, what each member
of that span-set denotes, what relationship each must bear to the subspace it measures,
what the caller learns from seeing the two together that neither alone could show, and what
invariants must hold *across* the members the operation returns.

We write the operation as a pure query, `RETRIEVEDOCVSPANSET(d)`, that observes the state
and returns a value, changing nothing. The entire content of this note is: *what is that
value, and what must hold of it?*

---

## The substrate we measure

We take the strand model of state as given. A document `d` — a T4-valid tumbler with
`zeros(d) = 2` (a document-level address) — carries an *arrangement* `M(d) : T ⇀ T`, a
partial function from V-positions in the document's current virtual stream to I-addresses,
the permanent keys of a content store `C : T ⇀ Val`, and a link store `L : T ⇀ Link`.

**The operation's precondition.** The entire apparatus below presupposes that `M(d)` is
*defined* — that `d` is an *allocated* document. We record this as the operation's
precondition (**W-pre**):

> `RETRIEVEDOCVSPANSET(d)` requires `d ∈ dom(M)` (equivalently, by M0 of ASN-0093,
> `Document(d) ∧ d ∈ dom(M)`: a T4-valid document-level tumbler that some K.σ
> registration event has placed into `dom(M)`).

An *allocated empty* document (`d ∈ dom(M)`, `M(d) = ∅`) legitimately yields the defined
empty span-set `⟨⟩`, whereas an *unallocated* identity (`d ∉ dom(M)`) lies outside
the operation's domain and signals failure rather than fabricating `⟨⟩`. Gregory's back end
confirms the separation operationally — an existing-but-empty document returns the empty
span-set with success, whereas a never-allocated identity fails and the back end signals
failure rather than an empty result (consultation).

We write

> `O(d) = dom(M(d))`

for the set of *occupied V-positions* of `d` (well-defined under W-pre). We partition
`O(d)` by *kind*. Each
V-position carries a subspace identifier in its first component, `subspace(v) = v₁`
(ASN-0036), and the docuverse fixes two of them: content positions carry `subspace = s_C`
and link positions carry `subspace = s_L`, with the convention `s_C = 1`, `s_L = 2`
(SubspaceConventionAxiom). For a subspace identifier `S` write

> `V_S(d) = {v ∈ O(d) : subspace(v) = S}`

for the *active V-positions of `d` in subspace `S`*. The two kinds Nelson names — text and
links — are exactly `V_{s_C}(d)` and `V_{s_L}(d)`.

We rely on these foundation facts about the shape of each `V_S(d)`:

- **S2** (functionality): each occupied V-position has a single I-address.
- **S3★** (referential integrity): a content position maps into `dom(C)`, a link position
  into `dom(L)` (ASN-0047).
- **S8-fin** (finiteness): `O(d)` is finite, hence each `V_S(d)` is finite.
- **S8a** (well-formedness): every `v ∈ O(d)` is zero-free, of depth `≥ 2`, all components
  positive.
- **S8-depth**: within one subspace, all active V-positions share a common depth `m_S ≥ 2`.
- **D-CTG★ / D-MIN★ / D-SEQ★** (per-subspace shape, ASN-0047): for each subspace `S`, when
  `V_S(d)` is non-empty it is contiguous, its minimum is the canonical `[S,1,…,1]` of depth
  `m_S`, and it forms the dense run

  > `V_S(d) = {[S,1,…,1,k] : 1 ≤ k ≤ n_S}` for some `n_S ≥ 1`.

- **CL-OWN / CL-UNIQ** (ASN-0047): a document's link-subspace arrangement holds only its
  own links, each at exactly one V-position.

We borrow the span machinery wholesale. A span `σ = (s, ℓ)` denotes the half-open interval
`⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` (T12), with `reach(σ) = s ⊕ ℓ` (ASN-0053). A span is
*well-formed* when `Pos(ℓ)` and `actionPoint(ℓ) ≤ #s`; it is *level-uniform* when
`#s = #ℓ`. The ordinal displacement `δ(n, m) = [0,…,0,n]` of length `m` (ASN-0034) is the
canonical pure depth-`m` shift, and `shift(t, n) = t ⊕ δ(n, #t)` advances `t`'s last
component by `n`. A *span-set* is a finite sequence of spans denoting the union of its
members; it is *normalized* when sorted and separated (ASN-0053). We measure the document
as a span-set, one member per kind.

---

## What the caller must be handed

We fix the *type* of the operation's result. It is a *normalized span-set* `Σ_d` of at most
two members — one per occupied subspace — not a content sequence (which would return records,
not spans) and not a pair of bare integers (the magnitudes are read off span boundaries, never
designated directly). We record this as **W0** (span-set-valued result): for an *allocated*
document `d` (W-pre), `RETRIEVEDOCVSPANSET(d)` returns a normalized span-set; for an
allocated document that is *empty in both counted subspaces* (`d ∈ dom(M)` with
`V_{s_C}(d) = V_{s_L}(d) = ∅`) it returns `⟨⟩`, the distinguished value denoting `∅` (which
is not a T12 span, since every well-formed span is non-empty — S2, ASN-0053). The caller
reads each member to learn the extent of one kind of content; the content itself, and the
identity of individual links, are the business of other operations.

---

## The extent of a single subspace

We reason first about *one* kind in isolation: what span describes the extent of subspace
`S` in document `d`? We are looking for a region that contains exactly the active positions
`V_S(d)` and nothing else — Nelson's requirement that one designate "a separated series of
items exactly, including nothing else" (4/25).

When `V_S(d) ≠ ∅`, D-SEQ★ hands us its shape directly: a dense run
`{[S,1,…,1,k] : 1 ≤ k ≤ n_S}` of depth `m_S`, with minimum `[S,1,…,1]` (D-MIN★) and
cardinality `n_S = |V_S(d)|`. Because the run is dense and contiguous, a *single* span
covers it exactly. Define the **extent span** of subspace `S`:

> `ext(d, S) = (start_S, δ(n_S, m_S))`,  where `start_S = [S,1,…,1]` of depth `m_S`.

We record `n_S = |V_S(d)|` as the **subspace extent** (W1) and `ext(d, S)` as its **span
encoding** (W2). We must show this span is legal and that it covers exactly the run.

**The extent span is well-formed.** We record **W3**: `ext(d, S)` satisfies T12. By
OrdinalDisplacement (ASN-0034), with `n_S ≥ 1` (the run is non-empty) and `m_S ≥ 1`, the
width `δ(n_S, m_S)` is a positive tumbler with `actionPoint(δ(n_S, m_S)) = m_S`; since
`#start_S = m_S`, the action point satisfies `actionPoint(δ(n_S, m_S)) = m_S ≤ #start_S`.
T12's two preconditions hold, so the span is well-formed; moreover it is level-uniform,
`#δ(n_S, m_S) = m_S = #start_S`. Its reach is, by OrdinalShift (ASN-0034),

> `reach(ext(d, S)) = start_S ⊕ δ(n_S, m_S) = shift(start_S, n_S) = [S,1,…,1,1+n_S]`,

one ordinal step past the last active position `[S,1,…,1,n_S]`, realizing the half-open
convention under which the last occupied position is included and the next is excluded.

**The extent span covers its subspace exactly.** This is the heart of the matter — Nelson's
"complete and exclusive" requirement (4/25), the two halves being completeness (account for
every position present) and exclusivity (admit nothing foreign). To state exclusivity we
must say *foreign to what*: a span's denotation `⟦ext(d, S)⟧` necessarily contains tumblers
deeper than the V-positions (whole subtrees hang below each address), so we restrict
attention to the addressable V-positions of the subspace at its working depth. Define the
*V-slice*

> `VSlice(S, m) = {t ∈ T : t₁ = S ∧ #t = m ∧ zeros(t) = 0}`

— the depth-`m`, zero-free tumblers of subspace `S`, the population from which active
V-positions are drawn (S8a). We record **W4** (ExactCoverage):

> `⟦ext(d, S)⟧ ∩ VSlice(S, m_S) = V_S(d)`.

The derivation is direct. `⟦ext(d, S)⟧ = {t : start_S ≤ t < [S,1,…,1,1+n_S]}`. Take any
`t ∈ VSlice(S, m_S)`. Such a `t` has the form `[S, t_2, …, t_{m_S}]` with all components
positive. The bounds `start_S = [S,1,…,1]` and `reach = [S,1,…,1,1+n_S]` share the common
prefix `[S,1,…,1]` of length `m_S − 1`, so by T5 (ContiguousSubtrees), applied with
`start_S ≤ t < reach`, every interior `t` extends that prefix — its first `m_S − 1`
components are pinned to `[S,1,…,1]`. The
only remaining freedom is in the last component, which the half-open bounds then pin to
`1 ≤ t_{m_S} ≤ n_S`. These are exactly the elements
`[S,1,…,1,k]` with `1 ≤ k ≤ n_S` — which is `V_S(d)` by D-SEQ★. So the span omits no active
position (completeness) and includes no inactive V-slice tumbler (exclusivity). The
load-bearing invariant here is contiguity: it is *because* D-CTG★ holds at
every reachable state — `V_S(d)` contains every V-slice tumbler lying (under T1) between its
own minimum and maximum — that a single half-open span can be exact.

**The count is read off the boundary.** Because the run is dense, `n_S` is recoverable from
the span alone: it is the last component of the width `δ(n_S, m_S)`, equivalently the gap
between the last component of `reach` and that of `start_S`. This is how a span-set
"indicates the number" (4/68) without designating a number directly (4/24): the magnitude is
implicit in the boundary, and made explicit only because the subspace is contiguous.

---

## The operation: one span per occupied subspace

We now assemble the members. Let

> `occupied(d) = {S ∈ {s_C, s_L} : V_S(d) ≠ ∅}`

be the *occupied subspaces* (W6). The operation returns the extent span of each, sorted:

> `RETRIEVEDOCVSPANSET(d) = ⟨ ext(d, S) : S ∈ occupied(d), in increasing S ⟩`,

the empty span-set `⟨⟩` when `occupied(d) = ∅`. We record this as **W7**
(OneSpanPerOccupiedSubspace): the result has exactly `|occupied(d)|` members — one per
occupied subspace, *never one per contiguous fragment and never one per individual item*.
A document with a thousand characters and three links yields two members, not a
thousand-and-three. This matches both
Nelson's "span-set indicating both the number of characters and the number of links" (4/68)
and Gregory's implementation, which emits at most one VSpec per subspace regardless of how
many crums populate it (consultation Q11, Q14, Q19).

**The result is a read-only observation.** We record **W8** (PureQuery): `Σ' = Σ`. The
operation writes nothing — no allocation, no arrangement change, no provenance. Its read-set
is narrower still: the result is a function of `dom(M(d))` (equivalently `M(d)`) and the
document identity alone. Each member is computed from `V_{s_C}(d)` and `V_{s_L}(d)`, and these
sets `V_S(d) = {v ∈ dom(M(d)) : v₁ = S}` are determined by `dom(M(d))` and the subspace
projection — the I-address *values* `M(d)(v)` are never consulted, and neither `C` nor `L` is
read to produce the span-set.

**Only the two counted subspaces appear.** Every occupied V-position of `d` lies in one of
the two counted subspaces, leaving none in a third. This is precisely S3★-aux
(SubspaceExhaustiveness, ASN-0047): `(A d, v : v ∈ dom(M(d)) : subspace(v) = s_C ∨
subspace(v) = s_L)`. We record **W9** (TwoKindsOnly):

> `O(d) = V_{s_C}(d) ⊔ V_{s_L}(d)`.

The derivation: by S8a every `v ∈ O(d)` has a well-formed first component, and S3★-aux
restricts that component to `{s_C, s_L}`, so `O(d) = {v ∈ O(d) : v₁ = s_C} ∪ {v ∈ O(d) : v₁ =
s_L} = V_{s_C}(d) ∪ V_{s_L}(d)`. The union is disjoint because `s_C ≠ s_L` (SC-NEQ), so no
`v` can satisfy both predicates. There is therefore *no third subspace* in which document
content could reside, hence no third member can ever arise in the span-set — the report is
intrinsically two-kinded.

**The result-cardinality, characterized as a weakest precondition.** The operation writes
nothing (W8), so any non-trivial postcondition a caller asserts is on the *value* it returns.
Since `RETRIEVEDOCVSPANSET` is a pure query whose result is `⟨ ext(d, S) : S ∈ occupied(d) ⟩`
(W7), its cardinality is `|occupied(d)|`, and `occupied(d)` is fixed by which of `V_{s_C}(d)`,
`V_{s_L}(d)` is non-empty (W6). Computing the weakest precondition for each result-shape
postcondition — and conjoining W-pre, since outside `dom(M)` the result is undefined rather
than `⟨⟩` — we record **W19** (ResultCardinalityWP). The empty result:

> `wp(RETRIEVEDOCVSPANSET(d), "result = ⟨⟩") ≡ d ∈ dom(M) ∧ V_{s_C}(d) = ∅ ∧ V_{s_L}(d) = ∅`.

The two-member result:

> `wp(RETRIEVEDOCVSPANSET(d), "|result| = 2") ≡ d ∈ dom(M) ∧ V_{s_C}(d) ≠ ∅ ∧ V_{s_L}(d) ≠ ∅`.

And the one-member result, characterized as an exclusive-or:

> `wp(RETRIEVEDOCVSPANSET(d), "|result| = 1") ≡ d ∈ dom(M) ∧ (V_{s_C}(d) = ∅ ⊻ V_{s_L}(d) = ∅)`.

Each equivalence is forced. The right-to-left direction reads off W6/W7: the named occupancy
pattern fixes `occupied(d)` and hence `|occupied(d)| = |result|`; for the empty case,
`occupied(d) = ∅` gives `result = ⟨⟩` directly. The left-to-right direction is the
*weakest*-precondition obligation — no strictly weaker state predicate implies the
postcondition, because `occupied(d)` is *determined* by the two emptiness bits (W6) and the
result is a total function of `occupied(d)` (W7), so any state satisfying the postcondition
*must* exhibit the named occupancy pattern. The three preconditions partition the allocated
states (`d ∈ dom(M)`) by the pair of emptiness bits — `(∅, ∅)`, exactly one empty, neither
empty — exhausting the result's three possible cardinalities.

---

## Why text and links must be reported apart

We can now answer *why* the operation returns a span-set rather than folding both kinds into
one extent — Nelson's design choice, which our formalism makes forced rather than arbitrary.

The two subspaces are not merely labelled differently; they are *disjoint subtrees of the
address space*, and no single contiguous span can cover both. Consider the denotation of
each extent span. We record **W10** (SubspaceConfinement): `(A t : t ∈ ⟦ext(d, S)⟧ : t₁ =
S)`, for `t` of any depth. The argument is two lines on the first
component. The bounds are `start_S = [S,1,…,1]` and `reach = [S,1,…,1,1+n_S]`, both with
first component `S`. Take any `t ∈ ⟦ext(d, S)⟧`, so `start_S ≤ t < reach`. If `t₁ < S`, then
by T1 the first divergence is at position `1` and `t < start_S` — contradicting `start_S ≤
t`. If `t₁ > S`, then by T1 `t > reach` — contradicting `t < reach`. Hence `t₁ = S`, for
`t` of any depth. It follows immediately that the two member spans are disjoint — **W11**
(Disjointness):

> `⟦ext(d, s_C)⟧ ∩ ⟦ext(d, s_L)⟧ = ∅`.

For any `t` in the intersection we would need `t₁ = s_C` and `t₁ = s_L` at once (W10),
impossible since `s_C ≠ s_L` (SC-NEQ, the `1 ≠ 2` of the convention). The text region and the link region therefore *cannot* be the
denotation of a single span: a span is a contiguous interval (T12), and `⟦ext(d, s_C)⟧` and
`⟦ext(d, s_L)⟧` are separated by every address between them — in particular the whole gap
from `[s_C,1,…,1,1+n_{s_C}]` up to `[s_L,1,…,1]`. To "designate the separated series exactly,
including nothing else" (4/25), one is therefore forced into a span-set of two members.

---

## Invariants across the members

We collect the constraints that must hold *among* the spans the operation returns — the
properties an alternative implementation must also satisfy for its report to be coherent.

**Uniform shape.** The result always describes the same kinds in the same order. The
candidate subspaces are the fixed pair `{s_C, s_L}` and the members are sorted by `S`, so
when both are occupied the text member precedes the link member, always. We record **W13**
(UniformShape): the result is a normalized span-set whose members occupy positions drawn
from the fixed, ordered kind-list `(s_C, s_L)`. The shape of the report is invariant across
the docuverse; only the magnitudes `n_S` differ. (That the result is already *normalized* —
sorted and separated, ASN-0053 — follows from W11: the two members are disjoint and ordered
`s_C < s_L`, with `reach(ext(d, s_C)) < start_{s_L}` by T1, so no merging is possible and the
sequence is in normal form.)

**Cross-kind independence.** The extent reported for one kind does not depend on the
population of the other. We record **W15** (Independence): `n_{s_C}(d)` is a function of
`V_{s_C}(d)` alone, and `n_{s_L}(d)` of `V_{s_L}(d)` alone; consequently an edit confined to
one subspace leaves the other subspace's reported extent unchanged. This follows because each
count is read off a *disjoint* position set: `V_S(d) = {v ∈ O(d) : v₁ = S}` is selected by
the predicate `v₁ = S`, and `s_C ≠ s_L` (SC-NEQ) makes `V_{s_C}(d)` and `V_{s_L}(d)` disjoint,
so `n_{s_C} = |V_{s_C}(d)|` and `n_{s_L} = |V_{s_L}(d)|` are computed from non-overlapping data
(W1).

**Partition of the counted content.** The members do not merely fail to overlap; together
they account for exactly the counted V-positions. We record **W16** (Partition):

> `(⊔ S : S ∈ occupied(d) : ⟦ext(d, S)⟧ ∩ VSlice(S, m_S)) = {v ∈ O(d) : v₁ ∈ {s_C, s_L}}`,

a *disjoint* union (W11 gives disjointness; W4 gives that each part is exactly `V_S(d)`; and
`O(d)` restricted to the counted subspaces is `V_{s_C}(d) ⊔ V_{s_L}(d)` by definition). No
counted position is orphaned — left outside every member — and no member claims a position
that is not active.

**The extent-content relationship.** Finally, the relationship each member bears to what a
reader would *find* on retrieving that subspace. W4 already fixes *which* V-slice tumblers the
member designates — the active positions of `S` are exactly those lying within `ext(d, S)`. The
one increment here is that each such position *carries content*. We record **W17**
(ExtentDeterminesPopulation), one S3★ step beyond W4: for each occupied `S` and each
`v ∈ ⟦ext(d, S)⟧ ∩ VSlice(S, m_S)`, `M(d)(v) ∈ dom(C)` for `S = s_C` and `M(d)(v) ∈ dom(L)` for
`S = s_L` (S3★).

**The count is faithful.** The link member's whole purpose (W0, after Nelson 4/68) is to
*indicate the number of links*, so we must bridge `n_{s_L}(d) = |V_{s_L}(d)|` to that number.
We record **W20** (FaithfulCount). Two foundation facts fix what the count measures. CL-OWN
restricts `V_{s_L}(d)` to links homed at `d` — a third party linking *into* `d`, owning its
link at another address, contributes nothing to `V_{s_L}(d)` and cannot perturb the reported
link extent. CL-UNIQ makes `M(d)` restricted to `V_{s_L}(d)` injective — each *arranged* link
occupies *exactly one* link-subspace V-position. Together they make
`M(d)|_{V_{s_L}(d)}` a bijection onto `ran(M(d)|_{s_L})`, the links *currently present in `d`'s
arrangement*, so `|V_{s_L}(d)|` counts exactly those links. We must be precise about the
counted quantity: it is the links arranged in `d`, which is a *subset* of all links homed at
`d`. The two coincide at link creation — the back end couples link allocation to
link-subspace insertion, so every freshly created home link of `d` enters `M(d)` at once
(consultation) — but the coupling is not a standing invariant: a later contraction of `d`'s
link subspace can remove a link from `M(d)` while its address and home document survive,
leaving a home link of `d` that no longer contributes to `V_{s_L}(d)`. No foundation
invariant forces every home link into `M(d)`, so `n_{s_L}(d)` faithfully counts the *arranged*
links, which is exactly what a per-subspace extent query of `M(d)` should report. The content
side carries the analogous guarantee through different premises: each content V-position
carries exactly one I-address (S2) drawn from `dom(C)` (S3★), so
`n_{s_C}(d) = |V_{s_C}(d)|` is the number of arranged content positions — faithful by
functionality and referential integrity.

---

## Comparing reports across documents

The uniform shape of a single report (W13) has a cross-document consequence: the *full
per-kind count vector* `(n_{s_C}(d), n_{s_L}(d))` is recoverable from a report that may omit
members. We record **W14** (Comparability). Because the operation emits only occupied
subspaces (W7), the result is a *subsequence* of the kind-list, so kind must be read from each
member's own subspace identifier `start₁ = S` (W2/W10), not from its list position. An absent
member is sound to read as `n_S(d) = 0`: by W6/W7 the operation omits kind `S` exactly when
`V_S(d) = ∅`, which is exactly when `n_S(d) = |V_S(d)| = 0` (W1). Hence for any two allocated
documents `d₁, d₂` the per-kind comparison `n_S(d₁)` versus `n_S(d₂)` is well-defined over the
*entire* fixed kind-list — text-extent to text-extent and link-extent to link-extent —
*provided both reports range over the same kind-list*.

---

## A worked instance

We instantiate the operation on a concrete document to check the key postconditions against
specific tumblers. Let `d` hold five characters of text and two home links, both subspaces at
the minimal working depth `m_{s_C} = m_{s_L} = 2` — the degenerate case in which the canonical
`[S,1,…,1]` form collapses to `[S,1]`, because the inner `1,…,1` segment has length
`m_S − 2 = 0`. By D-SEQ★ the active positions are

> `V_{s_C}(d) = {[1,1], [1,2], [1,3], [1,4], [1,5]}`,  `n_{s_C} = 5`,
> `V_{s_L}(d) = {[2,1], [2,2]}`,  `n_{s_L} = 2`

(recall `s_C = 1`, `s_L = 2`). The extent spans (W2) are

> `ext(d, s_C) = ([1,1], δ(5,2)) = ([1,1], [0,5])`,
> `ext(d, s_L) = ([2,1], δ(2,2)) = ([2,1], [0,2])`,

so the operation returns the normalized span-set

> `RETRIEVEDOCVSPANSET(d) = ⟨ ([1,1], δ(5,2)), ([2,1], δ(2,2)) ⟩`.

**W3 (well-formed).** For `ext(d, s_C)`: the width `δ(5,2) = [0,5]` is positive (`Pos`), its
action point is its last position `2`, and `#start = #[1,1] = 2`, so `actionPoint ≤ #start`;
level-uniform since `#[0,5] = 2 = #[1,1]`. Its reach is `[1,1] ⊕ [0,5] = shift([1,1], 5) =
[1,6] = [s_C,1+n_{s_C}]` (empty interior segment at `m_S = 2`). Identically `ext(d, s_L)` has
reach `[2,3] = [s_L,1+n_{s_L}]`. Both satisfy T12.

**W4 (exact coverage).** `⟦ext(d, s_C)⟧ = {t : [1,1] ≤ t < [1,6]}`. Intersecting with
`VSlice(s_C, 2) = {[1,j] : j ≥ 1}` pins the depth-2, prefix-`[1,·]` tumblers with last
component in `1..5`, i.e. `{[1,1],…,[1,5]} = V_{s_C}(d)`. Likewise `⟦ext(d, s_L)⟧ ∩
VSlice(s_L, 2) = {[2,1],[2,2]} = V_{s_L}(d)`. Neither span omits an active position nor admits
an inactive V-slice tumbler.

**W11 (disjointness).** Every `t ∈ ⟦ext(d, s_C)⟧` has `t₁ = 1`; every `t ∈ ⟦ext(d, s_L)⟧` has
`t₁ = 2` (W10). Since `1 ≠ 2`, `⟦ext(d, s_C)⟧ ∩ ⟦ext(d, s_L)⟧ = ∅`.

**W13 (uniform shape).** The members are listed in increasing `S`, text (`s_C = 1`) before
link (`s_L = 2`). They are separated: `reach(ext(d, s_C)) = [1,6] < [2,1] = start_{s_L}` by
T1 (first component `1 < 2`), so no merge is possible and the sequence is already normalized.

**W16 (partition).** The disjoint union of the two V-slice intersections is `{[1,1],…,[1,5]} ⊔
{[2,1],[2,2]}`, which is exactly `{v ∈ O(d) : v₁ ∈ {s_C, s_L}} = V_{s_C}(d) ⊔ V_{s_L}(d) =
O(d)` (the last equality by W9). No counted position is orphaned and no member claims an
inactive position.

The degenerate `m_S = 2` instance shows the canonical machinery surviving the collapse: with
no interior `1`'s, `start_S = [S,1]` and the count `n_S` lives entirely in the last component
of the width.

**A one-member instance.** The single-occupied-subspace boundary — the default state of a
freshly populated document with text but no links yet — sits between `⟨⟩` (W0) and the
two-member report. Let `d'` hold three characters of text and *no* links, depth
`m_{s_C} = 2`. By D-SEQ★,

> `V_{s_C}(d') = {[1,1], [1,2], [1,3]}`,  `n_{s_C} = 3`,
> `V_{s_L}(d') = ∅`,  `n_{s_L} = 0`.

The link subspace is empty, so `occupied(d') = {s_C}` (W6) and `|occupied(d')| = 1` (W7): the
operation emits exactly one member and *no* link member. The single extent span (W2) is
`ext(d', s_C) = ([1,1], δ(3,2)) = ([1,1], [0,3])`, so

> `RETRIEVEDOCVSPANSET(d') = ⟨ ([1,1], δ(3,2)) ⟩`.

**W3 (well-formed).** `δ(3,2) = [0,3]` is positive (`Pos`), its action point is its last
position `2 = #[1,1]`, and it is level-uniform (`#[0,3] = 2 = #[1,1]`); its reach is
`[1,1] ⊕ [0,3] = [1,4] = [s_C,1+n_{s_C}]` (empty interior segment at `m_S = 2`). T12 holds.

**W4 (exact coverage).** `⟦ext(d', s_C)⟧ = {t : [1,1] ≤ t < [1,4]}`; intersecting with
`VSlice(s_C, 2) = {[1,j] : j ≥ 1}` pins the last component to `1..3`, giving
`{[1,1],[1,2],[1,3]} = V_{s_C}(d')` — neither omitting an active position nor admitting an
inactive one.

**W7 / W13 (single-member normal form).** With one occupied subspace the result is the
singleton `⟨ext(d', s_C)⟩`, trivially normalized: a one-member sequence is sorted and
separated, so no merge is possible.

**W14 (absent member, zero count).** The empty link subspace contributes *no* member to the
report, yet `n_{s_L}(d') = |V_{s_L}(d')| = 0` (W1) — a defined zero count alongside an emitted
member count of one. This verifies W14 against `d'`: iterating the fixed kind-list
`(s_C, s_L)` and reading each member by its subspace identifier `start₁ = S`, the sole member
`ext(d', s_C)` (with `start₁ = 1 = s_C`) supplies `s_C` present with count `3`, while `s_L` is
absent and reads as count `0`.

**A depth-`3` instance: prefix-confinement is non-vacuous.** Both instances above fix
`m_S = 2`, where the canonical prefix `[S,1,…,1]` collapses to length `m_S − 1 = 1`. There the
only same-depth V-slice tumblers in play are `[S,j]` (excluded when `j > n_S` by the
last-component bound) and `[S',·]` with `S' ≠ S` (excluded by the first component). The step
that W4 and W10 actually turn on — T5's confinement of every interior tumbler to the prefix
`[S,1,…,1]`, which rules out tumblers diverging at an *interior* position while still carrying
an admissible last component — is vacuous at `m_S = 2`. We exercise it at `m_S = 3`, where the
prefix `[S,1]` has length `m_S − 1 = 2` and an interior position genuinely exists between the
subspace identifier and the last component.

Let `V_S(d)` have depth `m_S = 3` with `n_S = 2`, so by D-SEQ★

> `V_S(d) = {[S,1,1], [S,1,2]}`,  `start_S = [S,1,1]`,

and the extent span (W2) is

> `ext(d, S) = ([S,1,1], δ(2,3)) = ([S,1,1], [0,0,2])`,  `reach(ext(d, S)) = [S,1,1] ⊕ [0,0,2] = [S,1,3]`.

The half-open denotation is `⟦ext(d, S)⟧ = {t : [S,1,1] ≤ t < [S,1,3]}`. Now take the V-slice
tumbler `[S,2,1] ∈ VSlice(S, 3)` — a depth-`3`, zero-free tumbler of subspace `S` that agrees
with the active positions on the first component but *diverges at the interior position `2`*,
and whose last component `1` nonetheless lies in the admissible range `1..n_S`. This is exactly
the candidate that the last-component bound alone would *not* reject. By T1 the first
disagreement decides the order: comparing `[S,2,1]` against `reach = [S,1,3]` at position `2`
gives `2 > 1`, so `[S,2,1] > [S,1,3]`, placing it past the upper bound. It is therefore *not*
in `⟦ext(d, S)⟧`. Equivalently, T5 applied to the common prefix `[S,1]` (length `m_S − 1 = 2`)
shared by `start_S` and `reach` confines every interior tumbler to that prefix, and `[S,2,1]`
fails to extend `[S,1]` — so it cannot slip in despite its admissible last component. The
remaining V-slice tumblers within the half-open bounds are exactly `[S,1,1]` and `[S,1,2]`,
their last components pinned to `1..2`, giving

> `⟦ext(d, S)⟧ ∩ VSlice(S, 3) = {[S,1,1], [S,1,2]} = V_S(d)`,

which confirms W4 in the regime where prefix-confinement does the actual work. The
`m_S = 2` instances above check W4 only where the prefix is trivial; this instance checks it
where an off-prefix, admissible-last-component tumbler must be — and is — excluded.

---

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| W0 | `RETRIEVEDOCVSPANSET(d)` returns a normalized span-set (≤ 2 members), or `⟨⟩` when both counted subspaces are empty; never a content sequence or a cardinality | introduced |
| W1 | `n_S(d) = |V_S(d)|` is the extent of subspace `S` in `d` | introduced |
| W2 | *for `S ∈ occupied(d)` (`V_S(d) ≠ ∅`)*, `ext(d, S) = ([S,1,…,1], δ(n_S, m_S))` is the extent span encoding `n_S` | introduced |
| W3 | *for `S ∈ occupied(d)`*, `ext(d, S)` is a well-formed, level-uniform T12 span with `reach = [S,1,…,1,1+n_S]` | introduced |
| W4 | ExactCoverage — *for `S ∈ occupied(d)`*, `⟦ext(d, S)⟧ ∩ VSlice(S, m_S) = V_S(d)` (complete and exclusive) | introduced |
| W6 | `occupied(d) = {S ∈ {s_C, s_L} : V_S(d) ≠ ∅}` | introduced |
| W7 | OneSpanPerOccupiedSubspace — result has exactly `|occupied(d)|` members, one per kind, not per fragment or item | introduced |
| W8 | PureQuery — `Σ' = Σ`; the operation writes nothing and its result is a function of `dom(M(d))` alone (`C`, `L`, and the I-address values `M(d)(v)` are not read), so it changes only when `M(d)` changes | introduced |
| W9 | TwoKindsOnly — `O(d) = V_{s_C}(d) ⊔ V_{s_L}(d)` (derived from S3★-aux); no third subspace holds content, so no third member can arise | introduced |
| W10 | SubspaceConfinement — `(A t : t ∈ ⟦ext(d, S)⟧ : t₁ = S)` | introduced |
| W11 | Disjointness — `⟦ext(d, s_C)⟧ ∩ ⟦ext(d, s_L)⟧ = ∅` | introduced |
| W13 | UniformShape — result is normalized, members drawn from the fixed ordered kind-list `(s_C, s_L)` | introduced |
| W14 | Comparability — per-kind comparison `n_S(d₁)` vs `n_S(d₂)` is well-defined across documents sharing a kind-list; an absent member reads as `n_S = 0` | introduced |
| W15 | Independence — `n_{s_C}` depends only on `V_{s_C}(d)`, `n_{s_L}` only on `V_{s_L}(d)`; subspace edits do not cross | introduced |
| W16 | Partition — the members disjointly cover exactly the counted active V-positions; no orphan, no phantom | introduced |
| W17 | ExtentDeterminesPopulation — each V-slice position within `ext(d, S)` carries content (`M(d)(v) ∈ dom(C)`/`dom(L)`, S3★); one step beyond W4's coverage equality | introduced |
| W19 | ResultCardinalityWP — `wp(·, "result = ⟨⟩") ≡ d ∈ dom(M) ∧ V_{s_C}(d) = ∅ ∧ V_{s_L}(d) = ∅`; `wp(·, "|result| = 2") ≡ d ∈ dom(M) ∧ V_{s_C}(d) ≠ ∅ ∧ V_{s_L}(d) ≠ ∅`; `wp(·, "|result| = 1") ≡ d ∈ dom(M) ∧ (V_{s_C}(d) = ∅ ⊻ V_{s_L}(d) = ∅)` | introduced |
| W20 | FaithfulCount — `n_{s_L} = |V_{s_L}(d)|` counts the links *arranged* in `d` exactly (CL-OWN restricts to home links, CL-UNIQ gives the bijection onto `ran(M(d)|_{s_L})`, a subset of all links homed at `d`); `n_{s_C} = |V_{s_C}(d)|` counts arranged content positions by functionality and referential integrity (S2/S3★) | introduced |

---

## Open Questions

Should a foundation extension ever relax D-CTG★ (on which W4's single-span exactness rests), what must the operation guarantee — is it obligated to emit the fragmented span-set, or may it report Gregory's single bounding span and leave faithful interpretation of interior gaps to the caller?

What permanence must the per-subspace extent report carry across a version fork that shares content with its ancestor?

What must the operation guarantee about a subspace's reported extent when some of its content is transcluded from a source document that is itself edited?

Must the per-subspace extents reported by the operation be derivable from, and consistent with, any single overall extent the document also exposes, and what would a disagreement signify?

Should the subspace convention be extended beyond text and links, what must the operation guarantee so that the kind-list remains fixed and the report stays comparable across documents of different vintages?
