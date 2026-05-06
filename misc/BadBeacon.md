# BadBeacon.sol

## Deskripsi dan Kegunaan
Penjelasan kode dan implementasi untuk BadBeacon.sol.

## Source Code

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

contract BadBeaconNoImpl {}

contract BadBeaconNotContract {
    function implementation() external pure returns (address) {
        return address(0x1);
    }
}

```
