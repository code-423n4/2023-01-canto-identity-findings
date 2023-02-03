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

QA2. The ``SubprotocolRegistry`` contract only allows a user to register, in case the wrong information is registered, there is no way to correct it. 


When this happens, the only way is to register a new one. This requires one to use a new name, and if the old name is the one that one really like, there is no way to get that name back.

Recommendation: alllow a registered user to update other information other than the ``name`` and owner address. 

QA3. ``cidFeeWallet`` is defined as immutable, that means even when the wallet is compromised, there is no way to change it. 

Recommendation:  change ``cidFeeWallet`` as a private variable, and add a function to allow the change of ``cidFeeWallet`` by a privileged owner. 

QA4. The ``tokenURL()`` implementation does not conform exactly to [eip-721](https://eips.ethereum.org/EIPS/eip-721):
```
 function tokenURI(uint256 _id) public view override returns (string) {
        if (ownerOf[_id] == address(0))
            // According to ERC721, this revert for non-existing tokens is required
            revert TokenNotMinted(_id);
        return string(abi.encodePacked(baseURI, _id, ".json"));
    }
```
1) It should revert only when the NFT is not valid, not because when the owner is zero. This is because a valid NFT might be burned to the zero address.  For example, one likes to implement a fully permissionless subprotcol, and sending the NFT to the zero address might be the way to do it. This does not mean the NFT is invalid. All the subprotocal data is still valid. 

2)  The function should be external instead of public, according to EIP-721. 


Recommendation:
```diff

- function tokenURI(uint256 _id)  public view override returns (string memory) {
+ function tokenURI(uint256 _id) external  view override returns (string memory) {

-       if (ownerOf[_id] == address(0))
+       if(_id >=1 && _id <= numMinted)
            // According to ERC721, this revert for non-existing tokens is required
            revert TokenNotMinted(_id);
        return string(abi.encodePacked(baseURI, _id, ".json"));
    }
```

QA5. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L159-L165

The NatSpec of the [add](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L165-L171) function fails to have the information that auser needs to approve the CidNFT for the transfer of the given NFT ``_nftIDToAdd`` before the calling the ``add`` function. This is important information for improving user experience. 

Recommendation: add the following to the NatSpec of the ``add`` function:  "A user needs to approve the CidNFT for the transfer of the given NFT ``_nftIDToAdd`` before the calling the add function; otherwise, the add function will revert."

QA6. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L202

The ``add`` function fails to check whether a new NFT added to replace an old one are the same or not, resulting unnecessary NFT transfers and gas and false emits. 

Recommendation: double check whether the new NFTID to be added is the same as the old NFTID that will be removed. If they are equal, return or revert. 

QA7. https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L315-L319
The last line of the comment is wrong. The correction is
 "@return subprotocolNFTIDs The ID list of the active NFTs at the queried subprotocol / CID NFT. 0 if it does not exist"

QA8. ``pragma`` should lock to a particular solidity compiler version.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L2

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L2

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L2

QA9. The ``add`` function requires the user to give approval to the ``CidNFT`` contract to transfer the protocol fee (note) to the ``cidFeeWallet`` and the subprotocol owner. Other, the function ``add()`` will revert. It is important to document this in the NatSpec of the ``add()`` function.

[https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L190-L194](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L190-L194)

```
if (subprotocolFee != 0) {
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
        }
```

QA10.  Both the ``add()`` and ``remove()`` functions might remove NFTs from the ``CidNFT`` contract and send them to the msg.sender:

```
nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);
```

The safeTransferFrom() function will call a callback function of the caller as follows:

```
 require(
            to.code.length == 0 ||
                ERC721TokenReceiver(to).onERC721Received(msg.sender, from, id, "") ==
                ERC721TokenReceiver.onERC721Received.selector,
            "UNSAFE_RECIPIENT"
        );
```

Therefore, reentrance attack is possible. Although this audit has not found a reentrancy attack scenario, for safety issue, it is still advisable to use openZeppelin's ReentrancyGuaud to eliminate future concerns: 
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol
