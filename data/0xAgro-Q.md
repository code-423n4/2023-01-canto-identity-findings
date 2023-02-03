# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Input Sanitization**](#1-Input-Sanitization)

[**Non-Critical**](#Non-Critical)
1. [**Use of Exponentiation Over Scientific Notation**](#1-Use-of-Exponentiation-Over-Scientific-Notation)
2. [**Order of Functions Not Compliant With Solidity Docs**](#2-Order-of-Functions-Not-Compliant-With-Solidity-Docs)
3. [**Long Lines**](#3-Long-Lines)
4. [**Inconsistent Named Returns**](#4-Inconsistent-Named-Returns)
5. [**Spelling Mistakes**](#5-Spelling-Mistakes)
6. [**Bulky Functions**](#6-Bulky-Functions)
7. [**Consider Gender Neutral Terminology**](#7-Consider-Gender-Neutral-Terminology)
8. [**Missing Max Fee Error Message**](#8-Missing-Max-Fee-Error-Message)
9. [**Inconsistent Trailing Period**](#9-Inconsistent-Trailing-Period)

# Low Severity

## 1. Input Sanitization

The `add` function in [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol) takes in an argument called `_key`. `_key` is only used in the `add` function if `_type == AssociationType.ORDERED`. To prevent possible user errors when calling the `add` function, consider adding a require statement forcing `_key` to be empty if `_type != AssociationType.ORDERED`.

# Non-Critical

## 1. Use of Exponentiation Over Scientific Notation

For better style and less computation consider replacing any power of 10 exponentiation (`10**3`) with its equivalent scientific notation (`1e3`). For code clarity it is understood why `100 * 10**18` is used. This issue is specifically about the `10**18`.

*/src/SubprotocolRegistry.sol*

```solidity
17:	uint256 public constant REGISTER_FEE = 100 * 10**18;
```
**Suggested Change**
```solidity
17:	uint256 public constant REGISTER_FEE = 100 * 1e18;
```

## 2. Order of Functions Not Compliant With Solidity Docs

The Solidity Style Guide suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol): external functions are positioned after public functions.

## 3. Long Lines

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

The following lines are longer than 120 characters, it is suggested to shorten these lines:

*/src/CidNFT.sol*
*  [148](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L148), [162](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L162), [185](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L185), [258](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L258), [261](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L261), [327](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L327). 

*/src/SubprotocolRegistry.sol*
*  [71](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L71), [72](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L72), [78](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L78). 

## 4. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. The following contracts only have named returns (EX. `returns(uint256 foo)`): [AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol).
3. The following contracts have both: [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol).

## 5. Spelling Mistakes

There are some spelling mistakes throughout the codebase. Consider fixing all spelling mistakes.

*/src/CidNFT.sol*

* The word `non-existent` is misspelled as [`non-existing`](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L138).
* The word `non-existent` is misspelled as [`non-existant`](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L274).
* The word `subprotocol` is misspelled as [`subprotocl`](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L306) (1).
* The word `subprotocol` is misspelled as [`subprotocl`](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L318) (2).

## 6. Bulky Functions

The core functions of the codebase (`add` and `remove` of [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L165)) are bulky and relatively hard to read as a result. Consider breaking the functions up into sub-functions (perhaps by association type).

## 7. Consider Gender Neutral Terminology

Permissionless software implies that anyone of any background that has access may use the system. Consider using as broad of language as possible when talking about users. Even if most are, probabilistically, not all users will be male.

*/src/CidNFT.sol*

```solidity
_mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
```

## 8. Missing Max Fee Error Message

When registering a new subprotocol in `register` of [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol) the max value is not checked. Although the function will revert if `_fee` exceeds the size of a `uint96`, no proper error message will trigger. Consider adding an error message to better indicate the max fee size.

## 9. Inconsistent Trailing Period

At the start of [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol) most comments do not have a trailing `.` ([ex](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L45)) unless followed by another sentence ([ex](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L66)). Consider sticking to the original style. The following cases void the original style:

*/src/CidNFT.sol*

* Followed by a sentence, but no trailing `.`: [L69](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L69), [L162](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L162), [L185](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L185), [L234](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L234), [L235](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L235), [L258](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L258), [L294](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L294), [L306](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L306), [L318](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L318). 
* Single sentence with trailing `.`: [L145](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L145), [L146](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L146), [L161](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L161).

*/src/SubprotocolRegistry.sol*

* Followed by a sentence, but no trailing `.`: [L31](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L31), [L77](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L77), [L78](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L78), [L105](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L105).

*/src/AddressRegistry.sol*

* Followed by a sentence, but no trailing `.`: [L40](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L40), [L61](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L61).