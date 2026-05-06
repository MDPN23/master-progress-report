# Executive Summary (Ringkasan Eksekutif)

## Latar Belakang Masalah
Pertumbuhan ekosistem blockchain memicu tingginya adopsi *Cross-Chain Bridge* (jembatan lintas-rantai) untuk mentransfer likuiditas. Sayangnya, jembatan menjadi target eksploitasi peretas yang bernilai miliaran dolar.
Salah satu serangan paling kritis adalah **Replay Attack** dan **Double Spending**, di mana penyerang menggunakan ulang bukti kriptografis (*signatures*) yang sama untuk merampok aset (token) secara berulang dari *smart contract* bridge. Hal ini terjadi karena kelemahan desain validasi dan ketiadaan ikatan konteks antara pesan dan arsitektur *smart contract* tujuan.

## Solusi: CANB Framework
Tesis ini memperkenalkan **Cross-chain Adaptive Nonce Bridge (CANB)**. CANB adalah arsitektur *smart contract* yang dirancang khusus untuk memitigasi *Replay Attack* lintas-rantai tanpa mengorbankan skalabilitas *gas fee*.

CANB membungkus lapisan validasi melalui 4 pilar pengamanan:
1.  **ContextHasher & DomainSeparator (EIP-712):** Mengikat ketat setiap tanda tangan dengan ID Jaringan (*Chain ID*) dan Alamat Kontrak (*Bridge Address*).
2.  **ContextValidator:** Memverifikasi parameter masuk, rute asal-tujuan, dan mencegah intrusi logika.
3.  **NonceStore (Bitmap):** Implementasi penyimpanan *nonce* menggunakan memori *bitmap* biner yang sangat murah secara *gas*, mencegah pembelanjaan ganda (*double spending*) secara deterministik.
4.  **BridgeExecutor (CEI Pattern):** Pemisahan lapisan eksekutor token dengan pola keamanan *Checks-Effects-Interactions*.

## Hasil Evaluasi & Pembuktian (Baseline vs CANB)

Dalam pengujian di lingkungan simulasi **Foundry**, CANB dibandingkan dengan desain jembatan konvensional (Baseline Bridge).

| Jenis Skenario Serangan | Kerentanan Baseline | Ketahanan CANB | Status Evaluasi |
| :--- | :--- | :--- | :--- |
| **Same-Chain Replay (AV-1)** | ❌ Rentan (Double Minting) | ✅ Aman (Revert bit nonce) | Lulus 100% |
| **Cross-Chain Replay (AV-2)** | ❌ Rentan (Aset diklaim di jaringan lain) | ✅ Aman (EIP-712 Block) | Lulus 100% |
| **Double Spending (AV-3)** | ❌ Rentan | ✅ Aman (Atomic execution) | Lulus 100% |
| **Amount Manipulation (AV-4)**| ❌ Rentan jika tanpa binding payload | ✅ Aman (Hashing payload) | Lulus 100% |
| **Cross-Bridge Replay (AV-5)**| ❌ Rentan (Diklaim di bridge B) | ✅ Aman (EIP-712 Address block) | Lulus 100% |

**Kesimpulan:** 
CANB terbukti memitigasi 100% vektor serangan *Replay Attack* sembari tetap mempertahankan biaya transaksi (Gas Benchmark) pada level efisiensi industri untuk eksekusi lintas-rantai. Framework ini bersifat *reproducible* dan *ready-to-deploy* pada jaringan berbasis EVM.
