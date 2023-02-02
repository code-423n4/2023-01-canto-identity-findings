## 01. Hardcoded `REGISTER_FEE` assumes that the `NOTE` token has 18 decimals

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17)

```javascript
// SubprotocolRegistry.sol
...

/// @notice Fee for registering a new subprotocol (100 $NOTE)
uint256 public constant REGISTER_FEE = 100 * 10**18;
```

### Impact

The initialization of `REGISTER_FEE` assumes that the `NOTE` token has 18 decimals. 
However, the implementation specs of `NOTE` haven’t been disclosed and there is no clear indication about its actual number of decimals. 
In case it’s different from 18, the fee will change accordingly and will mismatch the mentioned value of `100 $NOTE`.

### Recommended Mitigation Steps

- The contract deployer has to make sure that `NOTE` token has 18 decimals; or
- If possible, inherit `NOTE` contract from `IERC20Metadata` interface and instead of hardcoding `REGISTER_FEE`, the following approach could be taken:
    
    ```javascript
    uint256 public immutable REGISTER_FEE;
    ...
    constructor() {
        ...
        uint8 _decimals = IERC20Metadata(address(note)).decimals();
    
        /// @notice Fee for registering a new subprotocol (100 $NOTE)
        uint256 public constant REGISTER_FEE = 100 * _decimals;
        ...
    }
    ```
---

## 02. Function parameter validation should be performed prior to the implementation of other functionality

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L165-L183](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L165-L183)

```javascript
// CidNFT.sol

...
function add(
    uint256 _cidNFTID,
    string calldata _subprotocolName,
    uint256 _key,
    uint256 _nftIDToAdd,
    AssociationType _type
) external {
    SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
        _subprotocolName
    );
    address subprotocolOwner = subprotocolData.owner;
    if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);
    address cidNFTOwner = ownerOf[_cidNFTID];
    if (
        cidNFTOwner != msg.sender &&
        getApproved[_cidNFTID] != msg.sender &&
        !isApprovedForAll[cidNFTOwner][msg.sender]
    ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
    if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols
    ...
```

### Impact

The following statement validates the `_nftIDtoAdd` param:

```javascript
if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols();
```

However, it is placed too low in the sequence of code lines, allowing a great deal of functionality to run ineffectively in case it fails.

### Recommended Mitigation Steps

Shift the respective statement above the other lines of code as shown below:

```javascript
// CidNFT.sol

...
function add(
    uint256 _cidNFTID,
    string calldata _subprotocolName,
    uint256 _key,
    uint256 _nftIDToAdd,
    AssociationType _type
) external {
    ------------------------------------------------------------------------------------------------------------     
    |                                            shift the if statement here                                                              
    ------------------------------------------------------------------------------------------------------------           
    SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(   
        _subprotocolName                                                                                                                                       
    );                                                                                                                                                                        
    address subprotocolOwner = subprotocolData.owner;                                                              
    if (subprotocolOwner == address(0)) revert SubprotocolDoesNotExist(_subprotocolName);                          
    address cidNFTOwner = ownerOf[_cidNFTID];                                                                      
    if (                                                                                                           
        cidNFTOwner != msg.sender &&                                                                              
        getApproved[_cidNFTID] != msg.sender &&                                                                   
        !isApprovedForAll[cidNFTOwner][msg.sender]                                                                
    ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);                                                            
 -----------------------------------------------------------------------------------------------------------------------       
 |  if (_nftIDToAdd == 0) revert NFTIDZeroDisallowedForSubprotocols(); // ID 0 is disallowed in subprotocols |----> shift above
 -----------------------------------------------------------------------------------------------------------------------
...
```

---

## 03. `immutable` state variables `cidFeeWallet` and `note` declared and initialized with the same values redundantly in the constructors of two different contracts

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L65)

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L119)

```javascript
// SubprotocolRegistry.sol

...
contract SubprotocolRegistry {
    ...
    /// @notice Reference to the $NOTE TOKEN
    ERC20 public immutable note;

    /// @notice Wallet that receives fees paid when registering
    address public immutable cidFeeWallet;
    ...

    /// @notice Sets the reference to the $NOTE contract
    /// @param _noteContract Address of the $NOTE contract
    /// @param _cidFeeWallet Address of the wallet that receives the fees
    constructor(address _noteContract, address _cidFeeWallet) {
        note = ERC20(_noteContract);
        cidFeeWallet = _cidFeeWallet;
    }
    ...
```

