# Summary

* [Pendahuluan](README.md)
* [Executive Summary (Ringkasan Eksekutif)](executive-summary.md)
* [Rencana Implementasi](architecture/implementation_plan.md)
* [Arsitektur Sistem](architecture/implementation_plan_architecture.md)
* [Sequence Diagram (Alur Eksekusi CANB)](architecture/sequence-diagrams.md)

## Dokumentasi Smart Contract
### CANB Framework (Treatment)
* [CANBGateway](smart-contracts/CANBGateway.md)
* [NonceStore](smart-contracts/NonceStore.md)
* [DomainSeparator](smart-contracts/DomainSeparator.md)
* [BridgeExecutor](smart-contracts/BridgeExecutor.md)
* [ContextHasher](smart-contracts/ContextHasher.md)
* [ContextValidator](smart-contracts/ContextValidator.md)
* [ValidationModule](smart-contracts/ValidationModule.md)

### Baseline Bridge (Control)
* [BaselineGateway](smart-contracts/BaselineGateway.md)
* [BaselineExecutor](smart-contracts/BaselineExecutor.md)

### Core Bridge & Interfaces
* [baseBridge](smart-contracts/baseBridge.md)
* [IBridgeBenchmark](smart-contracts/IBridgeBenchmark.md)
* [IValidationModule](smart-contracts/IValidationModule.md)
* [INonceStore](smart-contracts/INonceStore.md)

### Libraries & Tokens
* [CANBErrors](smart-contracts/CANBErrors.md)
* [CANBEvents](smart-contracts/CANBEvents.md)
* [MockERC20](smart-contracts/MockERC20.md)
* [mockTelU1](smart-contracts/mockTelU1.md)

## Keamanan & Pengujian
### Threat Modelling (Model Ancaman)
* [Skenario Serangan Utama](security-analysis/attack_scenarios.md)
* [AV-1: Same-Chain Replay](security-analysis/threat-modelling/av1-same-chain-replay.md)
* [AV-2: Cross-Chain Replay](security-analysis/threat-modelling/av2-cross-chain-replay.md)
* [AV-3: Double Spending](security-analysis/threat-modelling/av3-double-spending.md)
* [AV-4: Amount Manipulation](security-analysis/threat-modelling/av4-amount-manipulation.md)
* [AV-5: Different Bridge Replay](security-analysis/threat-modelling/av5-different-bridge-replay.md)

### Test Suites (Ringkasan)
* [Attack Test Suite](security-analysis/attack-test-suite.md)
* [Unit Test Suite](security-analysis/unit-test-suite.md)

### Unit Tests Code
* [NonceStore Test](tests/NonceStore.md)
* [DomainSeparator Test](tests/DomainSeparator.md)
* [ContextHasher Test](tests/ContextHasher.md)
* [ContextValidator Test](tests/ContextValidator.md)
* [BridgeExecutor Test](tests/BridgeExecutor.md)
* [ValidationModule Test](tests/ValidationModule.md)

### Attack Tests Code
* [Same Context Replay](tests/SameContextReplay.md)
* [Cross Context Replay](tests/CrossContextReplay.md)
* [Cross Bridge Replay](tests/CrossBridgeReplay.md)
* [Message Type Confusion](tests/MessageTypeConfusion.md)
* [Nonce Ordering Attack](tests/NonceOrderingAttack.md)

### Integration Tests & Benchmark Code
* [Baseline Bridge Integration](tests/BaselineBridge.md)
* [CANB Bridge Integration](tests/CANBBridge.md)
* [Comparison Benchmark](tests/ComparisonBenchmark.md)

### Hasil Eksekusi Uji (Dry-Run Logs)
* [Dry-Run: Unit Tests](results/unit-tests-results.md)
* [Dry-Run: Attack Tests](results/attack-tests-results.md)
* [Dry-Run: Integration Tests](results/integration-tests-results.md)

## Evaluasi & Deployment
* [Panduan Reproduksi (Quick Start)](deployment-evaluation/reproduction-guide.md)
* [Dokumen Deployment Script](deployment-evaluation/deploy-script.md)
* [DeployAll Script Code](scripts/DeployAll.md)
* [Analisis Biaya Gas (Gas Benchmark)](deployment-evaluation/gas-benchmark.md)
* [Gas Snapshot Report](tests/gas-snapshot-report.md)

## Laporan Progres Mingguan
* [Laporan Kemajuan Utama](progress-logs/progrep.md)
* [Progres Minggu 1](progress-logs/progW1.md)
* [Progres Minggu 2](progress-logs/progW2.md)
* [Progres Minggu 3](progress-logs/progW3.md)
* [Progres Minggu 4](progress-logs/progW4.md)
* [Progres Minggu 5](progress-logs/progW5.md)
* [Progres Minggu 6](progress-logs/progW6.md)
* [Progres Minggu 7](progress-logs/progW7.md)
* [Progres Minggu 8](progress-logs/progW8.md)
