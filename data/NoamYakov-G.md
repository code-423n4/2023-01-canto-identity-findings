
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Using `storage` instead of `memory` for structs/arrays saves gas | 1 |  2100 |
| [G&#x2011;02] | State variables only set in the constructor should be declared `immutable` | 1 |  2097 |
| [G&#x2011;03] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 2 |  90 |
| [G&#x2011;04] | Avoid contract existence checks by using low level calls | 4 |  400 |

Total: 8 instances over 4 issues with **4687 gas** saved.

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions.

## Gas Optimizations

### [G&#x2011;01]  Using `storage` instead of `memory` for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declaring the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

*There is 1 instance of this issue:*

```solidity
File: src\SubprotocolRegistry.sol

/// @audit `nftAddress`, `ordered`, `primary` and `active` aren't used
89          SubprotocolData memory subprotocolData = subprotocols[_name];
```

### [G&#x2011;02]  State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmaccess - **100 gas**) with a `PUSH32` (**3 gas**). 

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*There is 1 instance of this issue:*

```solidity
File: src\CidNFT.sol

127         baseURI = _baseURI;
```

### [G&#x2011;03]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

*There are 2 instances of this issue:*

```solidity
File: src\CidNFT.sol

148         _mint(msg.sender, ++numMinted); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back

150         for (uint256 i = 0; i < _addList.length; ++i) {
```

### [G&#x2011;04]  Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

*There are 4 instances of this issue:*

```solidity
File: src\AddressRegistry.sol

/// @audit ownerOf()
43          if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
```

```solidity
File: src\CidNFT.sol

/// @audit getSubprotocol()
172         SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
173             _subprotocolName
174         );

/// @audit getSubprotocol()
244         SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
245             _subprotocolName
246         );
```

```solidity
File: src\SubprotocolRegistry.sol

/// @audit supportsInterface()
93          if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
```
