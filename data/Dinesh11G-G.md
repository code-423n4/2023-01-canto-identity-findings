### 1st BUG
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::215 => if (lengthBeforeAddition == 0) {
2023-01-canto-identity/src/CidNFT.sol::225 => activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
2023-01-canto-identity/src/CidNFT.sol::227 => emit ActiveDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd, lengthBeforeAddition);
2023-01-canto-identity/src/CidNFT.sol::276 => uint256 arrayLength = activeData.values.length;
```
#### Tools used
Manual VS code review

### 2nd BUG
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::6 => import "solmate/utils/SafeTransferLib.sol";
2023-01-canto-identity/src/SubprotocolRegistry.sol::6 => import "solmate/utils/SafeTransferLib.sol";
```
#### Tools used
Manual VS code review