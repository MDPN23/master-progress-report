# AV-1-1: Time-Delayed Replay Attack (Sleeping Replay)

## Klasifikasi Ancaman
| Properti | Nilai |
|---|---|
| **Attack Vector** | AV-1-1 |
| **Nama** | Time-Delayed Replay / Sleeping Replay |
| **STRIDE Category** | Spoofing + Tampering |
| **CIAAN Properties** | Authentication + Integrity |
| **Severity** | High |
| **Discovered** | Minggu 12 — Formal Nonce Lifecycle Analysis |

---

## Deskripsi

Time-Delayed Replay (atau *Sleeping Replay*) adalah varian dari Same-Context Replay (AV-1) yang mengeksploitasi **ketiadaan batasan temporal** pada nonce. Signature yang valid dapat di-replay **kapan saja di masa depan** selama nonce belum dikonsumsi.

Attacker menunda eksekusi replay hingga kondisi menguntungkan — misalnya harga token naik atau bridge memiliki likuiditas lebih besar.

### Motivasi Akademis
Dimotivasikan oleh temuan **Golaszewski et al. (SSR 2025)** yang mendemonstrasikan bahwa *cryptographic binding* yang optional/incomplete di FIDO UAF menghasilkan *session transfer attacks* serupa.

---

## Mekanisme Serangan

1. Attacker mengobservasi transaksi lock beserta signature
2. Attacker **menahan signature** dan menunggu (sleeping) — bisa berhari/berbulan
3. Ketika kondisi menguntungkan, attacker submit signature usang
4. Tanpa temporal constraint, signature diterima

## Exploit Condition

Bridge hanya mengecek:
- ✅ Signature validity (ECDSA recovery)
- ✅ Nonce freshness (bitmap check)
- ❌ **Temporal freshness (ABSENT)**

---

## Mitigasi CANB: Time-Bounded Nonce Construction

CANB memperkenalkan **temporal freshness constraint**:

```solidity
function validNonce(
    uint256 chainId, bytes32 bridgeId, uint256 nonce,
    uint256 messageTimestamp, uint256 freshnessWindow
) external view returns (bool) {
    uint256 bucket = nonce / 256;
    uint256 bitPos = nonce % 256;
    uint256 mask = 1 << bitPos;
    require(_bitmaps[chainId][bridgeId][bucket] & mask == 0, "Nonce consumed");
    require(block.timestamp <= messageTimestamp + freshnessWindow, "Nonce expired");
    return true;
}
```

### Formal Definition
Nonce valid iff:
1. `slot[n/256] & (1 << n%256) = 0` (belum dikonsumsi)
2. `block.timestamp ≤ timestamp + T` (dalam freshness window)

### Parameter T
- T ditentukan empiris dari distribusi legitimate lock-to-claim latency (persentil ke-99 + safety margin)
- Kandidat: `{60, 300, 600, 1800, 3600}` detik

---

## Kontribusi Novel

> [!IMPORTANT]
> AV-1-1 dan time-bounded nonce merupakan **novel engineering contribution** — absent dari seluruh 13 studi yang di-review.

## Referensi
- Golaszewski et al. (SSR 2025) — Formal-methods analysis of FIDO UAF channel binding
- Wang et al. (ICSE 2026) — Signature replay vulnerability taxonomy
- Zhang et al. (ICSE 2025) — Cryptographic defects in Ethereum smart contracts
