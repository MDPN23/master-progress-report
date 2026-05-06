# ContextValidator.t.sol

## Deskripsi dan Kegunaan
Pengujian unit untuk memeriksa validasi rute dan tipe pesan.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {ContextValidator} from "../../src/canb/ContextValidator.sol";
import {IBridgeBenchmark} from "../../src/interfaces/IBridgeBenchmark.sol";
import {CANBErrors} from "../../src/libraries/CANBErrors.sol";

contract ContextValidatorTest is Test {
    ContextValidator cv;

    function setUp() public {
        vm.chainId(84532); // Base Sepolia
        cv = new ContextValidator();
    }

    function _makeValidMsg() internal pure returns (IBridgeBenchmark.BridgeMessage memory) {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA),
            recipient: address(0xB),
            token: address(0xC),
            amount: 100 ether,
            nonce: 1,
            sourceChainId: 4202,        // Lisk
            destinationChainId: 84532,  // Base (matches deployment)
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function test_ValidContext_MINT() public view {
        cv.validateContext(_makeValidMsg());
        // No revert = success
    }

    function test_ValidContext_RELEASE() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.messageType = IBridgeBenchmark.MessageType.RELEASE;
        cv.validateContext(msg1);
    }

    function test_RevertOnZeroSourceChainId() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.sourceChainId = 0;
        vm.expectRevert(CANBErrors.InvalidChainId.selector);
        cv.validateContext(msg1);
    }

    function test_RevertOnZeroDestinationChainId() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.destinationChainId = 0;
        vm.expectRevert(CANBErrors.InvalidChainId.selector);
        cv.validateContext(msg1);
    }

    function test_RevertOnSameChainRoute() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.sourceChainId = 84532;
        msg1.destinationChainId = 84532;
        vm.expectRevert(abi.encodeWithSelector(CANBErrors.SameChainRoute.selector, 84532));
        cv.validateContext(msg1);
    }

    function test_RevertOnChainIdMismatch() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.destinationChainId = 999; // Not the deployment chain
        vm.expectRevert(abi.encodeWithSelector(CANBErrors.ChainIdMismatch.selector, 84532, 999));
        cv.validateContext(msg1);
    }

    function test_RevertOnInvalidMessageType_LOCK() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.messageType = IBridgeBenchmark.MessageType.LOCK;
        vm.expectRevert(abi.encodeWithSelector(CANBErrors.InvalidMessageType.selector, uint8(0)));
        cv.validateContext(msg1);
    }

    function test_RevertOnInvalidMessageType_BURN() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeValidMsg();
        msg1.messageType = IBridgeBenchmark.MessageType.BURN;
        vm.expectRevert(abi.encodeWithSelector(CANBErrors.InvalidMessageType.selector, uint8(1)));
        cv.validateContext(msg1);
    }
}

```
