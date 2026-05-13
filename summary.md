# 6h shadow benchmark — 2026-05-11

Continuous Core-primary shadow run comparing the `select_core_baseline`
template against `select_optimized_template` across one full 6-hour
window. Every cycle is signed with Ed25519 (`bench-receipt/1`) and pinned
in [manifest.json](manifest.json) by SHA-256.

## Run parameters

| Field | Value |
| --- | --- |
| Data source | Local Bitcoin Core 290000 (`http://127.0.0.1:8332`, pruned) |
| Interval | 300 s |
| Duration | 21,600 s (6 h) |
| Started at (unix) | 1778517478 |
| Ended at (unix) | 1778539078 |
| Cycles completed | 72 |
| Block weight budget | 4,000,000 WU |
| Max tx count | 4,000 |
| Minimum fee rate | 1.0 sat/vB |
| Signing alg | Ed25519 |
| Signing `key_id` | `a1d577350a959ecd` |

## Results

| Metric | Value |
| --- | --- |
| Mempool size range | 218 – 6,863 txs |
| Uplift min | 124.10 bps |
| Uplift median | 577.51 bps |
| Uplift mean | 627.06 bps |
| Uplift p95 | 1376.18 bps |
| Uplift max | 1499.18 bps |
| Total uplift across run | 7,400,254 sats |

Uplift is computed as
`(optimized_fees − baseline_fees) / baseline_fees × 10000` per cycle, with
`baseline_fees` from `select_core_baseline` and `optimized_fees` from
`select_optimized_template` running over the same Core mempool snapshot.

## Verification

Every file in this directory verifies:

```powershell
$env:BENCHMARK_RECEIPT_ED25519_PK = (Get-Content pubkey.hex -Raw)
Get-ChildItem cycle_*.json, manifest.json | ForEach-Object {
    .\target\release\verify_bench_receipt.exe $_.FullName
}
```

Confirm the manifest still pins every envelope byte-for-byte:

```powershell
$m = Get-Content manifest.json -Raw | ConvertFrom-Json
foreach ($p in $m.payload.envelope_digests_sha256.PSObject.Properties) {
    $exp = $p.Value
    $act = (Get-FileHash -Algorithm SHA256 $p.Name).Hash.ToLower()
    if ($exp -ne $act) { Write-Host "MISMATCH $($p.Name)" }
}
```

Both checks were run at archive time:

- `verify_bench_receipt` ok=73 fail=0 (72 cycles + manifest)
- manifest digest pinning: 72 envelopes checked, 0 mismatches

## Reproducing

```powershell
$env:BENCHMARK_RECEIPT_ED25519_SK = '<hex 32-byte seed>'
$env:BTC_RPC_URL                  = 'http://127.0.0.1:8332'
$env:BTC_RPC_USER                 = '<core rpc user>'
$env:BTC_RPC_PASSWORD             = '<core rpc password>'
$env:NATIVEBTC_API_KEY            = '<optional fastpath key>'
.\target\release\shadow_runner.exe `
    --interval-secs 300 `
    --duration-secs 21600 `
    --out-dir target\shadow_run
```

Note: the published `pubkey.hex` is the demo verifier key only. Production
runs must rotate the signing seed and update `pubkey.hex` accordingly.

Note: ran on Hardware Consumer Intel i7, 16 GB RAM, WiFi.

## Updated info
# Replayable benchmark corpus

This run is a self-contained selector replay corpus. It contains raw mempool snapshots, cross-regime sampling, adversarial replay profiles, deterministic comparison files, an explicit integration-path statement, a failure-case ledger, and a signed manifest that pins every artifact by SHA-256. It deliberately separates valuation-grade empirical results from derived robustness and stress diagnostics; no aggregate uplift is computed across real and synthetic tiers.

## Snapshot library

| Snapshot | File | Captured at | Core height | Transactions | Total fees | SHA-256 |
| ---: | --- | ---: | ---: | ---: | ---: | --- |
| 1 | snapshots/mempool_0001.json | 1778714055 | 949278 | 365 | 196217 | `1d359b096155e2ff62008fb1505098584390fd260b04c7653397463bf2ffa451` |
| 2 | snapshots/mempool_0002.json | 1778714060 | 949278 | 384 | 199769 | `5e95026ec01c3b2a86e9cdb1f5e3f415a9b46950850c1071034b5588e5385050` |
| 3 | snapshots/mempool_0003.json | 1778714065 | 949278 | 393 | 207927 | `e61f5f150bdc844978ef6d14ee2a9ca80b552ad00df31c4451ef052fa4e6f51f` |

## Methodology guardrails

| Rule | Treatment |
| --- | --- |
| Base data | SHA-pinned empirical mempool snapshots captured from Bitcoin Core. |
| Transform model | Synthetic scenarios are deterministic transforms: `S' = T(S, theta)`. |
| Valuation rule | Use Tier 1 empirical results only for market-edge economics. |
| Reporting rule | Do not compute or present an overall uplift across empirical and synthetic scenarios. |
| Robustness rule | Tier 2 and Tier 3 results describe fragility, sensitivity, and failure modes only. |

