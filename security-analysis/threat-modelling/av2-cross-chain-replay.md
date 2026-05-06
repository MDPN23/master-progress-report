# AV-2: Cross-Chain Replay Attack

## Deskripsi Vektor Serangan
Berbeda dengan AV-1, pada serangan *Cross-Chain Replay*, penyerang menggunakan *signature* sah yang ditujukan untuk **Chain A**, namun ia mengirimkannya (*submit*) ke instansiasi *bridge* yang berada di **Chain B** atau **Chain C**. 

Jika *smart contract* tidak mengikat pesan dengan identitas jaringan (*Chain ID*), *bridge* di rantai lain akan menganggap *signature* tersebut valid dan mencairkan aset yang seharusnya tidak ada.

## Diagram Simulasi Serangan (Baseline vs CANB)

```mermaid
sequenceDiagram
    autonumber
    actor Attacker
    participant Relayer
    participant BridgeB as Bridge di Polygon (Chain 137)
    participant BridgeC as Bridge di Arbitrum (Chain 42161)

    Attacker->>Relayer: Lock 100 TELU to Polygon
    Relayer-->>Attacker: Generate Signature (S_Poly)
    
    Note over Attacker, BridgeB: Klaim Sah di Polygon
    Attacker->>BridgeB: claim(S_Poly)
    BridgeB-->>Attacker: ✅ Mint 100 wTELU
    
    rect rgb(255, 235, 235)
        Note over Attacker, BridgeC: Skenario Baseline (Rentang)
        Attacker->>BridgeC: claim(S_Poly) [CROSS-CHAIN REPLAY]
        BridgeC->>BridgeC: Cek ECDSA tanpa Chain ID
        BridgeC-->>Attacker: ❌ Mint 100 wTELU di Arbitrum (ILLEGAL!)
    end

    rect rgb(235, 255, 235)
        Note over Attacker, BridgeC: Skenario CANB (Aman)
        Attacker->>BridgeC: processMessage(S_Poly) [CROSS-CHAIN REPLAY]
        BridgeC->>BridgeC: ContextValidator.validateContext()
        BridgeC-->>Attacker: ❌ REVERT (ChainIdMismatch / Invalid Context)
    end
```

## Analisis Mitigasi CANB
Serangan AV-2 dimitigasi secara penuh menggunakan dua lapisan pertahanan ganda:
1.  **ContextHasher & DomainSeparator (EIP-712):** Mengikat tanda tangan kriptografis dengan `block.chainid` tempat kontrak di-*deploy*.
2.  **ContextValidator:** Secara eksplisit menolak (REVERT) pesan masuk yang mana `destinationChainId` tidak sesuai dengan parameter lokal rantai yang sedang berjalan.
