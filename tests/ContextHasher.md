# ContextHasher.t.sol

## Deskripsi dan Kegunaan
Pengujian unit untuk logika hashing konteks, memastikan digest EIP-712 dihitung secara presisi.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {DomainSeparator} from "../../src/canb/DomainSeparator.sol";
import {ContextHasher} from "../../src/canb/ContextHasher.sol";
import {IBridgeBenchmark} from "../../src/interfaces/IBridgeBenchmark.sol";

contract ContextHasherTest is Test {
    DomainSeparator ds;
    ContextHasher hasher;

    function setUp() public {
        ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
    }

    function _makeMessage() internal pure returns (IBridgeBenchmark.BridgeMessage memory) {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA),
            recipient: address(0xB),
            token: address(0xC),
            amount: 100 ether,
            nonce: 1,
            sourceChainId: 84532,
            destinationChainId: 4202,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function test_StructHash_Deterministic() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage();
        bytes32 h1 = hasher.computeStructHash(msg1);
        bytes32 h2 = hasher.computeStructHash(msg1);
        assertEq(h1, h2);
    }

    function test_StructHash_DiffersOnFieldChange() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage();
        IBridgeBenchmark.BridgeMessage memory msg2 = _makeMessage();
        msg2.amount = 200 ether;
        assertTrue(hasher.computeStructHash(msg1) != hasher.computeStructHash(msg2));
    }

    function test_StructHash_DiffersOnNonceChange() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage();
        IBridgeBenchmark.BridgeMessage memory msg2 = _makeMessage();
        msg2.nonce = 2;
        assertTrue(hasher.computeStructHash(msg1) != hasher.computeStructHash(msg2));
    }

    function test_ContextHash_IncludesDomainSeparator() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage();
        bytes32 structHash = hasher.computeStructHash(msg1);
        bytes32 expected = keccak256(
            abi.encodePacked("\x19\x01", ds.getDomainSeparator(), structHash)
        );
        assertEq(hasher.computeContextHash(msg1), expected);
    }

    function test_ContextHash_DiffersAcrossChains() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage();
        bytes32 hash1 = hasher.computeContextHash(msg1);

        // Deploy new hasher on different chain
        vm.chainId(999);
        DomainSeparator ds2 = new DomainSeparator();
        ContextHasher hasher2 = new ContextHasher(address(ds2));
        bytes32 hash2 = hasher2.computeContextHash(msg1);

        assertTrue(hash1 != hash2, "Context hash should differ across chains");
    }

    function test_ContextHash_DiffersOnMessageType() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage();
        IBridgeBenchmark.BridgeMessage memory msg2 = _makeMessage();
        msg2.messageType = IBridgeBenchmark.MessageType.RELEASE;
        assertTrue(hasher.computeContextHash(msg1) != hasher.computeContextHash(msg2));
    }
}

```
