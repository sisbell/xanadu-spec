> **ASN-0123 · The CREATENEWVERSION Operation** — Operations layer  
> **Depends on:** [ASN-0034 · Tumbler Algebra](../foundation/ASN-0034-tumbler-algebra.md), [ASN-0036 · Strand Model](../foundation/ASN-0036-strand-model.md), [ASN-0040 · Tumbler Baptism](../foundation/ASN-0040-tumbler-baptism.md), [ASN-0042 · Tumbler Ownership](../foundation/ASN-0042-tumbler-ownership.md), [ASN-0043 · Link Model](../foundation/ASN-0043-link-model.md), [ASN-0045 · Tumbler Fields](../foundation/ASN-0045-tumbler-fields.md), [ASN-0047 · Transition Model](../foundation/ASN-0047-transition-model.md), [ASN-0053 · Span Algebra](../foundation/ASN-0053-span-algebra.md), [ASN-0058 · Mapping Block Algebra](../foundation/ASN-0058-bundle-algebra.md), [ASN-0086 · Typed Relations on Address Sets](../protocols/ASN-0086-typed-relations-on-address-sets.md), [ASN-0093 · Allocation Substrate](../foundation/ASN-0093-allocation-substrate.md), [ASN-0098 · Link Projection Displacement](../foundation/ASN-0098-link-projection-displacement.md)  
> [Condensed statements →](ASN-0123-createnewversion-operation.statements.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0123: The CREATENEWVERSION Operation

*2026-06-12*

## The Problem

A document in this system is an allocated, owned identity whose substance lives elsewhere: the content store `C` holds immutable values at permanent element-level addresses, and the document's arrangement `M(d)` maps its V-positions onto those addresses. We ask what it must mean to *make a new version* of an existing document `d_src` — Nelson's CREATENEWVERSION, which "creates a new document with the contents of document `<doc id>` … The new document's id will indicate its ancestry" [LM 4/66].

Two naive readings present themselves, and both fail.

*The copy.* Allocate fresh content addresses carrying the same values, and arrange the new document over the copies. This fails: identity in the content store is identity of origin, not of value — distinct allocation events produce distinct addresses regardless of what is stored at them (S4, OriginBasedIdentity, ASN-0036; GlobalUniqueness, ASN-0034) — so the version's copies would share no identity with the source's content. G2 derives why that is fatal.

*The alias.* Let the "version" be a second name for the same arrangement — share `M` itself. Then the version is not independently editable: every edit through either name moves both, non-destruction of the source fails, and the version has no arrangement of its own in which to diverge.

So a version must be a third thing, and the consultation record shows Nelson designed exactly that third thing: a **fresh identity** whose arrangement is a **transcription** — the same content addresses, in the same order, taken at the moment of forking — after which the two arrangements are independent state. Exactly one thing is minted: the identity. Ownership is inherited, not re-allocated (when the forker owns the source). Content is shared by reference, and its duplication is the thing the design must forbid: "No copying operations are required among the documents throughout the system" [LM 2/36]; users make new documents out of old ones "without damaging the originals" [LM 2/45].

This note derives the operation from the guarantees it must keep, states its contract, and proves the invariants the topic demands: the permanence and untouchedness of the source; the ancestry encoded in the new identity; the ownership and editability that follow the fork; what forking reveals about the boundary between identity and content; the carry-through of links anchored to content the version transcludes; and the navigability of the ancestry chain across iterated forks.

**Scope.** We specify the fork alone. Creating documents from nothing, establishing accounts or principals, comparing versions, the editing operations themselves, link creation, content delivery, and replication are all out of scope; we touch them only where a frame condition or a foundation invariant of theirs bears on the fork's guarantees.

## State and Local Apparatus

We work in the extended state of the transition-model foundation (ASN-0047):

> `Σ = (C, L, E, M, R)`

with `C : T ⇀ Val` the content store (immutable and append-only, P0), `L : T ⇀ Link` the link store (immutable, L12), `E ⊆ T` the entity set with its document stratum `E_doc = {e ∈ E : Document(e)}` (so `zeros(e) = 2` and T4-valid, per ASN-0045), `M` the per-document arrangements with content subspace `s_C` and link subspace `s_L` (`s_C = 1`, `s_L = 2` by SubspaceConventionAxiom), and `R ⊆ T_elem × E_doc` the provenance relation. A document's content-subspace positions are canonical and gap-free: `V_{s_C}(d) = {[s_C, 1, …, 1, k] : 1 ≤ k ≤ n}` for some `n ≥ 0` (D-SEQ★). Composite transitions are finite sequences of the atomic vocabulary, valid when each step's precondition holds at its intermediate state and the couplings J0, J1★, J1'★ hold initial-to-final (ValidComposite★). Ownership is the prefix model of ASN-0042: principals `Π` with account-tier prefixes (`zeros(pfx(π)) ≤ 1`, O1a), effective owner `ω(a)` the unique most-specific covering principal (O2), and allocation authority confined to one's own domain (O5).

Reading ASN-0042's ownership vocabulary over ASN-0047's states is a hybrid the two foundations do not assemble for us — O2's totality of `ω` is a theorem about ASN-0042's *own* reachable states, resting on its bootstrap and delegation dynamics — so we state the standing assumptions under which the reading is sound.

**PS (PrincipalStructure).** Every reachable docuverse state carries an ASN-0042-conforming principal structure:

> (i) *Dynamics* — `Π` and `pfx` satisfy O1a (account-tier prefixes) and O1b (prefix injectivity), and evolve per O12, O13, O15 across every atomic transition: principals persist, no prefix ever changes, and at most one principal enters per transition, by delegation.
> (ii) *Authority* — `allocated_by` attaches to K.δ: every entity-creating step is performed by some existing principal (O16) inside its own domain (O5).
> (iii) *Bootstrap coverage* — some `π₀ ∈ Π₀` covers the bootstrap node: `pfx(π₀) ≼ n₀`.
> (iv) *Incumbency* — every principal occupies a baptized entity: `pfx(π) ∈ E` for every `π ∈ Π`. An ASN-0042-conforming structure satisfies PrefixBaptismCoupling (`pfx(π) ∈ Σ.B`); since `pfx(π)` is account-tier (`zeros(pfx(π)) ≤ 1`, O1a) it is entity-level, so `pfx(π)`, a baptized entity-level address, is an entity of `E`. An account-tier principal (`zeros(pfx(π)) = 1`) thereby occupies an `Account` entity carrying a document sub-allocator `A_doc` (ASN-0047).

From (i)–(iii), coverage of the registry — O4 with `E` in the registry role — is derived, not assumed: every `e ∈ E` extends `n₀`. For the bootstrap itself this is reflexive; for nodes it is NodeBaptism(b) directly; for K.δ case (ii) outputs it propagates by induction, because no increment disturbs position 1 — a `k > 0` step appends positions (TA5(b), TA5(d)), and a `k = 0` step modifies only `sig(t) = #t` (TA5-SigValid on the T4-valid operand), with `#t ≥ 3` since the operand is a non-node entity (`zeros(t) ≥ 1`) — and `n₀ = [1]` asks agreement at position 1 alone. Transitivity of `≼` then gives `pfx(π₀) ≼ n₀ ≼ e`. Coverage is exactly what O2's uniqueness argument consumes, so it applies verbatim over these states: **`ω : E → Π` is total and single-valued at every reachable state**. In particular `ω(d_src)` is defined at the operation's branch guard.

For allocation we use the baptism apparatus of ASN-0040. The **version namespace** of a document `d` is the sibling stream at depth 1:

> `S(d, 1) = c₁, c₂, c₃, …` where `c₁ = inc(d, 1)` and `cₙ₊₁ = inc(cₙ, 0)`,

whose n-th member is, by the SiblingStream postcondition at depth 1 (which contributes `1 − 1 = 0` separator zeros), the tumbler `cₙ = [d₁, …, d_{#d}, n]` — the source's components with the single component `n` appended. We write `d·k` for this length-`(#d + 1)` extension of `d` by final component `k`, so `cₖ = d·k`. This is the namespace ASN-0047 names `A_v(d)`, the version sub-allocator of `d`. We treat each entity-creating step as the baptism of its address, so `E` plays the registry role of ASN-0040's `B` at the *entity level* — the node-, account-, and document-tier addresses (`zeros ≤ 2`). Element-level baptisms (content, links) register in `C` and `L` instead, so `E` is exactly the entity-level restriction of the baptismal registry: a baptized address with `zeros ≤ 2` is an entity of `E`.

Two small tools, used throughout, are worth defining once.

**trunc (SingleComponentTruncation).** For `t ∈ T` with `#t ≥ 2`, `trunc(t)` is the tumbler of length `#t − 1` agreeing with `t` everywhere it is defined:

> `#trunc(t) = #t − 1  ∧  (A i : 1 ≤ i ≤ #t − 1 : trunc(t)ᵢ = tᵢ)`

Membership `trunc(t) ∈ T` is immediate from T0 — a finite sequence over ℕ of length ≥ 1, since `#t ≥ 2`. For every member of a version stream, truncation recovers the parent: `v ∈ S(d, 1) ⟹ trunc(v) = d`, because `v = d·k` agrees with `d` on positions `1 … #d` and has length `#d + 1`.

**Z-mono (ZeroCountMonotonicity).** Prefixing cannot lose zeros:

> `(A p, q ∈ T : p ≼ q : zeros(p) ≤ zeros(q))`

since `q` agrees with `p` on positions `1 … #p` — so every zero of `p` is a zero of `q` — and `q`'s further positions can only contribute more.

**SA (StoredAddressAntichain).** No stored address extends another: at every reachable state `dom(C) ∪ dom(L)` is an antichain under `≼`, so for every stored `a`

> `{t ∈ T : a ≼ t} ∩ (dom(C) ∪ dom(L)) = {a}`.

*Proof.* By LP-Sub (ASN-0098) every stored address has the structural form `[d, 0, s, k]`: a T4-valid document tumbler `d` with `zeros(d) = 2`, the field separator, a subspace identifier, one ordinal. Suppose `a = [d₀, 0, s, k]` and `b = [d', 0, s', k']` are both stored with `a ≺ b`. Then `#d' = #b − 3 > #a − 3 = #d₀`, and `b` agrees with `a` on positions `1 … #a` — in particular `b` carries `a`'s separator zero at position `#d₀ + 1`. Since `#d' ≥ #d₀ + 1`, that position lies inside `b`'s document prefix `d'`, whose positions `1 … #d₀` already carry `d₀`'s two zeros; so `zeros(d') ≥ 3`, contradicting `zeros(d') = 2`. No proper extension exists, and the displayed intersection is `{a}` by reflexivity of `≼`. ∎

**VN-B1 (VersionNamespaceContiguity).** ASN-0040's B1 is an invariant of *its* transition system (Bop, B0a, seed conformance); it does not transfer to ASN-0047's K.δ vocabulary by citation, so we prove its analog — and we prove it once for both document-producing sibling streams the operation allocates into: the *version* stream `S(d, 1)` rooted at a document (`zeros(d) = 2`, base-spawned by the depth-1 step `inc(d, 1)`), and the *account-document* stream `S(pfx(π), 2)` rooted at an account (`zeros(pfx(π)) = 1`, base-spawned by the depth-2 step `inc(pfx(π), 2)` — the namespace ASN-0047 names `A_doc(pfx(π))`). Both are B6-valid (a document tumbler is a T10a output, S7d, DocumentAllocationDiscipline, ASN-0036; an account prefix is entity-resident by PS incumbency), and both produce members at the document tier. Uniformly: let `(p, g)` be a B6-valid namespace with `g ∈ {1, 2}` and `zeros(p) + (g − 1) = 2`, so each stream member

> `cₙ = [p₁, …, p_{#p}, 0, …, 0, n]` (length `#p + g`, with `g − 1` separator zeros — none when `g = 1`, one when `g = 2` — before the final ordinal `n ≥ 1`) has `zeros(cₙ) = 2` and is a T4-valid document (B6(a)).

Then at every reachable state:

> `E ∩ S(p, g) = {c₁, …, c_m}` for some `m ≥ 0` — the realized children are a contiguous prefix of the stream.

*Proof, by induction over the atomic transitions.* *Base:* `E₀ = {n₀}` with `zeros(n₀) = 0`, while every stream member has `zeros = 2`; the intersection is the empty prefix. *Step:* only K.δ changes `E`, adding one fresh `e ∉ E`; if `e ∉ S(p, g)` the intersection is unchanged, so suppose `e = c_j ∈ S(p, g)` with `j ≥ 1`. Which K.δ instances can land there?

> *Case (i), `Node(e)`* — impossible: `zeros(e) = 2 ≠ 0`.
> *The base-tier spawn, `k = g`* — the one inter-tier step whose output carries exactly `g − 1` separator zeros. For `g = 1` it is the version step (output `t·1`, operand `t ∈ E_doc`); for `g = 2` it is the descent (output `inc(t, 2) = [t₁, …, t_{#t}, 0, 1]` by TA5(d), operand `zeros(t) ≤ 1`). In both, matching the output to `c_j` forces — by length, then T3 componentwise — `t = p` and `j = 1`: the only base-tier arrival is `c₁ = inc(p, g)`. Freshness `c₁ ∉ E` gives `m = 0`, and the intersection becomes `{c₁}`.
> *The other inter-tier spawn, `k = 3 − g`* — impossible by the penultimate component. A version step (`k = 1`) outputs `t·1`, whose penultimate component is `t_{#t} ≠ 0` (T4 forbids a trailing zero), so it cannot equal a `g = 2` member `[p₁, …, p_{#p}, 0, j]` (penultimate `0`); a descent (`k = 2`) outputs `[t₁, …, t_{#t}, 0, 1]`, whose penultimate component is the appended separator `0`, so it cannot equal a `g = 1` member `[p₁, …, p_{#p}, j]` (penultimate `p_{#p} ≠ 0`).
> *`k = 0` (sibling)* — the operand `t ∈ E` is T4-valid (every member of `Σ.E` is), so `sig(t) = #t` (TA5-SigValid) and the output is `t` with its final component incremented (TA5(c)). Output `= c_j` forces `#t = #p + g`, agreement with `c_j` on positions `1 … #p + g − 1`, and final component `j − 1 ≥ 1` (a trailing zero would break `t`'s T4-validity) — i.e. `t = c_{j−1}`. The operand constraint puts `c_{j−1} ∈ E`, so `j − 1 ≤ m` by the induction hypothesis; freshness `c_j ∉ E` gives `j > m`. Hence `j = m + 1` — the frontier, and nothing but (this is FrontierEquivalence, ASN-0047, read in stream coordinates).

In both arriving cases the new intersection is `{c₁, …, c_{m+1}}`: contiguity is preserved. ∎

**nextv, nextd (VersionFrontier, DocumentFrontier).** The next unallocated member of a document-producing namespace, given the registry — for the version namespace of a document `d`, and for the account-document namespace of an account-tier principal `π`:

> `nextv(E, d) = next(E, d, 1)`  and  `nextd(E, π) = next(E, pfx(π), 2)`

with `next` as in ASN-0040: `next(E, p, g) = inc(p, g)` when `E ∩ S(p, g) = ∅`, else `inc(max(E ∩ S(p, g)), 0)`. Both are well-defined at every reachable state. `E` is finite (it grows from the finite `E₀` by at most one element per atomic transition, SequentialTransitionAxiom). `(d, 1)` is B6-valid — `d` is a document-tier T10a output (S7d, DocumentAllocationDiscipline, ASN-0036), T4-valid with `zeros(d) = 2` (M0, ASN-0093), depth `1 ∈ {1, 2}`, `zeros(d) + 0 = 2 ≤ 3`; and `(pfx(π), 2)` is B6-valid — `pfx(π)` is account-tier — `zeros(pfx(π)) = 1` from the account-tier hypothesis of this definition's lead-in (equivalently the Account-entity occupancy of PS incumbency), O1a contributing only the `≤ 1` ceiling — T4-valid and entity-resident (PS incumbency), depth `2 ∈ {1, 2}`, `zeros(pfx(π)) + 1 = 2 ≤ 3`. Each satisfies VN-B1's side condition `zeros(p) + (g − 1) = 2`, so VN-B1 applies: the realized children `E ∩ S(p, g)` form a contiguous prefix `{c₁, …, c_m}`, with `m = hwm(E, p, g)` (the high-water mark counts exactly these children). From contiguity the frontier follows. When `m = 0`, `next(E, p, g) = inc(p, g) = c₁` by the stream's base case; when `m ≥ 1`, the stream is strictly T1-increasing (S0, StreamOrdering), so `max(E ∩ S(p, g)) = c_m`, and the recurrence `c_{m+1} = inc(c_m, 0)` gives `next(E, p, g) = inc(c_m, 0) = c_{m+1}`. Both cases collapse to a single statement, in which `c_{hwm(E, p, g) + 1}` reads `c_{m+1}`:

> `next(E, p, g) = c_{hwm(E, p, g) + 1}` — the gap-free successor; in particular `nextv(E, d) = c_{hwm(E, d, 1) + 1}` and `nextd(E, π) = c_{hwm(E, pfx(π), 2) + 1}`.

## Deriving the Operation from Its Guarantees

We do not design the operation and then check it; we let the guarantees dictate the clauses. Three guarantees are owed.

### G1: the ancestry must be readable from the identity — the allocation clause

Nelson's stipulation is a postcondition on the result: "The new document's id will indicate its ancestry" [LM 4/66]. For the fork performed by the source's owner, we demand of the new identity `v`:

> `R_id:  v ∈ E' ∖ E  ∧  Document(v)  ∧  trunc(v) = d_src`

and we reason backward. `trunc(v) = d_src` fixes `#v = #d_src + 1` and component agreement on positions `1 … #d_src`: the candidates are exactly the single-component extensions `d_src·x`. `Document(v)` requires T4-validity, which at the last position requires `x ≠ 0`; conversely every `x ≥ 1` introduces no new zero and no adjacent-zero pattern, so `zeros(d_src·x) = zeros(d_src) = 2` and every such candidate is a T4-valid document. The candidate set `{d_src·x : x ≥ 1}` is therefore precisely the sibling stream `S(d_src, 1)` — and B6(d_src, 1) holds, so the stream is a legitimate baptismal namespace whose every member is T4-valid (B6(a)).

Freshness strikes out the members already in `E`. Among the fresh candidates the choice is still not free: if the allocator may skip, the realized children cease to be a contiguous prefix of the stream, the high-water mark loses its meaning, and the enumeration of a document's versions can no longer terminate at the first absentee (VN-B1, and the frontier identity `nextv(E, d) = c_{hwm + 1}` derived from it). Contiguity forces the minimal fresh candidate:

> `v = nextv(E, d_src)`

The allocation clause is thereby derived, not designed. Note also what it does *not* require: no global counter, no clock, and — decisively — no bookkeeping held by the source. The frontier is a function of the registry alone.

### G2: links must carry through and correspondence must be computable — the transcription clause

The guarantee owed is "links may be refractively followed from a point or span in one version to corresponding places in any other version. Thus a link to one version of a Prismatic Document is a link to all versions" [LM 2/26] — owed not only to links existing at the fork, but to links not yet made, since an endset may later be formed over any addresses whatever (L4, EndsetGenerality).

Let `A = {M(d_src)(u) : u ∈ V_{s_C}(d_src)}` be the content addresses the source currently arranges. Fix any `a ∈ A`. The unit-depth span `(a, δ(1, #a))` is T12-well-formed (its action point is `#a ≤ #a`), an endset containing it alone is admissible in any slot of a future link, and its coverage contains `a`. For that link to be discoverable from the version `v`, LP12 requires `coverage ∩ ran(Σ'.M(v)) ≠ ∅`. That alone does not yet name `a`: the span's coverage is the full subtree `{t : a ≼ t}` (PrefixSpanCoverage, ASN-0043), not the singleton. SA closes the gap. Every member of `ran(Σ'.M(v))` is a stored address (S3★), and `a` is itself stored (`a ∈ A ⊆ dom(C)` by S3★ at `Σ`, persisting by P0); by SA the only stored member of `a`'s subtree is `a`, so the required intersection is non-empty exactly when `a ∈ ran(Σ'.M(v))`. Since the construction works uniformly for every `a ∈ A`, the guarantee forces

> `A ⊆ ran(Σ'.M(v))` — range preservation is *necessary*.

Could the obligation be met with fresh addresses bearing equal values — a copy? No. By S4 and GlobalUniqueness, fresh allocation yields different addresses however equal the values; `coverage` is a set of addresses, indifferent to values; the strap that holds `a` holds no copy of `a`. Under copying, `coverage ∩ ran(Σ'.M(v)) = ∅` for every anchor in the source — carry-through fails totally. The same substitution breaks correspondence (which must rest on shared identity, not value comparison — the ground on which any out-of-scope comparison operation stands) and forges attribution (the copy's `origin` names the version, S7). This is the precise content of "what the design must forbid at the moment of forking": **the operation may not allocate content.** Sharing is forced.

Range preservation alone would still leave the version's initial *order* unspecified. But the operation's meaning is "a new document *with the contents of* the source" [LM 4/66] — the same contents in the same arrangement — and determinism demands the post-arrangement be a function of the source's arrangement, with no manufactured information. The canonical such function is the transcription itself:

> `M'(v) = M(d_src)|_{V_{s_C}(d_src)}`

— equality of mappings on the canonical positions, which makes the boundary correspondence total (the two documents agree position-for-position by construction) and carries the order- and width-preservation of the source's run structure (M0, M1, ASN-0058) unchanged.

What of the source's *link-subspace* arrangement? Transcribing it is not merely omitted; it is impossible — no reachable transition can seat a foreign-origin link in `v`'s link subspace (V2b, ForeignLinkExclusion). So `dom(M'(v)) = V_{s_C}(d_src)` exactly, and content anchoring is the *only* channel by which connectivity can cross a fork — which, by the range argument above, is also a sufficient one.

### G3: the source must be untouched — the frame, and the asymmetry of the record

Non-destruction is Nelson's conservation law: changes are made "without damaging the originals" [LM 2/45], and a fork may not reach back — "the owner of a document may delete bytes from the owner's current version, but those bytes remain in all other documents where they have been included" [LM 4/11]. We enumerate the `d_src`-indexed observables and demand each unchanged: membership (`d_src ∈ E'`), arrangement (`M'(d_src) = M(d_src)`), the values at its content addresses (`C'` agrees with `C` on `dom(C)`), its provenance row (`{(a, d) ∈ R' : d = d_src} = {(a, d) ∈ R : d = d_src}`), and its links' values (`L' = L`). The fork must be *strictly additive*.

This frame has a structural consequence: since nothing may be written at the source, whatever records the fork must live elsewhere — and the state offers only two elsewheres: the fresh identity itself (its position in the address tree, G1) and the provenance row of the *version*. The couplings then pin the latter exactly. J1★ requires a provenance entry for every address newly in `v`'s content-subspace range — and at the fork every carried address is range-new in `v`, which had no arrangement — so `R' ⊇ {(a, v) : a ∈ A}`; J1'★ caps `R' ∖ R` at exactly those pairs. A fork without its provenance row, or with any extra row, is not a valid composite.

## The Operation

The three derived clauses assemble into the contract. We name the abstract operation `VERSION`; it specifies the protocol request CREATENEWVERSION.

```
VERSION(π, d_src)

Preconditions
  P-src    d_src ∈ E_doc                (the source is an allocated document)
  P-prin   π ∈ Π                        (the forking principal)
  P-bdy    Σ is a composite boundary    (VERSION is invoked as a whole composite,
                                         its initial state the terminal boundary
                                         of whatever composite precedes it —
                                         whole-request serialization; see the
                                         atomicity remark)
  P-tier   ω(d_src) = π  ∨  zeros(pfx(π)) = 1   (the operation's domain
           condition; the identity clause branches on the same guard)

  — P-bdy fixes the invocation context, not a source-side condition; P-tier is
  well-formed because ω(d_src) is defined (PS). P-tier's second disjunct restricts
  the cross-owner case (ω(d_src) ≠ π) to an account-tier forker: a node-tier
  non-owner (zeros(pfx(π)) = 0) commands no document-producing namespace — one K.δ
  off pfx(π) at depth 2 yields inc(pfx(π), 2) = [pfx(π), 0, 1], an Account
  (zeros = 1), not a Document — so forking a foreign document there would demand
  minting both an account and a document under it, more than the single identity
  VERSION allocates, placing it outside the operation's domain. Scope note: no
  authority over d_src is required (the source is read without permission), and no
  condition is placed on M(d_src) (the empty source is admitted, n = 0 below).

Abbreviations (evaluated at the initial state Σ)
  n  :=  |V_{s_C}(d_src)|
  A  :=  {M(d_src)(u) : u ∈ V_{s_C}(d_src)}          (the carried I-addresses)

Identity clause
  if  ω(d_src) = π  →  v := nextv(E, d_src)                  (fork in place)
  []  ω(d_src) ≠ π  →  v := nextd(E, π)                      (fork across ownership)
  fi
  — In both branches v is a single entity-allocation at a registry frontier: nextv
    in d_src's version namespace (owned), or nextd in π's account-document namespace
    S(pfx(π), 2) = A_doc(pfx(π)) (cross-owner; ASN-0047, AllocatorHierarchy).

Effect (net, from Σ to Σ')
  E'      =  E ∪ {v}
  M'(v)   =  M(d_src)|_{V_{s_C}(d_src)}              (the snapshot; ∅ when n = 0)
  M'(d)   =  M(d)         for every d ∈ E_doc        (in particular d = d_src)
  C'      =  C
  L'      =  L
  R'      =  R ∪ {(a, v) : a ∈ A}
  Π' = Π — no principal is introduced, no prefix changes (O12, O13, O15)

Result
  v — the operation's value is the fresh identity; in the owned case the
  source is recoverable from it:  trunc(v) = d_src.
```

**V-WF (WellFormedness).** `VERSION` is realizable as a valid composite at every reachable `Σ` with `d_src ∈ E_doc` — the owned branch at any forker tier, the cross-owner branch presupposing an account-tier forker (per P-tier, which excludes the node-tier case from the operation's domain). The step sequence is an identity allocation, then — when `n ≥ 1` — one K.μ⁺ and `|A|` K.ρ steps. The identity allocation is a *single* K.δ in both covered cases: the owned case (`v = nextv(E, d_src)`, a version step in `d_src`'s existing namespace) and the account-tier cross-owner case (`v = nextd(E, π)`, one `k = 2` descent or `k = 0` sibling in `π`'s account document sub-allocator `A_doc(pfx(π))`), so `allocated_by(π, v)` holds with `v ∈ S(pfx(π), 2)`. Under the account-tier restriction every covered VERSION allocates exactly one identity. We discharge ValidComposite★'s two clauses. *Clause 1, preconditions at intermediate states.* K.δ at `Σ`: in the owned branch the operand is the stream frontier — sub-case `k = 1` with `t = d_src ∈ E_doc` when the namespace is empty, sub-case `k = 0` with `t = max(E ∩ S(d_src, 1))` otherwise (`t ∈ E`, `¬Node(t)`); `parent(v) = parent(d_src) ∈ E` by K.δ-ID.parent-0/1 and P8; freshness `v ∉ E` by the `nextv` choice (VN-B1 — the realized children are exactly `{c₁, …, c_hwm}`, so its successor `c_{hwm + 1}` lies outside `E`). In the cross-owner branch — account-tier by P-tier — the forker `π` allocates the identity at the frontier `v = nextd(E, π)` of its account document sub-allocator `A_doc(pfx(π))`. `nextd` resolves to one document-level K.δ off that namespace's contiguous prefix (VN-B1): when `hwm(E, pfx(π), 2) = 0`, a `k = 2` descent `v = inc(pfx(π), 2)`, operand `pfx(π)` with `pfx(π) ∈ E` (PS incumbency) and `zeros(pfx(π)) = 1 ≤ 1`; when `hwm(E, pfx(π), 2) ≥ 1`, a `k = 0` sibling `v = inc(c, 0)` off the prior frontier `c = max(E ∩ S(pfx(π), 2))`, operand `c` with `c ∈ E` and `¬Node(c)` (a document). The operand precondition `parent(v) ∈ E` holds — `parent(v) = pfx(π) ∈ E` by K.δ-ID.parent-2 and PS incumbency in the descent case, `parent(v) = parent(c) ∈ E` by K.δ-ID.parent-0/1 and P8 in the sibling case. Freshness `v ∉ E` is the `nextd` choice — exactly as the owned branch's freshness is the `nextv` choice — discharged by VN-B1 over `S(pfx(π), 2)`: the realized children are exactly `{c₁, …, c_hwm}`, so the successor `c_{hwm + 1} = nextd(E, π)` lies outside `E`. The produced `v` lies in `S(pfx(π), 2)` (just shown); that stream is at depth 2, contributing one separator zero, so `zeros(v) = zeros(pfx(π)) + 1 = 2` (the forker is account-tier, `zeros(pfx(π)) = 1`), and every stream member is T4-valid (B6(a), ASN-0040) — whence `Document(v)` directly, discharging `v ∈ E_doc` for the K.μ⁺ precondition below. K.μ⁺ at the post-K.δ state: `v ∈ E_doc` holds, the extension is strict (`n ≥ 1`), every image lies in `dom(C)` (S3★ at `Σ`, and `C` is unchanged), the transcribed positions are the canonical set `{[s_C, 1, …, 1, k] : 1 ≤ k ≤ n}` so S8a, S8-depth, D-CTG★, D-MIN★ and the content-subspace restriction are immediate. Each K.ρ: `a ∈ dom(C)` and `v ∈ E_doc` hold. *Clause 2, couplings initial-to-final.* J0 is vacuous (`dom(C') = dom(C)`); J1★ and J1'★ are discharged exactly by the `R'` clause, as derived in G3. When `n = 0` the composite is the identity allocation alone and every coupling is vacuous. The post-state is therefore reachable and satisfies every per-state invariant (ExtendedReachableStateInvariants). Moreover, because `Σ` is a composite boundary (P-bdy) and VERSION is a valid composite, `Σ'` is the *terminal* boundary of that composite, and so additionally satisfies the composite-boundary properties P4★ ∧ P4a ∧ P7a (ExtendedReachableStateInvariants, boundary clause). ∎

*Remark (relation to the foundation's fork composite).* ASN-0047's J4 has the same three-step shape; the distinction is that where J4 ties its content operand to the version frontier, CREATENEWVERSION fixes it to the *named source* at every invocation — the two coinciding on first forks and whenever the source is unmodified between forks.

*Remark (atomicity — what the foundations do and do not supply).* SequentialTransitionAxiom makes each *atomic* step indivisible; it does not make the composite a unit. The sequence has a genuine interior state — after K.δ, before K.μ⁺ — at which `v ∈ E_doc` with `M(v) = ∅`, and nothing in the foundation forbids another composite from beginning there (what serialization concurrent forks require is among the open questions). What the foundations do supply is boundary-level: the couplings are evaluated initial-to-final over the valid composite, so the final boundary — the first state at which `v` is the operation's value — carries the snapshot and its provenance rows in full. No composite boundary of a valid VERSION exhibits the version without its snapshot; only interior states do, and at those the operation has not returned.

**V3 (SourceFrame).** The Effect clause *stipulates* the net frame G3 demands; we verify the step sequence *delivers* it. Every `d_src`-indexed observable is unchanged from `Σ` to `Σ'`:

> `d_src ∈ E'`;  `M'(d_src) = M(d_src)`;  `C' = C` and `L' = L` (stores and their values untouched);  `{(a, d) ∈ R' : d = d_src} = {(a, d) ∈ R : d = d_src}` — the fork is strictly additive and writes no forward pointer.

*Proof, by composing the step frames.* K.δ frames `C`, `L`, `R` outright and extends `E` by `{v}` alone, so `d_src ∈ E'`; its arrangement clause adds only the fresh key `v` with `M'(v) = ∅`, and `v ≠ d_src` — `v ∉ E` at the step while `d_src ∈ E` — so `M(d_src)` is untouched. K.μ⁺ (present when `n ≥ 1`) names `v` and frames every other document's arrangement, and frames `C`, `E`, `R`. Each K.ρ frames `C`, `L`, `E`, and every arrangement, and grows `R` by one pair; across the composite `R' ∖ R = A × {v}`, every added pair carrying second component `v ≠ d_src`, so the `d_src` provenance row survives verbatim. Conjoining the per-step frames yields the displayed equalities. The "no forward pointer" reading is the same fact stated negatively: the state's `d_src`-indexed components are exactly its registry membership, its arrangement, the stores its arrangement reaches into, and its provenance row — each unchanged — so at `Σ'` nothing `d_src`-indexed mentions `v`, which is what V7's navigation asymmetry exploits. ∎

## The Identity: Ancestry, Rank, Chains, Navigation

**V0 (FreshUniquePermanentIdentity).** Exactly one identity is allocated, it collides with nothing, and it never goes away:

> `E' = E ∪ {v}` with `v ∉ E`; `v` is distinct from the output of every other allocation event; and `(A Σ'' : Σ' →* Σ'' : v ∈ Σ''.E)`.

Freshness is the `nextv` choice (owned case) or the explicit constraint (cross-owner case). The count is exactly one in each of the two in-domain branches: the owned branch is a single version K.δ, and the account-tier cross-owner branch a single document K.δ in `π`'s existing document namespace. These are exactly P-tier's resolved domain — owned, or account-tier cross-owner — so no third branch contributes to the count. Distinctness from *all* other allocation events — versions of other documents, documents, accounts, content, links — is GlobalUniqueness (ASN-0034), which rules out collisions alike from the same allocator, sibling allocators, and allocators at different hierarchy depths. It applies because the version sub-allocator `A_v(d) = S(d, 1)` is T10a-conforming — its base spawned by `inc(d, 1)` (`k' = 1 ∈ {1, 2}`), its siblings advanced by `inc(·, 0)` — which is exactly GlobalUniqueness's hypothesis; so two versions of one document are distinct by its same-allocator case directly, with no appeal to B8's same-namespace case. The same-allocator argument needs no serialization assumption, so this distinctness is unconditional on commit order. Permanence is P1 (EntityPermanence): no transition removes an entity. Immutability of the identity itself needs no separate mechanism: identities are the *keys* of `E` and `M`, not stored values; P3 admits no contraction and no rewriting, so "renumbering" — removal plus reinsertion — is unavailable in the transition vocabulary. This is Nelson's permanence claim made structural: "New items may be continually inserted in tumbler-space while the other addresses remain valid" [LM 4/19].

**V4 (AncestryPrefix).** For the owned fork, the version's identity bears to the source's identity the relation *daughter by single-component extension*, and the relation is total — every identifying field is accounted for:

> `v ∈ S(d_src, 1)`, i.e. `v = d_src·k` for some `k ≥ 1`; hence
> (a) `d_src ≺ v` and `trunc(v) = d_src`;
> (b) `#v = #d_src + 1` and `zeros(v) = zeros(d_src) = 2`, so `Document(v)` with T4-validity;
> (c) `N(v) = N(d_src)`, `U(v) = U(d_src)`, and `D(v) = D(d_src)` extended by the final component `k`;
> (d) `acct(v) = acct(d_src)`.

(a) and (b) are the SiblingStream postcondition with B5/B5a (depth 1 adds no separator; sibling steps preserve zeros) and B6(a) (T4-validity of every stream member). For (c): `v` agrees with `d_src` on positions `1 … #d_src` and appends one nonzero component, so the zero positions of `v` are exactly those of `d_src`; T4b's unique parse is determined by the zero positions, so the node and user fields are component-identical and the document field gains exactly the final component. (d) follows from (c) by the AccountField definition. So **the version differs from the source in the document field alone** — the address records descent precisely where Nelson said it would: "The Document field of the tumbler may be continually subdivided, with new subfields in the tumbler indicating daughter documents and versions" [LM 4/29]. The ancestry is readable by truncation, and — see V5 — the final component is readable as the version's rank.

**V5 (ChronologicalRank).** The ordinal in the identity records allocation order in the version namespace, and nothing else:

> (a) the k-th allocation into `S(d_src, 1)` (in commit order) receives the k-th stream member `d_src·k`; reading rank as *fork* order is exact precisely under VD below — when forks of `d_src` are the namespace's only allocations;
> (b) the allocator is *registry-pure*: `(A Σ₁, Σ₂ : Σ₁.E ∩ S(d, 1) = Σ₂.E ∩ S(d, 1) : nextv(Σ₁.E, d) = nextv(Σ₂.E, d))` — `C`, `M`, `L`, `R` are not arguments;
> (c) ranks are never reused: identities never leave `E` (P1), so a rank once taken is taken forever.

(a) is VN-B1 and the `nextv` frontier identity (`nextv(E, d) = c_{hwm + 1}`, derived above from VN-B1 + S0) under serialized commits (SequentialTransitionAxiom, ASN-0047): every arrival in the namespace is a frontier arrival (VN-B1's preservation argument), so the j-th arrival lands at rank `hwm + 1 = j`, and the stream is strictly T1-increasing (S0, StreamOrdering). The fork-counting reading needs VD because the namespace has other lawful clients — ASN-0047's own J4 composite allocates on `A_v(d_src)`, and any discipline-conforming K.δ may take the frontier — so one interleaved non-VERSION allocation gives the k-th fork a rank exceeding k. (b) is the argument list of `nextv`, observed when we derived it. The consequence of (b) deserves emphasis: *rank and content state are orthogonal.* Two forks separated only by an edit of the source transcribe different snapshots (V2) yet take consecutive ranks; the address arithmetic never looks at the arrangement. And per (c), even a version later abandoned by its owner — its arrangement contracted to nothing — holds its rank forever. Time itself is not encoded: the stream order is creation order, but "'time' is not included in the tumbler. Time is kept track of separately" [LM 4/18].

**V6 (IterativeClosure and UnboundedDepth).** The operation is closed over its own output, and composes without structural bound:

> `Document(v)` holds and `ω'(v)` is the forker (V8, V9), so `VERSION(·, v)` is enabled at `Σ'` with no further setup. For a chain `w₀ = d`, `wⱼ₊₁ ∈ S(wⱼ, 1)`: `#wⱼ = #d + j`, `zeros(wⱼ) = 2` at every depth, and `(A i : 0 ≤ i ≤ j : trunc^i(wⱼ) = w_{j−i})` — the full derivation path is read by iterated truncation.

Each fork appends exactly one component, and — the structurally decisive point — versioning uses the gentlest increment the algebra offers: depth 1, which consumes *no separator*. The zeros budget (`zeros ≤ 3`, T4) is therefore never approached; `B6(wⱼ, 1)` holds at every depth, unconditionally. Contrast child-spawning at depth 2, which spends a separator and is budget-bounded (TA5a): version chains have no analogous ceiling. With T0(b) — no maximum tumbler length — the abstract specification requires that fork depth be unbounded. The requirement is mandatory because a fixed cap `C` leaves the system, at the cap, only fatal moves: a further fork of the deepest version must either renumber existing addresses — breaking the permanence V0 guarantees, "new items may be continually inserted in tumbler-space while the other addresses remain valid" [LM 4/19] — or refuse the fork — breaking this very closure, since "all have possible descendants" [LM 4/19]. Call this the *renumber-or-refuse dilemma*; no choice of `C` escapes it, because the obligation falls on the address format, not on storage — the space is *finite but unlimited* [LM 4/22], finitely many versions at any instant with no ceiling baked into the representation. (Gregory's fixed mantissa is exactly such a cap and violates this; see the evidence section.) A version of a version still carries the full chain of descent in its own name because no fork ever disturbs the digits to its left: existing addresses are never modified (V0), so each ancestor's full address survives as a prefix of every descendant's.

**V7 (NavigationAsymmetry).** The two directions of ancestry navigation rest on different resources, and neither is the source's own state:

> *Upward* — from any version, every ancestor is computed by iterated truncation: a pure function of the identity, consulting no state (the same intrinsic-computation discipline as T2).
> *Downward* — the *owned* (address-discoverable) versions of `d` are the registry query `E ∩ S(d, 1) = {c₁, …, c_{hwm}}`, gap-free (VN-B1), so enumeration terminates at the first absentee; the full owned-descendant set `{e ∈ E : d ≺ e}` is T1-contiguous (T5), a single range scan of the ordered registry — and every address-encoded descendant of `d` is owned by `ω(d)`, since no account-tier prefix (`zeros ≤ 1`, O1a) can cover past `d` (`zeros = 2`, Z-mono). Cross-owner versions are *not* recovered here — severed from the source's subtree, they fall outside every address-based descendant scan (VD develops this kernel).
> *Never* — a read of the source's own components: by V3 no `d_src`-indexed state mentions `v`, so there is nothing there to read.

A boundary condition worth recording: in a system whose document-creation discipline emits single-component document fields, the truncation chain bottoms out *syntactically* — truncating the chain's base document strips the last document-field component and leaves a trailing zero, which is not T4-valid — so the base of a version chain is decidable from the identity alone. And a disambiguation: the version has two "parents" in two senses. Its entity-hierarchy parent is the *account* — `parent(v) = parent(d_src)` by K.δ-ID.parent-0/1 — while its derivation ancestor is the *source*, `trunc(v) = d_src`. The address encodes both at once: the account by the unchanged N/U fields (V4c), the derivation by the extended D field.

**VD (VersionNamespaceDiscipline) and the limit of what the digits assert.** Nelson is careful: "In a sense the version, or subdocument number is only an accidental extension of the document number, and strictly implies no specific relationship of derivation" [LM 4/29]. Our formalization must honor the caveat. What V4 proves unconditionally is *allocation lineage*: `v` was baptized under `d_src` (T10a discipline, GlobalUniqueness). The further reading — "`v` is a *version derived from* `d_src`" — is supplied by the operation, not the digits. We therefore record the derivation relation as event-generated:

> **derives** — `derives(v, d)` holds iff some `VERSION(·, d)` invocation produced `v`;

and we state the discipline under which the address decodes it:

> **VD** — every allocation into a version namespace is a fork of its parent: `(A d ∈ E_doc, w ∈ E ∩ S(d, 1) :: w entered E as the output of a VERSION(·, d) invocation)`.

Under VD, ancestry is decidable from the identity alone for the *address-encoded* derivations — those whose result lies in the source's own version stream. Restricted to `v ∈ S(d, 1)`, the registry decides derivation: `derives(v, d) ⟺ v ∈ E`. The restriction is essential, and the *unrestricted* biconditional is false. Its forward direction `derives(v, d) ⟹ v ∈ S(d, 1)` is refuted by the severance theorem: a cross-owner fork `VERSION(π, d)` with `π ≠ ω(d)` makes `derives(v, d)` hold with `v ∈ E` (V0), yet `¬(d ≼ v)` (severance, V9), and since every member of the stream properly extends its parent (`S(d, 1) ⊆ {t : d ≼ t}`, ASN-0040 S1; cf. V4a), `v ∉ S(d, 1)`. Such a derivation escapes every address-based descendant scan — `v` lies in neither `S(d, 1)` nor `{e : d ≺ e}`, so no registry enumeration over the source's subtree reaches it — and is recoverable only through the shared-content witness (V9w), never the registry. Without VD, truncation still yields allocation lineage and nothing more — exactly T6's caution that the document field records who baptized which number, not whose content was copied. VD is coherent as a global discipline because version namespaces collide with nothing: distinct B6-valid namespaces are disjoint (B7), so `S(d, 1)` is untouched by document creation under the account (a different namespace) and by every other document's version stream. The numbers carry the genealogy because the operation is disciplined to make them do so — Nelson's own formulation of the stipulation.

*A worked instance* (the addresses are the implementation's, but the arithmetic is the abstraction's): `d = 1.1.0.1.0.1`; first fork `v₁ = 1.1.0.1.0.1.1`; second fork `v₂ = 1.1.0.1.0.1.2`; fork of the first version `w = 1.1.0.1.0.1.1.1`. Then `trunc(w) = v₁`, `trunc(v₁) = d`, `trunc(d) = 1.1.0.1.0` — trailing zero, not T4-valid: the chain base is found. The account field `1.1.0.1` is identical throughout; the ranks 1, 2 read directly off the final components.

## Ownership and Editability After the Fork

**V8 (OwnershipInheritance).** When the forker owns the source, the version's owner is the source's owner — inherited structurally, with nothing allocated to record it:

> `ω(d_src) = π  ⟹  ω'(v) = π`, with `Π' = Π` and `acct(v) = acct(d_src)`.

*Proof.* Write `coverers(x) = {π'' ∈ Π : pfx(π'') ≼ x}`. We show `coverers(v) = coverers(d_src)`. (⊇) `pfx(π'') ≼ d_src ≺ v` chains to `pfx(π'') ≼ v`: agreement on the shorter prefix's positions composes, and lengths add. (⊆) Suppose `pfx(π'') ≼ v`. Both `pfx(π'')` and `d_src` are prefixes of `v`, hence comparable (Covering-chain lemma, ASN-0042). The case `d_src ≼ pfx(π'')` gives `zeros(pfx(π'')) ≥ zeros(d_src) = 2` by Z-mono, contradicting O1a's account-tier bound; so `pfx(π'') ≼ d_src`. The coverer sets coincide; `ω` selects the unique maximal-length coverer (O2), `Π` is unchanged by the composite (no delegation occurs, O15), so `ω'(v) = ω(d_src)`. ∎

There is no ownership record to create because there is no ownership ledger in the state: ownership is positional (O1, PrefixDetermination), carried by the account-determining fields — and those fields the fork provably does not touch (V4c, V4d). The fork is ownership *exercised*, not transferred: "Whoever owns a specific node, account, document or version may in turn designate (respectively) new … versions, by forking their integers" [LM 4/17].

**V9 (CrossOwnerFork and the Severance Theorem).** When the forker `π` does not own the source — and, per the identity clause's restriction, holds its own document-creation namespace (`zeros(pfx(π)) = 1`), so no principal is minted and `Π' = Π` — three things hold, the first of them a theorem, not a design choice. Write `π_o := ω(d_src)` for the source's owner (the forker is `π`, matching the operation signature `VERSION(π, d_src)`):

> Let `π_o := ω(d_src) ≠ π`. By V-WF, the cross-owner fork allocates `v = nextd(E, π)` as a *single* document-level K.δ in `π`'s account document sub-allocator `A_doc(pfx(π))` — `allocated_by(π, v)` with `v ∈ S(pfx(π), 2)` — carrying `Document(v)` and `zeros(v) = 2`, so `v ∈ E_doc`. One step from that membership gives the positional form V9's arguments rest on: the depth-2 sibling stream contributes `2 − 1 = 1` separator zero (ASN-0040, `S(p, d)`), so `v = [pfx(π)₁, …, pfx(π)_{#pfx(π)}, 0, k]` with `k ≥ 1` — `pfx(π)` extended by one separator zero at position `#pfx(π) + 1` and one nonzero ordinal. The two facts the severance and ownership claims consume now follow *structurally*, discharging what the identity clause deferred:
> — **O5(i), `pfx(π) ≼ v`** — immediate from the displayed form (`v` agrees with `pfx(π)` on positions `1 … #pfx(π)`; SiblingStream S1, ASN-0040), with no appeal to the O5 axiom;
> — **O5(ii), maximality — now a theorem** — let `π'' ∈ Π` cover `v` (`pfx(π'') ≼ v`) and suppose `#pfx(π'') > #pfx(π)`, i.e. `#pfx(π'') ≥ #pfx(π) + 1`. The length-`(#pfx(π) + 1)` prefix of `v` is `w = [pfx(π), 0]` with `zeros(w) = zeros(pfx(π)) + 1 = 2`; since `w` and `pfx(π'')` are both prefixes of `v` with `#w ≤ #pfx(π'')`, `w ≼ pfx(π'')`, so Z-mono gives `zeros(pfx(π'')) ≥ zeros(w) = 2`, contradicting O1a's account-tier bound. Hence `#pfx(π'') ≤ #pfx(π)` for every coverer of `v` — exactly `(A π'' ∈ Π : pfx(π'') ≼ v ⟹ #pfx(π'') ≤ #pfx(π))`.
> Then:
> (a) **Severance** — `¬(d_src ≼ v)`: the new identity *cannot* lie in the source's subtree, so prefix-encoded ancestry is unattainable, not merely omitted;
> (b) **Ownership** — `ω'(v) = π`: the forker owns the fork outright;
> (c) **Editability** — the forker's right to edit `v` follows from (b) and from nothing about the source's permissions, which the operation never consulted (P-src is the entire source-side precondition).

*Proof of (a).* Suppose `d_src ≼ v`. From `ω(d_src) = π_o`: `pfx(π_o) ≼ d_src ≼ v`, so the maximality established structurally in the preamble (O5(ii)) forces `#pfx(π_o) ≤ #pfx(π)`. Both `pfx(π_o)` and `pfx(π)` are prefixes of `v`, hence comparable (Covering-chain); with the length bound, `pfx(π_o) ≼ pfx(π)`. Equality would give `π_o = π` by O1b — excluded — so `pfx(π_o) ≺ pfx(π)`, and by the Prefix postcondition `#pfx(π_o) < #pfx(π)`. Now compare `pfx(π)` with `d_src`: both are prefixes of `v`, hence comparable. `d_src ≼ pfx(π)` gives `zeros(pfx(π)) ≥ 2` by Z-mono, contradicting O1a. So `pfx(π) ≼ d_src` — but then `π` covers `d_src` with a strictly longer prefix than `π_o`'s, contradicting `ω(d_src) = π_o` (the effective owner is the coverer of maximal length, O2). Both branches close; `¬(d_src ≼ v)`. ∎

*Proof of (b).* `pfx(π) ≼ v` by O5(i) (preamble); any coverer of `v` longer than `pfx(π)` would violate the maximality O5(ii) (preamble); `Π' = Π`, so `π` is the maximal coverer and `ω'(v) = π`. ∎

This is Nelson's "versioning by inclusion": "Another user … is free to create his or her own alternative version of the document he or she does not own" [LM 2/32–2/40] — the version is *theirs*, a windowing document over shared material, and "only the owner has a right to withdraw a document or change it" [LM 2/29] now works in the forker's favor on the fork while continuing to protect the source. The operation's permission structure realizes the denial-as-fork shape (O10): the request a non-owner cannot have — mutating another's document — is answered by a fresh, fully-owned document in the requester's own domain, with the source untouched. And the severance theorem sharpens the consultation record's observation: the loss of address-encoded ancestry across ownership boundaries is *forced* by the ownership axioms. What survives is content — which brings us to the witness.

**V9w (SharedContentWitness).** When the fork carries content (`A ≠ ∅`, equivalently `n ≥ 1`), what durably records the cross-owner relationship — and reinforces the owned case — is dual provenance over the shared addresses:

> `(A a ∈ A :: (a, d_src) ∈ R'  ∧  (a, v) ∈ R')`, and both rows persist in every successor state (P2).

The first conjunct holds at `Σ` — and hence at `Σ'`, since the fork adds only `(·, v)` rows (`R ⊆ R'`) — by the composite-boundary property P4★. VERSION is invoked at a composite boundary (P-bdy), where `Contains_C(Σ) ⊆ R` (P4★); each `a ∈ A` is `M(d_src)(u)` for some `u ∈ V_{s_C}(d_src)`, so `(a, d_src) ∈ Contains_C(Σ)` by the definition of content-subspace containment, whence `(a, d_src) ∈ R ⊆ R'`. The second is V13. The witness is *identity-based* — it survives arbitrary later editing of either arrangement, since `R` never shrinks — and it is *symmetric*: it asserts that both documents have contained the same content, not which derived from which. Note also that `origin(a)` need not name `d_src` at all — for content the source had itself transcluded, attribution traces to the true authoring documents (S7); the fork is faithful to authorship rather than to the immediate copying path. Direction, where the state offers it, comes from the address (V4); across ownership boundaries it is simply not in the state — the honest limit of the design, and exactly what the implementation evidence shows: nothing else is recorded. When the source carries no content (`A = ∅` — the `n = 0` case the contract admits, or a links-only source whose content subspace is likewise empty), even this dual-provenance witness is vacuous: a cross-owner fork is then severed by address (V9) and unwitnessed by provenance (`R' ∖ R = A × {v} = ∅`, V13), leaving `v` state-indistinguishable from a document created fresh under `π`'s account — the derivation surviving only as the off-state `derives` event (VD), with no state-level trace whatever.

## Content: Shared, Never Duplicated

**V1 (ZeroContentFootprint).** The fork allocates no content and no links:

> `C' = C  ∧  L' = L`

Whatever the source's extent, the sole allocation is `ΔE = {v}` — exactly one identity minted (V0, in both branches), with no content or link address added. What scales is arrangement and bookkeeping, by content-*position* count rather than byte volume: `ΔM` is one arrangement function on the `n` canonical positions, every image a pre-existing address; `ΔR = A × {v}` with `|A| ≤ n`. This clause is a prohibition, not an economy (G2): fresh-address duplication is forbidden, not merely declined. The unchanged material keeps its original addresses; only content later *written into* the version (out-of-scope operations) would earn new addresses, under the version's own number.

**V2 (ArrangementTranscription).** The version's initial arrangement is the source's content-subspace arrangement — the function itself:

> `M'(v) = M(d_src)|_{V_{s_C}(d_src)}`, so `dom(M'(v)) = V_{s_C}(d_src)` and `ran(M'(v)) = A ⊆ dom(C)`.

Three glosses. *Function-level, representation-free:* the specification constrains the mapping, not its decomposition; by M3 (RepresentationInvariance) and M11/M12 (canonical decomposition exists and is unique), any block structure with this denotation conforms — an implementation may re-derive the mapping from the source and consolidate adjacent runs freely, and no observer can tell. *Snapshot:* the right-hand side is evaluated at `Σ`; the version captures the source *as it stands at the fork* — the fork-time snapshot whose orthogonality to rank V5 draws out. *Content subspace only:* the link subspace of `v` is empty at birth, and necessarily so —

**V2b (ForeignLinkExclusion).** No reachable transition seats a link of foreign origin in any document's link subspace:

> `(A d, x : x ∈ dom(M(d)) ∧ subspace(x) = s_L : origin(M(d)(x)) = d)` (CL-OWN), and the sole link-subspace extension transition carries precondition `origin(ℓ) = d` (K.μ⁺_L);

every link the source arranges has `origin = d_src ≠ v`, so the fork *cannot* carry the source's link arrangement — not as a choice it declines but as a transition that does not exist. Cross-fork connectivity therefore has exactly one channel, content anchoring, and V10 shows that channel is total over what the question asks of it.

**V13 (ProvenanceCoupling).** The provenance clause is pinned, both ways, by the couplings:

> `R' = R ∪ {(a, v) : a ∈ A}` — J1★ forces every pair in (each carried address is range-new in `v`'s content subspace), and J1'★ forbids any pair beyond.

Each row is permanent (P2). Together with structural attribution — every `a` carries its authoring document in its own digits, `origin(a)` (S7) — the post-state supports "who wrote what" queries structurally: the version's range partitions by origin into the spans its sources authored, which is the ground on which royalty apportionment ("the original owner and the modifier split the royalty in proportion to who wrote what" [LM 2/45]) can stand. The apportionment itself is out of scope; the fork's obligation is to keep the ground true, which V1 and V13 jointly do.

**V12 (IdentityContentBoundary).** Forking exhibits, at the hard case, that identity and content are independently variable:

> at `Σ'`: `d_src ≠ v` (V0), yet `M'(v) = M'(d_src)|_{V_{s_C}(d_src)}` (V2 with V3) — two identities, one body of content.

The map from identity to content-subspace arrangement is therefore non-injective *by construction*: identity is not recoverable from content, however total the content. Conversely the content bears no mark of the identities arranging it — `C` is untouched, and the same address serves both documents (the sharing that S5, UnrestrictedSharing, ASN-0036, guarantees possible, here made actual by a single operation). A document created empty separates the two notions by absence — a name with nothing under it — and leaves open the misreading that identity is merely the label of a content-aggregate awaiting accretion. The fork separates them at *totality*: a document whose every arranged byte was authored elsewhere is nonetheless a distinct document with a distinct, permanent address. One caution closes the loop (V4's honesty clause, [LM 4/29]): the new identity *asserts* allocation lineage and nothing else; derivation is the operation's act, witnessed by VD and `R`, not a property the digits could carry alone. And no version is privileged: both documents stand in `E_doc` with identical operational standing — "there is no 'basic' version of a document set apart from other versions" [LM 2/19] — the source's only structural distinction being that it is the truncation image of its forks.

## After the Fork: Independence and Carry-Through

**V11 (EditIndependence).** The version is independently editable from the instant of its creation, and the independence is mutual:

> (a) *Immediacy* — `v ∈ E'_doc` with `ω'(v)` the forker: the version stands under the same enabling conditions as any allocated document, with nothing `v`-specific outstanding — no allocation, registration, or unlock owed first on `v`'s account (an `n = 0` fork has nothing to contract or reorder, exactly as an empty document does not). Mutation authority follows ownership ("only the owner has a right to … change it" [LM 2/29]), a discipline the ownership model layers on the transition vocabulary.
> (b) *Isolation, both directions* — every arrangement transition names one document `d` and frames all others: `(A d' : d' ≠ d : M''(d') = M'(d'))` (the K.μ family). By induction over any subsequent transition sequence, edits scoped to `v` leave `M(d_src)` pointwise fixed, and edits scoped to `d_src` leave `M(v)` pointwise fixed.
> (c) *The shared substance is beyond reach from either side* — `(A a ∈ dom(C) :: C''(a) = C'(a))` (P0): "deletion" at either document is contraction of that document's own arrangement, never a write to the store or to the other's arrangement. The bytes "remain in all other documents where they have been included" [LM 4/11].

On the snapshot question, the design record draws a line our model makes precise. The stored fact is the snapshot: `M'(v)` is pinned by `M(d_src)` at the boundary, and (b) guarantees no later source edit reaches it — that direction is the *required* half. The reverse direction — a derivative that *tracks* its source, Nelson's window "fixed at a relatively fixed location in the document space, in which case updates are seen automatically" [LM 2/37] — is not a propagation channel into `M(v)` and must not be one; it is realizable as a *read-time query* against the evolving source, which shared identity keeps computable ("What has this passage become?"). Both of Nelson's window modes thus coexist with the isolation invariant, which is why the architecture need not — and does not — choose between them at the fork.

**V10 (LinkCarryThrough).** Links anchored to content the version transcludes carry through, totally, structurally, and at zero cost to the operation:

> `(A a ∈ dom(Σ'.L), i : 1 ≤ i ≤ |Σ'.L(a)| : project(a, i, v, Σ') ≠ ∅  ⟺  coverage(Σ.L(a).eᵢ) ∩ A ≠ ∅)`

*Proof.* `L' = L` (V1), so `Σ'.L(a) = Σ.L(a)` for every `a ∈ dom(Σ.L)` and in particular `coverage(Σ'.L(a).eᵢ) = coverage(Σ.L(a).eᵢ)` — the right-hand side may be read at `Σ`. `ran(Σ'.M(v)) = A` (V2). LP12's per-slot biconditional — `project(a, i, d, Σ') ≠ ∅ ⟺ coverage ∩ ran(Σ'.M(d)) ≠ ∅` — instantiated at `d = v` gives the claim. ∎

Four corollaries discharge the question's demands.

*(i) Totality and refraction.* Every link any of whose slots covers at least one carried address is discoverable from the version at the fork boundary; and since the source's arrangement is untouched (V3), it remains discoverable from the source — shared anchors make a link discoverable from both documents at once (the LP16 shape). This is "a link to one version of a Prismatic Document is a link to all versions" [LM 2/26], scoped exactly as the architecture can honor it: to the versions holding the anchor's address.

*(ii) Zero per-link work.* The operation neither reads nor writes `L` (frame). Carry-through is not a migration the fork performs per link; it is a consequence of the address algebra — the link "is not between points, but between spans of data … a strap between bytes" [LM 4/42], the version holds the very same bytes, so the straps hold in the version. The guarantee extends to links created *after* the fork against the shared addresses: such a link did not exist when the version was forked — the fork could not have migrated it, knowing nothing of it — yet it reaches the version, by LP12 evaluated at the later state, *provided* the version still arranges the anchor's address there; if an intervening contraction has removed it, corollary (iii) governs.

*(iii) Conditionality, exactly Nelson's.* "If any of the bytes are left to which a link is attached, that link remains on them" [LM 4/42]. Should the version's owner later contract `M(v)` off some addresses, links anchored only there cease to be discoverable *from the version* (LP10) while persisting unchanged in the store (L12, LP13) and remaining discoverable from every document still arranging the addresses — the source included, whose hold the fork never weakens.

*(iv) The boundary of the guarantee.* Anchors in the source's *link subspace* — links about links, arranged at `d_src` — do not carry, and cannot (V2b). The guarantee is precisely over content anchors: links anchored to content the version transcludes, which is what the design owes and all it owes.

*A worked instance of carry-through* (same convention as the addressing instance above — the addresses are the implementation's, the arithmetic the abstraction's — and we reuse its source `d_src = 1.1.0.1.0.1`). Let `d_src` arrange three content positions over two addresses, the first shared:

> `M(d_src) = { [1,1] ↦ a₁,  [1,2] ↦ a₂,  [1,3] ↦ a₁ }`,  with  `a₁ = 1.1.0.1.0.1.0.1.1`,  `a₂ = 1.1.0.1.0.1.0.1.2`

— element-level content addresses (`zeros = 3`, element field `[s_C, k]`, subspace `s_C = 1`). Then `n = |V_{s_C}(d_src)| = 3`, while `A = {a₁, a₂}` has `|A| = 2 < n`: the repeat at `[1,3]` is the gap. Let `ℓ ∈ dom(L)` be a link whose from-slot anchors the shared address — `Σ.L(ℓ).e₁ = {(a₁, δ(1, #a₁))}`, the unit-depth span whose `coverage` is the subtree `{t : a₁ ≼ t}` (PrefixSpanCoverage) — its other slots immaterial here beyond L3's non-empty type. The fork is owned (`π = ω(d_src)`); with no prior versions, `v = nextv(E, d_src) = inc(d_src, 1) = 1.1.0.1.0.1.1`, the `v₁` of the addressing instance. The four content-connectivity postconditions, read against the named `Σ'`:

> *V2* — `M'(v) = M(d_src)|_{V_{s_C}(d_src)}`: `M'(v)([1,1]) = a₁`, `M'(v)([1,2]) = a₂`, `M'(v)([1,3]) = a₁`; `ran(M'(v)) = A`.
> *V13* — `R' ∖ R = {(a, v) : a ∈ A} = {(a₁, v), (a₂, v)}`: two rows for three transcribed positions, since the row `(a₁, v)` demanded by both `[1,1]` and `[1,3]` is one set element. Provenance counts shared addresses, not positions — `|R' ∖ R| = |A| = 2`, the concrete face of `|A| < n`.
> *V9w* — at the boundary, `a₁ = M(d_src)([1,1])` gives `(a₁, d_src) ∈ Contains_C(Σ) ⊆ R` (P4★), and `(a₁, v) ∈ R' ∖ R` (V13); so `{(a₁, d_src), (a₁, v)} ⊆ R'`, and symmetrically for `a₂`. Each shared address is witnessed under both names, permanently (P2).
> *V10* — by SA the sole stored member of `a₁`'s subtree is `a₁` itself (`a₁ ⋠ a₂`: they share length 9 and disagree at the final component), so `coverage(Σ.L(ℓ).e₁) ∩ ran(M'(v)) = {t : a₁ ≼ t} ∩ {a₁, a₂} = {a₁}`; the version positions holding `a₁` are `[1,1]` and `[1,3]`, whence `project(ℓ, 1, v, Σ') = {[1,1], [1,3]} ≠ ∅`. The biconditional's right side reads `coverage(Σ.L(ℓ).e₁) ∩ A = {a₁} ≠ ∅` — both sides true. The single link refracts onto the version, landing on *every* position that holds the shared address, not merely one.

Refraction is then literal in both directions: the source arrangement is untouched (V3), so `project(ℓ, 1, d_src, Σ') = {[1,1], [1,3]}` equally — at `Σ'` the one link `ℓ` is discoverable from `d_src` and `v` at once, "a link to one version of a Prismatic Document is a link to all versions" [LM 2/26], exhibited on the specific shared addresses rather than by abstract instantiation of LP12.

*A cross-owner worked instance* (the same source, content, and link as the carry-through instance above; only the fork's ownership changes, so the contrast isolates exactly the cross-owner delta). The source's owner is the account `π_o` with `pfx(π_o) = acct(d_src) = 1.1.0.1` (`zeros = 1`). Let the forker be a *second* account under the same node — `π` with `pfx(π) = 1.1.0.2` (`zeros(pfx(π)) = 1`, discharging the branch's account-tier restriction; `π ≠ π_o` by O1b, since `1.1.0.2 ≠ 1.1.0.1`). `VERSION(π, d_src)` mints `v = nextd(E, π)` as a single document K.δ in `π`'s own document namespace — with no prior documents under `π`, the `k = 2` descent `v = inc(pfx(π), 2) = 1.1.0.2.0.1` (`N(v) = 1.1`, `U(v) = 2`, `D(v) = 1`, `zeros(v) = 2`, a Document; `pfx(π) = 1.1.0.2 ≼ v` by O5(i)). The branch's distinctive postconditions, read against `Σ'`:

> *V9(a), severance* — `¬(d_src ≼ v)`, exhibited at the divergence position. The identities `d_src = 1.1.0.1.0.1` and `v = 1.1.0.2.0.1` agree on positions 1–3 (node field `1.1` and its separator) and **diverge at position 4** — the last component of the account field, where `d_src` carries the source-owner's user digit `1` and `v` the forker's `2`. A disagreement at position `4 ≤ #d_src` makes neither tumbler a prefix of the other (T1), so `d_src ⋠ v`: prefix-encoded ancestry is *unattainable* here, not merely omitted. The severance is structural, not an artifact of these digits — the account prefixes `pfx(π_o) = 1.1.0.1 ≼ d_src` and `pfx(π) = 1.1.0.2 ≼ v` are themselves account-tier (`zeros = 1`) and already split at position 4, so `v`'s account cannot nest under `d_src`'s; V9(a)'s abstract argument made arithmetic.
> *V0 / V1, single mint* — `E' = E ∪ {v}` with `v` fresh, and `C' = C ∧ L' = L`. `π`'s account `1.1.0.2 ∈ E` already carries its document sub-allocator (PS incumbency), so the fork is one document K.δ growing the registry by the single identity `v` (V0).
> *V9(b), ownership* — `ω'(v) = π`: `pfx(π) = 1.1.0.2` covers `v`, and no account-tier prefix (`zeros ≤ 1`, O1a) covers `v` more specifically — every prefix of `v` longer than `pfx(π)` carries `zeros ≥ 2` (the length-5 `1.1.0.2.0` and `v` itself) — so `π` is the maximal coverer (O2). The fork is the forker's outright, the source's permissions never read (P-src is the whole source-side precondition).
> *V9w / V10, the content witness survives severance* — the branch's point. Transcription never consults `v`'s address: `M'(v) = M(d_src)|_{V_{s_C}(d_src)}` again maps `[1,1] ↦ a₁`, `[1,2] ↦ a₂`, `[1,3] ↦ a₁`, with `ran(M'(v)) = A = {a₁, a₂}` — *the same* `a₁ = 1.1.0.1.0.1.0.1.1`, `a₂ = …2`, digits unchanged, still carrying `origin = d_src` (S7). So V9w lands precisely as in the owned reading: `(a₁, d_src) ∈ R` (P4★ at the boundary, P-bdy) and `(a₁, v) ∈ R' ∖ R` (V13) give `{(a₁, d_src), (a₁, v)} ⊆ R'`, permanent (P2), and symmetrically for `a₂`. And V10 carries through identically: `coverage(Σ.L(ℓ).e₁) ∩ A = {a₁} ≠ ∅`, so `project(ℓ, 1, v, Σ') = {[1,1], [1,3]} ≠ ∅` — the link refracts onto the cross-owner version, landing on every position that holds `a₁`, *though* `d_src ⋠ v`. The orthogonality is now concrete: severance is a fact about `v`'s digits, carry-through a fact about the digits of `A` — which the fork leaves untouched — so exactly where the address can no longer encode the tie (V7's downward limit), the shared content still witnesses it (V9w).

## The Implementation Evidence

Gregory's udanax-green realizes the contract with high fidelity; we record the correspondences and then the deviations, citing the consultation findings.

*Allocation is the registry frontier, literally.* `docreatenewversion` builds a DOCUMENT→DOCUMENT hint and `findisatoinsertnonmolecule` computes the new identity by querying the granfilade for the maximum existing key under the source (`findpreviousisagr`), truncating to version depth and incrementing — `inc(d, 1)` when no child exists, `inc(max-child, 0)` thereafter. This is `nextv` verbatim: stateless against the store (V5's registry-purity), monotone, gap-free. The golden tests pin the addresses: `1.1.0.1.0.1` forks to `1.1.0.1.0.1.1` then `1.1.0.1.0.1.2`; the fork of the first version is `1.1.0.1.0.1.1.1` (V4, V6). The single appended digit — no `.0.` separator — confirms depth-1 allocation and `zeros` constant at 2.

*Zero content writes.* The only granfilade write is one GRANORGL entry — the new document, with an empty POOM; no GRANTEXT path is reachable from `docreatenewversion`. The version's V→I entries and its spanfilade DOCISPAN entries scale with the source's span count, never its byte volume (V1). The source's DOCISPAN family and the version's coexist permanently and independently — the implementation's form of V9w's dual witness.

*Transcription is re-derived, which the spec licenses.* `docopyinternal` does not copy the source's tree nodes; it resolves the source's V-span to I-spans (`specset2ispanset`) and inserts them into the fresh POOM (`insertpm`), where same-origin adjacent runs may coalesce. The mapping is preserved; the decomposition may differ — exactly the freedom V2's function-level statement grants via representation invariance.

*The source is read-only, without authority.* The source is read via `NOBERTREQUIRED` — no permission check of any kind — and nothing is written to its orgl or POOM: no forward pointer, no child list, no flag (V3, V7). The ownership check (`isthisusersdocument`) selects only the allocation hint. In the cross-owner branch the hint switches to account-anchored document creation (`makehint(ACCOUNT, DOCUMENT, …)`) — though which tumbler actually anchors it differs by code path, and the FEBE path gets it wrong (deviation 4) — the `homedoc` field of the copied spans is cleared, and no back-pointer of any kind is stored: the version is structurally indistinguishable from a fresh document, and ancestry is recoverable only through shared I-addresses — the open-mode path living out the severance theorem (V9) and its witness (V9w).

*Snapshot semantics.* The golden test `modify_original_after_version` shows the version frozen at the source's fork-time content while the source moves on; two successive forks bracketing an edit hold different mappings under consecutive ranks (V2, V5, V11).

*Whole-request isolation.* The backend is single-threaded and run-to-completion: one event loop dispatches one request handler at a time, and `docreatenewversion`'s allocate–retrieve–populate steps all execute, in memory, within one invocation before any other request from any session can be dispatched; disk writes are deferred to idle. The composite's interior state — orgl allocated, POOM unpopulated — exists only inside that invocation: the serialization the atomicity remark leaves to the implementation, realized architecturally rather than specified. One edge the architecture leaves open: a failure between allocation and population returns early without cleanup, leaving an orphaned empty document in the granfilade — a committed K.δ without the rest of its composite — though the request reports failure and the orphan's address is never released to a client.

Four deviations from the abstract specification:

1. **Bounded fork depth.** `NPLACES = 16` caps the tumbler mantissa; `tumblerincrement` aborts fatally on overflow, bounding fork depth at roughly ten levels above a six-component document. The comment "increased from 11 to support deeper version chains" records that the previous bound was hit in practice — the cap is not hypothetical. This violates V6's unbounded-depth requirement (T0(b)): the defect is the *existence* of a fixed cap, which falls to V6's renumber-or-refuse dilemma at whatever value it is set. A conforming implementation must *remove* the cap (variable-length tumblers), not merely enlarge it.
2. **The subspace boundary is unimplemented at extraction.** `doretrievedocvspanfoo` — self-described "a kluge not yet kluged" — returns the source's *total* V-extent with no subspace logic; its body is identical to the whole-document retrieval. For text-only sources this coincides with V2's clause; for a links-only source the link extent is mis-transcribed as if it were content, and `acceptablevsa`, the would-be validator, is a stub returning TRUE. V2's content-subspace restriction is the specification the kluge acknowledges it owes. (Versioning an empty document, by contrast, behaves exactly as the degenerate `n = 0` case prescribes.)
3. **Session-layer ceremony.** The `addtoopen` / `logbertmodified` / `doclose` dance around the new version exists to keep the fresh document from being garbage-collected as created-but-unwritten. The abstract model has no such collection — identities are permanent unconditionally (P1) — so this machinery has no specification counterpart.
4. **The principal structure is cooperative, not enforced.** PS has no backend mechanism behind it. The account validator (`validaccount`) is a stub returning TRUE; the `XACCOUNT` handler discards even that result and installs the client-supplied account tumbler unconditionally; the multi-user daemon never initializes a session's account at connection (the call is commented out). Allocation is containment-checked against the *claimed* account — the allocator verifies the new address lands under the session's account prefix — but never legitimacy-checked: nothing verifies the prefix names any principal. And the FEBE CREATENEWVERSION handler passes the *source document's* address as the cross-owner placement anchor (the open-mode `doopen` path correctly passes the session account), seating the fork inside the foreign document's own namespace — violating the O5 confinement PS assumes, and with it the severance theorem's hypothesis. A conforming implementation must enforce what udanax-green leaves to front-end cooperation: a real `Π`, coverage of `E`, allocation confined to the allocator's domain.

## Claims Introduced

| Label | Statement | Status |
|-------|-----------|--------|
| PS | standing assumption: an ASN-0042-conforming principal structure rides the docuverse states — O1a/O1b with O12/O13/O15 dynamics, `allocated_by` on K.δ under O5/O16, bootstrap coverage `pfx(π₀) ≼ n₀`, and incumbency `pfx(π) ∈ E` (every principal occupies a baptized entity, via PrefixBaptismCoupling + the entity-level `B`=`E` identification) — whence `ω : E → Π` is total at every reachable state | introduced |
| trunc | `trunc(t)` is the `(#t − 1)`-prefix of `t`; for every `v ∈ S(d, 1)`, `trunc(v) = d` | introduced |
| Z-mono | `p ≼ q ⟹ zeros(p) ≤ zeros(q)` | introduced |
| SA | stored addresses form a prefix antichain: for `a ∈ dom(C) ∪ dom(L)`, `{t : a ≼ t} ∩ (dom(C) ∪ dom(L)) = {a}` | introduced |
| nextv, nextd | `nextv(E, d) = next(E, d, 1)` and `nextd(E, π) = next(E, pfx(π), 2)` — the version- and account-document-namespace frontiers, each a function of the registry and the base address alone | introduced |
| VN-B1 | for every B6-valid document-producing namespace `(p, g)` (`g ∈ {1, 2}`, `zeros(p) + (g − 1) = 2` — covering version streams `S(d, 1)` and account-document streams `S(pfx(π), 2)`), `E ∩ S(p, g)` is a contiguous prefix at every reachable state: K.δ's freshness and operand constraints admit only frontier arrivals | introduced |
| VERSION | the fork composite: one fresh identity, the source's content-subspace arrangement transcribed as a snapshot, provenance recorded; `C`, `L`, and every existing arrangement framed | introduced |
| V-WF | VERSION is a valid composite at every reachable state with `d_src ∈ E_doc` (empty source included; the cross-owner branch additionally requires an account-tier forker, `zeros(pfx(π)) = 1`, so exactly one identity is minted); invoked at a composite boundary (P-bdy), its post-state is the terminal boundary, satisfying both the per-state invariants and the composite-boundary properties P4★ ∧ P4a ∧ P7a | introduced |
| derives | `derives(v, d)` iff some `VERSION(·, d)` invocation produced `v` | introduced |
| VD | version namespaces are populated only by VERSION with the parent as named source; under VD the registry decides address-encoded derivation — for `v ∈ S(d, 1)`, `derives(v, d) ⟺ v ∈ E`; the unrestricted forward direction `derives(v, d) ⟹ v ∈ S(d, 1)` fails for cross-owner forks (severance, V9) | introduced |
| V0 | exactly one fresh identity is allocated; globally unique across all allocation events; permanent and never renumbered | introduced |
| V1 | `C' = C ∧ L' = L` — zero content/link allocation, one identity minted, regardless of source size; the arrangement and provenance deltas scale with content-position count, not byte volume | introduced |
| V2 | `M'(v) = M(d_src)|_{V_{s_C}(d_src)}` — function-level snapshot at fork time; representation-free; content subspace only | introduced |
| V2b | no reachable transition seats a foreign-origin link in a document's link subspace; content anchoring is the only cross-fork connectivity channel | introduced |
| V3 | source frame: every `d_src`-indexed state component is unchanged; the fork is strictly additive and writes no forward pointer | introduced |
| V4 | owned fork: `v ∈ S(d_src, 1)`, `trunc(v) = d_src`, `zeros(v) = 2`, node/user fields preserved, document field extended by exactly one component, `acct(v) = acct(d_src)` | introduced |
| V5 | the version's rank equals its allocation order in the namespace — fork order exactly under VD; allocation is registry-pure (content-blind); ranks are never reused; time is not encoded | introduced |
| V6 | closure: the output satisfies the operation's preconditions; depth-1 forking never consumes the separator budget; fork depth must be unbounded; ancestors are recovered by iterated truncation | introduced |
| V7 | navigation asymmetry: child→ancestor is stateless address arithmetic; ancestor→descendant is a registry query over the *owned* versions (contiguous, gap-free; cross-owner versions are severed from the address scan, recoverable only via V9w); neither reads the source's state | introduced |
| V8 | owned fork inherits ownership: `ω'(v) = ω(d_src)` via coverer-set equality; no ownership record is created | introduced |
| V9 | cross-owner fork: `ω'(v)` is the forker; `¬(d_src ≼ v)` is a theorem (severance); no authority over the source is consulted; editability follows from owning the fork | introduced |
| V9w | for `A ≠ ∅`: dual provenance `{(a, d_src), (a, v)} ⊆ R'` for every carried `a` (source-side row via P4★ at the boundary invocation, P-bdy; version-side via V13); permanent, identity-based, and symmetric — it does not orient derivation. For a content-empty (or links-only) cross-owner source (`A = ∅`): the witness is vacuous, leaving no state-level trace of the derivation, which survives only as the `derives` event (VD) | introduced |
| V10 | link carry-through: `project(a, i, v, Σ') ≠ ∅ ⟺ coverage(Σ.L(a).eᵢ) ∩ A ≠ ∅` — total over content anchors, zero per-link work; extends to links created after the fork while the version still arranges the anchor | introduced |
| V11 | the version stands under the same edit-enabling conditions as any document from the instant of creation, nothing `v`-specific outstanding; per-document frames isolate both edit directions; shared content is immutable from either side; source-tracking is a query, not a channel | introduced |
| V12 | identity/content boundary: two distinct identities over one content body; identity is not a function of content; the address asserts allocation lineage, the operation supplies derivation | introduced |
| V13 | `R' = R ∪ {(a, v) : a ∈ A}`, pinned from below by J1★ and from above by J1'★ | introduced |

## Open Questions

- What invariant must govern allocations into a document's version namespace by operations other than versioning, if ancestry decoded from the identity alone is to remain sound?
- What guarantee, if any, must make the direction of derivation recoverable when the only surviving witness of a cross-ownership fork is symmetric shared-content provenance?
- Beyond the fork — whose content-anchoring obligation V2b and V10 settle as complete — must any separate, non-fork mechanism make link-subspace material (links about links) itself versionable across derivations?
- What serialization guarantee must version allocation provide when concurrent forks of one source are attempted under a single owning authority?
- What must a version guarantee about its relationship to later source states for location-fixed windowing to be realizable without breaching arrangement isolation?
- Can any withdrawal or supersession semantics coexist with identity permanence and cross-version link carry-through, and under what constraints?
- What does a provenance record assert once the arrangement entry that justified it has been contracted away, and what guarantee separates historical containment from derivation evidence?
- What minimum shared-identity guarantee must survive arbitrary divergence of the two arrangements for version correspondence to remain computable?
