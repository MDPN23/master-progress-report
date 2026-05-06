# DeployAll.s.sol

## Deskripsi dan Kegunaan
Skrip Foundry untuk melakukan deployment secara terstruktur untuk seluruh komponen (Baseline dan CANB) pada jaringan simulasi.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Script.sol";
import {MockERC20} from "../src/token/MockERC20.sol";
import {BaselineGateway} from "../src/baseline/BaselineGateway.sol";
import {BaselineExecutor} from "../src/baseline/BaselineExecutor.sol";
import {DomainSeparator} from "../src/canb/DomainSeparator.sol";
import {ContextHasher} from "../src/canb/ContextHasher.sol";
import {NonceStore} from "../src/canb/NonceStore.sol";
import {ContextValidator} from "../src/canb/ContextValidator.sol";
import {ValidationModule} from "../src/canb/ValidationModule.sol";
import {BridgeExecutor} from "../src/canb/BridgeExecutor.sol";
import {CANBGateway} from "../src/canb/CANBGateway.sol";

/**
 * @title DeployAll
 * @notice Deploys the complete CANB experiment infrastructure:
 *         - MockERC20 token
 *         - Baseline bridge (gateway + executor)
 *         - CANB bridge (full stack: domain separator, hasher, nonce store,
 *           context validator, validation module, executor, gateway)
 *
 * @dev Usage:
 *   forge script script/DeployAll.s.sol \
 *     --rpc-url $RPC_URL \
 *     --broadcast \
 *     --private-key $DEPLOYER_KEY \
 *     -vvvv
 *
 *   Environment variables:
 *     RELAYER_ADDRESS — Address of the authorized relayer (default: deployer)
 *     TOKEN_NAME      — ERC-20 token name (default: "Mock TelU Token")
 *     TOKEN_SYMBOL    — ERC-20 token symbol (default: "mTELU")
 *     INITIAL_SUPPLY  — Initial token supply in whole units (default: 1_000_000)
 */
contract DeployAll is Script {
    function run() external {
        // ─── Configuration ──────────────────────────────────────────────
        address relayerAddr = vm.envOr("RELAYER_ADDRESS", msg.sender);
        string memory tokenName = vm.envOr("TOKEN_NAME", string("Mock TelU Token"));
        string memory tokenSymbol = vm.envOr("TOKEN_SYMBOL", string("mTELU"));
        uint256 initialSupply = vm.envOr("INITIAL_SUPPLY", uint256(1_000_000));

        vm.startBroadcast();

        // ─── 1. Token ───────────────────────────────────────────────────
        MockERC20 token = new MockERC20(tokenName, tokenSymbol, 18, initialSupply);
        console.log("MockERC20 deployed at:", address(token));

        // ─── 2. Baseline Bridge ─────────────────────────────────────────
        BaselineGateway baselineGw = new BaselineGateway();
        BaselineExecutor baselineExec = new BaselineExecutor(address(token));
        baselineGw.setRelayer(relayerAddr, true);
        baselineGw.setExecutor(address(baselineExec));
        baselineExec.setGateway(address(baselineGw));

        console.log("BaselineGateway deployed at:", address(baselineGw));
        console.log("BaselineExecutor deployed at:", address(baselineExec));

        // ─── 3. CANB Bridge Stack ───────────────────────────────────────

        // 3a. Deploy sub-modules
        DomainSeparator domainSep = new DomainSeparator();
        console.log("DomainSeparator deployed at:", address(domainSep));

        ContextHasher ctxHasher = new ContextHasher(address(domainSep));
        console.log("ContextHasher deployed at:", address(ctxHasher));

        NonceStore nonceStore = new NonceStore();
        console.log("NonceStore deployed at:", address(nonceStore));

        ContextValidator ctxValidator = new ContextValidator();
        console.log("ContextValidator deployed at:", address(ctxValidator));

        // 3b. Deploy ValidationModule (orchestrator)
        ValidationModule valMod = new ValidationModule(
            address(domainSep),
            address(ctxHasher),
            address(nonceStore),
            address(ctxValidator)
        );
        console.log("ValidationModule deployed at:", address(valMod));

        // 3c. Authorize ValidationModule in NonceStore
        nonceStore.setAuthorizedCaller(address(valMod), true);

        // 3d. Register relayer in ValidationModule
        valMod.setRelayer(relayerAddr, true);

        // 3e. Deploy BridgeExecutor
        // Note: We create a separate token for CANB (mirror token)
        MockERC20 canbToken = new MockERC20(
            string.concat(tokenName, " (Wrapped)"),
            string.concat("w", tokenSymbol),
            18,
            0 // No initial supply — minted by bridge
        );
        console.log("CANB MockERC20 (wrapped) deployed at:", address(canbToken));

        BridgeExecutor bridgeExec = new BridgeExecutor(address(canbToken));
        console.log("BridgeExecutor deployed at:", address(bridgeExec));

        // 3f. Deploy CANBGateway (entry point)
        CANBGateway canbGw = new CANBGateway(address(valMod), address(bridgeExec));
        console.log("CANBGateway deployed at:", address(canbGw));

        // 3g. Wire up permissions
        bridgeExec.setGateway(address(canbGw));
        canbGw.setRelayer(relayerAddr, true);
        canbToken.transferOwnership(address(bridgeExec));

        // ─── Summary ────────────────────────────────────────────────────
        console.log("");
        console.log("=== DEPLOYMENT SUMMARY ===");
        console.log("Chain ID:", block.chainid);
        console.log("Relayer:", relayerAddr);
        console.log("");
        console.log("--- Baseline ---");
        console.log("Gateway:", address(baselineGw));
        console.log("Executor:", address(baselineExec));
        console.log("");
        console.log("--- CANB ---");
        console.log("Gateway:", address(canbGw));
        console.log("ValidationModule:", address(valMod));
        console.log("DomainSeparator:", address(domainSep));
        console.log("ContextHasher:", address(ctxHasher));
        console.log("NonceStore:", address(nonceStore));
        console.log("ContextValidator:", address(ctxValidator));
        console.log("BridgeExecutor:", address(bridgeExec));
        console.log("");
        console.log("--- Tokens ---");
        console.log("Source Token:", address(token));
        console.log("Wrapped Token:", address(canbToken));
        console.log("========================");

        vm.stopBroadcast();
    }
}

```
