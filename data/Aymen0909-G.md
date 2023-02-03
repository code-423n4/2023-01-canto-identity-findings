# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Optimize the `register` function (saves ~1800 Gas) | 1 |
| 2  | Use `unchecked` blocks to save gas  |  2 |
| 3  | Duplicated input check statements should be refactored to a modifier  |  2 |
| 4  | `public` functions not called by the contract should be declared `external` instead |  1 |
| 5  | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops |  1  |


## Findings

### 1- Optimize the `register` function (saves ~1800 Gas) :

The function [register](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L79-L101) in the SubprotocolRegistry can be optimized to save **~1800 Gas** (as shown in the table below) on each call by modifying the following :

* Place the input check statements at the beginning of the function.
* Save `subprotocolData` data directly to `storage` instead of using `memory` first.

Savings : 

|     | min      |    avg    | median  | max  |
| --- |:--------:|:---------:|:-------:|:----:|
| Before | 5690     | 63110 | 54258  | 85453 |
| After  | 1180     | 61268 | 53434  | 84629 |
|     |       | -1842 |  |  | 

Thus the function should be modified as follow :

```
function register(
    bool _ordered,
    bool _primary,
    bool _active,
    address _nftAddress,
    string calldata _name,
    uint96 _fee
) external {
    if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
    if (
        !ERC721(_nftAddress).supportsInterface(
            type(CidSubprotocolNFT).interfaceId
        )
    ) revert NotASubprotocolNFT(_nftAddress);
    SubprotocolData storage subprotocolData = subprotocols[_name];
    if (subprotocolData.owner != address(0))
        revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
    SafeTransferLib.safeTransferFrom(
        note,
        msg.sender,
        cidFeeWallet,
        REGISTER_FEE
    );
    subprotocolData.owner = msg.sender;
    subprotocolData.fee = _fee;
    subprotocolData.nftAddress = _nftAddress;
    subprotocolData.ordered = _ordered;
    subprotocolData.primary = _primary;
    subprotocolData.active = _active;
    emit SubprotocolRegistered(
        msg.sender,
        _name,
        _nftAddress,
        _ordered,
        _primary,
        _active,
        _fee
    );
}
```


### 2- Use `unchecked` blocks to save gas (saves ~40 gas) :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 2 instances of this issue:

File: CidNFT.sol [Line 191](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191)
```
uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

The above operation cannot overflow 2**256 because the value of `subprotocolFee` is always less than 2**96 and `CID_FEE_BPS == 1000`, so the operation should be marked as `unchecked`. 

File: CidNFT.sol [Line 225](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L225)
```
activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
```

The above operation cannot realistically overflow as this whould lead to a very big tokens array (which is not really possible), so the operation should be marked as `unchecked`. 

### 3- Duplicated input check statements should be refactored to a modifier (saves ~400 gas) :

The following check statement is repeated in both `add` and `remove` functions :

```
address cidNFTOwner = ownerOf[_cidNFTID];
if (
    cidNFTOwner != msg.sender &&
    getApproved[_cidNFTID] != msg.sender &&
    !isApprovedForAll[cidNFTOwner][msg.sender]
) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```

Because this check is done on the function caller (msg.sender) it can be placed at the beginning of the function and thus can be refactored into a modifier to save gas, it should be replaced by a `isAllowed` modifier as follow :

```
modifier isAllowed(uint256 _cidNFTID){
    address cidNFTOwner = ownerOf[_cidNFTID];
    if (
        cidNFTOwner != msg.sender &&
        getApproved[_cidNFTID] != msg.sender &&
        !isApprovedForAll[cidNFTOwner][msg.sender]
    ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
    _;
}
```

Savings (saves in total ~400 gas) : 

- add function (saves ~300 gas) :

|     | min      |    avg    | median  | max  |
| --- |:--------:|:---------:|:-------:|:----:|
| Before | 8378 | 50503 | 52935  | 129892 |
| After  | 5460 | 50207 | 52941  | 129906 |
|        |      | -296  |        |        |


- remove function (saves ~100 gas) :

|     | min      |    avg    | median  | max  |
| --- |:--------:|:---------:|:-------:|:----:|
| Before | 6367  | 14082 | 9372   | 26324  |
| After  | 3460  | 13986 | 9378   | 26330 |
|        |       | -96 |        |  |


### 4- `public` functions not called by the contract should be declared `external` instead :

The `public` functions that are not called inside the contract should be declared `external` instead to save gas.

There is 1 instance of this issue:

File: CidNFT.sol [Line 136](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L136)
```
function tokenURI(uint256 _id) public view override returns (string memory)
```

### 5- `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as in the case when used in for & while loops :

There is 1 instance of this issue:

File: CidNFT.sol [Line 150](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150)
```
for (uint256 i = 0; i < _addList.length; ++i) 
```