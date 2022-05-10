# Wearable Contracts Audit

Date: 05-05-2022
Commit: `b722ba2185eb00bf54335414e711e68328d70d1b`
Author: Facundo Spagnuolo<facundo.spagnuolo@gmail.com>

# 0. Assumptions

#### 0.1. `ERC721BaseCollectionV2`'s owner is a trusted party

`ERC721BaseCollectionV2` is initialized with an owner account that is meant to be a `Forwarder` instance. However, 
controls the forwarder has full access over critical functions in the collections like adding and editing items, 
marking the collection as completed or approved, setting the URI, among others.

#### 0.2. `ERC721BaseCollectionV2`'s owner has creator's rights

`ERC721BaseCollectionV2` defines a creator address that is set during initialization, these address has access to 
certain critic functions in the collection. However, the owner has full control over the creator too. The owner is 
allowed to set a new creator at any time through `transferCreatorship`.

## 1. Critical

None

## 2. High

#### 2.1. `BeaconProxyFactory` does not enforce a specific beacon implementation

With the new implementation of collections factory introduced by `ERC721CollectionFactoryV3` uses a beacon proxy
factory behind the scenes called `BeaconProxyFactory`. The problem is that the beacon implementation is not enforced 
in the factory itself. This means that it will depend entirely on how the factory is configured at deployment time. 
For example, these could be upgradeable or not.

Consider denoting in the factory itself what's the `IBeacon` implementation that will be used for the factory to make
clear what's the expected beacon behavior. 

## 3. Medium

#### 3.1. `ERC721BaseCollectionV2` is publicly initializable

