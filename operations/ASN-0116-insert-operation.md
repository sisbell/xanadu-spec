> **ASN-0116 · INSERT Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0084 · Cut-Point Rearrangements](../foundation/ASN-0084-bundle-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0116-insert-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0116: INSERT Operation

*2026-06-08*

## The problem

We are asked what happens when new material is placed into a document. The
question sounds elementary — make room, drop the content in — but each word
of it hides an obligation. *New material* must acquire an identity. *Make
room* must displace what was there without destroying it. *Drop it in* must
leave the document a single coherent sequence, and must do so without
disturbing the links that point into the displaced region, nor the other
documents that may be reading the very same content through shared addresses.

Nelson states the operation in one sentence: an INSERT "inserts `<text set>`
in document `<doc id>` at `<doc vsa>`. The v-stream addresses of any following
characters in the document are increased by the length of the inserted text"
(4/66). Two effects hide in that sentence. Something is *added* — the text set
acquires a home and an identity. Something *shifts* — the following addresses
increase. We will find that these two effects live in two different layers of
the system, and that almost every invariant we must preserve is a statement
about keeping those layers from contaminating each other.

We work in the address space `T` of tumblers under the lexicographic total
order T1, with the displacement algebra `⊕`, `⊖`, and the ordinal shift
`shift(v, n) = v ⊕ δ(n, #v)` that advances a tumbler's final component by `n`
while fixing its prefix (foundation: OrdinalShift, OrdinalDisplacement). That
formula requires `n ≥ 1` — `δ(0, #v)` is the zero tumbler, which TumblerAdd
cannot apply (it requires `Pos(w)`) — so at the boundary we adopt the standard
convention `shift(t, 0) := t`, the identity shift (as in ASN-0036 S8 and
ASN-0058 OrdinalShiftBase), with `shift(p, 0) = p` and `shift(a, 0) = a`. We
take as given the two-layer state: a **content store** `Σ.C : T ⇀ Val`, the
append-only ground truth of what content exists, and a per-document
**arrangement** `Σ.M(d) : T ⇀ T`, the partial function from V-positions to
I-addresses that records how document `d` currently arranges that content. A
V-position carries a subspace identifier in its first component, written
`subspace(v) = v₁`; content lives in the text subspace `s_C`, links in `s_L`
(link placement is a distinct operation drawing on K.λ, not K.α).
We write `V_S(d) = {v ∈ dom(M(d)) : subspace(v) = S}` for the V-positions of
`d` in subspace `S`. We work inside ASN-0047's extended state
`Σ = (C, L, E, M, R)`; beyond `C` and `M` the one further component INSERT
touches is the **provenance relation** `Σ.R ⊆ T_elem × E_doc` (pairs of an
element-level I-address and a document), the record coupling each content
I-address to the document that placed it, where `E_doc = dom(M)` is the set of
allocated documents.

The standing well-formedness facts we will lean on, all inherited from the
arrangement model: every active V-position is zero-free of depth `m ≥ 2` with
all components positive (S8a); within one subspace of one document the
positions share a common depth (S8-depth); the text subspace is *dense* —
`V_S(d) = {[S, 1, …, 1, k] : 1 ≤ k ≤ N}` for some `N ≥ 0` (D-SEQ, with `N = 0`
the empty case), so it occupies a contiguous, gap-free run of ordinals
starting at the canonical first position `[S, 1, …, 1]`. We abbreviate the
`k`-th position of this run as `q_k = [S, 1, …, 1, k]` of depth `m`, where
`m` is pinned by the insertion precondition below. With `m` so pinned, we
observe the single arithmetic fact that does all the work below:

> `shift(q_k, n) = q_{k+n}` — advancing the last component by `n` carries the
> `k`-th slot to the `(k+n)`-th, leaving the shared prefix `[S, 1, …, 1]`
> untouched (by OrdinalShift, since `actionPoint(δ(n, m)) = m = #q_k`).

## What is allocated, and why it must be fresh

Consider first the content being inserted. At the instant new content enters the
document it acquires a *permanent identity* — an I-address — that is strictly distinct
from the document's *arrangement* of it. "The address of a byte in its native
document is of no concern to the user or to the front end; indeed, it may be
constantly changing… but since the links are to the bytes themselves, any
links to those bytes remain stably attached to them" (4/11, 4/30). Identity is
permanent; arrangement is ephemeral. INSERT is the moment identity is minted.

What identity? Gregory's evidence settles the mechanism precisely. The address
is not drawn from a global counter; it is *derived from the current state* of
the document's own content region. The allocator finds the greatest I-address
already present beneath the document's content scope and returns its successor
— `findpreviousisagr` followed by an increment of one (granf2.c:164, 169). Two
consequences follow that an abstract specification must record. First,
allocation is **monotonic**: the new address strictly exceeds every content
address previously allocated under this document. Second, allocation consults
*position*, never *content* — there is "no hash table, no byte comparison, no
deduplication mechanism anywhere in the insert path" (Q17). Inserting the same
bytes twice yields two distinct addresses.

This is exactly the content-allocation transition of the substrate, and we do
not re-derive it: allocating one unit is the foundation operation **K.α
(ContentAllocation, ASN-0093)**, which commits a fresh content address `a`
scoped to `d`, with

> `a ∉ dom(C)`, `origin(a) = d`, `subspace_I(a) = s_C`.

K.α's emission lemmas **FirstEmissionFreshness** and
**SubsequentEmissionFreshness** discharge `a ∉ dom(C) ∪ dom(L)` against the whole
store. The
`findpreviousisagr`-and-increment evidence above is the concrete realisation of
K.α's subsequent-emission branch `a = inc(a_prev, 0)`, where
`a_prev = max{a' ∈ dom(C) : origin(a') = d}`.

For a span of `n` units, INSERT is the `n`-fold composition of K.α along the
single content sub-allocator chain `A_C(d)`: the start `a` is fixed and each
successive address advances by `inc(·, 0) = shift(·, 1)` (each chain element is
T4-valid — **ChainElementT4Validity**, ASN-0093 — and a valid address's
significant position is its last, so `inc(·, 0)` increments the final component —
**TA5-SigValid** and **TA5(c)**, ASN-0034). The allocated run is therefore exactly

> `A_new = {shift(a, k) : 0 ≤ k < n}`,

contiguous on `d`'s content chain and fresh as a whole — `A_new ∩ dom(C) = ∅` —
because each K.α step is fresh against the store as it stands after the previous
step (Q14). The run `A_new` is thus `n` fresh, contiguous, origin-stamped
I-addresses, with the content values written there.

**IP0 (OriginIdentity)** *(restatement of K.α freshness + **S4 (OriginBasedIdentity,
ASN-0036)**: I-addresses from distinct allocation events are distinct regardless of
stored value).* *For each `k` with
`0 ≤ k < n`, `shift(a, k) ∉ dom(C)`, and `shift(a, k)` is distinct from every
I-address in `dom(C)` regardless of whether the freshly written value
`C'(shift(a, k)) = w_k` equals `C(b)` for any existing `b ∈ dom(C)`.*

Identity is intensional (by origin), not extensional (by value). Were two
equal-valued insertions to share an address, a link to one would silently
become a link to the other, and the "strap between bytes" (4/42) would bind the
wrong bytes.

## What shifts, and what the shift must preserve

Now the second effect. The text set is to be placed at a V-position `p`, and
"the v-stream addresses of any following characters… are increased by the
length of the inserted text" (4/66). We must say exactly which positions
follow, by how much, and — the subtle part — what relationship the displaced
positions bear to what they held before.

Let `S = subspace(p)` and let `p = q_J` be a **valid insertion position** in
the foundation's sense (ASN-0036): `ValidFirstInsertionPosition(d, p, m)` when
`V_S(d)` is empty — `p = q_1`, the canonical first position — and
`ValidInsertionPosition(d, p)` when `V_S(d) = {q_1, …, q_N}` — `p = q_J` for
some `1 ≤ J ≤ N+1`, with `J = N+1` the *append* case, where
`p = shift(max(V_S(d)), 1)` is one past the end. This
is the full precondition on the insertion point — depth-`m`, subspace-`S`,
S8a-well-formed, seated at or one-past an existing slot so that no gap can
open — exactly the content of the two foundation predicates' postconditions.

The displacement is then completely determined. Reading off `shift(q_k, n) =
q_{k+n}`:

- **Suffix shifts uniformly.** For `v = q_k ∈ V_S(d)` with `v ≥ p` (i.e.
  `k ≥ J`): the position moves to `shift(v, n) = q_{k+n}`, and *it carries its
  content with it*: `M'(d)(shift(v, n)) = M(d)(v)`. The shift is by the same
  constant `n` for every following position, so their relative order is
  preserved exactly.
- **Prefix is untouched.** For `v = q_k ∈ V_S(d)` with `v < p` (i.e. `k < J`):
  `M'(d)(v) = M(d)(v)`. No position before the cut moves.
- **The vacated slots receive the new content.** The positions
  `q_J, …, q_{J+n-1}` — that is, `{shift(p, k) : 0 ≤ k < n}` — are now free, and
  map in lockstep to the freshly allocated run:
  `M'(d)(shift(p, k)) = shift(a, k)` for `0 ≤ k < n`.

A V-position never *binds* content; it is an ordinal slot, not a container (Q2).
After the insertion,
the relation "position `q_J` holds content `X`" is gone — `q_J` now holds new
content, and `X` has moved to `q_{J+n}`. What is preserved is the orthogonal
relation: *content `X` keeps its I-address, and the arrangement re-coordinates
itself around that fixed identity.* The shift is a relabelling of slots, not a
transport of bindings. The invariant runs in the content layer
(`X ↦ I-address`, immutable), never in the position layer (`slot ↦ X`,
deliberately fluid).

We must also state what the operation leaves alone. The displacement is
confined to subspace `S`. In Gregory's implementation the insertion cut is
bounded above by the *next subspace boundary*, so a text insertion at `1.x`
shifts text positions but never reaches link positions at `2.x` (Q12, Q13). Abstractly this is the foundation's subspace
confinement of the ordinal shift: the subspace identifier sits in the
V-position's first component (`subspace(v) = v₁`, SubspaceProjection,
ASN-0036), and the shift fixes every component but the last —
`subspace(shift(v, n)) = subspace(v)` by **OrdShiftHom (a)
(OrdinalShiftPreservation, ASN-0036)**, extended to `n = 0` by **SUBCONF
(SubspaceConfinement, ASN-0084)** — so it cannot cross into another
subspace's region. Hence:

