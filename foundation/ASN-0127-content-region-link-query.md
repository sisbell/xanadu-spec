> **ASN-0127 · Content-Region Link Query** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](ASN-0036-strand-model.md), [ASN-0043 · Link Model](ASN-0043-link-model.md), [ASN-0047 · Transition Model](ASN-0047-transition-model.md), [ASN-0058 · Mapping Block Algebra](ASN-0058-bundle-algebra.md), [ASN-0093 · Allocation Substrate](ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0127-content-region-link-query.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0127: Content-Region Link Query

*The foundation algebra for "which links does this content region reach, and what stays stable as state evolves?"*

A reader looks at a stretch of arranged content and asks: *what connects to this from elsewhere?* The substrate question underneath this — independent of any reader-facing operation — is: given a region of V-positions in a document, what set of I-addresses does that region currently cover, and what comprehension over the link store does it induce? This note answers that — and only that.

The algebra factors cleanly through arrangement. The reader names a region in V-coordinates; the document's arrangement resolves that region to a set of I-addresses; the link store is then queried against those I-addresses. The two phases are independent — Phase 1 consults `Σ.M`, Phase 2 consults `Σ.L` — and the stability properties of the composite are determined by which state component each operation moves. The foundation supplies the named primitives, the keystone invariance meta-lemma, and the anchoring taxonomy that distinguishes count-and-set behavior depending on whether the reader's request is *fixed* in the permanent address space or *resolved* through a live arrangement.

## State and notation

Addresses are tumblers from `T` (ASN-0034), totally ordered under T1. We operate over the extended state `Σ = (C, L, E, M, R)` of ASN-0047: content store `C : T ⇀ Val` (append-only with immutable values, S0, ASN-0036), link store `L : T ⇀ Link` (append-only with immutable values, L12, finite at every reachable state, L-fin — both ASN-0043), entity set `E`, arrangement family `M(d) : T ⇀ T` for each `d ∈ dom(M)`, and provenance relation `R`. Links carry endset tuples `Σ.L(a) = (e₁, …, eₙ)` with `n ≥ 3` (L3, ASN-0043); `coverage(e) ⊆ T` is the address set an endset denotes, a deterministic function of its spans (Coverage, ASN-0043).

The K-transition vocabulary is ASN-0047's. The *atomic* vocabulary is `V_atomic = {K.α, K.δ, K.λ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` — `K.α` (content allocation), `K.δ` (entity/document registration), `K.λ` (link creation), `K.μ⁺` and `K.μ⁺_L` (content/link-subspace arrangement extension), `K.μ⁻` (arrangement contraction), `K.ρ` (provenance) — with `K.μ~` (reordering) the named composite of K.μ⁻ + K.μ⁺ (ASN-0047), not itself atomic. `K.λ` is the unique transition that modifies `Σ.L`.

## Phase 1: Region projection through arrangement

Given a document `d ∈ dom(Σ.M)` and a query region `W ⊆ T`, the document's current arrangement resolves the region to a set of I-addresses:

**F-IMG (ImageDefinition).** *For `d ∈ dom(Σ.M)` and `W ⊆ T`:*

> `image(W, d, Σ) ≡ {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d))}`

*For `d ∉ dom(Σ.M)`, `image(W, d, Σ)` is undefined.*

*Degenerate cases.* `image(∅, d, Σ) = ∅` (empty region); and `image(W, d, Σ) = ∅` whenever `W ∩ dom(Σ.M(d)) = ∅` — in particular for a freshly registered document whose arrangement is empty (`dom(Σ.M(d)) = ∅`, the K.δ `Document` post-state, ASN-0047), where the image is `∅` for every `W`.

The intersection `W ∩ dom(Σ.M(d))` is load-bearing: V-positions named by `W` but absent from `d`'s arrangement contribute nothing, so the image fabricates no I-address absent from the arrangement. The image is a forward image of a partial function on its defined domain — a basic projection.

When `W` is a contiguous V-span in a single subspace, the image is a finite union of contiguous I-runs: the restriction `Σ.M(d)|W` admits a unique maximally merged run decomposition (C1a, RestrictionDecomposition, ASN-0058) whose I-extents the image unions. When `v ∈ W` has `subspace(v) = s_L`, S3★ (ASN-0047) routes `Σ.M(d)(v) ∈ dom(Σ.L)` and the image picks up a link address; endsets may reference any address in `T` (L4, ASN-0043), so the link subspace is admissible as a coverage target.

The cross-state claims below — here and throughout the discovery-anchored analysis — evaluate the image at a post-state `Σ'`; their definedness condition `d ∈ dom(Σ'.M)` (F-IMG) is discharged once for all of them by M1 (ArrangementMonotonicity, ASN-0047), whose `dom(Σ.M) ⊆ dom(Σ'.M)` — chained over `→*` where needed — carries `d ∈ dom(Σ.M)` to every successor state.

**F-IMG-MONO (ImageMonotonicityUnderArrangementExtension).** *If `Σ → Σ'` extends `Σ.M(d)` (a K.μ⁺ or K.μ⁺_L step that adds positions to `d`'s arrangement while agreeing on prior positions), then for every `W ⊆ T`:*

> `image(W, d, Σ) ⊆ image(W, d, Σ')`.

*Derivation. The extension frame (K.μ⁺/K.μ⁺_L, ASN-0047) gives `dom(Σ.M(d)) ⊆ dom(Σ'.M(d))` with `Σ'.M(d)(v) = Σ.M(d)(v)` for every `v ∈ dom(Σ.M(d))`. Take any `b ∈ image(W, d, Σ)`; by F-IMG, `b = Σ.M(d)(v)` for some `v ∈ W ∩ dom(Σ.M(d))`. Then `v ∈ W ∩ dom(Σ'.M(d))` (prior domain is included) and `Σ'.M(d)(v) = Σ.M(d)(v) = b` (agreement on the prior domain), so `b ∈ image(W, d, Σ')`.*

**F-IMG-CONTR (ImageContractionUnderArrangementContraction).** *If `Σ → Σ'` contracts `Σ.M(d)` (a K.μ⁻ step), then:*

> `image(W, d, Σ') ⊆ image(W, d, Σ)`.

*Derivation. Symmetric to F-IMG-MONO. The contraction frame (K.μ⁻, ASN-0047) gives `dom(Σ'.M(d)) ⊆ dom(Σ.M(d))` with `Σ'.M(d)(v) = Σ.M(d)(v)` for every `v ∈ dom(Σ'.M(d))` (retained-domain agreement). Take any `b ∈ image(W, d, Σ')`; then `b = Σ'.M(d)(v)` for some `v ∈ W ∩ dom(Σ'.M(d))`, whence `v ∈ W ∩ dom(Σ.M(d))` and `Σ.M(d)(v) = Σ'.M(d)(v) = b`, so `b ∈ image(W, d, Σ)`.*

**F-IMG-SWING (ImageSwingUnderReorder).** *If `Σ → Σ'` is a K.μ~ reorder of `d`'s arrangement with witnessing bijection `π`, then `image(W, d, Σ') = {Σ.M(d)(u) : u ∈ π⁻¹(W) ∩ dom(Σ.M(d))}`. The total range is preserved (LP11, ASN-0098: `ran(Σ'.M(d)) = ran(Σ.M(d))`) but the forward image of a fixed sub-region `W` may move; the index-set cardinality is pinned — `|π⁻¹(W) ∩ dom(Σ.M(d))| = |W ∩ dom(Σ.M(d))|` — so under injective `Σ.M(d)` the image cardinality is pinned as well: only membership change is realizable. The shapes a moved image can take, and how injectivity decides their availability, are the subject of F-IMG-TAX.*

*Derivation. K.μ~-FIX (ASN-0047) gives `dom(Σ'.M(d)) = dom(Σ.M(d))`, so the witness `π : dom(Σ.M(d)) → dom(Σ'.M(d))` is a bijection of `dom(Σ.M(d))` onto itself, satisfying the bijection equation `Σ'.M(d)(π(u)) = Σ.M(d)(u)` for every `u ∈ dom(Σ.M(d))`. Unfolding F-IMG at `Σ'` and reindexing each `v = π(u)`: since `π` ranges over all of `dom(Σ'.M(d))`, `v ∈ W ⟺ u ∈ π⁻¹(W)`, and `Σ'.M(d)(v) = Σ'.M(d)(π(u)) = Σ.M(d)(u)`, whence `image(W, d, Σ') = {Σ'.M(d)(v) : v ∈ W ∩ dom(Σ'.M(d))} = {Σ.M(d)(u) : u ∈ π⁻¹(W) ∩ dom(Σ.M(d))}`. That `π` need not fix `W` setwise is why the image membership can change. The cardinality, however, is not free to move under an arbitrary reorder: `π` is a bijection on `dom(Σ.M(d))`, so `|π⁻¹(W) ∩ dom(Σ.M(d))| = |W ∩ dom(Σ.M(d))|` always. When `Σ.M(d)` is injective, these equal-size index sets carry to equal-size images — the image can only change membership, never gain or lose.*

**F-IMG-TAX (MovedImageShapeTaxonomy).** *A moved image — `image(W, d, Σ') ≠ image(W, d, Σ)` under a K.μ~ reorder — stands to its predecessor in exactly one of two shapes: containment motion (`⊆`-comparable — strict, since equality is no move — so the cardinality changes) or incomparable motion (neither `⊆` nor `⊇`). Injectivity decides which shapes are available: under injective `Σ.M(d)` only incomparable motion is realizable; non-injectivity — content sharing (M13/M14, ASN-0058) — makes containment motion available but does not force it: incomparable motion remains realizable as well.*

*Derivation. The two shapes are exhaustive and exclusive for a pair of distinct sets: either one strictly contains the other or neither contains the other. Under injective `Σ.M(d)`, containment motion is unavailable: F-IMG-SWING pins the image cardinality, so a moved image takes two distinct equal-size values, and distinct equal-size finite sets cannot nest (the image is finite: `image(W, d, Σ) ⊆ ran(Σ.M(d))` and `dom(Σ.M(d))` is finite by S8-fin, ASN-0036); the injective witness below realizes the one remaining shape. Under non-injective `Σ.M(d)` the cardinality pinning does not carry from index sets to images: the gain and loss witnesses realize containment motion in both directions, while the four-position witness realizes incomparable motion under a non-injective arrangement — so non-injectivity is necessary for containment motion but not sufficient to force one.*

*Witness admissibility. Each reorder witness below — injective, gain, loss, and four-position — is an admissible K.μ~ instance (ASN-0047) fired from a conforming pre-state: pin `vₖ = [1, k]` (`k = 1, …, 4`; each witness's domain an initial segment of these, the canonical shape available under D-SEQ★, ASN-0047), and pin the images `a, b, c` as the first three emissions of `d`'s content sub-allocator `A_C(d)`, committed to `dom(Σ.C)` by prior K.α steps (ASN-0093) — pairwise distinct by ChainEnumerationInjectivity (ASN-0093). Every pinned position is a content-subspace V-position whose image lies in `dom(Σ.C)`, so the value-side invariant S3★ (ASN-0047) holds at each witness pre-state, and the shared-image assignments of the gain, loss, and four-position witnesses are available because K.μ⁺'s precondition demands only membership `∈ dom(C)` per new mapping: content sharing is permitted (M13/M14, ASN-0058). On this pinned domain any permutation of these same-depth content-subspace positions is length-preserving (iii) and subspace-preserving (iv) automatically and link-subspace-fixing (v) vacuously (no `s_L`-position in the domain); each witness's `π` is defined as a permutation of this pinned position set, so `dom(Σ'.M(d)) = π(dom(Σ.M(d))) = dom(Σ.M(d))` by construction; the shape invariants (i) — S8a, S8-depth, D-CTG★, D-MIN★, all properties of that unchanged domain — persist in the post-state, and S3★ persists alongside them because the reorder preserves the arrangement's range, `ran(Σ'.M(d)) = ran(Σ.M(d))` (LP11, ASN-0098); each witness's value assignment takes at least two distinct values (K.μ~'s precondition); and each witness's `π` moves some position's image — the non-trivial net effect (ii).*

*Injective witness (incomparable motion). With `Σ.M(d) : v₁ ↦ a, v₂ ↦ b` (injective, `a ≠ b`) and `W = {v₁}`, `image(W, d, Σ) = {a}`; the transposition reorder `π = (v₁ v₂)` yields `Σ'.M(d) : v₁ ↦ b, v₂ ↦ a` and `π⁻¹(W) = {v₂}`, so `image(W, d, Σ') = {b}` — the same cardinality, membership moved, the two values `⊆`-incomparable.*

*Gain witness (containment motion, non-injective). With `Σ.M(d) : v₁ ↦ a, v₂ ↦ a, v₃ ↦ b` (so `a` is shared) and `W = {v₁, v₂}`, `image(W, d, Σ) = {a}`; the reorder `π` given by `π(v₁) = v₁, π(v₂) = v₃, π(v₃) = v₂` yields `Σ'.M(d) : v₁ ↦ a, v₂ ↦ b, v₃ ↦ a`, and `π⁻¹(W) = {v₁, v₃}` gives `image(W, d, Σ') = {a, b}` — a gain from one member to two, with `ran(Σ'.M(d)) = ran(Σ.M(d)) = {a, b}` preserved throughout.*

*Loss witness (containment motion, non-injective). With `Σ.M(d) : v₁ ↦ a, v₂ ↦ b, v₃ ↦ b` (so `b` is shared) and `W = {v₁, v₂}`, `image(W, d, Σ) = {a, b}`; the reorder `π` given by `π(v₁) = v₃, π(v₂) = v₁, π(v₃) = v₂` yields `Σ'.M(d) : v₁ ↦ b, v₂ ↦ b, v₃ ↦ a`, and `π⁻¹(W) = {v₂, v₃}` gives `image(W, d, Σ') = {b}` — a loss from two members to one (`{b} ⊊ {a, b}`), again with the total range preserved.*

*Four-position witness (incomparable motion, non-injective). With `Σ.M(d) : v₁ ↦ a, v₂ ↦ b, v₃ ↦ c, v₄ ↦ a` (`a, b, c` pairwise distinct, so `a` is shared by `v₁` and `v₄`) and `W = {v₁, v₂}`, `image(W, d, Σ) = {a, b}`; the transposition `π = (v₂ v₃)` yields `Σ'.M(d) : v₁ ↦ a, v₂ ↦ c, v₃ ↦ b, v₄ ↦ a` with `image(W, d, Σ') = {a, c}` — `⊆`-incomparable with `{a, b}` despite the non-injective arrangement.*

## Phase 2: Per-link matching

Given an I-address set `I ⊆ T`, the per-link relevance test names which links the set reaches:

**F-MATCH (MatchPredicate).** *For `a ∈ dom(Σ.L)` and `I ⊆ T`:*

> `matches(a, I, Σ) ≡ (E i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ I ≠ ∅)`.

A link matches the I-address set when *some* slot's coverage meets it.

**F-FIND (FindPrimitive).** *The bare comprehension:*

> `findlinks(I, Σ) ≡ {a ∈ dom(Σ.L) : matches(a, I, Σ)}`.

*Degenerate case.* `findlinks(∅, Σ) = ∅`: for every `a ∈ dom(Σ.L)` and every slot `i`, `coverage(Σ.L(a).eᵢ) ∩ ∅ = ∅`, so F-MATCH's slot existential has no non-empty intersection to witness and `matches(a, ∅, Σ)` is false; the comprehension therefore collects no link. The empty I-argument is exactly the Phase-2 input produced by an empty Phase-1 image (see F-V).

**F-UDIST (UnionDistributivity).** *For all I-address sets `I₁, I₂ ⊆ T` — no disjointness required:*

> `findlinks(I₁ ∪ I₂, Σ) = findlinks(I₁, Σ) ∪ findlinks(I₂, Σ)`.

*Derivation. Fix `a ∈ dom(Σ.L)` and unfold the match predicate at `I₁ ∪ I₂`: `matches(a, I₁ ∪ I₂, Σ) ≡ (E i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ (I₁ ∪ I₂) ≠ ∅)`. Intersection distributes over union — `coverage(Σ.L(a).eᵢ) ∩ (I₁ ∪ I₂) = (coverage(Σ.L(a).eᵢ) ∩ I₁) ∪ (coverage(Σ.L(a).eᵢ) ∩ I₂)` — and a union is non-empty iff one of its parts is, so the slot test becomes `coverage(Σ.L(a).eᵢ) ∩ I₁ ≠ ∅ ∨ coverage(Σ.L(a).eᵢ) ∩ I₂ ≠ ∅`. The existential distributes over this disjunction, giving `matches(a, I₁, Σ) ∨ matches(a, I₂, Σ)`. None of these steps consults `I₁ ∩ I₂`, so the law holds for arbitrary `I₁, I₂`. Set-builder over the disjunction splits the comprehension into `findlinks(I₁, Σ) ∪ findlinks(I₂, Σ)`.*

**F-IMONO (FindMonotonicityInI — corollary of F-UDIST).** *For all I-address sets `I' ⊆ I ⊆ T`:*

> `findlinks(I', Σ) ⊆ findlinks(I, Σ)`.

*Derivation. Write `I = I' ∪ (I ∖ I')` and apply F-UDIST: `findlinks(I, Σ) = findlinks(I', Σ) ∪ findlinks(I ∖ I', Σ) ⊇ findlinks(I', Σ)`. Monotonicity in the I-argument is thus immediate from union-distributivity.*

## The two-phase composite

**F-V (TwoPhaseFactoring).** *The two-phase combinator composes the projection with the per-link comprehension. For `d ∈ dom(Σ.M)`, `W ⊆ T`:*

> `findlinks_V(W, d, Σ) ≡ findlinks(image(W, d, Σ), Σ)`,

*undefined when `d ∉ dom(Σ.M)`. Degenerate case: `findlinks_V(W, d, Σ) = ∅` whenever `image(W, d, Σ) = ∅` — in particular for `W = ∅`, for any `W` with `W ∩ dom(Σ.M(d)) = ∅`, and for a freshly registered `d` with empty arrangement — since `findlinks(∅, Σ) = ∅` (F-FIND).*

**F-FULL (FullRegionReduction).** *For `d ∈ dom(Σ.M)` and any region `W ⊇ dom(Σ.M(d))` — in particular `W = dom(Σ.M(d))` or `W = T`:*

> `findlinks_V(W, d, Σ) = {a ∈ dom(Σ.L) : discoverable_from(a, d, Σ)}`,

*where `discoverable_from` is ASN-0098's discovery predicate, defined at every `a ∈ dom(Σ.L)` and `d ∈ dom(Σ.M)` — exactly the range of the comprehension.*

*Derivation. `W ⊇ dom(Σ.M(d))` gives `W ∩ dom(Σ.M(d)) = dom(Σ.M(d))`, so `image(W, d, Σ) = {Σ.M(d)(v) : v ∈ dom(Σ.M(d))} = ran(Σ.M(d))` (F-IMG). Unfolding F-V and F-FIND, `findlinks_V(W, d, Σ) = {a ∈ dom(Σ.L) : matches(a, ran(Σ.M(d)), Σ)}`, and F-MATCH at `I = ran(Σ.M(d))` reads `(E i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ ran(Σ.M(d)) ≠ ∅)` — the right-hand side of LP12 (DiscoverabilityCharacterisation, ASN-0098). So `matches(a, ran(Σ.M(d)), Σ) ⟺ discoverable_from(a, d, Σ)` at every `a ∈ dom(Σ.L)`, and set-builder closes the equality.*

**F-VDIST (RegionUnionDistributivity).** *For `d ∈ dom(Σ.M)` and any V-regions `W₁, W₂ ⊆ T` — no disjointness required:*

> `findlinks_V(W₁ ∪ W₂, d, Σ) = findlinks_V(W₁, d, Σ) ∪ findlinks_V(W₂, d, Σ)`.

*Derivation. The image is a forward image of the partial function `Σ.M(d)`, and forward image distributes over union of its argument. Unfolding F-IMG, `image(W₁ ∪ W₂, d, Σ) = {Σ.M(d)(v) : v ∈ (W₁ ∪ W₂) ∩ dom(Σ.M(d))}`; since `(W₁ ∪ W₂) ∩ dom(Σ.M(d)) = (W₁ ∩ dom(Σ.M(d))) ∪ (W₂ ∩ dom(Σ.M(d)))`, the image splits as `image(W₁, d, Σ) ∪ image(W₂, d, Σ)`. Then `findlinks_V(W₁ ∪ W₂, d, Σ) = findlinks(image(W₁ ∪ W₂, d, Σ), Σ) = findlinks(image(W₁, d, Σ) ∪ image(W₂, d, Σ), Σ) = {F-UDIST} findlinks(image(W₁, d, Σ), Σ) ∪ findlinks(image(W₂, d, Σ), Σ) = findlinks_V(W₁, d, Σ) ∪ findlinks_V(W₂, d, Σ)`. The middle step is exactly where F-UDIST must be unrestricted: even when `W₁ ∩ W₂ = ∅`, the two images may overlap — distinct V-positions can resolve to a shared I-address under content sharing (M13/M14, ASN-0058) — so a disjointness-restricted union law would not close this composition.*

## The stability keystone

The meta-lemma below is the keystone of the *store-fixed* lane: its hypothesis is literal store equality `Σ.L = Σ'.L`.

**F-CIL (ComprehensionInvariantUnderΣL — meta-lemma).** *If `Σ.L = Σ'.L` as partial functions, then for every comprehension*

> `{a ∈ dom(Σ.L) : P(a, Σ)}`

*whose membership predicate `P` consults only `Σ.L` and query-data (never `Σ.M`, `Σ.C`, `Σ.E`, `Σ.R`):*

> `{a ∈ dom(Σ.L) : P(a, Σ)} = {a ∈ dom(Σ'.L) : P(a, Σ')}`.

*Derivation chain. `Σ.L = Σ'.L` gives `dom(Σ.L) = dom(Σ'.L)` and per-link value equality `Σ.L(a) = Σ'.L(a)`. Component-wise tuple equality on link values (L6, ASN-0043) gives `|Σ.L(a)| = |Σ'.L(a)|` and per-slot endset equality `Σ.L(a).eᵢ = Σ'.L(a).eᵢ`. Coverage is a deterministic function of its endset argument, so per-slot coverage agrees. Any membership predicate built from these evaluates identically at the two states; set extensionality closes the equality.*

A weaker per-link form survives where F-CIL's global hypothesis fails — under K.λ, `dom(Σ'.L) = dom(Σ.L) ∪ {ℓ_new} ≠ dom(Σ.L)`, while each prior key `a ∈ dom(Σ.L)` keeps its value:

**F-CIL-perlink (PerLinkInvarianceUnderValuePreservation — sub-lemma).** *For any `a` with `a ∈ dom(Σ.L) ∩ dom(Σ'.L)` and `Σ'.L(a) = Σ.L(a)`: `matches(a, I, Σ) ⟺ matches(a, I, Σ')` for every `I ⊆ T`.*

*Derivation. From the per-link value equality `Σ'.L(a) = Σ.L(a)`, component-wise tuple equality on link values (L6) gives arity equality `|Σ'.L(a)| = |Σ.L(a)|` and per-slot endset equality `Σ'.L(a).eᵢ = Σ.L(a).eᵢ`; coverage is a deterministic function of its endset argument, so per-slot coverage agrees, `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`. The `matches` existential `(E i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ I ≠ ∅)` is built from exactly the arity bound and the per-slot coverage, so it evaluates identically at `Σ` and `Σ'`.*

## Operational consequences

F-CIL turns the question "which transitions preserve the result?" into the question "which transitions preserve `Σ.L`?"

**F-PRES (PublishedFramePreservation).** *Every transition in `V_atomic ∖ {K.λ} = {K.α, K.δ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` and the composite `K.μ~` preserves the link store: `dom(Σ'.L) = dom(Σ.L) ∧ (A a ∈ dom(Σ.L) :: Σ'.L(a) = Σ.L(a))`. The atomic operations publish `L' = L` in their effect frame (ASN-0047). The composite `K.μ~` is K.μ⁻ + K.μ⁺ and preserves `Σ.L` by composition.*

**F-INERT (LinkStoreInertPreservation).** *For every transition in `(V_atomic ∪ {K.μ~}) ∖ {K.λ}` and every `I ⊆ T`:*

> `findlinks(I, Σ) = findlinks(I, Σ')`.

*F-PRES gives `Σ.L = Σ'.L`; F-CIL forces the equality at a single step. The lift to any path `Σ →* Σ'` whose every atomic step is in `V_atomic ∖ {K.λ}` is by induction on path length: a length-0 path gives the equality trivially, and a length-`(n + 1)` path factors as `Σ →* Σ'' → Σ'` with `findlinks(I, Σ) = findlinks(I, Σ'')` by the induction hypothesis and `findlinks(I, Σ'') = findlinks(I, Σ')` by the single-step case; transitivity of `=` closes.*

**F-LAMBDA (KλInducedIncrement).** *For a single-step transition `Σ → Σ'` produced by `K.λ` allocating a fresh link `ℓ_new` with endsets `(e₁, …, e_N)`, and any `I ⊆ T`:*

> `findlinks(I, Σ') = findlinks(I, Σ) ⊎ ({ℓ_new} if matches(ℓ_new, I, Σ') else ∅)`.

*The two parts are disjoint: K.λ's freshness precondition (FirstEmissionFreshness/SubsequentEmissionFreshness, ASN-0093) gives `ℓ_new ∉ dom(Σ.L) ∪ dom(Σ.C)`, hence `ℓ_new ∉ findlinks(I, Σ)`. The prior-key contribution is preserved by F-CIL-perlink applied at each `a ∈ dom(Σ.L)`; the fresh-key contribution is the singleton `{ℓ_new}` exactly when the match holds at the new state.*

`K.λ` is therefore the unique single-step source of change in `findlinks(I, Σ)` for *fixed* `I` — the existence-anchored result — and its effect there is fully characterized.

## Anchoring: existence vs discovery

The crux of how a caller experiences `findlinks_V`'s behavior is *how the I-address argument is obtained*, because that choice fixes whether the answer is a stable property of the permanent store or a live reading of the current arrangement.

### Existence anchoring

The request is given directly as a fixed I-address set `I ⊆ T` in the permanent address space; the match predicate then turns only on `coverage(Σ.L(a).eᵢ) ∩ I`.

**E-INV (CoveragePermanence).** *For fixed `I` and any `Σ →* Σ'`, every `a ∈ dom(Σ.L)` satisfies `a ∈ dom(Σ'.L)` and `matches(a, I, Σ') ⟺ matches(a, I, Σ)`.*

*Derivation. LP13 (UnconditionalLinkPersistence, ASN-0098) gives `a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)` across `Σ →* Σ'` — exactly F-CIL-perlink's hypothesis, `a ∈ dom(Σ.L) ∩ dom(Σ'.L)` with per-link value equality. F-CIL-perlink then delivers `matches(a, I, Σ') ⟺ matches(a, I, Σ)`.*

**E-MONO (ExistenceMonotonicity).** *For fixed `I`, `Σ →* Σ' ⟹ findlinks(I, Σ) ⊆ findlinks(I, Σ')`.*

*The store grows across the transitive closure (Store Monotonicity★, ASN-0098), coverage is invariant (E-INV), so the matching set only gains members.*

**E-CONS (CreationConservation).** *For fixed `I`, the set difference `findlinks(I, Σ') ∖ findlinks(I, Σ)` over `Σ →* Σ'` consists of exactly those links created on that path whose stored value matches `I`.*

*The "exactly" is a two-direction set equality, and the statement uses two phrases — "created on the path" and "whose stored value matches `I`" — that must be pinned down before either direction can be proved. We fix them first.*

*The anchor. By SequentialTransitionAxiom (ASN-0047) the path is a finite sequence of atomic steps `Σ = Σ₀ → Σ₁ → ⋯ → Σₙ = Σ'` drawn from `V_atomic`; say `a` is "created on the path" when some step `Σₖ → Σₖ₊₁` is a K.λ allocating `a`. This event reading coincides with the set-difference reading `a ∈ dom(Σ'.L) ∖ dom(Σ.L)`, in both directions. From event to difference: K.λ's freshness precondition at the creating state (FirstEmissionFreshness/SubsequentEmissionFreshness, ASN-0093) gives `a ∉ dom(Σₖ.L)`, and Store Monotonicity★ (ASN-0098) on the prefix `Σ →* Σₖ` gives `dom(Σ.L) ⊆ dom(Σₖ.L)`, whence `a ∉ dom(Σ.L)`; K.λ's effect puts `a ∈ dom(Σₖ₊₁.L)`, and Store Monotonicity★ on the suffix `Σₖ₊₁ →* Σ'` lifts this to `a ∈ dom(Σ'.L)`. From difference to event: suppose `a ∈ dom(Σ'.L) ∖ dom(Σ.L)`. The index set `{j : a ∈ dom(Σⱼ.L)}` is a non-empty subset of `{0, …, n}` (it contains `n`), so it has a least element, and `a ∉ dom(Σ₀.L)` places that least element at some `k + 1 ≥ 1`; minimality gives `a ∉ dom(Σₖ.L)`. The step `Σₖ → Σₖ₊₁` therefore changes `dom(L)`; since `K.λ` is the unique `Σ.L`-modifying transition (State and notation — formally, every other atomic step publishes `L' = L`, F-PRES), that step is a K.λ, and its effect `L' = L ∪ {ℓ_new ↦ (e₁, …, e_N)}` gives `dom(Σₖ₊₁.L) ∖ dom(Σₖ.L) ⊆ {ℓ_new}`, forcing `a = ℓ_new` — the step is the K.λ allocating `a` itself. The two readings agree; both directions below use them interchangeably.*

*The match. `matches` is state-indexed, so the statement's state-free "whose stored value matches `I`" needs a warrant: for `a` created at `Σₖ → Σₖ₊₁`, E-INV instantiated on the suffix `Σₖ₊₁ →* Σ'` — LP13's value persistence `Σ'.L(a) = Σₖ₊₁.L(a)`, the path form of L12's value permanence, feeding F-CIL-perlink, whose match test reads only the stored value — gives `matches(a, I, Σₖ₊₁) ⟺ matches(a, I, Σ')`. Match-at-creation and match-at-`Σ'` are therefore the same predicate of the per-link constant value; we evaluate at `Σ'`.*

*Exclusion (`⊆`) — the direction that needs E-INV. Take any `a ∈ findlinks(I, Σ') ∖ findlinks(I, Σ)`. Either `a ∉ dom(Σ.L)` or `a ∈ dom(Σ.L)`. Suppose `a ∈ dom(Σ.L)`: then E-INV gives `matches(a, I, Σ) ⟺ matches(a, I, Σ')`; from `a ∈ findlinks(I, Σ')` we have `matches(a, I, Σ')`, hence `matches(a, I, Σ)`, and together with `a ∈ dom(Σ.L)` this places `a ∈ findlinks(I, Σ)` — contradicting `a ∉ findlinks(I, Σ)`. So the second case is impossible: only `a ∉ dom(Σ.L)` survives, which together with `a ∈ dom(Σ'.L)` (F-FIND's comprehension presupposes it) is exactly the set-difference reading; the anchor's difference-to-event half exhibits the creating K.λ step, and `a` matches at `Σ'` by its membership in `findlinks(I, Σ')`.*

*Inclusion (`⊇`) — the converse. Take any `a` created on the path with `matches(a, I, Σ')`. The anchor's event-to-difference half delivers both memberships: `a ∈ dom(Σ'.L)`, by K.λ's effect at the creating step lifted through Store Monotonicity★ on the suffix `Σₖ₊₁ →* Σ'`; and `a ∉ dom(Σ.L)`, by K.λ's freshness at `Σₖ` (FirstEmissionFreshness/SubsequentEmissionFreshness, ASN-0093) pulled back through Store Monotonicity★ on the prefix `Σ →* Σₖ`. Then `a ∈ dom(Σ'.L) ∧ matches(a, I, Σ')` is exactly `a ∈ findlinks(I, Σ')` (F-FIND), while `a ∉ dom(Σ.L)` excludes `a` from `findlinks(I, Σ)`, whose comprehension ranges over `dom(Σ.L)` (F-FIND); so `a` sits in the difference. Creation is therefore the sole source of change.*

### Discovery anchoring

The request is resolved through a querying document's current arrangement. Given `d_q ∈ dom(Σ.M)` and a query V-region `W ⊆ T`, the I-address argument is the state-resolved image `image(W, d_q, Σ)`, and the combinator applied is `findlinks_V` itself (F-V) — discovery anchoring introduces no new function, only a mode of use: the I-argument is read off the live arrangement at query time rather than given as a fixed set.

**D-PRES (PresentTenseResolution).** *Editing `d_q` moves content into or out of the queried V-region without any link being created or retracted, so the resolved request — and hence `findlinks_V` — can change while `dom(Σ.L)` is fixed.*

**D-ABSORB (ImageMotionAbsorption).** *Across any `Σ.L`-preserving transition `Σ → Σ'` (F-PRES), image-motion is necessary but not sufficient for the discovery set to move: `findlinks_V(W, d_q, Σ') ≠ findlinks_V(W, d_q, Σ)` requires `image(W, d_q, Σ') ≠ image(W, d_q, Σ)`, but a moved image can leave the discovery set fixed.*

*Derivation. Necessity: if the image is unchanged, F-INERT at the common I-argument gives `findlinks_V(W, d_q, Σ') = findlinks(image(W, d_q, Σ'), Σ') = findlinks(image(W, d_q, Σ), Σ) = findlinks_V(W, d_q, Σ)`. Insufficiency: by F-MATCH's per-slot existential, a displaced in-region I-address relocates a link only when it was that link's sole in-region witness and the swapped-in address does not re-witness the same links — a multi-span slot can absorb the motion. Insufficiency witness: with `Σ.M(d_q) : v₁ ↦ a, v₂ ↦ b` injective and `W = {v₁}` — the injective-witness shape of F-IMG-TAX, whose transposition `π = (v₁ v₂)` is admissible by the same construction, here with `a, b` pinned as the first two emissions of `A_C(d_q)`: distinct allocated content addresses `a, b ∈ dom(Σ.C)` (ChainEnumerationInjectivity, ASN-0093), so S3★ holds at the pre-state and carries to the post-state (range preserved, LP11, ASN-0098) — let `dom(Σ.L)` hold the single link `ℓ = [d_q, 0, s_L, 1]`, the first emission of `d_q`'s link sub-allocator `A_L(d_q)` (FirstEmission, ASN-0093 — discharging L0, L1, and L1c structurally, and L1a since `home(ℓ) = d_q ∈ dom(Σ.M)`), whose value is the conforming triple with slot-1 endset the two-span set `{(a, δ(1, #a)), (b, δ(1, #b))}` (coverage `subtree(a) ∪ subtree(b)`), slot-2 endset `∅`, and non-empty type slot `Θ = {(a_θ, δ(1, #a_θ))}` (L3) at `a_θ = inc(ℓ, 0) = [d_q, 0, s_L, 2]`, an unallocated ghost type (L9, ASN-0043): the reorder moves the image `{a} ↦ {b}`, yet slot 1 meets the image at both states (`a ∈ subtree(a)`, `b ∈ subtree(b)`), so `findlinks_V(W, d_q, ·)` stays fixed at the singleton `{ℓ}` while the image moves. The conclusion is independent of the choice of slot 3: the store holds exactly one link and slot 1 alone witnesses the match at both states, so F-MATCH's existential fires at `ℓ` — and the comprehension returns `{ℓ}` — whatever `Θ` covers.*

**D-NONMONO (DiscoveryNonMonotonicity).** *`findlinks_V` is not monotone across `Σ →* Σ'`. By case analysis on the K-transition:*

- *K.μ⁺ or K.μ⁺_L on `d_q`*: the arrangement extends, so `image(W, d_q, Σ) ⊆ image(W, d_q, Σ')` (F-IMG-MONO). These transitions preserve `Σ.L` (F-PRES), so `findlinks(·, Σ) = findlinks(·, Σ')` for any fixed I-argument (F-INERT); this bridges the comprehension's evaluation state, letting it be held fixed at `Σ'` while only the image moves. Hence `findlinks_V(W, d_q, Σ) = findlinks(image(W, d_q, Σ), Σ) = findlinks(image(W, d_q, Σ), Σ') ⊆ findlinks(image(W, d_q, Σ'), Σ') = findlinks_V(W, d_q, Σ')` — the middle equality by F-INERT, the inclusion by F-IMG-MONO then F-IMONO evaluated at `Σ'`. The discovery set can only grow; the new I-addresses falling in `W`'s positions are what add the new link matches, evaluated against the unchanged store.
- *K.μ⁻ on `d_q`*: the arrangement contracts, so `image(W, d_q, Σ') ⊆ image(W, d_q, Σ)` (F-IMG-CONTR). K.μ⁻ preserves `Σ.L` (F-PRES), so `findlinks(·, Σ') = findlinks(·, Σ)` for any fixed I-argument (F-INERT). Hence `findlinks_V(W, d_q, Σ') = findlinks(image(W, d_q, Σ'), Σ') = findlinks(image(W, d_q, Σ'), Σ) ⊆ findlinks(image(W, d_q, Σ), Σ) = findlinks_V(W, d_q, Σ)` — the middle equality by F-INERT, the inclusion by F-IMONO evaluated at `Σ`. The discovery set can only shrink.
- *K.μ~ on `d_q`*: the reorder holds `Σ.L` fixed (F-PRES/F-INERT), so every motion of the discovery set comes through the image (D-ABSORB), and F-IMG-SWING moves the image only when `W` is *not* fixed setwise by `π` — when `π⁻¹(W) ∩ dom(Σ.M(d_q)) = W ∩ dom(Σ.M(d_q))`, image and discovery set are both invariant. When the image *does* move, the case split follows F-IMG-TAX's two shapes. *Containment image-motion* (`image(W, d_q, Σ) ⊆ image(W, d_q, Σ')` or the reverse): F-IMONO applies in that step — bridged through F-INERT, since K.μ~ preserves `Σ.L` — and `findlinks_V` moves monotonically, exactly as in the K.μ⁺ and K.μ⁻ clauses. *Incomparable image-motion*: the monotone transfer is unavailable — `findlinks` is monotone in its I-argument (F-IMONO) but not order-reflecting — and non-monotonicity is established directly by the Worked illustration's reorder clause: an injective lateral swing at fixed cardinality — neither `⊆` nor `⊇`, with no link created or retracted — refutes monotonicity, alongside a cardinality-changing swing confirming that discovery-set cardinality is not pinned even when image cardinality is.
- *Transitions not on `d_q`*: every such transition's arrangement frame (ASN-0047) gives `Σ'.M(d_q) = Σ.M(d_q)`, so `image(W, d_q, Σ) = image(W, d_q, Σ')`; the result changes only if `K.λ` adds a matching link (F-LAMBDA).

**D-CWP (ContractionStabilityWP).** *Fix a K.μ⁻ contraction `Σ → Σ'` on the query document `d_q` with retention set `R = ⋃ {[S, 1, …, 1, k] : S ∈ {s_C, s_L} ∧ 1 ≤ k ≤ n'_S}` (ASN-0047), so that `enabled(K.μ⁻[d_q, R])` holds — the retention counts `n'_S ∈ {0, …, n_S}` are admissible and at least one subspace strictly contracts — and `Σ'.M(d_q) = Σ.M(d_q) ↾ R`. The post-state image reduces to a pre-state quantity (the **bridge**):*

> `image(W, d_q, Σ') = {Σ.M(d_q)(v) : v ∈ W ∩ R} ≡ I_R`,

*since `dom(Σ'.M(d_q)) = R` (D-SEQ★, ASN-0047, gives `R ⊆ dom(Σ.M(d_q))`, so `Σ.M(d_q) ↾ R` has domain exactly `R`) and `Σ'.M(d_q)(v) = Σ.M(d_q)(v)` on `v ∈ R` (retained-domain agreement). Write `Δ ≡ image(W, d_q, Σ) ∖ I_R` for the I-addresses the contraction drops from the queried region (well-defined, with `image(W, d_q, Σ) = I_R ∪ Δ`, by F-IMG-CONTR). The contraction leaves the discovery set fixed*

> `findlinks_V(W, d_q, Σ') = findlinks_V(W, d_q, Σ)`  *iff*  `findlinks(Δ, Σ) ⊆ findlinks(I_R, Σ)`

*— i.e. iff every link reaching a dropped I-address also reaches an I-address retained within the queried region (`I_R`). Both `I_R = {Σ.M(d_q)(v) : v ∈ W ∩ R}` and `Δ = image(W, d_q, Σ) ∖ {Σ.M(d_q)(v) : v ∈ W ∩ R}` are functions of the pre-state `Σ` and the retention set `R` alone — the bridge eliminates every post-state quantity — so the biconditional is a genuine precondition on `(Σ, R)`, evaluable before the step.*

*Derivation. K.μ⁻ preserves `Σ.L` (F-PRES), so `findlinks(I, Σ') = findlinks(I, Σ)` for every fixed `I` (F-INERT); in particular `findlinks_V(W, d_q, Σ') = findlinks(image(W, d_q, Σ'), Σ) = findlinks(I_R, Σ)` by the bridge — the comprehension may be evaluated at `Σ`. Expanding the pre-state set through `image(W, d_q, Σ) = I_R ∪ Δ` and applying F-UDIST (no disjointness required): `findlinks_V(W, d_q, Σ) = findlinks(I_R ∪ Δ, Σ) = findlinks(I_R, Σ) ∪ findlinks(Δ, Σ) = findlinks_V(W, d_q, Σ') ∪ findlinks(Δ, Σ)`. Writing `A = findlinks_V(W, d_q, Σ') = findlinks(I_R, Σ)` and `B = findlinks(Δ, Σ)`, this reads `findlinks_V(W, d_q, Σ) = A ∪ B`, so the stability equation `A = findlinks_V(W, d_q, Σ)` becomes `A = A ∪ B`, which holds iff `B ⊆ A` — exactly `findlinks(Δ, Σ) ⊆ findlinks(I_R, Σ)`. This is the weakest precondition for discovery-anchored stability under this single K.μ⁻ step — the discovery analog, on the contraction side, of ASN-0098's LP12a (ContractionDiscoverabilityWP).*

*Boundary case `R = ∅`. Full clearance of a non-empty `d_q` is a valid K.μ⁻ (every retention count `n'_S = 0`; strict contraction holds because `d_q` is non-empty, some `n_S ≥ 1`). Here `I_R = {Σ.M(d_q)(v) : v ∈ W ∩ ∅} = ∅`, so `Δ = image(W, d_q, Σ)` and the stability condition collapses to `findlinks(image(W, d_q, Σ), Σ) ⊆ findlinks(∅, Σ) = ∅` (F-FIND), i.e. `findlinks_V(W, d_q, Σ) = ∅`: full clearance preserves the discovery set exactly when it was already empty. The uniform characterization over arbitrary transitions and regions remains open (Q3).*

**D-ZERO (PresentNotHistorical).** *A discovery zero `findlinks_V(W, d_q, Σ) = ∅` asserts that no link in `dom(Σ.L)` is presently reachable through the queried region — no link's coverage meets `image(W, d_q, Σ)`. In general this is weaker than unreachability from `d_q`'s arrangement at large; that document-level reading is exactly the full-region specialization `W ⊇ dom(Σ.M(d_q))` (F-FULL). It does not assert historical absence. A link whose endpoints have left `d_q`'s consulted arrangement merely ceases to be reachable through it — the region's image drops the departed I-addresses (F-IMG-CONTR) — so it leaves the discovery set while remaining a permanent member of the store (L12).*

*By contrast, an existence zero against fixed `I` certifies historical absence. Take any path `Σ₀ →* Σ` from the initial state (`L₀ = ∅`, ASN-0047). By E-CONS, the difference `findlinks(I, Σ) ∖ findlinks(I, Σ₀)` consists of exactly those links created on that path whose stored value matches `I`; `findlinks(I, Σ) = ∅` makes the difference empty, so no link matching `I` was created at any state along the path, and `findlinks(I, Σ₀) = ∅` (immediate from `L₀ = ∅`) rules out a pre-existing match. No link satisfying `I` was ever created.*

## Worked illustration

Take a single document `d` with three text positions `v_1, v_2, v_3` mapping to content addresses `a_1, a_2, a_3` respectively, and two stored links, each a conforming triple (L3) with a non-empty type endset at slot 3: `L_1 = ({a_1}, {a_3}, Θ)` and `L_2 = ({a_2}, {a_3}, Θ)`, with type endset `Θ = {a_θ}`, where we pin `a_θ = [d, 0, s_L, 1]` — the first-emission shape of `d`'s link sub-allocator `A_L(d)` (FirstEmission, ASN-0093); whether anything is stored at `a_θ` is immaterial, since a type endset may reference an unallocated address (ghost types, L9, ASN-0043).

*Coverage of the endset shorthand.* Each singleton `{x}` here abbreviates the canonical unit-depth endset `{(x, δ(1, #x))}`, whose coverage is the entire subtree `coverage({x}) = {t ∈ T : x ≼ t} = subtree(x)` (PrefixSpanCoverage, ASN-0043) — never the bare singleton `{x}`, which no endset can denote. The empty endset `∅`, by contrast, denotes nothing: `coverage(∅) = ∅`, the union over no spans (Coverage, ASN-0043), so an empty slot can never witness F-MATCH's existential. The slot reductions below rest on one structural premise: the generating addresses are pairwise prefix-incomparable. This holds because `a_1, a_2, a_3` are distinct content addresses of the same document `d`, hence siblings on `d`'s content chain `A_C(d)` (ChainMembershipForOrigin, ASN-0093) and pairwise prefix-incomparable (T10a.2, ASN-0034); and `a_θ = [d, 0, s_L, 1]` is prefix-incomparable with each `a_i`, in two cited steps. First the lengths agree: each `a_i`, as a sibling on `A_C(d)`, has the form `[d, 0, s_C, kᵢ]` of length `#d + 3` — FirstEmission (ASN-0093) gives the chain's first element `#E = 2` (length `#d + 3`) and the sibling step `inc(·, 0)` preserves length (TA5(c), ASN-0034) — and `#a_θ = #d + 3` by inspection. Then: distinctness `a_θ ≠ a_i` is T7 (SubspaceDisjointness, ASN-0034), both being element-level (`zeros = 3`) with `E(a_θ)₁ = s_L ≠ s_C = E(a_i)₁`; and a prefix relation between distinct tumblers is proper, forcing a length gap (`p ≺ q ⟹ #p < #q`, Prefix, ASN-0034), which the equal lengths rule out. So `a_θ ⋠ a_i ∧ a_i ⋠ a_θ` — a fortiori `a_θ ∉ {a_1, a_2, a_3}`. Under this premise each subtree meets the query I-set only at its own generating address: for any `I ⊆ {a_1, a_2, a_3}`, `coverage({a_i}) ∩ I = subtree(a_i) ∩ I = {a_i} ∩ I` (no other listed address lies under `a_i`), and `subtree(a_θ) ∩ {a_1, a_2, a_3} = ∅`.

*Phase 1.* `W = {v_1, v_2}` yields `image(W, d, Σ) = {a_1, a_2}`.

*Phase 2.* `findlinks({a_1, a_2}, Σ)` — both links match via slot 1, every slot intersected against the full query I-set `{a_1, a_2}`. For `L_1`, `coverage(e₁) ∩ {a_1, a_2} = subtree(a_1) ∩ {a_1, a_2} = {a_1} ≠ ∅` (since `a_1 ⋠ a_2`); for `L_2`, `coverage(e₁) ∩ {a_1, a_2} = subtree(a_2) ∩ {a_1, a_2} = {a_2} ≠ ∅` (since `a_2 ⋠ a_1`). The other slots do not fire: both links' slot 2 is `{a_3}`, so `coverage(e₂) ∩ {a_1, a_2} = subtree(a_3) ∩ {a_1, a_2} = ∅` (`a_3 ⋠ a_1`, `a_3 ⋠ a_2`), and the type slot gives `coverage(e₃) ∩ {a_1, a_2} = subtree(a_θ) ∩ {a_1, a_2} = ∅` (`a_θ` prefix-incomparable with each). The match is carried entirely by slot 1, and the result is `{L_1, L_2}`.

*Stability under K.α* — allocating fresh content `a_4` adds nothing to `image(W, d, Σ)` (V-positions in `W` are unchanged); F-INERT carries the result. A bare K.α is not by itself a valid composite — J0 (ValidComposite★, ASN-0047) requires every freshly allocated I-address to be arranged in the composite's post-state — so read the event as the K.α atomic step of a J0-satisfying composite, say K.α then K.μ⁺ appending a fresh fourth position `v_4 ↦ a_4` with K.ρ recording `(a_4, d)` (J1★): `v_4 ∉ W`, and any J0-discharging placement outside `W` leaves the conclusion intact. ✓

*Stability under K.μ⁻* — with `v_1 = [1,1], v_2 = [1,2], v_3 = [1,3]`, K.μ⁻ retains an initial segment `{[s_C, 1, …, 1, k] : 1 ≤ k ≤ n'_{s_C}}` of the sequential positions (D-SEQ★), never a mid-sequence position. Retaining `n'_{s_C} = 1` keeps only the prefix `{v_1}`, removing both `v_2` and `v_3`. Then `W ∩ dom(Σ'.M(d)) = {v_1, v_2} ∩ {v_1} = {v_1}`, so `image(W, d, Σ')` shrinks to `{a_1}` and `findlinks_V(W, d, Σ')` shrinks to `{L_1}`. In D-CWP's terms this is the failing branch, decided pre-step: the retention set is `R = {v_1}`, so `W ∩ R = {v_1}` gives `I_R = {a_1}` and `Δ = {a_2}`, and `findlinks(Δ, Σ) = {L_2} ⊄ {L_1} = findlinks(I_R, Σ)` — `L_2` reaches the dropped `a_2` (slot 1) and no retained in-region address — so the wp predicts exactly the observed drop `{L_1, L_2} ↦ {L_1}`. ✓ D-NONMONO contraction clause; D-CWP failing branch.

*Stable contraction (D-CWP satisfied branch)* — The drop just shown is not forced: D-CWP's stability condition is realizable with a genuine drop, `Δ ≠ ∅`. Continue from the four-position state arranged by the *Stability under K.α* bullet's composite — `dom(M(d)) = {v_1, v_2, v_3, v_4}` with `v_4 = [1, 4] ↦ a_4`, the store unchanged, holding `L_1` and `L_2` — written `Σ₊` here. By K.α's binding precondition (ASN-0093), `a_4` is the next sibling emission on `d`'s content chain `A_C(d)`, so it joins `a_1, a_2, a_3` as a fourth sibling of the form `[d, 0, s_C, k_4]` and length `#d + 3` (FirstEmission and TA5(c), as in the structural premise), and the structural premise extends verbatim: `a_4` is pairwise prefix-incomparable with `a_1, a_2, a_3` (T10a.2, ASN-0034) and with `a_θ` (the same two cited steps — T7 distinctness, then equal lengths against Prefix's length gap). Hence no slot of either stored link reaches `a_4` — every slot coverage is a subtree generated at some `x ∈ {a_1, a_2, a_3, a_θ}` with `x ⋠ a_4` — and `findlinks({a_4}, Σ₊) = ∅`. Now query `W' = {v_1, v_4}` and contract with `n'_{s_C} = 3` (admissible — strict in the content subspace, `3 < 4`; post-state `Σ₊'`): the retention set is `R = {v_1, v_2, v_3}`, so the contraction drops exactly `v_4`. D-CWP's pre-state quantities: `image(W', d, Σ₊) = {a_1, a_4}`; `W' ∩ R = {v_1}` gives `I_R = {a_1}`; so `Δ = {a_1, a_4} ∖ {a_1} = {a_4}` — non-empty, a genuine in-region drop. The stability condition holds: `findlinks(Δ, Σ₊) = findlinks({a_4}, Σ₊) = ∅ ⊆ findlinks(I_R, Σ₊)`, the inclusion unconditional from the empty left side. Direct computation confirms the wp's verdict: pre-step, `findlinks_V(W', d, Σ₊) = findlinks({a_1, a_4}, Σ₊) = {L_1}` — `L_1` fires via slot 1 (`subtree(a_1) ∩ {a_1, a_4} = {a_1} ≠ ∅`), `L_2` is excluded (its coverages `subtree(a_2)`, `subtree(a_3)`, `subtree(a_θ)` all miss `{a_1, a_4}`) — and post-step the bridge gives `image(W', d, Σ₊') = I_R = {a_1}`, so `findlinks_V(W', d, Σ₊') = findlinks({a_1}, Σ₊) = {L_1}` (F-INERT; `L_1` again via slot 1, `L_2` missing the subset `{a_1}` a fortiori). The discovery set holds fixed across a contraction that drops an in-region I-address — separating the wp from the cruder sufficient condition "no in-region I-address was dropped". Stability survives even a *link-bearing* drop (`findlinks(Δ, Σ) ≠ ∅`) when every link reaching a dropped address re-witnesses through a retained one. In a variant history whose store holds `L_1` and `L_4 = ({a_1}, {a_2}, Θ)` — a conforming triple reaching both `a_1` (slot 1) and `a_2` (slot 2) — with `L_2` never created, write `Σ_w` for the three-position pre-state (`v_1 ↦ a_1, v_2 ↦ a_2, v_3 ↦ a_3`) and apply the contraction shape of the *Stability under K.μ⁻* bullet (`W = {v_1, v_2}`, `n'_{s_C} = 1`, `R = {v_1}`, so again `I_R = {a_1}`, `Δ = {a_2}`). Now `findlinks(Δ, Σ_w) = {L_4} ≠ ∅` — the dropped address is link-bearing: `L_4`'s slot 2 fires (`subtree(a_2) ∩ {a_2} = {a_2} ≠ ∅`) while `L_1` misses `{a_2}` at every slot — yet `L_4` re-witnesses at the retained `a_1` through slot 1 (`subtree(a_1) ∩ {a_1} = {a_1} ≠ ∅`), so `findlinks(I_R, Σ_w) = {L_1, L_4}` and the condition `{L_4} ⊆ {L_1, L_4}` holds. Direct computation again agrees: `findlinks_V(W, d, Σ_w) = findlinks({a_1, a_2}, Σ_w) = {L_1, L_4}` pre-step (`L_1` via slot 1 at `a_1`; `L_4` via slot 1 at `a_1` and slot 2 at `a_2`), and `findlinks_V(W, d, Σ_w') = findlinks({a_1}, Σ_w) = {L_1, L_4}` post-step (bridge, then F-INERT). The same contraction shape that dropped `L_2` above leaves `L_4` discoverable — D-CWP's clause "also reaches an I-address retained within the queried region" doing exactly its work. ✓ D-CWP satisfied branch, at both `Δ`-shapes: link-free drop and re-witnessed drop.

*Rise under K.μ⁺ (store-fixed)* — Continue from the contracted state of the *Stability under K.μ⁻* bullet, naming it `Σ₁`: `dom(Σ₁.M(d)) = {v_1}`, so `image(W, d, Σ₁) = {a_1}` and `findlinks_V(W, d, Σ₁) = {L_1}`. The pre-existing link `L_2 = ({a_2}, {a_3}, Θ)` still resides in `dom(Σ₁.L)` (K.μ⁻ preserved the store) and its from-endpoint `a_2` still resides in `dom(Σ₁.C)` (content is permanent, P0, ASN-0047) — but `a_2 ∉ image(W, d, Σ₁)`, so `L_2 ∉ findlinks_V(W, d, Σ₁)`: the link and its target both persist, yet `L_2` is presently undiscoverable through `d`. Now apply K.μ⁺ adding `v_2 ↦ a_2`, a valid content-subspace extension restoring the contiguous segment `{v_1, v_2}` (D-SEQ★) whose image `a_2 ∈ dom(Σ₁.C)` discharges referential integrity (S3★); call the result `Σ₂`. As a single-step composite this extension also owes J1★ (ValidComposite★, ASN-0047) — `a_2` is new to the content-subspace range of `M(d)` at `Σ₁`, so `(a_2, d) ∈ Σ₂.R` is required — and the obligation is met by the standing record: P4★ put `(a_2, d)` into `R` at the composite boundary where `v_2 ↦ a_2` was first arranged, and P2 carries it across the contraction (whose frame is `R' = R`). The link store is untouched — `Σ₂.L = Σ₁.L` (F-PRES) — so no link is created. Yet `image(W, d, Σ₂) = {a_1, a_2}`, and `L_2` re-enters via slot 1 (`coverage(e₁) ∩ {a_1, a_2} = subtree(a_2) ∩ {a_1, a_2} = {a_2} ≠ ∅`): `findlinks_V(W, d, Σ₂) = {L_1, L_2}`. Thus `L_2 ∉ findlinks_V(W, d, Σ₁)` while `L_2 ∈ findlinks_V(W, d, Σ₂)` — the discovery set rises under a pure arrangement extension, with no link created or modified. ✓ D-NONMONO extension clause. This store-fixed rise is precisely the motion existence anchoring cannot exhibit: against fixed `I`, only K.λ ever changes `findlinks(I, ·)` (F-LAMBDA, E-CONS), so the existence-anchored set never rises without a creation — here discovery rises on arrangement alone.

*Swing under K.μ~ (store-fixed)* — Return to the initial state `Σ` (all three positions live: `v_1 ↦ a_1, v_2 ↦ a_2, v_3 ↦ a_3`) and narrow the query to the single position `W₀ = {v_1}`. Then `image(W₀, d, Σ) = {a_1}` and `findlinks_V(W₀, d, Σ) = {L_1}`: only `L_1` matches, via slot 1 (`coverage(e₁) ∩ {a_1} = subtree(a_1) ∩ {a_1} = {a_1} ≠ ∅`), while `L_2`'s slot 1 misses (`subtree(a_2) ∩ {a_1} = ∅`, since `a_2 ⋠ a_1`) and `L_2`'s slot-2/slot-3 coverages `subtree(a_3)`, `subtree(a_θ)` miss `{a_1}` as well. Now apply the transposition reorder `π = (v_1 v_2)` — admissible as a K.μ~ on `d` by F-IMG-TAX's witness-admissibility construction, whose pinned shape (initial-segment positions `[1, k]`, images emissions of `A_C(d)`) is exactly this state. The bijection equation `Σ'.M(d)(π(u)) = Σ.M(d)(u)` gives `Σ'.M(d) : v_1 ↦ a_2, v_2 ↦ a_1, v_3 ↦ a_3`. The link store is untouched — `Σ'.L = Σ.L` (F-PRES) — so no link is created or retracted. But `W₀ = {v_1}` is not fixed setwise by `π` (`π⁻¹({v_1}) = {v_2}`), so the image swings: `image(W₀, d, Σ') = {a_2}`, and now only `L_2` matches (slot 1: `subtree(a_2) ∩ {a_2} = {a_2} ≠ ∅`); `L_1` no longer matches, since its slots — anchored at `a_1`, `a_3`, `a_θ` — all miss `{a_2}`. Hence `findlinks_V(W₀, d, Σ') = {L_2}`. The discovery set moves `{L_1} ↦ {L_2}`: a lateral swing — neither `{L_1} ⊆ {L_2}` nor the reverse — at the same cardinality (the arrangement is injective, so F-IMG-SWING permits only membership change), with no link created or retracted. Each displaced image member is here the *sole* in-region witness for its link, which is why the swing reaches the link set rather than being absorbed by the multi-slot existential (D-ABSORB). Had the query stayed at `W = {v_1, v_2}` — fixed setwise by `π` (`π⁻¹({v_1, v_2}) = {v_1, v_2}`) — both image and discovery set would be invariant; the swing requires a region the reorder does not preserve. *Cardinality-changing variant.* The lateral swing above moved the discovery set at fixed cardinality, but the cardinality is not forced. Admit one further link `L_2' = ({a_2}, ∅, Θ)` (a conforming triple — empty to-endset admissible, `Θ = {a_θ}` mandatory) so that `a_2` is reached by two links where `a_1` is reached by one. `L_2'` leaves the pre-state result untouched — its from-slot misses `{a_1}` (`coverage(e₁) ∩ {a_1} = subtree(a_2) ∩ {a_1} = ∅`, since `a_2 ⋠ a_1`), its empty to-endset has `coverage(∅) = ∅` so slot 2 cannot fire, and its type slot misses as well (`coverage(e₃) ∩ {a_1} = subtree(a_θ) ∩ {a_1} = ∅`, as established above), so `findlinks_V(W₀, d, Σ) = {L_1}` still — while at the post-state both links reaching `a_2` fire (`subtree(a_2) ∩ {a_2} = {a_2} ≠ ∅` for each): `findlinks_V(W₀, d, Σ') = {L_2, L_2'}`. The same transposition reorder now swings `{L_1} ↦ {L_2, L_2'}` — cardinality `1 ↦ 2`, with no link created or retracted by the reorder (`L_2'` was already stored and `Σ'.L = Σ.L`). This needs no content sharing: the arrangement stays injective, so the *image* cardinality remains pinned at 1 (F-IMG-SWING); only the *discovery-set* cardinality moves, and it moves purely because distinct I-addresses match distinct link-counts. ✓ D-NONMONO reorder clause.

*K.λ adding L_3* `= ({a_1}, ∅, Θ)` (a conforming triple; the empty to-endset is admissible, the type slot `Θ = {a_θ} ≠ ∅` is mandatory): F-LAMBDA gives `findlinks({a_1, a_2}, Σ') = {L_1, L_2, L_3}` — the prior result plus the new link's match, which fires via slot 1 (`coverage(e₁) ∩ {a_1, a_2} = subtree(a_1) ∩ {a_1, a_2} = {a_1} ≠ ∅`).

*Existence vs discovery zero.* Suppose K.μ⁻ removes all of `v_1, v_2, v_3` — the `R = ∅` full-clearance contraction of D-CWP's boundary case. Then `image(W, d, Σ') = ∅`, `findlinks_V(W, d, Σ') = ∅` (discovery zero — present absence); and since the pre-state discovery set `findlinks_V(W, d, Σ) = {L_1, L_2}` is non-empty, D-CWP's `R = ∅` stability condition `findlinks_V(W, d, Σ) = ∅` fails — the discovery set is correctly *not* stable, dropping `{L_1, L_2} ↦ ∅`. But `findlinks({a_1, a_2}, Σ') = {L_1, L_2}` (existence non-zero — K.μ⁻ preserves `Σ.L`, so the fixed-`I` comprehension `findlinks({a_1, a_2}, ·)` is unchanged by F-INERT, with per-link coverage permanence by E-INV).

## Properties established

| Claim | Statement | Role |
|-------|-----------|------|
| F-IMG | `image(W, d, Σ) = {Σ.M(d)(v) : v ∈ W ∩ dom(Σ.M(d))}` | Phase 1 primitive |
| F-IMG-MONO | image grows under K.μ⁺/K.μ⁺_L | image stability |
| F-IMG-CONTR | image shrinks under K.μ⁻ | image stability |
| F-IMG-SWING | image may move under K.μ~; index-set cardinality pinned, image cardinality pinned under injectivity | image instability |
| F-IMG-TAX | moved-image shapes: containment vs incomparable; injectivity decides availability | image instability |
| F-MATCH | match predicate (existential over slots) | Phase 2 primitive |
| F-FIND | comprehension primitive `findlinks(I, Σ)` | Phase 2 primitive |
| F-UDIST | `findlinks(I₁ ∪ I₂) = findlinks(I₁) ∪ findlinks(I₂)` for all `I₁, I₂` | Phase 2 algebra |
| F-IMONO | `I' ⊆ I ⟹ findlinks(I') ⊆ findlinks(I)` | Phase 2 algebra (corollary of F-UDIST) |
| F-V | `findlinks_V(W, d, Σ) = findlinks(image(W, d, Σ), Σ)` | two-phase combinator (definition) |
| F-FULL | `W ⊇ dom(Σ.M(d)) ⟹ findlinks_V(W, d, Σ) = {a ∈ dom(Σ.L) : discoverable_from(a, d, Σ)}` | full-region boundary — bridge to ASN-0098 discovery |
| F-VDIST | `findlinks_V(W₁ ∪ W₂, d, Σ) = findlinks_V(W₁, d, Σ) ∪ findlinks_V(W₂, d, Σ)` | composite algebra |
| F-CIL | comprehension over `dom(Σ.L)` with `Σ.L`-only predicate is `Σ.L`-stable | store-fixed-lane keystone |
| F-CIL-perlink | per-link version under per-link value preservation | sub-lemma |
| F-PRES | `V_atomic ∖ {K.λ}` and `K.μ~` preserve `Σ.L` | transition vocabulary |
| F-INERT | preservation ⟹ result invariance | operational consequence |
| F-LAMBDA | `K.λ` increments result by the newly matching singleton (or nothing) | unique store-modifying op |
| E-INV | coverage permanence (per-link, against fixed `I`) | existence anchoring |
| E-MONO | existence-anchored result is `→*`-monotone | existence anchoring |
| E-CONS | path-level set difference is exactly matching creations | existence anchoring |
| D-PRES | editing `d_q` moves the resolved request while `dom(Σ.L)` is fixed | discovery anchoring |
| D-ABSORB | image-motion is necessary but not sufficient for discovery-set motion under store-fixed steps | discovery anchoring |
| D-NONMONO | discovery-anchored result is non-monotone (K-case analysis) | discovery anchoring |
| D-CWP | K.μ⁻ stability iff every dropped-region link also reaches an I-address retained within the queried region (`I_R`) | discovery anchoring (wp) |
| D-ZERO | discovery zero ≠ historical absence | discovery anchoring |

## Open questions

**Q1.** What is the relationship between `findlinks_V` and a content-keyed query that names addresses through `Σ.C` rather than `Σ.M`? Both are content-region queries in a broad sense; this note treats only the arrangement-mediated case.

**Q2.** For the slot-indexed conjunctive query `{a ∈ dom(Σ.L) : (A (i, Jᵢ) ∈ φ :: i ≤ |Σ.L(a)| ∧ coverage(Σ.L(a).eᵢ) ∩ Jᵢ ≠ ∅)}` — each specified slot constrained by its own filter set, unspecified slots unconstrained; this is the matching semantics of Gregory's link retrieval (find-links-from-to-three matches each specified specset against its own endset slot, conjunctively) — which parts of the Phase-2 algebra (F-UDIST, F-IMONO) survive, and in which filter argument?

**Q3.** D-CWP computes the weakest precondition for discovery-anchored stability under a K.μ⁻ contraction on the query document. What is the corresponding weakest precondition for an arbitrary transition `Σ → Σ'` and region `W` — a uniform characterization across the whole K-vocabulary (extension, reorder, and off-document transitions alongside contraction) of when `findlinks_V(W, d, Σ) = findlinks_V(W, d, Σ')`, of which D-CWP is the contraction instance?

**Q4.** How does this foundation compose with ASN-0098's link projection displacement? `image()` and the LP** results both consult `Σ.M`; the natural composition is "project a link through arrangement, then ask if the projection meets a content region" — but the operational composition is not addressed here.
