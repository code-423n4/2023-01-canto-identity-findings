# Report

## QA Findings


| |Issue|Instances|
|-|:-|:-:|
| [L-1] | Wrong math operation in constant REGISTER_FEE | 1 |
| [NC-1] | Error NotAuthorizedForSubprotocolNFT is not used | 1 |

### [L-1] Wrong math operation in constant REGISTER_FEE
Math used in a constant REGISTER_FEE produce an output with 20 digits instead of 18 (like in most tokens) and it can lead to unexpected behaviors in the future like cutted values, lack of compatibility with other tokens even if at the moment it's not the case.

*Instances (1)*:
```solidity
File: SubprotocolRegistry.sol

17:    uint256 public constant REGISTER_FEE = 100 * 10**18;

```
### [NC-1] Error NotAuthorizedForSubprotocolNFT is not used
Not used errors will lead to confusion and unnecessary costs in gas.

*Instances (1)*:
```solidity
File: CidNFT.sol

106:    error NotAuthorizedForSubprotocolNFT(address caller, uint256 subprotocolNFTID);

```
