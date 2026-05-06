# ValidationModule.t.sol

## Deskripsi dan Kegunaan
Pengujian unit untuk keseluruhan alur pipa validasi pesan masuk.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "forge-std/Test.sol";
import {DomainSeparator} from "../../src/canb/DomainSeparator.sol";
import {ContextHasher} from "../../src/canb/ContextHasher.sol";
import {NonceStore} from "../../src/canb/NonceStore.sol";
import {ContextValidator} from "../../src/canb/ContextValidator.sol";
import {ValidationModule} from "../../src/canb/ValidationModule.sol";
import {IBridgeBenchmark} from "../../src/interfaces/IBridgeBenchmark.sol";
import {CANBErrors} from "../../src/libraries/CANBErrors.sol";

contract ValidationModuleTest is Test {
    ValidationModule vm_mod;
    DomainSeparator ds;
    ContextHasher hasher;
    NonceStore nonceStore;
    ContextValidator ctxValidator;

    uint256 relayerPk = 0xA11CE;
    address relayer;
    uint256 SOURCE_CHAIN = 4202;

    function setUp() public {
        vm.chainId(84532);
        relayer = vm.addr(relayerPk);

        ds = new DomainSeparator();
        hasher = new ContextHasher(address(ds));
        nonceStore = new NonceStore();
        ctxValidator = new ContextValidator();

        vm_mod = new ValidationModule(
            address(ds), address(hasher), address(nonceStore), address(ctxValidator)
        );

        // Authorize ValidationModule to consume nonces
        nonceStore.setAuthorizedCaller(address(vm_mod), true);
        // Authorize relayer
        vm_mod.setRelayer(relayer, true);
    }

    function _makeMessage(uint256 nonce) internal view returns (IBridgeBenchmark.BridgeMessage memory) {
        return IBridgeBenchmark.BridgeMessage({
            sender: address(0xA),
            recipient: address(0xB),
            token: address(0xC),
            amount: 100 ether,
            nonce: nonce,
            sourceChainId: SOURCE_CHAIN,
            destinationChainId: block.chainid,
            messageType: IBridgeBenchmark.MessageType.MINT
        });
    }

    function _signMessage(IBridgeBenchmark.BridgeMessage memory message)
        internal view returns (bytes memory)
    {
        bytes32 contextHash = hasher.computeContextHash(message);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(relayerPk, contextHash);
        return abi.encodePacked(r, s, v);
    }

    // ─── Success ────────────────────────────────────────────────────────

    function test_ValidateMessage_FullPipeline() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage(1);
        bytes memory sig = _signMessage(msg1);

        (bool valid, bytes32 ctxHash) = vm_mod.validateMessage(msg1, sig);
        assertTrue(valid);
        assertTrue(ctxHash != bytes32(0));
    }

    function test_VerifyMessage_DryRun() public view {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage(1);
        bytes memory sig = _signMessage(msg1);
        assertTrue(vm_mod.verifyMessage(msg1, sig));
    }

    // ─── Failure Cases ──────────────────────────────────────────────────

    function test_RevertOnUnauthorizedRelayer() public {
        uint256 fakePk = 0xDEAD;
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage(1);
        bytes32 contextHash = hasher.computeContextHash(msg1);
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(fakePk, contextHash);
        bytes memory sig = abi.encodePacked(r, s, v);

        vm.expectRevert(
            abi.encodeWithSelector(CANBErrors.UnauthorizedRelayer.selector, vm.addr(fakePk))
        );
        vm_mod.validateMessage(msg1, sig);
    }

    function test_RevertOnReplayNonce() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage(1);
        bytes memory sig = _signMessage(msg1);

        vm_mod.validateMessage(msg1, sig);

        // Second attempt with same nonce should fail
        vm.expectRevert();
        vm_mod.validateMessage(msg1, sig);
    }

    function test_RevertOnWrongChainDestination() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage(1);
        msg1.destinationChainId = 999; // Wrong chain
        bytes memory sig = _signMessage(msg1);

        vm.expectRevert(
            abi.encodeWithSelector(CANBErrors.ChainIdMismatch.selector, 84532, 999)
        );
        vm_mod.validateMessage(msg1, sig);
    }

    function test_VerifyMessage_ReturnsFalseForUsedNonce() public {
        IBridgeBenchmark.BridgeMessage memory msg1 = _makeMessage(1);
        bytes memory sig = _signMessage(msg1);

        vm_mod.validateMessage(msg1, sig);
        assertFalse(vm_mod.verifyMessage(msg1, sig));
    }
}

```
