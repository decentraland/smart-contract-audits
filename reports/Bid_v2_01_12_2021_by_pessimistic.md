# Decentraland Bid

tags: Public,Results

Project Info

repo - https://github.com/decentraland/bid-contract/tree/feat/add-royalties

commit - 80ae6c417006a563127cdec41884b2eea8520593

scope - contracts/bid/

SLOC-P - 469

documentation - Readme in the repository

link - https://decentraland.org/

## Project assessment

There are four non-code aspects of the project that we assess as part of the audit:

documentation - OK
compilation - OK
tests - OK
dependency management - OK
Code Issues
Most issues are described in format

<contract>.<function> lines (<severity>, <type>) - description

### Severity levels are:

- critical - loss of funds or complete fail of project
- medium - the contracts don’t behave as expected, but assets are safe
- low - doesn’t really affect operations, but might lead to a more serious problem in the future versions of the code
  Type loosely describes what is this issue about:

- code - the code doesn’t follow the best practices. Overall idea/design is fine, but the implementation could be improved.

- oo - overpowered owner. The code could be rewritten in a trustless manner.

### Findings

- ERC721Bid.sol (med, oo) - The contract owner is able to set fees to any value at any time, as well as to pause the contract, preventing any interaction with it. We recommend designing contracts in a trustless manner or implementing proper key management, e.g., setting up a multisig.

- ERC721Bid.sol (low, code) - Bids are accepted by using safeTransferFrom() and providing bidId as a \_data argument. However, if the token owner provides wrong bidId, the contract execution fails with an irrelevant message. Namely, with the one used for insufficient funds or insufficient allowance. We recommend adding meaningful error description for the mentioned case.

- ERC721Bid.sol (low, code) - Argument bidId in getBidByBidder() function is redundant and is not used anywhere. We recommend removing it.

- ERC721Bid.sol (low, code) - In the code there is a function for clearing expired bids. However, there is no option for clearing bids that have expired due to another bid being accepted. We recommend implementing cleanup function that checks bidCounter for the token and bid index.

- ERC721Bid.sol (low, code) - Lines 488-490 contain if clause with a revert() function inside. We recommend using require() function instead.

- NativeMetaTransaction.sol (low, code) - executeMetaTransaction() function is payable, however there is no logic for withdrawing ether in ERC721Bid.sol. Ether might get stuck in the contract.
