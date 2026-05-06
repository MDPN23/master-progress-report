# ValidationModule.sol

## Deskripsi dan Kegunaan
`ValidationModule.sol` berperan sebagai **Orkestrator Jalur Pipa Validasi (Validation Pipeline Orchestrator)**. Kontrak ini adalah jantung keamanan dari *framework* CANB.

Ia memanggil dan mengagregasi semua tahap validasi secara berurutan saat *Gateway* menerima pesan dari relayer, yaitu:
1. Menjalankan pemeriksaan rute `ContextValidator`.
2. Mengamankan integritas struktur menggunakan *hash* `ContextHasher`.
3. Memulihkan (*recover*) alamat penandatangan (`ECDSA.recover`) dari *hash* yang diamankan tadi, lalu mencocokkannya dengan daftar *relayer* resmi.
4. Terakhir, jika tanda tangan sah, ia "mengkonsumsi" (*consume*) nonce lewat `NonceStore` sehingga tanda tangan ini tidak pernah bisa digunakan ulang selamanya.

Hanya jika keempat langkah ini lulus dengan sempurna, fungsi akan mengembalikan status valid yang merangsang *Gateway* untuk mencairkan atau mencetak token.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import {IBridgeBenchmark} from "../interfaces/IBridgeBenchmark.sol";
import {IValidationModule} from "../interfaces/IValidationModule.sol";
import {DomainSeparator} from "./DomainSeparator.sol";
import {ContextHasher} from "./ContextHasher.sol";
import {NonceStore} from "./NonceStore.sol";
import {ContextValidator} from "./ContextValidator.sol";
import {CANBErrors} from "../libraries/CANBErrors.sol";
import {CANBEvents} from "../libraries/CANBEvents.sol";

/**
 * @title ValidationModule
 * @notice Pipeline orchestrator — aggregates all CANB validation stages.
 * @dev Validation pipeline order:
 *   1. ContextValidator  — route + message type checks
 *   2. ContextHasher     — compute EIP-712 context hash
 *   3. ECDSA.recover     — verify relayer signature
 *   4. NonceStore        — consume nonce (replay guard)
 */
contract ValidationModule is IValidationModule {
    DomainSeparator public immutable domainSep;
    ContextHasher public immutable hasher;
    NonceStore public immutable nonceStore;
    ContextValidator public immutable ctxValidator;

    address public owner;
    mapping(address => bool) public authorizedRelayers;
    bytes32 public immutable bridgeId;

    modifier onlyOwner() {
        if (msg.sender != owner) revert CANBErrors.NotOwner();
        _;
    }

    constructor(
        address _domainSep,
        address _hasher,
        address _nonceStore,
        address _ctxValidator
    ) {
        require(_domainSep != address(0) && _hasher != address(0)
            && _nonceStore != address(0) && _ctxValidator != address(0),
            "ValidationModule: zero address");

        owner = msg.sender;
        domainSep = DomainSeparator(_domainSep);
        hasher = ContextHasher(_hasher);
        nonceStore = NonceStore(_nonceStore);
        ctxValidator = ContextValidator(_ctxValidator);
        bridgeId = keccak256(abi.encodePacked("CANB-BRIDGE", block.chainid, address(this)));
    }

    // ─── Admin ──────────────────────────────────────────────────────────

    function setRelayer(address relayer, bool status) external onlyOwner {
        if (relayer == address(0)) revert CANBErrors.ZeroAddress();
        authorizedRelayers[relayer] = status;
        if (status) {
            emit CANBEvents.RelayerAdded(relayer);
        } else {
            emit CANBEvents.RelayerRemoved(relayer);
        }
    }

    // ─── IValidationModule ──────────────────────────────────────────────

    /// @inheritdoc IValidationModule
    function validateMessage(
        IBridgeBenchmark.BridgeMessage calldata message,
        bytes calldata signature
    ) external override returns (bool, bytes32) {
        // Stage 1: Route + type validation
        ctxValidator.validateContext(message);

        // Stage 2: Compute EIP-712 context hash
        bytes32 contextHash = hasher.computeContextHash(message);

        // Stage 3: Recover signer and verify authorization
        address recovered = ECDSA.recover(contextHash, signature);
        if (!authorizedRelayers[recovered]) {
            emit CANBEvents.ValidationFailed(contextHash, recovered, "unauthorized relayer");
            revert CANBErrors.UnauthorizedRelayer(recovered);
        }

        // Stage 4: Consume nonce (reverts on duplicate or out-of-order)
        nonceStore.consumeNonce(message.sourceChainId, bridgeId, message.nonce);

        emit CANBEvents.ValidationPassed(contextHash, recovered, message.nonce);
        return (true, contextHash);
    }

    /// @inheritdoc IValidationModule
    function verifyMessage(
        IBridgeBenchmark.BridgeMessage calldata message,
        bytes calldata signature
    ) external view override returns (bool) {
        // Dry-run: verify without consuming nonce
        bytes32 contextHash = hasher.computeContextHash(message);
        address recovered = ECDSA.recover(contextHash, signature);

        if (!authorizedRelayers[recovered]) return false;
        if (nonceStore.isNonceUsed(message.sourceChainId, bridgeId, message.nonce)) return false;

        return true;
    }
}
```
