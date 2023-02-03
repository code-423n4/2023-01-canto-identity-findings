## CidNFT.sol
# 1 
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L186-L187
Instead of writing this line, write it like this:
ERC721(subprotocolData.nftAddress).safeTransferFrom(msg.sender, address(this), _nftIDToAdd);
The median of gas with the original code: 52935
The median of gas with my change: 52928
# 2 
Links:
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L256
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L264
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L270
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L285
 
Instead of assigning the value to the variable, just do this:
ERC721(subprotocolData.nftAddress).safeTransferFrom(address(this), msg.sender, currNFTID);
The median of gas with the original code: 9372
The median of gas with my change: 9364
##SubprotocolRegistry.sol
# 1 
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L89-L99
Instead of assigning to a "memory" variable and then assigning again, make the first variable "storage" directly.
Code changed:

SubprotocolData storage subprotocolData = subprotocols[_name];
if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
subprotocolData.owner = msg.sender;subprotocolData.fee = _fee;
if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
      revert NotASubprotocolNFT(_nftAddress);
subprotocolData.nftAddress = _nftAddress;
subprotocolData.ordered = _ordered;
subprotocolData.primary = _primary;
subprotocolData.active = _active; 

The median of gas with the original code: 54258
The median of gas with my change: 53421