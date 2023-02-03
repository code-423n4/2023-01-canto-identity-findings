## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `register` METHOD CAN BE GAS OPTIMIZED BETTER | 1 |
| [GAS-2](#GAS-2) | UNNECESSARY IF-ELSE CONDITION USED | 1 |
| [GAS-3](#GAS-3) | ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW | 2 |
| [GAS-4](#GAS-4) | USING BOOLS FOR STORAGE INCURS OVERHEAD | 3 |

### [GAS-1] `register` METHOD CAN BE GAS OPTIMIZED BETTER

In `register` method of `SubprotocolRegistry.sol`, storing `subprotocolData` data as memory instead of storage costs more than 2100 gas than what it should cost. 

*Instance:*
```solidity
File: SubprotocolRegistry.sol

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

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L87-L100)

It is recommended to Update it's implementation as given below to save more than 2100 gas:

Mitigated Code:

```solidity
File: SubprotocolRegistry.sol

      SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
        if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
        SubprotocolData storage subprotocolData = subprotocols[_name];
        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
            revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L87-L100)

### [GAS-2] UNNECESSARY IF-ELSE CONDITION USED

The `else` part can handle all the condition including condition when `lengthBeforeAddition` is equal to zero. So `if` part is unnecessary utilising gas. It is recommended to Mitigate this by removing if-else condition and only utilising Line 222 to Line 225.

*Instance (1):*
```solidity
File: CidNFT.sol

215:    if (lengthBeforeAddition == 0) {
216:        uint256[] memory nftIDsToAdd = new uint256[](1);
217:        nftIDsToAdd[0] = _nftIDToAdd;
218:        activeData.values = nftIDsToAdd;
219:        activeData.positions[_nftIDToAdd] = 1; // Array index + 1
220:    } else {
221:        // Check for duplicates
222:        if (activeData.positions[_nftIDToAdd] != 0)
223:            revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
224:        activeData.values.push(_nftIDToAdd);
225:        activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
226:    }

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L215-L220)

### [GAS-3] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.

*Instances (2):*
```solidity
File: CidNFT.sol

148:      _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back

150:      for (uint256 i = 0; i < _addList.length; ++i) {

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L148)

### [GAS-4] USING BOOLS FOR STORAGE INCURS OVERHEAD

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from `false` to `true`, after having been `true` in the past.

*Instances (3):*
```solidity
File: CidNFT.sol

34:      bool ordered;
35:      bool primary;
36:      bool active;

```
[Link to Code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L148)