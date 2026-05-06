# NonceStore.t.sol

## Deskripsi dan Kegunaan
Pengujian unit (Unit Test) untuk memastikan fungsi perlindungan replay pada NonceStore bekerja dengan baik.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {NonceStore} from "../../src/canb/NonceStore.sol";
import {CANBErrors} from "../../src/libraries/CANBErrors.sol";
import {CANBEvents} from "../../src/libraries/CANBEvents.sol";

contract NonceStoreTest is Test {
    NonceStore store;
    uint256 chainId = 84532; // Base Sepolia
    bytes32 bridgeId = keccak256("TEST-BRIDGE");

    function setUp() public {
        store = new NonceStore();
    }

    // ─── Success Cases ──────────────────────────────────────────────────

    function test_ConsumeNonce_Success() public {
        bool result = store.consumeNonce(chainId, bridgeId, 1);
        assertTrue(result);
        assertTrue(store.isNonceUsed(chainId, bridgeId, 1));
    }

    function test_ConsumeNonce_SequentialMultiple() public {
        for (uint256 i = 1; i <= 5; i++) {
            store.consumeNonce(chainId, bridgeId, i);
            assertTrue(store.isNonceUsed(chainId, bridgeId, i));
        }
    }

    function test_ConsumeNonce_EmitsEvent() public {
        vm.expectEmit(true, true, false, true);
        emit CANBEvents.NonceConsumed(chainId, bridgeId, 1);
        store.consumeNonce(chainId, bridgeId, 1);
    }

    function test_NextExpectedNonce_Advances() public {
        assertEq(store.nextExpectedNonce(chainId, bridgeId), 0);
        store.consumeNonce(chainId, bridgeId, 1);
        assertEq(store.nextExpectedNonce(chainId, bridgeId), 1);
        store.consumeNonce(chainId, bridgeId, 2);
        assertEq(store.nextExpectedNonce(chainId, bridgeId), 2);
    }

    // ─── Revert Cases ───────────────────────────────────────────────────

    function test_ConsumeNonce_RevertOnDuplicate() public {
        store.consumeNonce(chainId, bridgeId, 1);
        vm.expectRevert(
            abi.encodeWithSelector(CANBErrors.NonceAlreadyConsumed.selector, chainId, bridgeId, 1)
        );
        store.consumeNonce(chainId, bridgeId, 1);
    }

    function test_ConsumeNonce_RevertOnSkip() public {
        store.consumeNonce(chainId, bridgeId, 1);
        vm.expectRevert(
            abi.encodeWithSelector(CANBErrors.NonceOutOfOrder.selector, 2, 3)
        );
        store.consumeNonce(chainId, bridgeId, 3); // Skip nonce 2
    }

    function test_ConsumeNonce_RevertOnZero() public {
        vm.expectRevert(
            abi.encodeWithSelector(CANBErrors.NonceOutOfOrder.selector, 1, 0)
        );
        store.consumeNonce(chainId, bridgeId, 0);
    }

    // ─── Bitmap Boundary ────────────────────────────────────────────────

    function test_BitmapBoundary_Nonce255to256() public {
        for (uint256 i = 1; i <= 256; i++) {
            store.consumeNonce(chainId, bridgeId, i);
        }
        assertTrue(store.isNonceUsed(chainId, bridgeId, 255));
        assertTrue(store.isNonceUsed(chainId, bridgeId, 256));
        assertFalse(store.isNonceUsed(chainId, bridgeId, 257));
    }

    // ─── Cross-Chain Isolation ──────────────────────────────────────────

    function test_CrossChainIsolation() public {
        uint256 liskChainId = 4202;
        store.consumeNonce(chainId, bridgeId, 1);
        store.consumeNonce(liskChainId, bridgeId, 1);
        assertTrue(store.isNonceUsed(chainId, bridgeId, 1));
        assertTrue(store.isNonceUsed(liskChainId, bridgeId, 1));
    }

    function test_CrossBridgeIsolation() public {
        bytes32 bridgeId2 = keccak256("OTHER-BRIDGE");
        store.consumeNonce(chainId, bridgeId, 1);
        store.consumeNonce(chainId, bridgeId2, 1);
        assertTrue(store.isNonceUsed(chainId, bridgeId, 1));
        assertTrue(store.isNonceUsed(chainId, bridgeId2, 1));
    }

    // ─── Access Control ─────────────────────────────────────────────────

    function test_OnlyAuthorized_CanConsume() public {
        address unauthorized = address(0xBEEF);
        vm.prank(unauthorized);
        vm.expectRevert("NonceStore: unauthorized");
        store.consumeNonce(chainId, bridgeId, 1);
    }
}

```
