# AV-3: Double Spending Attack

## Deskripsi Vektor Serangan
*Double Spending* (Pembelanjaan Ganda) adalah varian dari *replay attack* yang berfokus pada sisi temporal. Pada skenario ini, pengguna memiliki *signature* yang valid untuk pencairan aset. Penyerang menembakkan **dua transaksi secara paralel (atau berurutan dengan sangat cepat)** ke jaringan dengan *signature* yang sama. 

Tujuannya adalah untuk mengeksploitasi keterlambatan pembaruan *state* (*state update delay*) dari *smart contract*. Jika kontrak memperbarui nilai *balance* atau *nonce* SETELAH pencetakan token, ada risiko eksekusi paralel menyebabkan token dicetak ganda.

## Diagram Simulasi Serangan (Baseline vs CANB)

```mermaid
flowchart TD
    Lock["🔒 Attacker locks 100 TELU"]
    
    Lock --> ClaimA["Tx 1: claim(100 wTELU)"]
    Lock --> ClaimB["Tx 2: claim(100 wTELU)"]
    
    subgraph Baseline [Baseline Bridge]
        direction TB
        ClaimA --> ResultA{"Mint Success"}
        ClaimB --> ResultB{"Mint Success"}
        ResultA --> Total1["Total: 200 wTELU ❌"]
        ResultB --> Total1
    end

    subgraph CANB [CANB Framework]
        direction TB
        ClaimA --> ResultC{"Mint Success (Bit=1)"}
        ClaimB --> ResultD{"REVERT: Nonce bit is 1"}
        ResultC --> Total2["Total: 100 wTELU ✅"]
        ResultD --> Total2
    end
```

## Analisis Mitigasi CANB
CANB menangani AV-3 melalui implementasi ketat dari modul `BridgeExecutor` yang mengadopsi pola **Checks-Effects-Interactions (CEI)** dan modul `NonceStore`.
1.  `NonceStore` mengkonsumsi *nonce* (Checks & Effects) pada saat validasi SEBELUM eksekutor token dipanggil. 
2.  Karena penulisan memori EVM untuk struktur *bitmap* terjadi di level transaksional yang atomik, transaksi kedua yang dieksekusi pada *block* yang sama atau setelahnya akan secara matematis membaca bit *nonce* bernilai `1` dan langsung di-*revert*.
