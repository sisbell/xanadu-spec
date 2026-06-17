> **ASN-0108 · FINDNEXTNLINKSFROMTOTHREE Operation** — condensed claim statements  
> [← Full note](ASN-0108-findnextnlinksfromtothree-operation.md) · [↑ Operations index](README.md) · [↑ Spec home](../README.md)

---

# ASN-0108 Claim Statements

*Source: ASN-0108-findnextnlinksfromtothree-operation.md (revised 2026-06-13) — Extracted: 2026-06-13*

## Definition — MatchingSet

```
Match(q, Σ) ⊆ dom(Σ.L)

the links the request presently reaches, determined at each state Σ by query q.

(M-fin)  Match(q, Σ) is finite, being a subset of the finite dom(Σ.L) (L-fin, ASN-0043).
(M-mut)  Match(q, ·) is not monotone in state evolution (ASN-0127 D-NONMONO):
         findlinks_V is non-monotone across Σ →* Σ'.
         It may gain members — a link created between calls (K.λ case; L12a adds to dom(Σ.L))
         and it may lose members — a link whose matched endpoint content is removed from the
         consulted arrangement d_q (K.μ⁻ case; coverage ∩ ran(Σ.M(d_q)) = ∅ of ASN-0098 LP12).

Under the discoverability reading:
  Match(q, Σ) = findlinks_V(W, d_q, Σ)   (F-V, ASN-0127)
  reducing at full region to {a ∈ dom(Σ.L) : discoverable_from(a, d_q, Σ)}  (F-FULL)
```

## Definition — EnumerationOrder

```
a ≺ b  ≡  κ(a) <_K κ(b)

where κ : Match → K is an ordering key into a totally-ordered set (K, <_K)
that is total and injective on Match(q, Σ).
```

## Definition — SuccessorSet

```
After(c, Σ)  =  { a ∈ Match(q, Σ) : κ(c) <_K κ(a) }
After(⊥, Σ)  =  Match(q, Σ)
```

## Definition — WindowFunction

```
Window(q, c, N, Σ)  =  the ≺-least min(N, |After(c, Σ)|) elements of After(c, Σ)

next-cursor  =  ≺-maximum of Window(q, c, N, Σ)   if Window is non-empty
             =  c unchanged                          if Window is empty
```

---

## Match — MatchingSet (DEF, definition)

`Match(q, Σ) ⊆ dom(Σ.L)` — the finite (M-fin / L-fin), non-monotone (M-mut = ASN-0127 D-NONMONO) matching set windowing delivers, under the discoverability reading (ASN-0127 `findlinks_V`, F-V/F-FULL)

## κ / ≺ — EnumerationOrder (DEF, definition)

`a ≺ b ≡ κ(a) <_K κ(b)` — enumeration order from a *total, injective* ordering key into a total order

## After — SuccessorSet (DEF, definition)

`After(c, Σ) = {a ∈ Match(q, Σ) : κ(c) <_K κ(a)}`; `After(⊥, Σ) = Match(q, Σ)` — the successor set

## Window — WindowFunction (DEF, definition)

`Window(q, c, N, Σ)` = the `≺`-least `min(N, |After(c, Σ)|)` elements of `After(c, Σ)`; next cursor = its `≺`-max

---

## W0 — TotalEnumerationOrder (PROP, predicate)

On `Match(q, Σ)`, the relation `≺` induced by a key `κ` that is *total and injective on `Match`* into a totally-ordered codomain is a strict total order: irreflexive and transitive (inherited from `<_K`), and trichotomous (any two distinct matching links are `≺`-comparable, since totality gives `κ(a)` and `κ(b)` both defined, injectivity gives `κ(a) ≠ κ(b)`, and `<_K` is total). A finite totally-ordered set has a unique enumeration; so `Match(q, Σ)` has a unique listing `a_1 ≺ a_2 ≺ … ≺ a_m`. But totality is not automatic: a key read from a *fixed* endset slice can be undefined on a link whose slice has empty coverage, so securing it constrains the slice.

## W1 — PositionUniqueness (PROP, predicate)

No two distinct matching links occupy the same enumeration position. The rank map `rank_Σ : Match(q, Σ) → {1, …, m}` sending the `≺`-least to `1` and counting up is a bijection; in particular it is injective. This is the formal content of "no two links could ever share a position": it is exactly the injectivity of `κ` on the matching set, and it is what lets "the `k`-th matching link" denote unambiguously.

## W2 — CursorByIdentity (PROP, predicate)

The cursor is the *identity* of the last link returned — a permanent link address — and not a positional offset into the result list. Resumption is defined relative to the cursor's key, not relative to a count of how many links preceded it.

Sub-claim — weakest precondition:

