1.add() There is a risk of primary being overwritten
In the add() execution process, will determine whether the primary exists, if it exists remove it first (while returning the old NFT), and then set the new primary

```solidity
    function add(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToAdd,
        AssociationType _type
    ) external {
...
else if (_type == AssociationType.PRIMARY) {
            if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
                // Remove to ensure that user gets NFT back   //<------------------------
                remove(_cidNFTID, _subprotocolName, 0, 0, _type);  
            }
            cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
            emit PrimaryDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd);

```

In the remove() implementation, safeTransferFrom() is used and all methods do not add nonreentrant

```solidity
    function remove(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToRemove,
        AssociationType _type
    ) public {
....    
            else if (_type == AssociationType.PRIMARY) {
                        uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
                        if (currNFTID == 0) revert PrimaryValueNotSet(_cidNFTID, _subprotocolName);
                        delete cidData[_cidNFTID][_subprotocolName].primary;
                        nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID); //<--------- There is the possibility of re-entry
                        emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
                    } 
```


So there is remove() re-entry, onERC721Received call add() set the same primary, so that the second add() will be the first add() primary was overwritten

Technically it is possible, it is recommended to check the primary must be equal to 0  again after remove()

Note: ORDERED has a similar problem

```soldity
    function add(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToAdd,
        AssociationType _type
    ) external {
...

        if (_type == AssociationType.ORDERED) {
            if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
            if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {
                // Remove to ensure that user gets NFT back
                remove(_cidNFTID, _subprotocolName, _key, 0, _type);
+               require(cidData[_cidNFTID][_subprotocolName].ordered[_key]==0,"reset key");
            }
            cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;
            emit OrderedDataAdded(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
        } else if (_type == AssociationType.PRIMARY) {
            if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
                // Remove to ensure that user gets NFT back
                remove(_cidNFTID, _subprotocolName, 0, 0, _type);
+               require(cidData[_cidNFTID][_subprotocolName].primary==0,"reset primary");
            }
            cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
            emit PrimaryDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd);
        }
```



2.Wrong parameter for remove() event
emit OrderedDataRemoved()/PrimaryDataRemoved()  The parameter subprotocolNFTID is wrong, it should use the removed id, not _nftIDToRemove which is useless for ORDERED and PRIMARY
Suggestion:

```solidity
    function remove(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToRemove,
        AssociationType _type
    ) public {
...    
        if (_type == AssociationType.ORDERED) {
            // We do not have to check if ordered is supported by the subprotocol. If not, the value will not be unset (which is checked below)
            uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].ordered[_key];
            if (currNFTID == 0)
                // This check is technically not necessary (because the NFT transfer would fail), but we include it to have more meaningful errors
                revert OrderedValueNotSet(_cidNFTID, _subprotocolName, _key);
            delete cidData[_cidNFTID][_subprotocolName].ordered[_key];
            nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
+           emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, currNFTID);            
-           emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);
        } else if (_type == AssociationType.PRIMARY) {
            uint256 currNFTID = cidData[_cidNFTID][_subprotocolName].primary;
            if (currNFTID == 0) revert PrimaryValueNotSet(_cidNFTID, _subprotocolName);
            delete cidData[_cidNFTID][_subprotocolName].primary;
            nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
+           emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, currNFTID);            
-           emit PrimaryDataRemoved(_cidNFTID, _subprotocolName, _nftIDToRemove);
        } else if (_type == AssociationType.ACTIVE) {        
```


