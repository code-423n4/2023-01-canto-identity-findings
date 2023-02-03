
### 01 FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

*Number of Instances Identified: 8*

### Description

Our Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

### Recommendation

`import {contract1 , contract2} from "filename.sol";`

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

```
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./SubprotocolRegistry.sol";
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
4: import "solmate/tokens/ERC721.sol";
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./CidSubprotocolNFT.sol";
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol

```
4: import "solmate/tokens/ERC721.sol";
```


### 02 NATSPEC IS INCOMPLETE

*Number of Instances Identified: 2*

Missing: `@param` 

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol

```
File: src/AddressRegistry.sol

42: function register(uint256 _cidNFTID) external {
52-53: function remove() external {
        uint256 cidNFTID = cidNFTs[msg.sender];
```



### 03 SCIENTIFIC NOTATION SHOULD BE USED RATHER THAN EXPONENTIATION

*Number of Instances Identified: 1*

Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. `10**18`)

Scientific notation should be used for better code readability

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```


### 04 USE LATEST SOLIDITY VERSION

*Number of Instances Identified: 3*

When deploying contracts, you should use the latest released version of Solidity. Apart from exceptional cases, only the latest version receives [security fixes](https://github.com/ethereum/solidity/security/policy#supported-versions). Furthermore, breaking changes as well as new features are introduced regularly.
Latest Version is 0.8.17

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

```
2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol

```
2: pragma solidity >=0.8.0;
```

