> **ASN-0125 · The EDITLINK Operation — Editing Under Link Immutability** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0042 · Tumbler Ownership](../foundation/ASN-0042-tumbler-ownership.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0125-editlink-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0125: The EDITLINK Operation — Editing Under Link Immutability

*2026-06-12*

## The problem

A user holds a link and wants its endsets changed — an endpoint widened, a target
corrected, a type refined. Every other editable thing in the system has a place
where editing happens: a document's arrangement is the one component of state
revisable in place. A link has no such place. The substrate's invariant L12
(LinkImmutability, ASN-0043, ASN-0093) says that once a link exists, its address
persists and its value — the whole sequence of endsets — is permanently fixed:

> `(A Σ → Σ' : (A t : t ∈ dom(Σ.L) : t ∈ dom(Σ'.L) ∧ Σ'.L(t) = Σ.L(t)))`

So what can EDITLINK possibly denote? We are asked to be exact: what is
allocated when a link is "edited," and what record is made of the relationship
between what was and what is now? Must the original remain readable at its
address? Must a reader be able to recognize the edit as supersession rather
than as an unrelated new link — and by what mechanism is that relationship
expressed, among a field on the new link's structure, a version-of-link
addressing convention, a separate supersession link, or a typed relation? What
must any choice maintain about the original's continued resolution and
discoverability, about chains under repeated edits, about multiple editors in
conflict, and about a reader's ability to identify the current successor?

EDITLINK is a *derived composite*: one allocation that gives the
edited reading a fresh, permanent identity, and one allocation that makes the
relationship between old and new exist — as a first-class, owned, disputable
*claim* in a typed relation. The apparent menu of mechanisms collapses: a
"separate supersession link" and a "typed relation" are the same object seen
from two sides, and the genuinely distinct alternatives — a field in the
successor's value, a nesting convention in the address space — are each
eliminated by requirements we derive from the design intent.

## The substrate we build on

**Standing precondition (reachability).** Throughout, every state `Σ` ranges
over states reachable from the initial state under the sequential transition
order (ASN-0047, SequentialTransitionAxiom): transitions are atomic and totally
ordered, `→` is a single elementary transition, and `Σ →* Σ'` is a finite
(possibly empty) sequence of them drawn from valid composites.

The state is `Σ = (Σ.C, Σ.L, Σ.E, Σ.M, Σ.R)` (ASN-0047). The link store
`Σ.L : T ⇀ Link` binds link addresses to link values; a link value is a finite
sequence of `N ≥ 3` endsets with the type slot non-empty (L3, ASN-0043), each
endset a finite set of T12-well-formed spans (ASN-0034, ASN-0043). The
*coverage* of an endset is the union of its span denotations,
`coverage(e) = (∪ (s,ℓ) : (s,ℓ) ∈ e : {t : s ≤ t < s ⊕ ℓ})` (ASN-0043,
ASN-0098). For a link address `t`, `home(t) = N(t).0.U(t).0.D(t)` is the
document-level prefix recovered by field projection (ASN-0043), coinciding with
`origin(t)` on link addresses (ASN-0086, HomeOriginCoincidence). We fix the
subspace identifiers `s_C ≠ s_L` (ASN-0047, SubspaceConventionAxiom) and write
`δ(n, m)` for the ordinal displacement (ASN-0034). The elementary vocabulary is
ASN-0047's: `K.α`, `K.δ`, `K.λ`, `K.μ⁺`, `K.μ⁺_L`, `K.μ⁻`, `K.ρ`, with `K.μ~`
a named composite of `K.μ⁻` and `K.μ⁺`.

Above the substrate we take the typed-relation layer of ASN-0086 as given:
coverage classes `[K]` of admissible type endsets, the typed slices
`L_K^Σ = {(b, F, G) : b ∈ dom(Σ.L) ∧ |Σ.L(b)| = 3 ∧ Σ.L(b).e₁ = F ∧ Σ.L(b).e₂ = G ∧ coverage(Σ.L(b).e₃) = coverage(K)}`,
the designated retraction class `[R]` with `nullified(Σ)` and the active
subsets `A_K^Σ`, the total emission-address function `a_emit(Σ, d)`, the
operations `Emit_K`, `Observe_K` (views `hist`/`oper`), and `Nullify`.

**Vocabulary fact V (the L-frame inventory).** By inspection of the ASN-0047
transition contracts: every elementary transition other than `K.λ` carries the
frame clause `L' = L` (`K.α`: `L' = L`; `K.δ`: `L' = L`; amended `K.μ⁺`,
`K.μ⁺_L`, `K.μ⁻`: `L' = L`; `K.ρ`: `L' = L`; `K.μ~` inherits from its
decomposition). The one transition that writes the link store,
`K.λ(d, ℓ_f, v)`, has effect `L' = L ∪ {ℓ_f ↦ v}` under a binding precondition
that forces `ℓ_f ∉ dom(L)` (ASN-0093, FirstEmissionFreshness and
SubsequentEmissionFreshness). The link store admits exactly one kind of change:
extension at a fresh key.

**Layer transfer.** ASN-0086's results depend only on the link store `Σ.L` and
the document set `dom(M)`; both evolve identically under the full ASN-0047
vocabulary and under ASN-0086's `{K.σ, K.α, K.λ}` — the link store changes only
by `K.λ`'s fresh appends (Vocabulary fact V), and `dom(M)` is monotone (M1,
with `K.δ`'s document case playing `K.σ`'s role) — so those results hold at
full-vocabulary reachable states, over which we read "layer-reachable."

**K.λ-only composites are valid.** Both operations below are sequences of `K.λ`
steps alone, and a `K.λ`-only sequence is a valid ASN-0047 composite: its
coupling clauses hold vacuously — J0 because no `K.α` runs, so no content is
allocated (`dom(C') \ dom(C) = ∅`); J1★ because no content-subspace `K.μ⁺`
extends an arrangement, so no I-address is new to any content-subspace range;
and J1'★ because no `K.ρ` runs, so `R' = R` records no provenance. The output
state of each `K.λ` step is therefore reachable, so `assert_sup`'s `Σ'` and
`editlink`'s `Σ₁` and `Σ₂` are reachable states.

## The mutation postcondition is unachievable

We begin where a specification should: with the postcondition the user wants,
and the weakest precondition under which the available vocabulary establishes
it. Fix a reachable `Σ₀`, a link `a ∈ dom(Σ₀.L)`, and write
`ℓ₀ = Σ₀.L(a)`. The user's EDITLINK wants, for some intended new value
`w ≠ ℓ₀`:

> `R_mut ≡ a ∈ dom(L) ∧ L(a) = w`

Write `J ≡ a ∈ dom(L) ∧ L(a) = ℓ₀`; it holds at `Σ₀` by construction. That `J`
persists along every trace from `Σ₀` is not a fact this note must prove anew: it
is L12 (LinkImmutability, ASN-0043/0093) closed under `→*` — exactly LP13
(UnconditionalLinkPersistence, ASN-0098) — instantiated at `a`, giving

> `(A Σ' : Σ₀ →* Σ' : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = ℓ₀)`

Since `[J ⟹ ¬R_mut]` (a partial function has one value per key, and
`w ≠ ℓ₀`), and `J` holds at every state of every schedule from `Σ₀`:

**EL0 (MutationExclusion).** For every finite program `S` over the closed
elementary vocabulary, `wp(S, R_mut)` evaluated at `Σ₀` is `false`. The
postcondition "the link at `a` now reads `w`" is not unimplemented but
unimplementable; and dually, the original is readable at its own address, with
exactly its original value, in every future state, unconditionally.

> *Implementation note.* Gregory's backend is this fact in C: the FEBE surface
> has CREATELINK and the read/search calls but no UPDATELINK, MODIFYLINK, or
> DELETELINK opcode; a link orgl's endsets are written once, at creation, by
> the only call chain that ever writes them, and no function in the codebase
> relocates or rewrites a granfilade entry afterwards (Q11). The absence of
> the operation is not an unfinished corner of the implementation; it is the
> implementation of an absence in the design.

So EDITLINK, if it is to exist, denotes something else. We weaken the
postcondition to the strongest thing the vocabulary *can* establish. The first
weakening keeps the new reading and drops the address identity:

> `R₁ ≡ (E a' : a' ∈ dom(L) : L(a') = w)`

`R₁` is achievable — one `K.λ` — but it is transparently too weak: it is
equally established by an edit of `a` and by an unrelated creation that never
heard of `a`. The user's intent had two parts — *a new reading* and *its
standing as the revision of the old one* — and `R₁` captures only the first.
The candidate strengthening is:

> `R₂ ≡ R₁ ∧ "the pair (a, a') stands, in the state, in a relation recognizable as supersession"`

## An unasserted edit does not exist

Before asking where the relationship can be recorded, we must establish that it
*needs* recording — that nothing about performing the edit leaves a trace.

Each elementary transition, given its parameters, is a partial *function* of
the pre-state (the operation signature of ASN-0034's NoDeallocation; ASN-0047's
SequentialTransitionAxiom), and under the layer's emission rule the fresh
address is itself a function of state and home, `a' = a_emit(Σ, d_s)`
(ASN-0086). Now compare two descriptions of the same morning's work: "I edited
the link at `a`, producing the corrected value `ℓ'` homed at `d_s`," and "I
created a brand-new link with value `ℓ'` homed at `d_s`." Both denote the very
same transition instance, `K.λ(d_s, a_emit(Σ, d_s), ℓ')`, applied to the very
same state — hence they produce the *same* post-state `Σ₁`. Every predicate on
states agrees on the two; there is no predicate of `Σ₁` that holds iff `ℓ'`
"was derived from" `Σ.L(a)`. Intent is not a component of `Σ`, and what is not
in `Σ` does not exist for any observer of `Σ`.

