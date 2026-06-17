> **ASN-0098 · Link Projection Displacement** — condensed claim statements  
> [← Full note](ASN-0098-link-projection-displacement.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0098 Claim Statements

*Source: ASN-0098-link-projection-displacement.md (revised 2026-05-24) — Extracted: 2026-06-03*

## Definition — Coverage

```
coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})
```

Each span `(s, ℓ)` denotes `{t ∈ T : s ≤ t < s ⊕ ℓ}` by T12 of ASN-0034, where `s ⊕ ℓ ∈ T` exists by TA0 because `Pos(ℓ)` and `actionPoint(ℓ) ≤ #s` are well-formedness conditions of the span. Coverage is a purely combinatorial property of the endset's span representation — it does not consult any state component.

## Definition — Project

```
project(e, d, Σ)
  defined when  d ∈ dom(Σ.M)
  ≡             {v ∈ dom(Σ.M(d)) : Σ.M(d)(v) ∈ coverage(e)}
```

For a link `a ∈ dom(Σ.L)` with slot `i ∈ {1, …, |Σ.L(a)|}`:
```
project(a, i, d, Σ) ≡ project(Σ.L(a).eᵢ, d, Σ)
  defined when  a ∈ dom(Σ.L) ∧ d ∈ dom(Σ.M)
```

Degenerate cases:
- `project(∅, d, Σ) = ∅` for every `d ∈ dom(Σ.M), Σ`
- `project(e, d, Σ) = ∅` for every `d ∈ dom(Σ.M), Σ` with `dom(Σ.M(d)) = ∅`

## Definition — DiscoverableFrom

```
discoverable_from(a, d, Σ)
  defined when  a ∈ dom(Σ.L) ∧ d ∈ dom(Σ.M)
  ≡             (E i : 1 ≤ i ≤ |Σ.L(a)| : project(a, i, d, Σ) ≠ ∅)
```

The link is *discoverable* at `Σ` iff there exists some document from which it is discoverable.

## Definition — SubstrateEmittableAddresses

```
F = {a ∈ T : (E d ∈ T, s ∈ {s_C, s_L}, k ≥ 1 :: zeros(d) = 2 ∧ d satisfies T4 ∧ a = [d, 0, s, k])}
```

Every `a ∈ F` has `#a = #d + 3`, `zeros(a) = 3`, and `#E(a) = 2` by direct inspection of the structural form.

## Definition — Tight

```
tight(e, Σ_e)
```

An endset `e` is *tight at state `Σ_e`* iff every span `(s, ℓ) ∈ e` is *canonical* — `ℓ = δ(n, #s)` for some `n ≥ 1`, equivalently `#ℓ = #s` with `ℓ` an ordinal displacement — and satisfies:
```
s ∈ dom(Σ_e.C) ∪ dom(Σ_e.L)  ∧  (A t ∈ F : s ≤ t < s ⊕ ℓ : t ∈ dom(Σ_e.C) ∪ dom(Σ_e.L))
```

Non-canonical spans are unconditionally non-tight at every state. Tightness is a construction discipline, not a structural invariant the system enforces.

---

## LP2 — SlotInvariance (LEMMA, lemma)

For every transition `Σ → Σ'`, every link `a ∈ dom(Σ.L)`, and every slot index `i ∈ {1, …, |Σ.L(a)|}`:
```
a ∈ dom(Σ'.L) ∧ Σ'.L(a).eᵢ = Σ.L(a).eᵢ
```

## Closure schema (★) — ClosureSchema (SCHEMA, lemma)

Let `P(Σ, Σ')` be a finite conjunction of membership-persistence clauses (`x ∈ dom(Σ.X) ⟹ x ∈ dom(Σ'.X)`) and value-preservation clauses (`f(Σ') = f(Σ)`, each accessor `f` well-defined once its accompanying membership clause holds). If the single-step guarantee `Σ → Σ' ⟹ P(Σ, Σ')` holds, then so does its closure:
```
Σ →* Σ' ⟹ P(Σ, Σ')
```

## LP3 — CoverageInvariance (LEMMA, lemma)

For every transition `Σ → Σ'`, every link `a ∈ dom(Σ.L)`, and every slot `i`:
```
a ∈ dom(Σ'.L) ∧ coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)
```

## LP3★ — MultiStepCoverageInvariance (LEMMA, lemma)

For every reachable state sequence `Σ →* Σ'`, every `a ∈ dom(Σ.L)`, and every slot `i`:
```
a ∈ dom(Σ'.L) ∧ coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)
```

Schema (★) applied to LP3, with `P(Σ, Σ') ≡ a ∈ dom(Σ'.L) ∧ coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`.

## Store Monotonicity★ — StoreMonotonicity (LEMMA, lemma)

For every reachable state sequence `Σ →* Σ'`:
```
dom(Σ.C) ⊆ dom(Σ'.C)  ∧  dom(Σ.L) ⊆ dom(Σ'.L)
```

## LP4 — ArrangementSpecificity (LEMMA, lemma)

For every transition `Σ → Σ'`, every endset `e`, and every document `d ∈ dom(Σ.M)` (with `d ∈ dom(Σ'.M)` lifted by M1 of ASN-0093):
```
Σ'.M(d) = Σ.M(d) ⟹ project(e, d, Σ') = project(e, d, Σ)
```

## LP5 — CrossDocumentIndependence (LEMMA, lemma)

Every operation in the K.μ family (K.μ⁺, K.μ⁺_L, K.μ⁻, K.μ~) has frame `(A d' : d' ≠ d : M'(d') = M(d'))` — it modifies at most one document's arrangement per transition. By LP4 applied to each unmodified document:
```
(A d' ∈ dom(Σ.M), d' ≠ d : project(e, d', Σ') = project(e, d', Σ))
```

## LP6 — ContentAllocationInvariance (LEMMA, lemma)

For every K.α transition `Σ → Σ'`, every endset `e`, and every document `d ∈ dom(Σ.M)`:
```
project(e, d, Σ') = project(e, d, Σ)
```

K.α modifies only `Σ.C`; frame `M' = M` and `dom(Σ.M) = dom(Σ'.M)` hold. Both conditions of LP4 are satisfied for every `d`.

## LP7 — LinkAllocationInvariance (LEMMA, lemma)

For every K.λ transition `Σ → Σ'`, every endset `e`, and every document `d ∈ dom(Σ.M)`:
```
project(e, d, Σ') = project(e, d, Σ)
```

K.λ modifies only `Σ.L`; frame `M' = M` and `dom(Σ.M) = dom(Σ'.M)` hold. Both conditions of LP4 are satisfied for every `d`.

## LP8 — DocumentRegistrationInvariance (LEMMA, lemma)

For any document-registration transition `Σ → Σ'` — K.δ in the `Document(e)` case (ASN-0047) — registering a fresh document `d_new` (with `d_new ∉ dom(Σ.M)`, `dom(Σ'.M) = dom(Σ.M) ∪ {d_new}`, `Σ'.M(d_new) = ∅`, and `Σ'.M(d) = Σ.M(d)` for every `d ∈ dom(Σ.M)`) and any endset `e`, both:

(a) Pre-state preservation:
```
(A d ∈ dom(Σ.M) :: project(e, d, Σ') = project(e, d, Σ))
```

(b) Newly-registered emptiness:
```
project(e, d_new, Σ') = ∅
```

## LP9 — ExtensionMonotonicity (LEMMA, lemma)

For every extension transition `Σ → Σ'` operating on document `d` — either K.μ⁺ (content-subspace extension) or K.μ⁺_L (link-subspace extension) — and every endset `e`:
```
project(e, d, Σ) ⊆ project(e, d, Σ')
```

The proof relies on exactly two structural facts about the post-state arrangement:
- (E1) *Strict domain extension:* `dom(Σ'.M(d)) ⊃ dom(Σ.M(d))`
- (E2) *Prior-domain agreement:* `(A v : v ∈ dom(Σ.M(d)) : Σ'.M(d)(v) = Σ.M(d)(v))`

The new V-positions that enter the projection are exactly:
```
project(e, d, Σ') ∖ project(e, d, Σ) = {v ∈ dom(Σ'.M(d)) ∖ dom(Σ.M(d)) : Σ'.M(d)(v) ∈ coverage(e)}
```

## LP10 — ContractionMonotonicity (LEMMA, lemma)

For every K.μ⁻ transition `Σ → Σ'` operating on `d`, and every endset `e`:
```
project(e, d, Σ') ⊆ project(e, d, Σ)
```

K.μ⁻ contracts `Σ.M(d)`: `dom(Σ'.M(d)) ⊂ dom(Σ.M(d))` with agreement on the retained domain.

The V-positions that leave the projection are exactly the arrangement entries removed by the operation:
```
project(e, d, Σ) ∖ project(e, d, Σ') = {v ∈ dom(Σ.M(d)) ∖ dom(Σ'.M(d)) : Σ.M(d)(v) ∈ coverage(e)}
```

*Boundary case — empty arrangement:* When `dom(Σ'.M(d)) = ∅`, `project(e, d, Σ') = ∅` for every endset `e`.

## LP11 — ReorderingBijection (LEMMA, lemma)

For every K.μ~ transition `Σ → Σ'` operating on `d` via the witnessing bijection `π : dom(Σ.M(d)) → dom(Σ'.M(d))`, and every endset `e`:
```
project(e, d, Σ') = π(project(e, d, Σ))
```
and
```
ran(Σ'.M(d)) = ran(Σ.M(d))
```

By K.μ~-FIX, `dom(Σ'.M(d)) = dom(Σ.M(d))`, so π permutes the domain. For any `v ∈ dom(Σ.M(d))`:
```
v ∈ project(e, d, Σ)
  ⟺ Σ.M(d)(v) ∈ coverage(e)               -- definition
  ⟺ Σ'.M(d)(π(v)) ∈ coverage(e)            -- bijection equation
  ⟺ π(v) ∈ project(e, d, Σ')              -- definition (π(v) ∈ dom(Σ'.M(d)))
```

Consequences: `|project(e, d, Σ')| = |project(e, d, Σ)|` and `{Σ'.M(d)(v') : v' ∈ project(e, d, Σ')} = coverage(e) ∩ ran(Σ.M(d))`.

## LP12 — DiscoverabilityCharacterisation (LEMMA, lemma)

For every link `a ∈ dom(Σ.L)`, document `d ∈ dom(Σ.M)`, and state `Σ`:
```
discoverable_from(a, d, Σ) ⟺ (E i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ ran(Σ.M(d)) ≠ ∅)
```

Per-slot biconditional: `project(a, i, d, Σ) ≠ ∅ ⟺ coverage(Σ.L(a).eᵢ) ∩ ran(Σ.M(d)) ≠ ∅`.

## LP12a — ContractionDiscoverabilityWP (LEMMA, lemma)

Fix a K.μ⁻ operation on document `d ∈ dom(Σ.M)` with retention parameters `(n'_{s_C}, n'_{s_L})` admissible under K.μ⁻'s precondition, and let
```
R := ⋃ {[S, 1, ..., 1, k] : S ∈ {s_C, s_L} ∧ 1 ≤ k ≤ n'_S}
```
denote the resulting retention set. For every link `a ∈ dom(Σ.L)`, the weakest precondition on the pre-state `Σ` under which `discoverable_from(a, d, Σ')` holds in the post-state `Σ' = K.μ⁻[d, R](Σ)` is:
```
wp(K.μ⁻[d, R], discoverable_from(a, d, ·))
  ≡ enabled(K.μ⁻[d, R]) ∧ (E i : 1 ≤ i ≤ |Σ.L(a)| : project(a, i, d, Σ) ∩ R ≠ ∅)
```

where `enabled(K.μ⁻[d, R])` is K.μ⁻'s applicability predicate (ASN-0047).

*Boundary case `R = ∅`:* The wp specialises:
```
(E i : project(a, i, d, Σ) ∩ ∅ ≠ ∅) ≡ (E i : ∅ ≠ ∅) ≡ false
```

The projection satisfies `project(a, i, d, Σ') = project(a, i, d, Σ) ∩ R` for each slot `i` under this operation.

## LP12b — ContentCanonicalLinkSubspaceWPFalse (LEMMA, lemma)

For `a ∈ dom(Σ.L)` whose every span is canonical with `s = [d_s, 0, s_C, k_s]`, and any K.μ⁻ retention parameters `n'_{s_C} = 0, n'_{s_L} > 0`, LP12a's wp evaluates to `false`.

The retention set `R = {[s_L, 1, ..., 1, k] : 1 ≤ k ≤ n'_{s_L}} ⊆ V_{s_L}(d)`. For each slot `i`:
```
project(a, i, d, Σ) ⊆ V_{s_C}(d)
```
from which:
```
project(a, i, d, Σ) ∩ R ⊆ V_{s_C}(d) ∩ V_{s_L}(d) = ∅
```

Hence `(E i : project(a, i, d, Σ) ∩ R ≠ ∅)` is false — the wp's pullback conjunct fails for all pre-states.

The argument turns on:
```
coverage(Σ.L(a).eᵢ) ∩ dom(Σ.L) = ∅
```
derived via LP-Fin Corollary at `X = s_C`, combined with the fact that every `dom(Σ.L)` element has subspace identifier `s_L` (L0, ASN-0093) while every F-candidate in the span's interval has subspace identifier `s_C`.

## LP13 — UnconditionalLinkPersistence (LEMMA, lemma)

For every reachable state sequence `Σ →* Σ'` and every link `a ∈ dom(Σ.L)`:
```
a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a)
```

Persistence requires only `a ∈ dom(Σ.L)` and is independent of arrangement state, so a holder can rely on the stored object permanently — whereas discoverability is arrangement-conditional (LP9–LP11) and cannot be assumed from any particular document without further conditions on that document's arrangement.

## LP14 — ProvenanceRecordingInvariance (LEMMA, lemma)

For every K.ρ transition `Σ → Σ'`, every endset `e`, and every document `d ∈ dom(Σ.M)`:
```
project(e, d, Σ') = project(e, d, Σ)
```

K.ρ only adds a pair to `Σ.R` (ASN-0047); frame `M' = M` and `dom(Σ.M) = dom(Σ'.M)` hold. Both conditions of LP4 are satisfied for every `d`.

## LP16 — TransclusionDiscoverability (LEMMA, lemma)

For any link `a ∈ dom(Σ.L)`, slot `i ∈ {1, …, |Σ.L(a)|}`, and documents `d_src, d_new ∈ dom(Σ.M)` at state `Σ`:
```
coverage(Σ.L(a).eᵢ) ∩ ran(Σ.M(d_src)) ∩ ran(Σ.M(d_new)) ≠ ∅
  ⟹  discoverable_from(a, d_src, Σ) ∧ discoverable_from(a, d_new, Σ)
```

Discoverability of a link from `d` is independent of the link's home document `home(a)` and of the origin documents of the coverage I-addresses — it turns solely on `coverage(Σ.L(a).eᵢ) ∩ ran(Σ.M(d))`.

## LP17 — GhostProjection (LEMMA, lemma)

Suppose at state `Σ` no document's arrangement reaches any I-address in `coverage(Σ.L(a).eᵢ)` for any slot `i`:
```
(A d ∈ dom(Σ.M), i : 1 ≤ i ≤ |Σ.L(a)| : coverage(Σ.L(a).eᵢ) ∩ ran(Σ.M(d)) = ∅)
```

Then `project(a, i, d, Σ) = ∅` for every `d, i`. The link is *orphaned*: not discoverable from any document. Nonetheless:
- `a` remains in `dom(Σ.L)` and `Σ.L(a)` is unchanged (by L12, ASN-0043)
- The coverage I-addresses themselves continue to exist in `dom(Σ.C)` (by S0)

## LP18 — Resurrection (LEMMA, lemma)

If `a` is orphaned at `Σ` and a subsequent transition sequence `Σ →* Σ'` introduces an arrangement entry `Σ'.M(d)(v) = a*` for some `d, v, a*` with `a* ∈ coverage(Σ.L(a).eᵢ)`, then `a` is discoverable from `d` at `Σ'`.

Preconditions and chain:
1. `a ∈ dom(Σ.L)` (from orphan premise)
2. Store Monotonicity★ lifts to `a ∈ dom(Σ'.L)` across `Σ →* Σ'`
3. LP3★ gives `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)`, so `a* ∈ coverage(Σ'.L(a).eᵢ)`
4. `v ∈ dom(Σ'.M(d))` and `Σ'.M(d)(v) = a* ∈ coverage(Σ'.L(a).eᵢ)`, so `v ∈ project(a, i, d, Σ')`

The architecture admits arbitrarily many cycles of orphanage and resurrection.

## LP-Sub — SubstrateContainment (LEMMA, lemma)

At every reachable state `Σ`:
```
dom(Σ.C) ∪ dom(Σ.L) ⊆ F
```

Every `a ∈ dom(Σ.C) ∪ dom(Σ.L)` inhabits a sub-allocator chain `A_C(d)` or `A_L(d)` of its origin `d = origin(a)` (ChainMembershipForOrigin, ASN-0093), whose elements FirstEmission and ChainDiscipline (ASN-0093) fix in the structural form `[d, 0, s, k]` with `s ∈ {s_C, s_L}` and `k ≥ 1`; the origin `d` is a T4-valid document tumbler with `zeros(d) = 2` (M0, ASN-0093).

## LP-Fin — IntervalFinitude (LEMMA, lemma)

For every canonical span `(s, ℓ)` — `ℓ = δ(n, #s)` for some `n ≥ 1` — whose start lies in `F` — `s ∈ F`, so `s = [d_0, 0, s', k_s]` for some T4-valid `d_0` with `zeros(d_0) = 2`, subspace `s' ∈ {s_C, s_L}`, chain index `k_s ≥ 1`:
```
(A s, ℓ : s ∈ F ∧ ℓ = δ(n, #s) for some n ≥ 1 : |F ∩ [s, s ⊕ ℓ)| < ∞)
```

Specifically:
```
|F ∩ [s, s ⊕ ℓ)| = n
```

Sub-case A (`z_2 < #d < #d_0`): contributes 0 candidates at each admissible `#d`; Sub-case B (`#d = #d_0`): contributes exactly `n` candidates. Where `z_1 < z_2 ≤ #d_0` are `d_0`'s two zero positions and admissible range is `#d ∈ {z_2 + 1, …, #d_0}`.

## LP-Fin Corollary — CanonicalIntervalCharacterisation (LEMMA, lemma)

For canonical span `(s, ℓ)` with `s = [d_0, 0, X, k_s]` (where `X ∈ {s_C, s_L}`) and `ℓ = δ(n, #s)`:
```
F ∩ [s, s ⊕ ℓ) = {[d_0, 0, X, k] : k_s ≤ k < k_s + n}
```

Every `t ∈ F ∩ [s, s ⊕ ℓ)` satisfies `subspace_I(t) = X` and `origin(t) = d_0`. The interval contains no F-candidates from any chain other than `A_X(d_0)`: by the `#d ≤ #d_0` bound, any cross-document candidate's document prefix is a proper prefix of `d_0` (`#d < #d_0`), excluded by sub-case A's separator argument, while `#d = #d_0` collapses to `d = d_0` by T3; the same-document cross-subspace chain `A_Y(d_0)` with `Y ≠ X` is excluded by sub-case B's subspace-component step (which forces `s'' = X`).

## LP19a — TightFreshness (LEMMA, lemma)

For any endset `e` tight at `Σ_e`, any reachable state sequence `Σ_e →* Σ`, and any K.α (or K.λ) transition `Σ → Σ'` allocating a fresh address `a_new`:
```
a_new ∉ coverage(e)
```

Preconditions used:
- K.α precondition: `a_new ∉ dom(Σ.C) ∪ dom(Σ.L)`, so by Store Monotonicity★ on `Σ_e →* Σ`: `a_new ∉ dom(Σ_e.C) ∪ dom(Σ_e.L)`
- Tightness at `Σ_e` with `a_new ∈ F` (by LP-Sub chain membership): if `a_new ∈ [s, s ⊕ ℓ)` for some span `(s, ℓ) ∈ e`, tightness forces `a_new ∈ dom(Σ_e.C) ∪ dom(Σ_e.L)` — contradiction

## LP19 — TightEndsetBoundaryExclusion (LEMMA, lemma)

Let `e` be an endset tight at `Σ_e`, and let `Σ_e →* Σ_n → Σ_{n+1}` be a reachable transition sequence whose final step is a K.μ⁺ (or K.μ⁺_L) transition operating on document `d`. For every `v_new ∈ dom(Σ_{n+1}.M(d)) ∖ dom(Σ_n.M(d))`, letting `a_new := Σ_{n+1}.M(d)(v_new)`, if `a_new` was freshly allocated by a K.α (or K.λ) step on the prefix `Σ_e →* Σ_n`:
```
v_new ∉ project(e, d, Σ_{n+1})
```

LP19a applied at the K.α/K.λ step on prefix `Σ_e →* Σ_n` yields `a_new ∉ coverage(e)`. Since coverage is a deterministic function of `e`'s spans (per ASN-0043) and `e` is a fixed endset value across the sequence, `a_new ∉ coverage(e)` carries to `Σ_{n+1}`. With `Σ_{n+1}.M(d)(v_new) = a_new ∉ coverage(e)`, the projection definition excludes `v_new`.

## LP20 — RangeConfinement (LEMMA, lemma)

For every endset `e`, document `d`, state `Σ`:
```
{Σ.M(d)(v) : v ∈ project(e, d, Σ)} = coverage(e) ∩ ran(Σ.M(d))
```

*Corollary (store-confinement form)* via S3★ (GeneralizedReferentialIntegrity, ASN-0047):
```
{Σ.M(d)(v) : v ∈ project(e, d, Σ)} ⊆ coverage(e) ∩ (dom(Σ.C) ∪ dom(Σ.L))
```

*Per-subspace partition* via S3★-aux (SubspaceExhaustiveness):
```
{Σ.M(d)(v) : v ∈ project(e, d, Σ) ∧ subspace(v) = s_C} ⊆ coverage(e) ∩ dom(Σ.C)
{Σ.M(d)(v) : v ∈ project(e, d, Σ) ∧ subspace(v) = s_L} ⊆ coverage(e) ∩ dom(Σ.L)
```

The full projection range partitions as:
```
{Σ.M(d)(v) : v ∈ project(e, d, Σ)} = {Σ.M(d)(v) : v ∈ project(e, d, Σ) ∧ subspace(v) = s_C}
                                   ∪ {Σ.M(d)(v) : v ∈ project(e, d, Σ) ∧ subspace(v) = s_L}
```

The union is exhaustive by S3★-aux; the two summands are disjoint because `dom(Σ.C) ∩ dom(Σ.L) = ∅` (SD, ASN-0093).

## LP21 — RepresentationInvariance (LEMMA, lemma)

For any two endsets `e₁, e₂` with `coverage(e₁) = coverage(e₂)`:
```
project(e₁, d, Σ) = project(e₂, d, Σ)
```

The projection depends only on coverage, not on the span decomposition of the endset. Two endsets with the same coverage are interchangeable for projection purposes.
