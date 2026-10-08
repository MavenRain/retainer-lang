# retainer-lang specification (draft)

Status: draft before milestone M0. `retainer-lang` is a working name.

Fill each section. The quoted line under each heading says what to write
there. Delete the quoted line when the section is complete.

Slice RL1 (2026-10-07) wrote sections 1 to 7, 9 and 10. Section 8 waits for
slice RL6. A tag C5 to C13 marks a design call of the build brief. No design
call is ruled. A quotation in double quotes is a verbatim quotation of
`design/DESIGN.md` (the design). A design section is named by its heading,
for example `## Books`.

## 1. Purpose

retainer-lang is a language for a retainer that a self-referential DAO
governs. One program gives the members, the decisions, the policy of each
decision and the choice rule of the DAO. The compiler checks the program and
writes one deployable EVM contract. The contract holds many engagements under
one DAO policy. The host is TinyCC. The target is EVM bytecode.

In the design, the retainer "is a time-indexed measure acted on by the
aggregation already fixed in the self-referential DAO" (the design, first
paragraph). Its type formers are F1 to F15 of `formers/FORMERS.md`. Its core
data types and core operations are only the types and operations of
`design/DESIGN.md` (the design).

## 2. Programs

A program is a sequence of definitions:

```
def NAME : TYPE := TERM
```

The first two definitions are fixed (host-forced, rule R1, O8):

```
def members : Nat := N
def decisions : Nat := K
```

`N` and `K` are positive integer literals. `members` sets the number of
members. `decisions` sets the size of the decision space `D`. The compiler
refuses a program that does not start with these two definitions
(`REFUSE_MEMBERS`, `REFUSE_DECISIONS`). Then the program gives
`policyOf` and `choice` (section 6).

The compiler refuses these host forms in a program:

| Code | Form |
|---|---|
| `REFUSE_DATA` | A family declaration. Only `domain/domain.lang` declares a family (rule R1). |
| `REFUSE_AXIOM` | An axiom (rule R2). |
| `REFUSE_REC` | `def rec`, and a use of a definition before its own definition (rule R3). |
| `REFUSE_NAME` | A definition that uses a core name or a prelude name (rule R4). |

It also refuses these domain forms:

| Code | Form | Source |
|---|---|---|
| `REFUSE_HISTORY_WRITE` | An entry that writes a `history` field other than by addition. | Kit rule C4; the design, `## Self-reference`: "A representation that rewrites recognized revenue below \(t^\*\) is rejected." |
| `REFUSE_BIFURCATION` | A self-constitution that is not single-valued (Schelling-Ising). | C10; the design, `## Admitted operations`: "What this refuses: ... a bifurcating self-constitution for a constant fee". |

`choice` reads the tally only, so the constitution is constant and each tally
has at most one decision. Thus no program in this form reaches
`REFUSE_BIFURCATION`. The code stays as the result of the regime check
(section 6, `regime`) for a future non-constant constitution.

A program cannot add a data type, an unproved fact or general recursion.
Recursion comes only from `fold` and `unfold` (F6, F7). The compiler writes
the target; a program cannot.

## 3. Type formers

The type formers are F1 to F15 of `formers/FORMERS.md`. This language uses
the tcc-evm-contract column of the realization matrix. `formers/tcc-evm-contract.md` gives
the host form of each former.

| ID | Status on tcc-evm-contract | Effect on this language | Open item |
|---|---|---|---|
| F1 to F15 | `langc check` and `langc eval`: DONE (copy of the tcc-wasm front end). EVM output: PLANNED (kit slice K4). | A program checks and evaluates now. `langc build` stops with `langc: PLANNED` until K4. | None. Kit slice K4. |

## 4. Structures

- **Monad**: `pure`, `map`, `bind` on `Option`. `choice` returns
  `Option D` ("Partiality of governance is `Option`-valued", `## Stance`). An
  entry returns `none` to revert (C2).
- **Algebra**: `fold` on `List`: the segments and the masses of a `Rate`
  (C6), the entries of the policy history (C12), the ballots (to make a
  tally). No `unfold`: the design has no unbounded structure to build.
- **Filterable**: `filter` on `List`: the policy history entries that overlap
  an accrual interval (C12).

The kit gives each carrier. None is missing.

## 5. Core types

Each definition is in host forms. `U256` and `Addr` are kit types of slice K2
(PLANNED). `Nat` is the 64-bit host `Nat`.

