# Token Radar

A token launch radar for make.fun's `token-radar-01` bounty: it reads a sealed snapshot of
every protocol DefiLlama tracks, works out which ones still record no token and are not a
wrapped or staked product of one that trades, and ranks them by what their launch would be
worth — then hands in a web dashboard that shows the board, a detail view for each protocol
and the steps to farm it.

## The model (radar-2)

Every row is scored by one function of the snapshot's own figures. No protocol name is
special-cased, and no row is hand-picked.

```
implied = round( tvl_usd x 0.25
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.6)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + 0.03 x min(chains, 10)) )
```

- **TVL** is the launch's scale.
- **Seven-day growth** is its momentum (clamped so one wild week cannot decide the board).
- **Age with no token** is how long the protocol has run without issuing one (an unlisted age
  uses 365 days).
- **Chain breadth** is how widely the protocol is deployed: a launch reaches more users the
  wider it is deployed, capped at 10 chains.

The board covers all 40 watchlist protocols (the floor) and adds 138 pre-token protocols
below the $50M floor that pass the watchlist's own exclusion rule — no CEX, no bridge or
liquid-staking product, no versioned pool, no wrapped or staked name, no exchange name.
178 rows in all. Live-token protocols are set aside with their market caps shown, never
mixed into the ranking.

Each row carries a seven-step farm plan derived from the protocol's own record (category,
chains, TVL, age): the action, what it costs and how long it takes.

## The dashboard

`dashboard/` is a static page with no network: a sortable, filterable board (search, category,
chain, pre-token / live-token tabs), a detail view for every protocol with the model's own
inputs and the farm plan, and a model panel that states the formula. The page recomputes each
row from the snapshot figures and checks the number `board.json` carries.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Node 22, standard library only. Two runs write the same bytes. The program writes
`board.json`, `data.js`, copies `dashboard/` and `screenshots/` to the output, and writes
`dossier.json` with the vendored make.fun library.

## License

MIT. See LICENSE.
