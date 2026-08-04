# Bittensor chain bloat: failed extrinsics, commitments, and what a cleanup would reclaim

**Snapshot as of 2026-08-03, block 8,766,284, runtime 441** (the Root Reborn upgrade went live earlier
the same day). Three instruments, all pinned and reproducible:

- a **full local index of the chain** (every block, extrinsic and event from block 0 to 8,766,182:
  212M extrinsics, ~894M events), used for exact counts;
- a **state census over RPC**: every storage key of every pallet enumerated at the snapshot block,
  value sizes sampled per storage item (`state_getKeysPaged` + `state_queryStorageAt`);
- a **1,347-block size sweep** across the whole history (raw block bodies + raw `System.Events`
  blobs, no decoding), used to measure where the gigabytes actually are.

τ = TAO. Sizes are SCALE-encoded payload bytes (key bytes + value bytes for state; extrinsic bytes and
event-blob bytes for history). Actual disk usage of a node is higher (db indexes, trie overhead), so
every figure here is a lower bound on what a node really pays.

## TL;DR

- **212.1M extrinsics ever landed on chain. 93.0M of them failed (43.9%).** Failure junk accounts for
  roughly **30 to 45 GB of the ~135 GB** of block + event payload an archive node stores.
- **The worst offenders are feeless calls**: `burned_register` (28.1M fails, 96.9% failure rate),
  `set_weights` (25.8M fails), `set_commitment` (11.6M fails, 61.8% failure rate), `serve_axon`
  (10.9M fails, 79.6%), `serve_prometheus` (4.8M fails, **99.6% failure rate**).
- **Commitments are a history problem, not a state problem.** The whole Commitments pallet state is
  **19.5 MB**. But the pallet has produced 18.8M extrinsics (8.9% of everything ever sent to the
  chain), 62% of which failed, all feeless. Peak day: 2026-03-28, **1.30M failed `set_commitment` in
  one day**.
- **The entire chain state is ~620 MB: state cleanup is measured in MB, flow cleanup in GB/year.**
  Deleting every stale commitment, every orphaned storage item (26.8 MB of dead pre-dTAO maps sit in
  `SubtensorModule` alone) and shrinking the drand window would reclaim ~70 MB. Meanwhile the
  chain currently grows ~**250 MB/day** (~90 GB/year), and about **2/3 of that is bookkeeping noise**:
  ~**175 MB/day of events** (mostly `Balances.Transfer`/`Deposit` pairs emitted by the root-claim
  payout machinery, ~2.2M per day) plus ~16 MB/day of failed extrinsics.
- Pool guards already crushed the failure rate (43.9% lifetime average, **17% over the last 30 days**),
  but ~29k failures/day still land (58k on peak days), dominated by two rejectable-at-validity
  classes: `CommittingWeightsTooFast` and `AccountNotAllowedCommit`. **90% of the month's failures
  paid zero fees**; the fee-paying minority contributed just 100.6 τ.

## How the numbers are made

Counts come from a complete chain index in PostgreSQL (block 0 → 8,766,182, gap-free: row count equals
head number). Failure status is the on-chain `System.ExtrinsicFailed` / `ExtrinsicSuccess` event per
extrinsic; failure reasons decode the `dispatch_error` module + error index through the runtime 441
metadata enums. State sizes: full key enumeration per pallet prefix at the pinned block; value sizes
fetched for 100% of small items and page-sampled (min 200 values) for large ones, then extrapolated by
mean value size. History sizes: 626 blocks sampled at a 14,000-block stride since genesis plus 721
blocks at a 300-block stride over the last 30 days; extrinsic bytes are the raw hex bodies,
event bytes are the raw `System.Events` storage value at that block. Per-call byte costs: 13,842
extrinsics decoded over 504 recent blocks, mean size per call. One assumption: a failed call costs
the same bytes as a successful one of the same call (true by construction, the body is identical; only
the result event differs).

