# BaselineGateway.sol

## Deskripsi dan Kegunaan
Kontrak gerbang utama (Gateway) baseline tanpa proteksi CANB. Digunakan sebagai kelompok kontrol (control group) dalam pengujian keamanan.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import {MessageHashUtils} from "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";
import {IBridgeBenchmark} from "../interfaces/IBridgeBenchmark.sol";

/**
 * @title BaselineGateway
 * @notice Intentionally VULNERABLE baseline bridge — the control group.
 *
 * @dev This contract deliberately omits several security measures to serve as
 *      the "before" benchmark in the thesis comparison:
 *
 *      V1 — No replay protection:  No nonce tracking or message‑hash registry.
 *      V2 — No chain‑ID binding:   Signature is valid on ANY chain.
 *      V3 — Weak sig verification: Uses raw `ecrecover` without EIP‑712 domain.
 *      V4 — CEI violation:         Transfer before state update in release path.
 *
 *      ⚠️  DO NOT deploy to production. This is a research artefact.
 */
contract BaselineGateway is IBridgeBenchmark {
    // ─── State ──────────────────────────────────────────────────────────
    address public owner;
    address public executor;
    mapping(address => bool) public relayers;

    // ─── Events (minimal — no contextHash, no chainId) ──────────────────
    event Deposit(address indexed sender, address indexed token, uint256 amount);
    event MessageProcessed(address indexed sender, uint256 amount);

    // ─── Modifiers ──────────────────────────────────────────────────────
    modifier onlyOwner() {
        require(msg.sender == owner, "Baseline: not owner");
        _;
    }

    // ─── Constructor ────────────────────────────────────────────────────
    constructor() {
        owner = msg.sender;
    }

    // ─── Admin ──────────────────────────────────────────────────────────

    /// @notice Set the executor contract address.
    function setExecutor(address _executor) external onlyOwner {
        require(_executor != address(0), "Baseline: zero address");
        executor = _executor;
    }

    /// @notice Register or revoke a relayer.
    function setRelayer(address _relayer, bool _status) external onlyOwner {
        require(_relayer != address(0), "Baseline: zero address");
        relayers[_relayer] = _status;
    }

    // ─── IBridgeBenchmark ───────────────────────────────────────────────

    /// @inheritdoc IBridgeBenchmark
    /// @dev V1 + V2 + V3: No nonce check, no chain‑ID binding, weak sig verify.
    ///      The signature is recovered using raw `ECDSA.recover` over a plain
    ///      hash (no EIP‑712 domain separator), and the recovered address is
    ///      only checked against `relayers` — without any replay guard.
    function processMessage(
        BridgeMessage calldata message,
        bytes calldata signature
    ) external override {
        // V3: Weak signature — plain keccak hash, no EIP‑712
        bytes32 rawHash = keccak256(
            abi.encodePacked(
                message.sender,
                message.recipient,
                message.token,
                message.amount,
                message.nonce,
                uint8(message.messageType)
                // V2: sourceChainId and destinationChainId intentionally EXCLUDED
            )
        );

        bytes32 ethSignedHash = MessageHashUtils.toEthSignedMessageHash(rawHash);
        address recovered = ECDSA.recover(ethSignedHash, signature);

        // Basic relayer check (but no nonce/replay guard → V1)
        require(relayers[recovered], "Baseline: invalid relayer");

        // V1: No `executedHashes` mapping — same message can be processed
        //     an unlimited number of times (replay vulnerability).

        emit MessageProcessed(message.sender, message.amount);
    }

    /// @inheritdoc IBridgeBenchmark
    function lockAssets(
        address token,
        uint256 amount,
        uint256 destinationChainId
    ) external override {
        require(amount > 0, "Baseline: zero amount");

        // Transfer tokens from sender to this contract (lock)
        (bool success, ) = token.call(
            abi.encodeWithSignature(
                "transferFrom(address,address,uint256)",
                msg.sender,
                address(this),
                amount
            )
        );
        require(success, "Baseline: transferFrom failed");

        // V2: No chainId in event
        emit Deposit(msg.sender, token, amount);
    }

    /// @inheritdoc IBridgeBenchmark
    function burnAssets(
        address token,
        uint256 amount,
        uint256 destinationChainId
    ) external override {
        require(amount > 0, "Baseline: zero amount");

        // Call burnFrom on the token (requires prior approval)
        (bool success, ) = token.call(
            abi.encodeWithSignature(
                "burnFrom(address,uint256)",
                msg.sender,
                amount
            )
        );
        require(success, "Baseline: burnFrom failed");

        emit Deposit(msg.sender, token, amount);
    }

    /// @inheritdoc IBridgeBenchmark
    function isAuthorizedRelayer(
        address relayer
    ) external view override returns (bool) {
        return relayers[relayer];
    }
}

```