| Type | Meaning (design section) | Definition |
|---|---|---|
| `Time` | "\(\mathrm{Time} = \mathbb{R}_{\ge 0}\)". "Continuous time is the denotation; block time is a sampling." (`## Semantic domain`) | `Nat`, seconds of block time. RULED 2026-10-07 (USER): Time stays a 64-bit Nat. (C5) |
| `Amount` | "\(\mathrm{Amount} = (A, 0, +)\) with \(A\) a cancellative commutative monoid, so the residual is well-defined." (`## Semantic domain`) | `U256`. Each subtraction is a residual `a - b` with `b <= a` (section 6). RULED 2026-10-07 (USER): Amount = uint256; the asset is one ERC-20 token for each engagement. (C5) |
| `Rate` | The density in the integral of `earned`. "The rate is a Dirac comb." "Locally constant rate." "Cliff and step-unlock are piecewise rates, not new denotations." (`## One measure, three instruments`) | A record of two lists. `segments : List Segment`, with `Segment = (from : Nat, until : Nat, num : U256, den : Nat)`, `from <= until`, `den` not 0. `masses : List Mass`, with `Mass = (at : Nat, amount : U256)`. Each offset counts seconds from the engagement start. (C6) |
| `Window` | Part of `Policy` and part of an engagement (`## Semantic domain`). The integral runs from \(\mathrm{start}(E)\) to \(\mathrm{end}(E)\). | `(start : Nat, end : Nat)` with `start <= end`. The engagement window is in block time. The policy window is in block time too (O2). (C7) |
| `Auth` | Gives the indicator \(\mathbf{1}_{\mathrm{Auth}(E,s)}\) in `earned`. "The pull chooses the actor. It does not change the obligation." (`## One measure, three instruments`) | `(instrument : Instrument, paused : Flag)`, with `Instrument = push | pull | stream` ("Settle, whether push, pull, or stream withdraw", `## Admitted operations`). `paused` is the policy that withdraws the authorization. (C8) |
| `Dispute` | "A dispute map acts on the residual only." (`## Books`) "Dispute applies \(\mathrm{Dispute}(v)\) to the residual only." (`## Admitted operations`) | `List Nat`: the firm share of the residual in basis points (at most 10000) for each verdict `v`. A verdict is a `Nat` position in the list. A verdict outside the list reverts. (C9) |
| `Policy` | "\(\mathrm{Policy} = \mathrm{Rate} \times \mathrm{Window} \times \mathrm{Auth} \times \mathrm{Dispute}\)". "Governance decides a policy, not a balance". (`## Semantic domain`) | `(rate : Rate, window : Window, auth : Auth, dispute : Dispute)`. |
| `D` | The decision space of \(\mathrm{policyOf} : D \to \mathrm{Policy}\) (`## Semantic domain`). | `Nat` below `decisions` (host-forced, rule R1, O8). (C10) |
| `Ballot` | \(\mathrm{ballotAt}\, s\) in `rateAt` (`## One measure, three instruments`). | `D`, one for each member. (C10) |
| `Tally` | "Anonymity is the orbit quotient" (`## Stance`): the orbit of a ballot profile under member relabeling. | `List Nat` of length `decisions`: the count of ballots for each `D`. The counts add to `members`. (C10) |
| `ChoiceRule` | "\(F : \mathrm{ChoiceRule}\,\mathrm{Obj}\, D\)" (`## Semantic domain`). | `Tally -> Option D`, the program definition `choice`. Then \(F\) = `choice` after the orbit projection, so \(F\) is anonymous. (C10) |
| `Engagement` | "the signed scope the rate is earned against: client, firm, `scopeHash` of the engagement letter, window, policy." (`## Semantic domain`) | `(client : Addr, firm : Addr, token : Addr, scopeHash : U256, window : Window)` and the state `countersignedAt : Option Nat`, `cancelledAt : Option Nat`. The policy is the one DAO policy (C11). `token` is the ruled ERC-20 (RULED 2026-10-07, USER). (C11) |
| `Book` | \(\mathrm{Book}\, L\) "reads rate, window, authorization, and dispute off \(\mathrm{Gov}\, L\)". "Let \(\mathrm{funded}\) be cash in custody and \(\mathrm{withdrawn}\) be what the firm has taken." (`## Semantic domain`, `## Books`) | For each engagement: `funded : U256`, `withdrawn : U256`, and the lazy checkpoint `history : U256`, `cursor : Nat` (C12). `funded` is the total cash paid into custody less the refunds. It is not the balance: the custody invariant names `balance` apart from `funded` (`## Books`). |

## 6. Core operations

The program gives `policyOf` and `choice`. The language gives the other
operations. `E` is an engagement, `t` and `s` are times, `now` is the block
time.