Caveats: per-call sizes use the current mix, so all-history byte attributions are a low estimate
(2023-era `set_weights` on 1024-uid subnets were fatter than today's 364-byte average). The census
block sits a few hours after the Root Reborn migration; a handful of failures that day
(`BetaBasketSeedInProgress`) are migration-transient and ignored.

## 1. Failed extrinsics: 93.0M of 212.1M, 43.9% of chain history

### By era

Quarterly buckets of 648,000 blocks (~90 days). `GB ext` / `GB ev` are the measured extrinsic and
event payload per era from the block sweep.

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

Two reads on this table. First, the failure epidemic peaked in 2024 (the registration-war era, 62 to
68% of everything failing) and has been squeezed down to 17 to 23% today by pool-level guards
(`SubtensorTransactionExtension`: weight rate limits, serving endpoint checks, stake sanity checks all
reject at the mempool now, before anything lands). Second, note the `GB ev` column: **event bytes per
era have exploded** while extrinsic bytes stayed flat. q13 wrote more event bytes in 7 weeks than any
full quarter of 2023 to 2025. Section 4 explains why.

### The all-time top offenders

Estimated bytes use today's mean size per call (footnote for `set_weights`: the true historical figure
is higher).

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

Add the `System.ExtrinsicFailed` event each failure writes (~110 B): another **~10 GB**. With the
historical size correction for early fat `set_weights`, the failure footprint lands at **30 to 45 GB
of the ~135 GB payload history**. None of it is removable retroactively (blocks are immutable, and
this is precisely why letting junk land is expensive: it is permanent). The gain is prospective.

Honorable mentions in the 100%-failure club: `System.set_heap_pages`, 169,838 attempts, **zero
successes ever** (a root-only call, so every single one was a bot paying a fee to be told
`BadOrigin`; it still runs at ~39k/month today). `Commitments.set_max_space`: 8,307 attempts, all
failed, same story. `register_network`: 655,808 fails for 209 successes, the lock-cost race at its
finest.

### The last 30 days (blocks ≥ 8,550,378)

30-day totals: 5,133,550 extrinsics landed, 875,742 failed (**17.1%**); last 7 days ran hotter,
1,355,486 landed and 317,678 failed (**23.4%**). Daily failure rate ranged 12.4% (2026-07-26) to
27.3% (2026-07-30). The 2026-07-17 emission cut (114 subnets switched off) visibly dropped the
baseline from ~30k to ~18k failures/day; the 07-28 to 07-31 spike (51 to 58k/day) was a third-party
bot wave. Fees: 788,564 of the month's failures (90%) paid nothing at all; the paying minority
contributed 100.6 τ, against 1,156.0 τ from successful calls.

Where failures come from now, decoded through the runtime enums:

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

The fee-paying failures at the bottom are healthy: `PriceLimitExceeded` / `SlippageTooHigh` are user
protection orders doing their job, and they pay their way. The problem is the top: **the two biggest
failure classes are feeless and deterministic against current state**, exactly the profile the
existing pool guards were built for. `CommittingWeightsTooFast` lands because a bot fires several
commits inside one rate-limit window; each passes pool validation against pre-inclusion state, then
every one after the first fails at dispatch. `AccountNotAllowedCommit` is deregistered miners' bots
still committing metadata to subnets they are no longer part of.

Individual spam is concentrated: the top 19 failing signers of the month are all
`commit_timelocked_*` bots at 11k to 58k failures each (the champion,
`5FLoWCDovMPeH3Gv4syQSZ8TuKcMv6N27g8diDU8zJSeRv8m`, wasted 57,617 calls). Right behind them, six
`set_commitment` bots each fired ~10k commits with a **100% failure rate**
(`5Gbt98eb…`, `5D7hgR7Q…`, `5EYwugTF…`, `5CypZ9mE…`, `5ELuakG3…`, `5Gx9ChZy…`): dead automation
nobody turned off, costing nothing to its owners.

