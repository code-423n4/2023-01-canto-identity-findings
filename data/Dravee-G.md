**Overview**

Risk Rating | Number of issues
--- | ---
Gas Issues | 6

**Table of Contents:**

- [1. Multiple accesses of a mapping/array should use a local variable cache](#1-multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
- [2. Using `storage` instead of `memory`](#2-using-storage-instead-of-memory)
- [3. Revert conditions that check input arguments should be at the top of the function](#3-revert-conditions-that-check-input-arguments-should-be-at-the-top-of-the-function)
- [4. Unchecking arithmetics operations that can't underflow/overflow](#4-unchecking-arithmetics-operations-that-cant-underflowoverflow)
- [5. `<array>.length` should not be looked up in every loop of a `for-loop`](#5-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)
- [6. Upgrade pragma](#6-upgrade-pragma)

## 1. Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed multiple times saves **~42 gas per access** due to not having to perform the same offset calculation every time.

As an example here, `cidData[_cidNFTID][_subprotocolName]` should've been replaced with `SubprotocolData storage _subProtocol = cidData[_cidNFTID][_subprotocolName];` (declared even before the `if/else if` statements):

```solidity
src/CidNFT.sol:
  197:             if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {   
  201:             cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;  
  205:             if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
  209:             cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
  213:             IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
  259:             uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].ordered[_key];
  263:             delete cidData[_cidNFTID][_subprotocolName].ordered[_key]; 
  267:             uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
  269:             delete cidData[_cidNFTID][_subprotocolName].primary;
  273:             IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
```

```diff
20,24d42
- [PASS] testIntegrationCaseOne() (gas: 381096)
- [PASS] testIntegrationCaseTwo() (gas: 395078)
11a30,34
+ [PASS] testIntegrationCaseOne() (gas: 380396)
+ [PASS] testIntegrationCaseTwo() (gas: 394548)

30,38c48,56
- [PASS] testAddRemoveByApprovedAccount() (gas: 227997)
- [PASS] testAddRemoveByApprovedAllAccount() (gas: 229891)
- [PASS] testAddRemoveByOwner() (gas: 207006)
- [PASS] testAddRemoveByUnauthorizedAccount() (gas: 244213)
- [PASS] testAddRemoveOrderedType() (gas: 206148)
- [PASS] testAddRemovePrimaryType() (gas: 205184)
- [PASS] testAddWithFee() (gas: 335150)
---
+ [PASS] testAddRemoveByApprovedAccount() (gas: 227646)
+ [PASS] testAddRemoveByApprovedAllAccount() (gas: 229540)
+ [PASS] testAddRemoveByOwner() (gas: 206655)
+ [PASS] testAddRemoveByUnauthorizedAccount() (gas: 243981)
+ [PASS] testAddRemoveOrderedType() (gas: 205796)
+ [PASS] testAddRemovePrimaryType() (gas: 204825)
+ [PASS] testAddWithFee() (gas: 334918)
40,44c58,62
- [PASS] testMintWithMultiAddItems() (gas: 401911)
- [PASS] testMintWithMultiAddItemsAndRevert() (gas: 264495)
- [PASS] testMintWithSingleAddList() (gas: 204003)
---
+ [PASS] testMintWithMultiAddItems() (gas: 401212)
+ [PASS] testMintWithMultiAddItemsAndRevert() (gas: 264263)
+ [PASS] testMintWithSingleAddList() (gas: 203771)
46,48c64,66
- [PASS] testOverWritingPrimary() (gas: 279001)
- [PASS] testOverwritingOrdered() (gas: 281662)
---
+ [PASS] testOverWritingPrimary() (gas: 278318)
+ [PASS] testOverwritingOrdered() (gas: 280991)
65c83
- | 2032312                        | 11180           |       |        |        |         |
---
+ | 1961833                        | 10828           |       |        |        |         |
69c87
- | add                            | 8378            | 50503 | 52935  | 129892 | 71      |
---
+ | add                            | 8571            | 50448 | 52749  | 129900 | 71      |
74c92
- | mint                           | 23466           | 51941 | 65266  | 155288 | 55      |
---
+ | mint                           | 23466           | 51923 | 65266  | 154729 | 55      |
78c96
- | remove                         | 6367            | 14082 | 9372   | 26324  | 26      |
---
+ | remove                         | 6367            | 14033 | 9374   | 26158  | 26      |
96c114
- | 11503111                                  | 57811           |     |        |     |         |
---
+ | 11432568                                  | 57459           |     |        |     |         |
```

## 2. Using `storage` instead of `memory`

While the use of slot packing can seem attractive and writing into memory before transposing the change in storage can seem like a good idea: it can often lead to losing the advantage of Slot packing.
Writing to a storage struct, done without any middle interruptions (conditions), actually takes advantage of Slot-packing for writing.
Therefore, here, consider using `storage` instead of `memory`:

```diff
File: SubprotocolRegistry.sol
- 89:         SubprotocolData memory subprotocolData = subprotocols[_name];
+ 89:         SubprotocolData storage subprotocolData = subprotocols[_name];
- 99:         subprotocols[_name] = subprotocolData;
```

`forge test --gas-report` gives the following savings, which are quite significant:

```diff
- [PASS] testRegisterDifferentAssociation() (gas: 1789239)
+ [PASS] testRegisterDifferentAssociation() (gas: 1785883)
- [PASS] testRegisterExistedProtocol() (gas: 863201)
+ [PASS] testRegisterExistedProtocol() (gas: 862093)
- [PASS] testReturnedDataMatchSubprotocol() (gas: 858501)
+ [PASS] testReturnedDataMatchSubprotocol() (gas: 857662)
- [PASS] testAddUnsupportedAssociationType() (gas: 2479112)
+ [PASS] testAddUnsupportedAssociationType() (gas: 2473239)
- [PASS] testAddWithFee() (gas: 335150)
+ [PASS] testAddWithFee() (gas: 334311)
- [PASS] testAddWithNotEnoughFee() (gas: 261628)
+ [PASS] testAddWithNotEnoughFee() (gas: 260789)

| src/SubprotocolRegistry.sol:SubprotocolRegistry contract |                 |       |        |       |         |
|----------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                          | Deployment Size |       |        |       |         |
- | 437899                                                   | 2447            |       |        |       |         |
---
+ | 391455                                                   | 2215            |       |        |       |         |

| src/SubprotocolRegistry.sol:SubprotocolRegistry contract |                 |       |        |       |         |
- | register                                                 | 5690            | 63110 | 54258  | 85453 | 74      |
---
+ | register                                                 | 5421            | 62540 | 53421  | 84614 | 74      |

| src/test/CidNFT.t.sol:CidNFTTest contract |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
- | 11503111                                  | 57811           |     |        |     |         |
---
+ | 11456607                                  | 57579           |     |        |     |         |

```

## 3. Revert conditions that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting storage operation costs in a function that may ultimately revert in the unhappy case.

```diff
File: CidNFT.sol
165:     function add(
166:         uint256 _cidNFTID,
167:         string calldata _subprotocolName,
168:         uint256 _key,
169:         uint256 _nftIDToAdd,
170:         AssociationType _type
171:     ) external { 
+ 172:       if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols
172:         SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
173:             _subprotocolName
174:         ); 
175:         address subprotocolOwner = subprotocolData.owner;
176:         if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
177:         address cidNFTOwner = ownerOf[_cidNFTID];
178:         if (
179:             cidNFTOwner != msg.sender &&
180:             getApproved[_cidNFTID] != msg.sender &&
181:             !isApprovedForAll[cidNFTOwner][msg.sender]
182:         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner); 
- 183:         if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols 
```

## 4. Unchecking arithmetics operations that can't underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block: <https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic>

Consider wrapping with an `unchecked` block here (around **25 gas saved** per instance, see `@notes`):

```solidity
CidNFT.sol:148:        _mint(msg.sender, ++numMinted); //@note can't realistically overflow
CidNFT.sol:150:        for (uint256 i = 0; i < _addList.length; ++i) { //@note classic unchecked on loops
CidNFT.sol:193:            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee); //@note can't underflow due to cidFee being a fraction of subprotocolOwner
CidNFT.sol:279:                uint256 befSwapLastNFTID = activeData.values[arrayLength - 1]; //@note if there's a position, there's at least one value, so this can't underflow
CidNFT.sol:280:                activeData.values[arrayPosition - 1] = befSwapLastNFTID; //@note can't underflow, position is at least 1
```

## 5. `<array>.length` should not be looked up in every loop of a `for-loop`

Reading an array's length at each iteration of a loop consumes more gas than necessary.
  
In the best case scenario (length read on a memory variable), caching the array length in the stack saves around **3 gas** per iteration.
In the worst case scenario (external calls at each iteration), the amount of gas wasted can be massive.

Here, consider caching the array's length in a variable before the for-loop, and use this new variable instead:

```solidity
CidNFT.sol:150:        for (uint256 i = 0; i < _addList.length; ++i) {
```

## 6. Upgrade pragma

Using newer compiler versions and the optimizer give gas optimizations. Also, additional safety checks are available for free.

The advantages here are:

- **Low level inliner** (>= 0.8.2): Cheaper runtime gas (especially relevant when the contract has small functions).
- **Optimizer improvements in packed structs** (>= 0.8.3)
- **Custom errors** (>= 0.8.4): cheaper deployment cost and runtime cost. *Note*: the runtime cost is only relevant when the revert condition is met. In short, replace revert strings by custom errors.
- **Contract existence checks** (>= 0.8.10): external calls skip contract existence checks if the external call has a return value

Consider upgrading here :

```solidity
AddressRegistry.sol:2:pragma solidity >=0.8.0;
CidNFT.sol:2:pragma solidity >=0.8.0;
CidSubprotocolNFT.sol:2:pragma solidity >=0.8.0;
SubprotocolRegistry.sol:2:pragma solidity >=0.8.0;
```
