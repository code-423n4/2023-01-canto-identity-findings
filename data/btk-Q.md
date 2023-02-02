| Total Low issues |
|-------------------|

| Risk    | Issues Details                                                            | Number        |
|---------|---------------------------------------------------------------------------|---------------|
| [L-01]  | Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists | 2             |
| [L-02]  | Loss of precision due to rounding                                         | 1             |
| [L-03]  | `Address(0)` checks                                                       | 2             |
| [L-04]  | Using `>/>=` without specifying an upper bound is unsafe                  | 3             |

| Total Non-Critical issues |
|----------------------------|

| Risk    | Issues Details                                                            | Number        |
|---------|---------------------------------------------------------------------------|---------------|
| [NC-01] | Use a more recent version of solidity                                     | All Contracts |
| [NC-02] | Constants in comparisons should appear on the left side                   | 6             |
| [NC-03] | Using vulnerable dependency of solmate                                    | 1             |
| [NC-04] | Use `bytes.concat()` and `string.concat()`                                | 2             |
| [NC-05] | Non-usage of specific imports                                             | 8             |
| [NC-06] | Include `@return` parameters in NatSpec comments                          | 8             |
| [NC-07] | Function writing does not comply with the `Solidity Style Guide`          | All Contracts |
| [NC-08] | Lines are too long                                                        | 5             |
| [NC-09] | Follow Solidity standard naming conventions                               | All Contracts |
| [NC-10] | Add NatSpec comment to `mapping`                                          | 4             |
| [NC-11] | Consider using `delete` rather than assigning zero to clear values        | 1             |
| [NC-12] | Use SMTChecker                                                            |               |
| [NC-13] | Use scientific notation rather than exponentiation                        | 1             |

## [L-01] Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

#### Description

Solmate's `SafeTransferLib.sol`, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn't exist.

> Ref: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

```solidity
/// @dev Note that none of the functions in this library check that a token has code at all! That responsibility is delegated to the caller.
```

#### Lines of code 

