# Gas report

## Author: rotcivegaf

## [G-01] Add `unchecked{}` in operations where the operands cannot over/underflow

```solidity
File: src/CidNFT.sol

148:        _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back

191:            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;

193:            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);

225:                activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
```

## [G-02] `++i` should be `unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)

```solidity
File: src/CidNFT.sol

148:        for (uint256 i = 0; i < _addList.length; ++i) {
```

## [G-03] Remove the if on the last path of if/else nest

The `AssociationType` enum have 3 constants if not the first and not the second, the only possible path is the third

```solidity
File: src/CidNFT.sol

211:        } else if (_type == AssociationType.ACTIVE) {

272:        } else if (_type == AssociationType.ACTIVE) {
```

### [G-04] Assign struct as an object instead as assignments of parameters

```solidity
File: src/CidNFT.sol

From:
91:        subprotocolData.owner = msg.sender;
92:        subprotocolData.fee = _fee;

95:        subprotocolData.nftAddress = _nftAddress;
96:        subprotocolData.ordered = _ordered;
97:        subprotocolData.primary = _primary;
98:        subprotocolData.active = _active;
99:        subprotocols[_name] = subprotocolData;

To:
        subprotocols[_name] = SubprotocolData({
            owner: msg.sender,
            fee: _fee,
            nftAddress: _nftAddress,
            ordered: _ordered,
            primary: _primary,
            active: _active
        });

Also remove:
89:        SubprotocolData memory subprotocolData = subprotocols[_name];

And consult directly `subprotocols[_name]`:
90:        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
To:
        if (subprotocols[_name].owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocols[_name].owner);
```
