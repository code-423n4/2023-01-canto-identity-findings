# CREATE THE NEW SUBPROTOCOL OBJECT ONLY AFTER THE VALIDATIONS PASSED

## Description
It's more performant to delay complex object creation after all the validations passed and we are sure that the transaction wont revert to save gas on those cases.

## Where
[/src/SubprotocolRegistry.sol/#L89](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/SubprotocolRegistry.sol/#L89)

## Recommendation
```diff
    function register(
        bool _ordered,
        bool _primary,
        bool _active,
        address _nftAddress,
        string calldata _name,
        uint96 _fee
    ) external {
        SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
        if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
-       SubprotocolData memory subprotocolData = subprotocols[_name];
+       address owner = subprotocols[_name].owner;
-       if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
+       if (owner != address(0)) revert SubprotocolAlreadyExists(_name, owner);
+       if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
+           revert NotASubprotocolNFT(_nftAddress);
+       SubprotocolData memory subprotocolData;
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
-       if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
-           revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        subprotocols[_name] = subprotocolData;
        emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
    }
```

# INITIALIZE `numMinted` TO ONE TO SAVE GAS ON FIRST MINT

## Description
Since the 0 tokenId is not valid for this NFT contract, the initial tokenId can be initialized as 1. This will reduce the gas consumption for the first minting since the mutation from the default value to a different one on state variables is always more gas intensive than a mutation between two different non default values.

## Where
[/src/CidNFT.sol#L67](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L67)

## Recommendation
```diff
contract CidNFT is ERC721, ERC721TokenReceiver {
-   uint256 public numMinted;
+   uint256 public numMinted = 1;

    function mint(bytes[] calldata _addList) external {
-       _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
+       _mint(msg.sender, numMinted++); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
        bytes4 addSelector = this.add.selector;
        for (uint256 i = 0; i < _addList.length; ++i) {
            (
                bool success, /*bytes memory result*/

            ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
            if (!success) revert AddCallAfterMintingFailed(i);
        }
    }

}
```

# USE EXISTING MEMORY VARIABLE `subprotocolData.owner` INSTEAD OF CREATE A NEW ONE `subprotocolOwner`

## Description
It's unnecessary to create a new memory variable for the subprocol owner. We can read its value from the struct in memory instead.

## Where
[/src/CidNFT.sol#L175](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L175)

## Recommendation
Remove the temp variable and use `subprotocolData.owner` instead

```diff
    function add(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToAdd,
        AssociationType _type
    ) external {
-       address subprotocolOwner = subprotocolData.owner;
-       if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
+       if (subprotocolData.owner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
        ...
        if (subprotocolFee != 0) {
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
-           SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
+           SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolData.owner, subprotocolFee - cidFee);
        }
    }

    function remove(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToRemove,
        AssociationType _type
    ) public {
        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
            _subprotocolName
        );
-       address subprotocolOwner = subprotocolData.owner;
-       if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
+       if (subprotocolData.owner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
        ...
    }
```

# MOVE INPUT VALIDATIONS TO THE TOP OF THE FUNCTION BODY WHEN POSSIBLE

## Description
It's more gas efficient to validate the function inputs as early as possible to avoid waste of gas. If the inputs are invalid then there is no need to waste more gas on the rest of the execution.

## Where
[/src/CidNFT.sol#L183](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L183)

## Recommendation
Move all the input validations (when possible) to the top of the function body

```diff
    function add(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToAdd,
        AssociationType _type
    ) external {
+       if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols

        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
            _subprotocolName
        );
        address subprotocolOwner = subprotocolData.owner;
        if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
        address cidNFTOwner = ownerOf[_cidNFTID];
        if (
            cidNFTOwner != msg.sender &&
            getApproved[_cidNFTID] != msg.sender &&
            !isApprovedForAll[cidNFTOwner][msg.sender]
        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
-       if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols
        ...
    }
```

# Use `unchecked` for operations not expected to overflow 

## Description
Use of unchecked can save gas where computation is known to be overflow/underflow safe.

## Where
* [/src/CidNFT.sol/#L150](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L150) - `++i`
* [/src/CidNFT.sol/#L148](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L148) - `++numMinted`
* [/src/CidNFT.sol/#L191](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L191) - `uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;`
* [/src/CidNFT.sol/#L225](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol/#L225) - `activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;`

## Recommendation
Add `unchecked { }` and casts to operations that cannot overflow.
