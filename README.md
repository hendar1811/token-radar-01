# Token Radar — radar-2

A token launch radar for the make.fun **token-radar-01** bounty. It reads a
sealed snapshot of the DefiLlama protocol universe and writes a web dashboard
that ranks the protocols that record no token by what their launch would be
worth, each with the steps to farm it.

## What it does

`main.ts` (Node 22, standard library only) reads `input/snapshot.json` and
writes to the output directory:

- `board.json` — the ranked board (197 rows on this snapshot)
- `data.js` — the same data for the dashboard page
- `dashboard/` — the static page (no network, no framework)
- `screenshots/` — three screenshots of the page
- `dossier.json` — the judge's dossier, built with `vendor/makefun-board.ts`

## The model (radar-2)

The board ranks **every genuine pre-token protocol** in the snapshot — the
watchlist is the floor, not the ceiling, and no arbitrary top-N cut is
applied. A protocol is genuine pre-token when DefiLlama records no token for
it and it is not a wrapped, staked or bridged product of a token that already
trades (the house's own watchlist rule, applied to the whole snapshot). That
is why WBTC, Binance staked ETH and the bridges are absent: their tokens
already trade. Live-token protocols are set aside and shown with their market
caps.

Implied value is a comparables valuation, computed from the snapshot itself:

```
implied = round( tvl_usd × multiple × growth × maturity × breadth )
```

- **multiple** — the median market-cap / TVL of the live-token protocols in
  the same category (at least 3 comparables), clamped to ×0.05–×0.80; a
  category with too few live comparables uses the snapshot-wide median.
  Exchanges and wrapped, staked or bridged products are never comparables.
- **growth** — `1 + clamp(change_7d, -50, 150) / 100 × 0.6`
- **maturity** — `0.75 + min(age_days, 730) / 730 × 0.25` (unknown age: 365 days)
- **breadth** — `1 + 0.03 × min(chains − 1, 5)`

No protocol name is ever special-cased. The dashboard page recomputes every
row with the same function and checks it against `board.json`, and a model
sandbox lets the reader move the growth weight and the multiple cap and watch
the board recompute live.

Each row carries a seven-step farm plan derived from the protocol's own
record (category, chains, TVL, age): the action, the capital, the hold
period, and the gas budget for the chain it runs on.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input input/snapshot.json
```

Then open `out/dashboard/index.html` in a browser. The page reads
`out/data.js` beside it and needs no network.

## Files

- `main.ts` — the entry program
- `lib/model.ts` — the model, the board builder, the farm plans
- `vendor/makefun-board.ts` — make.fun's dossier library, copied from the kit
- `dashboard/` — `index.html`, `app.css`, `app.js`
- `screenshots/` — the board, a protocol detail, and a filtered view