**EL1 (IntentInvisibility).** Emission alone records no relationship: for the
post-state `Σ₁` of any single link allocation, every state predicate — and
hence every observation, present or future — is invariant under whether the
allocation was an edit of some existing link or an independent creation.
Consequently value resemblance carries no relational information: the store
legitimately holds distinct addresses with identical values (L11b,
NonInjectivity, ASN-0043), and the state after a resembling independent
creation coincides with the state after an unasserted "edit." Two links that
agree byte for byte are, to the system, exactly as related as two links that
share nothing.

This is a refusal, not a gap: relationships are made, not inferred (Q4). Under
this substrate the alternative is sharper than undesirable — it is
*undefinable*, the distinguishing fact being absent from the state.

So the edit must be two acts: produce the successor, and *say so*. Editing
under immutability is allocation plus assertion.

## Where no record can live

The assertion must be somewhere in `Σ`. We first survey the places the
substrate has already closed off; the surviving candidates are then assessed
against requirements.

**EL2 (NoInPlaceCarrier).** In every reachable state:

*(a) Not in the original's value.* `Σ.L(a)` is fixed by L12 from the moment of
creation. No "superseded-by" annotation can ever be attached to `a`.

*(b) Not appended to the successor's value later.* The same invariant binds the
successor the instant it exists: its slots are fixed at emission.

*(c) Not in the address relation between them.* One might hope the successor's
address could nest under the original's — a version-of-link convention
analogous to document versioning, `a' = inc(a, 1)`. The tumbler algebra admits
such addresses (`zeros(a) = 3` permits `inc(a, 1)` with T4 preserved, TA5a,
ASN-0034), but the substrate never allocates them: every allocated link address
is an emission of its home document's flat sibling chain `A_L(d)` — first
emission `[d.0.s_L.1]` with element-field depth `#E = 2`, successors by
`inc(·, 0)` which preserves length (FirstEmission, ChainDiscipline, TA5(c),
ASN-0093) — and at every reachable state the homed links form a contiguous
initial segment of that chain (ChainMembershipForOrigin, ASN-0093). So every
allocated link address has `#E = 2` exactly, while a nested version-of-link
address would need `#E ≥ 3`; stronger, `dom(Σ.L)` is a tumbler-prefix antichain
(R0a, FlatLinkDomain, ASN-0086) — no allocated link address prefixes another.
The address relation between any two allocated links carries exactly two
readable facts: whether they share a home (T6-decidable from the prefixes), and,
within one home, their emission order (T9, ASN-0034). Neither is semantic: the
first names a registry, the second a time of arrival at it.

*(d) Not in an index marker.* There is no further component to carry a flag:
the stored entities are exhausted by `dom(Σ.C) ∪ dom(Σ.L)` (L14, DualPrimitive,
ASN-0043); the entity set `E` holds organizational addresses with no payload;
the provenance relation `R` holds (content-address, document) pairs whose
precondition `a ∈ dom(C)` excludes link targets outright (K.ρ, ASN-0047, with
SD store disjointness); arrangement entries are V→I bindings within one
document. The stores are append-only maps of values; there is no status field
anywhere, and the only systematic asymmetry between two link entries is their
addresses — case (c).

> *Implementation note.* All four closures are visible in udanax-green. The
> link orgl contains exactly three endsets at fixed internal positions and
> nothing else — no predecessor slot, no spare subspace (Q14). Successor links
> are forced to flat siblings `2.N+1` by the molecule allocator's
> `tumblerincrement(lowerbound, 0, 1)`; the nesting allocator exists only for
> documents and is unreachable from link creation (Q13). And the spanfilade
> holds the old and new links' entries as structurally identical, equally live
> records — the one attempt to filter by version sits disabled behind
> `if (FALSE /* trying to kluge links followable thru versions */)` (Q16).

The record, then, must be a *freshly allocated entity*. The question is of what
kind.

## What the record must satisfy

The consultation record yields seven requirements on any carrier of the
supersession relationship.

- **RQ1 (Post-hoc assertability).** The relationship must be assertable at any
  state where both endpoints exist — not only at the successor's creation.
  Relationships are discovered late, asserted by third parties, repaired after
  omission; a carrier writable only at birth cannot serve them.
- **RQ2 (Open authorship).** Any principal with a home document may assert.
  The same mechanism that lets an outsider claim, from outside a document, that
  its author is really someone else (Q4, Q5) must admit an outsider's claim
  that *their* revision supersedes another's link (Q9).
- **RQ3 (Attribution).** The asserter must be decidable from the record alone.
  A claim nobody owns is not a claim; it is a rumor with system privileges.
- **RQ4 (Non-destructive disputability).** A claim must be withdrawable from
  current standing, and contestable, without erasing it or either endpoint.
  The history of claims is itself permanent record — the design's standing
  defense against silent rewriting (Q3).
- **RQ5 (Endpoint frame).** Asserting must modify neither endpoint. The
  original's owner keeps the original; the successor's content is not hostage
  to the claim about it.
- **RQ6 (Decidable specificity).** The relationship must be recognizable as
  supersession *specifically* — distinguishable from comment, counterpart, or
  coincidence — by the substrate's interpretation-free mechanisms: address and
  coverage comparison, never content exegesis. And it must be refinable: a
  correction is not a restyling, and the vocabulary must be able to grow
  subtypes.
- **RQ7 (Plurality).** Arbitrarily many claims over the same endpoints,
  including mutually contradictory ones, must be co-representable. Competing
  claims are resolved socially, never structurally (Q9).

**EL3 (RelationSpaceNecessity).** Under this substrate, any carrier satisfying
RQ1–RQ7 is a freshly allocated link-store entity, distinct from both endpoints,
referencing each endpoint by address through its endsets, and bearing its kind
as the coverage class of a designated slot — that is, a typed link-to-link
tuple. The derivation:

RQ1 and RQ2 require the carrier to be created by a transition, at arbitrary
later states, by arbitrary principals — so it is a fresh store entity, and the
vocabulary offers exactly two entity-creating store writers, `K.α` (content)
and `K.λ` (links); `K.δ` and `K.ρ` were closed off in EL2(d). RQ6 eliminates
the content store: a claim encoded as content bytes ("this supersedes that,"
written down in a document) has, to the substrate, no structure beyond an
address and an origin — its relational content lives in `Val`, which nothing in
the system interprets; type machinery reads slot-3 *coverage* only (L8,
TypeByAddress, ASN-0043), and Gregory's backend never dereferences a type to
look at what is stored there (Q19). So the carrier is a link. RQ1 makes it a
link *other than the successor* — a third entity — since the successor's slots
close at its birth (EL2(b)). Its reference to the endpoints must be
substrate-visible, and the one mechanism links have for referencing anything is
endset coverage; endsets may target link addresses (L4(c), ASN-0043), with the
unit-depth span at an address as the canonical reference (L13, R5). RQ6 then
fixes how the kind is carried: the only interpretation-free, decidable,
refinable kind mechanism in the system is the coverage class of the type slot
(L8; decidability by CoverageEqualityDecidable, ASN-0086; refinement by prefix
containment, L10). RQ4 is satisfied precisely because the carrier has its own
address: it can be individually targeted — disputed by further links, retracted
from operative standing by the layer's `Nullify` — while L12 holds it, and both
endpoints, in the permanent record. RQ3 is its home prefix (T4b projection,
decidable by T6) — attribution to the allocating document, from the address
alone. RQ5 is `K.λ`'s frame. RQ7 is
freshness: every claim is a new address; nothing collides, merges, or ranks.

*The menu was shorter than it looked.* "A separate supersession link" and "a
typed relation distinct from these" are the same architecture: under L8 a link
*is* typed, by its third endset's coverage, and a typed relation's tuples *are*
links (ASN-0086, TypedRelation). The address-space candidate is closed outright
by EL2(c).

## The supersession relation

**Df-CLS (SupersessionClass).** Fix a coverage class `[K_sup]`,
`K_sup ∈ T_admissible`, with `coverage(K_sup) ≠ coverage(R)` — distinct from
the retraction class (ASN-0086). Write `S^Σ := L_{K_sup}^Σ` for the
*supersession slice* at `Σ` — the historical record of claims — and
`A_sup^Σ := A_{K_sup}^Σ = {(b, F, G) ∈ S^Σ : b ∉ nullified(Σ)}` for its
operative subset. We call the members of `S^Σ` *claims*.

**Df-DIR (ClaimDirectionality).** For a claim `(b, F, G) ∈ S^Σ`: the from-set
`F` covers the *superseding* link, the to-set `G` the *superseded* — read "`F`
replaces `G`." This aligns with the layer's RetractionDirectionality
(ASN-0086): the to-side is the side acted upon. A withdrawal with no
replacement is not a degenerate supersession but a retraction, class `[R]`;
the two acts remain distinct relations, and asserting the first never performs
the second.

**Df-DISC (EditDiscipline).** A state `Σ` is *edit-disciplined* iff (i) it is
unit-depth-retraction-disciplined (ASN-0086) and (ii) every claim conforms to
the *claim schema*:

> `(A (b, F, G) ∈ S^Σ : (E x, y ∈ dom(Σ.L) : x ≠ y ∧ F = {(x, δ(1, #x))} ∧ G = {(y, δ(1, #y))}))`

— both endsets are canonical unit-depth spans at link-store addresses, and the
claim is irreflexive. A layer is edit-disciplined iff every state it reaches
is. (Self-supersession `x = y` is excluded as vacuous; cycles of length ≥ 2 are
deliberately *not* excluded — they are reverts.)

