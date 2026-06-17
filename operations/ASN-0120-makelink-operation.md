> **ASN-0120 · The MAKELINK Operation — Connection Recorded by Content Identity** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0042 · Tumbler Ownership](../foundation/ASN-0042-tumbler-ownership.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0120-makelink-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0120: The MAKELINK Operation — Connection Recorded by Content Identity

*2026-06-08*

## The problem

A link in Nelson's design is "not between points, but between spans of data...
a strap between bytes." The operation that fastens such a strap is MAKELINK. The
system is handed three *endset arguments* — a from, a to, and a type — each
naming content regions somewhere in the docuverse, together with a *home
document* in which the link is to live. MAKELINK ties the regions together and
returns the link's identity.

We are asked to be exact about what that act consists of. What does the system
allocate as the link's identity, and how permanent is it? What does it record
about each endset — and in what coordinates, such that the record means the same
thing tomorrow as today? Where does the link itself reside, and what relationship
must its home document bear to the content its endsets touch? What does supplying
*three* endsets rather than two disclose about directionality, about typing, and
about the line between a bare connection and a typed relation? And what invariants
must the operation preserve once the link exists — about the permanence of the
link's identity, the immutability of the endsets it recorded, the independence of
where the link *lives* from what it *connects*, and the system's ability to
discover the link from any region its endsets reference?

We shall find that the whole content of MAKELINK is a single conversion of
coordinates. The endset arguments name content by its *arrangement position* — a
V-position in a document — but the link records each endset by *content identity*
— the I-address of the content there. Position is mutable; identity is permanent.
Everything the question asks about — survivability, immutability, discoverability
that ignores residence — is a consequence of recording at the identity level and
of the orthogonality between a link's home (where it lives) and its endsets (what
it touches).

## The substrate we build on

**Standing precondition (reachability).** Throughout, every state `Σ` ranges over
states reachable from the initial state `Σ₀` under the sequential transition order
(ASN-0047, SequentialTransitionAxiom).

We take the strand and link models as given. The *content store*
`Σ.C : T ⇀ Val` (ASN-0036) binds *I-addresses* to values; it is append-only and
immutable — once `a ∈ dom(Σ.C)`, `a` persists and `Σ.C(a)` never changes (S0
ContentImmutability; S1 StoreMonotonicity). The *arrangement* of a document `d` is
a partial function `Σ.M(d) : T ⇀ T` (ASN-0036) from V-positions to I-addresses,
a genuine function (S2) whose every image lies in the content store (S3
ReferentialIntegrity), and it is the one component of state editable in place. The
*link store* `Σ.L : T ⇀ Link` (ASN-0043, ASN-0093) maps link addresses to link
values and is permanent (L12 LinkImmutability).

A *link value* is a finite sequence of `N ≥ 3` endsets,
`Link = (e₁, …, eₙ)`, each `eᵢ ∈ Endset = 𝒫_fin(Span)` (ASN-0043, L3). A span
`(s, ℓ)` satisfies T12 (ASN-0034) and denotes the order-convex set
`⟦(s, ℓ)⟧ = {t : s ≤ t < s ⊕ ℓ}`; the *coverage* of an endset is the union of its
span denotations, `coverage(e) = (∪ (s,ℓ) : (s,ℓ) ∈ e : ⟦(s,ℓ)⟧)` (ASN-0043,
ASN-0098). For a link address `a`, `home(a) = N(a).0.U(a).0.D(a)` is the
document-level prefix recovered by field projection (ASN-0043), coinciding with
`origin(a)` on link addresses (ASN-0086, HomeOriginCoincidence). We write
`subspace(v) = v₁` (ASN-0036) and fix the link subspace `s_L` and content
subspace `s_C` with `s_C ≠ s_L` (ASN-0047, SubspaceConventionAxiom). For a
document `d ∈ dom(Σ.M)`, the link sub-allocator `A_L(d)` produces fresh
link-subspace addresses scoped to `d`, with first emission `[d.0.s_L.1]` and
successors `inc(·, 0)` (ASN-0093, K.λ; FirstEmission, ChainDiscipline).

The link-creation effect is a composite `Σ →* Σ'` of two elementary
transitions: the substrate's `K.λ` (LinkAllocation, ASN-0093, ASN-0047)
followed by the link-subspace arrangement extension `K.μ⁺_L` (ASN-0047) that
seats the new link in its home document's V-stream. We reserve `→`, as the
foundations do (SequentialTransitionAxiom), for single elementary transitions;
the composite passes through an intermediate state between its two steps and is
written `Σ →* Σ'`. MAKELINK is the user-level operation those two elementary
transitions implement; our task is to say what it
must guarantee abstractly. As a composite it must satisfy both clauses of
ASN-0047's ValidComposite★: the elementary preconditions of `K.λ` and `K.μ⁺_L`
(discharged where each is invoked below) and the coupling constraints J0, J1★,
J1'★ between initial and final state. The coupling constraints hold *vacuously*,
for two distinct reasons. MAKELINK allocates no content (`Σ'.C = Σ.C`, ML10), so
there is no fresh content address (J0 vacuous) and no content-subspace range-new
I-address (J1★ vacuous) — the only V-position added is a link-subspace one (via
`K.μ⁺_L`). J1'★ quantifies instead over new provenance entries `(a, d) ∈ R' \ R`,
and `R' \ R = ∅`: both `K.λ` and `K.μ⁺_L` carry `E' = E ∧ R' = R` in their
ASN-0047 frames (ML10).

## What the endset arguments name, and what resolution recovers

The from/to/type arguments do not arrive as I-addresses. They arrive as
*content-region specifications*: each is a spec-set
`R = ⟨(d₁, σ₁), …, (dₚ, σₚ)⟩`, a finite sequence of V-specs each naming a source
document `d_j` and a V-span `σ_j = (u_j, ℓ_j)` over it. The conditions such a
spec-set must satisfy we collect into a single *well-formedness* predicate, which
MAKELINK requires as a precondition — it is the surface ML1's content-containment
rests on:

> `wf(R, Σ) ≡ (A j : 1 ≤ j ≤ p : d_j ∈ dom(Σ.M) ∧ subspace(u_j) = s_C ∧ #u_j ≥ 2 ∧ (E n_j ≥ 1 : ℓ_j = δ(n_j, #u_j)))`

Each `σ_j` thus names an allocated source (`d_j ∈ dom(Σ.M)`), lies in the *content
subspace* (`subspace(u_j) = s_C`) at some depth `m = #u_j ≥ 2`, and carries an
*ordinal displacement* `ℓ_j = δ(n_j, m)` — equivalently
`#ℓ_j = #u_j ∧ actionPoint(ℓ_j) = #u_j` (equal length places the action point at
the last component with zeros below it, recovering the δ-form; the action-point
conjunct alone would admit longer displacements that are not ordinal). The
action-point conjunct is the tight half of T12's `actionPoint(ℓ_j) ≤ #u_j`, and
with `Pos(ℓ_j)` from `n_j ≥ 1`, every `σ_j` is T12-well-formed (ASN-0034). Note what `wf` does *not* require: that
`#u_j` equal the common depth S8-depth fixes on `V_{s_C}(d_j)` (the analogue of
ASN-0058's ContentReference condition (iii), which is moreover undefined when
`V_{s_C}(d_j) = ∅`). A spec whose depth differs from the arrangement's is
admitted; its interval `⟦σ_j⟧` may still capture active positions at the
arrangement's own depth — a depth-2 start `[1,1]` bounds depth-3 positions, since
`[1,1] ≤ [1,1,1] < [1,2]` under T1 — and `ρ` below resolves exactly those: the
active-position filter, not a depth match, decides what is recovered. An endset
argument that reaches into the link subspace — a link pointing at another link —
is deferred (Open Questions).
A V-span lives in *arrangement* coordinates — the positions a reader currently sees
— and arrangement is exactly the mutable component of state.

We must ask: what would happen if MAKELINK simply *stored the V-positions*? A
subsequent edit to `d_j` — an insertion before `σ_j`, a deletion, a rearrangement
— changes `Σ.M(d_j)`, displacing the very V-positions the link named. The link
would then point at whatever content drifted into those positions, or at nothing.
Nelson's survivability requirement — "if any of the bytes are left to which a link
is attached, that link remains on them" — would fail. So storing positions cannot
be right.

What must be true for the recorded endset to survive editing? It must reference
something that editing does *not* move. The content store is precisely that: by S0
an I-address, once allocated, denotes the same content for all time and is never
removed. Editing rearranges the V-to-I mapping; it never disturbs the I-addresses
themselves. Therefore MAKELINK must, at creation, read each named V-position
*through* its source arrangement and record the I-address it currently maps to.
This is the conversion at the heart of the operation.

We define *endset resolution* accordingly. For a spec-set `R` at state `Σ`,

