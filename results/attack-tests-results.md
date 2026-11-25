# Hasil Pengujian (Dry-Run): Attack Tests (47/47 Passed)

Dokumen ini berisi log eksekusi langsung dari Foundry (`forge 1.7.1`, `Solc 0.8.30`) untuk **7 attack test suites (47 test cases)** — mencakup verifikasi 4-Tier (`B0`, `B1`, `B2`, `CANB`) untuk `AV-1`, `AV-1-1` (*Sleeping Replay*), `AV-2`, `AV-2-1`, `AV-4`, `AV-5`, dan invarian konservasi aset **DV2** ($\Delta\text{Assets} = 0$).

## Output Log (`forge test --match-path "test/attack/*"`)

```text
Ran 5 tests for test/attack/SleepingReplay.t.sol:SleepingReplayTest
[PASS] test_AV1_1_B0_AcceptsSleepingSignature() (gas: 90411)
[PASS] test_AV1_1_B1_AcceptsSleepingSignature() (gas: 113294)
[PASS] test_AV1_1_B2_ModernStandard_AcceptsSleepingSignature() (gas: 115119)
[PASS] test_AV1_1_CANB_BlocksSleepingSignature() (gas: 68991)
[PASS] test_AV1_1_CANB_ExactBoundaryCheck() (gas: 195256)
Suite result: ok. 5 passed; 0 failed; 0 skipped; finished in 6.35ms (3.38ms CPU time)

Ran 6 tests for test/attack/MessageTypeConfusion.t.sol:MessageTypeConfusionTest
[PASS] test_Baseline_AcceptsBurnTypeInbound() (gas: 66841)
[PASS] test_Baseline_AcceptsLockTypeInbound() (gas: 66797)
[PASS] test_Baseline_TypeConfusion_DifferentTypesSameMsg() (gas: 82137)
[PASS] test_CANB_AcceptsMintType() (gas: 149274)
[PASS] test_CANB_RejectsBurnTypeInbound() (gas: 36881)
[PASS] test_CANB_RejectsLockTypeInbound() (gas: 36950)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 13.61ms (8.42ms CPU time)

Ran 8 tests for test/attack/SameContextReplay.t.sol:SameContextReplayTest
[PASS] test_Baseline_FirstProcess_Succeeds() (gas: 67118)
[PASS] test_Baseline_SameReplay_Succeeds() (gas: 80958)
[PASS] test_Baseline_TripleReplay_Succeeds() (gas: 94657)
[PASS] test_Baseline_UnboundedReplay_10Times() (gas: 192917)
[PASS] test_CANB_FirstProcess_Succeeds() (gas: 149866)
[PASS] test_CANB_SameReplay_Reverts() (gas: 163796)
[PASS] test_CANB_SequentialNonces_Success() (gas: 263683)
[PASS] test_CANB_StaleSignature_Reverts() (gas: 166691)
Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 13.75ms (21.87ms CPU time)

Ran 8 tests for test/attack/CrossContextReplay.t.sol:CrossContextReplayTest
[PASS] test_Baseline_CrossChainReplay_Succeeds() (gas: 128047)
[PASS] test_Baseline_LegitOnChainA() (gas: 67627)
[PASS] test_Baseline_ModifiedDestination_StillWorks() (gas: 81826)
[PASS] test_Baseline_SameSignatureBothChains() (gas: 128357)
[PASS] test_CANB_ChainASigInvalidOnChainB() (gas: 53586)
[PASS] test_CANB_CrossChainReplay_Reverts() (gas: 185557)
[PASS] test_CANB_DomainSeparatorsDiffer() (gas: 17625)
[PASS] test_CANB_LegitOnChainA() (gas: 150378)
Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 14.37ms (12.95ms CPU time)

Ran 6 tests for test/attack/FourTierSecurityMatrix.t.sol:FourTierSecurityMatrixTest
[PASS] test_DV2_AssetConservation_DeltaZero() (gas: 518504)
[PASS] test_Matrix_AV1_SameContextReplay() (gas: 501673)
[PASS] test_Matrix_AV2_1_CrossBridgeReplay() (gas: 551488)
[PASS] test_Matrix_AV2_CrossChainReplay() (gas: 529159)
[PASS] test_Matrix_AV4_MessageTypeConfusion() (gas: 259997)
[PASS] test_Matrix_AV5_OutOfOrderNonce() (gas: 282835)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 14.38ms (20.05ms CPU time)

Ran 7 tests for test/attack/CrossBridgeReplay.t.sol:CrossBridgeReplayTest
[PASS] test_Baseline_CrossBridgeReplay_Succeeds() (gas: 127129)
[PASS] test_Baseline_SigValidOnAnyInstance() (gas: 127283)
[PASS] test_Baseline_UnlimitedCrossBridge() (gas: 238573)
[PASS] test_CANB_CrossBridgeReplay_Reverts() (gas: 184631)
[PASS] test_CANB_DomainSeparatorsDifferPerInstance() (gas: 17449)
[PASS] test_CANB_Instance2_NeedsOwnSig() (gas: 149736)
[PASS] test_CANB_WrongInstanceSig_Reverts() (gas: 52657)
Suite result: ok. 7 passed; 0 failed; 0 skipped; finished in 14.50ms (12.61ms CPU time)

Ran 7 tests for test/attack/NonceOrderingAttack.t.sol:NonceOrderingAttackTest
[PASS] test_BackwardNonce_Reverts() (gas: 223982)
[PASS] test_LargeNonceGap_Reverts() (gas: 169290)
[PASS] test_NoncePersistsAfterRevert() (gas: 195611)
[PASS] test_NonceSkip_Reverts() (gas: 59763)
[PASS] test_NonceZero_Reverts() (gas: 59495)
[PASS] test_OutOfOrder_Reverts() (gas: 167294)
[PASS] test_SequentialNonces_Succeed() (gas: 404303)
Suite result: ok. 7 passed; 0 failed; 0 skipped; finished in 17.15ms (40.36ms CPU time)

Ran 7 test suites in 18.41ms (94.11ms CPU time): 47 tests passed, 0 failed, 0 skipped (47 total tests)
```
