# 🛡️ Attack Test Suite Documentation

**Penelitian:** *Mitigation Framework for Double-Spending and Replay Attacks in DeFi Cross-Chain Bridges*
**Total Scenarios:** 36 attack scenarios across 5 test files
**Framework:** Foundry (forge test)

---

## 1. Test File Structure

```
test/attack/
├── SameContextReplay.t.sol      — 8 scenarios
├── CrossContextReplay.t.sol     — 8 scenarios
├── CrossBridgeReplay.t.sol      — 7 scenarios
├── MessageTypeConfusion.t.sol   — 6 scenarios
└── NonceOrderingAttack.t.sol    — 7 scenarios
```

---

## 2. Attack Type Summary

| # | Attack Type | File | Vulnerability Tested | CANB Protection |
|:-:|:--|:--|:--|:--|
| 1 | Same-Context Replay | `SameContextReplay.t.sol` | V1 (no replay guard) | P1 (nonce tracking) + P2 (hash registry) |
| 2 | Cross-Context Replay | `CrossContextReplay.t.sol` | V2 (no chainId binding) | P4 (EIP-712 domain separator) |
| 3 | Cross-Bridge Replay | `CrossBridgeReplay.t.sol` | V2 (no contract binding) | P4 (verifyingContract in domain) |
| 4 | Message Type Confusion | `MessageTypeConfusion.t.sol` | V3 (weak validation) | ContextValidator type filter |
| 5 | Nonce Ordering Attack | `NonceOrderingAttack.t.sol` | V1 (no ordering) | P1 (sequential enforcement) |

---

## 3. Scenario Breakdown

### 3.1 Same-Context Replay (8 scenarios)

**Objective:** Replay the exact same signed message on the same chain/bridge instance.

| # | Scenario | Target | Expected Result |
|:-:|:--|:--|:--|
| 1 | Baseline first process | BaselineGateway | ✅ Succeeds |
| 2 | Baseline same replay | BaselineGateway | ✅ Succeeds (VULNERABLE) |
| 3 | Baseline triple replay | BaselineGateway | ✅ Succeeds (VULNERABLE) |
| 4 | Baseline unbounded replay (10x) | BaselineGateway | ✅ All succeed (VULNERABLE) |
| 5 | CANB first process | CANBGateway | ✅ Succeeds |
| 6 | CANB same replay | CANBGateway | ❌ Reverts (NonceAlreadyConsumed) |
| 7 | CANB stale signature | CANBGateway | ❌ Reverts (signature mismatch) |
| 8 | CANB sequential nonces | CANBGateway | ✅ All succeed (1–5 in order) |

### 3.2 Cross-Context Replay (8 scenarios)

**Objective:** Replay a signature from Chain A on Chain B.

| # | Scenario | Target | Expected Result |
|:-:|:--|:--|:--|
| 1 | Baseline legit on Chain A | BaselineGateway A | ✅ Succeeds |
| 2 | Baseline cross-chain replay | BaselineGateway B | ✅ Succeeds (VULNERABLE) |
| 3 | Baseline same sig both chains | Both | ✅ Both succeed (VULNERABLE) |
| 4 | Baseline modified destination | BaselineGateway | ✅ Still works (no chainId in hash) |
| 5 | CANB legit on Chain A | CANBGateway A | ✅ Succeeds |
| 6 | CANB cross-chain replay | CANBGateway B | ❌ Reverts (domain separator mismatch) |
| 7 | CANB Chain A sig on Chain B | CANBGateway B | ❌ Reverts |
| 8 | CANB domain separators differ | Both | ✅ Verified different |

### 3.3 Cross-Bridge Replay (7 scenarios)

**Objective:** Replay a signature across different bridge contract instances on the same chain.

| # | Scenario | Target | Expected Result |
|:-:|:--|:--|:--|
| 1 | Baseline cross-bridge replay | Instance 1→2 | ✅ Succeeds (VULNERABLE) |
| 2 | Baseline sig valid on any instance | Instance 2→1 | ✅ Succeeds (VULNERABLE) |
| 3 | Baseline unlimited cross-bridge | Both (5x each) | ✅ All succeed (VULNERABLE) |
| 4 | CANB cross-bridge replay | Instance 1→2 | ❌ Reverts (verifyingContract differs) |
| 5 | CANB domain separators differ | Both | ✅ Verified different |
| 6 | CANB instance 2 needs own sig | Instance 2 | ✅ Succeeds with correct sig |
| 7 | CANB wrong instance sig | Instance 2 | ❌ Reverts |

### 3.4 Message Type Confusion (6 scenarios)

**Objective:** Submit incorrect message types (e.g., LOCK as inbound).

| # | Scenario | Target | Expected Result |
|:-:|:--|:--|:--|
| 1 | Baseline accepts LOCK inbound | BaselineGateway | ✅ Accepts (VULNERABLE) |
| 2 | Baseline accepts BURN inbound | BaselineGateway | ✅ Accepts (VULNERABLE) |
| 3 | CANB rejects LOCK inbound | CANBGateway | ❌ Reverts (InvalidMessageType) |
| 4 | CANB rejects BURN inbound | CANBGateway | ❌ Reverts (InvalidMessageType) |
| 5 | CANB accepts MINT (valid) | CANBGateway | ✅ Succeeds |
| 6 | Baseline type confusion | BaselineGateway | ✅ Both types processed (VULNERABLE) |

### 3.5 Nonce Ordering Attack (7 scenarios)

**Objective:** Manipulate nonce order — skip, reverse, use zero.

| # | Scenario | Target | Expected Result |
|:-:|:--|:--|:--|
| 1 | Nonce skip (2 before 1) | CANBGateway | ❌ Reverts (NonceOutOfOrder) |
| 2 | Nonce zero | CANBGateway | ❌ Reverts (expected 1, got 0) |
| 3 | Out-of-order (1→5) | CANBGateway | ❌ Reverts |
| 4 | Backward nonce (3→1) | CANBGateway | ❌ Reverts (NonceAlreadyConsumed) |
| 5 | Sequential nonces (1→10) | CANBGateway | ✅ All succeed |
| 6 | Large nonce gap (1→1000) | CANBGateway | ❌ Reverts |
| 7 | Nonce persists after revert | CANBGateway | ✅ Nonce 2 works after failed 5 |

---

## 4. Running Attack Tests

```bash
# Run all attack tests
forge test --match-path "test/attack/*" -v

# Run specific attack type
forge test --match-contract SameContextReplayTest -vvv

# Run with gas report for thesis data
forge test --match-path "test/attack/*" --gas-report
```

---

## 5. Results Matrix

| Attack Type | Baseline Result | CANB Result | Security Delta |
|:--|:--|:--|:--|
| Same-Context Replay | 🔴 VULNERABLE (unlimited replays) | 🟢 BLOCKED (nonce consumed) | +100% |
| Cross-Context Replay | 🔴 VULNERABLE (cross-chain replay) | 🟢 BLOCKED (domain separator) | +100% |
| Cross-Bridge Replay | 🔴 VULNERABLE (cross-instance) | 🟢 BLOCKED (verifyingContract) | +100% |
| Message Type Confusion | 🔴 VULNERABLE (no type check) | 🟢 BLOCKED (type validation) | +100% |
| Nonce Ordering | N/A (no nonce system) | 🟢 ENFORCED (sequential order) | +100% |

> **Conclusion:** CANB successfully mitigates all 5 attack vectors with a 100% security improvement across 36 test scenarios.
