> **ASN-0036 · Strand Model** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md)  
> [Condensed statements →](ASN-0036-strand-model.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0036: Strand Model

*2026-03-14; revised 2026-03-21, 2026-03-22, 2026-03-22, 2026-03-28, 2026-04-09, 2026-04-11, 2026-05-29*

We wish to understand what formal invariants govern the relationship between permanent content storage and mutable document arrangement in Xanadu. Nelson separated these concerns into two address spaces — Istream for content identity and Vstream for document positions — and asserted this separation as the architectural foundation on which permanence, transclusion, and attribution all rest. We seek the abstract properties that define this separation: what must hold in any correct implementation, regardless of the underlying data structures.

The approach is: model the system as two state components, derive what each must guarantee independently, then identify the invariants connecting them. Nelson provides architectural intent; Gregory's implementation reveals which properties are load-bearing.

Nelson conceived the two streams as inseparable aspects of a single architecture. Gregory implemented them as distinct enfilade types with different stability characteristics. Between these two accounts we find the abstract structure: a content store that grows but never changes, and a family of arrangement functions that change freely but may reference only what the store contains.


## Two components of state

The observation that motivates the entire design is that content EXISTS independently of how it is ARRANGED. A paragraph does not cease to exist when removed from a document — it merely ceases to appear there. Nelson states this plainly:

> "Instead, suppose we create an append-only storage system. User makes changes, the changes difflessly into the storage system, filed, as it were, chronologically." [LM 2/14]

This observation forces the state into two components:

**Σ.C (ContentStore).** The *content store*: a partial function mapping Istream addresses to content values. `T` is the set of tumblers (ASN-0034); `Val` is an unspecified set of content values, opaque at this level of abstraction. The domain `dom(Σ.C)` is the set of I-addresses at which content has been stored.

*Formal Contract:*
- *Axiom:* `Σ.C : T ⇀ Val` — the content store is a partial function from tumblers to content values.
- *Definition:* `dom(Σ.C) = {a ∈ T : Σ.C(a) is defined}` — the set of I-addresses at which content has been stored.

**Σ.M(d) (Arrangement).** The *arrangement* of document `d`: a partial function mapping Vstream positions to Istream addresses. The domain `dom(Σ.M(d))` is the set of V-positions currently active in `d`; the range `ran(Σ.M(d))` is the set of I-addresses that `d` currently references.

A conventional system merges these — "the file" IS the content IS the arrangement. Editing overwrites. Saving destroys the prior state. Nelson rejected this explicitly: "Virtually all of computerdom is built around the destructive replacement of successive whole copies of each current version." The two-component model is his alternative: editing modifies `M(d)` while `C` remains invariant. The separation is the premise; what follows are the invariants it must satisfy.

*Formal Contract:*
- *Axiom:* `Σ.M(d) : T ⇀ T` — the arrangement of document `d` is a partial function from V-position tumblers to I-address tumblers.
- *Axiom (domain restriction):* `dom(Σ.M(d)) ⊆ {t ∈ T : zeros(t) = 0 ∧ #t ≥ 2}` — arrangements map only V-positions; every active key is a zero-free tumbler of depth at least 2 (a subspace identifier followed by a within-subspace ordinal).
- *Definition (S8a — V-position well-formedness):* We name the domain restriction above `S8a`. By T0 (ASN-0034), `zeros(t) = 0` holds exactly when every component is positive, so `S8a` reads equivalently in per-component form: every active V-position is a zero-free tumbler of depth at least 2 with all components positive.
- *Definition:* `dom(Σ.M(d)) = {v ∈ T : Σ.M(d)(v) is defined}` — the set of V-positions currently active in `d`.
- *Definition:* `ran(Σ.M(d)) = {Σ.M(d)(v) : v ∈ dom(Σ.M(d))}` — the set of I-addresses that `d` currently references.

We call this paired state the *strand*: the two-component object `(Σ.C, Σ.M)` — an immutable content store woven together with the family of mutable arrangements that reference it. The remainder of this ASN derives the invariants that govern a strand.

## The content store

We ask: what must `C` guarantee? Nelson requires that any historical version be reconstructable, that content transcluded across documents maintain its meaning, and that attribution be permanent. Working backward from these guarantees — what must `C` satisfy for them to hold?

Suppose `C(a)` could change from value `w` to `w'` in some state transition. Then every document whose arrangement maps a V-position to `a` would silently show different content — with no editing operation having touched any arrangement. Historical versions, which reconstruct their state by reassembling Istream fragments, would silently present altered text. Content transcluded from one document into another would mutate without the including document's knowledge or consent. Nelson: "Users may create new published documents out of old ones indefinitely, making whatever changes seem appropriate — without damaging the originals." Mutation of `C(a)` damages every original that contains `a`.

We therefore require:

**S0 (Content immutability).** For every state transition `Σ → Σ'`:

`[a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a)]`

Once content is stored at address `a`, both the address and its value are fixed for all future states. This is the central invariant of the two-stream architecture.

*Formal Contract:*
- *Axiom (design requirement):* For every state transition `Σ → Σ'`, `(A a : a ∈ dom(Σ.C) : a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a))`.
- *Postconditions:* (a) Domain persistence — `a ∈ dom(Σ.C) ⟹ a ∈ dom(Σ'.C)`. (b) Value preservation — `a ∈ dom(Σ.C) ⟹ Σ'.C(a) = Σ.C(a)`.
- *Frame:* No condition on arrangements — the postcondition holds for arbitrary `Σ'.M(d)` and arbitrary changes to any document's arrangement.

**S1 (Store monotonicity).** `[dom(Σ.C) ⊆ dom(Σ'.C)]`

S0 and S1 together establish `C` as an *append-only log*. New entries may be added — each at a fresh address guaranteed unique by T9 and T10 (ASN-0034) — but no existing entry may be modified or removed.

Nelson states this as an explicit design commitment: "The true storage of text should be in a system that stores each change and fragment individually, assimilating each change as it arrives, but keeping the former changes." Gregory's implementation confirms the commitment. Of the seventeen FEBE commands Nelson specifies, none modifies existing Istream content. There is no MODIFY, UPDATE, or REPLACE operation. The absence is structural — the protocol provides no mechanism for mutating stored content.

*Proof.* We wish to show that for every state transition `Σ → Σ'`, `dom(Σ.C) ⊆ dom(Σ'.C)`.

Let `a ∈ dom(Σ.C)` be arbitrary. By S0 (content immutability), `a ∈ dom(Σ.C)` implies the conjunction `a ∈ dom(Σ'.C) ∧ Σ'.C(a) = Σ.C(a)`. The first conjunct yields `a ∈ dom(Σ'.C)` directly. Since `a` was chosen arbitrarily from `dom(Σ.C)`, we have established `(A a : a ∈ dom(Σ.C) : a ∈ dom(Σ'.C))`, which is `dom(Σ.C) ⊆ dom(Σ'.C)` by definition of subset inclusion. ∎

*Formal Contract:*
- *Preconditions:* State transition `Σ → Σ'` in a system satisfying S0 (content immutability).
- *Postconditions:* `dom(Σ.C) ⊆ dom(Σ'.C)`.


## The arrangement and referential integrity

Vstream is where mutability lives. Each document's arrangement `M(d)` maps V-positions to I-addresses, presenting stored content as a readable sequence. Unlike `C`, arrangements change freely — content can be added, removed, and reordered.

**S2 (Arrangement functionality).** Each V-position maps to exactly one I-address, by the `Σ.M(d) : T ⇀ T` partial-function declaration:

`(A d, v, a₁, a₂ : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a₁ ∧ Σ.M(d)(v) = a₂ : a₁ = a₂)`

*Formal Contract:*
- *Derivation:* The `Σ.M(d) : T ⇀ T` partial-function declaration yields the single-image property by unfolding the meaning of "partial function": a function relates each domain element to at most one image, so two images of a common argument coincide.
- *Postconditions:* (single image) `(A d, v, a₁, a₂ : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a₁ ∧ Σ.M(d)(v) = a₂ : a₁ = a₂)` — each V-position has at most one I-address image.
- *Depends:* the `Σ.M(d) : T ⇀ T` axiom (this ASN) — the partial-function declaration from which the single-image property is unfolded.

The bridge between the two state components is a well-formedness condition:

**S3 (Referential integrity).** `(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∈ dom(Σ.C))`

Every V-reference resolves. If a document's arrangement says "at position `v`, display the content at I-address `a`," then `a` must be in `dom(C)`. There are no dangling references.

Any transition that establishes a V-mapping `M(d)(v) = a` must therefore have `a ∈ dom(Σ'.C)` in the post-state. S1 (store monotonicity) then guarantees that once `a` enters `dom(C)` it remains, so a valid reference cannot become dangling through any subsequent state transition.

Content unreferenced by any current arrangement still persists. Since S0's antecedent is `a ∈ dom(Σ.C)` alone, not conditioned on whether `a` appears in any `ran(M(d))`, such content is never reclaimed. Nelson requires this for history — he calls such content "deleted bytes — not currently addressable, awaiting historical backtrack functions, may remain included in other versions," and version reconstruction depends on the availability of Istream fragments from prior arrangements.

*Formal Contract (S3):*
- *Axiom (well-formedness invariant):* In every state `Σ`, `(A d, v : v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∈ dom(Σ.C))` — equivalently, `ran(Σ.M(d)) ⊆ dom(Σ.C)`.
- *Preservation across transitions:* For an operation that adds a V-mapping `M(d)(v) = a`, the post-state must satisfy `a ∈ dom(Σ'.C)` — the I-address must exist in the post-state.
- *Frame:* S3 asserts `ran(M(d)) ⊆ dom(C)` only; the converse `dom(C) ⊆ ⋃_d ran(M(d))` is not asserted.
- *Depends:* S1 (store monotonicity) — once a reference is valid, S1 prevents the target from being removed.


## Content identity

What distinguishes transclusion from coincidence? In conventional systems, identity is by value — two files with identical bytes are "the same." In Xanadu, identity is by address.

**S4 (Origin-based identity).** For I-addresses `a₁`, `a₂` produced by distinct allocation events:

`a₁ ≠ a₂`

regardless of whether `Σ.C(a₁) = Σ.C(a₂)`. Two independent writings of the word "hello" produce distinct I-addresses. A transclusion of existing content shares the original I-address.

S4 follows directly from GlobalUniqueness (ASN-0034), which establishes that no two distinct allocation events — whether from the same allocator or different allocators, whether simultaneous or separated by years — produce the same address. The two-stream architecture exploits this guarantee: when `Σ.M(d₁)(v₁) = Σ.M(d₂)(v₂)` for documents `d₁ ≠ d₂`, the system knows this is transclusion — shared content with a common origin — not coincidental value equality. The structural test for shared identity is address equality, decidable from the addresses alone (T3, ASN-0034) without value comparison.

S4 creates a fundamental asymmetry in the system. The content store `C` is oblivious to values — it does not care whether `C(a₁) = C(a₂)`. But the arrangement family `M` is sensitive to addresses — two arrangements that map to the same I-address share content structurally, while two arrangements that map to different I-addresses with equal values do not. Nelson captures the distinction:

> "Remember the analogy between text and water. Water flows freely, ice does not. The free-flowing, live documents on the network are subject to constant new use and linkage... Any detached copy someone keeps is frozen and dead, lacking access to the new linkage." [LM 2/48]

Live content shares I-addresses. Dead copies create new ones. The difference is structural — computable from the state alone.

*Proof.* We are given I-addresses `a₁, a₂ ∈ dom(Σ.C)` produced by distinct allocation events within a system conforming to T10a (allocator discipline, ASN-0034). We wish to show `a₁ ≠ a₂`.

GlobalUniqueness (ASN-0034) establishes the following invariant: for every pair of addresses `a, b` produced by distinct allocation events in any reachable system state, `a ≠ b`. The invariant's precondition requires only that `a₁` and `a₂` arise from distinct allocation events under T10a — it places no condition on the values `Σ.C(a₁)` and `Σ.C(a₂)`. Since `a₁` and `a₂` are produced by distinct allocation events by hypothesis, GlobalUniqueness yields `a₁ ≠ a₂` directly. ∎

*Formal Contract:*
- *Preconditions:* `a₁, a₂ ∈ dom(Σ.C)` produced by distinct allocation events within a system conforming to T10a (allocator discipline, ASN-0034).
- *Postconditions:* `a₁ ≠ a₂`, regardless of whether `Σ.C(a₁) = Σ.C(a₂)`.
- *Frame:* The content store `C` and value domain `Val` play no role in the proof — distinctness is a property of the addressing scheme alone.


## Sharing

The arrangement function `M(d)` need not be injective. This is not a deficiency but a design requirement — it is what makes transclusion work.

**S5 (Unrestricted sharing).** The same I-address may appear in the ranges of multiple arrangements, and at multiple V-positions within a single arrangement. S0–S3 are consistent with any finite sharing multiplicity — they place no constraint on `|{(d, v) : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a}|`:

`(A N ∈ ℕ :: (E Σ :: Σ is the initial state of a model of S0–S3 ∧ (E a ∈ dom(Σ.C) :: |{(d, v) : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a}| > N)))`

In any particular state, the sharing multiplicity of each address is a definite finite number — possibly zero for orphaned content — but no invariant imposes a uniform bound that holds across all states.

Nelson: "The virtual byte stream of a document may include bytes from any other document." And: "A document may have a window to another document, and that one to yet another, indefinitely. Thus A contains part of B, and so on. One document can be built upon another, and yet another document can be built upon that one, indefinitely." Transclusion is recursive and unlimited.

S4 and S5 together make quotation a first-class structural relationship: any number of documents can quote the same passage, and the system knows they are all quoting — not independently writing — because they share I-addresses.

*Proof.* We wish to show that for every `N ∈ ℕ`, there exists a state `Σ` that is the initial state of a model of S0–S3 (S0, S1 holding vacuously, S2, S3 holding on the state) in which some I-address has sharing multiplicity exceeding `N`. We give two constructions — one for cross-document sharing, one for within-document sharing — each succeeding for arbitrary `N`.

**Shared facts.** Both constructions use the same content store `C = {a ↦ w}` for a single I-address `a` and arbitrary `w ∈ Val`, and V-positions of the form `[1, k]` with `k ≥ 1`. S0 (content immutability) and S1 (store monotonicity) are transition-level invariants — quantified over transitions `Σ → Σ'`. To exhibit each witness as a genuine model of S0–S3 (not merely of the state-level fragment), we take it as the *initial state of the trivial transition system whose transition relation is empty*: with no transition `Σ → Σ'` to range over, the universally quantified S0 and S1 hold vacuously, so the witness models S0 and S1. The only invariants that constrain the single state itself are the state-level S2 and S3, which we discharge per construction. S3 (referential integrity) holds identically in both constructions: the sole I-address referenced by any arrangement is `a`, which lies in the content domain `dom(C) = {a}` by construction. The two constructions differ only in document/V-position multiplicity; we verify the remaining state-level invariant S2 (arrangement functionality) per construction.

**Cross-document construction.** Fix `N ∈ ℕ`. Define state `Σ_N = (C_N, M_N)` by:

- `C_N = {a ↦ w}` for a single I-address `a` and arbitrary value `w ∈ Val`.
- `N + 1` documents `d₁, …, d_{N+1}` with explicit witnesses `dᵢ = [1, 0, 1, 0, i]` for `i = 1, …, N + 1`. The `dᵢ` are pairwise distinct by T3 (CanonicalRepresentation, ASN-0034) since they have distinct last components. Fix a single V-position `v = [1, 1]` shared across all `N + 1` documents, and define each arrangement as `M_N(dᵢ) = {v ↦ a}`. The pairs `(dᵢ, v)` are distinct since the `dᵢ` are distinct.

We verify the construction-specific invariant. S2 (arrangement functionality): each `M_N(dᵢ)` contains a single entry `{v ↦ a}` — the domain has one element, so `M_N(dᵢ)` is a function. With the shared facts, `Σ_N` satisfies the state-level invariants S2, S3.

The sharing multiplicity of `a` in `Σ_N` is `|{(d, v) : v ∈ dom(M_N(d)) ∧ M_N(d)(v) = a}| = N + 1`, since each of the `N + 1` documents contributes exactly one pair `(dᵢ, v)` (with the same fixed `v = [1, 1]` across all `i`). Thus the multiplicity exceeds `N`.

**Within-document construction.** Fix `N ∈ ℕ`. Define state `Σ'_N = (C'_N, M'_N)` by:

- `C'_N = {a ↦ w}` for a single I-address `a` and arbitrary value `w ∈ Val`.
- One document `d = [1, 0, 1, 0, 1]` with `M'_N(d) = {v₁ ↦ a, v₂ ↦ a, …, v_{N+1} ↦ a}` where `vₖ = [1, k]` for `k = 1, …, N + 1` — pairwise distinct V-positions (distinctness follows from distinct last components by T3, ASN-0034).

We verify the construction-specific invariant. S2 (arrangement functionality): the `vₖ` are pairwise distinct by construction (distinct last components, T3 — CanonicalRepresentation, ASN-0034), so each V-position maps to exactly one I-address (namely `a`); `M'_N(d)` is a well-defined function. With the shared facts, `Σ'_N` satisfies the state-level invariants S2, S3.

The within-document sharing multiplicity is `|{v : v ∈ dom(M'_N(d)) ∧ M'_N(d)(v) = a}| = N + 1 > N`.

**Conclusion.** Each construction yields, for arbitrary `N ∈ ℕ`, the initial state of a model of S0–S3 in which the sharing multiplicity exceeds `N`. Since each witness is a genuine model of the full invariant set S0–S3, sharing multiplicity exceeding any given finite bound is consistent with S0–S3 alone. No finite cap on `|{(d, v) : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a}|` is entailed by S0–S3 — neither across documents nor within a single document. ∎

*Formal Contract:*
- *Preconditions:* `N ∈ ℕ` arbitrary.
- *Postconditions:* There exists a state `Σ` — the initial state of a model of S0–S3 — such that for some `a ∈ dom(Σ.C)`, `|{(d, v) : v ∈ dom(Σ.M(d)) ∧ Σ.M(d)(v) = a}| > N`. The construction works both across documents (multiplicity `N + 1` over `N + 1` documents) and within a single document (multiplicity `N + 1` at `N + 1` distinct V-positions).
- *Depends:* S0, S1, S2, S3, T3 (ASN-0034).


## Structural attribution

Every V-position can be traced to the document that originally created its content.

The projection `D(a)` is well-defined only when `zeros(a) ≥ 2` (per T4's field correspondence: `zeros = 0` is node-only, `zeros = 1` is node+user, `zeros ≥ 2` has a document field). Since Istream addresses designate content elements within documents, we require:

**S7b (Element-level I-addresses).** We require that every address in `dom(Σ.C)` is an element-level tumbler: `(A a ∈ dom(Σ.C) :: zeros(a) = 3)`.

By T4's field correspondence, `zeros(a) = 3` means all four identifying fields — node, user, document, element — are present, and the element field contains the content-level address.

*Formal Contract (S7b):*
- *Axiom (design requirement):* `(A a ∈ dom(Σ.C) :: zeros(a) = 3)`.
- *Postconditions:* By T4's field correspondence, all four identifying fields — node, user, document, element — are present and the element field exists. The projections `N(a)`, `U(a)`, `D(a)`, `E(a)` supplied by T4b are all well-defined.
- *Depends:* T4 (HierarchicalParsing, ASN-0034) — field correspondence; T4b (UniqueParse, ASN-0034) — projection definitions; T10a.4 (T4PreservationUnderDiscipline, ASN-0034) — T4-validity (no adjacent zeros, `a₁ ≠ 0 ∧ a_{#a} ≠ 0`) and the bound `zeros(a) ≤ 3`.

**S7a (Document-scoped allocation).** Every Istream address is allocated under the tumbler prefix of the document that created it. That is, for every `a ∈ dom(Σ.C)`, the document-level prefix of `a` — the tumbler `N(a).0.U(a).0.D(a)` obtained by truncating the element field, where `N(a)`, `U(a)`, `D(a)` are the partial projections supplied by T4b (UniqueParse, ASN-0034) — identifies the document whose owner performed the allocation that placed `a` into `dom(C)`.

Nelson's baptism principle establishes it: "The owner of a given item controls the allocation of the numbers under it." A document owner baptises element addresses under that document's prefix, so the home document is ascertainable from the address alone.

*Formal Contract (S7a):*
- *Axiom (design requirement):* `(A a : a ∈ dom(Σ.C) :: the document-level prefix N(a).0.U(a).0.D(a) is the tumbler of the document whose owner performed the allocation that placed a into dom(C))`.
- *Depends:* T4 (HierarchicalParsing, ASN-0034) — defines the prefix structure; T4b (UniqueParse, ASN-0034) — defines projections `N`, `U`, `D`; S7b (Element-level I-addresses) — supplies `zeros(a) = 3` for every `a ∈ dom(Σ.C)`; T10a (AllocatorDiscipline, ASN-0034) — establishes the baptism principle; T10a.4 (T4PreservationUnderDiscipline, ASN-0034) — T4 preservation.

**S7d (Document allocation discipline).** Every document is addressed by a document-level tumbler (`zeros = 2`) arising from an allocation event under T10a's allocator discipline (ASN-0034). Distinct documents arise from distinct allocation events.

*Formal Contract (S7d):*
- *Axiom (design requirement):* Every document tumbler `d` satisfies `zeros(d) = 2` and is the result of an allocation event under T10a; distinct documents arise from distinct allocation events.
- *Postconditions:* By GlobalUniqueness (ASN-0034), distinct documents have distinct document-level tumblers.
- *Depends:* T10a (AllocatorDiscipline, ASN-0034) — allocation events; T10a.4 (T4PreservationUnderDiscipline, ASN-0034) — T4 preservation, here at `zeros = 2`; T4 (HierarchicalParsing, ASN-0034) — field correspondence at `zeros = 2`; GlobalUniqueness (ASN-0034) — uniqueness across allocation events.

**S7 (Structural attribution).** For every `a ∈ dom(Σ.C)`, define the *origin* as the document-level prefix obtained by truncating the element field:

`origin(a) = N(a).0.U(a).0.D(a)`

This is the full document tumbler `N.0.U.0.D` — uniquely identifying the allocating document across the system.

Since I-addresses are permanent (S0) and unique (S4), this attribution is permanent and unseverable.

We note a subtlety. S7 identifies the document that ALLOCATED the I-address — the document where the content was first created. This is distinct from the document where the content currently appears. When content is transcluded from document B into document A, the reader viewing A sees the content, but S7 traces it to B. The distinction between "where I am reading" (Vstream context, document A) and "where this came from" (Istream structure, document B) is precisely the two-stream separation made visible.

*Proof.* We wish to show that for every `a ∈ dom(Σ.C)`, the function `origin(a) = N(a).0.U(a).0.D(a)` is well-defined, uniquely identifies the document that allocated `a`, and that this identification is permanent and unseverable.

**Well-definedness.** By S7b (element-level I-addresses), `zeros(a) = 3`, and by T10a.4 (T4PreservationUnderDiscipline, ASN-0034), `a` is T4-valid; hence T4's field-decomposition machinery applies to `a`. By T4 (HierarchicalParsing, ASN-0034), `zeros(a) = 3` means `a` contains exactly three zero-valued field separators, and the partial projections supplied by T4b (UniqueParse, ASN-0034) — `N(a)`, `U(a)`, `D(a)`, `E(a)` — extract the node, user, document, and element fields respectively, each a finite sequence of natural numbers whose non-separator components are strictly positive — strict positivity discharged not by T4 but by T0's carrier ℕ (ASN-0034), under which `tᵢ ≠ 0 ⇔ tᵢ > 0`. T4 supplies the separator/zero-count structure (`zeros(a) = 3` with no two zeros adjacent and non-zero first and last components); the equivalent reading that each present field is non-empty — has at least one component — is T4a (SyntacticEquivalence, ASN-0034). The projections `N(a)`, `U(a)`, and `D(a)` are therefore all well-defined with at least one strictly positive component each. The truncation `origin(a)` — formed by concatenating the node field, a zero separator, the user field, a zero separator, and the document field — is a well-defined tumbler satisfying `zeros(origin(a)) = 2`, placing it at the document level in T4's hierarchy.

**Identification.** By S7a (document-scoped allocation), every I-address is allocated under the tumbler prefix of the document that created it. The document-level prefix of `a` — precisely `origin(a)`, the tumbler `N.0.U.0.D` obtained by truncating the element field — identifies the document whose owner performed the allocation that placed `a` into `dom(C)`. This is not a lookup or annotation: the address structurally encodes its provenance. S7a ensures that `origin(a)` IS the allocating document's tumbler.

**Uniqueness across documents.** By S7d's postcondition, distinct documents have distinct document-level tumblers. By T3 (CanonicalRepresentation, ASN-0034), this distinctness is decidable by component-wise comparison. Therefore, for any `a₁, a₂ ∈ dom(Σ.C)` allocated under distinct documents: `origin(a₁) ≠ origin(a₂)`. The origin function discriminates allocating documents without ambiguity.

**Permanence.** By S0 (content immutability), once `a ∈ dom(Σ.C)`, then `a ∈ dom(Σ'.C)` for all successor states `Σ'` — the address persists. Since `a` is a tumbler — a fixed sequence of components, not a mutable reference — and `origin(a)` is computed from the components of `a` alone via T4's deterministic field decomposition, `origin(a)` yields the same result in every state in which `a` exists. ∎

*Formal Contract:*
- *Preconditions:* `a ∈ dom(Σ.C)` in a system conforming to S7a (document-scoped allocation), S7b (element-level I-addresses), S7d (document allocation discipline), T4 (HierarchicalParsing, ASN-0034) — separator/zero-count structure, T4a (SyntacticEquivalence, ASN-0034) — the non-empty-field reading of each present field, T0 (CarrierSetDefinition, ASN-0034) — strict positivity of non-separator components via the carrier ℕ, T4b (UniqueParse, ASN-0034) — supplies the projections `N(a)`, `U(a)`, `D(a)`, `E(a)` from which `origin(a)` is computed, T10a (allocator discipline, ASN-0034), and T10a.4 (T4PreservationUnderDiscipline, ASN-0034) — T4 preservation. The strict equality `zeros(a) = 3` itself comes from S7b axiomatically.
- *Postconditions:* (a) `origin(a)` is well-defined and is a document-level tumbler with `zeros(origin(a)) = 2`. (b) `origin(a)` is the tumbler of the document that allocated `a`. (c) For `a₁, a₂` allocated under distinct documents, `origin(a₁) ≠ origin(a₂)`. (d) `origin(a)` is invariant across all states in which `a ∈ dom(Σ.C)`.
- *Frame:* The content values `Σ.C(a)` and arrangement functions `Σ.M(d)` play no role — attribution is a property of the addressing scheme alone.

## Correspondence-run partition

The arrangement `M(d)` maps individual V-positions to I-addresses. Because `dom(M(d))` is finite (S8-fin), the mapping decomposes into finitely many *correspondence runs* — maximal contiguous blocks of V-positions whose images advance in lockstep with them under ordinal displacement. S8 establishes that this run decomposition exists, partitions `dom(M(d))`, and is unique.

**S8-fin (Finite arrangement).** For each document `d`, `dom(Σ.M(d))` is finite. This is a design requirement on every reachable state: no document arrangement is permitted to hold infinitely many V-positions.

*Formal Contract:*
- *Axiom (design requirement):* For every state `Σ` and document `d`, `dom(Σ.M(d))` is a finite set.
- *Postconditions:* `|dom(Σ.M(d))| < ∞` — the arrangement has finite cardinality. Consequently `ran(Σ.M(d))` is finite (image of a finite set under a function).
- *Frame:* No constraint on the unbounded growth of `dom(C)`; only individual arrangements are required to be finite at any given state.
**subspace (V-position subspace identifier).** For any tumbler `v` of depth `#v ≥ 1`, define:

`subspace(v) = v₁`

extracting the subspace identifier as the first component of a V-position.

*Formal Contract:*
- *Signature:* `subspace : T → ℕ` — projects the first component of a tumbler.
- *Preconditions:* `v ∈ T`, `#v ≥ 1` (so that `v₁` is well-defined as the first component of a non-empty tumbler).
- *Definition:* `subspace(v) = v₁`.

**S8-depth (Fixed-depth V-positions).** Within a given subspace `s` of document `d`, all V-positions share the same tumbler depth:

`(A d, u, w : u ∈ dom(Σ.M(d)) ∧ w ∈ dom(Σ.M(d)) ∧ subspace(u) = subspace(w) : #u = #w)`

Gregory's evidence supports it: V-addresses in the text subspace consistently use the form `s.x` — two tumbler digits, where `s` is the subspace identifier and `x` is the ordinal. Any correct implementation must satisfy this constraint.

*Formal Contract:*
- *Axiom (design requirement):* `(A d, u, w : u ∈ dom(Σ.M(d)) ∧ w ∈ dom(Σ.M(d)) ∧ subspace(u) = subspace(w) : #u = #w)`.
- *Postconditions:* Within a subspace `s` of document `d`, if `V_s(d) ≠ ∅` then there exists a common depth `m_s ≥ 2` (by S8a) such that every V-position with `v₁ = s` has length `m_s`. For empty `V_s(d)` no witness depth is asserted. Distinct subspaces may have distinct depths.
- *Depends:* S8a — for the lower bound `m_s ≥ 2`.

S8-depth allows us to define "consecutive V-positions" precisely. Within a subspace, consecutive positions differ only at the ordinal (last) component: a position `v` is followed by `shift(v, 1)` (equivalently `v ⊕ δ(1, #v)` per OrdinalShift, ASN-0034), the next ordinal at the same depth.

### Shift preservation for V-positions

Ordinal shift `shift(v, n)` (OrdinalShift, ASN-0034) preserves a V-position's subspace identifier and its S8a well-formedness, as the following lemma establishes.

**OrdShiftHom** — *OrdinalShiftPreservation* (LEMMA). For a V-position `v` with `#v = m ≥ 2` and `n ≥ 1`:

(a) `subspace(shift(v, n)) = subspace(v)`.

(b) When `v` satisfies S8a, `shift(v, n)` satisfies S8a.

*Proof.* Write `shift(v, n) = v ⊕ δ(n, m)` with `δ(n, m) = [0, ..., 0, n]` of length `m` (OrdinalShift, OrdinalDisplacement, ASN-0034). By OrdinalDisplacement, `actionPoint(δ(n, m)) = m`, so the addition is well-defined since `actionPoint(δ(n, m)) = m ≤ #v`. By TumblerAdd, the result `r = v ⊕ δ(n, m)` is built component-wise: for `1 ≤ i < m`, `rᵢ = vᵢ` (these positions precede the action point and are copied from `v`); at `i = m`, `rₘ = vₘ + n`. There are no positions beyond the action point, and `#r = m` (TA0, ASN-0034).

*Part (a).* Since `m ≥ 2`, position 1 lies in the copy-from-`v` region, so `r₁ = v₁`. By definition `subspace(r) = r₁ = v₁ = subspace(v)`.

*Part (b).* Assume `v` satisfies S8a: `zeros(v) = 0`, `#v = m ≥ 2`, and `vᵢ ≥ 1` for every `i`. For `1 ≤ i < m`, `rᵢ = vᵢ ≥ 1`; at `i = m`, `rₘ = vₘ + n ≥ 1 + 1 > 0`. Every component of `r` is positive, so `zeros(r) = 0` and `(A i : 1 ≤ i ≤ #r : rᵢ > 0)`, with `#r = m ≥ 2`. Hence `shift(v, n)` satisfies S8a. ∎

*Instance.* Let `v = [1, 3, 5]` (text subspace `v₁ = 1`, depth `m = 3`, satisfying S8a) and `n = 2`. Then `shift(v, 2) = v ⊕ δ(2, 3) = [1, 3, 5] ⊕ [0, 0, 2] = [1, 3, 7]` (action point 3; components 1 and 2 copied from `v`, component 3 receives `5 + 2 = 7`). (a) `subspace(shift(v, 2)) = [1, 3, 7]₁ = 1 = v₁ = subspace(v)`. (b) `[1, 3, 7]` has `zeros = 0`, every component positive (`1, 3, 7 ≥ 1`), and depth `3 ≥ 2`, so S8a holds on `shift(v, 2)`.

*Formal Contract:*
- *Preconditions:* `v ∈ T`, `#v = m ≥ 2`, `n ≥ 1`.
- *Postconditions:* (a) `subspace(shift(v, n)) = subspace(v)`. (b) When `v` satisfies S8a, `shift(v, n)` satisfies S8a.
- *Depends:* OrdinalShift (ASN-0034) — `shift(v, n) = v ⊕ δ(n, m)`; OrdinalDisplacement (ASN-0034) — `δ(n, m) = [0, ..., 0, n]` with action point `m`; TumblerAdd (ASN-0034) — the component formula copying positions before the action point; TA0 (length preservation, ASN-0034) — `#shift(v, n) = m`; S8a (V-position well-formedness) — supplies `vᵢ ≥ 1` for part (b).

**S8 (Correspondence-run partition).** For each document `d`, the active V-positions `dom(Σ.M(d))` decompose into finitely many *correspondence runs*. Under the convention `shift(t, 0) := t`, a correspondence run is a triple `(v, a, n)` with `v ∈ dom(M(d))`, `a = M(d)(v)`, and `n ≥ 1`, such that for every `k` with `0 ≤ k < n`:

(a) **Lockstep displacement** — `shift(v, k) ∈ dom(M(d))` and `M(d)(shift(v, k)) = shift(a, k)`: the V-positions and their images advance in lockstep under ordinal displacement.

(b) **Well-defined label** — `a = M(d)(v)` exists and is unique because `M(d)` is a function (S2), and `a ∈ dom(Σ.C)` by referential integrity (S3). Each lockstep image `shift(a, k)` (for `0 ≤ k < n`, with the convention `shift(a, 0) := a`) likewise lies in `dom(Σ.C)`.

A run is *maximal* when it admits neither forward extension (no run `(v, a, n+1)`) nor backward extension (no lockstep predecessor `u` with `shift(u, 1) = v`, `u ∈ dom(M(d))`, `shift(M(d)(u), 1) = a`). The maximal runs partition `dom(Σ.M(d))`, and the maximal-run decomposition is unique.

*Proof.* We partition `dom(M(d))` by constructing the maximal correspondence runs explicitly, then derive their existence, uniqueness, and the displacement identity from a single combinatorial structure: the lockstep-successor partial function.

**Lockstep successor.** Define the partial function `succ` on `dom(M(d))` by: `succ(v) = shift(v, 1)` precisely when `shift(v, 1) ∈ dom(M(d))` and `M(d)(shift(v, 1)) = shift(M(d)(v), 1)`; otherwise `succ(v)` is undefined. When `succ(v)` is defined we say `v` is *lockstep-linked* to `shift(v, 1)`.

**`succ` stays within a subspace and preserves well-formedness.** Let `v ∈ dom(M(d))`, so `v` satisfies S8a. By OrdShiftHom (a), `subspace(shift(v, 1)) = subspace(v)`, and by OrdShiftHom (b) `shift(v, 1)` again satisfies S8a; by S8-depth it shares `v`'s depth `m`. Thus every lockstep link stays inside one subspace, and `succ` is a partial function on the well-formed V-positions of that subspace. On the image side, when `v` is lockstep-linked we have `shift(v, 1) ∈ dom(M(d))`, so `shift(a, 1) = M(d)(shift(v, 1)) ∈ ran(M(d)) ⊆ dom(Σ.C)` by S3.

**`succ` is injective and acyclic.** Suppose `succ(u) = succ(u')`. Then `shift(u, 1) = shift(u', 1)`. Since `shift` preserves depth, `#u = #shift(u, 1) = #shift(u', 1) = #u'`; TS2 (ShiftInjectivity, ASN-0034) applied at this common depth gives `u = u'`. So `succ` is injective on its domain — each V-position has at most one lockstep successor and at most one lockstep predecessor. There are no cycles: TS4 (ShiftStrictIncrease, ASN-0034) gives `shift(v, 1) > v`, so each `succ`-step strictly increases under T1; a cycle would yield `v < v`, contradicting T1 irreflexivity (ASN-0034).

**Chain decomposition.** The facts just established pin down the shape of `succ`'s graph: each vertex has at most one lockstep successor (`succ` is a partial function, out-degree ≤ 1) and at most one lockstep predecessor (`succ` is injective, in-degree ≤ 1), and `dom(M(d))` is finite (S8-fin) with no cycles (acyclicity). A finite directed graph whose every vertex has in-degree and out-degree at most one and which contains no cycle decomposes into disjoint simple paths — its connected components are exactly the maximal chains. Concretely, for each `v ∈ dom(M(d))` form its orbit: follow `succ` forward — `v, succ(v), succ(succ(v)), …` — until `succ` is undefined (the forward walk terminates because `dom(M(d))` is finite by S8-fin and acyclicity forbids revisiting), and follow the inverse `succ⁻¹` (single-valued by injectivity) backward until undefined. The concatenated walk is a finite sequence `v⁰, v¹, …, v^{n−1}` with `vⁱ⁺¹ = succ(vⁱ)`, in which `v⁰` has no lockstep predecessor (the *head*) and `v^{n−1}` has no lockstep successor (the *tail*). Membership in the same orbit is an equivalence relation, so the orbits partition `dom(M(d))`; each is a *maximal chain*.

**Chains are runs, and the displacement identity holds at every `k`.** Let `v⁰, …, v^{n−1}` be a maximal chain. Put `v = v⁰`, `a = M(d)(v⁰)`, length `n`. We show by induction on `i` that `vⁱ = shift(v, i)` and `M(d)(vⁱ) = shift(a, i)` for `0 ≤ i < n`. The base `i = 0` is the convention `shift(v, 0) = v` and `shift(a, 0) = a`. For the step, assume `vⁱ = shift(v, i)` and `M(d)(vⁱ) = shift(a, i)`; since `vⁱ⁺¹ = succ(vⁱ)` is defined, `vⁱ⁺¹ = shift(vⁱ, 1) = shift(shift(v, i), 1)` and `M(d)(vⁱ⁺¹) = shift(M(d)(vⁱ), 1) = shift(shift(a, i), 1)`. We collapse the inner-then-outer shift to `shift(·, i+1)` by cases on `i`. At `i = 0` the inner shift amount is 0, outside TS3's preconditions (`n₁ ≥ 1`); here `shift(shift(v, 0), 1) = shift(v, 1)` and `shift(shift(a, 0), 1) = shift(a, 1)` by the convention `shift(t, 0) := t`. For `i ≥ 1` both shift amounts are `≥ 1`, so TS3 (ShiftComposition, ASN-0034) applies and gives `shift(shift(v, i), 1) = shift(v, i+1)` and `shift(shift(a, i), 1) = shift(a, i+1)`. Either way `vⁱ⁺¹ = shift(v, i+1)` and `M(d)(vⁱ⁺¹) = shift(a, i+1)`. For `i = 0`, `shift(v, 0) = v ∈ dom(M(d))` is a well-formed V-position by S8a; for `i ≥ 1`, OrdShiftHom makes each `shift(v, i)` a well-formed V-position of the same subspace and depth; in either case `shift(a, i) = M(d)(vⁱ) ∈ ran(M(d)) ⊆ dom(Σ.C)` by S3. So `(v, a, n)` is a correspondence run and conjunct (a)'s displacement identity holds at every `0 ≤ k < n` with both sides well-formed. Conjunct (b) holds by S2 and S3 as above.

**Maximality and uniqueness.** The run `(v, a, n)` built from a maximal chain is itself maximal: its head `v⁰` has no lockstep predecessor (no backward extension) and its tail `v^{n−1}` has no lockstep successor (no forward extension). Conversely any maximal run is a maximal chain. Because the orbit of every element under an injective acyclic partial function is uniquely determined — the forward and backward walks are forced at each step — the decomposition into maximal chains is unique; hence the maximal-run decomposition is unique.

**Partition.** *Empty case.* When `dom(M(d)) = ∅` there are zero orbits and hence zero maximal runs, and the empty union of runs partitions the empty set; the remaining clauses treat `dom(M(d)) ≠ ∅`. *Coverage.* Every `v ∈ dom(M(d))` lies in its own orbit, hence in exactly one maximal run; at minimum `(v, M(d)(v), 1)` is a run (conjunct (a) holds trivially at `k = 0` by the convention `shift(t, 0) := t`) that extends to the unique maximal run through `v`. *Disjointness.* Two distinct maximal runs are vertex-disjoint: if they shared a V-position `w`, both would equal the orbit of `w` and so coincide. *Finiteness.* By S8-fin, `dom(M(d))` is finite, so there are finitely many orbits, each finite, hence finitely many maximal runs. Taking the union over subspaces — each chain lying in a single subspace by OrdShiftHom (a) — the maximal runs partition `dom(M(d))`, establishing conjuncts (a) and (b). ∎

*Formal Contract:*
- *Preconditions:* `dom(M(d))` finite (S8-fin); `M(d)` a function (S2); referential integrity (S3); `(A v ∈ dom(M(d)) :: zeros(v) = 0 ∧ #v ≥ 2 ∧ (A i : 1 ≤ i ≤ #v : vᵢ > 0))` (S8a); within each subspace, all V-positions share a common depth (S8-depth). Convention: `shift(t, 0) := t`.
- *Postconditions:* `dom(M(d))` is the disjoint union of finitely many maximal correspondence runs `(vⱼ, aⱼ, nⱼ)`: (a) within each run, `shift(vⱼ, k) ∈ dom(M(d))` and `M(d)(shift(vⱼ, k)) = shift(aⱼ, k)` for `0 ≤ k < nⱼ`, with `shift(vⱼ, k)` a well-formed V-position (OrdShiftHom) and `shift(aⱼ, k) ∈ dom(Σ.C)` (S3); (b) the label `aⱼ = M(d)(vⱼ)` is well-defined by S2 and lies in `dom(Σ.C)` by S3; (c) the maximal-run decomposition is unique.
- *Depends:* (*Local properties*) S2 (ArrangementFunctionality) — uniquely determined image `a = M(d)(v)` and the labels; S3 (referential integrity) — `M(d)(v) ∈ dom(Σ.C)`, placing every lockstep image in the content domain; S8a — well-formed V-positions; S8-depth — common depth within a subspace, used for the equal-depth TS2 application; S8-fin — finite `dom(M(d))`, bounding chains; OrdShiftHom (OrdinalShiftPreservation) — subspace and S8a preservation under shift, confining each chain to one subspace. (*Foundation claims, ASN-0034*) T1 (LexicographicOrder) — irreflexivity, ruling out lockstep cycles; TS2 (ShiftInjectivity) — injectivity of `succ` at common depth; TS3 (ShiftComposition) — `shift(shift(·, i), 1) = shift(·, i+1)`, lifting lockstep edges to the displacement identity at general `k`; TS4 (ShiftStrictIncrease) — `shift(v, 1) > v`, supplying acyclicity; OrdinalShift, OrdinalDisplacement — the shift operation and its action-point semantics.

## Arrangement contiguity

Nelson states that the Vstream is always a "dense, contiguous sequence" — after removal, "the v-stream addresses of any following characters in the document are [decreased] by the length of the [deleted] text" [LM 4/66]. The Vstream has no concept of empty positions: "if you have 100 bytes, you have addresses 1 through 100." Nelson's "addresses 1 through 100" describes character positions, so the contiguity properties below are stated for the text subspace (S = 1).

Abbreviate `S = subspace(v) = v₁` (per S8a), and write `V_S(d) = {v ∈ dom(M(d)) : subspace(v) = S}` for the set of V-positions in subspace S of document d. The specialization to the text subspace is `V_1(d) = {v ∈ dom(M(d)) : subspace(v) = 1}`. All V-positions in a given subspace share the same tumbler depth (S8-depth).

**D-CTG (VContiguity).** For each document d, V_1(d) (the text subspace) is either empty or occupies every intermediate position between its extremes:

`(A d, u, q : u ∈ V_1(d) ∧ q ∈ V_1(d) ∧ u < q : (A v : subspace(v) = 1 ∧ #v = #u ∧ zeros(v) = 0 ∧ u < v < q : v ∈ V_1(d)))`

In words: within the text subspace, V-positions form a contiguous ordinal range with no gaps. If positions [1, 3] and [1, 7] are occupied, then every position [1, k] with 3 < k < 7 must also be occupied.

*Formal Contract:*
- *Axiom (design requirement):* `(A d, u, q : u ∈ V_1(d) ∧ q ∈ V_1(d) ∧ u < q : (A v : subspace(v) = 1 ∧ #v = #u ∧ zeros(v) = 0 ∧ u < v < q : v ∈ V_1(d)))`.
- *Preconditions:* `subspace(v) = 1`; `zeros(v) = 0` ⟺ S8a positivity, by T0; V-positions share a common depth (S8-depth).
- *Postconditions:* V_1(d) is either empty or occupies every position strictly between its extremes (at the fixed depth).
- *Frame:* D-CTG is a constraint on well-formed text-subspace arrangements.
- *Depends:* S8a (V-position well-formedness); S8-depth (common depth within subspace); T1 (LexicographicOrder, ASN-0034) — defines the order.

For the text subspace at depth m = 2, this is a finite condition: the intermediates between [1, a] and [1, b] are the finitely many [1, i] with a < i < b. Combined with S8-fin (dom(M(d)) is finite), contiguity at depth 2 says V_1(d) occupies a single unbroken block of ordinals.

**D-CTG-depth (SharedPrefixReduction).** For depth m ≥ 3, all positions in a non-empty V_1(d) share components 2 through m − 1. Contiguity reduces to contiguity of the last component alone — structurally identical to the depth 2 case.

*Proof.* Let V_1(d) be non-empty with common depth `m` (S8-depth) and `m ≥ 3` (non-triviality bound, per the Preconditions). Suppose for contradiction that V_1(d) contains two positions u and x with u < x (both depth m) whose first point of disagreement is at component j with 2 ≤ j ≤ m − 1 — that is, uᵢ = xᵢ for all i < j, and uⱼ < xⱼ (the inequality follows from u < x by T1(i), since j is the first disagreeing component and j ≤ min(m, m)).

We construct infinitely many intermediates. For any natural number n > uⱼ₊₁, define w of length m by:

- wᵢ = uᵢ for 1 ≤ i ≤ j (agreeing with u on the first j components),
- wⱼ₊₁ = n,
- wᵢ = 1 for j + 2 ≤ i ≤ m (an empty range when j = m − 1, in which case wⱼ₊₁ = w_m is already the last component; otherwise this clause fills components j + 2 through m).

Then w has depth m (it has m components by construction), and subspace(w) = w₁ = u₁ = 1 (since j ≥ 2, the first component is copied from u). We verify u < w < x:

- **w > u**: w agrees with u on components 1 through j. At component j + 1, wⱼ₊₁ = n > uⱼ₊₁. Since j + 1 ≤ m = min(m, m), by T1(i), w > u.
- **w < x**: w agrees with x on components 1 through j − 1 (since u and x agree on these components by the definition of j). At component j, wⱼ = uⱼ < xⱼ. Since j ≤ m − 1 ≤ min(m, m), by T1(i), w < x.

We also verify that w satisfies S8a — necessary because D-CTG ranges over V_1(d) ⊆ dom(M(d)), and every position in dom(M(d)) satisfies S8a. By construction, every component of w is at least 1: wᵢ = uᵢ ≥ 1 for i ≤ j by S8a applied to u; wⱼ₊₁ = n > uⱼ₊₁ ≥ 1 (again by S8a on u); and wᵢ = 1 for j + 2 ≤ i ≤ m. Hence zeros(w) = 0 and `(A i : 1 ≤ i ≤ #w : wᵢ > 0)`. Combined with #w = m ≥ 3 ≥ 2, w satisfies S8a — so the candidate w qualifies for D-CTG's consequent.

Since u < w < x, subspace(w) = 1, #w = m = #u, and w satisfies S8a, D-CTG requires w ∈ V_1(d). We now exhibit infinitely many admissible values of n. T0(a) (UnboundedComponentValues, ASN-0034) supplies, for any natural-number bound M, one witness n ∈ ℕ with n > M. Iterating: starting from M₀ = uⱼ₊₁, T0(a) supplies n₁ > M₀; setting M₁ = n₁, T0(a) supplies n₂ > M₁ ≥ n₁; continuing, we obtain a strictly increasing sequence n₁ < n₂ < n₃ < … of natural numbers, all exceeding uⱼ₊₁. The sequence is infinite and pairwise distinct. Distinct values of n yield distinct tumblers w (they differ at component j + 1, so by T3, CanonicalRepresentation, ASN-0034, they are unequal). This produces infinitely many distinct positions in V_1(d), contradicting S8-fin (dom(M(d)) is finite).

Therefore no two positions in V_1(d) can disagree at any component j with 2 ≤ j ≤ m − 1. All positions share components 2 through m − 1, and contiguity reduces to contiguity of the last component (component m) alone. ∎

*Formal Contract:*
- *Preconditions:* V_1(d) non-empty; common depth `m` (S8-depth); `m ≥ 3`.
- *Postconditions:* `(A u, x ∈ V_1(d), j : 2 ≤ j ≤ m − 1 : uⱼ = xⱼ)`. Contiguity of V_1(d) reduces to contiguity of the m-th (last) component.
- *Depends:* (*Local properties*) D-CTG (VContiguity) — any tumbler strictly between two positions in subspace 1 at depth `m` lies in `V_1(d)`; S8a — `m ≥ 2` and componentwise positivity of V-positions; S8-depth — common depth `#w = m`; S8-fin — finiteness of `V_1(d)`. (*Foundation claims, ASN-0034*) T0(a) (UnboundedComponentValues) — for any bound `M`, a natural-number witness `n > M`; T1 case (i) (LexicographicOrder) — first-divergence comparison; T3 (CanonicalRepresentation) — distinct component sequences yield distinct tumblers.

Nelson's statement specifies not just contiguity but also the starting ordinal: "addresses 1 through 100," not "42 through 141." All ordinal numbering in the tumbler system starts at 1: the first child is always .1 (LM 4/20), link positions within a document begin at 1 (LM 4/31), and position 0 is structurally unavailable since zero serves as a field separator (T4, ASN-0034). V-positions follow the same convention.

**D-MIN (VMinimumPosition).** For each document d with V_1(d) non-empty:

`min(V_1(d)) = [1, 1, ..., 1]`

where the tuple has length m (the common depth of V-positions in the text subspace per S8-depth), and every component is 1.

At depth 2 this gives min(V_1(d)) = [1, 1].

*Formal Contract:*
- *Axiom (design requirement):* `V_1(d) ≠ ∅ ⟹ min(V_1(d)) = [1, 1, ..., 1]` of length `m_1` (the common depth per S8-depth).
- *Preconditions:* V_1(d) non-empty; common depth `m_1` (S8-depth) with `m_1 ≥ 2` (S8a).
- *Postconditions:* Every component of `min(V_1(d))` equals 1; in particular the text subspace identifier `min(V_1(d))₁ = 1` and the within-subspace ordinal starts at the minimum positive value.
- *Depends:* S8a, S8-depth, T1 (LexicographicOrder, ASN-0034) — defines `min`.

We now derive the general form: the contiguity, minimum, and finiteness constraints together force V_1(d) into a single block of last-component values. The proof below establishes this in four steps.

**D-SEQ (SequentialPositions).** For each document d, if V_1(d) is non-empty, then there exists n ≥ 1 such that:

`V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n}`

where the tuple has length m, the common V-position depth in the text subspace (S8-depth). By S8a, every V-position has depth `≥ 2`, so `m ≥ 2`; the derivation below relies on this lower bound. At depth 2 this gives V_1(d) = {[1, k] : 1 ≤ k ≤ n}, matching Nelson's "addresses 1 through n."

*Proof.* Let V_1(d) be non-empty and let m be the common depth of all V-positions in the text subspace (S8-depth guarantees a common depth exists). By S8a, every V-position has `#v ≥ 2`, so `m ≥ 2`.

**Step 1: shared prefix.** We show that every position in V_1(d) has the form [1, 1, …, 1, k] — that is, components 2 through m − 1 are all equal to 1, with only the last component varying.

*Case m = 2.* Every position has exactly two components. By the definition `V_1(d) = {v ∈ dom(M(d)) : subspace(v) = 1}` together with `subspace(v) = v₁`, every position in V_1(d) has `v₁ = 1` — the subspace identifier sits at component 1. The second component is a single ordinal. There are no intermediate components (components 2 through m − 1 is the empty range 2 through 1), so the shared-prefix condition holds vacuously. Every position is [1, k] for some k, which is [1, 1, …, 1, k] with zero intervening 1s.

*Case m ≥ 3.* By D-CTG-depth (SharedPrefixReduction), all positions in V_1(d) share components 2 through m − 1. By D-MIN (VMinimumPosition), the minimum element of V_1(d) is [1, 1, …, 1] — a tuple of length m with every component equal to 1. Since the minimum shares components 2 through m − 1 with every other position, and those components of the minimum are all 1, every position in V_1(d) has components 2 through m − 1 equal to 1. Every position is therefore [1, 1, …, 1, k] for some value k at the m-th component.

**Step 2: minimum k.** By D-MIN, min(V_1(d)) = [1, 1, …, 1] of length m. In the representation [1, 1, …, 1, k], the minimum has k = 1 at the last component. Since the minimum is in V_1(d), the set of k-values attained by positions in V_1(d) includes 1.

**Step 3: contiguity of k-values.** Let k₁ < k₂ be two values attained by positions v₁ = [1, 1, …, 1, k₁] and v₂ = [1, 1, …, 1, k₂] in V_1(d). Both have subspace 1 and depth m. By T1(i) (LexicographicOrder, ASN-0034), v₁ < v₂ since they agree on components 1 through m − 1 and differ first at component m where k₁ < k₂. For any k ∈ ℕ with k₁ < k < k₂, the tuple w = [1, 1, …, 1, k] satisfies subspace(w) = 1, #w = m, and v₁ < w < v₂ (again by T1(i), since w agrees with both on components 1 through m − 1 and k₁ < k < k₂ at component m). Moreover w satisfies S8a: every component is strictly positive — the leading m − 1 components are all 1, and the last component k satisfies k > k₁ ≥ 1 — so zeros(w) = 0; and #w = m ≥ 2 inherits the depth bound S8a places on v₁. By D-CTG (VContiguity), w ∈ V_1(d). Therefore every k ∈ ℕ between any two attained k-values is itself attained — the k-values form a contiguous range.

**Step 4: finiteness.** By S8-fin (Finite arrangement), dom(M(d)) is finite, so V_1(d) ⊆ dom(M(d)) is finite. The k-values form a finite contiguous range.

**Assembly.** The k-values form a finite contiguous set of positive integers (Step 3, Step 4) that contains 1 (Step 2). Let n = max(k-values); this maximum is well-defined since the set is finite and non-empty (1 ∈ k-values). Then n ≥ 1. By Step 3 applied between 1 and n, every integer with 1 ≤ k ≤ n is attained, so {1, …, n} ⊆ k-values. By definition of n as the maximum, k-values ⊆ {1, …, n}. Hence the k-values are exactly {1, 2, …, n}. By Step 1, V_1(d) = {[1, 1, …, 1, k] : 1 ≤ k ≤ n}. ∎

*Formal Contract:*
- *Preconditions:* V_1(d) non-empty; common V-position depth m (S8-depth), with `m ≥ 2` inherited from S8a.
- *Postconditions:* `(E n : n ≥ 1 : V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n})` where each tuple has length m.
- *Depends:* (*Local properties*) D-CTG (VContiguity) — any tumbler strictly between attained positions in subspace 1 at depth `m` lies in `V_1(d)`; D-CTG-depth (SharedPrefixReduction) — at `m ≥ 3`, all positions in `V_1(d)` share components 2 through `m − 1`; D-MIN (VMinimumPosition) — `min(V_1(d)) = [1, …, 1]`; S8a — `m ≥ 2`; S8-depth — the common depth `m`; S8-fin — finiteness of `V_1(d)`. (*Foundation claims, ASN-0034*) T1 case (i) (LexicographicOrder) — first-divergence comparison.

D-CTG is a design constraint on well-formed document states. We verify the base case: before any operations, dom(M(d)) = ∅ for all d (the arrangement is a partial function; no content has been allocated, so no V-mapping exists), so V_1(d) = ∅. D-CTG holds vacuously (no u, q exist to trigger its antecedent), and D-MIN holds vacuously (its antecedent requires V_1(d) non-empty).

## Valid insertion position

When V_1(d) is contiguous with |V_1(d)| = N positions, we write its elements as v₀, v₁, ..., v_{N−1} where v₀ is the minimum (D-MIN) and v_{j+1} = shift(v_j, 1) for 0 ≤ j < N − 1 (D-SEQ).

**Definition (ValidInsertionPosition, non-empty case).** For a document `d` with `V_1(d) ≠ ∅`, the *binary* predicate `ValidInsertionPosition(d, v)` is satisfied when:

- The common V-position depth `m` of V_1(d) is fixed by S8-depth. By S8a, `m ≥ 2`.
- Setting `N = |V_1(d)|`, the predicate holds iff `v = min(V_1(d))` or `v = shift(min(V_1(d)), j)` for some `j ∈ {1, ..., N}`.

**Definition (ValidFirstInsertionPosition, empty case).** For a document `d` with `V_1(d) = ∅`, the *ternary* predicate `ValidFirstInsertionPosition(d, v, m)` is satisfied when `m ∈ ℕ` with `m ≥ 2` and `v = [1, 1, ..., 1]` of depth `m`.

*Formal Contract (ValidInsertionPosition, non-empty case).*
- *Signature:* `ValidInsertionPosition(d, v)` — a *binary* predicate on document `d` and V-position `v`. The common V-position depth `m` is determined by `d` via S8-depth and read from state.
- *Preconditions:* Document `d` with `V_1(d) ⊆ dom(M(d))` non-empty; D-CTG holds on V_1(d); D-MIN gives `min(V_1(d)) = [1, ..., 1]` and D-SEQ gives `V_1(d) = {[1, ..., 1, k] : 1 ≤ k ≤ N}` (both needed to discharge the explicit form (d)); `m ≥ 2` is the common depth of V_1(d) by S8-depth and S8a.
- *Definition:* `ValidInsertionPosition(d, v)` holds iff, writing `N = |V_1(d)|`, `v = min(V_1(d))` or `v = shift(min(V_1(d)), j)` for some `j ∈ {1, ..., N}`.
- *Postconditions:* (a) `subspace(v) = 1` and `#v = m` (the state-fixed common depth). (b) `v` satisfies S8a: `zeros(v) = 0` and all components positive. (c) For fixed `d`, exactly `N + 1` values of `v` satisfy the predicate. (d) The explicit form is `v = [1, 1, ..., 1, 1 + j]` of depth `m`, with last component `1 + j` and all `m − 1` preceding components equal to 1 (matching the D-SEQ notation).
- *Derivation:* By D-MIN, `min(V_1(d)) = [1, 1, ..., 1]` of depth `m`. By OrdinalShift (ASN-0034), `shift([1, ..., 1], j)` leaves the leading `m − 1` components unchanged and advances the last component to `1 + j`, so `shift([1, ..., 1], j) = [1, ..., 1, 1 + j]` for `j ≥ 1`; at `j = 0` the position is `v = min(V_1(d)) = [1, ..., 1]` by D-MIN. This is (d). Every component is then `≥ 1` — the leading `m − 1` equal 1, the last `1 + j ≥ 1` — so `zeros(v) = 0` with componentwise positivity (b), and OrdShiftHom (a) fixes `v₁ = 1` as the text subspace identifier. For `j ≠ j'` in `{0, ..., N}` the last components `1 + j ≠ 1 + j'` (NAT-order, ASN-0034), so the length-`m` tumblers diverge at position `m` and are distinct by T3 (ASN-0034), giving exactly `N + 1` positions (c).
- *Depends:* D-MIN, D-CTG, D-CTG-depth, D-SEQ; S8a, S8-fin, S8-depth; OrdShiftHom (subspace and S8a preservation), OrdinalShift (last-component value, ASN-0034); T3 (ASN-0034).

*Formal Contract (ValidFirstInsertionPosition, empty case).*
- *Signature:* `ValidFirstInsertionPosition(d, v, m)` — a *ternary* predicate on document `d`, V-position `v`, and depth `m`.
- *Preconditions:* Document `d` with `V_1(d) = ∅`; `m ∈ ℕ` with `m ≥ 2`.
- *Definition:* `ValidFirstInsertionPosition(d, v, m)` holds iff `v = [1, 1, ..., 1]` of depth `m`.
- *Postconditions:* (a) `subspace(v) = 1` and `#v = m`. (b) `v` satisfies S8a: `zeros(v) = 0` and all components positive. (c) For fixed `d` and `m`, exactly one value of `v` satisfies the predicate.
- *Depends:* S8a — for the lower bound `m ≥ 2`; T0 (ASN-0034) — for componentwise positivity of the constant tuple.

### Valid insertion position examples

**Non-empty case (binary predicate).** Let subspace S = 1 and suppose V₁(d) = {[1, 1], [1, 2], [1, 3]}, so N = 3 and min(V₁(d)) = [1, 1]. The depth `m = 2` is read from state via S8-depth. The values of `v` satisfying `ValidInsertionPosition(d, v)` are:

- j = 0: v = min(V₁(d)) = [1, 1]
- j = 1: v = shift([1, 1], 1) = [1, 2]
- j = 2: v = shift([1, 1], 2) = [1, 3]
- j = 3: v = shift([1, 1], 3) = [1, 4]

That gives N + 1 = 4 positions. Any successor state whose `V₁(d)` gains a position at, say, [1, 2] must still satisfy D-CTG and D-MIN.

**Empty case (ternary predicate).** V₁(d) = ∅. Choosing depth m = 2, the unique `v` satisfying `ValidFirstInsertionPosition(d, v, 2)` is `[1, 1]`. D-MIN requires min(V₁(d)) = [1, 1] once the subspace becomes non-empty, so the position is exactly the one D-MIN demands. Choosing m = 3 instead, `ValidFirstInsertionPosition(d, v, 3)` is satisfied uniquely by `v = [1, 1, 1]`; by T3, this is a different tumbler.


## Worked example

We instantiate the state model with specific tumblers to ground the abstractions. Consider two documents: document `d₁` at tumbler `1.0.1.0.1` and document `d₂` at tumbler `1.0.1.0.2`. The user creates `d₁` with the text "hello" (five characters), then creates `d₂` which transcludes three characters ("llo") from `d₁` and appends two new characters ("ws"). At each state we exhibit the correspondence-run partition S8 proves and check the design constraints (S0, S3, S7, D-SEQ).

**Initial state Σ₀**: empty. `dom(C) = ∅`, `dom(M(d₁)) = dom(M(d₂)) = ∅`.

**After creating d₁ with "hello"** — state Σ₁. Five I-addresses are allocated under `d₁`'s prefix, with element-level tumblers (`zeros = 3`):

| I-address `a` | `C(a)` |
|---|---|
| `1.0.1.0.1.0.1.1` | 'h' |
| `1.0.1.0.1.0.1.2` | 'e' |
| `1.0.1.0.1.0.1.3` | 'l' |
| `1.0.1.0.1.0.1.4` | 'l' |
| `1.0.1.0.1.0.1.5` | 'o' |

The arrangement `M(d₁)` maps V-positions (in subspace 1, text) to these I-addresses:

| V-position `v` | `M(d₁)(v)` |
|---|---|
| `1.1` | `1.0.1.0.1.0.1.1` |
| `1.2` | `1.0.1.0.1.0.1.2` |
| `1.3` | `1.0.1.0.1.0.1.3` |
| `1.4` | `1.0.1.0.1.0.1.4` |
| `1.5` | `1.0.1.0.1.0.1.5` |

*Check S0*: no prior content existed, so the implication holds vacuously. *Check S3*: every V-reference resolves — `ran(M(d₁)) ⊆ dom(C)`. *Check S7*: for `a = 1.0.1.0.1.0.1.3`, `origin(a) = 1.0.1.0.1 = d₁` — the document-level prefix directly identifies the allocating document. *Verify S8 (correspondence-run partition)*: the five V-positions form a single maximal run `(v₀, a₀, 5)` with `v₀ = 1.1 = [1, 1]` (depth `m = 2`) and `a₀ = 1.0.1.0.1.0.1.1`. The lockstep displacement identity `M(d₁)(shift(v₀, k)) = shift(a₀, k)` holds at every `0 ≤ k < 5`; we exercise it at `k ≥ 1`. At `k = 1`: `shift(v₀, 1) = [1, 2] = 1.2` by OrdinalShift (ASN-0034) (component 1 preserved, last component `1 + 1 = 2`), and `shift(a₀, 1) = 1.0.1.0.1.0.1.2 = M(d₁)(1.2)`. At `k = 4`: `shift(v₀, 4) = [1, 5] = 1.5` and `shift(a₀, 4) = 1.0.1.0.1.0.1.5 = M(d₁)(1.5)`. V-side and I-side advance in lockstep — OrdShiftHom keeps each `shift(v₀, k)` a depth-2 text-subspace V-position, and each `shift(a₀, k) = M(d₁)(shift(v₀, k)) ∈ dom(C)` by S3, so S7b makes it an element-level I-address (`zeros = 3`). The run is maximal: `shift(v₀, 5) = 1.6 ∉ dom(M(d₁))`, so it admits no forward extension. The single run covers `V₁(d₁) = {1.1, …, 1.5}`.

*Check D-SEQ*: V₁(d₁) = {[1, k] : 1 ≤ k ≤ 5}, satisfying D-SEQ with n = 5. D-CTG holds (no gaps in the ordinal range 1..5) and D-MIN holds (min = [1, 1]).

**After creating d₂ with transclusion + append** — state Σ₂. The transclusion of "llo" from `d₁` shares the original I-addresses. The append of "ws" allocates two new I-addresses under `d₂`'s prefix:

| I-address `a` | `C(a)` |
|---|---|
| `1.0.1.0.2.0.1.1` | 'w' |
| `1.0.1.0.2.0.1.2` | 's' |

The content store now has 7 entries (5 from `d₁`, 2 new from `d₂`).

The arrangement `M(d₂)`:

| V-position `v` | `M(d₂)(v)` | origin |
|---|---|---|
| `1.1` | `1.0.1.0.1.0.1.3` | `d₁` (transcluded 'l') |
| `1.2` | `1.0.1.0.1.0.1.4` | `d₁` (transcluded 'l') |
| `1.3` | `1.0.1.0.1.0.1.5` | `d₁` (transcluded 'o') |
| `1.4` | `1.0.1.0.2.0.1.1` | `d₂` (native 'w') |
| `1.5` | `1.0.1.0.2.0.1.2` | `d₂` (native 's') |

*Check S0*: all 5 prior entries in `dom(C)` remain with unchanged values. The transition added 2 new entries. *Check S3*: every V-reference in `M(d₂)` resolves — positions `1.1`–`1.3` reference I-addresses from `d₁` (which exist by S1), positions `1.4`–`1.5` reference the newly allocated addresses. *Check S7*: for `a = 1.0.1.0.1.0.1.4` (the second 'l' in `d₂`), `origin(a) = 1.0.1.0.1 = d₁` — attribution traces to the originating document, not to `d₂` where the content currently appears. *Check S5*: the I-address `1.0.1.0.1.0.1.3` now appears in both `ran(M(d₁))` and `ran(M(d₂))` — sharing multiplicity is 2. *Verify S8 (correspondence-run partition)*: `M(d₂)` decomposes into **two** maximal runs, broken at the transclusion/append boundary. Run A is `([1, 1], 1.0.1.0.1.0.1.3, 3)` covering `1.1`–`1.3`: the displacement identity holds at `k = 1`, `M(d₂)(shift([1,1], 1)) = M(d₂)(1.2) = 1.0.1.0.1.0.1.4 = shift(1.0.1.0.1.0.1.3, 1)`, and at `k = 2`, `M(d₂)(1.3) = 1.0.1.0.1.0.1.5 = shift(1.0.1.0.1.0.1.3, 2)`. Run A is forward-maximal because the lockstep breaks at the boundary: `M(d₂)(shift([1,3], 1)) = M(d₂)(1.4) = 1.0.1.0.2.0.1.1`, whereas `shift(1.0.1.0.1.0.1.5, 1) = 1.0.1.0.1.0.1.6` — the transcluded `d₁` content ends and native `d₂` content begins, so the images jump from `…1.5` to `…2.1` and no longer advance in step. Run B is `([1, 4], 1.0.1.0.2.0.1.1, 2)` covering `1.4`–`1.5`, with `M(d₂)(shift([1,4], 1)) = M(d₂)(1.5) = 1.0.1.0.2.0.1.2 = shift(1.0.1.0.2.0.1.1, 1)` at `k = 1`. The two runs partition `dom(M(d₂)) = {1.1, …, 1.5}` (note that contiguous V-positions need not map to contiguous I-addresses — the boundary is exactly where one run ends and the next begins). *Check D-SEQ*: V₁(d₁) is unchanged — {[1, k] : 1 ≤ k ≤ 5}, D-SEQ with n = 5. V₁(d₂) = {[1, k] : 1 ≤ k ≤ 5}, D-SEQ with n = 5. Both satisfy D-CTG and D-MIN.

**After deleting "llo" from d₁** — state Σ₃. DELETE removes V-positions `1.3`–`1.5` from `M(d₁)`:

| V-position `v` | `M(d₁)(v)` |
|---|---|
| `1.1` | `1.0.1.0.1.0.1.1` |
| `1.2` | `1.0.1.0.1.0.1.2` |

*Check S0*: all 7 entries in `dom(C)` remain. The I-addresses `1.0.1.0.1.0.1.3`–`.5` are no longer in `ran(M(d₁))` but persist in `dom(C)`; these three addresses are now "orphaned" from `d₁`'s perspective, but still referenced by `M(d₂)` — persistence is unconditional (S0). *Check two-stream separation (S0 frame)*: the deletion modified `M(d₁)` but `C` is unchanged — separation holds. *Verify S8 (correspondence-run partition)*: the now-two V-positions `1.1` and `1.2` form a single maximal run `([1, 1], 1.0.1.0.1.0.1.1, 2)` — `M(d₁)(shift([1,1], 1)) = M(d₁)(1.2) = 1.0.1.0.1.0.1.2 = shift(1.0.1.0.1.0.1.1, 1)` at `k = 1` — partitioning the two-element `dom(M(d₁))`; `M(d₂)` is unchanged. *Check D-SEQ*: V₁(d₁) = {[1, k] : 1 ≤ k ≤ 2}, D-SEQ with n = 2. D-CTG holds (no gaps in 1..2) and D-MIN holds (min = [1, 1]). V₁(d₂) is unchanged — D-SEQ with n = 5.

The lifecycle above exercises the contiguity constraints at depth 2 on every well-formed state (Σ₁–Σ₃: D-CTG, D-MIN, D-SEQ all hold).

**Contiguity violation (depth 2).** Consider the candidate `V₁(d) = {[1,1], [1,3]}`. Now `[1,2]` is an intermediate between `[1,1]` and `[1,3]` that is absent — D-CTG is violated. A state with a gap in the ordinal range between occupied extremes is not a well-formed document arrangement.

**Higher depth (depth 3).** Let document `d'` have `M(d') = {[1,1,1] ↦ a₁, [1,1,2] ↦ a₂, [1,1,3] ↦ a₃}`, so `V₁(d') = {[1,1,1], [1,1,2], [1,1,3]}`. *D-CTG check*: the only intermediate at subspace 1 and depth 3 between the extremes `[1,1,1]` and `[1,1,3]` is `[1,1,2]`, which is present. ✓ *D-MIN check*: `min(V₁(d')) = [1,1,1] = [S, 1, 1]`, all post-subspace components equal to 1. ✓

**Contiguity violation (depth ≥ 3).** Suppose instead `V₁(d') = {[1,1,1], [1,2,1]}`. D-CTG requires every intermediate with subspace 1 and depth 3 between `[1,1,1]` and `[1,2,1]` to be present. But `[1,1,2], [1,1,3], [1,1,4], …` are all intermediates — infinitely many, contradicting S8-fin. This is D-CTG-depth in action: positions differing before the last component cannot coexist in a finite arrangement.

## Properties Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| Σ.C | Content store: `T ⇀ Val`, mapping I-addresses to content values | introduced |
| Σ.M(d) | Arrangement for document `d`: `T ⇀ T`, mapping V-positions to I-addresses | introduced |
| S0 | Content immutability: `a ∈ dom(C) ⟹ a ∈ dom(C') ∧ C'(a) = C(a)` for all transitions | design requirement |
| S1 | Store monotonicity: `dom(C) ⊆ dom(C')` for all transitions | from S0 |
| S2 | Arrangement functionality: `M(d)` is a function — each V-position maps to exactly one I-address | axiom |
| S3 | Referential integrity: `(A d, v : v ∈ dom(M(d)) : M(d)(v) ∈ dom(C))` | design; uses S1 |
| S4 | Origin-based identity: distinct allocations produce distinct I-addresses regardless of value equality | from GlobalUniqueness, T3 (ASN-0034) |
| S5 | Unrestricted sharing: S0–S3 do not entail any finite bound on sharing multiplicity | consistent with S0, S1, S2, S3 |
| S7a | Document-scoped allocation: every I-address is allocated under the originating document's prefix | design; uses T4, T4b, T10a, T10a.4 (ASN-0034), S7b |
| S7b | Element-level I-addresses: `(A a ∈ dom(C) :: zeros(a) = 3)` | design; uses T4, T4b, T10a.4 (ASN-0034) |
| S7d | Document allocation discipline: every document is addressed by a document-level tumbler (`zeros = 2`) arising from an allocation event under T10a; distinct documents arise from distinct allocation events | design; uses T10a, T10a.4, T4 (ASN-0034) |
| S7 | Structural attribution: `origin(a) = N(a).0.U(a).0.D(a)` — full document prefix | from S7a, S7b, S7d, S0, S4, T4, T4a, T4b, T0, T3, T10a.4, GlobalUniqueness (ASN-0034) |
| S8-fin | Finite arrangement: `dom(M(d))` is finite for every document `d` | design requirement |
| Σ.M(d) domain restriction (S8a) | `dom(Σ.M(d)) ⊆ {t ∈ T : zeros(t) = 0 ∧ #t ≥ 2}` — arrangements map only V-positions; per-component form stated at the S8a definition site | axiom (definitional); T0 (ASN-0034) |
| subspace(v) | V-position subspace identifier: `subspace(v) = v₁`; well-defined when `#v ≥ 1` | introduced; uses T0 (ASN-0034) |
| S8-depth | Fixed-depth V-positions: `(A d, u, w : u ∈ dom(M(d)) ∧ w ∈ dom(M(d)) ∧ subspace(u) = subspace(w) : #u = #w)` | design; uses S8a |
| OrdShiftHom | Ordinal shift preservation: (a) `subspace(shift(v, n)) = subspace(v)`; (b) `shift(v, n)` preserves S8a — both from TumblerAdd on `δ(n, m)` | lemma; uses OrdinalShift, OrdinalDisplacement, TumblerAdd, TA0 (ASN-0034), S8a |
| S8 | Correspondence-run partition: `dom(M(d))` is the disjoint union of finitely many maximal runs `(vⱼ, aⱼ, nⱼ)` with `M(d)(shift(vⱼ, k)) = shift(aⱼ, k)` for `0 ≤ k < nⱼ`; maximal decomposition is unique | theorem from S2, S3, S8-fin, S8a, S8-depth, OrdShiftHom, T1, TS2, TS3, TS4, OrdinalShift, OrdinalDisplacement (ASN-0034) |
| D-CTG | V-position contiguity: V_1(d) forms a contiguous ordinal range with no gaps — design constraint on well-formed document states | design; uses S8a, S8-depth, T1 (ASN-0034) |
| D-MIN | V-position minimum: non-empty V_1(d) has minimum [1, 1, ..., 1] with every component equal to 1 — design constraint | design requirement |
| D-CTG-depth | Shared prefix reduction: at depth m ≥ 3, all positions in V_1(d) share components 2 through m − 1, so contiguity reduces to the last component | corollary of D-CTG, S8a, S8-fin, S8-depth, T0(a), T1, T3 (ASN-0034) |
| D-SEQ | Sequential positions: non-empty V_1(d) = {[1, 1, ..., 1, k] : 1 ≤ k ≤ n} for some n ≥ 1 | from D-CTG, D-CTG-depth, D-MIN, S8a, S8-fin, S8-depth, T1 (ASN-0034) |
| ValidInsertionPosition | Binary predicate `ValidInsertionPosition(d, v)` (non-empty case): when V_1(d) ≠ ∅, m is the common depth of V_1(d) (state-determined via S8-depth), and v = min(V_1(d)) or v = shift(min(V_1(d)), j) for j ∈ {1, ..., N} where N = |V_1(d)| | introduced |
| ValidFirstInsertionPosition | Ternary predicate `ValidFirstInsertionPosition(d, v, m)` (empty case): when V_1(d) = ∅, m ≥ 2, and v = [1, 1, ..., 1] of depth m | introduced |


## Open Questions

What constraints must the content store's value domain `Val` satisfy — must all entries be uniform in type, or must `Val` support heterogeneous content (text, links, media) as first-class distinctions?

What must the system guarantee about the computability of the sharing inverse — given an I-address, what is the cost bound for determining which documents currently reference it?

Under what conditions, if any, may the referential integrity invariant S3 be temporarily violated — must it hold at every observable state, or only at quiescent states between operations?

What abstract property distinguishes content that exists but is unreachable from all current arrangements from content that exists and is reachable — and must the system maintain this distinction as queryable state?

What must each well-formed editing operation (DELETE, INSERT, COPY, REARRANGE) — and the displacement mechanism underlying insertion at a ValidInsertionPosition — guarantee in order to preserve the contiguity invariants D-CTG, D-MIN, and S2, including the case where insertion coincides with an occupied V-position?

The strand model fixes only the lower bound m ≥ 2 for V-position depth in an empty subspace; the specific value is a one-time allocation convention chosen by the first-placing operation, not a strand-level commitment. What operation-layer constraints determine the canonical choice of m (e.g., m = 2 for basic INSERT/DELETE versus deeper subdivisions Nelson contemplated)? What downstream capabilities — nested hierarchies, link subdivision, future extensibility — does each depth choice unlock or foreclose?

The strand model treats subspace alignment — a V-position's subspace identifier `subspace(v) = v₁` matching the first element-field component of the I-address `M(d)(v)` it maps to — as an operations-layer preservation obligation rather than a state-level invariant on arrangements. Which editing operations must establish this alignment for the V-positions they produce, and under what allocation conventions is preservation automatic versus requiring explicit operation-level enforcement?
