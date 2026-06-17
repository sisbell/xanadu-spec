> **ASN-0133 · Substrate Quiescence** — condensed claim statements  
> [← Full note](ASN-0133-substrate-quiescence.md) · [↑ Protocols index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0133 Claim Statements

*Source: ASN-0133-substrate-quiescence.md (revised unknown) — Extracted: 2026-06-15*

> **Note:** The ASN body does not include an explicit statement registry table. Types and constructs are inferred from in-body labels (H-*, Q-*, X-DEF, etc.) and proof structure.

---

## Definition — Rule

A *rule* is a triple `ρ = (D_ρ, T_ρ, Post_ρ)`: a domain expression `D_ρ ∈ QD`, a Boolean trigger `T_ρ : D_ρ → Bool` in PL, and an *emission contract* `Post_ρ` — a meta-level contract over (argument, state, emitted call set) constraining what any fire of ρ must emit, the calls drawn from the operation surface `{Emit_K, Nullify_Binary}`.

A *registry* `R` is a finite set of rules.

A *fire* of `(ρ, x)` at Σ with `x ∈ [D_ρ]_Σ` is: if `T_ρ(x, Σ) = ⊥`, a no-op (`Σ' = Σ`); otherwise the application of some emission set satisfying `Post_ρ(x, Σ, ·)` — a finite sequence of `→_sh` steps through the surface, `Σ →_sh* Σ'`.

---

## Definition — ExtinctionDisciplined

Rule ρ is *extinction-disciplined* iff for every reachable Σ and `x ∈ [D_ρ]_Σ` with `T_ρ(x, Σ) = ⊤`, every fire of `(ρ, x)` yields Σ' with `T_ρ(x, Σ') = ⊥`.

A registry is extinction-disciplined iff each rule is.

---

## Definition — Quiescent

`quiescent_R(Σ) ≡ (∀ ρ ∈ R :: (∀ x ∈ [D_ρ]_Σ :: ¬T_ρ(x, Σ)))`

The outer `∀ ρ ∈ R` is a finite static expansion into a PC0 conjunction (not a PC1 quantification — `R` is a finite metalevel set of rule-triples). Each conjunct `(∀ x ∈ [D_ρ]_Σ :: ¬T_ρ(x, Σ))` is a PL predicate: PC1 over a QD domain, finite at every reachable state (QD-fin), with `¬` by PC0.

---

## Definition — Work

`W(σ)` is the set of (rule, argument, index) triples at which a trigger is true along σ.

The load-bearing quantity is the *per-σ* count `|W(σ)|`.

---

## Definition — FairSequence

A fire sequence `σ = (Σ₀, s₁, Σ₁, s₂, …)` is *fair* iff for every `(ρ, x)` and every index `k` with `(ρ, x)` trigger-true at `Σ_k`, some later index `m > k` discharges that occurrence one of three ways:

- **real-fired**: a non-no-op fire of `(ρ, x)` at a step past `k`
- **removed**: `x ∉ [D_ρ]_{Σ_m}`
- **falsified in place**: `T_ρ(x, Σ_m) = ⊥` with `x ∈ [D_ρ]_{Σ_m}` still

This is the *per-occurrence* reading: every trigger-true index incurs its own later discharge, so a single earlier discharge cannot excuse an argument that *stays* trigger-true across a later tail.

---

## Definition — StronglyFairSequence

A fire sequence `σ` is *strongly fair* iff every `(ρ, x)` trigger-true at *infinitely many* indices along σ is *real-fired at infinitely many indices*.

*Regime form:* In the all-SF, extinction-disciplined regime over a non-grow-only domain, H-SFAIR holds iff *no* `(ρ, x)` is trigger-true at infinitely many indices.

---

## Definition — ScopeRestriction

A *scope* is any Boolean PL predicate `S` over addresses.

The scope's *restriction of ρ* is a QD filter `{x ∈ [D_ρ]_Σ : β_ρ^S(x)}` whose body `β_ρ^S` is a Boolean PL predicate at `D_ρ`'s sort relating the element to `S`, with *S-monotonicity* as the standing constraint: `β_ρ^S` must be monotone in `S` — `S` occurring only positively — so that `S' ⟹ S` yields `β_ρ^{S'} ⟹ β_ρ^S` pointwise.

The three canonical bodies:
- **per-emitter**: `S(addr(x))`, scoping a tuple by its own address
- **per-target**: `(∃ y ∈ addrs_G(x) :: S(y))`, scoping a tuple by its denoted targets; the domain is `addrs_G(x)` (finite), never `coverage_G(x)` (non-finite, not a QD domain)
- **per-source**: `(∃ y ∈ addrs_F(x) :: S(y))`

`quiescent_{R,S}(Σ)` restricts Q0's inner quantification to the elements this filter retains.

---

## H-HOME — HomeDischargeability (HYPO, requires)

Both surface emitters route through `K.λ_sh`, whose gate requires a registered home `d ∈ dom(Σ.M)` (ASN-0126/0128), and the surface excludes `K.σ` and `K.α` — so a fire can neither register its own home nor allocate its own content.

Each `Post_ρ` therefore directs its emissions to a home `d ∈ dom(Σ.M)`, and `Post_ρ`-satisfiability at a trigger-true `(x, Σ)` *presupposes* such a home — supplied by `Σ₀` or by a prior environment step (the environment, unlike a fire, may register documents through `K.σ`), never by the fire itself.

Where no registered home exists the fire has no admissible emission set, so the dischargeability of `Post_ρ` is itself a standing hypothesis.

---

## H-FIN — FireFiniteness (HYPO, requires)

Every `Post_ρ`-satisfying emission set is finite — equivalently, every admissible fire's *step run* (its own `→_sh` steps) terminates. The demand is universal over the contract's admissible choices.

---

## H-ATOM — FireAtomicity (HYPO, requires)

A fire's post-state `Σ'` is read *immediately* after the fire's own `→_sh` steps, with *no* environment step interleaved among them — environment steps fall *between* fires, never within one.

