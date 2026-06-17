> **ASN-0124 · The FINDDOCSCONTAINING Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0082 · Strand Projection Displacement](../foundation/ASN-0082-strand-projection-displacement.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md), [ASN-0127 · Content-Region Link Query](../foundation/ASN-0127-content-region-link-query.md)  
> [Condensed statements →](ASN-0124-finddocscontaining-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0124: The FINDDOCSCONTAINING Operation

*2026-06-12*

## The Problem

A reader holds some material — a passage, a scatter of fragments — and asks the system a question that runs opposite to ordinary reading: not "what does this document include?" but "who, anywhere, includes this?" Nelson specifies the answer twice, and both sentences are short enough to be load-bearing in full: "This returns a list of all documents containing any portion of the material included by `<vspec set>`" [LM 4/70], and "This returns a list of all documents containing any of the material specified by the span addresses, regardless of where the native copies are located" [LM 4/63]. The operation is the reverse half of the windowing promise — "it must also be possible for the reader to ask to see whatever documents window to the current document. Both are available at any time" [LM 2/36].

Before this is a specification rather than a slogan, five things must be settled. *What is returned* — bare document identities, or identities paired with what each contains? *What relationship* must a returned document bear to the queried material — full containment, partial overlap, or some provenance kinship? *Must the source document itself appear*, and does the home of the native copies enjoy any privilege? *What does naming several non-contiguous regions reveal* that naming one span cannot? And *what invariants* hold the answer together — completeness over the whole docuverse, soundness against material a document once held but holds no longer, stability under edits anywhere, and reach through chains of transclusion?

We will find that every one of these questions is answered by a single design decision, made before this operation was ever named: **containment is a present-tense, address-keyed relation between a document's current arrangement and the queried material.** Material is a set of permanent I-addresses; a document contains some of it exactly when the document's arrangement currently maps some position onto one of those addresses; the operation is the comprehension of that predicate over the entire document stratum.

This note is the document-containment sibling of ASN-0127's content-region link query. The two share the two-phase shape — a named V-region resolves through a live arrangement to an I-address set, and a comprehension is taken against that set — but the comprehensions run over different stores with inverted stability profiles: ASN-0127's ranges over links, whose endset coverages are immutable store values, so its fixed-`I` query only ever grows; ours ranges over documents keyed by their *arrangement ranges*, which are mutable state, so the live answer breathes in and out. That inversion is the heart of the soundness story, and we do not rebuild ASN-0127's machinery — we cite its image primitive and its resolution-drift results where they apply.

**Scope.** We specify the containment query alone. Link discovery and endset search, version comparison, origin reporting, content delivery, the editing operations themselves, and inter-server replication (BEBE) are out of scope; we touch the editing transitions only as the K-vocabulary events against which stability must be stated. The state is the single unified docuverse state; partial visibility of a distributed realization is a refinement concern noted only in the evidence section.

## State and Local Apparatus

We work in the extended state of the transition-model foundation (ASN-0047):

> `Σ = (C, L, E, M, R)`

with `C : T ⇀ Val` the content store (append-only, immutable values — P0), `L : T ⇀ Link` the link store (immutable — L12), `E` the entity set with document stratum `E_doc = {e ∈ E : Document(e)}` (`zeros = 2`, T4-valid — ASN-0045), `M` the per-document arrangement family with `dom(M) = E_doc`, and `R ⊆ T_elem × E_doc` the provenance relation. Subspace identifiers are fixed at `s_C = 1`, `s_L = 2` (SubspaceConventionAxiom), with `subspace(v) = v₁` (ASN-0036). Every arrangement position routes by subspace: content positions map into `dom(C)`, link positions into `dom(L)` (S3★), every position is one or the other (S3★-aux), and the two stores are disjoint (SD, ASN-0093). A document's content-subspace positions are the canonical gap-free segment `V_{s_C}(d) = {[s_C, 1, …, 1, k] : 1 ≤ k ≤ n}` (D-SEQ★). The atomic transition vocabulary is `{K.α, K.δ, K.λ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` with `K.μ~` the named reordering composite; composites are valid when each step's precondition holds at its intermediate state and the couplings J0, J1★, J1'★ hold initial-to-final (ValidComposite★); the per-boundary properties P4★ (`Contains_C(Σ) ⊆ R`), P4a (trace witnessing), and P7a hold at every composite boundary. ASN-0047's current-containment relation is the object this whole note is about:

> `Contains_C(Σ) = {(a, d) : d ∈ E_doc ∧ (E v : v ∈ dom(M(d)) ∧ subspace(v) = s_C : M(d)(v) = a)}`.

For region resolution we take ASN-0127's Phase-1 primitive as given: `image(W, d, Σ) = {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d))}` for `d ∈ dom(Σ.M)`, `W ⊆ T` (F-IMG), undefined for unregistered `d`, together with its dynamics (F-IMG-MONO, F-IMG-CONTR, F-IMG-SWING) and the present-tense-resolution observation D-PRES. What containment needs is the *content-subspace restriction* of that image — the queried material is material, not link machinery — so we define it and tie it back to the unrestricted image.

**FD-IMGC (ContentImage).** *For `d ∈ dom(Σ.M)` and `W ⊆ T`:*

> `image_C(W, d, Σ) ≡ {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d)) ∧ subspace(v) = s_C}`,

*undefined for `d ∉ dom(Σ.M)`; and the restriction is exactly intersection with the content store:*

> `image_C(W, d, Σ) = image(W, d, Σ) ∩ dom(Σ.C)`.

*Derivation of the equality. (⊆) Every contributing position has `subspace(v) = s_C`, so its image lies in `dom(Σ.C)` by S3★. (⊇) Take `b ∈ image(W, d, Σ) ∩ dom(Σ.C)`; then `b = Σ.M(d)(v)` for some `v ∈ W ∩ dom(Σ.M(d))`, and `subspace(v) ∈ {s_C, s_L}` by S3★-aux. Were `subspace(v) = s_L`, S3★ would put `b ∈ dom(Σ.L)`, contradicting `b ∈ dom(Σ.C)` by store disjointness (SD). So `subspace(v) = s_C` and `b ∈ image_C(W, d, Σ)`.*

A region that sweeps link-subspace positions therefore resolves to content only: queries name *material*, and the link entries a document arranges are invisible to them. Taking the whole position space as the region gives the document's current material in total:

**FD-RAN (ContentRange).** *For `d ∈ dom(Σ.M)`:*

> `ran_C(d, Σ) ≡ image_C(T, d, Σ) = {Σ.M(d)(v) : v ∈ V_{s_C}(d)}`,

*and the alignment with the foundation's containment relation is definitional: `a ∈ ran_C(d, Σ) ⟺ (a, d) ∈ Contains_C(Σ)` — unfolding both sides yields the same existential over content-subspace positions (with `d ∈ E_doc = dom(Σ.M)` supplied by the comprehension's guard). `ran_C(d, Σ)` is finite (`dom(Σ.M(d))` is finite, S8-fin) and `ran_C(d, Σ) ⊆ dom(Σ.C)` (FD-IMGC at `W = T`).*

## Naming the Material: VSpec-Sets and Resolution

The asker does not hold I-addresses; the asker holds *places* — regions of documents. Nelson's argument to the operation is a vspec-set: a set of document-scoped V-regions. We formalize the regions extensionally, as ASN-0127 does, with span-sets (ASN-0053) as their finite presentation; since everything below consumes a region setwise, two span-set presentations with the same denotation are interchangeable, and the normalization algebra of ASN-0053 (S8, S9) supplies a canonical form without affecting any result.

**FD-Q (VSpecSet).** *A vspec-set at Σ is a finite set `Q = {(d₁, W₁), …, (d_p, W_p)}` of pairs with each `d_j ∈ dom(Σ.M)` and each `W_j ⊆ T` a V-region. The pairs may name the same document more than once, and may name different documents — the queried material may span the docuverse.*

**FD-RES (Resolution).** *The resolution of a vspec-set is the union of its content images:*

> `resolve(Q, Σ) ≡ (∪ (d, W) : (d, W) ∈ Q : image_C(W, d, Σ))`.

*Postconditions: (a) groundedness — `resolve(Q, Σ) ⊆ dom(Σ.C)` (FD-IMGC); the resolution phase cannot inject an unallocated or link-store address into the query, whatever region the asker names. (b) finiteness — a finite union of finite sets (S8-fin). (c) flattening — the pair structure of `Q` is discarded: the value is a bare I-address set, and nothing downstream can recover which named region contributed which address.*

