# Progress Report - Minggu 10

**Tanggal:** 20 Mei 2026  
**Fokus Utama:** Desain Eksperimen Komparatif dan Framework Analisis Statistik.

## Ringkasan Aktivitas
Minggu ke-10 difokuskan pada finalisasi desain eksperimen three-phase protocol (B1, B2, B3) dan pemilihan metode analisis statistik yang tepat untuk setiap hipotesis.

## Pencapaian & Output

### 1. Finalisasi Three-Phase Experimental Protocol
* **Phase B1 (Control):** Protokol eksekusi transaksi legitimate tanpa adversarial input. Metrik: FRR (harus = 0), baseline gas, baseline latency.
* **Phase B2 (Attack):** Injeksi payload adversarial across 6 attack sub-vectors (AV-1 sampai AV-5, termasuk AV-2-1). Langsung menguji H1a dan H1b.
* **Phase B3 (Load):** Concurrent legitimate + adversarial traffic untuk mengukur degradasi throughput.

### 2. Statistical Analysis Framework
Memetakan setiap hipotesis ke metode statistik yang sesuai:
* **H1a → Binomial Test** (H₀: blocking rate = 50%, expected: 100%) + Clopper-Pearson 95% CI.
* **H1b → Empirical Invariant Verification** (ΔAssets = Σ Locked - Σ Minted = 0).
* **H1c → One-sample t-test** (H₀: μ_overhead ≥ 20%) + **One-way ANOVA** across 3 testnets.

### 3. Testbed Network Selection Justification
Mendokumentasikan alasan pemilihan 3 testnet yang architecturally distinct:
* Base Sepolia (Optimism L2 Rollup), Lisk Sepolia (EVM Sidechain), Kaia Kairos (Klaytn L1 BFT).
* Variasi finality mechanism dan gas pricing model memungkinkan ANOVA cross-chain consistency test.

## Rencana Minggu Selanjutnya (Minggu 11)
* Review akhir threat model STRIDE/CIAAN mapping.
* Memulai persiapan alignment proposal dengan framework DSR.