> `ρ(R, Σ) = (∪ j : 1 ≤ j ≤ p : { Σ.M(d_j)(v) : v ∈ dom(Σ.M(d_j)) ∧ v ∈ ⟦σ_j⟧ })`

— the set of I-addresses to which the named, currently-active V-positions map. We
must discharge `ρ(R, Σ) ⊆ dom(Σ.C)`, and this turns on a confinement step the
ordinal-displacement precondition now supplies. Because `ℓ_j = δ(n_j, m)` acts at
depth `m = #u_j`, `u_j ⊕ ℓ_j = shift(u_j, n_j)` agrees with `u_j` on positions
`1..m−1` and differs only in the last (ASN-0034, OrdinalShift). Both endpoints of
`⟦σ_j⟧ = {t : u_j ≤ t < u_j ⊕ ℓ_j}` therefore share the length-`(m−1)` prefix
`p = (u_j)_1 … (u_j)_{m−1}` (non-empty since `m ≥ 2`), so `p ≼ u_j` and
`p ≼ u_j ⊕ ℓ_j`; by T5 (ContiguousSubtrees, ASN-0034) every `t` with
`u_j ≤ t ≤ u_j ⊕ ℓ_j` satisfies `p ≼ t`, hence in particular every `t ∈ ⟦σ_j⟧` has
`t₁ = (u_j)₁ = s_C`. Thus every `v ∈ ⟦σ_j⟧` — a fortiori every active such
`v ∈ dom(Σ.M(d_j))` — has `subspace(v) = s_C`. Generalized referential integrity
(S3★, ASN-0047) discharges containment on exactly these content-subspace positions
(`subspace(v) = s_C ⟹ Σ.M(d_j)(v) ∈ dom(Σ.C)`), giving `ρ(R, Σ) ⊆ dom(Σ.C)`: every
recovered address is real content. This is ASN-0058's `resolve` lifted to a spec-set,
with the correspondence scoped to `resolve`'s definedness domain: writing
`resolve(d_j, σ_j)` for that ASN's recovery of the I-address runs under `σ_j`, the
function is defined only when `(d_j, σ_j)` is a *ContentReference* — which requires,
beyond T12, the non-empty subspace `V_{(u_j)₁}(d_j) ≠ ∅` and the depth match
`#u_j = m` against the common arrangement depth, both of which `wf` above declines
to impose — and *well-formed*, with every depth-`m` position of `⟦σ_j⟧` active in
`d_j`'s arrangement. On that domain the two agree, and the agreement is two
short steps. First, the domains coincide by the definition of restriction:
`resolve(d_j, σ_j)` decomposes `f = M(d_j)|⟦σ_j⟧` (ASN-0058, Resolution), and
`dom(f) = {v ∈ dom(Σ.M(d_j)) : v ∈ ⟦σ_j⟧}` is exactly the active-filtered set
whose images `ρ` collects for spec `j` — that contribution is `ran(f)`.
Second, the I-addresses `resolve(d_j, σ_j)`'s runs name are exactly `ran(f)`,
both inclusions from ASN-0058's decomposition conditions. Each run
`(v_j, a_j, n_j)` satisfies B3 (Consistency): `f(v_j + k) = a_j + k` for
`0 ≤ k < n_j` — which both places every block position `v_j + k` in `dom(f)`
and exhibits every run-named address `a_j + k` as an image of `f`, so the
run-named set lies in `ran(f)`. Conversely, every `v ∈ dom(f)` lies in
exactly one block by B1 (Coverage), say `v = v_j + k` with `0 ≤ k < n_j`,
whence `f(v) = a_j + k` by B3 is run-named, so `ran(f)` lies in the
run-named set. Hence `ρ`'s contribution for spec `j`
is exactly the set of I-addresses `resolve(d_j, σ_j)`'s runs name. Off it `resolve`
has no value, and `ρ` extends it along independent axes: the active-position filter
(`v ∈ dom(Σ.M(d_j))`) resolves *partial* spans — MAKELINK must accept a span some of
whose positions have since been deleted — and the same filter gives depth-mismatched
and empty-subspace specs a value (possibly `∅`) where `resolve` is not defined at
all.

The resolved set is then packaged as an endset. The postcondition pins each
stored endset's *coverage* exactly, while leaving its *span decomposition* free:
the operation stores some `e_j ∈ Endset` whose spans are canonical — each of the
form `(s, δ(n, #s))` with `s ∈ ρ(R_j, Σ)` and `n ≥ 1` — and whose coverage traces
the resolved set exactly on `F`, ASN-0098's substrate-emittable address set (the
tumblers of structural form `[d.0.s.k]` with `d` a T4-valid document-level
tumbler, `s ∈ {s_C, s_L}`, `k ≥ 1`; LP-Sub confines `dom(Σ.C) ∪ dom(Σ.L) ⊆ F` at
every reachable state):

> `coverage(e_j) ∩ F = ρ(R_j, Σ)`  (the *recovery equation*).

We must say why the trace is taken on `F` rather than on the current store. A
store-trace condition `coverage(e_j) ∩ dom(Σ.C) = ρ(R_j, Σ)` would leave coverage
unpinned at the *unallocated frontier* of a content chain: with
`ρ(R_j, Σ) = {a₁}` and `a₂ = inc(a₁, 0)` not yet allocated, the record
`{(a₁, δ(2, #a₁))}` meets the current content store in `{a₁}` alone yet covers
`a₂ ∈ F`; were it admitted, a later `K.α` allocating `a₂` and a later `K.μ⁺`
arranging it into some `d''` would make the link discoverable from `d''` (LP12,
ASN-0098) through content never resolved at creation. The recovery equation
forbids exactly this: every `F`-candidate a span covers must itself be resolved,
whether allocated or not. (Interior over-reach — a span absorbing an
allocated-but-unresolved `a ∈ dom(Σ.C) ∖ ρ(R_j, Σ)` — is excluded by the store
trace already; the one leak was the frontier, where store and `F` diverge, and
the `F`-trace closes it.)

The reference decomposition records each I-address `aₖ ∈ ρ(R_j, Σ)` by its
unit-depth span `(aₖ, δ(1, #aₖ))`, whose denotation is the subtree
`{t : aₖ ≼ t}` (ASN-0043, PrefixSpanCoverage) and whose `F`-trace is its root
alone, `F ∩ ⟦(aₖ, δ(1, #aₖ))⟧ = {aₖ}` (ASN-0098, LP-Fin Corollary at `n = 1`,
applicable since `aₖ ∈ dom(Σ.C) ⊆ F`) — so the reference decomposition satisfies
the recovery equation. A decomposition that merges a run of chain-adjacent
*resolved* addresses `a₁, …, aₙ` (each `aₖ₊₁ = inc(aₖ, 0)` and each
`aₖ ∈ ρ(R_j, Σ)`) into one wider canonical span `(a₁, δ(n, #a₁))` has the same
coverage and is an equally legal record; and the recovery equation is what
restricts merging to runs *all of whose chain addresses are resolved*, since the
wider span's `F`-trace is the whole run, `F ∩ ⟦(a₁, δ(n, #a₁))⟧ = {a₁, …, aₙ}`
(LP-Fin Corollary), and one unresolved member — skipped or frontier — already
violates the equation. The coverage equality is a span merge, not a store-trace
fact. We adopt, once for this ASN, the convention `shift(t, 0) := t` —
ASN-0034's OrdinalShift is defined only for amounts `≥ 1`, and we extend it at
zero exactly as ASN-0036 (S8) and ASN-0058 (OrdinalShiftBase) do. Each chain
address is T4-valid — `aₖ ∈ ρ(R_j, Σ) ⊆ dom(Σ.C)`, and every content-store
entry is T4-valid (StoreT4Validity, ASN-0093) — so `sig(aₖ) = #aₖ`
(TA5-SigValid, ASN-0034) and the
sibling step is exactly the last-component shift,
`aₖ₊₁ = inc(aₖ, 0) = shift(aₖ, 1)`; hence each unit span's reach `shift(aₖ, 1)`
equals the next span's start — consecutive unit spans are *adjacent*,
level-uniform, at one common length (`#aₖ₊₁ = #aₖ`, TA5(c)). Along the run,
`aₖ = shift(a₁, k−1)` for every `1 ≤ k ≤ n`: the case `k = 1` is the
convention, the case `k = 2` is the sibling step itself (`a₂ = shift(a₁, 1)`,
no composition needed), and each step from `k ≥ 2` to `k + 1` composes by TS3
(ShiftComposition, ASN-0034, both amounts `k − 1 ≥ 1` and `1` in its domain):
`aₖ₊₁ = shift(shift(a₁, k−1), 1) = shift(a₁, k)`. ASN-0053's merge (S3),
applied inductively along the run — at the step absorbing unit span `k + 1`,
the accumulated prefix `(a₁, δ(k, #a₁))` reaches
`a₁ ⊕ δ(k, #a₁) = shift(a₁, k)`, which is the next start `aₖ₊₁` —
concatenates the half-open intervals exactly:
`(∪ k : 1 ≤ k ≤ n : ⟦(aₖ, δ(1, #aₖ))⟧) = ⟦(a₁, δ(n, #a₁))⟧`. The endset is a finite set of such
spans — finite because `ρ(R_j, Σ)` is finite (`p` is finite and each
`dom(Σ.M(d_j))` is finite by S8-fin, ASN-0036) — which discharges K.λ's
`e_j ∈ Endset = 𝒫_fin(Span)` precondition.

