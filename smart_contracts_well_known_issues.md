# Smart Contracts Well Knonw Issues

- LAND & Estate Contracts: The `getFingerprint`, and some function to get assets from the users can run out of gas for Estates bigger than 4000 LANDs and users with a lot of assets. They can use other functions to workaround this.
- Collections: Missing the ERC165 interface registration.
- CollectionManager: Possible front-running of MANA being paid when creating a collection. It does not generate an issue because the MANA goes to the DAO and can be returned.
- Rewards Smart Contract can't support two signatures for the same item at the same time. Also, the whole tx will fail if one item has been already claimed.
- TokenWrapper and TokenManager Aragon's apps allow anyone with balance to call any not blacklisted smart contract. This makes things like the token escape hatch useless. Anyone can move tokens only if they are sent by mistake.
- The Rentals contract is not compliant with the LANDRegistry `tokenMetadata` interface (`bytes4(keccak256("getMetadata(uint256)"))`).
