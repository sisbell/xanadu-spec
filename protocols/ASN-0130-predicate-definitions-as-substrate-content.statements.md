> **ASN-0130 · Predicate Definitions as Substrate Content** — condensed claim statements  
> [← Full note](ASN-0130-predicate-definitions-as-substrate-content.md) · [↑ Protocols index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0130 Claim Statements

*Source: ASN-0130-predicate-definitions-as-substrate-content.md (revised unknown) — Extracted: 2026-06-14*

## PR-ENC — EncodingDiscipline (DEF, function)

An *encoding* is an injective map from *signed syntactic terms* — a pair of an ordered, sorted parameter context `Γ_D = ⟨x₁ : C₁, …, x_k : C_k⟩` (each `Cᵢ ∈ Codom`; `k = 0` for closed terms) and a body in ASN-0129's grammar extended with *applied definitional references* (PR-SIG), and no view component (PR-VIEW) — to finite content-value sequences, with a decidable parse and **prefix-freeness**: no parse-valid sequence is a proper prefix of another; equivalently, the parse is *self-delimiting*, determining its own extent from its start. The domain is *syntax only*: grammatical well-formedness, no typing requirement.

A *definition artifact* is a contiguous run `A_def = {shift(a, k) : 0 ≤ k < n}` (`n ≥ 1`) of content addresses holding a parse-valid encoding, identified by its start `a`.

The discipline reserves a countable supply of *expansion names* `ν₁, ν₂, …`, which no recorded parameter name and no body binder may inhabit.

---

## PR-ENC-uniq — EncodingUniqueness (LEMMA, lemma)

At most one parse-valid run starts at any address: were `{shift(a, k) : 0 ≤ k < n}` and `{shift(a, k) : 0 ≤ k < n'}` both parse-valid with `n < n'`, the shorter's value sequence would be a proper prefix of the longer's — both parse-valid encodings — contradicting prefix-freeness.

---

## PR-DISC — RegistrationDiscipline (DEF, predicate)

Call a derivation *registration-disciplined* iff:
- every `L_pdef`-growing step along it is the deposit branch of a `register_pred` call (PR0), and
- every `L_pd_stable`-growing step is the deposit branch of a `certify_pd_stable` call (PR5a).

On such a derivation every active `pdef` tuple is the trace of a validated registration, and every active `pd_stable` tuple the trace of a validated certification.

---

## PR-SIG — SignatureFunction (DEF, function)

`sig(a) = (Γ_D, C_D)`: the parse layer's recorded context, paired with the result sort the WT + WT-ref pass derives for `a`'s body *at `a`'s first registration*.

`sig` is defined exactly on the *ever-registered* addresses, by induction on first-registration order — well-founded, registration events being totally ordered along the derivation (PR2). At `a`'s first registration, every referenced address `r` carries an active `pdef` tuple at the pre-state, so `r`'s first registration is strictly earlier (PR2(a)) and `sig(r)` is already defined by the induction.

Once defined, `sig(a)` never changes. It re-derives identically at every later deposit event for `a`: the body is content-fixed (S0), each consulted `sig(r)` was fixed at its own earlier first registration, and the pass is deterministic.

---

## WT-ref — WellTypingReference (RULE, lemma)

For a context `Γ`, an address `r` with `sig(r)` *defined* and `sig(r) = (⟨x₁ : C₁, …, x_k : C_k⟩, C_r)`, and `Γ ⊢ eᵢ : Cᵢ` for each `1 ≤ i ≤ k`:

`Γ ⊢ r(e₁, …, e_k) : C_r`

Definedness of `sig(r)` is the rule's domain condition — a reference to a never-registered address has no typing judgment, not a false one.

---

## PR0 — DefinitionRegistration (CONTRACT, method)

The operation surface exposes `register_pred(d, A_def)`: *validate, then emit through the `pdef`-class emit it wraps*, returning the classifier tuple's address.

**Validation** runs in full on every call at the call state Σ, writing `a := min(A_def)` (T1) and `n := |A_def|`:

- **(0)** `A_def ≠ ∅`, `|A_def| < ∞`
- **(i)** `A_def ⊆ dom(Σ.C)` and `A_def = {shift(a, k) : 0 ≤ k < n}`
- **(ii)** The self-delimiting parse from `a` succeeds and consumes precisely the presented extent
- **(iii)** `Γ_D ⊢ body : C_D` under WT + WT-ref; each referent `r` is ever-registered: `(∃ (b, F, G) ∈ L_pdef^Σ :: r ∈ addrs(F))`
- **(iv)** Every definitional reference names an actively-registered definition: for each referenced address `r`, some `(b, F, G) ∈ A_pdef^Σ` with `r ∈ addrs(F)`

On success: `Emit_pdef(Σ, d, {a}, A_def)` — depositing `(enc({a}), enc(A_def), pdef)` at frontier address `a_emit(Σ, d)` on a miss.

**Uniqueness.** At most one active `pdef` tuple per I0 class at every state reached. All validated tuples at one start are I0-equal (PR-ENC-uniq) — *at most one active registration per definition address*.

**Postcondition:**
`POST-ref ≡ (∃ (b, F', G') ∈ A_pdef^{Σ'} :: addrs(F') = {a})`

**Weakest precondition (general, on registration-disciplined derivations):**

`wp(register_pred(d, A_def), POST-ref) ≡ (∃ (b, F', G') ∈ A_pdef^Σ :: addrs(F') = {a}) ∨ (VALID(Σ, A_def) ∧ d ∈ dom(Σ.M) ∧ C3(Σ, d))`

with `VALID` = conditions (0)–(iv).

**Weakest precondition (on additionally surface-disciplined derivations, DR eliminating C3):**

`wp(register_pred(d, A_def), POST-ref) ≡ (∃ (b, F', G') ∈ A_pdef^Σ :: addrs(F') = {a}) ∨ (VALID(Σ, A_def) ∧ d ∈ dom(Σ.M))`

---

## PR1 — ValidationPermanence (INV, predicate)

At any state Σ reached by a registration-disciplined derivation (PR-DISC), if `(b, F, G) ∈ L_pdef^Σ`, then:

- The run `addrs(G)`, with start `addrs(F) = {a}`, passed PR0's validation at its deposit's pre-state: parse-valid (ii), well-typed (iii), every reference registered (iv).
- The run holds the same values at Σ and at every `→_sh*`-successor.

**Permanence by conjunct:**

- **(ii) and (iii)** are content/signature-intrinsic: parse-validity (ii) reads only the immutable run; well-typedness (iii) reads the run plus each consulted `sig(r)`, fixed at `r`'s first registration (PR-SIG). Permanent without re-validation.
- **(iv)** is a deposit-time reference-endorsement: permanent as a fact about the deposit's pre-state, not as a standing fact at Σ. A later `Nullify_Binary` on a referent's `pdef` tuple de-registers it without affecting (ii) or (iii).

---

## PR2 — AcyclicReference (LEMMA, lemma)

Under the registration discipline (PR-DISC), deposits into the `pdef` class are `→_sh` steps, totally ordered along any derivation. For an ever-registered definition D, write `e₁(D)` for its *earliest* deposit event.

**(a)** *Every deposit event sees each referent registered strictly earlier.*
A deposit is the miss branch of a `register_pred` whose validation (iv) had just passed at the pre-state: each referenced address `r` carries an active tuple there, which entered `L_pdef` at some strictly earlier deposit event for the definition at `r`. Hence `e₁(r) < e₁(D)`.

**(b)** *Self-reference fails at every deposit event.*
Deposits occur only on a dedup miss, and all validated tuples at one start are I0-equal (PR-ENC-uniq), so at a deposit event for D every existing tuple denoting D's start is inactive — and the depositing tuple does not yet exist during its own validation. Condition (iv) therefore has no witness for a reference to D's own start.

**Consequence:** The reference relation on ever-registered definitions embeds in the strict order `e₁(r) < e₁(D)` — irreflexive, acyclic, a DAG with no cycle check ever run.

**Consequence (termination):** Definitional expansion terminates: expansion descends reference edges with `e₁` strictly decreasing among finitely many deposit events, each term finite; the expanded result is a pure PL term.

---

## PR3 — EvaluationByReference (DEF, function)

Three layers, each pinned.

**Resolution** — address to signed term: read content values from `a` along its origin chain — successive addresses `shift(a, k)`, the chain's `inc(·, 0)` siblings — feeding the self-delimiting parse, which determines the run's extent from content alone and yields `(Γ_D, body)`. Resolution consults no slice and no tuple.

**Expansion** — `expand(a)`: in `body`, replace each applied reference by the referent's expansion with the expanded arguments substituted for its parameters. References are processed bottom-up, siblings left to right. At a node `r(·)` with expanded arguments `E₁, …, E_k`:

1. Take `expand(r)` — the recursion is well-founded because descent strictly decreases first-registration rank (PR2).
2. Rename `expand(r)`'s parameters *and* every binding site in it to expansion names from PR-ENC's reserved supply: the least-indexed names occurring nowhere in the term under construction, the `Eⱼ`, or `expand(r)` itself — assigned in fixed order (parameters first in signature order, then binding sites depth-first, left to right).
3. Substitute `Eⱼ` simultaneously for the renamed `j`-th parameter.

`expand` is a *function* of immutable content: two evaluators expanding the same address at any two states obtain the same concrete term.

**Evaluation** — `evaluate(a, args, view, Σ)`:

*Precondition*: `a` is *ever-registered* at Σ: `∃ (b, F, G) ∈ L_pdef^Σ` with `addrs(F) = {a}`.

`args` is a `Γ_D`-*environment*: one value of sort `Cᵢ` per parameter `xᵢ`, per `sig(a) = (Γ_D, C_D)`.

`evaluate(a, args, view, Σ)` = the ASN-0129 denotation of `expand(a)` at `(args, view, Σ)`.

Active registration is *not* required of `a` or any referent: the precondition is ever-registration.

---

## PR3a — ExpansionWellTyping (LEMMA, lemma)

On registration-disciplined derivations (PR-DISC): for every ever-registered `a` with `sig(a) = (Γ_D, C_D)`, `expand(a)` is a pure PL term with `Γ_D ⊢ expand(a) : C_D`.

**Supporting lemma WT-α (renaming).** If `Γ ⊢ u : C` and `ρ` renames variables sort-preservingly and injectively — acting on `dom(Γ)` and on `u`'s binding sites, its image names pairwise distinct and occurring nowhere in `u` — then `ρΓ ⊢ ρu : C`.

**Supporting lemma WT-W (weakening).** If `Γ ⊢ u : C`, `y ∉ dom(Γ)`, and `y` occurs nowhere in `u` as a binder, then `Γ, y : C′ ⊢ u : C` for any sort `C′`. Iterated, WT-W adjoins any finite set of variables each satisfying both provisos.

**Proof structure (substitution induction):** For a reference node `r(e₁, …, e_k)` typed at context `Γ` by WT-ref with `sig(r) = (⟨x₁:C₁,…,x_k:C_k⟩, C_r)`, expanded arguments `E₁,…,E_k`, and expansion names `y₁,…,y_k` (fresh for `dom(Γ)` and all `Eⱼ`):

1. Structural induction gives `Γ ⊢ Eᵢ : Cᵢ` for each `i`.
2. Rank induction gives `expand(r) ∈ PL` with `⟨x₁:C₁,…,x_k:C_k⟩ ⊢ expand(r) : C_r`. WT-α yields `⟨y₁:C₁,…,y_k:C_k⟩ ⊢ u : C_r` where `u` is the renamed term.
3. Iterated WT-W adjoins `dom(Γ)`: `Γ, y₁:C₁, …, y_k:C_k ⊢ u : C_r`.
4. Sequential substitution by PC2 (`k` steps, last parameter first), lifting `Γ ⊢ Eⱼ : Cⱼ` at each step: `Γ ⊢ u[y₁ ↦ E₁, …, y_k ↦ E_k] : C_r`.

The replacement node carries sort `C_r`, the host derivation re-types unchanged above it, and the induction closes with `Γ_D ⊢ expand(a) : C_D`. Membership in PL holds by the same induction: no reference node survives, so `expand(a)` is pure.

---

## PR-VIEW — ViewTransparency (DEF, predicate)

Call a term *view-independent* iff it contains no view-parameterized constituent and no collection-valued behavior atom on UV's rewrite list (`succs`, `sources_to`, `chain`, `stale`): a syntactic condition, decided by the same finite scan that decides well-typing.

A view-independent term's denotation is invariant in the view argument.

References are *view-transparent*: a referent contributes spelling, never scope — the artifact fixes the term, the reader fixes which state the term's parameterized reads see.

---

## PR4 — VersioningBySupersession (DEF, function)

To update a predicate:

1. Register the successor via `register_pred` (PR0).
2. Emit `supersedes` (the shipped S2 class, ASN-0128) from the old definition's address to the new.

`tip(a)` resolves the current version of the lineage rooted at `a`. Competing successors make a branch and `tip` returns ⊥.

---

## PR5 — DynamicsCertification (DEF, predicate)

**ST⁺**: per-instantiation ⊤-stability established by PD0's rules under the *Parameters* reading fixed below. ST⁺ is a *sound superset* of PD0's literal closed-term **ST**: every literal-ST closed term is ST⁺; the two coincide exactly at `k = 0`.

**Three load-bearing qualifications:**

- *Purity*: the certified object is `expand(a)` (PR3, well-typed by PR3a) — the pure term, not the artifact's reference-bearing spelling.
- *View*: the surface certifies only *view-independent* expansions (PR-VIEW's syntactic class). Their denotation is invariant in the view argument; no view index on the certificate.
- *Parameters*: ST⁺'s parameter reading: the checker runs PD0's rules with each parameter treated as a bound constant of its declared sort. The certificate asserts that *every* `Γ_D`-instantiation of `expand(a)` is ⊤-stable: once true at a reachable Σ for given `args`, true at every `→_sh*`-successor for the same `args`.

**Aggregate extension**: the aggregate rule's threshold position is extended from "ℕ literal" to *an ℕ literal or an environment-bound parameter*. Soundness: a bound parameter's value is the same `args` on both sides of every transition, so count-threshold persistence carries over unconditionally.

**Universal lint** (one-quantifier PL term, view `active`):

`(∀ t ∈ M_pdef :: is_pd_stable(t))`

where `M_pdef` at view `active` = `⋃ addrs(F)` over `A_pdef^Σ` (D1) — under the registration discipline (PR-DISC), exactly the registered definition addresses — and `is_pd_stable(t)` iff some active certificate's F covers `t`.

---

## PR5a — CertificationSurface (CONTRACT, method)

The operation surface exposes `certify_pd_stable(d, a)`: *validate, then emit through the `pd_stable`-class emit it wraps*, returning the certificate tuple's address.

**Validation** at call state Σ, in stated order:

- **(0)** *Predicate sort*: `sig(a)` is defined with Boolean result sort — `sig(a) = (Γ_D, Bool)`.
- **(i)** *Target status*: `a` is actively registered — some `(b, F, G) ∈ A_pdef^Σ` with `addrs(F) = {a}`.
- **(ii)** *Well-posedness*: `expand(a)` is view-independent (PR-VIEW's syntactic scan — no view-parameterized constituent and no UV-rewritten collection atom).
- **(iii)** *Class membership*: `expand(a) ∈ ST⁺` by PD0's rules under PR5's *Parameters* reading.

On success: `Emit_pd_stable(Σ, d, {a}, ∅)` — depositing `(enc({a}), ∅, pd_stable)` on a miss.

**Postcondition:**
`POST-cert ≡ (∃ (b, F', G') ∈ A_pd_stable^{Σ'} :: addrs(F') = {a})`

**Weakest precondition (on registration-disciplined derivations):**

`wp(certify_pd_stable(d, a), POST-cert) ≡ (∃ (b, F', G') ∈ A_pd_stable^Σ :: addrs(F') = {a}) ∨ (CVALID(Σ, a) ∧ d ∈ dom(Σ.M) ∧ C3(Σ, d))`

with `CVALID` = conjunction (0)–(iii). On additionally surface-disciplined derivations (DR), `C3` vanishes.

**Permanence for the slice**: at any state Σ reached by a registration-disciplined derivation (PR-DISC), if `(b, F, G) ∈ L_pd_stable^Σ` with `addrs(F) = {a}`, then at the deposit's pre-state: `sig(a)` had Boolean result sort (condition (0)), `expand(a)` was view-independent, and `expand(a) ∈ ST⁺` held. These facts are permanent: `expand(a)` is the same concrete term at every state (PR3's determinacy), view-independence and ST⁺ are syntactic properties of that fixed spelling, and the Boolean sort is fixed at `a`'s first registration (PR-SIG).

---

## PS1 — PredicateDefinition (REG, datatype)

`pdef` — Multi, idem=⊤, behaviors=∅.

**Slot convention:**
- `F = enc({a})`: one unit-depth span denoting the definition's address; `addrs(F) = {a}`.
- `G = enc(A_def)`: the run; `addrs(G) = A_def`.

Marks a content run as a validated predicate definition. Multi shape: both facts recoverable by denotation (AD, ASN-0128). Idempotent (idem=⊤): dedup by I0 coverage identity on both slots — same start, same run.

---

## PS2 — StabilityCertificate (REG, datatype)

`pd_stable` — Unary, idem=⊤, behaviors=∅.

**Slot convention:**
- `F = enc({a})`: the certified definition's address; `addrs(F) = {a}`.
- `G = ∅` (Unary).

Asserts **ST⁺** certification (PR5) of the view-independent expansion of the definition at `a`. Emitted only by `certify_pd_stable` (PR5a). All certificates for one definition are I0-equal; identity is by slot-F coverage `subtree(a)` alone.

---

## Definition — SignedTerm

A *signed syntactic term* is a pair `(Γ_D, body)` where:
- `Γ_D = ⟨x₁ : C₁, …, x_k : C_k⟩` is an ordered, sorted parameter context; each `Cᵢ ∈ Codom`; `k = 0` for closed terms.
- `body` is in ASN-0129's grammar extended with applied definitional references (PR-SIG).
- No view component (PR-VIEW).

---

## Definition — Sig

`sig : Address → (ParameterContext × Sort)` (partial)

Domain: the *ever-registered* addresses.

`sig(a) = (Γ_D, C_D)` where:
- `Γ_D` = the parse layer's recorded parameter context from `a`'s encoding.
- `C_D` = the result sort derived by the WT + WT-ref pass on `a`'s body at `a`'s first registration event.

Defined by induction on first-registration order. At `a`'s first registration: for each referenced `r`, PR2(a) gives `e₁(r) < e₁(a)`, so `sig(r)` is already defined. The pass is a terminating syntax-directed walk; `C_D` is uniquely determined. Once defined, `sig(a)` is immutable.

---

## Definition — Expand

`expand : Address → PL_Term` (total on ever-registered addresses)

```
expand(a):
  (Γ_D, body) := resolve(a)                     -- content-read, no slice consulted
  return substitute_references(body)

substitute_references(t):
  for each reference node r(e₁, …, e_k) in t, bottom-up, siblings left-to-right:
    E_i := substitute_references(e_i)  for each i     -- expand arguments first
    u   := expand(r)                                   -- rank-induction, well-founded by PR2
    y₁,…,y_k := fresh expansion names, least-indexed not in:
                 (term under construction) ∪ {E₁,…,E_k} ∪ u
                 assigned: parameters in signature order,
                           then binding sites of u depth-first, left-to-right
    replace r(e₁,…,e_k) with u[x₁↦y₁,…,x_k↦y_k][y₁↦E₁,…,y_k↦E_k]
  return t    -- now a pure PL term (no reference nodes)
```

Result is a function of immutable content: deterministic, same concrete term at every state.

---

## Definition — Evaluate

`evaluate : Address × Environment × View × State → Value` (partial)

**Precondition**: `a` is ever-registered at Σ: `∃ (b, F, G) ∈ L_pdef^Σ` with `addrs(F) = {a}`.

**Parameter**: `args` is a `Γ_D`-environment — one value of sort `Cᵢ` per parameter `xᵢ`, per `sig(a) = (Γ_D, C_D)`.

`evaluate(a, args, view, Σ) = ⟦expand(a)⟧(args, view, Σ)`

where `⟦·⟧` is the ASN-0129 denotation.

Active registration is *not* required. Purity (PC4), termination (PC5), decidability, and the ceiling (PC6) hold verbatim for the expansion, which is a fixed pure PL term before evaluation begins.

---

## Definition — VALID

`VALID(Σ, A_def) : Bool` — the validation predicate for PR0.

Writing `a := min(A_def)`, `n := |A_def|`:

- **(0)** `A_def ≠ ∅ ∧ |A_def| < ∞`
- **(i)** `A_def ⊆ dom(Σ.C) ∧ A_def = {shift(a, k) : 0 ≤ k < n}`
- **(ii)** The self-delimiting parse from `a` succeeds consuming exactly the presented extent
- **(iii)** `Γ_D ⊢ body : C_D` under WT + WT-ref; each referent `r`: `∃ (b, F, G) ∈ L_pdef^Σ :: r ∈ addrs(F)`
- **(iv)** Each referenced `r`: `∃ (b, F, G) ∈ A_pdef^Σ :: r ∈ addrs(F)`

`VALID(Σ, A_def) ⟺ (0) ∧ (i) ∧ (ii) ∧ (iii) ∧ (iv)`

---

## Definition — CVALID

`CVALID(Σ, a) : Bool` — the validation predicate for PR5a.

- **(0)** `sig(a)` is defined ∧ result sort of `sig(a)` is `Bool`
- **(i)** `∃ (b, F, G) ∈ A_pdef^Σ :: addrs(F) = {a}`
- **(ii)** `expand(a)` is view-independent
- **(iii)** `expand(a) ∈ ST⁺`

`CVALID(Σ, a) ⟺ (0) ∧ (i) ∧ (ii) ∧ (iii)`

---

## Definition — STPlus

`ST⁺ ⊆ PL_Term` — the certified stability class.

A pure PL term `t` with free variables among `Γ_D = ⟨x₁:C₁,…,x_k:C_k⟩` is in `ST⁺` iff PD0's rules classify it as ⊤-stable under the *parameter reading*: each parameter `xᵢ` treated as a bound constant of sort `Cᵢ`.

The *parameter reading* extends PD0's aggregate rule: the count threshold may be an ℕ literal *or* an environment-bound ℕ-sorted parameter.

`t ∈ ST⁺` asserts: for every environment `args` binding `Γ_D`, if `⟦t⟧(args, view, Σ) = ⊤` at any reachable Σ, then `⟦t⟧(args, view, Σ') = ⊤` for every `→_sh*`-successor Σ'.

`ST⁺` is a sound superset of PD0's literal closed-term `ST`. At `k = 0`, they coincide.
