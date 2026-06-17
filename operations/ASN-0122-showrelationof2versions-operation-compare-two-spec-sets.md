> **ASN-0122 · SHOWRELATIONOF2VERSIONS — Correspondence Between Two Spec-Sets** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0122-showrelationof2versions-operation-compare-two-spec-sets.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0122: SHOWRELATIONOF2VERSIONS — Correspondence Between Two Spec-Sets

*2026-06-12*

## The Problem

Nelson specifies the operation's deliverable in a single sentence: "This returns a list of ordered pairs of the spans of the two spec-sets that correspond" [LM 4/70]. He also tells us what the deliverable is *for*: a facility holding multiple versions "is not terribly useful unless it can help you intercompare them in detail — unless it can show you, word for word, what parts of two versions are the same" [LM 2/20].

We are therefore looking for three things. First, the *relation*: what fact about the state makes a position of one spec-set "the same as" a position of the other. Second, the *report*: what an ordered pair of spans is, and exactly what a reader may conclude from one. Third, the *binding* between them: what the returned list must contain, what it must not contain, how it must be arranged, and what keeps it true — when the comparison is restricted to portions of documents rather than full extents, when a spec-set is compared with itself, when the shared material reached the two operands through a chain of intermediaries, and when either operand is subsequently rearranged.

A word on the standard we hold ourselves to. A line-diff utility *guesses* an alignment between two texts; its output is a heuristic, and "complete" is not a property one can demand of it. We will find that in this system correspondence is not a judgment to be approximated but a structural fact to be read off the state. Completeness and soundness then become obligations, not aspirations.

## State, Instances, and Spec-Sets

We work over the extended state `Σ = (C, L, E, M, R)` of the foundations: the immutable content store `C`, the link store `L`, the entity set `E` with its document stratum `E_doc`, the per-document arrangements `M(d)`, and the provenance relation `R`. The arrangement invariants we lean on throughout are functionality (S2), referential integrity (S3★), well-formedness and uniform depth of V-positions (S8a, S8-depth), finiteness (S8-fin), and the canonical sequential shape of each non-empty subspace (D-SEQ★): `V_S(d) = {[S, 1, …, 1, k] : 1 ≤ k ≤ n_S}` at the subspace's common depth `m_S(d)`. We adopt ASN-0058's OrdinalShiftBase convention: for a tumbler `t` and natural `k ≥ 0`, `t + k` denotes `shift(t, k)`, with `t + 0 = t`.

The operation compares *positions*, and a position is meaningful only relative to the document that arranges it. So our first definition names that unit.

**Definition (Inst — PositionInstance).** `Inst_Σ = {(d, v) : d ∈ E_doc ∧ v ∈ dom(Σ.M(d))}` — the currently-arranged positions of all documents, each tagged with its document. We write `Inst_C` for the content-subspace instances (`subspace(v) = s_C`) and `Inst_L` for the link-subspace instances; by S3★-aux these two classes exhaust `Inst_Σ`.

Every instance resolves to a stored address, and the resolution is single-valued because arrangements are functions (S2):

**Definition (res — Resolution).** `res_Σ(d, v) = Σ.M(d)(v)`, total on `Inst_Σ`. By S3★ its value lies in `dom(C)` for content instances and in `dom(L)` for link instances.

A *spec-set* is the operand syntax: a finite set of documents, each carrying a finite V-span-set (ASN-0053) describing which of its positions participate. Spans are the right description language because arrangement domains are order-convex per subspace (D-CTG★) and spans denote order-convex ranges (T12).

**Definition (Spec-set and region).** A spec-set is a finite set `ρ = {(d₁, S₁), …, (d_j, S_j)}` with each `d_i ∈ E_doc` and each `S_i` a finite set of T12-well-formed spans. Its *region* at Σ is

`R_Σ(ρ) = (∪ i :: {d_i} × (⟦S_i⟧ ∩ V_{s_C}(d_i)))`

First, the intersection with the arrangement domain means a span clips itself against what the document currently arranges: tumblers a span denotes but the document does not currently occupy contribute no instance — the question "which parts correspond?" is asked of the documents *as they stand*. Second, that domain is the *content-subspace* slice `V_{s_C}(d_i)`, not the full arrangement, so the same clip confines every region to content instances — X9 shows this discards no correspondence. A T12-well-formed span pins only the subspace of its *start*: `subspace(start) = s_C` fixes the first component `s₁`, but it does not pin the subspace of every position the span *denotes*. A span whose width acts below `#s` crosses the subspace boundary — with the fixed identifiers `s_C = 1`, `s_L = 2`, the span `σ = ([1,5], [3])` is T12-well-formed (`Pos([3])`, `actionPoint([3]) = 1 ≤ #[1,5] = 2`) yet `reach(σ) = [1,5] ⊕ [3] = [4]`, so `⟦σ⟧` contains `[2,7]`, a link-subspace position. The `∩ V_{s_C}(d_i)` discards every such position before it becomes an instance, so a content-only region is delivered whatever the operand spans denote.

Whole-document comparison is not a separate notion. When `V_{s_C}(d) ≠ ∅`, D-SEQ★ gives positions `[s_C, 1, …, 1, k]` for `1 ≤ k ≤ n` at depth `m`, and the single span `σ_full = ([s_C, 1, …, 1], δ(n, m))` is T12-well-formed with `⟦σ_full⟧ ∩ V_{s_C}(d) = V_{s_C}(d)`: the span runs from the minimum position up to but not including its `n`-fold shift, so every depth-`m` position with final component in `1 … n` lies inside, and everything else is removed by the intersection. "Compare the two documents" is thus the window instance `ρ_i = {(d_i, {σ_full})}`, and nothing we prove about windows needs a whole-document special case. Both operands may name the same document; a spec-set may also name several documents, in which case its region simply unions their windows.

When `V_{s_C}(d) = ∅` no `σ_full` exists — `δ(n, m)` requires `n ≥ 1` — and the whole-document operand for an empty document is `(d, ∅)`, carrying the empty span-set. More generally a spec-set may itself be empty (`j = 0`), a span may clip to nothing against the current arrangement, and one or both regions may come out empty. All of these are legal operands, and they answer uniformly: with `P = ∅` or `Q = ∅` the rectangle `P × Q` is empty, so the relation defined below is the empty relation `corr = ∅`; the chain partition of X11 exists and is unique vacuously; and the canonical report is the empty list `⟨⟩`, which conforms. An empty report is a meaningful answer — no part of one region is the same as any part of the other — not an error condition.

