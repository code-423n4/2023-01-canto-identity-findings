
------------

| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | Increments can be unchecked | 1 | 40
| [G-02] | Usage of uints or ints smaller than 32 bytes incurs overhead | 2 | 56

----------

## G-01 Increments can be unchecked

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used. It saves **30-40 gas** per loop.

_There is 1 instance of this issue_

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

```
File: src/CidNFT.sol

150: for (uint256 i = 0; i < _addList.length; ++i) {
```

-------

## G-02 Usage of uints or ints smaller than 32 bytes incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

_There are 2 instances of this issue:_

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
File: src/SubprotocolRegistry.sol

52: uint96 fee
85: uint96 _fee
```

----------
