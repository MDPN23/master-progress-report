# ERC20ReturnFalseMock.sol

## Deskripsi dan Kegunaan
Penjelasan kode dan implementasi untuk ERC20ReturnFalseMock.sol.

## Source Code

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import {ERC20} from "../../token/ERC20/ERC20.sol";

abstract contract ERC20ReturnFalseMock is ERC20 {
    function transfer(address, uint256) public pure override returns (bool) {
        return false;
    }

    function transferFrom(address, address, uint256) public pure override returns (bool) {
        return false;
    }

    function approve(address, uint256) public pure override returns (bool) {
        return false;
    }
}

```
