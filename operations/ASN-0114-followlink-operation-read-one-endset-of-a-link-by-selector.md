> **ASN-0114 · FOLLOWLINK — Reading One Endset of a Link by Selector** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0114-followlink-operation-read-one-endset-of-a-link-by-selector.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0114: FOLLOWLINK — Reading One Endset of a Link by Selector

*2026-06-04*

## The problem

A Xanadu link is a package of connection. Nelson tells us it has at least three
ends — a from-set, a to-set, a type-set — and that "the from-set may be an
arbitrary collection of spans, pointing anywhere in the docuverse. Similarly,
the to-set may be an arbitrary collection of spans pointing anywhere in the
docuverse. We adopt the same convention for link types" (4/43). The three ends
are symmetric; each is an *endset*, an arbitrary, possibly discontiguous set of
spans on the tumbler line.

We are asked a narrow and specific question. Given the address of a link and a
selector that names *one* of its ends, what must the system hand back? We are
not asked to read the whole link, not asked to search for links by their
endset content, not asked to resolve anything into the current arrangement of
some particular document. We are asked only: name a link, name one of its ends,
and receive that end. Call this operation FOLLOWLINK.

The temptation is to treat this as trivial — surely we just return what is
stored. But the discipline of specification forces precise questions. *What
relationship* must the returned object bear to the stored end? *What* does
returning one end disclose about the link, and what must it conceal? *Which*
results are admissible and which are forbidden? And what is the boundary between
a legitimate answer of "nothing" and an illegitimate request?

## The substrate we build on

We take the link model as given. A link value is a finite sequence of `N ≥ 3`
endsets, `Σ.L(a) = (e₁, …, e_N)`, with each `eᵢ ∈ Endset` and `Endset =
𝒫_fin(Span)` a finite set of well-formed spans (ASN-0043, L3; ASN-0034, T12).
The link store `Σ.L : T ⇀ Link` (ASN-0093) maps a tumbler address to its link
value; once an address is in `dom(Σ.L)`, its value is permanently fixed
(ASN-0043, L12 — LinkImmutability). Slot index is a primitive of the model:
`Σ.L(a).eᵢ` is the i-th endset, with slots distinguished (ASN-0043, L6) and the
type slot `e₃` mandated non-empty (L3). Within an endset, however, span order
carries no meaning — an endset is an unordered set (L5).

The ambient state `Σ` is the five-tuple of ASN-0047,
`Σ = (Σ.C, Σ.L, Σ.E, Σ.M, Σ.R)` — content store, link store, entity set,
arrangement family, and provenance relation — and `Σ →* Σ'` is its reachability
relation. FOLLOWLINK consults only the link store `Σ.L`.

For each endset we have its *coverage*, the set of addresses it designates:

> `coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})`

(ASN-0098/ASN-0043). Coverage is a purely combinatorial function of the
endset's spans; it consults no other component of state. A *span-set* (ASN-0053)
is a finite sequence of spans denoting the union of its spans' position sets,
`⟦Σ⟧`, with two span-sets equivalent when they denote the same set. We will
write the result of FOLLOWLINK as a span-set and measure it by its coverage,
writing `coverage(R) := ⟦R⟧` for a span-set `R`. An equality `coverage(R) =
coverage(eᵢ)` then unfolds to `⟦R⟧ = coverage(eᵢ)`.

Two consequences of ASN-0053's S2 — every well-formed span denotes a non-empty
set — we name them here. No object assembled from one or more
spans can have empty coverage; hence coverage vanishes exactly on the empty
object. For a span-set `R` this is the *first collapse*,
`coverage(R) = ∅ ⟺ R = ⟨⟩`; for an endset `e` it is the *second collapse*,
`coverage(e) = ∅ ⟺ e = ∅`. Both are written coverage-first, so each `⟸` is
immediate — the empty object covers nothing — and each `⟹` is the contrapositive
of S2.

## The selector and its domain

