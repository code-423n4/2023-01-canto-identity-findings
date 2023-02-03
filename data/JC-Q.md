
## Summary

N-01 constants should be defined rather than using magic numbers | 1 instance 
N-02 Use a more recent version of solidity | 3 instances 
N-03 Large multiples of ten should use scientific notation   | 2 instances 
N-04 revert() statements should have descriptive reason strings | 1 instance 
N-05 The visibility for constructor is ignored | 3 instances 

L-01 Lack of zero-address checks for immutable addresses | 1 instance 

Total: 11 instances in 6 issues

---


## N-01 constants should be defined rather than using magic numbers

Instance(1):

CidNFT.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191

```solidity
       uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```


## N-02 Use a more recent version of solidity

Use a solidity version of at least 0.8.2 to get compiler automatic inlining 
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads 
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings 
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value. 
Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>).
Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

Instances (3):

CidNFT.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2

```solidity
pragma solidity >=0.8.0;
```

SubprotocolRegistry.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2

```solidity
pragma solidity >=0.8.0;
```

AddressRegistry.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2

```solidity
pragma solidity >=0.8.0;
```

## N-03 Large multiples of ten should use scientific notation  

Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals (e.g. 1000000), for readability

Instances(2):

CidNFT.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L17

```solidity
    uint256 public constant CID_FEE_BPS = 1_000;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191

```solidity
    uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```


## N-04 revert() statements should have descriptive reason strings

Instances(3):

CidNFT.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L196

```solidity
if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L204
```solidity
if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L212

```solidity
if (!subprotocolData.active) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
```

## N-05 The visibility for constructor is ignored

Instances(3):

CidNFT.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119-L131

```solidity
    constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI,
        address _cidFeeWallet,
        address _noteContract,
        address _subprotocolRegistry
    ) ERC721(_name, _symbol) {
        baseURI = _baseURI;
        cidFeeWallet = _cidFeeWallet;
        note = ERC20(_noteContract);
        subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
    }
```

SubprotocolRegistry.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65-L68

```solidity
    constructor(address _noteContract, address _cidFeeWallet) {
        note = ERC20(_noteContract);
        cidFeeWallet = _cidFeeWallet;
    }
```

AddressRegistry.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36-L38

```solidity
    constructor(address _cidNFT) {
        cidNFT = _cidNFT;
    }
```


## L-01 Lack of zero-address checks for immutable addresses 

Lack of zero-address checks for immutable addresses will force contract redeployment if zero-address used accidentally. 
Zero-address checks as input validation on address parameters are always a best practice. 
This is especially true for critical addresses that are immutable and set in the constructor because they cannot be changed later. 
Accidentally using zero addresses here will lead to failing logic or force contract redeployment and increased gas costs. 

Recommend adding zero-address input validation for the address cidFeeWallet in the constructor.

Instance(1):

CidNFT.sol

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L128

```solidity
        cidFeeWallet = _cidFeeWallet;
```

