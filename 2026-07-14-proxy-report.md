# Bittensor proxy stats

**Snapshot as of 2026-07-14, block 8,616,072.**

On-chain snapshot of the Substrate `Proxy` pallet: the live delegation graph (`Proxy.Proxies`) plus
`Proxy.Announcements`, valued at that block. The unit of analysis is the **delegation** (who may act
for whom) and the **delegate** (the controller), not the account.

## The proxy graph

| Metric                                        | Value |
|-----------------------------------------------|------:|
| Delegators (accounts that granted ≥1 proxy)   | 6,691 |
| Delegations (delegator → delegate edges)      | 7,706 |
| Unique delegates (controllers)                | 2,215 |
| Pending announcements (`Proxy.Announcements`) |     6 |
| Delegators that are also delegates            |   118 |

### Proxy types (by delegation)

Each grant is scoped to a `ProxyType`. `Exfiltration-capable` = the grant can move balance/stake to
the delegate (verified against the runtime `InstanceFilter`); the rest are operational only.

| ProxyType              | Delegations | Exfiltration-capable |
|------------------------|------------:|:--------------------:|
| Staking                |       6,258 |          —           |
| Any                    |       1,151 |         yes          |
| Transfer               |          78 |         yes          |
| Registration           |          75 |          —           |
| NonTransfer            |          68 |          —           |
| ChildKeys              |          30 |          —           |
| RootClaim              |          13 |          —           |
| Owner                  |          11 |          —           |
| SwapHotkey             |           9 |          —           |
| SudoUncheckedSetCode   |           3 |          —           |
| SmallTransfer          |           2 |         yes          |
| NonFungible            |           2 |          —           |
| NonCritical            |           2 |         yes          |
| Governance             |           2 |          —           |
| SubnetLeaseBeneficiary |           1 |          —           |
| RootWeights            |           1 |          —           |
| **TOTAL**              |   **7,706** |                      |

### Delegations per delegator

| Proxies on the account | Delegators |
|-----------------------:|-----------:|
|                      1 |      6,368 |
|                      2 |        227 |
|                      3 |         26 |
|                      4 |         10 |
|                      5 |          3 |
|                      6 |          4 |
|                      7 |          3 |
|                      8 |          6 |
|                      9 |          2 |
|                     10 |          1 |
|                     11 |         21 |
|                     19 |          1 |
|                     20 |         19 |

## Value under delegation (mark / spot, block 8,616,072)

