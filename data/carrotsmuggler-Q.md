# Low Impact

### 1. Re-entrancy in `SubprotocolRegistry.sol` `register()` allows for registering same name multiple times and emitting false events.

The function `register()` makes an external call (`_nftAddress.supportsInterface()`) but has no re-entrancy lock. 
```solidity
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
        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
            revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        subprotocols[_name] = subprotocolData;
        emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
    }
```
If a malicious address is passed in the `_nftAddress` field, the malicious address can take control and call `register()` again, with the same `_name`. This call will not revert since `subProtocolData` is defined in the memory, not storage and thus `subprotocols[_name]` is still unset. This will lead to two emitted events, with the same value in the `_name` field which can throw off front-end elements parsing subgraphs for events.

Advised to use re-entrancy lock on `register()` function