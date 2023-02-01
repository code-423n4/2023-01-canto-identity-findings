# [QA] - pragma version old and floating

## Description
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103

## Where
* https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/AddressRegistry.sol/#L2
* https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L2
* https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidSubprotocolNFT.sol/#L2
* https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/SubprotocolRegistry.sol/#L2

## Recommendation
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
solidity-specific/locking-pragmas

# [QA] - `SubprotocolData` IS DEFINED TWICE WITH DIFFERENT PROPERTIES

## Description
There're two different struct objects called `SubprotocolData` with different properties. This is confusing and could result on unexpected behaviors if one is used instead of the other one by mistake.

## Where
* https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L46
* https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/SubprotocolRegistry.sol/#L27

## Recommendation
It's not a good practice to have structures with the same name that means different things. The recommendation is to rename one of them with a different name.

# [QA] - ADD SPACES TO CODE TO IMPROVE READABILITY

## Description
From the readability perspective, it's really difficult to follow the code on `CidNFT.sol` since the code is compacted. It's recommended to add more empty lines on the code to make it easier to read.

## Where
[/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol)

## Recommendation
Add spacing between lines of code

# [QA] - USE OPENZEPPELIN'S [`EnumerableSet`](https://docs.openzeppelin.com/contracts/3.x/api/utils#EnumerableSet) FOR ACTIVE TYPE PROTOCOLS

## Description
EnumerableSet has the functionality that it's required for the data manipulation on the case of Active `AssociationType`. It's always recommended to use well tested libraries instead of create a custom implementation when possible.

## Where
[/src/CidNFT.sol#L40](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L40)

## Recommendation

# [QA] - USE `_safeMint()` INSTEAD OF `mint()`

## Description
Even though there is a comment that says that the use of `_mint()` instead of `_safeMint()` is intentional I still believe that `_safeMint()` should be used instead.

For reference, [OpenZeppelin's documentation for `_mint`](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-\_mint-address-uint256-) states: "Usage of this method is discouraged, use \_safeMint whenever possible".

## Where
[/src/CidNFT.sol#L148](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L148)

## Recommendation
Use `_safeMint()` instead of `_mint()`

## [QA] - CREATE AN AUXILIARY FUNCTION/MODIFIER FOR CidNFT ownership validation

## Description
It's recommended to reduce the code duplication. Since the nft ownership validation is being done twice, it should be better to use an auxiliary function for such validation.

## Where
* [/src/CidNFT.sol#L177-L182](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L177-L182)
* [/src/CidNFT.sol#L249-L254](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L249-L254)

## Recommendation
Create an auxiliary function

```diff
    contract CidNFT is ERC721, ERC721TokenReceiver {
        
        ...
        
        function add(
            uint256 _cidNFTID,
            string calldata _subprotocolName,
            uint256 _key,
            uint256 _nftIDToAdd,
            AssociationType _type
        ) external {
            ...
+           _validateNFTOwnership(_cidNFTID);       
-           address cidNFTOwner = ownerOf[_cidNFTID];
-           if (
-               cidNFTOwner != msg.sender &&
-               getApproved[_cidNFTID] != msg.sender &&
-               !isApprovedForAll[cidNFTOwner][msg.sender]
-           ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
            ...
        }
        
        ...

        function remove(
            uint256 _cidNFTID,
            string calldata _subprotocolName,
            uint256 _key,
            uint256 _nftIDToRemove,
            AssociationType _type
        ) public {
            ...
+           _validateNFTOwnership(_cidNFTID);            
-           address cidNFTOwner = ownerOf[_cidNFTID];
-           if (
-               cidNFTOwner != msg.sender &&
-               getApproved[_cidNFTID] != msg.sender &&
-               !isApprovedForAll[cidNFTOwner][msg.sender]
-           ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
            ...
        }
        
        ...

+       function _validateNFTOwnership(uint256 _cidNFTID) internal {
+           address cidNFTOwner = ownerOf[_cidNFTID];
+           if (
+               cidNFTOwner != msg.sender &&
+               getApproved[_cidNFTID] != msg.sender &&
+               !isApprovedForAll[cidNFTOwner][msg.sender]
+           ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
+       }        
        
        ...
    }
```

# [LOW] - ON MINTING, THE `_cidNFTID` used on the `add()` should be the new minted id.

## Description
The `_cidNFTID` used on the `add()` call should be the new one instead of be open to user input.

## Where
[/src/CidNFT.sol/#L147-L157](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L147-L157)

## Recommendation
```diff
    function mint(bytes[] calldata _addList) external {
+       uint256 newTokenId = numMinted;
+       unchecked {
+           ++numMinted;
+       }        
-        _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
+        _mint(msg.sender, newTokenId); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
        bytes4 addSelector = this.add.selector;
        for (uint256 i = 0; i < _addList.length; ++i) {
            (
                bool success, /*bytes memory result*/

-           ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
+           ) = address(this).delegatecall(abi.encodePacked(addSelector, newTokenId, _addList[i]));
            if (!success) revert AddCallAfterMintingFailed(i);
        }
    }
```