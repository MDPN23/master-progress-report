# Gas Snapshot Report (4-Tier: B0 vs B1 vs B2 vs CANB)

## Deskripsi dan Kegunaan
File ini merupakan rekaman (*snapshot*) resmi dari Foundry (`forge 1.7.1`, `Solc 0.8.30`) mengenai konsumsi gas seluruh operasi `lockAssets`, `processMessage` (Stage E1 Cold-Start), dan *Amortized Batch* (Stage E3 `N=50` & `N=100`) pada arsitektur komparasi 4-Tier (`B0`, `B1`, `B2`, dan `CANB`).

## Snapshot Data (`.gas-snapshot-4tier`)

```text
ComparisonBenchmarkTest:test_Baseline_LockAssets() (gas: 83842)
ComparisonBenchmarkTest:test_B1_Conventional_LockAssets() (gas: 83963)
ComparisonBenchmarkTest:test_B2_ModernStandard_LockAssets() (gas: 106747)
ComparisonBenchmarkTest:test_CANB_LockAssets() (gas: 107169)
ComparisonBenchmarkTest:test_Baseline_ProcessMessage() (gas: 54228)
ComparisonBenchmarkTest:test_B1_Conventional_ProcessMessage() (gas: 77067)
ComparisonBenchmarkTest:test_B2_ModernStandard_ProcessMessage() (gas: 78702)
ComparisonBenchmarkTest:test_CANB_ProcessMessage() (gas: 137130)
ComparisonBenchmarkTest:test_E3_AmortizedBatch_N50() (gas: 6790854)
ComparisonBenchmarkTest:test_E3_AmortizedBatch_N100() (gas: 13593425)
```

## Ringkasan Amortisasi Per Transaksi (`processMessage`)
- **Steady-State Avg (Tx #2–#100):** `B0 = 14,106` | `B1 = 36,967` | `B2 = 37,469` | **`CANB = 23,912`** (`-35.31%` vs B1, `-36.18%` vs B2)
- **Overall Avg ($N = 100$):** `B0 = 14,405` | `B1 = 37,258` | `B2 = 37,715` | **`CANB = 24,816`** (`-33.39%` vs B1, `-34.20%` vs B2)
