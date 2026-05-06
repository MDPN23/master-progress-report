# CrossContextReplay.t.sol

## Deskripsi dan Kegunaan
Pengujian skenario serangan replay lintas-konteks (AV-2). Menguji ketahanan Domain Separator.

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

/// @title Cross-Context Replay Attack Tests
/// @notice Tests replaying signatures across different chains.
/// @dev 8 scenarios — Baseline V2 (no chainId binding) is exploitable.
contract CrossContextReplayTest is Test {
    // Chain A (Base Sepolia)
    BaselineGateway baselineA;
    CANBGateway canbA;
    ContextHasher hasherA;
    ValidationModule valModA;

    // Chain B (Lisk Sepolia)
    BaselineGateway baselineB;
    CANBGateway canbB;
    ContextHasher hasherB;
    ValidationModule valModB;

    MockERC20 tokenA;
    MockERC20 tokenB;

    uint256 relayerPk = 0xA11CE;
    address relayer;

    function setUp() public {
        relayer = vm.addr(relayerPk);

        // ── Chain A: Base Sepolia (84532) ──
        vm.chainId(84532);
        tokenA = new MockERC20("TestA", "TSTA", 18, 1_000_000);
        baselineA = new BaselineGateway();
        baselineA.setRelayer(relayer, true);

        (canbA, valModA, hasherA) = _deployCANB(tokenA);

        // ── Chain B: Lisk Sepolia (4202) ──
        vm.chainId(4202);
        tokenB = new MockERC20("TestB", "TSTB", 18, 1_000_000);
        baselineB = new BaselineGateway();
        baselineB.setRelayer(relayer, true);

        (canbB, valModB, hasherB) = _deployCANB(tokenB);
    }

    function _deployCANB(MockERC20 tk)
        internal returns (CANBGateway gw, ValidationModule vm_, ContextHasher h)
    {
        DomainSeparator ds = new DomainSeparator();
        h = new ContextHasher(address(ds));
        NonceStore ns = new NonceStore();
        ContextValidator cv = new ContextValidator();
        vm_ = new ValidationModule(address(ds), address(h), address(ns), address(cv));
        ns.setAuthorizedCaller(address(vm_), true);
        vm_.setRelayer(relayer, true);

        BridgeExecutor exec = new BridgeExecutor(address(tk));
        gw = new CANBGateway(address(vm_), address(exec));
        exec.setGateway(address(gw));
        gw.setRelayer(relayer, true);
        tk.transferOwnership(address(exec));
    }

    function _makeMsg(uint256 srcChain, uint256 dstChain)
        internal pure returns (IBridgeBenchmark.BridgeMessage memory)
    {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA), recipient: address(0xB), token: address(0xC),
            amount: 100 ether, nonce: 1,
            sourceChainId: srcChain, destinationChainId: dstChain,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function _signBaselineMsg(IBridgeBenchmark.BridgeMessage memory m)
        internal view returns (bytes memory)
    {
        bytes32 rawHash = keccak256(abi.encodePacked(
            m.sender, m.recipient, m.token, m.amount, m.nonce, uint8(m.messageType)
        ));
        bytes32 ethHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ethHash);
        return abi.encodePacked(r, s, v);
    }

    // ─── Scenario 1: Baseline legit on Chain A ──────────────────────────
    function test_Baseline_LegitOnChainA() public {
        vm.chainId(84532);
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(4202, 84532);
        baselineA.processMessage(m, _signBaselineMsg(m));
    }

    // ─── Scenario 2: Baseline cross-chain replay SUCCEEDS (V2) ──────────
    function test_Baseline_CrossChainReplay_Succeeds() public {
        vm.chainId(84532);
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(4202, 84532);
        bytes memory sig = _signBaselineMsg(m);
        baselineA.processMessage(m, sig);

        // Replay on Chain B — should succeed because no chainId in hash
        vm.chainId(4202);
        baselineB.processMessage(m, sig); // VULNERABLE ✓
    }

    // ─── Scenario 3: Baseline same sig works on BOTH chains ─────────────
    function test_Baseline_SameSignatureBothChains() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(4202, 84532);
        bytes memory sig = _signBaselineMsg(m);

        vm.chainId(84532);
        baselineA.processMessage(m, sig);
        vm.chainId(4202);
        baselineB.processMessage(m, sig);
        // Both succeed = cross-chain replay vulnerability confirmed
    }

    // ─── Scenario 4: Baseline replay with modified destination ──────────
    function test_Baseline_ModifiedDestination_StillWorks() public {
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(4202, 84532);
        bytes memory sig = _signBaselineMsg(m);
        vm.chainId(84532);
        baselineA.processMessage(m, sig);

        // Change destination — sig still valid because chainId not in hash
        m.destinationChainId = 1001;
        vm.chainId(1001);
        // Sig is valid because baseline hash doesn't include chain IDs
        baselineA.processMessage(m, sig);
    }

    // ─── Scenario 5: CANB legit on Chain A ──────────────────────────────
    function test_CANB_LegitOnChainA() public {
        vm.chainId(84532);
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(4202, 84532);
        bytes32 ctxHash = hasherA.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        vm.prank(relayer);
        canbA.processMessage(m, sig);
    }

    // ─── Scenario 6: CANB cross-chain replay REVERTS ────────────────────
    function test_CANB_CrossChainReplay_Reverts() public {
        vm.chainId(84532);
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(4202, 84532);
        bytes32 ctxHash = hasherA.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        vm.prank(relayer);
        canbA.processMessage(m, sig);

        // Replay on Chain B — domain separator mismatch → revert
        vm.chainId(4202);
        m.destinationChainId = 4202;
        m.sourceChainId = 84532;
        vm.prank(relayer);
        vm.expectRevert(); // Context hash mismatch due to domain separator
        canbB.processMessage(m, sig);
    }

    // ─── Scenario 7: CANB sig from Chain A invalid on Chain B ───────────
    function test_CANB_ChainASigInvalidOnChainB() public {
        vm.chainId(84532);
        IBridgeBenchmark.BridgeMessage memory m = _makeMsg(84532, 4202);
        bytes32 ctxHash = hasherA.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        // Sig was created with Chain A's domain separator
        // Trying on Chain B's gateway should fail
        vm.chainId(4202);
        vm.prank(relayer);
        vm.expectRevert();
        canbB.processMessage(m, sig);
    }

    // ─── Scenario 8: CANB domain separators differ per chain ────────────
    function test_CANB_DomainSeparatorsDiffer() public view {
        bytes32 dsA = DomainSeparator(address(hasherA.domainSeparator())).getDomainSeparator();
        bytes32 dsB = DomainSeparator(address(hasherB.domainSeparator())).getDomainSeparator();
        assertTrue(dsA != dsB, "Domain separators must differ across chains");
    }
}

```
