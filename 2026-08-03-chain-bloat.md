# Bittensor chain bloat: failed extrinsics, commitments, and what a cleanup would reclaim

**Snapshot as of 2026-08-03, block 8,766,284, runtime 441** (Root Reborn landed the same day).
Three instruments, all pinned and reproducible: a full local index of the chain (blocks 0 to
8,766,182: 212M extrinsics, ~894M events), a state census over RPC (every storage key of every
pallet, value sizes sampled), and a 1,347-block raw size sweep across history.

τ = TAO. Sizes are SCALE-encoded payload bytes; real node disk is higher (db indexes, trie
overhead), so every figure is a lower bound.

## TL;DR

- **212.1M extrinsics ever landed, 93.0M failed (43.9%)**: roughly **25 to 40 GB of the ~135 GB**
  an archive node stores is failure junk.
- **The worst offenders are feeless**: `burned_register` (28.1M fails, 96.9% failure rate),
  `set_weights` (25.8M), `set_commitment` (11.6M, 61.8%), `serve_axon` (10.9M, 79.6%),
  `serve_prometheus` (4.8M, **99.6%**).
- **Commitments are a history problem, not a state problem**: 19.5 MB of state vs 18.8M extrinsics
  (8.9% of everything ever sent), 62% failed, all feeless. Peak day 2026-03-28: **1.30M failed
  `set_commitment`**.
- **The whole chain state is ~620 MB**: perfect state cleanup reclaims ~70 MB. The chain grows
  ~**250 MB/day** (~90 GB/year) and **~2/3 of that is noise**: ~175 MB/day of events (mostly
  `Balances.Transfer`/`Deposit` from the root-claim machinery, ~2.2M/day) + ~14 MB/day of failures.
- Failure rate today: **17% over 30 days** (43.9% lifetime), ~29k/day still landing. **90% paid
  zero fees** (the rest paid 100.6 τ). Two rejectable-at-validity classes dominate:
  `CommittingWeightsTooFast` and `AccountNotAllowedCommit`.

## How the numbers are made

- Counts: gap-free PostgreSQL index of blocks 0 → 8,766,182; failure = the on-chain
  `ExtrinsicFailed` event; reasons decoded through the runtime 441 error enums.
- State: full key enumeration per pallet at the pinned block; values fetched fully for small items,
  page-sampled (min 200) and extrapolated for large ones.
- History: 626 blocks at a 14,000-block stride since genesis + 721 blocks over the last 30 days;
  raw body bytes + raw `System.Events` blob bytes.
- Per-call byte costs: 13,842 extrinsics decoded over 504 recent blocks. A failed call costs the
  same bytes as a successful one (identical body).
- Caveats: all-history byte attributions use today's per-call sizes (a low estimate: 2023-era
  `set_weights` were fatter); migration-transient failures of snapshot day
  (`BetaBasketSeedInProgress`) ignored.

## 1. Failed extrinsics: 93.0M of 212.1M, 43.9% of chain history

### By era

Quarterly buckets of 648,000 blocks. `GB ext` / `GB ev` = measured extrinsic and event payload.

| Era | Dates                |          Landed |         Failed | Fail rate |   GB ext |    GB ev |
|----:|----------------------|----------------:|---------------:|----------:|---------:|---------:|
|  q0 | 2023-03 → 2023-06    |      18,329,790 |      9,307,153 |     50.8% |     10.8 |      1.5 |
|  q1 | 2023-06 → 2023-09    |       4,256,454 |      1,228,891 |     28.9% |      4.4 |      0.3 |
|  q2 | 2023-09 → 2023-12    |       9,955,096 |      4,667,733 |     46.9% |      4.4 |      1.0 |
|  q3 | 2023-12 → 2024-03    |      27,893,226 |     17,439,280 |     62.5% |      7.0 |      3.2 |
|  q4 | 2024-03 → 2024-06    |      19,544,799 |     13,228,570 | **67.7%** |      8.8 |      2.5 |
|  q5 | 2024-06 → 2024-09    |      14,538,895 |      9,607,416 |     66.1% |      6.4 |      2.3 |
|  q6 | 2024-09 → 2024-12    |      11,711,626 |      6,822,100 |     58.3% |      6.4 |      1.3 |
|  q7 | 2024-12 → 2025-03    |      15,869,707 |      8,401,714 |     52.9% |      6.1 |      1.9 |
|  q8 | 2025-03 → 2025-06    |      14,829,186 |      5,426,463 |     36.6% |      6.4 |      2.6 |
|  q9 | 2025-06 → 2025-09    |      13,545,766 |      4,304,424 |     31.8% |      4.8 |      2.4 |
| q10 | 2025-09 → 2025-12    |      14,925,538 |      2,947,711 |     19.8% |      6.5 |      3.7 |
| q11 | 2025-12 → 2026-03    |      17,944,748 |      3,379,393 |     18.8% |      7.7 |      4.5 |
| q12 | 2026-03 → 2026-06    |      20,474,907 |      4,836,832 |     23.6% |      8.5 |      7.4 |
| q13 | 2026-06 → 2026-08-03 |       8,270,165 |      1,408,941 |     17.0% |      3.4 |      8.3 |
|     | **Total**            | **212,089,903** | **93,006,621** | **43.9%** | **91.8** | **42.9** |