## 2. Commitments deep dive

### Mechanics (why it spams so well)

`Commitments.set_commitment` is **feeless** (`Pays::No`) and whitelisted even in safe mode. Guards:
the hotkey must be registered on the subnet (`CanCommit`), at most 3 fields, and a per-user space
budget of **`MaxSpace` = 3,100 bytes per epoch** (the default; the on-chain override storage has never
been set, and the 8,307 attempts to call `set_max_space` all failed since it is root-only). Data is
overwritten in place per (netuid, hotkey): the state cannot grow past registered keys × max payload.
Commitments are purged only when a subnet is dissolved (`purge_netuid`, wired into the dissolution
path). Timelock-encrypted fields auto-reveal via drand and move to `RevealedCommitments`, capped at
the last 10 reveals per key.

### Usage: 18.8M extrinsics since 2023-12-14

- **7,173,351 successful + 11,634,138 failed (61.8%)**, i.e. 8.9% of every extrinsic ever landed.
- Current run rate: 15 to 20k `set_commitment`/day, of which 24 to 46% fail.
- Peak day: **2026-03-28, 1,302,931 failed + 14,556 successful**. One coordinated fleet: 757 hotkeys
  firing in lockstep (the top signers each landed exactly 2,899 failures that day), 1.3M permanent
  junk entries written into history in 24 hours, for free.
- Last 30 days by subnet (successes + fails): sn102 **149,636 + 30,155**, sn64 15,128 + **105,810
  (87.5% fail)**, sn123 57,820 + 7,417, sn68 33,947, sn47 15,273 + 5,468, sn33 9,460. Every one of
  these subnets currently has **emission switched off**; the fleets keep committing anyway.

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

Structure of the 28,775 live commitments:

- **Age**: 13,273 entries (46%) were last touched **over a year ago**; another 8,461 are 90 to 365
  days old. Only 1,447 were updated in the last week. Two thirds of the pallet is an abandoned
  guestbook.
- **Location**: 25,760 entries (89.5%) sit on subnets with emission currently off.
- **Content**: overwhelmingly small `Raw` payloads (the average entry is ~98 bytes of miner metadata:
  IPs, repo URLs, model hashes); 731 entries carry the `ResetBondsFlag` marker, 40 use `BigRaw` (up
  to 512 B), 11 are currently timelock-encrypted (matches the `TimelockedIndex`). The single largest
  entry is 994 B on sn21. Nobody is close to abusing size per entry; the pallet's own bounds work.
- Oddity: `LastCommitment` and `UsedSpaceOf` hold 45k/37k keys for 28.8k live commitments: ~16k
  rate-limit and quota rows outlive their commitment (cleared only on dissolution, not when a
  commitment disappears or a hotkey deregisters).

### What a Commitments cleanup would reclaim

Purging every commitment untouched for a year, every entry on emission-off subnets, every orphaned
`LastCommitment`/`UsedSpaceOf` row, and trimming `RevealedCommitments` to the last reveal would
reclaim **on the order of 15 of the 19.5 MB**. That is the whole prize in state. The history already
written (18.8M extrinsics, ~4 GB with their events) is permanent.

The real lever is the flow: at the current 15 to 20k/day the pallet writes ~1.6 GB/year of history
(bodies + `Commitment` events + `ExtrinsicFailed` events), 40% of it failures. Concretely:
`AccountNotAllowedCommit` is checkable at pool validation (registration is in state; the same check
`CanCommit` runs at dispatch), which would erase the six 100%-fail bots and ~156k landed
failures/month without touching any legitimate miner.

## 3. The state census: where a node's state actually is

Whole-chain storage at block 8,766,284, every key counted, sizes per storage item (key bytes + value
bytes, trie overhead excluded).

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

The biggest single items, chain-wide:

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