| Operation | Type | Meaning (design section) |
|---|---|---|
| `policyOf` | `D -> Policy` | "\(\mathrm{policyOf} : D \to \mathrm{Policy}\)" (`## Semantic domain`). A total program definition. (C10) |
| `choice` | `Tally -> Option D` | \(F\) and its aggregation \(L\) at a constant constitution (`## Stance`, `## Semantic domain`). A program definition. (C10) |
| `Gov` | `Tally -> Option D` | "\(\mathrm{Gov}\, L = \mathrm{orbitProjection}\,\mathrm{act} \ggg L.\mathrm{functor}\)" (`## Stance`). The compiler makes the table of `choice` over each tally. A second check compares each table entry with `choice` (the certificate). (C10) |
| `GovPi` | `Tally -> Option Policy` | "\(\mathrm{Gov}_\pi\, L = \mathrm{policyOf} \circ \mathrm{Gov}\, L\)" (`## Self-reference`). `map policyOf` after `Gov`. (C10) |
| `IsSelfConstitutingPolicy` | compile-time check | "\(\mathrm{IsSelfConstitutingPolicy}\, F \iff \exists\, L,\; \mathrm{Gov}_\pi\, L = \mathrm{policyOf} \circ F\)" (`## Self-reference`). At a constant constitution, `Gov` is `choice`, so the check holds for each tally where `choice` gives a decision. (C10) |
| `regime` | compile-time check | The three regimes (`## Self-reference`). Arrow-Debreu: `choice` gives a decision for each tally. Arrow-impossibility: some tally has none; then "Accrual freezes at the last self-constituting witness, or at the zero rate if there has never been one." Schelling-Ising: refused (`REFUSE_BIFURCATION`, section 2). (C10) |
| `amend` | a member (`Nat`) and a `D`; it changes the ballots and the policy history | "\(\llbracket\mathrm{amendPolicy}\rrbracket = \mathrm{policyOf} \circ \mathrm{Gov} = \mathrm{Gov}_\pi\)" (`## Self-reference`); "Amend is \(\mathrm{Gov}\), as before." (`## Admitted operations`). A member casts a new ballot. If the new tally gives a decision `d` that is not the last decision, the policy history gets one entry `(now, d)`. If it gives none, the history does not change. (C10, C12) |
| `rateAt` | `Engagement -> Time -> Rate` | "\(\mathrm{rateAt}(L, E, s) = \mathrm{Rate}\bigl(\mathrm{policyOf}\,((\mathrm{Gov}\, L).\mathrm{obj}\,(\mathrm{ballotAt}\, s))\bigr)(s)\)" (`## One measure, three instruments`). The rate of the policy history entry that is in force at `s`. (C6, C12) |
| `earned` | `Engagement -> Time -> Amount` | "\(\mathrm{earned}(L, E, t) = \int_{\mathrm{start}(E)}^{\min(t,\,\mathrm{end}(E))} \mathbf{1}_{\mathrm{Auth}(E,s)}\cdot\mathrm{rateAt}(L, E, s)\, ds\)". "Zero before start, nondecreasing in \(t\)" (`## One measure, three instruments`). Definition below. (C6, C7, C8, C12) |
| `recognized` | `Engagement -> Time -> Amount` | "\(\mathrm{recognized}(t) = \min(\mathrm{funded}(t), \mathrm{earned}(t))\)" (`## Books`). |
| `unearned` | `Engagement -> Time -> Amount` | "\(\mathrm{unearned}(t) = \mathrm{funded}(t) - \mathrm{recognized}(t)\)" (`## Books`). A residual: `recognized <= funded`. |
| `withdrawable` | `Engagement -> Time -> Amount` | "\(\mathrm{withdrawable}(t) = \mathrm{recognized}(t) - \mathrm{withdrawn}(t)\)" (`## Books`). A residual: "\(\mathrm{withdrawn} \le \mathrm{earned}\)" and `withdrawn <= funded`. For `pull`, see O1. |
| `fund` | `Engagement -> Amount -> Book` | "Fund is a monoid action on \(\mathrm{funded}\), identity on \(L\) and on earned history." (`## Admitted operations`) |
| `settle` | `Engagement -> Book` | "Settle, whether push, pull, or stream withdraw, adds \(\mathrm{withdrawable}(t)\) to \(\mathrm{withdrawn}\)." (`## Admitted operations`) |
| `cancel` | `Engagement -> Book` | "Cancel at \(t^\*\) clips the future integral and returns \(\mathrm{unearned}(t^\*)\) to the client. The firm keeps what is recognized." (`## Admitted operations`) |
| `dispute` | `Engagement -> Nat -> Book` (the verdict) | "Dispute applies \(\mathrm{Dispute}(v)\) to the residual only." (`## Admitted operations`) See O3. |
| `close` | `Engagement -> Time -> Amount` | "Period close at \(T\) emits one unearned-revenue event by evaluation, not by chase: debit unearned and credit revenue by \(\mathrm{recognized}(T) - \mathrm{recognized}(T^-)\)." (`## Books`) (C13) |
| `open`, `countersign` | `Engagement -> Book` | "An engagement is the signed scope" (`## Semantic domain`); "the engagement signature is the authorization" (`## One measure, three instruments`). The client opens, the firm countersigns (O5). (C11) |

