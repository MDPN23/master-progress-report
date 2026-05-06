# 🎓 Tesis: Cross-chain Adaptive Nonce Bridge (CANB) Framework

Selamat datang di repositori utama untuk **Laporan Kemajuan Tesis (Master Progress Report)**. Repositori dan dokumentasi ini dibuat dan disusun oleh saya sebagai bagian dari evaluasi penelitian tesis.

Repositori ini merangkum seluruh perjalanan penelitian, desain arsitektur, hingga implementasi kode Solidity yang saya buat untuk memitigasi celah kerentanan *Replay Attack* dan *Double Spending* pada protokol *Cross-Chain Bridge*. 

## 📖 Tentang Dokumentasi Ini

Dokumentasi ini dirancang agar sepenuhnya *self-contained* dan terintegrasi mulus dengan **Gitbook**. Di dalamnya, Anda akan menemukan:
1. **Laporan Mingguan (Progress Logs)**: Rekam jejak mingguan dari proses pengerjaan dan pencapaian tesis ini.
2. **Arsitektur & Rencana Implementasi**: Blueprint dari desain sistem CANB yang saya rancang secara teknis.
3. **Dokumentasi Smart Contract**: Penjelasan fungsionalitas beserta *source code* utuh untuk setiap modul *smart contract* (termasuk modul CANB, kontrol Baseline, dan antarmukanya).
4. **Keamanan & Evaluasi (Threat Modelling)**: Uji simulasi serangan yang membandingkan ketahanan CANB versus Baseline Bridge terhadap eksploitasi peretas.
5. **Data Empiris & Hasil Dry-Run**: Evaluasi *gas benchmark*, snapshot Foundry, serta log eksekusi asli dari rangkaian integrasi.

## 🚀 Membaca Laporan
Seluruh navigasi telah diatur rapi di bagian *sidebar* sebelah kiri (dirender oleh Gitbook melalui `SUMMARY.md`). Anda bisa mulai membaca dari **Executive Summary**, masuk ke penjelasan **Arsitektur**, atau langsung meninjau **Hasil Evaluasi Keamanan**.

Jika Anda ingin mereproduksi atau menguji kode ini di mesin lokal (menggunakan Foundry), silakan menuju ke [Panduan Reproduksi](deployment-evaluation/reproduction-guide.md).

---
*Dibuat dengan semangat penelitian untuk membangun ekosistem cross-chain yang lebih aman.*