The recovery equation pins coverage *extensionally*: every admissible record
satisfies

> `coverage(e_j) = (∪ a : a ∈ ρ(R_j, Σ) : {t : a ≼ t})`.

For each canonical span `(s, δ(n, #s))` of `e_j`, the `F`-trace
`{shift(s, k) : 0 ≤ k < n}` (LP-Fin Corollary, applicable since
`s ∈ ρ(R_j, Σ) ⊆ dom(Σ.C) ⊆ F`) lies inside `coverage(e_j) ∩ F = ρ(R_j, Σ)`.
Before the merge identity can be read right to left, the trace — which arrives
in shift-form — must be identified as a sibling chain run: each trace member
lies in `ρ(R_j, Σ) ⊆ dom(Σ.C)`, hence is T4-valid (StoreT4Validity, ASN-0093)
with `sig = #` (TA5-SigValid, ASN-0034), so the sibling step coincides with
the last-component shift, `shift(shift(s, k), 1) = inc(shift(s, k), 0)` —
the same transfer the left-to-right derivation above made explicit — and the
trace is an `inc(·, 0)` chain run all of whose members are resolved. The span
is therefore the merge of a fully-resolved chain run, and its denotation —
the merge identity above, read right to left — is the union of exactly those
addresses' unit subtrees. Conversely, each `a ∈ ρ(R_j, Σ) = coverage(e_j) ∩ F`
lies in some span's interval, is by LP-Fin Corollary one of that span's chain
addresses, and so carries its whole unit subtree `{t : a ≼ t}` inside the span's
denotation. Coverage is thus a function of `ρ(R_j, Σ)` alone: any two admissible
records are coverage-equal, and the decomposition is the only freedom that
remains. Link-value equality at the model's own level (component-wise tuple
equality over endsets-as-span-sets, L6, ASN-0043) is therefore *deliberately not
pinned* by MAKELINK: for chain-adjacent resolved `a₁, a₂`, the records
`{(a₁, δ(1, #a₁)), (a₂, δ(1, #a₂))}` and `{(a₁, δ(2, #a₁))}` are distinct
link values with equal coverage, and either is a legal record of the same
resolution. For `ρ(R, Σ) ≠ ∅`, no admissible representation makes `coverage(e)`
equal to `ρ(R, Σ)` itself: coverage is a union of order-convex intervals, each
carrying the whole unit subtree of its chain addresses — and a resolved
address's proper descendants lie outside `F` (LP-Fin Corollary at `n = 1`),
hence outside `ρ(R, Σ) ⊆ F` — while `ρ(R, Σ)` is a bare finite set; ASN-0053
(S7, CoveringExistence) likewise guarantees only *covering*,
`coverage(e) ⊇ ρ(R, Σ)` (here a direct consequence of the recovery equation),
and the containment is strict whenever any address is resolved. The boundary
`ρ(R_j, Σ) = ∅` is the one case of exact equality; we settle it here. The
span-shape clause admits no span at all — every span requires a root
`s ∈ ρ(R_j, Σ)` — so `e_j = ∅` is the unique admissible record, and the
recovery equation holds with both sides empty, `coverage(∅) = ∅ = ρ(R_j, Σ)`.
For the from and to slots the boundary is *admitted*: the operation's
enabling condition `enabled` (MLop) constrains only the type slot's
resolution, and `wf` does not require a spec to capture any active position
(nor even `p ≥ 1`) — a well-formed spec all of whose positions have since
been deleted, or whose interval contains no active position at all
(depth-mismatched, or beyond the arrangement's extent; the interval itself
is never empty — T12 puts `u_j ∈ ⟦σ_j⟧` by TA-strict), resolves empty, and
the operation is defined on it. The empty record is legal: `K.λ`'s value
precondition constrains only slot 3 (`e₃ ≠ ∅`, L3; the non-type slots are
unconstrained), so `(∅, e₂, e₃)` and `(e₁, ∅, e₃)` are both legal link
values. And the empty slot is *inert* in ML9's discoverability test: with
`ρ(R_j, Σ) = ∅` the conjunct `ρ(R_j, Σ) ∩ ran(Σ.M(d')) ≠ ∅` is false at
every `d'`, so the empty slot never witnesses the existential, and the link
is discoverable only through its populated endsets. What the empty non-type
endset *means* for the link's connection — beyond the definedness, legality,
and inertness settled here — is deferred to the first Open Question.

The covering surplus — the tumblers lying in a resolved address's subtree but
strictly below it, `coverage(e_j) ∖ ρ(R_j, Σ)` under the extensional form — is
never a store address. The chain is three cited facts: by PrefixSpanCoverage
(ASN-0043), each resolved `aₖ`'s subtree is a unit span's denotation,
`{t : aₖ ≼ t} = ⟦(aₖ, δ(1, #aₖ))⟧`; by LP-Fin Corollary at `n = 1` (ASN-0098,
applicable since `aₖ ∈ ρ(R_j, Σ) ⊆ dom(Σ.C) ⊆ F` via LP-Sub),
`F ∩ {t : aₖ ≼ t} = {aₖ}`; and by LP-Sub, `dom(Σ.C) ∪ dom(Σ.L) ⊆ F`. Hence no
proper descendant of a resolved address is a store address. Two consequences
follow. First, the
store trace is exact at the creating state:
`coverage(e_j) ∩ dom(Σ.C) = ρ(R_j, Σ)`, by set algebra from the recovery
equation and `dom(Σ.C) ⊆ F` (LP-Sub). Second, the record is *tight at `Σ`* in
ASN-0098's sense: every span is canonical with its start in the store
(`s ∈ ρ(R_j, Σ) ⊆ dom(Σ.C)`), and every `F`-candidate inside a span lies in
`ρ(R_j, Σ) ⊆ dom(Σ.C)` by the recovery equation. Tightness is what makes the
trace *stable*: by LP19a (TightFreshness, ASN-0098), no address freshly
allocated at any later state ever enters `coverage(e_j)`, so
`coverage(e_j) ∩ dom(Σ''.C) = ρ(R_j, Σ)` at every `Σ''` with `Σ →* Σ''` — the
record can never come to cover content that did not exist when the link was
made. We name this
**ML1 (EndsetResolution)**: each endset
argument is recorded as I-addresses recovered by reading the source arrangement at
creation time, so the stored endset references content by identity, not by position;
its coverage traces `F` in exactly the resolved set — extensionally,
`coverage(e_j) = (∪ a : a ∈ ρ(R_j, Σ) : {t : a ≼ t})` — and hence meets the
content store, now and at every later state, in exactly the resolved set.

A single V-span may resolve to *several non-contiguous* I-address runs: if the
source document's span covers content transcluded from two origins, the two runs
carry different I-addresses and cannot be merged into one contiguous span (ASN-0058,
M16 CrossOriginMergeImpossibility). ML1's recovery equation
`coverage(e_j) ∩ F = ρ(R_j, Σ)` holds whatever this contiguity structure.
The decomposition *is* observable, and through the model's own accessors. Link
equality is component-wise tuple equality over span-sets (L6, ASN-0043), so two
coverage-equal decompositions are distinct link values; and span access by
membership — the one accessor L5 grants (ASN-0043) — is itself
decomposition-sensitive: for chain-adjacent resolved `a₁, a₂`, the test
`(a₁, δ(2, #a₁)) ∈ e` holds of `{(a₁, δ(2, #a₁))}` and fails of
`{(a₁, δ(1, #a₁)), (a₂, δ(1, #a₂))}`. A read-back of the stored value exposes
it too: ASN-0086's `Observe_K` returns the raw `(a, F, G)` triples even though
its *matching* is coverage-based. The representation independence we may claim is
therefore scoped: the observables this ASN's claims consult — projection
(ASN-0098, LP21), type matching (L8), discoverability (LP12, ML9 below) — are
each a function of coverage alone. We name this **ML2
(RepresentationIndependence)**: the stored record's *coverage* is pinned — all
admissible records are coverage-equal, the extensional form being a function of
`ρ(R_j, Σ)` alone — while its span decomposition is the one residual freedom:
observable via endset membership (L5) and value equality (L6), invisible to the
coverage-determined observables this ASN consults.

