# Panduan Reproduksi (Quick Start Guide)

Dokumen ini ditujukan bagi pembimbing, penguji, atau peneliti yang ingin mereproduksi seluruh **114/114 pengujian** (Unit, 4-Tier Attack Matrix, AV-1-1 Sleeping Replay, dan 4-Tier Gas Benchmark) secara deterministik di komputer lokal.

## 1. Lingkungan Terverifikasi (Environment Fingerprint)
- **Compiler:** `Solc 0.8.30` (`via_ir = true`, `optimizer = true`, `optimizer_runs = 200`)
- **Framework:** Foundry `forge 1.7.1` (Commit `4072e48`)

Jika belum menginstal Foundry:
```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

## 2. Kloning & Kompilasi Repositori
```bash
git clone <URL_REPOSITORI_ANDA> thesis-canb
cd thesis-canb/ccBridge
forge build
```

## 3. Menjalankan Seluruh Test Suite (114/114 Passing)
```bash
forge test --summary
```
*Expected Output:* `17 test suites`, **114 passed, 0 failed, 0 skipped** (51 CANB unit tests, 47 attack tests, 14 integration/benchmark tests, 2 template tests).

## 4. Verifikasi Matriks Serangan 4-Tier & AV-1-1 (Stage E2)
```bash
# Matriks 4-Tier (B0 vs B1 vs B2 vs CANB) & Invarian Delta Assets = 0
forge test --match-contract FourTierSecurityMatrixTest -vv

# AV-1-1 Sleeping Replay (vm.warp +2h & exact boundary T = 3600s)
forge test --match-contract SleepingReplayTest -vv

# Seluruh Attack Suite (47 tests across 7 suites)
forge test --match-path "test/attack/*" -vv
```

## 5. Menjalankan 4-Tier Apple-to-Apple Gas Benchmark (Stage E1 & E3)
```bash
# Menampilkan log lengkap Cold Tx #1, Steady-State Avg, Overall Avg (N=50 & N=100), dan % overhead
forge test --match-contract ComparisonBenchmarkTest -vv

# Menyimpan snapshot gas 4-Tier
forge snapshot --match-contract ComparisonBenchmarkTest --snap .gas-snapshot-4tier
```

## 6. Deployment Simulasi Lokal (Anvil — Opsional)
```bash
# Terminal 1
anvil

# Terminal 2
forge script script/DeployAll.s.sol:DeployAll --rpc-url http://localhost:8545 --broadcast
```
