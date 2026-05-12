# Bitcoin-Template-Benchmark

Live 6-hour shadow benchmark of an ancestor-aware Bitcoin block template engine against a Core-style fee-rate baseline.

The benchmark was executed against a live Bitcoin Core node using real mempool snapshots over 72 signed benchmark cycles.

## Benchmark Summary

* Median uplift: +578 bps over baseline
* Minimum uplift: +124 bps
* Never negative across the full benchmark window
* Total additional fees captured: 7,400,254 sats
* Every benchmark cycle cryptographically signed with Ed25519
* SHA-256-pinned manifest for full-run verification

## Latest Live Snapshot

Example benchmark cycle from the completed run:

| Metric                   | Baseline       | Optimized      |
| ------------------------ | -------------- | -------------- |
| Total Fees               | 2,314,120 sats | 2,569,436 sats |
| Block Weight             | 3,723,114      | 3,995,956      |
| Transaction Count        | 3,167          | 3,562          |
| Fee / Weight             | 0.621555       | 0.643009       |
| Expected Revenue         | 2,314,120 sats | 2,560,838 sats |
| Compact Block Efficiency | 0.0000         | 0.9759         |

### Measured Improvement

* Additional fees captured: +255,316 sats
* Uplift: +1103 bps
* Package candidates evaluated: 3,704
* Weight budget respected: 3,995,956 / 4,000,000

The uplift comes from package-aware CPFP transaction selection: identifying profitable ancestor/child transaction groups that naive fee-rate-only selection leaves behind.

## Included Verification Artifacts

This repository contains:

* Signed benchmark receipts
* Verification manifests
* Cross-language receipt verifiers
* Benchmark methodology and reproducibility artifacts

All benchmark receipts are independently verifiable.

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
* Cross-language verifiers
* Deployment/runtime infrastructure

## Notice

© 2026 Emiliano Solazzi. All rights reserved.

Benchmark artifacts are published strictly for verification and evaluation purposes. No source code is included or licensed for production, commercial, or derivative use.

## Acquisition / Licensing

The full platform — including the Rust optimizer, Flow API, shadow benchmark system, deployment stack, and verification tooling — is available for acquisition or strategic licensing.

Contact:

* [emiliano@nativebtc.org](mailto:emiliano@nativebtc.org)
* [coma.retained@gmail.com](mailto:coma.retained@gmail.com)

