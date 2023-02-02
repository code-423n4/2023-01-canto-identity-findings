# QA report

## Author: rotcivegaf

## Non-critical

### [N‑01] Typo

```solidity
File: src/CidNFT.sol

/// @audit: `existant` to `existent`
274:            uint256 arrayPosition = activeData.positions[_nftIDToRemove]; // Index + 1, 0 if non-existant

/// @audit: `subprotocl` to `subprotocol`
306:    /// @return subprotocolNFTID The ID of the primary NFT at the queried subprotocl / CID NFT. 0 if it does not exist

/// @audit: `subprotocl` to `subprotocol`
318:    /// @return subprotocolNFTIDs The ID of the primary NFT at the queried subprotocl / CID NFT. 0 if it does not exist
```

### [N‑02] Use scientific notation

[Look in the solidity documentation](https://docs.soliditylang.org/en/v0.8.17/types.html#rational-and-integer-literals)

```solidity
File: src/SubprotocolRegistry.sol

/// @audit: `100 * 10**18` to `100e18`
17:    uint256 public constant REGISTER_FEE = 100 * 10**18;
```

### [N‑03] Named imports can be used

`import "<CONTRACT>.sol";` => `import {X} from "<CONTRACT>.sol";`

```solidity
File: src/CidNFT.sol

5:import "solmate/tokens/ERC20.sol";

6:import "solmate/utils/SafeTransferLib.sol";

7:import "./SubprotocolRegistry.sol";
```

```solidity
File: src/SubprotocolRegistry.sol

4:import "solmate/tokens/ERC721.sol";

5:import "solmate/tokens/ERC20.sol";

6:import "solmate/utils/SafeTransferLib.sol";

7:import "./CidSubprotocolNFT.sol";
```

```solidity
File: src/AddressRegistry.sol

4:import "solmate/tokens/ERC721.sol";
```

### [N‑04] Move validation to the top of the function

First input parameters validations
Second others validations

```solidity
File: src/SubprotocolRegistry.sol

88:        if (!(_ordered || _primary || _active)) revert NoTypeSpecified(_name);

90:        if (subprotocolData.owner != address(0)) revert SubprotocolAlreadyExists(_name, subprotocolData.owner);

93:        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
94:            revert NotASubprotocolNFT(_nftAddress);
```

### [N‑05] Don't use the same name for two different structs

The struct `SubprotocolData` inside of the contracts [CidNFT](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/CidNFT.sol#L46-L53) and [SubprotocolRegistry](https://github.com/code-423n4/2023-01-canto-identity/blob/bf705da36e5b2adc93d46064a07ad0a21f9391e1/src/SubprotocolRegistry.sol#L27-L37) have the same name with different parameters

### [N‑07] Declare address as specific contract type

Specify the type in the parameter/contract instead of cast it, after

```solidity
File: src/AddressRegistry.sol

/// @audit: `address` to `ERC721`
14:    address public immutable cidNFT;
36:    constructor(address _cidNFT) {
/// @audit: Remove cast
43:        if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
```

```solidity
File: src/CidNFT.sol

/// @audit: `address` to `ERC20`
124:        address _noteContract,
/// @audit: Remove cast
129:        note = ERC20(_noteContract);
```

```solidity
File: src/SubprotocolRegistry.sol

/// @audit: `address` to `ERC721`
33:        address nftAddress;
48:        address indexed nftAddress,
60:    error NotASubprotocolNFT(address nftAddress);
83:        address _nftAddress,
/// @audit: Remove cast
93:        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))

/// @audit: `address` to `ERC20`
65:    constructor(address _noteContract, address _cidFeeWallet) {
/// @audit: Remove cast
66:        note = ERC20(_noteContract);
```

### [N‑07] Lint

Remove space:

```solidity
File: src/CidNFT.sol

153:\n
```

### [N‑08] Unused error

```solidity
File: src/CidNFT.sol

106:    error NotAuthorizedForSubprotocolNFT(address caller, uint256 subprotocolNFTID);
```

### [N‑09] Move the test folder to the root repository folder

### [N-10] Constants should be defined and documented rather than using magic numbers

```solidity
File: src/CidNFT.sol

191:            uint256 cidFee = (subprotocolFee * CID_FEE_BPS) / 10_000;
```

### [N-11] Assign struct as an object instead as assignments of parameters

```solidity
File: src/CidNFT.sol

From:
91:        subprotocolData.owner = msg.sender;
92:        subprotocolData.fee = _fee;

95:        subprotocolData.nftAddress = _nftAddress;
96:        subprotocolData.ordered = _ordered;
97:        subprotocolData.primary = _primary;
98:        subprotocolData.active = _active;
99:        subprotocols[_name] = subprotocolData;

To:
            subprotocols[_name] = SubprotocolData({
                owner: msg.sender,
                fee: _fee,
                nftAddress: _nftAddress,
                ordered: _ordered,
                primary: _primary,
                active: _active
            });
```