The first thing FOLLOWLINK needs is a way to name *which* end. Gregory's
implementation fixes the selector as a literal integer index — `1`, `2`, `3` for
from, to, type — used directly as a coordinate that addresses one of three
stored positions, and rejects every other value at the input boundary by a
strict whitelist (`get1.c:71`, `== 1 || == 2 || == 3`). We abstract from the
particular numbering: a selector is an index into the slot structure, and the
admissible selectors for a link `a` are exactly `{1, …, |Σ.L(a)|}`. Nelson's
n-set generality (4/79) means we do not freeze the upper bound at 3; we freeze
only that the selector must name an *existing* slot.

This gives the operation's domain. We seek the weakest precondition under which
FOLLOWLINK can deliver "the endset named by the selector." The postcondition we
want is `R is a span-set ∧ coverage(R) = coverage(Σ.L(a).eᵢ)`. For the
right-hand `coverage(Σ.L(a).eᵢ)` to be well-defined, two things must hold:
`Σ.L(a)` must exist, which requires `a ∈ dom(Σ.L)`; and the slot `eᵢ` must
exist, which requires `1 ≤ i ≤ |Σ.L(a)|`. Nothing else is needed — the
right-hand side is a function of the link value alone. Hence

> `wp(followlink(a, i), R is a span-set ∧ coverage(R) = coverage(Σ.L(a).eᵢ))`
> `≡ a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)|`.

We name this the *selector-validity* condition and adopt it as the operation's
precondition. We define:

> **F0 (FollowLink).** `followlink(Σ, a, i)` is *defined* (returns a span-set)
> exactly when `a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)|`; otherwise it returns the
> distinguished error value `⊥`.

**Status of the result — a relation, determinate up to coverage.** The wp above
fixes the precondition under which a defined call must return a span-set `R` with
`coverage(R) = coverage(Σ.L(a).eᵢ)`; we must exhibit such an `R`. This is
immediate: `Σ.L(a).eᵢ ∈ Endset = 𝒫_fin(Span)` is *already* a finite set of
well-formed spans, so any sequencing of its members is a span-set whose coverage
is `coverage(Σ.L(a).eᵢ)` by the definition of `coverage` as a union over the
endset's spans. The recorded endset is thus its own witness, and the postcondition
is satisfiable. Because the postcondition pins only the *coverage* of a satisfying
`R`, we read `followlink(Σ, a, i)` as a *relation* standing for any such span-set;
wherever it appears as a single term inside `coverage(·)`, that term is
well-defined.

In two cases the relation collapses to a *single* value, and there the bare
equality `followlink(Σ, a, i) = v` is itself well-defined. (i) *Out of domain:*
when the call is undefined, F0 fixes the value at the distinguished `⊥`. (ii)
*Empty end:* when `coverage(Σ.L(a).eᵢ) = ∅`, the postcondition forces
`coverage(R) = ∅`, whence the first collapse gives `R = ⟨⟩` uniquely.

## What the result must be: exact coverage, no more and no less

The crux is the relationship between the returned span-set and the endset the
link records. Nelson is emphatic that an endset is "not a point and not
necessarily a single span" — it is the *whole* span-set, and naming one end must
yield that whole thing "faithfully" (Q1). He grounds this in the span-set
construct itself: "if you want to designate a separated series of items exactly,
including nothing else, you do this by a span-set" (4/25). The operative phrase
is *including nothing else*. The returned object must denote the recorded end
**exactly** — over-coverage and under-coverage are both forbidden (Q2, Q9).

We state this as the central postcondition of FOLLOWLINK and call it exactness:

> **F1 (CoverageExactness).** For `a ∈ dom(Σ.L)` and `1 ≤ i ≤ |Σ.L(a)|`, with
> `R = followlink(Σ, a, i)`:
> `coverage(R) = coverage(Σ.L(a).eᵢ)`.

The two inclusions of this set equality are the two failure modes Nelson rules
out. `coverage(R) ⊇ coverage(Σ.L(a).eᵢ)` forbids under-coverage: every address
the link records at that end is reported. `coverage(R) ⊆ coverage(Σ.L(a).eᵢ)`
forbids over-coverage: no address the link does not record may appear. Because
the endset *is* the connection — the from-set is precisely what the link is
"from" — there is no wider or narrower region for a faithful answer to report
(Q9).

