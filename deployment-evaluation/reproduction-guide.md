# Panduan Reproduksi (Quick Start Guide)

Dokumen ini ditujukan bagi penguji, kontributor, atau pengembang yang ingin menjalankan pengujian simulasi keamanan pada CANB Framework secara mandiri di komputer lokal.

Penelitian ini sepenuhnya didasarkan pada kerangka pengujian **Foundry**. Semua pengujian serangan (*threat modelling*) dapat dibuktikan kebenarannya secara deterministik.

## 1. Persiapan Lingkungan (Prerequisites)

Pastikan Anda telah menginstal Foundry. Jika belum, buka terminal dan jalankan:
```bash
curl -L https://foundry.paradigm.xyz | bash
```
Kemudian jalankan perintah `foundryup` untuk memastikan Anda memiliki versi terbaru.

## 2. Kloning Repositori
Masuk ke dalam direktori kerja dan kloning repositori tesis:
```bash
git clone <URL_REPOSITORI_ANDA> thesis-canb
cd thesis-canb/ccBridge
```

## 3. Menjalankan Kompilasi
Kompilasi seluruh *smart contract* untuk memastikan tidak ada kesalahan sintaks:
```bash
forge build
```

## 4. Menjalankan Simulasi Serangan (Attack Vectors)
Anda dapat memverifikasi bahwa *Baseline Bridge* gagal menahan serangan sementara *CANB Gateway* berhasil memblokirnya, dengan menjalankan Test Suite Serangan.

Jalankan perintah berikut:
```bash
forge test --match-path test/attack/* -vv
```

*Expected Output:*
Semua pengujian dengan penamaan `test_CANB_*_Reverts` akan lulus (*PASS*), yang berarti serangan tersebut berhasil dicegah. Sementara tes bernama `test_Baseline_*` akan berhasil mengeksploitasi kontrak (simulasi serangan berjalan).

## 5. Menjalankan Unit Test Penuh
Untuk memeriksa fungsi atomik dari setiap komponen arsitektur CANB (NonceStore, DomainSeparator, dll):
```bash
forge test --match-path test/unit/* -vv
```

## 6. Menjalankan Gas Benchmark (Evaluasi Performa)
Untuk melihat secara persis perbandingan penggunaan gas (Gwei/USD) antara sistem Baseline dan CANB:
```bash
forge test --match-contract ComparisonBenchmark --gas-report
```

## 7. Melakukan Deployment Simulasi (Opsional)
Jika Anda ingin menyebarkan kontrak ini ke *local node* (Anvil):
1. Buka terminal baru dan jalankan `anvil`.
2. Di terminal utama, jalankan skrip deployment:
```bash
forge script script/DeployAll.s.sol:DeployAll --rpc-url http://localhost:8545 --broadcast
```
