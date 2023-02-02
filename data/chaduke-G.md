G1. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L89-L90

Two gas savings here: 1) No need to cache the whole struct, since we only need to read the owner information; 2) we don't even need to cache the owner since we only need it ONCE. The revision is as follows.

```javascript
        SubprotocolData memory subprotocolData;  // @audit: only declare it no copy here
        if (subprotocols[_name].owner != address(0))  // @audit: only read it once, no caching necessary
                revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
        subprotocolData.owner = msg.sender;
```

G2. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L87-L93

The three checks (if statements) for the ``register`` function can be executed first to save gas (short-circuit):
```
        
        if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
        if (subprotocols[_name].owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
            revert NotASubprotocolNFT(_nftAddress);

        SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
        SubprotocolData memory subprotocolData;
       
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
```

G3. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L178-L183
These two if-statements can get executed right in the beginning to short-circuit and save gas.
```
if (
            cidNFTOwner != msg.sender &&
            getApproved[_cidNFTID] != msg.sender &&
            !isApprovedForAll[cidNFTOwner][msg.sender]
        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
        if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in sub
```

G4. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L199
It is much more gas-saving to replace this line by the following few lines to avoid another function call and the complexity (several if-else statements) in the ``remove()`` function.
```javascript
            
            uint256 _nftIDToRemove = cidData[_cidNFTID][_subprotocolName].ordered[_key] ;
            if (_nftIDToRemove != 0) {
                // Remove to ensure that user gets NFT back
                nftToAdd.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
                emit OrderedDataRemoved(_cidNFTID, _subprotocolName, _key, _nftIDToRemove);               
            }
 ```

G5. Introducing a modifier called ``typeMatch(type)`` can save much gas due to short circuit. The original code will send fees and NFTs first before matching  type, a waste of gas when there is a mismatch. Such refactoring also improves the readability of the ``add`` function. We also perform the zero address check for the owner here. Implementing it using assembly can further save more gas. 
```javascript
modifier typeMatch(AssociationType _type, string calldata _subprotocolName) {
      SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
            _subprotocolName
        );
      if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
      if(!(_type == AssociationType.ORDERED && subprotocolData.ordered || 
         _type == AssociationType.PRIMARY && subprotocolData.primary || 
         _type == AssociationType.ACTIVE && subprotocolData.active)
      ) {
               revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
      }
      _;
}
function add(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToAdd,
        AssociationType _type
    ) typeMatch(_type, _subprotocolName) external
```