A consequence worth isolating concerns discontiguity. Nelson insists the result
must not be "flattened to a single span" when the end touches "a broken,
discontiguous set of bytes" (Q1, 4/42). We observe this is not an independent
requirement — it follows from F1. Suppose `coverage(Σ.L(a).eᵢ)` is
*disconnected*: there exist `p < q < r` in `T` with `p, r ∈ coverage(eᵢ)` but
`q ∉ coverage(eᵢ)`. A single span `σ` is order-convex — `⟦σ⟧` contains every
position between any two of its members (ASN-0053, S0). First, `R ≠ ⟨⟩`:
disconnectedness supplies `p, r ∈ coverage(eᵢ)`, so `coverage(eᵢ) ≠ ∅`, and by F1
`coverage(R) ≠ ∅`, which forces `R ≠ ⟨⟩` by the first collapse; hence `|R| ≥ 1`.
Next, `|R| ≠ 1`: if `R` were the singleton `⟨σ⟩` with `⟦σ⟧ ⊇ {p, r}`, then
`q ∈ ⟦σ⟧ = coverage(R)`, yet
`q ∉ coverage(eᵢ)` — contradicting F1. Hence a faithful `R` over a disconnected
end must comprise two or more spans. We record:

> **F2 (DiscontiguityFaithfulness).** If `coverage(Σ.L(a).eᵢ)` is disconnected,
> then any `R` satisfying F1 has `|R| ≥ 2`. The discontiguous structure of the
> recorded end survives into the result; coverage exactness alone enforces it.

Gregory's evidence corroborates this at the mechanical level: when an endset
spans several non-contiguous regions, the implementation preserves them as
several distinct spans, with span consolidation explicitly disabled
(`orglinks.c:412–413`). It does not merge them into an enclosing range. The
implementation thereby satisfies F1, and F2 with it.

## Representation is free; coverage is bound

F1 constrains the result at the level of *coverage* — the set of positions —
not at the level of *span representation*. This is deliberate, and the
implementation evidence shows why it must be. Gregory's tracing (Q13) reveals
that an endset stored as raw address spans is returned by a pure copy chain,
exponent and all preserved exactly, whereas an endset stored relative to a
document's arrangement is *recomputed* by tumbler subtraction, producing a
result that denotes the same positions through a structurally different tumbler.
A span may even be presented "as a pair of tumblers" or "as address + difference
tumbler" (Q7). Two requests, or two implementations, may legitimately return
span-sets that are denotationally equal but representationally distinct.

We therefore bind the contract at coverage and leave representation free:

> **F3 (RepresentationInvariance).** Any two span-sets `R, R'` each satisfying
> F1 for the same `(Σ, a, i)` are denotationally equal: `coverage(R) =
> coverage(R')`. The operation's guarantee is a property of the position set,
> not of the span decomposition or the ordering of spans within the result.

This is consonant with the substrate. Endsets are unordered sets (L5), and type
identity itself is defined on coverage, not on span-set identity — "two type
endsets with different span decompositions but identical address coverage denote
the same type" (ASN-0043, L8). Coverage is the semantically load-bearing
projection throughout the link model; it is the correct level at which to pin
FOLLOWLINK. The order in which Gregory's implementation happens to emit spans
(sorted by stored position, Q16) is below the abstraction: it is a determinate
artifact of one implementation, not a guarantee the contract makes.

## The pure-read frame

Nelson treats reading as something the system performs without disturbing what
it reads — "the network will not, may not monitor what is read" (2/59) — a
principle that only makes sense if requesting is non-mutating by nature (Q10).
FOLLOWLINK reads; it must therefore leave the world as it found it. Gregory
confirms the operation is phrased purely to "return" data and alters nothing it
touches (Q10, Q19); it does not even require the link's home document, or any
referenced document, to be open (Q19).