## What "Correspond" Must Mean

We now derive the relation rather than posit it. Suppose the report hands us a pair asserting that position `p` of the first region and position `q` of the second are "the same part." For intercomparison to be trustworthy, the assertion must satisfy three demands.

*It must certify, not resemble.* Reading either side must be reading the same stored thing, not a lookalike. *It must be durable.* The assertion must concern state that editing cannot silently rewrite. *It must be local.* It claims to be a fact about the two spec-sets, so it must be checkable from their arrangements.

There are two candidate predicates on `(p, q)`: value equality, `C(res p) = C(res q)`, and address equality, `res p = res q`. We test each.

Value equality fails certification. By S4 (origin-based identity) distinct allocation events yield distinct addresses *regardless of stored values*, and nothing in K.α constrains values to be distinct across events — so a state is reachable in which two independently created passages coincide byte for byte. A value-based relation cannot distinguish genuine sharing from coincidence. Nelson is emphatic on this point: identity in this system is established by creation, not by value; the machine traces provenance, not resemblance. Value comparison also fails locality — it must open the store — and we shall see that the implementation's most striking property is that it never does.

Address equality satisfies all three demands. *Certification:* the two feet are bound to one store entry, so by functionality of `C` there is one value present, not two equal ones — "word for word the same" becomes entailment rather than a check. *Durability:* allocated addresses are permanent (T8), their bindings immutable (P0/S0), and every arrangement edit transports addresses verbatim — we prove the transport theorems below. *Locality:* equality of tumblers is decided by the intrinsic comparison (T2, T3) from the two arrangement restrictions alone.

So we define:

**Definition (corr — Correspondence).** For finite instance sets `P, Q ⊆ Inst_Σ`:

`corr_Σ(P, Q) = {(p, q) ∈ P × Q : res_Σ(p) = res_Σ(q)}`

`corr` is the *kernel* of the resolution map — the equivalence "resolves to the same address" — intersected with the operand rectangle `P × Q`.

**X1 (IdentityBasis).** `(p, q) ∈ corr_Σ(P, Q) ⟺ res_Σ(p) = res_Σ(q)`; on content instances the shared address `a` lies in `dom(C)` (S3★), and both feet denote the single stored value `C(a)`. Value identity is entailed by membership and never consulted to decide it — the defining comprehension mentions `res` and nothing else. ∎

