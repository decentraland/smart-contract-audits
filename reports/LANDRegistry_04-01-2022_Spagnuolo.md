# LAND Registry Audit

## 0. Assumptions
- `proxyOwner` is a trusted party
- `landBalance` is a trusted party
- `estateRegistry` is a trusted party
- `authorizedDeploy` are trusted parties  

## Doubts
- No owner?
- Quien es el proxy owner
- Quienes son deployers?
- Ping by proxy owner?
- Check operators vs managers: jerarquia owner - allowance - manager - operator

## 1. Critical

None  

## 2. High

None 

## 3. Medium

### 3.1. Land balance token accounting can be broken by proxy owner

Even though this method is protected by the `onlyProxyOwner` modifier, it's weird it can be called multiple times.
The proxy owner can re-set the `landBalance` token at any time, it doesn't even require an upgrade to do that.
Doing that could break the accounting logic if there were some tokens already assigned.

Consider either checkpointing the registered balances per token, or having a way to allow users migrating from one token
to another. Or, if it's actually not possible to re-set the `landBalance` token, state that in `setLandBalanceToken`.

Note: Additionally, it is also clear the `LANDRegistry` must be the controller of the MiniMe `landBalance` token.
You could probably make a small check in `setLandBalanceToken` to be sure about that. Otherwise, transfers will be
bricked until you grant that permission. 

### 3.2. Mint/burn ops of land balance token are not validated

Every time a token is transferred in `LANDRegistry`, `destroyTokens` and `generateTokens` are called in `landBalance` 
for the sender and the receiver respectively. Both methods return a boolean in order to tell whether the tokens have
been generated/destroyed properly. This value is not checked at all.

Consider adding a `require` statement to make sure the transfer occurred as expected.

### 3.3. Land balance could be manipulated if transfers are allowed

`LANDRegistry` exposes a function called `registerBalance` that allows minting `landBalance` based on the 
`LANDRegistry`'s balance of the sender. Since `landBalance` is a MiniMeToken (ERC20), the user could transfer these
tokens to another address. 

Then, there is a second function called `unregisterBalance` that allows users to burn their `landBalance`. The problem
is that after doing this, users are allow to call `registerBalance` again, minting new tokens.

The Decentraland team, confirmed `landBalance` transfers are blocked, however please be aware that in case these are
turned on, balances can be manipulated.

## 4. Low

### 4.1. `OwnerUpdate` event is not emitted in `LANDRegistry`
LandRegistry's `OwnerUpdate` event isn't emitted during construction nor when transferring owner.

### 4.2. `OwnerUpdate` event is not emitted in `Proxy`
Proxy's `OwnerUpdate` event isn't emitted during construction nor when transferring owner.

### 4.3. Inconsistent query methods behavior
In `ERC721Basic` you will find lots of query methods, some of them fail when querying the zero address, and some
others don't. For example, `isApprovedForAll` does not fail when querying with a zeroed `operator`. However, 
`isAuthorized` does revert if `operator` is the zero address.

I suggest following one single pattern for all the query methods, either strict or flexible enough.

### 4.4. Fully allowed operators cannot set themselves as `udpateManager`
`setUpdateManager` in `LANDRegistry` requires that `operator` cannot be equal to `msg.sender`, banning fully allowed 
operators to set themselves as update managers, although they can set other accounts.

Simply change that `require ` statement to simply check that `owner` is not `msg.sender` instead.  

### 4.5. `UpdateOperator` not emitted on `transfers`

`LANDRegistry` has an internal method called `_doTransferFrom` that is used for all transfers. This method cleans
the update operator of the asset being transferred, but no `UpdateOperator` event is emitted.
  
I recommend extracting the update operator logic to an internal method and reusing it from `_doTransferFrom`. 

### 4.6. Land owner contracts are not necessarily compliant
`_tokenMetadata` in `LANDRegistry` calls `supportsInterface` to the owner of the asset being queried if it is actually
a contract. However, the owner contract may not support ERC165. Note that transfers are fully allowed to any contract 
address if it implements `IERC721Receiver` (except for some edge cases with the `estateRegistry`). As you can see, 
`IERC721Receiver` is not compliant with ERC165. 

Consider either forcing owner contracts to implement ERC165 or adding some sort of try-catch logic.

## 5. Notes

