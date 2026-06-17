> **ASN-0111 · READLINK — Reading a Link's Structure by Its Own Address** — condensed claim statements  
> [← Full note](ASN-0111-readlink-operation-read-a-link's-structure-by-its-address.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0111 Claim Statements

*Source: ASN-0111-readlink-operation-read-a-link's-structure-by-its-address.md (revised 2026-06-10) — Extracted: 2026-06-11*

## Definition — Coverage

> `coverage(e) = (∪ (s, ℓ) : (s, ℓ) ∈ e : {t ∈ T : s ≤ t < s ⊕ ℓ})`.

## Definition — LinkType

> `Link = {(e₁, e₂, ..., eₙ) : N ≥ 3, each eᵢ ∈ Endset}`,  `Endset = 𝒫_fin(Span)`

with the standard-triple convention assigning slot 1 the *from*-endset, slot 2 the *to*-endset, and slot 3 the *type*-endset.

## Definition — StructuralScreen

A candidate tumbler `a ∈ T` passes the structural screen left to right:

> `T4-valid(a) ∧ zeros(a) = 3 ∧ subspace_I(a) = s_L ∧ #E(a) ≥ 2`

## Definition — PostconditionsRL0

> `R_ok ≡ a ∈ dom(Σ.L) ∧ result = Σ.L(a)`,   `R_⊥ ≡ result = ⊥`.

---

## readlink — Readlink (DEF, FUNCTION)

> `readlink : T × 𝒮 → Link ∪ {⊥}`
> `readlink(a, Σ) = Σ.L(a)`   when `a ∈ dom(Σ.L)`
> `readlink(a, Σ) = ⊥`        when `a ∉ dom(Σ.L)`.

`𝒮` is the extended state space of ASN-0047, whose members are the states `Σ = (C, L, E, M, R)`, second argument restricted to reachable states. Pure read, frame condition `Σ' = Σ`.

---

## RL0 — TotalityAndSuccess (LEMMA, BICONDITIONAL)

`readlink(a, Σ)` is defined for every `a ∈ T`, and

> `readlink(a, Σ) ∈ Link ⟺ a ∈ dom(Σ.L)`,   `readlink(a, Σ) = ⊥ ⟺ a ∉ dom(Σ.L)`.

Every conjunct of the structural screen `T4-valid(a) ∧ zeros(a) = 3 ∧ subspace_I(a) = s_L ∧ #E(a) ≥ 2` is necessary, and no satisfiable address-computable predicate is sufficient (witness: `dom(Σ₀.L) = ∅`).

Sub-claims:

(a) `wp(result := readlink(a, Σ), R_ok) ≡ a ∈ dom(Σ.L) ∧ readlink(a, Σ) = Σ.L(a) ≡ a ∈ dom(Σ.L)`

(b) `wp(result := readlink(a, Σ), R_⊥) ≡ readlink(a, Σ) = ⊥ ≡ a ∉ dom(Σ.L)`

(c) The two weakest preconditions are complementary: at each `(a, Σ)` exactly one of the two postconditions is attainable.

---

## RL1 — Completeness (LEMMA, UNIVERSALLY-QUANTIFIED)

For `a ∈ dom(Σ.L)`, each returned endset equals the recorded one exactly, omitting nothing and introducing nothing:

> `(A i : 1 ≤ i ≤ |Σ.L(a)| : readlink(a, Σ).eᵢ = Σ.L(a).eᵢ)`.

Corollaries (since the output is `Σ.L(a)`):

(a) Satisfies L3: arity ≥ 3, non-empty type slot, connective slots may be `∅`.

(b) Satisfies L5: membership not sequence within an endset.

(c) Satisfies Endset well-formedness: T12 spans — the read can return neither a malformed nor an empty span, and always returns a usable type endset.

(d) Inherits L4-generality of the recorded spans.

---

## RL2 — RolePreservation (LEMMA, STRUCTURAL)

For `a ∈ dom(Σ.L)`, the read preserves the link's arity, and slot position is part of the value:

> `|readlink(a, Σ)| = |Σ.L(a)|`,  and for each `1 ≤ i ≤ |Σ.L(a)|` the positional accessor `readlink(a, Σ).eᵢ` is a model primitive (L6, ASN-0043), with link equality componentwise.

In the arity-3 case slot 1 is *from*, slot 2 is *to*, and slot 3 is *type*; for `N > 3` (L3, ASN-0043) the higher slots are returned under their own indices.

---

## RL3 — TypeByAddress (LEMMA, INTERPRETATION)

The relationship the type records is fixed by `coverage(e₃)` — the set of addresses the type-set names — and not by whatever is, or is not, stored at those addresses. Two links share a type exactly when their type endsets have equal coverage (L8, ASN-0043), a relation on address sets, decided without dereferencing a single one. Ghost types are permitted (L9, ASN-0043), and the read of a ghost-typed link is no less complete than any other.

---

## SOV — StoreOnlyCompositeValidity (LEMMA, VALIDITY)

A composite whose steps touch neither `dom(C)`, nor a content-subspace arrangement range, nor `R`, satisfies the coupling constraints J0, J1★, and J1'★ vacuously at every boundary:

- J0 quantifies over fresh content addresses, and `dom(C') \ dom(C) = ∅`
- J1★ quantifies over content-subspace range-new I-addresses, of which none arise
- J1'★ quantifies over new provenance entries, and `R' \ R = ∅`

and is therefore a valid composite (ValidComposite★, ASN-0047) whenever each step's elementary precondition holds at its intermediate state.

---

## RL4 — NestingLocality (LEMMA, CONGRUENCE)

For any reachable states `Σ₁, Σ₂` and any address `a ∈ dom(Σ₁.L) ∩ dom(Σ₂.L)` with `Σ₁.L(a) = Σ₂.L(a)`:

> `readlink(a, Σ₁) = readlink(a, Σ₂)`

The failure branch supplies the complementary congruence: for `a ∉ dom(Σ₁.L) ∧ a ∉ dom(Σ₂.L)`:

> `readlink(a, Σ₁) = ⊥ = readlink(a, Σ₂)`

Together the two branches make `readlink` a function of `(a, Σ.L(a))` alone, with `Σ.L(a)` read as a value in `Link` extended by "undefined". Corollary: covered link addresses are returned as addresses, never dereferenced, witnessed by an explicit branched-history state pair satisfying:

> `Σ₁.L(c) = ℓ_c = Σ₂.L(c)`,  `Σ₁.L(a') = v₁ ≠ v₂ = Σ₂.L(a')`,  `a' ∈ coverage(Σ₁.L(c).e₂)`

---

## RL5 — Determinacy (LEMMA, STABILITY)

`readlink` is a pure function of `(a, Σ.L(a))` (RL4). Moreover, the read is stable across the whole future:

> `(A Σ, Σ' : Σ →* Σ' ∧ a ∈ dom(Σ.L) : readlink(a, Σ') = readlink(a, Σ))`.

Sub-claims:

(a) The structural screen is a one-sided test: failure proves permanent absence; passage proves nothing about the future.

(b) Three permanence families exhaust permanent absence:
   - *Depth*: `#E(a) > 2 ⟹ a ∉ F ⟹ a ∉ dom(Σ'.L)` at every reachable `Σ'`
   - *Lineage*: `N(a)₁ ≠ 1 ⟹ a ∉ dom(Σ'.L)` at every reachable `Σ'`
   - *User-field*: `#U(a) ≥ 2 ⟹ a ∉ dom(Σ'.L)` at every reachable `Σ'`

(c) Every member of the residual class (screen-passing, `#E(a) = 2`, `N(a)₁ = 1`, `#U(a) = 1`) is allocatable in some extension of any history that has not yet allocated it.

(d) Caching discipline: `⊥` is cacheable exactly where one of the address-computable permanence proofs from (b) is in hand.

---

## RL6 — RecordedNotResolved (LEMMA, INDEPENDENCE)

`readlink(a, Σ)` depends only on `Σ.L` — indeed only on `Σ.L(a)` (RL4) — and is independent of every document arrangement. Consequently the read succeeds and returns the complete structure even for an *orphaned* link — one whose endpoint content is currently arranged in no document, so that resolving its endsets would yield nothing. The link's structure persists unconditionally (L12; LP13 of ASN-0098), and the read surfaces it unconditionally.