---

## H-FAIR — FairnessHypothesis (HYPO, requires)

A fire sequence σ from Σ₀ is *fair* (per Definition — FairSequence above).

---

## H-SFAIR — StrongFairnessHypothesis (HYPO, requires)

A fire sequence σ from Σ₀ is *strongly fair* (per Definition — StronglyFairSequence above).

`H-SFAIR ⟹ H-FAIR` for infinite σ.

For finite σ ending trigger-true: satisfies H-SFAIR vacuously yet violates H-FAIR's end-of-sequence obligation.

---

## H-RF — FiniteRealFires (HYPO, requires)

A fire sequence σ from Σ₀ has *finitely many real (non-no-op) fires*.

---

## Q0 — Recognizability (LEMMA, lemma)

`quiescent_R ∈ PL` for *every* registry — its value at every reachable Σ is decidable in finite time by any observer from Σ and the registry alone: one PL term, pure (PC4) and terminating (PC5), its evaluation finitely many verdicts.

*Single-view case:* the conjunction is formed as written.

*Heterogeneous-view case:* pay an explicit fixed-view-base rewrite — a change of spelling, not of value (by PC3's fixed-view rebuild equations and UV's default-view definition) — choosing top-level audit, which renders every constituent:
- PC3's four view-parameterized constituents (`members`, `targets_of`, `is_K`, `M_K`): audit and active values via PC3's `⋃`/`∃`-over-fixed-base device; default values via UV filter `{· : ¬filtered(·)}` over the active rebuild, with `filtered(·) ≡ (∃ J ∈ Φ, J ≠ K_queried :: is_filtered_J(·))`
- Three set-valued behavior collections (`succs`, `sources_to`, `stale`): fixed-view (active at every term view), default value the UV filter over the raw active reading
- `chain`: available unfiltered only by reading the atom directly at a non-default term view; its full-walk value is not rebuildable from any fixed-view base (C-reach, PC6a)
- Everything else (view-stable Booleans, verdict/optional atoms, fixed-view slices): one value at every term view

*Concurrency obligation:* Realized on a shared substrate, `quiescent_R` is in general a multi-read; for the verdict to be sound about the single state Σ, constituent reads must be pinned to one committed index — ASN-0134's reader-snapshot obligation (V2; MIC clause 6).

---

## Q1 — Absorption (LEMMA, lemma)

At any Σ with `quiescent_R(Σ)`, every fire of every `(ρ, x)` is a no-op, so `Σ' = Σ` — and the registry may halt its firing on first detection.

*Proof:* Immediate from RG's no-op clause and Q0's definition.

Recognizability and absorption are *unconditional* — independent of the dynamics hypotheses the termination results assume.

---

## Q2 — ContractOnOutputs (PROP, predicate)

Extinction discipline constrains emissions, not bodies: it is decided by `T_ρ` evaluations at the pre- and post-states — both public PL facts — so any two bodies with the same outputs are equivalent under it, and nondeterministic bodies are admissible whenever *all* their permitted outputs flip the trigger.

---

## Q3 — StaticCheckability (LEMMA, lemma)

If `Post_ρ` is *strong enough* — every emission set satisfying it at a trigger-true `(x, Σ)` produces a post-state falsifying `T_ρ(x, ·)` — then ρ is extinction-disciplined.

**Marker pattern (decidable-match case):** The trigger is the negated existential `¬(∃ c ∈ L_K :: a ∈ coverage_G(c))` over a grow-only audit slice `L_K`, and `Post_ρ` deposits *exactly the witness the `∃` quantifies over* — a `K`-tuple covering `a` — so the deposit grows `L_K`, the existential goes ⊤, the trigger goes ⊥, and "strong enough" is decided by a finite syntactic comparison of trigger spelling against emission form.

