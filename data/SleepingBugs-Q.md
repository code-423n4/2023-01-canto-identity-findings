

# Low

| | Issue index |
| ----------- | ----------- |
| 1 | [Missing checks for address(0x0) when assigning values to `address` state or `immutable` variables](#missing-checks-for-address(0x0)-when-assigning-values-to-`address`-state-or-`immutable`-variables) |
## Missing checks for address(0x0) when assigning values to `address` state or `immutable` variables 

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/Picodes/a6ca8ab593a9d1fbfc322815bec08069)

### Summary

Zero address should be checked for state variables, immutable variables. A zero address can lead into problems.

### Code Snippet

0 address control should be done in these parts
 
[CidNFT.sol#L129-L130](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/CidNFT.sol#L129-L130

[SubprotocolRegistry.sol#L66](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/SubprotocolRegistry.sol#L66

### Recommendation

Check zero address before assigning or using it


# Informational

| | Issue index |
| ----------- | ----------- |
| 1 | [Maximum line length exceeded](#maximum-line-length-exceeded) |
| 2 | [Constants should be defined rather than using magic numbers](#constants-should-be-defined-rather-than-using-magic-numbers) |
| 3 | [Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10 ** 18`)](#use-scientific-notation-(e.g.-`1e18`)-rather-than-exponentiation-(e.g.-`10-**-18`)) |
| 4 | [Missing Natspec](#missing-natspec) |
| 5 | [Undocumented parameters](#undocumented-parameters) |
| 6 | [Include return parameters in natspec comments](#include-return-parameters-in-natspec-comments) |
## Maximum line length exceeded

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Solidity newer guidelines suggest 120 characters. 
Reference: Long lines should be wrapped to conform with Solidity Style [guidelines](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length)

Following lines with more than 120:

[CidNFT.sol#L144](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L144)

[CidNFT.sol#L146](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L146)

[CidNFT.sol#L148](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L148)

[CidNFT.sol#L162](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L162)

[CidNFT.sol#L185](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L185)

[CidNFT.sol#L258](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L258)

[CidNFT.sol#L261](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L261)

[CidNFT.sol#L327](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L327)

[SubprotocolRegistry.sol#L71](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L71)

[SubprotocolRegistry.sol#L72](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L72)

[SubprotocolRegistry.sol#L78](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L78)

## Constants should be defined rather than using magic numbers

`NOTE`: None of these findings where found by [4naly3er output - NC](https://gist.github.com/Picodes/a6ca8ab593a9d1fbfc322815bec08069)

- [CidNFT.sol#L191](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L191)
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;

## Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10 ** 18`)

- [SubprotocolRegistry.sol#L17](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L17)
    uint256 public constant REGISTER_FEE = 100 * 10**18;

## Missing Natspec 

NatSpec is missing for the following functions / constructor / modifiers:

- [CidNFT.sol#L339-L346](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/CidNFT.sol#L339-L346)
function onERC721Received(

## Undocumented parameters

In a some places, a parameter is missing in the documentation:

- [CidNFT.sol#L75-L110](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/CidNFT.sol#L75-L110)

- [CidNFT.sol#L339-L346](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/CidNFT.sol#L339-L346)

- [AddressRegistry.sol#L26-L33](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/AddressRegistry.sol#L26-L33)

- [AddressRegistry.sol#L40-L49](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/AddressRegistry.sol#L40-L49)

- [SubprotocolRegistry.sol#L45-L60](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/SubprotocolRegistry.sol#L45-L60)


## Include return parameters in natspec comments

If Return parameters are declared, you must prefix them with ”/// @return”. [References](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html)

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the `@return` tag, they do incomplete analysis.

### Recommendation

Include return parameters in NatSpec comments

Recommendation Code Style:

```solidity
    /// @notice information about what a fooFighter function does
		/// @param fooParam what the fooParam represents
		/// @return what is the fooReturnValue returned by fooFighter
		function fooFighter(uint256 fooParam) public returns (uint256 fooReturnValue) {
      ...
    }
```

- [CidNFT.sol#L339-L346](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/CidNFT.sol#L339-L346)
function onERC721Received(
