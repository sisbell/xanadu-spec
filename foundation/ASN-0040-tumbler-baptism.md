> **ASN-0040 · Tumbler Baptism** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md)  
> [Condensed statements →](ASN-0040-tumbler-baptism.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0040: Tumbler Baptism

*2026-03-15*

We seek to understand what it means for a position to enter the tumbler hierarchy. The algebra (ASN-0034) gives us an infinite space of well-formed addresses — ordered by T1, structured into fields by T4, permanently allocated by T8, strictly increasing by T9. But the algebra cannot distinguish between a position that *has been assigned* and one that merely *could be*. Something marks the transition from arithmetic possibility to system fact.

Nelson calls this transition *baptism*:

> "Whoever owns a specific node, account, document or version may in turn designate (respectively) new nodes, accounts, documents and versions, by forking their integers. We often call this the 'baptism' of new numbers."

Three observations are compressed into that sentence. Baptism is *hierarchical* — it descends level by level through the field structure. Baptism is *sequential* — positions arrive in order, not arbitrarily. And baptism is *permanent* — "Any address, once assigned, remains valid forever." Authorization (who may baptize) is out of scope here; we characterize the structural mechanism: how the set of baptized positions grows, and what it preserves as it grows.

Gregory's implementation locates the moment of baptism precisely: an address becomes real only when it is committed to the persistent store. A candidate computed but never committed does not exist. We therefore take commitment — the entry of an address into the registry — as the defining event of baptism.

We formalize baptism as the growth law of the address space.


## State space and transitions

We work within the foundation's transition framework (ASN-0034, AllocatedSet and NoDeallocation): the *state space* `𝒮`, a closed *transition vocabulary* `Σ` of partial operations, and reachability `→*` as the reflexive-transitive closure of the induced transition relation. A *state* is `s`. Obligations of the form `(A s, s' : s → s' : …)` constrain every admissible transition; `(A s : s reachable from s_init : I(s))` is a state invariant.

The initial state s_init has s_init.B = B₀, the seed set established at genesis; "reachable" without qualification means reachable from s_init.

The execution discipline under a single baptismal authority we make explicit:

**B-Seq (Sequential Commitment).** Within a single baptismal authority, the states actually realized by execution lie on one transition path from s_init: the visited states are totally ordered by `→*`.

*Justification.* Grounded in implementation: Gregory's udanax-green commits baptisms through a single serialized path, so the realized history is a strict linear sequence.

*Formal Contract:*
- *Axiom:* The states realized under a single baptismal authority — one serialized commit path — are totally ordered by `→*`: for any two such reachable states s, s', either s →* s' or s' →* s.


## The baptismal registry

We introduce the central state component:

**s.B (BaptismalRegistry).** s.B ⊆ T — the set of baptized tumblers.

A tumbler t is *baptized* iff t ∈ s.B. Initially s.B contains a finite seed set B₀ ⊆ T of root addresses established at system genesis. Thereafter it grows monotonically.

s.B is the *committed registry* of baptized positions, distinct from the foundation's `allocated(s)` (AllocatedSet, ASN-0034), which is the allocator's *realized domain*.

**B0a (Baptismal Closure).** Σ partitions into two classes whose treatment of the s.B component is fixed:

  - *Baptismal operations.* For each (p, d) satisfying B6 (Valid Depth, below), `baptize(p, d) ∈ Σ` adjoins a single element to the registry: `op(s).B = s.B ∪ {next(s.B, p, d)}`.
  - *s.B-frame operations.* Every other `op ∈ Σ` preserves the registry: `(A op ∈ Σ \ {baptize(p, d) : B6(p, d)}, s ∈ dom(op) : op(s).B = s.B)`.

We call it the *s.B-frame dispatch*.

**B0a-frame (Frame Preservation — corollary of B0a).** Any state invariant `I(s)` that is a predicate on the registry component alone — `I(s) ≡ φ(s.B)` for some `φ` — is preserved by every s.B-frame transition: if `op` is an s.B-frame operation then `op(s).B = s.B` (B0a), so `φ(op(s).B) = φ(s.B)`.

Irrevocability follows immediately:

**B0 (Irrevocability — corollary of B0a).** `(A s, s' : s → s' : s.B ⊆ s'.B)`. In the baptismal branch Bop adjoins a single element, giving `s.B ⊆ op(s).B`; in the s.B-frame branch B0a gives `op(s).B = s.B`. So `s.B ⊆ op(s).B` in both, hence `s.B ⊆ s'.B` for every transition. Nelson: "New items may be continually inserted in tumbler-space while the other addresses remain valid."

B0 is a single-step law. We extend it to finite transition sequences:

**B0★ (Multi-step Irrevocability — corollary of B0).** `(A s, s' : s →* s' : s.B ⊆ s'.B)`, where s →* s' denotes the reflexive-transitive closure of the transition relation — that is, s' is reachable from s by a finite (possibly empty) sequence of transitions.

*Proof.* By induction on the length of the witnessing transition sequence. *Base* (empty path, s = s'): `s.B ⊆ s.B` by reflexivity of ⊆. *Step:* given `s → s₁ →* s'`, B0 gives `s.B ⊆ s₁.B` for the single step and the inductive hypothesis gives `s₁.B ⊆ s'.B`; transitivity of ⊆ composes them to `s.B ⊆ s'.B`. ∎


## The sibling stream

Consider a parent address p ∈ T and a baptismal depth d ≥ 1. From TA5, `inc(p, d)` produces a tumbler strictly greater than p that extends p by d components: d − 1 zero separators followed by 1. This is the *first child* of p at depth d. Repeated sibling increments yield a counting sequence:

  c₁ = inc(p, d)

  cₙ₊₁ = inc(cₙ, 0)    for n ≥ 1

**S(p,d) (SiblingStream).** We call the sequence c₁, c₂, c₃, ... the *sibling stream* of p at depth d, written S(p, d). By TA5(c), each sibling increment preserves the tumbler's length and advances only the last significant component by 1. Every element of S(p, d) has the form [p₁, ..., p_{#p}, 0, ..., 0, n] — the parent's components, then d − 1 zeros, then the ordinal n. We establish this canonical form and the uniform length #cₙ = #p + d by induction:

*Proof.* By induction on n.

*Base case (n = 1).* c₁ = inc(p, d) with d ≥ 1. By TA5(d) (ASN-0034), c₁ has length #p + d: the first #p components are preserved from p (TA5(b)), the next d − 1 positions #p + 1 through #p + d − 1 are zero-valued field separators, and the final position #p + d has value 1. This is exactly [p₁, ..., p_{#p}, 0, ..., 0, 1] with d − 1 zeros and ordinal 1.

*Inductive step.* Assume cₙ = [p₁, ..., p_{#p}, 0, ..., 0, n] with d − 1 zeros and #cₙ = #p + d for some n ≥ 1. Since n ≥ 1, position #p + d holds value n > 0, so sig(cₙ) = #p + d — the ordinal position is the last significant component. Consider cₙ₊₁ = inc(cₙ, 0). By TA5(c), cₙ₊₁ has the same length as cₙ (#cₙ₊₁ = #p + d) and differs from cₙ only at position sig(cₙ) = #p + d, where cₙ₊₁ at that position equals n + 1. All other positions are unchanged: the first #p components remain p₁, ..., p_{#p} (since every position i ≤ #p satisfies i < sig(cₙ) = #p + d), and the d − 1 zeros at positions #p + 1 through #p + d − 1 remain zero (since each such position j satisfies j < #p + d = sig(cₙ)). Therefore cₙ₊₁ = [p₁, ..., p_{#p}, 0, ..., 0, n + 1], the claimed form with ordinal n + 1. ∎

*Formal Contract:*
- *Definition:* S(p, d) = c₁, c₂, c₃, ... where c₁ = inc(p, d) and cₙ₊₁ = inc(cₙ, 0) for n ≥ 1.
- *Preconditions:* p ∈ T, d ≥ 1.
- *Postconditions:* `(A n ≥ 1 : cₙ = [p₁, ..., p_{#p}, 0, ..., 0, n])` with d − 1 zeros, `#cₙ = #p + d`, `sig(cₙ) = #p + d`, and `cₙᵢ = pᵢ` for `1 ≤ i ≤ #p`.
- *Depends:* TA5(b) (prefix preservation), TA5(c) (sibling structure), TA5(d) (child structure).

Every element of S(p, d) shares the invariant prefix [p₁, ..., p_{#p}, 0, ..., 0] across positions 1 through #p + d − 1, varying only in the ordinal at the last position #p + d.

**S0 (StreamOrdering).** `(A i, j : 1 ≤ i < j : cᵢ < cⱼ)`.

*Proof.* By TA5(a), each sibling increment strictly advances its argument: cₙ₊₁ = inc(cₙ, 0) > cₙ. T1 transitivity lifts this per-step increase across arbitrary gaps, and T1 irreflexivity keeps the inequalities strict, giving `(A i, j : 1 ≤ i < j : cᵢ < cⱼ)`. ∎

*Formal Contract:*
- *Preconditions:* p ∈ T, d ≥ 1. S(p, d) = c₁, c₂, ... defined by c₁ = inc(p, d), cₙ₊₁ = inc(cₙ, 0).
- *Postconditions:* `(A i, j : 1 ≤ i < j : cᵢ < cⱼ)` — the sibling stream is strictly increasing.
- *Depends:* TA5(a)/T1 (the per-step strict increase and its transitive/irreflexive lifting); TA5(c)/TA5(d) (well-formedness of the operands cₙ ∈ T).

**S1 (StreamPrefix).** `(A n : n ≥ 1 : p ≼ cₙ)` — every stream element extends p as a prefix.

*Proof.* `p ≼ cₙ` is immediate from S(p,d)'s postconditions `#cₙ = #p + d ≥ #p` (since d ≥ 1) and `cₙᵢ = pᵢ` for `1 ≤ i ≤ #p`, by the Prefix definition (ASN-0034). ∎

*Formal Contract:*
- *Relation:* `≼` is the foundation Prefix relation (Prefix, ASN-0034).
- *Preconditions:* p ∈ T, d ≥ 1. S(p, d) = c₁, c₂, ... defined by c₁ = inc(p, d), cₙ₊₁ = inc(cₙ, 0).
- *Postconditions:* `(A n : n ≥ 1 : p ≼ cₙ)` — every stream element extends p as a prefix.

Nelson describes exactly this process: "One digit can become several by a forking or branching process. This consists of creating successive new digits to the right."


## Depth and field structure

Baptism interacts with the field hierarchy through the depth parameter. Recall from ASN-0034 that zeros(t) — the count of zero-valued components — determines the hierarchical level: 0 for node, 1 for user, 2 for document, 3 for element. When baptism crosses from one level to the next, it must introduce a new zero separator.

**B5 (Field Advancement).** `zeros(inc(p, d)) = zeros(p) + (d − 1)`.

For d = 1: zeros is preserved — the child is at the same hierarchical level. For d = 2: zeros advances by 1 — the child descends one level, the increment emitting one separator zero (TA5(d)'s lone intermediate zero) followed by the first child's ordinal 1.

*Proof.* Let t' = inc(p, d). Since d ≥ 1, TA5(d) applies: t' has length #p + d, with the first #p components preserved from p (TA5(b)), d − 1 zero-valued components at positions #p + 1 through #p + d − 1, and a final component of value 1 at position #p + d.

We partition the components of t' into three ranges and count zeros in each. Positions 1 through #p are identical to the corresponding components of p by TA5(b), contributing exactly zeros(p) zero-valued components. Positions #p + 1 through #p + d − 1 are the field separators introduced by the increment — there are d − 1 of them, each zero-valued, contributing d − 1 zeros. (When d = 1 this range is empty, contributing none; when d = 2 it contains exactly one zero.) Position #p + d holds value 1, contributing no zeros.

Since these three ranges exhaust all #p + d positions of t', the total zero count is zeros(t') = zeros(p) + (d − 1) + 0 = zeros(p) + (d − 1). ∎

*Formal Contract:*
- *Preconditions:* p ∈ T with d ≥ 1.
- *Postconditions:* `zeros(inc(p, d)) = zeros(p) + (d − 1)`.

B5 establishes the zeros count for the *first* child c₁ of a stream. The sibling stream preserves it:

**B5a (Sibling Zeros Preservation).** `(A t : t_{sig(t)} > 0 : zeros(inc(t, 0)) = zeros(t))`

*Proof.* Let t' = inc(t, 0). By TA5(c), t' has the same length as t (#t' = #t) and differs from t only at position sig(t), where t'_{sig(t)} = t_{sig(t)} + 1. At every other position, t'_i = t_i.

We count zeros in t' by comparing each component with the corresponding component of t. At every position i ≠ sig(t), t'_i = t_i, so position i is zero-valued in t' exactly when it is zero-valued in t — these positions contribute identically to both zeros(t') and zeros(t). At position sig(t), the precondition gives t_{sig(t)} > 0, so this position contributes no zero to zeros(t). After the increment, t'_{sig(t)} = t_{sig(t)} + 1 ≥ 2 > 0, so this position contributes no zero to zeros(t') either. Since every position contributes identically to both zero counts, zeros(t') = zeros(t). ∎

*Formal Contract:*
- *Preconditions:* t ∈ T with t_{sig(t)} > 0.
- *Postconditions:* `zeros(inc(t, 0)) = zeros(t)`.

To apply B5a across the sibling stream S(p, d), we discharge its precondition: every cₙ satisfies cₙ_{sig(cₙ)} > 0. By S(p, d), sig(cₙ) = #p + d and (cₙ)_{#p+d} = n ≥ 1 > 0, so every stream element satisfies the precondition. Combined with B5, every element of S(p, d) inherits the zeros count established at c₁:

  `(A n ≥ 1 : zeros(cₙ) = zeros(p) + (d − 1))`

All elements in a stream share the same hierarchical level.

**B6 (Valid Depth).** Baptism at depth d from parent p is valid when:

  (i) p satisfies T4,

  (ii) d ∈ {1, 2}, and

  (iii) zeros(p) + (d − 1) ≤ 3.

Given a T4-valid parent (i), conditions (ii) and (iii) are necessary and sufficient for T4 preservation of the sibling stream (proved below). The valid combinations are:

| Parent level | d = 1 (same level) | d = 2 (level crossing) |
|---|---|---|
| Node (zeros = 0) | node child | user child |
| User (zeros = 1) | user child | document child |
| Document (zeros = 2) | sub-document / version | element child |
| Element (zeros = 3) | sub-element | **invalid** |

*Proof.* The theorem is: given a T4-valid parent (i), conditions (ii) and (iii) are necessary and sufficient for stream T4-preservation. We prove sufficiency (all three conditions imply T4 preservation) and then necessity of (ii) and (iii).

**(⟸) Sufficiency.** Assume (i) p satisfies T4, (ii) d ∈ {1, 2}, and (iii) zeros(p) + (d − 1) ≤ 3. We show every element of S(p, d) satisfies T4.

For the first child c₁ = inc(p, d): TA5a (IncrementPreservesT4, ASN-0034) states that for any t satisfying T4, inc(t, k) satisfies T4 iff `k = 0`, or `k = 1 ∧ zeros(t) ≤ 3`, or `k = 2 ∧ zeros(t) ≤ 2`. With t = p and k = d, conditions (i) and (ii) make p T4-valid and put d ∈ {1, 2}. For d = 1, the first child is c₁ = inc(p, 1), i.e. k = 1, which TA5a conditions on zeros(p) ≤ 3; this is discharged by T4-validity of p (T4 permits at most three zeros). For d = 2, TA5a's `k = 2 ∧ zeros(t) ≤ 2` branch requires zeros(p) ≤ 2, which is exactly condition (iii) specialized to d = 2. The TA5a case applicable at the chosen d is therefore satisfied, so c₁ satisfies T4.

For subsequent siblings cₙ₊₁ = inc(cₙ, 0): TA5a's `k = 0` case states that inc(t, 0) satisfies T4 for any T4-valid t with no further constraint — sibling increment modifies only position sig(t), advancing a positive value by one (TA5(c)), so no zeros are added and no new adjacencies are introduced. Since c₁ satisfies T4, and each sibling increment preserves T4, by induction every cₙ satisfies T4.

**(⟹) Necessity (of (ii) and (iii), given (i)).** Violating condition (ii) or (iii) produces a T4 violation in the stream.

*Condition (ii) is necessary for T4.* Let d ≥ 3. The first child c₁ = inc(p, d) is an increment at k = d ≥ 3, and TA5a (IncrementPreservesT4, ASN-0034) states that inc(t, k) violates T4 for every k ≥ 3 — the same foundation clause the sufficiency direction invokes for the k = d case. No choice of p avoids this: TA5a's k ≥ 3 failure is independent of p's content.

*Condition (iii) is necessary at d = 2.* At d = 1, condition (iii) reduces to zeros(p) ≤ 3, which is already discharged by T4-validity of p (condition (i)) — so at d = 1 it imposes no additional constraint, and the d = 2 case is the only binding necessity claim for (iii). Let zeros(p) + (d − 1) > 3 with p satisfying T4. By B5, zeros(c₁) = zeros(p) + (d − 1) > 3. But T4 requires zeros(t) ≤ 3 for any valid address — at most three field separators for the four-level hierarchy. The first child already exceeds the zero budget, so c₁ violates T4. ∎

*Formal Contract:*
- *Preconditions:* p ∈ T, d ∈ ℕ with d ≥ 1.
- *Postconditions:* (a) Sufficiency: `(p satisfies T4 ∧ d ∈ {1, 2} ∧ zeros(p) + (d − 1) ≤ 3) ⟹ (A n ≥ 1 : cₙ ∈ S(p, d) satisfies T4)`. (b) Necessity, given a T4-valid parent (i): violating (ii) or (iii) forces a stream T4 violation.


## Namespace disjointness

Each parent-depth pair defines a namespace. Distinct namespaces must produce non-overlapping address ranges, or global uniqueness collapses.

**B7 (Namespace Disjointness).** For distinct valid pairs (p, d) ≠ (p', d'):

  S(p, d) ∩ S(p', d') = ∅

provided both `(p, d)` and `(p', d')` satisfy B6.

*Proof.* Every element of S(p, d) has the canonical form `[p₁, …, p_{#p}, 0, …, 0, n]` (length #p + d, with d − 1 separating zeros, ordinal n at the last position), and sibling increments fix every position except the last (TA5(c)). So all elements of S(p, d) share the invariant prefix `[p₁, …, p_{#p}, 0, …, 0]` on positions 1 through #p + d − 1, and symmetrically all of S(p', d') share their own invariant prefix on positions 1 through #p' + d' − 1. To establish disjointness it suffices to exhibit a *fixed* position — one at or below #p + d − 1 in S(p, d) and at or below #p' + d' − 1 in S(p', d') — at which the two invariant prefixes carry different values: any such disagreement makes every element of one stream differ from every element of the other (T3). We argue by cases on the base lengths #p + d and #p' + d'.

*Length split (unequal base length, #p + d ≠ #p' + d').* Every element of S(p, d) has length #p + d and every element of S(p', d') has length #p' + d'. With unequal lengths, no element of one can equal any element of the other (T3, CanonicalRepresentation: equal tumblers have equal length), so the streams are disjoint.

*Equal base length (#p + d = #p' + d').* We examine the two ways the parents' lengths can relate.

*Equal-length parents (#p = #p').* Then d = d', and since (p, d) ≠ (p', d') we have p ≠ p'. By T3 the parents differ at some position j with 1 ≤ j ≤ #p, so pⱼ ≠ p'ⱼ. Since d ≥ 1, j ≤ #p ≤ #p + d − 1, so position j lies in the invariant prefix of both streams (and equally below #p' + d' − 1, as #p' = #p). Every element of S(p, d) carries pⱼ at position j and every element of S(p', d') carries p'ⱼ ≠ pⱼ there, so the streams disagree at the fixed position j and are disjoint.

*Unequal-length parents (#p ≠ #p').* With d, d' ∈ {1, 2} (B6(ii)) and #p + d = #p' + d', the parent lengths differ by at most 1, so WLOG #p' = #p + 1, d = 2, d' = 1. Consider the fixed position #p + 1. In S(p, 2) it is the lone separating zero, value 0, and #p + 1 = #p + 2 − 1 places it at the last invariant-prefix position. In S(p', 1) there are no separating zeros and positions 1 … #p' = 1 … #p + 1 are preserved from p', so position #p + 1 equals p'_{#p'}, the last component of p'; this is the last invariant-prefix position there too, since #p + 1 = #p' = (#p' + 1) − 1. Since (p', d') satisfies B6, p' satisfies T4, whose field-segment constraint forbids a zero final component (T4: p'_{#p'} ≠ 0). The two streams thus carry 0 and a nonzero value at the fixed position #p + 1, so they disagree there and are disjoint.

In every case the two streams disagree at a fixed position (or differ in length outright), so `S(p, d) ∩ S(p', d') = ∅`. ∎

B6(i) is load-bearing: dropping it admits aliasing. A pure-trailing-zero parent and its truncation at the next depth produce the identical base, hence the identical stream — e.g. ([1, 0], 1) and ([1], 2) both yield base [1, 0, 1] and stream {[1, 0, n] : n ≥ 1}.

*Formal Contract:*
- *Preconditions:* (p, d) and (p', d') both satisfy B6, with (p, d) ≠ (p', d').
- *Postconditions:* `S(p, d) ∩ S(p', d') = ∅`.
- *Depends:* S(p, d) postconditions (canonical form [p, 0^{d−1}, n], uniform length #p + d, and the invariant prefix across positions 1 … #p + d − 1), TA5(c) (sibling increments fix every position but the last, so the invariant prefix is shared across the stream), B6 (T4-validity and d ∈ {1, 2} of both parents), TA5(d) (base-address length and component structure underlying the canonical form), T3 (CanonicalRepresentation — unequal lengths give distinct tumblers, driving the length split; disagreement at a fixed position gives distinct tumblers, closing the equal-length cases), T4 (valid parent has a nonzero last component — field-segment constraint t_{#t} ≠ 0 — closing the unequal-length-parents case).


## Seed conformance and registry finiteness

**B₀ conf. (SeedConformance).** B₀ is finite, `(A p, d : B6(p, d) : children(B₀, p, d) is a contiguous prefix of S(p, d))`, and `(A t ∈ B₀ : t satisfies T4)`.

B₀ conformance fixes the seed as a finite set; B0a constrains every transition to add at most one element. The composition yields a registry-wide finiteness invariant:

**B_fin (Registry Finiteness).** `(A s : s reachable from s_init : s.B is finite)`.

*Proof.* By induction on the number of state transitions from the initial state.

*Base case.* In the initial state, s.B = B₀. By B₀ conf. (SeedConformance), B₀ is finite. The invariant holds at genesis.

*Inductive step.* Assume s.B is finite for state s with registry B. The s.B-frame case is discharged by B0a-frame (finiteness is a predicate on s.B). In the baptismal case B0a sets B' = B ∪ {a} for a single new element a — a finite set plus a singleton, hence finite. By induction, s.B is finite in every reachable state. ∎

*Formal Contract:*
- *Invariant:* `(A s : s reachable from s_init : s.B is finite)`.
- *Base:* B₀ conf. — B₀ is finite.
- *Preservation:* B0a — every transition either leaves s.B unchanged or replaces it by its union with a single element (a finite set unioned with a singleton is finite).


## Children and the next address

We define the *children* of parent p at depth d in state B:

  children(B, p, d) = B ∩ S(p, d)

— the baptized addresses that belong to the sibling stream. The next address in a namespace is determined by the current registry state:

**next(B,p,d) (NextAddress).**

  next(B, p, d) = if children(B, p, d) = ∅ then inc(p, d) else inc(max(children(B, p, d)), 0)

— find the greatest baptized sibling and produce its immediate successor; if none exists, produce the first child.

*Justification of well-definedness.* When children(B, p, d) = ∅ the result is inc(p, d); otherwise children(B, p, d) is a non-empty finite subset of T (finite by B_fin), which T1 totally orders, so its max exists. Both branches land in T by TA5 (TA5(d) for inc(p, d), TA5(c) for inc(max(...), 0)), so next is total on its domain. ∎

*Formal Contract:*
- *Definition:* next(B, p, d) = if children(B, p, d) = ∅ then inc(p, d) else inc(max(children(B, p, d)), 0), where children(B, p, d) = B ∩ S(p, d).
- *Preconditions:* B ⊆ T finite (discharged by B_fin when B = s.B for a reachable s); p ∈ T; d ≥ 1; S(p, d) defined.
- *Postconditions:* next(B, p, d) ∈ T — the result is a valid tumbler.
- *Depends:* TA5(c) (sibling increment well-definedness), TA5(d) (child increment well-definedness), T1 (total order guarantees max exists).


## The contiguous prefix property

We claim that children(B, p, d) is always a *prefix* of the sibling stream: the first m elements for some m ≥ 0, with no gaps.

**B1 (Contiguous Prefix).** `(A p, d : B6(p, d) : (A n : n ≥ 1 ∧ cₙ ∈ B ⟹ (A i : 1 ≤ i < n : cᵢ ∈ B)))`.

Equivalently: for every B6-valid namespace (p, d), children(B, p, d) = {c₁, ..., cₘ} for some m ≥ 0.

*Proof.* By induction on the number of state transitions from the initial state.

*Base case.* In the initial state, s.B = B₀. By B₀ conf. (SeedConformance), children(B₀, p, d) is a contiguous prefix of S(p, d) for every B6-valid (p, d). B1 holds at genesis.

*Inductive step.* Assume B1 holds for state s with registry B. The s.B-frame case is discharged by B0a-frame (B1 is a predicate on s.B); the baptismal case we now treat.

*Baptismal transition.* By B0a, a baptismal transition sets B' = B ∪ {a} where a = next(B, p₀, d₀) for some (p₀, d₀) satisfying B6. We must show that children(B', p, d) is a contiguous prefix of S(p, d) for every B6-valid (p, d). Two cases exhaust the possibilities.

*Target namespace: (p, d) = (p₀, d₀).* By the inductive hypothesis, children(B, p₀, d₀) = {c₁, ..., cₘ} for some m ≥ 0. Two sub-cases arise from the definition of next (NextAddress).

When m = 0: children(B, p₀, d₀) = ∅, so a = next(B, p₀, d₀) = inc(p₀, d₀) = c₁, the first element of S(p₀, d₀) by the definition of the sibling stream. Therefore children(B', p₀, d₀) = {c₁}, a contiguous prefix of length 1.

When m ≥ 1: the maximum of children(B, p₀, d₀) is cₘ, since the prefix {c₁, ..., cₘ} is strictly ordered by S0 (StreamOrdering). The definition of next gives a = inc(cₘ, 0), which is exactly c_{m+1} by the sibling-stream recurrence. By B0 (Irrevocability), B ⊆ B', so {c₁, ..., cₘ} ⊆ B'. Together with the new element c_{m+1} ∈ B', we obtain children(B', p₀, d₀) = {c₁, ..., cₘ, c_{m+1}}, a contiguous prefix of length m + 1.

*All other B6-valid namespaces: (p, d) ≠ (p₀, d₀) with (p, d) satisfying B6.* Both (p₀, d₀) and (p, d) meet B7's preconditions, so B7 gives S(p₀, d₀) ∩ S(p, d) = ∅, hence a ∉ S(p, d). Therefore children(B', p, d) = children(B, p, d), a contiguous prefix by the inductive hypothesis.

In both the target namespace and every other B6-valid namespace, children(B', p, d) is a contiguous prefix of S(p, d).

Since B1 is preserved in the target namespace and in every other B6-valid namespace, B1 holds for B' under baptismal transitions; the s.B-frame case was discharged by B0a-frame. By induction on the transition sequence, B1 holds in every reachable state. ∎

*Formal Contract:*
- *Invariant:* `(A p, d : B6(p, d) : (A n : n ≥ 1 ∧ cₙ ∈ s.B ⟹ (A i : 1 ≤ i < n : cᵢ ∈ s.B)))` — equivalently, for every B6-valid (p, d), children(s.B, p, d) = {c₁, ..., cₘ} for some m ≥ 0.
- *Base:* B₀ conf. — seed set satisfies contiguous prefix for every B6-valid (p, d).
- *Preservation:* Each baptismal transition preserves B1 in the target namespace (by B0a, B0, S0, TA5(c), and the next definition) and in every other B6-valid namespace (by B7).

From B₀ conformance (T4 for seeds) and B6(i) (T4 for parents), we derive by induction on the baptism sequence that T4 validity is a registry-wide invariant:

**B10 (T4ValidityInvariant).** `(A t ∈ s.B : t satisfies T4)`

*Proof.* By induction on the number of state transitions from the initial state.

*Base case.* In the initial state, s.B = B₀. By B₀ conf. (SeedConformance), every t ∈ B₀ satisfies T4. The invariant holds at genesis.

*Inductive step.* Assume B10 holds for state s with registry B — every t ∈ B satisfies T4. The s.B-frame case is discharged by B0a-frame (B10 is a predicate on s.B); the baptismal case we now treat.

*Baptismal transition.* By B0a, the transition sets B' = B ∪ {a} where a = next(B, p, d) for some (p, d) satisfying B6. We must show every t ∈ B' satisfies T4. For elements t ∈ B, the inductive hypothesis gives t satisfies T4 directly. It remains to show the new element a satisfies T4.

By the definition of next (NextAddress), a = next(B, p, d) is a stream element of S(p, d): the first child a = inc(p, d) = c₁ when children(B, p, d) = ∅, and the sibling a = inc(cⱼ, 0) = c_{j+1} ∈ S(p, d) otherwise, where cⱼ = max(children(B, p, d)) (the maximum exists because B is finite by B_fin and T1 totally orders the non-empty finite set children(B, p, d) ⊆ B). Since (p, d) satisfies B6, B6's sufficiency result (§B6) gives that every element of S(p, d) satisfies T4; in particular a does.

So a satisfies T4. With every element of B satisfying T4 by the inductive hypothesis, every element of B' = B ∪ {a} satisfies T4; the s.B-frame case was discharged by B0a-frame. By induction on the transition sequence, B10 holds in every reachable state. ∎

*Formal Contract:*
- *Invariant:* `(A t ∈ s.B : t satisfies T4)` — every baptized address satisfies T4 (HierarchicalParsing).
- *Base:* B₀ conf. — every seed element satisfies T4.
- *Preservation:* Each baptismal transition adds a = next(s.B, p, d) ∈ S(p, d); since (p, d) satisfies B6, B6's sufficiency result gives every element of S(p, d) — hence a — satisfies T4. B0a ensures no non-baptismal mechanism introduces elements that might violate T4.

## The high water mark

B1 yields a simplification: the entire allocation state of a namespace reduces to a single natural number.

**hwm(B,p,d) (HighWaterMark).** hwm(B, p, d) = #children(B, p, d) — the *high water mark*.

*Justification.* Let m = #children(B, p, d). By B1 (Contiguous Prefix), children(B, p, d) = {c₁, ..., cₘ} — the first m elements of the sibling stream S(p, d) with no gaps. The count therefore determines the prefix: children(B, p, d) is a single-valued function of m, with max(children) = cₘ when m ≥ 1. ∎

*Formal Contract:*
- *Definition:* hwm(B, p, d) = #children(B, p, d) where children(B, p, d) = {cₙ ∈ S(p, d) : cₙ ∈ B}.
- *Preconditions:* B6(p, d); B satisfies B1 for (p, d); p ∈ T, d ≥ 1; S(p, d) defined.
- *Invariant:* for B6-valid (p, d), hwm(B, p, d) = m implies children(B, p, d) = {c₁, ..., cₘ} and max(children) = cₘ (when m ≥ 1).
- *Depends:* B1 (contiguous prefix), S0 (stream ordering).

Because children(B, p, d) = {c₁, ..., cₘ} is a contiguous prefix (B1), the maximum is always cₘ and the next element is always c_{m+1}. The operational definition of next — "find max, increment" — reduces to counting:

**B2 (High Water Mark Sufficiency).** `next(B, p, d) = c_{hwm(B,p,d) + 1}`.

Concretely: if hwm = 0, then next = inc(p, d) — the first child; if hwm = m > 0, then next = inc(cₘ, 0) — the next sibling. The cardinality of the existing children is a sufficient statistic for the next allocation.

*Proof.* Let m = hwm(B, p, d) = #children(B, p, d). By B1 (Contiguous Prefix), children(B, p, d) = {c₁, ..., cₘ} for this m — the first m elements of S(p, d) with no gaps. The argument splits into two cases exhausting the possible values of m.

*Case 1: m = 0.* The children set is empty: children(B, p, d) = ∅. By the definition of next (NextAddress), next(B, p, d) = inc(p, d). By the definition of the sibling stream, c₁ = inc(p, d). Since hwm + 1 = 0 + 1 = 1, the claim c_{hwm+1} = c₁ = inc(p, d) = next(B, p, d) holds.

*Case 2: m ≥ 1.* The children set is non-empty: children(B, p, d) = {c₁, ..., cₘ}. We must identify max(children(B, p, d)). By S0 (StreamOrdering), the sibling stream is strictly increasing: c₁ < c₂ < ... < cₘ under the lexicographic order T1. The maximum of a finite strictly ordered set is its last element, so max(children(B, p, d)) = cₘ. By the definition of next, next(B, p, d) = inc(cₘ, 0). By the recursive clause of the sibling stream definition, c_{m+1} = inc(cₘ, 0). Since hwm + 1 = m + 1, the claim c_{hwm+1} = c_{m+1} = inc(cₘ, 0) = next(B, p, d) holds.

In both cases, next(B, p, d) = c_{hwm(B,p,d) + 1}. ∎

*Formal Contract:*
- *Preconditions:* B satisfies B1 for all B6-valid (p, d); (p, d) satisfies B6; S(p, d) = c₁, c₂, ... defined by c₁ = inc(p, d), cₙ₊₁ = inc(cₙ, 0).
- *Postconditions:* `next(B, p, d) = c_{hwm(B,p,d) + 1}`.


## Atomicity

**B4 (Atomic Baptism — corollary of the foundation Σ signature).** Each `baptize(p, d) ∈ Σ` is a single edge of `→`: no transition interposes between evaluating `next(s.B, p, d)` and committing the union.

*Depends:* NoDeallocation (ASN-0034) — each `op ∈ Σ` is a partial function `𝒮 ⇀ 𝒮`, a transition being the pair `(s, op(s))`.


## The baptism operation

We now specify the baptism operation itself.

**Bop (Baptism).** The operation baptize(p, d) is defined by:

  PRE: B6(p, d) — depth validity; no parent-baptized prerequisite is imposed
  POST: s'.B = s.B ∪ {next(s.B, p, d)}; only s.B is modified
  ATOMIC: B4 — committed on one edge of →

*Proof of well-definedness and correctness.*

**Well-definedness.** The postcondition invokes next(s.B, p, d), which NextAddress establishes lies in T for any finite B ⊆ T, p ∈ T, d ≥ 1; B_fin (§B_fin) discharges the finiteness premise for B = s.B at any reachable s, so next(s.B, p, d) ∈ T here.

**Freshness.** Let a = next(s.B, p, d). We show a ∉ s.B. By definition children(s.B, p, d) = s.B ∩ S(p, d) ⊆ S(p, d). Two branches of the next definition arise.

*children(s.B, p, d) = ∅.* Then a = inc(p, d) = c₁ ∈ S(p, d). Were a ∈ s.B, then a ∈ s.B ∩ S(p, d) = children(s.B, p, d) = ∅ — impossible. So a ∉ s.B.

*children(s.B, p, d) ≠ ∅.* Then a = inc(max(children(s.B, p, d)), 0). Write q = max(children(s.B, p, d)). Since q ∈ children(s.B, p, d) ⊆ S(p, d), q = cₘ for some m, and a = inc(cₘ, 0) = c_{m+1} ∈ S(p, d). By S0 / TA5(a), inc(·, 0) strictly advances its argument, so a > q ≥ x for every x ∈ children(s.B, p, d); hence a ∉ children(s.B, p, d). Since a ∈ S(p, d), from a ∉ children(s.B, p, d) = s.B ∩ S(p, d) we get a ∉ s.B.

In both branches a ∉ s.B. ∎

*Formal Contract:*
- *Preconditions:* p ∈ T, d ∈ ℕ with d ≥ 1; B6(p, d) holds. (B1, B10, B_fin are reachable-state invariants, not caller obligations.)
- *Atomicity:* B4 — each `baptize(p, d) ∈ Σ` is a single atomic edge of the transition graph.
- *Postconditions:* s'.B = s.B ∪ {next(s.B, p, d)} with next(s.B, p, d) ∉ s.B; s'.B satisfies B0, B1, B10, and B_fin.
- *Frame:* Only s.B is modified.


## Ghost elements: baptism without content

A baptized position need not contain anything. Nelson names these *ghost elements*:

> "While servers, accounts and documents logically occupy positions on the developing tumbler line, no specific element need be stored in tumbler-space to correspond to them. Hence we may call them ghost elements."

A ghost element is "virtually present in tumbler-space, since links may be made to them which embrace all the contents below them." The position is in s.B — it has been baptized, it is permanent, it anchors a namespace for children — but nothing is stored at that address.

**B3 (Ghost Validity).** Baptism establishes a position as a permanent, addressable anchor — its membership in s.B — independently of whether any content is ever stored there. A baptized position that holds no content is a *ghost element*: it is permanent, ordered, and able to anchor a namespace for children, yet stores nothing. The admissible configurations of a tumbler t ∈ T in a reachable state s are therefore:

  - baptized and populated: t ∈ s.B with content stored at t
  - baptized and empty: t ∈ s.B with nothing stored — a ghost element (permitted)
  - unbaptized: t ∉ s.B — not a system entity


## A baptism traced

We trace a concrete sequence to ground the formal development. Begin with B₀ = {[1]} — a single root node. We verify B₀ conformance. First, [1] satisfies T4: a single positive component, no zeros. Second, [1] does not belong to any sibling stream — membership in S(p, d) requires element length #p + d, and no valid parent p with d ∈ {1, 2} satisfies #p + d = 1 (since #p ≥ 1). Therefore children(B₀, p, d) = ∅ for all (p, d), which is trivially a contiguous prefix of length 0. The seed is conforming.

**Step 1: first user.** Namespace ([1], 2) — node [1], depth 2 (level crossing to user).

  next(B₀, [1], 2) = inc([1], 2) = [1, 0, 1]

TA5(d) appends d − 1 = 1 zero separator and child value 1. B5: zeros([1, 0, 1]) = 1 = 0 + (2 − 1). B6: d = 2 and zeros([1]) + 1 = 1 ≤ 3. B1: children = {[1, 0, 1]}, a prefix of length 1.

State: B₁ = {[1], [1, 0, 1]}.

**Step 2: second user.** Same namespace ([1], 2).

  next(B₁, [1], 2) = inc([1, 0, 1], 0) = [1, 0, 2]

TA5(c): sibling increment preserves length, advances position sig([1, 0, 1]) = 3, so the ordinal goes from 1 to 2. B5a: zeros([1, 0, 2]) = 1 = zeros([1, 0, 1]) — sibling preserves zeros. B1: children = {[1, 0, 1], [1, 0, 2]}, a prefix of length 2.

State: B₂ = {[1], [1, 0, 1], [1, 0, 2]}.

**Step 3: document under first user.** Namespace ([1, 0, 1], 2) — user [1, 0, 1], depth 2 (level crossing to document).

  next(B₂, [1, 0, 1], 2) = inc([1, 0, 1], 2) = [1, 0, 1, 0, 1]

B5: zeros([1, 0, 1, 0, 1]) = 2 = 1 + (2 − 1). B6: d = 2 and zeros([1, 0, 1]) + 1 = 2 ≤ 3. B1: children = {[1, 0, 1, 0, 1]}, a prefix of length 1. B7: S([1], 2) elements have length 3; S([1, 0, 1], 2) elements have length 5 — the *Length split* case of B7 gives disjointness.

State: B₃ = {[1], [1, 0, 1], [1, 0, 2], [1, 0, 1, 0, 1]}.

**Step 4: sub-document under first document.** Namespace ([1, 0, 1, 0, 1], 1) — document [1, 0, 1, 0, 1], depth 1 (intra-level descent to sub-document, exercising d = 1).

  next(B₃, [1, 0, 1, 0, 1], 1) = inc([1, 0, 1, 0, 1], 1) = [1, 0, 1, 0, 1, 1]

TA5(d) with d = 1: d − 1 = 0 intermediate zeros, so no zero separator is inserted; the value 1 is appended at position #p + d = 6. B5: zeros([1, 0, 1, 0, 1, 1]) = 2 = zeros([1, 0, 1, 0, 1]) + (1 − 1) — d = 1 contributes no new zeros, so the parent's zero count is preserved. B6: p = [1, 0, 1, 0, 1] satisfies T4 (last component 1 is positive), d = 1 ∈ {1, 2}, and B6(iii) at d = 1 reduces to zeros(p) ≤ 3, which holds since zeros([1, 0, 1, 0, 1]) = 2 ≤ 3. B1: children(B₄, [1, 0, 1, 0, 1], 1) = {[1, 0, 1, 0, 1, 1]}, a contiguous prefix of length 1, witnessing prefix extension under a fresh namespace at d = 1.

State: B₄ = {[1], [1, 0, 1], [1, 0, 2], [1, 0, 1, 0, 1], [1, 0, 1, 0, 1, 1]}.

**Step 5: element under first document.** Namespace ([1, 0, 1, 0, 1], 2) — document [1, 0, 1, 0, 1], depth 2 (level crossing to element). This is the first trace step to reach the deepest hierarchical level, and it lands exactly on the tightest sufficiency boundary.

  next(B₄, [1, 0, 1, 0, 1], 2) = inc([1, 0, 1, 0, 1], 2) = [1, 0, 1, 0, 1, 0, 1]

TA5(d) with d = 2: appends d − 1 = 1 zero separator at position 6 and child value 1 at position 7. B5: zeros([1, 0, 1, 0, 1, 0, 1]) = 3 = 2 + (2 − 1) — the depth-2 increment crosses from document to element, advancing the zero count to its maximum. B6: d = 2 ∈ {1, 2}, and B6(iii) holds *at the boundary*: zeros([1, 0, 1, 0, 1]) + 1 = 3 ≤ 3, i.e. zeros(p) = 2 ≤ 2 — exactly TA5a's `k = 2 ∧ zeros(t) ≤ 2` constraint at equality, with zero slack. The result is T4-valid: zeros = 3 is permitted (the element level saturates the four-level budget), no two zeros are adjacent (positions 2, 4, 6 are separated by positive ordinals), and the last component is positive. B1: children([1, 0, 1, 0, 1], 2) = {[1, 0, 1, 0, 1, 0, 1]}, a prefix of length 1. B7: namespace ([1, 0, 1, 0, 1], 2) has element length 7, while the Step 4 namespace ([1, 0, 1, 0, 1], 1) has length 6 — the *Length split* case gives disjointness despite the shared parent.

State: B₅ = {[1], [1, 0, 1], [1, 0, 2], [1, 0, 1, 0, 1], [1, 0, 1, 0, 1, 1], [1, 0, 1, 0, 1, 0, 1]}.

**Step 6: sub-element under that element.** Namespace ([1, 0, 1, 0, 1, 0, 1], 1) — element [1, 0, 1, 0, 1, 0, 1], depth 1 (intra-level descent). This exercises the other binding boundary: a depth-1 baptism from a parent whose zero budget is already saturated.

  next(B₅, [1, 0, 1, 0, 1, 0, 1], 1) = inc([1, 0, 1, 0, 1, 0, 1], 1) = [1, 0, 1, 0, 1, 0, 1, 1]

TA5(d) with d = 1: d − 1 = 0 intermediate zeros, so no separator; the value 1 is appended at position 8. B5: zeros([1, 0, 1, 0, 1, 0, 1, 1]) = 3 = 3 + (1 − 1) — d = 1 adds no zero, so the element's saturated count is preserved. B6: d = 1 ∈ {1, 2}, and B6(iii) at d = 1 reduces to zeros(p) ≤ 3, holding *at the boundary* zeros([1, 0, 1, 0, 1, 0, 1]) = 3 ≤ 3 — exactly TA5a's `k = 1 ∧ zeros(t) ≤ 3` constraint at equality. The result is T4-valid: zeros = 3 within budget, no adjacent zeros, last component positive. B1: children([1, 0, 1, 0, 1, 0, 1], 1) = {[1, 0, 1, 0, 1, 0, 1, 1]}, a prefix of length 1.

State: B₆ = {[1], [1, 0, 1], [1, 0, 2], [1, 0, 1, 0, 1], [1, 0, 1, 0, 1, 1], [1, 0, 1, 0, 1, 0, 1], [1, 0, 1, 0, 1, 0, 1, 1]}.

Nelson's "Items 2.1, 2.2, 2.3, 2.4" is exactly this mechanism — successive baptisms under parent 2 at depth 1, yielding the sibling stream 2.1, 2.2, 2.3, 2.4 by repeated application of inc(·, 0). The sequence is determined, contiguous, and the ordinals carry no semantics beyond order.

**B7 illustrated — equal-length parents.** The equal-length-parents case is witnessed by S([1, 0, 1], 1) and S([1, 0, 2], 1) from state B₂: both have element length 4 with equal-length distinct parents, so a shared element would force [1, 0, 1] = [1, 0, 2] by T3 — disjoint.

**B7 illustrated — nesting prefixes.** A harder witness: two namespaces whose elements share a length and whose parents nest. Suppose node [1, 1] has been baptized via inc([1], 1) = [1, 1] (TA5(d) with k = 1: #t' = 2, zero intermediate zeros, position 2 set to 1). Consider S([1], 2) and S([1, 1], 1). Both streams have element length 3: #[1] + 2 = #[1, 1] + 1 = 3. The prefixes nest — [1] ≼ [1, 1] — with p = [1], d = 2, p' = [1, 1], d' = 1.

At position 2 of each stream: inc([1], 2) = [1, 0, 1] — the value at position 2 is 0, the zero separator produced by TA5(d) with d − 1 = 1 intermediate zero. inc([1, 1], 1) = [1, 1, 1] — the value at position 2 is p'₂ = 1 > 0 (by T4, valid addresses do not end in zero, so the last component of [1, 1] is positive). Sibling increments inc(·, 0) modify only the last component (TA5(c)), so position 2 is invariant across both streams: always 0 in S([1], 2), always 1 in S([1, 1], 1). The streams disagree at a fixed position and are therefore disjoint.

**Unbounded extent exhibited.** The trace stops at B₆ with hwm(B₆, [1], 2) = 2. Three further sibling baptisms in ([1], 2) yield [1, 0, 3], [1, 0, 4], [1, 0, 5], advancing hwm to 5 — and nothing in the construction caps the count.


## Uniqueness

**B8 (Uniqueness).** Baptismal acts in distinct namespaces produce distinct addresses unconditionally; baptismal acts within a single namespace produce distinct addresses under a single baptismal authority:

  cross-namespace: `(A a, b : produced in distinct namespaces : a ≠ b)`;

  same-namespace: `(A a, b : produced in one namespace under a single authority : a ≠ b)`.

The proof splits two ways: distinct baptisms within the same namespace (the authority-dependent clause), and baptisms in different namespaces (the unconditional clause).

*Proof.* Let a be the address produced by β₁ in namespace (p, d), and b the address produced by β₂ in namespace (p', d'). We proceed by case analysis on whether the two baptisms target the same or different namespaces.

*Case 1: same namespace — (p, d) = (p', d').* Here we take β₁ and β₂ to be commits under a single baptismal authority, so B-Seq (Sequential Commitment) applies: the realized states are totally ordered by →*. Let s₁ be the state on which β₁ acts and s₂ the state on which β₂ acts, with successor states s₁' = β₁(s₁) and s₂' = β₂(s₂). By B4 (Atomic Baptism), each baptism is a single Σ-edge: no realized state lies strictly between s₁ and s₁', nor strictly between s₂ and s₂'. We first establish s₁ ≠ s₂. Suppose s₁ = s₂. Since this is the same-namespace case, both acts apply the same operation `baptize(p, d) ∈ Σ`, which is a partial *function* on 𝒮 (foundation Σ signature, B4). Determinism then gives β₁ = (s₁, baptize(p, d)(s₁)) = (s₂, baptize(p, d)(s₂)) = β₂, contradicting the distinctness of the two acts. Hence s₁ ≠ s₂. By B-Seq, s₁ and s₂ are comparable under →*. Since the postcondition a ≠ b is symmetric in the two acts, relabel β₁ and β₂ so that s₁ →* s₂ with s₁ ≠ s₂. We now advance this to s₁' →* s₂, where s₁' = β₁(s₁) is the immediate successor of s₁. The states s₁' and s₂ are both realized, so B-Seq makes them comparable: s₁' →* s₂ or s₂ →* s₁'. Suppose the latter held. Composing with the relabeled s₁ →* s₂ gives s₁ →* s₂ →* s₁'. But by B4 (Atomic Baptism) the step s₁ → s₁' is a single Σ-edge, so no realized state lies strictly between s₁ and s₁'; the only states t with s₁ →* t →* s₁' are t = s₁ and t = s₁'. Hence s₂ ∈ {s₁, s₁'}, and since s₂ ≠ s₁ we obtain s₂ = s₁', whence s₁' →* s₂ holds reflexively. In either case s₁' →* s₂.

By the Bop postcondition, s₁'.B = s₁.B ∪ {a}, so a ∈ s₁'.B. From s₁' →* s₂, B0★ gives s₁'.B ⊆ s₂.B, hence a ∈ s₂.B.

Let m₁ = hwm(s₁.B, p, d) and m₂ = hwm(s₂.B, p, d). By B2 (High Water Mark Sufficiency), a = c_{m₁+1} and b = c_{m₂+1}, where cₙ denotes the n-th element of S(p, d). Since a = c_{m₁+1} ∈ s₂.B and B1 (Contiguous Prefix) holds for s₂, the children of (p, d) in s₂ include {c₁, ..., c_{m₁+1}}, so hwm(s₂.B, p, d) ≥ m₁ + 1. That is, m₂ ≥ m₁ + 1, hence m₂ + 1 ≥ m₁ + 2 > m₁ + 1. The indices m₁ + 1 and m₂ + 1 are distinct with m₁ + 1 < m₂ + 1. By S0 (StreamOrdering), c_{m₁+1} < c_{m₂+1} under the lexicographic order T1. By T1 irreflexivity, c_{m₁+1} ≠ c_{m₂+1}. Therefore a ≠ b.

*Case 2: different namespaces — (p, d) ≠ (p', d').* By construction, a ∈ S(p, d) — baptism in namespace (p, d) produces the next element of its sibling stream — and b ∈ S(p', d') by the same reasoning. By B7 (Namespace Disjointness), S(p, d) ∩ S(p', d') = ∅, so a ≠ b.

In both cases a ≠ b. ∎

*Formal Contract (cross-namespace — unconditional):*
- *Preconditions:* β₁ produces a in namespace (p, d) and β₂ produces b in namespace (p', d') with (p, d) ≠ (p', d'), where both (p, d) and (p', d') satisfy B6; the system conforms to B7. No single-authority or B-Seq assumption is required.
- *Postconditions:* `a ≠ b`.

*Formal Contract (same-namespace — single authority):*
- *Preconditions:* β₁, β₂ are distinct baptismal acts in one namespace (p, d) = (p', d') satisfying B6, committed under a single baptismal authority (so B-Seq applies), in a system conforming to B-Seq, B0★ (which subsumes B0), B0a, B1, B2, and B4.
- *Postconditions:* `a ≠ b`.


## Unbounded growth

Nelson insists that the address space imposes no capacity limits:

> "A tumbler consists of a series of integers. Each integer has no upper limit."

**B9 (Unbounded Extent).** `(A p, d : B6(p, d) : (A M ∈ ℕ : (E s' : s →* s' via baptisms : hwm(s'.B, p, d) ≥ M)))`.

No architectural limit constrains how many children a position may have.

*Proof.* The argument is constructive: we exhibit the required sequence of baptismal transitions.

Let m = hwm(s.B, p, d) — the current count of children in namespace (p, d). If m ≥ M, set s' = s (the empty transition sequence witnesses s →* s via reflexivity) and the claim holds trivially. Otherwise m < M, and we construct a sequence of M − m baptismal transitions, each `baptize(p, d) ∈ Σ` targeting namespace (p, d). We show by induction on k that k successive baptismal transitions s → s₁ → ... → sₖ produce a state sₖ with hwm(sₖ.B, p, d) = m + k.

*Base case (k = 0).* s₀ = s with hwm(s.B, p, d) = m = m + 0. The claim holds by the reflexive case of →*.

*Inductive step.* Assume sₖ is a state reachable from s by k baptismal transitions in namespace (p, d), with hwm(sₖ.B, p, d) = m + k < M. We perform the transition `sₖ → sₖ₊₁` induced by `baptize(p, d) ∈ Σ` — that is, sₖ₊₁ = baptize(p, d)(sₖ). The preconditions of Bop are satisfied: B6(p, d) holds by hypothesis; by B4 (Atomic Baptism), each baptism is a single transition edge, so the constructed sequence is a chain of M − m successive edges.

By Bop, the postcondition gives sₖ₊₁.B = sₖ.B ∪ {next(sₖ.B, p, d)}. By B2 (High Water Mark Sufficiency), next(sₖ.B, p, d) = c_{m+k+1}, the (m + k + 1)-th element of the sibling stream S(p, d). This element lies in T: c₁ = inc(p, d) ∈ T by TA5(d), and each cₙ₊₁ = inc(cₙ, 0) ∈ T by TA5(c), the ordinal advancing by successor closure (NAT-closure).

The new element c_{m+k+1} is fresh — by the freshness argument of Bop, it does not appear in sₖ.B. The contiguous prefix property is preserved — by B1 preservation under Bop, children(sₖ₊₁.B, p, d) = {c₁, ..., c_{m+k+1}}. Therefore hwm(sₖ₊₁.B, p, d) = m + k + 1.

After M − m steps, hwm(s_{M−m}.B, p, d) = m + (M − m) = M. Setting s' = s_{M−m}, we have s →* s' via the M − m baptismal transitions, and hwm(s'.B, p, d) = M ≥ M. ∎

*Formal Contract:*
- *Preconditions:* (p, d) satisfying B6(p, d); M ∈ ℕ; current state s reachable from s_init.
- *Postconditions:* There exists s' with s →* s' via a finite sequence of baptismal transitions such that hwm(s'.B, p, d) ≥ M.
- *Depends:* TA5(c), TA5(d) — inc(·, 0) and inc(·, d) are total on T, supplying each cₙ; NAT-closure — ℕ is closed under successor, so the child ordinal n + 1 ∈ ℕ at every step, hence grows without bound.


## Properties Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| s.B | B ⊆ T — the set of baptized tumblers (baptismal registry) | introduced |
| S(p,d) | Sibling stream: c₁ = inc(p, d), cₙ₊₁ = inc(cₙ, 0) | from TA5(b), TA5(c), TA5(d) |
| hwm(B,p,d) | High water mark: #children(B, p, d) — sufficient allocation statistic | from B1, S0 |
| next(B,p,d) | Next address: if children = ∅ then inc(p, d) else inc(max(children), 0) | from TA5(c), TA5(d), T1 |
| Bop | baptize(p, d): PRE B6; ATOMIC B4; POST s'.B = s.B ∪ {next(s.B, p, d)}; FRAME modifies only s.B | from B0a, B4, B6, B_fin, next def., S0, TA5(a) |
| S0 | `(A i, j : 1 ≤ i < j : cᵢ < cⱼ)` — stream strictly ordered | from TA5(a), T1 |
| S1 | `(A n : n ≥ 1 : p ≼ cₙ)` — all stream elements extend parent | from TA5(b), TA5(c), TA5(d) |
| B0 | `s.B ⊆ s'.B` for all transitions — irrevocability (analogous to T8 for the registry component) | from B0a |
| B0★ | `s.B ⊆ s'.B` for all s →* s' (reflexive-transitive closure of transitions) — multi-step irrevocability | labelled corollary of B0 |
| B-Seq | States realized under a single baptismal authority are totally ordered by →* — no two commits fork from a shared state (sequential commitment) | model axiom (grounded in implementation) |
| B0a | Σ partitions into baptismal operations (the `baptize(p, d)` for B6-valid (p, d), each acting on s.B as in Bop) and s.B-frame operations (every other op satisfies `op(s).B = s.B`) — registry grows only through baptism | design requirement |
| B₀ conf. | B₀ is finite, `children(B₀, p, d)` is a contiguous prefix for every B6-valid (p, d), and `(A t ∈ B₀ : t satisfies T4)` — seed conformance | design requirement |
| B_fin | `(A s reachable : s.B is finite)` — registry finiteness | from B₀ conf., B0a |
| B1 | `B6(p, d) ⟹ (cₙ ∈ B ⟹ (A i : 1 ≤ i < n : cᵢ ∈ B))` — contiguous prefix over B6-valid namespaces (requires conforming B₀) | from B₀ conf., B0, B0a, B6, B7, next def., S0, TA5(c) |
| B2 | `next(B, p, d) = c_{hwm+1}` — high water mark sufficiency (from B1) | from B1, S0, NextAddress |
| B3 | Baptism (`t ∈ s.B`) is independent of content: a baptized position may hold nothing — a ghost element | introduced |
| B4 | Each `baptize(p, d) ∈ Σ` is a single edge of `→` — no transition interposes between evaluating `next` and committing the union | corollary of foundation Σ signature |
| B5 | `zeros(inc(p, d)) = zeros(p) + (d − 1)` — field advancement | from TA5(b), TA5(d) |
| B5a | `zeros(inc(t, 0)) = zeros(t)` — sibling increment preserves zeros | from TA5(c) |
| B6 | `p satisfies T4`, `d ∈ {1, 2}`, and `zeros(p) + (d − 1) ≤ 3` — valid depth | from T4, TA5, B5 |
| B7 | `(p, d) ≠ (p', d') ⟹ S(p, d) ∩ S(p', d') = ∅` — namespace disjointness | from S(p,d) structure, TA5(c), B6, TA5(d), T3, T4 |
| B8 | Distinct-namespace baptisms produce distinct addresses (unconditional); same-namespace baptisms produce distinct addresses under a single authority — uniqueness | from B-Seq, B0★, B1, B2, B4, B7, S0, T1 |
| B9 | `(A p, d : B6(p, d) : (A M ∈ ℕ : (E s' : s →* s' via baptisms : hwm(s'.B, p, d) ≥ M)))` — unbounded extent | from B1, B2, B4, B6, Bop, TA5(c), TA5(d), NAT-closure |
| B10 | `(A t ∈ s.B : t satisfies T4)` — registry-wide T4 validity | from B₀ conf., B0a, B6, TA5(c), TA5a |


## Open Questions

- Must a parent position be baptized before children can be baptized beneath it? Nelson's ownership model implies yes; Gregory's implementation does not check at structural levels. Resolution depends on the ownership model (Tumbler Ownership).
- Under what activation discipline does `allocated(s) ⊆ s.B` hold — what must align each allocator-extension transition with a baptismal operation, and cover the genesis allocator domain with the seed?
- What concrete seed sets B₀ are valid — which root configurations satisfy B₀ conformance while providing a viable system genesis?
- Must the specification distinguish between a ghost element that could hold content and a structural position that cannot — or is this distinction derivable from the field structure alone?
- Under what conditions may bulk allocation — baptizing a contiguous range of k positions in a single operation — satisfy B4's atomicity and B1's contiguity requirements?
- Given that concurrency across independent owners is concurrency over disjoint namespaces (where B7 already guarantees non-overlap), what must coexisting replicas allocating in a shared namespace — divergent branches outside B-Seq's scope — guarantee about cross-replica baptism ordering to maintain global uniqueness without centralized coordination?
- What invariants must element-level subspace partitioning (T7) satisfy so that the contiguous prefix property holds independently within each subspace?
