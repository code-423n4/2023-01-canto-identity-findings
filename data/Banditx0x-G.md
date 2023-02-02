1. subprotocolNFTID is used only in the getPrimaryData() and getActiveData() functions. Can use memory instead of storage.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L312

2. uint256 cidNFTID in function remove() can be stored using "memory" as it is not used externally.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L52-L53

3. cidNFTID in function getCID() can be stored in "memory" as it is not used externally.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L62-L63




