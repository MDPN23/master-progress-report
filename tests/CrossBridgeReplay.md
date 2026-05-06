# CrossBridgeReplay.t.sol

## Deskripsi dan Kegunaan
Pengujian skenario serangan replay lintas-jembatan (AV-3).

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

/// @title Cross-Bridge Replay Attack Tests
/// @notice Tests replaying signatures across different bridge instances on the same chain.
/// @dev 7 scenarios
contract CrossBridgeReplayTest is Test {
    BaselineGateway baselineInstance1;
    BaselineGateway baselineInstance2;
    CANBGateway canbInstance1;
    CANBGateway canbInstance2;
    ContextHasher hasher1;
    ContextHasher hasher2;

    MockERC20 token;
    uint256 relayerPk = 0xA11CE;
    address relayer;

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);
        token = new MockERC20("Test", "TST", 18, 1_000_000);

        // ── Two baseline instances on same chain ──
        baselineInstance1 = new BaselineGateway();
        baselineInstance1.setRelayer(relayer, true);
        baselineInstance2 = new BaselineGateway();
        baselineInstance2.setRelayer(relayer, true);

        // ── Two CANB instances on same chain ──
        (canbInstance1, hasher1) = _deployCANB();
        MockERC20 token2 = new MockERC20("Test2", "TST2", 18, 1_000_000);
        (canbInstance2, hasher2) = _deployCANB2(token2);
    }

    function _deployCANB() internal returns (CANBGateway gw, ContextHasher h) {
        DomainSeparator ds = new DomainSeparator();
        h = new ContextHasher(address(ds));
        NonceStore ns = new NonceStore();
        ContextValidator cv = new ContextValidator();
        ValidationModule vm_ = new ValidationModule(address(ds), address(h), address(ns), address(cv));
        ns.setAuthorizedCaller(address(vm_), true);
        vm_.setRelayer(relayer, true);

        BridgeExecutor exec = new BridgeExecutor(address(token));
        gw = new CANBGateway(address(vm_), address(exec));
        exec.setGateway(address(gw));
        gw.setRelayer(relayer, true);
        token.transferOwnership(address(exec));
    }

    function _deployCANB2(MockERC20 tk) internal returns (CANBGateway gw, ContextHasher h) {
        DomainSeparator ds = new DomainSeparator();
        h = new ContextHasher(address(ds));
        NonceStore ns = new NonceStore();
        ContextValidator cv = new ContextValidator();
        ValidationModule vm_ = new ValidationModule(address(ds), address(h), address(ns), address(cv));
        ns.setAuthorizedCaller(address(vm_), true);
        vm_.setRelayer(relayer, true);

        BridgeExecutor exec = new BridgeExecutor(address(tk));
        gw = new CANBGateway(address(vm_), address(exec));
        exec.setGateway(address(gw));
        gw.setRelayer(relayer, true);
        tk.transferOwnership(address(exec));
    }

    function _makeMsg() internal pure returns (IBridgeBenchmark.BridgeMessage memory) {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function _signBaselineMsg(IBridgeBenchmark.BridgeMessage memory m) internal view returns (bytes memory) {
        bytes32 rawHash = keccak256(abi.encodePacked(
            m.sender, m.recipient, m.token, m.amount, m.nonce, uint8(m.messageType)
        ));
        bytes32 ethHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ethHash);
        return abi.encodePacked(r, s, v);
    }

    // ─── Scenario 1: Baseline allows cross-bridge replay ────────────────
    function test_Baseline_CrossBridgeReplay_Succeeds() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg();
        bytes memory sig = _signBaselineMsg(m);
        baselineInstance1.processMessage(m, sig);
        baselineInstance2.processMessage(m, sig); // VULNERABLE
    }

    // ─── Scenario 2: Baseline sig valid on any instance ─────────────────
    function test_Baseline_SigValidOnAnyInstance() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg();
        bytes memory sig = _signBaselineMsg(m);
        // Process on instance 2 first, then instance 1
        baselineInstance2.processMessage(m, sig);
        baselineInstance1.processMessage(m, sig);
    }

    // ─── Scenario 3: Baseline unlimited cross-bridge ────────────────────
    function test_Baseline_UnlimitedCrossBridge() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg();
        bytes memory sig = _signBaselineMsg(m);
        for (uint256 i = 0; i < 5; i++) {
            baselineInstance1.processMessage(m, sig);
            baselineInstance2.processMessage(m, sig);
        }
    }

    // ─── Scenario 4: CANB blocks cross-bridge replay ────────────────────
    function test_CANB_CrossBridgeReplay_Reverts() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg();
        bytes32 ctxHash = hasher1.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        vm.prank(relayer);
        canbInstance1.processMessage(m, sig);

        // Same sig on instance 2 fails — different domain separator (verifyingContract)
        vm.prank(relayer);
        vm.expectRevert();
        canbInstance2.processMessage(m, sig);
    }

    // ─── Scenario 5: CANB domain separators differ per instance ─────────
    function test_CANB_DomainSeparatorsDifferPerInstance() public view {
        bytes32 ds1 = DomainSeparator(address(hasher1.domainSeparator())).getDomainSeparator();
        bytes32 ds2 = DomainSeparator(address(hasher2.domainSeparator())).getDomainSeparator();
        assertTrue(ds1 != ds2, "Different contract addresses = different separators");
    }

    // ─── Scenario 6: CANB instance2 needs its own sig ───────────────────
    function test_CANB_Instance2_NeedsOwnSig() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg();
        bytes32 ctxHash2 = hasher2.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash2);
        bytes memory sig2 = abi.encodePacked(r, s, v);

        vm.prank(relayer);
        canbInstance2.processMessage(m, sig2);
    }

    // ─── Scenario 7: CANB sig crafted for instance1 fails on instance2 ──
    function test_CANB_WrongInstanceSig_Reverts() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg();
        // Sign for instance 1
        bytes32 ctxHash1 = hasher1.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash1);
        bytes memory sig1 = abi.encodePacked(r, s, v);

        // Try on instance 2
        vm.prank(relayer);
        vm.expectRevert();
        canbInstance2.processMessage(m, sig1);
    }
}

```
