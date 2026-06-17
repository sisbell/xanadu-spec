> **ASN-0121 · The FINDLINKSFROMTOTHREE Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md), [ASN-0127 · Content-Region Link Query](../foundation/ASN-0127-content-region-link-query.md)  
> [Condensed statements →](ASN-0121-findlinksfromtothree-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0121: The FINDLINKSFROMTOTHREE Operation

*2026-06-08*

We are asked to characterise the operation that, given a *description of links* phrased
as four bounding sets, returns the links that fit the description. Nelson names it
`FINDLINKSFROMTOTHREE`:

> "This returns a list of all links which are (1) in `<home set>`, (2) from all or any
> part of `<from set>`, and (3) to all or any part of `<to set>` and `<three set>`."
> (4/69)

The four sets are the **home-set** (where the links reside), the **from-set** (what
their first endset references), the **to-set** (what their second endset references),
and the **three-set** (their type or connector endset). The question we must answer is
not *how* a back end finds these links — that is mechanism — but *what* the answer must
be: for an arbitrary link to belong in the result, what must hold of it, and what must
the result as a whole guarantee against the body of links as it stands at the moment of
inquiry. We want a specification an alternative implementation would also have to meet.

We write the system state as ASN-0047's five-tuple `Σ = (Σ.C, Σ.L, Σ.E, Σ.M, Σ.R)` —
the content store, the link store, the entity set, the family of document arrangements,
and the provenance relation. This is the state the transition vocabulary `→` (below)
operates on; the query itself reads only the link store `Σ.L` (FL-LOC, below). We use
`coverage(e)` (ASN-0043) for the set of tumbler addresses an endset
references, `home(a)` (ASN-0043) for the document-level prefix at which a link address
`a` resides, and the total order and span machinery of ASN-0034 throughout.

## What is being matched

A link `a ∈ dom(Σ.L)` carries a value `Σ.L(a) = (e₁, e₂, …)` of at least three endsets
(L3). The first three slots are, by convention, the *from-endset* `e₁`, the *to-endset*
`e₂`, and the *type-endset* `e₃`. L3 permits arity `N ≥ 3`, so a link may carry further
endsets `e₄, …, eₙ` beyond the third — the n-set form Nelson calls for (4/79). This
operation — FINDLINKS*FROMTOTHREE* — constrains exactly the first three slots: the
satisfaction rule `sat` (below) tests `e₁, e₂, e₃` and leaves any higher slots
`e₄ … eₙ` unconstrained, so a higher-arity link is matched on its first three endsets
alone and remains in the result space. Each endset references a set of I-addresses, its
coverage. The link also resides somewhere: `home(a)` is a document-level tumbler,
extracted from the *address* `a` by field projection, **not** from the endsets. These
two facts — what a link *connects* (its endset coverages) and where it *lives* (its
home) — are the raw material the request will constrain.

A request is a four-tuple

  `q = (H, F, G, Θ) ∈ (Endset ∪ {∗})⁴`,

where each component is either an *endset* (ASN-0043's `Endset = 𝒫_fin(Span)`) or the
distinguished *wildcard* `∗` (Nelson's NOSPECS — "no specification"). By convention the
home-component `H` is rooted at organizational prefixes (node-, account-, or
document-level addresses) and the three endset-components `F, G, Θ` at element-level
I-addresses, though the grammar enforces no rooting: `athome` (below) is plain coverage
membership for whatever endset `H` is — `home(a)` itself is always the document-level
field projection `N(a).0.U(a).0.D(a)`, and a node- or account-rooted span tests its
membership in the broader subtree it denotes. Every component thus denotes, through
`coverage` (ASN-0043), a set of tumbler addresses — we call `q` an *I-address request* —
and the wildcard denotes "no constraint."

We must say precisely what it is for a link to satisfy *one* component. Nelson's rule
is sharp:

> "A link satisfies a search request if one span of each endset satisfies a
> corresponding part of the request." (4/58)

"One span … satisfies a corresponding part" is an existential — it suffices that the
endset and the request set share a single address. We capture this as a *touch*
relation between an endset `e` and a request set `r`:

  `touch(e, r) ≡ coverage(e) ∩ coverage(r) ≠ ∅`.

`touch(e, r)` holds exactly when some address lies in both coverages — when *one span*
of `e` covers an address *also* covered by `r`. The endset need not match `r` in its
entirety; a partial, single-span overlap is enough. This is the disjunction that lives
*inside* a slot — "from all or any part of" the requested set.

The corresponding residence test, for a link `a` and a home-set `H`, asks only that the
link's residence fall in the requested region:

  `athome(a, H) ≡ home(a) ∈ coverage(H)`.

Because a *unit-depth* prefix-rooted home span `(p, δ(1, #p))` denotes the subtree
`{t : p ≼ t}` (PrefixSpanCoverage, ASN-0043 — whose precondition is exactly this
unit-depth displacement) — order-convex/contiguous under T1 (T5, ASN-0034) — `H` may bound
residence at the granularity of a node, an account, or a single document, and `athome`
tests membership of `home(a)` against that subtree. A wider home span bounds residence to
an order-convex *sub-range* of a subtree (T12, ASN-0034) rather than the whole of it;
`athome` is defined uniformly as coverage membership in either case, so nothing below
depends on `H` being unit-depth — only the subtree *reading* of the residence bound does.
Root level interacts with span width the same way: a unit-depth span rooted at an
*element-level* `p` (`zeros(p) = 3`) covers the subtree of `p`, which contains no
document-level tumbler, so against it `athome` is uniformly `false`; but a *wide*
element-rooted span may straddle a length boundary and so cover a document-level tumbler
— e.g. `p = [1,0,1,0,1,0,1,1]` with `ℓ = [0,0,0,0,1,1,1,1]` (T12-well-formed) gives
`p ⊕ ℓ = [1,0,1,0,2,1,1,1]`, and the document tumbler `[1,0,1,0,2]` lies in
`coverage((p, ℓ))` — so `athome(a, H)` can hold for a link homed at that document. The
traces below exercise only unit-depth spans rooted at organizational prefixes, where the
subtree reading holds.

Both tests must be *decidable* if the operation under specification is to be a realisable
query and not merely a mathematically defined set, and they are — by exactly the argument
that discharged the analogous concern in the foundation (ASN-0086
CoverageEqualityDecidable).

**FL-DEC (decidability of the component tests).** For any two endsets `e, r ∈ Endset`,
`touch(e, r)` is decidable using only T2 comparisons and TumblerAdd; and for any link
address `a` and endset `H`, `athome(a, H)` is decidable likewise. *Proof.* By
`Endset = 𝒫_fin(Span)` (ASN-0043), `coverage(e)` and `coverage(r)` are finite unions of
half-open T1-intervals `[s, s ⊕ ℓ)` (T12, ASN-0034). Run the cell-decomposition procedure
of ASN-0086's CoverageEqualityDecidable on `e ∪ r`: each coverage is constant on every
cell between consecutive sorted span endpoints, so `coverage(e) ∩ coverage(r) ≠ ∅` iff
the representative of some nonempty cell lies in both — the same finitely many T2 tests,
run for intersection-nonemptiness rather than coverage equality. The home test
`athome(a, H)` is a single point `home(a)` against a finite interval union — the T2
span-membership test of ASN-0086's ActiveSubset. ∎

## The satisfaction rule: the AND of the ORs

A wildcard component imposes nothing; a span-set component imposes its touch- or
residence-test. We lift each component to a per-link predicate:

  `lift(e, ∗) ≡ true`,    `lift(e, r) ≡ touch(e, r)`   for `r ≠ ∗`,
  `liftH(a, ∗) ≡ true`,   `liftH(a, H) ≡ athome(a, H)`  for `H ≠ ∗`.

The full satisfaction predicate conjoins the four lifted components:

  `sat(a, q, Σ) ≡ liftH(a, H) ∧ lift(Σ.L(a).e₁, F) ∧ lift(Σ.L(a).e₂, G) ∧ lift(Σ.L(a).e₃, Θ)`.

This is the structure Nelson calls "the AND of the ORs." *Within* each constrained slot
the test is a disjunction — one overlapping address is enough (the `≠ ∅` in `touch`).
*Across* the four slots the tests are conjoined — a link qualifies only if it resides in
the home-set **and** its from-endset touches the from-set **and** its to-endset touches
the to-set **and** its type-endset touches the three-set. Matching any single criterion
alone is insufficient; the returned link is the *intersection* of the four constraints,
each individually satisfiable by a partial match. Gregory's back end realises exactly
this conjunction: each endset slot is queried independently against its own subspace of
the link index, and the three resulting link-sets are intersected
(`intersectlinksets`), a link surviving only if it appears in every non-wildcard slot's
set — the AND — while within a slot any single span overlap admits it — the OR
(consultation Q11, Q15).

## The answer is forced

To speak of the body of links "as it stands at the moment of inquiry" we must
distinguish links that are *currently addressable* from those that have been withdrawn.
Nelson's retracted links are "not currently addressable" (4/9). ASN-0086 defines

  `nullified(Σ) = { a ∈ dom(Σ.L) : (E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G')) }`

— the link addresses targeted by a retraction tuple in the retraction relation `L_R^Σ`;
its non-decrease across the transition vocabulary is derived below, once `→` is fixed. The
currently addressable links are

  `addressable(Σ) = dom(Σ.L) \ nullified(Σ)`

— a definition this ASN introduces: ASN-0086 supplies `nullified` (and the per-type
active subsets `A_K^Σ`), but the store-wide, request-independent complement is named
here.

Several claims below quantify over transitions `Σ → Σ'` and over reachable `Σ →* Σ'`, so
we must say what relation `→` ranges over. We take `→` to be ASN-0047's atomic transition
vocabulary — the allocation operations K.α (content) and K.λ (link), document and entity
registration K.δ, the arrangement-editing operations K.μ⁺, K.μ⁺_L, K.μ⁻ (extension,
contraction), and provenance recording K.ρ — together with the named composite K.μ~
(reordering), which ASN-0047 defines as a K.μ⁻ + K.μ⁺ composition, not an atomic step;
`Σ →* Σ'` is the reflexive-transitive (reachability) closure of `→`. Two monotonicity
facts about this *whole* vocabulary underwrite the permanence claims, and we record them
once here. First, `dom(Σ.L)` is non-decreasing across `→`: every operation other than
K.λ preserves the link store outright — `dom(Σ'.L) = dom(Σ.L)` with per-link value
equality (F-PRES, PublishedFramePreservation, ASN-0127, covering K.α, K.δ, K.μ⁺, K.μ⁺_L,
K.μ⁻, K.ρ and the composite K.μ~) — and K.λ only extends it (L12a); ASN-0098's
StoreMonotonicity★ lifts this to `dom(Σ.L) ⊆ dom(Σ'.L)` across `→*`. Second, `nullified`
is non-decreasing across `→`. The argument is structural: `nullified(Σ)` is a function of
`Σ.L` *alone* — it is defined through the retraction relation `L_R^Σ`, which is itself
determined by `Σ.L`: a projection of the arity-3 slice of the link store, selected from
`dom(Σ.L)` by the arity-3 and slot-3 coverage tests on stored values — so every F-PRES
step, holding `Σ'.L = Σ.L`, leaves both
`L_R^Σ` and `nullified(Σ)` unchanged. Across the one link-store-changing operation K.λ,
R6a (ASN-0086, RetractionStability:
`a ∈ nullified(Σ) ⟹ a ∈ nullified(Σ')`) supplies monotonicity. So `nullified` is constant
across every non-K.λ step (F-PRES) and monotone (R6a) across K.λ, hence non-decreasing
across all of `→` and, by induction, across `→*`.

Now we may derive, rather than stipulate, the answer set. Demand of any candidate answer
`R` two things. *Soundness*: `(A a : a ∈ R : a ∈ addressable(Σ) ∧ sat(a, q, Σ))` —
nothing returned is withdrawn or fails a criterion. *Completeness*:
`(A a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) : a ∈ R)` — nothing qualifying is omitted. The
addressability conjunct of soundness is essential and not implied by the matching rule:
retraction is *not* one of the four criteria, so a nullified link `a` with `sat(a, q, Σ)`
true still satisfies every criterion. Were soundness to demand only `sat(a, q, Σ)`, both
`R_min = { a ∈ addressable(Σ) : sat(a, q, Σ) }` and the larger
`R_max = { a ∈ dom(Σ.L) : sat(a, q, Σ) }` — which retains nullified-but-satisfying links —
would meet the two demands, and the answer would not be forced: the residual freedom is
exactly whether to return retracted-but-satisfying links, precisely the freedom Nelson's
"not currently addressable" (4/9) closes. The addressability conjunct removes that slack.
With it, the predicate soundness *permits* into `R` for any link is
`a ∈ addressable(Σ) ∧ sat(a, q, Σ)`; the predicate completeness *forces* into `R` is the
same. The two demands meet with no slack between them, leaving no design freedom:

  `findlinks_FTT(q, Σ) = { a ∈ addressable(Σ) : sat(a, q, Σ) }`.   **(FL-DEF)**

*Notation — the FTT subscript.* The bare name `findlinks` is already taken: ASN-0127's
existence primitive F-FIND defines `findlinks(I, Σ) ≡ {a ∈ dom(Σ.L) : matches(a, I, Σ)}`,
and the two operations disagree in substance, not merely in signature, so we do not
overload the symbol. The disagreements are three. (i) *Slot regime* — F-FIND's `matches`
is slot-agnostic, an existential over every slot `1 ≤ i ≤ |Σ.L(a)|` (type and higher
slots included) against a single I-address set, whereas `sat` is positional over the
first three slots, each tested against its own request component. (ii) *Range* —
F-FIND's comprehension runs over the full `dom(Σ.L)` with no nullification filter,
whereas FL-DEF runs over `addressable(Σ)`. (iii) *Dynamics*, in consequence — F-FIND is
monotone across `→*` (E-MONO, ASN-0127), whereas `findlinks_FTT` shrinks under retraction
(FL-WP(c), FL-RET below). Neither operation is a restriction of the other: a link
touching the request only through a higher slot `e₄` is found by F-FIND yet fails `sat`,
and a nullified link satisfying all four criteria is found by F-FIND yet excluded here.
The subscript follows ASN-0127's own variant pattern (`findlinks_V`); the second phase of
`findlinks_V` (F-V) remains the foundation's bare `findlinks`, not the present operation.

*Corollary (of FL-DEC): the answer is computable.* `sat(a, q, Σ)` is decidable per link:
it is a conjunction of four tests, each either a wildcard's `true` or an instance of
FL-DEC — `touch` on the three endset slots, `athome` on the home slot. The
*addressability filter* over which FL-DEF ranges is computable by exactly ASN-0086's
ActiveSubset argument: the retraction relation `L_R^Σ` is selected from the finite
`dom(Σ.L)` by CoverageEqualityDecidable (the slot-3 type-coverage test against the
retraction coverage class), and `nullified(Σ)` is decided by T2 span-membership against
the finitely many retraction to-coverages, so `addressable(Σ) = dom(Σ.L) \ nullified(Σ)`
is enumerable. With both the filter and `sat` decidable, and
`findlinks_FTT(q, Σ) ⊆ dom(Σ.L)` finite by L-fin (`|dom(Σ.L)| < ∞`, ASN-0093), the result is
computed by deciding `sat` over the finitely many addressable links. ∎

**FL-LOC (link-store locality).** For fixed `q`, `findlinks_FTT(q, Σ)` is a function of
`Σ.L` alone. *Proof.* `nullified` is a function of `Σ.L`, as established above, hence so
is `addressable(Σ) = dom(Σ.L) \ nullified(Σ)`; and `sat(a, q, Σ)` reads only the stored
value `Σ.L(a)`, the address projection `home(a)`, and the fixed `q`. No clause consults
`Σ.C`, `Σ.M`, `Σ.E`, or `Σ.R`. ∎ The query writes nothing: its frame is the whole of
`Σ` — content store, arrangements, and link store are left unchanged.

**FL-SND (soundness).** `(A a : a ∈ findlinks_FTT(q, Σ) : a ∈ addressable(Σ) ∧ sat(a, q, Σ))`.
No returned link is withdrawn, and none fails any of the four criteria. In contrapositive
form: a nullified link is not returned even when every criterion holds, and a link with any
constrained slot *wholly disjoint* from the request — `coverage(Σ.L(a).eᵢ) ∩ coverage(Rᵢ) = ∅`
for a constrained `Rᵢ`, or `home(a) ∉ coverage(H)` for a constrained `H` — is
not returned. There are no false positives.

**FL-CMP (completeness).** `(A a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) : a ∈ findlinks_FTT(q, Σ))`.
Every currently addressable link meeting all four criteria is returned; none is silently
omitted. Together with FL-SND, the result is *exactly* the satisfying subset of
`addressable(Σ)` — read at the inquiry state `Σ`, a *current snapshot* of the store: a
newly created addressable matching link enters the answer, and a nullified link leaves
it.

## Non-impedance: junk links do not obstruct

Nelson's most emphatic claim about link search is a scaling guarantee:

> "THE QUANTITY OF LINKS NOT SATISFYING A REQUEST DOES NOT IN PRINCIPLE IMPEDE SEARCH ON
> OTHERS." (4/60)

We can state its abstract content precisely. `sat` is decided per link, independently of
every other link in the store: a link's match status is a function of its own value, its
own address, and the fixed request. Answer membership, however, conjoins a second test —
addressability — and that conjunct is not per-link: `nullified(Σ)` is computed from
retraction tuples stored at *other* addresses, so a non-matching link of retraction kind
can grow it and thereby remove an existing match from the answer (exactly the exit route
FL-WP(c) computes below). Insensitivity therefore holds for additions that nullify no
existing link — which is precisely what the lemma's first hypothesis demands.

**FL-JUNK (non-impedance).** Let `Σ →* Σ'` be any reachable sequence that retracts no
*existing* link and whose added links — the set `dom(Σ'.L) \ dom(Σ.L)`, of arbitrary
size — all fail the request: `nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)` and
`(A a : a ∈ dom(Σ'.L) \ dom(Σ.L) : ¬ sat(a, q, Σ'))` (the inclusion
`dom(Σ.L) ⊆ dom(Σ'.L)` holds across every `→*` by the monotonicity recorded above, and
`nullified(Σ) ⊆ nullified(Σ') ∩ dom(Σ.L)` likewise — `nullified` is non-decreasing and
`nullified(Σ) ⊆ dom(Σ.L)` by definition — so the first hypothesis's substantive content
is its `⊆` half: no link of `dom(Σ.L)` becomes nullified across the sequence). Then
`findlinks_FTT(q, Σ') = findlinks_FTT(q, Σ)`. The body of irrelevant links, however vast,
neither enlarges the answer nor displaces a qualifying link from it. We deliberately do
*not* demand the stronger `nullified(Σ') = nullified(Σ)`: an added junk link may be
*born-nullified* — covered at allocation by a standing retraction tuple over a ghost
address, exactly the FL-WP(a) hazard exercised in Trace 7(a) — and a sequence adding such
a link retracts nothing in any operational sense, yet grows `nullified(Σ')` by the new
address, so the stronger hypothesis would silently exclude it. Nelson's claim is about
arbitrary quantities of junk; the weaker hypothesis covers them all.

The proof rests on value persistence across the closure: for every `a ∈ dom(Σ.L)`,
`Σ'.L(a) = Σ.L(a)` — F-PRES on every non-K.λ step and L12 across K.λ, packaged across
`→*` as LP13 (ASN-0098, UnconditionalLinkPersistence) — and `home(a)` is a projection of
the fixed address `a`, so `sat(a, q, ·)` is constant across the sequence; with
`nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)`, an existing `a ∈ dom(Σ.L)` has
`a ∈ nullified(Σ') ⟺ a ∈ nullified(Σ)`, so both membership conjuncts of every existing
link are unchanged — the hypothesis is consumed on existing links only, which is why
restricting it to `dom(Σ.L)` costs the proof nothing. The added links fail `q` by
hypothesis and so enter neither answer (a born-nullified addition is excluded twice over,
by its failed `sat` and by addressability). Hence the satisfying addressable set is
unchanged.

## Residence and endpoints are orthogonal axes

The four sets fall into two kinds, serving different purposes. The home-set bounds
*residence*; the three endset-sets bound *endpoints*. Nelson keeps them separate by
design:

> "A link need not point anywhere in its home document. Its home document indicates who
> owns it, and not what it points to." (4/12)

This separation is visible directly in `sat`. The residence conjunct `liftH(a, H)`
depends only on `home(a)`, which is the field projection `N(a).0.U(a).0.D(a)` of the
*address* `a` — the endset values never enter it. The three endpoint conjuncts depend
only on `Σ.L(a)`'s endsets — the address-as-residence never enters them.

**FL-RES (residence–endpoint independence).** The home criterion is a function of the
link address alone; the from/to/type criteria are functions of the link value alone. The
four constraints are therefore *independent* slots of the request: residence may be
constrained without constraining endpoints, and conversely. In particular, with
`F = G = Θ = ∗` the result is every addressable link residing in `H`, irrespective of
what it connects; with `H = ∗` the result is every addressable link whose endpoints
match, irrespective of where it lives.

The independence is what makes link discovery powerful. Because residence is a separate
axis, one may ask for *all* links between two passages "regardless of who made them"
(4/63) by leaving the home-set unconstrained, and one may equally ask for *all* links
owned within a given document by leaving the endpoints unconstrained. Were residence
conflated with the endpoints, one could only find links one already owned. Gregory's
retrieval path confirms the independence operationally: link end-sets are read from the
link's own structure with no consultation of the home document's residence record, and
with the open-status of the home document explicitly bypassed (consultation Q17).

We note an implementation divergence worth recording, since it sharpens what the abstract
claim demands. Gregory's back end currently *ignores* the home-set entirely: a dead-code
guard (`TRUE||!homeset`) replaces the caller's residence bound with a fixed, effectively
universal range, so every search is global in the residence axis (consultation Q12). The
abstract operation requires `liftH` to bound results by residence; the implementation
realises only the `H = ∗` case. An alternative implementation must restore the residence
constraint to meet FL-RES.

## Directionality is positional, not symmetric

A link "is typically directional. Thus it has a from-set, the bytes the link is 'from,'
and a to-set, the bytes the link is 'to'" (4/42). Discovery must respect this asymmetry,
and `sat` does: the from-component `F` is lifted against `e₁` *only*, and the
to-component `G` against `e₂` *only*. The two are never pooled.

**FL-DIR (positional directionality).** The from-criterion tests `Σ.L(a).e₁` and the
to-criterion tests `Σ.L(a).e₂`; the slots are matched by position, not symmetrically.
The asymmetry is observable, and we exhibit an explicit witness. Take two distinct
content I-addresses
`x = [1,0,1,0,1,0,1,5]` and `y = [1,0,1,0,1,0,1,9]` (both element-level, `zeros = 3`,
text subspace `s_C = 1`, differing only in the last component), and the unit-depth request
endsets `X = {(x, δ(1,#x))}` and `Y = {(y, δ(1,#y))}`. By PrefixSpanCoverage (ASN-0043),
`coverage(X) = {t : x ≼ t}` and `coverage(Y) = {t : y ≼ t}`; since `x` and `y` are
equal-length and distinct they are prefix-incomparable (Prefix, ASN-0034: a proper prefix
is strictly shorter), so these subtrees are disjoint (T10, PartitionIndependence,
ASN-0034) and `coverage(X) ∩ coverage(Y) = ∅`. Now let `a ∈ dom(Σ.L)` be a link with
from-endset `e₁ = X` and to-endset `e₂ = Y`, and let `a ∉ nullified(Σ)` (its type endset
and home are immaterial here; its addressability is not — FL-DEF ranges over
`addressable(Σ)`, so `sat` alone does not confer membership). Then
`coverage(e₁) ∩ coverage(X) ≠ ∅` (it contains `x`), `coverage(e₁) ∩ coverage(Y) = ∅`,
`coverage(e₂) ∩ coverage(Y) ≠ ∅` (contains `y`), and `coverage(e₂) ∩ coverage(X) = ∅`.
Checking both requests against FL-DEF: for `q = (∗, X, Y, ∗)`, `lift(e₁, X) = true` and
`lift(e₂, Y) = true`, so `sat(a, q, Σ)` holds and, `a` being addressable by hypothesis,
`a ∈ findlinks_FTT((∗, X, Y, ∗), Σ)`; for the
reversed `q' = (∗, Y, X, ∗)`, `lift(e₁, Y) ≡ touch(e₁, Y) = (coverage(X) ∩ coverage(Y) ≠ ∅)
= false`, so `sat(a, q', Σ)` fails and `a ∉ findlinks_FTT((∗, Y, X, ∗), Σ)`. Reversing the two
endpoint constraints is therefore not a no-op.

This is exactly what keeps "links *from* X" and "links *to* X" two different, answerable
queries. Bind the from-set to `X` and leave the to-set open — `findlinks_FTT((∗, X, ∗, ∗), Σ)`
— and the result is every link *originating* at `X`. Bind the to-set to `X` and leave the
from-set open — `findlinks_FTT((∗, ∗, X, ∗), Σ)` — and the result is every link *arriving* at
`X`, the backlink query. Had the two ends been merged into one undirected set, both
collapse to "every link touching `X`," and the direction the author asserted would be
lost. Gregory stores the two ends under distinct index subspaces and queries each against
its own slot, so a reversed request finds no link by the wrong-slot match (consultation
Q16) — the directional distinction is enforced at the index level.

## The type is a first-class slot, matched by address

For links to be discoverable by their *kind* of connection, the type must be a full
endset, structurally identical to the from- and to-sets, and matched the same way.
Nelson:

> "A link's type is specified by yet another end-set, pointing anywhere in the docuverse.
> This is symmetrical with the other endsets." (4/44)

The decisive property is *what* the type slot matches against:

> "The search mechanism does not actually look at what is stored under the 'type' it is
> searching for; it merely considers the type's address." (4/44–4/45)

In our terms, the three-component `Θ` is lifted against `coverage(e₃)` — the *address
set* the type endset references — exactly as the from- and to-components are lifted
against their endset coverages. The content store `Σ.C` is never consulted at type
addresses; matching is by address identity.

**FL-TYP (type by address).** The type criterion tests `touch(Σ.L(a).e₃, Θ)`, an overlap
of address coverages, and never reads content stored at any type address. Three
consequences follow. *(a) Ghost types.* A type address need not lie in `dom(Σ.C)`; an
endset whose coverage includes addresses with no stored content is a valid, matchable
type (L9, TypeGhostPermission, ASN-0043) — "Link types may be ghost elements" (4/45).
*(b) Independent constraint.* Because
the type participates in `sat` on equal footing with from and to, a request may constrain
type alone — `findlinks_FTT((∗, ∗, ∗, Θ), Σ)` returns every addressable link of a kind
touching `Θ`, leaving from and to open. *(c) Hierarchy by containment.* A type request
whose span is rooted at a supertype address `p` covers the whole subtree `{t : p ≼ t}`
and so matches all and only subtypes of `p` — L10 (TypeHierarchyByContainment, ASN-0043,
via PrefixSpanCoverage) applied on the request side; the
type slot is searchable for super- and sub-types without any registry. Gregory's index
keys the type endset by its I-addresses under a dedicated type-subspace, matched by
address-overlap and never by stored value, and treats an empty type request as imposing
no type constraint (consultation Q14) — the latter an *encoding* fact, not a semantic
exception: the wire format serialises the empty specset identically to the NOSPECS
sentinel and the parser collapses both to the same absent slot, so an "empty type
request" there *is* the wildcard of FL-WILD below, and the constrained-but-empty `Θ = ∅`
request of FL-EMP is inexpressible in that encoding (consultation-34 Q2). The
address-matching and ghost-validity properties are concrete there.

## Wildcards drop slots, they do not empty the result

A wildcard component is "no specification," and `lift(e, ∗) = true` makes it the
universal constraint — it drops out of the conjunction rather than contributing the empty
set.

**FL-WILD (wildcard semantics).** A wildcard slot imposes no constraint:
`findlinks_FTT` with a wildcard component returns exactly the links the *remaining*
constrained slots admit. In the limit `findlinks_FTT((∗, ∗, ∗, ∗), Σ) = addressable(Σ)` —
all currently addressable links — and a single constrained slot yields precisely the
links matching that slot alone. This is the formal reading of Nelson's "If the home-set
is the whole docuverse, all links between these two elements are returned" (4/63): an
unconstrained axis widens, never empties, the result.

A wildcard must not be conflated with a *constrained* slot that happens to bound nothing.
The request grammar admits both: a slot may be left unspecified (`∗`, NOSPECS) or
specified with an endset of empty coverage (the empty endset `∅`, with `coverage(∅) = ∅`).
These are opposite elements of the conjunction.

**FL-EMP (empty constraint is the zero, not the unit).** For a constrained slot whose
endset has empty coverage, `lift(e, ∅) ≡ touch(e, ∅) ≡ coverage(e) ∩ ∅ ≠ ∅` is `false`
for every link `a` (and likewise `liftH(a, H) ≡ home(a) ∈ ∅` is `false` when `H` has empty
coverage). Hence if *any* constrained component of `q` has empty coverage,
`findlinks_FTT(q, Σ) = ∅` regardless of the store's contents. This is the polar opposite of the
wildcard: `∗` is the *unit* of the conjunction (`lift(e, ∗) = true`, drops out, admits
whatever the other slots admit), whereas the empty endset is the *zero*
(`lift(e, ∅) = false`, forces the whole conjunction to `false`). Gregory's back end
realises exactly this asymmetry — a NOSPECS slot is omitted from the intersection (the
slot is simply not consulted), whereas a constrained slot that resolves to no I-addresses
short-circuits the entire find to the empty link-set *before* `intersectlinksets` is even
reached (consultation Q7).

The `touch` test is symmetric in its two coverages, so the same zero behaviour appears
when the empty endset sits on the *link's* side rather than the request's. L3 constrains
only the type slot to be non-empty (`e₃ ≠ ∅`); a stored link may legitimately carry an
empty from- or to-endset (`e₁ = ∅` or `e₂ = ∅`). For such a link, against *any*
constrained from-request `F ≠ ∗`,

  `lift(∅, F) ≡ touch(∅, F) ≡ coverage(∅) ∩ coverage(F) = ∅ ∩ coverage(F) = ∅`,

so `lift(∅, F) = false` and the link is correctly excluded from every constrained
from-slot — *from nothing is not a from-match*. On that axis it is admitted only under
the from-wildcard `F = ∗`, where `lift(∅, ∗) = true` drops the slot from the conjunction
and the link is matched on its remaining (non-empty) slots alone. The to-side is identical
with `e₂` and the to-request `G`. Empty coverage on *either* side of `touch` — the request
component (FL-EMP above) or the link's own endset (here) — annihilates that slot's test;
the two are the same zero, and this is the intended "a link with no from-endpoint is
discoverable only as a to-match, never as a from-match" semantics.

We record a second implementation divergence here. Gregory's `intersectlinksets`, given
three empty (wildcard) slots, returns the empty set rather than the universal set — the
degenerate all-wildcard request yields nothing in the current back end (consultation
Q15). The abstract semantics, and Nelson's intent, require the universal answer. An
alternative implementation must treat the fully-unconstrained request as returning all
addressable links to meet FL-WILD. (Note this is the all-*wildcard* case; by FL-EMP an
all-*empty* request `((∅,∅,∅,∅))` correctly returns `∅` under both the abstract semantics
and the back end.)

## The result is a current snapshot

Two stability facts about the snapshot — the reading recorded with FL-CMP above — follow
from immutability. First, a link's match
status is *permanent once created*, modulo retraction: `Σ.L(a)` is fixed by L12 and
`home(a)` is fixed by the address, so `sat(a, q, ·)` is constant for a fixed `q` across
the link's life. Second, retraction is the *only* way for an addressable matching link to
leave the answer.

**FL-MON (monotone accumulation absent retraction).** For any reachable `Σ →* Σ'` with
`a ∉ nullified(Σ')`: if `a ∈ findlinks_FTT(q, Σ)` then `a ∈ findlinks_FTT(q, Σ')`. A matching
link, once found and not withdrawn, stays found as the store grows. (By LP13 (ASN-0098,
UnconditionalLinkPersistence) `Σ'.L(a) = Σ.L(a)` across the reachability closure `Σ →* Σ'`,
and `home(a)` is a projection of the fixed address `a`, so `sat(a, q, Σ') = sat(a, q, Σ)`;
and `a ∈ addressable(Σ')` because `a ∈ dom(Σ'.L)` by link-store monotonicity across
`Σ →* Σ'` (ASN-0098 StoreMonotonicity★) and `a ∉ nullified(Σ')` by hypothesis.)

### The only result-changing transition

FL-MON above and FL-STB below are monotonicity and invariance statements; they do not
isolate *which* single transition can move a link into or out of the answer. Since
`findlinks_FTT(q, ·)` is a function of `Σ.L` alone (FL-LOC), and every operation in `→` other
than K.λ preserves `Σ.L` (F-PRES, ASN-0127, as
recorded above), *K.λ is the unique result-changing transition*. A single K.λ step can
change the answer in three ways, and we
compute the weakest precondition of each:
the entry of a newly created *ordinary* link, the entry of a newly created *retraction*
link, and the survival of an existing match under a retraction-bearing K.λ. These three
exhaust the result changes a single step admits. An *existing* non-member cannot enter:
it failed `sat` — constant for it across the step, `Σ.L(a)` being fixed by L12 and
`home(a)` by the address — or it lay in `nullified(Σ)`, where R6a keeps it; either way it
stays out. And no existing match leaves across an *ordinary* K.λ: case (a) establishes
`L_R^{Σ'} = L_R^Σ`, which leaves `nullified` fixed on `dom(Σ.L)`, so both membership
conjuncts of every existing member persist — exits occur only under the
retraction-bearing step of case (c). A retraction link is a first-class link with its
own address, and Nelson's link model exempts no type from search — "there is essentially
nothing in the Xanadu system except documents and their arbitrary links" (4/41), and the
search mechanism "does not actually look at what is stored under the 'type' … it merely
considers the type's address" (4/44–4/45) — so a type-`Θ` query touching the retraction
class returns a retraction link like any other (consultation Q2). The operation must
therefore surface a fresh retraction link when it matches; its entry's weakest
precondition differs from the ordinary case precisely because the *same* K.λ grows
`L_R`, carrying a self-retraction term.

**FL-WP (weakest precondition for the result-changing step).**

*Scope of the wp — additional precondition given an enabled step.* Each case below opens
"Let `Σ → Σ'` be a K.λ step that allocates a fresh address …", which presupposes the step's
own applicability predicate `enabled(K.λ)` — ASN-0093's K.λ binding precondition: the home
document is allocated (`d ∈ dom(Σ.M)`), the address is pinned to the link sub-allocator's
frontier (`ℓ = [d.0.s_L.1]` on first emission, `ℓ = inc(ℓ_prev, 0)` thereafter), and the
value has L3 shape (arity `≥ 3`, each slot an endset, slot-3 type endset non-empty).
Freshness `ℓ ∉ dom(Σ.L)` is a consequence of the frontier binding
(FirstEmissionFreshness / SubsequentEmissionFreshness, ASN-0093), not a precondition
conjunct. The displayed conjunctions are therefore the weakest *additional* precondition
under which the named, enabled K.λ step lands `ℓ` (resp. `b`, `a`) in the answer; the full
weakest precondition is `enabled(K.λ) ∧ ⟨displayed conjunction⟩`. We carry `enabled(K.λ)`
implicitly rather than redisplay it in
each case.

*(a) Entry of a fresh ordinary link.* Let `Σ → Σ'` be a K.λ step that allocates a fresh
address `ℓ ∉ dom(Σ.L)` with value `Σ'.L(ℓ) = (F, G, Θ, e₄, …, e_N)` of arity
`N = |Σ'.L(ℓ)| ≥ 3` — the first three slots are `(F, G, Θ)`, the higher slots absent when
`N = 3` — homed at `d = home(ℓ)`. We must cut the partition on *retraction-relation membership*, not on coverage
class alone, because ASN-0086's `L_R^Σ = {(a, F, G) : a ∈ dom(Σ.L) ∧ |Σ.L(a)| = 3 ∧ …}`
requires **arity exactly 3** in addition to the slot-3 coverage test. We therefore call the
link *ordinary (non-retraction)* exactly when it does not enter the retraction relation —
`ℓ ∉ L_R^{Σ'}` — which by ASN-0086's definition is `¬(|Σ'.L(ℓ)| = 3 ∧ coverage(Σ'.L(ℓ).e₃) = coverage(R))`,
where `R` is ASN-0086's designated retraction-type representative and the slot-3 test is the
foundation's coverage equality `coverage(Σ'.L(ℓ).e₃) = coverage(R)` (equivalently, `Σ'.L(ℓ).e₃ ∈ [R]`,
the endset lying in `R`'s `~`-class). This subsumes two sub-cases: the type endset does not match the
retraction coverage (`coverage(Θ) ≠ coverage(R)`, any arity), *or* the type endset
does match but the link's arity is not 3 (`coverage(Θ) = coverage(R)` with
`N > 3` — a higher-arity link, still in the result space, whose slot-3 coverage tests retraction
yet which never enters the triple-restricted `L_R`). In either sub-case `ℓ ∉ L_R^{Σ'}`;
membership of an *existing* address in `L_R` reads its stored value (the arity-3 conjunct
and the slot-3 coverage test), and every prior value persists unchanged across K.λ (L12),
so with `ℓ` the only address `Σ.L` gains, no tuple enters or leaves `L_R`:
`L_R^{Σ'} = L_R^Σ`. Then

  `wp(K.λ, ℓ ∈ findlinks_FTT(q, ·)) ≡ ¬(E (b, F', G') ∈ L_R^Σ :: ℓ ∈ coverage(G')) ∧ liftH_d(q.H) ∧ lift(F, q.F) ∧ lift(G, q.G) ∧ lift(Θ, q.Θ)`,

where `liftH_d(q.H) ≡ (q.H = ∗) ∨ (d ∈ coverage(q.H))`. Every conjunct is a pre-state
predicate — `ℓ` itself is pre-state-determined, pinned to the sub-allocator frontier by the
enabled step (scope note above). In post-state terms the first conjunct reads
`ℓ ∉ nullified(Σ')`; the equivalence is derived below.
*Derivation.* By FL-DEF, `ℓ ∈ findlinks_FTT(q, Σ') ⟺ ℓ ∈ addressable(Σ') ∧ sat(ℓ, q, Σ')`. We treat
the two conjuncts in turn.

The addressability conjunct does *not* drop out by freshness alone, and we must carry it.
With `L_R^{Σ'} = L_R^Σ` (established above), ASN-0086's definition gives
`nullified(Σ') = { a ∈ dom(Σ'.L) : (E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G')) }`. Freshness
`ℓ ∉ dom(Σ.L)` guarantees only that *this* step emits no retraction targeting `ℓ`; it does
not exclude a *pre-existing* retraction tuple `(b, F', G') ∈ L_R^Σ` whose to-coverage already
names `ℓ`. Endset coverage may reference ghost addresses with no stored content (L4
EndsetGenerality and L9 TypeGhostPermission, ASN-0043; R5, ASN-0086; LP17/LP18
orphan/resurrection, ASN-0098), so the future address `ℓ` can be
uncovered while merely fresh against `dom(Σ.L)` yet covered once it enters `dom(Σ'.L)` — exactly
the regime in which `nullified(Σ)`, restricted to `dom(Σ.L)`, omits `ℓ` "before" allocation
while `nullified(Σ')` includes it "after." In that case `ℓ ∈ nullified(Σ')`, so
`ℓ ∉ addressable(Σ')` and `ℓ ∉ findlinks_FTT(q, Σ')` *even though* `sat(ℓ, q, Σ')` holds. The
addressability conjunct is therefore not vacuous, and the weakest precondition must retain it:
`ℓ ∉ nullified(Σ') ≡ ¬(E (b, F', G') ∈ L_R^Σ :: ℓ ∈ coverage(G'))` — the displayed first
conjunct, in pre-state form. The conjunct would be dischargeable only under a stated
retraction discipline; this ASN works over the *full* ASN-0047 transition vocabulary,
which imposes no such discipline, so we keep the conjunct explicit.

The matching conjunct `sat(ℓ, q, Σ')` reads only the committed value's first three slots
`(F, G, Θ)` and the committed address (via `home(ℓ) = d`) — both fixed by the operation's own
arguments, with no further pre-state dependence; the higher slots `e₄, …, e_N` are
unconstrained by `sat` — so it equals the four-way conjunction
`liftH_d(q.H) ∧ lift(F, q.F) ∧ lift(G, q.G) ∧ lift(Θ, q.Θ)`. The wp is therefore the displayed
*five*-way conjunction: a fresh link enters the answer iff its just-committed value and home meet
all four lifted criteria of `q` *and* no standing retraction tuple already covers its address.

*(b) Entry of a fresh retraction link.* The ordinariness cut of case (a) sets aside
exactly one realisable K.λ result change: the entry of a fresh link that *itself* enters the
retraction relation; we compute
its weakest precondition rather than scope it out. Let `Σ → Σ'` be a K.λ step that allocates
a fresh address `b ∉ dom(Σ.L)` with value `Σ'.L(b) = (F_b, G', Θ_b)` of *arity exactly 3*
whose type endset matches the retraction coverage — `coverage(Θ_b) = coverage(R)` —
so by ASN-0086's slot-3 test (now applicable, the arity-3 conjunct met) `b ∈ L_R^{Σ'}`. This
is precisely the complement of case (a)'s `ℓ ∉ L_R^{Σ'}`, so (a) and (b) partition the
fresh-link space exhaustively. We do *not* assume the empty-from RetractionDirectionality
convention: this ASN works over the full ASN-0047 vocabulary, which admits attribution-bearing
retractions whose from-slot `F_b` may be non-empty (ASN-0086's convention reserves the from-set
for "attribution-bearing endset content *or* … empty for unattributed retractions"). By
ASN-0086's `nullified`, only the to-coverage `coverage(G')` of a retraction tuple bears on
nullification, so the targets are read from `G'` regardless of `F_b`. The retraction relation
grows by exactly this one tuple: `L_R^{Σ'} = L_R^Σ ∪ {(b, F_b, G')}` — every prior tuple
persists by L12, and `b` is the only address `Σ.L` gains. Then

  `wp(K.λ, b ∈ findlinks_FTT(q, ·)) ≡ ¬(E (c, F'', G'') ∈ L_R^Σ :: b ∈ coverage(G'')) ∧ b ∉ coverage(G') ∧ liftH_d(q.H) ∧ lift(F_b, q.F) ∧ lift(G', q.G) ∧ lift(Θ_b, q.Θ)`,

with `d = home(b)`. *Derivation.* By FL-DEF,
`b ∈ findlinks_FTT(q, Σ') ⟺ b ∈ addressable(Σ') ∧ sat(b, q, Σ')`. The matching conjunct reads
the committed value `(F_b, G', Θ_b)` and home `d` — both fixed by the operation's own
arguments — so `sat(b, q, Σ') = liftH_d(q.H) ∧ lift(F_b, q.F) ∧ lift(G', q.G) ∧ lift(Θ_b, q.Θ)`.
(In the unattributed sub-case `F_b = ∅`, FL-EMP gives `lift(∅, q.F) = false` under any
*constrained* `q.F` and `true` only under the from-wildcard, so an empty-from retraction link
is from-discoverable only when `q.F = ∗`; a non-empty `F_b` is tested exactly like any other
from-endset.) The addressability conjunct unfolds over the *extended* retraction relation: for
the fresh `b`,
`b ∈ nullified(Σ') ⟺ (E (c, F'', G'') ∈ L_R^{Σ'} :: b ∈ coverage(G''))
⟺ (E (c, F'', G'') ∈ L_R^Σ :: b ∈ coverage(G'')) ∨ b ∈ coverage(G')`, splitting the
existential over the disjoint union `L_R^Σ ∪ {(b, F_b, G')}`. The first disjunct is the
case-(a) hazard — a *pre-existing* retraction tuple already covering the ghost-allocated `b`;
the second is the *self-retraction* term `b ∈ coverage(G')`, in which the very tuple just
committed names its own address (a self-nullifying retraction). Negating,
`b ∉ nullified(Σ') ≡ ¬(E (c, F'', G'') ∈ L_R^Σ :: b ∈ coverage(G'')) ∧ b ∉ coverage(G')`.
This is precisely case (a)'s addressability conjunct with the self-retraction term
`b ∉ coverage(G')` adjoined — live here because `b` is the fresh retractor's own address.
Conjoining matching and addressability gives the
displayed *six*-way wp: a fresh retraction link enters the answer iff its committed value and
home meet all four lifted criteria of `q`, no standing retraction tuple already covered its
address, *and* it does not retract itself.

*(c) Survival of an existing match under retraction.* Let `Σ → Σ'` be a K.λ step that
commits a *retraction tuple* whose to-coverage is `coverage(G')`, leaving every link value
and home untouched (L12). For an existing link `a ∈ dom(Σ.L)`,

  `wp(K.λ_retract, a ∈ findlinks_FTT(q, ·)) ≡ a ∈ findlinks_FTT(q, Σ) ∧ a ∉ coverage(G')`.

*Derivation.* `sat(a, q, ·)` is constant across the step (`Σ'.L(a) = Σ.L(a)` by L12,
`home(a)` fixed), so by FL-DEF `a ∈ findlinks_FTT(q, Σ') ⟺ a ∈ addressable(Σ') ∧ sat(a, q, Σ)`.
We need the *exact* membership equation for `nullified(Σ')` on existing addresses, both
directions — the ⊆ direction is what makes this the *weakest* precondition, licensing
`a ∉ nullified(Σ')` from `a ∉ nullified(Σ) ∧ a ∉ coverage(G')`; R6b supplies only the ⊇
half (hitting `coverage(G')` forces nullification), and the split below supplies ⊆ from
the singleton extension of `L_R`.
The retraction-bearing K.λ commits exactly the one tuple `(b, F_b, G')` (we write its
from-slot `∅` below as a placeholder, since `nullified` reads only the to-coverage `G'` and the
from-slot — empty or attribution-bearing — is immaterial to this case), and by L12
(immutability) every prior tuple persists unchanged, so the retraction relation grows by
exactly that tuple: `L_R^{Σ'} = L_R^Σ ∪ {(b, ∅, G')}`. Unfolding ASN-0086's definition of
`nullified` at `Σ'` over this relation, for an existing `a ∈ dom(Σ.L)`:
`a ∈ nullified(Σ') ⟺ (E (c, F'', G'') ∈ L_R^{Σ'} :: a ∈ coverage(G''))
⟺ (E (c, F'', G'') ∈ L_R^Σ :: a ∈ coverage(G'')) ∨ a ∈ coverage(G')
⟺ a ∈ nullified(Σ) ∨ a ∈ coverage(G')`,
where the middle step splits the existential over the disjoint union `L_R^Σ ∪ {(b, ∅, G')}`.
This equation is stated and used only on the existing-link slice `a ∈ dom(Σ.L)`; the fresh
retractor address `b ∈ dom(Σ'.L) \ dom(Σ.L)` lies outside it, so the self-retraction term
`b ∈ coverage(G')` does not arise here — it is case (b)'s live conjunct. Negating,
`a ∈ addressable(Σ') ⟺ a ∉ nullified(Σ) ∧ a ∉ coverage(G')`. Conjoining with `sat` and
folding `a ∉ nullified(Σ) ∧ sat(a, q, Σ)` back into `a ∈ findlinks_FTT(q, Σ)` gives the stated
wp: a found link survives a retraction step exactly when the retraction's to-coverage does
not name it. Setting `a ∉ coverage(G')` to hold for all `a` already in the answer recovers
FL-MON's no-retraction hypothesis. That this conjunct's failure is the *sole* route by
which a match leaves the answer is the present analysis's own conclusion — case (a) bars
exit under an ordinary K.λ, and F-PRES bars it under every other operation; FL-RET below
is the downstream claim that an exit so taken is permanent.

## Stability under content editing

If linked content is later edited, what of the result must remain stable? Xanadu links
attach to bytes — to I-addresses (content identity) — not to V-positions:

> "links can survive editing. If any of the bytes are left to which a link is attached,
> that link remains on them." (4/42)

Editing operations rewrite arrangements `Σ.M`; they do not touch the link store `Σ.L`
(F-PRES, ASN-0127; values immutable, L12) nor the content store `Σ.C` (append-only, S0),
and they do not alter the I-addresses an endset references. Because every request is
phrased over I-addresses (the content-identity regime), the answer is a function of
`Σ.L` and `q` alone (FL-LOC), and neither moves under editing.

**FL-STB (stability under editing).** For a transition `Σ → Σ'` that preserves the link
store — `Σ'.L = Σ.L` — and any request `q`,
`findlinks_FTT(q, Σ') = findlinks_FTT(q, Σ)`. This is FL-LOC routed through ASN-0127's
meta-lemma F-CIL (ComprehensionInvariantUnderΣL): FL-DEF is the comprehension
`{ a ∈ dom(Σ.L) : a ∉ nullified(Σ) ∧ sat(a, q, Σ) }`, whose membership predicate
consults only `Σ.L` and query-data (FL-LOC), so F-CIL
delivers the equality from the single hypothesis `Σ'.L = Σ.L`; in particular
retraction-set preservation (`nullified(Σ') = nullified(Σ)`) is a consequence of the
link-store hypothesis, not an independent assumption. Pure-arrangement edits (insertion,
deletion, rearrangement) and content appends preserve `Σ.L` (F-PRES, ASN-0127) and so
leave the answer invariant. The membership of the result may be expressed
through different V-positions before and after the edit, but the *set of link
identities returned is unchanged*.

Nelson's one exception — a link drops from results when an *entire* endset's content is
deleted, "nothing left at one end" (4/42) — concerns a different phrasing of the request.
A *V-spec* request names its target through a document's current arrangement; ASN-0127
formalises exactly this front-end as the two-phase combinator `findlinks_V` (F-V,
TwoPhaseFactoring): a V-region `W` is first resolved through the arrangement to the
I-address set `image(W, d, Σ)`, and only then does a link query run. The fragility is
then a theorem rather than an observation: `findlinks_V` is non-monotone across
arrangement edits — extension grows it, contraction shrinks it, reordering can move it
(D-NONMONO, ASN-0127) — and it can change while `dom(Σ.L)` is fixed (D-PRES, ASN-0127).
In particular, if an endpoint's content has been fully removed from every arrangement,
no V-spec resolves to
its I-addresses, so a V-spec request cannot name it. Under the I-address regime the same
link is *orphaned* but still content-identity-findable: its endset I-addresses persist
(content is never destroyed, S0), and a direct I-address request still matches it
(consultation Q18 documents the surviving index entries; ASN-0098's LP17/LP18 give the
orphan/resurrection cycle). The abstract operation, specified over I-addresses, is stable
(FL-STB); the arrangement-mediated naming of the request is a separable front-end
convenience whose fragility under full deletion is a property of the *resolution* (F-V's
first phase), not of the underlying link query.

## Retraction is permanent absence from current inquiry

When a link is retracted, what must the system guarantee about its absence from
subsequent answers to the same four-set inquiry? Complete and consistent absence from the
current line of descent.

**FL-RET (retraction absence).** If `a ∈ nullified(Σ)`, then for every reachable
`Σ →* Σ'` and every request `q`, `a ∉ findlinks_FTT(q, Σ')`. The exclusion is total: even if
`a`'s endsets would still satisfy every endpoint criterion, `a ∉ addressable(Σ')` removes
it from FL-DEF, and the non-decrease of `nullified` across `→` and `→*` (recorded in "The
answer is forced") keeps it out forever. A retracted link does not linger as a phantom
result, and its *exclusion* disturbs no other link's membership: for any `b ≠ a`, neither
membership conjunct reads `a`'s status — `sat(b, q, ·)` reads only `b`'s own value and
address, and `b ∈ nullified(·)` is an existential over the stored tuples of the audit
slice `L_R`, which is selected by stored-value tests alone and is not edited by `a`'s
nullification. That same fact bounds the claim: `a`'s *stored value* is not silenced by
`a`'s exclusion. If `a` is itself a retraction tuple, it remains in `L_R^{Σ'}` after `a`
is nullified and continues to nullify its targets — retraction-of-retraction is a
non-fixpoint operation (R6b, ASN-0086) — so retracting a retractor removes *it* from
every answer without restoring the `sat`-satisfying links its to-coverage excludes.

The guarantee is *scoped* to current addressability, as Nelson's "not currently
addressable" (4/9) demands and no more. A retracted link is removed from current
inquiry but not destroyed — it may remain in other versions that captured it before
retraction, and a time-qualified or version-qualified inquiry into a prior state could
still surface it. Those scopes are out of the present operation, which inquires against
the current state. We mark the version-scoped behaviour as an open question rather than a
claim.

## Cross-document reach

Must the discovery reach across all documents whose arrangements could surface the same
links? It must, and FL-LOC makes the reach automatic rather than something to be
iterated for: `findlinks_FTT(q, Σ)` is a function of `Σ.L` and `q` alone — the arrangements
`Σ.M` do not appear in it. The search is therefore intrinsically a global
content-identity sieve over the link
store, not a per-document enumeration.

**FL-REACH (cross-document reach).** For any request `q`, `findlinks_FTT(q, Σ)` is
independent of `Σ.M` (immediate from FL-LOC). Four consequences
follow. *(a) Every home is reached.* The store is
searched whole; a link is eligible regardless of which document homes it, so in-links —
stored, as the links relevant to a reading typically are, in documents other than the
one being read — are found on equal footing with out-links. *(b) Transclusion is found once.* When the same endpoint content is shared
across documents, the link is indexed by that content's I-addresses and is found exactly
once by content identity, however many documents surface it (consultation Q20). *(c)
Whole-docuverse residence.* Setting `H = ∗` imposes no residence bound, returning all
matching links wherever homed — Nelson's "if the home-set is the whole docuverse, all
links … are returned" (4/63). *(d) Superset of the satisfying discoverable links.*

  `findlinks_FTT(q, Σ) ⊇ ⋃_d { a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) ∧ discoverable_from(a, d, Σ) }`

— immediate from FL-DEF, every member of the right-hand union being addressable and
satisfying. The inclusion is *strict* whenever a satisfying, addressable *orphan* exists:
an addressable `a` with `sat(a, q, Σ)` whose endset I-addresses lie in no arrangement
range fails `discoverable_from(a, d, Σ)` for every `d` yet lies in `findlinks_FTT(q, Σ)`. The
restriction to *satisfying* links is what makes the comparison sound: `discoverable_from`
is request-independent — by LP12 (ASN-0098) it consults `ran(Σ.M(d))`, never `q` — so the
bare union `⋃_d { a : discoverable_from(a, d, Σ) }` is the set of all non-orphan links,
and against `q = (∗, ∅, ∗, ∗)`, where FL-EMP forces `findlinks_FTT(q, Σ) = ∅`, it is not
contained in the result. The operation
is therefore at least as complete as any document-by-document enumeration of the
*satisfying* links, and strictly more so in the presence of satisfying orphans. No
qualifying link is missed for want of a document to look in.

## A worked instance

We verify the principal claims against one concrete store. Fix a document
`d = [1,0,1,0,1]` (`zeros(d) = 2`, document-level). Under `d` sit three text I-addresses
in subspace `s_C = 1`,

  `p = [1,0,1,0,1,0,1,1]`,  `x = [1,0,1,0,1,0,1,5]`,  `y = [1,0,1,0,1,0,1,9]`,

and two type addresses `τ = [1,0,1,0,9,0,3,1]` and `σ = [1,0,1,0,9,0,3,2]` (distinct, in a
type subspace). Three links are homed at `d` (link subspace `s_L = 2`):

  `a₁ = [1,0,1,0,1,0,2,1]`,  `a₂ = [1,0,1,0,1,0,2,2]`,  `a₃ = [1,0,1,0,1,0,2,3]`,

with values (writing each endset by a unit-depth span on the stated address, so its
coverage is that address's subtree):

| link | `e₁` (from) | `e₂` (to) | `e₃` (type) |
|------|-------------|-----------|-------------|
| `a₁` | `{x}`-subtree | `{y}`-subtree | `{τ}`-subtree |
| `a₂` | `{y}`-subtree | `{x}`-subtree | `{τ}`-subtree |
| `a₃` | `{p}`-subtree | `{x}`-subtree | `{σ}`-subtree |

The five content/type addresses are pairwise non-nesting, so their subtree coverages are
pairwise disjoint (T10, ASN-0034). Take request endsets `X, Y, P` covering the subtrees
of `x, y, p`
respectively, and `Θ_τ` covering the subtree of `τ`. Fix a retraction-type
representative at address `ρ = [1,0,1,0,9,0,3,7]` in the type subspace, so
`coverage(R) = {t : ρ ≼ t}` — `ρ` is equal-length with `τ` and `σ` and distinct, hence
non-nesting, so its subtree is disjoint from both stored type subtrees. `nullified(Σ)`
is then computed, not assumed: no stored link has slot-3 coverage equal to
`coverage(R)`, so `L_R^Σ = ∅`, `nullified(Σ) = ∅`, and
`addressable(Σ) = {a₁, a₂, a₃}`.

*Trace 1 — directional from/to (exercises FL-SND, FL-CMP, FL-DIR).* For
`q = (∗, X, Y, ∗)`, evaluate `sat` per link: `a₁` has `lift(e₁, X) = true` (coverage
contains `x`) and `lift(e₂, Y) = true`, home/type wildcards drop, so `sat(a₁, q, Σ)` holds;
`a₂` has `lift(e₁, X) ≡ touch(e₁, X) = false` (its from-coverage is `y`'s subtree,
disjoint from `X`), so it fails; `a₃` has `lift(e₁, X) ≡ touch(e₁, X) = false` (from-coverage
is `p`'s subtree), so it fails. By FL-DEF,
`findlinks_FTT((∗, X, Y, ∗), Σ) = {a₁}`. Soundness (FL-SND): the one returned link is
addressable (`nullified(Σ) = ∅`) and satisfies every constrained slot. Completeness
(FL-CMP): `a₂, a₃` are correctly absent, both failing
the from-slot. Reversing to `q' = (∗, Y, X, ∗)` gives `findlinks_FTT(q', Σ) = {a₂}` by the
symmetric computation — `a₁` now fails `lift(e₁, Y)`. The two answers differ, witnessing
FL-DIR: `a₁ ∈ findlinks_FTT(q,Σ) \ findlinks_FTT(q',Σ)` and `a₂ ∈ findlinks_FTT(q',Σ) \ findlinks_FTT(q,Σ)`.

*Trace 2 — type alone (FL-TYP).* For `q = (∗, ∗, ∗, Θ_τ)`, only the type slot constrains:
`a₁` and `a₂` have `lift(e₃, Θ_τ) = true` (both type-touch `τ`), while `a₃` has type `σ`,
disjoint from `Θ_τ`, so it fails. `findlinks_FTT((∗, ∗, ∗, Θ_τ), Σ) = {a₁, a₂}` — the kind-of
link query, regardless of endpoints.

*Trace 3 — wildcard vs. empty (FL-WILD, FL-EMP).* The all-wildcard request returns
everything addressable: `findlinks_FTT((∗, ∗, ∗, ∗), Σ) = {a₁, a₂, a₃}`. By contrast the
request `(∗, ∅, ∗, ∗)` — a *constrained* from-slot with empty coverage — gives
`lift(eᵢ, ∅) = false` for every link, so `findlinks_FTT((∗, ∅, ∗, ∗), Σ) = ∅`. The empty slot
annihilates; the wildcard slot widens.

*Trace 4 — retraction (FL-RET), exercising the determinacy of `nullified`.*
`nullified(Σ)` is not a free parameter — by FL-LOC it is a function of the stored
values, and over the three-link store above it is empty, as computed. To retract `a₁`
the store must hold a witness. Extend it with a retraction link at the link
sub-allocator frontier after `a₃`,

  `r₄ = [1,0,1,0,1,0,2,4]`  with value  `(∅, {(a₁, δ(1,#a₁))}, {ρ}-subtree)`

— arity exactly 3 with slot-3 coverage equal to `coverage(R)`, so `r₄ ∈ L_R^Σ`, and its
to-coverage `{t : a₁ ≼ t}` (PrefixSpanCoverage, ASN-0043) names `a₁`. Computing:
`nullified(Σ) = {a₁}` — the one to-coverage in `L_R^Σ` contains `a₁` by reflexivity of
`≼` and no other link address, `a₂`, `a₃`, `r₄` being equal-length siblings of `a₁`,
prefix-incomparable with it — so `addressable(Σ) = {a₂, a₃, r₄}`. Trace 1's
`q = (∗, X, Y, ∗)` now yields `findlinks_FTT(q, Σ) = ∅`: `a₁` is excluded by FL-DEF even
though its endsets still satisfy every endpoint criterion; `a₂` and `a₃` fail the
from-slot as in Trace 1; and `r₄` fails it by FL-EMP's link-side rule,
`lift(∅, X) = false`. By FL-RET, `a₁` stays excluded along every reachable `Σ →* Σ'`.

*Trace 5 — empty link endset (FL-EMP link-side symmetry).* Were an additional link `a₄` homed
at `d` with from-endset `e₁ = ∅`, to-endset `e₂ = {x}`-subtree, and some non-empty type
endset (well-formed: L3 constrains only the type slot to be non-empty, so an empty `e₁` is
permitted), then under the constrained from-request `q = (∗, X, ∗, ∗)` it fails —
`lift(e₁, X) ≡ touch(∅, X) = coverage(∅) ∩ coverage(X) = ∅`, so `false` — and is absent.
Under the to-request `q' = (∗, ∗, X, ∗)`, with the from-slot wildcarded, it is admitted —
`lift(e₂, X) = true` while `lift(e₁, ∗) = true` drops the empty from-slot. The link with no
from-endpoint is found only as a to-match, never as a from-match — the same zero as the
empty *request* component, now on the link's side.

*Trace 6 — residence axis (exercises FL-RES, and FL-SND on the home slot).* The earlier
traces all fix `H = ∗`, so the residence criterion is never exercised concretely. We do so
now, starting again from the base three-link store (Trace 4's `r₄` not present, so
`L_R^Σ = ∅`, `nullified(Σ) = ∅`, and `a₁, a₂, a₃` are all addressable; on the
post-Trace-4 store, where `a₁` is nullified, the answers below would differ). Augment
that store with a second document `d' = [1,0,1,0,2]` (document-level,
`zeros(d') = 2`, non-nesting with `d` — they are equal-length and differ in the last
component) and a further link homed there,

  `a₅ = [1,0,1,0,2,0,2,1]`,  so `home(a₅) = N(a₅).0.U(a₅).0.D(a₅) = [1,0,1,0,2] = d'`,

carrying endpoints *identical* to `a₁`'s — from-endset `{x}`-subtree, to-endset
`{y}`-subtree, type `{τ}`-subtree. (Its endsets reference content homed under `d`;
cross-document endsets are admissible, L4.) Take three home-sets, each a unit-depth span
whose coverage is the subtree of its root (`coverage = {t : root ≼ t}`, PrefixSpanCoverage,
ASN-0043; order-convex under T5, ASN-0034):

- `H_d` rooted at the document `d = [1,0,1,0,1]`, covering `{t : d ≼ t}`;
- `H_other` rooted at the document `d' = [1,0,1,0,2]`, covering `{t : d' ≼ t}`;
- `H_node` rooted at the node `[1]`, covering `{t : [1] ≼ t}` — every address beneath node 1.

Hold the endpoint constraints fixed at `X, Y` and vary only `H`. Both `a₁` and `a₅` satisfy
the endpoint slots (`lift(e₁, X) = true`, `lift(e₂, Y) = true`), so the from/to/type axes
*cannot* separate them — any difference in the answer is residence alone.

*Document-granularity, excluding `a₁`.* For `q = (H_other, X, Y, ∗)`,
`athome(a₁, H_other) = (home(a₁) = d ∈ {t : d' ≼ t})`; since `d` and `d'` are equal-length
and non-nesting, `d ∉ coverage(H_other)`, so `liftH(a₁, H_other) = false` and `a₁` is
excluded *purely on `liftH`* — its endsets still touch `X` and `Y`. Symmetrically
`athome(a₅, H_other) = (d' ∈ {t : d' ≼ t}) = true` by reflexivity of `≼`, so
`findlinks_FTT((H_other, X, Y, ∗), Σ) = {a₅}`.

*Document-granularity, readmitting `a₁`.* For `q = (H_d, X, Y, ∗)`,
`athome(a₁, H_d) = (d ∈ {t : d ≼ t}) = true` (reflexivity), readmitting `a₁`, while
`athome(a₅, H_d) = (d' ∈ {t : d ≼ t}) = false`, so `findlinks_FTT((H_d, X, Y, ∗), Σ) = {a₁}`.
The two document-bounded requests differ only in `H` — the endpoint slots are byte-for-byte
identical — yet the result flips between `{a₅}` and `{a₁}`. Residence is varied while
endpoints are held fixed, and the answer changes: orthogonality witnessed directly, exactly
as FL-RES asserts.

*Node-granularity, admitting both (the T5 subtree reading of `athome`).* For
`q = (H_node, X, Y, ∗)`, both documents lie beneath node 1 — `[1] ≼ d` and `[1] ≼ d'`, each
extending the one-component prefix `[1]` — so `athome` holds for both and
`findlinks_FTT((H_node, X, Y, ∗), Σ) = {a₁, a₅}`. This is genuinely a residence *test*, not its
absence: a link homed outside node 1's subtree would fail `liftH(·, H_node)`, whereas the
wildcard `H = ∗` of Trace 1 imposes no test at all. The node-rooted span verifies that
`athome` reads `home(a)`'s membership in the contiguous subtree `{t : [1] ≼ t}` (T5), so
the home-set bounds residence at node — and, by the same construction, account or
document — granularity.

*Trace 7 — FL-WP's load-bearing hazards (case (a) ghost-pre-coverage, case (b)
self-retraction, and first-class retrieval of a standing retractor).* The earlier traces
evaluate `findlinks_FTT` at single states; here we exercise the two
subtle conjuncts of FL-WP against a concrete K.λ step, with the retraction-type
representative `ρ` as fixed in the setup (`coverage(R) = {t : ρ ≼ t}`); let `Θ_ρ` be a
request type endset covering `ρ`'s subtree. Starting again from the base three-link
store (Trace 4's `r₄` not present), reserve two not-yet-allocated link addresses under
`d`, `ℓ = [1,0,1,0,1,0,2,5]` and `b = [1,0,1,0,1,0,2,6]`.

*Case (a), ghost-pre-coverage.* Suppose the store already holds a *standing* retraction link
`r₁ = [1,0,1,0,1,0,2,4]` with value `(∅, G_ℓ, {ρ}-subtree)`, where `G_ℓ` is a unit-depth span
covering `{t : ℓ ≼ t}` — the subtree of the *future* address `ℓ`. Then `r₁ ∈ L_R^Σ` (arity 3,
slot-3 coverage equal to `coverage(R)`), and its to-coverage names `ℓ` although `ℓ ∉ dom(Σ.L)` —
a ghost target (L4, EndsetGenerality, ASN-0043; LP17, ASN-0098). So `nullified(Σ)`
restricted to `dom(Σ.L)` omits
`ℓ`. Now a K.λ step allocates `ℓ` with the *ordinary* value `(X, Y, {τ}-subtree)` — type `τ`,
`coverage(τ) ≠ coverage(R)`, so `ℓ ∉ L_R^{Σ'}` and `L_R^{Σ'} = L_R^Σ`. Take `q = (∗, X, Y, ∗)`.
The matching conjunct holds: `lift(X, q.F) = lift(Y, q.G) = true`, home/type wildcards drop, so
`sat(ℓ, q, Σ')`. Yet FL-WP(a)'s addressability conjunct
`¬(E (c, F', G') ∈ L_R^Σ :: ℓ ∈ coverage(G'))` is *false* — `r₁` witnesses
`ℓ ∈ coverage(G_ℓ)` — so `ℓ ∈ nullified(Σ')`, `ℓ ∉ addressable(Σ')`, and `ℓ ∉ findlinks_FTT(q, Σ')`
despite the match. The wp correctly predicts absence, exhibiting the addressability conjunct as
non-vacuous: freshness against `dom(Σ.L)` does not discharge it.

*Case (b), self-retraction.* Now a K.λ step allocates the fresh address `b` with the arity-3
retraction value `(∅, G_self, {ρ}-subtree)`, where `G_self` is a unit-depth span covering
`{t : b ≼ t}` — `b`'s *own* subtree. Then `coverage({ρ}-subtree) = coverage(R)` with arity 3,
so `b ∈ L_R^{Σ'}` and `L_R^{Σ'} = L_R^Σ ∪ {(b, ∅, G_self)}`. Take `q = (∗, ∗, ∗, Θ_ρ)`: the
type slot matches (`lift({ρ}-subtree, Θ_ρ) = true`), the other slots wildcard, so
`sat(b, q, Σ')`. But FL-WP(b)'s self-retraction conjunct `b ∉ coverage(G')` is *false* —
`b ∈ coverage(G_self)` by reflexivity of `≼` — so `b ∈ nullified(Σ')` and `b ∉ findlinks_FTT(q, Σ')`:
the link nullifies its own address. The wp's sixth conjunct is exactly what predicts this.
The same query also witnesses the positive half of the first-class-retrieval claim: the
*standing* retractor `r₁` is addressable at `Σ'` — neither to-coverage in `L_R^{Σ'}` names
its address (`G_ℓ` covers `ℓ`'s subtree, `G_self` covers `b`'s, and both are
prefix-incomparable with the equal-length sibling `r₁`) — and `sat(r₁, q, Σ')` holds: its
type endset touches `Θ_ρ`, the remaining slots are wildcards, and the stored empty
from-endset is admitted under `q.F = ∗` (FL-EMP's link-side rule). So
`findlinks_FTT((∗, ∗, ∗, Θ_ρ), Σ') = {r₁}` — `b`, the only other link whose type touches `ρ`'s
subtree, is excluded by its self-retraction — a type-`Θ` query touching the retraction
class returns a standing retraction link like any other, exercising the FL-EMP wildcard
admission on a stored link. (Had
`b` instead carried arity `N > 3` with the same retraction-class type, ASN-0086's triple
restriction would keep `b ∉ L_R^{Σ'}`, routing it to case (a) — no self-retraction term, and it
would be returned by `q` whenever no *standing* tuple covered it.)

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| FL-DEF | `findlinks_FTT(q, Σ) = { a ∈ addressable(Σ) : sat(a, q, Σ) }`, with `sat` the conjunction of the four lifted slot-criteria (AND of the ORs); `addressable(Σ) = dom(Σ.L) \ nullified(Σ)` introduced here over ASN-0086's `nullified`; the FTT subscript keeps the operation distinct from ASN-0127's slot-agnostic, unfiltered `findlinks` (F-FIND), of which it is not a restriction; the operation has frame `Σ` (reads only, writes nothing) | introduced |
| FL-LOC | Link-store locality — for fixed `q`, `findlinks_FTT(q, Σ)` is a function of `Σ.L` alone; `Σ.C`, `Σ.M`, `Σ.E`, `Σ.R` are never consulted | introduced |
| FL-DEC | Decidability — `touch(e, r)` is decidable by ASN-0086's CoverageEqualityDecidable cell-decomposition run for intersection-nonemptiness, and `athome(a, H)` by the same T2 span-membership test; corollary at FL-DEF: `sat` is decidable per link, `nullified(Σ)` is computable by ASN-0086's ActiveSubset argument, and `findlinks_FTT(q, Σ) ⊆ dom(Σ.L)` is a finite, computable set (L-fin, ASN-0093) | introduced |
| FL-SND | Soundness — `a ∈ findlinks_FTT(q, Σ) ⟹ a ∈ addressable(Σ) ∧ sat(a, q, Σ)`; no returned link is withdrawn or fails any criterion; no false positives | introduced |
| FL-CMP | Completeness — every `a ∈ addressable(Σ)` with `sat(a, q, Σ)` is returned; the result is exactly the satisfying subset; no silent omission | introduced |
| FL-JUNK | Non-impedance — across any `Σ →* Σ'` with `nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)` (no existing link becomes nullified; added junk may itself be born-nullified) whose added links all fail the request, `findlinks_FTT(q, Σ') = findlinks_FTT(q, Σ)`: the result is invariant under such additions regardless of their quantity; `sat` is decided per link, and the first hypothesis is what holds the non-per-link addressability conjunct fixed | introduced |
| FL-RES | Residence–endpoint independence — the home criterion is a function of the link address alone, the endpoint criteria of the link value alone; the four slots are orthogonal constraints | introduced |
| FL-DIR | Positional directionality — `F` matches `e₁` only and `G` matches `e₂` only; reversing the from/to constraints can change the result, keeping "from X" and "to X" distinct queries | introduced |
| FL-TYP | Type by address — the type criterion tests `coverage(e₃)` by address overlap, never reads stored content; ghost types are matchable, type may be constrained alone, and prefix-rooted type spans match subtype subtrees | introduced |
| FL-WILD | Wildcard semantics — a wildcard slot is the *unit* of the conjunction (drops out, imposes no constraint); the all-wildcard request returns `addressable(Σ)` | introduced |
| FL-EMP | Empty-constraint zero — a constrained slot with empty coverage (`∅`) gives `lift = false` for every link, so any empty constrained component forces `findlinks_FTT(q, Σ) = ∅`; empty-spec (zero) is distinct from wildcard/NOSPECS (unit). By the symmetry of `touch`, the same zero applies to a *link's* own empty endset (L3 permits `e₁ = ∅` or `e₂ = ∅`): such a link is excluded from any constrained from-/to-slot and admitted on that axis only under the corresponding wildcard | introduced |
| FL-MON | Monotone accumulation absent retraction — an unretracted matching link, once found, stays found as the store grows | introduced |
| FL-WP | Weakest preconditions for the unique result-changing transition K.λ — per-case wp's for the three result-changing cases, partitioned by retraction-relation membership; displayed in cases (a)–(c) | introduced |
| FL-STB | Stability under editing — for any request, the result is invariant under any transition preserving `Σ.L`; FL-LOC routed through F-CIL (ComprehensionInvariantUnderΣL, ASN-0127), so retraction-set preservation follows from the single link-store hypothesis; pure-arrangement edits and content appends (F-PRES, ASN-0127) do not change which links are returned | introduced |
| FL-RET | Retraction absence — a retracted link is permanently and completely absent from every subsequent current-state inquiry; its exclusion disturbs no other link's membership conjuncts, though a retracted retractor's stored value continues to nullify its targets (R6b, ASN-0086) | introduced |
| FL-REACH | Cross-document reach — for any request `findlinks_FTT` is independent of `Σ.M`: global over the store, finds transcluded content once, returns all links under a whole-docuverse home-set, and contains every satisfying, addressable link that any document surfaces — `findlinks_FTT(q, Σ) ⊇ ⋃_d { a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) ∧ discoverable_from(a, d, Σ) }`, strict given satisfying orphans (not a superset of the bare, request-independent discoverable union) | introduced |

## Open Questions

What must a version-qualified or time-qualified link inquiry guarantee, so that a link retracted in the current state remains discoverable in the prior states or versions that captured it before retraction?

What invariant must connect an I-address request to its arrangement-mediated (V-spec) phrasing, so that the two regimes agree exactly except on endpoints whose content has been fully removed from every arrangement?

Under what conditions on the home-set's coverage is the residence criterion equivalent to a single subtree-prefix test, so that residence bounding reduces to one containment check rather than a span-set membership?

What must hold of the type endset's coverage for the subtype-by-containment reading to be exact, neither admitting addresses outside the intended supertype subtree nor omitting intended subtypes?

What completeness guarantee must hold across a federation of independently administered stores, so that a single four-set inquiry reaches links homed in stores other than the one receiving the request?
