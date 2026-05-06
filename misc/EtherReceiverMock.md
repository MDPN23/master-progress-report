# EtherReceiverMock.sol

## Deskripsi dan Kegunaan
Penjelasan kode dan implementasi untuk EtherReceiverMock.sol.

## Source Code

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

contract EtherReceiverMock {
    bool private _acceptEther;

    function setAcceptEther(bool acceptEther) public {
        _acceptEther = acceptEther;
    }

    receive() external payable {
        if (!_acceptEther) {
            revert();
        }
    }
}

```
