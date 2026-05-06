# Hasil Pengujian (Dry-Run): Attack Tests

Dokumen ini berisi log eksekusi langsung dari Foundry untuk membuktikan bahwa *smart contract* merespons sesuai dengan skenario keamanan yang dirancang.

## Output Log

```text
Compiling 5 files with Solc 0.8.30
Solc 0.8.30 finished in 10.48s
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


Ran 6 tests for test/attack/MessageTypeConfusion.t.sol:MessageTypeConfusionTest
[PASS] test_Baseline_AcceptsBurnTypeInbound() (gas: 20814)
[PASS] test_Baseline_AcceptsLockTypeInbound() (gas: 20770)
[PASS] test_Baseline_TypeConfusion_DifferentTypesSameMsg() (gas: 29804)
[PASS] test_CANB_AcceptsMintType() (gas: 146714)
[PASS] test_CANB_RejectsBurnTypeInbound() (gas: 36634)
[PASS] test_CANB_RejectsLockTypeInbound() (gas: 36703)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 6.80ms (4.14ms CPU time)

Ran 8 tests for test/attack/CrossContextReplay.t.sol:CrossContextReplayTest
[PASS] test_Baseline_CrossChainReplay_Succeeds() (gas: 35997)
[PASS] test_Baseline_LegitOnChainA() (gas: 21583)
[PASS] test_Baseline_ModifiedDestination_StillWorks() (gas: 29476)
[PASS] test_Baseline_SameSignatureBothChains() (gas: 36307)
[PASS] test_CANB_ChainASigInvalidOnChainB() (gas: 53292)
[PASS] test_CANB_CrossChainReplay_Reverts() (gas: 182755)
[PASS] test_CANB_DomainSeparatorsDiffer() (gas: 17625)
[PASS] test_CANB_LegitOnChainA() (gas: 147800)
Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 7.17ms (7.29ms CPU time)

Ran 7 tests for test/attack/CrossBridgeReplay.t.sol:CrossBridgeReplayTest
[PASS] test_Baseline_CrossBridgeReplay_Succeeds() (gas: 35079)
[PASS] test_Baseline_SigValidOnAnyInstance() (gas: 35233)
[PASS] test_Baseline_UnlimitedCrossBridge() (gas: 96075)
[PASS] test_CANB_CrossBridgeReplay_Reverts() (gas: 181829)
[PASS] test_CANB_DomainSeparatorsDifferPerInstance() (gas: 17449)
[PASS] test_CANB_Instance2_NeedsOwnSig() (gas: 147158)
[PASS] test_CANB_WrongInstanceSig_Reverts() (gas: 52363)
Suite result: ok. 7 passed; 0 failed; 0 skipped; finished in 8.88ms (8.82ms CPU time)

Ran 8 tests for test/attack/SameContextReplay.t.sol:SameContextReplayTest
[PASS] test_Baseline_FirstProcess_Succeeds() (gas: 21074)
[PASS] test_Baseline_SameReplay_Succeeds() (gas: 28608)
[PASS] test_Baseline_TripleReplay_Succeeds() (gas: 36001)
[PASS] test_Baseline_UnboundedReplay_10Times() (gas: 90119)
[PASS] test_CANB_FirstProcess_Succeeds() (gas: 147288)
[PASS] test_CANB_SameReplay_Reverts() (gas: 160710)
[PASS] test_CANB_SequentialNonces_Success() (gas: 258799)
[PASS] test_CANB_StaleSignature_Reverts() (gas: 163853)
Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 10.66ms (25.90ms CPU time)

Ran 7 tests for test/attack/NonceOrderingAttack.t.sol:NonceOrderingAttackTest
[PASS] test_BackwardNonce_Reverts() (gas: 219675)
[PASS] test_LargeNonceGap_Reverts() (gas: 166136)
[PASS] test_NoncePersistsAfterRevert() (gas: 191880)
[PASS] test_NonceSkip_Reverts() (gas: 57185)
[PASS] test_NonceZero_Reverts() (gas: 56917)
[PASS] test_OutOfOrder_Reverts() (gas: 164140)
[PASS] test_SequentialNonces_Succeed() (gas: 396528)
Suite result: ok. 7 passed; 0 failed; 0 skipped; finished in 11.40ms (33.61ms CPU time)

Ran 5 test suites in 17.34ms (44.91ms CPU time): 36 tests passed, 0 failed, 0 skipped (36 total tests)

```
