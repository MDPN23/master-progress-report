# 🎓 Tesis: Cross-chain Adaptive Nonce Bridge (CANB) Framework

Selamat datang di repositori utama untuk **Laporan Kemajuan Tesis (Master Progress Report)**. Repositori dan dokumentasi ini dibuat dan disusun oleh saya sebagai bagian dari evaluasi penelitian tesis.

Repositori ini merangkum seluruh perjalanan penelitian, desain arsitektur, hingga implementasi kode Solidity yang saya buat untuk memitigasi celah kerentanan *Replay Attack* dan *Double Spending* pada protokol *Cross-Chain Bridge*. 

## 📖 Tentang Dokumentasi Ini

Dokumentasi ini dirancang agar sepenuhnya *self-contained* dan terintegrasi mulus dengan **Gitbook**. Di dalamnya, Anda akan menemukan:
1. **Laporan Mingguan (Progress Logs)**: Rekam jejak mingguan dari proses pengerjaan dan pencapaian tesis ini (Minggu 1–14).
2. **Arsitektur & Rencana Implementasi**: Blueprint dari desain sistem CANB yang saya rancang secara teknis.
3. **Dokumentasi Smart Contract**: Penjelasan fungsionalitas beserta *source code* utuh untuk setiap modul *smart contract* (termasuk modul CANB, kontrol Baseline, dan antarmukanya).
4. **Keamanan & Evaluasi (Threat Modelling)**: Model ancaman dengan **7 sub-vektor serangan** (AV-1 sampai AV-5, termasuk AV-1-1 dan AV-2-1), dipetakan ke STRIDE/CIAAN, beserta simulasi Foundry.
5. **Analisis Keamanan Formal**: Definisi formal Context Binding Function CB(σ, C), time-bounded nonce construction, dan security argument (3 properties).
6. **Data Empiris & Hasil Dry-Run**: Evaluasi *gas benchmark*, snapshot Foundry, serta log eksekusi asli dari rangkaian integrasi.

## 🔬 Status Penelitian (Minggu 14 — Juni 2026)

| Fase | Status |
|------|--------|
| Gap Analysis & Problem Formalization | ✅ Completed |
| Literature Review (13 studi, 8 dimensi) | ✅ Completed |
| Formal Security Analysis (CB(σ,C)) | ✅ Completed |
| Threat Modelling (7 sub-vektor) | ✅ Completed |
| Artefact Engineering (15 Contracts, ~795 LoC) | ✅ Completed |
| Statistical Framework (Binomial, t-test, ANOVA) | ✅ Completed |
| Testnet Deployment (Base, Lisk, Kaia) | 🔄 In Progress |
| Empirical Evaluation (B1-B3) | 📋 Planned |

## 🚀 Membaca Laporan
Seluruh navigasi telah diatur rapi di bagian *sidebar* sebelah kiri (dirender oleh Gitbook melalui `SUMMARY.md`). Anda bisa mulai membaca dari **Executive Summary**, masuk ke penjelasan **Arsitektur**, atau langsung meninjau **Hasil Evaluasi Keamanan**.

Jika Anda ingin mereproduksi atau menguji kode ini di mesin lokal (menggunakan Foundry), silakan menuju ke [Panduan Reproduksi](deployment-evaluation/reproduction-guide.md).

---
*Dibuat dengan semangat penelitian untuk membangun ekosistem cross-chain yang lebih aman.*
