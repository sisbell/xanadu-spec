> **ASN-0124 · The FINDDOCSCONTAINING Operation** — condensed claim statements  
> [← Full note](ASN-0124-finddocscontaining-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0124 Claim Statements

*Source: ASN-0124-finddocscontaining-operation.md (revised 2026-06-12) — Extracted: 2026-06-13*

## Definition — ContentImage

**FD-IMGC (ContentImage)**

For `d ∈ dom(Σ.M)` and `W ⊆ T`:

> `image_C(W, d, Σ) ≡ {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d)) ∧ subspace(v) = s_C}`,

undefined for `d ∉ dom(Σ.M)`; and the restriction is exactly intersection with the content store:

> `image_C(W, d, Σ) = image(W, d, Σ) ∩ dom(Σ.C)`.

---

## Definition — ContentRange

**FD-RAN (ContentRange)**

For `d ∈ dom(Σ.M)`:

> `ran_C(d, Σ) ≡ image_C(T, d, Σ) = {Σ.M(d)(v) : v ∈ V_{s_C}(d)}`,

and the alignment with the foundation's containment relation is definitional:

> `a ∈ ran_C(d, Σ) ⟺ (a, d) ∈ Contains_C(Σ)`.

`ran_C(d, Σ)` is finite (`dom(Σ.M(d))` is finite, S8-fin) and `ran_C(d, Σ) ⊆ dom(Σ.C)` (FD-IMGC at `W = T`).

---

## Definition — VSpecSet

**FD-Q (VSpecSet)**

A vspec-set at Σ is a finite set `Q = {(d₁, W₁), …, (d_p, W_p)}` of pairs with each `d_j ∈ dom(Σ.M)` and each `W_j ⊆ T` a V-region. The pairs may name the same document more than once, and may name different documents.

---

## Definition — Resolution

**FD-RES (Resolution)**

The resolution of a vspec-set is the union of its content images:

> `resolve(Q, Σ) ≡ (∪ (d, W) : (d, W) ∈ Q : image_C(W, d, Σ))`.

Postconditions:
- (a) groundedness — `resolve(Q, Σ) ⊆ dom(Σ.C)`;
- (b) finiteness — a finite union of finite sets;
- (c) flattening — the pair structure of `Q` is discarded: the value is a bare I-address set.

---

## FD-ASKER — AskerIndependence (LEMMA, lemma)

If `Σ.M(d_t)(v) = a = Σ.M(d_o)(u)` with `subspace(v) = subspace(u) = s_C`, then

> `resolve({(d_t, {v})}, Σ) = {a} = resolve({(d_o, {u})}, Σ)`.

The asker's starting document is consumed entirely at resolution; the search itself never sees it.

---

## Definition — ContainmentComprehension

**FD-FIND (ContainmentComprehension)**

For any `I ⊆ T` and state Σ:

> `finddocs(I, Σ) ≡ {d ∈ dom(Σ.M) : ran_C(d, Σ) ∩ I ≠ ∅}`.

Degenerate cases: `finddocs(∅, Σ) = ∅`; a freshly registered document (`dom(Σ.M(d)) = ∅`, the K.δ Document post-state) has `ran_C(d, Σ) = ∅` and is never a member.

---

## Definition — TheOperation

**FD-V (TheOperation)**

FINDDOCSCONTAINING is the two-phase composite:

> `finddocs_V(Q, Σ) ≡ finddocs(resolve(Q, Σ), Σ)`,

defined whenever every document named in `Q` is registered. The codomain is `𝒫(E_doc)`, each member a T4-valid document tumbler (`zeros = 2`, M0). Equal resolutions give equal answers:

> `resolve(Q₁, Σ) = resolve(Q₂, Σ) ⟹ finddocs_V(Q₁, Σ) = finddocs_V(Q₂, Σ)`.

---

## FD-COMPLETE — DocuverseCompleteness (INV, predicate)

> `(A d : d ∈ dom(Σ.M) ∧ ran_C(d, Σ) ∩ I ≠ ∅ : d ∈ finddocs(I, Σ))`.

The quantifier ranges over the entire document stratum `dom(Σ.M) = E_doc` at Σ. The signature `finddocs(I, Σ)` admits no locality, authorship, or asker parameter, so no sub-docuverse restriction is even expressible.

---

## FD-SOUND — PresentWitnessSoundness (INV, predicate)

> `(A d : d ∈ finddocs(I, Σ) : (E v, a :: v ∈ dom(Σ.M(d)) ∧ subspace(v) = s_C ∧ Σ.M(d)(v) = a ∧ a ∈ I))`.

Every member carries a present witness pair `(v, a)`: a live position currently mapped onto queried material.

---

## FD-GROUND — GhostAddressInertness (LEMMA, lemma)

> `finddocs(I, Σ) = finddocs(I ∩ dom(Σ.C), Σ)`.

Derivation: `ran_C(d, Σ) ⊆ dom(Σ.C)` (FD-RAN), so `ran_C(d, Σ) ∩ I = ran_C(d, Σ) ∩ (I ∩ dom(Σ.C))` for every `d`.

---

## FD-PART — AnyPortionSufficiency (LEMMA, lemma)

For `a ∈ I` and `d ∈ dom(Σ.M)`:

> `a ∈ ran_C(d, Σ) ⟹ d ∈ finddocs(I, Σ)`.

And membership never demands coverage — no clause of FD-FIND requires `I ⊆ ran_C(d, Σ)`, so a document arranging exactly one address of a thousand-address query is as much a member as one arranging them all.

---

## FD-UDIST — UnionDistributivity (LEMMA, lemma)

For all `I₁, I₂ ⊆ T` — no disjointness required:

> `finddocs(I₁ ∪ I₂, Σ) = finddocs(I₁, Σ) ∪ finddocs(I₂, Σ)`.

With FD-RES this gives the per-region decomposition of the operation itself:

> `finddocs_V(Q, Σ) = (∪ (d, W) ∈ Q : finddocs(image_C(W, d, Σ), Σ))`.

---

## FD-IMONO — MonotonicityInMaterial (LEMMA, lemma)

> `I' ⊆ I ⟹ finddocs(I', Σ) ⊆ finddocs(I, Σ)`,

by FD-UDIST at `I = I' ∪ (I ∖ I')`.

---

## FD-LOCAL — PerDocumentLocality (LEMMA, lemma)

Write `χ(d, I, Σ) ≡ ran_C(d, Σ) ∩ I ≠ ∅` for the membership criterion. χ is a function of `I` and `Σ.M(d)` alone. Two corollaries:

- (i) *cross-document independence* — any transition with `Σ'.M(d) = Σ.M(d)` leaves `d`'s membership unchanged, in both directions;
- (ii) *non-impedance* — enlarging the docuverse (new documents, new content, new links, new provenance) can never remove `d`.

---

## FD-SELF — SelfInclusion (LEMMA, lemma)

Every naming document with a non-trivial region is a member of its own query's answer: for `(d, W) ∈ Q` with `image_C(W, d, Σ) ≠ ∅`, `d ∈ finddocs_V(Q, Σ)`. For the single-region query the statement is a biconditional:

> `d ∈ finddocs_V({(d, W)}, Σ) ⟺ image_C(W, d, Σ) ≠ ∅`,

and

> `image_C(W, d, Σ) = ∅ ⟹ finddocs_V({(d, W)}, Σ) = ∅`.

---

## FD-NEUT — OriginNeutrality (LEMMA, lemma)

- (a) *frame observation* — χ is a function of `I` and `Σ.M(d)` alone (FD-LOCAL), so it cannot see nativeness.
- (b) For `a ∈ I`: `origin(a) ∈ finddocs(I, Σ)` iff `origin(a) ∈ dom(Σ.M)` and `ran_C(origin(a), Σ) ∩ I ≠ ∅`.
- (c) There are reachable states in which `origin(a) ∉ finddocs({a}, Σ)` while transcluders of `a` are members. [Construction given in body at FD-NEUT.]

---

## FD-IDENT — AddressIdentityKeying (LEMMA, lemma)

- (a) *Value-blindness.* `finddocs` is a function of `(I, Σ.M)` (FD-LOCAL aggregated over `d`): two states agreeing on `M` give identical answers for every `I`, whatever their content stores hold.
- (b) *Coincidence exclusion.* If `a₁ ≠ a₂` with `Σ.C(a₁) = Σ.C(a₂)`, then a document arranging only `a₂` satisfies `ran_C(d, Σ) ∩ {a₁} = ∅` and is excluded from `finddocs({a₁}, Σ)`.
- (c) *Provenance kinship is not sufficient.* For `a' = inc(a, 0)` a sibling emission on the same content chain (`origin(a') = origin(a)`, `a' ≠ a`), a document arranging only `a'` is excluded from `finddocs({a}, Σ)`.

---

## FD-CHAIN — FlatChainReach (LEMMA, lemma)

Fix `a ∈ dom(Σ.C)`.

- (a) *Propagation.* A transclusion composite from `d_i` to `d_{i+1}` whose copied region's content image contains `a` yields `a ∈ ran_C(d_{i+1}, ·)` at its boundary; the address arriving in `d_{i+1}` is the same `a` that arrived in `d_i`.
- (b) *Flat evaluation.* At any state Σ, `finddocs({a}, Σ) = {d ∈ dom(Σ.M) : a ∈ ran_C(d, Σ)}` collects the entire current sharing set of `a` in one comprehension; the criterion mentions no path, no copy event, no `Σ.R`, no other document (FD-LOCAL); a chain `d₀ → d₁ → ⋯ → d_n` of transclusion composites, each propagating `a`, leaves all of `d₀, …, d_n` simultaneous members, found without any iterative chain-following.
- (c) *Severance immunity.* If `d_mid` later contracts `a` away, every other document's membership is untouched (FD-LOCAL(i)), so the ends remain co-listed without the middle.

---

## FD-VERS — ForkMembershipDuplication (LEMMA, lemma)

Let `Σ →* Σ'` be a J4 fork composite with content source operand `d_op` and fresh version `d_new`. Then for every `I`:

> `finddocs(I, Σ') = finddocs(I, Σ) ∪ ({d_new} if d_op ∈ finddocs(I, Σ) else ∅)`,

and in particular:

> `d_new ∈ finddocs(I, Σ') ⟺ d_op ∈ finddocs(I, Σ')`.

---

## FD-COOC — CooccurrenceByComposition (LEMMA, lemma)

For fragments `I₁, …, I_k`, define `inc(d) = {j : ran_C(d, Σ) ∩ I_j ≠ ∅}`. Then:

> `finddocs(∪_j I_j, Σ) = {d : inc(d) ≠ ∅}`   (FD-UDIST)

> `∩_j finddocs(I_j, Σ) = {d : inc(d) = {1, …, k}}`   (recombiners)

> `{d : I ⊆ ran_C(d, Σ)} = ∩_{a ∈ I} finddocs({a}, Σ)`   for `I ≠ ∅`

(At `I = ∅`, the empty intersection read within the universe `dom(Σ.M)` is `dom(Σ.M)`, and the identity extends to `I = ∅` as well.)

---

## FD-LOSSY — MergedResultUnderdetermination (LEMMA, lemma)

There are reachable states `Σ¹, Σ²` and fragments `I₁, I₂` with

> `finddocs(I₁ ∪ I₂, Σ¹) = finddocs(I₁ ∪ I₂, Σ²)`

but different incidences (i.e., `inc(d)` differs between the two states for the single candidate document). [Construction given in body.]

---

## FD-CONVEX — SingleSpanConvexityForcing (LEMMA, lemma)

Let `σ` be a V-span over `d`'s content positions (T12) with `u, q ∈ ⟦σ⟧ ∩ V_{s_C}(d)`, `u < q`. Then every intervening content position is dragged in: for `v ∈ V_{s_C}(d)` with `u < v < q` — span denotations are order-convex (T12(c)), so `v ∈ ⟦σ⟧`, whence `Σ.M(d)(v) ∈ image_C(⟦σ⟧, d, Σ) ⊆ resolve`. The two-region vspec-set `{(d, W₁), (d, W₂)}` with `u ∈ W₁`, `q ∈ W₂`, `v ∉ W₁ ∪ W₂` resolves to `image_C(W₁, d, Σ) ∪ image_C(W₂, d, Σ)` and excludes connective-only documents whenever the connective image is disjoint from the fragment images (guaranteed when `Σ.M(d)` is injective on the span).

---

## FD-FRAME — NonArrangementInertness (LEMMA, lemma)

Every transition that fixes the content-subspace arrangement family fixes the answer: for every `I`, K.α, K.λ, K.ρ (arrangement frames `M' = M`), K.δ (Node/Account cases frame `M`; the Document case adds `d_new` with `M'(d_new) = ∅`, never a member, others framed), and K.μ⁺_L (adds only `s_L`-positions to one document, so `V_{s_C}(d)` and its images are unchanged) all satisfy:

> `finddocs(I, Σ') = finddocs(I, Σ)`.

---

## FD-STEP — ArrangementStepCharacterization (LEMMA, lemma)

The only movers are the content-subspace arrangement transitions, and each moves the answer in exactly one place:

- K.μ⁺ on `d` (content extension, new images `N = {Σ'.M(d)(v) : v ∈ dom(Σ'.M(d)) ∖ dom(Σ.M(d))}`): `ran_C(d, Σ') = ran_C(d, Σ) ∪ N`, all other documents framed, so

  > `finddocs(I, Σ') = finddocs(I, Σ) ∪ ({d} if N ∩ I ≠ ∅ else ∅)`.

- K.μ⁻ on `d` (contraction with retention set `Ret`): writing `ran_Ret ≡ {Σ.M(d)(v) : v ∈ Ret ∧ subspace(v) = s_C}`, the retained-domain agreement gives `ran_C(d, Σ') = ran_Ret ⊆ ran_C(d, Σ)`, others framed, so

  > `finddocs(I, Σ') = (finddocs(I, Σ) ∖ {d}) ∪ ({d} if ran_Ret ∩ I ≠ ∅ else ∅)`.

- K.μ~ on `d` (reorder with witnessing bijection π): the bijection equation `Σ'.M(d)(π(v)) = Σ.M(d)(v)` makes `ran_C(d, Σ') = ran_C(d, Σ)`, so

  > `finddocs(I, Σ') = finddocs(I, Σ)`.

---

## FD-CWP — ContractionSurvivalWP (LEMMA, lemma)

Fix a K.μ⁻ on `d` with retention set `Ret`. The weakest precondition on the pre-state under which the edited document survives in the answer is:

> `wp(K.μ⁻[d, Ret], d ∈ finddocs(I, ·)) ≡ enabled(K.μ⁻[d, Ret]) ∧ (E v : v ∈ Ret ∧ subspace(v) = s_C : Σ.M(d)(v) ∈ I)`.

The whole answer is preserved iff survival is owed only where it was held:

> `finddocs(I, Σ') = finddocs(I, Σ) ⟺ (d ∈ finddocs(I, Σ) ⟹ ran_Ret ∩ I ≠ ∅)`.

Boundary case `Ret = ∅` (full clearance): the existential is false, so the document drops iff it was a member.

---

## FD-FRESH — InsertionInvariance (LEMMA, lemma)

The in-vocabulary insertion composite on `d` at position `p` of an `N`-position content segment with `n ≥ 1` fresh units — iterated K.α allocating `A_new = {a'₁, …, a'ₙ}`; full content clear K.μ⁻ on `d` at `n'_{s_C} = 0`; one rebuild K.μ⁺ re-populating the canonical segment of length `N + n`; K.ρ recording `(a', d)` for each `a' ∈ A_new` — is a valid composite, and for every `I` fixed at the pre-state with `I ⊆ dom(Σ.C)`:

> `finddocs(I, Σ_post) = finddocs(I, Σ_pre)`.

The net initial-to-final effect realizes ASN-0082's gap-shift contract (I3/I3-L/I3-V/I3-CS). The pure append (`p = N + 1`) needs no clear at all — a bare K.μ⁺ extending the segment with images in `A_new` — with the same conclusion.

---

## FD-NONMONO — LiveNonMonotonicity (LEMMA, lemma)

Across `Σ →* Σ'` neither inclusion holds in general: the transclusion step grows the answer (FD-STEP, K.μ⁺ with `N ∩ I ≠ ∅`), and the contraction step shrinks it (FD-CWP's failing branch). For the two-phase operation there is one further motion: the resolution itself is present-tense — editing a named document moves `resolve(Q, ·)` even while every containment fact is fixed (D-PRES, ASN-0127).

---

## FD-VDYN — TwoPhasePerTransitionDynamics (LEMMA, lemma)

Fix a vspec-set `Q` with every named document registered at Σ. Write `I = resolve(Q, Σ)`, `I' = resolve(Q, Σ')`. Four cases exhaust the vocabulary:

- (a) No named content motion (K.α, K.λ, K.ρ, K.δ anywhere; K.μ⁺_L anywhere; K.μ⁺, K.μ⁻, K.μ~ on unnamed documents): `I' = I` and `finddocs_V(Q, Σ') = finddocs(I, Σ')`, governed by FD-FRAME and FD-STEP.

- (b) Extension of a named document — K.μ⁺ on named `d_q`: monotone growth,

  > `finddocs_V(Q, Σ) ⊆ finddocs_V(Q, Σ')`.

- (c) Contraction of a named document — K.μ⁻ on named `d_q`: monotone shrinkage,

  > `finddocs_V(Q, Σ') ⊆ finddocs_V(Q, Σ)`.

- (d) Reorder of a named document — K.μ~ on named `d_q` with witnessing bijection π:

  > `finddocs_V(Q, Σ') = finddocs(I', Σ') = finddocs(I', Σ)`,

  with

  > `image_C(W, d_q, Σ') = image_C(π⁻¹(W), d_q, Σ)`.

  Stability condition: if π fixes every named region setwise on the content positions — `π⁻¹(W) ∩ V_{s_C}(d_q) = W ∩ V_{s_C}(d_q)` for each `(d_q, W) ∈ Q` — then `I' = I` and the answer is unchanged. Image motion is necessary for answer motion but not sufficient.

---

## Definition — ProvenanceQuery

**FD-HIST (ProvenanceQuery)**

> `finddocs_R(I, Σ) ≡ {d ∈ dom(Σ.M) : (E a : a ∈ I : (a, d) ∈ Σ.R)}`.

---

## FD-RMONO — HistoricalMonotonicity (LEMMA, lemma)

> `Σ →* Σ' ⟹ finddocs_R(I, Σ) ⊆ finddocs_R(I, Σ')`.

Derivation: `R ⊆ R'` per transition (P2), `dom(M) ⊆ dom(M')` (M1), both lifted over the finite decomposition of `→*`; the criterion reads only these monotone components.

---

## FD-SUPER — LiveBoundedByHistorical (LEMMA, lemma)

At every composite boundary Σ:

> `finddocs(I, Σ) ⊆ finddocs_R(I, Σ)`.

Derivation: a member's present witness (FD-SOUND) is a pair `(a, d) ∈ Contains_C(Σ)` with `a ∈ I` (FD-RAN alignment), and P4★ places it in `Σ.R`.

---

## FD-WITNESS — EverContainedEqualsOnceLive (LEMMA, lemma)

For every valid trace `Σ₀ →* Σ₁ →* ⋯ →* Σ_n = Σ` (each `Σ_k` a composite boundary):

> `finddocs_R(I, Σ) = (∪ k : 0 ≤ k ≤ n : finddocs(I, Σ_k))`.

---

## Definition — GhostCharacterization

**FD-GHOST (GhostCharacterization)**

> `ghosts(I, Σ) ≡ finddocs_R(I, Σ) ∖ finddocs(I, Σ)`.

By FD-WITNESS (the `k = n` term contributing nothing to the difference):

> `ghosts(I, Σ) = (∪ k : 0 ≤ k < n : finddocs(I, Σ_k)) ∖ finddocs(I, Σ)`.

These are exactly the documents that contained queried material at some past boundary and contain none of it now.

---

## FD-COINC — CoincidenceOnNonShrinkingHistories (LEMMA, lemma)

Call a valid trace *range-non-decreasing* when:

> `(A k, d : 0 ≤ k < n ∧ d ∈ dom(Σ_k.M) : ran_C(d, Σ_k) ⊆ ran_C(d, Σ_{k+1}))`.

Along such a trace the two queries coincide at the endpoint:

> `finddocs_R(I, Σ) = finddocs(I, Σ)`.

Ghosts are therefore exactly the residue of contraction: divergence between the index and the truth begins with the first deletion, and not before.
