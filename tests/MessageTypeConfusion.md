# MessageTypeConfusion.t.sol

## Deskripsi dan Kegunaan
Pengujian skenario serangan kebingungan tipe pesan (Message Type Confusion). Memastikan bahwa pesan MINT dan RELEASE tidak bisa dimanipulasi.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
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
import {CANBErrors} from "../../src/libraries/CANBErrors.sol";

/// @title Message Type Confusion Attack Tests
/// @notice Tests submitting incorrect message types.
/// @dev 6 scenarios
contract MessageTypeConfusionTest is Test {
    CANBGateway canb;
    ContextHasher hasher;
    BaselineGateway baseline;
    MockERC20 token;

    uint256 relayerPk = 0xA11CE;
    address relayer;

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);
        token = new MockERC20("Test", "TST", 18, 1_000_000);

        baseline = new BaselineGateway();
        baseline.setRelayer(relayer, true);

        DomainSeparator ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
        NonceStore ns = new NonceStore();
        ContextValidator cv = new ContextValidator();
        ValidationModule vm_ = new ValidationModule(address(ds), address(hasher), address(ns), address(cv));
        ns.setAuthorizedCaller(address(vm_), true);
        vm_.setRelayer(relayer, true);

        BridgeExecutor exec = new BridgeExecutor(address(token));
        canb = new CANBGateway(address(vm_), address(exec));
        exec.setGateway(address(canb));
        canb.setRelayer(relayer, true);
        token.transferOwnership(address(exec));
    }

    function _signBaselineMsg(IBridgeBenchmark.BridgeMessage memory m) internal view returns (bytes memory) {
        bytes32 rawHash = keccak256(abi.encodePacked(
            m.sender, m.recipient, m.token, m.amount, m.nonce, uint8(m.messageType)
        ));
        bytes32 ethHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ethHash);
        return abi.encodePacked(r, s, v);
    }

    function _signCANBMsg(IBridgeBenchmark.BridgeMessage memory m) internal view returns (bytes memory) {
        bytes32 ctxHash = hasher.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        return abi.encodePacked(r, s, v);
    }

    // ─── Scenario 1: Baseline accepts LOCK type as inbound ──────────────
    function test_Baseline_AcceptsLockTypeInbound() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.LOCK // Wrong for inbound!
        });
        baseline.processMessage(m, _signBaselineMsg(m)); // No type validation = VULNERABLE
    }

    // ─── Scenario 2: Baseline accepts BURN type as inbound ──────────────
    function test_Baseline_AcceptsBurnTypeInbound() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.BURN
        });
        baseline.processMessage(m, _signBaselineMsg(m)); // VULNERABLE
    }

    // ─── Scenario 3: CANB rejects LOCK type as inbound ──────────────────
    function test_CANB_RejectsLockTypeInbound() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.LOCK
        });
        bytes memory sig = _signCANBMsg(m);
        vm.prank(relayer);
        vm.expectRevert(abi.encodeWithSelector(CANBErrors.InvalidMessageType.selector, uint8(0)));
        canb.processMessage(m, sig);
    }

    // ─── Scenario 4: CANB rejects BURN type as inbound ──────────────────
    function test_CANB_RejectsBurnTypeInbound() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.BURN
        });
        bytes memory sig = _signCANBMsg(m);
        vm.prank(relayer);
        vm.expectRevert(abi.encodeWithSelector(CANBErrors.InvalidMessageType.selector, uint8(1)));
        canb.processMessage(m, sig);
    }

    // ─── Scenario 5: CANB accepts MINT type (valid) ─────────────────────
    function test_CANB_AcceptsMintType() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
        bytes memory sig = _signCANBMsg(m);
        vm.prank(relayer);
        canb.processMessage(m, sig); // OK
    }

    // ─── Scenario 6: Baseline type confusion with modified sig ──────────
    function test_Baseline_TypeConfusion_DifferentTypesSameMsg() public {
        IBridgeBenchmark.BridgeMessage memory m1 = IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
        IBridgeBenchmark.BridgeMessage memory m2 = m1;
        m2.messageType = IBridgeBenchmark.MessageType.LOCK;

        baseline.processMessage(m1, _signBaselineMsg(m1));
        baseline.processMessage(m2, _signBaselineMsg(m2));
        // Both processed — baseline doesn't distinguish inbound/outbound types
    }
}

```
