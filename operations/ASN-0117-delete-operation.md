> **ASN-0117 · DELETE Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0084 · Cut-Point Rearrangements](../foundation/ASN-0084-bundle-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0117-delete-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0117: DELETE Operation

*2026-06-08*

## The problem

We are asked what happens when content over a span of a document is deleted.
The word *delete* arrives loaded with a meaning we must immediately discard.
In almost every system one has ever used, to delete is to destroy — to
overwrite, to free, to make the bytes cease to be. Nelson's design inverts
this completely. His annotation for the deleted state reads: "DELETED BYTES
(not currently addressable, awaiting historical backtrack functions, may
remain included in other versions.)" (4/9). Three clauses, and every one of
them asserts that the bytes are *still there*. Deletion in Xanadu removes
content from a document's *arrangement* while the content itself survives,
permanent and untouched, in the store.

The operation Nelson names is terse: "DELETEVSPAN: This removes the given span
from the given document" (4/66). The whole subtlety hides in the word
*removes*. What is removed is not the content but the document's current
*placement* of it — the mapping that said "here, at this position in my
sequence, sits that content." Gregory's evidence makes the layering exact: the
delete path operates entirely on the document's arrangement enfilade and
"leaves the granfilade entirely untouched, such that the I-addresses
underlying the deleted span remain resolvable to their original bytes even
though no POOM currently references them" (Q15). One layer changes; the other
does not — and this time the discipline is even sharper than for insertion,
because deletion writes *nothing* to the permanent store at all.

We work in the address space `T` of tumblers under the lexicographic total
order T1, with the displacement algebra `⊕`, `⊖`, and the ordinal shift
`shift(v, n) = v ⊕ δ(n, #v)` that moves a tumbler's final component while
fixing its prefix (foundation: OrdinalShift, OrdinalDisplacement, ASN-0034).
We take the two-layer state as given: a **content store** `Σ.C : T ⇀ Val`, the
append-only ground truth of what content exists (Nelson's Istream, the
permascroll), and a per-document **arrangement** `Σ.M(d) : T ⇀ T`, the partial
function from V-positions to I-addresses recording how document `d` currently
arranges that content (Nelson's Vstream). A V-position carries a subspace
identifier in its first component, `subspace(v) = v₁`; content lives in the
text subspace `s_C`, links in `s_L`. We write
`V_S(d) = {v ∈ dom(M(d)) : subspace(v) = S}`.

The standing well-formedness facts, inherited from the arrangement model:
every active V-position is zero-free of depth `m ≥ 2` with all components
positive (S8a, ASN-0036); within one subspace of one document the positions
share a common depth (S8-depth); the text subspace is *dense* —
`V_S(d) = {[S, 1, …, 1, k] : 1 ≤ k ≤ N}` for some `N ≥ 0` (D-SEQ), a
contiguous, gap-free run of ordinals from the canonical first position. We
abbreviate the `k`-th slot of this run `q_k = [S, 1, …, 1, k]` of depth `m`,
and carry the depth-2 text case `m = 2` of the foundation displacement work,
so `q_k = [S, k]` and `ord(q_k) = [k]`. The single arithmetic fact that does
all the work below:

> `σ(q_k) = q_{k−c}` for `k ≥ J + c`, where `q_J` is the first deleted slot and
> `c` the deletion width — *left*-shifting the last component by `c` carries
> the `k`-th slot to the `(k−c)`-th, leaving the
> shared prefix `[S, 1, …, 1]` untouched. This is the ordinal subtraction
> `ord(q_k) ⊖ w_ord` of the foundation contraction (ASN-0082), well-defined and
> order-preserving on the surviving suffix.

## What is removed, and what must survive

The consultation is unanimous and emphatic on the central point. When content
over a span is deleted, the operation "removes that content from the document's
**Vstream** (its current arrangement) while the content itself survives
permanently in the **Istream**" (Q1). The bytes "remain in all other documents
where they have been included" (4/11); previous versions still contain them;
links still resolve to them; historical backtrack can reconstruct any prior
arrangement (Q1, Q8). The deletion changes only where — and whether — the
*deleting document* presently places the content. It changes nothing about
whether the content *exists*.

Our state model makes this a statement of one line. The deleted material
occupies a span of V-positions `{q_J, …, q_{J+c−1}}` in `d`, mapping to the
I-addresses

> `A_del = {M(d)(q_k) : J ≤ k < J + c}`.

The unit of deletion is a *span*: it carries at once an arrangement feature (its
V-extent in the Vstream) and an existence fact (the I-addressed bytes it covers),
"there is no choice as to what lies between; this is implicit in the choice of
first and last point" (4/25, Q4) — and DELETE acts on the former while leaving the
latter untouched in `C`.

What "removal" does is delete those `c` mappings from `M(d)` — and *only* that.
The content store is a strict frame condition of the operation: the granfilade
is never consulted, never written, never freed (Q15). This is exactly the
foundation contraction's **ContentStoreFrame**, which fixes `Σ'.C = Σ.C` —
both domain and per-address value (ASN-0082 **D-I**). Every I-address in
`A_del` is still in `dom(C')` with its value intact; the bytes are, in
Nelson's phrase, "not currently addressable" *from `d`'s present view*, never
"not existing." We record the core guarantee, the one any implementation of
non-destructive editing must satisfy.

**P0 (NonDestruction).** *DELETE does not touch the content store:
`dom(C') = dom(C)` and `(A b : b ∈ dom(C) : C'(b) = C(b))`. In particular every
deleted I-address survives: `A_del ⊆ dom(C')` with content preserved.*

Gregory confirms the architecture enforces this structurally: the codebase
maintains *two distinct deletion primitives*, and the document-span delete
calls only the one that operates on the arrangement enfilade, never the one
that would touch the permanent store (Q15). The non-destruction guarantee is
thus a frame condition, not a courtesy — and a load-bearing one: historical
backtrack, transclusion across documents, and link survival all presume the
bytes endure, so an implementation that freed them would break all three at
once while appearing to honour "the span is gone from this document."

## What shifts, and what the shift must preserve

Now the arrangement effect. The deleted span is removed and the surrounding
content must re-close into a single gap-free sequence. Nelson states the rule
for insertion verbatim — "the v-stream addresses of any following characters…
are increased by the length of the inserted text" (4/66) — and deletion is its
exact symmetric inverse: the V-addresses of characters following the deleted
span are *decreased* by the span's length, so the survivors "close the gap"
and the document "stays in canonical order" (Q2, Q9). The governing constraint
is the enfilade requirement that "all changes, once made, left the file
remaining in canonical order" (1/34): the Vstream is dense, with no holes.

Let `S = subspace(p) = s_C`, `p = q_J` the first deleted position, `w` the
deletion width with `w₁ = 0`, `#w = #p = 2`, `Pos(w)`, and write `c = w₂` for
the count of deleted slots — so `w = [0, c]`, whose ordinal projection is
`w_ord = [c]` (OrdinalDisplacementProjection, ASN-0082), identified with the
natural number `c` by the depth-2 singleton convention (ASN-0084). The deleted
block is then `{q_J, …, q_{J+c−1}}`
and `r = p ⊕ w = q_{J+c}` is the first surviving position past the gap. By the
foundation SubspaceConventionAxiom (ASN-0047/ASN-0093) the text subspace
identifier is `s_C = 1`, so `V_S(d) = V_1(d)` and ASN-0082's contraction —
stated literally for `S = 1` on `V_1(d)` — applies here verbatim, licensing every
D-clause we cite below at `S = s_C`. The three regions of the foundation
contraction (ASN-0082 **ThreeRegions**) partition `V_S(d)` by trichotomy of T1:

- `L = {v ∈ V_S(d) : v < p}` — the prefix, untouched;
- `X = {v ∈ V_S(d) : p ≤ v < r}` — the deleted block, `|X| = c`;
- `R = {v ∈ V_S(d) : v ≥ r}` — the suffix, shifted left.

*Notational convention.* From here on the bare letters `L`, `X`, `R` name these
three regions and nothing else. The extended state's link store and provenance
relation are always written state-qualified — `Σ.L`, `Σ.R`, primed `Σ'.L`,
`Σ'.R` — and K.μ⁻'s retention set, which ASN-0047 states under the letter `R`,
is quoted below under the letter `Ret`.

The displacement is then completely determined: it is the foundation
contraction of ASN-0082. Reading off its clauses:

- **Suffix shifts uniformly left.** For `v = q_k ∈ R` (i.e. `k ≥ J + c`), the
  position moves to `σ(v) = vpos(S, ord(v) ⊖ w_ord) = q_{k−c}`, and *it carries
  its content with it*: `M'(d)(σ(v)) = M(d)(v)` (ASN-0082 **D-SHIFT**). The shift
  is by the same constant `c` for every following position, so the relative
  order of the survivors is preserved exactly (ASN-0082 **D-BJ**: `σ` is an
  order-preserving injection).
- **Prefix is untouched.** For `v ∈ L`: `M'(d)(v) = M(d)(v)` (ASN-0082 **D-L**).
  No position before the cut moves.
- **The gap closes exactly.** The first surviving suffix position lands precisely
  where the deletion began: `ord(r) ⊖ w_ord = ord(p)`, so `σ(q_{J+c}) = q_J`
  (ASN-0082 **D-SEP**). There is no gap and no overlap between `L` and the shifted
  suffix (ASN-0082 **D-DP**, dense partition).

Here is the answer to *what relationship the remaining content bears to its
prior V-positions*. The consultation draws the distinction with care (Q2, Q9):
a V-position never *binds* content; it is an ordinal slot, not a container.
After the deletion, the relation "position `q_k` holds content `X`" has been
rewritten — `X`, if it survived in `R`, now sits at `q_{k−c}`. What is preserved
is the orthogonal relation: *each surviving piece of content keeps its
I-address, and the arrangement re-coordinates itself around that fixed
identity.* The left-shift is a relabelling of slots in the Vstream, not a
transport of bindings; the permanent I-addresses of the survivors do not
change at all.

The displacement is confined to the subspace `S`. Gregory's evidence makes this
structural: a text deletion at `1.x` cannot reach link positions at `2.x`,
because the displacement acts on the deepest ordinal component and a
cross-subspace shift would require subtracting a finer-grained width from a
coarser address — which the arithmetic refuses (Q18). Abstractly this is
already in the foundation: the contraction's shift
`σ(v) = vpos(S, ord(v) ⊖ w_ord)` fixes the subspace component by construction
(ASN-0082 **D-SHIFT**), the same subspace preservation that **OrdShiftHom**
(OrdinalShiftPreservation, ASN-0036) records for the forward shift. Hence the
cross-subspace and cross-document frames hold (ASN-0082 **D-CS**, **D-CD**).

We collect the arrangement effect as a named operation.

**DELETE(`d`, `p`, `w`).**

*Precondition.* `Σ` is a *composite boundary of a valid transition trace* from
the initial state `Σ₀` (P4a's sense, ASN-0047) — a state reached by elementary
transitions drawn from valid composites, so by that hypothesis the per-state
invariant package of ExtendedReachableStateInvariants (ASN-0047) holds at `Σ`;
`d ∈ dom(M)`; `S = subspace(p) = s_C`; `m = #p = 2`, equal to
the common depth S8-depth fixes on `V_S(d)`; `p ∈ V_S(d)` is S8a-well-formed;
`w₁ = 0`, `#w = #p`, `Pos(w)` — so `w = [0, c]` with `c = w₂ ≥ 1`, positivity
falling on the sole non-zero component; and *containment* — the
deleted span lies within the arranged run: `p = q_J` and `r = p ⊕ w = q_{J+c}`
with `1 ≤ J` and `J + c ≤ N + 1` (the case `J + c = N + 1` deletes a suffix,
leaving `R = ∅`). This is exactly the foundation contraction's precondition
(ASN-0082), together with the boundary hypothesis the invariant-preservation
argument below relies on.

*Effect.* DELETE is one arrangement contraction realising ASN-0082's
displacement family, with the content store held in frame. Which ASN-0047
realisation it is splits on whether any suffix survives the cut — on whether
`R = ∅`: when a non-empty suffix survives (`R ≠ ∅`), DELETE is the K.μ⁻ + K.μ⁺
composite; when the deletion reaches the end of the run (`R = ∅`), no survivor
shifts and DELETE is a lone K.μ⁻. The foundation transition
**K.μ⁻ (ArrangementContraction)** is a *prefix-retention truncation*: it keeps a
contiguous prefix of each subspace run *at the survivors' original V-positions* —
its postcondition fixes `M'(d)(v) = M(d)(v)` on the retained domain
`Ret := ∪_S {[S, 1, …, 1, k] : 1 ≤ k ≤ n'_S}` (the set ASN-0047 writes as `R`,
renamed per the convention above). We take the two cases in turn.

*Case `R ≠ ∅` (`J + c ≤ N`): the K.μ⁻ + K.μ⁺ composite.* When survivors remain
past the gap, DELETE is the foundation *composite* of two atomic transitions of
ASN-0047's extended-state model, whose components we access state-qualified as
`Σ.C`, `Σ.L`, `Σ.E`, `Σ.M`, `Σ.R`:

1. a **K.μ⁻** step that contracts the text subspace to its surviving prefix
   `L = {q_1, …, q_{J−1}}` (retention count `n'_{s_C} = J − 1`), while holding
   the link subspace at full retention (`n'_{s_L} = n_{s_L}`); the text count
   `J − 1 < N` supplies K.μ⁻'s required strictly-contracting subspace;
2. a **K.μ⁺** step that re-places the `N − c − (J − 1)` survivors at the
   closed-up text positions `{q_J, …, q_{N−c}}`, each carrying the I-address it
   held before — the former images of `q_{J+c}, …, q_N` (each in `dom(C)`, so
   K.μ⁺'s `a ∈ dom(C)` placement precondition is met) — yielding the dense run
   `{q_1, …, q_{N−c}}` that discharges K.μ⁺'s D-CTG/D-MIN obligations. Because
   `R ≠ ∅`, at least one survivor is re-placed (`N − c − (J − 1) ≥ 1`), so this
   K.μ⁺ adds at least one mapping and meets K.μ⁺'s strict-extension precondition
   (`dom(M'(d)) ⊃ dom(M(d))`, ASN-0047).

*Case `R = ∅` (`J + c = N + 1`): K.μ⁻ alone.* When the deletion reaches the end of
the run there is no suffix to shift: the survivors are exactly the prefix
`L = {q_1, …, q_{J−1}}` *at their original V-positions*, and since `J + c = N + 1`
gives `J − 1 = N − c`, that prefix is already the closed-up dense run
`{q_1, …, q_{N−c}}`. No re-placement is needed, and a K.μ⁺ step would have
`N − c − (J − 1) = 0` survivors to add — an *empty* extension, which K.μ⁺'s
strict-extension precondition (`dom(M'(d)) ⊃ dom(M(d))`) forbids, so the two-step
composite has no realisation here. DELETE is instead a *single* **K.μ⁻** step: a
prefix-retention truncation of the text subspace to count `n'_{s_C} = J − 1 = N − c`
(with `n'_{s_C} < N` since `c ≥ 1`, supplying the strictly-contracting subspace),
the link subspace held at full retention. The delete-everything sub-case
`J = 1, c = N` is this with `n'_{s_C} = 0`.

In both cases the net effect on `M(d)` is exactly ASN-0082's left-shift
displacement (vacuous on the suffix when `R = ∅`), which is why we read DELETE's
clauses off ASN-0082 below.

DELETE's coupling and frame obligations are discharged for both realisations —
trivially via J2 for the `R = ∅` single step, explicitly for the `R ≠ ∅`
composite. The single step is an *elementary* K.μ⁻, self-sufficient by J2
(ContractionIsolation, ASN-0047): it carries no composite-coupling clause and
supplies `Σ'.C = Σ.C ∧ Σ'.L = Σ.L ∧ Σ'.E = Σ.E ∧ Σ'.R = Σ.R` directly, so every
obligation below holds for it outright. For the `R ≠ ∅` composite we discharge
ValidComposite clause 2 (J0, J1★, J1'★
evaluated *only* between the initial state `Σ` and the final state `Σ'`)
explicitly, all vacuously. **J0** (every freshly allocated I-address appears in some
arrangement) holds because DELETE allocates no content — `dom(C') = dom(C)` (P0)
— so its antecedent `dom(C') ∖ dom(C)` is empty. **J1★** quantifies over *every* document
`d' ∈ E'_doc`: any I-address new to `d'`'s content-subspace range must be
recorded in the provenance relation `Σ'.R`. For `d' ≠ d` the discharge is
immediate — DEL-FDOC gives
`M'(d') = M(d')`, so no address is range-new for any other document. For the
operated `d`, DELETE introduces *no* range-new content. The post-state
content-subspace range has two summands, `M(d)(L) ∪ M(d)(R)`: a post-state
image either is retained on the prefix `L` at its own position (D-L) or is
re-placed from the suffix `R` by the K.μ⁺ step (D-SHIFT), and in both cases it
equals `M(d)(v)` for some content-subspace position `v ∈ dom(M(d))` of the
initial state — so J1★'s "new to the range" trigger, the conjunct
`¬(E v ∈ dom(M(d)) : subspace(v) = s_C ∧ M(d)(v) = a)`, is false for every `a`
in the post-state range. **J1'★** (every new
provenance entry requires range-new content) holds because DELETE adds no
provenance, so its antecedent `Σ'.R ∖ Σ.R` is empty. Both component steps fix the
link store, the entity set, and the provenance relation (`Σ'.L = Σ.L`,
`Σ'.E = Σ.E`, `Σ'.R = Σ.R`: K.μ⁻'s per-subspace-scope frame and the amended
K.μ⁺'s frame each list all three, ASN-0047), so the composite does too; the
`R = ∅` single step has the same three clauses from J2's discharge above. We
name this frame discharge **DEL-CFRAME**. With clause 2 thus discharged, DELETE is a valid composite
appended to the valid trace whose final boundary is the pre-state `Σ` (the
precondition's boundary hypothesis), so the extended trace is itself valid and
the post-state `Σ'` is a composite boundary of it; the three composite-boundary
properties — **P4★** (`Contains_C(Σ') ⊆ Σ'.R`), **P4a** (TraceWitnessing),
**P7a** (provenance coverage) — hold there directly by
**ExtendedReachableStateInvariants** (ASN-0047). We name DELETE's clauses but
derive them by citation:

- (DEL-REMOVE) The arrangement loses exactly `c` V→I correspondences in subspace
  `S`: the surviving domain contracts by precisely the deletion width,
  `|{v ∈ dom(M'(d)) : subspace(v) = S}| = N − c`, and the top `c` position labels
  leave the domain, `(A k : N − c < k ≤ N : q_k ∉ dom(M'(d)))`. The deleted
  I-addresses `A_del` persist in `C` (P0); they are not removed from anything else,
  and may be mapped by other positions of `d` or by other documents. (The
  count-and-label form is what stays robust under within-document sharing (S5/M13):
  a deleted-span label `q_k` with `k ≤ N − c` is reoccupied by the shifted survivor
  (DEL-SHIFT), whose image may coincide with the old one, so per-pair absence can
  fail while the count contraction always holds.)
- (DEL-SHIFT) `(A v : v ∈ R : σ(v) ∈ dom(M'(d)) ∧ M'(d)(σ(v)) = M(d)(v))` —
  verbatim ASN-0082 **D-SHIFT**, with `σ(q_k) = q_{k−c}`.
- (DEL-LEFT) `(A v : v ∈ L : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))` —
  ASN-0082 **D-L**.
- (DEL-DOM) `{v ∈ dom(M'(d)) : subspace(v) = S} = L ∪ {σ(v) : v ∈ R}` —
  ASN-0082 **D-DOM**.
- (P0) `Σ'.C = Σ.C` — ASN-0082 **D-I**, the content-store frame.

*Frame.*
- (DEL-CFRAME) `Σ'.L = Σ.L ∧ Σ'.E = Σ.E ∧ Σ'.R = Σ.R` — discharged for both
  realisations in the *Effect* coupling paragraph above, which also draws the
  boundary-properties conclusion (P4★, P4a, P7a at the post-state). The link
  store is fixed in both domain and per-address value (`dom(Σ'.L) = dom(Σ.L)`
  and `(A a : a ∈ dom(Σ.L) : Σ'.L(a) = Σ.L(a))`); on the fixed entity set, P1
  (EntityPermanence) and P8 (EntityHierarchy) survive DELETE trivially.
- (DEL-FSUB) `(A S' : S' ≠ S : {v ∈ dom(M'(d)) : subspace(v) = S'} =
  {v ∈ dom(M(d)) : subspace(v) = S'}` and `M'(d)` agrees there`)` —
  ASN-0082 **D-CS**. In particular the document's *links* (subspace `s_L`) are
  not moved by a text deletion.
- (DEL-FDOC) `(A d' : d' ≠ d : M'(d') = M(d'))` — ASN-0082 **D-CD**.

## The document remains one coherent sequence

We must check that the result is well-formed — that closing the gap has not
left a hole, overlaid two positions, or broken the density that lets spans
name contiguous regions. The post-contraction domain is exactly ASN-0082's
dense run **D-SEQ-post**,

> `V_S(d') = {q_1, …, q_{N−c}}`,

of length `N' = N − c`. The prefix `L = {q_1, …, q_{J−1}}` abuts the shifted
suffix `{q_J, …, q_{N−c}}` (the images of `q_{J+c}, …, q_N` under
`σ(q_k) = q_{k−c}`) flush — the gap-closure `σ(q_{J+c}) = q_J` (D-SEP) seats the
two with no hole and no overlap.
The post-state `Σ'` is a composite boundary of the extended valid trace, as
discharged in the *Effect* coupling paragraph above, so the per-state invariant
package of **ExtendedReachableStateInvariants** (ASN-0047) holds there.
We name only the conjuncts the deletion actively reshapes, all of them
ASN-0082's post-contraction preservation family: **D-SEQ-post**/**D-MIN-post**
(`min(V_S(d')) = q_1`)/**D-CTG-post** for the dense run, **S8a-post** and
**S8-depth-post** for the positions, **S2-post** for single-valuedness,
**S8-fin-post** for finiteness, and **S3-post** for referential integrity —
read at the two-subspace level as S3★ (GeneralizedReferentialIntegrity,
ASN-0047), which the cited package delivers directly: the *text* V-positions
resolve into `dom(Σ'.C)` and the *link* V-positions into `dom(Σ'.L)`. (The
per-subspace split matters to the statement, not the proof: the whole-range
form `ran(M'(d)) ⊆ dom(Σ'.C)` would be false for any document containing a link,
since its preserved link positions map into `dom(Σ.L)`, which by store
disjointness (SD, ASN-0093) is disjoint from `dom(Σ.C)`.) This is the answer to
*how the survivors
sit within the V-stream after the cut*: reading end to end yields the original
content with exactly the deleted span omitted, the stream around it re-closed
into a single coherent ordinal sequence (Q2).

One per-state conjunct of the package deserves explicit mention, because the
deletion *materially re-cuts* it: **S8★** (PerSubspaceSpanDecomposition,
ASN-0047), the partition of `M(d)|_{V_S}` into maximal V→I correspondence runs.
Closing the gap can fuse or split runs — writing `a_k = M(d)(q_k)` for the
pre-state image of the `k`-th slot, across the closed boundary the
survivors `M'(d)(q_{J−1}) = a_{J−1}` and `M'(d)(q_J) = a_{J+c}` need not advance
in lockstep, so the maximal-run decomposition of the post-state differs from
that of the pre-state. But S8★ is a per-state obligation, pinning no particular
decomposition *across* states, so the pre/post re-cut is no violation; what the
post-state owes is that the maximal-run decomposition of its contracted text
arrangement `V_S(d') = {q_1, …, q_{N−c}}` exists *and is unique within that
state* — uniqueness being S8's condition (c), which S8★ retains on the content
subspace. Both are delivered by S8 (ASN-0036), whose preconditions the
post-state package above supplies conjunct by conjunct: S8-fin by S8-fin-post,
S2 by S2-post, S3 by S3★ restricted to the text subspace, S8a by S8a-post, and
S8-depth by S8-depth-post. Hence S8★ holds in the post-state notwithstanding
the re-cut. The
remaining extended-state per-state conjuncts the deletion leaves undisturbed —
**S3★-aux** (subspace exhaustiveness), **CL-OWN** (link-subspace ownership),
**CL-UNIQ** (link-subspace position uniqueness) — are preserved trivially:
DEL-FSUB carries the entire link subspace through verbatim and the text
positions stay `s_C`-tagged, so every clause quantifying over `s_L` positions
or over subspace tags is untouched.

One subtlety the evidence insists on. The cut at the span's boundaries is
*clean*: because the deletion endpoints `p` and `r` fall on existing position
boundaries, the surviving blocks are split at exact boundaries and no
zero-width or degenerate position is ever produced (Q11, Q12). A boundary that
fell strictly interior to a single addressed unit would require splitting it,
but at the abstract level of whole-unit V-positions the deletion either contains
a position or does not — there is no half-contained slot. Every surviving
position is a full, S8a-well-formed ordinal, and `V_S(d')` is the dense run
above with no fragments. We record the survivor-structure fact.

**P2 (GapClosure).** *The surviving content closes into the dense run
`V_S(d') = {q_1, …, q_{N−c}}` of length `N − c`. The prefix `L` is fixed; the
suffix `R` shifts left uniformly by `c` via the order-preserving injection `σ`,
carrying each survivor's I-address unchanged (`M'(d)(σ(v)) = M(d)(v)`). The
underlying arithmetic identity `ord(r) ⊖ w_ord = ord(p)` holds unconditionally
(ASN-0082 D-SEP(a)); when `R ≠ ∅` it reads positionally as the gap closing
exactly — `σ(q_{J+c}) = q_J`, the first survivor landing where the deletion began
(ASN-0082 D-SEP(b)). In the suffix-delete case `J + c = N + 1`, `R = ∅` and
`q_{J+c} = q_{N+1}` is not an arranged position: there is no gap to close, and the
positional reading is vacuous. Relative order and density are preserved; no hole,
no overlap, no degenerate position.*

## Invariants the operation must preserve

We discharge the invariants the question names. Each is a statement about
keeping the content layer and the arrangement layer from contaminating each
other — and, this time, the content layer is touched not at all, which makes
three of the four nearly immediate.

**Content permanence, and address permanence.** This is P0, and it is the whole
non-destruction guarantee. The store is unchanged in domain and value
(ASN-0082 D-I). Address permanence — that DELETE allocates and frees nothing and
rebinds nothing — is P0 itself: `dom(C') = dom(C)` says the domain neither
shrinks nor grows, and `(A b : b ∈ dom(C) : C'(b) = C(b))` says no address is
rebound.

A remark on well-definedness, in Dijkstra's spirit of establishing that an
argument is in a function's domain before using it. The left-shift
`ord(v) ⊖ w_ord` is defined and yields a *positive* ordinal only when the
surviving positions genuinely lie past the deleted width — which the
containment precondition (`p = q_J`, `r = q_{J+c}`, `J ≥ 1`) guarantees, via
the foundation lemma **OrdinalExceedsDisplacement** (ASN-0082): for every
`v ∈ R`, `ord(v) ⊖ w_ord` is well-defined, positive, and equal to `ord(p)` at
`v = r`. The containment precondition is doing real work here, and Gregory's
evidence shows what its absence buys: the implementation's delete path enforces
only a non-zero-width gate — no bounds check against the arranged extent exists
anywhere in the call path — and a span beginning before the first arranged
position is processed as if valid, shifting in-range positions left by the full
out-of-range width and acknowledging success to the caller before the work is
even attempted (Q4). Outside containment the implementation exhibits not an
alternative semantics but a silent corruption; the abstract operation therefore
carries containment as a *precondition*, and how a caller-facing operation
should totalize DELETE over ill-formed spans — by rejection or by clipping to
the arranged run — is a separate specification obligation we leave open below.

**Cross-document arrangement isolation.** Suppose another document `d'` arranges
some of the same content `d` does — `ran(M(d')) ∩ A_del ≠ ∅`, the transclusion
case. Can deleting from `d` perturb `d'`? It cannot, and the proof is the
conjunction of two facts already in hand. By DEL-FDOC, `M'(d') = M(d')` — `d'`'s
arrangement is a separate object, named and modified by nothing in DELETE's
effect. By P0, the shared I-addresses retain their content — the bytes `d'`
reads are immutable. Therefore `d'` resolves every one of its V-positions to the
same content, in the same order, before and after: "the owner of a document may
delete bytes from the owner's current version, but those bytes remain in all
other documents where they have been included" (4/11, Q3, Q5, Q10). This is the
cross-document frame the evidence confirms structurally — DELETE resolves
exactly one document's arrangement and reaches no other (Q17); the evidence
record's own designation for this frame axiom is *F0* (a name of the
consultation material, not a label of this specification or its foundations),
and its statement here is DEL-FDOC. Sharing is by
reference to immutable identity, so a deletion in one sharer is invisible to the
rest.

**P5 (DocumentIsolation).** *For every `d' ≠ d`: `M'(d') = M(d')`, and every
V-position of `d'` resolves to identical content across the transition, in
whichever store its subspace designates — the two cases below exhausting `d'`'s
positions by S3★-aux (SubspaceExhaustiveness, ASN-0047), which admits no
subspace beyond `s_C` and `s_L`. For content-subspace positions
(`subspace(v') = s_C`): `M'(d')(v') ∈ dom(Σ'.C)` with
`C'(M'(d')(v')) = C(M(d')(v'))` (P0). For link-subspace positions
(`subspace(v') = s_L`): `M'(d')(v') ∈ dom(Σ'.L)` with
`Σ'.L(M'(d')(v')) = Σ.L(M(d')(v'))` (DEL-CFRAME). The arrangement and resolved
content of every other
document — including any that transcludes the deleted I-addresses — are invariant
under DELETE on `d`.*

**Link survival, and discoverability across documents.** A link's endsets
reference I-addresses, not V-positions (4/42, 4/30). DELETE removes no I-address
(P0) and adds, removes, or edits no link, so the link store is held entirely
fixed — `Σ'.L = Σ.L` in both domain and value (DEL-CFRAME). Coverage invariance
is the foundation's: DELETE realises as a one- or two-step transition sequence,
so **LP3★** (MultiStepCoverageInvariance,
ASN-0098) gives `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)` at every slot
`i`. Every link designates exactly the same content after the deletion as before.
The link is anchored to bytes that still exist; the strap stays attached. This is
Nelson's survivability clause (4/43, Q6, Q19).

What deletion *can* change is the link's discoverability *from `d`* — and here
the layering is precise. A link `a` is discoverable from a document iff some
slot's coverage meets that document's arranged I-address range:
`discoverable_from(a, d, Σ) ⟺ (E i : coverage(Σ.L(a).eᵢ) ∩ ran(M(d)) ≠ ∅)`
(foundation **LP12 (DiscoverabilityCharacterisation)**, ASN-0098). DELETE
shrinks `d`'s range — `ran(M'(d)) ⊆ ran(M(d))` — directly from its own clauses,
accounting for *both* subspaces. The surviving domain splits into the text
positions `L ∪ σ(R)` (DEL-DOM) and the link positions `V_{s_L}(d)` carried
through verbatim (DEL-FSUB) — and these two parts are exhaustive: S3★-aux
(SubspaceExhaustiveness, ASN-0047) confines every V-position to subspace `s_C`
or `s_L`, so DEL-FSUB's quantifier over `S' ≠ S` ranges over `s_L` alone and no
third subspace exists to contribute positions. The full post-state range
therefore decomposes — writing `M(d)(Y)` for the image of a position set `Y`
under `M(d)`, while `M(d)|_Y` keeps its standard restriction meaning — as
`ran(M'(d)) = M(d)(L) ∪ M(d)(R) ∪ ran(M(d)|_{V_{s_L}(d)})`. Each
summand lies in `ran(M(d))`: the two text summands because DEL-LEFT and DEL-SHIFT
preserve every surviving position's I-address value (`M'(d)(v) = M(d)(v)` on `L`,
`M'(d)(σ(v)) = M(d)(v)` on the image of `R`), and the link summand because
DEL-FSUB holds the `s_L` positions and their images fixed. DEL-REMOVE drops the
deleted correspondences, contributing nothing new. Hence
`ran(M'(d)) ⊆ ran(M(d))`. If the
deletion removes the *last* V-position of `d` mapping into a link's coverage,
that link becomes *undiscoverable from `d`*: every slot's coverage now misses
`d`'s range — `coverage(Σ'.L(a).eᵢ) ∩ ran(M'(d)) = ∅` for all `i` — so
`discoverable_from(a, d, Σ')` fails by **LP12 (DiscoverabilityCharacterisation)**
(ASN-0098) read at `Σ'`. This undiscoverability is *per-document*; only when `d`
was the link's sole arranging document does it amount to the global orphanhood of
**LP17 (GhostProjection)**, whose hypothesis quantifies over every document's
arrangement. But three things remain true, and they
are exactly Nelson's design intent:

- *The link persists.* `a ∈ dom(Σ'.L)` with `Σ'.L(a) = Σ.L(a)` (L12). The link
  orgl survives in storage; only the V→I bridge through `d`'s arrangement that
  *let `d` find it* has been severed. Following the link directly still resolves
  to the still-existing bytes (Q19).
- *The deleted material stays discoverable from any document that still arranges
  it.* Discoverability from a document `d'` depends only on
  `coverage(eᵢ) ∩ ran(M(d'))` (foundation **LP12 (DiscoverabilityCharacterisation)**,
  ASN-0098), and `d'`'s arrangement is untouched (P5) while the I-addresses persist
  (P0). So if `d'` still maps an address in `A_del`, the link — and the content —
  remain discoverable from `d'` regardless of `d`'s deletion (LP12 applied to `d'`).
  This is the answer to *the discoverability of deleted material from other
  documents that still arrange it* (Q5, Q7): yes, unconditionally.
- *The link is re-discoverable from `d` if the content is re-arranged.* Because
  the I-addresses never left `C`, any later operation that places one of them
  back into `d`'s arrangement makes the link discoverable from `d` again. The
  chain is the foundation's own: across any later `Σ' →* Σ''` the link persists
  in the store (Store Monotonicity★, ASN-0098) with coverage unchanged (LP3★),
  so an arrangement entry `Σ''.M(d)(v) = a*` with `a* ∈ coverage(Σ''.L(a).eᵢ)`
  makes LP12's criterion true at `Σ''`. This is the proof chain of **LP18**
  (Resurrection, ASN-0098) applied per-document — no step of it needs LP18's
  global orphan premise. Deletion is not a one-way door at the content layer.

**P4 (LinkSurvival).** *For every link `a ∈ dom(Σ.L)` and slot `i`, the stored
endset has unchanged coverage: `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`
(LP3★, MultiStepCoverageInvariance, ASN-0098), so no link's designated content
changes, and the link store is untouched
(`Σ'.L = Σ.L`). A link discoverable from `d` before the deletion remains
discoverable from `d` iff some surviving V-position of `d` still maps into its
coverage; otherwise it is undiscoverable from `d` (LP12 read at `Σ'`) yet
persists (L12), remains discoverable from every other document that still
arranges its coverage (LP12), and is re-discoverable from `d` should the content
be re-arranged (Store Monotonicity★ + LP3★ + LP12 at the later state).*

## A weakest precondition: when is discoverability preserved?

P4 leaves one question pointed but unanswered: under what condition does DELETE
*preserve*, rather than possibly shrink, the set of links discoverable from `d`?
The wp below is conditional in the *shrinking* direction, and the place it fails
is exactly the loss of discoverability from `d` that P4 records.

Write `D(d, Σ) = {a ∈ dom(Σ.L) : discoverable_from(a, d, Σ)}`. We seek

> `wp(DELETE, "D(d, Σ') = D(d, Σ)")`.

P4 already established the full-document range decomposition, accounting for
*both* subspaces:
`ran(M'(d)) = M(d)(L) ∪ M(d)(R) ∪ ran(M(d)|_{V_{s_L}(d)}) ⊆ ran(M(d))`
(as P4's derivation records).
We refine the subset to the *exact* loss:

> `ran(M'(d)) = ran(M(d)) \ A_del^{excl}`,

where `A_del^{excl} = A_del \ M(d)(L ∪ R)` is the set of deleted I-addresses
that *no surviving position of `d` also maps* — the addresses `d` loses from its
range entirely.
`A_del` consists of *text* content addresses — each is the image of a
`subspace(v) = s_C` position, so `A_del ⊆ dom(Σ.C)` by S3★
(GeneralizedReferentialIntegrity, ASN-0047), while the unchanged `s_L` images
that P4's decomposition carries through verbatim lie in `dom(Σ.L)` (S3★ again);
store disjointness (SD, ASN-0093: `dom(Σ.C) ∩ dom(Σ.L) = ∅`) then makes `A_del`
disjoint from those images. So removing exactly `A_del^{excl}` from the full
prior range `ran(M(d))` — which already contained those same `s_L` images —
yields precisely the full post-state range — with no remainder, because S3★-aux
leaves no third-subspace images for the decomposition to have missed. The link-subspace term is what makes the refinement
exact rather than merely a text-subspace identity: LP12 evaluates discoverability
against the full `ran(M(d))`, and a link's coverage may reference link-subspace
addresses (L4(c), cross-subspace endsets). (If a deleted I-address is also
arranged elsewhere in `d` — the within-document sharing that S5/M13 of the
arrangement model permit — it does not leave the range.) The link store is fixed
throughout — `dom(Σ'.L) = dom(Σ.L)`
(DEL-CFRAME) — so `D(d, ·)` is computed over the *same* index set before and after,
and the quantification "for every prior link `a ∈ dom(Σ.L)`" below exhausts
`dom(Σ'.L)` as well; were DELETE permitted to add a link, `D(d, Σ')` could acquire
a member with no pre-image and the identity `D(d, Σ') = D(d, Σ)` would fail.
Substituting into LP12, for every prior link `a`,

```
  discoverable_from(a, d, Σ')
    ⟺ (E i : coverage(eᵢ) ∩ (ran(M(d)) \ A_del^{excl}) ≠ ∅).
```

A link drops from `D(d, ·)` precisely when *all* of its witnesses in `d` lay in
`A_del^{excl}` — when the deleted span carried the link's last anchor in `d`.
Therefore

> `wp(DELETE, D(d, Σ') = D(d, Σ)) ≡ DELETE-pre ∧ (A a ∈ dom(Σ.L) :`
> `(E i : coverage(Σ.L(a).eᵢ) ∩ ran(M(d)) ≠ ∅) ⟹ (E i : coverage(Σ.L(a).eᵢ) ∩ (ran(M(d)) \ A_del^{excl}) ≠ ∅))`.

The quantifier structure is essential. Discoverability of a link is *existential*
over its slots — `discoverable_from(a, d, Σ) ⟺ (E i : coverage(eᵢ) ∩ ran(M(d)) ≠ ∅)`
(LP12) — so preservation must be stated per *link*, not per *slot*. A per-slot
universal would wrongly reject a link that loses all witnesses in one slot but
keeps a witness in another: that link is still discoverable (some slot survives),
so `D(d, ·)` retains it, yet a per-slot reading would falsify the implication on
the emptied slot. The per-link existential above is exactly the weakest condition,
matching the "last witness" reading below; a per-slot universal would be merely
sufficient, not necessary.

The derived consequence is exact and informative. Discoverability from `d` is
preserved precisely when the deleted span removed *no link's last witness* — when
every link still has something left at each end *within `d`*. This is Nelson's
survivability qualifier "if anything is left at each end" (4/43) read at the
level of one document's discoverability: the link itself never dies (P4), but a
*document's ability to find it* survives exactly when the deletion spared at
least one of that document's anchors to it.

## A worked deletion

Fix the text subspace `S = s_C` at depth `m = 2`, so `q_k = [s_C, k]` and
`σ(q_k) = q_{k−c}`. Let `d` hold `N = 5` text positions,
`V_S(d) = {q_1, …, q_5}`, with `M(d)(q_k) = a_k` for `k = 1, …, 5`. Each `a_k`
is a permanent I-address in `dom(C)`.

**Delete the span at `p = q_3` of width `c = 2`** (removing `q_3, q_4`). Here
`J = 3`, `w = [0, 2]`, `r = p ⊕ w = q_5`, `R = {q_5}`, `L = {q_1, q_2}`,
`X = {q_3, q_4}`. Containment holds: `J = 3 ≥ 1` and `J + c = 5 ≤ N + 1 = 6`.

*Content frame (P0).* `Σ'.C = Σ.C`. The deleted I-addresses
`A_del = {a_3, a_4}` remain in `dom(C')` with their bytes intact. Nothing is
allocated or freed. ✓ P0.

*Removal (DEL-REMOVE).* The two correspondences `q_3 ↦ a_3` and `q_4 ↦ a_4`
leave the arrangement. The position labels that actually vacate the domain are the
top `c = 2`, namely `q_4, q_5`; the deleted-span label `q_3` stays in `dom(M'(d))`
but is reoccupied by the shifted survivor `q_5 → q_3`, so `M'(d)(q_3) = a_5`
(DEL-SHIFT). The surviving domain is `{q_1, q_2, q_3}` (DEL-DOM). ✓ DEL-REMOVE, DEL-DOM.

*Shift (DEL-SHIFT, DEL-LEFT).* Prefix `q_1, q_2` unchanged (DEL-LEFT). The lone
suffix position shifts left by `c = 2`:

```
  q_5 → q_3   carrying a_5      (σ(q_5) = q_{5−2} = q_3,  M'(d)(q_3) = a_5)
```

The gap closes exactly: `σ(r) = σ(q_5) = q_3 = q_J` (D-SEP). ✓ DEL-SHIFT, D-SEP.

*Domain (DEL-DOM, P2).* Surviving index set `{1, 2}` (prefix) ∪ `{3}` (shifted
suffix) = `{1, 2, 3}`, consecutive and gap-free. So `V_S(d') = {q_1, q_2, q_3}`,
the dense run with `N' = N − c = 3`. ✓ P2, D-SEQ-post.

*Reading end to end* now yields `a_1, a_2, a_5` — the original content with the
third and fourth units omitted and the stream re-closed, exactly Nelson's
canonical-order guarantee (Q2). The bytes `a_3, a_4` are not gone: they sit in
`C`, recoverable by backtrack, still resolved by any link or any other document
that names them.

**A multi-position suffix shift (`|R| ≥ 2`).** This case exercises the *uniform*
left-shift of an entire suffix of two or more positions, all by the same `c`,
with their relative order preserved. Take the same `N = 5` document and **delete
the span at `p = q_2` of width `c = 1`** (removing `q_2`). Here `J = 2`,
`w = [0, 1]`, `r = p ⊕ w = q_3`, `L = {q_1}`, `X = {q_2}`,
`R = {q_3, q_4, q_5}` — three suffix positions. Containment holds: `J = 2 ≥ 1`
and `J + c = 3 ≤ N + 1 = 6`.

*Shift (DEL-SHIFT, DEL-LEFT).* Prefix `q_1` unchanged (DEL-LEFT). All three
suffix positions shift left by `c = 1`, each carrying its I-address:

```
  q_3 → q_2   carrying a_3      (σ(q_3) = q_{3−1} = q_2,  M'(d)(q_2) = a_3)
  q_4 → q_3   carrying a_4      (σ(q_4) = q_{4−1} = q_3,  M'(d)(q_3) = a_4)
  q_5 → q_4   carrying a_5      (σ(q_5) = q_{5−1} = q_4,  M'(d)(q_4) = a_5)
```

Every following position moves by the *same* constant `c = 1`, and the source
order `q_3 < q_4 < q_5` is carried to the image order `q_2 < q_3 < q_4` — the
shift is the order-preserving injection `σ` (D-BJ), demonstrated here across
multiple positions rather than one. The gap closes exactly: `σ(r) = σ(q_3) = q_2
= q_J` (D-SEP).

*Domain (DEL-DOM, P2).* Surviving index set `{1}` (prefix) ∪ `{2, 3, 4}` (shifted
suffix) = `{1, 2, 3, 4}`, consecutive and gap-free. So `V_S(d') = {q_1, …, q_4}`,
the dense run with `N' = N − c = 4`. Reading end to end yields `a_1, a_3, a_4, a_5`
— the second unit omitted, the remaining three re-closed in their original
relative order. ✓ DEL-SHIFT, D-BJ, P2.

**Boundary — leading-span delete (`J = 1`, `R ≠ ∅`).** This case exercises
step-1 K.μ⁻ *emptying* the text subspace and step-2 K.μ⁺ re-adding survivors
*into the emptied subspace*, re-pinning S8-depth from scratch. Take the same
`N = 5` document and **delete the opening span at
`p = q_1` of width `c = 1`** (removing `q_1`). Here `J = 1`, `w = [0, 1]`,
`r = p ⊕ w = q_2`, `L = ∅`, `X = {q_1}`, `R = {q_2, q_3, q_4, q_5}`. Containment
holds: `J = 1 ≥ 1` and `J + c = 2 ≤ N + 1 = 6`. Since `R ≠ ∅`, DELETE is the
K.μ⁻ + K.μ⁺ composite.

*Step 1 (K.μ⁻ to empty).* The text subspace contracts to its surviving prefix
`L = ∅` — retention count `n'_{s_C} = J − 1 = 0` — while the link subspace holds
at full retention. After this step `d`'s text arrangement is *empty*:
`V_{s_C}(d) = ∅`. The strict-contraction obligation is met (`0 < N = 5`).

*Step 2 (K.μ⁺ from empty).* The `N − c − (J − 1) = 4` survivors are re-placed at
the closed-up text positions `{q_1, q_2, q_3, q_4}`, each carrying the I-address
it held before — the former images of `q_2, q_3, q_4, q_5`:

```
  q_1 ↦ a_2     q_2 ↦ a_3     q_3 ↦ a_4     q_4 ↦ a_5
```

Because step 1 emptied the text subspace, this K.μ⁺ inserts into an *empty*
subspace and re-pins S8-depth from scratch at `m = 2` (S8a, S8-depth), seating
the canonical first position `min(V_{s_C}(d')) = q_1` (D-MIN) and the dense run
`{q_1, …, q_4}` (D-CTG). Each placed I-address is in `dom(C)` (P0), so K.μ⁺'s
placement precondition is met, and four mappings are added — the strict-extension
precondition is satisfied. Equivalently `σ(q_k) = q_{k−1}` carries each `v ∈ R`
left by `c = 1` with its content (DEL-SHIFT).

*Net effect.* `V_S(d') = {q_1, …, q_4} = {q_1, …, q_{N−1}}`, the dense run with
`N' = N − c = 4`, every survivor's I-address carried unchanged from its pre-state
slot. The gap closes exactly: `σ(r) = σ(q_2) = q_1 = q_J` (D-SEP). Reading end to
end yields `a_2, a_3, a_4, a_5` — the opening unit omitted, the rest re-closed in
order. The deleted `a_1` persists in `C` (P0). ✓ DEL-SHIFT, D-BJ, D-SEP, P2,
D-SEQ-post.

**Boundary — suffix delete and delete-everything (`R = ∅`).** Both `R = ∅`
deletions are the *single* K.μ⁻ realisation (no K.μ⁺): a prefix-retention
truncation of the text subspace to count `n'_{s_C} = J − 1 = N − c`, the link
subspace held at full retention; no position is shifted (DEL-SHIFT vacuous).
Take first the *suffix delete* `p = q_4`, `c = 2`, so `r = q_6`, `R = ∅`,
retention count `n'_{s_C} = 3`: `q_4, q_5` are removed, and the surviving prefix
`{q_1, q_2, q_3}` stays at its original positions — already the closed-up dense
run — so `V_S(d') = {q_1, q_2, q_3}`, `N' = 3`. *Delete-everything* is the
`n'_{s_C} = 0` specialisation: `p = q_1`, `c = N = 5`, all five mappings removed,
`V_S(d') = ∅`, the empty arrangement denoting the empty partial function. In both,
K.μ⁻'s self-sufficiency (J2) supplies the frames directly — `Σ'.C = Σ.C`
(P0, every `a_k` surviving permanently in `C`) and
`Σ'.L = Σ.L ∧ Σ'.E = Σ.E ∧ Σ'.R = Σ.R` (DEL-CFRAME) — and the per-state
package (including S3★) together with the composite-boundary properties P4★,
P4a, and P7a holds at the post-state by ExtendedReachableStateInvariants
(ASN-0047). The document then arranges all but the
deleted tail (or, for delete-everything, no text at all), yet every former
I-address remains permanent and reconstructible in `C` (Q20). ✓ DEL-DOM,
P2 (with `N' = N − c`, including `N' = 0`), S3★, DEL-CFRAME, P4★, P4a, P7a.

**Within-document sharing.** Modify the scenario so that `a_2 = a_5` — the
images of `q_2` and `q_5` coincide (the `a_k` were never declared distinct, and
S2 gives each position exactly one image, so coinciding images are the only way
`d` can arrange the content `a_5` at *two* positions). Delete `p = q_5`,
`c = 1`. Then
`A_del = {a_5}` but `A_del^{excl} = ∅`, because `a_5` is still mapped by the
surviving `q_2`. A link whose coverage contains `a_5` remains discoverable from
`d` *despite* the deletion — the wp's preservation condition holds because
something was left at that end within `d`. ✓ P4, wp.

**Cross-document transclusion (P5 in the concrete).** Introduce a *second*
document `d'` that transcludes some of `d`'s content. Concretely, let `d'` arrange the deleted I-addresses: with
`V_S(d') = {q_1, q_2}` and `M(d')(q_1) = a_3`, `M(d')(q_2) = a_4` — the very two
addresses `A_del = {a_3, a_4}` that the primary scenario deletes from `d`. Note
that `d'`'s positions are the *same tumblers* as the opening slots of `d`'s run:
V-positions carry no document prefix — by S8a they are zero-free depth-2
tumblers `[s_C, k]`, and D-MIN★/D-SEQ★ (ASN-0047) force every document's
canonical run to begin `[s_C, 1], [s_C, 2], …` — so document scoping lives
entirely in `M(d)` and `M(d')` being distinct partial functions over the shared
position vocabulary, never in the position values themselves. This is
transclusion by reference: `d'` shares `d`'s content identity without copying,
since both arrange the *same* permanent I-addresses (S5/M13 across documents).

Now perform exactly the primary deletion on `d` — `p = q_3`, `c = 2`, removing
`q_3, q_4` from `d`. We check the two facts P5 names against this concrete `d'`:

- *Arrangement untouched (DEL-FDOC).* `d' ≠ d`, so `M'(d') = M(d')` verbatim:
  `q_1 ↦ a_3` and `q_2 ↦ a_4` under `M(d')` are exactly the bindings they were
  before. DELETE
  resolves `d`'s arrangement enfilade alone and names no position of `d'`; the
  left-shift, the gap-closure, the domain contraction all happen inside `d`.
  (Indeed in the post-state the same tumbler `q_1` carries `a_1` under `M'(d)`
  and `a_3` under `M'(d')` — the position is shared vocabulary; the binding is
  per-document.) ✓
  DEL-FDOC, P5.
- *Resolved content unchanged (P0).* `Σ'.C = Σ.C`, so `a_3, a_4 ∈ dom(C')` with
  `C'(a_3) = C(a_3)`, `C'(a_4) = C(a_4)`. Hence `d'` resolves `q_1` to the same
  bytes `C(a_3)` and `q_2` to the same bytes `C(a_4)` after the deletion as
  before. Reading `d'` end to end yields exactly what it yielded before `d`'s
  deletion. ✓ P0, P5.

The deletion in `d` is therefore *invisible* to `d'`: `d'`'s arrangement and the
content it resolves are bit-for-bit identical across the transition, even though
the same I-addresses just vanished from `d`'s present view. This is the formal
content of Nelson's "may remain included in other versions" (4/9, 4/11) — the
deleted bytes "remain in all other documents where they have been included."
Were DELETE to free `a_3, a_4` from the store, `d'` would resolve `q_1, q_2` to
nothing and the transclusion would shatter; non-destruction (P0) is precisely
what keeps the sharer whole. And any link whose coverage contains `a_3` or `a_4`
stays discoverable from `d'` regardless of `d`'s deletion, since
`coverage(eᵢ) ∩ ran(M'(d')) = coverage(eᵢ) ∩ ran(M(d')) ≠ ∅` is untouched
(LP12 applied to `d'`, with `ran(M'(d')) = ran(M(d'))` by P5) — the
discoverability loss the wp computes is strictly *local to `d`*. ✓ P5, P4.

## What we have established

One effect, one layer touched, the other held in perfect frame. On the content
layer DELETE does *nothing*: `Σ'.C = Σ.C`, append-only taken to its limit — not
even an append (P0). The deleted bytes survive at their permanent
I-addresses forever; this is the non-destruction guarantee, and it is a frame
condition, not a courtesy. On the arrangement layer DELETE is a uniform left-
shift confined to one subspace of one document (DEL-REMOVE, DEL-SHIFT, DEL-LEFT,
DEL-DOM, DEL-FSUB, DEL-FDOC), closing the gap exactly and re-coordinating the
suffix around fixed content identities. The well-formedness of the V-stream is
preserved (D-SEQ/D-MIN/D-CTG-post with `N' = N − c`), the survivors re-close into
a single dense run (P2), every link survives because it anchors on immutable
identity (P4), the deleted material stays discoverable from every other document
that still arranges it (P4, P5), and every other document is isolated because
identity is shared by reference, not by arrangement (P5). The bytes endure;
only their placement in this one document's present view is withdrawn.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| DELETE | Operation: remove the span `(p, w)` of width `c` from document `d`'s arrangement; shift the suffix left to close the gap; touch the content store not at all | introduced |
| P0 (NonDestruction) | `Σ'.C = Σ.C` — `dom(C') = dom(C)` with all values preserved; every deleted I-address `A_del` survives in `C` (ASN-0082 D-I) | introduced |
| P2 (GapClosure) | Survivors close into the dense run `{q_1, …, q_{N−c}}`; prefix fixed, suffix shifts left by `c` carrying I-addresses unchanged, gap closes exactly, order and density preserved | introduced |
| P4 (LinkSurvival) | Every endset's coverage is unchanged and the link store untouched, `Σ'.L = Σ.L` (coverage invariance by LP3★, ASN-0098); a link undiscoverable from `d` (LP12 at `Σ'`) still persists (L12), stays discoverable from other documents arranging it (LP12), and is re-discoverable on re-arrangement (Store Monotonicity★ + LP3★ + LP12) | introduced |
| P5 (DocumentIsolation) | Every other document's arrangement and resolved content — including transcluders of the deleted I-addresses — are invariant under DELETE on `d` | introduced |
| DEL-REMOVE | The arrangement loses exactly `c` V→I correspondences (`|{v ∈ dom(M'(d)) : subspace(v) = S}| = N − c`) and the top `c` labels `{q_{N−c+1}, …, q_N}` leave `dom(M'(d))`; the deleted I-addresses persist in `C` | introduced |
| DEL-SHIFT | Suffix positions `v ∈ R` move to `σ(v) = q_{k−c}`, carrying their I-address (ASN-0082 D-SHIFT) | introduced |
| DEL-LEFT | Prefix positions `v < p` are unchanged (ASN-0082 D-L) | introduced |
| DEL-DOM | `V_S(d')` is the dense run `{q_1, …, q_{N−c}}` with the gap closed (ASN-0082 D-DOM, D-SEP) | introduced |
| DEL-CFRAME | `Σ'.L = Σ.L ∧ Σ'.E = Σ.E ∧ Σ'.R = Σ.R` in both realisations (discharged in the *Effect* coupling paragraph); P1/P8 preserved on the fixed entity set | introduced |
| DEL-FSUB | Positions in subspaces `S' ≠ S` (notably links) are unchanged (ASN-0082 D-CS) | introduced |
| DEL-FDOC | Arrangements of all documents `d' ≠ d` are unchanged (ASN-0082 D-CD) | introduced |

## Open Questions

How must a caller-facing deletion totalize the present operation — by rejecting or by clipping a caller-supplied span that falls outside the arranged run — given that the containment precondition places such spans outside DELETE's domain?

Under what conditions may a deletion and a concurrent operation on the same document's content scope both be applied without a serializing authority while preserving canonical order?

What invariant relates a content-based discovery index to the arrangement after a deletion, given that the deleted I-addresses persist while the deleting document no longer arranges them?

What must the system guarantee about the reconstructibility of a prior arrangement from the permanent content store after a deletion, and what state beyond the content store must persist for backtrack to be exact?

What relationship must hold between a deletion that orphans a link from one document and the obligations of the documents that continue to arrange the same content?