- [CidNFT.sol#L6](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L6)
- [SubprotocolRegistry.sol#L6](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L6)

#### Recommended Mitigation Steps

Add a contract exist check in functions or use Openzeppelin `safeERC20` instead.

## [L-02] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

```solidity
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

#### Lines of code 

- [CidNFT.sol#L191](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191)

#### Recommended Mitigation Steps

Add scalars so roundings are negligible.

## [L-03] `Address(0)` checks

#### Description

Check of `address(0)` to protect the code from `(0x0000000000000000000000000000000000000000)` address problem just in case. This is best practice or instead of suggesting that they verify `_address != address(0)`, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

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

#### Lines of code 

- [CidNFT.sol#L124](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L124)
- [CidNFT.sol#L125](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L125)

#### Recommended Mitigation Steps

Add checks for address(0) when assigning values to address state variables.

## [L-04] Using `>/>=` without specifying an upper bound is unsafe

#### Description

There will be breaking changes in future versions of solidity, and at that point your code will no longer be compatable. While you may have the specific version to use in a configuration file, others that include your source files may not.

```solidity
pragma solidity >=0.8.0;
```

#### Lines of code 

- [AddressRegistry.sol#L2](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2)
- [SubprotocolRegistry.sol#L2](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2)
- [CidNFT.sol#L2](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2)

#### Recommended Mitigation Steps

Use a fixed version of solidity.

## [NC-01] Use a more recent version of solidity

#### Description

For security, it is best practice to use the [latest Solidity version](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

```solidity
pragma solidity >=0.8.0;
```

#### Lines of code 

- [AddressRegistry.sol#L2](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2)
- [SubprotocolRegistry.sol#L2](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2)
- [CidNFT.sol#L2](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2)

#### Recommended Mitigation Steps

Old version of Solidity is used `(0.8.0)`, newer version can be used `(0.8.17)`.

## [NC-02] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
        if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); 
```

#### Lines of code 

- [AddressRegistry.sol#L54](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L54)
- [CidNFT.sol#L183](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L183)
- [CidNFT.sol#L215](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L215)
- [CidNFT.sol#L260](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L260)
- [CidNFT.sol#L268](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L268)
- [CidNFT.sol#L275](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L275)

#### Recommended Mitigation Steps

```solidity
        if (0 == _nftIDToAdd) revert NFTIDZeroDisallowedForSubprotocols(); 
```

## [NC-03] Using vulnerable dependency of solmate

#### Description

The `package.json` configuration file says that the project is using `6.2.0` of solmate which has a not last update version.

```java script
{
  "name": "@rari-capital/solmate",
  "license": "AGPL-3.0-only",
  "version": "6.2.0",
  "description": "Modern, opinionated and gas optimized building blocks for smart contract development.",
  "files": [
    "src/**/*.sol"
  ],
```

#### Lines of code 

- [package.json#L4](https://github.com/transmissions11/solmate/blob/dd13c61b5f9cb5c539a7e356ba94a6c2979e9eb9/package.json#L4)

#### Recommended Mitigation Steps

Use solmate latest version `6.7.0`.

## [NC-04] Use `bytes.concat()` and `string.concat()`

#### Description

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

> https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

#### Lines of code 

- [CidNFT.sol#L140](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L140)
- [CidNFT.sol#L154](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L154)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()` instead (note that you must use a solidity version of at least 0.8.12).

## [NC-05] Non-usage of specific imports     

#### Description

Using import declarations of the form import `{<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

The Solidity docs recommend specifying imported symbols explicitly.

> Ref: https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

#### Lines of code 

- [CidNFT.sol#L5](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L5)
- [CidNFT.sol#L6](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L6)
- [CidNFT.sol#L7](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L7)
- [SubprotocolRegistry.sol#L4](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L4)
- [SubprotocolRegistry.sol#L5](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L5)
- [SubprotocolRegistry.sol#L](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L6)
- [SubprotocolRegistry.sol#L7](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L7)
- [AddressRegistry.sol#L4](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L4)

#### Recommended Mitigation Steps

Use specific imports syntax per solidity [docs](https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files) recommendation.

## [NC-06] Include `@return` parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `@return`. Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [CidNFT.sol#L344](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L344)

#### Recommended Mitigation Steps

Include the `@return` argument in the NatSpec comments.

## [NC-07] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external` / `public` / `internal` / `private`
- `view` / `pure`

#### Lines of code 

- [AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)
- [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)
- [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

#### Recommended Mitigation Steps

Follow [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html?highlight=Style#style-guide).

## [NC-08] Lines are too long

#### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

> Ref: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### Lines of code 

- [CidNFT.sol#L148](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L148)
- [CidNFT.sol#L185](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L185)
- [CidNFT.sol#L258](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L258)
- [CidNFT.sol#L261](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L261)
- [SubprotocolRegistry.sol#L78](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L78)

#### Recommended Mitigation Steps

Split the long lines when they reach the max length.

## [NC-09] Follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)
- [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)
- [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

#### Recommended Mitigation Steps

Follow solidity standard naming convention.

## [NC-10] Add NatSpec comment to `mapping`   

#### Description

Add NatSpec comments describing mapping keys and values.

```solidity
    mapping(address => uint256) private cidNFTs;
```

#### Lines of code 

- [AddressRegistry.sol#L21](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L21)
- [CidNFT.sol#L42](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L42)
- [CidNFT.sol#L48](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L48)
- [CidNFT.sol#L70](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L70)

#### Recommended Mitigation Steps

```solidity
 /// @dev  address(owner) => uint256(token Id)
    mapping(address => uint256) private cidNFTs;
```

## [NC-11] Consider using `delete` rather than assigning zero to clear values

#### Description

The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

```solidity
            activeData.positions[_nftIDToRemove] = 0;
```

#### Lines of code 

- [CidNFT.sol#L284](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L284)

#### Recommended Mitigation Steps

```solidity
            delete activeData.positions[_nftIDToRemove];
```

## [NC-12] Use SMTChecker

#### Description

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

> Ref: https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src)

#### Recommended Mitigation Steps

Use SMTChecker.

## [NC-13] Use scientific notation rather than exponentiation

#### Description

While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist.

```solidity
    uint256 public constant REGISTER_FEE = 100 * 10**18;
```

#### Lines of code 

- [SubprotocolRegistry.sol#L1](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17)

#### Recommended Mitigation Steps

Use scientific notation `(e.g. 1e18)` rather than exponentiation `(e.g. 10**18)`.
