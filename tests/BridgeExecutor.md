# BridgeExecutor.t.sol

## Deskripsi dan Kegunaan
Pengujian unit untuk fungsi eksekusi (minting/releasing) dengan pola CEI.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {MockERC20} from "../../src/token/MockERC20.sol";
import {BridgeExecutor} from "../../src/canb/BridgeExecutor.sol";
import {CANBErrors} from "../../src/libraries/CANBErrors.sol";

contract BridgeExecutorTest is Test {
    BridgeExecutor executor;
    MockERC20 token;
    address gateway = address(0x6A7E);
    address recipient = address(0xBEEF);
    bytes32 ctxHash = keccak256("test-context");

    function setUp() public {
        token = new MockERC20("Test", "TST", 18, 0);
        executor = new BridgeExecutor(address(token));
        executor.setGateway(gateway);
        // Transfer token ownership to executor so it can mint
        token.transferOwnership(address(executor));
    }

    // ─── Mint ───────────────────────────────────────────────────────────

    function test_MintTokens_Success() public {
        vm.prank(gateway);
        executor.mintTokens(recipient, 100 ether, ctxHash);
        assertEq(token.balanceOf(recipient), 100 ether);
    }

    function test_MintTokens_RevertOnZeroAmount() public {
        vm.prank(gateway);
        vm.expectRevert(CANBErrors.ZeroAmount.selector);
        executor.mintTokens(recipient, 0, ctxHash);
    }

    function test_MintTokens_RevertOnZeroAddress() public {
        vm.prank(gateway);
        vm.expectRevert(CANBErrors.ZeroAddress.selector);
        executor.mintTokens(address(0), 100 ether, ctxHash);
    }

    function test_MintTokens_RevertOnNonGateway() public {
        vm.prank(address(0xBAD));
        vm.expectRevert("BridgeExec: not gateway");
        executor.mintTokens(recipient, 100 ether, ctxHash);
    }

    // ─── Release (CEI Pattern) ──────────────────────────────────────────

    function test_ReleaseTokens_Success() public {
        // Seed executor with tokens
        vm.prank(gateway);
        executor.mintTokens(address(executor), 500 ether, ctxHash);
        // Record deposit for recipient
        vm.prank(gateway);
        executor.recordDeposit(recipient, 200 ether);

        vm.prank(gateway);
        executor.releaseTokens(recipient, 200 ether, ctxHash);
        assertEq(executor.lockedBalances(recipient), 0);
    }

    function test_ReleaseTokens_RevertOnInsufficient() public {
        vm.prank(gateway);
        executor.recordDeposit(recipient, 50 ether);

        vm.prank(gateway);
        vm.expectRevert("BridgeExec: insufficient locked");
        executor.releaseTokens(recipient, 100 ether, ctxHash);
    }
}

```
