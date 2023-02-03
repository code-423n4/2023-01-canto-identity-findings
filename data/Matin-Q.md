
## Summary

### Low-Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | ```indexed``` keyword for reference type variables returns the obscure 32 bytes hash of the ```string```. | 7 | 

Total: 7 instances over 1 issue


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Use a more recent version of solidity | 4 | 
| [N&#x2011;02] | Use scientific notation (e.g. `1e18`) rather than exponential form (e.g. `10**18`) | 1 | 
| [N&#x2011;03] | File is missing NatSpec | 2 | 
| [N&#x2011;04] | Consider using ```delete``` rather than assigning zero to default values | 1 | 

Total: 8 instances over 4 issues

Note: The table above was created considering the **automatic findings** and thus, those are not included.



## Low-Risk Issues

### [L&#x2011;01]  ```indexed``` keyword for reference type variables returns the obscure 32 bytes hash of the ```string```
when the ```indexed``` keyword is used for reference typed variables such as string, it will return the hash of the mentioned string.
Thus, the event which is supposed to inform all of the applications subscribed to its emitting transaction (e.g. front-end of the DApp), 
would get a meaningless and obscure 32 bytes that correspond to keccak256 of an encoded string. For more information about the 
indexed events, one can check here(https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=indexed#events).

*There are 7 instances of this issue:*

```solidity
    event SubprotocolRegistered(
        address indexed registrar,
        string indexed name,
        address indexed nftAddress,
        bool ordered,
        bool primary,
        bool active,
        uint96 fee
    );
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L47

```solidity
    event OrderedDataAdded(
        uint256 indexed cidNFTID,
        string indexed subprotocolName,
        uint256 indexed key,
        uint256 subprotocolNFTID
    );
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L77

```solidity
    event PrimaryDataAdded(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L81

```solidity
    event ActiveDataAdded(
        uint256 indexed cidNFTID,
        string indexed subprotocolName,
        uint256 subprotocolNFTID,
        uint256 arrayIndex
    );
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L84

```solidity
    event OrderedDataRemoved(
        uint256 indexed cidNFTID,
        string indexed subprotocolName,
        uint256 indexed key,
        uint256 subprotocolNFTID
    );
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L90

```solidity
    event PrimaryDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L94

```solidity
    event ActiveDataRemoved(uint256 indexed cidNFTID, string indexed subprotocolName, uint256 subprotocolNFTID);
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L95

## Non-critical Issues

### [N&#x2011;01]  Use a more recent version of solidity.
Using version 0.8.17 for the solidity compiler is better.

*There are 4 instances of this issue:*

```solidity
    pragma solidity >=0.8.0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L2
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L2
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/AddressRegistry.sol#L2
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidSubprotocolNFT.sol#L2
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L22



### [N&#x2011;02]  Use scientific notation (e.g. `1e18`) rather than exponential form (e.g. `10**18`)

*There is 1 instance of this issue:*

```solidity
    uint256 public constant REGISTER_FEE = 100 * 10**18;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/SubprotocolRegistry.sol#L17




### [N&#x2011;03]  File is missing NatSpec
Some functions miss NatSpec (@inheritdoc)

*There are 2 instances of this issue:*

```solidity
    function onERC721Received(
        address, /*operator*/
        address, /*from*/
        uint256, /*id*/
        bytes calldata /*data*/
    ) external pure returns (bytes4) {
        return ERC721TokenReceiver.onERC721Received.selector;
    }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L339

```solidity
    function supportsInterface(bytes4 interfaceId) public pure override returns (bool) {
        return interfaceId == CID_SUBPROTOCOL_INTERFACE_ID || super.supportsInterface(interfaceId);
    }
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidSubprotocolNFT.sol#L16




### [N&#x2011;04]  Consider using ```delete``` rather than assigning zero to default values

*There is 1 instance of this issue:*

```solidity
    activeData.positions[_nftIDToRemove] = 0;
```
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L284
