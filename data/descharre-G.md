# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01       |Make for loops unchecked| 5 |  1 |
|G-02       |Use call instead of delegate call| 3257 | 1 |
|G-03       |Use multiple if statements instead of && | 77 |  2 |
|G-04       |Initialize a static array instead of a dynamic array | 19 |  1  |
|G-05       |Use an unchecked block when operands can't over/underflow | 26 | 2   |


# Details
## [G-01] Make for loops unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow. The longer the array, the more gas you will save.

[CidNFT.sol#L150-L156](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150-L156): `mint()` gas saved: 5
```diff
-       for (uint256 i = 0; i < _addList.length; ++i) {
+       for (uint256 i = 0; i < _addList.length;) {
            (
                bool success, /*bytes memory result*/

            ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
            if (!success) revert AddCallAfterMintingFailed(i);
+           unchecked {
+               i++;
+           }
        }
```
## [G-02] Use call instead of delegate call
When you use call for a function in the same contract it does exactly the same as delegate call. Delegate call is used to execute functions of another contract on behalf of the callers storage. 

[CidNFT.sol#L154](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L154): `mint()` gas saved: 3257
## [G-03] Use multiple if statements instead of &&
It's more gas efficiënt when you use multiple if statements instead of 1 statement with multiple requirements.

[CidNFT.sol#L182](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L182): `add()` gas saved: 39
```solidity
        if (
            cidNFTOwner != msg.sender &&
            getApproved[_cidNFTID] != msg.sender &&
            !isApprovedForAll[cidNFTOwner][msg.sender]
        ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
```
Can be changed to:
```solidity
        if (cidNFTOwner != msg.sender) {
            if (getApproved[_cidNFTID] != msg.sender) {
                if (!isApprovedForAll[cidNFTOwner][msg.sender]) {
                    revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
                }
            }
        }
```
Same can be done for:
- [CidNFT.sol#L250-L254](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L250-L254): `remove()` gas saved: 38
## [G-04] Initialize a static array instead of a dynamic array

[CidNFT.sol#L216)](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L216): `remove()` gas saved: 19
```diff
-    uint256[] memory nftIDsToAdd = new uint256[](1);
-    nftIDsToAdd[0] = _nftIDToAdd;
+    uint256[1] memory nftIDsToAdd = [_nftIDToAdd];
     activeData.values = nftIDsToAdd;
```
## [G-05] Use an unchecked block when operands can't over/underflow 
The only case where division can cause an overflow is the expression type(int).min / (-1). This is not the case for the following division
- [CidNFT.sol#L191](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191): `add()` gas saved: 11

When the calculation is + 1 and the type is uint256 it will take a massive amount of time and gas to make it overflow. And in most cases it's also not possible to overflow due to previous checks.
- [CidNFT.sol#L225](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L225): `add()` gas saved: 15
