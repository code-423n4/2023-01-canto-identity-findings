
# Gas
| | Issue index|
| ----------- | ----------- |
| 1 | [Variable can be declared outside the loop to save gas](#variable-can-be-declared-outside-the-loop-to-save-gas) |
| 2 | [`++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#`++i-/-i++`-should-be-`unchecked{++i}-/-unchecked{i++}`-when-it-is-not-possible-for-them-to-overflow,-as-is-the-case-when-used-in-for-loop-and-while-loops) |
## Variable can be declared outside the loop to save gas

Can be declared outside of the loop and then reassigned each iteration

- [CidNFT.sol#L152](https://github.com/code-423n4/2023-01-canto-identity/blob/d7843c5d7ab731ba959cdb1389ba6bd5a2f5bbd3/src/CidNFT.sol#L152)
bool success, /*bytes memory result*/


## `++i / i++` should be `unchecked{++i} / unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

- [CidNFT.sol#L150](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L150)
        for (uint256 i = 0; i < _addList.length; ++i) {


