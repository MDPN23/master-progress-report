# CANBErrors.sol

## Deskripsi dan Kegunaan
Pustaka (Library) yang mendefinisikan custom errors untuk menghemat penggunaan gas saat terjadi kegagalan transaksi.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/**
 * @title CANBErrors
 * @notice Custom error definitions for the CANB framework.
 * @dev Custom errors are cheaper than `require(…, "string")` — each saves
 *      roughly 200+ gas on revert because no string is ABI‑encoded.
 */
library CANBErrors {
    // ─── Nonce Errors ───────────────────────────────────────────────────
    /// @dev Thrown when a nonce has already been consumed (replay attempt).
    error NonceAlreadyConsumed(uint256 chainId, bytes32 bridgeId, uint256 nonce);

    /// @dev Thrown when nonces are submitted out of sequential order.
    error NonceOutOfOrder(uint256 expected, uint256 received);

    // ─── Signature / Auth Errors ────────────────────────────────────────
    /// @dev Thrown when the recovered signer is not an authorized relayer.
    error UnauthorizedRelayer(address recovered);

    /// @dev Thrown when the ECDSA signature is malformed or invalid.
    error InvalidSignature();

    // ─── Context Errors ─────────────────────────────────────────────────
    /// @dev Thrown when the EIP‑712 context hash does not match expectations.
    error InvalidContextHash(bytes32 expected, bytes32 actual);

    /// @dev Thrown when the destination chain ID in the message does not
    ///      match the chain on which the contract is deployed.
    error ChainIdMismatch(uint256 expected, uint256 received);

    /// @dev Thrown when the message type is not valid for the current operation.
    error InvalidMessageType(uint8 messageType);

    // ─── Route Errors ───────────────────────────────────────────────────
    /// @dev Thrown when source and destination chain IDs are the same.
    error SameChainRoute(uint256 chainId);

    /// @dev Thrown when a chain ID is zero (invalid).
    error InvalidChainId();

    // ─── Access Control ─────────────────────────────────────────────────
    /// @dev Thrown when a non‑owner tries to call an owner‑only function.
    error NotOwner();

    /// @dev Thrown when the zero address is passed where a valid address is required.
    error ZeroAddress();

    // ─── Token Errors ───────────────────────────────────────────────────
    /// @dev Thrown when a token transfer or approval fails.
    error TokenTransferFailed();

    /// @dev Thrown when the bridged amount is zero.
    error ZeroAmount();
}

```
