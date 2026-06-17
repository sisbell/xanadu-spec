> **ASN-0045 · Tumbler Fields** — condensed claim statements  
> [← Full note](ASN-0045-tumbler-fields.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0045 Claim Statements

*Source: ASN-0045-tumbler-fields.md (revised 2026-03-17) — Extracted: 2026-05-28*

## Definition — T4Valid

`T4-valid(t) ≡ zeros(t) ≤ 3 ∧ (A i : 1 ≤ i < #t : ¬(tᵢ = 0 ∧ tᵢ₊₁ = 0)) ∧ t₁ ≠ 0 ∧ t_{#t} ≠ 0`

## Definition — ExactlyOneOf

`exactly-one-of(P₁, P₂, P₃, P₄) ≡ (P₁ ∨ P₂ ∨ P₃ ∨ P₄) ∧ (A i, j : 1 ≤ i < j ≤ 4 :: ¬(Pᵢ ∧ Pⱼ))`

---

## Node — Node (DEF, predicate)

`Node(t) ≡ T4-valid(t) ∧ zeros(t) = 0`

- Total on carrier T (no precondition)
- Postcondition: `(A t : T :: Node(t) ⟺ T4-valid(t) ∧ zeros(t) = 0)`

## Account — Account (DEF, predicate)

`Account(t) ≡ T4-valid(t) ∧ zeros(t) = 1`

- Total on carrier T (no precondition)
- Postcondition: `(A t : T :: Account(t) ⟺ T4-valid(t) ∧ zeros(t) = 1)`

## Document — Document (DEF, predicate)

`Document(t) ≡ T4-valid(t) ∧ zeros(t) = 2`

- Total on carrier T (no precondition)
- Postcondition: `(A t : T :: Document(t) ⟺ T4-valid(t) ∧ zeros(t) = 2)`

## Element — Element (DEF, predicate)

`Element(t) ≡ T4-valid(t) ∧ zeros(t) = 3`

- Total on carrier T (no precondition)
- Postcondition: `(A t : T :: Element(t) ⟺ T4-valid(t) ∧ zeros(t) = 3)`

## Partition — Partition (LEMMA, lemma)

`(A t : T : T4-valid(t) :: exactly-one-of(Node(t), Account(t), Document(t), Element(t)))`

Where `exactly-one-of` expands as defined above. Quantifier ranges over full carrier T; antecedent T4-valid(t) restricts to parseable tumblers. Makes no claim about T4-invalid t.

## Off-Domain Vacuity — OffDomainVacuity (LEMMA, lemma)

`(A t : T : ¬T4-valid(t) :: ¬Node(t) ∧ ¬Account(t) ∧ ¬Document(t) ∧ ¬Element(t))`