### 5.1. ERC721 compliance
Based on the specification detailed in [ethereum.org](https://eips.ethereum.org/EIPS/eip-721#specification)
there a few discrepancies mentioned below. I know the implementation was completed before the standard was
considered finished. However, I think it's important to report them:
 
- `ownerOf` must throw when querying NFTs assigned to zero address
- `balanceOf` must throw for queries about the zero address

### 5.2. Shadowing functions
`LANDRegistry` is shadowing some important functions inherited from its upper contracts. 
Doing this makes hard to follow what's exactly going on. For instance, both `transferFrom` and `_isContract` 
implementations of `ERC721Base` are being completely ignored by `LANDRegistry`.

Another example is the implementation of `tokenMetadata`, it clearly adds some custom logic depending on whether the 
owner is a contract or not, and ends up ignoring completely the implementation provided by the upper contract 
`ERC721Metadata`. I'd expect to call `super.tokenMetadata(...)` at some point instead.  

I recommend avoiding default implementations that are not being used at all, either drop them or make sure to 
explicitly call the upper contracts at some point. Otherwise, if there is a strong reason to avoid doing it, leave
a comment to improve its readability. 

### 5.3. Shadowing variables
Similarly to what occurs with shadowed functions, there is shadowing of some storage variables in the `LANDRegistry`
contract. One of the most critical ones is the `owner` storage variable defined in `OwnableStorage` which is being 
shadowed by a function parameter in different occasions.

I recommend avoiding this so it becomes obvious what variables you're actually referring to in each context.  

### 5.4. Duplicated modifiers
`LANDRegistry` defines a modifier called `onlyOwnerOf` that it's not being used. In fact, it could actually use the 
one inherited form `ERC721Basic` called `onlyHolder`.

### 5.5. Inconsistent documentation

#### 5.5.1. `AssetRegistryStorage` - `_approval`

The inline documentation for this storage variable is a bit vague, it simply says "Approval array". It actually tells
what address has been approved for each asset indexed by ID. 

#### 5.5.2. `ERC721Basic` - `ownerOf`

The inline documentation says it returns `uint256 the assetId`, when it actually returns the owner of the queried asset.

#### 5.5.3. `ERC721Basic` - `getApproved`

The inline documentation says this function returns "bool true if the asset has been approved by the holder". It 
actually returns the address that has been approved for an asset.

#### 5.5.4. `ERC721Basic` - `isAuthorized` 

The inline documentation states that it returns "true if the asset has been approved by the holder" which is not 
consistent with what the implementation is actually doing. Based on how `approve` works, token owners cannot approve 
themselves, but `isAuthorized` returns true when it's being queried with the owner.

I think the implementation is correct, please double-check, in that case I'd simply update the inline documentation. 

#### 5.5.5. `ERC721Basic` - `safeTransferFrom`

The inline documentation says it calls "the method `onNFTReceived` on the target address if there's code associated 
with it", but actually `onERC721Received` is being called instead. 

#### 5.5.6. `ERC721Basic` - `_moveToken`

The inline documentation for this function is placed right after it instead of right above it.

### 5.6. Optimizations

#### 5.6.1. Runs

Both LAND Registry implementations were deployed using only 200 runs

#### 5.6.2. `_moveToken`

This functions loads `ownerOf` at least 4 times, this could be cached or at least use `from` instead since it is 
already ensured that these are the same with the `isCurrentOwner` modifier

#### 5.6.3. `onlyUpdateAuthorized`

It checks three times if the sender if the owner of the asset, loading `ownerOf` three times indeed. 

#### 5.6.4. `_clearApproval`

There is no need to check that `holder` is actually the owner of the asset being cleared. Similarly to other internal
methods like `_addAssetTo` or `_removeAssetFrom`, this one could be trusted based on how it's being used.

### 5.7. Correctness

- Unnecessary `return` statements when defining returning variables.
- Flipping function parameters order arbitrary, e.g. `(holder, operator)` and `(operator, holder)` are both used indistinctly.
- Terms `owner`, `holder`, and `user` are being used indistinctly.
- `authorizedDeploy` probably is meant to `authorizedDeployer` instead.
- Reduce repeated code to reduce surface attack, e.g.: `assignMultipleParcels` and `assignNewParcel`.
- `IERC721Enumerable` has 3 methods commented out with `TODO` notes.
- Missing revert reasons
