## 01. Wrong parameter is provided for event OrderedDataRemoved and PrimaryDataRemoved
The emitted events [OrderedDataRemoved](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L265) and [PrimaryDataRemoved](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L271) use a wrong subprotocolNFTID - `_nftIDToRemove`:
```
emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);
emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
```

The `_nftIDToRemove` is only used for `AssociationType.ACTIVE`. Both the two `_nftIDToRemove` should be replaced with `currNFTID`:
```
emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, currNFTID);
emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, currNFTID);
```

## 02. Should check isActive() for the subprotocol NFT ID in CidNFT.add()
We should disallow adding inactive subprotocol NFTS in [CidNFT.add()](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L165).

Mitigation sample:
```
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..3f8757a 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -5,6 +5,7 @@ import {ERC721, ERC721TokenReceiver} from "solmate/tokens/ERC721.sol";
 import "solmate/tokens/ERC20.sol";
 import "solmate/utils/SafeTransferLib.sol";
 import "./SubprotocolRegistry.sol";
+import "./CidSubprotocolNFT.sol";

 /// @title Canto Identity Protocol NFT
 /// @notice CID NFTs are at the heart of the CID protocol. All key/values of subprotocols are associated with them.
@@ -182,6 +183,7 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
         ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
         if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols

+        require(CidSubprotocolNFT(subprotocolData.nftAddress).isActive(_nftIDToAdd), "not active");
         // The CID Protocol safeguards the NFTs of subprotocols. Note that these NFTs are usually pointers to other data / NFTs (e.g., to an image NFT for profile pictures)
         ERC721 nftToAdd = ERC721(subprotocolData.nftAddress);
         nftToAdd.safeTransferFrom(msg.sender, address(this), _nftIDToAdd);
```

## 03. Lack of event CIDNFTRemoved when AddressRegistry.register() overwrites a previously registered ID
In [AddressRegistry.register()](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L42):
* If the same ID has already been registered before, nothing should be done
* If a different ID has been registered before, event CIDNFTRemoved should be thrown, otherwise event CIDNFTRemoved is incomplete.

Mitigation sample:
```
diff --git a/src/AddressRegistry.sol b/src/AddressRegistry.sol
index 82bdd5e..3c8a36e 100644
--- a/src/AddressRegistry.sol
+++ b/src/AddressRegistry.sol
@@ -44,6 +44,13 @@ contract AddressRegistry {
             // We only guarantee that a CID NFT is owned by the user at the time of registration
             // ownerOf reverts if non-existing ID is provided
             revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
+        uint256 prev = cidNFTs[msg.sender];
+        if (prev == _cidNFTID) {
+            return;
+        }
+        if (prev != 0) {
+            emit CIDNFTRemoved(msg.sender, prev);
+        }
         cidNFTs[msg.sender] = _cidNFTID;
         emit CIDNFTAdded(msg.sender, _cidNFTID);
     }
```

## 04. The data state of CidNFT contract may be overwrite through reentrancy
A user may overwrite the CiaNFT's data state through reentrancy, which causes its NFT to be locked in the contract.

For example:
1. A user deploys a contract X, transfer CidNFT `c1` and subprotocol NFT `subId1`, `subId2` to X
2. X call `CiaNFT.add(c1, "subName", 0, subId1, PRIMARY)` to add subId1 to `c1`: `cidData[c1][subName].primary` is set to subId1
3. X call `CiaNFT.add(c1, "subName", 0, subId2, PRIMARY)` to add subId2 to `c1`:
  - It will trigger `remove(c1, "subName", 0, 0, PRIMARY)` to [set `cidData[c1][subName].primary = 0`, and transfer `subId1` back to X](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L269-L270)
    - The safeTransferFrom will trigger the callback - `X.onERC721Received()`.
    - In the callback, if X calls `CiaNFT.add(c1, "subName", 0, subId1, PRIMARY)` again, `cidData[c1][subName].primary` will be set to `subId1` again
  - After callback and remove returns, [`cidData[c1][subName].primary` is set to `subId2`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L209)
4. Now, both `subId1` and `subId2` is transferred to CiaNFT, but only `subId2` is recorded in `cidData[c1][subName].primary`, `subId1` is lost.

Recommended Mitigation:
Option1: add nonReentrant to `CiaNFT.add()` and `CiaNFT.remove()`
Option2: use `transferFrom()` instead of `safeTransferFrom()` in `CiaNFT.remove()`
Option3: transfer the removed NFT after state update in `CiaNFT.add()`

Sample for Option3:

```
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index d6a23db..d0fa1ae 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -194,19 +194,23 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
         }
         if (_type == AssociationType.ORDERED) {
             if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
-            if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {
-                // Remove to ensure that user gets NFT back
-                remove(_cidNFTID, _subprotocolName, _key, 0, _type);
-            }
+            uint256 prev = cidData[_cidNFTID][_subprotocolName].ordered[_key];
+            require(prev != _nftIDToAdd);
             cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;
+            if (prev != 0) {
+                nftToAdd.safeTransferFrom(address(this), msg.sender, prev);
+                emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, prev);
+            }
             emit OrderedDataAdded(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
         } else if (_type == AssociationType.PRIMARY) {
             if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
-            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
-                // Remove to ensure that user gets NFT back
-                remove(_cidNFTID, _subprotocolName, 0, 0, _type);
-            }
+            uint256 prev = cidData[_cidNFTID][_subprotocolName].primary;
+            require(prev != _nftIDToAdd);
             cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
+            if (prev != 0) {
+                nftToAdd.safeTransferFrom(address(this), msg.sender, prev);
+                emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, prev);
+            }
             emit PrimaryDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd);
         } else if (_type == AssociationType.ACTIVE) {
             if (!subprotocolData.active) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
```
