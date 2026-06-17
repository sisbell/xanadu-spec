> **ASN-0042 · Tumbler Ownership** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md), [ASN-0040 · Tumbler Baptism](ASN-0040-tumbler-baptism.md)  
> [Condensed statements →](ASN-0042-tumbler-ownership.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0042: Tumbler Ownership

*2026-03-15*

We are looking for what it means to *own* a position in the tumbler hierarchy. The tumbler algebra (ASN-0034) gives us a permanently expanding, totally ordered, hierarchically structured address space. But the algebra is silent on authority — it tells us that addresses exist and how they compare, not who may act upon them. Ownership is the layer of meaning that binds addresses to principals.

The investigation yields a central finding: ownership is not a table the system maintains but a *theorem about addresses*. The two-place ownership predicate `owns(π, a)` — "does this principal own this address?" — reduces to a prefix comparison between `pfx(π)` and `a`, requiring no consultation of mutable system state. The one-place effective-owner function `ω(a)` — "who owns this address?" — must additionally consult the principal registry to select the longest matching prefix; it is state-relativized, not absolute. This has consequences for delegation, for the boundaries of authority, and for the architectural response when a principal encounters content it does not own.

We derive each property from Nelson's design intent, corroborated by Gregory's implementation evidence, and state them at the level of abstraction required of any conforming implementation.


## Ownership as a Structural Predicate

We begin with the most fundamental question: how does the system determine who owns an address?

Nelson gives a striking answer. Ownership is not recorded in a registry external to the address — it is *readable from the address itself*:

> "The basic principle is that of owned numbers. Numbers are owned by individuals or companies, and subnumbers under them are bestowed on other individuals and companies on whatever basis the owners choose." (LM 4/17)

Gregory's implementation confirms this with unusual force. The sole ownership predicate in udanax-green — `isthisusersdocument` — delegates entirely to `tumbleraccounteq`, a function that decides prefix containment of its two tumbler arguments. No table is consulted. No file is opened. No registry is queried. The function receives two tumblers, performs arithmetic on their components, and returns a boolean.

`tumbleraccounteq` realizes the two-place predicate `owns(π, a)` (O1 below) and only that: invoked once against the single account of the connected session, it decides whether one account prefix contains an address. The longest-match selector `ω` (O2) is an abstract obligation a conforming implementation must discharge, not something `tumbleraccounteq` secures. At the account level, where exactly one account is ever in play, ownership *is* the containment comparison.

**pfx(π) (OwnershipPrefix).**

We introduce the principals. Let `Π` denote the set of *principals* — the ownership subjects. Each principal `π ∈ Π` is associated with an *ownership prefix* `pfx(π) ∈ T`, a valid tumbler (satisfying T4) that serves as the root of their namespace.

The mapping `pfx : Π → T` is a primitive of the ownership model, with codomain constrained to valid tumblers — `pfx(π) ∈ T` with `T4(pfx(π))`.

*Formal Contract:*
- *Axiom:* `pfx : Π → T` is a total mapping assigning each principal its ownership prefix.
- *Preconditions:* `π ∈ Π`.
- *Postconditions:* (a) `pfx(π) ∈ T`. (b) `T4(pfx(π))` — the prefix is a valid tumbler satisfying HierarchicalParsing.

**O1b (PrefixInjectivity).** Distinct principals have distinct prefixes in every reachable state — a derived reachable-state invariant: `(A Σ reachable, π₁, π₂ ∈ Π_Σ : pfx(π₁) = pfx(π₂) ⟹ π₁ = π₂)`.

The ownership question "does `π` own `a`?" is answered by examining these two tumblers alone, by prefix containment:

**O1 (PrefixDetermination).** Principal `π` owns address `a` iff `pfx(π)` is a prefix of `a`:

  `owns(π, a)  ≡  pfx(π) ≼ a`

where `p ≼ a` denotes that `p` is a prefix of `a` in the sense of Prefix (PrefixRelation) — the components of `p` match the leading components of `a`. T5 (ContiguousSubtrees) is the structural property of the address space that `≼` partitions the space into contiguous subtrees; the relation itself is supplied by Prefix.

Deciding `owns(π, a)` requires only the tumbler pair `(pfx(π), a)`; this is why `tumbleraccounteq` accepts both tumblers as arguments and not just the address.

O1 is a definition: we define the ownership predicate `owns(π, a)` to be identical with prefix containment `pfx(π) ≼ a`. We verify that the definition is well-formed and that it satisfies the **decidability** postcondition: `owns(π, a)` is decidable from `pfx(π)` and `a` alone, without consulting any mutable system state — no registry, table, or transition history is required.

*Well-formedness.* The prefix relation `≼` is defined by Prefix (PrefixRelation): `p ≼ a ⟺ #a ≥ #p ∧ (A i : 1 ≤ i ≤ #p : pᵢ = aᵢ)`. For `owns(π, a)` to be well-defined, two conditions must hold. First, `pfx(π)` must be a valid tumbler — this holds by the definition of `pfx`, which requires every principal's prefix to satisfy T4 (HierarchicalParsing). Second, the component-wise comparison must be determinate — by T3 (CanonicalRepresentation), each component `pᵢ` and `aᵢ` is a uniquely determined natural number, so equality at each position is decidable.

*Decidability.* The prefix check `pfx(π) ≼ a` requires one length comparison `#a ≥ #pfx(π)` followed by at most `#pfx(π)` component comparisons, each a comparison of natural numbers. The entire computation uses `pfx(π)` and `a` alone, consulting no mutable system state — so `owns(π, a)` is decidable from the prefix and the address without external state, discharging the decidability postcondition.

*Design justification.* Nelson states that "numbers are owned by individuals or companies, and subnumbers under them are bestowed on other individuals and companies" (LM 4/17) — ownership is legible from the address itself. Gregory's `tumbleraccounteq` confirms the decision procedure: it decides prefix containment of the two tumblers. The definition `owns(π, a) ≡ pfx(π) ≼ a` formalizes this structural containment exactly. ∎

*Formal Contract:*
- *Definition:* `owns(π, a) ≡ pfx(π) ≼ a`, where `≼` is the prefix relation defined by Prefix (PrefixRelation).
- *Preconditions:* `π ∈ Π`, `a ∈ T`. (`T4(pfx(π))` holds for every `π ∈ Π` by the `pfx` axiom's postcondition (b), so it is inherited, not assumed.)
- *Postconditions:* `owns(π, a)` is a total, decidable predicate on `Π × T`.


## The Account-Level Boundary

Not every prefix match constitutes an ownership claim. The tumbler hierarchy has four structural levels — node, user, document, element — separated by zero-valued components (T4). The allocation mechanism is uniform across all levels — any address holder can subdivide — but ownership authority is hierarchical, and the hierarchy has a definite floor.

Nelson is explicit on this point: "once assigned a User account, the user will have full control over its subdivision forevermore" (LM 4/29). This is the strongest authority statement in the specification, and it appears only at the account level. At the document level, ownership is defined with specific enumerated rights: "only the owner has a right to withdraw a document or change it" (LM 2/29). At the version level, Nelson is deliberately cautious: "the version, or subdocument number is only an accidental extension of the document number, and strictly implies no specific relationship of derivation" (LM 4/29). The design intent is clear: baptism (allocation) is uniform; authority (ownership) flows from the account. Everyone at every level can fork sub-addresses — that is the mechanism. But what one can *do* with what one has forked depends on one's position in the ownership hierarchy.

We formalize this asymmetry:

**O1a (AccountOwnershipBoundary).** Ownership principals exist only at node level or account level:

  `(A Σ reachable from Σ₀, π ∈ Π_Σ : zeros(pfx(π)) ≤ 1)`

Sub-account allocation — creating documents, versions, elements — does not introduce new ownership principals. It exercises the allocator's rights within an existing principal's domain.

**acct(a) (AccountField).**

We define `acct(a)`, the *account field* of a valid tumbler `a` — the node-and-user portion that fixes ownership, with defining property `zeros(acct(a)) ≤ 1` for every valid tumbler.

Gregory confirms the account-level boundary with unusual force. The prefix-containment check of `tumbleraccounteq` (O1) stops at the second zero: once the zero-counter reaches two — the account/document separator — the function returns true unconditionally, ignoring everything beyond. The implementation has no mechanism for finer-grained discrimination: `isthisusersdocument` (in all three build targets — `be.c`, `socketbe.c`, `xumain.c`) delegates directly to `tumbleraccounteq` with no intervening check. There is no per-document, per-version, or per-element authorization predicate anywhere in the codebase. The BERT system tracks per-document open/close state, but its authorization fallback is `isthisusersdocument` — account-level.

The consequence: sub-account allocation (creating documents, versions, elements) creates addresses within the allocating principal's domain but does not partition that domain into sub-ownerships. A document address `N.0.U.0.D.0.E` and a different document address `N.0.U.0.D'.0.E'` under the same account are owned by the same principal — the one whose prefix matches `N.0.U`. Below the account level, there is only the binary distinction of "mine" versus "not mine."

O1a permits nesting *within* the account level. T4 allows multi-component user fields: `pfx(π₁) = [1, 0, 2]` and `pfx(π₂) = [1, 0, 2, 3]` both satisfy `zeros ≤ 1`, and `pfx(π₁) ≺ pfx(π₂)`. Nelson designed this deliberately: "accounts can spin off accounts" (LM 4/19). The User field is a tree, not a flat namespace — a principal may delegate a sub-account by forking a longer user field within its own prefix. Gregory confirms: `tumbleraccounteq` applied to account `[1, 0, 2, 3]` checks positions 0, 2, and 3, while account `[1, 0, 2]` checks only positions 0 and 2 — the child account is a strict refinement. What O1a prevents is *document-level* or *element-level* principals: no principal has `zeros(pfx(π)) ≥ 2`. The floor of ownership is the account level, but within that floor, the user-field tree can grow arbitrarily deep.

**FieldStructure.** For a valid tumbler `a` satisfying T4 with `zeros(a) = z`, T4b (UniqueParse) decomposes `a` uniquely into `z + 1` fields — node, then (for `z ≥ 1`) user, (for `z ≥ 2`) document, (for `z = 3`) element — each a contiguous segment with a single zero between consecutive segments. By T4a (SyntacticEquivalence) every segment is non-empty (node length `α ≥ 1`, user length `β ≥ 1` when present, and so on), and by T4's positive-component constraint every non-separator component is strictly positive; the separators occupy exactly the zero-valued positions, the first at position `α + 1`, the second (if present) at `α + 1 + β + 1`, and so on. Component-wise access is decidable from T3 (CanonicalRepresentation), so the decomposition is computable from `a` alone. The case distinction `z ∈ {0, 1, 2, 3}` is exhaustive by T4's zero-count clause.

*Formal Contract:*
- *Preconditions:* `a ∈ T` is a valid tumbler satisfying T4 (HierarchicalParsing — positive non-separator components, at most three zeros) and T4a (SyntacticEquivalence — no adjacent zeros, no leading or trailing zero).
- *Definition:* `acct(a) = a` when `zeros(a) = 0`; `acct(a) = N(a) ++ [0] ++ U(a)` when `zeros(a) ≥ 1`, where `N(a)` and `U(a)` are the node and user fields extracted by `fields(a)` (T4b UniqueParse), with component-wise access decidable from T3 (CanonicalRepresentation).
- *Postconditions* (all four follow from FieldStructure): (a) `acct(a)` is a valid tumbler satisfying T4. (b) `zeros(acct(a)) ≤ 1`. (c) When `zeros(a) ≤ 1`: `acct(a) = a`. (d) When `zeros(a) ≥ 2`: `acct(a)` is a proper prefix of `a` with `zeros(acct(a)) = 1`.


## Ownership Domains

Each principal's prefix determines a set of addresses — their *domain*:

**Definition (OwnershipDomain).** For principal `π ∈ Π`, define `odom(π) = {a ∈ T : pfx(π) ≼ a}` (`odom`, distinct from ASN-0034's allocator `dom`).

The prefix relation `≼` satisfies a comparability property:

**Covering-chain lemma (PrefixesOfCommonAddressAreComparable).** Any two tumbler prefixes of a common address are `≼`-comparable:

  `(A x, p, q ∈ T : p ≼ x ∧ q ≼ x ⟹ p ≼ q ∨ q ≼ p)`

*Proof.* By Prefix (PrefixRelation), `p ≼ x` expands to `#x ≥ #p ∧ (A i : 1 ≤ i ≤ #p : pᵢ = xᵢ)`, and `q ≼ x` expands to `#x ≥ #q ∧ (A i : 1 ≤ i ≤ #q : qᵢ = xᵢ)`. Both `p` and `q` agree with `x` on their respective leading components. Without loss of generality let `#p ≤ #q`. For each `i` with `1 ≤ i ≤ #p`, both equalities apply: `pᵢ = xᵢ = qᵢ`. Hence `pᵢ = qᵢ` for `1 ≤ i ≤ #p`, and `#p ≤ #q`, so `p ≼ q` by the Prefix definition. ∎

By T5 (ContiguousSubtrees), every ownership domain is a contiguous interval under the lexicographic order T1. This is a mathematical consequence of prefix containment and the tree-to-line mapping, not a policy choice. If `a, c ∈ odom(π)` and `a ≤ b ≤ c`, then `b ∈ odom(π)`. No address can escape from the interior of someone's domain.

Domains nest whenever prefixes nest:

  `pfx(π₁) ≼ pfx(π₂)  ⟹  odom(π₂) ⊆ odom(π₁)`

The proof unfolds the prefix relation componentwise. Suppose `a ∈ odom(π₂)`, so `pfx(π₂) ≼ a`: by Prefix (PrefixRelation) of ASN-0034, this expands to `#a ≥ #pfx(π₂)` and `pfx(π₂)ⱼ = aⱼ` for `1 ≤ j ≤ #pfx(π₂)`. The hypothesis `pfx(π₁) ≼ pfx(π₂)` likewise expands to `#pfx(π₁) ≤ #pfx(π₂)` and `pfx(π₁)ᵢ = pfx(π₂)ᵢ` for `1 ≤ i ≤ #pfx(π₁)`. Composing the two component equalities: for each `i` with `1 ≤ i ≤ #pfx(π₁)`, we have `pfx(π₁)ᵢ = pfx(π₂)ᵢ = aᵢ`. The length chain `#pfx(π₁) ≤ #pfx(π₂) ≤ #a` gives `#a ≥ #pfx(π₁)`. Both clauses of the Prefix relation are satisfied, so `pfx(π₁) ≼ a`, hence `a ∈ odom(π₁)`. This is the prefix relation's transitivity, derived directly from the Prefix (PrefixRelation) definition. This covers all nesting cases — both cross-level (a node operator's domain containing an account domain) and same-level (an account holder's domain containing a sub-account domain, as when `pfx(π₁) = [1, 0, 2]` and `pfx(π₂) = [1, 0, 2, 3]` both satisfy O1a with `zeros = 1`).

As a corollary, when the nesting is cross-level — `zeros(pfx(π₁)) < zeros(pfx(π₂))` — the containing principal operates at a strictly higher level of the field hierarchy (node containing account, for instance). But the defining condition is prefix containment alone, not the zero count.


## State Axioms

*Notation.* Throughout this ASN, `Σ.B` denotes ASN-0040's baptismal registry `s.B` (`Σ.B ⊆ T`), carried as a component of the richer ownership state `Σ`. It is the set of tumblers brought into existence by the baptism procedure. We say "allocated address" and "address in `Σ.B`" interchangeably. The registry grows monotonically: `Σ.B ⊆ Σ'.B` whenever `Σ → Σ'` (B0 of ASN-0040).

*Reachability convention.* All states `Σ` discussed in this ASN are assumed to be *reachable from the bootstrap state* `Σ₀` — that is, there exists a finite sequence `Σ₀ → Σ_1 → ... → Σ` of state transitions producing `Σ`.

**BootstrapContainment (derived).** `Σ` reachable from `Σ₀` ⟹ `Π₀ ⊆ Π_Σ`. Proof: iterate O12 along the witnessing sequence `Σ₀ → ... → Σ`.

**O12 (PrincipalPersistence).** Once a principal joins Π, no operation removes it:

  `(A Σ, Σ' : Σ → Σ' ⟹ Π_Σ ⊆ Π_{Σ'})`

Nelson's architecture contains no concept of account revocation, and Gregory's codebase contains no deletion path for account entries.

**O13 (PrefixImmutability).** Once established, a principal's ownership prefix cannot be altered:

  `(A π ∈ Π_Σ, Σ, Σ' : Σ → Σ' ∧ π ∈ Π_{Σ'} ⟹ pfx_{Σ'}(π) = pfx_Σ(π))`

The prefix is a tumbler, and addresses are permanent (T8): no operation of the tumbler algebra mutates an existing tumbler in place.

**O14 (BootstrapPrincipal).** The initial principal set satisfies the following labeled conjuncts:

  **O14.1 (Nonempty):** `Π₀ ≠ ∅`

  **O14.2 (Coverage):** `(A a ∈ Σ₀.B : (E π ∈ Π₀ : pfx(π) ≼ a))`

  **O14.3 (Finite):** `|Π₀| < ∞`

  **O14.4 (AccountTier):** `(A π ∈ Π₀ : zeros(pfx(π)) ≤ 1)`

  **O14.5 (Injective):** `(A π₁, π₂ ∈ Π₀ : pfx(π₁) = pfx(π₂) ⟹ π₁ = π₂)`

  **O14.6 (Valid):** `(A π ∈ Π₀ : T4(pfx(π)))`

  **O14.7 (NonNesting):** `(A π₁, π₂ ∈ Π₀ : π₁ ≠ π₂ ⟹ pfx(π₁) ⋠ pfx(π₂) ∧ pfx(π₂) ⋠ pfx(π₁))`

  **O14.8 (Baptized):** `(A π ∈ Π₀ : pfx(π) ∈ Σ₀.B)`

  **O14.9 (Registry):** `Σ₀.B is an ASN-0040-reachable registry conforming to B₀ conf.`

In a single-node system, `Π₀ = {π_N}` where `π_N` is the node operator with a node-level prefix (`zeros = 0 ≤ 1`); non-nesting holds vacuously (a singleton set has no distinct pairs), and all other base-case clauses hold trivially — a single-component positive tumbler like `[1]` satisfies T4 (no zeros, no adjacency or boundary violations). This single-node case witnesses satisfiability of O14; the multi-node instance is verified clause-by-clause in the Worked Example below.

**O15 (PrincipalClosure).** Principals enter Π exclusively through bootstrap (in Π₀) or through a delegation act of an existing principal subject to five structural conditions, named the *delegation predicate* `delegated_Σ(π, π')`; no other mechanism introduces principals. Each state transition introduces at most one new principal, and any newly introduced principal `π'` traces back to a delegating predecessor `π` whose prefix is a strict ancestor of `pfx(π')`:

  `(A Σ, Σ' : Σ → Σ' ⟹ |Π_{Σ'} ∖ Π_Σ| ≤ 1)`

  `(A π' ∈ Π_{Σ'} ∖ Π_Σ : (E π ∈ Π_Σ :`
  `      (i)    [ancestry]        pfx(π) ≺ pfx(π')`
  `      (ii)   [authorization]   (A π'' ∈ Π_Σ : pfx(π'') ≼ pfx(π') ⟹ #pfx(π'') ≤ #pfx(π))`
  `      (iii)  [structural-tier] zeros(pfx(π')) ≤ 1`
  `      (iv)   [top-down-order]  ¬(E π'' ∈ Π_Σ : pfx(π') ≺ pfx(π''))`
  `      (v)    [fresh-valid]     T4(pfx(π')) ∧ pfx(π') ∉ Σ.B ))`

Nelson's design contains no concept of principals appearing outside the delegation hierarchy, and Gregory's codebase provides no mechanism for it.

**Definition (delegated).** We name the conjunction of conditions (i)–(v) above the *delegation predicate*, with a four-place signature: `delegated(Σ, Σ', π, π')` holds iff `Σ → Σ'`, `π ∈ Π_Σ`, `π' ∈ Π_{Σ'} ∖ Π_Σ`, and conditions (i)–(v) hold for `(π, π')` at `Σ` (the subscript form `delegated_Σ(π, π')` abbreviates `delegated(Σ, Σ', π, π')` for the `Σ'` bound by the surrounding formula). Evaluation-state convention: the delegator prefix `pfx(π)` is read at `Σ`, the delegate prefix `pfx(π')` at `Σ'`.

We name a *parent relation* `R_Σ` on `Π_Σ` and write `covers_Σ*` for its reflexive-transitive closure. For a non-bootstrap principal `π' ∈ Π_Σ`, `R_Σ(π, π')` holds iff `π` is the most-specific covering principal of `pfx(π')` in `Π_Σ` — the unique `π ∈ Π_Σ` with `pfx(π) ≺ pfx(π')` of maximal prefix length. This `π` is unique: the covering principals of the common tumbler `pfx(π')` are `≼`-comparable (covering-chain lemma, Ownership Domains section) and have pairwise distinct prefixes (O1b), so their prefix lengths are distinct and a single maximal-length one exists. Then `covers_Σ* = ∪_{m ≥ 0} R_Σ^m`, where `R_Σ^0` is the identity relation on `Π_Σ` and `R_Σ^{m+1} = R_Σ^m ∘ R_Σ`. Equivalently, `covers_Σ*(π, π')` iff `π = π'` or there is a finite chain `π = π^{(0)}, π^{(1)}, ..., π^{(m)} = π'` (`m ≥ 1`) of principals in `Π_Σ` with each consecutive pair `(π^{(j)}, π^{(j+1)})` related by `R_Σ`.

**Delegation edges are cover edges (bridge).** A real delegation event and a structural cover step coincide on the edge they introduce: whenever `delegated_Σ(π_d, π')` holds along `Σ → Σ'`, condition (ii) makes `π_d` the most-specific principal of `Π_Σ` covering `pfx(π')`. Evaluating `R_{Σ'}` over `Π_{Σ'} = Π_Σ ∪ {π'}`, the only candidate not already in `Π_Σ` is `π'` itself, and `π'` cannot be a strict cover of its own prefix (`pfx(π') ⊀ pfx(π')`); so the most-specific strict cover of `pfx(π')` in `Π_{Σ'}` coincides with that in `Π_Σ`, namely `π_d`, and `R_{Σ'}(π_d, π')` holds. This edge persists into every later state by O13 (PrefixImmutability), which fixes `pfx(π')` against subsequent transitions. Hence every chain of delegation events is a `covers`-chain.

**StrictLongestCover (lemma).** *General form:* let `χ ∈ Π_{Σ'}` cover an address `a` (`pfx(χ) ≼ a`), and suppose no principal of `Π_{Σ'}` covers `a` with a prefix strictly extending `pfx(χ)` (the *no-strict-extension* hypothesis). Then every covering principal `π'' ∈ Π_{Σ'}` with `π'' ≠ χ` satisfies `#pfx(π'') < #pfx(χ)`; hence `χ` is the unique coverer of `a` of maximal prefix length in `Π_{Σ'}`.

*Proof.* The prefixes `pfx(π'')` and `pfx(χ)` are both prefixes of the common address `a`, so by the covering-chain lemma they are `≼`-comparable. Three cases exhaust the comparison. If `pfx(π'') = pfx(χ)`, then O1b (PrefixInjectivity) forces `π'' = χ`, excluded since `π'' ≠ χ`. If `pfx(χ) ≺ pfx(π'')`, then `π''` covers `a` with a prefix strictly extending `pfx(χ)`, excluded by the no-strict-extension hypothesis. The only consistent case is `pfx(π'') ≺ pfx(χ)`, which gives `#pfx(π'') < #pfx(χ)`. Since `χ` itself covers `a`, it is the unique coverer of maximal prefix length. ∎

*Newly-delegated corollary.* Suppose `delegated_Σ(π, π')` holds along `Σ → Σ'`, so by O15 `π'` is the sole principal in `Π_{Σ'} ∖ Π_Σ` and conditions (i),(ii),(iv) hold. Then (Part 1) no `π'' ∈ Π_Σ` satisfies `pfx(π') ≼ pfx(π'')`: equality would force `π'' = π'` by O1b, impossible since `π' ∉ Π_Σ`, and `pfx(π') ≺ pfx(π'')` contradicts condition (iv). Consequently (Part 2), for every `a` with `pfx(π') ≼ a` the no-strict-extension hypothesis holds for `χ = π'` — no `π'' ∈ Π_Σ` strictly extends `pfx(π')`, and `π'` does not strictly extend itself — so the general form applies: `π'` is the unique coverer of `a` of maximal prefix length in `Π_{Σ'}`, every pre-existing coverer `π'' ∈ Π_Σ` having `#pfx(π'') < #pfx(π')`.

**NestingByDelegation (derived).** In every reachable state `Σ`, any two distinct principals are either non-nesting in their prefixes, or one strictly extends the other and the extending principal was introduced into `Π` via a chain of delegations originating at the shorter-prefix principal:

  `(A Σ : Σ reachable from Σ₀ : (A π₁, π₂ ∈ Π_Σ : π₁ ≠ π₂ ⟹`
  `      (pfx(π₁) and pfx(π₂) are non-nesting) ∨`
  `      (pfx(π₁) ≺ pfx(π₂) ∧ covers_Σ*(π₁, π₂)) ∨`
  `      (pfx(π₂) ≺ pfx(π₁) ∧ covers_Σ*(π₂, π₁)) ))`

where `covers_Σ*` is the reflexive-transitive closure of the parent relation `R_Σ` defined above. Equality `pfx(π₁) = pfx(π₂)` is excluded by O1b.

We derive this by induction on the transition sequence `Σ₀ → Σ_1 → ... → Σ`.

*Base case:* By O14.7 (NonNesting), all initial principals in `Π_{Σ_0}` have pairwise non-nesting prefixes. So the first disjunct holds directly for every pair `π₁, π₂ ∈ Π_{Σ_0}` with `π₁ ≠ π₂`.

*Inductive step:* Suppose the invariant holds at `Σ_n` and `Σ_n → Σ_{n+1}` via some delegation `delegated_{Σ_n}(π_d, π')` introducing `π'` (by O15, at most one new principal per step; if none is introduced, the invariant is preserved trivially — every disjunct at `Σ_n` lifts to `Σ_{n+1}` by the witness-preservation argument given immediately below). Consider any pair `π₁, π₂ ∈ Π_{Σ_{n+1}}` with `π₁ ≠ π₂`. If both lie in `Π_{Σ_n}`, the IH applies at `Σ_n` and each disjunct lifts to `Σ_{n+1}`. *Witness preservation:* the non-nesting disjunct depends only on the two prefixes, which are preserved across `Σ_n → Σ_{n+1}` by O13 (PrefixImmutability) — `pfx_{Σ_{n+1}}(π_j) = pfx_{Σ_n}(π_j)` for `j ∈ {1, 2}`, since both lie in `Π_{Σ_n}` — so non-nesting at `Σ_n` carries over to `Σ_{n+1}`. The strict-extension disjuncts have the form `covers_{Σ_n}*(π_a, π_b)`, a chain of `R_{Σ_n}`-steps; since `R` is monotone — `R_{Σ_n} ⊆ R_{Σ_{n+1}}`, because prefixes are immutable (O13) and the most-specific covering principal of any prefix is preserved as `Π` grows — the same chain witnesses `covers_{Σ_{n+1}}*(π_a, π_b)`. The preservation is by condition (iv): if `π'` (the one newcomer admitted by `Σ_n → Σ_{n+1}`) had a prefix strictly between `pfx(π)` and some `pfx(π_b)` already covered by `π` in `Π_{Σ_n}`, then `pfx(π') ≺ pfx(π_b)` with `π_b ∈ Π_{Σ_n} ⊆ Π_{Σ_{n+1}}`, violating condition (iv) — `¬(E π'' ∈ Π_{Σ_n} : pfx(π') ≺ pfx(π''))` — at `π'`'s own introducing step. So no newcomer can interpose itself as a more-specific cover, and `R_{Σ_n}`-edges persist into `Σ_{n+1}`. Otherwise one of `π₁, π₂` is `π'`; without loss of generality let `π₂ = π'` and `π₁ ∈ Π_{Σ_n}` (`π₁ = π'` would force `π₁ = π₂`). Compare `pfx(π₁)` and `pfx(π')`:

- *Non-nesting:* The first disjunct holds. ✓
- *`pfx(π') ≼ pfx(π₁)` (i.e. `pfx(π') ≺ pfx(π₁)` or `pfx(π') = pfx(π₁)`):* Impossible by the StrictLongestCover newly-delegated corollary (Part 1): since `π₁ ∈ Π_{Σ_n}`, it cannot satisfy `pfx(π') ≼ pfx(π₁)`. Both the strict-extension and equality cases are thereby excluded.
- *`pfx(π₁) ≺ pfx(π')`:* We must establish `covers_{Σ_{n+1}}*(π₁, π')`. By condition (ii), the delegator `π_d` is the most-specific principal in `Π_{Σ_n}` covering `pfx(π')`, so `pfx(π_d) ≼ pfx(π')`. From the hypothesis `pfx(π₁) ≺ pfx(π')`, `π₁` also covers `pfx(π')`. Both `pfx(π_d)` and `pfx(π₁)` are prefixes of the common tumbler `pfx(π')`, so by the covering-chain lemma (Ownership Domains section) they are `≼`-comparable. Three sub-cases exhaust the comparison:
   * *`pfx(π_d) ≺ pfx(π₁)` (strict).* This case is impossible. The strict extension gives `#pfx(π₁) > #pfx(π_d)`, and `π₁ ∈ Π_{Σ_n}` covers `pfx(π')`, contradicting condition (ii) — which requires `#pfx(π'') ≤ #pfx(π_d)` for every `π'' ∈ Π_{Σ_n}` covering `pfx(π')`. Eliminated by the most-specific clause.
   * *`pfx(π_d) = pfx(π₁)`.* By O1b at `Σ_n`, equal prefixes force `π_d = π₁`. The delegation step `delegated_{Σ_n}(π_d, π')` induces the cover edge `R_{Σ_{n+1}}(π₁, π')` (by the bridge above), which is itself the chain `covers_{Σ_{n+1}}*(π₁, π')`. ✓
   * *`pfx(π₁) ≺ pfx(π_d)` (strict).* Apply the IH to the pair `(π₁, π_d) ∈ Π_{Σ_n} × Π_{Σ_n}` (distinct by the strict prefix relation, so the IH's hypothesis `π₁ ≠ π_d` is met). Since `pfx(π₁) ≺ pfx(π_d)`, the IH's first disjunct (non-nesting) is excluded; its third disjunct (`pfx(π_d) ≺ pfx(π₁)`) contradicts our strict ordering; so the second disjunct applies, yielding `covers_{Σ_n}*(π₁, π_d)`. Concatenating with the cover edge `R_{Σ_{n+1}}(π_d, π')` induced by the current delegation step `delegated_{Σ_n}(π_d, π')` (by the bridge above) produces `covers_{Σ_{n+1}}*(π₁, π')`. ✓

In every sub-case, one of the three disjuncts holds for `(π₁, π')`. By symmetry, the same holds for `(π', π₂)` when `π₁ = π'`. Induction completes the derivation. ∎

NestingByDelegation makes the structural geometry of `Π_Σ` explicit: principals form a forest under the strict-extension order, with the roots being the bootstrap principals of `Π_{Σ_0}`, and parent-child edges supplied by delegation events. Sub-delegates of a principal `π` are precisely the descendants of `π` in the forest, and any other principal in `Π_Σ` has a non-nesting prefix.

**allocated_by_Σ(π, a) (AllocatedBy).**

*Axiom:* `allocated_by_Σ(π, a)` is a primitive relation of the ownership model.
- *Signature:* `allocated_by_Σ : Principal × Tumbler → Bool`
- *Semantics:* `allocated_by_{Σ'}(π, a)` holds when the baptism procedure, executing on behalf of `π`, produced `a` during the transition yielding `Σ'`.

**O5 (SubdivisionAuthority).** Only the principal with the longest matching prefix may allocate new addresses within its domain:

  `(A Σ, Σ', a, π : Σ → Σ' ∧ π ∈ Π_Σ ∧ a ∈ Σ'.B ∖ Σ.B ∧ allocated_by_{Σ'}(π, a)  ⟹  pfx(π) ≼ a  ∧  (A π' ∈ Π_Σ : pfx(π') ≼ a ⟹ #pfx(π') ≤ #pfx(π)))`

**O16 (AllocationClosure).** Every address entering `Σ.B` in a state transition was allocated by some principal in `Π_Σ`:

  `(A Σ, Σ', a : Σ → Σ' ∧ a ∈ Σ'.B ∖ Σ.B  ⟹  (E π ∈ Π_Σ : allocated_by_{Σ'}(π, a)))`

Gregory confirms: every allocation path in udanax-green originates from a session with an account tumbler — there is no mechanism for addresses to appear without an allocating principal.

**O17b (BaptismalRegistryCoupling).** Every ownership transition that changes the baptismal registry does so by an ASN-0040 baptism. Formally, every `Σ → Σ'` falls in one of two branches: a frame branch `Σ'.B = Σ.B` that leaves the registry untouched, or a baptism branch `Σ'.B = Σ.B ∪ {next(Σ.B, p, d)}` for some `(p, d)` satisfying B6 — the transition restricting to a `Bop(p, d)` step (ASN-0040) on the registry component:

  `(A Σ, Σ' : Σ → Σ' ⟹ Σ'.B = Σ.B ∨ (E p, d : B6(p, d) : Σ'.B = Σ.B ∪ {next(Σ.B, p, d)} ∧ next(Σ.B, p, d) ∉ Σ.B))`

The coupling is sharpened for principal-introducing transitions: every transition that admits a new principal `π'` takes the baptism branch — never the frame branch — and the element it baptizes is exactly `pfx(π')`.

  `(A Σ, Σ', π' : Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ Σ'.B = Σ.B ∪ {pfx(π')})`

**O17c (PrincipalPrefixNextForm, derived).** Composing O17b's principal-introduction clause with its general baptism branch, the baptized prefix of every introduced principal has next-reachable form:

  `(A Σ, Σ', π' : Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ (E p, d : B6(p, d) : pfx(π') = next(Σ.B, p, d)))`

**RegistryReachability (derived).** In every reachable state the baptismal registry is an ASN-0040-reachable registry conforming to B₀ conf.:

  `(A Σ : Σ reachable from Σ₀ : Σ.B is an ASN-0040-reachable registry conforming to B₀ conf.)`

*Base case.* `Σ₀.B` is an ASN-0040-reachable registry conforming to B₀ conf., by O14.9 (Registry).

*Inductive step.* By O17b (BaptismalRegistryCoupling), each transition `Σ → Σ'` either leaves the registry untouched (`Σ'.B = Σ.B`, preserving reachability trivially) or restricts to a single `Bop(p, d)` edge with `(p, d)` B6-valid (`Σ'.B = Σ.B ∪ {next(Σ.B, p, d)}`). ASN-0040's transition relation is closed over reachable registries — a `Bop(p, d)` step from an ASN-0040-reachable registry yields an ASN-0040-reachable registry — so `Σ'.B` is reachable whenever `Σ.B` is. ∎

**O17 (AllocatedAddressValidity, derived).** Every allocated address is a valid tumbler:

  `(A Σ reachable from Σ₀, a : a ∈ Σ.B ⟹ T4(a))`

*Proof.* By RegistryReachability (derived above), in every reachable state `Σ` the registry `Σ.B` is an ASN-0040-reachable registry. ASN-0040's B10 (T4ValidityInvariant) holds over every such reachable registry — `(A t ∈ s.B : T4(t))` — so every address in `Σ.B` satisfies T4. ∎

**Freshness-(v).** An alias for the pair `T4(pfx(π')) ∧ pfx(π') ∉ Σ.B` of condition (v).

**Shared invariant induction (O1a / O1b / T4-validity).** O1a (AccountOwnershipBoundary), O1b (PrefixInjectivity), and T4-validity of every principal's prefix are reachable-state invariants, established here by a single induction on the reachability sequence with a common base case and non-delegation step and a per-invariant delegation step. *Base:* the corresponding conjunct of O14 holds on `Π₀` — O14.4 (AccountTier) for O1a (`zeros(pfx(π)) ≤ 1`), O14.5 (Injective) for O1b (`pfx` injective), O14.6 (Valid) for T4 (`T4(pfx(π))`). *Non-delegation step:* O15 admits no new principal and O13 (PrefixImmutability) fixes existing prefixes, so any property of the principal-to-prefix assignment carries unchanged from `Π_Σ` to `Π_{Σ'}`. *Delegation step:* in each case existing principals persist by O12 (PrincipalPersistence) with prefixes preserved by O13, so it remains to discharge the sole new principal `π'`:

- *O1a:* `π'` satisfies `zeros(pfx(π')) ≤ 1` by condition (iii).
- *T4:* Freshness-(v) supplies `T4(pfx(π'))` directly.
- *O1b:* suppose for contradiction `pfx(π') = pfx(π''')` for some existing `π''' ∈ Π_Σ`. Then `pfx(π''') ≼ pfx(π')`, so by condition (ii) `#pfx(π''') ≤ #pfx(π)`; but condition (i) gives `#pfx(π) < #pfx(π')`, so `#pfx(π''') ≤ #pfx(π) < #pfx(π') = #pfx(π''')` — a contradiction. Since O15 admits at most one new principal per transition, no collision among newcomers is possible; existing-vs-existing distinctness carries from `Σ` to `Σ'` by O13 (the inductive hypothesis O1b at `Σ` gives `pfx_Σ(π'_1) ≠ pfx_Σ(π'_2)` for distinct `π'_1, π'_2 ∈ Π_Σ`, and O13 preserves both prefixes).

Hence every principal in every reachable state satisfies all three invariants. ∎

**O18 (DelegationBaptizes, derived).** Delegation materially baptizes the delegate's prefix freshly — the transition that introduces a new principal into `Π` enters its prefix into `Σ.B` as a newly registered tumbler, not previously present:

  `(A Σ, Σ', π' : Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ pfx(π') ∈ Σ'.B ∖ Σ.B)`

*Proof.* By O17b's principal-introduction coupling, the introducing transition takes the baptism branch with `Σ'.B = Σ.B ∪ {pfx(π')}`, so `pfx(π') ∈ Σ'.B`; Freshness-(v) gives `pfx(π') ∉ Σ.B`. Hence `pfx(π') ∈ Σ'.B ∖ Σ.B`. ∎

**NamespacePrincipalExclusivity (derived).** For any prefix `p`, once `p ∈ Σ.B` no later transition can adopt `p` as a new principal's prefix — immediate from Freshness-(v) (a delegate prefix must satisfy `p ∉ Σ.B`) under B0 monotonicity of `Σ.B` (ASN-0040), which makes the freshness failure permanent.

**PrefixBaptismCoupling (derived).** In every reachable state, every principal's prefix is itself baptized:

  `(A Σ : Σ reachable from Σ₀ : (A π ∈ Π_Σ : pfx(π) ∈ Σ.B))`

We derive this by induction on the transition sequence `Σ₀ → Σ_1 → ... → Σ`.

*Base case.* In the initial state `Σ₀`, the claim is `(A π ∈ Π_{Σ_0} : pfx(π) ∈ Σ_0.B)`. This is O14.8 (Baptized) directly.

*Inductive step.* Assume every `π ∈ Π_{Σ_n}` satisfies `pfx_{Σ_n}(π) ∈ Σ_n.B`, and consider a transition `Σ_n → Σ_{n+1}`. By O15 (PrincipalClosure), every principal in `Π_{Σ_{n+1}}` either was already present in `Π_{Σ_n}` or is the unique newcomer admitted by the delegation conditions (since `|Π_{Σ_{n+1}} ∖ Π_{Σ_n}| ≤ 1`). Let `π ∈ Π_{Σ_{n+1}}` be arbitrary; two cases exhaust its membership.

*Case 1 — `π ∈ Π_{Σ_n}` (carried forward).* By O13 (PrefixImmutability), `pfx_{Σ_{n+1}}(π) = pfx_{Σ_n}(π)`, and the IH gives `pfx_{Σ_n}(π) ∈ Σ_n.B`. By B0 (Irrevocability) of ASN-0040, `Σ_n.B ⊆ Σ_{n+1}.B`, so `pfx_{Σ_{n+1}}(π) ∈ Σ_{n+1}.B`.

*Case 2 — `π ∈ Π_{Σ_{n+1}} ∖ Π_{Σ_n}` (newly introduced).* By O18 (DelegationBaptizes), `pfx(π) ∈ Σ_{n+1}.B` directly.

In both cases, `pfx(π) ∈ Σ_{n+1}.B`. Induction on the transition sequence carries the property to every reachable state. ∎

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`.
- *Postconditions:* `pfx(π) ∈ Σ.B`.
- *Invariant:* Principal registry and baptismal registry are coupled in every reachable state — no principal exists without an allocated prefix.


## The Exclusivity Invariant

Can two principals simultaneously own the same address?

Nelson uses the definite article throughout: "*the* owner of a given item" (LM 4/20), not "an owner." Exclusivity, however, is not a property of the ownership predicate. Gregory's `tumbleraccounteq` decides *containment* — whether one account tumbler is a prefix of an address — and for a nested address `a` with `pfx(π_N) = [1] ≼ a` and `pfx(π_A) = [1, 0, 2] ≼ a` the containment check returns true for *both* distinct principals. Exclusivity is instead a property of the effective-ownership function `ω`, established by the longest-match selection rule that defines it (O2).

For non-nesting prefixes, T10 (PartitionIndependence) gives disjointness immediately: two principals whose prefixes satisfy `pfx(π₁) ⋠ pfx(π₂) ∧ pfx(π₂) ⋠ pfx(π₁)` have disjoint domains. The interesting case is nested domains — when a node operator's domain contains an account holder's. Here, Nelson is explicit: the node operator creates accounts, but "once assigned a User account, the user will have full control over its subdivision forevermore" (LM 4/29). Delegation permanently transfers effective ownership of the subdomain.

We first state a coverage requirement — every allocated address falls within some principal's domain:

**O4 (DomainCoverage).** For every allocated address in any reachable state, at least one principal's prefix contains it:

  `(A Σ : Σ reachable from Σ₀ : (A a ∈ Σ.B : (E π ∈ Π_Σ : pfx(π) ≼ a)))`

The reachability quantifier is essential: the proof is by induction on the length of the transition sequence leading to `Σ`, and the induction operates along the witnessing path. We prove that in every reachable state `Σ`, every allocated address is covered by at least one principal's prefix.

*Base case.* In the initial state `Σ₀`, the claim is `(A a ∈ Σ₀.B : (E π ∈ Π₀ : pfx(π) ≼ a))`. This is O14.2 (Coverage), which asserts exactly that the initial principals cover all initially allocated addresses. The base case holds.

*Inductive step.* Assume the claim holds in state `Σ`: every `a ∈ Σ.B` has a covering principal in `Π_Σ`. We must show it holds in any successor state `Σ'` with `Σ → Σ'`. Let `a ∈ Σ'.B` be an arbitrary allocated address. Two cases arise, exhausting `Σ'.B = Σ.B ∪ (Σ'.B ∖ Σ.B)`.

*Case 1: `a ∈ Σ.B` (address was already allocated).* By the inductive hypothesis, there exists `π ∈ Π_Σ` with `pfx(π) ≼ a`. By O12 (PrincipalPersistence), `Π_Σ ⊆ Π_{Σ'}`, so `π ∈ Π_{Σ'}`. By O13 (PrefixImmutability), `pfx_{Σ'}(π) = pfx_Σ(π)`, so the prefix relation `pfx_{Σ'}(π) ≼ a` is preserved. Hence `a` has a covering principal in `Π_{Σ'}`.

*Case 2: `a ∈ Σ'.B ∖ Σ.B` (address is newly allocated).* By O16 (AllocationClosure), there exists a principal `π ∈ Π_Σ` such that `allocated_by_{Σ'}(π, a)` — every newly allocated address was allocated by some existing principal. By O5 (SubdivisionAuthority), whenever `π` allocates `a`, the first conjunct of the postcondition gives `pfx(π) ≼ a` — the allocator's prefix covers the allocated address. By O12, `π ∈ Π_Σ ⊆ Π_{Σ'}`, and by O13, `pfx_{Σ'}(π) = pfx_Σ(π)`. Hence `pfx_{Σ'}(π) ≼ a`, and `a` has a covering principal in `Π_{Σ'}`.

In both cases, every address in `Σ'.B` is covered by a principal in `Π_{Σ'}`. By induction on the transition sequence, the coverage invariant holds in every reachable state. ∎

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`.
- *Postconditions:* `(E π ∈ Π_Σ : pfx(π) ≼ a)`.
- *Invariant:* Coverage holds in every reachable state — no allocated address is orphaned from the principal hierarchy.

We resolve nesting by specificity. Before stating exclusivity we name the principal that wins the contest:

**ω_Σ(a) (EffectiveOwner).** The *effective owner* of an allocated address `a` at a reachable state `Σ` is the principal in `Π_Σ` with the longest matching prefix. Formally, `ω_Σ` is the partial function on `T` with domain `Σ.B`, written `ω_Σ : Σ.B → Π_Σ`, defined by:

  `ω_Σ(a) = π  ≡  π ∈ Π_Σ  ∧  pfx(π) ≼ a  ∧  (A π' ∈ Π_Σ : π' ≠ π ∧ pfx(π') ≼ a : #pfx(π) > #pfx(π'))`

The domain restriction `ω_Σ : Σ.B → Π_Σ` state-relativizes both the address `a` (input) and the selected principal (output): the quantifier ranges over the state-relativized principal registry `Π_Σ` rather than a global `Π`.

*Notation.* We write bare `ω(a)` and `Π` for `ω_Σ(a)` and `Π_Σ` when the state is fixed by context, supplying the subscript whenever states must be disambiguated.

**O2 (OwnershipExclusivity).** For every reachable state `Σ` and every allocated address `a ∈ Σ.B`, there exists exactly one principal in `Π_Σ` that effectively owns `a` — equivalently, `ω_Σ : Σ.B → Π_Σ` is a total well-defined function:

  `(A Σ reachable, a ∈ Σ.B : (E! π ∈ Π_Σ : ω_Σ(a) = π))`

We prove that for every `a ∈ Σ.B` exactly one principal `π` satisfies the defining equivalence of `ω(a)`. The argument decomposes into four steps: non-emptiness of the covering set, total ordering of covering prefixes, finiteness, and uniqueness of the witnessing principal.

*Step 1: Non-emptiness.* Let `a ∈ Σ.B` and define `C(a) = {π ∈ Π : pfx(π) ≼ a}`, the set of principals whose prefix covers `a`. By O4 (DomainCoverage), every allocated address falls within at least one principal's domain, so `C(a) ≠ ∅`.

*Step 2: Total ordering of covering prefixes.* By the covering-chain lemma, any two tumbler prefixes of a common address are `≼`-comparable. Applied to `pfx(π₁), pfx(π₂)` for any `π₁, π₂ ∈ C(a)` — both prefixes of the common address `a` — the prefixes are comparable. Since `π₁, π₂` were arbitrary members of `C(a)`, `{pfx(π) : π ∈ C(a)}` is a chain under `≼`.

*Step 3: Finiteness.* Each covering prefix `p ≼ a` is uniquely determined by its length: since `p ≼ a` requires `pᵢ = aᵢ` for all `1 ≤ i ≤ #p`, the prefix of length `k` covering `a` can only be `[a₁, …, a_k]`. There are at most `#a` possible lengths (from `1` to `#a`), so at most `#a` distinct covering prefixes exist; and by O1b (PrefixInjectivity), each such prefix is held by at most one principal, so `|C(a)| ≤ #a`. The covering set is finite.

*Step 4: Existence and uniqueness of the maximum.* A non-empty finite chain has a unique maximum. Therefore there exists a unique maximal length `ℓ* = max{#pfx(π) : π ∈ C(a)}`, and by Step 3 the covering prefix of length `ℓ*` is uniquely determined as `[a₁, …, a_{ℓ*}]`. It remains to show that exactly one principal holds this prefix. Suppose `π₁, π₂ ∈ C(a)` both satisfy `#pfx(π₁) = #pfx(π₂) = ℓ*`. By Step 3, `pfx(π₁) = [a₁, …, a_{ℓ*}] = pfx(π₂)`. By O1b (PrefixInjectivity), equal prefixes imply `π₁ = π₂`. Hence there is exactly one principal `π* ∈ C(a)` achieving the maximal prefix length, and `π*` satisfies the defining equivalence: `pfx(π*) ≼ a` and for every `π' ≠ π*` with `pfx(π') ≼ a`, `#pfx(π*) > #pfx(π')`.

We conclude: for every `a ∈ Σ.B`, there exists exactly one `π ∈ Π` with `ω(a) = π`. Equivalently, `ω : Σ.B → Π` is a total well-defined function in every reachable state. ∎

*Formal Contract:*
- *Definition:* `ω_Σ : Σ.B → Π_Σ` with `ω_Σ(a) = π ≡ π ∈ Π_Σ ∧ pfx(π) ≼ a ∧ (A π' ∈ Π_Σ : π' ≠ π ∧ pfx(π') ≼ a ⟹ #pfx(π) > #pfx(π'))`.
- *Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`. Reachability is inherited from O4 (invoked in Step 1).
- *Postconditions:* `(E! π ∈ Π_Σ : ω_Σ(a) = π)` — exactly one principal satisfies the defining equivalence.
- *Invariant:* `ω` is a total well-defined function on `Σ.B` in every reachable state.

**SelfOwnershipAtPrefix (derived).** Every principal is the effective owner of its own prefix:

  `(A Σ : Σ reachable from Σ₀ : (A π ∈ Π_Σ : ω_Σ(pfx(π)) = π))`

Let `Σ` be reachable from `Σ₀` and `π ∈ Π_Σ`. By PrefixBaptismCoupling, `pfx(π) ∈ Σ.B`, so `ω_Σ(pfx(π))` is defined by O2. We show `π` is the unique longest-match principal at `pfx(π)`. Write `C(pfx(π)) = {π' ∈ Π_Σ : pfx(π') ≼ pfx(π)}` for the set of principals whose prefix covers `pfx(π)`. Reflexivity of the prefix relation gives `pfx(π) ≼ pfx(π)`, so `π ∈ C(pfx(π))`. For any other `π'' ∈ C(pfx(π))` with `π'' ≠ π`: `pfx(π'') ≼ pfx(π)`, and by O1b (PrefixInjectivity) `π'' ≠ π` forces `pfx(π'') ≠ pfx(π)`. The conjunction `pfx(π'') ≼ pfx(π) ∧ pfx(π'') ≠ pfx(π)` yields the strict prefix `pfx(π'') ≺ pfx(π)`, hence `#pfx(π'') < #pfx(π)`. Therefore `π` achieves the strictly longest match in `C(pfx(π))`. By O2's defining equivalence, `ω_Σ(pfx(π)) = π`. ∎

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`.
- *Postconditions:* `ω_Σ(pfx(π)) = π`.
- *Invariant:* The boundary of `odom(π)` is structurally inhabited by `π` itself — every principal owns its own delegation point in every reachable state.

The exclusivity of ownership is load-bearing. If two parties owned the same address, the system could not determine who is entitled to subdivide the space beneath it, who originated the content, or whose delegation created the address.


## Permanence and Refinement

Nelson is emphatic: ownership does not expire.

> "Once assigned a User account, the user will have full control over its subdivision forevermore." (LM 4/29)

"Forevermore" is strong language in a technical specification. But the naive reading — that `ω(a)` never changes — is too strong. Consider a node operator `π₁` with `pfx(π₁) = [1]`. Before any delegation, `ω(a) = π₁` for every address `a` with node field `1`. When `π₁` delegates account prefix `[1, 0, 2]` to principal `π₂`, the effective owner of every address under `[1, 0, 2]` changes from `π₁` to `π₂` — the longer prefix wins. So "forevermore" cannot mean `ω` is static.

The correct invariant is monotonic refinement — `ω(a)` can change only through delegation, and only by becoming more specific:

**O3 (OwnershipRefinement).** The effective owner of an address changes only when delegation introduces a principal with a strictly longer matching prefix. No other transition alters `ω`:

  `(A a ∈ Σ.B, Σ, Σ' : Σ reachable from Σ₀ ∧ Σ → Σ' ∧ ω_{Σ'}(a) ≠ ω_Σ(a)  ⟹  (E π_d ∈ Π_Σ, π' ∈ Π_{Σ'} ∖ Π_Σ : pfx(π') ≼ a ∧ #pfx(π') > #pfx(ω_Σ(a)) ∧ delegated_Σ(π_d, π')))`

We prove that every change in effective ownership is witnessed by a new principal with a strictly longer matching prefix, by examining what the effective owner function depends on and what a state transition can alter.

The effective owner `ω_Σ(a)` is defined (O2) as the principal in `Π_Σ` with the longest prefix matching `a`. This definition depends on exactly three inputs: the address `a`, the set of principals `Π_Σ`, and the prefix function `pfx` restricted to `Π_Σ`. We show that a transition `Σ → Σ'` can disturb at most one of these inputs.

*The address is invariant.* By B0 (Irrevocability) of ASN-0040, once `a ∈ Σ.B`, the address `a` persists in the baptismal registry of every subsequent state with unchanged components.

*No existing principal is removed.* By O12 (PrincipalPersistence), `Π_Σ ⊆ Π_{Σ'}`. Every principal present in `Σ` remains present in `Σ'`.

*No existing prefix is altered.* By O13 (PrefixImmutability), for every `π ∈ Π_Σ`, `pfx_{Σ'}(π) = pfx_Σ(π)`. The prefix of every surviving principal is identical across the transition.

These three facts together imply that the set of covering principals from `Π_Σ` is preserved exactly:

  `{π ∈ Π_Σ : pfx_Σ(π) ≼ a} = {π ∈ Π_{Σ'} ∩ Π_Σ : pfx_{Σ'}(π) ≼ a}`

This equality follows from O12 (`Π_Σ ⊆ Π_{Σ'}`) and O13 (`pfx_{Σ'} = pfx_Σ` on `Π_Σ`). In particular, the longest match among `Π_Σ` — which is `ω_Σ(a)` — remains a covering principal in `Σ'` with the same prefix length.

Now suppose `ω_{Σ'}(a) ≠ ω_Σ(a)`. Since `ω_Σ(a)` is still present in `Π_{Σ'}` with the same prefix (by O12 and O13), and since `ω_Σ(a)` was the longest match in `Π_Σ`, the only way for the longest-match computation over `Π_{Σ'}` to yield a *different* result is for some principal in `Π_{Σ'} ∖ Π_Σ` to cover `a` with a strictly longer prefix. That is, there must exist `π' ∈ Π_{Σ'} ∖ Π_Σ` satisfying both `pfx(π') ≼ a` and `#pfx(π') > #pfx(ω_Σ(a))`.

To see why the new principal's prefix must be *strictly* longer: if `#pfx(π') ≤ #pfx(ω_Σ(a))`, then `ω_Σ(a)` would still be the longest (or tied-longest) match. The equal-length case `#pfx(π') = #pfx(ω_Σ(a))` is ruled out by the same equal-prefix step that excludes case (1) of the StrictLongestCover lemma: two coverers of the common address `a` with equal prefix length agree componentwise with `a` over their shared length (Prefix), so their prefixes are identical and O1b (PrefixInjectivity) forces `π' = ω_Σ(a)` — impossible, since `π' ∈ Π_{Σ'} ∖ Π_Σ` while `ω_Σ(a) ∈ Π_Σ`. The shorter case `#pfx(π') < #pfx(ω_Σ(a))` leaves `ω_Σ(a)` as the longest match, contradicting `ω_{Σ'}(a) ≠ ω_Σ(a)`. So a new covering principal can only displace `ω_Σ(a)` by being strictly longer.

By O15 (PrincipalClosure), `π' ∈ Π_{Σ'} ∖ Π_Σ` arrived through bootstrap or through delegation. By BootstrapContainment, `Π₀ ⊆ Π_Σ`; combined with `π' ∉ Π_Σ`, the bootstrap case is excluded. The remaining clause of O15 supplies an existing principal `π_d ∈ Π_Σ` for which the full delegation predicate `delegated_Σ(π_d, π')` — all five conditions, with `T4(pfx(π'))` and freshness following by Freshness-(v) — held at `π'`'s introducing event. Every change to `ω(a)` is attributable to a specific delegation act in the transition `Σ → Σ'`, witnessed by the pair `(π_d, π')`.

We conclude: `ω_{Σ'}(a) ≠ ω_Σ(a)` implies `(E π_d ∈ Π_Σ, π' ∈ Π_{Σ'} ∖ Π_Σ : pfx(π') ≼ a ∧ #pfx(π') > #pfx(ω_Σ(a)) ∧ delegated_Σ(π_d, π'))` — the new principal `π'` arrived via a specific delegation act by `π_d`, not by bootstrap. ∎

*Corollary (monotonic refinement).* For every transition `Σ → Σ'` between reachable states and every `a ∈ Σ.B`, `#pfx(ω_{Σ'}(a)) ≥ #pfx(ω_Σ(a))`. The precondition `a ∈ Σ.B` ensures `ω_Σ(a)` is defined; the corollary then derives `ω_{Σ'}(a)` is defined via B0 (Irrevocability of ASN-0040), since `Σ.B ⊆ Σ'.B` so `a ∈ Σ'.B`. We split on whether the effective owner changes. *Case `ω_{Σ'}(a) = ω_Σ(a)`:* the same principal owns `a` in both states, and by O13 (PrefixImmutability) its prefix is unchanged, so `#pfx(ω_{Σ'}(a)) = #pfx(ω_Σ(a))`. *Case `ω_{Σ'}(a) ≠ ω_Σ(a)`:* by the proof body just established, the new effective owner has a strictly longer prefix, so `#pfx(ω_{Σ'}(a)) > #pfx(ω_Σ(a))`. Both cases yield `#pfx(ω_{Σ'}(a)) ≥ #pfx(ω_Σ(a))`. Once a principal `π` becomes the effective owner through longest-match, only a *more specific* delegation can supersede it.

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `a ∈ Σ.B`, `Σ → Σ'`, `ω_{Σ'}(a) ≠ ω_Σ(a)`.
- *Postconditions:* `(E π_d ∈ Π_Σ, π' ∈ Π_{Σ'} ∖ Π_Σ : pfx(π') ≼ a ∧ #pfx(π') > #pfx(ω_Σ(a)) ∧ delegated_Σ(π_d, π'))` — the change is witnessed by both the new principal `π'` (with a strictly longer matching prefix) and the delegator `π_d` (the existing principal whose authority condition (ii) admits `π'`). Monotonic-refinement corollary: `#pfx(ω_{Σ'}(a)) ≥ #pfx(ω_Σ(a))` for `a ∈ Σ.B`.

**OwnershipDomainPermanence (Ownership-domain permanence).** No principal external to `odom(π)` can alter effective ownership within `odom(π)`. Changes to `ω(a)` for addresses in a principal's domain arise only from that principal's own delegation acts or from delegation acts of its sub-delegates:

  `(A π ∈ Π_Σ, Σ, Σ' : Σ → Σ' ∧ (E a ∈ odom(π) ∩ Σ.B : ω_{Σ'}(a) ≠ ω_Σ(a))  ⟹  (E π_d ∈ Π_Σ : pfx(π) ≼ pfx(π_d) ∧ covers_Σ*(π, π_d) ∧ (E π' ∈ Π_{Σ'} ∖ Π_Σ : delegated_Σ(π_d, π'))))`

That is: if any address in `odom(π)` changes effective owner across a single transition, the delegator `π_d` responsible for that transition has a prefix extending `pfx(π)` and is reached from `π` along the delegation cover-chain — formally `covers_Σ*(π, π_d)`, the relation of the *Principal Registry* section. In words, `π_d` is `π` itself or a sub-delegate of `π`.

We prove this directly for a single transition `Σ → Σ'`. The formal statement quantifies over one transition; we make no induction on transition count.

Assume `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ odom(π) ∩ Σ.B`, `Σ → Σ'`, and `ω_{Σ'}(a) ≠ ω_Σ(a)`.

*Step 1 — a new principal with a strictly longer matching prefix witnesses the change.* By O3 (OwnershipRefinement), `ω_{Σ'}(a) ≠ ω_Σ(a)` implies the existence of `π' ∈ Π_{Σ'} ∖ Π_Σ` with `pfx(π') ≼ a` and `#pfx(π') > #pfx(ω_Σ(a))`. By O15 (PrincipalClosure), `π'` entered `Π` either through bootstrap (`π' ∈ Π₀`) or through delegation. By BootstrapContainment, `Π₀ ⊆ Π_Σ`; combined with `π' ∉ Π_Σ`, this excludes `π' ∈ Π₀`, ruling out the bootstrap case. The remaining clause of O15 applies: there exists `π_d ∈ Π_Σ` with `delegated_Σ(π_d, π')`.

*Step 2 — the new principal's prefix strictly extends `pfx(π)`.* Since `a ∈ odom(π)`, we have `pfx(π) ≼ a`. The chain `#pfx(π') > #pfx(ω_Σ(a)) ≥ #pfx(π)` holds: the second inequality follows because `π ∈ Π_Σ` covers `a`, and `ω_Σ(a)` is by O2 the longest-prefix covering principal in `Π_Σ`. Hence `#pfx(π') > #pfx(π)`. Both `pfx(π)` and `pfx(π')` are prefixes of the common address `a`, so by the covering-chain lemma they are `≼`-comparable; the strict length inequality `#pfx(π) < #pfx(π')` fixes the direction, giving `pfx(π) ≼ pfx(π')` and hence `pfx(π) ≺ pfx(π')`.

*Step 3 — the delegator's prefix extends `pfx(π)`.* By condition (i) of the `delegated` relation, `pfx(π_d) ≺ pfx(π')`. By condition (ii), `π_d` is the most-specific covering principal of `pfx(π')` in `Π_Σ`: `(A π'' ∈ Π_Σ : pfx(π'') ≼ pfx(π') ⟹ #pfx(π'') ≤ #pfx(π_d))`. From Step 2, `pfx(π) ≼ pfx(π')` and `π ∈ Π_Σ`, so taking `π'' = π` gives `#pfx(π) ≤ #pfx(π_d)`. Both `pfx(π)` and `pfx(π_d)` are prefixes of the common tumbler `pfx(π')` (the former by Step 2; the latter by condition (i)), so by the covering-chain lemma they are `≼`-comparable; the length inequality `#pfx(π) ≤ #pfx(π_d)` fixes the direction, giving `pfx(π) ≼ pfx(π_d)`.

*Step 4 — the delegator satisfies `covers_Σ*(π, π_d)`.* Both `π` and `π_d` lie in `Π_Σ`, and Step 3 gives `pfx(π) ≼ pfx(π_d)`. If `pfx(π) = pfx(π_d)`, then O1b (PrefixInjectivity) forces `π = π_d`, and `covers_Σ*(π, π_d)` holds reflexively (`R_Σ^0` is the identity on `Π_Σ`). Otherwise `pfx(π) ≺ pfx(π_d)`, and the pair `π, π_d ∈ Π_Σ` is distinct. Apply NestingByDelegation to this pair: its non-nesting disjunct is excluded because `pfx(π) ≺ pfx(π_d)` nests them, and its reverse-extension disjunct `pfx(π_d) ≺ pfx(π)` is excluded by the same strict ordering. Only the middle disjunct survives, yielding `covers_Σ*(π, π_d)` — `π_d` is reached from `π` by a finite chain of cover edges, which by the delegation-edges-are-cover-edges bridge is a chain of delegation events.

Steps 1–4 establish the postcondition for a single transition: `(E π_d ∈ Π_Σ : pfx(π) ≼ pfx(π_d) ∧ covers_Σ*(π, π_d) ∧ (E π' ∈ Π_{Σ'} ∖ Π_Σ : delegated_Σ(π_d, π')))`. ∎

**Corollary (OwnershipDomainPermanence★, multi-step).** The single-transition property extends to the transitive closure `Σ →⁺ Σ'`: every change to `ω(a)` for `a ∈ odom(π) ∩ Σ.B` along any reachable transition sequence is induced by delegators whose prefixes all extend `pfx(π)`. Let `Σ →⁺ Σ'` abbreviate `Σ → Σ_1 → ... → Σ_n = Σ'` for some `n ≥ 1`:

  `(A π ∈ Π_Σ, Σ, Σ', a : Σ reachable from Σ₀ ∧ Σ →⁺ Σ' ∧ a ∈ odom(π) ∩ Σ.B  ⟹  (A i, 0 ≤ i < n : ω_{Σ_{i+1}}(a) ≠ ω_{Σ_i}(a) ⟹ (E π_d^{(i)} ∈ Π_{Σ_i}, π'^{(i)} ∈ Π_{Σ_{i+1}} ∖ Π_{Σ_i} : pfx(π) ≼ pfx(π_d^{(i)}) ∧ delegated_{Σ_i}(π_d^{(i)}, π'^{(i)}))))`

We prove this by induction on the path length `n`. The reachability of each intermediate `Σ_i` is automatic: `Σ` is reachable from `Σ₀` by hypothesis, and `Σ →⁺ Σ_i` extends the witnessing sequence, so `Σ_i` is reachable from `Σ₀` for every `0 ≤ i ≤ n`. *Base case `n = 1`:* The hypothesis reduces to a single transition `Σ → Σ'`; the single-transition OwnershipDomainPermanence applies directly, yielding the required `π_d^{(0)}` with `pfx(π) ≼ pfx(π_d^{(0)})` whenever `ω(a)` changes.

*Inductive step.* Assume the corollary holds for sequences of length `n`; consider a sequence of length `n + 1`: `Σ → Σ_1 → ... → Σ_n → Σ_{n+1}`. By the induction hypothesis applied to the prefix `Σ →⁺ Σ_n`, the chain conclusion holds for every transition with index `0 ≤ i < n` along that prefix. It remains to handle the final transition `Σ_n → Σ_{n+1}`. The single-transition OwnershipDomainPermanence applies provided `Σ_n` reachable from `Σ₀` (discharged above) and `a ∈ odom(π) ∩ Σ_n.B` — we discharge the latter from the original hypotheses. The persistence of `a` follows from B0★ (MultiStepIrrevocability) of ASN-0040 applied along the path `Σ →⁺ Σ_n`: `a ∈ Σ.B ⊆ Σ_n.B` since the baptismal registry is monotone under the reflexive-transitive closure of `→`. The persistence of `π ∈ Π_{Σ_n}` follows from iterated O12: `Π_Σ ⊆ Π_{Σ_1} ⊆ ... ⊆ Π_{Σ_n}`, so `π ∈ Π_{Σ_n}`. The persistence of `a ∈ odom(π)` is structural — `odom(π) = {a : pfx(π) ≼ a}` depends only on `pfx(π)` and `a`, both of which are fixed as values (O13 immutability for the prefix; tumbler addresses are immutable values in `Σ.B` under B0 of ASN-0040). With premises discharged, the single-transition statement yields the required `π_d^{(n)}` with `pfx(π) ≼ pfx(π_d^{(n)})` whenever `ω_{Σ_{n+1}}(a) ≠ ω_{Σ_n}(a)`. Combined with the inductive conclusion for the earlier transitions, the chain conclusion holds for all `0 ≤ i ≤ n`. ∎

The corollary localizes ownership-change provenance to a principal's domain: every delegator that participates in a chain of changes to `ω(a)` within `odom(π)` has a prefix extending `pfx(π)`, so changes to `ω` within `odom(π)` arise only from `π`'s own delegation choices or, recursively, from sub-delegates' choices within their own sub-domains. No delegator outside `odom(π)` can induce a change, and the addresses `π` has not sub-delegated remain under `π`'s effective ownership.

Nelson's mention of "someone who has bought the document rights" (LM 2/29) implies ownership can *transfer*, but Gregory's codebase contains no transfer mechanism; O3 describes the refinement regime for the system as specified.

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ odom(π) ∩ Σ.B`, `Σ → Σ'`, `ω_{Σ'}(a) ≠ ω_Σ(a)`.
- *Postconditions:* `(E π_d ∈ Π_Σ : pfx(π) ≼ pfx(π_d) ∧ covers_Σ*(π, π_d) ∧ (E π' ∈ Π_{Σ'} ∖ Π_Σ : delegated_Σ(π_d, π')))` — the responsible delegator `π_d` and the newly introduced principal `π'` are both witnessed existentially; the new principal is the cause of the ownership change, and the delegator `π_d` satisfies `covers_Σ*(π, π_d)` (by NestingByDelegation).
- *Invariant:* Effective ownership within `odom(π)` is sovereign — no delegation by a principal external to `odom(π)` can alter `ω(a)` for any `a ∈ odom(π)`.


## Structural Provenance

The ownership prefix is embedded in the permanent address. Because every principal's prefix satisfies `zeros(pfx(π)) ≤ 1` (O1a), the longest-match computation depends only on the node and user fields — the portion captured by `acct(a)`. The document and element fields are irrelevant to ownership determination.

**O6 (StructuralProvenance).** The effective owner of an allocated address is determined entirely by its account field:

  `(A Σ reachable, a, b ∈ Σ.B : acct(a) = acct(b) ⟹ ω(a) = ω(b))`

We prove that equal account fields imply equal effective owners by showing that the prefix comparisons determining ownership depend only on the account field. The argument requires a structural property of `acct`: for any valid tumbler `a`, the account field is a prefix of the address itself:

**AccountPrefix (AccountPrefix).** `(A a ∈ T : T4(a) ⟹ acct(a) ≼ a)`

We prove that for any tumbler `a` satisfying T4 (HierarchicalParsing), `acct(a) ≼ a` — the account field is a prefix of the address. The T4 restriction is essential: `acct` relies on the field decomposition `fields(a)` whose well-definedness is given by T4b (UniqueParse) — for a tumbler like `[0, 0, 1]`, adjacent zeros violate T4a (SyntacticEquivalence) and the field decomposition is ill-defined. By O17 (AllocatedAddressValidity, derived from ASN-0040 B10), all allocated addresses satisfy T4, so the restriction does not limit application.

The Prefix (PrefixRelation) definition of ASN-0034 requires two conditions: `#a ≥ #acct(a)` and `(A i : 1 ≤ i ≤ #acct(a) : acct(a)ᵢ = aᵢ)`. Both follow directly from FieldStructure (established with AccountField above). When `zeros(a) = 0`, FieldStructure gives `acct(a) = a`, so `acct(a) ≼ a` reflexively. When `zeros(a) ≥ 1`, FieldStructure gives that `acct(a) = N(a) ++ [0] ++ U(a)` reproduces exactly the leading `#acct(a) = α + 1 + β` components of `a` (discharging the component condition), and that any document/element fields occupy positions strictly after `α + 1 + β` — so `#a = #acct(a)` when `zeros(a) = 1` (no further fields) and `#a > #acct(a)` when `zeros(a) ≥ 2` (discharging the length condition). Hence `acct(a) ≼ a` in every case, with equality when `zeros(a) ≤ 1` and strict prefix when `zeros(a) ≥ 2`. ∎

*Formal Contract:*
- *Preconditions:* `a ∈ T`, `T4(a)`.
- *Definition:* `acct(a)` as defined in AccountField (Formal Contract, *The Account-Level Boundary* section).
- *Postconditions:* `acct(a) ≼ a`. When `zeros(a) ≤ 1`: `acct(a) = a` (equality). When `zeros(a) ≥ 2`: `acct(a) ≺ a` (strict prefix).

The proof of O6 proceeds in two directions, under the precondition that `Σ` is reachable from `Σ₀` — the condition that licenses the appeal to O1a (AccountOwnershipBoundary), a derived reachable-state invariant. *Forward:* we must show that for any principal `π` — by O1a, every principal satisfies `zeros(pfx(π)) ≤ 1` — the relation `pfx(π) ≼ a` implies `pfx(π) ≼ acct(a)`. The decomposition steps below apply `fields(a)`, T4b, and T4c to `a`; their precondition `T4(a)` holds by O17, since `a ∈ Σ.B`. Two cases arise from the zero count.

When `zeros(pfx(π)) = 0`: the prefix contains no zero separators, so every component of `pfx(π)` is nonzero. Since `pfx(π) ≼ a`, the first `#pfx(π)` components of `a` all equal the corresponding components of `pfx(π)`, and are therefore all nonzero. Two sub-cases arise from the zero count of `a`.

When `zeros(a) = 0`: by T4c (LevelDetermination), zero count zero means the tumbler is a node-level address — the entire sequence is the node field, so `acct(a) = a`. Since `pfx(π) ≼ a = acct(a)`, the result is immediate.

When `zeros(a) ≥ 1`: by T4b (UniqueParse), `fields(a)` decomposes `a` uniquely; the components preceding `a`'s first zero separator constitute `a`'s node field `N(a)`. Since `pfx(π)`'s components are all nonzero and match `a`'s leading components, `pfx(π)` lies entirely within `a`'s node field: `pfx(π) ≼ N(a)`. And `N(a) ≼ acct(a)` by the definition of `acct` (which includes the node field and, when present, the user field). Hence `pfx(π) ≼ acct(a)`.

In both sub-cases, `pfx(π) ≼ acct(a)`.

When `zeros(pfx(π)) = 1`: the prefix has the form `N₁...Nα.0.U₁...Uβ`, with a zero separator at position `α + 1`. The prefix relation `pfx(π) ≼ a` forces `a_{α+1} = 0`, hence `zeros(a) ≥ 1`. By T4's positive-component constraint applied to `a`, all components before this zero are positive (they match `N₁...Nα`, which are positive by T4 applied to `pfx(π)`), so by T4a (SyntacticEquivalence) this zero cannot be adjacent to another zero or appear at position 1; by T4b (UniqueParse) applied to `a`, `a`'s field decomposition is unique, and since positions `1..α` are positive while `a_{α+1} = 0`, position `α + 1` is uniquely identified as `a`'s node-user field separator. This aligns `pfx(π)`'s field structure with `a`'s: the node fields match (`a`'s node field is `N₁...Nα`), and the prefix relation forces `pfx(π)`'s user-field components `U₁...Uβ` to match the first `β` components of `a`'s user field. Since `acct(a)` captures `a` through its full user field, `pfx(π) ≼ acct(a)`.

In both cases, `pfx(π) ≼ a` implies `pfx(π) ≼ acct(a)`. *Reverse:* suppose `pfx(π) ≼ acct(a)`. By AccountPrefix, `acct(a) ≼ a`. By transitivity of the prefix relation, `pfx(π) ≼ a`. We conclude the biconditional:

  `pfx(π) ≼ a  ≡  pfx(π) ≼ acct(a)`

Now, when `acct(a) = acct(b)`, substitution gives `pfx(π) ≼ acct(a) ≡ pfx(π) ≼ acct(b)`, and hence `pfx(π) ≼ a ≡ pfx(π) ≼ b`. The set of covering principals is identical for `a` and `b`. By O2 (OwnershipExclusivity), the effective owner `ω` is the unique longest-match principal in the covering set; since the covering sets coincide, the longest match is the same, giving `ω(a) = ω(b)`. ∎

*Corollary (owner prefix containment).* The effective owner's prefix is always embedded within the account field: `pfx(ω(a)) ≼ acct(a)`. This is the O6 biconditional instantiated at `π = ω(a)`: by O1a, `zeros(pfx(ω(a))) ≤ 1`, so the biconditional applies to `ω(a)`, and by definition of `ω`, `pfx(ω(a)) ≼ a`; the forward direction then yields `pfx(ω(a)) ≼ acct(a)`. The containment may be strict when the address occupies a sub-account position that the effective owner controls but has not delegated. Equality `pfx(ω(a)) = acct(a)` holds when no intermediate sub-account structure extends beyond the owner's prefix; this is the common case for addresses allocated directly at the principal's own account level.

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `a, b ∈ Σ.B`, `acct(a) = acct(b)`.
- *Postconditions:* `ω(a) = ω(b)`.
- *Invariant:* `pfx(ω(a)) ≼ acct(a)` for all `a ∈ Σ.B`.

Nelson: "You always know where you are, and can at once ascertain the home document of any specific word or character" (LM 2/40).

Gregory confirms: the User field in the tumbler `Node.0.User.0.Doc.0.Element` is a permanent structural component, read directly by `tumbleraccounteq`. There is no indirection, no lookup, no level of abstraction that could mask the origin.


## Subdivision Authority

Of the rights that ownership confers, one is essential to the ownership model itself: the right to create sub-positions. O5 (SubdivisionAuthority) requires that the allocator of any newly baptized address be the most-specific covering principal in `Π_Σ`. We develop here its consequences for the relation between allocation and effective ownership.

*Corollary (allocator is effective owner for non-introducing transitions).* If `Σ → Σ'` is a transition with `Π_{Σ'} = Π_Σ` (no principal introduced) and `allocated_by_{Σ'}(π, a)` for `a ∈ Σ'.B ∖ Σ.B`, then `ω_{Σ'}(a) = π`. *Proof.* O5 supplies `pfx(π) ≼ a` and `(A π' ∈ Π_Σ : pfx(π') ≼ a ⟹ #pfx(π') ≤ #pfx(π))`. By O13 (PrefixImmutability), every `π' ∈ Π_Σ` retains its prefix at `Σ'`; combined with the hypothesis `Π_{Σ'} = Π_Σ` (and O12 (PrincipalPersistence), which is consistent with it), the covering set and prefix-length data in `Π_{Σ'}` coincide with those in `Π_Σ`. Hence `π` achieves the unique longest match in `Π_{Σ'}` for `a`. By O2 (applied at `Σ'`), `ω_{Σ'}(a) = π`. ∎

Nelson: "The owner of a given item controls the allocation of the numbers under it" (LM 4/20). This is the *right to baptize* — not the baptism mechanism itself (which belongs to the tumbler baptism specification), but the authorization constraint that governs who may invoke it.

Gregory confirms: `docreatenewdocument` always uses `taskptr->account` — the session's own prefix — as the allocation hint. The allocation algorithm operates within the boundary determined by the session's account tumbler. There is no parameter that allows specifying someone else's prefix as the allocation target.


## Delegation

Ownership is not held at a single level — it flows downward through the hierarchy. Nelson calls this "baptism," but we must separate two concepts: *ownership delegation*, which introduces a new principal into `Π`, and *allocation*, which creates addresses within an existing principal's domain. The allocation mechanism is uniform at all levels (T10a); the ownership consequences differ.

We use the *strict prefix* relation throughout: `p ≺ a  ≡  p ≼ a ∧ p ≠ a` (equivalently, `p ≼ a ∧ #p < #a` — the equivalence holds because `p ≼ a ∧ #p = #a` gives `p = a` by T3).

**O7 (OwnershipDelegation).** A principal `π` may delegate a sub-prefix to a new principal `π'` along a transition `Σ → Σ'`, provided the delegation predicate `delegated_Σ(π, π')` is satisfied (which entails `zeros(pfx(π')) ≤ 1` by condition (iii)) and `π` holds subdivision authority over `pfx(π')`. Upon delegation:

  `(A Σ, Σ', π, π' : Σ → Σ' ∧ delegated_Σ(π, π') :`

  (a) `ω_{Σ'}(a) = π'` for all `a ∈ odom(π') ∩ Σ'.B`

  (b) `π'` may allocate new addresses within `odom(π')` (O5 applies to `π'`)

  (c) immediately upon entry at `Σ'`, `π'` may delegate to a new principal `π''` whose prefix is a next-reachable first child `p'' = next(Σ'.B, p, d)` of an already-baptized prefix (for some B6-valid `(p, d)`) — the only admissible delegate prefixes, by O17c — subject to obligations (i) [ancestry: `pfx(π') ≺ p''`], (iii) [structural-tier], and (v) [fresh-valid] on the choice of `p''` (conditions (ii) and (iv) being automatic given (i) and the original delegation's condition (iv))

We prove each postcondition under the hypothesis that `delegated_Σ(π, π')` holds for a transition `Σ → Σ'`, with `π ∈ Π_Σ` and `π' ∈ Π_{Σ'} ∖ Π_Σ`.

*Postcondition (a): `ω_{Σ'}(a) = π'` for all `a ∈ odom(π') ∩ Σ'.B`.*

Let `a ∈ odom(π') ∩ Σ'.B` be arbitrary. By the definition of domain, `pfx(π') ≼ a`, so `π'` covers `a`. We must show that `π'` achieves the strictly longest matching prefix among all principals in `Π_{Σ'}`.

By O15 (PrincipalClosure), at most one new principal enters `Π` per transition, and `π'` is that principal by the delegation predicate's membership clause `π' ∈ Π_{Σ'} ∖ Π_Σ`. Therefore `Π_{Σ'} = Π_Σ ∪ {π'}`. The hypothesis `delegated_Σ(π, π')` supplies conditions (i),(ii),(iv), so the StrictLongestCover newly-delegated corollary (Part 2) applies directly to `π'` and `a`: every pre-existing covering principal `π'' ∈ Π_Σ` satisfies `#pfx(π'') < #pfx(π')`, and `π'` is the unique coverer of `a` of maximal prefix length in `Π_{Σ'}`. By O2 (OwnershipExclusivity), `ω_{Σ'}(a) = π'`.

*Postcondition (b): O5 applies to `π'`.*

O5 (SubdivisionAuthority) constrains the allocator of any newly baptized address to be a most-specific covering principal; we show that an allocation within `odom(π')` is performed by `π'`. A fresh address may be baptized within `odom(π')` — for instance via `Bop(pfx(π'), 2)`, applicable because B6 (ValidDepth, ASN-0040) holds for `(pfx(π'), 2)`: `T4(pfx(π'))` by Freshness-(v), `d = 2`, and `zeros(pfx(π')) + 1 ∈ {1, 2}` by O1a, all within B6's bounds. Let `a ∈ Σ''.B ∖ Σ'.B` be such an address allocated within `odom(π')` in a successor transition `Σ' → Σ''`, so `pfx(π') ≼ a`. By O16 (AllocationClosure), `a` has some allocator `π''' ∈ Π_{Σ'}`. By O5, `pfx(π''') ≼ a` and `π'''` is a most-specific covering principal of `a` in `Π_{Σ'}`. The StrictLongestCover corollary (Part 2) is stated for *any* `a` with `pfx(π') ≼ a` and consults no `Σ'.B` membership: every pre-existing covering principal `π'' ∈ Π_Σ` satisfies `#pfx(π'') < #pfx(π')`. Our `a` satisfies `pfx(π') ≼ a`, so — together with `Π_{Σ'} = Π_Σ ∪ {π'}` (O15) — every covering principal of `a` in `Π_{Σ'}` other than `π'` has prefix strictly shorter than `pfx(π')`; hence `#pfx(π''') ≤ #pfx(π')`. Since `π'` itself covers `a`, the most-specific cover has length at least `#pfx(π')`, forcing `#pfx(π''') = #pfx(π')`. By O1b (PrefixInjectivity), at most one principal carries that prefix, so `π''' = π'`. The allocation is therefore performed by `π'` itself — `allocated_by_{Σ''}(π', a)` — establishing that `π'` holds subdivision authority over `odom(π')`.

*Postcondition (c): recursive delegation (conditional on remaining most-specific).*

Since `π' ∈ Π_{Σ'}`, the delegation relation's conditions are satisfiable with `π'` as delegator for a sub-prefix `p''` with `pfx(π') ≺ p''` *immediately upon entry* — that is, at `Σ'`. Condition (i) — `pfx(π') ≺ p''` — is a binding constraint on the choice of `p''`, not an automatic consequence: `p''` must be selected so that it strictly extends `pfx(π')`. Condition (ii) requires that `π'` be the most-specific covering principal of `p''` in `Π_{Σ'}` — equivalently, no `π'' ∈ Π_{Σ'}` with `pfx(π'') ≼ p''` has `#pfx(π'') > #pfx(π')`. We derive this directly from condition (iv) of the original delegation `delegated_Σ(π, π')`: `¬(E π'' ∈ Π_Σ : pfx(π') ≺ pfx(π''))` — no principal in `Π_Σ` has a prefix strictly extending `pfx(π')`. By O15, `Π_{Σ'} ∖ Π_Σ = {π'}`, and `pfx(π')` does not strictly extend itself, so the same non-existence carries over: no `π'' ∈ Π_{Σ'}` has `pfx(π') ≺ pfx(π'')`. Hence no covering principal of `p''` in `Π_{Σ'}` has prefix length exceeding `#pfx(π')`; `π'` achieves the maximum, satisfying condition (ii). Condition (iv) — `¬(E π'' ∈ Π_{Σ'} : p'' ≺ pfx(π''))` — holds independent of the choice of `p''`: any `π'' ∈ Π_Σ` with `p'' ≺ pfx(π'')` would, since `pfx(π') ≺ p''`, also satisfy `pfx(π') ≺ pfx(π'')`, contradicting the original delegation's (iv); and the sole new principal `π'` (by O15) does not extend `p''`, since `pfx(π') ≺ p''`. Conditions (ii) and (iv) are thus automatic at `Σ'` — but only *given* (i): the derivation of (ii) presupposes `pfx(π') ≼ p''`, which is exactly (i), and is then combined with the original delegation's condition (iv). The binding obligations on `p''` are therefore (i) [ancestry: `pfx(π') ≺ p''`], (iii) [structural-tier: `zeros(p'') ≤ 1`], and (v) [fresh-valid: `T4(p'') ∧ p'' ∉ Σ'.B`]. By O17c, the admitting transition baptizes `p''` in next-reachable form, so `p'' = next(Σ'.B, p, d)` for some B6-valid `(p, d)` — a next-reachable stream extension of an already-baptized prefix.

The authorization constraint is carried by the `delegated` relation — condition (ii) requires `π` to be the most-specific covering principal. This prevents a grandparent from delegating within a sub-domain it has already handed off: if `π₁` delegates `[1, 0, 2, 3]` to `π₂`, then `π₁` cannot subsequently delegate `[1, 0, 2, 3, 5]` to `π₃`, because `π₂` — not `π₁` — is the most-specific covering principal for that prefix.

Nelson: "Whoever owns a specific node, account, document or version may in turn designate (respectively) new nodes, accounts, documents and versions, by forking their integers" (LM 4/17). The allocation mechanism is uniform ("the entire tumbler works like that," LM 4/19), but the resulting authority is hierarchical: delegation at node and account level creates principals with full sovereignty over their domain, while allocation at document and version level exercises mechanical subdivision rights within the parent principal's domain without establishing independent ownership standing. ∎

*Formal Contract:*
- *Preconditions:* `delegated_Σ(π, π')`, `Σ → Σ'`.
- *Postconditions:* (a) `(A a ∈ odom(π') ∩ Σ'.B : ω_{Σ'}(a) = π')`; (b) `π'` satisfies O5 for allocations within `odom(π')`; (c) immediately upon entry at `Σ'`, `π'` may delegate to a new principal whose prefix is a next-reachable first child `p'' = next(Σ'.B, p, d)` of an already-baptized prefix (the form imposed by O17c), with binding obligations (i) [ancestry: `pfx(π') ≺ p''`], (iii) [structural-tier], and (v) [fresh-valid: `T4(p'') ∧ p'' ∉ Σ'.B`]; conditions (ii) and (iv) are then automatic *given* (i) together with the original delegation's condition (iv).
- *Invariant:* Delegation confers full sovereignty — the delegate becomes the effective owner of its entire domain immediately upon delegation, and acquires the rights to allocate and sub-delegate within that domain.

The delegation is irrevocable:

**O8 (IrrevocableDelegation).** Once principal `π` delegates to `π'`, the delegating parent never regains effective ownership of addresses in the delegate's domain:

  `(A π, π', a, Σ_d, Σ_d^{post}, Σ' : Σ_d reachable from Σ₀ ∧ delegated(Σ_d, Σ_d^{post}, π, π') ∧ Σ_d^{post} →* Σ' ∧ π' ∈ Π_{Σ'} ∧ a ∈ odom(π') ∩ Σ'.B : ω_{Σ'}(a) ≠ π)`

The full trajectory is `Σ_d → Σ_d^{post} →* Σ'`.

The domain restriction `odom(π') ∩ Σ'.B` ensures `ω` is applied only to addresses where it is defined (grounded by O4).

We prove that in every state `Σ'` reachable from the delegation state, the delegating parent `π` is never the effective owner of any address in the delegate's domain. The argument is direct: we show that the longest-match computation in `Σ'` always finds a principal with a strictly longer prefix than `π`, so `π` cannot be `ω_{Σ'}(a)`.

Let `Σ_d` denote the state in which `delegated(Σ_d, Σ_d^{post}, π, π')` holds along the introducing edge `Σ_d → Σ_d^{post}` (with `Σ_d` reachable from `Σ₀` by hypothesis), and let `Σ'` be any state with `Σ_d^{post} →* Σ'` (`Σ'` is then also reachable from `Σ₀`, by composing the witnessing sequence to `Σ_d^{post}` with the transitions to `Σ'`). Let `a ∈ odom(π') ∩ Σ'.B` be arbitrary.

*The delegate persists with an unchanged prefix.* The precondition fixes the introducing transition `Σ_d → Σ_d^{post}` (at which `π'` enters `Π`) together with `Σ_d^{post} →* Σ'`. By O13 (PrefixImmutability) iterated along `Σ_d^{post} →* Σ'`, `pfx_{Σ'}(π') = pfx_{Σ_d^{post}}(π')`. The delegate is present at `Σ'` with the prefix it received at the delegation transition.

*The delegate covers the address.* The precondition `a ∈ odom(π') = {t : pfx(π') ≼ t}` gives `pfx(π') ≼ a` directly; by O13 (PrefixImmutability) `pfx_{Σ'}(π') = pfx(π')`, so `pfx_{Σ'}(π') ≼ a` holds in `Σ'`.

*The delegate's prefix is strictly longer than the parent's.* By condition (i) of the delegation relation, `pfx_{Σ_d}(π) ≺ pfx_{Σ_d^{post}}(π')` — the delegator's prefix at the delegation transition's source strictly extends to the delegate's prefix at the transition's target — which gives `#pfx_{Σ_d}(π) < #pfx_{Σ_d^{post}}(π')`. By O13 iterated along `Σ_d →^* Σ'` (and using O12 to carry `π` from `Π_{Σ_d}` into `Π_{Σ'}`), both prefixes are immutable: `pfx_{Σ'}(π) = pfx_{Σ_d}(π)` and `pfx_{Σ'}(π') = pfx_{Σ_d^{post}}(π')`. The strict length inequality `#pfx_{Σ'}(π) < #pfx_{Σ'}(π')` holds at every `Σ'` with `π' ∈ Π_{Σ'}`.

*The parent cannot be the longest match.* The effective owner `ω_{Σ'}(a)` is defined (O2) as the principal in `Π_{Σ'}` with the longest matching prefix for `a`. Suppose for contradiction that `ω_{Σ'}(a) = π`. Then by the definition of `ω`, `π` would need to satisfy `(A π'' ∈ Π_{Σ'} : π'' ≠ π ∧ pfx_{Σ'}(π'') ≼ a ⟹ #pfx_{Σ'}(π) > #pfx_{Σ'}(π''))`. But `π' ∈ Π_{Σ'}` with `π' ≠ π` (they are distinct — `π` was already in `Π` before delegation while `π'` was newly introduced, and their prefixes differ in length) and `pfx_{Σ'}(π') ≼ a`, yet `#pfx_{Σ'}(π) < #pfx_{Σ'}(π')` — contradicting the requirement. Therefore `ω_{Σ'}(a) ≠ π`. ∎

*Design confirmation.* The implementation provides no revocation path — no revocation command, no forced reclamation.

*Formal Contract:*
- *Preconditions:* `Σ_d` reachable from `Σ₀`, `delegated(Σ_d, Σ_d^{post}, π, π')` (with `Σ_d → Σ_d^{post}` the introducing edge), `Σ_d^{post} →* Σ'`, `π' ∈ Π_{Σ'}`, `a ∈ odom(π') ∩ Σ'.B`.
- *Postconditions:* `ω_{Σ'}(a) ≠ π`.
- *Invariant:* Once delegation occurs, the parent's prefix is permanently shorter than the delegate's, so the parent can never regain longest-match status for any address in the delegate's domain.


## Node-Locality

Ownership authority does not propagate across node boundaries. A principal's effective ownership is bounded by its node prefix.

**O9 (NodeLocalOwnership).** In any state `Σ` reachable from `Σ₀`, for a principal `π`, the ownership predicate `owns(π, a)` can hold only for allocated addresses `a` whose node field extends the principal's node field:

  `(A Σ reachable from Σ₀, π ∈ Π_Σ, a ∈ Σ.B : owns(π, a)  ⟹  N(pfx(π)) ≼ N(a))`

We must show that if `owns(π, a)` holds for an allocated address `a` in a reachable `Σ`, then `N(pfx(π)) ≼ N(a)` — the principal's node field is a prefix of the address's node field. By O1 (PrefixDetermination), `owns(π, a) ≡ pfx(π) ≼ a`, so the hypothesis gives `pfx(π) ≼ a`: by the Prefix (PrefixRelation) definition of ASN-0034, the components of `pfx(π)` match the leading components of `a`, that is, `#a ≥ #pfx(π)` and `aᵢ = pfx(π)ᵢ` for all `1 ≤ i ≤ #pfx(π)`. The field-extraction steps below apply T4b/T4c to `a`; their precondition `T4(a)` holds by O17, since `a ∈ Σ.B` by hypothesis. By O1a (AccountOwnershipBoundary), a derived reachable-state invariant, `zeros(pfx(π)) ≤ 1`. Two cases exhaust the possibilities.

*Case 1: `zeros(pfx(π)) = 0` (node-level principal).* Every component of `pfx(π)` is strictly positive — T4's positive-component constraint requires that every non-separator component be positive, and the absence of zeros means every component is a non-separator. By T4c (LevelDetermination), a tumbler with no zeros is a node-level address, and by T4b (UniqueParse) its node field is the tumbler itself: `N(pfx(π)) = pfx(π)`, with `#N(pfx(π)) = #pfx(π)`.

Since `pfx(π) ≼ a`, the first `#pfx(π)` components of `a` match those of `pfx(π)` and are therefore all strictly positive. By T4b (UniqueParse), the node field `N(a)` consists of the components of `a` preceding the first zero-valued component (or all components of `a` if no zero occurs). Since positions `1` through `#pfx(π)` of `a` are all positive, the first zero of `a` — if it exists — occurs at position `#pfx(π) + 1` or later. Therefore `#N(a) ≥ #pfx(π) = #N(pfx(π))`. The first `#N(pfx(π))` components of `N(a)` are `a₁, ..., a_{#pfx(π)}`, which equal `pfx(π)₁, ..., pfx(π)_{#pfx(π)}` by the prefix relation, and these are exactly the components of `N(pfx(π))`. Hence `N(pfx(π)) ≼ N(a)`.

Note that the inequality may be strict: TA5(d) permits `inc([1, 2], 1) = [1, 2, 1]` with `zeros = 0`, so addresses with node fields strictly extending the principal's node field exist. In such cases `N(pfx(π)) ≺ N(a)` — the address belongs to a longer node path that shares the principal's node prefix.

*Case 2: `zeros(pfx(π)) = 1` (account-level principal).* By FieldStructure applied to `pfx(π)` (which has `zeros = 1`), the prefix decomposes uniquely as `N₁. ... .Nₐ . 0 . U₁. ... .Uᵦ` with node and user segments non-empty (`α ≥ 1`, `β ≥ 1`) and all field components positive (`Nᵢ > 0`, `Uⱼ > 0`). The node field is `N(pfx(π)) = [N₁, ..., Nₐ]`, and the single separator sits at position `α + 1`.

Since `pfx(π) ≼ a`, the first `α + 1 + β` components of `a` match those of `pfx(π)`:
- Positions `1` through `α`: `aᵢ = Nᵢ > 0` for each `1 ≤ i ≤ α`.
- Position `α + 1`: `a_{α+1} = 0`, matching the zero separator of `pfx(π)`.
- Positions `α + 2` through `α + 1 + β`: `a_{α+1+j} = Uⱼ > 0` for each `1 ≤ j ≤ β`.

By FieldStructure, `N(a)` is the segment of `a` before its first separator. Positions `1` through `α` of `a` are positive and position `α + 1` is zero (matching `pfx(π)`'s separator), so FieldStructure locates `a`'s first separator at position `α + 1` and gives `N(a) = [a₁, ..., aₐ] = [N₁, ..., Nₐ] = N(pfx(π))`. The prefix relation holds with equality: `N(pfx(π)) = N(a)`, which implies `N(pfx(π)) ≼ N(a)`.

In both cases `N(pfx(π)) ≼ N(a)`. The case distinction is exhaustive by O1a. ∎

The consequence is that ownership cannot cross node boundaries. A principal at node `[1]` cannot own addresses at node `[2]`, because `[1]` is not a prefix of `[2, ...]`. The node field's leading components must match — only the *length* of the node field may differ, and only for node-level principals (Case 1 above).

The same human being would therefore hold *separate, independent* ownership roots on each node — distinct principals with distinct prefixes, distinct domains, and no structural relationship between them. Nelson's "docuverse" is a forest of independently owned trees rooted at nodes, not a single tree with a universal authority. The node operator delegates accounts within its node; those accounts have no automatic standing on any other node.

Gregory's implementation has no cross-node communication, no remote ownership lookup, and no federation of identity. The account tumbler is per-session, per-node. But the abstract property does not depend on these implementation choices — it follows from the prefix geometry of T4 and the structural ownership predicate of O1.

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ Σ.B`, `owns(π, a)`.
- *Postconditions:* `N(pfx(π)) ≼ N(a)`. When `zeros(pfx(π)) = 1`: `N(pfx(π)) = N(a)` (equality). When `zeros(pfx(π)) = 0`: `N(pfx(π)) ≼ N(a)` (proper prefix permitted).


## The Fork as Ownership Boundary

When a principal seeks to modify content it does not own, the system's response is not an error but a creative act. This is the architectural expression of the ownership boundary.

**O10 (DenialAsFork).** When principal `π` requires modification of content at address `a` but `ω(a) ≠ π`, the system provides an alternative: `π` may create a new address `a'` within `odom(π)`:

  (a) `ω(a') = π` — the new address is fully owned by the requesting principal

  (b) the original address `a` persists in the registry (`a ∈ Σ'.B`, by B0) with its effective ownership unchanged (`ω_{Σ'}(a) = ω_Σ(a) ≠ π`) — no ownership is transferred. The fork allocates a fresh `a'` and invokes no operation on `a`; any content effects are governed by the content model, which lies outside this ASN's ownership state `Σ`

  (c) `zeros(a') = zeros(pfx(π)) + 1` — the fork sits exactly one structural tier below the principal's prefix (user level when `π` is node-level, document level when `π` is account-level). Content-bearing depth (element level, `zeros = 3`) is not guaranteed by O10 itself; it requires further organizational baptisms within the prefix-subtree `{t : a' ≼ t}`, conducted under the same sovereignty.

Condition (a) entails a structural consequence: since `ω(a') = π` gives `pfx(π) ≼ a'`, and the O6 biconditional (`pfx(π) ≼ a' ≡ pfx(π) ≼ acct(a')`, holding for all principals with `zeros(pfx(π)) ≤ 1` — i.e., all principals by O1a) yields `pfx(π) ≼ acct(a')`. The address structure necessarily records the fork within the requesting principal's account domain. Condition (c)'s zero-count relation `zeros(a') = zeros(pfx(π)) + 1` is discharged by the construction below.

Nelson: "Thus users may create new published documents out of old ones indefinitely, making whatever changes seem appropriate — without damaging the originals. This is done by inclusion links" (LM 2/45). Gregory confirms the structural mechanism: `docreatenewversion`, when invoked on a document belonging to a different account, routes the allocation through `makehint(ACCOUNT, DOCUMENT, 0, wheretoputit, &hint)` — placing the fork under the requesting principal's account, not under the source document.

The forked address lives entirely within `odom(π)`. It satisfies O2 (π is its exclusive owner), O3 corollary (π's account-level ownership is permanent), O5 (π may further subdivide it), and O6 (its provenance records π as the creator). From the ownership model's perspective, the fork is a new independent address that happens to share content identity with the original.

We must establish that such an `a'` exists in every reachable state — that `π` can always find an address within `odom(π)` where it remains the effective owner. A single baptism by `π` produces such an address in every reachable state, for both `zeros(pfx(π)) = 0` and `zeros(pfx(π)) = 1`.

*Construction.* By RegistryReachability, every reachable `Σ.B` is an ASN-0040-reachable registry, so `hwm` and `next` are well-defined on it (their B1 and finiteness preconditions hold). Set `a' = next(Σ.B, pfx(π), 2)`, the single baptism in the stream `S(pfx(π), 2)`. Every element of `S(pfx(π), 2)` has the form `pfx(π).0.k` for `k ≥ 1` (ASN-0040 SiblingStream), so `pfx(π) ≼ a'` (the first `#pfx(π)` components of `a'` reproduce `pfx(π)`), hence `a' ∈ odom(π)`. The depth-2 increment opens exactly one zero separator beyond `pfx(π)` (B5) and sibling advance preserves the zero count (B5a), so `zeros(a') = zeros(pfx(π)) + 1`: the resulting `a'` is at user level (`zeros(a') = 1`) when `zeros(pfx(π)) = 0`, and at document level (`zeros(a') = 2`) when `zeros(pfx(π)) = 1`.

*B6 verification.* The single baptism invokes `Bop(pfx(π), 2)`. ASN-0040's B6 (ValidDepth) requires (i) `T4(pfx(π))`, (ii) `d ∈ {1, 2}`, and (iii) `zeros(pfx(π)) + (d − 1) ≤ 3`. With `d = 2`: by O1a, `zeros(pfx(π)) ∈ {0, 1}`, so `zeros(pfx(π)) + (d − 1) = zeros(pfx(π)) + 1 ∈ {1, 2}`, both bounded by `3`. `T4(pfx(π))` holds via O14.6 (Valid) for bootstrap principals and, for subsequently introduced principals, via Freshness-(v) (preserved by O13). B6 is satisfied; the baptism is a well-defined operation of ASN-0040.

*Non-coverage analysis.* We show `ω_{Σ'}(a') = π` by ruling out sub-delegate coverage of `a'`. Every sub-delegate `π_i` of `π` (i.e., `π_i ∈ Π_Σ` with `pfx(π) ≺ pfx(π_i)`) satisfies `zeros(pfx(π_i)) ≤ 1` by O1a. Classify by the component of `pfx(π_i)` at position `#pfx(π) + 1`:

  - *Form A (`pfx(π).x.…`):* the component at position `#pfx(π) + 1` is strictly positive — either because `pfx(π_i)` extends the node field (`zeros(pfx(π_i)) = 0`, only possible when `zeros(pfx(π)) = 0`) or because the prefix proceeds further within the same field before reaching its zero separator (`zeros(pfx(π_i)) = 1` with the separator strictly later). Coverage of `a'` would require `pfx(π_i)_{#pfx(π) + 1} = a'_{#pfx(π) + 1} = 0`, contradicting Form A's positive component. No Form A sub-delegate covers `a'`.

  - *Form B (`pfx(π).0.Y`):* the component at position `#pfx(π) + 1` is `0` — a zero separator falls immediately after `pfx(π)`. When `zeros(pfx(π)) = 1`, this would consume a second zero (the first already at `pfx(π)`'s own user-field separator), violating O1a's `zeros(pfx(π_i)) ≤ 1`; hence Form B is empty in the `zeros(pfx(π)) = 1` case, and the analysis terminates here. When `zeros(pfx(π)) = 0`, the separator is `pfx(π_i)`'s user-field separator. Since `π_i ∈ Π_Σ`, its prefix `pfx(π_i)` is T4-valid as a standing invariant on principal prefixes; T4a (SyntacticEquivalence) forbids a trailing zero, so the user-field segment after the separator is non-empty and `#pfx(π_i) ≥ #pfx(π) + 2`. By T4a, T4's positive-component constraint, and O1a (`zeros(pfx(π_i)) ≤ 1`), `pfx(π_i)` continues with strictly positive user-field components and no further zero, with first user-field component `U^{(i)}_1 = pfx(π_i)_{#pfx(π) + 2}`. Since `a'` has length exactly `#pfx(π) + 2`, any Form B sub-delegate of length `> #pfx(π) + 2` is not a prefix of `a'` by length alone. A Form B sub-delegate of length exactly `#pfx(π) + 2` (so `pfx(π_i) = pfx(π).0.U^{(i)}_1`) covers `a'` iff `U^{(i)}_1 = hwm_0 + 1`.

  Restricting attention to length-(#pfx(π) + 2) Form B sub-delegates — the only ones that can cover `a'` by length, per the length analysis above — apply PrefixBaptismCoupling: for each such `π_i ∈ Π_Σ`, the entire prefix is `pfx(π_i) = pfx(π).0.U^{(i)}_1`, and PrefixBaptismCoupling places this prefix in `Σ.B`. Hence `pfx(π).0.U^{(i)}_1 ∈ Σ.B`. By ASN-0040's definition of `S(pfx(π), 2)` as the sibling stream of depth-2 tumblers under `pfx(π)` — every element has the form `pfx(π).0.k` for some `k ≥ 1` — the tumbler `pfx(π).0.U^{(i)}_1` lies in `S(pfx(π), 2)`. Combined with its membership in `Σ.B`, we have `pfx(π).0.U^{(i)}_1 ∈ S(pfx(π), 2) ∩ Σ.B`. By B1 (ContiguousPrefix), available on the reachable registry `Σ.B` by RegistryReachability, `children(Σ.B, pfx(π), 2) = {pfx(π).0.k : 1 ≤ k ≤ hwm_0}`, so `pfx(π).0.U^{(i)}_1 ∈ children(Σ.B, pfx(π), 2)` forces `U^{(i)}_1 ≤ hwm_0`. So `U^{(i)}_1 ≠ hwm_0 + 1`, and no length-(#pfx(π) + 2) Form B sub-delegate covers `a'`. Combined with the prior exclusion of longer Form B sub-delegates by length, no Form B sub-delegate covers `a'`.

In both `zeros(pfx(π)) = 0` and `zeros(pfx(π)) = 1` cases, no sub-delegate of `π` covers `a'`. A principal `π'' ∈ Π_Σ` covering `a'` with `pfx(π) ≺ pfx(π'')` would be exactly such a sub-delegate (its prefix strictly extends `pfx(π)` and prefixes `a'`); the Form A/B analysis therefore discharges the StrictLongestCover lemma's no-strict-extension hypothesis for `χ = π` and `a'`. The general form then applies directly: every covering principal `π'' ≠ π` of `a'` satisfies `#pfx(π'') < #pfx(π)`, so `π` achieves the unique longest matching prefix in `Π_Σ` for `a'`, and `ω_{Σ'}(a') = π` (where `Σ'` is the post-baptism state; `Π_{Σ'} = Π_Σ` by O15, since baptism introduces no principals).

*Per-baptism authorization.* By O16 (AllocationClosure), the freshly baptized `a' ∈ Σ'.B ∖ Σ.B` has some allocator `π''' ∈ Π_Σ`; by O5 (SubdivisionAuthority) that allocator is a most-specific covering principal of `a'`. The non-coverage analysis established that `π` is the unique longest-match — hence most-specific — covering principal of `a'` in `Π_Σ`, and O1b (PrefixInjectivity) makes the principal of that prefix unique, so `π''' = π`. The single baptism is therefore performed by `π` itself — `allocated_by_{Σ'}(π, a')` — and with B6 verified above, the baptism is authorized.

*Trajectory closure.* The baptism does not modify `Π` (by O15) or remove any pre-existing baptized address from the registry (by B0 Irrevocability of ASN-0040: `Σ.B ⊆ Σ'.B`). The original address `a` remains in `Σ'.B` with its ownership unchanged: `ω_{Σ'}(a) = ω_Σ(a) ≠ π`. The fork postcondition is satisfied: `a' ∈ odom(π) ∩ Σ'.B ∧ ω_{Σ'}(a') = π`, with `a ∈ Σ'.B` unchanged. ∎

The parent's fork at `hwm_0 + 1` is structurally outside every sub-delegate's authority and structurally inside `π`'s.

*Formal Contract:*
- *Preconditions:* `Σ` reachable from `Σ₀`, `π ∈ Π_Σ`, `a ∈ Σ.B`, `ω(a) ≠ π`.
- *Postconditions:* `(E Σ', a' : Σ → Σ' ∧ a' ∈ odom(π) ∩ Σ'.B ∧ ω_{Σ'}(a') = π ∧ zeros(a') = zeros(pfx(π)) + 1 ∧ a ∈ Σ'.B ∧ allocated_by_{Σ'}(π, a'))`, where `a' = pfx(π).0.{hwm_0 + 1}`, `hwm_0 := hwm(Σ.B, pfx(π), 2)`, and `Σ → Σ'` is a single baptism performed by `π` alone, as recorded by the `allocated_by_{Σ'}(π, a')` conjunct.
- *Invariant:* In every reachable state, an ownership denial is escapable — a principal facing `ω(a) ≠ π` can always obtain an effectively-owned address within `odom(π)`.

## Worked Example

We verify the properties against a concrete scenario. Let principals `π_N` and `π_M` be node operators with `pfx(π_N) = [1]` (`zeros = 0`) and `pfx(π_M) = [2]` (`zeros = 0`) — two independent nodes in a multi-node system. Initially, `Π₀ = {π_N, π_M}`.

*Convention.* Subscript labels `Σ_0, Σ_1, Σ_2, …` denote trajectory milestones, not single transitions; each segment may comprise multiple `Bop` calls whose cumulative `Σ.B` is recorded at the next milestone (order-immaterial by B0 Irrevocability of ASN-0040).

We check that O14's bootstrap clauses are satisfied: `Π₀ ≠ ∅`; each `pfx` has `zeros ≤ 1` (both have `zeros = 0`); `pfx` is injective on `Π₀` (`[1] ≠ [2]`); each prefix satisfies T4 (HierarchicalParsing — single positive component, zero-count `0 ≤ 3`) and T4a (SyntacticEquivalence — no adjacent zeros, no leading or trailing zero — vacuously, since there are no zeros); the pair is non-nesting (`[1] ⋠ [2]` and `[2] ⋠ [1]`, since component 1 differs); and each principal's prefix lies in `Σ₀.B` (we assume the bootstrap state was seeded with `[1], [2] ∈ Σ₀.B`, satisfying O14.8 (Baptized); additional seeds required for downstream B1 obligations are tabulated below). `|Π₀| = 2 < ∞`. ✓

**Bootstrap seeds.** `Σ_0`'s baptismal registry is a single bootstrap snapshot whose well-formedness (B1 contiguity, B6 depth) is ASN-0040's responsibility. From the ownership perspective only two facts matter: which addresses are in `Σ_0.B`, and who covers them. Beyond `[1], [2]` (O14.8 (Baptized)), we seed four positions, all covered by `π_N` (since `[1] ≼ ·` and `[2] ⋠ ·` for each):

| Seed | Coverage in `Π_0` |
|------|-------------------|
| `[1, 0, 1]` | `π_N` |
| `[1, 0, 2, 0, 1], [1, 0, 2, 0, 2], [1, 0, 2, 0, 3]` | `π_N` |
| `a_1 = [1, 0, 2, 0, 3, 0, 1]` | `π_N` |
| `a_3 = [1, 0, 7, 0, 1, 0, 1]` | `π_N` |

The delegated prefix `[1, 0, 2]` is not in the seed registry, so it satisfies O18's freshness conjunct `[1, 0, 2] ∉ Σ₀.B` when the delegation transition baptizes it.

**State Σ₀.** `π_N` and `π_M` are the bootstrap principals. For any address `a` with node field `1`, `ω(a) = π_N` (the only matching prefix in `Π₀`); for any address `a` with node field `2`, `ω(a) = π_M`. O2 holds — each address has a single longest match. O4 holds for any address under either node.

**Delegation.** `π_N` delegates account prefix `[1, 0, 2]` to new principal `π_A`. Now `Π_{Σ₁} = {π_N, π_M, π_A}`. The milestone arrow `Σ₀ → Σ₁` here is the lone delegation transition — it bundles no other `Bop` calls — so the single-transition lemma O3 applies to it directly.

*Verifying the conditions of `delegated_{Σ₀}(π_N, π_A)`:*

- **(i)** `pfx(π_N) ≺ pfx(π_A)`: `[1] ≺ [1, 0, 2]` — the delegate's prefix strictly extends the delegator's (length 1 vs 3, components match). ✓
- **(ii)** `π_N` is the most-specific covering principal for `[1, 0, 2]` in `Π_{Σ₀}`: the candidates whose prefix covers `[1, 0, 2]` are those `π''` with `pfx(π'') ≼ [1, 0, 2]`. Of `{π_N, π_M}`, only `π_N` (with `[1] ≼ [1, 0, 2]`) covers; `π_M`'s prefix `[2]` does not. So `π_N` is the unique — and hence most-specific — covering principal. ✓
- **Membership** `π_A ∈ Π_{Σ₁} ∖ Π_{Σ₀}`: newly introduced (the binder's freshness clause). ✓
- **(iii)** `zeros(pfx(π_A)) = 1 ≤ 1`: account-level prefix. ✓
- **(iv)** `¬(E π'' ∈ Π_{Σ₀} : pfx(π_A) ≺ pfx(π''))`: the only principals are `π_N` (prefix `[1]`, shorter than `[1, 0, 2]`, cannot be strict extension) and `π_M` (prefix `[2]`, not even a covering relation). No existing principal has a prefix strictly extending `[1, 0, 2]`. ✓
- **(v)** [fresh-valid] `T4([1, 0, 2])` (single zero at position 2, flanked by positives — no adjacent zeros, no leading or trailing zero) and `[1, 0, 2] ∉ Σ₀.B`. The admitting transition takes O17b's baptism branch, which fixes `pfx(π_A) = [1, 0, 2] = next(Σ₀.B, [1], 2)` with `(p, d) = ([1], 2)` B6-valid (`[1]` satisfies T4, `d = 2 ∈ {1, 2}`, `zeros([1]) + (2 − 1) = 1 ≤ 3`): the stream `S([1], 2)` has `c₁ = inc([1], 2) = [1, 0, 1]` and `c₂ = inc([1, 0, 1], 0) = [1, 0, 2]`. The seed `[1, 0, 1] ∈ Σ₀.B` is `c₁`, so `children(Σ₀.B, [1], 2) = {[1, 0, 1]}`, `hwm = 1`, and `next = inc(c₁, 0) = [1, 0, 2] = c₂`. Freshness then follows from ASN-0040's `Bop` postcondition `next(s.B, p, d) ∉ s.B`. ✓

*Verifying O7's postconditions for `π_A`:*

- **O7(a)**: For every `a ∈ odom(π_A) ∩ Σ₁.B`, `ω_{Σ₁}(a) = π_A`. Any such `a` has `pfx(π_A) = [1, 0, 2] ≼ a`. Pre-existing covering principals from `Π_{Σ₀}`: only `π_N` (since `π_M`'s `[2]` cannot cover an address starting with `1`), and `#pfx(π_N) = 1 < 3 = #pfx(π_A)`. By O2, `ω_{Σ₁}(a) = π_A`. ✓
- **O7(b)**: `π_A` may allocate within `odom(π_A)` per O5. The most-specific covering check now ranges over `Π_{Σ₁}`; for `a` strictly extending `[1, 0, 2]`, `π_A` is the unique principal with longest matching prefix. ✓
- **O7(c)**: `π_A` may further delegate a next-reachable sub-prefix of its domain to a new principal; conditions (i)–(iv) become satisfiable with `π_A` in the role of delegator, and (v) reduces to validity and freshness of the chosen prefix. At `Σ_1` the next-reachable first child admitted by O17b's baptism branch is `[1, 0, 2, 1] = next(Σ_1.B, [1, 0, 2], 1)`; delegating a later sibling such as `[1, 0, 2, 3]` first requires baptizing its stream-predecessors so that O17b's next-reachable form is attainable for it. ✓

*Unbounded recursion of delegation (witnessing O7(c)).* The recursive right of O7(c) supports a delegation chain of arbitrary length. We witness it with a chain of account-level delegates rooted at a node principal `π_0` with `pfx(π_0) = [1]` (`zeros = 0`). *Boundary step* `π_0 → π_1`: `pfx(π_1) = [1, 0, 1]` opens the user field (appending the separator and first user-field component), with `pfx(π_0) ≺ pfx(π_1)`, `zeros = 1`, and T4 holding (single zero at position 2, flanked by positives). *Uniform inductive step* `π_k → π_{k+1}` for `k ≥ 1`: `pfx(π_{k+1}) = [1, 0, 1, …, 1]` (`k + 3` components) appends one user-field component to `pfx(π_k)`, preserving `pfx(π_k) ≺ pfx(π_{k+1})`, `zeros = 1`, and T4. The chain extends to arbitrary length while keeping `zeros = 1`, so every link satisfies condition (iii). Conditions (ii) and (iv) hold at each link because the covering set of `pfx(π_{k+1})` in the state after `π_0, …, π_k` are introduced is exactly `{π_0, …, π_k}` — by NestingByDelegation, any other principal's prefix is non-nesting with the chain, hence cannot cover `pfx(π_{k+1})` (covering-chain lemma) — whose maximal-length member is `π_k`. Condition (v) [fresh-valid] holds at each link: `T4(pfx(π_{k+1}))` (single zero, flanked by positives) and `pfx(π_{k+1}) ∉ Σ.B`. The freshness and the next-reachable form needed by O17b's baptism branch both follow because the single appended user-field component makes `pfx(π_{k+1})` the *first child* of the stream `S(pfx(π_k), 1)`: `c₁ = inc(pfx(π_k), 1) = [1, 0, 1, …, 1, 1] = pfx(π_{k+1})` (TA5(d) appends one position with value 1), and no element of that stream is baptized before this link, so `hwm(Σ.B, pfx(π_k), 1) = 0` and `next(Σ.B, pfx(π_k), 1) = inc(pfx(π_k), 1) = pfx(π_{k+1})`, with freshness from ASN-0040's `Bop` postcondition. The boundary step is identical with `(p, d) = ([1], 2)`: `next(Σ.B, [1], 2) = inc([1], 2) = [1, 0, 1] = pfx(π_1)` when `S([1], 2)` is yet unbaptized. Each `(p, d)` is B6-valid (`zeros(p) + (d − 1) = 1 ≤ 3`), so every link is a single-step stream extension admitted by O17b's baptism branch.

**State Σ₁.** The address `a₁ = [1, 0, 2, 0, 3, 0, 1]` (a document element under account `[1, 0, 2]`) is in `Σ_0.B` under `π_N`'s coverage at genesis per the bootstrap snapshot table above (only `pfx(π_N) = [1] ≼ a₁`, while `pfx(π_M) = [2] ⋠ a₁`). Following the delegation `delegated_{Σ_0}(π_N, π_A)` introducing `π_A` with `pfx(π_A) = [1, 0, 2]`, both principals' prefixes cover `a₁`: `[1] ≼ a₁` and `[1, 0, 2] ≼ a₁`. The longer match is `[1, 0, 2]`, so `ω_{Σ_1}(a₁) = π_A`. We verify:

- **O1**: `pfx(π_A) ≼ a₁` — the first three components match; `owns(π_A, a₁)` is decidable from `pfx(π_A) = [1, 0, 2]` and `a₁ = [1, 0, 2, 0, 3, 0, 1]` alone. ✓
- **O1a**: `zeros(pfx(π_A)) = 1 ≤ 1`. ✓
- **O1b**: `pfx(π_N) = [1] ≠ [1, 0, 2] = pfx(π_A)`, so injectivity holds. ✓
- **O2**: `ω(a₁) = π_A` — unique longest match. `π_N` also matches but `#[1, 0, 2] > #[1]`. ✓
- **O3 (refinement)**: In the transition `Σ₀ → Σ₁`, `ω(a₁)` changed from `π_N` to `π_A`. O3's postcondition exhibits both delegator and delegate witnesses for the change. *Delegator witness:* `π_d = π_N ∈ Π_{Σ₀}` — the existing principal whose condition (ii) authorization admits the new delegate, satisfying `delegated_{Σ₀}(π_N, π_A)` as verified in *Verifying the conditions of `delegated_{Σ₀}(π_N, π_A)`* above. *Delegate witness:* `π' = π_A ∈ Π_{Σ₁} ∖ Π_{Σ₀}` with `pfx(π_A) ≼ a₁` and `#pfx(π_A) = 3 > 1 = #pfx(π_N) = #pfx(ω_{Σ₀}(a₁))`. The pair `(π_d, π') = (π_N, π_A)` discharges O3's postcondition `(E π_d ∈ Π_Σ, π' ∈ Π_{Σ'} ∖ Π_Σ : pfx(π') ≼ a ∧ #pfx(π') > #pfx(ω_Σ(a)) ∧ delegated_Σ(π_d, π'))`. ✓
- **O4**: `pfx(π_N) ≼ a₁` provides coverage. ✓

**Allocation.** `π_A` allocates element address `a₂ = [1, 0, 2, 0, 5, 0, 1]`. This is sub-account allocation — no new principal is created. `Π` is unchanged.

- **O5**: `pfx(π_A) = [1, 0, 2] ≼ a₂` and `π_A` has the longest matching prefix — the allocator is the most-specific covering principal. ✓
- **O6**: `acct(a₂) = [1, 0, 2] = pfx(π_A)` — the account field directly names the effective owner (equality case). ✓

**Concrete witness for SelfOwnershipAtPrefix.** By PrefixBaptismCoupling, `pfx(π_A) = [1, 0, 2] ∈ Σ_1.B`. We verify SelfOwnershipAtPrefix at the concrete boundary `a₆ = pfx(π_A) = [1, 0, 2]`. The covering set is `C(a₆) = {π ∈ Π_{Σ_1} : pfx(π) ≼ [1, 0, 2]}`. Candidates: `π_N` (prefix `[1]`, `[1] ≼ [1, 0, 2]` since the first component matches and `#[1] ≤ #[1, 0, 2]`), `π_M` (prefix `[2]`, fails: `2 ≠ 1`), `π_A` (prefix `[1, 0, 2]`, `[1, 0, 2] ≼ [1, 0, 2]` reflexively — every component matches and lengths are equal). So `C(a₆) = {π_N, π_A}` with prefix lengths `1` and `3`. The longest match is `π_A`, hence `ω(a₆) = π_A`. ✓

**Sub-account namespaces (Σ₁ → Σ₂).** `π_A` baptizes two sub-account positions `[1, 0, 2, 1]` and `[1, 0, 2, 2]` as organizational namespaces — addresses entered into `Σ.B` without introducing new principals. By O5, `π_A` is the most-specific covering principal of each and is authorized as allocator; by O15, namespace baptism admits no new principal, so `Π` is unchanged. Their well-formedness (B6 depth, B1 contiguity within `S([1, 0, 2], 1)`) holds by ASN-0040; we record only the ownership-relevant outcome `Σ_2.B ⊇ {[1, 0, 2, 1], [1, 0, 2, 2], a_2}`, which also supplies ASN-0040's B1 prerequisite for baptizing `[1, 0, 2, 3]` at the later delegation.

We verify the ownership properties under one of the namespaces. Address `a₄ = [1, 0, 2, 1, 0, 1, 0, 1]` is a document element under `[1, 0, 2, 1]`:

- **O2**: Both `pfx(π_N) = [1] ≼ a₄` and `pfx(π_A) = [1, 0, 2] ≼ a₄`. Longest match: `ω(a₄) = π_A`. ✓
- **O6**: `acct(a₄) = [1, 0, 2, 1]` and `pfx(ω(a₄)) = [1, 0, 2]`. The containment `pfx(ω(a₄)) ≼ acct(a₄)` holds but equality does not — the account field extends beyond the owner's prefix because `[1, 0, 2, 1]` has not been delegated. The provenance invariant holds: any address with `acct = [1, 0, 2, 1]` has effective owner `π_A`. ✓
- **O5**: Only `π_A` may allocate within either namespace sub-account — the most-specific covering principal. ✓

O18 (DelegationBaptizes) makes each namespace baptism a permanent commitment: since `[1, 0, 2, 1], [1, 0, 2, 2] ∈ Σ_2.B`, no future delegation transition can use either as a new principal's prefix — delegated prefixes must be drawn from `Σ'.B ∖ Σ.B`, but these slots are now occupied. By NamespacePrincipalExclusivity, provenance under `acct = [1, 0, 2, 1]` (and likewise `[1, 0, 2, 2]`) remains `π_A` forever. The next available slot in the stream, `[1, 0, 2, 3] = c_3`, is correspondingly free for either continuation; the running trajectory takes the *delegation* branch, described next.

**Sub-delegation (Σ₂ → Σ₃).** `π_A` delegates account sub-prefix `[1, 0, 2, 3]` to a new principal `π_B`, yielding `Π_{Σ₃} = {π_N, π_M, π_A, π_B}` with `pfx(π_B) = [1, 0, 2, 3]`. Condition (v) is dischargeable because the stream-predecessors `[1, 0, 2, 1], [1, 0, 2, 2] ∈ Σ_2.B` make `[1, 0, 2, 3] = next(Σ_2.B, [1, 0, 2], 1)`; O18's freshness conjunct holds since `[1, 0, 2, 3] ∉ Σ_2.B`, and `pfx(π_B)` enters `Σ_3.B` by O18. The delegator `π_A` is the allocator of that prefix — `allocated_by_{Σ_3}(π_A, [1, 0, 2, 3])` — since O16 gives the fresh prefix some allocator in `Π_{Σ_2}`, O5 and delegation condition (ii) both identify that allocator as the most-specific cover of `[1, 0, 2, 3]`, and O1b makes it unique: `π_A`. Along this path no element of the depth-2 stream `S([1, 0, 2, 3], 2)` is yet baptized — the prior baptisms anchor under different parents — so `hwm(Σ_3.B, [1, 0, 2, 3], 2) = 0`.

**Account-level permanence — verifying O8 (Irrevocability) for `π_N` over `a₁`.** The delegation `delegated_{Σ₀}(π_N, π_A)` introduces `π_A` in state Σ₁ with `pfx(π_A) = [1, 0, 2]`, and `a₁ = [1, 0, 2, 0, 3, 0, 1] ∈ odom(π_A) ∩ Σ₁.B`. O8's postcondition requires `ω_{Σ'}(a₁) ≠ π_N` for every `Σ'` with `Σ₀ →⁺ Σ'`. We trace three successor states:
- *Σ₁ (immediately post-delegation):* the covering principals for `a₁` in `Π_{Σ₁} = {π_N, π_M, π_A}` are `π_N` (prefix `[1]`, length 1) and `π_A` (prefix `[1, 0, 2]`, length 3); `π_M`'s `[2]` does not cover. Longest match: `ω_{Σ₁}(a₁) = π_A ≠ π_N`. ✓
- *Σ₂ (after `π_A` allocates `a₂`):* allocation does not change `Π` or any prefix. The covering set and longest match are unchanged: `ω_{Σ₂}(a₁) = π_A ≠ π_N`. ✓
- *Σ₃ (the sub-delegation of `[1, 0, 2, 3]` to `π_B`):* `Π_{Σ₃} = {π_N, π_M, π_A, π_B}`. The address `a₁ = [1, 0, 2, 0, 3, 0, 1]` has fourth component `0`, but `pfx(π_B) = [1, 0, 2, 3]` has fourth component `3`, so `pfx(π_B) ⋠ a₁`; `π_B` does not cover. The longest match for `a₁` remains `π_A`. `ω_{Σ₃}(a₁) = π_A ≠ π_N` — sub-delegation moves `a₁` only to principals strictly longer than `π_A`, never back to `π_N`. ✓

*Verifying O9 (Node-locality) across nodes.* Consider address `a₅ = [2, 0, 1, 0, 1, 0, 1]` — node `[2]`, user `[1]`, document `[1]`, element `[1]`. We check O9 (`owns(π, a) ⟹ N(pfx(π)) ≼ N(a)`) for each principal in `Π_{Σ₁}` and confirm consistency with the longest-match outcome:
- `π_M` (`pfx(π_M) = [2]`, `N(pfx(π_M)) = [2]`): `pfx(π_M) ≼ a₅` (first component `2 = 2`), so `owns(π_M, a₅)` holds. `N(a₅) = [2]` and `N(pfx(π_M)) = [2] ≼ [2]`. ✓
- `π_N` (`pfx(π_N) = [1]`, `N(pfx(π_N)) = [1]`): the prefix condition `pfx(π_N) ≼ a₅` requires `(a₅)₁ = 1`, but `(a₅)₁ = 2`. So `owns(π_N, a₅)` is false. O9 is vacuously satisfied. The structural barrier is in the node field itself — no account-level principal under node `[1]` can ever own an address under node `[2]`.
- `π_A` (`pfx(π_A) = [1, 0, 2]`, `N(pfx(π_A)) = [1]`): `pfx(π_A) ≼ a₅` requires `(a₅)₁ = 1`, false. `owns(π_A, a₅)` is false; O9 vacuous.

Longest match: only `π_M` covers `a₅`. `ω(a₅) = π_M`. The node operator for node `[2]` exclusively governs all addresses under that node — `π_N` and `π_A`, both rooted at node `[1]`, are structurally barred from owning any address whose node field is `[2]`. This is O9 in operation: ownership authority cannot cross the node boundary because the first field of any address syntactically anchors which node-rooted principals can cover it.

Now consider a sub-delegation under `π_M`: suppose `π_M` later delegates account prefix `[2, 0, 1]` to `π_C`. Address `a₅` has account field `[2, 0, 1]`; after this delegation, `ω(a₅) = π_C` (longer match). For O9: `N(pfx(π_C)) = [2] ≼ N(a₅) = [2]`. ✓ A principal under node `[2]` may govern addresses under node `[2]`, but the node-boundary remains rigid — no chain of delegations originating from `π_N` (node `[1]`) can ever introduce a principal whose prefix crosses into node `[2]`, because delegation condition (i) requires `pfx(π) ≺ pfx(π')`, which preserves the first component.

Now consider address `a₃ = [1, 0, 7, 0, 1, 0, 1]` under a different account. `pfx(π_A) = [1, 0, 2] ⋠ a₃` (component 3: `2 ≠ 7`). Only `pfx(π_N) = [1] ≼ a₃`, so `ω(a₃) = π_N`. The node operator retains effective ownership of all addresses not covered by a delegated account.

**Fork (O10).** Suppose `π_A` wishes to modify the content at `a₃ = [1, 0, 7, 0, 1, 0, 1]`. Since `ω(a₃) = π_N ≠ π_A`, the system does not grant modification. Instead, `π_A` creates a fork: a new document-level address `a' = [1, 0, 2, 0, 6]` within `odom(π_A)`. We trace the single-baptism trajectory and verify O10's conditions.

*Trajectory.* `π_A` has `pfx(π_A) = [1, 0, 2]` with `zeros = 1`. The pre-fork state `Σ_pre := Σ_2` is reached from `Σ_0` by the *Delegation* and *Allocation* segments above; `π_A` (the most-specific covering principal of `[1, 0, 2]`, O5-authorized) additionally baptizes `[1, 0, 2, 0, 4]` and `[1, 0, 2, 0, 5]` in the document stream `S([1, 0, 2], 2)`. Each baptism's well-formedness (B6 depth, B1 contiguity) holds by ASN-0040, available on `Σ_pre.B` by RegistryReachability. The ownership-relevant outcome is the cumulative registry `Σ_2.B ⊇ Σ_0.B ∪ {[1, 0, 2], [1, 0, 2, 0, 4], [1, 0, 2, 0, 5], a_2, [1, 0, 2, 1], [1, 0, 2, 2]}`, so `children(Σ_pre.B, [1, 0, 2], 2) = {[1, 0, 2, 0, k] : 1 ≤ k ≤ 5}` and `hwm(Σ_pre.B, [1, 0, 2], 2) = 5`.

The single baptism: `b_1 = next(Σ_pre.B, [1, 0, 2], 2) = [1, 0, 2, 0, 6]` (sibling-advance branch, `hwm = 5 > 0`; well-formedness by ASN-0040 B6). This is a document-level address with `zeros = 2`. *O5 check at `Σ_pre`:* the most-specific covering principal of `[1, 0, 2, 0, 6]` in `Π_{Σ_pre} = {π_N, π_M, π_A}` is `π_A` (matches first three components; `π_N` matches only `[1]`; `π_M` does not cover); no sub-delegate of `π_A` exists, and any would have positive at position 4, where `b_1` has 0. `π_A` is O5-authorized. Result: `Σ_pre → Σ'` with `b_1 = a' ∈ Σ'.B`.

*Verifying O10's postconditions at `Σ'`:*

- **O10(a)**: `pfx(π_A) = [1, 0, 2] ≼ [1, 0, 2, 0, 6] = a'`, and `π_A` has the longest matching prefix in `Π_{Σ'}`, so `ω_{Σ'}(a') = π_A`. ✓
- **O10(a) corollary**: by (a), `pfx(π_A) = [1, 0, 2] ≼ a'`; the O6 biconditional gives `pfx(π_A) ≼ acct(a') = [1, 0, 2]`. ✓
- **O10(b)**: `a₃ ∈ Σ_pre.B ⊆ Σ'.B` (by B0 Irrevocability of ASN-0040 — the baptismal registry is monotone under `→`). `Π_{Σ'} = Π_{Σ_pre}` (baptism introduces no principals by O15). Hence `ω_{Σ'}(a₃) = ω_{Σ_pre}(a₃) = π_N` as before — ownership unchanged, none transferred. ✓

*Field-opening boundary case.* The `π_A` fork above exhibits the sibling-advance branch (`hwm_0 = 5`) on concrete addresses. The complementary field-opening branch (`hwm_0 = 0`) is witnessed by the *Sub-delegation* milestone (state `Σ_3`), where `hwm(Σ_3.B, [1, 0, 2, 3], 2) = 0`.

Should `π_B` fork against content it does not own — say `a₃ = [1, 0, 7, 0, 1, 0, 1]`, with `ω_{Σ_3}(a₃) = π_N` — the single baptism `next(Σ_3.B, [1, 0, 2, 3], 2)` takes the field-opening branch (`hwm_0 = 0`): `inc([1, 0, 2, 3], 2) = [1, 0, 2, 3, 0, 1]` (TA5(d)), a document-level address (`zeros = 2`) with `pfx(π_B) ≼ a'`. O10 gives `ω(a') = π_B`, `π_B`-authorization, and `a₃` untouched.

*Node-level fork.* The structurally distinct node-level case — `zeros(pfx(π)) = 0`, where the Form-A sub-analysis (node-field-extending sub-delegates) becomes live — is witnessed by the node operator `π_N` (`pfx(π_N) = [1]`, `zeros = 0`). Suppose `π_N` requires modification of `a₅` under node `[2]`, where `ω(a₅) = π_M ≠ π_N`; `π_N` forks within `odom(π_N)`. The namespace is `S([1], 2)`: its elements are `c_1 = inc([1], 2) = [1, 0, 1]` and `c_2 = inc([1, 0, 1], 0) = [1, 0, 2]`, and both lie in `Σ.B` (`[1, 0, 1]` seeded; `[1, 0, 2] = pfx(π_A)` entered by PrefixBaptismCoupling at the delegation). Hence `children(Σ.B, [1], 2) = {[1, 0, 1], [1, 0, 2]}` — contiguous, so `hwm_0 = 2`. The single baptism takes the sibling-advance branch: `a' = next(Σ.B, [1], 2) = inc([1, 0, 2], 0) = [1, 0, 3]`. We verify O10(c): `zeros(a') = zeros([1, 0, 3]) = 1 = zeros(pfx(π_N)) + 1` — a user-level namespace slot exactly one tier below the node field. *Ownership.* The covering principals of `[1, 0, 3]` in `Π = {π_N, π_M, π_A}` reduce to `π_N` alone (`[1] ≼ [1, 0, 3]`); `π_M`'s `[2]` does not cover, and `π_A`'s `[1, 0, 2]` fails at position 3 (`2 ≠ 3`). So `ω(a') = π_N`. *Form-A exclusion.* A node-field-extending sub-delegate of `π_N` would be Form A — strictly positive at position `#pfx(π_N) + 1 = 2` — but `a'` carries `0` at position 2, so no Form-A sub-delegate can cover it; concretely none exists in `Π` (`π_A` is Form B, with its zero separator at position 2). The lone Form-B sub-delegate `π_A = [1, 0, 2]` has first user-field component `U_1 = 2 = hwm_0`, hence `U_1 ≠ hwm_0 + 1 = 3`, so it too fails to cover `a'`. The node-level branch of the O10 construction is thereby exhibited on a concrete address.

The fork transforms the ownership boundary into a creative act: `π_A` now has a fully owned address `a'` whose ownership is entirely independent.


## Properties Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| O1 | `owns(π, a) ≡ pfx(π) ≼ a` — ownership is prefix containment; decidability postcondition: decidable from `pfx(π)` and `a` alone, without mutable state | definition (decidability via Prefix, T3) |
| O1a | `(A Σ reachable from Σ₀, π ∈ Π_Σ : zeros(pfx(π)) ≤ 1)` — ownership principals exist only at node or account level | derived invariant; base case O14.4, preserved by Delegation cond. (iii), O13, O15 |
| O1b | `pfx` is injective — distinct principals have distinct prefixes | derived invariant; base case O14.5, preserved by Delegation length contradiction, O13, O15 |
| O2 | Every allocated address has exactly one effective owner `ω(a)`, determined by longest matching prefix | from O4, O1b, Prefix, T3, Covering-chain lemma |
| Covering-chain lemma (PrefixesOfCommonAddressAreComparable) | `(A x, p, q ∈ T : p ≼ x ∧ q ≼ x ⟹ p ≼ q ∨ q ≼ p)` — any two tumbler prefixes of a common address are `≼`-comparable | from Prefix, T3 |
| SelfOwnershipAtPrefix | `(A Σ reachable, π ∈ Π_Σ : ω_Σ(pfx(π)) = π)` — every principal effectively owns its own prefix | from O1b, O2, PrefixBaptismCoupling |
| O3 | `ω(a)` changes only through a delegation act introducing a new principal with a strictly longer matching prefix; the postcondition exhibits both the delegator `π_d` and the delegate `π'` — monotonic refinement | from B0 (ASN-0040), O12, O13, O1b, O14, O15 |
| OwnershipDomainPermanence | No external delegation can alter effective ownership within `odom(π)` — changes to `ω(a)` inside a principal's domain are induced only by a delegator `π_d` with `covers_Σ*(π, π_d)` | from Delegation, O1b, O3, O14, O15, NestingByDelegation, Prefix; multi-step corollary OwnershipDomainPermanence★ additionally from B0★ (ASN-0040) |
| O4 | `(A a ∈ Σ.B : (E π ∈ Π_Σ : pfx(π) ≼ a))` — every allocated address is covered by some principal | from O14, O16, O5, O12, O13 |
| O5 | Only the principal with the longest matching prefix may allocate within its domain — subdivision authority | axiom |
| AccountPrefix | `(A a ∈ T : T4(a) ⟹ acct(a) ≼ a)` — the account field is a prefix of any valid address | from T3, T4, Prefix, AccountField |
| O6 | `(A Σ reachable, a, b ∈ Σ.B : acct(a) = acct(b) ⟹ ω(a) = ω(b))` — effective owner determined entirely by account field | from O1a, O2, O17, AccountPrefix |
| O7 | Delegation (authorized by `delegated`) confers effective ownership (O2), subdivision authority (O5), and recursive delegation (O7) | from Delegation, O2, O5, O15 |
| O8 | `Σ_d reachable ∧ delegated(Σ_d, Σ_d^{post}, π, π') ∧ Σ_d^{post} →* Σ' ∧ π' ∈ Π_{Σ'} ∧ a ∈ odom(π') ∩ Σ'.B ⟹ ω_{Σ'}(a) ≠ π` — delegating parent never regains ownership | from Delegation, O2, O12, O13, O15, B0★ (ASN-0040) |
| O9 | `(A π ∈ Π_Σ, a ∈ Σ.B : owns(π, a) ⟹ N(pfx(π)) ≼ N(a))` — ownership bounded by node field | from O1, O1a, T4, Prefix |
| O10 | Non-ownership of target yields a fork: new address `a'` in `odom(π)` with `ω(a') = π`, `zeros(a') = zeros(pfx(π)) + 1` (one structural tier below `pfx(π)`), and original `a` retained in the registry with ownership unchanged | from O1a, O1b, O6, O17b, PrefixBaptismCoupling, TA5(c), TA5(d), ASN-0040 `next`, ASN-0040 `hwm`, ASN-0040 B1/B6, B0 (ASN-0040) |
| O12 | `(A Σ, Σ' : Σ → Σ' ⟹ Π_Σ ⊆ Π_{Σ'})` — principal persistence | axiom |
| O13 | `pfx_{Σ'}(π) = pfx_Σ(π)` for all transitions — prefix immutability | axiom |
| O14 | `Π₀ ≠ ∅`, initial principals cover all initially allocated addresses, `\|Π₀\| < ∞`, `zeros ≤ 1`, `pfx` injective on `Π₀`, `T4(pfx(π))`, pairwise non-nesting, and every initial principal's prefix lies in `Σ₀.B` | axiom |
| O15 | Principals enter Π exclusively through bootstrap or delegation; `\|Π_{Σ'} ∖ Π_Σ\| ≤ 1` per transition | axiom |
| StrictLongestCover | A coverer `χ` of `a` with no strictly-longer coverer in `Π_{Σ'}` is the unique maximal-prefix coverer; newly-delegated corollary: a `π'` from `delegated_Σ(π, π')` satisfying (i),(ii),(iv) is the unique strict longest-match coverer of any `a` it covers | from covering-chain lemma, O1b, O15, delegation conditions (i),(ii),(iv) |
| NestingByDelegation | `(A Σ reachable, π₁ ≠ π₂ ∈ Π_Σ : pfx(π₁), pfx(π₂) non-nesting ∨ covers_Σ*(π₁, π₂) ∨ covers_Σ*(π₂, π₁))` — distinct principals are either non-nesting or related by a cover-chain (`R_Σ`-closure), which by the delegation-edges-are-cover-edges bridge is a chain of delegation events | from O1b, O12, O13, O14.7, O15, delegation condition (iv), covering-chain lemma, StrictLongestCover |
| O16 | `(A a ∈ Σ'.B ∖ Σ.B : (E π ∈ Π_Σ : allocated_by_{Σ'}(π, a)))` — allocation closure | axiom |
| O17 | `(A Σ reachable from Σ₀, a : a ∈ Σ.B ⟹ T4(a))` — every allocated address is a valid tumbler | derived via RegistryReachability from ASN-0040 B10 |
| O17b | Every registry-changing ownership transition is an ASN-0040 baptism: `Σ'.B = Σ.B ∨ Σ'.B = Σ.B ∪ {next(Σ.B, p, d)}` for some B6-valid `(p, d)`; principal-introducing transitions take the baptism branch, baptizing `pfx(π')` (`Σ'.B = Σ.B ∪ {pfx(π')}`) | axiom (coupling) |
| O17c | `Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ (E p, d : B6(p, d) : pfx(π') = next(Σ.B, p, d))` — every introduced principal's prefix has next-reachable form | derived by composing O17b's principal-introduction clause with its general baptism branch |
| O18 | `Σ → Σ' ∧ π' ∈ Π_{Σ'} ∖ Π_Σ ⟹ pfx(π') ∈ Σ'.B ∖ Σ.B` — delegation materially baptizes the delegate's prefix as a fresh registration | derived from O17b coupling (baptism branch) and Freshness-(v) |
| RegistryReachability | `(A Σ reachable : Σ.B is an ASN-0040-reachable registry conforming to B₀ conf.)` — every reachable registry is B₀-conformant | derived invariant; base case O14.9 (Registry), step O17b (`Bop` edge, ASN-0040-closed) |
| PrefixBaptismCoupling | `(A Σ reachable, π ∈ Π_Σ : pfx(π) ∈ Σ.B)` — every principal's prefix is itself baptized in every reachable state | from O13, O14.8, O15, O18, B0 (ASN-0040) |
| `ω_Σ(a)` | `ω_Σ : Σ.B → Π_Σ` — the state-relativized effective owner function | from O4, O1b, Prefix, T3 |
| OwnershipDomain | `{a ∈ T : pfx(π) ≼ a}` — the ownership domain of a principal | introduced |
| `acct(a)` | the node-and-user account field of `a`, with `zeros(acct(a)) ≤ 1` — case definition in AccountField (*The Account-Level Boundary*) | from T4b, T3 |
| `allocated_by_Σ(π, a)` | Primitive relation: `a` was allocated by `π` in transition producing `Σ`; mechanism out of scope, constrained by O5 and O16 | axiom |
| Delegation | `π'` introduced into `Π` by act of `π`, with `pfx(π) ≺ pfx(π')`, `π` most-specific covering principal, no existing principal extends `pfx(π')`, `zeros(pfx(π')) ≤ 1`, and `T4(pfx(π')) ∧ pfx(π') ∉ Σ.B` (condition (v), fresh-valid) — the conditions form the complete admission gate | introduced |
| `pfx(π)` | `ownershipPrefix : Principal → Tumbler` — total, codomain constrained to `T4(pfx(π))` | axiom |


## Open Questions

- If ownership transfer is permitted, what invariants must it preserve and what formal relationship must hold between the inalienable provenance recorded in an address (O6) and the effective owner (O2) once the two diverge?
- Must the system enforce that no principal can claim an ownership prefix that overlaps an existing principal's domain, and what are the invariants of this enforcement?
- What formal guarantees must the system provide about content accessibility when the effective owner ceases to exist as a principal?
- Must ownership domains be dense (every address in the domain is reachable) or can gaps exist between baptized siblings within a domain?
- What invariants must a cross-node identity federation satisfy to remain consistent with O9, if such federation is introduced?
- Must delegation events be recorded, or is the structural evidence of the address hierarchy sufficient to reconstruct the delegation history?
- What must an implementation guarantee to realize the longest-match effective-owner selection `ω` (O2) on which exclusivity, refinement, and irrevocable delegation depend, given that account-level containment decides only single-principal coverage and never arbitrates among multiple covering principals?
