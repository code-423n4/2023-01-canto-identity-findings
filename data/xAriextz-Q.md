##  GENERAL
Consider not using the character "_" in function parameters. This is only recommended when there is shadowing, and it is better to do it with the "_" at the end of the variable name instead of in the beginning. 

## CidNFT.sol
# 1
Links:
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L172-L183
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L244-L254
Consider using a function for this repeated code.
# 2 
In case a CidNFT gets stolen from a wallet, they can use that NFT for stealing all the NFTs from subprotocols that were added.
# 3
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L265
The NFT that is being removed is the one taken from the mapping, which doesn't have to coincide with the _nftIDToRemove. This can lead to a user misunderstanding the function and removing a NFT they didn't want to. Also, the parameter of that is being emitted in this event may be wrong because what I just have stated. Example: I call the function remove() with a _nftIDToRemove = 0. In the event we will emit _nftIDToRemove, but in reality the NFT that was removed is not that on. The value that should be emitted is "currNFTID".
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L271
Same as "3" before but in the PRIMARY associations.
# 5 
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L190-L194
In case that the subprotocolFee is too low the cidFee will be 0. This will lead to calling the safeTransferFrom with a 0 amount of tokens, which may not what is intended to do and also will use gas needlessly.  There will be rounding problems in general with low subprotoclFees
# 6 
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L290-L337
Consider returning directly the values instead of assigning them. If some contract uses this functions to get data from the contract, it will save some gas.
## SubprotocolRegistry.sol
# 1
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L46
I guess the naming of the first parameter of the event is a typo.
# 2
Link: https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L92
Consider checking if the fee is between some kind of max and min values.