The resolution phase is the only phase that consults the asker's documents at all, which gives the first reach property almost for free:

**FD-ASKER (AskerIndependence).** *Resolution sees through transclusion: naming material through a transcluder and naming it through its origin produce the same resolved set. If `Σ.M(d_t)(v) = a = Σ.M(d_o)(u)` with `subspace(v) = subspace(u) = s_C`, then `resolve({(d_t, {v})}, Σ) = {a} = resolve({(d_o, {u})}, Σ)`. The asker's starting document is consumed entirely at resolution; the search itself never sees it.*

This is the formal content of Nelson's "regardless of where the native copies are located" read from the asker's side: a vspec is a way of *pointing at* I-addresses, and transclusion means the transcluder's arrangement points at the *same* addresses — that is what inclusion by reference is (ASN-0036, S5; "bytes native elsewhere have an ordinal position in the byte stream just as if they were native to the document" [LM 4/11]).

## The Operation

We now have material as a grounded I-address set, and we have the per-document containment fact `a ∈ ran_C(d, Σ)`. The operation is the comprehension of the one over the other.

**FD-FIND (ContainmentComprehension).** *For any `I ⊆ T` and state Σ:*

> `finddocs(I, Σ) ≡ {d ∈ dom(Σ.M) : ran_C(d, Σ) ∩ I ≠ ∅}`.

*Degenerate cases: `finddocs(∅, Σ) = ∅` (no intersection can be non-empty); a freshly registered document (`dom(Σ.M(d)) = ∅`, the K.δ Document post-state) has `ran_C(d, Σ) = ∅` and is never a member.*

**FD-V (TheOperation).** *FINDDOCSCONTAINING is the two-phase composite:*

> `finddocs_V(Q, Σ) ≡ finddocs(resolve(Q, Σ), Σ)`,

*defined whenever every document named in `Q` is registered. By FD-RES(b) and L-fin-style finiteness of the stratum — `dom(Σ.M)` is finite at every reachable state, since `E` grows by one entity per K.δ from the finite `E₀` — the result is a finite set of bare document identities: the codomain is `𝒫(E_doc)`, each member a T4-valid document tumbler (`zeros = 2`, M0). Nothing else is returned: no pairing with the matched material, no positions within the member, no multiplicity. Membership is idempotent — a document matching through a hundred positions and a document matching through one are the same kind of member. Because the composite consults `Q` only through `resolve(Q, Σ)`, `finddocs_V` is a function of the resolved set: equal resolutions give equal answers — `resolve(Q₁, Σ) = resolve(Q₂, Σ) ⟹ finddocs_V(Q₁, Σ) = finddocs_V(Q₂, Σ)`, the asker-independence corollary of FD-ASKER.*

The bare-identity codomain is Nelson's own quantifier — "a list of all *documents*" — and it makes the operation a *membership oracle*: it tells you which documents, never where within them. Recovering the where is a different question, asked per returned document against its own arrangement, and it belongs to the content-region layer, not here.

Two immediate lemmas pin down what the I-argument can and cannot do.

**FD-GROUND (GhostAddressInertness).** *`finddocs(I, Σ) = finddocs(I ∩ dom(Σ.C), Σ)`.*

*Derivation. `ran_C(d, Σ) ⊆ dom(Σ.C)` (FD-RAN), so `ran_C(d, Σ) ∩ I = ran_C(d, Σ) ∩ (I ∩ dom(Σ.C))` for every `d`; the membership predicate is unchanged. Addresses never allocated, and link-store addresses, contribute nothing — a query cannot be poisoned by naming them.*

**FD-PART (AnyPortionSufficiency).** *A single shared address suffices and is all that is ever required: for `a ∈ I` and `d ∈ dom(Σ.M)`, `a ∈ ran_C(d, Σ) ⟹ d ∈ finddocs(I, Σ)`; and membership never demands coverage — no clause of FD-FIND requires `I ⊆ ran_C(d, Σ)`, so a document arranging exactly one address of a thousand-address query is as much a member as one arranging them all.*

*Derivation. The first half is the comprehension read at the witness `a`. For the second half it suffices that the predicate is an intersection-non-emptiness test, which is monotone in evidence: one witness closes it. The relationship each member bears to the query is therefore exactly* shared address of some portion *— weaker than full containment. The conjunctive "contains all of it" question is a derived query, obtained by composition.*

This existential reading is what the operation is *for*. A full-containment rule could never discover the document quoting one sentence of a long work — the common case — and the discovery mechanism behind "find all uses" must err toward inclusion on the strength of a fragment [LM 4/70, 4/63: "any portion", "any of the material"].

The operation's contract, stated as the two named obligations the topic demands — both definitional for the comprehension, both *non-trivial obligations on any refinement that materializes an index*:

