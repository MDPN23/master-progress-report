# ⛽ Gas Snapshot & Benchmark Documentation

**Penelitian:** *Mitigation Framework for Double-Spending and Replay Attacks in DeFi Cross-Chain Bridges*
**Tool:** Foundry `forge snapshot`
**Test File:** `test/integration/ComparisonBenchmark.t.sol`

---

## 1. Overview

Gas snapshot digunakan untuk mengukur **overhead** yang dihasilkan oleh proteksi keamanan CANB dibandingkan dengan Baseline yang vulnerable. Overhead target: **≤20%**.

> **Catatan:** Hasil gas di bawah ini diukur dengan `via-ir = true` dan `optimizer_runs = 200`. Gas sesungguhnya di testnet mungkin sedikit berbeda karena state size dan EVM implementation.

---

## 2. Gas Snapshot Results

### Raw Snapshot (dari `gas-snapshot.txt`)

```
ComparisonBenchmarkTest:test_Baseline_LockAssets() (gas: 54808)
ComparisonBenchmarkTest:test_Baseline_ProcessMessage() (gas: 24778)
ComparisonBenchmarkTest:test_CANB_LockAssets() (gas: 107120)
ComparisonBenchmarkTest:test_CANB_ProcessMessage() (gas: 134109)
```

### Comparison Table

| Operation | Baseline (gas) | CANB (gas) | Overhead | Overhead % |
|:--|:--:|:--:|:--:|:--:|
| `processMessage()` | 24,778 | 134,109 | +109,331 | — |
| `lockAssets()` | 54,808 | 107,120 | +52,312 | +95.4% |

> [!IMPORTANT]
> `processMessage()` pada Baseline **tidak melakukan mint/release** — hanya verifikasi signature dan emit event. Sedangkan CANB `processMessage()` menjalankan **full pipeline** (ContextValidator → ContextHasher → ECDSA.recover → NonceStore.consume → BridgeExecutor.mint). Perbandingan yang lebih adil adalah antara operasi yang setara (lock vs lock, atau full-flow vs full-flow).

### Adjusted Comparison (Apples-to-Apples)

Untuk perbandingan yang fair, kita harus membandingkan **full-flow end-to-end**:

| Full Flow | Baseline | CANB | Overhead |
|:--|:--:|:--:|:--:|
| Lock assets (user → bridge) | 54,808 | 107,120 | +95.4% |
| Process + Mint (relayer call) | ~24,778* | 134,109 | — |

*\*Baseline processMessage hanya verify sig + emit, tanpa mint.*

---

## 3. Overhead Analysis

### 3.1 Sumber Overhead CANB

| Component | Estimated Gas Cost | Justification |
|:--|:--:|:--|
| ContextValidator (route check) | ~3,000 | 3x SLOAD + comparison |
| ContextHasher (EIP-712 hash) | ~8,000 | keccak256 + abi.encode |
| ECDSA.recover | ~3,000 | ecrecover precompile |
| NonceStore.consumeNonce | ~25,000 | SSTORE (cold) + SLOAD |
| BridgeExecutor.mintTokens | ~50,000 | ERC20._mint (storage write) |
| Events emission | ~5,000 | ValidationPassed + TokensMinted |

### 3.2 Gas Optimization Techniques Applied

1. **Bitmap nonce storage** — 1 SSTORE per 256 nonces (vs. 1 SSTORE per nonce)
2. **Immutable domain separator** — No runtime computation, cached at deploy
3. **Custom errors** — ~200 gas saved per revert vs `require(…, "string")`
4. **Calldata** over memory — Struct passed as `calldata` wherever possible

---

## 4. Running the Benchmark

```bash
# Generate gas snapshot
forge snapshot --match-contract ComparisonBenchmark --snap gas-snapshot.txt

# View snapshot
cat gas-snapshot.txt

# Compare with previous snapshot
forge snapshot --match-contract ComparisonBenchmark --diff gas-snapshot.txt

# Full gas report (per-function breakdown)
forge test --match-contract ComparisonBenchmark --gas-report
```

---

## 5. Interpretation for Thesis

Meskipun overhead CANB **lebih tinggi dari target 20%** pada operasi `lockAssets`, hal ini karena:

1. **CANB `lockAssets` melakukan lebih banyak**: emit event dengan `contextHash` + `nonce` + `destinationChainId`, sementara Baseline hanya emit `Deposit(sender, token, amount)`.
2. **CANB mencatat deposit di BridgeExecutor** (`recordDeposit`) yang menambah 1 SSTORE.
3. **Security overhead adalah trade-off yang terukur** — setiap gas yang ditambahkan berkontribusi pada proteksi spesifik.

> [!NOTE]
> Untuk bab pembahasan thesis, argumentasikan bahwa overhead gas ini adalah **biaya keamanan yang justified** mengingat CANB menghilangkan 100% attack surface pada 36 skenario serangan.
