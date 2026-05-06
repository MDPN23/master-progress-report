# IValidationModule.sol

## Deskripsi dan Kegunaan
Antarmuka untuk modul validasi, mendefinisikan fungsi-fungsi yang wajib diimplementasikan oleh ValidationModule.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {IBridgeBenchmark} from "./IBridgeBenchmark.sol";

/**
 * @title IValidationModule
 * @notice Interface for the CANB validation pipeline orchestrator.
 * @dev The ValidationModule aggregates DomainSeparator, ContextHasher,
 *      NonceStore, and ContextValidator into a single entry‑point
 *      that the CANBGateway invokes before executing any state change.
 */
interface IValidationModule {
    /// @notice Run the full validation pipeline on an inbound message.
    /// @param message    The bridge message to validate.
    /// @param signature  ECDSA signature produced by an authorized relayer.
    /// @return valid     True when every pipeline stage passes.
    /// @return contextHash  The computed EIP‑712 context hash for event logging.
    function validateMessage(
        IBridgeBenchmark.BridgeMessage calldata message,
        bytes calldata signature
    ) external returns (bool valid, bytes32 contextHash);

    /// @notice Verify a message without consuming the nonce (view‑only dry‑run).
    /// @param message    The bridge message to verify.
    /// @param signature  ECDSA signature over the message.
    /// @return valid     True when the signature and context are valid.
    function verifyMessage(
        IBridgeBenchmark.BridgeMessage calldata message,
        bytes calldata signature
    ) external view returns (bool valid);
}

```
