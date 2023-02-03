# QA Report


## Low Issues


|             | Issue                                              | Instances|
|-------------|----------------------------------------------------|----------|
| [L-1](#L-1) | `getActiveData()` can revert for valid active data | 1 |
| [L-2](#L-2) | Immutable variables should have a check for address(0) | 3 |
| [L-3](#L-3) | Misleading comment for `CID_FEE_BPS` | 1 |
| [L-4](#L-4) | Can register a subprotocol with no name | 1 |

## Non Critical Issues


|               | Issue                                              | Instances|
|---------------|----------------------------------------------------|----------|
| [NC-1](#NC-1) | Scientific Notation | 4 |
| [NC-2](#NC-2) | Use a more recent version of Solidity | 3 |
| [NC-3](#NC-3) | Use `bytes.concat()` and `string.concat()` instead of `abi.encodePacked()` | 3 |
| [NC-4](#NC-4) | Imports should be cleaner | 8 |
| [NC-5](#NC-5) | Order of functions not following Solidity standard practice | 1 |



### [L-1] `getActiveData()` can revert for valid active data
There is a smaller limit on the number of elements that can be in a dynamic memory array than the number of elements that can be in [a dynamic storage array](https://blog.soliditylang.org/2020/04/06/memory-creation-overflow-bug/).  Starting from Solidity version 0.6.5 the maximum allocation size for dynamic memory arrays is `2**64-1`, while storage arrays can store up to `2**256-1` elements.
This means `getActiveData` can revert if `cidData[_cidNFTID][_subprotocolName].active.values.length > 2**64 - 1`.

The issue is that "valid" active data list will revert `getActiveData` when queried.

Note that this is extremely unlikely to happen, given that the NFT in question needs to have more than `2**64-1` tokens, and for this number of instances to be added to a specific `cidData`.



*Instances (1)*:
```solidity
File: src/CidNFT.sol
326: function getActiveData(uint256 _cidNFTID, string calldata _subprotocolName)
327:         external
328:         view
329:         returns (uint256[] memory subprotocolNFTIDs)
330:     {
331:         subprotocolNFTIDs = cidData[_cidNFTID][_subprotocolName].active.values;
332:     }

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

To mitigate this, ensure in `add()` the active data length cannot exceed `type(uint64).max`.

```diff
File: src/CidNFT.sol
220:             if (lengthBeforeAddition == 0) {
221:                 uint256[] memory nftIDsToAdd = new uint256[](1);
222:                 nftIDsToAdd[0] = _nftIDToAdd;
223:                 activeData.values = nftIDsToAdd;
224:                 activeData.positions[_nftIDToAdd] = 1; // Array index + 1
+                } else if (lengthBeforeAddition == type(uint64).max) {
+                    revert ActiveArrayFull();
225:             } else {
226:                 // Check for duplicates
227:                 if (activeData.positions[_nftIDToAdd] != 0)
228:                     revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
229:                 activeData.values.push(_nftIDToAdd);
230:                 activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
231:             }
```


### [L-2] Immutable variables should have a check for address(0)
To avoid the need to re-deploy in case an address argument was mistakenly set, add a check for address 0 in constructors for immutable variables
*Instances (3)*:
```solidity
File: src/CidNFT.sol
129:         note = ERC20(_noteContract);
130:         subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);


```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol
66: note = ERC20(_noteContract);

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)

NB: these variables were not mentioned in the Automated Findings list.

### [L-3] Misleading comment for `CID_FEE_BPS`
`CID_FEE_BPS` is charged in `add()` if the subprotocol has a fee.
The comment is however stating that this fee is `charged for every mint`.
This is incorrect, as there is **no minting fee** - ie minting a `CIDNft` is free. The only time a user pays a fee is when adding a subprotocol to their CIDNft **if** the subprotocol in question has a fee.

*Instances (1)*:
```solidity
File: src/CidNFT.sol
16:     /// @notice Fee (in BPS) that is charged for every mint (as a percentage of the mint fee). Fixed at 10%.

17:     uint256 public constant CID_FEE_BPS = 1_000;


```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

### [L-4] Can register a subprotocol with no name
It is currently possible to add a subprotocol in the registry with an empty string as `_name`.
This should not be the case, mainly because it can easily create problems for front-ends querying such subprotocol, and also making it confusing to users - we can imagine how a page would display in such case "This is the subprotocol [blank space]".

Add this test to `SubprotocolRegistry.t.sol`, which passes
```solidity
function testRegisterSubProtocolEmptyName() public {
    vm.startPrank(user1);
    string memory name = "";
    SubprotocolNFT subprotocolNFTOne = new SubprotocolNFT();

    subprotocolRegistry.register(true, false, false, address(subprotocolNFTOne), name, 0);
}
```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)

Add a check to ensure the name is not the empty string

```diff
File: src/SubprotocolRegistry.sol
79:     function register(
80:         bool _ordered,
81:         bool _primary,
82:         bool _active,
83:         address _nftAddress,
84:         string calldata _name,
85:         uint96 _fee
86:     ) external {
+         if (keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked(""))) revert NameCantBeEmpty();
```


### [NC-1] Scientific Notation
It is good practice to use scientific notation - e.g `1e3` instead of using exponents or explicitly writing number of high decimals

*Instances (4)*:
```solidity
File: src/CidNFT.sol

17:     uint256 public constant CID_FEE_BPS = 1_000;

195:             uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol

17:     uint256 public constant REGISTER_FEE = 100 * 10**18;

17:     uint256 public constant REGISTER_FEE = 100 * 10**18;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)

### [NC-2] Use a more recent version of Solidity
 Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>), at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(,), at least 0.8.13 to get the ability to use using for with a list of free functions

*Instances (3)*:
```solidity
File: src/AddressRegistry.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)

```solidity
File: src/CidNFT.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol

2: pragma solidity >=0.8.0;

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)

### [NC-3] Use `bytes.concat()` instead of `abi.encodePacked()`
Since version 0.8.4 for appending bytes or strings, `bytes.concat()` can be used instead of `abi.encodePacked()`

*Instances (2)*:
```solidity
File: src/CidNFT.sol

155:             ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

### [NC-4] Imports should be cleaner
For readability, it is good practice to only import the libraries and contracts your need from a file.

Update your imports as such `import {contract1 , contract2} from "file.sol";`

*Instances (8)*:
```solidity
File: src/AddressRegistry.sol

4: import "lib/solmate/src/tokens/ERC721.sol";

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol)

```solidity
File: src/CidNFT.sol

5: import "lib/solmate/src/tokens/ERC20.sol";

6: import "lib/solmate/src/utils/SafeTransferLib.sol";

7: import "./SubprotocolRegistry.sol";

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)

```solidity
File: src/SubprotocolRegistry.sol

4: import "lib/solmate/src/tokens/ERC721.sol";

5: import "lib/solmate/src/tokens/ERC20.sol";

6: import "lib/solmate/src/utils/SafeTransferLib.sol";

7: import "./CidSubprotocolNFT.sol";

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol)


### [NC-5] Order of functions not following Solidity standard practice
Follow Solidity's [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) for function ordering: constructor, receive/fallback, external, public, internal and private.

*Instances (1)*:
```solidity
File: src/CidNFT.sol
136: function tokenURI(uint256 _id) public view override returns (string memory) {
//@audit this function should be placed after external functions

```
[Link to code](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol)
