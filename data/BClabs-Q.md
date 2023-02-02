# 1 Zero address check missing in constructors

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L65-L68
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L119-L131
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L36-L38

# 2 Use static solidity version

Instead of a floating Solidity version uses a fixed one (0.8.0 instead of >=0.8.0)

# 3 Use a more recent version of solidity
All the contracts in scope are using >=0.8.0.

Consider using the latest stable version of solidity to ensure the compiler contains the latest security fixes.

# 4 Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
Scientific notation can be used for better code readability.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L17

# 5 Comment is wrong and should be fixed

A comment is wrong, data fits in 2 slots, not 1.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L26

# 6 NFT jsons could run out
Since baseURI is immutable and nft metadata is usually uploaded to ipfs, there is no way to add additional jsons after the data has been submitted, 
nor can one change the base URI. One could argue that metadata could be uploaded to a server and updated there, but it's bad since admins can then manipulate metadata as they wish.

The number of NFTs should therefore be limited to the number of JSONs or add a setBseUri function. Or in case there is just one variant of metadata, all URI should point to that one instance.

# 7 In case subprotocol NFT will be eligible for an airdrop, there is no way for the owners to claim it

Since cidNFT.sol is also an escrow for the NFTs, there is a chance that the NFTs are eligible for an airdrop. If the tokens or nfts are then dropped to the contract, there is no way of sweeping them.

# 8 `Remove()` call in `add()` function uses the wrong nftIDToRemove

When calling `remove()` in cases of ORDERED and PRIMARY, `_nftIDToRemove` should be the id of the nft, not a fixed 0, which then emits an event with the wrong parameter.

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L207
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L199