# CANBBridge.t.sol

## Deskripsi dan Kegunaan
Pengujian integrasi alur end-to-end pada jembatan CANB yang aman.

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

/// @title CANB Bridge Integration Test
/// @notice Full-flow tests for the CANB (secure) bridge.
contract CANBBridgeTest is Test {
    CANBGateway gateway;
    ValidationModule valMod;
    BridgeExecutor executor;
    ContextHasher hasher;
    MockERC20 token;

    uint256 relayerPk = 0xA11CE;
    address relayer;
    address user = address(0xBEEF);

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);
        token = new MockERC20("Test", "TST", 18, 1_000_000);

        DomainSeparator ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
        NonceStore ns = new NonceStore();
        ContextValidator cv = new ContextValidator();
        valMod = new ValidationModule(address(ds), address(hasher), address(ns), address(cv));
        ns.setAuthorizedCaller(address(valMod), true);
        valMod.setRelayer(relayer, true);

        executor = new BridgeExecutor(address(token));
        gateway = new CANBGateway(address(valMod), address(executor));
        executor.setGateway(address(gateway));
        gateway.setRelayer(relayer, true);
        token.transferOwnership(address(executor));
    }

    function test_FullFlow_MintViaProcessMessage() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: user, recipient: user, token: address(token),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
        bytes32 ctxHash = hasher.computeContextHash(m);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        vm.prank(relayer);
        gateway.processMessage(m, sig);

        assertEq(token.balanceOf(user), 100 ether);
    }

    function test_FullFlow_SequentialMessages() public {
        for (uint256 i = 1; i <= 3; i++) {
            IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
                sender: user, recipient: user, token: address(token),
                amount: 50 ether, nonce: i,
                sourceChainId: 4202, destinationChainId: 84532,
                messageType: IBridgeBenchmark.MessageType.MINT
            });
            bytes32 ctxHash = hasher.computeContextHash(m);
            (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ctxHash);
            vm.prank(relayer);
            gateway.processMessage(m, abi.encodePacked(r, s, v));
        }
        assertEq(token.balanceOf(user), 150 ether);
    }
}

```