The same resolution applies *uniformly* to all three endset arguments — from, to,
and type are read through their sources by one procedure, with no slot privileged
at the conversion step (**ML3, UniformResolution**).

> *Implementation note.* Gregory's CREATELINK realizes ρ in `vspanset2sporglset`,
> which calls `permute` to walk each source document's arrangement and emit one
> *sporgl* — an `(I-origin, I-width, source-doc)` triple — per POOM crum
> overlapping the query span (Q12, Q13). The result is stored, never the input
> V-positions. The decomposition is deterministic given the enfilade state but
> *not* canonical: the same resolved I-address set is stored under different
> sporgl fragmentations depending on insertion history, prior rearrangement, and
> input span structure — the concrete counterpart of ML2's coverage-equivalence.
> Each emitted sporgl's I-span is moreover confined to currently-allocated
> istream addresses: crum I-widths are byte-exact at insertion, crum merging
> (`isanextensionnd`) requires exact adjacency, and the retrieval path
> (`retrieverestricted` → `context2span`) only clips — it never rounds up to
> crum boundaries or reaches into the unallocated permascroll frontier. The
> concrete records thus satisfy the recovery equation's `F`-trace, not merely
> the store trace.

## The link's identity

With the endsets resolved, MAKELINK must mint the link's identity. The home
document `d` names which document owns the link, and the link's address is
allocated under `d`'s prefix: `a` is the fresh emission of `A_L(d)` (ASN-0093,
K.λ). Three properties of this address are abstract and load-bearing.

It is *fresh*: `a ∉ dom(Σ.L)` at the creating state (FirstEmissionFreshness /
SubsequentEmissionFreshness, ASN-0093). It is *home-scoped*: `home(a) = d`, by the
sub-allocator's construction (the address extends `[d.0.s_L]`). And it is
*permanent and never reused*: by GlobalUniqueness (ASN-0034) no other allocation
event in the system ever produces `a`; by allocation permanence (T8, ASN-0034) `a`
is never removed from the allocated set; and by L12 (ASN-0043) the value `Σ.L(a)`
is fixed for all time once written. We name this **ML0 (IdentityAllocation)**: the
link's identity is a fresh, permanent, never-reused link-subspace address
allocated under the home document.

A corollary settles whether two MAKELINK calls with identical endset arguments and
identical home could *coalesce* into one link: they cannot. Each call draws a fresh
emission of `A_L(d)` (SubsequentEmissionFreshness, ASN-0093), so the second call's
address differs from the first's already-allocated one; value-coincidence of the
recorded triples is permitted (L11b NonInjectivity, ASN-0043), but
address-coincidence is excluded. Distinct identities are therefore *guaranteed*, not
contingent — the never-reuse half of ML0 is exactly what forbids coalescing.

Nelson's premise that a link's home "does not change" is now a theorem rather than
an assumption: the home fields `N(a).0.U(a).0.D(a)` are the leftmost components of
the link's *own* address, fixed at allocation, and no operation rewrites an
address. Residence is built into identity; there is no mutable home attribute that
could drift.

> *Implementation note.* Gregory allocates the link orgl at `docISA.0.2.N`,
> independently per home document, the counter advancing monotonically (Q11). The
> "shift" that `findnextlinkvsa` could in principle perform is structurally a no-op
> because links are appended at the document's V-extent (Q17) — abstractly just
> ML0's freshness.

## Residence, and its independence from what the link connects

Where does the link reside? In two senses, both recorded by MAKELINK. The link
*object* enters the link store, `Σ'.L = Σ.L ∪ {a ↦ (e₁, e₂, e₃)}`. And the link
*reference* enters the home document's arrangement in the link subspace, via
`K.μ⁺_L` (ASN-0047): a fresh link-subspace V-position `v_a` of `d` is bound to `a`,
making the link a member of `d`'s V-stream (so `d`'s owner can enumerate the links
it homes). The elementary precondition of `K.μ⁺_L` is discharged at the intermediate
state left by `K.λ`: there `a ∈ dom(Σ.L)` with `origin(a) = home(a) = d` (ML0).
For `a ∉ ran(M(d))`: `K.λ`'s frame leaves `M` untouched, so `ran(M(d))` at the
intermediate state is the pre-state range, and by S3★-aux (ASN-0047) every image
is a link-subspace one or a content-subspace one, each branch closed separately.
The link-subspace images lie in `dom(Σ.L)` (S3★/CL-OWN, ASN-0047), and
`a ∉ dom(Σ.L)` at the pre-state (freshness, ML0). The content-subspace images lie
in `dom(Σ.C)` (S3★), and `a ∉ dom(Σ.C)` too — `K.λ`'s freshness is against the
whole store, `a ∉ dom(Σ.C) ∪ dom(Σ.L)` (FirstEmissionFreshness /
SubsequentEmissionFreshness, ASN-0093), and `dom(Σ.C)` is unchanged by `K.λ`.
Neither branch admits `a`, so `a ∉ ran(M(d))`. Finally, the bound V-position
`v_a`: its two-branch determination — fully substrate-determined when
`V_{s_L}(d) ≠ ∅`, and carrying the one parameter the substrate leaves to its
caller, the first-link depth, when `V_{s_L}(d) = ∅` — meets `K.μ⁺_L`'s
required form by construction.
The home document is thereby the link's residence and the locus of its
ownership.

Now the orthogonality. The precondition imposes no constraint relating `d` to
any `ρ(R_j, Σ)`: the address `a` extends `d`'s prefix, while the resolved sets
are finite subsets of `dom(Σ.C)` determined solely by the source arrangements.
The substantive consequence: all three resolved sets may be disjoint from
everything under `d`'s prefix — a link living in document C that connects
regions of A and B, touching nothing in C. We name
this **ML4 (ResidenceApplicationOrthogonality)**: a link's home document and the
content its endsets reference are independent; connecting two documents never
forces the link to live inside either, and a link need not point anywhere in its
home. The home determines *ownership*; the endsets determine *connection*; the two
are separate coordinates. This is the invariant that makes annotation possible — a
reader comments on another's published document by homing a link at *her own*
address whose endsets reach into *his* content, modifying nothing of his.

## Three endsets: directionality, typing, and relation versus connection

MAKELINK records the endsets as an *ordered* triple, and the order carries
meaning. By slot distinction (ASN-0043, L6), the recorded link is positionally
addressable, and `(F, G, Θ) ≠ (G, F, Θ)` whenever `F ≠ G`: the from-side and the
to-side are a *stable, recoverable* distinction. We name this **ML5
(OrderedEndsets)**: MAKELINK preserves which region plays which role.

But what kind of distinction is it? Two readings are possible — a *semantic*
direction (this end is "from," that end is "to," for the user to interpret) or a
*traversal* restriction (the link may be followed only from→to). Two facts force
the first reading and forbid the second. Nelson: "what 'from' and 'to'
mean depend on the specific case" — the system attaches no universal semantics —
and the discoverability invariant below (ML9, via LP12, ASN-0098) indexes *every*
endset symmetrically, so a
reader at the to-side finds the link as readily as one at the from-side. The
ordering is therefore a labeling the user may rely on, not a one-way valve. We
record this as the directionality half of ML5: the recorded order fixes roles
without restricting reachability. The design does contemplate a degenerate
*one-sided* link — one with no second region. Nelson: "since it has only one
side, we use the first endset to designate the matter pointed at. To call
this 'from' is inane" (LM 4/48). The slot convention this states — populate
the first endset, leave the second empty — is recorded here as informative
Nelson usage, not enforced: no precondition of the operation carries it, and
both degenerate forms `(∅, e₂, e₃)` and `(e₁, ∅, e₃)` are legal link values.
In this model the one-sided link is exactly the empty-resolution boundary
case.

The third endset reveals the difference between a *connection* and a *relation*.
A link with only from and to asserts that two regions are tied together — a bare
connection. The third endset is a *type*: a classifying address-set that says in
*what way* they are tied. The substrate transition `K.λ` requires a non-empty type
endset (`e₃ ≠ ∅`, L3, ASN-0093). Since the type argument is `ρ`-resolved like the
others (ML3), MAKELINK must carry this as an *operation precondition on its type
argument*: the supplied type spec must resolve non-empty,

> `ρ(R₃, Σ) ≠ ∅`  (equivalently, `R₃` names at least one currently-active V-position).