One transitional wart: the runtime currently holds **two generations of the stake maps side by side**,
`Alpha` (755,830 keys, 110.4 MB) next to `AlphaV2` (439,150 keys, 67.6 MB) and `TotalHotkeyShares`
(113,486) next to `TotalHotkeySharesV2` (22,879). A progressive in-block migration is walking the old
maps (`AlphaV2MapLastKey` cursor, part of the runtime 441 rollout that landed the day of this
snapshot). Fine as a transition; the thing to verify once the cursor completes is that the V1 maps
actually get cleared, because the pallet has form here, as the next list shows.

### Junk found in state

- **Drand pulses: 31.5 MB for a 1-week window.** `Drand.Pulses` keeps `MAX_KEPT_PULSES = 216,000`
  rounds (one week of 3-second quicknet rounds, ~146 B each with keys). Pruning works (the map is
  exactly at its cap), but the consumers are timelocked weight commits and commitment reveals whose
  reveal horizons are epochs (hours), not days. A 1-day window would hold ~4.5 MB: **27 MB reclaimed**
  for one constant.
- **86% of the Swap pallet is its dead predecessor.** Twelve storage items of the retired V3
  concentrated-liquidity AMM survive in state under a runtime that no longer declares them:
  `TickIndexBitmapWords` (1,256 keys), `Ticks` (256), `Positions` (128), `AlphaSqrtPrice`,
  `FeeGlobalTao`, `FeeGlobalAlpha`, `ScrapReservoirTao`, `CurrentLiquidity`, `CurrentTick`,
  `LastPositionId`, `EnabledUserLiquidity`, `SwapV3Initialized` (plus one 34-byte unidentified
  single key). 195,324 B of the pallet's 226,372 B total are unreachable orphans; a `kill_prefix`
  migration erases them.
- **26.8 MB of dead pre-dTAO storage inside `SubtensorModule`.** Nine orphaned items survive under
  hashes the runtime no longer declares, identified here by brute-forcing old storage names:
  `TotalHotkeyStake` (122,469 keys, 8.8 MB), `PendingdHotkeyEmission` (48,144, 4.2 MB, historic typo
  included), `LastHotkeyEmissionDrain` (44,746, 3.9 MB), `TotalColdkeyStake` (40,823, 2.9 MB),
  `StakeDeltaSinceLastEmissionDrain` (21,490, 2.8 MB), `LastAddStakeIncrease` (2,117),
  `ColdkeyArbitrationBlock` (469), plus two unidentified account-keyed maps (44,044 keys, 3.8 MB).
  This is the dTAO cutover's unswept floor: the old global-stake and emission-drain accounting was
  replaced in early 2025 and never deleted.
- **`UsedWork` never forgets.** Every PoW registration proof ever accepted inserts a work blob with
  no removal path anywhere in the code. PoW registration is nearly dead (805k fails for 24k successes
  lifetime), the map only grows: 24,125 blobs, 1.8 MB at the snapshot.
- **Rate-limit residue.** `LastTxBlock` and friends keep one entry per account that ever touched a
  rate-limited call, forever (removal exists only inside hotkey-swap flows).

The honest headline of this section: **the perfect state cleanup reclaims ~70 MB of a 620 MB
state**. Substrate state is not where Bittensor's disk goes; it fits in RAM. The next section is
where the disk goes.

## 4. Where the disk actually goes: 135 GB of history, and the new events firehose

Measured payload across the sweep: **91.8 GB of extrinsic bodies + 42.9 GB of event blobs**. Headers,
justifications and db overhead come on top; an archive node stores all of it forever, a pruned
validator keeps the blocks too unless `--blocks-pruning` is set.

The composition flipped in 2026. Extrinsic bytes are flat (~70 MB/day now, peak eras were similar),
but **events went from a 2023 to 2025 average of ~3 KB/block to 24.3 KB/block today**: ~175 MB/day,
2.5× the extrinsic flow, ~64 GB/year run rate.

One day of events (2026-08-02/03, counts):

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

