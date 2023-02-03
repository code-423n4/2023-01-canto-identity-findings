# Gas Report
## Finding Summary

1. [**Reversion Can Be Done Earlier**](#1-Reversion-Can-Be-Done-Earlier) (-73726, -9.434%)
2. [**Convert To AND and Split Reversion**](#2-Convert-To-AND-and-Split-Reversion) (-546, -0.057%)
3. [**uint256 Iterator Checked Each Iteration**](#3-uint256-Iterator-Checked-Each-Iteration) (-315, -0.102%)

## 1. Reversion Can Be Done Earlier

Error checks should revert as early as possible to prevent unnecessary computation. 

*/src/CidNFT.sol*

* On [L183](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L183) `_nftIDToAdd == 0` is checked. `_nftIDToAdd` is a parameter and therefore can be checked at the very beginning of `add`.

*/src/SubprotocolRegistry.sol*

* On [L88](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L88) `!(_ordered || _primary || _active)` is checked. All three variables are parameters and therefore can be checked at the very beginning of `register`.
* On [L93](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L93) `!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId)` is checked. `_nftAddress` is a parameter and therefore can be checked at the very beginning of `register` (please consider possible security ramifications).

## 2. Convert To AND and Split Reversion

[L88](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L88) can be converted to an AND statement and transformed as follows to save gas:

**From**
```solidity
88:	if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
```
**To**
```solidity
if (!_ordered) {
    if (!_primary) {
        if (!_active) revert NoTypeSpecified(_name);
    }
}
```

## 3. uint256 Iterator Checked Each Iteration

A `uint256` iterator will not overflow before the check variable overflows. Unchecking the iterator increment saves gas.

**Example**
```solidity
//From
for (uint256 i; i < len; ++i) {
	//Do Something
}
//To
for (uint256 i; i < len;) {
	//Do Something
	unchecked { ++i; }
}
```

*/src/CidNFT.sol*

```solidity
150:	for (uint256 i = 0; i < _addList.length; ++i) {
```
