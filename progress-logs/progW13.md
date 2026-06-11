# Progress Report - Minggu 13

**Tanggal:** 10 Juni 2026  
**Fokus Utama:** Formalisasi Context Binding Function CB(σ, C) dan Security Argument.

## Ringkasan Aktivitas
Minggu ke-13 merupakan minggu paling signifikan secara akademis — penulisan definisi formal Context Binding Function dan security argument yang menjadi core knowledge contribution penelitian ini.

## Pencapaian & Output

### 1. Formal Definition: Context Binding Function CB(σ, C)
Mendefinisikan 4 definisi formal:
* **Definition 1 (Execution Context):** C = (chainId, bridgeId, contractAddr, sender, recipient, amount, nonce, timestamp)
* **Definition 2 (Domain Separator):** D berdasarkan EIP-712 typed structured data hashing dan NIST SP 800-185 domain separation standard.
* **Definition 3 (Context Hash):** H(C) = keccak256(0x19 || 0x01 || D || keccak256(MESSAGE_TYPEHASH || C))
* **Definition 4 (Context Binding Function):** CB(σ, C) = valid iff (1) authorized signer, (2) nonce fresh, (3) timestamp valid.

### 2. Time-Bounded Nonce Specification
Formalisasi validNonce() function dengan temporal freshness constraint:
* Bitmap check: slot[n/256] & (1 << n%256) = 0
* Temporal check: block.timestamp ≤ timestamp + T
* Dokumentasi trade-off parameter T dan rencana empirical optimization.

### 3. Security Argument — 3 Properties
Penulisan proof sketch untuk tiga security properties:
* **P1 (Domain Binding):** Signature valid di C₁ → invalid di C₂ dengan probability ≥ 1-2⁻²⁵⁶ (under keccak256 collision resistance).
* **P2 (Nonce Uniqueness + Freshness):** Monotonically increasing bitmap + temporal bound = deterministic rejection.
* **P3 (Asset Conservation):** P1 + P2 + CEI pattern → ΔAssets = 0.

### 4. Referensi Baru Ditambahkan ke Bibliography
* `golaszewski2025binding` — SSR 2025 (Springer LNCS)
* `wang2026signature` — ICSE 2026
* `zhang2025cryptodefects` — ICSE 2025
* `eip712` — Ethereum EIP-712 Specification
* `nist800185` — NIST SP 800-185

## Rencana Minggu Selanjutnya (Minggu 14)
* Finalisasi proposal alignment dengan DSR framework.
* Integrasi CB(σ, C) dan time-bounded nonce ke dalam seluruh section tesis.
