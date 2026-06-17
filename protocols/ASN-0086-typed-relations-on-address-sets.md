> **ASN-0086 · Typed Relations on Address Sets** — Protocols layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md)  
> [Condensed statements →](ASN-0086-typed-relations-on-address-sets.statements.md) · [↑ Protocols index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0086: Typed Relations on Address Sets

*Drawing the link model forward into a relational vocabulary*

ASN-0043 establishes the link as a primitive: an addressed, owned, typed connection between spans of content. ASN-0093 wraps that primitive (along with content and document allocation) in three K-operations — K.σ, K.α, K.λ — that fix the sibling-frontier emission discipline and the sub-allocator chain structure. This note layers on top of ASN-0093's K-operations, adopting a different vocabulary for the link store: where ASN-0043 speaks of *links* and *endsets*, we speak of *tuples* and *typed relations*. The two vocabularies describe one object — a standard-triple link `(F, G, Θ)` at address `a ∈ dom(Σ.L)` is a tuple in a typed relation indexed by `Θ`.

We are looking for what a relation algebra over the link store affords. The answer is that predicates compose more cleanly over typed relations than over endsets, and several substrate-level guarantees — most centrally the *active/audit distinction* between the audit trail and the operational currently-in-effect set — become easier to state and prove in this form. The structure here is not the set-theoretic typed relation (a subset of `℘(A) × ℘(A)`, distinguished only by content): each tuple carries an address that participates in the relation's identity, which the content-only projection `(a, F, G) ↦ (coverage(F), coverage(G))` discards. That address is what we exploit throughout.


## The Two Foundational Sets

**Foundation.** We work in systems satisfying ASN-0093 (and therefore ASN-0043, ASN-0036, ASN-0034). ASN-0093's SubspaceConventionAxiom fixes `s_C = 1 ∧ s_L = 2`, with named consequence `SC-NEQ: s_C ≠ s_L`.

**Assumption — EmptyInitialLinkStore.** This note takes the system's initial state `Σ_init` to be the *fresh-system root*, with all three stores empty: `dom(Σ_init.C) = ∅`, `dom(Σ_init.M) = ∅`, and in particular `dom(Σ_init.L) = ∅` (the fresh-system boot condition).

**State transition relation.** We write `Σ → Σ'` for the substrate's *dom-extending* one-step transition relation, which we identify exactly with the union of ASN-0093's three K-operations: `→ ≡ K.σ ∪ K.α ∪ K.λ`. A K.σ-step extends `dom(Σ.M)` (registering `M'(d) = ∅` at a document-level `d`), a K.α-step extends `dom(Σ.C)`, and a K.λ-step extends `dom(Σ.L)`, each at a fresh key per its ASN-0093 contract; ASN-0093's frame conditions leave the other two components unchanged.

**Definition — Reachability.** `Σ' is →-reachable from Σ`, written `Σ →* Σ'`, is the reflexive-transitive closure of `→`.

By the K.σ/K.α/K.λ frame conditions stated above, `Σ →* Σ'` entails `dom(Σ.C) ⊆ dom(Σ'.C)`, `dom(Σ.M) ⊆ dom(Σ'.M)`, `dom(Σ.L) ⊆ dom(Σ'.L)`, with `Σ'.C|_{dom(Σ.C)} = Σ.C`, `Σ'.M|_{dom(Σ.M)} = Σ.M`, `Σ'.L|_{dom(Σ.L)} = Σ.L`.

**Working domain — `→*`-reachable states.** All results below are stated over states `→*`-reachable from `Σ_init`. **RT-closure**: the class is closed under `→`. Each `→`-step is a single K.σ/K.α/K.λ primitive, which preserves the invariant catalog *published by ASN-0093's K-operation contracts* (the S/M/C invariants of ASN-0036 and ASN-0093, together with the L-invariants those contracts carry). For K.λ, the step lands its single fresh link key at `a_emit(Σ, d)` (Definition — `a_emit`).

**Definition — AddressUniverse.** The substrate's address universe at state Σ is

`A^Σ = dom(Σ.C) ∪ dom(Σ.L)`

By L14 (DualPrimitive, ASN-0043), `A^Σ` is the entirety of stored-entity addresses at Σ — no state component maps an address outside `dom(Σ.C) ∪ dom(Σ.L)` to an entity value, so no third category exists. (SD, StoreDisjointness, ASN-0093, supplies the disjointness `dom(Σ.C) ∩ dom(Σ.L) = ∅`.)

**Definition — Partition.** Define:

`A_doc^Σ = dom(Σ.C)` &nbsp; — content addresses
`A_rel^Σ = dom(Σ.L)` &nbsp; — link-store addresses

We claim `A^Σ = A_doc^Σ ⊔ A_rel^Σ` (disjoint union); the disjointness is `dom(Σ.C) ∩ dom(Σ.L) = ∅`, i.e. SD (StoreDisjointness, ASN-0093).

*Notation.* All three sets are state-dependent — `A^Σ`, `A_doc^Σ`, and `A_rel^Σ` grow monotonically as the substrate evolves (by S1 and L12a). Where the ambient state is unambiguous, we drop the superscript and write `A`, `A_doc`, `A_rel`.

**Definition — AdmissibleTypes.** The set of *admissible types* is

`T_admissible = {K ∈ Endset : K ≠ ∅}`

— non-empty endsets, eligible to serve as a link's type endset by L3 (NEndsetStructure, ASN-0043).

By L4 (EndsetGenerality, ASN-0043) and L9 (TypeGhostPermission, ASN-0043), `T_admissible` is unconstrained by content existence: type endsets may reference any tumbler addresses, including ghost addresses that carry no stored entity at Σ. Type indices in what follows range over `T_admissible`.

## Allocator Structure

ASN-0093 supplies the sub-allocator structure this note relies on: for each `d ∈ dom(Σ.M)`, ChainDiscipline and FirstEmission (ASN-0093) establish two sub-allocator chains `A_C(d)` (content) and `A_L(d)` (link), anchored respectively at `b_C(d) := [d.0.s_C]` and `b_L(d) := [d.0.s_L]`, with first emissions `[d.0.s_C.1]` and `[d.0.s_L.1]`. We use ASN-0093's names directly throughout.

*Derived chain facts.* By ChainDiscipline (ASN-0093), each `A_C(d)`, `A_L(d)` is an instance of ASN-0040's sibling stream `S(p, d)`. We use one `S(p, d)` postcondition throughout, holding along the whole `inc(·, 0)` chain: **(UL) uniform length** — `#cₙ = #c₁` for every chain element (from `S(p, d)`'s `#cₙ = #p + d`).

**Definition — `a_emit(Σ, d)`.** For any `d ∈ dom(Σ.M)`, the *fresh emission address* `a_emit(Σ, d)` is the value of the first/subsequent-emission formula:

`a_emit(Σ, d) = [d.0.s_L.1]` when `{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d} = ∅` (*first-emission* branch);
`a_emit(Σ, d) = inc(ℓ_prev, 0)` otherwise, where `ℓ_prev := max{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d}` (*subsequent-emission* branch).

The max is the unique T1-extremum of a finite (L-fin, ASN-0043) non-empty set, by T1 (LexicographicOrder, ASN-0034) trichotomy. Both branches evaluate from `(Σ, d)` alone, so `a_emit` is a *total* function of `(Σ, d ∈ dom(Σ.M))`.


## The Typed Relation

**Definition — TypeEquivalence.** Two admissible types are *type-equivalent* iff they cover the same address set:

`K ~ K' ≡ coverage(K) = coverage(K')`

This is L8's (TypeByAddress, ASN-0043) notion of `same_type`, lifted from links to type endsets themselves. The quotient `T_admissible / ~` is the set of *coverage classes*; the equivalence class of `K` is written `[K]`.

**Lemma — CoverageEqualityDecidable.** For any two endsets `e, e' ∈ Endset`, the predicate `coverage(e) = coverage(e')` is decidable using only T2 comparisons and TumblerAdd. *Proof.* By `Endset = 𝒫_fin(Span)` (ASN-0043), `e ∪ e'` is finite, so `coverage(e)` and `coverage(e')` are finite unions of half-open T1-intervals `[s, s ⊕ ℓ) = {t : s ≤ t < s ⊕ ℓ}` (T12, SpanWellDefinedness, ASN-0034), each upper endpoint `s ⊕ ℓ` computed by TumblerAdd (ASN-0034). The finite endpoint set `P = {s : (s, ℓ) ∈ e ∪ e'} ∪ {s ⊕ ℓ : (s, ℓ) ∈ e ∪ e'}`, sorted under T1 by T2 (IntrinsicComparison, ASN-0034) into distinct values `c₁ < … < c_m`, partitions `[c₁, c_m)` into finitely many *cells* — the points `{c_k}` and open gaps `(c_k, c_{k+1})`. (If `P = ∅`, i.e. `e = e' = ∅`, both coverages are `∅` and the predicate holds.) No endpoint falls strictly inside a cell, so each coverage is constant — wholly in or wholly out — on every cell, and each is a union of cells. Membership of a cell in a coverage is a finite disjunction over the spans of `e` (resp. `e'`), each disjunct a finite conjunction of T2 comparisons on `P`'s values, exhibiting no interior tumbler.

