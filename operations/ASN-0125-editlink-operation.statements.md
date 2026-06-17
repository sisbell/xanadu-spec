> **ASN-0125 · The EDITLINK Operation — Editing Under Link Immutability** — condensed claim statements  
> [← Full note](ASN-0125-editlink-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0125 Claim Statements

*Source: ASN-0125-editlink-operation.md (revised 2026-06-12) — Extracted: 2026-06-13*

## Definition — Listed

```
listed(t, d, Σ)  ≡  t ∈ ran(Σ.M(d))
```
Equivalently `(t, d) ∈ Contains(Σ)` (ASN-0047, CurrentContainment). For a link, only its home can list it — a link-subspace image has `origin = d` (CL-OWN), and a content-subspace image lies in `dom(C)`, disjoint from `dom(L)` (SD).

## Definition — Active

```
active(a, Σ)  ≡  a ∉ nullified(Σ)
```

## Definition — DisciplineConformance

`DC(ℓ')` — a value-level predicate whose witnesses are drawn from the editlink pre-state `Σ`:

```
coverage(ℓ'.e₃) ≠ coverage(R)
∧  (if |ℓ'| = 3 ∧ coverage(ℓ'.e₃) = coverage(K_sup) then
      (E x, y ∈ dom(Σ.L) : x ≠ y
         ∧ ℓ'.e₁ = {(x, δ(1, #x))}
         ∧ ℓ'.e₂ = {(y, δ(1, #y))}))
```

## Definition — InSet / OutSet

```
in(y, Σ)  = {e ∈ Ŝ^Σ : old(e) = y}
out(x, Σ) = {e ∈ Ŝ^Σ : new(e) = x}
```

---

## EL0 — MutationExclusion (LEMMA, lemma)

Fix a reachable `Σ₀`, a link `a ∈ dom(Σ₀.L)`, and write `ℓ₀ = Σ₀.L(a)`. Let `R_mut ≡ a ∈ dom(L) ∧ L(a) = w` for some intended `w ≠ ℓ₀`, and `J ≡ a ∈ dom(L) ∧ L(a) = ℓ₀`.

The persistence invariant (L12 closed under `→*`, LP13 at `a`):
```
(A Σ' : Σ₀ →* Σ' : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = ℓ₀)
```
Since `[J ⟹ ¬R_mut]` and `J` holds at every reachable state from `Σ₀`:

**For every finite program `S` over the closed elementary vocabulary, `wp(S, R_mut)` evaluated at `Σ₀` is `false`.**

The original remains readable at its address with its exact value forever. Dually the original is readable at its own address, with exactly its original value, in every future state, unconditionally.

---

## EL1 — IntentInvisibility (LEMMA, lemma)

Two descriptions of the same work — "I edited the link at `a`, producing the corrected value `ℓ'` homed at `d_s`" and "I created a brand-new link with value `ℓ'` homed at `d_s`" — both denote the very same transition instance `K.λ(d_s, a_emit(Σ, d_s), ℓ')` applied to the same state, hence produce the *same* post-state `Σ₁`.

**Emission alone records no relationship: for the post-state `Σ₁` of any single link allocation, every state predicate — and hence every observation, present or future — is invariant under whether the allocation was an edit of some existing link or an independent creation.**

Consequently:
- Value resemblance carries no relational information: the store legitimately holds distinct addresses with identical values (L11b, NonInjectivity, ASN-0043).
- The state after a resembling independent creation coincides with the state after an unasserted "edit."
- There is no predicate of `Σ₁` that holds iff `ℓ'` "was derived from" `Σ.L(a)`. Intent is not a component of `Σ`, and what is not in `Σ` does not exist for any observer of `Σ`.

---

## EL2 — NoInPlaceCarrier (LEMMA, lemma)

In every reachable state, the supersession record can live:

*(a) Not in the original's value.* `Σ.L(a)` is fixed by L12 from the moment of creation. No "superseded-by" annotation can ever be attached to `a`.

*(b) Not appended to the successor's value later.* The same invariant (L12) binds the successor the instant it exists: its slots are fixed at emission.

*(c) Not in the address relation between them.* Every allocated link address is an emission of its home document's flat sibling chain `A_L(d)` — first emission `[d.0.s_L.1]` with element-field depth `#E = 2`, successors by `inc(·, 0)` which preserves length (FirstEmission, ChainDiscipline, TA5(c), ASN-0093) — and at every reachable state the homed links form a contiguous initial segment of that chain (ChainMembershipForOrigin, ASN-0093). So every allocated link address has `#E = 2` exactly, while a nested version-of-link address would need `#E ≥ 3`; stronger, `dom(Σ.L)` is a tumbler-prefix antichain (R0a, FlatLinkDomain, ASN-0086) — no allocated link address prefixes another. The address relation between any two allocated links carries exactly two readable facts: whether they share a home (T6-decidable from the prefixes), and, within one home, their emission order (T9, ASN-0034). Neither is semantic.

*(d) Not in an index marker.* The stored entities are exhausted by `dom(Σ.C) ∪ dom(Σ.L)` (L14, DualPrimitive, ASN-0043); the entity set `E` holds organizational addresses with no payload; the provenance relation `R` holds (content-address, document) pairs whose precondition `a ∈ dom(C)` excludes link targets outright; arrangement entries are V→I bindings within one document. There is no status field anywhere, and the only systematic asymmetry between two link entries is their addresses.

**Therefore the record must be a freshly allocated entity.**

---

## RQ1–RQ7 — RecordRequirements (REQ, requires)

Seven requirements on any carrier of the supersession relationship:

- **RQ1 (Post-hoc assertability).** The relationship must be assertable at any state where both endpoints exist — not only at the successor's creation.
- **RQ2 (Open authorship).** Any principal with a home document may assert.
- **RQ3 (Attribution).** The asserter must be decidable from the record alone.
- **RQ4 (Non-destructive disputability).** A claim must be withdrawable from current standing, and contestable, without erasing it or either endpoint.
- **RQ5 (Endpoint frame).** Asserting must modify neither endpoint.
- **RQ6 (Decidable specificity).** The relationship must be recognizable as supersession *specifically* — distinguishable from comment, counterpart, or coincidence — by the substrate's interpretation-free mechanisms: address and coverage comparison, never content exegesis. And it must be refinable: the vocabulary must be able to grow subtypes.
- **RQ7 (Plurality).** Arbitrarily many claims over the same endpoints, including mutually contradictory ones, must be co-representable. Competing claims are resolved socially, never structurally.

---

## EL3 — RelationSpaceNecessity (LEMMA, lemma)

**Under this substrate, any carrier satisfying RQ1–RQ7 is a freshly allocated link-store entity, distinct from both endpoints, referencing each endpoint by address through its endsets, and bearing its kind as the coverage class of a designated slot — that is, a typed link-to-link tuple.**

Derivation:
- RQ1 and RQ2 require the carrier to be created by a transition at arbitrary later states by arbitrary principals → fresh store entity; vocabulary offers exactly `K.α` (content) and `K.λ` (links); `K.δ` and `K.ρ` closed off in EL2(d).
- RQ6 eliminates the content store: a claim encoded as content bytes has no structure beyond an address and an origin — its relational content lives in `Val`, which nothing in the system interprets; type machinery reads slot-3 *coverage* only (L8, TypeByAddress, ASN-0043). So the carrier is a link.
- RQ1 makes it a link *other than the successor* — a third entity — since the successor's slots close at its birth (EL2(b)).
- Reference to endpoints must be substrate-visible; the one mechanism links have for referencing anything is endset coverage; endsets may target link addresses (L4(c), ASN-0043), with the unit-depth span at an address as the canonical reference (L13, R5).
- RQ6 fixes the kind mechanism: coverage class of the type slot (L8; decidability by CoverageEqualityDecidable, ASN-0086; refinement by prefix containment, L10).
- RQ4 is satisfied because the carrier has its own address: it can be individually targeted while L12 holds it and both endpoints in the permanent record.
- RQ3 is its home prefix (T4b projection, decidable by T6).
- RQ5 is `K.λ`'s frame.
- RQ7 is freshness: every claim is a new address.

"A separate supersession link" and "a typed relation distinct from these" are the same architecture: under L8 a link *is* typed by its third endset's coverage, and a typed relation's tuples *are* links (ASN-0086, TypedRelation). The address-space candidate is closed outright by EL2(c).

---

## Df-CLS — SupersessionClass (DEF, predicate)

Fix a coverage class `[K_sup]`, `K_sup ∈ T_admissible`, with `coverage(K_sup) ≠ coverage(R)` — distinct from the retraction class (ASN-0086).

```
S^Σ       := L_{K_sup}^Σ
           = {(b, F, G) : b ∈ dom(Σ.L) ∧ |Σ.L(b)| = 3
                        ∧ Σ.L(b).e₁ = F ∧ Σ.L(b).e₂ = G
                        ∧ coverage(Σ.L(b).e₃) = coverage(K_sup)}

A_sup^Σ   := A_{K_sup}^Σ = {(b, F, G) ∈ S^Σ : b ∉ nullified(Σ)}
```

Members of `S^Σ` are called *claims*. `A_sup^Σ` is the operative subset.

---

## Df-DIR — ClaimDirectionality (DEF, predicate)

For a claim `(b, F, G) ∈ S^Σ`: the from-set `F` covers the *superseding* link, the to-set `G` the *superseded* — read "`F` replaces `G`."

This aligns with the layer's RetractionDirectionality (ASN-0086): the to-side is the side acted upon.

A withdrawal with no replacement is not a degenerate supersession but a retraction, class `[R]`; the two acts remain distinct relations, and asserting the first never performs the second.

---

## Df-DISC — EditDiscipline (DEF, predicate)

A state `Σ` is *edit-disciplined* iff:

*(i)* it is unit-depth-retraction-disciplined (ASN-0086); and

*(ii)* every claim conforms to the *claim schema*:
```
(A (b, F, G) ∈ S^Σ :
   (E x, y ∈ dom(Σ.L) : x ≠ y
      ∧ F = {(x, δ(1, #x))}
      ∧ G = {(y, δ(1, #y))}))
```
— both endsets are canonical unit-depth spans at link-store addresses, and the claim is irreflexive.

A layer is edit-disciplined iff every state it reaches is.

(Self-supersession `x = y` is excluded as vacuous; cycles of length ≥ 2 are deliberately *not* excluded — they are reverts.)

---

## Df-LAY — EditingLayer (DEF, predicate)

The *editing layer* issues exactly the operations `{assert_sup, editlink, Nullify}` together with the substrate's link-framing transitions `{K.α, K.δ, K.μ⁺, K.μ⁺_L, K.μ⁻, K.ρ}` (and `K.μ~`, their composite) and the *bare* `K.λ` — confined to original-link creation: emission whose slot-3 coverage is neither `coverage(K_sup)` nor `coverage(R)`.

Its one *discipline commitment*: every `[K_sup]` emission routes through `assert_sup` or `editlink` (under `DC`), every `[R]` emission through `Nullify`.

A state is *editing-layer-reachable* iff it is reached from the initial state `Σ₀` by a finite sequence of editing-layer operations.

---

## EL-DM — DisciplineMaintenance (LEMMA, lemma)

**Every editing-layer-reachable state is edit-disciplined.**

*Base.* The initial state `Σ₀` (ASN-0047) has `L₀ = ∅`, hence `S^{Σ₀} = L_{K_sup}^{Σ₀} = ∅` and `L_R^{Σ₀} = ∅`; both clauses of Df-DISC hold vacuously over an empty link store.

*Step.* Discharge for each editing-layer operation:

- *L-framing transitions and original-creating bare `K.λ`.* By Vocabulary fact V the six framing transitions (and `K.μ~`) carry `L' = L`, leaving `S^Σ` and `L_R^Σ` identical. The bare original-creating `K.λ` extends `dom(L)` at one fresh key whose slot-3 coverage inhabits neither disciplined class, adding nothing to either slice; every prior tuple keeps its value (L12) and witnesses remain in the grown `dom(L)`.

- *`Nullify`.* Emits exactly one `[R]`-class tuple with to-set `{(t, δ(1, #t))}` where `t ∈ dom(Σ'.L)` at the post-state; deposits no `[K_sup]` claim, leaving clause (ii) untouched.

- *`assert_sup`.* EL6(v): `Σ'` is edit-disciplined when `Σ` is.

- *`editlink`.* EL7(vi): `Σ₂` is edit-disciplined when `Σ` is.

By induction over the editing-layer operations, edit-discipline holds at every editing-layer-reachable state.

---

## EL4 — SingleTarget (LEMMA, lemma)

For `e = (b, F, G) ∈ S^Σ` whose endsets meet the Df-DISC(ii) form with witnesses `x, y` (so `F = {(x, δ(1,#x))}`, `G = {(y, δ(1,#y))}`, `x, y ∈ dom(Σ.L)`, `x ≠ y`):

```
coverage(F) ∩ dom(Σ.L) = {x}
coverage(G) ∩ dom(Σ.L) = {y}
```

*Proof sketch.* `coverage({(x, δ(1, #x))}) = {t : x ≼ t}` (PrefixSpanCoverage, ASN-0043); for `t ∈ dom(Σ.L)` with `x ≼ t`, the antichain R0a forces `t = x`.

The hypothesis is schema-conformance of `e` alone, not edit-discipline of `Σ`. We may therefore write `addr(e) = b`, `new(e) = x`, `old(e) = y` as total accessors on any schema-conforming claim at any reachable state.

Write `Ŝ^Σ = {e ∈ S^Σ : e is schema-conforming}` for the schema-conforming claims; at an edit-disciplined state every claim conforms, so `Ŝ^Σ = S^Σ`.

---

## Df-SUCC — SuccessorRelations (DEF, function)

At any state `Σ`, ranging over the schema-conforming claims `Ŝ^Σ` (EL4):

```
succ_h(Σ) = {(old(e), new(e)) : e ∈ Ŝ^Σ}
succ_o(Σ) = {(old(e), new(e)) : e ∈ Ŝ^Σ ∧ addr(e) ∉ nullified(Σ)}
```

Both are finite (L-fin) relations on `dom(Σ.L)`, with `succ_o(Σ) ⊆ succ_h(Σ)`; at editing-layer states `Ŝ^Σ = S^Σ` (EL-DM), so the comprehensions range over the whole supersession slice.

---

## EL5 — RecordMonotonicity (LEMMA, lemma)

For every `Σ →* Σ'`:

*(a)* `S^Σ ⊆ S^{Σ'}`, `Ŝ^Σ ⊆ Ŝ^{Σ'}`, and `succ_h(Σ) ⊆ succ_h(Σ')`.

The slice inclusion is R3 (TypedSliceMonotonicity, ASN-0086) at `[K_sup]`, lifted across `→*` by finite composition; schema-conformance rides along, being value-and-domain-determined — a conforming claim's witnesses satisfy `x, y ∈ dom(Σ.L) ⊆ dom(Σ'.L)`. Claims accumulate; none is ever lost.

*(b)* `nullified(Σ) ⊆ nullified(Σ')`.

The `[R]`-slice likewise only grows, so nullification is one-way (R6a, ASN-0086): a claim once retracted from operative standing never silently regains it (re-assertion is a *new* claim at a fresh address — the shape of R6c).

*(c)* `succ_o` is neither monotone nor antitone: emission adds operative pairs (EL6), `Nullify` removes them. The operative relation is the one revisable view; the historical relation is the unrevisable record.

---

## ASSERTop — AssertSup (DEF, function)

Precondition: `x, y ∈ dom(Σ.L) ∧ x ≠ y ∧ d_a ∈ dom(Σ.M)`

```
assert_sup(x, y, d_a) ≜ Emit_{K_sup}(Σ, d_a, {(x, δ(1, #x))}, {(y, δ(1, #y))})
```

One `K.λ` at home `d_a`, emitting the claim "`x` supersedes `y`" at the fresh address `b = a_emit(Σ, d_a)`. The spans are T12-well-formed (`Pos(δ(1, #x))`; action point `#x ≤ #x`), the slots are endsets, the arity is 3, and slot 3 is `K_sup ≠ ∅`, so `K.λ`'s L3 precondition is discharged.

---

## EL6 — AssertionContract (LEMMA, lemma)

When invoked at a reachable `Σ` satisfying its precondition, `assert_sup(x, y, d_a)` yields `Σ'` with:

*(i) Allocation.* Exactly one fresh address: `b ∉ dom(Σ.L) ∪ dom(Σ.C)` (emission freshness, ASN-0093), `home(b) = d_a`.

*(ii) Record.* `e_b = (b, {(x, δ(1,#x))}, {(y, δ(1,#y))}) ∈ S^{Σ'}` with `new(e_b) = x`, `old(e_b) = y`; hence `(y, x) ∈ succ_h(Σ')`.

*(iii) Active at birth.* If `Σ` is edit-disciplined, `b ∉ nullified(Σ')`, so `(y, x) ∈ succ_o(Σ')`. (ASN-0086 wp Case 2 under disciplined simplification; `K_sup ≁ R` discharges the self-nullification guard.)

*(iv) Frame and activity.*
```
Σ'.C = Σ.C,  Σ'.M = Σ.M,  Σ'.E = Σ.E,  Σ'.R = Σ.R
```
Every prior link-store entry is unchanged.

*Unconditionally:* `nullified(Σ') ∩ dom(Σ.L) = nullified(Σ)` — the lone new tuple has slot-3 coverage `coverage(K_sup) ≠ coverage(R)`, so the `[R]`-slice does not grow and the nullification status of no pre-existing address changes.

*Under edit-discipline on `Σ`:* the full `nullified(Σ') = nullified(Σ)` follows — the only candidate new member is the fresh `b`, and the unit-depth retraction discipline together with the antichain R0a discharges wp Case 2's third conjunct.

*(v) Discipline and permanence.* `Σ'` is edit-disciplined when `Σ` was. At every `Σ' →* Σ''`, `e_b ∈ S^{Σ''}` with value fixed and `(y, x) ∈ succ_h(Σ'')` (EL5a).

---

## EDITop — Editlink (DEF, function)

Precondition: `a ∈ dom(Σ.L) ∧ d_s, d_a ∈ dom(Σ.M) ∧ ℓ' L3-conforming ∧ DC(ℓ')`

```
editlink(a, ℓ', d_s, d_a) ≜
  a' := a_emit(Σ, d_s);
  Σ₁ := K.λ(d_s, a', ℓ');
  (Σ₂, b) := assert_sup(a', a, d_a) at Σ₁;
  return (Σ₂, a', b)
```

`assert_sup`'s precondition is discharged at `Σ₁`: `a' ∈ dom(Σ₁.L)` by the emission, `a ∈ dom(Σ₁.L)` by monotonicity, `a' ≠ a` by freshness, `d_a ∈ dom(Σ₁.M)` by M1.

Additional notes:
- `ℓ' = Σ.L(a)` (value-identical successor) is admitted.
- Neither `d_s` nor `d_a` is constrained relative to `home(a)` — third-party edit-by-fork is the same composite.
- A *revert* is `assert_sup(a, a', d)` alone — no new successor allocation.

---

## EL7 — EditContract (LEMMA, lemma)

When invoked at a reachable `Σ` satisfying its precondition, `editlink(a, ℓ', d_s, d_a)` yields `Σ₂` with:

*(i) What is allocated.* Exactly **two** fresh link-subspace addresses — the successor `a'` on `A_L(d_s)` and the claim `b` on `A_L(d_a)` — pairwise distinct from each other and from everything prior.
```
Σ₂.C = Σ.C,  Σ₂.M = Σ.M,  Σ₂.E = Σ.E,  Σ₂.R = Σ.R
```

*(ii) The new reading.* `Σ₂.L(a') = ℓ'`, `home(a') = d_s`. The successor is *born unlisted*: `a' ∉ ran(Σ₂.M(d))` for every `d`.

*(iii) The relationship.* `(a, a') ∈ succ_h(Σ₂)`, witnessed by the claim at `b`; at edit-disciplined `Σ`, also `(a, a') ∈ succ_o(Σ₂)`.

*(iv) The frame on the original.* `Σ₂.L(a) = Σ.L(a)` unconditionally (L12); the original's listing is untouched (both steps frame `M`). Under edit-discipline on `Σ`:
```
nullified(Σ₂) = nullified(Σ)
```
(`DC(ℓ')` bars a retraction-class successor; both fresh addresses escape pre-existing retraction coverage by freshness + R0a.)

*(v) Permanence.* At every `Σ₂ →* Σ₃`: `a`, `a'`, `b` all persist with fixed values, and `(a, a') ∈ succ_h(Σ₃)`.

*(vi) Discipline preservation.* `Σ₂` is edit-disciplined when `Σ` is — secured by `DC(ℓ')` for step 1 and EL6(v) for step 2.

Step 1 (`K.λ(d_s, a', ℓ')`): every prior claim keeps its witnesses (`x, y ∈ dom(Σ.L) ⊆ dom(Σ₁.L)`, values fixed by L12). The new value `ℓ'` at `a'` is governed by `DC(ℓ')`, whose witnesses are pinned at the pre-state `dom(Σ.L)`. The successor `a'` is a claim at `Σ₁` — member of `S^{Σ₁}` — iff `|ℓ'| = 3 ∧ coverage(ℓ'.e₃) = coverage(K_sup)`, exactly `DC`'s schema guard; in that case `DC(ℓ')` supplies the conforming witnesses. Otherwise `a' ∉ S^{Σ₁}` and Df-DISC(ii) holds vacuously on `a'`. In every case step 1 adds no `[R]`-tuple. Step 2, `assert_sup`, preserves edit-discipline by EL6(v).

---

## EL8 — ClaimStanding (LEMMA, lemma)

For every claim `e ∈ S^Σ` in a disciplined state:

*(a)* It is permanent in membership and value (EL5a).

*(b)* It is attributed:
```
home(addr(e)) = N(addr(e)).0.U(addr(e)).0.D(addr(e))
```
computable from the address alone by field projection (T4b), decidably (T6), identifying the document under whose prefix the claim was allocated. The substrate carries no principal set; resolving a home to a named owner is the office of an ownership layer (ASN-0042).

*(c)* It is open: the schema imposes no relation among `home(addr(e))`, `home(old(e))`, `home(new(e))` — first-party, second-party, and third-party claims are structurally identical.

*(d)* It is itself addressable: `addr(e) ∈ dom(Σ.L)`, so claims can be the targets of endsets (L4(c)) — endorsed, disputed, commented, retracted (`Nullify`), or themselves edited (`editlink` applies to a claim, `DC` permitting) — with no new machinery.

---

## EL9 — ThreeAxes (LEMMA, lemma)

For a link `a ∈ dom(Σ.L)`:

*(1) Resolution — permanent and unconditional.*
```
(A Σ' : Σ →* Σ' : a ∈ dom(Σ'.L) ∧ Σ'.L(a) = Σ.L(a))
```
Nothing gates the lookup: no arrangement state, no activity status, no provenance appears in it.

*(2) Listing — mutable in both directions.* `listed(a, d, Σ)` (only at `d = home(a)`, CL-OWN) can be changed in both directions:

*De-listing construction.* Let `a` sit at `[s_L, j]` in `V_{s_L}(d) = {[s_L, 1], …, [s_L, n]}`. Apply `K.μ⁻` with `n'_{s_L} = j − 1` (drops `a` together with the whole suffix `ℓ_{j+1}, …, ℓ_n`), then re-seat the `n − j` survivors in their old order by successive `K.μ⁺_L`, landing each one position below its prior seat. The result has `a` gone from `ran(M(d))`.

Re-listing `a` is then one `K.μ⁺_L` with `origin(a) = d ∧ a ∉ ran(M(d))`.

*(3) Activity — monotone downward, per claim.* `active(a, Σ) ≡ a ∉ nullified(Σ)` can only fall (EL5b), by an explicit, itself-permanent, itself-attributed retraction tuple; restoration is re-assertion at a fresh address, never reinstatement in place (R6c).

**The axes are independent, and — by EL6(iv) — superseding moves none of them.** Each demotion is a separate act by an authorized party, separately attributed, separately permanent in the record.

---

## EL10 — PositionEpochality (LEMMA, lemma)

**There exist reachable `Σ →* Σ' →* Σ''`, a document `d`, a position `v`, and links `ℓ₁ ≠ ℓ₂` — both permanently resolvable throughout — with:**

```
Σ.M(d)(v) = ℓ₁,   v ∉ dom(Σ'.M(d)),   Σ''.M(d)(v) = ℓ₂
```

*Construction.* Let `d` list two links, `V_{s_L}(d) = {[s_L,1], [s_L,2]}` with `[s_L,2] ↦ ℓ₁`, and let `ℓ₂` be homed at `d` but unlisted. Apply `K.μ⁻` with link-subspace retention `n'_{s_L} = 1`; then `K.μ⁺_L` for `ℓ₂`: the substrate assigns `v_ℓ = shift(max(V_{s_L}(d)), 1) = shift([s_L,1], 1) = [s_L,2]` — the very position `ℓ₁` vacated, now bound to `ℓ₂`.

Addresses never re-bind: every allocation is fresh (Vocabulary fact V) and every binding is permanent (L12). Therefore surviving references must bind addresses, never positions — the claim schema Df-DISC(ii) complies with this, and the construction shows the compliance is load-bearing, not stylistic.

---

## EL11 — TwoRegimeDiscovery (LEMMA, lemma)

A schema-conforming claim `e ∈ Ŝ^Σ` is findable in two ways:

*(a) Contextual (arrangement-gated).* For a schema-conforming claim `e` and any document `d`:
```
project(Σ.L(addr(e)).e₂, d, Σ) ≠ ∅  ⟺  listed(old(e), d, Σ)
```

*Proof sketch.* By LP12 (ASN-0098): left side is `coverage(G) ∩ ran(Σ.M(d)) ≠ ∅` with `G` the to-set. `coverage(G) = {t : old(e) ≼ t}` (EL4's computation). Any member of `ran(Σ.M(d))` lies in `dom(Σ.C) ∪ dom(Σ.L)` (S3★). No content address extends `old(e)` (subspace mismatch, SC-NEQ, L0). A link address extends `old(e)` only if equal (R0a). So the intersection is `{old(e)} ∩ ran(Σ.M(d))`, nonempty iff `old(e)` is listed — and by Df-LISTED only at `d = home(old(e))`.

Symmetrically for the from-side and `new(e)`.

*(b) Archival (arrangement-independent).* The predicates `e ∈ Ŝ^Σ` and `old(e) = y` are functions of stored values, decidable by coverage comparison (CoverageEqualityDecidable; T2 span membership; EL4). The sets
```
in(y, Σ)  = {e ∈ Ŝ^Σ : old(e) = y}
out(x, Σ) = {e ∈ Ŝ^Σ : new(e) = x}
```
are computable directly from `Σ.L` alone — consulting no arrangement — at every state and for any `y, x`.

**The supersession question is answerable, completely and decidably, at every state, whatever every arrangement says.**

---

## EL12 — ForkPermanence (LEMMA, lemma)

**Two editors independently superseding the same link produce a permanent, co-visible fork.**

Run `editlink(a, ·, ·, ·)` twice from any disciplined reachable state, in any combination of homes: freshness yields distinct successors `a'₁ ≠ a'₂` and distinct claims `e₁ ≠ e₂` (same home: the chain advances past the first emission; different homes: cross-document disjointness with T10); both `(a, a'₁)` and `(a, a'₂)` enter `succ_h` — permanently (EL5a) — and `succ_o` at birth (EL6(iii), the second invocation's active-at-birth conclusion resting on EL7(vi): the first `editlink` leaves the intermediate state edit-disciplined).

The vocabulary contains no transition that merges, ranks, or removes either. The complete competing-claim set, with asserters, is one archival query: `in(a, Σ)`.

Conversely — without the assertion steps the same two emissions leave `succ_h` untouched (EL1): the "fork" of intentions never existed in state. **Fork visibility is exactly assertion-deep.**

---

## EL13 — TemporalErasure (LEMMA, lemma)

**Cross-home claim order is not a fact of the state.** For `d₁ ≠ d₂ ∈ dom(Σ.M)` and values `v₁, v₂`, the two interleavings of emissions commute to the same state:

```
K.λ(d₂, a_emit(·, d₂), v₂) ∘ K.λ(d₁, a_emit(·, d₁), v₁) (Σ)
  = K.λ(d₁, a_emit(·, d₁), v₁) ∘ K.λ(d₂, a_emit(·, d₂), v₂) (Σ)
```

*Proof.* `a_emit(Σ', d)` depends only on the `d`-homed subset of `dom(Σ'.L)` (ASN-0086, EmitAddress, with HomeOriginCoincidence); an emission homed at `d₁` leaves the `d₂`-homed subset unchanged, so each address is the same in both orders; the enabledness of each step consults only its own home's set and `dom(M)`; and the two map-unions at distinct fresh keys commute, all other components being framed.

Consequently no function of the final state distinguishes which of two cross-home claims was asserted later. Within one home the opposite holds: the chain enumeration is strictly increasing (T9; ChainEnumerationInjectivity, ASN-0093), so claims homed at one document are totally ordered by their addresses — *per-home* "latest" is well-defined and state-recoverable — per-document-chain, not per-principal.

---

## Df-CUR — CurrencyQuery (DEF, function)

For `y ∈ dom(Σ.L)`:

```
reach_o(y, Σ) = least subset of dom(Σ.L) containing y
                and closed under succ_o(Σ)-images
```
(finite and computable — the closure grows within finite `dom(Σ.L)`; bound function `|dom(Σ.L)| − |computed set|`)

```
current(y, Σ) = {z ∈ reach_o(y, Σ) : ¬(E w :: (z, w) ∈ succ_o(Σ))}
```

The sink test `¬(E w :: (z, w) ∈ succ_o(Σ))` reads only the *operative claims* out of `z` (a claim-activity filter on `addr(e)`), not `z`'s own activity.

---

## EL14 — CurrencyRelational (LEMMA, lemma)

`current` is a total, computable, *set-valued* query, and the set is irreducibly a set:

*(a)* `|current(y, Σ)| = 1` at states with one asserted, unretracted, linear chain from `y`. And `current(y, Σ) = {y}` when `y` has no operative successor: an unedited link is its own current version.

*(b)* `|current(y, Σ)| ≥ 2` at any fork state (EL12).

*(c)* `current(y, Σ) = ∅` is reachable: assert `x` supersedes `y`, then assert `y` supersedes `x`. Both claims are permanent; while both are operative, `reach_o(y) = {y, x}` has no sink. The operative record says "each replaces the other." The repair is `Nullify` on one claim — not deletion — and a sink reappears. The two-view structure (operative vs. historical) makes the standoff survivable.

*(d)* No state-definable selector canonically identifies the latest edit. A temporal/recency selector is not a state function across homes (EL13). A non-temporal tie-break (e.g., T1-least claim address) is a function of `Σ` but ranks namespaces, not times — definable, yet not canonical. Making `|current| = 1` an invariant would require refusing well-formed emissions or erasing claims; the substrate does neither. **The layer owes disclosure, not decision:** `current(y, Σ)` entire, each member with its supporting claims and their homes (EL8b) and its own activity status, the original always still readable beside them (EL9(1)), narrowing applied as the reader's declared policy.

*(e) Activity-agnostic membership.* `current(y, Σ)` is built from `succ_o(Σ)`, whose only activity filter is the *claim*-address test `addr(e) ∉ nullified(Σ)` (Df-SUCC) — never a test on the endpoint links `old(e)`, `new(e)`. Therefore:

```
z ∈ current(y, Σ)   does not imply   active(z, Σ)
```

A successor may be `Nullify`'d as an endpoint while its claim stands; `(a, a') ∈ succ_o` and `current(a) = {a'}` then coexist with `a' ∈ nullified`. Sink membership and member activity are independent axes (EL9(3)). `current` answers *which readings are supersession-sinks under the operative claims*, not *which readings are themselves operative*.

---

## EL15 — ChainConnectivity (LEMMA, lemma)

For a chain of asserted edits `a₀, a₁, …, aₙ` with each `(aᵢ, aᵢ₊₁) ∈ succ_h(Σ)`:

*(a)* Every member is permanently resolvable at its own address with its original value (EL0).

*(b)* Every asserted hop is permanently in `succ_h` (EL5a), so *historical connectivity is monotone non-decreasing*: the `succ_h`-component of any member never loses a node or an edge at any future state.

*(c)* Every hop is locally recoverable from either endpoint alone — `in(aᵢ, Σ)` and `out(aᵢ, Σ)` are single arrangement-free observations (EL11b) — so the historical component is traversable edge-by-edge in both directions from any member, with no privileged entry point.

*(d)* What is *not* guaranteed:
- *Completeness:* an edit whose author omitted the assertion contributes no hop (EL1 — the relationship does not exist, and resemblance cannot reconstruct it).
- *Operative integrity:* a nullified claim drops from `succ_o` while remaining in `succ_h`.

Member-to-ends traversability of the *operative* chain is therefore a derived property — holding exactly when the chain was fully asserted and no hop demoted.

---

## EL16 — ReferenceSurvival (LEMMA, lemma)

Let `c ∈ dom(Σ.L)` be any link with `a ∈ coverage(Σ.L(c).eᵢ)` for some slot `i` — a pre-existing reference to the original, made by anyone, anywhere. Across `editlink(a, ℓ', d_s, d_a)` and arbitrary further evolution `Σ →* Σ'`:

*(i)* The referring slot is unchanged in value and coverage (L12; LP2, LP3★):
```
Σ'.L(c).eᵢ = Σ.L(c).eᵢ
```
Nobody's context is rewritten by someone else's edit.

*(ii)* The referent still resolves, to the identical value:
```
Σ'.L(a) = Σ.L(a)  (EL0)
```
The reference means today what it meant when made.

*(iii)* The road forward exists and is one observation long: `in(a, Σ') ∋ e` with `new(e) = a'`, attributed to `home(addr(e))`. The reference reaches the successor not by being re-pointed but by *composition with the record*.

Against this, the two rejected regimes fail:
- **Mutation** (excluded by EL0): would preserve the reference's *spelling* while silently re-pointing its *meaning* — every citation, comment, and dispute attached to `a` would qualify content its authors never saw.
- **Silent re-creation** (step 1 without step 2): passes (i) and (ii) vacuously and fails (iii) — the successor exists, fresh and disconnected, indistinguishable from a stranger (EL1); old references keep their exact referent and gain no road forward.

**The asserted edit is the unique regime preserving both the exact past and the reachable future.**
