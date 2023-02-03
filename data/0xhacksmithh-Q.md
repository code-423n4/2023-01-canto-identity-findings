### [Low-1] Absence of Re-entrancy checks in ```mint()```
Due to absence of reentrancy gaurd in ```mint()``` function, attacker can re-enter same function many times when nft transfer to his address(specifically using attack contracts) and mint as much as NFT he can, as minting is free

If action perform multiple times and attackers mint a significiant amount of Nfts each time, then may minted token amount reach max value of Uint256, after that it will lead to overflow condition and minting function will be useless.

So in that case Nfts only available to some group of people(or attackers), normal users unable to use protocol features i.e  unable to link their Subprotocol to CID NFTs.
This lead to some sorts of centralization.
  
*Instances(1)*
```solidity
File :: CidNFT.sol
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147-L157


function mint(bytes[] calldata _addList) external { // @audit-issue absence of reeentrancy, so some user will mint all CID nft 
        _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
        bytes4 addSelector = this.add.selector; // @audit-info 
        for (uint256 i = 0; i < _addList.length; ++i) { // @audit uncheck
            (
                bool success, /*bytes memory result*/

            ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i])); // @audit-info
            if (!success) revert AddCallAfterMintingFailed(i);
        }
    }
```


### [Low-2] Old, Floating Solidity version used
- Use a solidity version of at least 0.8.4 to get `bytes.concat()`
 instead of `abi.encodePacked(<bytes>,<bytes>)`
- Use a solidity version of at least 0.8.12 to get `string.concat()`
 instead of `abi.encodePacked(<str>,<str>)`
- Use a solidity version of at least 0.8.13 to get the ability to use `using for`
  with a list of free functions
- Use solidity version 0.8.17 will provide an overall gas optimization

*Instances(3)*
```solidity
File :: CidNFT.sol
File :: SubprotocolRegistry.sol
File :: AddressRegistry.sol
```