**FD-COMPLETE (DocuverseCompleteness).** *`(A d : d ∈ dom(Σ.M) ∧ ran_C(d, Σ) ∩ I ≠ ∅ : d ∈ finddocs(I, Σ))` — no document anywhere whose current arrangement meets the material may be omitted. The quantifier ranges over the entire document stratum `dom(Σ.M) = E_doc` at Σ — every version of every document under every node and account (each version is its own document entity); nodes (`zeros = 0`) and accounts (`zeros = 1`) are themselves entities but not documents, so they lie outside the range and can never be returned (matching FD-V's codomain `𝒫(E_doc)`). The signature `finddocs(I, Σ)` admits no locality, authorship, or asker parameter, so no sub-docuverse restriction is even expressible — completeness is global by construction, relative to the one unified state.*

**FD-SOUND (PresentWitnessSoundness).** *`(A d : d ∈ finddocs(I, Σ) : (E v, a :: v ∈ dom(Σ.M(d)) ∧ subspace(v) = s_C ∧ Σ.M(d)(v) = a ∧ a ∈ I))` — every member carries a present witness pair `(v, a)`: a live position currently mapped onto queried material. No document is admitted on the basis of content it once held and has since contracted away, on value resemblance, or on provenance records.*

## Query Algebra

The comprehension inherits a small, exact algebra from the shape of its predicate.

**FD-UDIST (UnionDistributivity).** *For all `I₁, I₂ ⊆ T` — no disjointness required:*

> `finddocs(I₁ ∪ I₂, Σ) = finddocs(I₁, Σ) ∪ finddocs(I₂, Σ)`.

*Derivation. `ran_C(d, Σ) ∩ (I₁ ∪ I₂) = (ran_C(d, Σ) ∩ I₁) ∪ (ran_C(d, Σ) ∩ I₂)`, and a union is non-empty iff a part is; set-builder over the disjunction splits the comprehension. With FD-RES this gives the per-region decomposition of the operation itself: `finddocs_V(Q, Σ) = (∪ (d, W) ∈ Q : finddocs(image_C(W, d, Σ), Σ))`.*

**FD-IMONO (MonotonicityInMaterial — corollary).** *`I' ⊆ I ⟹ finddocs(I', Σ) ⊆ finddocs(I, Σ)`, by FD-UDIST at `I = I' ∪ (I ∖ I')`.*

**FD-LOCAL (PerDocumentLocality).** *Write `χ(d, I, Σ) ≡ ran_C(d, Σ) ∩ I ≠ ∅` for the membership criterion. χ is a function of `I` and `Σ.M(d)` alone: unfolding FD-RAN, it is built from `dom(Σ.M(d))`, the tumbler projection `subspace(·)`, the values `Σ.M(d)(·)`, and `I` — no other document's arrangement, no `Σ.C` value, no `Σ.L`, no `Σ.R`, no allocation history occurs in it. Two corollaries: (i)* cross-document independence *— any transition with `Σ'.M(d) = Σ.M(d)` leaves `d`'s membership unchanged, in both directions; (ii)* non-impedance *— enlarging the docuverse (new documents, new content, new links, new provenance — all framing `d`'s arrangement, hence inert by (i)) can never remove `d`: the quantity of material not satisfying a request does not impede the answer on material that does. (Membership is added only by extending the edited document's own content arrangement — K.μ⁺, FD-STEP — never by anything happening elsewhere in the docuverse.) Nelson states this principle for link search [LM 4/60]; here it falls out of locality.*

## Self-Inclusion and Origin-Neutrality

Must the document in which the queried material natively originates appear in its own result? The right answer is that the question dissolves: there is no clause anywhere in FD-FIND that could *see* nativeness. We state the positive form first.

**FD-SELF (SelfInclusion).** *Every naming document with a non-trivial region is a member of its own query's answer: for `(d, W) ∈ Q` with `image_C(W, d, Σ) ≠ ∅`, `d ∈ finddocs_V(Q, Σ)`. For the single-region query the statement is a biconditional, and self-membership is equivalent to the answer being non-empty at all:*

> `d ∈ finddocs_V({(d, W)}, Σ) ⟺ image_C(W, d, Σ) ≠ ∅`, *and* `image_C(W, d, Σ) = ∅ ⟹ finddocs_V({(d, W)}, Σ) = ∅`.

*Derivation. Take `a ∈ image_C(W, d, Σ)`, witnessed by position `v`. Then `a ∈ resolve(Q, Σ)` (FD-RES), and the same `v` witnesses `a ∈ ran_C(d, Σ)` since `image_C(W, d, Σ) ⊆ image_C(T, d, Σ) = ran_C(d, Σ)` — sub-regions image into the full range. So `χ(d, resolve(Q, Σ), Σ)` holds and `d` is a member (FD-PART). For the biconditional's other direction: if the image is empty, the singleton query's resolution is empty and `finddocs(∅, Σ) = ∅` (FD-FIND) — nobody is a member, `d` included. A non-degenerate query always lists its own naming document; trivially, then, "the source appears" whenever the source is where the asker pointed.*

**FD-NEUT (OriginNeutrality).** *(a) — frame observation — χ is a function of `I` and `Σ.M(d)` alone (FD-LOCAL), so it cannot see nativeness: the document that allocated a queried address is tested by exactly the same predicate as every other document. (b) Consequently the origin appears precisely when it qualifies: for `a ∈ I`, `origin(a) ∈ finddocs(I, Σ)` iff `origin(a) ∈ dom(Σ.M)` and `ran_C(origin(a), Σ) ∩ I ≠ ∅` — in particular, whenever the origin still arranges `a` itself. (c) And the origin can fail to qualify: there are reachable states in which `origin(a) ∉ finddocs({a}, Σ)` while transcluders of `a` are members.*

*Construction for (c). Take any reachable Σ₀′ with two registered documents `d₁, d₂` (K.δ scaffolding per ASN-0047). Run the valid insertion composite on `d₁` — K.α allocating fresh `a` on `d₁`'s content chain `A_C(d₁)` (so `origin(a) = d₁`, ASN-0093/FirstEmission), K.μ⁺ arranging `v ↦ a`, K.ρ recording `(a, d₁)` (J0, J1★ discharged) — then the transclusion composite into `d₂` — K.μ⁺ arranging `u ↦ a` (precondition `a ∈ dom(C)` holds), K.ρ recording `(a, d₂)` (J1★) — then the contraction K.μ⁻ on `d₁` retaining nothing of its content subspace (`n'_{s_C} = 0`; valid and self-sufficient, J2). At the resulting boundary: `a ∈ dom(C)` still (P0 — the contraction's frame is `C' = C`; the material itself cannot be destroyed), `ran_C(d₁) ∌ a`, `ran_C(d₂) ∋ a`, so `finddocs({a}) = {d₂}`. The owner deleted the bytes from the owner's document, and "those bytes remain in all other documents where they have been included" [LM 4/11] — the origin holds no privilege against the present-tense criterion and suffers no exclusion either.*

## Identity, Not Resemblance

**FD-IDENT (AddressIdentityKeying).** *(a)* Value-blindness. *`finddocs` is a function of `(I, Σ.M)` (FD-LOCAL aggregated over `d`): two states agreeing on `M` give identical answers for every `I`, whatever their content stores hold. (b)* Coincidence exclusion. *Independently created material is unrelated however its values compare: if `a₁ ≠ a₂` with `Σ.C(a₁) = Σ.C(a₂)` — distinct allocation events necessarily yield distinct addresses regardless of value, S4 (ASN-0036) — then a document arranging only `a₂` satisfies `ran_C(d, Σ) ∩ {a₁} = ∅` and is excluded from `finddocs({a₁}, Σ)`. "Wrote the same words" and "holds the same material" are different relations, and the operation computes only the second. (c)* Provenance kinship is not sufficient. *Sharing an origin does not match: for `a' = inc(a, 0)` a sibling emission on the same content chain (`origin(a') = origin(a)`, `a' ≠ a` by ChainEnumerationInjectivity, ASN-0093), a document arranging only `a'` is excluded from `finddocs({a}, Σ)`. Membership is per-address identity — finer than source-document kinship, blind to value.*

## Reach: Transclusion Chains and Version Forks

Transclusion, in this state model, is nothing but a content-subspace arrangement extension whose new images are *existing* addresses — a K.μ⁺ drawing its images from another document's region, with K.ρ discharging J1★. The reach questions — must the search cross document boundaries, must containment carry through chains `A → B → C`, must it do so without traversal — all reduce to one observation: identity propagates at copy time, so by query time there is no chain left to follow.

**FD-CHAIN (FlatChainReach).** *Fix `a ∈ dom(Σ.C)`. (a)* Propagation. *A transclusion composite from `d_i` to `d_{i+1}` whose copied region's content image contains `a` — the new mappings of the K.μ⁺ on `d_{i+1}` include some `u ↦ a`, the address read off `d_i`'s arrangement by FD-IMGC — yields `a ∈ ran_C(d_{i+1}, ·)` at its boundary. Since the image read from `d_i` consists of the very addresses `d_i` arranges, the address arriving in `d_{i+1}` is* the same `a` *that arrived in `d_i` — depth of copying never mints a new identity. (b)* Flat evaluation. *At any state Σ, `finddocs({a}, Σ) = {d ∈ dom(Σ.M) : a ∈ ran_C(d, Σ)}` collects the entire current sharing set of `a` in one comprehension: the criterion mentions no path, no copy event, no `Σ.R`, no other document (FD-LOCAL) — so a chain `d₀ → d₁ → ⋯ → d_n` of transclusion composites, each propagating `a`, leaves all of `d₀, …, d_n` simultaneous members, found without any iterative chain-following, and so found for every `I ∋ a` (FD-IMONO). (c)* Severance immunity. *Membership of the chain's ends does not depend on its middle: if `d_mid` later contracts `a` away, every other document's membership is untouched (FD-LOCAL(i)), so the ends remain co-listed without the middle. Containment "carries through the chain" not because the chain is consulted but because it never needs to be — the sharing relation the operation reads is flat.*

Version forking is the limiting case of transclusion — a whole-document copy onto a fresh identity — and duplicates membership exactly:

**FD-VERS (ForkMembershipDuplication).** *Let `Σ →* Σ'` be a J4 fork composite (ASN-0047) with content source operand `d_op` and fresh version `d_new`. Then for every `I`:*

> `finddocs(I, Σ') = finddocs(I, Σ) ∪ ({d_new} if d_op ∈ finddocs(I, Σ) else ∅)`,

*and in particular `d_new ∈ finddocs(I, Σ') ⟺ d_op ∈ finddocs(I, Σ')`.*

