> **ASN-0123 · The CREATENEWVERSION Operation** — condensed claim statements  
> [← Full note](ASN-0123-createnewversion-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0123 Claim Statements

*Source: ASN-0123-createnewversion-operation.md (revised 2026-06-12) — Extracted: 2026-06-14*

## Definition — VersionNamespace

The **version namespace** of a document `d` is the sibling stream at depth 1:

> `S(d, 1) = c₁, c₂, c₃, …` where `c₁ = inc(d, 1)` and `cₙ₊₁ = inc(cₙ, 0)`,
> whose n-th member is the tumbler `cₙ = [d₁, …, d_{#d}, n]`.

`d·k` denotes the length-`(#d + 1)` extension of `d` by final component `k`, so `cₖ = d·k`. This is the namespace ASN-0047 names `A_v(d)`, the version sub-allocator of `d`.

## Definition — NextFrontier

`next(E, p, g) = inc(p, g)` when `E ∩ S(p, g) = ∅`, else `inc(max(E ∩ S(p, g)), 0)`

---

## PS — PrincipalStructure (ASSUMPTION, axiom)

Every reachable docuverse state carries an ASN-0042-conforming principal structure:

> (i) *Dynamics* — `Π` and `pfx` satisfy O1a (account-tier prefixes) and O1b (prefix injectivity), and evolve per O12, O13, O15 across every atomic transition: principals persist, no prefix ever changes, and at most one principal enters per transition, by delegation.
> (ii) *Authority* — `allocated_by` attaches to K.δ: every entity-creating step is performed by some existing principal (O16) inside its own domain (O5).
> (iii) *Bootstrap coverage* — some `π₀ ∈ Π₀` covers the bootstrap node: `pfx(π₀) ≼ n₀`.
> (iv) *Incumbency* — every principal occupies a baptized entity: `pfx(π) ∈ E` for every `π ∈ Π`.

Consequence: **`ω : E → Π` is total and single-valued at every reachable state**.

---

## trunc — SingleComponentTruncation (DEF, function)

For `t ∈ T` with `#t ≥ 2`, `trunc(t)` is the tumbler of length `#t − 1` agreeing with `t` everywhere it is defined:

> `#trunc(t) = #t − 1  ∧  (A i : 1 ≤ i ≤ #t − 1 : trunc(t)ᵢ = tᵢ)`

Membership `trunc(t) ∈ T` is immediate from T0. For every member of a version stream:

> `v ∈ S(d, 1) ⟹ trunc(v) = d`

---

## Z-mono — ZeroCountMonotonicity (LEMMA, lemma)

Prefixing cannot lose zeros:

> `(A p, q ∈ T : p ≼ q : zeros(p) ≤ zeros(q))`

since `q` agrees with `p` on positions `1 … #p` — so every zero of `p` is a zero of `q` — and `q`'s further positions can only contribute more.

---

## SA — StoredAddressAntichain (INV, predicate)

No stored address extends another: at every reachable state `dom(C) ∪ dom(L)` is an antichain under `≼`, so for every stored `a`

> `{t ∈ T : a ≼ t} ∩ (dom(C) ∪ dom(L)) = {a}`

---

## nextv, nextd — VersionFrontier, DocumentFrontier (DEF, function)

The next unallocated member of a document-producing namespace, given the registry — for the version namespace of a document `d`, and for the account-document namespace of an account-tier principal `π`:

> `nextv(E, d) = next(E, d, 1)`  and  `nextd(E, π) = next(E, pfx(π), 2)`

with `next` as in ASN-0040: `next(E, p, g) = inc(p, g)` when `E ∩ S(p, g) = ∅`, else `inc(max(E ∩ S(p, g)), 0)`.

Derived frontier identity (from VN-B1 and S0):

> `next(E, p, g) = c_{hwm(E, p, g) + 1}`, in particular
> `nextv(E, d) = c_{hwm(E, d, 1) + 1}` and `nextd(E, π) = c_{hwm(E, pfx(π), 2) + 1}`

---

## VN-B1 — VersionNamespaceContiguity (INV, predicate)

For every B6-valid document-producing namespace `(p, g)` with `g ∈ {1, 2}` and `zeros(p) + (g − 1) = 2` — covering version streams `S(d, 1)` (`zeros(d) = 2`, `g = 1`) and account-document streams `S(pfx(π), 2)` (`zeros(pfx(π)) = 1`, `g = 2`) — at every reachable state:

> `E ∩ S(p, g) = {c₁, …, c_m}` for some `m ≥ 0` — the realized children are a contiguous prefix of the stream.

K.δ's freshness and operand constraints admit only frontier arrivals; each arriving case produces `{c₁, …, c_{m+1}}`, preserving contiguity.

---

## VERSION — Version (DEF, operation)

```
VERSION(π, d_src)

Preconditions
  P-src    d_src ∈ E_doc                (the source is an allocated document)
  P-prin   π ∈ Π                        (the forking principal)
  P-bdy    Σ is a composite boundary
  P-tier   ω(d_src) = π  ∨  zeros(pfx(π)) = 1

Abbreviations (evaluated at the initial state Σ)
  n  :=  |V_{s_C}(d_src)|
  A  :=  {M(d_src)(u) : u ∈ V_{s_C}(d_src)}          (the carried I-addresses)

Identity clause
  if  ω(d_src) = π  →  v := nextv(E, d_src)                  (fork in place)
  []  ω(d_src) ≠ π  →  v := nextd(E, π)                      (fork across ownership)
  fi

Effect (net, from Σ to Σ')
  E'      =  E ∪ {v}
  M'(v)   =  M(d_src)|_{V_{s_C}(d_src)}              (the snapshot; ∅ when n = 0)
  M'(d)   =  M(d)         for every d ∈ E_doc        (in particular d = d_src)
  C'      =  C
  L'      =  L
  R'      =  R ∪ {(a, v) : a ∈ A}
  Π' = Π

Result
  v — the operation's value; in the owned case: trunc(v) = d_src
```

---

## V-WF — WellFormedness (LEMMA, lemma)

VERSION is realizable as a valid composite at every reachable `Σ` with `d_src ∈ E_doc` — the owned branch at any forker tier, the cross-owner branch presupposing an account-tier forker (per P-tier, which excludes the node-tier case from the operation's domain). The step sequence is an identity allocation, then — when `n ≥ 1` — one K.μ⁺ and `|A|` K.ρ steps. The identity allocation is a *single* K.δ in both covered cases: the owned case (`v = nextv(E, d_src)`, a version step in `d_src`'s existing namespace) and the account-tier cross-owner case (`v = nextd(E, π)`, one `k = 2` descent or `k = 0` sibling in `π`'s account document sub-allocator `A_doc(pfx(π))`), so `allocated_by(π, v)` holds with `v ∈ S(pfx(π), 2)`. Under the account-tier restriction every covered VERSION allocates exactly one identity.

Invoked at a composite boundary (P-bdy), its post-state is the terminal boundary of that composite, satisfying both ExtendedReachableStateInvariants and the composite-boundary properties P4★ ∧ P4a ∧ P7a.

---

## derives — Derives (DEF, predicate)

> **derives** — `derives(v, d)` holds iff some `VERSION(·, d)` invocation produced `v`

---

## VD — VersionNamespaceDiscipline (DEF, predicate)

Every allocation into a version namespace is a fork of its parent:

> `(A d ∈ E_doc, w ∈ E ∩ S(d, 1) :: w entered E as the output of a VERSION(·, d) invocation)`

Under VD, the registry decides address-encoded derivation — for `v ∈ S(d, 1)`:

> `derives(v, d) ⟺ v ∈ E`

The unrestricted forward direction `derives(v, d) ⟹ v ∈ S(d, 1)` fails for cross-owner forks (severance, V9): a cross-owner fork makes `derives(v, d)` hold with `v ∈ E` yet `¬(d ≼ v)`, so `v ∉ S(d, 1)`.

---

## V0 — FreshUniquePermanentIdentity (LEMMA, lemma)

Exactly one identity is allocated, it collides with nothing, and it never goes away:

> `E' = E ∪ {v}` with `v ∉ E`; `v` is distinct from the output of every other allocation event; and `(A Σ'' : Σ' →* Σ'' : v ∈ Σ''.E)`.

The count is exactly one in each of the two in-domain branches (owned: single version K.δ; account-tier cross-owner: single document K.δ in `π`'s document namespace) — exactly P-tier's resolved domain. Distinctness from all other allocation events is by GlobalUniqueness (ASN-0034). Permanence is P1 (EntityPermanence).

---

## V1 — ZeroContentFootprint (LEMMA, lemma)

The fork allocates no content and no links:

> `C' = C  ∧  L' = L`

Whatever the source's extent, the sole allocation is `ΔE = {v}` — exactly one identity minted (V0), with no content or link address added. `ΔM` is one arrangement function on the `n` canonical positions, every image a pre-existing address; `ΔR = A × {v}` with `|A| ≤ n`.

---

## V2 — ArrangementTranscription (LEMMA, lemma)

The version's initial arrangement is the source's content-subspace arrangement — the function itself:

> `M'(v) = M(d_src)|_{V_{s_C}(d_src)}`, so `dom(M'(v)) = V_{s_C}(d_src)` and `ran(M'(v)) = A ⊆ dom(C)`.

---

## V2b — ForeignLinkExclusion (INV, predicate)

No reachable transition seats a link of foreign origin in any document's link subspace:

> `(A d, x : x ∈ dom(M(d)) ∧ subspace(x) = s_L : origin(M(d)(x)) = d)` (CL-OWN)

and the sole link-subspace extension transition carries precondition `origin(ℓ) = d` (K.μ⁺_L).

---

## V3 — SourceFrame (LEMMA, lemma)

Every `d_src`-indexed observable is unchanged from `Σ` to `Σ'`:

> `d_src ∈ E'`;  `M'(d_src) = M(d_src)`;  `C' = C` and `L' = L` (stores and their values untouched);  `{(a, d) ∈ R' : d = d_src} = {(a, d) ∈ R : d = d_src}` — the fork is strictly additive and writes no forward pointer.

---

## V4 — AncestryPrefix (LEMMA, lemma)

For the owned fork (`ω(d_src) = π`), the version's identity bears to the source's identity the relation *daughter by single-component extension*, and the relation is total:

> `v ∈ S(d_src, 1)`, i.e. `v = d_src·k` for some `k ≥ 1`; hence
> (a) `d_src ≺ v` and `trunc(v) = d_src`;
> (b) `#v = #d_src + 1` and `zeros(v) = zeros(d_src) = 2`, so `Document(v)` with T4-validity;
> (c) `N(v) = N(d_src)`, `U(v) = U(d_src)`, and `D(v) = D(d_src)` extended by the final component `k`;
> (d) `acct(v) = acct(d_src)`.

---

## V5 — ChronologicalRank (LEMMA, lemma)

The ordinal in the identity records allocation order in the version namespace, and nothing else:

> (a) the k-th allocation into `S(d_src, 1)` (in commit order) receives the k-th stream member `d_src·k`; reading rank as *fork* order is exact precisely under VD — when forks of `d_src` are the namespace's only allocations;
> (b) the allocator is *registry-pure*: `(A Σ₁, Σ₂ : Σ₁.E ∩ S(d, 1) = Σ₂.E ∩ S(d, 1) : nextv(Σ₁.E, d) = nextv(Σ₂.E, d))` — `C`, `M`, `L`, `R` are not arguments;
> (c) ranks are never reused: identities never leave `E` (P1), so a rank once taken is taken forever.

---

## V6 — IterativeClosure (LEMMA, lemma)

The operation is closed over its own output, and composes without structural bound:

> `Document(v)` holds and `ω'(v)` is the forker (V8, V9), so `VERSION(·, v)` is enabled at `Σ'` with no further setup. For a chain `w₀ = d`, `wⱼ₊₁ ∈ S(wⱼ, 1)`: `#wⱼ = #d + j`, `zeros(wⱼ) = 2` at every depth, and `(A i : 0 ≤ i ≤ j : trunc^i(wⱼ) = w_{j−i})` — the full derivation path is read by iterated truncation.

`B6(wⱼ, 1)` holds at every depth unconditionally — depth-1 forking never consumes the separator budget. Fork depth must be unbounded: a fixed cap `C` leaves the system, at the cap, only fatal moves — a further fork must either renumber existing addresses (breaking V0) or refuse the fork (breaking this closure). No choice of `C` escapes this renumber-or-refuse dilemma.

---

## V7 — NavigationAsymmetry (LEMMA, lemma)

The two directions of ancestry navigation rest on different resources, and neither is the source's own state:

> *Upward* — from any version, every ancestor is computed by iterated truncation: a pure function of the identity, consulting no state.
> *Downward* — the *owned* (address-discoverable) versions of `d` are the registry query `E ∩ S(d, 1) = {c₁, …, c_{hwm}}`, gap-free (VN-B1), so enumeration terminates at the first absentee; the full owned-descendant set `{e ∈ E : d ≺ e}` is T1-contiguous (T5), a single range scan of the ordered registry — and every address-encoded descendant of `d` is owned by `ω(d)`, since no account-tier prefix (`zeros ≤ 1`, O1a) can cover past `d` (`zeros = 2`, Z-mono). Cross-owner versions are *not* recovered here — severed from the source's subtree, they fall outside every address-based descendant scan.
> *Never* — a read of the source's own components: by V3 no `d_src`-indexed state mentions `v`, so there is nothing there to read.

---

## V8 — OwnershipInheritance (LEMMA, lemma)

When the forker owns the source, the version's owner is the source's owner — inherited structurally, with nothing allocated to record it:

> `ω(d_src) = π  ⟹  ω'(v) = π`, with `Π' = Π` and `acct(v) = acct(d_src)`.

Write `coverers(x) = {π'' ∈ Π : pfx(π'') ≼ x}`. The proof establishes `coverers(v) = coverers(d_src)`:
- (⊇) `pfx(π'') ≼ d_src ≺ v` chains to `pfx(π'') ≼ v`.
- (⊆) Suppose `pfx(π'') ≼ v`. Both `pfx(π'')` and `d_src` are prefixes of `v`, hence comparable. The case `d_src ≼ pfx(π'')` gives `zeros(pfx(π'')) ≥ zeros(d_src) = 2` by Z-mono, contradicting O1a's account-tier bound; so `pfx(π'') ≼ d_src`.

The coverer sets coincide; `ω` selects the unique maximal-length coverer (O2); `Π` is unchanged by the composite; so `ω'(v) = ω(d_src) = π`.

---

## V9 — CrossOwnerFork (LEMMA, lemma)

Let `π_o := ω(d_src) ≠ π` and `zeros(pfx(π)) = 1` (account-tier forker, per P-tier). The cross-owner fork allocates `v = nextd(E, π)` in `π`'s account document sub-allocator `A_doc(pfx(π))` with `v ∈ S(pfx(π), 2)`, so `v = [pfx(π)₁, …, pfx(π)_{#pfx(π)}, 0, k]` with `k ≥ 1`. This structural form gives:

- O5(i) as immediate: `pfx(π) ≼ v`
- O5(ii) as theorem: `(A π'' ∈ Π : pfx(π'') ≼ v ⟹ #pfx(π'') ≤ #pfx(π))`

Then:

> (a) **Severance** — `¬(d_src ≼ v)`: the new identity cannot lie in the source's subtree, so prefix-encoded ancestry is unattainable, not merely omitted;
> (b) **Ownership** — `ω'(v) = π`: the forker owns the fork outright;
> (c) **Editability** — the forker's right to edit `v` follows from (b) and from nothing about the source's permissions, which the operation never consulted (P-src is the entire source-side precondition).

---

## V9w — SharedContentWitness (LEMMA, lemma)

When the fork carries content (`A ≠ ∅`, equivalently `n ≥ 1`), what durably records the cross-owner relationship — and reinforces the owned case — is dual provenance over the shared addresses:

> `(A a ∈ A :: (a, d_src) ∈ R'  ∧  (a, v) ∈ R')`, and both rows persist in every successor state (P2).

The source-side row holds at `Σ` via P4★ at the composite boundary (P-bdy): each `a ∈ A` is `M(d_src)(u)` for some `u ∈ V_{s_C}(d_src)`, so `(a, d_src) ∈ Contains_C(Σ) ⊆ R ⊆ R'`. The version-side row is V13.

For a content-empty (or links-only) cross-owner source (`A = ∅`): the witness is vacuous, leaving no state-level trace of the derivation, which survives only as the `derives` event (VD), with no state-level trace whatever.

---

## V10 — LinkCarryThrough (LEMMA, lemma)

> `(A a ∈ dom(Σ'.L), i : 1 ≤ i ≤ |Σ'.L(a)| : project(a, i, v, Σ') ≠ ∅  ⟺  coverage(Σ.L(a).eᵢ) ∩ A ≠ ∅)`

---

## V11 — EditIndependence (LEMMA, lemma)

The version is independently editable from the instant of its creation, and the independence is mutual:

> (a) *Immediacy* — `v ∈ E'_doc` with `ω'(v)` the forker: the version stands under the same enabling conditions as any allocated document, with nothing `v`-specific outstanding.
> (b) *Isolation, both directions* — `(A d' : d' ≠ d : M''(d') = M'(d'))` (the K.μ family). By induction over any subsequent transition sequence, edits scoped to `v` leave `M(d_src)` pointwise fixed, and edits scoped to `d_src` leave `M(v)` pointwise fixed.
> (c) *The shared substance is beyond reach from either side* — `(A a ∈ dom(C) :: C''(a) = C'(a))` (P0): deletion at either document is contraction of that document's own arrangement, never a write to the store or to the other's arrangement.

---

## V12 — IdentityContentBoundary (LEMMA, lemma)

Forking exhibits, at the hard case, that identity and content are independently variable:

> at `Σ'`: `d_src ≠ v` (V0), yet `M'(v) = M'(d_src)|_{V_{s_C}(d_src)}` (V2 with V3) — two identities, one body of content.

The map from identity to content-subspace arrangement is non-injective *by construction*: identity is not recoverable from content, however total the content.

---

## V13 — ProvenanceCoupling (LEMMA, lemma)

The provenance clause is pinned, both ways, by the couplings:

> `R' = R ∪ {(a, v) : a ∈ A}` — J1★ forces every pair in (each carried address is range-new in `v`'s content subspace), and J1'★ forbids any pair beyond.

Each row is permanent (P2).
