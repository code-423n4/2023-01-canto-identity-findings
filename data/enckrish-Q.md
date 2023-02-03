# QA Report

## [NC-01] `addList` feature in `mint` may be unusable
**Filename**: *src/CidNFT.sol*
Setting the `_addList` parameter in `mint` properly, allows to add subprotocols to the minted NFT in the `mint` call itself. 
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147-L157
```solidity=
    function mint(bytes[] calldata _addList) external {
        _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
        bytes4 addSelector = this.add.selector;
        for (uint256 i = 0; i < _addList.length; ++i) {
            (
                bool success, /*bytes memory result*/

            ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
            if (!success) revert AddCallAfterMintingFailed(i);
        }
    }
```
To set it properly, the user needs to know beforehand the NFT ID that he would be getting, but if another `mint` transaction is mined before, then the ID to be minted will change.
If wrong NFT ID is passed in `_addList`, then `add` will revert, since the caller won't be authorised to use that NFT, causing the `mint` transaction to fail. 
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L178-L182
```solidity=178
        if (
            cidNFTOwner != msg.sender &&
            getApproved[_cidNFTID] != msg.sender &&
            !isApprovedForAll[cidNFTOwner][msg.sender]
        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```
So, if there are multiple transactions that are trying to mint, then setting addList carries a high risk of transaction failure.

## [NC-02] Declare `cidNFT` address as of type ERC721   
**Filename**: *src/AddressRegistry.sol*
ERC721 token addresses can be declared of type `ERC20` itself, preventing repeated use of type conversions using `ERC721(address)` when using it.
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L14
```solidity=14
    address public immutable cidNFT;
```
Instead declare as:
```solidity=14
    ERC721 public immutable cidNFT;
```
Along with this the `constructor` may be written as:
```solidity=36
    constructor(ERC721 _cidNFT) {
        cidNFT = _cidNFT;
    }
```

## [NC-03] SafeTransfer may be removed for NOTE transfers 
`NOTE` token reverts when `transfer`/`transferFrom` fails, so `safeTransfer`/`safeTransferFrom` is not required.
**Filename**: *src/SubprotocolRegistry.sol*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L87
```solidity=87
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
```
**Filename**: *src/CidNFT.sol*
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L192
```solidity=192
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
```

## [NC-04] Unnecessary parameter in `NoTypeSpecified` error
**Filename**: *src/SubprotocolRegistry.sol*
`NoTypeSpecified` error contains the `name` passed for registering a new `Subprotocol` with. Since, getting the name gives us no new insight into the error, it should be removed. 
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L88

## [N-05] Use a initializer style syntax for setting Subprotocol value
**Filename**: *src/SubprotocolRegistry.sol*
Currently, all members of `SubProtocolData` struct are set individually in `SubprotocolRegistry.register`. 
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L91
```solidity=91
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
            revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        subprotocols[_name] = subprotocolData;
```
Consider changing it to:
```solidity=
        revert NotASubprotocolNFT(_nftAddress);
        subProtocolData = SubprotocolData({owner: msg.sender, 
                                           fee: _fee, 
                                           nftAddress: _nftAddress, 
                                           ordered: _ordered, 
                                           primary: _primary, 
                                           active: _active
                                          })
```

## [NC-06] Unnecessary use of `ERC721.safeTransferFrom` when pulling NFTs to contract
**Filename**: *src/CidNFT.sol*
`CidNFT.add.nftToAdd` is pulled to the `CidNFT` contract using `safeTransferFrom`, which can be replaced with `transfer`, since the contract expects to get the NFT.
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L187
```solidity=187
        nftToAdd.safeTransferFrom(msg.sender, address(this), _nftIDToAdd);
```

## [NC-07] Store `storage` reference used repeatedly in a storage pointer for readability
**Filename**: *src/CidNFT.sol*
In `CidNFT.add` and `CidNFT.remove`, `cidData[_cidNFTID][_subprotocolName]` is used repeatedly (6 times in `add`, and 10 in `remove`) , and using a storage pointer for it will increase the code readability by a large amount.

```solidity
        SubprotocolData storage ref = cidData[_cidNFTID][_subprotocolName];
```