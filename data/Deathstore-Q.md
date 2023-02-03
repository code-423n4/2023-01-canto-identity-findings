
### ZERO FEE
```
 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
         // Charge fee (subprotocol & CID fee) if configured
         uint96 subprotocolFee = subprotocolData.fee;
-        if (subprotocolFee != 0) {
             uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
             SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
             SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
```
If owner of subprotocol set fee <10, then cidFeeWallet won't be able to have any token's
as possible variants:
+ add baseFee for all subprotocols or for all nonzero fee subprotocols (for example 1 NOTE)
+ increase fee for low fee protocol(for example 50%(yes, it's still problem with fee as 4 note)

as an example some subprotocols can be used in big projects, that won't be happy to pay for each transatcion to cidFeeWallet. And they would collect their 9 note for each transaction without fee  or set to 0  and coolect fees on their own)

or if you don't care about this case, it's better to just divide by 10, and not multiplying at all (saves gas)
### NOT USE SAFE
```
@@ -261,13 +258,13 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
                 // This check is technically not necessary (because the NFT transfer would fail), but we include it to have more meaningful errors
                 revert OrderedValueNotSet(_cidNFTID, _subprotocolName, _key);
             delete cidData[_cidNFTID][_subprotocolName].ordered[_key];
-            nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
+            nftToRemove.transferFrom(address(this), msg.sender, currNFTID);  //as hypothesis in line 148 about not using safe mint we can use simple transfreFrom
             emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);
         } else if (_type == AssociationType.PRIMARY) {
             uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
             if (currNFTID == 0) revert PrimaryValueNotSet(_cidNFTID, _subprotocolName);
             delete cidData[_cidNFTID][_subprotocolName].primary;
-            nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
+            nftToRemove.transferFrom(address(this), msg.sender, currNFTID); //as hypothesis in line 148 about not using safe mint we can use simple transfreFrom
             emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
         } else if (_type == AssociationType.ACTIVE) {
             IndexedArray storage activeData = cidData[_cidNFTID][_subprotocolName].active;
@@ -281,8 +278,8 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
                 activeData.positions[befSwapLastNFTID] = arrayPosition;
             }
             activeData.values.pop();
             activeData.positions[_nftIDToRemove] = 0;
-            nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);

+            nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove); //as hypothesis in line 148 about not using safe mint we can use simple transfreFrom
             emit ActiveDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
         }
     }
```
As it mentioned in functioned mint "// We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back". 
On the same purpose it's cheaper to use `transferFrom` instead of `safeTransferFrom`.
`safe` just add require to another protocol if it has function `onERC721Received`. and as our hypothesis it's not neccesary to check it. + gas economy