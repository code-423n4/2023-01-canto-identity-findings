1. In CidNFT.sol line 37, baseURI string can be set as immutable since it is never changed. https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L37


2. In CidNFT.sol line 150 the `++i` in the for loop definition can be moved to an unchecked block: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150

from:
```solidity
for (uint256 i = 0; i < _addList.length; ++i) {
    // do stuff
}
```
to:
```solidity
for (uint256 i = 0; i < _addList.length;) {
    // do stuff
    unchecked { ++i; }
}
```

3. In CidNFT.sol line 279 to line 281 the entire thing can be moved to an unchecked block, since even if it underflowed it would revert with: index out of bounds: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L279-L281

from:
```solidity
uint256 befSwapLastNFTID = activeData.values[arrayLength - 1];
activeData.values[arrayPosition - 1] = befSwapLastNFTID;
activeData.positions[befSwapLastNFTID] = arrayPosition;
```
to:
```solidity
unchecked {
    uint256 befSwapLastNFTID = activeData.values[arrayLength - 1];
    activeData.values[arrayPosition - 1] = befSwapLastNFTID;
    activeData.positions[befSwapLastNFTID] = arrayPosition;
}
```

4. In CidNFT.sol line 225 the addition can be moved to unchecked block. Since it is incrementing u256 it should never overflow. If it does there are much bigger problems than overflowing so this doesn't have to be checked: https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L225

from:
```solidity
activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
```

to:
```solidity
unchecked {
    activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
}