# Rental Contracts

Date: 2022-06-22 <br>
Author: Facundo Spagnuolo<facundo.spagnuolo@gmail.com> <br>
Commit: [`b95d844db6cec390aa82271b6d0fbea067e956cc`](https://github.com/decentraland/rentals-contract/tree/b95d844db6cec390aa82271b6d0fbea067e956cc) <br>
Amendment: [`fd0039fe5e44a06cd8a8a2a5760d0879bf674b5b`](https://github.com/decentraland/rentals-contract/tree/fd0039fe5e44a06cd8a8a2a5760d0879bf674b5b) <br>

## 0. Assumptions

#### 0.1. ERC721 assets manage operators in a safe way

If the implementation of each ERC721 handles operators correctly, there shouldn't be any side attacks to this process.
This means, there should only be one operator per token, and the operator is allowed to update non-critical information.
For example, the operator shouldn't be able to increase allowances, or modify update managers, etc. Moreover, operators
should be unset every time tokens are transferred.

## 1. Critical

None

## 2. High

#### 2.1. Paying token can be changed at any point in time

The `Rental`'s owner can change the token used to pay for transactions at any point in time. Some unexpected scenarios
could occur if this happens. For example, prices set in a rent are completely detached from the token, this could make 
users pay an unexpected value for a rent.  Additionally, token transfers are not being checked.

Consider either moving the token address to the rent data or making it immutable. 

**Update**: Fixed in [`9fb5735fd173953a75cdc3194ba418ff555c84a6`](https://github.com/decentraland/rentals-contract/commit/9fb5735fd173953a75cdc3194ba418ff555c84a).

## 3. Medium

None

## 4. Low

#### 4.1. `OwnableUpgradeable` init function is not called

This issue is carried from the `NonceVerifiable` contract. This contract does not expose an initialization method, 
leaving up to the user the decision of how the contracts in the inheritance chain should be initialized. In this case,
note that `Rentals` inherits from `OwnableUpgradeable` and `EIP712Upgradeable` through `NonceVerifiable` and 
`NativeMetaTransaction` respectively, and it does call `__EIP712_init` but not `__Ownable_init`. Consider making sure
`OwnableUpgradeable` is initialized correctly. 

This issue is marked as low severity since the implementation actually circumvents what the initialization method does
by calling `_transferOwnership` manually. However, it denotes a wrong understanding of how initialization functions of 
upgradeable contracts should be used.

**Update**: `NonceVerifiable` has been split into three separate contracts: `AssetNonceVerifiable`,
`ContractNonceVerifiable`, and `SignerNonceVerifiable`. `ContractNonceVerifiable` is the only one inheriting from
`OwnableUpgradeable` which is now calling `__Ownable_init`. `Rentals` now inherits these three new contracts, and it 
explicitly calls `ContractNonceVerifiable`'s initialize function from it. Fixed in [`93a3471df9e53b11bf2f36fc8139f8ae8f99b34e`](https://github.com/decentraland/common-contracts/commit/93a3471df9e53b11bf2f36fc8139f8ae8f99b34e).

#### 4.2. `Rentals` does not call all the expected initialize functions

`Rentals` inherits from many upgradeable contracts: `ContractNonceVerifiable`, `SignerNonceVerifiable`, 
`AssetNonceVerifiable`, `NativeMetaTransaction`, `ReentrancyGuardUpgradeable`, `PausableUpgradeable`. It is expected to 
call all the initialize functions corresponding to each contract, even thought some of these could be empty functions, 
it's a good practice in terms of completeness and to make sure the reader can treat upper implementations as a black box. 

Consider calling `__AssetNonceVerifiable_init` and `__SignerNonceVerifiable_init`.

**Update**: Not addressed.

#### 4.3. Unnecessary manual static-call to compute `IERC721Rentable#ownerOf`

`Rentals` performs a manual `staticcall` to perform `IERC721Rentable#ownerOf` calls when the interface itself
declares this method as view. This is what actually the compiler does behind the scenes, there is no need to manually 
perform a `staticcall`. Consider simply calling `ownerOf`.

**Update**: Fixed in [`7985658928a119a97a4d5c57e71d9b3618f38b09`](https://github.com/decentraland/rentals-contract/pull/52/commits/7985658928a119a97a4d5c57e71d9b3618f38b09).

#### 4.4. Unnecessary manual static-call to compute `IERC721Rentable#supportsInterface`

`Rentals` performs a manual `staticcall` to perform `IERC721Rentable#supportsInterface` calls when the interface itself
declares this method as view. This is what actually the compiler does behind the scenes, there is no need to manually 
perform a `staticcall`. Consider simply calling `supportsInterface`.

**Update**: Fixed in [`7985658928a119a97a4d5c57e71d9b3618f38b09`](https://github.com/decentraland/rentals-contract/pull/52/commits/7985658928a119a97a4d5c57e71d9b3618f38b09).

#### 4.5. Unnecessary manual static-call to compute `IERC721Rentable#verifyFingerprint`

`Rentals` performs a manual `staticcall` to perform `IERC721Rentable#verifyFingerprint` calls when the interface itself
declares this method as view. This is what actually the compiler does behind the scenes, there is no need to manually 
perform a `staticcall`. Consider simply calling `verifyFingerprint`.

**Update**: Fixed in [`7985658928a119a97a4d5c57e71d9b3618f38b09`](https://github.com/decentraland/rentals-contract/pull/52/commits/7985658928a119a97a4d5c57e71d9b3618f38b09).

## 5. Notes

#### 5.1. `acceptListing` validations can be checked before validating signatures

This method performs a bunch of validations that doesn't necessarily belong to a wrong usage of the tenant. 
For example, all the validations corresponding to how the `Lending` object is constructed are responsibility of the 
lessor, e.g.: min and max days ranges, array lengths, etc. Therefore, a fail-fast approach can be applied here, these 
validations can be checked before checking things like signatures which may consume much more gas usage.

**Update**: Fixed in [`d5944bcc2e87d95ec588a0fb50b9e7a696fb0568`](https://github.com/decentraland/rentals-contract/commit/d5944bcc2e87d95ec588a0fb50b9e7a696fb0568).

#### 5.2. `acceptListing` can be optimized working with `Listing` in memory

Note that this function relies on the entire `Listing` object. Therefore, it makes sense to copy it once to memory 
instead of performing a bunch of copy instructions to calldata everytime the function needs to access to an object
property.

**Update**: Fixed in [`bfe7522314ae24d27aa927d74dee4718b90e55a7`](https://github.com/decentraland/rentals-contract/pull/52/commits/bfe7522314ae24d27aa927d74dee4718b90e55a7).

#### 5.3. `acceptOffer` can be optimized working with `Offer` in memory

Note that this function relies on the entire `Offer` object. Therefore, it makes sense to copy it once to memory
instead of performing a bunch of copy instructions to calldata everytime the function needs to access to an object
property.

**Update**: Fixed in [`bfe7522314ae24d27aa927d74dee4718b90e55a7`](https://github.com/decentraland/rentals-contract/pull/52/commits/bfe7522314ae24d27aa927d74dee4718b90e55a7).

#### 5.4. `Rentals` events are not indexed

None of the events used in the `Rentals` contract are indexed. Consider indexing their content in consonance with its
expected usage.

**Update**: Fixed in [`75b9f7f175070d4dbcd2daf84249c3dc568e0fb3`](https://github.com/decentraland/rentals-contract/commit/75b9f7f175070d4dbcd2daf84249c3dc568e0fb3).

#### 5.5. `_handleTokenTransfers` can be optimized

In this function the storage variables `token` is loaded twice. Consider keeping the memory reference making sure it
is loaded from storage only once.

**Update**: Fixed in [`2e3be0ab3621bdcac4f94406d8112b902e003fa2`](https://github.com/decentraland/rentals-contract/commit/2e3be0ab3621bdcac4f94406d8112b902e003fa2).

#### 5.6. `Listing` and `Offer` could use separate variables for contract, signer, and asset nonces

These two struct objects define a property called `nonces` that is an array with length 3 that actually refer to the
contract nonce, signer nonce, and asset nonce respectively. Instead of making sure the developer always indexes this
property in the right way, three different properties could be defined instead. In fact, it is more expensive to work
with an array of objects rather than working with the objects individually if you know the length beforehand.

**Update**: Not addressed.

#### 5.7. Extract magic numbers to constants

There are a few magic numbers in the contract that can be extracted to internal constant variables. Even though, some
of these have a comment next to it, it improves its readability, and it is less error-prone to use variables instead:
- Rentals#L342: 1_000_000
- Rentals#L382: 86400
- Rentals#L414: 1_000_000

**Update**: Fixed in [`28e9bf786fb95547cef45d9c102a352cc1acedab`](https://github.com/decentraland/rentals-contract/commit/28e9bf786fb95547cef45d9c102a352cc1acedab)
 and [`11e220b6d410222d0df348c401a7b25b9100fbfe`](https://github.com/decentraland/rentals-contract/commit/11e220b6d410222d0df348c401a7b25b9100fbfe).

#### 5.8. Typos

- Rentals#L107: transfered => transferred
- Rentals#L183: MISSMATCH => MISMATCH
- Rentals#L196: MISSMATCH => MISMATCH
- Rentals#L197: MISSMATCH => MISMATCH
- Rentals#L238: MISSMATCH => MISMATCH
- Rentals#L357: Fingerpint => Fingerprint
- Rentals#L416: transfered => transferred

**Update**: Fixed in [`a776d039d9ab8611338db8f5734fd7f4163e0bdb`](https://github.com/decentraland/rentals-contract/commit/a776d039d9ab8611338db8f5734fd7f4163e0bdb).

#### 5.9. No linter

It is highly recommended to set up a linter to make sure contracts follow the same structure and styling across the
project. Consider setting up solhint which the linter widely adopted by the community.

**Update**: Fixed in [`7ff008a4a5f2756e5d6893600c70e21cc33e53a3`](https://github.com/decentraland/rentals-contract/commit/7ff008a4a5f2756e5d6893600c70e21cc33e53a3).

#### 5.10. No default `test` script

The `package.json` does not specify any `test` script, it actually errors with `"Error: no test specified"`. Consider
denoting and making clear how tests should be run for this project.

**Update**: Fixed in [`e1755f1b0d91137817fd00bff8959c46cf7d78fc`](https://github.com/decentraland/rentals-contract/commit/e1755f1b0d91137817fd00bff8959c46cf7d78fc).

#### 5.11. Disable initializers for implementation contracts

OpenZeppelin v4.7.0 introduced a new internal function in `Initializable` called [`_disableInitializers`](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3450).
This function is meant to be used for upgradeable contracts in order to disable initializer functions in the implementation contracts. 
Consider upgrading the OpenZeppelin library and make use of it.

**Update**: Fixed in [`64aab0c636606a61afc6eaf0e7391e51ff4934d1`](https://github.com/decentraland/rentals-contract/pull/52/commits/64aab0c636606a61afc6eaf0e7391e51ff4934d1).

#### 5.12. ECDSA signature malleability

A [`high-severity issue`](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h)
has been reported by the OpenZeppelin team in their ECDSA library which is used from the `Rentals` contract.

Consider upgrading the OpenZeppelin library to v4.7.3 or later.

**Update**: Fixed in [`d4fd7d1e5503569ba8bfcc224b873e1fa2547785`](https://github.com/decentraland/rentals-contract/pull/52/commits/d4fd7d1e5503569ba8bfcc224b873e1fa2547785).