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
