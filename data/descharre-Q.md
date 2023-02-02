# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       |Other contract addresses can only be set once|6|
|L-02       |Unspecific compiler version pragma|6|


## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | Use scientific notation (1E18) rather than exponentation (10**18) | 1 |
|N-02       | Require() or revert() statement that check input arguments should be at the top of the function | 1 |
|8       |Miscellaneous|1|
# Details
## Low Risk
## [L-01] Other contract addresses can only be set once
Other contract addresses can only be set once in the constructor and they are immutable. There is no other setter method for this. This makes it so there is no room for error and can lead to redeployment.
- [AddressRegistry.sol#L14](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L14)
- [CidNFT.sol#L24-L30](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L24-L30)
- [SubprotocolRegistry.sol#L20-L23](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L20-L23)
## [L-02] Unspecific compiler version pragma
Avoid floating or unspecif pragmas for non-library contracts.
Contracts should be deployed with the same compiler version that they have been tested with. When you lock the pragma, it ensures that the contract does not get accidently deployed with a compiler that has bugs.

This is for all the 3 contracts in scope
## Non critical
## [N-01] Use scientific notation (1E18) rather than exponentation (10**18)
Scientific notation should be used for better code readability.

[SubprotocolRegistry.sol#L17](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17)
## [N-02] Require() or revert() statement that check input arguments should be at the top of the function
This way no gas is wasted when the function is destined to fail.

- [SubprotocolRegistry.sol#L88](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L88)
- [SubprotocolRegistry.sol#L93](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L93)
- [CidNFT.sol#L183](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L183)

