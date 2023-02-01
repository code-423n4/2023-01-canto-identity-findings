
### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)
#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::37 => string public baseURI;
2023-01-canto-identity/src/CidNFT.sol::67 => uint256 public numMinted;
```





### [G02] `++i/i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for` and `while` loops


#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::150 => for (uint256 i = 0; i < _addList.length; ++i) {
```





### [G03] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed
#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::189 => uint96 subprotocolFee = subprotocolData.fee;
2023-01-canto-identity/src/SubprotocolRegistry.sol::32 => uint96 fee;
2023-01-canto-identity/src/SubprotocolRegistry.sol::52 => uint96 fee
2023-01-canto-identity/src/SubprotocolRegistry.sol::85 => uint96 _fee
```




### [G04] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function


#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::176 => revert SubprotocolDoesNotExist(_subprotocolName);
2023-01-canto-identity/src/CidNFT.sol::182 => revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
2023-01-canto-identity/src/CidNFT.sol::196 => revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
```


### [G05] Use a more recent version of solidity

#### Impact
Use a solidity version of at least 0.8.10 to have external calls skip
 contract existence checks if the external call has a return value
#### Findings:
```
2023-01-canto-identity/src/AddressRegistry.sol::2 => pragma solidity >=0.8.0;
2023-01-canto-identity/src/CidNFT.sol::2 => pragma solidity >=0.8.0;
2023-01-canto-identity/src/SubprotocolRegistry.sol::2 => pragma solidity >=0.8.0;
```


### [G06] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate

#### Impact
Saves a storage slot for the mapping. Depending on the circumstances 
and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. 
Reads and subsequent writes can also be cheaper when a function requires
 both values and they both fit in the same storage slot
#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::42 => mapping(uint256 => uint256) positions;
2023-01-canto-identity/src/CidNFT.sol::48 => mapping(uint256 => uint256) ordered;
```