```javascript
// CidNFT.sol

...
contract CidNFT is ERC721, ERC721TokenReceiver {
    ...
    /// @notice Wallet that receives CID fees
    address public immutable cidFeeWallet;

    /// @notice Reference to the NOTE TOKEN
    ERC20 public immutable note;
    ...

    /// @notice Sets the name, symbol, baseURI, and the address of the auction factory
    /// @param _name Name of the NFT
    /// @param _symbol Symbol of the NFT
    /// @param _baseURI NFT base URI. {id}.json is appended to this URI
    /// @param _cidFeeWallet Address of the wallet that receives the fees
    /// @param _noteContract Address of the $NOTE contract
    /// @param _subprotocolRegistry Address of the subprotocol registry
    constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI,
        address _cidFeeWallet,
        address _noteContract,
        address _subprotocolRegistry
    ) ERC721(_name, _symbol) {
        baseURI = _baseURI;
        cidFeeWallet = _cidFeeWallet;
        note = ERC20(_noteContract);
        subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
    }
    ...
```

### Impact

The `immutable` state variables `cidFeeWallet` and `note` are declared and initialized with the same values in the constructors of two separate contracts: `SubprotocolRegistry` and `CidNFT`. 
Given that same `immutable`s are defined in two different places, this approach is not only suboptimal, but error-prone as well.

### Recommended Mitigation Steps

Amend the project architecture by initializing the respective variables only once, making them accessible throughout the entire codebase.

---

## 04. The name of `CIDNFTAdded` event could confuse the end user

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L42](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L42)

```javascript
// AddressRegistry.sol
...

/// @notice Register a CID NFT to the address of the caller. NFT has to be owned by the caller
/// @dev Will overwrite existing registration if any exists
function register(uint256 _cidNFTID) external {
    if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
        // We only guarantee that a CID NFT is owned by the user at the time of registration
        // ownerOf reverts if non-existing ID is provided
        revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
    cidNFTs[msg.sender] = _cidNFTID;
    emit CIDNFTAdded(msg.sender, _cidNFTID);
}
...
```

### Impact

Based on the logical context, the event `CIDNFTAdded` is supposed to be emitted whenever a new `_cidNFT` is added, which is indeed the case in `register` function, 
but only when `register` function is called for the first time by the same `msg.sender`. 

Starting from the second `register` call of the respective `msg.sender`, `_cidNFT` is not simply added, it overwrites the previously stored `_cidNFT`, which again is the intended behavior according to NatSpec. 

However, the issue here is that the user could be easily confused by the name of the event, i.e. `CIDNFTAdded`, which doesn’t hint in any way that the previous `_cidNFT` was overwritten.

Additionally, the user also might be interested in knowing which exactly `_cidNFT` has been overwritten.

### Recommended Mitigation Steps

- Use a suitable event name
- Consider including the ID of the overwritten `_cidNFT` as an additional event parameter

---

## 05. Misleading `cidNFT` state variable name

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L14](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L14)

```javascript
// AddressRegistry.sol

contract AddressRegistry {
	...

	/// @notice Address of the CID NFT
	address public immutable cidNFT;
		
	/// @notice Stores the mappings of users to their CID NFT
	mapping(address => uint256) private cidNFTs;
	...

	/// @param _cidNFT Address of the CID NFT contract
	constructor(address _cidNFT) {
	    cidNFT = _cidNFT;
	}

	...
```

### Impact

Examining the declaration of the state variables `cidNFT` and `cidNFTs`, a certain degree of ambiguity arises as to the actual purpose of `cidNFT`. 

It is getting clarified only when one reads the `constructor`'s NatSpec.

### Recommended Mitigation Steps

Rename `cidNFT` to `cidNFTContract` or something relevant.

---

## 06. State variable `baseURI` should be `immutable`

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L37](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L37)

```javascript
// CidNFT.sol

...
contract CidNFT is ERC721, ERC721TokenReceiver {
    ...
    string public baseURI;
    ...
	
    /// @notice Sets the name, symbol, baseURI, and the address of the auction factory
    /// @param _name Name of the NFT
    /// @param _symbol Symbol of the NFT
    /// @param _baseURI NFT base URI. {id}.json is appended to this URI
    /// @param _cidFeeWallet Address of the wallet that receives the fees
    /// @param _noteContract Address of the $NOTE contract
    /// @param _subprotocolRegistry Address of the subprotocol registry
    constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI,
        address _cidFeeWallet,
        address _noteContract,
        address _subprotocolRegistry
    ) ERC721(_name, _symbol) {
        baseURI = _baseURI;
        cidFeeWallet = _cidFeeWallet;
        note = ERC20(_noteContract);
        subprotocolRegistry = SubprotocolRegistry(_subprotocolRegistry);
    }

    /// @notice Get the token URI for the provided ID
    /// @param _id ID to retrieve the URI for
    /// @return tokenURI The URI of the queried token (path to a JSON file)
    function tokenURI(uint256 _id) public view override returns (string memory) {
        if (ownerOf[_id] == address(0))
            // According to ERC721, this revert for non-existing tokens is required
            revert TokenNotMinted(_id);
        return string(abi.encodePacked(baseURI, _id, ".json"));
    }
    ...
```

