# NonceStore.sol

## Deskripsi dan Kegunaan
`NonceStore.sol` adalah kontrak pintar yang berfungsi untuk mencegah serangan pengulangan (*replay attacks*) dalam framework CANB. Kontrak ini menggunakan struktur data *bitmap* untuk menyimpan status *nonce* secara sangat efisien dari segi gas. 

Daripada menggunakan satu *storage slot* penuh untuk setiap *nonce*, `NonceStore` memampatkan 256 *nonce* ke dalam satu slot memori (`uint256`), sehingga secara signifikan mengurangi biaya operasi `SSTORE` (penyimpanan ke blockchain). Ini memastikan bahwa setiap pesan lintas rantai (*cross-chain message*) diproses secara berurutan dan setiap *nonce* hanya bisa dikonsumsi satu kali.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {INonceStore} from "../interfaces/INonceStore.sol";
import {CANBErrors} from "../libraries/CANBErrors.sol";
import {CANBEvents} from "../libraries/CANBEvents.sol";

/**
 * @title NonceStore
 * @notice Bitmap-based nonce management for gas-efficient replay protection.
 * @dev Nonces tracked per (chainId, bridgeId). Each 256-nonce range uses one uint256 bitmap slot.
 */
contract NonceStore is INonceStore {
    mapping(uint256 => mapping(bytes32 => mapping(uint256 => uint256))) private _bitmaps;
    mapping(uint256 => mapping(bytes32 => uint256)) public nextExpectedNonce;

    address public owner;
    mapping(address => bool) public authorizedCallers;

    modifier onlyOwner() {
        if (msg.sender != owner) revert CANBErrors.NotOwner();
        _;
    }

    modifier onlyAuthorized() {
        require(msg.sender == owner || authorizedCallers[msg.sender], "NonceStore: unauthorized");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function setAuthorizedCaller(address caller, bool status) external onlyOwner {
        authorizedCallers[caller] = status;
    }

    /// @inheritdoc INonceStore
    function consumeNonce(uint256 chainId, bytes32 bridgeId, uint256 nonce)
        external override onlyAuthorized returns (bool)
    {
        uint256 expected = nextExpectedNonce[chainId][bridgeId] + 1;
        if (nonce != expected) {
            if (_isNonceSet(chainId, bridgeId, nonce)) {
                revert CANBErrors.NonceAlreadyConsumed(chainId, bridgeId, nonce);
            }
            revert CANBErrors.NonceOutOfOrder(expected, nonce);
        }

        uint256 bucket = nonce / 256;
        uint256 bitPos = nonce % 256;
        uint256 mask = 1 << bitPos;
        uint256 current = _bitmaps[chainId][bridgeId][bucket];

        if (current & mask != 0) {
            revert CANBErrors.NonceAlreadyConsumed(chainId, bridgeId, nonce);
        }

        _bitmaps[chainId][bridgeId][bucket] = current | mask;
        nextExpectedNonce[chainId][bridgeId] = nonce;

        emit CANBEvents.NonceConsumed(chainId, bridgeId, nonce);
        return true;
    }

    /// @inheritdoc INonceStore
    function isNonceUsed(uint256 chainId, bytes32 bridgeId, uint256 nonce)
        external view override returns (bool)
    {
        return _isNonceSet(chainId, bridgeId, nonce);
    }

    function _isNonceSet(uint256 chainId, bytes32 bridgeId, uint256 nonce)
        internal view returns (bool)
    {
        uint256 bucket = nonce / 256;
        uint256 bitPos = nonce % 256;
        return (_bitmaps[chainId][bridgeId][bucket] & (1 << bitPos)) != 0;
    }
}
```