### 6.1 The definition of `earned`

The policy history is a list of entries `(t_i, d_i)`, in order of `t_i`
(C12). Entry `i` is in force on the interval `(t_i, t_(i+1)]`. The last entry
is in force after `t_i`. Before the first entry, the rate is zero.

The accrual interval of entry `i` for `E` at time `t` is the intersection of:

- the interval where entry `i` is in force,
- the window of `E` and the window of `policyOf d_i` (O2, C7),
- the interval from `countersignedAt` to `cancelledAt` (C8, C11),
- the interval up to `t`.

Entry `i` adds zero if its interval is empty or if the `Auth` of
`policyOf d_i` has `paused` set (C8). Otherwise, for an interval `(a, b]`, it
adds `G(b - S) - G(a - S)`. `S` is the start of `E`. `G` is the cumulative
integral of the rate of `policyOf d_i`:

```
G(x) = sum over the segments of floor(num * (clamp(x, from, until) - from) / den)
     + sum of amount over the masses with at <= x
```

The first accrual interval of `E` includes its start when its policy,
windows and signature authorize accrual at `S`. This interval is `[S, b]`
and adds `G(b - S)`, without subtracting `G(0)`. The singleton `[S, S]`
is not empty. Thus a mass of 100 at offset 0 earns 100 at `S` and remains
100 afterwards; the open-interval formula would incorrectly subtract it.
Other intervals have an open lower endpoint, so an amendment or a later
signature cannot earn a past mass again.

`G` is always computed from the segment start, never added per call (C6).
Thus a closed linear stream of deposit `d` is one segment with `from = 0`,
`until = end - start`, `num = d` and `den = end - start` (for `end > start`),
and `earned(end) = d` exactly ("\(\mathrm{earned}(t) = \mathrm{clamp}(0, d, \mathrm{rate}\cdot(t-\mathrm{start}))\)", `## One measure, three instruments`).

`earned` is the sum over the entries. It satisfies the law of `## Admitted
operations`: "\(\mathrm{earned}(t_2) = \mathrm{earned}(t_1) + \mathrm{earned}(t_1, t_2)\)".
An amendment at `t*` adds an entry that is in force only after `t*`, so it
"contributes only on \((t^\*, \mathrm{end}]\)" (`## Self-reference`).

## 7. Target and instance encoding

The compiler writes one contract. Its creation code takes the address of each
member, in order. The contract has no other instance.

Storage (C11, C12). The back end (kit slice K4) gives the slot numbers in the
order below. A map uses a SHA3 slot.

| Data | Encoding |
|---|---|
| Member addresses | `members` words, set by the creation code, never written again |
| Ballots | One word for each member: the `D` of the member |
| Policy history | An append-only array of `(t_i, d_i)`: a length word and one word for each entry (C12) |
| Engagement counter | One word: the next engagement id (C11) |
| Engagements | A map from the id to the fields of `Engagement` and `Book` (section 5) |
| Funding log | For each engagement, an append-only array of `(time, funded)`, so that `recognized(T)` for `T < now` reads `funded(T)` (O9) |
| Close state | For each engagement, the last close time `T_prev` and `recognized(T_prev)` |

`now` is TIMESTAMP and the caller is CALLER (C2). An entry writes all state
first, then makes its token calls in order (checks, effects, interactions).
Each ERC-20 call checks the return data: a `true` word or empty data passes
(O6); any other result reverts (C2). A failed guard reverts.

Entries (C13). Each entry that changes `funded` or `withdrawn` first folds
`earned` up to `now` into `history` and `cursor` (the lazy checkpoint, C12).