**X2 (CoincidenceExclusion).** There are reachable states containing instances `p ≠ q` with `C(res p) = C(res q)` and `res p ≠ res q`; every such pair is excluded from `corr`. *Construction.* Fix documents `d₁, d₂ ∈ E_doc` — pre-existing, or each created by a K.δ step, itself a valid composite (it allocates no content, so J0, J1★, and J1'★ hold vacuously). K.α constrains the address it allocates, never the value stored, so for each `i` run one valid composite in the sense of ValidComposite★: K.α deposits the same `v ∈ Val` at a fresh `a_i`, K.μ⁺ installs `a_i` at a content-subspace position of `M(d_i)` (its precondition `a_i ∈ dom(C)` holds at the intermediate state), and K.ρ records `(a_i, d_i)`. Between the composite's endpoints J0 holds — the freshly allocated address is arranged in the post-state — and J1★/J1'★ hold — the range-new address and the new provenance entry are one and the same pair — so each composite is valid and the final state is reachable. Being distinct allocation events, `a₁ ≠ a₂` by S4 — "regardless of whether `C(a₁) = C(a₂)`" — with GlobalUniqueness (ASN-0034) behind it. The resulting instances resolve to distinct addresses and do not correspond. ∎

X2 is the negative half of the basis, and it is normative: an implementation that matched by value would over-report, and its pairs could no longer be read as "the same part." Human-judged equivalence — translation, paraphrase, parallel formulation — is a different mechanism, an asserted relation owned by its asserter and no part of this operation.

**X3 (Symmetry).** `corr_Σ(Q, P) = corr_Σ(P, Q)⁻¹`. Equality of addresses is symmetric, so swapping the operands transposes every member. The *within-pair* order is therefore semantic, not decorative: slot `i` of a reported pair draws from operand `i`, and that convention is exactly what makes the converse statement contentful. ∎

Equality of addresses is also transitive: `res p = res q` and `res q = res r` give `res p = res r` — correspondence composes through a shared instance.

## Which Positions May Participate

We restricted regions to the content subspace. We must say exactly what the restriction discards — and the answer will be: no sharing. The link subspace, if admitted, would contribute no cross-position and no cross-document pair; its entire contribution is a forced self-diagonal, fixed by the operand regions without consulting the resolution map. Consider the unrestricted instance space and ask which pairs involving link instances can satisfy `res p = res q`.

*Cross-document link instances never correspond.* A link-subspace position arranges one of the document's own links: CL-OWN gives `origin(M(d)(v)) = d`. If `(d₁, v₁)` and `(d₂, v₂)` are link instances sharing address `ℓ`, then `d₁ = origin(ℓ) = d₂` — `origin` is a projection of the address itself (S7), hence single-valued — so the documents coincide.

*A content instance never corresponds to a link instance.* The first resolves into `dom(C)`, the second into `dom(L)` (S3★), and the stores are disjoint (SD/L14): one address cannot inhabit both.

*Same-document link instances correspond only to themselves.* Within one document the link-subspace restriction of `M(d)` is injective (CL-UNIQ), so a shared address forces `v₁ = v₂`.

**X9 (SubspaceVacuity).** Over unrestricted instances, `corr` decomposes as the content-subspace relation, disjointly unioned with the forced diagonal `{((d, v), (d, v)) : (d, v) ∈ P ∩ Q ∩ Inst_L}`. By the three arguments above, for any pair with a link-instance foot the predicate `res p = res q` reduces to instance equality `p = q` — so the diagonal is determined by `P ∩ Q` alone, with the resolution map never consulted. We state the loss precisely. The discarded pairs do carry arrangement-domain information: that each such `(d, v)` is an instance at all is exactly what the region's clipping against `dom(M(d))` certified. What they carry none of is correspondence information — no sharing between distinct positions, distinct documents, or the two operand sides is expressible by a link-subspace foot. In that sense, and that sense exactly, the content-subspace restriction is lossless for this operation. ∎

The content-subspace restriction is thus *semantic*, not defensive: over the link subspace the question "which parts are the same?" has only the empty or the reflexive answer.

## The Relation Is Finite, Decidable, and Local

**X0 (RelationWellFormed).** `corr_Σ(P, Q)` is a finite relation, and membership is decidable from the operand representations and the two arrangement restrictions alone. *Proof.* Each region is a subset of finitely many finite arrangement domains (S8-fin), so `P × Q` is finite. Membership is tumbler equality (T3), decided by the intrinsic comparison procedure (T2) on the two resolved addresses, which the restrictions `res|P` and `res|Q` supply. ∎

The locality half deserves its own name, because it is the load-bearing frame property of the entire specification:

**X5 (Locality).** `corr_Σ(P, Q)` is determined by the two restricted resolution maps `(res_Σ|P, res_Σ|Q)`: any two states agreeing on these maps return the same relation, and neither map can be dropped. The regions are not independent data — `P = dom(res_Σ|P)` and `Q = dom(res_Σ|Q)` — so the four-tuple `(P, Q, res_Σ|P, res_Σ|Q)` is a presentational expansion of the same two maps. *Proof.* Membership of `(p, q)` in `corr_Σ(P, Q)` is, by the definition of `corr`, the conjunction `p ∈ P ∧ q ∈ Q ∧ res_Σ(p) = res_Σ(q)`. The first two conjuncts read only `P` and `Q`, the domains `dom(res_Σ|P)` and `dom(res_Σ|Q)`. In the third, `p ∈ P` gives `res_Σ(p) = (res_Σ|P)(p)` and `q ∈ Q` gives `res_Σ(q) = (res_Σ|Q)(q)`, so the equality test consults the two restricted maps and nothing else — not `C`, `L`, `E`, `R`, nor `M(d)` at any position outside the regions. Hence if `Σ` and `Σ′` satisfy `res_Σ|P = res_{Σ′}|P` and `res_Σ|Q = res_{Σ′}|Q` — equal as maps, hence equal in domain — every membership test agrees and `corr_Σ(P, Q) = corr_{Σ′}(P, Q)`. The dependence is exact: changing `res|P` at a single instance `p` — toward or away from some `res(q)` — adds or deletes the pair `(p, q)`, so neither map can be dropped. The relation therefore factors precisely through `(res_Σ|P, res_Σ|Q)`. ∎

Three consequences, each answering a question the operation's user will ask.

*Determinism.* Two evaluations against the same restrictions return the same relation; there is no hidden input and no nondeterminism to average over.

*Indifference.* Any transition that leaves both restrictions intact leaves the relation intact: content allocation (K.α), link allocation (K.λ), provenance recording (K.ρ), entity creation (K.δ), and every arrangement edit confined to other documents or to positions outside the regions — each carries a frame clause guaranteeing exactly this. The comparison of two documents cannot be perturbed from anywhere else in the docuverse.

*Memorylessness.* The relation consults present arrangements only. A mapping contracted away has no instance and contributes nothing; no record of past arrangement states — wherever the system may keep such records — can manufacture a pair.

## Windows: Restriction Is Exact

The operands are windows onto documents, so we must say how the relation behaves under windowing. The answer is the cleanest one possible.

**X4 (WindowRestriction).** For `P′ ⊆ P` and `Q′ ⊆ Q`:

`corr_Σ(P′, Q′) = corr_Σ(P, Q) ∩ (P′ × Q′)`

*Proof.* Both sides are comprehensions over the same predicate, restricted to nested rectangles. ∎

Three corollaries unpack what this buys. *Monotonicity:* shrinking windows can only shrink the relation. *Compositionality:* `corr(P ∪ P″, Q) = corr(P, Q) ∪ corr(P″, Q)` — a large comparison assembles from window comparisons with no seams. *No invention, no silent loss:* a sub-extent query reports no pair the full comparison lacks; every full-comparison pair with both feet inside the windows survives verbatim; and a correspondence crossing a window boundary is reported exactly on its in-window part — the out-of-window remainder is excluded, and the in-window portion is not droppable. This is the precise content of "the spans *of the two spec-sets*" in Nelson's sentence, and of his demand that "highlighting the corresponding parts is a vital aspect of intercomparison" [LM 3/13]: the report is bounded by the request and exhaustive within it.

Correspondence is pointwise — no context surrounds a position for a window to disturb — so whole-document comparison is merely the largest window.

## The Pair

`corr` is a finite set of position-pairs; the report must present it finitely and legibly. Raw enumeration would be correct but unreadable. The structure to compress by is the one the foundations use everywhere: lockstep runs (S8 correspondence runs in ASN-0036; mapping blocks in ASN-0058). When `n` consecutive positions of one document carry the same addresses as `n` consecutive positions of the other, one record should say so.

**Definition (Correspondence pair).** A pair is `γ = (d₁, u; d₂, w; n)` with `n ≥ 1`, denoting

`⟦γ⟧ = {((d₁, u + k), (d₂, w + k)) : 0 ≤ k < n}`

`γ` is *consistent at Σ* when every denoted element lies in `Inst_Σ × Inst_Σ` with `res_Σ(d₁, u + k) = res_Σ(d₂, w + k)`; it is *confined to (P, Q)* when `⟦γ⟧ ⊆ P × Q`.

**X10 (PairSemantics).** Let `γ = (d₁, u; d₂, w; n)` be consistent. Then:

(a) *Equal extent, one width.* Both feet sets `{u + k : 0 ≤ k < n}` and `{w + k : 0 ≤ k < n}` have cardinality exactly `n`: distinct offsets land on distinct positions, because for `0 ≤ k₁ < k₂ < n` we have `u + k₁ < u + k₂` — when `k₁ = 0` by TS4 (`u < shift(u, k₂)`), and when `k₁ ≥ 1` by TS5 (ShiftAmountMonotonicity, ASN-0034), whose statement at amounts `k₁ < k₂` is `shift(u, k₁) < shift(u, k₂)` directly. A single width serves both sides structurally; there is no second width to disagree.

(b) *Offset alignment.* The `k`-th position of the first span corresponds to the `k`-th position of the second, for each `k`: within the pair, relative offset is shared. The two starts say *where the shared material sits in each document's current arrangement*; the absolute positions are unrelated to one another and may differ arbitrarily.

(c) *Trace identity.* The pair determines one address sequence `a_k = res(d₁, u + k) = res(d₂, w + k)`. Both sides realize the *same* sequence of stored occurrences in the same order — hence, by X1, the same value sequence. A reported pair asserts exactly this: these two spans are arrangements of the same content occurrences, in the same order. It asserts nothing else — nothing about neighbouring positions, nothing about how the sharing arose, nothing about any other state component. ∎

A remark on `n`. `n` counts lockstep steps, not an address-space difference — tumbler differences are not counts (ASN-0034). The count is well-defined because D-SEQ★ makes content positions dense: within a fixed-depth content subspace the positions are successive final components, so a `k`-fold shift visits exactly `k` arranged positions. The span presentation of each foot is `(u, δ(n, m))`, an ordinal-displacement span at the subspace's own depth (the depths `m` may differ between the two documents; each foot shifts within its own).

A pair must finally identify, per side, *which document* the span inhabits — self-comparison and multi-document spec-sets make this non-redundant. The minimal record is therefore: (document, start) for each side, in operand order, plus the one shared width.

## The Report and Its Canonical Form

**Definition (Report; conformance).** A report is a finite list `Γ = ⟨γ₁, …, γ_r⟩` with denotation `⟦Γ⟧ = (∪ i : 1 ≤ i ≤ r : ⟦γ_i⟧)`. `Γ` *conforms* for `(Σ, P, Q)` when every `γ_i` is consistent and confined, and

`⟦Γ⟧ = corr_Σ(P, Q)`

— soundness (`⊆`, guaranteed per pair by consistency and confinement) and completeness (`⊇`) together. Reports are *equivalent* when their denotations agree. Conformance is thus denotational, and representation granularity is free, exactly as for span-sets (ASN-0053): splitting a pair into two abutting pairs changes nothing reported.

If a relation element `(p, q)` went unreported, a reader inspecting `p` would conclude "this part of the first spec-set is not in the second" — a falsehood about the very question the operation answers. The relation is finite and decidable (X0); there is no excuse of approximation. Soundness without completeness reports sameness only where convenient, and a same/different picture that is wrong anywhere inside the requested windows is wrong simpliciter.

Free granularity still leaves the deterministic presentation to fix. Two structures do it.

**X11 (CanonicalReport).** Define the successor of a relation element by `succ((d₁, u), (d₂, w)) = ((d₁, u + 1), (d₂, w + 1))`. On `corr_Σ(P, Q)`: (a) every element has at most one successor — `succ` is single-valued, so `succ(e)` names one element, whether or not it lies in the relation — and at most one predecessor *within the relation*, by shift injectivity (TS2) per coordinate: two elements sharing a successor agree after a unit shift on each foot, so they agree on each foot and coincide, TS2's equal-depth precondition holding because two content V-positions of one document share a depth (S8-depth); (b) no chain cycles, since feet strictly increase (TS4). Hence the relation partitions uniquely into maximal succ-chains; each chain is the denotation of exactly one consistent, confined pair — its *maximal pair*; and the *canonical report* `CANON(Σ, P, Q)` — the maximal pairs listed in strictly increasing lexicographic order of (first foot, second foot), instances ordered by T1 on document then position — exists and is unique. *Uniqueness of the partition* is the standard maximal-run argument (S8, ASN-0036; M12a, ASN-0058), here applied to the succ-relation on `corr` in place of a single arrangement function: two chains sharing an element coincide by unique forward and backward extension. *Strictness of the order:* distinct maximal pairs sharing both starts would share their first element and hence coincide; sharing only the first start happens exactly under fan-out, and the second key separates. ∎

The canonical order settles the "ordering of pairs" question without smuggling in semantics. The *list* order is presentational determinism — first-operand-major, second foot as tie-break — while the *within-pair* order is semantic: slot `i` belongs to operand `i` (X3). And the symmetry promised earlier can now be finished:

**X3 (continued).** The canonical report of the swapped comparison is the pairwise transpose `(d₂, w; d₁, u; n)` of the original's pairs, re-listed under the transposed sort key. Maximality is orientation-independent — the successor condition is symmetric in the two feet — so transposition is a bijection of canonical pairs, not merely of relation elements. Comparing A with B and comparing B with A yield mirror reports exactly. ∎

At the level of these maximal pairs, the windowing of X4 takes a sharp form:

**X4c (IntervalClipping).** If both windows are single spans — `P′ = {d₁} × (⟦σ_P⟧ ∩ V_{s_C}(d₁))`, `Q′ = {d₂} × (⟦σ_Q⟧ ∩ V_{s_C}(d₂))` — and `γ = (d₁, u; d₂, w; n)` is a maximal pair of the wider comparison, then `⟦γ⟧ ∩ (P′ × Q′)` is the denotation of at most one pair. *Proof.* Index `⟦γ⟧` by `k ∈ [0, n)`, the `k`-th element being `((d₁, u + k), (d₂, w + k))`; since `γ` is a maximal pair of the wider comparison it is confined to that comparison's regions (X11), and those regions are content-confined by their `∩ V_{s_C}` clip, so each foot is already a content instance — the `V_{s_C}` clips are already met and this element lies in `P′ × Q′` iff `u + k ∈ ⟦σ_P⟧` and `w + k ∈ ⟦σ_Q⟧`. Put `K_P = {k ∈ [0, n) : u + k ∈ ⟦σ_P⟧}` and `K_Q` symmetrically. The map `k ↦ u + k` is strictly increasing — TS4 (ASN-0034) at the base `k = 0`, TS5 (ASN-0034) at amounts `k₁ < k₂` — and `⟦σ_P⟧` is order-convex (T12(c), ASN-0034). Hence `K_P` is an integer interval: for `k₁ < k < k₂` with `k₁, k₂ ∈ K_P`, monotonicity gives `u + k₁ < u + k < u + k₂` with the extremes in `⟦σ_P⟧`, so convexity puts `u + k ∈ ⟦σ_P⟧`, i.e. `k ∈ K_P`; likewise `K_Q`. The surviving offsets are `K_P ∩ K_Q`, an intersection of two integer intervals and so again an integer interval (possibly empty) — the index-domain reflection of the span intersection-closure S1 (ASN-0053). If empty the clip denotes no pair; otherwise it is a contiguous `[k_lo, k_hi]`, and `⟦γ⟧ ∩ (P′ × Q′) = ⟦(d₁, u + k_lo; d₂, w + k_lo; k_hi − k_lo + 1)⟧`, exactly one consistent, confined pair. ∎

## Self-Comparison

Nothing in the definitions requires distinct documents or distinct regions, so self-comparison is not an edge case to legislate; it is an evaluation to read off.

**X8 (SelfCorrespondence).** (a) *The diagonal is forced.* `{(p, p) : p ∈ P ∩ Q} ⊆ corr_Σ(P, Q)`, by reflexivity of equality; comparing a region with itself always yields at least the full diagonal, and for an interval window the diagonal is a single maximal pair of full width. (b) *Triviality characterized.* `corr_Σ(P, P)` equals the diagonal **iff** `res|P` is injective: if injective, `res p = res q ⟹ p = q`; if not, any witnesses `p ≠ q` with a shared address contribute the off-diagonal pairs `(p, q)` *and* `(q, p)` (X3). (c) *Windows as detector.* For disjoint windows `P ∩ Q = ∅` drawn from one document, the diagonal is empty and `corr_Σ(P, Q) ≠ ∅ ⟺ ran(res|P) ∩ ran(res|Q) ≠ ∅` — the comparison becomes a pure detector of content shared between the two regions, returning the exact pairs. ∎

Self-correspondence is both a trivial identity and a non-trivial diagonal, with the boundary at injectivity. The diagonal part is trivial — it certifies only that the document is itself. The information lives entirely in the off-diagonal, and it exists precisely when the same stored content is arranged at more than one position, which the foundations not only permit but leave unbounded (S5 UnrestrictedSharing; ASN-0058 M13–M14: repeated occurrences are permanently independent arrangement entries). Nelson's reading concurs: applied to a single document, the substantive thing self-comparison reveals is where the same source material is used in more than one place, and it degenerates to the whole-extent identity exactly when there is no internal sharing. Windowed self-comparison (c) is the operational instrument — two disjoint windows ask "do these regions share material?" and receive the pairs themselves.

## A Worked Example

The machinery deserves to be run once on concrete data. Let `a`, `b`, `c` be three distinct content addresses in `dom(C)` — distinct allocation events, hence distinct tumblers by S4, whatever their stored values. Two documents arrange them at the common depth `m = 2`, where `[1, k]` abbreviates the depth-2 V-position with subspace component `s_C = 1` and final component `k`:

`M(d₁): [1,1] ↦ a, [1,2] ↦ b, [1,3] ↦ c, [1,4] ↦ b`
`M(d₂): [1,1] ↦ b, [1,2] ↦ c`

Both arrangements are D-SEQ★-canonical, and `res` restricted to `d₁` is deliberately *not* injective: `b` is arranged twice.

*The relation.* Compare whole extents: `P = {d₁} × V_{s_C}(d₁)` via `σ_full = ([1,1], δ(4,2))` and `Q = {d₂} × V_{s_C}(d₂)` via `([1,1], δ(2,2))`. Of the `4 × 2 = 8` candidate pairs in `P × Q`, exactly three satisfy `res p = res q`:

`corr_Σ(P, Q) = {((d₁,[1,2]), (d₂,[1,1])), ((d₁,[1,3]), (d₂,[1,2])), ((d₁,[1,4]), (d₂,[1,1]))}`

The repeated `b` produces *fan-out*: the one instance `(d₂,[1,1])` stands in two relation elements, one per arrangement of `b` in `d₁`. No element involves `(d₁,[1,1])`: `a` is arranged nowhere in `d₂` — and by X2 this would remain so even if `C(a)` happened to equal `C(b)` byte for byte.

*Maximal pairs and the canonical report (X11).* Chase successors. `succ((d₁,[1,2]), (d₂,[1,1])) = ((d₁,[1,3]), (d₂,[1,2]))`, which lies in the relation — `b, c` advance in lockstep — while the next step `((d₁,[1,4]), (d₂,[1,3]))` does not, since `[1,3] ∉ dom(M(d₂))`. The remaining element `((d₁,[1,4]), (d₂,[1,1]))` has no predecessor within the relation (its second foot would require a position `w` with `w + 1 = [1,1]`, and `[1,0]` is excluded by S8a) and no successor (`[1,5] ∉ dom(M(d₁))`). The unique chain partition is therefore one width-2 pair and one width-1 pair,

`γ₁ = (d₁, [1,2]; d₂, [1,1]; 2)` and `γ₂ = (d₁, [1,4]; d₂, [1,1]; 1)`,

with `⟦γ₁⟧ ∪ ⟦γ₂⟧ = corr_Σ(P, Q)` exactly — sound, complete, confined. X10(c) on `γ₁`: both sides realize the single address trace `⟨b, c⟩`. The first feet are distinct (`[1,2] < [1,4]` under T1), so the primary key alone orders the list: `CANON(Σ, P, Q) = ⟨γ₁, γ₂⟩`.

*The tie-break, exercised by the swap (X3).* The swapped comparison transposes pairwise: `γ₁ᵀ = (d₂, [1,1]; d₁, [1,2]; 2)` and `γ₂ᵀ = (d₂, [1,1]; d₁, [1,4]; 1)` now *share* their first foot `(d₂, [1,1])` — fan-out lands two chains on one start, exactly the situation X11's strictness clause anticipates — and the second-foot key separates them: `[1,2] < [1,4]` gives `CANON(Σ, Q, P) = ⟨γ₁ᵀ, γ₂ᵀ⟩`. Without the tie-break the canonical list would not be well-defined here.

*Window clipping (X4, X4c).* Narrow the first operand to the span `([1,3], δ(2,2))`, so `P′ = {(d₁,[1,3]), (d₁,[1,4])}`, keeping `Q` whole. X4 predicts `corr_Σ(P′, Q) = corr_Σ(P, Q) ∩ (P′ × Q)` — the first element drops, two remain — and recomputing from the definition confirms it. The width-2 pair `γ₁` meets the window in only its `k = 1` element, so it clips to the single pair `(d₁, [1,3]; d₂, [1,2]; 1)` — at most one pair, as X4c bounds — and `CANON(Σ, P′, Q) = ⟨(d₁, [1,3]; d₂, [1,2]; 1), (d₁, [1,4]; d₂, [1,1]; 1)⟩`: the boundary-crossing correspondence is reported exactly on its in-window part, neither dropped nor extended.

*Self-comparison and the sharing detector (X8).* Whole-extent self-comparison of `d₁` returns the four diagonal elements plus the off-diagonal witnesses of the repeated `b` — `((d₁,[1,2]), (d₁,[1,4]))` and its transpose — six elements in all; the departure from the bare diagonal is exactly the non-injectivity of `res|P` (X8(b)). X8(c)'s detector is the windowed form: the disjoint windows `P′ = {(d₁,[1,1]), (d₁,[1,2])}` and `Q′ = {(d₁,[1,3]), (d₁,[1,4])}` have empty diagonal, `ran(res|P′) ∩ ran(res|Q′) = {a, b} ∩ {c, b} = {b} ≠ ∅`, and the comparison returns precisely the internal sharing:

`corr_Σ(P′, Q′) = {((d₁,[1,2]), (d₁,[1,4]))}` — the single pair `(d₁, [1,2]; d₁, [1,4]; 1)`.

Every count above is forced by the definitions; one six-entry state exercises fan-out, maximality, the tie-break, clipping, and the self-comparison boundary.

## Stability: Edits Transport, Never Re-Pair

We come to the dynamic obligations. The arrangement-edit vocabulary of the foundations is: extension (K.μ⁺, K.μ⁺_L), contraction (K.μ⁻, and ASN-0082's shifting contraction), and reordering (K.μ~); the non-arrangement transitions were dismissed by X5. Every edit transports stored addresses verbatim — none rewrites a binding (P0/S0, L12) — and each induces a *position map* on the surviving positions of the edited document. That single observation yields all the stability theorems at once.

**X-T (TransportLemma).** Let `Σ →* Σ′`, and let `τ : D → Inst_{Σ′}` and `υ : D′ → Inst_{Σ′}` be injective maps on instance sets `D, D′ ⊆ Inst_Σ` satisfying `res_{Σ′}(τ p) = res_Σ(p)` and `res_{Σ′}(υ q) = res_Σ(q)` on their domains. Then for `P ⊆ D`, `Q ⊆ D′`:

`corr_{Σ′}(τ(P), υ(Q)) = (τ × υ)(corr_Σ(P, Q))`

*Proof.* `res′(τ p) = res′(υ q) ⟺ res p = res q`, by the two preservation equations read in both directions; injectivity carries the rectangle across. ∎

The lemma says: wherever an edit provides a res-preserving relocation of the compared positions, the new correspondence is the *image* of the old — conserved element for element, coordinates re-expressed. We instantiate it across the vocabulary. In each single-document instance below, the other operand's restriction is untouched by the edit's frame clause, so `υ = id`; when both operands draw on the edited document, instantiate `τ` and `υ` from the same position map.

**X7 (EditTransport).**

*(i) Reordering.* K.μ~ on `d₁` carries an admissible bijection `π` with `Σ′.M(d₁)(π(v)) = Σ.M(d₁)(v)` on a fixed domain (K.μ~-FIX). With `τ(d₁, v) = (d₁, π(v))`, X-T gives `corr_{Σ′}(τ(P), Q) = (τ × id)(corr_Σ(P, Q))`. Nothing enters and nothing leaves: the witnessed shared addresses, the multiplicities, the entire relation are conserved under the permutation. In wp form, in the style of LP12a — for the one-foot case, second foot `q` on a document other than `d₁`: `wp(K.μ~[d₁, π], ((d₁, x), q) ∈ corr) ≡ enabled ∧ ((d₁, π⁻¹(x)), q) ∈ corr` — membership afterwards is membership before, pulled back through the edit. When both feet lie on `d₁` (self-comparison, X8), both pull back: `wp(K.μ~[d₁, π], ((d₁, x), (d₁, y)) ∈ corr) ≡ enabled ∧ ((d₁, π⁻¹(x)), (d₁, π⁻¹(y))) ∈ corr`.

*(ii) Contraction.* K.μ⁻ restricts `M(d₁)` to a retained set; survivors keep both their positions and their addresses, so `τ = id` on survivor instances. With `Q` drawn off the edited document, X-T degenerates to `corr_{Σ′} = corr_Σ ∩ (Surv × Q)` — *contraction acts on the relation exactly as a window* (X4) — and a pair survives iff its `d₁`-foot survives: `wp(K.μ⁻[d₁, n′], (p, q) ∈ corr) ≡ enabled ∧ p surviving ∧ (p, q) ∈ corr`. When both operands draw on `d₁`, both feet are at risk: the relation becomes `corr_{Σ′} = corr_Σ ∩ (Surv × Surv)`, and the wp gains the symmetric conjunct — `enabled ∧ p surviving ∧ q surviving ∧ (p, q) ∈ corr`.

*(iii) Shifting contraction.* ASN-0082's contraction removes a span and closes the gap: survivors relocate by the piecewise map `τ = id` on the left region `L` and `τ = σ` on the right region `R`, and `M′(σ(v)) = M(v)` is the operation's own postcondition (D-SHIFT, D-L), which discharges res-preservation. Injectivity, however, is not free here as it is in (i) and (ii) — where `τ` is a given bijection or the identity. The piecewise `τ` carries exactly the obligation those cases lack, and we discharge it in three steps: `id` is injective on `L`; `σ` is injective on `R` (D-BJ, ASN-0082); and the two images cannot collide — gap-closure could *a priori* carry a shifted right-region position onto a surviving left-region one, and `L ∩ Q₃ = ∅` (D-DP(a), ASN-0082) is precisely what forbids it. With injectivity and res-preservation both in hand, X-T applies, so realistic deletion is covered: the surviving correspondence is the `σ`-image of the old.

*(iv) Extension.* K.μ⁺ and K.μ⁺_L leave prior mappings unchanged (their agreement clause), so on any regions drawn from the prior domain the relation is literally unchanged; over full extents it can only grow, and every new pair stands on a newly arranged foot. Growth is monotone; established pairs are untouchable.

*Synthesis.* Across the entire edit vocabulary, surviving content is **re-addressed, never re-paired**: which stored occurrence corresponds to which is conserved under the edit's position map, while the V-coordinates in the report are re-resolved against the new arrangement. The canonical decomposition may *refragment* — a permutation that scatters a previously contiguous shared run breaks lockstep adjacency, and the same denotation re-presents as several shorter maximal pairs, one per surviving co-advancing block — but fragmentation is presentation. The denotation transported by X-T is exact. ∎

**X6 (ChainInvisibility).** Now let content travel: `d⁰` shares into `d¹`, `d¹` into `d²`, and so on — each step a composite installing into the next document addresses drawn from the previous one's range (K.μ⁺ steps; the fork composite J4 is the canonical instance, its order-preserving bijection `φ` satisfying `M′(d_new)(φ(v)) = M(d_op)(v)`). Each step is an X-T map *across documents*. Then:

(a) Each sharing step transports correspondence undiminished: at the post-state, `graph(φ) ⊆ corr(d_op extent, d_new extent)` — every copied position corresponds, at full width.

(b) Steps compose under two premises. *Endpoint persistence:* the conclusion compares the endpoints at the evaluation state, so `d⁰`'s restriction on the transported domain must persist from the chain's first step to that state — in a pure chain this is automatic, since each step's frame clause touches only its target document; an edit to `d⁰` itself is not excluded but enters the composite as one more X7 position map applied on the `d⁰` side. *Interleaved intermediate edits:* an edit striking an intermediate `d^i` *between* its incoming and outgoing steps enters the same way — the outgoing step copies from `d^i`'s current arrangement, so the edit's position map `π_i` is interposed and the composite reads `φ_k ∘ … ∘ φ_{i+1} ∘ π_i ∘ φ_i ∘ … ∘ φ₁`. Each factor is injective and res-preserving, hence so is the composite, on its (possibly shrunken) domain — a contraction's map carries only survivors. Under these premises X-T applies to the composite: the endpoints correspond exactly on the transported material, independently of `k`. There is nothing in the state for chain length to act on: the address arriving at `d^k` is the *same tumbler* that left `d⁰` — no hop count, no generation marker, no attenuation exists to consult.

(c) By X5 the endpoint relation depends on the endpoint restrictions alone: once material has arrived at `d^k`, every intermediate document may be rearranged, contracted to nothing, or ignored — `corr(d⁰, d^k)` is unmoved. The middle of the chain can vanish; the correspondence cannot.

(d) Kernel transitivity gives the local composition law: `(p, q) ∈ corr(P, Q)` and `(q, r) ∈ corr(Q, R)` imply `(p, r) ∈ corr(P, R)` — pairwise reports compose soundly through a shared middle region. ∎

Completeness across transitive transclusion chains is therefore not an extra axiom. It is X1 (the basis is the address), P0/S0 (addresses travel verbatim and bindings never change), and X5 (only the endpoints are consulted), assembled.

## The Operation

We can now write the contract. The operation is an *observation*: it computes a value and changes nothing — the first operation in this family whose frame condition is total.

**X12 (COMPARE — SHOWRELATIONOF2VERSIONS).**

- *Operands:* spec-sets `ρ₁, ρ₂`. Each names one document — the two-version case — or several; `ρ₁` and `ρ₂` may name the same document, with equal, overlapping, or disjoint windows.
- *Precondition:* every named `d ∈ E_doc`; every span T12-well-formed; every span a content-subspace span (`subspace(start) = s_C`).
- *Result:* a report `Γ` for `(Σ, R_Σ(ρ₁), R_Σ(ρ₂))`; the *reference result* is the canonical report `CANON(Σ, R_Σ(ρ₁), R_Σ(ρ₂))` of X11.
- *Binding postconditions — required of every conforming implementation:* (R1) *soundness* — every listed pair is consistent at Σ and confined to the regions; (R2) *completeness* — `⟦Γ⟧ ⊇ corr_Σ(R_Σ(ρ₁), R_Σ(ρ₂))`; jointly with R1, `⟦Γ⟧ = corr`; (R3) *deterministic presentation* — the emitted report is a function of `(ρ₁, ρ₂, res_Σ|P, res_Σ|Q)`: X5 guarantees the relation admits this, and the implementation must fix one presentation of it, with no hidden input and no nondeterminism.
- *Reference presentation — defines `CANON`, not required for conformance:* (R4) *canonical form* — the maximal pairs of X11, listed in lexicographic first-operand-major order with the second foot as tie-break, each record carrying (document, start) per side in operand order plus the one shared width. R4 names the unique reference result this specification reasons with; a conforming implementation may emit any presentation satisfying R1–R3 — finer-than-maximal pairs, a different packing of the record — since report equivalence is denotational.
- *Frame:* `Σ′ = Σ`. COMPARE allocates nothing, arranges nothing, links nothing, records nothing. Moreover its value is a function of the operands and the two restricted arrangements alone (X5): it reads neither the values in `C` nor `L`, `E`, `R`, nor any other document's arrangement.

## Implementation Observations (udanax-green)

Gregory's `correspond.c` pipeline converts each spec-set's V-spans to address intervals through the document's arrangement index, intersects the two interval sets, restricts both spec-sets to the common intervals, and zips the restricted V-span lists into pairs. Several of our claims appear in it with satisfying literalness; two deficiencies appear as well, and the abstract claims adjudicate both.

*The record.* Each emitted pair is three tumblers: a composite (document·V-start) for each side and a single shared width — X10's minimal record, with each side's (document, start) packed into one tumbler by concatenating the V-start's digits after the document identifier. The packing is representation, not content.

*Identity basis and locality, realized.* The comparison never reads stored bytes: membership is interval intersection on addresses, so independently typed identical passages — distinct addresses by construction — produce the empty result (X1, X2). It consults only the two documents' arrangement indices: the document-membership index, whose entries are known to survive deletion stalely, is never opened on this path, so stale entries cannot fabricate pairs. X5's memorylessness is enforced by access pattern rather than by filtering.

*Clipping, realized three times.* Boundary overhang is clipped where V-spans are converted to address intervals (the conversion applies the same integer delta to both axes of an arrangement block), again at interval intersection (the narrower overlap is taken), and finally by re-intersecting the mapped-back V-spans with the original query spans. The net effect is X4's restriction identity, and the bounded index traversal — only nodes overlapping the query window are visited — is the computational dividend of X5's locality: a sub-range query performs a sub-range walk, never a whole-document scan with post-hoc filtering.

*Ordering.* Retrieval re-sorts index leaves into the first document's V-order, the intersection loop iterates the first operand's intervals outermost, and the zipper consumes both restricted span lists monotonically: the emitted list follows the first spec-set's V-order, consistent with X11's first-operand-major canonical order; on injective windows the two coincide exactly.

*Fragmentation under rearrangement.* When rearrangement has scattered a shared run, the implementation emits one pair per co-advancing block — concretely, one per arrangement-index leaf covering the shared interval. This is X7's refragmentation in the flesh: denotation conserved, presentation split; the particular count is implementation granularity.

*Deficiency 1 — completeness under repetition.* When the same address range is arranged at several positions within a compared region, the pairing loop consumes a per-interval width budget equal to the shared interval's width: one pair is emitted and the remaining occurrences are orphaned, even though the retrieval layer faithfully enumerated them all. Self-comparison consequently reports only the diagonal — the zipper advances both feet in lockstep and is structurally incapable of emitting an off-diagonal pair. Against this specification that is a completeness violation (R2) on every non-injective restriction; the implementation conforms exactly on injective windows. The missing pairs remain *recoverable* by windowing — disjoint-window self-comparison returns the sharing pairs one region-pair at a time, as the surviving golden test demonstrates — but recoverability through extra queries does not discharge R2. As with the fixed mantissa against T0, the implementation must either meet the abstract claim or document its matching semantics as a deviation.

*Deficiency 2 — the subspace precondition, violated rather than enforced.* Fed spec-sets that include link-subspace positions, the original pipeline aborts: the back-conversion from addresses to positions signals success unconditionally while leaving its output empty for the link region, and the empty span-set propagates into intersection as a null operand. X9 says the excluded territory could never have contributed a pair, so the crash is a violated precondition, not a missing feature — and the content-subspace filter later placed in front of the pipeline implements X12's precondition exactly.

*Minor.* Width arithmetic internally truncates multi-digit tumbler widths to machine integers; for windows whose widths exceed that range the clipping arithmetic would silently err — the same genus of bound violation as the fixed mantissa, recorded for completeness.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| Inst | `Inst_Σ = {(d, v) : d ∈ E_doc ∧ v ∈ dom(Σ.M(d))}` — document-tagged arranged positions | introduced |
| res | `res_Σ(d, v) = Σ.M(d)(v)` — total resolution map on instances (S2, S3★) | introduced |
| R_Σ(ρ) | spec-set region: `(∪ i :: {d_i} × (⟦S_i⟧ ∩ V_{s_C}(d_i)))` — spans clipped to the current arrangement | introduced |
| corr | `corr_Σ(P, Q) = {(p, q) ∈ P × Q : res p = res q}` — kernel of res on the operand rectangle | introduced |
| γ | correspondence pair `(d₁, u; d₂, w; n)` with lockstep denotation, consistency, confinement | introduced |
| ⟦Γ⟧ | report denotation; conformance ≡ sound ∧ complete ∧ confined; equivalence is denotational | introduced |
| X0 | corr is finite and decidable from the operands plus the two arrangement restrictions | introduced |
| X1 | membership ⟺ shared address; shared value entailed (one store entry), never consulted | introduced |
| X2 | equal values at distinct addresses are reachable and never correspond | introduced |
| X3 | `corr(Q, P) = corr(P, Q)⁻¹`; canonical reports transpose pairwise | introduced |
| X4 | `corr(P′, Q′) = corr(P, Q) ∩ (P′ × Q′)` — windowing is exact restriction | introduced |
| X4c | with single-span windows, a maximal pair clips to at most one pair | introduced |
| X5 | corr is a function of the two restrictions: deterministic, edit-indifferent elsewhere, memoryless | introduced |
| X-T | injective res-preserving relocations transport corr: `corr′ = (τ × υ)(corr)` | introduced |
| X6 | sharing chains transport correspondence undiminished; endpoints alone determine it; pairwise reports compose through a shared middle | introduced |
| X7 | reorder/contract/shift/extend act by their position maps: survivors re-addressed, never re-paired; presentation may refragment | introduced |
| X8 | diagonal forced; self-comparison trivial ⟺ restriction injective; disjoint windows detect internal sharing | introduced |
| X9 | link instances contribute only the forced self-diagonal on `P ∩ Q`, decided by instance equality without `res`; the restriction loses no correspondence information | introduced |
| X10 | one width serves both sides; k-th↔k-th offset alignment; identical address (hence value) trace | introduced |
| X11 | unique maximal-pair decomposition; lexicographic order yields a deterministic canonical report | introduced |
| X12 | COMPARE: pure observation — binding: sound, complete, confined, deterministic presentation (R1–R3); CANON is the non-binding reference presentation (R4); Σ unchanged | introduced |

## Open Questions

- Can a matching-based report carrying multiplicity annotations be information-equivalent to the full position-level correspondence relation, or is position-level completeness irreducible in the report itself?
- Which pair granularity must an interoperable report fix — maximal co-advancing runs, or runs aligned to contiguous shared allocation intervals — given that both denote the same relation?
- What guarantees must pairwise correspondence provide for n-way alignment across many documents to be soundly composed from pairwise reports?
- What consistency contract must a derived correspondence index satisfy for cached reports to remain exact across arrangement edits to either compared document?
- Should correspondence extend to content referenced by stored spans but arranged in neither compared document, or is arrangement-presence the correct basis for what counts as part of a version?
- If the subspace vocabulary ever grows beyond content and links, what property makes a new subspace correspondence-bearing rather than structurally vacuous?
