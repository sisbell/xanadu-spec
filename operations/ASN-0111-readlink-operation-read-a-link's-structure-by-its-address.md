> **ASN-0111 · READLINK — Reading a Link's Structure by Its Own Address** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0111-readlink-operation-read-a-link's-structure-by-its-address.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0111: READLINK — Reading a Link's Structure by Its Own Address

*2026-06-10*

## The problem

A Xanadu link is not markup buried in content; it is a first-class object with its own
permanent address in the link subspace. Because it is addressable, we can name it
directly and ask the system a question that has nothing to do with where the link points:
*what relationship does this object record?* This note specifies the operation that answers
that question — the direct read of a link by its own identity. We call it `readlink`.

We must be careful to separate `readlink` from three neighbouring operations that are out
of scope here. *Following* a link takes one of its endsets and resolves it against a chosen
document's arrangement to obtain current positions. *Searching* for a link supplies content
regions and asks which links touch them. *Counting* asks how many do. All three combine the
link with something beyond it — an arrangement, or a query's spec-set. `readlink` is the operation
by which the link discloses, in full and unconditionally, the relationship it was built to hold.

## The link as a readable object

We take from the foundations the shape of a stored link. The *link store* `Σ.L : T ⇀ Link`
(ASN-0043) maps addresses to link values, where a link value is a finite sequence of endsets

> `Link = {(e₁, e₂, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`,  `Endset = 𝒫_fin(Span)`

with the standard-triple convention assigning slot 1 the *from*-endset, slot 2 the *to*-endset,
and slot 3 the *type*-endset. Each endset is a finite set of spans over the tumbler line; each
span `(s, ℓ)` denotes a contiguous region `{t : s ≤ t < s ⊕ ℓ}` (T12, ASN-0034). The relationship
a link records is therefore not a pair of points but a triple of *endsets* — each a finite set of
spans — reaching, possibly discontiguously, anywhere in the docuverse. The address-set an endset
denotes is its `coverage` (ASN-0043):

> `coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})`.

Every link address is, by the substrate invariants, an element-level, T4-valid tumbler in the
link subspace: `zeros(a) = 3`, `subspace_I(a) = s_L`, `#E(a) ≥ 2` (L0, L1, L1b, L0b of ASN-0043).
These facts are what make a link nameable in the same address space as content.

## Deriving the read

A standing precondition governs everything below: `readlink` is specified over `→*`-reachable,
invariant-satisfying states `Σ` (ASN-0047, ASN-0093). Where we write "for a state `Σ`," read "for a
reachable, invariant-satisfying `Σ`."

We are looking for an operation that, given an address, returns the relationship recorded there.
The minimal honest specification is a lookup in the link store. We must first decide the
operation's shape: a partial function gated by the precondition `a ∈ dom(Σ.L)`, after the pattern
of the allocation operations (K.α, K.λ of ASN-0093), or a total request whose contract includes
reporting absence. The deciding observation is the insufficiency of address-only tests: no
satisfiable predicate computable from the address alone is sufficient for membership in
`dom(Σ.L)` — proved below, at the structural screen. Nelson's design points the same way — the address space is sparsely occupied by design,
an address naming no stored object is a valid coordinate rather than malformed input, and the
retrieval requests are all-that-is-there questions for which "nothing" is a legitimate answer.
Gregory's implementation concurs: a retrieval at an unallocated link address is answered with the
protocol's distinguished failure reply, inside the operation's contract, not rejected as a
violation of it. We therefore make the read total, over a codomain extended by one distinguished
failure value `⊥` (with `⊥ ∉ Link`). Writing `𝒮` for the extended state space of ASN-0047, whose
members are the states `Σ = (C, L, E, M, R)`, for any address `a ∈ T` and reachable `Σ ∈ 𝒮` (the
standing precondition):

> `readlink : T × 𝒮 → Link ∪ {⊥}`
> `readlink(a, Σ) = Σ.L(a)`   when `a ∈ dom(Σ.L)`
> `readlink(a, Σ) = ⊥`        when `a ∉ dom(Σ.L)`.

It is a pure read: its frame condition is that it changes nothing, `Σ' = Σ`.

The first commitment is on the success criterion. The read returns a link value exactly on
allocated link addresses, and `⊥` — the report that no link lives at `a` — everywhere else.

**RL0 (Totality and success).** `readlink(a, Σ)` is defined for every `a ∈ T`, and

> `readlink(a, Σ) ∈ Link ⟺ a ∈ dom(Σ.L)`,   `readlink(a, Σ) = ⊥ ⟺ a ∉ dom(Σ.L)`.

Reasoning backward, we render "the result is the recorded relationship at `a`" as a guarded
formula, and dually for failure:

> `R_ok ≡ a ∈ dom(Σ.L) ∧ result = Σ.L(a)`,   `R_⊥ ≡ result = ⊥`.