- **Cross-subspace frame.** For every `S' ≠ S`, the positions of `d` in
  subspace `S'` are unchanged in both domain and value.
- **Cross-document frame.** For every `d' ≠ d`, `M'(d') = M(d')`.

We collect the arrangement effect as a named operation.

**INSERT(`d`, `p`, `w₀ … w_{n-1}`).**

*Precondition.* `d ∈ dom(M) = E_doc` (the document is an allocated entity, so the
provenance step below has a legal home); `Σ` is reachable from `Σ₀` by a valid
transition trace — hence a composite boundary, so the per-state invariants together
with the composite-boundary properties of ExtendedReachableStateInvariants
(ASN-0047) hold at the pre-state; `n ≥ 1`; `(A k : 0 ≤ k < n : w_k ∈ Val)` — each
inserted unit is a well-formed content value; `S = subspace(p) = s_C`;
`m := #p ≥ 2`, and when `V_S(d) ≠ ∅` this `m` equals the common depth that
S8-depth fixes on `V_S(d)`; `p` is S8a-well-formed; and `p` is a valid insertion
position in the foundation sense (ASN-0036). The position predicates are:

- if `V_S(d) = ∅`: `ValidFirstInsertionPosition(d, p, m)` — `p` is the canonical
  first position `[S, 1, …, 1]` of depth `m`, and this first insertion *fixes*
  the subspace depth at `m` for every later insertion;
- if `V_S(d) ≠ ∅`: `ValidInsertionPosition(d, p)` — `p = q_J` for some
  `1 ≤ J ≤ N+1`, with `J = N+1` the *append* case `p = shift(max(V_S(d)), 1)`.

Allocation supplies `a` as the K.α-fresh origin-`d` content I-start (above),
with `A_new ∩ dom(C) = ∅`.

*Effect.* INSERT is the composite of `n` content allocations (K.α, ASN-0093), an
arrangement contraction–extension pair `K.μ⁻` then `K.μ⁺` (degenerating to a single
`K.μ⁺` when no suffix moves) whose net effect realises the post-insertion shift of
ASN-0082's I3 family, and `n` provenance recordings (K.ρ, ASN-0047) that couple each
allocated address to `d`. Two facts the arrangement clauses share are worth
recording once. The **block-disjointness fact**: as ordinals `q`, the three index
intervals `{1, …, J-1}` (left), `{J, …, J+n-1}` (block), and `{J+n, …, N+n}`
(shifted suffix) are consecutive — with no integer gap — and pairwise disjoint, their
union being `{1, …, N+n}` (immediate from `0 < J ≤ N+1`: the right endpoint of each
interval is one below the left endpoint of the next). The **gapped/filled bridge**:
ASN-0082's I3 family fixes its values on the *gapped* arrangement `M'₀(d)`, whose
subspace-`S` domain (by I3-V/I3-CS) is the left prefix `{1, …, J-1}` together with the
shifted suffix `{J+n, …, N+n}`, excluding the inserted block; INSERT's post-state is
`M'(d) = M'₀(d) ∪ {block fill}`, the `{block fill}` mapping the block interval
`{J, …, J+n-1}`. By the block-disjointness fact the block is disjoint from the gapped
domain, so the union adds an entry at every block slot and leaves every gapped value —
left and shifted-suffix alike — unchanged. The block is moreover wholly subspace-`S`:
every filled position `shift(p, k)` has `subspace(shift(p, k)) = subspace(p) = S`
(**OrdShiftHom (a)**, ASN-0036, extended to `k = 0` by **SUBCONF**, ASN-0084), so the
union also leaves the *cross-subspace* slice of the gapped arrangement untouched — in
every subspace `S' ≠ S`, `M'(d)` and `M'₀(d)` have the same positions and the same
values. We name the clauses:

- (I-ALLOC) `dom(C') = dom(C) ∪ A_new`, with `C'(shift(a, k)) = w_k` for
  `0 ≤ k < n` — the K.α effect (ASN-0093), iterated `n` times along `A_C(d)`.
- (I-IMM) `(A b : b ∈ dom(C) : C'(b) = C(b))` — K.α append-only (C0, ASN-0093).
- (I-SHIFT) `(A v : v ∈ V_S(d) ∧ v ≥ p : shift(v, n) ∈ dom(M'(d)) ∧
  M'(d)(shift(v, n)) = M(d)(v))` — ASN-0082 **I3 (PostInsertionShift)** fixes these
  values on the shifted-suffix region `{J+n, …, N+n}` of `M'₀(d)`; the gapped/filled
  bridge carries them to `M'(d)` unchanged.
- (I-LEFT) `(A v : v ∈ V_S(d) ∧ v < p : v ∈ dom(M'(d)) ∧ M'(d)(v) = M(d)(v))` —
  ASN-0082 **I3-L (PostInsertionLeftFrame)** fixes these values on the left region
  `{1, …, J-1}` of `M'₀(d)`; the gapped/filled bridge carries them to `M'(d)`
  unchanged.
- (I-NEW) `(A k : 0 ≤ k < n : shift(p, k) ∈ dom(M'(d)) ∧
  M'(d)(shift(p, k)) = shift(a, k))` — the `{block fill}` of the gapped/filled bridge:
  the block positions `shift(p, k) = q_{J+k}` (`0 ≤ k < n`), left free by the gapped
  arrangement, mapped in lockstep to the K.α run `A_new`.
- (I-DOM) `{v ∈ dom(M'(d)) : subspace(v) = S} =
  {q_1, …, q_{J-1}} ∪ {q_J, …, q_{J+n-1}} ∪ {q_{J+n}, …, q_{N+n}}` — the gapped/filled
  bridge at the domain level: I3-CS (PostInsertionSubspaceClosure) supplies the gapped
  domain (left prefix `{q_1, …, q_{J-1}}` and shifted suffix `{q_{J+n}, …, q_{N+n}}`),
  and INSERT's own I-NEW `{block fill}` supplies the middle block `{q_J, …, q_{J+n-1}}`.
- (I-PROV) `R' = R ∪ {(shift(a, k), d) : 0 ≤ k < n}` — the `n` provenance records
  coupling each freshly allocated I-address to its inserting document, by **K.ρ
  (ProvenanceRecording, ASN-0047)** iterated `n` times. The
  record is `(shift(a, k), d)` with `shift(a, k)` element-level content (S7b/C1) and
  `d` document-level, matching `Σ.R ⊆ T_elem × E_doc`. These are the only additions
  to `R`; INSERT removes nothing from it (P2 of ASN-0047, R monotone).