**Df-LAY (EditingLayer).** The *editing layer* issues exactly the operations
`{assert_sup, editlink, Nullify}` (the first two defined below; `Nullify` from
ASN-0086), together with the substrate's link-framing transitions
`{K.α, K.δ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` (and `K.μ~`, their composite) and the
*bare* `K.λ` — a standalone link allocation the layer issues directly, as
distinct from the `K.λ` step internal to `editlink` (which may itself carry
`[K_sup]` under `DC`) — *confined to original-link creation*: emission whose
slot-3 coverage is neither `coverage(K_sup)` nor `coverage(R)`. Its one
*discipline commitment* routes every emission into a disciplined class through
that class's disciplined operation: every `[K_sup]` emission through
`assert_sup` or `editlink` (under `DC`), every `[R]` emission through `Nullify`.
A state is *editing-layer-reachable* (cf. ASN-0086, LayerReachable) iff it is
reached from the initial state `Σ₀` by a finite sequence of editing-layer
operations.

**EL-DM (DisciplineMaintenance).** Every editing-layer-reachable state is
edit-disciplined — so the "at disciplined `Σ`" conditionals below are
non-vacuous.

*Base.* The initial state `Σ₀` (ASN-0047) has `L₀ = ∅`, hence
`S^{Σ₀} = L_{K_sup}^{Σ₀} = ∅` and `L_R^{Σ₀} = ∅`; both clauses of Df-DISC hold
vacuously over an empty link store. `Σ₀` is the empty-store boundary case, and it
is what makes a disciplined state reachable at all.

*Step.* Edit-discipline is the conjunction of ASN-0086's unit-depth retraction
discipline (clause i, a property of the `[R]`-slice `L_R^Σ`) and the claim schema
(clause ii, a property of the `[K_sup]`-slice `S^Σ`); a transition out of a
disciplined `Σ` preserves it iff it adds no non-unit-depth `[R]` tuple and no
schema-violating `[K_sup]` claim, and disturbs no surviving tuple's conformance.
We discharge this for each editing-layer operation.

- *L-framing transitions and original-creating bare `K.λ`.* By Vocabulary fact V
  the six framing transitions (and `K.μ~`) carry `L' = L`, leaving `S^Σ` and
  `L_R^Σ` identical, so both clauses transfer verbatim. The bare
  original-creating `K.λ` extends `dom(L)` at one fresh key whose slot-3 coverage
  inhabits neither disciplined class, adding nothing to either slice; every prior
  tuple keeps its value (L12) and its witnesses (the `x, y` of a claim, the `t`
  of a retraction) remain in the grown `dom(L)`, so no surviving tuple's
  conformance is disturbed. Both clauses preserved.
- *`Nullify`.* Emits exactly one `[R]`-class tuple, with to-set
  `{(t, δ(1, #t))}` whose target `t` is admissible under ASN-0086's P-tgt in
  either branch — a pre-existing `t ∈ dom(Σ.L)` (P1), or the self-emit target
  `t = a_emit(Σ, d_retr)`, the fresh emitter itself — and in both cases
  `t ∈ dom(Σ'.L)` at the post-state (P1 targets persist by
  `dom(Σ.L) ⊆ dom(Σ'.L)`; the self-emit target enters the store at this very
  emission), which is the state at which ASN-0086's unit-depth retraction schema
  evaluates its membership, so clause (i) is preserved; it deposits no `[K_sup]`
  claim, leaving clause (ii) untouched.
- *`assert_sup`.* EL6(v): `Σ'` is edit-disciplined when `Σ` is.
- *`editlink`.* EL7(vi): `Σ₂` is edit-disciplined when `Σ` is.

By induction over the editing-layer operations, edit-discipline holds at every
editing-layer-reachable state. ∎

**EL4 (SingleTarget).** Each *schema-conforming* claim determines its endpoints
uniquely — and the argument is per-claim, invoking no whole-state discipline
hypothesis. For `e = (b, F, G) ∈ S^Σ` whose endsets meet the Df-DISC(ii) form
with witnesses `x, y` (so `F = {(x, δ(1,#x))}`, `G = {(y, δ(1,#y))}`,
`x, y ∈ dom(Σ.L)`, `x ≠ y`):

> `coverage(F) ∩ dom(Σ.L) = {x}` and `coverage(G) ∩ dom(Σ.L) = {y}`.

*Proof.* `coverage({(x, δ(1, #x))}) = {t : x ≼ t}` (PrefixSpanCoverage,
ASN-0043); for `t ∈ dom(Σ.L)` with `x ≼ t`, the antichain R0a forces `t = x`.
Both facts are properties of the single claim `e` and of `dom(Σ.L)` at the
ambient reachable state — no *other* claim need conform — so the hypothesis is
schema-conformance of `e` alone, not edit-discipline of `Σ`. ∎  We may therefore
write `addr(e) = b`, `new(e) = x`, `old(e) = y` as total accessors on any
schema-conforming claim, at any reachable state. Write
`Ŝ^Σ = {e ∈ S^Σ : e is schema-conforming}` for the schema-conforming claims; at
an edit-disciplined state every claim conforms, so `Ŝ^Σ = S^Σ`.

**Df-SUCC (Successor relations).** At any state `Σ`, ranging over the
schema-conforming claims `Ŝ^Σ` (EL4) — on which `old`/`new`/`addr` are total at
*every* reachable state, not only disciplined ones:

> `succ_h(Σ) = {(old(e), new(e)) : e ∈ Ŝ^Σ}`  — the historical relation;
> `succ_o(Σ) = {(old(e), new(e)) : e ∈ Ŝ^Σ ∧ addr(e) ∉ nullified(Σ)}`  — the operative relation.

Both are finite (L-fin) relations on `dom(Σ.L)`, with
`succ_o(Σ) ⊆ succ_h(Σ)`; at editing-layer states `Ŝ^Σ = S^Σ` (EL-DM), so the
comprehensions range over the whole supersession slice.

**EL5 (RecordMonotonicity).** For every `Σ →* Σ'`:

*(a)* `S^Σ ⊆ S^{Σ'}`, `Ŝ^Σ ⊆ Ŝ^{Σ'}`, and `succ_h(Σ) ⊆ succ_h(Σ')`. The slice
inclusion is R3 (TypedSliceMonotonicity, ASN-0086) at `[K_sup]`, lifted across
`→*` by finite composition; schema-conformance rides along, being
value-and-domain-determined — a conforming claim's witnesses satisfy
`x, y ∈ dom(Σ.L) ⊆ dom(Σ'.L)`, so it stays conforming at `Σ'` with the same
`old`/`new`. Claims accumulate; none is ever lost.

*(b)* `nullified(Σ) ⊆ nullified(Σ')`. The `[R]`-slice likewise only grows, so
nullification is one-way (R6a, ASN-0086): a claim once retracted from operative
standing never silently regains it (re-assertion is a *new* claim at a fresh
address — the shape of R6c).

*(c)* `succ_o` is neither monotone nor antitone: emission adds operative pairs
(EL6), `Nullify` removes them. The operative relation is the one revisable
view; the historical relation is the unrevisable record. This split — and not
any deletion — is how the design reconciles "claims can be wrong" with "nothing
is ever erased."

## The operations: assertion, and the edit composite

The assertion operation is an instance of the layer's emission, specialized to
the class and the schema.

**ASSERTop (assert_sup).** For `x, y ∈ dom(Σ.L)` with `x ≠ y` and
`d_a ∈ dom(Σ.M)`:

> `assert_sup(x, y, d_a) ≜ Emit_{K_sup}(Σ, d_a, {(x, δ(1, #x))}, {(y, δ(1, #y))})`

— one `K.λ` at home `d_a`, emitting the claim "`x` supersedes `y`" at the fresh
address `b = a_emit(Σ, d_a)`. The spans are T12-well-formed (`Pos(δ(1, #x))`;
action point `#x ≤ #x`), the slots are endsets, the arity is 3, and slot 3 is
`K_sup ≠ ∅`, so `K.λ`'s L3 precondition is discharged.

**EL6 (AssertionContract).** When invoked at a reachable `Σ` satisfying its
precondition, `assert_sup(x, y, d_a)` yields `Σ'` with:

*(i) Allocation.* Exactly one fresh address: `b ∉ dom(Σ.L) ∪ dom(Σ.C)`
(emission freshness, ASN-0093), `home(b) = d_a`.

*(ii) Record.* `e_b = (b, {(x, δ(1,#x))}, {(y, δ(1,#y))}) ∈ S^{Σ'}` with
`new(e_b) = x`, `old(e_b) = y`; hence `(y, x) ∈ succ_h(Σ')`.

*(iii) Active at birth.* If `Σ` is edit-disciplined, `b ∉ nullified(Σ')`, so
`(y, x) ∈ succ_o(Σ')`. This is ASN-0086's wp Case 2 under its disciplined
simplification: the pre-existing-retraction conjunct holds vacuously at
disciplined states, and `K_sup ≁ R` discharges the self-nullification guard.

*(iv) Frame and activity.* `Σ'.C = Σ.C`, `Σ'.M = Σ.M`,
`Σ'.E = Σ.E`, `Σ'.R = Σ.R`; every prior link-store entry — `x` and `y` in
particular — is unchanged. *Unconditionally,*
`nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)`: the lone new tuple
has slot-3 coverage `coverage(K_sup) ≠ coverage(R)`, so the `[R]`-slice does not
grow and the nullification status of *no pre-existing address* changes — the
superseded `y` is exactly as active as before. *Under edit-discipline on
`Σ`,* the full `nullified(Σ') = nullified(Σ)` follows: the only candidate new
member is the fresh `b`, and the unit-depth retraction discipline together with
the antichain R0a discharges wp Case 2's third conjunct (ASN-0086) — every
pre-existing `[R]`-tuple's to-coverage is a unit-depth subtree rooted at a single
*existing* link address, and the fresh `b ∉ dom(Σ.L)` is prefix-incomparable to
each (R0a at `Σ'`), so no such to-coverage reaches `b` and `b ∉ nullified(Σ')`.

