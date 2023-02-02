# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     1     |

## [G-001] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:1

[src/CidNFT.sol#L216](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L216)

```solidity
216:    uint256[] memory nftIDsToAdd = new uint256[](1);
```

### Recommendation

Using `storage` instead of `memory`
