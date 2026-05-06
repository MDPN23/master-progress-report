# NonceOrderingAttack.t.sol

## Deskripsi dan Kegunaan
Pengujian serangan manipulasi urutan nonce (Nonce Ordering).

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {MockERC20} from "../../src/token/MockERC20.sol";
import {IBridgeBenchmark} from "../../src/interfaces/IBridgeBenchmark.sol";
import {DomainSeparator} from "../../src/canb/DomainSeparator.sol";
import {ContextHasher} from "../../src/canb/ContextHasher.sol";
import {NonceStore} from "../../src/canb/NonceStore.sol";
import {ContextValidator} from "../../src/canb/ContextValidator.sol";
import {ValidationModule} from "../../src/canb/ValidationModule.sol";
import {BridgeExecutor} from "../../src/canb/BridgeExecutor.sol";
import {CANBGateway} from "../../src/canb/CANBGateway.sol";
import {CANBErrors} from "../../src/libraries/CANBErrors.sol";

/// @title Nonce Ordering Attack Tests
/// @notice Tests nonce manipulation — skipping, reordering, zero nonce.
/// @dev 7 scenarios
contract NonceOrderingAttackTest is Test {
    CANBGateway canb;
    ContextHasher hasher;
    ValidationModule valMod;
    NonceStore nonceStore;
    MockERC20 token;

    uint256 relayerPk = 0xA11CE;
    address relayer;

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);
        token = new MockERC20("Test", "TST", 18, 1_000_000);

        DomainSeparator ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
        nonceStore = new NonceStore();
        ContextValidator cv = new ContextValidator();
        valMod = new ValidationModule(address(ds), address(hasher), address(nonceStore), address(cv));
        nonceStore.setAuthorizedCaller(address(valMod), true);
        valMod.setRelayer(relayer, true);

        BridgeExecutor exec = new BridgeExecutor(address(token));
        canb = new CANBGateway(address(valMod), address(exec));
        exec.setGateway(address(canb));
        canb.setRelayer(relayer, true);
        token.transferOwnership(address(exec));
    }

    function _makeMsg(uint256 nonce) internal pure returns (IBridgeBenchmark.BridgeMessage memory) {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: nonce,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function _signMsg(IBridgeBenchmark.BridgeMessage memory m) internal view returns (bytes memory) {
        bytes32 ctxHash = hasher.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        return abi.encodePacked(r, s, v);
    }

    /// @dev Helper: sign then process as relayer
    function _processAsRelayer(IBridgeBenchmark.BridgeMessage memory m) internal {
        bytes memory sig = _signMsg(m);
        vm.prank(relayer);
        canb.processMessage(m, sig);
    }

    // ─── Scenario 1: Nonce skip reverts ─────────────────────────────────
    function test_NonceSkip_Reverts() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(2);
        bytes memory sig = _signMsg(m);
        vm.prank(relayer);
        vm.expectRevert();
        canb.processMessage(m, sig);
    }

    // ─── Scenario 2: Nonce zero reverts ─────────────────────────────────
    function test_NonceZero_Reverts() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(0);
        bytes memory sig = _signMsg(m);
        vm.prank(relayer);
        vm.expectRevert();
        canb.processMessage(m, sig);
    }

    // ─── Scenario 3: Out-of-order nonce reverts ─────────────────────────
    function test_OutOfOrder_Reverts() public {
        _processAsRelayer(_makeMsg(1));

        IBridgeBenchmark.BridgeMessage memory m5 = _makeMsg(5);
        bytes memory sig = _signMsg(m5);
        vm.prank(relayer);
        vm.expectRevert();
        canb.processMessage(m5, sig);
    }

    // ─── Scenario 4: Backward nonce reverts ─────────────────────────────
    function test_BackwardNonce_Reverts() public {
        for (uint256 i = 1; i <= 3; i++) {
            _processAsRelayer(_makeMsg(i));
        }

        IBridgeBenchmark.BridgeMessage memory mOld = _makeMsg(1);
        bytes memory sig = _signMsg(mOld);
        vm.prank(relayer);
        vm.expectRevert();
        canb.processMessage(mOld, sig);
    }

    // ─── Scenario 5: Sequential nonces succeed ──────────────────────────
    function test_SequentialNonces_Succeed() public {
        for (uint256 i = 1; i <= 10; i++) {
            _processAsRelayer(_makeMsg(i));
        }
    }

    // ─── Scenario 6: Large nonce gap reverts ────────────────────────────
    function test_LargeNonceGap_Reverts() public {
        _processAsRelayer(_makeMsg(1));

        IBridgeBenchmark.BridgeMessage memory mLarge = _makeMsg(1000);
        bytes memory sig = _signMsg(mLarge);
        vm.prank(relayer);
        vm.expectRevert();
        canb.processMessage(mLarge, sig);
    }

    // ─── Scenario 7: Nonce state persists after revert ──────────────────
    function test_NoncePersistsAfterRevert() public {
        _processAsRelayer(_makeMsg(1));

        // Try invalid nonce 5 (reverts)
        IBridgeBenchmark.BridgeMessage memory m5 = _makeMsg(5);
        bytes memory sig5 = _signMsg(m5);
        vm.prank(relayer);
        vm.expectRevert();
        canb.processMessage(m5, sig5);

        // Nonce 2 should still work
        _processAsRelayer(_makeMsg(2));
    }
}

```