A type spec whose V-positions are all inactive — content deleted, or a document
opened that never held the type content — resolves to `∅`; on such input the
operation is *undefined* and must be rejected before `K.λ` is attempted, since
at the boundary `e₃ = ∅` is the unique admissible record (the recovery
equation's span-shape clause) and an empty `e₃` violates L3. The precondition
is also *sufficient* for L3's type clause, by a one-step discharge through the
recovery equation: the stored record satisfies
`coverage(e₃) ∩ F = ρ(R₃, Σ) ≠ ∅`, so `coverage(e₃) ≠ ∅`; since
`coverage(∅) = ∅`, it follows that `e₃ ≠ ∅`. Necessity and sufficiency
together make `ρ(R₃, Σ) ≠ ∅` exactly the operation precondition that L3
induces on the type argument. With the precondition met, every link
MAKELINK creates carries a classifier, and by L8
(TypeByAddress) the type is matched by the *addresses* its endset covers, not by
any content stored there. The type argument is `ρ`-resolved exactly as from and to
(ML3), so it resolves to stored content like any other endset
(`ρ(R₃, Σ) ⊆ dom(Σ.C)`). What L8 buys here is narrower: type
*matching* compares addresses, so two links share a type when their type endsets
cover the same addresses, whatever content sits there. We name this **ML6
(TypedRelation)**: the third endset, recorded identically to from and to but read
as a classifier by address, is what distinguishes a typed relation from an untyped
connection. The structural cost of typing is one more I-address endset; the
semantic gain is the difference between "these are linked" and "these are linked
*thus*."

> *Implementation note.* Gregory's CREATELINK does *not* enforce the non-empty
> type precondition: an empty type sporgl set resolves to `NULL`, passes its
> insertion guards silently, and a link is stored with no type endset at all.
> The from and to slots take the same silent path — a well-formed V-span whose
> positions have all been deleted resolves to an empty sporgl set
> (`vspanset2sporglset` appends nothing and signals success), every insertion
> loop in `insertpm` and `insertspanf` is skipped, and the call succeeds —
> concretely confirming the admitted empty-resolution boundary for the
> non-type slots; the type slot is the one place where implementation
> (store silently) and specification (reject) part ways.

A *restriction* follows directly. Since all three arguments are `ρ`-resolved from
content-subspace V-specs (ML1, ML3), `ρ(R_i, Σ) ⊆ dom(Σ.C)` for every endset, so
MAKELINK-via-V-specs produces *only* content-backed endsets: it creates neither a
*ghost type* (L9, TypeGhostPermission, ASN-0043) nor any ghost or foreign endset
(the full generality of L4, EndsetGenerality, ASN-0043). Reaching addresses outside
the content store requires supplying an I-address as the endset argument directly,
bypassing V-span resolution — a distinct argument shape, out of scope here.

## The invariants MAKELINK preserves

We collect the guarantees, each now a consequence of how the operation records.

**Permanence (ML7).** The link's address persists and its value is fixed: for
every later transition `Σ' → Σ''`, `a ∈ dom(Σ''.L)` and `Σ''.L(a) = Σ'.L(a)`
(L12, ASN-0043). The link, once made, is not unmade by any editing of the content
it connects — because editing touches `Σ.M`, and the link lives in `Σ.L`. The
guarantee is unconditional: the transition vocabulary contains no operation that
removes a link address or rewrites its value.

**Endset survivability (ML8).** ML7 already fixes the recorded value `Σ'.L(a)` for
all time, and ML1 fixes its content-coverage as `ρ(R_j, Σ)`. What ML8 adds is the
*consequence* those two buy together: the endset reference *survives editing of the
content it names*. Editing a source document changes `Σ.M(d_j)` but never the
I-addresses already recorded in `Σ.L(a)`. Suppose content is later inserted before,
deleted from, or rearranged around a referenced region: the V-positions move, but
the stored I-addresses do not, and by S0 those I-addresses still denote their
original content. So the endset remains valid as long as any of its content
survives — which, for published content that S0 keeps forever, is always. This is
exactly Nelson's survivability, and it is *bought* by the V→I conversion of ML1:
had MAKELINK stored positions, value-fixity of the record (ML7) would not yield
survival of the *reference*. Survivability is thus not a fresh invariant on the
store but the payoff of recording at content identity rather than position.

**Residence-independence of discoverability (ML9).** This is the operation's
sharpest guarantee, and it follows by a short weakest-precondition argument. Take
as postcondition that the new link is discoverable from a document `d'` — meaning,
in the abstract characterization of ASN-0098 (LP12),

> `discoverable_from(a, d', Σ') ⟺ (E i : 1 ≤ i ≤ 3 : coverage(Σ'.L(a).eᵢ) ∩ ran(Σ'.M(d')) ≠ ∅)`.

Since MAKELINK sets `Σ'.L(a) = (e₁, e₂, e₃)`, the right-hand side reduces in two
steps.

*Fact (a) — the coverage/`ρ` gap collapses on the post-state store.* By
generalized referential integrity paired with subspace exhaustiveness (S3★
with S3★-aux, ASN-0047) at the post-state — S3★-aux makes every V-position's
subspace `s_C` or `s_L`, and S3★ closes each branch into the corresponding
store, the same pairing the `a ∉ ran(M(d))` discharge above already used — an
arrangement's images lie in the post-state store:
`ran(Σ'.M(d')) ⊆ dom(Σ'.C) ∪ dom(Σ'.L)`, and that store is
`dom(Σ.C) ∪ dom(Σ.L) ∪ {a}` (`Σ'.C = Σ.C` by ML10;
`dom(Σ'.L) = dom(Σ.L) ∪ {a}`). The intersection
`coverage(eᵢ) ∩ ran(Σ'.M(d'))` therefore consults `coverage(eᵢ)` only at
post-state store addresses, so what we need is `coverage(eᵢ)`'s trace on that
store — and that trace is exactly the resolved set,
`coverage(eᵢ) ∩ (dom(Σ'.C) ∪ dom(Σ'.L)) = ρ(R_i, Σ)`. The content half is an
application: `dom(Σ'.C) = dom(Σ.C)` (ML10), so ML1's store-trace exactness
gives `coverage(eᵢ) ∩ dom(Σ'.C) = ρ(R_i, Σ)`. The link half
is empty, `coverage(eᵢ) ∩ dom(Σ'.L) = ∅` — and here the fresh `a` is disposed of
together with every pre-existing link address: each `eᵢ` is a union of canonical
spans `(s, δ(n, #s))` rooted at resolved content addresses
`s ∈ ρ(R_i, Σ) ⊆ dom(Σ.C)` (of form `[d.0.s_C.k]`), so by ASN-0098 (LP-Fin
Corollary) every F-address in `coverage(eᵢ)` carries `subspace_I = s_C`; every
element of `dom(Σ'.L)` lies in `F` (LP-Sub at `Σ'`) and carries
`subspace_I = s_L` (L0, ASN-0093), and `s_C ≠ s_L` — so no covered tumbler is a
link address, and in particular `a ∉ coverage(eᵢ)`. The covering surplus — the
non-store descendants in `coverage(eᵢ)` (PrefixSpanCoverage; LP-Fin Corollary at
`n = 1`; LP-Sub) — cannot meet an arrangement range and so drops out. Hence
`coverage(eᵢ) ∩ ran(Σ'.M(d')) = ρ(R_i, Σ) ∩ ran(Σ'.M(d'))`.

*Fact (b) — the post-state range equals the pre-state range for the test.* For
`d' ≠ d`, `K.μ⁺_L` touches only the home document's arrangement, so
`Σ'.M(d') = Σ.M(d')` and `ran(Σ'.M(d')) = ran(Σ.M(d'))`. For the boundary case
`d' = d` — the home document itself, exactly the case ML4 highlights — `K.μ⁺_L`
extends the arrangement by the single binding `v_a ↦ a`, so
`ran(Σ'.M(d)) = ran(Σ.M(d)) ∪ {a}`. But Fact (a) established
`a ∉ coverage(eᵢ)`, hence also `a ∉ ρ(R_i, Σ)` (a subset of `coverage(eᵢ)`,
ML1), so the added point is inert on both sides of Fact (a)'s equation:
`coverage(eᵢ) ∩ ran(Σ'.M(d)) = coverage(eᵢ) ∩ ran(Σ.M(d))` and
`ρ(R_i, Σ) ∩ ran(Σ'.M(d)) = ρ(R_i, Σ) ∩ ran(Σ.M(d))`. In both cases the test
reads against the pre-state range.

Composing (a) and (b) — and conjoining the operation's definedness, since
`makelink` is *partial* (ML0 requires `d ∈ dom(Σ.M)`, ML6 requires
`ρ(R₃, Σ) ≠ ∅`) and the postcondition `discoverable_from(a, d', ·)` is itself
defined only for `d' ∈ dom(Σ.M)`,

> `wp(makelink(d, R₁, R₂, R₃), discoverable_from(a, d', ·))`
> `≡ enabled(makelink(d, R₁, R₂, R₃)) ∧ d' ∈ dom(Σ.M) ∧ (E i : 1 ≤ i ≤ 3 : ρ(R_i, Σ) ∩ ran(Σ.M(d')) ≠ ∅)`,

where `enabled(makelink(d, R₁, R₂, R₃))` is the operation's enabling
precondition. The `wf` and `ρ(R₃, Σ) ≠ ∅` conjuncts
remain essential: without them the formula would assert the postcondition reachable
on inputs the operation rejects — e.g. an endset spec that escapes the content
subspace, or an empty type spec (ML6).

Beyond the operation's own enabledness, the home document `d` does not appear in
the discoverability test on the right. The condition for finding the
link from `d'` is solely that `d'`'s arrangement reaches one of the I-addresses the
link recorded — and that holds for *any* document sharing the content, the home
document among them only incidentally. Residence fixes ownership; it imposes no
restriction whatever on discoverability scope. Because the criterion is symmetric
across all three endsets, the link is reachable from the from-regions, the
to-regions, and the type-regions alike. We name this **ML9
(DiscoverabilityDecoupledFromResidence)**: MAKELINK makes the link discoverable
from every content region any of its endsets references, independently of where
the link resides. The consequence extends to every later state `Σ''` with
`Σ' →* Σ''`, on two premises stated uniformly. The content half is ML1's
stable store trace: `coverage(eᵢ) ∩ dom(Σ''.C) = ρ(R_i, Σ)` at every `Σ''`
(tightness with LP19a). The link half is Fact (a)'s subspace exclusion, which
is state-uniform: every `F`-address in `coverage(eᵢ)` carries
`subspace_I = s_C` — a property of the fixed endset value alone (L12 fixes
`eᵢ`; LP-Fin Corollary fixes its `F`-trace) — while every element of
`dom(Σ''.L)` lies in `F` and carries `subspace_I = s_L` (LP-Sub and L0,
invariants of every reachable state), so
`coverage(eᵢ) ∩ dom(Σ''.L) = ∅` at every `Σ''`; independently, any address
freshly allocated after creation is excluded from `coverage(eᵢ)` by LP19a.
Since discoverability at `Σ''` consults `coverage(eᵢ) ∩ ran(Σ''.M(d''))`
(LP12) and that range lies in `dom(Σ''.C) ∪ dom(Σ''.L)` (S3★ with S3★-aux,
as in Fact (a)), the two halves
together yield the future-state consequence: the link can become discoverable
from a new document only by that document's arrangement reaching the
*originally resolved* content.

Note what discharges ML9: *nothing beyond recording the endsets as I-addresses in
the store.* The discoverability is not a separate indexing action MAKELINK must
remember to perform; it is the standing meaning of having content-identity endsets
present in `Σ.L`.

> *Implementation note.* Gregory's spanfilade is the concrete index that realizes
> this biconditional, keyed by I-address with the home dimension explicitly nulled
> out (Q14, Q20) — the implementation's way of guaranteeing exactly that home
> plays no role.

**Frame (ML10).** MAKELINK allocates no content and edits no other document:
`Σ'.C = Σ.C` (the operation reads source arrangements, it does not write content),
and `Σ'.M(d') = Σ.M(d')` for every `d' ≠ d` (only the home document's link
subspace is extended). The entity set and provenance relation are likewise
untouched: `Σ'.E = Σ.E ∧ Σ'.R = Σ.R`, inherited from the frames of `K.λ` and
`K.μ⁺_L` (ASN-0047) — MAKELINK creates no entity and records no provenance.
Existing link-store entries are
untouched — the store only gains `a ↦ (e₁, e₂, e₃)`. The coordinates the endsets
read — every source's *content-subspace* arrangement — are unmodified by the act
of being linked into: the one arrangement change anywhere is the home's
link-subspace seating `v_a ↦ a`, so a source `d_j ≠ d` is untouched outright,
and a source coinciding with the home (`d_j = d`, which `wf` admits and ML4 and
L4(b) (ASN-0043) anticipate) gains only that link-subspace binding, its
content-subspace restriction unchanged. A link *to* a region changes nothing
about that region, which is why links can be made to published material one
does not own.

**MLop (MakelinkOperation).** `makelink(d, R₁, R₂, R₃)` is a partial
operation on reachable states, defined exactly on its enabling precondition

> `enabled(makelink(d, R₁, R₂, R₃)) ≡ d ∈ dom(Σ.M) ∧ (A i : 1 ≤ i ≤ 3 : wf(R_i, Σ)) ∧ ρ(R₃, Σ) ≠ ∅`

— `wf` and `ρ` as defined in the resolution section (ML1). When enabled, the
effect is the composite `Σ →* Σ'` — a ValidComposite★ of two named elementary
steps, `K.λ` followed by `K.μ⁺_L` — and its net effect is two entries. The
link store gains exactly one,

> `Σ'.L = Σ.L ∪ {a ↦ (e₁, e₂, e₃)}`,

where `a` is the fresh emission of `A_L(d)` (ML0) and each `e_i` is an
admissible record of `ρ(R_i, Σ)` — canonical spans rooted in the resolved set,
satisfying the recovery equation `coverage(e_i) ∩ F = ρ(R_i, Σ)` (ML1), the
decomposition free per ML2. The home arrangement gains exactly one binding,

> `Σ'.M(d) = Σ.M(d) ∪ {v_a ↦ a}`,  where  `v_a = shift(max(V_{s_L}(d)), 1)` if `V_{s_L}(d) ≠ ∅`,  and  `v_a = [s_L, 1]` if `V_{s_L}(d) = ∅`.

The first branch is fully determined by the substrate (ASN-0047, K.μ⁺_L). In
the second the substrate determines `v_a` only relative to a depth:
`ValidFirstLinkPosition(d, v_a, m)` is parameterized by a *caller-chosen*
`m ≥ 2` (ASN-0047), and `makelink(d, R₁, R₂, R₃)` carries no depth argument —
so the choice falls to the operation, not to `K.μ⁺_L`. We fix it by
convention: `m = 2`, the least depth S8a admits, giving first link V-position
`v_a = [s_L, 1]`. Any fixed `m ≥ 2` would serve equally — no claim of this
ASN consults the link-subspace depth — but the operation must name one, and
the minimal choice adds no unforced structure.

The two branch selections just made are *independent*, and we must check the
contract where they decouple. The `a`-branch is store-keyed: `K.λ` chooses
first versus subsequent emission by the homed set
`{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d}` (ASN-0093). The `v_a`-branch is
arrangement-keyed: it tests `V_{s_L}(d)`. Agreement is forced in one
direction only — an empty homed set forces `V_{s_L}(d) = ∅`, since every
link-subspace image of `Σ.M(d)` lies in `dom(Σ.L)` with origin `d` (S3★ with
CL-OWN, ASN-0047), so first emission never pairs with the successor position.
The converse fails at a reachable boundary: a `K.μ⁻` on `d` with retention
`n'_{s_L} = 0` empties `V_{s_L}(d)` while every homed link persists in
`dom(Σ.L)` (P3/L12, ASN-0047) — the links are unseated, not unmade — and the
link-subspace depth ceases to be pinned (`m_L(d)` is defined only while
`V_{s_L}(d) ≠ ∅`, ASN-0047). At such a *contracted home* a single call takes
the subsequent-emission branch for `a` — `a = inc(ℓ_prev, 0)` with
`ℓ_prev = max{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d}` — together with the
first-position branch for `v_a = [s_L, 1]`, the depth re-pinned at the
convention. The mixed case satisfies the contract unchanged: `a`'s freshness
is against the whole store and never consults the arrangement
(SubsequentEmissionFreshness, ASN-0093); the residence-section discharge of
`K.μ⁺_L`'s precondition used only `a ∉ ran(M(d))`, which the contraction
only makes easier to meet (the range shrank); and the post-state singleton
`V_{s_L}(d) = {[s_L, 1]}` satisfies D-MIN★ and D-SEQ★ as any first seating
does. The contracted home is a boundary of the selectors, not of the
guarantees.

Everything else is frame,
exactly as ML10 states, and the operation returns `a`, the link's identity
(ML0). ML0–ML10 are the per-facet guarantees of this contract.

## A worked example

Fix four documents `A`, `B`, `C`, `D`, all in `dom(Σ.M)`, and create a link homed
in `C` that connects content of `A` to content of `B`, typed by content of `D` —
touching nothing in `C`. This is the annotation shape of ML4.

*Source content.* Take the three source arrangements at depth 2 in the content
subspace, where D-SEQ★ (ASN-0047) fixes the canonical position shape. In `A`,
two active V-positions: `V_{s_C}(A) = {[s_C, 1], [s_C, 2]}`, mapping to content
I-addresses `Σ.M(A)([s_C, 1]) = a₁ = A.0.s_C.1` and
`Σ.M(A)([s_C, 2]) = a₂ = A.0.s_C.2`, so `{a₁, a₂} ⊆ ran(Σ.M(A))`. In `B`, one:
`V_{s_C}(B) = {[s_C, 1]}` with `Σ.M(B)([s_C, 1]) = b₁ = B.0.s_C.1`. The type
content lives in a fourth document `D ∈ dom(Σ.M)`, `D ∉ {A, B, C}`, again with
one: `V_{s_C}(D) = {[s_C, 1]}` with `Σ.M(D)([s_C, 1]) = θ₁ = D.0.s_C.1`. The
home `C`'s arrangement reaches none of these addresses:
`{a₁, a₂, b₁, θ₁} ∩ ran(Σ.M(C)) = ∅`. Last, pin `C`'s link state: `C` homes
no links — `{ℓ' ∈ dom(Σ.L) : origin(ℓ') = C} = ∅` — whence also
`V_{s_L}(C) = ∅`, since a link-subspace image of `Σ.M(C)` would lie in
`dom(Σ.L)` with origin `C` (S3★ with CL-OWN, ASN-0047).

*Arguments.* The three spec-sets are exhibited concretely against these
arrangements — each a single V-spec over its source:

> `R₁ = ⟨(A, σ₁)⟩` with `σ₁ = ([s_C, 1], δ(2, 2))`;  `R₂ = ⟨(B, ([s_C, 1], δ(1, 2)))⟩`;  `R₃ = ⟨(D, ([s_C, 1], δ(1, 2)))⟩`;  `home = C`.

We trace `R₁`'s resolution end to end; the other two are the same computation at
width 1. *Well-formedness:* `wf(R₁, Σ)` checks its one spec — the source is
allocated (`A ∈ dom(Σ.M)`, stipulated), the start lies in the content subspace
(`subspace([s_C, 1]) = s_C`) at depth `#[s_C, 1] = 2 ≥ 2`, and the displacement
is ordinal at that depth (`ℓ₁ = δ(2, 2)` with `n₁ = 2 ≥ 1`) — so `wf(R₁, Σ)`
holds. *Interval:* `[s_C, 1] ⊕ δ(2, 2) = shift([s_C, 1], 2) = [s_C, 3]`
(OrdinalShift, ASN-0034), so `⟦σ₁⟧ = {t : [s_C, 1] ≤ t < [s_C, 3]}`.
*Active-position filter:* every member of `⟦σ₁⟧` carries subspace `s_C` (ML1's
prefix-confinement step), so the filter can admit only content-subspace
positions of `A`, whatever its link subspace holds; of `A`'s active positions,
`[s_C, 1]` and `[s_C, 2]` lie in the interval —
`[s_C, 1] ≤ [s_C, 1] < [s_C, 2] < [s_C, 3]` under T1, the last component
deciding — and `V_{s_C}(A)` holds no others, so
`dom(Σ.M(A)) ∩ ⟦σ₁⟧ = {[s_C, 1], [s_C, 2]}`. *Images:* reading the two active
positions through the arrangement,
`ρ(R₁, Σ) = {Σ.M(A)([s_C, 1]), Σ.M(A)([s_C, 2])} = {a₁, a₂}`. By the same steps,
`ρ(R₂, Σ) = {Σ.M(B)([s_C, 1])} = {b₁}` and
`ρ(R₃, Σ) = {Σ.M(D)([s_C, 1])} = {θ₁} ≠ ∅`, so the type precondition of ML6 is
met.

*Identity (ML0).* `C` homes no links (stipulated), so `K.λ`'s first-emit
predicate `{ℓ' ∈ dom(Σ.L) : origin(ℓ') = C} = ∅` fires and `A_L(C)` emits the
fresh link address `a = C.0.s_L.1` (FirstEmission, ASN-0093) — `C`'s first
link. It is fresh (`a ∉ dom(Σ.L)`, FirstEmissionFreshness), home-scoped
(`home(a) = N(a).0.U(a).0.D(a) = C`), and distinct from every content address
(`subspace_I(a) = s_L ≠ s_C`).

*Record (ML1, ML2).* `Σ'.L(a) = (e₁, e₂, e₃)` with
`e₁ = {(a₁, δ(1,#a₁)), (a₂, δ(1,#a₂))}`, `e₂ = {(b₁, δ(1,#b₁))}`,
`e₃ = {(θ₁, δ(1,#θ₁))}`. Checking ML1/ML2: `coverage(e₁) ∩ F = {a₁, a₂} =
ρ(R₁, Σ)` — each unit span's `F`-trace is its root alone (LP-Fin Corollary at
`n = 1`, ASN-0098), so the subtrees of `a₁` and `a₂` contribute no further
`F`-address — and likewise `coverage(e₂) ∩ F = {b₁}`. ML2 made concrete: `a₁` and `a₂` are
consecutive chain siblings of `A_C(A)` — `a₂ = inc(a₁, 0) = shift(a₁, 1)` by
TA5-SigValid, so the two unit spans are adjacent and their intervals concatenate
exactly (ASN-0053, S3) — hence the single wider canonical span `(a₁, δ(2, #a₁))`
has the same coverage as the two unit spans; `e₁` may legally be recorded either
way — distinct link values, coverage-equal, indistinguishable to the
coverage-determined observables ML2 scopes (projection, type matching,
discoverability), though separable by the membership test
`(a₁, δ(2, #a₁)) ∈ e₁`. The merge is legal precisely because *both* chain addresses
are resolved: had `ρ(R₁, Σ)` been `{a₁}` alone, the wider span would place `a₂`
in `coverage(e₁) ∩ F` and violate the recovery equation — whether `a₂` were
allocated-but-unresolved or still the chain's unallocated frontier.

