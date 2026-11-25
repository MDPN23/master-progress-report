# 🎓 Tesis: Cross-chain Adaptive Nonce Bridge (CANB) Framework

Selamat datang di repositori utama untuk **Laporan Kemajuan Tesis (Master Progress Report — Semester 3 / Implementasi Proyek 2)**. Repositori dan dokumentasi ini disusun sebagai bagian dari evaluasi penelitian tesis S2 berbasis **Design Science Research (DSR)**.

Repositori ini merangkum seluruh perjalanan penelitian, desain arsitektur, formalisasi kriptografi, hingga implementasi dan validasi empiris kode Solidity untuk memitigasi kerentanan *Replay Attack* dan *Double Spending* pada protokol *Cross-Chain Bridge*.

## 📖 Tentang Dokumentasi Ini

Dokumentasi ini dirancang agar sepenuhnya *self-contained* dan terintegrasi mulus dengan **Gitbook**. Di dalamnya, Anda akan menemukan:
1. **Laporan Progres (Progress Logs)**: Rekam jejak pengerjaan tesis (Minggu 1–14 serta Fase Validasi 4-Tier Semester 3 / Nov 2026).
2. **Arsitektur & Rencana Implementasi**: Blueprint desain 4-Layer CANB dan arsitektur komparasi **4-Tier (`B0/B1/B2/CANB`)**.
3. **Dokumentasi Smart Contract**: Penjelasan fungsionalitas beserta kode sumber untuk 15 kontrak inti CANB serta 3 kontrak kontrol baseline (*ceteris paribus*).
4. **Keamanan & Evaluasi (Threat Modelling)**: Model ancaman dengan **7 sub-vektor serangan** (`AV-1`, `AV-1-1`, `AV-2`, `AV-2-1`, `AV-3`, `AV-4`, `AV-5`), dipetakan ke STRIDE/CIAAN, beserta verifikasi Foundry (**114/114 tests passing**).
5. **Analisis Keamanan Formal**: Definisi formal Context Binding Function $CB(\sigma, C)$, *time-bounded nonce construction*, dan argumen keamanan (3 properties).
6. **Data Empiris 4-Tier & Hasil Dry-Run**: Evaluasi *apple-to-apple gas benchmark* (Stage E1 Cold-Start & Stage E3 Steady-State $N=50, 100$), snapshot Foundry, serta grafik perbandingan gas.

## 🔬 Status Penelitian (Semester 3 — Implementasi Proyek 2 | 70% Selesai)

| Fase Penelitian | Bobot | Status | Capaian Konkret | Progres |
| :--- | :---: | :---: | :--- | :---: |
| 1. Literature Review, Gaps (P1–P4) & Posisi Industri | 15% | ✅ Completed | 34 referensi terverifikasi, 13-studi SoK, pemetaan GMP/Intent modern | 15% |
| 2. Threat Model & Formal Design ($CB(\sigma, C)$) | 15% | ✅ Completed | 7 sub-vektor (STRIDE/CIAAN), penemuan AV-1-1, 4-Layer CANB | 15% |
| 3. Core CANB Implementation & Local Unit/Attack Tests | 25% | ✅ Completed | 15 kontrak CANB (`Solc 0.8.30`), dokumentasi Gitbook, 87 base tests | 25% |
| 4. 4-Tier Baseline Refactor & Local Empirical Validation | 15% | ✅ Completed | `B0/B1/B2/CANB`, **114/114 tests**, Stage E3 (`-35.31%` vs B1, `-36.18%` vs B2) | 15% |
| 5. Multi-Testnet Deployment & Latency Data (DV4) | 10% | 🔄 Next Milestone | Base Sepolia, Lisk Sepolia, Kaia Kairos (Des 2026) | 0% |
| 6. Statistical Evaluation (Binomial, $t$-test, ANOVA) | 10% | 📋 Scheduled | Uji hipotesis lintas-chain H1a–H1c (Des 2026) | 0% |
| 7. Final Thesis Manuscript (Bab 4–5) & Sidang | 10% | 📋 Scheduled | Integrasi hasil empiris & Sidang Tesis (Jan–Feb 2027) | 0% |
| **Total Overall Progress** | **100%** | | **Core Artefact & Local Ch. 4 Data Ready** | **70%** |

## 🚀 Membaca Laporan
Seluruh navigasi diatur di bagian *sidebar* sebelah kiri (dirender oleh Gitbook melalui `SUMMARY.md`). Anda bisa mulai dari **[Executive Summary](executive-summary.md)**, meninjau **[Analisis Biaya Gas 4-Tier](deployment-evaluation/gas-benchmark.md)**, atau langsung mereproduksi seluruh **114/114 pengujian** di mesin lokal melalui **[Panduan Reproduksi](deployment-evaluation/reproduction-guide.md)**.

---
*Dibuat dengan semangat penelitian untuk membangun ekosistem cross-chain yang lebih aman.*