The failure epidemic peaked in 2024 (62 to 68%) and pool-level guards
(`SubtensorTransactionExtension`) squeezed it to 17 to 23% today. Meanwhile the `GB ev` column
exploded: q13 wrote more event bytes in 7 weeks than any full quarter before 2026 (section 4).

### The all-time top offenders

Bytes use today's mean size per call (`set_weights`' true historical figure is higher).

| Call                                                  |          Fails |  Successes |  Fail rate | Est. wasted GB |
|-------------------------------------------------------|---------------:|-----------:|-----------:|---------------:|
| `SubtensorModule.set_weights`                         |     25,765,857 | 34,514,897 |      42.7% |            9.4 |
| `SubtensorModule.burned_register`                     |     28,139,311 |    912,845 |  **96.9%** |            4.0 |
| `Commitments.set_commitment`                          |     11,634,138 |  7,173,351 |      61.8% |            2.6 |
| `SubtensorModule.serve_axon`                          |     10,872,669 |  2,791,977 |      79.6% |            1.5 |
| `SubtensorModule.commit_timelocked_mechanism_weights` |      1,638,409 |  5,536,893 |      22.8% |            0.9 |
| `SubtensorModule.commit_timelocked_weights`           |      1,335,162 |  4,156,867 |      24.3% |            0.8 |
| `SubtensorModule.serve_prometheus`                    |      4,818,646 |     18,041 |  **99.6%** |            0.6 |
| `SubtensorModule.commit_crv3_weights`                 |        882,679 |  4,707,964 |      15.8% |            0.5 |
| `SubtensorModule.remove_stake`                        |      2,655,854 |  5,262,481 |      33.5% |            0.4 |
| `SubtensorModule.register` (PoW)                      |        805,513 |     24,132 |      97.1% |            0.2 |
| `SubtensorModule.register_network`                    |        655,808 |        209 | **99.97%** |            0.1 |
| everything else                                       |      3,802,575 |            |            |           ~0.9 |
| **Total failed**                                      | **93,006,621** |            |            |     **~22 GB** |

Add ~2.3 GB of `ExtrinsicFailed` events (~25 B each). The body bytes bracket two ways: exact fail
counts × today's per-call sizes give 22 GB (low, old `set_weights` were fatter), per-quarter fail
share × measured quarter bytes gives 39.5 GB (high, failures skew toward small calls). So the
failure footprint is **25 to 40 GB of the ~135 GB payload history**, none of it removable
retroactively. 100%-failure club: `System.set_heap_pages` (169,838 attempts, zero successes ever,
still ~39k/month), `Commitments.set_max_space` (8,307, all failed, root-only), `register_network`
(655,808 fails for 209 successes).

### The last 30 days (blocks ≥ 8,550,378)

5,133,550 landed, 875,742 failed (**17.1%**); last 7 days: 23.4%. The 2026-07-17 emission cut (114
subnets off) dropped the baseline from ~30k to ~18k failures/day; the 07-28 to 07-31 spike (51 to
58k/day) was a bot wave. 90% of failures paid nothing; the rest paid 100.6 τ (successes: 1,156 τ).

