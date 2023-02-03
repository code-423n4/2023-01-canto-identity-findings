### Table of Contents
- [FINDINGS](#findings)
- [Using immutable on variables that are only set in the constructor and never after](#using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after)
- [Using storage instead of memory for structs/arrays saves gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)
  - [Use storage instead of memory(saves 570 gas on average)](#use-storage-instead-of-memorysaves-570-gas-on-average)
- [require() or revert() statements that check input arguments should be at the top of the function](#require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [Reorder the if's to fail cheaply incase of a revert](#reorder-the-ifs-to-fail-cheaply-incase-of-a-revert)
  - [Reorder the if's to have cheaper checks before](#reorder-the-ifs-to-have-cheaper-checks-before)
- [Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate)
  - [CidNFT.sol.add(): cidData\[\_cidNFTID\]\[\_subprotocolName\] should be cached in local storage](#cidnftsoladd-ciddata_cidnftid_subprotocolname-should-be-cached-in-local-storage)
- [Avoid contract existence checks by using solidity version 0.8.10 or later](#avoid-contract-existence-checks-by-using-solidity-version-0810-or-later)
- [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)](#using-unchecked-blocks-to-save-gas---increments-in-for-loop-can-be-unchecked---save-30-40-gas-per-loop-iteration)
- [Use a more recent version of solidity](#use-a-more-recent-version-of-solidity)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

## Using immutable on variables that are only set in the constructor and never after 
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

For short strings you can simulate immutable via a function that returns the value or you can store into a bytes32 for the majority of short strings.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L37
```solidity
File: /src/CidNFT.sol
37:    string public baseURI;
```

## Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L89
### Use storage instead of memory(saves 570 gas on average)
**Gas benchmarks**
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 5690    | 63110   | 54258 | 85453 |
| After  | 5421    | 62540   | 53421 | 84614 |

```solidity
File: /src/SubprotocolRegistry.sol
89:        SubprotocolData memory subprotocolData = subprotocols[_name];
```

```diff
diff --git a/src/SubprotocolRegistry.sol b/src/SubprotocolRegistry.sol
index 579f579..779d976 100644
--- a/src/SubprotocolRegistry.sol
+++ b/src/SubprotocolRegistry.sol
@@ -86,7 +86,7 @@ contract SubprotocolRegistry {
     ) external {
         SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
         if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
-        SubprotocolData memory subprotocolData = subprotocols[_name];
+        SubprotocolData storage subprotocolData = subprotocols[_name];
         if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
         subprotocolData.owner = msg.sender;
         subprotocolData.fee = _fee;
@@ -96,7 +96,6 @@ contract SubprotocolRegistry {
         subprotocolData.ordered = _ordered;
         subprotocolData.primary = _primary;
         subprotocolData.active = _active;
-        subprotocols[_name] = subprotocolData;
         emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
     }
```
Note that after changing to storage  we no longer need the [Line 99](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L99)
```solidity
99:        subprotocols[_name] = subprotocolData;
```
This is because we are assigning to storage directly and not just a copy

##  require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

**I've made some notes before the diffs explaining the how and why**

### Reorder the if's to fail cheaply incase of a revert

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L165-L229
```solidity
File: /src/CidNFT.sol
165:    function add(
166:        uint256 _cidNFTID,
167:        string calldata _subprotocolName,
168:        uint256 _key,
169:        uint256 _nftIDToAdd,
170:        AssociationType _type
171:    ) external {
172:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
173:            _subprotocolName
174:        );
175:        address subprotocolOwner = subprotocolData.owner;
176:        if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
177:        address cidNFTOwner = ownerOf[_cidNFTID];
178:        if (
179:            cidNFTOwner != msg.sender &&
180:            getApproved[_cidNFTID] != msg.sender &&
181:            !isApprovedForAll[cidNFTOwner][msg.sender]
182:        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
183:        if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols
```
Since `_nftIDToAdd` is a function input, we can check it first then revert if it does't meet the desired condition. This way we avoid wasting gas doing the other operation if we ultimately revert due to a function parameter not being valid

```diff
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..14688e3 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -169,6 +169,7 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
         uint256 _nftIDToAdd,
         AssociationType _type
     ) external {
+        if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols
         SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
             _subprotocolName
         );
@@ -180,7 +181,7 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
             getApproved[_cidNFTID] != msg.sender &&
             !isApprovedForAll[cidNFTOwner][msg.sender]
         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
-        if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols
```

### Reorder the if's to have cheaper checks before
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L237-L255
```solidity
File: /src/CidNFT.sol
237:    function remove(
238:        uint256 _cidNFTID,
239:        string calldata _subprotocolName,
240:        uint256 _key,
241:        uint256 _nftIDToRemove,
242:        AssociationType _type
243:    ) public {
244:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
245:            _subprotocolName
246:        );
247:        address subprotocolOwner = subprotocolData.owner;
248:        if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
249:        address cidNFTOwner = ownerOf[_cidNFTID];
250:        if (
251:            cidNFTOwner != msg.sender &&
252:            getApproved[_cidNFTID] != msg.sender &&
253:            !isApprovedForAll[cidNFTOwner][msg.sender]
254:        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```

We have an external call on [Line 244](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L244-L246) which is a very expensive call to make.
```solidity
//@audit: external call --> getSubprotocol(_subprotocolName );
244:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
245:            _subprotocolName
246:        );
```

If we make this call and then we happen to revert on [Line 250](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L250) we would end up wasting too much gas on that external call. It would be way cheaper to have the cheap checks come first thus if we revert on the following, we would not waste gas on the external function call.
```solidity
249:        address cidNFTOwner = ownerOf[_cidNFTID];
250:        if (
251:            cidNFTOwner != msg.sender &&
252:            getApproved[_cidNFTID] != msg.sender &&
253:            !isApprovedForAll[cidNFTOwner][msg.sender]
254:        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```

We could save gas by refactoring as follows. 

```diff
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..ece8cdd 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -241,11 +241,7 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
         uint256 _nftIDToRemove,
         AssociationType _type
     ) public {
-        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
-            _subprotocolName
-        );
-        address subprotocolOwner = subprotocolData.owner;
-        if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
+
         address cidNFTOwner = ownerOf[_cidNFTID];
         if (
             cidNFTOwner != msg.sender &&
@@ -253,6 +249,12 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
             !isApprovedForAll[cidNFTOwner][msg.sender]
         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);

+        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
+            _subprotocolName
+        );
+        address subprotocolOwner = subprotocolData.owner;
+        if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
+
```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L195-L214
### CidNFT.sol.add(): cidData\[\_cidNFTID]\[\_subprotocolName] should be cached in local storage
```solidity
File: /src/CidNFT.sol
195:        if (_type == AssociationType.ORDERED) {

197:            if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {

200:            }
201:            cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;

205:            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {

208:            }
209:            cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;

213:            IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
```

## Avoid contract existence checks by using solidity version 0.8.10 or later

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L172-L174
```solidity
File: /src/CidNFT.sol

//@audit: getSubprotocol(_subprotocolName);
172:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
173:            _subprotocolName
174:        );

//@audit: getSubprotocol(_subprotocolName);
244:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
245:            _subprotocolName
246:        );
```

## Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L79-L86
```solidity
File: /src/SubprotocolRegistry.sol

//@audit:  uint96 _fee
79:    function register(
80:        bool _ordered,
81:        bool _primary,
82:        bool _active,
83:        address _nftAddress,
84:        string calldata _name,
85:        uint96 _fee
86:    ) external {
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L280
```solidity
File: /src/CidNFT.sol
280:                activeData.values[arrayPosition - 1] = befSwapLastNFTID;
```

The operation `arrayPosition - 1` cannot underflow due to the check on [Line 275](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L275) that ensures that the value of `arrayPosition` is not equal to `0`. This means when perfoming the above operation , the least value `arrayPosition` could hold is `1` thus we can never underflow when subtracting by 1, worst case would result in `0`

## Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)
The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.

e.g Let's work with a sample loop below.

```solidity
for(uint256 i; i < 10; i++){
//doSomething
}

```
can be written as shown below.
```solidity
for(uint256 i; i < 10;) {
  // loop logic
  unchecked { i++; }
}
```

We can also write  it as an inlined function like below.

```solidity
function inc(i) internal pure returns (uint256) {
  unchecked { return i + 1; }
}
for(uint256 i; i < 10; i = inc(i)) {
  // doSomething
}
```

**Affected code**
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L150-L156
```solidity
File: /src/CidNFT.sol
150:        for (uint256 i = 0; i < _addList.length; ++i) {
```

The above should be modified to:
```diff
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..175dc32 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -147,12 +147,15 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
     function mint(bytes[] calldata _addList) external {
         _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
         bytes4 addSelector = this.add.selector;
-        for (uint256 i = 0; i < _addList.length; ++i) {
+        for (uint256 i = 0; i < _addList.length;) {
             (
                 bool success, /*bytes memory result*/

             ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
             if (!success) revert AddCallAfterMintingFailed(i);
+            unchecked {
+                ++i;
+            }
         }
     }
```

## Use a more recent version of solidity

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L2
```solidity
File: /src/SubprotocolRegistry.sol
2:pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L2
```solidity
File: /src/AddressRegistry.sol
2:pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L2
```solidity
File: /src/CidNFT.sol
2:pragma solidity >=0.8.0;
```