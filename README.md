# token-radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Man899/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Man899/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Token Radar ranks every pre-token protocol in a sealed DefiLlama snapshot by what its launch would be worth, and shows how to farm each one. It covers all 40 watchlist protocols plus 158 early pre-token protocols below the $50M floor — 198 rows in all — priced by category comparables, not by a flat TVL multiple.

## How it works

`main.ts` (Node 22, standard library only) reads `input/snapshot.json`, scores every rankable pre-token protocol with the radar-2 model in `lib/model.ts`, and writes `board.json`, the dashboard's `data.js`, and `dossier.json` (via `vendor/makefun-board.ts`).

**The model (radar-2).** Implied value is the entry's own claim, never a price and never a prediction that a token is issued:

```
implied = round(tvl_usd x comp(category) x growth x maturity x breadth)
```

- `comp(category)` — the median market-cap / TVL ratio of the live-token protocols in the same category (at least 3 peers, else the global median), clamped to [0.05, 1.00]. A lending protocol is priced like lending protocols that already trade.
- `growth` — `1 + clamp(change_7d, -60, 120) / 100 x 0.5`.
- `maturity` — `0.70 + min(age_days, 1095) / 1095 x 0.30`; age is the time since DefiLlama first listed the protocol, i.e. how long it has run with no token on record.
- `breadth` — `1 + min(chains - 1, 4) x 0.02`: TVL spread over more chains is stickier.

A protocol is rankable when its record carries no token (`mcap_usd`, `gecko_id` and `symbol` all null) and it is not a wrapped, staked or bridged product of a token that already trades (no Bridge / Liquid Staking / Restaking / CEX category, no whole word wrapped / staked / staking in the name, no version marker). Live-token protocols are never ranked; they are shown with their market caps in the Live tab and serve as the comparables that price the board.

**The dashboard** (`dashboard/`, static, no network) shows a top-10 spotlight, a sortable and filterable board (search, category, chain, watchlist-only), a detail view for every protocol with the model's step-by-step breakdown, its live-token comparables and a six-step farm plan with costs and times, plus the full methodology and category multiples. "No token" means no token on record — not proof there is none.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

Then open `out/dashboard/index.html` in a browser (it reads `out/data.js` beside it). Node 22.18 or later is required; no dependencies are installed.

## Layout

- `main.ts` — entrypoint: snapshot in, `board.json` + `data.js` + `dashboard/` + `screenshots/` + `dossier.json` out.
- `lib/model.ts` — the radar-2 model, the farm plans and the board checks.
- `vendor/makefun-board.ts` — make.fun's dossier library, as shipped in the kit.
- `dashboard/` — the static page (`index.html`, `app.css`, `app.js`).
- `screenshots/` — the board, a protocol's detail view and a filtered view.

---

Entered on [make.fun](https://make.fun/?ref=5e0gj8t6&utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards
