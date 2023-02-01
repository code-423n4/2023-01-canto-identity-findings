# GAS OPTIMIZATION REPORT

---

### Summary of optimizations


| Number     | Issue details                                                                                                                                         | Instances |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------: |
| [G-1](#G1) | `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops |     1     |
| [G-2](#G3) | Using storage instead of memory for structs/arrays saves gas.                                                                                         |     1     |
| [G-3](#G4) | Usage of`uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.                                                                              |     4     |
| [G-4](#G5) | Import only what you need.                                                                                                                            |     8     |

*Total: 4 issues.*

---

### Gas Optimizations

### <a id=G1>[G-1]</a> `++i/i++` should be `unchecked{++i}`/`unchecked{i++}` when  it is not possible for them to overflow, for example when used in `for` and `while` loops

##### Description

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck [cannot be made inline](https://github.com/ethereum/solidity/issues/10695).
Example for loop:

```Solidity
for (uint i = 0; i < length; i++) {
// do something that doesn't change the value of i
}
```

In this example, the for loop post condition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body `is 2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. You should manually do this by:

```Solidity
for (uint i = 0; i < length; i = unchecked_inc(i)) {
	// do something that doesn't change the value of i
}

function unchecked_inc(uint i) returns (uint) {
	unchecked {
		return i + 1;
	}
}
```

Or just:

```Solidity
for (uint i = 0; i < length;) {
	// do something that doesn't change the value of i
	unchecked { i++; 	}
}
```

Note that it’s important that the call to `unchecked_inc` is inlined. This is only possible for solidity versions starting from `0.8.2`.

Gas savings: roughly speaking this can save 30-40 gas per loop iteration. For lengthy loops, this can be significant!
(This is only relevant if you are using the default solidity checked arithmetic.)

##### Recommendation

Use the `unchecked` keyword

##### *Instances (1):*

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L150 )

```solidity
150: for (uint256 i = 0; i < _addList.length; ++i) {
```

### <a id=G3>[G-2]</a> Using storage instead of memory for structs/arrays saves gas.

##### Description

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.
Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct([src](https://ethereum.stackexchange.com/questions/128380/why-using-storage-keyword-instead-of-memory-cost-less-gas))

##### Recommendation

Use storage for `struct/array`

##### *Instances (1):*

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L216 )

```solidity
216: uint256[] memory nftIDsToAdd = new uint256[](1);
```

### <a id=G4>[G-3]</a> Usage of `uint`s/`int`s smaller than 32 bytes (256 bits) incurs overhead.

##### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

##### Recommendation

Use `uint256` instead.

##### *Instances (4):*

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L189 )

```solidity
189: uint96 subprotocolFee = subprotocolData.fee;
```

File: [2023-01-canto-identity/src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L32 )

```solidity
32: uint96 fee;
52: uint96 fee
85: uint96 _fee
```

### <a id=G5>[G-4]</a> Import only what you need.

##### Description

Importing only what is needed is ideal for gas optimization.

##### Recommendation

Import only what is necessary from file.

##### *Instances (8):*

File: [2023-01-canto-identity/src/AddressRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L4 )

```solidity
4: import "solmate/tokens/ERC721.sol";
```

File: [2023-01-canto-identity/src/CidNFT.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L5 )

```solidity
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./SubprotocolRegistry.sol";
```


File: [2023-01-canto-identity/src/SubprotocolRegistry.sol](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L4 )

```solidity
4: import "solmate/tokens/ERC721.sol";
5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./CidSubprotocolNFT.sol";
```