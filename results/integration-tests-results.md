# Hasil Pengujian (Dry-Run): Integration Tests

Dokumen ini berisi log eksekusi langsung dari Foundry untuk membuktikan bahwa *smart contract* merespons sesuai dengan skenario keamanan yang dirancang.

## Output Log

```text
Compiling 3 files with Solc 0.8.30
Solc 0.8.30 finished in 6.29s
Compiler run successful with warnings:
Warning (5667): Unused function parameter. Remove or comment out the variable name to silence this warning.
  --> src/baseline/BaselineGateway.sol:97:9:
   |
97 |         uint256 destinationChainId
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^

Warning (5667): Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> src/baseline/BaselineGateway.sol:120:9:
    |
120 |         uint256 destinationChainId
    |         ^^^^^^^^^^^^^^^^^^^^^^^^^^


Ran 2 tests for test/integration/BaselineBridge.t.sol:BaselineBridgeTest
[PASS] test_FullFlow_LockAndProcess() (gas: 55776)
[PASS] test_ProcessMessage_Succeeds() (gas: 24487)
Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 2.34ms (1.50ms CPU time)

Ran 4 tests for test/integration/ComparisonBenchmark.t.sol:ComparisonBenchmarkTest
[PASS] test_Baseline_LockAssets() (gas: 54808)
[PASS] test_Baseline_ProcessMessage() (gas: 24778)
[PASS] test_CANB_LockAssets() (gas: 107120)
[PASS] test_CANB_ProcessMessage() (gas: 134109)
Suite result: ok. 4 passed; 0 failed; 0 skipped; finished in 2.37ms (2.02ms CPU time)

Ran 2 tests for test/integration/CANBBridge.t.sol:CANBBridgeTest
[PASS] test_FullFlow_MintViaProcessMessage() (gas: 152176)
[PASS] test_FullFlow_SequentialMessages() (gas: 207796)
Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 3.60ms (3.87ms CPU time)

Ran 3 test suites in 10.10ms (8.31ms CPU time): 8 tests passed, 0 failed, 0 skipped (8 total tests)

```
