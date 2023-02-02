## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | State variables only set in the `constructor` should be declared immutable | 1 | 2100 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Setting the `constructor` to `payable` | 3 | 39 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Do not calculate constants | 1 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Using `delete` statement can save gas | 1 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Use hardcoded address instead `address(this)` | 5 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Optimize names to save gas | 3 | 66 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Public Functions To External | 2 | - |
| [GAS&#x2011;8](#GAS&#x2011;8) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 2 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops | 1 | 35 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Use solidity version 0.8.17 to gain some gas boost | 3 | 264 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Using `storage` instead of `memory` saves gas | 1 | 350 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Using `10**X` for constants isn't gas efficient  | 1 | - |

Total: 24 contexts over 11 issues

## Gas Optimizations



### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> State variables only set in the `constructor` should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

#### <ins>Proof Of Concept</ins>

```solidity
127: baseURI = _baseURI
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L127





### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
36: constructor(address _cidNFT)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L36

```solidity
119: constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI,
        address _cidFeeWallet,
        address _noteContract,
        address _subprotocolRegistry
    ) ERC721(_name, _symbol)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L119

```solidity
65: constructor(address _noteContract, address _cidFeeWallet)
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L65





### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

#### <ins>Proof Of Concept</ins>

```solidity
17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L17





### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
284: activeData.positions[_nftIDToRemove] = 0;

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L284





### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
154: ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L154

```solidity
187: nftToAdd.safeTransferFrom(msg.sender, address(this), _nftIDToAdd);

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L187

```solidity
264: nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
264: nftToRemove.safeTransferFrom(address(this), msg.sender, currNFTID);
285: nftToRemove.safeTransferFrom(address(this), msg.sender, _nftIDToRemove);

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L264

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L264

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L285





#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`






### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.




### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function tokenURI(uint256 _id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L136

```solidity
function remove(
        uint256 _cidNFTID,
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToRemove,
        AssociationType _type
    ) public {
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L237






### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
189: uint96 subprotocolFee = subprotocolData.fee;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L189

```solidity
32: uint96 fee;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L32






### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> `++i`/`i++` Should Be `unchecked{++i}`/`unchecked{i++}` When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

#### <ins>Proof Of Concept</ins>


```solidity
150: for (uint256 i = 0; i < _addList.length; ++i) {
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L150





### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Use solidity version 0.8.17 to gain some gas boost

Upgrade to the latest solidity version 0.8.17 to get additional gas savings.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/AddressRegistry.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/CidNFT.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L2





### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Using `storage` instead of `memory` saves gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another `memory` array/struct

#### <ins>Proof Of Concept</ins>


```solidity
89: SubprotocolData memory subprotocolData = subprotocols[_name];

```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L89





### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Using `10**X` for constants isn't gas efficient 

In Solidity, a constant expression in a variable will compute the expression everytime the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form 10**X can be rewritten as 1eX to save the gas cost from the calculation with the exponentiation operator **.

#### <ins>Proof Of Concept</ins>


```solidity
17: uint256 public constant REGISTER_FEE = 100 * 10**18;
```

https://github.com/code-423n4/2023-01-canto-identity/tree/main/src/SubprotocolRegistry.sol#L17



#### <ins>Recommended Mitigation Steps</ins>

Replace 10**X with 1eX



