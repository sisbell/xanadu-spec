> **ASN-0121 · The FINDLINKSFROMTOTHREE Operation** — condensed claim statements  
> [← Full note](ASN-0121-findlinksfromtothree-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0121 Claim Statements

*Source: ASN-0121-findlinksfromtothree-operation.md (revised 2026-06-08) — Extracted: 2026-06-11*

## Definition — Touch

```
touch(e, r) ≡ coverage(e) ∩ coverage(r) ≠ ∅
```

`touch(e, r)` holds exactly when some address lies in both coverages. `e, r ∈ Endset`; `Endset = 𝒫_fin(Span)` (ASN-0043). Defined for endsets `e` and request set `r`.

---

## Definition — AtHome

```
athome(a, H) ≡ home(a) ∈ coverage(H)
```

`a` is a link address; `H` is a home-component endset or `∗`; `home(a)` is the document-level field projection `N(a).0.U(a).0.D(a)`.

---

## Definition — Lift

```
lift(e, ∗) ≡ true
lift(e, r) ≡ touch(e, r)   for r ≠ ∗
```

`e ∈ Endset` is a link's endset value; `r ∈ Endset ∪ {∗}` is the request component.

---

## Definition — LiftH

```
liftH(a, ∗) ≡ true
liftH(a, H) ≡ athome(a, H)   for H ≠ ∗
```

`a` is a link address; `H ∈ Endset ∪ {∗}` is the home-component of the request.

---

## Definition — LiftHD

```
liftH_d(q.H) ≡ (q.H = ∗) ∨ (d ∈ coverage(q.H))
```

`d = home(ℓ)` is the home of a freshly allocated link address; `q.H` is the home-component of the request `q`. Used in FL-WP cases (a) and (b).

---

## Definition — Sat

```
sat(a, q, Σ) ≡ liftH(a, H) ∧ lift(Σ.L(a).e₁, F) ∧ lift(Σ.L(a).e₂, G) ∧ lift(Σ.L(a).e₃, Θ)
```

`a ∈ dom(Σ.L)` is a link address; `q = (H, F, G, Θ) ∈ (Endset ∪ {∗})⁴` is a request four-tuple; `Σ.L(a) = (e₁, e₂, …)` with `e₁` the from-endset, `e₂` the to-endset, `e₃` the type-endset. Higher slots `e₄, …, eₙ` are unconstrained.

---

## Definition — Nullified

```
nullified(Σ) = { a ∈ dom(Σ.L) : (E (b, F', G') ∈ L_R^Σ :: a ∈ coverage(G')) }
```

`L_R^Σ` is the retraction relation — the arity-3 slice of the link store whose slot-3 coverage equals `coverage(R)`, where `R` is the designated retraction-type representative. Defined in ASN-0086.

---

## Definition — Addressable

```
addressable(Σ) = dom(Σ.L) \ nullified(Σ)
```

The set of currently addressable links: all links in the link store minus those targeted by a retraction tuple. Introduced in this ASN over ASN-0086's `nullified`.

---

## FL-DEF — FindlinksFttDef (DEF, function)

```
findlinks_FTT(q, Σ) = { a ∈ addressable(Σ) : sat(a, q, Σ) }
```

`q = (H, F, G, Θ) ∈ (Endset ∪ {∗})⁴`; `Σ = (Σ.C, Σ.L, Σ.E, Σ.M, Σ.R)` the five-tuple system state (ASN-0047); `sat` the conjunction of the four lifted slot-criteria (AND of the ORs); `addressable(Σ) = dom(Σ.L) \ nullified(Σ)` introduced here over ASN-0086's `nullified`; the FTT subscript keeps the operation distinct from ASN-0127's slot-agnostic, unfiltered `findlinks` (F-FIND), of which it is not a restriction; the operation has frame `Σ` (reads only, writes nothing).

---

## FL-LOC — LinkStoreLocality (LEMMA, lemma)

For fixed `q`, `findlinks_FTT(q, Σ)` is a function of `Σ.L` alone; `Σ.C`, `Σ.M`, `Σ.E`, `Σ.R` are never consulted.

*Proof.* `nullified` is a function of `Σ.L`, as established above, hence so is `addressable(Σ) = dom(Σ.L) \ nullified(Σ)`; and `sat(a, q, Σ)` reads only the stored value `Σ.L(a)`, the address projection `home(a)`, and the fixed `q`. No clause consults `Σ.C`, `Σ.M`, `Σ.E`, or `Σ.R`.

---

## FL-DEC — Decidability (LEMMA, lemma)

`touch(e, r)` is decidable by ASN-0086's CoverageEqualityDecidable cell-decomposition run for intersection-nonemptiness, and `athome(a, H)` by the same T2 span-membership test; corollary at FL-DEF: `sat` is decidable per link, `nullified(Σ)` is computable by ASN-0086's ActiveSubset argument, and `findlinks_FTT(q, Σ) ⊆ dom(Σ.L)` is a finite, computable set (L-fin, ASN-0093).

*Proof.* By `Endset = 𝒫_fin(Span)` (ASN-0043), `coverage(e)` and `coverage(r)` are finite unions of half-open T1-intervals `[s, s ⊕ ℓ)` (T12, ASN-0034). Run the cell-decomposition procedure of ASN-0086's CoverageEqualityDecidable on `e ∪ r`: each coverage is constant on every cell between consecutive sorted span endpoints, so `coverage(e) ∩ coverage(r) ≠ ∅` iff the representative of some nonempty cell lies in both — the same finitely many T2 tests, run for intersection-nonemptiness rather than coverage equality. The home test `athome(a, H)` is a single point `home(a)` against a finite interval union — the T2 span-membership test of ASN-0086's ActiveSubset.

---

## FL-SND — Soundness (LEMMA, lemma)

```
(A a : a ∈ findlinks_FTT(q, Σ) : a ∈ addressable(Σ) ∧ sat(a, q, Σ))
```

No returned link is withdrawn, and none fails any of the four criteria. In contrapositive form: a nullified link is not returned even when every criterion holds, and a link with any constrained slot *wholly disjoint* from the request — `coverage(Σ.L(a).eᵢ) ∩ coverage(Rᵢ) = ∅` for a constrained `Rᵢ`, or `home(a) ∉ coverage(H)` for a constrained `H` — is not returned. There are no false positives.

---

## FL-CMP — Completeness (LEMMA, lemma)

```
(A a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) : a ∈ findlinks_FTT(q, Σ))
```

Every currently addressable link meeting all four criteria is returned; none is silently omitted. Together with FL-SND, the result is *exactly* the satisfying subset of `addressable(Σ)`.

---

## FL-JUNK — NonImpedance (LEMMA, lemma)

Let `Σ →* Σ'` be any reachable sequence with:

```
nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)
```

and

```
(A a : a ∈ dom(Σ'.L) \ dom(Σ.L) : ¬ sat(a, q, Σ'))
```

(the inclusion `dom(Σ.L) ⊆ dom(Σ'.L)` holds across every `→*` by monotonicity, and `nullified(Σ) ⊆ nullified(Σ') ∩ dom(Σ.L)` likewise — so the first hypothesis's substantive content is its `⊆` half: no link of `dom(Σ.L)` becomes nullified across the sequence). Then:

```
findlinks_FTT(q, Σ') = findlinks_FTT(q, Σ)
```

The body of irrelevant links, however vast, neither enlarges the answer nor displaces a qualifying link from it. `sat` is decided per link, and the first hypothesis is what holds the non-per-link addressability conjunct fixed.

---

## FL-RES — ResidenceEndpointIndependence (LEMMA, lemma)

The home criterion is a function of the link address alone; the from/to/type criteria are functions of the link value alone. The four constraints are therefore *independent* slots of the request: residence may be constrained without constraining endpoints, and conversely.

In particular:
- with `F = G = Θ = ∗` the result is every addressable link residing in `H`, irrespective of what it connects;
- with `H = ∗` the result is every addressable link whose endpoints match, irrespective of where it lives.

---

## FL-DIR — PositionalDirectionality (LEMMA, lemma)

The from-criterion tests `Σ.L(a).e₁` and the to-criterion tests `Σ.L(a).e₂`; the slots are matched by position, not symmetrically.

Witness: Take two distinct content I-addresses `x = [1,0,1,0,1,0,1,5]` and `y = [1,0,1,0,1,0,1,9]` and unit-depth request endsets `X = {(x, δ(1,#x))}` and `Y = {(y, δ(1,#y))}`. By PrefixSpanCoverage, `coverage(X) = {t : x ≼ t}` and `coverage(Y) = {t : y ≼ t}`; since `x` and `y` are equal-length and distinct they are prefix-incomparable (Prefix, ASN-0034), so `coverage(X) ∩ coverage(Y) = ∅`. For `a ∈ dom(Σ.L)` with from-endset `e₁ = X`, to-endset `e₂ = Y`, `a ∉ nullified(Σ)`:

```
a ∈ findlinks_FTT((∗, X, Y, ∗), Σ)
a ∉ findlinks_FTT((∗, Y, X, ∗), Σ)
```

For `q = (∗, X, Y, ∗)`: `lift(e₁, X) = true` and `lift(e₂, Y) = true`, so `sat(a, q, Σ)` holds.
For `q' = (∗, Y, X, ∗)`: `lift(e₁, Y) ≡ touch(e₁, Y) = (coverage(X) ∩ coverage(Y) ≠ ∅) = false`, so `sat(a, q', Σ)` fails.

Reversing the two endpoint constraints is therefore not a no-op.

---

## FL-TYP — TypeByAddress (LEMMA, lemma)

The type criterion tests `touch(Σ.L(a).e₃, Θ)`, an overlap of address coverages, and never reads content stored at any type address.

Three consequences follow:

*(a) Ghost types.* A type address need not lie in `dom(Σ.C)`; an endset whose coverage includes addresses with no stored content is a valid, matchable type (L9, TypeGhostPermission, ASN-0043).

*(b) Independent constraint.* Because the type participates in `sat` on equal footing with from and to, a request may constrain type alone:

```
findlinks_FTT((∗, ∗, ∗, Θ), Σ)
```

returns every addressable link of a kind touching `Θ`, leaving from and to open.

*(c) Hierarchy by containment.* A type request whose span is rooted at a supertype address `p` covers the whole subtree `{t : p ≼ t}` and so matches all and only subtypes of `p` — L10 (TypeHierarchyByContainment, ASN-0043, via PrefixSpanCoverage) applied on the request side; the type slot is searchable for super- and sub-types without any registry.

---

## FL-WILD — WildcardSemantics (LEMMA, lemma)

A wildcard slot imposes no constraint: `findlinks_FTT` with a wildcard component returns exactly the links the *remaining* constrained slots admit.

In the limit:

```
findlinks_FTT((∗, ∗, ∗, ∗), Σ) = addressable(Σ)
```

— all currently addressable links — and a single constrained slot yields precisely the links matching that slot alone.

---

## FL-EMP — EmptyConstraintZero (LEMMA, lemma)

For a constrained slot whose endset has empty coverage:

```
lift(e, ∅) ≡ touch(e, ∅) ≡ coverage(e) ∩ ∅ ≠ ∅
```

is `false` for every link `a` (and likewise `liftH(a, H) ≡ home(a) ∈ ∅` is `false` when `H` has empty coverage). Hence if *any* constrained component of `q` has empty coverage:

```
findlinks_FTT(q, Σ) = ∅
```

regardless of the store's contents. `∗` is the *unit* of the conjunction (`lift(e, ∗) = true`, drops out), whereas the empty endset is the *zero* (`lift(e, ∅) = false`, forces the whole conjunction to `false`).

By the symmetry of `touch`, the same zero applies to a *link's* own empty endset (L3 permits `e₁ = ∅` or `e₂ = ∅`): for any constrained from-request `F ≠ ∗`,

```
lift(∅, F) ≡ touch(∅, F) ≡ coverage(∅) ∩ coverage(F) = ∅ ∩ coverage(F) = ∅
```

so `lift(∅, F) = false` and the link is excluded from any constrained from-slot. Under the from-wildcard `F = ∗`, `lift(∅, ∗) = true` drops the slot. Empty coverage on *either* side of `touch` annihilates that slot's test.

---

## FL-MON — MonotoneAccumulation (LEMMA, lemma)

For any reachable `Σ →* Σ'` with `a ∉ nullified(Σ')`:

```
a ∈ findlinks_FTT(q, Σ) ⟹ a ∈ findlinks_FTT(q, Σ')
```

A matching link, once found and not withdrawn, stays found as the store grows.

*Proof.* By LP13 (ASN-0098, UnconditionalLinkPersistence) `Σ'.L(a) = Σ.L(a)` across the reachability closure `Σ →* Σ'`, and `home(a)` is a projection of the fixed address `a`, so `sat(a, q, Σ') = sat(a, q, Σ)`; and `a ∈ addressable(Σ')` because `a ∈ dom(Σ'.L)` by link-store monotonicity across `Σ →* Σ'` (ASN-0098 StoreMonotonicity★) and `a ∉ nullified(Σ')` by hypothesis.

---

## FL-WP — WeakestPrecondition (LEMMA, lemma)

Weakest preconditions for the unique result-changing transition K.λ — per-case wp's for the three result-changing cases, partitioned by retraction-relation membership.

*Scope.* Each case presupposes `enabled(K.λ)` — ASN-0093's K.λ binding precondition: the home document is allocated (`d ∈ dom(Σ.M)`), the address is pinned to the link sub-allocator's frontier, and the value has L3 shape (arity `≥ 3`, each slot an endset, slot-3 type endset non-empty). The displayed conjunctions are the weakest *additional* precondition under which the named, enabled K.λ step lands the address in the answer.

*(a) Entry of a fresh ordinary link.* Let `Σ → Σ'` be a K.λ step allocating a fresh address `ℓ ∉ dom(Σ.L)` with value `Σ'.L(ℓ) = (F, G, Θ, e₄, …, e_N)` of arity `N ≥ 3`, homed at `d = home(ℓ)`, with `ℓ ∉ L_R^{Σ'}` (the link does not enter the retraction relation — `¬(|Σ'.L(ℓ)| = 3 ∧ coverage(Σ'.L(ℓ).e₃) = coverage(R))`), so `L_R^{Σ'} = L_R^Σ`. Then:

```
wp(K.λ, ℓ ∈ findlinks_FTT(q, ·)) ≡
    ¬(E (b, F', G') ∈ L_R^Σ :: ℓ ∈ coverage(G'))
  ∧ liftH_d(q.H)
  ∧ lift(F, q.F)
  ∧ lift(G, q.G)
  ∧ lift(Θ, q.Θ)
```

where `liftH_d(q.H) ≡ (q.H = ∗) ∨ (d ∈ coverage(q.H))`.

*(b) Entry of a fresh retraction link.* Let `Σ → Σ'` be a K.λ step allocating a fresh address `b ∉ dom(Σ.L)` with value `Σ'.L(b) = (F_b, G', Θ_b)` of arity exactly 3 with `coverage(Θ_b) = coverage(R)`, so `b ∈ L_R^{Σ'}` and `L_R^{Σ'} = L_R^Σ ∪ {(b, F_b, G')}`, with `d = home(b)`. Then:

```
wp(K.λ, b ∈ findlinks_FTT(q, ·)) ≡
    ¬(E (c, F'', G'') ∈ L_R^Σ :: b ∈ coverage(G''))
  ∧ b ∉ coverage(G')
  ∧ liftH_d(q.H)
  ∧ lift(F_b, q.F)
  ∧ lift(G', q.G)
  ∧ lift(Θ_b, q.Θ)
```

The second conjunct `b ∉ coverage(G')` is the self-retraction term — the very tuple just committed names its own address.

*(c) Survival of an existing match under retraction.* Let `Σ → Σ'` be a K.λ step committing a retraction tuple whose to-coverage is `coverage(G')`, so `L_R^{Σ'} = L_R^Σ ∪ {(b, ∅, G')}`. For an existing link `a ∈ dom(Σ.L)`:

```
wp(K.λ_retract, a ∈ findlinks_FTT(q, ·)) ≡
    a ∈ findlinks_FTT(q, Σ) ∧ a ∉ coverage(G')
```

*Derivation.* `sat(a, q, ·)` is constant across the step (`Σ'.L(a) = Σ.L(a)` by L12, `home(a)` fixed). Unfolding `nullified` at `Σ'` for existing `a ∈ dom(Σ.L)`:

```
a ∈ nullified(Σ') ⟺
    (E (c, F'', G'') ∈ L_R^Σ :: a ∈ coverage(G'')) ∨ a ∈ coverage(G')
⟺  a ∈ nullified(Σ) ∨ a ∈ coverage(G')
```

Negating: `a ∈ addressable(Σ') ⟺ a ∉ nullified(Σ) ∧ a ∉ coverage(G')`. Conjoining with `sat` and folding back into FL-DEF gives the stated wp.

---

## FL-STB — StabilityUnderEditing (LEMMA, lemma)

For a transition `Σ → Σ'` that preserves the link store — `Σ'.L = Σ.L` — and any request `q`:

```
findlinks_FTT(q, Σ') = findlinks_FTT(q, Σ)
```

This is FL-LOC routed through ASN-0127's meta-lemma F-CIL (ComprehensionInvariantUnderΣL): FL-DEF is the comprehension `{ a ∈ dom(Σ.L) : a ∉ nullified(Σ) ∧ sat(a, q, Σ) }`, whose membership predicate consults only `Σ.L` and query-data (FL-LOC), so F-CIL delivers the equality from the single hypothesis `Σ'.L = Σ.L`; in particular retraction-set preservation (`nullified(Σ') = nullified(Σ)`) is a consequence of the link-store hypothesis, not an independent assumption. Pure-arrangement edits (insertion, deletion, rearrangement) and content appends preserve `Σ.L` (F-PRES, ASN-0127) and so leave the answer invariant.

---

## FL-RET — RetractionAbsence (LEMMA, lemma)

If `a ∈ nullified(Σ)`, then for every reachable `Σ →* Σ'` and every request `q`:

```
a ∉ findlinks_FTT(q, Σ')
```

The exclusion is total: `a ∉ addressable(Σ')` removes it from FL-DEF, and the non-decrease of `nullified` across `→` and `→*` keeps it out forever.

For any `b ≠ a`: neither membership conjunct reads `a`'s status — `sat(b, q, ·)` reads only `b`'s own value and address, and `b ∈ nullified(·)` is an existential over the stored tuples of the audit slice `L_R`, which is selected by stored-value tests alone and is not edited by `a`'s nullification. If `a` is itself a retraction tuple, it remains in `L_R^{Σ'}` after `a` is nullified and continues to nullify its targets — retraction-of-retraction is a non-fixpoint operation (R6b, ASN-0086).

---

## FL-REACH — CrossDocumentReach (LEMMA, lemma)

For any request `q`, `findlinks_FTT(q, Σ)` is independent of `Σ.M` (immediate from FL-LOC).

Four consequences follow:

*(a) Every home is reached.* The store is searched whole; a link is eligible regardless of which document homes it.

*(b) Transclusion is found once.* When the same endpoint content is shared across documents, the link is indexed by that content's I-addresses and is found exactly once by content identity, however many documents surface it.

*(c) Whole-docuverse residence.* Setting `H = ∗` imposes no residence bound, returning all matching links wherever homed.

*(d) Superset of the satisfying discoverable links.*

```
findlinks_FTT(q, Σ) ⊇ ⋃_d { a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) ∧ discoverable_from(a, d, Σ) }
```

The inclusion is *strict* whenever a satisfying, addressable *orphan* exists: an addressable `a` with `sat(a, q, Σ)` whose endset I-addresses lie in no arrangement range fails `discoverable_from(a, d, Σ)` for every `d` yet lies in `findlinks_FTT(q, Σ)`.