**~2.2M `Balances.Transfer`/`Deposit` events per day, ~81% of all event bytes.** Anatomy, from block
inspection: every block runs ~5 hook-driven root claims (`RootClaimed`, no extrinsic behind them),
and each claim's basket redemption moves TAO through subnet pallet accounts with real
`Currency::transfer` calls, ~38 `Transfer` events per claim (135 to 616 per block observed). On top,
a steady ~111 hook `Deposit` events every block from per-subnet coinbase. None of this is user
activity; it is the emission ledger being written into the event log. At ~80 B per `Transfer` and
~45 B per `Deposit` this machinery alone writes **~140 MB/day, ~52 GB/year**, and it is why q13's
event column dwarfs every historical quarter.

For comparison, the encrypted mempool is in the ledger too: `MevShield.announce_next_key` lands
every block at 1,192 B (~8.6 MB/day), the price of its per-block key rotation.

### Current growth budget (last 30 days, measured)

| Flow                                                                     |   MB/day | GB/year | Verdict                                                                        |
|--------------------------------------------------------------------------|---------:|--------:|--------------------------------------------------------------------------------|
| Emission bookkeeping events (`Transfer`/`Deposit`/`Withdraw` from hooks) |     ~140 |     ~52 | removable: use event-less transfer primitives or one aggregate event per claim |
| Other events (weights, pulses, tx receipts)                              |      ~35 |     ~13 | mostly legitimate                                                              |
| Successful extrinsic bodies                                              |      ~58 |     ~21 | the actual product                                                             |
| Failed extrinsic bodies + their `ExtrinsicFailed` events                 |      ~16 |      ~6 | removable: validity-level rejection + fees                                     |
| **Total**                                                                | **~250** | **~90** | **~2/3 is noise**                                                              |

## 5. The cleanup ledger: everything deletable, in one table

One-shot **state** cleanup (a single runtime migration):

| Target                                                                                                                                               | Mechanism            | Reclaims    |
|------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|-------------|
| 9 dead pre-dTAO `SubtensorModule` items (`TotalHotkeyStake`, `PendingdHotkeyEmission`, `LastHotkeyEmissionDrain`, `TotalColdkeyStake`, `StakeDeltaSinceLastEmissionDrain`, `LastAddStakeIncrease`, `ColdkeyArbitrationBlock`, 2 unidentified) | `kill_prefix`        | 26.8 MB     |
| `Drand.Pulses` window, 1 week → 1 day (`MAX_KEPT_PULSES` 216,000 → 28,800)                                                                           | one constant         | 27.0 MB     |
| Commitments idle ≥ 1 year (13,273 entries) + orphaned `LastCommitment`/`UsedSpaceOf` rows + `RevealedCommitments` trimmed to the last reveal          | migration            | ~15 MB      |
| `UsedWork` (24,125 PoW blobs, zero readers)                                                                                                          | `kill_prefix`        | 1.8 MB      |
| 12 Swap V3 orphan items                                                                                                                              | `kill_prefix`        | 0.2 MB      |
| **State total**                                                                                                                                      |                      | **~71 MB**  |
| Watchlist: `Alpha` + `TotalHotkeyShares` V1 maps, once the V2 migration cursor completes                                                             | verify, then clear   | ~121 MB     |

Ongoing **flow** cleanup (policy changes, valued at the current run rate):

| Target                                                                                                    | Mechanism                                  | Avoids           |
|-----------------------------------------------------------------------------------------------------------|--------------------------------------------|------------------|
| Emission bookkeeping events (root-claim `Transfer` fan-out, per-subnet coinbase `Deposit`s)                | event-less transfers or 1 event per claim  | ~52 GB/year      |
| Landed failures (feeless commit/commitment spam)                                                          | pool-level guards + fees on failed calls   | ~6 GB/year       |
| **Flow total**                                                                                            |                                            | **~58 of ~90 GB/year** |

The asymmetry is the whole report: the one-shot prize is ~71 MB, the policy prize is ~58 GB **per
year**, forever.

