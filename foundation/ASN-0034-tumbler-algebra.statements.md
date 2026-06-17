> **ASN-0034 · Tumbler Algebra** — condensed claim statements  
> [← Full note](ASN-0034-tumbler-algebra.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0034 Claim Statements

*Source: ASN-0034-tumbler-algebra.md (revised 2026-03-26) — Extracted: 2026-04-17*

## ActionPoint — ActionPoint

Defines the action point of a positive tumbler as the index of its first nonzero component. Because the tumbler is positive, at least one nonzero component exists, so the minimum is always well-defined and falls within the tumbler's length.

*Formal Contract:*
- *Preconditions:* w ∈ T, Pos(w)
- *Definition:* actionPoint(w) = min({i : 1 ≤ i ≤ #w ∧ wᵢ ≠ 0})
- *Depends:* T0 (CarrierSetDefinition) — the precondition `w ∈ T`, the length `#w`, and the component projection `wᵢ` used in the definition and postconditions all come from T0's characterisation of T as finite sequences over ℕ with length ≥ 1; the definition's `min({i : 1 ≤ i ≤ #w ∧ wᵢ ≠ 0})` step invokes T0's well-ordering of ℕ — the set is a nonempty subset of ℕ (nonempty by TA-Pos's existential, a subset of ℕ because indices `i` with `1 ≤ i ≤ #w` lie in ℕ), and well-ordering supplies the least element that `min` names, so the "minimum exists" claim is discharged from T0 rather than left as an implicit appeal to the least-element principle; additionally, the third postcondition's step `w_{actionPoint(w)} ≥ 1` follows from `w_{actionPoint(w)} ≠ 0` (by the definition of `actionPoint(w)` as the least index with a nonzero component) together with T0's discreteness axiom (no `m ∈ ℕ` with `0 < m < 1`), so the "nonzero ⇒ `≥ 1`" inference is discharged from T0's ℕ properties rather than left implicit. TA-Pos (PositiveTumbler) — supplies the predicate `Pos(w)` in the precondition and guarantees the existence of an index `i` with `wᵢ ≠ 0`, making the set `{i : 1 ≤ i ≤ #w ∧ wᵢ ≠ 0}` nonempty so that T0's well-ordering yields a least element and the `min` in the definition is well-defined.
- *Postconditions:* 1 ≤ actionPoint(w) ≤ #w; wᵢ = 0 for all i < actionPoint(w); w_{actionPoint(w)} ≥ 1

---

## AllocatedSet — AllocatedSet

Defines the concrete set of allocated addresses at any reachable system state as the union of finite prefixes drawn from each activated allocator's theoretical domain. Establishes that allocation-relevant properties proved over T10a's abstract per-allocator chains transfer without modification to the realized finite prefixes seen in any actual execution state.

*Formal Contract:*
- *Definitions:*
  - *State:* s ∈ 𝒮 is a configuration of the allocator tree — the set of activated allocators and, for each, the count nₛ(A) of sibling increments performed. Here 𝒮 denotes the state space of the allocation system, distinct from the symbol S used in TA7a/Vocabulary for positive-component ordinals.
  - *Realized domain:* domₛ(A) = {t₀, t₁, …, t_{nₛ(A)}} where tᵢ₊₁ = inc(tᵢ, 0), a finite prefix of dom(A) (T10a).
  - *Allocated set:* allocated(s) = ⋃ { domₛ(A) : A activated in s }.
  - *Initial state:* allocated(s₀) = {t₀} where t₀ is the root allocator's T4-valid base address.
  - *State transition:* s → s' is the application of one operation from Σ — the system's transition vocabulary, introduced by NoDeallocation (forward reference, stated after this section), which supplies the closure frame assumption that every reachable state transition is governed by some op ∈ Σ. Allocation-affecting transitions either advance an allocator's frontier (adding one sibling output) or spawn a child allocator (activating a new allocator whose base address enters the allocated set).
  - *Domain embedding:* For every reachable state s and every activated allocator A: (i) `domₛ(A) ⊆ dom(A)`; (ii) `domₛ(A) = {tᵢ : 0 ≤ i ≤ nₛ(A)}` is the initial segment of T10a's `inc(·, 0)` enumeration of `dom(A)`, with enumeration indices preserved; (iii) `dom(A) ⊇ ⋃ { domₛ(A) : s reachable from s₀ }` — the theoretical chain contains every realized address across reachable states, but the reverse inclusion is not asserted (it would require a liveness/progress axiom, driving `nₛ(A)` arbitrarily high for every activated A, that no axiom of this ASN furnishes). Consequently, any predicate or claim defined over `dom(A)` in terms of the `tᵢ`-indexed chain (notably `same_allocator` (T10a), `allocated_before` (T9), and T9's forward-ordering conclusion) applies unchanged to pairs `a, b ∈ domₛ(A)` — the transfer rests on facts (i) and (ii) alone, independent of (iii).
- *Depends:* T10a (AllocatorDiscipline) — governs the allocator tree structure (sibling production by inc(·, 0), child-spawning by inc(·, k') with k' ∈ {1, 2}, root's T4-valid base address, and at-most-once child-spawning constraint) and supplies the per-allocator domain definition `dom(A) = {tₙ : n ≥ 0}` of which domₛ(A) is a finite prefix; justifies the initial-segment structure of the embedding by restricting sibling production to in-order `inc(·, 0)` steps. T9 (ForwardAllocation) — supplies the enumeration-based `allocated_before` ordering and the per-allocator forward-ordering conclusion that transfers to realized allocations. NoDeallocation (NoDeallocation) [forward reference — NoDeallocation is stated after this section] — introduces the transition vocabulary Σ and its closure frame assumption (every reachable state transition is the application of some op ∈ Σ); AllocatedSet's state-transition clause, both in the prose of this section and in the Formal Contract, cites Σ to name the operations that advance an allocator's frontier or spawn a child allocator, and the asymmetry discussion in fact (iii) explicitly leans on NoDeallocation's "forbids shrinkage but not stagnation" reading of Σ's semantics.

---

## D0 — DisplacementWellDefined

Establishes that when a < b and the divergence point does not exceed a's length, the displacement b ⊖ a is a well-defined positive tumbler whose action point equals the divergence index. The round-trip a ⊕ (b ⊖ a) recovers b exactly when a is no longer than b; if a is strictly longer than b, the round-trip is guaranteed to fail.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, a < b, divergence(a, b) ≤ #a
- *Depends:* Divergence (Divergence) — the proof eliminates case (ii) to establish case (i): `k ≤ min(#a, #b)` with `aₖ ≠ bₖ`; symmetry `divergence(b, a) = divergence(a, b)` bridges the ZPD result to `k`. T1 (LexicographicOrder) — case (ii) yields `b < a` for the prefix sub-case, contradicting `a < b`; case (i) gives `aₖ < bₖ`. TA2 (WellDefinedSubtraction) — establishes `b ⊖ a ∈ T` and `#(b ⊖ a) = max(#a, #b)` from `a < b` entailing `b ≥ a`. TumblerSub (TumblerSub) — supplies the component formulas for the explicit computation of `w = b ⊖ a`; the conditional postcondition (zpd defined ⇒ `Pos(b ⊖ a)`, `actionPoint(b ⊖ a) = zpd(b, a)`) supplies positivity and action point identification. ZPD (ZPD) — the Relationship-to-Divergence identification `zpd(b, a) = divergence(b, a)` in case (i). T3 (CanonicalRepresentation) — the round-trip boundary concludes `a ⊕ w ≠ b` from `#(a ⊕ w) ≠ #b`. TA-Pos (PositiveTumbler) — defines `Pos` for the postcondition `Pos(b ⊖ a)` (derived via TumblerSub's conditional postcondition). ActionPoint (ActionPoint) — the postcondition `actionPoint(b ⊖ a) = divergence(a, b)` uses ActionPoint's definition (derived via TumblerSub's conditional postcondition and the ZPD–Divergence identification). TA0 (WellDefinedAddition) — establishes `a ⊕ (b ⊖ a) ∈ T` from `Pos(w)` and `actionPoint(w) ≤ #a`. TumblerAdd (TumblerAdd) — the result-length identity `#(a ⊕ w) = #w` drives the round-trip boundary argument.
- *Postconditions:* b ⊖ a ∈ T, Pos(b ⊖ a) (TA-Pos), actionPoint(b ⊖ a) = divergence(a, b), #(b ⊖ a) = max(#a, #b), a ⊕ (b ⊖ a) ∈ T, #a > #b → a ⊕ (b ⊖ a) ≠ b

---

## D1 — DisplacementRoundTrip

Proves the displacement round-trip identity: when a < b, the divergence point does not exceed a's length, and a is no longer than b, then a ⊕ (b ⊖ a) = b exactly. This is the affirmative half of D0's conditional — the length constraint #a ≤ #b is the precise condition that closes the round-trip.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, a < b, divergence(a, b) ≤ #a, #a ≤ #b
- *Depends:* Divergence (Divergence) — case analysis eliminates case (ii), establishing agreement aᵢ = bᵢ for i < k. T1 (LexicographicOrder) — derives aₖ < bₖ from a < b at divergence point k. ZPD (ZPD) — identifies zpd(b, a) = divergence(a, b) = k via the ZPD–Divergence relationship in case (i). TumblerSub (TumblerSub) — supplies component formulas for w = b ⊖ a and establishes w ∈ T from b ≥ a. TA-Pos (PositiveTumbler) — defines Pos(w), satisfied since wₖ = bₖ − aₖ ≥ 1. TA0 (WellDefinedAddition) — establishes a ⊕ w ∈ T from Pos(w) and actionPoint(w) = k ≤ #a. TumblerAdd (TumblerAdd) — supplies the constructive component-by-component definition and the result-length identity #(a ⊕ w) = #w. T3 (CanonicalRepresentation) — concludes a ⊕ w = b from component-wise equality and #(a ⊕ w) = #b.
- *Postconditions:* a ⊕ (b ⊖ a) = b

---

## D2 — DisplacementUnique

Proves that the displacement carrying a to b is unique: any positive tumbler w satisfying a ⊕ w = b must equal the canonical displacement b ⊖ a. The argument applies left cancellation after D1 supplies a second witness, so no two distinct displacements can produce the same target from the same source.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, w ∈ T, a < b, divergence(a, b) ≤ #a, #a ≤ #b, Pos(w), actionPoint(w) ≤ #a, a ⊕ w = b
- *Postconditions:* w = b ⊖ a

---

## Divergence — Divergence

Given two distinct tumblers `a ≠ b`, `divergence(a, b)` returns the exact index where they first differ — either the position of the first mismatched component, or `min(#a, #b) + 1` when all shared components agree but lengths differ. The function is symmetric and always defined for distinct tumblers (exhaustiveness guaranteed by T3: if neither case applied, the tumblers would be equal).

*Formal Contract:*
- *Definition:* For `a, b ∈ T` with `a ≠ b`: (i) if `∃ k ≤ min(#a, #b)` with `aₖ ≠ bₖ` and `(A i : 1 ≤ i < k : aᵢ = bᵢ)`, then `divergence(a, b) = k`; (ii) if `(A i : 1 ≤ i ≤ min(#a, #b) : aᵢ = bᵢ)` and `#a ≠ #b`, then `divergence(a, b) = min(#a, #b) + 1`. Exactly one case applies (exhaustiveness by T3: if neither case holds, `a = b`).
- *Depends:* T3 (CanonicalRepresentation) — exhaustiveness: if neither case (i) nor case (ii) applies, all shared components agree and `#a = #b`, whence T3 yields `a = b`, contradicting `a ≠ b`.
- *Postconditions:* `divergence(a, b) = divergence(b, a)` for all `a ≠ b` (symmetry: in case (i), the least `k` with `aₖ ≠ bₖ` is unchanged by swapping operands; in case (ii), `min(#a, #b) + 1` is symmetric).

---

## GlobalUniqueness — GlobalUniqueness

No two distinct allocation events — whether from the same allocator, sibling allocators, or allocators at different hierarchy depths — ever produce the same address. The proof proceeds by strong induction on allocator tree depth, with five cases ruling out every possible collision scenario. As a consequence, each address belongs to exactly one allocator's domain.

*Formal Contract:*
- *Preconditions:* `a, b ∈ T` produced by distinct allocation events — where an allocation event is either root initialization or an invocation of `inc(t, k)` — within a system conforming to T10a (allocator discipline). Each address is assigned a producing allocator by its generating event: the root's base address to root by initialization; each `inc` output to a single allocator via the event-taxonomy rule (`k = 0` to the executing allocator, `k > 0` to the newly created child). The domain prefix of a non-root allocator `A`, spawned by `c₀ = inc(t, k')`, is the parent domain element `t`; every `a ∈ dom(A)` satisfies `t ≼ a` (by TA5(b), TA5(d), TA5-SigValid, and T10a.1).
- *Depends:* AllocatedSet (AllocatedSet) — supplies the producing-allocator taxonomy the proof rests on: *allocation event* as root initialization or `inc(t, k)` invocation in a reachable system state, grounding "distinct allocation events" in the Preconditions and underwriting the per-event assignment of each output to a producing allocator. T9 (ForwardAllocation) — Case 1: `allocated_before(a, b)` implies `a < b`, supplying the strict ordering from which distinctness is extracted. T1 (LexicographicOrder) — Case 1: part (a) irreflexivity converts `a < b` to `a ≠ b`. T10 (PartitionIndependence) — Case 3: applied directly once `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`, `p₁ ≼ a`, and `p₂ ≼ b` are established; T10's proof locates a divergence position, transfers it to `a` and `b`, and concludes `a ≠ b`. T10a (AllocatorDiscipline) — Case 5 and the exhaustiveness routing: the `inc(·, 0)`-only sibling restriction underwrites the uniform-length argument; the child-spawning bound `k' ∈ {1, 2}` pins down the values in the Case 5 length and zero-count computations; the per-parent uniqueness constraint (each `(t, k')` spawns at most one child) excludes `k'₁ = k'₂` in the `p₁ = p₂` routing, forcing distinct zero counts and directing the pair to Case 4. T10a.1 (Uniform sibling length) — Cases 2 and 5: every sibling shares the allocator's base length, letting the proof assign length `γ` to root outputs and `γ + k'` to any child's full sibling stream. T10a.3 (Length separation) — Case 2: a descendant at depth `d ≥ 1` produces outputs of length `≥ γ + d ≥ γ + 1`, yielding `#a ≠ #b` between root and non-root outputs. T10a.4 (T4 preservation) — every output of a conforming allocator satisfies T4, so every domain prefix is T4-valid; this supplies the precondition for TA5-SigValid (fixing `sig(cₙ) = #cₙ`) in the Case 5 extension argument and the precondition for T4a in the Case 5 length-collision argument. T3 (CanonicalRepresentation) — Cases 2, 3, 4, 5: tumbler equality requires component-by-component agreement at every position of a shared length, whence `#a ≠ #b` gives `a ≠ b` (Cases 2, 5), position-wise divergence at some `k` gives `a ≠ b` (Case 3), and equal zero sets are a necessary consequence of equality (Case 4). T4 (HierarchicalParsing) — invoked through T10a's root initialization constraint (every root output and descendant is T4-valid) and through T10a.4's extension of T4 to all domain prefixes, furnishing T4a's precondition. T4a (HierarchicalParsingRefinements) — Case 5 length-collision argument: condition (iii) of T4a (the last component of a T4-valid address is non-zero) fixes `p₂[#p₂] ≠ 0`, so when `p₂` extends `p₁` by exactly one position the extending component contributes no zero, leaving `zeros(p₂) = zeros(p₁)`. TA5 (IncrementPostconditions) — TA5(b) supplies agreement on positions `1 ≤ i ≤ #t` between `inc(t, k)` and `t` for `k > 0`, underwriting `t ≼ c₀` (extension argument) and the preservation of prefix content through child-spawning; TA5(c) supplies `#inc(t, 0) = #t` and single-position modification at `sig`, used for the uniform-length induction in Case 5; TA5(d) supplies `#inc(t, k') = #t + k'` for `k' > 0`, furnishing the length expansion across child-spawning (Cases 2, 5) and the zero-separator bookkeeping (one separator when `k' = 2`, none when `k' = 1`) that distinguishes zero counts in the exhaustiveness routing. TA5-SigValid (TA5-SigValid) — Case 5 extension argument: for T4-valid `cₙ`, `sig(cₙ) = #cₙ`, so each `inc(cₙ, 0)` modifies only the terminal position, leaving positions `1, …, #t` fixed across every sibling step. Prefix (PrefixRelation) — Cases 3 and 5 extension argument: the ≼ definition (agreement on shorter-length positions together with strict length excess) specifies non-nesting (Case 3: `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`) and proper prefixing (`t ≼ c₀` derived from TA5(b) and TA5(d)) on which T10's precondition and the case-5 analysis rest.
- *Invariant:* For every pair of addresses `a, b` arising from distinct allocation events (root initialization or `inc` invocations) in any reachable system state: `a ≠ b`.
- *Postconditions:* (1) Domain Disjointness — for distinct allocators `A₁ ≠ A₂`, `dom(A₁) ∩ dom(A₂) = ∅`. A shared address would require two distinct allocation events producing the same value, contradicting the invariant. (2) Well-defined owning allocator — each address value belongs to at most one allocator's domain (by Domain Disjointness), so the per-event producing-allocator assignment induces a unique owning allocator per address value: the unique `A` satisfying `a ∈ dom(A)`.
- *Proof structure:* Strong induction on allocator tree depth *d*. Claim `U(d)`: all pairs at depth ≤ *d* produce distinct outputs. Base (`d = 0`): sole root allocator, Case 1. Step (`d → d + 1`): Cases 1–5 are self-contained; the `p₁ = p₂` exhaustiveness routing invokes `U(d)` to establish shared parentage — identical prefix values held by distinct parents at depth ≤ *d* would be a repeated output contradicting `U(d)` — then applies T10a's per-parent uniqueness to separate spawning parameters.

---

## NAT-addcompat — NatAdditionOrderAndSuccessor

Addition on ℕ is monotone with respect to order: if `n ≥ p` then `m + n ≥ m + p` for all `m`. Additionally, every natural number is strictly less than its successor: `n < n + 1`.

*Formal Contract:*
- *Axiom:* `(A m, n, p ∈ ℕ : n ≥ p : m + n ≥ m + p)` (order-compatibility of addition); `(A n ∈ ℕ :: n < n + 1)` (strict successor inequality).

---

## NAT-closure — NatArithmeticClosureAndIdentity

ℕ is closed under successor (`n + 1 ∈ ℕ`) and binary addition (`m + n ∈ ℕ`), with `0` serving as the additive identity (`0 + n = n`).

*Formal Contract:*
- *Axiom:* `(A n ∈ ℕ :: n + 1 ∈ ℕ)` (successor closure); `(A m, n ∈ ℕ :: m + n ∈ ℕ)` (addition closure); `(A n ∈ ℕ :: 0 + n = n)` (additive identity).

---

## NAT-discrete — NatDiscreteness

No natural number lies strictly between `n` and `n + 1`; formally, `m ≤ n < m + 1` forces `n = m`. This discreteness axiom is independent of strict total order — it rules out the density that characterizes ℝ while leaving ℕ's order structure otherwise intact.

*Formal Contract:*
- *Axiom:* `(A m, n ∈ ℕ :: m ≤ n < m + 1 ⟹ n = m)` (discreteness).

---

## NAT-order — NatStrictTotalOrder

The natural numbers are strictly totally ordered by `<`: no number precedes itself, the order is transitive, and for any two naturals exactly one of less-than, equality, or greater-than holds. The non-strict companion `≤` is defined from `<` directly and inherits these guarantees.

*Formal Contract:*
- *Axiom:* `(A n ∈ ℕ :: ¬(n < n))` (irreflexivity); `(A m, n, p ∈ ℕ : m < n ∧ n < p : m < p)` (transitivity); `(A m, n ∈ ℕ :: exactly one of m < n, m = n, n < m)` (trichotomy). The non-strict relation `≤` on ℕ is defined by `m ≤ n ⟺ m < n ∨ m = n`.

---

## NAT-wellorder — NatWellOrdering

Every nonempty subset of ℕ contains a least element under `<`, making `min(S)` unconditionally well-defined for any nonempty S ⊆ ℕ. This well-ordering principle is what grounds induction and termination arguments over natural numbers.

*Formal Contract:*
- *Axiom:* `(A S : S ⊆ ℕ ∧ S ≠ ∅ : (E m ∈ S :: (A n ∈ S :: m ≤ n)))` (least-element principle).

---

## NoDeallocation — NoDeallocation

Every state transition in the system is required to leave the allocated address set at least as large as it found it — no operation may remove a previously allocated address. This formalizes the permanence guarantee: once a tumbler is allocated, it remains allocated for all time.

*Formal Contract:*
- *Signature of Σ:* Each element of Σ is a partial function `op : 𝒮 ⇀ 𝒮`. The predicate `op(s) defined` abbreviates `s ∈ dom(op)`; when it holds, `op(s) ∈ 𝒮` is the unique successor state. A state transition `s → s'` is exactly a pair `(s, op(s))` with `op ∈ Σ` and `s ∈ dom(op)` — no transition exists at `s` for operations undefined at `s`. This signature is the frame against which the axiom's `op(s) defined` clause is to be read.
- *Axiom:* `(A op ∈ Σ, s ∈ 𝒮 :: op(s) defined ⟹ allocated(s) ⊆ allocated(op(s)))`, where Σ is the system's complete (closed) transition vocabulary of partial functions on 𝒮 (per the signature above) and 𝒮 is the state space of the allocation system (the script-S form is used here to keep the state-space symbol distinct from the unrelated symbol S in TA7a/Vocabulary, which denotes positive-component ordinals). Frame assumption: Σ is closed, so every reachable state transition is governed by this constraint.

---

## OrdinalDisplacement — OrdinalDisplacement

δ(n, m) is the canonical "pure depth-m shift" tumbler — a sequence of length m that is zero everywhere except at the last position, which holds n ≥ 1. It acts at depth m and serves as the unit displacement that later shift operations are built from.

*Formal Contract:*
- *Preconditions:* n ≥ 1, m ≥ 1
- *Definition:* δ(n, m) = [0, 0, …, 0, n] of length m
- *Postconditions:* δ(n, m) ∈ T (by T0), Pos(δ(n, m)) (by TA-Pos), actionPoint(δ(n, m)) = m (by ActionPoint)

---

## OrdinalShift — OrdinalShift

Shifting a tumbler v by n increments only its last component by n, leaving all earlier components unchanged, and is computed as tumbler addition v ⊕ δ(n, #v). The result has the same length and same prefix as v, with the final component strictly increased.

*Formal Contract:*
- *Preconditions:* v ∈ T, n ≥ 1
- *Definition:* shift(v, n) = v ⊕ δ(n, m) where m = #v
- *Postconditions:* shift(v, n) ∈ T, #shift(v, n) = #v, shift(v, n)ᵢ = vᵢ for i < m, shift(v, n)ₘ = vₘ + n ≥ 1

---

## PartitionMonotonicity — PartitionMonotonicity

Within any prefix-delimited partition of the address space, all allocated addresses are totally ordered by T1 consistently with per-allocator allocation order. For sibling sub-partitions with non-nesting prefixes p₁ < p₂, every address extending p₁ precedes every address extending p₂ — the prefix hierarchy imposes a global cross-allocator ordering on top of each allocator's local order.

*Formal Contract:*
- *Preconditions:* A system conforming to T10a (allocator discipline); a partition with prefix `p ∈ T`; up to two child-spawning events from `p`, via `inc(p, k')` with `k' ∈ {1, 2}` as permitted by T10a, each establishing a child prefix whose sibling stream is produced by repeated `inc(·, 0)`.
- *Depends:* T5 (ContiguousSubtrees) — the Partition-structure paragraph cites T5 (prefix convexity) to establish that `subtree(p) = {t : p ≼ t}` is a contiguous interval under T1, so no address from outside the partition can interleave between two addresses inside it. T10a (AllocatorDiscipline) — supplies the partition structure: the parent allocator may spawn at most two children from `p` via `inc(p, k')` with `k' ∈ {1, 2}` (the at-most-once-per-`(t, k')` constraint), each child producing its sibling stream by repeated `inc(·, 0)`; T10a also supplies the temporal ordering invoked when `p` is shown to precede all its descendants, and the existence half of the *Total ordering* paragraph's reach-membership argument (every allocated descendant of `p` lies in some sibling stream `dom(cᵢ)` of one of `p`'s children). T10a.1 (Uniform sibling length) — invoked in the *Uniform length* paragraph and re-derived inductively from TA5(c) to argue that every sibling prefix in a child's stream has the same length as the child's base; also invoked in the *Total ordering* paragraph's *Uniqueness within a reach* clause to give `#u = #u'` for distinct siblings of the same stream, from which `subtree(u) ∩ subtree(u') = ∅` follows. T10a.4 (T4 preservation) — invoked in the Notation paragraph and Cross-depth-ordering paragraph to supply the T4-validity precondition of TA5-SigValid for every allocated address, so that `sig(t) = #t` at every step where the proof needs it. TA5-SigValid (SigOnValidAddresses) — invoked in the Notation paragraph to identify `sig(c) = #c` and in the Cross-depth-ordering paragraph to identify `sig(uⱼ) = #uⱼ = #p + 1` and `sig(vⱼ) = #vⱼ = #p + 2`; together with TA5(c) this pins down which positions sibling increments modify, which in turn fixes the position-`#p + 1` component of every address in `reach(c₁)` and `reach(c₂)`. TA5 (HierarchicalIncrement) — postcondition (a) is invoked in the *Distinctness* paragraph to argue that each application of `inc(·, 0)` produces a strictly greater tumbler, giving the strictly increasing sibling sequence `t₀ < t₁ < t₂ < ...`, and re-invoked in the *Total ordering* paragraph for the same monotonicity used in *Uniqueness within a reach*; postcondition (b) is invoked when arguing that child-spawning increments `inc(s, k')` with `k' > 0` preserve positions `1..#s`, so inherited components carry into the descendant base; postcondition (c) is invoked when arguing that sibling increments `inc(·, 0)` preserve length and modify only the significant position (e.g., the *Uniform length* paragraph and the position-`#p + 1` propagation argument); postcondition (d) is invoked when characterising the depth-1 child as `(c₁)_{#p+1} = 1` with length `#p + 1` and the depth-2 child as `(c₂)_{#p+1} = 0` with length `#p + 2`. T1 (LexicographicOrder), case (i) — the Cross-depth-ordering paragraph derives `b < a` for `a ∈ reach(c₁)`, `b ∈ reach(c₂)` from `b_{#p+1} = 0 < 1 ≤ a_{#p+1}` at divergence position `#p + 1`; case (ii) — the *Total ordering* paragraph invokes T1 case (ii) to conclude `p < a` for any allocated `a ≠ p` with `p ≼ a`, since `p ≺ a` gives `#p < #a`; the *Non-nesting* paragraph cites T1 case (ii) when arguing that a proper prefix relationship `q ≺ r` requires `#q < #r`. Prefix (PrefixRelation) — the proof unfolds `subtree(c) = {t : c ≼ t}` and the sibling-prefix relations via the Prefix definition; the derived postcondition `p ≺ q ⟹ #p < #q` is used to conclude that distinct equal-length siblings cannot stand in a prefix relation, and the same length-then-equality argument underwrites *Uniqueness within a reach*. PrefixOrderingExtension — invoked at sibling sub-partition prefixes `tᵢ < tⱼ` (non-nesting, established just above) to conclude that every address extending `tᵢ` precedes every address extending `tⱼ`. T9 (ForwardAllocation) — invoked in the *Total ordering* paragraph to secure consistency of the established T1 ordering with per-allocator allocation order within each sibling stream.
- *Postconditions:* (1) For sibling sub-partition prefixes `tᵢ < tⱼ` (with `0 ≤ i < j`) within any single child allocator's stream, and any `a, b ∈ T` with `tᵢ ≼ a` and `tⱼ ≼ b`: `a < b`. (2) Within each sub-partition with prefix `tᵢ`, for any `a, b` allocated by the same allocator: `allocated_before(a, b) ⟹ a < b`. (3) When both depth-1 and depth-2 children are spawned from `p` (with `c₁ = inc(p, 1)` and `c₂ = inc(p, 2)`), let `reach(c) = ⋃_{s ∈ dom(c)} subtree(s)` denote the *allocator reach* of a child base `c` — the union of prefix-closures over every sibling produced by `c`'s allocator. Every address in `reach(c₂)` precedes every address in `reach(c₁)`: every `a ∈ reach(c₁)` has `a_{#p+1} ≥ 1`, every `b ∈ reach(c₂)` has `b_{#p+1} = 0`, both reaches agree with `p` on positions `1..#p`, and T1 case (i) at the divergence position `#p + 1` gives `b < a`. Equivalently, for any `b` with `p ≼ b` and `b_{#p+1} = 0`, and any `a` with `p ≼ a` and `a_{#p+1} ≥ 1`: `b < a`.
- *Invariant:* For every reachable system state, the set of allocated addresses within any prefix-delimited partition is totally ordered by T1 consistently with per-allocator allocation order.

---

## TA-Pos — PositiveTumbler

Defines positivity for tumblers: a tumbler is positive if at least one component is nonzero, and a zero tumbler has all components equal to zero. Every positive tumbler is strictly greater under T1 than every zero tumbler of any length, regardless of their respective lengths.

*Formal Contract:*
- *Definition:* `Pos(t)` (positive) iff `(E i : 1 ≤ i ≤ #t : tᵢ ≠ 0)`. Zero tumbler: `(A i : 1 ≤ i ≤ #t : tᵢ = 0)`.
- *Depends:* T0 (CarrierSetDefinition) — the carrier set `T`, the length `#t`, and the component projection `tᵢ` used in the Definition (`Pos(t)` iff `(E i : 1 ≤ i ≤ #t : tᵢ ≠ 0)`) and in the zero-tumbler companion definition (`(A i : 1 ≤ i ≤ #t : tᵢ = 0)`) all come from T0's characterisation of T as finite sequences over ℕ with length ≥ 1; the postcondition proof's step that `Pos(t)` furnishes a smallest index `k` with `tₖ ≠ 0` invokes T0's well-ordering of ℕ applied to the nonempty subset `{i : 1 ≤ i ≤ #t ∧ tᵢ ≠ 0} ⊆ ℕ`, so the least-element claim is discharged from T0 rather than left implicit; additionally, the postcondition proof's step `zₖ = 0 < tₖ` because `tₖ ≥ 1` as a nonzero natural number is licensed by T0's discreteness axiom (no `m ∈ ℕ` with `0 < m < 1`), so the "nonzero ⇒ `≥ 1`" inference is discharged from T0's ℕ properties rather than left implicit; furthermore, the postcondition proof's case `#z < k` step deriving `#z + 1 ≤ #t` from `#z < k ≤ #t` is licensed by T0's discreteness axiom in its contrapositive form — for `m, n ∈ ℕ`, `m < n ⟹ m + 1 ≤ n`, since the contrary assumption `¬(m + 1 ≤ n)` gives `n < m + 1` by trichotomy, combined with `m < n` yields `m ≤ n < m + 1`, whence discreteness forces `n = m`, contradicting `m < n` by irreflexivity — instantiated at `m = #z, n = k` and then chained with `k ≤ #t` by ≤-transitivity this yields the T1 case (ii) witness `#z + 1 ≤ #t`, a use of discreteness distinct from the `zₖ = 0 < tₖ` step above (that step ruled out `0 < tₖ < 1` for a single natural; this one forces a unit gap between strictly ordered pairs), so the `m < n ⟹ m + 1 ≤ n` inference is discharged from T0's ℕ properties rather than left implicit. T1 (LexicographicOrder) — the postcondition proof invokes T1 case (i) when `#z ≥ k` to conclude `z < t` from `zₖ = 0 < tₖ`, and T1 case (ii) when `#z < k` to conclude `z < t` from `z` being a proper prefix of `t`.
- *Postconditions:* `(A t ∈ T, z ∈ T : Pos(t) ∧ (A i : 1 ≤ i ≤ #z : zᵢ = 0) :: z < t)` — every positive tumbler is strictly greater under T1 than every zero tumbler of any length.

---

## Prefix — PrefixRelation

Defines the prefix relation p ≼ q: p's length does not exceed q's and every component of p matches the corresponding component of q. A proper prefix p ≺ q additionally requires p ≠ q, which forces #p < #q strictly — equal-length agreement would make the tumblers identical by T3.

*Formal Contract:*
- *Definition:* `p ≼ q` iff `#p ≤ #q ∧ (∀i : 1 ≤ i ≤ #p : qᵢ = pᵢ)`. Proper prefix: `p ≺ q` iff `p ≼ q ∧ p ≠ q`.
- *Depends:* T0 (CarrierSetDefinition) — the definition uses length `#p`, `#q` and component projection `pᵢ`, `qᵢ` for `p, q ∈ T`, which T0 introduces; the bound `#p ≤ #q` invokes T0's non-strict ordering `≤` on ℕ to compare the lengths of the two tumblers, and the universal quantification `(∀i : 1 ≤ i ≤ #p : qᵢ = pᵢ)` invokes T0's `≤` on ℕ to range the component-projection index over the positions `1, ..., #p` of `p`; the derived postcondition `p ≺ q ⟹ #p < #q` further invokes T0's defining clause for `≤`, namely `m ≤ n ⟺ m < n ∨ m = n` — given the conjunct `#p ≤ #q` of `p ≼ q` together with `#p ≠ #q` (the alternative `#p = #q` being discharged by the contradiction with T3 cited below), the disjunction collapses to `#p < #q`, supplying the strict ℕ-inequality at the length level that the postcondition asserts; this discharges the appeals to length comparison, component-index range, and the `≤`-with-strictness step from T0 rather than leaving them implicit, matching the per-step citation convention established for `T1` and `TA5`. T3 (CanonicalRepresentation) — the derived postcondition `p ≺ q ⟹ #p < #q` invokes T3: when `#p = #q`, the prefix condition `(∀i : 1 ≤ i ≤ #p : qᵢ = pᵢ)` exhausts both tumblers' positions and T3 forces `p = q`, contradicting the strictness clause `p ≠ q` of `≺`.
- *Derived postcondition:* `p ≺ q ⟹ #p < #q` (by T3 to rule out `#p = #q`, then T0's `≤`-unfolding to convert `#p ≤ #q ∧ #p ≠ #q` into `#p < #q`).

---

## PrefixOrderingExtension — PrefixOrderingExtension

If p₁ < p₂ and neither is a prefix of the other, then every tumbler extending p₁ precedes every tumbler extending p₂ under T1. The divergence position witnessing p₁ < p₂ carries through to all extensions, so the relative order of the two prefix subtrees is fully determined by the prefixes alone.

*Formal Contract:*
- *Preconditions:* `p₁, p₂ ∈ T` with `p₁ < p₂` (T1) and `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁` (non-nesting); `a, b ∈ T` with `p₁ ≼ a` and `p₂ ≼ b`.
- *Depends:* T1 (LexicographicOrder), case (i) — supplies the divergence-position witness `k ≤ min(m, n)` with `p₁ₖ < p₂ₖ` from the hypothesis `p₁ < p₂` (after case (ii) is excluded), and is re-applied at the same `k` to derive `a < b` via `aₖ = p₁ₖ < p₂ₖ = bₖ` and component agreement `aᵢ = bᵢ` on positions `1 ≤ i < k`; case (ii) — case (ii) of T1 is excluded as the source of `p₁ < p₂` because it would give `p₁ ≼ p₂`, contradicting the non-nesting precondition. Prefix (PrefixRelation) — the preconditions `p₁ ≼ a` and `p₂ ≼ b` are unfolded via Prefix to `#a ≥ m`, `aᵢ = p₁ᵢ` for `1 ≤ i ≤ m`, and symmetrically `#b ≥ n`, `bᵢ = p₂ᵢ` for `1 ≤ i ≤ n`; these transfers carry the divergence at position `k` from the prefixes onto the extensions and supply the length bounds `k ≤ min(#a, #b)` required by T1 case (i).
- *Postconditions:* `a < b` under T1.

---

## ReverseInverse — ReverseInverse

Under the conditions that a and w share the same length k, a ≥ w, w is positive, and a has all-zero components before w's action point, subtracting w from a and then adding w back recovers a exactly. This establishes that tumbler subtraction and addition are mutually inverse within this constrained setting.

*Formal Contract:*
- *Preconditions:* `a ∈ T`, `w ∈ T`, `a ≥ w`, `Pos(w)`, `k = #a`, `#w = k`, `(A i : 1 ≤ i < k : aᵢ = 0)`, where `k` is the action point of `w`
- *Postconditions:* `(a ⊖ w) ⊕ w = a`

---

## Span — Span

A span is a pair (start address, length) that denotes a contiguous range of tumblers on the tumbler line, from the start address up to but not including the result of displacing by the length. The two validity conditions — the length must be positive and its action point must not exceed the depth of the start — are precisely what guarantee the upper bound is a legal tumbler, so any pair satisfying them yields a well-defined set of addresses.

*Formal Contract:*
- *Preconditions:* `s ∈ T`, `ℓ ∈ T`, `Pos(ℓ)`, `actionPoint(ℓ) ≤ #s`
- *Definition:* `span(s, ℓ) = {t ∈ T : s ≤ t < s ⊕ ℓ}`

---

## T0 — CarrierSetDefinition

Posits the carrier set T as the set of all finite sequences of natural numbers with at least one component, and introduces the length operator (#) and component projection (aᵢ) as primitives. This is an axiom, not a derivation — it establishes the raw material from which every other property in the system is built. The standard arithmetic facts about ℕ that proofs need are separated into their own axioms so each proof cites only what it actually uses.

*Formal Contract:*
- *Axiom:* T is the set of all finite sequences over ℕ with length ≥ 1, equipped with length `#· : T → ℕ` satisfying `#a ≥ 1` for all `a ∈ T`, and component projection `·ᵢ` yielding `aᵢ ∈ ℕ` for each `1 ≤ i ≤ #a`.

---

## T0(a) — UnboundedComponentValues

For every tumbler and every component position within it, no natural number bounds the values that can appear at that position — there always exists a tumbler of the same depth whose component at that position exceeds any given bound. This establishes that address space within any subtree is inexhaustible, and no finite quota limits allocation beneath any node.

*Formal Contract:*
- *Postcondition:* For every tumbler `t ∈ T` and every component position `i` with `1 ≤ i ≤ #t`, and for every bound `M ∈ ℕ`, there exists `t' ∈ T` with `#t' = #t` that agrees with `t` at all positions except `i`, where `t'.dᵢ > M`.

---

## T0(b) — UnboundedLength

There is no maximum tumbler length — for every natural number n, a tumbler of at least n components exists in T. Together with T0(a), this makes the address space infinite in two independent dimensions: unlimited siblings at any level, and unlimited nesting depth.

*Formal Contract:*
- *Postcondition:* For every `n ∈ ℕ` with `n ≥ 1`, there exists `t ∈ T` with `#t ≥ n`.

T0 is what separates the tumbler design from fixed-width addressing. Nelson: "New items may be continually inserted in tumbler-space while the other addresses remain valid." The word "continually" carries the weight — it means the process of creating new addresses never terminates. Between any two sibling addresses, the forking mechanism can always create children: "One digit can become several by a forking or branching process. This consists of creating successive new digits to the right." Each daughter can have daughters without limit, and each digit is itself unbounded.

The address space is unbounded in two dimensions: T0(a) ensures each component is unbounded (unlimited siblings at any level) and T0(b) ensures the number of components is unbounded (unlimited nesting depth). Together they make the address space infinite in both dimensions, which Nelson calls "finite but unlimited" — at any moment finitely many addresses exist, but there is no bound on how many can be created: "A span that contains nothing today may at a later time contain a million documents."

We observe that Gregory's implementation uses a fixed 16-digit mantissa of 32-bit unsigned integers, giving a large but finite representable range. When the allocation primitive `tumblerincrement` would exceed this range structurally (requiring a 17th digit), it detects the overflow and terminates fatally. The general addition routine `tumbleradd` silently wraps on digit-value overflow. Both behaviors violate T0. The abstract specification demands unbounded components; a correct implementation must either provide them or demonstrate that the reachable state space never exercises the bound. The comment `NPLACES 16 /* increased from 11 to support deeper version chains */` records that the original bound of 11 was concretely hit in practice — version chains deeper than 3–4 levels caused fatal crashes.

---

## T1 — LexicographicOrder

Defines a strict total order on T by lexicographic comparison: two tumblers are compared component-by-component left to right, with the first disagreement deciding the outcome, and a shorter tumbler preceding any proper extension of itself. The order is irreflexive, satisfies trichotomy, and is transitive — making any two tumblers comparable and giving the "tumbler line" its linear structure on which spans, link endsets, and content reference all depend.

*Formal Contract:*
- *Definition:* `a < b` iff `∃ k ≥ 1` with `(A i : 1 ≤ i < k : aᵢ = bᵢ)` and either (i) `k ≤ min(#a,#b) ∧ aₖ < bₖ`, or (ii) `k = #a+1 ≤ #b`.
- *Depends:* T0 (CarrierSetDefinition) — the definition uses length `#a` and component projection `aₖ` for `a ∈ T`, which T0 introduces; the proof additionally invokes the strict-total-order structure of `<` on ℕ that T0 enumerates, namely irreflexivity (part (a), to contradict `aₖ < aₖ`), trichotomy (part (b), Case 2, to resolve the ordering of disagreeing components `aₖ` and `bₖ`), and transitivity (part (c), sub-case (i,i), to compose `aₖ < bₖ` and `bₖ < cₖ` into `aₖ < cₖ`); furthermore, part (b) Trichotomy defines the *first divergence position* `k` as the least positive integer at which `a` and `b` disagree, invoking T0's well-ordering of ℕ applied to the subset `{i ∈ ℕ : 1 ≤ i ≤ min(m, n) ∧ aᵢ ≠ bᵢ} ∪ ({min(m, n) + 1} when m ≠ n, else ∅) ⊆ ℕ` — nonempty in Cases 2 and 3 by construction (Case 1 covers the empty-set situation, where `m = n` and `aᵢ = bᵢ` for all `i`, and the proof reduces to equality by T3 without invoking a divergence position) — to supply the least element that the "first divergence position" names; Case 2's rebuttal "if `k' < k`, the minimality of `k` gives `a_{k'} = b_{k'}`" and Case 3's hypothesis that `aᵢ = bᵢ` for all `1 ≤ i ≤ min(m, n)` (which holds precisely because `k = min(m, n) + 1` was chosen minimal, so no smaller position belongs to the divergence set) both rest on this minimality, so the least-element claim is discharged from T0 rather than left implicit. T3 (CanonicalRepresentation) — postcondition (b) requires the bridge from component-level agreement to tumbler equality: Case 1 invokes T3 to conclude `a = b` from `m = n` and `aᵢ = bᵢ` for all `i`; Case 2 invokes its reverse direction (tumblers that differ in any component are distinct) to conclude `a ≠ b` from `aₖ ≠ bₖ`, which is load-bearing because trichotomy's "exactly one" outcome requires ruling out `a = b` as a third concurrent option alongside the produced witness for `a < b` or `b < a`; Case 3 invokes its contrapositive to conclude `a ≠ b` from `m ≠ n`.
- *Postconditions:* (a) Irreflexivity — `(A a ∈ T :: ¬(a < a))`. (b) Trichotomy — `(A a,b ∈ T :: exactly one of a < b, a = b, b < a)`. (c) Transitivity — `(A a,b,c ∈ T : a < b ∧ b < c : a < c)`. (d) Non-strict order — `a ≤ b ⟺ a < b ∨ a = b`; `a ≥ b ⟺ b ≤ a`.

Nelson's assertion that the tumbler line is total — that two addresses are never incomparable — is architecturally load-bearing. Spans are defined as contiguous regions on the tumbler line: "A span in the tumbler line, represented by two tumblers, refers to a subtree of the entire docuverse." If two addresses were incomparable, the interval between them would be undefined, and the entire machinery of span-sets, link endsets, and content reference would collapse.

---

## T10 — PartitionIndependence

If two tumblers p₁ and p₂ are incomparable — neither is a prefix of the other — then every address beneath p₁ is distinct from every address beneath p₂, with no communication or central registry required. This is the formal basis for coordination-free allocation: the prefix hierarchy partitions address space so that independent owners of disjoint subtrees can baptize new addresses simultaneously without any risk of collision.

*Formal Contract:*
- *Preconditions:* `p₁, p₂ ∈ T` with `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁`; `a, b ∈ T` with `p₁ ≼ a` and `p₂ ≼ b`.
- *Postconditions:* `a ≠ b`.
- *Depends:* Prefix — supplies the definition of `≼` invoked at each case of the length split ("the definition of ≼ (Prefix) requires `p₂ᵢ = p₁ᵢ`…" in Case 1 and symmetrically in Case 2) and again when transferring the divergence to `a` and `b` ("Prefix gives `aᵢ = p₁ᵢ`…" and "Prefix similarly gives `bᵢ = p₂ᵢ`…"); T0 (CarrierSetDefinition) — the proof uses length `#p₁`, `#p₂`, `#a`, `#b` and component projection `p₁ᵢ`, `p₂ᵢ`, `aₖ`, `bₖ` for tumblers in T (the opening "`p₁ = p₁₁. ... .p₁ₘ`" and "`p₂ = p₂₁. ... .p₂ₙ`" name these carrier operators, and each subsequent "`aₖ = p₁ₖ`" / "`bₖ = p₂ₖ`" step indexes through them), all of which T0 introduces; the length split "Case 1: `m ≤ n`" / "Case 2: `m > n`" invokes T0's non-strict order `≤` on ℕ, since the two cases are jointly exhaustive on `m, n ∈ ℕ` because T0's defining clause `m ≤ n ⟺ m < n ∨ m = n` composed with T0's trichotomy of `<` makes `m ≤ n ∨ m > n` a law on ℕ; the identification `ℓ = m` at "since `m ≤ n`, we have `j ≤ m = ℓ`" in Case 1 (and symmetrically `ℓ = n` at "`j ≤ n = ℓ`" in Case 2) unfolds `ℓ = min(m, n)` against T0's `≤`, since `min(m, n) = m` whenever `m ≤ n` is the two-element specialisation of T0's well-ordering of ℕ applied to `{m, n}` combined with reflexivity of `≤` (`m ≤ m`, obtained from `m ≤ n ⟺ m < n ∨ m = n` with `n := m`); the least-element construction `k = min{j : 1 ≤ j ≤ ℓ ∧ p₁ⱼ ≠ p₂ⱼ}` invokes T0's well-ordering of ℕ, which makes `min(S)` well-defined for every nonempty `S ⊆ ℕ` and so licenses naming the least divergence index; the chain "`k ≤ ℓ ≤ min(m, n)`" together with the derived "`k ≤ m`" / "`k ≤ n`" steps that license the Prefix appeals `aₖ = p₁ₖ` and `bₖ = p₂ₖ` invokes T0's transitivity of `≤` on ℕ (lifted from transitivity of `<` through `m ≤ n ⟺ m < n ∨ m = n`), composed with the elementary bounds `min(m, n) ≤ m` and `min(m, n) ≤ n` supplied by the well-ordering of `{m, n}`; these per-step discharges match the citation convention established for `T1`, `TA5`, and `Prefix`. T3 (CanonicalRepresentation) — the reverse direction of T3 (tumblers that differ in any component are distinct) is the step that converts the component-level inequality `aₖ ≠ bₖ` into the tumbler-level conclusion `a ≠ b`.

The proof is elementary, but the property is architecturally profound. Nelson: "The owner of a given item controls the allocation of the numbers under it." No central allocator is needed. No coordination protocol is needed. The address structure itself makes collision impossible.

Nelson: "Whoever owns a specific node, account, document or version may in turn designate (respectively) new nodes, accounts, documents and versions, by forking their integers. We often call this the 'baptism' of new numbers." Baptism is the mechanism by which ownership domains are established — the owner of a number creates sub-numbers beneath it, and those sub-numbers belong exclusively to the owner.

---

## T10a-N — AllocatorDisciplineNecessity

T10a's restriction of the sibling stream to inc(·, 0) is not merely sufficient but necessary. Relaxing it to admit inc(·, k) with any k > 0 produces a co-sibling pair where the first output is a strict prefix of the second, directly falsifying T10a.2 (NonNestingSiblingPrefixes); since the construction is parametric in k, every relaxation witnesses a failing pair.

*Formal Contract:*
- *Preconditions:* T10a's sibling restriction is relaxed to permit `inc(·, k)` with any `k ≥ 0` in the sibling stream of a single allocator. `t₀ ∈ T`; `k > 0` is arbitrary; the allocator emits `t₁ = inc(t₀, 0)` and `t₂ = inc(t₁, k)` as co-sibling outputs under the relaxation.
- *Postconditions:* `t₁ ≼ t₂` — the two distinct co-sibling outputs stand in a prefix relation, falsifying T10a.2 (NonNestingSiblingPrefixes), which asserts that distinct sibling outputs of one allocator are prefix-incomparable (`tᵢ ⋠ tⱼ ∧ tⱼ ⋠ tᵢ`). T10a.2 is T10a's own within-allocator consequence and the vehicle by which T10a supplies T10's non-nesting precondition on co-sibling prefixes, so the falsification is internal to T10a and its downstream effect on T10 flows through T10a.2 rather than through a direct application of T10 to the pair. Since the construction is parametric in `k > 0`, the `k = 0` sibling restriction of T10a is necessary for T10a.2 to hold among co-sibling outputs of a single allocator: every `k > 0` admitted by a relaxation witnesses a failing pair.
- *Depends:* T0 (CarrierSetDefinition) — the parametric length step rests on four ℕ-level facts that T0 enumerates. First, from `k > 0` in ℕ, T0's discreteness (`m ≤ n < m + 1 ⟹ n = m`, instantiated at `m = 0`) gives `k ≥ 1`: no natural number lies strictly between `0` and `0 + 1`, so the only way `k > 0` can hold is `k ≥ 1`. Second, T0's order-compatibility of addition (`p ≤ n ⟹ m + p ≤ m + n`, applied with `m = #t₁`, `p = 1`, `n = k`) lifts this `1 ≤ k` to `#t₁ + 1 ≤ #t₁ + k` — the length-level non-strict step that feeds the transitivity chain. Third, T0's strict successor inequality `n < n + 1` (for every `n ∈ ℕ`), instantiated at `n = #t₁`, gives `#t₁ < #t₁ + 1`: the length equation supplied by TA5(d) names a ℕ-sum, and T0 is the source of the strict inequality between `n` and `n + 1` at the length level. Fourth, T0's defining clause `m ≤ n ⟺ m < n ∨ m = n` discharges two roles here: it combines `#t₁ < #t₁ + 1` with `#t₁ + 1 ≤ #t₁ + k` to produce the strict inequality `#t₁ < #t₁ + k = #t₂`, and it subsequently weakens `#t₁ < #t₂` to the non-strict `#t₁ ≤ #t₂` that Prefix's length precondition demands — the same `<`-to-`≤` step that Prefix's own Depends attributes to T0 — so the strict ℕ-inequality flows into Prefix's non-strict conjunct rather than being silently identified with it. These appeals are discharged from T0 rather than left implicit, matching the per-step citation convention established for `T1`, `TA5`, `Prefix`, and `T10` on comparable uses of length comparison, ℕ-successor, order-compatibility, discreteness, and `≤`-unfolding. T10a (AllocatorDiscipline) — supplies the `k = 0` sibling restriction whose relaxation this argument considers; without T10a the clause "relaxing the sibling restriction" has no referent. T10a.6 (DomainDisjointness) — distinct allocators have disjoint domains (`dom(X) ∩ dom(Y) = ∅`); this places the child allocator's base address `t₂ = inc(t₁, k)` (for any `k > 0`) outside the parent's domain, so under T10a the nesting `t₁ ≼ t₂` crosses the parent–child domain boundary rather than lying within the parent's sibling stream, and therefore does not offer the pair to T10's non-nesting precondition among co-sibling outputs. TA5 (HierarchicalIncrement) — TA5(d) gives the length equation `#t₂ = #t₁ + k` for arbitrary `k > 0` (the strict inequality `#t₁ < #t₂` that follows is discharged by the T0 chain cited above), and TA5(b) for `k > 0` gives agreement of `t₂` with `t₁` on positions `1..#t₁` — an agreement clause quantified over all original positions irrespective of `k`, so parametric in `k`; these are the two TA5-supplied facts that, combined with the T0-supplied discreteness, order-compatibility, strict successor, and `<`-to-`≤` steps, reconstruct the Prefix conditions on the pair `(t₁, t₂)`. Prefix (PrefixRelation) — its definition is invoked to convert "agreement on all `#t₁` positions and `#t₁ ≤ #t₂`" into `t₁ ≼ t₂`. T10a.2 (NonNestingSiblingPrefixes) — T10a's own within-allocator consequence that distinct sibling outputs of one allocator are prefix-incomparable (`tᵢ ⋠ tⱼ ∧ tⱼ ⋠ tᵢ`); the pair `(t₁, t₂)` produced by the relaxation is distinct (the strict length inequality `#t₁ < #t₂` forces `t₁ ≠ t₂`) and nesting (`t₁ ≼ t₂`), so it is exactly a counterexample to T10a.2. Citing T10a.2 here names the clause that the conclusion directly falsifies, keeping the refutation internal to T10a and matching the per-step citation convention without having to promote the co-sibling pair to T10 as candidate independent-allocator prefixes — a promotion for which this ASN supplies no contract clause. T10 (PartitionIndependence) — supplies the non-nesting precondition `p₁ ⋠ p₂ ∧ p₂ ⋠ p₁` of which T10a.2 is the co-sibling instance; T10 is cited only to situate the downstream impact of T10a.2's falsification (the partition-independence guarantee collapses on co-sibling pairs), not as the clause directly falsified by this argument.

---

## T10a.1 — UniformSiblingLength

All siblings produced by a single allocator share the same tumbler length as the base address. Because sibling production uses only inc(·, 0), which preserves length by TA5, the entire sibling stream is length-uniform.

*Formal Contract:*
- *Precondition:* Allocator with base address `t₀`, producing siblings by `inc(·, 0)`.
- *Postcondition:* `(A n ≥ 0 : #tₙ = #t₀)` — all siblings have the same length as the base address.

---

## T10a.2 — NonNestingSiblingPrefixes

Distinct outputs of a single allocator are prefix-incomparable — neither sibling is a prefix of the other. Because T10a.1 guarantees all siblings share the same length, a prefix relation between distinct siblings would require unequal lengths, yielding a contradiction; the result supplies the within-allocator non-nesting condition that T10 requires.

*Formal Contract:*
- *Precondition:* `tᵢ`, `tⱼ` are distinct siblings from the same allocator (`i ≠ j`).
- *Postcondition:* `tᵢ ⋠ tⱼ ∧ tⱼ ⋠ tᵢ` — neither is a prefix of the other.
- *Depends:* T10a.1 (UniformSiblingLength) establishes `#tᵢ = #tⱼ`; Prefix (PrefixRelation) establishes that equal-length tumblers are prefix-related only if identical. Together: distinct siblings have equal length, so a proper prefix relation would require `#tᵢ < #tⱼ` or `#tⱼ < #tᵢ`, contradicting equality.

---

## T10a.3 — LengthSeparation

Child allocator outputs are strictly longer than any output of their parent allocator, with length increasing additively at each spawning step. Along any lineage the cumulative length offset equals the sum of the spawning increments, so outputs at different nesting depths always differ in length and therefore never coincide (by T3).

*Formal Contract:*
- *Precondition:* Parent allocator with sibling length `γ`; `t` is a parent sibling (so `#t = γ` by T10a.1); child spawned via `inc(t, k')` with `k' ∈ {1, 2}` (per T10a).
- *Postcondition:* All child outputs have length `γ + k' > γ`. No child output equals any parent sibling (by T3, tumblers of different lengths are distinct). Descendant at depth `d` along a lineage with child-spawning parameters `k'₁, …, k'_d` (each `k'_i ∈ {1, 2}`) has output length exactly `γ + k'₁ + … + k'_d ≥ γ + d`; along any lineage the cumulative length is strictly increasing with depth, so outputs at different nesting depths never collide (by T3). Local monotonicity: for any ancestor allocator A at depth `d_A` and descendant allocator B at depth `d_B > d_A` on the same lineage, `#output(B) − #output(A) = k'_{d_A+1} + … + k'_{d_B} ≥ d_B − d_A ≥ 1`.

---

## T10a.4 — T4PreservationUnderDiscipline

Every address produced anywhere in an allocator tree conforming to T10a satisfies T4 (HierarchicalParsing). The root base address satisfies T4 by the initialization constraint, and TA5a (IncrementPreservesT4) propagates the invariant through every inc(·, 0) and inc(·, k') step, so T4 compliance holds at all depths by induction.

*Formal Contract:*
- *Preconditions:* Allocator tree conforming to T10a (including initialization: root base address satisfies T4).
- *Postconditions:* Every output at every depth satisfies T4.
- *Proof structure:* Induction on allocator tree depth. Base: T10a initialization constraint. Step: TA5a preservation under `inc(·, 0)` and `inc(·, k')` with `k' ∈ {1, 2}`.
- *Depends:* T10a (allocator discipline and initialization constraint), T4 (HierarchicalParsing — the invariant preserved by induction), TA5a (IncrementPreservesT4 — the preservation mechanism).

---

## T10a.5 — CrossAllocatorIncomparability

Any two allocators that are not in an ancestor-descendant relationship produce mutually prefix-incomparable outputs — no output of one can be a prefix of any output of the other. The argument traces to the lowest common ancestor: the two spawning paths diverge at sibling outputs of that ancestor, and length separation (T10a.3) together with T4-based component analysis (TA5-SigValid) shows the divergence is irreconcilable. Together with T10a.2, this delivers T10's non-nesting precondition for every allocator pair in the tree.

*Formal Contract:*
- *Precondition:* Allocators X and Y conforming to T10a, not in an ancestor-descendant relationship.
- *Postcondition:* For all x ∈ domain(X) and y ∈ domain(Y): x ⋠ y ∧ y ⋠ x.
- *Depends:* T10a (at-most-once child-spawning constraint), T10a.1 (uniform sibling length), T10a.3 (length separation across depths), T10a.4 (T4 preservation for TA5-SigValid applicability), T4 (HierarchicalParsing — TA5-SigValid precondition: `sig(t) = #t` holds only for T4-valid addresses), TA5 (postconditions (b), (c), (d)), TA5-SigValid (sig = length for T4-valid addresses), T3 (distinct same-length tumblers diverge at some position), Prefix (definition of ≼). Together with T10a.2, this delivers T10's precondition for every non-ancestor-descendant allocator pair in the tree. Ancestor-descendant pairs inherently nest (TA5(b), TA5(d)); their output distinctness follows from length separation (T10a.3).

---

## T10a.6 — DomainDisjointness

Any two distinct allocators have disjoint domains — no tumbler address can belong to more than one allocator's output stream. The proof splits into two cases: ancestor–descendant pairs are separated by strict length differences, while non-ancestor–descendant pairs are separated by prefix-incomparability. As a corollary, whenever two addresses share an allocator, that allocator is uniquely determined by the pair.

*Formal Contract:*
- *Precondition:* `X` and `Y` are distinct allocators conforming to T10a.
- *Postcondition:* `dom(X) ∩ dom(Y) = ∅`. Witness-uniqueness corollary: when `same_allocator(a, b)` holds, the witnessing allocator `A` with `a, b ∈ dom(A)` is uniquely determined by the pair `(a, b)`. Single-valuedness of the enumeration indices `i, j` with `a = tᵢ, b = tⱼ` further requires the within-allocator injectivity supplied by T10a.7.
- *Depends:* T10a.1 (UniformSiblingLength) — supplies the per-domain uniform length used in case 1; T10a.3 (LengthSeparation) — gives the strict length inequality `γ_Y > γ_X` between ancestor and descendant sibling streams, contradicting any shared element in case 1; T10a.5 (CrossAllocatorIncomparability) — gives prefix-incomparability of all outputs across non-ancestor–descendant allocators, which together with reflexivity of `≼` excludes any shared element in case 2; Prefix (PrefixRelation) — reflexivity `t ≼ t`, used to derive the contradiction in case 2.

---

## T10a.7 — EnumerationInjectivity

Within a single allocator, the sequential enumeration of its domain is injective — distinct indices always produce distinct addresses. Because each step strictly increases under the tumbler order, the full enumeration is strictly monotone, making the index of any address a single-valued function of that address. Together with T10a.6, this makes the enumeration indices of any same-allocator pair uniquely determined, which is required for T9's allocated_before predicate to be well-defined.

*Formal Contract:*
- *Precondition:* Allocator A conforming to T10a, with domain `dom(A) = {tₙ : n ≥ 0}` where `t₀` is the base address and `tₙ₊₁ = inc(tₙ, 0)`.
- *Postcondition:* The map `n ↦ tₙ` is injective: `(A m, n ≥ 0 : m ≠ n : tₘ ≠ tₙ)`. Equivalently (by T1 trichotomy, applied to the strict inequality established in the proof), `(A m, n ≥ 0 : m < n : tₘ < tₙ)` — the enumeration is strictly increasing under the tumbler order T1.
- *Depends:* T10a (AllocatorDiscipline) — supplies the enumeration `tₙ₊₁ = inc(tₙ, 0)` whose injectivity is the claim. TA5 (HierarchicalIncrement), postcondition (a) — strict monotonicity `inc(tₙ, 0) > tₙ`, providing the per-step increase at the base case and in the inductive step. T1 (LexicographicOrder), postcondition (c) transitivity — lifts the per-step increase to `tₘ < tₙ` across arbitrary gaps; postcondition (a) irreflexivity — converts the strict inequality `tₘ < tₙ` into distinctness `tₘ ≠ tₙ`.

---

## T10a — AllocatorDiscipline

Allocators must produce sibling addresses exclusively by shallow increment inc(·,0), and may spawn child allocators only via a single deep increment with k'∈{1,2} subject to zero-count bounds from TA5a. This discipline ensures all outputs have controlled prefix relationships, prevents length collisions within and across allocators, and preserves the T4 structural invariant throughout the entire allocator tree by induction.

*Formal Contract:*
- *Definitions:*
  - *Domain:* `dom(A) = {tₙ : n ≥ 0}` where `t₀` is the allocator's base address and `tₙ₊₁ = inc(tₙ, 0)` — the `inc(·, 0)` chain extending the base. Child-spawning outputs `inc(t, k')` with `k' > 0` are excluded from the parent's domain and serve as the base of the child allocator, becoming the initial element of `dom(child)`.
  - *Same allocator:* `same_allocator(a, b) ≡ ∃A : a ∈ dom(A) ∧ b ∈ dom(A)` — the predicate asserting that `a` and `b` both lie in some allocator's domain.
- *Axiom:* The root allocator's base address satisfies T4. Allocators produce sibling outputs exclusively by `inc(·, 0)`; child-spawning uses exactly one `inc(·, k')` with `k' ∈ {1, 2}`, subject to the runtime precondition — imposed by the discipline and checked before each spawn — that `zeros(t) ≤ 3` when `k' = 1` and `zeros(t) ≤ 2` when `k' = 2`, where `zeros(·)` is the zero-count function defined in TA5a. The chosen `inc(·, k')` establishes the child's prefix, after which the parent resumes sibling production with `inc(·, 0)`. Each `(t, k')` pair — domain element and spawning parameter — yields at most one child-spawning event. Depends: T4 (HierarchicalParsing — the invariant the root must satisfy and the discipline must preserve; also the original site of the `zeros(·)` definition), TA5 (IncrementPostconditions — defines `inc` and its length/positional postconditions), TA5a (IncrementPreservesT4 — supplies both the `zeros(·)` symbol the axiom names in its precondition and the preservation bounds `k' = 1 ∧ zeros(t) ≤ 3`, `k' = 2 ∧ zeros(t) ≤ 2` that the discipline adopts as runtime constraints on child-spawning parameters).
- *Postconditions:*
  - T10a.1 (Uniform sibling length): For every allocator with base address b, all sibling outputs a satisfy #a = #b. Depends: TA5 (TA5(c): #inc(t, 0) = #t).
  - T10a.2 (Non-nesting sibling prefixes): For all siblings a, b from the same allocator, same_allocator(a, b) ∧ a ≠ b → a and b are prefix-incomparable. Depends: T10a.1 (#a = #b) and Prefix (equal-length tumblers are prefix-related only if identical).
  - T10a.3 (Length separation): For every child allocator spawned by `inc(·, k')` with k' ∈ {1, 2} from a parent with base length m, all child outputs c satisfy #c = m + k', and across d nesting levels the separation is exact: #output = m + k'₁ + k'₂ + … + k'_d. Local monotonicity: for any ancestor-descendant pair (A, B) on a lineage, #output(B) > #output(A). Depends: T10a.1 (uniform sibling length) and TA5 (TA5(d): #inc(t, k') = #t + k' for k' > 0).
  - T10a.4 (T4 preservation): The root allocator's base address satisfies T4 (initialization constraint); since `inc(·, 0)` unconditionally preserves T4 (TA5a) and child-spawning uses `k' ∈ {1, 2}` within TA5a bounds, every output of a conforming allocator satisfies T4 by induction on the allocator tree. Depends: T4 (HierarchicalParsing — the invariant preserved by induction), TA5a (inc(·, 0) preserves T4 unconditionally; inc(·, k') with k' ∈ {1, 2} preserves T4 within zero-count bounds).
  - T10a.5 (Cross-allocator prefix-incomparability): For any two allocators X and Y not in an ancestor-descendant relationship, for all x ∈ domain(X) and y ∈ domain(Y), x ⋠ y ∧ y ⋠ x. Depends: T10a (at-most-once constraint), T10a.1 (uniform length), T10a.3 (length separation), T10a.4 (T4 preservation), T4 (HierarchicalParsing — TA5-SigValid precondition), TA5 (increment postconditions), TA5-SigValid (sig = length for T4-valid addresses), T3 (divergence of distinct tumblers), Prefix (prefix definition). Together with T10a.2, this delivers T10's precondition for every non-ancestor-descendant allocator pair in the tree. Ancestor-descendant pairs inherently nest (TA5(b), TA5(d)); their output distinctness follows from length separation (T10a.3).
  - T10a.6 (Domain disjointness): For any two distinct allocators X and Y, dom(X) ∩ dom(Y) = ∅. Case 1 (ancestor–descendant): T10a.1 + T10a.3 — uniform per-domain length (T10a.1) combined with the strict length inequality between ancestor and descendant streams (T10a.3) excludes any common element. Case 2 (non-ancestor–descendant): T10a.5 + Prefix reflexivity — cross-allocator prefix-incomparability (T10a.5) is contradicted by `t ≼ t` were `t` shared. Witness-uniqueness corollary: when `same_allocator(a, b)` holds, the witnessing allocator A is uniquely determined by the pair (a, b). Single-valuedness of the enumeration indices i, j with a = tᵢ, b = tⱼ requires also the within-allocator injectivity of T10a.7; the two results together underwrite well-definedness of T9's allocated_before. Depends: T10a.1 (uniform sibling length), T10a.3 (length separation), T10a.5 (cross-allocator prefix-incomparability), Prefix (reflexivity of ≼).
  - T10a.7 (Enumeration injectivity): For every allocator A with domain `dom(A) = {tₙ : n ≥ 0}` where `tₙ₊₁ = inc(tₙ, 0)`, the indexing map `n ↦ tₙ` is injective: `m ≠ n → tₘ ≠ tₙ`. Proof: TA5(a) gives per-step strict increase `tₙ₊₁ > tₙ`; induction on the gap via T1 (c) transitivity yields `tₘ < tₙ` for `m < n`; T1 (a) irreflexivity converts strict inequality into distinctness. Equivalently, the enumeration is strictly increasing under the tumbler order. Together with T10a.6, this makes the enumeration indices `i, j` with `a = tᵢ, b = tⱼ` single-valued functions of the pair `(a, b)` whenever `same_allocator(a, b)` holds, supplying the second of the two pieces required for well-definedness of T9's `allocated_before`. Depends: T10a (enumeration definition), TA5 (TA5(a): `inc(t, 0) > t`), T1 (T1 (a): irreflexivity; T1 (c): transitivity).
  - T10a-N (Necessity of sibling restriction): Under the relaxed rule (any k ≥ 0 in the sibling stream), the pair a₁ = inc(b, 0) and a₂ = inc(a₁, k') with k' > 0 are sibling outputs satisfying a₁ ≺ a₂ — by TA5(b) (agreement on all positions of a₁) and TA5(d) (#a₂ > #a₁), invoking the Prefix definition. This falsifies T10a.2 (NonNestingSiblingPrefixes) — T10a's own within-allocator consequence that distinct sibling outputs of one allocator are prefix-incomparable — and T10a.2 is the vehicle by which T10a supplies T10's non-nesting precondition on co-sibling prefixes, so the downstream effect on T10 flows through T10a.2 rather than through a direct application of T10 to the pair. The sibling restriction (`k = 0` only) is therefore necessary for prefix-incomparability. The remaining axiom components — the `k' ∈ {1, 2}` child-spawning bound, the at-most-once constraint, and the root T4 initialization — serve T4 preservation (TA5a) and child-prefix uniqueness respectively; their necessity for those distinct purposes is not addressed by this argument. Depends: TA5 (TA5(b): agreement on positions 1..#t; TA5(d): #inc(t, k') = #t + k'), Prefix (prefix definition), T10a.2 (NonNestingSiblingPrefixes — the within-allocator consequence directly falsified, matching the per-step citation convention without needing to promote the pair to T10 as candidate independent-allocator prefixes), and T10 (non-nesting precondition supplied downstream through T10a.2).

---

## T12 — SpanWellDefinedness

A span is the set of all tumblers from a start address s up to but not including the displaced endpoint s⊕ℓ, and this definition is well-formed: the endpoint exists (TA0), the span is non-empty because s is always a member (TA-strict), and the span is order-convex under T1 — any tumbler lying between two span members is itself in the span.

*Formal Contract:*
- *Preconditions:* `s ∈ T`, `ℓ ∈ T`, `Pos(ℓ)`, `actionPoint(ℓ) ≤ #s`
- *Definition:* `span(s, ℓ) = {t ∈ T : s ≤ t < s ⊕ ℓ}`
- *Postconditions:* (a) `s ⊕ ℓ ∈ T` (endpoint exists, by TA0). (b) `s ∈ span(s, ℓ)` (non-empty, by TA-strict). (c) `span(s, ℓ)` is order-convex under T1 (for all `a, c ∈ span(s, ℓ)` and `b ∈ T`, `a ≤ b ≤ c` implies `b ∈ span(s, ℓ)`).

---

## T2 — IntrinsicComparison

The tumbler order from T1 is computable from the two tumblers alone, consulting no external state — only their component sequences and lengths. The comparison terminates after examining at most min(#a,#b) component pairs, making it both intrinsic and bounded.

*Formal Contract:*
- *Preconditions:* `a, b ∈ T` — two well-formed tumblers (finite sequences over ℕ with `#a ≥ 1` and `#b ≥ 1`, per T0).
- *Depends:* T3 (CanonicalRepresentation) — postcondition (a) requires T3 for the equality case: when all `min(m, n)` component pairs agree and `m = n`, T3 bridges component-level agreement to `a = b`.
- *Postconditions:* (a) The ordering among `a` and `b` under T1 is determined. (b) At most `min(#a, #b)` component pairs are examined. (c) The only values consulted are `{aᵢ : 1 ≤ i ≤ #a}`, `{bᵢ : 1 ≤ i ≤ #b}`, `#a`, and `#b`.
- *Frame:* No external data structure is read or modified — the comparison is a pure function of the two tumblers.

---

## T3 — CanonicalRepresentation

Tumbler equality is exactly component-wise sequence equality — two tumblers are equal if and only if they have the same length and identical values at every position. No normalization, quotient, or external identification is imposed; the raw component sequences must be literally identical.

*Formal Contract:*
- *Postcondition:* Tumbler equality is sequence equality: `a = b ⟺ #a = #b ∧ (A i : 1 ≤ i ≤ #a : aᵢ = bᵢ)`. No quotient, normalization, or external identification is imposed on T.
- *Depends:* T0 (CarrierSetDefinition) — the biconditional restates the extensional definition of sequence equality for T0's carrier set T of finite sequences over ℕ; without T0's characterisation, there is no basis for identifying tumbler equality with component-wise identity.

---

## T4 — HierarchicalParsing

Valid tumblers encode a four-level containment hierarchy (node, user, document, element) using zero-valued components as field separators. The axiom constrains separator placement: at most three zeros, none adjacent, and neither the first nor last component may be zero — guaranteeing every field segment is non-empty.

*Formal Contract:*
- *Axiom:* Valid address tumblers satisfy the following positional conditions on the component sequence, none of which appeals to any prior notion of field presence: `zeros(t) ≤ 3`; `(A i : 1 ≤ i < #t : ¬(tᵢ = 0 ∧ tᵢ₊₁ = 0))`; `t₁ ≠ 0`; `t_{#t} ≠ 0` (the last three together form the field-segment constraint, guaranteeing that the maximal non-zero contiguous sub-sequences of `t` — the field segments — are all non-empty). Positivity of non-zero components is not a separate axiom clause: T0's carrier ℕ already makes `tᵢ ≠ 0 ⇔ tᵢ > 0`.
- *Preconditions:* T4b (UniqueParse) requires T3 (CanonicalRepresentation) — canonical representation fixes the component sequence of `t`, ensuring separator positions computed by scanning for zeros are uniquely determined.
- *Postconditions:* T4a (SyntacticEquivalence): the field-segment constraint (no two adjacent zeros, `t₁ ≠ 0`, `t_{#t} ≠ 0`) is equivalent to the condition that every field segment of `t` is non-empty — giving the semantic reading (once T4c supplies labels) that every present field has at least one component. T4b (UniqueParse): the function `fields(t)` that extracts node, user, document, and element fields is well-defined and uniquely computable from `t` alone. T4c (LevelDetermination): `zeros(t)` bijects with hierarchical level — `zeros(t) = 0` ↔ node address, `zeros(t) = 1` ↔ user address, `zeros(t) = 2` ↔ document address, `zeros(t) = 3` ↔ element address.

---

## T4a — SyntacticEquivalence

The three positional conditions from T4 (no adjacent zeros, nonzero first and last components) are logically equivalent to the condition that every field segment of the tumbler is non-empty. This records the bridge between T4's positional form and the semantic reading that every present field contributes at least one component.

*Formal Contract:*
- *Preconditions:* `t ∈ T` — so by T0, every component of `t` lies in ℕ and a non-zero component is automatically strictly positive; no additional positivity assumption on T4 is invoked.
- *Postconditions:* The three positional conditions of T4's field-segment constraint — (i) no two zeros are adjacent, (ii) `t₁ ≠ 0`, (iii) `t_{#t} ≠ 0` — hold if and only if every field segment of `t` (a maximal contiguous sub-sequence of non-zero positions delimited by the zeros of `t`) is non-empty. Once T4c assigns hierarchical labels to segments, this is equivalent to the semantic statement that every present field has at least one component.

---

## T4b — UniqueParse

Because T4's role assignment makes zeros exactly the field separators — and T0's carrier ℕ ensures no other value can play that role — the decomposition function fields(t) is uniquely recoverable by a single scan of the component sequence. T3's canonical representation ensures there is no alternative encoding that could yield a different scan result.

*Formal Contract:*
- *Preconditions:* `t` satisfies T3 (CanonicalRepresentation): the component sequence of `t` is fixed by sequence identity, with no alternative encoding yielding different component values. `t` satisfies the T4 constraints (at most three zero-valued components, field-segment constraint — no two zeros adjacent, `t₁ ≠ 0`, `t_{#t} ≠ 0`). Positivity of non-zero components is supplied by T0's carrier ℕ, not by a separate T4 clause.
- *Postconditions:* `fields(t)` — the decomposition into node, user, document, and element sub-sequences — is well-defined and uniquely determined by `t`.

---

## T4c — LevelDetermination

The count of zero-valued components in a valid tumbler is a bijection onto hierarchical level: zero zeros means node address, one means user, two means document, three means element. This follows because every zero is a separator (T4b), so the zero count equals the separator count, and the number of fields present is always the separator count plus one.

*Formal Contract:*
- *Preconditions:* `t` satisfies the T4 constraints (at most three zero-valued components, field-segment constraint — no two zeros adjacent, `t₁ ≠ 0`, `t_{#t} ≠ 0`). Positivity of non-zero components is supplied by T0's carrier ℕ. `t` satisfies T4b (UniqueParse): every zero in `t` is a field separator and every separator is a zero, so the separator positions are exactly the zero-valued positions.
- *Postconditions:* `zeros(t)` counts exactly the number of field separators in `t`, and the number of fields present equals `zeros(t) + 1`. The mapping `zeros(t) → hierarchical level` is a bijection on `{0, 1, 2, 3}`: distinct zero counts imply distinct hierarchical levels (injectivity), and every level in {node, user, document, element} is realized by exactly one zero count (surjectivity).

---

## T5 — ContiguousSubtrees

If two tumblers a and c share a common prefix p, then every tumbler b between them in the lexicographic order also shares that prefix. This means every prefix-defined subtree occupies a contiguous interval on the tumbler line — no address from an unrelated subtree can appear between two addresses in the same subtree.

*Formal Contract:*
- *Preconditions:* `a, b, c ∈ T`; `p` is a tumbler prefix with `#p ≥ 1`; `p ≼ a`; `p ≼ c`; `a ≤ b ≤ c` under the lexicographic order T1.
- *Depends:* Prefix (PrefixRelation) — the preconditions `p ≼ a` and `p ≼ c` are unfolded via Prefix to `#a ≥ #p`, `#c ≥ #p`, and `aᵢ = cᵢ = pᵢ` for `1 ≤ i ≤ #p`; the conclusion `p ≼ b` re-folds component-wise agreement of `b` with `p` on positions `1, ..., #p` back into the Prefix relation. T1 (LexicographicOrder), case (i) — Subcases 1a and 1b derive contradictions `b < a` and `c < b` from divergence-position witnesses at position `k`; Case 2 likewise derives `c < b` from a divergence-position witness at position `k ≤ #b`; case (ii) — Case 2 invokes T1 case (ii) on the hypothesis `a ≤ b` (with `a ≠ b` forced by `#a > #b`) to characterise `a < b` as either case-(i) divergence or proper-prefix, then excludes the proper-prefix branch by `#a > #b`. T3 (CanonicalRepresentation) — Case 2 cites T3 to conclude `a ≠ b` from `#a > #b`, since distinct lengths imply distinct tumblers.
- *Postconditions:* `p ≼ b` — the tumbler `b` extends the prefix `p`, and therefore belongs to the same subtree as `a` and `c`.

This is Nelson's key insight: "The first point of a span may designate a server, an account, a document or an element; so may the last point. There is no choice as to what lies between; this is implicit in the choice of first and last point." T5 is what makes this true. A span between two endpoints under the same prefix captures exactly the addresses under that prefix between those endpoints — no addresses from unrelated subtrees can interleave.

Because the hierarchy is projected onto a flat line (T1), containment in the tree corresponds to contiguity on the line. Nelson: "A span may be visualized as a zone hanging down from the tumbler line — what is called in computer parlance a depth-first spanning tree." Every subtree maps to a contiguous range, and every contiguous range within a subtree stays within the subtree.

---

## T6 — DecidableContainment

Containment queries (same server? same account? same document? one address a prefix of another?) can be decided by extracting and comparing the relevant field sequences from the two tumbler representations alone, with no external index or registry. The decidability is a direct corollary of T4's structural constraints; it is stated separately because it is load-bearing for decentralized operation.

*Formal Contract:*
- *Preconditions:* `a, b ∈ T` are valid tumblers satisfying T4 (at most three zeros, no adjacent zeros, no leading or trailing zero). Positivity of non-zero components is supplied by T0's carrier ℕ.
- *Postconditions:* (a) The procedure terminates and returns YES iff `N(a) = N(b)` (componentwise). (b) The procedure terminates and returns YES iff `zeros(a) ≥ 1 ∧ zeros(b) ≥ 1 ∧ N(a) = N(b) ∧ U(a) = U(b)`; returns NO if either tumbler lacks a user field. (c) The procedure terminates and returns YES iff `zeros(a) ≥ 2 ∧ zeros(b) ≥ 2 ∧ N(a) = N(b) ∧ U(a) = U(b) ∧ D(a) = D(b)`; returns NO if either tumbler lacks a document field. (d) The procedure terminates and returns YES iff `zeros(a) ≥ 2 ∧ zeros(b) ≥ 2 ∧ #D(a) ≤ #D(b) ∧ (A k : 1 ≤ k ≤ #D(a) : D(a)ₖ = D(b)ₖ)`; returns NO if either tumbler lacks a document field. All decisions use only the tumbler representations of `a` and `b`, via `fields(t)` (T4(b)) and componentwise comparison on finite sequences of natural numbers.

T6 is a corollary: it follows immediately from T4 — we extract the relevant fields and compare. We state it separately because the decidability claim is load-bearing for decentralized operation, but it introduces no independent content beyond T4.

We must note what T6(d) does NOT capture. The document field records the *allocation hierarchy* — who baptised which sub-number — not the *derivation history*. Version `5.3` was allocated under document `5`, but this tells us nothing about which version's content was copied to create `5.3`. Nelson: "the version, or subdocument number is only an accidental extension of the document number, and strictly implies no specific relationship of derivation." Formal version-derivation requires the version graph, not just the address.

Nelson confirms that shared prefix means shared containing scope: "The owner of a given item controls the allocation of the numbers under it." The prefix IS the path from root to common ancestor. But he cautions: "Tumblers do not affect the user-level structure of the documents; they only provide a mapping mechanism, and impose no categorization and no structure on the contents of a document." Shared prefix guarantees containment and ownership, never semantic categorization.

Gregory's implementation confirms the distinction between ordering and containment. The codebase provides both `tumblercmp` (total order comparison) and `tumbleraccounteq` (prefix-matching predicate with zero-as-wildcard semantics). The latter truncates the candidate to the length of the parent and checks for exact match — this is the operational realization of T6, and it is a genuinely different algorithm from the ordering comparison.

---

## T7 — SubspaceDisjointness

For two element-level addresses (those with exactly three zero delimiters), differing in the first element-field component places them in distinct subspaces — text (1) and links (2) cannot share an address. This structural separation guarantees that arithmetic within one subspace never produces addresses belonging to another.

*Formal Contract:*
- *Preconditions:* `a, b ∈ T` with `zeros(a) = zeros(b) = 3` (both are element-level addresses with well-formed field structure per T4).
- *Postconditions:* `a.E₁ ≠ b.E₁ ⟹ a ≠ b`.

We state T7 explicitly because it is load-bearing for the guarantee that operations within one content type do not interfere with another. T7 is the structural basis — arithmetic within subspace 1 cannot produce addresses in subspace 2, because the subspace identifier is part of the address, not metadata.

The ordering T1 places all text addresses (subspace 1) before all link addresses (subspace 2) within the same document, because `1 < 2` at the subspace position. This is a consequence, not an assumption — it falls out of the lexicographic order.

---

## T8 — AllocationPermanence

The set of allocated addresses grows monotonically across every state transition: once an address enters the allocated set it is never removed. This follows directly from the absence of any removal operation in the transition vocabulary, so the guarantee covers all present and future operations.

*Formal Contract:*
- *Invariant:* For every state transition s → s', `allocated(s) ⊆ allocated(s')`.
- *Depends:* AllocatedSet (AllocatedSet) — defines `allocated(s)` as the union of realized allocator domains, state as allocator tree configuration, and state transition as application of an operation from Σ. NoDeallocation (the system defines no removal operation — the sole premise required for the monotonicity conclusion; any transition either preserves or modifies the allocated set, and the axiom precludes removal, so every modification is an addition).
- *Frame:* The monotonicity conclusion holds for every operation in Σ, present or future, because NoDeallocation constrains the entire transition vocabulary: no operation that removes an element from `allocated(s)` may belong to Σ.

---

## T9 — ForwardAllocation

When two addresses are produced by the same allocator and one is allocated before the other, the earlier-allocated address is strictly smaller under the tumbler total order. Well-definedness of "allocated before" requires domain disjointness across allocators and index injectivity within a single allocator; the strict ordering itself follows from the monotonicity of the increment operation and transitivity of the total order.

*Formal Contract:*
- *Definitions:*
  - `allocated_before(a, b)` ≡ `a = tᵢ ∧ b = tⱼ ∧ i < j` in T10a's enumeration of `dom(A)`. The predicate is well-defined on pairs `a, b` satisfying `same_allocator(a, b)` (T10a) by the joint action of two T10a consequences. T10a.6 (DomainDisjointness) makes the witnessing allocator `A` a unique function of `(a, b)` — cross-allocator uniqueness. T10a.7 (EnumerationInjectivity) makes the index `n` with `c = tₙ` in `dom(A)` a unique function of `c` — within-allocator uniqueness — so the indices `i, j` with `a = tᵢ, b = tⱼ` are uniquely determined once `A` is fixed. The bare existential in `same_allocator` alone would leave both forms of ambiguity unresolved; T10a.6 and T10a.7 together close them both.
- *Depends:* T10a (AllocatorDiscipline) — supplies the definitions of `dom(A)` and `same_allocator`, and establishes that each allocator produces its sibling outputs exclusively by repeated application of `inc(·, 0)`. T10a.6 (DomainDisjointness) — cross-allocator witness uniqueness: under `same_allocator(a, b)`, the witnessing allocator `A` is uniquely determined by `(a, b)`. T10a.7 (EnumerationInjectivity) — within-allocator index uniqueness: the map `n ↦ tₙ` on each allocator's enumeration is injective, so no tumbler in `dom(A)` carries two distinct indices. Together, T10a.6 and T10a.7 make `i` and `j` single-valued functions of `(a, b)`, underwriting well-definedness of `allocated_before` as written. TA5 (HierarchicalIncrement), postcondition (a) — strict monotonicity of `inc`: `inc(tᵢ, 0) > tᵢ`, providing the base case. T1 (LexicographicOrder), postcondition (c) — transitivity of the strict order, providing the inductive step.
- *Preconditions:* `a, b ∈ T` with `same_allocator(a, b) ∧ allocated_before(a, b)`.
- *Postconditions:* `a < b` under the tumbler order T1.

---

## TA-LC — LeftCancellation

TumblerAdd is left-cancellative: if a ⊕ x = a ⊕ y with both additions well-defined, then x = y. Differing action points between x and y lead to immediate contradiction via TumblerAdd's prefix-copy rule; once a shared action point is established, component-wise equality at every position and equal lengths force x = y by canonical representation.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, w ∈ T, a < b, a ≥ w, b ≥ w
- *Postconditions:* a ⊖ w ≤ b ⊖ w

### Verification of TA4

**Claim.** `(a ⊕ w) ⊖ w = a` under the full precondition: `k = #a`, `#w = k`, `(A i : 1 ≤ i < k : aᵢ = 0)`.

*Proof.* Let `k` be the action point of `w`. Since `k = #a`, the addition `a ⊕ w` produces a result `r` with: `rᵢ = aᵢ = 0` for `i < k` (by the zero-prefix condition), `rₖ = aₖ + wₖ`, and `rᵢ = wᵢ` for `i > k`. Crucially, there are no components of `a` beyond position `k` — the tail replacement discards nothing. By the result-length identity, `#r = #w = k`, so `r = [0, ..., 0, aₖ + wₖ]`.

Now subtract `w` from `r`. The subtraction scans for the first divergence between `r` and `w`. For `i < k`: `rᵢ = 0 = wᵢ` (both are zero — `aᵢ` by the zero-prefix precondition, `wᵢ` by definition of action point). Two sub-cases arise at position `k`.

*Sub-case (i): `aₖ > 0`.* Then `rₖ = aₖ + wₖ > wₖ`, and the first divergence is at position `k`. The subtraction produces: positions `i < k` get zero, position `k` gets `rₖ - wₖ = aₖ`, and positions `i > k` copy from `r`, giving `rᵢ = wᵢ`. Since `k = #a` and `#w = k`, there are no trailing components. The result is `[0, ..., 0, aₖ] = a`. For valid addresses, T4's field-segment constraint forces `a_{#a} ≠ 0` at the last position, and since `a_{#a} ∈ ℕ` by T0, `a_{#a} ≠ 0 ⇒ a_{#a} > 0`; specialising to `k = #a` gives `aₖ > 0`, so this sub-case always applies in the address context.

*Sub-case (ii): `aₖ = 0`.* Then `a` is a zero tumbler. The addition gives `rₖ = wₖ`. Since `#r = #w` (result-length identity) and `#w = k` (precondition), we have `r = w`. The subtraction `w ⊖ w` yields the zero tumbler of length `k`, which is `a`. ∎

### Cancellation properties of ⊕

TumblerAdd's constructive definition determines each component of the result from exactly one input. This makes the operation left-cancellative.

**TA-LC (LeftCancellation).** If a ⊕ x = a ⊕ y with both sides well-defined (TA0 satisfied for both), then x = y.

*Proof.* We show that from the hypothesis `a ⊕ x = a ⊕ y`, with both additions satisfying TA0, it follows that `x = y`. The argument proceeds in two stages: first we establish that `x` and `y` share the same action point, then we show component-wise and length equality.

Let `k₁` be the action point of `x` and `k₂` the action point of `y`. Both exist because TA0 requires `Pos(x)` and `Pos(y)`, so each has at least one nonzero component. We eliminate both strict orderings.

**Case k₁ < k₂.** Since `k₁ < k₂` and the action point is the first nonzero component, every component of `y` before position `k₂` is zero — in particular `y_{k₁} = 0`. Position `k₁` therefore falls in the prefix-copy region of the addition `a ⊕ y`: by TumblerAdd, `(a ⊕ y)_{k₁} = a_{k₁}`. In the addition `a ⊕ x`, position `k₁` is the action point itself, so TumblerAdd gives `(a ⊕ x)_{k₁} = a_{k₁} + x_{k₁}`. From `a ⊕ x = a ⊕ y` we obtain `a_{k₁} + x_{k₁} = a_{k₁}`, hence `x_{k₁} = 0`. But `k₁` is the action point of `x`, so by definition `x_{k₁} > 0` — contradiction.

**Case k₂ < k₁.** Since `k₂ < k₁` and the action point is the first nonzero component, every component of `x` before position `k₁` is zero — in particular `x_{k₂} = 0`. Position `k₂` therefore falls in the prefix-copy region of the addition `a ⊕ x`: by TumblerAdd, `(a ⊕ x)_{k₂} = a_{k₂}`. In the addition `a ⊕ y`, position `k₂` is the action point itself, so TumblerAdd gives `(a ⊕ y)_{k₂} = a_{k₂} + y_{k₂}`. From `a ⊕ x = a ⊕ y` we obtain `a_{k₂} = a_{k₂} + y_{k₂}`, hence `y_{k₂} = 0`. But `k₂` is the action point of `y`, so by definition `y_{k₂} > 0` — contradiction.

Both strict orderings are impossible, so `k₁ = k₂`. Write `k` for this common action point. We now verify that `x` and `y` agree at every position and have the same length.

**Positions i < k.** Both `x` and `y` have action point `k`, so by definition of action point every component before `k` is zero: `xᵢ = 0` and `yᵢ = 0`. Therefore `xᵢ = yᵢ = 0`.

**Position i = k.** TumblerAdd gives `(a ⊕ x)_k = a_k + x_k` and `(a ⊕ y)_k = a_k + y_k`. From `a ⊕ x = a ⊕ y` we get `a_k + x_k = a_k + y_k`, hence `x_k = y_k` by cancellation in ℕ.

**Positions i > k.** For both additions, positions after the action point fall in the tail-copy region of TumblerAdd: `(a ⊕ x)_i = x_i` and `(a ⊕ y)_i = y_i`. From `a ⊕ x = a ⊕ y` we get `x_i = y_i`.

**Length.** By T3 (CanonicalRepresentation), `a ⊕ x = a ⊕ y` implies `#(a ⊕ x) = #(a ⊕ y)`. The result-length identity (TumblerAdd) gives `#(a ⊕ w) = #w` for any well-defined addition. Applying this to both sides: `#x = #(a ⊕ x) = #(a ⊕ y) = #y`.

All components of `x` and `y` agree at every position and `#x = #y`, so `x = y` by T3 (CanonicalRepresentation).  ∎

TumblerAdd is *left-cancellative*: the start position can be "divided out" from equal results, recovering the displacement uniquely. This is a direct consequence of TumblerAdd's constructive definition — each component of the result is determined by exactly one input, so equality of results propagates back to equality of inputs.

*Worked example.* Let a = [2, 5] and suppose a ⊕ x = a ⊕ y = [2, 8]. We recover x and y uniquely. First, the action points must agree. Suppose k_x = 1: then (a ⊕ x)₁ = a₁ + x₁ = 2 + x₁ = 2, giving x₁ = 0, which contradicts k_x = 1 being the first nonzero component. So k_x ≠ 1, and since #x ≤ 2 (from the result length), k_x = 2. Now suppose k_y = 1: then (a ⊕ y)₁ = a₁ + y₁ = 2 + y₁ = 2, giving y₁ = 0, which contradicts k_y = 1. So k_y = 2. At position k = 2: a₂ + x₂ = 5 + x₂ = 8 gives x₂ = 3, and a₂ + y₂ = 5 + y₂ = 8 gives y₂ = 3. For i < k: x₁ = 0 = y₁ (both zero before the action point). From the result-length identity: #(a ⊕ x) = #x, so #x = 2 = #y. By T3, x = y = [0, 3].

*Formal Contract:*
- *Preconditions:* a, x, y ∈ T; Pos(x); Pos(y); actionPoint(x) ≤ #a; actionPoint(y) ≤ #a; a ⊕ x = a ⊕ y
- *Postconditions:* x = y

---

## TA-MTO — ManyToOne

Two tumblers a and b yield the same result under displacement w if and only if they agree on every component from position 1 through w's action point. This is TumblerAdd's many-to-one property made precise: components of the starting position beyond the action point are overwritten by the displacement's tail and cannot influence or be recovered from the result.

*Formal Contract:*
- *Preconditions:* w ∈ T, Pos(w), a ∈ T, b ∈ T, #a ≥ actionPoint(w), #b ≥ actionPoint(w)
- *Postconditions:* a ⊕ w = b ⊕ w ⟺ (A i : 1 ≤ i ≤ actionPoint(w) : aᵢ = bᵢ)

---

## TA-RC — RightCancellationFailure

TumblerAdd is not right-cancellative: distinct tumblers a ≠ b can satisfy a ⊕ w = b ⊕ w for the same positive displacement w. The mechanism is tail replacement — any two starting positions that agree up to the action point but differ beyond it are mapped to the same result, so the starting position cannot be recovered from the result alone.

*Formal Contract:*
- *Postconditions:* ∃ a, b, w ∈ T : Pos(w) ∧ actionPoint(w) ≤ #a ∧ actionPoint(w) ≤ #b ∧ a ≠ b ∧ a ⊕ w = b ⊕ w

The mechanism is TumblerAdd's tail replacement: components of the start position after the action point are discarded and replaced by the displacement's tail. Any two starts that agree on components 1..k and differ only on components after k will produce the same result under any displacement with action point k. Formally:

---

## TA-assoc — AdditionAssociative

Tumbler addition is associative: (a ⊕ b) ⊕ c = a ⊕ (b ⊕ c) whenever action points are properly ordered (k_b ≤ #a and k_c ≤ #b). The result length always equals the length of the rightmost operand, and the effective action point of a composed displacement b ⊕ c is min(k_b, k_c).

*Formal Contract:*
- *Preconditions:* `a ∈ T`, `b ∈ T`, `c ∈ T`, `Pos(b)`, `Pos(c)`, `k_b ≤ #a`, `k_c ≤ #b` (where `k_b = actionPoint(b)`, `k_c = actionPoint(c)`; these left-side conditions subsume the right-side conditions since `k_b ≤ #a` implies `min(k_b, k_c) ≤ #a`)
- *Postconditions:* `(a ⊕ b) ⊕ c = a ⊕ (b ⊕ c)`; `#((a ⊕ b) ⊕ c) = #(a ⊕ (b ⊕ c)) = #c`; `actionPoint(b ⊕ c) = min(k_b, k_c)`

**Addition is not commutative.** We do NOT require `a ⊕ b = b ⊕ a`. The operands play asymmetric roles: the first is a *position*, the second is a *displacement*. Swapping them is semantically meaningless. Gregory's `absadd` confirms: the first argument supplies the prefix, the second the suffix — the reverse call gives a different (and typically wrong) result.

**There is no multiplication or division.** Gregory's analysis of the complete codebase confirms: no `tumblermult`, no `tumblerdiv`, no scaling operation of any kind. The arithmetic repertoire is: add, subtract, increment, compare. Tumblers are *addresses*, not quantities. You don't multiply two file paths or divide an address by three.

**Tumbler differences are not counts.** Nelson is emphatic: "A tumbler-span is not a conventional number, and it does not designate the number of bytes contained. It does not designate a number of anything." The difference between two addresses specifies *boundaries*, not *cardinality*. What lies between those boundaries depends on the actual population of the tree, which is dynamic and unpredictable. Between sibling addresses 3 and 7, document 5 might have arbitrarily many descendants — the span from 3 to 7 encompasses all of them, and their count is unknowable from the addresses alone. The arithmetic is an *addressing calculus*, not a *counting calculus*.

The absence of algebraic structure beyond a monotone shift is not a deficiency. It is the *minimum* that addressing requires. An ordered set with an order-preserving shift and an allocation increment is sufficient. Anything more would be unused machinery and unverified obligation.

---

## TA-strict — StrictIncrease

Applying any positive displacement strictly advances a tumbler: a ⊕ w > a whenever the addition is well-defined. This rules out degenerate no-op models that otherwise satisfy all addition axioms, and is the foundational guarantee that address spans of the form [s, s ⊕ ℓ) are always non-empty.

*Formal Contract:*
- *Preconditions:* `a ∈ T`, `w ∈ T`, `Pos(w)`, `k ≤ #a` where `k` is the action point of `w`
- *Postconditions:* `a ⊕ w > a`

---

## TA0 — WellDefinedAddition

Adding a positive displacement w to a position a yields a valid tumbler, provided w's action point does not exceed a's length. The action-point constraint ensures the piecewise construction can actually access position k within a; without it, the operation is undefined. The result inherits its length from w, not from a.

*Formal Contract:*
- *Preconditions:* a ∈ T, w ∈ T, Pos(w) (TA-Pos, this ASN), actionPoint(w) ≤ #a (ActionPoint, this ASN)
- *Depends:* T0 (CarrierSetDefinition) — supplies the carrier-set definition of T referenced by the preconditions `a ∈ T` and `w ∈ T`: without T0, `T` is an undefined symbol and the membership assertions have no meaning; the proof's assertion `#w ≥ 1` is T0's length axiom `(A a ∈ T :: #a ≥ 1)` applied to the hypothesis `w ∈ T`, not a consequence of TumblerAdd (TumblerAdd supplies the equality `#(a ⊕ w) = #w` but the lower bound on `#w` itself is T0's length axiom); and the postcondition `a ⊕ w ∈ T` reassembles T0's three-part membership characterisation of T as the set of finite sequences over ℕ with length ≥ 1 — TumblerAdd supplies that each component of `a ⊕ w` lies in ℕ, and T0's length axiom supplies `#(a ⊕ w) ≥ 1` (via `#(a ⊕ w) = #w ≥ 1`), jointly discharging the T0 membership condition. TA-Pos (PositiveTumbler, this ASN) — supplies the predicate `Pos(·)` used in the precondition `Pos(w)`; without this citation `Pos` is an undefined symbol in the preconditions, and the action-point existence argument in the proof has no licensed source for the positivity hypothesis that ActionPoint requires as input. TumblerAdd (TumblerAdd, this ASN) — the proof delegates entirely to TumblerAdd's piecewise construction: component membership in ℕ and the result-length identity `#(a ⊕ w) = #w` are TumblerAdd postconditions. ActionPoint (ActionPoint, this ASN) — the precondition `actionPoint(w) ≤ #a` references ActionPoint's definition of the action-point function.
- *Postconditions:* a ⊕ w ∈ T, #(a ⊕ w) = #w

---

## TA1-strict — StrictOrderPreservation

When the action point of the displacement lands at or after the divergence point of two strictly ordered positions, addition preserves strict order — the original ordering relationship survives intact. If the action point falls before the divergence, the two results collapse to equality; order degrades but never reverses.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, w ∈ T, a < b, Pos(w), actionPoint(w) ≤ min(#a, #b), actionPoint(w) ≥ divergence(a, b)
- *Postconditions:* a ⊕ w < b ⊕ w

But TA1 alone does not guarantee that addition *advances* a position. It preserves relative order between two positions but is silent about the relationship between `a` and `a ⊕ w`. We need:

---

## TA1 — OrderPreservationUnderAddition

Adding the same positive displacement to two ordered positions preserves their relative order weakly: a < b implies a ⊕ w ≤ b ⊕ w. This holds universally — regardless of where the action point falls relative to the divergence — so no ordering relationship can be reversed by a common advancement.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, w ∈ T, a < b, Pos(w) (TA-Pos, this ASN), actionPoint(w) ≤ min(#a, #b) (ActionPoint, this ASN)
- *Depends:* T0 (CarrierSetDefinition) — the carrier set `T`, the length `#·`, and the component projection `·ᵢ` used throughout the preconditions, proof, and postconditions come from T0's characterisation of T as finite sequences over ℕ with length ≥ 1; the sub-case `j = k` step "addition of a fixed natural number preserves strict inequality on ℕ" — concluding `aₖ + wₖ < bₖ + wₖ` from `aₖ < bₖ` — is T0's order-compatibility of `+` (`(A m, n, p ∈ ℕ : n ≥ p : m + n ≥ m + p)`) combined with trichotomy to promote `≥` to `>` when the inputs are strictly ordered, not a consequence of TA0, TumblerAdd, T1, or T3; this use of T0 is independent of any T0 use transitively inherited through TA0, so it is cited directly here to match the per-step citation convention established by TA0, TumblerAdd, ActionPoint, and TA-Pos. TA-Pos (PositiveTumbler, this ASN) — supplies the predicate `Pos(w)` used in the precondition; without this citation `Pos` is an undefined symbol in the preconditions, and the proof's appeal to ActionPoint for `k = actionPoint(w)` has no licensed source for the positivity hypothesis that ActionPoint requires as input. ActionPoint (ActionPoint, this ASN) — the precondition `actionPoint(w) ≤ min(#a, #b)` references ActionPoint's definition of the action-point function; the proof's opening step "Let `k` be the action point of `w`" names `k = actionPoint(w)` and relies on ActionPoint's postconditions `1 ≤ k ≤ #w` and `wᵢ = 0 for i < k` (implicit in TumblerAdd's prefix-copy region for `i < k` but introduced here through `k`'s definition), so ActionPoint is cited directly rather than left to transitive inheritance through TA0 or TumblerAdd. TA0 (WellDefinedAddition) — both `a ⊕ w` and `b ⊕ w` must be well-defined members of T with length `#w`. TumblerAdd (TumblerAdd) — the three-region piecewise structure determines how each component of the result is computed. T1 (LexicographicOrder) — case analysis on the ordering `a < b` drives the proof structure, and T1 case (i) establishes the ordering of the results. T3 (CanonicalRepresentation) — component-wise agreement with equal length concludes `a ⊕ w = b ⊕ w` in the prefix and j > k sub-cases.
- *Postconditions:* a ⊕ w ≤ b ⊕ w

Strict order preservation holds under a tighter condition. We first need a precise notion of where two tumblers first differ.

---

## TA2 — WellDefinedSubtraction

Tumbler subtraction a ⊖ w is well-defined whenever a ≥ w, producing a valid tumbler whose length is max(#a, #w). The result lies in T and correctly represents the displacement needed to reach a from w.

*Formal Contract:*
- *Preconditions:* a ∈ T, w ∈ T, a ≥ w
- *Depends:* TumblerSub (TumblerSub) — supplies the piecewise construction of `a ⊖ w`: zero-padding, case split on divergence, componentwise definition, and result length `max(#a, #w)`. T0 (CarrierSetDefinition) — the proof uses T0 in two ways: (1) T0's minimum-length guarantee `#a ≥ 1` for `a ∈ T` yields `p = max(#a, #w) ≥ 1`; (2) T0's carrier-set characterisation — membership in T as a finite sequence over ℕ with length ≥ 1 — is the criterion applied in both cases to conclude `a ⊖ w ∈ T`. T1 (LexicographicOrder) — the proof derives `a > w` from `a ≥ w ∧ a ≠ w` via T1 trichotomy, then uses T1's two cases to establish `aₖ ≥ wₖ` at the divergence point. T3 (CanonicalRepresentation) — the proof concludes `a ≠ w` from the existence of a padded divergence: if `a = w` by T3, the padded sequences would be identical, contradicting the case hypothesis.
- *Postconditions:* a ⊖ w ∈ T, #(a ⊖ w) = max(#a, #w)

---

## TA3-strict — OrderPreservationUnderSubtractionStrict

Subtracting a common lower bound from two equal-length ordered positions preserves strict order: if a < b and both dominate w with equal lengths, then a ⊖ w < b ⊖ w. The equal-length precondition is load-bearing — without it the piecewise subtraction could shift the divergence point in a way that collapses or reverses the ordering.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, w ∈ T, a < b, a ≥ w, b ≥ w, #a = #b
- *Postconditions:* a ⊖ w < b ⊖ w

---

## TA3 — OrderPreservationUnderSubtractionWeak

Tumbler subtraction preserves weak order: if a < b and both addresses are at least as large as the subtrahend w, then subtracting w from a yields a result no greater than subtracting w from b. The guarantee is weak (≤ rather than strict <) because equal results are possible when a and b diverge only below the subtraction point.

*Formal Contract:*
- *Preconditions:* a ∈ T, b ∈ T, w ∈ T, a < b, a ≥ w, b ≥ w
- *Depends:* TA2 (WellDefinedSubtraction) — establishes that `a ⊖ w` and `b ⊖ w` are well-formed tumblers in T, making the comparison well-defined. TumblerSub (TumblerSub) — supplies the component-wise subtraction definition (zero-padded divergence, three-phase formula) and the precondition consequence that `aₖ > wₖ` at `k = zpd(a, w)`, used throughout every case. ZPD (ZeroPaddedDivergence) — supplies the existence biconditional (zpd defined iff not zero-padded-equal) used in Sub-case B1 and the B1–B2 bridge to establish well-definedness of `dₐ` and `d_b`, the first-position characterisation (`zpd(a, w) = min{k : aₖ ≠ wₖ}`) used throughout Sub-cases B2–B4 to compare divergence positions, and the pre-zpd agreement guarantee (`aᵢ = wᵢ` for `i < zpd(a, w)`) invoked throughout Case B. T1 (LexicographicOrder) — provides the strict ordering `<` for the initial case split on `a < b` (prefix vs. component divergence), the derived relations `≥` and `≤` (T1 postcondition (d)) for the preconditions and postcondition, and the ordering comparisons in every sub-case. T3 (CanonicalRepresentation) — concludes `a ⊖ w = b ⊖ w` from component-wise agreement in Sub-case A2's length-equality branch. TA6 (ZeroTumblers) — establishes that a zero tumbler is strictly less than any positive tumbler, used in Sub-cases A1, A3, and B1 to conclude `a ⊖ w < b ⊖ w` when one result is the zero tumbler.
- *Postconditions:* a ⊖ w ≤ b ⊖ w

---

## TA4 — PartialInverse

Tumbler addition and subtraction are partial inverses only under tight structural conditions: (a ⊕ w) ⊖ w = a holds exactly when the action point of w coincides with the last position of a and all components of a before that position are zero. The restriction is necessary because addition discards the trailing structure of its first argument below the action point, making recovery impossible in the general case.

*Formal Contract:*
- *Preconditions:* `a ∈ T`, `w ∈ T`, `Pos(w)`, `k = #a`, `#w = k`, `(A i : 1 ≤ i < k : aᵢ = 0)`, where `k` is the action point of `w`
- *Postconditions:* `(a ⊕ w) ⊖ w = a`

Gregory's analysis confirms that `⊕` and `⊖` are NOT inverses in general. The implementation's `absadd` is asymmetric: the first argument supplies the high-level prefix, the second supplies the low-level suffix. When `d = a ⊖ b` strips a common prefix (reducing the exponent), `b ⊕ d` puts the difference in the wrong operand position — `absadd`'s else branch discards the first argument entirely and returns the second. The operand-order asymmetry causes total information loss even before any digit overflow.

The reverse direction is equally necessary:

---

## TA5-SIG — LastSignificantPosition

Defines sig(t) as the index of the rightmost nonzero component of tumbler t; for an all-zero tumbler, sig(t) = #t by convention. The result always satisfies 1 ≤ sig(t) ≤ #t, giving a well-defined handle on where a tumbler's significant content ends.

*Formal Contract:*
- *Preconditions:* `t ∈ T` (any tumbler with `#t ≥ 1`).
- *Definition:* `sig(t) = max({i : 1 ≤ i ≤ #t ∧ tᵢ ≠ 0})` when `(E i : 1 ≤ i ≤ #t : tᵢ ≠ 0)`; `sig(t) = #t` when `(A i : 1 ≤ i ≤ #t : tᵢ = 0)`.
- *Postconditions:* `1 ≤ sig(t) ≤ #t`.

---

## TA5-SigValid — SigOnValidAddresses

For any valid address satisfying T4, the signature function sig(t) equals the length of the tumbler — that is, the last component is always the rightmost nonzero position. This follows directly from T4's field-segment constraint, which forbids a zero final component.

*Formal Contract:*
- *Precondition:* `t` satisfies T4 (valid address tumbler: at most three zero-valued field separators, field-segment constraint — no two zeros adjacent, `t₁ ≠ 0`, `t_{#t} ≠ 0`). Positivity of non-zero components is supplied by T0's carrier ℕ.
- *Guarantee:* `sig(t) = #t`.

---

## TA5 — HierarchicalIncrement

The increment operation inc(t, k) produces a new address strictly greater than t under lexicographic order: k = 0 advances the rightmost nonzero component to yield the next peer at the same depth, while k > 0 appends k new components to yield a child address at depth k below t. For valid addresses, inc(t, 0) produces the next sibling; inc(t, k) for k > 0 produces a descendant, with t's zero-extension subtree lying strictly between t and the result.

*Formal Contract:*
- *Preconditions:* `t ∈ T`, `k ≥ 0`.
- *Definition:* `inc(t, k)`: when `k = 0`, modify position `sig(t)` (TA5-SIG) to `t_{sig(t)} + 1`; when `k > 0`, extend by `k` positions with `k - 1` zeros and final `1`.
- *Depends:* T0 (CarrierSetDefinition) — the conclusion `t' ∈ T` at the end of the construction ("`t'` is a finite sequence of natural numbers with length ≥ 1, so `t' ∈ T`") rests on T0's characterisation of T as finite sequences over ℕ with length ≥ 1; the sibling step `t'_{sig(t)} = t_{sig(t)} + 1` (construction for `k = 0` and postcondition (c)) invokes T0's closure of ℕ under successor to confirm that `t_{sig(t)} + 1 ∈ ℕ`, so that `t'` remains a sequence over ℕ; the child construction's designated components — the `k − 1` field separators `0` at positions `#t + 1, ..., #t + k − 1` and the first child `1` at position `#t + k` (construction for `k > 0` and postcondition (d)) — name members of ℕ that T0 enumerates, with `0` supplied as the additive identity and `1` supplied by closure of ℕ under successor, so that the ℕ-membership of the components set in the child branch is discharged from T0 rather than left implicit; the verification of (a) in Case `k = 0` invokes T0's strict successor inequality (`n < n + 1` for every `n ∈ ℕ`) at the step "`t'_j = t_j + 1 > t_j`, since `n + 1 > n` for every `n ∈ ℕ`", supplying the strict inequality at the divergence position `j = sig(t)` that T1 case (i) then consumes, so the "`t_j + 1 > t_j`" claim is discharged from T0 rather than left as an implicit appeal to ℕ-successor behaviour; the verification of (a) in Case `k > 0` invokes two further T0 facts at the length step `#t + 1 ≤ #t + k` that supplies T1 case (ii) with its non-strict witness-position bound, namely T0's discreteness (`m ≤ n < m + 1 ⟹ n = m`, instantiated at `m = 0`) to sharpen the hypothesis `k > 0` in ℕ to `k ≥ 1` — no natural lies strictly between `0` and `0 + 1` — and T0's order-compatibility of addition (`p ≤ n ⟹ m + p ≤ m + n`, instantiated at `m = #t`, `p = 1`, `n = k`) to lift `1 ≤ k` to `#t + 1 ≤ #t + k`, so the "`m + 1 ≤ m + k`" step is discharged from T0 rather than left as an implicit appeal to ℕ-arithmetic, matching the per-step citation convention established for `T1`, `TumblerAdd`, `ActionPoint`, and `TA-Pos` and already applied in T10a-N's Depends for the structurally identical step at `m = #t₁`, `p = 1`, `n = k`. T1 (LexicographicOrder) — postcondition (a) invokes T1 case (i) at divergence position `sig(t)` for `k = 0`, and T1 case (ii) with the proper-prefix condition `#t + 1 ≤ #t'` for `k > 0`. TA5-SIG (LastSignificantPosition) — the symbol `sig(t)` in the definition, postcondition (b)'s exclusion quantifier, and postcondition (c)'s increment target all resolve against this definition.
- *Postconditions:* `t' ∈ T`. (a) `t' > t` under T1. (b) When `k = 0`: `(A i : 1 ≤ i ≤ #t ∧ i ≠ sig(t) : t'ᵢ = tᵢ)`. When `k > 0`: `(A i : 1 ≤ i ≤ #t : t'ᵢ = tᵢ)`. (c) When `k = 0`: `#t' = #t`, `t'_{sig(t)} = t_{sig(t)} + 1`. (d) When `k > 0`: `#t' = #t + k`, positions `#t + 1 ... #t + k - 1` are `0`, position `#t + k` is `1`.

Gregory's analysis reveals a critical distinction: `inc(t, 0)` does NOT produce the immediate successor of `t` in the total order. In the general case, it produces the smallest same-length tumbler that agrees with `t` on positions `1, ..., sig(t) − 1` and has a strictly larger component at position `sig(t)`. When `sig(t) = #t` — which holds for valid addresses by TA5-SigValid — this is the smallest same-length tumbler strictly greater than `t`: the *next peer* at the same hierarchical depth. When `sig(t) < #t` (i.e., trailing zeros exist beyond the rightmost nonzero component), the gap between `t` and `inc(t, 0)` contains same-length tumblers as well — for example, `(2, 0, 0)` and `inc((2, 0, 0), 0) = (3, 0, 0)` have `(2, 0, 1)` strictly between them. The gap between `t` and `inc(t, 0)` contains the entire subtree of `t`: all tumblers of the form `t.x₁. ... .xₘ` for any `m ≥ 1` and any `x₁ ≥ 0`. The true immediate successor in the total order is `t.0` — the zero-extension — by the prefix convention (T1 case (ii)). For any `k > 0`, `inc(t, k)` does NOT produce the immediate successor of `t` in the total order. For `k = 1` the result is `t.1`; for `k = 2` the result is `t.0.1`. In both cases, `t.0` (the true immediate successor) lies strictly between `t` and the result. The gap between `t` and `inc(t, k)` contains `t`'s entire subtree of zero-extensions. For address allocation, the distinction is harmless: allocation cares about advancing the counter past all existing addresses, not about visiting every point in the total order.

The conditions under which `inc` preserves the structural validity constraint T4 — including the boundary on `k` and the role of `zeros(t)` — are established in TA5a (IncrementPreservesT4).

| Label | Statement | Status |
|-------|-----------|--------|
| TA5 | `inc(t, k)` produces `t' > t` with same-length structure for `k = 0` (sibling) and extended structure for `k > 0` (child) | proved (this property) |
| TA5-SIG | `sig(t)` is the rightmost nonzero component position of `t`, or `#t` when all components are zero | definition (separate property) |
| TA5-SigValid | For every valid address satisfying T4, `sig(t) = #t` | proved (separate property) |
| TA5a | `inc(t, k)` preserves T4 iff `k = 0`, or `k = 1 ∧ zeros(t) ≤ 3`, or `k = 2 ∧ zeros(t) ≤ 2`; violated for `k ≥ 3` | proved (separate property) |

---

## TA5a — IncrementPreservesT4

The increment operation inc(t, k) produces a valid address (satisfying T4) if and only if k = 0, or k = 1 with at most three existing zero components, or k = 2 with at most two existing zero components; for k ≥ 3 the result always violates T4 because the appended separator zeros create an adjacent-zero or empty-field violation. This gives the precise depth limit for child allocation from any given valid address.

*Formal Contract:*
- *Precondition:* `t` satisfies T4 (valid address tumbler), `k ≥ 0`. `zeros(t) = #{i : 1 ≤ i ≤ #t ∧ tᵢ = 0}` (T4).
- *Guarantee:* `inc(t, k)` satisfies T4 iff `k = 0`, or `k = 1 ∧ zeros(t) ≤ 3`, or `k = 2 ∧ zeros(t) ≤ 2`.
- *Failure:* For `k ≥ 3`, `inc(t, k)` violates T4 (adjacent zeros create an empty field).

---

## TA6 — ZeroTumblers

No all-zero tumbler is a valid address — every zero tumbler is excluded by the boundary rule T4 enforces on first components. As a sentinel benefit, every zero tumbler is strictly less than every positive tumbler under the lexicographic order, making them safe lower-bound markers for uninitialized values and unbounded span endpoints.

*Formal Contract:*
- *Depends:* T0 (CarrierSetDefinition) — Conjunct 1 uses T0's guarantee `#t ≥ 1` for all `t ∈ T` to establish that `t₁` is defined, and Conjunct 2 uses T0's constraint `aᵢ ∈ ℕ` to equate the hypothesis form `tⱼ > 0` with `tⱼ ≠ 0`, matching the TA-Pos definition of `Pos(t)`. T1 (LexicographicOrder) — invoked transitively through TA-Pos's postcondition for Conjunct 2; TA-Pos's proof uses T1 case (i) at the first positive position of `t` and T1 case (ii) when `s` is a proper prefix of `t`, and Conjunct 2 inherits this case analysis without repeating it. T4 (HierarchicalParsing) — Conjunct 1 relies on T4's boundary clause `t₁ ≠ 0` to reject zero tumblers as valid addresses. TA-Pos (PositiveTumbler) — Conjunct 2 cites TA-Pos's postcondition `(A t ∈ T, z ∈ T : Pos(t) ∧ (A i : 1 ≤ i ≤ #z : zᵢ = 0) :: z < t)` as the canonical statement of the zero-tumbler-below-positive-tumbler relation; the conjunct is now a restatement of that postcondition, with operand names swapped (TA6's `s` plays the role of TA-Pos's `z`, and TA6's `t` coincides with TA-Pos's `t`), rather than an independent reproof, which prevents drift between the two statements.
- *Postconditions:* (a) `(A t ∈ T : (A i : 1 ≤ i ≤ #t : tᵢ = 0) ⟹ t is not a valid address)`. (b) `(A s, t ∈ T : (A i : 1 ≤ i ≤ #s : sᵢ = 0) ∧ (E j : 1 ≤ j ≤ #t : tⱼ > 0) ⟹ s < t)` — a restatement of TA-Pos's postcondition with operand names swapped; TA-Pos is the canonical site, and any tightening of this statement must be reflected there.

Zero tumblers serve as *sentinels*: they mark uninitialized values, denote "unbounded" when used as span endpoints, and act as lower bounds.

---

## TA7a — SubspaceClosure

Defines the subspace S as the set of tumblers with all positive components, and characterizes exactly when ⊕ and ⊖ preserve membership in S. Addition stays in S when all tail components of the displacement are positive; subtraction stays in S in several cases but can escape to T \ S when the action point is 1 and the leading components agree, or when operand lengths differ.

*Formal Contract:*
- *Preconditions:* For `⊕`: `o ∈ S`, `w ∈ T`, `Pos(w)`, `actionPoint(w) ≤ #o`. For `⊖`: `o ∈ S`, `w ∈ T`, `Pos(w)`, `o ≥ w`.
- *Postconditions:* `o ⊕ w ∈ T`, `#(o ⊕ w) = #w`. `o ⊖ w ∈ T`. For `⊕`, the result is in S when all tail components of `w` (after the action point) are positive. For `⊖` with `actionPoint(w) ≥ 2` and `#w ≤ #o`: the divergence falls at position 1, TumblerSub produces `o` itself (a no-op), and the result is in S. For `⊖` with `actionPoint(w) = 1` and divergence at position `d = 1` (i.e., `o₁ ≠ w₁`): `r₁ = o₁ - w₁ > 0` and `rᵢ = oᵢ > 0` for `i > 1`, so the result is in S when `#w ≤ #o`. For `⊖` with `actionPoint(w) = 1` and divergence at position `d > 1` (i.e., `o₁ = w₁`): the result has `r₁ = 0` and lies in `T \ S` (counterexample: `[5, 3] ⊖ [5, 1] = [0, 2]`). For `⊖` when `#w > #o`: the result inherits trailing zeros at positions `#o + 1` through `#w` and lies in `T \ S`. For `⊖` on single-component ordinals (`#o = 1`, `#w = 1`): the result is in `S ∪ Z`: `[x] ⊖ [n] ∈ S` when `x > n`, and `[x] ⊖ [n] ∈ Z` when `x = n`.
- *Frame:* The subspace identifier `N`, held as structural context, is not an operand and is never modified by either operation.
- *Definition:* **S** = {o ∈ T : #o ≥ 1 ∧ (A i : 1 ≤ i ≤ #o : oᵢ > 0)} — ordinals with all positive components, matching the shape of an element field under T4 (a non-zero run delimited by separators; positivity of non-zero entries is supplied by T0's carrier ℕ).

---

## TS1 — ShiftOrderPreservation

Shift is order-preserving on equal-length tumblers: if v₁ < v₂ and both have length m, shifting both by the same positive amount n yields shift(v₁, n) < shift(v₂, n). The relative ordering of same-length tumblers is invariant under shift.

*Formal Contract:*
- *Preconditions:* v₁ ∈ T, v₂ ∈ T, n ≥ 1, #v₁ = #v₂ = m, v₁ < v₂
- *Postconditions:* shift(v₁, n) < shift(v₂, n)

---

## TS2 — ShiftInjectivity

Shift is injective over same-length tumblers: if shifting v₁ and v₂ by the same positive amount n yields identical results, then v₁ and v₂ must have been equal. This rules out any collisions introduced by the shift operation.

*Formal Contract:*
- *Preconditions:* v₁ ∈ T, v₂ ∈ T, n ≥ 1, #v₁ = #v₂ = m
- *Postconditions:* shift(v₁, n) = shift(v₂, n) ⟹ v₁ = v₂

---

## TS3 — ShiftComposition

Two successive shifts compose into a single shift: applying shift by n₁ and then by n₂ is identical to a single shift by n₁ + n₂. Shift preserves tumbler length throughout.

*Formal Contract:*
- *Preconditions:* v ∈ T, n₁ ≥ 1, n₂ ≥ 1, #v = m
- *Postconditions:* shift(shift(v, n₁), n₂) = shift(v, n₁ + n₂)
- *Frame:* #shift(shift(v, n₁), n₂) = #v = m (shift preserves tumbler length)

---

## TS4 — ShiftStrictIncrease

Shifting a tumbler by any positive amount strictly increases it — the shifted result is always greater than the original under the lexicographic order. This makes shift a strict advance with no fixed points.

*Formal Contract:*
- *Preconditions:* v ∈ T, n ≥ 1, #v = m
- *Postconditions:* shift(v, n) > v

---

## TS5 — ShiftAmountMonotonicity

Larger shift amounts produce strictly larger results on the same tumbler: if n₂ > n₁ ≥ 1, then shift(v, n₂) > shift(v, n₁). The output of shift is strictly monotone in the shift amount.

*Formal Contract:*
- *Preconditions:* v ∈ T, n₁ ≥ 1, n₂ > n₁, #v = m
- *Postconditions:* shift(v, n₁) < shift(v, n₂)

---

## TumblerAdd — TumblerAdd

Defines tumbler addition (⊕) as a hierarchical position-advance operation: below the action point components are copied from the base address, at the action point the components are summed, and above the action point components are taken from the displacement. The result always strictly exceeds the base address and is at least as large as the displacement itself.

*Formal Contract:*
- *Preconditions:* a ∈ T, w ∈ T, Pos(w) (TA-Pos), actionPoint(w) ≤ #a (ActionPoint)
- *Definition:* k = actionPoint(w) (ActionPoint); rᵢ = aᵢ if i < k; rₖ = aₖ + wₖ; rᵢ = wᵢ if i > k
- *Depends:* T0 (CarrierSetDefinition) — membership `a ⊕ w ∈ T` is concluded via T0's characterisation of T as finite sequences over ℕ with length ≥ 1; the strict-advancement chain `rₖ = aₖ + wₖ ≥ aₖ + 1 > aₖ` invokes T0's order-compatibility of `+` (from `wₖ ≥ 1` infer `aₖ + wₖ ≥ aₖ + 1`) and T0's strict successor inequality (`aₖ + 1 > aₖ`); the dominance proof's equality sub-case (`aₖ = 0 ⟹ rₖ = wₖ`) invokes T0's additive identity (`0 + wₖ = wₖ`); the dominance proof's divergence sub-case step "the least such `j` is a divergence point" invokes T0's well-ordering of ℕ applied to the nonempty subset `{j : 1 ≤ j < k ∧ aⱼ > 0} ⊆ ℕ` — nonempty by the sub-case hypothesis that some `aⱼ > 0` for `j < k`, a subset of ℕ because `j` ranges over natural-number indices `1 ≤ j < k` — to supply the least element named by "the least such `j`", a use of well-ordering independent of the well-ordering propagated into TumblerAdd through `actionPoint(w)` (which `ActionPoint` discharges internally over the set `{i : 1 ≤ i ≤ #w ∧ wᵢ ≠ 0}`), so the "least such `j`" claim is discharged from T0 rather than left implicit, matching the per-step citation convention established for `TA-Pos` and `ActionPoint`. ActionPoint (ActionPoint) — supplies `k = actionPoint(w)` with bounds `1 ≤ k ≤ n`, the zeros-below-action-point fact `wᵢ = 0 for i < actionPoint(w)`, and the minimum-nonzero value `wₖ ≥ 1`; the dominance proof `a ⊕ w ≥ w` uses the zeros-below-k fact to license both the divergence case (`wⱼ = 0 < aⱼ = rⱼ` at the least `j < k` with `aⱼ > 0`) and the equality sub-case (`rᵢ = aᵢ = 0 = wᵢ` for `i < k` when `aᵢ = 0` throughout). TA-Pos (PositiveTumbler) — defines the predicate `Pos(w)` used in the precondition. T1 (LexicographicOrder) — both ordering postconditions invoke T1 case (i) at the first divergence position. T3 (CanonicalRepresentation) — the equality sub-case of `a ⊕ w ≥ w` concludes `r = w` from component-wise agreement and equal length.
- *Postconditions:* a ⊕ w ∈ T, #(a ⊕ w) = #w, a ⊕ w > a (T1), a ⊕ w ≥ w (T1, T3)

---

## TumblerSub — TumblerSub

Defines component-wise tumbler subtraction `a ⊖ w`, producing a result whose nonzero components begin at the first zero-padded divergence point and whose length is `max(#a, #w)`. The result captures the positional difference from `w` to `a`, but the round-trip `a ⊕ (b ⊖ a) = b` holds only when two independent conditions are met: `divergence(a, b) ≤ #a` (so TumblerAdd's precondition is satisfied) and `#a ≤ #b` (so the result length matches `#b`).

*Formal Contract:*
- *Preconditions:* a ∈ T, w ∈ T, a ≥ w (T1). Consequence: when zpd(a, w) is defined, aₖ > wₖ at k = zpd(a, w).
- *Depends:* T0 (CarrierSetDefinition) — membership `a ⊖ w ∈ T` is concluded via T0's characterisation of T as finite sequences over ℕ with length ≥ 1. T1 (LexicographicOrder) — the precondition `a ≥ w` is a T1 ordering; the precondition consequence derives `w < a` from `a ≥ w ∧ a ≠ w` via T1 trichotomy. Divergence (Divergence) — the precondition consequence proceeds by case analysis on Divergence's two cases for the pair `(w, a)`. ZPD (ZPD) — defines `zpd(a, w)` and supplies the ZPD–Divergence relationship identifying `zpd(a, w) = divergence(a, w) = k` in case (i). TA-Pos (PositiveTumbler) — defines `Pos` for the conditional postcondition `Pos(a ⊖ w)`. ActionPoint (ActionPoint) — supplies the action-point identification for the conditional postcondition `actionPoint(a ⊖ w) = zpd(a, w)`.
- *Definition:* a ⊖ w computed by case analysis on k = zpd(a, w) (ZPD), all component references using zero-padded values (aᵢ = 0 for i > #a, wᵢ = 0 for i > #w); rᵢ = 0 for i < k, rₖ = aₖ − wₖ, rᵢ = aᵢ (zero-padded) for i > k; when zpd(a, w) is undefined, a ⊖ w = [0, …, 0]; #(a ⊖ w) = max(#a, #w)
- *Postconditions:* a ⊖ w ∈ T, #(a ⊖ w) = max(#a, #w); when zpd(a, w) is defined: Pos(a ⊖ w) (TA-Pos), actionPoint(a ⊖ w) = zpd(a, w) (ActionPoint)

---

## ZPD — ZeroPaddedDivergence

Defines the zero-padded divergence `zpd(a, w)` as the first position at which two tumblers disagree after both are extended to length `max(#a, #w)` by appending zeros. The function is partial: it is undefined when the padded sequences agree everywhere, which occurs when one operand is a prefix of the other with all trailing components zero — a case where Divergence fires but zpd does not. When defined, zpd is symmetric and equals the ordinary divergence index whenever the disagreement falls within the shared length of both operands.

*Formal Contract:*
- *Domain:* a ∈ T, w ∈ T
- *Definition:* Pad to length L = max(#a, #w): aᵢ = 0 for i > #a, wᵢ = 0 for i > #w. If (A i : 1 ≤ i ≤ L : aᵢ = wᵢ), zpd(a, w) is undefined. Otherwise, zpd(a, w) = min {k : 1 ≤ k ≤ L ∧ aₖ ≠ wₖ}.
- *Depends:* Divergence (Divergence) — the Relationship to Divergence postconditions consume Divergence's two-case structure (case (i): component divergence at `k ≤ min(#a, #w)`, case (ii): prefix divergence at `min(#a, #w) + 1`) and its domain restriction `a ≠ b`, which gates the postcondition's guard `a ≠ w`.
- *Codomain:* When defined, zpd(a, w) ∈ {1, ..., max(#a, #w)}.
- *Partiality:* zpd(a, w) is undefined iff a and w are zero-padded-equal.
- *Postconditions (Symmetry):* `zpd(a, w)` is defined iff `zpd(w, a)` is defined, and when defined, `zpd(a, w) = zpd(w, a)`. Padding both operands to `max(#a, #w)` and scanning for the first disagreement are symmetric operations: `aₖ ≠ wₖ` iff `wₖ ≠ aₖ`.
- *Postconditions (Relationship to Divergence):* For `a ≠ w`: in Divergence case (i) — component divergence at `k ≤ min(#a, #w)` — both operands possess actual values at positions `1, ..., k`, so no padding is consulted and `zpd(a, w) = divergence(a, w)`; in Divergence case (ii) — one operand is a proper prefix of the other — if the longer operand has a nonzero component beyond position `min(#a, #w)`, then `zpd(a, w)` is defined and `zpd(a, w) ≥ divergence(a, w)`, and if all trailing components of the longer operand are zero, then `zpd(a, w)` is undefined (the operands are zero-padded-equal despite being distinct).

---