> Write the **successor set**
> `After(c, Σ)  =  { a ∈ Match(q, Σ) : κ(c) <_K κ(a) }`,  and  `After(⊥, Σ) = Match(q, Σ)`.
>
> *Identity cursor.*
> `wp(resume_id, R)  ≡  κ(c) computable`
>
> *Offset cursor.*
> `frozen-prefix(resume_offset)  ≡  |{a ∈ Match(q, Σ') : κ(a) ≤_K κ(c)}| = j`
>
> `wp(resume_offset, R)  ≡  j' = j  ∨  (j ≥ m' ∧ j' ≥ m')`
>
> where `j' = |{a ∈ Match(q, Σ') : κ(a) ≤_K κ(c)}|` and `m' = |Match(q, Σ')|`.
>
> Three conditions nest strictly: membership-identity ⟹ frozen-prefix `j' = j` ⟹ the genuine weakest.

## W3 — DeterministicWindow (PROP, predicate)

`Window(q, c, N, Σ)` is a deterministic function of its arguments alone. It reads no hidden per-reader session state. Two requests with the same `(q, c, N)` against the same `Σ` return the identical batch. The protocol is *stateless* in the sense that the server retains nothing between calls: the entire continuation state the reader needs is the cursor it carries, and re-presenting the same cursor re-derives the same continuation.

## W4 — PartitionUnderStability (LEMMA, lemma)

Against a fixed `(M, κ)`, the successive windows `W_0, W_1, W_2, …` — taken under *any* schedule of per-call sizes `N_i ≥ 1`, possibly a different size on each call — are pairwise disjoint, consecutive in `≺`, and their union is all of `M`, each link appearing exactly once. The reader sees every matching link, in `≺`-order, with no gap and no repeat.

Formal induction:

> Let `M = {a_1 ≺ a_2 ≺ … ≺ a_m}` (W0). Define cumulative cut-points `S_0 = 0` and `S_{i+1} = S_i + N_i`. `W_i` is the block of ranks `[S_i+1, … , min(S_{i+1}, m)]`. The bound function `t_i = |After(c_i, Σ)| = m - S_i` strictly decreases by `N_i ≥ 1` per full-batch call; the loop stops on the first window that is short (fewer than `N_i` requested).

## W5 — OrderStability (LEMMA, lemma)

The key satisfies **clause 1 (cut-point preservation)** at a cursor `c` across a transition `Σ → Σ'` when, for every `a` matching in *both* states,

> `κ_{Σ'}(c) <_K κ_{Σ'}(a) ⟺ κ_Σ(c) <_K κ_Σ(a)`

the cursor's classification of each surviving link as ahead of or behind it does not move.

**Claim:** if clause 1 holds at every cursor the reader actually holds — each across the state change between the call that set that cursor and the next — then resumption is *coherent*, in two scoped guarantees:

- *No re-delivery* (**unconditional** — needs no termination hypothesis): no already-seen link that matches *continuously* across the intervening states — from its delivery until a later window could re-deliver it — is delivered again.
- *No skip* (**under the hypothesis that the pass terminates**): no link that *remains* an undelivered tail matcher — matching *continuously* across the intervening states, from the state in which its cursor was set until a later window can reach it — is missed.

Clause 1 at the held cursors is *sufficient* but **not necessary**: coherence is inherently a *whole-pass* property, not a per-cursor one. A link matching in only one of the relevant states cannot be "delivered exactly once" and falls outside both.

**Clause 2 (tail-order preservation)**: for every pair `a, b` lying in the tail in `Σ` (`κ_Σ(c) <_K κ_Σ(a)` and `κ_Σ(c) <_K κ_Σ(b)`),

> `κ_{Σ'}(a) <_K κ_{Σ'}(b) ⟺ κ_Σ(a) <_K κ_Σ(b)`

is *not* necessary for coherence: it governs only *which order* the surviving tail is served in, never *whether* a tail link is reached.

## W6 — AllocationMonotoneKeyGivesMonotoneAppend (LEMMA, lemma)

If the ordering key is *allocation-monotone* — a link allocated later receives a key strictly greater than every key already issued — then a newly created matching link is `≺`-greater than every previously enumerated link: append-at-tail.