We capture this as a frame condition, the part of the specification that says
what does *not* change:

> **F4 (PureRead).** `followlink` induces no state transition. For the state
> `Σ` against which it is evaluated, the post-state equals `Σ` in every
> component — the content store `Σ.C`, the link store `Σ.L`, the entity set
> `Σ.E`, every arrangement `Σ.M(d)`, and the provenance relation `Σ.R` are
> identical before and after.

An implementation that satisfied F1 but, say, advanced an arrangement or
perturbed a referenced document as a side effect would not have implemented
FOLLOWLINK.

## Determinism over time

From coverage exactness (F1) and the permanence of a link's value across any
reachable transition sequence (LP13), a determinism property follows without
further assumption. The result of FOLLOWLINK depends only on `Σ.L(a).eᵢ`
(through F1), and that endset persists unchanged once `a ∈ dom(Σ.L)`. Two
requests for the same end of the same link, separated by any sequence of
intervening operations on the system, must therefore denote the same positions.

> **F5 (TemporalDeterminism).** Let `Σ →* Σ'` be any reachable transition
> sequence with `a ∈ dom(Σ.L)`. Then `a ∈ dom(Σ'.L)` and `coverage(followlink(
> Σ', a, i)) = coverage(followlink(Σ, a, i))` for every valid selector `i`.

*Derivation.* LP13 (UnconditionalLinkPersistence, ASN-0098) supplies the
persistence directly: "for every reachable state sequence `Σ →* Σ'` and every
`a ∈ dom(Σ.L)`: `a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)`." Hence `Σ'.L(a).eᵢ =
Σ.L(a).eᵢ`, and F1 applied at each state equates the coverages of the two
results. ∎

## Confinement: one end tells nothing of the others

The question asks what reading one end exposes about the link "without naming or
returning the other endsets." Here Nelson issues a useful correction (Q5): the
Xanadu design makes *no promise of secrecy* about a link's other ends — on the
contrary, bidirectional discoverability is the whole point, and the other ends
are reachable through other operations. So the property we want is not
confidentiality. It is *operational confinement*: this particular operation,
given selector `i`, reads and returns slot `i` and nothing else. The other ends
are not withheld by policy; they are simply not what this request computes.

This confinement is already forced by F1, in one step. F1 fixes
`coverage(followlink(Σ, a, i))` to `coverage(Σ.L(a).eᵢ)` — a function of slot `i`
alone. So for any two links `a, a'` agreeing on slot `i`'s coverage,
`coverage(followlink(Σ, a, i)) = coverage(Σ.L(a).eᵢ) = coverage(Σ.L(a').eᵢ) =
coverage(followlink(Σ, a', i))`, whatever the other slots hold: the contents at
`j ≠ i` never enter F1's right-hand side, so they cannot reach the result. We
isolate this consequence:

> **F6 (SlotConfinement).** `followlink(Σ, a, i)` is a function of the single
> endset `Σ.L(a).eᵢ` (up to coverage). Formally, for links `a, a'` with
> `coverage(Σ.L(a).eᵢ) = coverage(Σ.L(a').eᵢ)` and arbitrary contents at all
> slots `j ≠ i`, the results satisfy `coverage(followlink(Σ, a, i)) =
> coverage(followlink(Σ, a', i))`. The *coverage* of the result thus turns on no `eⱼ` with
> `j ≠ i`.

F6 confines the result only at the level of coverage. A caller, though, receives
a concrete span-set, not its coverage; and the contract — F1 fixing coverage, F3
leaving the span decomposition free — does not forbid an admissible result whose
*representation* is computed from some `eⱼ`, `j ≠ i` (splitting `eᵢ`'s coverage,
say, at a position chosen by inspecting `eⱼ`). Such a result honors F1 yet leaks
`eⱼ` into the span-set the caller actually receives. Representation-level
non-exposure of the other ends is therefore *not* a guarantee the contract makes;
like the span ordering F3 leaves free, it lies below the abstraction. The one
implementation we have evidence for nonetheless delivers it, bounding each
retrieval to a width-one query over the requested slot alone (Q12, Q18).

