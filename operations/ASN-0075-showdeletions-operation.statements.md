> **ASN-0075 · SHOWDELETIONS Operation** — condensed claim statements  
> [← Full note](ASN-0075-showdeletions-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0075 Claim Statements

*Source: ASN-0075-showdeletions-operation.md (revised 2026-05-25) — Extracted: 2026-06-03*

## Definition — Current

```
CURRENT(a, d)  ≡  a ∈ ran(M(d))
```

Applies to pairs `(a, d)` with `a ∈ dom(C)` and `d ∈ E_doc`. Every `a ∈ dom(C)` has `subspace_I(a) = s_C`; that conjunct is not carried in the predicate.

---

## Definition — Deleted

```
DELETED(a, d)  ≡  (a, d) ∈ R  ∧  a ∉ ran(M(d))
```

Applies to pairs `(a, d)` with `a ∈ dom(C)` and `d ∈ E_doc`.

---

## Definition — NeverIncluded

```
NEVER_INCLUDED(a, d)  ≡  (a, d) ∉ R
```

Applies to pairs `(a, d)` with `a ∈ dom(C)` and `d ∈ E_doc`.

---

## Definition — DeletedFromAWithB

```
DeletedFromAWithB(d_A, d_B)
   =  {a ∈ dom(C) :
         DELETED(a, d_A)
       ∧ CURRENT(a, d_B)}
```

Precondition: `d_A ∈ E_doc ∧ d_B ∈ E_doc`.

---

## Definition — DeletedFromBWithA

```
DeletedFromBWithA(d_A, d_B)
   =  {a ∈ dom(C) :
         DELETED(a, d_B)
       ∧ CURRENT(a, d_A)}
```

Precondition: `d_A ∈ E_doc ∧ d_B ∈ E_doc`. Symmetric counterpart of `DeletedFromAWithB`.

---

## Definition — ShowDeletions

```
SHOWDELETIONS(d_A, d_B)
   =  (DeletedFromAWithB(d_A, d_B), DeletedFromBWithA(d_A, d_B))
```

Precondition: `d_A ∈ E_doc ∧ d_B ∈ E_doc`. The two halves are disjoint: membership in `DeletedFromAWithB` requires `CURRENT(a, d_B)` (i.e., `a ∈ ran(M(d_B))`); membership in `DeletedFromBWithA` requires `DELETED(a, d_B)`, whose second conjunct is `a ∉ ran(M(d_B))`.

---

## CURRENT — Current (DEF, predicate)

`CURRENT(a, d) ≡ a ∈ ran(M(d))`

---

## DELETED — Deleted (DEF, predicate)

`DELETED(a, d) ≡ (a, d) ∈ R ∧ a ∉ ran(M(d))`

---

## NEVER_INCLUDED — NeverIncluded (DEF, predicate)

`NEVER_INCLUDED(a, d) ≡ (a, d) ∉ R`

---

## D-WIT — DWit (LEMMA, lemma)

Let `Σ` be a composite-boundary state. For every `a ∈ dom(Σ.C)` and `d ∈ Σ.E_doc`, if `a ∈ ran(M(d))` then `(a, d) ∈ R`.

Formally: `a ∈ dom(C) ∧ a ∈ ran(M(d)) ⟹ (a, d) ∈ R`

---

## D-EXH — DExh (LEMMA, lemma)

Let `Σ` be a state reachable from `Σ_0` by a finite sequence of valid composite transitions (equivalently, `Σ` is a composite boundary). For every `(a, d)` with `a ∈ dom(Σ.C)` and `d ∈ Σ.E_doc`, exactly one of `CURRENT(a, d)`, `DELETED(a, d)`, `NEVER_INCLUDED(a, d)` holds.

The three predicates cover the non-impossible cases of the cross-product `(a ∈ ran(M(d))) × ((a, d) ∈ R)`:

| `a ∈ ran(M(d))` | `(a, d) ∈ R` | Predicate |
|---|---|---|
| Yes | Yes | CURRENT |
| Yes | No | impossible (excluded by D-WIT) |
| No  | Yes | DELETED |
| No  | No  | NEVER_INCLUDED |

---

## D-DISCR — DDiscr (LEMMA, lemma)

No function computable from `(Σ.C, Σ.L, Σ.E, Σ.M)` alone can distinguish `DELETED(a, d)` from `NEVER_INCLUDED(a, d)` for arbitrary `(a, d)`.

Witness: two reachable states `Σ_1`, `Σ_2` with `(Σ_1.C, Σ_1.L, Σ_1.E, Σ_1.M) = (Σ_2.C, Σ_2.L, Σ_2.E, Σ_2.M)` yet `DELETED(a, d)` at `Σ_1` and `NEVER_INCLUDED(a, d)` at `Σ_2`. The histories differ only in `R`: `R_1 ⊇ {(a, d), (a, d')}` and `R_2 ⊇ {(a, d')}`, with `(a, d) ∈ R_1 \ R_2`.

---

## D-NEED — DNeed (COROLLARY, lemma)

Any system supporting SHOWDELETIONS must maintain at least one state component beyond `(C, L, E, M)` whose value disambiguates `DELETED(a, d)` from `NEVER_INCLUDED(a, d)` at every reachable state; `C* = R` suffices.

Formally: `¬(∃ f : (C, L, E, M) → Bool . ∀ Σ reachable . f(Σ.C, Σ.L, Σ.E, Σ.M) = DELETED(a, d))` implies the system must carry `C*` with `C* = R`.

---

## DeletedFromAWithB — DeletedFromAWithB (DEF, function)

`{a ∈ dom(C) : DELETED(a, d_A) ∧ CURRENT(a, d_B)}`

---

## DeletedFromBWithA — DeletedFromBWithA (DEF, function)

`{a ∈ dom(C) : DELETED(a, d_B) ∧ CURRENT(a, d_A)}`

Symmetric counterpart of `DeletedFromAWithB`.

---

## SHOWDELETIONS — ShowDeletions (DEF, function)

Observational operation:

```
SHOWDELETIONS(d_A, d_B)
   =  (DeletedFromAWithB(d_A, d_B), DeletedFromBWithA(d_A, d_B))
```

Precondition: `d_A ∈ E_doc ∧ d_B ∈ E_doc`.

Weakest precondition for result `q` (the set-equality postcondition):

```
wp(SHOWDELETIONS(d_A, d_B), q)  =  d_A ∈ E_doc  ∧  d_B ∈ E_doc
```

---

## D-BOUND — DBound (PRE, requires)

The pre-state `Σ` is reachable from `Σ_0` by a finite sequence of valid composite transitions under ValidComposite★ (ASN-0047), so SHOWDELETIONS is invoked at a composite boundary.

Precondition: `d_A ∈ E_doc ∧ d_B ∈ E_doc ∧ Σ is a composite-boundary state`.

---

## D-DISJ — DDisj (LEMMA, lemma)

Documents with disjoint `R`-projections — `{a : (a, d_A) ∈ R} ∩ {a : (a, d_B) ∈ R} = ∅` — satisfy `Q0` at any composite-boundary state `Σ`.

Where `Q0` abbreviates:

```
DeletedFromAWithB(d_A, d_B) = ∅  ∧  DeletedFromBWithA(d_A, d_B) = ∅
```

Equivalently:

```
(A a ∈ dom(C) :
    ¬(DELETED(a, d_A)  ∧  CURRENT(a, d_B))
  ∧ ¬(DELETED(a, d_B)  ∧  CURRENT(a, d_A)))
```

---

## D-SUBSP — DSubsp (CLAIM, lemma)

SHOWDELETIONS operates only over the content subspace (`s_C`); cross-document deletion comparison is structurally meaningful only there.

Formally: both output sets are subsets of `dom(C)`, and `∀ a ∈ dom(C) . subspace_I(a) = s_C` and `dom(C) ∩ dom(L) = ∅` (L14), so no link address can appear in any output.

---

## D-IDENT — DIdent (CLAIM, lemma)

For every `a` in either output set, the returned reference is precisely the I-address `a` — not a copy with new identity.

Formally: the output sets are subsets of `dom(C)`; each element is an existing I-address, not a freshly allocated copy.

---

## D-ORIG — DOrig (CLAIM, lemma)

For every `a` in either output set, `origin(a)` is determined and identifies a unique document — the originating allocator of `a`.

Formally: `∀ a ∈ (DeletedFromAWithB(d_A, d_B) ∪ DeletedFromBWithA(d_A, d_B)) . origin(a) ∈ E_doc` and `origin(a)` is invariant across all states in which `a ∈ dom(C)` (S7, ASN-0036).

---

## D-ORD — DOrd (CLAIM, lemma)

Each output half is a finite subset of `dom(C) ⊆ T`, hence linearly ordered by the restriction of T1 (ASN-0034) to that subset. The operation carries no ordering of its own — it takes no input ordering to preserve and emits a set; T1-orderability is a property of the output addresses, not a structure the operation transports.

---

## D-SYM — DSym (CLAIM, lemma)

Argument swap maps each output half into the other:

```
SHOWDELETIONS(d_A, d_B)  =  (X, Y)
SHOWDELETIONS(d_B, d_A)  =  (Y, X)
```

where `X = DeletedFromAWithB(d_A, d_B)` and `Y = DeletedFromBWithA(d_A, d_B)`.

By name-substitution: `DeletedFromAWithB(d_B, d_A) = {a ∈ dom(C) : DELETED(a, d_B) ∧ CURRENT(a, d_A)} = DeletedFromBWithA(d_A, d_B)`.

---

## D-OBS — DObs (CLAIM, lemma)

SHOWDELETIONS modifies no state component; it is purely observational.

Formally, for state `Σ = (C, L, E, M, R)` and the state `Σ'` obtaining after the operation:

```
Σ'.C  =  Σ.C
Σ'.L  =  Σ.L
Σ'.E  =  Σ.E
Σ'.R  =  Σ.R
(A d ∈ E_doc ::  Σ'.M(d) = Σ.M(d))
```

---

## D-STORE — DStore (CLAIM, property)

The output is not required to be stored as a document; it is a query result.

Formally: SHOWDELETIONS writes no state component (D-OBS), so the result is delivered to the caller and no persistent artefact is created in any component of `Σ`.

---

## D-RECONS — DRecons (CLAIM, lemma)

The output depends only on the current state `Σ`. It does not depend on the particular sequence of transitions by which `Σ` was reached.

Formally: each predicate `CURRENT`, `DELETED`, `NEVER_INCLUDED` is defined in terms of `Σ.M`, `Σ.R`, `dom(Σ.C)`, `subspace_I` only. For any two transition histories `h_1`, `h_2` with `h_1(Σ_0) = h_2(Σ_0) = Σ`:

```
SHOWDELETIONS_{h_1}(d_A, d_B)  =  SHOWDELETIONS_{h_2}(d_A, d_B)
```
