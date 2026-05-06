# BaselineExecutor.sol

## Deskripsi dan Kegunaan
Kontrak eksekutor untuk sistem baseline. Mengeksekusi pencetakan dan pelepasan token tanpa proteksi tambahan (tanpa pola CEI ketat).

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {MockERC20} from "../token/MockERC20.sol";

/**
 * @title BaselineExecutor
 * @notice Mint and release executor for the BaselineGateway — no validation.
 *
 * @dev This executor deliberately performs NO additional security checks.
 *      It blindly mints or releases tokens when called by the gateway,
 *      making the baseline bridge fully exploitable for research purposes.
 *
 *      V4 (CEI violation): In `releaseAssets`, the token transfer happens
 *      BEFORE the state update, enabling reentrancy attacks.
 *
 *      ⚠️  DO NOT deploy to production. This is a research artefact.
 */
contract BaselineExecutor {
    // ─── State ──────────────────────────────────────────────────────────
    address public owner;
    address public gateway;
    MockERC20 public token;

    /// @dev Tracks locked balances per user (used in release flow).
    mapping(address => uint256) public lockedBalances;

    // ─── Events ─────────────────────────────────────────────────────────
    event TokensMinted(address indexed to, uint256 amount);
    event TokensReleased(address indexed to, uint256 amount);

    // ─── Modifiers ──────────────────────────────────────────────────────
    modifier onlyOwner() {
        require(msg.sender == owner, "BaselineExec: not owner");
        _;
    }

    modifier onlyGateway() {
        require(msg.sender == gateway, "BaselineExec: not gateway");
        _;
    }

    // ─── Constructor ────────────────────────────────────────────────────
    constructor(address _token) {
        owner = msg.sender;
        token = MockERC20(_token);
    }

    // ─── Admin ──────────────────────────────────────────────────────────

    /// @notice Set the gateway that is allowed to invoke mint/release.
    function setGateway(address _gateway) external onlyOwner {
        require(_gateway != address(0), "BaselineExec: zero address");
        gateway = _gateway;
    }

    // ─── Execution ──────────────────────────────────────────────────────

    /// @notice Mint wrapped tokens to a recipient — NO validation checks.
    /// @param to      Recipient of the minted tokens.
    /// @param amount  Amount to mint.
    function mintTokens(address to, uint256 amount) external onlyGateway {
        // No nonce check, no signature check — intentionally vulnerable.
        token.mint(to, amount);
        emit TokensMinted(to, amount);
    }

    /// @notice Record a deposit (lock) from a user.
    /// @param from    User whose tokens were locked.
    /// @param amount  Locked amount.
    function recordDeposit(address from, uint256 amount) external onlyGateway {
        lockedBalances[from] += amount;
    }

    /// @notice Release locked tokens back to user.
    /// @dev V4: Transfer happens BEFORE state update → reentrancy‑vulnerable.
    /// @param to      Recipient of released tokens.
    /// @param amount  Amount to release.
    function releaseTokens(address to, uint256 amount) external onlyGateway {
        require(lockedBalances[to] >= amount, "BaselineExec: insufficient locked");

        // ⚠️  V4: CEI VIOLATION — transfer before state change
        token.transfer(to, amount);

        // State update happens AFTER external call → reentrancy window
        lockedBalances[to] -= amount;

        emit TokensReleased(to, amount);
    }
}

```
