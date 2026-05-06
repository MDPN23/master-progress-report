# ComparisonBenchmark.t.sol

## Deskripsi dan Kegunaan
Skrip pengujian benchmark yang membandingkan konsumsi gas antara Baseline dan CANB.

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

/// @title Comparison Benchmark — Side-by-side gas comparison
/// @notice Measures gas for identical operations on Baseline vs CANB.
/// @dev Run with: forge snapshot --match-contract ComparisonBenchmark
contract ComparisonBenchmarkTest is Test {
    BaselineGateway baseline;
    CANBGateway canb;
    ContextHasher hasher;
    MockERC20 tokenBaseline;
    MockERC20 tokenCANB;

    uint256 relayerPk = 0xA11CE;
    address relayer;
    address user = address(0xBEEF);

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);

        // ── Baseline ──
        tokenBaseline = new MockERC20("Base", "BASE", 18, 1_000_000);
        baseline = new BaselineGateway();
        baseline.setRelayer(relayer, true);

        // ── CANB ──
        tokenCANB = new MockERC20("CANB", "CANB", 18, 1_000_000);
        DomainSeparator ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
        NonceStore ns = new NonceStore();
        ContextValidator cv = new ContextValidator();
        ValidationModule vm_ = new ValidationModule(address(ds), address(hasher), address(ns), address(cv));
        ns.setAuthorizedCaller(address(vm_), true);
        vm_.setRelayer(relayer, true);

        // Fund user for lock tests (mint BEFORE transferring ownership)
        tokenBaseline.mint(user, 10_000 ether);
        tokenCANB.mint(user, 10_000 ether);

        BridgeExecutor exec = new BridgeExecutor(address(tokenCANB));
        canb = new CANBGateway(address(vm_), address(exec));
        exec.setGateway(address(canb));
        canb.setRelayer(relayer, true);
        tokenCANB.transferOwnership(address(exec));
    }

    // ─── processMessage Gas Comparison ──────────────────────────────────

    function test_Baseline_ProcessMessage() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: user, recipient: user, token: address(tokenBaseline),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
        bytes32 rawHash = keccak256(abi.encodePacked(
            m.sender, m.recipient, m.token, m.amount, m.nonce, uint8(m.messageType)
        ));
        bytes32 ethHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ethHash);
        baseline.processMessage(m, abi.encodePacked(r, s, v));
    }

    function test_CANB_ProcessMessage() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: user, recipient: user, token: address(tokenCANB),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
        bytes32 ctxHash = hasher.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        vm.prank(relayer);
        canb.processMessage(m, abi.encodePacked(r, s, v));
    }

    // ─── lockAssets Gas Comparison ───────────────────────────────────────

    function test_Baseline_LockAssets() public {
        vm.startPrank(user);
        tokenBaseline.approve(address(baseline), 100 ether);
        baseline.lockAssets(address(tokenBaseline), 100 ether, 4202);
        vm.stopPrank();
    }

    function test_CANB_LockAssets() public {
        vm.startPrank(user);
        tokenCANB.approve(address(canb), 100 ether);
        canb.lockAssets(address(tokenCANB), 100 ether, 4202);
        vm.stopPrank();
    }
}

```
