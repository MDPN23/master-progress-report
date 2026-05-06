# IBridgeBenchmark.sol

## Deskripsi dan Kegunaan
Antarmuka (Interface) untuk mendefinisikan standar pengujian benchmark pada jembatan lintas-rantai.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/**
 * @title IBridgeBenchmark
 * @notice Shared interface for both BaselineGateway and CANBGateway.
 * @dev Provides a uniform surface for gas benchmarking and comparison testing.
 *      Both the vulnerable baseline and the secured CANB implementations
 *      conform to this interface so the ComparisonBenchmark test can invoke
 *      identical call‐paths on each.
 */
interface IBridgeBenchmark {
    // ─── Message Types ──────────────────────────────────────────────────
    enum MessageType {
        LOCK,       // Lock assets on source chain
        BURN,       // Burn wrapped assets for release
        MINT,       // Mint wrapped assets on destination
        RELEASE     // Release locked assets on source
    }

    // ─── Structs ────────────────────────────────────────────────────────
    struct BridgeMessage {
        address sender;
        address recipient;
        address token;
        uint256 amount;
        uint256 nonce;
        uint256 sourceChainId;
        uint256 destinationChainId;
        MessageType messageType;
    }

    // ─── Functions ──────────────────────────────────────────────────────

    /// @notice Process an inbound bridge message (lock, burn, mint, or release).
    /// @param message  The bridge message payload.
    /// @param signature  Relayer/validator ECDSA signature over the message.
    function processMessage(
        BridgeMessage calldata message,
        bytes calldata signature
    ) external;

    /// @notice Lock ERC‑20 tokens on the source chain.
    /// @param token   Address of the ERC‑20 token to lock.
    /// @param amount  Amount of tokens (in smallest unit).
    /// @param destinationChainId  Target chain for minting.
    function lockAssets(
        address token,
        uint256 amount,
        uint256 destinationChainId
    ) external;

    /// @notice Burn wrapped tokens to initiate a release on the source chain.
    /// @param token   Address of the wrapped ERC‑20 token.
    /// @param amount  Amount of tokens to burn.
    /// @param destinationChainId  Target chain for releasing.
    function burnAssets(
        address token,
        uint256 amount,
        uint256 destinationChainId
    ) external;

    /// @notice Check whether a given relayer address is authorized.
    /// @param relayer  Address to check.
    /// @return True if the address is an authorized relayer.
    function isAuthorizedRelayer(address relayer) external view returns (bool);
}

```
