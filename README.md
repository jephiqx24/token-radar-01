# Token Radar — the best token launch radar

[![make.fun bounty](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/jephiqx24/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/jephiqx24/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01)

A radar for protocol token launches: it reads a sealed snapshot of every protocol
DefiLlama tracks, finds the ones with no token yet, ranks them by what their launch
would be worth, and shows a reader how to farm each one.

## The model (radar-2)

`implied = TVL x fitted category multiple x growth x maturity`

The multiple is fitted at run time from the snapshot's own live-token protocols,
per category: the median market-cap/TVL ratio, shrunk toward the global median
(0.2138x) and capped at 2.0x. Growth is the seven-day TVL change clamped to
-60..+120%, maturity is time since first listing. One function scores every row;
no protocol name is special-cased.

Out-of-sample check (5-fold cross-validation over the snapshot's 128 live-token
protocols): median absolute error 0.93, 20% of predictions within 2x of the true
market cap, Spearman rank correlation 0.40. A fixed 0.25 multiple scores 0.89 /
30% / 0.43 on the same folds — TVL alone predicts a market cap only loosely, so
this board is a screen for where to look, not a price.

## The board

- 178 rows: the full 40-protocol watchlist (every protocol with $50M+ TVL and no
  token) plus 138 pre-token protocols below the floor that meet the watchlist's
  own bar (no wrapped, staked or bridged product, no exchange, no versioned pool).
- Every row carries an implied value and an 8-step farm plan with costs and times.
- The 24 largest live-token protocols are set aside with their market caps: they
  are facts, not a model, and they are what the multiples are fitted from.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Node 22 or later, standard library only, no network. Two runs write byte-identical
output. See `BUILD_NOTES.md` for the model, the validation, and the verification.
