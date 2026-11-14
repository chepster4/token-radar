# Token Radar — model radar-2

A token launch radar: it reads a frozen DefiLlama protocol snapshot, ranks every
pre-token protocol by an implied value, and shows a reader how to farm each one.

## The model

Implied value is calibrated on the market itself, not a flat multiple. For each
category, the peer multiple is the median market-cap-to-TVL ratio of the live-token
protocols in that category, shrunk toward the global median with a prior of 5 peers
and clamped to 0.05–1.0x. A category with no listed peers falls back to the global
median. Each live ratio is winsorized at 2.0x first, so one outlier cannot set a
category's price.

    implied = round(tvl_usd x peer_multiple(category)
                    x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                    x (0.6 + min(age_days, 1095) / 1095 x 0.4))

The board covers the whole watchlist (TVL ≥ $50M, no token on record) and adds the
early radar: genuine launch candidates below the floor, by TVL. Wrapped, staked and
bridged products are skipped — they are not launches of their own. Live-token
protocols are set aside with their market caps shown, and they are the peers the
model prices the board against. Every protocol gets a seven-step farm plan with a
cost and a time for each step, derived from its category, chains, TVL and age.

## Run it

    mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json

Writes `board.json`, `data.js`, `dashboard/`, `screenshots/` and `dossier.json`
into the output directory. Node 22, standard library only, no network, deterministic:
two runs write the same bytes.

## Layout

- `main.ts` — entrypoint: read the snapshot, score it, write the outputs.
- `lib/model.ts` — the model: calibration, ranking, farm plans, board checks.
- `vendor/makefun-board.ts` — make.fun's dossier library (vendored from the kit).
- `dashboard/` — the page: `index.html`, `app.css`, `app.js`. Static, no network.
- `screenshots/` — board, detail, filtered and live views of the page.