With the new implementation of collections factory introduced by `ERC721CollectionFactoryV3`, collections are now 
probably upgradeable. A well known issue is that uninitialized implementations could lock upgradeable contracts 
depending on the upgradeability mechanism behind it. See [Open Zeppelin's security advisory](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-5vp3-v4hc-gx76)
for instance. 

Consider initializing `ERC721BaseCollectionV2` from `ERC721CollectionV2`'s constructor.

#### 3.2. `ERC721Initializable` is not ERC165-compliant therefore not ERC721-compliant 

From the [specification](https://eips.ethereum.org/EIPS/eip-721#specification) defined for EIP-721: 

> Every ERC-721 compliant contract must implement the ERC721 and ERC165 interfaces

`ERC721Initializable` defines a function called `_initERC721` that basically registers all the interfaces supported 
by this contract. However, the interface ID of the ERC165 `0x01ffc9a7` is not being registered. The problem is that 
inheriting contracts that are meant to be used through proxies are not compatible with the ERC165 standard, therefore
not fully compatible with the ERC721 standard too. This is the case of `ERC721BaseCollectionV2` for example.

Consider registering ERC165's interface `0x01ffc9a7` in `initERC721`.

#### 3.3. `CollectionManager`'s `manageCollection` is susceptible to selector clashing 

`CollectionManager` is a contract that allows Decentraland's committee to manage arbitrary ERC721 collections. 
It basically provides an interface that allows the committee to trigger calls to ERC721 collections making it possible
to manage many collections from a single place. These calls have to be whitelisted first by the same committee as a 
double-checking process. In order to tell the list of calls that are allowed to be triggered to the collections, 
function selectors are being used. It is a well-known issue it is simple to find selectors clashing very quickly.

#### 3.4. Manager's allowance can be manipulated with transactions ordering in `ERC721BaseCollectionV2` 

`ERC721BaseCollectionV2` implements a method called `issueTokens` that allows the collection's creator or allowed 
minters to issue tokens. Allowed minters have an allowance value set by the creator through `setItemsMinters` that 
is decreased every time a new item is minted. This mechanism is susceptible of front-running, in case the collection's
creator wants to change the allowance value, the minter could place a transaction right before it to mint tokens using
his current allowance value, while being able to use the new allowance value right after that.

Consider implementing some sort of increase/decrease allowance mechanism similar to the 
[one proposed for ERC20s](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#). 

#### 3.5. `CollectionManager`'s rarities payment process can be front-run 

The `CollectionManager` contract allows users to create collections. Every time a new collection is created, the user
has to pay for each of the rarities being requested. However, the rarities price could be changed right before a 
collection is created by the `Rarities`'s owner, forcing the user to pay a larger amount to the one expected.

Consider implementing a mechanism similar to the one built in `CollectionStore` where the user tells the price that is 
willing to pay for each rarity.

## 4. Low

#### 4.1. `ContextMixin` does not check calldata length

`ContextMixin` decodes an address from the last 32 bytes of the calldata without checking if the calldata is long 
enough. This could result in reading garbage from memory. Consider performing a length validation to make sure there 
is a sender actually encoded.

#### 4.2. `MetaTxForwarder` does not bubble-up reverted transactions

`MetaTxForwarder` implements a method called `forwardMetaTx` that forwards any desired call to the specified target and 
redirects any returned data in case the call succeeds. However, in case the function call reverts, a generic revert 
reason is thrown. Ideally, it would be useful to bubble up the received reason if there was any.

#### 4.3. `OwnableInitializable` does not prevent initializing it more than once

`OwnableInitializable` has an internal method called `_initOwnable` that triggers the initialization logic. Based on 
the name of this function and the inline documentation tied to it, that is logic should only be triggered once. 
However, this function does not prevent inheriting contracts from calling it more than once. Consider adding some
validation to make sure this method cannot be triggered more than once.

#### 4.4. `NativeMetaTransaction`'s `nonces` doesn't have an explicit visibility

`NativeMetaTransaction` defines a storage mapping called `nonces` that does not have an explicit visibility modifier.
Consider marking it as `internal` to avoid inheriting contracts to manipulate it undesirably or at least making it
`public` explicitly.

#### 4.5. `NativeMetaTransaction` does not bubble-up reverted transactions

`NativeMetaTransaction` implements a method called `executeMetaTransaction` that forwards any desired call to itself 
making use of the meta-transactions pattern. This function redirects any returned data in case the call succeeds. 
However, in case the function call reverts, a generic revert reason is thrown. Ideally, it would be useful to bubble 
up the received reason if there was any.

#### 4.6. `EIP712Base` implements a confusing domain separator

EIP712 proposes different fields that can be used as part of the domain separator of each protocol. Two of these are
the chain ID and an arbitrary salt value. `EIP712Base` implements this standard defining a domain separator that uses
these fields in a weird way, the chain ID is being used a salt value. Note that this implementation is still compliant
with the standard tho. However, it is a bit confusing. Consider using the chain ID as the proper chain ID value while
ignoring the salt field.

#### 4.7. `Rarities` internal access is error-prone

Even though there is no mistake in the current implementation, the way rarities are stored in `Rarities` is error-prone.
On one hand, there is a mapping called `raritiesIndex` that is supposed to work along with the `rarities` storage array.
However, this mapping contains values that are not actually array indexes, but they refer to the index `value - 1`.
Note this is not how array indexes are meant to be used in Solidity. Consider using directly a mapping to access
rarities instead of an array, and consider renaming the indexes mapping to something more accurate like `raritiesId` 
or similar.

Additionally, the `raritiesIndex` mapping has to be indexed using `keccak256(bytes(rarity.name.toLowerCase()))`, and
this logic is repeated every time the mapping is accessed. Consider extracting this to an internal getter so the reader
and the dev team don't mess up.

#### 4.8. `Rarities`'s `rarityIndex` doesn't have an explicit visibility

`Rarities` defines a storage mapping called `rarityIndex` that does not have an explicit visibility modifier.
Consider marking it as `internal` to avoid inheriting contracts to manipulate it undesirably or at least making it
`public` explicitly.

#### 4.9. `ERC721CollectionFactoryV2` implementations are not necessarily `Ownable` compliant

`ERC721CollectionFactoryV2` relies on `MinimalProxyFactory` which simply creates proxy based on an arbitrary 
implementation contract that can be set at any time by the factory owner. However, `ERC721CollectionFactoryV2` requires
this implementation contract to be `Ownable` compliant since it is calling `transferOwnership` right after creating a 
new proxy instance. This is not enforced by code, and it may produce unexpected errors. 

Consider either making it explicit or importing that logic directly to the `MinimalProxyFactory` contract.

#### 4.10. `ERC721CollectionFactoryV3` changes the purpose of its implementation

`ERC721CollectionFactoryV3` is an upgraded version of `ERC721CollectionFactoryV2` using beacon proxies. These factories
have a method called `_setImplementation`. In V2 this method in used in order to change the implementation of the proxy 
contract the factory should deploy every time a new instance was created. However, V3 introduces beacon proxies, making
this method a bit different. The implementation being set is no longer the implementation of the proxy contract. 
Instead, this parameter is being used in order to set the beacon of the proxies that will be deployed every time a new 
instance is requested to the factory.

Consider either renaming the related variables and methods to something more accurate or writing some inline 
documentation to avoid confusing the reader.

#### 4.11. `CollectionManager` does not bubble-up reverted transactions

`CollectionManager` implements a method called `createCollection` that ends up forwarding a call to an `IForwarder`
instance received by parameter. However, in case this call does not succeed, a generic revert reason is thrown. 
Ideally, it would be useful to bubble up the received reason if there was any.

#### 4.12. `CollectionStore` does not emit `SetFeeOwner` during initialization

`CollectionStore` implements a method called `setFeeOwner` that emits a `SetFeeOwner` event every time a new fee owner
is set in the contract. However, the contract constructor sets a fee owner manually without emitting this event. 
Consider calling the `setFeeOwner` function instead.

#### 4.13. `ERC721BaseCollectionV2` does not emit `SetApproved` during initialization

`ERC721BaseCollectionV2` implements a method called `setApproved` that emits a `SetApproved` event when the `approved`
attribute is set. However, the contract also defines an `initialize` function to initialize the contract that sets this
attribute manually without emitting this event. Consider calling the `setApproved` function instead.

#### 4.14. `ERC721BaseCollectionV2` does not emit `CreatorshipTransferred` during initialization

`ERC721BaseCollectionV2` implements a method called `transferCreatorship` that emits a `CreatorshipTransferred` event 
every time a new creator is set. However, the contract also defines an `initialize` function to initialize the contract 
that sets the creator manually without emitting this event. Consider calling the `transferCreatorship` function instead.

#### 4.15. `ERC721BaseCollectionV2` does not emit `SetEditable` during initialization

`ERC721BaseCollectionV2` implements a method called `setEditable` that emits a `SetEditable` event every time the
`editable` attribute is set. However, the contract also defines an `initialize` function to initialize the contract that 
sets this attribute manually without emitting this event. Consider calling the `setEditable` function instead.

#### 4.16. Duplicated `Rarities` reference between `ERC721BaseCollectionV2` and `CollectionManager`

The `CollectionManager` contract has a reference to the `Rarities` contract that can be changed at any time through the
`setRarities` function. Every time `CollectionManager` creates a new `ERC721BaseCollectionV2` instance, this reference 
is passed and stored in the collection contract. However, the `Rarities` reference in `ERC721BaseCollectionV2` cannot
be changed. This is a weird pattern as these reference may not be the same, and it is relevant when adding items to 
the collection. Consider having a single source of truth for the current `Rarities` instance. 

## 5. Notes

#### 5.1. `ContextMixin` can be optimized

ContextMixin loads the whole calldata array to memory in order to extract the original sender. 
Consider using `calldataload` instead to load exactly the desired chunk.

#### 5.2. `ContextMixin` mask is ambiguous

ContextMixin relies on a mask to decode the original sender from the calldata. It uses the following mask 
`0xffffffffffffffffffffffffffffffffffffffff` to decode an address applying it to a word read from calldata.
It is ambiguous to the reader if the mask is computed as least or most significant bits. Consider using a 
32 bytes mask instead `0x000000000000000000000000ffffffffffffffffffffffffffffffffffffffff`.

#### 5.3. `OwnableInitializable` repeated logic

OwnableInitializable repeats the logic of updating `_owner` and emitting the `OwnershipTransferred` event at least 3
times. Consider extracting these to an internal method called `_setOwner` or similar to avoid potential issues.

#### 5.4. `NativeMetaTransaction` misleading wording

NativeMetaTransaction uses an internal struct called `MetaTransaction` that has a bytes array parameter called 
`functionSelector`. However, this parameter is actually meant to be a data array to be sent as a calldata, which is 
a wider concept compared to a function selector. Consider renaming it to avoid confusing readers.

#### 5.5. `Rarities` implementations does not explicitly extend from `IRarities`

Both `Rarities` and `RaritiesWithOracle` do not explicitly implement the proposed interface contract `IRarities`. 
Consider making both explicitly inherit from `IRarities`.

#### 5.6. `Rarities` express a price only using a numeric value

That said, the `CollectionManager` is where the asset that will be used to pay for the rarities comes in.
It's weird that the implementation supports changing the asset which may affect completely the pricing logic.

#### 5.7. Meaningless `_setImplementation` method in proxy factories

Both factories `MinimalProxyFactory` and `BeaconProxyFactory` defines an internal method called `_setImplementation`.
This method is not being used in the entire codebase except from the constructor in both cases. Consider inlining the
implementation or at least making it `private` to denote that.

#### 5.8. `CollectionManager` does not explicitly extend from `ICollectionManager`

`CollectionManager` does not explicitly implement the proposed interface contract and used by other dependencies.
Consider making `CollectionManager` explicitly inherit from `ICollectionManager`.

#### 5.9. Collections hash value can be cached in `CollectionManager`

`CollectionManager` implements a method called `manageCollection` that performs a check against a collection contract
received by parameter to validate that the returned value after calling `COLLECTION_HASH` is equals to 
`keccak256("Decentraland Collection")`. This value is computed in runtime when it can be extracted to a constant 
variable instead.

#### 5.10. `IERC721CollectionV2`'s `items` method is ambiguous

`IERC721CollectionV2` exposes a method called `items` in the interface that returns a tuple of 7 items but none of these
is named. This is error-prone and may lead the user to use this interface mistakenly. Consider naming each of the 
returned values.

#### 5.11. `CollectionStore`'s `buy` method does not check arrays' length properly

`CollectionStore` defines a method called `buy` in order to allow a user to buy a list of ERC721 tokens from a 
collection. This method receives a list of items, and each item is meant to be a list of IDs and beneficiaries to 
denote the list of tokens the user is willing to buy from each collection for each beneficiary. After performing the 
entire accounting logic the `CollectionStore` contract calls `issueTokens` to each of the collections given by the user 
in the list. There is where the validation to make sure that the length of the item IDs and beneficiaries take place.
Consider performing this validation in the `CollectionStore` at the beginning of the process to avoid consuming the 
user's gas unnecessarily.

#### 5.12. `ERC721BaseCollectionV2`'s `itemMinters` actually means minter's allowance

`ERC721BaseCollectionV2` defines a storage mapping called `itemMinters` that specifies a numeric value per account.
This value actually means the number of items a minter is allowed to mint on behalf of the collection's creator, i.e.
an allowance value. Consider renaming it to something more accurate.

#### 5.13. Weird import paths

There are multiple imports typos as follows: `import x//y.sol`. Consider looking for all the matches with the regex 
`import.*\/\/` and fix them. 

#### 5.14. Re-use standard ERC721 implementation from OpenZeppelin

Comparing the current implementation in `ERC721Initializable` to the one provided by OpenZeppelin, there is no 
difference at all. Consider reusing to avoid maintaining an implementation of a standard that has already been 
widely adopted by the community.

#### 5.15. Unused variables or methods

- `EIP712Base` defines a struct related to the standard's domain separator called `EIP712Domain` but it is never used.
- `CollectionManager` defines a storage variable called `pricePerItem` that is not being used at all.

##### 5.16. There is no clear naming convention for variables 

Storage variables, function parameters, and event arguments are sometimes prefixed with `_` and sometimes not.
Specially the case for storage variables, it does nevertheless their visibility. Consider adopting a single convention
that protects you from shadowing but stays consistent along the codebase.

#### 5.17. Solidity linter is failing

The Solidity linter command fails with the current status of the contracts. Consider fixing it or dropping it entirely.