What, then, *does* the result expose? Exactly `coverage(Σ.L(a).eᵢ)` — the set of
addresses the selected end targets — and, derivable from it, two further facts.
First, a *partial* disclosure of home documents, and we must state its reach
precisely. `coverage(Σ.L(a).eᵢ)` is a set of arbitrary tumblers, not a set of
T4-valid document-bearing addresses: by L4 (EndsetGenerality, ASN-0043) endset
spans "may reference *any* addresses in the tumbler space," and L9 permits ghost
targets, so a covered address may sit at node level (`zeros = 0`) or user level
(`zeros = 1`), carrying no document field, or be a *non-T4-valid interior tumbler*
of a half-open interval `{t : s ≤ t < s ⊕ ℓ}` (e.g. one with adjacent zeros) on
which the field projections are undefined entirely. For exactly those covered
addresses that *are* T4-valid and document-bearing — `zeros(t) ≥ 2`, on which
T4b's field projections `N`, `U`, `D` are defined (ASN-0034, T4) — the home
document is readable directly off the result, with no separate disclosure step
(Q4): for such an address, revealing the region *is* revealing the document it
lands in. The "region *is* documents" equivalence therefore holds only over this
document-bearing slice; over the non-conforming addresses just enumerated, L4
expressly permits ends that name no document at all, and there the disclosure is
of address region alone.
Second, the mere success of the request at selector `i` exposes that the link
has at least `i` slots, `|Σ.L(a)| ≥ i`. Beyond these, the result's coverage
discloses nothing of the other ends — not the from-set when the to-set was
asked, not the type.

## The empty end versus the invalid selector

A subtle but decisive distinction remains. Slots `1` and `2` (and any slot
beyond `3`) may legitimately be empty — a link may record no spans at a given
end. Slot `3` is the lone exception: L3 (ASN-0043; ASN-0093) mandates
`Σ.L(a).e₃ ≠ ∅` for every stored link, so the type end is never empty. The
empty endset's coverage is the empty set, and by F1 any faithful result
has coverage `∅`. Here — unlike the non-empty case — the result is forced to a
*unique* span-set: by the first collapse, `coverage(R) = ∅` gives `R = ⟨⟩`. The
two collapses also discharge the slot-`3` guarantee at once: combining the second
collapse with L3's `Σ.L(a).e₃ ≠ ∅` gives `coverage(Σ.L(a).e₃) ≠ ∅` for every
`a ∈ dom(Σ.L)`, whence the first collapse gives `followlink(Σ, a, 3) ≠ ⟨⟩`. The
type selector thus never yields the empty-success `⟨⟩`; the empty-versus-invalid
collision this section forbids is reachable only at the non-type slots — `1`, `2`,
and any slot beyond `3`.
Nelson is firm that this is a *successful* answer, not a
failure: emptiness is "a first-class, valid state," the correct answer to a
valid question, and "a span that contains nothing today may at a later time
contain a million documents" (Q6, 4/25). An invalid selector — one naming no
existing slot — is a categorically different thing: it is not a value at all but
a domain violation, "there was no valid question to answer" (Q6).

The design demands these two outcomes be *distinguishable*. If an invalid
selector returned the empty span-set, a caller could not tell "this end
legitimately holds nothing (and may hold something tomorrow)" from "this request
was ill-formed (and never meant anything)." That confusion would destroy the
very guarantee that emptiness is a real, evolving state of a valid address. We
make the distinction an invariant:

> **F7 (EmptyVersusInvalid).** The empty span-set `⟨⟩` (a success, denoting
> `∅`) and the error value `⊥` (a domain violation) are distinct return
> categories, `⟨⟩ ≠ ⊥`. For a valid selector `1 ≤ i ≤ |Σ.L(a)|` over a link
> `a ∈ dom(Σ.L)` whose end `eᵢ` is empty, `followlink(Σ, a, i) = ⟨⟩`. For an
> invalid selector — `i < 1`, or `i > |Σ.L(a)|`, or `a ∉ dom(Σ.L)` —
> `followlink(Σ, a, i) = ⊥`. An implementation that collapses these two cases is
> incorrect.