*idem=⊤ dedup caveat:* Firing reads `T_ρ(a, Σ) = ⊤`, so no `c ∈ L_K` covers `a`; a dedup hit would need an active — hence audit (`A_K ⊆ L_K`) — tuple covering it; therefore fire and dedup-hit cannot co-occur, the emit is necessarily a miss, deposits, and grows `L_K`. Even a born-nullified deposit joins `L_K` and flips the audit-read trigger.

*Not effective in general:* Outside the Marker pattern, the strong-enough condition quantifies over `(x, Σ)`: read over reachable trigger-true pairs it is reachability-quantified; read over all states it is a sound over-approximation of X-DEF but a PL-validity question this note neither shows decidable nor equips with a procedure.

---

## Q4 — Locality (PROP, predicate)

Extinction discipline is per-rule: its definition mentions no other rule. Registries compose pointwise — and precisely because the property is local, it cannot alone be sufficient for termination: locally disciplined rules can re-arm *each other* without bound.

---

## Q-EXT — ExtinctionByClass (LEMMA, lemma)

If `T_ρ` is an **SF spelling** (⊥-stable, PD0, ASN-0129), extinction discipline strengthens to *at-most-once firing per argument along any derivation*: the disciplined fire makes `T_ρ(x, ·)` false, and SF makes false permanent — no later `→_sh` step re-arms it, whether another rule's fire *or an environment step*, on any target (PD0's ⊥-stability is step-agnostic).

The count of real fires of ρ is then bounded by the total growth of `[D_ρ]` alone, with no analysis of other rules' emissions *or of the environment's*.

*Proof:* Immediate composition of X-DEF with PD0's ⊥-stability.

---

## Q-FLIP — FalsifierAccounting (PROP, predicate)

For triggers *not* in SF, what can re-arm them is the falsifier inventory ASN-0129's FP enumerates, read off with PD1/PD2:

1. A retraction shrinking an active slice the trigger reads
2. A BH1-type emission moving a default-view result
3. A BH4-footprint change from any deposit in a watched home
4. A *bare deposit growing an active slice*, which:
   - flips an `∃`-shaped active-view trigger `⊥→⊤` (PD1: `(∃ x ∈ M_K :: P(x))` at view active)
   - perturbs a non-monotone verdict atom such as `target_of`/`targets_keyed` (PD2)

The folklore rule "no retraction ⟹ each trigger flips at most once" is unsound against the shipped view machinery — a registry whose triggers read default views, age atoms, *or growing active slices* re-arms without any retraction.

*SF triggers are immune*: ⊥-stability (PD0) makes a falsified SF trigger permanent against every item above, deposits included.

---

## Q5 — RealFiresAreBounded (LEMMA, lemma)

For *every* fire sequence σ from Σ₀, with no registry-level hypothesis, the real (non-no-op) fires number at most `|W(σ)|`.

*Proof:* Each real fire at step k+1 witnesses `(ρ, x, k) ∈ W(σ)`, and the map real-fire ↦ `(ρ, x, k)` is injective *by the step index alone* (distinct real fires occupy distinct steps, so their triples differ in the index component). The injection bounds the count.

This bites exactly on σ's with `|W(σ)| < ∞`, and is vacuous (≤ ∞) on a σ in which a rule spins on a fixed `x`.

---

## Q5a — ExtinctionBound (LEMMA, lemma)

For an all-SF, *extinction-disciplined* registry (every trigger an SF spelling, every rule extinction-disciplined — instantiated by Marker-pattern rules (Q3)):

Real fires number at most `Σ_ρ |⋃_k [D_ρ]_{Σ_k}|`

— each argument fires each rule at most once (Q-EXT, consuming both the SF spelling and the extinction discipline), so the only unbounded-real-fire route is unbounded *new* arguments.

*SF alone does not bound the count:* an SF trigger paired with a contract that emits something other than its own falsifier stays ⊤ on a fixed argument forever (⊥-stability permits ⊤→⊤). Extinction discipline is the second required ingredient.

*Closed special case:* with the registry the only depositor, `bounded-domain-growth ⟺ H-RF`; Q5a says nothing H-RF did not.

*Open sequence:* `|⋃_k [D_ρ]_{Σ_k}| < ∞` bounds external input as much as internal enlargement; `bounded-domain-growth ⟹ H-RF` but not conversely.

---

## Q6 — TerminationUnderFairness (LEMMA, lemma)

**Preconditions:** H-RF, H-FAIR, H-FIN.

