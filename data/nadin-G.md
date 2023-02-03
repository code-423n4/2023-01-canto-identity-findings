## Gas Optimizations
Issue  
## [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
## [G-02] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
## [G-03] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
## [G-04] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
## [G-05] USE BYTES32 INSTEAD OF STRING
## [G-06] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE
## [G-07] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
## [G-08] ADDED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT
Total: 51 contexts over 09 issues

## [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance
Contexts: 03
File: main/src/AddressRegistry.sol

    36:    constructor(address _cidNFT) {
    37:        cidNFT = _cidNFT;
    38:    }

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L36-L38

File: main/src/SubprotocolRegistry.sol

    65:    constructor(address _noteContract, address _cidFeeWallet) {
    66:        note = ERC20(_noteContract);
    67:        cidFeeWallet = _cidFeeWallet;
    68:    }

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L65-L68

File: main/src/CidNFT.sol

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

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L119-L131

## [G-02] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
It is not necessary to have both a named return and a return statement.
Contexts: 06
File: main/src/AddressRegistry.sol

    62:    function getCID(address _user) external view returns (uint256 cidNFTID) {

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L62-L64

File: main/src/SubprotocolRegistry.sol

    106:    function getSubprotocol(string calldata _name) external view returns (SubprotocolData memory) {

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L106-L108

File: main/src/CidNFT.sol

    299:    ) external view returns (uint256 subprotocolNFTID) {
    310:        returns (uint256 subprotocolNFTID)
    322:        returns (uint256[] memory subprotocolNFTIDs)
    335:    ) external view returns (bool nftIncluded) {

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L295-L301
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L307-L313
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L319-L325
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L331-L337

## [G-03] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Contexts: 04
File: main/src/SubprotocolRegistry.sol

    32:        uint96 fee;
    52:        uint96 fee
    85:        uint96 _fee

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L32
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L52
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L85

File: main/src/CidNFT.sol

    189:        uint96 subprotocolFee = subprotocolData.fee;

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L189

## [G-04] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.
Contexts: 04
File: main/src/SubprotocolRegistry.sol

    89:        SubprotocolData memory subprotocolData = subprotocols[_name];

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L89

File: main/src/CidNFT.sol

    172:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(
    216:                uint256[] memory nftIDsToAdd = new uint256[](1);
    244:        SubprotocolRegistry.SubprotocolData memory subprotocolData = subprotocolRegistry.getSubprotocol(

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L172
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L216
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L244

## [G-05] USE BYTES32 INSTEAD OF STRING
Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.
Contexts: 31
File: main/src/SubprotocolRegistry.sol

    40:    mapping(string => SubprotocolData) private subprotocols;
    47:        string indexed name,
    58:    error SubprotocolAlreadyExists(string name, address owner);
    59:    error NoTypeSpecified(string name);
    84:        string calldata _name,
    106:     function getSubprotocol(string calldata _name) external view returns (SubprotocolData memory) {

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L40
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L47
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L58
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L59
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L84
https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/SubprotocolRegistry.sol#L106

File: main/src/CidNFT.sol

    37:    string public baseURI;
    70:    mapping(uint256 => mapping(string => SubprotocolData)) internal cidData;
    77:        string indexed subprotocolName,
    81:    event PrimaryDataAdded(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
    84:        string indexed subprotocolName,
    90:        string indexed subprotocolName,
    94:    event PrimaryDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
    95:    event ActiveDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
    102:    error SubprotocolDoesNotExist(string subprotocolName);
    104:    error AssociationTypeNotSupportedForSubprotocol(AssociationType associationType, string subprotocolName);
    107:    error ActiveArrayAlreadyContainsID(uint256 cidNFTID, string subprotocolName, uint256 nftIDToAdd);
    108:    error OrderedValueNotSet(uint256 cidNFTID, string subprotocolName, uint256 key);
    109:    error PrimaryValueNotSet(uint256 cidNFTID, string subprotocolName);
    110:    error ActiveArrayDoesNotContainID(uint256 cidNFTID, string subprotocolName, uint256 nftIDToRemove);
    120:        string memory _name,
    121:        string memory _symbol,
    122:        string memory _baseURI,
    136:    function tokenURI(uint256 _id) public view override returns (string memory) {
    140:        return string(abi.encodePacked(baseURI, _id, ".json"));
    167:        string calldata _subprotocolName,
    239:        string calldata _subprotocolName,
    297:        string calldata _subprotocolName,
    307:    function getPrimaryData(uint256 _cidNFTID, string calldata _subprotocolName)
    319:    function getActiveData(uint256 _cidNFTID, string calldata _subprotocolName)
    333:        string calldata _subprotocolName,

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol

## [G-06] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.
Contexts: 01
File: main/src/CidNFT.sol

    70:    mapping(uint256 => mapping(string => SubprotocolData)) internal cidData;

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L70

## [G-07] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.
Contexts: 01
File: main/src/CidNFT.sol

    150:        for (uint256 i = 0; i < _addList.length; ++i) {

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L150

## [G-08] ADDED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT
EX: require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
Contexts: 01
File: main/src/CidNFT.sol

    193:            SafeTransferLib.safeTransferFrom(note, msg.sender, subprotocolOwner, subprotocolFee - cidFee);

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L193