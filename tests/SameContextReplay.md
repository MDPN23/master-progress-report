# SameContextReplay.t.sol

## Deskripsi dan Kegunaan
Pengujian skenario serangan replay pada konteks dan chain yang sama (AV-1). Membuktikan bahwa CANB memitigasi eksploitasi ini.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import {MessageHashUtils} from "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";
import {MockERC20} from "../../src/token/MockERC20.sol";
import {BaselineGateway} from "../../src/baseline/BaselineGateway.sol";
import {IBridgeBenchmark} from "../../src/interfaces/IBridgeBenchmark.sol";
import {DomainSeparator} from "../../src/canb/DomainSeparator.sol";
import {ContextHasher} from "../../src/canb/ContextHasher.sol";
import {NonceStore} from "../../src/canb/NonceStore.sol";
import {ContextValidator} from "../../src/canb/ContextValidator.sol";
import {ValidationModule} from "../../src/canb/ValidationModule.sol";
import {BridgeExecutor} from "../../src/canb/BridgeExecutor.sol";
import {CANBGateway} from "../../src/canb/CANBGateway.sol";

/// @title Same-Context Replay Attack Tests
/// @notice Tests replay of the exact same message on the same chain/bridge.
/// @dev 8 scenarios — Baseline should be vulnerable, CANB should reject.
contract SameContextReplayTest is Test {
    // ─── Baseline ───────────────────────────────────────────────────────
    BaselineGateway baseline;
    // ─── CANB Stack ─────────────────────────────────────────────────────
    CANBGateway canb;
    ValidationModule valMod;
    ContextHasher hasher;
    NonceStore nonceStore;
    MockERC20 token;
    BridgeExecutor canbExec;

    uint256 relayerPk = 0xA11CE;
    address relayer;

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);
        token = new MockERC20("Test", "TST", 18, 1_000_000);

        // ── Baseline setup ──
        baseline = new BaselineGateway();
        baseline.setRelayer(relayer, true);

        // ── CANB setup ──
        DomainSeparator ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
        nonceStore = new NonceStore();
        ContextValidator cv = new ContextValidator();
        valMod = new ValidationModule(address(ds), address(hasher), address(nonceStore), address(cv));
        nonceStore.setAuthorizedCaller(address(valMod), true);
        valMod.setRelayer(relayer, true);

        canbExec = new BridgeExecutor(address(token));
        canb = new CANBGateway(address(valMod), address(canbExec));
        canbExec.setGateway(address(canb));
        canb.setRelayer(relayer, true);
        token.transferOwnership(address(canbExec));
    }

    function _baselineMsg() internal pure returns (IBridgeBenchmark.BridgeMessage memory) {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function _signBaseline(IBridgeBenchmark.BridgeMessage memory m) internal view returns (bytes memory) {
        bytes32 rawHash = keccak256(abi.encodePacked(
            m.sender, m.recipient, m.token, m.amount, m.nonce, uint8(m.messageType)
        ));
        bytes32 ethHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ethHash);
        return abi.encodePacked(r, s, v);
    }

    function _signCANB(IBridgeBenchmark.BridgeMessage memory m) internal view returns (bytes memory) {
        bytes32 ctxHash = hasher.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        return abi.encodePacked(r, s, v);
    }

    // ─── Scenario 1: Baseline allows first process ──────────────────────
    function test_Baseline_FirstProcess_Succeeds() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        baseline.processMessage(m, _signBaseline(m));
        // No revert = success ✓
    }

    // ─── Scenario 2: Baseline allows REPLAY (VULNERABLE) ───────────────
    function test_Baseline_SameReplay_Succeeds() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        bytes memory sig = _signBaseline(m);
        baseline.processMessage(m, sig);
        baseline.processMessage(m, sig); // REPLAY succeeds!
    }

    // ─── Scenario 3: Baseline allows TRIPLE replay ─────────────────────
    function test_Baseline_TripleReplay_Succeeds() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        bytes memory sig = _signBaseline(m);
        baseline.processMessage(m, sig);
        baseline.processMessage(m, sig);
        baseline.processMessage(m, sig); // 3x replay all succeed
    }

    // ─── Scenario 4: Baseline unbounded replay ─────────────────────────
    function test_Baseline_UnboundedReplay_10Times() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        bytes memory sig = _signBaseline(m);
        for (uint256 i = 0; i < 10; i++) {
            baseline.processMessage(m, sig);
        }
    }

    // ─── Scenario 5: CANB first process succeeds ────────────────────────
    function test_CANB_FirstProcess_Succeeds() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        bytes memory sig = _signCANB(m);
        vm.prank(relayer);
        canb.processMessage(m, sig);
    }

    // ─── Scenario 6: CANB BLOCKS replay ─────────────────────────────────
    function test_CANB_SameReplay_Reverts() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        bytes memory sig = _signCANB(m);
        vm.prank(relayer);
        canb.processMessage(m, sig);

        vm.prank(relayer);
        vm.expectRevert(); // NonceAlreadyConsumed
        canb.processMessage(m, sig);
    }

    // ─── Scenario 7: CANB blocks incremented-nonce with old sig ─────────
    function test_CANB_StaleSignature_Reverts() public {
        IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
        bytes memory sig = _signCANB(m);
        vm.prank(relayer);
        canb.processMessage(m, sig);

        // Attacker tries with nonce=2 but uses old signature
        IBridgeBenchmark.BridgeMessage memory m2 = _baselineMsg();
        m2.nonce = 2;
        vm.prank(relayer);
        vm.expectRevert(); // Signature mismatch
        canb.processMessage(m2, sig);
    }

    // ─── Scenario 8: CANB sequential nonces work ────────────────────────
    function test_CANB_SequentialNonces_Success() public {
        for (uint256 i = 1; i <= 5; i++) {
            IBridgeBenchmark.BridgeMessage memory m = _baselineMsg();
            m.nonce = i;
            bytes memory sig = _signCANB(m);
            vm.prank(relayer);
            canb.processMessage(m, sig);
        }
    }
}

```
