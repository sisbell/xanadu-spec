> **ASN-0047 · Transition Model** — condensed claim statements  
> [← Full note](ASN-0047-transition-model.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0047 Claim Statements

*Source: ASN-0047-transition-model.md (revised 2026-03-22) — Extracted: 2026-06-02*

## Σ.E — EntitySet (DEF, predicate)

**Σ.E ⊆ T** — the set of allocated entity addresses. Every e ∈ E satisfies T4-valid(e). Entities are organisational — nodes, accounts, documents — not content; element-level addresses live in dom(C), not E:

`(A e ∈ E :: ¬Element(e))`

Equivalently, E ⊆ {t : T4-valid(t) ∧ zeros(t) ≤ 2}.

*Stratification:* By T4c and the exclusion clause above, Σ.E partitions into exactly three strata:
- E_node = {e ∈ E : Node(e)}
- E_account = {e ∈ E : Account(e)}
- E_doc = {e ∈ E : Document(e)}

---

## Σ.R — ProvenanceRelation (DEF, predicate)

**Σ.R ⊆ T_elem × E_doc** — where T_elem = {a ∈ T : Element(a)}. The pair (a, d) ∈ R records that document d contained I-address a in its arrangement at the composite boundary at which the entry was made.

---

## Σ₀ — InitialState (DEF, definition)

The initial state Σ₀ = (C₀, L₀, E₀, M₀, R₀) is:

- C₀ = ∅ (no content allocated)
- L₀ = ∅ (no links allocated)
- E₀ = {n₀} where n₀ = `[1]` — the canonical single-component bootstrap node
- M₀(d) = ∅ for all d — (E₀)_doc = ∅, so every arrangement is the empty partial function
- R₀ = ∅ (no provenance recorded)

**Structural form of n₀:** The bootstrap node is fixed as `[1]` — a one-element tumbler with `zeros(n₀) = 0`, satisfying `Node(n₀)` and `T4-valid(n₀)`.

---

## parent(e) — Parent (DEF, function)

For a non-node entity e (where ¬Node(e)), define **parent(e)** using T4b's partial projections N, U, D, E:

- *Account case (Account(e)):* `parent(e) = N(e)` — the node-prefix projection. Since `Account(e)` requires `zeros(e) = 1`, T4b's parse `e = N(e).0.U(e)` is defined with `zeros(N(e)) = 0`, giving `zeros(parent(e)) = 0 = zeros(e) − 1`.
- *Document case (Document(e)):* `parent(e) = N(e).0.U(e)` — the account-prefix projection. Since `Document(e)` requires `zeros(e) = 2`, T4b's parse `e = N(e).0.U(e).0.D(e)` is defined with `zeros(N(e).0.U(e)) = 1`, giving `zeros(parent(e)) = 1 = zeros(e) − 1`.

In each case parent(e) is a valid address at the next higher level: `zeros(parent(e)) = zeros(e) − 1` is a derivable property of T4b's projections, not a stipulation.

---

## Contains(Σ) — CurrentContainment (DEF, function)

The *current containment* of state Σ is the set of all document-content pairs where the content is presently in the document's arrangement:

`Contains(Σ) = {(a, d) : d ∈ E_doc ∧ a ∈ ran(M(d))}`

---

## Contains_C(Σ) — ContentContainment (DEF, function)

`Contains_C(Σ) = {(a, d) : d ∈ E_doc ∧ (E v : v ∈ dom(M(d)) ∧ subspace(v) = s_C : M(d)(v) = a)}`

---

## Valid composite — ValidCompositeAmended (DEF, definition)

A composite transition `Σ →* Σ'` in the extended state Σ = (C, L, E, M, R) is *valid* iff it is a finite sequence of atomic transitions `Σ = Σ₀ → Σ₁ → ... → Σₙ = Σ'` — drawn from the atomic vocabulary K.α (amended), K.δ, K.λ, K.μ⁺ (amended), K.μ⁺_L, K.μ⁻ (amended), and K.ρ — satisfying the clauses below. The named composite K.μ~ is not atomic; it may appear in the sequence as shorthand for its K.μ⁻ + K.μ⁺ decomposition.

1. *Transition preconditions (intra-composite sequencing).* Each step `Σᵢ → Σᵢ₊₁` satisfies the *elementary* precondition of its transition kind, evaluated at the *intermediate* state `Σᵢ`.
2. *Coupling constraints (initial-to-final).* J0, J1★, and J1'★ hold for the composite as a whole — evaluated *only* between the initial state Σ and the final state Σ'.

---

## K.α — ContentAllocation (DEF, transition)

ASN-0093's K.α (ContentAllocation), with frame extended by `E' = E ∧ R' = R`. Freshness `a ∉ dom(C) ∪ dom(L)` is SubAllocFresh at `x = C`.

*Effect:* `C' = C ∪ {a ↦ v}`.

*Frame:* `L' = L; (A d :: M'(d) = M(d))` (E and R held in frame).

---

## K.δ — EntityCreation (DEF, transition)

A fresh entity address enters E with initial state:

`E' = E ∪ {e}` where `e ∉ E ∧ T4-valid(e) ∧ ¬Element(e)`

*Precondition* splits on `Node(e)`:

- **Case (i) Node(e):** Required: `T4-valid(e) ∧ Node(e)`, together with freshness and bootstrap-lineage conjuncts from NodeBaptism (a)/(b).
- **Case (ii) ¬Node(e):** `e = inc(t, k)` for some operand `t` and `k ∈ {0, 1, 2}`. Required uniformly: `parent(e) ∈ E`. Per-sub-case:
  - *k = 0 (sibling):* `t ∈ E ∧ ¬Node(t)`
  - *k = 1 (version):* `t ∈ E_doc`
  - *k = 2 (descent):* `t ∈ E ∧ zeros(t) ≤ 1`

*Structural identities on `e = inc(t, k)`:*
- **K.δ-ID.zeros-0/1:** `zeros(e) = zeros(t)` for k ∈ {0, 1}
- **K.δ-ID.zeros-2:** `zeros(e) = zeros(t) + 1` for k = 2
- **K.δ-ID.parent-0/1:** `parent(e) = parent(t)` for k ∈ {0, 1}
- **K.δ-ID.parent-2:** `parent(e) = t` for k = 2

*Frame:* C' = C; L' = L; R' = R. Arrangement frame: `M' = M` for Node(e)/Account(e); `dom(M') = dom(M) ∪ {e}` with `M'(e) = ∅` for Document(e).

---

## K.μ⁺ — ArrangementExtension (DEF, transition)

New V→I mappings are added to some d ∈ E_doc, with existing mappings unchanged:

`dom(M'(d)) ⊃ dom(M(d)) ∧ (A v : v ∈ dom(M(d)) : M'(d)(v) = M(d)(v))`

*Precondition:* `d ∈ E_doc`; for every new mapping M'(d)(v) = a, `a ∈ dom(C)`; new V-positions satisfy S8a, and the resulting arrangement M'(d) satisfies S8-depth; dom(M'(d)) is finite; resulting arrangement satisfies D-CTG and D-MIN.

*Frame:* C' = C; E' = E; (A d' : d' ≠ d : M'(d') = M(d')); R' = R.

---

## K.μ⁻ — ArrangementContraction (DEF, transition)

Existing V→I mappings are removed from some d ∈ E_doc, with surviving mappings unchanged:

`dom(M'(d)) ⊂ dom(M(d)) ∧ (A v : v ∈ dom(M'(d)) : M'(d)(v) = M(d)(v))`

*Precondition:*
- `d ∈ E_doc`
- The caller selects a *retention count* `n' ∈ {0, 1, ..., n}` subject to strict-contraction constraint `n' < n`. The contracted arrangement is `M'(d) = M(d) ↾ R` to the retained domain `R := {[1, 1, ..., 1, k] : 1 ≤ k ≤ n'}`.

*Frame:* C' = C; E' = E; R' = R; (A d' : d' ≠ d : M'(d') = M(d')).

---

## K.μ~ — ArrangementReordering (DEF, transition)

K.μ~ — *arrangement reordering* — is a **named composite** of K.μ⁻ + K.μ⁺. For d ∈ E_doc with `M(d)|_{dom_C}` taking at least two distinct values, K.μ~ realises the *bijection equation*:

`(E π : π is a bijection dom(M(d)) → dom(M'(d)) : (A v ∈ dom(M(d)) :: M'(d)(π(v)) = M(d)(v)))`

π is admissible iff:
- (i) the induced post-state `M'(d)` satisfies the arrangement-shape invariant package S8a, S8-depth, D-CTG★, D-MIN★
- (ii) the net effect is non-trivial: `M'(d) ≠ M(d)`
- (iii) π is *length-preserving:* `(A v ∈ dom(M(d)) :: #π(v) = #v)`
- (iv) π is *subspace-preserving:* `(A v ∈ dom(M(d)) :: subspace(π(v)) = subspace(v))`
- (v) π is *link-subspace fixing:* `(A v ∈ dom_L(M(d)) :: π(v) = v)`

*Frame (derived):* C' = C; E' = E; R' = R; L' = L; (A d' : d' ≠ d : M'(d') = M(d')).

---

## K.λ — LinkAllocation (DEF, transition)

ASN-0093's K.λ (LinkAllocation), with frame extended by `E' = E ∧ R' = R`. Freshness `ℓ ∉ dom(L) ∪ dom(C)` is SubAllocFresh at `x = L`.

*Effect:* `L' = L ∪ {ℓ ↦ (e₁, …, eₙ)}`.

*Frame:* `C' = C; (A d' :: M'(d') = M(d'))` (E and R held in frame).

---

## K.ρ — ProvenanceRecording (DEF, transition)

A document-content association enters R:

`R' = R ∪ {(a, d)}` where `a ∈ dom(C) ∧ d ∈ E_doc`

*Precondition:* `a ∈ dom(C)` ∧ `d ∈ E_doc`. The level constraint Element(a) follows from S7b.

*Frame:* C' = C; L' = L; E' = E; (A d :: M'(d) = M(d)).

---

## K.μ⁺_L — LinkSubspaceExtension (DEF, transition)

Extends a document's arrangement in the link subspace.

*Precondition:*
- d ∈ E_doc
- ℓ ∈ dom(L)
- origin(ℓ) = d
- ℓ ∉ ran(M(d))
- V-position v_ℓ satisfies:
  - subspace(v_ℓ) = s_L
  - If V_{s_L}(d) = ∅: `ValidFirstLinkPosition(d, v_ℓ, m)` — for any chosen `m ≥ 2` it fixes the unique well-formed first link V-position `v_ℓ = [s_L, 1, ..., 1]` of depth `m`
  - If V_{s_L}(d) ≠ ∅: `#v_ℓ = m_L(d)` and `v_ℓ = shift(max(V_{s_L}(d)), 1)`

*Effect:* `M'(d) = M(d) ∪ {v_ℓ ↦ ℓ}`, with `dom(M'(d)) = dom(M(d)) ∪ {v_ℓ} ⊃ dom(M(d))` (strict extension).

*Frame:* `C' = C; L' = L; E' = E; (A d' : d' ≠ d : M'(d') = M(d')); R' = R`.

---

## K.μ~-FIX — ReorderingDomainFixity (LEMMA, lemma)

`dom(M'(d)) = dom(M(d))`.

D-SEQ★ at the pre- and post-states gives `V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` at common depth `m_S` and `V_S(d') = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` at common depth `m'_S` for each subspace S. Since π is a bijection that bijects V_S(d) onto V_S(d'), `n'_S = n_S`. Length preservation (admissibility (iii)) gives `m'_S = m_S`. With both equal, `V_S(d') = V_S(d)`. Taking union over subspaces S, `dom(M'(d)) = dom(M(d))`.

---

## J0 — AllocationPlacementCoupling (COUPLING, predicate)

Content allocation K.α always co-occurs with arrangement extension K.μ⁺:

`(A Σ →* Σ', a : a ∈ dom(C') \ dom(C) : (E d, v : d ∈ E'_doc ∧ v ∈ dom(M'(d)) : M'(d)(v) = a))`

Every freshly allocated I-address appears in some arrangement in the post-state.

---

## J2 — ContractionIsolation (PROP, predicate)

The elementary transition K.μ⁻ requires no coupling — it is self-sufficient with respect to P0–P2, L12, and the operative provenance bound P4★. As an elementary transition, K.μ⁻ satisfies:

`C' = C ∧ L' = L ∧ E' = E ∧ R' = R`

---

## J3 — ReorderingIsolation (PROP, predicate)

The named composite K.μ~ is likewise self-sufficient:

`C' = C ∧ L' = L ∧ E' = E ∧ R' = R`

By **K.μ~-RANGE** (range-invariance), `Contains(Σ') = Contains(Σ)`, and J1★ is vacuous since no range-new content arises across the composite.

---

## J4 — ForkComposite (DEF, definition)

**Definition (Fork).** A *fork* of d_src to d_new is a composite transition `Σ →* Σ'`. Write `d_op` for the *content source operand* of the fork.

**Allocation and operand-tracking rule:**
- k = 1 sub-case fires when `A_v(d_src)` has no prior emission: `d_new = inc(d_src, 1)` and `d_op = d_src`
- k = 0 sub-case fires when `A_v(d_src)` already has a frontier: `d_new = inc(prev_version, 0)` and `d_op = prev_version = max(dom(A_v(d_src)))`

*Precondition:* `d_src ∈ E_doc ∧ d_op ∈ E_doc ∧ V_{s_C}(d_op) ≠ ∅`

It consists of:

(i) a K.δ case (ii) step producing d_new on d_src's version chain `A_v(d_src)`, with d_new ∉ E_doc; in both sub-cases `d_src ≼ d_new`

(ii) K.μ⁺ populating M'(d_new) via the unique order-preserving bijection `φ : V_{s_C}(d_op) → V_{s_C}(d_new)`:

`(A v ∈ V_{s_C}(d_op) :: M'(d_new)(φ(v)) = M(d_op)(v))`

(iii) K.ρ recording provenance for each a ∈ ran(M'(d_new))

and no other elementary steps. Derived consequence: `ran(M'(d_new)) = ran(M(d_op)|_{V_{s_C}(d_op)})`.

---

## P1 — EntityPermanence (INV, predicate)

The entity set admits only extensions:

`(A Σ → Σ' :: E ⊆ E')`

No transition removes an entity. P1 holds uniformly across levels:

`[e ∈ E ∧ Node(e) ⟹ e ∈ E']`
`[e ∈ E ∧ Account(e) ⟹ e ∈ E']`
`[e ∈ E ∧ Document(e) ⟹ e ∈ E']`

---

## P2 — ProvenancePermanence (INV, predicate)

The provenance relation admits only extensions:

`(A Σ → Σ' :: R ⊆ R')`

Once the system records that d referenced a, that record persists.

---

## P4a — TraceWitnessing (PROP, predicate)

A *valid transition trace to Σ* is a finite sequence of composite boundaries `Σ₀ →* Σ₁ →* ... →* Σ_n = Σ` in which each `Σ_j →* Σ_{j+1}` is a valid composite transition; the finite set of states `{Σ₀, ..., Σ_n}` is the *transition history* of Σ along that trace, and `M_k` denotes the arrangement family of trace state `Σ_k`.

`(A valid trace Σ₀ →* ... →* Σ_n = Σ :: (A (a, d) ∈ R :: (E Σ_k ∈ {Σ₀, ..., Σ_n} : (E v ∈ dom(M_k(d)) : subspace(v) = s_C ∧ M_k(d)(v) = a))))`

---

## P6 — ExistentialCoherence (INV, predicate)

For every I-address in the content store, its origin document exists as an entity:

`(A a ∈ dom(C) :: origin(a) ∈ E_doc)`

---

## P7 — ProvenanceGrounding (INV, predicate)

Every provenance entry references allocated content:

`(A (a, d) ∈ R :: a ∈ dom(C))`

---

## P7a — ProvenanceCoverage (PROP, predicate)

Every I-address in the content store has at least one provenance record:

`(A a ∈ dom(C) :: (E d :: (a, d) ∈ R))`

---

## P8 — EntityHierarchy (INV, predicate)

Every non-node entity has its parent in E:

`(A e ∈ E : ¬Node(e) : parent(e) ∈ E)`

---

## m_L(d) — LinkSubspaceDepth (DEF, function)

For a subspace `S ∈ {s_C, s_L}`, `m_S(d)` is the depth of document `d`'s *current* S-subspace arrangement — the common depth that S8-depth fixes on `V_S(d)` whenever that set is non-empty, bounded below by the S8a lower bound `m_S(d) ≥ 2`. `m_L(d)` is the link-subspace instance. `m_S(d)` is well-defined only while `V_S(d) ≠ ∅`. After full clearance of a subspace (`V_S(d) = ∅`), the next insertion re-pins `m_S(d)` from scratch at any value `≥ 2` by S8a.

---

## NodeBaptism — NodeBaptism (AX, axiom)

No docuverse transition mints a node address. At every K.δ node-allocation event — every elementary K.δ transition placing an entity `e` with `Node(e)` into E:

- (a) *Freshness:* `e ∉ Σ.E` at the state Σ of allocation;
- (b) *Bootstrap lineage:* `n₀ ≼ e` under the tumbler-prefix order.

The bootstrap node `n₀ ∈ E₀` is itself baptised at `Σ₀`.

---

## FrontierEquivalence — FrontierEquivalence (LEMMA, lemma)

For every reachable state `Σ` and every operand `t ∈ Σ.E` with `¬Node(t)`, ActivatedEmission supplies an activated entity-level sub-allocator `A` whose domain contains `t` (unique by T10a.6). Then:

`inc(t, 0) ∉ Σ.E ⟺ t is the frontier of A's (t, 0)-branch`

i.e., the operational predicate "the `(t, 0)` increment has not yet been consumed" is logically equivalent to "no prior K.δ event has fired `(t, 0)` on `A`'s chain."

---

## ChildSpawnFreshness — ChildSpawnFreshness (LEMMA, lemma)

For every reachable state `Σ`, every operand `t ∈ Σ.E`, and every `k' ∈ {1, 2}` admissible at `t`:

`inc(t, k') ∉ Σ.E ⟺ the (t, k') child-spawn has not yet been performed`

Note `inc(t, k')` is non-node: for `k' = 2`, K.δ-ID.zeros-2 gives `zeros(inc(t, 2)) = zeros(t) + 1 ≥ 1`; for `k' = 1`, admissibility requires `Document(t)`, so `zeros(inc(t, 1)) = 2`. The preconditions impose no `¬Node(t)` constraint on the operand.

---

## ActivatedEmission — ActivatedEmission (INV, predicate)

Every non-node entity is an emission of an activated entity-level sub-allocator:

`(A e ∈ Σ.E : ¬Node(e) : (E A : Activated(A) ∧ EntityLevel(A) : e ∈ dom(A)))`

Holds vacuously at Σ₀ (E₀ = {n₀} with Node(n₀)); preserved by K.δ (each non-node entity enters E only via a T10a inc-step on an activated sub-allocator, per K.δ case (ii)) and frame on all other transitions.

---

## NodeLineage — NodeLineage (INV, predicate)

`(A e ∈ E : Node(e) : n₀ ≼ e)`, where `≼` is the prefix order on tumblers.

---

## b_C(d), b_L(d) — SubAllocatorAnchors (DEF, function)

For each `d ∈ E_doc`, two element-field bases sit immediately under d:

- `b_C(d) := [d.0.s_C]` (single-component element field with E₁ = s_C; zeros = 3, #E = 1) — the **content sub-allocator anchor**.
- `b_L(d) := [d.0.s_L]` (single-component element field with E₁ = s_L; zeros = 3, #E = 1) — the **link sub-allocator anchor**.

Under SubspaceConventionAxiom (`s_C = 1` and `s_L = 2`): `b_C(d) = inc(d, 2) = [d.0.1]` and `b_L(d) = inc(b_C(d), 0) = [d.0.2]`. The anchors are not themselves in `dom(C) ∪ dom(L)` — content addresses have `#E ≥ 2` (C1b), link addresses have `#E ≥ 2` (L1b), and the anchors have `#E = 1`.

---

## Allocator hierarchy — AllocatorHierarchy (DEF, definition)

Content and link sub-allocators are sibling element-field allocators under d, sharing prefix `[d.0]`; T10a-conformance applies to each frontier separately; cross-document collisions prevented by T10, cross-subspace by L14.

**Sub-allocator names:**
- `A_C(d)` — d's **content sub-allocator**, anchor `b_C(d) = [d.0.s_C]`, first emission `[d.0.s_C.1]`. Outputs `a` satisfy `a ∈ dom(C)`, `subspace_I(a) = s_C`, `origin(a) = d`, `zeros(a) = 3`.
- `A_L(d)` — d's **link sub-allocator**, anchor `b_L(d) = [d.0.s_L]`, first emission `[d.0.s_L.1]`. Outputs `ℓ` satisfy `ℓ ∈ dom(L)`, `subspace_I(ℓ) = s_L`, `origin(ℓ) = d`, `zeros(ℓ) = 3`.
- `A_v(d)` — d's **version sub-allocator**. First emission is `inc(d, 1)`. Outputs inhabit `E_doc`.
- `A_doc(A)` — account A's **document sub-allocator**. First emission is `inc(A, 2)` with `zeros = 2`, `parent(·) = A`. Outputs inhabit `E_doc`.
- `A_account(N)` — node N's **account sub-allocator**. First emission is `inc(N, 2)` with `zeros = 1`, `parent(·) = N`. Outputs inhabit `E_account`.

---

## SubAllocatorBundle — SubAllocatorBundle (LEMMA, lemma)

For each `d ∈ E_doc`, the entity-allocation event placing d into E_doc activates a content sub-allocator `A_C(d)` with anchor `b_C(d) = [d.0.s_C]` and a link sub-allocator `A_L(d)` with anchor `b_L(d) = [d.0.s_L]`. The standing properties of these chains — T10a-conforming `inc(·, 0)` sibling-advance discipline; determinate first emission `[d.0.s_C.1]` (resp. `[d.0.s_L.1]`) with `origin = d`, `#E = 2`, `zeros = 3`, T4-valid and fresh against `dom(Σ.C) ∪ dom(Σ.L)` at the allocating state — are inherited from ASN-0093's sub-allocator lemmas.

Cross-subspace disjointness delta: `dom(A_C(d)) ∩ dom(A_L(d)) = ∅`, and for any d ≠ d', `dom(A_C(d)) ∩ dom(A_C(d')) = ∅`, `dom(A_L(d)) ∩ dom(A_L(d')) = ∅`, `dom(A_C(d)) ∩ dom(A_L(d')) = ∅`.

---

## S3★-aux — SubspaceExhaustiveness (INV, predicate)

In every reachable state, all V-positions have subspace s_C or s_L:

`(A d, v : v ∈ dom(M(d)) : subspace(v) = s_C ∨ subspace(v) = s_L)`

---

## CL-OWN — LinkSubspaceOwnership (INV, predicate)

In every reachable state:

`(A d, v : v ∈ dom(M(d)) ∧ subspace(v) = s_L : origin(M(d)(v)) = d)`

Every document's link-subspace arrangement contains only its own links.

---

## CL-UNIQ — LinkSubspacePositionUniqueness (INV, predicate)

Within each document's link-subspace arrangement, each link occupies exactly one V-position — the restriction of M(d) to dom_L is injective:

`(A d, v₁, v₂ : v₁ ∈ dom(M(d)) ∧ v₂ ∈ dom(M(d)) ∧ subspace(v₁) = s_L ∧ subspace(v₂) = s_L ∧ M(d)(v₁) = M(d)(v₂) : v₁ = v₂)`

---

## SequentialTransitionAxiom — SequentialTransitionAxiom (AX, axiom)

Transitions `Σ → Σ'` are atomic, uninterruptible, and totally ordered. The reflexive-transitive closure `Σ →* Σ'` denotes a finite (possibly empty) sequence of atomic transitions `Σ = Σ₀ → Σ₁ → ... → Σₙ = Σ'`.

---

## SubspaceConventionAxiom — SubspaceConventionAxiom (AX, axiom)

Fixed subspace identifiers: `s_C = 1 ∧ s_L = 2`. Consequence: `SC-NEQ` (`s_C ≠ s_L`, i.e., `1 ≠ 2`). The same identifiers serve V-positions via `subspace(v) = v₁` and element-level addresses via `subspace_I(a) = E(a)₁`.

---

## Endset — Endset (DEF, definition)

`Endset = 𝒫_fin(Span)` — a finite set of well-formed spans `(s, ℓ)` satisfying T12; the empty set ∅ is a valid endset.

---

## Link — Link (DEF, definition)

`Link = {(e₁, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`; `|L|` is the arity. StandardTriple convention (arity 3, `(F, G, Θ)`) is applied in worked examples only, not as a structural restriction.

---

## L-fin — LinkStoreFiniteness (INV, predicate)

LinkStoreFiniteness: `|dom(Σ.L)| < ∞`.

---

## L0 — SubspacePartition (INV, predicate)

SubspacePartition:

`(A a ∈ dom(Σ.L) :: subspace_I(a) = s_L)` and `(A a ∈ dom(Σ.C) :: subspace_I(a) = s_C)`

---

## L1 — LinkElementLevel (INV, predicate)

LinkElementLevel: `(A a ∈ dom(Σ.L) :: zeros(a) = 3)` — every link address is an element-level tumbler.

---

## L1a — LinkScopedAllocation (INV, predicate)

LinkScopedAllocation: `(A a ∈ dom(Σ.L) :: origin(a) ∈ E_doc)` — every link address is allocated under the tumbler prefix of a document.

---

## L3 — NEndsetStructure (INV, predicate)

NEndsetStructure:

`(A a ∈ dom(Σ.L) :: |Σ.L(a)| ≥ 3 ∧ (A i : 1 ≤ i ≤ |Σ.L(a)| : Σ.L(a).eᵢ ∈ Endset) ∧ Σ.L(a).e₃ ≠ ∅)`

Every link is a sequence of at least three endsets with the type endset (slot 3) non-empty.

---

## L12 — LinkImmutability (INV, predicate)

LinkImmutability:

`(A Σ → Σ' : (A a : a ∈ dom(Σ.L) : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)))`

Once created, a link's address persists in `dom(L)` and its value is permanently fixed.

---

## C-fin — ContentStoreFiniteness (INV, predicate)

ContentStoreFiniteness: `|dom(Σ.C)| < ∞`.

---

## L1c — LinkAllocatorConformance (INV, predicate)

LinkAllocatorConformance: every `ℓ ∈ dom(L)` has a structural inc-chain from its home document to `ℓ` — a finite sequence `(t₀, …, tₙ)` with `t₀ = origin(ℓ)`, `tₙ = ℓ`, each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfying T10a's per-step admissibility (T4-validity preservation, zero-count side condition at `kᵢ = 2`), `k₁ = 2`, and `#tᵢ > #origin(ℓ)` at every step.

---

## L14 — StoreDisjointness (INV, predicate)

StoreDisjointness: `dom(C) ∩ dom(L) = ∅` — unscoped store disjointness.

---

## M1 — ArrangementMonotonicity (INV, predicate)

ArrangementMonotonicity: `(A Σ → Σ' :: dom(M) ⊆ dom(M'))`. Constrains the *document set* `dom(M) = E_doc` (the allocated documents, which only grow via K.δ), **not** the per-document arrangement `dom(M(d))`.

---

## P0 — ContentPermanence (INV, predicate)

The content store admits only extensions, and existing entries are immutable:

`(A Σ → Σ' :: dom(C) ⊆ dom(C') ∧ (A a : a ∈ dom(C) : C'(a) = C(a)))`

Subsumes ASN-0036's S0 (ContentImmutability) and S1 (StoreMonotonicity).

---

## S3★ — GeneralizedReferentialIntegrity (INV, predicate)

The arrangement maps V-positions to addresses in the store appropriate to their subspace:

`(A d, v : v ∈ dom(Σ.M(d)) : (subspace(v) = s_C ⟹ Σ.M(d)(v) ∈ dom(Σ.C)) ∧ (subspace(v) = s_L ⟹ Σ.M(d)(v) ∈ dom(Σ.L)))`

where `subspace(v)` denotes the first component of the V-position. S3★ supersedes S3 (ASN-0036) for the extended state Σ = (C, L, E, M, R).

---

## D-CTG★ — PerSubspaceContiguity (INV, predicate)

`(A d, S : V_S(d) ≠ ∅ : V_S(d) is contiguous under T1 restricted to the depth-m_S, subspace-S slice)`, where the slice is the set of depth-m_S positive-component tuples whose first component is S (`m_S` fixed by S8-depth; "positive" denoting S8a-compatible domain, components in ℕ⁺), and T1 is LexicographicOrder.

*Contiguous* unpacks as closed-interval membership: for every `v_lo, v_hi ∈ V_S(d)` and every tuple `z` in the slice with `v_lo ≤ z ≤ v_hi` under T1, `z ∈ V_S(d)`.

---

## D-MIN★ — PerSubspaceMinimumPosition (INV, predicate)

`(A d, S : V_S(d) ≠ ∅ : min(V_S(d)) = [S, 1, ..., 1] of depth m_S)`, the minimum taken under T1 on the depth-m_S, subspace-S slice.

---

## D-SEQ★ — PerSubspaceSequentialPositions (INV, predicate)

For each non-empty subspace S in M(d):

`V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` for some `n_S ≥ 1`,

where the inner positions are of uniform depth m_S (the common depth within subspace S, by S8-depth), and `n_S = |V_S(d)|`. At the practical depth `m_S = 2` the inner "1, ..., 1" segment has length `m_S - 2 = 0`, so the canonical form degenerates to `{[S, k] : 1 ≤ k ≤ n_S}`.

Derived from D-CTG★ + D-MIN★ + S8-depth + S8-fin + S8a.

---

## P3 — ArrangementMutabilityOnly (INV, predicate)

No component other than M admits contraction or value rewriting:

`(A Σ → Σ' :: dom(C) ⊆ dom(C') ∧ dom(L) ⊆ dom(L') ∧ E ⊆ E' ∧ R ⊆ R' ∧ (A a ∈ dom(C) :: C'(a) = C(a)) ∧ (A ℓ ∈ dom(L) :: L'(ℓ) = L(ℓ)))`

The only component that can lose information is M. P3 is the synthesis of P0 ∧ P1 ∧ P2 ∧ L12.

---

## P4★ — ProvenanceBounds (PROP, predicate)

`Contains_C(Σ) ⊆ R`

P4★ bounds provenance by the *content-subspace* restriction of containment (scoped to the content subspace so it coexists with P7).

---

## J1★ — ExtensionRecordsProvenance (COUPLING, predicate)

`(A Σ →* Σ', d ∈ E'_doc, a : (E v ∈ dom(M'(d)) : subspace(v) = s_C ∧ M'(d)(v) = a) ∧ ¬(E v ∈ dom(M(d)) : subspace(v) = s_C ∧ M(d)(v) = a) : (a, d) ∈ R')`

J1★ is range-based: it triggers whenever an I-address `a` is new to the content-subspace range of M'(d), regardless of whether the V-position carrying it existed in dom(M(d)).

---

## J1'★ — ProvenanceRequiresExtension (COUPLING, predicate)

`(A Σ →* Σ', a, d : (a, d) ∈ R' \ R : (E v ∈ dom(M'(d)) : subspace(v) = s_C ∧ M'(d)(v) = a) ∧ ¬(E v ∈ dom(M(d)) : subspace(v) = s_C ∧ M(d)(v) = a))`

J1'★ is likewise range-based: every new provenance entry `(a, d) ∈ R' \ R` must correspond to an I-address `a` that is new to the content-subspace range.

---

## ValidComposite★ — ValidCompositeAmended (DEF, definition)

A composite transition `Σ →* Σ'` in the extended state Σ = (C, L, E, M, R) is *valid* iff it is a finite sequence of atomic transitions `Σ = Σ₀ → Σ₁ → ... → Σₙ = Σ'` — drawn from the atomic vocabulary K.α (amended), K.δ, K.λ, K.μ⁺ (amended), K.μ⁺_L, K.μ⁻ (amended), and K.ρ — satisfying:

1. *Transition preconditions (intra-composite sequencing).* Each step `Σᵢ → Σᵢ₊₁` satisfies the *elementary* precondition of its transition kind, evaluated at the *intermediate* state `Σᵢ`. K.μ~ appearing in the sequence is shorthand for its K.μ⁻ + K.μ⁺ decomposition: admissibility clause (ii) requires a non-trivial net effect `M'(d) ≠ M(d)`, whose necessary-and-sufficient existence condition is the K.μ~ precondition (`M(d)|_{dom_C}` takes at least two distinct values).
2. *Coupling constraints (initial-to-final).* J0, J1★, and J1'★ hold for the composite as a whole — evaluated *only* between the initial state Σ and the final state Σ'. A composite that satisfies clause (1) but violates clause (2) is not a valid composite.

---

## S8★ — PerSubspaceSpanDecomposition (INV, predicate)

For each `d ∈ E_doc` and each subspace `S ∈ {s_C, s_L}`, the per-subspace arrangement `M(d)|_{V_S(d)}` decomposes into a finite set of correspondence runs `{(v_j, a_j, n_j)}`: every `v ∈ V_S(d)` lies in exactly one run, and within each run the V-positions and I-addresses advance by shift in lockstep. S8★ retains ASN-0036's S8 conditions (a) (lockstep displacement) and (b) (label well-definedness); condition (c) (uniqueness of the maximal-run decomposition) is retained only on the content subspace.

- *Content subspace:* `M(d)|_{V_{s_C}(d)}` by direct application of ASN-0036's S8.
- *Link subspace:* `M(d)|_{V_{s_L}(d)}` by the trivial length-1 decomposition `{(v, M(d)(v), 1) : v ∈ V_{s_L}(d)}` — every link-subspace V-position constitutes its own length-1 run.

---

## ExtendedReachableStateInvariants — ExtendedReachableStateInvariants (THM, theorem)

Every state reachable from Σ₀ by a finite sequence of *elementary* transitions drawn from valid composites satisfies the *per-state invariants*:

S2 ∧ S3★ ∧ S3★-aux ∧ S4 ∧ S7a ∧ S7b ∧ C1b ∧ C1c ∧ S7d ∧ S8a ∧ S8-fin ∧ S8-depth ∧ S8★ ∧ C-fin ∧ D-CTG★ ∧ D-MIN★ ∧ D-SEQ★ ∧ P6 ∧ P7 ∧ P8 ∧ NodeLineage ∧ ActivatedEmission ∧ L0 ∧ L1 ∧ L1a ∧ L1b ∧ L1c ∧ L3 ∧ L14 ∧ L-fin ∧ CL-OWN ∧ CL-UNIQ

Every state at a composite boundary additionally satisfies the *composite-boundary properties*:

P4★ ∧ P4a ∧ P7a

---

## ExtendedTransitionInvariants — ExtendedTransitionInvariants (THM, theorem)

Every valid composite transition `Σ →* Σ'` satisfies:

P3

---

## K.α's `E(a)₁ = s_C` precondition — ContentAllocationSubspacePrecondition (PRE, requires)

K.α's `E(a)₁ = s_C` precondition (inherited from ASN-0093's K.α) pins `subspace_I(a) = s_C` for every allocated content address; every `a ∈ dom(C)` has `subspace_I(a) = s_C`.

---

## K.μ⁺ amendment — ContentSubspaceRestriction (DEF, definition)

K.μ⁺ is amended with a content-subspace restriction: new V-positions must satisfy `subspace(v) = s_C`. This complements K.μ⁺_L, which handles link-subspace extensions exclusively. The restriction `subspace(v) = s_C` confines every K.μ⁺-added V-position to the content subspace, so its image lies in dom(C) and S3★ is discharged. With this amendment, the two transitions partition arrangement extensions by subspace.

The amended K.μ⁺ precondition requires `M'(d)` to satisfy D-CTG★ and D-MIN★ restricted to the content subspace `S = s_C` — i.e., `V_{s_C}(d)` is contiguous with `min(V_{s_C}(d)) = [s_C, 1, ..., 1]` when non-empty.

*Frame (extended state):* `C' = C; L' = L; E' = E; (A d' : d' ≠ d : M'(d') = M(d')); R' = R`.

---

## K.μ⁻ (per-subspace scope) — PerSubspaceContractionScope (DEF, definition)

The extended state adds the link subspace, so two changes apply over the elementary definition:
1. The link-store frame clause `L' = L` is added.
2. The elementary single-content-subspace retention count `n'` generalizes to a *per-subspace* retention count: under D-SEQ★ at Σ each non-empty `V_S(d)` has canonical shape `{[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` for `S ∈ {s_C, s_L}`, and the caller selects, for each S, a retention count `n'_S ∈ {0, 1, ..., n_S}` (with `n'_S = 0` when `V_S(d) = ∅`), subject to at least one S admitting strict contraction `n'_S < n_S`. The contracted arrangement is the restriction `M'(d) = M(d) ↾ R` to `R := ∪_{S ∈ {s_C, s_L}} {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}`.

*Frame (extended state):* `C' = C; L' = L; E' = E; R' = R; (A d' : d' ≠ d : M'(d') = M(d'))`.