| Call                                  | Error                         | Fee paid | Count (30d) |
|---------------------------------------|-------------------------------|----------|------------:|
| `commit_timelocked_mechanism_weights` | `CommittingWeightsTooFast`    | No       |     448,600 |
| `set_commitment`                      | `AccountNotAllowedCommit`     | No       |     156,220 |
| `commit_timelocked_mechanism_weights` | `TooManyUnrevealedCommits`    | No       |      94,168 |
| `commit_timelocked_weights`           | `TooManyUnrevealedCommits`    | No       |      54,390 |
| `System.set_heap_pages`               | `BadOrigin`                   | Yes      |      39,374 |
| `commit_timelocked_mechanism_weights` | `HotKeyNotRegisteredInSubNet` | No       |      15,477 |
| `add_stake_limit`                     | `Swap.PriceLimitExceeded`     | Yes      |      12,951 |
| `set_mechanism_weights`               | `WeightVecLengthIsLow`        | No       |       8,207 |
| `set_commitment`                      | `SpaceLimitExceeded`          | No       |       7,414 |
| `add_stake_limit`                     | `SlippageTooHigh`             | Yes      |       4,455 |

The fee-paying rows at the bottom are healthy (user protection orders doing their job). The top two
classes are feeless and deterministic against state: rate-window duplicates that pass pool
validation then die at dispatch, and deregistered miners' bots still committing. Spam is
concentrated: the top 19 failing signers are all `commit_timelocked_*` bots at 11k to 58k failures
each (champion: `5FLoWCDovMPeH3Gv4syQSZ8TuKcMv6N27g8diDU8zJSeRv8m`, 57,617), followed by six
`set_commitment` bots at ~10k calls each with a **100% failure rate** (`5Gbt98eb…`, `5D7hgR7Q…`,
`5EYwugTF…`, `5CypZ9mE…`, `5ELuakG3…`, `5Gx9ChZy…`).

## 2. Commitments deep dive

### Mechanics

- `set_commitment` is **feeless** (`Pays::No`) and whitelisted even in safe mode.
- Guards: hotkey registered on the subnet (`CanCommit`), max 3 fields, **`MaxSpace` = 3,100
  bytes/user/epoch** (default, never overridden on-chain).
- One entry per (netuid, hotkey), overwritten in place; purged only at subnet dissolution
  (`purge_netuid`). Timelocked fields auto-reveal via drand into `RevealedCommitments` (last 10
  kept).

### Usage: 18.8M extrinsics since 2023-12-14

- **7,173,351 successful + 11,634,138 failed (61.8%)**, 8.9% of every extrinsic ever landed.
- Current run rate: 15 to 20k/day, 24 to 46% failing.
- Peak day 2026-03-28: **1,302,931 failed + 14,556 ok**. One fleet of 757 hotkeys in lockstep (top
  signers at exactly 2,899 failures each), 1.3M junk entries in 24 hours, free.
- Last 30 days by subnet (ok + fails): sn102 **149,636 + 30,155**, sn64 15,128 + **105,810 (87.5%
  fail)**, sn123 57,820 + 7,417, sn68 33,947, sn47 15,273 + 5,468, sn33 9,460. All of these have
  **emission off**; the fleets commit anyway.

### State: the entire pallet is 19.5 MB

| Storage               |   Keys |             Bytes (keys + values) |
|-----------------------|-------:|----------------------------------:|
| `CommitmentOf`        | 28,775 | 4.95 MB (2.82 values + 2.13 keys) |
| `RevealedCommitments` | 16,482 |             7.61 MB (6.39 values) |
| `LastCommitment`      | 44,903 |             3.50 MB (mostly keys) |
| `UsedSpaceOf`         | 37,137 |                           3.34 MB |
| `LastBondsReset`      |    735 |                           0.05 MB |
| `TimelockedIndex`     |      1 |                             375 B |
| **Total**             |        |                     **≈ 19.5 MB** |

The 28,775 live entries:

- **Age**: 13,273 (46%) last touched **over a year ago**, 8,461 more are 90 to 365 days old; only
  1,447 updated in the last week.
- **Location**: 25,760 (89.5%) sit on emission-off subnets.
- **Content**: mostly small `Raw` payloads (~98 B average: IPs, repo URLs, model hashes); 731
  `ResetBondsFlag`, 40 `BigRaw`, 11 timelocked (matches `TimelockedIndex`). Largest entry: 994 B.
  The per-entry bounds work.
- **Oddity**: `LastCommitment`/`UsedSpaceOf` hold 45k/37k keys for 28.8k commitments; ~16k
  rate-limit rows outlive their commitment (cleared only on dissolution).

