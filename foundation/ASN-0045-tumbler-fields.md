> **ASN-0045 · Tumbler Fields** — Foundation layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](ASN-0034-tumbler-algebra.md)  
> [Condensed statements →](ASN-0045-tumbler-fields.statements.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0045: Tumbler Fields

*2026-03-17*

The tumbler hierarchy (T4, ASN-0034) parses every T4-valid address into four field levels separated by zero components. T4c (LevelDetermination, ASN-0034) already pins those levels to zero-count: zeros(t) ∈ {0, 1, 2, 3} corresponds to node, user, document, element. ASN-0045 names those levels as predicates over T for downstream use, with one rename (user → account) recorded below.

## Naming Convention

T4c labels the level with zeros(t) = 1 as a *user address*. ASN-0045 adopts *account* as the canonical predicate name and treats T4c's *user* as an alias for the same address class. Nelson uses both terms in Literary Machines — "user" for the field-name slot, "account" for the addressable allocation (LM 4/29). The udanax-green implementation settles on "account" in its structural and addressing code (`tumbleraccounteq`, `ACCOUNT` constant). The other three labels (node, document, element) are taken verbatim from T4c.

The rename applies only to the address-class label. T4b's projection symbol `U : T ⇀ T` (the user-component projection on a parsed tumbler) is unchanged by ASN-0045; downstream uses of `U` and `t.Uᵢ` continue without rebinding.

## Hierarchy Level Definitions

For any tumbler t, T4-valid(t) (T4, ASN-0034) means t parses as a well-formed address. T4 is a foundation axiom characterizing valid address tumblers but introduces no one-place predicate symbol; ASN-0045 coins `T4-valid` as the conjunction of T4's four clauses, pinned down explicitly:

**T4-valid** — `T4-valid(t) ≡ zeros(t) ≤ 3 ∧ (A i : 1 ≤ i < #t : ¬(tᵢ = 0 ∧ tᵢ₊₁ = 0)) ∧ t₁ ≠ 0 ∧ t_{#t} ≠ 0`.

Given T4-valid(t), T4c assigns t to exactly one level by zeros(t). We name the four corresponding predicates by definitional equivalence:

**Node** — `Node(t) ≡ T4-valid(t) ∧ zeros(t) = 0`.

**Account** — `Account(t) ≡ T4-valid(t) ∧ zeros(t) = 1`.

**Document** — `Document(t) ≡ T4-valid(t) ∧ zeros(t) = 2`.

**Element** — `Element(t) ≡ T4-valid(t) ∧ zeros(t) = 3`.

Each predicate is a one-place proposition on the tumbler carrier T (T0, ASN-0034). The definitions are total: for any t : T, T4-valid(t) is a well-formed proposition (T4, ASN-0034) and zeros(t) is a natural number — the cardinality of a finite index set over T0's carrier ℕ, where T4 (ASN-0034) defines zeros(t) — so each conjunction is a well-formed proposition without precondition.

## Well-Definedness

Two facts about the T4-valid subdomain drive the corollary: T4's arithmetic bound `zeros(t) ≤ 3` (T4, ASN-0034) together with `zeros(t) ∈ ℕ` (T0, ASN-0034) confines the zero-count to `{0, 1, 2, 3}`, and T4c's map zeros(t) → level (T4c, ASN-0034) names each zeros-class as one of node/account/document/element. We derive Partition as a corollary in three steps.

*Binding.* Fix t : T with T4-valid(t). By the definitions above, each of Node(t), Account(t), Document(t), Element(t) reduces to `zeros(t) = k` for k ∈ {0, 1, 2, 3} respectively.

*At-least-one.* For T4-valid t, `zeros(t) ∈ {0, 1, 2, 3}`. This follows from the arithmetic bound, not from the bijection's domain. T4's axiom (ASN-0034) gives the upper bound `zeros(t) ≤ 3`, and T0's carrier ℕ (ASN-0034) gives the lower bound `zeros(t) ≥ 0`, since `zeros(t)` is a cardinality and hence a natural number. Together `0 ≤ zeros(t) ≤ 3` confines zeros(t) to the bounded segment, but the step from this bound to the *disjunction* `zeros(t) = 0 ∨ 1 ∨ 2 ∨ 3` requires three ℕ facts working in concert, not discreteness alone. Write n for zeros(t). The numerals are consecutive — `0 < 1 < 2 < 3` — by NAT-addcompat's strict successor inequality (ASN-0034), `k < k + 1`. We exhaust the segment by a single application schema, stated once and applied at the three interior boundaries. For each `m ∈ {0, 1, 2}`, NAT-discrete (ASN-0034), `m ≤ n < m + 1 ⟹ n = m`, rules out the open gap `m < n < m + 1` as vacuous in ℕ: any n in that gap would satisfy `m ≤ n < m + 1` and hence be forced to `n = m`, contradicting the strict `n > m` that defines the gap, so no natural number lies strictly between consecutive numerals. With every gap empty, NAT-order's trichotomy (ASN-0034) against the boundaries `0, 1, 2, 3` exhausts the segment: the lower bound `0 ≤ n` excludes `n < 0` and the upper bound `n ≤ 3` excludes `n > 3`, and the three interior gaps are vacuous, leaving only the four boundary values themselves. Each boundary comparison is trichotomy (NAT-order); the emptiness of each gap is discreteness (NAT-discrete); the consecutiveness that makes the numerals adjacent is the successor inequality (NAT-addcompat). Hence `0 ≤ n ≤ 3 ∧ n ∈ ℕ ⟹ n ∈ {0, 1, 2, 3}` (the numerals 1, 2, 3 grounded in NAT-closure, ASN-0034). Hence `zeros(t) ∈ {0, 1, 2, 3}`. Reading the conclusion off the bijection's domain would be circular: T4c's claim to be "a bijection on `{0, 1, 2, 3}`" presupposes that the zero-count being labeled already lies in `{0, 1, 2, 3}`, which is precisely what at-least-one must establish. With `zeros(t) ∈ {0, 1, 2, 3}` secured by the bound, T4c then attaches the level name to whichever value `zeros(t)` takes. Hence at least one of the four equalities zeros(t) = k holds, so at least one of the four predicates holds at t.

*At-most-one.* zeros(t) is a single natural number — the cardinality of a fixed finite index set (T4, ASN-0034) — so it is a function of t and equals at most one value. Each of Node, Account, Document, Element is defined directly as `zeros(t) = k` for a distinct k ∈ {0, 1, 2, 3}; the predicates never route through the level labels, so the comparison is between zero-counts, not levels. The four values 0, 1, 2, 3 are pairwise distinct as natural numbers: NAT-addcompat's strict successor inequality (ASN-0034), `k < k + 1`, gives the strict chain `0 < 1 < 2 < 3`, and NAT-order's trichotomy and irreflexivity (ASN-0034) convert each strict inequality into distinctness — exactly the same numeral-ordering content the at-least-one derivation routes through NAT-addcompat and NAT-order, not a T0-carrier fact (T0 posits only the carrier ℕ, deferring such arithmetic to the separate ℕ axioms). Since zeros(t) is single-valued and the four targets are distinct, no two of the equalities `zeros(t) = k` can hold at once, so no two predicates hold simultaneously at t. The disjointness rests solely on the functionality of zeros(t) (T4) and the pairwise distinctness of 0, 1, 2, 3 in ℕ (NAT-order's trichotomy/irreflexivity over the chain `0 < 1 < 2 < 3` supplied by NAT-addcompat); T4c's injectivity — a statement about distinct *levels* in the codomain — does no work here.

Combining the two yields the Partition postcondition:

**Partition** — `(A t : T : T4-valid(t) :: exactly-one-of(Node(t), Account(t), Document(t), Element(t)))`.

The quantifier ranges over the full carrier T; the antecedent T4-valid(t) restricts the assertion to parseable tumblers. Partition makes no claim about T4-invalid t.

## Examples

*Positive cases (T4-valid).* Each row classifies under exactly one predicate.

| Tumbler | zeros(t) | Level |
|---------|----------|-------|
| [7] | 0 | Node |
| [7, 0, 3] | 1 | Account |
| [7, 0, 3, 0, 5] | 2 | Document |
| [7, 0, 3, 0, 5, 0, 1] | 3 | Element |

*Counter-examples (T4-invalid).* For each, ¬T4-valid(t) holds, so all four predicates evaluate to false and Partition makes no claim.

| Tumbler | T4 clause violated | Why all four predicates are false |
|---------|--------------------|-----------------------------------|
| [7, 0, 0, 3] | adjacent zeros | T4-valid fails; each predicate's left conjunct is false |
| [0, 7] | leading zero | T4-valid fails; each predicate's left conjunct is false |
| [7, 0] | trailing zero | T4-valid fails; each predicate's left conjunct is false |
| [1, 0, 1, 0, 1, 0, 1, 0, 1] | zeros(t) = 4 > 3 violates the bound `zeros(t) ≤ 3` | T4-valid fails; each predicate's left conjunct is false |

The counter-examples show why Partition's antecedent T4-valid(t) is load-bearing: dropping it would force at-least-one to fail on every T4-invalid tumbler. But the rows also expose a uniform consequence worth stating as a property rather than leaving to four examples. Each of the four predicates is a conjunction whose left conjunct is T4-valid(t); when ¬T4-valid(t), conjunction elimination falsifies all four at once, independently of zeros(t). This is the complement of Partition over the carrier — off the valid subdomain no predicate holds — and we record it as Off-Domain Vacuity below. Together with Partition it gives the full picture over T: exactly one predicate holds where T4-valid(t), and exactly zero where ¬T4-valid(t).

## Properties Introduced

**Node** (`Node(t) ≡ T4-valid(t) ∧ zeros(t) = 0`)

- *Preconditions.* None (predicate is total on T).
- *Definition.* The two-place conjunction above.
- *Depends.* T0 (carrier ℕ), NAT-closure (the constant 0 as additive identity, `0 + n = n`, grounding `0 ∈ ℕ` uniformly with the numerals 1, 2, 3 sourced by the sibling predicates), T4 (T4-valid) for the base biconditional. The level-correspondence postcondition alone depends additionally on T4c (LevelDetermination), T4b (UniqueParse), and T3 (CanonicalRepresentation): T4c supplies the *node address* ⟺ `zeros(t) = 0` bijection, and T4b and T3 discharge T4c's applicability at t — T4-valid(t) supplies the T4 positional constraints, and T3 together with those constraints supplies T4b at t, licensing T4c's bijection postcondition.
- *Postconditions.*
  - `(A t : T :: Node(t) ⟺ T4-valid(t) ∧ zeros(t) = 0)`.
  - *Level correspondence:* `(A t : T : T4-valid(t) :: Node(t) ⟺ t is a node address per T4c)` — derived: fix t : T with T4-valid(t); the definition `Node(t) ≡ T4-valid(t) ∧ zeros(t) = 0` collapses under the T4-valid antecedent to `Node(t) ⟺ zeros(t) = 0`. T4c's preconditions (the T4 positional constraints together with T4b) are discharged at t by T4-valid(t) (supplying the T4 constraints) and T3 (CanonicalRepresentation, universal, supplying T4b at t), so T4c's bijection postcondition instantiated at t supplies `zeros(t) = 0 ⟺ t is a node address`; chaining the two biconditionals yields `Node(t) ⟺ t is a node address`.

**Account** (`Account(t) ≡ T4-valid(t) ∧ zeros(t) = 1`)

- *Preconditions.* None.
- *Definition.* The two-place conjunction above.
- *Depends.* T0, T4 (T4-valid), NAT-closure (the constant 1). T4c justifies the *account* level label only; it does no work in this base biconditional and is not a proof dependency of it. The rename-equivalence postcondition alone depends on T4c (LevelDetermination), T4b (UniqueParse), and T3 (CanonicalRepresentation): T4c supplies the *user address* ⟺ `zeros(t) = 1` bijection, and T4b and T3 discharge T4c's applicability at t — T4-valid(t) supplies the T4 positional constraints, and T3 together with those constraints supplies T4b at t, licensing T4c's bijection postcondition.
- *Postconditions.*
  - `(A t : T :: Account(t) ⟺ T4-valid(t) ∧ zeros(t) = 1)`.
  - *Rename equivalence:* `(A t : T : T4-valid(t) :: Account(t) ⟺ t is a user address per T4c)` — derived: fix t : T with T4-valid(t); the definition `Account(t) ≡ T4-valid(t) ∧ zeros(t) = 1` collapses under the T4-valid antecedent to the biconditional `Account(t) ⟺ zeros(t) = 1`. Before invoking T4c we discharge its applicability at t: T4c's preconditions are the T4 positional constraints together with T4b (UniqueParse). T4-valid(t) supplies the T4 positional constraints directly, and T3 (CanonicalRepresentation, universal) together with those same constraints supplies T4b at t, so T4c's bijection postcondition is licensed at t. T4c's *Postcondition* (the bijection clause) instantiated at t then supplies `zeros(t) = 1 ⟺ t is a user address`; chaining the two biconditionals yields `Account(t) ⟺ t is a user address`. ASN-0045's *account* and T4c's *user address* denote the same predicate on the T4-valid subdomain.

**Document** (`Document(t) ≡ T4-valid(t) ∧ zeros(t) = 2`)

- *Preconditions.* None.
- *Definition.* The two-place conjunction above.
- *Depends.* T0, T4 (T4-valid), NAT-closure (successor and addition closure ground the numeral `2 := 1 + 1`) for the base biconditional. The level-correspondence postcondition alone depends additionally on T4c (LevelDetermination), T4b (UniqueParse), and T3 (CanonicalRepresentation): T4c supplies the *document address* ⟺ `zeros(t) = 2` bijection, and T4b and T3 discharge T4c's applicability at t — T4-valid(t) supplies the T4 positional constraints, and T3 together with those constraints supplies T4b at t, licensing T4c's bijection postcondition.
- *Postconditions.*
  - `(A t : T :: Document(t) ⟺ T4-valid(t) ∧ zeros(t) = 2)`.
  - *Level correspondence:* `(A t : T : T4-valid(t) :: Document(t) ⟺ t is a document address per T4c)` — derived: fix t : T with T4-valid(t); the definition `Document(t) ≡ T4-valid(t) ∧ zeros(t) = 2` collapses under the T4-valid antecedent to `Document(t) ⟺ zeros(t) = 2`. T4c's preconditions (the T4 positional constraints together with T4b) are discharged at t by T4-valid(t) (supplying the T4 constraints) and T3 (CanonicalRepresentation, universal, supplying T4b at t), so T4c's bijection postcondition instantiated at t supplies `zeros(t) = 2 ⟺ t is a document address`; chaining the two biconditionals yields `Document(t) ⟺ t is a document address`.

**Element** (`Element(t) ≡ T4-valid(t) ∧ zeros(t) = 3`)

- *Preconditions.* None.
- *Definition.* The two-place conjunction above.
- *Depends.* T0, T4 (T4-valid), NAT-closure (successor and addition closure ground the numeral `3 := 2 + 1`) for the base biconditional. The level-correspondence postcondition alone depends additionally on T4c (LevelDetermination), T4b (UniqueParse), and T3 (CanonicalRepresentation): T4c supplies the *element address* ⟺ `zeros(t) = 3` bijection, and T4b and T3 discharge T4c's applicability at t — T4-valid(t) supplies the T4 positional constraints, and T3 together with those constraints supplies T4b at t, licensing T4c's bijection postcondition.
- *Postconditions.*
  - `(A t : T :: Element(t) ⟺ T4-valid(t) ∧ zeros(t) = 3)`.
  - *Level correspondence:* `(A t : T : T4-valid(t) :: Element(t) ⟺ t is an element address per T4c)` — derived: fix t : T with T4-valid(t); the definition `Element(t) ≡ T4-valid(t) ∧ zeros(t) = 3` collapses under the T4-valid antecedent to `Element(t) ⟺ zeros(t) = 3`. T4c's preconditions (the T4 positional constraints together with T4b) are discharged at t by T4-valid(t) (supplying the T4 constraints) and T3 (CanonicalRepresentation, universal, supplying T4b at t), so T4c's bijection postcondition instantiated at t supplies `zeros(t) = 3 ⟺ t is an element address`; chaining the two biconditionals yields `Element(t) ⟺ t is an element address`.

**Partition**

- *Preconditions.* None.
- *Definition.* `(A t : T : T4-valid(t) :: exactly-one-of(Node(t), Account(t), Document(t), Element(t)))`, where `exactly-one-of(P₁, P₂, P₃, P₄) ≡ (P₁ ∨ P₂ ∨ P₃ ∨ P₄) ∧ (A i, j : 1 ≤ i < j ≤ 4 :: ¬(Pᵢ ∧ Pⱼ))`.
- *Depends.* Node, Account, Document, Element (definitions above), T4 (axiom `zeros(t) ≤ 3` supplies the upper bound for at-least-one; zeros(t) is the cardinality of a fixed finite index set, hence a single-valued function of t, for at-most-one), T0 (carrier ℕ supplies `zeros(t) ≥ 0` for at-least-one), NAT-discrete, NAT-order, NAT-addcompat (jointly enumerate the bounded segment `0 ≤ n ≤ 3 ∧ n ∈ ℕ ⟹ n ∈ {0, 1, 2, 3}` for at-least-one: NAT-order's trichotomy supplies each case split against a numeral boundary, NAT-discrete pins the value once bracketed between consecutive numerals, and NAT-addcompat's `k < k + 1` makes the numerals 0 < 1 < 2 < 3 consecutive; and jointly supply the pairwise distinctness of 0, 1, 2, 3 in ℕ for at-most-one: NAT-addcompat's `k < k + 1` gives the chain `0 < 1 < 2 < 3` and NAT-order's trichotomy/irreflexivity converts each strict inequality into distinctness), NAT-closure (grounds the numerals 1, 2, 3 in the enumeration). T4c is **not** a premise of this exactly-one-of derivation: the postcondition asserts a relation among the four predicates, each defined as `zeros(t) = k`, so the derivation compares zero-counts and never routes through the level names. T4c contributes nomenclature only — the level names node/account/document/element for reporting — and the per-predicate level-correspondence postconditions discharge its applicability separately.
- *Postcondition.* `(A t : T : T4-valid(t) :: exactly-one-of(Node(t), Account(t), Document(t), Element(t)))` — derived by confining the range to `zeros(t) ∈ {0, 1, 2, 3}` via T4's bound `zeros(t) ≤ 3` and T0's `zeros(t) ≥ 0` for the at-least-one direction, and combining the functionality of zeros(t) (T4) with the pairwise distinctness of 0, 1, 2, 3 in ℕ (NAT-addcompat's chain `0 < 1 < 2 < 3` together with NAT-order's trichotomy/irreflexivity) for the at-most-one direction, per the *Well-Definedness* derivation above. T4c plays no part in this derivation; the level names it supplies are nomenclature only.

**Off-Domain Vacuity**

- *Preconditions.* None.
- *Definition.* `(A t : T : ¬T4-valid(t) :: ¬Node(t) ∧ ¬Account(t) ∧ ¬Document(t) ∧ ¬Element(t))`.
- *Depends.* Node, Account, Document, Element (definitions above) — each predicate is a conjunction whose left conjunct is T4-valid(t); T4 (the source of T4-valid).
- *Postcondition.* `(A t : T : ¬T4-valid(t) :: ¬Node(t) ∧ ¬Account(t) ∧ ¬Document(t) ∧ ¬Element(t))` — derived: fix t : T with ¬T4-valid(t). Each of the four predicates is `T4-valid(t) ∧ zeros(t) = k`; with the left conjunct false, conjunction elimination falsifies each conjunction regardless of zeros(t), so all four predicates are false at t. This is the off-domain complement of Partition: combined, `exactly-one-of` holds on the T4-valid subdomain and `exactly-zero` holds off it, so over the full carrier T every tumbler satisfies at most one of the four predicates.

## Summary

| Label | Statement | Status |
|-------|-----------|--------|
| Node | `Node(t) ≡ T4-valid(t) ∧ zeros(t) = 0` | definition coined by ASN-0045 (T4, T0); level correspondence on T4-valid t derived from T4c (via T4b, T3): equivalent to T4c's *node address* |
| Account | `Account(t) ≡ T4-valid(t) ∧ zeros(t) = 1` | definition coined by ASN-0045 (T4, T0); rename/level correspondence on T4-valid t derived from T4c (via T4b, T3): equivalent to T4c's *user address* |
| Document | `Document(t) ≡ T4-valid(t) ∧ zeros(t) = 2` | definition coined by ASN-0045 (T4, T0); level correspondence on T4-valid t derived from T4c (via T4b, T3): equivalent to T4c's *document address* |
| Element | `Element(t) ≡ T4-valid(t) ∧ zeros(t) = 3` | definition coined by ASN-0045 (T4, T0); level correspondence on T4-valid t derived from T4c (via T4b, T3): equivalent to T4c's *element address* |
| Partition | `(A t : T : T4-valid(t) :: exactly-one-of(Node(t), Account(t), Document(t), Element(t)))` | derived from T4 bound `zeros(t) ≤ 3` + T0 `zeros(t) ≥ 0` (at-least-one) and functionality of zeros (T4) + distinctness of 0,1,2,3 in ℕ (NAT-addcompat + NAT-order) (at-most-one); T4c not a premise — nomenclature only |
| Off-Domain Vacuity | `(A t : T : ¬T4-valid(t) :: ¬Node(t) ∧ ¬Account(t) ∧ ¬Document(t) ∧ ¬Element(t))` | derived from the shared left conjunct T4-valid(t) by conjunction elimination; complement of Partition over T |
