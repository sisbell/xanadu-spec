> **ASN-0040 · Tumbler Baptism** — condensed claim statements  
> [← Full note](ASN-0040-tumbler-baptism.md) · [↑ Foundation index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0040 Claim Statements

*Source: ASN-0040-tumbler-baptism.md (revised 2026-03-15) — Extracted: 2026-05-29*

## Definition — Children

```
children(B, p, d) = B ∩ S(p, d)
```
The set of baptized addresses belonging to the sibling stream of parent p at depth d.

---

## s.B — BaptismalRegistry (DEF, state-component)

```
s.B ⊆ T — the set of baptized tumblers.

A tumbler t is baptized iff t ∈ s.B. Initially s.B = B₀. Thereafter it grows
monotonically.
```

---

## S(p,d) — SiblingStream (DEF, inductive-definition)

```
Definition: S(p, d) = c₁, c₂, c₃, ... where c₁ = inc(p, d) and cₙ₊₁ = inc(cₙ, 0)
for n ≥ 1.

Preconditions: p ∈ T, d ≥ 1.

Postconditions:
  (A n ≥ 1 : cₙ = [p₁, ..., p_{#p}, 0, ..., 0, n])  with d − 1 zeros
  #cₙ = #p + d
  sig(cₙ) = #p + d
  cₙᵢ = pᵢ  for 1 ≤ i ≤ #p
```

---

## hwm(B,p,d) — HighWaterMark (DEF, function)

```
hwm(B, p, d) = #children(B, p, d)
where children(B, p, d) = {cₙ ∈ S(p, d) : cₙ ∈ B}.

Preconditions: B6(p, d); B satisfies B1 for (p, d); p ∈ T, d ≥ 1; S(p, d) defined.

Invariant: for B6-valid (p, d), hwm(B, p, d) = m implies
  children(B, p, d) = {c₁, ..., cₘ}  and  max(children) = cₘ  (when m ≥ 1).
```

---

## next(B,p,d) — NextAddress (DEF, function)

```
next(B, p, d) = if children(B, p, d) = ∅ then inc(p, d)
                else inc(max(children(B, p, d)), 0)
where children(B, p, d) = B ∩ S(p, d).

Preconditions: B ⊆ T finite (discharged by B_fin when B = s.B for a reachable s);
  p ∈ T; d ≥ 1; S(p, d) defined.

Postconditions: next(B, p, d) ∈ T.
```

---

## Bop — Baptize (DEF, operation)

```
PRE:    B6(p, d)
ATOMIC: B4 — each baptize(p, d) ∈ Σ is a single atomic edge of the transition graph
POST:   s'.B = s.B ∪ {next(s.B, p, d)}  with  next(s.B, p, d) ∉ s.B
        s'.B satisfies B0, B1, B10, and B_fin
FRAME:  only s.B is modified
```

---

## S0 — StreamOrdering (LEMMA, postcondition)

```
Preconditions: p ∈ T, d ≥ 1. S(p, d) = c₁, c₂, ... defined by c₁ = inc(p, d),
  cₙ₊₁ = inc(cₙ, 0).

Postconditions: (A i, j : 1 ≤ i < j : cᵢ < cⱼ) — the sibling stream is strictly
  increasing.
```

---

## S1 — StreamPrefix (LEMMA, postcondition)

```
Relation: ≼ is the foundation Prefix relation (ASN-0034).

Preconditions: p ∈ T, d ≥ 1. S(p, d) = c₁, c₂, ... defined by c₁ = inc(p, d),
  cₙ₊₁ = inc(cₙ, 0).

Postconditions: (A n : n ≥ 1 : p ≼ cₙ) — every stream element extends p as a prefix.
```

---

## B0 — Irrevocability (LEMMA, corollary)

```
(A s, s' : s → s' : s.B ⊆ s'.B)
```

---

## B0★ — MultiStepIrrevocability (LEMMA, corollary)

```
(A s, s' : s →* s' : s.B ⊆ s'.B)

where s →* s' denotes the reflexive-transitive closure of the transition relation —
that is, s' is reachable from s by a finite (possibly empty) sequence of transitions.
```

---

## B-Seq — SequentialCommitment (AXIOM, model-axiom)

```
Axiom: The states realized under a single baptismal authority — one serialized commit
path — are totally ordered by →*: for any two such reachable states s, s', either
s →* s' or s' →* s.
```

---

## B0a — BaptismalClosure (AXIOM, design-requirement)

```
Σ partitions into two classes:

  Baptismal operations: For each (p, d) satisfying B6, baptize(p, d) ∈ Σ adjoins a
    single element to the registry:
      op(s).B = s.B ∪ {next(s.B, p, d)}

  s.B-frame operations: Every other op ∈ Σ preserves the registry:
      (A op ∈ Σ \ {baptize(p, d) : B6(p, d)}, s ∈ dom(op) : op(s).B = s.B)
```

---

## B₀ conf. — SeedConformance (AXIOM, design-requirement)

```
B₀ is finite,
(A p, d : B6(p, d) : children(B₀, p, d) is a contiguous prefix of S(p, d)),
and
(A t ∈ B₀ : t satisfies T4).
```

---

## B_fin — RegistryFiniteness (INV, invariant)

```
Invariant: (A s : s reachable from s_init : s.B is finite).

Base:         B₀ conf. — B₀ is finite.
Preservation: B0a — every transition either leaves s.B unchanged or replaces it by
              its union with a single element (a finite set unioned with a singleton
              is finite).
```

---

## B1 — ContiguousPrefix (INV, invariant)

```
Invariant: (A p, d : B6(p, d) :
              (A n : n ≥ 1 ∧ cₙ ∈ s.B ⟹ (A i : 1 ≤ i < n : cᵢ ∈ s.B)))

Equivalently: for every B6-valid (p, d),
  children(s.B, p, d) = {c₁, ..., cₘ}  for some m ≥ 0.

Base:         B₀ conf. — seed set satisfies contiguous prefix for every B6-valid (p, d).
Preservation: Each baptismal transition preserves B1 in the target namespace (by B0a,
              B0, S0, TA5(c), and the next definition) and in every other B6-valid
              namespace (by B7).
```

---

## B2 — HighWaterMarkSufficiency (LEMMA, postcondition)

```
Preconditions: B satisfies B1 for all B6-valid (p, d); (p, d) satisfies B6;
  S(p, d) = c₁, c₂, ... defined by c₁ = inc(p, d), cₙ₊₁ = inc(cₙ, 0).

Postconditions: next(B, p, d) = c_{hwm(B,p,d) + 1}
```

---

## B3 — GhostValidity (DEF, admissible-configurations)

```
The admissible configurations of a tumbler t ∈ T in a reachable state s are:

  - baptized and populated:  t ∈ s.B  with content stored at t
  - baptized and empty:      t ∈ s.B  with nothing stored — a ghost element (permitted)
  - unbaptized:              t ∉ s.B  — not a system entity
```

---

## B4 — AtomicBaptism (AXIOM, corollary)

```
Each baptize(p, d) ∈ Σ is a single edge of →: no transition interposes between
evaluating next(s.B, p, d) and committing the union.
```

---

## B5 — FieldAdvancement (LEMMA, postcondition)

```
Preconditions: p ∈ T with d ≥ 1.

Postconditions: zeros(inc(p, d)) = zeros(p) + (d − 1)
```

---

## B5a — SiblingZerosPreservation (LEMMA, postcondition)

```
Preconditions: t ∈ T with t_{sig(t)} > 0.

Postconditions: zeros(inc(t, 0)) = zeros(t)
```

---

## B6 — ValidDepth (PREDICATE, necessity-sufficiency)

```
Baptism at depth d from parent p is valid when:
  (i)   p satisfies T4
  (ii)  d ∈ {1, 2}
  (iii) zeros(p) + (d − 1) ≤ 3

Preconditions: p ∈ T, d ∈ ℕ with d ≥ 1.

Postconditions:
  (a) Sufficiency:
        (p satisfies T4 ∧ d ∈ {1, 2} ∧ zeros(p) + (d − 1) ≤ 3)
        ⟹ (A n ≥ 1 : cₙ ∈ S(p, d) satisfies T4)

  (b) Necessity, given a T4-valid parent (i): violating (ii) or (iii) forces a stream
      T4 violation.
```

---

## B7 — NamespaceDisjointness (LEMMA, postcondition)

```
Preconditions: (p, d) and (p', d') both satisfy B6, with (p, d) ≠ (p', d').

Postconditions: S(p, d) ∩ S(p', d') = ∅
```

---

## B8 — Uniqueness (THEOREM, two-case)

```
Distinct-namespace baptisms produce distinct addresses unconditionally;
same-namespace baptisms produce distinct addresses under a single authority:

  cross-namespace:  (A a, b : produced in distinct namespaces : a ≠ b)
  same-namespace:   (A a, b : produced in one namespace under a single authority : a ≠ b)

Formal Contract (cross-namespace — unconditional):
  Preconditions: β₁ produces a in namespace (p, d) and β₂ produces b in namespace
    (p', d') with (p, d) ≠ (p', d'), where both (p, d) and (p', d') satisfy B6;
    the system conforms to B7. No single-authority or B-Seq assumption is required.
  Postconditions: a ≠ b.

Formal Contract (same-namespace — single authority):
  Preconditions: β₁, β₂ are distinct baptismal acts in one namespace (p, d) = (p', d')
    satisfying B6, committed under a single baptismal authority (so B-Seq applies),
    in a system conforming to B-Seq, B0★ (which subsumes B0), B0a, B1, B2, and B4.
  Postconditions: a ≠ b.
```

---

## B9 — UnboundedExtent (THEOREM, postcondition)

```
(A p, d : B6(p, d) : (A M ∈ ℕ : (E s' : s →* s' via baptisms : hwm(s'.B, p, d) ≥ M)))

Preconditions: (p, d) satisfying B6(p, d); M ∈ ℕ; current state s reachable from
  s_init.

Postconditions: There exists s' with s →* s' via a finite sequence of baptismal
  transitions such that hwm(s'.B, p, d) ≥ M.
```

---

## B10 — T4ValidityInvariant (INV, invariant)

```
Invariant: (A t ∈ s.B : t satisfies T4)

Base:         B₀ conf. — every seed element satisfies T4.
Preservation: Each baptismal transition adds a = next(s.B, p, d) ∈ S(p, d); since
              (p, d) satisfies B6, B6's sufficiency result gives every element of
              S(p, d) — hence a — satisfies T4. B0a ensures no non-baptismal mechanism
              introduces elements that might violate T4.
```
