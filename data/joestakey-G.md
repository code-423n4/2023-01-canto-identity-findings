# Gas Report

## Table of Contents
|      | Issue                                                                            |
|------|----------------------------------------------------------------------------------|
| G-01 | Use `storage` instead of `memory` in `SubprotocolRegistry.register`              |
| G-02 | Refactor `CidNFT.remove()`                                                       |
| G-03 | `mint()` can save gas by using `unchecked` blocks                                |




## Summary

> A few optimizations allow to save some gas upon `SubprotocolRegistry.register()`, `CidNFT.mint()` and `CidNFT.remove()` calls, important user-facing functions of the protocol.
> The total gas saved on users function calls is `474`
> The total gas saved on deployment is `92,295`


# [G-01] Use `storage` instead of `memory` in `SubprotocolRegistry.register`

Assigning the data from `subprotocols[_name]` to the memory variable `subprotocolData` causes all fields of the struct `SubprotocolData` to be read from storage, incurring extra cost. Declaring `subprotocolData` with the `storage` keyword will make both function calls and deployment cheaper.

## Code changes

### SubprotocolRegistry.sol

```diff
79:     function register(
80:         bool _ordered,
81:         bool _primary,
82:         bool _active,
83:         address _nftAddress,
84:         string calldata _name,
85:         uint96 _fee
86:     ) external {
87:         SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
88:         if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
-89:         SubprotocolData memory subprotocolData = subprotocols[_name];
+89:         SubprotocolData storage subprotocolData = subprotocols[_name];
90:         if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
91:         subprotocolData.owner = msg.sender;
92:         subprotocolData.fee = _fee;
93:         if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
94:             revert NotASubprotocolNFT(_nftAddress);
95:         subprotocolData.nftAddress = _nftAddress;
96:         subprotocolData.ordered = _ordered;
97:         subprotocolData.primary = _primary;
98:         subprotocolData.active = _active;
-99:         subprotocols[_name] = subprotocolData;
100:         emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
101:     }
```

## Tools Used

Manual Analysis, Foundry


- Gas costs before:


| Contract            | Method            | Avg    | Med    | Max    |
|---------------------|-------------------|--------|--------|--------|
| SubprotocolRegistry | Deployment        |        |        | 437899 |
|---------------------|-------------------|--------|--------|--------|
| SubprotocolRegistry | register          |62499   | 54258  | 85453  |


- Gas costs after:

| Contract            | Method            | Avg    | Med    | Max    |
|---------------------|-------------------|--------|--------|--------|
| SubprotocolRegistry | Deployment        |        |        | 391455 |
|---------------------|-------------------|--------|--------|--------|
| SubprotocolRegistry | register          |62152   | 53421  | 84614  |

This saves on average:
- `46,444` gas upon deployment for `SubprotocolRegistry.sol`
- `347` gas on every `SubprotocolRegistry.register` call

# [G-02] Refactor `CidNFT.remove()`

1 - This function separates the association types in three blocks, and calls `safeTransferFrom` on the `nftToRemove` inside each block. You can save gas on each call and on deployment by refactoring it:
- declare the stack variable `currNFTID` out of the conditional blocks.
- place the `safeTransferFrom` call at the end of the function.

2 - The last `else if` can be replaced by an `else` condition: There is only 3 `associationTypes`, so the last `else if` is equivalent to an `else`

## Code changes

### CidNFT.sol

