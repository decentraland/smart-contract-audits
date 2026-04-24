# Smart Contracts Well Knonw Issues

## Estate
 - The `getFingerprint`, and some function to get assets from the users can run out of gas for Estates bigger than 4000 LANDs and users with a lot of assets. They can use other functions to workaround this.

## Collections
  - Missing the ERC165 interface registration.
  - Possible front-running of MANA being paid when creating a collection. It does not generate an issue because the MANA goes to the DAO and can be returned.

## Rentals
- The Rentals contract is not compliant with the LANDRegistry `tokenMetadata` interface (`bytes4(keccak256("getMetadata(uint256)"))`).
- The Rentals contract accepts rents for Estates with size 0. It is a NFT rentable contract, no further or specific checks worth it.

## Rewards
- Rewards Smart Contract can't support two signatures for the same item at the same time. Also, the whole tx will fail if one item has been already claimed.

## Token Wrapper and Token Manager

- TokenWrapper and TokenManager Aragon's apps allow anyone with balance to call any not blacklisted smart contract. This makes things like the token escape hatch useless. Anyone can move tokens only if they are sent by mistake.

# Off-Chain Marketplace Smart Contract

- Smart-contract wallets (ERC-1271 signers, e.g. Safe, Coinbase Smart Wallet, account-abstraction accounts) are not supported. Listings signed by these wallets are rejected by the marketplace backend and will not be indexed. Users must sign with an externally-owned account (EOA).
- Because of the above, meta-transactions are also not supported for smart-contract wallets.
- Cancellation (`cancelSignature`) and the per-signature usage counter (the `uses` limit in `Checks`) are keyed by the hash of the raw signature bytes, in both the Marketplace and the CouponManager. This is safe for EOA signatures because ECDSA enforces a canonical 65-byte encoding. Smart-contract wallets, however, validate signatures through their own fallback handlers, which commonly accept multiple valid byte representations of the same signed authorization (for example, a signature and the same signature with arbitrary trailing bytes appended both pass validation). Under those conditions, the raw-bytes accounting would allow alternate encodings to sidestep a cancellation or a single-use limit on a trade or coupon. This is the underlying reason smart-contract signers are unsupported. A contract-level fix keying both by the EIP-712 digest is in development for the Marketplace and the CouponManager and will ship as a redeploy.

# Signature Validation (All Contracts)

Across all Decentraland contracts that validate off-chain signatures (e.g. Marketplace trades, CouponManager coupons, and any other signature-gated flow), only externally-owned accounts (EOAs) are supported and encouraged.

Smart-contract wallets (ERC-1271 signers such as Safe, Coinbase Smart Wallet, account-abstraction accounts) are not supported. The reason is that these wallets validate signatures through their own fallback handlers, which commonly accept multiple valid byte representations of the same signed authorization (for example, a signature and the same signature with arbitrary trailing bytes appended both pass validation). Any accounting that ties signature identity to the raw signature bytes can therefore be sidestepped by submitting an alternate encoding.


