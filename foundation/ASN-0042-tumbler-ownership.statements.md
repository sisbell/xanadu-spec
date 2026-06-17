> **ASN-0042 · Tumbler Ownership** — condensed claim statements  
> [← Full note](ASN-0042-tumbler-ownership.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0042 Claim Statements

*Source: ASN-0042-tumbler-ownership.md (revised 2026-03-15) — Extracted: 2026-05-30*

## Definition — OwnershipPrefix

`pfx : Π → T` — total mapping assigning each principal its ownership prefix. `pfx(π) ∈ T`, `T4(pfx(π))` for all `π ∈ Π`.

## Definition — OwnershipDomain

`odom(π) = {a ∈ T : pfx(π) ≼ a}`

## Definition — AccountField

*Preconditions:* `a ∈ T` satisfying T4 (HierarchicalParsing) and T4a (SyntacticEquivalence).

`acct(a) = a` when `zeros(a) = 0`; `acct(a) = N(a) ++ [0] ++ U(a)` when `zeros(a) ≥ 1`, where `N(a)` and `U(a)` are the node and user fields extracted by `fields(a)` (T4b UniqueParse), with component-wise access decidable from T3 (CanonicalRepresentation).

*Postconditions:* (a) `acct(a)` is a valid tumbler satisfying T4. (b) `zeros(acct(a)) ≤ 1`. (c) When `zeros(a) ≤ 1`: `acct(a) = a`. (d) When `zeros(a) ≥ 2`: `acct(a)` is a proper prefix of `a` with `zeros(acct(a)) = 1`.

## Definition — EffectiveOwner

`ω_Σ : Σ.B → Π_Σ` with `ω_Σ(a) = π ≡ π ∈ Π_Σ ∧ pfx(π) ≼ a ∧ (A π' ∈ Π_Σ : π' ≠ π ∧ pfx(π') ≼ a ⟹ #pfx(π) > #pfx(π'))`.

*Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`.

## Definition — AllocatedBy

*Signature:* `allocated_by_Σ : Principal × Tumbler → Bool`

*Semantics:* `allocated_by_{Σ'}(π, a)` holds when the baptism procedure, executing on behalf of `π`, produced `a` during the transition yielding `Σ'`.

## Definition — DelegationPredicate

`delegated(Σ, Σ', π, π')` holds iff `Σ → Σ'`, `π ∈ Π_Σ`, `π' ∈ Π_{Σ'} ∖ Π_Σ`, and conditions (i)–(v) hold for `(π, π')` at `Σ`:

- (i) [ancestry] `pfx(π) ≺ pfx(π')`
- (ii) [authorization] `(A π'' ∈ Π_Σ : pfx(π'') ≼ pfx(π') ⟹ #pfx(π'') ≤ #pfx(π))`
- (iii) [structural-tier] `zeros(pfx(π')) ≤ 1`
- (iv) [top-down-order] `¬(E π'' ∈ Π_Σ : pfx(π') ≺ pfx(π''))`
- (v) [fresh-valid] `T4(pfx(π')) ∧ pfx(π') ∉ Σ.B`

The subscript form `delegated_Σ(π, π')` abbreviates `delegated(Σ, Σ', π, π')` for the `Σ'` bound by the surrounding formula. Evaluation-state convention: the delegator prefix `pfx(π)` is read at `Σ`, the delegate prefix `pfx(π')` at `Σ'`.

## Definition — ParentRelation

`R_Σ(π, π')` holds iff `π` is the most-specific covering principal of `pfx(π')` in `Π_Σ` — the unique `π ∈ Π_Σ` with `pfx(π) ≺ pfx(π')` of maximal prefix length. `covers_Σ* = ∪_{m ≥ 0} R_Σ^m`, where `R_Σ^0` is the identity relation on `Π_Σ` and `R_Σ^{m+1} = R_Σ^m ∘ R_Σ`. Equivalently, `covers_Σ*(π, π')` iff `π = π'` or there is a finite chain `π = π^{(0)}, π^{(1)}, ..., π^{(m)} = π'` (`m ≥ 1`) with each consecutive pair `(π^{(j)}, π^{(j+1)})` related by `R_Σ`.

## Definition — Freshness

`Freshness-(v)`: alias for the pair `T4(pfx(π')) ∧ pfx(π') ∉ Σ.B` of condition (v) of the delegation predicate.

---

## O1 — PrefixDetermination (DEF, definition)

`owns(π, a) ≡ pfx(π) ≼ a`, where `≼` is the prefix relation defined by Prefix (PrefixRelation).

*Preconditions:* `π ∈ Π`, `a ∈ T`. (`T4(pfx(π))` holds for every `π ∈ Π` by the `pfx` axiom's postcondition (b), so it is inherited, not assumed.)

*Postconditions:* `owns(π, a)` is a total, decidable predicate on `Π × T`.

## O1a — AccountOwnershipBoundary (INV, predicate)

`(A Σ reachable from Σ₀, π ∈ Π_Σ : zeros(pfx(π)) ≤ 1)`

## O1b — PrefixInjectivity (INV, predicate)

`(A Σ reachable, π₁, π₂ ∈ Π_Σ : pfx(π₁) = pfx(π₂) ⟹ π₁ = π₂)`

## O2 — OwnershipExclusivity (DERIVED, lemma)

`(A Σ reachable, a ∈ Σ.B : (E! π ∈ Π_Σ : ω_Σ(a) = π))`

Equivalently, `ω_Σ : Σ.B → Π_Σ` is a total well-defined function in every reachable state.

*Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`. Reachability is inherited from O4 (invoked in Step 1).

*Postconditions:* `(E! π ∈ Π_Σ : ω_Σ(a) = π)` — exactly one principal satisfies the defining equivalence.

## Covering-chain lemma — PrefixesOfCommonAddressAreComparable (LEMMA, lemma)

`(A x, p, q ∈ T : p ≼ x ∧ q ≼ x ⟹ p ≼ q ∨ q ≼ p)`

## SelfOwnershipAtPrefix — SelfOwnershipAtPrefix (DERIVED, lemma)

`(A Σ : Σ reachable from Σ₀ : (A π ∈ Π_Σ : ω_Σ(pfx(π)) = π))`

*Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`.

*Postconditions:* `ω_Σ(pfx(π)) = π`.

## O3 — OwnershipRefinement (DERIVED, lemma)

`(A a ∈ Σ.B, Σ, Σ' : Σ reachable from Σ₀ ∧ Σ → Σ' ∧ ω_{Σ'}(a) ≠ ω_Σ(a) ⟹ (E π_d ∈ Π_Σ, π' ∈ Π_{Σ'} ∖ Π_Σ : pfx(π') ≼ a ∧ #pfx(π') > #pfx(ω_Σ(a)) ∧ delegated_Σ(π_d, π')))`

*Monotonic-refinement corollary:* `#pfx(ω_{Σ'}(a)) ≥ #pfx(ω_Σ(a))` for `a ∈ Σ.B`.

*Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`, `Σ → Σ'`, `ω_{Σ'}(a) ≠ ω_Σ(a)`.

*Postconditions:* `(E π_d ∈ Π_Σ, π' ∈ Π_{Σ'} ∖ Π_Σ : pfx(π') ≼ a ∧ #pfx(π') > #pfx(ω_Σ(a)) ∧ delegated_Σ(π_d, π'))` — the change is witnessed by both the new principal `π'` (with a strictly longer matching prefix) and the delegator `π_d`.

## OwnershipDomainPermanence — OwnershipDomainPermanence (DERIVED, lemma)

`(A π ∈ Π_Σ, Σ, Σ' : Σ → Σ' ∧ (E a ∈ odom(π) ∩ Σ.B : ω_{Σ'}(a) ≠ ω_Σ(a)) ⟹ (E π_d ∈ Π_Σ : pfx(π) ≼ pfx(π_d) ∧ covers_Σ*(π, π_d) ∧ (E π' ∈ Π_{Σ'} ∖ Π_Σ : delegated_Σ(π_d, π'))))`

*Multi-step corollary (OwnershipDomainPermanence★):*

`(A π ∈ Π_Σ, Σ, Σ', a : Σ reachable from Σ₀ ∧ Σ →⁺ Σ' ∧ a ∈ odom(π) ∩ Σ.B ⟹ (A i, 0 ≤ i < n : ω_{Σ_{i+1}}(a) ≠ ω_{Σ_i}(a) ⟹ (E π_d^{(i)} ∈ Π_{Σ_i}, π'^{(i)} ∈ Π_{Σ_{i+1}} ∖ Π_{Σ_i} : pfx(π) ≼ pfx(π_d^{(i)}) ∧ delegated_{Σ_i}(π_d^{(i)}, π'^{(i)}))))`

where `Σ →⁺ Σ'` abbreviates `Σ → Σ_1 → ... → Σ_n = Σ'` for some `n ≥ 1`.

*Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ odom(π) ∩ Σ.B`, `Σ → Σ'`, `ω_{Σ'}(a) ≠ ω_Σ(a)`.

*Postconditions:* `(E π_d ∈ Π_Σ : pfx(π) ≼ pfx(π_d) ∧ covers_Σ*(π, π_d) ∧ (E π' ∈ Π_{Σ'} ∖ Π_Σ : delegated_Σ(π_d, π')))`.

## O4 — DomainCoverage (DERIVED, lemma)

`(A Σ : Σ reachable from Σ₀ : (A a ∈ Σ.B : (E π ∈ Π_Σ : pfx(π) ≼ a)))`

*Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`.

*Postconditions:* `(E π ∈ Π_Σ : pfx(π) ≼ a)`.

## O5 — SubdivisionAuthority (AX, axiom)

`(A Σ, Σ', a, π : Σ → Σ' ∧ π ∈ Π_Σ ∧ a ∈ Σ'.B ∖ Σ.B ∧ allocated_by_{Σ'}(π, a) ⟹ pfx(π) ≼ a ∧ (A π' ∈ Π_Σ : pfx(π') ≼ a ⟹ #pfx(π') ≤ #pfx(π)))`

## AccountPrefix — AccountPrefix (LEMMA, lemma)

`(A a ∈ T : T4(a) ⟹ acct(a) ≼ a)`

When `zeros(a) ≤ 1`: `acct(a) = a` (equality). When `zeros(a) ≥ 2`: `acct(a) ≺ a` (strict prefix).

*Preconditions:* `a ∈ T`, `T4(a)`.

*Postconditions:* `acct(a) ≼ a`.

## O6 — StructuralProvenance (DERIVED, lemma)

`(A Σ reachable, a, b ∈ Σ.B : acct(a) = acct(b) ⟹ ω(a) = ω(b))`

*Corollary (owner prefix containment):* `pfx(ω(a)) ≼ acct(a)` for all `a ∈ Σ.B`.

*Preconditions:* `Σ` reachable from `Σ₀`, `a, b ∈ Σ.B`, `acct(a) = acct(b)`.

*Postconditions:* `ω(a) = ω(b)`. Invariant: `pfx(ω(a)) ≼ acct(a)` for all `a ∈ Σ.B`.

## O7 — OwnershipDelegation (DERIVED, lemma)

`(A Σ, Σ', π, π' : Σ → Σ' ∧ delegated_Σ(π, π') :`

(a) `ω_{Σ'}(a) = π'` for all `a ∈ odom(π') ∩ Σ'.B`

(b) `π'` may allocate new addresses within `odom(π')` (O5 applies to `π'`)

(c) immediately upon entry at `Σ'`, `π'` may delegate to a new principal `π''` whose prefix is a next-reachable first child `p'' = next(Σ'.B, p, d)` of an already-baptized prefix (for some B6-valid `(p, d)`) — the only admissible delegate prefixes, by O17c — subject to obligations (i) [ancestry: `pfx(π') ≺ p''`], (iii) [structural-tier: `zeros(p'') ≤ 1`], and (v) [fresh-valid: `T4(p'') ∧ p'' ∉ Σ'.B`] on the choice of `p''` (conditions (ii) and (iv) being automatic given (i) and the original delegation's condition (iv))

*Preconditions:* `delegated_Σ(π, π')`, `Σ → Σ'`.

*Postconditions:* (a) `(A a ∈ odom(π') ∩ Σ'.B : ω_{Σ'}(a) = π')`; (b) `π'` satisfies O5 for allocations within `odom(π')`; (c) as stated above.

## O8 — IrrevocableDelegation (DERIVED, lemma)

`(A π, π', a, Σ_d, Σ_d^{post}, Σ' : Σ_d reachable from Σ₀ ∧ delegated(Σ_d, Σ_d^{post}, π, π') ∧ Σ_d^{post} →* Σ' ∧ π' ∈ Π_{Σ'} ∧ a ∈ odom(π') ∩ Σ'.B : ω_{Σ'}(a) ≠ π)`

The full trajectory is `Σ_d → Σ_d^{post} →* Σ'`. The domain restriction `odom(π') ∩ Σ'.B` ensures `ω` is applied only to addresses where it is defined (grounded by O4).

*Preconditions:* `Σ_d` reachable from `Σ₀`, `delegated(Σ_d, Σ_d^{post}, π, π')` (with `Σ_d → Σ_d^{post}` the introducing edge), `Σ_d^{post} →* Σ'`, `π' ∈ Π_{Σ'}`, `a ∈ odom(π') ∩ Σ'.B`.

*Postconditions:* `ω_{Σ'}(a) ≠ π`.

## O9 — NodeLocalOwnership (DERIVED, lemma)

`(A Σ reachable from Σ₀, π ∈ Π_Σ, a ∈ Σ.B : owns(π, a) ⟹ N(pfx(π)) ≼ N(a))`

When `zeros(pfx(π)) = 1`: `N(pfx(π)) = N(a)` (equality). When `zeros(pfx(π)) = 0`: `N(pfx(π)) ≼ N(a)` (proper prefix permitted).

*Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ Σ.B`, `owns(π, a)`.

*Postconditions:* `N(pfx(π)) ≼ N(a)`.

## O10 — DenialAsFork (DERIVED, lemma)

Sub-conditions:

(a) `ω(a') = π` — the new address is fully owned by the requesting principal

(b) the original address `a` persists in the registry (`a ∈ Σ'.B`, by B0) with its effective ownership unchanged (`ω_{Σ'}(a) = ω_Σ(a) ≠ π`) — no ownership is transferred

(c) `zeros(a') = zeros(pfx(π)) + 1` — the fork sits exactly one structural tier below the principal's prefix

*Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ Σ.B`, `ω(a) ≠ π`.

*Postconditions:* `(E Σ', a' : Σ → Σ' ∧ a' ∈ odom(π) ∩ Σ'.B ∧ ω_{Σ'}(a') = π ∧ zeros(a') = zeros(pfx(π)) + 1 ∧ a ∈ Σ'.B ∧ allocated_by_{Σ'}(π, a'))`, where `a' = pfx(π).0.{hwm_0 + 1}`, `hwm_0 := hwm(Σ.B, pfx(π), 2)`, and `Σ → Σ'` is a single baptism performed by `π` alone, as recorded by the `allocated_by_{Σ'}(π, a')` conjunct.

## O12 — PrincipalPersistence (AX, axiom)

`(A Σ, Σ' : Σ → Σ' ⟹ Π_Σ ⊆ Π_{Σ'})`

## O13 — PrefixImmutability (AX, axiom)

`(A π ∈ Π_Σ, Σ, Σ' : Σ → Σ' ∧ π ∈ Π_{Σ'} ⟹ pfx_{Σ'}(π) = pfx_Σ(π))`

## O14 — BootstrapPrincipal (AX, axiom)

**O14.1 (Nonempty):** `Π₀ ≠ ∅`

**O14.2 (Coverage):** `(A a ∈ Σ₀.B : (E π ∈ Π₀ : pfx(π) ≼ a))`

**O14.3 (Finite):** `|Π₀| < ∞`

**O14.4 (AccountTier):** `(A π ∈ Π₀ : zeros(pfx(π)) ≤ 1)`

**O14.5 (Injective):** `(A π₁, π₂ ∈ Π₀ : pfx(π₁) = pfx(π₂) ⟹ π₁ = π₂)`

**O14.6 (Valid):** `(A π ∈ Π₀ : T4(pfx(π)))`

**O14.7 (NonNesting):** `(A π₁, π₂ ∈ Π₀ : π₁ ≠ π₂ ⟹ pfx(π₁) ⋠ pfx(π₂) ∧ pfx(π₂) ⋠ pfx(π₁))`

**O14.8 (Baptized):** `(A π ∈ Π₀ : pfx(π) ∈ Σ₀.B)`

**O14.9 (Registry):** `Σ₀.B is an ASN-0040-reachable registry conforming to B₀ conf.`

## O15 — PrincipalClosure (AX, axiom)

`(A Σ, Σ' : Σ → Σ' ⟹ |Π_{Σ'} ∖ Π_Σ| ≤ 1)`

`(A π' ∈ Π_{Σ'} ∖ Π_Σ : (E π ∈ Π_Σ :`
`      (i)    [ancestry]        pfx(π) ≺ pfx(π')`
`      (ii)   [authorization]   (A π'' ∈ Π_Σ : pfx(π'') ≼ pfx(π') ⟹ #pfx(π'') ≤ #pfx(π))`
`      (iii)  [structural-tier] zeros(pfx(π')) ≤ 1`
`      (iv)   [top-down-order]  ¬(E π'' ∈ Π_Σ : pfx(π') ≺ pfx(π''))`
`      (v)    [fresh-valid]     T4(pfx(π')) ∧ pfx(π') ∉ Σ.B ))`

## StrictLongestCover — StrictLongestCover (LEMMA, lemma)

*General form:* let `χ ∈ Π_{Σ'}` cover an address `a` (`pfx(χ) ≼ a`), and suppose no principal of `Π_{Σ'}` covers `a` with a prefix strictly extending `pfx(χ)` (the *no-strict-extension* hypothesis). Then every covering principal `π'' ∈ Π_{Σ'}` with `π'' ≠ χ` satisfies `#pfx(π'') < #pfx(χ)`; hence `χ` is the unique coverer of `a` of maximal prefix length in `Π_{Σ'}`.

*Newly-delegated corollary:* Suppose `delegated_Σ(π, π')` holds along `Σ → Σ'`, so by O15 `π'` is the sole principal in `Π_{Σ'} ∖ Π_Σ` and conditions (i),(ii),(iv) hold. Then (Part 1) no `π'' ∈ Π_Σ` satisfies `pfx(π') ≼ pfx(π'')`: equality would force `π'' = π'` by O1b, impossible since `π' ∉ Π_Σ`, and `pfx(π') ≺ pfx(π'')` contradicts condition (iv). Consequently (Part 2), for every `a` with `pfx(π') ≼ a` the no-strict-extension hypothesis holds for `χ = π'` — no `π'' ∈ Π_Σ` strictly extends `pfx(π')`, and `π'` does not strictly extend itself — so the general form applies: `π'` is the unique coverer of `a` of maximal prefix length in `Π_{Σ'}`, every pre-existing coverer `π'' ∈ Π_Σ` having `#pfx(π'') < #pfx(π')`.

## NestingByDelegation — NestingByDelegation (DERIVED, lemma)

`(A Σ : Σ reachable from Σ₀ : (A π₁, π₂ ∈ Π_Σ : π₁ ≠ π₂ ⟹`
`      (pfx(π₁) and pfx(π₂) are non-nesting) ∨`
`      (pfx(π₁) ≺ pfx(π₂) ∧ covers_Σ*(π₁, π₂)) ∨`
`      (pfx(π₂) ≺ pfx(π₁) ∧ covers_Σ*(π₂, π₁)) ))`

where `covers_Σ*` is the reflexive-transitive closure of the parent relation `R_Σ`. Equality `pfx(π₁) = pfx(π₂)` is excluded by O1b.

## O16 — AllocationClosure (AX, axiom)

`(A Σ, Σ', a : Σ → Σ' ∧ a ∈ Σ'.B ∖ Σ.B ⟹ (E π ∈ Π_Σ : allocated_by_{Σ'}(π, a)))`

## O17 — AllocatedAddressValidity (DERIVED, lemma)

`(A Σ reachable from Σ₀, a : a ∈ Σ.B ⟹ T4(a))`

## O17b — BaptismalRegistryCoupling (AX, axiom)

`(A Σ, Σ' : Σ → Σ' ⟹ Σ'.B = Σ.B ∨ (E p, d : B6(p, d) : Σ'.B = Σ.B ∪ {next(Σ.B, p, d)} ∧ next(Σ.B, p, d) ∉ Σ.B))`

Principal-introducing sharpening:

`(A Σ, Σ', π' : Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ Σ'.B = Σ.B ∪ {pfx(π')})`

## O17c — PrincipalPrefixNextForm (DERIVED, lemma)

`(A Σ, Σ', π' : Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ (E p, d : B6(p, d) : pfx(π') = next(Σ.B, p, d)))`

## O18 — DelegationBaptizes (DERIVED, lemma)

`(A Σ, Σ', π' : Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ pfx(π') ∈ Σ'.B ∖ Σ.B)`

*Preconditions:* `Σ → Σ'`, `π' ∈ Π_{Σ'} ∖ Π_Σ`.

*Postconditions:* `pfx(π') ∈ Σ'.B ∖ Σ.B`.

## RegistryReachability — RegistryReachability (INV, predicate)

`(A Σ : Σ reachable from Σ₀ : Σ.B is an ASN-0040-reachable registry conforming to B₀ conf.)`

## PrefixBaptismCoupling — PrefixBaptismCoupling (DERIVED, lemma)

`(A Σ : Σ reachable from Σ₀ : (A π ∈ Π_Σ : pfx(π) ∈ Σ.B))`

*Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`.

*Postconditions:* `pfx(π) ∈ Σ.B`.

## ω_Σ(a) — EffectiveOwner (DEF, function)

`ω_Σ : Σ.B → Π_Σ` with `ω_Σ(a) = π ≡ π ∈ Π_Σ ∧ pfx(π) ≼ a ∧ (A π' ∈ Π_Σ : π' ≠ π ∧ pfx(π') ≼ a ⟹ #pfx(π) > #pfx(π'))`.

*Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`. Reachability is inherited from O4 (invoked in Step 1).

*Postconditions:* `(E! π ∈ Π_Σ : ω_Σ(a) = π)` — exactly one principal satisfies the defining equivalence.

## OwnershipDomain — OwnershipDomain (DEF, definition)

`odom(π) = {a ∈ T : pfx(π) ≼ a}` (`odom`, distinct from ASN-0034's allocator `dom`).

## acct(a) — AccountField (DEF, function)

*Preconditions:* `a ∈ T` is a valid tumbler satisfying T4 (HierarchicalParsing — positive non-separator components, at most three zeros) and T4a (SyntacticEquivalence — no adjacent zeros, no leading or trailing zero).

*Definition:* `acct(a) = a` when `zeros(a) = 0`; `acct(a) = N(a) ++ [0] ++ U(a)` when `zeros(a) ≥ 1`, where `N(a)` and `U(a)` are the node and user fields extracted by `fields(a)` (T4b UniqueParse), with component-wise access decidable from T3 (CanonicalRepresentation).

*Postconditions* (all four follow from FieldStructure): (a) `acct(a)` is a valid tumbler satisfying T4. (b) `zeros(acct(a)) ≤ 1`. (c) When `zeros(a) ≤ 1`: `acct(a) = a`. (d) When `zeros(a) ≥ 2`: `acct(a)` is a proper prefix of `a` with `zeros(acct(a)) = 1`.

## allocated_by_Σ(π, a) — AllocatedBy (AX, axiom)

*Signature:* `allocated_by_Σ : Principal × Tumbler → Bool`

*Semantics:* `allocated_by_{Σ'}(π, a)` holds when the baptism procedure, executing on behalf of `π`, produced `a` during the transition yielding `Σ'`.

## Delegation — DelegationPredicate (DEF, definition)

`delegated(Σ, Σ', π, π')` holds iff `Σ → Σ'`, `π ∈ Π_Σ`, `π' ∈ Π_{Σ'} ∖ Π_Σ`, and conditions (i)–(v) hold for `(π, π')` at `Σ` (the subscript form `delegated_Σ(π, π')` abbreviates `delegated(Σ, Σ', π, π')` for the `Σ'` bound by the surrounding formula). Evaluation-state convention: the delegator prefix `pfx(π)` is read at `Σ`, the delegate prefix `pfx(π')` at `Σ'`.

- (i) [ancestry] `pfx(π) ≺ pfx(π')`
- (ii) [authorization] `(A π'' ∈ Π_Σ : pfx(π'') ≼ pfx(π') ⟹ #pfx(π'') ≤ #pfx(π))`
- (iii) [structural-tier] `zeros(pfx(π')) ≤ 1`
- (iv) [top-down-order] `¬(E π'' ∈ Π_Σ : pfx(π') ≺ pfx(π''))`
- (v) [fresh-valid] `T4(pfx(π')) ∧ pfx(π') ∉ Σ.B`

## pfx(π) — OwnershipPrefix (AX, axiom)

*Axiom:* `pfx : Π → T` is a total mapping assigning each principal its ownership prefix.

*Preconditions:* `π ∈ Π`.

*Postconditions:* (a) `pfx(π) ∈ T`. (b) `T4(pfx(π))` — the prefix is a valid tumbler satisfying HierarchicalParsing.
