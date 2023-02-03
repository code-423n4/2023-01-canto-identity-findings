# QA (LOW & NON-CRITICAL)

## [L-01] The protocol fees can be truncated

### Proof of Concept
When registering, a user defines the allowed association types and an optional fee that is charged when adding the subprotocol to a CID NFT. If the subprotocol owner configured fee is less than the MAGIC_NUMBER, the subprotocol owners will receive their fee while the protocol will not due to truncation.

```solidity
        // Charge fee (subprotocol & CID fee) if configured
        uint96 subprotocolFee = subprotocolData.fee;
        if (subprotocolFee != 0) {
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
        }
```

[Link](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L188-L194)



### Impact
Loss of protocol fees 
### Recommended Mitigation Steps
At SubprotocolRegistry contract's `register` function, add this require function;

```solidity
    uint constant MAGIC_NUMBER = 9; // <------ @audit-info

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
        SubprotocolData memory subprotocolData = subprotocols[_name];
        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
        require(_fee > MAGIC_NUMBER, "less than magic"); // <----- @audit-info
```


## [NC-01] The protocol does not earn any fees for every CidNFT minted

### Proof of Concept
The NATSPEC of CidNFT contract states that the protocol will charge 10% of the fees for every NFT minted. 

>
    /// @notice Fee (in BPS) that is charged for every mint (as a percentage of the mint fee). Fixed at 10%.
    uint256 public constant CID_FEE_BPS = 1_000;
[Link](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L16-L17)

However, the fees are only charged during subprotocol registration at SubprotocolRegistry contract's `register` function
[Link](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L87)

or ,if the subprotocol has configures fees, when `add` function called at `CidNFT` contract.
```solidity
        // Charge fee (subprotocol & CID fee) if configured
        uint96 subprotocolFee = subprotocolData.fee;
        if (subprotocolFee != 0) {
            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
            SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, cidFee);
            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);
        }
```

[Link](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L188-L194)

`CidNFT` contract has mint function to mint the CiDNFT's and it does not cut fees as opposed to NATSPEC
```solidity
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
[Link](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L147-L157)


### Impact
The protocol will not gain fees

### Recommended Mitigation Steps
If the functionality is thought to be the fees are cut only during add function call and subprotocol registry as written above, the NATSPEC should be corrected.



