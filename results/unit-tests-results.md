# Hasil Pengujian (Dry-Run): Unit Tests

Dokumen ini berisi log eksekusi langsung dari Foundry untuk membuktikan bahwa *smart contract* merespons sesuai dengan skenario keamanan yang dirancang.

## Output Log

```text
No files changed, compilation skipped

Ran 6 tests for test/unit/DomainSeparator.t.sol:DomainSeparatorTest
[PASS] test_DomainSeparator_CapturesChainId() (gas: 5451)
[PASS] test_DomainSeparator_DiffersAcrossChains() (gas: 152600)
[PASS] test_DomainSeparator_DiffersAcrossContracts() (gas: 149735)
[PASS] test_DomainSeparator_IsImmutable() (gas: 6496)
[PASS] test_DomainSeparator_IsNonZero() (gas: 5537)
[PASS] test_DomainSeparator_MatchesManualComputation() (gas: 6184)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 905.81µs (573.30µs CPU time)

Ran 8 tests for test/unit/ContextValidator.t.sol:ContextValidatorTest
[PASS] test_RevertOnChainIdMismatch() (gas: 9789)
[PASS] test_RevertOnInvalidMessageType_BURN() (gas: 9845)
[PASS] test_RevertOnInvalidMessageType_LOCK() (gas: 9713)
[PASS] test_RevertOnSameChainRoute() (gas: 9574)
[PASS] test_RevertOnZeroDestinationChainId() (gas: 9299)
[PASS] test_RevertOnZeroSourceChainId() (gas: 9579)
[PASS] test_ValidContext_MINT() (gas: 6540)
[PASS] test_ValidContext_RELEASE() (gas: 6619)
Suite result: ok. 8 passed; 0 failed; 0 skipped; finished in 922.59µs (356.88µs CPU time)

Ran 6 tests for test/unit/BridgeExecutor.t.sol:BridgeExecutorTest
[PASS] test_MintTokens_RevertOnNonGateway() (gas: 15806)
[PASS] test_MintTokens_RevertOnZeroAddress() (gas: 15760)
[PASS] test_MintTokens_RevertOnZeroAmount() (gas: 17797)
[PASS] test_MintTokens_Success() (gas: 75940)
[PASS] test_ReleaseTokens_RevertOnInsufficient() (gas: 42331)
[PASS] test_ReleaseTokens_Success() (gas: 107383)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 1.12ms (456.20µs CPU time)

Ran 6 tests for test/unit/ContextHasher.t.sol:ContextHasherTest
[PASS] test_ContextHash_DiffersAcrossChains() (gas: 345914)
[PASS] test_ContextHash_DiffersOnMessageType() (gas: 13251)
[PASS] test_ContextHash_IncludesDomainSeparator() (gas: 14948)
[PASS] test_StructHash_Deterministic() (gas: 8327)
[PASS] test_StructHash_DiffersOnFieldChange() (gas: 8715)
[PASS] test_StructHash_DiffersOnNonceChange() (gas: 8825)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 2.28ms (539.28µs CPU time)

Ran 6 tests for test/unit/ValidationModule.t.sol:ValidationModuleTest
[PASS] test_RevertOnReplayNonce() (gas: 103922)
[PASS] test_RevertOnUnauthorizedRelayer() (gas: 37642)
[PASS] test_RevertOnWrongChainDestination() (gas: 28425)
[PASS] test_ValidateMessage_FullPipeline() (gas: 92331)
[PASS] test_VerifyMessage_DryRun() (gas: 38313)
[PASS] test_VerifyMessage_ReturnsFalseForUsedNonce() (gas: 101747)
Suite result: ok. 6 passed; 0 failed; 0 skipped; finished in 2.50ms (7.27ms CPU time)

Ran 11 tests for test/unit/NonceStore.t.sol:NonceStoreTest
[PASS] test_BitmapBoundary_Nonce255to256() (gas: 1161145)
[PASS] test_ConsumeNonce_EmitsEvent() (gas: 63782)
[PASS] test_ConsumeNonce_RevertOnDuplicate() (gas: 64273)
[PASS] test_ConsumeNonce_RevertOnSkip() (gas: 63856)
[PASS] test_ConsumeNonce_RevertOnZero() (gas: 19827)
[PASS] test_ConsumeNonce_SequentialMultiple() (gas: 83743)
[PASS] test_ConsumeNonce_Success() (gas: 60302)
[PASS] test_CrossBridgeIsolation() (gas: 109388)
[PASS] test_CrossChainIsolation() (gas: 109150)
[PASS] test_NextExpectedNonce_Advances() (gas: 65555)
[PASS] test_OnlyAuthorized_CanConsume() (gas: 18099)
Suite result: ok. 11 passed; 0 failed; 0 skipped; finished in 4.37ms (4.78ms CPU time)

Ran 6 test suites in 15.02ms (12.10ms CPU time): 43 tests passed, 0 failed, 0 skipped (43 total tests)

```
