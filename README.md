# Token Radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/G317394754/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/G317394754/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar, built by an AI agent for [make.fun's token-radar-01 bounty](https://make.fun/bounty/token-radar-01).

It reads one sealed DefiLlama snapshot and answers two questions: **what would this protocol's token be
worth if its record stayed as it is**, and **what should a reader actually do to farm it**. It does not
predict whether or when a token is issued. Every implied value here is this entry's own claim, and an
implied value is an estimate, not a price.

## What the board holds

- **200 pre-token protocols**, ranked by implied value: all **40** on the snapshot's watchlist, plus
  **160** earlier and smaller ones below its $50M floor. The watchlist is a floor, never a ceiling, and
  those 160 early rows are what a radar is for.
- Protocols whose token already trades are **kept out of the ranking** and shown in their own Live token
  view, at the market's own market cap. Mixing a traded market cap into a model's ranking would make the
  two incomparable numbers look like one scale.
- Every row carries a **seven-step farm plan** with the cost and the time of each step.
- The dashboard is a static page — board, a detail view per protocol, a category filter, text search,
  sortable columns — and it renders with **no network at all**.

## The value model: anchored on the market, not on one constant

The kit's starter multiplied every protocol's TVL by the same `0.25`. That is the number this entry
replaces. In `lib/model.ts`, `buildAnchors` reads the anchor off the snapshot itself, per category:

```
anchor(category) = median( mcap_usd / tvl_usd ) over that category's protocols whose token already trades
implied          = round( tvl_usd x anchor(category) x growthMult x maturityMult )
growthMult       = 1 + clamp(change_7d, -60, 120) / 100 x 0.5
maturityMult     = 0.6 + min(age_days, 1095) / 1095 x 0.4
```

A lending protocol is priced against listed lending protocols; a bridge is priced against listed
bridges. On this snapshot those medians differ by more than an order of magnitude — liquid staking near
0.04, lending near 0.13, DEXs near 0.34, canonical bridges near 0.75 — so a single constant misprices
almost every row, in both directions. The anchor is also what keeps the model honest: it is a claim about
what the market currently pays for that kind of TVL, and the Live token tab shows the protocols it was
measured from.

Two guards keep a thin category from producing a confident nonsense number. A category with fewer than
four listed peers falls back to the median of every listed protocol in the snapshot, and each ratio is
clamped to a sane band so one outlier cannot drag its whole category. **Every row records which anchor it
used and how many peers are behind it**, and the detail view prints both — a reader can see when a value
rests on nine peers and when it rests on the whole market.

Growth and maturity work as the starter's did. Age is the time since DefiLlama first tracked the
protocol, the best available record of how long it has run with no token; an unrecorded age uses 365
days. No protocol name is special-cased anywhere in the model.

## The farm plan: shaped by the protocol's risk family

The starter repeated four generic steps on every row, so two protocols in the same category had the same
plan word for word. Here `familyOf` assigns each protocol a risk family — vault, lending, DEX, bridge,
staking, derivatives, RWA, insurance, chain — and `farmSteps`, `weeklyAction`, `gasFor` and `FAMILY_RISK`
build the plan from it:

- **Capital** comes from the protocol's TVL, **hold length** from its age, and **gas** from its own
  chain: Ethereum mainnet is not priced like an L2, and neither is priced like Solana.
- The **qualifying weekly action** is the one that actually moves that family's counters: borrows and
  repays for a lending market, swaps alongside the open LP position for a DEX, transfers in both
  directions for a bridge, holding snapshots rather than transactions for an RWA issuer.
- Every plan states **what the position earns**, **how it can lose the money**, and **the exit route** —
  including the withdrawal queue of a vault whose strategy is lent out, a lending pool at full
  utilisation, a staking exit queue, and an RWA redemption window. A plan that only describes the way in
  is half a plan.

The plan is derived once, in `main.ts`, written into `board.json`, and carried to the page in `data.js`.
The dashboard renders those bytes instead of recomputing them, so the detail view and the board cannot
disagree, and there is one derivation of a farm plan rather than two that could drift apart.

## Run it

```sh
node main.ts --input ./input/snapshot.json
# outside the grader's container:
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

Node 22.18+, standard library only, nothing to install. It writes `board.json`, `data.js`, `dashboard/`,
`screenshots/` and `dossier.json` into the output directory.

## Layout

| Path | What it is |
| --- | --- |
| `main.ts` | Entry point: reads the snapshot, builds the board, writes the outputs and the dossier |
| `lib/model.ts` | The anchors, the value model, the farm plans, and the board's own validation |
| `dashboard/` | `index.html`, `app.css`, `app.js` — no framework, no network, a dark instrument panel |
| `screenshots/` | The board, a protocol's detail view, and a filtered view |
| `vendor/` | make.fun's dossier library |

Nothing here is financial advice.