### Impact

As can be seen above, the state variable `string public baseURI` is initialized in the `constructor`. After that, its value remains intact. 

### Recommended Mitigation Steps

`baseURI` should be `immutable`:

```javascript
string public immutable baseURI;
```

---

## 07. Use scientific notation in lieu of exponentiation

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17)

```diff
// SubprotocolRegistry.sol

- uint256 public constant REGISTER_FEE = 100 * 10**18;
+ uint256 public constant REGISTER_FEE = 100 * 1e18;
```

---

## 08. Missing `@param` in NatSpec

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L40](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L40)

```javascript
// AddressRegistry.sol

/// @notice Register a CID NFT to the address of the caller. NFT has to be owned by the caller
/// @dev Will overwrite existing registration if any exists
function register(uint256 _cidNFTID) external {
    if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
        // We only guarantee that a CID NFT is owned by the user at the time of registration
        // ownerOf reverts if non-existing ID is provided
        revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
    cidNFTs[msg.sender] = _cidNFTID;
    emit CIDNFTAdded(msg.sender, _cidNFTID);
}
```

Missing `@param _cidNFTID ..........` in NatSpec.

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L327](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L327)

```javascript
// CidNFT.sol

/// @notice Check if a provided NFT ID is included in the active data list that is associated with a CID NFT / Subprotocol
/// @param _cidNFTID ID of the CID NFT to query
/// @param _subprotocolName Name of the subprotocol to query
/// @return nftIncluded True if the NFT ID is in the list
function activeDataIncludesNFT(
    uint256 _cidNFTID,
    string calldata _subprotocolName,
    uint256 _nftIDToCheck
) external view returns (bool nftIncluded) {
    nftIncluded = cidData[_cidNFTID][_subprotocolName].active.positions[_nftIDToCheck] != 0;
}
```

Missing `@param _nftIDToCheck ..........` in NatSpec.

---

## 09. Missing NatSpec

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L339](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L339)

```javascript
// CidNFT.sol

function onERC721Received(
    address, /*operator*/
    address, /*from*/
    uint256, /*id*/
    bytes calldata /*data*/
) external pure returns (bytes4) {
    return ERC721TokenReceiver.onERC721Received.selector;
}
```

---

## 10. Misleading NatSpec

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L26](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L26)

```javascript
// SubprotocolRegistry.sol

/// @notice Data that is associated with a subprotocol.
/// @dev Data types are chosen such that all data fits in one slot
struct SubprotocolData {
    /// @notice Owner (registrant) of the subprotocol
    address owner;
    /// @notice Optional cost in NOTE to add an NFT
    /// @dev Maximum value is (2^96 - 1) / 10^18 =~ 80 billion. Zero for no fee
    uint96 fee;
    address nftAddress;
    bool ordered;
    bool primary;
    bool active;
}
```

The NatSpec says: 

```javascript
/// @dev Data types are chosen such that all data fits in one slot
```

However, the storage of `SubprotocolData` struct occupies two slots rather than one.

### Proof of Concept

Here is the storage allocation layout of `SubprotocolData` struct members:

| Struct member | Size in bytes  | Slot number (within struct) |
| --- | --- | --- |
| address owner | 20 | 0 |
| uint96 fee | 12 | 0 |
| address nftAddress | 20 | 1 |
| bool ordered | 1 | 1 |
| bool primary | 1 | 1 |
| bool active | 1 | 1 |

---

## 11. Error in NavSpec

### Context

[https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L112](https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L112)

```javascript
// CidNFt.sol

/// @notice Sets the name, symbol, baseURI, and the address of the auction factory
/// @param _name Name of the NFT
/// @param _symbol Symbol of the NFT
/// @param _baseURI NFT base URI. {id}.json is appended to this URI
/// @param _cidFeeWallet Address of the wallet that receives the fees
/// @param _noteContract Address of the $NOTE contract
/// @param _subprotocolRegistry Address of the subprotocol registry
constructor(
    string memory _name,
    ...
```

The first line of the NatSpec says:

```javascript
/// @notice Sets the name, symbol, baseURI, and the address of the auction factory
```

However, there is no `auction factory` in the project.