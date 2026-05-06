# AV-1: Same-Chain Replay Attack

## Deskripsi Vektor Serangan
Dalam skenario serangan ini, seorang pengguna (atau *attacker*) yang sah melakukan penguncian aset (lock) **satu kali** di *source chain*. Ketika *relayer* men-generate bukti kriptografis (*proof/signature*) untuk mencairkan token di *destination chain*, penyerang menangkap *signature* tersebut.

Setelah penyerang mengklaim aset secara sah, ia kembali **mengirimkan ulang (replay)** *signature* yang persis sama ke *smart contract* tujuan untuk mencetak (*mint*) token lagi secara ilegal.

## Diagram Simulasi Serangan (Baseline vs CANB)

```mermaid
sequenceDiagram
    autonumber
    actor Attacker
    participant Relayer
    participant Baseline as Baseline Bridge (Control)
    participant CANB as CANB Gateway (Treatment)

    Attacker->>Relayer: Lock 100 TELU (Source Chain)
    Relayer-->>Attacker: Generate Signature (S1)
    
    rect rgb(255, 235, 235)
        Note over Attacker, Baseline: Skenario Baseline (Rentang)
        Attacker->>Baseline: claimTokens(S1)
        Baseline-->>Attacker: ✅ Mint 100 wTELU
        Attacker->>Baseline: claimTokens(S1) [REPLAY]
        Baseline-->>Attacker: ❌ Mint 100 wTELU (DOUBLE MINT!)
    end

    rect rgb(235, 255, 235)
        Note over Attacker, CANB: Skenario CANB (Aman)
        Attacker->>CANB: processMessage(S1)
        CANB->>CANB: NonceStore.consumeNonce()
        CANB-->>Attacker: ✅ Mint 100 wTELU
        Attacker->>CANB: processMessage(S1) [REPLAY]
        CANB->>CANB: NonceStore.consumeNonce()
        CANB-->>Attacker: ❌ REVERT (NonceAlreadyConsumed)
    end
```

## Analisis Mitigasi CANB
CANB memitigasi serangan ini menggunakan modul `NonceStore`. Setiap *nonce* direpresentasikan dalam struktur *bitmap* biner (`uint256`). Ketika *signature* `S1` dieksekusi pertama kali, bit *nonce* akan diubah dari `0` menjadi `1`. Pada saat penyerang mencoba me-replay `S1`, `NonceStore` mengecek bit tersebut dan langsung menggagalkan transaksi (REVERT) karena mendeteksi *nonce* yang sama.