*Derivation. J4's derived consequence gives `ran(M'(d_new)) = ran(M(d_op)|_{V_{s_C}(d_op)})`, and the transcription map φ lands in `V_{s_C}(d_new)`, so `ran_C(d_new, Σ') = ran_C(d_op, Σ)`. The fork's steps frame every existing arrangement — `d_op`'s included — so `ran_C(d, Σ') = ran_C(d, Σ)` for all prior `d`, and every prior membership is unchanged (FD-LOCAL); the only candidate change is the fresh `d_new`, whose criterion evaluates to `d_op`'s. A query on any portion of a document's material therefore returns, immediately and permanently-until-edited, every version sharing those addresses — and as the two arrangements diverge by subsequent edits, their memberships diverge with them, each governed by its own arrangement alone.*

## What Scattered Regions Reveal

A vspec-set may name several non-contiguous regions — across one document or several. What does this buy over a single span? Three results: the union query is exactly the union of the fragment queries (so one call reveals only the *union* of sharing); the per-fragment family reveals the *co-occurrence pattern* — which documents recombine pieces that are separate at the source; and the single contiguous span cannot even pose the fragmentary question, because its denotation is convex.

**FD-COOC (CooccurrenceByComposition).** *For fragments `I₁, …, I_k` (each, say, a fragment's resolution), define a document's incidence `inc(d) = {j : ran_C(d, Σ) ∩ I_j ≠ ∅}`. Then: the union query yields the documents touching* any *fragment, `finddocs(∪_j I_j, Σ) = {d : inc(d) ≠ ∅}` (FD-UDIST); the intersection of the per-fragment queries yields the* recombiners *— the documents drawing together all the scattered pieces — `∩_j finddocs(I_j, Σ) = {d : inc(d) = {1, …, k}}`; and full containment at address grain is the finest such composition: for `I ≠ ∅`, `{d : I ⊆ ran_C(d, Σ)} = ∩_{a ∈ I} finddocs({a}, Σ)`. The guard is the one boundary the identity needs: at `I = ∅` the left side is all of `dom(Σ.M)` — every document vacuously contains all of nothing — while an intersection over an empty index set is undefined until a universe is declared; read within the universe `dom(Σ.M)`, the empty intersection is `dom(Σ.M)` and the identity extends to `I = ∅` as well. Each equality is the comprehension read pointwise. The primitive is the OR; every AND-shaped question — collage detection, coverage testing — is a derived query, obtained by issuing fragments separately and composing, never by one call.*

**FD-LOSSY (MergedResultUnderdetermination).** *The single merged answer under-determines the incidence pattern: there are reachable states `Σ¹, Σ²` and fragments `I₁, I₂` with `finddocs(I₁ ∪ I₂, Σ¹) = finddocs(I₁ ∪ I₂, Σ²)` but different incidences.*

*Construction, at the level of FD-NEUT(c). Register a single document `d` (K.δ scaffolding per ASN-0047; its node and account ancestors are entities but not documents, so `dom(M) = {d}` at every state below — the comprehension has exactly one candidate to test). Let `a₁ = [d.0.s_C.1]` and `a₂ = inc(a₁, 0)` be the first two emissions of `d`'s content chain `A_C(d)` (FirstEmission; distinct by ChainEnumerationInjectivity, ASN-0093), and fix the fragments `I₁ = {a₁}`, `I₂ = {a₂}` — tumbler sets, well-defined at every state whether or not their members are yet allocated. For `Σ¹`, run one valid insertion composite on `d`: K.α allocating `a₁` (first-emission branch), K.μ⁺ arranging the depth-2 position `[1,1] ↦ a₁` (the D-MIN★-conforming first position), K.ρ recording `(a₁, d)`; J0 holds — the composite's one fresh address is arranged at its boundary — and J1★/J1'★ hold — the one range-new address is the one recorded. At this boundary `ran_C(d, Σ¹) = {a₁}`, and `a₂ ∉ dom(Σ¹.C)`: unallocated, hence inert to every query (FD-GROUND). So `finddocs(I₁ ∪ I₂, Σ¹) = {d}`, witnessed by `a₁`, and `inc(d) = {1}` at `Σ¹`. For `Σ²`, continue the same trace with one further valid composite: K.α allocating `a₂` (subsequent-emission branch, `a₂ = inc(a₁, 0)`), K.μ⁻ on `d` at `n'_{s_C} = 0` (full content clear — strict contraction, since `n_{s_C} = 1`), K.μ⁺ arranging `[1,1] ↦ a₂` (its precondition holds at its intermediate state: `a₂ ∈ dom(C)` by the preceding K.α step, and the singleton segment satisfies D-CTG★/D-MIN★), K.ρ recording `(a₂, d)`. The couplings hold initial-to-final: J0 — the composite's one fresh address `a₂` is arranged at the final boundary; J1★ — the content range went `{a₁} → {a₂}`, so the one range-new address is `a₂`, recorded; J1'★ — the one new provenance entry is that range-new address. At this boundary `ran_C(d, Σ²) = {a₂}`, while `a₁ ∈ dom(Σ².C)` still (P0) but is arranged nowhere — allocated, yet contributing no member. So `finddocs(I₁ ∪ I₂, Σ²) = {d}`, witnessed by `a₂`, and `inc(d) = {2}` at `Σ²`. The two answers are equal as sets — `{d} = {d}`, and the equality holds over the whole stratum, since `d` is the only document at either state (any additional registered-but-never-extended document would have `ran_C = ∅` at both states and belong to neither answer) — while the incidence is `{1}` at `Σ¹` and `{2}` at `Σ²`. Together with the bare-identity codomain (FD-V) — no positions, no multiplicity, no matched material — the answer reveals* which documents, *and deliberately nothing else; what was matched, and where, are recoverable only by further queries (FD-COOC for the which-fragment, per-document region queries for the where).*

**FD-CONVEX (SingleSpanConvexityForcing).** *A single contiguous V-span cannot name scattered fragments exactly. Let `σ` be a V-span over `d`'s content positions (T12) with `u, q ∈ ⟦σ⟧ ∩ V_{s_C}(d)`, `u < q`. Then every intervening content position is dragged in: for `v ∈ V_{s_C}(d)` with `u < v < q` — and every same-depth position strictly between two members of `V_{s_C}(d)` is itself a member, by the canonical gap-free form D-SEQ★ — span denotations are order-convex (T12(c)), so `v ∈ ⟦σ⟧`, whence `Σ.M(d)(v) ∈ image_C(⟦σ⟧, d, Σ) ⊆ resolve`. By FD-PART, every document sharing* only the connective material *is admitted. The two-region vspec-set `{(d, W₁), (d, W₂)}` with `u ∈ W₁`, `q ∈ W₂`, `v ∉ W₁ ∪ W₂` resolves to `image_C(W₁, d, Σ) ∪ image_C(W₂, d, Σ)` — exactly the fragments' material — and excludes the connective-only documents whenever the connective image is disjoint from the fragment images (guaranteed when `Σ.M(d)` is injective on the span). "There is no choice as to what lies between; this is implicit in the choice of first and last point" [LM 4/25]; designating "a separated series of items exactly, including nothing else" requires the span-set [LM 4/25] — and FD-CONVEX is why: precision about scattered sharing is expressible only through the multi-region query.*

## Dynamics: Stability Under Editing

We now fix the material `I` and ask which transitions can move `finddocs(I, ·)`, and by how much. The *resolved* I-set is the stable object — its subject matter cannot be destroyed (P0: `dom(C)` only grows and values never change, so a grounded `I` remains grounded forever) — and all dynamics are stated against it. The two-phase operation adds one further motion, through resolution.

**FD-FRAME (NonArrangementInertness).** *Every transition that fixes the content-subspace arrangement family fixes the answer: for every `I`,* K.α, K.λ, K.ρ *(arrangement frames `M' = M`),* K.δ *(Node/Account cases frame `M`; the Document case adds `d_new` with `M'(d_new) = ∅`, never a member, others framed), and* K.μ⁺_L *(adds only `s_L`-positions to one document, so `V_{s_C}(d)` and its images are unchanged) all satisfy `finddocs(I, Σ') = finddocs(I, Σ)`. Derivation: in each case `χ(d, I, ·)` is unchanged for every `d` (FD-LOCAL), and the comprehension's domain either is unchanged or gains only non-members. Allocating content, creating links, recording provenance, registering documents — none of it moves containment.*

**FD-STEP (ArrangementStepCharacterization).** *The only movers are the content-subspace arrangement transitions, and each moves the answer in exactly one place:*

- *K.μ⁺ on `d` (content extension, new images `N = {Σ'.M(d)(v) : v ∈ dom(Σ'.M(d)) ∖ dom(Σ.M(d))}`): `ran_C(d, Σ') = ran_C(d, Σ) ∪ N` (extension frame: prior positions agree, new positions are content-subspace by the amended precondition), all other documents framed, so*

  > `finddocs(I, Σ') = finddocs(I, Σ) ∪ ({d} if N ∩ I ≠ ∅ else ∅)` — *growth at most by the edited document; this is the transclusion case when `N` draws on existing queried addresses.*

- *K.μ⁻ on `d` (contraction with retention set `Ret`): writing `ran_Ret ≡ {Σ.M(d)(v) : v ∈ Ret ∧ subspace(v) = s_C}` — a pre-state quantity — the retained-domain agreement gives `ran_C(d, Σ') = ran_Ret ⊆ ran_C(d, Σ)`, others framed, so*

  > `finddocs(I, Σ') = (finddocs(I, Σ) ∖ {d}) ∪ ({d} if ran_Ret ∩ I ≠ ∅ else ∅)` — *shrinkage at most by the edited document.*

