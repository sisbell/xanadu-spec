> **ASN-0086 · Typed Relations on Address Sets** — condensed claim statements  
> [← Full note](ASN-0086-typed-relations-on-address-sets.md) · [↑ Protocols index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0086 Claim Statements

*Source: ASN-0086-typed-relations-on-address-sets.md (revised unknown) — Extracted: 2026-06-02*

## Definition — AddressUniverse

`A^Σ = dom(Σ.C) ∪ dom(Σ.L)`

By L14 (DualPrimitive, ASN-0043), `A^Σ` is the entirety of stored-entity addresses at Σ — no state component maps an address outside `dom(Σ.C) ∪ dom(Σ.L)` to an entity value.

## Definition — AddressPartition

`A_doc^Σ = dom(Σ.C)` — content addresses
`A_rel^Σ = dom(Σ.L)` — link-store addresses

`A^Σ = A_doc^Σ ⊔ A_rel^Σ` (disjoint union); disjointness is `dom(Σ.C) ∩ dom(Σ.L) = ∅` (SD, StoreDisjointness, ASN-0093).

All three sets grow monotonically as the substrate evolves (by S1 and L12a).

## Definition — AdmissibleTypes

`T_admissible = {K ∈ Endset : K ≠ ∅}`

Non-empty endsets, eligible to serve as a link's type endset by L3 (NEndsetStructure, ASN-0043). By L4 (EndsetGenerality, ASN-0043) and L9 (TypeGhostPermission, ASN-0043), `T_admissible` is unconstrained by content existence: type endsets may reference any tumbler addresses, including ghost addresses.

## Definition — TypeEquivalence

`K ~ K' ≡ coverage(K) = coverage(K')`

This is L8's (TypeByAddress, ASN-0043) notion of `same_type`, lifted from links to type endsets themselves. The quotient `T_admissible / ~` is the set of *coverage classes*; the equivalence class of `K` is written `[K]`.

## Definition — StateTransition

`→ ≡ K.σ ∪ K.α ∪ K.λ`

The substrate's *dom-extending* one-step transition relation. A K.σ-step extends `dom(Σ.M)`, a K.α-step extends `dom(Σ.C)`, and a K.λ-step extends `dom(Σ.L)`, each at a fresh key per its ASN-0093 contract; frame conditions leave the other two components unchanged.

**Definition — Reachability.** `Σ' is →-reachable from Σ`, written `Σ →* Σ'`, is the reflexive-transitive closure of `→`.

`Σ →* Σ'` entails `dom(Σ.C) ⊆ dom(Σ'.C)`, `dom(Σ.M) ⊆ dom(Σ'.M)`, `dom(Σ.L) ⊆ dom(Σ'.L)`, with `Σ'.C|_{dom(Σ.C)} = Σ.C`, `Σ'.M|_{dom(Σ.M)} = Σ.M`, `Σ'.L|_{dom(Σ.L)} = Σ.L`.

## Definition — EmitAddress

For any `d ∈ dom(Σ.M)`, the *fresh emission address* `a_emit(Σ, d)` is:

`a_emit(Σ, d) = [d.0.s_L.1]` when `{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d} = ∅` (*first-emission* branch);
`a_emit(Σ, d) = inc(ℓ_prev, 0)` otherwise, where `ℓ_prev := max{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d}` (*subsequent-emission* branch).

`a_emit` is a *total* function of `(Σ, d ∈ dom(Σ.M))`.

## Definition — TypedRelation

For each `K ∈ T_admissible` and state Σ, the *typed relation of type K at Σ* is

`L_K^Σ = {(a, F, G) : a ∈ dom(Σ.L) ∧ |Σ.L(a)| = 3 ∧ Σ.L(a).e₁ = F ∧ Σ.L(a).e₂ = G ∧ coverage(Σ.L(a).e₃) = coverage(K)}`

Each member is a triple of (tuple-address, from-endset, to-endset). The `|Σ.L(a)| = 3` conjunct restricts every `L_K` to standard-triple links.

## Definition — StandardTripleLinkStore

`L^Σ = ⨆_{[K] ∈ T_admissible / ~} L_K^Σ`

The substrate's standard-triple link store at state Σ is the disjoint union over coverage classes. The union is disjoint by Lemma — SliceUniqueness.

*Notation — subscript read modulo `~`.* Since the slot-3 criterion tests `coverage(Σ.L(a).e₃) = coverage(K)`, any `K ~ K'` induces `L_K^Σ = L_{K'}^Σ`; the subscript is a *coverage-class* index.

## Definition — TupleAddressMap

`addr : L^Σ → A_rel^Σ` defined by `addr(a, F, G) = a`, with codomain `A_rel^Σ = dom(Σ.L)` and image the arity-3 slice `{a ∈ dom(Σ.L) : |Σ.L(a)| = 3}`.

## Definition — RetractionType

Fix a designated coverage class `[R]` reserved for retraction, represented by any `R ∈ T_admissible` whose coverage selects the conventional retraction address set. The corresponding typed relation `L_R^Σ` is the *retraction relation at state Σ*.

**Convention — RetractionDirectionality.** For the retraction coverage class `[R]`, the to-set carries the retraction's targets — addresses whose tuples are being withdrawn from the active subset — and the from-set is reserved for attribution-bearing endset content or is left empty for unattributed retractions.

## Definition — Nullified

`nullified(Σ) = {a ∈ A_rel^Σ : (E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G'))}`

The existential checks `coverage(G')` only (per Convention RetractionDirectionality), ranges over `L_R^Σ` (triple-restricted by construction), and the set-builder restriction `a ∈ A_rel^Σ` confines `nullified(Σ)` to link-store addresses.

## Definition — ActiveSubset

For each `K ∈ T_admissible`, the *active subset of type K at state Σ* is

`A_K^Σ = {(a, F, G) ∈ L_K^Σ : a ∉ nullified(Σ)}`

`A_K^Σ` is a finite, computable set: `L_K^Σ` is selected by CoverageEqualityDecidable, `nullified(Σ)` by CoverageEqualityDecidable and T2 span-membership.

## Definition — UnitDepthRetractionDiscipline

A state Σ is *unit-depth-disciplined* iff every `(b, F', G') ∈ L_R^Σ` has to-endset `G' = {(t, δ(1, #t))}` for some target `t ∈ A_rel^Σ`. Membership `t ∈ A_rel^Σ` is evaluated at the state Σ in question.

A *layer* satisfies the *unit-depth retraction discipline* iff every state it reaches is unit-depth-disciplined.

## Definition — RelationalLayer

The relational layer's link-store operations are `{Emit_K, Observe_K, Nullify}`. Its one *discipline commitment* is a single predicate over `→`-steps: every `→`-step `Σ → Σ'` that grows the retraction slice — i.e. with `L_R^Σ ⊊ L_R^{Σ'}` — is a `Nullify`.

## Definition — LayerReachable

A state is *layer-reachable* iff it is `→*`-reachable from `Σ_init` (Definition — Reachability) by a finite sequence of `→`-steps each obeying the discipline commitment (Definition — relational layer).

---

## Lemma — CoverageEqualityDecidable

For any two endsets `e, e' ∈ Endset`, the predicate `coverage(e) = coverage(e')` is decidable using only T2 comparisons and TumblerAdd.

**Procedure:** By `Endset = 𝒫_fin(Span)` (ASN-0043), `e ∪ e'` is finite, so `coverage(e)` and `coverage(e')` are finite unions of half-open T1-intervals `[s, s ⊕ ℓ)`. Construct endpoint set `P = {s : (s, ℓ) ∈ e ∪ e'} ∪ {s ⊕ ℓ : (s, ℓ) ∈ e ∪ e'}`, sorted under T1 into distinct values `c₁ < … < c_m`. Each coverage is constant on every cell between endpoints. A gap cell `(c_k, c_{k+1})` is nonempty iff `c_k.0 ≠ c_{k+1}`. Two finite unions of intervals are equal iff they assign the same membership to every tumbler; since each coverage is constant on every cell, compare membership at one representative per nonempty cell.

## Lemma — SliceUniqueness

Each tuple address `a ∈ dom(Σ.L)` indexes *at most one* slice `L_K^Σ`.

`Σ.L` is a partial function `T ⇀ Link` (ASN-0043), so `a` carries a single value `Σ.L(a)`, hence a single slot-3 endset `Σ.L(a).e₃` and a single coverage class `[Σ.L(a).e₃]`; thus `a` lies in no two slices.

## Fact — HomeOriginCoincidence

On link addresses, `home` and `origin` coincide: both are the NUDE-prefix projection `N(·).0.U(·).0.D(·)`.

`{a ∈ dom(Σ.L) : home(a) = d} = dom(Σ.L) ∩ {ℓ' : origin(ℓ') = d}` for any document `d`.

---

## L-ContiguousPrefix — ContiguousPrefix (LEMMA, lemma)

At every `→*`-reachable state Σ, for every `d ∈ dom(Σ.M)` there exists `J_d^Σ ∈ ℤ_{≥-1}` such that the homed-set is a contiguous initial segment of `A_L(d)`'s chain enumeration, and (when non-empty) admits a unique T1-maximum at chain index `J_d^Σ`:

`(A Σ : Σ →*-reachable :: (A d ∈ dom(Σ.M) :: (E J_d^Σ ∈ ℤ_{≥-1} :: {a ∈ dom(Σ.L) : home(a) = d} = {incʲ(d.0.s_L.1, 0) : 0 ≤ j ≤ J_d^Σ})))`

(with `J_d^Σ = -1` denoting the empty set when no link is homed at `d`).

*Unique T1-maximum on non-empty homed-sets.* When `J_d^Σ ≥ 0`, `max{a ∈ dom(Σ.L) : home(a) = d}` under T1 is well-defined and equals `inc^{J_d^Σ}(d.0.s_L.1, 0)`.

## R0 — TupleAddressFreshness (LEMMA, lemma)

For any →*-reachable state Σ, any caller-supplied home `d ∈ dom(Σ.M)`, and any `(F, G, K) ∈ Endset × Endset × T_admissible`, there exists a state Σ' with Σ → Σ' that emits a tuple with content (F, G) of type K at an address `a` that is *fresh* against `dom(Σ.L)` and *on-chain* in `A_L(d)`, homed at `d`, with Σ' itself →*-reachable:

`(A Σ : Σ →*-reachable :: (A d ∈ dom(Σ.M), F, G ∈ Endset, K ∈ T_admissible :: (E Σ', a : Σ → Σ' ∧ a ∉ dom(Σ.L) ∧ a ∈ A_L(d) :: a ∈ dom(Σ'.L) ∧ Σ'.L(a) = (F, G, K) ∧ home(a) = d ∧ Σ' →*-reachable)))`

*Value-shape consequence.* The standard triple `(F, G, K)` discharges K.λ's L3 precondition directly — arity is 3, both content slots `F, G ∈ Endset`, and `K ∈ T_admissible` forces a non-empty type slot.

## R0a — FlatLinkDomain (LEMMA, lemma)

At every →*-reachable state Σ, `dom(Σ.L)` is a tumbler-prefix antichain:

`(A Σ : Σ →*-reachable :: (A a, a' ∈ dom(Σ.L) :: a ≼ a' ⟹ a = a'))`

## R1 — AddressInjectivity (LEMMA, lemma)

The map `addr : L → A_rel` is an injection:

`(A (a, F, G), (a', F', G') ∈ L : a = a' :: F = F' ∧ G = G')`

## R2 — TupleAddressPermanence (ALIAS, lemma)

TupleAddressPermanence is L12 (LinkImmutability, ASN-0043) in tuple vocabulary: an allocated tuple address resolves permanently to the same relational content.

*Consequence.* Two agents independently filing tuples with identical `(F, G)` under identical `K` produce distinct addresses (R0 produces a fresh address regardless of value). By L11b (NonInjectivity, ASN-0043), value-level coincidence is permitted; by R1, address-level identity nevertheless distinguishes them.

## R3 — TypedSliceMonotonicity (LEMMA, lemma)

Each typed relation grows monotonically:

`(A Σ → Σ', K ∈ T_admissible :: L_K^Σ ⊆ L_K^{Σ'})`

where `L_K^Σ` denotes the typed relation evaluated at state `Σ`.

## R4 — TupleAddressDisjointness (ALIAS, predicate)

TupleAddressDisjointness is SD (StoreDisjointness, ASN-0093: `dom(Σ.C) ∩ dom(Σ.L) = ∅`) under the partition aliases `A_doc^Σ = dom(Σ.C)`, `A_rel^Σ = dom(Σ.L)`:

`A_doc^Σ ∩ A_rel^Σ = ∅`

## R5 — TupleSelfTargeting (LEMMA, lemma)

A tuple's from-set or to-set may reference tuple addresses. For any →*-reachable state Σ and any `a ∈ A_rel^Σ`, the unit-depth span `(a, δ(1, #a))` is well-formed and may appear in the from-set or to-set of an emitted tuple, with `a` in its coverage.

Sub-claims:

(a) *Span well-formedness.* `zeros(a) = 3`; `#E(a) ≥ 2` so `#a ≥ 1`. The span `(a, δ(1, #a))` satisfies T12 (SpanWellDefinedness, ASN-0034) — action point `#a ≤ #a`. By PrefixSpanCoverage (ASN-0043), `coverage({(a, δ(1, #a))}) = {t : a ≼ t}`, which contains `a` by reflexivity of `≼`.

(b) *Endset admissibility.* By L4(c) (EndsetGenerality, ASN-0043), endset spans may reference link-subspace addresses. The singleton endset `G_self = {(a, δ(1, #a))}` is an admissible `Endset` member at any slot of an emitted link.

(c) *Self-targeting emission (to-set case).* For any `d ∈ dom(Σ.M)` and `K ∈ T_admissible`, R0 at triple `(∅, G_self, K)` and home `d` produces a fresh emitter `a' ∉ dom(Σ.L)` with `Σ'.L(a') = (∅, G_self, K)` and `a ∈ coverage(Σ'.L(a').e₂)`.

(d) *From-set case.* Symmetric: the triple `(G_self, ∅, K)` is L3-conforming. R0 yields fresh `a''` with `Σ''.L(a'') = (G_self, ∅, K)` and `a ∈ coverage(Σ''.L(a'').e₁)`.

## R-Scope — SingleTupleScope (LEMMA, lemma)

At every →*-reachable state Σ, for any caller-supplied `d_retr ∈ dom(Σ.M)` and any target `a` admissible under Nullify's precondition P-tgt — that is, `a ∈ A_rel^Σ` (P1) *or* `a = a_emit(Σ, d_retr)` (self-emit) — the `→`-step taken by `Nullify(Σ, d_retr, a) = Emit_R(Σ, d_retr, ∅, {(a, δ(1, #a))})` contributes exactly `a` to the nullified set:

`{t : a ≼ t} ∩ A_rel^{Σ'} = {a}`

where `(Σ', _) = Nullify(Σ, d_retr, a)`. The result is *arity-independent*: it holds regardless of `|Σ.L(a)|`.

## R6a — RetractionStability (LEMMA, lemma)

Once a tuple's address is nullified, it stays nullified across all future state transitions:

`(A Σ → Σ', a : a ∈ nullified(Σ) :: a ∈ nullified(Σ'))`

## R6b — SingleDepthRetraction (DEF-Consequence, predicate)

A retractor's tuple nullifies its targets through a single-pass check over the audit slice, with no regard to the retractor's own status:

`(A Σ, a, b, F', G' : a ∈ A_rel^Σ ∧ (b, F', G') ∈ L_R^Σ ∧ a ∈ coverage(G') : a ∈ nullified(Σ))`

All clauses are evaluated at the single state Σ. The membership test consults the audit slice `L_R^Σ`, not the active subset `A_R^Σ` — retraction-of-retraction is a non-fixpoint operation: nullifying a retractor `b` does not "undo" `b`'s nullifying effect on its prior targets.

## R6c — RestorationByReemission (LEMMA, lemma)

Once retracted, a tuple stays out of every active subset at any state reachable from Σ:

`(A Σ, K, (a, F, G) ∈ L_K^Σ : a ∈ nullified(Σ) : (A Σ' : Σ →* Σ' :: (a, F, G) ∉ A_K^{Σ'}))`

*Consequence — `A_K` is not monotone, though `L_K` is.* R3 (lifted along `→*`) makes the audit slice monotone (`Σ →* Σ' ⟹ L_K^Σ ⊆ L_K^{Σ'}`); the active subset is not. To "restore" content, emit a fresh tuple with the desired value (R0); the new tuple receives a fresh address.

---

## Unit-depth retraction discipline — UnitDepthRetractionDiscipline (COMMITMENT, predicate)

Layer-level convention: every `L_R^Σ` tuple has to-endset of the form `{(b, δ(1, #b))}` for some target `b ∈ A_rel^Σ`.

Discharged for every layer-reachable state by induction:

- *Base:* `Σ_init.L = ∅`, so `L_R^{Σ_init} = ∅` and `Σ_init` is unit-depth-disciplined vacuously.
- *Step:* K.σ and K.α do not touch `Σ.L`; `Emit_K` at `K ≁ R` leaves the retraction slice unchanged; a higher-arity K.λ at `K ~ R` enters `dom(Σ'.L)` but not `L_R` (triple-restricted). The only `L_R`-growing step kind is a raw arity-3 K.λ at `K ~ R`, which by the discipline commitment is a `Nullify`, adding `(b, ∅, {(a, δ(1, #a))})` with `a ∈ A_rel^{Σ'}` by P-tgt.

## Relational layer — RelationalLayer (COMMITMENT, predicate)

Operation set `{Emit_K, Observe_K, Nullify}`; layer invokes `Emit_K` at `K ~ R` only via the `Nullify` alias, satisfying the unit-depth retraction discipline.

One *discipline commitment*: every `→`-step `Σ → Σ'` with `L_R^Σ ⊊ L_R^{Σ'}` is a `Nullify`. This quantifies over *all* `→`-steps that `→ ≡ K.σ ∪ K.α ∪ K.λ` admits.

---

## Emit_K — EmitK (OP, method)

`Emit_K : Σ × dom(Σ.M) × Endset × Endset → Σ' × A_rel^{Σ'}`

where Σ ranges over the `→*`-reachable states.

*Precondition.* `K ∈ T_admissible`, `d ∈ dom(Σ.M)`.

*Effect.* Given input state Σ, caller-supplied home document `d ∈ dom(Σ.M)`, and finite endsets `F, G ∈ Endset`, `Emit_K(Σ, d, F, G)` invokes K.λ at home `d` with value `(F, G, K)`. The fresh address is `a = a_emit(Σ, d)`. The returned `(Σ', a)` satisfies:

- `a ∉ dom(Σ.L)`
- `a ∈ dom(Σ'.L)`
- `home(a) = d`
- `Σ'.L(a) = (F, G, K)` (permanent by R2)

*Frame.* `Σ'.C = Σ.C` and `Σ'.M = Σ.M`.

## Observe_K — ObserveK (OP, function)

For `K ∈ T_admissible`, a pattern `(F̂, Ĝ) ∈ ℘_fin(T) × ℘_fin(T)`, and a view selector:

`Observe_K : Σ × ℘_fin(T) × ℘_fin(T) × View → ℘_fin(L_K^Σ)`

where `View ∈ {hist, oper}`.

Returns `{(a, F, G) ∈ view : F̂ ⊆ coverage(F) ∧ Ĝ ⊆ coverage(G)}`

with `view = L_K^Σ` if `View = hist` and `view = A_K^Σ` if `View = oper`. Observe leaves Σ unchanged.

*Pattern domain — `T`, not `A^Σ`.* Patterns range over `T` so a pattern may target ghost addresses. The match relation `F̂ ⊆ coverage(F)` is decidable because `F̂` is finite and each per-span membership test `t ∈ coverage(F)` is decidable by T2 (IntrinsicComparison, ASN-0034).

## Nullify — Nullify (OP, method)

*Preconditions:*
- **P0**: `d_retr ∈ dom(Σ.M)`
- **P-tgt**: `a ∈ A_rel^Σ` (P1) *or* `a = a_emit(Σ, d_retr)` (self-emit)

`Nullify(Σ, d_retr, a) ≡ Emit_R(Σ, d_retr, ∅, {(a, δ(1, #a))})`

*Effect.* Emits a tuple into the retraction relation with empty from-set and a unit-depth to-span targeting `a`, homed at `d_retr`. R0 at `d_retr` emits the retraction triple `(∅, {(a, δ(1, #a))}, R)`, depositing a fresh emitter address `b` with `Σ'.L(b) = (∅, {(a, δ(1, #a))}, R)`. Under P-tgt, `a ∈ nullified(Σ')`, persisting thereafter by R6a.

The to-span `(a, δ(1, #a))` is T12-well-formed for *any* tumbler `a` (`#a ≥ 1` by T0, `actionPoint(δ(1, #a)) = #a ≤ #a`).

---

## wp Case 1 — NullifyWeakestPrecondition (DERIVED, predicate)

`wp(Nullify(Σ, d_retr, a), "{t : a ≼ t} ∩ A_rel^{Σ'} = {a}") ≡ P0(Σ, d_retr) ∧ (P1(Σ, a) ∨ a = a_emit(Σ, d_retr))`

where P0: `d_retr ∈ dom(Σ.M)` and P1: `a ∈ A_rel^Σ`, over the ambient `→*`-reachable domain. The weakest precondition *coincides* with the operation's own precondition `P0 ∧ P-tgt`: every legal Nullify call attains single-tuple scope.

The wp does not constrain `|Σ.L(a)|`, by R-Scope's arity-independence.

## wp Case 2 — EmitKWeakestPrecondition (DERIVED, predicate)

Over the `→*`-reachable working domain:

`wp(Emit_K(Σ, d, F, G), (a, F, G) ∈ A_K^{Σ'}) ≡ d ∈ dom(Σ.M) ∧ (K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G)) ∧ ¬(E (b, F', G') ∈ L_R^Σ :: a_emit(Σ, d) ∈ coverage(G'))`

(over `→*`-reachable Σ; `K` is an index, not a free wp variable)

The third conjunct asserts that no pre-existing retraction tuple in `L_R^Σ` already covers `a_emit(Σ, d)`.

*Disciplined-domain simplification.* At a layer-reachable state the third conjunct holds vacuously (unit-depth retraction discipline + R0a give `a_emit(Σ, d) ∉ coverage(G')` for any pre-existing `(_, _, G') ∈ L_R^Σ`), so the wp reduces to:

`d ∈ dom(Σ.M) ∧ (K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G))`

The `a_emit(Σ, d) ∉ coverage(G)` escape branch is non-redundant: a `K ~ R` call with `G = ∅` leaves `a ∉ nullified(Σ')`, so the postcondition holds where a bare `K ≁ R` would wrongly reject it.
