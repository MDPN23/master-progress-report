# DomainSeparator.t.sol

## Deskripsi dan Kegunaan
Pengujian unit untuk memvalidasi pembuatan dan penggunaan Domain Separator EIP-712 yang benar.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {DomainSeparator} from "../../src/canb/DomainSeparator.sol";

contract DomainSeparatorTest is Test {
    DomainSeparator ds;

    function setUp() public {
        ds = new DomainSeparator();
    }

    function test_DomainSeparator_IsNonZero() public view {
        assertTrue(ds.DOMAIN_SEPARATOR() != bytes32(0));
    }

    function test_DomainSeparator_IsImmutable() public view {
        bytes32 sep1 = ds.getDomainSeparator();
        bytes32 sep2 = ds.getDomainSeparator();
        assertEq(sep1, sep2);
    }

    function test_DomainSeparator_CapturesChainId() public view {
        assertEq(ds.DEPLOYMENT_CHAIN_ID(), block.chainid);
    }

    function test_DomainSeparator_DiffersAcrossChains() public {
        // Deploy on "current" chain
        bytes32 sep1 = ds.getDomainSeparator();

        // Deploy on a different chain
        vm.chainId(999);
        DomainSeparator ds2 = new DomainSeparator();
        bytes32 sep2 = ds2.getDomainSeparator();

        assertTrue(sep1 != sep2, "Separators should differ across chains");
    }

    function test_DomainSeparator_DiffersAcrossContracts() public {
        DomainSeparator ds2 = new DomainSeparator();
        assertTrue(
            ds.getDomainSeparator() != ds2.getDomainSeparator(),
            "Separators should differ across contract instances"
        );
    }

    function test_DomainSeparator_MatchesManualComputation() public view {
        bytes32 expected = keccak256(
            abi.encode(
                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
                keccak256(bytes("CANB")),
                keccak256(bytes("1")),
                block.chainid,
                address(ds)
            )
        );
        assertEq(ds.getDomainSeparator(), expected);
    }
}

```
