> **ASN-0128 · Substrate Type Operational Semantics** — condensed claim statements  
> [← Full note](ASN-0128-substrate-type-operational-semantics.md) · [↑ Protocols index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0128 Claim Statements

*Source: ASN-0128-substrate-type-operational-semantics.md (revised unknown) — Extracted: 2026-06-11*

## Definition — AddressDenotation (AD)

A span is *address-denoting* iff it is unit-depth, `(x, δ(1, #x))`, and it *denotes* its start `x`. An endset `e` is address-denoting iff every span in it is, and its denoted set is `addrs(e) = {x : (x, δ(1, #x)) ∈ e}` — finite, since endsets are finite span sets (`Endset = 𝒫_fin(Span)`), and read off the spans by inspection. For a finite address set `X ⊆ T`, the *canonical encoding* is `enc(X) = {(x, δ(1, #x)) : x ∈ X}` — one canonical span per address, each T12-well-formed (`#x ≥ 1` by T0) — and `addrs(enc(X)) = X`.

The exposed `Emit_K` emits canonical encodings; `Nullify_Binary`'s fixed endsets are encodings of singletons — its from-fill `(d_retr, δ(1, #d_retr))` and its to-span `(a, δ(1, #a))` are both unit-depth — so every endset the surface emits is address-denoting by construction.

Two result regimes: *Membership predicates* (`is_K`, `is_filtered`) take an address argument and test coverage — `addr ∈ coverage(F)` — total over all tuples, address-denoting or not, decidable. *Enumeration predicates* (`members`, `targets_of`, `sources_to`, `succs`, `tip`, `chain`, `target_of`, `targets_keyed`) return denoted addresses, assembled from `addrs(·)` of the relevant endsets — each result finite and computable (L-fin many tuples, finitely many spans each). `is_in_chain` is Boolean-valued but enumeration-derived.

---

## Definition — ArgumentMatching (AM)

An argument naming a **source vertex** — `targets_of(x)`, `succs(x)`, `chain(addr)`, `tip(addr)`, `is_in_chain(addr, ·)`, `target_of(source, K)`, `targets_keyed(source)` — is matched by *exact denotation*: the tuples consulted are those whose F denotes the argument, `x ∈ addrs(F)`.

The family's one reverse lookup, whose argument names an **asserted-about address** — `sources_to(target)` — is matched by *coverage*: `target ∈ coverage(G)`.

(`members` takes no address argument; BH4's `age` takes an exact chain address and `stale` an ordinal horizon `h ∈ ℕ` — neither matches an endset.)

---

## R-C0 — RecordWellFormedness (PRE, requires)

A registration record is well-formed when `shape ∈ {Unary, Binary, Multi}`, `idem ∈ {⊤, ⊥}`, and every attached behavior is compatible with the rest of the record: read-filter (BH1) requires Unary; determinate-walk (BH2) and typed-reverse-lookup (BH3) require Binary; age-staleness (BH4) requires `idem = ⊥`, with no shape clause. R-C0 is enforced by failing construction (R-VAL). `Σ_init.registry` is well-formed entry-wise, on top of ASN-0126's C0 (finiteness, key uniqueness, representatives in `T_admissible`).

---

## R-VAL — ConstructionValidation (LEMMA, lemma)

Both well-formedness conditions are decidable at `Σ_init` construction, and construction is where the substrate checks them — the registry's only write point (ASN-0126, The registry). C0's key uniqueness is pairwise `CoverageEqualityDecidable` (ASN-0086) over the finitely many stored representatives — `O(|registry|²)` decidable tests, each on the representative endsets directly; membership of each representative in `T_admissible` is non-emptiness of a finite span set; and R-C0's clauses are finite case checks per entry (shape and idem against their enumerations, each attached behavior against the compatibility table). A declaration set failing any test yields no `Σ_init`: there is no partially-registered substrate, and no runtime path ever re-validates. What a passing declaration set yields: `Σ_init.C`, `Σ_init.M`, `Σ_init.L` are `Σ_init^{0086}`'s components verbatim, so in particular `Σ_init.L = ∅`.

---

## R-TR — ExtendedRecordTransition (DEF, function)

Over extended-record states the transition relation is

`→_sh ≡ K.σ ∪ K.α ∪ K.λ_sh`

with ASN-0126's step effects and preconditions throughout (its GatedTransitionRelation), the registry clauses read against the extended record: `K.λ_sh`'s precondition (0) — the deposited value is a standard triple — is unchanged; (i) "K is registered" is a key-side test, and the keys are unchanged; (ii) `Sh-conf(K, F, G)` reads the stored record's shape component, the only part of the value `Sh-conf` consults (ASN-0126, Shape-conformance); and each step kind's registry frame `Σ'.registry = Σ.registry` ranges over the triple-valued registry. `→_sh*` is the reflexive-transitive closure, and *reachable* means `→_sh*`-reachable from R-VAL's `Σ_init`.

---

## R1 — ExtendedRegistryInvariance (INV, invariant)

The extended record is constant at every `→_sh*`-reachable state (R-TR). ASN-0126's P1 argument is frame-based — every step kind carries `Σ'.registry = Σ.registry` in its frame (R-TR) and no step has the registry in its effect — and reads no part of the stored value, so it lifts to the extended value verbatim.

---

## R2 — IdemStability (INV, invariant)

For any registered K, `idem(K)` takes the same value at every reachable state — immediate from R1, exactly as ASN-0126's P2 (ShapeStability) follows for the shape component.

---

## RP — RecordProjection (LEMMA, lemma)

Let `ρ` act on states by keeping C, M, L and every registry key, and projecting each registry value `(shape, idem, behaviors) ↦ shape`. Then:

(i) `ρ(Σ_init)` is a well-formed ASN-0126 initial state — its C, M, L are ASN-0086's initial components by `Σ_init`'s construction (R-VAL), exactly what ASN-0126 adjoins its registry to; C0's clauses (finiteness, key uniqueness, representatives in `T_admissible`) are key-side and untouched; and every projected value is a bare shape.

(ii) Each `→_sh` step over extended-record states (R-TR) maps under `ρ` to an ASN-0126 `→_sh` step: the gate's verdict agrees, because `Sh-conf` reads `(F, G)` and the shape component, which `ρ` preserves; the C/M/L effects are identical; and each step kind's registry frame `Σ'.registry = Σ.registry` projects to the same frame.

(iii) By induction on derivation length, every reachable extended-record state `Σ` has `ρ(Σ)` ASN-0126-reachable.

---

## RP-a — RecordProjectionSingleStateTransfer (LEMMA, lemma)

An ASN-0126 claim whose conclusion is a predicate of one reachable state, reading only C, M, L, registration status (key-side, `ρ`-preserved), and shapes, holds at every reachable extended-record state Σ by evaluation at `ρ(Σ)`, ASN-0126-reachable by (iii) and sharing the read components.

---

## RP-b — RecordProjectionPathTransfer (LEMMA, lemma)

A claim whose conclusion constrains *steps or successors* transfers by derivation projection. By (ii) and induction, an extended-record derivation `Σ →_sh* Θ` projects to an ASN-0126 derivation `ρ(Σ) →_sh* ρ(Θ)`; single-state hypotheses hold at `ρ(Σ)` by sharing; the ASN-0126 claim applied there constrains `ρ(Θ)`; and the per-successor conclusion, again over shared components, pulls back to Θ. A transition invariant transfers the same way across each genuine extended-record step: the step's projection is the step it quantifies over.

---

## RP-c — RecordProjectionStepLift (LEMMA, lemma)

An ASN-0126 claim asserting the *existence* of a `→_sh` step lifts to the extended-record system when the step's preconditions read only `ρ`-preserved data: the preconditions hold at Σ iff at `ρ(Σ)`, the C/M/L effects are determined identically, and every step kind frames the registry — the asserted step is an extended-record step verbatim. P5 (GateRealizability) is the canonical instance.

---

## I0 — SamenessIsCoverageEquality (DEF, predicate)

Two emitted pairs `(F, G)` and `(F', G')` are *the same* for de-duplication iff `coverage(F) = coverage(F')` and `coverage(G) = coverage(G')` — `coverage` over endsets `Endset = 𝒫_fin(Span)` as ASN-0043 defines it — each equality decidable by CoverageEqualityDecidable (ASN-0086).

A strictly finer criterion is available — denoted-set equality, `addrs(F) = addrs(F') ∧ addrs(G) = addrs(G')` — but under it the active subset could hold coverage-equal tuples that no argument selects between. The finer criterion is rejected on that ground.

---

## I0a — MinimalElementsIdentity (LEMMA, lemma)

For an address-denoting endset `e`, the ≼-minimal elements of `coverage(e)` are exactly the ≼-minimal elements of `addrs(e)`.

*The separating pair:* an endset holding unit-depth spans at `t` and at an extension `t.x` denotes `{t, t.x}`, while one holding the span at `t` alone denotes `{t}` — the denoted sets differ, the coverages are equal by subtree absorption (`subtree(t.x) ⊆ subtree(t)`, PrefixSpanCoverage).

(⊆) Let `t` be ≼-minimal in `coverage(e)`. The coverage is the union of the denoted subtrees, so `r ≼ t` for some denoted `r`; `r` itself lies in the coverage (reflexivity of ≼), so `t`'s minimality forces `t = r`. And `t` is minimal among denoted addresses: a denoted `r' ≺ t` would lie in the coverage and contradict `t`'s minimality there.

(⊇) Let `r` be ≼-minimal among denoted addresses. Then `r` lies in the coverage, and is minimal there: a coverage element `t ≺ r` would lie in some denoted `r''`'s subtree, giving a denoted `r'' ≼ t ≺ r` and contradicting `r`'s minimality among denoted addresses.

An address-denoting endset's coverage thus determines and is determined by its ≼-minimal denoted addresses.

---

## I1 — IdemDedupSemantics (DEF, contract)

Under `idem(K) = ⊤`, the de-duplication check belongs to the *surface operation* `Emit_K(Σ, d, F, G)` and to it alone; the transition relation is R-TR's `→_sh`, unchanged. The contract has four clauses:

*Order — gate first.* The gate's preconditions (K registered and `Sh-conf(K, F, G)`; arity 3 holds by construction) are evaluated on the presented values before any dedup consultation; a gate-failing call is rejected — no step, no address — even when an I0-equal active tuple exists.

*Miss.* If no member of `A_K^Σ` is I0-equal to `(F, G)`, `Emit_K` invokes `K.λ_sh`, and ASN-0086's emit contract holds as inherited through the gate: a fresh `a = a_emit(Σ, d)`, `home(a) = d`, the deposit `(F, G, K)` at `a`, frame on C, M, and registry.

*Hit.* If some `(a', F', G') ∈ A_K^Σ` is I0-equal to `(F, G)`, `Emit_K` returns `a'` and **takes no step**: `Σ' = Σ`, nothing is deposited, and ASN-0086's emit postconditions are forfeited wholesale — `dom(Σ.L)` does not grow, no address is fresh, and the returned address's home is `home(a')`, which may differ from the caller's `d`. The `d` argument is consulted only on a miss. Along a K-surface-emitted derivation, the matching tuple is unique (I1a); off that discipline several may match, and `Emit_K` returns the T1-least matching address.

*Home validation — branch-local.* On a miss — the only branch that reads `d` — the check is enforced by rejection: a miss with `d ∉ dom(Σ.M)` takes no step and returns no address. On a hit, `d` is not read at all: the call is admitted and answered with the existing address whatever `d` holds.

---

## I1a — ActiveIdemUniqueness (LEMMA, lemma)

A derivation `Σ_init →_sh* Σ` is *K-surface-emitted* iff every `L_K`-growing step along it — every `Θ →_sh Θ'` with `L_K^Θ ⊊ L_K^{Θ'}` — is the deposit branch of an `Emit_K` invocation (no raw `K.λ_sh` bypass; for [R], the wrapper routes through `Emit_R`, so wrapper-routed steps qualify).

For `idem(K) = ⊤`, the active subset holds at most one tuple per I0-class at every state a K-surface-emitted derivation reaches:

`(A (a, F, G), (a', F', G') ∈ A_K^Σ : coverage(F) = coverage(F') ∧ coverage(G) = coverage(G') : a = a')`

*Proof* by induction along the K-surface-emitted derivation. Base: `L_K^{Σ_init} = ∅`, vacuous. Step: K.σ and K.α leave `Σ.L` unchanged. A `K.λ_sh` deposit of a K-tuple grows `L_K`, hence is by the derivation's classification the deposit branch of an `Emit_K` — which fires only on a miss: at the pre-state its I0-class had no active member, so at the post-state it has at most one — the deposit itself, when it lands active rather than born nullified. No tuple changes class post hoc — coverage is a pure function of the stored endsets, which are immutable.

---

## I2 — AuditSliceNotConsulted (LEMMA, lemma)

The de-duplication test reads `A_K^Σ`, not the audit slice `L_K^Σ`. A tuple emitted and later nullified is in `L_K^Σ` but not in `A_K^Σ`; a subsequent `Emit_K` with the same `(F, G)` is **not** a no-op — it attempts a fresh tuple at a new address. Resurrection-after-nullification is the design intent (ASN-0086's R6c, RestorationByReemission: a fresh tuple at a fresh address).

One caveat: the fresh address is the frontier slot of the home's link chain — `a_emit(Σ, d) = chain_d(f_d^Σ)` (FrontierUnification, ASN-0126, at extended-record states by RP-a) — and if a prior range-G retraction's coverage includes that slot, the resurrection emit is itself born nullified (Corollary RangeSterilization, ASN-0126; successor-quantified, so it crosses to extended-record states by RP-b). Resurrection is guaranteed only where the home chain's next slots are unsterilized.

---

## I3 — BornNullifiedTransparency (LEMMA, lemma)

ASN-0126's wp analysis separates the gate from the landing: a gate-clearing emit deposits into the audit slice `L_K^{Σ'}`, but lands in the active subset `A_K^{Σ'}` only if two further conjuncts hold — C2 (the emit is not a self-nullifying retraction) and C3 (no pre-existing retraction tuple's coverage includes the fresh address). Under either failure the tuple is *born nullified*: in `L_K^{Σ'}`, absent from `A_K^{Σ'}`.

By I2, a later `idem = ⊤` emit with the same `(F, G)` does not see a born-nullified tuple in its dedup check; it consults only the active subset, from which the born-nullified tuple is absent — whether the later emit misses outright turns on the rest of the class, since an intervening same-pair deposit may have landed active in the interim.

---

## SD — SurfaceDiscipline (DEF, predicate)

A derivation `Σ_init →_sh* Σ` is *surface-disciplined* iff every `L_R`-growing step along it — every `Θ →_sh Θ'` with `L_R^Θ ⊊ L_R^{Θ'}` — is a `Nullify_Binary` invocation. This is ASN-0086's RelationalLayer commitment ("every `→`-step with `L_R^Σ ⊊ L_R^{Σ'}` is a `Nullify`") read against `→_sh` and this note's wrapper.

Since the exposed `Emit_K` rejects every R-class call (the `K ≁ R` precondition, I6), the wrapper is the only surface route into `L_R`, so a derivation is surface-disciplined exactly when it is [R]-surface-emitted (I1a).

---

## DR — DisciplineRestoration (THEOREM, theorem)

**Statement.** Along a surface-disciplined derivation, no retraction tuple's to-coverage ever contains a later emit's fresh address: C3's existential is empty over `L_R^Θ` at every state Θ the derivation reaches, so wp conjunct C3 holds at every gate-clearing emit, a tuple is born nullified only through C2's self-nullification, and sterilization (Corollary RangeSterilization, ASN-0126) is unreachable.

**Wrapper wp.** *For single-tuple scope — the postcondition of ASN-0086's wp Case 1 — the weakest precondition at this surface is the operation's own:*

`wp(Nullify_Binary(Σ, d_retr, a), {t : a ≼ t} ∩ A_rel^{Σ'} = {a}) ≡ P0 ∧ P-reg ∧ P-tgt` — at every state a surface-disciplined derivation reaches (SD)

The equivalence is read under I6's attainability convention. *Necessity, at every reachable state:* a call failing P0 or P-reg is rejected — no step — so the wp is false by the convention alone; a call failing P-tgt (with P0 ∧ P-reg, so `a_emit(Σ, d_retr)` is defined) is likewise rejected, and ¬P-tgt gives `a ∉ A_rel^Σ = A_rel^{Σ'}`, so the intersection omits `a` and cannot equal `{a}`. *Sufficiency, on the disciplined domain:* every R-tuple at a state a surface-disciplined derivation reaches entered at a `Nullify_Binary` invocation with a P-tgt-valid target at its pre-state; by domain monotonicity (L12a per step, via RP-b) `a` remains a link address at every later state; for any later fresh address `f = a_emit(Θ, d) ∉ dom(Θ.L)` while `a ∈ dom(Θ.L)`, so `f ≠ a`; and R0a (FlatLinkDomain) at the post-state of that emit's `K.λ_sh` step forces `a ≼ f → a = f`, contradicting distinctness. Hence C3's existential is empty throughout the disciplined domain.

---

## I4 — ConcurrentEmitFirstCommit (LEMMA, lemma)

Two writers emitting tuples with same `(F, G, K)` against the same Σ race *ahead of* the substrate relation: `→_sh` is a sequential, interleaved step relation (it inherits ASN-0086's model), so a serializing authority orders the two calls before either becomes a step. The emission address is pinned per home to the chain frontier (FrontierUnification, ASN-0126), so allocation is first-to-commit.

For `idem = ⊤`: the winner deposits at the fresh address, and *provided the deposit lands active* (I3's caveat), the loser's emit, evaluated against the winner's post-state, finds the now-active tuple by I1 and returns the winner's address. If the winner's deposit is born nullified, the loser's dedup check sees an empty class (I2, I3) and the loser deposits a second tuple at the next frontier slot.

For `idem = ⊥`: both emissions produce distinct addresses, and both tuples appear in `A_K^{Σ₂}`, the second writer's post-state — modulo I3's born-nullified cases.

Where both calls extend a surface-disciplined derivation (SD), C3 cannot fail at either (DR) and C2 concerns only self-nullifying retraction emits, so for non-R types every deposit lands active and the unqualified first-to-commit reading is exact.

---

## I5 — IdemFalseAlwaysFresh (LEMMA, lemma)

Under `idem(K) = ⊥`, no de-duplication test runs, and home validation is unconditional: with no hit branch, every call reads `d`, and a call with `d ∉ dom(Σ.M)` is rejected — no step, no address — exactly as I1's miss branch rejects. Every gate-clearing call with `d ∈ dom(Σ.M)` invokes `K.λ_sh` and produces a fresh address regardless of `(F, G)` content, and the new tuple appears in `A_K^{Σ'}` (modulo I3's born-nullified cases).

*The exposed signature.*
`Emit_K : Σ × T × ℘_fin(T) × ℘_fin(T) ⇀ Σ' × A_rel^{Σ'}`
— a partial operation presenting `(d, X_F, X_G)`, whose emitted endsets are the canonical encodings `F = enc(X_F)`, `G = enc(X_G)` (AD). The home slot is `T` rather than `dom(Σ.M)` because home validation is branch-local (I1).

---

## I6 — IdemEmitSurfaceContract (LEMMA, lemma)

The caller-facing contract for `Emit_K` under `idem(K) = ⊤`, with its weakest precondition.

*Postcondition:*

`POST(a★) ≡ (E F★, G★ :: (a★, F★, G★) ∈ A_K^{Σ'} ∧ coverage(F★) = coverage(F) ∧ coverage(G★) = coverage(G))`

*Preconditions — uniform.* Three clauses checked first on every call (I1's order): the gate — K registered and `Sh-conf(K, F, G)`; arity 3 holds by construction — and the retraction exclusion `K ≁ R`. A call failing any clause is rejected, no step, no address.

*Branch condition.* The admitted call is a *hit* iff some `(a', F', G') ∈ A_K^Σ` is I0-equal to the presented pair, a *miss* otherwise.

*Hit.* No step: `Σ' = Σ`; returned address `a★ = a'`, the T1-least match (unique along K-surface-emitted derivations, I1a). POST holds at `Σ' = Σ` from the incumbent itself.

*Miss.* The branch reads `d` and rejects `d ∉ dom(Σ.M)`, contributing `d ∈ dom(Σ.M)`. An admitted miss takes the `K.λ_sh` step: fresh `a★ = a_emit(Σ, d)`, deposit `(F, G, K)` at `a★`. Landing active requires C2 (`K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G)`) — C2's first disjunct is the surface's own `K ≁ R` precondition, so C2 holds at every admitted call — and C3 (no pre-existing retraction tuple's to-coverage contains `a_emit(Σ, d)`).

*The wp, assembled:*

`wp(Emit_K under idem = ⊤, POST) ≡ pre ∧ (hit(Σ, F, G) ∨ (d ∈ dom(Σ.M) ∧ C3))`

(C2 is absorbed into `pre`: its first disjunct is the `K ≁ R` clause.)

*Disciplined-domain reduction.* At states reached by surface-disciplined derivations (SD) — and every derivation driven through this surface alone is one — the C3 conjunct vanishes (DR):

`wp(Emit_K, POST) ≡ pre ∧ (hit(Σ, F, G) ∨ d ∈ dom(Σ.M))`

*Corollary — the idem-⊥ wp.* Setting `hit ≡ ⊥`:

`wp(Emit_K under idem = ⊥, POST) ≡ pre ∧ d ∈ dom(Σ.M) ∧ C3`

reducing at states reached by surface-disciplined derivations to `pre ∧ d ∈ dom(Σ.M)`.

---

## Definition — ReadFilter (BH1)

**Applies to:** Unary

**Provides:** `is_filtered(addr) → Bool` — true iff some `(a, F, ∅) ∈ A_K^Σ` has `addr ∈ coverage(F)`: a membership predicate on the filter type's own active view, total and decidable.

**Rewrite scope.** Writing `Φ` for the set of types registered with BH1, each contributing its own `is_filtered_J` over its own active view:

- `members(K', default) = {x ∈ members(K', active) : (A J ∈ Φ, J ≠ K' : ¬is_filtered_J(x))}`
- `targets_of(x, default)` likewise drops the denoted targets so filtered

With a single BH1 registration: `{x ∈ members(K', active) : ¬is_filtered(x)}`.

The rewrite is *result-side only* — arguments are never filtered. Nothing else is rewritten: the active-view equations D1–D3 keep their values, the membership predicates are untouched, and raw `Observe_K` — both its hist and oper selectors — never filters.

---

## Definition — DeterminateWalk (BH2)

**Applies to:** Binary

**Provides:**
- `succs(x) → set of addrs` — the denoted targets of active K-tuples whose F denotes `x` (`x ∈ addrs(F)`): one step, no closure.
- `chain(addr) → ordered list of addrs` — the maximal *determinate* walk from `addr`: the longest sequence `addr = x₀, x₁, …, xₙ` with each `xᵢ₊₁` the unique successor (`succs(xᵢ) = {xᵢ₊₁}`) and all elements distinct. The walk stops at `xₙ` when `succs(xₙ) = ∅` (a sink), when `|succs(xₙ)| ≥ 2` (a branch), or when the unique successor already occurs in the sequence (a cycle). Termination: each extension appends a member of the finite vertex set `V`, pairwise distinct, so after `n` extensions `t = |V| − n ≥ 0` is a natural-number bound decreasing by one per extension.
- `tip(addr) → addr | ⊥` — `xₙ` when `chain(addr)` stops at a sink; `⊥` when the walk stops at a branch or a cycle.
- `is_in_chain(addr, target) → Bool` — `target ∈ chain(addr)`: membership in the walk's result list — exact denoted vertices, an enumeration-derived test, never a coverage test.

Chain-walking reads the active view, never the audit slice. BH1 filtering does not rewrite the walk.

---

## Definition — TypedReverseLookup (BH3)

**Applies to:** Binary

**Provides:**
- `sources_to(target) → set of addrs` — `⋃ { addrs(F) : (a, F, G) ∈ A_K^Σ ∧ target ∈ coverage(G) }`; the argument is tested by coverage; the result is denoted addresses (finite).
- `target_of(source, K) → addr | ⊥` — when exactly one active K-tuple's F denotes `source` *and* its G is address-denoting, the denoted target `y` (`addrs(G) = {y}`: one span by Binary, unit-depth by the address-denoting hypothesis); ⊥ in every other case — none, several, or a unique tuple whose single G-span is non-unit-depth.
- `targets_keyed(source) → map K → addr` — joins `target_of` across every Binary type K registered with BH3; ⊥-valued types are omitted, so the map's keys are exactly those K where `target_of(source, K)` is address-valued.

---

## Definition — AgeStaleness (BH4)

**Applies to:** any shape, with `idem = ⊥`

**Compatibility, derived.** The `idem = ⊥` clause is forced: under `idem = ⊤` an assertion's address is pinned to its first emission (I1's hit branch returns the incumbent), so `age(a)` would count from the original deposit however often re-asserted; `stale(h)` would classify by first utterance; `retract_stale` would retract assertions just re-asserted. Under `idem = ⊥` every admitted emit deposits at the frontier (I5): renewal restarts age at zero.

**Provides:** for an active event tuple at address `a`, `d = home(a)`. Every `a ∈ dom(Σ.L)` lies in its home's contiguous initial segment at a unique chain index — `a = chain_d(j)` for exactly one `j` (L-ContiguousPrefix, ASN-0086, at extended-record states via RP-a; uniqueness by strict T1-ascent of `chain_d` under sibling advance):

- `age(a) → ℕ` — `f_d^Σ − 1 − j`: how many deposits homed at `d` postdate the event. The chain interleaves every type homed at `d`, so age counts the home's subsequent link traffic, not K-events alone.
- `stale(h) → set of event-addrs` — `{a : (a, F, G) ∈ A_K^Σ ∧ age(a) > h}` for an ordinal horizon `h ∈ ℕ`; finite and computable (L-fin; age is arithmetic on chain indices). The horizon is *home-relative*: each `age(a)` is denominated in its own home's traffic; `stale` unions over every active K-tuple.
- `retract_stale(d_retr, h)` — for a caller-supplied retracting document `d_retr`: one `Nullify_Binary(·, d_retr, a)` per `a ∈ stale(h)`, the stale set evaluated once at the batch's initial state, `d_retr` held constant across the batch. P0 (`d_retr ∈ dom(Σ.M)`) is evaluated once at batch entry — on failure no constituent is issued. Once it holds there, it persists to every constituent pre-state by domain monotonicity. Net postcondition: `(A a : a ∈ stale(h) evaluated at batch entry : a ∈ nullified(Σ_fin))`.

---

## D1 — Members (DEF, function)

`members(K, active) → set of addrs` — the denoted F-addresses of the active K-tuples:

`members(K, active) = ⋃ { addrs(F) : (a, F, G) ∈ A_K^Σ }`

For Unary this is the set of marked addresses; for Binary and Multi it is the set of K-sources. Finite and computable: L-fin bounds the tuples, and each `addrs(F)` is read off finitely many spans (AD).

---

## D2 — IsK (DEF, predicate)

`is_K(addr) → Bool` — true iff some `(a, F, G) ∈ A_K^Σ` has `addr ∈ coverage(F)`: the membership regime, total and decidable (AD).

At *F-denoting states* (every `(a, F, G) ∈ A_K^Σ` has address-denoting F — which holds at every state a K-surface-emitted derivation reaches):

`is_K(addr) ⟺ (E x : x ∈ members(K, active) : x ≼ addr)`

The ⟸ direction holds at every state; ⟹ is what F-denotation buys. The selector is load-bearing: `is_K` is a membership predicate, never rewritten (BH1's Rewrite scope), so the bridge holds against `members`' `active` reading only.

---

## D3 — TargetsOf (DEF, function)

`targets_of(x, active) → set of addrs` — for a source address `x`, the denoted targets across the active K-tuples whose F denotes `x`:

`targets_of(x, active) = ⋃ { addrs(G) : (a, F, G) ∈ A_K^Σ ∧ x ∈ addrs(F) }`

For Unary, ∅ always. The argument is matched by denotation — `x ∈ addrs(F)`, per AM.

*Derived composition:*

`targets_under(addr) = ⋃ { targets_of(x, active) : x ∈ members(K, active) ∧ x ≼ addr }`

— finite and computable (D1's bounds, prefix tests componentwise). At states whose active K-slice is F-denoting (D2's hypothesis):

`targets_under(addr) = ⋃ { addrs(G) : (a, F, G) ∈ A_K^Σ ∧ addr ∈ coverage(F) }`

since for an address-denoting F: `addr ∈ coverage(F) ⟺ (E x : x ∈ addrs(F) : x ≼ addr)` (PrefixSpanCoverage).

---

## D4 — ReverseAccessIsBehavioral (LEMMA, lemma)

Target-side queries — "which sources point at a given target" — are not in the default set. They require the BH3 (typed-reverse-lookup) behavior, which provides `sources_to(target)`. The default trio reads only forward: source membership and source-to-target projection. Reverse access is a behavior-conditional capability, deliberately opt-in.

D1–D3 are well-defined on every reachable state by ASN-0086's active-subset definition, AD's finiteness, and R1 ensuring K's registration record is consulted identically at every state.

---

## R-C1 — DesignationNonCollision (PRE, requires)

The three designated classes are pairwise non-`~`-equivalent: `[K_ret] ≠ [K_sup]`, `[K_sup] ≠ [K_R]`, `[K_R] ≠ [K_ret]`. The three entries being mandatory, a colliding designation would violate C0's key uniqueness and no `Σ_init` — no substrate at all — could be constructed (enforcement: C0, R-VAL).

---

## S1 — Retired (DEF, definition)

`retired` (the designated class `[K_ret]`) — **Unary, idem=⊤, behaviors={BH1}**.

Marks an address as lifecycle-retired: the default view on every other type excludes it from enumeration results, with the exclusion's coverage scope per BH1's Effect and Rewrite scope. Active subsets are untouched — nothing is nullified, and the marked content remains reachable under the `active` selector.

---

## S2 — Supersedes (DEF, definition)

`supersedes` (the designated class `[K_sup]`) — **Binary, idem=⊤, behaviors={BH2}**.

Slot convention: a tuple asserts that the address its F denotes *is superseded by* the address its G denotes — edges run old → new. `succs` thus steps from a version to its claimed replacements, and `tip()` resolves to the current head when the active supersession edges from the queried address form a determinate, acyclic walk, and to ⊥ at a branch or a cycle (BH2's verdicts).

---

## S3 — Retraction (DEF, definition)

`R` (the designated class `[K_R]`, ASN-0086's RetractionType) — **Binary, idem=⊤, behaviors=∅**.

The retraction relation. Live retraction operation: the unit-depth wrapper

`Nullify_Binary(Σ, d_retr, a) ≡ Emit_R(Σ, d_retr, {r}, {(a, δ(1, #a))})`

with canonical from-fill `r = (d_retr, δ(1, #d_retr))`, preconditions P0 (`d_retr ∈ dom(Σ.M)`) and P-reg ([R] registered Binary) and P-tgt (`a ∈ A_rel^Σ ∨ a = a_emit(Σ, d_retr)`) — enforced as rejecting preconditions at this surface. The exposed `Emit_K` rejects every R-class call by its uniform `K ≁ R` precondition (I6); `Nullify_Binary` is the one retraction entry point.

Enforcing P-tgt means: a call failing P-tgt (or P0, or P-reg) takes no `→_sh` step — the state is unchanged and no address is returned.

*P-tgt uniform check:* The wrapper's from-fill puts `subtree(d_retr)` into its I0 identity, so `d_retr` is read on both branches; P0 is evaluated on every call.

*Branch condition.* Hit iff some `(b', F', G') ∈ A_R^Σ` has `coverage(F') = subtree(d_retr)` and `coverage(G') = subtree(a)`.

*Miss.* Fresh emitter `b = a_emit(Σ, d_retr)`, `home(b) = d_retr`, deposit `({r}, {(a, δ(1, #a))}, R)`, `A_rel^{Σ'} = A_rel^Σ ∪ {b}`; and P-tgt being enforced, the iff-P-tgt postconditions hold outright: `a ∈ nullified(Σ')`, coverage nullification, single-tuple scope `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}`, and persistence.

*Hit.* **No step**: `Σ' = Σ`; the wrapper returns the existing tuple's address (unique by I1a under a surface-disciplined derivation). From the pre-existing tuple: `a ∈ nullified(Σ)` (R6b hypotheses: the matching tuple in `A_R^Σ ⊆ L_R^Σ` with `a ∈ coverage(G')`; `a ∈ A_rel^Σ` by P-tgt residence enforcement); single-tuple scope `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}` (⊇ by residence; ⊆ by R0a antichain); persistence by R6a.
