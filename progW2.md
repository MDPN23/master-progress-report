# 📋 Progress Minggu 2 — Mulai Desain Basis Benchmark Contract

**Penelitian:** *Mitigation Framework for Double-Spending and Replay Attacks in DeFi Cross-Chain Bridges*
**Periode:** Minggu ke-2 pasca Seminar Proposal
**Status:** ✅ Selesai

---

## 1. Tujuan Minggu Ini

Mendesain **baseline benchmark contract** — implementasi bridge cross-chain *tanpa* CANB sebagai kontrol eksperimen untuk mengukur DV1, DV2, DV3.

---

## 2. Filosofi Desain Benchmark

Dalam DSR, setiap artefak harus dievaluasi terhadap kondisi *tanpa artefak*:

- **Baseline Bridge (tanpa CANB):** Menerima semua transaksi valid tanpa domain separation/nonce binding kontekstual
- **CANB-Enabled Bridge:** Full validation pipeline

Perbandingan menghasilkan:
- **DV1:** % serangan replay terblokir (baseline = 0%, CANB target ≥99.9%)
- **DV2:** Overhead latency dari validasi CANB
- **DV3:** Overhead gas dari validasi CANB

| Prinsip Desain | Penjelasan |
| :--- | :--- |
| **Functional Equivalence** | Workflow lock-mint/burn-release identik, minus validasi keamanan |
| **Same Interface** | Interface sama agar test harness reusable |
| **Measurable** | Emit event yang sama untuk pengukuran apple-to-apple |
| **Deliberately Vulnerable** | Sengaja tidak cek nonce uniqueness & domain separation |

---

## 3. Arsitektur Benchmark Contract

### 3.1 Komponen Overview

```
BaselineBridge System
├── BaselineGateway.sol        # Entry point — cek format dasar saja
├── BaselineLockRelease.sol    # Lock source, release destination
├── BaselineMintBurn.sol       # Mint destination, burn return
└── BaselineEventLogger.sol    # Event emission untuk data collection
```

### 3.2 Interface Definition (Shared)

```solidity
interface IBridgeBenchmark {
    function lockAssets(address token, uint256 amount, uint256 destChainId, address recipient)
        external returns (bytes32 messageHash);
    function processMessage(bytes32 messageHash, address token, uint256 amount,
        address recipient, uint256 sourceChainId, uint256 nonce, bytes calldata signature)
        external returns (bool success);
    function burnAssets(address wrappedToken, uint256 amount, uint256 destChainId, address recipient)
        external returns (bytes32 messageHash);
    function releaseAssets(bytes32 messageHash, address token, uint256 amount,
        address recipient, uint256 sourceChainId, uint256 nonce, bytes calldata signature)
        external returns (bool success);
}
```

### 3.3 Baseline Gateway — Deliberately Minimal

```solidity
contract BaselineGateway {
    // ⚠️ TIDAK ADA domain separation
    // ⚠️ TIDAK ADA context-aware nonce binding
    // ⚠️ Hanya basic signature recovery

    function processMessage(bytes32 messageHash, uint256 amount, address recipient,
        uint256 sourceChainId, uint256 nonce, bytes calldata signature) external {
        address signer = ECDSA.recover(messageHash, signature);
        require(authorizedRelayers[signer], "Unauthorized");
        // TIDAK ada nonce uniqueness check
        // TIDAK ada context validation
        _executeMint(recipient, amount);
        emit MessageProcessed(messageHash, sourceChainId, nonce, true);
    }
}
```

> [!WARNING]
> Baseline **sengaja vulnerable** — ini adalah *control experiment*, TIDAK BOLEH di-deploy ke mainnet.

### 3.4 Vulnerability Points yang Sengaja Dibiarkan

| Vulnerability | Exploit yang Dimungkinkan |
| :--- | :--- |
| **No Domain Separation** | Signature di Chain A bisa di-replay di Chain B |
| **No Nonce Binding** | Transaksi sama bisa disubmit berkali-kali |
| **No Context Validation** | Attacker bisa claim message dari chain manapun |
| **Simple ECDSA Recovery** | Tanpa EIP-712, hash collision lebih mudah |

---

## 4. Studi Kasus Bridge Exploit untuk Validasi Desain

### 4.1 Wormhole ($325M, Feb 2022)
- Tidak memvalidasi guardian signatures dengan benar → forge attestation → 120K wETH
- **Baseline mereplikasi:** Signature check ada, domain separation tidak

### 4.2 Multichain ($126M, Jul 2023)
- Nonce reuse across 10+ chains → repeated collateral release
- **Baseline mereplikasi:** Nonce ada, uniqueness TIDAK di-enforce

### 4.3 Ronin ($625M, Mar 2022)
- 5/9 validator key compromised → replay lock-mint transactions
- **Baseline mereplikasi:** Single relayer authority tanpa multi-factor context check

---

## 5. Gas Profiling Framework

```
Gas Overhead = (Gas_CANB - Gas_Baseline) / Gas_Baseline × 100%
Target: < 20%
```

| Operasi | Expected Gas (Baseline) |
| :--- | :--- |
| `lockAssets()` | ~65,000 gas |
| `processMessage()` | ~85,000 gas |
| `burnAssets()` | ~55,000 gas |
| `releaseAssets()` | ~75,000 gas |

---

## 6. Test Harness & Expected Results

| Test Case | Baseline | CANB | DV |
| :--- | :---: | :---: | :--- |
| Legitimate Transfer | ✅ Pass | ✅ Pass | DV2, DV3 |
| Same-Context Replay | ✅ Pass (vuln) | ❌ Revert | DV1 |
| Cross-Context Replay | ✅ Pass (vuln) | ❌ Revert | DV1 |
| Double-Spend | ✅ Pass (vuln) | ❌ Revert | DV1 |
| High Load (100 tx) | ✅ Pass | ✅ Pass | DV2, DV3 |

---

## 7. Deliverables Minggu 2

- [x] Arsitektur baseline benchmark contract (4 komponen)
- [x] Shared interface definition (`IBridgeBenchmark`)
- [x] 4 vulnerability points teridentifikasi & didokumentasi
- [x] 3 studi kasus exploit historis (Wormhole, Multichain, Ronin)
- [x] Gas profiling framework + Foundry snapshot setup
- [x] Test harness design dengan expected results matrix

### Transisi ke Minggu 3
Fokus: **Desain attack scheme** — merancang skenario serangan spesifik untuk menguji efektivitas CANB versus baseline.
