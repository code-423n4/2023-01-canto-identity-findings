### OVERWORK WITH SAFE
```
contract CidNFT is ERC721, ERC721TokenReceiver {
 
         ERC721 nftToAdd = ERC721(subprotocolData.nftAddress);
-        nftToAdd.safeTransferFrom(msg.sender, address(this), _nftIDToAdd);
+        nftToAdd.transferFrom(msg.sender, address(this), _nftIDToAdd); // delete Safe | no need in checking safe because this protocol have onERC721Received
```
No need in checking safe because this protocol have `onERC721Received` function
### OVERWORK FUNCTION
```
         } else if (_type == AssociationType.PRIMARY) {
             if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
-            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
+            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {  //overwork in cidData[_cidNFTID][_subprotocolName].primary
                 // Remove to ensure that user gets NFT back
-                remove(_cidNFTID, _subprotocolName, 0, 0, _type);
+                //remove(_cidNFTID, _subprotocolName, 0, 0, _type); no matter in calling remove function
+                uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
+                ERC721(subprotocolData.nftAddress).safeTransferFrom(address(this), msg.sender, currNFTID); //in line 147 when 
+                emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, 0);
+
             }
```
Calling remove function is overwork. We can remove it in this branch, without any checks
### ACTIVE ADD OPTIMISATION
``` 
@@ contract CidNFT is ERC721, ERC721TokenReceiver {
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
+            // Check for duplicates
+            if (activeData.positions[_nftIDToAdd] != 0) // its harder for first iteration but less check for next elements
+                revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
+            activeData.values.push(_nftIDToAdd);
+            activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
             emit ActiveDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd, lengthBeforeAddition);
         }
     }
```
It is possible to not check length == 0  each time. Pushing first element same as the  pushing others. Yes, for first element code do more complex check, but i thought in ACTIVE mode pushing of next elements will be more often.
### USING DELETE INSTEAD OF `= 0`
```
@@ -281,8 +278,8 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
                 activeData.positions[befSwapLastNFTID] = arrayPosition;
             }
             activeData.values.pop();
-            activeData.positions[_nftIDToRemove] = 0;
+            delete activeData.positions[_nftIDToRemove]; // delete cost lower than  (= 0)
             emit ActiveDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
         }
     }
```
Delete is more gas efficent in storage than `= 0`
### OPTIMISING RETURN
```
    function getPrimaryData(uint256 _cidNFTID, string calldata _subprotocolName)
        external
        view
        returns (uint256)
    {
        return cidData[_cidNFTID][_subprotocolName].primary;
    }

    function getOrderedData(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key
    ) external view returns (uint256) {
        return cidData[_cidNFTID][_subprotocolName].ordered[_key];
    }

    function activeDataIncludesNFT( //may be use return without using a variable
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _nftIDToCheck
    ) external view returns (bool) {
        return cidData[_cidNFTID][_subprotocolName].active.positions[_nftIDToCheck] != 0;
    }
```
There is no need in allocating some memory for return
### PACKING VARIABLES IN STRUCTURE
```
@@ -30,10 +30,11 @@ contract SubprotocolRegistry {
         /// @notice Optional cost in NOTE to add an NFT
         /// @dev Maximum value is (2^96 - 1) / 10^18 =~ 80 billion. Zero for no fee
         uint96 fee;
-        address nftAddress;
         bool ordered;
         bool primary;
         bool active;
+        address nftAddress;
+        // change order of variables for better packing in the slots
     }
 ```
All booleans will be in one slot with fee instead of allocating 32 bytes more
### CHECK BEFORE ALLOCATING MEMORY + TRANSFER AFTER CHECKS
```
         string calldata _name,
         uint96 _fee
     ) external {
-        SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);

         if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);

+       if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId)) // you can check it before allocating memory
        SubprotocolData memory subprotocolData = subprotocols[_name];
        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
+        SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);

        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
-        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))

             revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        subprotocols[_name] = subprotocolData;

         emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
     }
```
it better to firstly check all conditions, secondly do transfer ERC20 token, and after write into storage  
### PACK ALL CHANGES IN ONE COMMAND
```
 @@ contract SubprotocolRegistry {

-        SubprotocolData memory subprotocolData = subprotocols[_name];
-        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
-        subprotocolData.owner = msg.sender;
-        subprotocolData.fee = _fee;
-        subprotocolData.nftAddress = _nftAddress;
-        subprotocolData.ordered = _ordered;
-        subprotocolData.primary = _primary;
-        subprotocolData.active = _active;
-        subprotocols[_name] = subprotocolData;
+        address subprotocolOwner = subprotocols[_name].owner;
+        if (subprotocolOwner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolOwner);
+        subprotocols[_name] = SubprotocolData({ // write in one command
+            owner : msg.sender,
+            fee : _fee,
+            nftAddress : _nftAddress,
+            ordered : _ordered,
+            primary : _primary,
+            active : _active
+        });
 
```
 instead of allocating memory and then pass it to storage.  better to write in one command