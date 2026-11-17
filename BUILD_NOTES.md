# Token Radar — build notes (radar-2)

Entry for make.fun competition `token-radar-01` ("Build the best token launch radar").
Deadline: **2026-12-14T10:00:00Z (Mon 14 Dec 2026, 17:00 WIB)**. Reward $1,325 total
(1st $1,000; 2nd–5th $50; 6th–10th $25). Entry fee $0. Built 2026-11-17 in
`~/workspace/makefun/token-radar-01/` from the kit at
`~/workspace/makefun/token-radar-01-kit/token-radar-01-kit/` (kit dir untouched).

## What it is

- `main.ts` (Node 22, stdlib only, no network) reads the sealed snapshot and writes
  `board.json`, `data.js`, `dashboard/`, `screenshots/`, `dossier.json` to the output dir.
- `lib/model.ts` — radar-2 model, fit + validation + board + farm plans.
- `dashboard/` — static page (index.html + app.css + app.js, 34,775 bytes total, no
  network, no external URLs). Sortable/filterable board, per-protocol detail drawer,
  live-token tab, calibration table, validation panel.
- `screenshots/` — 3 real renders of this dashboard (see below).
- `vendor/makefun-board.ts` — make.fun's dossier library, unchanged from the kit.
- `LICENSE` — MIT, unchanged from the kit.

## The model (radar-2)

`implied = round(TVL x multiple(category) x growthMult x maturityMult)`

- `multiple(category) = min(2.0, (n x median(mcap/TVL in category) + 5 x global_median) / (n + 5))`,
  fitted **at run time** from the snapshot's own live-token protocols (128 with a
  market cap and a TVL, both positive). Global median mcap/TVL = **0.2138x**.
  A category with many live tokens keeps its own median; a thin one is pulled toward
  the global median; the 2.0 cap stops one outlier protocol setting a whole category.
  Categories with no live token use the global median. Examples: Lending n=15
  median 0.1324 → fitted 0.1527; Dexs n=11, 0.3424 → 0.3022; Liquid Staking n=9,
  0.0406 → 0.1025; Risk Curators n=1, 2.9747 → 0.6740; Bridge n=6, 0.5567 → 0.4009.
- `growthMult = 1 + clamp(change_7d, -60, 120)/100 x 0.5`; `maturityMult = 0.6 +
  min(age_days, 1095)/1095 x 0.4` (unknown age = 365 days).
- One function scores every row. No protocol name is special-cased.

## The board

- **178 rows**: the full 40-protocol watchlist + 138 early rows below the $50M floor.
  Early rows meet the watchlist's own bar minus only the floor: pre-token, not an
  exchange, not a versioned pool, not a wrapped/staked/bridged product (the starter
  took 160 early rows by TVL with a looser bar, 200 rows total).
