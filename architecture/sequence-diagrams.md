# Sequence Diagram: CANB Validation Pipeline

Dokumen ini memberikan representasi visual dari arsitektur *backbone* CANB, memperlihatkan bagaimana sebuah pesan dari pengguna luar divalidasi sebelum mencairkan atau mencetak token.

## Alur Eksekusi ValidationModule

Pipeline validasi CANB menerapkan pemisahan tugas (Separation of Concerns) yang ketat melalui beberapa sub-modul keamanan.

```mermaid
sequenceDiagram
    autonumber
    actor Relayer
    participant Gateway as CANBGateway
    participant Validator as ContextValidator
    participant Hasher as ContextHasher
    participant Nonce as NonceStore
    participant Executor as BridgeExecutor

    Relayer->>Gateway: processMessage(message, signature)
    Gateway->>ValidationModule: validateMessage(message, signature)
    
    rect rgb(240, 248, 255)
        Note over ValidationModule, Validator: Tahap 1: Validasi Rute Logis
        ValidationModule->>Validator: validateContext(message)
        Validator-->>ValidationModule: OK (Rute & Tipe Pesan Valid)
    end

    rect rgb(250, 240, 255)
        Note over ValidationModule, Hasher: Tahap 2: Pembentukan Hash EIP-712
        ValidationModule->>Hasher: computeContextHash(message)
        Hasher-->>ValidationModule: returns bytes32 contextHash
    end

    rect rgb(255, 240, 240)
        Note over ValidationModule: Tahap 3: Pemulihan Tanda Tangan
        ValidationModule->>ValidationModule: ECDSA.recover(contextHash, signature)
        ValidationModule->>ValidationModule: Cek authorizedRelayers[recovered]
    end

    rect rgb(240, 255, 240)
        Note over ValidationModule, Nonce: Tahap 4: Konsumsi Nonce (Anti-Replay)
        ValidationModule->>Nonce: consumeNonce(chainId, bridgeId, nonce)
        Nonce-->>ValidationModule: returns true (Bit=1)
    end

    ValidationModule-->>Gateway: Validation Passed (true, contextHash)

    rect rgb(255, 255, 240)
        Note over Gateway, Executor: Tahap Eksekusi (CEI Pattern)
        Gateway->>Executor: mintTokens / releaseTokens (berdasarkan Tipe)
        Executor->>Executor: State Update (Checks-Effects)
        Executor->>Token: Eksekusi Interaksi Token
    end
```

## Keunggulan Arsitektur Ini
1.  **Pemisahan Tugas:** Keamanan komputasi *hash*, verifikasi *signature*, dan konsumsi state terisolasi. Jika satu pola logika berubah, modul lainnya tetap terlindungi.
2.  **Anti-Reentrancy by Design:** CANB tidak pernah melakukan pemanggilan *external token* sebelum seluruh tahap validasi (sampai `consumeNonce`) selesai.
3.  **Transparansi Eksekusi:** Setiap langkah akan dibatalkan (*revert*) secara atomik jika ada satu tahap yang gagal (misalnya, alamat relayer tidak terdaftar, nonce diulang, dll).