### What a cleanup reclaims

Purging idle-for-a-year entries, emission-off entries, orphaned rate-limit rows and trimming
reveals: **~15 of the 19.5 MB**. The written history (~4 GB) is permanent. The lever is the flow:
~1.6 GB/year at the current rate, 40% failures; `AccountNotAllowedCommit` alone is checkable at
pool validation and would erase the six 100%-fail bots (~156k landed failures/month).

## 3. The state census: where a node's state actually is

Every key of every pallet at block 8,766,284 (key + value bytes, trie overhead excluded):

| Pallet                                                                                         |      Keys |   Est. size |
|------------------------------------------------------------------------------------------------|----------:|------------:|
| `SubtensorModule` (202 storage items)                                                          | 3,415,632 |      420 MB |
| `System` (553,283 live accounts, 56 B `AccountInfo` each)                                      |   555,694 |       75 MB |
| `EVM` + `Ethereum` + `Contracts` (4,971 contracts, 29.5 MB of bytecode, 270,130 storage slots) |   282,604 |       71 MB |
| `Drand`                                                                                        |   216,004 |     31.5 MB |
| `Commitments`                                                                                  |   128,033 |     19.5 MB |
| `Proxy`                                                                                        |     8,630 |      1.0 MB |
| `Swap`                                                                                         |     3,173 |     0.23 MB |
| `Multisig`, `Crowdloan`, `MevShield`, `LimitOrders`, `AlphaAssets`, rest                       |      ~900 |     ~0.1 MB |
| **Total**                                                                                      | **~4.6M** | **~620 MB** |

Biggest single items:

| Item                                          |    Keys | Est. size | What it is                                                   |
|-----------------------------------------------|--------:|----------:|--------------------------------------------------------------|
| `SubtensorModule.Alpha`                       | 755,830 |  110.4 MB | stake shares per (hotkey, coldkey, subnet)                   |
| `System.Account`                              | 553,283 |   75.3 MB | every live account                                           |
| `SubtensorModule.AlphaV2`                     | 439,150 |   67.6 MB | `Alpha`'s successor (see below)                              |
| `SubtensorModule.LastColdkeyHotkeyStakeBlock` | 453,172 |   54.4 MB | one block number per (coldkey, hotkey) pair that ever staked |
| `EVM.AccountStorages`                         | 270,130 |   40.0 MB | contract storage slots                                       |
| `SubtensorModule.Owner`                       | 308,263 |   34.5 MB | hotkey → coldkey                                             |
| `Drand.Pulses`                                | 216,000 |   31.5 MB | one week of drand rounds                                     |
| `SubtensorModule.StakingHotkeys`              | 140,910 |   29.9 MB | hotkey list per staking coldkey                              |
| `SubtensorModule.TotalHotkeyAlphaLastEpoch`   | 255,629 |   23.0 MB | per-epoch stake snapshot                                     |

Transitional wart: **two generations of the stake maps live side by side** (`Alpha` next to
`AlphaV2`, `TotalHotkeyShares` 113,486 next to `TotalHotkeySharesV2` 22,879); a progressive
in-block migration is walking the old maps (`AlphaV2MapLastKey` cursor, runtime 441). Verify the
V1 maps actually get cleared once it completes, because the pallet has form here:

### Junk found in state

- **Drand pulses: 31.5 MB for a 1-week window** (`MAX_KEPT_PULSES` = 216,000 rounds, ~146 B each).
  Consumers (timelocked commits, commitment reveals) work in hours, not days: a 1-day window holds
  4.5 MB, **27 MB reclaimed for one constant**.
- **86% of the Swap pallet is its dead predecessor.** Twelve V3 AMM items the runtime no longer
  declares: `TickIndexBitmapWords` (1,256 keys), `Ticks` (256), `Positions` (128),
  `AlphaSqrtPrice`, `FeeGlobalTao`, `FeeGlobalAlpha`, `ScrapReservoirTao`, `CurrentLiquidity`,
  `CurrentTick`, `LastPositionId`, `EnabledUserLiquidity`, `SwapV3Initialized` (+1 unidentified
  34 B key). 195,324 of 226,372 B are unreachable orphans.