## 6. Recommendations, ranked by reclaimed bytes

1. **Silence the emission bookkeeping (~52 GB/year).** Root-claim basket redemptions and coinbase
   deposits should move funds with the event-less balance primitives (or emit one aggregated event
   per claim instead of ~38). Subtensor already did exactly this once: per-account emission events
   were trimmed years ago for the same reason. This single change roughly **halves chain growth**.
2. **Extend pool guards to the two failure factories (~5 GB/year and a cleaner mempool).**
   `AccountNotAllowedCommit` (registration is checkable at validation, the dispatch already does it)
   and `CommittingWeightsTooFast`/`TooManyUnrevealedCommits` (add a `provides` tag per
   (hotkey, netuid, rate-window) so duplicate commits in one window fight in the pool instead of all
   landing). These two classes are ~89% of the month's 875,742 landed failures.
3. **Make chronic feeless failure expensive.** A failed feeless call should pay: charge the normal
   fee when a `Pays::No` call fails (post-dispatch repricing, the hook exists in
   `pallet_transaction_payment`). The six 100%-fail commitment bots and the `set_heap_pages` bot
   only exist because failure is free or near-free.
4. **One-shot state migration (~70 MB, cosmetic but cheap).** Purge: the nine dead pre-dTAO
   `SubtensorModule` items (26.8 MB), commitments untouched for 365 days (13,273 entries) and their
   orphaned `LastCommitment`/`UsedSpaceOf` rows, `RevealedCommitments` beyond the last reveal, the
   twelve V3 swap orphan items, `UsedWork` (1.8 MB), and drop `MAX_KEPT_PULSES` to one day (27 MB).
   None of it changes behavior for any live actor. And once the `AlphaV2` migration cursor finishes,
   verify the V1 maps get cleared: that is another ~120 MB that must not join this list.
5. **For node operators, today, no runtime change needed:** a non-archive validator with
   `--blocks-pruning 256` and state pruning drops nearly all of the 135 GB history; only archives
   need to carry the junk. Worth documenting officially given the growth rate.

## Appendix: reproduction

- Failure counts: index every block's extrinsics + `System.Events`, join `ExtrinsicFailed` by
  extrinsic index; quarterly buckets are `block / 648,000`. Reasons: `dispatch_error.Module.{index,
  error[0]}` through runtime 441 metadata (`SubtensorModule` = pallet index 7, `Commitments` = 18,
  `Swap` = 28).
- State census: for each pallet, `state_getKeysPaged(twox128(pallet), 1000, …, at)`; bucket keys by
  bytes 16..32 (`twox128(item)`); values via `state_queryStorageAt` in chunks. Orphan items are key
  buckets whose item hash matches no storage entry in the runtime metadata (identified here by
  brute-forcing candidate names from old pallet sources).
- Commitments detail: decode `CommitmentOf` values as `Registration { deposit: u64, block: u32,
  info }`; netuid from key bytes 32..34 (`Identity` hasher); ages against the snapshot block.
- Block sweep: `chain_getBlock` (raw body sizes) + `state_getStorage(System.Events)` (raw blob size)
  at sampled heights, MUV archive for history, OTF endpoints for state; every scan pinned to one
  explicit endpoint and one block hash.
- Live subnet set at snapshot: 129 (`NetworksAdded`), 27 with `SubnetEmissionEnabled = true`.
- One auxiliary pin: the Commitments decode ran at block 8,766,303, within an hour of the main
  snapshot; that storage drifts by far less than the rounding shown. `System.Account` was walked in
  full: 553,283 live keys, 56 B values (subtensor uses u64 balances). For contrast, the chain indexer
  knows 2.32M addresses that ever appeared in history: 4 in 5 addresses ever seen hold no account
  state today.

On-chain data, for information only. Addresses shown are public chain data; automation labels
("bot") are inferences from behavior (volume, failure rate), not identity claims.