Under H-RF and H-FAIR, the registry's own drive to fire is exhausted after finitely many real fires — each a finite step run by H-FIN — leaving a registry-inert tail past which any residual non-quiescence is environment-driven.

**Reaching and holding, by hypothesis package:**

- **Regime (i) — + footprint-relevant state eventually constant** (`dom(Σ.M)`/`Σ.L` portions every `[D_ρ]` and `T_ρ` reads; FP): reached and held, for *any* registry.
- **Regime (ii), grow-only — + Q5a's package (all-SF, extinction-disciplined, bounded growth) with grow-only domains, under weak H-FAIR**: reached and held — the structural route, no environment-idle hypothesis.
- **Regime (ii), non-grow-only — + Q5a's package, non-grow-only**: bounds the real-fire count; *reaching* quiescence defers to an environment hypothesis for each non-grow-only rule — regime (i) for that rule directly, or strong fairness (H-SFAIR).

*Proof sketch:* H-RF bounds the real fires, so σ has a last real fire at index N (or none). By H-FIN each real fire is a finite step run, so the inert tail is reached in finitely many `→_sh` steps. No fire past N is real; every fire past N is a no-op (RG). Any `(ρ, x)` trigger-true at some `Σ_m`, m ≥ N, is by H-FAIR eventually real-fired (impossible past N without exceeding H-RF), or removed, or falsified in place — each past N coming from an *environment* step.

**Holding failure (non-grow-only, no extra hypothesis):** An environment oscillating one non-grow-only argument's domain membership re-presents it with all-empty gaps — quiescence *is reached* (each gap state satisfies `quiescent_R`) but not *held*.

**Reaching failure (non-grow-only, no extra hypothesis):** An environment cycling finitely many trigger-true arguments `{x₁, …, x_j}` out of phase — each `xᵢ` removed from its domain before its fire and re-presented so at every state at least one stands trigger-true — with H-RF (zero real fires), bounded growth (`⋃_k [D_ρ] = {x₁, …, x_j}`), and weak H-FAIR (each argument discharged by removal) all holding, yet *no* state quiescent.

H-SFAIR closes the reaching failure: by its regime form no `(ρ, x)` is trigger-true at infinitely many indices; with bounded growth the arguments are finitely many, so each has a maximum trigger-true index `N′`; past `max(N, N′)` `quiescent_R` holds, and every settled SF trigger is immune to re-arming.

---

## Q7 — ScopeRecognizability (LEMMA, lemma)

`quiescent_{R,S} ∈ PL` for *every* registry, decidable and observer-uniform:

The scope adds a PC1 filter `{x ∈ [D_ρ]_Σ : β_ρ^S(x)}` to each rule's inner quantification; the outer `∀ ρ ∈ R` is the same finite static expansion (V-IDX); the scope body `β_ρ^S` adds whatever view-sensitive constituents `S` reads to the rebuild Q0 already performs — so by Q0's fixed-view-base rewrite, `quiescent_{R,S} ∈ PL`.

The claim covers tuple-domained rules — `ρ_R` among them, whose `dom(Tup)` restriction is a well-typed QD filter for whichever scoping body it declares:
- per-emitter: `{c ∈ L_cmt : S(addr(c))}`
- per-target: `{c ∈ L_cmt : (∃ y ∈ addrs_G(c) :: S(y))}`

---

## Q8 — ScopeAbsorption (PROP, predicate)

At a scope-quiescent state:
- in-scope fires are no-ops (absorption relativizes)
- out-of-scope fires are unconstrained
- an out-of-scope emission may re-arm an in-scope trigger — the *scope specialization of re-entry* (Q1), the out-of-scope emission playing the environment's role relative to the in-scope sub-registry — which is itself detectable per-state by Q7

---

## Q9 — ScopeAntiMonotonicity (LEMMA, lemma)

For the S-monotone scoping bodies SC admits (standing constraint: `S` occurring only positively in `β_ρ^S`):

`S' ⟹ S` shrinks each filtered domain:

`{x ∈ [D_ρ] : β_ρ^{S'}(x)} ⊆ {x ∈ [D_ρ] : β_ρ^S(x)}` (by `β_ρ`'s S-monotonicity)

Quiescence over the smaller domain is the weaker demand, so:

`quiescent_S ⟹ quiescent_{S'}`

The converse fails whenever the larger scope holds work no smaller scope sees.

*Without S-monotonicity the nesting inverts:* a body with `S` under negation — `β_ρ^S(x) ≡ ¬S(addr(x))` — grows the filtered domain as `S` shrinks, so for the everywhere-true scope `S` and everywhere-false `S'` (whence `S' ⟹ S`), the `S`-filtered domain is empty (`quiescent_S` vacuously ⊤) while the `S'`-filtered domain is all of `[D_ρ]` (`quiescent_{S'}` possibly ⊥) — inverting the implication.
