



| S No. | Issue | Instances | Gas Savings |
|-----|-----|-----|-----|
| [G-01] | ++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS | 1 | 40
| [G-02] | USAGE OF UINTS or INTS SMALLER THAN 32 BYTES - 256 BITS INCURS OVERHEAD | 2 | 56


### G-01 ++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

*Number of Instances Identified: 1*

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

```
150: for (uint256 i = 0; i < _addList.length; ++i) {
```


### G-02 USAGE OF UINTS or INTS SMALLER THAN 32 BYTES - 256 BITS INCURS OVERHEAD

*Number of Instances Identified: 2*

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.4

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a `uint8` costs an extra [**22-28 gas**]([https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977)) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol

```
52: uint96 fee
85: uint96 _fee
```
