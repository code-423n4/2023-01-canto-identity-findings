## Low Risk

### 

## Non-Critical

### Use IERC721 instead of ERC721

It's better to use the interface instead of the interface of a specific implementation

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#LL186C14-L186C14

```
        ERC721 nftToAdd = ERC721(subprotocolData.nftAddress);
```