## Missing parameter validation in `constructor`
Some parameters of constructors are not checked for invalid values
```
        baseURI = _baseURI;
        cidFeeWallet = _cidFeeWallet;
        note = ERC20(_noteContract);
        subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
```
[CidNFT.sol#L127-L131](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L127-L131)