```diff
237:     function remove(
238:         uint256 _cidNFTID,
239:         string calldata _subprotocolName,
240:         uint256 _key,
241:         uint256 _nftIDToRemove,
242:         AssociationType _type
243:     ) public {
244:         SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
245:             _subprotocolName
246:         );
247:         address subprotocolOwner = subprotocolData.owner;
248:         if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
249:         address cidNFTOwner = ownerOf[_cidNFTID];
250:         if (
251:             cidNFTOwner != msg.sender &&
252:             getApproved[_cidNFTID] != msg.sender &&
253:             !isApprovedForAll[cidNFTOwner][msg.sender]
254:         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
+255:        uint256 currNFTID;
256:         ERC721 nftToRemove = ERC721(subprotocolData.nftAddress);
257:         if (_type == AssociationType.ORDERED) {
258:             // We do not have to check if ordered is supported by the subprotocol. If not, the value will not be unset (which is checked below)
+259:             currNFTID = cidData[_cidNFTID][_subprotocolName].ordered[_key];
-259:             uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].ordered[_key];
260:             if (currNFTID == 0)
261:                 // This check is technically not necessary (because the NFT transfer would fail), but we include it to have more meaningful errors
262:                 revert OrderedValueNotSet(_cidNFTID, _subprotocolName, _key);
263:             delete cidData[_cidNFTID][_subprotocolName].ordered[_key];
-264:             nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
265:             emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);
266:         } else if (_type == AssociationType.PRIMARY) {
+267:             currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
-267:             uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
268:             if (currNFTID == 0) revert PrimaryValueNotSet(_cidNFTID, _subprotocolName);
269:             delete cidData[_cidNFTID][_subprotocolName].primary;
-270:             nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
271:             emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
-272:         } else if (_type == AssociationType.ACTIVE) {
+272:         } else {
273:             IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
274:             uint256 arrayPosition = activeData.positions[_nftIDToRemove]; // Index + 1, 0 if non-existant
275:             if (arrayPosition == 0) revert ActiveArrayDoesNotContainID(_cidNFTID, _subprotocolName, _nftIDToRemove);
276:             uint256 arrayLength = activeData.values.length;
277:             // Swap only necessary if not already the last element
278:             if (arrayPosition != arrayLength) {
279:                 uint256 befSwapLastNFTID = activeData.values[arrayLength - 1];
280:                 activeData.values[arrayPosition - 1] = befSwapLastNFTID;
281:                 activeData.positions[befSwapLastNFTID] = arrayPosition;
282:             }
283:             activeData.values.pop();
284:             activeData.positions[_nftIDToRemove] = 0;
-285:             nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
+285:             currNFTID = _nftIDToRemove;
286:             emit ActiveDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
287:         }
+288:         nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
289:     }
```

## Tools Used

Manual Analysis, Foundry


- Gas costs before:

| Contract            | Method            | Avg    | Med    | Max    |
|---------------------|-------------------|--------|--------|--------|
| CidNFT              | Deployment        |        |        |2032312 |
|---------------------|-------------------|--------|--------|--------|
| CidNFT              | remove            |14082   | 9372   | 26324  |


- Gas costs after:

| Contract            | Method            | Avg    | Med    | Max    |
|---------------------|-------------------|--------|--------|--------|
| CidNFT              | Deployment        |        |        |1986461 |
|---------------------|-------------------|--------|--------|--------|
| CidNFT              | remove            |14017   | 9278   | 26268  |


This saves on average:
- `45,851` gas upon deployment for `CidNFT`
- `65` gas on every `CidNFT.remove` call

# [G-03] `mint()` can save gas by using `unchecked` blocks

The call to `ERC721_mint()` can be placed in an unchecked block, to save gas on the `++numMinted` operation.
For the same reason, you can place the i increment in an unchecked block, gas can be saved on every iteration

## Code changes

### CidNFT.sol

```diff
147: function mint(bytes[] calldata _addList) external {
+            unchecked {
148:         _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
+            }
149:         bytes4 addSelector = this.add.selector;
-150:         for (uint256 i = 0; i < _addList.length; ++i) {
+150:         for (uint256 i = 0; i < _addList.length;) {
151:             (
152:                 bool success, /*bytes memory result*/
153: 
154:             ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
155:             if (!success) revert AddCallAfterMintingFailed(i);
+                unchecked {
+                  ++i;
+                }
156:        }
157:     }
```

## Tools Used

Manual Analysis, Foundry


- Gas costs before:

| Contract            | Method            | Avg    | Med    | Max    |
|---------------------|-------------------|--------|--------|--------|
| CidNFT              | mint              |52243   | 65266  | 155094 |


- Gas costs after:

| Contract            | Method            | Avg    | Med    | Max    |
|---------------------|-------------------|--------|--------|--------|
| CidNFT              | mint              |52181   | 65209  | 154897 |


This saves on average:
- `62` gas on every `CidNFT.mint()` call
