# Progress Report - Minggu 9

**Tanggal:** 13 Mei 2026  
**Fokus Utama:** Rigor Cycle — Pemetaan Root Cause State Isolation dan Penyempurnaan Metodologi Komparatif.

## Ringkasan Aktivitas
Pada minggu ke-9, penelitian memasuki fase **Rigor Cycle** dari metodologi Design Science Research (DSR). Fokus utama adalah memetakan akar masalah *State Isolation* secara formal ke kebutuhan *atomic nonce consumption* dan *domain separation*, serta menyempurnakan metodologi komparatif antara Baseline dan CANB.

## Pencapaian & Output

### 1. Pemetaan Root Cause → Cryptographic Primitives
Melakukan analisis mendalam terhadap hubungan antara State Isolation dan dua primitif kriptografi yang mendasari CANB:
* **Domain Separation (EIP-712):** Signature harus terikat ke konteks eksekusi spesifik (chainId, bridgeId, contractAddr).
* **Atomic Nonce Consumption (Bitmap):** Setiap nonce harus dikonsumsi secara atomik dan irreversible pada semua execution path, termasuk revert path.

### 2. Refinement Security Delta Methodology
Menyempurnakan pendekatan pengukuran *security delta* antara Vulnerable Baseline dan CANB:
* Mendefinisikan **Dependent Variables** secara formal: DV1 (Blocking Rate), DV2 (Asset Conservation), DV3 (Gas Overhead), DV4 (Latency).
* Memastikan setiap DV memiliki formula pengukuran yang eksplisit dan testable.

### 3. Literature Deep-Dive: Cross-Chain Security Landscape
Review mendalam terhadap paper Zhang et al. (RAID 2024) dan Augusto et al. (IEEE S&P 2024) untuk memvalidasi bahwa gap yang diidentifikasi di literature review benar-benar belum diisi oleh penelitian lain.

## Rencana Minggu Selanjutnya (Minggu 10)
* Melanjutkan penyempurnaan desain eksperimen Phase B1 (Control) dan B2 (Attack).
* Memulai integrasi statistical analysis framework (Binomial Test, t-test, ANOVA).
