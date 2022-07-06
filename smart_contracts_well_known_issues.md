# Smart Contracts Well Knonw Issues

- Estate: The getFingerprint function can run out of gas for Estates bigger than 4000 LANDs.
- Collections: Missing the ERC165 interface registration.
- Rewards Smart Contract can't support two signatures for the same item at the same time. Also, the whole tx will fail if one item has been already claimed.
