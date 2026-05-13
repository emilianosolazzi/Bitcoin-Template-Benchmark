# Bitcoin-Template-Benchmark

Live shadow benchmark of an ancestor-aware Bitcoin block template engine against a Core-style fee-rate baseline, plus a signed proof-of-execution receipt against an actual mined Bitcoin block.

The benchmark was executed against a live Bitcoin Core node using real mempool snapshots. Benchmark cycles and live template/block receipts are cryptographically signed with Ed25519.

## Benchmark Summary

* Median uplift: +578 bps over baseline
* Minimum uplift: +124 bps
* Never negative across the 6-hour benchmark window
* Total additional fees captured: 7,400,254 sats
* Every benchmark cycle cryptographically signed with Ed25519
* SHA-256-pinned manifest for full-run verification

## Latest Live Template Snapshot

Fresh live template snapshot from the current proof run:

| Metric            | Baseline       | Optimized      |
| ----------------- | -------------- | -------------- |
| Total Fees        | 1,390,685 sats | 1,456,635 sats |
| Block Weight      | 3,938,637      | 3,995,655      |
| Transaction Count | 3,021          | 3,272          |
| Mempool Size      | 3,598 tx       | 3,598 tx       |

### Measured Improvement

* Additional fees captured: +65,950 sats
* Uplift: +474.23 bps
* Template build latency: 4.623s
* Weight budget respected: 3,995,655 / 4,000,000
* Receipt kind: template-snapshot/1
* Signature: Ed25519, key_id `a1d577350a959ecd`

This snapshot was generated immediately after a real mined block cleared part of the mempool.

## Latest Proof-of-Execution vs Mined Block

The same live run also captured a real mined block and produced a signed block receipt.

| Metric                    | Value                                                                |
| ------------------------- | -------------------------------------------------------------------- |
| Block Height              | 949271                                                               |
| Block Hash                | 0000000000000000000158f3785780acf73cd7c1311c87da8ccbdb5c99d18f06     |
| Mined By                  | F2Pool, from coinbase tag                                            |
| Actual Block Fees         | 3,658,018 sats                                                       |
| Actual Block Weight       | 3,995,365                                                            |
| Actual Transaction Count  | 3,798                                                                |
| Pre-block Mempool Size    | 7,087 tx                                                             |
| Core-style Baseline Fees  | 3,449,384 sats                                                       |
| Optimized Template Fees   | 3,655,963 sats                                                       |
| Optimizer vs Actual Block | -5.62 bps                                                            |
| Snapshot Age at Block     | 9.653s                                                               |

### Interpretation

The optimizer selected 3,655,963 sats from the pre-block mempool, while the actual mined block captured 3,658,018 sats.

That is only 2,055 sats below the mined block, or -5.62 bps versus the real block. In other words, the optimizer was effectively at parity with the actual F2Pool block template using only local Bitcoin Core mempool data.

This is stronger than a synthetic benchmark because it compares the optimizer against an actual mined Bitcoin block observed on-chain.

## Performance Note

After Rayon parallelization, template build latency improved by roughly 2x.

Observed live performance:

* 7,040 tx mempool: 20.155s build time, +595.68 bps uplift
* 7,087 tx mempool: 38.005s build time, +598.89 bps uplift
* 3,598 tx mempool: 4.623s build time, +474.23 bps uplift

Sub-second template construction requires an algorithmic rewrite, not just more parallelism. The documented next steps are marginal-feerate cutoff, replacement-search gating, and reducing the target-weight sweep to a near-single-pass selector.

## Included Verification Artifacts

This repository contains:

* Signed benchmark receipts
* Signed template-snapshot receipts
* Signed block-receipt proof-of-execution receipts
* Verification manifests
* Cross-language receipt verifiers
* Benchmark methodology and reproducibility artifacts

All benchmark and live proof receipts are independently verifiable.

## Verification

Receipts use:

* Ed25519 signatures
* Deterministic canonical JSON serialization
* Cross-language verification compatibility

Verified across:

* Rust
* Python
* Go
* JavaScript

## Infrastructure

The benchmark and execution stack includes:

* Ancestor-aware package optimizer
* Bitcoin Core integration
* Flow API
* Signed benchmark receipt system
* Shadow benchmark runner
* Live template watcher
* Proof-of-execution block receipts
* Cross-language verifiers
* Deployment/runtime infrastructure

## Notice

© 2026 Emiliano Solazzi. All rights reserved.

Benchmark artifacts are published strictly for verification and evaluation purposes. No source code is included or licensed for production, commercial, or derivative use.

## Acquisition / Licensing

The full platform — including the Rust optimizer, Flow API, shadow benchmark system, live template watcher, proof-of-execution receipts, deployment stack, and verification tooling — is available for acquisition or strategic licensing.

Contact:

* [emiliano@nativebtc.org](mailto:emiliano@nativebtc.org)
* [coma.retained@gmail.com](mailto:coma.retained@gmail.com)