This claim is sharpened by an instructive divergence in the implementation
evidence. Gregory finds that the implementation's FOLLOWLINK path, when the
requested end holds no content, takes a NULL return from its retrieval step and
propagates *failure* — it emits a protocol-level error rather than an empty
result (Q17, `sporgl.c:93` → `putrequestfailed`). This conflates the empty end
with the invalid request, exactly the collapse F7 forbids. Gregory also shows
that a sibling operation over the same store distinguishes them correctly,
returning empty on success (Q17). The lesson for the abstract specification is
clear: F7 is a genuine obligation that one real implementation fails to meet.
An alternative implementation must separate "valid end, presently empty" from
"no such end" — must return `⟨⟩` for the former and `⊥` for the latter — to
honor Nelson's design intent. The whitelist that rejects out-of-range selectors
at the boundary (Q12) is the correct mechanism for producing `⊥`; the error must
not also be produced for the empty-but-valid case.

We record the wp form of the valid/invalid boundary, which is simply the
negation structure of the precondition F0:

> `wp(followlink(a, i), result ≠ ⊥) ≡ a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)|`,
> and the complementary `wp(followlink(a, i), result = ⊥)` is the negation of
> that condition.

The wp that actually probes F7's empty/non-empty split asks when the result *is*
the empty span-set. Working backward from `R = ⟨⟩`: the call must first be
defined, else the result is `⊥ ≠ ⟨⟩`, supplying the two domain conjuncts; when
defined, F1 gives `coverage(R) = coverage(Σ.L(a).eᵢ)`. The two collapses close
the chain: the first, composed with F1, gives `R = ⟨⟩ ⟺ coverage(Σ.L(a).eᵢ) = ∅`,
and the second gives `coverage(Σ.L(a).eᵢ) = ∅ ⟺ Σ.L(a).eᵢ = ∅`. Hence

> `wp(followlink(a, i), R = ⟨⟩) ≡ a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ |Σ.L(a)| ∧ Σ.L(a).eᵢ = ∅`.

## Independence from content existence

One further property is forced by the same evidence. The selected end is defined
by *address*, and FOLLOWLINK returns address regions, not content. Whether
content or links actually exist at the covered addresses is irrelevant to the
operation. Nelson makes this explicit for the type end especially: "there is no
need for the presence of elements at the addresses specified. Link types may be
ghost elements" (4/45), and the system "does not actually look at what is stored
under the 'type' ... it merely considers the type's address" (4/44–4/45).
Gregory confirms the operation succeeds for an *orphaned* link whose endpoint
content is undiscoverable, reading the link's recorded endset directly without
consulting any document's arrangement (Q19).

> **F8 (ContentIndependence).** `followlink(Σ, a, i)` is defined and satisfies
> F1 whenever `a ∈ dom(Σ.L)` and `1 ≤ i ≤ |Σ.L(a)|`, irrespective of whether any
> address in `coverage(Σ.L(a).eᵢ)` currently holds content or a link in `Σ`. The
> result reports the recorded region; the existence of material at that region
> is a separate question the operation does not ask.

This places the precondition at its true minimum. FOLLOWLINK needs the link to
exist and the slot to exist — and nothing more. It does not need an open
document, a populated endpoint, or a reachable target.

## A boundary we must respect: the recorded end versus its resolution

The implementation evidence repeatedly describes FOLLOWLINK as returning
*V-positions* in a queried document, obtained by converting the recorded
addresses through that document's arrangement and silently dropping any address
the document does not currently reference (Q11, Q15, Q20). We must be precise
about what belongs to *this* operation and what does not.