- *K.μ~ on `d` (reorder with witnessing bijection π): the domain is fixed (K.μ~-FIX), π is subspace-preserving, and the bijection equation `Σ'.M(d)(π(v)) = Σ.M(d)(v)` makes the content images a reindexed copy: `ran_C(d, Σ') = ran_C(d, Σ)` (the whole-arrangement analogue is LP11, ASN-0098), so*

  > `finddocs(I, Σ') = finddocs(I, Σ)` — *rearrangement never moves containment.*

**FD-CWP (ContractionSurvivalWP).** *Fix a K.μ⁻ on `d` with retention set `Ret` (per-subspace initial segments, ASN-0047; `Ret ⊆ dom(Σ.M(d))` by D-SEQ★). The weakest precondition on the pre-state under which the edited document survives in the answer is its own enabling condition plus a retained witness:*

> `wp(K.μ⁻[d, Ret], d ∈ finddocs(I, ·)) ≡ enabled(K.μ⁻[d, Ret]) ∧ (E v : v ∈ Ret ∧ subspace(v) = s_C : Σ.M(d)(v) ∈ I)`,

*the existential being exactly `ran_Ret ∩ I ≠ ∅`, a function of `(Σ, Ret)` evaluable before the step. The whole answer is preserved iff survival is owed only where it was held: `finddocs(I, Σ') = finddocs(I, Σ) ⟺ (d ∈ finddocs(I, Σ) ⟹ ran_Ret ∩ I ≠ ∅)` — contraction can never create membership (FD-STEP), so the edited document is the only contingency. Boundary case `Ret = ∅` (full clearance): the existential is false, so the document drops iff it was a member — Nelson's "the editing document drops out of the current answer" [Q8 consultation; LM 4/9, 4/11], with the drop's exact condition computed pre-step.*

**FD-FRESH (InsertionInvariance).** *Editing a returned document elsewhere — inserting fresh material at any position, shifting everything after it — never moves the answer. Stated in the atomic vocabulary, the insertion composite on `d` at position `p` of an `N`-position content segment, with `n ≥ 1` fresh units, is:* K.α *iterated `n` times, allocating fresh `A_new = {a'₁, …, a'ₙ}` along `d`'s content chain `A_C(d)`; the full content clear* K.μ⁻ *on `d` at `n'_{s_C} = 0`, link subspace retained (omitted when `V_{s_C}(d) = ∅` — the first-insertion case); one rebuild* K.μ⁺ *re-populating the canonical segment of length `N + n` — position `k` takes `d`'s old `k`-th image for `1 ≤ k < p`, takes `a'_{k−p+1}` for `p ≤ k < p + n`, and takes `d`'s old `(k − n)`-th image for `p + n ≤ k ≤ N + n`; and* K.ρ *recording `(a', d)` for each `a' ∈ A_new`. This is a valid composite of the declared model. Each step's precondition holds at its intermediate state: the rebuild's images are all allocated — the old ones by P0, the fresh ones by the preceding K.α steps — and its result is the canonical gap-free segment (D-CTG★/D-MIN★); the cleared intermediate state has `V_{s_C}(d) = ∅`, which satisfies the per-state shape package vacuously (D-CTG★, D-MIN★, D-SEQ★ quantify over non-empty subspaces), so every elementary-reachable state of the composite keeps ASN-0047's ExtendedReachableStateInvariants. The couplings hold initial-to-final: J0 — every `a' ∈ A_new` is arranged at the boundary; J1★ — the range-new addresses are exactly `A_new`, each recorded by a K.ρ step; J1'★ — conversely, each new provenance entry is one of those range-new addresses; the old images' mid-composite absence from the arrangement is harmless precisely because the couplings are initial-to-final. The net effect, initial-to-final, is ASN-0082's gap-shift contract realized without traversing any gapped state: positions `≥ p` re-mapped at `shift(v, n)` carrying their old images (I3), the left region's mappings unchanged (I3-L), the vacated gap holding `A_new`, the post-domain exactly the canonical segment (I3-V, I3-CS). Then for every `I` fixed at the pre-state with `I ⊆ dom(Σ.C)`:*

> `finddocs(I, Σ_post) = finddocs(I, Σ_pre)`.

*Derivation, step by step from FD-FRAME and FD-STEP. The K.α steps: FD-FRAME — no motion. The clear: FD-STEP's contraction clause at `ran_Ret = ∅` (no content position retained) — `d` drops iff it was a member, every other document framed (FD-LOCAL). The rebuild: FD-STEP's growth clause with new-image set `N_step = ran_C(d, Σ_pre) ∪ A_new`; K.α's freshness gives `A_new ∩ dom(Σ_pre.C) = ∅ ⊇ A_new ∩ I`, so `N_step ∩ I ≠ ∅ ⟺ ran_C(d, Σ_pre) ∩ I ≠ ∅` — `d` re-enters exactly iff it dropped. The K.ρ steps: FD-FRAME. Net: identity. The deep reason is that the membership criterion contains no V-position term at all (FD-LOCAL reads only the* range *of the content arrangement): positional shift is invisible by construction, and fresh allocation is outside any pre-existing query by the monotone freshness of allocation (ASN-0093). The pure append (`p = N + 1`) needs no clear at all — a bare K.μ⁺ extending the segment with images in `A_new` — with the same conclusion.*

**FD-NONMONO (LiveNonMonotonicity).** *Across `Σ →* Σ'` neither inclusion holds in general: the transclusion step grows the answer (FD-STEP, K.μ⁺ with `N ∩ I ≠ ∅` — realized in the FD-CHAIN propagation), and the contraction step shrinks it (FD-CWP's failing branch — realized in the FD-NEUT(c) construction). The live answer breathes with the arrangements — Nelson's live set "shrinks (deletions) and grows (new inclusions)" by design [Q8 consultation]. For the two-phase operation there is one further motion: the* resolution *itself is present-tense — editing a* named *document moves `resolve(Q, ·)` even while every containment fact is fixed (D-PRES, ASN-0127).*

**FD-VDYN (TwoPhasePerTransitionDynamics).** *Fix a vspec-set `Q` with every named document registered at Σ — so `finddocs_V(Q, ·)` is defined at both ends of any transition, `dom(M)` being monotone (M1) — call `d` named when `(d, W) ∈ Q` for some `W`, and across a transition `Σ → Σ'` write `I = resolve(Q, Σ)`, `I' = resolve(Q, Σ')`. FD-IMGC's defining comprehension consults only `W` and the content-subspace restriction of `Σ.M(d)`, so the resolution moves only when some named document's content-subspace arrangement moves. Four cases exhaust the vocabulary.*

*(a) No named content motion — K.α, K.λ, K.ρ, K.δ anywhere; K.μ⁺_L anywhere (it adds only `s_L`-positions, which `image_C` filters out); K.μ⁺, K.μ⁻, K.μ~ on unnamed documents. Then `I' = I` and the two-phase answer moves exactly as the fixed-`I` answer moves, `finddocs_V(Q, Σ') = finddocs(I, Σ')`, governed by FD-FRAME and FD-STEP — a K.μ⁺ on an unnamed document can still grow it, a K.μ⁻ on one can still shrink it, through the containing rather than the pointing.*

*(b) Extension of a named document — K.μ⁺ on named `d_q`: monotone growth,*

> `finddocs_V(Q, Σ) ⊆ finddocs_V(Q, Σ')`.

