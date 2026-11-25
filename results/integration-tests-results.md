# Hasil Pengujian (Dry-Run): Integration & 4-Tier Benchmark Tests (14/14 Passed)

Dokumen ini berisi log eksekusi langsung dari Foundry (`forge 1.7.1`, `Solc 0.8.30`) untuk rangkaian **Integration Tests** dan **4-Tier Apple-to-Apple Gas Benchmark** (Stage E1 Cold-Start & Stage E3 Steady-State Batch `N=50` dan `N=100`).

## Output Log (`forge test --match-path "test/integration/*" -vv`)

```text
Ran 2 tests for test/integration/BaselineBridge.t.sol:BaselineBridgeTest
[PASS] test_FullFlow_LockAndProcess() (gas: 88000)
[PASS] test_ProcessMessage_Succeeds() (gas: 55641)
Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 3.46ms (1.70ms CPU time)

Ran 2 tests for test/integration/CANBBridge.t.sol:CANBBridgeTest
[PASS] test_FullFlow_MintViaProcessMessage() (gas: 154736)
[PASS] test_FullFlow_SequentialMessages() (gas: 211471)
Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 3.51ms (4.13ms CPU time)

Ran 10 tests for test/integration/ComparisonBenchmark.t.sol:ComparisonBenchmarkTest
[PASS] test_B1_Conventional_LockAssets() (gas: 83963)
[PASS] test_B1_Conventional_ProcessMessage() (gas: 77067)
[PASS] test_B2_ModernStandard_LockAssets() (gas: 106747)
[PASS] test_B2_ModernStandard_ProcessMessage() (gas: 78702)
[PASS] test_Baseline_LockAssets() (gas: 83842)
[PASS] test_Baseline_ProcessMessage() (gas: 54228)
[PASS] test_CANB_LockAssets() (gas: 107169)
[PASS] test_CANB_ProcessMessage() (gas: 137130)
[PASS] test_E3_AmortizedBatch_N50() (gas: 6790854)
Logs:
  === STAGE E3 AMORTIZED GAS BENCHMARK ===
  Batch size N: 50
  B0   (Naive)      | Cold: 43642 | Steady Avg: 14106 | Overall Avg: 14696
  B1   (Legacy)     | Cold: 66067 | Steady Avg: 36967 | Overall Avg: 37549
  B2   (ModernStd)  | Cold: 62069 | Steady Avg: 37469 | Overall Avg: 37961
  CANB (4-Layer)    | Cold: 114312 | Steady Avg: 23912 | Overall Avg: 25720
  CANB vs B1 Overall Overhead (bps, 100=1%): -3150
  CANB vs B2 Overall Overhead (bps, 100=1%): -3224
  CANB vs B1 Steady  Overhead (bps, 100=1%): -3531
  CANB vs B2 Steady  Overhead (bps, 100=1%): -3618

[PASS] test_E3_AmortizedBatch_N100() (gas: 13593425)
Logs:
  === STAGE E3 AMORTIZED GAS BENCHMARK ===
  Batch size N: 100
  B0   (Naive)      | Cold: 44038 | Steady Avg: 14106 | Overall Avg: 14405
  B1   (Legacy)     | Cold: 66067 | Steady Avg: 36967 | Overall Avg: 37258
  B2   (ModernStd)  | Cold: 62069 | Steady Avg: 37469 | Overall Avg: 37715
  CANB (4-Layer)    | Cold: 114312 | Steady Avg: 23912 | Overall Avg: 24816
  CANB vs B1 Overall Overhead (bps, 100=1%): -3339
  CANB vs B2 Overall Overhead (bps, 100=1%): -3420
  CANB vs B1 Steady  Overhead (bps, 100=1%): -3531
  CANB vs B2 Steady  Overhead (bps, 100=1%): -3618

Suite result: ok. 10 passed; 0 failed; 0 skipped (14 total integration & benchmark tests)
```