*Seating (MLop).* `V_{s_L}(C) = ∅` (stipulated), so the second branch of
MLop's `v_a` determination fires: `v_a = [s_L, 1]`, the first link V-position
at the conventional depth `m = 2`, and the home arrangement gains exactly the
binding `Σ'.M(C) = Σ.M(C) ∪ {[s_L, 1] ↦ a}`. At `Σ'` the link subspace of `C`
is `V_{s_L}(C) = {[s_L, 1]}`, checkable directly against the shape invariants:
its minimum is `[s_L, 1]` (D-MIN★, ASN-0047) and it is the canonical initial
segment `{[s_L, k] : 1 ≤ k ≤ 1}` (D-SEQ★) — the one branch of MLop this ASN
fixes by its own convention, here verified against a concrete state. The two
selectors aligned here — first emission with first position — only because
`C` is link-virgin: at a contracted home (links homed, link subspace cleared
by a `K.μ⁻` with `n'_{s_L} = 0`) the `a`-branch reads the store and emits the
*subsequent* sibling while the `v_a`-branch fires exactly as here — MLop's
mixed case.

*Discoverability (ML9).* Evaluate `discoverable_from(a, d', Σ')` for each document.
From `A`: `coverage(e₁) ∩ ran(Σ'.M(A)) ⊇ {a₁} ≠ ∅` — discoverable. From `B`:
`coverage(e₂) ∩ ran(Σ'.M(B)) ⊇ {b₁} ≠ ∅` — discoverable. From `D`, the type
source: `coverage(e₃) ∩ ran(Σ'.M(D)) ⊇ {θ₁} ≠ ∅` — discoverable from the
type-region too, witnessing the endset symmetry of ML5/ML9. From the home `C`:
the link's endsets reference only `A`-, `B`-, and `D`-content, none of it in
`ran(Σ'.M(C))` (stipulated above); the one address `C`'s arrangement gained is `a`
itself — the seating `[s_L, 1] ↦ a` — in the link subspace and outside every
`coverage(eᵢ)`. So
`coverage(eᵢ) ∩ ran(Σ'.M(C)) = ∅` for all `i`, and the link is *not* discoverable
from its own home. This is ML9 made concrete: discovery follows the content the
endsets name (`A`, `B`, and `D`), not the residence (`C`). The link lives in `C`
yet is found from `A`, `B`, and `D` — residence and reachability are orthogonal.