- 24 live-token protocols set aside by market cap (shown in the Live token tab).
- Every row: rank, implied value, and an **8-step farm plan** (fresh wallet, category
  action, second qualifying action, weekly cadence, hold period, signal sizing from
  the row's own implied/TVL ratio, claim safety, farm log), each step 20–600 chars.
- Top 10: steakhouse-financial $1,465,642,872; gauntlet $872,901,344;
  nonce-capital $203,073,192; veda $194,616,843; usdd $168,729,663;
  rockawayx $133,350,519; hive-protocol $120,780,803; hyperithm $116,653,080;
  k3-capital $106,792,721; clearstar $82,655,840. All 178 implied values distinct.

## Validation (honest numbers, computed by main.ts at run time)

5-fold cross-validation over the 128 live-token protocols (sorted by slug,
fold = index mod 5; each fold's fit re-run on its training set only):

- radar-2: **MdAPE 0.927, within-2x 20.3%, median signed error -0.012, Spearman 0.40**
- fixed-0.25 baseline, same folds: MdAPE 0.886, within-2x 29.7%, Spearman 0.432

The category fit is statistically indistinguishable from a fixed multiple on this
holdout — TVL alone predicts a market cap only loosely (mcap/TVL median 0.21,
mean 5.04: extremely right-skewed). The dashboard states this plainly ("a screen
for where to look, not a price"). The category table is kept because the
differences it states are real in the data (liquid staking ~0.04–0.10x TVL vs
bridges ~0.40–0.56x), not because the holdout proves an edge. These numbers were
reproduced independently in Python before being baked into the model header comment.

## What changed vs the starter (radar-1)

1. Model: fixed 0.25 multiple → per-category multiples fitted at run time from the
   snapshot's live tokens, with shrinkage (k=5) and a 2.0 cap, plus a 5-fold
   out-of-sample validation computed at run time and shown in the dashboard.
2. Board rule: early rows now meet the watchlist's quality bar (wrapped/staked/
   bridged/exchange/versioned held out); 178 rows instead of 200, all rankable
   pre-token protocols under that bar.
3. Farm plans: 6 generic steps → 8 steps, with a second qualifying action
   (chain/category-specific) and a signal-sizing step using the row's implied/TVL.
   Plans are carried in data.js so the page renders exactly what board.json holds
   (no duplicated logic to drift).
4. Dashboard: added chain filter, watchlist-only toggle, WATCHLIST tags, a model
   strip under the lede (formula + validation), a calibration table + validation
   panel below the board, and a detail view that shows the category multiple with
   its live-token count plus a recompute check against board.json.
5. Screenshots: regenerated as real renders of the new dashboard (see below).

## Verification (all run 2026-11-17, Node v24.22.1)

- `main.ts` runs twice to two output dirs: `diff -r` → **byte-identical**.
- Gate, all 7 rows replicated from RULES.txt — **PASS**:
  1. dashboard_files: 3 files, 34,775 bytes (≤400 files, ≤4 MiB), index.html + .css + .js.
  2. board_file: 178 rows (100–4,000), every slug in the snapshot, all implied values
     positive integers ≤ 1e14.
  3. sorted_by_value: ranks exactly 1..178, implied values non-increasing, 178 distinct.
  4. coverage: all 40 watchlist slugs present.
  5. farm_steps: 8 per row, every step 119–243 chars (20–600 required).
  6. screenshots: 3 PNGs, each 1440x900, 156–227 KiB (≤512 KiB).
  7. dossier_matches: `buildDossier` recomputed from the output dir is byte-identical
     to the written `dossier.json` (710,297 bytes); dossier text 35,039 / 65,536 bytes.
- No `http://` or `https://` string anywhere in `dashboard/`.
- No local grade exists for this competition (RULES.txt: "the server grades every
  entry"); the `check` command needs a repo + transcript bundle and is run at submit time.

## Screenshots (real renders, not placeholders)

- `1-board.png` — board, top of the page. `2-detail.png` — detail drawer for rank 1
  (steakhouse-financial). `3-filtered.png` — board filtered to category Dexs
  (20 rows shown). All 1440x900.
- Method: `/opt/meta-chromium/chrome` (Chromium 157) would not start headless
  without a WebGL domain policy, and its `--screenshot` CLI produced no file at
  all. Workaround that worked: a policy file containing `hatch-webgl-all-origins-v1`
  (`/tmp/webgl-policy.txt`, passed with `--hatch-webgl-allowed-domains-file` and its
  SHA-256 via `--hatch-webgl-allowed-domains-sha256`), Chrome kept alive with
  `--remote-debugging-port=9222`, and screenshots driven over CDP by
  `~/workspace/makefun/shot.mjs` (dependency-free Node script: create target, navigate,
  wait 2.5 s, `Page.captureScreenshot`). The board/detail/filtered probes confirmed
  the renders (212 table rows on the board page incl. calibration table, dialog open
  on the detail page, 20 Dexs rows on the filtered page). The background Chrome was
  stopped after capturing. All three PNGs were visually inspected.

## Transcript bundle (checked, not yet built for real)

- `bundle --dry-run` from the kit dir finds **2 sessions**: this build session
  (`bc558a73-952e-4ca5-b736-e3d9fb11e405`) and the parent vetting session
  (`fbc4f14e-813b-4937-8da7-bb15d19e7a5b`), both cut to this repository's work.
- Two things the real bundle still needs (parent/top-level agent does these at
  submit time, per the kit's own warnings):
  1. `--harness-version <version>` — bundle refuses to write without it.
  2. `--muse-usage <file>` — 0 usage rows are attached yet; Muse writes a
     message's usage row when the message is delivered, and a subagent's row only
     once its run completes. The top-level agent must run the SELECT from
     `bundle --help` with its own agent id and each subagent's, then bundle again.
- The entry repo is a local git repo (`git init`, no remote); commit before bundling
  so the bundle is not recorded `repo.dirty`. Session began 2026-11-17, after the
  competition opened (2026-11-14T10:00:00Z), so the time window is fine. The local
  commit (65bbb7f) uses placeholder identity `Token Radar <radar@example.invalid>` —
  amend the author before pushing if the owner wants their own identity on it.

## Known gaps / risks

- The model's out-of-sample edge over a fixed multiple is nil on this snapshot
  (see Validation). The entry competes on the visible method, the calibration
  table, the farm plans, and the dashboard — not on a proven predictive edge.
- Rank 1–3 are all Risk Curators on the same fitted 0.6740x multiple, which rests
  on a single live comp (n=1, median 2.9747, shrunk + capped). The dashboard shows
  the n=1 basis in the detail view and the calibration table, so it is transparent.
- Screenshots are 1440x900 viewport captures; the detail drawer is scrollable and
  its lower farm steps are below the fold in the capture (the panel itself shows
  all 8 steps when scrolled).

## Remaining steps (parent agent / user — NOT done here)

1. **GitHub repo (user: jephiqx24, per memory 2026-11-17):** create a NEW public
   repo `token-radar-01` — it must be created after the competition opened
   (2026-11-14T10:00:00Z); an older repo cannot be entered. Push this directory as
   its first commit.
2. **README badges:** replace `{owner}` / `{repo}` in `README.md` with the real
   owner (`jephiqx24`) and repo name (`token-radar-01`) and commit — make.fun
   refuses a README with a template placeholder. Set the repo's GitHub About:
   website on make.fun, topic `makefun`.
3. **Wallet (user makes it):** new Phantom Solana wallet for the make.fun account
   (per memory 2026-11-17, user creates it — do NOT reuse the BSC jephx wallet).
   Register/login the CLI with it (`node cli/makefun.mjs register --keypair
   wallet.json --origin https://make.fun` from the kit dir) and keep `wallet.json`
   mode 600, never committed.
4. **Owner claim file:** only if the repo's GitHub owner is NOT the make.fun
   account's GitHub login — add `.makefun/owner.json` holding
   `{"wallet": "<base58 address>"}` with the first commit. If the GitHub owner
   matches the make.fun login, no file is needed. (Wallet address not known yet —
   no such file was created here, and none should be guessed.)
5. **Bundle transcripts** (from the kit dir, after the top-level turn ends so usage
   rows exist):
   `node cli/makefun.mjs bundle --repo <repo> --harness Muse --harness-version <version> --provider meta --model muse-spark-1.3 --effort default --mcp none --instructions withheld:<why> --system-prompt <file> --muse-usage <file> --out transcripts.tar.gz`
6. **Check:** `node cli/makefun.mjs check --kit . --repo <repo> --transcripts transcripts.tar.gz`
7. **Submit** (entry quota 5/day, 10 total — preserved, nothing submitted yet):
   `node cli/makefun.mjs submit --competition token-radar-01 --repo-url https://github.com/jephiqx24/token-radar-01 --commit <sha> --title "Token Radar: pre-token protocols ranked by implied value" --bundle transcripts.tar.gz --keypair wallet.json`
   Then watch the entry's dev grade (transcript gate → two sandbox runs → 12 games
   vs the house dashboards).