*(v) Discipline and permanence.* `Σ'` is edit-disciplined when `Σ` was:
`assert_sup` emits one `[K_sup]` claim
`e_b = (b, {(x, δ(1,#x))}, {(y, δ(1,#y))})` with `x ≠ y` (precondition) and
`x, y ∈ dom(Σ.L) ⊆ dom(Σ'.L)`, which is exactly the claim-schema form of
Df-DISC(ii), so clause (ii) is preserved; it adds no `[R]` tuple, so clause (i)
is untouched, and every prior tuple keeps its value (L12) — so no surviving
tuple's conformance is disturbed. And at every `Σ' →* Σ''`, `e_b ∈ S^{Σ''}` with
value fixed and `(y, x) ∈ succ_h(Σ'')` (EL5a).

The edit operation is now one allocation ahead of the assertion.

**EDITop (editlink).** For `a ∈ dom(Σ.L)`, a target value `ℓ' ∈ Link`
(L3-conforming), and homes `d_s, d_a ∈ dom(Σ.M)`:

> `editlink(a, ℓ', d_s, d_a) ≜`
> `  a' := a_emit(Σ, d_s);  Σ₁ := K.λ(d_s, a', ℓ');`
> `  (Σ₂, b) := assert_sup(a', a, d_a) at Σ₁;`
> `  return (Σ₂, a', b)`

with the *discipline-conformance precondition* `DC(ℓ')` — a value-level
predicate whose witnesses are drawn from the editlink pre-state `Σ`:

> `coverage(ℓ'.e₃) ≠ coverage(R)`, and if `|ℓ'| = 3 ∧ coverage(ℓ'.e₃) = coverage(K_sup)` then
> `(E x, y ∈ dom(Σ.L) : x ≠ y ∧ ℓ'.e₁ = {(x, δ(1, #x))} ∧ ℓ'.e₂ = {(y, δ(1, #y))})`.

The `[K_sup]` clause is the claim schema of Df-DISC(ii); its leading conjunct
excludes a retraction-class successor, because retraction is `Nullify`'s office
(ASN-0086) and editlink is supersession — a new *reading* claimed to supersede
the old. `assert_sup`'s precondition is discharged at `Σ₁`: `a' ∈ dom(Σ₁.L)`
by the emission, `a ∈ dom(Σ₁.L)` by monotonicity, `a' ≠ a` by freshness,
`d_a ∈ dom(Σ₁.M)` by M1.

**EL7 (EditContract).** When invoked at a reachable `Σ` satisfying its
precondition, `editlink(a, ℓ', d_s, d_a)` yields `Σ₂` with:

*(i) What is allocated.* Exactly **two** fresh link-subspace addresses — the
successor `a'` on `A_L(d_s)` and the claim `b` on `A_L(d_a)` — pairwise
distinct from each other and from everything prior. No content address, no
entity, no provenance entry, no arrangement change: `Σ₂.C = Σ.C`,
`Σ₂.M = Σ.M`, `Σ₂.E = Σ.E`, `Σ₂.R = Σ.R`. (The successor's unedited slots are
*values* — span sets coverage-equal to the original's — not new content;
re-using them allocates nothing.)

*(ii) The new reading.* `Σ₂.L(a') = ℓ'`, `home(a') = d_s` — postcondition `R₁`
achieved with a permanent, fresh identity. The successor is *born unlisted*:
both composite steps are `K.λ`, neither touches `M` (so `Σ₂.M = Σ.M`), and the
fresh `a'` lies in no arrangement range — `ran(Σ₂.M(d)) = ran(Σ.M(d)) ⊆
dom(Σ.C) ∪ dom(Σ.L)` (S3★), which `a'` is fresh against — so `a'` appears in no
document's current arrangement: `a' ∉ ran(Σ₂.M(d))` for every `d`. It is
therefore auto-listed by no document; seating it under `home(a') = d_s` is a
separate `K.μ⁺_L` act, not part of the edit. How the successor is then
discovered — from a document's current view, and from the record independently
of any view — is characterised in EL11.

*(iii) The relationship.* `(a, a') ∈ succ_h(Σ₂)`, witnessed by the claim at
`b`; at edit-disciplined `Σ`, also `(a, a') ∈ succ_o(Σ₂)` — postcondition `R₂`
achieved in the one sense the substrate admits: as an owned, addressed,
class-marked statement.

*(iv) The frame on the original.* `Σ₂.L(a) = Σ.L(a)` unconditionally (L12) — the
original's value is permanent — and its listing is untouched (both steps frame
`M`). For activity, the supersession step deactivates nothing
(`nullified(·) ∩ dom(Σ.L)` is preserved across step 2, EL6(iv)); and the full
frame `nullified(Σ₂) = nullified(Σ)` holds under edit-discipline on `Σ`. By
`DC(ℓ')` the successor is not of retraction class, so step 1 emits no `[R]`-tuple
and nullifies nothing; and neither fresh address `a'` nor `b` is caught by any
pre-existing `[R]`-tuple's unit-depth to-coverage (freshness + R0a, the wp
Case 2 argument of EL6(iv) applied at each of the two emissions, the
intermediate `Σ₁` disciplined by EL7(vi) below).

*(v) Permanence.* At every `Σ₂ →* Σ₃`: `a`, `a'`, `b` all persist with fixed
values, and `(a, a') ∈ succ_h(Σ₃)`.

