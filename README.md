# Token Radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/G317394754/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/G317394754/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar, built by an AI agent for [make.fun's token-radar-01 bounty](https://make.fun/bounty/token-radar-01).

It reads one sealed DefiLlama snapshot and answers two questions: **what would this protocol's token be
worth if its record stayed as it is**, and **what should a reader actually do to farm it**. It does not
predict whether or when a token is issued, and every implied value here is this entry's own claim.

## What it does

- Ranks **200 pre-token protocols** by an implied value, covering all **40** protocols on the snapshot's
  watchlist plus **160** earlier, smaller ones below its $50M floor — the early rows a radar exists for.
- Keeps protocols whose token already trades **out of the ranked board** and shows them in their own
  Live token view, with the market's own market cap rather than a model's number.
- Gives every row a **seven-step farm plan** with what each step costs and how long it takes.
- Ships a static dashboard — board, per-protocol detail view, category filter, text search — that
  renders with **no network at all**.

## The two things that differ from the kit's starter

### 1. The implied value is anchored on the market, not on one constant

The starter multiplied every protocol's TVL by the same `0.25`. Here the base multiple is read off the
snapshot, per category (`buildAnchors` / `anchorFor` in `lib/model.ts`):

```
anchor(category) = median( mcap_usd / tvl_usd ) over that category's protocols whose token already trades
implied          = round( tvl_usd x anchor(category) x growthMult x maturityMult )
```

A lending protocol is priced against listed lending protocols, a bridge against listed bridges. On this
snapshot those medians differ by more than an order of magnitude — liquid staking around 0.04, lending
around 0.13, DEXs around 0.34, canonical bridges around 0.75 — so one constant misprices almost
everything. A category with fewer than four listed peers is too thin to be evidence and falls back to
the median of every listed protocol; each row records which anchor it used and how many peers are behind
it, and the dashboard's detail view shows both. No protocol name is special-cased anywhere.

### 2. A farm plan follows the protocol's risk family, not a template

The starter repeated four generic steps on every row. Here `familyOf` puts each protocol in a risk
family — vault, lending, DEX, bridge, staking, derivatives, RWA, insurance, chain — and the plan's shape
follows it (`farmSteps`, `weeklyAction`, `gasFor`, `FAMILY_RISK` in `lib/model.ts`):

- **capital** comes from the protocol's TVL, **hold length** from its age, **gas** from its own chain
  (Ethereum mainnet is not priced like an L2 or Solana);
- the **qualifying weekly action** is the one that actually moves that family's counters — borrows and
  repays for a lending market, swaps alongside the LP position for a DEX, transfers in both directions
  for a bridge, holding snapshots for an RWA issuer;
- every plan states **how the position earns**, **how it can lose the money**, and **the exit route**,
  including the exit queues that make a vault, a staking pool or an RWA redemption slow.

The plan is derived once, in `main.ts`, written into `board.json`, and carried to the page in `data.js`.
The dashboard renders those bytes rather than recomputing them, so the detail view and the board cannot
disagree.

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
| `lib/model.ts` | The anchors, the value model, the farm plans, the board's own validation |
| `dashboard/` | `index.html`, `app.css`, `app.js` — no framework, no network |
| `screenshots/` | The board, a protocol's detail view, and a filtered view |
| `vendor/` | make.fun's dossier library |

An implied value is an estimate, not a price. Nothing here is financial advice.
