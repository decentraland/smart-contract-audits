# Smart Contracts Well Knonw Issues

- Estate: The getFingerprint function can run out of gas for Estates bigger than 4000 LANDs.
- Collections: Missing the ERC165 interface registration.
- Rewards Smart Contract can't support two signatures for the same item at the same time. Also, the whole tx will fail if one item has been already claimed.
- TokenWrapper and TokenManager Aragon's apps allow anyone with balance to call any not blacklisted smart contract. This makes things like the token escape hatch useless. Anyone can move tokens only if they are sent by mistake.
