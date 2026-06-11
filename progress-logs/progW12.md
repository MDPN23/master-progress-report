# Progress Report - Minggu 12

**Tanggal:** 3 Juni 2026  
**Fokus Utama:** Proposal Formalization dan Formal Security Analysis — Discovery AV-1-1.

## Ringkasan Aktivitas
Minggu ke-12 menandai awal fase kritis: alignment proposal dengan framework Technology Development (DSR) dan dimulainya analisis keamanan formal. Selama proses ini, analisis mendalam terhadap nonce lifecycle mengungkapkan attack vector yang sebelumnya belum teridentifikasi.

## Pencapaian & Output

### 1. Discovery: AV-1-1 (Time-Delayed Replay / Sleeping Replay)
Selama analisis formal terhadap nonce lifecycle, ditemukan bahwa implementasi nonce berbasis bitmap (dan semua implementasi bridge yang di-review) **tidak memiliki batasan temporal**. Signature yang valid dapat di-replay kapan saja di masa depan selama nonce belum dikonsumsi. Ini membuka *sleeping replay* attack dimana attacker menunggu kondisi yang menguntungkan sebelum replay.

**Implikasi:** Attack vector ini (AV-1-1) dimotivasikan langsung oleh temuan Golaszewski et al. (SSR 2025) yang mendemonstrasikan bahwa cryptographic binding yang optional/incomplete di FIDO UAF menghasilkan session transfer attacks serupa.

### 2. Motivasi Time-Bounded Nonce Construction
Discovery AV-1-1 memotivasi pengembangan **time-bounded nonce** — ekstensi bitmap nonce dengan constraint temporal:
* `block.timestamp ≤ C.timestamp + T`
* Parameter T menjadi tunable security-usability parameter.
* Ini merupakan novel engineering contribution yang absent dari seluruh bridge implementations yang di-review.

### 3. Literature Review: Formal Binding Analysis
Melakukan studi mendalam terhadap paper-paper yang mendukung formalisasi:
* Golaszewski et al. (2025): Formal-methods analysis of cryptographic binding di FIDO UAF.
* Wang et al. (ICSE 2026): Taxonomy 5 tipe Signature Replay Vulnerability (SRV).
* Zhang et al. (ICSE 2025): 9 tipe cryptographic defects termasuk SSR dan CSR.

## Rencana Minggu Selanjutnya (Minggu 13)
* Formalisasi Context Binding Function CB(σ, C).
* Penulisan Security Argument (3 properties + proof sketch).
