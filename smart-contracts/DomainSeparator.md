# DomainSeparator.sol

## Deskripsi dan Kegunaan
`DomainSeparator.sol` mengimplementasikan standar pengikatan *domain* **EIP-712** untuk arsitektur CANB. Kontrak ini secara aman mengikat setiap pesan *cross-chain* pada satu rantai dan satu alamat kontrak tujuan tertentu.

Pemisah *domain* (*domain separator*) ini dihasilkan satu kali di konstruktor dan disimpan secara permanen (*immutable*). Pengikatan ketat ini memastikan bahwa meskipun penyerang mencegat (*intercept*) tanda tangan kriptografis di satu *chain*, mereka tidak bisa menggunakannya kembali (*replay*) di *chain* lain atau pada *smart contract* yang berbeda. Ini adalah pertahanan mendasar terhadap serangan replay lintas rantai.

## Source Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/**
 * @title DomainSeparator
 * @notice EIP‑712 domain binding for the CANB framework.
 *
 * @dev Computes and stores an **immutable** `DOMAIN_SEPARATOR` at construction
 *      time. The separator encodes the contract name, version, chain ID, and
 *      verifying contract address so that signatures are irrevocably bound to
 *      a specific deployment. This prevents cross‑chain and cross‑contract
 *      replay attacks.
 *
 *      The domain separator is computed once in the constructor and cached in
 *      an immutable variable for gas efficiency.
 *
 *      Reference: EIP-712 (https://eips.ethereum.org/EIPS/eip-712)
 */
contract DomainSeparator {
    // ─── Constants ──────────────────────────────────────────────────────

    /// @dev EIP‑712 domain type hash.
    bytes32 public constant DOMAIN_TYPEHASH =
        keccak256(
            "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
        );

    /// @dev Protocol name included in the domain separator.
    string public constant DOMAIN_NAME = "CANB";

    /// @dev Protocol version included in the domain separator.
    string public constant DOMAIN_VERSION = "1";

    // ─── Immutables ─────────────────────────────────────────────────────

    /// @notice The cached EIP‑712 domain separator.
    bytes32 public immutable DOMAIN_SEPARATOR;

    /// @notice The chain ID captured at deployment time.
    uint256 public immutable DEPLOYMENT_CHAIN_ID;

    // ─── Constructor ────────────────────────────────────────────────────

    constructor() {
        DEPLOYMENT_CHAIN_ID = block.chainid;
        DOMAIN_SEPARATOR = _computeDomainSeparator();
    }

    // ─── Internal ───────────────────────────────────────────────────────

    /// @dev Compute the EIP‑712 domain separator for this contract instance.
    function _computeDomainSeparator() internal view returns (bytes32) {
        return keccak256(
            abi.encode(
                DOMAIN_TYPEHASH,
                keccak256(bytes(DOMAIN_NAME)),
                keccak256(bytes(DOMAIN_VERSION)),
                block.chainid,
                address(this)
            )
        );
    }

    // ─── View ───────────────────────────────────────────────────────────

    /// @notice Returns the cached domain separator.
    /// @dev In a production bridge the separator would be recomputed if
    ///      `block.chainid` changed (hard fork), but for thesis experiments
    ///      on stable testnets the immutable value is sufficient.
    function getDomainSeparator() public view returns (bytes32) {
        return DOMAIN_SEPARATOR;
    }
}
```
