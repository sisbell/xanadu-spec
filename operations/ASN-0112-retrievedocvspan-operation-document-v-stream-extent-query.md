> **ASN-0112 · RETRIEVEDOCVSPAN Operation — Document V-Stream Extent Query** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md)  
> [Condensed statements →](ASN-0112-retrievedocvspan-operation-document-v-stream-extent-query.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0112: RETRIEVEDOCVSPAN Operation — Document V-Stream Extent Query

*2026-06-04*

We are trying to understand the simplest question one can put to a document: *given
only your name, where does your content begin and how far does it reach?* The caller
hands over a document identity and nothing else — no range, no position, no selection —
and expects back a single answer that bounds the whole. Our task is to say, formally,
what that answer is, what it must describe, and what may be true of it.

The operation is a *boundary query*, not a content read. It takes no span argument and
delivers no bytes. Nelson fixes its shape exactly: it "returns a span determining the
origin and extent of the V-stream of document `<doc id>`" (4/68). So the input is a bare
document identity and the output is *one span*: a pair of an origin and an extent. We must
decide what that origin and extent denote, what relationship they bear to the document's
present arrangement, what the caller gains over what the identity already disclosed, and
what invariants constrain the span the operation may legally return.

We write the operation as a pure query, `RETRIEVEDOCVSPAN(d)`, that observes the state and
returns a value, changing nothing. We record this no-mutation guarantee as **V-frame**:
the post-state equals the observed state, `Σ' = Σ`, so every component — content store `C`,
link store `L`, entity set `E`, arrangement family `M`, provenance relation `R` — is left
intact.

---

## The substrate we measure

We take the strand model of state as given. A document `d` carries an *arrangement*
`M(d) : T ⇀ T`, a partial function from V-positions — positions in the document's current
virtual stream — to I-addresses, the permanent keys of a content store `C : T ⇀ Val`. We
write

> `O(d) = dom(M(d))`

for the set of *occupied V-positions* of `d`: the positions that currently carry content
in the arrangement. This set is exactly what RETRIEVEDOCVSPAN must bound. We rely on these
foundation facts:

- **S2** (functionality): each occupied V-position has a single I-address.
- **S3★** (generalized referential integrity, ASN-0047): each occupied V-position maps
  into the store appropriate to its subspace —
  `(A v : v ∈ O(d) : subspace(v) = s_C ⟹ M(d)(v) ∈ dom(C)) ∧ (A v : v ∈ O(d) : subspace(v) = s_L ⟹ M(d)(v) ∈ dom(L))`.
- **S3★-aux** (subspace exhaustiveness, ASN-0047): every occupied V-position carries one
  of exactly two subspaces — `(A v : v ∈ O(d) : subspace(v) = s_C ∨ subspace(v) = s_L)`.
  There is no third subspace, so an arrangement occupies the content subspace, the link
  subspace, both, or neither.
- **S8-fin** (finiteness): `O(d)` is finite.
- **S8a** (well-formedness): every `v ∈ O(d)` is zero-free, of depth `≥ 2`, all
  components positive; `subspace(v) = v₁`.
- **S8-depth**: within one subspace all occupied V-positions share a common depth.
- **D-CTG★ / D-MIN★ / D-SEQ★** (per-subspace shape, ASN-0047): for *each* non-empty
  subspace `S ∈ {s_C, s_L}`, the positions `V_S(d) = {v ∈ O(d) : subspace(v) = S}` are
  contiguous, their minimum is the canonical `[S,1,…,1]`, and they form the dense run
  `{[S,1,…,1,k] : 1 ≤ k ≤ n_S}` for some `n_S ≥ 1`. We write the content instance
  `D-MIN`/`D-SEQ` for `S = s_C` and the link instance for `S = s_L`; both inherit the same
  dense-run shape.
- **S0 / P0** (content immutability and permanence): once `a ∈ dom(C)`, `a` stays in
  `dom(C)` forever and `C(a)` never changes.
- **L12** (link immutability, ASN-0043): once `a ∈ dom(L)`, `a` stays in `dom(L)` forever
  and the link value `L(a)` never changes — the link-store analogue of S0/P0.

Two subspaces inhabit the arrangement: content positions carry `subspace = s_C` and link
positions carry `subspace = s_L`, with the fixed convention `s_C = 1`, `s_L = 2`
(SubspaceConventionAxiom). Because `s_C < s_L` at the first component, T1 places every
content position before every link position.

We borrow the span machinery wholesale. A span `σ = (s, ℓ)` denotes the half-open interval
`⟦σ⟧ = {t ∈ T : s ≤ t < s ⊕ ℓ}` (T12), with `reach(σ) = s ⊕ ℓ` (ASN-0053). A span is
*well-formed* when `Pos(ℓ)` and `actionPoint(ℓ) ≤ #s`; it is *level-uniform* when
`#s = #ℓ` (S6, ASN-0053). The ordinal shift `shift(t, n) = t ⊕ δ(n, #t)` advances `t`'s last
component by `n` (ASN-0034).

---

## What the caller must be handed

Nelson fixes the *type* of the result: a span, "the origin and extent of the V-stream"
(4/68). Not a sequence of records — that would be a content read. Not a count: "a
tumbler-span is not a conventional number, and it does not designate the number of bytes
contained. It does not designate a number of anything" (4/24). The result is a *boundary
description* — two tumblers, a start and a width, whose meaning is "from here, this far,"
with everything between implicit (4/25).

The operation therefore returns a *span-set* (ASN-0053) — never a content sequence, never a
cardinality. We record this as **V0** (span-set result), the uniform codomain

> `RETRIEVEDOCVSPAN : dom(M) → SpanSet`,

where `SpanSet` is ASN-0053's type of finite span sequences. For a non-empty document
`RETRIEVEDOCVSPAN(d)` returns the singleton span-set `⟨σ_d⟩` carrying one well-formed span
`σ_d = (origin_d, extent_d)`; for an empty document (`O(d) = ∅`) it returns the empty span-set
`⟨⟩`. The two are distinguishable by denotation: `⟨⟩` denotes `∅`, while
`⟨σ_d⟩` denotes the non-empty `⟦σ_d⟧` (S2, ASN-0053: every well-formed span is non-empty), so
no populated result can be confused with the empty one.

The caller reads `origin_d` to learn where the V-stream begins and `extent_d` to learn how far it
reaches; the content itself, and any per-piece count, are the business of other operations.

---

## The bounding span and its two endpoints

Nelson's intent is that origin and extent describe the document as a whole implicitly —
"there is no choice as to what lies between; this is implicit in the choice of first and last
point" (4/25). Reasoning from "origin and extent of the V-stream," we must produce a span that
*spans the whole document* — one region containing all of its arranged content. The occupied set
`O(d)` is finite (S8-fin) and totally ordered by T1, so when non-empty it has a least
element and a greatest element. Define

> `origin_d = min O(d)`,  `reach_d = shift(max O(d), 1)`,  `extent_d = reach_d ⊖ origin_d`,

and `σ_d = (origin_d, extent_d)`. The reach advances one ordinal step past the maximum
occupied position, realizing the half-open convention under which the last occupied
position is included and the next is excluded. We must show this is well-defined and
forced.

**The origin is an occupied position.** We record **V1**: when `O(d) ≠ ∅`,
`origin_d = min O(d)` and `origin_d ∈ O(d)`. The minimum of a finite, totally ordered,
non-empty set exists, is unique, and is a member. So the reported origin is never a
fictitious lower boundary; it is the actual V-address at which the document's first
arranged content sits. Gregory's implementation realizes exactly this: the query reads the
arrangement-tree root's V-displacement, which is maintained to equal the minimum V-address
of any content in the document (consultation Q12, Q15, Q20) — "the grasp is always
occupied" (Q20). The start it reports for a text-bearing document is `1.1`, the first
character position, not a padded `1.0` (Q15).

**The extent is a well-formed displacement.** We first establish that `σ_d` is a legal T12
span regardless of whether its endpoints share a depth. Since `reach_d = shift(max O(d), 1) >
max O(d) ≥ origin_d` (TS4, ShiftStrictIncrease), we have `origin_d < reach_d`. The first
position at which `origin_d` and `reach_d` diverge, `k = divergence(origin_d, reach_d)`,
satisfies `k ≤ #origin_d` in every case: in the single-subspace case both tumblers lie in one
subspace `s` (content or link), so by S8-depth they share the common depth of that subspace —
`#origin_d = #max O(d) = #reach_d` (OrdinalShift preserves depth) — and the first divergence of
two equal-length tumblers cannot exceed their shared length, giving `k ≤ min(#origin_d, #reach_d)
= #origin_d`; in the cross-subspace case they differ already at position 1 (`s_C` vs `s_L`), so
`k = 1 ≤ #origin_d`. By D0 (DisplacementWellDefined, ASN-0034) — applicable because `origin_d <
reach_d` and `divergence(origin_d, reach_d) ≤ #origin_d` — the displacement `extent_d =
reach_d ⊖ origin_d` is a positive tumbler with `actionPoint(extent_d) = k ≤ #origin_d`. Hence
`(origin_d, extent_d)` satisfies T12 and is a well-formed span.

**The span covers every occupied position.** We record **V2** (covering): `O(d) ⊆ ⟦σ_d⟧`.
The denotation is `⟦σ_d⟧ = {t : origin_d ≤ t < origin_d ⊕ extent_d}`, so we must locate the
*actual* reach `r⋆ = origin_d ⊕ extent_d` and show `max O(d) < r⋆`. Two cases on the relative
depths:

- *`#origin_d ≤ #reach_d`* (in particular the case `#origin_d = #reach_d`, i.e. `level_compat(origin_d, reach_d)`). By
  D1 (DisplacementRoundTrip, ASN-0034) the round-trip closes exactly: `r⋆ = origin_d ⊕
  (reach_d ⊖ origin_d) = reach_d`. Then for any `v ∈ O(d)`, `origin_d ≤ v ≤ max O(d) <
  reach_d = r⋆`, so `v ∈ ⟦σ_d⟧`.
- *`#origin_d > #reach_d`* (content deeper than the maximal link position). Unequal endpoint
  depths force the cross-subspace case — single-subspace endpoints are equidepth by S8-depth —
  so `k = divergence(origin_d, reach_d) = 1` (divergence at the subspace component, `s_C` vs
  `s_L`). By D0 the round-trip *fails* — `r⋆ ≠ reach_d` — so we compute `r⋆` componentwise,
  first the width by TumblerSub, then the sum by TumblerAdd. Write `p = #origin_d`,
  `q = #reach_d`, so `p > q`. *The width, by TumblerSub at `zpd(reach_d, origin_d) = 1`:*
  `extent_d₁ = reach_d₁ − origin_d₁` (the action-point component); `extent_dᵢ = reach_dᵢ`
  (zero-padded) for `i > 1` — that is, `extent_dᵢ = reach_dᵢ` for `2 ≤ i ≤ q` and
  `extent_dᵢ = 0` for `q < i ≤ p`; and `#extent_d = max(p, q) = p` (TA2), with
  `actionPoint(extent_d) = 1` (TumblerSub's conditional postcondition). *The sum, by
  TumblerAdd at `k = 1`:* at the action point the components cancel in ℕ —
  `r⋆₁ = origin_d₁ + extent_d₁ = origin_d₁ + (reach_d₁ − origin_d₁) = reach_d₁` — and the
  tail copies the width, `r⋆ᵢ = extent_dᵢ` for `i > 1`, so `r⋆ᵢ = reach_dᵢ` for `2 ≤ i ≤ q`
  and `r⋆ᵢ = 0` for `q < i ≤ p`, with `#r⋆ = #extent_d = p` (result-length identity). So
  `r⋆` agrees with `reach_d` on every position `1 ≤ i ≤ q`, and `q < p` strictly: `reach_d`
  is a proper prefix of `r⋆`, and `reach_d < r⋆` (T1 case (ii)). Hence
  `max O(d) < reach_d < r⋆`, and again every `v ∈ O(d)` lies in `⟦σ_d⟧`.

In both cases `r⋆ ≥ reach_d > max O(d)`, so coverage holds; whether `r⋆` equals or strictly
exceeds `reach_d` is recorded by **V-ReachTight** (reach tightness):
`reach(σ_d) = reach_d ⟺ #origin_d ≤ #reach_d`. Both directions are already discharged by V2's
two covering cases above — case 1 (`#origin_d ≤ #reach_d`) closes the round-trip to
`r⋆ = reach_d`, and case 2 (`#origin_d > #reach_d`) computes `reach_d < r⋆`. This is
strictly weaker than equal endpoint depths: it holds automatically in the single-subspace
regime (S8-depth makes the endpoints equidepth) and, in the cross-subspace case, throughout
`m_C ≤ m_L`.

**Whether the returned span is level-uniform.** The same depth axis settles whether `σ_d`
satisfies S6 (`#start = #width`, ASN-0053). By TA2 (WellDefinedSubtraction, ASN-0034) the
displacement length is `#extent_d = max(#origin_d, #reach_d)`, so `#origin_d = #extent_d`
holds exactly when `#origin_d ≥ #reach_d`. We record **V-LevelUniform**: `σ_d` is
level-uniform `⟺ #origin_d ≥ #reach_d`. In the single-subspace regime the endpoints are
equidepth (the shared premise established for V-ReachTight above), so `#origin_d = #reach_d`
and the span is level-uniform; in the
cross-subspace case it is level-uniform precisely when content is no shallower than the
maximal link position (`m_C ≥ m_L`), and strictly non-level-uniform when `m_C < m_L`.

**The constructed endpoint is the tightest same-depth covering bound.** We record **V3**
(bounding): `origin_d` is the greatest lower bound of `O(d)`, and the *constructed endpoint*
`reach_d` is the *least* admissible upper bound of `max O(d)` among tumblers of its depth.
The lower bound is unconditional: any span
`σ'` with `O(d) ⊆ ⟦σ'⟧` satisfies
`start(σ') ≤ min O(d) = origin_d`. The upper bound requires an argument. Write `w = max O(d)`.
Because every V-position is zero-free with all components positive (S8a), the rightmost nonzero
component of `w` is its last, so `sig(w) = #w` (TA5-SIG, ASN-0034); hence
`reach_d = shift(w, 1) = w ⊕ δ(1, #w)` coincides with `inc(w, 0)`, since OrdinalShift
(ASN-0034) advances the same last component that `inc(·, 0)` modifies at `sig(w)`. ASN-0034's
TA5 (HierarchicalIncrement) settles the tightness directly: `inc(w, 0)` is the smallest
same-length tumbler strictly greater than `w`, while the true T1-immediate successor of `w` is
the deeper zero-extension `w.0` (by the prefix convention, T1 case (ii)), satisfying
`w < w.0 < inc(w, 0) = reach_d`. So `reach_d` is *not* the least strict upper bound of `w`
over all of `T` — the chain just displayed exhibits the strictly smaller upper bound `w.0` —
but it is the least strict upper bound of `w` at `w`'s depth — V3's claim. V3's minimality is
order-theoretic, over same-depth tumblers; whether a well-formed covering span can *attain*
the denotational reach `w.0` is a separate question (see Open Questions).

---

## The Vstream is what we measure, not the Istream

Nelson is emphatic that the report is over the *V-stream* — the present arrangement — not
the permanent content store. "This returns a span determining the origin and extent of the
**V-stream**" (4/68). The distinction is sharp and load-bearing.

Content that has been removed from the arrangement persists permanently in the store (S0,
P0) but leaves `O(d) = dom(M(d))`. Such content is, in Nelson's phrase, "not currently
addressable" (4/9): it "may remain included in other versions" (4/11) but is gone from this
document's current Vstream. We record **V4** (Vstream-bounded): `extent_d` is computed from
`O(d)` alone, so content present in `dom(C)` but absent from `dom(M(d))` — deleted-but-stored
content, or content native elsewhere and not arranged here — contributes nothing to the
reported span. The extent measures *what the arrangement currently contains*, not *what the
store has ever held*.

---

## Exact cover within a subspace; a bounding box across subspaces

The decisive structural question is whether the single returned span exactly traces the
occupied content or merely encloses it. The answer depends on how many subspaces the
arrangement occupies. By S3★-aux a non-empty `O(d)` occupies
exactly one subspace or exactly both, so the two cases below are jointly exhaustive.

**Single subspace: exact cover.** Suppose `O(d)` lies entirely in one subspace `s` — either
content (`s = s_C`) or, in the link-only case (content empty, one or more links arranged — a
reachable state), the link subspace (`s = s_L`). By D-SEQ★ (the per-subspace dense-run
shape, ASN-0047, instantiated at `S = s`) the occupied positions are
`{[s,1,…,1,k] : 1 ≤ k ≤ n_s}`, a dense run with no internal gaps. Then `origin_d = [s,1,…,1]`
(D-MIN★), `max O(d) = [s,1,…,1,n_s]`, `reach_d = [s,1,…,1,n_s+1]`, and `⟦σ_d⟧` restricted to
depth-`m_s` positions is exactly that run. The restriction needs two steps, because D-CTG★
constrains only slice tuples lying between two members of `V_S(d)` — it is silent both on the
half-open boundary cell `(max O(d), reach_d)` and on depth-`m_s` tumblers that are not slice
tuples (a zero component, or first component `≠ s`). *(i) Prefix-pinning.* Let `t` be any
depth-`m_s` tumbler with `origin_d ≤ t < reach_d`. The endpoints agree on positions
`1..m_s−1` (the value `s` followed by 1's), so if `t` first diverges from that common prefix
at some `j ≤ m_s−1`, a component below the prefix value forces `t < origin_d` and a component
above it forces `t > reach_d` (T1 case (i), a contradiction either way); hence `t` carries
the prefix `[s,1,…,1]` exactly. *(ii) Discreteness at the boundary cell.* With the prefix
pinned, the T1 comparisons reduce to the final component: `t ≥ origin_d` gives `t_{m_s} ≥ 1`,
and `t < reach_d` gives `t_{m_s} < n_s + 1`, whence `t_{m_s} ≤ n_s` because no natural lies
strictly between `n_s` and `n_s + 1` — exactly the TA5 tightness of
`reach_d = inc(max O(d), 0)` as the least same-length tumbler above `max O(d)`, established
at V3. So `t = [s,1,…,1,k]` with `1 ≤ k ≤ n_s`, a member of the D-SEQ★ run. Call a tumbler
`t` *occupied-depth* at `(Σ, d)` iff `#t = m_S(d)` for some subspace `S ∈ {s_C, s_L}` with
`V_S(d) ≠ ∅` — its depth is the S8-depth common depth of some non-empty subspace of `d`'s
arrangement; in the cross-subspace case with `m_C ≠ m_L` there are exactly two occupied
depths. We record **V5**
(exact cover): when all occupied positions share one subspace, `⟦σ_d⟧` contains no
occupied-depth position outside `O(d)` — the span is a faithful trace, "dense and contiguous,"
with the document forming "an unbroken sequence" (4/11). In the single-subspace case the
sole non-empty subspace is `s`, so the only occupied depth is `m_s`, and steps (i)–(ii)
dispose of every depth-`m_s` tumbler in `⟦σ_d⟧`. The density is supplied by the
per-subspace D-SEQ★, so the claim holds for a link-only document exactly as for a content-only
one. The golden case confirms the content instance: eleven characters of text report
`1.1 for 0.11`, the half-open interval `[1.1, 1.12)` covering exactly positions
`1.1 … 1.11` (consultation Q15).

**Two subspaces: a bridging bounding box.** Now suppose `O(d)` holds both content
(`subspace = s_C`) and link (`subspace = s_L`) positions. Then `origin_d` is the content
start `[s_C,1,…]` (since `s_C < s_L`), but `max O(d)` is a link position `[s_L, …]`. The
reach crosses from subspace `s_C` into subspace `s_L`, so `⟦σ_d⟧` contains *every* position
between them — including the unoccupied void separating the two subspaces, where nothing is
arranged. We record **V6** (cross-subspace bounding box — the negation of V5): when
occupied positions span more than one subspace, `⟦σ_d⟧` contains an *occupied-depth* position
outside `O(d)` — the span is a bounding box, not an exact cover. The witness is
`w⋆ = [s_C,1,…,1,n_C+1]` (at the occupied content depth `m_C`, where `n_C = |V_{s_C}(d)|` —
occupied-depth by the V5 definition, since `V_{s_C}(d) ≠ ∅`): it
is a content position, hence below every `s_L` reach by T1, so `origin_d ≤ w⋆ < reach_d` and
`w⋆ ∈ ⟦σ_d⟧`; yet its final component `n_C+1` places it just past the dense content run
`{[s_C,1,…,1,k] : k ≤ n_C}` (D-SEQ★), so `w⋆ ∉ O(d)`. V6 is existential, so this single
depth-`m_C` witness suffices; whether depth-`m_L` tumblers supply
further witnesses when `m_L ≠ m_C` is immaterial. The bare strict inclusion
`O(d) ⊊ ⟦σ_d⟧` follows as a corollary, but it cannot carry the dichotomy: even in V5's
exact-cover case the inclusion is strict, since the zero-extension `origin_d.0` satisfies
`origin_d < origin_d.0 < reach_d` (T1 case (ii); then divergence at the final position,
`1 < n_s+1`) yet is no V-position (S8a forbids its zero component). A span denotes one
convex region (`⟦σ_d⟧` is
order-convex under T1, ASN-0053 S0), and a document occupying two disjoint subspaces is a
*separated series* — "if you want to
designate a separated series of items exactly, including nothing else, you do this by a
span-set, which is a series of spans" (4/25). Fragmentation is unrepresentable in a single
span, so a multi-subspace document can only be reported by enclosure, never by exact
decomposition. The golden case is stark: ten
characters plus one link report `1.1 for 1.2`, whose reach `[1,1] ⊕ [1,2] = [2,2]` bridges
from the text start straight across the gap into link space (consultation Q11, Q19).

---

## The origin is permanent; the extent is a function of the extremes

The origin remains fixed for the life of the document: the home position is permanent, "any
address … may be specified by a permanent tumbler address" (4/19), while only the extent and
internal ordering shift under editing.

We can make this precise. While the content subspace is occupied, D-MIN pins
`min V_{s_C}(d) = [s_C,1,…,1]`, and since `s_C` is the least subspace identifier, this is
also `min O(d) = origin_d` whenever content is present. We record **V8** (origin
permanence): for every document state in which the content subspace is non-empty,
`origin_d = [s_C,1,…,1]`, invariant under all editing that leaves content present. (The depth
`m_C` is fixed throughout any content-present regime by the re-pinning discipline of ASN-0047's
`m_S(d)`: the content depth is re-pinned only after the content subspace is fully cleared, so it
holds constant across every state in which content remains present.) Editing
relocates I-addresses and shuffles V-positions, but it never moves the start of the stream:
"the front-end application is unaware" of where bytes natively live (4/11), and the V-origin
holds steady at the canonical first position. The origin is the stable anchor against which
every other V-address is read.

The extent is not the origin's opposite in any blanket sense; we must say exactly what it
depends on. Both endpoints consult only the *extremes* of the occupied set:
`origin_d = min O(d)` and `extent_d = shift(max O(d), 1) ⊖ origin_d` are functions of the
pair `(min O(d), max O(d))` and of nothing else — never the values `M(d)(v)`, never the
interior of `O(d)`. We record **V9** (the span is a function of the extremes). Two
consequences follow, one unconditional, one regime-dependent. *Rearrangement invariance.* A
pure rearrangement permutes `M(d)` while preserving `O(d) = dom(M(d))`; only the values
`M(d)(v)` are permuted, the extremes stand, and the reported span is *identical* before and
after: reorder the document and its origin and extent do not move. *Composition
sensitivity.* A composition change — adding or removing occupied positions — moves the span
*iff* it moves an extreme, and it need not. The two directions of this biconditional rest on
different facts, and we must discharge them separately. The forward direction — the span
moves only if an extreme moves — is the contrapositive of V9 itself: when both extremes
stand, the span, a function of the extremes alone, is identical. The converse — an extreme
moves only if the span moves — is not a functionality fact but an *injectivity* fact about
the map `(min O(d), max O(d)) ↦ (origin_d, extent_d)`, and it requires proof. We record it
as **V9a** (extremes recoverable) and argue by exhibiting an explicit inverse: the extremes
are computable from the returned pair, so distinct extreme pairs cannot yield equal spans.

The minimum is immediate: `min O(d) = origin_d`. The maximum we recover in two steps —
first the constructed endpoint `reach_d` from the returned pair, then `max O(d)` from
`reach_d`. Write `o = origin_d`, `e = extent_d`, `r = reach_d`; recall `#e = max(#o, #r)`
(TA2), and observe that both `o` and `r` are zero-free with all components positive — `o`
is an occupied position (V1, S8a), and `r = shift(max O(d), 1)` advances only the last
component of one (OrdinalShift). The recovery must split on the relative depth of the
endpoints, and the split is decided by the returned width itself, through its final
component:

> `e_{#e} > 0 ⟺ #o ≤ #r`.

*If `#o ≤ #r`*, the final position `#e = #r` of `e` receives, by TumblerSub, either the
action-point difference `r_{#r} − o_{#r}` (when `zpd(r, o) = #r`; positive because at the
zpd `r_k > o_k`, with `o` zero-padded beyond `#o` contributing `0` in the sub-case
`#o < #r`) or the tail copy `r_{#r}` (when `zpd(r, o) < #r`; positive by zero-freeness of
`r`) — positive either way. *If `#o > #r`*, unequal extreme depths force the cross-subspace
case (S8-depth makes single-subspace extremes equidepth) with `zpd(r, o) = 1` — the
derivation at V2's second covering case — so TumblerSub zero-pads `e` at every position
`#r < i ≤ #o`; in particular `e_{#e} = e_{#o} = 0`. The two cases of the recovery are then:

- *Final component positive* (`#o ≤ #r`). D1 applies — its preconditions `o < r` and
  `divergence(o, r) ≤ #o` were discharged at V2 — and the round-trip closes:
  `o ⊕ e = o ⊕ (r ⊖ o) = r`. The reach is recovered as `r = o ⊕ e`.
- *Final component zero* (`#o > #r`, so `#e = #o`). The round-trip fails (D0), and recovery
  goes componentwise through TumblerSub's formula at `zpd(r, o) = 1`: `e₁ = r₁ − o₁`,
  `eᵢ = rᵢ` for `2 ≤ i ≤ #r`, and `eᵢ = 0` for `#r < i ≤ #o`. Zero-freeness of `r` is now
  load-bearing: the copied components `e₂, …, e_{#r}` are all positive and the padded tail
  is all zero, so the rightmost nonzero position of `e` sits exactly at `#r` —
  `sig(e) = #r` (TA5-SIG). The depth of `r` is therefore read off `e`, and no
  zero-padded-equal collision across reach depths is possible: distinct reaches of distinct
  depths against the same origin produce widths with distinct `sig`. Componentwise,
  `r = [o₁ + e₁, e₂, …, e_{sig(e)}]`.

In both cases `r` is a function of `(o, e)`. Finally `max O(d)` is a function of `r` alone:
the shift is depth-preserving and advances exactly the final component (OrdinalShift), so
`max O(d)` agrees with `r` on positions `1..#r − 1` and has final component `r_{#r} − 1`,
well-defined in ℕ because `r_{#r} = (max O(d))_{#r} + 1 ≥ 2`. (Abstractly: TS2 gives
injectivity of the shift within a depth, and depth preservation plus T3 separates reaches
across depths; the explicit decrement is the inverse those two facts promise.) The
composite `(o, e) ↦ (min O(d), max O(d))` inverts the extremes-to-span map; the map is
injective, and an extreme cannot move while the span stands — the converse direction
holds. As a by-product, the case discriminator is V-ReachTight's condition read off the
returned value: the returned width's final component is positive exactly when
`reach(σ_d) = reach_d`.

With both directions discharged, the two regimes illustrate how often a composition change
actually moves an extreme. In the single-subspace regime every composition
change does: by D-SEQ★ the occupied set is the dense run `{[s,1,…,1,k] : 1 ≤ k ≤ n_s}`, so
any change to `n_s` moves `max O(d) = [s,1,…,1,n_s]` and with it the final component of
`extent_d`, which equals `n_s` (TumblerSub at `zpd = m_s` gives `extent_d = [0,…,0,n_s]`;
V12 reads the count from this identity). In the cross-subspace regime the extremes are the
content minimum and the link maximum, and a content-side composition change that keeps
content present (`n'_{s_C} ≥ 1`) touches neither: in the worked report below, a `K.μ⁻` with
retention counts `n'_{s_C} = 2`, `n'_{s_L} = 1` removes the content position `[1,3]` — the
composition changes — yet `min O(d) = [1,1]` and `max O(d) = [2,1]` stand, so `origin_d`,
`reach_d`, and `extent_d` are all identical before and after; likewise a content extension
adds `[s_C,1,…,1,n_C+1]`, still below every link position by T1, and the span again does
not move. The golden cases display the same insensitivity: ten characters plus one link
(consultation Q11, Q19) and the worked report's three characters plus one link both report
extent `[1,2]`.

V8's boundary is reached at exactly two points within the editing vocabulary
`{K.μ⁺, K.μ⁺_L, K.μ⁻, K.μ~}` (ASN-0047), symmetric across the content/link divide; everywhere
else in that vocabulary the origin holds. We record **V18** (origin migration bounds V8),
scoped to transitions that keep the document non-empty so the origin stays defined.
*Content-clearing* — a `K.μ⁻` contraction that empties the content subspace
(`V_{s_C}(d) = ∅`) while one or more links survive (`V_{s_L}(d) ≠ ∅`): V8's hypothesis fails
and `origin_d` migrates *up* from the content anchor `[s_C,1,…,1]` to the link minimum
`[s_L,1,…,1]` (D-MIN★ at `S = s_L`).
*First-content insertion* — a `K.μ⁺` extension into a link-only document (V5), where
`origin_d = [s_L,1,…,1]`: the first content position occupies `[s_C,1,…,1]`, and since
`s_C < s_L` the origin migrates *down* to the content anchor, restoring V8's regime (D-MIN★ at
`S = s_C`). The remaining transitions fix the origin by a uniform argument. Each leaves the
content-occupancy status unchanged: `K.μ~` (reordering) preserves `O(d)` wholesale; a `K.μ⁺`
into a content-present document and a `K.μ⁻` retaining at least one content position keep
`V_{s_C}(d) ≠ ∅`; `K.μ⁺_L` never touches `V_{s_C}(d)` — in particular a link-only document
stays link-only — and a `K.μ⁻` on a link-only document retaining at least one link
(`n'_{s_L} ≥ 1`) likewise keeps `V_{s_C}(d) = ∅` with `V_{s_L}(d) ≠ ∅`. Whichever subspace
the origin reads — the content anchor `[s_C,1,…,1]` when `V_{s_C}(d) ≠ ∅` (since `s_C < s_L`),
the link anchor `[s_L,1,…,1]` when link-only — that subspace remains non-empty across the
step, so its depth `m_S(d)` holds constant (the re-pinning discipline cited at V8) and
D-MIN★ pins its minimum at `[S,1,…,1]` of that depth in both the pre- and post-state. The
unchanged occupancy status selects the same pin before and after, and the pin itself does
not move, so `origin_d` is fixed. Gregory confirms the
content-clearing case: deleting all text while links remain is a permitted, non-empty state
reporting the link span (`2.1 for 0.1` in the golden link-only configuration), not the empty
result (deletion consultation).

---

## Every document answers, including the empty one

Nelson asks whether some documents have undefined origin and extent. The answer is no — and
the empty document is the case that tests it. `CREATENEWDOCUMENT` "creates an empty document"
(4/65); a freshly created or fully emptied document has `O(d) = ∅`.

We record **V11** (total answerability via a distinguished empty result): `RETRIEVEDOCVSPAN`
is defined for every allocated document. When `O(d) = ∅`, the result is the *empty span-set*
`⟨⟩` (V0). The empty
span-set carries no origin and no extent: `origin_d = min O(d)` is *undefined* when `O(d) = ∅`
(the minimum of the empty set does not exist), and there is no extent tumbler. This is the
honest content of the empty case — there is no first occupied position, hence no origin to
report. Nelson's span model admits exactly this absence: "a span that contains nothing today
may at a later time contain a million documents" (4/25). Emptiness is a *valid state of the
address space*, not an undefined result. Gregory's
implementation realizes the distinguished value by returning zeros for both displacement and
width when the arrangement tree holds no content, independent of any residual tree structure
left by prior deletions (consultation Q13). We read those zeros as a *sentinel* — an encoding
of "no origin, no extent" — and not as a legal tumbler: the zero tumbler is precisely the
value TA6 forbids as an address.

---

## What the caller learns beyond the name

We record **V12** (information gain): the result of `RETRIEVEDOCVSPAN(d)` determines
time-varying facts about the arrangement that the permanent identity `d` cannot, because `d`
is the query's fixed argument — a document address that persists unchanged across every
transition (P1, EntityPermanence, ASN-0047) — while the result is recomputed against the
present state. Concretely, the returned span-set decides emptiness by the presence or absence of a
component span (`RETRIEVEDOCVSPAN(d) = ⟨⟩ ⟺ O(d) = ∅`, V11) and, when non-empty in the
single-subspace regime, its span `σ_d` fixes the occupied count exactly, and the count is
read off the returned value itself. The direct route is through the width: the endpoints
share depth `m_s` and agree on positions `1..m_s−1` (the value `s` followed by 1's), so
TumblerSub at `zpd = m_s` gives `extent_d = [0,…,0,n_s]`, and the final component of the
returned width *is* `n_s = |O(d)|` (D-SEQ★; the worked report's `0.3` is this identity at
`n_s = 3`). Equivalently, because single-subspace endpoints are equidepth, V-ReachTight
licenses computing `reach_d = origin_d ⊕ extent_d = reach(σ_d)` from the returned pair, and
`n_s + 1` is its final component. The identity `d` — invariant under every edit — reports
none of these.

---

## Independence, permanence, and stability

Three faithfulness questions remain, all about how the report relates to *other* state.

**Per-document independence.** Suppose two documents `d₁` and `d₂` share content — the same
I-address occupies a position in each. We record **V13** (independence): `σ_{d₁}` depends
only on `O(d₁) = dom(M(d₁))`, and `σ_{d₂}` only on `O(d₂)`; neither defers to, inherits from,
or is altered by the other. Shared content is referenced once in the store but belongs fully
to each document's own arrangement: a transcluded position "has an ordinal position in the
byte stream just as if it were native" (4/11) and counts toward *that* document's extent. So
`RETRIEVEDOCVSPAN(d₁)` and `RETRIEVEDOCVSPAN(d₂)` report distinct, independently computed
spans even over identical content — "no arrangement … is a priori better than other
arrangements" (2/19), and each document answers for its own bounds on its own terms.

**Permanence of the underlying content.** We record **V14** (permanence): every *occupied*
position in `O(d)` — every position the span covers that actually carries content — maps,
through `M(d)`, to a permanent, immutable image, the store depending on the position's
subspace (S3★). A *content* position (`subspace(v) = s_C`) maps to an I-address in `dom(C)`,
permanent and immutable by content permanence (S0, P0); a *link* position
(`subspace(v) = s_L`) maps to a link address in `dom(L)`, permanent and immutable by link
permanence (L12). The arrangement (Vstream) is fluid; the content
identities it references are eternal. So even when the originating owner "deletes" content
from this document's current version, the underlying bytes persist (the 4/11
deletion-permanence point cited at V4) — sharing strengthens rather than threatens the
permanence of what any reported span ultimately denotes.

**Determinism.** We record **V16** (determinism): `σ_d` is a pure function of `O(d)`, so two
queries against an unchanged arrangement return identical spans, and the returned span is a
*snapshot* — a value fixed at the instant of the query, not a live view — so a later edit to
`d` (or to any document supplying `d`'s transcluded content) is reported only by a fresh
query, never by mutation of the already-returned value. Gregory grounds this — the reported
bounds are computed from a width summary that the arrangement tree maintains *independent of
the physical tree's shape* (enfilade confluence), so the answer depends only on the logical
arrangement, never on how the structure was built or rebalanced (consultation Q14).

---

## Implementation conformance: the extent stays non-negative

*Implementation remark (conformance to V2).* Prior deletions can drive *intermediate*
arrangement-tree entries to negative displacements, but the root width is recomputed as a
maximum-minus-minimum reach and remains non-negative, so no editing transient surfaces a
zero-or-below extent (consultation Q18) — consistent with V2's positivity (`Pos(extent_d)`).

*Implementation remark (reach tightness, evidence for V-ReachTight).* The
implementation in fact realizes only `m_C = m_L`: content and link V-positions are placed at
the same depth — both depth 2 — distinguished only by the first-component value `s_C = 1` vs
`s_L = 2`, never by depth (consultation Q2: `findvsatoappend`, `findnextlinkvsa`, and
`setlinkvsas` all emit depth-2 V-addresses). The cross-subspace endpoints are therefore
level-compatible (`#origin_d = #reach_d`), so V-ReachTight fires affirmatively
and `reach(σ_d) = reach_d` exactly.

---

## A worked report

Take the document `d = [1.0.1.0.5]` (a document-level tumbler, `zeros(d) = 2`). Give its
content subspace three positions and its link subspace one:

> `M(d) = { [1,1] ↦ a, [1,2] ↦ b, [1,3] ↦ a, [2,1] ↦ ℓ }`,

where `a, b` are content I-addresses and `ℓ` is a link I-address. The occupied set is
`O(d) = {[1,1], [1,2], [1,3], [2,1]}`, totally ordered by T1 as written (since `1 < 2` at the
first component).

Compute the span. `origin_d = min O(d) = [1,1]`. `max O(d) = [2,1]`, so
`reach_d = shift([2,1], 1) = [2,2]`. The extent is `[2,2] ⊖ [1,1]`: the tumblers first differ
at position 1 (`2 ≠ 1`), so `extent_d = [2-1, 2] = [1,2]`. Thus

> `RETRIEVEDOCVSPAN(d) = ⟨([1,1], [1,2])⟩`,  i.e. the singleton span-set "1.1 for 1.2".

Verify the claims. **V1**: `origin_d = [1,1] ∈ O(d)`, an occupied content position. ✓
**V2**: `⟦σ_d⟧ = {t : [1,1] ≤ t < [2,2]}` contains all four occupied positions. ✓
**V6**: it *also* contains the occupied-depth position `[1,4]` — the witness `w⋆`
(`n_C = 3`), covered but unoccupied — along with `[1,5], …` in the inter-subspace void: a
bounding box, not an exact cover. ✓ **V2** (T12 legality): `extent_d =
[1,2]` is positive with `actionPoint = 1 ≤ 2 = #origin_d`. ✓

Now drop the link, leaving `O'(d) = {[1,1], [1,2], [1,3]}`. Then `origin_d = [1,1]` (V8,
unchanged), `max = [1,3]`, `reach = [1,4]`, `extent = [1,4] ⊖ [1,1] = [0,3]`, giving
`⟨([1,1], [0,3])⟩` — "1.1 for 0.3", an exact cover of three contiguous positions (V5), with the
origin fixed exactly where it was (V8). Reordering these three positions — permuting which
I-address sits at each — leaves `O'(d)` unchanged and so returns the identical span (V9).

**An endpoint-depth-divergent variant (one line).** When `m_C = 3 > m_L = 2`:
`M(d) = { [1,1,1] ↦ a, [1,1,2] ↦ b, [2,1] ↦ ℓ }` gives `origin_d = [1,1,1]`, `reach_d = [2,2]`,
`extent_d = [1,2,0]` of depth 3, and the actual reach `r⋆ = [2,2,0]` overshoots `reach_d` exactly
as V2's second covering case predicts (coverage and T12 legality survive; what lapses is
V-ReachTight `reach(σ_d) = reach_d` — V3's same-depth tightness of
`reach_d` relative to `max O(d)` is intact, since `reach_d = [2,2]` remains the least strict
same-depth upper bound of `max O(d) = [2,1]`).

**The mirror variant (one line).** When `m_C = 2 < m_L = 3` — the one regime where
V-ReachTight holds while V-LevelUniform fails (the two properties part ways in both
unequal-depth regimes — the depth-divergent variant above realizes the opposite split — and
this is the remaining realizable quadrant, `(¬Tight, ¬LU)` being impossible since
`#origin_d > #reach_d` and `#origin_d < #reach_d` exclude each other):
`M(d) = { [1,1] ↦ a, [2,1,1] ↦ ℓ }` gives
`origin_d = [1,1]`, `max O(d) = [2,1,1]`, `reach_d = [2,1,2]`, and
`extent_d = [2,1,2] ⊖ [1,1] = [1,1,2]` of depth 3 (TumblerSub at `zpd = 1`); the round trip
*closes* — `r⋆ = [1,1] ⊕ [1,1,2] = [2,1,2] = reach_d` by D1 at `#origin_d = 2 ≤ 3 = #reach_d`,
so V-ReachTight fires affirmatively — yet `#origin_d = 2 < 3 = #extent_d` breaks S6, and the
returned span is strictly non-level-uniform (V-LevelUniform's negative branch); both width
discriminators read correctly off the result: `extent_d₁ = 1 > 0` flags the bounding box
(V9b) and the final component `extent_d₃ = 2 > 0` flags the tight reach (V9a).

---

## Preconditions and well-definedness

For the report to be defined we require:

1. `d ∈ dom(M)` — the document is allocated (M0, M1). An unallocated identity names no
   arrangement and has nothing to report.

Authorization is a deployment-level access gate outside the value semantics this ASN
specifies.

Under precondition 1 the result is total: by S8-fin the occupied set is finite, so its
minimum and maximum (when non-empty) exist and the span is computed by V1–V2; when empty the
result is the distinguished empty span-set `⟨⟩` (V11), carrying no origin and no extent. No
further argument is needed — the operation consumes no caller-supplied position, so there is
no range to validate.

The precondition for *legality* is trivial, but a caller wanting to know *what kind* of answer
it will get — a faithful trace or a mere bounding box — is asking a non-trivial weakest-
precondition question, and we can answer it. Take the distinguished result
property `Exact ≡ "⟦σ_d⟧ contains no occupied-depth position outside O(d)"` (occupied-depth as
defined at V5; vacuously true on
the `⟨⟩` result, where there is no `σ_d`). Reasoning backward from `Exact`, we ask which states
`Σ` guarantee it. We claim

> `wp(RETRIEVEDOCVSPAN(d), Exact) = (O(d) occupies at most one subspace)`.

The derivation runs through V5 and V6. If `O(d)` is empty the result is `⟨⟩` and `Exact` holds
vacuously by definition; if `O(d)` lies in a single subspace `s`, V5 gives `Exact` directly: the
dense run `{[s,1,…,1,k]}` is covered with no occupied-depth position left over (V5's
prefix-pinning and boundary-discreteness argument). Conversely, if `O(d)` occupies
*both* subspaces, V6 supplies an occupied-depth witness `w⋆ ∈ ⟦σ_d⟧ \ O(d)` at the occupied
content depth `m_C`, so `¬Exact`. The two directions exhaust the cases by S3★-aux. So the single-subspace condition is both necessary and sufficient, hence
the *weakest* precondition. The companion reach property factors the same way along the
orthogonal endpoint axis. The contingent tightness property — analogous to `Exact` —

> `Tight ≡ "reach(σ_d) = reach_d"`  (the delivered span's denotational reach attains the
> constructed endpoint),

is true in some states and false in others (vacuously true on the `⟨⟩` result, where there
is no `σ_d`). The backward reasoning here needs no fresh case analysis: V-ReachTight already
establishes `reach(σ_d) = reach_d ⟺ #origin_d ≤ #reach_d` for the non-empty case, and the
empty case is vacuous. Disjoining the empty-result branch with V-ReachTight's condition gives
the *weakest* precondition directly:

> `wp(RETRIEVEDOCVSPAN(d), Tight) = (O(d) = ∅ ∨ #origin_d ≤ #reach_d)`.

A caller can thus decide *before* querying whether the answer will be exact
(check single-subspace occupancy) and whether its reach is the tight `reach_d` (check the
endpoint depths), without inspecting the returned span. Both properties are equally decidable
*after* the fact, from the returned width alone, and the two discriminators sit at opposite
ends of that width. *Tightness* reads the final component: the width's final component is
positive exactly when the reach is tight (the V9a discriminator). *Exactness* reads the first
component; we record **V9b** (exactness discriminator): for any non-empty result,

> `extent_d₁ = 0 ⟺ Exact`.

The derivation is two lines on the `zpd` case split already in hand. In the single-subspace
case the endpoints agree on positions `1..m_s−1`, so they first diverge at
`zpd(reach_d, origin_d) = m_s ≥ 2` (S8a), and TumblerSub zeroes every position below the
action point — in particular `extent_d₁ = 0`. In the cross-subspace case the endpoints
diverge already at position 1 (`s_C` vs `s_L`, the divergence established for T12 legality),
so `zpd = 1` and `extent_d₁ = reach_d₁ − origin_d₁ = s_L − s_C ≥ 1 > 0`. Since
single-subspace occupancy is exactly `wp(RETRIEVEDOCVSPAN(d), Exact)` (V5, V6), the first
component is zero precisely on the exact covers; on the empty result `⟨⟩` there is no width
to test, matching `Exact`'s vacuous truth there. The worked report displays both
discriminators at once: the cross-subspace width `[1,2]` opens with `1` (bounding box) and
closes with `2 > 0` (tight reach); the content-only `[0,3]` opens with `0` (exact cover) and
closes with `3 > 0` (tight reach); the depth-divergent variant's `[1,2,0]` opens with `1` and
closes with `0` (bounding box, with overshooting reach); the mirror variant's `[1,1,2]` opens
with `1` and closes with `2 > 0` (bounding box with tight reach — the non-level-uniform
quadrant).

---

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| V-frame | `Σ' = Σ` — the query mutates no state component (`C, L, E, M, R` all unchanged) | introduced |
| V0 | `RETRIEVEDOCVSPAN : dom(M) → SpanSet` (uniform ASN-0053 span-set codomain): the singleton span-set `⟨σ_d⟩` carrying one well-formed span `σ_d = (origin_d, extent_d)` for a non-empty document, or the empty span-set `⟨⟩` (denoting `∅`) when `O(d) = ∅` — never a content sequence, never a count | introduced |
| V1 | When `O(d) ≠ ∅`, `origin_d = min O(d)` under T1 and `origin_d ∈ O(d)` (the origin is an occupied position) | introduced |
| V2 | `O(d) ⊆ ⟦σ_d⟧` (coverage); the actual reach `r⋆ = origin_d ⊕ extent_d ≥ reach_d = shift(max O(d), 1) > max O(d)`; the span `(origin_d, extent_d)` is always a well-formed T12 span | introduced |
| V3 | `origin_d` is the greatest lower bound of `O(d)`; the *constructed endpoint* `reach_d` is the least strict upper bound of `max O(d)` among tumblers at the depth of `max O(d)` | introduced |
| V-ReachTight | `reach(σ_d) = reach_d ⟺ #origin_d ≤ #reach_d` — the denotational reach attains the constructed endpoint exactly when origin depth does not exceed reach depth; automatic in the single-subspace regime | introduced |
| V-LevelUniform | `σ_d` is level-uniform (S6: `#origin_d = #extent_d`) `⟺ #origin_d ≥ #reach_d`, since `#extent_d = max(#origin_d, #reach_d)` (TA2); always level-uniform in the single-subspace regime | introduced |
| V4 | `extent_d` is computed from `O(d) = dom(M(d))` alone; content in `dom(C)` but absent from the arrangement (deleted, or native elsewhere) contributes nothing (Vstream-bounded, not Istream) | introduced |
| V5 | When all occupied positions share one subspace, `⟦σ_d⟧` contains no occupied-depth position outside `O(d)`, where `t` is *occupied-depth* at `(Σ, d)` iff `#t = m_S(d)` for some `S ∈ {s_C, s_L}` with `V_S(d) ≠ ∅` (exact cover of a contiguous run) | introduced |
| V6 | When occupied positions span more than one subspace, `⟦σ_d⟧` contains an occupied-depth position outside `O(d)` (witness `w⋆ = [s_C,1,…,1,n_C+1]` at depth `m_C`) — the negation of V5 (bounding box, not exact cover); corollary: `O(d) ⊊ ⟦σ_d⟧`; forced because a span denotes one convex region (ASN-0053 S0) and cannot trace a separated series | introduced |
| V8 | While the content subspace is non-empty, `origin_d = [s_C,1,…,1]`, invariant under all editing that leaves content present (origin permanence) | introduced |
| V9 | `origin_d` and `extent_d` are functions of the extremes `(min O(d), max O(d))` alone — never of the values `M(d)(v)` or the interior of `O(d)`; a pure rearrangement (preserving `O(d)`) returns the identical span, and a composition change moves the span iff it moves an extreme (forward direction by functionality of the extremes-to-span map, converse by V9a) — every composition change moves an extreme in the single-subspace regime (final component of `extent_d` equals `n_s`), but not in general in the cross-subspace regime, where content-side changes keeping `n'_{s_C} ≥ 1` leave the extremes, hence the span, fixed | introduced |
| V9a | The extremes-to-span map `(min O(d), max O(d)) ↦ (origin_d, extent_d)` is injective, by explicit inverse: `min O(d) = origin_d`; `reach_d = origin_d ⊕ extent_d` when the returned width's final component is positive (`⟺ #origin_d ≤ #reach_d`, D1 round-trip), else `#reach_d = sig(extent_d)` and `reach_d` is read componentwise off TumblerSub's formula at `zpd = 1` (zero-freeness of `reach_d`, S8a); `max O(d)` is `reach_d` with final component decremented (OrdinalShift, TS2) — so a moved extreme always moves the span (extremes recoverable) | introduced |
| V9b | For any non-empty result, `extent_d₁ = 0 ⟺ Exact` — single-subspace endpoints first diverge at `zpd(reach_d, origin_d) = m_s ≥ 2` (S8a), so TumblerSub zeroes position 1 of the width; cross-subspace endpoints diverge at position 1, so `extent_d₁ = s_L − s_C ≥ 1` (TumblerSub at `zpd = 1`); the V5/V6 dichotomy identifies the zero case with `Exact` — the slot-1 partner of V9a's final-component tightness discriminator (exactness discriminator) | introduced |
| V11 | The operation is total over allocated documents; `O(d) = ∅` yields the distinguished empty span-set `⟨⟩` (V0), with `origin_d` undefined and no extent — the implementation's zeros are a sentinel, not a legal address (TA6) | introduced |
| V12 | The result of `RETRIEVEDOCVSPAN(d)` determines time-varying arrangement facts that the permanent identity `d` cannot: the returned span-set decides emptiness (`RETRIEVEDOCVSPAN(d) = ⟨⟩ ⟺ O(d) = ∅`, V11) and, when non-empty in the single-subspace regime, its span `σ_d` fixes the exact occupied count `|O(d)| = n_s` — the final component of the returned width `extent_d = [0,…,0,n_s]` (TumblerSub at `zpd = m_s`), equivalently the final component of `reach(σ_d) = reach_d` less one (V-ReachTight); `d` is invariant under every edit (P1, EntityPermanence, ASN-0047) and reports none of these (information gain) | introduced |
| V13 | `σ_d` depends only on `O(d)`; two documents sharing content report independent spans; transcluded positions count toward the borrowing document's extent (independence) | introduced |
| V14 | Every *occupied* position in `O(d)` maps through `M(d)` to a permanent, immutable image, by subspace (S3★): content positions to `dom(C)` (S0, P0), link positions to `dom(L)` (L12); covered-but-unoccupied positions in the cross-subspace case (V6) carry no `M(d)` image; sharing preserves what the span denotes (permanence) | introduced |
| V16 | `σ_d` is a pure function of `O(d)`; equal arrangements return identical spans, independent of how the arrangement was built; the returned span is a snapshot, not a live view (determinism) | introduced |
| V18 | Within the non-empty-preserving editing vocabulary `{K.μ⁺, K.μ⁺_L, K.μ⁻, K.μ~}` (ASN-0047), V8's origin moves only at the two content-occupancy-toggling transitions: a `K.μ⁻` content-clearing migrates `origin_d` up to the link minimum `[s_L,1,…,1]`, a `K.μ⁺` first-content insertion into a link-only document migrates it down to the content anchor `[s_C,1,…,1]`; `K.μ⁺_L`, `K.μ~`, and occupancy-preserving `K.μ⁺`/`K.μ⁻` fix the origin (origin migration bounds V8) | introduced |

## Open Questions

In the *multi-subspace* case — where the inter-subspace void places unoccupied positions between the endpoints — what invariant, if any, can relate the reported extent to the count of occupied positions, given that the dense single-subspace coincidence (final component `= |O(d)|`, settled by V5) fails there and a span designates boundaries, not a cardinality?

Under what conditions must the reported origin be the document's permanent tumbler identity rather than the minimum occupied V-position, and when do these coincide?

What faithfulness must a report of a designated historical version preserve relative to a report of the present arrangement of the same document?

What invariant must relate the whole-document span to the bounding spans of the document's individual correspondence runs, so that the global extent composes from local ones?

What must the report guarantee about origin and extent when content occupies V-positions whose addressing arithmetic has been driven outside the well-formed range by prior editing?

Under what conditions can a well-formed covering span's denotational reach attain the T1-immediate successor of the maximal occupied position?
