## Use latest Solidity version with a stable pragma statement
[AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol), [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol), [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)

## Event emitted with wrong parameters
```
emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);
```
```
emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
```
Should be `currNFTID` instead of `_nftIDToRemove` for both lines.

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L265
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L271