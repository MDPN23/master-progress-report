# mockTelU1.sol

## Deskripsi dan Kegunaan
Kontrak token mock TelU1 yang digunakan untuk simulasi aset asli dalam pengujian jembatan lintas-rantai.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title MockTelU1
 * @dev A simple ERC-20 token for DeFi use cases.
 *
 * Features:
 *  - Standard ERC-20 (transfer, approve, transferFrom)
 *  - Burnable (holders can burn their own tokens)
 *  - Permit (gasless approvals via EIP-2612 signatures)
 *  - Owner-only minting
 *  - Configurable decimals
 */
contract MockTelU1 is ERC20, ERC20Burnable, ERC20Permit, Ownable {
    uint8 private immutable _DECIMALS;

    /**
     * @param name_        Token name (e.g. "TelU Token")
     * @param symbol_      Token symbol (e.g. "TELU")
     * @param decimals_    Number of decimals (commonly 18)
     * @param initialSupply  Amount of tokens to mint to the deployer (in whole units, NOT wei)
     */
    constructor(
        string memory name_,
        string memory symbol_,
        uint8 decimals_,
        uint256 initialSupply
    )
        ERC20(name_, symbol_)
        ERC20Permit(name_)
        Ownable(msg.sender)
    {
        _DECIMALS = decimals_;
        _mint(msg.sender, initialSupply * 10 ** decimals_);
    }

    // ─── Views ───────────────────────────────────────────────────────────

    function decimals() public view override returns (uint8) {
        return _DECIMALS;
    }

    // ─── Owner Functions ─────────────────────────────────────────────────

    /**
     * @dev Mint new tokens. Only callable by the contract owner.
     * @param to       Recipient address
     * @param amount   Amount in the smallest unit (wei-equivalent)
     */
    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }

    /**
     * @dev Convenience function: mint tokens to multiple addresses at once.
     * @param recipients  Array of recipient addresses
     * @param amounts     Array of amounts (must match recipients length)
     */
    function mintBatch(
        address[] calldata recipients,
        uint256[] calldata amounts
    ) external onlyOwner {
        require(recipients.length == amounts.length, "Length mismatch");
        for (uint256 i = 0; i < recipients.length; i++) {
            _mint(recipients[i], amounts[i]);
        }
    }
}

```
