> **ASN-0122 · SHOWRELATIONOF2VERSIONS — Correspondence Between Two Spec-Sets** — condensed claim statements  
> [← Full note](ASN-0122-showrelationof2versions-operation-compare-two-spec-sets.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0122 Claim Statements

*Source: ASN-0122-showrelationof2versions-operation-compare-two-spec-sets.md (revised 2026-06-12) — Extracted: 2026-06-13*

## Definition — PositionInstance

**Definition (Inst — PositionInstance).**
`Inst_Σ = {(d, v) : d ∈ E_doc ∧ v ∈ dom(Σ.M(d))}` — the currently-arranged positions of all documents, each tagged with its document. We write `Inst_C` for the content-subspace instances (`subspace(v) = s_C`) and `Inst_L` for the link-subspace instances; by S3★-aux these two classes exhaust `Inst_Σ`.

---

## Definition — Resolution

**Definition (res — Resolution).**
`res_Σ(d, v) = Σ.M(d)(v)`, total on `Inst_Σ`. By S3★ its value lies in `dom(C)` for content instances and in `dom(L)` for link instances.

---

## Definition — SpecSetRegion

**Definition (Spec-set and region).**
A spec-set is a finite set `ρ = {(d₁, S₁), …, (d_j, S_j)}` with each `d_i ∈ E_doc` and each `S_i` a finite set of T12-well-formed spans. Its *region* at Σ is

`R_Σ(ρ) = (∪ i :: {d_i} × (⟦S_i⟧ ∩ V_{s_C}(d_i)))`

---

## Definition — Correspondence

**Definition (corr — Correspondence).** For finite instance sets `P, Q ⊆ Inst_Σ`:

`corr_Σ(P, Q) = {(p, q) ∈ P × Q : res_Σ(p) = res_Σ(q)}`

`corr` is the *kernel* of the resolution map — the equivalence "resolves to the same address" — intersected with the operand rectangle `P × Q`.

---

## Definition — CorrespondencePair

**Definition (Correspondence pair).**
A pair is `γ = (d₁, u; d₂, w; n)` with `n ≥ 1`, denoting

`⟦γ⟧ = {((d₁, u + k), (d₂, w + k)) : 0 ≤ k < n}`

`γ` is *consistent at Σ* when every denoted element lies in `Inst_Σ × Inst_Σ` with `res_Σ(d₁, u + k) = res_Σ(d₂, w + k)`; it is *confined to (P, Q)* when `⟦γ⟧ ⊆ P × Q`.

---

## Definition — ReportConformance

**Definition (Report; conformance).**
A report is a finite list `Γ = ⟨γ₁, …, γ_r⟩` with denotation `⟦Γ⟧ = (∪ i : 1 ≤ i ≤ r : ⟦γ_i⟧)`. `Γ` *conforms* for `(Σ, P, Q)` when every `γ_i` is consistent and confined, and

`⟦Γ⟧ = corr_Σ(P, Q)`

— soundness (`⊆`, guaranteed per pair by consistency and confinement) and completeness (`⊇`) together. Reports are *equivalent* when their denotations agree.

---

## X0 — RelationWellFormed (LEMMA/lemma)

`corr_Σ(P, Q)` is a finite relation, and membership is decidable from the operand representations and the two arrangement restrictions alone.

*Proof.* Each region is a subset of finitely many finite arrangement domains (S8-fin), so `P × Q` is finite. Membership is tumbler equality (T3), decided by the intrinsic comparison procedure (T2) on the two resolved addresses, which the restrictions `res|P` and `res|Q` supply. ∎

---

## X1 — IdentityBasis (LEMMA/lemma)

`(p, q) ∈ corr_Σ(P, Q) ⟺ res_Σ(p) = res_Σ(q)`; on content instances the shared address `a` lies in `dom(C)` (S3★), and both feet denote the single stored value `C(a)`. Value identity is entailed by membership and never consulted to decide it — the defining comprehension mentions `res` and nothing else. ∎

---

## X2 — CoincidenceExclusion (LEMMA/lemma)

There are reachable states containing instances `p ≠ q` with `C(res p) = C(res q)` and `res p ≠ res q`; every such pair is excluded from `corr`.

*Construction.* Fix documents `d₁, d₂ ∈ E_doc` — pre-existing, or each created by a K.δ step, itself a valid composite (it allocates no content, so J0, J1★, and J1'★ hold vacuously). K.α constrains the address it allocates, never the value stored, so for each `i` run one valid composite in the sense of ValidComposite★: K.α deposits the same `v ∈ Val` at a fresh `a_i`, K.μ⁺ installs `a_i` at a content-subspace position of `M(d_i)` (its precondition `a_i ∈ dom(C)` holds at the intermediate state), and K.ρ records `(a_i, d_i)`. Between the composite's endpoints J0 holds — the freshly allocated address is arranged in the post-state — and J1★/J1'★ hold — the range-new address and the new provenance entry are one and the same pair — so each composite is valid and the final state is reachable. Being distinct allocation events, `a₁ ≠ a₂` by S4 — "regardless of whether `C(a₁) = C(a₂)`" — with GlobalUniqueness (ASN-0034) behind it. The resulting instances resolve to distinct addresses and do not correspond. ∎

---

## X3 — Symmetry (LEMMA/lemma)

`corr_Σ(Q, P) = corr_Σ(P, Q)⁻¹`. Equality of addresses is symmetric, so swapping the operands transposes every member. The *within-pair* order is therefore semantic, not decorative: slot `i` of a reported pair draws from operand `i`, and that convention is exactly what makes the converse statement contentful. ∎

*(continued)* The canonical report of the swapped comparison is the pairwise transpose `(d₂, w; d₁, u; n)` of the original's pairs, re-listed under the transposed sort key. Maximality is orientation-independent — the successor condition is symmetric in the two feet — so transposition is a bijection of canonical pairs, not merely of relation elements. Comparing A with B and comparing B with A yield mirror reports exactly. ∎

---

## X4 — WindowRestriction (LEMMA/lemma)

For `P′ ⊆ P` and `Q′ ⊆ Q`:

`corr_Σ(P′, Q′) = corr_Σ(P, Q) ∩ (P′ × Q′)`

*Proof.* Both sides are comprehensions over the same predicate, restricted to nested rectangles. ∎

---

## X4c — IntervalClipping (LEMMA/lemma)

If both windows are single spans — `P′ = {d₁} × (⟦σ_P⟧ ∩ V_{s_C}(d₁))`, `Q′ = {d₂} × (⟦σ_Q⟧ ∩ V_{s_C}(d₂))` — and `γ = (d₁, u; d₂, w; n)` is a maximal pair of the wider comparison, then `⟦γ⟧ ∩ (P′ × Q′)` is the denotation of at most one pair.

*Proof.* Index `⟦γ⟧` by `k ∈ [0, n)`, the `k`-th element being `((d₁, u + k), (d₂, w + k))`; since `γ` is a maximal pair of the wider comparison it is confined to that comparison's regions (X11), and those regions are content-confined by their `∩ V_{s_C}` clip, so each foot is already a content instance — the `V_{s_C}` clips are already met and this element lies in `P′ × Q′` iff `u + k ∈ ⟦σ_P⟧` and `w + k ∈ ⟦σ_Q⟧`. Put `K_P = {k ∈ [0, n) : u + k ∈ ⟦σ_P⟧}` and `K_Q` symmetrically. The map `k ↦ u + k` is strictly increasing — TS4 (ASN-0034) at the base `k = 0`, TS5 (ASN-0034) at amounts `k₁ < k₂` — and `⟦σ_P⟧` is order-convex (T12(c), ASN-0034). Hence `K_P` is an integer interval: for `k₁ < k < k₂` with `k₁, k₂ ∈ K_P`, monotonicity gives `u + k₁ < u + k < u + k₂` with the extremes in `⟦σ_P⟧`, so convexity puts `u + k ∈ ⟦σ_P⟧`, i.e. `k ∈ K_P`; likewise `K_Q`. The surviving offsets are `K_P ∩ K_Q`, an intersection of two integer intervals and so again an integer interval (possibly empty) — the index-domain reflection of the span intersection-closure S1 (ASN-0053). If empty the clip denotes no pair; otherwise it is a contiguous `[k_lo, k_hi]`, and `⟦γ⟧ ∩ (P′ × Q′) = ⟦(d₁, u + k_lo; d₂, w + k_lo; k_hi − k_lo + 1)⟧`, exactly one consistent, confined pair. ∎

---

## X5 — Locality (LEMMA/lemma)

`corr_Σ(P, Q)` is determined by the two restricted resolution maps `(res_Σ|P, res_Σ|Q)`: any two states agreeing on these maps return the same relation, and neither map can be dropped. The regions are not independent data — `P = dom(res_Σ|P)` and `Q = dom(res_Σ|Q)` — so the four-tuple `(P, Q, res_Σ|P, res_Σ|Q)` is a presentational expansion of the same two maps.

*Proof.* Membership of `(p, q)` in `corr_Σ(P, Q)` is, by the definition of `corr`, the conjunction `p ∈ P ∧ q ∈ Q ∧ res_Σ(p) = res_Σ(q)`. The first two conjuncts read only `P` and `Q`, the domains `dom(res_Σ|P)` and `dom(res_Σ|Q)`. In the third, `p ∈ P` gives `res_Σ(p) = (res_Σ|P)(p)` and `q ∈ Q` gives `res_Σ(q) = (res_Σ|Q)(q)`, so the equality test consults the two restricted maps and nothing else — not `C`, `L`, `E`, `R`, nor `M(d)` at any position outside the regions. Hence if `Σ` and `Σ′` satisfy `res_Σ|P = res_{Σ′}|P` and `res_Σ|Q = res_{Σ′}|Q` — equal as maps, hence equal in domain — every membership test agrees and `corr_Σ(P, Q) = corr_{Σ′}(P, Q)`. The dependence is exact: changing `res|P` at a single instance `p` — toward or away from some `res(q)` — adds or deletes the pair `(p, q)`, so neither map can be dropped. The relation therefore factors precisely through `(res_Σ|P, res_Σ|Q)`. ∎

---

## X-T — TransportLemma (LEMMA/lemma)

Let `Σ →* Σ′`, and let `τ : D → Inst_{Σ′}` and `υ : D′ → Inst_{Σ′}` be injective maps on instance sets `D, D′ ⊆ Inst_Σ` satisfying `res_{Σ′}(τ p) = res_Σ(p)` and `res_{Σ′}(υ q) = res_Σ(q)` on their domains. Then for `P ⊆ D`, `Q ⊆ D′`:

`corr_{Σ′}(τ(P), υ(Q)) = (τ × υ)(corr_Σ(P, Q))`

*Proof.* `res′(τ p) = res′(υ q) ⟺ res p = res q`, by the two preservation equations read in both directions; injectivity carries the rectangle across. ∎

---

## X6 — ChainInvisibility (LEMMA/lemma)

Let content travel: `d⁰` shares into `d¹`, `d¹` into `d²`, and so on — each step a composite installing into the next document addresses drawn from the previous one's range (K.μ⁺ steps; the fork composite J4 is the canonical instance, its order-preserving bijection `φ` satisfying `M′(d_new)(φ(v)) = M(d_op)(v)`). Each step is an X-T map *across documents*. Then:

(a) Each sharing step transports correspondence undiminished: at the post-state, `graph(φ) ⊆ corr(d_op extent, d_new extent)` — every copied position corresponds, at full width.

(b) Steps compose under two premises. *Endpoint persistence:* the conclusion compares the endpoints at the evaluation state, so `d⁰`'s restriction on the transported domain must persist from the chain's first step to that state. *Interleaved intermediate edits:* an edit striking an intermediate `d^i` between its incoming and outgoing steps enters as a position map `π_i` interposed and the composite reads `φ_k ∘ … ∘ φ_{i+1} ∘ π_i ∘ φ_i ∘ … ∘ φ₁`. Each factor is injective and res-preserving, hence so is the composite, on its (possibly shrunken) domain. Under these premises X-T applies to the composite: the endpoints correspond exactly on the transported material, independently of `k`.

(c) By X5 the endpoint relation depends on the endpoint restrictions alone: once material has arrived at `d^k`, every intermediate document may be rearranged, contracted to nothing, or ignored — `corr(d⁰, d^k)` is unmoved.

(d) Kernel transitivity gives the local composition law: `(p, q) ∈ corr(P, Q)` and `(q, r) ∈ corr(Q, R)` imply `(p, r) ∈ corr(P, R)` — pairwise reports compose soundly through a shared middle region. ∎

---

## X7 — EditTransport (LEMMA/lemma)

*(i) Reordering.* K.μ~ on `d₁` carries an admissible bijection `π` with `Σ′.M(d₁)(π(v)) = Σ.M(d₁)(v)` on a fixed domain (K.μ~-FIX). With `τ(d₁, v) = (d₁, π(v))`, X-T gives `corr_{Σ′}(τ(P), Q) = (τ × id)(corr_Σ(P, Q))`. In wp form — for the one-foot case, second foot `q` on a document other than `d₁`: `wp(K.μ~[d₁, π], ((d₁, x), q) ∈ corr) ≡ enabled ∧ ((d₁, π⁻¹(x)), q) ∈ corr`. When both feet lie on `d₁` (self-comparison, X8): `wp(K.μ~[d₁, π], ((d₁, x), (d₁, y)) ∈ corr) ≡ enabled ∧ ((d₁, π⁻¹(x)), (d₁, π⁻¹(y))) ∈ corr`.

*(ii) Contraction.* K.μ⁻ restricts `M(d₁)` to a retained set; survivors keep both their positions and their addresses, so `τ = id` on survivor instances. With `Q` drawn off the edited document, X-T degenerates to `corr_{Σ′} = corr_Σ ∩ (Surv × Q)` — contraction acts on the relation exactly as a window (X4) — and a pair survives iff its `d₁`-foot survives: `wp(K.μ⁻[d₁, n′], (p, q) ∈ corr) ≡ enabled ∧ p surviving ∧ (p, q) ∈ corr`. When both operands draw on `d₁`, both feet are at risk: the relation becomes `corr_{Σ′} = corr_Σ ∩ (Surv × Surv)`, and the wp gains the symmetric conjunct — `enabled ∧ p surviving ∧ q surviving ∧ (p, q) ∈ corr`.

*(iii) Shifting contraction.* ASN-0082's contraction removes a span and closes the gap: survivors relocate by the piecewise map `τ = id` on the left region `L` and `τ = σ` on the right region `R`, and `M′(σ(v)) = M(v)` is the operation's own postcondition (D-SHIFT, D-L), which discharges res-preservation. Injectivity: `id` is injective on `L`; `σ` is injective on `R` (D-BJ, ASN-0082); and `L ∩ Q₃ = ∅` (D-DP(a), ASN-0082) forbids image collision. With injectivity and res-preservation both in hand, X-T applies.

*(iv) Extension.* K.μ⁺ and K.μ⁺_L leave prior mappings unchanged (their agreement clause), so on any regions drawn from the prior domain the relation is literally unchanged; over full extents it can only grow, and every new pair stands on a newly arranged foot.

*Synthesis.* Across the entire edit vocabulary, surviving content is re-addressed, never re-paired: which stored occurrence corresponds to which is conserved under the edit's position map, while the V-coordinates in the report are re-resolved against the new arrangement. ∎

---

## X8 — SelfCorrespondence (LEMMA/lemma)

(a) *The diagonal is forced.* `{(p, p) : p ∈ P ∩ Q} ⊆ corr_Σ(P, Q)`, by reflexivity of equality; comparing a region with itself always yields at least the full diagonal, and for an interval window the diagonal is a single maximal pair of full width.

(b) *Triviality characterized.* `corr_Σ(P, P)` equals the diagonal **iff** `res|P` is injective: if injective, `res p = res q ⟹ p = q`; if not, any witnesses `p ≠ q` with a shared address contribute the off-diagonal pairs `(p, q)` *and* `(q, p)` (X3).

(c) *Windows as detector.* For disjoint windows `P ∩ Q = ∅` drawn from one document, the diagonal is empty and `corr_Σ(P, Q) ≠ ∅ ⟺ ran(res|P) ∩ ran(res|Q) ≠ ∅` — the comparison becomes a pure detector of content shared between the two regions, returning the exact pairs. ∎

---

## X9 — SubspaceVacuity (LEMMA/lemma)

Over unrestricted instances, `corr` decomposes as the content-subspace relation, disjointly unioned with the forced diagonal `{((d, v), (d, v)) : (d, v) ∈ P ∩ Q ∩ Inst_L}`. For any pair with a link-instance foot the predicate `res p = res q` reduces to instance equality `p = q` — so the diagonal is determined by `P ∩ Q` alone, with the resolution map never consulted.

Sub-arguments establishing this decomposition:

- *Cross-document link instances never correspond.* A link-subspace position arranges one of the document's own links: CL-OWN gives `origin(M(d)(v)) = d`. If `(d₁, v₁)` and `(d₂, v₂)` are link instances sharing address `ℓ`, then `d₁ = origin(ℓ) = d₂` — `origin` is a projection of the address itself (S7), hence single-valued — so the documents coincide.

- *A content instance never corresponds to a link instance.* The first resolves into `dom(C)`, the second into `dom(L)` (S3★), and the stores are disjoint (SD/L14): one address cannot inhabit both.

- *Same-document link instances correspond only to themselves.* Within one document the link-subspace restriction of `M(d)` is injective (CL-UNIQ), so a shared address forces `v₁ = v₂`. ∎

---

## X10 — PairSemantics (LEMMA/lemma)

Let `γ = (d₁, u; d₂, w; n)` be consistent. Then:

(a) *Equal extent, one width.* Both feet sets `{u + k : 0 ≤ k < n}` and `{w + k : 0 ≤ k < n}` have cardinality exactly `n`: distinct offsets land on distinct positions, because for `0 ≤ k₁ < k₂ < n` we have `u + k₁ < u + k₂` — when `k₁ = 0` by TS4 (`u < shift(u, k₂)`), and when `k₁ ≥ 1` by TS5 (ShiftAmountMonotonicity, ASN-0034), whose statement at amounts `k₁ < k₂` is `shift(u, k₁) < shift(u, k₂)` directly. A single width serves both sides structurally; there is no second width to disagree.

(b) *Offset alignment.* The `k`-th position of the first span corresponds to the `k`-th position of the second, for each `k`: within the pair, relative offset is shared. The two starts say *where the shared material sits in each document's current arrangement*; the absolute positions are unrelated to one another and may differ arbitrarily.

(c) *Trace identity.* The pair determines one address sequence `a_k = res(d₁, u + k) = res(d₂, w + k)`. Both sides realize the *same* sequence of stored occurrences in the same order — hence, by X1, the same value sequence. A reported pair asserts exactly this: these two spans are arrangements of the same content occurrences, in the same order. It asserts nothing else — nothing about neighbouring positions, nothing about how the sharing arose, nothing about any other state component. ∎

---

## X11 — CanonicalReport (LEMMA/lemma)

Define the successor of a relation element by `succ((d₁, u), (d₂, w)) = ((d₁, u + 1), (d₂, w + 1))`. On `corr_Σ(P, Q)`:

(a) every element has at most one successor — `succ` is single-valued, so `succ(e)` names one element, whether or not it lies in the relation — and at most one predecessor *within the relation*, by shift injectivity (TS2) per coordinate: two elements sharing a successor agree after a unit shift on each foot, so they agree on each foot and coincide, TS2's equal-depth precondition holding because two content V-positions of one document share a depth (S8-depth);

(b) no chain cycles, since feet strictly increase (TS4).

Hence the relation partitions uniquely into maximal succ-chains; each chain is the denotation of exactly one consistent, confined pair — its *maximal pair*; and the *canonical report* `CANON(Σ, P, Q)` — the maximal pairs listed in strictly increasing lexicographic order of (first foot, second foot), instances ordered by T1 on document then position — exists and is unique. *Uniqueness of the partition* is the standard maximal-run argument (S8, ASN-0036; M12a, ASN-0058), here applied to the succ-relation on `corr` in place of a single arrangement function: two chains sharing an element coincide by unique forward and backward extension. *Strictness of the order:* distinct maximal pairs sharing both starts would share their first element and hence coincide; sharing only the first start happens exactly under fan-out, and the second key separates. ∎

---

## X12 — COMPARE (OP/spec)

**X12 (COMPARE — SHOWRELATIONOF2VERSIONS).**

- *Operands:* spec-sets `ρ₁, ρ₂`. Each names one document — the two-version case — or several; `ρ₁` and `ρ₂` may name the same document, with equal, overlapping, or disjoint windows.
- *Precondition:* every named `d ∈ E_doc`; every span T12-well-formed; every span a content-subspace span (`subspace(start) = s_C`).
- *Result:* a report `Γ` for `(Σ, R_Σ(ρ₁), R_Σ(ρ₂))`; the *reference result* is the canonical report `CANON(Σ, R_Σ(ρ₁), R_Σ(ρ₂))` of X11.
- *Binding postconditions — required of every conforming implementation:*
  - (R1) *soundness* — every listed pair is consistent at Σ and confined to the regions;
  - (R2) *completeness* — `⟦Γ⟧ ⊇ corr_Σ(R_Σ(ρ₁), R_Σ(ρ₂))`; jointly with R1, `⟦Γ⟧ = corr`;
  - (R3) *deterministic presentation* — the emitted report is a function of `(ρ₁, ρ₂, res_Σ|P, res_Σ|Q)`: X5 guarantees the relation admits this, and the implementation must fix one presentation of it, with no hidden input and no nondeterminism.
- *Reference presentation — defines `CANON`, not required for conformance:*
  - (R4) *canonical form* — the maximal pairs of X11, listed in lexicographic first-operand-major order with the second foot as tie-break, each record carrying (document, start) per side in operand order plus the one shared width. R4 names the unique reference result this specification reasons with; a conforming implementation may emit any presentation satisfying R1–R3 — finer-than-maximal pairs, a different packing of the record — since report equivalence is denotational.
- *Frame:* `Σ′ = Σ`. COMPARE allocates nothing, arranges nothing, links nothing, records nothing. Moreover its value is a function of the operands and the two restricted arrangements alone (X5): it reads neither the values in `C` nor `L`, `E`, `R`, nor any other document's arrangement.
