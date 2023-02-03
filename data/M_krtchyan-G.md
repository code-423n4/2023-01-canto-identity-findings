# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 1 |
| [GAS-2](#GAS-2) | Functions guaranteed to revert when called by normal users can be marked `payable` | 6 |
| [GAS-3](#GAS-3) | Use nested if and, avoid multiple check combinations | 1 |
| [GAS-4](#GAS-4) | Ternary over `if ... else` | 1 |
| [GAS-5](#GAS-5) | Check and then add data to the structure | 1 |
## [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L85
```solidity
File: /src/SubprotocolRegistry.sol

//@audit: _fee 
85:        uint96 _fee;
```

## [G-02] Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.The extra opcodes avoided costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

`Total Instances: 6 `
`Gas savings: 6 * 21 = 126 gas`

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L42
```solidity
File: /src/AddressRegistry.sol
42:    function register(uint256 _cidNFTID) external {

52:    function getCID(address _user) external view returns (uint256 cidNFTID) {
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L79
```solidity
File: /src/SubprotocolRegistry.sol
79:    function register(
        bool _ordered,
        bool _primary,
        bool _active,
        address _nftAddress,
        string calldata _name,
        uint96 _fee
    ) external {

106:    function getSubprotocol(string calldata _name) external view returns (SubprotocolData memory) {
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147
```solidity
File: /src/SubprotocolRegistry.sol
147:    function mint(bytes[] calldata _addList) external {

165:    function add(
            uint256 _cidNFTID,
            string calldata _subprotocolName,
            uint256 _key,
            uint256 _nftIDToAdd,
            AssociationType _type
        ) 
        
237:    function remove(
            uint256 _cidNFTID,
            string calldata _subprotocolName,
            uint256 _key,
            uint256 _nftIDToRemove,
            AssociationType _type
        ) public {
```

### [G-03] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L251

```solidity
1 results 

src\CidNFT.sol:
  251: if (
            cidNFTOwner != msg.sender &&
            getApproved[_cidNFTID] != msg.sender &&
            !isApprovedForAll[cidNFTOwner][msg.sender]
        )
```

### [G-04]  Ternary over `if ... else`
Using ternary operator instead of the if else statement saves gas.

For instance, the code block below may be refactored as follows:

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L215
```
 lengthBeforeAddition == 0
     ? {
             uint256[] memory nftIDsToAdd = new uint256[](1);
                nftIDsToAdd[0] = _nftIDToAdd;
                activeData.values = nftIDsToAdd;
                activeData.positions[_nftIDToAdd] = 1; // Array index + 1
     }
     : {
             // Check for duplicates
                if (activeData.positions[_nftIDToAdd] != 0)
                    revert ActiveArrayAlreadyContainsID(_cidNFTID, _subprotocolName, _nftIDToAdd);
                activeData.values.push(_nftIDToAdd);
                activeData.positions[_nftIDToAdd] = lengthBeforeAddition + 1;
     }
```
### [G-05]  Check and then add data to the structure
it would be optimal to do a check and then add data to the structure, and not add 2 parameters, then if (), then the rest
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L93
```
        if (!ERC721(_nftAddress).supportsInterface(type(CidSubprotocolNFT).interfaceId))
            revert NotASubprotocolNFT(_nftAddress);
        subprotocolData.owner = msg.sender;
        subprotocolData.fee = _fee;
        subprotocolData.nftAddress = _nftAddress;
        subprotocolData.ordered = _ordered;
        subprotocolData.primary = _primary;
        subprotocolData.active = _active;
        subprotocols[_name] = subprotocolData;
```

