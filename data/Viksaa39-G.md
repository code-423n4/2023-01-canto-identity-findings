**Setting the Constructor to Payable**

We can save gas if we mark constructor as payable. 

There are 2 instances of this issue:

- [https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119)
- [https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36)