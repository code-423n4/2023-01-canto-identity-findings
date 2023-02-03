# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Immutable state variables lack zero address checks | Low | 3 |
| 2 | Use Openzeppelin `EnumerableSet.UintSet` for managing `SubprotocolData.active` | NC | 1 |
| 3 | Duplicated `require()` checks should be refactored to a modifier  | NC | 2 |
| 4 | Use scientific notation | NC | 3 |

## Findings

### 1- Immutable state variables lack zero address checks  :

Constructors should check the values written in an immutable state variables(address) is not the zero value (address(0))

#### Risk : Low

#### Proof of Concept
Instances include:

File: SubprotocolRegistry.sol [Line 66](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L66)
```
note = ERC20(_noteContract);
```

File: CidNFT.sol [Line 129-130](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L129-L130)
```
note = ERC20(_noteContract);
subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.

### 2- Use Openzeppelin `EnumerableSet.UintSet` for managing `SubprotocolData.active` :

#### Risk : NON CRITICAL

In the `CidNFT` contract you should consider using openzeppelin's `EnumerableSet.UintSet` library for handling the `SubprotocolData.active` values, this library has the functions add, remove, contains already pre-built and thus can be used immediatly in a single line of code by the contract instead of writing many lines to do so. The use of this library will improve the readability of the code.

The library can be used as follows :

```
using EnumerableSet for EnumerableSet.UintSet;

struct SubprotocolData {
    /// @notice Mapping for ordered type
    mapping(uint256 => uint256) ordered;
    /// @notice Value for primary type
    uint256 primary;
    /// @notice List for active type
    EnumerableSet.UintSet active;
}
```

And the contract add/remove logic should be replace by the inbuilt add/remove of the library and existance checks can be perfomed by the contains method.

### 3- Duplicated input check statements should be refactored to a modifier :

Input check statements used multiple times inside a contract should be refactored to a modifier for better readability and also to save gas.

#### Risk : NON CRITICAL

#### Proof of Concept

The following check statement is repeated in both `add` and `remove` functions :

```
address cidNFTOwner = ownerOf[_cidNFTID];
if (
    cidNFTOwner != msg.sender &&
    getApproved[_cidNFTID] != msg.sender &&
    !isApprovedForAll[cidNFTOwner][msg.sender]
) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```

Because this check is done on the caller (msg.sender) it can be placed at the beginning of the function and thus can be refactored into a modifier, it should be replaced by a `isAllowed` modifier as follow :

```
modifier isAllowed(uint256 _cidNFTID){
    address cidNFTOwner = ownerOf[_cidNFTID];
    if (
        cidNFTOwner != msg.sender &&
        getApproved[_cidNFTID] != msg.sender &&
        !isApprovedForAll[cidNFTOwner][msg.sender]
    ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
    _;
}
```


### 4- Use scientific notation :

When using multiples of 10 you shouldn't use decimal literals or exponentiation (e.g. 1000000, 10**18) but use scientific notation for better readability.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

File: CidNFT.sol [Line 17](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L17)

```
uint256 public constant CID_FEE_BPS = 1_000;
```

File: CidNFT.sol [Line 191](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191)
```
uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

File: SubprotocolRegistry.sol [Line 17](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17)

```
uint256 public constant REGISTER_FEE = 100 * 10**18;
```

#### Mitigation

Use scientific notation for the instances aforementioned.
