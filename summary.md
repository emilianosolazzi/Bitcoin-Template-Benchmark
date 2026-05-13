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

This run is a self-contained selector replay corpus. It contains raw mempool snapshots, deterministic congestion scenarios, deterministic comparison files, an explicit integration-path statement, a failure-case ledger, and a signed manifest that pins every artifact by SHA-256.

## Snapshot library

| Snapshot | File | Captured at | Core height | Transactions | Total fees | SHA-256 |
| ---: | --- | ---: | ---: | ---: | ---: | --- |
| 1 | snapshots/mempool_0001.json | 1778713128 | 949276 | 403 | 392216 | `912e0a7f9311e4ee30a2270320e03de3958831f2cbce76789daa6ff00c1b3ec4` |
| 2 | snapshots/mempool_0002.json | 1778713133 | 949276 | 418 | 399960 | `24f224cadeb3bd67e7fd642b2745da28d9c68608254f1ea3797aa7079ef6f98d` |
| 3 | snapshots/mempool_0003.json | 1778713138 | 949276 | 436 | 454947 | `413d4450cb3c95582b427e98215d00e229d1713cbe42cd536cd69e41da66d08b` |

## Uplift distribution

| Group | Count | Min bps | Median bps | Mean bps | P95 bps | Max bps | Stddev bps | Negative | <= 0 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| all | 15 | 0.00 | 606.98 | 456.30 | 851.05 | 851.05 | 356.55 | 0 | 3 |
| congestion_10_sat_vb | 3 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0 | 3 |
| congestion_2_sat_vb | 3 | 513.44 | 606.98 | 579.36 | 617.67 | 617.67 | 46.82 | 0 | 0 |
| congestion_5_sat_vb | 3 | 71.58 | 72.24 | 72.02 | 72.24 | 72.24 | 0.31 | 0 | 0 |
| half_block_pressure | 3 | 744.40 | 849.71 | 815.06 | 851.05 | 851.05 | 49.96 | 0 | 0 |
| live_full | 3 | 744.40 | 849.71 | 815.06 | 851.05 | 851.05 | 49.96 | 0 | 0 |

## Deterministic replay comparisons

| Snapshot | Scenario | Mempool txs | Baseline fees | Optimized fees | Uplift sats | Uplift bps | Optimized weight |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | live_full | 403 | 361499 | 392216 | 30717 | 849.71 | 658792 |
| 1 | congestion_2_sat_vb | 211 | 252776 | 268119 | 15343 | 606.98 | 220064 |
| 1 | congestion_5_sat_vb | 25 | 153661 | 154771 | 1110 | 72.24 | 17437 |
| 1 | congestion_10_sat_vb | 7 | 135367 | 135367 | 0 | 0.00 | 4024 |
| 1 | half_block_pressure | 403 | 361499 | 392216 | 30717 | 849.71 | 658792 |
| 2 | live_full | 418 | 368591 | 399960 | 31369 | 851.05 | 671702 |
| 2 | congestion_2_sat_vb | 221 | 258955 | 274950 | 15995 | 617.67 | 230030 |
| 2 | congestion_5_sat_vb | 25 | 153661 | 154771 | 1110 | 72.24 | 17437 |
| 2 | congestion_10_sat_vb | 7 | 135367 | 135367 | 0 | 0.00 | 4024 |
| 2 | half_block_pressure | 418 | 368591 | 399960 | 31369 | 851.05 | 671702 |
| 3 | live_full | 436 | 423427 | 454947 | 31520 | 744.40 | 742791 |
| 3 | congestion_2_sat_vb | 228 | 311527 | 327522 | 15995 | 513.44 | 291715 |
| 3 | congestion_5_sat_vb | 26 | 155081 | 156191 | 1110 | 71.58 | 18002 |
| 3 | congestion_10_sat_vb | 8 | 136787 | 136787 | 0 | 0.00 | 4589 |
| 3 | half_block_pressure | 436 | 423427 | 454947 | 31520 | 744.40 | 742791 |

## Failure cases

Failure means the optimized template captured fewer or equal raw fees than the Core-style baseline for the same snapshot and scenario.

| Snapshot | Scenario | Baseline fees | Optimized fees | Delta sats | Uplift bps |
| --- | --- | ---: | ---: | ---: | ---: |
| 1 | congestion_10_sat_vb | 135367 | 135367 | 0 | 0.0000 |
| 2 | congestion_10_sat_vb | 135367 | 135367 | 0 | 0.0000 |
| 3 | congestion_10_sat_vb | 136787 | 136787 | 0 | 0.0000 |

## Integration path

| Layer | Status |
| --- | --- |
| Bitcoin Core | Live snapshot input through `getrawmempool(verbose=true)`; archived snapshots replay from `snapshots/mempool_*.json`. |
| Simulation/replay | Fully implemented here; deterministic comparisons are byte-stable for the same snapshot and selector version. |
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

