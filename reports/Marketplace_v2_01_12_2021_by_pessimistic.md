#Decentraland Marketplace V2

tags: Public,Results

Project Info

repo - https://github.com/decentraland/marketplace-contracts/pull/56

commit - 86de52dd82978746537a7c65be28df6fc1fc1885

scope - MarketplaceV2.sol + RoyaltiesManager.sol

SLOC-P - 450

documentation - https://github.com/decentraland/proposals/blob/master/dsp/dsp-0013/0013.md

## Project assessment

There are four non-code aspects of the project that we assess as part of the audit:

documentation - OK
compilation - OK
tests - 91/91 OK, coverage 75%
dependency management - OK
Code Issues
Most issues are described in format

<contract>.<function> lines (<severity>, <type>) - description

### Severity levels are:

- critical - loss of funds or complete fail of project
- medium - the contracts don’t behave as expected, but assets are safe
- low - doesn’t really affect operations, but might lead to a more serious problem in the future versions of the code

### Type loosely describes what is this issue about:

- code - the code doesn’t follow the best practices. Overall idea/design is fine, but the implementation could be improved.

- logic - the code’s logic is incorrect, as it is a bug, or a missed edge case, or improper integration with other contract

- gas - an opportunity to save gas

- docs - documentation issue

### Findings

- MarketplaceV2 (med, logic) - the contract is Pausable, however pause and unpause functions (with proper access control over calling corresponding internal functions) are not implemented. Thus, there is no real possibility to make an actual pause.
- NativeMetaTransaction.executeMetaTransaction (low, code) - function is marked as payable, but no ether transfer specified in the contract. Consider removing payable keyword to prevent locked money issue.
- RoyaltiesManager.getRoyaltiesReceiver L46 (low, code) - consider returning a struct instead of separate values in IERC721CollectionV2 interface to avoid comma notation, which is error-prone.
- RoyaltiesManager (low, code) - since royalties is relatively new area, it is not yet standardized. Nevertheless, implementing one of the proposed standards (i.e. EIP 2981) might be useful for future integration.
- RoyaltiesManager.getRoyaltiesReceiver L32 (low, code) - avoid returning non-initialized values. In this case return value might be explicitly set to address(0).
- MarketplaceV2 L20 (low, gas) - Variable acceptedToken is set during contract deployment and never changes later. Declare it as immutable to reduce gas consumption.
- MarketplaceV2.\_requireERC721 L469 (low, code) - redundant isContract() check, since the compiler includes this check before external calls.
- MarketplaceV2 (low, code) - Consider declaring functions as external instead of public when possible to improve code readability and optimize gas consumption.
- MarketplaceV2 (low, docs) - most of NatSpec comments do not include @return parameter, consider adding it to description.