*An edit, and what survives it (ML7, ML1, ML8, ML9).* Everything so far is a
creation-state check; the headline guarantee is about what happens *after*. So
we edit a source. `A`'s content arrangement is the depth-2
`V_{s_C}(A) = {[s_C, 1], [s_C, 2]}` with `[s_C, 1] ↦ a₁`, `[s_C, 2] ↦ a₂`,
fixed in the source-content stipulation above. Apply the foundation contraction
K.μ⁻ (ASN-0047) to `A` with retention `n'_{s_C} = 1` (the link subspace, if
populated, retained in full): the transition `Σ' → Σ''` restricts `A`'s
arrangement to the retained set, so `[s_C, 1] ↦ a₁` survives and
`a₂ ∉ ran(Σ''.M(A))`. Against this concrete edit:

(i) *Permanence (ML7).* K.μ⁻'s frame carries `L' = L`, so
`Σ''.L(a) = Σ'.L(a)` — the link object is untouched by the edit.

(ii) *Stable trace (ML1).* K.μ⁻'s frame also carries `C' = C`, so
`coverage(e₁) ∩ dom(Σ''.C) = {a₁, a₂} = ρ(R₁, Σ)`, exactly as at creation —
the instance at `Σ''` of the stable store trace ML1 promises at every later
state. The record still names both addresses, the deleted one included.

(iii) *Survivability (ML8), and the partial span.* The position that carried
`a₂` is gone, but the referent is not: `a₂ ∈ dom(Σ''.C)` with
`Σ''.C(a₂) = Σ.C(a₂)` (S0) — the endset reference survives the deletion of
its position because the record holds identities, not positions. And the link
remains discoverable from `A` through what survives:
`coverage(e₁) ∩ ran(Σ''.M(A)) = {a₁} ≠ ∅`, so `discoverable_from(a, A, Σ'')`
holds (LP12). Had `e₁` been recorded as the single merged span
`(a₁, δ(2, #a₁))`, that one I-span now covers one I-address still arranged
in `A` (`a₁ ∈ ran(Σ''.M(A))`) and one no longer arranged
(`a₂ ∉ ran(Σ''.M(A))`) — the I-side counterpart of the partial V-span over
a partially-deleted arrangement that motivated `ρ`'s active-position filter,
a structural parallel in different coordinates rather than the same shape —
and the link stays on the surviving content:
Nelson's "if any of the bytes are left to which a link is attached, that
link remains on them," exercised.

(iv) *The other documents (ML9).* K.μ⁻ touches only `A`'s arrangement
(`(A d' : d' ≠ A : Σ''.M(d') = Σ'.M(d'))`), so the tests at `B`, `D`, and `C`
read unchanged ranges (LP4, ASN-0098): the link is still discoverable from
`B` (via `b₁`) and `D` (via `θ₁`), and still not discoverable from its home
`C`. Editing one source moves discoverability only where the edit lands —
ML9's future-state consequence, concretely.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| ML0 | IdentityAllocation: the link's identity is a fresh (`a ∉ dom(Σ.L)`), permanent (never removed, never reused — GlobalUniqueness, T8), value-fixed (L12) link-subspace address allocated by `A_L(d)` under home `d`, with `home(a) = d` | introduced |
| ML1 | EndsetResolution: under precondition `wf(R,Σ) ≡ (A j : 1 ≤ j ≤ p : d_j ∈ dom(Σ.M) ∧ subspace(u_j) = s_C ∧ #u_j ≥ 2 ∧ (E n_j ≥ 1 : ℓ_j = δ(n_j, #u_j)))`, each endset argument `R = ⟨(d₁,σ₁),…,(d_p,σ_p)⟩` is recorded as the I-addresses `ρ(R,Σ) = (∪ j : 1 ≤ j ≤ p : {Σ.M(d_j)(v) : v ∈ dom(Σ.M(d_j)) ∧ v ∈ ⟦σ_j⟧}) ⊆ dom(Σ.C)`, read through the source arrangements at creation; the stored endset has canonical spans rooted in `ρ(R,Σ)` and satisfies the recovery equation `coverage(e_j) ∩ F = ρ(R_j,Σ)`, equivalently `coverage(e_j) = (∪ a : a ∈ ρ(R_j,Σ) : {t : a ≼ t})`; consequently `coverage(e_j) ∩ dom(Σ.C) = ρ(R_j,Σ)` and `e_j` is tight at Σ (ASN-0098), so by LP19a the content trace is stable at all later states; the boundary `ρ(R_j,Σ) = ∅` is admitted for the non-type slots, with `e_j = ∅` the unique admissible record | introduced |
| ML2 | RepresentationIndependence: the stored endset's coverage is pinned extensionally by ML1's recovery equation — all admissible records are coverage-equal — and the span decomposition is the only residual freedom; the decomposition remains observable via endset membership (L5) and value equality (L6), but the observables this ASN's claims consult — projection (LP21), type matching (L8), discoverability (LP12) — are functions of coverage alone and cannot distinguish coverage-equal records, which is why the postcondition pins coverage and leaves decomposition free | introduced |
| ML3 | UniformResolution: from, to, and type arguments are resolved by one procedure with no slot privileged at the V→I conversion step | introduced |
| ML4 | ResidenceApplicationOrthogonality: home document and endset content are independent; the precondition relates `d` to no `ρ(R_j,Σ)`; a link may home anywhere and point anywhere, connecting two documents without residing in either | introduced |
| ML5 | OrderedEndsets: the recorded triple is ordered, `(F,G,Θ) ≠ (G,F,Θ)` for `F ≠ G` (L6); the order fixes from/to roles semantically without restricting reachability (discovery is endset-symmetric); the one-sided slot convention (LM 4/48: populate the first slot, leave the second empty) is informative Nelson usage, not enforced — the operation admits both degenerate forms `(∅, e₂, e₃)` and `(e₁, ∅, e₃)` | introduced |
| ML6 | TypedRelation: operation precondition `ρ(R₃,Σ) ≠ ∅`, necessary and sufficient for K.λ's `e₃ ≠ ∅` (L3) via the recovery equation (`coverage(e₃) ∩ F = ρ(R₃,Σ)` with `coverage(∅) = ∅`); the third endset, recorded like from/to but matched by address (L8), distinguishes a typed relation from a bare connection; the type resolves to stored content like any other endset (`ρ(R₃,Σ) ⊆ dom(Σ.C)`) | introduced |
| ML7 | Permanence: `(A Σ' → Σ'' : a ∈ dom(Σ'.L) : a ∈ dom(Σ''.L) ∧ Σ''.L(a) = Σ'.L(a))` — the made link is not broken by any editing of the content it connects | introduced |
| ML8 | EndsetSurvivability: editing a source document changes `Σ.M` but never the recorded I-addresses, which by S0 denote their original content permanently — the endset reference survives all editing of the content it names (consequence of ML7 ∧ ML1) | introduced |
| ML9 | DiscoverabilityDecoupledFromResidence: `wp(makelink, discoverable_from(a, d', ·)) ≡ enabled(makelink) ∧ d' ∈ dom(Σ.M) ∧ (E i : ρ(R_i,Σ) ∩ ran(Σ.M(d')) ≠ ∅)`, where `enabled(makelink)` is the operation's enabling precondition (MLop) — beyond enabledness, the home `d` does not appear in the discoverability test; the consequence persists at every later `Σ''` via the stable content trace (LP19a) together with the state-uniform link-store exclusion `coverage(eᵢ) ∩ dom(Σ''.L) = ∅` (LP-Fin Corollary subspace `s_C` versus LP-Sub/L0 subspace `s_L`) | introduced |
| ML10 | Frame: `Σ'.C = Σ.C`; `Σ'.E = Σ.E ∧ Σ'.R = Σ.R` (inherited from the K.λ/K.μ⁺_L frames); `(A d' ≠ d : Σ'.M(d') = Σ.M(d'))`; existing `Σ.L` entries unchanged; every source's content-subspace arrangement is unmodified by being linked into — a source coinciding with the home gains only the link-subspace seating `v_a ↦ a` (`v_a` as determined in MLop) | introduced |
| MLop | MakelinkOperation (DEF, operation): signature `makelink(d, R₁, R₂, R₃)` — home document and three spec-set arguments (from, to, type), a partial operation on reachable states; precondition `enabled(makelink(d, R₁, R₂, R₃)) ≡ d ∈ dom(Σ.M) ∧ (A i : 1 ≤ i ≤ 3 : wf(R_i, Σ)) ∧ ρ(R₃, Σ) ≠ ∅`, where `wf` and `ρ` are as in ML1; effect (the composite `Σ →* Σ'`, a ValidComposite★ of elementary K.λ then K.μ⁺_L): `Σ'.L = Σ.L ∪ {a ↦ (e₁, e₂, e₃)}` with `a` the fresh emission of `A_L(d)` (ML0) and each `e_i` an admissible record of `ρ(R_i, Σ)` per ML1/ML2, plus the home seating `Σ'.M(d) = Σ.M(d) ∪ {v_a ↦ a}` with `v_a = shift(max(V_{s_L}(d)), 1)` when `V_{s_L}(d) ≠ ∅` and `v_a = [s_L, 1]` (first link V-position at the conventional depth `m = 2`) when `V_{s_L}(d) = ∅`; the `a`-branch (store-keyed: K.λ's emit predicate over `{ℓ' : origin(ℓ') = d}`) and the `v_a`-branch (arrangement-keyed: `V_{s_L}(d)`) are independent selectors, diverging at a contracted home — homed links present, `V_{s_L}(d) = ∅` after K.μ⁻ with `n'_{s_L} = 0` — where subsequent emission pairs soundly with the first position; returns `a`; frame per ML10 | introduced |

## Open Questions

What does an empty non-type endset — the admitted degenerate one-sided link — assert about the link's connection?

What must the operation guarantee when an endset argument references content in the link subspace — a link whose endset points at another link — for the resolved record to remain well-formed?
