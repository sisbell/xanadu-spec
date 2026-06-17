> **ASN-0120 · The MAKELINK Operation — Connection Recorded by Content Identity** — condensed claim statements  
> [← Full note](ASN-0120-makelink-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0120 Claim Statements

*Source: ASN-0120-makelink-operation.md (revised 2026-06-08) — Extracted: 2026-06-11*

## Definition — WellFormed

`wf(R, Σ) ≡ (A j : 1 ≤ j ≤ p : d_j ∈ dom(Σ.M) ∧ subspace(u_j) = s_C ∧ #u_j ≥ 2 ∧ (E n_j ≥ 1 : ℓ_j = δ(n_j, #u_j)))`

where `R = ⟨(d₁, σ₁), …, (dₚ, σₚ)⟩` is a finite sequence of V-specs, each naming a source document `d_j` and a V-span `σ_j = (u_j, ℓ_j)`.

## Definition — EndsetResolutionFn

`ρ(R, Σ) = (∪ j : 1 ≤ j ≤ p : { Σ.M(d_j)(v) : v ∈ dom(Σ.M(d_j)) ∧ v ∈ ⟦σ_j⟧ })`

the set of I-addresses to which the named, currently-active V-positions map; defined for `R = ⟨(d₁,σ₁),…,(d_p,σ_p)⟩` satisfying `wf(R, Σ)`.

## Definition — Coverage

`coverage(e) = (∪ (s,ℓ) : (s,ℓ) ∈ e : ⟦(s,ℓ)⟧)`

the union of span denotations for all spans in endset `e ∈ Endset = 𝒫_fin(Span)`.

## Definition — Enabled

`enabled(makelink(d, R₁, R₂, R₃)) ≡ d ∈ dom(Σ.M) ∧ (A i : 1 ≤ i ≤ 3 : wf(R_i, Σ)) ∧ ρ(R₃, Σ) ≠ ∅`

where `wf` and `ρ` are as defined above.

## Definition — DiscoverableFrom

`discoverable_from(a, d', Σ') ⟺ (E i : 1 ≤ i ≤ 3 : coverage(Σ'.L(a).eᵢ) ∩ ran(Σ'.M(d')) ≠ ∅)`

defined for `d' ∈ dom(Σ.M)`.

---

## ML0 — IdentityAllocation (POST, ensures)

The link's identity is a fresh (`a ∉ dom(Σ.L)`), permanent (never removed, never reused — GlobalUniqueness, T8), value-fixed (L12) link-subspace address allocated by `A_L(d)` under home `d`, with `home(a) = d`.

## ML1 — EndsetResolution (POST, ensures)

Under precondition `wf(R,Σ) ≡ (A j : 1 ≤ j ≤ p : d_j ∈ dom(Σ.M) ∧ subspace(u_j) = s_C ∧ #u_j ≥ 2 ∧ (E n_j ≥ 1 : ℓ_j = δ(n_j, #u_j)))`, each endset argument `R = ⟨(d₁,σ₁),…,(d_p,σ_p)⟩` is recorded as the I-addresses `ρ(R,Σ) = (∪ j : 1 ≤ j ≤ p : {Σ.M(d_j)(v) : v ∈ dom(Σ.M(d_j)) ∧ v ∈ ⟦σ_j⟧}) ⊆ dom(Σ.C)`, read through the source arrangements at creation; the stored endset has canonical spans rooted in `ρ(R,Σ)` and satisfies the recovery equation `coverage(e_j) ∩ F = ρ(R_j,Σ)`, equivalently `coverage(e_j) = (∪ a : a ∈ ρ(R_j,Σ) : {t : a ≼ t})`; consequently `coverage(e_j) ∩ dom(Σ.C) = ρ(R_j,Σ)` and `e_j` is tight at Σ (ASN-0098), so by LP19a the content trace is stable at all later states; the boundary `ρ(R_j,Σ) = ∅` is admitted for the non-type slots, with `e_j = ∅` the unique admissible record.

## ML2 — RepresentationIndependence (LEMMA, lemma)

The stored endset's coverage is pinned extensionally by ML1's recovery equation — all admissible records are coverage-equal — and the span decomposition is the only residual freedom; the decomposition remains observable via endset membership (L5) and value equality (L6), but the observables this ASN's claims consult — projection (LP21), type matching (L8), discoverability (LP12) — are functions of coverage alone and cannot distinguish coverage-equal records, which is why the postcondition pins coverage and leaves decomposition free.

## ML3 — UniformResolution (INV, predicate)

From, to, and type arguments are resolved by one procedure with no slot privileged at the V→I conversion step.

## ML4 — ResidenceApplicationOrthogonality (INV, predicate)

Home document and endset content are independent; the precondition relates `d` to no `ρ(R_j,Σ)`; a link may home anywhere and point anywhere, connecting two documents without residing in either.

## ML5 — OrderedEndsets (INV, predicate)

The recorded triple is ordered, `(F,G,Θ) ≠ (G,F,Θ)` for `F ≠ G` (L6); the order fixes from/to roles semantically without restricting reachability (discovery is endset-symmetric); the one-sided slot convention (LM 4/48: populate the first slot, leave the second empty) is informative Nelson usage, not enforced — the operation admits both degenerate forms `(∅, e₂, e₃)` and `(e₁, ∅, e₃)`.

## ML6 — TypedRelation (PRE, requires)

Operation precondition `ρ(R₃,Σ) ≠ ∅`, necessary and sufficient for K.λ's `e₃ ≠ ∅` (L3) via the recovery equation (`coverage(e₃) ∩ F = ρ(R₃,Σ)` with `coverage(∅) = ∅`); the third endset, recorded like from/to but matched by address (L8), distinguishes a typed relation from a bare connection; the type resolves to stored content like any other endset (`ρ(R₃,Σ) ⊆ dom(Σ.C)`).

## ML7 — Permanence (INV, predicate)

`(A Σ' → Σ'' : a ∈ dom(Σ'.L) : a ∈ dom(Σ''.L) ∧ Σ''.L(a) = Σ'.L(a))` — the made link is not broken by any editing of the content it connects.

## ML8 — EndsetSurvivability (LEMMA, lemma)

Editing a source document changes `Σ.M` but never the recorded I-addresses, which by S0 denote their original content permanently — the endset reference survives all editing of the content it names (consequence of ML7 ∧ ML1).

## ML9 — DiscoverabilityDecoupledFromResidence (LEMMA, lemma)

`wp(makelink, discoverable_from(a, d', ·)) ≡ enabled(makelink) ∧ d' ∈ dom(Σ.M) ∧ (E i : ρ(R_i,Σ) ∩ ran(Σ.M(d')) ≠ ∅)`, where `enabled(makelink)` is the operation's enabling precondition (MLop) — beyond enabledness, the home `d` does not appear in the discoverability test; the consequence persists at every later `Σ''` via the stable content trace (LP19a) together with the state-uniform link-store exclusion `coverage(eᵢ) ∩ dom(Σ''.L) = ∅` (LP-Fin Corollary subspace `s_C` versus LP-Sub/L0 subspace `s_L`).

## ML10 — Frame (POST, ensures)

`Σ'.C = Σ.C`; `Σ'.E = Σ.E ∧ Σ'.R = Σ.R` (inherited from the K.λ/K.μ⁺_L frames); `(A d' ≠ d : Σ'.M(d') = Σ.M(d'))`; existing `Σ.L` entries unchanged; every source's content-subspace arrangement is unmodified by being linked into — a source coinciding with the home gains only the link-subspace seating `v_a ↦ a` (`v_a` as determined in MLop).

## MLop — MakelinkOperation (DEF, operation)

Signature `makelink(d, R₁, R₂, R₃)` — home document and three spec-set arguments (from, to, type), a partial operation on reachable states; precondition

> `enabled(makelink(d, R₁, R₂, R₃)) ≡ d ∈ dom(Σ.M) ∧ (A i : 1 ≤ i ≤ 3 : wf(R_i, Σ)) ∧ ρ(R₃, Σ) ≠ ∅`,

where `wf` and `ρ` are as in ML1; effect (the composite `Σ →* Σ'`, a ValidComposite★ of elementary K.λ then K.μ⁺_L):

> `Σ'.L = Σ.L ∪ {a ↦ (e₁, e₂, e₃)}`

with `a` the fresh emission of `A_L(d)` (ML0) and each `e_i` an admissible record of `ρ(R_i, Σ)` per ML1/ML2, plus the home seating

> `Σ'.M(d) = Σ.M(d) ∪ {v_a ↦ a}`

with `v_a = shift(max(V_{s_L}(d)), 1)` when `V_{s_L}(d) ≠ ∅` and `v_a = [s_L, 1]` (first link V-position at the conventional depth `m = 2`) when `V_{s_L}(d) = ∅`; the `a`-branch (store-keyed: K.λ's emit predicate over `{ℓ' : origin(ℓ') = d}`) and the `v_a`-branch (arrangement-keyed: `V_{s_L}(d)`) are independent selectors, diverging at a contracted home — homed links present, `V_{s_L}(d) = ∅` after K.μ⁻ with `n'_{s_L} = 0` — where subsequent emission pairs soundly with the first position; returns `a`; frame per ML10.