In `R_ok` the left conjunct guards the dereference in the right, so both postconditions are
well-formed on every state — `Σ.L(a)` is consulted only where the guard places `a` in the domain.
The read is the assignment `result := readlink(a, Σ)`, and its wp is the textual substitution:

> `wp(result := readlink(a, Σ), R_ok)`
> `≡ a ∈ dom(Σ.L) ∧ readlink(a, Σ) = Σ.L(a)`   {substitution `result := readlink(a, Σ)`}
> `≡ a ∈ dom(Σ.L)`   {definition: on the domain, `readlink(a, Σ) = Σ.L(a)`}

and dually

> `wp(result := readlink(a, Σ), R_⊥) ≡ readlink(a, Σ) = ⊥ ≡ a ∉ dom(Σ.L)`,

the last step because off the domain the definition gives `⊥`, while on it the result is
`Σ.L(a) ∈ Link` and `⊥ ∉ Link`. The two weakest preconditions are complementary, so at each
`(a, Σ)` exactly one of the two postconditions is attainable. These are exactly RL0's two
biconditionals.

A reader holding a candidate tumbler `a ∈ T` can evaluate a *structural screen*, from the address
alone and left to right:

> `T4-valid(a) ∧ zeros(a) = 3 ∧ subspace_I(a) = s_L ∧ #E(a) ≥ 2`

The leading conjunct is decidable by direct inspection of the component sequence — at most three
zeros, none adjacent, `a₁ ≠ 0`, `a_{#a} ≠ 0` (T4, ASN-0034) — and is necessary by L0b (ASN-0043).
The first two conjuncts jointly guard the well-definedness of what follows: `subspace_I(·)` and
the element-field projection `E(·)` are defined on T4-valid tumblers with `zeros(·) = 3` and a
non-empty element field (T4b, ASN-0034; SubspaceI, ASN-0043), and that last domain condition is
itself discharged by the first two conjuncts — `T4-valid(a) ∧ zeros(a) = 3 ⟹ #E(a) ≥ 1`, since
T4a's field-segment equivalence makes every field segment of a T4-valid tumbler non-empty, and
T4b/T4c fix that the three separators delimit four fields of which the fourth is the element
field, so `E(a)` exists and `E(a)₁` is defined. Under the left-to-right reading the screen is
therefore evaluable on all of `T`, including tumblers such as
`[1, 0, 0, 2, 0, 3]` on which the later conjuncts alone would have no value. The remaining
conjuncts are necessary by L1, L0, and L1b (ASN-0043) respectively. *Every conjunct is necessary*
— so a failed screen guarantees `⊥` without an invocation. *No satisfiable address-computable predicate is sufficient* — at the initial state `Σ₀`
(ASN-0047), `dom(Σ₀.L) = ∅`, so any satisfiable address-only predicate has a witness `a` with
`a ∉ dom(Σ₀.L)`, and sufficiency fails there. Hence no caller can discharge a membership
precondition from the address alone; only the outcome — `Link` versus `⊥` — settles membership in
`dom(Σ.L)`.

## Completeness: the read returns the whole relationship

A direct read answers no query, so there is no satisfying fragment at which it could stop. It
must return the endsets *entire*.

**RL1 (Completeness).** For `a ∈ dom(Σ.L)` — the branch on which the result is a link value — each
returned endset equals the recorded one exactly, omitting nothing and introducing nothing:

> `(A i : 1 ≤ i ≤ |Σ.L(a)| : readlink(a, Σ).eᵢ = Σ.L(a).eᵢ)`.

The set equality captures both directions at once: every recorded span `(s, ℓ) ∈ Σ.L(a).eᵢ` is
returned, and no span outside `Σ.L(a).eᵢ` appears in the result. The justification is immediate from
the definition: `readlink(a, Σ) = Σ.L(a)` componentwise. The codomain provides no enforcement here:
`Link` is closed under shrinking a connective slot (for `(F, G, Θ) ∈ Link` with `|F| ≥ 2`, any
`(F', G, Θ)` with `F' ⊊ F` is again an arity-3 sequence of endsets), so fragments of stored values
do inhabit `Link ∪ {⊥}` — completeness is enforced by the definition, not by the type.

This is why the read recovers the arbitrary, broken collections that endsets are permitted to be.
An endset may scatter spans across many documents and across discontiguous regions within one;
the read returns every piece, because completeness is over the recorded structure, not over any
region a caller happened to name. Because the read copies the recorded spans unmodified, it
inherits their L4-generality (ASN-0043) without adding any confinement: a returned span may point
across documents, within the home document, or into the link subspace at other links. The link's
home document `home(a) = N(a).0.U(a).0.D(a)` is no part of this returned value; it is read off the
*key* `a` by T4 field projection (L2, ASN-0043), recoverable by any caller who already holds `a`.