*Derivation. The extension frame (new positions added, prior positions agreeing) grows FD-IMGC's comprehension monotonically: `image_C(W, d_q, Σ) ⊆ image_C(W, d_q, Σ')` for every `W` — F-IMG-MONO (ASN-0127) restricted through FD-IMGC — and every other named arrangement is framed, so `I ⊆ I'`. Then `finddocs(I, Σ) ⊆ finddocs(I, Σ')` by FD-STEP's growth clause, and `finddocs(I, Σ') ⊆ finddocs(I', Σ')` by FD-IMONO.*

*(c) Contraction of a named document — K.μ⁻ on named `d_q`: monotone shrinkage,*

> `finddocs_V(Q, Σ') ⊆ finddocs_V(Q, Σ)`.

*Derivation. Retained-domain agreement shrinks the comprehension: `image_C(W, d_q, Σ') ⊆ image_C(W, d_q, Σ)` — F-IMG-CONTR through FD-IMGC — others framed, so `I' ⊆ I`. Then `finddocs(I', Σ') ⊆ finddocs(I, Σ')` by FD-IMONO, and `finddocs(I, Σ') ⊆ finddocs(I, Σ)` by FD-STEP's shrinkage clause.*

*(d) Reorder of a named document — K.μ~ on named `d_q` with witnessing bijection π: the genuinely two-phase case. Every fixed-`I` answer is invariant (FD-STEP, reorder clause), so all motion is resolution motion:*

> `finddocs_V(Q, Σ') = finddocs(I', Σ') = finddocs(I', Σ)`, *with* `image_C(W, d_q, Σ') = image_C(π⁻¹(W), d_q, Σ)`

*— the second equality by FD-STEP at the fixed set `I'`; the swing law from domain fixity (K.μ~-FIX), subspace preservation, and the bijection equation `Σ'.M(d_q)(π(v)) = Σ.M(d_q)(v)`, i.e. F-IMG-SWING restricted through FD-IMGC. Stability condition: if π fixes every named region setwise on the content positions — `π⁻¹(W) ∩ V_{s_C}(d_q) = W ∩ V_{s_C}(d_q)` for each `(d_q, W) ∈ Q` — then `I' = I` and the answer is unchanged. Image motion is necessary for answer motion but not sufficient. Necessity is the stability condition just stated — a fixed named image leaves `I' = I`. Insufficiency is a fact about the fixed post-state: the reorder pins every containment fact (FD-STEP, reorder clause), so `finddocs(·, Σ') = finddocs(·, Σ)` addresswise, and by FD-UDIST the answer moves iff some document meets exactly one of the resolved sets `I, I'` — its content range disjoint from the other; the comprehension *absorbs* the swing when none does, i.e. when every document meets both resolved sets or neither. Absorption is reachable: by valid insertion and transclusion composites (the FD-NEUT(c) pattern) let `d_q` arrange `[1,1] ↦ a, [1,2] ↦ b` and a second document `d_x` arrange both `a` and `b`, and name `W = {[1,1]}`, so `I = {a}`. The admissible reorder π swapping `[1,1] ↔ [1,2]` (two distinct content values, non-trivial net effect) carries the named image to `{b}`, so `I' = {b} ≠ I`; yet `finddocs({a}, Σ) = {d_q, d_x} = finddocs({b}, Σ)` — both documents arrange both addresses — so `finddocs_V(Q, ·)` stands while the image moves. When it is not absorbed the answer genuinely moves while every containment fact stands — the worked illustration exhibits exactly this: a reorder of `d_A` silently drops `d_C` from a re-issued query, `d_C` arranging `a₃` alone and so meeting `I = {a₂, a₃}` but not `I' = {a₁, a₂}`. The pointing moves; nothing contained moves.*

## The Historical Companion: Provenance, Ghosts, and the Index Bound

The foundation state carries a second containment-shaped component: the provenance relation `R`, monotone (P2), populated exactly when material newly enters a document's content range (J1★ from below, J1'★ from above), bounded from below by current containment at every composite boundary (P4★: `Contains_C(Σ) ⊆ R`), and witnessed against actual past arrangements along every valid trace (P4a). This is precisely the abstract shape of a write-once containment *index*, and it induces a second, distinct query:

**FD-HIST (ProvenanceQuery).** *`finddocs_R(I, Σ) ≡ {d ∈ dom(Σ.M) : (E a : a ∈ I : (a, d) ∈ Σ.R)}`. (The guard `d ∈ dom(Σ.M)` loses nothing: `R ⊆ T_elem × E_doc` and `dom(M)` is monotone, M1.)*

**FD-RMONO (HistoricalMonotonicity).** *Across `Σ →* Σ'`: `finddocs_R(I, Σ) ⊆ finddocs_R(I, Σ')`. Derivation: `R ⊆ R'` per transition (P2), `dom(M) ⊆ dom(M')` (M1), both lifted over the finite decomposition of `→*` (SequentialTransitionAxiom); the criterion reads only these monotone components. The historical answer never loses a member — the abstract form of a write-only index.*

**FD-SUPER (LiveBoundedByHistorical).** *At every composite boundary Σ: `finddocs(I, Σ) ⊆ finddocs_R(I, Σ)`. Derivation: a member's present witness (FD-SOUND) is a pair `(a, d) ∈ Contains_C(Σ)` with `a ∈ I` (FD-RAN alignment), and P4★ places it in `Σ.R`. The implementation evidence's formal finding `actual_docs(i) ⊆ find_documents(i)` is this inclusion, derived here from the foundation couplings rather than observed in code.*

**FD-WITNESS (EverContainedEqualsOnceLive).** *For every valid trace `Σ₀ →* Σ₁ →* ⋯ →* Σ_n = Σ` (each `Σ_k` a composite boundary):*

> `finddocs_R(I, Σ) = (∪ k : 0 ≤ k ≤ n : finddocs(I, Σ_k))`.

*Derivation. (⊆) For `d ∈ finddocs_R(I, Σ)` take `a ∈ I` with `(a, d) ∈ Σ.R`; P4a yields a trace state `Σ_k` and a position `v ∈ dom(M_k(d))` with `subspace(v) = s_C ∧ M_k(d)(v) = a` — that is, `a ∈ ran_C(d, Σ_k)`, so `d ∈ finddocs({a}, Σ_k) ⊆ finddocs(I, Σ_k)` (FD-IMONO). (⊇) For `d ∈ finddocs(I, Σ_k)`, its witness pair lies in `Contains_C(Σ_k) ⊆ Σ_k.R` (P4★ at the boundary `Σ_k`) `⊆ Σ.R` (P2 along the suffix), and `d ∈ dom(Σ.M)` by M1. The right-hand union is therefore the same set along* every *valid trace — the historical query answers exactly "which documents have ever contained any of this material," with "ever" meaning: at some composite boundary of the witnessed history.*

**FD-GHOST (GhostCharacterization).** *Define `ghosts(I, Σ) ≡ finddocs_R(I, Σ) ∖ finddocs(I, Σ)`. By FD-WITNESS (the `k = n` term contributing nothing to the difference): `ghosts(I, Σ) = (∪ k : 0 ≤ k < n : finddocs(I, Σ_k)) ∖ finddocs(I, Σ)` — exactly the documents that contained queried material at some past boundary and contain none of it now. The live operation excludes them, and must: FD-SOUND is precisely the statement that membership is never owed to `ghosts`. The two queries are not rivals but answers to different questions — "contains" versus "has ever contained" — and both are well-defined over the same state.*

