# Report

## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1] | Use bytes32 instead of string | 1 |
| [GAS-2] | Use private instead of public for constant | 1 |
| [GAS-3] | Use unchecked keyword inside for loops | 1 |

### [GAS-1] Use bytes32 instead of string
Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming than bytes32.

*Instances (1)*:
```solidity
File: CidNFT.sol

37:    string public baseURI;

```
### [GAS-2] Use private instead of public for constant
Public constant in solidity require that their value is known at compile time, so their value must be included in the contract code. This results in a larger contract size, which in turn requires more gas to deploy the contract.

*Instances (1)*:
```solidity
File: CidNFT.sol

17:    uint256 public constant CID_FEE_BPS = 1_000;

```
### [GAS-3] Use unchecked keyword inside for loops
Unchecked keyword should be used inside the for loop for incrementor part if there is no risk of overflowing a value. It saves about 30-40 gas per loop.

*Instances (1)*:
```solidity
File: CidNFT.sol

150:    for (uint256 i = 0; i < _addList.length; ++i) {

```