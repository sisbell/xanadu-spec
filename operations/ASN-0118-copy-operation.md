> **ASN-0118 · The COPY Operation — Transclusion as Shared Reference** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0042 · Tumbler Ownership](../foundation/ASN-0042-tumbler-ownership.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0084 · Cut-Point Rearrangements](../foundation/ASN-0084-bundle-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0118-copy-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0118: The COPY Operation — Transclusion as Shared Reference

*2026-06-11*

## The problem

Nelson draws one line through the whole architecture and asks us never to cross
it: "inclusion by reference, not copy." When content already living in the
docuverse is placed into a new document, the system must reuse the *identity* of
that content rather than manufacture a second copy of it. The operation that
performs this placement is COPY, and Nelson calls its effect *transclusion*.

We are asked to be exact about what that means. The system is handed a *spec-set*
— an ordered series of spans naming content in one or more existing documents —
together with a destination document and a V-position within it. COPY places the
named material at that position. The question is a question about *boundaries*:
what is reused from the existing content store and what is recorded fresh in the
destination; what relationship the placed material must bear to its source's
identity, to the destination's prior arrangement, and to any other document that
already shares those addresses; what assembling content from several
non-contiguous sources discloses about shared identity, ownership independence,
and the line between reuse and replication; and what invariants COPY must
preserve about content immutability, the permanence of the source's identity
inside the destination, the survival of links anchored to the reused content, and
the isolation of the source from the act of being copied from.

We shall find that "place content by reference" decomposes into a *resolution*
step (read the spec-set through its source arrangements to recover a sequence of
content addresses) and a *placement* step (bind those very addresses to fresh
V-positions in the destination), and that the entire content of the operation —
the thing that makes it transclusion rather than copying — is one frame
condition: **the content store does not grow.** Every other property follows from
that, from the permanence of the address space, and from the fact that the
destination owns its arrangement but not the content it arranges.

## The substrate we build on

**Standing precondition (reachability).** Throughout, every state `Σ` ranges over
states reachable from the initial state `Σ₀` under the sequential transition order
(ASN-0047, SequentialTransitionAxiom). This licenses the per-state invariant
citations below, which ASN-0047 collects in `ExtendedReachableStateInvariants` and
guarantees only of reachable states.

**Standing precondition (composite boundary).** COPY is itself an ASN-0047
composite, so it is invoked *at a composite boundary*: its pre-state `Σ` is the
final state of a completed composite, never a state reached mid-composite. This licenses the *composite-boundary properties* that
ASN-0047 collects separately from the per-state invariants — in particular P4★
(`Contains_C(Σ) ⊆ R`), which holds at composite boundaries but may fail at an
intermediate atomic state.

We take the strand model as given. The *content store* `Σ.C : T ⇀ Val`
(ASN-0036) binds content addresses — *I-addresses* — to values. It is append-only
and immutable: once `a ∈ dom(Σ.C)`, `a` persists and `Σ.C(a)` never changes
(ASN-0036, S0 ContentImmutability; S1 StoreMonotonicity). The *arrangement* of a
document `d` is a partial function `Σ.M(d) : T ⇀ T` (ASN-0036) from V-positions to
I-addresses; it is a genuine function (S2 ArrangementFunctionality) whose every
image lies in the store appropriate to the V-position's subspace — content
positions into `dom(Σ.C)`, link positions into `dom(Σ.L)` (ASN-0047, S3★
GeneralizedReferentialIntegrity, refining ASN-0036's S3) — and it is the one
component of state that may be edited in place. The *link store* `Σ.L : T ⇀ Link`
(ASN-0043, ASN-0093) is permanent (L12 LinkImmutability). The *entity set*
`Σ.E ⊆ T` (ASN-0047) holds the allocated organisational addresses — nodes,
accounts, documents — its document stratum coinciding with the arrangement
domain, `E_doc = dom(Σ.M)` (ASN-0047, M1). A document records,
besides its arrangement, a *provenance* relation `Σ.R ⊆ T × T` (ASN-0047): a pair
`(a, d)` records that document `d` has referenced I-address `a`.

We write `V_S(d) = {v ∈ dom(Σ.M(d)) : subspace(v) = S}` for the V-positions of
document `d` in subspace `S`, where `subspace(v) = v₁` (ASN-0036). The text
subspace is `s_C` and the link subspace `s_L` (ASN-0047, SubspaceConventionAxiom);
this ASN concerns transclusion of *content*, so its placements land in `s_C`. We
use the ordinal-shift abbreviation `t + k ≡ shift(t, k)` for a tumbler `t` and
natural `k`, with `t + 0 = t` (ASN-0034, OrdinalShift; ASN-0058,
OrdinalShiftBase). For an I-address `a ∈ dom(Σ.C)`, `origin(a) = N(a).0.U(a).0.D(a)`
is the document-level prefix recovered from the address by field projection
(ASN-0036, S7); it is the tumbler of the document that allocated `a`, and it is
invariant across every state in which `a` is stored (S7(d)).

## What a spec-set names, and what resolution recovers

A *V-spec* is a pair `ρ = (d_s, σ)`: an allocated source document
`d_s ∈ dom(Σ.M)` together with a span `σ = (s, ℓ)` admissible under two
conditions, which we state natively. It is *well-formed* in the sense of T12
(ASN-0034): `Pos(ℓ)` and `actionPoint(ℓ) ≤ #s`, so its denotation
`⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` is a well-defined order-convex set of tumblers.
And it draws from a non-empty source subspace:
`V_{subspace(s)}(d_s) ≠ ∅`. The *active positions* of a V-spec are those
tumblers the span denotes that the source arrangement actually binds,
`act(ρ, Σ) = dom(Σ.M(d_s)) ∩ ⟦σ⟧` — the intersection of the source document's
bound V-positions (ASN-0036) with the span's denotation, and exactly the domain
of ASN-0058's restriction `M(d_s)|⟦σ⟧` on which its `resolve` is defined. The
set is finite (subset of the finite `dom(Σ.M(d_s))`, S8-fin) and totally
ordered (subset of the totally ordered carrier `T`, T1), hence has a unique
ascending enumeration `v₁ < … < v_k`. The two admissibility
conditions are ASN-0058's ContentReference conditions (ii) and (i) inherited
verbatim; its condition (iii) — which pins the width's tumbler length to the
start's and both to the source subspace's common V-position depth,
`#ℓ = #u = m` (S8-depth) — we drop entirely, so every ContentReference is a
V-spec, but a V-spec's width may carry any length T12 admits, equal to the
start's or not.

The drop costs nothing downstream, because no consequence this ASN consumes
rests on either half of condition (iii) — not the depth pin `= m`, and not the
shape pin `#ℓ = #s` (level-uniformity, ASN-0053 S6): no clause of CP0–CP12
mentions `#s` or `#ℓ`, and the two facts the arithmetic does need — the bound
active set is single-subspace by content-residence
(`act(ρ, Σ) ⊆ V_{s_C}(d_s)`, the operation's precondition below) and
single-depth by S8-depth (ASN-0036) applied to the active positions themselves
— hold *regardless* of `#s`, of `#ℓ`, and of where `ℓ`'s action point falls.
(A content span may, but need not, be *ordinal-level* — action point at the
deepest component, `actionPoint(ℓ) = #ℓ`, so the span advances along the last
component alone.) Nor is level-uniformity a fact about spans that the design
merely happens not to use here. Nelson names the same-level span as the
special case of the concept — a span refers to a subtree, "or merely to a
series of elements of the same type, which is a degenerate case of a subtree"
(4/24) — and his difference arithmetic gives the width a shape of its own: its
leading zeros mark the level at which the span diverges from its start, and
everything after its first nonzero field is copied from the *end* tumbler
(4/32–4/33), the start's trailing digits being discarded by addition. The
width's shape mirrors the span's end, not its start; pinning `#ℓ = #s` would
formalize the degenerate case and constrain a quantity the addition never
consults. Neither relaxation is merely notational: a shape-mismatched span can
still capture active positions, and it is then admissible and resolves. Over a
depth-2 text subspace, the spec `s = [1,1,5]`, `ℓ = [0,9,0]`
is T12-well-formed (`Pos(ℓ)`, `actionPoint(ℓ) = 2 ≤ 3 = #s`); its denotation
`{t : [1,1,5] ≤ t < [1,10,0]}` contains exactly the
depth-2 positions `[1,k]` with `2 ≤ k ≤ 10`, capturing whichever of those the
source binds; under the inherited condition (iii) it would be rejected
(`#ℓ = 3 ≠ m = 2`), and here it is admitted, resolving by restriction to those
bound positions. The non-level-uniform spec `s = [1,1,5]`, `ℓ = [0,9]`
(`actionPoint(ℓ) = 2 ≤ 3 = #s`, `#ℓ = 2 ≠ 3 = #s`) is admitted the same way:
its denotation `{t : [1,1,5] ≤ t < [1,10]}` captures the depth-2 positions
`[1,k]` with `2 ≤ k ≤ 9`. Beyond the two conditions the span's boundary tumblers are
constrained no further: the start `s` need not carry the subspace depth, need
not be a bound position of `d_s`, and need not be an S8a-well-formed V-position
at all — under the partial binding admitted below, `s` is a pure *boundary*,
and every well-formedness fact resolution consumes is recovered from the active
positions themselves, whose membership in `dom(Σ.M(d_s))` supplies S8a
(ASN-0036).

Gregory's udanax-green matches the relaxed admissibility: `acceptablevsa` — the
hook placed to validate V-addresses — is an unconditional pass;
`specset2ispanset` checks only that the document address is non-zero; and span
classification is pure tumbler-order intersection at full precision, with no
depth check, rejection, or normalization anywhere — a start with a zero
component, an unbound shape, or a mismatched depth selects less, never errors.
Its clipping arithmetic diverges at resolution: boundary differences are
converted to integer offsets in a way that discards sub-depth structure, so a
cross-depth boundary can be mis-read as a content offset. The abstract
resolution defined below is immune, restricting to the bound positions and
reading each image through the arrangement rather than by offset arithmetic.

A *spec-set* `R = ⟨ρ₁, …, ρ_q⟩` is a finite ordered sequence of V-specs
with `q ≥ 1` — ASN-0058's *ContentReferenceSequence* shape, its members relaxed
to V-specs as above (we write the spec-set length as `q`, reserving `p` for the
insertion position introduced with the operation below). The ordering is part of the request — Nelson is explicit that "if you
want to designate a separated series of items exactly, including nothing else, you
do this by a span-set, which is a series of spans" (4/25): a spec-set is a
*sequence*, not a set, and its "exactly" is the *exclusion* of unwanted
intermediate content between the named pieces.

Because `act` intersects the denotation with the *bound* positions, a V-spec whose
span names positions the source does not bind is admitted, and `act` resolves it by
restriction to the bound subset. COPY thus does not require ASN-0058's optional
full-binding well-formedness condition
(`{v : u ≤ v < reach(σ) ∧ #v = m} ⊆ dom(M(d_s))`); it acts on whatever the
boundaries determine. This is Nelson's span semantics: content is designated by its
boundaries, "what lies between... is implicit in the choice of first and last
point," and "a span that contains nothing today may at a later time contain a
million documents" (4/25).

We restrict attention to *content* spec-sets: under content-residence every active
position is in the text subspace, `subspace(vⱼ) = s_C`, so by referential integrity
(S3★) each resolves to a content address `Σ.M(d_s)(vⱼ) ∈ dom(Σ.C)`. The
single-subspace confinement and common depth noted above are the only
arrangement-side premises the arithmetic needs.

We define **resolution** as the flat I-address sequence obtained by expanding
ASN-0058's `resolve` (Resolution). ASN-0058 resolves a content reference
`(d_s, σ)` by reading the restriction `M(d_s)|⟦σ⟧`, decomposing it into maximal
runs ordered by V-start, and returning *compressed run-pairs*
`resolve(d_s, σ) = ⟨(a₁, n₁), …, (aₖ, nₖ)⟩`; a content-reference sequence resolves
by concatenation, `resolve(R) = resolve(ρ₁) ⌢ … ⌢ resolve(ρ_q)`. The flat sequence
we use is the address-by-address expansion of those run-pairs:

> `resolve(R, Σ) = expand(resolve(R))`,  where
> `expand(⟨(aⱼ, nⱼ)⟩ⱼ) = ⟨a₁, a₁+1, …, a₁+(n₁−1), …, aₖ, …, aₖ+(nₖ−1)⟩`     (CP0)

This is not a new object — it is ASN-0058's resolution listed one address at a
time rather than run-by-run, and every address it lists is the image
`Σ.M(d_s)(v)` of a bound active position `v ∈ act(ρ, Σ)`. Fix one V-spec
`ρ = (d_s, σ)`. Its runs `(vⱼ, aⱼ, nⱼ)` partition `act(ρ, Σ)` — the domain
`dom(M(d_s)|⟦σ⟧)` — into
disjoint maximal runs (ASN-0058, C1a), and the per-run lockstep those runs carry
(ASN-0058: MaximalRun condition 1 and decomposition consistency B3, extended to
restrictions by the same C1a — the restriction-level analogue of ASN-0036's S8
lockstep on whole arrangements) fixes each run's images in step with its bound
positions: `Σ.M(d_s)(vⱼ + k) = aⱼ + k` for every `0 ≤ k < nⱼ`. Hence the address `aⱼ + k`
that `expand` emits at interior offset `k` is *exactly* the image
`Σ.M(d_s)(vⱼ + k)` of the bound position `vⱼ + k ∈ act(ρ, Σ)` — run interiors
included. C1b (ResolutionSequenceOrder) lists the runs in strictly increasing
V-start order, fixing the order in which `expand` emits them; so across all runs
of `ρ` — and, concatenating in `R`'s order, across the whole spec-set — every
`cᵢ` the flat sequence lists, run-leading or interior, is the image of a bound
active position. Write
`resolve(R, Σ) = ⟨c₀, c₁, …, c_{W−1}⟩`, with `W = |resolve(R,Σ)|` the total count
of resolved addresses (the sum of the run widths `nⱼ`). Three facts about this
object we record as the *resolution integrity* claim CP0:

- **(a) Every resolved address already exists.** `cᵢ ∈ dom(Σ.C)` for `0 ≤ i < W`.
  We read this off the per-position grounding just established:
  each `cᵢ` — run-leading or run-interior alike — is `Σ.M(d_s)(v)` for some active
  position `v ∈ act(ρ, Σ) ⊆ dom(Σ.M(d_s))` with `subspace(v) = s_C`
  (content-residence), so referential integrity S3★ gives `Σ.M(d_s)(v) ∈ dom(Σ.C)`.
  *Nothing named by the spec-set is invented; it is all found.* This is the
  precondition that the placement to come will require, met by the source
  arrangements alone.
- **(b) Resolution is a pure read.** `resolve` is a function of `Σ`; it modifies no
  component — not `Σ.C`, not any `Σ.M(d)`, not `Σ.L`, not `Σ.R`. The source
  document is consulted, never altered, by the act of resolving a spec-set against
  it.
- **(c) Non-contiguity survives resolution.** ASN-0058's C1a
  (RestrictionDecomposition) supplies the unique maximal-run decomposition of *any*
  restriction `M(d_s)|⟦σ⟧` whose domain lies in a single subspace, and that
  single-subspace precondition is met here by content-residence
  (`act(ρ, Σ) ⊆ V_{s_C}(d_s)`). When a single V-span covers content the source
  itself assembled from several disjoint I-regions, that decomposition returns
  several run-pairs in V-start order (C1b, ResolutionSequenceOrder) — an ordering
  of whatever runs the bound subset yields, independent of binding — so the
  expanded sequence is *not* one contiguous run and records as many distinct
  origins as the source content had homes.

By the spec-set definition a spec-set has `q ≥ 1`; but even a
non-empty spec-set may resolve to `W = 0` when partial binding leaves every named
position unbound (resolution restricting to the empty bound subset). We exclude
that degenerate outcome from the operation below by requiring `W ≥ 1`, since
placing nothing is a no-op.

## The COPY operation

We now specify the placement. Let the destination be an allocated document
`d ∈ dom(Σ.M)` and let `p` be a V-position in its text subspace at which the
material is to land — a *valid insertion position*: `p = min(V_{s_C}(d))` or a
shift thereof when `V_{s_C}(d) ≠ ∅` (ASN-0036, ValidInsertionPosition), or the
canonical first position `[s_C, 1, …, 1]` when `V_{s_C}(d) = ∅`
(ValidFirstInsertionPosition). Let `resolve(R, Σ) = ⟨c₀, …, c_{W−1}⟩` with
`W ≥ 1`.

**Precondition — content residence.** Every active position of every V-spec in `R`
lies in the text subspace:
`(A ρ ∈ R, v ∈ act(ρ, Σ) : subspace(v) = s_C)`.

**COPY(`Σ, d, p, R`)** is the composite transition `Σ →* Σ'` (ASN-0047,
SequentialTransitionAxiom, reserves `Σ → Σ'` for atomic steps; COPY's
decomposition into atomic steps is exhibited in the next section, and every
clause below relates the initial state `Σ` to the final state `Σ'`), with the
following effect on the destination arrangement, displacement of its prior
content, and recording of provenance, and with the frame conditions that say
what it leaves alone.

*Effect — placement.* The `W` resolved addresses are bound, in order, to the `W`
V-positions starting at `p`:

> `(A i : 0 ≤ i < W : Σ'.M(d)(p + i) = cᵢ)`     (CP2)

*Effect — displacement.* Prior text content at or beyond `p` is shifted forward by
`W` to make room, its bindings carried intact:

> `(A v : v ∈ V_{s_C}(d) ∧ v ≥ p : Σ'.M(d)(v + W) = Σ.M(d)(v))`     (CP3a)

This is the post-insertion shift of ASN-0082 (I3, PostInsertionShift) instantiated
at width `W`.

*Effect — domain closure (text subspace).* The text-subspace V-positions of the
post-state are exactly the left-frame positions, the placement positions, and the
shifted positions — and *nothing else*; in particular the pre-shift positions in
`[p, max]` are *vacated*, not left doubly bound:

> `{v ∈ dom(Σ'.M(d)) : subspace(v) = s_C} =`
> `  {v ∈ V_{s_C}(d) : v < p} ∪ {p + i : 0 ≤ i < W} ∪ {v + W : v ∈ V_{s_C}(d) ∧ v ≥ p}`     (CP3c)

CP3c is the COPY analogue of ASN-0082's I3-V (PostInsertionVacating) and D-DOM
(domain characterization), which COPY's displacement otherwise borrows wholesale.

*Effect — provenance.* The provenance relation closes to the pre-state pairs plus
exactly one pair per placed address:

> `Σ'.R = Σ.R ∪ {(cᵢ, d) : 0 ≤ i < W}`     (CP8)

CP8's `⊇` direction is the membership `(A i : 0 ≤ i < W : (cᵢ, d) ∈ Σ'.R)`; its
`⊆` direction pins `Σ'.R ∖ Σ.R` to pairs of placed addresses with `d`, so no
spurious pair can enter.

*Frame — left of the insertion point.*

> `(A v : v ∈ V_{s_C}(d) ∧ v < p : Σ'.M(d)(v) = Σ.M(d)(v))`     (CP3b)

*Frame — content store.*

> `dom(Σ'.C) = dom(Σ.C) ∧ (A a : a ∈ dom(Σ.C) : Σ'.C(a) = Σ.C(a))`     (CP1)

*Frame — entity set.*

> `Σ'.E = Σ.E`     (CP12)

COPY mints no node, account, or document: the composite below contains no K.δ
step, and every step it does contain (K.μ⁻, K.μ⁺, K.ρ) frames `E` unchanged
(ASN-0047). CP12 completes the operation's *closure inventory* — the clauses
bounding each state component from above, where the effect clauses constrain
only from below: CP3c closes `d`'s text-subspace domain, so each text V-position
carries exactly one binding and `d`'s per-state invariants — S2 functionality
among them — are dischargeable from the postconditions alone; CP6's
domain-equality conjunct closes `d`'s non-text subspaces; CP8's `⊆` direction
bounds `Σ.R`; CP12 bounds `E`; and the remaining components are pinned outright
by the equalities CP1 (content store), CP7a (link store), and CP6's
other-document clause.

*Frame — link store, other subspaces, other documents.*

> `Σ'.L = Σ.L`     (CP7a)
>
> `{v ∈ dom(Σ'.M(d)) : subspace(v) ≠ s_C} = {v ∈ dom(Σ.M(d)) : subspace(v) ≠ s_C}`
> `  ∧ (A v : v ∈ dom(Σ.M(d)) ∧ subspace(v) ≠ s_C : Σ'.M(d)(v) = Σ.M(d)(v))`
>
> `(A d' : d' ≠ d : Σ'.M(d') = Σ.M(d'))`     (CP6)

The middle clause has two conjuncts. The pointwise conjunct preserves `d`'s
*pre-state* non-text bindings; the domain-equality conjunct pins `d`'s non-`s_C`
domain to exactly its pre-state value.

The last clause includes every source document `d_s ≠ d` and every other document
in the docuverse. The single self-reference case `d_s = d` — *self-transclusion* —
is admitted; see CP9.

## COPY as a valid composite

We must show that CP8's membership is *produced* rather than merely *required*,
that its bound is respected, and that the operation is admissible under
ASN-0047's transition discipline. We do all three at once by exhibiting COPY as
a valid ASN-0047 composite (ValidComposite) and reading the obligations off its
atomic steps. The decomposition splits on whether the insertion point has
trailing content to displace.

One observation serves both cases. *Placement-position well-formedness*: for a
valid insertion position `p`, every placement position `p + i` with `0 ≤ i < W`
is S8a-valid and carries the text-subspace common depth `m_{s_C}(d)`. For S8a:
`p` is itself S8a-valid (ValidInsertionPosition postcondition (b), or
ValidFirstInsertionPosition postcondition (b) when `V_{s_C}(d) = ∅`), and each
`p + i = shift(p, i)` preserves S8a by OrdShiftHom(b) (ASN-0036), with
`p + 0 = p` S8a-valid directly. For depth: when `V_{s_C}(d) ≠ ∅`,
`#p = m_{s_C}(d)` (ValidInsertionPosition postcondition (a)); when
`V_{s_C}(d) = ∅`, `m_{s_C}(d)` is *undefined* in the pre-state (ASN-0047,
LinkSubspaceDepth: `m_S(d)` is well-defined only while `V_S(d) ≠ ∅`) —
ValidFirstInsertionPosition fixes `#p = m` for a *chosen* parameter `m ≥ 2`, and
this choice is what *defines* `m_{s_C}(d) := m` for the post-state; we do not
equate `#p` with an as-yet-undefined depth. Either way, writing `m_{s_C}(d)` for
the now-established post-state depth,
`#(p + i) = #shift(p, i) = #p = m_{s_C}(d)`, so every placement position has
that depth — preserving S8-depth where the subspace was populated and
establishing it where it was empty, exactly as OrdShiftHom(b) preserves S8a.

*Append or empty case* (`p = max+1`, or `V_{s_C}(d) = ∅`). No prior position
satisfies `v ≥ p`, so CP3a is vacuous and the effect is a pure extension: a single
K.μ⁺ step adds the `W` placement positions `[p, p+W)` bound to `c₀,…,c_{W−1}`,
leaving every prior mapping intact — exactly K.μ⁺'s strict-extension frame
`(A v ∈ dom(Σ.M(d)) : Σ'.M(d)(v) = Σ.M(d)(v))`. The placement positions are
S8a-valid at depth `m_{s_C}(d)` by the placement-position observation. The
resulting text run is the contiguous block `[min, max+W]` (or `[p, p+W)` when
empty), discharging K.μ⁺'s D-CTG★/D-MIN★ precondition. CP3c is discharged in
its degenerate instance: no prior position satisfies `v ≥ p`, so its shifted
summand is empty and its left summand is the whole prior text domain; K.μ⁺ is
strict extension, vacating nothing and adding exactly the placement positions,
so the post-state text domain equals the prior text domain ∪ `[p, p+W)` —
CP3c's union.

*Displacing case* (`p ≤ max`, so trailing content exists). Here a pure K.μ⁺ is
*not* a faithful decomposition, and the difference is structural. The displacement
(CP3a) rewrites the existing binding `v ↦ a` into `(v+W) ↦ a`, which *removes* `v`
from `dom(Σ.M(d))` — whereas K.μ⁺ is strict extension and leaves every prior
mapping fixed, growing the domain only. No K.μ⁺ can vacate `v`. The effect is
therefore a *contraction-then-extension* composite. Write `p = min + j` with
`0 ≤ j < N`. **(i)** A K.μ⁻ step contracts `d`'s text subspace to the retained
prefix `[min, p)` — retention count `n'_{s_C} = j` (a strict contraction, since
`j < N`) — removing the trailing positions `{min + i : j ≤ i < N}`. K.μ⁻ takes a
*per-subspace* retention count (ASN-0047, K.μ⁻ PerSubspaceContractionScope), so we
must fix the link subspace too: the step retains `d`'s link subspace *in full*,
`n'_{s_L} = n_{s_L}`. This is a non-strict retention on `s_L`, which is admissible
because the text subspace already supplies the strict contraction K.μ⁻ requires of
*some* subspace (`n'_{s_C} = j < N`). Consequently the contraction leaves
`d`'s link-subspace V-positions untouched, and step (ii)'s K.μ⁺ adds only `s_C`
positions (the placement and the displaced trailing content) — so `d`'s
link-subspace arrangement is carried through *both* steps unchanged. This is what
discharges CP6's `subspace(v) ≠ s_C` conjunct: every non-text V-position of `d`
survives the composite with its binding intact. The
intermediate state `Σ₁` satisfies the per-state invariants: its text run
`{min + i : 0 ≤ i < j}` is a contiguous block from `min` (D-CTG★/D-MIN★ restricted
to `s_C`, the latter vacuous when `j = 0`); `Σ₁.M(d)` is a restriction of the
function `Σ.M(d)`, hence itself a function (S2) with surviving images unchanged and
still in `dom(Σ.C)` (S3★); and its domain is a subset of a finite set (S8-fin).
K.μ⁻ frames the content store, link store, and provenance unchanged
(`Σ₁.C = Σ.C`, `Σ₁.L = Σ.L`, `Σ₁.R = Σ.R`). **(ii)** A K.μ⁺ step then re-adds, on
top of the retained prefix, both the `W` placement positions `[p, p+W)` bound to
`c₀,…,c_{W−1}` (CP2) and the displaced trailing positions
`{(min+i)+W : j ≤ i < N} = [p+W, max+W]` — the equality by shift composition
(ASN-0034, TS3; ASN-0084, Extended Associativity), `(min+i)+W = min+(i+W)` —
bound to their original images `Σ.M(d)(min+i)` (CP3a). K.μ⁺'s image-membership
precondition holds for these displaced bindings: S3★ at `Σ` places each trailing
image `Σ.M(d)(min+i)` in `dom(Σ.C)`, and K.μ⁻'s content frame carries
`dom(Σ₁.C) = dom(Σ.C)` to the state where K.μ⁺ fires. Each retained mapping is
left intact — K.μ⁺'s strict-extension frame. The freshly added V-positions are
well-formed (S8a) and carry the subspace common depth, both kinds by one route:
a shift from an S8a-valid base of depth `m_{s_C}(d)` preserves S8a
(OrdShiftHom(b), ASN-0036) and length (`#shift(v, k) = #v`, OrdinalShift,
ASN-0034). The placement positions `{p + i : 0 ≤ i < W}` instantiate this at
base `p` (the placement-position observation); the displaced trailing positions
`{(min+i)+W : j ≤ i < N}` instantiate it at base `min + i ∈ V_{s_C}(d)`, which
is S8a-valid at depth `m_{s_C}(d)` by S8a and S8-depth on the pre-state
arrangement. The resulting text run is the
contiguous block `[min, max+W]`, discharging K.μ⁺'s D-CTG★/D-MIN★ precondition.
Steps (i)–(ii) together reproduce CP2, CP3a, and CP3b (the left prefix is
retained by (i) and untouched by (ii)). They produce CP3c as well: the
post-state text domain equals K.μ⁻'s retained prefix `[min, p)` united with
K.μ⁺'s added ranges `[p, p+W) ∪ [p+W, max+W]` — exactly CP3c's three-range
union — because step (i) vacates each pre-shift position in `[p, max]` of its
old binding and step (ii) reintroduces none of those bindings (each displaced
image re-enters only at `v + W`, and a vacated tumbler re-enters the domain
only as a placement or shifted position of the union, under its single new
binding), while nothing outside the union is added by either step.

To these arrangement steps the composite appends its provenance steps, with a
determinate inventory we fix once: **one K.ρ step per range-new address not
already in `Σ.R`**, and no other K.ρ step. It is these K.ρ steps that put pairs
into `Σ.R`. Each fires its
elementary precondition (ASN-0047, K.ρ: `a ∈ dom(C) ∧ d ∈ E_doc`) at the
intermediate state where it runs: `cᵢ ∈ dom(Σ.C)` by CP0(a), held across the
arrangement steps by the content frame CP1 (no K.α runs, so `dom(C)` does not
move), and `d ∈ dom(Σ.M) = E_doc` by hypothesis — so ValidComposite clause 1 (each
step's elementary precondition at its intermediate state) is discharged uniformly
across the K.μ⁻, K.μ⁺, and K.ρ steps alike. ASN-0047's ValidComposite further
requires all three couplings — J0, J1★, and J1'★ — to hold initial-to-final. J0 (AllocationPlacementCoupling), which demands that every
freshly allocated I-address appear in some arrangement, is discharged *vacuously*:
COPY runs no K.α step (CP1 gives `dom(Σ'.C) = dom(Σ.C)`), so
`dom(Σ'.C) ∖ dom(Σ.C) = ∅` and J0's universal quantifier ranges over the empty
set. The remaining two couplings carry the provenance obligation, and both are
universally quantified over *every* document of the post-state — J1★ over every
`d' ∈ E'_doc` and every address new to that document's content-subspace range,
J1'★ over every pair entering `R` — so the discharge must cover the full
quantifier, not only the destination's placed addresses. We split off the other
documents first.

*Every `d' ≠ d` — vacuous, by the frames.* The arrangement steps K.μ⁻ and K.μ⁺
frame every other document's arrangement unchanged (ASN-0047, the per-step
clause `(A d'' : d'' ≠ d : M'(d'') = M(d''))`), and each K.ρ step frames every
arrangement unchanged; the net effect is CP6's other-document clause. With CP12
giving `E'_doc = E_doc`, we have `Σ'.M(d') = Σ.M(d')` for every `d' ≠ d`: no
address is new to `d'`'s content-subspace range, and J1★'s instances at `d'`
range over the empty set — the same vacuity that discharged J0, reached through
the arrangement frames rather than the content frame. J1'★ is vacuous off the
destination for the symmetric reason: the composite's only `Σ.R`-touching steps
are its K.ρ recordings, each adding a pair `(cᵢ, d)` at `d`, so `Σ'.R ∖ Σ.R`
contains no pair `(a, d')` with `d' ≠ d`.

*The destination — range-new ⊆ placed.* At `d` itself, J1★ fires for every
address new to `d`'s content-subspace range, not only for the placed `cᵢ`, so
before any case split we must establish that nothing *but* a placed address can
be range-new. The couplings are evaluated *initial-to-final* (`Σ` to `Σ'`), so
the intermediate removal and re-addition of the displaced positions across
steps (i)–(ii) is invisible to the check; what matters is the net change in the
set J1★'s witness clause tests. Writing
`ran_C(Σ, d) = {a : (E v ∈ dom(Σ.M(d)) : subspace(v) = s_C ∧ Σ.M(d)(v) = a)}`
for that set — `d`'s *content-subspace range* — we claim the range equation

> `ran_C(Σ', d) = ran_C(Σ, d) ∪ {c₀, …, c_{W−1}}`

and read it off the arrangement clauses. `⊇`: every pre-state text image
survives — a position `v < p` keeps its binding (CP3b), and a position `v ≥ p`
carries its image intact to `v + W` (CP3a), still a text-subspace position
since shift preserves subspace (OrdShiftHom(a), ASN-0036) — and each `cᵢ`
enters as the image of the text-subspace position `p + i` (CP2). `⊆`: CP3c
closes the post-state text-subspace domain to exactly the left, placement, and
shifted positions, and the same clauses give their images — `Σ.M(d)(v)` for a
left position `v` (CP3b), `cᵢ` for a placement position (CP2), `Σ.M(d)(v)` for
a shifted position `v + W` (CP3a) — each either a pre-state text image or a
placed address. Hence `ran_C(Σ', d) ∖ ran_C(Σ, d) ⊆ {c₀, …, c_{W−1}}`: every
address J1★ can demand a record for at `d` is a placed address, and the
universal obligation reduces to deciding the membership `(cᵢ, d) ∈ Σ'.R` for
each placed `cᵢ`. (The link-discoverability section below derives the
full-range analogue `ran(Σ'.M(d)) = ran(Σ.M(d)) ∪ {c₀, …, c_{W−1}}` from the
same clauses together with CP6's non-text closure; the content-subspace
restriction proved here is the set J1★'s witness clause — `subspace(v) = s_C`
— actually consults.)

The discharge now turns on a single *membership* obligation per placed address,
which we must read off J1★ exactly. For
each `cᵢ` that is *range-new* — in `ran_C(Σ', d) ∖ ran_C(Σ, d)`, i.e. placed by
CP2 *and* not already in the content-subspace range `ran_C(Σ, d)` of the
pre-state `Σ` — ASN-0047's coupling J1★
(ExtensionRecordsProvenance) demands the membership `(cᵢ, d) ∈ Σ'.R`. (Placement
alone does not make an address range-new: a placed `cᵢ` that `d` already binds in
the pre-state — the self-transclusion of CP9, or content `d` previously transcluded
— is placed yet *not* range-new, and is handled in the "not range-new" branch
below.) J1★ is stated as a requirement on the *final* relation `Σ'.R`,
not as a demand for any particular atomic step, so it is satisfiable two ways, and
the range-new case splits accordingly:

- **Range-new and not previously recorded** (`(cᵢ, d) ∉ Σ.R`). Here permanence has
  nothing to carry, so the membership can only be produced by a fresh K.ρ step
  recording `(cᵢ, d)`. The resulting pair is genuinely new to `R`, so J1'★
  (ProvenanceRequiresExtension) constrains it — and permits it, since its
  condition is exactly that `cᵢ` be range-new. The composite includes one K.ρ per
  such address, producing `(cᵢ, d) ∈ Σ'.R`.
- **Range-new yet already recorded** (`(cᵢ, d) ∈ Σ.R`). This configuration is
  reachable: `d` referenced `cᵢ` earlier and then contracted those V-positions away
  (K.μ⁻ removes from the range; P2 keeps the provenance pair forever), so `cᵢ` is
  absent from the *current* range yet present in `Σ.R` — exactly a re-COPY of
  previously-deleted transcluded content. J1★'s membership `(cᵢ, d) ∈ Σ'.R` is
  already discharged by provenance permanence P2 carrying the pre-state pair
  forward, so the canonical composite emits *no* K.ρ step here, per the
  inventory fixed above. An *admissible variant* emits a redundant K.ρ anyway:
  the step is a no-op on the relation — the pair already stands in `Σ.R`, so it
  contributes nothing to `Σ'.R ∖ Σ.R`, and J1'★, which constrains only pairs
  genuinely new to `R`, is indifferent to it. Gregory's udanax-green takes the
  redundant route: deletion never clears the spanfilade, and `insertspanf`
  records unconditionally — no duplicate check exists anywhere in the insertion
  path — deduplicating only at query time.

For each `cᵢ` that is *not* range-new — already in the content-subspace range of
`M(d)` in the pre-state (already transcluded into `d`, or, in self-transclusion
CP9, already bound by one of `d`'s own pre-state content positions) — J1★ does not
fire at all. The membership still holds: by P4★ (`Contains_C(Σ) ⊆ R`, ASN-0047),
available since `Σ` is a composite boundary (standing precondition),
`(cᵢ, d) ∈ Σ.R` already, and provenance permanence (P2) carries it into `Σ'`.
Across all three branches the membership `(cᵢ, d) ∈ Σ'.R` holds; fresh recording
occurs exactly for the range-new addresses *not already in `Σ.R`*. CP8's `⊆`
direction holds by the same exhibition: the composite's only `Σ.R`-touching steps
are these K.ρ recordings, each adding a placed pair `(cᵢ, d)`, so
`Σ'.R ∖ Σ.R ⊆ {(cᵢ, d) : 0 ≤ i < W}` — with J1'★ independently confining any new
pair to range-new content of its document.

## The transclusion frame: content is referenced, never allocated

The single claim that makes COPY *transclusion* and not *copying* is CP1:
`dom(Σ'.C) = dom(Σ.C)`. This is the *defining frame condition* of COPY — a design
stipulation, not a theorem. We adopt it as the operation's content and then show
it is consistent with the placement, and that dropping it yields a different
operation (REPLICATE) entirely.

The placement (CP2) requires, for each `i`, that `Σ'.M(d)(p + i) = cᵢ` be a legal
arrangement binding — and since `p + i` is a content-subspace position (`p` is a
text-subspace insertion position, its subspace carried by OrdShiftHom(a)),
referential integrity (S3★, `s_C` branch) demands its image lie in the content
store, `cᵢ ∈ dom(Σ'.C)`. There are two ways an operation *could* discharge
this. The first is to *find* `cᵢ` already present; the second is to *allocate* a
fresh address and copy the value into it. *Given* CP1, only the first is available
— and it suffices: resolution integrity CP0(a) gives `cᵢ ∈ dom(Σ.C)`, and store
monotonicity (S1) lifts this to `cᵢ ∈ dom(Σ'.C)` with no growth. So under CP1 the
placement's referential-integrity obligation —
`(A i : 0 ≤ i < W : cᵢ ∈ dom(Σ'.C))` — is dischargeable: every address the
placement names is already present, and CP1 is in no tension with the binding it
must support. The destination binds *the very I-addresses the sources bind* — the
placed material and its source share one content identity, not two equal copies of
one value.

The contrast with the rejected alternative is the formal content of Nelson's
"inclusion by reference, not copy." Suppose an operation REPLICATE did the second
thing: allocate fresh `c'ᵢ ∉ dom(Σ.C)`, set `Σ'.C(c'ᵢ) = Σ.C(cᵢ)`, and bind
`Σ'.M(d)(p+i) = c'ᵢ`. Then `dom(Σ'.C) ⊋ dom(Σ.C)`, and — by the document-scoped
allocation discipline (S7a) — `origin(c'ᵢ) = d`, the destination. The placed
material would carry the destination's name, not the source's; its value would be
free to diverge from the source's future edits; and no address would connect it
back to where it came from. CP1 is precisely the prohibition of REPLICATE: *COPY
must add Vstream references to existing Istream content and must never mint new
Istream content for the included material.* The moment an operation stores fresh
native content for what it includes, it has manufactured a second identity, and it
is replication.

We record the immutability consequence as CP10: because `Σ.C` is untouched
(CP1), content immutability S0 is preserved trivially across the COPY composite
`Σ →* Σ'` — every atomic step frames `Σ.C`, so the preservation holds step by
step, hence initial-to-final — every previously stored address keeps its value,
and in particular the reused `cᵢ` carry into the destination *the same bytes*
they hold at the source, because they are the same bytes.

## What stays the source's, and what the destination makes its own

The split the operation enforces is sharp, and it falls along the
Istream/Vstream seam.

| Aspect | Status under COPY | By |
|---|---|---|
| I-address (content identity) | **shared** with source — same address bound | CP2 |
| content value (bytes) | **identical** — store unchanged | CP1, CP10 |
| `origin` / home document | **unchanged** — the original allocator's, read off the address | CP5 |
| ownership of the content | **unchanged** — the original allocator's | CP5 |
| V-position (arrangement slot) | **the destination's own**, freshly bound | CP2 |
| provenance record `(cᵢ, d)` | **the destination's own**, freshly recorded | CP8 |

The destination owns its arrangement — the *placement* of the material in its own
virtual byte stream — and the provenance entry recording that it now refers to the
content. It does not own, and COPY does not transfer to it, the content's
identity, value, or home. This is CP5, **OriginInvariance**: for every placed
address, `origin(cᵢ)` is unchanged by COPY (CP1 keeps `cᵢ` in the store,
and S7(d) makes `origin` constant while it is stored), and it equals the document
that *originally allocated* `cᵢ`. That allocator may be the spec-set source, a
third document the source had itself transcluded from (a chained transclusion), or
`d` itself — when `d` copies back content it once allocated, or self-transcludes.
COPY's guarantee is not "never `d`" but *invariance*: whoever allocated `cᵢ` keeps
the attribution, since COPY never reallocates and S7(d) holds `origin` fixed while
`cᵢ` is stored. Attribution is *structural*: it is read off the address, not stored
as detachable metadata, so the placement cannot strip it. The owner of the source content
retains the content; the destination has acquired an arrangement and a reference,
nothing more. This is exactly Nelson's "Document A can include Document B, even
though Document B is owned by someone else" (2/35), made into a frame condition.

## The destination's prior arrangement is preserved

Placing material at `p` must not damage what the destination already held. CP3
discharges this in three parts. Content strictly before `p` is untouched (CP3b).
Content at or beyond `p` is shifted forward uniformly by `W` (CP3a), its bindings
carried intact — no existing V-position loses its content, and the relative order
of any two prior positions is preserved because shift is strictly
order-preserving (ASN-0034, TS1: `u < v ⟹ u + W < v + W`) and injective (TS2).
Nothing prior is deleted, reordered relative to its neighbours, or rebound to
different content. The only change to prior content is a uniform forward
displacement of the trailing region's V-addresses.

Two consequences matter beyond bookkeeping. First, because the prior content's
*I-addresses* are unchanged — only its V-positions move — anything anchored to
those I-addresses survives the placement unmoved; links attach to content
identity, not arrangement position (CP7 below). Second, the post-state arrangement
is again a well-formed, contiguous, sequential text subspace. We derive the
no-holes tiling explicitly from ordinal arithmetic, splitting on whether the
destination's text subspace is already populated.

*Empty destination* (`V_{s_C}(d) = ∅`, so `N = 0`). Here COPY establishes the
document's first content rather than preserving prior content. The valid insertion
position is the canonical `p = [s_C, 1, …, 1]` (ValidFirstInsertionPosition); the
displacement (CP3a) and left-frame (CP3b) clauses are vacuous — there is no prior
`v ≥ p` to shift and none `v < p` to fix — and the placement (CP2) lays the `W`
positions `p, p+1, …, p+(W−1)`. The post-state run is `{p + i : 0 ≤ i < W}`, which
has minimum `p = [s_C, 1, …, 1]` (D-MIN *established*), is the sequential block of
`W` positions from that minimum (D-SEQ *established*), and has no interior hole
(D-CTG). In the empty case D-MIN and D-SEQ are established, not preserved — COPY
lays down the first content the document holds.

*Non-empty destination* (`V_{s_C}(d) ≠ ∅`, so `N ≥ 1`). Before COPY, `V_{s_C}(d)`
is the contiguous run `{min + i : 0 ≤ i < N}` with `min = [s_C, 1, …, 1]` (D-MIN)
and `N = |V_{s_C}(d)|` (D-SEQ), so its top is `max = min + (N−1)`. The valid
insertion position `p` is `min + j` for some `0 ≤ j ≤ N` (ASN-0036,
ValidInsertionPosition), i.e. `p`'s ordinal lies in `[min, max+1]`. COPY lays
the post-state out as three ordinal ranges, using `+` for the ordinal shift:

- *Left, unmoved* (CP3b): the positions `v < p`, namely `{min + i : 0 ≤ i < j}`,
  occupying the ordinal interval `[min, p)`.
- *Placement* (CP2): the `W` positions `p, p+1, …, p+(W−1)`, occupying `[p, p+W)`.
- *Shifted right* (CP3a): each `v ≥ p`, namely `{min + i : j ≤ i < N}`, moves to
  `v + W`, occupying `[p+W, max+W]`.

These three ordinal intervals are consecutive and non-overlapping: the left run
ends just below `p` where the placement begins, and the placement ends just below
`p+W` where the shifted run begins. Disjointness is ordinal arithmetic, not I3 —
shift is strictly order-preserving (ASN-0034, TS1) and a strict advance
(`v + W > v`, TS4), so no shifted position lands at or below any placement or
left position, and the intervals do not overlap. Nor is there a gap: each
`v = min + i` with `j ≤ i < N` shifts to `(min + i) + W = min + (i + W)` by shift
composition (ASN-0034, TS3; ASN-0084, Extended Associativity), so the shifted
positions are exactly the consecutive ordinals
`{min + (i+W) : j ≤ i < N} = [p+W, max+W]`, abutting the placement block
`[p, p+W)` at `p+W` with nothing skipped. Their union is the single contiguous
run `[min, max+W] = {min + i : 0 ≤ i < N+W}`. Hence
`V_{s_C}(d)` after COPY has minimum `min` unchanged (D-MIN preserved), is the
sequential block of `N+W` positions from `min` (D-SEQ preserved), and has no
interior hole (D-CTG preserved). The destination's editorial order — "this
order may be continually altered by editorial operations" (4/30) — has been
extended, not corrupted.

## Source isolation, and the asymmetry of awareness

CP6 states that COPY modifies no document but the destination: every source `d_s`
distinct from `d`, and every other document, has `Σ'.M(d_s) = Σ.M(d_s)`. Combined
with the read-only character of resolution (CP0(b)) and the content, link, and
entity frames (CP1, CP7a, CP12), this says the source is untouched by the act of
being copied from. The COPY composite writes to `Σ.M(d)`, `Σ.R`, and nothing
else.

Yet *isolation of the act* is not *unawareness of the connection*, and the two
must not be confused. Because the placed material shares the source's I-address
(CP2), the connection from source to destination is recoverable from the source
side: the source's content now appears, by the same address, inside `d`, and any
query that asks "which documents refer to this address?" — the provenance relation
`Σ.R`, which CP8 has just extended with `(cᵢ, d)` — answers with `d` among them.
This is the architectural opposite of what copying does. A true copy severs the
source from the destination; Nelson's "detached copy ... is frozen and dead,
lacking access to the new linkage" (2/48). Transclusion, by sharing identity,
keeps the source permanently and discoverably connected while changing nothing
about it. The source document's *state* is isolated from the copy; the source
content's *connectedness* is enlarged by it. COPY is the operation that achieves
both at once — and it can do so only because it never copies.

## Shared identity across documents

CP2 binds destination V-positions to addresses that other documents may already
bind. We record the consequence as CP4, **MultiplicityIncrease**: COPY adds `W`
new `(document, V-position)` references (one per placement, CP2), so the total
number of references into the placed set `{c₀, …, c_{W−1}}` increases by exactly
`W`. The exactness rests on the same domain closure that pins the range: the
shifted bindings *replace* their pre-state reference pairs rather than adding to
them (each `(v, a)` becomes `(v + W, a)`, the pre-shift position vacated by
CP3c), and CP3c with CP6 — the domain-equality conjunct for `d`'s non-text
subspaces, the other-document clause for the rest of the docuverse — excludes
any other reference from being created or destroyed. The per-address arithmetic is finer: for a fixed placed address `cᵢ`, its own
reference count increases by the number of times `cᵢ` *occurs in*
`resolve(R, Σ)` — at least one, and more only when a single source address is
resolved at several positions of the sequence. The aggregate increase `W` and the
per-address increase (the occurrence count) are distinct quantities, and they
coincide only when every resolved address is distinct. This
realizes the unrestricted-sharing capacity of the strand model (ASN-0036, S5): one
I-address may be referenced from arbitrarily many V-positions across arbitrarily
many documents. Each such reference is an independent arrangement entry. Two
distinct V-positions binding the same `cᵢ` cannot be merged or identified — they
are permanently independent occurrences of one shared identity (ASN-0058, M14
IndependentOccurrences, M14a). The documents that already shared `cᵢ` are
unaffected (CP6); the destination simply joins the set of references to a single,
unchanged identity. *The shared thing is one identity referenced from many places,
never many copies of one identity* — which is precisely what makes attribution
traceable, correspondence between the appearances computable, and a change to the
content coherent across every reference.

Self-transclusion (`d_s = d`) is the same phenomenon within one document. CP9
records it: when the source and destination are one document, resolution reads the
pre-state, so the placed addresses are those the document's own positions bound
*before* the displacement; the effect then adds new V-positions, in the same
document, referring to the same I-addresses. The result is a document with two (or
more) V-positions mapping to one address — admitted by S5 and permanently
independent by M14. No content is duplicated; the arrangement simply references
the same identity twice.

## Survival of links anchored to the reused content

A link's endpoints are *endsets* — sets of spans over I-addresses (ASN-0043) — and
a link is discoverable from a document `d` exactly when some endset's address
coverage meets the document's arrangement range, `coverage(e) ∩ ran(Σ.M(d)) ≠ ∅`
(ASN-0098, LP12). Links bind to content identity, not arrangement position. CP7
collects what COPY must guarantee about them.

First, the link store is framed: `Σ'.L = Σ.L` (CP7a). COPY creates, alters, and
deletes no link. Second — and this is the substantive guarantee, CP7b
**LinkSurvivalUnderReuse** — placing the resolved addresses into the destination's
range makes every link anchored to them discoverable from the destination. Let `a`
be a link with `coverage(Σ.L(a).eⱼ) ∩ {c₀, …, c_{W−1}} ≠ ∅` for some endset `j`.
After COPY the placed addresses are in `ran(Σ'.M(d))` (CP2), so
`coverage(Σ.L(a).eⱼ) ∩ ran(Σ'.M(d)) ≠ ∅`, and the discoverability
characterisation evaluated at the post-state — `discoverable_from(a, d, Σ') ⟺
(E i : coverage(Σ'.L(a).eᵢ) ∩ ran(Σ'.M(d)) ≠ ∅)` (ASN-0098, LP12), with
`Σ'.L = Σ.L` by CP7a so coverage is unchanged — yields `a` discoverable from `d`.
The link survives the reuse of its content and is
inherited by the new transcluding document — *because* COPY shares the I-addresses
rather than copying them. Had COPY allocated fresh addresses (REPLICATE), the
fresh addresses would lie outside every existing link's coverage, and not one link
would follow the placed material; the connections would be lost, the copy "frozen
and dead." This discoverability holds regardless of the link's home document and
regardless of which *other* documents already share those addresses, because the
LP12 criterion mentions only `ran(Σ'.M(d))` — the destination's own range — and is
silent on `home(a)` and on every other document: any
document that transcludes content covered by a link can discover that link, which
is the multi-endpoint, refraction-friendly behaviour Nelson requires of windowed
content.

Fix a link `a` not already discoverable from `d` at `Σ`, and ask what must hold
of `Σ` for `a` to be discoverable from `d` after COPY. Pulling the post-state criterion back through
the operation — `Σ'.L = Σ.L` (CP7a), and
`ran(Σ'.M(d)) = ran(Σ.M(d)) ∪ {c₀, …, c_{W−1}}` (`⊇`: CP2 adds the placed
addresses, while CP3a/CP3b move prior text positions but preserve their
I-addresses and CP6 fixes the non-text bindings, so the prior range is retained;
`⊆`: CP3c closes the text-subspace domain to exactly the left, placement, and
shifted positions — whose images are the prior text images and the placed
addresses — and CP6's domain-equality conjunct pins the non-text domain to its
pre-state value, so no other address can enter the range) — gives

> `wp(COPY, "a discoverable from d") ≡ enabled(COPY(Σ, d, p, R)) ∧ (E j : coverage(Σ.L(a).eⱼ) ∩ {c₀, …, c_{W−1}} ≠ ∅)`,

matching the `enabled(op) ∧ ⟨pullback⟩` form of ASN-0098's LP12a. Here
`enabled(COPY(Σ, d, p, R))` is the operation's applicability predicate in
full — `d ∈ dom(Σ.M)`; `p` a valid insertion position for `d`; every member of
`R` a V-spec admissible at `Σ`, i.e. its source allocated (`d_s ∈ dom(Σ.M)`),
its span T12-well-formed, and its source subspace non-empty
(`V_{subspace(s)}(d_s) ≠ ∅`); content residence; and `W ≥ 1`. The spec-set
conjuncts are partly *state-dependent* — source allocation and subspace
non-emptiness are facts about `Σ`, not about the syntax of `R`, and without
`d_s ∈ dom(Σ.M)` the arrangement `Σ.M(d_s)`, hence `resolve(R, Σ)`, is
undefined — so they belong inside the guard: it is this full predicate that
makes the pullback well-formed, `W` and the `cᵢ` being defined exactly on the
pre-states where every member of `R` resolves and COPY applies.

The pullback conjunct is not vacuous: it fails precisely when none of the resolved
addresses lies under any of `a`'s endsets, and in that case COPY — though it
faithfully shares identity — brings `a` no closer to `d`. Discoverability is thus
*conditional on what the spec-set names*, which is exactly the lever transclusion
gives an author: to inherit a link, place content the link already covers.

Third, links anchored to the destination's *prior* content survive. The
displacement moves prior V-positions but preserves their images, so every prior
I-address is retained in the post-state range:
`ran(Σ'.M(d)) ⊇ {Σ.M(d)(v) : v ∈ V_{s_C}(d)}`, since each `v < p` keeps its binding
(CP3b) and each `v ≥ p` carries it to `v + W` with `Σ'.M(d)(v + W) = Σ.M(d)(v)`
(CP3a). A link whose endset coverage meets `{Σ.M(d)(v) : v ∈ V_{s_C}(d)}` therefore
still meets `ran(Σ'.M(d))`, and LP12 at the post-state — with `Σ'.L = Σ.L` (CP7a)
leaving coverage unchanged — keeps it discoverable from `d`. The strap stays on the
same bytes even as their V-positions slide forward.

## Non-contiguous assembly, and the boundary between reuse and replication

A copy from a single contiguous source is the case where transclusion and
replication look most alike: one block of content in, one block out, and only the
*identity* of the resulting addresses — shared versus minted — tells them apart.
Assembling from several non-contiguous sources is the case that forces the
distinction into the open, and CP11 makes this precise.

By CP0(c), a spec-set drawn from non-contiguous sources resolves to a sequence
`⟨c₀, …, c_{W−1}⟩` whose addresses fall into several maximal contiguous runs,
each run drawn from one source region. The placement (CP2) lays these out as a
block decomposition of the destination's new region, one mapping block per run
(ASN-0058, bundle algebra). Within a block, all addresses share an origin
(`origin(cᵢ + 1) = origin(cᵢ)` for contiguous addresses, ASN-0058 M16a); across a
block boundary where the origins differ, the blocks *cannot be merged* (ASN-0058,
M16 CrossOriginMergeImpossibility). Therefore the multiset of origins carried by
the placed material — written with multiset brackets `⦃·⦄`, counted once per
placed address, so a home contributing several addresses appears with that
multiplicity —

> `⦃ origin(cᵢ) : 0 ≤ i < W ⦄`,

is preserved verbatim into the destination's arrangement: each fragment retains
its distinct home, and each home remains queryable from the destination address
that binds it. This is CP11, **OriginMultisetPreservation**.

Now contrast replication. Under REPLICATE — the rejected operation of the
transclusion-frame section, where `origin(c'ᵢ) = d` was already established for
every minted address — the origin multiset collapses to `⦃d, d, …, d⦄`, and the
seams between the source regions are erased. So discontiguity is a *test* that distinguishes
the two operations even when a single-source copy would not: COPY's non-contiguous
placement names `k` distinct parents, each still live, each still owed
attribution; REPLICATE's names one, the destination, and the provenances are
gone. Nelson's "annotated collage" assembled from many windows (2/45) is exactly
the structure CP11 preserves, and the inert single-origin mass is exactly what
CP1 forbids. The boundary the design must hold is the boundary of CP1 — *add
references, never mint content* — and the non-contiguous case is where holding it
or breaking it becomes visible at every seam.

## A worked assembly from two sources

We instantiate the operation on concrete tumblers, choosing a non-contiguous
two-source case so that CP11 has something to separate. Use the document-level
prefixes `d_A = 1.0.1.0.7`, `d_B = 1.0.1.0.8`, and destination `d = 1.0.1.0.9`
(each `zeros = 2`, a document address). Content addresses sit at `[d.0.s_C.k]`
with `s_C = 1` (ASN-0093), so they have `zeros = 3` and subspace identifier `1`.
Let the stores hold

- `a₁ = 1.0.1.0.7.0.1.1`, `a₂ = 1.0.1.0.7.0.1.2` (source A's content), and
- `b₁ = 1.0.1.0.8.0.1.1` (source B's content),

with arrangements (text subspace, depth-2 V-positions `[1, k]`)

- `Σ.M(d_A) = { [1,1] ↦ a₁, [1,2] ↦ a₂ }`,
- `Σ.M(d_B) = { [1,1] ↦ b₁ }`,
- `Σ.M(d)   = { [1,1] ↦ x₁, [1,2] ↦ x₂ }` with `x₁, x₂` allocated by `d`
  itself, so `origin(x₁) = origin(x₂) = d`.

Take the spec-set `R = ⟨(d_A, σ_A), (d_B, σ_B)⟩` with
`σ_A = ([1,1], δ(2,2)) = ([1,1], [0,2])` and `σ_B = ([1,1], δ(1,2)) = ([1,1], [0,1])`.
Both widths are ordinal displacements at the V-position depth
(`actionPoint(ℓ) = #ℓ = 2 = #s`) — the fully aligned case, though admissibility
requires no such alignment.
Their denotations are `⟦σ_A⟧ = {t : [1,1] ≤ t < [1,3]}` and
`⟦σ_B⟧ = {t : [1,1] ≤ t < [1,2]}`, so `act((d_A,σ_A),Σ) = {[1,1],[1,2]}` and
`act((d_B,σ_B),Σ) = {[1,1]}`. Resolution reads each through its arrangement:

> `resolve(R, Σ) = ⟨a₁, a₂⟩ ⌢ ⟨b₁⟩ = ⟨a₁, a₂, b₁⟩`,  so `W = 3`.

Place at `p = [1,2]` — a valid insertion position, since with `N = 2` the
admissible positions are `[1,1], [1,2], [1,3]` (here `j = 1`). COPY yields

> `Σ'.M(d) = { [1,1] ↦ x₁, [1,2] ↦ a₁, [1,3] ↦ a₂, [1,4] ↦ b₁, [1,5] ↦ x₂ }`.

We check the claims numerically.

- **CP1 (store unchanged).** `a₁, a₂, b₁, x₁, x₂` are all pre-existing; the
  binding introduces no fresh I-address, so `dom(Σ'.C) = dom(Σ.C)`.
- **CP2 (placement).** `[1,2] ↦ a₁`, `[1,3] ↦ a₂`, `[1,4] ↦ b₁` — the `W = 3`
  resolved addresses at `p, p+1, p+2`.
- **CP3a (shift).** The one prior position `≥ p`, `[1,2] ↦ x₂`, moves to
  `[1,2] + 3 = [1,5] ↦ x₂`; CP3b leaves `[1,1] ↦ x₁` untouched. The post-state
  text run is `{[1,1], …, [1,5]}` — contiguous, `N + W = 5` positions, minimum
  `[1,1]` unchanged.
- **CP11 (origin multiset).** `origin(a₁) = origin(a₂) = 1.0.1.0.7 = d_A` and
  `origin(b₁) = 1.0.1.0.8 = d_B`, so the placed multiset is
  `⦃d_A, d_A, d_B⦄`, carried verbatim into `d`'s arrangement. `a₁, a₂` form one
  block (`a₂ = a₁ + 1`, same origin); `b₁` is a second block whose origin differs,
  so the two cannot merge (ASN-0058, M16). Two live parents are named, neither of
  them `d`. A REPLICATE would instead mint three addresses under `d` and collapse
  the multiset to `⦃d, d, d⦄`, erasing the seam between source A and source B.
- **CP8 (provenance).** We classify each resolved address against `d`'s pre-state
  content-subspace range, `ran(Σ.M(d))|_{s_C} = {x₁, x₂}`. None of `a₁, a₂, b₁`
  lies in `{x₁, x₂}` — each was allocated by a source (`d_A` or `d_B`), so by S4
  (OriginBasedIdentity) it is distinct from `d`'s own `x₁, x₂`. All three placed
  addresses are therefore *range-new*: the placement (CP2) makes each new to
  `d`'s content-subspace range in `Σ'`, so J1★ obliges the membership
  `(·, d) ∈ Σ'.R` for each. Stipulating that `d` has never referenced them
  (`(a₁, d), (a₂, d), (b₁, d) ∉ Σ.R`), the canonical inventory runs one K.ρ per
  address, and J1'★ admits exactly those three. The composite thus runs three
  provenance steps, yielding `Σ'.R = Σ.R ∪ {(a₁, d), (a₂, d), (b₁, d)}` — fresh
  recording for every range-new address, and nothing else enters (CP8's `⊆`
  direction).

  To exhibit the already-referenced branch, vary the spec-set to *re-place* `d`'s
  own `x₁`: append `(d, σ_x)` with `σ_x = ([1,1], δ(1,2))`, resolving the extra
  address `x₁` (a self-transclusion, CP9). Now `x₁ ∈ ran(Σ.M(d))|_{s_C}` already in
  the pre-state, so the not-range-new branch applies: `(x₁, d) ∈ Σ.R` by P4★
  (`Σ` is a composite boundary), and provenance permanence P2 carries it into
  `Σ'`. The membership `(x₁, d) ∈ Σ'.R` holds without any fresh K.ρ record.

The numbers exhibit the whole point: the destination's V-positions are new, but
every I-address and every origin in the assembly is borrowed, intact, from the
sources.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| CP0 | `resolve(R, Σ)` reads each active source position through its arrangement, in spec-set order, yielding `⟨c₀,…,c_{W−1}⟩` with (a) every `cᵢ ∈ dom(Σ.C)`, (b) resolution a pure read of `Σ`, (c) non-contiguity of sources preserved as distinct runs | introduced |
| CP1 | TransclusionFrame: `dom(Σ'.C) = dom(Σ.C) ∧ (A a ∈ dom(Σ.C) : Σ'.C(a) = Σ.C(a))` — COPY allocates no content; the placed material refers to existing I-addresses | introduced |
| CP2 | Placement: `(A i : 0 ≤ i < W : Σ'.M(d)(p + i) = cᵢ)` — `W` destination V-positions, freshly bound, carry the resolved (pre-existing) I-addresses; the placed material shares the source's content identity | introduced |
| CP3 | PriorArrangementPreservation: left content unchanged (CP3b, `v < p`), trailing text content shifted forward by `W` with bindings intact (CP3a, `v ≥ p`), and the text-subspace domain closed to left ∪ placement ∪ shifted with the pre-shift positions vacated (CP3c) — so S2 functionality is dischargeable from the postconditions; order-preserving, injective, non-destructive | introduced |
| CP4 | MultiplicityIncrease: total references into the placed set increase by exactly `W`; each placed `cᵢ`'s own reference count increases by its occurrence count in `resolve(R, Σ)` (≥ 1); distinct V-positions binding one address are permanently independent occurrences (S5, M14) | introduced |
| CP5 | OriginInvariance: `origin(cᵢ)` is unchanged by COPY (S7(d)) and equals the document that *originally allocated* `cᵢ` — the spec-set source, a third document the source transcluded from, or `d` itself (copy-back / self-transclusion); attribution and ownership stay with that allocator | introduced |
| CP6 | SourceIsolation: `(A d' ≠ d : Σ'.M(d') = Σ.M(d'))` and cross-subspace frame, the latter closing `d`'s non-`s_C` domain to its pre-state value (`{v ∈ dom(Σ'.M(d)) : subspace(v) ≠ s_C} = {v ∈ dom(Σ.M(d)) : subspace(v) ≠ s_C}`) with bindings preserved — every source and every other document is unmodified; the source's connectedness nonetheless grows (shared identity + provenance) | introduced |
| CP7 | Links: (a) `Σ'.L = Σ.L`; (b) LinkSurvivalUnderReuse — any link whose endset coverage meets `{c₀,…,c_{W−1}}` becomes discoverable from `d` in `Σ'`; links to the destination's prior content remain discoverable (prior images retained in range via CP3a/CP3b, LP12) | introduced |
| CP8 | ProvenanceClosure: `Σ'.R = Σ.R ∪ {(cᵢ, d) : 0 ≤ i < W}` — membership (`⊇`) by a fresh K.ρ step for range-new addresses not already in `Σ.R` (J1'★-admissible; the canonical composite's only K.ρ steps), by permanence P2 for range-new addresses already in `Σ.R` (re-COPY of deleted content; a redundant K.ρ is an admissible no-op variant), and by P4★ + P2 for addresses already in `d`'s current range; the `⊆` direction pins `Σ'.R ∖ Σ.R` to the placed pairs | introduced |
| CP9 | SelfTransclusionAdmissibility: when `d_s = d`, resolution reads the pre-state, so placement adds independent V-positions of `d` referring to addresses `d` already bound; no content is duplicated | introduced |
| CP10 | ImmutabilityPreservation: S0 preserved across COPY (corollary of CP1); reused content carries identical bytes into the destination because they are the same bytes | introduced |
| CP11 | OriginMultisetPreservation: `⦃origin(cᵢ) : 0 ≤ i < W⦄` is preserved into the destination's arrangement; cross-origin blocks cannot merge (M16) | introduced |
| CP12 | EntityFrame: `Σ'.E = Σ.E` — COPY mints no node, account, or document (the composite runs no K.δ step); with CP8's `⊆` direction, every state component is bounded above by the operation's own clauses | introduced |

## Open Questions

ASN-0058's C2 (ResolutionWidthPreservation) equates a content reference's resolved
width with its named ordinal extent only for *well-formed* (fully bound) spans;
under the partial binding COPY admits, the resolved width `W` may fall strictly
below the named extent. What, if anything, must COPY guarantee about the
relationship between a partially-bound span's nominal extent and its smaller placed
width, given that the design treats the shortfall as silent?

What must the operation guarantee when a spec-set assembles source spans whose
boundary tumblers carry differing depths into one destination region?

Under what conditions must a link discoverable from the destination after COPY
become undiscoverable again if the destination later removes the transcluded
positions?

What relationship must hold between the shared identity COPY establishes and the
correspondence relation that lets a reference to one appearance of content serve
as a reference to all its appearances?

What must transclusion into the link subspace guarantee, and how does placing a
link by reference differ from placing content by reference?
