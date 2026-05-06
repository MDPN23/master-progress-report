# BridgeExecutor.sol

## Deskripsi dan Kegunaan
`BridgeExecutor.sol` adalah modul yang beroperasi secara ketat memegang kendali aset, dan bertugas sebagai eksekutor aksi pencetakan (*mint*) atau pelepasan (*release*) token.

Kontrak ini mengimplementasikan pola keamanan **CEI (Checks-Effects-Interactions)** secara ketat. Hal ini berarti setiap pembaruan keadaan internal (*state update*) diselesaikan SEBELUM modul memanggil kontrak eksternal (seperti mentransfer token). Tujuan utamanya adalah untuk sepenuhnya menutup celah serangan *Reentrancy*, memastikan eksekusi manipulasi aset jembatan tetap kebal dari manipulasi rekursif. Kontrak ini hanya mengizinkan modifikasi perintah yang datang dari `CANBGateway`.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {MockERC20} from "../token/MockERC20.sol";
import {CANBErrors} from "../libraries/CANBErrors.sol";
import {CANBEvents} from "../libraries/CANBEvents.sol";

/**
 * @title BridgeExecutor
 * @notice Guarded mint/release executor for CANBGateway.
 * @dev Follows CEI (Checks-Effects-Interactions) pattern strictly:
 *      state updates happen BEFORE any external token call.
 */
contract BridgeExecutor {
    address public owner;
    address public gateway;
    MockERC20 public token;

    mapping(address => uint256) public lockedBalances;

    event TokensMinted(address indexed to, uint256 amount, bytes32 contextHash);
    event TokensReleased(address indexed to, uint256 amount, bytes32 contextHash);

    modifier onlyOwner() {
        if (msg.sender != owner) revert CANBErrors.NotOwner();
        _;
    }

    modifier onlyGateway() {
        require(msg.sender == gateway, "BridgeExec: not gateway");
        _;
    }

    constructor(address _token) {
        require(_token != address(0), "BridgeExec: zero token");
        owner = msg.sender;
        token = MockERC20(_token);
    }

    function setGateway(address _gateway) external onlyOwner {
        if (_gateway == address(0)) revert CANBErrors.ZeroAddress();
        gateway = _gateway;
    }

    /// @notice Mint wrapped tokens — only callable by gateway after validation.
    function mintTokens(address to, uint256 amount, bytes32 contextHash) external onlyGateway {
        if (amount == 0) revert CANBErrors.ZeroAmount();
        if (to == address(0)) revert CANBErrors.ZeroAddress();
        token.mint(to, amount);
        emit TokensMinted(to, amount, contextHash);
    }

    /// @notice Record deposit (lock) from a user.
    function recordDeposit(address from, uint256 amount) external onlyGateway {
        lockedBalances[from] += amount;
    }

    /// @notice Release locked tokens — CEI pattern enforced.
    function releaseTokens(address to, uint256 amount, bytes32 contextHash) external onlyGateway {
        if (amount == 0) revert CANBErrors.ZeroAmount();
        if (to == address(0)) revert CANBErrors.ZeroAddress();
        require(lockedBalances[to] >= amount, "BridgeExec: insufficient locked");

        // CEI: State update BEFORE external call
        lockedBalances[to] -= amount;

        // Interaction last
        token.transfer(to, amount);
        emit TokensReleased(to, amount, contextHash);
    }
}
```
