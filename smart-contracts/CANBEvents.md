# CANBEvents.sol

## Deskripsi dan Kegunaan
Pustaka (Library) yang mendefinisikan events log yang di-emit oleh sistem CANB untuk pelacakan off-chain.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/**
 * @title CANBEvents
 * @notice Event definitions for the CANB framework.
 * @dev Every validation attempt — whether it passes or fails — emits an event
 *      so that off‑chain monitoring can track bridge activity in real time.
 */
library CANBEvents {
    // ─── Bridge Lifecycle ───────────────────────────────────────────────

    /// @dev Emitted when assets are locked on the source chain.
    event AssetsLocked(
        address indexed sender,
        address indexed token,
        uint256 amount,
        uint256 nonce,
        uint256 destinationChainId,
        bytes32 contextHash
    );

    /// @dev Emitted when wrapped assets are minted on the destination chain.
    event AssetsMinted(
        address indexed recipient,
        address indexed token,
        uint256 amount,
        uint256 nonce,
        uint256 sourceChainId,
        bytes32 contextHash
    );

    /// @dev Emitted when wrapped assets are burned to initiate a release.
    event AssetsBurned(
        address indexed sender,
        address indexed token,
        uint256 amount,
        uint256 nonce,
        uint256 destinationChainId,
        bytes32 contextHash
    );

    /// @dev Emitted when locked assets are released back on the source chain.
    event AssetsReleased(
        address indexed recipient,
        address indexed token,
        uint256 amount,
        uint256 nonce,
        uint256 sourceChainId,
        bytes32 contextHash
    );

    // ─── Validation Pipeline ────────────────────────────────────────────

    /// @dev Emitted when a message passes all validation stages.
    event ValidationPassed(
        bytes32 indexed contextHash,
        address indexed relayer,
        uint256 nonce
    );

    /// @dev Emitted when a message fails validation at any stage.
    event ValidationFailed(
        bytes32 indexed contextHash,
        address indexed relayer,
        string reason
    );

    // ─── Nonce Management ───────────────────────────────────────────────

    /// @dev Emitted when a nonce is successfully consumed.
    event NonceConsumed(
        uint256 indexed chainId,
        bytes32 indexed bridgeId,
        uint256 nonce
    );

    // ─── Access Control ─────────────────────────────────────────────────

    /// @dev Emitted when a relayer is added to the authorized set.
    event RelayerAdded(address indexed relayer);

    /// @dev Emitted when a relayer is removed from the authorized set.
    event RelayerRemoved(address indexed relayer);
}

```
