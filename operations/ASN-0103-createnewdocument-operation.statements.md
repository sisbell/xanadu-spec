> **ASN-0103 · CREATENEWDOCUMENT Operation** — condensed claim statements  
> [← Full note](ASN-0103-createnewdocument-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0103 Claim Statements

*Source: ASN-0103-createnewdocument-operation.md (revised 2026-06-04) — Extracted: 2026-06-08*

## Definition — DocumentChainFrontier

`D_A = {e ∈ E : Document(e) ∧ parent(e) = A ∧ #e = #A + 2}`

The length-restricted document-chain frontier beneath account `A`. Versions satisfy `Document(·) ∧ parent(·) = A` but carry length `≥ #A + 3`; the length filter `#e = #A + 2` excludes them. Equivalently: `D_A = E ∩ S(A, 2)` (proven in Effect One via T4b unique-parse and SiblingStream canonical form `[A, 0, n]`).

---

## CND.def — CreateNewDocumentDef (DEF, definition)

CREATENEWDOCUMENT(A) is a substrate composite Σ →* Σ' under ValidComposite★ (ASN-0047) realised as a single K.δ firing (case (ii): k=2 off A when D_A=∅, else k=0 off max(D_A)) registering d into E_doc with M(d)=∅; it returns d

---

## CND.pre — CreateNewDocumentPre (PRE, requires)

Preconditions: A ∈ E ∧ Account(A); the invoking principal π owns the account (pfx(π) ≼ A, O1; ASN-0042). No content argument

---

## CND.A-act — AccountActivation (AXIOM, axiom)

Standing assumption owed by (out-of-scope) account provisioning: A ∈ E ∧ Account(A) ⟹ Activated(A_doc(A))

---

## CND.alloc — DocumentAlloc (POST, ensures)

Allocates exactly one fresh document address d from A_doc(A)=S(A,2): d = inc(A,2) if D_A=∅ else inc(max(D_A),0), where D_A = {e ∈ E : Document(e) ∧ parent(e)=A ∧ #e=#A+2} is the length-restricted document-chain frontier (versions, length ≥ #A+3, excluded); with Document(d), zeros(d)=2, parent(d)=A, T4-valid(d), d ∉ E

---

## CND.empty — EmptyArrangement (POST, ensures)

M'(d) = ∅: dom(M'(d)) = ∅ and ran(M'(d)) = ∅ — the new document holds no V-positions, no V→I mappings, no content; in particular it references no I-address and so shares none with any document at Σ' (later sharing by transclusion/COPY is permitted — S5, ASN-0036 — and out of scope)

---

## CND.C-frame — ContentStoreFrame (FRAME, ensures)

C' = C: the content store is entirely unchanged — no byte added, no value altered. Creation adds a place, not content (ghost element)

---

## CND.L-frame — LinkStoreFrame (FRAME, ensures)

L' = L: the link store is unchanged

---

## CND.R-frame — ProvenanceFrame (FRAME, ensures)

R' = R: the provenance relation is unchanged

---

## CND.E — EntitySetEffect (POST, ensures)

E' = E ∪ {d} with d ∉ E: every existing entity persists (E ⊆ E') and the document population grows by exactly one (|E'_doc| = |E_doc| + 1)

---

## CND.doc-frame — DocumentArrangementFrame (FRAME, ensures)

(A d' ∈ E_doc : M'(d') = M(d')): every existing document's arrangement is wholly untouched

---

## CND.monotone — AddressMonotonicity (LEMMA, lemma)

d is never a reuse and stays distinct from every other document address, present and future: d ∉ E (established uniformly over all of E in Effect One), distinctness from every other document address (same-chain, version chains, other accounts) by GlobalUniqueness (ASN-0034 — distinct allocation events never collide); existing addresses remain valid by permanence T8 (ASN-0034)

---

## CND.subAlloc — SubAllocatorActivation (POST, ensures)

Creation activates A_C(d) and A_L(d) (content and link sub-allocators, anchors [d.0.s_C], [d.0.s_L]) without emission; both subspaces are available but empty at Σ' (SubAllocatorBundle, ASN-0047)

---

## CND.own — StructuralOwnership (LEMMA, lemma)

Ownership is structural (derivable over (C,L,E,M,R)): parent(d)=A and A ≼ d (every A_doc(A) emission has form [A,0,j]), so with pfx(π) ≼ A (CND.pre) and prefix transitivity, owns(π,d) ≡ pfx(π) ≼ d (O1; ASN-0042) — d ∈ odom(π)

---

## CND.refer — ImmediateReferability (LEMMA, lemma)

d is immediately, permanently, and unambiguously referable: a link may target d at Σ' before any content exists; uniqueness is decentralised (B8, ASN-0040) and identity is immutable for the life of the system

---

## CND.atomicity — OperationAtomicity (THEOREM, lemma)

The single-K.δ decomposition is atomic by the sequential-transition axiom (ASN-0093); no observable intermediate state exists, so all invariants hold throughout. Coupling constraints J0, J1★, J1'★ hold vacuously

---

## CND.inv — PostStateInvariants (INV, predicate)

Σ' satisfies ExtendedReachableStateInvariants (ASN-0047) and the transition invariant P3; verified directly for {P0, P1, M0, S2, S3★, P6, P8, S7d, ActivatedEmission, T8}, vacuous on dom(M'(d))=∅ for the arrangement family, frame-inherited otherwise
