# 📋 Unit Test Suite Documentation

**Penelitian:** *Mitigation Framework for Double-Spending and Replay Attacks in DeFi Cross-Chain Bridges*
**Total Tests:** 43 unit test functions across 6 test files
**Framework:** Foundry (forge test)
**Coverage Target:** ≥95%

---

## 1. Test File Structure

```
test/unit/
├── NonceStore.t.sol       — 11 test functions
├── DomainSeparator.t.sol  — 6 test functions
├── ContextHasher.t.sol    — 6 test functions
├── ContextValidator.t.sol — 8 test functions
├── ValidationModule.t.sol — 6 test functions
└── BridgeExecutor.t.sol   — 6 test functions
```

---

## 2. Test Breakdown Per Contract

### 2.1 NonceStore.t.sol (11 tests)

| # | Test Function | Category | Deskripsi |
|:-:|:--|:--|:--|
| 1 | `test_ConsumeNonce_Success` | Success | Consume nonce 1 berhasil, bitmap ter-set |
| 2 | `test_ConsumeNonce_SequentialMultiple` | Success | Consume nonce 1–5 secara berurutan |
| 3 | `test_ConsumeNonce_EmitsEvent` | Success | Verify `NonceConsumed` event di-emit |
| 4 | `test_NextExpectedNonce_Advances` | Success | Counter `nextExpectedNonce` naik setelah consume |
| 5 | `test_ConsumeNonce_RevertOnDuplicate` | Revert | Nonce yang sama di-reject (`NonceAlreadyConsumed`) |
| 6 | `test_ConsumeNonce_RevertOnSkip` | Revert | Skip nonce 2 → revert (`NonceOutOfOrder`) |
| 7 | `test_ConsumeNonce_RevertOnZero` | Revert | Nonce 0 di-reject (expected 1) |
| 8 | `test_BitmapBoundary_Nonce255to256` | Boundary | Test transisi antar bitmap bucket (255→256) |
| 9 | `test_CrossChainIsolation` | Isolation | Nonce 1 di chainId berbeda tidak bertabrakan |
| 10 | `test_CrossBridgeIsolation` | Isolation | Nonce 1 di bridgeId berbeda tidak bertabrakan |
| 11 | `test_OnlyAuthorized_CanConsume` | Access | Unauthorized address tidak bisa consume |

### 2.2 DomainSeparator.t.sol (6 tests)

| # | Test Function | Category | Deskripsi |
|:-:|:--|:--|:--|
| 1 | `test_DomainSeparator_IsNonZero` | Value | Separator bukan bytes32(0) |
| 2 | `test_DomainSeparator_IsImmutable` | Immutable | Dua kali panggil → nilai sama |
| 3 | `test_DomainSeparator_CapturesChainId` | ChainId | `DEPLOYMENT_CHAIN_ID` == `block.chainid` |
| 4 | `test_DomainSeparator_DiffersAcrossChains` | Cross-chain | Deploy di chain berbeda → separator berbeda |
| 5 | `test_DomainSeparator_DiffersAcrossContracts` | Cross-contract | Dua instance → separator berbeda |
| 6 | `test_DomainSeparator_MatchesManualComputation` | EIP-712 | Output cocok dengan manual keccak256 computation |

### 2.3 ContextHasher.t.sol (6 tests)

| # | Test Function | Category | Deskripsi |
|:-:|:--|:--|:--|
| 1 | `test_StructHash_Deterministic` | Determinism | Hash yang sama untuk input yang sama |
| 2 | `test_StructHash_DiffersOnFieldChange` | Sensitivity | Beda amount → beda hash |
| 3 | `test_StructHash_DiffersOnNonceChange` | Sensitivity | Beda nonce → beda hash |
| 4 | `test_ContextHash_IncludesDomainSeparator` | EIP-712 | Context hash = `keccak256(\x19\x01 ‖ DS ‖ struct)` |
| 5 | `test_ContextHash_DiffersAcrossChains` | Cross-chain | Deploy beda chain → beda context hash |
| 6 | `test_ContextHash_DiffersOnMessageType` | Sensitivity | MINT vs RELEASE → beda hash |

### 2.4 ContextValidator.t.sol (8 tests)

| # | Test Function | Category | Deskripsi |
|:-:|:--|:--|:--|
| 1 | `test_ValidContext_MINT` | Success | MINT message valid → no revert |
| 2 | `test_ValidContext_RELEASE` | Success | RELEASE message valid → no revert |
| 3 | `test_RevertOnZeroSourceChainId` | Revert | sourceChainId = 0 → `InvalidChainId` |
| 4 | `test_RevertOnZeroDestinationChainId` | Revert | destinationChainId = 0 → `InvalidChainId` |
| 5 | `test_RevertOnSameChainRoute` | Revert | source == destination → `SameChainRoute` |
| 6 | `test_RevertOnChainIdMismatch` | Revert | destination ≠ deployment → `ChainIdMismatch` |
| 7 | `test_RevertOnInvalidMessageType_LOCK` | Revert | LOCK type inbound → `InvalidMessageType` |
| 8 | `test_RevertOnInvalidMessageType_BURN` | Revert | BURN type inbound → `InvalidMessageType` |

### 2.5 ValidationModule.t.sol (6 tests)

| # | Test Function | Category | Deskripsi |
|:-:|:--|:--|:--|
| 1 | `test_ValidateMessage_FullPipeline` | Success | Full 4-stage pipeline berhasil |
| 2 | `test_VerifyMessage_DryRun` | Success | Dry-run tanpa consume nonce |
| 3 | `test_RevertOnUnauthorizedRelayer` | Auth | Signature dari non-relayer → revert |
| 4 | `test_RevertOnReplayNonce` | Replay | Nonce kedua kali → revert |
| 5 | `test_RevertOnWrongChainDestination` | Chain | destinationChainId salah → revert |
| 6 | `test_VerifyMessage_ReturnsFalseForUsedNonce` | State | Dry-run return false setelah nonce consumed |

### 2.6 BridgeExecutor.t.sol (6 tests)

| # | Test Function | Category | Deskripsi |
|:-:|:--|:--|:--|
| 1 | `test_MintTokens_Success` | Success | Mint 100 tokens → balance correct |
| 2 | `test_MintTokens_RevertOnZeroAmount` | Revert | amount=0 → `ZeroAmount` |
| 3 | `test_MintTokens_RevertOnZeroAddress` | Revert | to=address(0) → `ZeroAddress` |
| 4 | `test_MintTokens_RevertOnNonGateway` | Access | Non-gateway caller → revert |
| 5 | `test_ReleaseTokens_Success` | Success | Release locked tokens → balance updated |
| 6 | `test_ReleaseTokens_RevertOnInsufficient` | Revert | Release > locked → revert |

---

## 3. Running the Tests

```bash
# Run all unit tests
forge test --match-path "test/unit/*" -v

# Run specific contract tests
forge test --match-contract NonceStoreTest -vvv

# Run with gas report
forge test --match-path "test/unit/*" --gas-report
```

---

## 4. Coverage Verification

```bash
# Generate coverage report
forge coverage --match-path "test/unit/*"
```

| Contract | Target | Verified |
|:--|:--:|:--:|
| NonceStore | 100% | ✅ |
| DomainSeparator | 100% | ✅ |
| ContextHasher | 100% | ✅ |
| ContextValidator | 100% | ✅ |
| ValidationModule | 95% | ✅ |
| BridgeExecutor | 95% | ✅ |
