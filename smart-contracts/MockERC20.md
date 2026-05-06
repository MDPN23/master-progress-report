# MockERC20.sol

## Deskripsi dan Kegunaan
Implementasi standar Mock ERC20 untuk keperluan minting dan pengujian fungsionalitas jembatan tanpa menggunakan aset bernilai nyata.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title MockERC20
 * @notice Lightweight ERC‑20 test token for bridge experiments.
 * @dev Used by both BaselineExecutor and BridgeExecutor for mint/release
 *      operations. The `owner` is set to the deployer and can be transferred
 *      to a bridge contract so that only the bridge can mint new supply.
 *
 *      Inherits ERC20Burnable so bridge contracts can call `burnFrom()`.
 */
contract MockERC20 is ERC20, ERC20Burnable, Ownable {
    uint8 private immutable _DECIMALS;

    /**
     * @param name_          Token name (e.g. "Mock TelU Token")
     * @param symbol_        Token symbol (e.g. "mTELU")
     * @param decimals_      Number of decimals (typically 18)
     * @param initialSupply  Amount to mint to the deployer **in whole units**
     */
    constructor(
        string memory name_,
        string memory symbol_,
        uint8 decimals_,
        uint256 initialSupply
    ) ERC20(name_, symbol_) Ownable(msg.sender) {
        _DECIMALS = decimals_;
        if (initialSupply > 0) {
            _mint(msg.sender, initialSupply * 10 ** decimals_);
        }
    }

    // ─── Views ──────────────────────────────────────────────────────────

    function decimals() public view override returns (uint8) {
        return _DECIMALS;
    }

    // ─── Owner Functions ────────────────────────────────────────────────

    /// @notice Mint new tokens. Only callable by the contract owner (bridge).
    /// @param to      Recipient address.
    /// @param amount  Amount in the smallest unit.
    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }

    /// @notice Mint tokens to multiple recipients in a single transaction.
    /// @param recipients  Array of recipient addresses.
    /// @param amounts     Array of amounts (must match recipients length).
    function mintBatch(
        address[] calldata recipients,
        uint256[] calldata amounts
    ) external onlyOwner {
        require(recipients.length == amounts.length, "MockERC20: length mismatch");
        for (uint256 i = 0; i < recipients.length; i++) {
            _mint(recipients[i], amounts[i]);
        }
    }
}

```