Because `readlink(a, Σ) = Σ.L(a)` verbatim, every link-store invariant transfers to the output in
one line. In particular the returned value satisfies, as corollaries of RL1: **L3** — arity at
least three with non-empty type slot `e₃ ≠ ∅`, while the connective from- and to-slots may
individually be `∅` (`∅` is a valid endset); **L5** — within an endset there is no operator
selecting "the j-th span", so the read exposes membership, not sequence; and **Endset
well-formedness** — `Endset = 𝒫_fin(Span)` with each span `(s, ℓ)` T12-well-formed
(`Pos(ℓ) ∧ actionPoint(ℓ) ≤ #s`), denoting a non-empty contiguous region (ASN-0034). The read can
return neither a malformed nor an empty span, and it always returns a usable type endset.

## The structure the read must preserve

A span in the from-set asserts something different from the same span in the to-set or the
type-set, so the read carries an obligation beyond returning a bag of spans: it must keep each
endset aligned with its slot.

**RL2 (Role preservation).** For `a ∈ dom(Σ.L)`, the read preserves the link's arity, and slot
position is part of the value:

> `|readlink(a, Σ)| = |Σ.L(a)|`,  and for each `1 ≤ i ≤ |Σ.L(a)|` the positional accessor
> `readlink(a, Σ).eᵢ` is a model primitive (L6, ASN-0043), with link equality componentwise.

In the arity-3 case slot 1 is *from*, slot 2 is *to*, and slot 3 is *type*; for `N > 3` (L3,
ASN-0043) the higher slots are returned under their own indices. The read keeps every endset
aligned with its slot — this is exactly the alignment that any role-respecting use of the link
depends upon.

## Type is interpreted by address, not by content

The type endset deserves separate treatment, because it is the one part of the structure whose
spans need not reference anything that exists.

**RL3 (Type-by-address).** The relationship the type records is fixed by `coverage(e₃)` — the set
of addresses the type-set names — and not by whatever is, or is not, stored at those addresses.
Two links share a type exactly when their type endsets have equal coverage (L8, ASN-0043), a
relation on address sets, decided without dereferencing a single one. The read therefore delivers
a fully interpretable type even when the type address holds nothing at all: ghost types are
permitted (L9, ASN-0043), and the read of a ghost-typed link is no less complete than any other.

## Faithful disclosure of nesting

Because links live in the same address space as content, an endset may name another link. The
to-set of a link can carry a span of width one over a link's own address (the canonical reflexive
span of L13, ASN-0043), making the link's target itself a link. What must the read guarantee about
such a target? That it never *follows* it: the result must depend on the single store entry at `a`
and on nothing stored at the addresses the endsets cover. We state this as a locality property —
one a candidate implementation can be checked against.

**RL4 (Nesting locality).** For any reachable states `Σ₁, Σ₂` and any address
`a ∈ dom(Σ₁.L) ∩ dom(Σ₂.L)` with `Σ₁.L(a) = Σ₂.L(a)`:

> `readlink(a, Σ₁) = readlink(a, Σ₂)`

— immediate from the definition `readlink(a, Σ) = Σ.L(a)` on the success branch. The failure
branch supplies the complementary congruence: for `a ∉ dom(Σ₁.L) ∧ a ∉ dom(Σ₂.L)`, the definition
gives `readlink(a, Σ₁) = ⊥ = readlink(a, Σ₂)`. Together the two branches make `readlink` a
function of `(a, Σ.L(a))` alone, with `Σ.L(a)` read as a value in `Link` extended by
"undefined". Note that "pure function of `(a, Σ.L)`" would be
strictly weaker: a function of the whole store may consult `Σ.L(a')` for covered addresses `a'`
and flatten, yet still be a function of `(a, Σ.L)`; RL4 excludes this.

Several constructions below must certify that a stipulated step sequence yields reachable states.
The certification is the same each time, so we record it once.

