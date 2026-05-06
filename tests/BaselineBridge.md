# BaselineBridge.t.sol

## Deskripsi dan Kegunaan
Pengujian integrasi alur end-to-end pada jembatan kontrol (Baseline) yang rentan.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {MessageHashUtils} from "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";
import {MockERC20} from "../../src/token/MockERC20.sol";
import {BaselineGateway} from "../../src/baseline/BaselineGateway.sol";
import {BaselineExecutor} from "../../src/baseline/BaselineExecutor.sol";
import {IBridgeBenchmark} from "../../src/interfaces/IBridgeBenchmark.sol";

/// @title Baseline Bridge Integration Test
/// @notice Full-flow tests for the baseline (vulnerable) bridge.
contract BaselineBridgeTest is Test {
    BaselineGateway gateway;
    BaselineExecutor executor;
    MockERC20 token;

    uint256 relayerPk = 0xA11CE;
    address relayer;
    address user = address(0x1234);

    function setUp() public {
        relayer = vm.addr(relayerPk);
        token = new MockERC20("Test", "TST", 18, 1_000_000);
        gateway = new BaselineGateway();
        executor = new BaselineExecutor(address(token));

        gateway.setRelayer(relayer, true);
        gateway.setExecutor(address(executor));
        executor.setGateway(address(gateway));

        // Fund user
        token.mint(user, 1000 ether);
    }

    function test_FullFlow_LockAndProcess() public {
        // User locks tokens
        vm.startPrank(user);
        token.approve(address(gateway), 100 ether);
        gateway.lockAssets(address(token), 100 ether, 4202);
        vm.stopPrank();

        assertEq(token.balanceOf(address(gateway)), 100 ether);
    }

    function test_ProcessMessage_Succeeds() public {
        IBridgeBenchmark.BridgeMessage memory m = IBridgeBenchmark.BridgeMessage({
            sender: user, recipient: user, token: address(token),
            amount: 100 ether, nonce: 1,
            sourceChainId: 4202, destinationChainId: 84532,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
        bytes32 rawHash = keccak256(abi.encodePacked(
            m.sender, m.recipient, m.token, m.amount, m.nonce, uint8(m.messageType)
        ));
        bytes32 ethHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, ethHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        gateway.processMessage(m, sig);
    }
}

```
