# ContextValidator.sol

## Deskripsi dan Kegunaan
`ContextValidator.sol` adalah modul validasi awal yang secara khusus memvalidasi "rute" dan "tipe" dari setiap pesan yang masuk sebelum diproses kriptografis lebih lanjut.

Kontrak ini bertugas memastikan integritas jalur logis (*routing logic*), seperti:
1. Memastikan ID rantai sumber (*source*) dan tujuan (*destination*) bukan nol dan saling berbeda.
2. Memastikan pesan yang masuk ke jembatan ini secara spesifik ditujukan untuk rantai (*chain*) ini (ID rantai tujuan sama dengan `block.chainid` tempat kontrak ini disebarkan).
3. Memastikan jenis pesan secara tegas valid dan didukung (misalnya, hanya `MINT` atau `RELEASE` yang boleh diproses untuk alur masuk).

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {IBridgeBenchmark} from "../interfaces/IBridgeBenchmark.sol";
import {CANBErrors} from "../libraries/CANBErrors.sol";

/**
 * @title ContextValidator
 * @notice Route and message-type validation for CANB bridge messages.
 * @dev Validates that:
 *   1. Source and destination chain IDs are non-zero and different.
 *   2. The destination chain ID matches the current deployment chain.
 *   3. The message type is valid for inbound processing.
 */
contract ContextValidator {
    uint256 public immutable deploymentChainId;

    constructor() {
        deploymentChainId = block.chainid;
    }

    /// @notice Validate route and message type of a bridge message.
    /// @param message The bridge message to validate.
    function validateContext(IBridgeBenchmark.BridgeMessage calldata message) external view {
        // Source and destination must be non-zero
        if (message.sourceChainId == 0) revert CANBErrors.InvalidChainId();
        if (message.destinationChainId == 0) revert CANBErrors.InvalidChainId();

        // Cannot bridge to the same chain
        if (message.sourceChainId == message.destinationChainId) {
            revert CANBErrors.SameChainRoute(message.sourceChainId);
        }

        // Destination must match this chain
        if (message.destinationChainId != deploymentChainId) {
            revert CANBErrors.ChainIdMismatch(deploymentChainId, message.destinationChainId);
        }

        // Message type must be MINT or RELEASE for inbound processing
        uint8 mt = uint8(message.messageType);
        if (mt != uint8(IBridgeBenchmark.MessageType.MINT) &&
            mt != uint8(IBridgeBenchmark.MessageType.RELEASE))
        {
            revert CANBErrors.InvalidMessageType(mt);
        }
    }
}
```
