# QA REPORT

---

### Summary of non-critical issues


| Number       | Issue details                                                                   | Instances |
| -------------- | --------------------------------------------------------------------------------- | :---------: |
| [NC-1](#NC3) | Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`). |     1     |
| [NC-2](#NC4) | Lines are too long.                                                             |     1     |
| [NC-3](#NC5) | For modern and more readable code, update import usages.                        |     9     |
| [NC-4](#NC7) | Use of`bytes.concat()` instead of `abi.encodePacked()`.                         |     2     |

*Total: 7 issues.*

### Non-critical Issues

### <a id=NC3>[NC-1]</a> Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`).

##### Description

Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`) that could lead to errors.

##### Recommendation

Use scientific notation instead of exponentiation.

##### *Instances (1):*

File: [2023-01-canto-identity/src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17 )

```solidity
17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```

### <a id=NC4>[NC-2]</a> Lines are too long.

##### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

##### Recommendation

Reduce number of characters per line to improve readability.

##### *Instances (1):*

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L185 )

```solidity
185: // The CID Protocol safeguards the NFTs of subprotocols. Note that these NFTs are usually pointers to other data / NFTs (e.g., to an image NFT for profile pictures)
```

### <a id=NC5>[NC-3]</a> For modern and more readable code, update import usages.

##### Description

Solidity code is cleaner in the following way: On the principle that clearer code is better code, you should import the things you want to use. Specific imports with curly braces allow us to apply this rule better. Check out this [article](https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a)

##### Recommendation

Import like this: `import {contract1 , contract2} from "filename.sol";`

##### *Instances (9):*

File: [2023-01-canto-identity/src/AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L4 )

```solidity
4: import "solmate/tokens/ERC721.sol";
```

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L5 )

```solidity
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./SubprotocolRegistry.sol";
```

File: [2023-01-canto-identity/src/CidSubprotocolNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidSubprotocolNFT.sol#L4 )

```solidity
4: import "solmate/tokens/ERC721.sol";
```

File: [2023-01-canto-identity/src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L4 )

```solidity
4: import "solmate/tokens/ERC721.sol";
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./CidSubprotocolNFT.sol";
```

### <a id=NC7>[NC-4]</a> Use of `bytes.concat()` instead of `abi.encodePacked()`.

##### Description

Rather than using `abi.encodePacked` for appending bytes, since version `0.8.4`, `bytes.concat()` is enabled.

##### Recommendation

Since version `0.8.4` for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.

##### *Instances (2):*

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L140 )

```solidity
140: return string(abi.encodePacked(baseURI, _id, ".json"));
154: ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
```
