QA1. The ``getCID`` fails to check if ``_user`` is still the owner of the NFT that has been registered: it is possible that the ``_user`` is not the owner of the registered cidNFT anymore.  

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L62-L64
```javascript
 function getCID(address _user) external view returns (uint256 cidNFTID) {
        cidNFTID = cidNFTs[_user]; 
}
```

For example, A registers NFT 44, but then transfers NFT 44 to somebody else. At this point, the registration is not valid anymore. However, since transfer is not part of this contract, there is no way for this contract to invalidate  the obsolete record immediately. This can be done if the new owner tries to register with the contract.

Recommendation: we can check if _user is still the owner of the NFT and revert if not. The invalidation of the record can be delayed to be done at the point of new owner's registration.
```javascript
 function getCID(address _user) external view returns (uint256) {
        
        uint 256 cidNFTID = cidNFTs[_user]; 
        if(ERC721(cidNFT).ownerOf(cidNFTID)  != _user) revert obsoleteRegistration();
        else return cidNFTID;     
}

```


