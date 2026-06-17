> **ASN-0119 · REARRANGE Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0084 · Cut-Point Rearrangements](../foundation/ASN-0084-bundle-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0119-rearrange-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0119: REARRANGE Operation

*2026-06-08*

## The problem

We are asked what happens when two regions of a document are transposed. Nelson
states the operation flatly: "Rearrange transposes two regions of text. With
three cuts, the two regions are from cut 1 to cut 2, and from cut 2 to cut 3...
With four cuts, the regions are from cut 1 to cut 2, and from cut 3 to cut 4"
(4/67). The sentence describes a *motion* — content that was here is now there,
content that was there is now here. It says nothing about what is *conserved*
under that motion, and the conservation is the whole of the matter.

The word *transposes* must be read against the design's deepest commitment: a
document is a mapping from positions to content, not the content itself. "The
address of a byte in its native document is of no concern to the user or to the
front end; indeed, it may be constantly changing; the front-end application is
unaware of this" (4/11). What an editorial operation rearranges is the mapping,
not what is mapped. Nelson is explicit that this is why links endure: "Note that
this order may be continually altered by editorial operations, but since the
links are to the bytes themselves, any links to those bytes remain stably
attached to them" (4/30).

So the task is to specify a permutation. We must say precisely which positions
are reassigned, what each one now denotes, what is left untouched, and then
discharge a list of obligations: that no content is created or destroyed, that
the document's total extent is conserved, that moved content remains findable
under its new position, that links survive the reordering, and that any other
document sharing the rearranged content is wholly unaffected. We will find that
every one of these obligations follows from a single structural fact — that
REARRANGE rewrites only the arrangement and never touches an I-address.

## The two streams

We work in the extended state `Σ = (C, L, E, M, R)` of ASN-0047 — its *state*,
with REARRANGE imported as an atomic arrangement-rearrangement primitive
(ASN-0084) that realizes the same *form* of arrangement change as ASN-0047's own
non-atomic `K.μ~` composite without ever vacating content, coinciding with an
admissible `K.μ~` whenever the net effect is non-trivial (`M'(d) ≠ M(d)`).
Coincidence is a five-clause claim — K.μ~'s admissibility (i)–(v) — whose four
clauses beyond non-triviality (ii) we discharge at the end of the
invariant-preservation section, once the facts they rest on are in hand. The
coincidence is not an equality of domains: K.μ~'s admissibility demands a
non-trivial net effect over an arrangement whose content-subspace mapping takes
at least two distinct values, while R-PRE imposes no value-based condition.
Shared content being reachable (ASN-0036, S5), an affected interval whose every
position maps to one I-address is legal, and a pivot on it is REARRANGE-legal
with `π ≠ id` yet `M'(d) = M(d)` — a value-degenerate identity-effect instance
that no admissible `K.μ~` realizes.
The model lifts the strand model (ASN-0036) and the link model
(ASN-0043) into a single arrangement whose V-positions inhabit two subspaces — the
text subspace `s_C` and the link subspace `s_L` (ASN-0047, S3★-aux). REARRANGE
mutates only the arrangement family `M`; the content store `C` and the link store
`L` are frames (`Σ'.C = Σ.C` and `Σ'.L = Σ.L` **(RA6)**), and the entity set `E` and the
provenance relation `R` are frozen alongside them (`Σ'.E = Σ.E` and `Σ'.R = Σ.R`
**(RA4)**). ASN-0084's frame names only the content store and the arrangement; lifting
the operation into the extended `(C, L, E, M, R)` state, we extend that frame with an
explicit clause for each component it does not name — RA6 for the link store `L`, and
RA4 for `E` and `R`. The
*content store* `Σ.C : T ⇀ Val` (ASN-0036, S0) is append-only and immutable; an
address `a ∈ dom(C)`, once allocated, denotes its value forever. This is the
Istream: the permanent record of *what content exists*. The *arrangement*
`Σ.M(d) : T ⇀ T` of a document `d` maps V-positions to I-addresses; it is the
Vstream, the record of *how content is currently ordered* in `d`. In the extended
model an arrangement carries both content and link V-positions — a content
position (`subspace(v) = s_C`) maps into `dom(C)`, a link position
(`subspace(v) = s_L`) into `dom(L)` (ASN-0047, S3★). The *link store*
`Σ.L : T ⇀ Link` (ASN-0043) records typed associations whose endsets reference
content by address. V-positions and I-addresses are tumblers ordered by T1
(ASN-0034); within the text subspace the active V-positions are contiguous and
share a common depth (ASN-0047, D-CTG★, D-SEQ★; S8-depth). On `s_C` these starred
per-subspace invariants coincide with ASN-0036's unstarred D-CTG/D-MIN/D-SEQ. The
link subspace `s_L` is carried untouched in the frame.

The distinction the operation turns on is the one ASN-0034's T6 already records:
*address versus position*. An I-address is permanent content identity; a
V-position is a mutable coordinate in one document's current order. REARRANGE
lives entirely in the second of these. We will write `M(d)(v)` for the I-address
that position `v` currently denotes. REARRANGE carries this *value* intact while
permuting the *key* under which it is filed.

We confine the operation to the text subspace `s_C` of one document, at the
working V-position depth 2 (`#v = 2`). This is the precise scope at which
ASN-0084's closed-form rearrangement permutations are established: REARRANGE_K is
*defined* only for `S = 1` — its CutSequence condition CS3 fixes every cut in the
text subspace, and its postconditions are written against `V_S(d)` with `S = s_C`.
The link subspace is left wholly in the frame: the operation neither names nor
rewrites any link-subspace V-position, so a position-permuting transposition never
has to honour the placement disciplines that govern where a document's links sit.
We make no claim about other subspaces or other depths. We adopt
ASN-0058's
ordinal-shift convention: for a V-position `v` and natural `k`, `v + k`
abbreviates `shift(v, k)` (ASN-0034) at `v`'s depth, with `v + 0 = v`; at depth 2
a text position has the form `[s_C, k]` and `ord(v) = k`. Because the active text
positions are contiguous and densely indexed (D-SEQ★), a *cut* may be named by the
V-position at which it falls, and the width of an interval between two cuts is the
ordinal difference of their positions.

## Cuts and regions

A *cut sequence* is a strictly ascending list of V-positions
`c₀ < c₁ < ... < c_{n-1}` in the text subspace `s_C` at depth 2, with
`n ∈ {3, 4}` and every cut landing on a boundary of the current arrangement
(ASN-0084, CutSequence — its conditions CS3/CS4 fix exactly this subspace and
depth). Three cuts specify a *pivot*; four cuts specify a *swap*. We require that
the affected interval lie entirely within the arrangement — every depth-2 text
position from `c₀` up to the last cut is active (ASN-0084, R-PRE) — so the cuts
genuinely partition existing content rather than naming holes.

For three cuts the affected interval `[c₀, c₂)` splits into two regions
(ASN-0084, RegionPartition)

      α = { v : c₀ ≤ v < c₁ },    β = { v : c₁ ≤ v < c₂ },

with widths `w_α = ord(c₁) − ord(c₀)` and `w_β = ord(c₂) − ord(c₁)`. For four
cuts the interval `[c₀, c₃)` splits into three,

      α = [c₀, c₁),    μ = [c₁, c₂),    β = [c₂, c₃),

where `μ` is the *intervening region* belonging to neither moved block. We write
`w_μ = ord(c₂) − ord(c₁)`. These regions and their ordinal-difference widths are
ASN-0084's RegionPartition (its widths read off the R-PRE consequences); both
moved-block widths are strictly positive — `w_α ≥ 1` and `w_β ≥ 1`, with
`w_μ ≥ 1` as well in the four-cut case — a consequence of CS2's strict ascent,
not a separate condition to check.

The cuts are coordinates in a single arrangement `M(d)`, so the geometry of the
regions is fixed before any reassignment occurs.

## The transposition as a permutation

REARRANGE is the operation **REARRANGE_K** of ASN-0084, applied to the text
subspace at depth 2. We do not redefine it; we import its specification and erect
the system-level guarantees on top. For a 3- or 4-cut sequence `K` satisfying the
preconditions R-PRE, `REARRANGE_K(Σ, d)` produces the post-state `M'(d)` fixed by
ASN-0084's **PivotPostcondition** (`n = 3`) or **SwapPostcondition** (`n = 4`),
together with the frame conditions R-FRAME-P / R-FRAME-S. The pivot freezes the
exterior, slides `β` to the front of the interval, and lets `α` follow:

      v < c₀ ∨ v ≥ c₂  ⟹  M'(d)(v) = M(d)(v),                  (ASN-0084 R-EXT)
      M'(d)(c₀ + j)       = M(d)(c₁ + j),   0 ≤ j < w_β,        (ASN-0084 R-P1)
      M'(d)(c₀ + w_β + j) = M(d)(c₀ + j),   0 ≤ j < w_α.        (ASN-0084 R-P2)

The swap (four cuts) is the same shape with the middle region threaded between:

      v < c₀ ∨ v ≥ c₃  ⟹  M'(d)(v) = M(d)(v),                  (ASN-0084 R-EXT)
      M'(d)(c₀ + j)             = M(d)(c₂ + j),  0 ≤ j < w_β,   (ASN-0084 R-S1)
      M'(d)(c₀ + w_β + j)       = M(d)(c₁ + j),  0 ≤ j < w_μ,   (ASN-0084 R-S2)
      M'(d)(c₀ + w_β + w_μ + j) = M(d)(c₀ + j),  0 ≤ j < w_α.   (ASN-0084 R-S3)

ASN-0084 proves these define a total function. The permutation we carry through
the rest of the note is the *cut-point-induced bijection* of ASN-0084's
**R-PPERM** (pivot) and **R-SPERM** (swap) — the specific map determined by the
cut sequence `K` and the region partition, given in closed form and proved
bijective by those lemmas —

      π : dom(M(d)) → dom(M(d)),   satisfying   M'(d)(π(v)) = M(d)(v),

fixing the exterior and permuting the affected interval. The displayed equation
is a correctness property of this `π`, not its definition: when `M(d)` is not
injective — the value-degenerate instance above — many bijections satisfy it,
the identity among them, and it is the cut-point `π`, not an arbitrary solution
of the equation, that the worked tables below read off.
The well-definedness lemmas **R-PIV** (pivot) and **R-SWP** (swap) establish that
the named destinations constitute a total function on `dom(M(d))` — whence the
domain identity `dom(M'(d)) = dom(M(d))` — and that the region destinations tile
`[ord(c₀), ord(c_{n-1}))` exactly (disjoint and exhausting, R-EXT covering the
complement, so every position is assigned exactly once). We take these results as
given and write

      dom(M'(d)) = dom(M(d))               **(RA2, ASN-0084 R-PIV / R-SWP)**

for the domain-preservation fact we lean on below. This is the formal content of
*transposition*: a reassignment of positions that loses none and invents none.

## What is preserved: I-address correspondence

The bijection equation `M'(d)(π(v)) = M(d)(v)` says exactly what the consultation
calls for. Each rearranged region "must consist of exactly the same content it
held before — the same bytes with the same permanent Istream identity"
(Question 2). The value filed at the moved position is the value that was filed
at the source position; the I-address is copied across the reassignment, never
recomputed. No operation in the specification reads or writes the content store:

      Σ'.C = Σ.C.                                                   **(RA0)**

So content permanence holds in its strongest form — not merely that I-addresses
survive (Question 9), but that the store is a verbatim frame. The relationship
each rearranged region bears to the positions it formerly occupied is therefore
one of *identity correspondence*: position `π(v)` now denotes precisely what
position `v` denoted, and "every byte in a transposed region corresponds to the
same byte before the move" (Question 2). What changes is the V-position; what is
preserved is the I-address, and with it the origin, the attribution, and every
relationship anchored to that address.

A consequence: the *set* of I-addresses the document references is invariant.
Since `π` is a bijection,

      ran(M'(d)) = { M'(d)(π(v)) : v ∈ dom(M(d)) }
                 = { M(d)(v)     : v ∈ dom(M(d)) }
                 = ran(M(d)).                                       **(RA1)**

The middle step is the pointwise equation `M'(d)(π(v)) = M(d)(v)`, established by
ASN-0084's **ArrangementRearrangement** (DEF) and the correctness clauses of
R-PPERM / R-SPERM. The resulting range equality `ran(M'(d)) = ran(M(d))` is
ASN-0084's range invariance **R-RI**. We label the pair **RA1**. The document points
at the same content after the rearrangement as before — only the order of pointing
has changed.

Two foundation invariants ride along on the same structural facts, and we
discharge them explicitly. *Functionality* is preserved — `M'(d)` is
single-valued (ASN-0036, **S2**) — because ASN-0084's R-PIV/R-SWP already
establish the post-state to be a total function, each V-position receiving
exactly one I-address. *Referential integrity* is preserved
in its per-subspace form (ASN-0047, **S3★**) — a content V-position maps into
`dom(C)` and a link V-position into `dom(L)`:

      v ∈ dom(M'(d)) ∧ subspace(v) = s_C  ⟹  M'(d)(v) ∈ dom(C),
      v ∈ dom(M'(d)) ∧ subspace(v) = s_L  ⟹  M'(d)(v) ∈ dom(L).

A document's arrangement carries link-subspace V-positions as well, whose images
are link addresses in `dom(L)`, not content addresses in `dom(C)`; and inside the
affected interval the image filed at a key generally *does* change
(`M'(d)(v) ≠ M(d)(v)`: a pivot's R-P1 branch refiles the front position `c₀` from
`M(d)(c₀)` to `M(d)(c₁)`).
The argument leans on a fact we derive once and reuse: *`π` permutes the text
subspace onto itself*,

      π(V_{s_C}(d)) = V_{s_C}(d).                                  **(RA2a)**

Every position with `subspace(v) ≠ s_C` is fixed pointwise by `π`'s non-S branch
(R-PPERM/R-SPERM). Suppose some `v ∈ V_{s_C}(d)` had `π(v) = w` with
`subspace(w) ≠ s_C`. Then `w ∈ dom(M(d))`, the non-S branch gives
`π(w) = w = π(v)`, and `v ≠ w` (their subspaces differ) — contradicting `π`'s
injectivity (RA2). So `π(V_{s_C}(d)) ⊆ V_{s_C}(d)`; `π` is injective and
`V_{s_C}(d)` is finite (S8-fin), so the restriction is onto, closing the
equality.
Take a text position `v ∈ dom(M'(d))` with `subspace(v) = s_C`. Then
`M'(d)(v) = M(d)(π⁻¹(v))`, and `π⁻¹(v)` is again a text position (RA2a);
pre-state S3★ applied at `π⁻¹(v)` gives
`M(d)(π⁻¹(v)) ∈ dom(C)`, so `M'(d)(v) ∈ dom(C)`. A link position `v` with
`subspace(v) = s_L` is fixed pointwise by the non-text-subspace frame
(ASN-0084, **R-NS** / R-FRAME-P/S(a)), so `M'(d)(v) = M(d)(v) ∈ dom(L)` by
pre-state S3★. Each subspace's inclusion thus carries to the post-state.

The contiguity and tiling invariants ride along on a single observation. Because
`dom(M'(d)) = dom(M(d))` (RA2) and `subspace(·)` is intrinsic to `v`, *every*
active-position set is literally unchanged as a set: the full key set `dom(M(d))`,
and with it both subspace slices
`V_{s_C}(d) = { v ∈ dom(M(d)) : subspace(v) = s_C }` and
`V_{s_L}(d) = { v ∈ dom(M(d)) : subspace(v) = s_L }`. `π` only reassigns the
I-address *value* filed at each `v`, never the set of keys. Every reachable-state
invariant that constrains an active-position set alone is therefore inherited
verbatim from the pre-state, none of them mentioning the values `M(d)(v)` that `π`
reshuffles. Concretely, the per-subspace invariants — contiguity (ASN-0047,
**D-CTG★**), sequentiality (**D-SEQ★**), the minimum position (**D-MIN★**), and
uniform per-subspace depth (**S8-depth**) — each quantify over *every* subspace,
the link subspace `s_L` no less than the text subspace `s_C`; and the
whole-arrangement invariants — V-position well-formedness (**S8a**) and
finiteness (**S8-fin**) — quantify over all of `dom(M(d))`. Each constrains only an
unchanged key set, so each held before the rearrangement and holds after it, for
`V_{s_L}(d)` exactly as for `V_{s_C}(d)`. The link subspace is in fact frozen more
strongly than this set-invariance argument needs: R-NS (ASN-0084) gives
`M'(d)(v) = M(d)(v)` at every `v` with `subspace(v) = s_L`, pinning its values
pointwise as well — the freeze on which the *value*-dependent link-subspace
invariants (S8★, CL-OWN, CL-UNIQ) rest. Subspace exhaustiveness
(**S3★-aux**) — every V-position carries subspace `s_C` or `s_L` — constrains the
full key set `V_{s_C}(d) ∪ V_{s_L}(d) = dom(M(d))`, equally unchanged by RA2, and
is inherited the same way.

One per-state invariant does turn on the values `π` reshuffles, so it cannot be
inherited this way and needs a positive argument: S8★ (ASN-0047, **S8★**,
PerSubspaceSpanDecomposition) constrains the maximal correspondence-run
decomposition of `M(d)`, a function of the I-address *values*, not of the key set
— and REARRANGE is precisely the operation that refragments it, a content subspace
that was one maximal run before a pivot breaking into several after. ASN-0084
already discharges it: **R-BLK** (RunDecompositionTransformation) carries the
pre-state run partition to a disjoint, covering run partition of `M'(d)`, and
**R-CANON** (CanonicalityOfMergeNormalForm) shows that partition's merge-normal
form is the unique maximal-run decomposition S8 guarantees — exactly S8★ on the
content subspace. On the link subspace, S8★'s trivial length-1 decomposition and
the value-dependent CL-OWN/CL-UNIQ ride untouched on the frozen `s_L` frame.

The remaining ASN-0047 obligations are general transition invariants — couplings,
permanence, and hierarchy. ASN-0047 defines its couplings and its
composite-boundary properties P4★/P4a/P7a only *relative to a valid composite* —
the J-couplings between a composite's initial and final states, the
composite-boundary properties at a boundary state — and its `ValidComposite★`
builds composites from a closed atomic vocabulary
`{K.α, K.δ, K.λ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` that does not contain REARRANGE. We
therefore extend that vocabulary with REARRANGE as a new atomic primitive: a single
REARRANGE step is, by fiat, a one-step valid composite, making `Σ → Σ'` a valid
composite transition and its post-state `Σ'` a composite boundary. One structural
fact underlies several of them: because `π` permutes the text subspace onto
itself (RA2a), the content-subspace value set is invariant,

      { M'(d)(v) : subspace(v) = s_C } = { M(d)(u) : subspace(u) = s_C }.

One closure remark first, since J1★ quantifies over every document in `E'_doc`
and `Contains_C` ranges over every document — as do the per-document arguments
already made above (S2, S3★, the set-invariance package, S8★), each stated at
the rearranged `d` alone: for every `d' ≠ d` the cross-document frame
`M'(d') = M(d')` (RA9) leaves `d'`'s arrangement — domain, values, and
content-subspace range — verbatim, so every per-document invariant and both
range-based couplings (J1★, and P4★ through `Contains_C`) are inherited
unchanged at `d'`, and the displayed arguments cover the only document whose
arrangement changes.
The three coupling obligations then hold vacuously, each by its own empty
antecedent. J0 (every freshly allocated I-address is placed in some arrangement) is
vacuous because `dom(C') = dom(C)` by RA0 — REARRANGE allocates no content, so no
I-address is fresh. J1'★ (every new provenance entry answers to a range-new content
address) is vacuous because `R' = R` by RA4 — no provenance entry
is new. J1★ (every content address newly entering the content-subspace range is
recorded in provenance) is the obligation that genuinely reads the arrangement: it
fires for an I-address that lies in `{ M'(d)(v) : subspace(v) = s_C }` but not in
`{ M(d)(u) : subspace(u) = s_C }`. By the content-subspace-range invariance just
displayed those two sets coincide, so no such I-address exists and J1★'s antecedent
is empty as well. Full-range invariance `ran(M'(d)) = ran(M(d))` (RA1) does not by
itself settle J1★ — J1★ is stated against the content-subspace range, not the full
range — so the subspace-preserving action of `π` is what closes it. The three
composite-boundary properties are next. P4★ (`Contains_C(Σ) ⊆ R`) is the one that
reads the mutated arrangement, and it is preserved by the same invariance at `d`
together with the RA9 frame at every `d' ≠ d` (the closure remark above), the
per-document contributions to `Contains_C` being unchanged in both cases:
`Contains_C(Σ') = Contains_C(Σ) ⊆ R = R'`. P7a (every content address carries a
provenance record) is trivial by frame — `dom(C)` and `R` are both frozen (RA0 and
RA4), so its quantifier and its witnessing set are pointwise unchanged.
P4a (TraceWitnessing) is the one that needs more than a frame: it quantifies over
valid *traces* to the state, not over the state alone, so `R' = R` (RA4) gives that
the *set* of provenance entries is unchanged but does not by itself supply each
entry's witness. ASN-0047 already establishes P4a by induction on the number of
valid composites in a trace, its step witnessing an entry new across the final
composite through the coupling J1'★ and a pre-existing entry through the inductive
hypothesis on the prefix. That induction is generic in the final composite, so
admitting REARRANGE to the vocabulary `{ASN-0047's composites} ∪ {REARRANGE}` leaves
it intact; only the new final-composite case needs checking. The final REARRANGE
step `Σ → Σ'`, declared a one-step valid composite, has `Σ'.R = Σ.R` (RA4): its
new-entry branch `Σ'.R \ Σ.R` is empty, so every `(a, d) ∈ Σ'.R` is pre-existing and
is witnessed through the inductive hypothesis on the prefix unchanged. The induction
therefore extends to the augmented vocabulary without modification, and every
reachable composite boundary — `Σ'` among them — satisfies P4a. The genuinely per-state
ExtendedReachableStateInvariants conjuncts that remain fall under a single closure
rule: every conjunct keyed only on frame-frozen components — `dom(C)` and its
values by RA0, `E` and `R` by RA4, `dom(L)` and its values by RA6 — is preserved by
those frames. This covers S4, S7a, S7b (all `dom(C)` properties, frozen verbatim by
RA0), S7d (a document-tumbler property, frozen with `E` by RA4), the C-family
(C1b, C1c, C-fin), the E-family (NodeLineage, ActivatedEmission), the L-family (L0,
L1, L1a, L1b, L1c, L3, L14, L-fin), and P6, P7, P8 — the only conjuncts not so keyed
being the value-dependent CL-OWN and CL-UNIQ.
ASN-0047's second transition theorem,
**ExtendedTransitionInvariants** (its sole conjunct **P3**,
ArrangementMutabilityOnly), holds with every conjunct at equality: `dom(C) =
dom(C')` with `C'(a) = C(a)` by RA0, `dom(L) = dom(L')` with `L'(ℓ) = L(ℓ)` by the
frozen link store (`Σ'.L = Σ.L`), and `E = E'`, `R = R'` by the `E`/`R` frame
RA4 — so the only component P3 permits to lose information, `M`, is the only one
REARRANGE rewrites. Two vocabulary-quantified obligations live outside the two
theorems and bind REARRANGE the moment it enters the vocabulary; each is
discharged at equality. M1 (ArrangementMonotonicity, ASN-0047) demands
`dom(M) ⊆ dom(M')` over the document set: REARRANGE rewrites `M(d)` at the
existing key `d` — the post-arrangement is total on the same key set (RA2), so
`d ∈ dom(M')` — and frames every other document (RA9), giving
`dom(M') = dom(M)`. At the foundation layer, NoDeallocation's closed-Σ frame
(ASN-0034; T8 is its monotonicity consequence) demands
`allocated(Σ) ⊆ allocated(Σ')` of every operation admitted to the transition
vocabulary: REARRANGE performs no allocation event and touches no allocator
state, so `allocated(Σ') = allocated(Σ)`. With ASN-0047's two invariant theorems
discharged and the vocabulary-quantified M1 and allocated-set obligations
discharged alongside them, the invariant package REARRANGE joins is accounted
for.

The deferred `K.μ~` coincidence is now discharged clause by clause against
K.μ~'s admissibility (i)–(v) (ASN-0047). The post-state shape package (i) —
S8a, S8-depth, D-CTG★, D-MIN★ — holds by the set-invariance argument above,
each invariant inherited on the unchanged key sets (RA2). Non-triviality (ii)
is the hypothesis of the coincidence claim itself. Length preservation (iii) —
`#π(v) = #v` — holds by the depth-2 closed forms of R-PPERM/R-SPERM: every text
destination is again a depth-2 position, and every non-text position is fixed.
Subspace preservation (iv) holds by RA2a on the text subspace and by pointwise
fixity elsewhere. Link-subspace fixity (v) — `π(v) = v` at every link-subspace
position, a key-level fact about `π` rather than about the values it files — is
supplied by the non-S branch of R-PPERM/R-SPERM, which fixes every position
with `subspace(v) ≠ s_C` pointwise. A non-trivial REARRANGE step is therefore
an admissible `K.μ~`.

## The intervening content

The four-cut case carries a region `μ` that is part of neither moved block, yet
cannot stay where it sits. The design must guarantee that this content is
"preserved in identity and connectivity even though its virtual position changes"
(Question 3). Our equations discharge this precisely. R-S2 reassigns each
position of `μ` while preserving its denotation:
`M'(d)(c₀ + w_β + k) = M(d)(c₁ + k)` for `0 ≤ k < w_μ`. The middle region is
moved as a block — its internal order is untouched — and its I-addresses are
carried intact, so it satisfies RA0 and RA1 exactly as the moved blocks do.

What is the *net* displacement of the middle? It departs ordinal `ord(c₁)` and
arrives at ordinal `ord(c₀) + w_β`. The displacement is

      (ord(c₀) + w_β) − ord(c₁) = w_β − w_α = (ord(c₃) − ord(c₂)) − (ord(c₁) − ord(c₀)),

the difference in the widths of the two swapped regions. Gregory's implementation
computes exactly this quantity for the middle slice — "the middle region
receives `diff[2] = |right_region| − |left_region|`, the difference in sizes of
the two swapped regions" (Question 11) — and the sign tells the direction: if the
block arriving at the front is wider than the block leaving it, the middle slides
forward; if narrower, backward; if the two are equal, the middle does not move at
all. This is not an arbitrary offset. It is the unique displacement that keeps the
middle *contiguous* with its new neighbours: for `μ` to tile the gap left between
the relocated `β` and the relocated `α`, it must shift by precisely the size
imbalance. The intervening content is thus conserved in identity and order, and
its position is determined — not chosen — by the geometry of the two regions
around it.

## V-extent conservation

The document's total extent must be unchanged: "the same bytes are present, in
the same quantity, merely permuted into a different order" (Question 7).
Conservation is immediate from RA2: since `dom(M'(d)) = dom(M(d))`, the active
text run is literally unchanged as a set, so its cardinality and its endpoints are
invariant:

      | dom(M'(d)) | = | dom(M(d)) |,    min and max V-position fixed.   **(RA3)**

REARRANGE is, in the language of conservation laws, a permutation of a fixed
multiset. Gregory's evidence confirms that the extent is conserved structurally
rather than by repair: the implementation rewrites only V-displacements and never
allocates, frees, or resizes a span, and the recomputation of the document's
width after the operation "should do nothing" — the author's own comment marks it
as a defensive no-op precisely because the extent cannot have changed
(Questions 7, 13). The boundaries of the affected interval are themselves fixed:
the regions tile `[c₀, c_{n-1})` before and after, so the exterior never moves and
the document neither grows nor shrinks.

## A worked transposition

We fix a concrete instance and check the postconditions against explicit
ordinals. Take a document `d` whose text
subspace holds five bytes "ABCDE" at the contiguous depth-2 positions
`[s_C, 1], …, [s_C, 5]`; write `a_k = M(d)([s_C, k])` for the I-address of the
k-th byte, so `ord([s_C, k]) = k`. We stipulate that each byte was committed by
its own allocation event, so the I-addresses — `a₁, …, a₅` here, `a₁, …, a₆` in
the swap below — are pairwise distinct (S4, OriginBasedIdentity, ASN-0036, riding
on GlobalUniqueness, ASN-0034); the footprint computations in this section and
the RA8b inequalities in the atomicity section lean on that distinctness, which
a shared-content pre-state (S5) would not supply.

*Pivot.* Transpose the single-byte region `α = {B}` with the three-byte region
`β = {C, D, E}`: cuts `c₀ = [s_C, 2]`, `c₁ = [s_C, 3]`, `c₂ = [s_C, 6]`, giving
`w_α = ord(c₁) − ord(c₀) = 1` and `w_β = ord(c₂) − ord(c₁) = 3`. R-P1 fills the
front of the interval with `β` — `M'([s_C,2]) = a₃`, `M'([s_C,3]) = a₄`,
`M'([s_C,4]) = a₅`; R-P2 places `α` behind it — `M'([s_C,5]) = a₂`; R-EXT keeps
`M'([s_C,1]) = a₁`. The new reading order is

      A C D E B.

The induced permutation `π` reads off these destination equations:

      π:  ord 1 ↦ ord 1,   ord 2 ↦ ord 5,   ord 3 ↦ ord 2,
          ord 4 ↦ ord 3,   ord 5 ↦ ord 4.

The postconditions check out numerically. The destination ordinals are `{2,3,4}`
(R-P1), `{5}` (R-P2), and `{1}` (R-EXT): pairwise disjoint and tiling `{1..5}`
exactly, so `π` is a bijection and `dom(M'(d)) = dom(M(d))` (**RA2**). The range is
`{a₃, a₄, a₅, a₂, a₁} = {a₁, …, a₅} = ran(M(d))` (**RA1**). The count is 5 and the
endpoints `ord 1`, `ord 5` are fixed (**RA3**). Now a sample link footprint: let
`a*` be a link whose coverage holds the "C" byte `a₃`. Before the move its
footprint is `{[s_C,3]}`; since `[s_C,3] = c₁ + 0`, the pivot branch of `π` gives
`π([s_C,3]) = c₀ + 0 = [s_C,2]`, and indeed `M'([s_C,2]) = a₃`. The footprint
travels through `π` to `{[s_C,2]}` — relocated, not lost.

*Swap.* Take "ABCDEF" at `[s_C,1..6]` and exchange `α = {B}` with `β = {E, F}`,
leaving the middle `μ = {C, D}` between them: cuts `c₀=[s_C,2]`, `c₁=[s_C,3]`,
`c₂=[s_C,5]`, `c₃=[s_C,7]`, so `w_α = 1`, `w_μ = 2`, `w_β = 2`. R-S1 brings `β`
to the front (`M'([s_C,2]) = a₅`, `M'([s_C,3]) = a₆`); R-S2 reseats `μ`
(`M'([s_C,4]) = a₃`, `M'([s_C,5]) = a₄`); R-S3 sends `α` to the back
(`M'([s_C,6]) = a₂`); R-EXT keeps `a₁`. The reading order is

      A E F C D B.

The middle departs `ord(c₁) = 3` and arrives `ord(c₀) + w_β = 4`, a net
displacement of `+1`, which is exactly `w_β − w_α = 2 − 1` — Gregory's `diff[2]`
(Question 11). Because `w_β > w_α`, the middle slides *forward* by precisely the
width imbalance, the unique shift that keeps `μ` contiguous between the relocated
`β` and `α`. Once more the destination ordinals `{2,3}`, `{4,5}`, `{6}`, `{1}`
tile `{1..6}` (**RA2**), the range `{a₁, …, a₆}` is unchanged (**RA1**), and the
extent is conserved (**RA3**).

## Links

A link's endsets reference content by *address*, and the link store is frozen —
RA6, `Σ'.L = Σ.L`, domain and value both. Nothing about a link changes when
content is rearranged; the operation neither reads nor writes `L`. This is the
whole secret of link survival, and it specializes cleanly to the cases the
consultation poses.

*A link anchored entirely within a moved region* (Question 4). Its endset
references I-addresses, all of which belong to the moved block. REARRANGE deletes
no content, so every referenced byte survives; the link "is not an editing
casualty" and "moves with its content" because its endsets are I-addresses, which
the operation never changes (Question 4). The link itself does nothing — it
continues to denote the same I-addresses — and those addresses now happen to be
arranged at new V-positions.

To make "moves with its content" precise we use ASN-0098's projection of a link
into a document, with one piece of local shorthand. ASN-0098's *Definition —
Coverage* assigns each endset `e` its referenced address set `coverage(e)`, a
purely combinatorial function of the endset that consults no state component. We
abbreviate

      coverage(a, i) := coverage(Σ.L(a).eᵢ),

suppressing the state argument; the suppression is harmless across the REARRANGE
transition exactly because RA6 freezes the link store:
`Σ'.L = Σ.L ⟹ Σ'.L(a).eᵢ = Σ.L(a).eᵢ ⟹
coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`, so `coverage(a, i)` is one fixed
address set across the transition. The link's footprint in `d` is ASN-0098's
projection (Definition — Project),

      project(a, i, d, Σ) = { v ∈ dom(M(d)) : M(d)(v) ∈ coverage(a, i) }.

The footprint then transports through `π` by
the bijection equation alone. For any `v ∈ dom(M(d))`,

      v ∈ project(a, i, d, Σ)
        ⟺ M(d)(v) ∈ coverage(a, i)            (definition of project)
        ⟺ M'(d)(π(v)) ∈ coverage(a, i)        (RA1: M'(d)(π(v)) = M(d)(v))
        ⟺ π(v) ∈ project(a, i, d, Σ'),        (definition of project; π(v) ∈ dom(M'(d)) by RA2)

and since `π` is a bijection of `dom(M(d))` (RA2), this is exactly

      project(a, i, d, Σ') = π( project(a, i, d, Σ) ).             **(RA7a)**

The footprint is carried *through* `π`: neither lost nor enlarged, only relocated
to where the content now sits.

*A link spanning both moved regions, or running from a moved region into
stationary content* (Question 5). Here the footprint may be split by a cut, and we
must say *precisely* what the rearrangement does to its contiguity. The positive
fact we can lean on is ASN-0084's **R-COMM** (PermutationShiftCommutativity):
within each region `π` commutes with ordinal shift, `π(v + k) = π(v) + k`, so it
acts there as a *constant displacement* — a rigid translation carrying the whole
region by one fixed net offset. Two quantities must be kept distinct here. The
within-region offset `k ≥ 0` in `π(v + k) = π(v) + k` is a forward ordinal shift
(ASN-0034 defines `shift` only for a positive amount, with `k = 0` the identity),
indexing positions within the region. The region's *net translation*
`ord(π(v₀)) − ord(v₀)` is the signed quantity that may be forward, backward, or
zero. R-COMM fixes the
rigid-translation structure independently of that net direction — so the net
translation need not itself be an ordinal shift. Reading the
constants off R-PPERM/R-SPERM: in the pivot every position of `β` moves by `−w_α`
(R-P1), every position of `α` by `+w_β` (R-P2), and the exterior by `0` (R-EXT);
in the swap the four constant displacements are `−(w_α+w_μ)`, `w_β−w_α`,
`w_β+w_μ`, and `0` for `β`, `μ`, `α`, and the exterior respectively. A constant
displacement is an order- and adjacency-preserving bijection on the region it acts
on.
Hence a footprint *confined to a single region* has its entire run structure
carried intact: the number of contiguous spans it comprises, and the gaps
between them, are exactly the same before and after. In particular a footprint that is a single contiguous run
inside one region remains a single contiguous run. We record this as a *sufficient*
condition for contiguity-preservation — not as a weakest precondition:

      project(a, i, d, Σ) ⊆ one region
        (the `s_C` exterior, α, μ, β, or the frozen link subspace `s_L`)
        ⟹  π preserves the footprint's run structure
            (in particular, a single run stays a single run).        **(RA7c)**

Every other postcondition of this note holds wherever the operation is defined
(`wp = R-PRE`); the footprint's contiguity is the single property REARRANGE does
not preserve in general, and so the one that needs a precondition *beyond* R-PRE.
Confinement matters because REARRANGE does not merely shift each region — it
*relocates the region blocks*, laying `β` before `α` (pivot) or `β, μ, α` (swap),
and so manufactures new *seams* where two formerly separated blocks now abut. Run
structure is preserved *within* a region; *across* regions a seam can heal
contiguity — two relocated blocks re-abut — or break it — a block pulls away from
what sat beside it. The configurations below exercise both sides of the boundary,
each drawn from the worked pivot above (`A B C D E ↦ A C D E B`, cuts
`c₀,c₁,c₂ = ord 2,3,6`, with the `π` table from that section).

*Within a region: run structure preserved (RA7c).* A footprint covering
`{C, E} = {a₃, a₅}` lies wholly inside `β`, with the *discontiguous* pre-footprint
`{ord 3, ord 5}` — a gap at `D`. The pivot carries `β` rigidly, `ord 3 ↦ ord 2`
and `ord 5 ↦ ord 4`, giving `{ord 2, ord 4}`: two singletons with the gap intact
at `ord 3`. A single run inside one region would likewise stay a single run. (The
freedom to carry such a gap is real: `coverage` is an arbitrary address set, L4
EndsetGenerality.)

*Across relocated blocks that re-abut: contiguity preserved.* A footprint covering
all of `α ∪ β = {B, C, D, E} = {a₂, a₃, a₄, a₅}` is a single contiguous run
`{ord 2, 3, 4, 5}` straddling the cut `c₁ = ord 3`. The `π` table carries it
`{ord 2, 3, 4, 5} ↦ {ord 5, 2, 3, 4}` — again the single run `{ord 2, 3, 4, 5}`,
because the relocated `β` and relocated `α` re-tile the interval and re-abut. Both
spanned blocks relocate; the exterior is not involved. The footprint straddles a
cut, so `project ⊆ one region` is false, yet contiguity survives.

*Across a fixed exterior and a relocated block: contiguity broken.* This is the
"running from a moved region into stationary content" case (Question 5). A footprint
covering `{A, B} = {a₁, a₂}` — the fixed exterior byte `A` together with the whole
moved region `α = {B}` — has the *contiguous* pre-footprint `{ord 1, ord 2}`
straddling the cut `c₀ = ord 2`. `π` fixes the exterior (`ord 1 ↦ ord 1`, R-EXT)
but sends `α` to the back (`ord 2 ↦ ord 5`, R-P2), giving the *discontiguous*
`{ord 1, ord 5}`. Both blocks are covered *completely* — no partial block is
involved — yet the run fragments, because the stationary exterior and the
relocated `α` separate.

*Across a partially covered relocated block: contiguity broken.* A footprint
covering `{B, C} = {a₂, a₃}` straddles the cut `c₁ = ord 3`, taking all of `α` but
only the first byte of `β`. Its *contiguous* pre-footprint `{ord 2, ord 3}` is sent
by `π` (`ord 2 ↦ ord 5`, `ord 3 ↦ ord 2`) to the *discontiguous* `{ord 2, ord 5}`.
This realizes Nelson's "a link end that was a single contiguous span before the
rearrange may become discontiguous afterward, because the bytes it holds onto have
moved to new virtual positions" (Question 5) and Gregory's directly-observed endset
fragmentation (Question 16). The link still connects precisely the same bytes; only
its picture in the current order has broken into pieces. The operative principle is
Nelson's: the link "must" do nothing except continue holding its bytes; the system
re-expresses the affected endset as a span-set in the new ordering.

*Discoverability under fragmentation.* Because `π` is a bijection, the footprint
is nonempty after exactly when it was nonempty before (immediate from RA7a):

      project(a, i, d, Σ') ≠ ∅   ⟺   project(a, i, d, Σ) ≠ ∅.      **(RA7b)**

RA7b is the per-slot fact; the discoverability conclusion follows from the deeper,
address-keyed view. Discovery answers by *address*: ASN-0098's **LP12**
(DiscoverabilityCharacterisation) reduces discoverability from `d` to
`coverage(a, i) ∩ ran(M(d)) ≠ ∅`, and by RA1 that intersection is invariant — so a
link discoverable from `d` before the rearrangement is discoverable from `d` after
it, surfacing at whatever V-positions the content now occupies (Question 8).
Fragmentation changes how many spans the footprint comprises; it does not change
*whether* the link is found.

## Discoverability of moved content

A user who navigates to a moved region's new position must find the content and
everything attached to it. This is the dual of the footprint transport RA7a. To
look "under the new position" is to evaluate `M'(d)(π(v))`, and that equals
`M(d)(v)` by RA1 — the same I-address the content always had. Position is resolved
to identity, and identity is what every index, link, and attribution is keyed on.
So a navigation to `π(v)` recovers exactly the content that lived at `v`, together
with its origin (RA0 leaves `origin` invariant) and its links (RA7a places their
footprints at `π(v)`).
"A user looking under the new position finds the content *and* every link,
annotation, and attribution it carried before the move — nothing is lost by
relocation" (Question 8). We record the consequence:

      moved content is discoverable under its new V-position,
      and resolves to its original I-address.                      **(RA5)**

## Atomicity: two cuts at once

Why transpose *together* rather than move one region and later the other? The
single-operation form is interpreted against one arrangement, and this exposes
two ordering invariants (Question 6).

First, the document passes from one canonical total order directly to another. A
move-then-move realization manufactures an intermediate arrangement that is itself
a real, addressable, observable document state — "not a neutral scratch step." We
make this concrete on the worked pivot, whose net permutation carries
`A B C D E ↦ A C D E B` (atomic cuts `ord 2,3,6`). Realize the same `π` by two
successive pivots, each itself a legal `REARRANGE_K`:

      Move 1 (cuts ord 2,3,5):  A B C D E  ↦  A C D B E   = Σ_mid
      Move 2 (cuts ord 4,5,6):  A C D B E  ↦  A C D E B   = Σ'

We verify these arithmetically. Move 1 is a pivot of `α₁ = {B}` (ord 2) against
`β₁ = {C, D}` (ord 3,4): R-P1 gives `M_mid([s_C,2]) = a₃`, `M_mid([s_C,3]) = a₄`;
R-P2 gives `M_mid([s_C,4]) = a₂`; the exterior is frozen — order `A C D B E`.
Move 2 is a pivot of `{B}` (now ord 4) against `{E}` (ord 5): it exchanges those
two, yielding `A C D E B`. That the composite reaches the atomic pivot's final
arrangement is an instance of a two-line general fact. First, the composite of
two arrangement rearrangements is itself an arrangement rearrangement whose
bijection is the composition: writing `π₁, π₂` for the two moves' bijections,
`M_mid(d)(π₁(v)) = M(d)(v)` and `M'_comp(d)(π₂(u)) = M_mid(d)(u)` give, at
`u = π₁(v)`, `M'_comp(d)((π₂ ∘ π₁)(v)) = M(d)(v)`, and `π₂ ∘ π₁` is a bijection
of `dom(M(d))`. Second, a rearrangement's post-arrangement is uniquely determined
by its bijection and the pre-state: the bijection equation inverts to
`M'(d)(u) = M(d)(π⁻¹(u))` since `π` is a bijection (RA2). Hence

      π₂ ∘ π₁ = π   ⟹   M'_comp(d) = M'(d),                        **(RA8a)**

both sides being `u ↦ M(d)(π⁻¹(u))` pointwise. In the worked instance the
composed table is exactly the atomic `π` table, so the final arrangements
coincide. But the intermediate `Σ_mid = A C D B E` is a distinct,
observable arrangement realized by *neither* endpoint — concretely
`M_mid([s_C,4]) = a₂`, while `M([s_C,4]) = a₄` and `M'([s_C,4]) = a₅`, so

      M_mid(d) ≠ M(d)  ∧  M_mid(d) ≠ M'(d).                       **(RA8b)**

A RETRIEVE issued between the two moves returns the order `A C D B E`, which has
no counterpart during the atomic transposition (Question 19). The existence of an
observable divergent intermediate is thus exhibited, not merely asserted: the
logical content of the final state is path-independent; the *visible history* is
not.

Second, the cut coordinates resolve against a single, unshifted frame. All of
`c₀, …, c_{n-1}` are coordinates in one `M(d)`, so the regions' boundaries cannot
drift out from under each other mid-operation. A sequential realization would have
to recompute the second move's cuts in a coordinate frame the first move already
perturbed; the atomic form fixes the frame so every cut is valid simultaneously
(Question 6). This is why the equations of the operation may use `c₀, …, c_{n-1}`
as if they were all meaningful at once: they are.

## Document isolation

If the rearranged content is shared with another document by transclusion, that
document's arrangement must be untouched. Every clause of the operation that
mutates state writes only `M(d)`; the frame is explicit — its cross-document and
content clauses are ASN-0084's R-FRAME-P/S(b)/(c), its link clause the lifted RA6:

      (∀ d' ≠ d :: M'(d') = M(d'))   ∧   Σ'.C = Σ.C   ∧   Σ'.L = Σ.L.  **(RA9)**

The isolation is structural, not a courtesy. Sharing in this model is by reference
to the Istream, not by copy: each document is its own V→I mapping over the common,
immutable content. REARRANGE rewrites `d`'s mapping. A document `d'` that
transcludes some of `d`'s content holds an *independent* mapping `M(d')` over the
same I-addresses, and RA0 guarantees those I-addresses are unmoved. Even when
`ran(M(d)) ∩ ran(M(d')) ≠ ∅` — the two documents genuinely share content —
permuting `d`'s arrangement reaches nothing in `d'`'s. "The transposition
reshuffles one document's references; it cannot touch the underlying content or
the independent arrangement of any document that includes it" (Question 10).
Gregory's evidence makes the same point at the level of mechanism: the operation
runs over a single document's arrangement tree, and a second document's tree is
"simply structurally unreachable from a single-document REARRANGE call"
(Question 20).

## Well-definedness, and a caveat on the arithmetic

The post-state is fixed by naming each destination directly, and ASN-0084's
R-PIV (pivot) and R-SWP (swap) establish that those destinations tile the
affected interval exactly, so `π` is a bijection and the result is a legal
arrangement. We elevate this to a requirement on the operation: REARRANGE is
well-defined only when the induced map is a bijection of `dom(M(d))` onto itself
preserving the domain (RA2). An alternative implementation must satisfy this no
matter how it computes positions.

REARRANGE is therefore a *partial* operation: it is defined exactly where its
preconditions R-PRE hold against `M(d)` (ASN-0084 states that REARRANGE_K "is
partial, defined exactly where R-PRE(K) holds"), and on any input outside that
domain it names no post-state. R-PRE's clauses are these: the document's
arrangement exists and its text subspace is non-empty (R-PRE(i)–(ii)); the cuts
form a CS1–CS5 cut sequence — in particular strictly ascending, by CS2
(R-PRE(iii)); and the affected interval `[c₀, c_{n-1})` lies wholly within the
active text subspace (R-PRE(iv)). The degenerate document sizes are instances
of this exclusion. An empty text subspace (`V_{s_C}(d) = ∅`) fails R-PRE(ii)
outright. A single active position cannot furnish an affected interval of the minimum
width — two positions for a pivot (`w_α, w_β ≥ 1`), three for a swap
(`w_α, w_μ, w_β ≥ 1`) — that strict ascent (CS2) together with R-PRE(iv) require. More
generally, any document whose active run is shorter than that minimum interval
cannot satisfy R-PRE(iv) and CS2 simultaneously.

The opposite extreme stays *inside* the domain. When `c₀ = min(V_{s_C}(d)) =
[s_C, 1]` the left exterior `{v : v < c₀}` is empty and R-EXT's `v < c₀` branch is a
vacuous quantifier; when the affected interval covers the whole active run, both
exteriors vanish. In each case the destinations still tile `[ord(c₀), ord(c_{n-1}))`
disjointly and exhaustively, so `π` remains a bijection of `dom(M(d))` (RA2) and the
extent is conserved (RA3). An empty exterior is a degenerate *branch*, not a
degenerate *input*.

We flag, as an observation rather than a claim, that computing destinations by a
*uniform displacement formula* per region — rather than by the tiling above — is
correct only when the two moved regions have equal width. Gregory's analysis shows
that the green implementation displaces region `α` by `ord(c₂) − ord(c₀)`
regardless of the widths, and that when `w_β > w_α` this drives the middle region to overlap the
relocated `α`, producing a V-position collision that violates the bijection
requirement (Question 14); the same unguarded arithmetic can push a text position
across a subspace boundary (Question 17). These are defects relative to the
abstract operation, which is specified by its target arrangement and admits no
such collision. The width imbalance that the middle region's displacement must
absorb (the section on intervening content) is exactly the quantity a uniform
formula mishandles. An implementation conforming to this specification must make
the regions tile, not merely shift each by a local offset.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| REARRANGE_K | Operation imported from ASN-0084: 3-/4-cut transposition in the text subspace at depth 2, specified by PivotPostcondition (R-EXT, R-P1, R-P2) or SwapPostcondition (R-EXT, R-S1, R-S2, R-S3) with frame R-FRAME-P/R-FRAME-S | imported (ASN-0084) |
| RA0 (ContentStoreFrame) | `Σ'.C = Σ.C` — the content store is a verbatim frame; no I-address is created, destroyed, or rebound | imported (ASN-0084 R-FRAME-P/S) |
| RA1 (IdentityCorrespondence) | `M'(d)(π(v)) = M(d)(v)` (ASN-0084 ArrangementRearrangement DEF + R-PPERM / R-SPERM correctness clauses), hence `ran(M'(d)) = ran(M(d))` (ASN-0084 R-RI) — I-addresses carried across the reassignment | imported (ASN-0084) |
| RA2 (Permutation) | The induced `π` (R-PPERM/R-SPERM) is a bijection of `dom(M(d))` onto itself; `dom(M'(d)) = dom(M(d))` | imported (ASN-0084 R-PIV/R-SWP) |
| RA2a (TextSubspaceClosure) | `π(V_{s_C}(d)) = V_{s_C}(d)` — `π` fixes every non-text position pointwise (R-PPERM/R-SPERM non-S branch), so injectivity (RA2) plus finiteness (S8-fin) close the text subspace onto itself | introduced |
| S2 (FunctionalityPreserved) | `M'(d)` is single-valued — the disjoint tiling of destinations (R-PIV/R-SWP) gives each V-position one I-address (ASN-0036 S2) | preserved |
| S3★ (ReferentialIntegrityPreserved) | per-subspace: `subspace(v) = s_C ⟹ M'(d)(v) ∈ dom(C)` and `subspace(v) = s_L ⟹ M'(d)(v) ∈ dom(L)` (ASN-0047 S3★; derivation in the body) | preserved |
| S8★ (SpanDecompositionPreserved) | `M'(d)` admits the unique maximal correspondence-run decomposition S8 guarantees — content subspace by ASN-0084 R-BLK + R-CANON, link subspace by the frozen frame (ASN-0047 S8★) | preserved |
| RA3 (VExtentConservation) | `\|dom(M'(d))\| = \|dom(M(d))\|`, and the active run's endpoints are fixed — the document's total extent is conserved | introduced |
| RA4 (EntityProvenanceFrame) | `Σ'.E = Σ.E ∧ Σ'.R = Σ.R` — the entity set and provenance relation are verbatim frames; REARRANGE writes only `M(d)` and touches neither (the `E`/`R` components of the lifted `K.μ~` frame) | introduced |
| RA5 (Discoverability) | Moved content is discoverable under its new V-position `π(v)` and resolves to its original I-address `M(d)(v)` | introduced |
| RA6 (LinkStoreFrame) | `Σ'.L = Σ.L` — links are untouched; a link anchored in a moved region survives and travels with its content because endsets reference unchanged I-addresses | introduced |
| RA7a (FootprintTransport) | `project(a, i, d, Σ') = π(project(a, i, d, Σ))` — a link's V-footprint is relocated through `π`, neither lost nor enlarged | introduced |
| RA7b (DiscoverabilityPreserved) | `project(a, i, d, Σ') ≠ ∅ ⟺ project(a, i, d, Σ) ≠ ∅` — fragmentation never costs discoverability; corollary of RA7a (`π` a bijection), with discoverability reduced to `coverage ∩ ran ≠ ∅` by ASN-0098 LP12 | introduced |
| RA7c (FootprintRunStructure) | `project(a, i, d, Σ) ⊆ one region ⟹ π preserves the footprint's run structure` — within-region confinement is sufficient (not necessary) for contiguity-preservation | introduced |
| RA8a (FinalStateInvariance) | `π₂ ∘ π₁ = π ⟹ M'_comp(d) = M'(d)` — a two-move composite whose composed bijection equals the atomic `π` reaches the same final arrangement (both are `u ↦ M(d)(π⁻¹(u))`) | introduced |
| RA8b (IntermediateDivergence) | A two-move composite passes through an observable intermediate arrangement (exhibited: `A C D B E` for the worked pivot) realized by neither endpoint of the atomic transposition | introduced |
| RA9 (DocumentIsolation) | `(∀ d' ≠ d :: M'(d') = M(d'))` together with RA0, RA6 — every other document, including transcluders of the rearranged I-addresses, is invariant | introduced |

## Open Questions

What must REARRANGE guarantee about boundary-hood when a cut in one document's arrangement resolves to an I-address that lies interior to a region of another document's independent arrangement of the same transcluded content?

Under what conditions may two rearrangements on the same document's content scope be applied without a serializing authority while leaving the final arrangement independent of their order?

What invariant must relate a content-based discovery index to the arrangement after a rearrangement, given that a link's footprint may fragment into arbitrarily many V-spans while its coverage is unchanged?

What run-structure guarantee must REARRANGE provide for a link footprint that spans three or more regions, given that within-region confinement (RA7c) is sufficient but not necessary for contiguity preservation?

What must the operation guarantee about the recoverability of a prior arrangement from the permanent content store, given that REARRANGE records only the new V→I mapping and the old order is no longer expressed?

What boundary-preservation guard must a refinement to closed-form displacement arithmetic impose, so that computing region destinations by a per-region offset can never carry a permuted position across a subspace boundary — the hazard the abstract tiling avoids by construction but a formula-based layer must discharge explicitly?
