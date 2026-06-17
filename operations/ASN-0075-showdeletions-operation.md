> **ASN-0075 · SHOWDELETIONS Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md)  
> [Condensed statements →](ASN-0075-showdeletions-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0075: SHOWDELETIONS Operation

*2026-05-25*

Nelson lists "show deletions" among the operations the system must provide (LM 4/79). The intuition is direct: given two documents that share content history, identify the content that was present in one but is absent from the other. We approach this abstractly. We do not specify how documents come to share history, nor how content is removed from an arrangement — those mechanics belong elsewhere. We specify only what the operation must produce, what guarantees it must offer over its output, and what state it consults.

The central difficulty is that two situations are observationally indistinguishable without further information: content `a` may be absent from document `d`'s arrangement because `d` once contained `a` and removed it (it was *deleted*), or because `d` was never an arrangement that contained `a` (it was *never included*). A "show deletions" operation must distinguish these. We will show that the provenance relation `R` introduced in the transition model supplies exactly the information required, and that any conforming implementation must therefore maintain state components — beyond `(C, L, E, M)` collectively — sufficient to disambiguate the two predicates `DELETED(a, d)` and `NEVER_INCLUDED(a, d)` at every reachable state. Without such components, deletion is not detectable as a kind separate from prior absence.

## Foundation Recap

We take from the foundation:

- **Content store** `Σ.C : T ⇀ Val` (ASN-0036, S0): a partial function from tumblers to content values, append-only with immutable values across transitions.
- **Arrangement** `Σ.M(d) : T ⇀ T` (ASN-0036, S2, S3, S8a, S8-depth): a per-document partial function from V-positions to I-addresses.
- **Entity set** `Σ.E ⊆ T` and its document partition `Σ.E_doc` (ASN-0047).
- **Provenance relation** `Σ.R ⊆ T_elem × E_doc` (ASN-0047), where `T_elem = {a ∈ T : Element(a)} ⊆ T` uses the foundation's element predicate `Element(·)` (ASN-0047). Its historical reading is supplied by P4★ and P4a (recapped below), not stipulated here.
- **Provenance permanence** `R ⊆ R'` across transitions (P2, ASN-0047): once `(a, d) ∈ R`, it remains so.
- **Provenance bounds** `Contains_C(Σ) ⊆ R` (P4★, ASN-0047): if `a` is currently in `d`'s content-subspace arrangement, then `(a, d) ∈ R`.
- **Historical fidelity** (P4a, ASN-0047): if `(a, d) ∈ R`, some prior reachable state had `a` in `d`'s content-subspace arrangement.
- **Provenance grounding** `R ⊆ dom(C) × E_doc` (P7, ASN-0047): every provenance pair references content that exists.
- **Origin function** `origin(a)` (ASN-0036, S7): every `a ∈ dom(C)` has a uniquely determined originating document, invariant across states.
- **Subspace projection** `subspace_I(a) = E(a)₁` (ASN-0047, SubspaceConventionAxiom): identifies the content (`s_C`) or link (`s_L`) subspace of an I-address.
- **Subspace convention** `s_C = 1, s_L = 2` (ASN-0047, SubspaceConventionAxiom).
- **Link subspace ownership** (CL-OWN, ASN-0047): link-subspace V-positions of `d` map only to link I-addresses with `origin = d`.

## The Three States of Content

We classify each pair `(a, d)` with `a ∈ dom(C)` and `d ∈ E_doc` into one of three states. Every `a ∈ dom(C)` already has `subspace_I(a) = s_C` (ASN-0047, ContentAllocationSubspacePrecondition; equivalently by L0), so we do not carry that conjunct in the predicates and sets below.

```
CURRENT(a, d)         ≡  a ∈ ran(M(d))
DELETED(a, d)         ≡  (a, d) ∈ R  ∧  a ∉ ran(M(d))
NEVER_INCLUDED(a, d)  ≡  (a, d) ∉ R
```

**Classification is at I-address-set granularity.** Each predicate reads only the *set* membership condition `a ∈ ran(M(d))`, never the number of V-positions at which `a` occurs. The foundation permits a single I-address to occupy multiple V-positions within one document: ASN-0036 (S5, M13) and ASN-0058 (M14) establish `(E d, a :: |{v : M(d)(v) = a}| > 1)` — transclusion may map the same content at several V-positions. The consequence is explicit and intended: if `d` references `a` at two V-positions and one is removed, `a ∈ ran(M(d))` still holds, so `CURRENT(a, d)` holds and `DELETED(a, d)` does not. A per-occurrence removal — distinguishing which of several V-positions holding the same I-address went away — is therefore *invisible* to this classification while any occurrence of `a` survives in `d`, and as a Vstream concern that our I-address-set predicates do not address, we scope it out of this operation.

**Lemma D-WIT (Content Witness Forces Provenance).** Let `Σ` be a composite-boundary state. For every `a ∈ dom(Σ.C)` and `d ∈ Σ.E_doc`, if `a ∈ ran(M(d))` then `(a, d) ∈ R`.

*Proof.* From `a ∈ ran(M(d))`, fix `v ∈ dom(M(d))` with `M(d)(v) = a`. From `a ∈ dom(Σ.C)`, L14 (`dom(C) ∩ dom(L) = ∅`) gives `a ∉ dom(L)`. By S3★-aux, `subspace(v) ∈ {s_C, s_L}`. The contrapositive of S3★'s link clause — `subspace(v) = s_L ⟹ M(d)(v) ∈ dom(L)` — together with `M(d)(v) = a ∉ dom(L)` forces `subspace(v) ≠ s_L`, so `subspace(v) = s_C`. Then `v` witnesses `v ∈ dom(M(d)) ∧ subspace(v) = s_C ∧ M(d)(v) = a`, so `(a, d) ∈ Contains_C(Σ)` by definition, and `Contains_C(Σ) ⊆ R` (P4★, ASN-0047). Hence `(a, d) ∈ R`. ∎

**Lemma D-EXH (Three-State Exhaustion).** Let `Σ` be a state reachable from `Σ_0` by a finite sequence of valid composite transitions (equivalently, `Σ` is a composite boundary). For every `(a, d)` with `a ∈ dom(Σ.C)` and `d ∈ Σ.E_doc`, exactly one of `CURRENT(a, d)`, `DELETED(a, d)`, `NEVER_INCLUDED(a, d)` holds.

*Proof.* The three predicates correspond to three of the four cases of the cross-product `(a ∈ ran(M(d))) × ((a, d) ∈ R)`:

| `a ∈ ran(M(d))` | `(a, d) ∈ R` | Predicate |
|---|---|---|
| Yes | Yes | CURRENT |
| Yes | No | impossible |
| No  | Yes | DELETED |
| No  | No  | NEVER_INCLUDED |

The "impossible" row is excluded by D-WIT: from `a ∈ ran(M(d))` and the lemma's hypothesis `a ∈ dom(Σ.C)`, D-WIT gives `(a, d) ∈ R`, contradicting the row's `(a, d) ∉ R`.

For each remaining row, label assignment is direct from the predicate definitions:

- Row 1 (`a ∈ ran(M(d)) ∧ (a, d) ∈ R`): CURRENT holds (definition); DELETED fails (`a ∈ ran(M(d))` falsifies its second conjunct); NEVER_INCLUDED fails (`(a, d) ∈ R`).
- Row 3 (`a ∉ ran(M(d)) ∧ (a, d) ∈ R`): DELETED holds (both conjuncts); CURRENT fails (`a ∉ ran(M(d))`); NEVER_INCLUDED fails (`(a, d) ∈ R`).
- Row 4 (`a ∉ ran(M(d)) ∧ (a, d) ∉ R`): NEVER_INCLUDED holds (definition); CURRENT fails (`a ∉ ran(M(d))`); DELETED fails (`(a, d) ∉ R` falsifies its first conjunct).

The four-row table is total over the two binary conditions, row 2 is excluded by D-WIT, and the bullets above assign exactly one label per remaining row. ∎

## Why the Provenance Relation Is Load-Bearing

We now show that the four foundation state components `(C, L, E, M)` are insufficient to support SHOWDELETIONS.

**Lemma D-DISCR (Discrimination Requires Provenance).** No function computable from `(Σ.C, Σ.L, Σ.E, Σ.M)` alone can distinguish `DELETED(a, d)` from `NEVER_INCLUDED(a, d)` for arbitrary `(a, d)`.

*Argument.* We exhibit two reachable states `Σ_1` and `Σ_2` for which `(Σ.C, Σ.L, Σ.E, Σ.M)` agree across every document but `DELETED(a, d)` and `NEVER_INCLUDED(a, d)` disagree.

*Notational convention.* In the histories below, each `→*` arrow denotes one valid composite under ValidComposite★ (ASN-0047); line breaks are visual aids only.

Two composite abbreviations recur. *Document creation:* K.δ case (ii) with `k = 2` (descent) requires `t ∈ E ∧ zeros(t) ≤ 1`. From `Σ_0` (whose only entity is the bootstrap node `n_0`, `zeros(n_0) = 0`), a single elementary K.δ step produces at most an account (`zeros = 1`); minting the first document (`zeros = 2`) requires a precursor account-creation step. We therefore write `K.δ(d) ≡ K.δ(A); K.δ(d)` as shorthand *for the first document only*, where `A = inc(n_0, 2)` is the account and `d = inc(A, 2)` is the document; any subsequent document is a version fork `d' = inc(d, 1)` (K.δ case (ii), `k = 1`), needing only `d ∈ E_doc`. *Content introduction:* each content-introduction composite bundles K.α (allocate `a`), K.μ⁺ (place `a` in an arrangement), and K.ρ (record the provenance pair); the bundle discharges the freshness obligation and the J0, J1★, and J1'★ couplings, so each such composite is valid under ValidComposite★.

Both histories begin at the initial state `Σ_0` (ASN-0047) and share the prefix `K.δ(d); K.δ(d')` — creating two documents `d, d'`. Both then invoke K.α(a, d) to allocate one content address. By K.α's first-emission rule (`{a' ∈ dom(C) : origin(a') = d} = ∅` initially), the allocated address is determinately `a = [d.0.s_C.1]` — a value fixed by `d` alone. Both histories pass the same `d` to the first-emission predicate, so both yield the same allocated address `a`. We further stipulate that both histories pass the *same* `v ∈ Val` argument to K.α — call it `v_a` — so that `C_1(a) = C_2(a) = v_a` and the content-store agreement in the table below holds at the value level. We fix the content-subspace V-position depth at `m_C = 2` throughout both histories — admissible because ValidFirstInsertionPosition (ASN-0036) treats `m` as operational input with `m ≥ 2` — giving `v = [s_C, 1] = [1, 1]` in `M(d)` and `v' = [s_C, 1] = [1, 1]` in `M(d')` as the canonical D-MIN★ first positions for each document's initially-empty content subspace. The histories then differ in where `a` is placed and which provenance pairs are recorded.

*History 1 (yields DELETED).*

```
Σ_0  →* K.δ(d)
     →* K.δ(d')                          [d' = inc(d, 1), version fork]
     →* K.α(a, d);   K.μ⁺(d,  v  ↦ a);  K.ρ(a, d)
     →* K.μ⁺(d', v' ↦ a);  K.ρ(a, d')
     →* K.μ⁻(d)              [retain n'_{s_C} = 0]
     =   Σ_1
```

The third composite places `a` in `M(d)` and records `(a, d) ∈ R'`. The fourth composite extends `M(d')` with the same `a` at `v' = [s_C, 1]` and records `(a, d')`. The K.μ⁻ step on `d` retains zero content-subspace V-positions (`n'_{s_C} = 0`), removing `v ↦ a` from `M(d)`; by P2 (`R ⊆ R'`), `(a, d) ∈ R_1` persists. Final state: `dom(C_1) = {a}`, `M_1(d) = ∅`, `M_1(d') = {v' ↦ a}`, `(a, d) ∈ R_1`. So `DELETED(a, d)` holds at `Σ_1`.

*History 2 (yields NEVER_INCLUDED).*

```
Σ_0  →* K.δ(d)
     →* K.δ(d')                          [d' = inc(d, 1), version fork]
     →* K.α(a, d);   K.μ⁺(d', v' ↦ a);  K.ρ(a, d')
     =   Σ_2
```

The third composite places `a` in `M(d')` (J0 requires placement in *some* document's arrangement, not specifically the origin's) and records `(a, d') ∈ R_2`; `d` is never extended with `a`, so `(a, d) ∉ R_2`. Final state: `dom(C_2) = {a}`, `M_2(d) = ∅`, `M_2(d') = {v' ↦ a}`, `(a, d) ∉ R_2`. So `NEVER_INCLUDED(a, d)` holds at `Σ_2`.

*Agreement on (C, L, E, M).* Comparing the components of `Σ_1` and `Σ_2`:

| Component | `Σ_1` | `Σ_2` |
|---|---|---|
| `dom(C)` | `{a}` | `{a}` |
| `C` value at `a` | the K.α-supplied value `v_a` | same |
| `L` | `∅` | `∅` |
| `E` | `{n_0, …, d, d'}` | `{n_0, …, d, d'}` |
| `E_doc` | `{d, d'}` | `{d, d'}` |
| `M(d)` | `∅` | `∅` |
| `M(d')` | `{v' ↦ a}` | `{v' ↦ a}` |

Neither history invokes K.λ, so `L_1 = L_2 = ∅`. Both histories execute the same K.δ sequence to create `d` and `d'`, so `E_1 = E_2` (entities are permanent by P1, and no entity-creating step distinguishes the two). `(Σ_1.C, Σ_1.L, Σ_1.E, Σ_1.M) = (Σ_2.C, Σ_2.L, Σ_2.E, Σ_2.M)` on every component. The histories differ only in `R`: `R_1 ⊇ {(a, d), (a, d')}` and `R_2 ⊇ {(a, d')}`, with `(a, d) ∈ R_1 \ R_2`.

Any function `f(C, L, E, M)` returns the same value at both states. But the classifications differ — `DELETED(a, d)` at `Σ_1`, `NEVER_INCLUDED(a, d)` at `Σ_2` — so `f` cannot be a discriminating predicate. ∎

**Corollary D-NEED (Auxiliary State Necessity).** Any system supporting SHOWDELETIONS must maintain at least one state component beyond `(C, L, E, M)` whose value disambiguates `DELETED(a, d)` from `NEVER_INCLUDED(a, d)` at every reachable state.

*Argument.* D-DISCR shows that the four foundation components `(C, L, E, M)` cannot distinguish DELETED from NEVER_INCLUDED, which SHOWDELETIONS' output sets require; hence a conforming system must carry at least one further component `C*`. `DELETED(a, d)` and `NEVER_INCLUDED(a, d)` differ on `R`-membership by their very definitions, and that difference does not depend on `Σ` being a composite boundary. So `R` disambiguates the two predicates at every reachable state, not merely at the composite boundaries where D-WIT and D-EXH operate — `C* = R` suffices throughout.

## The SHOWDELETIONS Operation

Let `d_A, d_B ∈ E_doc`. The operation takes two documents and observes the state. We define the asymmetric output sets:

```
DeletedFromAWithB(d_A, d_B)
   =  {a ∈ dom(C) :
         DELETED(a, d_A)
       ∧ CURRENT(a, d_B)}

DeletedFromBWithA(d_A, d_B)
   =  {a ∈ dom(C) :
         DELETED(a, d_B)
       ∧ CURRENT(a, d_A)}
```

Each asymmetric set captures content deleted from one document and still arranged in the other — the still-current copy in the partner document is the *witness* that makes the deletion observable as recoverable. The `DELETED` conjunct — specifically its provenance requirement `(a, d_A) ∈ R` — is what keeps the report from conflating deletion with addition. A naive set-difference of current ranges, `ran(M(d_B)) \ ran(M(d_A))`, would mix content that `d_A` once held and deleted with content that `d_B` merely acquired and `d_A` never received; requiring `(a, d_A) ∈ R` excludes the latter, since never-included content satisfies `NEVER_INCLUDED(a, d_A)` rather than `DELETED(a, d_A)`.

**Definition (SHOWDELETIONS).** The operation is the ordered pair:

```
SHOWDELETIONS(d_A, d_B)
   =  (DeletedFromAWithB(d_A, d_B), DeletedFromBWithA(d_A, d_B))
```

The two halves are necessarily disjoint. Membership in `DeletedFromAWithB` requires `CURRENT(a, d_B)`, i.e. `a ∈ ran(M(d_B))`; membership in `DeletedFromBWithA` requires `DELETED(a, d_B)`, whose second conjunct is `a ∉ ran(M(d_B))`. The two range-membership conditions on `M(d_B)` are directly contradictory, so no `a` can belong to both halves.

**Boundary precondition (D-BOUND).** The pre-state `Σ` is reachable from `Σ_0` by a finite sequence of valid composite transitions under ValidComposite★ (ASN-0047), so SHOWDELETIONS is invoked at a composite boundary.

The operation's precondition is `d_A ∈ E_doc ∧ d_B ∈ E_doc ∧ Σ is a composite-boundary state`. Its postcondition characterises the result set-theoretically. We capture this in wp form. Let `q` abbreviate the predicate:

```
Result = (DeletedFromAWithB(d_A, d_B), DeletedFromBWithA(d_A, d_B))
```

The predicate `q` is pure set equality: the operation returns the two comprehensions by definition, so establishing `q` requires only that they are computable — `d_A, d_B ∈ E_doc` and the finiteness of `dom(C)` and the arrangements. That finiteness (C-fin, S8-fin) holds at *every* reachable state, not only at composite boundaries. Hence the genuine weakest precondition for `q` carries no boundary conjunct:

```
wp(SHOWDELETIONS(d_A, d_B), q)  =  d_A ∈ E_doc  ∧  d_B ∈ E_doc
```

The operation always terminates with `q` true when this holds, since each output set is a comprehension over the finite `dom(C)` (C-fin, ASN-0047) whose membership tests are bounded by the finite arrangements (S8-fin, ASN-0036) and the finite relation `R ⊆ dom(C) × E_doc` (P7, ASN-0047).

The operation's *stated* precondition (D-BOUND) is strictly stronger than `wp(op, q)`: it adds the composite-boundary conjunct, which is not needed to compute `q` but is load-bearing for the report's *meaning* — D-WIT and D-EXH hold only at composite boundaries (resting on P4★, which couples `Contains_C` into `R` only across complete composites), so the boundary requirement is what licenses the three-state exhaustion underwriting `DELETED`.

Because SHOWDELETIONS writes no state component (D-OBS), wp computations for state-level predicates pass through unchanged from the pre-state: for any `P` depending only on `Σ`, `wp(SHOWDELETIONS, P) = d_A ∈ E_doc ∧ d_B ∈ E_doc ∧ P(Σ)`. Two state-level postconditions are worth deriving explicitly, since they characterise *when* the operation surfaces structurally meaningful facts.

*Non-emptiness of one report half.* Let `Q1` abbreviate `DeletedFromAWithB(d_A, d_B) ≠ ∅`. Unpacking the definition of `DeletedFromAWithB`:

```
wp(SHOWDELETIONS(d_A, d_B), Q1)
   =  d_A ∈ E_doc  ∧  d_B ∈ E_doc
    ∧  (E a ∈ dom(C) :  (a, d_A) ∈ R
                       ∧ a ∉ ran(M(d_A))
                       ∧ a ∈ ran(M(d_B)))
```

So `DeletedFromAWithB` is non-empty exactly when some content address inhabits `d_A`'s history through `R`, has been removed from `d_A`'s current arrangement, and remains in `d_B`'s current arrangement.

*Vacuity of both report halves.* Let `Q0` abbreviate `DeletedFromAWithB(d_A, d_B) = ∅ ∧ DeletedFromBWithA(d_A, d_B) = ∅`. `Q0` is a state-level predicate over `M`, `R`, `dom(C)`, so by the general rule above the wp formula is the well-definedness condition conjoined with `Q0` unpacked at the pre-state:

```
wp(SHOWDELETIONS(d_A, d_B), Q0)
   =  d_A ∈ E_doc  ∧  d_B ∈ E_doc
    ∧  (A a ∈ dom(C) :
            ¬(DELETED(a, d_A)  ∧  CURRENT(a, d_B))
          ∧ ¬(DELETED(a, d_B)  ∧  CURRENT(a, d_A)))
```

The joint report is empty exactly when no content has been deleted from one document while remaining current in the other.

**Lemma D-DISJ (Disjoint Provenance Empties the Report).** Documents with disjoint `R`-projections — `{a : (a, d_A) ∈ R} ∩ {a : (a, d_B) ∈ R} = ∅` — satisfy `Q0` at any composite-boundary state `Σ`. *Proof.* `Q0` requires every `a ∈ dom(C)` to falsify *both* conjuncts `DELETED(a, d_A) ∧ CURRENT(a, d_B)` (conjunct 1) and `DELETED(a, d_B) ∧ CURRENT(a, d_A)` (conjunct 2). Partition `dom(C)` into three groups by `R`-projection membership, and show each group falsifies both conjuncts.

*Group 1: `(a, d_A) ∈ R`.* Disjointness gives `(a, d_B) ∉ R`. For conjunct 1, `CURRENT(a, d_B)` requires `a ∈ ran(M(d_B))`; D-WIT would then give `(a, d_B) ∈ R`, contradicting `(a, d_B) ∉ R`. So `CURRENT(a, d_B)` fails and conjunct 1 is falsified. Conjunct 2 is falsified more directly: `DELETED(a, d_B)` has first conjunct `(a, d_B) ∈ R`, which `(a, d_B) ∉ R` negates outright.

*Group 2: `(a, d_B) ∈ R`.* By the symmetric argument, disjointness gives `(a, d_A) ∉ R`. Conjunct 2's `CURRENT(a, d_A)` is excluded by D-WIT applied to `d_A` (it would force `(a, d_A) ∈ R`), falsifying conjunct 2; and conjunct 1's `DELETED(a, d_A)` fails directly because its first conjunct `(a, d_A) ∈ R` is negated by `(a, d_A) ∉ R`.

*Group 3: neither `(a, d_A) ∈ R` nor `(a, d_B) ∈ R`.* The address is classified `NEVER_INCLUDED` against both documents; `DELETED(a, d_A)` and `DELETED(a, d_B)` both fail on their first conjuncts, so both conjuncts are falsified trivially.

The three groups are exhaustive (disjointness rules out membership in both `R`-projections, so no fourth group arises). Every `a ∈ dom(C)` falsifies both conjuncts, and `Q0` holds. ∎

## Restriction to the Content Subspace

**Claim D-SUBSP.** SHOWDELETIONS operates only over the content subspace (`s_C`).

*Justification.* Both output sets are subsets of `dom(C)`. Every `a ∈ dom(C)` has `subspace_I(a) = s_C` (established in "The Three States of Content"), and `dom(C) ∩ dom(L) = ∅` by L14, so no link address can ever appear in an output. The restriction to the content subspace is thus immediate from `output ⊆ dom(C)`.

The content/link asymmetry is what makes cross-document deletion comparison meaningful only over `s_C`. Content-subspace addresses can be shared between documents, so one document can serve as the still-current witness for another's deletion. Link material cannot — by CL-OWN (ASN-0047), `subspace(v) = s_L ∧ M(d)(v) = a` forces `origin(a) = d`, so a document's link-subspace V-positions reference only its own link addresses and no comparison document ever holds another's link as witness.

## Identity Preservation

**Claim D-IDENT.** For every `a` in either output set, the returned reference is precisely the I-address `a` — not a copy with new identity.

*Justification.* The output sets are defined as subsets of `dom(C)`. Each element is an existing I-address. We return addresses, not values. Since the output is a set of existing I-addresses, any other reference to `a` — a link endset or another document's arrangement entry — names the same `a`, with no aliasing or shadow copy introduced by the operation.

## Origin Traceability

**Claim D-ORIG.** For every `a` in either output set, `origin(a)` is determined and identifies a unique document — the originating allocator of `a`.

*Justification.* By S7 (ASN-0036), `origin(a)` is defined for every `a ∈ dom(C)` and is invariant across all states in which `a ∈ dom(C)`. The output sets are subsets of `dom(C)`, so `origin` is well-defined on every output element. The output need carry no extra "origin annotation" beyond the address itself — origin is derived structurally from the address.

## Order Availability

**Claim D-ORD.** Each output half is a finite subset of `dom(C) ⊆ T`, hence linearly ordered by the restriction of T1 (ASN-0034) to that subset. The operation carries no ordering of its own — it takes no input ordering to preserve and emits a set; T1-orderability is a property of the output addresses, not a structure the operation transports.

*Justification.* The output sets are subsets of `dom(C)`, finite by C-fin (ASN-0047), and T1 is a strict total order on `T` (ASN-0034). The restriction of a total order to a finite subset is again a total order, so each half is linearly ordered by its own addresses with no appeal to any document's arrangement.

We note explicitly what is *not* recoverable: the V-position order in which a deleted address appeared in the document from which it was removed. V-position information is local to a current arrangement and is not preserved by `R`, so the deleted document's "original ordering" is a property of an arrangement no longer present. The witness document's arrangement does impose a V-order on the still-current addresses, but that order is observable only through the witness and is not part of the abstract output.

## Symmetry

**Claim D-SYM.** Argument swap maps each output half into the other:

```
SHOWDELETIONS(d_A, d_B)  =  (X, Y)
SHOWDELETIONS(d_B, d_A)  =  (Y, X)
```

where `X = DeletedFromAWithB(d_A, d_B)` and `Y = DeletedFromBWithA(d_A, d_B)`.

*Justification.* By name-substitution in the definitions: `DeletedFromAWithB(d_B, d_A)` reads as "addresses with `DELETED(a, d_B) ∧ CURRENT(a, d_A)`," which is exactly `DeletedFromBWithA(d_A, d_B)`. Likewise the other half.

The content-level guarantee — the union of both halves as a set of I-addresses — is therefore symmetric in the operands. The presentation labelling (which half is "from A" vs. "from B") swaps accordingly.

## A Worked Example

We illustrate SHOWDELETIONS on the canonical scenario: a document is forked, and the two siblings diverge by each deleting different content.

*Setup.* Begin at `Σ_0` (the initial state of ASN-0047). Using the first-document shorthand `K.δ(d_A) ≡ K.δ(A); K.δ(d_A)` established in D-DISCR (with `A = inc(n_0, 2)` and `d_A = inc(A, 2)`), apply the composite

```
Σ_0  →* K.δ(d_A)                                             [precursor: K.δ(A); K.δ(d_A)]
     →* K.α(a, d_A);  K.μ⁺(d_A, [1,1] ↦ a);  K.ρ(a, d_A)
     →* K.α(b, d_A);  K.μ⁺(d_A, [1,2] ↦ b);  K.ρ(b, d_A)
     →* K.α(c, d_A);  K.μ⁺(d_A, [1,3] ↦ c);  K.ρ(c, d_A)
     →* K.δ(d_B)                                                  [d_B = inc(d_A, 1)]
     →* K.μ⁺(d_B, [1,1] ↦ a, [1,2] ↦ b, [1,3] ↦ c);  K.ρ(a, d_B);  K.ρ(b, d_B);  K.ρ(c, d_B)
     →* K.μ~(d_A)  [permute so c at [1,2], b at [1,3]]
     →* K.μ⁻(d_A)  [retain n'_{s_C} = 2 of content subspace]
     →* K.μ⁻(d_B)  [retain n'_{s_C} = 2 of content subspace]
     =   Σ
```

The first four lines create `d_A` with three content addresses `a, b, c` (all with `origin = d_A` by S7), arranged at `[1,1], [1,2], [1,3]`; the per-line K.ρ steps record the corresponding provenance, with each `K.μ⁺; K.ρ` bundle following the bundle pattern stated above. Line 5 forks `d_A` to `d_B = inc(d_A, 1)` (K.δ case (ii), `k = 1`); line 6 populates `d_B` by transclusion — the *same* I-addresses `a, b, c` are referenced from `d_B`'s V-positions — and records the three accompanying provenance pairs in one composite. The resulting provenance relation contains `R ⊇ {(a, d_A), (b, d_A), (c, d_A), (a, d_B), (b, d_B), (c, d_B)}`.

The last three lines effect a divergent edit. Lines 7–8 reorder `M(d_A)` to put `b` at the trailing position `[1,3]` and then truncate, removing `b` from `d_A`'s arrangement. Line 9 removes `c` from `d_B`'s arrangement directly — no prior rearrangement is needed because `c` is already at the trailing position `[1,3]`, so K.μ⁻ retaining the first two content positions drops exactly `c`. By P2, the deletions leave `R` unchanged.

*Resulting state.*

| Component | Value |
|---|---|
| `dom(C)` | `{a, b, c}` |
| `origin` | `origin(a) = origin(b) = origin(c) = d_A` |
| `E_doc` | `{d_A, d_B}` |
| `M(d_A)` | `{[1,1] ↦ a, [1,2] ↦ c}` |
| `M(d_B)` | `{[1,1] ↦ a, [1,2] ↦ b}` |
| `R ⊇` | `{(a, d_A), (b, d_A), (c, d_A), (a, d_B), (b, d_B), (c, d_B)}` |

*Classifying each pair.* For each of the six pairs `(x, d) ∈ {a, b, c} × {d_A, d_B}`, D-EXH yields a unique classification:

| Pair | `x ∈ ran(M(d))?` | `(x, d) ∈ R?` | Class |
|---|---|---|---|
| `(a, d_A)` | yes | yes | CURRENT |
| `(b, d_A)` | no  | yes | DELETED |
| `(c, d_A)` | yes | yes | CURRENT |
| `(a, d_B)` | yes | yes | CURRENT |
| `(b, d_B)` | yes | yes | CURRENT |
| `(c, d_B)` | no  | yes | DELETED |

*Computing the output.*

```
DeletedFromAWithB(d_A, d_B)  =  {x ∈ dom(C) : DELETED(x, d_A) ∧ CURRENT(x, d_B)}  =  {b}
DeletedFromBWithA(d_A, d_B)  =  {x ∈ dom(C) : DELETED(x, d_B) ∧ CURRENT(x, d_A)}  =  {c}
SHOWDELETIONS(d_A, d_B)       =  ({b}, {c})
```

Only `b` is deleted from `d_A` while remaining in `d_B`; only `c` is deleted from `d_B` while remaining in `d_A`. The shared content `a` is current in both and reported in neither half.

*Verifying the claims on this state.*

- *D-EXH.* The classification table assigns each pair exactly one class; mutual exclusion is by construction (DELETED requires `x ∉ ran(M(d))`, CURRENT requires `x ∈ ran(M(d))`).
- *D-IDENT.* The returned `b` is the same I-address that inhabits `dom(C)` and `ran(M(d_B))` — no value has been copied into a new tumbler. The same holds for `c`.
- *D-ORIG.* `origin(b) = origin(c) = d_A`, derivable from the tumblers themselves via S7. The output addresses self-identify their allocator.
- *D-SYM.* Applying the definition with operands swapped, `DeletedFromAWithB(d_B, d_A) = {x : DELETED(x, d_B) ∧ CURRENT(x, d_A)} = {c}` and `DeletedFromBWithA(d_B, d_A) = {b}`. So `SHOWDELETIONS(d_B, d_A) = ({c}, {b})` — the component-swap of `SHOWDELETIONS(d_A, d_B)`.

The example also illustrates the structural significance of the witness: `b` is reported as deleted from `d_A` only because `d_B` still holds it; if `d_B` had also deleted `b`, the pair `(b, d_A)` would still be DELETED, but `b` would not appear in `DeletedFromAWithB` because the witness condition `CURRENT(b, d_B)` would fail. Cross-document SHOWDELETIONS exposes exactly the asymmetric losses — deletions that one document made and the other did not.

## Observational Frame

**Claim D-OBS.** SHOWDELETIONS does not modify any state component.

Formally, for state `Σ = (C, L, E, M, R)` and the state `Σ'` obtaining after the operation:

```
Σ'.C  =  Σ.C
Σ'.L  =  Σ.L
Σ'.E  =  Σ.E
Σ'.R  =  Σ.R
(A d ∈ E_doc ::  Σ'.M(d) = Σ.M(d))
```

The operation reads `M(d_A)`, `M(d_B)`, and `R`; it computes the output sets; it returns them. It allocates nothing, rewrites nothing, and invokes no transition relation — observationality is immediate from the definition, which is a pair of set-builder comprehensions over `Σ`.

Consequences: SHOWDELETIONS is repeatable on the same state (yields identical results). Because the operation is observational, its result is merely delivered to the caller and is not stored as a document or otherwise integrated into the persistent store (**D-STORE**); the system creates no persistent artefact of its own accord.

## State-Functional Independence

**Claim D-RECONS.** The output depends only on the current state `Σ`. It does not depend on the particular sequence of transitions by which `Σ` was reached.

*Justification.* Each predicate `CURRENT`, `DELETED`, `NEVER_INCLUDED` is defined in terms of components of `Σ` only (`M`, `R`, `dom(C)`, `subspace_I`). The output sets are characterised entirely by these projections. Two distinct transition histories yielding the same `Σ` therefore yield identical SHOWDELETIONS outputs.

P4a (historical fidelity, ASN-0047) ensures that whenever the operation reports `DELETED(a, d)`, there really was a past state where `a` was in `d`'s arrangement — but the *route* to that past state is irrelevant to the report itself.

## Edge Cases

*Documents with no shared content.* Disjoint `R`-projections yield `Q0` (both halves empty) by D-DISJ.

*Both arrangements empty.* If `dom(M(d_A)) = dom(M(d_B)) = ∅`, then `ran(M(d_A)) = ran(M(d_B)) = ∅`, so `CURRENT` fails for every `a` on both sides. Both halves are empty.

*Same document compared against itself.* If `d_A = d_B`, then for each `a`, `DELETED(a, d_A) ∧ CURRENT(a, d_A)` is contradictory directly: `DELETED(a, d_A)` requires `a ∉ ran(M(d_A))` while `CURRENT(a, d_A)` requires `a ∈ ran(M(d_A))`, and these two range-membership conditions cannot both hold. This is the unconditional disjointness argument of the SHOWDELETIONS definition specialised to a single document. Both halves are empty. The operation is well-defined and trivially yields the empty pair.

*Asymmetric population.* If `d_A` has rich history (large `R`-projection) but its current arrangement is empty, while `d_B`'s arrangement currently holds many of the addresses `d_A` historically held, then `DeletedFromAWithB` may be large and `DeletedFromBWithA` may be empty. The asymmetry of the two halves directly mirrors the asymmetry of the editing histories.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| CURRENT | `CURRENT(a, d) ≡ a ∈ ran(M(d))` | introduced |
| DELETED | `DELETED(a, d) ≡ (a, d) ∈ R ∧ a ∉ ran(M(d))` | introduced |
| NEVER_INCLUDED | `NEVER_INCLUDED(a, d) ≡ (a, d) ∉ R` | introduced |
| D-WIT | At a composite-boundary state, `a ∈ dom(C) ∧ a ∈ ran(M(d)) ⟹ (a, d) ∈ R` | introduced |
| D-EXH | For every composite-boundary state Σ (reachable by valid composite transitions) and every `(a, d)` with `a ∈ dom(Σ.C)`, `d ∈ Σ.E_doc`, exactly one of CURRENT, DELETED, NEVER_INCLUDED holds | introduced |
| D-DISCR | No function of `(C, L, E, M)` alone can distinguish DELETED from NEVER_INCLUDED for arbitrary `(a, d)` | introduced |
| D-NEED | Any system supporting SHOWDELETIONS must maintain at least one state component `C*` beyond `(C, L, E, M)` whose value disambiguates DELETED from NEVER_INCLUDED at every reachable Σ; `C* = R` suffices | introduced |
| DeletedFromAWithB | `{a ∈ dom(C) : DELETED(a, d_A) ∧ CURRENT(a, d_B)}` | introduced |
| DeletedFromBWithA | Symmetric counterpart of DeletedFromAWithB | introduced |
| SHOWDELETIONS | Observational operation `SHOWDELETIONS(d_A, d_B) = (DeletedFromAWithB(d_A, d_B), DeletedFromBWithA(d_A, d_B))` | introduced |
| D-BOUND | SHOWDELETIONS is invoked at composite-boundary states | introduced |
| D-DISJ | At a composite-boundary state, documents with disjoint content-subspace `R`-projections satisfy `Q0` (both report halves empty) | introduced |
| D-SUBSP | The operation restricts to the content subspace `s_C`; cross-document deletion comparison is structurally meaningful only there | introduced |
| D-IDENT | Output references are I-addresses themselves; no copies, no new identities | introduced |
| D-ORIG | Every output element `a` has determinate `origin(a)` | introduced |
| D-ORD | Each output half is a finite subset of T, hence linearly ordered by T1's restriction; the operation carries no ordering of its own | introduced |
| D-SYM | `SHOWDELETIONS(d_B, d_A)` is the component-swapped pair of `SHOWDELETIONS(d_A, d_B)` | introduced |
| D-OBS | SHOWDELETIONS modifies no state component; it is purely observational | introduced |
| D-STORE | The output is not required to be stored as a document; it is a query result | introduced |
| D-RECONS | The output depends only on the current state, not on transition history | introduced |

## Open Questions

What abstract characterisation of "shared content history" between two documents, expressed solely in terms of R, predicts when SHOWDELETIONS will yield non-empty results?

When deleted content has been removed from every document that ever contained it, through what state component does the system still retain the option to expose it for query or recovery?

How should SHOWDELETIONS report content that was deleted from both compared documents but remains current in a third document not in the pair?

If the system supports concurrent state transitions, what consistency model must SHOWDELETIONS observe to deliver coherent joint snapshots of M and R?

How does SHOWDELETIONS generalise to families of more than two documents, and what witness-structure replaces the binary asymmetric pair?

Under what conditions on the witness arrangement does the deletion set admit a finite presentation as a union of contiguous I-address spans, and when must it enumerate addresses singly?

What guarantees must the witness's V-order satisfy to ensure that presentation-ordered output of SHOWDELETIONS corresponds to a user-meaningful reading sequence rather than a structural accident?

Should the system distinguish content "deleted with a witness in a prior arrangement of the same document" from "deleted with a witness in a sibling document," and what additional structure would that distinction require?

What must a restoration operation guarantee so that consuming a subset of a SHOWDELETIONS output reintroduces deleted content into a target arrangement while preserving origin and link-resolvability?
