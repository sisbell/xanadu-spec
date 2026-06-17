> **ASN-0132 · The FINDNUMOFLINKSFROMTOTHREE Operation** — condensed claim statements  
> [← Full note](ASN-0132-findnumoflinksfromtothree-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0132 Claim Statements

*Source: ASN-0132-findnumoflinksfromtothree-operation.md (revised 2026-06-13) — Extracted: 2026-06-13*

## Definition — CountLinksFTT

For a request `q = (H, F, G, Θ)` and state `Σ`:

`countlinks_FTT(q, Σ) ≡ |{ a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) }|`

The operation reads `Σ` and returns a natural number; its frame is `Σ` — it writes nothing, mutating no component of the state.

## Definition — SatPredicate

`sat(a, q, Σ) ≡ liftH(a, H) ∧ lift(Σ.L(a).e₁, F) ∧ lift(Σ.L(a).e₂, G) ∧ lift(Σ.L(a).e₃, Θ)`

where `q = (H, F, G, Θ) ∈ (Endset ∪ {∗})⁴`, each component an endset or the wildcard `∗`.

## Definition — Addressable

`addressable(Σ) = dom(Σ.L) \ nullified(Σ)`

The link store minus those targeted by a retraction tuple.

---

## CN-DEF — CountLinksFTT (DEF, function)

`countlinks_FTT(q, Σ) ≡ |{ a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) }|`; the operation reads `Σ`, returns ℕ, and has frame `Σ` (writes nothing); defined through the shared relation `sat` (ASN-0121), not through the enumeration operation; well-defined because the counted set is a finite, computable subset of `dom(Σ.L)` (L-fin ASN-0093, FL-DEC ASN-0121).

## CN-LOC — LinkStoreLocality (LEMMA, lemma)

Link-store locality — for fixed `q`, `countlinks_FTT(q, Σ)` is a function of `Σ.L` alone; `Σ.C`, `Σ.M`, `Σ.E`, `Σ.R` are never consulted (from FL-LOC, ASN-0121).

Formal basis: `sat(a, q, Σ)` consults only the stored value `Σ.L(a)` and the address projection `home(a)`, and `addressable(Σ)` is a function of `Σ.L` alone; therefore the counted set — and therefore the count — is a function of `Σ.L` and `q` alone.

## CN-UNIT — UnitIsLinkIdentity (THM, theorem)

The unit is link identity — each addressable satisfying link contributes exactly `1`, independent of anchoring (endset span/address) multiplicity, transclusion multiplicity, and arrangement-appearance multiplicity (cross-version surfacing being an instance of the last, forking being link-store-inert by J4, ASN-0047).

Formal statement: For every request `q` and state `Σ`, each `a ∈ addressable(Σ)` with `sat(a, q, Σ)` contributes exactly `1` to `countlinks_FTT(q, Σ)`, and each `a` with `¬sat(a, q, Σ)` or `a ∉ addressable(Σ)` contributes `0`. The contribution of a link is independent of:

- (a) the number of spans or addresses its endsets reference
- (b) the number of documents through which its endpoint content is reachable
- (c) the number of arrangement positions at which it surfaces

Sub-claim (a): The from-clause `lift(Σ.L(a).e₁, F) ≡ touch(Σ.L(a).e₁, F) ≡ coverage(Σ.L(a).e₁) ∩ coverage(F) ≠ ∅` is a single existential over the addresses of the endset; the link enters the set once or not at all.

Sub-claim (b): Transclusion is a property of `Σ.M`; by CN-LOC the count never consults `Σ.M`, so `a` is one address in `Σ.L`, counted once.

Sub-claim (c): `discoverable_from(a, d, Σ)` is an `Σ.M`-mediated relation; by CN-LOC it does not enter the count.

## CN-ENUM — CountEqualsEnumCardinality (THM, theorem)

Count equals enumeration length at a single state, structurally (both are the cardinality of one set), and may differ across distinct states evaluated by separate inquiries.

Formal statement:

`countlinks_FTT(q, Σ) = |findlinks_FTT(q, Σ)|`

because both sides are the cardinality of the single set `{a ∈ addressable(Σ) : sat(a, q, Σ)}` — the right side by FL-DEF (ASN-0121), the left by CN-DEF.

Qualifier: the equality holds whenever both sides are evaluated against the *same* `Σ`.

## CN-ZERO — ZeroIsPresentStoreExistential (THM, theorem)

A positive present-store existential (no addressable link satisfies `q`), distinct from "not found" (excluded by FL-JUNK) and "not displayed" (excluded by CN-LOC); a degenerate empty-coverage request also yields `0` (FL-EMP) but asserts only that the request names nothing.

Formal statement:

`countlinks_FTT(q, Σ) = 0  ⟺  (A a : a ∈ addressable(Σ) : ¬sat(a, q, Σ))`

A zero count asserts that *no* addressable link in the store satisfies the four sets at `Σ` — that the satisfying set is empty. It is a positive statement about the contents of the link store, not a report that a search failed or that nothing could be displayed.

Precondition for the substantive guarantee: the constrained components of `q` have non-empty coverage (non-degenerate request). If a constrained component of `q` has empty coverage, `lift` is `false` for every link (FL-EMP, ASN-0121) and the count is `0` vacuously — the *empty-request* zero, distinct in meaning from the *empty-store* zero of CN-ZERO.

## CN-SNAP — CountIsSnapshot (THM, theorem)

The count is a measurement of `Σ`, recomputed per inquiry, recorded in no state component; it may change under any mutation and the specification imposes no obligation that a prior count remain valid (recompute-on-read).

Formal statement: `countlinks_FTT(q, Σ)` is a function of the state `Σ`. No component of `Σ` records it; there is no stored counter that the operation reads or maintains. After any mutation `Σ → Σ'` the value `countlinks_FTT(q, Σ')` may differ from `countlinks_FTT(q, Σ)`, and the specification imposes no obligation that the earlier value remain valid. Re-establishing the count requires re-evaluating the cardinality at the current state.

## CN-STAB — InvarianceUnderArrangementEditing (THM, theorem)

For fixed `q`, any link-store-preserving transition (content insertion/deletion/rearrangement, content allocation, provenance recording — F-PRES ASN-0127) leaves the count invariant; in particular a reverse-orphaned link still contributes to a home-bounded count, residence being a projection of the permanent address.

Formal statement: For a fixed request `q`, any transition `Σ → Σ'` that preserves the link store —

`dom(Σ'.L) = dom(Σ.L)` and `Σ'.L(a) = Σ.L(a)` for all `a`

— satisfies `countlinks_FTT(q, Σ') = countlinks_FTT(q, Σ)`.

Proof basis: the hypothesis `Σ'.L = Σ.L` entails `nullified(Σ') = nullified(Σ)` (since `nullified(Σ)` is selected from the retraction relation `L_R^Σ` which `Σ.L` determines); the count is a function of `Σ.L` alone (CN-LOC), so it is fixed.

## CN-RETRACT — RetractionExcludesImmediately (THM, theorem)

A nullified link contributes `0` to every count immediately and permanently (R6a ASN-0086, FL-RET ASN-0121) while remaining in `dom(Σ.L)` with fixed value (L12 ASN-0043); the count ranges over the active view `addressable(Σ)`, reconciling immediate exclusion with store permanence.

Formal statement: If `a ∈ nullified(Σ)`, then `a` contributes `0` to `countlinks_FTT(q, Σ)` for every `q`, and continues to contribute `0` at every reachable successor state (the nullified set never shrinks — R6a, ASN-0086). Yet `a` remains in `dom(Σ.L)` with its value `Σ.L(a)` permanently fixed (L12, ASN-0043). The count ranges over `addressable(Σ) = dom(Σ.L) \ nullified(Σ)`; it counts the *active view*, not the *store*.

## CN-MONO — MonotoneAccumulationAbsentRetraction (THM, theorem)

Absent retraction of counted links, the count is non-decreasing across `Σ →* Σ'`; a fresh link that satisfies `q` and is addressable increments it by `1` (the body derives the wp precondition, which differs for ordinary vs. retraction links); `K.λ` is the only count-changing transition.

Formal statement: Across any `Σ →* Σ'` in which no currently-counted link becomes nullified:

`countlinks_FTT(q, Σ) ≤ countlinks_FTT(q, Σ')`

and each newly created *ordinary* link that satisfies `q` and is addressable increments the count by exactly `1`; a newly created *retraction* link that satisfies `q` and is addressable likewise increments it by `1`, under the stronger precondition FL-WP(b) (ASN-0121) records.

Weakest-precondition for ordinary link `ℓ` (fresh, `ℓ ∉ dom(Σ.L)`, `L_R^{Σ'} = L_R^Σ`):

`wp(create ℓ, countlinks_FTT(q, ·) = countlinks_FTT(q, Σ) + 1) = sat(ℓ, q, Σ') ∧ ¬(E (b, F', G') ∈ L_R^Σ :: ℓ ∈ coverage(G'))`

For a fresh *retraction* link `b` (where `L_R^{Σ'} = L_R^Σ ∪ {(b, F', G')}`), `b` adds `1` exactly when it satisfies `q` and clears the self-retraction conjunct `b ∉ coverage(G')`.

Under the unit-depth retraction discipline (ASN-0086) the second conjunct of the ordinary case is automatic and the precondition collapses to `sat(ℓ, q, Σ')`.

## CN-ORPHAN — OrphansAreCounted (THM, theorem)

A satisfying addressable link is counted regardless of whether any arrangement surfaces it (`discoverable_from` irrelevant); the count is an existence census over `addressable(Σ)` whose counted set is a superset of the cross-document union of surfaced satisfying links (FL-REACH, ASN-0121), exceeding that union's cardinality by exactly the orphans.

Formal statement: A link `a ∈ addressable(Σ)` with `sat(a, q, Σ)` is counted whether or not any document surfaces its endpoint content — that is, whether or not `discoverable_from(a, d, Σ)` holds for any `d`.

Superset relation: the counted set is a superset of the cross-document union of surfaced satisfying links:

`⋃_d {a : a ∈ addressable(Σ) ∧ sat(a, q, Σ) ∧ discoverable_from(a, d, Σ)}`

and the count exceeds the cardinality of that union by exactly the orphans — the satisfying addressable links no document surfaces.

## CN-OBT — CountedIdentityIsPermanentHandle (THM, theorem)

Each of the `N` counted identities is a permanent address (ASN-0093) with fixed value (L12, ASN-0043), so a count of `N` warrants `N` durable handles answering `q`, obtainable in principle; it does not warrant on-demand delivery, which crosses a separate boundary.

Formal statement: The `N` counted by `countlinks_FTT(q, Σ) = N` are `N` distinct link addresses, and each address is permanent — valid at every reachable successor state (ASN-0093) with its stored value fixed (L12, ASN-0043). A count of `N` therefore warrants more than a tally: it warrants `N` durable identities answering `q`, each obtainable *in principle* because its handle never expires. This is the derived content the count carries beyond its cardinality; what it does *not* carry is on-demand delivery of those links, a separate concern across a separate boundary (out of scope here) subject to availability the count never speaks to.