**FD-COINC (CoincidenceOnNonShrinkingHistories).** *Call a valid trace* range-non-decreasing *when every composite preserves or grows every content range: `(A k, d : 0 ≤ k < n ∧ d ∈ dom(Σ_k.M) : ran_C(d, Σ_k) ⊆ ran_C(d, Σ_{k+1}))` — sufficient syntactic condition: no composite of the trace contains a K.μ⁻ step (then every atomic step frames or extends arrangements, FD-FRAME/FD-STEP; note a reorder's decomposition contains K.μ⁻, though its net effect satisfies the semantic hypothesis anyway). Along such a trace the two queries coincide at the endpoint: `finddocs_R(I, Σ) = finddocs(I, Σ)`. Derivation: (⊇) FD-SUPER. (⊆) FD-WITNESS gives liveness at some `Σ_k`; the chained range inclusions carry the witness `a ∈ ran_C(d, Σ_k) ⊆ ⋯ ⊆ ran_C(d, Σ)`. Ghosts are therefore* exactly *the residue of contraction: divergence between the index and the truth begins with the first deletion, and not before.*

The pair (FD-SUPER, FD-WITNESS) is the complete correctness story for the obvious implementation strategy. A monotone index keyed by I-address — record `(a, d)` whenever material enters `d`'s content range, never erase — computes `finddocs_R` *exactly*: J1★ pins it from below (no placement path may skip recording, so the index can never silently omit a live container — completeness of the index is the coupling invariant, not an implementation virtue), and J1'★ pins it from above (no record without a corresponding placement, so the index contains no fabrications — every entry is a true "once contained", FD-WITNESS). What the raw index answer lacks is only FD-SOUND: recovering the live operation from it requires filtering each candidate through its current arrangement — the per-candidate test `χ(d, I, Σ)`, a lookup against `M(d)`, not a re-search. An implementation that skips the filter has not implemented FINDDOCSCONTAINING wrongly so much as implemented FD-HIST instead.

## Worked Illustration

Pin three registered documents `d_A, d_B, d_C` at a reachable boundary, and let `a₁, a₂, a₃` be the first three emissions of `d_A`'s content sub-allocator `A_C(d_A)` (FirstEmission, ASN-0093) — pairwise distinct (ChainEnumerationInjectivity), each with `origin = d_A`. By valid composites (insertion on `d_A`; transclusion composites with their K.ρ steps discharging J1★): `d_A` arranges `[1,k] ↦ a_k` for `k = 1, 2, 3`; `d_B` transcludes `a₂, a₃` from `d_A`; `d_C` transcludes `a₃`, naming *`d_B`'s* region as its source — resolution through `d_B` reads off the same address `a₃` (FD-ASKER), so the chain `d_A → d_B → d_C` exists only in the history, not in the state. Call the boundary Σ.

*Reach.* `Q = {(d_A, W)}` with `W ∋ [1,2], [1,3]`: `resolve(Q, Σ) = {a₂, a₃}` and `finddocs_V(Q, Σ) = {d_A, d_B, d_C}` — the naming document itself (FD-SELF), and both transcluders, including the depth-2 one, in a single comprehension with no chain-following (FD-CHAIN(b)).

*Fragments.* With `I₁ = {a₁}`, `I₂ = {a₃}`: `finddocs(I₁, Σ) = {d_A}`, `finddocs(I₂, Σ) = {d_A, d_B, d_C}`, and the union query returns their union (FD-UDIST) — while the recombiners of both fragments are `finddocs(I₁, Σ) ∩ finddocs(I₂, Σ) = {d_A}` (FD-COOC), information the single merged answer cannot carry (FD-LOSSY). A single span over `[1,1]…[1,3]` would resolve `a₂` as well (FD-CONVEX); had a fourth document transcluded only the connective `a₂`, the span query would admit it and the two-fragment vspec-set would not.

*Edits.* `d_B` inserts fresh material mid-document — every position shifts, the answer to every pre-resolved query is unchanged (FD-FRESH). `d_A` reorders by the admissible bijection π swapping `[1,1] ↔ [1,3]` — enabled, since `d_A`'s content arrangement takes three pairwise-distinct values, where a reorder of `d_C`'s single-entry arrangement could not fire (K.μ~ requires at least two distinct values and a non-trivial net effect). Every fixed-`I` answer is unchanged (FD-STEP, reorder clause): `ran_C(d_A)` is the same set, merely re-positioned. But re-issuing the vspec-set `Q' = {(d_A, {[1,2], [1,3]})}` now resolves through the post-reorder arrangement `[1,1] ↦ a₃, [1,2] ↦ a₂, [1,3] ↦ a₁` to `{a₂, a₁}` where before it resolved to `{a₂, a₃}` — and the two-phase answer moves from `finddocs({a₂, a₃}) = {d_A, d_B, d_C}` to `finddocs({a₁, a₂}) = {d_A, d_B}`: `d_C` silently drops while no containment fact anywhere has changed. The pointing moved, not the containing (FD-VDYN(d)). `d_B` contracts away `a₃` but retains `a₂`: for `I₂` the survival wp fails at `d_B` (`ran_Ret ∩ I₂ = ∅`, FD-CWP) and the live answer drops to `{d_A, d_C}` — the middle of the historical chain is severed and the ends are co-listed without it (FD-CHAIN(c)).

*Ghosts.* Now `d_A` full-clears (`Ret = ∅`, J2). Live: `finddocs(I₂, ·) = {d_C}` — the origin is gone (FD-NEUT(c)), the material itself untouched in `dom(C)` (P0). Historical: `(a₃, d_A), (a₃, d_B), (a₃, d_C) ∈ R` from the J1★ recordings, so `finddocs_R(I₂, ·) = {d_A, d_B, d_C}`: `ghosts(I₂, ·) = {d_A, d_B}` — exactly the once-but-not-now containers (FD-GHOST), each witnessed live at an earlier boundary of this very trace (FD-WITNESS). A fork of `d_C` at this point would re-grow the live answer by the fresh version, immediately (FD-VERS).

## The Implementation Evidence

Gregory's udanax-green realizes the two-phase shape and most of the algebra with striking literalness; its one deep deviation is exactly the FD-HIST/FD-SOUND boundary. Correspondences first, citing the consultation traces.

*Two phases, literally.* `dofinddocscontaining` is `specset2ispanset` (resolution: V-specs through each named document's POOM to I-spans, `do2.c`) composed with `finddocscontainingsp` (comprehension: a 2-D spanfilade query, `spanf1.c`) — FD-V's factoring as a two-line function body. Resolution flattens the multi-document specset into one I-span list before the search ever runs — FD-RES(c)'s flattening, observed at `do2.c:14-46`.

*The comprehension is global and type-scoped, not subtree-scoped.* The spanfilade query restricts ORGLRANGE to the band `[4, 5)` — the `DOCISPAN` *type discriminator*, which every document's entries inhabit via `prefixtumbler(docisa, 4)` — and restricts SPANRANGE to the queried I-spans; no document-address filter exists, and passing one is a hard `gerror` in both build modes (`retrie.c:244-250`, "shouldent happen till we try something fancier"). The global quantification of FD-COMPLETE is thus architecturally forced: a sub-docuverse restriction is not merely unused but impossible — matching FD-FIND's signature, which has nowhere to put one.

*Any-portion matching, to a single point.* The overlap gate `crumqualifies2d` (`retrie.c:270-305`) implements half-open interval intersection — a crum qualifies iff `query_start < crum_end ∧ crum_start ≤ query_end` — and the inline comment `/* <= was < 12/20/84 */` records a deliberate loosening to admit single-boundary-point contact. FD-PART realized down to one shared I-address.

*Bare, deduplicated identities.* Result entries are `typeaddress` — one tumbler, no span field; the matching SPANRANGE interval sits in the retrieval context and is never read (`spanf1.c:171-181`); `isinlinklist` deduplicates; the wire format is a count plus bare addresses (`putfe.c`). FD-V's codomain and FD-LOSSY's positionlessness, exactly. Per-fragment results are accumulated by union per I-span with dedup — FD-UDIST — and the front-end's recourse for co-occurrence is separate calls, as FD-COOC prescribes.

*The index coupling has no silent-omission channel.* Every placement path that gives a document content runs `insertpm` paired with `insertspanf(…, DOCISPAN)` inside `docopy`/`docopyinternal` (`do1.c:60-62, 78-79`) — INSERT, COPY, and CREATENEWVERSION alike (the version's whole-extent DOCISPAN write at fork time is FD-VERS's mechanism, and the fork requires no BERT on either document). The suspicious commented-out `insertspanf` in `doappend` is redundancy, not omission — `appendpm` routes through `doinsert` and hence through the recording pair, as the comment "appendpm includes insertspanf!" says. This is J1★ realized: the abstract claim that completeness of the historical index is a coupling invariant is what the code's discipline amounts to.

*Stability by representation.* DOCISPAN entries carry only `(document, I-span)` — the spanfilade has no V-dimension (`wisp.h`) — so positional edits cannot touch the index, and fresh insertions allocate I-addresses beyond any prior query's range: Gregory's analysis of INSERT-stability is FD-FRESH's argument, made of storage layout. No BERT or `findorgl` is consulted for queried or returned documents (`NOBERTREQUIRED`; addresses are decoded straight off index keys) — membership is index-keyed, with authority untreated (an open question below).

Deviations:

1. **Green implements the historical query, not the live one.** The spanfilade is write-only — no `deletespanf` exists; `dodeletevspan` touches only the POOM (`deletevspanpm`, `orglinks.c:145-152`) — and `finddocscontainingsp` performs no post-check of candidates against current POOMs, so documents that once held the material are returned forever: ghost documents, `actual_docs(i) ⊆ find_documents(i)`. In this note's terms, XU.87.1's FINDDOCSCONTAINING computes `finddocs_R` (FD-HIST) — monotone (FD-RMONO ↔ the write-only index), complete over live containers (FD-SUPER ↔ the observed superset), every entry once-live (FD-WITNESS ↔ entries written only at placement) — while Nelson's present-tense contract, and his explicit "the editing document drops out of the current answer," specify `finddocs` (FD-SOUND). The gap is exactly the per-candidate arrangement filter the historical-companion section requires of any live implementation — the test `χ(d, I, Σ)` against the current POOM, the I→V check green itself runs on its link-following path but omits here; without it the operation is `finddocs_R`, not the live `finddocs`.
2. **Reach is one server's state.** The abstract Σ is the unified docuverse; green's quantification is complete over its own store, and the multi-server melding that would make "regardless of where the native copies are located" physically global (BEBE, the subrepresentation model) was explicitly left unimplemented in XU.87.1. The specification's completeness is semantic, over the single state; a distributed refinement owes either that state's coherence or an explicit availability qualification — it must not convert "temporarily unreachable" into silent omission.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| FD-IMGC | `image_C(W, d, Σ) = {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d)) ∧ subspace(v) = s_C} = image(W, d, Σ) ∩ dom(Σ.C)` — content-restricted region image | introduced |
| FD-RAN | `ran_C(d, Σ) = image_C(T, d, Σ)`; `a ∈ ran_C(d, Σ) ⟺ (a, d) ∈ Contains_C(Σ)`; finite, grounded in `dom(Σ.C)` | introduced |
| FD-Q | a vspec-set is a finite set of pairs `(d, W)`, `d ∈ dom(Σ.M)`, `W ⊆ T`, possibly spanning many documents | introduced |
| FD-RES | `resolve(Q, Σ) = ∪ image_C(W, d, Σ)` — grounded, finite, and flattening (pair structure discarded) | introduced |
| FD-ASKER | naming material through a transcluder resolves identically to naming it through its origin; the starting document is consumed entirely at resolution | introduced |
| FD-FIND | `finddocs(I, Σ) = {d ∈ dom(Σ.M) : ran_C(d, Σ) ∩ I ≠ ∅}` — the containment comprehension | introduced |
| FD-V | FINDDOCSCONTAINING `= finddocs(resolve(Q, Σ), Σ)`; codomain `𝒫(E_doc)` — bare, deduplicated document identities only; a function of the resolved set, so equal resolutions give equal answers | introduced |
| FD-COMPLETE | no document anywhere in `dom(Σ.M)` whose current content range meets `I` may be omitted; no locality/asker/authorship restriction is expressible | introduced |
| FD-SOUND | every member carries a present witness `(v, a)` — never admitted on past content, value resemblance, or provenance records | introduced |
| FD-GROUND | `finddocs(I, Σ) = finddocs(I ∩ dom(Σ.C), Σ)` — ghost and link-store addresses in the query are inert | introduced |
| FD-PART | one shared address suffices; coverage `I ⊆ ran_C(d, Σ)` is never required — membership relation is shared address of some portion | introduced |
| FD-UDIST | `finddocs(I₁ ∪ I₂, Σ) = finddocs(I₁, Σ) ∪ finddocs(I₂, Σ)`, whence per-region decomposition of the operation | introduced |
| FD-IMONO | `I' ⊆ I ⟹ finddocs(I', Σ) ⊆ finddocs(I, Σ)` | introduced |
| FD-LOCAL | membership criterion `χ(d, I, Σ)` is a function of `(I, Σ.M(d))` alone — cross-document independence and non-impedance | introduced |
| FD-SELF | each naming pair with non-empty content image puts its document in the answer; for single-region queries, self-membership ⟺ non-degeneracy | introduced |
| FD-NEUT | the criterion never consults `origin`/`R`; the origin appears iff it currently qualifies, and reachable states exclude it while transcluders remain | introduced |
| FD-IDENT | `finddocs` is a function of `(I, Σ.M)` — value-blind; coincidental equal values (S4) and same-origin sibling addresses never match | introduced |
| FD-CHAIN | copy steps propagate the same address; the comprehension collects the whole current sharing set flatly, path-free; middle severance leaves the ends co-listed | introduced |
| FD-VERS | a J4 fork duplicates membership: `finddocs(I, Σ') = finddocs(I, Σ) ∪ ({d_new} iff d_op ∈ finddocs(I, Σ))` | introduced |
| FD-COOC | union query = union; recombiners = intersection of per-fragment queries; full containment = `∩_{a ∈ I} finddocs({a}, Σ)` for `I ≠ ∅` (empty intersection read in the universe `dom(Σ.M)`) — AND is derived, the primitive is OR | introduced |
| FD-LOSSY | the merged answer under-determines per-fragment incidence and carries no positions or multiplicity | introduced |
| FD-CONVEX | a single span's convex denotation (T12(c), D-SEQ★) forcibly includes connective material; fragment-exact discovery requires the multi-region vspec-set | introduced |
| FD-FRAME | K.α, K.λ, K.ρ, K.δ, K.μ⁺_L leave `finddocs(I, ·)` unchanged for every `I` | introduced |
| FD-STEP | K.μ⁺ grows the answer by at most the edited document; K.μ⁻ shrinks it by at most the edited document; K.μ~ never moves it | introduced |
| FD-CWP | `wp(K.μ⁻[d, Ret], d ∈ finddocs(I, ·)) ≡ enabled ∧ ran_Ret ∩ I ≠ ∅`; answer preserved iff membership held implies a retained witness | introduced |
| FD-FRESH | the in-vocabulary insertion composite (iterated K.α; full-content-clear K.μ⁻; one rebuild K.μ⁺; K.ρ) — net effect = ASN-0082's I3/I3-L/I3-V/I3-CS initial-to-final — leaves `finddocs(I, ·)` unchanged for every pre-resolved grounded `I`; the criterion has no V-position term | introduced |
| FD-NONMONO | the live answer is non-monotone in both directions across `→*`; additional two-phase motion enters only through resolution drift (D-PRES) | introduced |
| FD-VDYN | per-transition two-phase dynamics: K.μ⁺ on a named document grows `finddocs_V`, K.μ⁻ on one shrinks it, K.μ~ on one moves it only through the swing `image_C(W, d, Σ') = image_C(π⁻¹(W), d, Σ)` (fixed-`I` answers invariant; unchanged when π fixes each named region setwise); all other transitions reduce to fixed-`I` motion | introduced |
| FD-HIST | `finddocs_R(I, Σ) = {d ∈ dom(Σ.M) : (E a ∈ I : (a, d) ∈ Σ.R)}` — the provenance-keyed historical query | introduced |
| FD-RMONO | `Σ →* Σ' ⟹ finddocs_R(I, Σ) ⊆ finddocs_R(I, Σ')` — the historical answer never loses a member | introduced |
| FD-SUPER | at every composite boundary, `finddocs(I, Σ) ⊆ finddocs_R(I, Σ)` (P4★) | introduced |
| FD-WITNESS | along every valid trace, `finddocs_R(I, Σ) = ∪_k finddocs(I, Σ_k)` — ever-contained = once-live, trace-invariantly | introduced |
| FD-GHOST | `ghosts(I, Σ) = finddocs_R ∖ finddocs` = documents live at a past boundary and not now; FD-SOUND = their exclusion | introduced |
| FD-COINC | on range-non-decreasing traces (e.g. K.μ⁻-free), `finddocs_R(I, Σ) = finddocs(I, Σ)` — ghosts are exactly the residue of contraction | introduced |

## Open Questions

- What coherence must containment queries guarantee against states interior to a composite transition, where the coupling invariants that bound provenance against containment need not yet hold?
- What contract must the historical query carry about *when* a returned document contained the material, given that the provenance relation records membership without order, time, or version rank?
- What soundness and stability obligations would an attribution-bearing refinement of the answer — documents paired with the matched material or its current positions — inherit beyond bare membership?
- Must containment queries reach arrangements that existed only at past states of a document, and what state component would have to witness them for such reach to be sound?
- Under what availability model can docuverse-global completeness be weakened for a distributed realization without "temporarily unreachable" collapsing into silent omission?
- What authority must the asker hold over the documents named in a vspec-set, and over the documents returned, for resolution and reporting to be permissible?
- Under what conditions may a monotone provenance record be compacted without breaking the trace-witnessing guarantee that grounds the historical query?
- Can a containment query soundly expose multiplicity — how many distinct positions or regions of a member carry queried material — and which of this note's stability results would survive that enrichment?