*Frame.*
- (F-SUB) `(A S' : S' ≠ S : {v ∈ dom(M'(d)) : subspace(v) = S'} =
  {v ∈ dom(M(d)) : subspace(v) = S'}` and `M'(d)` agrees with `M(d)` there`)` —
  the set equality is two inclusions, each a fact about the *gapped* `M'₀(d)` carried
  to `M'(d)` by the bridge's cross-subspace clause (Effect, above). Every prior
  cross-subspace position persists
  with its value (`{v ∈ dom(M(d)) : subspace(v) = S'} ⊆
  {v ∈ dom(M'(d)) : subspace(v) = S'}`, with agreement there) by ASN-0082 **I3-X
  (PostInsertionCrossSubspaceFrame)**; and INSERT adds no cross-subspace position
  (the reverse inclusion `{v ∈ dom(M'(d)) : subspace(v) = S'} ⊆
  {v ∈ dom(M(d)) : subspace(v) = S'}`) by ASN-0082 **I3-CX
  (PostInsertionCrossSubspaceClosure)**.
- (F-DOC) `(A d' : d' ≠ d : M'(d') = M(d'))` — ASN-0082 **I3-D
  (PostInsertionCrossDocumentFrame)**.
- (F-LINK) `Σ'.L = Σ.L` — the link store is untouched. INSERT's only K-atomics are
  K.α (content), K.μ⁻/K.μ⁺ (arrangement), and K.ρ (provenance); none touches `Σ.L`.
- (F-ENT) `Σ'.E = Σ.E` — the entity set is untouched. INSERT registers no entity
  (it requires `d ∈ dom(M) = E_doc` already).

We derive once, from these clauses, the range identity of the post-state
arrangement:

- (RAN) **Range identity.** `ran(M'(d)) = ran(M(d)) ∪ A_new`, and the I-addresses
  *new to the content-subspace range* of `M'(d)` are exactly
  `A_new = {shift(a, k) : 0 ≤ k < n}`. In the content subspace, I-LEFT keeps the
  left images verbatim, I-SHIFT carries each suffix image to its new slot (so those
  addresses are range-old — already in `ran(M(d))`, merely re-slotted), and I-NEW
  adds exactly `A_new`; hence the content-subspace range gains precisely `A_new` and
  loses nothing. Across the other subspaces F-SUB fixes the per-position images
  (`{M'(d)(v) : subspace(v) = S'} = {M(d)(v) : subspace(v) = S'}` for every
  `S' ≠ S`), so the cross-subspace range is unchanged. Taking the union of the
  content-subspace and cross-subspace contributions gives the full-range identity
  `ran(M'(d)) = ran(M(d)) ∪ A_new`.

## INSERT as a valid composite over the K-vocabulary

To license the appeal to ASN-0047's reachable-state machinery —
**ExtendedReachableStateInvariants** for the post-state — INSERT must be exhibited as a
*valid composite* in the precise sense of ASN-0047's **ValidComposite★**: a finite
sequence of atomic transitions whose (clause 1) per-step
preconditions each hold at the intermediate state that step acts on, and whose (clause 2)
coupling constraints J0, J1★, J1'★ hold between the composite's initial and final states.

INSERT sequences just four of the
atomics — `K.α`, `K.μ⁻`, `K.μ⁺`, `K.ρ`. The arrangement change is *not*
itself one of these atomics. It rewrites
the I-address at *existing* suffix positions — `M(d)(q_k)` at `q_k` becomes
`M'(d)(q_{k+n})` at `q_{k+n}` — which K.μ⁺'s prior-domain agreement
(`M'(d)(v) = M(d)(v)` for `v ∈ dom(M(d))`) forbids, while it strictly *grows* the
domain, which K.μ⁻ and K.μ~ (the latter by K.μ~-FIX, `dom(M'(d)) = dom(M(d))`) both
forbid. ASN-0082's I3 family is a displacement *postcondition spec*, not a
K-transition. We therefore exhibit INSERT as an explicit sequence and discharge each
step's precondition.

*Suffix-present case `1 ≤ J ≤ N`* (a genuine suffix `{q_J, …, q_N}` must move). The
sequence, read left to right with each step evaluated against the state its
predecessors leave, is

> `K.α₁, …, K.αₙ`  →  `K.μ⁻`  →  `K.μ⁺`  →  `K.ρ₁, …, K.ρₙ`.

- *`K.α₁, …, K.αₙ` (allocate).* Each commits one fresh content address along `A_C(d)`,
  with committed value `w_k ∈ Val` by precondition — discharging K.α's `v ∈ Val`
  typing obligation. Freshness splits by branch. For `k = 0` the start address
  `a = shift(a, 0)` is a
  *first* emission exactly when `d`'s content region is empty
  (`{a' ∈ dom(C) : origin(a') = d} = ∅`); there `a = [d.0.s_C.1]` and freshness is
  discharged by **FirstEmissionFreshness**. When the region is non-empty, `k = 0` is
  itself a subsequent emission and **SubsequentEmissionFreshness** applies. For every `k ≥ 1` the prior
  in-insert allocation `shift(a, k−1)` has already made the region non-empty, so the
  `k`-th step acts on a store already holding `{shift(a, 0), …, shift(a, k−1)}` and
  **SubsequentEmissionFreshness** gives `shift(a, k) ∉ dom(C) ∪ dom(L)`. The
  precondition `d ∈ dom(M)` holds throughout — no K.α step touches `M`. After these `n`
  steps `dom(C)` has grown by `A_new` and `M(d)` is still the original `{q_1, …, q_N}`.
- *`K.μ⁻` (vacate the suffix).* Acting on `d ∈ E_doc`, retain the content-subspace
  prefix `n'_{s_C} = J−1` and the link subspace in full (`n'_{s_L} = n_{s_L}`). Since
  `J−1 < N = n_{s_C}`, the content subspace contracts strictly, so K.μ⁻'s "at least one
  subspace strictly contracts" precondition is met; the retained domain is
  `{q_1, …, q_{J−1}} ∪ V_{s_L}(d)`. The intermediate text subspace is now the prefix
  alone, the link subspace untouched. The bound `J−1 < N` holds down to `J = 1`, so
  K.μ⁻ fires throughout `1 ≤ J ≤ N`.
