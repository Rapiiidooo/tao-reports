# Bittensor on-chain stats

Reproducible, no-nonsense deep-dives into the Bittensor (TAO) chain. Every number here is pulled
straight from mainnet and pinned to a block, so anyone can re-run it and check.

**Snapshot as of 2026-07-14, block 8,616,072.**

## Reports

| Report                    | What it covers                                                                                                   | Snapshot         |
|---------------------------|------------------------------------------------------------------------------------------------------------------|------------------|
| [Multisig](./2026-07-14-multi-sig.md) | Every multisig coldkey on the network: signer schemes (M-of-N), holdings, and which ones own or validate subnets | block ~8,616,000 |

More to come: validators, subnet flows, staking yield, whale moves.

## How the numbers are made

- Data is read live from Bittensor mainnet: Substrate storage (events, extrinsics, `System.Account`) and the SDK runtime
  APIs (`StakeInfoRuntimeApi`, subnet pools).
- Every report is pinned to a block height and a date. Chain state moves, so a re-run later will differ.
- Token values: alpha is priced through each subnet's swap pool. Two figures can appear:
    - **mark / spot**: alpha at the current pool rate, what a portfolio "shows".
    - **realizable**: after AMM slippage on exit, what you would actually get. Usually a few percent lower.
- τ = TAO. 1 τ = 1e9 RAO.

## Reproduce

Each report states the block it used and the storages or runtime calls behind every figure. Point any
archive node at the same block and you get the same numbers.

## Disclaimer

On-chain data, for information only. Not financial advice. Identities are self-declared on-chain
labels and can be wrong or impersonated.

## License

MIT
