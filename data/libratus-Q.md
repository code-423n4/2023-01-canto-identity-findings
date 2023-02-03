## L-01 Make CidNFT external functions non-reentrant

It is currently possible to reenter `remove` function as well as `mint` and `add` (because `remove` can be called from them). This is because `remove()` uses `ERC721.safeTransferFrom` which can invoke the callback on the receiver side.

While there were no possible attacks spotted at the moment, even minor code changes can introduce issues related to reentrancy.

Recommendation is to apply reentrancy guard modifiers to these functions.

## NC-01 Active subprotocols are a subset of Ordered. Their value can be reconsidered

In order to simplify the contract it may be viable to remove `Active` subprotocol type. `Ordered` can be used instead as it's also capable of holding multiple entries. The only downside is that clients would have to manage keys when adding/removing elements themselves, possibly relying on emitted events.

## NC-02 Return id when minting an identity

A little QoL change could be to return the id of minted identity from `mint` function like the following:
```diff
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..79bf848 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -144,8 +144,9 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
     /// @dev An address can mint multiple CID NFTs, but it can only set one as associated with it in the AddressRegistry
     /// @param _addList An optional list of encoded parameters for add to add subprotocol NFTs directly after minting.
     /// The parameters should not include the function selector itself, the function select for add is always prepended.
-    function mint(bytes[] calldata _addList) external {
-        _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
+    function mint(bytes[] calldata _addList) external returns (uint256) {
+        uint256 id = ++numMinted;
+        _mint(msg.sender, id); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
         bytes4 addSelector = this.add.selector;
         for (uint256 i = 0; i < _addList.length; ++i) {
             (
@@ -154,6 +155,7 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
             ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
             if (!success) revert AddCallAfterMintingFailed(i);
         }
+        return id;
     }
```

## NC-03 Code can be simplified when adding subprotocol entires of type Active 

There is no need to check whether there are any existing entries of a subprotocol. The following condition can be removed:
```diff
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..79bf848 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -212,18 +214,11 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
             if (!subprotocolData.active) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
             IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
             uint256 lengthBeforeAddition = activeData.values.length;
-            if (lengthBeforeAddition == 0) {
-                uint256[] memory nftIDsToAdd = new uint256[](1);
-                nftIDsToAdd[0] = _nftIDToAdd;
-                activeData.values = nftIDsToAdd;
-                activeData.positions[_nftIDToAdd] = 1; // Array index + 1
-            } else {
-                // Check for duplicates
-                if (activeData.positions[_nftIDToAdd] != 0)
-                    revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
-                activeData.values.push(_nftIDToAdd);
-                activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
-            }
+
+            if (activeData.positions[_nftIDToAdd] != 0)
+                revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
+            activeData.values.push(_nftIDToAdd);
+            activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
             emit ActiveDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd, lengthBeforeAddition);
         }
     }

```

## NC-04 Events have obsolete values

When removing subprotocol entries, parameter `_nftIDToRemove` only matters if working with type Active. However, it is included in the events for Ordered and Primary types.

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L265

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L271

It is pointless to include `_nftIDToRemove` in these events