## Tiered uplift distribution

| Group | Count | Min bps | Median bps | Mean bps | P95 bps | Max bps | Stddev bps | Negative | <= 0 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Tier 1 empirical | 3 | 1206.16 | 1219.84 | 1229.21 | 1261.63 | 1261.63 | 23.60 | 0 | 0 |
| Tier 2 derived robustness | 18 | 195.03 | 1206.16 | 2124.72 | 9168.70 | 9347.19 | 3007.74 | 0 | 0 |
| Tier 3 stress diagnostic | 12 | 0.00 | 1206.16 | 614.61 | 1261.63 | 1261.63 | 614.83 | 0 | 6 |

## Scenario uplift distribution

| Tier | Scenario | Count | Min bps | Median bps | Mean bps | P95 bps | Max bps | Stddev bps | Negative | <= 0 |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Tier 3 stress diagnostic | adversarial_no_package_edge | 3 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0 | 3 |
| Tier 3 stress diagnostic | adversarial_rbf_top_fee | 3 | 1206.16 | 1219.84 | 1229.21 | 1261.63 | 1261.63 | 23.60 | 0 | 0 |
| Tier 3 stress diagnostic | adversarial_unbroadcast_top_fee | 3 | 1206.16 | 1219.84 | 1229.21 | 1261.63 | 1261.63 | 23.60 | 0 | 0 |
| Tier 3 stress diagnostic | congestion_10_sat_vb | 3 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0 | 3 |
| Tier 2 derived robustness | congestion_2_sat_vb | 3 | 962.26 | 1014.79 | 999.15 | 1020.40 | 1020.40 | 26.18 | 0 | 0 |
| Tier 2 derived robustness | congestion_5_sat_vb | 3 | 195.03 | 219.25 | 211.18 | 219.25 | 219.25 | 11.42 | 0 | 0 |
| Tier 2 derived robustness | half_block_pressure | 3 | 1206.16 | 1219.84 | 1229.21 | 1261.63 | 1261.63 | 23.60 | 0 | 0 |
| Tier 1 empirical | live_full | 3 | 1206.16 | 1219.84 | 1229.21 | 1261.63 | 1261.63 | 23.60 | 0 | 0 |
| Tier 2 derived robustness | regime_high_fee_band | 3 | 1224.00 | 1292.09 | 1278.39 | 1319.08 | 1319.08 | 40.01 | 0 | 0 |
| Tier 2 derived robustness | regime_low_fee_tail | 3 | 7736.33 | 9168.70 | 8750.74 | 9347.19 | 9347.19 | 720.99 | 0 | 0 |
| Tier 2 derived robustness | regime_mid_fee_band | 3 | 233.46 | 298.18 | 279.67 | 307.38 | 307.38 | 32.90 | 0 | 0 |

## Scenario profiles