Both coverages lie within `[c₁, c_m)`, agreeing trivially (both `∅`) on the exterior cells `(−∞, c₁)` and `[c_m, ∞)`. Two finite unions of intervals are equal as tumbler-sets iff they assign the same membership to every tumbler; since each coverage is constant on every cell, it suffices to compare membership at one representative tumbler per *nonempty* cell. A point cell `{c_k}` is nonempty with representative `c_k`. A gap cell `(c_k, c_{k+1})` is nonempty iff it contains a tumbler, decided by a single T2 comparison: the immediate T1-successor of `c_k` is its zero-extension `c_k.0` (T1 case (ii), ASN-0034), and since `c_k < c_{k+1}` forces `c_k.0 ≤ c_{k+1}`, the gap is nonempty iff `c_k.0 ≠ c_{k+1}` — in which case `c_k.0` is a representative (`c_k < c_k.0 < c_{k+1}`). Empty gaps are *skipped*, not compared: they contribute no tumbler to either coverage, so both restrict to `∅` there and an empty gap can neither witness a spurious inequality nor mask a real difference elsewhere. On each nonempty cell the endpoint-based indicator — constant on the cell because no endpoint lies strictly inside — coincides with set-membership of the cell's representative, so comparing the two indicator vectors over the point cells and the provably-nonempty gaps decides `coverage(e) = coverage(e')`. ∎

**Definition — TypedRelation.** For each `K ∈ T_admissible` and state Σ, the *typed relation of type K at Σ* is

`L_K^Σ = {(a, F, G) : a ∈ dom(Σ.L) ∧ |Σ.L(a)| = 3 ∧ Σ.L(a).e₁ = F ∧ Σ.L(a).e₂ = G ∧ coverage(Σ.L(a).e₃) = coverage(K)}`

Each member is a triple of (tuple-address, from-endset, to-endset). The pair `(F, G)` is the *relational content* of the tuple; `a` is the *tuple address*. The `|Σ.L(a)| = 3` conjunct restricts every `L_K` to standard-triple links; by L3 (NEndsetStructure, ASN-0043) the store may also hold higher-arity links (`|Σ.L(a)| > 3`), which then inhabit `A_rel^Σ = dom(Σ.L)` but index no tuple of any `L_K`. The substrate's standard-triple link store at state Σ is therefore the disjoint union over coverage classes:

`L^Σ = ⨆_{[K] ∈ T_admissible / ~} L_K^Σ`

The union is disjoint by Lemma — SliceUniqueness, stated next. Where ambient state is clear we drop the superscript and write `L_K`, `L`.

*Notation — subscript read modulo `~`.* Since the slot-3 criterion tests `coverage(Σ.L(a).e₃) = coverage(K)`, any `K ~ K'` induces `L_K^Σ = L_{K'}^Σ`; the subscript is a *coverage-class* index, depending only on `[K]`.

**Lemma — SliceUniqueness.** Each tuple address `a ∈ dom(Σ.L)` indexes *at most one* slice `L_K^Σ`. *Proof.* `Σ.L` is a partial function `T ⇀ Link` (ASN-0043, Definition of LinkStore), so `a` carries a single value `Σ.L(a)`, hence a single slot-3 endset `Σ.L(a).e₃` and a single coverage class `[Σ.L(a).e₃]`; thus `a` lies in no two slices. ∎

**Definition — TupleAddress.** Define `addr : L^Σ → A_rel^Σ` by `addr(a, F, G) = a`, with codomain `A_rel^Σ = dom(Σ.L)` and image the arity-3 slice `{a ∈ dom(Σ.L) : |Σ.L(a)| = 3}`. That `addr` is an injection is R1 (AddressInjectivity), below.


## Tuple Identity (R0, R1, R2)

Each tuple emission allocates a fresh address (R0), the address-to-pair binding is a function (R1), and the binding is permanent (R2).

**Fact — HomeOriginCoincidence.** On link addresses, `home` and `origin` coincide: both are the NUDE-prefix projection `N(·).0.U(·).0.D(·)` (ASN-0043's `Home`, ASN-0036's `origin`, ASN-0034's T4b field extraction). Consequently `{a ∈ dom(Σ.L) : home(a) = d} = dom(Σ.L) ∩ {ℓ' : origin(ℓ') = d}` for any document `d`.

**L-ContiguousPrefix — ContiguousPrefix.** At every `→*`-reachable state Σ, for every `d ∈ dom(Σ.M)` there exists `J_d^Σ ∈ ℤ_{≥-1}` such that the homed-set is a contiguous initial segment of `A_L(d)`'s chain enumeration, and (when non-empty) admits a unique T1-maximum at chain index `J_d^Σ`:

`(A Σ : Σ →*-reachable :: (A d ∈ dom(Σ.M) :: (E J_d^Σ ∈ ℤ_{≥-1} :: {a ∈ dom(Σ.L) : home(a) = d} = {incʲ(d.0.s_L.1, 0) : 0 ≤ j ≤ J_d^Σ})))`

(with `J_d^Σ = -1` denoting the empty set when no link is homed at `d`).

*Proof.* ChainMembershipForOrigin (ASN-0093, link half) gives, at every reachable state, `dom(Σ.L) ∩ {ℓ' : origin(ℓ') = d} = {s_1, …, s_{n_d}}`, a contiguous initial segment of the link sub-allocator chain `A_L(d)` (anchor `[d.0.s_L.1]` by FirstEmission, sibling recurrence `inc(·, 0)`). The translation to the stated form uses two identifications. First, by HomeOriginCoincidence, `{a ∈ dom(Σ.L) : home(a) = d} = dom(Σ.L) ∩ {ℓ' : origin(ℓ') = d}`. Second, re-index the chain by `j = (chain position) − 1` so that `s_{j+1} = incʲ(d.0.s_L.1, 0)`; then the segment `{s_1, …, s_{n_d}}` is `{incʲ(d.0.s_L.1, 0) : 0 ≤ j ≤ J_d^Σ}` with `J_d^Σ = n_d − 1 ∈ ℤ_{≥-1}` (the empty case `n_d = 0` giving `J_d^Σ = -1`).

*Unique T1-maximum on non-empty homed-sets.* When `J_d^Σ ≥ 0` (the homed-set is non-empty), `max{a ∈ dom(Σ.L) : home(a) = d}` under T1 (LexicographicOrder, ASN-0034) is well-defined and equals `inc^{J_d^Σ}(d.0.s_L.1, 0)`, the chain element at index `J_d^Σ`. By ChainEnumerationInjectivity (ASN-0093) in its strict-order form `(A m, n ≥ 1 : m < n : t_m < t_n)`, the contiguous prefix `{t_1, …, t_{n_d}}` admits `t_{n_d}` as its unique maximum; under the re-indexing, `t_{n_d} = inc^{J_d^Σ}(d.0.s_L.1, 0)`. ∎

**R0 — TupleAddressFreshness.** For any →*-reachable state Σ, any caller-supplied home `d ∈ dom(Σ.M)`, and any `(F, G, K) ∈ Endset × Endset × T_admissible`, there exists a state Σ' with Σ → Σ' that emits a tuple with content (F, G) of type K at an address `a` that is *fresh* against `dom(Σ.L)` and *on-chain* in `A_L(d)`, homed at `d`, with Σ' itself →*-reachable:

`(A Σ : Σ →*-reachable :: (A d ∈ dom(Σ.M), F, G ∈ Endset, K ∈ T_admissible :: (E Σ', a : Σ → Σ' ∧ a ∉ dom(Σ.L) ∧ a ∈ A_L(d) :: a ∈ dom(Σ'.L) ∧ Σ'.L(a) = (F, G, K) ∧ home(a) = d ∧ Σ' →*-reachable)))`

*Value-shape consequence.* The standard triple `(F, G, K)` discharges K.λ's L3 precondition directly from R0's typed hypotheses — arity is 3, both content slots `F, G ∈ Endset`, and `K ∈ T_admissible` forces a non-empty type slot — so the caller discharges no separate value requirement.

*Proof.* Fix the caller-supplied `d ∈ dom(Σ.M)` bound by the universal. We invoke K.λ at home `d` with value `(F, G, K)` ∈ Endset × Endset × T_admissible, which satisfies K.λ's L3-discharge precondition by its typed signature (Value-shape consequence above). K.λ's contract supplies the fresh address `a` via its first/subsequent emission rule, in the branch selected by `a_emit` (Allocator Structure).

- *First emission* (`a_emit`'s first-emission branch fires, under the predicate `{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d} = ∅`): `a = a_emit(Σ, d) = [d.0.s_L.1]`. By FirstEmission (ASN-0093), `E(a)₁ = s_L`, `origin(a) = d` (hence `home(a) = d`), `#E(a) = 2`, `zeros(a) = 3`, and `a` is T4-valid. By ChainDiscipline + FirstEmission (ASN-0093), `A_L(d)` is active at every state with `d ∈ dom(Σ.M)` and `a = t₁^L(d)` is its first emission, so `a ∈ A_L(d)` — discharging the *on-chain admissibility* postcondition. Freshness is FirstEmissionFreshness (ASN-0093), whose link first-emit case (gated by exactly this predicate) gives `a ∉ dom(Σ.L) ∪ dom(Σ.C)` at the committing K.λ-event.
- *Subsequent emission* (`a_emit`'s subsequent-emission branch fires, under `{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d} ≠ ∅`): `a = a_emit(Σ, d) = inc(ℓ_prev, 0)` with `ℓ_prev := max{ℓ' ∈ dom(Σ.L) : origin(ℓ') = d}` (Definition — `a_emit`, well-defined there). *On-chain admissibility (K.λ's "produced by `A_L(d)`" gating precondition).* By ChainMembershipForOrigin (ASN-0093), the homed-set is a contiguous initial segment of `A_L(d)`'s chain enumeration, so its T1-maximum `ℓ_prev` is a chain element and the emission `a = inc(ℓ_prev, 0)` is the next element of `A_L(d)` — so `a ∈ A_L(d)`, discharging the gating precondition. *Shape.* `ℓ_prev ∈ dom(Σ.L)` is T4-valid (L1c, ASN-0043, with T10a.4, ASN-0034); by TA5-SigValid and TA5(c) (ASN-0034), `inc(ℓ_prev, 0)` advances only the terminal component, so `origin(a) = d` (hence `home(a) = d`), `E(a)₁ = s_L`, `zeros(a) = 3`, `#E(a) = #E(ℓ_prev) ≥ 2`, and `a` is T4-valid (TA5a at `k = 0`, ASN-0034). Freshness is SubsequentEmissionFreshness (ASN-0093), whose within-document, cross-document (T10), and cross-subspace (T7) cases jointly give `a ∉ dom(Σ.L) ∪ dom(Σ.C)` at the committing K.λ-event.

In either branch, K.λ's effect is `Σ'.L = Σ.L ∪ {a ↦ (F, G, K)}` with `Σ'.C = Σ.C` and `Σ'.M = Σ.M` per K.λ's Frame, witnessing R0's existential conclusion.

*L-invariant preservation across the K.λ-step.* The emission above is a valid K.λ `→`-step from the `→*`-reachable pre-state Σ — its on-chain gating and freshness preconditions discharged in the freshness bullets. By RT-closure, Σ' is therefore `→*`-reachable (its preservation clause carries the invariant catalog to Σ'). This completes the final conjunct of R0's conclusion. ∎

**R0a — FlatLinkDomain.** At every →*-reachable state Σ, `dom(Σ.L)` is a tumbler-prefix antichain:

`(A Σ : Σ →*-reachable :: (A a, a' ∈ dom(Σ.L) :: a ≼ a' ⟹ a = a'))`

*Proof.* The argument decomposes into two cases on `home(a)` vs. `home(a')`.

*Case 1 — Cross-home (`home(a) ≠ home(a')`).* Let `d = home(a)` and `d' = home(a')` with `d ≠ d'`.

Suppose, toward contradiction, that `a ≼ a'`. Then `a' = a · w` for some suffix `w` (the digits appended to `a` to obtain `a'`). Zero counts add along concatenation: `zeros(a') = zeros(a) + zeros(w)`. By L1 (LinkElementLevel, ASN-0043), `zeros(a) = zeros(a') = 3`, so `zeros(w) = 0` — `w` contains no zero positions. By the `Home` definition (ASN-0043), `home(·) = N(·).0.U(·).0.D(·)` — the prefix of the link extending through the document-field `D(·)` and ending *just before* the third zero. Since `a ≼ a'`, the positions `1..#a` of `a'` agree pointwise with all of `a`; the remaining positions `#a + 1 .. #a'` of `a'` are `w`, which contains no zeros. Therefore every zero of `a'` sits at a position `≤ #a`, and the three zeros of `a'` are *exactly* the three zeros of `a`, at the same positions. In particular, `a'`'s third zero sits at the position of `a`'s third zero — call this position `p₃`, with `p₃ ≤ #a`. The `home` prefix has length `p₃ − 1` (the positions up to and including `D(·)`, which immediately precedes the third zero). Since `p₃ − 1 < p₃ ≤ #a`, the prefix of `a'` of length `p₃ − 1` agrees pointwise with the prefix of `a` of length `p₃ − 1` (by `a ≼ a'` applied at positions `1..#a`); equivalently, `N(a') = N(a)`, `U(a') = U(a)`, and `D(a') = D(a)` — the three NUDE field-components delimited by `a'`'s first three zeros coincide with those of `a` position-by-position. Therefore `home(a') = N(a').0.U(a').0.D(a') = N(a).0.U(a).0.D(a) = home(a) = d`, contradicting `d' ≠ d`. Hence `¬(a ≼ a')`.

With `¬(a ≼ a')`, the R0a implication `a ≼ a' ⟹ a = a'` holds vacuously in this case.

*Case 2 — Same-home (`home(a) = home(a') = d`).* By L-ContiguousPrefix (ContiguousPrefix, established above), the set `{a'' ∈ dom(Σ.L) : origin(a'') = d}` is a contiguous initial segment of `A_L(d)`'s chain enumeration `(t_1, t_2, t_3, …)` with `t_1 = [d.0.s_L.1]` and `t_{n+1} = inc(t_n, 0)`. Hence both `a` and `a'` are chain elements: `a = t_i` and `a' = t_j` for some `i, j ≥ 1`. By (UL), `#a = #t_i = #t_1 = #a'` — all chain elements have equal length. If `a ≼ a'`, then by the prefix definition (positions `1..#a` of `a'` agree with `a`) combined with `#a = #a'`, `a` and `a'` coincide pointwise, so `a = a'` by T3 (CanonicalRepresentation, ASN-0034).

Combining Cases 1 and 2, `a ≼ a' ⟹ a = a'` at every →*-reachable Σ. ∎

**R1 — AddressInjectivity.** The map `addr : L → A_rel` is an injection:

`(A (a, F, G), (a', F', G') ∈ L : a = a' :: F = F' ∧ G = G')`

*Proof.* `Σ.L` is a partial function `T ⇀ Link` (ASN-0043, Definition of LinkStore). Function-ness gives uniqueness of value: if `a = a'`, then `Σ.L(a) = Σ.L(a')`, and that single value determines the from-endset `F` and to-endset `G` stored at `a`. Therefore `F = F'` and `G = G'`. (Slice well-definedness — that `a` indexes exactly one coverage-class slice — is Lemma — SliceUniqueness.) ∎

**R2 — TupleAddressPermanence** is L12 (LinkImmutability, ASN-0043) in tuple vocabulary: an allocated tuple address resolves permanently to the same relational content.

*Consequence.* *Distinct emissions are distinguishable even when content matches.* Two agents independently filing tuples with identical `(F, G)` under identical `K` produce distinct addresses (R0 produces a fresh address regardless of value). By L11b (NonInjectivity, ASN-0043), value-level coincidence is permitted; by R1, address-level identity nevertheless distinguishes them. The substrate does not silently merge them.


## Append-Only Slices (R3)

**R3 — TypedSliceMonotonicity.** Each typed relation grows monotonically:

`(A Σ → Σ', K ∈ T_admissible :: L_K^Σ ⊆ L_K^{Σ'})`

where `L_K^Σ` denotes the typed relation evaluated at state `Σ`.

*Proof.* Let `(a, F, G) ∈ L_K^Σ`. By Definition of `L_K^Σ` (membership at the type slot is by coverage-equivalence, not by literal endset value), `a ∈ dom(Σ.L)` with `Σ.L(a) = (F, G, K'')` for some `K'' ∈ T_admissible` satisfying `coverage(K'') = coverage(K)`. By L12a (LinkStoreMonotonicity, ASN-0043), `dom(Σ.L) ⊆ dom(Σ'.L)`; by R2, `Σ'.L(a) = (F, G, K'')` — the literal value stored at `a` is preserved exactly. The membership test for `L_K^{Σ'}` is `coverage(Σ'.L(a).e₃) = coverage(K)`, i.e., `coverage(K'') = coverage(K)`, which holds by the choice of `K''`. The arity conjunct is likewise preserved: `|Σ'.L(a)| = |Σ.L(a)| = 3` by R2, since the stored value `(F, G, K'')` is preserved exactly. Therefore `(a, F, G) ∈ L_K^{Σ'}`. ∎


## Subspace Disjointness (R4)

**R4 — TupleAddressDisjointness** is SD (StoreDisjointness, ASN-0093: `dom(Σ.C) ∩ dom(Σ.L) = ∅`) under the partition aliases `A_doc^Σ = dom(Σ.C)`, `A_rel^Σ = dom(Σ.L)`, giving `A_doc^Σ ∩ A_rel^Σ = ∅`.


## Self-Reference (R5)

**R5 — TupleSelfTargeting.** A tuple's from-set or to-set may reference tuple addresses. Specifically, for any →*-reachable state Σ and any `a ∈ A_rel^Σ`, the unit-depth span `(a, δ(1, #a))` is well-formed and may appear in the from-set or to-set of an emitted tuple, with `a` in its coverage.

*Proof.* Fix any `a ∈ A_rel^Σ` at any →*-reachable state Σ. By L1a (LinkScopedAllocation, ASN-0043) applied at `a`, `home(a) ∈ dom(Σ.M)`, so `dom(Σ.M) ≠ ∅` — R0's home precondition can be discharged at some allocated document; Σ's `→*`-reachability discharges R0's reachability precondition. The self-targeting emission may be homed at *any* allocated document, not only at `home(a)`.

*(Step 1 — Span well-formedness.)* By L1 (ASN-0043), `zeros(a) = 3`; by L1b (ASN-0043), `#E(a) ≥ 2`, so `#a ≥ 1`. By OrdinalDisplacement (ASN-0034), `δ(1, #a) = [0, …, 0, 1]` is a positive tumbler of length `#a` with action point `#a`. The span `(a, δ(1, #a))` satisfies T12 (SpanWellDefinedness, ASN-0034) — its action point `#a` satisfies `actionPoint(δ(1, #a)) = #a ≤ #a`. By PrefixSpanCoverage (ASN-0043), `coverage({(a, δ(1, #a))}) = {t : a ≼ t}`, which contains `a` by reflexivity of `≼`.

*(Step 2 — Endset admissibility.)* By L4(c) (EndsetGenerality, ASN-0043), endset spans may reference link-subspace addresses. By L13 (ReflexiveAddressing, ASN-0043) applied at `b = a`, the unit-depth span `(a, δ(1, #a))` is the canonical reference span for `a`. The singleton endset `G_self = {(a, δ(1, #a))}` is therefore an admissible `Endset` member at any slot of an emitted link.

*(Step 3 — Self-targeting emission via R0.)* Pick any `d ∈ dom(Σ.M)` and any `K ∈ T_admissible`. The triple `(∅, G_self, K)` is L3-conforming (Value-shape consequence, R0), with `∅, G_self ∈ Endset` (Step 2) the content slots and `K ∈ T_admissible` the type slot. Apply R0 at this triple and home `d`: R0 produces a fresh emitter `a' ∉ dom(Σ.L)` and `→*`-reachable post-state Σ' with `Σ'.L(a') = (∅, G_self, K)`. The self-reference is recorded at the substrate level: `a ∈ coverage(Σ'.L(a').e₂)` — the to-set case.

*(Step 4 — From-set case by parallel emission.)* The from-set case is symmetric. The triple `(G_self, ∅, K)` is L3-conforming (Value-shape consequence, R0), as in Step 3 with the content slots swapped. R0 applied at home `d` yields a fresh emitter address `a''` with conforming post-state Σ'' satisfying `Σ''.L(a'') = (G_self, ∅, K)` and `a ∈ coverage(Σ''.L(a'').e₁)` — the from-set case. ∎

*Consequence.* Self-targeting is what makes the Nullify operation of *Three Operations* possible — a tuple that names another tuple's address in an endset slot, emitted without mutating any existing entry.


## The Active Subset (R6a, R6b, R6c)

**Definition — RetractionType.** Fix a designated coverage class `[R]` reserved for retraction, represented by any `R ∈ T_admissible` whose coverage selects the conventional retraction address set. The corresponding typed relation `L_R^Σ` is the *retraction relation at state Σ*. By L9 (TypeGhostPermission, ASN-0043), `R` need not refer to anything stored — its coverage is an address set, chosen by convention — and `L_R^Σ` is well-defined as a coverage-class slice regardless of whether any literal representative endset has yet been stored.

**Convention — RetractionDirectionality.** For the retraction coverage class `[R]`, the to-set carries the retraction's targets — addresses whose tuples are being withdrawn from the active subset — and the from-set is reserved for attribution-bearing endset content (e.g., the retractor's own address, a self-targeting emission by R5) or is left empty for unattributed retractions. L7 (DirectionalFlexibility, ASN-0043) permits this layer-level naming choice.

**Definition — Nullified.** The set of *nullified* tuple addresses at state `Σ` is

`nullified(Σ) = {a ∈ A_rel^Σ : (E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G'))}`

The existential checks `coverage(G')` only, per Convention RetractionDirectionality, and ranges over `L_R^Σ`, which is triple-restricted by construction (Definition — TypedRelation). The set-builder restriction `a ∈ A_rel^Σ` confines `nullified(Σ)` to link-store addresses: content, document, and ghost addresses in `coverage(G')` are not collected.

**Definition — ActiveSubset.** For each `K ∈ T_admissible`, the *active subset of type K at state Σ* is

`A_K^Σ = {(a, F, G) ∈ L_K^Σ : a ∉ nullified(Σ)}`

`A_K^Σ` is a finite, computable set: `L_K^Σ` is selected by CoverageEqualityDecidable, `nullified(Σ)` by CoverageEqualityDecidable and T2 (IntrinsicComparison, ASN-0034) span-membership, and `A_K^Σ = L_K^Σ \ {(a, F, G) ∈ L_K^Σ : a ∈ nullified(Σ)}`.

**R6a — RetractionStability.** Once a tuple's address is nullified, it stays nullified across all future state transitions:

`(A Σ → Σ', a : a ∈ nullified(Σ) :: a ∈ nullified(Σ'))`

*Proof.* Suppose `a ∈ nullified(Σ)`. By Definition of `nullified(Σ)`, this entails `a ∈ A_rel^Σ = dom(Σ.L)`, and there exist `b ∈ dom(Σ.L)` and `(b, F', G') ∈ L_R^Σ` with `a ∈ coverage(G')`. By the coverage-equivalence membership criterion of `L_R^Σ`, the literal value stored at `b` in Σ is `Σ.L(b) = (F', G', R'')` for some `R'' ∈ T_admissible` with `coverage(R'') = coverage(R)` — the third entry need not equal `R` literally; only its coverage must. We exhibit the same witness at Σ': by L12a (LinkStoreMonotonicity, ASN-0043) applied to `a ∈ A_rel^Σ`, `a ∈ dom(Σ.L) ⊆ dom(Σ'.L) = A_rel^{Σ'}`, discharging the `a ∈ A_rel^{Σ'}` predicate required by Definition of `nullified(Σ')`. By R3 (applied to the type slice indexed by `R`), `L_R^Σ ⊆ L_R^{Σ'}`, so `(b, F', G') ∈ L_R^{Σ'}`. By R2, `b ∈ dom(Σ'.L)` with `Σ'.L(b) = (F', G', R'')` — the literal stored value is preserved exactly, so in particular `G'` is preserved. Since `coverage` is a pure function on endset values, `coverage(G')` is a single fixed set, and `a ∈ coverage(G')` is a state-independent proposition once `G'` has been fixed. Therefore `a ∈ nullified(Σ')`. ∎

**R6b — SingleDepthRetraction.** A retractor's tuple nullifies its targets through a single-pass check over the audit slice, with no regard to the retractor's own status:

`(A Σ, a, b, F', G' : a ∈ A_rel^Σ ∧ (b, F', G') ∈ L_R^Σ ∧ a ∈ coverage(G') : a ∈ nullified(Σ))`

All clauses are evaluated at the single state Σ.

*Proof.* By Definition of `nullified`, `a ∈ nullified(Σ) ⟺ a ∈ A_rel^Σ ∧ (E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G'))`. The three hypotheses `a ∈ A_rel^Σ`, `(b, F', G') ∈ L_R^Σ`, and `a ∈ coverage(G')` discharge this biconditional's right-hand side directly, with `(b, F', G')` as the witness. The membership test consults the audit slice `L_R^Σ`, which retains `b`'s tuple regardless of `b`'s active-subset status — so the result holds even when `b` is itself nullified, making retraction-of-retraction a non-fixpoint operation: nullifying a retractor `b` does not "undo" `b`'s nullifying effect on its prior targets. ∎

**R6c — RestorationByReemission.** Once retracted, a tuple stays out of every active subset at any state reachable from Σ:

`(A Σ, K, (a, F, G) ∈ L_K^Σ : a ∈ nullified(Σ) : (A Σ' : Σ →* Σ' :: (a, F, G) ∉ A_K^{Σ'}))`

*Proof.* Induction on the `→`-chain length `n` witnessing `Σ →* Σ'`. *Base* (`n = 0`): `Σ_0 = Σ`, so `(a, F, G) ∈ L_K^{Σ_0}` and `a ∈ nullified(Σ_0)` are the precondition restated at `Σ_0`; by Definition of `A_K`, `a ∈ nullified(Σ_0)` jointly with `(a, F, G) ∈ L_K^{Σ_0}` give `(a, F, G) ∉ A_K^{Σ_0}`. *IH at `Σ_k`:* `(a, F, G) ∈ L_K^{Σ_k}` and `a ∈ nullified(Σ_k)`. *Step:* R6a gives `a ∈ nullified(Σ_{k+1})`; R3 gives `(a, F, G) ∈ L_K^{Σ_{k+1}}`. *Conclusion at `Σ_n = Σ'`:* by Definition of `A_K`, `(a, F, G) ∉ A_K^{Σ'}`. ∎

To "restore" content, emit a fresh tuple with the desired value (R0). The new tuple receives a fresh address; the retracted tuple keeps its address (R2) and stays out of `A_K` (R6a).

*Consequence — `A_K` is not monotone, though `L_K` is.* R3, lifted along `→*` by the same induction as R6c, makes the audit slice monotone (`Σ →* Σ' ⟹ L_K^Σ ⊆ L_K^{Σ'}`); the active subset is not. A retraction removes a tuple from `A_K` while leaving it in `L_K` — so `A_K` shrinks across that transition, falsifying `⊆`. A later re-emission (R0) admits a fresh tuple at a fresh address into `A_K` — so `A_K` grows across that transition, falsifying `⊇`. Neither inclusion holds in general between `A_K^Σ` and `A_K^{Σ'}`.


## Three Operations

The six properties yield three operations that span every `Σ.L` change the relational layer effects.

**Definition — Emit_K.** `Emit_K` is a family of state-transforming operations indexed by `K ∈ T_admissible`:

`Emit_K : Σ × dom(Σ.M) × Endset × Endset → Σ' × A_rel^{Σ'}`

Where Σ ranges over the `→*`-reachable states. `Emit_K` is `K.λ` of ASN-0093 specialized to value `(F, G, K)`; the address `a_emit` and the function/totality properties follow from R0 (its L3 precondition is discharged by R0's *Value-shape consequence*).

*Precondition.* `K ∈ T_admissible`, `d ∈ dom(Σ.M)`.

*Effect.* Given input state Σ, caller-supplied home document `d ∈ dom(Σ.M)`, and finite endsets `F, G ∈ Endset`, `Emit_K(Σ, d, F, G)` invokes K.λ at home `d` with value `(F, G, K)`. The fresh address is `a = a_emit(Σ, d)` (Definition — `a_emit`, Allocator Structure). The returned `(Σ', a)` satisfies `a ∉ dom(Σ.L)`, `a ∈ dom(Σ'.L)`, `home(a) = d`, and `Σ'.L(a) = (F, G, K)`. By R2, this binding is permanent across all subsequent transitions.

*Frame.* `Σ'.C = Σ.C` and `Σ'.M = Σ.M` (K.λ's frame).

**Definition — Observe_K.** For `K ∈ T_admissible`, a pattern `(F̂, Ĝ) ∈ ℘_fin(T) × ℘_fin(T)`, and a view selector, Observe is a pure read with signature

`Observe_K : Σ × ℘_fin(T) × ℘_fin(T) × View → ℘_fin(L_K^Σ)`

where `View ∈ {hist, oper}` selects between `L_K^Σ` (audit) and `A_K^Σ` (operational). It returns

`{(a, F, G) ∈ view : F̂ ⊆ coverage(F) ∧ Ĝ ⊆ coverage(G)}`

with `view = L_K^Σ` if `View = hist` and `view = A_K^Σ` if `View = oper`. Observe leaves Σ unchanged.

*Pattern domain — `T`, not `A^Σ`.* Patterns range over `T` (not `A^Σ`) so a pattern may target ghost addresses (L9 (TypeGhostPermission, ASN-0043), L4 (EndsetGenerality, ASN-0043)). The match relation `F̂ ⊆ coverage(F)` (and `Ĝ ⊆ coverage(G)`) is decidable because `F̂` is finite and each per-span membership test `t ∈ coverage(F)` is decidable by T2 (IntrinsicComparison, ASN-0034).

**Definition — Nullify.** *Precondition* **P0**: `d_retr ∈ dom(Σ.M)` (the home document must be allocated, discharging the internal `Emit_R`'s K.λ home-precondition). Write **P1** for `a ∈ A_rel^Σ`. *Precondition* **P-tgt**: `P1 ∨ (a = a_emit(Σ, d_retr))` — the target is either an already-allocated relational address (P1) or the call's own self-emit address (self-emit); these two branches are the only admissible targets.

The two branches differ only in when ownership is evaluated: the P1 branch retracts material owned-at-Σ, the self-emit branch retracts material owned-at-commit (the same `→`-step baptizes `a` under `d_retr` and retracts it).

Nullify is the composition

`Nullify(Σ, d_retr, a) ≡ Emit_R(Σ, d_retr, ∅, {(a, δ(1, #a))})`

That is, emit a tuple into the retraction relation with empty from-set and a unit-depth to-span targeting `a`, homed at the caller-supplied `d_retr ∈ dom(Σ.M)` (P0). The to-span `(a, δ(1, #a))` is T12-well-formed for *any* tumbler `a` (`#a ≥ 1` by T0, `actionPoint(δ(1, #a)) = #a ≤ #a`), so R0 at `d_retr` emits the retraction triple `(∅, {(a, δ(1, #a))}, R)`, depositing a fresh emitter address `b` with `Σ'.L(b) = (∅, {(a, δ(1, #a))}, R)`. *Effect:* the retractor `b` is added to `dom(Σ'.L)`; and under P-tgt, `a` lies in the to-coverage and in `A_rel^{Σ'}` (P1 keeps `a ∈ A_rel^Σ ⊆ A_rel^{Σ'}`; the self-emit branch deposits the retractor at `b = a`), so `a ∈ nullified(Σ')`, persisting thereafter by R6a.

**R-Scope — SingleTupleScope.** At every →*-reachable state Σ, for any caller-supplied `d_retr ∈ dom(Σ.M)` and any target `a` admissible under Nullify's precondition P-tgt — that is, `a ∈ A_rel^Σ` (P1) *or* `a = a_emit(Σ, d_retr)` (self-emit) — the `→`-step taken by `Nullify(Σ, d_retr, a) = Emit_R(Σ, d_retr, ∅, {(a, δ(1, #a))})` contributes exactly `a` to the nullified set:

`{t : a ≼ t} ∩ A_rel^{Σ'} = {a}`

where `(Σ', _) = Nullify(Σ, d_retr, a)`. The result is *arity-independent*: it holds regardless of `|Σ.L(a)|`.

*Proof.* By hypothesis Σ is `→*`-reachable, and the K.λ `→`-step carries this to Σ' (RT-closure), so R0a applies at each end. The K.λ `→` step taken by `Emit_R` adds a single fresh emitter address `b` produced by K.λ at `d_retr`: `b ∉ dom(Σ.L)` by K.λ's freshness postcondition, and `b` is deposited at `[d_retr.0.s_L.1]` (first-emission case) or at `inc(ℓ_prev, 0)` (subsequent-emission case) — exactly `b = a_emit(Σ, d_retr)`. So `dom(Σ'.L) = dom(Σ.L) ∪ {b}`, and `a ∈ A_rel^{Σ'}` in both admissible branches: under P1, `a ∈ dom(Σ.L) ⊆ dom(Σ'.L)`; under self-emit, `a = b ∈ dom(Σ'.L)`. We show `{a' ∈ dom(Σ'.L) : a ≼ a'} = {a}` by ruling out every other on-chain extension of `a`. In the P1 case (where `a ∈ dom(Σ.L)`), R0a's antichain at Σ gives `{a' ∈ dom(Σ.L) : a ≼ a'} = {a}`. In the self-emit case (where `a ∉ dom(Σ.L)` — `a = b` is fresh), R0a at Σ has no instance whose first argument is `a`; instead, R0a at Σ' supplies the claim — `a ∈ dom(Σ'.L)`, so for any `a' ∈ dom(Σ.L)`, `a ≼ a' ⟹ a = a'` would contradict `a ∉ dom(Σ.L)`, giving `¬(a ≼ a')` and hence `{a' ∈ dom(Σ.L) : a ≼ a'} = ∅`. The only new key is `b`: `a ⋠ b` by R0a at Σ' applied to `dom(Σ'.L) = dom(Σ.L) ∪ {b}`, which is an antichain (so distinct members `a, b` are prefix-incomparable when both lie in it — and in the self-emit case `a = b` contributes `a` itself, already counted). Combining, `{a' ∈ dom(Σ'.L) : a ≼ a'} = {a}` after the step in each branch, hence `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}`: Nullify's `→` step contributes exactly `a` to `nullified(Σ')`, never a sub-tree of `A_rel`. The argument consults only `a`'s tumbler prefix and R0a's antichain, never the arity `|Σ.L(a)|`, so the conclusion is arity-independent — it holds equally when `a` is a higher-arity address. ∎

**Definition — Unit-depth retraction discipline.** A state Σ is *unit-depth-disciplined* iff every `(b, F', G') ∈ L_R^Σ` has to-endset `G' = {(t, δ(1, #t))}` for some target `t ∈ A_rel^Σ`. Membership `t ∈ A_rel^Σ` is evaluated at the state Σ in question, not at any producing call's pre-state. A *layer* satisfies the *unit-depth retraction discipline* iff every state it reaches is unit-depth-disciplined.

**Definition — relational layer.** The relational layer's link-store operations are `{Emit_K, Observe_K, Nullify}`. Its one *discipline commitment* is a single predicate over `→`-steps: every `→`-step `Σ → Σ'` that grows the retraction slice — i.e. with `L_R^Σ ⊊ L_R^{Σ'}` — is a `Nullify`. This quantifies over *all* `→`-steps that `→ ≡ K.σ ∪ K.α ∪ K.λ` admits (raw `K.λ`, not just the layer aliases).

**Definition — layer-reachable.** A state is *layer-reachable* iff it is `→*`-reachable from `Σ_init` (Definition — Reachability) by a finite sequence of `→`-steps each obeying the discipline commitment (Definition — relational layer).

We discharge the *unit-depth retraction discipline* (Definition — Unit-depth retraction discipline) for every layer-reachable state by induction over such sequences. *Base:* `Σ_init.L = ∅`, so `L_R^{Σ_init} = ∅` and `Σ_init` is unit-depth-disciplined vacuously. *Step:* consider a transition `Σ → Σ'` from a unit-depth-disciplined layer-reachable Σ, split by whether the step grows `L_R`. A substrate step `K.σ` or `K.α` does not touch `Σ.L` (its K-operation frame leaves `Σ.L` fixed), so `L_R^{Σ'} = L_R^Σ`. A non-relational emission `Emit_K` at `K ≁ R` leaves the retraction slice unchanged, `L_R^{Σ'} = L_R^Σ`. A higher-arity K.λ emission (`|Σ.L(a)| > 3`), even at `K ~ R`, cannot grow `L_R` — `L_R^Σ` is triple-restricted by the `|Σ.L(a)| = 3` conjunct of *Definition — TypedRelation*, so the fresh higher-arity tuple inhabits `dom(Σ'.L)` but enters no `L_R`. The only remaining step kind that *can* grow `L_R` is a raw arity-3 K.λ at `K ~ R`. By the discipline commitment, the sole `L_R`-growing step kind (raw arity-3 K.λ at `K ~ R`) is, in any layer-reachable trajectory, a `Nullify` (Definition — layer-reachable). Hence every `L_R`-growing step is a `Nullify`, which adds the single tuple `(b, ∅, {(a, δ(1, #a))})`; its to-span is unit-depth with target `a ∈ A_rel^{Σ'}` by P-tgt (Definition — Nullify). L12a (LinkStoreMonotonicity, ASN-0043) keeps `a ∈ A_rel` at every later state, so the added tuple stays unit-depth-disciplined thereafter. Hence every layer-reachable state is unit-depth-disciplined.


## Weakest-Precondition Analysis

We analyze the weakest preconditions of two postconditions: Nullify's single-tuple scope at Σ' (Case 1) and membership of Emit_K's fresh tuple in the active subset at Σ' (Case 2). Both are written in the standard wp notation `wp(S, R)` — the weakest predicate over the prior state Σ that guarantees the post-state Σ' satisfies R after S executes.

*Case 1 — wp(Nullify(Σ, d_retr, a), "single-tuple scope at Σ'").* The "single-tuple scope" postcondition is `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}` (the to-span's `A_rel`-intersection at Σ' is exactly `a`, with no other link address falling within the prefix-subtree of `a`). Working backward through Nullify's definition `Nullify(Σ, d_retr, a) ≡ Emit_R(Σ, d_retr, ∅, {(a, δ(1, #a))})`, over the ambient `→*`-reachable domain, the weakest precondition is

`P0(Σ, d_retr) ∧ (P1(Σ, a) ∨ a = a_emit(Σ, d_retr))`

— where P0: `d_retr ∈ dom(Σ.M)` and P1: `a ∈ A_rel^Σ`. The `→*`-reachability of Σ is the ambient domain assumption — it supplies R0a's antichain.

*Reduction of the postcondition.* P0 is what executes Nullify and produces a post-state, so it is a conjunct of every precondition that can guarantee the postcondition: dropping P0 admits `d_retr ∉ dom(Σ.M)`, leaving the internal `Emit_R`'s K.λ home-precondition undischarged — Nullify does not execute, no post-state Σ' is produced, and the postcondition is unreachable. Assume P0 henceforth. Write `e = a_emit(Σ, d_retr)` for the fresh emitter address. The internal `Emit_R`'s K.λ `→`-step deposits its retractor tuple at `e` (a fresh key, `e ∉ dom(Σ.L)` by K.λ's freshness postcondition), and modifies no existing entry (L12a pointwise agreement), so `A_rel^{Σ'} = dom(Σ'.L) = dom(Σ.L) ∪ {e} = A_rel^Σ ∪ {e}`.

We claim that, under P0, the postcondition is equivalent to `a ∈ A_rel^{Σ'}`. (⟸) Suppose `a ∈ A_rel^{Σ'} = A_rel^Σ ∪ {e}` — that is, `a ∈ A_rel^Σ` (P1) or `a = e` (self-emit), exactly the two P-tgt-admissible branches. On each branch R-Scope (SingleTupleScope) gives `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}`, the postcondition. (⟹) Suppose the postcondition holds. Its right side `{a}` lies in its left side, so `a ∈ A_rel^{Σ'}`. Hence postcondition `⟺ a ∈ A_rel^{Σ'} = A_rel^Σ ∪ {e} ⟺ a ∈ A_rel^Σ ∨ a = e`, which is exactly `P1(Σ, a) ∨ a = a_emit(Σ, d_retr)`. Conjoining the execution requirement P0 yields the stated weakest precondition.

The two disjuncts are mutually exclusive: if `a ∈ A_rel^Σ = dom(Σ.L)` then `a ≠ e`, since `e` is fresh against `dom(Σ.L)`. Both fall under R-Scope (SingleTupleScope), whose domain `P1 ∨ self-emit` is co-extensive with Nullify's target precondition P-tgt, so the weakest precondition `P0 ∧ (P1 ∨ a = a_emit)` *coincides* with the operation's own precondition `P0 ∧ P-tgt`: every legal Nullify call attains single-tuple scope, and no legal call fails to.

The wp does not constrain `|Σ.L(a)|`, by R-Scope's arity-independence (SingleTupleScope): the scope conclusion holds on each branch regardless of the target's arity.

*Case 2 — wp(Emit_K(Σ, d, F, G), "(a, F, G) ∈ A_K^{Σ'}").* The Definition of `Emit_K` guarantees `(a, F, G) ∈ L_K^{Σ'}` for the fresh emission unconditionally (K.λ deposits `(F, G, K)` at the chain-deterministic address `a`, which is then a member of `L_K^{Σ'}` by coverage-equivalence membership), but is silent on `(a, F, G) ∈ A_K^{Σ'}`, which turns on whether `a ∈ nullified(Σ')`. The post-state retraction slice depends on the K-relation: `L_R^{Σ'} = L_R^Σ ∪ {(a, F, G)}` when `K ~ R`, and `L_R^{Σ'} = L_R^Σ` when `K ≁ R`. The address `a` that `Emit_K(Σ, d, F, G)` deposits is exactly `a_emit(Σ, d)` (Definition — `a_emit`, Allocator Structure).

*Result.* Over the `→*`-reachable working domain, the weakest precondition is

`wp(Emit_K(Σ, d, F, G), (a, F, G) ∈ A_K^{Σ'}) ≡ d ∈ dom(Σ.M) ∧ (K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G)) ∧ ¬(E (b, F', G') ∈ L_R^Σ :: a_emit(Σ, d) ∈ coverage(G'))`   (over `→*`-reachable Σ; `K` is an index, not a free wp variable)

The third conjunct is a *state predicate*, not a trajectory property: it asserts that no pre-existing retraction tuple in `L_R^Σ` already covers the fresh emission address `a_emit(Σ, d)`, and is finitely checkable over the finite slice `L_R^Σ` (L-fin, ASN-0043) by CoverageEqualityDecidable and T2 span-membership.

*Disciplined-domain simplification.* At a layer-reachable state (Definition — layer-reachable) the third conjunct holds vacuously, so the wp reduces to `d ∈ dom(Σ.M) ∧ (K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G))`. The unit-depth retraction discipline gives every pre-existing `L_R^Σ` tuple a unit-depth to-span `{(b, δ(1, #b))}` with coverage `{t : b ≼ t}` for some `b ∈ A_rel^Σ`; the fresh `a = a_emit(Σ, d)` is prefix-incomparable with every such `b` by K.λ's emission rule together with R0a, so `a ∉ coverage(G')` for any pre-existing retraction `(_, _, G') ∈ L_R^Σ`, discharging the third conjunct.

*Derivation (both directions).* `d ∈ dom(Σ.M)` is Emit_K's home-precondition; it is a conjunct of every guaranteeing precondition for the same execution-necessity reason given for P0 in Case 1 (without it K.λ does not execute and no post-state Σ' is produced). With the home-precondition established and the index admissible (per the Result statement above), Σ' exists and `(a, F, G) ∈ L_K^{Σ'}` holds unconditionally (Definition of Emit_K), so the postcondition `(a, F, G) ∈ A_K^{Σ'}` is equivalent to `a ∉ nullified(Σ')`. It therefore suffices to show, over the `→*`-reachable working domain, that `a ∉ nullified(Σ')` is equivalent to the conjunction of the second and third conjuncts of the Result.

`→*`-reachability of Σ gives R0a's antichain at Σ; the K.λ `→`-step carries it to Σ' (RT-closure). The fresh emission lands at `a = a_emit(Σ, d) ∈ dom(Σ'.L) = A_rel^{Σ'}`, so by Definition of `nullified` and `a ∈ A_rel^{Σ'}`, `a ∈ nullified(Σ') ⟺ (E (b, F', G') ∈ L_R^{Σ'} :: a ∈ coverage(G'))`. The post-state retraction slice splits into its pre-existing part and the possible fresh addition: `L_R^{Σ'} = L_R^Σ ∪ {(a, F, G)}` when `K ~ R`, and `L_R^{Σ'} = L_R^Σ` when `K ≁ R`. A *pre-existing* tuple covers `a` iff `(E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G'))`; the *fresh* tuple lies in `L_R^{Σ'}` and covers `a` iff `K ~ R ∧ a_emit(Σ, d) ∈ coverage(G)`. Hence

`a ∈ nullified(Σ') ⟺ (E (b, F', G') ∈ L_R^Σ :: a_emit(Σ, d) ∈ coverage(G')) ∨ (K ~ R ∧ a_emit(Σ, d) ∈ coverage(G))`.

Negating both sides, `a ∉ nullified(Σ') ⟺ ¬(E (b, F', G') ∈ L_R^Σ :: a_emit(Σ, d) ∈ coverage(G')) ∧ (K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G))` — exactly the third and second conjuncts of the Result. The `a_emit(Σ, d) ∉ coverage(G)` escape branch is non-redundant: a `K ~ R` call with `G = ∅` (empty coverage) leaves `a ∉ nullified(Σ')`, so the postcondition holds where a bare `K ≁ R` would wrongly reject it. Both the necessary and the sufficient direction are thereby established, and with `(a, F, G) ∈ L_K^{Σ'}` unconditional, the stated formula is the weakest precondition over the `→*`-reachable working domain. (Over the layer-reachable sub-domain the third conjunct is vacuous — see the Disciplined-domain simplification above — recovering the two-conjunct form.)

## Worked Sketch

We illustrate the structure of a retraction cycle in the relational vocabulary, building on the ASN-0043 worked example. Concrete tumbler values are fixed up front; the cycle proceeds in five steps (Step 0 through Step 4), each step's header naming what it exercises.

*Setup.* Fix:

- `s_L = 2` (link subspace identifier — matching ASN-0093 SubspaceConventionAxiom and the ASN-0043 worked example).
- `d = 1.0.1.0.1` — document address, `zeros(d) = 2`, length `5`, T4-valid; `d ∈ dom(Σ_{-1}.M)` (already allocated by some prior K.σ step at or before `Σ_{-1}`).
- `c₁ = 1.0.1.0.1.0.1.1`, `c₂ = 1.0.1.0.1.0.1.2` — two content addresses in `dom(Σ_{-1}.C)`, both with `subspace_I = 1 = s_C`, `zeros = 3`, depth `8`. Both result from prior K.α invocations at home `d`: `c₁` is the first emission of `A_C(d)` per FirstEmission (ASN-0093) (concretely `c₁ = [d.0.s_C.1]` with `s_C = 1`), and `c₂ = inc(c₁, 0)` is the second per the sibling recurrence — placing the example state in the substrate's K-operation vocabulary, parallel to Step 0's K.λ first-emission of `a₁` from `A_L(d)`.
- `k = 3`, `r = 4` — single-component ghost addresses for the classification type `K = {(k, δ(1, 1))}` and the retraction coverage class `[R]` with `R = {(r, δ(1, 1))}`. By construction `coverage(K) ∩ coverage(R) = ∅` (first components 3 and 4 differ; no tumbler extends both prefixes); `K` and `R` lie in distinct coverage classes. *T4-validity note.* Type-endset ghost addresses (per L9, TypeGhostPermission, ASN-0043) need not satisfy T4 — `T4-valid(·)` is required only of allocator outputs under T10a (S7d for documents, ASN-0093 L1c for links). We use single-component ghost addresses here for legibility.
- `F₁ = {(c₁, δ(1, 8))}`, `G₁ = {(c₂, δ(1, 8))}` — singleton-span endsets covering `c₁` and `c₂` respectively (by PrefixSpanCoverage).
- `Σ_{-1}.L = ∅`, so `L_K^{Σ_{-1}} = ∅`, `L_R^{Σ_{-1}} = ∅`, `nullified(Σ_{-1}) = ∅`, `A_K^{Σ_{-1}} = ∅`.

*Step 0 — first-emission case: K.λ at `d` from empty homed-set, exhibiting `a₁`.* `Σ_{-1} → Σ_0` via `K.λ` (equivalently `Emit_K(Σ_{-1}, d, F₁, G₁)`) emitting `(F₁, G₁, K)` at home `d`. ASN-0093's K.λ first-emission predicate `{ℓ' ∈ dom(Σ_{-1}.L) : origin(ℓ') = d} = ∅` fires (`dom(Σ_{-1}.L) = ∅`), so K.λ deposits at `[d.0.s_L.1]`. Computing concretely: `d = 1.0.1.0.1`, so `d.0` extends `d` with a zero at position 6 to give `1.0.1.0.1.0`; `d.0.s_L = 1.0.1.0.1.0.2`; and `d.0.s_L.1 = 1.0.1.0.1.0.2.1`. So `a₁ := [d.0.s_L.1] = 1.0.1.0.1.0.2.1` (`= t_1^L(d)` by FirstEmission, ASN-0093).

K.λ's effect at this step deposits `Σ_0.L = {a₁ ↦ (F₁, G₁, K)}` with `Σ_0.M = Σ_{-1}.M` and `Σ_0.C = Σ_{-1}.C` per K.λ's Frame. Verification at `a₁`: `zeros(a₁) = 3`, `E(a₁) = [2, 1]`, `E(a₁)₁ = 2 = s_L`, `#E(a₁) = 2`, T4-valid, `origin(a₁) = home(a₁) = 1.0.1.0.1 = d`. ✓ FirstEmissionFreshness (ASN-0093) gives `a₁ ∉ dom(Σ_{-1}.L) ∪ dom(Σ_{-1}.C)` at the K.λ-event committing `a₁`. By R0 (TupleAddressFreshness) and R1 (AddressInjectivity), `a₁` is a fresh, distinct tuple address.

After Step 0: `L_K^{Σ_0} = {(a₁, F₁, G₁)}` (witnessing R3 over the empty `L_K^{Σ_{-1}}`); `L_R^{Σ_0} = ∅`; `nullified(Σ_0) = ∅`; `A_K^{Σ_0} = L_K^{Σ_0} = {(a₁, F₁, G₁)}`. By L-ContiguousPrefix at Σ_0 with `J_d^{Σ_0} = 0`, the homed-link set at `d` is the singleton prefix `{a₁} = {inc⁰(d.0.s_L.1, 0)}` of `A_L(d)`'s chain enumeration. ✓

*Step 1: Nullify a₁.* `Σ_0 → Σ_1` via `Nullify(Σ_0, d, a₁) = Emit_R(Σ_0, d, ∅, {(a₁, δ(1, 8))})`. This emission's to-set `{(a₁, δ(1, 8))}` references the link address `a₁` — witnessing *R5* (TupleSelfTargeting): the to-set of an `L_R` tuple refers to another link's address.

Emit_R invokes K.λ at home `d`. The first/subsequent emission predicate fires *subsequent* (since `{ℓ' ∈ dom(Σ_0.L) : origin(ℓ') = d} = {a₁} ≠ ∅`); `ℓ_prev := max{ℓ' ∈ dom(Σ_0.L) : origin(ℓ') = d} = a₁`; K.λ deposits at `inc(a₁, 0) = 1.0.1.0.1.0.2.2`. Set `b₁ = 1.0.1.0.1.0.2.2` — by ChainEnumerationInjectivity (ASN-0093), `b₁` is the second chain element of `A_L(d)` (the first being `a₁ = t_1^L(d)`). By T10a.2 (NonNestingSiblingPrefixes, ASN-0034), `a₁` and `b₁` are distinct siblings of `A_L(d)` and are therefore prefix-incomparable; in particular `a₁ ⋠ b₁` — witnessing *R0* (TupleAddressFreshness): `b₁ ∉ dom(Σ_0.L)` is fresh by SubsequentEmissionFreshness (ASN-0093), the subsequent-emission branch the realized prefix `{a₁}` at Σ_0 selects (matching R0's own subsequent-branch discharge).

Each emission's L-invariants hold by R0. ✓

Emit the retraction: `Σ_1.L = Σ_0.L ∪ {b₁ ↦ (∅, {(a₁, δ(1, 8))}, R)}`. Now compute:

- `coverage({(a₁, δ(1, 8))})`: by PrefixSpanCoverage with `#a₁ = 8`, `= {t : a₁ ≼ t}`. Membership: `a₁ ∈ coverage` by reflexivity of `≼`; `b₁ ∉ coverage` since `a₁` and `b₁` agree on positions `1..7` (both `1.0.1.0.1.0.2`) but differ at position `8` (`1` vs `2`) at equal length — neither is a prefix of the other. ✓
- `L_K^{Σ_1} = {(a₁, F₁, G₁)}` — unchanged. Witnesses *R3* (TypedSliceMonotonicity): `L_K^{Σ_0} = {(a₁, F₁, G₁)} ⊆ L_K^{Σ_1}` since the emission targets `L_R`, not `L_K`. Also witnesses *R2* (TupleAddressPermanence): `Σ_1.L(a₁) = Σ_0.L(a₁) = (F₁, G₁, K)`. ✓
- `L_R^{Σ_1} = {(b₁, ∅, {(a₁, δ(1, 8))})}` — the only retraction tuple; no other tuple has type slot coverage-equivalent to `R` (the tuple at `a₁` has type `K` with `coverage(K) ≠ coverage(R)`). Also witnesses *R3* applied to the `R` coverage class: `L_R^{Σ_0} = ∅ ⊆ L_R^{Σ_1}`. ✓
- `nullified(Σ_1) = {a ∈ {a₁, b₁} : a ∈ coverage({(a₁, δ(1, 8))})} = {a₁}`. By Definition of `nullified`, the existential ranges over `L_R^{Σ_1}` (audit slice), so the test is whether `(b₁, ∅, {(a₁, δ(1, 8))}) ∈ L_R^{Σ_1}` directly witnesses `a₁ ∈ coverage(G')` — yes — without recursive evaluation of `b₁`'s status. This exercises the audit-slice quantification (Definition of `nullified`) on which R6b rests. ✓
- `A_K^{Σ_1} = L_K^{Σ_1} \ {(a, F, G) : a ∈ nullified(Σ_1)} = ∅`. ✓

The audit predicate `(a₁, F₁, G₁) ∈ L_K` remains true forever (witnessing *R3*); the operational predicate `(a₁, F₁, G₁) ∈ A_K` flips to false at `Σ_1`.

*Observing the active/audit distinction at Σ_1.* `Observe_K` (Definition — Observe_K, Three Operations) surfaces this flip operationally through its `View` selector. Fix the non-trivial pattern `F̂ = {c₁}`, `Ĝ = ∅`. The subset-match `F̂ ⊆ coverage(F₁)` is genuine, not vacuous: `coverage(F₁) = coverage({(c₁, δ(1, 8))}) = {t : c₁ ≼ t}` (PrefixSpanCoverage, `#c₁ = 8`) contains `c₁` by reflexivity of `≼`, so `{c₁} ⊆ coverage(F₁)`; and `Ĝ = ∅ ⊆ coverage(G₁)` trivially. The tuple `(a₁, F₁, G₁)` therefore matches the pattern, and the two views diverge:

- `Observe_K(Σ_1, {c₁}, ∅, hist)` ranges over `view = L_K^{Σ_1} = {(a₁, F₁, G₁)}`, finds the match, and returns `{(a₁, F₁, G₁)}` — the audit trail still records the now-retracted classification.
- `Observe_K(Σ_1, {c₁}, ∅, oper)` ranges over `view = A_K^{Σ_1} = ∅`, so it returns `∅` — the operational view omits the retracted tuple even though its from-endset still covers `c₁`.

The same matching pattern thus yields the historical tuple under `hist` and nothing under `oper`: the `View` selector is exactly the operational handle on the active/audit distinction. The subset-match `{c₁} ⊆ coverage(F₁)` is decided by the single membership test `c₁ ∈ coverage(F₁)` (T2, IntrinsicComparison, ASN-0034), per the Observe_K decidability note — so the match is verified by computation, not merely asserted decidable. ✓

*Step 2: Restore by re-emission.* To restore the classification, we do *not* attempt to nullify the retraction (which by R6b would be ineffective — single-depth checking ignores it). Instead, `Σ_1 → Σ_2` via `Emit_K(d, F₁, G₁)`, re-using the same home `d` as `a₁`. K.λ at home `d` evaluates the subsequent-emission predicate: `{ℓ' ∈ dom(Σ_1.L) : origin(ℓ') = d} = {a₁, b₁} ≠ ∅`; `ℓ_prev := max{a₁, b₁} = b₁` (by T1 lexicographic order, `b₁ > a₁` since they share prefix `1.0.1.0.1.0.2` and differ at position 8 by `2 > 1`); K.λ deposits at `inc(b₁, 0) = 1.0.1.0.1.0.2.3`. Set `a₂ = 1.0.1.0.1.0.2.3` — `A_L(d)`'s third chain element. *R0* witness: `a₂ ∉ dom(Σ_1.L)` is fresh by SubsequentEmissionFreshness (ASN-0093); *R1* (AddressInjectivity) witness: the new tuple address `a₂` is distinct from both `a₁` and `b₁`, so the map `addr` remains injective. The L-invariants at `a₂` discharge as at `b₁`, with element-field ordinal `E(a₂) = [2, 3]`; by L-ContiguousPrefix at Σ_2, `a₂ = inc²(d.0.s_L.1, 0)` and `a₁, b₁, a₂` are `A_L(d)`'s first three chain elements in order.

Then `Σ_2.L = Σ_1.L ∪ {a₂ ↦ (F₁, G₁, K)}` and:

- `L_K^{Σ_2} = {(a₁, F₁, G₁), (a₂, F₁, G₁)}` — two coverage-class members with identical `(F, G)` at distinct addresses. Witnesses *R3* (monotone extension `L_K^{Σ_1} ⊆ L_K^{Σ_2}`), *R1* (distinct addresses for the two tuples), and *L11b/R2 Consequence* (distinct emissions distinguishable even when content matches). ✓
- `nullified(Σ_2) = {a₁}` — unchanged. Witnesses *R6a* (RetractionStability): `a₁ ∈ nullified(Σ_1) ⟹ a₁ ∈ nullified(Σ_2)`. The only `L_R` tuple is still at `b₁`, whose `coverage(G')` contains `a₁` but not `a₂` since `a₁` and `a₂` are distinct siblings in `A_L(d)`. Deciding `a₂ ∈ nullified(Σ_2)` again requires only the single-pass audit-slice check over `L_R^{Σ_2}` (Definition of `nullified`), which finds no witnessing tuple. ✓
- `A_K^{Σ_2} = {(a₂, F₁, G₁)}` — the new tuple is active; `a₁` remains in `L_K` but excluded from `A_K` by *R6c* (RestorationByReemission: `(a₁, F₁, G₁) ∈ L_K^{Σ_2} \ A_K^{Σ_2}` for the retracted historical record, and the restoration is the fresh `(a₂, F₁, G₁) ∈ A_K^{Σ_2}` at a different address). ✓

The relational content `(F₁, G₁)` is again present in `A_K`, but at a different tuple address. Provenance and audit cleanly distinguish the two emissions: `a₁` is the historical record, `a₂` is the current assertion.

*Step 3 — Retracting the retractor exhibits R6b's non-fixpoint semantics.* `Σ_2 → Σ_3` via `Nullify(Σ_2, d, b₁) = Emit_R(Σ_2, d, ∅, {(b₁, δ(1, 8))})` — a retraction whose to-set targets the retractor `b₁` itself. Emit_R invokes K.λ at home `d`. The first/subsequent emission predicate fires *subsequent* (since `{ℓ' ∈ dom(Σ_2.L) : origin(ℓ') = d} = {a₁, b₁, a₂} ≠ ∅`); `ℓ_prev := max{a₁, b₁, a₂} = a₂` (by T1 lex order on the shared prefix `1.0.1.0.1.0.2` with last components `1 < 2 < 3`); K.λ deposits at `inc(a₂, 0) = 1.0.1.0.1.0.2.4`. Set `b₂ = 1.0.1.0.1.0.2.4` — `A_L(d)`'s fourth chain element, fresh against `dom(Σ_2.L)` by SubsequentEmissionFreshness (ASN-0093). The L-invariants at `b₂` discharge as at `b₁`, with element-field ordinal `E(b₂) = [2, 4]`.

Then `Σ_3.L = Σ_2.L ∪ {b₂ ↦ (∅, {(b₁, δ(1, 8))}, R)}` and:

- `L_K^{Σ_3} = {(a₁, F₁, G₁), (a₂, F₁, G₁)}` — unchanged from Σ_2; the new emission targets `L_R`, not `L_K`. Witnesses *R3*. ✓
- `L_R^{Σ_3} = {(b₁, ∅, {(a₁, δ(1, 8))}), (b₂, ∅, {(b₁, δ(1, 8))})}` — both retraction tuples persist (*R3* applied to the `R` coverage class; *R2* preserves the original retraction tuple at `b₁`). ✓
- *Deciding `a₁ ∈ nullified(Σ_3)`.* The tuple `(b₁, ∅, {(a₁, δ(1, 8))}) ∈ L_R^{Σ_3}` has `coverage(G') = {t : a₁ ≼ t}` ∋ `a₁` by reflexivity of `≼`; witness found, so `a₁ ∈ nullified(Σ_3)` — by R6b's single-pass check over the audit slice `L_R^{Σ_3}`, independent of `b₁`'s own status. ✓
- *Deciding `b₁ ∈ nullified(Σ_3)`.* The new tuple `(b₂, ∅, {(b₁, δ(1, 8))})` has `coverage(G') = {t : b₁ ≼ t}`; `b₁ ∈ coverage(G')` by reflexivity. Witness found. `b₁ ∈ nullified(Σ_3)`. ✓
- Computing `nullified(Σ_3) = {a ∈ A_rel^{Σ_3} : (E witness)}`: by inspection on each member of `A_rel^{Σ_3} = {a₁, b₁, a₂, b₂}`:
  - `a₁`: witness `(b₁, …)` above ⟹ `a₁ ∈ nullified(Σ_3)`.
  - `b₁`: witness `(b₂, …)` above ⟹ `b₁ ∈ nullified(Σ_3)`.
  - `a₂`: neither retraction tuple's to-coverage contains `a₂` (a₁ ⋠ a₂ via R0a; b₁ ⋠ a₂ via R0a) ⟹ `a₂ ∉ nullified(Σ_3)`.
  - `b₂`: neither retraction tuple's to-coverage contains `b₂` (a₁ ⋠ b₂ via R0a; b₁ ⋠ b₂ via R0a) ⟹ `b₂ ∉ nullified(Σ_3)`.
  Therefore `nullified(Σ_3) = {a₁, b₁}`. *R6a* witnessed: `a₁ ∈ nullified(Σ_2) ⟹ a₁ ∈ nullified(Σ_3)`. ✓
- `A_K^{Σ_3} = L_K^{Σ_3} \ {(a, F, G) : a ∈ nullified(Σ_3)} = {(a₂, F₁, G₁)}` — *unchanged from `A_K^{Σ_2}`*. The retraction of the retractor `b₁` has no operational effect on the active subset of `K`: `(a₂, F₁, G₁)` remains active because `a₂ ∉ nullified(Σ_3)`, and `(a₁, F₁, G₁)` remains excluded because the original retraction tuple at `b₁` still witnesses `a₁ ∈ nullified(Σ_3)` independently of `b₁`'s own status. ✓
- `A_R^{Σ_3} = L_R^{Σ_3} \ {(a, F, G) : a ∈ nullified(Σ_3)} = {(b₂, ∅, {(b₁, δ(1, 8))})}` — the original retractor at `b₁` is excluded (`b₁ ∈ nullified(Σ_3)`) yet, per R6b, still witnesses `a₁ ∈ nullified(Σ_3)` because `nullified` ranges over the audit slice `L_R^{Σ_3}`, not `A_R^{Σ_3}`. ✓

The retraction of the retractor leaves `A_K^{Σ_3}` unchanged (R6b): restoring `(F₁, G₁)` to active assertion requires fresh emission at a fresh address (Step 2's pattern), not retraction-of-retraction, which only grows `L_R`.

*Step 4 — a self-nullifying emission verifies the wp Case 2 false branch.* The wp of Case 2 (`wp(Emit_K(Σ, d, F, G), (a, F, G) ∈ A_K^{Σ'})`) collapses to false exactly when an `Emit_K` call at a type `K ~ R` lands its own deterministic emission address in its to-coverage; the Result's load-bearingness argument asserts this abstractly, and we now instantiate it. The fresh emission address at `d` in Σ_3 is `a_emit(Σ_3, d)`: the subsequent-emission branch fires (`{ℓ' ∈ dom(Σ_3.L) : origin(ℓ') = d} = {a₁, b₁, a₂, b₂} ≠ ∅`), `ℓ_prev := max{a₁, b₁, a₂, b₂} = b₂` (shared prefix `1.0.1.0.1.0.2`, last components `1 < 2 < 3 < 4`), so `a_emit(Σ_3, d) = inc(b₂, 0) = 1.0.1.0.1.0.2.5`. Set `a₃ = 1.0.1.0.1.0.2.5` — `A_L(d)`'s fifth chain element, with `#a₃ = 8`. Because `a₃` is caller-computable from `(Σ_3, d)` *before* the emission commits, a caller can target it: `Σ_3 → Σ_4` via `Emit_R(Σ_3, d, ∅, {(a₃, δ(1, 8))})` — i.e. `Emit_K` at `K := R`, `F := ∅`, `G := {(a₃, δ(1, #a₃))}`, the to-set rooted at the call's own emission address. (With `#a₃ = 8`, `Emit_R(Σ_3, d, ∅, {(a₃, δ(1, 8))}) = Nullify(Σ_3, d, a₃)` by Definition — Nullify: this is the self-emit branch of Nullify's target precondition P-tgt. Here P1 (`a ∈ A_rel^Σ`) is false (`a₃ ∉ dom(Σ_3.L)`), but the self-emit disjunct `a₃ = a_emit(Σ_3, d)` holds, so P-tgt is satisfied; with P0 (`d ∈ dom(Σ_3.M)`) also holding, the call is legal and executes as the self-emit instance of wp Case 1.)

K.λ deposits `(∅, {(a₃, δ(1, 8))}, R)` at `a₃ = a_emit(Σ_3, d)`, so `Σ_4.L = Σ_3.L ∪ {a₃ ↦ (∅, {(a₃, δ(1, 8))}, R)}`. Compute:

- *Both disjuncts of the wp's second conjunct fail.* `K = R`, so `K ~ R` holds — the first disjunct `K ≁ R` fails. And `coverage({(a₃, δ(1, 8))})`: by PrefixSpanCoverage with `#a₃ = 8`, `= {t : a₃ ≼ t}`, which contains `a₃` by reflexivity of `≼` — so `a_emit(Σ_3, d) = a₃ ∈ coverage(G)`, and the second disjunct `a_emit(Σ_3, d) ∉ coverage(G)` fails. Meanwhile the home-precondition `d ∈ dom(Σ_3.M)` holds. ✓
- `L_R^{Σ_4} = L_R^{Σ_3} ∪ {(a₃, ∅, {(a₃, δ(1, 8))})}` — the fresh tuple is type-`R` (its slot-3 coverage equals `coverage(R)`), so it joins the very retraction slice it can witness against.
- *Self-nullification.* The fresh tuple `(a₃, ∅, {(a₃, δ(1, 8))}) ∈ L_R^{Σ_4}` has `coverage(G') ∋ a₃`, witnessing `a₃ ∈ nullified(Σ_4)` by the single-pass audit-slice check (Definition of `nullified`). The emission nullifies its own address. ✓
- `A_R^{Σ_4} = L_R^{Σ_4} \ {(a, F, G) : a ∈ nullified(Σ_4)}`: since `a₃ ∈ nullified(Σ_4)`, the fresh tuple `(a₃, ∅, {(a₃, δ(1, 8))}) ∉ A_R^{Σ_4}`. The emission is born inactive. ✓

Thus the fresh tuple lies in `L_R^{Σ_4}` (audit) but not in `A_R^{Σ_4}` (operational): for this `Emit_R` call the wp postcondition `(a, F, G) ∈ A_K^{Σ'}` *fails* though its home-precondition `d ∈ dom(Σ_3.M)` is met. This is the concrete instance of the disjunction's false branch — the home-precondition alone does not suffice.

*L-ContiguousPrefix verification at Σ_2 and Σ_3.* Both Σ_2 and Σ_3 are `→*`-reachable (each is built by a chain of `Emit_K`/K.λ `→`-steps), so L-ContiguousPrefix here is its reachable case, which coincides with ChainMembershipForOrigin (ASN-0093). The set of link addresses homed at `d` is `{a₁, b₁, a₂} = {incʲ(d.0.s_L.1, 0) : 0 ≤ j ≤ 2}` at Σ_2 — a contiguous prefix of `A_L(d)`'s chain enumeration — so L-ContiguousPrefix holds at Σ_2 with `J_d^{Σ_2} = 2`. At Σ_3, the homed set extends contiguously to `{a₁, b₁, a₂, b₂} = {incʲ(d.0.s_L.1, 0) : 0 ≤ j ≤ 3}`, so L-ContiguousPrefix holds at Σ_3 with `J_d^{Σ_3} = 3`. ✓ Each of `a₁ = 1.0.1.0.1.0.2.1`, `b₁ = 1.0.1.0.1.0.2.2`, `a₂ = 1.0.1.0.1.0.2.3`, `b₂ = 1.0.1.0.1.0.2.4` has element-field projection of length 2 (E = `[2, 1]`, `[2, 2]`, `[2, 3]`, `[2, 4]` respectively) by (UL). ✓


## Properties Introduced

| Label | Type | Statement |
|-------|------|-----------|
| A^Σ | DEF | Address universe at state Σ: `dom(Σ.C) ∪ dom(Σ.L)` |
| A_doc^Σ, A_rel^Σ | DEF | Partition of `A^Σ` into content addresses (`dom(Σ.C)`) and link-store addresses (`dom(Σ.L)`) |
| T_admissible | DEF | Admissible types: `{K ∈ Endset : K ≠ ∅}` — the indexing domain for typed relations |
| ~ | DEF | TypeEquivalence: `K ~ K' ≡ coverage(K) = coverage(K')` — coverage-equivalence on admissible types (= L8 lifted) |
| L_K^Σ | DEF | Typed relation (coverage-class slice): `{(a, F, G) : a ∈ dom(Σ.L) ∧ |Σ.L(a)| = 3 ∧ Σ.L(a).e₁ = F ∧ Σ.L(a).e₂ = G ∧ coverage(Σ.L(a).e₃) = coverage(K)}` |
| L^Σ | DEF | Standard-triple link store: `⨆_{[K] ∈ T_admissible / ~} L_K^Σ` |
| addr | DEF | Map `(a, F, G) ↦ a : L^Σ → A_rel^Σ`, injection with image the arity-3 slice `{a : |Σ.L(a)| = 3}` |
| nullified(Σ) | DEF | Tuple addresses targeted by some `L_R^Σ` to-set |
| A_K^Σ | DEF | Active subset: `{(a, F, G) ∈ L_K^Σ : a ∉ nullified(Σ)}` |
| → | DEF | Dom-extending state transition relation `→ ≡ K.σ ∪ K.α ∪ K.λ` |
| Unit-depth retraction discipline | COMMITMENT | Layer-level convention: every `L_R^Σ` tuple has to-endset of the form `{(b, δ(1, #b))}` for some target `b ∈ A_rel^Σ` (Three Operations) |
| R0 | LEMMA | TupleAddressFreshness — emission allocates an address fresh against `dom(Σ.L)` and on-chain in `A_L(d)`, homed at `d` |
| L-ContiguousPrefix | LEMMA | ContiguousPrefix — `{a ∈ dom(Σ.L) : home(a) = d} = {incʲ(d.0.s_L.1, 0) : 0 ≤ j ≤ J_d^Σ}` for some `J_d^Σ ∈ ℤ_{≥-1}`, with unique T1-maximum at chain index `J_d^Σ` when non-empty (= ChainMembershipForOrigin, ASN-0093, link half, restated at `→*`-reachable states) |
| R0a | LEMMA | FlatLinkDomain — `dom(Σ.L)` is an antichain in `≼` at every →*-reachable state |
| R1 | LEMMA | AddressInjectivity — `addr` is an injection (= function property of `Σ.L`) |
| R2 | ALIAS | TupleAddressPermanence — addresses persist with values intact (definitional alias of L12) |
| R3 | LEMMA | TypedSliceMonotonicity — each `L_K^Σ` is monotone |
| R4 | ALIAS | TupleAddressDisjointness — `A_doc^Σ ∩ A_rel^Σ = ∅` (definitional alias of SD, StoreDisjointness, ASN-0093) |
| R5 | LEMMA | TupleSelfTargeting — for any `a ∈ A_rel^Σ`, the span `(a, δ(1, #a))` is admissible as an endset member |
| R-Scope | LEMMA | SingleTupleScope — for any P-tgt-admissible target `a` (P1 or self-emit), `Nullify(Σ, d_retr, a)`'s `→`-step gives `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}`; arity-independent |
| R6a | LEMMA | RetractionStability — once nullified, always nullified |
| R6b | DEF-Consequence | SingleDepthRetraction — `nullified` quantifies over the audit slice `L_R^Σ`, not `A_R^Σ`, so a retractor nullifies its targets independent of its own status (retraction-of-retractor is a non-fixpoint) |
| R6c | LEMMA | RestorationByReemission — restoration is fresh emission, never retraction-of-retraction |
| Relational layer | COMMITMENT | Operation set `{Emit_K, Observe_K, Nullify}`; layer invokes `Emit_K` at `K ~ R` only via the `Nullify` alias, satisfying the unit-depth retraction discipline; see Definition — relational layer |
| Emit_K | OP | State-transforming: `Σ × dom(Σ.M) × Endset × Endset → Σ' × A_rel^{Σ'}`, operationally K.λ specialized to value `(F, G, K)`; caller-supplied home `d ∈ dom(Σ.M)` and `K ∈ T_admissible` (Three Operations) |
| Observe_K | OP | Pure read: `Σ × ℘_fin(T) × ℘_fin(T) × View → ℘_fin(L_K^Σ)`, selecting `L_K^Σ` or `A_K^Σ` (Three Operations) |
| Nullify | OP | `Nullify(Σ, d_retr, a) ≡ Emit_R(Σ, d_retr, ∅, {(a, δ(1, #a))})`; precondition P0 ∧ P-tgt (target `a ∈ A_rel^Σ` or self-emit) |

## Open Questions

- What invariants must hold between `L_K` and the arrangements `Σ.M` when relational predicates depend on whether the from-set or to-set content is currently visible in some document?
- Should multi-arity links (`|Σ.L(a)| > 3`) define multiple binary projections, or be regarded directly as elements of higher-arity typed relations `L_K^{(n)} ⊆ A_rel × ℘(A)^n`?
- Under what conditions is `Nullify(b)` for `b ∈ L_R` operationally meaningful, given that R6b makes single-depth checking ignore the second-order retraction?
- What ordering, if any, must the substrate guarantee on Observe results — by emission cycle, by tuple address, or unordered as set semantics suggest?
- Must Emit be atomic with respect to concurrent Observe, and if so, what is the consistency model under which `A_K` transitions are observed?
- What guarantees does the substrate provide about the cardinality bound of `nullified(Σ)` relative to `dom(Σ.L)` — is unbounded retraction permitted, or must some structural ratio hold?
- Should L1b's substrate-level admission `#E ≥ 2` (ASN-0043) be tightened to `#E = 2` at the source, or does retaining `#E ≥ 2` leave needed headroom for higher-arity or future element-field variants?
- Should the relational layer's unit-depth retraction discipline (Definition, Three Operations) be elevated to a substrate-level guarantee on `L_R` to-spans — e.g., by introducing a designated K-operation for retraction with a unit-depth shape constraint — or is it correctly a layer convention? The design tradeoff is whether the substrate should expose any value-shape constraint on retraction tuples.
- Can higher layers introduce new admissible types `K ∈ T_admissible` dynamically without coordination, given L9 (TypeGhostPermission), and what happens when two layers independently choose colliding type addresses?
- What discipline must a higher-layer operation — one beyond the K.σ/K.α/K.λ substrate of this note — satisfy for retraction stability (R6a/R6c) to survive its transitions, and is that discipline a structural invariant the substrate can guarantee or one each layer must re-establish?