The abstract operation specified here — "read one endset of a link by selector"
— yields the endset *the link records*: a span-set over tumbler space, exact by
F1, content-independent by F8, permanent by F5. The implementation's additional
step — projecting that recorded endset into the live arrangement of a particular
document, and filtering out addresses absent from that document's current
view — is a *separable concern*. It is the resolution of an endset against a
chosen document, and it is explicitly outside the scope of this note. We observe
that bundling the two has an observable cost the abstract specification helps us
name: the filtering means a resolved result can shrink relative to the recorded
end (Q15), and the same recorded end can resolve differently against different
documents (Q11). These are properties of *resolution*, not of FOLLOWLINK.
FOLLOWLINK's contract is with the recorded end; F1's exactness is exactness *to
what the link records*, which by F5 is invariant. The shrinkage Nelson allows in
Q3 — an end "shrinking" when its bytes are deleted — is likewise a property of
how the end renders into a current arrangement, not a change to the recorded
end, which persists by L12.

## A worked instance

The two claims a reader cannot check abstractly — F2 (a disconnected end forces
`|R| ≥ 2`) and F7 (valid-empty `⟨⟩` versus invalid `⊥`) — are worth discharging
against a specific link. Fix a document-level tumbler `d = 1.0.1.0.5` (`zeros(d)
= 2`), and let its content sub-allocator produce the element-level addresses
`aₖ = 1.0.1.0.5.0.1.k` for `k ≥ 1` (`zeros = 3`, subspace `s_C = 1`). Let
`a ∈ dom(Σ.L)` be a link of arity `N = 3` with

- `e₁ = {(a₃, δ(2, #a₃)), (a₇, δ(2, #a₇))}`,
- `e₂ = ∅`,
- `e₃ = {(τ, δ(1, #τ))}` for some type address `τ` (non-empty, as L3 requires).

*Checking F1 and F2 on the from-end.* Each span is canonical. By OrdinalShift
(ASN-0034), `a₃ ⊕ δ(2, #a₃)` increments the last component by 2, yielding `a₅`,
so `(a₃, δ(2, #a₃))` denotes the half-open interval `{t ∈ T : a₃ ≤ t < a₅}`.
This is a region of *all* of `T`, not a four-element set: it also contains, e.g.,
`a₃.1 = 1.0.1.0.5.0.1.3.1`, which satisfies `a₃ < a₃.1 < a₅` under T1. Restricted
to the emittable addresses `F` (ASN-0098), however, the interval contributes only
`{a₃, a₄}` (LP-Fin Corollary). Likewise `(a₇, δ(2, #a₇))` denotes
`{t ∈ T : a₇ ≤ t < a₉}`, contributing `{a₇, a₈}` within `F`. Hence
`coverage(e₁) = [a₃, a₅) ∪ [a₇, a₉)`, with `coverage(e₁) ∩ F = {a₃, a₄, a₇, a₈}`.
This coverage is disconnected: take `p = a₃`, `q = a₅`, `r = a₇`; under T1 (same
prefix, last components `3 < 5 < 7`) we have `p < q < r` with `p, r ∈ coverage(e₁)`
but `q ∉ coverage(e₁)` (the interval `[a₃, a₅)` excludes its upper endpoint `a₅`,
and `a₅ < a₇`). F1 demands `coverage(R) = [a₃, a₅) ∪ [a₇, a₉)`. Could a single
span suffice — `R = ⟨σ⟩`? It would need `a₃, a₇ ∈ ⟦σ⟧`; by span convexity (S0)
`⟦σ⟧` would then contain `a₅`, forcing `a₅ ∈ coverage(R)` and breaking the
equality with `coverage(e₁)`. So `|R| ≥ 2`, which is exactly F2; the witness
`R = ⟨(a₃, δ(2, #a₃)), (a₇, δ(2, #a₇))⟩` has `|R| = 2` and satisfies F1. This is
precisely the multi-region preservation Gregory's evidence records at
`orglinks.c:412–413`.

