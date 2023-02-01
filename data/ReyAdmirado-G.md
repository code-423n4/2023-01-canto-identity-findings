
| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#) |
| 2 | [Stack variable used as a cheaper cache for a state variable is only used once](#) |
| 3 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#) |
| 4 | [can make the variable outside the loop to save gas](#) |
| 5 | [`++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#) |
| 6 | [use a more recent version of solidity](#) |
| 7 | [using `bool` for storage incurs overhead](#) |
| 8 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#) |
| 9 | [Ternary over if ... else](#) |
| 10 | [public functions not called by the contract should be declared external instead](#) |
| 11 | [Setting the constructor to payable](#) |
| 12 | [instead of assigning to zero use `delete`](#) |
| 13 | [can precalculate part of code](#) |
| 14 | [do the cheaper checks first](#) |


## 1. State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas).

`baseURI`
- [CidNFT.sol#L37](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L37)


## 2. Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

`subprotocolOwner`
- [CidNFT.sol#L247](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L247)


## 3. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

`subprotocolFee - cidFee` cant be 0 because `cidFee` is one tenth of `subprotocolFee` and we checked `subprotocolFee` not to be 0 before. unchecked {} these 3 lines because they cant over or underflow
- [CidNFT.sol#L191-L193](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191-L193)


## 4. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

`success` 
- [CidNFT.sol#L152](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L152)


## 5. `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

This saves 30-40 gas PER LOOP

- [CidNFT.sol#L150](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150)


## 6. use a more recent version of solidity

use a specific version for the contract

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have `external` calls skip contract existence checks if the external call has a return value
Use a solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>)
Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions
Use a solidity version of at least 0.8.15 because the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
Use a solidity version of at least 0.8.17 to  get prevention of the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.


## 7. using `bool` for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past

- [SubprotocolRegistry.sol#L34](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L34)
- [SubprotocolRegistry.sol#L35](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L35)
- [SubprotocolRegistry.sol#L36](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L36)


## 8. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

just make `subprotocolFee` as 256 uint256 and let the other bits be 0
- [CidNFT.sol#L189](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L189)


## 9. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

instead of the L215 if statement and L220 else we can use Ternary operator
- [CidNFT.sol#L215-226](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L215-226)


## 10. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`tokenURI`
- [CidNFT.sol#L136](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L136)


## 11. Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

- [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)
- [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)
- [AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)


## 12. instead of assigning to zero use `delete` 

assigning to zero uses more gas than using `delete` , and they both assign variable to default value. so it is encouraged if the data is no longer needed use delete instead.

- [CidNFT.sol#L284](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L284)


## 13. can precalculate part of code

here there is a possibility to precalculate a operation which will reduce use of 1 operation in the function. instead of doing 2 operations every time function runs it will be 1 instead.

the `CID_FEE_BPS/10_000` in  `(subprotocolFee * CID_FEE_BPS) / 10_000` can be pre calculated
- [CidNFT.sol#L191](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191)


## 14. do the cheaper checks first 

these if statements can be sorted better so if the revert should happen for the cheaper ones they should be done first. this is gives us the chance to save gas.

the L178 if statement and L183 if statement should switch places
- [CidNFT.sol#L178-L183](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L178-L183)
