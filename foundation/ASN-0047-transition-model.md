> **ASN-0047 · Transition Model** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md), [ASN-0043 · Link Model](ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](ASN-0045-tumbler-fields.md), [ASN-0093 · Allocation Substrate](ASN-0093-allocation-substrate.md)  
> [Condensed statements →](ASN-0047-transition-model.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0047: Transition Model

*2026-03-17, revised 2026-03-22*

ASN-0036 established two components of system state — a permanent content store C and mutable document arrangements M(d) — and proved content, once stored, is immutable (S0): arrangement mutation never reaches the stored content. These are properties of the invariants. We have not yet classified the transitions. In what primitive ways can the state change, and what must each change preserve?

The consultation answers reveal a state model richer than the two-space analysis captured. Nelson enumerates the ways the docuverse changes — new documents created, new content inserted, new links established, views rearranged — and is equally precise about what cannot happen: content is never destroyed, addresses are never reassigned, history is never erased. Gregory reduces eight protocol commands to six kinds of persistent modification, distributed across three storage layers with distinct permanence contracts.

We seek the abstract taxonomy. Not the protocol commands, which are interface design, but the primitive modifications and their invariants. The central result is a *mutability hierarchy*: the state components arrange into three temporal layers, each with its own permanence contract. Destructive change — removal and reordering — is confined entirely to the most mutable layer.


## Notation

This ASN draws on projection functions and predicates established in the foundation ASNs (ASN-0034 Tumbler Algebra, ASN-0036 Strand Model, ASN-0043 Link Model, ASN-0045 Tumbler Fields). For reader convenience we list them here, fixing one notation per concept and pointing to the defining ASN. Notation introduced for the first time in this ASN is marked "introduced here."

*I-address (element-level) projections.* For each `a ∈ T_elem`:

- `E(a)` (ASN-0034, T4b): the element-field projection — the sequence of components after the last zero separator. `E(a)` is itself a positive-component tumbler with `#E(a) ≥ 1`.
- `subspace_I(a)` (ASN-0043, Definition — SubspaceI): the I-address subspace identifier, equal to `E(a)₁` — distinct from ASN-0036's V-position projection `subspace(v) = v₁`.
- `origin(a)` (ASN-0036, S7 — definition `origin(a) = N(a).0.U(a).0.D(a)`; S7a — allocation grounding): the document address `d ∈ E_doc` under whose allocator a was minted. For each `a ∈ dom(C)`, `origin(a) ∈ E_doc`; for each `ℓ ∈ dom(L)`, `origin(ℓ) ∈ E_doc` (L1a). origin is recovered by truncating a to the document prefix (zeros = 2).
- `Element(a)`, `Node(a)`, `Account(a)`, `Document(a)` (ASN-0045): the level predicates, holding when `zeros(a)` is 3, 0, 1, 2 respectively. Address validity is `T4-valid(t)` (ASN-0045).
- `#E(a)` (ASN-0034): the depth (component count) of the element-field projection `E(a)` supplied by T4b — i.e. `#a` minus the length of the node/user/document prefix together with its three zero separators.

*V-position (arrangement-domain) projections.* For each `v ∈ dom(M(d))`:

- `subspace(v)` (ASN-0036): the first component `v₁` of the V-position tumbler — the subspace identifier at the V-position level. By S8a, every `v ∈ dom(M(d))` satisfies `v₁ ≥ 1`, so `subspace(v) ∈ ℕ⁺`. In this ASN the two subspaces are `s_C` (content/text) and `s_L` (link), with `s_C ≠ s_L` (SC-NEQ, a consequence of ASN-0093's `FixedSubspaceIdentifiers` axiom).
- `#v` (ASN-0034): the depth of v. By S8-depth, V-positions within a fixed subspace under a fixed document share a common depth `m_S`.

*Entity-hierarchy projections.* For each non-node entity `e ∈ E`:

- `parent(e)` (introduced here, §The state model): the tumbler obtained by truncating e's last field together with its preceding zero separator. `parent(e)` is the entity-hierarchy spine — defined only for non-node entities (`¬Node(e)`), and producing a valid address at the next-higher level: `zeros(parent(e)) = zeros(e) − 1`.

*Content/link domain notation.* `dom_C(M(d)) := V_{s_C}(d) := {v ∈ dom(M(d)) : subspace(v) = s_C}`; symmetrically `dom_L(M(d)) := V_{s_L}(d) := {v ∈ dom(M(d)) : subspace(v) = s_L}`. The `V_S(d)` form generalises to any subspace S; the two spellings are denotationally identical.

*Entity-level allocator.* An *activated* sub-allocator (ASN-0034's AllocatedSet) whose output addresses satisfy `zeros(·) ≤ 2`: node, account, document, and version sub-allocators. We use ASN-0034's term *activated* directly: a sub-allocator is activated once an allocation event has activated it, and its realized domain `domₛ(A)` — the finite prefix of `dom(A)` enumerated at state `s` — grows monotonically across transitions, `domₛ(A) ⊆ dom_{s'}(A)`, under NoDeallocation (ASN-0034).

*Set-inclusion notation.* Throughout this ASN, `⊂` and `⊃` denote *proper* (strict) subset and superset respectively — `A ⊂ B ≡ A ⊆ B ∧ A ≠ B`, and `A ⊃ B ≡ B ⊂ A`. The non-strict relations are written `⊆` and `⊇`.


## The state model

ASN-0036 gave us C and M(d). Two phenomena require additional state components.

First, entities come into existence. Nelson describes exactly two document creation modes: ex nihilo (a fresh empty document) and forking (a new document derived from an existing one). Gregory confirms both use the same allocation mechanism, differing only in whether the new arrangement starts empty or populated. We need an explicit record of which entities exist.

**Definition (Entity set).** **Σ.E ⊆ T** — the set of allocated entity addresses. Every e ∈ E satisfies T4-valid(e) (T4, ASN-0034). Entities are organisational — nodes, accounts, documents — not content; element-level addresses live in dom(C), not E:

`(A e ∈ E :: ¬Element(e))`

Equivalently, E ⊆ {t : T4-valid(t) ∧ zeros(t) ≤ 2}.

*Consequence (Stratification).* By T4c (ASN-0045) and the exclusion clause above, Σ.E partitions into exactly three strata:

- E_node = {e ∈ E : Node(e)} — server nodes
- E_account = {e ∈ E : Account(e)} — user accounts
- E_doc = {e ∈ E : Document(e)} — documents (zeros = 2)

For a non-node entity e (where ¬Node(e)), define **parent(e)** using T4b's (UniqueParse, ASN-0034) partial projections N, U, D, E:

- *Account case (Account(e)).* `parent(e) = N(e)` — the node-prefix projection. Since `Account(e)` requires `zeros(e) = 1`, T4b's parse `e = N(e).0.U(e)` is defined with `zeros(N(e)) = 0`, giving `zeros(parent(e)) = 0 = zeros(e) − 1`.
- *Document case (Document(e)).* `parent(e) = N(e).0.U(e)` — the account-prefix projection. Since `Document(e)` requires `zeros(e) = 2`, T4b's parse `e = N(e).0.U(e).0.D(e)` is defined with `zeros(N(e).0.U(e)) = 1`, giving `zeros(parent(e)) = 1 = zeros(e) − 1`.

In each case parent(e) is a valid address at the next higher level: `zeros(parent(e)) = zeros(e) − 1` is a derivable property of T4b's projections, not a stipulation. The two cases together define parent uniformly on non-node entities (the Node case is excluded by the precondition `¬Node(e)`, since nodes have no parent in the entity-hierarchy spine).

Non-empty arrangements arise only for document entities. Links are owned by documents (`origin(ℓ) ∈ E_doc`, by L1a) but inhabit a separate state component L, not E_doc: L1 (ASN-0043) requires `zeros(ℓ) = 3` for every link address, and Document (ASN-0045) requires `zeros(t) = 2`, so `Document(ℓ)` is false and `ℓ ∉ E`. Nelson describes links as owned entities with internal structure ("a package of connecting or marking information... owned by a user... thereafter maintained by the back end"); the link store L gives them their own first-class state component, distinct from the entity set E.

Second, removal of content from an arrangement does not erase the historical fact of prior containment. Gregory: the reverse index "accumulates entries from every content addition but is never trimmed." Nelson: "every previous arrangement remains reconstructable." The system must answer "which documents have ever contained content with origin *a*?" — a question about history, not about current state.

**Definition (Provenance relation).** **Σ.R ⊆ T_elem × E_doc** — where T_elem = {a ∈ T : Element(a)} (ASN-0045). The pair (a, d) ∈ R records that document d contained I-address a in its arrangement at the composite boundary at which the entry was made.

The full system state is:

> **Σ = (C, L, E, M, R)**

where C : T ⇀ Val is the content store (per ASN-0036), L : T ⇀ Link is the link store, and M is the arrangement family carrying the foundation's typing verbatim — `M(d) : T ⇀ T` is *partial* (ASN-0036, ASN-0093), with `dom(M)` the set of allocated documents exactly as in the foundations.

**Bridging lemma (M–E_doc).** The document-set role is carried here by `E_doc`, while the foundations phrase document allocation through `dom(M)`. The two coincide:

  `dom(M) = E_doc`   (†)

We prove (†) by induction on the transition sequence. *Base:* the initial state has `(E₀)_doc = ∅ = dom(M₀)`, so the two sets coincide. *Step:* assume `dom(M) = E_doc` at the pre-state and consider each transition. The K.δ `Document(e)` case grows both sets by the same `{e}` — it registers `e` into `E_doc` and sets `dom(M') = dom(M) ∪ {e}` with `M'(e) = ∅` — preserving the equality. The arrangement-mutating transitions K.μ⁺, K.μ⁺_L, and K.μ⁻ act on the partial function `M(d)` at a *fixed* `d ∈ E_doc` (each requires `d ∈ E_doc` as a precondition); they alter the V→I content of `M(d)` but neither add nor remove a key of `M`, so the document set `dom(M)` is framed and `E_doc` is likewise untouched. Every remaining transition (K.δ at `Node(e)`/`Account(e)`, K.α, K.λ, K.ρ) frames `M` entirely. In all cases both sets receive the same change (or none), so `dom(M) = E_doc` is maintained at the post-state. ∎ Inherited `dom(M)`-phrased foundation results therefore apply under (†), with `E_doc` naming the document set.

*Notational convention (default value).* For a tumbler `d ∉ dom(M)`, the expression `M(d)` abbreviates the empty partial function `∅`, keeping range expressions such as `ran(M(d))` defined on all of `T` (yielding `∅` off `E_doc`). `M(d) = ∅` does not signal allocation status — a freshly registered document also has `M(d) = ∅` — so `E_doc`-membership, not the test `M(d) = ∅`, is the discriminating predicate for allocation.

The sole exception is ASN-0093 M2 (EmptyArrangement), `(A d ∈ dom(M) :: M(d) = ∅)`, which is *not* inherited: ASN-0047 populates arrangements via K.μ⁺/K.μ⁺_L. M2 holds at registration (the K.δ `Document(e)` effect sets `M'(e) = ∅`) but is superseded by K.μ⁺/K.μ⁺_L.

**Definition (Initial state).** The initial state Σ₀ = (C₀, L₀, E₀, M₀, R₀) is:

- C₀ = ∅ (no content allocated)
- L₀ = ∅ (no links allocated)
- E₀ = {n₀} where n₀ = `[1]` — the canonical single-component bootstrap node
- M₀(d) = ∅ for all d — (E₀)_doc = ∅, so every arrangement is the empty partial function
- R₀ = ∅ (no provenance recorded)

**Structural form of n₀.** The bootstrap node is fixed as `[1]` — a one-element tumbler with `zeros(n₀) = 0`, satisfying `Node(n₀)` and `T4-valid(n₀)`. The NodeLineage invariant (`n₀ ≼ e`) constrains every node address to extend `[1]` by prefix, ruling out disconnected-forest allocations.

**Initial state invariant verification.** Each Class (a) per-state invariant of ExtendedReachableStateInvariants holds at Σ₀, most vacuously. We enumerate the verifications to make the base case of the inductive proof explicit:

- *Entity invariants.* `E₀ = {n₀}` with `Node(n₀)`, `T4-valid(n₀)` (`[1]` is T4-valid), and `zeros(n₀) = 0` (no separators). The exclusion `(A e ∈ E :: ¬Element(e))` holds since `zeros(n₀) = 0 ≠ 3`.
- *NodeLineage* `(A e ∈ E₀ : Node(e) : n₀ ≼ e)`: instantiates at `e = n₀`, requiring `n₀ ≼ n₀`, which holds by reflexivity of the tumbler-prefix order.
- *P8 (Entity hierarchy)* `(A e ∈ E₀ : ¬Node(e) : parent(e) ∈ E₀)`: vacuously satisfied — `n₀` is the only entity in `E₀` and `Node(n₀)`, so the quantifier scope is empty.
- *ActivatedEmission (Entity emission tracking)* `(A e ∈ E₀ : ¬Node(e) : (E A : Activated(A) ∧ EntityLevel(A) : e ∈ dom(A)))`: vacuously satisfied — `E₀ = {n₀}` with `Node(n₀)`, so the `¬Node(e)` quantifier scope is empty.
- *S7d (Document allocation discipline)*: `(E₀)_doc = ∅`, vacuous.
- *S2, S3★, S3★-aux, S8a, S8-depth, S8-fin, S8★, D-CTG★, D-MIN★, D-SEQ★*: `M₀(d) = ∅` for all `d`, so `dom(M₀(d)) = ∅` and `V_S(d) = ∅` for every subspace `S`. Each invariant holds vacuously over the empty arrangement domain.
- *S4, S7a, S7b, C1b, C1c (content invariants)*: `dom(C₀) = ∅`, vacuous.
- *C-fin (Content store finiteness)*: `|dom(C₀)| = |∅| = 0 < ∞`. ✓
- *P6 (Existential coherence)*: `dom(C₀) = ∅`, vacuous.
- *L0, L1, L1a, L1b, L1c, L3, L14, L-fin*: `dom(L₀) = ∅`, vacuous. L-fin: `|∅| = 0 < ∞`. L14: `dom(C₀) ∩ dom(L₀) = ∅ ∩ ∅ = ∅`.
- *CL-OWN, CL-UNIQ*: no link-subspace V-positions exist in `M₀` (the arrangements are empty), so both quantifiers have empty scope.
- *P7 (Provenance grounding)*: `R₀ = ∅`, vacuous.

The Class (b) composite-boundary properties at Σ₀:

- *P4★* `Contains_C(Σ₀) ⊆ R₀`: `Contains_C(Σ₀) = ∅ ⊆ ∅ = R₀`. ✓
- *P4a (Trace witnessing)*: `R₀ = ∅`, vacuous.
- *P7a (Provenance coverage)*: `dom(C₀) = ∅`, vacuous.

This closes the inductive base for ExtendedReachableStateInvariants; the inductive step is the per-elementary verification in the proof body.

**SequentialTransitionAxiom.** We rely on ASN-0093's SequentialAtomicTransitions axiom: transitions `Σ → Σ'` are atomic, uninterruptible, and totally ordered.

**Notation.** Throughout this ASN, `Σ → Σ'` denotes a single atomic transition (one elementary K.* event). The reflexive-transitive closure `Σ →* Σ'` denotes a finite (possibly empty) sequence of atomic transitions `Σ = Σ₀ → Σ₁ → ... → Σₙ = Σ'`; the valid such sequences are characterised by ValidComposite★ below. Per-transition predicates stated over `Σ → Σ'` (P0, P1, P2, L12, P3) hold at every atomic step and lift to every composite by transitivity of inclusion and equality. Coupling constraints (J0, J1★, J1'★) are stated over composites `Σ →* Σ'`; they bind initial and final states across a valid composite, not individual atomic steps.


## Permanence

We classify each component of the five-component state `Σ = (C, L, E, M, R)` by the transitions it admits. The named per-component predicates below — P0, L12, P1/P8, P2 — fix what each component permits; the resulting partition into permanence contracts is collected once, with its cross-layer bridges, in the *Temporal decomposition* table at the end of this note.

**P0 (Content permanence).** The content store admits only extensions, and existing entries are immutable:

`(A Σ → Σ' :: dom(C) ⊆ dom(C') ∧ (A a : a ∈ dom(C) : C'(a) = C(a)))`

This is S0 of ASN-0036, restated for the full state model. C is *append-only with immutable values*. Nelson: "Instead, suppose we create an append-only storage system." Gregory confirms: no deletion or update operation exists for the content store.

**L12 (Link permanence).** The link store admits only extensions, and existing entries are immutable:

`(A Σ → Σ' :: dom(L) ⊆ dom(L') ∧ (A a : a ∈ dom(L) : L'(a) = L(a)))`

This is L12 of ASN-0043 (LinkImmutability), restated for the full state model. L shares C's contract — *append-only with immutable values*: once a link's address is allocated, the address persists and the triple of endsets bound to it never changes. No transition deletes or rewrites a link.

**P1 (Entity permanence).** The entity set admits only extensions:

`(A Σ → Σ' :: E ⊆ E')`

No transition removes an entity. This specialises T8 (AllocationPermanence, ASN-0034) to the entity set. P1 holds uniformly across levels:

`[e ∈ E ∧ Node(e) ⟹ e ∈ E']`
`[e ∈ E ∧ Account(e) ⟹ e ∈ E']`
`[e ∈ E ∧ Document(e) ⟹ e ∈ E']`

Nelson: "New items may be continually inserted in tumbler-space while the other addresses remain valid." The address space is a growing tree; entities are born but never die.

**P8 (Entity hierarchy).** Every non-node entity has its parent in E:

`(A e ∈ E : ¬Node(e) : parent(e) ∈ E)`

This ensures the entity set is hierarchically well-formed: every account has its node in E, every document has its account in E. Combined with P1, the hierarchy only grows — once an entity's parent chain is established, it persists.

*Derivation.* K.δ for non-root entities requires parent(e) ∈ E as a precondition (below). P1 preserves the parent's membership across subsequent transitions. Base case: E₀ = {n₀} with Node(n₀), so the quantifier is vacuously satisfied. Inductive step: K.δ adds e with parent(e) ∈ E ⊆ E' (by precondition and P1); all other transitions have E' ⊇ E, preserving existing parent relationships. ∎

**P2 (Provenance permanence).** The provenance relation admits only extensions:

`(A Σ → Σ' :: R ⊆ R')`

Once the system records that d referenced a, that record persists. Gregory: the provenance structure is "a permanently-growing reverse index that accumulates entries from every content addition but is never trimmed."

Arrangements admit three modes of change — extension (new V→I mappings added), contraction (existing V→I mappings removed), and reordering (V-positions of existing mappings change while the multiset of referenced I-addresses is preserved); no other component admits contraction or reordering. Gregory: the arrangement layer is "the sole locus of destructive mutation."


## Elementary transitions

We seek the elementary modifications — the state changes from which all system operations compose. Each is defined by its effect and its frame: what changes and what does not.

**Convention.** A transition with unsatisfied preconditions does not fire.

**Frame convention for inherited transitions.** Inherited K.* transitions extend ASN-0093's frame with `E' = E ∧ R' = R`, since ASN-0093 has no E or R components.

**K.α (Content allocation).** ASN-0093's K.α (ContentAllocation), with frame extended by `E' = E ∧ R' = R` (Frame convention for inherited transitions). Freshness `a ∉ dom(C) ∪ dom(L)` is SubAllocFresh at `x = C` (Lemma SubAllocatorFreshness, *Allocator hierarchy under documents*, below).

*Effect:* `C' = C ∪ {a ↦ v}`.

*Frame:* `L' = L; (A d :: M'(d) = M(d))` (E and R held in frame per the opening clause).

**NodeLineage (Derived invariant, NodeDescentFromBootstrap).** `(A e ∈ E : Node(e) : n₀ ≼ e)`, where `≼` is the prefix order on tumblers (ASN-0034).

**NodeBaptism (Axiom, boundary input — node provisioning).** No docuverse transition mints a node address. At every K.δ node-allocation event — every elementary K.δ transition placing an entity `e` with `Node(e)` into E:

- (a) *Freshness:* `e ∉ Σ.E` at the state Σ of allocation;
- (b) *Bootstrap lineage:* `n₀ ≼ e` under the tumbler-prefix order.

The bootstrap node `n₀ ∈ E₀` is itself baptised at `Σ₀`.

**NodeRootedForest (Derived structure).** Nodes enter E only via NodeBaptism, never as `inc`-outputs, so the `inc`-allocator structure is a *forest*: each baptised node `N` roots an independent allocator subtree whose members are the `inc`-descendants of `N`. Within one subtree `N` is the sole root and T10a's discipline holds with `N` as base address, so ASN-0034's GlobalUniqueness applies with `N` discharging its strong-induction base case.

*Subtree-scoped GlobalUniqueness (SSGU).* For any `inc`-output `a` with `N ≼ a` for a baptised node `N`, GlobalUniqueness scoped to `N`'s subtree (ASN-0034, via T10a.6 domain-disjointness, `N` at the base case) assigns `a` to exactly one allocation event within that subtree; cross-node distinctness excludes every event under a distinct baptised node `N' ≠ N`, by a case-split on prefix-comparability of the two nodes. *Incomparable nodes* (`N ⋠ N' ∧ N' ⋠ N`): the non-nesting precondition of T10 (PartitionIndependence, ASN-0034) is met at `(N, N')`, so every address extending `N` differs from every address extending `N'`, and in particular `a ≠ a'` for any `inc`-output `a'` under `N'`. *Nested nodes* (WLOG `N ≼ N'` with `#N < #N'`, the configuration NodeBaptism permits since multi-component node tumblers are T4-legal): here the prefix cones overlap — any `inc`-output `a'` under `N'` satisfies `N ≼ N' ≼ a'` — so T10's non-nesting precondition fails and T10 does not apply. Distinctness instead follows by the zero-separator divergence: any non-node `inc`-output `a` assigned to `N`'s subtree carries the field-separating zero introduced by its first descent off the node `N` at position `#N + 1`, so `a_{#N + 1} = 0`; whereas `a'` with `N' ≼ a'` agrees with `N'` at position `#N + 1`, where `(N')_{#N + 1} ≠ 0` because `N'` is a node (`zeros(N') = 0`, every component nonzero). The two diverge at position `#N + 1`, so `a ≠ a'`. Because this divergence falls at position `#N + 1 ≤ #a'`, a position present in both addresses, it yields not merely distinctness but prefix-incomparability whenever both operands carry an actual component there.

**ActivatedEmission (Per-state invariant, EntityEmissionTracking).** Every non-node entity is an emission of a activated entity-level sub-allocator:

  `(A e ∈ Σ.E : ¬Node(e) : (E A : Activated(A) ∧ EntityLevel(A) : e ∈ dom(A)))`

*Preservation.* At Σ₀ the property holds vacuously — `E₀ = {n₀}` with `Node(n₀)`, so the quantifier scope is empty. K.δ admits a non-node entity `e` into E only via an `inc`-step `e = inc(t, k)` that is a T10a transition on a activated entity-level sub-allocator (established by K.δ case (ii)), so `e` enters that allocator's realized domain at the very event that places it into E. P1 preserves both `e ∈ E` and the allocator's activated status across all subsequent transitions; every non-K.δ transition holds E in frame.

**FrontierEquivalence (Lemma).** For every reachable state `Σ` and every operand `t ∈ Σ.E` with `¬Node(t)`, ActivatedEmission supplies a activated entity-level sub-allocator whose domain contains `t` — establishing existence — and `A` denotes that allocator, unique by T10a.6 (DomainDisjointness, ASN-0034). Then:

  `inc(t, 0) ∉ Σ.E ⟺ t is the frontier of A's (t, 0)-branch`

— i.e., the operational predicate "the `(t, 0)` increment has not yet been consumed" is logically equivalent to "no prior K.δ event has fired `(t, 0)` on `A`'s chain" (the next sibling-increment of `t` on `A`). The term "frontier" is well-defined because, by T10a.7 (EnumerationInjectivity, ASN-0034), the chain map `n ↦ tₙ` is injective, so each chain index names a distinct address.

*Proof.* The biconditional decomposes into two implications, each proved separately.

*Forward direction (⟹).* Assume `inc(t, 0) ∉ Σ.E`. We show `t` is the frontier of `A`'s `(t, 0)`-branch — i.e., no prior K.δ event has fired `(t, 0)` on `A`'s chain. Argue by contrapositive: suppose for contradiction that some prior K.δ event *had* fired `(t, 0)`, producing the address `inc(t, 0)` and placing it into `E` at that earlier state. By TA5(c), `inc(t, 0)` is a single determinate address (functional determinism — the same operand-and-parameter pair always produces the same output). By P1 (E-monotonicity), any address once placed into E persists across every subsequent transition. So `inc(t, 0)` is in E at the present state Σ — contradicting the assumed `inc(t, 0) ∉ Σ.E`. Hence no prior firing of `(t, 0)` occurred on `A`'s chain; t is the frontier.

*Reverse direction (⟸).* Assume `t` is the frontier of `A`'s `(t, 0)`-branch — no prior K.δ event has fired `(t, 0)` on `A`'s chain. We show `inc(t, 0) ∉ Σ.E`. Suppose for contradiction `inc(t, 0) ∈ Σ.E`. Then some allocation event in the system history placed `inc(t, 0)` into E. E is populated by two disjoint mechanisms — NodeBaptism (which places only node addresses) and T10a `inc`-events; we first exclude the former. By TA5(c), `zeros(inc(t, 0)) = zeros(t)`, and since `¬Node(t)` gives `zeros(t) ≥ 1`, we have `zeros(inc(t, 0)) ≥ 1`, so `¬Node(inc(t, 0))` — `inc(t, 0)` is non-node and therefore cannot have been a NodeBaptism output. Hence the placing event was a T10a `inc`-event. The address `inc(t, 0)` is, by TA5(c), the next sibling-increment of `t`, hence a member of `t`'s own node-rooted subtree with `N ≼ t`. By SSGU (NodeRootedForest), `inc(t, 0)` is produced by exactly one allocation event within that subtree, and T10a.6 forbids any realized domain other than `A` from containing it — so the producing allocator is `A`, the allocator whose realized domain contains `t`. So the placing event was a K.δ event firing `(t, 0)` on `A`'s chain — contradicting the frontier assumption. Hence `inc(t, 0) ∉ Σ.E`.

Together, the two implications yield the biconditional `inc(t, 0) ∉ Σ.E ⟺ t is the frontier of A's (t, 0)-branch`. ∎

**ChildSpawnFreshness (Lemma).** For every reachable state `Σ`, every operand `t ∈ Σ.E`, and every `k' ∈ {1, 2}` admissible at `t`:

  `inc(t, k') ∉ Σ.E ⟺ the (t, k') child-spawn has not yet been performed`

Note `inc(t, k')` is non-node: for `k' = 2`, K.δ-ID.zeros-2 gives `zeros(inc(t, 2)) = zeros(t) + 1 ≥ 1`; for `k' = 1`, admissibility requires `Document(t)` (`zeros(t) = 2`), so K.δ-ID.zeros-0/1 gives `zeros(inc(t, 1)) = 2`. The preconditions impose no `¬Node(t)` constraint on the operand: a `k' = 2` descent may be spawned off a node operand.

*Proof.* *Forward (⟹), by contrapositive.* Suppose the `(t, k')` child-spawn had already fired. By TA5(d), `inc(t, k')` is a single determinate address, placed into `E` at that earlier event; by P1 (E-monotonicity) it persists, so `inc(t, k') ∈ Σ.E`, contradicting `inc(t, k') ∉ Σ.E`. *Reverse (⟸).* Suppose `inc(t, k') ∈ Σ.E`. Since `inc(t, k')` is non-node (above), NodeBaptism — which places only node addresses — is excluded, so the placing event was a T10a `inc`-event. Since `k' ≥ 1`, TA5(b) gives agreement of `inc(t, k')` with `t` on positions `1..#t`, so `t ≼ inc(t, k')`; the address therefore lies in `t`'s own node-rooted subtree, rooted at the baptised node `N` with `N ≼ t`. By SSGU (NodeRootedForest), the determinate address `inc(t, k')` is produced by exactly one allocation event within that subtree — the `(t, k')` child-spawn that seeds the spawned child allocator's base. Hence that spawn was performed, contradicting the frontier-freshness assumption. ∎

**K.δ (Entity creation).** A fresh entity address enters E with initial state:

`E' = E ∪ {e}` where `e ∉ E ∧ T4-valid(e) ∧ ¬Element(e)`

*Precondition.* The precondition splits on `Node(e)`, reflecting two distinct allocation disciplines — protocol-established node baptism versus T10a-conforming inc-allocation under a parent entity.

- **Case (i) Node(e).** No operand `t` is consumed (`e` is supplied by the node-provisioning boundary, not by inc). Required: `T4-valid(e) ∧ Node(e)`, together with the freshness and bootstrap-lineage conjuncts supplied directly by NodeBaptism (a)/(b) — the boundary axiom — outside T10a's standard discharge layer.
- **Case (ii) ¬Node(e).** `e = inc(t, k)` for some operand `t` and `k ∈ {0, 1, 2}`. The case-level "where"-clause conjuncts `e ∉ E ∧ T4-valid(e) ∧ ¬Element(e)` apply uniformly to all three sub-cases. The freshness conjunct `e ∉ E` is discharged as a single live-state read against Σ in every sub-case — the sub-cases differ solely in *which* state fact the guard encodes, recorded per sub-case below; cross-event distinctness by GlobalUniqueness (ASN-0034). Required uniformly: `parent(e) ∈ E`. Per-sub-case additional requirements:
  - *k = 0 (sibling):* `t ∈ E ∧ ¬Node(t)`. These are the only sub-case-specific operand-admissibility conjuncts: `t ∈ E` (the operand must be an allocated entity) and `¬Node(t)` (so `parent(t)` is well-defined under T4b). No additional freshness conjunct is imposed here — the case-level `e ∉ E` (with `e = inc(t, 0)`) reads the current frontier index: by FrontierEquivalence's biconditional, `inc(t, 0) ∉ Σ.E` iff `t` is the frontier of `A`'s chain. The structural identities `parent(t) = parent(e)` and `zeros(t) = zeros(e)` hold by TA5(c) on `e = inc(t, 0)` (K.δ-ID.parent-0/1, K.δ-ID.zeros-0/1).
  - *k = 1 (version):* `t ∈ E_doc`. The operand must be an allocated document — only an existing document can be versioned. Nelson's CREATENEWVERSION operates on `<doc id>`, an allocated document (LM 4/66); Gregory's `docreatenewversion` retrieves the source's vspan via `doretrievedocvspanfoo`, which fails on a source not present in the granfilade. (`Document(t)` follows from `t ∈ E_doc` by the definition of E_doc.) Spawn admissibility: `k' = 1 ∈ {1, 2}`, and T10a's zero-count side condition `zeros(spawnPt) ≤ 3` holds a fortiori since `t` is document-level (`zeros(t) = 2`). The case-level `e ∉ E` (with `e = inc(t, 1)`) is the enforcement of T10a's at-most-once-per-`(t, 1)` discipline, discharged by ChildSpawnFreshness at `k' = 1`.
  - *k = 2 (descent):* `t ∈ E ∧ zeros(t) ≤ 1` (equivalently, `Node(t) ∨ Account(t)`). The zeros bound follows from the case-level precondition `¬Element(e)` (`zeros(e) ≤ 2`) combined with the structural identity `zeros(e) = zeros(t) + 1` (K.δ-ID.zeros-2). The structural identity `parent(e) = t` holds by TA5(d) on `e = inc(t, 2)` together with T4b's parent projection (K.δ-ID.parent-2). Spawn admissibility: `k' = 2 ∈ {1, 2}`, and the bound `zeros(t) ≤ 1` discharges T10a's zero-count side condition for `k' = 2` (`zeros(spawnPt) ≤ 2`) a fortiori. The case-level `e ∉ E` (with `e = inc(t, 2)`) is the enforcement of T10a's at-most-once-per-`(t, 2)` discipline, discharged by ChildSpawnFreshness at `k' = 2`.
  - Structural identities on `e = inc(t, k)` (consequences of TA5 + T4b's parent projection):
    - **K.δ-ID.zeros-0/1.** `zeros(e) = zeros(t)` for k ∈ {0, 1}. *Derivation:* TA5(c) preserves zeros for k = 0; TA5(d) at k = 1 appends a final `1` with no new zero, so zeros is preserved.
    - **K.δ-ID.zeros-2.** `zeros(e) = zeros(t) + 1` for k = 2. *Derivation:* TA5(d) at k = 2 appends one zero separator and a final `1`.
    - **K.δ-ID.parent-0/1.** `parent(e) = parent(t)` for k ∈ {0, 1}. *Derivation:* k = 0 leaves the trailing-component position unchanged; k = 1 extends by one non-zero component without crossing a zero separator; in either case T4b's truncation past the last separator yields the same prefix.
    - **K.δ-ID.parent-2.** `parent(e) = t` for k = 2. *Derivation:* k = 2 introduces a new zero separator immediately after t, making t itself the parent prefix under T4b.

    These four identities discharge the case-level requirement `parent(e) ∈ E` against the operand's own membership: combined with P8 at `t` for the k ∈ {0, 1} cases (giving `parent(e) = parent(t) ∈ E` by K.δ-ID.parent-0/1), and directly from `t ∈ E` for the k = 2 case (giving `parent(e) = t ∈ E` by K.δ-ID.parent-2).

*Subsumption of ASN-0093's K.σ.* ASN-0047 has no separate K.σ primitive: when `Document(e)`, K.δ subsumes ASN-0093's K.σ (DocumentRegistration) by entering `e` into `E_doc` with `M'(e) = ∅` (the totality convention).

Nelson identifies two document-creation modes — ex nihilo and forking. At the elementary level, both begin with K.δ producing an empty document. When the source's content subspace is non-empty, forking is compound: K.δ followed by arrangement extension and provenance recording (J4 below).

*Frame:* C' = C; L' = L; R' = R. The entity effect `E' = E ∪ {e}` is uniform across all cases. The arrangement frame splits on the level of `e`:

- *Node(e) or Account(e):* `M' = M` — `e ∉ E_doc`, so `dom(M) = E_doc` is unchanged and the arrangement family is untouched.
- *Document(e):* `dom(M') = dom(M) ∪ {e}` with `M'(e) = ∅` and `M'(d') = M(d')` for every `d' ∈ dom(M)`. Since `dom(M) = E_doc` (Bridging lemma (M–E_doc)), entering `e` into `E_doc` *is* growing `dom(M)` by `{e}`, with `M'(e) = ∅`.

**V-position depth (operational).** For a subspace `S ∈ {s_C, s_L}`, we write `m_S(d)` for the depth of document `d`'s *current* `S`-subspace arrangement — the common depth that S8-depth (uniform depth within a subspace, ASN-0036) fixes on `V_S(d)` whenever that set is non-empty, bounded below by the S8a lower bound `m_S(d) ≥ 2`. We write `m_C(d)` and `m_L(d)` for the content- and link-subspace instances. `m_S(d)` is well-defined only while `V_S(d) ≠ ∅`. After full clearance of a subspace (`V_S(d) = ∅`), the next insertion re-pins `m_S(d)` from scratch — at any value `≥ 2` by S8a, not necessarily the prior depth.

**K.μ⁺ (Arrangement extension).** New V→I mappings are added to some d ∈ E_doc, with existing mappings unchanged:

`dom(M'(d)) ⊃ dom(M(d)) ∧ (A v : v ∈ dom(M(d)) : M'(d)(v) = M(d)(v))`

Extension is pure addition — the domain grows, and no existing value is altered.

By value-preservation, no position already in dom(M(d)) can carry a new value, so the newly-mapped positions dom(M'(d)) \ dom(M(d)) are exactly those K.μ⁺ adds disjoint from dom(M(d)).

*Precondition:* `d ∈ E_doc`; for every new mapping M'(d)(v) = a, `a ∈ dom(C)` (S3, ASN-0036 — since K.μ⁺'s frame holds C' = C, referential integrity reduces to membership in the pre-state content store); new V-positions satisfy S8a (all components strictly positive), and the resulting arrangement M'(d) satisfies S8-depth (uniform depth within each subspace); dom(M'(d)) is finite (S8-fin); the resulting arrangement satisfies D-CTG (contiguity of the content subspace `V_1(d)`, ASN-0036) and D-MIN (minimum position of the non-empty content subspace, ASN-0036). *First content insertion:* when `V_{s_C}(d) = ∅`, the depth of the first content V-position is pinned by `ValidFirstInsertionPosition(d, v, m)` (ASN-0036), which for any chosen `m ≥ 2` fixes the unique well-formed first content minimum `v = [s_C, 1, …, 1]` at that depth; K.μ⁺ realises this predicate directly for the content subspace. When K.μ⁺ inserts `K > 1` content positions at once into an empty content subspace, `ValidFirstInsertionPosition` fixes only the minimum; the full first-insertion block `{[s_C, 1, …, 1, k] : 1 ≤ k ≤ K}` of depth `m` is pinned not by that single-position predicate but by the K.μ⁺ D-CTG/D-MIN postconditions on the resulting `M'(d)` (equivalently, by D-SEQ evaluated at Σ'). *Pairwise V-position distinctness on new mappings:* the newly added V-positions `{v_1, …, v_k} := dom(M'(d)) ∖ dom(M(d))` are pairwise distinct (S2, ArrangementFunctionality, ASN-0036).

In a composite transition, K.α may precede K.μ⁺, extending dom(C) before K.μ⁺ executes. At that intermediate state the freshly allocated address is already in the content store, satisfying the precondition. From the composite perspective, the I-address in a new mapping falls into one of two cases:

(i) Freshly allocated — co-occurring K.α places a into dom(C) before K.μ⁺ maps to it. Nelson: "new content enters Istream permanently."

(ii) Previously existing — a ∈ dom(C) at the composite's initial state. This is transclusion: "the copy shares I-addresses with the source. No new content is created in Istream."

*Frame:* C' = C; E' = E; (A d' : d' ≠ d : M'(d') = M(d')); R' = R.

**K.μ⁻ (Arrangement contraction).** Existing V→I mappings are removed from some d ∈ E_doc, with surviving mappings unchanged:

`dom(M'(d)) ⊂ dom(M(d)) ∧ (A v : v ∈ dom(M'(d)) : M'(d)(v) = M(d)(v))`

*Precondition:*
- `d ∈ E_doc`.
- *Suffix-prefix retention (constructive specification, pre-state checkable).* At the elementary level only the content subspace is populated, so by ASN-0036's D-SEQ (D-MIN fixing the minimum, D-CTG fixing contiguity) the content-subspace positions `V_1(d)`, when non-empty, have canonical shape `{[1, 1, ..., 1, k] : 1 ≤ k ≤ n}`. The caller selects a *retention count* `n' ∈ {0, 1, ..., n}` (with `n' = 0` clearing the subspace), subject to the strict-contraction constraint `n' < n`. The contracted arrangement is then determined constructively as the restriction `M'(d) = M(d) ↾ R` to the retained domain `R := {[1, 1, ..., 1, k] : 1 ≤ k ≤ n'}`. The choice of `n'` is the operation's degree of freedom, verifiable at the pre-state without computing M'(d) explicitly: ASN-0036's D-SEQ at Σ supplies `n`, and the caller commits to a retention count.

The per-state arrangement invariants (S2, S3, S8a, S8-depth, S8-fin, D-CTG, D-MIN, D-SEQ of ASN-0036 at M'(d)) and the equivalence of this constructive precondition with the post-state characterization are derived consequences of the restriction form `M'(d) = M(d) ↾ R`.

*Frame:* C' = C; E' = E; R' = R; (A d' : d' ≠ d : M'(d') = M(d')).

**K.ρ (Provenance recording).** A document-content association enters R:

`R' = R ∪ {(a, d)}` where `a ∈ dom(C) ∧ d ∈ E_doc`

*Precondition:* `a ∈ dom(C)` ∧ `d ∈ E_doc`. The level constraint Element(a) follows from S7b (every a ∈ dom(C) satisfies Element(a)).

*Frame (extended state):* C' = C; L' = L; E' = E; (A d :: M'(d) = M(d)).

The per-component modification-mode mapping — which transitions touch which component, and that only M admits contraction or reordering — is tabulated canonically in the *Temporal decomposition* table below; each per-transition effect/frame line above establishes it mechanically.

We observe that neither split nor merge appears as an elementary transition. Nelson addresses this explicitly: the effect of splitting a document is achieved by creating two new documents and transcluding different portions of the original into each. Merging is creating a new document and transcluding from multiple sources. Both compose from K.δ, K.μ⁺, and K.ρ — the elementary transitions suffice.


## Amendments to existing transitions

Only the transitions listed below are amended in the extended state; K.α and K.ρ carry over unchanged, their extended-state frames stated once at their elementary definitions.

**K.μ⁺ amendment (ContentSubspaceRestriction).** K.μ⁺ is amended with a content-subspace restriction: new V-positions must satisfy `subspace(v) = s_C`. This complements K.μ⁺_L, which handles link-subspace extensions exclusively. The restriction `subspace(v) = s_C` confines every K.μ⁺-added V-position to the content subspace, so its image lies in dom(C) and S3★ is discharged. With this amendment, the two transitions partition arrangement extensions by subspace. The amended K.μ⁺ precondition is correspondingly strengthened: where the elementary definition required the resulting `M'(d)` to satisfy D-CTG and D-MIN on the content subspace `V_1(d)`, the extended-state precondition requires `M'(d)` to satisfy D-CTG★ and D-MIN★ (defined immediately below) restricted to the content subspace `S = s_C` — i.e. `V_{s_C}(d)` is contiguous under the V-ordering with `min(V_{s_C}(d)) = [s_C, 1, ..., 1]` when non-empty. The link subspace is preserved by the frame (`M'(d')` unchanged for `d' ≠ d`, and within `d` the link-subspace V-positions are untouched since K.μ⁺ adds only content-subspace positions), so the D-CTG★/D-MIN★ obligations on `V_{s_L}(d)` carry over from the pre-state unchanged.

*Frame (extended state).* `C' = C; L' = L; E' = E; (A d' : d' ≠ d : M'(d') = M(d')); R' = R`.

**D-CTG★ / D-MIN★.** ASN-0036's D-CTG and D-MIN have a link-subspace exemption accommodating Nelson's tombstoning design (LM 4/9). This ASN introduces strengthened forms D-CTG★ and D-MIN★ that apply uniformly across both subspaces:

  **D-CTG★ (per-subspace contiguity).** `(A d, S : V_S(d) ≠ ∅ : V_S(d) is contiguous under T1 restricted to the depth-m_S, subspace-S slice)`, where the slice is the set of depth-m_S positive-component tuples whose first component is S (`m_S` fixed by S8-depth, ASN-0036; "positive" denoting the S8a-compatible domain, components in ℕ⁺), and T1 is LexicographicOrder (ASN-0034); we abbreviate this restriction *the V-ordering on S* below. *Contiguous* unpacks as closed-interval membership: for every `v_lo, v_hi ∈ V_S(d)` and every tuple `z` in the slice with `v_lo ≤ z ≤ v_hi` under T1, `z ∈ V_S(d)` — well-defined whenever S8-depth and S8a hold at the state under consideration. The closed-interval formulation is what D-CTG★ unpacks to in the derivations below — appeals to D-CTG★ discharge to "every depth-m_S positive tuple lex-between two named members of V_S(d) is itself in V_S(d)" without further unpacking.

  **D-MIN★ (per-subspace minimum position).** `(A d, S : V_S(d) ≠ ∅ : min(V_S(d)) = [S, 1, ..., 1] of depth m_S)`, the minimum taken under T1 on the depth-m_S, subspace-S slice.

D-CTG★/D-MIN★ constrain the arrangement layer `M(d)`; link permanence is discharged independently on `dom(L)` by L12 (LinkImmutability).

**S8★ (per-subspace span decomposition).** S8★ states the per-subspace analogue of ASN-0036's S8 (CorrespondenceRunPartition):

For each `d ∈ E_doc` and each subspace `S ∈ {s_C, s_L}`, the per-subspace arrangement `M(d)|_{V_S(d)}` decomposes into a finite set of correspondence runs `{(v_j, a_j, n_j)}`: every `v ∈ V_S(d)` lies in exactly one run, and within each run the V-positions and I-addresses advance by shift in lockstep. S8★ retains the finite-run *partition* together with ASN-0036's S8 conditions (a) (lockstep displacement) and (b) (label well-definedness); condition (c) (uniqueness of the maximal-run decomposition) is retained only on the content subspace. The decomposition is established by two distinct routes, one per subspace — the content subspace by direct application of ASN-0036's S8 (keeping (c)), the link subspace by a trivial length-1 decomposition (dropping (c)):

- *Content subspace.* `M(d)|_{V_{s_C}(d)} : V_{s_C}(d) → dom(C)` is an application of ASN-0036's S8: S3★ restricted to V_{s_C}(d) is exactly S3 (with target `dom(C)`), and S2, S7b, C1b (ASN-0093), S8a, S8-depth, S8-fin are elementary-preserved per the verification below. One closure step is load-bearing: ASN-0036's S8 condition (a) quantifies the lockstep image `shift(v, k) ∈ dom(M(d))` over the *full* arrangement, so applying it to the projection requires that lockstep images of content V-positions land back in `V_{s_C}(d)` rather than escaping into `V_{s_L}(d)`. This is OrdShiftHom(a) (ASN-0036): `subspace(shift(v, k)) = subspace(v) = s_C` for every `v ∈ V_{s_C}(d)` and `k ≥ 0` (under the `shift(t, 0) := t` convention, the `k = 0` case is the identity), so the content projection is shift-closed and ASN-0036's S8 quantifier over `dom(M(d))` restricts soundly to `V_{s_C}(d)`.
- *Link subspace.* `M(d)|_{V_{s_L}(d)} : V_{s_L}(d) → dom(L)` cannot use ASN-0036's S8 directly because its range lies in `dom(L)` not `dom(C)`, falsifying S3; S7b/C1b also do not apply (they constrain `dom(C)`-resident addresses). S8★(s_L) is instead discharged by the *trivial length-1 decomposition* `{(v, M(d)(v), 1) : v ∈ V_{s_L}(d)}` — every link-subspace V-position constitutes its own length-1 correspondence run. The run-cover partition (every `v ∈ V_{s_L}(d)` lies in exactly one run) holds by construction — each `v` is the sole element of its singleton run. ASN-0036's condition (a) (lockstep displacement) under the substitution `dom(C) → dom(L)` requires `shift(vⱼ, k) ∈ dom(M(d))` and `M(d)(shift(vⱼ, k)) = shift(aⱼ, k)` together with `shift(aⱼ, k) ∈ dom(L)`, for `0 ≤ k < nⱼ`; at `nⱼ = 1` the quantifier reduces to the single case `k = 0`. We adopt ASN-0036's S8 convention `shift(t, 0) := t` for any tumbler `t` — the same convention the content-subspace discharge above inherits from S8. Under this convention, the lockstep clause at `k = 0` reduces to `M(d)(v) = M(d)(v)`, which holds trivially, and the label-membership clause is `M(d)(v) ∈ dom(L)`, which holds because every link-subspace V-position targets `dom(L)` in the extended state. ASN-0036's condition (b) (label well-definedness) under the same substitution requires `aⱼ = M(d)(vⱼ)` well-defined by S2 with `aⱼ ∈ dom(L)`; both hold for the singleton labels. The trivial decomposition invokes no S8 machinery beyond the run-counting structure itself, and finiteness of the decomposition follows from S8-fin's finiteness of `dom(M(d))`. Because (c) is dropped, S8★(s_L) asserts the existence of *a* run-partition satisfying (a) and (b) — the length-1 decomposition — not the canonical maximal-run partition of ASN-0036's S8: when two adjacent link V-positions happen to stand in lockstep, both the length-1 decomposition and a longer maximal one satisfy (a)/(b), so the link-subspace partition is non-canonical.

**D-SEQ★ (per-subspace sequential positions, derived).** For each non-empty subspace S in M(d):

  `V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` for some `n_S ≥ 1`,

where the inner positions are of uniform depth m_S (the common depth within subspace S, by S8-depth), and `n_S = |V_S(d)|`. At the practical depth `m_S = 2` the inner "1, ..., 1" segment has length `m_S - 2 = 0`, so the canonical form degenerates to `{[S, k] : 1 ≤ k ≤ n_S}`; worked examples below at `m_S = 2` invoke this degeneracy without further annotation.

D-SEQ★ is re-established in full detail here from the amended D-CTG★ + D-MIN★ + S8-depth + S8-fin + S8a.

*Derivation.* Fix d and a non-empty subspace S, and abbreviate `m := m_S`, `n := n_S`. By D-MIN★, V_S(d) contains the minimum position `v_min` (of the form `[S, 1, ..., 1]` of depth m). By S8-depth, every v ∈ V_S(d) has #v = m. By S8a, every component of every v ∈ V_S(d) is strictly positive (in ℕ⁺). By S8-fin, V_S(d) is finite; let n := |V_S(d)|. The V-ordering on a fixed subspace at a fixed depth is the standard lexicographic order on ℕ⁺-valued tuples. We treat the cases `m = 2` (the practical case driving every text-subspace worked example) and `m ≥ 3` separately so the m = 2 derivation is self-contained — no degenerate notation, no deferral.

*Case m = 2.* Every v ∈ V_S(d) has the form `[S, v_2]` with `v_1 = S` (by the definition of V_S(d), which projects M(d) to positions with `subspace(v) = S = v_1`) and `v_2 ∈ ℕ⁺` (by S8a). D-MIN★ gives `v_min = [S, 1] ∈ V_S(d)`. By S8-fin, let `v_max = max(V_S(d)) = [S, k_max]` for some `k_max ∈ ℕ⁺`. For each `k ∈ ℕ⁺` with `1 ≤ k ≤ k_max`, the depth-2 positive tuple `z = [S, k]` satisfies `v_min ≤ z ≤ v_max` under the V-ordering on S (subspace identifier matches; lex order on the terminal component agrees with the natural order on `k`); D-CTG★'s closed-interval membership then places `z ∈ V_S(d)`. Conversely, any v ∈ V_S(d) is `[S, v_2]` with `v_2 ∈ ℕ⁺` and `v_2 ≤ k_max` (since `v ≤ v_max`), so `v` is one of the listed tuples. Therefore `V_S(d) = {[S, k] : 1 ≤ k ≤ k_max}`. Counting gives `k_max = n`, delivering the m = 2 specialisation of D-SEQ★ directly. ∎ (case m = 2)

*Case m ≥ 3.* The depth supports an inner range `2 ≤ j ≤ m - 1` between the subspace position (1) and the terminal position (m), and the derivation proceeds in two steps.

*Step 1 (inner positions fixed at 1).* We show that every v ∈ V_S(d) satisfies `v_j = 1` for `2 ≤ j ≤ m - 1`. The inner positions `2 ≤ j ≤ m - 1` form a nonempty range for `m ≥ 3`, degenerating to the single position `j = 2` at `m = 3`. Suppose for contradiction that some v ∈ V_S(d) has v_j ≥ 2 at the *minimal* inner position j with `2 ≤ j ≤ m - 1`. By minimality, `v_l = 1` for `2 ≤ l < j`; combined with `v_1 = S`, v agrees with `v_min` on positions 1..j - 1, and `v_j > v_min[j] = 1`, so `v_min < v` in lex order. For each integer `M ≥ 2`, define the depth-m tuple
  `u_M := [S, 1, ..., 1, 1, M, 1, ..., 1]`
with `S` at position 1, `1` at every position from 2 through j, `M` at position j + 1, and `1` at every remaining position from j + 2 through m. (When j = m - 1, the trailing range j + 2..m is empty; the tuple becomes `[S, 1, ..., 1, 1, M]` with M at the terminal — the construction's placement of M coincides with the terminal position whenever the minimal inner position is the rightmost-but-one.) Each u_M has all positive components, so it inhabits the V-ordering's domain at depth m.

We verify `v_min < u_M < v` for each M ≥ 2:
  - `v_min < u_M`: v_min and u_M agree on positions 1..j (both have `S` at 1 and `1` everywhere through position j); they first differ at position j + 1, where `v_min[j+1] = 1 < M = u_M[j+1]`.
  - `u_M < v`: u_M and v agree on positions 1..j - 1 (both have `S` at 1 and `1` at positions 2..j - 1); they first differ at position j, where `u_M[j] = 1 < v_j` (since v_j ≥ 2 by hypothesis).
Each u_M is a depth-m positive tuple with subspace identifier S satisfying `v_min < u_M < v`, so by D-CTG★'s closed-interval membership (v_min, v ∈ V_S(d) bracket a closed interval), u_M ∈ V_S(d). The map `M ↦ u_M` is injective (u_M and u_{M'} disagree at position j+1 whenever M ≠ M'), so `{u_M : M ≥ 2}` is a countably infinite subset of V_S(d). This contradicts S8-fin's finiteness of `dom(M(d))`, discharging the hypothesis that some `v ∈ V_S(d)` has an inner position ≥ 2.

Therefore no v ∈ V_S(d) has an inner position ≥ 2: every v has `v_j = 1` for `2 ≤ j ≤ m - 1`, and the only remaining freedom is in the terminal position v_m. So every v ∈ V_S(d) has the form `[S, 1, ..., 1, k]` for some `k ∈ ℕ⁺`, where the inner "1, ..., 1" segment has length `m - 2 ≥ 1`.

*Step 2 (terminal contiguity).* Restricted to terminal-varying tuples `[S, 1, ..., 1, k]`, the V-ordering coincides with the natural order on `k`. By S8-fin, n < ∞; let `v_max = max(V_S(d)) = [S, 1, ..., 1, k_max]` for some `k_max ∈ ℕ⁺` (well-defined since V_S(d) is finite and non-empty). By D-CTG★'s closed-interval-membership content, every depth-m positive tuple z with subspace identifier S satisfying `v_min ≤ z ≤ v_max` is in V_S(d) (v_min and v_max are both in V_S(d), bracketing a closed interval admissible to the D-CTG★ premise); restricted to terminal-varying tuples `[S, 1, ..., 1, k]`, this gives `{[S, 1, ..., 1, k] : 1 ≤ k ≤ k_max} ⊆ V_S(d)`. The reverse inclusion follows from v_max being the maximum: any `[S, 1, ..., 1, k]` with `k > k_max` would exceed v_max in lex order. Hence `V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ k_max}` and counting gives `k_max = n`, the m ≥ 3 form of D-SEQ★. ∎ (case m ≥ 3)

The two cases together cover every reachable state under S8a + S8-depth. The canonical form for D-SEQ★ thus reads `{[S, k] : 1 ≤ k ≤ n_S}` at m = 2 and `{[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` at m ≥ 3 (with the inner "1, ..., 1" segment of length `m - 2`); the consolidated statement at the head of this definition uses the m ≥ 3 spelling and silently degenerates to the m = 2 form when the inner segment has length zero.

**K.μ⁻ amendment (PerSubspaceScope).** The extended state adds the link subspace, so two changes apply over the elementary definition. First, the link-store frame clause `L' = L` is added. Second, the elementary single-content-subspace retention count `n'` generalizes to a *per-subspace* retention count: under D-SEQ★ at Σ each non-empty `V_S(d)` has canonical shape `{[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` for `S ∈ {s_C, s_L}`, and the caller selects, for each S, a retention count `n'_S ∈ {0, 1, ..., n_S}` (with `n'_S = 0` when `V_S(d) = ∅`), subject to at least one S admitting strict contraction `n'_S < n_S`. The contracted arrangement is the restriction `M'(d) = M(d) ↾ R` to `R := ∪_{S ∈ {s_C, s_L}} {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}`; D-SEQ★ at Σ supplies `(n_{s_C}, n_{s_L})`, keeping the precondition pre-state checkable. The effect, contraction shape, and per-subspace postconditions otherwise carry over from the elementary definition; the constructive/post-state equivalence is proved next.

*Frame (extended state).* `C' = C; L' = L; E' = E; R' = R; (A d' : d' ≠ d : M'(d') = M(d'))`.

**K.μ⁻ admissible contraction shape (equivalence of constructive and post-state characterizations).** K.μ⁻'s precondition specifies the post-state constructively via the per-subspace retention count `n'_S`. We show this is *equivalent* to the post-state characterization "M'(d) satisfies D-CTG★ + D-MIN★ + D-SEQ★ and dom(M'(d)) ⊂ dom(M(d)) (strict, proper subset) with value preservation on survivors," strictness included on both sides.

*Forward direction (constructive ⟹ post-state invariants).* The constructive form `M'(d) = M(d) ↾ R` with `R = ∪_S {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` satisfies D-CTG★ at Σ' (each non-empty `V_S(d')` is the contiguous prefix `{[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}`), D-MIN★ at Σ' (when non-empty, `min(V_S(d')) = [S, 1, ..., 1, 1]`), and D-SEQ★ at Σ' (the canonical shape is exhibited by construction). Restriction preserves single-valuedness (S2), referential integrity (S3★), componentwise positivity (S8a), uniform depth (S8-depth), and finiteness (S8-fin) on survivors.

*Reverse direction (post-state invariants ⟹ constructive form).* Hypothesise a candidate contraction `M_cand(d)` with `dom(M_cand(d)) ⊂ dom(M(d))` (strict, proper subset), value-preservation on survivors (`(A v ∈ dom(M_cand(d)) :: M_cand(d)(v) = M(d)(v))`), and D-CTG★ + D-MIN★ at Σ' together with S2 and S3★. We show that `M_cand(d)` equals `M(d) ↾ R` for some per-subspace `R = ∪_S {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` of the constructive form. Fix a subspace S, and write `V_S(d') := {v ∈ dom(M_cand(d)) : subspace(v) = S}` for the candidate's per-subspace projection. If `V_S(d') = ∅`, the conclusion holds with `n'_S = 0`. Otherwise D-SEQ★ at Σ' is derived from the candidate's D-CTG★/D-MIN★ together with S8-fin/S8a/S8-depth at the candidate: S8-fin because `dom(M_cand(d))` is a subset of the finite set `dom(M(d))`; S8a and S8-depth because every `v ∈ dom(M_cand(d))` is a pre-state V-position (`v ∈ dom(M(d))`) and so retains its pre-state shape — zero-free, all components positive, depth `m_S`. The derived D-SEQ★ then gives `V_S(d') = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` for some `n'_S ≥ 1` directly. S8-depth at Σ' inherits `m_S` from the surviving V-positions of Σ — no survivor changes depth, since each `v ∈ dom(M_cand(d))` keeps its pre-state V-position identity — making `V_S(d')` and `V_S(d)` share the same canonical D-SEQ★ shape `V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` and `V_S(d') = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` at the common depth `m_S`. Explicit comparison via the trailing-component bijection: define `φ_S : {1, ..., n_S} → V_S(d)` by `φ_S(k) = [S, 1, ..., 1, k]`, and symmetrically `φ'_S : {1, ..., n'_S} → V_S(d')`. By T3 (CanonicalRepresentation, ASN-0034), distinct values of `k` at a fixed inner-tuple pattern produce distinct tumblers, so `φ_S` and `φ'_S` are both bijections — `φ_S` between `{1, ..., n_S}` and `V_S(d)`, and `φ'_S` between `{1, ..., n'_S}` and `V_S(d')`. Set inclusion `V_S(d') ⊆ V_S(d)` translates pointwise under these bijections: every `[S, 1, ..., 1, k'] ∈ V_S(d')` is also in `V_S(d)`, so `k' ∈ {1, ..., n_S}`, giving `{1, ..., n'_S} ⊆ {1, ..., n_S}`, which forces `n'_S ≤ n_S` by direct inspection of initial-segment containment in ℕ. Taking the union of these per-subspace prefix shapes gives `dom(M_cand(d)) = R := ∪_S {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}`; combined with the hypothesised value-preservation on survivors, `M_cand(d) = M(d) ↾ R`, the constructive form. Finally, the strict-subset hypothesis `dom(M_cand(d)) ⊂ dom(M(d))` is a proper containment, so the union `R` omits at least one V-position of `dom(M(d))`; since every `V_S(d')` is the prefix `{1, ..., n'_S}` of `V_S(d) = {1, ..., n_S}` under the bijections above, the omitted position lies in some subspace S with `n'_S < n_S`, recovering the constructive precondition's strict-contraction clause `(E S :: n'_S < n_S)`. The biconditional therefore matches on every conjunct, strictness included. ∎

## Allocator hierarchy under documents

The content- and link-subspace allocators are organized as sibling element-field sub-allocators rooted at each document. The anchor and sub-allocator notation used throughout this ASN — `b_C(d)`, `b_L(d)`, `A_C(d)`, `A_L(d)` — follows ASN-0093 (the anchors and chains of its FirstEmission and ChainDiscipline lemmas).

For each `d ∈ E_doc`, the document-level address `d` (zeros = 2) is the root of d's allocator subtree. Per ASN-0093, two element-field bases sit immediately under d:

- `b_C(d) := [d.0.s_C]` (single-component element field with E₁ = s_C; zeros = 3, #E = 1) — the **content sub-allocator anchor**.
- `b_L(d) := [d.0.s_L]` (single-component element field with E₁ = s_L; zeros = 3, #E = 1) — the **link sub-allocator anchor**.

These anchors are structurally producible via T10a inc steps from `d`. Under SubspaceConventionAxiom (`s_C = 1` and `s_L = 2`), `b_C(d) = inc(d, 2) = [d.0.1]` (TA5(d) with k = 2), and `b_L(d) = inc(b_C(d), 0) = inc([d.0.1], 0) = [d.0.2]` (TA5(c)). The anchors are not themselves in `dom(C) ∪ dom(L)` — content addresses have `#E ≥ 2` (C1b, ASN-0093), link addresses have `#E ≥ 2` (L1b), and the anchors have `#E = 1` — so they inhabit the foundation carrier set `T` but no state component of Σ.

**Sub-allocator names** (A_C(d) and A_L(d) per ASN-0093; A_v(d), A_doc(A), A_account(N) introduced here):

- `A_C(d)` — d's **content sub-allocator**, with anchor `b_C(d) = [d.0.s_C]` and first emission `[d.0.s_C.1]`. Its outputs `a` satisfy `a ∈ dom(C)`, `subspace_I(a) = s_C`, `origin(a) = d`, and `zeros(a) = 3` (element-level).
- `A_L(d)` — d's **link sub-allocator**, with anchor `b_L(d) = [d.0.s_L]` and first emission `[d.0.s_L.1]`. Its outputs `ℓ` satisfy `ℓ ∈ dom(L)`, `subspace_I(ℓ) = s_L`, `origin(ℓ) = d`, and `zeros(ℓ) = 3` (element-level).
- `A_v(d)` — d's **version sub-allocator**, with no element-field anchor (it emits at the entity-hierarchy level, not the element-field level). Its first emission is `inc(d, 1)`, an entity-level address with `zeros = 2` (a new document under T4b sharing `parent(·) = parent(d)`); subsequent emissions are sibling-advances `inc(prev_version, 0)` on its frontier. Its outputs are versions of d and inhabit `E_doc`.
- `A_doc(A)` — account `A`'s **document sub-allocator** (introduced here), defined for each `A ∈ E_account`. Its first emission is `inc(A, 2)` — equivalently, the determinate tumbler `[A.0.1]` — an entity-level address with `zeros = 2` and `parent(·) = A` under T4b. Subsequent emissions are sibling-advances `inc(prev_doc, 0)` on its frontier. Its outputs are original (non-version) documents under `A` and inhabit `E_doc`.
- `A_account(N)` — node `N`'s **account sub-allocator** (introduced here), defined for each `N ∈ E_node`. Its first emission is `inc(N, 2)` — equivalently, the determinate tumbler `[N.0.1]` — an entity-level address with `zeros = 1` and `parent(·) = N` under T4b. Subsequent emissions are sibling-advances `inc(prev_account, 0)` on its frontier. Its outputs are accounts under `N` and inhabit `E_account`.

**ParentAllocatorDispatch (sub-lemma).** For any entity `t` produced by an activated allocator, the unique allocator `A` whose realized domain `domₛ(A)` contains `t` is determined by T10a.6 (DomainDisjointness, ASN-0034) — every `t` inhabits exactly one allocator's domain — and that membership is preserved across every subsequent transition by AllocatedSet's `domₛ(A) ⊆ dom_{s'}(A)` monotonicity (ASN-0034). The identification rests on a structural fact of the K.δ construction: the level of an entity address — its zero count, hence its T4c stratum (ASN-0045) — is fixed by the sub-allocator family that can emit it, because the spawn parameter `k` of the activating K.δ step determines the resulting zero count.

- **Account level (`Account(t)`, `zeros(t) = 1`).** `t`'s unique owning allocator is `A_account(parent(t))`, and `t ∈ dom(A_account(parent(t)))`.
- **Document level (`Document(t)`, `zeros(t) = 2`).** Exactly one of two cases holds, mutually exclusive and exhaustive by T10a.6: **(a')** `t ∈ dom(A_doc(parent(t)))` — ex nihilo, and `A_v(t)` (when spawned) is a child of `A_doc(parent(t))`, with spawnPt `t`, spawnParam `1`; or **(b')** `t ∈ dom(A_v(d'))` for some `d' ∈ E_doc` — fork, and `A_v(t)` is a child of `A_v(d')`, with spawnPt `t`, spawnParam `1`.

*Proof.* By T4c (ASN-0045) the zero count fixes the level: `zeros = 1` for accounts, `zeros = 2` for documents. For each level we identify the sub-allocator families that emit addresses at that level under the K.δ construction, then recover the allocator base from the address prefix.

*Account level.* The only K.δ step producing a `zeros = 1` address is a k = 2 spawn off a node `N` — `inc(N, 2) = [N.0.1]`, the first emission of `A_account(N)` — followed by k = 0 sibling-advances `inc(·, 0)`, which preserve length and zero count (TA5(c)) and so remain at `zeros = 1` under the node prefix `N`. No k = 1 step (which appends a component and raises the zero count by one) and no k = 2 step off a non-node yields a `zeros = 1` address. Hence every account inhabits some account sub-allocator `A_account(N)`. The node base is recovered structurally: every emission of `A_account(N)` extends `N` by prefix (ChainPrefixExtension-style, via TA5(b)/(c)) and carries `parent(·) = N` (T4b), so `N = parent(t)`. Combined with T10a.6 uniqueness, `t`'s owning allocator is exactly `A_account(parent(t))`, discharging the account-level claim.

*Document level.* A `zeros = 2` address arises either from a k = 2 spawn off an account `A` — `inc(A, 2) = [A.0.1]`, the first emission of `A_doc(A)`, plus k = 0 advances (the ex-nihilo case) — or from a k = 1 spawn off a document `d'` — `inc(d', 1)`, the first emission of `A_v(d')`, plus k = 0 advances (the fork case). These exhaust the document-level emitters: no other spawn parameter produces `zeros = 2`. T10a.6 makes (a') and (b') mutually exclusive — the cross-allocator disjointness `dom(A_doc(parent(t))) ∩ dom(A_v(d')) = ∅` forbids `t` inhabiting both — and exhaustive over the document-level emitters just enumerated. In case (a') the recovered account base is `parent(t)` (by the account-level prefix argument applied one level down); in case (b') the version parent `d'` is the base of `t`'s owning version allocator. ∎

Outputs of `A_C(d)` and `A_L(d)` are *not* entity-level (their outputs inhabit `dom(C) ∪ dom(L)` at `zeros = 3`); outputs of `A_v(d)` *are* entity-level (they enter `E_doc` at `zeros = 2`). All three are T10a-conforming sub-allocators within d's allocator subtree.

Once each element-field anchor heads a frontier (established by ASN-0093's sub-allocator lemmas, bundled as SubAllocatorBundle below), the sub-allocator behaves as a T10a-conforming `inc(·, 0)` chain: the first content address under d is `[d.0.s_C.1]`, subsequent siblings advance by `inc([d.0.s_C.k], 0)` (TA5(c)); the first link address is `[d.0.s_L.1]`, subsequent siblings by `inc(ℓ_prev, 0)`. The two frontiers advance independently — each inc step operates locally under its subspace prefix.

**Sub-allocator activation (SubAllocatorBundle).** For each `d ∈ E_doc`, the entity-allocation event placing d into E_doc activates a content sub-allocator `A_C(d)` with anchor `b_C(d) = [d.0.s_C]` and a link sub-allocator `A_L(d)` with anchor `b_L(d) = [d.0.s_L]`, the anchors constructed as in *Allocator hierarchy under documents* above. The standing properties of these chains — T10a-conforming `inc(·, 0)` sibling-advance discipline with strictly increasing enumeration; determinate first emission `[d.0.s_C.1]` (resp. `[d.0.s_L.1]`) with `origin = d`, `#E = 2`, `zeros = 3`, T4-valid and fresh against `dom(Σ.C) ∪ dom(Σ.L)` at the allocating state; and every chain element T4-valid with `zeros(·) = 3`, inhabiting its subspace (`subspace_I(·) = s_C` along `A_C(d)`, `s_L` along `A_L(d)`) — are inherited from ASN-0093's sub-allocator lemmas. The one obligation the bundle must discharge beyond those is the cross-subspace disjointness delta.

This delta is: `dom(A_C(d)) ∩ dom(A_L(d)) = ∅`, and for any d ≠ d', `dom(A_C(d)) ∩ dom(A_C(d')) = ∅`, `dom(A_L(d)) ∩ dom(A_L(d')) = ∅`, `dom(A_C(d)) ∩ dom(A_L(d')) = ∅`. The within-document disjointness is DisjointSubAllocatorChains (ASN-0093) directly; the cross-document same-subspace clauses are CrossDocumentDisjointness (ASN-0093) at anchor pairs `(b_C(d), b_C(d'))` and `(b_L(d), b_L(d'))`. The cross-subspace fourth clause `dom(A_C(d)) ∩ dom(A_L(d')) = ∅` (anchors `[d.0.s_C]`, `[d'.0.s_L]`) is the genuine delta, dispatched by CrossDocDisjoint (below) at `(s₁, s₂) = (s_C, s_L)`.

**Lemma (Cross-document disjointness chain — account-level and cross-subspace extension).** The document-level same-subspace case is ASN-0093's CrossDocumentDisjointness (derivation chain T10a.{2,5} → T10). This lemma extends that fact to cross-subspace anchor pairings and to account-level entity pairs.

*Statement.* For any two distinct same-level entities `e₁ ≠ e₂` (both with `zeros(eᵢ) = z` for a fixed level `z`), and any sub-allocator prefixes `p₁ := [e₁.0.s₁]`, `p₂ := [e₂.0.s₂]` with `s₁, s₂ ≥ 1` (possibly distinct), the prefixes satisfy `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`; by T10 (PartitionIndependence, ASN-0034) every address extending `p₁` differs from every address extending `p₂`.

*Document level (`e₁ = d₁, e₂ = d₂ ∈ E_doc`, `s₁ = s₂`).* The same-subspace anchor pairs `(b_C(d₁), b_C(d₂))` and `(b_L(d₁), b_L(d₂))` are exactly the hypotheses of ASN-0093's CrossDocumentDisjointness, whose T10a.{2,5} → T10 derivation we inherit unchanged.

*Extension (the two deltas).* The cross-subspace pairing (`s₁ ≠ s₂`) and the account-level pairing (`e₁, e₂ ∈ E_account`) both follow from a single observation: the divergence between `p₁` and `p₂` lies entirely within the entity portion, strictly before the subspace-component slots at positions `#e₁ + 2`, `#e₂ + 2`. Case-split on the prefix relationship of `e₁`, `e₂` (exhaustive):

- *Prefix-comparable* (WLOG `e₁ ≺ e₂`): Prefix + T3 (ASN-0034) with `e₁ ≠ e₂` give `#e₁ < #e₂`; the shared-level precondition `zeros(e₁) = zeros(e₂) = z` pins `e₂`'s zero count, so its positions beyond `#e₁` carry no further separators and `e₂[#e₁+1] ≠ 0`, while `p₁` seats its own zero separator at `#e₁+1`. Since `#p₁ = #e₁ + 2 ≤ #e₂ + 2 = #p₂`, the divergence index `#e₁ + 1` sits strictly inside both prefixes.
- *Prefix-incomparable* (`e₁ ⋠ e₂ ∧ e₂ ⋠ e₁`): the divergence position `k ≤ min(#e₁, #e₂)` carries to `p₁[k] = e₁[k] ≠ e₂[k] = p₂[k]`.

In both branches the subspace components `s₁, s₂` (positions `#eᵢ + 2`) are never consulted — so the conclusion is independent of whether `s₁ = s₂` or `s₁ ≠ s₂` — and the only level-dependent input is the shared-level precondition `zeros(e₁) = zeros(e₂) = z`, which holds for account-level pairs (`z = 1`) exactly as for document-level pairs (`z = 2`). Prefix (ASN-0034) witnesses `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`, and T10 closes the lemma. ∎

We abbreviate this lemma **CrossDocDisjoint**. It dispatches the document-level cross-subspace pairing between `b_C(d) = [d.0.s_C]` and `b_L(d') = [d'.0.s_L]` (`s₁ ≠ s₂`) and the account-level pairing, where the document sub-allocator's first emission `inc(A, 2) = [A.0.1]` is the relevant prefix (the difference between minted-direct and minted-via-anchor lies in the activation discharge, not in the cross-entity disjointness analysis).

**Lemma (Cross-document entity distinctness).** Distinct K.δ document-allocation events produce distinct document addresses. We abbreviate this lemma **CrossDocEntityDisjoint**; it is the entity-level analogue of CrossDocDisjoint, and it subsumes the node-nesting sub-argument **CrossNodeAccountBase** stated within.

*Statement.* For distinct documents `d₁ ≠ d₂` with `parent(d₁) = A₁`, `parent(d₂) = A₂`: `d₁ ≠ d₂`. We case-split on whether the parent accounts coincide.

*Case `A₁ ≠ A₂` (distinct parents) — account non-nesting.* We apply T10 (PartitionIndependence, ASN-0034) at the account level with `p₁ = A₁, p₂ = A₂`. The non-nesting precondition `A₁ ⋠ A₂ ∧ A₂ ⋠ A₁` follows from T10a's discipline on accounts as a binary case-split: two distinct accounts emitted by the same account sub-allocator (siblings under a common parent node) are prefix-incomparable by T10a.2 (NonNestingSiblingPrefixes); two distinct accounts emitted by different account sub-allocators (under distinct parent nodes `N₁ ≠ N₂`) are prefix-incomparable by T10a.5 (CrossAllocatorIncomparability), whose non-ancestor-descendant precondition is discharged by CrossNodeAccountBase below.

*CrossNodeAccountBase.* The two account sub-allocator bases are prefix-incomparable even under node nesting. Each account sub-allocator of a node `N` is based at `b_account(N) = [N.0.1]` (the `inc(N, 2)` descent, `zeros = 1`, account-level — *Sub-allocator names*). We show `b_account(N₁) ⋠ b_account(N₂) ∧ b_account(N₂) ⋠ b_account(N₁)` for any `N₁ ≠ N₂`. If neither node is a prefix of the other, they diverge at some position `k ≤ min(#N₁, #N₂)` and both bases inherit that divergence at `k`, so neither is a prefix of the other. If one node nests in the other — say `N₁ ≼ N₂` with `#N₁ < #N₂` (the nesting configuration SSGU permits) — then this is exactly the zero-separator divergence proved in SSGU (NodeRootedForest), instantiated at `a := b_account(N₁)`, `a' := b_account(N₂)`: `b_account(N₁) = inc(N₁, 2)` is a non-node `inc`-output under `N₁`, so it carries the field-separating zero at position `#N₁ + 1`, while `b_account(N₂)` extends `N₂` whose component there is nonzero. SSGU's divergence at the shared position `#N₁ + 1` gives prefix-incomparability of the two bases directly. Either way the bases are prefix-incomparable, so the two account sub-allocators are non-ancestor-descendant (their domains extend their respective bases by prefix, per ChainPrefixExtension-style TA5(b)/(c)), discharging T10a.5's precondition; T10a.5 then yields `A₁ ⋠ A₂ ∧ A₂ ⋠ A₁`. ∎ (CrossNodeAccountBase)

*Document distinctness.* With `A₁ ⋠ A₂ ∧ A₂ ⋠ A₁` established, CrossDocDisjoint's non-nesting-prefix ⟹ partition-independence step applies at the account-prefix pair `(A₁, A₂)`: every document `[Aᵢ.0.k]` extends its parent account (the parent prefix preserved across `inc(A, 2)` and `inc(·, 0)` emissions by TA5(b)/(c)), so `d₁ ≠ d₂` across all sibling positions `k, k' ≥ 1` — not only the first emissions.

*Case `A₁ = A₂` (same parent).* Within a single parent account, two distinct documents need not inhabit the same sub-allocator chain: a direct document `[A.0.1]` lies on `A_doc(A)`, whereas a version `[A.0.1.1]` lies on `A_v([A.0.1])`, yet both have parent account `A` (a version preserves `parent(d_new) = parent(d_src)` by K.δ-ID.parent-0/1). Distinctness across such cross-chain same-parent pairs is discharged by SSGU (NodeRootedForest) across distinct K.δ allocation events — both documents inhabit the single node-rooted subtree of the node above their shared parent account `A`, so SSGU assigns each to a unique allocation event within that subtree; this covers cross-chain pairs without requiring co-residence on one chain. The single-chain sub-case (two siblings on the same `A_doc(A)`) is the special case where GlobalUniqueness reduces to within-chain enumeration injectivity. ∎ (CrossDocEntityDisjoint)

Cross-subspace collisions are further prevented by L14 (StoreDisjointness) — ASN-0093's SD (StoreDisjointness), collected in the *Inherited from foundation* table below: every content address has `subspace_I(a) = s_C`, every link address has `subspace_I(ℓ) = s_L`, and `s_C ≠ s_L`, so no allocation in one subspace can produce an address inhabiting the other.

**Lemma (SubAllocatorFreshness).** *Single freshness-discharge for element-field sub-allocator emissions, parametric in `x ∈ {C, L}`.* Let `d ∈ E_doc` and let `A_x(d)` be d's content (`x = C`) or link (`x = L`) sub-allocator. Suppose a K.α (for `x = C`) or K.λ (for `x = L`) firing emits the address `a` per the operation's emission cases. Then `a ∉ dom(C) ∪ dom(L)` at the pre-state, discharged in three parts:

- *Seed* (first emission, predicate `{a' ∈ dom(x-store) : origin(a') = d} = ∅`): `a` is the determinate first emission `[d.0.s_x.1]`, fresh against `dom(C) ∪ dom(L)` by FirstEmissionFreshness (ASN-0093).
- *Frontier advance* (subsequent emission, predicate non-empty): `a = inc(max{a' ∈ dom(x-store) : origin(a') = d}, 0)` (TA5(c)) is the next sibling on `A_x(d)`'s inc chain; freshness against the same-subspace store is GlobalUniqueness (ASN-0034) on `A_x(d)`'s activated chain (the T10a-conforming `inc(·, 0)` chain of ChainDiscipline / ChainEnumerationInjectivity, ASN-0093).
- *Cross-subspace*: freshness against the opposite store is SC-NEQ + T7 (SubspaceDisjointness, ASN-0034), equivalently L14 (StoreDisjointness) at the pre-state.

We abbreviate this **SubAllocFresh**.


## Link allocation

**K.λ (LinkAllocation).** ASN-0093's K.λ (LinkAllocation), with frame extended by `E' = E ∧ R' = R` (Frame convention for inherited transitions). Freshness `ℓ ∉ dom(L) ∪ dom(C)` is SubAllocFresh at `x = L`.

*Effect:* `L' = L ∪ {ℓ ↦ (e₁, …, eₙ)}`.

*Frame:* `C' = C; (A d' :: M'(d') = M(d'))` (E and R held in frame per the opening clause).

Cross-document disjointness for distinct home documents is CrossDocumentDisjointness (ASN-0093), applied at anchor pair `(b_L(d), b_L(d'))`.


## K.δ case (ii) discharge and parent-allocator activation

A non-node K.δ event's step `e = inc(t, k)` acts on a parent entity-level sub-allocator that depends on k. The operand-admissibility and freshness conditions for each sub-case are fixed at the K.δ box (case (ii), k = 0/1/2) and are not restated here; this section records only the *parent-allocator activation* each k induces, and the discharge of the T10a child-spawn premise.

- *k = 0:* no new allocator is activated — `e = inc(t, 0)` is a T10a sibling-advance on the allocator already tracking `t` (the same allocator carrying `parent(e)` by the `parent(t) = parent(e)` identity).

- *k = 1:* `e = inc(t, 1)` is the T10a child-spawn that activates `A_v(t)`, t's version sub-allocator. Its unique parent allocator is identified by ParentAllocatorDispatch (*Sub-allocator names*) from t's owning allocator at the present state — case (a') if t is a document under its account, case (b') if t is a version. Its spawnPt premise (`t` in its parent allocator's realized domain) is supplied by the minting K.δ event that placed t into its owning allocator, preserved by P1, with the K.δ box's `t ∈ E_doc` conjunct furnishing the membership.

- *k = 2:* the activation case. `e = inc(t, 2)` spawns a *new entity-level sub-allocator* under `t` — `A_account(t)` (t's *account sub-allocator*) when t is a node, `A_doc(t)` (t's *document sub-allocator*) when t is an account, both catalogued in *Sub-allocator names* above — with `t` as spawnPt and `e` as the first emission.

  T10a child-spawn admissibility for this k = 2 descent requires the spawnPt `t` to inhabit `dom(parent_allocator)` at the spawn event — t must already lie in the realized domain of whatever sits *above* the newly-spawned allocator in the allocator tree, namely the source that minted `t`. The spawnPt premise `t ∈ dom(parent_allocator)` is supplied by one of three sources according to `t`'s level; the remainder of the discharge is uniform across all three:

  | `t`'s level | spawned allocator (parent above) | spawnPt-premise source |
  |-------------|----------------------------------|------------------------|
  | account (`zeros(t) = 1`) | `A_doc(t)` (parent `A_account(parent(t))`) | ParentAllocatorDispatch account-level identification (`t ∈ dom(A_account(parent(t)))`); membership preserved to the present event by AllocatedSet monotonicity |
  | non-bootstrap node | `A_account(t)` (no parent allocator: `t` is its base) | NodeBaptism: the boundary membership `t ∈ Σ.E_node` stands in for a parent-allocator membership |
  | bootstrap node `n₀` | `A_account(n₀)` (no parent allocator: `n₀` is its base) | NodeBaptism: `n₀ ∈ E₀` is the boundary baptism of `Σ₀`, supplying the premise with no prior K.δ event |

  With the spawnPt premise supplied, the rest discharges identically across all three rows: spawn admissibility is established at the K.δ box (k = 2 sub-case), and `e` enters as the spawned allocator's first emission. The parent allocator on which distinctness is thereafter preserved is the activated `A_account(t)` (base `t`) for the node rows and `A_account(parent(t))` for the account row.


## Generalized referential integrity

**S3★ (GeneralizedReferentialIntegrity).** The arrangement maps V-positions to addresses in the store appropriate to their subspace:

  `(A d, v : v ∈ dom(Σ.M(d)) : (subspace(v) = s_C ⟹ Σ.M(d)(v) ∈ dom(Σ.C)) ∧ (subspace(v) = s_L ⟹ Σ.M(d)(v) ∈ dom(Σ.L)))`

where `subspace(v)` denotes the first component of the V-position. S3★ supersedes S3 (ASN-0036) for the extended state Σ = (C, L, E, M, R): S3 requires every V-position to map into dom(C), which is violated by link-subspace mappings targeting dom(L). S3 remains valid when restricted to states with no link-subspace mappings — the four-component model of the prior sections has only content-subspace V-positions, for which S3★ reduces to S3.

**S3★-aux (SubspaceExhaustiveness).** In every reachable state, all V-positions have subspace s_C or s_L:

  `(A d, v : v ∈ dom(M(d)) : subspace(v) = s_C ∨ subspace(v) = s_L)`


## Link-subspace extension

**K.μ⁺_L (LinkSubspaceExtension).** Extends a document's arrangement in the link subspace.

*Precondition:*
- d ∈ E_doc
- ℓ ∈ dom(L)  (the target link must already exist in dom(L) — placed there by some prior K.λ)
- origin(ℓ) = d  (only home-document links may be arranged)
- ℓ ∉ ran(M(d))  (the link is not already arranged at any V-position in d's arrangement — first-arrangement constraint)
- V-position v_ℓ satisfies:
  - subspace(v_ℓ) = s_L
  - If V_{s_L}(d) = ∅ (first link-subspace insertion): a free depth parameter `m ≥ 2` is supplied by the caller, and `v_ℓ = [s_L, 1, ..., 1]`, the minimum position of depth `m` (D-MIN★, S8a). We write `ValidFirstLinkPosition(d, v_ℓ, m)` for this condition — the link-subspace analog of ASN-0036's `ValidFirstInsertionPosition(d, v, m)`: for any chosen `m ≥ 2` it fixes the unique well-formed first link V-position `v_ℓ = [s_L, 1, ..., 1]` of depth `m` (`subspace(v_ℓ) = s_L`, `#v_ℓ = m`, `zeros(v_ℓ) = 0`, all components positive). This insertion pins the link-subspace depth as `m_L(d) := m` for every subsequent insertion. The depth is thus a checkable input, not a quantity read from the empty arrangement.
  - If V_{s_L}(d) ≠ ∅: `#v_ℓ = m_L(d)`, the established link-subspace depth (S8-depth within the link subspace), and `v_ℓ = shift(max(V_{s_L}(d)), 1)`, extending the contiguous range (D-CTG★). OrdShiftHom (ASN-0036, clause (a) subspace preservation under shift and clause (b) S8a preservation under shift) is subspace-parametric in v₁, so it applies to v_ℓ at v₁ = s_L exactly as at v₁ = s_C; clause (a) supplies subspace(v_ℓ) = s_L, clause (b) supplies S8a preservation, and OrdinalShift's length-preservation postcondition `#shift(v, n) = #v` (ASN-0034) together with S8-depth supplies S8-depth preservation (`#v_ℓ = #max(V_{s_L}(d)) = m_L(d)`).

*Effect:* `M'(d) = M(d) ∪ {v_ℓ ↦ ℓ}`, with `dom(M'(d)) = dom(M(d)) ∪ {v_ℓ} ⊃ dom(M(d))` (strict extension; the disjointness `v_ℓ ∉ dom(M(d))` discharging the strict inequality is verified immediately below).

*Frame:* `C' = C; L' = L; E' = E; (A d' : d' ≠ d : M'(d') = M(d')); R' = R`

We verify `v_ℓ ∉ dom(M(d))`, as required for M'(d) to be a proper extension preserving S2 (ArrangementFunctionality) — and equivalently to discharge the strict-extension `⊃` in the effect clause above. The verification decomposes by subspace: by S3★-aux (SubspaceExhaustiveness), every `v ∈ dom(M(d))` satisfies `subspace(v) ∈ {s_C, s_L}`, so `dom(M(d)) = V_{s_C}(d) ∪ V_{s_L}(d)`. We discharge `v_ℓ ∉ V_{s_L}(d)` and `v_ℓ ∉ V_{s_C}(d)` separately.

(a) *Disjointness from link-subspace positions, `v_ℓ ∉ V_{s_L}(d)`.* Two cases:
 - *V_{s_L}(d) = ∅:* vacuously, `v_ℓ ∉ ∅`.
 - *V_{s_L}(d) ≠ ∅:* `v_ℓ = shift(max(V_{s_L}(d)), 1) > max(V_{s_L}(d))` by TS4 (ShiftStrictIncrease, ASN-0034), so v_ℓ strictly exceeds every member of V_{s_L}(d) and cannot equal any of them.

(b) *Disjointness from content-subspace positions, `v_ℓ ∉ V_{s_C}(d)`.* By construction `subspace(v_ℓ) = s_L`, while every `v ∈ V_{s_C}(d)` has `subspace(v) = s_C` (by definition of `V_{s_C}(d)`). Since `s_L ≠ s_C` (SC-NEQ), the two tumblers differ in their first component. By T3 (CanonicalRepresentation, ASN-0034), tumblers are extensionally identified by their component sequence — two tumblers differing in any component (and a fortiori in their first component) are distinct — so `v_ℓ ∉ V_{s_C}(d)`.

Combining (a) and (b) with S3★-aux's disjoint-union form: `v_ℓ ∉ V_{s_L}(d) ∪ V_{s_C}(d) = dom(M(d))`.

The preconditions ensure that after the extension, D-CTG★ (contiguity), D-MIN★ (minimum position), and S8-depth (uniform depth) hold for the link subspace of d. S3★ is satisfied: `subspace(v_ℓ) = s_L` and `M'(d)(v_ℓ) = ℓ ∈ dom(L')`.

The origin restriction `origin(ℓ) = d` distinguishes link-subspace extension from content-subspace extension, where K.μ⁺ intentionally permits `origin(a) ≠ d` — content transclusion, an established architectural feature. Link transclusion — arranging a foreign-origin link in a document's link subspace — is excluded by design: a link's home document is what carries its ownership ("A link need not point anywhere in its home document. Its home document indicates who owns it," LM 4/12), so arranging a link with `origin(ℓ) ≠ d` would place an out-link in a document that does not own it. The byte stream admits transclusion; links do not. K.μ⁺_L enforces `origin(ℓ) = d` structurally, as a precondition.


## Link-subspace ownership

**CL-OWN (LinkSubspaceOwnership).** In every reachable state:

  `(A d, v : v ∈ dom(M(d)) ∧ subspace(v) = s_L : origin(M(d)(v)) = d)`

Every document's link-subspace arrangement contains only its own links. K.μ⁺_L's precondition `origin(ℓ) = d` ensures ownership at creation; link-subspace fixity under K.μ~ ensures preservation through reordering.

**CL-UNIQ (LinkSubspacePositionUniqueness).** Within each document's link-subspace arrangement, each link occupies exactly one V-position — the restriction of M(d) to dom_L is injective:

  `(A d, v₁, v₂ : v₁ ∈ dom(M(d)) ∧ v₂ ∈ dom(M(d)) ∧ subspace(v₁) = s_L ∧ subspace(v₂) = s_L ∧ M(d)(v₁) = M(d)(v₂) : v₁ = v₂)`

Equivalently, `M(d)|_{dom_L}` is a partial injection from V-positions to link addresses.


## Decomposition of K.μ~

K.μ~ — *arrangement reordering* — is a **named composite** of K.μ⁻ + K.μ⁺ (not a primitive transition). This section gives its preconditions and realisation.

**Full-clearance form (canonical statement).** K.μ~ is realised in the *full-clearance form* (`n'_{s_C} = 0`): K.μ⁻ clears the entire content subspace — content-only removal — while retaining every link-subspace position pointwise, and K.μ⁺ rebuilds the content subspace at fresh positions, framing the retained link positions. For a given π the concrete realisation is the K.μ⁺ write set `{π(v) ↦ M(d)(v) : v ∈ V_{s_C}(d)}`; the retention of link-subspace mappings under the clearance is the clause-(v) discharge (Step (A), Case `s_L`). This form realises *every* admissible π without per-π precondition checks, since K.μ⁻'s suffix-removal precondition holds vacuously at the full-subspace suffix and K.μ⁺ writes at fresh positions. Steps (A)–(B) below establish this — Step (A) that the realised π is subspace-preserving and link-subspace fixing (hence admissible), Step (B) that the post-state referential invariant S3★(Σ') holds for every realisable π.

**LRP (full-clearance preserves the link subspace pointwise).** K.μ⁻'s content-only clearance leaves every link-subspace V-position in place with its value, and K.μ⁺ writes only content-subspace V-positions (its content-subspace amendment), so no link-subspace position is added, removed, or altered. Hence

  `dom_L(M'(d)) = dom_L(M(d))`  and  `M'(d)|_{dom_L} = M(d)|_{dom_L}`.

*Proof.* Write `M_int(d)` for the intermediate arrangement of `d` between the K.μ⁻ and K.μ⁺ atomic steps. For each `v ∈ dom_L(M(d))`: `v` survives K.μ⁻'s content-only clearance (the removal set is the content suffix, disjoint from `dom_L`), so `v ∈ dom(M_int(d))` with `M_int(d)(v) = M(d)(v)` by K.μ⁻'s value-preservation on survivors; K.μ⁺ frames existing positions, so `M'(d)(v) = M_int(d)(v) = M(d)(v)`. Conversely, K.μ⁺'s content-subspace amendment forbids any link-subspace write, so `dom_L(M'(d)) ⊆ dom_L(M_int(d)) = dom_L(M(d))`. The two inclusions give the stated functional and domain identities. ∎

*Preconditions of K.μ~.* The operation has two explicit preconditions:
- `d ∈ E_doc`
- `M(d)|_{dom_C(M(d))}` takes at least two distinct values (equivalently, `M(d)` restricted to `dom_C(M(d))` is not constant-valued; this entails `|dom_C(M(d))| ≥ 2` but is strictly stronger) — necessary and sufficient for clause (ii) (net effect `M'(d) ≠ M(d)`) to admit a witness (see *Necessity and sufficiency of the precondition* below).

For `d ∈ E_doc` with `M(d)|_{dom_C}` taking at least two distinct values, K.μ~ realises the *bijection equation*:

  `(E π : π is a bijection dom(M(d)) → dom(M'(d)) : (A v ∈ dom(M(d)) :: M'(d)(π(v)) = M(d)(v)))`

π is admissible iff (i) the induced post-state `M'(d)` would satisfy the arrangement-*shape* invariant package on `M'(d)` — S8a, S8-depth, D-CTG★, D-MIN★, from which the derived D-SEQ★ follows (D-SEQ★ at Σ' and S8-fin(Σ') are discharged in the Class (a) paragraph *K.μ~ discharge for the arrangement-shape invariants* below); this package constrains which V-position *domains* exist, not which I-address each position carries — (ii) the net effect is non-trivial, `M'(d) ≠ M(d)`; (iii) π is *length-preserving*: `(A v ∈ dom(M(d)) :: #π(v) = #v)`; (iv) π is *subspace-preserving*: `(A v ∈ dom(M(d)) :: subspace(π(v)) = subspace(v))`; and (v) π is *link-subspace fixing*: `(A v ∈ dom_L(M(d)) :: π(v) = v)`. Admissibility is the full conjunction (i)–(v). The two referential obligations outside clause (i) are discharged separately: S3★ (each content V-position's image lies in `dom(C)`, each link V-position's in `dom(L)`) by Step (B), and S8★ (per-subspace span decomposition) by the inherited K.μ⁺ and K.μ⁻ S8★ columns (the *S8★ (Per-subspace span decomposition)* matrix entry in *Extended reachable-state invariants*): its content subspace via ASN-0036's S8, its link subspace via the length-1 decomposition. Clause (iv) excludes cross-subspace permutations, which would relocate a `dom(C)` value to an `s_L` position and break S3★. Clause (iii) confines K.μ~ to a permutation of the document's existing V-positions *at fixed depth*: a permutation that relocated a V-position to a new depth would not be length-preserving. Clause (iii) thus yields per-subspace depth fixity, from which K.μ~-FIX (Domain fixity) below follows. Clause (ii) makes K.μ~ a real reordering: a permutation whose net effect is the identity arrangement is not a K.μ~ transition (the system simply does not change). Note that `M'(d) ≠ M(d)` is strictly stronger than the map-level `π ≠ id`: under S5 (UnrestrictedSharing, ASN-0036) two distinct content V-positions may carry the same I-address (transclusion), so the swap of two such equal-valued positions is a non-identity *map* with net-identity *effect* — clause (ii)'s net-effect form excludes it.

*Step (A) — The full-clearance decomposition produces a subspace-preserving, link-fixing π.* We show the π produced by the K.μ⁻ + K.μ⁺ full-clearance decomposition is subspace-preserving and link-subspace fixing — hence admissible. S3★-aux(Σ) confines every source subspace to `{s_C, s_L}`, so the two cases below — content positions, which go to K.μ⁺'s write set, and link positions, which LRP fixes pointwise — are exhaustive.

*Case `subspace(v) = s_C`.* The image `π(v)` is a position K.μ⁺ writes. K.μ⁺'s content-subspace amendment precondition admits only content-subspace write positions, so `subspace(π(v)) = s_C = subspace(v)`. This precondition is checked on the realisation; it *is* content-subspace preservation, not an inference from the post-state.

*Case `subspace(v) = s_L`.* By LRP the link-subspace positions of `M'(d)` are exactly those of `M(d)`, unchanged, with `M'(d)|_{dom_L} = M(d)|_{dom_L}`, while the content-subspace positions of `M'(d)` are exactly K.μ⁺'s write set, which the content sources of the preceding case already exhaust bijectively. The link sources therefore map onto the retained link positions, so `π(v) ∈ dom_L(M'(d)) = dom_L(M(d))` and `subspace(π(v)) = s_L = subspace(v)`. Pointwise link fixity (clause (v), `π(v) = v`) follows here at its first use: for `v ∈ dom_L(M(d))` write `M(d)(v) = ℓ`; subspace preservation places `π(v) ∈ dom_L(M(d))`, and the bijection equation together with LRP gives `M(d)(π(v)) = M'(d)(π(v)) = M(d)(v) = ℓ`, so `v` and `π(v)` are link-subspace V-positions in `dom(M(d))` mapping to the same link `ℓ`. CL-UNIQ at Σ — link-subspace injectivity of `M(d)|_{dom_L}`, supplied as the ExtendedReachableStateInvariants per-state hypothesis at the reachable pre-state Σ — forces `π(v) = v`. Post-state CL-UNIQ follows from the same functional identity: `M'(d)|_{dom_L} = M(d)|_{dom_L}` (LRP) with `M(d)|_{dom_L}` injective (CL-UNIQ at Σ) gives `M'(d)|_{dom_L}` injective, i.e. CL-UNIQ at Σ'.

Thus the realised π preserves subspace and fixes the link subspace pointwise — hence is admissible. ∎ (Step (A))

*Proof of Step (B) — mechanical realisability of any admissible π (result **K.μ~-S3★**: the realisation establishes `S3★(Σ')`).* This step *establishes* `S3★(Σ')` at the post-state `Σ' = (C, L, E, M', R)` from the two Class (a) cells. The realisable π are precisely those whose content images satisfy K.μ⁺'s content-subspace amendment precondition — equivalently (Step (A)), the subspace-preserving ones — so K.μ⁺'s amendment fires on the full-clearance write set by realisability. Let `Σ_int = (C, L, E, M_int, R)` denote the intermediate state between the K.μ⁻ and K.μ⁺ atomic steps of the decomposition. Three sub-claims, dispatched in order:

  *(B.1) S3★(Σ_int).* K.μ⁻'s effect is `M_int(d) = M(d) ↾ R'` (restriction to a subset of `dom(M(d))`) with values unchanged on survivors, and `M_int(d') = M(d')` for every `d' ≠ d`; K.μ⁻'s frame on `C` and `L` gives `dom(C_int) = dom(C)` and `dom(L_int) = dom(L)`. For every `v ∈ dom(M_int(d))`: `v ∈ dom(M(d))` (survivors lie in the pre-state domain), and `M_int(d)(v) = M(d)(v)` (value-preservation on survivors); the subspace `subspace(v)` is a property of the V-position tumbler itself, unchanged by the restriction. By S3★(Σ) (inductive hypothesis at the pre-state) at `v`: when `subspace(v) = s_C`, `M(d)(v) ∈ dom(C)`; when `subspace(v) = s_L`, `M(d)(v) ∈ dom(L)`. Substituting `M_int(d)(v) = M(d)(v)` and `dom(C) = dom(C_int)`, `dom(L) = dom(L_int)`: when `subspace(v) = s_C`, `M_int(d)(v) ∈ dom(C_int)`; when `subspace(v) = s_L`, `M_int(d)(v) ∈ dom(L_int)`. The same dispatch on every untouched arrangement `M_int(d')` for `d' ≠ d` gives S3★(Σ_int) globally.

  *(B.2) K.μ⁺'s new content-subspace positions target dom(C).* The K.μ⁺ step in the K.μ~ decomposition (full-clearance form) writes `{π(v) ↦ M(d)(v) : v ∈ V_{s_C}(d)}` at fresh V-positions disjoint from `dom(M_int(d))`. Realisability of π is exactly K.μ⁺'s content-subspace amendment precondition on this write set — each `π(v)` for `v ∈ V_{s_C}(d)` is a content-subspace position (Step (A)) — so the amendment (new positions have `subspace = s_C`) is satisfied on the entire write set by construction. By K.μ⁺'s referential-integrity precondition `M'(d)(v) ∈ dom(C')` for every new content-subspace mapping (evaluated at the post-K.μ⁻ intermediate state, which `Σ_int` makes available): each newly written value lies in `dom(C_int) = dom(C')` (`C' = C_int = C` by frame across both atomic steps).

  *(B.3) S3★(Σ').* B.1 (survivors framed by K.μ⁺ carry S3★ from S3★(Σ_int)) and B.2 (new content positions target `dom(C')`) jointly establish S3★(Σ') over `dom(M'(d))`; framed arrangements `M'(d') = M(d')` for `d' ≠ d` carry S3★ from the pre-state. ∎ (Step (B))

**K.μ~-FIX (Domain fixity).** `dom(M'(d)) = dom(M(d))`. D-SEQ★ at the pre- and post-states gives `V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` at common depth `m_S` (S8-depth at Σ) and `V_S(d') = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S}` at common depth `m'_S` (S8-depth at Σ') for each subspace S. Since π is a bijection and (by subspace preservation) bijects V_S(d) onto V_S(d'), `n'_S = n_S` (equal cardinality). Length preservation (admissibility (iii)) gives `#π(v) = #v = m_S` for every `v ∈ V_S(d)`, so every post-state position `π(v) ∈ V_S(d')` has depth `m_S`; by S8-depth at Σ', the common depth satisfies `m'_S = m_S`. With both the length `n'_S = n_S` and the depth `m_S = m'_S` equal, the two canonical sequences coincide componentwise: `V_S(d') = V_S(d)`. Taking the union over subspaces S, `dom(M'(d)) = dom(M(d))`, so π is a permutation of dom(M(d)).

**K.μ~-RANGE (range-invariance).** K.μ~ does *not* preserve per-position values on dom_C — reassigning which V-position carries which I-address is the whole point of reordering, `M'(d)(π(v)) = M(d)(v)`. What it preserves is the *range*, the set of content I-addresses: `ran(M'(d)) = ran(M(d))`, and consequently `Contains_C(Σ') = Contains_C(Σ)` and `Contains(Σ') = Contains(Σ)`. *Proof.* By K.μ~-FIX, π is a bijection `dom(M(d)) → dom(M'(d))` with `dom(M'(d)) = dom(M(d))`, and the bijection equation `M'(d)(π(v)) = M(d)(v)` holds for every `v ∈ dom(M(d))`; as π ranges over the full domain, the image set `{M'(d)(π(v)) : v} = {M(d)(v) : v}` gives `ran(M'(d)) = ran(M(d))`. Subspace preservation (Step (A)) splits this identity along the `{s_C, s_L}` partition, giving in particular the content-subspace range equality `ran(M'(d)|_{dom_C}) = ran(M(d)|_{dom_C})` — the quantity `Contains_C` consumes (the link half is fixed pointwise by LRP, `M'(d)|_{dom_L} = M(d)|_{dom_L}`). K.μ~ frames `C`, `E`, and every arrangement other than `d`'s (Frame, below), so the document-content containment sets `Contains_C` and `Contains` are unchanged at the boundary. In particular `ran(M'(d)) \ ran(M(d))` is empty, so the J1★ coupling has no new containment pairs to record. ∎

*Necessity and sufficiency of the precondition.* The precondition is stated against the pre-state `Σ` of a K.μ~ event. Necessity assumes π admissible, so subspace-preservation (iv) and link-fixity (v) both enter as hypotheses — (v) established in Step (A), Case `s_L`. CL-UNIQ at `Σ` holds by the ExtendedReachableStateInvariants inductive hypothesis on the per-state invariants at the reachable pre-state.
  - *Necessity (universal closure over admissible π).* Suppose K.μ~ admits some π. Then π satisfies admissibility (i) (the arrangement-shape package on M'(d)) and admissibility (ii) (net effect `M'(d) ≠ M(d)`); S3★(Σ') holds as a derived consequence (Step (B)). By admissibility (v) (link-fixity, a hypothesis here) together with K.μ~-FIX (`dom(M'(d)) = dom(M(d))`), π fixes `dom_L(M(d))` pointwise and preserves each subspace, so it maps `dom_C(M(d))` bijectively onto itself; the net change `M'(d) ≠ M(d)` therefore cannot lie in the link subspace and must lie in the content subspace, `M'(d)|_{dom_C} ≠ M(d)|_{dom_C}`. The remaining step is new: suppose for contradiction `M(d)|_{dom_C}` were constant with common value `a`. Content-subspace closure places `π⁻¹(u) ∈ dom_C(M(d))` for every `u ∈ dom_C(M(d))`, so the bijection equation gives `M'(d)(u) = M(d)(π⁻¹(u)) = a = M(d)(u)` — whence `M'(d)|_{dom_C} = M(d)|_{dom_C}`, contradicting clause (ii). Therefore `M(d)|_{dom_C}` is non-constant: it takes at least two distinct values. ∎
  - *Sufficiency (existence witness).* When `M(d)|_{dom_C}` takes at least two distinct values, an admissible π exists. Pick `v₁, v₂ ∈ dom_C(M(d))` with `M(d)(v₁) ≠ M(d)(v₂)` (value-distinctness forces `v₁ ≠ v₂`) and take the *transposition witness* `π_swap` that swaps `v₁ ↔ v₂` and fixes every other position of `dom(M(d))`. The one non-trivial check is clause (ii): the bijection equation at `v₂` gives `M'(d)(v₁) = M(d)(v₂) ≠ M(d)(v₁)`, so `M'(d) ≠ M(d)`. The remaining clauses hold directly: (iv) and (v) because `π_swap` moves only the two content positions `v₁, v₂` and fixes all of `dom_L`; (iii) because `v₁, v₂` share the content-subspace depth `m_{s_C}` (S8-depth at Σ); (i) because `π_swap` permutes `dom(M(d))`, so `dom(M'(d)) = dom(M(d))` and the shape package inherits from the pre-state. The full-clearance form realises `π_swap`, and the post-state obligation S3★(Σ') follows from the general Step (B) derivation. A single witness suffices.

  The precondition is caller-checked: a transition whose `M(d)|_{dom_C}` is constant-valued (in particular when `|dom_C(M(d))| ≤ 1`, but also any state in which every content V-position shares a single I-address by transclusion) does not fire, and the caller is responsible for verifying this condition before invoking K.μ~.

*Frame (derived).* C' = C; E' = E; R' = R; L' = L; (A d' : d' ≠ d : M'(d') = M(d')) — by composition of K.μ⁻ and K.μ⁺ frames.

*Intermediate-state admissibility.* At Σ_int (post-K.μ⁻, pre-K.μ⁺): C_int = C, M_int(d) = M(d) ↾ V_{s_L}(d). K.μ⁺'s preconditions at Σ_int discharge: `d ∈ E_doc` (frame); referential integrity from `M(d)(v) ∈ dom(C)` for `v ∈ V_{s_C}(d)` at pre-state; content-subspace restriction from K.μ~'s subspace-preserving precondition; S8a/S8-depth/S8-fin/D-CTG★/D-MIN★ on M'(d) from K.μ~'s postcondition. S2 holds because π is a bijection: K.μ⁻'s (full-clearance) restriction preserves single-valuedness on the link-subspace survivors, and K.μ⁺ adds content-subspace positions at fresh positions disjoint from the intermediate domain by its value-preservation clause, so no V-position acquires a second image.

## Coupling and isolation

The elementary transitions do not all occur independently. Some must co-occur to maintain invariants (coupling); some must leave other components unchanged (isolation). The weakest-precondition calculus makes the coupling constraints visible.

The K.ρ/K.μ⁺ coupling is range-based, not unconditional — it triggers only when K.μ⁺ adds an I-address new to the document's content-subspace range; see J1★ in *Scoped coupling constraints* below.

**Definition (Current containment).** The *current containment* of state Σ is the set of all document-content pairs where the content is presently in the document's arrangement:

`Contains(Σ) = {(a, d) : d ∈ E_doc ∧ a ∈ ran(M(d))}`

This is a derived quantity of the state — it captures what each document currently displays.

**Contains_C(Σ) (ContentContainment).**

  `Contains_C(Σ) = {(a, d) : d ∈ E_doc ∧ (E v : v ∈ dom(M(d)) ∧ subspace(v) = s_C : M(d)(v) = a)}`

**P4★ (ProvenanceBounds, content-subspace).**

  `Contains_C(Σ) ⊆ R`

P4★ bounds provenance by the *content-subspace* restriction of containment (scoped to the content subspace so it coexists with P7).

The ordering matters: J0 couples K.α with K.μ⁺, and S3 requires the I-address to exist before the V→I mapping is created, so K.α precedes K.μ⁺. Similarly, J4's fork compounds K.δ + K.μ⁺ + K.ρ, and K.μ⁺ requires d ∈ E_doc, which K.δ establishes — so K.δ precedes K.μ⁺. The net effect of a composite transition is the composition of its elementary effects.

For freshly created documents d ∈ E'_doc \ E_doc, the pre-state has d ∉ E_doc, so M(d) = ∅ by the default-value convention (Bridging lemma (M–E_doc)). Consequently ran(M(d)) = ∅, and the set difference ran(M'(d)) \ ran(M(d)) reduces to ran(M'(d)): all content placed in a new document counts as newly introduced. The coupling constraints below quantify over E'_doc, not E_doc, making them applicable to freshly created documents without special cases.

**J0 (Allocation requires placement).** Content allocation K.α always co-occurs with arrangement extension K.μ⁺:

`(A Σ →* Σ', a : a ∈ dom(C') \ dom(C) : (E d, v : d ∈ E'_doc ∧ v ∈ dom(M'(d)) : M'(d)(v) = a))`

Every freshly allocated I-address appears in some arrangement in the post-state — the containing document may itself have been freshly created by K.δ in the same composite transition.

**P4a (Trace witnessing — trace property).** A *valid transition trace to Σ* is a finite sequence of composite boundaries `Σ₀ →* Σ₁ →* ... →* Σ_n = Σ` in which each `Σ_j →* Σ_{j+1}` is a valid composite transition (ValidComposite★); the finite set of states `{Σ₀, ..., Σ_n}` is the *transition history* of Σ along that trace, and `M_k` denotes the arrangement family of trace state `Σ_k`. P4a asserts that every `(a, d) ∈ R` had a *content-subspace* containment witness in *some* trace state — the content `a` was arranged in `d` at some point in the document's history, not necessarily at the moment the entry was recorded:

`(A valid trace Σ₀ →* ... →* Σ_n = Σ :: (A (a, d) ∈ R :: (E Σ_k ∈ {Σ₀, ..., Σ_n} : (E v ∈ dom(M_k(d)) : subspace(v) = s_C ∧ M_k(d)(v) = a))))`

The witnessing existential ranges over the finite set `{Σ₀, ..., Σ_n}` of trace states. Provenance rides on the permanent I-address and survives deletion from the current arrangement, so the witness is whatever historical version contained the content, not a live moment-of-recording check (Nelson, LM 4/9, 4/11 — included content "may remain included in other versions" after deletion from the current one). The content-subspace qualification is essential: no link-subspace V-position can witness provenance (J-LV(ii), *Scoped coupling constraints* below).

**J2 (Contraction isolation).** The elementary transition K.μ⁻ requires no coupling — it is self-sufficient with respect to P0–P2, L12, and the operative provenance bound P4★ (defined above). As an elementary transition, K.μ⁻ satisfies:

`C' = C ∧ L' = L ∧ E' = E ∧ R' = R`

The frame line above discharges P0, P1, P2, and L12 directly: each constrains a store (C, E, R, L) that K.μ⁻ leaves fixed. The only non-trivial case is the operative provenance bound P4★, where the frame does not settle the obligation because P4★ couples R against the live content set Contains_C, and K.μ⁻ does change M(d). Contraction can only remove pairs from Contains_C and frames R, so `Contains_C(Σ') ⊆ Contains_C(Σ)` with `R' = R`. Hence K.μ⁻ introduces no new P4★ violation relative to Σ — whatever P4★'s status at the (possibly intermediate) pre-state Σ, the post-state's operative-content set is a subset of the pre-state's against an unchanged range, so any P4★ membership obligation already discharged at Σ remains discharged at Σ'. No co-occurring transition is needed to maintain any system invariant.

Deletion is purely presentational — it changes what appears, not what exists or what has been. Gregory confirms: contraction "never triggers" provenance recording, and the provenance structure "is never pruned."

**J3 (Reordering isolation).** The named composite K.μ~ is likewise self-sufficient:

`C' = C ∧ L' = L ∧ E' = E ∧ R' = R`

By **K.μ~-RANGE** (range-invariance), `Contains(Σ') = Contains(Σ)`, and J1★ is vacuous since no range-new content arises across the composite. All invariants are trivially maintained.

**J4 (Fork composite).** Nelson's forking creation mode — version creation with ancestry indication (LM 4/66, CREATENEWVERSION) — is a composite whose elementary steps are exactly K.δ + K.μ⁺ + K.ρ, all serving the new document d_new. Its K.δ allocation discipline, content-source operand-tracking rule, and the checkable discriminator separating a fork from an independent sibling-document allocation are given in Definition (Fork) below. In every case d_new extends d_src's document-field prefix (d_src ≼ d_new), so d_new carries the ancestry-by-address indication. This matches Gregory's `docreatenewversion`, which performs the identical `findisatoinsertnonmolecule` allocation for every version (the first allocation produces `d_src.1`, subsequent allocations advance the last component to `d_src.k+1`), special-casing neither. By contrast, the k = 0 *sibling-document* allocation under the source's account (`docreatenewdocument`) and the k = 2 hierarchical descent are *not* forks: their outputs do not extend d_src's prefix and carry no ancestry-by-address indication.

**Definition (Fork).** A *fork* of d_src to d_new is a composite transition `Σ →* Σ'`. Write `d_op` for the *content source operand* of the fork — the operand `t` of the K.δ step (i), whose current content subspace is transcluded into d_new. **Allocation and operand-tracking rule.** Fork is version creation on d_src's version chain `A_v(d_src)`, with K.δ address allocation uniform across first and subsequent versions; the content source operand tracks the same K.δ sub-case. The two sub-cases are:
- the k = 1 sub-case fires when `A_v(d_src)` has no prior emission (`dom(A_v(d_src)) = ∅`, no version yet exists): the allocated version is `d_new = inc(d_src, 1)` and the content source is `d_op = d_src` — the base of d_src's version sub-allocator;
- the k = 0 sub-case fires when `A_v(d_src)` already has a frontier (`dom(A_v(d_src)) ≠ ∅`, a prior version exists): the allocated version is `d_new = inc(prev_version, 0)` on `A_v(d_src)`'s frontier, and the content source is `d_op = prev_version = max(dom(A_v(d_src)))` — the frontier element of d_src's version sub-allocator, so the formal conjunct `d_op ∈ dom(A_v(d_src))` holds. A subsequent (k = 0) version thus inherits the *current edited content* of the prior version it forks from, capturing edits absent from the base d_src. This conjunct is the checkable discriminator: a k = 0 *fork* requires `d_op ∈ dom(A_v(d_src))`, whereas an independent sibling-document allocation takes its k = 0 operand from `A_doc(parent(d_src))`'s frontier (`d_op ∈ dom(A_doc(parent(d_src)))`, hence `d_op ∉ dom(A_v(d_src))`) and is not a fork.

The *precondition* is d_src ∈ E_doc ∧ d_op ∈ E_doc ∧ V_{s_C}(d_op) ≠ ∅, together with the per-sub-case activation condition on the K.δ step stated above, which also fixes `d_op` and supplies the structural discriminator separating a fork from an independent sibling-document creation. The content-source clause `V_{s_C}(d_op) ≠ ∅` is what makes this a fork: creation from a source with an empty content subspace is ex nihilo (K.δ alone), not a fork.

It consists of:

(i) a K.δ case (ii) step producing d_new on d_src's version chain `A_v(d_src)`, with d_new ∉ E_doc, allocating d_new and fixing the operand `t = d_op` per Definition (Fork)'s allocation and operand-tracking rule; in both sub-cases d_src ≼ d_new,

(ii) K.μ⁺ populating M'(d_new) by a *complete, order- and multiplicity-preserving* transclusion of `d_op`'s content subspace. Let `φ : V_{s_C}(d_op) → V_{s_C}(d_new)` be the unique order-preserving bijection between the two content-subspace V-position sets. Both are D-SEQ★-sequential — `{[s_C, 1, …, 1, k] : 1 ≤ k ≤ n}`, differing only in depth (d_new's content subspace is freshly rebased to its own depth `m_new`) — so the order-preserving bijection exists, is unique, and maps the k-th source position to the k-th target position. The K.μ⁺ step establishes

  `(A v ∈ V_{s_C}(d_op) :: M'(d_new)(φ(v)) = M(d_op)(v))`.

This *depth-rebasing bijection* is the characterization of the fork, and it captures both properties a version copy demands. **Order preservation**: `φ` is order-preserving, so the relative V-order of the source content is reproduced in d_new — a fork cannot place the source's content in a permuted order. **Multiplicity preservation**: `φ` is injective, so two distinct source V-positions carrying the *same* I-address — as S5 (UnrestrictedSharing, ASN-0036) permits, e.g. `M(d_op)([s_C,1]) = M(d_op)([s_C,2]) = a` — map to two distinct target V-positions, each retaining that I-address; no duplicate is collapsed and the document's content count is preserved. Range equality is now a *derived consequence*, not the characterization: `ran(M'(d_new)) = {M'(d_new)(φ(v)) : v ∈ V_{s_C}(d_op)} = {M(d_op)(v) : v ∈ V_{s_C}(d_op)} = ran(M(d_op)|_{V_{s_C}(d_op)})` — every content I-address of the source is inherited at a corresponding position, and no new content addresses are introduced, so every target lies in the pre-existing content store. The content source operand `d_op` is fixed by Definition (Fork)'s operand-tracking rule, so for a subsequent (k = 0) version the edit-inheritance noted there holds. Nelson's CREATENEWVERSION "creates a new document with the contents of document `<doc id>`" (LM 4/66) — the *whole* current arrangement in its source order, never a deliberate proper subset and never a reordering or collapse (partial inclusion is a separate mechanism: COPY/quote-links/versioning-by-inclusion, LM 4/67); and Gregory's `docreatenewversion` copies the source document's entire content subspace in source order — including all prior INSERT/DELETE edits — re-seating each content piece at a fresh sequential V-position, retaining duplicate I-addresses at distinct V-positions as separate entries, and never filtering, reordering, or deduplicating any content-subspace subset. The order is reproduced exactly; only the numeric V-values are re-derived (the depth rebasing of `φ`). Only the *link* subspace is excluded, which J4 reconstructs as `V_{s_L}(d_new) = ∅` below.

(iii) K.ρ recording provenance for each a ∈ ran(M'(d_new)),

and no other elementary steps. Step (ii) must produce a content-subspace arrangement on d_new that is the `φ`-image of d_op's content-subspace arrangement (order- and multiplicity-preserving, with range equality as a consequence) and discharges the per-state arrangement invariants (S2, S3★, S8a, S8-depth, S8-fin, S8★, D-CTG★, D-MIN★, D-SEQ★) at the post-state. The two derived conjuncts hold off the listed shape invariants: D-SEQ★ follows from D-CTG★ + D-MIN★ + S8-depth + S8-fin + S8a on `V_{s_C}(d_new)`, and S8★ follows from ASN-0036's S8 (CorrespondenceRunPartition) applied to the content-subspace projection of M'(d_new), with S8★(s_L) vacuous since `V_{s_L}(d_new) = ∅` (established below). The contiguous choice of V-positions `[s_C, 1, …, 1, k]` from the minimum (used in the invariant discharge below) is exactly what realises `φ` as the order-preserving map: assigning the k-th source position's image to `[s_C, 1, …, 1, k]` makes `φ` order-preserving by construction.

*Discharge of arrangement-side invariants.* Step (ii)'s K.μ⁺ creates only content-subspace V-positions (by the K.μ⁺ amendment) targeting addresses in `ran(M(d_op)|_{V_{s_C}(d_op)}) ⊆ dom(C)` (by S3★'s content clause at the pre-state, which is the only S3★ clause supplying a dom(C) containment — the unrestricted `ran(M(d_op))` includes link-subspace targets in dom(L), which are excluded from dom(C) by L14); C is frame-preserved across the composite (none of K.δ, K.μ⁺, K.ρ modify C), so S3★'s content clause holds at the post-state. *Link-subspace clearance via K.δ initialisation.* Step (i)'s K.δ event places `d_new` into `E_doc`; since `d_new` is a document, K.δ's Document case registers it into `dom(M) = E_doc` with the *explicit* effect `M'(d_new) = ∅` (`dom(M') = dom(M) ∪ {d_new}`, `M'(d_new) = ∅`) — not a totality-convention default. Hence at the intermediate state Σ_post-K.δ, `dom(M(d_new)) = ∅`, and therefore `V_{s_L}(d_new) = ∅`. Step (ii)'s K.μ⁺ then adds only content-subspace V-positions (by the K.μ⁺ amendment), so the link subspace remains empty across step (ii): at the post-K.μ⁺ state, `V_{s_L}(d_new) = ∅` still holds. The fork composite includes no K.μ⁺_L step, so no link-subspace V-position is ever placed in `M(d_new)`. Hence at the fork's post-state Σ', `V_{s_L}(d_new) = ∅`, and D-CTG★, D-MIN★, S8-depth, S8-fin, and S8a all hold vacuously on d_new's link subspace. Step (ii)'s K.μ⁺ must establish D-CTG★, D-MIN★, S8a, S8-depth, S8-fin on `V_{s_C}(d_new)` by its postconditions — the choice of V-positions in step (ii) must be invariant-discharging, but the specific V-positions are operation-specific. By choosing V-positions contiguously from the minimum `[s_C, 1, ..., 1]`, D-CTG★ and D-MIN★ hold for the content subspace of d_new.

*Discharge of coupling constraints under the amended K.μ⁺.* J1★ is satisfied because step (ii)'s K.μ⁺ creates only content-subspace V-positions (by the amendment) and step (iii)'s K.ρ records provenance for each `a ∈ ran(M'(d_new))`, covering every content-subspace extension. J1'★ is satisfied because each new `(a, d_new) ∈ R' \ R` has `a ∈ ran(M'(d_new))` from a content-subspace extension — S3★'s content clause gives `M'(d_new)(v) ∈ dom(C)` for each such v, so `ran(M'(d_new)) ⊆ dom(C)` and P7 compatibility is maintained. Link-subspace mappings carry no provenance (J-LV) and are not copied — the forked document's link subspace starts empty (established above). This is consistent with Nelson's design: each document owns only its home links, and links from the source remain discoverable through the shared I-addresses via refractive following — "a link to one version of a Prismatic Document is a link to all versions" (Nelson).

Since none of K.δ, K.μ⁺, K.ρ modify C (each has C' = C in its frame), a fork satisfies dom(C') = dom(C) — no new content is created. The provenance conclusion — that (a, d_new) ∈ R' for every a ∈ ran(M'(d_new)) — follows from J1★ applied to the fresh-document case: the convention M(d_new) = ∅ gives that every content-subspace mapping in M'(d_new) is new to d_new's content-subspace range, and J1★ directly requires provenance recording for each such address. No additional constraint beyond J1★ is needed.

The new document d_new is created empty (K.δ), its arrangement extended by the φ-image of d_op's content-subspace arrangement (K.μ⁺, with `M'(d_new)(φ(v)) = M(d_op)(v)`, whence `ran(M'(d_new)) = ran(M(d_op)|_{V_{s_C}(d_op)})`), and the new associations recorded (K.ρ). The precondition V_{s_C}(d_op) ≠ ∅ ensures K.μ⁺ is well-formed. Since K.μ⁺ (amended) creates only content-subspace V-positions, the I-addresses it maps to must lie in dom(C) (by S3★'s content clause). Only content-subspace V-positions in d_op have I-addresses in dom(C) — link-subspace V-positions map to dom(L), and dom(L) ∩ dom(C) = ∅ (L14). With V_{s_C}(d_op) ≠ ∅, there is at least one content I-address to transclude, so the strict domain extension dom(M'(d_new)) ⊃ dom(M(d_new)) = ∅ is satisfiable. Nelson: "the new document's id will indicate its ancestry."

An immediate consequence of J1★ and J2 is that the provenance relation diverges from current containment over time. The operative provenance bound is P4★ (defined above); over the link-free fragment (`dom(L) = ∅`) every V-position is content-subspace, `Contains(Σ) = Contains_C(Σ)`, and P4★ reads as the unscoped `Contains(Σ) ⊆ R`.

Every I-address currently in some arrangement is recorded in R. But the converse does not hold: (a, d) ∈ R does not imply a ∈ ran(M(d)). Stale entries persist from earlier states where d contained a before contraction removed it. These entries are not errors — they are the system's historical memory of content associations, monotonically truthful, never retracting a claim once made. Gregory: "find_documents returns historically accurate results, not current state."


## Scoped coupling constraints

**J1★ (ExtensionRecordsProvenance, content-subspace).**

  `(A Σ →* Σ', d ∈ E'_doc, a : (E v ∈ dom(M'(d)) : subspace(v) = s_C ∧ M'(d)(v) = a) ∧ ¬(E v ∈ dom(M(d)) : subspace(v) = s_C ∧ M(d)(v) = a) : (a, d) ∈ R')`

J1★ is range-based: it triggers whenever an I-address `a` is new to the content-subspace range of M'(d), regardless of whether the V-position carrying it existed in dom(M(d)). This is the range-based structure `a ∈ ran(M'(d)) \ ran(M(d))`, scoped to the content subspace.

*Content-boundedness of the content-subspace range (reused below).* By S3★'s content clause — `(A u : u ∈ dom(M(d)) ∧ subspace(u) = s_C : M(d)(u) ∈ dom(C))` — the content-subspace range is content-bounded at every state: `ran(M(d)|_{s_C}) ⊆ dom(C)`. Consequently any `a ∉ dom(C)` is automatically new to the content-subspace range, `a ∉ ran(M(d)|_{s_C})` — the *range-new discharge*.

*Derivation of J1★ from preserving P4★.* The design choice to preserve is P4★ (`Contains_C(Σ) ⊆ R`) — the form that retains Nelson's "every recorded containment is captured in the reverse index" intent in the two-subspace state (scoped per P4★ above). Computing wp backward from `Contains_C(Σ') ⊆ R'` under the K.μ⁺ amendment (which adds only content-subspace V-positions) and the K.μ⁺ frame `R' = R`:

`wp(K.μ⁺ (amended), Contains_C(Σ') ⊆ R') = (A a : a ∈ ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) : (a, d) ∈ R)`

The right-hand side collapses to R because K.μ⁺ frames R, and the difference set on the left is content-subspace-scoped because the K.μ⁺ amendment introduces only content-subspace V-positions, so any new entry in `Contains_C(Σ') \ Contains_C(Σ)` is captured by `ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C})`. The wp computation is vacuous for K.μ⁺_L on P4★ (J-LV(i) below). The requirement "every new content-subspace range entry already in R" is not generally true for fresh content; K.μ⁺ (amended) in isolation cannot maintain P4★. Therefore, to maintain P4★, K.ρ must co-occur within the composite, extending R so that the composite post-state satisfies `(A a : a ∈ ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) : (a, d) ∈ R')` — which is J1★ above.

**J1'★ (ProvenanceRequiresExtension, content-subspace).**

  `(A Σ →* Σ', a, d : (a, d) ∈ R' \ R : (E v ∈ dom(M'(d)) : subspace(v) = s_C ∧ M'(d)(v) = a) ∧ ¬(E v ∈ dom(M(d)) : subspace(v) = s_C ∧ M(d)(v) = a))`

J1'★ is likewise range-based: every new provenance entry `(a, d) ∈ R' \ R` must correspond to an I-address `a` that is new to the content-subspace range — present in the content-subspace range of M'(d) but absent from the content-subspace range of M(d).

*Derivation of J1'★ from preserving P4a.* J1'★ is the *converse* coupling. Its derivation runs wp backward from P4a (Trace witnessing, stated above): `(A (a, d) ∈ R :: (E Σ_k in the transition history :: a is in the content-subspace range of M_k(d)))`. P4a is the requirement that R carry no entry unanchored in some content-subspace containment witness in some trace state (the witnessing state may be the present one). Its preservation burden falls, by R₀ = ∅ and P2-persistence of existing entries, on the transition that *extends* R — K.ρ. Because K.ρ holds M in frame (`M' = M`, so `dom(M'(d)) = dom(M(d))` and `Contains_C(Σ') = Contains_C(Σ)`), the K.ρ-step post-state shares its arrangement with the pre-state, and the *step-local* obligation specialises to "every R-entry this K.ρ step adds has a content-subspace witness in the arrangement at the moment of the step." Computing wp backward from that step-local obligation under K.ρ:

`wp(K.ρ, (A (a, d) ∈ R' \ R :: (E v ∈ dom(M'(d)) : subspace(v) = s_C ∧ M'(d)(v) = a)))`
`= {K.ρ frames M, so M'(d) = M(d)}`
`(A (a, d) ∈ R' \ R :: (E v ∈ dom(M(d)) : subspace(v) = s_C ∧ M(d)(v) = a))`

The right-hand side requires every entry K.ρ adds to already inhabit the content-subspace range of M(d) at the K.ρ pre-state. K.ρ in isolation cannot guarantee this for a freshly recorded provenance entry; therefore, to maintain P4a, the matching K.μ⁺ must co-occur within the composite, placing `a` into the content-subspace range. The aggregate composite obligation — every new entry in `R' \ R` corresponds to a content-subspace range change — is the coupling J1'★ stated above, evaluated at the composite boundary (`Σ → Σ'`) with the witness required at the composite endpoint Σ'.

**J-LV (Link-subspace provenance vacuity).** Link-subspace V-positions neither trigger nor witness provenance. A V-position `v` with `subspace(v) = s_L ≠ s_C` (SC-NEQ) maps to a link address in `dom(L)`, and `dom(L) ∩ dom(C) = ∅` (L14), so `v` never enters the content-subspace range `ran(M'(d)|_{s_C})`. Two consequences follow: (i) *no trigger* — a link-subspace extension (K.μ⁺_L) leaves `ran(M(d)|_{s_C})` unchanged, so the J1★ obligation and the P4★ wp computation are vacuous on it; (ii) *no witness* — no link-subspace V-position can witness a provenance entry under P7's `a ∈ dom(C)` grounding, so P4a's witnessing existential is necessarily content-subspace. P7 (ProvenanceGrounding) — `(A (a, d) ∈ R :: a ∈ dom(C))` — is preserved across K.μ⁺_L because R is held in frame.

**ValidComposite★ (ValidComposite, amended).** A composite transition `Σ →* Σ'` in the extended state Σ = (C, L, E, M, R) is *valid* iff it is a finite sequence of atomic transitions `Σ = Σ₀ → Σ₁ → ... → Σₙ = Σ'` — drawn from the atomic vocabulary K.α (amended), K.δ, K.λ, K.μ⁺ (amended), K.μ⁺_L, K.μ⁻ (amended), and K.ρ — satisfying the clauses below. The named composite K.μ~ is not atomic; it may appear in the sequence as shorthand for its K.μ⁻ + K.μ⁺ decomposition, expanding to those two atomic steps (per clause (1)).

1. *Transition preconditions (intra-composite sequencing).* Each step `Σᵢ → Σᵢ₊₁` satisfies the *elementary* precondition of its transition kind, evaluated at the *intermediate* state `Σᵢ`. K.μ~ appearing in the sequence is shorthand for its K.μ⁻ + K.μ⁺ decomposition (per its definition above): admissibility clause (ii) requires a non-trivial net effect `M'(d) ≠ M(d)`, whose necessary-and-sufficient existence condition is the K.μ~ precondition stated at its definition above. When the existence condition holds, K.μ~ always expands into two consecutive elementary steps, each satisfying its own precondition at the respective intermediate state. This clause is what enforces intra-composite ordering — e.g., that K.α precedes K.μ⁺ when the latter places a freshly allocated I-address `a`, since K.μ⁺'s referential-integrity precondition `a ∈ dom(C)` would fail at the pre-K.α intermediate state otherwise. Step preconditions are *local* to the elementary transition; they say nothing about the composite's endpoints.
2. *Coupling constraints (initial-to-final).* J0, J1★, and J1'★ hold for the composite as a whole — evaluated *only* between the initial state Σ and the final state Σ'. The coupling predicates quantify over the *net* change between Σ and Σ' (e.g., `a ∈ dom(C') \ dom(C)`); they do not constrain the order or shape of intermediate steps, only that the *aggregate* effect of the composite must satisfy them. The couplings J0, J1★, and J1'★ are *imposed* validity conditions, not axioms of the elementary transition system: a composite that satisfies clause (1) but violates clause (2) — for instance, K.α alone without an accompanying K.μ⁺ and K.ρ, every elementary precondition holding at each intermediate state — is not a valid composite.

## Orphan links and coupling flexibility

The coupling constraints do not require K.λ to be paired with K.μ⁺_L. A composite consisting of K.λ alone satisfies ValidComposite★ vacuously: J0 is vacuous (no content allocated), J1★ is vacuous (no content-subspace extension), and J1'★ is vacuous (no provenance change). The orphan state is therefore a valid composite endpoint, not an error condition: the result is a link in dom(L) with no placement in any document's arrangement — an *orphan link*. The per-state invariants are preserved by K.λ's frame (M, C, E, R unchanged) and its preconditions (L growing by the single conforming entry ℓ), exactly as established by the Class (a) verification for K.λ; we do not restate those preservation facts here. Nelson explicitly diagrams "deleted links" as a category of document content (LM 4/9): links that exist in permanent storage but are "not currently addressable, awaiting historical backtrack functions."

We do not add a J0 analog for links — the orphan state is architecturally intentional, satisfying both the permanence guarantee (L12: links are immutable once created) and the owner's right to withdraw (Nelson, LM 2/29).


## Destruction confinement

We now state the central structural theorem — a generalisation of ASN-0036's content-store invariance under arrangement mutation to the extended state.

**P3 (ArrangementMutabilityOnly).** No component other than M admits contraction or value rewriting:

  `(A Σ → Σ' :: dom(C) ⊆ dom(C') ∧ dom(L) ⊆ dom(L') ∧ E ⊆ E' ∧ R ⊆ R' ∧ (A a ∈ dom(C) :: C'(a) = C(a)) ∧ (A ℓ ∈ dom(L) :: L'(ℓ) = L(ℓ)))`

The only component that can lose information is M.

*Proof.* By case analysis on the seven elementary transitions. K.α extends dom(C) preserving existing entries, with L, E, and R in its frame. K.δ extends E, with C, L, and R in its frame. K.λ extends dom(L) preserving existing entries, with C, E, and R in its frame. K.μ⁺ (amended), K.μ⁺_L, and K.μ⁻ have C, L, E, and R in their frames. K.ρ extends R, with C, L, and E in its frame. Each preserves every conjunct. The named composite K.μ~ decomposes into K.μ⁻ followed by K.μ⁺, both of which preserve the conjuncts, so K.μ~ does as well. General composite transitions, being finite sequences of elementary ones, preserve the conjuncts by transitivity of ⊆ and equality. ∎

P3 is the synthesis of P0 ∧ P1 ∧ P2 ∧ L12 — one named per-transition predicate over `Σ → Σ'` covering every component except M. P0 subsumes ASN-0036's S0 (ContentImmutability) and S1 (StoreMonotonicity), and L12 extends ASN-0043's link immutability. (Content-store invariance under arrangement mutation is already covered: every M-mutating transition frames `C' = C` by P0.)

The permanent record (what content exists, what links exist, which entities have been created, what provenance has been recorded) can only grow.


## Worked example: entity hierarchy by K.δ

We exercise the four K.δ patterns — case (i) node baptism, case (ii) k = 2 account descent, case (ii) k = 2 document descent, case (ii) k = 0 sibling document allocation — by building the chain `n₀ = 1 → 1.2 → 1.2.0.1 → 1.2.0.1.0.1 → 1.2.0.1.0.2` from Σ₀ (with E₀ = {1}).

**Step 1: K.δ case (i) — baptise node `1.2`.** Address `1.2` is supplied by the node-provisioning boundary, not by inc. Preconditions: `T4-valid(1.2)`, `Node(1.2)` (zeros = 0), `1.2 ∉ E₀` (discharged by NodeBaptism (a)), `n₀ ≼ 1.2` (`[1] ≼ [1, 2]`, discharged by NodeBaptism (b)). Effect: `E₁ = {1, 1.2}`, all other components frame.

**Step 2: K.δ case (ii) k = 2 — allocate account `1.2.0.1 = inc(1.2, 2)`.** TA5(d) gives `zeros = 1`, `parent = 1.2`. Preconditions: `parent(e) = 1.2 ∈ E₁`; `zeros(1.2) = 0 ≤ 2`; `1.2.0.1 ∉ E₁` holds because this is the first `(1.2, 2)` spawn — freshness per the K.δ k = 2 at-most-once read, discharged by ChildSpawnFreshness at `k' = 2` (the operand `1.2` is a node, admissible since ChildSpawnFreshness imposes no `¬Node` constraint on its operand); cross-event distinctness by GlobalUniqueness (ASN-0034). Effect: `E₂ = E₁ ∪ {1.2.0.1}`.

**Step 3: K.δ case (ii) k = 2 — allocate document `1.2.0.1.0.1 = inc(1.2.0.1, 2)`.** TA5(d) gives `zeros = 2`, `parent = 1.2.0.1`. Preconditions analogous to Step 2; `d ∉ E₂` holds because this is the first `(1.2.0.1, 2)` spawn — freshness per the K.δ k = 2 at-most-once read, discharged by ChildSpawnFreshness at `k' = 2`; cross-event distinctness by GlobalUniqueness (ASN-0034) at the document sub-allocator `A_doc(1.2.0.1)`. Effect: `E₃ = E₂ ∪ {1.2.0.1.0.1}`, with `M₃(1.2.0.1.0.1) = ∅` and SubAllocatorBundle activating the content and link sub-allocators (anchors `b_C(d) = [d.0.1]`, `b_L(d) = [d.0.2]`).

**Step 4: K.δ case (ii) k = 0 — allocate sibling document `1.2.0.1.0.2 = inc(1.2.0.1.0.1, 0)` under the same account.** Operand `t = 1.2.0.1.0.1`, the document placed by Step 3. The k = 0 dispatch differs from k ∈ {1, 2} in two distinctive features: (a) freshness `inc(t, 0) ∉ E` is discharged as the k = 0 frontier read of FrontierEquivalence, rather than the at-most-once-per-`(t, k')` read used at k ∈ {1, 2}; and (b) the dispatch through T10a.6 (DomainDisjointness) routes the verification against t's actual provenance — for this t, `A_doc(parent(t)) = A_doc(1.2.0.1)`, the account's document sub-allocator activated by Step 3.

*Structural identities (K.δ-ID).* TA5(c) gives `inc(t, 0)` by advancing the rightmost nonzero component: `inc(1.2.0.1.0.1, 0) = 1.2.0.1.0.2`. The named identities discharge the structural side conditions:
- K.δ-ID.zeros-0/1 at k = 0: `zeros(e) = zeros(t) = 2` — `1.2.0.1.0.2` has zero separators at positions 2 and 4 (matching `1.2.0.1.0.1`), so `zeros(e) = 2 = zeros(t)` directly. The document-level stratum is preserved.
- K.δ-ID.parent-0/1 at k = 0: `parent(e) = parent(t) = 1.2.0.1` — T4b's parent projection truncates at the last separator, giving the prefix `[1, 2, 0, 1]` for both `t = [1, 2, 0, 1, 0, 1]` and `e = [1, 2, 0, 1, 0, 2]`. The account `1.2.0.1` is the shared parent.

*Precondition discharge.*
- `t = 1.2.0.1.0.1 ∈ E₃` (placed by Step 3 into `E₃`, preserved by P1).
- `¬Node(t)`: `zeros(t) = 2 ≥ 1`, so `t` is not a node. T4b's parent projection is therefore defined at `t`.
- `inc(t, 0) = 1.2.0.1.0.2 ∉ E₃`: at `Σ₃ = (C₃, L₃, E₃, M₃, R₃)`, `t = 1.2.0.1.0.1` is the frontier of `A_doc(1.2.0.1)`'s chain (placed as its first emission by Step 3, with no later sibling-advance), so FrontierEquivalence gives `inc(t, 0) ∉ E₃`.

*Owning allocator.* For `t = 1.2.0.1.0.1`: Step 3 placed `t` into `dom(A_doc(1.2.0.1))`, so case (a') of ParentAllocatorDispatch (*Sub-allocator names*) applies (`A_doc(parent(t)) = A_doc(1.2.0.1)`), and the K.δ k = 0 event produces a sibling-advance on `A_doc(1.2.0.1)`'s frontier, yielding `e = inc(t, 0) = 1.2.0.1.0.2` as the second emission of that chain.

*Effect.* `E₄ = E₃ ∪ {1.2.0.1.0.2}`, with `M₄(1.2.0.1.0.2) = ∅` (the K.δ `Document(e)` case effect) and SubAllocatorBundle activating the content and link sub-allocators for the new sibling document (anchors `b_C(1.2.0.1.0.2) = [1.2.0.1.0.2.0.1]`, `b_L(1.2.0.1.0.2) = [1.2.0.1.0.2.0.2]`).

The zero-count progression `0 → 1 → 2` (Steps 1–3) exhausts the entity stratum at the document level: a hypothetical fourth k = 2 descent would produce `zeros = 3`, which is the Element stratum and falls outside E. Step 4's k = 0 sibling-increment preserves the document-level stratum (`zeros = 2`) while extending the population of `E_doc` under the same account.

After Step 4 fires, `1.2.0.1.0.2 ∈ E₄`, so the next K.δ k = 0 dispatch on `A_doc(1.2.0.1)`'s frontier operates on the *new* frontier `1.2.0.1.0.2`, producing `inc(1.2.0.1.0.2, 0) = 1.2.0.1.0.3` — a third sibling document.


## Worked example: fork with subsequent insertion

**Vacuity conventions for the worked traces (stated once).** To keep each step's verification to its substantive checks, we fix three conventions that hold throughout every worked example below and omit their per-step repetition. (1) *J1'★ on R-framing steps.* Any elementary step that holds R in frame has `R' \ R = ∅`, so J1'★ is vacuously satisfied at that step. (2) *Link-side vacuity under an empty link store.* At any state with `dom(L) = ∅`, the link invariants L0 (L-clause), L1, L1a, L1b, L1c, L3, L-fin, CL-OWN, and CL-UNIQ hold vacuously, S3★'s link clause is vacuous, and L14 reduces to `dom(C) ∩ ∅ = ∅`. (3) *Carry-forward of unlisted invariants.* Any Class (a) per-state invariant not explicitly verified at a step is inherited from the prior state by the relevant frame or restriction. The composite-boundary property P7a likewise carries forward across any step holding both `dom(C)` and `R` in frame — its witnessing provenance entries are unchanged — so such steps omit it; P7a is listed only at steps where `dom(C)` grows. Each step below lists only its non-vacuous checks.

We trace a concrete scenario to ground the abstract definitions. Let the starting state Σ₁ contain node 1, account 1.0.1, and document d₁ = 1.0.1.0.1 with two characters:

> C₁ = {1.0.1.0.1.0.1.1 ↦ 'H', 1.0.1.0.1.0.1.2 ↦ 'i'}
> E₁ = {1, 1.0.1, 1.0.1.0.1}
> M₁(d₁) = {[1,1] ↦ 1.0.1.0.1.0.1.1, [1,2] ↦ 1.0.1.0.1.0.1.2}
> R₁ = {(1.0.1.0.1.0.1.1, d₁), (1.0.1.0.1.0.1.2, d₁)}

We write a₁ = 1.0.1.0.1.0.1.1 and a₂ = 1.0.1.0.1.0.1.2 for brevity.

**Fork d₁ to d₂ = 1.0.1.0.1.1.** This is J4's compound K.δ + K.μ⁺ + K.ρ — the k = 1 version-creation case.

*K.δ:* E₂ = E₁ ∪ {1.0.1.0.1.1}. The address 1.0.1.0.1.1 = inc(1.0.1.0.1, 1) is obtained from d₁ = 1.0.1.0.1 by TA5's k = 1 child-allocation rule — a version of d₁ at the next address-space level. M₂(d₂) = ∅.

*A_v(d₁) activation.* The K.δ step is k = 1 with operand t = d₁ ∈ (E₁)_doc, so it is the T10a child-spawn step that activates `A_v(d₁)`, d₁'s version sub-allocator. By ParentAllocatorDispatch (*Sub-allocator names*), `d₁ = 1.0.1.0.1` is an original document inhabiting `dom(A_doc(1.0.1))`, so case (a') applies and the parent allocator of `A_v(d₁)` is `A_doc(parent(d₁)) = A_doc(1.0.1)`; the spawnPt premise `d₁ ∈ dom(A_doc(1.0.1))` is discharged by the precondition `t = d₁ ∈ (E₁)_doc`. child-spawn admissibility: spawn parameter `k' = 1 ∈ {1, 2}`, no zero-count side condition at `k' = 1`, and T10a's direct per-`(d₁, 1)` uniqueness axiom makes the spawn occur at most once (satisfied trivially as the first invocation). GlobalUniqueness (ASN-0034) on the newly activated `A_v(d₁)` delivers `1.0.1.0.1.1 ∉ E₁`. The output `inc(d₁, 1) = 1.0.1.0.1.1` is the first emission of `A_v(d₁)`, opening the chain for subsequent K.δ k = 0 versions (`inc(1.0.1.0.1.1, 0) = 1.0.1.0.1.2`, etc.) as sibling-advances on `A_v(d₁)`'s frontier.

*K.μ⁺:* M₂(d₂) = {[1,1] ↦ a₁, [1,2] ↦ a₂}. The same I-addresses as d₁ — transclusion, case (ii). No new content enters C. The V-positions [1,1] and [1,2] satisfy S8a (all components strictly positive, zeros = 0) and S8-depth (the two new positions share depth 2 among themselves within subspace s_C). The depth is d₂'s own free choice ≥ 2 — d₂ is created with M₂(d₂) = ∅, so its content-subspace depth is re-pinned from scratch by the first insertion (ValidFirstInsertionPosition) independently of d₁; the transclusion copies d₁'s I-addresses, not its V-position depths. The shared first component 1 — identifying subspace s_C — is a subspace-identity fact via `subspace(v)` (ASN-0036) rather than a clause of S8-depth itself.

*K.ρ:* R₂ = R₁ ∪ {(a₁, d₂), (a₂, d₂)}.

Verification against the resulting state Σ₂:

- *J0:* No fresh content (dom(C₂) = dom(C₁)), so vacuously satisfied.
- *J1★:* ran(M₂(d₂)|_{s_C}) \ ran(M₁(d₂)|_{s_C}) = {a₁, a₂} \ ∅ = {a₁, a₂} (M₁(d₂) = ∅ since d₂ ∉ (E₁)_doc). Both (a₁, d₂) and (a₂, d₂) are in R₂. ✓
- *J1'★:* `R₂ \ R₁ = {(a₁, d₂), (a₂, d₂)}` — both are new provenance entries from the K.ρ step. For each, the address must be new to d₂'s content-subspace range: `a₁ ∈ ran(M₂(d₂)|_{s_C}) = {a₁, a₂}` and `a₁ ∉ ran(M₁(d₂)|_{s_C}) = ∅` (M₁(d₂) = ∅), and symmetrically for a₂. Both entries are anchored in content-subspace range extensions introduced by the K.μ⁺ step of this composite. ✓
- *J4:* d₂ ∈ E₂_doc \ E₁_doc; the order-preserving bijection φ sends d₁'s content positions to d₂'s, M₂(d₂)(φ(v)) = M₁(d₁)(v): φ([1,1]) = [1,1] ↦ a₁ and φ([1,2]) = [1,2] ↦ a₂, so order is preserved (a₁ before a₂ in both) and multiplicity is preserved (a₁ ≠ a₂, no collapse). Derived range equality: ran(M₂(d₂)) = {a₁, a₂} = ran(M₁(d₁)|_{s_C}), d_op = d₁. ✓
- *S2 (Arrangement functionality) on d₂:* the K.μ⁺ step adds the mappings {[1,1] ↦ a₁, [1,2] ↦ a₂} at V-positions disjoint from `dom(M₂_int(d₂)) = ∅` (the K.δ step initialised `M₂_int(d₂) = ∅`); since the extension occurs at fresh positions, single-valuedness is preserved trivially. ✓
- *S3★:* every V-position in M₂(d₂) has `subspace(v) = s_C` (the K.μ⁺ amendment supplies this at every step that adds positions; the K.δ step initialised `M₂(d₂) = ∅`), and each maps to `a₁, a₂ ∈ dom(C₂)` by S3★'s content clause. ✓
- *S4 (Content distinctness, per ASN-0036) on d₂'s range:* a₁ ≠ a₂ as content addresses — both inhabit dom(C₁), and the inductive hypothesis at Σ₁ (S4 over dom(C₁)) supplies the distinctness from the pre-state. The K.δ + K.μ⁺ + K.ρ fork composite holds C in frame (no new K.α event), so the dom(C) population — and therefore S4 — is preserved exactly across the composite. ✓
- *D-CTG★ on V_{s_C}(d₂) = {[1,1], [1,2]}:* under the V-ordering on s_C (lex order on depth-2 positive tuples with first component 1), the only depth-2 positive tuple with first component 1 lex-between [1,1] and [1,2] is the bounds themselves — there is no third position to check; contiguity holds. ✓
- *D-MIN★ on V_{s_C}(d₂):* `min(V_{s_C}(d₂)) = [1,1] = [s_C, 1]` of depth m_{s_C} = 2 — matching the D-MIN★ canonical form `[S, 1, ..., 1]`. ✓
- *D-SEQ★ on V_{s_C}(d₂):* `V_{s_C}(d₂) = {[1,1], [1,2]}` matches the canonical form `{[s_C, k] : 1 ≤ k ≤ 2}` at `n_{s_C} = 2` and `m_{s_C} = 2`. ✓
- *P4★:* Contains_C(Σ₂) = {(a₁, d₁), (a₂, d₁), (a₁, d₂), (a₂, d₂)} ⊆ R₂. ✓
- *P4a:* the two new entries `R₂ \ R₁ = {(a₁, d₂), (a₂, d₂)}` are each witnessed by Σ₂ itself — the K.μ⁺ step of this composite installs `M₂(d₂)([1,1]) = a₁` and `M₂(d₂)([1,2]) = a₂` with `subspace([1,1]) = subspace([1,2]) = s_C`, so taking `Σ_k = Σ₂`, `v = [1,1]` witnesses `(a₁, d₂)` and `v = [1,2]` witnesses `(a₂, d₂)`. The carried-forward entries `(a₁, d₁), (a₂, d₁)` retain their Σ₁ witnesses. ✓
- *P8:* parent(d₂) = parent(1.0.1.0.1.1) = 1.0.1 ∈ E₁ ⊆ E₂ (k = 1 preserves parent(d_new) = parent(d_src), so parent(d₂) = parent(d₁) = 1.0.1). The existing non-node entity 1.0.1 (account) retains parent(1.0.1) = 1 ∈ E₂. ✓

**Insert new content into d₂.** Compound K.α + K.μ⁺ + K.ρ.

*K.α:* Allocate a₃ = 1.0.1.0.1.1.0.1.1 with C₃(a₃) = '!'. The address falls under d₂'s prefix (S7a): origin(a₃) = 1.0.1.0.1.1 = d₂. The freshness of a₃ — i.e., `a₃ ∉ dom(C₂)` — is discharged by the first-emission clause at d₂'s content sub-allocator. This K.α event is the first emission of d₂'s content sub-allocator A_C(d₂) — d₂ was created at the immediately preceding K.δ step with the convention dom_s(A_C(d₂)) = ∅ at activation. FirstEmissionFreshness (ASN-0093) directly supplies `[d₂.0.s_C.1] ∉ dom(Σ.C) ∪ dom(Σ.L)` at the state of allocation, discharging `a₃ ∉ dom(C₂)` in full; the first-emission clause commits *only* the first emission of the activated sub-allocator, not every output, so subsequent emissions of A_C(d₂) would discharge freshness via GlobalUniqueness (ASN-0034) on its inc chain instead. Freshness against A_C(d₂)'s own prior emissions is vacuous at the empty initial domain.

*K.μ⁺:* M₃(d₂) = M₂(d₂) ∪ {[1,3] ↦ a₃}. V-position [1,3] has `subspace([1,3]) = 1 = s_C` and depth 2, matching [1,1] and [1,2] — S8-depth holds at the common depth `m_{s_C} = 2`. Referential integrity: a₃ ∈ dom(C₃) (S3). ✓

*K.ρ:* R₃ = R₂ ∪ {(a₃, d₂)}.

Verification:

- *J0:* a₃ ∈ dom(C₃) \ dom(C₂), and d₂ ∈ E₃_doc with M₃(d₂)([1,3]) = a₃. ✓
- *J1★:* ran(M₃(d₂)|_{s_C}) \ ran(M₂(d₂)|_{s_C}) = {a₃}, and (a₃, d₂) ∈ R₃. ✓
- *J1'★:* `R₃ \ R₂ = {(a₃, d₂)}` — the K.ρ step adds exactly this entry. The address `a₃` is new to d₂'s content-subspace range: `a₃ ∈ ran(M₃(d₂)|_{s_C}) = {a₁, a₂, a₃}` and `a₃ ∉ ran(M₂(d₂)|_{s_C}) = {a₁, a₂}`. The new provenance is anchored in the K.μ⁺ step's content-subspace range extension. ✓
- *S3★:* the new V-position [1,3] has `subspace([1,3]) = s_C` (K.μ⁺ amendment) and maps to `a₃ ∈ dom(C₃)`; existing V-positions retain their mappings into dom(C₂) ⊆ dom(C₃) by frame and P0. ✓
- *P4★:* Contains_C(Σ₃) adds (a₃, d₂); this pair is in R₃. ✓
- *P6:* origin(a₃) = d₂ = 1.0.1.0.1.1 ∈ E₃_doc. ✓
- *P7:* (a₃, d₂) ∈ R₃ and a₃ ∈ dom(C₃). ✓
- *P7a:* dom(C₃) = {a₁, a₂, a₃}; a₁ and a₂ retain provenance from R₂ ⊆ R₃, and a₃ has new provenance (a₃, d₂) ∈ R₃. Every a ∈ dom(C₃) has at least one provenance entry. ✓
- *L0 (C-clause):* reaffirmed for the new address a₃ by K.α's `E(a₃)₁ = s_C` precondition; prior addresses carry forward from Σ₂. ✓

**Delete a₃ from d₂'s arrangement (K.μ⁻).** Remove the mapping at V-position [1,3] — the maximum end of V_{s_C}(d₂), satisfying the K.μ⁻ amendment's D-CTG★/D-MIN★ postcondition.

*K.μ⁻:* dom(M₄(d₂)) = {[1,1], [1,2]} ⊂ dom(M₃(d₂)) = {[1,1], [1,2], [1,3]}. The surviving mappings are unchanged: M₄(d₂)([1,1]) = a₁, M₄(d₂)([1,2]) = a₂. D-MIN★: min(V_1(d₂)) = [1,1] = [s_C, 1]. D-CTG★: {[1,1], [1,2]} is contiguous.

Verification:

- *J2:* C₄ = C₃; E₄ = E₃; R₄ = R₃. All permanent and historical state unchanged. ✓
- *P4★:* Contains_C(Σ₄) = {(a₁, d₁), (a₂, d₁), (a₁, d₂), (a₂, d₂)}. The pair (a₃, d₂) is no longer in Contains_C — d₂ no longer displays a₃. Yet (a₃, d₂) ∈ R₄: the stale entry persists. Contains_C(Σ₄) ⊂ Contains_C(Σ₃), while R₄ = R₃. ✓
- *S3★:* surviving mappings retain their content-subspace V-positions and dom(C₄) = dom(C₃) targets by restriction; the removed mapping at [1,3] no longer participates. ✓
- *D-CTG★ / D-MIN★ / D-SEQ★ at Σ₄:* `V_{s_C}(d₂) = {[1,1], [1,2]}` is the contiguous prefix `{[s_C, k] : 1 ≤ k ≤ 2}` with minimum [1,1] = [s_C, 1] — the suffix-removal shape required by K.μ⁻ at the post-state. ✓

The divergence is now concrete: R₄ records that d₂ once contained a₃, while the current arrangement does not. This is the historical memory that J2 preserves — deletion is purely presentational.

**Reorder d₂'s arrangement (K.μ~).** Swap V-positions [1,1] and [1,2].

*K.μ~ firing precondition:* M₄(d₂)|_{dom_C} = {[1,1] ↦ a₁, [1,2] ↦ a₂} with a₁ ≠ a₂ — two distinct values, so K.μ~ fires.

*K.μ~:* The bijection π : {[1,1], [1,2]} → {[1,1], [1,2]} with π([1,1]) = [1,2] and π([1,2]) = [1,1]. The definition requires M₅(d₂)(π(v)) = M₄(d₂)(v) for all v ∈ dom(M₄(d₂)), giving M₅(d₂) = {[1,1] ↦ a₂, [1,2] ↦ a₁}. Both target V-positions satisfy S8a (all components strictly positive) and S8-depth (uniform depth 2 within subspace s_C), with subspace(v) = 1 for both positions.

Verification:

- *J3:* C₅ = C₄; E₅ = E₄; R₅ = R₄. All permanent and historical state unchanged. ✓
- *ran preservation:* ran(M₅(d₂)) = {a₁, a₂} = ran(M₄(d₂)). The multiset of referenced I-addresses is identical; only V-positions changed. ✓
- *P4★:* Contains_C(Σ₅) = Contains_C(Σ₄) ⊆ R₄ = R₅. Since ran is preserved for d₂ and no other arrangement changed, the current containment set is unchanged. ✓
- *S3★:* both V-positions retain `subspace(v) = s_C` (the swap permutes content-subspace positions only) and map into dom(C₅) = dom(C₄). ✓

Reordering is the simplest transition to verify: it touches nothing beyond the V-position mapping, and all invariants hold by the frame conditions alone.


## Worked example: subsequent-version fork (k = 0)

The first-version fork above (d₁ → d₂) exercises J4's k = 1 sub-case. We now exercise the k = 0 sub-case — a *subsequent* version on the same chain — to verify J4 step (ii)'s claim concretely: the content source is the K.δ operand `d_op = prev_version`, so a subsequent version inherits the prior version's *edits*.

Return to the state immediately after *Insert new content into d₂* (state Σ₃ above, before the deletion). At that point d₂ = 1.0.1.0.1.1 is the sole emission on d₁'s version chain `A_v(d₁)` and carries the edit a₃ = 1.0.1.0.1.1.0.1.1 that is *absent* from d₁:

> C₃ = {a₁ ↦ 'H', a₂ ↦ 'i', a₃ ↦ '!'}
> E₃_doc ⊇ {d₁, d₂},  d₁ = 1.0.1.0.1,  d₂ = 1.0.1.0.1.1
> M₃(d₂) = {[1,1] ↦ a₁, [1,2] ↦ a₂, [1,3] ↦ a₃}
> M₃(d₁) = {[1,1] ↦ a₁, [1,2] ↦ a₂}   (unedited base)
> R₃ ⊇ {(a₁, d₂), (a₂, d₂), (a₃, d₂)}

**Fork d₂ to d₃ = 1.0.1.0.1.2.** This is J4's k = 0 sub-case. The content source operand is `d_op = prev_version = d₂` — the current frontier of `A_v(d₁)` — *not* the base d₁. Precondition: d₁ ∈ E_doc (the chain root d_src), d_op = d₂ ∈ E_doc, and V_{s_C}(d₂) = {[1,1], [1,2], [1,3]} ≠ ∅.

*K.δ (k = 0):* d₃ = inc(d₂, 0) = inc(1.0.1.0.1.1, 0) = 1.0.1.0.1.2 by TA5(c) (advance the rightmost nonzero component). E' = E₃ ∪ {1.0.1.0.1.2}, M'(d₃) = ∅.

*A_v(d₁) frontier-advance discharge.* The operand t = d₂ = 1.0.1.0.1.1 is the current frontier of `A_v(d₁)` (placed as its first emission by the k = 1 fork above, with no later sibling-advance). Freshness `inc(d₂, 0) = 1.0.1.0.1.2 ∉ E₃` is discharged via FrontierEquivalence, not the direct per-(t, k') axiom. The output d₃ is the second emission of `A_v(d₁)`, and d₁ ≼ d₃ (1.0.1.0.1 ≼ 1.0.1.0.1.2), so d₃ carries the ancestry-by-address indication.

*K.μ⁺:* M'(d₃) = {[1,1] ↦ a₁, [1,2] ↦ a₂, [1,3] ↦ a₃} — the same I-addresses, in the same order, as the *edited* d₂, transcluding d_op = d₂'s current content subspace. The order-preserving bijection φ sends d₂'s content positions to d₃'s identically on the last component: φ([1,k]) = [1,k] ↦ M(d₂)([1,k]) for k = 1, 2, 3, so `M'(d₃)(φ(v)) = M(d₂)(v)` holds and order is preserved. Derived range equality `ran(M'(d₃)) = ran(M(d_op)|_{V_{s_C}(d_op)}) = ran(M₃(d₂)|_{s_C}) = {a₁, a₂, a₃}` follows; crucially a₃ — d₂'s edit, absent from d₁ — is inherited at its source position. No new content enters C (C' = C₃). The three content-subspace V-positions satisfy S8a (positive components, zeros = 0) and S8-depth (uniform depth 2).

*K.ρ:* R' = R₃ ∪ {(a₁, d₃), (a₂, d₃), (a₃, d₃)}.

Verification against the resulting state Σ':

- *J0:* dom(C') = dom(C₃); no fresh content, so vacuously satisfied. ✓
- *J1★:* ran(M'(d₃)|_{s_C}) \ ran(M₃(d₃)|_{s_C}) = {a₁, a₂, a₃} \ ∅ = {a₁, a₂, a₃} (M₃(d₃) = ∅ since d₃ ∉ E₃_doc). Each of (a₁, d₃), (a₂, d₃), (a₃, d₃) is in R'. ✓
- *J1'★:* R' \ R₃ = {(a₁, d₃), (a₂, d₃), (a₃, d₃)}; each address is new to d₃'s content-subspace range ({a₁, a₂, a₃} present in ran(M'(d₃)|_{s_C}), absent from ran(M₃(d₃)|_{s_C}) = ∅). Each entry is anchored in the K.μ⁺ step's content-subspace extension. ✓
- *J4 (φ-copy against the operand):* d₃ ∈ E'_doc \ E₃_doc, and the order-preserving bijection φ against d_op = d₂ gives M'(d₃)(φ(v)) = M(d₂)(v) for every v ∈ V_{s_C}(d₂); derived range equality ran(M'(d₃)) = {a₁, a₂, a₃} = ran(M(d_op)|_{V_{s_C}(d_op)}). The copy is against the *operand* d₂. It would *lose content* against the original base d₁: ran(M₃(d₁)|_{s_C}) = {a₁, a₂} omits a₃, so a d_src-fixed source would drop a₃ — precisely the silent edit-loss the operand-tracking source prevents. ✓
- *S2 (Arrangement functionality) on d₃:* the K.μ⁺ adds {[1,1] ↦ a₁, [1,2] ↦ a₂, [1,3] ↦ a₃} at positions disjoint from dom(M'_int(d₃)) = ∅ (the K.δ step initialised M'(d₃) = ∅); the extension is at fresh positions, so single-valuedness holds trivially. ✓
- *S3★:* every V-position in M'(d₃) has subspace(v) = s_C (K.μ⁺ amendment) and maps to a₁, a₂, a₃ ∈ dom(C') = dom(C₃); content is frame-preserved across the composite (no K.α). ✓
- *D-CTG★ on V_{s_C}(d₃) = {[1,1], [1,2], [1,3]}:* contiguous at depth 2 — no depth-2 positive tuple with first component 1 lies between consecutive members. ✓
- *D-MIN★ on V_{s_C}(d₃):* min = [1,1] = [s_C, 1], depth m_{s_C} = 2 — matching the canonical form [S, 1, ..., 1]. ✓
- *D-SEQ★ on V_{s_C}(d₃):* {[1,1], [1,2], [1,3]} = {[s_C, k] : 1 ≤ k ≤ 3} at n_{s_C} = 3, m_{s_C} = 2. ✓
- *P4★:* Contains_C(Σ') ⊇ {(a₁, d₃), (a₂, d₃), (a₃, d₃)}, all in R'. ✓
- *P8:* parent(d₃) = parent(1.0.1.0.1.2) = 1.0.1 ∈ E₃ ⊆ E' (k = 0 preserves parent(d_new) = parent(prev_version) = parent(d₁) = 1.0.1). ✓

The k = 0 fork inherits a₃ — the prior version's edit — because the transclusion source is the operand d₂ (the chain frontier), not the base d₁, instantiating the operand-tracking rule of J4 step (ii). The address allocation, by contrast, is uniform with the k = 1 case — a frontier-advance on `A_v(d₁)` — so the two sub-cases share an allocation discipline while differing only in which document supplies the transcluded content.


## Worked example: fork of a duplicate-I-address source

The three forks above all transclude sources whose content V-positions carry *pairwise-distinct* I-addresses, so they exercise J4 step (ii)'s order-preservation clause but leave its *multiplicity-preservation* clause — the load-bearing distinction between the φ-copy characterization and mere range equality — unchecked against the scenario that separates the two. We supply that scenario here: a source in which two content V-positions carry the *same* I-address, as S5 (UnrestrictedSharing, ASN-0036) permits.

*Source state.* Let document `d_op = 1.0.1.0.3 ∈ E_doc` carry a single content address `a = 1.0.1.0.3.0.1.1` transcluded at two distinct content V-positions:

> C ⊇ {a ↦ 'x'},  a = 1.0.1.0.3.0.1.1
> M(d_op) = {[1,1] ↦ a, [1,2] ↦ a}
> R ⊇ {(a, d_op)}

The duplicate arises by two K.μ⁺ transclusions of the *same* pre-existing `a` (case (ii), no new content): `[1,1] ↦ a` then `[1,2] ↦ a`. This is consistent with S2 (each V-position has at most one image — both positions do) and exhibits the multiplicity S5 permits (`|{(d_op, v) : M(d_op)(v) = a}| = 2`). Content-subspace V-positions: `V_{s_C}(d_op) = {[1,1], [1,2]}` — contiguous (D-CTG★), minimum `[1,1] = [s_C, 1]` (D-MIN★), shape `{[s_C, k] : 1 ≤ k ≤ 2}` (D-SEQ★ at `n_{s_C} = 2`, `m_{s_C} = 2`).

**Fork d_op to d_new = 1.0.1.0.3.1.** This is J4's k = 1 sub-case (first version on `A_v(d_op)`). Precondition: `d_src = d_op ∈ E_doc`, `d_op ∈ E_doc`, `V_{s_C}(d_op) = {[1,1], [1,2]} ≠ ∅`.

*K.δ (k = 1):* `d_new = inc(d_op, 1) = 1.0.1.0.3.1`. E' = E ∪ {d_new}, M'(d_new) = ∅. d_op ≼ d_new carries the ancestry-by-address indication.

*K.μ⁺:* The order-preserving bijection `φ : V_{s_C}(d_op) → V_{s_C}(d_new)` sends `φ([1,k]) = [1,k]` (identical last component; d_new's content subspace re-pins at depth `m_new = 2`). Step (ii) establishes `M'(d_new)(φ(v)) = M(d_op)(v)` for each `v`:

> M'(d_new)([1,1]) = M(d_op)([1,1]) = a
> M'(d_new)([1,2]) = M(d_op)([1,2]) = a

so `M'(d_new) = {[1,1] ↦ a, [1,2] ↦ a}` — **two** target positions, **one** I-address. No new content enters C (C' = C).

*K.ρ:* R' = R ∪ {(a, d_new)} — one provenance entry, since `a` is the sole distinct content address.

Verification against Σ':

- *J4 (multiplicity preservation, the distinctive postcondition):* φ is injective, so the two distinct source positions `[1,1] ≠ [1,2]` map to two distinct target positions `[1,1] ≠ [1,2]`, each retaining `a`. The duplicate is **not** collapsed: `|dom(M'(d_new))| = 2 = |V_{s_C}(d_op)|`, so the document's content count is preserved. The deduplicating outcome `M'(d_new) = {[1,1] ↦ a}` — count 1 — is *not* produced, even though it satisfies the same derived range equality (see next item). This is exactly the case the φ-bijection characterization rules out and range equality alone cannot. ✓
- *Derived range equality (lossy, hence non-discriminating):* `ran(M'(d_new)) = {a} = ran(M(d_op)|_{V_{s_C}(d_op)})`. The range projection collapses the two positions to the single address `{a}`, so it holds equally of the correct count-2 arrangement and the wrong count-1 deduplication — confirming that range equality is a *consequence* of the φ-copy, never its characterization. ✓
- *S2 (Arrangement functionality) on d_new despite non-injectivity:* S2 constrains single-valuedness — each V-position has at most one image — not injectivity. Here `[1,1] ↦ a` and `[1,2] ↦ a` give each of the two distinct positions exactly one image, so S2 holds; the two positions sharing the image `a` is permitted (S2 does not require distinct positions to carry distinct addresses). The K.μ⁺ adds both mappings at positions disjoint from `dom(M'_int(d_new)) = ∅`. ✓
- *J1★:* `ran(M'(d_new)|_{s_C}) \ ran(M(d_new)|_{s_C}) = {a} \ ∅ = {a}` (M(d_new) = ∅ since d_new ∉ E_doc pre-fork); `(a, d_new) ∈ R'`. A single new range entry, hence a single provenance entry, even though two V-positions carry it. ✓
- *D-CTG★ / D-MIN★ / D-SEQ★ on V_{s_C}(d_new) = {[1,1], [1,2]}:* contiguous at depth 2, minimum `[1,1] = [s_C, 1]`, shape `{[s_C, k] : 1 ≤ k ≤ 2}` at `n_{s_C} = 2`. The shape invariants constrain the *V-positions*, which remain two and sequential regardless of the collision in their images. ✓
- *S3★:* both content-subspace V-positions map to `a ∈ dom(C') = dom(C)`; content is frame-preserved (no K.α). ✓

The fork of a duplicate source thus produces a *non-injective* `M'(d_new)` — two V-positions, one I-address — which S2 permits but does not require, and which the φ-bijection forces. A faithful version copy reproduces the source's sharing structure exactly; a deduplicating range copy would silently drop a position and undercount the contents. The φ-injectivity postcondition checked above (`|dom(M'(d_new))| = 2 = |V_{s_C}(d_op)|`) is what rules the deduplicating copy out.


## Worked example: fork with depth rebasing (m_new ≠ m_old)

The three forks above all re-pin d_new's content subspace at the *same* depth as the source (`m_new = m_old = 2`), so `φ([1, k]) = [1, k]` is the identity on V-position values and the depth-*rebasing* half of J4 step (ii)'s characterization is never exercised. Yet `m_new` is a free caller choice (`≥ 2` by S8a) at d_new's first content insertion — d_new starts empty (`V_{s_C}(d_new) = ∅`), so ValidFirstInsertionPosition (ASN-0036) admits any depth `m ≥ 2`, and `m_new = m_old` is *not* forced. We exercise a fork with `m_new = 3 ≠ 2 = m_old`, the case where φ is genuinely non-identity, to verify it remains an order- and multiplicity-preserving bijection and that D-SEQ★/D-CTG★/D-MIN★ hold at the rebased depth.

*Source state.* Let document `d_op = 1.0.1.0.5 ∈ E_doc` carry two distinct content addresses transcluded at content depth `m_old = 2`:

> C ⊇ {a₁ ↦ 'p', a₂ ↦ 'q'},  a₁ = 1.0.1.0.5.0.1.1,  a₂ = 1.0.1.0.5.0.1.2
> M(d_op) = {[1,1] ↦ a₁, [1,2] ↦ a₂}
> R ⊇ {(a₁, d_op), (a₂, d_op)}

Content-subspace V-positions: `V_{s_C}(d_op) = {[1,1], [1,2]}` — contiguous (D-CTG★), minimum `[1,1] = [s_C, 1]` (D-MIN★), shape `{[s_C, k] : 1 ≤ k ≤ 2}` (D-SEQ★ at `n_{s_C} = 2`, `m_old = 2`).

**Fork d_op to d_new = 1.0.1.0.5.1.** This is J4's k = 1 sub-case (first version on `A_v(d_op)`). Precondition: `d_src = d_op ∈ E_doc`, `d_op ∈ E_doc`, `V_{s_C}(d_op) = {[1,1], [1,2]} ≠ ∅`.

*K.δ (k = 1):* `d_new = inc(d_op, 1) = 1.0.1.0.5.1`. E' = E ∪ {d_new}, M'(d_new) = ∅. d_op ≼ d_new carries the ancestry-by-address indication.

*K.μ⁺ (depth rebased to `m_new = 3`):* the caller's first content insertion into the empty `V_{s_C}(d_new)` pins `m_new = 3`. The order-preserving bijection `φ : V_{s_C}(d_op) → V_{s_C}(d_new)` therefore maps the depth-2 source position `[s_C, 1, ..., 1, k]` (here `[1, k]`) to the depth-3 target position `[s_C, 1, ..., 1, k]` (here `[1, 1, k]`): `φ([1, k]) = [1, 1, k]` — *non-identity* on V-position values, a genuine depth rebasing. Step (ii) establishes `M'(d_new)(φ(v)) = M(d_op)(v)`:

> M'(d_new)([1,1,1]) = M(d_op)([1,1]) = a₁
> M'(d_new)([1,1,2]) = M(d_op)([1,2]) = a₂

so `M'(d_new) = {[1,1,1] ↦ a₁, [1,1,2] ↦ a₂}` at depth 3. No new content enters C (C' = C).

*K.ρ:* R' = R ∪ {(a₁, d_new), (a₂, d_new)}.

Verification against Σ':

- *J4 order preservation under non-identity φ:* the source order `[1,1] < [1,2]` maps to `[1,1,1] < [1,1,2]` (both compare by their last component, equal in positions 1..2). φ is order-preserving across the depth change — the relative content order survives rebasing from depth 2 to depth 3. ✓
- *J4 multiplicity preservation:* φ is injective, so the two distinct source positions map to two distinct target positions, each retaining its I-address (`a₁ ≠ a₂`); `|dom(M'(d_new))| = 2 = |V_{s_C}(d_op)|`. Had the source carried a duplicate I-address, injectivity of φ would preserve it identically across the rebasing, as the duplicate-source example showed at equal depth. ✓
- *D-SEQ★ / D-CTG★ / D-MIN★ on `V_{s_C}(d_new) = {[1,1,1], [1,1,2]}` at the rebased depth 3:* contiguous (D-CTG★, the last component runs 1, 2 with no gap — D-CTG-depth reduces contiguity to the last component since positions 2..m−1 are the constant 1), minimum `[1,1,1] = [s_C, 1, 1]` (D-MIN★, all-ones tuple of length 3), shape `{[s_C, 1, k] : 1 ≤ k ≤ 2}` (D-SEQ★ at `n_{s_C} = 2`, `m_new = 3`). Each target is a zero-free length-3 tumbler with all components positive (S8a), and all share depth 3 (S8-depth). ✓
- *Derived range equality:* `ran(M'(d_new)) = {a₁, a₂} = ran(M(d_op)|_{V_{s_C}(d_op)})` — every source I-address inherited at a corresponding rebased position, no new content. ✓
- *S3★:* both V-positions map to addresses in `dom(C') = dom(C)`; content frame-preserved. ✓

The depth rebasing thus changes only the numeric V-values (the depth-3 re-derivation of φ), never the order, multiplicity, or range of the transcluded content — confirming J4 step (ii)'s characterization in its non-degenerate `m_new ≠ m_old` case, which the equal-depth examples could not reach.


## Worked example: interior content replacement

We trace the interior-position case of the content-replacement composite — the *separate, range-changing* K.μ⁻ + K.μ⁺ pair identified in the *Elementary transitions* section (the form excluded from K.μ~ by its range-preservation clause). To parameterise the contraction depth we write `n'_{s_C} = k₀ − 1` for the post-contraction content-subspace count, where `1 ≤ k₀ ≤ n_{s_C}` is the lowest cut position (the K.μ⁻ removes the suffix from position `k₀` upward): the interior case takes a cut below the top, `n'_{s_C} = k₀ − 1` with `k₀ < n_{s_C}`, rather than the single-position pair at `k₀ = n_{s_C}`.

*Initial state.* Let document `d = 1.0.1.0.1` have four content-subspace mappings, with `aₖ := 1.0.1.0.1.0.1.k` for `k ∈ {1, 2, 3, 4}`:

> C ⊇ {a₁ ↦ char₁, a₂ ↦ char₂, a₃ ↦ char₃, a₄ ↦ char₄}
> M(d) = {[1,1] ↦ a₁, [1,2] ↦ a₂, [1,3] ↦ a₃, [1,4] ↦ a₄}
> R ⊇ {(a₁, d), (a₂, d), (a₃, d), (a₄, d)}

Content-subspace V-positions: `V_{s_C}(d) = {[1,1], [1,2], [1,3], [1,4]}` — contiguous (D-CTG★), minimum `[1,1] = [s_C, 1]` (D-MIN★), uniform depth 2 (S8-depth), structural form `{[s_C, k] : 1 ≤ k ≤ 4}` (D-SEQ★ at `n_{s_C} = 4`, `m_{s_C} = 2`; the general D-SEQ★ form `{[s_C, 1, ..., 1, k]}` has no intermediate 1s since the inner range from position 2 to position `m_{s_C} − 1 = 1` is empty). Link subspace: `V_{s_L}(d) = ∅`. The four pre-state provenance entries are assumed established by prior J0/J1★ couplings at d's initial population (the details are not material here).

**Goal.** Replace the I-address at the interior V-position `[1,2]` with a freshly allocated `a₂' ≠ a₂` of new content value. Positions `[1,3]` and `[1,4]` lie strictly above `[1,2]` under the V-ordering on `s_C` (T1 of ASN-0034 restricted to depth-2 positive tuples with first component 1), so a single-position K.μ⁻ + K.μ⁺ pair at `[1,2]` alone would leave `V_{s_C}(d)` with an interior hole at `[1,2]` between `[1,1]` and `[1,3]` at the intermediate state — the interior-hole shape excluded by D-CTG★. The replacement therefore decomposes as a multi-position K.μ⁻ removing the suffix from `[1,2]` upward, followed by K.α allocating `a₂'`, then a multi-position K.μ⁺ rebuilding the suffix with `a₂'` at `[1,2]` and the previously-mapped `a₃, a₄` at `[1,3], [1,4]`, and finally K.ρ recording the new provenance — four elementary steps in this order. (An alternative valid ordering, K.α before K.μ⁻, produces the same composite endpoints; the order chosen here keeps the K.μ⁻ removal at the head of the trace, matching the narrative of "interior replacement = remove suffix, then rebuild.")

**Step 1: K.μ⁻ — remove the interior suffix `{[1,2], [1,3], [1,4]}`.** Effect: `M_int(d) = {[1,1] ↦ a₁}`. Frame: `C_int = C`, `L_int = L`, `E_int = E`, `R_int = R`.

*Precondition discharge.* K.μ⁻'s explicit preconditions are (a) `d ∈ E_doc` — satisfied by hypothesis (`d = 1.0.1.0.1 ∈ E_doc`); and (b) the per-subspace retention choice, verified at the pre-state against `n_{s_C} = 4` read from D-SEQ★ at Σ: the caller commits to retention count `n'_{s_C} = 1` (the link subspace is empty, `n'_{s_L} = n_{s_L} = 0`), with `n'_{s_C} = 1 ∈ {0, …, 4}` and strict contraction `n'_{s_C} = 1 < 4 = n_{s_C}` on the content subspace. Everything else is *derived*, not a separate check: non-emptiness `dom(M(d)) ≠ ∅` is forced by strict contraction (per K.μ⁻'s definition), and the per-state arrangement invariants S2, S3★, S8a, S8-depth, S8-fin, D-CTG★, and D-MIN★ at `M_int(d)` are derived consequences of the constructive restriction form — verified ex post at `M_int` below under *Intermediate-state verification*.

*Chosen contraction shape (degree of freedom, not a precondition).* No separate per-subspace shape precondition is checked at firing time; the suffix-removal shape on each subspace follows from the constructive restriction form (see *K.μ⁻ admissible contraction shape*). The shape we *choose* for this trace (the operation's degree of freedom, designed-in to support the goal of replacing the interior position `[1,2]` by removing the suffix from `[1,2]` upward):
- *Content subspace.* `V_{s_C}(d) = {[1,1], [1,2], [1,3], [1,4]}` shrinks to `V_{s_C}(d_int) = {[1,1]}` — a partial suffix removal retaining `n'_{s_C} = 1` of the four pre-state positions. The retained prefix `{[s_C, 1]}` and the removed suffix `{[s_C, k] : 1 < k ≤ 4}` are the partition forced by the post-state D-CTG★ + D-MIN★ + D-SEQ★ once the contraction commits.
- *Link subspace.* `V_{s_L}(d) = V_{s_L}(d_int) = ∅` — no link-subspace positions to remove (the post-state shape is trivially the empty arrangement).

Since `dom(M(d)) ≠ ∅` (precondition (b)) and the chosen contraction shape produces strict contraction on the content subspace (4 → 1), the effect-clause requirement `dom(M_int(d)) ⊂ dom(M(d))` is satisfied at the whole-arrangement level. K.μ⁻ commits; the per-subspace shape is what the post-state invariants confirm ex post, not what was verified as a precondition ex ante.

*Intermediate-state verification at M_int.* We check the Class (a) per-state invariants at M_int below (the per-state / composite-boundary temporal-scope distinction is established once, in the *Extended reachable-state invariants* preamble); P4★ holds at M_int only as a consequence of K.μ⁻'s contraction, with restoration at the trailing K.ρ.

- *D-CTG★ at M_int:* `V_{s_C}(d_int) = {[1,1]}` is a singleton — vacuously contiguous under the V-ordering on `s_C` (no two distinct members bracket an interval). ✓
- *D-MIN★ at M_int:* `min(V_{s_C}(d_int)) = [1,1] = [s_C, 1]` of depth `m_{s_C} = 2`. ✓
- *D-SEQ★ at M_int:* `V_{s_C}(d_int) = {[s_C, 1]}` matches `{[s_C, k] : 1 ≤ k ≤ 1}` at `n_{s_C} = 1` (`m_{s_C} = 2`). ✓
- *S2, S3★, S8a, S8-depth, S8-fin at M_int:* the surviving mapping `[1,1] ↦ a₁` is functional, has all-positive components and uniform depth 2 in `s_C`, with `a₁ ∈ dom(C_int) = dom(C)`. ✓
- *Per-state invariants at M_int:* P6/P7/P8 preserved by K.μ⁻'s frame on C, E, R. ✓
- *P4★ at M_int (consequence, not requirement).* `Contains_C(M_int) = {(a₁, d)} ⊆ Contains_C(Σ) ⊆ R = R_int`. P4★ holds at M_int as a consequence of K.μ⁻'s monotonicity: K.μ⁻ can only shrink Contains_C and R is unchanged (J2). The pairs `(a₂, d), (a₃, d), (a₄, d)` exit Contains_C at this step but remain in R as stale entries. ValidComposite★ does not require P4★ at intermediate states; the next intermediate state M_post (after K.μ⁺ but before K.ρ) genuinely violates P4★ at the pair `(a₂', d)`, with restoration occurring at the trailing K.ρ.

**Step 2: K.α — allocate the replacement address `a₂'`.** Allocate `a₂' = 1.0.1.0.1.0.1.5 = inc(a₄, 0)` (the next sibling on d's content sub-allocator's frontier under TA5(c)) with `C'(a₂') = char₂'` for some new content value. Effect: `C' = C ∪ {a₂' ↦ char₂'}`. Frame: L, E, M (= M_int), R unchanged.

Preconditions: Element(a₂') (zeros = 3, element-field `[1, 5]`); origin(a₂') = `1.0.1.0.1` = d ∈ E_doc; `subspace_I(a₂') = 1 = s_C`; `a₂' ∉ dom(C)` by GlobalUniqueness (ASN-0034) on the content sub-allocator's inc chain; `a₂' ∉ dom(L) = ∅` vacuously. ✓

**Step 3: K.μ⁺ — rebuild the suffix `{[1,2] ↦ a₂', [1,3] ↦ a₃, [1,4] ↦ a₄}`.** Effect: `M_post(d) = {[1,1] ↦ a₁, [1,2] ↦ a₂', [1,3] ↦ a₃, [1,4] ↦ a₄}`. Frame: C', L, E, R unchanged.

Preconditions at the post-K.α intermediate state:
- *d ∈ E_doc; disjoint extension; value preservation.* New positions `{[1,2], [1,3], [1,4]}` are disjoint from `dom(M_int(d)) = {[1,1]}`; the existing mapping at `[1,1]` retains its value `a₁`. ✓
- *K.μ⁺ amendment (content-subspace restriction).* Each new V-position has `subspace(v) = s_C` — first components of `[1,2], [1,3], [1,4]` are all `1 = s_C`. ✓ The amendment scopes the rebuild to the content subspace; on a state with a non-empty link subspace, the same K.μ⁻ + K.μ⁺ replacement pair would re-add only content-subspace positions, leaving the link subspace untouched.
- *Referential integrity (S3 content clause).* `a₂' ∈ dom(C')` (post-K.α); `a₃, a₄ ∈ dom(C) ⊆ dom(C')` by P0 frame on the prior content addresses. ✓
- *S8a, S8-depth, S8-fin on M_post.* New positions have all strictly positive components; `V_{s_C}(d_post) = {[1,1], [1,2], [1,3], [1,4]}` of uniform depth 2; cardinality 4 < ∞. ✓
- *D-CTG★, D-MIN★ on M_post.* `V_{s_C}(d_post)` is contiguous under the V-ordering on `s_C` (every depth-2 positive tuple with first component 1 lex-between `[1,1]` and `[1,4]` — i.e., `[1,2]` and `[1,3]` — is present), with `min = [1, 1] = [s_C, 1]`. ✓

*P4★ status at M_post.* `Contains_C(M_post) = {(a₁, d), (a₂', d), (a₃, d), (a₄, d)}`. K.μ⁺ holds R in frame (R = R_int), so `(a₂', d) ∉ R` at M_post — `Contains_C(M_post) ⊄ R`. Restoration occurs at the K.ρ step.

**Step 4: K.ρ — record provenance for the new address.** Effect: `R' = R ∪ {(a₂', d)}`. Preconditions: `a₂' ∈ dom(C')` (post-K.α); `d ∈ E_doc`. ✓ **P4★ restored**: `(a₂', d) ∈ R'`, so `Contains_C(M_post) ⊆ R'`.

**Composite verification at `Σ →* Σ'`.**

Net change across the composite:
- `dom(C') \ dom(C) = {a₂'}` — one new content address.
- `dom(M'(d)) = dom(M(d)) = {[1,1], [1,2], [1,3], [1,4]}` — the V-position domain returns to its pre-state shape after the K.μ⁻ + K.μ⁺ round-trip.
- `ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) = {a₁, a₂', a₃, a₄} \ {a₁, a₂, a₃, a₄} = {a₂'}` — only `a₂'` is new to d's content-subspace range; `a₃` and `a₄` are re-added but were already in the pre-state range.
- `R' \ R = {(a₂', d)}` — one new provenance entry.

Coupling verification:
- *J0.* `a₂' ∈ dom(C') \ dom(C)`, and the placement clause is witnessed by `M'(d)([1,2]) = a₂'` at d ∈ E'_doc. ✓
- *J1★ (new-address coupling).* Computing the content-subspace ranges at the composite endpoints:
  - *Pre-composite (Σ):* `ran(M(d)|_{s_C}) = {M(d)(v) : v ∈ V_{s_C}(d)} = {M(d)([1,1]), M(d)([1,2]), M(d)([1,3]), M(d)([1,4])} = {a₁, a₂, a₃, a₄}`.
  - *Post-composite (Σ'):* `ran(M'(d)|_{s_C}) = {M'(d)(v) : v ∈ V_{s_C}(d')} = {M'(d)([1,1]), M'(d)([1,2]), M'(d)([1,3]), M'(d)([1,4])} = {a₁, a₂', a₃, a₄}`.
  - *Difference:* `ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) = {a₁, a₂', a₃, a₄} \ {a₁, a₂, a₃, a₄} = {a₂'}`.
  J1★ requires `(a₂', d) ∈ R'`, which K.ρ supplied at Step 4. ✓ The re-added addresses `a₃` and `a₄` are *not* new to d's content-subspace range — they appear in both the pre-state range `{a₁, a₂, a₃, a₄}` and the post-state range `{a₁, a₂', a₃, a₄}` — so J1★ does not require fresh provenance for them, even though they pass through the K.μ⁻ + K.μ⁺ cycle internally. J1★ is range-based and evaluated only between Σ and Σ', so the intermediate dispossession at M_int (where the ranges transit through `ran(M_int(d)|_{s_C}) = {a₁}` and `ran(M_post(d)|_{s_C}) = {a₁, a₂', a₃, a₄}` before reaching `ran(M'(d)|_{s_C})`) is invisible to the coupling.
- *J1'★ (new-provenance check; vacuity on re-added addresses).* The single new provenance entry `(a₂', d) ∈ R' \ R` corresponds to `a₂'` being new to d's content-subspace range, witnessed by the explicit set computation above: `a₂' ∈ ran(M'(d)|_{s_C}) = {a₁, a₂', a₃, a₄}` and `a₂' ∉ ran(M(d)|_{s_C}) = {a₁, a₂, a₃, a₄}`. *Vacuity on re-added addresses:* `a₃` and `a₄` pass through the K.μ⁻ + K.μ⁺ cycle but generate no entries in `R' \ R` — the pre-existing `(a₃, d), (a₄, d) ∈ R` carry through by P2, no fresh K.ρ is invoked for them, and J1'★ therefore has nothing to check for them at the composite boundary; equivalently, neither `a₃` nor `a₄` lies in `ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) = {a₂'}`. ✓

Post-state invariant verification:
- *P4★ (Contains_C ⊆ R).* `Contains_C(Σ') ⊇ {(a₁, d), (a₂', d), (a₃, d), (a₄, d)}`; each pair is in R' — `(a₁, d), (a₃, d), (a₄, d) ∈ R ⊆ R'` by P2, and `(a₂', d) ∈ R'` by K.ρ. The stale pair `(a₂, d) ∈ R' \ Contains_C(Σ')` records that d once contained `a₂`, the historical fact that survives the replacement. ✓
- *P6 (Existential coherence).* `origin(a₂') = d ∈ E_doc`; pre-existing content addresses retain their origin entities by frame. ✓
- *P7 (Provenance grounding).* `(a₂', d) ∈ R'` has `a₂' ∈ dom(C')`; pre-existing R entries retain their grounding by P0. ✓
- *P7a (Provenance coverage).* every `a ∈ dom(C')` has at least one provenance entry — `a₁, a₂, a₃, a₄` retain their pre-state entries (R ⊆ R' by P2), and `a₂'` has the freshly added `(a₂', d)`. ✓
- *D-CTG★, D-MIN★ at Σ'.* `V_{s_C}(d') = {[1,1], [1,2], [1,3], [1,4]}` contiguous, minimum `[1,1] = [s_C, 1]`. ✓


## Worked example: prior-provenance and first-time-transcluded replacements

We trace the *two-step* and *three-step* replacement composite variants — distinct from the four-step *fresh-content* form exercised in *Worked example: interior content replacement* above by the I-address class involved. Both variants re-use a pre-existing `dom(C)` address (no K.α), but they differ in whether `d` has prior provenance for it: the two-step form uses `(aₓ, d) ∈ R` (P2-preserved from a prior insertion-deletion cycle), the three-step form adds a trailing K.ρ to record first-time provenance.

*Common pre-state Σ_a.* Document `d = 1.0.1.0.1 ∈ E_doc`, content store `C ⊇ {a₁ ↦ char₁, a₂ ↦ char₂, aₓ ↦ charₓ}`, arrangement `M(d) = {[1,1] ↦ a₁, [1,2] ↦ a₂}` (so `V_{s_C}(d) = {[1,1], [1,2]}`, satisfying D-CTG★/D-MIN★/D-SEQ★ at `n_{s_C} = 2`). The replacement target `aₓ` is already in `dom(C)` in each variant, but the two variants fix *different* concrete addresses for it because their origin requirements differ: the two-step form needs `aₓ` to be `d`'s own previously-arranged content (`origin(aₓ) = d`), while the three-step form needs `aₓ` to be foreign-origin content `d` has never displayed (`origin(aₓ) = d_src ≠ d`). Each variant states its literal below. Goal in each: replace `[1,2] ↦ a₂` with `[1,2] ↦ aₓ`.

**Two-step variant — prior-provenance replacement.** Here `aₓ = 1.0.1.0.1.0.1.5`, with tumbler `[1,0,1,0,1,0,1,5]` (zeros at positions 2,4,6), so `origin(aₓ) = N.0.U.0.D = 1.0.1.0.1 = d` — `aₓ` is `d`'s own content. Σ_a additionally has `(aₓ, d) ∈ R` (aₓ was previously arranged at some V-position of d, K.μ⁻ removed it, but P2 retained the entry). Composite: K.μ⁻ (remove `[1,2]`) → K.μ⁺ (add `[1,2] ↦ aₓ`). Net change: `dom(C') = dom(C)` (no K.α), `dom(M'(d)) = {[1,1], [1,2]}` (the V-position domain returns to its pre-state shape), `ran(M'(d)|_{s_C}) = {a₁, aₓ}`, `R' = R`.

*Step 1: K.μ⁻ — remove `[1,2]`.* Caller chooses `(n'_{s_C}, n'_{s_L}) = (1, 0)` against pre-state `(n_{s_C}, n_{s_L}) = (2, 0)`. Strict contraction `1 < 2` on the content subspace; K.μ⁻ fires. `M_int(d) = {[1,1] ↦ a₁}`.

*Step 2: K.μ⁺ — add `[1,2] ↦ aₓ`.* Precondition `aₓ ∈ dom(C_int) = dom(C)` ✓ (pre-existing in `dom(C)`); new V-position `[1,2]` satisfies the K.μ⁺ amendment (`subspace([1,2]) = s_C`) and is disjoint from `dom(M_int(d)) = {[1,1]}`. K.μ⁺ fires, producing `M'(d) = {[1,1] ↦ a₁, [1,2] ↦ aₓ}`. D-CTG★, D-MIN★, D-SEQ★ at the post-state read off `V_{s_C}(d') = {[1,1], [1,2]}` directly.

*Composite coupling verification at Σ →* Σ'.*
- *J0:* `dom(C') = dom(C)`, no fresh content allocation — vacuous. ✓
- *J1★ (range-new content-subspace coupling):* `ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) = {a₁, aₓ} \ {a₁, a₂} = {aₓ}`. J1★ requires `(aₓ, d) ∈ R'`. Pre-state has `(aₓ, d) ∈ R ⊆ R'` (P2). ✓ — *no K.ρ is invoked, and J1★ is discharged by the pre-state membership alone.* This is what distinguishes the two-step form: the substantive precondition `(aₓ, d) ∈ R` at the pre-state is what makes K.ρ unnecessary at the composite boundary.
- *J1'★ (new-provenance check):* `R' \ R = ∅` (no K.ρ in the composite). Vacuous. ✓

*Post-state invariants:* S2 (the new mapping at `[1,2]` is single-valued and at a position pairwise distinct from `[1,1]`); S3★ (the new content-subspace mapping targets `aₓ ∈ dom(C')`); P4★ (`Contains_C(Σ') ⊆ R'` holds because `(aₓ, d) ∈ R ⊆ R'`; the stale `(a₂, d) ∈ R' \ Contains_C(Σ')` records the historical fact that d once contained a₂); D-CTG★/D-MIN★/D-SEQ★ on `V_{s_C}(d') = {[1,1], [1,2]}` (the canonical shape at `n_{s_C} = 2`). The historical asymmetry is concrete: `R' = R ⊇ {(a₂, d), (aₓ, d)}` — both pre-state entries persist — while `Contains_C(Σ') = {(a₁, d), (aₓ, d)}` reflects only the current arrangement. ∎

**Three-step variant — first-time transcluded replacement.** Here `aₓ = 1.0.1.0.2.0.1.5`, with tumbler `[1,0,1,0,2,0,1,5]` (zeros at positions 2,4,6), so `origin(aₓ) = N.0.U.0.D = 1.0.1.0.2 = d_src ≠ d` and `subspace_I(aₓ) = E(aₓ)₁ = 1 = s_C` (still content-subspace). Σ_a has `(aₓ, d) ∉ R` — aₓ was allocated by another document `d_src = 1.0.1.0.2 ≠ d` (its origin), and recorded as `(aₓ, d_src) ∈ R`, but d has never previously arranged aₓ. (The discriminator for `(aₓ, d) ∉ R` is whether `aₓ` was ever arranged in `d`, not its origin: foreign origin is one way to obtain `(aₓ, d) ∉ R`, but a `d`-origin address that K.μ⁺ only ever transcluded into some `d' ≠ d` is likewise never arranged in `d` and so also has `(aₓ, d) ∉ R` while `aₓ ∈ dom(C)`.) Composite: K.μ⁻ (remove `[1,2]`) → K.μ⁺ (add `[1,2] ↦ aₓ`) → K.ρ (record `(aₓ, d)`). Net change: `dom(C') = dom(C)`, `ran(M'(d)|_{s_C}) = {a₁, aₓ}`, `R' = R ∪ {(aₓ, d)}`.

*Steps 1 and 2 (K.μ⁻, K.μ⁺):* identical to the two-step variant's Steps 1 and 2. After K.μ⁺, the intermediate state `Σ_post-K.μ⁺` has `Contains_C(Σ_post-K.μ⁺) ⊇ {(aₓ, d)}` but `R_post-K.μ⁺ = R ∌ (aₓ, d)` — *P4★ transiently fails* at this composite-internal state. ValidComposite★ allows the transient failure; restoration comes at Step 3.

*Step 3: K.ρ — record `(aₓ, d)`.* Preconditions: `aₓ ∈ dom(C_post-K.μ⁺) = dom(C)` ✓; `d ∈ E_doc` ✓. K.ρ fires, producing `R' = R ∪ {(aₓ, d)}`. **P4★ restored**: `Contains_C(Σ') ⊆ R'` because the new pair `(aₓ, d)` is now in `R'`.

*Composite coupling verification at Σ →* Σ'.*
- *J0:* `dom(C') = dom(C)`, no fresh content allocation — vacuous. ✓
- *J1★:* `ran(M'(d)|_{s_C}) \ ran(M(d)|_{s_C}) = {aₓ}`. J1★ requires `(aₓ, d) ∈ R'`. Pre-state has `(aₓ, d) ∉ R`. K.ρ at Step 3 supplies `(aₓ, d) ∈ R'`. ✓ — *the K.ρ step is required*; without it, J1★ would fail at the composite boundary, invalidating the composite under ValidComposite★. The substantive precondition `(aₓ, d) ∉ R` at the pre-state is what forces the third K.ρ step versus the two-step form's pre-state membership.
- *J1'★:* `R' \ R = {(aₓ, d)}`. For the entry `(aₓ, d)`, the witnessing content-subspace V-position is `[1,2]` at the post-state: `aₓ ∈ ran(M'(d)|_{s_C})` (since `M'(d)([1,2]) = aₓ` with `subspace([1,2]) = s_C`) and `aₓ ∉ ran(M(d)|_{s_C}) = {a₁, a₂}`. The K.ρ entry is anchored in the K.μ⁺ extension. ✓

*Post-state invariants:* S2, S3★, D-CTG★/D-MIN★/D-SEQ★ discharged analogously to the two-step variant; P4★ differs — it transiently fails at `Σ_post-K.μ⁺` (Step 2) and is restored only by the trailing K.ρ at Step 3, as noted above, whereas in the two-step variant `(aₓ, d) ∈ R` pre-exists and P4★ holds at every intermediate state. The notable difference at the historical layer: `R' = R ∪ {(aₓ, d)}` introduces a fresh provenance pair (`d` is now historically recorded as having contained aₓ for the first time), whereas the two-step variant added no new entry to R. ∎


## Worked example: link allocation and arrangement

We verify the central postconditions on concrete tumbler values. By SubspaceConventionAxiom (FixedSubspaceIdentifiers), `s_C = 1` and `s_L = 2` throughout (and SC-NEQ `1 ≠ 2` is satisfied automatically). Consider document `d` at address `1.0.1.0.1` with two text content addresses allocated and arranged.

*Initial state.* `dom(C) = {1.0.1.0.1.0.1.1, 1.0.1.0.1.0.1.2}`, `dom(L) = ∅`, `E_doc = {1.0.1.0.1}`, `R = {(1.0.1.0.1.0.1.1, d), (1.0.1.0.1.0.1.2, d)}` (implicit from prior J0/J1★ of allocation).

Arrangement: `M(d) = {[1,1] ↦ 1.0.1.0.1.0.1.1, [1,2] ↦ 1.0.1.0.1.0.1.2}`.

Text-subspace V-positions: `V_1(d) = {[1,1], [1,2]}` — contiguous (D-CTG★), minimum at `[1,1]` (D-MIN★), depth 2 (S8-depth). Link subspace: `V_2(d) = ∅`.

**Step 1: K.λ — allocate link.** Create link `ℓ = 1.0.1.0.1.0.2.1` with value `(F, G, Θ)`.

Precondition verification:
- `d = 1.0.1.0.1 ∈ E_doc`
- `ℓ ∉ dom(L) ∪ dom(C)`: this is the first-emission case (no prior d-origin link), so freshness is SubAllocFresh at `x = L` (seed part for `dom(L)`; cross-subspace part — ℓ's element field `2.1` against content's `1.1`, `1.2` — for `dom(C)`)
- `zeros(ℓ) = 3`: zeros at positions 2, 4, 6 in the tumbler `1.0.1.0.1.0.2.1`
- `subspace_I(ℓ) = 2 = s_L`
- `origin(ℓ) = 1.0.1.0.1 = d`
- Forward allocation: no prior links in dom(L) with origin d, so vacuously satisfied
- `(F, G, Θ) ∈ Link` by assumption (L3)

Effect: `L' = {1.0.1.0.1.0.2.1 ↦ (F, G, Θ)}`. Frame: C, E, M, R unchanged.

Post-state verification:
- L14: `dom(C) ∩ dom(L') = ∅` — content addresses have `subspace_I(a) = 1`, link has `subspace_I(ℓ) = 2`, and `1 ≠ 2`
- L0: all dom(L') addresses have subspace s_L = 2; all dom(C) addresses have subspace s_C = 1
- L3: `L'(ℓ) = (F, G, Θ)` with `F, G, Θ ∈ Endset`
- L-fin: `dom(L') = {ℓ}` is a singleton, hence finite. ✓
- *L11a (Link distinctness for this K.λ event)*: this is the *first-link case* for `d` — the K.λ precondition predicate `{ℓ' ∈ dom(L) : origin(ℓ') = d} = ∅` holds because `dom(L) = ∅` at the pre-state, so the first-emission path of K.λ applies. The emitted address `ℓ = [d.0.s_L.1] = 1.0.1.0.1.0.2.1` is the first emission of d's link sub-allocator `A_L(d)`, and **FirstEmissionFreshness** (ASN-0093) directly supplies `ℓ ∉ dom(Σ.L) ∪ dom(Σ.C)` at the state of allocation — discharging both the freshness conjunct of K.λ's precondition and the L11a obligation that distinct K.λ events produce distinct link addresses. Subsequent emissions of `A_L(d)` discharge L11a distinctness by GlobalUniqueness (ASN-0034) on its inc chain. ✓
- S3★, CL-OWN: M unchanged, hold from pre-state

**Step 2: K.μ⁺_L — arrange the link in d.** Map the newly allocated `ℓ` into d's link subspace at the minimum link V-position.

Precondition verification:
- `d ∈ E_doc`
- `ℓ = 1.0.1.0.1.0.2.1 ∈ dom(L')`
- `origin(ℓ) = 1.0.1.0.1 = d`
- `ℓ ∉ ran(M(d))`: pre-state `M(d) = {[1,1] ↦ a₁, [1,2] ↦ a₂}` has range `{a₁, a₂} ⊆ dom(C)`; since `ℓ ∈ dom(L)` and `dom(L) ∩ dom(C) = ∅` (L14), `ℓ ∉ ran(M(d))`
- `subspace(v_ℓ) = 2 = s_L`
- `V_{s_L}(d) = ∅`, so this first link-subspace insertion fixes `m_L(d) = 2` (any value ≥ 2 is admissible; we take 2 here), giving `v_ℓ = [s_L, 1] = [2, 1]` (D-MIN★ for empty link subspace)
- `#v_ℓ = 2 = m_L(d)` (S8-depth)

Effect: `M'(d) = {[1,1] ↦ 1.0.1.0.1.0.1.1, [1,2] ↦ 1.0.1.0.1.0.1.2, [2,1] ↦ 1.0.1.0.1.0.2.1}`.

Post-state verification:
- S3★: `subspace([1,1]) = 1 = s_C` and `M'(d)([1,1]) = 1.0.1.0.1.0.1.1 ∈ dom(C)`; `subspace([1,2]) = 1 = s_C` and `M'(d)([1,2]) = 1.0.1.0.1.0.1.2 ∈ dom(C)`; `subspace([2,1]) = 2 = s_L` and `M'(d)([2,1]) = 1.0.1.0.1.0.2.1 ∈ dom(L')`
- CL-OWN: the only link-subspace position is `[2,1]` with `origin(M'(d)([2,1])) = origin(1.0.1.0.1.0.2.1) = 1.0.1.0.1 = d`
- D-CTG★: `V_1(d) = {[1,1], [1,2]}` contiguous; `V_2(d) = {[2,1]}` singleton, trivially contiguous
- D-MIN★: `min(V_1(d)) = [1,1] = [s_C, 1]`; `min(V_2(d)) = [2,1] = [s_L, 1]`
- L14: subspace identifiers 1 and 2 are distinct (SC-NEQ), so dom(C) ∩ dom(L') = ∅
- L-fin: dom(L') = {ℓ} is unchanged from Step 1; still finite. ✓

**Step 3: K.μ~ — reorder text, verify link fixity.** Swap text: `π([1,1]) = [1,2]`, `π([1,2]) = [1,1]`. Clause (v) link-subspace fixity (realised via **LRP** + CL-UNIQ) forces `π([2,1]) = [2,1]`.

Let `a₁ = 1.0.1.0.1.0.1.1` and `a₂ = 1.0.1.0.1.0.1.2`. Pre-state: `M'(d) = {[1,1] ↦ a₁, [1,2] ↦ a₂, [2,1] ↦ ℓ}`. Firing precondition: `M'(d)|_{dom_C} = {[1,1] ↦ a₁, [1,2] ↦ a₂}` with a₁ ≠ a₂ — two distinct values, so K.μ~ fires. K.μ~ decomposes as K.μ⁻ (full content-subspace clearance, retaining `[2,1]`) followed by K.μ⁺ (re-adding `{[1,1] ↦ a₂, [1,2] ↦ a₁}`). Intermediate-state admissibility discharges from the K.μ~ Decomposition section's checks.

Post-state: `M''(d) = {[1,1] ↦ a₂, [1,2] ↦ a₁, [2,1] ↦ ℓ}`.

Post-state verification:
- *S3★:* `subspace([1,1]) = 1 = s_C` and `M''(d)([1,1]) = a₂ ∈ dom(C)`; `subspace([1,2]) = 1 = s_C` and `M''(d)([1,2]) = a₁ ∈ dom(C)`; `subspace([2,1]) = s_L` and `M''(d)([2,1]) = ℓ ∈ dom(L')`. ✓
- *L14:* dom(C) ∩ dom(L') = ∅ unchanged from Step 2. ✓
- *L-fin:* dom(L') = {ℓ} unchanged; still finite. ✓
- *D-CTG★/D-MIN★:* V_{s_C}(d) = {[1,1], [1,2]} and V_{s_L}(d) = {[2,1]} are both unchanged from Step 2 (K.μ~ preserves dom by K.μ~-FIX); contiguity and minima are inherited.
- *CL-OWN:* the link-subspace mapping is fixed pointwise, so origin(M''(d)([2,1])) = origin(ℓ) = d remains satisfied. ✓

**Step 4: K.λ + K.μ⁺_L — allocate and arrange a second link.** To exercise link-subspace contraction below we need a non-singleton link subspace. Allocate `ℓ₂ = 1.0.1.0.1.0.2.2 = inc(ℓ, 0)` (the next sibling on d's link frontier under TA5(c), per K.λ's subsequent-link case) with some value `(F', G', Θ')`; then arrange `ℓ₂` at `v_{ℓ₂} = shift(max(V_{s_L}(d)), 1) = shift([2,1], 1) = [2,2]` (D-CTG★ case of K.μ⁺_L).

Effect after both transitions: `L = {ℓ ↦ (F, G, Θ), ℓ₂ ↦ (F', G', Θ')}`, `M''(d) = {[1,1] ↦ a₂, [1,2] ↦ a₁, [2,1] ↦ ℓ, [2,2] ↦ ℓ₂}`. Link-subspace V-positions: `V_{s_L}(d) = {[2,1], [2,2]}` — contiguous (D-CTG★), minimum at `[2,1] = [s_L, 1]` (D-MIN★), depth 2 (S8-depth), structural form `{[s_L, k] : 1 ≤ k ≤ 2}` (D-SEQ★ with `n_{s_L} = 2`, `m_{s_L} = 2`).

Post-state verification (for the K.λ + K.μ⁺_L composite):
- *L11a (Link distinctness for this K.λ event)*: this is the *subsequent-link case* for `d` — the K.λ precondition predicate `{ℓ' ∈ dom(L) : origin(ℓ') = d} ≠ ∅` holds because Step 1's `ℓ ∈ dom(L)` has `origin(ℓ) = d`, so the subsequent-emission path of K.λ applies. The emitted address `ℓ₂ = inc(max{ℓ' ∈ dom(L) : origin(ℓ') = d}, 0) = inc(ℓ, 0) = 1.0.1.0.1.0.2.2` advances `A_L(d)`'s frontier by one sibling-increment via TA5(c). Freshness `ℓ₂ ∉ dom(L) ∪ dom(C)` is SubAllocFresh at `x = L` (frontier-advance part against `dom(L)`, cross-subspace part against `dom(C)`), giving `ℓ₂ ≠ ℓ` — the L11a obligation that this K.λ event produce an address distinct from every prior K.λ event under d (here, the single prior event at Step 1 producing ℓ). ✓
- *S3★:* the new link-subspace position `[2,2]` has `subspace([2,2]) = s_L` and maps to `ℓ₂ ∈ dom(L')`; existing positions retain their pre-state values. ✓
- *CL-OWN:* `origin(M''(d)([2,2])) = origin(ℓ₂) = d` (K.λ's `origin(ℓ₂) = d` precondition combined with the K.μ⁺_L placement). ✓
- *CL-UNIQ:* `ℓ₂` is fresh to `dom(L)` (K.λ's allocation precondition), so no prior V-position references it; the new V-position `[2,2]` is therefore the unique link-subspace V-position mapping to `ℓ₂`. ✓
- *L0/L1/L1a/L3/L-fin:* each established for `ℓ₂` by K.λ's preconditions and inherited at the post-state.
- *L14:* `dom(C) ∩ dom(L') = ∅` — the new link `ℓ₂` has `subspace_I(ℓ₂) = s_L = 2`, distinct from `s_C = 1`. ✓

**Step 5: K.μ⁻ — admissible suffix removal of links.** Remove the mapping at `[2,2]` — the maximum end of `V_{s_L}(d)`, a 1-element suffix of the link-subspace range.

*Constructive precondition (caller's choice, verified at the pre-state).* K.μ⁻'s constructive precondition asks the caller to commit to per-subspace retention counts `(n'_{s_C}, n'_{s_L})` against the pre-state values `(n_{s_C}, n_{s_L}) = (2, 2)` read from D-SEQ★ at Σ (the pre-state Σ has `V_{s_C}(d) = {[1,1], [1,2]}` and `V_{s_L}(d) = {[2,1], [2,2]}`, both matching the canonical `{[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` form at depth 2). For the goal of removing the maximum link-subspace position, the caller chooses `(n'_{s_C}, n'_{s_L}) = (2, 1)`. Precondition checks at the pre-state:
- `d ∈ E_doc`: ✓ (carried forward from earlier steps).
- Per-subspace retention counts in admissible range: `n'_{s_C} = 2 ∈ {0, 1, 2} = {0, …, n_{s_C}}`, ✓; `n'_{s_L} = 1 ∈ {0, 1, 2} = {0, …, n_{s_L}}`, ✓.
- Strict contraction on at least one subspace: `n'_{s_L} = 1 < 2 = n_{s_L}` (the link subspace shrinks strictly), ✓; the content subspace is held fixed (`n'_{s_C} = n_{s_C}`). (`dom(M(d)) ≠ ∅` is not a separate check — strict contraction forces it, per K.μ⁻'s definition.)

The precondition is satisfied; K.μ⁻ fires. The retained domain is `R := ∪_{S ∈ {s_C, s_L}} {[S, 1, ..., 1, k] : 1 ≤ k ≤ n'_S} = {[1,1], [1,2], [2,1]}`, and the constructive effect is `M'''(d) = M''(d) ↾ R`.

*Post-state shape (derived from the constructive precondition).* `dom(M'''(d)) = {[1,1], [1,2], [2,1]} ⊂ dom(M''(d))`. Surviving mappings unchanged: `M'''(d)([1,1]) = a₂`, `M'''(d)([1,2]) = a₁`, `M'''(d)([2,1]) = ℓ`. The content subspace is unchanged: `V_{s_C}(d') = {[1,1], [1,2]}`. The link subspace contracts to a 1-element suffix prefix: `V_{s_L}(d') = {[2,1]}`. Per-subspace shape:
- *Content subspace.* `V_{s_C}(d') = V_{s_C}(d)` — zero-suffix removal at `n'_{s_C} = 2`.
- *Link subspace.* `V_{s_L}(d') = {[2,1]}` — 1-element suffix-prefix retention at `n'_{s_L} = 1`.

Post-state invariant verification:
- *S3★:* surviving mappings retain their pre-state values; `[2,1] ↦ ℓ ∈ dom(L)` satisfies the link clause. ✓
- *D-CTG★:* `V_{s_C}(d') = {[1,1], [1,2]}` and `V_{s_L}(d') = {[2,1]}` are each contiguous. ✓
- *D-MIN★:* `min(V_{s_C}(d')) = [1,1] = [s_C, 1]`; `min(V_{s_L}(d')) = [2,1] = [s_L, 1]`. ✓
- *D-SEQ★:* `V_{s_L}(d') = {[s_L, 1]}` matches `{[s_L, k] : 1 ≤ k ≤ 1}` at `n_{s_L} = 1` (`m_{s_L} = 2`). ✓
- *CL-OWN:* `origin(M'''(d)([2,1])) = origin(ℓ) = d` (preserved from pre-state by frame on the surviving mapping). ✓
- *CL-UNIQ:* the surviving link-subspace mapping is the singleton `{[2,1] ↦ ℓ}`; vacuously injective. ✓
- *L12:* `dom(L)` unchanged — `ℓ₂` remains in `dom(L)` despite no longer being arranged. ✓ This is the *orphan link* state Nelson identifies (LM 4/9): `ℓ₂ ∈ dom(L)` but `ℓ₂ ∉ ran(M'''(d))` for any d.

An attempt to remove `[2,1]` while retaining `[2,2]` is excluded by D-MIN★ (the missing-minimum shape); an attempt to remove an interior position while retaining both endpoints is excluded by D-CTG★ (the interior-hole shape).


## Cross-layer invariants

**P6 (Existential coherence).** For every I-address in the content store, its origin document exists as an entity:

`(A a ∈ dom(C) :: origin(a) ∈ E_doc)`

*Derivation.* K.α allocates a under origin(a)'s prefix (S7a, ASN-0036), and requires origin(a) ∈ E_doc as a precondition — the allocation mechanism inc(·, k) operates on an existing tumbler within the ownership domain. P1 preserves entity membership across subsequent transitions; P0 preserves a ∈ dom(C). Initial state: dom(C₀) = ∅, so the quantifier is vacuously satisfied. Inductive step: each K.α has origin(a) ∈ E_doc by precondition; P0 preserves a; P1 preserves origin(a). ∎

**P7 (Provenance grounding).** Every provenance entry references allocated content:

`(A (a, d) ∈ R :: a ∈ dom(C))`

*Derivation.* K.ρ requires a ∈ dom(C) as a precondition. P0 preserves dom(C). By induction: initially R₀ = ∅ (vacuous). Each K.ρ adds (a, d) with a ∈ dom(C); P0 ensures a remains in dom(C') for all subsequent states; P2 ensures (a, d) remains in R'. ∎

**P7a (Provenance coverage).** Every I-address in the content store has at least one provenance record, `(A a ∈ dom(C) :: (E d :: (a, d) ∈ R))`.

*Derivation.* Composite-boundary property, discharged in the *Class (b)* proof below (P7a per-property paragraph). ∎


## Extended reachable-state invariants

The atomicity guarantee of SequentialTransitionAxiom commits *elementary* transitions to single-event atomicity — not composites. A composite Σ →* Σ' is a sequence of atomic elementary transitions, and the intermediate states between elementary steps are real, observable states of the transition system. Properties of the reachable-state space therefore partition by *temporal scope* into two classes:

- *Per-state invariants* hold at **every** reachable state — every initial state, every elementary-transition target state, every intermediate state within a composite. Each elementary transition preserves these individually, so they are true invariants of the elementary transition system.
- *Composite-boundary properties* hold only at *composite boundaries* (the initial Σ and final Σ' of any valid composite) and may transiently fail at intermediate states within a composite. They are not invariants of the elementary transition system — they are properties guaranteed by the J0/J1★/J1'★ couplings of ValidComposite★, restored at the close of each valid composite rather than preserved by each elementary step. We name them *composite-boundary properties* throughout.

**ExtendedReachableStateInvariants.** Every state reachable from Σ₀ by a finite sequence of *elementary* transitions drawn from valid composites satisfies the *per-state invariants* below; every state at a composite boundary additionally satisfies the *composite-boundary properties* below.

  *Per-state invariants* (Class (a) of the proof below; temporal scope per the section preamble above):

  S2 ∧ S3★ ∧ S3★-aux ∧ S4 ∧ S7a ∧ S7b ∧ C1b ∧ C1c ∧ S7d ∧ S8a ∧ S8-fin ∧ S8-depth ∧ S8★ ∧ C-fin ∧ D-CTG★ ∧ D-MIN★ ∧ D-SEQ★ ∧ P6 ∧ P7 ∧ P8 ∧ NodeLineage ∧ ActivatedEmission ∧ L0 ∧ L1 ∧ L1a ∧ L1b ∧ L1c ∧ L3 ∧ L14 ∧ L-fin ∧ CL-OWN ∧ CL-UNIQ

  All per-state invariants except ActivatedEmission are discharged cell-by-cell in the Class (a) verification matrix below; ActivatedEmission is the one per-state invariant discharged separately, by the self-contained induction in its definition box.

  *Composite-boundary properties* (Class (b) of the proof below; temporal scope per the section preamble above):

  P4★ ∧ P4a ∧ P7a

The *Composite-boundary verification matrix* below records, for each Class (b) property, where it transiently fails and the coupling that restores it. P4★ and P7a are composite-boundary state-predicates; P4a is grouped with them as a trace property, with its witnessing domain made formal in its definition box.

ASN-0036's S7d (document allocation discipline) is preserved unchanged: every `d ∈ E_doc` is T4-valid with `zeros(d) = 2`, placed in E_doc by a K.δ event whose freshness guard `e ∉ E` and GlobalUniqueness distinctness preservation are the case-(ii) preconditions discharged at the K.δ definition.

**ExtendedTransitionInvariants (per-transition).** Every valid composite transition `Σ →* Σ'` satisfies:

  P3

P3 (defined in *Destruction confinement*) covers every per-transition monotonicity obligation, so naming P3 alone suffices here.

*Proof.* The two temporal-scope classes of the section preamble carry different proof obligations and so demand different induction variables. Class (a) per-state invariants are proved by induction over *elementary* transitions, with the per-elementary verification matrix below as the inductive step; this reaches the intermediate states of each composite directly, which an induction on composite count would skip. Class (b) composite-boundary properties are proved by induction on the number of valid composite transitions from Σ₀, the appropriate coarser variable. The per-transition invariants are addressed last, in a single elementary-case check.

**Base.** The extended initial state Σ₀ satisfies every per-state invariant (verified in *The state model* under "Initial state invariant verification" — L₀ = ∅ satisfies link invariants vacuously, including L3; S3★ reduces to S3, and P4★ reduces to the unscoped bound `Contains(Σ) ⊆ R` (over the link-free initial fragment `Contains = Contains_C`, per the *Scoped coupling constraints* discussion); S3★-aux holds vacuously since M₀(d) = ∅ for all d; D-CTG★, D-MIN★ (and the derived D-SEQ★) hold vacuously since V_S(d) = ∅ for every subspace S). The per-transition invariants have no base case — they are vacuous before any transition has occurred — and enter the induction at the first step.

**Class (a): Per-state invariants.** These are the *per-state invariants* enumerated in the ExtendedReachableStateInvariants definition above — every reachable-state property except the Class (b) composite-boundary triple P4★, P4a, P7a and ActivatedEmission.

*Verification matrix.* Each cell names the load-bearing discharge for that (invariant, transition) pair; `frame` indicates the transition holds the relevant state component unchanged and so trivially preserves the invariant; `n/a` indicates the invariant's scope does not intersect the transition's effect (e.g., L0's L-clause is `n/a` for transitions that frame both L and C). The matrix columns are the *elementary* transitions only. K.μ~ is a K.μ⁻+K.μ⁺ composite and so inherits both of those columns.

| Invariant | K.α | K.δ | K.λ | K.μ⁺ | K.μ⁺_L | K.μ⁻ | K.ρ |
|-----------|-----|-----|-----|------|--------|------|-----|
| S2 | frame | frame (M(e)=∅ on new entity disjoint) | frame | new positions disjoint from dom(M(d)) by value-preservation clause | new v_ℓ ∉ dom(M(d)) (verified at K.μ⁺_L) | restriction of M(d) | frame |
| S3★ | frame | frame (Node/Account); new doc M'(e)=∅ (vacuous, Document) | frame | amendment: subspace(v)=s_C ⟹ M'(d)(v)∈dom(C); link clause framed | precondition: ℓ∈dom(L) and subspace(v_ℓ)=s_L | restriction (values unchanged) | frame |
| S3★-aux | frame | frame (Node/Account); new doc M'(e)=∅ (vacuous, Document) | frame | amendment: new positions have subspace s_C | precondition: new v_ℓ has subspace s_L | restriction (subspaces of survivors unchanged) | frame |
| S4 (content distinctness, per ASN-0036) | SubAllocFresh at x = C | frame (does not add to dom(C)) | frame (does not add to dom(C)) | frame (no new addresses) | frame | frame | frame |
| Entity distinctness (derived; distinct K.δ events ⟹ distinct e) | frame (does not add to E) | same-event freshness e∉E is the case-(ii) precondition at the K.δ definition (NodeBaptism (a) for nodes); cross-document by CrossDocEntityDisjoint (subsuming CrossNodeAccountBase); same-parent cross-chain by GlobalUniqueness/T10a.6 | frame | frame | frame | frame | frame |
| L11a (link distinctness, derived; distinct K.λ events ⟹ distinct ℓ) | frame | frame | same-event freshness ℓ∉dom(L)∪dom(C) is SubAllocFresh at x = L; cross-document by CrossDocDisjoint at the link-anchor pair (b_L(d₁), b_L(d₂)) | frame | frame | frame | frame |
| S7a | precondition: origin(a)∈E_doc | frame (e∉dom(C)) | frame | frame | frame | frame | frame |
| S7b | K.α's `zeros(a) = 3` precondition (per ASN-0093) | frame | frame (link addresses not in dom(C)) | frame | frame | frame | frame |
| C1b (ASN-0093) | K.α allocator chain produces E(a)=[s_C,k] ⟹ #E(a)≥2 | frame | frame | frame | frame | frame | frame |
| C1c (ASN-0093) | structural inc-chain established per K.α allocation (first via SubAllocatorBundle, subsequent via TA5(c)); preserved by P0 | frame | frame | frame | frame | frame | frame |
| S7d | frame | establishes new d∈E_doc via T10a per-`(t,k')` discipline; preserved by P1 | frame | frame | frame | frame | frame |
| S8a, S8-depth, S8-fin | frame | new doc has M(d)=∅ (vacuous) | frame | precondition: positivity, depth, finite extension | precondition: positivity, depth m_L(d), finite | restriction preserves all three | frame |
| S8★ | frame | new doc M(d)=∅ (vacuous) | frame | re-apply ASN-0036's S8 to extended M(d')\|_{V_{s_C}(d')} | length-1 decomposition on extended M(d')\|_{V_{s_L}(d')} | s_C: re-apply ASN-0036's S8 to contracted M'(d)\|_{V_{s_C}(d)}; s_L: length-1 decomposition on survivors | frame |
| D-CTG★ / D-MIN★ | frame | frame | frame | K.μ⁺ precondition discharge | K.μ⁺_L preconditions (empty/shift cases) | satisfied by construction (contraction shape at K.μ⁻ definition) | frame |
| D-SEQ★ | frame | frame | frame | derived from D-CTG★ + D-MIN★ + S8-depth + S8-fin + S8a | derived | derived | frame |
| P6 | precondition: origin(a)∈E_doc; preserved by P0/P1 | frame (does not add to dom(C)) | frame | frame | frame | frame | frame |
| P7 | frame (does not add to R) | frame | frame | frame | frame | frame | precondition: a∈dom(C); preserved by P0 |
| P8 | frame | parent(e)∈E precondition for ¬Node(e); vacuous for Node | frame | frame | frame | frame | frame |
| NodeLineage | frame | K.δ case (i): n₀≼e from NodeBaptism (b); case (ii): outside Node scope | frame | frame | frame | frame | frame |
| L0 (C-clause) | K.α's `E(a)₁ = s_C` precondition (per ASN-0093): subspace_I(a)=s_C; preserved by P0 | frame | frame | frame | frame | frame | frame |
| L0 (L-clause) | frame | frame | precondition: subspace_I(ℓ)=s_L; preserved by L12 | frame | frame | frame | frame |
| L1, L1a, L1b | frame | frame | preconditions (zeros(ℓ)=3, origin(ℓ)∈E_doc, #E(ℓ)≥2); preserved by L12 | frame | frame | frame | frame |
| L1c | frame | frame | structural inc-chain established per K.λ allocation (first via SubAllocatorBundle, subsequent via TA5(c)); preserved by L12 | frame | frame | frame | frame |
| L3 | frame | frame | precondition: N ≥ 3 ∧ (A i : 1 ≤ i ≤ N : eᵢ ∈ Endset) ∧ e₃ ≠ ∅; preserved by L12 | frame | frame | frame | frame |
| L14 | K.α's `E(a)₁ = s_C` precondition (per ASN-0093) ⟹ subspace_I(a)=s_C ≠ s_L; preserves disjointness | frame | precondition ℓ∉dom(C); subspace_I(ℓ)=s_L preserves disjointness | frame | frame | frame | frame |
| L-fin | frame | frame | extends dom(L) by one (finite + 1 = finite) | frame | frame | frame | frame |
| C-fin | extends dom(C) by one (finite + 1 = finite) | frame | frame | frame | frame | frame | frame |
| CL-OWN | frame | frame (Node/Account); new doc M'(e)=∅ (vacuous, Document) | frame | frame (no link-subspace V-positions added) | precondition: origin(ℓ)=d | frame (survivors retain origin) | frame |
| CL-UNIQ | frame | frame (Node/Account); new doc M'(e)=∅ (vacuous, Document) | frame | frame (no link-subspace V-positions added; M(d)\|_{dom_L} unchanged) | precondition: ℓ∉ran(M(d)) ensures unique placement | restriction of an injection remains injective | frame |

The matrix is a navigational index; each cell summarises the load-bearing argument.

*S2 (ArrangementFunctionality).* K.μ⁺ adds at V-positions disjoint from dom(M(d)) (the K.μ⁺ definition's value-preservation clause forces new mappings to disjoint positions, so extending a partial function at disjoint elements preserves single-valuedness); K.μ⁺_L adds at `v_ℓ ∉ dom(M(d))` (verified in *Link-subspace extension*); K.μ⁻ restricts M(d) (restriction of a function is a function); all others hold M in frame.

*S3★ (GeneralizedReferentialIntegrity)* and *S3★-aux (SubspaceExhaustiveness).* The two invariants are discharged together here. *Base* (Σ₀): `M₀ = ∅`, so both hold vacuously. *Step:* K.α, K.ρ hold M in frame; K.δ holds M in frame in its Node/Account cases and, in its Document case, adds only the new empty arrangement `M'(e) = ∅` (vacuous over its empty domain); K.μ⁺ (amended) creates only content-subspace positions (`subspace(v) = s_C`), targeting dom(C); K.μ⁺_L creates only link-subspace positions (`subspace(v_ℓ) = s_L`), targeting dom(L); K.μ⁻ restricts dom(M(d)) without altering survivors' subspaces or values. The exhaustiveness conjunct (S3★-aux) holds because every position-creating step commits a position in `{s_C, s_L}`; the referential-integrity conjunct (S3★) holds because each such position targets the store matching its subspace.

*S4 (Origin-based identity, content distinctness — per ASN-0036).* S4 (per ASN-0036) is stated strictly over `dom(C)`: `a₁, a₂ ∈ dom(C)` produced by distinct allocation events satisfy `a₁ ≠ a₂`. Under the elementary transitions of this ASN, only K.α touches `dom(C)`; every other transition holds C in frame and so preserves S4 trivially (`dom(C') = dom(C)` ⟹ no new allocation events on the content store). For K.α, freshness of `a` against `dom(C)` (same-document distinctness) is the same-subspace part of SubAllocFresh at `x = C`; cross-document distinctness for two K.α events under d₁ ≠ d₂ is CrossDocDisjoint at the content-anchor pair `(b_C(d₁), b_C(d₂))`.

*S7d (Document allocation discipline).* Every `d ∈ E_doc` is T4-valid with `zeros(d) = 2`, placed in E_doc by a K.δ event on the parent allocator's realized domain. Document distinctness — the "distinct documents arise from distinct allocation events" half of S7d's postcondition — is CrossDocEntityDisjoint (established as a named lemma above, covering both same- and cross-parent pairs). Documents enter `E_doc` by three K.δ routes, each discharging `zeros(d) = 2` via the relevant K.δ-ID.zeros identity and freshness via the corresponding discharge:
- *k = 2 (descent off an account, `zeros = 1 → 2`).* The operand `t` is an account (`zeros(t) = 1`); K.δ-ID.zeros-2 gives `zeros(d) = zeros(t) + 1 = 2`. Freshness and distinctness are the case-(ii) preconditions discharged at the K.δ definition.
- *k = 1 (version off a document, `zeros = 2` preserved).* The operand `t` is a document (`zeros(t) = 2`); K.δ-ID.zeros-0/1 preserves `zeros(d) = zeros(t) = 2`. Freshness and distinctness are the case-(ii) preconditions discharged at the K.δ definition.
- *k = 0 (sibling off a document, `zeros = 2` preserved).* The operand `t` is a document (`zeros(t) = 2`); K.δ-ID.zeros-0/1 preserves `zeros(d) = zeros(t) = 2`. Freshness is the caller-checked guard `inc(t, 0) ∉ E`, the k = 0 frontier read of FrontierEquivalence.

Preserved by P1.

*K.μ~ discharge for the arrangement-shape invariants.* The shape stipulations (S8a, S8-depth, D-CTG★, D-MIN★) are stipulated on `M'(d)` by K.μ~ admissibility (i), and S8★(Σ') is established by the inherited K.μ⁺ and K.μ⁻ S8★ columns — K.μ~ being their composite — so its only S8★ delta is the union of those two columns' deltas, recorded in the *S8★ (Per-subspace span decomposition)* entry below. Two members carry a rider beyond that decomposition. **S8-fin(Σ')** — bundled in the matrix with S8a/S8-depth but *not* part of the shape package — is discharged independently of admissibility (i) and K.μ~-FIX: K.μ⁻ restricts dom(M(d)) (a subset of a finite set is finite) and K.μ⁺ adds finitely many positions (finite + finite = finite). **D-SEQ★** is then derived at Σ' from its constituents D-CTG★ + D-MIN★ + S8-depth + S8a together with that S8-fin(Σ'), via the standard D-SEQ★ derivation.

*S8a, S8-depth, S8-fin.* Established at arrangement-extending transitions: K.μ⁺ amendment's preconditions on new V-positions (positivity, depth, finiteness preserved by finite extension); K.μ⁺_L preconditions (positivity, fixed per-document depth `m_L(d)`, finite extension); K.μ⁻ restriction preserves all three; all others hold M in frame.

*S8★ (Per-subspace span decomposition).* The two-route construction is given at S8★'s definition box (the sole normative site); the constituent invariants S2/S7b/C1b/S8a/S8-depth/S8-fin are elementary-preserved as the matrix shows. Only the per-transition preservation deltas remain. *K.μ⁺* extends the content-subspace projection `M'(d)|_{V_{s_C}(d')}`, re-establishing S8★(s_C) by reapplying that definition's content route to it. *K.μ⁺_L* extends the link-subspace projection, re-establishing S8★(s_L) by that definition's link route on the new survivor. *K.μ⁻* restricts both projections, re-applying each route's construction to the contracted projection (the S8★ preconditions are preserved by restriction). All other transitions hold M in frame.

*D-CTG★ / D-MIN★.* K.μ⁺'s original precondition list (stated at the K.μ⁺ definition in *Elementary transitions*) requires the resulting `M'(d)` to satisfy D-CTG and D-MIN, strengthened to D-CTG★ / D-MIN★ in the extended state — contiguity and minimum-position preservation on the extended content subspace is therefore discharged by precondition, not by the K.μ⁺ amendment (which adds only the `subspace(v) = s_C` restriction). K.μ⁺_L preconditions cover the D-CTG case (non-empty link subspace) and the D-MIN case (empty); K.μ⁻ satisfies D-CTG★ and D-MIN★ at Σ' by construction (contraction shape at the K.μ⁻ definition); all others hold M in frame.

*P6 (Existential coherence), P7 (Provenance grounding), P8 (Entity hierarchy).* Discharge: the inductive *Derivation* in each definition box — P6 and P7 in the *Cross-layer invariants* section above, P8 in its definition paragraph in the *Permanence* section.

*NodeLineage* `(A e ∈ E : Node(e) : n₀ ≼ e)`. Base: `E₀ = {n₀}` with `n₀ ≼ n₀` by reflexivity of the tumbler-prefix order. K.δ case (i) — `Node(e)` — has `n₀ ≼ e` as an explicit precondition, discharged by NodeBaptism (b) (the boundary's bootstrap-lineage commitment supplies `n₀ ≼ e` directly at every node-allocation event); the inductive hypothesis carries `n₀ ≼ e'` for every prior node. K.δ case (ii) — `¬Node(e)` — adds a non-node, outside the Node quantifier; existing nodes unchanged. All other transitions hold E in frame.

*L0 (SubspacePartition).* L-clause from K.λ's precondition `subspace_I(ℓ) = s_L`; preserved by L12. C-clause from K.α's `E(a)₁ = s_C` precondition (per ASN-0093) — equivalently `subspace_I(a) = s_C`; preserved by P0.

*L1b (Link element-field depth).* `#E(ℓ) ≥ 2`. *First-link case:* the link sub-allocator emits `ℓ = [d.0.s_L.1]`. The address `d` is a document tumbler with `zeros(d) = 2`; the emission appends one zero separator and the two-component suffix `[s_L, 1]`, giving `zeros(ℓ) = 3` and T4-validity (FirstEmission / ChainElementT4Validity, ASN-0093). Applying T4b (UniqueParse, ASN-0034) to `ℓ` at `zeros = 3` makes all four projections N, U, D, E well-defined, with `E(ℓ) = [s_L, 1]` (the suffix following the third zero separator), so `#E(ℓ) = 2` directly. *Subsequent-link case:* K.λ emits `ℓ = inc(prev, 0)` (TA5(c)), with `prev` T4-valid by T10a.4. The single structural fact: TA5(c) gives `#ℓ = #prev` and modifies only the *value* at position `sig(prev)`, and TA5-SigValid gives `sig(prev) = #prev` — the terminal, non-separator element-field component, which is nonzero and stays nonzero. Hence no zero is added, moved, or removed: every separator position is fixed and the zero positions of `ℓ` coincide with those of `prev`. Both claims read off this fact directly — `zeros(ℓ) = zeros(prev) = 3`, and (T4b's field parse being identical) `E(ℓ)` occupies the same positions as `E(prev)`, giving `#E(ℓ) = #E(prev) ≥ 2` inductively. Preserved by L12 thereafter.

*C1c (Content allocator conformance).* Every `a ∈ dom(C)` must be reachable from a T4-valid document-level seed `s` (`zeros(s) = 2`) by a *structural inc-chain* with `k₁ = 2`: each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfying TA5's structural preconditions (operand T4-validity, zeros bound at k = 2), plus length monotonicity `#tᵢ > #s`. Base: `dom(C₀) = ∅`, vacuous. K.α *first-emission case*: emits `a = [d.0.s_C.1]` (SubAllocatorBundle) under `d ∈ E_doc`. Under SubspaceConventionAxiom (`s_C = 1`), the structural inc-chain `t₀ = d, t₁ = inc(d, 2) = b_C(d) = [d.0.1], t₂ = inc(t₁, 1) = a = [d.0.1.1]` satisfies per-step inc-rule conformance: `s = d` is T4-valid with `zeros(s) = 2`; `k₁ = 2, k₂ = 1`, each in `{0, 1, 2}`; the only `k = 2` step is `k₁` whose operand `t₀ = d` has `zeros(d) = 2 ≤ 2`; the only `k = 1` step is `k₂` whose operand `t₁ = b_C(d)` has `zeros(b_C(d)) = zeros(d) + 1 = 3 ≤ 3`, licensing the step under TA5a's `k = 1 ∧ zeros(t) ≤ 3` clause — the boundary case, admissible but exactly tight; and `#tᵢ > #d` at every step by TA5(d)'s length-extension and TA5(c)'s length-preservation. K.α *subsequent-emission case*: emits `a = inc(prev, 0)` (TA5(c)) under the same `d`, extending prev's chain by one additional step `kₙ₊₁ = 0`. All other transitions hold C in frame, with persistence by P0.

*L1c (Link allocator conformance).* Identical in form to C1c, with one structural difference: the link chain is C1c's content chain extended by a single intermediate `inc(·, 0)` step that advances the content anchor `b_C(d) = [d.0.s_C]` to the link anchor `b_L(d) = [d.0.s_L]` (resting on SubspaceConventionAxiom's `s_L = s_C + 1`) before the final `k = 1` descent. Base: `dom(L₀) = ∅`, vacuous. K.λ *first-link case*: emits `ℓ = [d.0.s_L.1]` (SubAllocatorBundle) under `d ∈ E_doc` via the chain `t₀ = d, t₁ = inc(d, 2) = b_C(d), t₂ = inc(t₁, 0) = b_L(d), t₃ = inc(t₂, 1) = ℓ`. Per-step `kᵢ`-conformance (`k₁ = 2, k₂ = 0, k₃ = 1`) and length monotonicity `#tᵢ > #d` follow exactly as in C1c; the sole `k = 1` step's operand `b_L(d)` has `zeros(b_L(d)) = 3` just as `b_C(d)` does in C1c's `k = 1` step, so the same TA5a boundary clause licenses it. K.λ *subsequent-link case*: emits `ℓ = inc(prev, 0)` (TA5(c)) under the same `d`, extending prev's chain by one additional step `kₙ₊₁ = 0`. All other transitions hold L in frame.

*L14 (StoreDisjointness).* ASN-0093's SD restated; cited from ASN-0093 (see L14 in the *Inherited from foundation* table).

*C-fin (Content store finiteness).* Matrix-cell discharge: finite extension by K.α, frame elsewhere. Its load-bearing role in the `max`-well-definedness of K.α's subsequent emission belongs to ASN-0093's K.α (foundation) and is inherited here unchanged.

*CL-OWN (LinkSubspaceOwnership).* K.μ⁺_L precondition `origin(ℓ) = d` at every new link-subspace mapping; K.μ⁻ frame on surviving mappings; all others hold M in frame (K.δ's Document case adds the new empty arrangement `M'(e) = ∅`, vacuous over its empty domain).

*CL-UNIQ (LinkSubspacePositionUniqueness).* K.μ⁺_L's first-arrangement precondition `ℓ ∉ ran(M(d))` ensures each newly arranged link occupies a unique V-position. K.μ⁻ restriction of an injective function remains injective. K.μ⁺ confines its additions to subspace `s_C` (the amendment's `subspace(v) = s_C` restriction), adding no link-subspace V-positions and leaving `M(d)|_{dom_L}` unchanged, so link-subspace injectivity is preserved — matching the CL-OWN/K.μ⁺ treatment. All other transitions hold M in frame (K.δ's Document case adds the new empty arrangement `M'(e) = ∅`, vacuous over its empty domain).

**Class (b): Composite-boundary properties** — the composite-boundary triple of the ExtendedReachableStateInvariants definition above, discharged at composite boundaries by the J0/J1★/J1'★ couplings of ValidComposite★.

*Composite-boundary verification matrix.* For each Class (b) property, the matrix records the discharge mechanism at the composite boundary and the elementary step at which it transiently fails.

| Property | Discharge at composite boundary | Transient failure within composite |
|----------|--------------------------------|------------------------------------|
| P4★ (`Contains_C(Σ) ⊆ R`) | J1★ at boundary supplies `(a, d) ∈ R'` for each new content-subspace containment | After K.μ⁺ before K.ρ: `(a, d) ∈ Contains_C(M_post)` but not yet in R |
| P4a (every R-entry witnessed by content-subspace containment in some trace state) | By induction along the witnessing trace (per-property argument below). | Not evaluated at intermediate states (see P4a definition box); discharged at the boundary Σ', which carries the witness placed by K.μ⁺. |
| P7a (every `a ∈ dom(C)` has a provenance entry) | J0 supplies `v ∈ dom(M'(d))` with `M'(d)(v) = a`; S3★ + L14 + S3★-aux force `subspace(v) = s_C`; J1★ then supplies `(a, d) ∈ R'` | After K.α before K.μ⁺/K.ρ: `a ∈ dom(C')` but no `(·, d)` entry in R' |

The matrix corresponds to the per-property arguments below.

P4★ (`Contains_C(Σ) ⊆ R`): For each `(a, d) ∈ Contains_C(Σ') \ Contains_C(Σ)`, J1★ at the composite boundary requires `(a, d) ∈ R'`. K.α, K.δ, K.ρ hold M in frame; K.μ⁺_L adds only link-subspace V-positions (excluded from Contains_C); K.μ⁻ can only shrink Contains_C; K.μ~ preserves Contains_C exactly by **K.μ~-RANGE** (`Contains_C(Σ') = Contains_C(Σ)`). Only K.μ⁺ may transiently violate P4★ (it adds a new content-subspace containment to Contains_C while framing R); J1★ supplies the co-occurring K.ρ that places `(a, d) ∈ R'`, restoring the bound at the composite boundary.

P4a (`(A (a, d) ∈ R :: (E Σ_k ∈ {Σ₀, ..., Σ_n} : (E v ∈ dom(M_k(d)) : subspace(v) = s_C ∧ M_k(d)(v) = a)))`, where `{Σ₀, ..., Σ_n}` is the transition history of the trace reaching Σ', as defined in the P4a definition box): discharged by induction along the witnessing trace (all other transitions hold R in frame). For a freshly recorded entry `(a, d) ∈ R' \ R`, the witnessing trace state is the composite endpoint Σ' itself, and that Σ' carries a *live* content-subspace witness is exactly the premise J1'★ supplies: ValidComposite★ clause (2) requires J1'★ of every valid composite, and J1'★'s first conjunct `(E v ∈ dom(M'(d)) : subspace(v) = s_C ∧ M'(d)(v) = a)` is evaluated at the post-state M' — so every new provenance entry has a content-subspace V-position mapping to `a` at Σ'. It forbids a composite that both places `a` (K.μ⁺) and removes it (K.μ⁻) before its endpoint, since such a composite would record `(a, d)` yet fail J1'★'s post-state conjunct at Σ' and so fail to be a valid composite at all. A persisted entry `(a, d) ∈ R` inherits a witnessing trace state from the inductive hypothesis, carried forward by P2.

P7a (`(A a ∈ dom(C) :: (E d :: (a, d) ∈ R))`): For `a ∈ dom(C') \ dom(C)`, J0 supplies `d ∈ E'_doc` and `v ∈ dom(M'(d))` with `M'(d)(v) = a`. We show the V-position `v` must be content-subspace, evaluated at the composite endpoint Σ' where both `v` and `a` co-exist. Suppose for contradiction `subspace(v) = s_L`. Then by S3★ at Σ' (link clause), `M'(d)(v) ∈ dom(L')`, i.e., `a ∈ dom(L')`. But `a ∈ dom(C')` (J0's defining membership) and L14 at Σ' gives `dom(C') ∩ dom(L') = ∅`, contradiction. By S3★-aux, `subspace(v) ∈ {s_C, s_L}`, so `subspace(v) = s_C`. *Range-new discharge for J1★.* J1★'s trigger has two conjuncts. The first, `a ∈ ran(M'(d)|_{s_C})`, follows directly: `subspace(v) = s_C` and `M'(d)(v) = a` place `v ∈ V_{s_C}(d')` with `M'(d)(v) = a`. The second, `a ∉ ran(M(d)|_{s_C})`, is the range-new discharge established at the J1★ derivation (content-boundedness of the content-subspace range): `a ∈ dom(C') \ dom(C)` gives `a ∉ dom(C)`, hence `a ∉ ran(M(d)|_{s_C})`. With both conjuncts discharged, J1★ supplies `(a, d) ∈ R'`. No transition removes from dom(C) (P0) or from R (P2), so P7a, once established, persists.

Coupling constraints J0, J1★, J1'★ hold for all valid composites by the analysis in the Scoped coupling constraints section.

**Per-transition invariant** (ExtendedTransitionInvariants: P3). P3 is proved by the seven-transition case analysis in *Destruction confinement*; transitivity of ⊆ and = over a finite composite lifts it to the composite boundary. ∎


## Temporal decomposition

The state Σ = (C, L, E, M, R) decomposes into three temporal layers: an *existential* layer (C, L, E) that admits only growth and per-entry immutability; a *historical* layer (R) that admits only growth and may become stale relative to current arrangements; and a *presentational* layer (M) that is freely mutable. Cross-layer bridges (defined in *Cross-layer invariants* above): P6 and L1a tie C and L to E; S3★ bridges M to {C, L}; P7/P7a bridge C and R; CL-OWN constrains the M→L bridge to a document's own links; P4★ bridges presentational and historical layers via Contains_C(Σ) ⊆ R.

| Layer | Components | Mutability | Transitions modifying this component |
|-------|-----------|------------|----------------------|
| Existential (functional) | C, L | Append-only domain; values immutable | K.α, K.λ |
| Existential (set) | E | Append-only membership; no value structure | K.δ |
| Historical | R | Append-only, entries may stale | K.ρ |
| Presentational | M | Fully mutable | K.δ (Document case: domain growth only, non-destructive — `dom(M') = dom(M) ∪ {e}`, `M'(e) = ∅`, since `dom(M) = E_doc` by the Bridging lemma); K.μ⁺, K.μ⁺_L, K.μ⁻ (elementary); K.μ~ (named composite, K.μ⁻ + K.μ⁺) |


## Properties Introduced

### New properties introduced by this ASN

| Label | Statement |
|-------|-----------|
| Σ.E | E ⊆ {t : T4-valid(t) ∧ zeros(t) ≤ 2} — entity addresses, partitioned by Node / Account / Document |
| Σ.R | R ⊆ T_elem × E_doc — provenance relation recording historical content associations |
| Σ₀ | Initial state: C₀ = ∅, E₀ = {n₀} (bootstrap node), M₀(d) = ∅ for all d, R₀ = ∅ |
| parent(e) | For ¬Node(e): tumbler obtained by truncating last field and preceding separator |
| Contains(Σ) | {(a, d) : d ∈ E_doc ∧ a ∈ ran(M(d))} — current containment, derived quantity of state |
| Contains_C(Σ) | `{(a, d) : d ∈ E_doc ∧ (E v : v ∈ dom(M(d)) ∧ subspace(v) = s_C : M(d)(v) = a)}` — content-scoped containment |
| Valid composite | Σ →* Σ' valid iff: (1) elementary preconditions at each intermediate state, (2) J0/J1★/J1'★ for the composite |
| K.α | Content allocation — extends dom(C) with a fresh content address. Normative contract: *Elementary transitions*. |
| K.δ | Entity creation — extends E with a fresh entity (node/account/document). Normative contract: *Elementary transitions*. |
| K.μ⁺ | Arrangement extension — extends a document's content-subspace arrangement dom(M(d)). Normative contract: *Elementary transitions*. |
| K.μ⁻ | Arrangement contraction — removes V→I mappings from a document's arrangement. Normative contract: *Elementary transitions*. |
| K.μ~ | Arrangement reordering — named composite (K.μ⁻ + K.μ⁺) permuting a document's arrangement. Normative contract: *Elementary transitions*. |
| K.λ | Link allocation — extends dom(L) with a fresh link address. Normative contract: *Link allocation*. |
| K.ρ | Provenance recording — extends R with a content↔document association. Normative contract: *Elementary transitions*. |
| K.μ⁺_L | Link-subspace arrangement extension — maps a fresh link-subspace V-position to a link. Normative contract: *Link-subspace extension*. |
| K.μ~-FIX | Domain fixity under K.μ~: dom(M'(d)) = dom(M(d)), making π a permutation of a fixed domain — from D-SEQ★ + bijection cardinality (n'_S = n_S) + subspace preservation + length preservation (admissibility (iii)) fixing per-subspace depth |
| J0 | Content allocation (K.α) always co-occurs with arrangement extension (K.μ⁺). A clause-(2) validity constraint of ValidComposite★ (see ValidComposite★ clause (2) for its imposed/derived status) |
| J2 | K.μ⁻ as elementary transition requires no coupling: C' = C ∧ L' = L ∧ E' = E ∧ R' = R |
| J3 | K.μ~ as named composite requires no coupling: C' = C ∧ L' = L ∧ E' = E ∧ R' = R |
| J4 | Fork composite: K.δ + K.μ⁺ + K.ρ (no other steps); content source is the K.δ operand d_op (d_src for k=1, prev_version for k=0); precondition V_{s_C}(d_op) ≠ ∅; content-subspace copy characterized by the unique order-preserving bijection φ: V_{s_C}(d_op) → V_{s_C}(d_new) with M'(d_new)(φ(v)) = M(d_op)(v) (order- and multiplicity-preserving, per CREATENEWVERSION full-copy semantics; range equality ran(M'(d_new)) = ran(M(d_op)|_{V_{s_C}(d_op)}) is a derived consequence); dom(C') = dom(C) follows from frames; provenance from J1★/J1'★ (extended-state couplings) |
| P1 | Entity set is monotonically growing: E ⊆ E' for every transition, uniformly across levels |
| P2 | Provenance relation is monotonically growing: R ⊆ R' for every transition |
| P4a | Trace witnessing: every (a, d) ∈ R has a witnessing state (present included) where a ∈ ran(M(d)) at content-subspace |
| P6 | Existential coherence: origin(a) ∈ E_doc for all a ∈ dom(C) |
| P7 | Provenance grounding: a ∈ dom(C) for all (a, d) ∈ R |
| P7a | Provenance coverage: (E d :: (a, d) ∈ R) for all a ∈ dom(C) — every I-address has provenance |
| P8 | Entity hierarchy: (A e ∈ E : ¬Node(e) : parent(e) ∈ E) — no orphan accounts or documents |
| m_L(d) | Depth of d's *current* link-subspace arrangement; see *Link-subspace extension* |
| NodeBaptism | Axiom (boundary input — node provisioning): node addresses are baptised at the network-provisioning boundary, not by any docuverse transition; see *Elementary transitions* for the freshness and bootstrap-lineage conjuncts. |
| FrontierEquivalence | Derived lemma: `inc(t, 0) ∉ Σ.E ⟺ t is the frontier of A's (t, 0)-branch` (where A is the allocator whose realized domain contains t, unique by T10a.6), for every reachable Σ and every `t ∈ Σ.E` with `¬Node(t)`; "frontier" is well-defined by T10a.7 |
| ChildSpawnFreshness | Derived lemma: `inc(t, k') ∉ Σ.E ⟺ the (t, k') child-spawn has not yet been performed`, for every reachable Σ, every `t ∈ Σ.E`, and every admissible `k' ∈ {1, 2}`; reverse direction via GlobalUniqueness/T10a.6 over the spawned child allocator's base; admits node operands (no `¬Node(t)` precondition) |
| ActivatedEmission | Per-state invariant (EntityEmissionTracking): `(A e ∈ Σ.E : ¬Node(e) : (E A : Activated(A) ∧ EntityLevel(A) : e ∈ dom(A)))` — every non-node entity inhabits some activated sub-allocator's domain. Holds vacuously at Σ₀; preserved by K.δ (each non-node entity enters E only via a T10a inc-step on an activated sub-allocator, per K.δ case (ii)) and frame on all other transitions |
| NodeLineage | Derived per-state invariant: `(A e ∈ E : Node(e) : n₀ ≼ e)` — every node in E descends structurally from the bootstrap node n₀ by tumbler-prefix relation. Discharged inductively from the base case `E₀ = {n₀}` (reflexivity) and the K.δ case (i) precondition `n₀ ≼ e` |
| b_C(d), b_L(d) | Virtual sub-allocator anchors under d: `b_C(d) = [d.0.s_C]`, `b_L(d) = [d.0.s_L]` — single-component element-field bases, not in dom(C) ∪ dom(L), serving as formal starting points for the content and link allocator chains under d |
| Allocator hierarchy | Content and link sub-allocators are sibling element-field allocators under d, sharing prefix `[d.0]`; T10a-conformance applies to each frontier separately; cross-document collisions prevented by T10, cross-subspace by L14 (= L0 + SC-NEQ) |
| SubAllocatorBundle | Bundling lemma (introduced here): for each d ∈ E_doc, the entity-allocation event placing d into E_doc activates two disjoint sub-allocators under d — content anchor `b_C(d) = [d.0.s_C]`, link anchor `b_L(d) = [d.0.s_L]`. Inheritance accounting and the cross-subspace disjointness delta are in the *Sub-allocator activation (SubAllocatorBundle)* definition box. |
| S3★-aux | Subspace exhaustiveness: `(A d, v : v ∈ dom(M(d)) : subspace(v) = s_C ∨ subspace(v) = s_L)` in every reachable state |
| CL-OWN | LinkSubspaceOwnership: `(A d, v : v ∈ dom(M(d)) ∧ subspace(v) = s_L : origin(M(d)(v)) = d)` — every document's link subspace contains only its own links |
| CL-UNIQ | LinkSubspacePositionUniqueness: `(A d, v₁, v₂ ∈ dom(M(d)) : subspace(v₁) = subspace(v₂) = s_L ∧ M(d)(v₁) = M(d)(v₂) : v₁ = v₂)` — each link occupies exactly one V-position in its home document's link subspace; injectivity of M(d)\|_{dom_L} |

The four structural identities `K.δ-ID.zeros-0/1`, `K.δ-ID.zeros-2`, `K.δ-ID.parent-0/1`, and `K.δ-ID.parent-2` are stated and derived inline at *Elementary transitions*, K.δ case (ii).

### Inherited from foundation (restated for narrative continuity)

These properties are foundation invariants of ASN-0093 (or earlier foundation ASNs). Their preservation under this ASN's new transition (K.μ⁺_L) and amended transitions (K.λ, K.μ⁺, K.μ⁻, K.μ~) is verified locally in the Class (a) matrix (e.g. the L0/L1c link-preservation arguments there) — with the sole exception of M1, which is document-set monotonicity and is discharged with P1 (see the M1 row).

| Label | Statement | Foundation source |
|-------|-----------|--------------------|
| SequentialTransitionAxiom | Axiom: transitions `Σ → Σ'` are atomic, uninterruptible, and totally ordered. | ASN-0093 (SequentialAtomicTransitions) |
| SubspaceConventionAxiom | Axiom: fixed subspace identifiers `s_C = 1 ∧ s_L = 2`, consequence `SC-NEQ`. The same identifiers serve V-positions via `subspace(v) = v₁` and element-level addresses via `subspace_I(a) = a's element-field first component`. | ASN-0093 (FixedSubspaceIdentifiers) |
| Endset | `Endset = 𝒫_fin(Span)` — a finite set of well-formed spans `(s, ℓ)` satisfying T12 (ASN-0034); the empty set ∅ is a valid endset. | ASN-0043 (Endset) |
| Link | `Link = {(e₁, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`; `|L|` is the arity. StandardTriple convention (arity 3, `(F, G, Θ)`) is applied in worked examples only, not as a structural restriction. | ASN-0043 (Link, StandardTriple) |
| L-fin | LinkStoreFiniteness: `|dom(Σ.L)| < ∞`. Holds at Σ₀ (`|∅| = 0`); preserved by K.λ (single-element extension) and L-frame elsewhere. | ASN-0043 (LinkStoreFiniteness) |
| L0 | SubspacePartition: `(A a ∈ dom(Σ.L) :: subspace_I(a) = s_L)` and `(A a ∈ dom(Σ.C) :: subspace_I(a) = s_C)` — both clauses are foundation invariants of ASN-0093. (The L-clause appears in ASN-0043's original L0; the C-clause was added in ASN-0093's foundation L0 and is supplied at allocation time by ASN-0093's K.α precondition `E(a)₁ = s_C`.) | ASN-0093 (SubspacePartition) |
| L1 | LinkElementLevel: `(A a ∈ dom(Σ.L) :: zeros(a) = 3)` — every link address is an element-level tumbler. | ASN-0093 (LinkElementLevel) |
| L1a | LinkScopedAllocation: `(A a ∈ dom(Σ.L) :: origin(a) ∈ E_doc)` — every link address is allocated under the tumbler prefix of a document (`E_doc = dom(M)` by the Bridging lemma). | ASN-0093 (LinkScopedAllocation) |
| L3 | NEndsetStructure: `(A a ∈ dom(Σ.L) :: |Σ.L(a)| ≥ 3 ∧ (A i : 1 ≤ i ≤ |Σ.L(a)| : Σ.L(a).eᵢ ∈ Endset) ∧ Σ.L(a).e₃ ≠ ∅)` — every link is a sequence of at least three endsets with the type endset (slot 3) non-empty. Inherited verbatim from ASN-0093's L3 (which itself inherits from ASN-0043's `Link` definition admitting arity `N ≥ 3`). | ASN-0093 (NEndsetStructure) |
| L12 | LinkImmutability: `(A Σ → Σ' : (A a : a ∈ dom(Σ.L) : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)))` — once created, a link's address persists in `dom(L)` and its value is permanently fixed. | ASN-0093 (LinkImmutability) |
| C-fin | ContentStoreFiniteness: `|dom(Σ.C)| < ∞`. Inherited as a per-state invariant of the extended state; established at Σ₀ (`|dom(C₀)| = 0`) and preserved by K.α (extends by one) with frame on all other transitions. | ASN-0093 (ContentStoreFiniteness) |
| L1c | LinkAllocatorConformance: every `ℓ ∈ dom(L)` has a structural inc-chain from its home document to `ℓ` — a finite sequence `(t₀, …, tₙ)` with `t₀ = origin(ℓ)`, `tₙ = ℓ`, each step `tᵢ = inc(tᵢ₋₁, kᵢ)` with `kᵢ ∈ {0, 1, 2}` satisfying T10a's per-step admissibility (T4-validity preservation, zero-count side condition at `kᵢ = 2`), `k₁ = 2`, and `#tᵢ > #origin(ℓ)` at every step. | ASN-0093 (LinkAllocatorConformance) |
| L14 | StoreDisjointness: `dom(C) ∩ dom(L) = ∅` — unscoped store disjointness. | ASN-0093 (SD, StoreDisjointness) |
| M1 | ArrangementMonotonicity: `(A Σ → Σ' :: dom(M) ⊆ dom(M'))`. Constrains the *document set* `dom(M) = E_doc` (the allocated documents, which only grow, via K.δ), **not** the per-document arrangement `dom(M(d))` — the latter contracts under K.μ⁻. Discharged explicitly (not in the Class (a) matrix): by the Bridging lemma `dom(M) = E_doc`, so M1 is P1 (`E ⊆ E'`) restricted to documents; K.δ grows `dom(M)` and `E_doc` together by `{e}` (†, above) and every other transition frames the document set, so `dom(M) ⊆ dom(M')` holds at each elementary step. | ASN-0093 (ArrangementMonotonicity) |

### Local extensions and strengthenings of foundation properties

| Label | Statement | Foundation source |
|-------|-----------|--------------------|
| P0 | Content store is append-only with immutable values: dom(C) ⊆ dom(C') ∧ C'(a) = C(a) for a ∈ dom(C) | Subsumes ASN-0036's S0 (ContentImmutability) and S1 (StoreMonotonicity) into a single unified statement |
| L14a | Inapplicable: ASN-0043's L14a is scoped to `s_C`-resident systems, but the extended two-subspace state is by construction not `s_C`-resident (K.μ⁺_L maps link-subspace V-positions into dom(L)), so L14a's hypothesis is unmet; where link-subspace mappings occur, S3★ routes them into dom(L) and CL-OWN constrains them to the home document | ASN-0043's L14a (NonTranscludability) |
| S3★ | Subspace-conditional referential integrity: text → dom(C), link → dom(L); supersedes S3 | ASN-0036's S3 (ReferentialIntegrity) is single-store; this ASN partitions the target by subspace |
| D-CTG★ | Per-subspace contiguity: `(A d, S : V_S(d) ≠ ∅ : V_S(d) is contiguous under the V-ordering on subspace S)` — local strengthening of ASN-0036's D-CTG dropping the link-subspace exemption; supersedes D-CTG within the extended state | ASN-0036's D-CTG (Contiguity) had a link-subspace exemption |
| D-MIN★ | Per-subspace minimum position: `(A d, S : V_S(d) ≠ ∅ : min(V_S(d)) = [S, 1, ..., 1] of depth m_S)` — local strengthening of ASN-0036's D-MIN dropping the link-subspace exemption; supersedes D-MIN within the extended state | ASN-0036's D-MIN (MinimumPosition) had a link-subspace exemption |
| D-SEQ★ | Per-subspace lex-sequential range: for each non-empty subspace S in M(d), `V_S(d) = {[S, 1, ..., 1, k] : 1 ≤ k ≤ n_S}` of uniform depth m_S — derived from D-CTG★ + D-MIN★ + S8-depth + S8-fin + S8a, per-subspace promotion of ASN-0036's D-SEQ to a system-wide invariant of the extended state | ASN-0036's D-SEQ (LexSequential) was per-document; this ASN promotes per-subspace and elevates to system-wide invariant |
| P3 | No component other than M — specifically C, L, E, R — admits contraction or reordering; quantitative monotonicity formalised as `dom(C) ⊆ dom(C') ∧ dom(L) ⊆ dom(L') ∧ E ⊆ E' ∧ R ⊆ R' ∧ (A a ∈ dom(C) :: C'(a) = C(a)) ∧ (A ℓ ∈ dom(L) :: L'(ℓ) = L(ℓ))` | Per-transition synthesis defined in *Destruction confinement* |
| P4★ | `Contains_C(Σ) ⊆ R` — provenance bounds scoped to the content subspace | This ASN's own provenance bound; content-subspace scoping (P7-coexistence rationale at the P4★ definition) |
| J1★ | Range-based content-subspace provenance coupling: provenance recording for I-addresses new to content-subspace range | This ASN's content-subspace scoping of the extension-records-provenance coupling |
| J1'★ | Range-based content-subspace converse coupling: provenance entries only from content-subspace range changes | This ASN's content-subspace scoping of the provenance-requires-extension coupling |
| ValidComposite★ | Valid composite in extended state: transition preconditions at each step (K.μ~ as shorthand for K.μ⁻ + K.μ⁺) + J0, J1★, J1'★ at composite boundary; supersedes ValidComposite | This ASN's own Valid composite definition extended for the two-subspace state |
| S8★ | Per-subspace span decomposition: for each d ∈ E_doc and each subspace S ∈ {s_C, s_L}, M(d)\|_{V_S(d)} decomposes into a finite set of correspondence runs satisfying ASN-0036's S8 conditions (a) and (b) on the per-subspace projection — content-subspace projection by ASN-0036's S8 directly, link-subspace projection by trivial length-1 decomposition | ASN-0036's S8 is stated under S3 (single store), failing on the unprojected M(d) once link-subspace V-positions target dom(L); S8★ restores S8 per-subspace using S3★'s per-subspace clauses |
| ExtendedReachableStateInvariants | Every reachable state satisfies the *per-state invariants* S2 ∧ S3★ ∧ S3★-aux ∧ S4 ∧ S7a ∧ S7b ∧ C1b ∧ C1c ∧ S7d ∧ S8a ∧ S8-fin ∧ S8-depth ∧ S8★ ∧ C-fin ∧ D-CTG★ ∧ D-MIN★ ∧ D-SEQ★ ∧ P6–P8 ∧ NodeLineage ∧ ActivatedEmission ∧ L0 ∧ L1 ∧ L1a ∧ L1b ∧ L1c ∧ L3 ∧ L14 ∧ L-fin ∧ CL-OWN ∧ CL-UNIQ (preserved by each elementary transition); every state at a composite boundary additionally satisfies the *composite-boundary properties* P4★ ∧ P4a ∧ P7a (discharged at boundaries by J0/J1★/J1'★, may transiently fail at intermediate states). P3 is *per-transition*: see ExtendedTransitionInvariants. Together supersedes ReachableStateInvariants | This ASN's own Reachable-state invariants synthesis extended to the two-subspace state |
| ExtendedTransitionInvariants | Every valid composite transition Σ →* Σ' satisfies P3 (defined in *Destruction confinement*) | This ASN's own per-transition synthesis |
| K.α's `E(a)₁ = s_C` precondition (inherited) | Pins `subspace_I(a) = s_C` for every allocated content address; see the K.α elementary definition in *Elementary transitions* | Inherited from ASN-0093's K.α (ContentAllocation) precondition |
| K.μ⁺ amendment | Content-subspace restriction (`subspace(v) = s_C`); existing D-CTG/D-MIN postconditions carry forward; partitions arrangement extension by subspace with K.μ⁺_L | Strengthening of this ASN's K.μ⁺ (defined in *Elementary transitions* above) at the extended-state introduction |
| K.μ⁻ (per-subspace scope) | The per-subspace D-CTG★/D-MIN★ postconditions stated at K.μ⁻'s definition apply to each subspace independently; valid contractions per-subspace are per-subspace suffix removals or full clearances (forced by D-CTG★ + D-MIN★ + D-SEQ★ at the post-state) | Strengthening of this ASN's K.μ⁻ (defined in *Elementary transitions* above): its postconditions are stated against the per-subspace D-CTG★/D-MIN★ invariants (which drop ASN-0036's D-CTG/D-MIN link-subspace exemption), carrying through to two subspaces |

## Open Questions

- What guarantees must the system provide about provenance when content is transitively shared through chains of transclusion?
- Can arrangement contraction on one document affect the discoverability of links attached to the same I-addresses from another document?
- What relationship must hold between a document's version lineage and its sequence of arrangement transitions?
- What additional permanence properties must the provenance relation satisfy for content that participates in link endsets?
- This ASN brings the link subspace under the system-wide D-CTG★ / D-MIN★ / D-SEQ★ regime (dropping the base forms' link-subspace exemption), so the link subspace is now a dense, contiguous, sequentially-positioned stream just as the content subspace is — what further invariants must the link subspace satisfy *beyond* this shared sequential structure (link-specific capacity bounds, ordering constraints tied to link semantics, or structural properties of endset references) that D-SEQ★ does not capture?
- Must the system guarantee that a fresh link address is always available within a document's link subspace, or can link allocation fail due to address space exhaustion?
- What must the system guarantee when concurrent operations target the same home document — must link address allocation be serialized, or can concurrent allocations produce distinct addresses without coordination?
- What protocol-level mechanism realizes node baptism at the network-provisioning boundary, and what coordination (if any) must it perform across independently-administered nodes to deliver the freshness and bootstrap-lineage that NodeBaptism takes as given?
- What invariants must a renumbering-aware link-arrangement contraction maintain so that an interior link can be withdrawn from `M(d)` while preserving D-CTG★ / D-MIN★? This ASN's K.μ⁻ contracts the link subspace by suffix removal alone, faithful to the implementation's gap-free POOM only for suffix deletions; the implementation's interior `DELETEVSPAN` instead compacts-and-renumbers surviving V-positions, an operation the present K.μ⁻ does not model. (Link permanence itself is already discharged on `dom(L)` by L12, independently of the arrangement, so this is a question about the contraction *operation*, not about tombstoning.)
- Should K.λ require `e₁ ∪ e₂ ≠ ∅` to exclude type-only links, or admit them as valid markers per Nelson's one-sided link case (LM 4/48)? If admitted, do one-sided links (exactly one of e₁, e₂ empty) and type-only markers (both empty) carry distinguishable semantics in endset-iterating consumers like L8's `same_type` and the discovery-set unions?
- Should the entity-allocation discipline admit account-level depth-1 extension (K.δ with `k = 1` and `Account(t)`) for future use cases such as account renaming or multi-account user identity, rather than reserving versioning to documents?