- Under an address-based key the forward hypothesis holds within a single home document's link allocator, whose successive links carry strictly increasing addresses (foundation T9, forward allocation; a single allocator chain is strictly increasing).
- Under *either content-drawn key* — Gregory's least-covered-tumbler key or the content-position foil — the hypothesis fails: a new link's key is drawn from *its endpoint content* — the least tumbler its endset covers (Gregory's reading) or its current V-position alike — and that content may sort anywhere among the existing keys, before, between, or after, so the append-at-tail conclusion is unavailable.

## W6a — CreationDoesNotDisturbSeenLinks (LEMMA, lemma)

For any key that is a function of `(address, matched-content boundary or position)`, the *creation* of `a_new` does not alter the key or the relative order of any already-enumerated link.

Formal statement:

> Link creation is a `K.λ` operation (ASN-0093), whose frame leaves the arrangement family `M` and the content store `C` unchanged and adds only a fresh address to `dom(Σ.L)` (addresses are never reused). So creation alters neither any existing link's *address* (no address is reused), nor any existing link's *matched-content boundary* (its endset is immutable by L12, and `K.λ` adds only a fresh entry to `dom(Σ.L)`), nor any existing link's *matched-content position* (`M` and `C` are framed, so no matched endpoint moves).
>
> Since `image(W, d_q, Σ') = image(W, d_q, Σ) =: I` (frozen by the K.λ frame) and F-LAMBDA at fixed `I` gives:
>
> `Match(q, Σ') = Match(q, Σ) ⊎ ({ℓ_new} if ℓ_new matches at Σ', else ∅)`
>
> the fresh link is added *disjointly*, without removing from or disturbing the prior matchers. The hazard of W6 is one of omission (a new link landing behind the cursor), never of duplication or reordering of delivered links.

## W7 — ResultMembershipNonMonotone (PROP, predicate)

`Match(q, ·)` may lose members across evolution even though `dom(Σ.L)` only grows — the loss direction of (M-mut). The windowing-specific consequence is what this does *across calls*: a link delivered in window `i` may be absent from the recomputed matching set at a later window `j > i`, having been orphaned between the two calls (M-mut). Windowed completeness (W4) is therefore *relative to a fixed state*; across mutation, a link the reader already saw may no longer be among the matchers, and the total it is paging toward may shrink.

## W8 — CursorSurvivesUnderComputableKey (LEMMA, lemma)

Resumption past a cursor `c` is well-defined whenever `κ(c)` is *computable* against the current set, *even if `c` itself has left `Match`*. `After(c, Σ')` is defined by `κ(c)` alone — it reads the held cursor value against the current matching set, requiring only that `κ(c)` remain computable, not that `c ∈ Match(q, Σ')`. The load-bearing property is **computability** of `κ(c)` — *not* value-invariance and *not* state-stability of comparisons.

- The **address-based key** secures it unconditionally and *definitionally*: `κ(c) = c` is the identity applied to a value already in the reader's hand (value-totality), so the cut-point is recoverable in every post-state with no state lookup at all.
- Gregory's **least-covered-tumbler key** secures it by a *different* route: the key is permanent, so even after orphaning removes the cursor's content from the consulted arrangement the endset endures and `κ(c)` stays *computable*.
- The **content-position key** fails: `κ(c)` becomes uncomputable when the V→I mapping it was drawn from is gone; the successor set then collapses and the call returns the empty window — *indistinguishable from genuine exhaustion* (W9).

## W9 — ExhaustionByShortWindow (PROP, predicate)

A window returning fewer than `N` links is the terminal signal — but it certifies two *different* things under two different conditions.

*The local fact (cardinality).* Whenever `κ(c)` is merely **computable** against the current set — so `After(c, Σ)` is a well-defined cut — a short window empties the successor set at the next cursor:

> `κ(c) computable ⟹ ( |Window(q, c, N, Σ)| < N ⟹ After(next-cursor, Σ) = ∅ )`

This is pure counting at the single state `Σ`; *no cut-point preservation is consulted*.

*The global guarantee (everything delivered).* That every continuously-matching tail matcher is in fact delivered, none skipped, is **not** a terminal-state fact: it is exactly W5's coherence (the *no skip* half) read along the visited-cursor sequence the pass walks. It is **not** secured by computability at `c` alone, but by clause 1 (cut-point preservation) at *every* visited cursor.

## W9a — FixedSetTermination (LEMMA, lemma)

Against a fixed matching set of size `m`, and *for the constant schedule `N_i = N`* (a fixed window size held across every call), the paging loop terminates in exactly

> `⌈m / N⌉ + [N divides m]` calls

the bound function `m - iN` decreasing by `N` each non-final call until it falls below `N`. (Under a variable schedule — which W4 also covers — termination still holds by W4's cumulative-cut-point argument, but the closed-form count above is specific to the fixed-`N` case; a variable schedule terminates in a call count that depends on the schedule.)

## W9b — CumulativeInflowSufficiency (LEMMA, lemma)

Over a *mutating* matching set the quantity that governs termination is not the *instantaneous* size of the reachable tail but its *cumulative inflow*, and that inflow has a base term and an event term. The **base** is the **initial tail**: the links reachable ahead of the head cursor `⊥` at the first call, finite by M-fin. A **tail-inflow event** is a `(link, transition)` pair in which that later transition places that link into the reachable tail ahead of the then-current cursor. The total inflow is `|initial tail| + |tail-inflow events|`.

Three sources contribute:
- (1) the initial-tail base
- (2) fresh matching links created ahead of the cursor (D-NONMONO's `K.λ` case)
- (3) *links that become discoverable ahead of a cursor by the extension-monotonicity of projection (LP9 of ASN-0098) — an arrangement extension drawing matched content into a consulted region — whether or not the link was ever previously a member* (subsuming both a previously-orphaned link resurrecting (LP18; W7) and a born ghost (L4/L9 of ASN-0043) first entering the view)

The loop terminates whenever:
- (i) the cursor's cut-point is preserved at each successive cursor — W5's clause 1, applied at every cursor the pass visits
- (i′) each visited cursor's key stays *computable* across that cursor's transition (W8), so clause 1 is well-defined and applicable at every visited cursor
- (ii) the total inflow `|initial tail| + |tail-inflow events|` is finite (equivalently, the initial tail is finite and inflow events eventually cease)

Conditions (i) and (i′) are distinct: clause 1 alone does not entail (i′), since clause 1 at a cursor presupposes that cursor's key is evaluable at *both* states of its transition.

Charge bound: each delivery charges to its most recent inflow contribution; distinct deliveries of one link have distinct most-recent contributions; so the total deliveries `≤ |initial tail| + |tail-inflow events|`.

## W9c — CutPointNecessity (LEMMA, lemma)

Cut-point preservation is load-bearing and not removable: *absent it, even zero inflow does not guarantee termination*.

Witness: with the mutable content-position (V-position) key, two links `a, b`, `N = 1`, zero inflow:
- Call 1 (`κ(a) = 1, κ(b) = 2`) delivers `a`, cursor `→ a`
- Call 2 delivers `b` (`After(a) = {b}`), cursor `→ b` (`κ = 2`)
- A rearrangement (`K.μ~`) moving `a`'s matched content to V-position `3`, `κ(a) = 3`, makes `After(b) = {a}`, so Call 3 re-delivers `a`, cursor `→ a` (`κ = 3`)
- A further rearrangement to `κ(b) = 4` makes `After(a) = {b}`, re-delivering `b`, and so on forever — every window full (`size 1 = N`), no short window ever appears, with *no* tail inflow at all.

Each rearrangement is a cut-point (clause-1) violation at the held cursor: with cursor at `b` (`κ = 2`) and `a` already delivered below it (`κ = 1`), the rearrangement `κ(a) → 3` lifts the previously-delivered `a` above the held cursor — `κ(c) <_K κ(a)` flips from false to true — exactly the discrimination W5's first clause forbids.

## W9d — Clause2NonNecessity (LEMMA, lemma)

By contrast, W5's *clause 2* — preservation of the relative `≺`-order *among the undelivered tail links* — is *not* required for termination. Were clause 1 to hold at every successive cursor while clause 2 failed (the unseen tail permuting freely between calls), every delivered link would still stay permanently below every later cursor and so never be re-delivered; under finite inflow the consumable supply remains finite and the loop still terminates.

Full state-stability W5 is thus *sufficient* for termination — it implies clause 1 at every cursor — but strictly stronger than termination needs; clause-2 tail-order preservation is sufficient-via-full-W5, not necessary.

## W10 — CursorCarriesNoAbsoluteProgress (PROP, frame)

The cursor exposes only the resume key — enough to request the next window — and reveals neither the rank of the cursor within `Match(q, Σ)` nor the cardinality `|Match(q, Σ)|`. The window reply is a batch of `≤ N` links and no more; it carries no "position `k` of `m`" field. Absolute progress is therefore *not derivable from the windowed protocol alone*. A reader who wants "`k` of `m`" must obtain `m` from a separate cardinality query — a distinct operation, out of scope here — and tally `k` itself by counting delivered links; and even then, because each windowed call re-derives the matching set (W3) and that set may move (W7), the separately-obtained `m` is only a snapshot, not a guarantee about what the next window will find.

## W11 — BoundaryObjectivity (COROLLARY, corollary)

*Corollary of W3.* `Window(q, c, N, Σ)` is a deterministic function of its arguments alone (W3), so the boundary it fixes is observed identically by every reader: any two issuing the same `(q, c, N)` against the same `Σ` resume from the identical *next cursor* — the `≺`-max of the batch W3 already determines. The cut is therefore a *system* property — fixed by the enumeration order and the window size — not a reader-side choice. A reader may freely vary only `N` and how it *displays* the results; *where* the cut falls, for a given `N`, is objective.