| Entry | Guard | Effect |
|---|---|---|
| `open(firm, token, scopeHash, start, end)` | `start <= end` | New engagement, client = caller. Returns the id. |
| `countersign(id)` | caller = firm; not countersigned | `countersignedAt = now`. |
| `fund(id, amount)` | caller = client; not cancelled | `transferFrom(client, contract, amount)`; `funded += amount`; funding log entry. |
| `settle(id)` | caller = firm | `w = withdrawable(now)`. `push`, `stream`: `transfer(firm, w)`. `pull`: `transferFrom(client, firm, w)` (O1). `withdrawn += w`. |
| `cancel(id)` | caller = client or firm; not cancelled | `cancelledAt = now`; `u = unearned(now)`; `transfer(client, u)`; `funded -= u`; funding log entry. See O3. |
| `dispute(id, v)` | after cancel or after the end (O3) | Split the residual: the firm gets `share(v)` basis points, the client gets the rest. |
| `cast(d)` | caller is a member; `d < decisions` | `amend` (section 6). |
| `close(id, T)` | `T <= now`; `T > T_prev` | Emit `UnearnedRevenue(id, recognized(T) - recognized(T_prev))`; set `T_prev = T`. |

Views (C2, C13): `earned(id)`, `recognized(id)`, `unearned(id)`,
`withdrawable(id)`, each at `now`.

The contract has no proof term. The checker erases each proof before the
lowering. The `Gov` table and its certificate are compile-time only; the
contract holds the table, not the check.

## 8. Host and target

> Write here: the host, its pinned commit (`PIN`), the facts from
> `probe/CAPABILITY.md` that set the design, and the check that the compiler
> runs on its output (for example the axiom report).

Known now (`docs/host/CAPABILITY.md`, slice K1): tcc 0.9.28rc 2026-09-04
mob@0fb54300 builds the compiler. geth `evm` 1.14.12 runs the tests. The K4
test path is a chain of `evm run --prestate ... --dump` calls. Slice RL6
fills `PIN` and `probe/CAPABILITY.md`.

## 9. Open items

- O1. Pull books. In `pull`, the cash is not in custody. Default: `settle`
  pulls `withdrawable = earned - withdrawn` with `transferFrom(client, firm)`
  and adds the same amount to `funded` and to `withdrawn` in one step. Then
  the pull books are equal to the push books. Not ruled.
- O2. The design gives a window to the engagement and to the policy.
  Default: the accrual uses the intersection of the two windows (C7), and the
  policy window is in block time. Not ruled.
- O3. Dispute timing. Default: only after cancel or after the end, on
  `unearned` only. Two gaps, found in RL1: (a) `cancel` pays `unearned` to the
  client at once, so after a cancel the residual is zero and a dispute has no
  effect; (b) the design does not say who gives the verdict `v`. Not ruled.
- O4. Gas grows with the number of policy history entries between two
  touches of an engagement (C12). Default: no cap; the CAPABILITY file reports
  the gas. Not ruled.
- O5. Engagement signature. Default: two transactions (the client opens, the
  firm countersigns). Alternative: EIP-712 signatures, checked with
  `ecrecover` in one transaction. Not ruled.
- O6. Non-standard ERC-20 tokens. Default: accept empty return data and
  refuse nothing else. A fee-on-transfer token breaks the custody invariant
  and is out of scope. Not ruled.
- O7. The full note `DENOTATIONAL-DESIGN-RETAINER.md` is not on disk.
  `design/DESIGN.md` holds the pasted part, verbatim. It is the specification
  until the USER gives the full note. Not ruled.
- O8. Host-forced (rule R1), found in RL1. C10 says that the program declares
  `D` as an enum. A program cannot declare a family. Default: the fixed first
  definitions `members` and `decisions`, and `D` = `Nat` below `decisions`, as
  escrow-lang does for `members`. Not ruled.
- O9. Found in RL1. `close(id, T)` with `T < now` needs `funded(T)`. The
  `Book` holds only the current `funded`. Default: a funding log for each
  engagement (section 7). Alternative: `close` only at `T = now`. Not ruled.

## 10. Milestones

| Milestone | Content |
|---|---|
| M0 | Host probe (`probe/CAPABILITY.md`); the core types construct and check; refusal list; examples and tests |
| M1 | Core operations |
| M2 | Queries and reads, or the contract |
| M3 | Hardening: speed, limits, a checked certificate for the output |

The build brief maps its slices onto the milestones: RL2 (types and the four
measures) and RL3 (governance) are M0 and M1. RL4 (entries and views) and
RL5 (examples, the differential test on geth) are M2. The kit slices K2 to K4
come first.

Status 2026-10-07: M0 not started. RL1 wrote this file. The kit has K1 only.
