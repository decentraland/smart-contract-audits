# Common Contracts

Date: 2022-06-22 <br>
Author: Facundo Spagnuolo<facundo.spagnuolo@gmail.com> <br>
Commit: [`aa419a79cd2e2dfe5bc3d838eecceed1dfef0387`](https://github.com/decentraland/common-contracts/commit/aa419a79cd2e2dfe5bc3d838eecceed1dfef0387) <br>
Amendment: [`93a3471df9e53b11bf2f36fc8139f8ae8f99b34e`](https://github.com/decentraland/common-contracts/commit/93a3471df9e53b11bf2f36fc8139f8ae8f99b34e) <br>

## 0. Assumptions

None

## 1. Critical

Nonce

## 2. High

#### 2.1. Ownable initialization is not exposed

The `NonceVerifiable` contract inherits from `OwnableUpgradeable` which offers two initialization methods called
`__Ownable_init` and `__Ownable_init_unchained`. These methods should be exposed to denote how `NonceVeriable` should
be initialized, otherwise the user will have to know what contracts in the inheritance chain have to be initialized and
how which tends to be error-prone.

**Update**: `NonceVerifiable` has been split into three separate contracts: `AssetNonceVerifiable`, 
`ContractNonceVerifiable`, and `SignerNonceVerifiable`. `ContractNonceVerifiable` is the only one inheriting from
`OwnableUpgradeable`. All these are following the expected initialization logic proposed by OpenZeppelin upgradeable
contracts making it robust and easy to follow. Fixed in [`93a3471df9e53b11bf2f36fc8139f8ae8f99b34e`](https://github.com/decentraland/common-contracts/commit/93a3471df9e53b11bf2f36fc8139f8ae8f99b34e).

#### 2.2. EIP712 initialization is not exposed 

The `NativeMetaTransaction` contract inherits from `EIP712Upgradeable` which offers two initialization methods called 
`__EIP712_init` and `__EIP712_init_unchained`. These methods should be exposed to denote how `NativeMetaTransaction`
should be initialized, otherwise the user will have to know what contracts in the inheritance chain have to be 
initialized and how which tends to be error-prone.

**Update**: `NativeMetaTransaction` is now exposing the expected initialization methods following the convention 
proposed by OpenZeppelin upgradeable contracts. Fixed in [`644be93568f057066fb96b75a1b36d915fa090c1`](https://github.com/decentraland/common-contracts/commit/644be93568f057066fb96b75a1b36d915fa090c1) 
and [`ef1ea53061592a8361a0c3ead99e7b3845650ed6`](https://github.com/decentraland/common-contracts/commit/ef1ea53061592a8361a0c3ead99e7b3845650ed6). 

## 3. Medium

None

## 4. Low

#### 4.1. Revert data is unnecessarily filtered when its length is lower than 68 bytes

`NativeMetaTransaction`'s `executeMetaTransaction` function implements a custom try-catch logic in order to either 
bubble-up detected reverts or simply return any returndata. However, note that the logic that bubbles-up errors
executes a different logic depending on the returndata length. If this one is greater than 68 bytes, then it redirects
its content, otherwise it reverts without any data. There is actually no need to make this distinction since
`NativeMetaTransaction` does not actually have full context about what's encoded in the returndata. It can simply 
redirect it nevertheless the returndata length.

**Update**: Fixed in [`0128e2dff7d6035e1d8e190c31ef1f76a7427d9e`](https://github.com/decentraland/common-contracts/commit/0128e2dff7d6035e1d8e190c31ef1f76a7427d9e).

#### 4.2. `NonceVerifiable`'s nonces concept could be misunderstood

The `NonceVerifiable` contract uses the term nonce to describe a functionality that does not work exactly to what we
typically expect out of a nonce. The right example can be found in the codebase itself. If you look at how 
`NativeMetaTransaction` uses this concept, you will note that nonces are incremented every time these are verified. 
This means that the contract can guarantee that each nonce cannot be used twice once verified. Note that this is not 
exactly how nonces work in `NonceVerifiable`, these can be verified multiple times while the bump methods are completely
detached from this process.

Consider renaming this concept to something more accurate.

**Update**: This concept has been renamed to `IndexVerifiable` in [`7f49a2eed38e76df1d29b8710c217e97041ba6b5`](https://github.com/decentraland/common-contracts/pull/17/commits/7f49a2eed38e76df1d29b8710c217e97041ba6b5).

## 5. Notes

#### 5.1. `MetaTransactionExecuted` should be indexed

The `NativeMetaTransaction` contract defines an event called `MetaTransactionExecuted` that does not index any of its
variables. Consider indexing the ones that makes sense based on its expected usage.

**Update**: Fixed in [`927b1b622327d5b94a79b73a112cda804da71896`](https://github.com/decentraland/common-contracts/commit/927b1b622327d5b94a79b73a112cda804da71896).

#### 5.2. `SignerNonceUpdated` should be indexed

The `NonceVerifiable` contract defines an event called `SignerNonceUpdated` that does not index any of its
variables. Consider indexing the ones that makes sense based on its expected usage.

**Update**: Fixed in [`927b1b622327d5b94a79b73a112cda804da71896`](https://github.com/decentraland/common-contracts/commit/927b1b622327d5b94a79b73a112cda804da71896).

#### 5.3. `AssetNonceUpdated` should be indexed

The `NonceVerifiable` contract defines an event called `AssetNonceUpdated` that does not index any of its
variables. Consider indexing the ones that makes sense based on its expected usage.

**Update**: Fixed in [`927b1b622327d5b94a79b73a112cda804da71896`](https://github.com/decentraland/common-contracts/commit/927b1b622327d5b94a79b73a112cda804da71896).

#### 5.4. `NonceVerifiable` events unnecessarily tracks previous values

It's an old practice to track previous values in events unless it is extremely necessary. Usually the trade-off between
the gas consumption vs the costs of accessing these previous values in any off-chain consumer process does not pay off.
Consider tracking the current or new values, or simply denoting these have been incremented if that's always the case. 

**Update**: Fixed in [`927b1b622327d5b94a79b73a112cda804da71896`](https://github.com/decentraland/common-contracts/commit/927b1b622327d5b94a79b73a112cda804da71896).

#### 5.5. `_bumpSignerNonce` and `bumpAssetNonce` could be optimized

The way these methods are written in a single line is causing at least two storage load operations, when it could simply 
use one. Consider splitting these implementations in order to keep track the current nonce value in memory to reduce 
gas costs.

**Update**: It has been proved the way it is implemented it's actually cheaper.

#### 5.6. `_getMsgSender` could be optimized

Avoid copying the entire calldata to memory to reduce gas costs. Opcodes like `calldataload` and `calldatasize`
can be used instead to implement the same functionality.

**Update**: Fixed in [`5eec9fa5a728a331015859cf4bd2297e97daacab`](https://github.com/decentraland/common-contracts/commit/5eec9fa5a728a331015859cf4bd2297e97daacab).

#### 5.7. `NonceVerifiable`'s storage variables have `public` visibility

Simply double check what's the intention here since inheriting contracts can make a wrong usage of what is expected
from this contract. Since inheriting contracts can manipulate these variables easily, it may make sense making them
`private` while providing an external method to query them. This is the case of `contractNonce`, `signerNonce`, and
`assetNonce`.

**Update**: Fixed in [`c7c69f36c559b23025113f130b0e836407f4dc12`](https://github.com/decentraland/common-contracts/commit/c7c69f36c559b23025113f130b0e836407f4dc12).

#### 5.8. `NativeMetaTransaction`'s `nonces` have `public` visibility

Simply double check what's the intention here since inheriting contracts can make a wrong usage of what is expected
from this contract. Since inheriting contracts can manipulate this variable easily, it may make sense making it 
`private` while providing an external method to query it.

**Update**: Fixed in [`c7c69f36c559b23025113f130b0e836407f4dc12`](https://github.com/decentraland/common-contracts/commit/c7c69f36c559b23025113f130b0e836407f4dc12).

#### 5.9. Typos  

- NonceVerifiable.sol#L71: CONTRACT_NONCE_MISSMATCH -> CONTRACT_NONCE_MISMATCH
- NonceVerifiable.sol#L76: SIGNER_NONCE_MISSMATCH -> SIGNER_NONCE_MISMATCH
- NonceVerifiable.sol#L86: ASSET_NONCE_MISSMATCH -> ASSET_NONCE_MISMATCH
- NativeMetaTransaction.sol#L75: implementator -> implementer or implementor

**Update**: Fixed in [`0ef867a35155d4cd262df0a21aed9e1a4720f7f8`](https://github.com/decentraland/common-contracts/commit/0ef867a35155d4cd262df0a21aed9e1a4720f7f8).

#### 5.10. No linter  

It is highly recommended to set up a linter to make sure contracts follow the same structure and styling across the 
project. Consider setting up solhint which the linter widely adopted by the community.

**Update**: Fixed in [`88be6334ec9c1d676116ebb854e8f6af872968e6`](https://github.com/decentraland/common-contracts/commit/88be6334ec9c1d676116ebb854e8f6af872968e6).

#### 5.11. No default `test` script  

The `package.json` does not specify any `test` script, it actually errors with `"Error: no test specified"`. Consider
denoting and making clear how tests should be run for this project.

**Update**: Fixed in [`333e317af267e838b244f05b9992b61bf08a98fa`](https://github.com/decentraland/common-contracts/commit/333e317af267e838b244f05b9992b61bf08a98fa).

#### 5.12. Disable initializers for implementation contracts

OpenZeppelin v4.7.0 introduced a new internal function in `Initializable` called [`_disableInitializers`](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3450).
This function is meant to be used for upgradeable contracts in order to disable initializer functions in the implementation contracts.
Consider upgrading the OpenZeppelin library and make use of it.

**Update**: Not addressed since these contracts are not meant to be used as standalone contracts.

#### 5.13. ECDSA signature malleability

A [`high-severity issue`](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h)
has been reported by the OpenZeppelin team in their ECDSA library which is used from the `NativeMetaTransaction` contract.

Consider upgrading the OpenZeppelin library to v4.7.3 or later.

**Update**: Fixed in [`5b4efc20f23861afbce8725946f979ef3a8d58a7`](https://github.com/decentraland/common-contracts/pull/17/commits/5b4efc20f23861afbce8725946f979ef3a8d58a7).