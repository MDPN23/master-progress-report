# Executive Summary (Ringkasan Eksekutif)

## Latar Belakang Masalah
Pertumbuhan ekosistem blockchain memicu tingginya adopsi *Cross-Chain Bridge* (jembatan lintas-rantai) untuk mentransfer likuiditas. Dari September 2021 hingga Juli 2023, eksploitasi bridge menghasilkan kerugian terkonfirmasi sebesar **$2.83 miliar** di 35+ insiden — mewakili 72% dari total kerugian eksploitasi DeFi [Zhang et al., RAID 2024]. Akar masalah utama adalah **State Isolation**: EVM pada satu chain tidak dapat secara native memverifikasi state, konteks, atau riwayat eksekusi chain lain.

Serangan paling kritis — **Replay Attack** dan **Double Spending** — terjadi karena penyerang menggunakan ulang bukti kriptografis (*signatures*) yang sama untuk merampok aset dari *smart contract* bridge, akibat kelemahan validasi dan ketiadaan ikatan konteks.

## Solusi: CANB Framework
Tesis ini memperkenalkan **Context-Aware Nonce Binding (CANB)** — arsitektur *smart contract* modular yang dirancang khusus untuk memitigasi serangan replay lintas-rantai melalui 4 pilar pengamanan:

1. **ContextHasher & DomainSeparator (EIP-712):** Mengikat ketat setiap tanda tangan dengan ID Jaringan, Alamat Kontrak, dan Bridge ID — sesuai dengan standar domain separation NIST SP 800-185.
2. **ContextValidator:** Memverifikasi parameter masuk, rute asal-tujuan, dan mencegah intrusi logika.
3. **NonceStore (Bitmap + Time-Bounded):** Penyimpanan nonce bitmap biner dengan *temporal freshness constraint* — mencegah double spending deterministik **dan** sleeping replay attack.
4. **BridgeExecutor (CEI Pattern):** Pemisahan lapisan eksekutor token dengan pola Checks-Effects-Interactions.

### Formal Security Foundation
CANB didefinisikan melalui **Context Binding Function CB(σ, C)** yang memastikan:
- **P1 (Domain Binding):** Signature valid di konteks C₁ → invalid di C₂ (probability ≥ 1-2⁻²⁵⁶)
- **P2 (Nonce Uniqueness + Freshness):** Bitmap monoton + temporal bound = rejection deterministik
- **P3 (Asset Conservation):** P1 + P2 + CEI → ΔAssets = 0

## Hasil Evaluasi & Pembuktian (Baseline vs CANB)

Dalam pengujian di lingkungan simulasi **Foundry**, CANB dibandingkan dengan desain jembatan konvensional (Baseline Bridge) pada **7 sub-vektor serangan**:

| Skenario Serangan | Baseline | CANB | Status |
| :--- | :--- | :--- | :--- |
| **AV-1: Same-Chain Replay** | ❌ Rentan (Double Minting) | ✅ Aman (Bitmap nonce) | Lulus 100% |
| **AV-1-1: Sleeping Replay** | ❌ Rentan (No temporal check) | ✅ Aman (Time-bounded T) | Lulus 100% |
| **AV-2: Cross-Chain Replay** | ❌ Rentan (Aset diklaim di chain lain) | ✅ Aman (EIP-712 chainId) | Lulus 100% |
| **AV-2-1: Cross-Bridge Replay** | ❌ Rentan (Diklaim di bridge B) | ✅ Aman (EIP-712 contractAddr) | Lulus 100% |
| **AV-3: Double Spending** | ❌ Rentan | ✅ Aman (Atomic execution + CEI) | Lulus 100% |
| **AV-4: Validator Impersonation** | ❌ Rentan jika tanpa registry | ✅ Aman (Validator registry) | Lulus 100% |
| **AV-5: Amount Manipulation** | ❌ Rentan jika tanpa binding payload | ✅ Aman (Payload hashing) | Lulus 100% |

### Metodologi Evaluasi
- **Binomial Test** (α = 0.05): Konfirmasi statistik blocking rate 100%
- **Empirical Invariant Verification**: ΔAssets = 0 pada seluruh fase eksperimen
- **One-sample t-test + One-way ANOVA**: Gas overhead <20% konsisten di 3 testnet (Base Sepolia, Lisk Sepolia, Kaia Kairos)

**Kesimpulan:**
CANB terbukti memitigasi 100% vektor serangan Replay Attack (termasuk novel sleeping replay) sembari mempertahankan gas overhead pada 17.2% — di bawah threshold 20%. Framework ini bersifat *reproducible* dan *ready-to-deploy* pada jaringan heterogen berbasis EVM.