- **26.8 MB of dead pre-dTAO storage inside `SubtensorModule`**, identified by brute-forcing old
  names: `TotalHotkeyStake` (122,469 keys, 8.8 MB), `PendingdHotkeyEmission` (48,144, 4.2 MB,
  historic typo included), `LastHotkeyEmissionDrain` (44,746, 3.9 MB), `TotalColdkeyStake`
  (40,823, 2.9 MB), `StakeDeltaSinceLastEmissionDrain` (21,490, 2.8 MB), `LastAddStakeIncrease`
  (2,117), `ColdkeyArbitrationBlock` (469), +2 unidentified account-keyed maps (44,044 keys,
  3.8 MB). The dTAO cutover's unswept floor.
- **`UsedWork` never forgets**: PoW proofs insert with no removal path anywhere. 24,125 blobs,
  1.8 MB, only grows.
- **Rate-limit residue**: `LastTxBlock` and friends keep one entry per account forever.

Headline: **the perfect state cleanup reclaims ~70 MB of a 620 MB state**. State fits in RAM; the
disk goes elsewhere.

## 4. Where the disk actually goes: 135 GB of history, and the events firehose

Measured: **91.8 GB of extrinsic bodies + 42.9 GB of events** (headers and db overhead on top; a
pruned validator keeps blocks too unless `--blocks-pruning` is set). Extrinsic bytes are flat
(~70 MB/day), but **events went from a 2023 to 2025 average of ~3 KB/block to 24.3 KB/block**:
~175 MB/day, 2.5× the extrinsic flow.

One day of events (2026-08-02/03):

| Event                                                     | Count/day |
|-----------------------------------------------------------|----------:|
| `Balances.Transfer`                                       | 1,257,892 |
| `Balances.Deposit`                                        |   916,184 |
| `System.ExtrinsicSuccess`                                 |   140,607 |
| `TransactionPayment.TransactionFeePaid`                   |   115,698 |
| `Balances.Withdraw`                                       |    49,491 |
| `SubtensorModule.WeightsSet`                              |    35,879 |
| `System.ExtrinsicFailed`                                  |    34,298 |
| `SubtensorModule.RootClaimed`                             |    33,579 |
| `Drand.NewPulse`                                          |    28,881 |
| `SubtensorModule.TimelockedWeightsCommitted` + `Revealed` |    52,239 |

**~2.2M `Balances.Transfer`/`Deposit` per day, ~81% of event bytes.** Anatomy: ~5 hook-driven root
claims per block, each redemption moving TAO through subnet pallet accounts with real
`Currency::transfer` calls (~38 `Transfer` per claim, 135 to 616 per block observed), plus ~111
coinbase `Deposit`s every block. No user activity, just the emission ledger written into the event
log: **~140 MB/day, ~52 GB/year**. (The encrypted mempool is in there too:
`MevShield.announce_next_key`, 1,192 B every block, ~8.6 MB/day.)

### Current growth budget (last 30 days, measured)

| Flow                                                                     |   MB/day | GB/year | Verdict                                                                        |
|--------------------------------------------------------------------------|---------:|--------:|--------------------------------------------------------------------------------|
| Emission bookkeeping events (`Transfer`/`Deposit`/`Withdraw` from hooks) |     ~140 |     ~52 | removable: use event-less transfer primitives or one aggregate event per claim |
| Other events (weights, pulses, tx receipts)                              |      ~35 |     ~13 | mostly legitimate                                                              |
| Successful extrinsic bodies                                              |      ~58 |     ~21 | the actual product                                                             |
| Failed extrinsic bodies + their `ExtrinsicFailed` events                 |      ~14 |      ~5 | removable: validity-level rejection + fees                                     |
| **Total**                                                                | **~250** | **~90** | **~2/3 is noise**                                                              |

## 5. The cleanup ledger: everything deletable, in one table

One-shot **state** cleanup (a single runtime migration):

