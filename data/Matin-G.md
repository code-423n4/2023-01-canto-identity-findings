
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Storing the struct value of the mapping as ```storage``` | 1 |  2000 |
| [G&#x2011;02] | Using uint256 costs low gas rather than ```enum``` types (or uint8) | 6 |  300 |
| [G&#x2011;03] | Keyword ```delete``` for mappings[_key] may save some tiny amount of gas rather than make it zero manually. | 1 |  40 |
| [G&#x2011;04] | Setting the ```constructor``` to be ```payable``` | 3 |  180 |
| [G&#x2011;05] | Use solidity version 0.8.17 to save some gas | 4 |  - |


Total: 15 instances over 5 issues with **2520 gas** saved

The table above as well as its gas numbers are created by considering the **automatic findings** which are not included.





## Gas Optimizations

### [G&#x2011;01]  Storing the struct value of the mapping as ```storage```
Storing the struct value of the mapping as ```storage``` costs less than ```memory``` inside a function.
Declaring the ```subprotocols[_name]``` as memory and re-assigning the values to it at the end of the function costs
about 2000 gas more than the case in which it is defined as storage (storage pointer for the struct value part of mapping).

*There is 1 instance of this issue:*

```solidity
    SubprotocolData memory subprotocolData = subprotocols[_name];
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L89



### [G&#x2011;02]  Using uint256 costs low gas rather than ```enum``` types (or uint8)
Using uint256 costs low gas rather than enum types (or uint8) for condition checks inside contract ```CidNFT```. As the compiler bypasses the
explicit ```uint8``` to `uint256``` conversion inside the conditional terms, it saves some gas.

*There are 6 instances of this issue:*

```solidity
    if (_type == AssociationType.ORDERED)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L195

```solidity
    else if (_type == AssociationType.PRIMARY)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L203

```solidity
    else if (_type == AssociationType.ACTIVE)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L211

```solidity
    if (_type == AssociationType.ORDERED)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L257

```solidity
   else if (_type == AssociationType.PRIMARY)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L266

```solidity
    else if (_type == AssociationType.ACTIVE)
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L272



### [G&#x2011;03]  Keyword ```delete``` for mappings[_key] may save some tiny amount of gas rather than make it zero manually
By using the keyword ```delete``` for a specific key of a mapping, the value of that mapping would be re-initialized to its default value.

*There is 1 instance of this issue:*

```solidity
    activeData.positions[_nftIDToRemove] = 0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L284



### [G&#x2011;04]  Setting the ```constructor``` to be ```payable``` 
Save some gas by setting the ```constructor``` to be ```payable```.

*There are 3 instances of this issue:*

```solidity
    constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI,
        address _cidFeeWallet,
        address _noteContract,
        address _subprotocolRegistry
    ) ERC721(_name, _symbol) {
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119

```solidity
    constructor(address _noteContract, address _cidFeeWallet) {
        note = ERC20(_noteContract);
        cidFeeWallet = _cidFeeWallet;
    }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65

```solidity
    constructor(address _cidNFT) {
        cidNFT = _cidNFT;
    }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L36




### [G&#x2011;05]  Use solidity version 0.8.17 to save some gas
Set the solidity compiler version to the recent version 0.8.17.

*There are 4 instances of this issue:*

```solidity
    pragma solidity >=0.8.0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidSubprotocolNFT.sol#L2

