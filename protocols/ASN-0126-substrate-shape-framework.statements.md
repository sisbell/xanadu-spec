> **ASN-0126 · Substrate Shape Framework** — condensed claim statements  
> [← Full note](ASN-0126-substrate-shape-framework.md) · [↑ Protocols index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0126 Claim Statements

*Source: ASN-0126-substrate-shape-framework.md (revised unknown) — Extracted: 2026-06-10*

## Definition — ShapeConformance

The predicate `Sh-conf(K, F, G)` is defined only for *registered* K. For a typed tuple `(F, G, K)` under a type K registered with shape s, `Sh-conf(K, F, G)` holds when:

- Unary: `|F| = 1` and `G = ∅` (equivalently `|G| = 0`);
- Binary: `|F| = 1` and `|G| = 1`;
- Multi: `|F| = 1` and `|G| < ∞`.

For an unregistered K, `shape(K)` does not exist and `Sh-conf(K, F, G)` carries no truth value.

---

## Definition — GatedTransitionRelation

`→_sh ≡ K.σ ∪ K.α ∪ K.λ_sh`

The step `K.λ_sh` is `K.λ` with three added preconditions: (0) the emitted value is a standard triple (arity 3); (i) K is registered; and (ii) `Sh-conf(K, F, G)`. Frame conditions per step kind:

- K.σ: `Σ'.C = Σ.C`, `Σ'.L = Σ.L`, `Σ'.registry = Σ.registry`
- K.α: `Σ'.M = Σ.M`, `Σ'.L = Σ.L`, `Σ'.registry = Σ.registry`
- K.λ_sh: `Σ'.C = Σ.C`, `Σ'.M = Σ.M`, `Σ'.registry = Σ.registry`

---

## Definition — Reachability

`Σ →_sh* Σ'` is the reflexive-transitive closure of `→_sh`; when it holds we call Σ' a *`→_sh*`-successor* of Σ. A state is *`→_sh*`-reachable* iff `Σ_init →_sh* Σ`, with `Σ_init` the registry-adjoined initial state.

---

## Definition — ForgetfulProjection

`π(Σ) = (Σ.C, Σ.M, Σ.L)` — the forgetful projection that drops the registry, returning an ASN-0086 three-component state. The framework constructs `Σ_init` by adjoining the registry to ASN-0086's three initial components, altering none of them: `π(Σ_init) = Σ_init^{0086}`.

---

## Definition — RegistryWellFormedness

A registry is well-formed when every stored representative lies in `T_admissible`, shape values lie in `{Unary, Binary, Multi}`, and coverage-class keys are unique: no two entries have `~`-equal keys.

---

## Definition — ShapeFunction

`shape(K)` denotes the shape the well-formed registry — a partial function of coverage classes — records for `[K]`; so `shape(K)` depends only on `[K]`, defined exactly on the registered coverage classes.

---

## Definition — HomedChain

`chain_d(j) = incʲ(d.0.s_L.1, 0)` for `j ≥ 0` — the enumeration in which L-ContiguousPrefix presents the homed-set: at every `→*`-reachable state the homed-set `{a' ∈ dom(Σ.L) : home(a') = d}` is the initial segment `{chain_d(j) : 0 ≤ j ≤ J_d^Σ}` for some `J_d^Σ ∈ ℤ_{≥-1}`, with unique T1-maximum `chain_d(J_d^Σ)` when non-empty. The frontier is `f_d^Σ = J_d^Σ + 1`.

---

## Definition — NullifyBinary

`Nullify_Binary(Σ, d_retr, a) ≡ Emit_R(Σ, d_retr, {r}, {(a, δ(1, #a))})`, canonical from-fill `r = (d_retr, δ(1, #d_retr))`.

*Preconditions.* **P0**: `d_retr ∈ dom(Σ.M)`. **P-reg**: [R] is registered Binary. The target `a` carries no precondition — `(a, δ(1, #a))` is T12-well-formed for any tumbler; P-tgt in particular is *not* assumed, it conditions the postconditions.

*Effect.* `Σ →_sh Σ'` deposits `({r}, {(a, δ(1, #a))}, R)` at the fresh emitter `b = a_emit(Σ, d_retr)` — `b ∉ dom(Σ.L)`, `b ∈ dom(Σ'.L)`, `home(b) = d_retr`, `Σ'.L(b) = ({r}, {(a, δ(1, #a))}, R)` — with Σ' itself `→_sh*`-reachable.

*Frame.* `Σ'.C = Σ.C`, `Σ'.M = Σ.M`, `Σ'.registry = Σ.registry`.

*Postconditions.* The step adjoins exactly the fresh key, so `A_rel^{Σ'} = A_rel^Σ ∪ {b}`; then:

- *Coverage nullification, unconditional.* `{t : a ≼ t} ∩ A_rel^{Σ'} ⊆ nullified(Σ')`.
- *Target nullification iff residence.* `a ∈ nullified(Σ') ⟺ a ∈ A_rel^{Σ'} ⟺ P-tgt at Σ` — where P-tgt is: `a ∈ A_rel^Σ ∨ a = a_emit(Σ, d_retr)`.
- *Single-tuple scope iff P-tgt.* `{t : a ≼ t} ∩ A_rel^{Σ'} = {a}` holds iff P-tgt holds at Σ.
- *Persistence.* `nullified(Σ') ⊆ nullified(Θ)` at every `→_sh*`-successor Θ of Σ'; and for any `K ∈ T_admissible` and any `(a', F', G') ∈ L_K^{Σ'}` with `a' ∈ nullified(Σ')`, `(a', F', G') ∉ A_K^Θ` at every `→_sh*`-successor Θ.

---

## C0 — RegistryWellFormedness (CONSTRAINT, predicate)

`Σ_init.registry` is well-formed (above) and finite: `|Σ_init.registry| < ∞`.

---

## P1 — RegistryInvariance (INV, predicate)

At every `→_sh*`-reachable state, `Σ.registry = Σ_init.registry` — the registry never drifts.

*Proof sketch.* By induction on the length of a `→_sh*`-derivation: the base case `Σ = Σ_init` is immediate, and each step preserves `Σ.registry = Σ_init.registry` by the frame condition for whichever of the three step kinds it is.

---

## P2 — ShapeStability (INV, predicate)

For any *registered* K, `shape(K)` takes the same value at every `→_sh*`-reachable state: since `shape(K)` is read from the P1-invariant registry, the same K cannot carry one shape at Σ and another at Σ'.

---

## P3 — ShConfWellFormedness (INV, predicate)

Every value a `→_sh`-step adjoins to `dom(Σ.L)` is a standard triple `(F, G, K)` whose K is registered and for which `Sh-conf(K, F, G)` holds.

*Proof.* `K.λ_sh` is the only step kind that extends `dom(Σ.L)`, and (0), (i), (ii) are among its preconditions — so every deposited value is a standard triple of arity 3 by (0), with K registered by (i) and conforming to K's shape by (ii).

---

## P4 — ShConfStateIndependence (INV, predicate)

For any *registered* K, any F, G, and any reachable Σ, Σ', `Sh-conf(K, F, G)` is defined at both states and its verdict at Σ equals its verdict at Σ'.

*Derivation.* `Sh-conf` respects `~`: for `K ~ K'` it reads only the span counts `|F|`, `|G|` and `shape(K) = shape(K')`, so `Sh-conf(K, F, G) = Sh-conf(K', F, G)`. It is stable because for registered K it reads only `(F, G)` and `shape(K)`, and since `shape(K)` is stable across states (P2), `Sh-conf(K, F, G)` evaluates the same against Σ as against any Σ' reachable from Σ. Registration status is itself state-independent by P1 — K is registered at Σ iff registered at Σ' — so the predicate is defined at Σ exactly when it is defined at Σ'.

---

## Lemma — ProjectionBridge (LEMMA, lemma)

`π` maps every `→_sh`-step to an ASN-0086 `→`-step, and hence every `→_sh*`-reachable state of this framework to a state `→*`-reachable from ASN-0086's initial state.

*Proof.* Each `→_sh`-step preserves the registry in its frame and acts on the C/M/L components exactly as the corresponding ASN-0086 step: a K.σ-step as `K.σ`, a K.α-step as `K.α`, and a `K.λ_sh`-step as a `K.λ` step — the last by effect-identity, its C/M/L action being `K.λ`'s. Hence whenever `Σ →_sh Σ'`, `π(Σ) → π(Σ')` in ASN-0086's relation. By induction on derivation length, `π` maps every `→_sh*`-reachable state to a state `→*`-reachable from ASN-0086's initial state: at the base, `π(Σ_init) = Σ_init^{0086}`, trivially `→*`-reachable from itself; and each step extends a `→`-derivation by the projected step. ∎

---

## B1 — SharedComponents (LEMMA, lemma)

Σ and `π(Σ)` share their C, M, and L components. Every ASN-0086 state-indexed function reads only C/M/L, so each agrees at Σ and `π(Σ)`, and is thereby well-defined on this note's four-component states by evaluation at the projection, `f(Σ) := f(π(Σ))`. In particular: `a_emit(π(Σ), d) = a_emit(Σ, d)` and `dom(π(Σ).L) = dom(Σ.L)`; consequently `A_rel^{π(Σ)} = A_rel^Σ`.

---

## B2 — LemmaTransfer (LEMMA, lemma)

Take any ASN-0086 result whose conclusion is a predicate over the C/M/L components of a single `→*`-reachable state. For each state Σ this note reasons about, `π(Σ)` is `→*`-reachable (ProjectionBridge), so the result holds at `π(Σ)`; since it constrains only the shared C/M/L components, its conclusion transfers to Σ directly.

*Scope restriction.* B2 yields no `→_sh`-successors: an existence-of-successor conclusion `∃ Σ' : Σ → Σ' ∧ …` transfers only to a `→`-successor of `π(Σ)`, which need not lift to a `→_sh`-step of Σ. A transition invariant of ASN-0086 (quantified over `→`-steps, e.g. L12) transfers only across a genuine `→_sh`-step `Σ →_sh Σ'`.

---

## B3 — PathTransfer (LEMMA, lemma)

Take any ASN-0086 result whose hypotheses are predicates over the C/M/L components of a single `→*`-reachable state and whose conclusion asserts, at every `→*`-successor of that state, again a predicate over C/M/L components. Such a result transfers: whenever its hypotheses hold at a `→_sh*`-reachable Σ, its conclusion holds at every `→_sh*`-successor Θ of Σ.

*Proof sketch.* ProjectionBridge's step mapping gives, by induction on the derivation `Σ →_sh* Θ`, `π(Σ) →* π(Θ)`; the hypotheses hold at `π(Σ)` (B1); the result applied at `π(Σ)` reaches its `→*`-successor `π(Θ)`; B1, applied at the `→_sh*`-reachable Θ, carries the per-successor conclusion from `π(Θ)` back to Θ.

*Exclusion.* Layer-scoped results do not transfer: ProjectionBridge delivers `→*`-reachability, never layer-reachability.

---

## Lemma — RegisteredAdmissible (LEMMA, lemma)

Every registered K satisfies `K ∈ T_admissible`.

*Proof.* By C0 the registry stores, for K's coverage class, a finite representative endset `K_j ∈ T_admissible`, and "K registered" means `coverage(K) = coverage(K_j)`. `K_j ∈ T_admissible` is non-empty, so it contains a span `(s, ℓ)`, T12-well-formed — `ℓ > 0` with action point at most `#s`. These are exactly the hypotheses of TA-strict (StrictIncrease, ASN-0034), giving `s < s ⊕ ℓ`, hence `s ∈ {t : s ≤ t < s ⊕ ℓ} ⊆ coverage(K_j)`, so `coverage(K_j) ≠ ∅`; hence `coverage(K) = coverage(K_j) ≠ ∅`, so `K ≠ ∅`, i.e. `K ∈ T_admissible`. ∎

---

## P5 — GateRealizability (LEMMA, lemma)

For any `→_sh*`-reachable Σ, any `d ∈ dom(Σ.M)`, any registered K, and any `F, G ∈ Endset` with `Sh-conf(K, F, G) = ⊤`, there exists Σ' with `Σ →_sh Σ'` depositing the standard triple `(F, G, K)` at the fresh address `a = a_emit(Σ, d)`:

`a ∉ dom(Σ.L) ∧ a ∈ dom(Σ'.L) ∧ Σ'.L(a) = (F, G, K) ∧ home(a) = d`,

with Σ' itself `→_sh*`-reachable.

---

## P6 — ReachableConformance (INV, predicate)

For every `→_sh*`-reachable Σ and every `a ∈ dom(Σ.L)`, the stored value `Σ.L(a)` is a standard triple `(F, G, K)` whose K is registered and for which `Sh-conf(K, F, G) = ⊤`.

*Derived* by induction on derivation length, with P6 as the induction hypothesis at the predecessor state. The base `Σ_init.L = ∅` holds vacuously. For the tuple a step newly deposits: P3 supplies all three conjuncts at the pre-state; the shape conjunct crosses unaided (property of the value alone); P1 carries registration across the step; P4 carries the conformance verdict. For a tuple already present: L12 (via B2), P1, and P4 give persistence of shape, registration, and conformance respectively.

---

## WP — GatedEmitWeakestPrecondition (LEMMA, lemma)

`wp(Emit under →_sh, (a, F, G) ∈ A_K^{Σ'})`  
`≡`  
`K registered ∧ Sh-conf(K, F, G) ∧ d ∈ dom(Σ.M) ∧ (K ≁ R ∨ a_emit(Σ, d) ∉ coverage(G)) ∧ ¬(∃ (b, F', G') ∈ L_R^Σ :: a_emit(Σ, d) ∈ coverage(G'))`

where the attainability reading is in force: `wp(g → S, R) ≡ g ∧ wp(S, R)` — on `¬g` nothing fires, so the wp is false. Precondition (0) (arity-3 triple), `K ∈ T_admissible` (RegisteredAdmissible), and key freshness (R0) are discharged by construction and contribute no wp conjunct.

---

## Lemma — FrontierUnification (LEMMA, lemma)

At every `→_sh*`-reachable Σ and every `d ∈ dom(Σ.M)`, `a_emit(Σ, d) = chain_d(f_d^Σ)`.

*Proof.* By B1, `a_emit(Σ, d) = a_emit(π(Σ), d)`; evaluate EmitAddress at `π(Σ)`. If the homed-set is empty — `J_d^Σ = -1` — the first-emission branch returns `d.0.s_L.1 = chain_d(0) = chain_d(f_d^Σ)`. Otherwise `J_d^Σ ≥ 0` and the subsequent-emission branch returns `inc(ℓ_prev, 0)` at the homed-set's T1-maximum `ℓ_prev = chain_d(J_d^Σ)`, giving `inc(chain_d(J_d^Σ), 0) = chain_d(f_d^Σ)`. Both branches name the frontier slot. ∎

*Frontier-landing corollary.* Every `K.λ_sh` deposit, raw or `Emit_K`-mediated, lands at the frontier of its home's chain and advances that frontier by one.

---

## Corollary — RangeSterilization (LEMMA, lemma)

Let Σ be `→_sh*`-reachable, `d ∈ dom(Σ.M)`, `(b, F', G') ∈ L_R^Σ` with `G' = {(g, ℓ)}` (single-span to-endset, the only form a Binary-registered R-tuple carries by P6), and `B = {j ≥ 0 : chain_d(j) ∈ coverage(G')}` the chain indices its to-span covers. Since `coverage(G') = {t : g ≤ t < g ⊕ ℓ}` is a single T1-interval and the chain ascends strictly, B is *consecutive*.

**(i) Covered frontier deposits are born nullified, whatever type they carry.**  
Let `Σ →_sh* Σ'` and let a `K.λ_sh`-step homed at `d` fire at Σ' with `f_d^{Σ'} ∈ B`, reaching post-state Θ: by frontier-landing its deposit `(F, G, K)` lands at `a = chain_d(f_d^{Σ'}) ∈ coverage(G')`. The retraction tuple persists into `L_R^Θ` (L12 via B2), so `a ∈ nullified(Θ)` and `(a, F, G) ∉ A_K^Θ`. The exclusion is type-blind and permanent: R6c bars the deposit from the active subset at every `→_sh*`-successor Θ' of Θ (B3).

**(ii) Exhaustion only by sacrifice.**  
The frontier `f_d` moves only at emissions homed at `d`, advancing by exactly one per step (frontier-landing). No emission homed at `d` can route around the block. Hence, from any Σ' as in (i) with `f_d^{Σ'} ∈ B` and B finite, induction on `k = max B − f_d^{Σ'} + 1` gives: exactly `k` further emissions homed at `d` — each born nullified by (i), each advancing the frontier by one — bring the frontier to `max B + 1`. When B is an up-set there is no exit once the frontier enters B: from entry onward every emission homed at `d` is born nullified — `d`'s link chain is sterilized outright.

**(iii) Resumption is conditional.**  
Suppose B is finite and exhausted: `f_d^{Σ'} = max B + 1` at some `→_sh*`-successor Σ' of Σ. The next emission homed at `d` lands at `chain_d(f_d^{Σ'}) ∉ coverage(G')`, so this tuple no longer blocks it; landing active, however, requires the full weakest precondition at Σ': C3's existential must fail over the whole of `L_R^{Σ'}` — no other retraction tuple's to-coverage may contain `chain_d(f_d^{Σ'})` — and C2 must hold.
