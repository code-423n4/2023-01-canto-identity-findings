# Gas Optimizations Report
[Excluded findings](#excluded-findings) are below the automated findings.

## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 4 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 2 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables rather than re-reading them from storage | 1 |
| [GAS-4](#GAS-4) | Use calldata instead of memory for function arguments that do not get mutated | 3 |
| [GAS-5](#GAS-5) | Don't initialize variables with default value | 1 |
| [GAS-6](#GAS-6) | Using `private` rather than `public` for constants, saves gas | 2 |
| [GAS-7](#GAS-7) | Use != 0 instead of > 0 for unsigned integer comparison | 3 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (4)*:
```solidity
File: src/CidNFT.sol

137:         if (ownerOf[_id] == address(0))

176:         if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);

248:         if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol

90:         if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol)

### <a name="GAS-2"></a>[GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (2)*:
```solidity
File: src/CidNFT.sol

150:         for (uint256 i = 0; i < _addList.length; ++i) {

214:             uint256 lengthBeforeAddition = activeData.values.length;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (1)*:
```solidity
File: src/CidNFT.sol

193:             SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

### <a name="GAS-4"></a>[GAS-4] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (3)*:
```solidity
File: src/CidNFT.sol

120:         string memory _name,

121:         string memory _symbol,

122:         string memory _baseURI,

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

### <a name="GAS-5"></a>[GAS-5] Don't initialize variables with default value

*Instances (1)*:
```solidity
File: src/CidNFT.sol

150:         for (uint256 i = 0; i < _addList.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

### <a name="GAS-6"></a>[GAS-6] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (2)*:
```solidity
File: src/CidNFT.sol

17:     uint256 public constant CID_FEE_BPS = 1_000;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol

17:     uint256 public constant REGISTER_FEE = 100 * 10**18;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol)

### <a name="GAS-7"></a>[GAS-7] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (3)*:
```solidity
File: src/AddressRegistry.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol)

```solidity
File: src/CidNFT.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol)

 --- 
## Excluded Findings

| No. | Issue |
| --- | --- |
| 1 | [Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#G-01-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead)|
| 2 | [++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#G-02-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops)|
| 3 | [Optimize names to save gas [22 gas per instance]](#G-03-optimize-names-to-save-gas-22-gas-per-instance)|
| 4 | [Setting the `constructor` to `payable` [~13 gas per instance]](#G-04-setting-the-constructor-to-payable-13-gas-per-instance)|
| 5 | [Use a more recent version of solidity](#G-05-use-a-more-recent-version-of-solidity)|
| 6 | [`<array>.length` should not be looked up in every loop of a `for`-loop](#G-06-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)|
---
### [G-01] Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

---
**Description:**

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so.

**Recommendation:**

Use a larger size then downcast where needed

**Lines of Code:** 

- [CidNFT.sol#L189](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L189)
- [SubprotocolRegistry.sol#L32](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L32)
- [SubprotocolRegistry.sol#L52](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L52)
- [SubprotocolRegistry.sol#L85](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L85)

---
### [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

---
**Description:**

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

**Recommendation:**

Using Solidity's unchecked block saves the overflow checks.

**Proof Of Concept:**

<https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---unnecessary-checked-arithmetic-in-for-loop>

**Lines of Code:** 

- [CidNFT.sol#L150](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L150)

---
### [G-03] Optimize names to save gas [22 gas per instance]

---
**Description:**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Recommendation:**

Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas. For example, the function IDs in the L1GraphTokenGateway.sol contract will be the most used; A lower method ID may be given.

**Proof Of Concept:**

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

**Lines of Code:** 

- [AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol)
- [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)
- [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol)

---
### [G-04] Setting the `constructor` to `payable` [~13 gas per instance]

---
**Lines of Code:** 

- [AddressRegistry.sol#L36](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L36)
- [CidNFT.sol#L119](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L119)
- [SubprotocolRegistry.sol#L65](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L65)

---
### [G-05] Use a more recent version of solidity

---
**Description:**

Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value.

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

**Lines of Code:** 

- [AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol)
- [CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol)
- [SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol)

---
### [G-06] `<array>.length` should not be looked up in every loop of a `for`-loop

---
**Description:**

The overheads outlined below are *PER LOOP*, excluding the first loop

 - storage arrays incur a Gwarmaccess (100 gas) 
 - memory arrays use `MLOAD` (3 gas) 
 - calldata arrays use `CALLDATALOAD` (3 gas) 

Caching the length changes each of these to a `DUP<N>` (3 gas), and gets rid of the extra `DUP<N>` needed to store the stack offset


**Lines of Code:** 

- [CidNFT.sol#L150](https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L150)