| Target                                                                                                                                                                                                                                        | Mechanism          | Reclaims   |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|------------|
| 9 dead pre-dTAO `SubtensorModule` items (`TotalHotkeyStake`, `PendingdHotkeyEmission`, `LastHotkeyEmissionDrain`, `TotalColdkeyStake`, `StakeDeltaSinceLastEmissionDrain`, `LastAddStakeIncrease`, `ColdkeyArbitrationBlock`, 2 unidentified) | `kill_prefix`      | 26.8 MB    |
| `Drand.Pulses` window, 1 week → 1 day (`MAX_KEPT_PULSES` 216,000 → 28,800)                                                                                                                                                                    | one constant       | 27.0 MB    |
| Commitments idle ≥ 1 year (13,273 entries) + orphaned `LastCommitment`/`UsedSpaceOf` rows + `RevealedCommitments` trimmed to the last reveal                                                                                                  | migration          | ~15 MB     |
| `UsedWork` (24,125 PoW blobs, zero readers)                                                                                                                                                                                                   | `kill_prefix`      | 1.8 MB     |
| 12 Swap V3 orphan items                                                                                                                                                                                                                       | `kill_prefix`      | 0.2 MB     |
| **State total**                                                                                                                                                                                                                               |                    | **~71 MB** |
| Watchlist: `Alpha` + `TotalHotkeyShares` V1 maps, once the V2 migration cursor completes                                                                                                                                                      | verify, then clear | ~121 MB    |

Ongoing **flow** cleanup (policy changes, valued at the current run rate):

| Target                                                                                      | Mechanism                                 | Avoids                 |
|---------------------------------------------------------------------------------------------|-------------------------------------------|------------------------|
| Emission bookkeeping events (root-claim `Transfer` fan-out, per-subnet coinbase `Deposit`s) | event-less transfers or 1 event per claim | ~52 GB/year            |
| Landed failures (feeless commit/commitment spam)                                            | pool-level guards + fees on failed calls  | ~5 GB/year             |
| **Flow total**                                                                              |                                           | **~57 of ~90 GB/year** |

The asymmetry is the whole report: ~71 MB once, versus ~57 GB **per year**, forever.

## 6. Recommendations, ranked by reclaimed bytes

1. **Silence the emission bookkeeping (~52 GB/year).** Move root-claim redemptions and coinbase
   deposits with event-less balance primitives, or emit one aggregate event per claim instead of
   ~38. Subtensor already trimmed per-account emission events once for the same reason. This alone
   roughly **halves chain growth**.
2. **Extend pool guards to the two failure factories (~5 GB/year).** `AccountNotAllowedCommit` is
   checkable at validation; give commits a `provides` tag per (hotkey, netuid, rate-window) so
   window duplicates fight in the pool instead of landing. Together: ~89% of the month's 875,742
   failures.
3. **Make chronic feeless failure expensive.** Charge the normal fee when a `Pays::No` call fails
   (post-dispatch repricing; the hook exists in `pallet_transaction_payment`). The 100%-fail bots
   exist because failure is free.
4. **One-shot state migration (~70 MB, cheap).** Everything in the section 5 state table; and once
   the `AlphaV2` cursor finishes, verify the V1 maps get cleared (~120 MB at stake).
5. **Node operators, today**: a non-archive validator with `--blocks-pruning 256` + state pruning
   drops nearly all of the 135 GB history. Worth documenting officially.

## Appendix: reproduction

- Failure counts: index every block's extrinsics + `System.Events`, join `ExtrinsicFailed` by
  extrinsic index; quarters are `block / 648,000`. Reasons: `dispatch_error.Module.{index,
  error[0]}` through runtime 441 metadata (`SubtensorModule` = pallet 7, `Commitments` = 18,
  `Swap` = 28).
- State census: `state_getKeysPaged(twox128(pallet), 1000, …, at)`, keys bucketed by bytes 16..32
  (`twox128(item)`), values via `state_queryStorageAt`. Orphans = key buckets matching no metadata
  storage entry, named by brute-forcing candidates from old pallet sources.
- Commitments: `CommitmentOf` values decoded as `Registration { deposit: u64, block: u32, info }`;
  netuid from key bytes 32..34; ages against the snapshot block. Auxiliary pin: block 8,766,303.
- Block sweep: `chain_getBlock` (raw bodies) + `state_getStorage(System.Events)` (raw blob) at
  sampled heights; every scan pinned to one explicit endpoint and one block hash. Stratified
  sampling error on the totals (95% CI): extrinsic bodies 91.8 ± 4.4 GB, event blobs 42.9 ± 2.2 GB.
- Subnets at snapshot: 129 live (`NetworksAdded`), 27 with `SubnetEmissionEnabled = true`.
  `System.Account` walked in full: 553,283 live keys, 56 B values (u64 balances); the chain indexer
  knows 2.32M addresses ever seen, so 4 in 5 hold no state today.

On-chain data, for information only. Addresses are public chain data; "bot" labels are behavioral
inferences (volume, failure rate), not identity claims.
