
# Decentraland Wearables Audit Report.

# 1. Summary

This document is a security audit report where [Decentraland Wearables](https://github.com/decentraland/wearables-contracts/tree/4c63728337f75543a6d7ff29dc8e02e0adfe252b) has been reviewed.

# 2. In scope

- [String.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/libs/String.sol) github commit 2a8c65e73181f860ed361aef984c4b537210e2bb.
- [CollectionStore.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/markets/v2/CollectionStore.sol) github commit 9d605cc24386ca90b94f6b6a7011cf09caf60ffe.
- [ERC721BaseCollectionV2.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/collections/v2/ERC721BaseCollectionV2.sol) github commit e4e9def1886dcbd14576d8ac955bea2662b5f28f.
- [ERC721Initializable.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/tokens/ERC721Initializable.sol) github commit e4e9def1886dcbd14576d8ac955bea2662b5f28f.
- [ERC721CollectionFactoryV2.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/factories/v2/ERC721CollectionFactoryV2.sol) github commit b786a3d6bdfeec21c8338965e41ae5af1a1f7a4a.
- [OwnableInitializable.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/commons/OwnableInitializable.sol) github commit e4e9def1886dcbd14576d8ac955bea2662b5f28f.
- [ContextMixin.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/commons/ContextMixin.sol) github commit e4e9def1886dcbd14576d8ac955bea2662b5f28f.
- [EIP712Base.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/commons/EIP712Base.sol) github commit e4e9def1886dcbd14576d8ac955bea2662b5f28f.
- [NativeMetaTransaction.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/commons/NativeMetaTransaction.sol) github commit e4e9def1886dcbd14576d8ac955bea2662b5f28f.
- [MinimalProxyFactory.sol](https://github.com/decentraland/wearables-contracts/blob/4c63728337f75543a6d7ff29dc8e02e0adfe252b/contracts/commons/MinimalProxyFactory.sol) github commit e8132adb00774d5b30ba57e4fd1c886867b73963.

# 3. Findings

** 5 ** were reported including:

- 1 low risk issue.
- 3 informational.
- 1 Undetermined.

## 3.1. Proxy Factory

### Severity: low

### Contracts Involved: `MinimalProxyFactory`, `ERC721CollectionFactoryV2`

### Description

`ERC721CollectionFactoryV2.createCollection` is a public function that can be called by any user to create a new collection (the owner will always be the owner of the proxy factory contract and the user will be able to select the creator address that manage the collection). However, `MinimalProxyFactory.createProxy` is also public but the ownership is not transfered to the proxy factory owner in this case (the calleris free to call `initialize` or not). Depending on how a collection is recognized to be a valid Decentraland collection this can be an issue.

### Recommendation

Change `ERC721CollectionFactoryV2.createProxy` function visibility to internal.

## 3.2. `executeMetaTransaction` does not validate ECDSA parameters

### Severity: informational

### Contracts Involved: `NativeMetaTransaction`

### Description

The function does not validate either of the `s` and `v`. values. See [ECDSA.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/cryptography/ECDSA.sol#L46).

### Recommendation

Add checks for the `s` and `v` parameters.

## 3.3. Meta Transaction Execution

### Severity: undertermined

### Contracts Involved: `NativeMetaTransaction`

### Description

Following the naming, it is unclear if `executeMetaTransaction` parameter `functionSignature` contains only the called function signature or the full call data (signature + prameters).

### Recommendation

Either change the parameter naming or the design.

## 3.4. Unlocked Pragma

### Severity: Informational

### Contracts Involved: 

- `MinimalProxyFactory`
- `OwnableInitializable`
- `ERC721CollectionFactoryV2`
- `String`

### Description

Every Solidity file specifies in the header a version number of the format pragma solidity `(^)0.*.*.` The caret (^) before the version number implies an unlocked pragma, meaning that the compiler will use the specified version and above, hence the term "unlocked."

### Recommendation

For consistency and to prevent unexpected behavior in the future, it is recommended to remove the caret to lock the file onto a specific Solidity version.

## 3.5. Privileged Roles and Ownership

### Severity: Informational

### Contracts Involved: `ERC721BaseCollectionV2`

### Description

The creator and the manager are allowed to modify items using `editItemsMetadata` (`isEditable` is set to true by default), please note that even the sold items can be modified.  

### Recommendation

If the description above is part of the design, the privileged roles need to be made clear to the users, especially depending on the level of privilege the contract allows to the different addresses.