| Scenario | Tier | Transform kind | Sampling regime | Adversarial profile | Description |
| --- | --- | --- | --- | --- | --- |
| adversarial_no_package_edge | Tier 3 stress diagnostic | adversarial_sampling | independent_high_fee | no_package_edge | Keep only independent high-fee transactions to expose tie/loss cases where package-aware selection has little raw-fee edge. |
| adversarial_rbf_top_fee | Tier 3 stress diagnostic | adversarial_metadata | full_mempool | rbf_top_fee | Mark the highest-fee third as BIP125 replaceable to test replacement-risk sensitivity. |
| adversarial_unbroadcast_top_fee | Tier 3 stress diagnostic | adversarial_metadata | full_mempool | unbroadcast_top_fee | Mark the highest-fee decile as unbroadcast/fresh to test compact-block risk penalties versus raw-fee baseline. |
| congestion_10_sat_vb | Tier 3 stress diagnostic | extreme_fee_floor | fee_floor | none | Keep transactions at or above 10 sat/vB plus required ancestors. |
| congestion_2_sat_vb | Tier 2 derived robustness | fee_floor | fee_floor | none | Keep transactions at or above 2 sat/vB plus required ancestors. |
| congestion_5_sat_vb | Tier 2 derived robustness | fee_floor | fee_floor | none | Keep transactions at or above 5 sat/vB plus required ancestors. |
| half_block_pressure | Tier 2 derived robustness | blockspace_compression | full_mempool | none | Replay the full mempool with a 2,000,000 WU block budget to stress marginal selection. |
| live_full | Tier 1 empirical | none | full_mempool | none | Replay the captured mempool without filtering; optimizer min fee remains 1 sat/vB. |
| regime_high_fee_band | Tier 2 derived robustness | cross_regime_sampling | high_fee_band | none | Sample the high-fee band to test the near-next-block regime where fewer packages are marginal. |
| regime_low_fee_tail | Tier 2 derived robustness | cross_regime_sampling | low_fee_tail | none | Sample the lower-fee tail of the mempool to test behaviour when package edge is scarce. |
| regime_mid_fee_band | Tier 2 derived robustness | cross_regime_sampling | mid_fee_band | none | Sample the middle fee-rate band to test selector stability away from extreme tails. |

## Deterministic replay comparisons

| Snapshot | Tier | Scenario | Mempool txs | Baseline fees | Optimized fees | Uplift sats | Uplift bps | Optimized weight |
| ---: | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | Tier 1 empirical | live_full | 365 | 174884 | 196217 | 21333 | 1219.84 | 336689 |
| 1 | Tier 2 derived robustness | congestion_2_sat_vb | 172 | 141357 | 155781 | 14424 | 1020.40 | 200889 |
| 1 | Tier 2 derived robustness | congestion_5_sat_vb | 23 | 50627 | 51737 | 1110 | 219.25 | 16682 |
| 1 | Tier 3 stress diagnostic | congestion_10_sat_vb | 7 | 35618 | 35618 | 0 | 0.00 | 4903 |
| 1 | Tier 2 derived robustness | half_block_pressure | 365 | 174884 | 196217 | 21333 | 1219.84 | 336689 |
| 1 | Tier 2 derived robustness | regime_low_fee_tail | 155 | 23833 | 42271 | 18438 | 7736.33 | 150314 |
| 1 | Tier 2 derived robustness | regime_mid_fee_band | 134 | 48617 | 49752 | 1135 | 233.46 | 110879 |
| 1 | Tier 2 derived robustness | regime_high_fee_band | 111 | 109349 | 123773 | 14424 | 1319.08 | 138857 |
| 1 | Tier 3 stress diagnostic | adversarial_unbroadcast_top_fee | 365 | 174884 | 196217 | 21333 | 1219.84 | 336689 |
| 1 | Tier 3 stress diagnostic | adversarial_rbf_top_fee | 365 | 174884 | 196217 | 21333 | 1219.84 | 336689 |
| 1 | Tier 3 stress diagnostic | adversarial_no_package_edge | 102 | 108777 | 108777 | 0 | 0.00 | 88939 |
| 2 | Tier 1 empirical | live_full | 384 | 177389 | 199769 | 22380 | 1261.63 | 347571 |
| 2 | Tier 2 derived robustness | congestion_2_sat_vb | 174 | 142138 | 156562 | 14424 | 1014.79 | 202011 |
| 2 | Tier 2 derived robustness | congestion_5_sat_vb | 23 | 50627 | 51737 | 1110 | 219.25 | 16682 |
| 2 | Tier 3 stress diagnostic | congestion_10_sat_vb | 7 | 35618 | 35618 | 0 | 0.00 | 4903 |
| 2 | Tier 2 derived robustness | half_block_pressure | 384 | 177389 | 199769 | 22380 | 1261.63 | 347571 |
| 2 | Tier 2 derived robustness | regime_low_fee_tail | 140 | 20496 | 39654 | 19158 | 9347.19 | 142780 |
| 2 | Tier 2 derived robustness | regime_mid_fee_band | 133 | 47563 | 49025 | 1462 | 307.38 | 110187 |
| 2 | Tier 2 derived robustness | regime_high_fee_band | 117 | 111633 | 126057 | 14424 | 1292.09 | 142663 |
| 2 | Tier 3 stress diagnostic | adversarial_unbroadcast_top_fee | 384 | 177389 | 199769 | 22380 | 1261.63 | 347571 |
| 2 | Tier 3 stress diagnostic | adversarial_rbf_top_fee | 384 | 177389 | 199769 | 22380 | 1261.63 | 347571 |
| 2 | Tier 3 stress diagnostic | adversarial_no_package_edge | 108 | 111061 | 111061 | 0 | 0.00 | 92745 |
| 3 | Tier 1 empirical | live_full | 393 | 185547 | 207927 | 22380 | 1206.16 | 353656 |
| 3 | Tier 2 derived robustness | congestion_2_sat_vb | 181 | 149897 | 164321 | 14424 | 962.26 | 206616 |
| 3 | Tier 2 derived robustness | congestion_5_sat_vb | 26 | 56914 | 58024 | 1110 | 195.03 | 18622 |
| 3 | Tier 3 stress diagnostic | congestion_10_sat_vb | 9 | 40658 | 40658 | 0 | 0.00 | 6133 |
| 3 | Tier 2 derived robustness | half_block_pressure | 393 | 185547 | 207927 | 22380 | 1206.16 | 353656 |
| 3 | Tier 2 derived robustness | regime_low_fee_tail | 142 | 20895 | 40053 | 19158 | 9168.70 | 144260 |
| 3 | Tier 2 derived robustness | regime_mid_fee_band | 137 | 49030 | 50492 | 1462 | 298.18 | 112857 |
| 3 | Tier 2 derived robustness | regime_high_fee_band | 120 | 117843 | 132267 | 14424 | 1224.00 | 144451 |
| 3 | Tier 3 stress diagnostic | adversarial_unbroadcast_top_fee | 393 | 185547 | 207927 | 22380 | 1206.16 | 353656 |
| 3 | Tier 3 stress diagnostic | adversarial_rbf_top_fee | 393 | 185547 | 207927 | 22380 | 1206.16 | 353656 |
| 3 | Tier 3 stress diagnostic | adversarial_no_package_edge | 111 | 117271 | 117271 | 0 | 0.00 | 94533 |