- *`K.μ⁺` (install block and shifted suffix).* Acting on `d`, add the I-NEW block
  and the I-SHIFT shifted suffix — the same mappings the Effect fixes (the values are
  pinned there, via I3); this step installs them as one domain-extending transition
  and discharges its preconditions. Clause 1 at this intermediate state: (i) every added target lies in
  `dom(C)` — the block targets `A_new`, just committed by K.α, and the shifted-suffix
  targets are the old suffix addresses `{M(d)(q_J), …, M(d)(q_N)} ⊆ dom(C)` — which is
  exactly why the allocations must precede this step; (ii) every added V-position is
  S8a-well-formed of depth `m` — the shifted-suffix positions by **I3-VP**/**I3-VD**
  (ASN-0082), and each new-block position `shift(p, k)` because `p` is S8a (precondition)
  of depth `m` and **OrdShiftHom** (ASN-0036) preserves zero-freedom, positivity,
  subspace, and depth `m` under the shift (the `k = 0` slot being `p` itself); (iii) the
  resulting content subspace `{q_1, …, q_{N+n}}` is the dense run — by the
  block-disjointness fact (Effect) the three index intervals `{1,…,J−1}`, `{J,…,J+n−1}`,
  `{J+n,…,N+n}` are consecutive, gap-free, and union to `{1,…,N+n}` — so S8-depth,
  D-CTG★, D-MIN★ hold; (iv) every
  added position sits in subspace `s_C`, meeting the amended K.μ⁺ content-subspace
  restriction; and (v) the domain grows strictly (`J−1 < N+n`) yet stays finite — its
  content subspace is the size-`(N+n)` run `{q_1, …, q_{N+n}}`, and its link subspace
  is `V_{s_L}(d)`, carried unchanged from the pre-state `Σ` (K.α leaves `M` alone, K.μ⁻
  retains the link subspace in full), hence finite by **S8-fin** (ASN-0036) at the
  composite boundary `Σ`, so `dom(M'(d))` is a union of two finite sets and is finite
  (equivalently **I3-fin**, ASN-0082). The prior positions
  `{q_1, …, q_{J−1}}` are untouched, so prior-domain agreement holds — K.μ⁺ never
  rewrites an existing entry.
- *`K.ρ₁, …, K.ρₙ` (record provenance).* The `k`-th records `(shift(a, k), d)`; its
  precondition `shift(a, k) ∈ dom(C') ∧ d ∈ E_doc` holds because `shift(a, k)` entered
  the store at its K.α step and `d ∈ E_doc` by precondition.

The `K.μ⁻` then `K.μ⁺` pair is the K-atomic realization of the Effect's
I-LEFT/I-SHIFT/I-NEW clauses — prefix fixed, suffix vacated and re-installed `n`
higher, block filled.

*Append case `J = N+1` and empty case `V_S(d) = ∅`* (no suffix moves). I-SHIFT is
vacuous and no contraction is needed; the sequence collapses to

> `K.α₁, …, K.αₙ`  →  `K.μ⁺`  →  `K.ρ₁, …, K.ρₙ`,

the single K.μ⁺ adding only the new block `{q_J, …, q_{J+n−1}} → A_new` above the
untouched prefix `{q_1, …, q_N}` (empty when `V_S(d) = ∅`). Its preconditions are
discharged exactly as in (i)–(v) above, with prior-domain agreement again holding
because the prefix is left in place. With `J−1 = N = n_{s_C}` the content subspace
would not contract strictly, so K.μ⁻ is *inapplicable* — and unnecessary, since
nothing is vacated.

*Clause 2 — the couplings at the composite boundary.* Because INSERT both allocates
content (I-ALLOC) and places it into the content subspace of `ran(M'(d))` (I-NEW), the
boundary couplings J0, J1★, J1'★ apply; all three are driven by the range identity RAN
(Effect), by which the I-addresses *new to the content-subspace range* of `M'(d)` are
exactly `A_new = {shift(a, k) : 0 ≤ k < n}`, the shifted-suffix addresses being
range-old. **J0 (AllocationPlacementCoupling)** — every freshly allocated I-address
appears in the post-state arrangement: I-NEW places each `shift(a, k)` at
`shift(p, k) ∈ dom(M'(d))` with `d ∈ E_doc`. **J1★ (ExtensionRecordsProvenance)** —
every content-subspace range-new address carries a record, universally over the
documents of `E'_doc`. The quantifier domain is `E'_doc = E_doc` by F-ENT — INSERT
registers no document — so two instantiations exhaust it. At any `d' ≠ d`, F-DOC
gives `M'(d') = M(d')`, so no address is new to `d'`'s content-subspace range and
the instance holds vacuously. At `d` itself, the range-new addresses are exactly
`A_new` (RAN), and I-PROV records `(shift(a, k), d)` for each. **J1'★
(ProvenanceRequiresExtension)** — every new entry `(a, d) ∈ R' ∖ R` is range-new: I-PROV
adds only `A_new` records, each range-new, and recording a range-old (shifted-suffix)
address would manufacture an entry with no range-new witness. So clause 2 holds between
the composite's initial and final states.

With clause 1 verified step-by-step and clause 2 discharged just above, INSERT is a
valid composite; since `Σ` is reachable from `Σ₀`
(precondition), the post-state is reachable too, and the appeal to
ExtendedReachableStateInvariants for its post-state is licensed.

## The document remains one coherent sequence

We must check that the result is well-formed — that we have not opened a gap,
overlaid two positions, or broken the density that lets spans name contiguous
regions. **ExtendedReachableStateInvariants (ASN-0047)** therefore delivers
the *entire* post-state invariant set at once — the per-state invariants S2
(single-valuedness), S3★ (referential integrity), S8a, S8-depth, S8★, D-CTG★,
D-MIN★, D-SEQ★, and the content-store validity S7b/C1/C1b/C1c of the freshly
allocated run, together with P7 (per-state) and the composite-boundary property P7a. The final
post-state has the same arrangement as the K.μ⁺ post-state — K.ρ does not touch `M` —
so the arrangement invariants the theorem returns there are the ones K.μ⁺ established.

One reading of I-DOM is worth singling out, since it is the formal content of
Nelson's assurance (Q10) that reading end to end yields the original content with the
new material interleaved at the chosen point: restricted to the whole text subspace
`S = s_C` — where the per-subspace starred forms D-CTG★/D-MIN★/D-SEQ★ reduce to the
unstarred D-CTG/D-MIN/D-SEQ of ASN-0036 — I-DOM places the new material at exactly the
connected, ordered, gap-free block `{q_J, …, q_{J+n-1}}` within one coherent ordinal
sequence. We record the connected-region fact as a claim.

**IP1 (InsertedRun).** *The inserted material forms a single correspondence run:
for `0 ≤ k < n`, `M'(d)(shift(p, k)) = shift(a, k)`, so V-positions and
I-addresses advance in lockstep over a contiguous block. The block
`{shift(p, k) : 0 ≤ k < n}` is order-isomorphic to its image
`{shift(a, k) : 0 ≤ k < n}` under T1 (both indexings are strictly increasing
in `k` — **TS3** composition with **TS4** strict increase, ASN-0034).*

IP1 records a correspondence run in S8's sense — lockstep V/I advance over the block
— though not necessarily a *maximal* one: when the left-adjacent slot `q_{J-1}` holds
the current greatest origin-`d` address `a_prev` (reachable in the ordinary append
case, where `q_N` already holds the greatest address — and more generally after a
K.μ~ reordering (ASN-0047) places `a_prev` at `q_{J-1}` for interior `J`), the fresh
start `a = inc(a_prev, 0) = shift(M(d)(q_{J-1}), 1)` is I-adjacent to the left run (in
the merge-condition sense of **M7**, MergeCondition, ASN-0058), so
the block I-merges backward into it and is not a standalone element of the maximal-run
partition S8★ delivers. Forward I-merging with the shifted suffix, by contrast, never
happens in *any* reachable state — including the IP5 regime where a suffix slot holds
content transcluded from another document, whose I-address need not lie below the fresh
`a`. A forward merge would require the block's I-terminus `shift(a, n−1)` to be
I-adjacent to the shifted-suffix head `M'(d)(q_{J+n}) = M(d)(q_J)`, i.e.
`M(d)(q_J) = shift(a, n)`. But `shift(a, n)` is the address immediately following the
allocated run `A_new` on the content chain `A_C(d)`, beyond the post-allocation frontier:
the origin-`d` entries of `dom(C')` form a contiguous initial segment of `A_C(d)` ending
at `shift(a, n−1)` (**ChainMembershipForOrigin**, ASN-0093), and every other entry of
`dom(C')` lies on its own origin's chain, disjoint from `A_C(d)`
(**ChainMembershipForOrigin** with **CrossDocumentDisjointness**, ASN-0093), so
`shift(a, n) ∉ dom(C')` — a fortiori `∉ dom(C)` — whereas `M(d)(q_J) ∈ dom(C)` by
**S3★** (the suffix head is a content-subspace image). The two I-addresses cannot
coincide, whatever the origin of `M(d)(q_J)` and however it orders against `a`.

The new region is *seamless in arrangement yet distinguishable in identity* (Q9): in
the V-stream there is no marker at the boundary `q_{J-1} | q_J | q_{J+n}` — reading
flows across it without interruption — while in the I-stream every inserted unit
carries a fresh, origin-stamped address that records exactly which span was
introduced. The seam is erased in the arrangement and preserved in the identity.

*Provenance coupling — the obligation allocation incurs.* The inserting document's
identity is minted into the address as content enters (4/11), and the implementation
writes a DOCISPAN provenance record per inserted I-span; I-PROV is the abstract
counterpart of that record, discharged within the same composite that allocates and
places the content — never deferred to a later transition.

## Invariants the operation must preserve

We now discharge the four invariants the question names. Each is a statement
about keeping the content layer and the arrangement layer from contaminating
each other.

**Content immutability.** Nothing already in the document is rewritten; only
arrangement changes (Q3). Formally this is I-IMM together with I-ALLOC's
disjointness: `dom(C)` grows by the fresh run `A_new`, and every prior
`b ∈ dom(C)` retains its value. The store is append-only — `dom(C') ⊇ dom(C)`,
monotone — and the only writes are to addresses that did not previously exist.
We name the layer invariant:

**IP2 (ContentAppendOnly).** *`dom(C) ⊆ dom(C')` and
`(A b : b ∈ dom(C) : C'(b) = C(b))`.* INSERT is purely additive on the content
layer.

**Position permanence.** Position permanence has two senses (Q6), each discharged
by a named claim. I-address permanence holds by IP0 and IP2 — no existing binding
is disturbed and no address repurposed. V-position impermanence is exactly what
the shift performs; we name it.

**IP3 (PositionImpermanence).** *A V-position binds no permanent content. When the
insertion point is occupied (`J ≤ N`), the block slots `{q_k : J ≤ k ≤
min(J+n−1, N)}` lie in `dom(M(d)) ∩ dom(M'(d))` yet `M'(d)(q_k) = shift(a, k−J) ≠
M(d)(q_k)` — the same slot now resolves to freshly minted content, since
`shift(a, k−J) ∈ A_new` is fresh (IP0) while `M(d)(q_k) ∈ dom(C)` by **S3★
(GeneralizedReferentialIntegrity, ASN-0047)**, since `subspace(q_k) = s_C` places the
image of a content-subspace V-position in the content store. The permanence guarantee
attaches to the I-address (IP0, IP2), never to the slot.*

**Link anchoring across the displacement.** A link's endsets reference
I-addresses, not V-positions (4/42, 4/30). Since INSERT removes no I-address
(IP2) and adds only fresh ones (IP0), every link designates *exactly the same
content* after the operation as before. We can state this without modelling the
link store in detail, using only the foundation notion that a link endpoint is
an endset whose `coverage` is a set of I-addresses, and that its appearance in
document `d` is the set of V-positions of `d` mapping into that coverage. Three
facts hold — the first about the target, the next two about the resolved
witnesses:

- *The link's target is unchanged.* For any endset `e`, `coverage(e)` is a
  function of `e`'s spans alone, and INSERT never edits a stored link value. Since
  INSERT is a composite of several atomic transitions — the `n` K.α steps, the
  K.μ⁻/K.μ⁺ arrangement pair, the `n` K.ρ steps — single-transition lemmas do not
  apply to `Σ → Σ'` directly; we cite the multi-step lemmas the foundation already
  provides: **LP13 (UnconditionalLinkPersistence, ASN-0098)** fixes
  `Σ'.L(a) = Σ.L(a)` for every prior link `a` across the composite, and **LP3★
  (MultiStepCoverageInvariance, ASN-0098)** gives
  `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)` for every prior link `a` and slot
  `i`. Coverage-invariance rests on endset immutability, not on freshness. Foundation **L4 (EndsetGenerality)**
  and **L9 (TypeGhostPermission)** let an endset reference *any* tumbler, including
  ghost addresses not yet in `dom(C)`, so a pre-existing endset may already name an
  address that INSERT now mints into `A_new`.
- *The shifted-suffix witnesses move uniformly.* A link whose coverage includes
  `M(d)(v)` for some shifted `v ≥ p` is now found at `shift(v, n)`, because
  `M'(d)(shift(v, n)) = M(d)(v)` (I-SHIFT) carries the same I-address to the new
  slot. The link did not move to *different content*; the content it always named
  simply sits at a higher V-address.
- *New-block witnesses.* Precisely because a prior endset `e` may reference an
  address in `A_new` (the ghost-reference case above), INSERT can *add* witnesses
  to such a link — whether or not that link was already discoverable elsewhere.
  After the operation the new block carries `M'(d)(shift(p, k)) = shift(a, k)`; if
  `shift(a, k) ∈ coverage(e)` for some `0 ≤ k < n`, the V-position `shift(p, k)`
  newly resolves into `coverage(e)`. This new-block gain occurs for *any* link with
  `coverage(e) ∩ A_new ≠ ∅`, including a link already discoverable through other
  witnesses. Only in the special sub-case where the link is *orphaned* at `Σ` —
  discoverable from no document at all — is the gain a **resurrection in the sense
  of LP18 (ASN-0098)**: an orphaned reference becoming discoverable exactly when an
  arrangement entry to its target appears. These witnesses live at the inserted
  block, not at any `shift(v, n)`.

This is the precise sense of Nelson's survivability clause restricted to
insertion (4/43): because insertion removes nothing, *every* link survives with
its designated content unchanged. We record it.

**IP4 (LinkSurvival).** *For every prior link `a ∈ dom(Σ.L)` and every slot `i`,
with `e = Σ.L(a).eᵢ` its endset, LP3★ (with LP13 across the composite, ASN-0098) fixes
`coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)` — no link's designated content
changes. The post-insert resolved-witness set of `e` in `d` is
`project(e, d, Σ') = {v ∈ dom(M'(d)) : M'(d)(v) ∈ coverage(e)}`, which decomposes
into four disjoint parts:*

- *Left witnesses: `{v ∈ V_S(d) : v < p ∧ M(d)(v) ∈ coverage(e)}`, preserved
  verbatim by I-LEFT.*
- *Shifted-suffix witnesses: `{shift(v, n) : v ∈ V_S(d) ∧ v ≥ p ∧ M(d)(v) ∈
  coverage(e)}`, carried to the new slot by I-SHIFT.*
- *Cross-subspace witnesses: `{v ∈ dom(M(d)) : subspace(v) ≠ S ∧ M(d)(v) ∈
  coverage(e)}`, preserved verbatim by F-SUB (a link's coverage may include images
  of `d`'s positions in another subspace).*
- *New-block witnesses, present iff `coverage(e) ∩ A_new ≠ ∅`:
  `{shift(p, k) : 0 ≤ k < n ∧ shift(a, k) ∈ coverage(e)}` (a resurrection in the
  sense of LP18 only when the link was orphaned at `Σ`).*

*The prior witness set `project(e, d, Σ)` partitions into left, suffix, and
cross-subspace witnesses, and INSERT maps these injectively into the post-insert
set: left and cross-subspace verbatim, suffix by the bijection `v ↦ shift(v, n)`
(I-SHIFT; injective by **TS2**, ShiftInjectivity, ASN-0034). Whether the prior set is *contained* in the post-insert set turns on the
suffix part. When no suffix witness shifts — the suffix part is empty — every prior
witness is a left or cross-subspace witness, retained at its own V-position, so
`project(e, d, Σ) ⊆ project(e, d, Σ')` (proper iff the new-block part is non-empty).
When at least one suffix witness is present, `project(e, d, Σ') ⊄ project(e, d, Σ)`
always holds: the largest shifted witness `shift(v_max, n)` — image of the greatest
suffix witness `v_max` — lands at a position `> v_max` (**TS4**, ShiftStrictIncrease,
ASN-0034) that, were it itself a prior
witness, would contradict `v_max` being the greatest suffix witness; so it carried no
coverage witness before. The reverse inclusion `project(e, d, Σ) ⊆ project(e, d, Σ')`
may or may not hold: a vacated suffix slot `v` need not leave the post-insert set,
because `M'(d)` re-populates `v` with new content — a block address from `A_new` when
`v` now falls in the block range, a shifted-suffix address otherwise — and that content
may itself lie in `coverage(e)` (L4/L9 permit an endset to reference both `A_new` and
the shifted-suffix addresses), keeping `v` a witness. So the two V-position sets are
not comparable in a fixed direction — incomparable in some configurations,
`project(e, d, Σ) ⊊ project(e, d, Σ')` in others. In every case the map is a bijection
from the prior set onto (left ∪ shifted-suffix ∪ cross-subspace). Hence the witness **count** is
non-decreasing,*

> `|project(e, d, Σ')| = |project(e, d, Σ)| + |{shift(p, k) : 0 ≤ k < n ∧ shift(a, k) ∈ coverage(e)}|`,

*and the resolved **content** grows monotonically,*

> `coverage(e) ∩ ran(M(d)) ⊆ coverage(e) ∩ ran(M'(d))`,

*with equality in both iff the new-block part is empty, i.e. iff
`coverage(e) ∩ A_new = ∅`.*

**Isolation of documents sharing I-addresses.** Suppose another document `d'`
arranges some of the same content `d` does — `ran(M(d')) ∩ ran(M(d)) ≠ ∅`. The
question is whether inserting into `d` can perturb `d'`. It cannot, and the
proof is the conjunction of three facts already in hand. By F-DOC,
`M'(d') = M(d')` — `d'`'s arrangement is untouched. By IP2, the
content I-addresses `d'` references retain their content — the bytes `d'` reads
are immutable (and by F-LINK any link-subspace images retain their link values).
Therefore `d'` resolves
every one of its V-positions to the same content, in the same order, before and
after: its arrangement *and its reader's experience* are identical (Q8). The isolation is a structural consequence of the two-layer split: INSERT
writes the arrangement of exactly one document (F-DOC) and appends to the global
content store without disturbing any existing entry (IP2). Sharing is by
reference to immutable identity, so an insertion into one sharer is invisible to
the others.

**IP5 (DocumentIsolation).** *For every `d' ≠ d`: `M'(d') = M(d')`, and for every
`v' ∈ dom(M(d'))` the resolved entity is invariant per subspace —
`subspace(v') = s_C ⟹ M'(d')(v') ∈ dom(C')` with `C'(M'(d')(v')) = C(M(d')(v'))`
(content value fixed by IP2), and `subspace(v') = s_L ⟹ M'(d')(v') ∈ dom(L')` with
`L'(M'(d')(v')) = L(M(d')(v'))` (link value fixed by F-LINK). The arrangement and
resolved content of every other document are invariant under INSERT on `d`.*

## A weakest precondition: when is discoverability preserved?

IP4 leaves one question pointed but unanswered: under what condition does INSERT
preserve, rather than merely not-shrink, the set of links discoverable from `d`?
It is tempting to assume the answer is "always" — insertion removes nothing.
Computing the weakest precondition shows otherwise, and the place it fails is
exactly the new-block-witness gap IP4 now records.

Write `D(d, Σ) = {a ∈ dom(Σ.L) : discoverable_from(a, d, Σ)}` for the links
discoverable from `d` (foundation `discoverable_from`, ASN-0098). We seek

> `wp(INSERT, "D(d, Σ') = D(d, Σ)")`.

By **LP12 (DiscoverabilityCharacterisation, ASN-0098)**, a link `a` is
discoverable from `d` iff some slot's coverage meets the document's I-address
range: `discoverable_from(a, d, Σ) ⟺ (E i : coverage(Σ.L(a).eᵢ) ∩ ran(M(d)) ≠
∅)`. LP12 consumes the *full* `ran(M(d))` — content and cross-subspace alike — so
the entire question reduces to how INSERT changes `ran(M(d))`. The full-range
identity is RAN above:

> `ran(M'(d)) = ran(M(d)) ∪ A_new`,

whose cross-subspace contribution is fixed by F-SUB and whose content-subspace
contribution gains exactly `A_new`. Substituting the
full-range identity RAN into
LP12, for every prior link `a` — and noting that the unsubscripted `coverage(eᵢ)`
below is well-defined because each slot's coverage is invariant pre-to-post across
the whole composite (LP13 + **LP3★ (MultiStepCoverageInvariance, ASN-0098)**, so
`coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`) —

```
  discoverable_from(a, d, Σ')
    ⟺ (E i : coverage(eᵢ) ∩ (ran(M(d)) ∪ A_new) ≠ ∅)
    ⟺ discoverable_from(a, d, Σ)  ∨  (E i : coverage(eᵢ) ∩ A_new ≠ ∅).
```

The biconditional is per-link, quantified over the *prior* links `a ∈ dom(Σ.L)`.
To lift it to the sets, the quantification must exhaust `D(d, Σ')` — which is by
definition a subset of `dom(Σ'.L)` — and F-LINK supplies exactly this:
`Σ'.L = Σ.L`, hence `dom(Σ'.L) = dom(Σ.L)`, so the composite mints no link and
every member of `D(d, Σ')` is a prior link the biconditional covers. Therefore
`D(d, Σ') = D(d, Σ) ∪ Added`, where
`Added = {a ∈ dom(Σ.L) : (E i : coverage(Σ.L(a).eᵢ) ∩ A_new ≠ ∅)}` is the set of
links the freshly minted run would newly witness. Since `D(d, Σ')` is the *union*
of the prior set with `Added`, the two coincide iff `Added ⊆ D(d, Σ)` — **not** iff
`Added = ∅`. The distinction is exactly the configuration L4/L9 permit: a link
whose endset has one span into `A_new` (a ghost reference) *and* another span
already meeting `ran(M(d))` lies in `Added` yet was *already* discoverable, so
adding its new-block witness leaves `D(d)` unchanged. The weakest precondition is
thus the operation's precondition conjoined with a *containment*, not an
emptiness. We record it as a named claim, parallel to IP0–IP5:

**IP6 (DiscoverabilityWP).** *The weakest precondition under which INSERT preserves
the set of links discoverable from `d` is*

> `wp(INSERT, D(d, Σ') = D(d, Σ)) ≡ INSERT-pre ∧
> {a ∈ dom(Σ.L) : (E i : coverage(Σ.L(a).eᵢ) ∩ A_new ≠ ∅)} ⊆ D(d, Σ)`.

In words: discoverability from `d` is preserved precisely when every link the new
run would newly witness was *already* discoverable from `d`. The strictly stronger
*sufficient* condition `(A a ∈ dom(Σ.L), i : coverage(Σ.L(a).eᵢ) ∩ A_new = ∅)` —
no prior endset references the allocated run at all — discharges the containment by
emptying `Added`, but it over-rejects: it refuses the ghost-plus-live-span
pre-states above, on which discoverability is in fact preserved. Two corollaries fall out. (i) A
sufficient condition discharging the wp for free is a tight-endset discipline: if
every prior endset is tight at its creation state (foundation `tight`, ASN-0098),
then **LP19a (TightFreshness)** gives `A_new ∩ coverage(e) = ∅` for every K.α-fresh
address, so `Added = ∅ ⊆ D(d, Σ)` and the wp reduces to `INSERT-pre`. (ii) Absent
that discipline, the containment wp is the sharpest statement available, and it is
the formal witness that "insertion preserves discoverability" is a *conditional*,
not a theorem.

## A worked insertion

Fix the text subspace `S = s_C` at depth `m = 2`, so `q_k = [s_C, k]` and
`shift(q_k, n) = [s_C, k+n] = q_{k+n}`. Let `d` currently hold `N = 5` text
positions, `V_S(d) = {q_1, …, q_5}`, with I-addresses
`M(d)(q_k) = a_k` for `k = 1, …, 5`. Suppose `d`'s content chain has greatest
I-address `a_max = [d.0.s_C.6]`, so K.α's next emission is
`a = inc(a_max, 0) = [d.0.s_C.7]`.

**Insert `XY` (`n = 2`) at `p = q_3`.** Here `J = 3`, `S = s_C`, `m = #p = 2`,
and `1 ≤ J ≤ N+1`, so `ValidInsertionPosition(d, q_3)` holds.

*Allocation (I-ALLOC, IP0).* `A_new = {shift(a, 0), shift(a, 1)} = {[d.0.s_C.7],
[d.0.s_C.8]}`, both fresh (`a_max` was `[d.0.s_C.6]`, so neither `.7` nor `.8`
was in `dom(C)`), contiguous, origin-`d`. Content written: `C'([d.0.s_C.7]) =
X`, `C'([d.0.s_C.8]) = Y`. ✓ IP0, IP2.

*Shift (I-SHIFT, I-LEFT).* Prefix `q_1, q_2` unchanged (I-LEFT). Suffix `q_k`
with `k ≥ 3` moves by `shift(·, 2)`:

```
  q_3 → q_5  carrying a_3      q_4 → q_6  carrying a_4      q_5 → q_7  carrying a_5
```

*New block (I-NEW, IP1).* The vacated block `{shift(p, 0), shift(p, 1)} = {q_3,
q_4}` maps in lockstep to `A_new`: `M'(d)(q_3) = [d.0.s_C.7]`, `M'(d)(q_4) =
[d.0.s_C.8]`. The pair `q_3 < q_4` is order-isomorphic to `[d.0.s_C.7] <
[d.0.s_C.8]`. ✓ IP1.

*Domain (I-DOM).* The three index intervals are `{1, 2}` (prefix), `{3, 4}`
(new), `{5, 6, 7}` (shifted suffix) — consecutive, disjoint, union `{1, …, 7}`.
So `V_S(d') = {q_1, …, q_7}`, the dense run with `N' = N + n = 7`. ✓ I-DOM.

*Reading end to end* now yields `a_1, a_2, X, Y, a_3, a_4, a_5` — the original
content with `XY` interleaved between the second and third units, exactly
Nelson's promise (Q10).

*Provenance (I-PROV, J0/J1★/J1'★).* The two freshly allocated addresses each
acquire a record: `R' = R ∪ {([d.0.s_C.7], d), ([d.0.s_C.8], d)}`. Trace the
three coupling constraints against this concrete shift. **J0** — every freshly
allocated I-address appears in `M'(d)`: `[d.0.s_C.7]` sits at `q_3` and
`[d.0.s_C.8]` at `q_4` (the new block above), both with `d ∈ E_doc`. ✓ **J1★** —
the addresses *new to the content-subspace range* of `M'(d)` are exactly
`A_new = {[d.0.s_C.7], [d.0.s_C.8]}` (the shifted suffix `a_3, a_4, a_5` was
already in `ran(M(d))`, hence range-old), and `R'` carries a record for each. ✓
**J1'★** — the only entries in `R' ∖ R` are these two, both range-new. The
subtle case is the shifted suffix: `a_3, a_4, a_5` now occupy the *new* slots
`q_5, q_6, q_7`, yet provenance keys on `(I-address, document)`, not on
V-position, and these addresses are *range-old* — already in `ran(M(d))` at the
pre-state, where by P4★ at the composite boundary the records `(a_3, d)`,
`(a_4, d)`, `(a_5, d)` already sit in `R`. Their V-positions changed, but their
I-addresses did not, so they induce **no** new entry.
The range-based coupling records exactly `A_new`. ✓
Finally **P7a** at the post-state for a prior address: `a_1` carried some
`(a_1, d') ∈ R` at the pre-state (P7a there), and `R ⊆ R'` preserves it, so `a_1`
remains covered; the two fresh addresses are covered by the records just added. ✓

**Links over the insertion (IP4, IP5, IP6).** Equip `d` with two links to drive the
link claims against this concrete shift.

*A link that both shifts and resurrects.* Write `g := [d.0.s_C.8]`, and let `ℓ`
carry the two-span endset

> `e = {(a_3, δ(1, #a_3)), (g, δ(1, #g))}`,

two unit-depth spans, one starting at the live address `a_3 = M(d)(q_3)`, one at
the ghost `g` — not yet in `dom(C)` — which L4/L9 permit an endset to name. The
coverage is *not* the two-element set `{a_3, g}`: by **PrefixSpanCoverage**
(ASN-0043) each unit-depth span denotes the entire (infinite) prefix-subtree of
its start, so `coverage(e) = {t : a_3 ≼ t} ∪ {t : g ≼ t}`. What the argument
consumes is only the intersections with `ran(M(d))` and `A_new`, and these we
discharge explicitly. Both spans are canonical with starts in the substrate form
`F` of ASN-0098 (`a_3 ∈ dom(C) ⊆ F` by **LP-Sub**; `g ∈ F` by its shape
`[d.0.s_C.8]`), so **LP-Fin** at `n = 1` gives `F ∩ {t : s ≼ t} = {s}` for each
start `s`: a subtree member beyond the start itself strictly extends it, forcing
`#E ≥ 3` (or a fourth zero), whereas every store entry — hence every arrangement
image (S3★) and every element of `A_new` (chain form, **ChainMembershipForOrigin**,
ASN-0093) — has `#E = 2`. Each subtree therefore meets `ran(M(d))` and `A_new` in
at most its start address, and the intersections reduce to start-membership checks:

- `coverage(e) ∩ ran(M(d)) = {a_3, g} ∩ ran(M(d)) = {a_3}` — `a_3 = M(d)(q_3)` is
  in the range, while the ghost `g` lies beyond the frontier `a_max = [d.0.s_C.6]`,
  so it is in neither `dom(C)` nor (by S3★) any arrangement range.
- `coverage(e) ∩ A_new = {a_3, g} ∩ A_new = {g}` — `g = shift(a, 1)` is the second
  fresh address, while `a_3 ∈ dom(C)` cannot be fresh.

So `project(e, d, Σ) = {q_3}`: one witness, at `q_3`. After the
insert, IP4's four parts are: left `∅`; cross-subspace `∅`; shifted-suffix
`{q_5}`, since `q_3 ≥ p` carries `a_3` to `shift(q_3, 2) = q_5`; new-block
`{q_4}`, since `shift(a, 1) = g ∈ coverage(e)` puts a witness at
`shift(p, 1) = q_4`. Hence `project(e, d, Σ') = {q_4, q_5}`. The prior witness set
`{q_3}` is **not** a subset of `{q_4, q_5}` — the witness was *relabelled*
(`q_3 → q_5`), not retained — an instance where the sets are incomparable, a suffix
witness being present (`q_3 ≥ p`). The
count rose by exactly the one new-block witness (`1 → 2`), and the resolved content
grew monotonically: by RAN, `coverage(e) ∩ ran(M'(d)) = (coverage(e) ∩ ran(M(d)))
∪ (coverage(e) ∩ A_new)`, i.e. `{a_3} ⊆ {a_3, g}`. ✓ IP4.

*The IP6 trap.* Was discoverability of `ℓ` from `d` *newly* gained? No — `ℓ` was
*already* discoverable via `a_3` (`coverage(e) ∩ ran(M(d)) = {a_3} ≠ ∅`), so
`ℓ ∈ D(d, Σ)`. Yet `ℓ ∈ Added`, since `coverage(e) ∩ A_new = {g} ≠ ∅`.
So `ℓ ∈ Added ∩ D(d, Σ)`: `ℓ` lies in `Added` yet contributes no *new*
discoverable link, having been discoverable already. This is the per-link
configuration that separates the two wp forms. The *sufficient* emptiness form
`Added = ∅` is already violated by `ℓ`, so it would refuse to certify
preservation — even though `ℓ`'s own membership in `Added` is harmless; the
*weakest* containment form tolerates `ℓ` precisely because `ℓ ∈ D(d, Σ)`, so `ℓ`
alone could never break the containment `Added ⊆ D(d, Σ)`. Whether `D(d)` is
preserved at the document level turns on the *whole* of `Added`, not on `ℓ`: here
`Added = {ℓ, ℓ'}` also contains the orphaned `ℓ'` (below), which `D(d, Σ)` does
not, so the containment fails and `D(d)` *does* change — driven entirely by `ℓ'`,
never by `ℓ`. ✓ IP6.

*A genuine resurrection.* Write `g' := [d.0.s_C.7]` and let `ℓ'` carry the
single-span endset `e' = {(g', δ(1, #g'))}`, so `coverage(e') = {t : g' ≼ t}` — a
unit-depth subtree rooted at a ghost. By the same LP-Fin reduction, `coverage(e')`
meets any store-backed set in at most `{g'}`; and `g'` is in neither `dom(C)` (it
lies beyond the frontier `a_max`) nor `dom(L)` (its subspace identifier is `s_C`,
while every link address carries `s_L` — L0), so
`coverage(e') ∩ ran(M(d'')) = ∅` for every document `d''`: `ℓ'` is orphaned at
`Σ`. After the insert the new block carries
`M'(d)(q_3) = g' ∈ coverage(e')`, so `q_3 ∈ project(e', d, Σ')`: `ℓ'`
becomes discoverable from `d`. Here `ℓ' ∈ Added ∖ D(d, Σ)`, so the combined
`Added = {ℓ, ℓ'} ⊄ D(d, Σ)`: IP6's containment fails, and
`D(d, Σ') = D(d, Σ) ∪ {ℓ'} ⊋ D(d, Σ)` — a real change to the discoverable set,
driven by `ℓ'`, and a **resurrection in LP18's sense** because `ℓ'` was orphaned.
✓ IP4 new-block, IP6 escape branch.

*Isolation (IP5).* Suppose `d'` also arranges `a_3`: `M(d')(q'_1) = a_3`. INSERT on
`d` leaves `M'(d') = M(d')` (F-DOC), and `a_3 ∈ dom(C)` retains its value (IP2).
So `d'` resolves
`q'_1` to `a_3`'s content exactly as
before — untouched by the insertion into `d`. ✓ IP5.

**Boundary — append (`J = N + 1 = 6`).** Take `p = q_6 = shift(max(V_S(d)), 1) =
shift(q_5, 1)`, `ValidInsertionPosition(d, q_6)`. No position `v ≥ q_6` lies in
`V_S(d)`, so I-SHIFT is vacuous; I-LEFT preserves all of `q_1, …, q_5`; the new
block `{q_6, q_7}` receives `A_new`. `V_S(d') = {q_1, …, q_5} ∪ {q_6, q_7} =
{q_1, …, q_7}`, `N' = 7`. The inserted material lands at the end, no suffix
moves. ✓ I-DOM, I-NEW, IP1.

**Boundary — empty subspace (`V_S(d) = ∅`).** The first insertion fixes the
depth. Choose `m = 2` and `p = q_1 = [s_C, 1]`, so
`ValidFirstInsertionPosition(d, p, 2)` holds. Insert `XY` (`n = 2`): no prefix,
no suffix; the new block `{q_1, q_2}` receives `A_new`. The empty *arrangement*
`V_S(d) = ∅` does *not* by itself fix the K.α start address: that is selected by
the *content region* `{a' ∈ dom(C) : origin(a') = d}`, a distinct condition,
because content is append-only (IP2) and persists through arrangement
contraction. We work both sub-cases.

*Sub-case (a) — fresh document, content region empty.* Here additionally
`{a' ∈ dom(C) : origin(a') = d} = ∅`, so `k = 0` is a *first* emission
`a = [d.0.s_C.1]`, freshness by FirstEmissionFreshness; the second allocation
`[d.0.s_C.2]` is then a subsequent emission (the region is now non-empty), by
SubsequentEmissionFreshness. So `A_new = {[d.0.s_C.1], [d.0.s_C.2]}`,
`V_S(d') = {q_1, q_2}`, `N' = 0 + 2 = 2`, depth pinned at `m = 2`.

*Sub-case (b) — re-insertion after full contraction.* Suppose `d` previously held
text later fully removed by `K.μ⁻` to `V_S(d) = ∅`, leaving the content region
non-empty — say greatest origin-`d` address `[d.0.s_C.6]`. Now `V_S(d) = ∅` while
`{a' ∈ dom(C) : origin(a') = d} ≠ ∅`, so even `k = 0` is a *subsequent* emission:
the start is `a = inc([d.0.s_C.6], 0) = [d.0.s_C.7]` (SubsequentEmissionFreshness),
and `A_new = {[d.0.s_C.7], [d.0.s_C.8]}`. The arrangement nevertheless starts
fresh — `p = q_1 = [s_C, 1]` re-establishes `min(V_S(d')) = q_1` (D-MIN★), and the
depth `m` is re-pinned freely (here `m = 2`) by this first insertion, independent
of the now-distinct content-address ordinals. So `V_S(d') = {q_1, q_2}`, `N' = 2`.

Either sub-case: the subspace depth is now pinned at `m = 2` for every later
insertion. ✓ I-DOM (with `J = 1`, prefix and suffix intervals empty), I-NEW, IP1.

**Boundary — front insertion into a non-empty document (`J = 1`).** Return to the
`N = 5` document `V_S(d) = {q_1, …, q_5}` and insert `XY` (`n = 2`) at `p = q_1`. Here
`J = 1`, `1 ≤ J ≤ N+1`, so `ValidInsertionPosition(d, q_1)` holds — and unlike the
empty-subspace case there *is* a suffix, the whole of `V_S(d)`, to displace. This is
the only branch exercising the `n'_{s_C} = 0` strict-contraction precondition.

*Vacate (K.μ⁻).* The retained content prefix is `n'_{s_C} = J − 1 = 0`, so the retained
domain `{q_1, …, q_0}` is empty and `V_{s_C}(d)` clears entirely; strict contraction
`0 < N = 5` still holds (the link subspace is retained in full), so K.μ⁻ *fires* rather
than being dropped. After it the content subspace is empty, the whole suffix vacated.

*Re-install (K.μ⁺).* Every `q_k` (all `k ≥ 1 = J`) shifts by `shift(·, 2)`:
`q_1 → q_3` carrying `a_1`, …, `q_5 → q_7` carrying `a_5` (I-SHIFT). The vacated block
`{q_1, q_2}` maps in lockstep to `A_new`: `M'(d)(q_1) = [d.0.s_C.7]`,
`M'(d)(q_2) = [d.0.s_C.8]` (I-NEW). The reinstalled subspace is `V_S(d') = {q_1, …, q_7}`
with `min(V_S(d')) = q_1` (D-MIN★) and `N' = 7`.

*Reading end to end* now yields `X, Y, a_1, a_2, a_3, a_4, a_5` — the inserted material
ahead of all prior content. ✓ K.μ⁻ strict contraction at `n'_{s_C} = 0`, I-SHIFT, I-NEW,
I-DOM, D-MIN★.

## What we have established

The claims established are catalogued below.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| INSERT | Operation: place `n` fresh content units at valid V-position `p` in document `d`, as the valid ASN-0047 composite `K.α`(×n) → `K.μ⁻` → `K.μ⁺` → `K.ρ`(×n) (K.μ⁻ dropped in the append/empty cases), whose arrangement net effect realises ASN-0082's I3 shift | introduced (composite) |
| IP0 (OriginIdentity) | The `n` allocated I-addresses `{shift(a,k) : 0 ≤ k < n}` are fresh and distinct from all prior addresses, independent of content value | restated (K.α freshness + S4, ASN-0036/0093) |
| IP1 (InsertedRun) | The inserted material forms one correspondence run: `M'(d)(shift(p,k)) = shift(a,k)`, V- and I-addresses advancing in lockstep over a contiguous block | introduced |
| IP2 (ContentAppendOnly) | `dom(C) ⊆ dom(C')` and existing values preserved; INSERT is purely additive on content | restated (C0, ASN-0093) |
| IP3 (PositionImpermanence) | A V-position binds no permanent content: an occupied block slot `q_k` (J ≤ k ≤ min(J+n−1, N)) satisfies `M'(d)(q_k) = shift(a,k−J) ≠ M(d)(q_k)`, resolving to fresh content; permanence attaches to the I-address (IP0, IP2), not the slot | introduced |
| IP4 (LinkSurvival) | Every prior endset's coverage is unchanged (LP13+LP3★, ASN-0098, across the composite); post-insert witness set = left ∪ shifted-suffix ∪ cross-subspace ∪ new-block; prior witnesses map bijectively onto the first three parts (suffix relabelled by `shift(·,n)`), so witness count is non-decreasing and resolved content grows monotonically (new-block is LP18 resurrection only when the link was orphaned) | introduced |
| IP5 (DocumentIsolation) | Every other document's arrangement and resolved content are invariant under INSERT on `d` | introduced |
| IP6 (DiscoverabilityWP) | `wp(INSERT, D(d,Σ')=D(d,Σ)) ≡ INSERT-pre ∧ {a : (∃i) coverage(Σ.L(a).eᵢ) ∩ A_new ≠ ∅} ⊆ D(d,Σ)` (containment, not emptiness); the emptiness form is sufficient but strictly stronger; discharged free under tight-endset discipline (LP19a) | introduced |
| I-ALLOC | `dom(C') = dom(C) ∪ A_new`, `C'(shift(a,k)) = w_k` | cited (K.α, ASN-0093), iterated |
| I-IMM | `(A b : b ∈ dom(C) : C'(b) = C(b))` — existing content values unchanged | cited (C0, ASN-0093) |
| I-PROV | `R' = R ∪ {(shift(a,k), d) : 0 ≤ k < n}` — provenance record per allocated address, within the same composite as allocation (not deferred) | cited (K.ρ, ASN-0047), iterated |
| I-SHIFT | V-positions `≥ p` in subspace `S` move to `shift(v,n)`, carrying their I-address | cited (I3, ASN-0082) |
| I-LEFT | V-positions `< p` in subspace `S` are unchanged | cited (I3-L, ASN-0082) |
| I-NEW | The vacated block `{shift(p,k)}` maps to the fresh run `{shift(a,k)}` | introduced (composition glue) |
| I-DOM | `V_S(d')` is the dense run `{q_1, …, q_{N+n}}` with `N' = N+n`; the post-state D-SEQ/D-MIN/D-CTG follow from this shape via ExtendedReachableStateInvariants (ASN-0047) | introduced (interval argument; prefix+suffix from I3-CS, ASN-0082; middle block from I-NEW) |
| F-SUB | Positions in subspaces `S' ≠ S` are unchanged (subspace confinement of the shift): prior positions persist (I3-X) and none are added (I3-CX) | cited (I3-X + I3-CX, ASN-0082) |
| F-DOC | Arrangements of all documents `d' ≠ d` are unchanged | cited (I3-D, ASN-0082) |
| F-LINK | `Σ'.L = Σ.L` — the link store is untouched | cited (frame; no K-atomic touches `Σ.L`) |
| F-ENT | `Σ'.E = Σ.E` — the entity set is untouched | cited (frame; INSERT registers no entity) |

## Open Questions

What must INSERT guarantee when the insertion point names a position that is currently shared, by transclusion, with another document's arrangement?

Under what conditions, if any, may two concurrent insertions into the same document's content scope both claim freshness without a serializing authority?

What must provenance guarantee when content is placed into a document not by fresh allocation but by transclusion of an address whose provenance already names a different origin document?

What relationship must hold between the inserted run's contiguity at creation and the system's obligations after later editing fragments that run?
