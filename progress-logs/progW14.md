# Progress Report - Minggu 14

**Tanggal:** 11 Juni 2026  
**Fokus Utama:** Finalisasi Proposal, Revisi Komprehensif Seluruh Section, dan Persiapan Deployment.

## Ringkasan Aktivitas
Minggu ke-14 menyelesaikan fase Proposal Formalization. Seluruh section (2-9) telah direvisi untuk mengintegrasikan: (a) CB(σ, C) formal definition, (b) time-bounded nonce construction, (c) AV-1-1 discovery, (d) referensi baru yang terverifikasi, dan (e) konsistensi hipotesis H1a-H1c yang telah di-refine.

## Pencapaian & Output

### 1. Revisi Komprehensif Progress Report
Seluruh section thesis telah direvisi dengan 3-view quality assurance (Academic Advisor, IEEE Reviewer, Senior Researcher):
* **Section 2 (Background):** Exploit examples di-reframe agar factually accurate.
* **Section 3 (Literature Review):** 3 gap subsections refocused (gap only, no CANB details). Tambah Wang ICSE 2026, Zhang ICSE 2025, Golaszewski SSR 2025.
* **Section 4 (Problem Statement):** P2 di-restructure ke Reactive Defense Gap. H1a fixed (100% within test suite, bukan ≥99.9%).
* **Section 5 (Methodology):** NEW Section 5.1.1 (CB formalization + time-bounded nonce + security argument). Section 5.2 updated dengan temporal check. Section 5.4 tambah Freshness Window Optimization.
* **Section 6 (Threat Model):** AV-1-1 ditambahkan. STRIDE table updated. Attacker capability "Delayed Replay" ditambahkan.
* **Section 7-9:** Cascade updates untuk konsistensi.

### 2. Bibliography Finalized
Total 34 entries, semua terverifikasi:
* 5 fabricated entries dari versi awal telah diganti dengan sumber valid.
* 5 entry baru ditambahkan (Golaszewski, Wang, Zhang, EIP-712, NIST 800-185).
* Brace balance: 273/273, zero syntax errors.

### 3. Presentasi HTML Disiapkan
Slide presentasi 16:9 (14 slides) yang menjawab seluruh pertanyaan penguji dari seminar sebelumnya telah disiapkan.

### 4. Kesiapan Deployment (W15+)
Artefact CANB fully configured untuk deployment ke:
* Base Sepolia (Optimism L2), Lisk Sepolia (EVM Sidechain), Kaia Kairos (Klaytn L1 BFT).
* Multi-provider RPC failover dan chain-specific finality thresholds telah dikonfigurasi.

## Status Keseluruhan
| Fase | Status |
|------|--------|
| Gap Analysis & Problem Formalization | ✅ Completed |
| Threat Modelling & Requirements | ✅ Completed |
| Artefact Engineering (15 Contracts) | ✅ Completed |
| Attack Vector Formulation (AV-1–5, AV-1-1, AV-2-1) | ✅ Completed |
| Formal Security Analysis (CB, Time-Bounded Nonce) | ✅ Completed |
| Testnet Deployment | 🔄 In Progress |
| Empirical Evaluation (B1-B3) | 📋 Planned |
| Statistical Analysis | 📋 Planned |
| Thesis Finalization | 📋 Planned |
