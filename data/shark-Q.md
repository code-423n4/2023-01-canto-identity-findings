## 1. Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)

File: `SubprotocolRegistry.sol` [Line 17](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17)

```solidity
    uint256 public constant REGISTER_FEE = 100 * 10**18;
```

can be changed to:

```solidity
    uint256 public constant REGISTER_FEE = 100 * 1e18;
```

## 2. Incomplete NatSpec

It is recommended that contracts be thoroughly documented using [NatSpec](https://docs.soliditylang.org/en/develop/natspec-format.html).

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L327-L337

```solidity
File: src/CidNFT.sol

/// @audit Missing '@param' _nftIDToCheck
327    /// @notice Check if a provided NFT ID is included in the active data list that is associated with a CID NFT / Subprotocol
328    /// @param _cidNFTID ID of the CID NFT to query
329    /// @param _subprotocolName Name of the subprotocol to query
330    /// @return nftIncluded True if the NFT ID is in the list
331    function activeDataIncludesNFT(
332        uint256 _cidNFTID,
333        string calldata _subprotocolName,
334        uint256 _nftIDToCheck
335    ) external view returns (bool nftIncluded) {
336        nftIncluded = cidData[_cidNFTID][_subprotocolName].active.positions[_nftIDToCheck] != 0;
337    }
```

All events are missing NatSpec completely:

File: `AddressRegistry.sol` [Line 26-27](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L26-L27)

```solidity
26    event CIDNFTAdded(address indexed user, uint256 indexed cidNFTID);
27    event CIDNFTRemoved(address indexed user, uint256 indexed cidNFTID);
```

File: `SubprotocolRegistry.sol` [Line 45-53](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L45-L53)

```solidity
45    event SubprotocolRegistered(
46        address indexed registrar,
47        string indexed name,
48        address indexed nftAddress,
49        bool ordered,
50        bool primary,
51        bool active,
52        uint96 fee
53    );
```

File: `CidNFT.sol` [Line 75-95](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L75-L95)

```solidity
75    event OrderedDataAdded(
76        uint256 indexed cidNFTID,
77        string indexed subprotocolName,
78        uint256 indexed key,
79        uint256 subprotocolNFTID
80    );
81    event PrimaryDataAdded(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
82    event ActiveDataAdded(
83        uint256 indexed cidNFTID,
84        string indexed subprotocolName,
85        uint256 subprotocolNFTID,
86        uint256 arrayIndex
87    );
88    event OrderedDataRemoved(
89        uint256 indexed cidNFTID,
90        string indexed subprotocolName,
91        uint256 indexed key,
92        uint256 subprotocolNFTID
93    );
94    event PrimaryDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
95    event ActiveDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
```

## 3. Use a more recent version of solidity

Use a solidity version of at least `0.8.12` to get `string.concat()` to be used instead of `abi.encodePacked(<str>,<str>)`

```solidity
File: src/CidNFT.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2

```solidity
File: src/SubprotocolRegistry.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2

```solidity
File: src/AddressRegistry.sol

2: pragma solidity >=0.8.0;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2

## 4. Use `delete` to clear variables instead of zero assignment, i.e. (0, 0x0, false)

A better way to indicate that you are clearing a variable is to use the [`delete`](https://docs.soliditylang.org/en/v0.8.17/types.html#delete) keyword.

For example:

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L284

```solidity
    activeData.positions[_nftIDToRemove] = 0;
```

could be changed to:

```solidity
    delete activeData.positions[_nftIDToRemove];
```

## 5. Typos

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L274

```solidity
File: src/CidNFT.sol

        /// @audit non-existant
274:    uint256 arrayPosition = activeData.positions[_nftIDToRemove]; // Index + 1, 0 if non-existant
```

## 6. Use named imports

Using named imports can improve code readability.

For example:

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L5

```solidity
File: src/CidNFT.sol

5: import "solmate/tokens/ERC20.sol";
6: import "solmate/utils/SafeTransferLib.sol";
7: import "./SubprotocolRegistry.sol";
```

can be changed to:

```solidity
File: src/CidNFT.sol

5: import {ERC20} from "solmate/tokens/ERC20.sol";
6: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
7: import {SubprotocolRegistry} from "./SubprotocolRegistry.sol";
```

rest of the instances:

- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L4-L7
- https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L4

## 7. Checks Effects Interactions Pattern is not used

The [Checks Effects Interactions Pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) recommends executing external calls after state changes to prevent reentrancy bugs.

For example:

```diff
    function register(
        bool _ordered,
        bool _primary,
        bool _active,
        address _nftAddress,
        string calldata _name,
        uint96 _fee
    ) external {
-       SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
        if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);
        SubprotocolData memory subprotocolData = subprotocols[_name];
        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
            revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        subprotocols[_name] = subprotocolData;
        emit SubprotocolRegistered(msg.sender, _name, _nftAddress, _ordered, _primary, _active, _fee);
+       SafeTransferLib.safeTransferFrom(note, msg.sender, cidFeeWallet, REGISTER_FEE);
    }
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L79-L101

## 8. `constant` variables should be defined rather than using magic numbers

```solidity
File: src/CidNFT.sol

/// @audit 10_000
191:    uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L191