*Checking F7 on the to-end and on a bad selector.* The request
`followlink(Σ, a, 2)` names a valid slot (`1 ≤ 2 ≤ 3`) whose end is empty;
`coverage(e₂) = ∅`, so by the first collapse `followlink(Σ, a, 2) = ⟨⟩` — a
*success* denoting nothing. By contrast
`followlink(Σ, a, 4)` names no slot (`4 > |Σ.L(a)| = 3`) and returns `⊥`;
similarly `followlink(Σ, a, 0)` (with `0 < 1`) returns `⊥`, as does
`followlink(Σ, a'', i)` for any `a'' ∉ dom(Σ.L)`. The two outcomes are distinct
return categories (F7): `⟨⟩` says "this end is presently empty," `⊥` says "there
was no such end to read." This is the divergence the implementation evidence
flags — Gregory's path would propagate a protocol failure for the empty `e₂`
(`sporgl.c:93` → `putrequestfailed`), collapsing the first outcome into the
second, the very conflation F7 forbids.

## Synthesis

FOLLOWLINK is, abstractly, a projection. Given a link address and a slot
selector, it returns that slot's endset, measured by coverage, under three
primary commitments — F1, F4, F7 — atop the F0 definedness precondition, with
F2, F3, F5, F6, and F8 following as corollaries. It reads one end and discloses
only that end — the addresses it targets, the documents that the
document-bearing addresses among them structurally name, and the fact that the
link has at least that many slots. Each of these is a property any
faithful implementation must satisfy; the one implementation we have evidence for
satisfies F1–F6 and F8 and *fails* F7, which is exactly the kind of obligation an
abstract specification exists to make visible.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| F0 | `followlink(Σ, a, i)` is defined (returns a span-set) iff `a ∈ dom(Σ.L) ∧ 1 ≤ i ≤ \|Σ.L(a)\|`; else returns `⊥` | introduced |
| F1 | CoverageExactness: `coverage(followlink(Σ, a, i)) = coverage(Σ.L(a).eᵢ)` — neither over- nor under-coverage | introduced |
| F2 | DiscontiguityFaithfulness: if `coverage(Σ.L(a).eᵢ)` is disconnected, any F1-result has `≥ 2` spans (corollary of F1 and span convexity) | introduced |
| F3 | RepresentationInvariance: any two F1-results for the same `(Σ, a, i)` are denotationally equal; the contract binds coverage, not span decomposition or order (corollary of F1) | introduced |
| F4 | PureRead frame: `followlink` induces no state transition; `Σ.C`, `Σ.L`, `Σ.E`, every `Σ.M(d)`, and `Σ.R` are unchanged | introduced |
| F5 | TemporalDeterminism: for `Σ →* Σ'` with `a ∈ dom(Σ.L)`, results at the two states are coverage-equal (corollary of F1 and link permanence, LP13) | introduced |
| F6 | SlotConfinement: the result's *coverage* is a function of `coverage(Σ.L(a).eᵢ)` alone, turning on no `eⱼ`, `j ≠ i` (corollary of F1); representation-level non-exposure of other ends is an implementation property, not a contract guarantee | introduced |
| F7 | EmptyVersusInvalid: `⟨⟩ ≠ ⊥`; a valid selector over an empty end returns `⟨⟩` (success); an invalid selector returns `⊥` (error); collapsing them is incorrect | introduced |
| F8 | ContentIndependence: defined and exact whenever the link and slot exist, regardless of whether covered addresses currently hold content or links (corollary of F0 and F1) | introduced |

## Open Questions

What normal form, if any, must the returned span-set satisfy, given that coverage alone underdetermines the span decomposition?

Under what conditions may resolving the returned endset against a particular document's arrangement legitimately report fewer positions than the recorded end covers?

Across a serialization or protocol boundary — where F7's abstract `⟨⟩`/`⊥` distinction must be re-encoded as wire-level signals — what must the encoding guarantee so that a valid empty endset and an absent link remain distinguishable to a remote caller?

Must a selector naming a non-existent higher slot be observationally distinct from one naming an existing but empty slot, beyond both being errors versus successes?

What must the operation guarantee about the span-set when the selected end's coverage includes addresses in more than one document — is reporting them as one set, with document identity recoverable only by parsing each address, sufficient?
