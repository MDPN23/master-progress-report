# INonceStore.sol

## Deskripsi dan Kegunaan
Antarmuka untuk manajemen Nonce, mendefinisikan fungsi penyimpanan dan konsumsi nonce secara aman.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/**
 * @title INonceStore
 * @notice Interface for bitmap‑based nonce management.
 * @dev Nonces are tracked per (chainId, bridgeId) tuple using a bitmap
 *      to minimise storage writes and gas consumption.
 */
interface INonceStore {
    /// @notice Consume a nonce, marking it as used.
    /// @param chainId   The source chain identifier.
    /// @param bridgeId  Unique bridge instance identifier.
    /// @param nonce     The nonce value to consume.
    /// @return success  True if the nonce was successfully consumed.
    function consumeNonce(
        uint256 chainId,
        bytes32 bridgeId,
        uint256 nonce
    ) external returns (bool success);

    /// @notice Check whether a nonce has already been used.
    /// @param chainId   The source chain identifier.
    /// @param bridgeId  Unique bridge instance identifier.
    /// @param nonce     The nonce value to query.
    /// @return used     True if the nonce was previously consumed.
    function isNonceUsed(
        uint256 chainId,
        bytes32 bridgeId,
        uint256 nonce
    ) external view returns (bool used);
}

```