`Total τ` = free + root stake + alpha at pool price. A delegation's *risk* is the value the delegate
could move out: `Any`/`Transfer`/`SmallTransfer`/`NonCritical` can exfiltrate; `Staking`/`NonTransfer`
cannot (a `Staking` proxy can stake/unstake/move, but freed TAO returns to the delegator's coldkey).

- **Total τ held by the 6,691 delegators : 807,023 τ** spot (≈ 781,120 τ realizable)
- **669,864 τ (83%) sits under `Staking`-only grants** — cannot be exfiltrated by the delegate
- Under any **exfiltration-capable** grant : **63,438 τ (8%)**, of which `Any` (unrestricted) : 51,294 τ
- Concentration: the single largest delegator (`5GH2aUTMRU…`) holds 280,137 τ ≈ 35% of the total
- Delegators > 1 τ : 4,697 (>10 τ : 2,358, >100 τ : 617, >1,000 τ : 86)

## Delegates (controllers) — who can act for the most accounts

`Any`/`Stk` = grants of that type; `τ reach` = spot value across controlled accounts; `τ exfil` =
value under an exfiltration-capable grant (what this delegate could actually move out).

|  # | Delegate                                           | Controls | Any |  Stk | τ reach | τ exfil | Identity |
|---:|----------------------------------------------------|---------:|----:|-----:|--------:|--------:|----------|
|  1 | `5HdbS6JuLrQ7uemx7QXN39mn5VBita8nV5txiiYoFVpu5nfg` |    1,551 |   0 | 1551 |  53,143 |    0.00 |          |
|  2 | `5DCUcSiqRU6QG52JtvqKrzxWRSYgoqupqc7CygkmMRMUrRZx` |      704 |   0 |  704 |  63,551 |    0.00 |          |
|  3 | `5EUMfBTTwTDMgxGCyY893VYe8UYgWR24bqQ56L5GLiZCKgqq` |      413 |   0 |  413 |  11,904 |    0.00 |          |
|  4 | `5CShWfpLMPHoT56bQgP9LaiMizWmWWj1JSHeB73zS7LjRtCb` |      262 |   0 |  262 |   1,744 |    0.00 |          |
|  5 | `5GbihDjoxwkLFH9oN8GJkHGhSe3FBNLKTxyd4SiFf9y8eNmZ` |      256 |   0 |  256 |  13,462 |    0.00 |          |
|  6 | `5FkDiB5cDmxUHBgsmtTUKGXT1qG3dGptp68KjGXTNRrSbvx6` |      167 |   0 |  167 |   7,331 |    0.00 |          |
|  7 | `5FcUWMfVRaoGc4r6vat9EKaUqqqajJi5JmJjo1T9eYYLn7SH` |      152 |   0 |  152 |   2,295 |    0.00 |          |
|  8 | `5CeJG2T47NxUAAc42q2zoU7qV1YFy4khL3ogHxooVjNKxUuw` |      151 |   3 |  151 |   9,704 |     164 |          |
|  9 | `5CXBTec7rYvH96M9aiDcBirBvJkYVYxAn4EqQNydJEkPNGCq` |      106 |   0 |  106 |     798 |    0.00 |          |
| 10 | `5GH1zeBbAeSLAtqSKPLzZgbYvv54L2YznLbmtJm6F2zGRtFM` |      105 |   0 |  105 |   2,217 |    0.00 |          |
| 11 | `5GE28n4BhNv9Nmo6AgwUxkQxtjjWwudeucMY9TPSVdF1YyC2` |       92 |   0 |   92 |     393 |    0.00 |          |
| 12 | `5EjRoPXHprnKsvF81PoyWmRcU5nqifyxjM9FmgErjsmKwrxD` |       81 |   0 |   81 |     820 |    0.00 |          |
| 13 | `5Gh8iu8W5MzkBXD5XtaP7UCDgxKFANRCb4ZLSJsRAnDBjAAR` |       72 |   0 |   72 |   1,366 |    0.00 |          |
| 14 | `5H9YSpwymn8ayjUwZQ7NDUSJiA3JsyYusgVY6bckCz7Vi78h` |       61 |   0 |   61 |  20,353 |    0.00 |          |
| 15 | `5Gdd5wKCs3xJGKT1wstr8Nu4SYuTa8TYqRYqxk8VhZ4k2dTq` |       55 |  54 |    1 |      12 |       9 |          |
| 16 | `5EJT9T8Gyx2fA8p4Hc8P2NXadmbkE3QwfaWL9udHNEfcqtw3` |       47 |   0 |   47 |     403 |    0.00 |          |
| 17 | `5Die7iLHdk4SLkmSYTydqmCGAR5WZmyJJEWLVy3yun76QvYY` |       46 |  46 |    0 |   1,598 |   1,598 |          |
| 18 | `5GteWuXtev1ETzKg4Dm3q7GndEtJ9kUq5nMaMzAzpskQV7qF` |       45 |   0 |   45 |   4,511 |    0.00 |          |
| 19 | `5F9rcrUBBDB3cuQMQRnchiprKmwRBNZRPyuE7nEyfmRxxMN8` |       43 |   0 |   43 |     196 |    0.00 |          |
| 20 | `5GjLw8npo4iHNsyJLUdBStyW3DAkqr8CxCi6JTnUGY2WRCG6` |       43 |   0 |   43 |     149 |    0.00 |          |
| 21 | `5DyGP1DhWyg4vqxBRK4WcurKhVr2sLvrk488zwpdAX1pcCXr` |       27 |   0 |   27 |     788 |    0.00 |          |
| 22 | `5GGpH7rhdeMYhFMR4spvtYxim86toT1ecfNJSZUsvcLESCiH` |       26 |   0 |   26 |     472 |    0.00 |          |
| 23 | `5E4G7WtGLD71PzRg4MavzLh6kkC1WNhihmXwrqpS7nP55w7E` |       25 |   0 |   25 |     164 |    0.00 |          |
| 24 | `5DkgPz2gNhaTLvUzVKyGyv8r6jSE9GuYDsZfcHQyFMqKCnPT` |       24 |   0 |   24 |   3,420 |    0.00 |          |
| 25 | `5F9bvpMJ3mzx7yLKdqdQzwsQbkcAKpc27zih8i5Et5m8Pz6h` |       23 |   0 |   23 |     126 |    0.00 |          |
| 26 | `5CiuGG5SYi4tkZRRHSBkDe85S38dEerofhBhohvFDsGCTYJh` |       22 |   0 |   22 |     607 |    0.00 |          |
| 27 | `5ECcQsHSBLxwiPKsFWmPa4TbM4xRRTbqGZwxvz24S3Z3E8nK` |       22 |  22 |    0 |    0.85 |    0.85 |          |
| 28 | `5DCTc3fkxweigbX4P789tVaDvCJfvCQy2ZPB4SZTcjDTSHW1` |       21 |  21 |    0 |    0.81 |    0.81 |          |
| 29 | `5G9mRiHgD5Zms4eWWW45KujeCbPr8EVSh5U6qHyo2jXzLTS2` |       20 |  20 |    0 |       2 |       2 |          |
| 30 | `5HdonKFAPRVFd2dBWvp1Y4zFipBNQJDzZaQAgv9xZCpnWTS1` |       20 |  20 |    0 |       2 |       2 |          |
| 31 | `5CczynUnQMe2cjQnpff99e4gtjJhM3UvzZiMMwXNnvEXPTS4` |       20 |  20 |    0 |       2 |       2 |          |
| 32 | `5GYgMTeo9LdufufVUyxPZpaESnFdpqvQujBEhWhBNCYbYTS8` |       20 |  20 |    0 |       2 |       2 |          |
| 33 | `5ES685kPiVX3zyviNsxiyhC7NfE7NjYPM86FjSsoSWGm1TS9` |       20 |  20 |    0 |       2 |       2 |          |
| 34 | `5CrXNfLqhX4PMN3ES1ZP9PihAFDZSgL7DVhd4UZFJQSTcTS5` |       20 |  20 |    0 |       2 |       2 |          |
| 35 | `5DiH9pEdYRYsdQNSzLLVXERrG8MmeCtmTpuKEuuYWu1nzTS3` |       20 |  20 |    0 |       2 |       2 |          |
| 36 | `5GuUrh3wMtBsiTsvWcJqDN5wZKhtREJUHjSE6wQNVVLuhTS7` |       20 |  20 |    0 |       2 |       2 |          |
| 37 | `5F7WawuFZiWvDq1cxTQd3PGzGRQo8uVUGyswfgC2uQkg9TS6` |       20 |  20 |    0 |       2 |       2 |          |
| 38 | `5HpmG8cL27D3w1mRXzBBwVgj6JUySBti6gKkMnYSmRCCtVqc` |       20 |   0 |   20 |     131 |    0.00 |          |
| 39 | `5C8M6nXLgFzsK68XnyJ4DSDrkcsESqWdsDjxKLYs3r5wTZbo` |       20 |   0 |    0 |     104 |     104 |          |
| 40 | `5G1sSMNG16pmQprVZe3R8kkAzhcJemVsknNp16oEKzDXrPZw` |       18 |   4 |   14 |    0.03 |    0.00 |          |

### Delegates ranked by exfiltration-capable value under control

|  # | Delegate                                           | τ exfil | Controls | Identity |
|---:|----------------------------------------------------|--------:|---------:|----------|
|  1 | `5Cq17EDWE9X7JfHMqc9eMStzEMhaAuWiMWyrGp41U5jx2SMV` |  11,521 |        1 |          |
|  2 | `5GLq32tv9Rxy3ngAaCyjmKbeukNPfh27p5ERWaAmYG6ZAVnn` |  11,521 |        1 |          |
|  3 | `5DbjRLaeWTWtdmm3fA64KgdfWHKPLywm25YjErvK7k2V1Agn` |  10,089 |        3 |          |
|  4 | `5FTw4dySS1GfyQKepgTWEgWHHLMovvk8LkKTF9t7UTdoCMRc` |   8,121 |        1 |          |
|  5 | `5GSp7EAQGkAkeZk296HpRwkpm7LjyzqjBTV91VNQh4RFCxCg` |   4,693 |        4 |          |
|  6 | `5CetDUVjiBGfp1awxbMe9sDH27uBTyA2ywnrSnjnh8ov77uy` |   4,419 |        1 |          |
|  7 | `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` |   4,255 |        2 |          |
|  8 | `5CCAyUnSU4qntZckfimBb9mbGAXxxeQEBDYG5KXVQJj7ghaT` |   3,874 |        1 |          |
|  9 | `5HUBTun3H7uRohUzgpXj5BxKT6TebTwHryjx6bm3acGWFUDN` |   2,589 |        1 |          |
| 10 | `5H7x9rhGUY8dkjz4x3iEJcWzUxxceWagdtxn3SsGpLjpwrqV` |   2,333 |        1 |          |
| 11 | `5DXbsJWxB715pouXw3fG5SWXdAsyZvbucy6BXeVgLDhETsN6` |   1,725 |        1 |          |
| 12 | `5HVaVVen5K5YJSp3cMUtjW8JMkMkpkiim7JaYZJgiyhWyURi` |   1,650 |        1 |          |
| 13 | `5Die7iLHdk4SLkmSYTydqmCGAR5WZmyJJEWLVy3yun76QvYY` |   1,598 |       46 |          |
| 14 | `5DRzkY2DK66Zt3GbhNkhvKcfEgnogo4ATiPzMxLPutddMc1g` |   1,348 |        1 |          |
| 15 | `5CP6HGu6ZcDnofVvwaMriPijupj8abFoyAiYtw3nmNNiptDG` |   1,027 |        1 |          |
| 16 | `5HMM1yDZTcejZkNWx3swfLvV6bHVv5RQAwWMBtvJD4YHHdJb` |     977 |        1 |          |
| 17 | `5EJqwJFABMiAEEfB2PCPgae9fVLdqNnLa3x5ZgpgwjTC7jpM` |     907 |        1 |          |
| 18 | `5HK82QjPedUJjSoaCrAFE8JCZt7FA757VkN9ccRzMMirE9cS` |     856 |        1 |          |
| 19 | `5FNiBVce6Eqgsyf1zg5BojjfdSDAz6KB1zw6SYgaQ8YMY5sE` |     780 |        1 |          |
| 20 | `5FgV1ApXXk2jJRxtARCBUetqW6xtyKWqvtsG7JM7g2TzngDd` |     597 |        2 |          |

## Delegators — the largest accounts under proxy

`Keyless` = `pure` (anonymous pure proxy) or `ms` (multisig derived account), both nonce 0.

|  # | Delegator                                          | Total τ | Free τ | α sn | Types                                                     | Keyless | Identity              |
|---:|----------------------------------------------------|--------:|-------:|-----:|-----------------------------------------------------------|:-------:|-----------------------|
|  1 | `5GH2aUTMRUh1RprCgH4x3tRyCaKeUi5BfmYCfs1NARA8R54n` | 280,137 |  6,637 |  128 | Staking                                                   |         |                       |
|  2 | `5EBuUXD6eXSSWVaT1NqaUQoAACUkAmEogzAfPQvDXTEQZ8Ff` |  37,650 |    525 |  127 | NonFungible,Registration,SwapHotkey                       |         | Openτensor Foundaτion |
|  3 | `5EFZf5pnTqLegv6gxCrb6TKBQBGz9xLJNK8x9eR273cSons6` |  21,865 |      2 |   91 | Staking                                                   |         |                       |
|  4 | `5FFQTAVQvnGtCrd6SXUe2gRhxFq8ycC523HdCHcjwDweFBkv` |  16,628 |    985 |    9 | Staking                                                   |         |                       |
|  5 | `5HiFDVNX4ivCJFt9RvgRCtQKmAPgAXGX8BRgX3XKqfY9fFve` |  11,521 |   0.02 |  125 | Any,ChildKeys,Registration,RootWeights,Staking,SwapHotkey |   ms    | TAO.app               |
|  6 | `5Ehk4sfD6EJfQuxHfGu8ZY1akU9CYqwok98WT1Cp5K11RGUy` |   9,923 |   0.48 |    2 | Staking                                                   |   ms    |                       |
|  7 | `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` |   8,121 |   0.35 |    2 | Any,Staking                                               |   ms    |                       |
|  8 | `5H1zcoQVXgWy9gdhHTzcjx5GmkiGFNsDHkY35XjMGcerk3vX` |   7,701 |      3 |    3 | Staking                                                   |         |                       |
|  9 | `5CtdXWd8CkvCJnjjXgZ5WodVpZxdBEpSgFjdqzgVAKLJm9bs` |   7,623 |   0.15 |   10 | Staking                                                   |         |                       |
| 10 | `5GfSJPS4majhCnhrJKTUcAbUfAgCHFAsN9nqxkhYsxkqWN8z` |   7,075 |   0.35 |      | Staking                                                   |         |                       |
| 11 | `5EbY1SebUuz189w1G96JN4mmHnShAECA6JefCRYhSLqyYg2c` |   6,559 |   0.18 |    2 | Owner                                                     |         | Bitcast               |
| 12 | `5DD2Eyit1WQRpWm6rudoZooLh8cFWsAXYtmcXeBZXoZAJk3V` |   6,461 |   0.35 |   20 | Staking                                                   |         |                       |
| 13 | `5F9LSSxrEjTmnWtUNAj4tCHBQTwVgc4EjDFoSe9BeD3fN9Yi` |   6,322 |   0.57 |   15 | Staking                                                   |         |                       |
| 14 | `5FjP8fFbMb6pEHfRZyRegsmwRUMDuLFNcaBu1TerWMPEKgNh` |   6,296 |    164 |    2 | Staking                                                   |         |                       |
| 15 | `5DqF59qqGVy1gi2aRXL6J6YXtPLVDmeD8AihU492f9i6qjKX` |   6,008 |   0.43 |   19 | Staking                                                   |         |                       |
| 16 | `5GjoDduJujeACNfPDRVBBpdEqwRRY8vAygUWjQNwwDAn5EE9` |   5,630 |   0.11 |   18 | Staking                                                   |         |                       |
| 17 | `5FNxL46parYKx4yPZcihEWuAFYtGE92SLgWStsHDjodanPSG` |   5,397 |   0.46 |   92 | Transfer                                                  |         |                       |
| 18 | `5FEfTh8SCMNi2ifFiQ18GKRFD6T5TUMtK5pxAxD11jz8NWu6` |   4,904 |     23 |   49 | Staking                                                   |         |                       |
| 19 | `5Gs88NDAZEmQJeHVFdseLzJm9K7VANwEpjoxg8AQ1kmh6mCC` |   4,867 |      2 |   38 | Staking                                                   |         |                       |
| 20 | `5DAQpczEK4vzBn1waHkC4BZGqGPZ1dwPxKVsj36JDofHAw3a` |   4,692 |   0.14 |    1 | Transfer                                                  |         | Hippius               |
| 21 | `5Eo1Fr1VuB39XjcEyYUcr9UXYpumLKjVF2JwLp3beyGdZcCT` |   4,419 |      4 |  128 | Any                                                       |  pure   |                       |
| 22 | `5H19DCvEos5rxH7mzbfv2yR1HLvJXQxt9phoz2p4CkkP55bL` |   4,138 |   0.00 |    1 | Any                                                       |  pure   |                       |
| 23 | `5CV3g8UsmfHZiE9XVSzGvzrSribaPFphPxbufdS7Xvhwt7cX` |   3,874 |     70 |    2 | Any,Staking                                               |  pure   |                       |
| 24 | `5EWuFGywpHmVVx9V1zhAf8qTrWTXguPEDXz3jgMLjmWXkhaV` |   3,853 |   0.78 |    2 | Staking                                                   |         |                       |
| 25 | `5FHrQMjzzAhmL5zS9ys87ZrGCwG3vsVT9hXAUWZQ8SNdRqig` |   3,529 |      2 |    1 | NonTransfer                                               |         | SOMA                  |
| 26 | `5FnumZwTeKy35oxdQw3jNKjKCkgQ8jZ4F8E7bwY6bP3ncTZP` |   3,448 |     50 |   19 | Staking                                                   |         |                       |
| 27 | `5H3XwzydgE2XUGoJCR4dSj7tkd7uxZDJqik69hux2DBcruom` |   3,404 |   0.52 |   23 | Staking                                                   |         |                       |
| 28 | `5D5S2wsVHaUjshUJnR7vE5x3oWhKMAF22Jr8TddBU9Yu5QsR` |   3,324 |   0.00 |   53 | Staking                                                   |         |                       |
| 29 | `5DwQnZoPF7rLJYESKmVKrpn7fWDXkoifFSBKUsxSP2pQNpKP` |   3,217 |   0.87 |      | RootClaim,Staking                                         |         |                       |
| 30 | `5EhxzAXgTCp4cTz5yVgtPbUqpW9hYkiVuZrhXonDGN4LfAh1` |   3,046 |     11 |   49 | RootClaim,Staking                                         |         |                       |
| 31 | `5Eq8b9p6zJMjEXyH9sX4DRMYspnUyorEKq3Zmha1WN6AC4sf` |   2,785 |    254 |  128 | NonTransfer                                               |         | Crucible Labs         |
| 32 | `5DhaScwytZJULWzNYTPeFho1BTPTi5LJgk3kuQUAyPQ3fDfw` |   2,721 |   0.35 |      | Staking                                                   |         |                       |
| 33 | `5HCkg3pPdgfrp5ioQ91hz1XEooYpgVwV91qNjSmKBwqMpTwx` |   2,679 |   0.39 |   18 | Staking                                                   |         |                       |
| 34 | `5Fazr9w27pKyQeR7BqWNQ98hv1n7YpWrjfPdLPJsEWiA6r4W` |   2,589 |      1 |    5 | Any                                                       |  pure   |                       |
| 35 | `5Gn45wbQZyLPZjCqBVKDwxahRS2EKLu6A3kKSwb8SeXTjQEM` |   2,543 |   0.35 |   15 | Staking                                                   |         |                       |
| 36 | `5EnBBVUAkzbiU7saXbadftBs1NMgMHBQcjhQ9viabQSRcdfN` |   2,405 |   0.62 |    2 | Staking                                                   |         |                       |
| 37 | `5GeqQa2GsGBhSX2p2q55hCAqG9H7iudfgapCcQbByMxbu6zc` |   2,333 |   0.50 |    2 | Any                                                       |  pure   |                       |
| 38 | `5H6JVDicq9HGt1ibkSUcCH1hj6y5YK98x9mMBQx4KpP4rJ59` |   2,265 |  1,005 |    7 | NonTransfer                                               |         |                       |
| 39 | `5Cxr8ayZY1gBBbXxSD8ds9HTZeg31woWmWFD1DF4krXJKkJ2` |   2,240 |   0.32 |    4 | Staking                                                   |         |                       |
| 40 | `5HRCtchrzcW8fmUsrWdTxDtkvtMwSrvKLtKhsEQfyEERGydu` |   2,191 |    111 |   51 | Staking                                                   |         |                       |
| 41 | `5F85xjhaRjSjk3qX5DPPav2S6ouVrxXANaFpaznz47iPPYgb` |   2,186 |      2 |    9 | Staking                                                   |   ms    |                       |
| 42 | `5DqTA5c8kRxfSi9XYZPRD1X2S7YkWfGTTUAvZGWQWudgtYAu` |   2,088 |   0.20 |   18 | Staking                                                   |         |                       |
| 43 | `5GEczqHddRzLSUNqpL4gekicYa5s6TXtoFDbph2QLMSMQVSY` |   1,973 |      4 |   50 | Staking                                                   |         |                       |
| 44 | `5FCSevLkofmKZRixMawp6jyyjBty1AeSCLa7N5Fv892DYkXX` |   1,925 |   0.03 |    2 | Staking                                                   |         | Sportstensor          |
| 45 | `5GBr8ma68mXeXF9GNGVTDDjLrmXB5C2JP9ieS6XEYt4SxQDn` |   1,851 |      8 |   62 | RootClaim,Staking                                         |         |                       |

## Pure proxies (keyless treasuries)

A pure proxy (`Proxy.create_pure`) is a keyless account driven only by its delegates. Multisig
accounts are also keyless (nonce 0); the two are split by cross-referencing known multisigs.

- Keyless delegators : 227 → **137 pure proxies** + 90 multisig accounts
- Pure-proxy combined value : **26,602 τ** spot
- **Pure proxies controlled by a multisig delegate : 49** (keyless treasury under M-of-N), worth 25,486 τ

|  # | Pure proxy                                         | Total τ | Free τ | Controlled by (delegate)                           | Type | ms? | Delegate identity |
|---:|----------------------------------------------------|--------:|-------:|----------------------------------------------------|------|:---:|-------------------|
|  1 | `5Eo1Fr1VuB39XjcEyYUcr9UXYpumLKjVF2JwLp3beyGdZcCT` |   4,419 |      4 | `5CetDUVjiBGfp1awxbMe9sDH27uBTyA2ywnrSnjnh8ov77uy` | Any  | yes |                   |
|  2 | `5H19DCvEos5rxH7mzbfv2yR1HLvJXQxt9phoz2p4CkkP55bL` |   4,138 |   0.00 | `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` | Any  | yes |                   |
|  3 | `5CV3g8UsmfHZiE9XVSzGvzrSribaPFphPxbufdS7Xvhwt7cX` |   3,874 |     70 | `5CCAyUnSU4qntZckfimBb9mbGAXxxeQEBDYG5KXVQJj7ghaT` | Any  | yes |                   |
|  4 | `5Fazr9w27pKyQeR7BqWNQ98hv1n7YpWrjfPdLPJsEWiA6r4W` |   2,589 |      1 | `5HUBTun3H7uRohUzgpXj5BxKT6TebTwHryjx6bm3acGWFUDN` | Any  | yes |                   |
|  5 | `5GeqQa2GsGBhSX2p2q55hCAqG9H7iudfgapCcQbByMxbu6zc` |   2,333 |   0.50 | `5H7x9rhGUY8dkjz4x3iEJcWzUxxceWagdtxn3SsGpLjpwrqV` | Any  | yes |                   |
|  6 | `5DhsVYewpCdQQUenHU52k5Cys1WsTWTt5V5m84D3n4L8FWDS` |   1,650 |   0.01 | `5HVaVVen5K5YJSp3cMUtjW8JMkMkpkiim7JaYZJgiyhWyURi` | Any  | yes |                   |
|  7 | `5HLBDbdKfPCPKW33sPPyut8dPRTXA413Yp4ZRBgVKfrk4PcD` |   1,348 |   0.03 | `5DRzkY2DK66Zt3GbhNkhvKcfEgnogo4ATiPzMxLPutddMc1g` | Any  | yes |                   |
|  8 | `5EWDWVFhwDHW3FmC3NSv9X3xzQLqN6ymaA6bGGfkGaYN2Pxh` |   1,027 |      7 | `5CP6HGu6ZcDnofVvwaMriPijupj8abFoyAiYtw3nmNNiptDG` | Any  | yes |                   |
|  9 | `5D1VXeeSdrfyrBdMe4SNwKnRsmzrjXES9dhx6kQkCHhJUPvS` |     907 |   0.25 | `5EJqwJFABMiAEEfB2PCPgae9fVLdqNnLa3x5ZgpgwjTC7jpM` | Any  | yes |                   |
| 10 | `5G6Qi4Z4VCi4DTXRZEH4HPs9XisVP7jPxvV771znEBp5hdJ3` |     856 |   0.09 | `5HK82QjPedUJjSoaCrAFE8JCZt7FA757VkN9ccRzMMirE9cS` | Any  |     |                   |
| 11 | `5D4BUpK2jf5ddLkhugMUPGHEeBQ2udWddypbks6RGpo3WFVw` |     780 |      3 | `5FNiBVce6Eqgsyf1zg5BojjfdSDAz6KB1zw6SYgaQ8YMY5sE` | Any  | yes |                   |
| 12 | `5EoYmDFszJEJ4rSewPLSmMyqU2kPveJg8AVHLYJnwjb6Gqtq` |     515 |      2 | `5FgV1ApXXk2jJRxtARCBUetqW6xtyKWqvtsG7JM7g2TzngDd` | Any  | yes |                   |
| 13 | `5EHcg2k2YxNiVed79n8CTjBhorVETUBjNiGh3iVSXGNdrvEp` |     505 |   0.50 | `5FqkimrwfRS8GVtdpDXUigGdE2X6Nc13fu7L3CBJzRWq3zrY` | Any  | yes |                   |
| 14 | `5DzsVV2L4M9r4uWoyarzPyhfeCv6DDAEs5rM2bpHjmerPcGa` |     278 |    278 | `5Fh8x6z748apzJSWxhTDFy9gt7C7YjoDVCNZQ3kabLtpRmWs` | Any  | yes |                   |
| 15 | `5GxEEHxLWpvcAGghL8LG7xNjrxZk4SEPW1VT43hGPZ2TP9mZ` |     269 |      1 | `5H3EPMRdMjNdSuGM3vN1zCf7m1tjEk16iApnWwbRdxvpNVWz` | Any  | yes |                   |
| 16 | `5CAc19iETJmWD2rYVX1ht58ghCpyHq86MoBNdx5TzLfinzcx` |     266 |      1 | `5GmP9xSsv1snGRTd8dVpHX7tRsj7HmyViUvaSPtACioZhJXG` | Any  | yes |                   |
| 17 | `5GGj3HU9CukgsZyy8aL6oTYjZLZM8FowH9UDq7Z3BbUz3UhZ` |     228 |   0.64 | `5EwNT4Qivtpz1ki139GhMFvRfchkGHf3LSMKN6RCt8trZbzZ` | Any  | yes |                   |
| 18 | `5GGQ4s1aK5uYcYu4dosbo6re1c9zoxMbo77HoaLXLSSwbqqA` |     223 |      1 | `5FxzokBqz9MzLnV9CjynzpxFE4eHnV3p9k4X4zGHGyd44Zkr` | Any  |     |                   |
| 19 | `5GCbpM77isMVgZeqrGSTae4pbWL2bDHffEnXq3ZapDvVyRUF` |     122 |     88 | `5F8LnZbyeUji8BAuuiguxawhyvLtPRvBzWgWnzRAfnXu218L` | Any  | yes |                   |
| 20 | `5GurNtB3yQFCh6CSmfH7LrYJDsvzup4diHZiaYtKe274nrMX` |     117 |   0.51 | `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` | Any  | yes |                   |

## Multisig ∩ proxy

Cross-referenced against 325 known multisig coldkeys.

- Multisigs that **granted** a proxy (delegators) : 90
- Multisigs that **hold** a proxy (delegates) : 51
- Accounts controlled by a multisig delegate : 57 (spot value 56,833 τ)
- Of those, **pure proxies controlled by a multisig** (keyless M-of-N treasury) : 49

### Accounts a multisig controls via proxy

| Multisig (delegate)                                | Controlled account                                 | Pure |      τ | Types                |
|----------------------------------------------------|----------------------------------------------------|:----:|-------:|----------------------|
| `5Cq17EDWE9X7JfHMqc9eMStzEMhaAuWiMWyrGp41U5jx2SMV` | `5HiFDVNX4ivCJFt9RvgRCtQKmAPgAXGX8BRgX3XKqfY9fFve` | yes  | 11,521 | Any                  |
| `5GLq32tv9Rxy3ngAaCyjmKbeukNPfh27p5ERWaAmYG6ZAVnn` | `5HiFDVNX4ivCJFt9RvgRCtQKmAPgAXGX8BRgX3XKqfY9fFve` | yes  | 11,521 | Any                  |
| `5FTw4dySS1GfyQKepgTWEgWHHLMovvk8LkKTF9t7UTdoCMRc` | `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` | yes  |  8,121 | Any                  |
| `5CetDUVjiBGfp1awxbMe9sDH27uBTyA2ywnrSnjnh8ov77uy` | `5Eo1Fr1VuB39XjcEyYUcr9UXYpumLKjVF2JwLp3beyGdZcCT` | yes  |  4,419 | Any                  |
| `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` | `5H19DCvEos5rxH7mzbfv2yR1HLvJXQxt9phoz2p4CkkP55bL` | yes  |  4,138 | Any                  |
| `5CCAyUnSU4qntZckfimBb9mbGAXxxeQEBDYG5KXVQJj7ghaT` | `5CV3g8UsmfHZiE9XVSzGvzrSribaPFphPxbufdS7Xvhwt7cX` | yes  |  3,874 | Any                  |
| `5HUBTun3H7uRohUzgpXj5BxKT6TebTwHryjx6bm3acGWFUDN` | `5Fazr9w27pKyQeR7BqWNQ98hv1n7YpWrjfPdLPJsEWiA6r4W` | yes  |  2,589 | Any                  |
| `5H7x9rhGUY8dkjz4x3iEJcWzUxxceWagdtxn3SsGpLjpwrqV` | `5GeqQa2GsGBhSX2p2q55hCAqG9H7iudfgapCcQbByMxbu6zc` | yes  |  2,333 | Any                  |
| `5HVaVVen5K5YJSp3cMUtjW8JMkMkpkiim7JaYZJgiyhWyURi` | `5DhsVYewpCdQQUenHU52k5Cys1WsTWTt5V5m84D3n4L8FWDS` | yes  |  1,650 | Any                  |
| `5DRzkY2DK66Zt3GbhNkhvKcfEgnogo4ATiPzMxLPutddMc1g` | `5HLBDbdKfPCPKW33sPPyut8dPRTXA413Yp4ZRBgVKfrk4PcD` | yes  |  1,348 | Any                  |
| `5CP6HGu6ZcDnofVvwaMriPijupj8abFoyAiYtw3nmNNiptDG` | `5EWDWVFhwDHW3FmC3NSv9X3xzQLqN6ymaA6bGGfkGaYN2Pxh` | yes  |  1,027 | Any                  |
| `5EJqwJFABMiAEEfB2PCPgae9fVLdqNnLa3x5ZgpgwjTC7jpM` | `5D1VXeeSdrfyrBdMe4SNwKnRsmzrjXES9dhx6kQkCHhJUPvS` | yes  |    907 | Any                  |
| `5FNiBVce6Eqgsyf1zg5BojjfdSDAz6KB1zw6SYgaQ8YMY5sE` | `5D4BUpK2jf5ddLkhugMUPGHEeBQ2udWddypbks6RGpo3WFVw` | yes  |    780 | Any                  |
| `5FgV1ApXXk2jJRxtARCBUetqW6xtyKWqvtsG7JM7g2TzngDd` | `5EoYmDFszJEJ4rSewPLSmMyqU2kPveJg8AVHLYJnwjb6Gqtq` | yes  |    515 | Any                  |
| `5FqkimrwfRS8GVtdpDXUigGdE2X6Nc13fu7L3CBJzRWq3zrY` | `5EHcg2k2YxNiVed79n8CTjBhorVETUBjNiGh3iVSXGNdrvEp` | yes  |    505 | Any                  |
| `5Fh8x6z748apzJSWxhTDFy9gt7C7YjoDVCNZQ3kabLtpRmWs` | `5DzsVV2L4M9r4uWoyarzPyhfeCv6DDAEs5rM2bpHjmerPcGa` | yes  |    278 | Any                  |
| `5H3EPMRdMjNdSuGM3vN1zCf7m1tjEk16iApnWwbRdxvpNVWz` | `5GxEEHxLWpvcAGghL8LG7xNjrxZk4SEPW1VT43hGPZ2TP9mZ` | yes  |    269 | Any                  |
| `5GmP9xSsv1snGRTd8dVpHX7tRsj7HmyViUvaSPtACioZhJXG` | `5CAc19iETJmWD2rYVX1ht58ghCpyHq86MoBNdx5TzLfinzcx` | yes  |    266 | Any                  |
| `5EwNT4Qivtpz1ki139GhMFvRfchkGHf3LSMKN6RCt8trZbzZ` | `5GGj3HU9CukgsZyy8aL6oTYjZLZM8FowH9UDq7Z3BbUz3UhZ` | yes  |    228 | Any                  |
| `5ExK3SD44TxcabvhsZbDY8hmf8jGHpPLvmxbXbNbMwwPyb9i` | `5E3CpnLgSMb7ggjLpqoKgPPyCmiqgwLfhozeDGo1qQH1rJWt` | yes  |    164 | Any                  |
| `5F8LnZbyeUji8BAuuiguxawhyvLtPRvBzWgWnzRAfnXu218L` | `5GCbpM77isMVgZeqrGSTae4pbWL2bDHffEnXq3ZapDvVyRUF` | yes  |    122 | Any                  |
| `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` | `5GurNtB3yQFCh6CSmfH7LrYJDsvzup4diHZiaYtKe274nrMX` | yes  |    117 | Any                  |
| `5FgV1ApXXk2jJRxtARCBUetqW6xtyKWqvtsG7JM7g2TzngDd` | `5DhnovbWPnbpDPwwmG4hf66YxECnnAbutd2dVdCmUBK5dSrr` | yes  |     82 | Any                  |
| `5Fh8x6z748apzJSWxhTDFy9gt7C7YjoDVCNZQ3kabLtpRmWs` | `5HquQNGA3Rgbtf26v7G4posZHRmnq8h7KFUREF1yJ56jRZH9` | yes  |     25 | Any                  |
| `5DtCHmzx8CovxTzhUvHakmaTZ5ph8DNpigL5JoVShaXDvHYU` | `5ExHnok2jC8BiqRtcStAxwfQZXqMGyo5LbXArUpNitPV8xXN` | yes  |     20 | SudoUncheckedSetCode |
| `5EvKDyLsDu2vXfeGDpmvgGX7fs1LMCkop5AFXYJqp8gY14RG` | `5GhyFXnwVWBEu8BTLpyNGx75cQ9guxz25FDT1JCZPs27vXEY` | yes  |      6 | Any                  |
| `5C5AMwdJcsRvdbekbg3QRNQ1Gna3FbTYd8knk2qkivR5BJNt` | `5Cck8aSY2t8B4se1KaHdCCbaj1NCTBg6eVHGSEN4dkwv44Vr` | yes  |      2 | Any                  |
| `5Dbv5uRa88EdqAr5yV5VN2eo8E7nF8wCH5KsTXXvM2qi9uqC` | `5DQwjxr5gTbgZARUoSDyKgB6hgbKBFcWMNVrAziXzLpyabsu` | yes  |      2 | Any                  |
| `5CpBADxtSgPAzmhWjZT78Lk6onCykEvfeboXMASAyfFdivAW` | `5E5KjtkytEm6LfhyTDSktpbHZpUxXppcmazpg8p8YmNXZkLX` | yes  |   0.93 | Any                  |
| `5DkPGJJyBvPWdv64Pgz8VCpvJ9pBNv1cRcFqEaiRZnWzM92w` | `5C5W3LJDNX2fy1sqaQ3FW1USjZQkVVA8QmzNZbGyL5CvA3pc` | yes  |   0.55 | Any                  |
| `5Hg4p59KZDkRkgyeXANXbRSTDG6zWfYYgDa4fzXHDLZ62QWT` | `5HggEL7dM2cqJTwSNsNGhoYVySG8HJkSVzZ2kPAjTLXSACG8` | yes  |   0.53 | Any                  |
| `5CAexBeAAHZdqKtv3gXYPwr5FNwpLwNJ9pZk6VePwnuMzJjd` | `5Do5ZAohByLfHA5hHZjUS4tVgCQG7dV9WjDtkvVTHiUsEwtd` | yes  |   0.27 | Any                  |
| `5Go71GRzZYwYMWh97iShdfyufuuzUKUsLf8HHfYPoKf7qwMf` | `5FffQmxw18EJumbtezCZNjePNJpiMh6U6QiR6cNKaJ3J58Jm` | yes  |   0.17 | Any                  |
| `5DeMSAtLwug1nuthS1mGNrYqh1eceUnd1aGngDx6CC87Xjda` | `5HAHa48EwaPodCXF6gkiwzKumM27Rj8B2qAN48E9sRsXWStL` | yes  |   0.13 | Any                  |
| `5G6G5HPp6bCWEHy2LAVzJ6eeARCnbGvKgav7SDyWsMmjzxNM` | `5Ffa7uuNrKXYja4xgHBBqzNxZXpT6ScJc1vTYSS8D1X2aJ1X` | yes  |   0.07 | Any                  |
| `5CiVtMcSVN1SGYYD8QmcLM8yBh3c3ujTJzRR8913ccCiMUfA` | `5EDAxF6DhwoExm6an6f5DFz5LnhitHvBACKLySoPiSjqdoPC` | yes  |   0.07 | Any                  |
| `5GbibGc5VbyJtE3em6EPhYqMhvTTbw7E5zNWkmr8wx3YJLJL` | `5Hk5UwJztBcAHrmYe9qhY6s15Qg6NBV3YthDEzxsJRwgJYfp` | yes  |   0.05 | Any                  |
| `5GYTo2WjDerprswqsnVkSVxLHQHXFD4ryKoCPYGvyRj3Fxz8` | `5FahSEAe7sKC2J6X3YWFS7ynds5m11qTWxGb69s326jtxx2F` | yes  |   0.04 | Any                  |
| `5CsaMZZchUb429JMrJvgsr53XVL8BcCJ44PhrgCC3D5ABEDb` | `5EqTyRyGGtX3NSfJdm5xn8VUTPQJkPpA98pZszpGLDaCwDFq` | yes  |   0.03 | Any                  |
| `5GSJdn8GBgt9AaFsW3RD2RGkx6dBcZRWhY3idXExDE3j1p8q` | `5HTAMws17QJpXLF9hYA9hgDQ1ykPRGPUQP6DDiNrUCJhRzKq` | yes  |   0.03 | Any                  |

### Multisigs that granted a proxy

90 multisigs set a proxy on themselves (showing the 30 with ≥ 20 τ;
the other 60 are dust validator-setup pairs).

| Multisig (delegator)                               |      τ | Grants to (delegate) → type                                                                                                                                                                                           |
|----------------------------------------------------|-------:|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `5HiFDVNX4ivCJFt9RvgRCtQKmAPgAXGX8BRgX3XKqfY9fFve` | 11,521 | `5C7UKd4m…` → SwapHotkey; `5CJyNgas…` → ChildKeys; `5Cq17EDW…` → Any; `5CyZCcg6…` → ChildKeys; `5EyKF7kc…` → Staking; `5FeNcqhf…` → RootWeights; `5FudzRud…` → Registration; `5GLq32tv…` → Any; `5HVmtneH…` → Staking |
| `5Ehk4sfD6EJfQuxHfGu8ZY1akU9CYqwok98WT1Cp5K11RGUy` |  9,923 | `5CRzqFHP…` → Staking; `5EyKF7kc…` → Staking                                                                                                                                                                          |
| `5HqaAYanMmQUh6agtnsXeWcHJDsoBF4eWwAkUPV7WEG3zZqW` |  8,121 | `5CRzqFHP…` → Staking; `5EyKF7kc…` → Staking; `5FTw4dyS…` → Any                                                                                                                                                       |
| `5F85xjhaRjSjk3qX5DPPav2S6ouVrxXANaFpaznz47iPPYgb` |  2,186 | `5GEzBxAV…` → Staking                                                                                                                                                                                                 |
| `5DXYuxYkUudh2doWt2UsCzjLnnbui8dBmJdQArRn64Rqeizx` |  1,538 | `5ERkZiR5…` → Staking                                                                                                                                                                                                 |
| `5Dj9woHuaPJsUEvX5pTvd4J8QnaKBzb3gNTtCS7Yo2PAqkp2` |  1,411 | `5HQrHWi2…` → Staking                                                                                                                                                                                                 |
| `5FtwryD1hutQevGm915avuw1TPz4k1xvRjDGN3NZ9ubp6QXV` |  1,210 | `5DD4iZSg…` → Staking                                                                                                                                                                                                 |
| `5FWSrvUejJ1NX1fcWgZVBNSpsrHr5C2AzZAUviosPEyUjVs9` |    897 | `5FCgHJto…` → Staking; `5FapNrHp…` → NonTransfer                                                                                                                                                                      |
| `5CTkibFohf93bxMGsUy17Qc1ENyZJYVQ3LARUXiHqEyCAaZB` |    885 | `5EqmG3gV…` → Staking                                                                                                                                                                                                 |
| `5HkGCkce7aKxinYtU588kjt7sy2HKrKgKyhbNoe13kvrPFT2` |    594 | `5HpqefpB…` → Staking                                                                                                                                                                                                 |
| `5GkZb6S3PSv6stahzWXgMg2PAe8CxEYSp3PXWPJybhLt1xiF` |    593 | `5HGXMFSN…` → Staking                                                                                                                                                                                                 |
| `5FnWKpesLZj1ZknKJZ6bzF3VucRxgD7VE4MFVDkh3WDbeUbL` |    587 | `5H4Tjmcv…` → Staking                                                                                                                                                                                                 |
| `5Fp5Amh3dhXZ5e8sBZ3NRX26kWytgaFK462beT9W7EeGwSjk` |    561 | `5CqaMZnC…` → Staking                                                                                                                                                                                                 |
| `5FKtGi1wGjN5fqve5j1TDTXkoKdZ1jJC8Ppc7anezMqbNx7z` |    560 | `5CaDD9KQ…` → Staking                                                                                                                                                                                                 |
| `5CSyxKQB7M2YmWdZPBnWWLqBq3ucQXCPuhdQ3aQfjyu37Zr2` |    557 | `5FRomxaP…` → Staking                                                                                                                                                                                                 |
| `5CGq1JQhmZmR8w3pXFqycUgX9mcLt8bPwMZ8YDJbEmELpw7X` |    547 | `5FJwhCxm…` → NonTransfer; `5FJwhCxm…` → Transfer; `5HYGHNZZ…` → Owner; `5HYGHNZZ…` → Staking; `5HYGHNZZ…` → Registration; `5HYGHNZZ…` → ChildKeys; `5HYGHNZZ…` → SwapHotkey; `5HYGHNZZ…` → SubnetLeaseBeneficiary    |
| `5Dbes6CDQN161m1oFxvs2ThqvYgpCq5VMtz6MoeUNdeBvh8L` |    531 | `5C5zREaG…` → Staking                                                                                                                                                                                                 |
| `5FLQ2m1ZgVd2qXfE4ZXtxyuqmjjJHycKqFEWvExCiNzUtEEe` |    523 | `5F9yhmZo…` → Staking                                                                                                                                                                                                 |
| `5CsiGTsNBAn1bNiGNEd5LYpo6bm3PXT5ogPrQmvpZaUb2XzZ` |    461 | `5DLXqvT8…` → Staking                                                                                                                                                                                                 |
| `5GCwezBrYRCxEESEUhfhGtJpfXT8MfNDdW1tFM2pH7rCm5c8` |    400 | `5FFTPUJW…` → Staking                                                                                                                                                                                                 |
| `5ESwpyuGxBmkXuQ1J8DqtmhFZQEDzLWKVup9xai567JRhvDN` |    382 | `5Dvsf6wA…` → Staking                                                                                                                                                                                                 |
| `5HCT4AarReToT1BKyLtJXJfSLs4zRS7dENnZ7iysqrqxXyV7` |    255 | `5DLXqvT8…` → Staking                                                                                                                                                                                                 |
| `5ENm74gN9ajYYUvvXwL2g2DbP2MkobAbntqV1Chvmb1gEQBC` |    246 | `5FyWspKc…` → Staking                                                                                                                                                                                                 |
| `5CTdHCps1vJ9LrZ3SPM8ZweXWtgg6vgEqEb5W8xzuZHQHXpA` |    183 | `5Esxxgir…` → NonTransfer; `5Esxxgir…` → Transfer                                                                                                                                                                     |
| `5E3CpnLgSMb7ggjLpqoKgPPyCmiqgwLfhozeDGo1qQH1rJWt` |    164 | `5DX1AZe8…` → Staking; `5EHVUNEq…` → Staking; `5ExK3SD4…` → Any                                                                                                                                                       |
| `5F7HAs8t6FnWp4RxaNi7JJMQnowZRsbRxaNrF8WqiAPujBqM` |    117 | `5HQRsE6H…` → Staking                                                                                                                                                                                                 |
| `5Ge57pPLDUKApt49W9ZFDsrjEMVMD1Xsn1euc8EJVHECU6Jn` |     98 | `5E1xWQfd…` → Staking                                                                                                                                                                                                 |
| `5Cqc9Mxct3dZMSrY1Mryv77H2su6SkSKqfqbiNahVWQo8he2` |     36 | `5CSk6qjE…` → Staking                                                                                                                                                                                                 |
| `5Hp7eihfcgjUyxm2Ckbj78f4Dng92yqeBo8jNS4LXYt7A7xU` |     23 | `5Gk6Z5gi…` → Staking                                                                                                                                                                                                 |
| `5ExHnok2jC8BiqRtcStAxwfQZXqMGyo5LbXArUpNitPV8xXN` |     20 | `5DtCHmzx…` → SudoUncheckedSetCode                                                                                                                                                                                    |

## Roles & identity

- Subnet-owner coldkeys in the proxy graph : 24
- Accounts with an on-chain identity : 44

| Account                                            | Identity              |      τ | In graph as |
|----------------------------------------------------|-----------------------|-------:|-------------|
| `5EBuUXD6eXSSWVaT1NqaUQoAACUkAmEogzAfPQvDXTEQZ8Ff` | Openτensor Foundaτion | 37,650 | delegator   |
| `5HiFDVNX4ivCJFt9RvgRCtQKmAPgAXGX8BRgX3XKqfY9fFve` | TAO.app               | 11,521 | delegator   |
| `5EbY1SebUuz189w1G96JN4mmHnShAECA6JefCRYhSLqyYg2c` | Bitcast               |  6,559 | delegator   |
| `5DAQpczEK4vzBn1waHkC4BZGqGPZ1dwPxKVsj36JDofHAw3a` | Hippius               |  4,692 | delegator   |
| `5FHrQMjzzAhmL5zS9ys87ZrGCwG3vsVT9hXAUWZQ8SNdRqig` | SOMA                  |  3,529 | delegator   |
| `5Eq8b9p6zJMjEXyH9sX4DRMYspnUyorEKq3Zmha1WN6AC4sf` | Crucible Labs         |  2,785 | delegator   |
| `5FCSevLkofmKZRixMawp6jyyjBty1AeSCLa7N5Fv892DYkXX` | Sportstensor          |  1,925 | delegator   |
| `5DhsVYewpCdQQUenHU52k5Cys1WsTWTt5V5m84D3n4L8FWDS` | Ridges                |  1,650 | delegator   |
| `5FTxn37JFkQn2c9jAY1SkdtqUHKACsV9ZjmCBnBaYcUktaBo` | NorthTensor           |  1,332 | delegator   |
| `5CGq1JQhmZmR8w3pXFqycUgX9mcLt8bPwMZ8YDJbEmELpw7X` | Verathos              |    547 | delegator   |
| `5GTPBjA4uXhuQ51SJB7Jd55JwY6dKEnbnjCrsSSEXy3MN63z` | Vidaio                |    430 | delegator   |
| `5DywxdtESjskgPZrDXL86qV44SpPgJuqs9X6noyJJwX9PaSD` | General Tensor        |    270 | delegator   |
| `5GxEEHxLWpvcAGghL8LG7xNjrxZk4SEPW1VT43hGPZ2TP9mZ` | TAO Private Network   |    269 | delegator   |
| `5CAc19iETJmWD2rYVX1ht58ghCpyHq86MoBNdx5TzLfinzcx` | Owner7                |    266 | delegator   |
| `5CTdHCps1vJ9LrZ3SPM8ZweXWtgg6vgEqEb5W8xzuZHQHXpA` | Nodexo                |    183 | delegator   |
| `5HBPFVcJxuhT6PFg93s5B5qa9GZjMQWRBWWEssZMDAMmRAvc` | DSV Vali              |    138 | delegator   |
| `5GcCZ2BPXBjgG88tXJCEtkbdg2hNrPbL4EFfbiVRvBZdSQDC` | Taostats              |    120 | delegator   |
| `5GurNtB3yQFCh6CSmfH7LrYJDsvzup4diHZiaYtKe274nrMX` | Owner5                |    117 | delegator   |
| `5EJQWShhBbbHPUSgQ5hN5zJbn4uWtDuvr1gy4xpJLhPhyuYL` | Astrid                |    105 | delegator   |
| `5GsbTgfvgCH4xdqSkiPb7EaBBFLHjWH5vfEALhJaewSFpZX9` | tao.bot               |    105 | delegator   |
| `5Gdjfu2KZPxFadRhRXtc1CGF8374mSGS2ZVi7NrNUAFtwibX` | -                     |    100 | delegator   |
| `5FLjGr5a1FHSSPe4Df8uVH5HBpeAkj2aJuWZR18XjFxA6pmH` | Tensorplex Labs       |     98 | delegator   |
| `5E6wTZZipkTmm6mci5jZp1FwXoUSGgG8CemFeC3DsV2nUGiM` | Djinn                 |     90 | delegator   |
| `5EJAqczgzCMvWcmXhKMZH4vMS5gPy8BjeuHjz5o5yN6RYzX2` | tao5                  |     50 | delegator   |
| `5E9fVY1jexCNVMjd2rdBsAxeamFGEMfzHcyTn2fHgdHeYc5p` | Yuma, a DCG Company   |     28 | delegator   |
| `5ENrWG7XFS7bcTnZFhYqj4W8EAAeYS3hAFUQfPPDMWeKHEcx` | 01Lamida, Inc.        |     24 | delegator   |
| `5CWzmvA17MAMQ9mnAecLxFXS2N8846rz6T7m4QNHyVtJVq4j` | TAO.com               |     21 | delegator   |
| `5HTogQKFuDaLq5ifWNhL2owMpLPHM3TAmujUUaTVj7FWzf6p` | Owner30               |      4 | delegator   |
| `5D53sX6AwAzXUB24G85Ch35hYr5rCpXjYFWpFTe4uzBb75sL` | Green Compute         |      3 | delegator   |
| `5CopvHeTZHNy6RMztuRFvViwwZbsZaVkJPA9zmRRnPAna73M` | Neural Internet       |      2 | delegator   |
| `5HjCYVfrWSkzTfJM5rkWBW3qTTJqXEFUzZrKty5hodpgfjyW` | Owner21               |      2 | delegator   |
| `5GLFMoUSHk9d9CkBnhwCYKEX8HUyrVkjgorfZuxYmnCdSKsz` | shangshan222          |      1 | delegator   |
| `5DkRocgbM16F41BLGs3UMoqwKdrbmkzQiUHgnzLHXrV9frob` | Cortex Foundαtion     |   0.88 | delegator   |
| `5GP8N57T2oja6qR9y3FQDYjrEFxnhmx3ZuiadE64yw6h5For` | <decommissioned>      |   0.73 | delegator   |
| `5FNKX2Af6xgcYMoAQs1ZuoanxNAV2wFosFKE9Zg1Fs9nXDMf` | RoundTable21          |   0.22 | delegate×2  |
| `5E7dez1zSF5L5NPSTYBrRRP8K5r7kKGNuwnrtzspHNP9n3EA` | Shugo                 |   0.20 | delegator   |
| `5DRnT7XqmAzkXAiScTaoRkkSkyKGaTx1ArJzZBsvsA1Rj5Xp` | Unit 410              |   0.02 | delegator   |
| `5H6ruUtxp2SzythEoJsQRQiUM9KzXQeJJuDUZEmQWAP11EdM` | Ymoyzic               |   0.00 | delegate×1  |
| `5CLyhrqcMHvrpzHDHpmn71qy1AEdrRzfyW9n3Kne8dnQB3du` | Owner47               |   0.00 | delegator   |
| `5Dz7y3o7GbDzfDAiF9CFnQTSGkTakP547eJZQGjwqrSM2eeL` | TaiHotech             |   0.00 | delegator   |

## Method

Full `Proxy.Proxies` storage map at block 8,616,072 (live grants, not replayed from events);
`Proxy.Announcements` read the same way. Free from `System.Account`, stake from `StakeInfoRuntimeApi`,
alpha valued per subnet swap pool (mark = pool price; realizable applies AMM slippage). Pure proxies =
keyless delegators (`nonce == 0`) minus known multisigs. Exfiltration classification verified against
the subtensor runtime `InstanceFilter`.