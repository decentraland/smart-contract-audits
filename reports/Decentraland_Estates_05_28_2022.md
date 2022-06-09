# Estate Registry Audit

Date: 05-28-2022
Author: Facundo Spagnuolo<facundo.spagnuolo@gmail.com>
Commit: 15c77b558f79521df155d4afe23c8449ecb96bd1

## 0. Assumptions
- The smart contracts audited in this process are `EstateRegistry`, `EstateStorage`, and `IEstateRegistry`.

## 1. Critical

None

## 2. High

None

## 3. Medium

None

## 4. Low

#### 4.1. MiniMe `destroyTokens` and `generateTokens` returned values are not always checked

The `EstateRegistry` contract implements a function called `_updateEstateLandBalance` where `destroyTokens` and 
`generateTokens` are being called for `estateLandBalance` in order to burn and mint tokens respectively. However, 
the return value is not being checked. Consider adding a require statement to check the return result, similar to how
it's being done in `registerBalance` and `unregisterBalance`.

#### 4.2. `SetLANDRegistry` is not emitted during initialization

The `EstateRegistry` contract defines a function called `initialize` where `landRegistry` is being set. There is also
a separate setter function called `setLANDRegistry` that allows the contract owner to update this reference. The 
difference is that `setLANDRegistry` emits a `SetLANDRegistry` event which is completely ignored in the `initialize` 
function. Consider either emitting this event in the `initialize` function as well or re-using `setLANDRegistry` from it.

#### 4.3. `verifyFingerprint` can returns a false positive result

The `EstateRegistry` contract defines a function called `verifyFingerprint` that receives two parameters. One is a 
estate ID and the other is a bytes array which is the fingerprint to be validated. The problem is that `EstateRegistry`
uses a `bytes32` for the fingerprints, and this function simply casts the bytes array into a `bytes32` variable. The 
problem is that many different bytes array can be considered as valid fingerprints as long as they have the define 
32 initial bytes. Consider either reverting or returning false if the given fingerprint is longer than 32 bytes.

## 5. Notes

#### 5.1. Optimize storage layout

`EstateStorage` implements many mappings that are indexed per estate ID. For example, `estateLandIds`, `estateData`, 
`estateLandIndex`, and `updateOperator`. These can be simplified in a single struct in order to avoid recomputing 
the storage pointer every time it is needed to access many pieces of information related to an estate. For example 
`_pushLandId` could be optimized a lot since it access to all these storage variables separately.  

#### 5.2. Shadowing variables

`EstateRegistry` is shadowing some storage variables. One of the most critical ones is the `owner` storage variable 
defined in `OwnableStorage` which is being shadowed by memory variables in many cases, for example in 
[`_pushLandId`](https://github.com/decentraland/land/blob/15c77b558f79521df155d4afe23c8449ecb96bd1/contracts/estate/EstateRegistry.sol#L473) 
or [`canSetUpdateOperator`](https://github.com/decentraland/land/blob/15c77b558f79521df155d4afe23c8449ecb96bd1/contracts/estate/EstateRegistry.sol#L43).

I recommend avoiding this, so it becomes obvious what variables you're actually referring to in each context.  

#### 5.3. There is no clear naming convention for variables

Function parameters sometimes are prefixed with `_` and sometimes not.
Consider adopting a single convention that stays consistent along the entire codebase.

#### 5.4. `_tokenId` is ambiguous

In some critical functions like `transferFrom` or `onERC721Received` `_tokenId` is used to refer an estate ID and a 
land ID respectively. Consider renaming these usages for `estateId` and `landId` respectively, similar to how it is 
being used in the rest of the codebase.

#### 5.5. `Update` event is ambiguous

The `Update` event is being triggered when estate's metadata is updated. Compared to other update-related events like
`UpdateOperator` or `UpdateManager` the name picked for this event is ambiguous. Consider renaming to something more
accurate like `UpdateMetadata`.

#### 5.6. Complete inline documentation

Many functions of the `EstateRegistry` contracts have their corresponding inline documentation. However, some critical
functions like `_isUpdateAuthorized` or `_isLandUpdateAuthorized` don't. Consider adding what you expect out of these
functions to give another perspective to the reader about their functionality.

#### 5.7. Optimizations
The following findings can be addressed in order to optimize gas consumption:
- `_transferLand` loads `landIndex[landId]` from storage twice
- `_setEstateLandBalanceToken` can be inlined into `setEstateLandBalanceToken`
- `_setEstateLandBalanceToken` is unnecessarily checking for non-zero address when it's only being used with a hardcoded one
- `registerBalance` burns the current balance and mints again the new desired amount, it could either burn or mint the difference 
