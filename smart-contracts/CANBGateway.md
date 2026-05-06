# CANBGateway.sol

## Deskripsi dan Kegunaan
`CANBGateway.sol` adalah titik masuk utama (*entry point*) untuk jembatan lintas-rantai (Cross-chain Adaptive Nonce Bridge - CANB). Kontrak ini berfungsi sebagai gerbang utama (*gateway*) yang mengatur izin (*Access Control List - ACL*) untuk *relayer* dan memproses semua operasi masuk (*inbound*) maupun keluar (*outbound*).

Semua pesan masuk harus melewati proses validasi di `ValidationModule` sebelum eksekusi (seperti pencetakan atau pelepasan aset) diserahkan ke `BridgeExecutor`. Pada arah keluar, ia mengelola proses penguncian (*lock*) dan pembakaran (*burn*) aset pengguna.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {IBridgeBenchmark} from "../interfaces/IBridgeBenchmark.sol";
import {IValidationModule} from "../interfaces/IValidationModule.sol";
import {ValidationModule} from "./ValidationModule.sol";
import {BridgeExecutor} from "./BridgeExecutor.sol";
import {CANBErrors} from "../libraries/CANBErrors.sol";
import {CANBEvents} from "../libraries/CANBEvents.sol";

/**
 * @title CANBGateway
 * @notice Entry point + ACL for the CANB (Cross-chain Adaptive Nonce Bridge).
 * @dev This is the secured bridge that serves as the treatment group.
 *      All inbound messages pass through ValidationModule before execution.
 *      Outbound operations (lock/burn) are handled directly.
 */
contract CANBGateway is IBridgeBenchmark {
    address public owner;
    ValidationModule public validationModule;
    BridgeExecutor public executor;

    mapping(address => bool) public relayers;
    uint256 private _nonceCounter;

    modifier onlyOwner() {
        if (msg.sender != owner) revert CANBErrors.NotOwner();
        _;
    }

    modifier onlyAuthorizedRelayer() {
        if (!relayers[msg.sender]) revert CANBErrors.UnauthorizedRelayer(msg.sender);
        _;
    }

    constructor(address _validationModule, address _executor) {
        require(_validationModule != address(0) && _executor != address(0),
            "CANBGateway: zero address");
        owner = msg.sender;
        validationModule = ValidationModule(_validationModule);
        executor = BridgeExecutor(_executor);
    }

    // ─── Admin ──────────────────────────────────────────────────────────

    function setRelayer(address relayer, bool status) external onlyOwner {
        if (relayer == address(0)) revert CANBErrors.ZeroAddress();
        relayers[relayer] = status;
        if (status) emit CANBEvents.RelayerAdded(relayer);
        else emit CANBEvents.RelayerRemoved(relayer);
    }

    // ─── IBridgeBenchmark — Inbound ─────────────────────────────────────

    /// @inheritdoc IBridgeBenchmark
    function processMessage(
        BridgeMessage calldata message,
        bytes calldata signature
    ) external override onlyAuthorizedRelayer {
        // Full validation pipeline
        (bool valid, bytes32 contextHash) = validationModule.validateMessage(message, signature);
        require(valid, "CANBGateway: validation failed");

        // Execute based on message type
        if (message.messageType == MessageType.MINT) {
            executor.mintTokens(message.recipient, message.amount, contextHash);
        } else if (message.messageType == MessageType.RELEASE) {
            executor.releaseTokens(message.recipient, message.amount, contextHash);
        } else {
            revert CANBErrors.InvalidMessageType(uint8(message.messageType));
        }
    }

    // ─── IBridgeBenchmark — Outbound ────────────────────────────────────

    /// @inheritdoc IBridgeBenchmark
    function lockAssets(
        address token,
        uint256 amount,
        uint256 destinationChainId
    ) external override {
        if (amount == 0) revert CANBErrors.ZeroAmount();
        if (destinationChainId == 0) revert CANBErrors.InvalidChainId();
        if (destinationChainId == block.chainid) revert CANBErrors.SameChainRoute(block.chainid);

        _nonceCounter++;
        uint256 nonce = _nonceCounter;

        // Transfer tokens from sender to executor (lock)
        (bool success, ) = token.call(
            abi.encodeWithSignature(
                "transferFrom(address,address,uint256)",
                msg.sender, address(executor), amount
            )
        );
        if (!success) revert CANBErrors.TokenTransferFailed();

        executor.recordDeposit(msg.sender, amount);

        bytes32 contextHash = keccak256(
            abi.encodePacked(msg.sender, token, amount, nonce, block.chainid, destinationChainId)
        );

        emit CANBEvents.AssetsLocked(msg.sender, token, amount, nonce, destinationChainId, contextHash);
    }

    /// @inheritdoc IBridgeBenchmark
    function burnAssets(
        address token,
        uint256 amount,
        uint256 destinationChainId
    ) external override {
        if (amount == 0) revert CANBErrors.ZeroAmount();
        if (destinationChainId == 0) revert CANBErrors.InvalidChainId();
        if (destinationChainId == block.chainid) revert CANBErrors.SameChainRoute(block.chainid);

        _nonceCounter++;
        uint256 nonce = _nonceCounter;

        (bool success, ) = token.call(
            abi.encodeWithSignature("burnFrom(address,uint256)", msg.sender, amount)
        );
        if (!success) revert CANBErrors.TokenTransferFailed();

        bytes32 contextHash = keccak256(
            abi.encodePacked(msg.sender, token, amount, nonce, block.chainid, destinationChainId)
        );

        emit CANBEvents.AssetsBurned(msg.sender, token, amount, nonce, destinationChainId, contextHash);
    }

    /// @inheritdoc IBridgeBenchmark
    function isAuthorizedRelayer(address relayer) external view override returns (bool) {
        return relayers[relayer];
    }
}
```