## Failure cases

Failure means the optimized template captured fewer or equal raw fees than the Core-style baseline for the same snapshot and scenario.

| Snapshot | Tier | Scenario | Baseline fees | Optimized fees | Delta sats | Uplift bps |
| --- | --- | --- | ---: | ---: | ---: | ---: |
| 1 | Tier 3 stress diagnostic | congestion_10_sat_vb | 35618 | 35618 | 0 | 0.0000 |
| 1 | Tier 3 stress diagnostic | adversarial_no_package_edge | 108777 | 108777 | 0 | 0.0000 |
| 2 | Tier 3 stress diagnostic | congestion_10_sat_vb | 35618 | 35618 | 0 | 0.0000 |
| 2 | Tier 3 stress diagnostic | adversarial_no_package_edge | 111061 | 111061 | 0 | 0.0000 |
| 3 | Tier 3 stress diagnostic | congestion_10_sat_vb | 40658 | 40658 | 0 | 0.0000 |
| 3 | Tier 3 stress diagnostic | adversarial_no_package_edge | 117271 | 117271 | 0 | 0.0000 |

## Integration path

| Layer | Status |
| --- | --- |
| Bitcoin Core | Live snapshot input through `getrawmempool(verbose=true)`; archived snapshots replay from `snapshots/mempool_*.json`. |
| Simulation/replay | Fully implemented here; deterministic comparisons are byte-stable for the same snapshot and selector version. |
| Adversarial replay | Implemented as deterministic metadata/regime transformations over archived snapshots; profiles are labeled per scenario. |
| Miner/template builder | Selector output is an ordered transaction set plus fee/weight summaries suitable for block-template assembly. |
| Stratum | Not implemented in this corpus; Stratum job construction/distribution remains a separate integration layer. |
| Production boundary | This corpus proves template selection behaviour, not coinbase construction, witness commitment, block propagation, or miner job delivery. |

## Verification

The manifest is a signed `bench-receipt/1` envelope with `receipt_kind=corpus-manifest/1`. It pins snapshots, scenario definitions, comparison files, summary, and pubkey.

```powershell
$env:BENCHMARK_RECEIPT_ED25519_PK = Get-Content pubkey.hex -Raw
.\target\release\verify_bench_receipt.exe manifest.json
```

Data source: bitcoin_core_rpc
Chain: main
Public key: `29e5833a915a6429a4e3a7948475c338ef436eb82be89c92f059704403db9d55`
Key id: `a1d577350a959ecd`

