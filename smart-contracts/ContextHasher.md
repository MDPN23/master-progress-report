# ContextHasher.sol

## Deskripsi dan Kegunaan
`ContextHasher.sol` adalah komponen kriptografis yang bertanggung jawab untuk menghitung *hash* konteks standar EIP-712 dari pesan-pesan jembatan (*bridge messages*).

Konteks ini sangat penting karena ia menyandikan SEMUA *field* atau kolom dari struktur `BridgeMessage`, termasuk ID rantai sumber (*source*) dan ID rantai tujuan (*destination*). Jika dikombinasikan dengan `DomainSeparator` (yang berisi `block.chainid` saat penyebaran), kontrak ini menciptakan sistem "pengikatan ganda" (*double binding*). Ini membuat setiap tanda tangan (*signature*) bersifat unik untuk satu rantai sumber tertentu ke satu rantai tujuan tertentu. Modul ini secara efektif melumpuhkan percobaan *replay attack* karena *hash* yang ditandatangani untuk satu rute lintas-rantai akan sama sekali tidak sah untuk rute lainnya.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {DomainSeparator} from "./DomainSeparator.sol";
import {IBridgeBenchmark} from "../interfaces/IBridgeBenchmark.sol";

/**
 * @title ContextHasher
 * @notice Computes EIP‑712 compliant context hashes for bridge messages.
 *
 * @dev The context hash encodes EVERY field of a BridgeMessage — including
 *      the source and destination chain IDs — inside an EIP‑712 typed‑data
 *      structure. Combined with the DomainSeparator (which itself contains
 *      `block.chainid`), this creates a double binding that makes signatures
 *      chain‑specific at two levels:
 *
 *        1. The struct hash includes `sourceChainId` + `destinationChainId`.
 *        2. The domain separator includes the deployment `chainId`.
 *
 *      An attacker who tries to replay a signature on a different chain will
 *      fail at step 2 even if they somehow forge the struct fields.
 */
contract ContextHasher {
    // ─── Constants ──────────────────────────────────────────────────────

    /// @dev EIP‑712 type hash for BridgeMessage.
    bytes32 public constant BRIDGE_MESSAGE_TYPEHASH =
        keccak256(
            "BridgeMessage(address sender,address recipient,address token,"
            "uint256 amount,uint256 nonce,uint256 sourceChainId,"
            "uint256 destinationChainId,uint8 messageType)"
        );

    // ─── Immutables ─────────────────────────────────────────────────────

    /// @notice Reference to the DomainSeparator contract.
    DomainSeparator public immutable domainSeparator;

    // ─── Constructor ────────────────────────────────────────────────────

    constructor(address _domainSeparator) {
        require(_domainSeparator != address(0), "ContextHasher: zero address");
        domainSeparator = DomainSeparator(_domainSeparator);
    }

    // ─── Public ─────────────────────────────────────────────────────────

    /// @notice Compute the EIP‑712 struct hash for a BridgeMessage.
    /// @param message  The bridge message to hash.
    /// @return structHash  The keccak256 hash of the encoded struct.
    function computeStructHash(
        IBridgeBenchmark.BridgeMessage calldata message
    ) public pure returns (bytes32 structHash) {
        structHash = keccak256(
            abi.encode(
                BRIDGE_MESSAGE_TYPEHASH,
                message.sender,
                message.recipient,
                message.token,
                message.amount,
                message.nonce,
                message.sourceChainId,
                message.destinationChainId,
                uint8(message.messageType)
            )
        );
    }

    /// @notice Compute the full EIP‑712 context hash (digest).
    /// @dev This is the value that gets signed by the relayer and recovered
    ///      during validation: `keccak256("\x19\x01" ‖ domainSeparator ‖ structHash)`.
    /// @param message  The bridge message to hash.
    /// @return contextHash  The final EIP‑712 digest.
    function computeContextHash(
        IBridgeBenchmark.BridgeMessage calldata message
    ) public view returns (bytes32 contextHash) {
        bytes32 structHash = computeStructHash(message);
        contextHash = keccak256(
            abi.encodePacked(
                "\x19\x01",
                domainSeparator.getDomainSeparator(),
                structHash
            )
        );
    }
}
```