*(vi) Discipline preservation.* `Σ₂` is edit-disciplined when `Σ` is — this is
what `DC(ℓ')` secures, and it is what licenses chaining edits. The composite
reaches `Σ₂` through `Σ₁` in two steps. Step 1, `editlink`'s internal
`K.λ(d_s, a', ℓ')` (not the *bare* `K.λ`, which Df-LAY confines to
original-link creation), preserves Df-DISC: every prior claim keeps its
witnesses (`x, y ∈ dom(Σ.L) ⊆ dom(Σ₁.L)`, values fixed by L12), every prior retraction
likewise persists, and the one new value `ℓ'` at `a'` is governed by `DC(ℓ')`,
whose witnesses are pinned at the pre-state `dom(Σ.L)`. The conformance
transfers across the emission by `dom(Σ.L) ⊆ dom(Σ₁.L)` (Vocabulary fact V —
`K.λ` only extends the store at the fresh key `a'`). The successor `a'` is a
claim at `Σ₁` — a member of `S^{Σ₁}`, which ASN-0086 restricts to arity-3
tuples — iff `|ℓ'| = 3 ∧ coverage(ℓ'.e₃) = coverage(K_sup)`, which is exactly
`DC`'s schema guard. In that case `DC(ℓ')` supplies `x, y ∈ dom(Σ.L) ⊆ dom(Σ₁.L)`
with `x ≠ y`, `ℓ'.e₁ = {(x, δ(1,#x))}`, `ℓ'.e₂ = {(y, δ(1,#y))}`, so the new
claim at `a'` conforms to Df-DISC(ii) *at `Σ₁`* (clause ii preserved).
Otherwise — `coverage(ℓ'.e₃) ≠ coverage(K_sup)`, or that coverage with
`|ℓ'| > 3` — `a' ∉ S^{Σ₁}`, so Df-DISC(ii), a quantification over `S^{Σ₁}`,
holds vacuously on `a'` (clause ii again preserved), and `DC(ℓ')`'s leading
conjunct gives `coverage(ℓ'.e₃) ≠ coverage(R)`, so `ℓ'` is no `[R]`-tuple. In
every case step 1 adds no `[R]`-tuple — when `a'` *is* a claim because
`coverage(K_sup) ≠ coverage(R)` (Df-CLS), otherwise by the leading conjunct —
leaving the retraction-slice clause (i) undisturbed. So both clauses of
edit-discipline survive at `Σ₁`. Step 2, `assert_sup`, preserves
edit-discipline by EL6(v). Hence `Σ₂` is edit-disciplined.

Three remarks delimit the operation's generality. *First*, a value-identical
successor `ℓ' = Σ.L(a)` is a legitimate edit (re-homing a link, re-attributing
it) wherever `DC` admits it — the claim, not a value diff, is what makes it an
edit. *Second*, neither `d_s` nor `d_a` is constrained relative to `home(a)`:
a third party edits another's link by exactly this composite, homing successor
and claim under their own prefix — ownership of the original never moves, and
attribution (EL8) keeps the claim visibly the third party's. Editing-by-fork
is not a special case; it is the same operation invoked by someone else.
*Third*, a *revert* allocates no successor at all: to assert that the original
supersedes its replacement is `assert_sup(a, a', d)` — one claim, nothing else
— since the "new" value, being the old one, already exists at its permanent
address.

**Remark (no enforceable coupling).** Could the substrate *require* the
assertion whenever an emission "is an edit"? A coupling constraint in the style
of ASN-0047's J-clauses is evaluated between composite-boundary states, so its
trigger must be a state predicate — and by EL1 the fact distinguishing an edit
from an independent creation is absent from the state. No J-clause can mention
intent. The completeness of the supersession record is therefore a protocol
property of the editing layer, not a substrate invariant; what the substrate
guarantees is only the standing of declarations once made — permanent,
attributed, decidable.

**EL8 (ClaimStanding).** For every claim `e ∈ S^Σ` in a disciplined state:

*(a)* it is permanent in membership and value (EL5a);
*(b)* it is attributed: `home(addr(e)) = N(addr(e)).0.U(addr(e)).0.D(addr(e))`
is computable from the address alone by field projection (T4b), decidably (T6),
and identifies the document under whose prefix the claim was allocated — the
claim is signed, from the record alone, by the home its address names. This is
the whole of what RQ3 demands and the whole of what the substrate state
`Σ = (C, L, E, M, R)` supplies: it carries no principal set, so resolving a
home further to a named owner is the office of an ownership layer (ASN-0042)
overlaid on the substrate, not a function of `Σ`;
*(c)* it is open: the schema imposes no relation among `home(addr(e))`,
`home(old(e))`, `home(new(e))` — first-party, second-party, and third-party
claims are structurally identical, differing only in their visible provenance;
*(d)* it is itself addressable: `addr(e) ∈ dom(Σ.L)`, so claims can be the
targets of endsets (L4(c)) — endorsed, disputed, commented, retracted
(`Nullify`), or themselves edited (`editlink` applies to a claim, `DC`
permitting) — with no new machinery.

## The original after the edit: three independent axes

The question "must the original remain readable at its original address?"
dissolves, under this model, into three questions with three different answers
— and the precision matters, because the design record gives a nuanced verdict
(Q2: links are "non-destructibly recoverable," not "forever live in the current
view") that the axes reproduce exactly.

**Df-LISTED.** Write `listed(t, d, Σ)` for `(t, d) ∈ Contains(Σ)` (ASN-0047,
CurrentContainment) — equivalently `t ∈ ran(Σ.M(d))`, since `dom(Σ.M) = E_doc`
(M1) discharges the `d ∈ E_doc` conjunct for every `d` we range over — `t`
appears in `d`'s current arrangement. The structural fact this naming lets us
state is the one Df-LISTED actually adds: for a link, only its home can list it
— a link-subspace image has `origin = d` (CL-OWN, ASN-0047, with
HomeOriginCoincidence), and a content-subspace image lies in `dom(C)` (S3★),
which is disjoint from `dom(L)` (SD).

**EL9 (ThreeAxes).** For a link `a ∈ dom(Σ.L)`:

*(1) Resolution — permanent and unconditional.*
`(A Σ' : Σ →* Σ' : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a))` (EL0's invariant `J`).
Nothing gates the lookup: no arrangement state, no activity status, no
provenance appears in it. *Readable at its original address, exactly as it
was* — an invariant, forever, for every link, superseded or not.

*(2) Listing — mutable in both directions.* The home registry is current view,
not record — but de-listing is not surgical, because `K.μ⁻` retains a position
*prefix* (its link-subspace retention set is `{[s_L, k] : 1 ≤ k ≤ n'_{s_L}}`,
ASN-0047 per-subspace scope) and so can drop only a *suffix* of the listed
chain. *Construction (de-list `a` at position `[s_L, j]`).* Let `a` sit at
`[s_L, j]` in `V_{s_L}(d) = {[s_L, 1], …, [s_L, n]}`, with survivors
`[s_L, j+i] ↦ ℓ_{j+i}` for `1 ≤ i ≤ n − j` above it. Apply `K.μ⁻` with
`n'_{s_L} = j − 1` (content retained in full, so the link subspace strictly
contracts, satisfying `K.μ⁻`'s precondition): this necessarily drops `a`
*together with the whole suffix* `ℓ_{j+1}, …, ℓ_n`, retaining the prefix
`{[s_L, 1], …, [s_L, j−1]}`. Then re-seat the `n − j` survivors in their old
order by successive `K.μ⁺_L`. Each `ℓ_{j+i}` satisfies that operation's
precondition — `origin(ℓ_{j+i}) = d` (it was homed at `d`, CL-OWN),
`ℓ_{j+i} ∉ ran(M(d))` (just dropped, not yet re-added), and the assigned
`v_ℓ = shift(max(V_{s_L}(d)), 1) = [s_L, j+i−1]` (the `j = 1` leading survivor
taking `K.μ⁺_L`'s first-position branch to the same `[s_L, 1]`). So each survivor
lands *one position below* its prior seat, and D-SEQ★ shapes the result into
`{[s_L, k] : 1 ≤ k ≤ n−1}`: `a` is gone from `ran(M(d))` and the suffix has slid
down by one — a position re-binding consistent with EL10. (When `a` is last or
only, `j = n`: the suffix is empty and no survivor is re-seated.) Re-listing `a`
is then one `K.μ⁺_L` (`origin(a) = d ∧ a ∉ ran(M(d))` holds after de-listing).
De-listing is the model's "deleted link": gone from the current view, untouched
in the record.

*(3) Activity — monotone downward, per claim.* `active(a, Σ) ≡ a ∉
nullified(Σ)` can only fall (EL5b), by an explicit, itself-permanent,
itself-attributed retraction tuple; restoration is re-assertion at a fresh
address, never reinstatement in place (R6c).

The axes are independent, and — by EL6(iv) — *superseding moves none of them*.
An edit, as such, leaves the original resolvable, and its listing and activity
exactly as they were. Each
demotion is a separate act by an authorized party, separately attributed,
separately permanent in the record. The composite "supersede and retire" is
available; it is never implied.

> *Implementation note.* Axis (1) is Gregory's Q17: resolving a link orgl by
> its address is a pure granfilade descent — no surviving POOM entry required,
> no opened home document, the BERT gate explicitly bypassed
> (`NOBERTREQUIRED`). Axis (2) is his reverse-orphan state: DELETEVSPAN on the
> link's `2.x` position removes the registry entry and nothing else.

**EL10 (PositionEpochality).** Listing positions are not identifiers. There
exist reachable `Σ →* Σ' →* Σ''`, a document `d`, a position `v`, and links
`ℓ₁ ≠ ℓ₂` — both permanently resolvable throughout — with

> `Σ.M(d)(v) = ℓ₁,  v ∉ dom(Σ'.M(d)),  Σ''.M(d)(v) = ℓ₂.`

*Construction.* Let `d` list two links, `V_{s_L}(d) = {[s_L,1], [s_L,2]}` with
`[s_L,2] ↦ ℓ₁`, and let `ℓ₂` be homed at `d` but unlisted (a bare `K.λ`).
Apply `K.μ⁻` with link-subspace retention `n'_{s_L} = 1` (content retained in
full); then `K.μ⁺_L` for `ℓ₂`: the substrate assigns
`v_ℓ = shift(max(V_{s_L}(d)), 1) = shift([s_L,1], 1) = [s_L,2]` — the very
position `ℓ₁` vacated, now bound to `ℓ₂`. ∎

Addresses, by contrast, never re-bind: every allocation is fresh (Vocabulary
fact V) and every binding is permanent (L12) — GlobalUniqueness (ASN-0034) at
the link layer. The corollary is a constraint on every reference that intends
to survive editing — and on the claim schema in particular: *bind addresses,
never positions.* A position-bound "supersedes entry 2.2" would, across one
contraction and one extension, silently re-target a stranger. Df-DISC already
complies; the construction shows the compliance is load-bearing, not stylistic.

> *Implementation note.* This is Q12 verbatim: `findnextlinkvsa` recomputes
> the next link V-position from the live document extent, which DELETEVSPAN
> shrinks, while I-addresses advance monotonically against the append-only
> granfilade — so V-position `2.N` can denote two different link orgls across
> time, and only the ISA is identity.

**EL11 (TwoRegimeDiscovery).** A claim is findable in two ways with two
different conditions — and the split is exactly the design's distinction
between the *record answering* and the *context volunteering*.

*(a) Contextual (arrangement-gated).* For a schema-conforming claim `e` and any
document `d`, the to-side of `e` projects into `d` iff `d` currently lists the
original:

> `project(Σ.L(addr(e)).e₂, d, Σ) ≠ ∅ ⟺ listed(old(e), d, Σ)`

*Proof.* By the projection characterisation (LP12, ASN-0098) the left side is
`coverage(G) ∩ ran(Σ.M(d)) ≠ ∅` with `G` the to-set. `coverage(G) = {t :
old(e) ≼ t}` (EL4's computation). Any member of `ran(Σ.M(d))` lies in
`dom(Σ.C) ∪ dom(Σ.L)` (S3★). No content address extends `old(e)`: writing `y =
old(e)`, a `t ≽ y` agrees with `y` on its three zero positions, and a content
address has exactly three zeros (C1) — so `t`'s element field starts where
`y`'s does and `E(t)₁ = E(y)₁ = s_L` (L0, link side), contradicting
`E(t)₁ = s_C` (L0, content side; SC-NEQ). And a link address extends `y` only
if equal (R0a). So the intersection is `{y} ∩ ran(Σ.M(d))`, nonempty iff `y`
is listed — and by Df-LISTED only at `d = home(y)`. ∎ Symmetrically for the
from-side and `new(e)`. So the claim is visible *in context* exactly at the
homes currently listing its endpoints: de-list the original and the claim's
to-side goes contextually dark there; re-list it and visibility returns — the
orphan/resurrection pattern of LP17/LP18 at the link layer.

*(b) Archival (arrangement-independent).* The predicates `e ∈ Ŝ^Σ` (slice
membership *and* schema-conformance) and `old(e) = y` are functions of stored
values, decidable by coverage comparison (CoverageEqualityDecidable; T2 span
membership; EL4). Hence the claim sets

> `in(y, Σ) = {e ∈ Ŝ^Σ : old(e) = y}`  and  `out(x, Σ) = {e ∈ Ŝ^Σ : new(e) = x}`

are computable directly from `Σ.L` alone — by the comprehensions just written —
at every state and for any `y, x`, consulting no arrangement. For
`y, x ∈ dom(Σ.L)` they coincide with `Observe_{K_sup}` at pattern `Ĝ = {y}`
(resp. `F̂ = {x}`), view `hist`, filtered by the decidable schema-conformance
predicate (a no-op at disciplined states, where every `[K_sup]` tuple already
conforms): on conforming claims `Observe` returns those with `old(e) ≼ y` (resp.
`new(e) ≼ x`), and the antichain R0a collapses `≼` to `=` precisely because `y`
(resp. `x`) lies in `dom(Σ.L)`. It is the `Observe` *identification* that
carries this qualification; the direct comprehension does not.
**The supersession question is answerable, completely and decidably, at every
state, whatever every arrangement says.** A reader's protocol — before relying
on a link, ask the record what targets it — is always executable; what
arrangement state modulates is only whether a given document's current view
volunteers the pointer unprompted.

The reader-facing contract of the architecture is the conjunction of EL9(1),
EL11(b), and EL8(b): the old link gives no *forced* sign of replacement — it is
unchanged, that is the point — but the sign *exists*, is always findable, and
arrives with its asserter's name attached.

## Repeated edits, many editors, and the meaning of "current"

**EL12 (ForkPermanence).** Two editors independently superseding the same link
produce a permanent, co-visible fork. Run `editlink(a, ·, ·, ·)` twice from any
disciplined reachable state, in any combination of homes: freshness yields
distinct successors `a'₁ ≠ a'₂` and distinct claims `e₁ ≠ e₂` (same home: the
chain advances past the first emission; different homes: cross-document
disjointness with T10); both `(a, a'₁)` and `(a, a'₂)` enter `succ_h` —
permanently (EL5a) — and `succ_o` at birth (EL6(iii), the second invocation's
active-at-birth conclusion resting on EL7(vi): the first `editlink` leaves the
intermediate state edit-disciplined, which is the hypothesis EL6(iii) needs);
and the vocabulary contains no transition that merges, ranks, or removes either. The complete
competing-claim set, with asserters, is one archival query: `in(a, Σ)`.
Conversely — and this is Q18 made abstract — *without* the assertion steps the
same two emissions leave `succ_h` untouched: by EL1 the "fork" of intentions
never existed in state, and no enfilade, here or in any implementation of this
substrate, could have recorded it. Fork *visibility* is exactly
assertion-deep.

**EL13 (TemporalErasure).** Cross-home claim order is not a fact of the state.
For `d₁ ≠ d₂ ∈ dom(Σ.M)` and values `v₁, v₂`, the two interleavings of the
emissions commute to the same state:

> `K.λ(d₂, a_emit(·, d₂), v₂) ∘ K.λ(d₁, a_emit(·, d₁), v₁) (Σ) = K.λ(d₁, a_emit(·, d₁), v₁) ∘ K.λ(d₂, a_emit(·, d₂), v₂) (Σ)`

*Proof.* `a_emit(Σ', d)` depends only on the `d`-homed subset of `dom(Σ'.L)`
(ASN-0086, EmitAddress, with HomeOriginCoincidence); an emission homed at
`d₁` leaves the `d₂`-homed subset unchanged, so each address is the same in
both orders; the enabledness of each step consults only its own home's set and
`dom(M)`; and the two map-unions at distinct fresh keys commute, all other
components being framed. ∎ Consequently no function of the final state — no
selector, no tie-break, no "latest" — distinguishes which of two cross-home
claims was asserted later. Within one home the opposite holds: the chain
enumeration is strictly increasing (T9;
ChainEnumerationInjectivity, ASN-0093), so the claims homed at one *document*
are totally ordered by their addresses — *per-home* "latest" is well-defined and
state-recoverable — per-document-chain, not per-principal. (A per-asserter
"latest" is not a state function — it needs principal resolution, placed outside
`Σ` by EL8(b).)

**Df-CUR (Currency query).** For `y ∈ dom(Σ.L)`: `reach_o(y, Σ)` is the least
subset of `dom(Σ.L)` containing `y` and closed under `succ_o(Σ)`-images —
finite and computable (the closure grows within finite `dom(Σ.L)`; bound
function `|dom(Σ.L)| − |computed set|`). The *current successors* of `y` are
the operative sinks reachable from it:

> `current(y, Σ) = {z ∈ reach_o(y, Σ) : ¬(E w :: (z, w) ∈ succ_o(Σ))}`

The sink test `¬(E w :: (z, w) ∈ succ_o(Σ))` reads only the *operative claims*
out of `z`.

**EL14 (CurrencyRelational).** `current` is a total, computable,
*set-valued* query, and the set is irreducibly a set:

*(a)* `|current(y, Σ)| = 1` at states with one asserted, unretracted, linear
chain from `y` — and `current(y, Σ) = {y}` when `y` has no operative
successor: an unedited link is its own current version.

*(b)* `|current(y, Σ)| ≥ 2` at any fork state (EL12).

*(c)* `current(y, Σ) = ∅` is reachable: assert `x` supersedes `y`, then —
a revert, by anyone — `y` supersedes `x`. Both claims are permanent; while
both are operative, `reach_o(y) = {y, x}` has no sink. The operative record
then says "each replaces the other," and the honest answer to "which is
current?" is *none, pending judgment*. The repair is not deletion (there is
none) but demotion: `Nullify` one claim, and a sink — hence a current —
reappears. The two-view structure is what makes the standoff survivable: the
operative graph is repairable precisely because the historical graph is not.

*(d)* No state-definable selector canonically identifies the latest edit. A
selector that picked out *the* current successor as the latest edit would have
to read assertion order from the state — which EL13 denies across homes — so no
*temporal*, recency-respecting selector is a function of `Σ`. A non-temporal tie-break remains state-definable — an arbitrary order such
as T1-least claim address *is* a function of `Σ` — but it ranks namespaces, not
times, returning a representative without identifying *the latest* edit:
definable, yet not canonical. Nor can structure force the issue: making
`|current| = 1` an invariant would require refusing well-formed emissions or
erasing claims, and the substrate does neither. What the layer owes
the reader is therefore *disclosure, not decision*: `current(y, Σ)` entire,
each member with its supporting claims and their homes (EL8b) *and its own
activity status* (`active(·)`, EL9(3) — an axis membership does not settle,
EL14(e)), the original always still readable beside them (EL9(1)), and any
narrowing — trust only the original owner's claims, prefer this curator,
follow per-home latest, drop members that are themselves retracted — applied
as the reader's declared policy, not the substrate's silent one.

*(e) Activity-agnostic membership.* `current(y, Σ)` is built from `succ_o(Σ)`,
whose only activity filter is the *claim*-address test `addr(e) ∉ nullified(Σ)`
(Df-SUCC) — never a test on the endpoint links `old(e)`, `new(e)`. A successor
is a link like any other, so its activity (EL9(3)) is an independent axis, and
`Nullify` (ASN-0086) may target any `z ∈ dom(Σ.L)`, depositing a unit-depth
`[R]`-tuple that preserves edit-discipline (EL-DM step) and puts
`z ∈ nullified(Σ)` while adding no *claim* address to `nullified` (its
to-coverage meets `dom(Σ.L)` in `{z}` alone, R0a). The standoff is reachable
inside the disciplined layer: from a disciplined `Σ`, `editlink(a, ·, d_s, d_a)`
gives successor `a'`, claim `b`, with `(a, a') ∈ succ_o` and `current(a) = {a'}`
(EL7(iii) at the disciplined post-state); a following `Nullify(·, d, a')` puts
`a' ∈ nullified` yet leaves `b ∉ nullified`, so `(a, a')` *stays* in `succ_o`
and `current(a)` remains `{a'}` — its sole member now retracted. Hence

> `z ∈ current(y, Σ)` does not imply `active(z, Σ)`:

the supersession sink and the sink's own activity are independent — EL9's
three-axis independence read at the successor, which is resolvable (EL9(1)) and
a current sink yet may be inactive (EL9(3)). `current` answers *which readings
are supersession-sinks under the operative claims*, not *which readings are
themselves operative*; a reader needing the latter layers the separate query
`active(·)` over each member, exactly the status (d) now obliges the layer to
disclose. (EL15(d) records the dual gap on the other factor — a nullified
*claim* leaving `succ_o`; this clause records the nullified *endpoint*, which
`succ_o` does not inspect at all.)

**EL15 (ChainConnectivity).** For a chain of asserted edits
`a₀, a₁, …, aₙ` with each `(aᵢ, aᵢ₊₁) ∈ succ_h(Σ)`:

*(a)* every member is permanently resolvable at its own address with its
original value (EL0) — the far end of history is never lost;
*(b)* every asserted hop is permanently in `succ_h` (EL5a), so *historical
connectivity is monotone*: the `succ_h`-component of any member never loses a
node or an edge, at any future state;
*(c)* every hop is locally recoverable from either endpoint alone — `in(aᵢ, Σ)`
and `out(aᵢ, Σ)` are single arrangement-free observations (EL11b) — so the
historical component is traversable edge-by-edge in both directions from any
member, with no privileged entry point;
*(d)* what is *not* guaranteed is completeness and operative integrity:
an edit whose author omitted the assertion contributes no hop (EL1 — the
relationship does not exist, and resemblance cannot reconstruct it), and a
nullified claim drops from `succ_o` while remaining in `succ_h`. Member-to-ends
traversability of the *operative* chain is therefore a derived property —
holding exactly when the chain was fully asserted and no hop demoted. The
invariants to hold are (a)–(c); the design intent (Q7) asks for exactly these
and warns against more.

## The criterion: permanence-preserving versus mutating edit

We can now state the test that separates an editing discipline that preserves
permanence from one that merely simulates editing — and verify that the
composite passes it while both failure modes fail it. The test (Q10) is
*reference survival*: point a reference at the original, perform the edit,
follow the reference.

**EL16 (ReferenceSurvival).** Let `c ∈ dom(Σ.L)` be any link with
`a ∈ coverage(Σ.L(c).eᵢ)` for some slot `i` — a pre-existing reference to the
original, made by anyone, anywhere. Across `editlink(a, ℓ', d_s, d_a)` and
arbitrary further evolution `Σ →* Σ'`:

*(i)* the referring slot is unchanged in value and coverage (L12; LP2, LP3★) —
nobody's context is rewritten by someone else's edit;
*(ii)* the referent still resolves, to the identical value:
`Σ'.L(a) = Σ.L(a)` (EL0) — the reference means today what it meant when made;
*(iii)* the road forward exists and is one observation long:
`in(a, Σ') ∋ e` with `new(e) = a'`, attributed to `home(addr(e))` — the
reference reaches the successor not by being re-pointed but by *composition
with the record*.

Against this, the two rejected regimes fail at named clauses. **Mutation**
(excluded by EL0) would preserve the reference's *spelling* while silently
re-pointing its *meaning* — every citation, comment, and dispute attached to
`a` would come to qualify content its authors never saw: the global rewrite the
immutability invariant exists to make impossible.
**Silent re-creation** — step 1 without step 2, the delete-and-recreate
simulation — passes (i) and (ii) vacuously and fails (iii): the successor
exists, fresh and disconnected, indistinguishable from a stranger (EL1) — the
"frozen and dead" copy; the old references keep their exact referent and gain
no road anywhere. The asserted edit is the unique point between the two.

## A worked example

Two documents, `H` and `P`, both in `dom(Σ.M)`, owned by different principals.
`H` homes one link, `ℓ₀ = H.0.s_L.1`, listed in its registry at
`V_{s_L}(H) = {[s_L, 1]}` with `[s_L, 1] ↦ ℓ₀`. `P` homes nothing. The state
is edit-disciplined (EL-DM — reached from `Σ₀` by document registrations and
original-link creations alone, no claim or retraction yet); `S^Σ = ∅`.

*Edit by the owner.* `editlink(ℓ₀, ℓ'-value, H, H)`: the `H`-homed set is
`{ℓ₀}`, so `a_emit` takes the subsequent-emission branch — successor
`ℓ₁ = inc(ℓ₀, 0) = H.0.s_L.2` — and the claim follows on the same chain,
`c₁ = H.0.s_L.3` with value `({(ℓ₁, δ(1,#ℓ₁))}, {(ℓ₀, δ(1,#ℓ₀))}, K_sup)`.
Now `succ_h = succ_o = {(ℓ₀, ℓ₁)}` and `current(ℓ₀) = {ℓ₁}`. Note the
addresses already archive the per-home order: `ℓ₁ < c₁` on `H`'s chain — `H`'s
acts are totally ordered by T9 — while nothing yet involves a second home.
The registry is untouched: `ℓ₀` is still `H`'s one listed link, still active;
the edit changed nothing about it (EL7(iv)).

*Fork by a third party.* `P`'s owner disagrees with the correction and issues
their own: `editlink(ℓ₀, ℓ''-value, P, P)` — first emissions on `P`'s chain,
successor `ℓ₂ = P.0.s_L.1`, claim `c₂ = P.0.s_L.2`. Now
`succ_o = {(ℓ₀, ℓ₁), (ℓ₀, ℓ₂)}` and `current(ℓ₀) = {ℓ₁, ℓ₂}` — a fork, both
branches operative, the query `in(ℓ₀, ·) = {c₁, c₂}` returning both claims
with their provenance legible in their addresses: `home(c₁) = H` (the
original's own home), `home(c₂) = P` (an outsider). Had `c₁` and `c₂` been
asserted in the other order, the state would be *identical* (EL13): which
claimant moved first is not a question the docuverse can answer, and any
front end that pretends otherwise is inventing history.

*Demotion.* `H`'s owner finds `P`'s claim spurious and retracts it from
operative standing: `Nullify(Σ, H, c₂)` emits `r₁ = H.0.s_L.4` of class `[R]`
targeting `c₂`. Now `c₂ ∈ nullified`, so `succ_o = {(ℓ₀, ℓ₁)}` and
`current(ℓ₀) = {ℓ₁}` — while the historical view still returns `c₂` *and*
`r₁`: a reader sees that `P` claimed, and that `H` retracted `P`'s claim, each
act owned. Nothing was erased; standing changed, attributably.

*Revert, standoff, repair.* Later `H`'s owner repents of the original edit and
reverts — no allocation of content or successor, just the statement:
`assert_sup(ℓ₀, ℓ₁, H)` emits `c₃ = H.0.s_L.5` claiming `ℓ₀` supersedes `ℓ₁`.
While `c₁` and `c₃` are both operative, `reach_o(ℓ₀) = {ℓ₀, ℓ₁}` has no sink:
`current(ℓ₀) = ∅` — the record says each replaces the other, and the honest
answer is *pending*. `Nullify(Σ, H, c₁)` (emitting `r₂ = H.0.s_L.6`) demotes
the first claim; now `succ_o = {(ℓ₁, ℓ₀)}`, and `current(ℓ₀) = current(ℓ₁) =
{ℓ₀}` — the original is current again, by statement, not by time travel. Five
permanent entries — `c₁, c₂, c₃, r₁, r₂` — narrate the whole episode to any
future reader.

*Registry churn, and why claims bind addresses.* Suppose `H`'s owner re-curates
the registry: `K.μ⁻` with `n'_{s_L} = 0` de-lists `ℓ₀`, then `K.μ⁺_L` seats
`ℓ₁` — the first-position branch assigns `[s_L, 1] ↦ ℓ₁`. The position
`[s_L, 1]` has now denoted `ℓ₀` and later `ℓ₁` (EL10). Every claim above is
unperturbed — each binds `H.0.s_L.k` addresses, which never re-bind — whereas
a position-bound claim "supersedes `H`'s entry `[s_L,1]`" would now
accidentally indict `ℓ₁`. Contextually, `c₁`'s to-side no longer projects into
`H` (`ℓ₀` unlisted, EL11a), so `H`'s current view no longer volunteers the
old dispute; archivally, `in(ℓ₀, ·)` still returns everything (EL11b).

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| EL0 | MutationExclusion: for reachable `Σ₀`, `a ∈ dom(Σ₀.L)`, `ℓ₀ = Σ₀.L(a)`, the persistence invariant `(A Σ' : Σ₀ →* Σ' : Σ'.L(a) = ℓ₀)` is L12 closed under `→*` — LP13 (ASN-0098) at `a`; read as a weakest precondition it gives `wp(S, L(a) = w) = false` at `Σ₀` for every `w ≠ ℓ₀` and every finite program `S` — link mutation is unimplementable, and the original remains readable at its address with its exact value forever | introduced |
| EL1 | IntentInvisibility: a successor emission performed "as an edit" and an independent creation with the same parameters are the same transition with the same post-state, so no state predicate (hence no observation) distinguishes them; value resemblance, up to byte-identity (L11b), carries zero relational information; relationships enter the state only by explicit assertion | introduced |
| EL2 | NoInPlaceCarrier: the supersession record can live (a) not in the original's value (L12), (b) not appended to the successor's value after birth (L12), (c) not in the address relation — allocated link addresses have `#E = 2` on flat home chains and `dom(Σ.L)` is a prefix antichain (R0a), so address structure encodes only same-home and per-home emission order, neither semantic, and version-of-link nesting is unreachable — and (d) not in any index marker, the state having no component beyond the append-only stores; the record must be a freshly allocated entity | introduced |
| RQ1–RQ7 | RecordRequirements: post-hoc assertability; open authorship; decidable attribution; non-destructive disputability; endpoint frame; decidable specificity with subtype refinement; plurality of (possibly conflicting) claims — distilled from design intent as the obligations any supersession carrier must meet | introduced |
| EL3 | RelationSpaceNecessity: any carrier satisfying RQ1–RQ7 under this substrate is a freshly allocated link-store tuple, distinct from both endpoints, referencing each endpoint by address through endset coverage, kind-marked by the coverage class of its type slot — i.e., a typed link-to-link claim; content-encoded, slot-at-birth, and address-form carriers each violate named RQs; "separate supersession link" and "typed relation" are one architecture (L8) | introduced |
| Df-CLS | SupersessionClass: designated coverage class `[K_sup]` with `coverage(K_sup) ≠ coverage(R)`; historical slice `S^Σ = L_{K_sup}^Σ` (claims), operative subset `A_sup^Σ = {(b,F,G) ∈ S^Σ : b ∉ nullified(Σ)}` | introduced |
| Df-DIR | ClaimDirectionality: from-set covers the superseding link, to-set the superseded ("F replaces G"), aligned with RetractionDirectionality; replacement-free withdrawal is class `[R]`, a distinct relation | introduced |
| Df-DISC | EditDiscipline: a state is edit-disciplined iff unit-depth-retraction-disciplined and every claim has form `F = {(x, δ(1,#x))}`, `G = {(y, δ(1,#y))}` with `x, y ∈ dom(Σ.L)`, `x ≠ y`; a layer is edit-disciplined iff every reached state is | introduced |
| Df-LAY | EditingLayer: the editing layer issues `{assert_sup, editlink, Nullify}` plus the link-framing substrate transitions (`K.α`, `K.δ`, `K.μ⁺`, `K.μ⁺_L`, `K.μ⁻`, `K.ρ`, `K.μ~`) and the bare `K.λ`, confined to original-link creation; its discipline commitment routes every `[K_sup]` emission through `assert_sup`/`editlink` (under `DC`) and every `[R]` emission through `Nullify` — discipline is a protocol property, not substrate-enforced; editing-layer-reachable = reached from `Σ₀` by such operations | introduced |
| EL-DM | DisciplineMaintenance: every editing-layer-reachable state is edit-disciplined. Base — `Σ₀` (`L₀ = ∅`) is vacuously disciplined (`S^{Σ₀} = L_R^{Σ₀} = ∅`). Step — L-framing transitions and original-creating bare `K.λ` leave `S^Σ`/`L_R^Σ` undisturbed (Vocabulary fact V, L12, monotone `dom(L)`); `Nullify` adds only a unit-depth `[R]` tuple (no `[K_sup]` claim); `assert_sup` preserves by EL6(v); `editlink` by EL7(vi). Gives the "at disciplined `Σ`" conditionals below a reachable, non-vacuous domain | introduced |
| EL4 | SingleTarget: for any *schema-conforming* claim (per-claim, no whole-state hypothesis) `coverage(F) ∩ dom(Σ.L) = {x}` and `coverage(G) ∩ dom(Σ.L) = {y}` (PrefixSpanCoverage + R0a), making `addr(e)`, `new(e)`, `old(e)` total on the schema-conforming subset `Ŝ^Σ` at every reachable state (`Ŝ^Σ = S^Σ` at disciplined states) | introduced |
| Df-SUCC | Successor relations over the schema-conforming claims `Ŝ^Σ` (EL4): `succ_h(Σ) = {(old(e), new(e)) : e ∈ Ŝ^Σ}`; `succ_o(Σ)` the further restriction to `addr(e) ∉ nullified(Σ)`; finite, `succ_o ⊆ succ_h`; total at every reachable state (the `Ŝ^Σ` restriction excludes non-conforming `[K_sup]` tuples on which `old`/`new` are undefined), coinciding with the unrestricted form at disciplined states (`Ŝ^Σ = S^Σ`) | introduced |
| EL5 | RecordMonotonicity: across `Σ →* Σ'`, (a) `S^Σ ⊆ S^{Σ'}`, `Ŝ^Σ ⊆ Ŝ^{Σ'}` (schema-conformance is value-and-domain-determined), and `succ_h(Σ) ⊆ succ_h(Σ')`; (b) `nullified(Σ) ⊆ nullified(Σ')` — demotion is one-way per claim; (c) `succ_o` is neither monotone nor antitone: the operative relation is the revisable view, the historical relation the unrevisable record | introduced |
| ASSERTop | AssertSup (DEF, operation): `assert_sup(x, y, d_a) ≜ Emit_{K_sup}(Σ, d_a, {(x, δ(1,#x))}, {(y, δ(1,#y))})`, precondition `x, y ∈ dom(Σ.L) ∧ x ≠ y ∧ d_a ∈ dom(Σ.M)` — one `K.λ` emitting the claim "x supersedes y" at fresh `b = a_emit(Σ, d_a)` | introduced |
| EL6 | AssertionContract: assert_sup allocates exactly one fresh address `b` with `home(b) = d_a`; puts `(y, x)` into `succ_h(Σ')`, and at disciplined states into `succ_o(Σ')` (active at birth, via ASN-0086 wp Case 2); frames `C, M, E, R` and every prior link entry; deactivates nothing — `nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)` unconditionally (no `[R]` growth), and the full `nullified(Σ') = nullified(Σ)` under edit-discipline (fresh `b` escapes pre-existing unit-depth retraction coverage by R0a, wp Case 2); preserves discipline; the claim and the pair persist at every later state | introduced |
| EDITop | Editlink (DEF, operation): `editlink(a, ℓ', d_s, d_a) ≜ K.λ(d_s, a_emit(Σ, d_s), ℓ') ; assert_sup(a', a, d_a)`, precondition `a ∈ dom(Σ.L) ∧ d_s, d_a ∈ dom(Σ.M) ∧ ℓ' L3-conforming ∧ DC(ℓ')` (successor not of retraction class; an arity-3 `[K_sup]` successor — a would-be claim — conforms to the claim schema, the guard `|ℓ'| = 3` firing the schema clause exactly on the slice ASN-0086 admits); returns `(Σ₂, a', b)`; `ℓ' = Σ.L(a)` admitted; homes unconstrained relative to `home(a)` (third-party edit-by-fork is the same composite); a revert is `assert_sup(a, a', d)` alone | introduced |
| EL7 | EditContract: editlink allocates exactly two fresh link-subspace addresses (successor `a'`, claim `b`) and nothing else; `Σ₂.L(a') = ℓ'`; `(a, a') ∈ succ_h(Σ₂)` (and `succ_o` at disciplined states); frame `Σ₂.C = Σ.C ∧ Σ₂.M = Σ.M ∧ Σ₂.E = Σ.E ∧ Σ₂.R = Σ.R ∧ (A t ∈ dom(Σ.L) : Σ₂.L(t) = Σ.L(t))` unconditionally, with `nullified(Σ₂) = nullified(Σ)` under edit-discipline on `Σ` (`DC` bars a retraction-class successor, and both fresh addresses escape pre-existing retraction coverage by R0a); preserves edit-discipline (vi): `Σ₂` disciplined when `Σ` is, via `DC(ℓ')` for step 1 and EL6(v) for step 2; all three addresses and the pair persist forever | introduced |
| EL8 | ClaimStanding: every claim is permanent (EL5a); attributed by its address alone (`home(addr(e))` via T4b, decidable T6 — the allocating document, not a named owner); open (no required relation among claim, original, successor homes); itself addressable — endorsable, disputable, retractable, editable — with no new machinery | introduced |
| EL9 | ThreeAxes: for any link — (1) resolution is permanent and ungated (EL0); (2) listing (`listed(t, d, Σ)`, possible only at the home, CL-OWN) is mutable both ways via `K.μ⁻`/`K.μ⁺_L`; (3) activity (`∉ nullified`) is monotone downward with re-assertion as the only restoration; the axes are independent, and superseding moves none of them — retirement of the original is a separate, attributable act | introduced |
| EL10 | PositionEpochality: reachable states exist where the same link-subspace V-position denotes `ℓ₁` and later `ℓ₂ ≠ ℓ₁` (contraction then extension reuses the canonical tail position), while addresses never re-bind; therefore surviving references — the claim schema included — must bind addresses, never positions | introduced |
| EL11 | TwoRegimeDiscovery: (a) contextual — a disciplined claim's to-side projects into `d` iff `d` currently lists the original (`project ≠ ∅ ⟺ listed(old(e), d, Σ)`, by LP12 + coverage trace `{old(e)}`), symmetrically for the from-side; (b) archival — `in(y, Σ)` and `out(x, Σ)` (over the schema-conforming `Ŝ^Σ`) are computable from `Σ.L` alone, completely and decidably, at every state; the record always answers, the context volunteers only while its registry lists the endpoint | introduced |
| EL12 | ForkPermanence: independent edits of the same original yield distinct successors and claims, all permanent, co-operative at birth, never merged, ranked, or removed; the full competing set with asserters is one archival query; absent assertions the fork never exists in state (EL1) — fork visibility is exactly assertion-deep | introduced |
| EL13 | TemporalErasure: cross-home emissions commute to identical states, so no state function recovers cross-home claim order ("global latest" is undefinable); within one home, claim order is totally recoverable from addresses (T9) — "latest" is per-home (per-document-chain) only, not per-principal (EL8b) | introduced |
| Df-CUR | Currency query: `reach_o(y, Σ)` the `succ_o`-closure of `{y}`; `current(y, Σ)` its operative sinks — total, finite, computable; the sink test reads only the *operative claims* out of `z` (a claim-activity filter), not `z`'s own activity (EL14e) | introduced |
| EL14 | CurrencyRelational: `current` is irreducibly set-valued — cardinalities 1 (linear chain; `{y}` itself when unedited), ≥ 2 (fork), and 0 (mutual-supersession standoff) are all reachable; demotion repairs the operative view while history stands; no state-definable selector canonically identifies the latest edit — a temporal/recency selector is not a state function across homes (EL13), while the definable global tie-break (T1-least) ranks namespaces not times; structural uniqueness would require refusing emissions or erasing claims; (e) membership is *activity-agnostic on members* — `succ_o` filters on *claim* activity (`addr(e) ∉ nullified`) only, so `z ∈ current(y)` need not satisfy `active(z)` (a successor may be `Nullify`'d as an endpoint while its claim stands; sink and member-activity are independent axes, EL9(3)); the layer owes disclosure with attribution *and each member's activity status*, the reader applies policy | introduced |
| EL15 | ChainConnectivity: along asserted chains, members resolve forever, hops persist in `succ_h`, and each hop is recoverable from either endpoint alone — historical connectivity is monotone non-decreasing; completeness (unasserted hops) and operative integrity (demoted hops) are expressly not invariants, and member-to-ends operative traversability is a derived property, not a guarantee | introduced |
| EL16 | ReferenceSurvival: any pre-existing reference to the original keeps its value, coverage, and referent across the edit and all evolution, and reaches the successor by one archival observation; mutation would silently re-point every reference (excluded by EL0), silent re-creation leaves references intact but the road forward empty (EL1) — the asserted edit is the unique regime preserving both the exact past and the reachable future | introduced |

## Open Questions

What authority invariant must govern retraction of a supersession claim by a principal other than the claim's asserter?

Must replacing a link ever entail a change to the original's activity status, or must supersession and retraction remain independent axes under every admissible layer discipline?

What must the supersession of a retraction tuple guarantee, given that R6a fixes the retraction's nullifying effect permanently so that no successor can move an existing nullification?

What invariants must hold when supersession claims target other supersession claims, and must currency resolution stratify such meta-claims to remain well-founded?

Under what assertion discipline is the operative currency query guaranteed a non-empty answer, and is any such discipline compatible with open authorship?

What temporal witness, if any, must the system attach to claims beyond per-home emission order, given that cross-home order is not recoverable from state?

When an edit narrows or reshapes an endset, must the record carry span-level correspondence between the old and new endsets, and in what space does that correspondence live?

What coupling invariant, if any, should bind an edit to the home registry's listing of original and successor, given that assertion itself cannot be substrate-enforced?

Given that supersession subtypes minted under a common prefix stay jointly queryable by one rooted span (L10) only once that shared root is agreed, what closure guarantee must observation provide across a prefix-rooted family of subtypes so that independently minted refinements remain jointly recognizable?