**SOV (Store-only composite validity).** A composite whose steps touch neither `dom(C)`, nor a
content-subspace arrangement range, nor `R`, satisfies the coupling constraints J0, J1★, and J1'★
vacuously at every boundary — J0 quantifies over fresh content addresses, and
`dom(C') \ dom(C) = ∅`; J1★ over content-subspace range-new I-addresses, of which none arise;
J1'★ over new provenance entries, and `R' \ R = ∅` — and is therefore a valid composite
(ValidComposite★, ASN-0047) whenever each step's elementary precondition holds at its
intermediate state.

The exclusion has force only if the state pair RL4 quantifies over actually exists — two reachable
states agreeing on the entry at the read address while disagreeing at a covered `a'` distinct from
it. We construct the witnessing pair. Its base hypothesis — a reachable state holding an allocated
document — must itself be exhibited, since at `Σ₀` the document stratum is empty (`(E₀)_doc = ∅`,
ASN-0047): K.δ case (ii) with `k = 2` at the bootstrap node `n₀ = [1]` yields the account
`inc(n₀, 2) = [1.0.1]` (operand `zeros(n₀) = 0 ≤ 1`, parent `n₀ ∈ E₀`, freshness by
ChildSpawnFreshness, ASN-0047), and a second `k = 2` step at `[1.0.1]` yields the document
`inc([1.0.1], 2) = [1.0.1.0.1]` (operand `zeros = 1 ≤ 1`); the post-state registers this document
in `dom(M)` with the empty arrangement and is a state of the required form. Take any reachable
`Σ*` with a document `d ∈ dom(Σ*.M)`, and
let `a'` be the frontier emission of `d`'s link sub-allocator `A_L(d)` at `Σ*` (the address K.λ's
binding precondition fixes, ASN-0093). Pick two conforming link values `v₁ ≠ v₂` — say, two
L3-conforming triples differing in their type endsets. K.λ's precondition constrains the value
only through L3, never through its content, so *both* steps are enabled at `Σ*`; branch the
history by taking K.λ at `a'` with `v₁` in one branch and with `v₂` in the other. The two
post-states have identical `dom(L)`, identical `dom(C)` and `dom(M)` (K.λ's frame), and identical
link-store entries everywhere except at `a'`. Now extend both branches by the *same* step: K.λ at
the next frontier `c = inc(a', 0)` with the one L3-conforming value
`ℓ_c = (∅, {(a', δ(1, #a'))}, Θ₀)` for some fixed non-empty endset `Θ₀` — slot 2 carries the
canonical reflexive span over `a'` (L13, ASN-0043), and the non-empty type slot `Θ₀` discharges
K.λ's value-shape conjunct (`N ≥ 3`, each slot in `Endset`, `e₃ ≠ ∅`). The *state-dependent*
conjuncts of this step's precondition consult only `dom(L)` (the frontier maximum) and `dom(M)`,
on which the branches agree; the value-shape conjunct is branch-independent and discharged by
`ℓ_c`'s form. The step is therefore enabled identically in both branches, and it allocates the
same address `c` in both. The steps of both branches — K.δ and K.λ alike — touch neither
`dom(C)`, nor a content-subspace arrangement range, nor `R`, so they compose into valid
composites (SOV); both resulting states are therefore reachable, as RL4's quantification requires.
Writing `Σ₁, Σ₂` for the
resulting states (their entries frozen thereafter by L12):

> `Σ₁.L(c) = ℓ_c = Σ₂.L(c)`,  `Σ₁.L(a') = v₁ ≠ v₂ = Σ₂.L(a')`,  `a' ∈ coverage(Σ₁.L(c).e₂)`,

the last by PrefixSpanCoverage (ASN-0043) and reflexivity of `≼`. RL4 applied at `c` forces
`readlink(c, Σ₁) = readlink(c, Σ₂)`, so the result embeds nothing read from `a'` — a flattening
reader, returning covered values dereferenced, would differ across the pair and so violate RL4. A
nested link address is disclosed as the tumbler address it is, whether it names content, another
link, or nothing at all.

## Determinacy and the immutability of the recorded relationship

A read is only as trustworthy as the stability of what it reads. The link store is append-only and
its values are frozen: once allocated, a link's address persists and its value never changes
(L12, L12a of ASN-0043). The read inherits this stability.

**RL5 (Determinacy).** `readlink` is a pure function of `(a, Σ.L(a))` (RL4). Moreover, the read is
stable across the whole future:

> `(A Σ, Σ' : Σ →* Σ' ∧ a ∈ dom(Σ.L) : readlink(a, Σ') = readlink(a, Σ))`.

Stability across `Σ →* Σ'` follows from LP13 (UnconditionalLinkPersistence, ASN-0098), giving
`a ∈ dom(Σ'.L)` and `Σ'.L(a) = Σ.L(a)`; hence `readlink(a, Σ') = Σ'.L(a) = Σ.L(a) = readlink(a, Σ)`.

A reader who has once read a link may rely on that reading permanently.

*The failure branch's stability is asymmetric in the structural screen.* For a *screen-failing*
address, `⊥` is permanent: each screen conjunct is necessary for membership in `dom(Σ'.L)` at
every reachable `Σ'` — the invariants behind RL0's necessity claims (L0b, L1, L0, L1b of
ASN-0043) hold at every reachable state — so a screen-failing `a` satisfies `a ∉ dom(Σ'.L)`
throughout the future. The screen is therefore a one-sided test: failure *proves* permanent
absence; passage, by itself, proves nothing about the future. Some
screen-passing addresses are unstable — take `a` at the frontier of an active link sub-allocator
`A_L(d)` (the address K.λ's binding precondition fixes, ASN-0093): `readlink(a, ·) = ⊥` at the
state before the K.λ step, and a link value after it; the worked read's
`a = [1.0.1.0.1.0.2.1]` below is exactly such a frontier address at the state preceding its
allocating step. Others are permanently absent despite passing the screen, and three families
witness this. *Depth:* at every reachable state `Σ'`, `dom(Σ'.C) ∪ dom(Σ'.L) ⊆ F` (LP-Sub,
ASN-0098), and the structural form of `F` (SubstrateEmittableAddresses, ASN-0098: every member is
`[d, 0, s, k]`) fixes `#E(·) = 2` on all of `F`. Hence `#E(a) > 2 ⟹ a ∉ F ⟹ a ∉ dom(Σ'.L)` at
every reachable `Σ'`. So
`a = [1.0.1.0.1.0.2.1.1]` passes every screen conjunct (T4-valid; `zeros(a) = 3`;
`E(a) = [2, 1, 1]`, so `subspace_I(a) = s_L` and `#E(a) = 3 ≥ 2`) yet is permanently absent.
Gregory's allocator concurs: its sole link-allocation path emits
addresses of the form `d.0.2.n`, advancing the final digit by the `inc(·, 0)` analogue; no
reachable path deepens the element field. *Lineage:* `a = [2.0.1.0.1.0.2.1]` passes the screen while its
node field `[2]` lies off the bootstrap lineage, and the exclusion is by contradiction. Suppose
`a ∈ dom(Σ'.L)` at some reachable `Σ'`. Then L1a (ASN-0047) gives
`home(a) = [2.0.1.0.1] ∈ dom(Σ'.M) = E'_doc`; P8 at the document gives
`parent([2.0.1.0.1]) = N(a).0.U(a) = [2.0.1] ∈ E'`; P8 again, at the account, gives
`parent([2.0.1]) = N(a) = [2] ∈ E'`. But `zeros([2]) = 0`, so `Node([2])`, and NodeLineage
(ASN-0047) then requires `n₀ ≼ [2]` — refuted at the first component, `[2]₁ = 2 ≠ 1 = (n₀)₁`.
The supposition fails, so `a ∉ dom(Σ'.L)` at every reachable `Σ'`. *User field:*
`a = [1.0.1.1.0.1.0.2.1]` passes the screen (zeros at positions 2, 5, 7, none adjacent, first and
last components nonzero; `E(a) = [2, 1]`, so `subspace_I(a) = s_L` and `#E(a) = 2`) while its user
field `U(a) = [1, 1]` has two components, and the exclusion runs by the same P8 chain. Suppose
`a ∈ dom(Σ'.L)` at some reachable `Σ'`. Then L1a gives `home(a) = [1.0.1.1.0.1] ∈ E'_doc`, and P8
gives `parent([1.0.1.1.0.1]) = N(a).0.U(a) = [1.0.1.1] ∈ E'` — an account whose user field has two
components. But every account in every reachable `E'` has a single-component user field: accounts
enter `E` only through K.δ case (ii) (ASN-0047) — with `k = 2` from a node, where
`inc(t, 2) = t.[0, 1]` (TA5(d)) carries user field `[1]`, or with `k = 0` from an existing
account, where `inc(t, 0)` modifies only the terminal component (TA5(c), TA5-SigValid) and so
preserves `#U` — while the `k = 1` branch takes only document operands and case (i) makes nodes;
induction over the history gives `#U(e) = 1` for every account in every reachable `E'`. The
supposition fails as before, so `a ∉ dom(Σ'.L)` at every reachable `Σ'`. The general test is
`#U(a) ≥ 2`. Gregory's account allocator concurs: seeded at the bootstrap node, its arithmetic
emits single-component user fields only; deeper user fields arise there solely from non-canonical
seeds, a path outside the model's transition vocabulary. The three families show that the
screen-passing class is heterogeneous, split by finer tests that are still address-computable:
depth (`#E(a) = 2` versus `#E(a) > 2`), lineage (`N(a)₁ = 1` versus `N(a)₁ ≠ 1` — with `n₀ = [1]`
fixed by Σ₀ of ASN-0047, `n₀ ≼ N(a)` is exactly `N(a)₁ = 1`), and user-field width (`#U(a) = 1`
versus `#U(a) ≥ 2`).

The split is exhaustive — outside the three families, no further permanently-absent
screen-passer hides. Call the *residual class* the screen-passing addresses with `#E(a) = 2`,
`N(a)₁ = 1`, and `#U(a) = 1`; we show every member is allocatable: for any reachable `Σ` with
`a ∉ dom(Σ.L)`, some extension `Σ →* Σ⁺` has `a ∈ dom(Σ⁺.L)`. The four fields of `a` are
zero-free (every zero of a `zeros(a) = 3` tumbler is a field separator, T4b of ASN-0034), and the
extension realises them in turn through the K.δ/K.λ vocabulary of ASN-0047, skipping any step
whose target is already present. *Node:* `N(a)` is all-positive with `N(a)₁ = 1`, i.e.
`n₀ ≼ N(a)`, so if `N(a) ∉ E` then K.δ case (i) baptises it (NodeBaptism). *Account:* the
`(N(a), 2)` spawn yields `[N(a).0.1]` (ChildSpawnFreshness — if the spawn is already spent, the
child is already in `E`), and `k = 0` sibling advances raise the single user component toward
`U(a)₁`; each advance fires at the chain's frontier (FrontierEquivalence), so `[N(a).0.U(a)₁]` is
reached or already present. *Document:* the `(account, 2)` spawn opens the document field at
`[1]`; sibling advances raise its first component to `D(a)₁`; each further component `D(a)ᵢ` is
opened by one `k = 1` version step (appending `[1]`, TA5(d)) and raised by sibling advances —
every zero-free document field is realised this way, and each intermediate document enters
`dom(M)` with the empty arrangement. *Element:* with `d := home(a) ∈ dom(M)` and
`E(a) = [s_L, k]`, the links homed at `d` form a contiguous initial segment
`{[d.0.s_L.j] : 1 ≤ j ≤ j₀}` of `A_L(d)`'s chain (ChainMembershipForOrigin, ASN-0093) with
`j₀ < k` since `a ∉ dom(Σ.L)`, and `k − j₀` K.λ steps at the frontier — each with any
L3-conforming value — deposit `a` as the chain's `k`-th emission. The steps touch neither
`dom(C)`, nor a content-subspace arrangement range, nor `R`, so they compose into valid
composites (SOV). Hence at a residual-class member `⊥` is never permanent,
and no address-computable permanence test can exist for it.

The caching discipline follows in three parts: (i) a success-branch result may be cached
permanently (L12, LP13); (ii) `⊥` may be cached permanently wherever an address-computable
permanence proof applies — screen failure, element-field depth `#E(a) > 2` (the depth family),
node field off the `n₀` lineage, `N(a)₁ ≠ 1` (the lineage family), or user-field width
`#U(a) ≥ 2` (the user-field family); (iii) `⊥` must not be cached at the residual class —
screen-passing, `#E(a) = 2`, `N(a)₁ = 1`, `#U(a) = 1` — each of whose members is allocated in some
extension of any history that has not yet allocated it, so that no permanence proof can exist
there. Caching `⊥` is sound exactly where such a proof is in hand: by the exhaustiveness of the
split, the proofs of (ii) cover the whole permanently-absent class, and the residual class is
exactly where caching is unsound.

## Recorded relationship versus resolved position

A link records its endsets as spans over the *permanent* address space. Resolving those spans
against a particular document's arrangement — mapping them to current positions — is a separate
act, the business of traversal and projection, conditional on the arrangement.

**RL6 (Recorded, not resolved).** `readlink(a, Σ)` depends only on `Σ.L` — indeed only on `Σ.L(a)`
(RL4) — and is independent of every document arrangement. Consequently the read succeeds and returns the complete structure even for an
*orphaned* link — one whose endpoint content is currently arranged in no document, so that resolving
its endsets would yield nothing (cf. the ghost-projection situation, ASN-0098). The link's structure
persists unconditionally (L12; LP13 of ASN-0098), and the read surfaces it unconditionally.

## A worked read

The claims above are abstract; we check them against one concrete link. Fix the subspace
convention `s_C = 1`, `s_L = 2` (ASN-0093). Take two documents

> `d₁ = [1.0.1.0.1]`,  `d₂ = [1.0.1.0.2]`  (each `zeros = 2`, T4-valid),

and a link homed in `d₁` at address

> `a = [1.0.1.0.1.0.2.1]`   (`zeros(a) = 3`; element field `E(a) = [2, 1]`, so
> `subspace_I(a) = E(a)₁ = s_L`, `#E(a) = 2` — the first emission of `d₁`'s link sub-allocator).

Let the stored value `Σ.L(a) = (F, G, Θ)` be the standard triple

- **from-set** `F = {([1.0.1.0.1.0.1.1], δ(2, 8)), ([1.0.1.0.2.0.1.1], δ(1, 8))}` — two spans
  scattered across *two* documents. `coverage(F)` is a *union of two half-open intervals*, not a
  finite list of points. The first span runs from `[1.0.1.0.1.0.1.1]` up to but not including
  `[1.0.1.0.1.0.1.1] ⊕ δ(2, 8) = [1.0.1.0.1.0.1.3]`. Since `…1.1 < …1.2 < …1.3` under T1, the
  half-open interval decomposes as `[…1.1, …1.2) ∪ […1.2, …1.3)`, and each part is the coverage
  of a unit-depth span (`δ(1, 8)` at `…1.1`, resp. at `…1.2`); by PrefixSpanCoverage (ASN-0043)
  the interval is therefore exactly the union of the two subtrees `{t : …1.1 ≼ t} ∪ {t : …1.2 ≼ t}`
  (e.g. `[1.0.1.0.1.0.1.1.0]`, `[1.0.1.0.1.0.1.2.5]`), an infinite tumbler set, *not* the two
  addresses `…1.1` and `…1.2` alone. The second span is the
  interval `[ [1.0.1.0.2.0.1.1], [1.0.1.0.2.0.1.2] )` under `d₂`. The element-level content
  I-addresses lying within `coverage(F)` are `[1.0.1.0.1.0.1.1]` and `[1.0.1.0.1.0.1.2]` under `d₁`
  and `[1.0.1.0.2.0.1.1]` under `d₂` — three `dom(C)` members that host content and are unarranged.
- **to-set** `G = ∅` — a legitimately empty connective slot.
- **type-set** `Θ = {([1.0.1.0.9.0.1.1], δ(1, 8))}` — a single span whose address sits under a
  document `[1.0.1.0.9]` that hosts no content: a *ghost* type, a label by location.

The configuration just stipulated — three allocated, content-bearing, *unarranged* I-addresses —
cannot arise at allocation, since J0 (AllocationPlacementCoupling, ASN-0047) requires every fresh
I-address to be arranged at its allocating composite's boundary; it is nevertheless reachable, and
we exhibit the route rather than assume it. Each of the three addresses is a chain emission of its
document's content sub-allocator (`[1.0.1.0.1.0.1.1]` and `[1.0.1.0.1.0.1.2]` the first two
emissions of `A_C(d₁)`, `[1.0.1.0.2.0.1.1]` the first of `A_C(d₂)`; `d₁` enters `dom(M)` by the
K.δ route exhibited in the RL4 construction, and `d₂ = inc(d₁, 0)` by a `k = 0` sibling step), and
each enters `dom(C)` inside a valid composite — K.α coupled with a
K.μ⁺ arranging the fresh address at the boundary (discharging J0) and a K.ρ recording `(a, d)`
for the range-new address (discharging J1★; J1'★ holds because each new provenance entry is
exactly that range-new pair). A subsequent K.μ⁻ on each document with content-subspace retention
`n'_{s_C} = 0` then empties the content V-positions while `dom(C)` retains all three entries
(P0). Each contraction is itself a valid composite — its couplings are vacuous, as J2
(ContractionIsolation, ASN-0047) records: K.μ⁻'s frame gives `C' = C` and `R' = R`, so J0 and
J1'★ quantify over empty sets, and the arrangement range only shrinks, so no range-new
content-subspace I-address arises for J1★. The provenance entries persist through the
contraction (P2), so P4★ and P7a hold at this and every subsequent composite boundary. The link
addresses of this section — `a`, then `a' = inc(a, 0)` and `c = inc(a', 0)` below — are the first
three emissions of `A_L(d₁)`'s chain, allocated by three bare K.λ steps, each a valid composite
by SOV.

A direct read returns the whole triple, grouped by slot:

> `readlink(a, Σ) = (F, ∅, Θ)`,
> with `readlink(a, Σ).e₁ = F`, `readlink(a, Σ).e₂ = ∅`, `readlink(a, Σ).e₃ = Θ`.

We can now check the load-bearing postconditions against this instance.

- *RL1 (completeness).* The read returns *both* from-spans, the empty to-set, and the type-span —
  every recorded span and nothing more.
- *RL2 (role preservation).* The three endsets come back under slots 1/2/3, not pooled. Were the
  read to return the bag `F ∪ Θ`, the reader could no longer tell that `[1.0.1.0.9.0.1.1]`
  classifies the link while the others are its source — a different relationship (L6). The read
  copies the stored `Σ.L(a).eᵢ` into `readlink(a, Σ).eᵢ` by a per-index rule that names no other
  slot (link equality is componentwise, L6): an arity-4 value `(F, ∅, Θ, e₄)` returns `e₄` under
  slot 4 by exactly the same copy.
- *RL3 (ghost-type completeness).* `Θ`'s address holds nothing, yet the read returns it intact.
  Its single span `([1.0.1.0.9.0.1.1], δ(1, 8))` is the canonical unit-depth span (`#s = 8 = #δ(1, 8)`),
  so by PrefixSpanCoverage (ASN-0043) `coverage(Θ) = {t : [1.0.1.0.9.0.1.1] ≼ t}` — the *subtree*
  beneath that address, an infinite tumbler set, not the single point `[1.0.1.0.9.0.1.1]`. As with
  the from-set above, the type is interpreted as this coverage *address-set* (L8) and matched against
  another type's coverage without dereferencing anything stored there.
- *Structural corollary (arity/type).* Arity is 3 and `Θ ≠ ∅` (L3), while the connective slot
  `G = ∅` is permitted — exactly the mandatory-type / permissive-direction split.

*A nested instance (RL4).* Links share the address space with content, so an endset may target
another link. Let `a' = inc(a, 0) = [1.0.1.0.1.0.2.2]` be a second link homed in `d₁` — the next
sibling on `d₁`'s link sub-allocator, so `a' ∈ dom(Σ.L)` — and consider a third link
`c = [1.0.1.0.1.0.2.3]`, also homed in `d₁`, whose to-set is the canonical reflexive span over `a'`:

> `Σ.L(c) = (∅, G_c, Θ)`,  `G_c = {([1.0.1.0.1.0.2.2], δ(1, 8))}`   (`#a' = 8 = #δ(1, 8)`, the
> canonical unit-depth span of L13, ASN-0043; `c` reuses the ghost type `Θ`, so it is a conforming
> arity-3 link with non-empty type slot).

The read returns `readlink(c, Σ).e₂ = G_c`, the span intact. By PrefixSpanCoverage (ASN-0043) its
coverage is the subtree `coverage(G_c) = {t : a' ≼ t}`, so `a' ∈ coverage(readlink(c, Σ).e₂)`; and
since `readlink(c, Σ)` is a function of `(c, Σ.L(c))` alone (RL4), whatever `a'` itself records
cannot enter the result — `a'` is disclosed *as the tumbler address it is*, unflattened. This
verifies RL4 against a concrete link→link target — the construction underlying compound and
faceted structures.

*An orphaned instance (RL6).* Suppose `a` is orphaned at `Σ` — no document arrangement maps any
V-position into the coverage of any of its endsets, so a *follow* of `F` against any arrangement
would resolve to the empty set and a *search* would find nothing to match (the ghost-projection
situation, LP17, ASN-0098). The READLINK obligation is unaffected by this hypothesis: the read
consults only `Σ.L`, never an arrangement (RL6), so `readlink(a, Σ) = (F, ∅, Θ)` still returns the
complete structure. The read thus distinguishes *the relationship is unwitnessed* (true here) from
*the relationship is gone* (false — `a ∈ dom(Σ.L)` and its value is fixed by L12 / LP13).

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| `readlink` | `readlink : T × 𝒮 → Link ∪ {⊥}` (`𝒮` the extended state space `Σ = (C, L, E, M, R)` of ASN-0047, second argument restricted to reachable states); `readlink(a, Σ) = Σ.L(a)` when `a ∈ dom(Σ.L)`, else `⊥`; pure read, frame `Σ' = Σ` | introduced |
| RL0 | Totality and success — defined for every `a ∈ T`; `readlink(a, Σ) ∈ Link ⟺ a ∈ dom(Σ.L)`, else `⊥`; every conjunct of the structural screen `T4-valid(a) ∧ zeros(a) = 3 ∧ subspace_I(a) = s_L ∧ #E(a) ≥ 2` is necessary, and no satisfiable address-computable predicate is sufficient (witness: `dom(Σ₀.L) = ∅`) | introduced |
| RL1 | Completeness — on `a ∈ dom(Σ.L)` the read returns every recorded span of every endset and no other; `readlink(a, Σ) = Σ.L(a)`; inherits L4-generality of the recorded spans. Corollaries (since the output is `Σ.L(a)`): satisfies L3 (arity ≥ 3, non-empty type slot, connective slots may be `∅`), L5 (membership not sequence within an endset), and Endset well-formedness (T12 spans) | introduced |
| RL2 | Role preservation — on `a ∈ dom(Σ.L)` the read preserves arity (`|readlink(a, Σ)| = |Σ.L(a)|`) and exposes slot position as a model primitive (L6); from/to/type grouping delivered as structure | introduced |
| RL3 | Type-by-address — the type is interpreted via `coverage(e₃)`, not via content at those addresses; ghost types read completely | introduced |
| SOV | Store-only composite validity — a composite whose steps touch neither `dom(C)`, nor a content-subspace arrangement range, nor `R` satisfies J0, J1★, and J1'★ vacuously, hence is a valid composite whenever each step's elementary precondition holds | introduced |
| RL4 | Nesting locality — `readlink` is a function of `(a, Σ.L(a))` alone: reachable `Σ₁, Σ₂` with `a ∈ dom(Σ₁.L) ∩ dom(Σ₂.L)` and `Σ₁.L(a) = Σ₂.L(a)` give `readlink(a, Σ₁) = readlink(a, Σ₂)`, and `a ∉ dom(Σ₁.L) ∧ a ∉ dom(Σ₂.L)` gives `readlink(a, Σ₁) = ⊥ = readlink(a, Σ₂)`; corollary: covered link addresses are returned as addresses, never dereferenced, witnessed by an explicit branched-history state pair | introduced |
| RL5 | Determinacy — pure function of `(a, Σ.L(a))` (RL4); success-branch results stable across all `Σ →* Σ'` by link immutability; the structural screen is a one-sided test (failure proves permanent absence; passage proves nothing about the future); three permanence families — depth `#E(a) > 2`, lineage `N(a)₁ ≠ 1`, user-field `#U(a) ≥ 2` — together with screen failure exhaust permanent absence, every member of the residual class (screen-passing, `#E(a) = 2`, `N(a)₁ = 1`, `#U(a) = 1`) being allocatable; `⊥` is cacheable exactly where one of these address-computable permanence proofs is in hand | introduced |
| RL6 | Recorded, not resolved — the read depends only on `Σ.L(a)`, succeeds for orphaned links, and returns the complete structure independent of any arrangement | introduced |

## Open Questions

What must the system guarantee a reader can conclude about a relationship's continued validity from a direct read alone, given that the read does not consult any arrangement?

What must FOLLOWLINK guarantee so that an endset legitimately empty at the read level stays distinguishable from one whose spans reference only currently-unwitnessed content, given that resolution against an arrangement collapses both to the empty position set?

What guarantee must hold so that reading two distinct links with identical recorded structure always yields results distinguishable by the reader, given that addresses, not values, carry link identity?
