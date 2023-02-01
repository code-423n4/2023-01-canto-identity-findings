      
###  [L01] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE   
      
#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::119-131 =>      constructor(
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

2023-01-canto-identity/src/SubprotocolRegistry.sol::65-68 =>     constructor(address _noteContract, address _cidFeeWallet) {
        note = ERC20(_noteContract);
        cidFeeWallet = _cidFeeWallet;
    }

2023-01-canto-identity/src/AddressRegistry.sol::36-38 =>     constructor(address _cidNFT) {
        cidNFT = _cidNFT;
    }

```
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

### Recommended Mitigation Steps
Add Event-Emit.

### Non-Critical Issues


### [N01] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()

#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::140 =>  return string(abi.encodePacked(baseURI, _id, ".json"));
2023-01-canto-identity/src/CidNFT.sol::154 => ) = address(this).delegatecall(abi.encodePacked(addSelector, _addList[i]));
```

      
###  [N02] SHOWING THE ACTUAL VALUES OF NUMBERS IN NATSPEC COMMENTS MAKES CHECKING AND READING CODE EASIER   
      
#### Findings:
```
2023-01-canto-identity/src/CidNFT.sol::17 =>  ruint256 public constant CID_FEE_BPS = 1_000;
2023-01-canto-identity/src/SubprotocolRegistry.sol::17 => uint256 public constant REGISTER_FEE = 100 * 10**18;
```




